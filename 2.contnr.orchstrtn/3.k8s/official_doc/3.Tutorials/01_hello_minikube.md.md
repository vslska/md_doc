# Hello Minikube — первый деплой в Kubernetes

>**Цели практики:**
>- Запустить локальный кластер minikube и проверить состояние компонента control plane.
>- Создать объект Deployment с тестовым приложением.
>- Проанализировать состояние Pod'ов, системных событий и логов контейнеров.
>- Создать объект Service для обеспечения сетевого доступа к приложению.
>- Включить дополнение metrics-server и проверить параметры сбора метрик.
>- Выполнить корректную очистку созданных ресурсов кластера.

---
## Содержание

1. [[#Подготовка окружения|Подготовка окружения]]
	1. [[#Подготовка окружения#Проверка установки minikube|Проверка установки minikube]]
	2. [[#Подготовка окружения#Проверка установки kubectl|Проверка установки kubectl]]
	3. [[#Подготовка окружения#Проверка установки Docker|Проверка установки Docker]]
2. [[#Запуск minikube|Запуск minikube]]
	1. [[#Запуск minikube#Выполнение команды запуска|Выполнение команды запуска]]
	2. [[#Запуск minikube#Последовательность процессов при запуске|Последовательность процессов при запуске]]
	3. [[#Запуск minikube#Ожидаемый вывод команды|Ожидаемый вывод команды]]
	4. [[#Запуск minikube#Проверка статуса кластера|Проверка статуса кластера]]
	5. [[#Запуск minikube#Контрольный чек-лист запуска|Контрольный чек-лист запуска]]
3. [[#Работа с веб-интерфейсом Dashboard|Работа с веб-интерфейсом Dashboard]]
	1. [[#Работа с веб-интерфейсом Dashboard#Запуск Dashboard|Запуск Dashboard]]
	2. [[#Работа с веб-интерфейсом Dashboard#Получение URL-адреса без автоматического открытия браузера|Получение URL-адреса без автоматического открытия браузера]]
4. [[#Создание объекта Deployment|Создание объекта Deployment]]
	1. [[#Создание объекта Deployment#Выполнение команды создания|Выполнение команды создания]]
	2. [[#Создание объекта Deployment#Структура параметров команды|Структура параметров команды]]
	3. [[#Создание объекта Deployment#Последовательность создания объектов|Последовательность создания объектов]]
	4. [[#Создание объекта Deployment#Проверка состояния Deployment|Проверка состояния Deployment]]
	5. [[#Создание объекта Deployment#Расшифровка столбцов вывода Deployment|Расшифровка столбцов вывода Deployment]]
	6. [[#Создание объекта Deployment#Проверка состояния Pod'ов|Проверка состояния Pod'ов]]
	7. [[#Создание объекта Deployment#Расшифровка столбцов вывода Pod'ов|Расшифровка столбцов вывода Pod'ов]]
	8. [[#Создание объекта Deployment#Просмотр журнала системных событий|Просмотр журнала системных событий]]
	9. [[#Создание объекта Deployment#Классификация системных событий|Классификация системных событий]]
	10. [[#Создание объекта Deployment#Просмотр конфигурации kubectl|Просмотр конфигурации kubectl]]
	11. [[#Создание объекта Deployment#Структура файла конфигурации|Структура файла конфигурации]]
	12. [[#Создание объекта Deployment#Просмотр логов контейнера|Просмотр логов контейнера]]
	13. [[#Создание объекта Deployment#Контрольный чек-лист шага|Контрольный чек-лист шага]]
5. [[#Создание объекта Service|Создание объекта Service]]
	1. [[#Создание объекта Service#Проблема сетевой адресации|Проблема сетевой адресации]]
	2. [[#Создание объекта Service#Назначение абстракции Service|Назначение абстракции Service]]
	3. [[#Создание объекта Service#Команда экспонирования|Команда экспонирования]]
	4. [[#Создание объекта Service#Структура параметров команды|Структура параметров команды]]
	5. [[#Создание объекта Service#Последовательность сетевой настройки|Последовательность сетевой настройки]]
	6. [[#Создание объекта Service#Проверка параметров Service|Проверка параметров Service]]
	7. [[#Создание объекта Service#Расшифровка столбцов вывода Service|Расшифровка столбцов вывода Service]]
	8. [[#Создание объекта Service#Организация доступа к сервису|Организация доступа к сервису]]
	9. [[#Создание объекта Service#Извлечение URL-адреса без запуска браузера|Извлечение URL-адреса без запуска браузера]]
	10. [[#Создание объекта Service#Проверка работоспособности сервиса через curl|Проверка работоспособности сервиса через curl]]
	11. [[#Создание объекта Service#Тестирование эндпоинтов приложения|Тестирование эндпоинтов приложения]]
	12. [[#Создание объекта Service#Контрольный чек-лист шага|Контрольный чек-лист шага]]
6. [[#Включение дополнения metrics-server|Включение дополнения metrics-server]]
	1. [[#Включение дополнения metrics-server#Назначение аддонов minikube|Назначение аддонов minikube]]
	2. [[#Включение дополнения metrics-server#Просмотр списка доступных аддонов|Просмотр списка доступных аддонов]]
	3. [[#Включение дополнения metrics-server#Активация metrics-server|Активация metrics-server]]
	4. [[#Включение дополнения metrics-server#Порядок запуска компонента|Порядок запуска компонента]]
	5. [[#Включение дополнения metrics-server#Проверка статуса запуска Pod'а metrics-server|Проверка статуса запуска Pod'а metrics-server]]
	6. [[#Включение дополнения metrics-server#Проверка состояния сервиса metrics-server|Проверка состояния сервиса metrics-server]]
	7. [[#Включение дополнения metrics-server#Запрос метрик использования ресурсов|Запрос метрик использования ресурсов]]
	8. [[#Включение дополнения metrics-server#Отключение metrics-server|Отключение metrics-server]]
7. [[#Очистка ресурсов|Очистка ресурсов]]
	1. [[#Очистка ресурсов#Удаление сервиса|Удаление сервиса]]
	2. [[#Очистка ресурсов#Удаление Deployment|Удаление Deployment]]
	3. [[#Очистка ресурсов#Последовательность удаления сущностей|Последовательность удаления сущностей]]
	4. [[#Очистка ресурсов#Проверка очистки пространства имен|Проверка очистки пространства имен]]
8. [[#Остановка и удаление кластера minikube|Остановка и удаление кластера minikube]]
	1. [[#Остановка и удаление кластера minikube#Остановка кластера|Остановка кластера]]
	2. [[#Остановка и удаление кластера minikube#Полное удаление кластера (опционально)|Полное удаление кластера (опционально)]]
9. [[#Дополнительные практические задания|Дополнительные практические задания]]
	1. [[#Дополнительные практические задания#Задание 1: Масштабирование объекта Deployment|Задание 1: Масштабирование объекта Deployment]]
	2. [[#Дополнительные практические задания#Задание 2: Проведение поэтапного обновления (Rolling Update)|Задание 2: Проведение поэтапного обновления (Rolling Update)]]
	3. [[#Дополнительные практические задания#Задание 3: Применение декларативного подхода (Declarative Management)|Задание 3: Применение декларативного подхода (Declarative Management)]]
	4. [[#Дополнительные практические задания#Задание 4: Экспонирование через Service типа NodePort|Задание 4: Экспонирование через Service типа NodePort]]
	5. [[#Дополнительные практические задания#Задание 5: Инспекция объектов кластера|Задание 5: Инспекция объектов кластера]]
10. [[#Сводный чек-лист выполнения практики|Сводный чек-лист выполнения практики]]
	1. [[#Сводный чек-лист выполнения практики#Базовый уровень|Базовый уровень]]
	2. [[#Сводный чек-лист выполнения практики#Углубленный уровень|Углубленный уровень]]
11. [[#Ключевые выводы|Ключевые выводы]]



----
## Подготовка окружения

### Проверка установки minikube

Для проверки версии инструмента minikube выполняется команда:

```Bash
minikube version
```

Ожидаемый результат:

```Bash
minikube version: v1.32.0
```

При отсутствии установленной утилиты minikube используется соответствующий сценарий установки в зависимости от операционной системы:

```Bash
# macOS
brew install minikube

# Linux (amd64)
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Windows (Chocolatey)
choco install minikube
```

### Проверка установки kubectl

Верификация клиентской версии утилиты управления `kubectl`:

```Bash
kubectl version --client
```

Ожидаемый результат:

```Bash
Client Version: v1.29.0
```

При отсутствии утилиты `kubectl` выполняется установка:

```Bash
# macOS
brew install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install kubectl /usr/local/bin/kubectl

# Windows
choco install kubectl
```

### Проверка установки Docker

Для проверки версии среды виртуализации Docker используется команда:

```Bash
docker --version
```

Ожидаемый результат:

```
Docker version 24.0.7
```

Примечание: minikube поддерживают работу через драйверы Docker, VirtualBox, HyperKit и другие среды, однако Docker является базовым стандартным драйвером.

---
## Запуск minikube

### Выполнение команды запуска

Запуск локального кластера выполняется с помощью следующей команды:

```bash
minikube start
```

### Последовательность процессов при запуске

1. Создание виртуальной среды или контейнера для узла кластера.
2. Установка и запуск компонентов Kubernetes control plane.
3. Настройка файла конфигурации `kubeconfig` для взаимодействия через `kubectl`.
4. Запуск системных Pod'ов (CoreDNS, etcd, kube-apiserver и др.).

### Ожидаемый вывод команды

```bash
minikube v1.32.0 on Darwin 14.2.1
Using the docker driver based on user configuration
Starting control plane node minikube in cluster minikube
Pulling base image ...
Creating docker container (CPUs=2, Memory=4000MB) ...
Preparing Kubernetes v1.29.0 on Docker 24.0.7 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
Configuring bridge CNI (Container Networking Interface) ...
Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
Enabled addons: default-storageclass, storage-provisioner
Done! kubectl is now configured to use "minikube" cluster
```

### Проверка статуса кластера

```bash
minikube status
```

Ожидаемый результат:

```bash
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

### Контрольный чек-лист запуска

- Команда `minikube start` выполнена без ошибок.
- Все управляющие компоненты находятся в состоянии `Running` по данным `minikube status`.
- Активный контекст `kubectl config current-context` указывает на `minikube`.

---
## Работа с веб-интерфейсом Dashboard

### Запуск Dashboard

Для инициализации и открытия веб-интерфейса управления Kubernetes Dashboard используется команда:

```bash
minikube dashboard
```

В результате работы команды разворачивается графический интерфейс пользователя (GUI) для визуального управления ресурсами кластера.

### Получение URL-адреса без автоматического открытия браузера

```bash
minikube dashboard --url
```

Ожидаемый результат:

```bash
http://127.0.0.1:XXXXX/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```

>**Примечание:** 
>Графический интерфейс удобен для первичного знакомства с ресурсами, но в промышленной эксплуатации основной утилитой администрирования является `kubectl`.

---
## Создание объекта Deployment

### Выполнение команды создания

Для создания сущности Deployment в императивном стиле используется команда:

```bash
kubectl create deployment hello-node \
  --image=registry.k8s.io/e2e-test-images/agnhost:2.53 \
  -- /agnhost netexec --http-port=8080
```

Образ `agnhost` — специализированный тестовый образ Kubernetes, запуск которого инициализирует встроенный HTTP-сервер на порту `8080`, обрабатывающий входящие сетевые запросы.

### Структура параметров команды

| **Параметр / Выражение**               | **Назначение и функциональная роль**                              |
| -------------------------------------- | ----------------------------------------------------------------- |
| `kubectl create deployment`            | Императивная команда создания объекта Deployment                  |
| `hello-node`                           | Уникальное имя создаваемого объекта Deployment                    |
| `--image=...`                          | Указание контейнерного образа для запускных емкостей              |
| `-- /agnhost netexec --http-port=8080` | Передаваемая команда и аргументы для исполнения внутри контейнера |

### Последовательность создания объектов

1. Создается сущность Deployment с именем `hello-node`.
2. Deployment автоматически генерирует и контролирует объект ReplicaSet.
3. ReplicaSet инициализирует Pod с контейнером на базе образа `agnhost`.
4. Системный агент `kubelet` на рабочем узле запускает целевой контейнер.

### Проверка состояния Deployment

```bash
kubectl get deployments
```

Ожидаемый результат:

```bash
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
hello-node   1/1     1            1           1m
```

### Расшифровка столбцов вывода Deployment

|**Название столбца**|**Значение**|
|---|---|
|`READY`|Соотношение готовых Pod'ов к целевому количеству|
|`UP-TO-DATE`|Количество Pod'ов, обновленных до текущей спецификации|
|`AVAILABLE`|Количество Pod'ов, доступных для обслуживания трафика|
|`AGE`|Время, прошедшее с момента создания объекта|

### Проверка состояния Pod'ов
```bash
kubectl get pods
```

Ожидаемый результат:

```bash
NAME                          READY   STATUS    RESTARTS   AGE
hello-node-5f76cf6ccf-br9b5   1/1     Running   0          1m
```

### Расшифровка столбцов вывода Pod'ов

| **Название столбца** | **Значение**                                                                     |
| -------------------- | -------------------------------------------------------------------------------- |
| `NAME`               | Автоматически сгенерированное имя Pod'а (`<deployment>-<replicaset>-<hash>`)     |
| `READY`              | Соотношение готовых контейнеров внутри Pod'а к общему их числу                   |
| `STATUS`             | Текущее состояние Pod'а (`Pending`, `Running`, `Succeeded`, `Failed`, `Unknown`) |
| `RESTARTS`           | Количество перезапусков контейнеров в Pod'е                                      |
| `AGE`                | Время жизни объекта Pod                                                          |

### Просмотр журнала системных событий

```bash
kubectl get events
```

Ожидаемый результат:

```bash
LAST SEEN   TYPE     REASON              OBJECT                               MESSAGE
2m          Normal   Scheduled           pod/hello-node-5f76cf6ccf-br9b5      Successfully assigned default/hello-node-... to minikube
2m          Normal   Pulling             pod/hello-node-5f76cf6ccf-br9b5      Pulling image "registry.k8s.io/e2e-test-images/agnhost:2.53"
2m          Normal   Pulled              pod/hello-node-5f76cf6ccf-br9b5      Successfully pulled image ...
2m          Normal   Created             pod/hello-node-5f76cf6ccf-br9b5      Created container agnhost
2m          Normal   Started             pod/hello-node-5f76cf6ccf-br9b5      Started container agnhost
2m          Normal   SuccessfulCreate    replicaset/hello-node-5f76cf6ccf     Created pod: hello-node-5f76cf6ccf-br9b5
2m          Normal   ScalingReplicaSet   deployment/hello-node                Scaled up replica set hello-node-... to 1
```

### Классификация системных событий

|**Тип события**|**Значение**|
|---|---|
|`Scheduled`|Планировщик `kube-scheduler` назначил Pod на конкретный узел|
|`Pulling`|Агент `kubelet` запустил процесс скачивания контейнерного образа|
|`Pulled`|Скачивание образа успешно завершено|
|`Created`|Контейнер успешно сформирован в среде выполнения|
|`Started`|Контейнер переведен в состояние запуска|
|`ScalingReplicaSet`|Произошло изменение масштаба объекта ReplicaSet|

### Просмотр конфигурации kubectl

```bash
kubectl config view
```

Ожидаемый результат (фрагмент):

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/user/.minikube/ca.crt
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
    client-certificate: /home/user/.minikube/profiles/minikube/client.crt
    client-key: /home/user/.minikube/profiles/minikube/client.key
```

### Структура файла конфигурации

|**Поле**|**Назначение**|
|---|---|
|`clusters`|Перечень доступных кластеров и адресов их API-серверов|
|`contexts`|Связки параметов доступа (указание кластера, пользователя и пространства имен)|
|`current-context`|Активный контекст, на который направляются команды `kubectl`|
|`users`|Учетные данные и сертификаты для аутентификации пользователей|

### Просмотр логов контейнера

```bash
# Автоматическое извлечение имени Pod'а
POD_NAME=$(kubectl get pods -l app=hello-node -o jsonpath='{.items[0].metadata.name}')
echo "Pod name: $POD_NAME"

# Чтение логов процесса
kubectl logs "$POD_NAME"
```

Ожидаемый результат:

```
I0911 09:19:26.677397       1 log.go:195] Started HTTP server on port 8080
I0911 09:19:26.677587       1 log.go:195] Started UDP server on port  8081
```

### Контрольный чек-лист шага

- Объект Deployment успешно сформирован.
- Pod находится в статусе `Running`.
- События жизненного цикла подтверждены через `kubectl get events`.
- Логи приложения доступны для чтения.

---
## Создание объекта Service

### Проблема сетевой адресации

Каждый Pod получает изолированный внутренний IP-адрес внутри сети кластера (например, `10.244.0.5`), немаршрутизируемый из внешней среды. При динамическом пересоздании Pod'а его IP-адрес меняется.

### Назначение абстракции Service

Объект Service предоставляет статическую точку входа (ClusterIP) и организует проксирование и балансировку сетевого трафика на группу Pod'ов с помощью селекторов меток.

### Команда экспонирования

```bash
kubectl expose deployment hello-node \
  --type=LoadBalancer \
  --port=8080
```

### Структура параметров команды

|**Параметр**|**Назначение**|
|---|---|
|`kubectl expose deployment`|Инициализация создания объекта Service для указанного Deployment|
|`hello-node`|Имя целевого объекта Deployment|
|`--type=LoadBalancer`|Тип создаваемого сервиса (`ClusterIP`, `NodePort`, `LoadBalancer`)|
|`--port=8080`|Сетевой порт, публично прослушиваемый сервисом|

### Последовательность сетевой настройки

1. Создается объект Service с именем `hello-node`.
2. Сервису выделяется внутренний IP-адрес из подсети ClusterIP.
3. Сервис связывается с Pod'ами, имеющими метку `app=hello-node`.
4. Системный компонент `kube-proxy` настраивает правила маршрутизации (iptables/IPVS) на узлах.

### Проверка параметров Service

```bash
kubectl get services
```

Ожидаемый результат:

```bash
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-node   LoadBalancer   10.108.144.78   <pending>     8080:30369/TCP   21s
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          23m
```

### Расшифровка столбцов вывода Service

|**Название столбца**|**Значение**|
|---|---|
|`NAME`|Уникальное имя сетевого сервиса|
|`TYPE`|Тип публикации сервиса (`ClusterIP`, `NodePort`, `LoadBalancer`, `ExternalName`)|
|`CLUSTER-IP`|Внутренний виртуальный IP-адрес сервиса|
|`EXTERNAL-IP`|Внешний IP-адрес, выделяемый облачным провайдером|
|`PORT(S)`|Соответствие внешнего порта сервиса внутреннему порту Pod'а/NodePort|
|`AGE`|Время с момента создания сервиса|

> **Причина статуса `<pending>` в поле `EXTERNAL-IP`:** 
> Локальная среда minikube по умолчанию не содержит внешнего облачного балансировщика нагрузки (Cloud Load Balancer). Доступ к сервису осуществляется через проксирующие команды minikube.

### Организация доступа к сервису

```bash
minikube service hello-node
```

При выполнении данной команды открывается веб-браузер со сгенерированным адресом вида `[http://192.168.49.2:30369/](http://192.168.49.2:30369/)`.

### Извлечение URL-адреса без запуска браузера

```bash
minikube service hello-node --url
```

Ожидаемый результат:

```bash
http://192.168.49.2:30369
```

### Проверка работоспособности сервиса через curl

```bash
URL=$(minikube service hello-node --url)
curl -s "$URL"
```

Ожидаемый результат:

```bash
NOW: 2024-01-15 10:30:45.123456789 +0000 UTC m=+123.456789012
HOSTNAME: hello-node-5f76cf6ccf-br9b5
CLIENT_VALUES: 192.168.49.1
```

### Тестирование эндпоинтов приложения

```bash
# Корневой маршрут
curl -s "$URL/"

# Доступ к оболочке командной строки (производственное предупреждение)
curl -s "$URL/shell"

# Произвольный маршрут
curl -s "$URL/any-path"
```

>**Предупреждение по безопасности:** 
 Эндпоинт `/shell` предоставляет возможность выполнения команд в контексте контейнера. Использование данного эндпоинта в продуктивных средах недопустимо по соображениям безопасности.

### Контрольный чек-лист шага

- Объект Service создан и отображается в выводе `kubectl get services`.
- Точка входа доступна через утилиту `minikube service`.
- Утилита `curl` получает валидный ответ от запущенного приложения.

---
## Включение дополнения metrics-server

### Назначение аддонов minikube

Аддоны (addons) представляют собой предконфигурированные расширения, расширяющие базовую функциональность локального кластера minikube.

### Просмотр списка доступных аддонов

```bash
minikube addons list
```

Ожидаемый результат (фрагмент):

```
|---------|-------------------------------|----------|
|  ADDON  |            PROFILE            |  STATUS  |
|---------|-------------------------------|----------|
| ingress | minikube                      | disabled |
| metrics-server | minikube               | disabled |
| dashboard | minikube                    | enabled  |
```

### Активация metrics-server

```bash
minikube addons enable metrics-server
```

Ожидаемый результат:

```bash
The 'metrics-server' addon is enabled
```

### Порядок запуска компонента

1. В пространстве имен `kube-system` создается Deployment `metrics-server`.
2. Создается внутренний сервис `metrics-server`.
3. Сервис начинает регулярный сбор агрегированных метрик использования CPU и оперативной памяти с Pod'ов и узлов.

### Проверка статуса запуска Pod'а metrics-server

```bash
kubectl get pods -n kube-system | grep metrics
```

Ожидаемый результат:

```bash
metrics-server-67fb648c5-xxxxx   1/1   Running   0   1m
```

### Проверка состояния сервиса metrics-server

```bash
kubectl get svc -n kube-system | grep metrics
```

Ожидаемый результат:

```bash
metrics-server   ClusterIP   10.96.241.45   <none>   80/TCP   1m
```

### Запрос метрик использования ресурсов

После завершения инициализации (требуется от 30 до 60 секунд) выполняется считывание показателей нагрузки:

```bash
# Пауза для накопления телеметрии
sleep 30

# Просмотр нагрузки на Pod'ы
kubectl top pods

# Просмотр нагрузки на узлы кластера
kubectl top nodes
```

Ожидаемый результат:

```
NAME                          CPU(cores)   MEMORY(bytes)
hello-node-5f76cf6ccf-br9b5   1m           6Mi
```

При возникновении ошибки `error: Metrics API not available` требуется повторно выждать интервал времени, необходимый компоненту для первичного сбора данных.

### Отключение metrics-server
```bash
minikube addons disable metrics-server
```

---
## Очистка ресурсов

### Удаление сервиса

```bash
kubectl delete service hello-node
```

### Удаление Deployment

```bash
kubectl delete deployment hello-node
```

### Последовательность удаления сущностей

1. Удаляется управляющий объект Deployment.
2. Автоматически удаляется дочерний объект ReplicaSet.
3. Инициируется процедура корректного завершения работы Pod'ов (graceful shutdown).
4. Удаляется объект Service.

### Проверка очистки пространства имен

```bash
kubectl get all
```

Ожидаемый результат:

```bash
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   30m
```

Примечание: Системный сервис `service/kubernetes` обеспечивает связь с API-сервером кластера и не подлежит удалению.

---
## Остановка и удаление кластера minikube

### Остановка кластера

```bash
minikube stop
```

Ожидаемый результат:

```bash
Node "minikube" stopped.
```

### Полное удаление кластера (опционально)

```bash
minikube delete
```

Ожидаемый результат:

```bash
Deleting "minikube" in docker ...
Removed all traces of the "minikube" cluster.
```

---
## Дополнительные практические задания

### Задание 1: Масштабирование объекта Deployment

Императивное изменение количества реплик Pod'ов с помощью команды `kubectl scale`:

```bash
# Создание исходного Deployment
kubectl create deployment hello-node \
  --image=registry.k8s.io/e2e-test-images/agnhost:2.53 \
  -- /agnhost netexec --http-port=8080

# Масштабирование количества реплик до 3
kubectl scale deployment hello-node --replicas=3

# Анализ результатов масштабирования
kubectl get pods
kubectl get deployment hello-node
```

> **Контрольный вопрос:**
>  Каким образом изменилось количество активных Pod'ов и статус объекта ReplicaSet?

### Задание 2: Проведение поэтапного обновления (Rolling Update)

Обновление версии используемого контейнерного образа без остановки сервиса:

```bash
# Назначение нового образа контейнера
kubectl set image deployment/hello-node \
  agnhost=registry.k8s.io/e2e-test-images/agnhost:2.54

# Отслеживание статуса обновления
kubectl rollout status deployment/hello-node

# Просмотр истории версий развертывания
kubectl rollout history deployment/hello-node

# Откат изменения к предыдущему состоянию
kubectl rollout undo deployment/hello-node
```

>**Контрольный вопрос:**
 Каков алгоритм ротации Pod'ов при использовании стратегии RollingUpdate?

### Задание 3: Применение декларативного подхода (Declarative Management)

Создание и применение манифеста конфигурации на базе YAML:

```bash
cat <<EOF > deployment.yaml
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
        command: ["/agnhost"]
        args: ["netexec", "--http-port=8080"]
        ports:
        - containerPort: 8080
EOF

# Декларативное применение конфигурации
kubectl apply -f deployment.yaml

# Проверка результатов
kubectl get deployment hello-node
kubectl get pods
```

>**Контрольный вопрос:** 
>В чем заключаются принципиальные архитектурные различия между командами `kubectl create` и `kubectl apply`?

### Задание 4: Экспонирование через Service типа NodePort

```bash
# Создание сервиса типа NodePort
kubectl expose deployment hello-node \
  --type=NodePort \
  --port=8080 \
  --name=hello-node-nodeport

# Проверка параметров
kubectl get svc hello-node-nodeport

# Извлечение URL
minikube service hello-node-nodeport --url
```

>**Контрольный вопрос:** 
>В чем заключаются отличия между типами сервисов NodePort и LoadBalancer на уровне маршрутизации трафика?

### Задание 5: Инспекция объектов кластера

```bash
# Детальный анализ Deployment
kubectl describe deployment hello-node

# Получение имени Pod'а и его детальный анализ
POD_NAME=$(kubectl get pods -l app=hello-node -o jsonpath='{.items[0].metadata.name}')
kubectl describe pod "$POD_NAME"

# Детальный анализ Service
kubectl describe service hello-node

# Вывод полной спецификации в формате YAML
kubectl get deployment hello-node -o yaml
```

---
## Сводный чек-лист выполнения практики

### Базовый уровень

-  Инициализация кластера `minikube start`
-  Проверка статуса `minikube status`
-  Создание развертывания `kubectl create deployment`
-  Инспекция списка Pod'ов `kubectl get pods`
-  Мониторинг системных событий `kubectl get events`
-  Извлечение логов контейнера `kubectl logs`
-  Создание сетевого сервиса `kubectl expose`
-  Проверка сетевой доступности `minikube service`
-  Активация дополнения `minikube addons enable metrics-server`
-  Считывание показателей `kubectl top pods`
-  Корректное удаление ресурсов `kubectl delete`
-  Остановка виртуальной среды `minikube stop`

### Углубленный уровень

-  Выполнение масштабирования `kubectl scale`
-  Проведение обновлений `kubectl set image` и отката `kubectl rollout undo`
-  Применение декларативных файлов спецификаций `kubectl apply -f`
-  Создание и проверка работы Service типа `NodePort`
-  Глубокий анализ спецификаций с помощью `kubectl describe` и вывода `-o yaml`


---
## Ключевые выводы

1. **Deployment** — управляющая абстракция высокого уровня, декларирующая состояние ReplicaSet и Pod'ов, а также отвечающая за процедуру обновления и масштабирования.
2. **Pod** — базовая единичная абстракция Kubernetes, объединяющая один или несколько контейнеров с общим сетевым стеком и томами.
3. **Service** — абстракция сетевой адресации, обеспечивающая постоянную точку входа и балансировку трафика между Pod'ами.
4. **kubectl** — основной CLI-инструмент управления, взаимодействующий с REST API сервером Kubernetes.
5. **minikube** — утилита развертывания одноузлового кластера Kubernetes для целей разработки и тестирования.
6. **addons** — модульные системы расширения базового функционала кластера.