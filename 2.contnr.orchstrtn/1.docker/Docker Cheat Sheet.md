# Docker Cheat Sheet

> [!SUMMARY] Главное
> Docker — контейнеризация приложений. Образ (image) → Контейнер (container).
> **Image** = read-only шаблон, **Container** = запущенный экземпляр + writable layer.

---

## Основные команды

### Управление контейнерами

| Команда | Описание | Пример |
|---------|----------|--------|
| `docker run` | Запустить контейнер | `docker run -d -p 8080:80 nginx` |
| `docker ps` | Запущенные контейнеры | `docker ps -a` (все) |
| `docker stop` | Остановить (SIGTERM) | `docker stop <id/name>` |
| `docker kill` | Убить (SIGKILL) | `docker kill <id/name>` |
| `docker rm` | Удалить контейнер | `docker rm -f <id>` |
| `docker exec` | Команда в контейнере | `docker exec -it <id> bash` |
| `docker logs` | Просмотр логов | `docker logs -f <id>` |
| `docker inspect` | Детальная инфа | `docker inspect <id>` |

### Управление образами

| Команда | Описание | Пример |
|---------|----------|--------|
| `docker images` | Список образов | `docker images -a` |
| `docker pull` | Скачать образ | `docker pull nginx:alpine` |
| `docker build` | Собрать образ | `docker build -t myapp .` |
| `docker rmi` | Удалить образ | `docker rmi <id/name>` |
| `docker tag` | Добавить тег | `docker tag myapp user/myapp:1.0` |
| `docker push` | Отправить в реестр | `docker push user/myapp` |
| `docker history` | Слои образа | `docker history <image>` |

### Очистка

```bash
docker container prune      # Удалить остановленные контейнеры
docker image prune          # Удалить "висячие" образы
docker volume prune         # Удалить неиспользуемые volumes
docker system prune -a      # Очистить всё (осторожно!)
docker system df            # Что занимает место
```

## docker run: основные флаги
```bash
docker run [OPTIONS] IMAGE [COMMAND]

# Режим
-d, --detach              # Фоновый режим
--rm                      # Автоудаление после остановки
--name <name>             # Имя контейнера
--restart <policy>        # no|on-failure|always|unless-stopped

# Сеть
-p, --publish             # Проброс порта: -p 8080:80
--network <net>           # Сеть: bridge|host|none|custom
--hostname <name>         # Hostname внутри контейнера

# Переменные
-e, --env                 # Env var: -e DB_HOST=localhost
--env-file <file>         # Файл с переменными

# Хранилище
-v, --volume              # Маппинг: -v /host:/container
--mount                   # Явный синтаксис (рекомендуется)

# Ресурсы
--cpus <num>              # Лимит CPU: --cpus=1.5
--memory <size>           # Лимит RAM: --memory=512m
--cpuset-cpus <list>      # Привязка к ядрам: --cpuset-cpus=0,1

# Безопасность
--user <uid:gid>          # Запуск от пользователя
--cap-add <CAP>           # Добавить capability
--cap-drop <CAP>          # Убрать capability (ALL для безопасности)
--read-only               # Read-only root FS
--security-opt            # Seccomp/AppArmor профили
```

### Примеры
```bash
# Веб-сервер
docker run -d -p 8080:80 --name web --restart unless-stopped nginx:alpine

# Приложение с БД
docker run -d \
  --name app \
  -e DATABASE_URL=postgres://user:pass@db:5432/app \
  --mount source=app-data,target=/data \
  --memory=512m --cpus=1.0 \
  myapp:1.0

# Одноразовый тест
docker run --rm -it alpine sh

# От не-root пользователя
docker run -d --user 1000:1000 --cap-drop=ALL nginx
```

