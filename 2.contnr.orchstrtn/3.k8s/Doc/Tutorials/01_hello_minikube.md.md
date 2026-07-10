# 🎯 Практика 1: Hello Minikube — первый деплой в Kubernetes

> 📌 **Цель**: Развернуть первое приложение в minikube, понять базовые команды kubectl, увидеть связь между Pod, Deployment и Service.
> 
> **Связь с теорией**: [[02.deployaments]], [[02.service]], [[01.pod]], [[01.k8s_overview]]
> 
> **Время**: 30-45 минут

---

## Цели практики

- [ ] Запустить minikube и проверить состояние кластера
- [ ] Создать Deployment с тестовым приложением
- [ ] Посмотреть Pod'ы, события, логи
- [ ] Создать Service для доступа к приложению
- [ ] Включить addon metrics-server
- [ ] Очистить ресурсы

---

## 🔧 Шаг 0: Подготовка окружения

### Проверка: minikube установлен?

```bash
minikube version
```

**Ожидаемый результат**:
```
minikube version: v1.32.0 (или новее)
commit: ...
```

**❌ Если не установлен**:
```bash
# macOS
brew install minikube

# Linux (amd64)
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Windows (Chocolatey)
choco install minikube
```

### Проверка: kubectl установлен?

```bash
kubectl version --client
```

**Ожидаемый результат**:
```
Client Version: v1.29.0 (или новее)
```

**❌ Если не установлен**:
```bash
# macOS
brew install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install kubectl /usr/local/bin/kubectl

# Windows
choco install kubectl
```

### Проверка: Docker установлен?

```bash
docker --version
```

**Ожидаемый результат**:
```
Docker version 24.0.7 (или новее)
```

> 💡 **Примечание**: minikube может использовать Docker, VirtualBox, HyperKit и другие драйверы. Docker — самый простой вариант.

---

## 🚀 Шаг 1: Запуск minikube

### Команда

```bash
minikube start
```

### Что происходит?

1. Создаётся виртуальная машина (или контейнер) для кластера
2. Устанавливается Kubernetes control plane
3. Настраивается kubeconfig для kubectl
4. Запускаются системные Pod'ы (coredns, etcd, kube-apiserver, etc.)

### Ожидаемый результат

```
😄  minikube v1.32.0 on Darwin 14.2.1
✨  Using the docker driver based on user configuration
📌  Using Docker Desktop driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=4000MB) ...
🐳  Preparing Kubernetes v1.29.0 on Docker 24.0.7 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube" cluster
```

### Проверка статуса

```bash
minikube status
```

**Ожидаемый результат**:
```
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

### ✅ Чек-лист шага 1

- [x] minikube запущен (`minikube start` выполнен успешно)
- [x] Все компоненты в статусе `Running` (`minikube status`)
- [x] kubectl настроен на minikube (`kubectl config current-context` показывает `minikube`)

---

## 🖥️ Шаг 2: Dashboard (опционально)

### Команда

```bash
minikube dashboard
```

### Что происходит?

Открывается веб-интерфейс Kubernetes Dashboard в браузере. Это GUI для управления кластером.

### Если не хочешь открывать браузер

```bash
minikube dashboard --url
```

**Ожидаемый результат**:
```
http://127.0.0.1:XXXXX/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```

> 💡 **Совет**: Dashboard полезен для новичков, но в production обычно не используется. Лучше учиться через kubectl.

### ✅ Чек-лист шага 2

- [x] Dashboard открылся (или URL получен)
- [x] Можешь видеть Pod'ы, Deployments, Services в UI

---

## 📦 Шаг 3: Создание Deployment

### Команда

```bash
kubectl create deployment hello-node \
  --image=registry.k8s.io/e2e-test-images/agnhost:2.53 \
  -- /agnhost netexec --http-port=8080
```

### Разбор команды

| Часть | Что делает |
|-------|------------|
| `kubectl create deployment` | Создаёт Deployment (императивный подход) |
| `hello-node` | Имя Deployment |
| `--image=...` | Образ контейнера |
| `-- /agnhost netexec --http-port=8080` | Команда, которая запустится в контейнере |

### Что такое `agnhost`?

Это тестовый образ от Kubernetes, который запускает простой HTTP-сервер на порту 8080. Он отвечает на все запросы.

