# Docker для Backend QA

## Оглавление
- [1. Image и Container](#1-image-и-container)
- [2. Dockerfile](#2-dockerfile)
- [3. Команды QA](#3-команды-qa)
- [4. Сети, порты, volumes](#4-сети-порты-volumes)
- [5. Docker Compose](#5-docker-compose)
- [6. Диагностика](#6-диагностика)

---

## 1. Image и Container

**Image** — шаблон файлов и настроек для запуска приложения, обычно из слоёв. **Container** — экземпляр образа со своим процессом и изолированным окружением.

~~~text
Dockerfile → Build → Image → Run → Container
                             └→ Container №2
~~~

Контейнер **не полноценная VM**: Linux-контейнеры разделяют ядро хоста. Docker Desktop на macOS использует Linux VM для Linux-контейнеров.

## 2. Dockerfile

~~~dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
EXPOSE 3000
CMD ["node", "dist/server.js"]
~~~

**Учебный пример:** dist/server.js должен быть предварительно собран. На практике применяют multi-stage build, non-root пользователя, .dockerignore и фиксацию версий.

| Инструкция | Для чего |
|---|---|
| FROM | Базовый image |
| WORKDIR | Рабочий каталог |
| COPY | Копирование в image |
| RUN | Команда при сборке |
| ENV | Переменная окружения |
| EXPOSE | Документация порта; не публикует его |
| CMD | Процесс запуска контейнера |

**RUN vs CMD:** во время build vs во время run.

## 3. Команды QA

| Задача | Команда |
|---|---|
| Запущенные контейнеры | `docker ps` |
| Все контейнеры | `docker ps -a` |
| Логи | `docker logs --tail 100 bonus-api` |
| Следить за логами | `docker logs -f bonus-api` |
| Выполнить shell | `docker exec -it bonus-api sh` |
| Свойства контейнера | `docker inspect bonus-api` |
| CPU/RAM | `docker stats` |
| Собрать образ | `docker build -t bonus-api:local .` |
| Запустить | `docker run --rm -p 8080:3000 --name bonus-api bonus-api:local` |

**Публикация порта:** `-p 8080:3000` означает host:container. Приложение должно слушать правильный адрес и порт.

## 4. Сети, порты, volumes

**localhost внутри контейнера — сам контейнер.** Для связи с PostgreSQL в общей Docker-сети обычно используют имя сервиса.

~~~text
Host: localhost:8080 → bonus-api:3000
bonus-api → postgres:5432
~~~

**Volume** сохраняет данные отдельно от изменяемого слоя контейнера. При пересоздании контейнера данные volume могут остаться. **Bind mount** подключает путь с хостовой машины.

## 5. Docker Compose

~~~yaml
services:
  postgres:
    image: postgres:17
    environment:
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test_only
      POSTGRES_DB: bonuses
    volumes:
      - pgdata:/var/lib/postgresql/data

  bonus-api:
    build: .
    ports:
      - "8080:3000"
    environment:
      DATABASE_URL: postgres://test:test_only@postgres:5432/bonuses
    depends_on:
      - postgres

volumes:
  pgdata:
~~~

Пароль — **только пример для локального теста**. `depends_on` не гарантирует готовность БД, лишь порядок старта в такой конфигурации. Для готовности добавляют healthcheck и соответствующее условие.

~~~bash
docker compose up -d --build
docker compose ps
docker compose logs --tail 100 bonus-api
docker compose down
~~~

**Осторожно:** `docker compose down -v` удаляет volumes с данными.

## 6. Диагностика

**Ситуация:** API недоступен.

1. `docker compose ps` — контейнер существует, не перезапускается?
2. `docker compose logs bonus-api` — ошибки запуска или БД?
3. `docker inspect bonus-api` — порты, сеть, env, exit code?
4. Проверить, слушает ли API 3000 внутри, а host пробрасывает 8080.
5. Проверить адрес зависимости: `postgres`, не `localhost`.

**Testcontainers** позволяет автоматизированно поднимать изолированные зависимости для интеграционных тестов. Это помогает управлять версиями и тестовыми данными.

**Вопросы:** image vs container? Что делает EXPOSE? Почему localhost не видит соседний сервис? Чем Docker Compose отличается от Dockerfile?

[Далее — Kubernetes](04-kubernetes-basics.md).