## Dockerfile: инструкции
```dockerfile
# БАЗОВЫЙ ОБРАЗ (всегда первая строка)
FROM python:3.11-slim # Сразу есть Python, образ легче Ubuntu

# МЕТАДАННЫЕ
LABEL maintainer="user@example.com"
LABEL version="1.0"

# ПЕРЕМЕННЫЕ (сборка + рантайм)
ENV APP_ENV=production \
    DB_HOST=localhost

# ПЕРЕМЕННЫЕ (только сборка)
ARG NODE_VERSION=18

# РАБОЧАЯ ДИРЕКТОРИЯ
WORKDIR /app

# ВЫПОЛНЕНИЕ КОМАНД (создаёт слой!)
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# КОПИРОВАНИЕ ФАЙЛОВ
COPY src/ ./src/
COPY app.py .
ADD archive.tar.gz /opt/  # + авто-распаковка

# ПОРТ (документация, не публикует!)
EXPOSE 8080

# ТОМЫ (подсказка)
VOLUME ["/data", "/logs"]

# ПОЛЬЗОВАТЕЛЬ (безопасность!)
USER 1000:1000

# HEALTHCHECK
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

# КОМАНДА ЗАПУСКА
CMD ["python", "app.py"]

# ТОЧКА ВХОДА (фиксированная часть)
ENTRYPOINT ["python"]
# CMD становятся аргументами ENTRYPOINT
```

> Best Practices
>
>- Используй конкретные теги (`nginx:1.25-alpine`, не `latest`)
>- Комбинируй команды в один `RUN` для уменьшения слоёв
>- Копируй зависимости до кода (кэш слоёв)
>- Запускай не от root (`USER 1000`)
>- Используй `.dockerignore`


## Хранилище: Volumes vs Bind Mounts
| Тип                | Синтаксис                                     | Где хранится               | Персистентность | Когда использовать   |
| ------------------ | --------------------------------------------- | -------------------------- | --------------- | -------------------- |
| **Volume**         | `--mount source=name,target=/data`            | `/var/lib/docker/volumes/` | Да              | БД, продакшен-данные |
| **Bind Mount**     | `--mount type=bind,source=/host,target=/data` | Любое место на хосте       | Да              | Дев-среда, конфиги   |
| **tmpfs**          | `--mount type=tmpfs,target=/tmp`              | ОЗУ                        | Нет             | Секреты, кэш         |
| **Writable layer** | По умолчанию                                  | overlay2                   | Нет             | Временные файлы      |
### Примеры
```bash
# Volume (рекомендуется для данных)
docker volume create mydata
docker run --mount source=mydata,target=/data nginx

# Bind mount (абсолютный путь!)
docker run --mount type=bind,source="$(pwd)"/config,target=/app/config nginx

# С -v (короткий синтаксис)
docker run -v mydata:/data nginx              # volume
docker run -v "$(pwd)"/config:/app nginx      # bind mount

# tmpfs (в памяти)
docker run --mount type=tmpfs,target=/tmp,tmpfs-size=100m nginx

# Read-only bind mount
docker run --mount type=bind,source=/config,target=/app/config,readonly nginx
```

>[!WARNING] Пути в bind mounts
>
>- `--mount`: только абсолютные пути (`source="$(pwd)"/path`)
>- `-v`: относительные создают volume, а не bind mount!
>- Docker Compose: относительные пути работают

## Сеть

### Основные команды
```bash
docker network ls                   # Список сетей
docker network create mynet         # Создать сеть
docker network inspect mynet        # Инфа о сети
docker network connect mynet app    # Подключить контейнер
docker network disconnect mynet app # Отключить
```

### Типы сетей
| Тип        | Описание                             | Пример                        |
| ---------- | ------------------------------------ | ----------------------------- |
| **bridge** | Изолированная сеть (default)         | `docker run --network bridge` |
| **host**   | Без изоляции (использует сеть хоста) | `docker run --network host`   |
| **none**   | Без сети                             | `docker run --network none`   |
| **custom** | Пользовательская bridge-сеть         | `docker network create mynet` |
### Пример: своя сеть
```bash
# Создать сеть
docker network create mynet

# Запустить контейнеры в одной сети
docker run -d --name db --network mynet postgres
docker run -d --name app --network mynet -e DB_HOST=db myapp

# Контейнеры видят друг друга по имени!
docker exec app ping db  # Работает!
```
## Отладка и мониторинг
```bash
# Логи
docker logs -f --tail 100 <container>
docker logs --since 1h <container>

# Войти в контейнер
docker exec -it <container> bash
docker exec -it <container> sh  # Для alpine

# Процессы внутри
docker top <container>
docker exec <container> ps aux

# Статистика ресурсов
docker stats                    # Все контейнеры
docker stats <container>        # Один контейнер

# Инспекция
docker inspect <container> | jq '.[0].NetworkSettings'
docker inspect <container> --format '{{.NetworkSettings.IPAddress}}'

# Копирование файлов
docker cp <container>:/path/file.txt ./local/
docker cp ./local/file.txt <container>:/path/

# Пауза/непауза
docker pause <container>
docker unpause <container>
```

