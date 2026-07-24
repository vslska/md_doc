
---
# 🎯 Практика 4.1: Предоставление внешнего доступа через Service типа LoadBalancer

> 📌 **Источник**: [kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/](https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/)
> 
> **Связь с теорией**: 
> - [[02.service]] — типы Service, селекторы, Endpoints
> - [[01.nodes]] — роль kube-proxy и Cloud Controller Manager
> - [[02.deployaments]] — управление репликами Stateless-приложений
> 
> **Время**: 30-40 минут

---

## 🎯 Цели

- [ ] Развернуть масштабируемое Stateless-приложение (5 реплик).
- [ ] Создать Service типа `LoadBalancer` и понять, чем он отличается от `NodePort` и `ClusterIP`.
- [ ] Проанализировать связь между Service, Endpoints и реальными IP-адресами Pod'ов.
- [ ] Получить доступ к приложению через внешний IP-адрес.

---

## 📚 Теория: Механика LoadBalancer "под капотом"

### 1. Что такое Service типа LoadBalancer?
Это надстройка над типом `NodePort`. Когда ты создаешь такой Service, происходит магия (но мы её разберем):
1. Kubernetes создает Service с типом `ClusterIP` и `NodePort` (как обычно).
2. **Cloud Controller Manager (CCM)** — компонент плоскости управления, специфичный для облака (AWS, GCP, Yandex Cloud), замечает новый Service типа `LoadBalancer`.
3. CCM делает API-вызов к облачному провайдеру: *"Создай мне внешний балансировщик нагрузки (например, AWS NLB или Yandex ALB)"*.
4. Облачный провайдер создает реальный сетевой ресурс, выделяет **публичный (внешний) IP-адрес** и настраивает его так, чтобы он перенаправлял трафик на `NodePort` всех рабочих узлов кластера.
5. Статус Service меняется с `<pending>` на реальный IP-адрес в поле `EXTERNAL-IP` (или `LoadBalancer Ingress`).

### 2. Поток трафика (Traffic Flow)
```text 
Интернет (Пользователь) 
   ↓
[Внешний IP: 8080] (Облачный Load Balancer)
   ↓
[Node IP 1: NodePort] или [Node IP 2: NodePort] (Любая нода кластера)
   ↓
kube-proxy (правила iptables/IPVS на ноде)
   ↓
[Pod IP: 8080] (Один из 5 запущенных Pod'ов, выбранный случайно)
```

### 3. Важное примечание для локального обучения (Minikube / Kind)
Поскольку у тебя нет реального облачного провайдера с CCM, внешний IP навсегда останется в статусе `<pending>`. 
**Решение:** Minikube имеет специальную команду `minikube tunnel`, которая эмулирует работу облачного Load Balancer'а, выделяя локальный IP-адрес из диапазона `127.0.0.0/8` или `192.168.x.x`.

---

## 🚀 Практика: Пошаговое выполнение

### Шаг 1: Развертывание Stateless-приложения

Создадим Deployment с 5 репликами простого веб-приложения.

```yaml
# load-balancer-example.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: load-balancer-example
  name: hello-world
spec:
  replicas: 5
  selector:
    matchLabels:
      app.kubernetes.io/name: load-balancer-example
  template:
    metadata:
      labels:
        app.kubernetes.io/name: load-balancer-example
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:2.0
        name: hello-world
        ports:
        - containerPort: 8080
```

Применяем и проверяем:
```bash
kubectl apply -f load-balancer-example.yaml
kubectl get deployments hello-world
kubectl get pods -l app.kubernetes.io/name=load-balancer-example -o wide
```
*Обрати внимание на колонку `IP` и `NODE`. У тебя 5 Pod'ов с разными внутренними IP, возможно, разбросанных по разным нодам (если кластер многоузловой).*

---

### Шаг 2: Создание Service типа LoadBalancer

Используем императивную команду для создания Service, который нацелен на наш Deployment.

```bash
kubectl expose deployment hello-world --type=LoadBalancer --name=my-service
```

Проверяем статус:
```bash
kubectl get services my-service
```

**Ожидаемый результат (в реальном облаке):**
```text
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)          AGE
my-service   LoadBalancer   10.3.245.137   104.198.205.71   8080:32377/TCP   54s
```

**Ожидаемый результат (в Minikube без tunnel):**
```text
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
my-service   LoadBalancer   10.109.0.0     <pending>     8080:30123/TCP   10s
```

---

### Шаг 3: Глубокий анализ Service (Связываем точки)

Выполним команду `describe`, чтобы увидеть, как Kubernetes связывает абстрактный Service с реальными Pod'ами.

```bash
kubectl describe services my-service
```

**Ключевые строки в выводе, которые ты должен понять:**
1. **`Selector: app.kubernetes.io/name=load-balancer-example`** → По этой метке Service ищет цели.
2. **`Type: LoadBalancer`** → Тип сервиса.
3. **`IP: 10.3.245.137`** → Внутренний ClusterIP (работает всегда).
4. **`LoadBalancer Ingress: 104.198.205.71`** → Тот самый внешний IP, выданный облаком (или Minikube).
5. **`NodePort: 32377/TCP`** → Порт, открытый на каждой ноде кластера (диапазон 30000-32767).
6. **`Endpoints: 10.0.0.6:8080, 10.0.1.6:8080, 10.0.1.7:8080 + 2 more...`** → **САМОЕ ВАЖНОЕ!** Это актуальный список `IP:Port` всех Pod'ов, которые соответствуют селектору. Именно сюда `kube-proxy` будет направлять трафик.