### Что происходит?

1. Создаётся Deployment `hello-node`
2. Deployment создаёт ReplicaSet
3. ReplicaSet создаёт Pod с контейнером `agnhost`
4. Kubelet на ноде запускает контейнер

### Проверка Deployment

```bash
kubectl get deployments
```

**Ожидаемый результат**:
```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
hello-node   1/1     1            1           1m
```

### Разбор колонок

| Колонка | Что означает |
|---------|--------------|
| `READY` | Сколько Pod'ов готово / сколько должно быть |
| `UP-TO-DATE` | Сколько Pod'ов обновлены до последней версии |
| `AVAILABLE` | Сколько Pod'ов доступны для трафика |
| `AGE` | Как давно создан Deployment |

### Проверка Pod'ов

```bash
kubectl get pods
```

**Ожидаемый результат**:
```
NAME                          READY   STATUS    RESTARTS   AGE
hello-node-5f76cf6ccf-br9b5   1/1     Running   0          1m
```

### Разбор колонок

| Колонка | Что означает |
|---------|--------------|
| `NAME` | Имя Pod'а (генерируется автоматически: `<deployment>-<replicaset>-<hash>`) |
| `READY` | Сколько контейнеров готово / сколько должно быть |
| `STATUS` | Текущий статус: Pending, Running, Succeeded, Failed, Unknown |
| `RESTARTS` | Сколько раз контейнер перезапускался |
| `AGE` | Как давно создан Pod |

### Проверка событий

```bash
kubectl get events
```

**Ожидаемый результат** (последние события):
```
LAST SEEN   TYPE     REASON              OBJECT                               MESSAGE
2m          Normal   Scheduled           pod/hello-node-5f76cf6ccf-br9b5      Successfully assigned default/hello-node-... to minikube
2m          Normal   Pulling             pod/hello-node-5f76cf6ccf-br9b5      Pulling image "registry.k8s.io/e2e-test-images/agnhost:2.53"
2m          Normal   Pulled              pod/hello-node-5f76cf6ccf-br9b5      Successfully pulled image ...
2m          Normal   Created             pod/hello-node-5f76cf6ccf-br9b5      Created container agnhost
2m          Normal   Started             pod/hello-node-5f76cf6ccf-br9b5      Started container agnhost
2m          Normal   SuccessfulCreate    replicaset/hello-node-5f76cf6ccf     Created pod: hello-node-5f76cf6ccf-br9b5
2m          Normal   ScalingReplicaSet   deployment/hello-node                Scaled up replica set hello-node-... to 1
```

### Разбор событий

| Тип | Что означает |
|-----|--------------|
| `Scheduled` | Планировщик назначил Pod на ноду |
| `Pulling` | Kubelet загружает образ |
| `Pulled` | Образ загружен |
| `Created` | Контейнер создан |
| `Started` | Контейнер запущен |
| `ScalingReplicaSet` | ReplicaSet масштабируется |

### Проверка конфигурации kubectl

```bash
kubectl config view
```

**Ожидаемый результат** (фрагмент):
```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/.../.minikube/ca.crt
    server: https://127.0.0.1:XXXXX
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
users:
- name: minikube
  user:
    client-certificate: /Users/.../.minikube/profiles/minikube/client.crt
    client-key: /Users/.../.minikube/profiles/minikube/client.key
```

### Разбор

| Поле | Что означает |
|------|--------------|
| `clusters` | Список кластеров (в нашем случае — minikube) |
| `contexts` | Список контекстов (связка cluster + user) |
| `current-context` | Активный контекст (куда kubectl отправляет команды) |
| `users` | Список пользователей (credentials для аутентификации) |

### Просмотр логов

```bash
# Сначала получим имя Pod'а
POD_NAME=$(kubectl get pods -l app=hello-node -o jsonpath='{.items[0].metadata.name}')
echo "Pod name: $POD_NAME"

# Теперь посмотрим логи
kubectl logs $POD_NAME
```

**Ожидаемый результат**:
```
I0911 09:19:26.677397       1 log.go:195] Started HTTP server on port 8080
I0911 09:19:26.677587       1 log.go:195] Started UDP server on port  8081
```

### ✅ Чек-лист шага 3