## Multi-stage сборка
```dockerfile
# Этап сборки
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Этап рантайма
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

>[!TIP] Преимущества
>
>- Меньший размер образа (нет компиляторов, dev-зависимостей)
>- Безопаснее (только runtime-зависимости)
>- Быстрее пуш/пулл

## Анализ образов
```bash
# Размер слоёв
docker history <image> --human --no-trunc

# Что занимает место
docker system df -v

# Сканирование уязвимостей
docker scan <image>
# Или trivy
trivy image <image>

# Экспорт/импорт
docker save myapp:1.0 | gzip > myapp.tar.gz
docker load < myapp.tar.gz
```

## Безопасность: чеклист
```bash
# ✅ Запуск не от root
docker run --user 1000:1000 ...

# ✅ Drop всех capabilities
docker run --cap-drop=ALL ...

# ✅ Добавить только нужные
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE ...

# ✅ Read-only файловая система
docker run --read-only --tmpfs /tmp ...

# ✅ No new privileges
docker run --security-opt=no-new-privileges:true ...

# ✅ Ограничить ресурсы
docker run --memory=512m --cpus=1.0 ...

# ✅ Seccomp профиль (по умолчанию включён)
docker run --security-opt seccomp=default.json ...
```

## Docker Compose (базово)
```yaml
version: '3.8'
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
      - web-data:/data
    environment:
      - ENV=prod
    networks:
      - mynet
    restart: unless-stopped

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - mynet

volumes:
  web-data:
  db-data:

networks:
  mynet:
    driver: bridge
```

```bash
# Команды
docker-compose up -d          # Запустить
docker-compose down           # Остановить
docker-compose logs -f        # Логи
docker-compose ps             # Статус
docker-compose exec db bash   # Войти в контейнер
```

---
## Официальные источники
[Docker CLI Reference](https://docs.docker.com/reference/cli/?spm=a2ty_o01.29997173.0.0.7ef45171HxIHXu)
[Docker CLI Reference](https://docs.docker.com/reference/dockerfile/?spm=a2ty_o01.29997173.0.0.7ef45171HxIHXu)
[Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/?spm=a2ty_o01.29997173.0.0.7ef45171HxIHXu)
[Storage Driver](https://docs.docker.com/storage/storagedriver/?spm=a2ty_o01.29997173.0.0.7ef45171HxIHXu)
[Use Vol.](https://docs.docker.com/storage/volumes/?spm=a2ty_o01.29997173.0.0.7ef45171HxIHXu)
[Network](https://docs.docker.com/network/?spm=a2ty_o01.29997173.0.0.7ef45171HxIHXu)
[Sec.](https://docs.docker.com/engine/security/?spm=a2ty_o01.29997173.0.0.7ef45171HxIHXu)

## Quick Start: типовой сценарий
```bash
# 1. Собрать образ
docker build -t myapp:1.0 .

# 2. Протестировать
docker run --rm -it myapp:1.0 --version

# 3. Запустить с данными
docker volume create myapp-data
docker run -d \
  --name myapp \
  --mount source=myapp-data,target=/data \
  -p 8080:8080 \
  --restart unless-stopped \
  myapp:1.0

# 4. Проверить
docker logs -f myapp
curl http://localhost:8080/health

# 5. Остановить
docker stop myapp && docker rm myapp
```

>[!NOTE] Полезные советы
>
>- Всегда используй теги (`nginx:1.25`, не `latest`)
>- `.dockerignore` обязателен (исключи `.git`, `node_modules`)
>- Логи пиши в stdout/stderr, не в файлы
>- Для БД всегда используй volumes
>- Регулярно обновляй базовые образы (security patches)