---

### Шаг 4: Проверка доступа (С учетом твоего окружения)

#### Вариант А: Ты используешь Minikube (Рекомендуемый способ для эмуляции)
1. Открой **новое окно терминала** и запусти (потребуются права sudo на Linux/macOS):
   ```bash
   minikube tunnel
   ```
   *Введи пароль, если попросит. Этот процесс будет работать в фоне, эмулируя облачный контроллер.*
2. В первом окне проверь Service снова:
   ```bash
   kubectl get services my-service
   ```
   *Теперь в колонке `EXTERNAL-IP` должен появиться реальный IP (например, `127.0.0.1` или `192.168.49.2`).*
3. Сделай запрос:
   ```bash
   # Замени <EXTERNAL-IP> на тот, что ты увидел в get services
   curl http://<EXTERNAL-IP>:8080
   ```

#### Вариант Б: Быстрый способ через NodePort (Если не хочешь использовать tunnel)
```bash
minikube service my-service --url
# Скопируй полученный URL и сделай curl по нему
curl http://127.0.0.1:XXXXX
```

**Ожидаемый ответ от приложения:**
```text
Hello, world!
Version: 2.0.0
Hostname: hello-world-6848c5c6f9-xxxxx
```
*Запусти `curl` несколько раз. Ты увидишь, как меняется `Hostname`. Это доказывает, что балансировка нагрузки работает и запросы попадают на разные Pod'ы!*

---

## 🧪 Практические задания (для глубокого понимания)

### Задание 1: Наблюдение за динамическими Endpoints в реальном времени
Service динамически отслеживает Pod'ы. Давай это проверим.

1. В **Терминале 1** запусти наблюдение за Endpoints:
   ```bash
   kubectl get endpoints my-service -w
   ```
2. В **Терминале 2** масштабируй приложение до 6 реплик:
   ```bash
   kubectl scale deployment hello-world --replicas=6
   ```
3. **Наблюдение:** В Терминале 1 ты почти мгновенно увидишь, как к списку `Endpoints` добавился шестой IP-адрес. Тебе не нужно перезагружать Service или настраивать балансировщик вручную. Kubernetes сделал это сам.

### Задание 2: Проверка устойчивости (Self-healing)
1. Удали один из Pod'ов вручную: `kubectl delete pod <имя_одного_из_pod_ов>`.
2. Сразу же сделай несколько запросов `curl http://<EXTERNAL-IP>:8080`.
3. **Вопрос:** Приложение упало? (Ответ: Нет, потому что Service мгновенно удалил IP умирающего Pod'а из `Endpoints`, и трафик пошел на оставшиеся 4 здоровых Pod'а, пока ReplicaSet создавал 5-й).

---

## 🐛 Troubleshooting

| Симптом | Возможная причина | Решение |
|---------|-------------------|---------|
| `EXTERNAL-IP` вечно висит как `<pending>` | Ты используешь Minikube/Kind без запущенного `minikube tunnel` или `metalLB`. | Запусти `minikube tunnel` в отдельном терминале или используй `minikube service <name>`. |
| `curl` возвращает `Connection refused` | Ты пытаешься обратиться к `ClusterIP` извне кластера или указал неверный порт. | Используй именно `EXTERNAL-IP` (или `minikube service --url`) и порт из колонки `PORT(S)` (первое число, до двоеточия). |
| В `Endpoints` пусто (`<none>`) | 1. Метки в `selector` Service не совпадают с метками Pod'ов. 2. Pod'ы не прошли `readiness` проверку. | Проверь: `kubectl get pods --show-labels` и сравни с `kubectl get service my-service -o yaml`. |

---

## 📊 Итоговый чек-лист

### Теория
- [ ] Понимаю роль Cloud Controller Manager в создании реального облачного балансировщика.
- [ ] Знаю полный путь трафика: External IP → NodePort → kube-proxy → Pod IP.
- [ ] Понимаю, что `Endpoints` — это динамический мост между Service и Pod'ами.

### Практика
- [ ] Развернул Deployment с 5 репликами.
- [ ] Создал Service типа `LoadBalancer`.
- [ ] Проанализировал вывод `kubectl describe service`, найдя Selector, NodePort и Endpoints.
- [ ] Успешно получил ответ от приложения через внешний IP (с помощью `minikube tunnel` или `minikube service`).
- [ ] Пронаблюдал, как Endpoints обновляются при масштабировании.

---

## 🔗 Связь с конспектами

| Что изучил на практике | Где почитать теорию в Obsidian |
|------------------------|--------------------------------|
| Тип Service LoadBalancer | [[02.service]] |
| Динамические Endpoints | [[02.service]] |
| Масштабирование Stateless-приложений | [[02.deployaments]] |

---

## 💡 Главный вывод модуля

Service типа `LoadBalancer` — это стандартный способ сделать приложение доступным из интернета в облачных средах. Его главная сила не в том, что он выдает внешний IP, а в том, что он **автоматически и динамически** поддерживает актуальный список `Endpoints`, перенаправляя трафик только на здоровые Pod'ы, независимо от того, как часто они создаются или умирают.

---