- [x] Deployment создан (`kubectl get deployments` показывает `hello-node`)
- [x] Pod запущен (`kubectl get pods` показывает `Running`)
- [x] События видны (`kubectl get events`)
- [x] Логи доступны (`kubectl logs <pod-name>`)

---

## 🌐 Шаг 4: Создание Service

### Проблема

Pod имеет внутренний IP (например, `10.244.0.5`), который недоступен извне кластера. Как получить доступ к приложению?

### Решение: Service

Service создаёт стабильную точку входа (ClusterIP) и может экспортировать Pod наружу.

### Команда

```bash
kubectl expose deployment hello-node \
  --type=LoadBalancer \
  --port=8080
```

### Разбор команды

| Часть | Что делает |
|-------|------------|
| `kubectl expose deployment` | Создаёт Service для Deployment |
| `hello-node` | Имя Deployment (Service будет направлять трафик на его Pod'ы) |
| `--type=LoadBalancer` | Тип Service (LoadBalancer, NodePort, ClusterIP) |
| `--port=8080` | Порт, который будет слушать Service |

### Что происходит?

1. Создаётся Service `hello-node`
2. Service получает ClusterIP (например, `10.108.144.78`)
3. Service находит Pod'ы с лейблом `app=hello-node` (автоматически от Deployment)
4. kube-proxy настраивает правила iptables для маршрутизации трафика

### Проверка Service

```bash
kubectl get services
```

**Ожидаемый результат**:
```
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-node   LoadBalancer   10.108.144.78   <pending>     8080:30369/TCP   21s
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          23m
```

### Разбор колонок

| Колонка | Что означает |
|---------|--------------|
| `NAME` | Имя Service |
| `TYPE` | Тип Service (ClusterIP, NodePort, LoadBalancer, ExternalName) |
| `CLUSTER-IP` | Внутренний IP Service (доступен только внутри кластера) |
| `EXTERNAL-IP` | Внешний IP (для LoadBalancer в облаке) |
| `PORT(S)` | Порт Service : порт Pod'а |
| `AGE` | Как давно создан Service |

### Почему `<pending>`?

В minikube нет настоящего cloud load balancer, поэтому `EXTERNAL-IP` остаётся `<pending>`. Но мы можем использовать `minikube service` для доступа.

### Доступ к Service

```bash
minikube service hello-node
```

### Что происходит?

Открывается браузер с URL вида `http://192.168.49.2:30369/`, где приложение отвечает.

### Если не хочешь открывать браузер

```bash
minikube service hello-node --url
```

**Ожидаемый результат**:
```
http://192.168.49.2:30369
```

### Тестирование через curl

```bash
# Получим URL
URL=$(minikube service hello-node --url)

# Сделаем запрос
curl $URL
```

**Ожидаемый результат**:
```
NOW: 2024-01-15 10:30:45.123456789 +0000 UTC m=+123.456789012
HOSTNAME: hello-node-5f76cf6ccf-br9b5
CLIENT_VALUES: 192.168.49.1
```

### Попробуй разные endpoints

```bash
# Главная страница
curl $URL/

# Shell (осторожно!)
curl $URL/shell

#任意 путь
curl $URL/any-path
```

> ⚠️ **Предупреждение**: Endpoint `/shell` позволяет выполнять команды в контейнере. **Никогда не используй это в production!**

### ✅ Чек-лист шага 4

- [x] Service создан (`kubectl get services` показывает `hello-node`)
- [ ] Service доступен через `minikube service hello-node`
- [x] curl возвращает ответ от приложения

---

## 🔌 Шаг 5: Включение addon metrics-server

### Что такое addons?

Minikube включает набор встроенных дополнений (addons), которые расширяют функциональность кластера.

### Список addons

```bash
minikube addons list
```

**Ожидаемый результат** (фрагмент):
```
|---------|-------------------------------|----------|
|  ADDON  |            PROFILE            |  STATUS  |
|---------|-------------------------------|----------|
| ingress | minikube                      | disabled |
| metrics-server | minikube               | disabled |
| dashboard | minikube                    | enabled  |
...
```

### Включение metrics-server

```bash
minikube addons enable metrics-server
```

**Ожидаемый результат**:
```
💡  metrics-server is an addon maintained by Kubernetes.
...
🌟  The 'metrics-server' addon is enabled
```

### Что происходит?

1. Создаётся Deployment `metrics-server` в namespace `kube-system`
2. Создаётся Service `metrics-server`
3. metrics-server начинает собирать метрики CPU/memory с Pod'ов и нод

### Проверка Pod'ов

```bash
kubectl get pods -n kube-system | grep metrics
```

**Ожидаемый результат**:
```
metrics-server-67fb648c5-xxxxx   1/1   Running   0   1m
```

### Проверка Service

```bash
kubectl get svc -n kube-system | grep metrics
```

**Ожидаемый результат**:
```
metrics-server   ClusterIP   10.96.241.45   <none>   80/TCP   1m
```

### Тестирование metrics-server

```bash
# Подожди 30-60 секунд, пока metrics-server соберёт данные
sleep 30

# Посмотрим использование ресурсов Pod'ами
kubectl top pods

# Посмотрим использование ресурсов нодами
kubectl top nodes
```

**Ожидаемый результат**:
```
NAME                          CPU(cores)   MEMORY(bytes)
hello-node-5f76cf6ccf-br9b5   1m           6Mi
```

### Если видишь ошибку

```
error: Metrics API not available
```

**Решение**: подожди ещё 30-60 секунд. metrics-server нужно время на запуск и сбор данных.

### Отключение metrics-server

```bash
minikube addons disable metrics-server
```

**Ожидаемый результат**:
```
🔥  metrics-server was successfully disabled
```

### ✅ Чек-лист шага 5

- [x] metrics-server включён
- [x] Pod metrics-server запущен (`kubectl get pods -n kube-system`)
- [x] `kubectl top pods` показывает использование ресурсов
- [x] metrics-server отключён (опционально)

---

## 🧹 Шаг 6: Очистка ресурсов

### Удаление Service

```bash
kubectl delete service hello-node
```

**Ожидаемый результат**:
```
service "hello-node" deleted
```

### Удаление Deployment

```bash
kubectl delete deployment hello-node
```

**Ожидаемый результат**:
```
deployment.apps "hello-node" deleted
```

### Что происходит?

1. Deployment удаляется
2. ReplicaSet удаляется
3. Pod'ы.terminateруются (graceful shutdown)
4. Service удаляется (если не удалил раньше)

### Проверка

```bash
kubectl get all
```

**Ожидаемый результат**:
```
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   30m
```

> 💡 **Примечание**: Service `kubernetes` — это встроенный Service для API server. Его нельзя удалить.

---

## 🛑 Шаг 7: Остановка minikube

### Остановка кластера

```bash
minikube stop
```

**Ожидаемый результат**:
```
✋  Stopping node "minikube"  ...
🛑  Node "minikube" stopped.
```

### Удаление кластера (опционально)

```bash
minikube delete
```

**Ожидаемый результат**:
```
🔥  Deleting "minikube" in docker ...
🔥  Deleting container "minikube" ...
🔥  Removing /Users/.../.minikube/machines/minikube ...
💀  Removed all traces of the "minikube" cluster.
```

> 💡 **Совет**: если планируешь продолжать обучение, не удаляй minikube — можно просто остановить и запустить снова.

### ✅ Чек-лист шага 7

- [x] minikube остановлен (`minikube stop`)
- [ ] (Опционально) minikube удалён (`minikube delete`)

---

## 🎓 Дополнительные задания (для углубления)

### Задание 1: Масштабирование Deployment

```bash
# Снова создай Deployment
kubectl create deployment hello-node \
  --image=registry.k8s.io/e2e-test-images/agnhost:2.53 \
  -- /agnhost netexec --http-port=8080

# Масштабируй до 3 реплик
kubectl scale deployment hello-node --replicas=3

# Проверь
kubectl get pods
kubectl get deployment hello-node
```

**Вопрос**: сколько Pod'ов ты видишь? Как изменился Deployment?

### Задание 2: Rolling Update

```bash
# Обнови образ (используем другой тег)
kubectl set image deployment/hello-node \
  agnhost=registry.k8s.io/e2e-test-images/agnhost:2.54

# Проверь статус rollout
kubectl rollout status deployment/hello-node

# Посмотри историю
kubectl rollout history deployment/hello-node

# Откати
kubectl rollout undo deployment/hello-node
```

**Вопрос**: что произошло с Pod'ами во время update? Как работает rolling update?

### Задание 3: Declarative подход

```bash
# Создай файл deployment.yaml
cat > deployment.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-node
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-node
  template:
    metadata:
      labels:
        app: hello-node
    spec:
      containers:
      - name: agnhost
        image: registry.k8s.io/e2e-test-images/agnhost:2.53
        args: ["/agnhost", "netexec", "--http-port=8080"]
        ports:
        - containerPort: 8080
EOF

# Примени
kubectl apply -f deployment.yaml

# Проверь
kubectl get deployment hello-node
kubectl get pods
```

**Вопрос**: в чём разница между `kubectl create` и `kubectl apply`?

### Задание 4: Service разных типов

```bash
# Создай Service типа NodePort
kubectl expose deployment hello-node \
  --type=NodePort \
  --port=8080 \
  --name=hello-node-nodeport

# Проверь
kubectl get svc hello-node-nodeport

# Получи URL
minikube service hello-node-nodeport --url
```

**Вопрос**: в чём разница между LoadBalancer и NodePort?

### Задание 5: Просмотр деталей

```bash
# Детальная информация о Deployment
kubectl describe deployment hello-node

# Детальная информация о Pod
kubectl describe pod <pod-name>

# Детальная информация о Service
kubectl describe service hello-node

# YAML манифест Deployment
kubectl get deployment hello-node -o yaml
```

**Вопрос**: что ты узнал нового из `describe` и `-o yaml`?

---

## 📊 Итоговый чек-лист практики

### Базовые навыки

- [ ] Запустил minikube (`minikube start`)
- [ ] Проверил статус (`minikube status`)
- [ ] Создал Deployment (`kubectl create deployment`)
- [ ] Посмотрел Pod'ы (`kubectl get pods`)
- [ ] Посмотрел события (`kubectl get events`)
- [ ] Посмотрел логи (`kubectl logs`)
- [ ] Создал Service (`kubectl expose`)
- [ ] Получил доступ к Service (`minikube service`)
- [ ] Включил addon (`minikube addons enable`)
- [ ] Проверил метрики (`kubectl top pods`)
- [ ] Очистил ресурсы (`kubectl delete`)
- [ ] Остановил minikube (`minikube stop`)

### Дополнительные задания

- [ ] Масштабировал Deployment (`kubectl scale`)
- [ ] Сделал rolling update (`kubectl set image`)
- [ ] Откатил rollout (`kubectl rollout undo`)
- [ ] Применил декларативный манифест (`kubectl apply -f`)
- [ ] Создал Service типа NodePort
- [ ] Посмотрел детали через `describe` и `-o yaml`

---

## 🔗 Связь с теорией

| Что сделал | Где в теории |
|------------|--------------|
| Создал Deployment | [[02.deployaments]] |
| Посмотрел Pod'ы | [[01.pod]], [[02.pod_lifecycle]] |
| Создал Service | [[02.service]] |
| Посмотрел события | [[03.pod_conditions]] |
| Включил metrics-server | [[03.observability]] |
| Сделал rolling update | [[02.deployaments]] (секция про updates) |
| Применил YAML | [[02.object_management]] |

---

## 💡 Что ты узнал

1. **Deployment** — это способ управлять Pod'ами (создание, масштабирование, обновление)
2. **Pod** — минимальная единица деплоя, содержит один или несколько контейнеров
3. **Service** — стабильная точка входа для Pod'ов (ClusterIP, NodePort, LoadBalancer)
4. **kubectl** — основной инструмент для работы с K8s (create, get, describe, logs, delete)
5. **minikube** — локальный кластер для обучения и разработки
6. **addons** — дополнения для расширения функциональности (metrics-server, dashboard, ingress)

---

## 🎯 Что дальше?

После этой практики ты готов к:

1. **Практике 2**: Declarative подход — создание YAML манифестов
2. **Практике 3**: Работа с ConfigMap и Secret
3. **Практике 4**: NetworkPolicy — настройка сетевых правил
4. **Практике 5**: HPA — автомасштабирование по CPU/memory

Или вернись к **Уровню 1: База** в roadmap и продолжай изучать теорию.

---
