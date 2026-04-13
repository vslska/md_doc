# Docker BuildKit

**BuildKit** — это современный движок сборки образов Docker (заменил устаревший legacy builder). Он делает сборку быстрее, образы — легче, а процесс — удобнее.

(НЕ ЗАБУДЬ ЕГО УСТАНОВИТЬ) 

---

 ### Почему BuildKit ?

1.  **Параллельность**: Собирает независимые стадии (stages) одновременно, а не строго по очереди.
2.  **Умные монтирования (`--mount`)**: Позволяет использовать файлы (например, `requirements.txt`) или кэш-директории прямо во время сборки без лишних слоев `COPY`.
3.  **Безопасность**: Умеет прокидывать секреты (SSH-ключи, API-токены) так, чтобы они не сохранялись в истории образа.
4.  **Визуализация**: Информативный интерфейс с прогресс-барами и логами по каждому шагу.

---

### `type=cache`

Это главная фишка для ускорения сборки. Мы монтируем папку с кэшем менеджера пакетов из хост-системы внутрь контейнера сборки.

#### Как это работает в Dockerfile:

```dockerfile
# 1. Монтируем requirements.txt (bind), чтобы не делать лишний COPY
# 2. Монтируем папку кэша pip, чтобы не качать либы заново при каждом изменении
RUN --mount=type=bind,source=requirements.txt,target=requirements.txt \
    --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt
```
> **Результат:** Если ты добавил одну библиотеку в список, `pip` скачает только её, а остальные 50 возьмет из кэша BuildKit мгновенно.

### Настройка для Fish Shell

В оболочке **fish** переменные окружения задаются иначе. Чтобы BuildKit работал всегда:

1. **Установить глобально:**
```bash
set -Ux DOCKER_BUILDKIT 1
```

2.  **Запустить разово (если нужно):**
```bash
env DOCKER_BUILDKIT=1 docker build -t my-app .
```

3. **Использовать современную команду (рекомендуется):**
```bash
docker buildx build -t my-app .
```

### Шаблон Dockerfile
```dockerfile
# Стадия сборки (Builder)
FROM python:3.12-slim AS builder

WORKDIR /app

# Установка системных зависимостей с кэшем apt
RUN --mount=type=cache,target=/var/cache/apt \
    apt-get update && apt-get install -y gcc

# Установка Python-пакетов с кэшем pip
RUN --mount=type=bind,source=requirements.txt,target=requirements.txt \
    --mount=type=cache,target=/root/.cache/pip \
    pip install --prefix=/install -r requirements.txt

# Финальная стадия (Runtime) — максимально легкая
FROM python:3.12-slim
COPY --from=builder /install /usr/local
COPY . /app
WORKDIR /app

CMD ["python", "main.py"]
```