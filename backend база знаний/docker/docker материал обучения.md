# 🐳 Docker для Backend разработчика: Структурированный курс

---

## 📖 Содержание курса

**Блок 1: Введение в Docker**
- Глава 1: Проблемы традиционной разработки и решения Docker
- Глава 2: Архитектура Docker и установка

**Блок 2: Основы работы с Docker** 
- Глава 3: Образы и контейнеры
- Глава 4: Управление контейнерами и практические сценарии

**Блок 3: Создание образов**
- Глава 5: Основы Dockerfile
- Глава 6: Продвинутые техники создания образов

**Блок 4: Управление данными и сетями**
- Глава 7: Volumes и управление данными
- Глава 8: Networks и сетевое взаимодействие

**Блок 5: Docker Compose**
- Глава 9: Основы Docker Compose
- Глава 10: Продвинутые возможности Compose

**Блок 6: Production-готовые решения**
- Глава 11: Production конфигурации
- Глава 12: Реальные проекты и CI/CD

**Блок 7: Экспертные техники**
- Глава 13: Оптимизация и безопасность
- Глава 14: Мониторинг и отладка

---

# Блок 1: Введение в Docker

## Глава 1: Проблемы традиционной разработки и решения Docker

### 1.1 Проблемы традиционной разработки

```
🏠 Локальная машина     →     🔧 Тестовый сервер     →     🚀 Продакшн
Python 3.9              →     Python 3.8             →     Python 3.7
Ubuntu 20.04            →     CentOS 7               →     RHEL 8
MySQL 8.0               →     MySQL 5.7              →     PostgreSQL 13
```

**Результат:** "У меня работает!" → "На тесте падает" → "В продакшне не запускается"

### 1.2 Docker решение

```
📦 Контейнер с приложением
├── Точно такая же ОС
├── Точно такие же зависимости  
├── Точно такие же настройки
└── Гарантированно работает везде
```

### 1.3 Сравнение: Виртуальные машины vs Docker

```
🖥️ ВИРТУАЛЬНЫЕ МАШИНЫ                 🐳 DOCKER КОНТЕЙНЕРЫ
┌─────────────────────────┐           ┌─────────────────────────┐
│     Приложение A        │           │     Приложение A        │
├─────────────────────────┤           ├─────────────────────────┤
│       Гостевая ОС       │           │     Docker Engine       │
├─────────────────────────┤           ├─────────────────────────┤
│     Гипервизор          │           │      Хостовая ОС        │
├─────────────────────────┤           └─────────────────────────┘
│      Хостовая ОС        │
└─────────────────────────┘

Размер: ~GB                           Размер: ~MB
Запуск: ~минуты                       Запуск: ~секунды
Ресурсы: Много                        Ресурсы: Мало
```

## Глава 2: Архитектура Docker и установка

### 2.1 Архитектура Docker

```
Docker Client → Docker Daemon → Images/Containers/Networks/Volumes
     ↓               ↓               ↓
docker run     Управление      Docker Hub/Registry
```

**Основные компоненты:**
- **Docker Client** - интерфейс командной строки
- **Docker Daemon** - фоновый процесс управления
- **Docker Images** - шаблоны для контейнеров
- **Docker Containers** - запущенные экземпляры
- **Docker Registry** - хранилище образов

### 2.2 Установка Docker

#### Ubuntu/Debian:
```bash
# Обновление пакетов
sudo apt update

# Установка зависимостей
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release

# Добавление GPG ключа Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Добавление репозитория
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Установка Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io

# Добавление пользователя в группу docker
sudo usermod -aG docker $USER
```

#### macOS:
```bash
# Установка через Homebrew
brew install --cask docker

# Или скачать Docker Desktop с официального сайта
```

#### Windows:
```bash
# Скачать Docker Desktop с официального сайта
# Требует WSL2 для лучшей производительности
```

### 2.3 Проверка установки

```bash
# Проверка версии
docker --version
# Docker version 24.0.5, build ced0996

# Проверка информации о системе
docker info

# Первый запуск
docker run hello-world
```

### 2.4 Жизненный цикл контейнера

```
📥 PULL IMAGE    →    🏗️ CREATE        →    ▶️ START       →    ⏸️ STOP        →    🗑️ REMOVE
docker pull           docker create        docker start       docker stop        docker rm
     ↓                      ↓                   ↓                  ↓                  ↓
Скачивание образа    Создание контейнера   Запуск процесса    Остановка          Удаление
```

---

# Блок 2: Основы работы с Docker

## Глава 3: Образы и контейнеры

### 3.1 Основные команды для образов

```bash
# Поиск образов
docker search nginx
docker search postgres --limit 5

# Скачивание образа
docker pull nginx
docker pull nginx:1.25-alpine
docker pull postgres:15.3

# Просмотр локальных образов
docker images
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Получение детальной информации об образе
docker inspect nginx:latest

# Просмотр истории слоев образа
docker history nginx:latest

# Удаление образов
docker rmi nginx:latest
docker rmi $(docker images -q)  # Удаление всех образов
```

### 3.2 Работа с контейнерами

```bash
# Запуск контейнера (различные способы)
docker run nginx                    # Блокирующий запуск
docker run -d nginx                 # Фоновый запуск
docker run -it ubuntu bash          # Интерактивный режим
docker run --rm nginx               # Автоудаление после остановки
docker run --name my-nginx nginx    # Именованный контейнер

# Проброс портов
docker run -d -p 8080:80 nginx                # Хост:Контейнер
docker run -d -p 127.0.0.1:8080:80 nginx      # Конкретный IP
docker run -d -P nginx                        # Автоматический проброс

# Переменные окружения
docker run -d -e POSTGRES_PASSWORD=secret postgres
docker run -d --env-file .env postgres

# Просмотр контейнеров
docker ps                           # Только запущенные
docker ps -a                        # Все контейнеры
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Управление контейнерами
docker start my-nginx               # Запуск остановленного
docker stop my-nginx                # Корректная остановка
docker kill my-nginx                # Принудительная остановка
docker restart my-nginx             # Перезапуск
docker pause my-nginx               # Приостановка
docker unpause my-nginx             # Возобновление

# Удаление контейнеров
docker rm my-nginx                  # Удаление остановленного
docker rm -f my-nginx               # Принудительное удаление
docker container prune              # Удаление всех остановленных
```

### 3.3 Работа с запущенными контейнерами

```bash
# Выполнение команд в контейнере
docker exec -it my-nginx bash
docker exec my-nginx ls -la /usr/share/nginx/html
docker exec -u root my-nginx apt update

# Просмотр логов
docker logs my-nginx
docker logs -f my-nginx              # Следить за логами
docker logs --tail 50 my-nginx       # Последние 50 строк
docker logs --since 2h my-nginx      # За последние 2 часа

# Копирование файлов
docker cp ./index.html my-nginx:/usr/share/nginx/html/
docker cp my-nginx:/etc/nginx/nginx.conf ./

# Мониторинг ресурсов
docker stats                        # Все контейнеры
docker stats my-nginx               # Конкретный контейнер
```

## Глава 4: Управление контейнерами и практические сценарии

### 4.1 Практический сценарий: Веб-сервер с кастомным контентом

```bash
# Создание директории с контентом
mkdir my-website
echo "<h1>Hello Docker!</h1>" > my-website/index.html

# Запуск nginx с монтированием директории
docker run -d \
  --name my-website \
  -p 8080:80 \
  -v $(pwd)/my-website:/usr/share/nginx/html \
  nginx:alpine

# Проверка
curl http://localhost:8080
```

### 4.2 Практический сценарий: База данных PostgreSQL

```bash
# Запуск PostgreSQL с постоянным хранением
docker run -d \
  --name my-postgres \
  -e POSTGRES_DB=myapp \
  -e POSTGRES_USER=developer \
  -e POSTGRES_PASSWORD=secret123 \
  -p 5432:5432 \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:15-alpine

# Подключение к базе
docker exec -it my-postgres psql -U developer -d myapp

# SQL команды в контейнере
docker exec my-postgres psql -U developer -d myapp -c "CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(100));"
```

### 4.3 Практический сценарий: Redis для кэширования

```bash
# Запуск Redis
docker run -d \
  --name my-redis \
  -p 6379:6379 \
  redis:alpine redis-server --appendonly yes

# Тестирование Redis
docker exec -it my-redis redis-cli
# 127.0.0.1:6379> SET test "Hello Redis"
# 127.0.0.1:6379> GET test
```

### 4.4 Отладка и диагностика

```bash
# Просмотр процессов в контейнере
docker top my-nginx

# Инспекция контейнера
docker inspect my-nginx | jq '.[0].NetworkSettings.IPAddress'

# Просмотр изменений в файловой системе
docker diff my-nginx

# Создание образа из контейнера
docker commit my-nginx my-custom-nginx:1.0

# Экспорт/импорт контейнеров
docker export my-nginx > my-nginx.tar
docker import my-nginx.tar my-nginx:backup
```

### 4.5 Многоконтейнерное приложение

```bash
# Создание сети
docker network create myapp-network

# База данных
docker run -d \
  --name db \
  --network myapp-network \
  -e POSTGRES_DB=api \
  -e POSTGRES_USER=api \
  -e POSTGRES_PASSWORD=secret \
  postgres:15-alpine

# Backend
docker run -d \
  --name api \
  --network myapp-network \
  -e DATABASE_URL=postgresql://api:secret@db:5432/api \
  my-api:latest

# Frontend
docker run -d \
  --name frontend \
  --network myapp-network \
  -p 80:80 \
  nginx:alpine
```

---

# Блок 3: Создание образов

## Глава 5: Основы Dockerfile

### 5.1 Анатомия Dockerfile

```dockerfile
# Комментарий: выбор базового образа
FROM node:18-alpine

# Метаданные
LABEL maintainer="developer@company.com"
LABEL version="1.0"
LABEL description="My awesome API"

# Переменные окружения
ENV NODE_ENV=production
ENV PORT=3000

# Установка системных пакетов
RUN apk add --no-cache \
    curl \
    wget \
    && rm -rf /var/cache/apk/*

# Создание пользователя
RUN addgroup -g 1001 -S nodejs \
    && adduser -S nextjs -u 1001

# Рабочая директория
WORKDIR /app

# Копирование файлов зависимостей
COPY package*.json ./

# Установка зависимостей
RUN npm ci --only=production \
    && npm cache clean --force

# Копирование исходного кода
COPY . .

# Изменение владельца файлов
RUN chown -R nextjs:nodejs /app
USER nextjs

# Открытие порта
EXPOSE 3000

# Проверка здоровья
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Команда запуска
CMD ["npm", "start"]
```

### 5.2 Выбор базового образа

```dockerfile
# ❌ Плохо: слишком большой
FROM ubuntu:20.04

# ✅ Лучше: специализированный
FROM node:18

# ✅ Отлично: минимальный
FROM node:18-alpine

# 🎯 Идеально: distroless для production
FROM gcr.io/distroless/nodejs18-debian11
```

**Сравнение размеров:**
```
ubuntu:20.04     → ~72MB
node:18          → ~993MB  
node:18-alpine   → ~172MB
distroless       → ~58MB
```

### 5.3 Инструкции Dockerfile

#### COPY vs ADD:
```dockerfile
# COPY - простое копирование (рекомендуется)
COPY src/ /app/src/
COPY package*.json ./

# ADD - копирование с дополнительными возможностями
ADD https://example.com/file.tar.gz /tmp/  # Скачивание URL
ADD archive.tar.gz /app/                   # Автораспаковка архивов
```

#### RUN оптимизация:
```dockerfile
# ❌ Плохо: много слоев
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN apt-get clean

# ✅ Хорошо: один слой
RUN apt-get update \
    && apt-get install -y \
        curl \
        wget \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
```

#### CMD vs ENTRYPOINT:
```dockerfile
# CMD - может быть переопределена
CMD ["npm", "start"]
# docker run myapp npm test  ← переопределит CMD

# ENTRYPOINT - всегда выполняется
ENTRYPOINT ["npm"]
CMD ["start"]
# docker run myapp test  ← выполнит "npm test"

# Комбинированный подход
ENTRYPOINT ["./docker-entrypoint.sh"]
CMD ["npm", "start"]
```

### 5.4 .dockerignore для оптимизации

```dockerignore
# Документация
*.md
docs/

# Зависимости (устанавливаются через package.json)
node_modules/
npm-debug.log*

# Системные файлы
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
*.swo

# Тесты (если не нужны в образе)
**/*.test.js
**/*.spec.js
coverage/

# CI/CD
.github/
.gitlab-ci.yml
Jenkinsfile

# Docker
Dockerfile*
docker-compose*.yml

# Environment
.env.local
.env.development
.env.test

# Логи
logs/
*.log

# Временные файлы
tmp/
temp/
*.tmp
```

## Глава 6: Продвинутые техники создания образов

### 6.1 Многоэтапная сборка (Multi-stage builds)

```dockerfile
# ==========================================
# Этап 1: Установка зависимостей
# ==========================================
FROM node:18-alpine AS dependencies

WORKDIR /app
COPY package*.json ./
RUN npm ci --include=dev

# ==========================================
# Этап 2: Сборка приложения
# ==========================================
FROM dependencies AS builder

COPY . .
RUN npm run build \
    && npm run test \
    && npm prune --production

# ==========================================
# Этап 3: Продакшн образ
# ==========================================
FROM node:18-alpine AS production

# Создание пользователя
RUN addgroup -g 1001 -S nodejs \
    && adduser -S nextjs -u 1001

WORKDIR /app

# Копирование только необходимого
COPY --from=builder --chown=nextjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nextjs:nodejs /app/package.json ./package.json

USER nextjs

EXPOSE 3000

CMD ["npm", "start"]
```

### 6.2 Оптимизация кэширования слоев

```dockerfile
# ❌ Плохо: кэш инвалидируется при изменении любого файла
FROM node:18-alpine
COPY . .
RUN npm install

# ✅ Хорошо: кэш зависимостей отдельно от кода
FROM node:18-alpine
WORKDIR /app

# Слой 1: package.json (меняется редко)
COPY package*.json ./
RUN npm ci --only=production

# Слой 2: исходный код (меняется часто)
COPY . .

# Слой 3: сборка (только если изменился код)
RUN npm run build
```

### 6.3 ARG и ENV переменные

```dockerfile
# ARG - переменные времени сборки
ARG NODE_VERSION=18
ARG BUILD_DATE
ARG VERSION

FROM node:${NODE_VERSION}-alpine

# ENV - переменные времени выполнения
ENV NODE_ENV=production
ENV PORT=3000
ENV BUILD_DATE=${BUILD_DATE}
ENV VERSION=${VERSION}

# Переменные по умолчанию
ENV DATABASE_URL=""
ENV REDIS_URL=""

# Использование переменных
RUN echo "Building version ${VERSION} on ${BUILD_DATE}"

LABEL version=$VERSION
LABEL build-date=$BUILD_DATE

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE $PORT

CMD ["npm", "start"]
```

Сборка с аргументами:
```bash
docker build \
  --build-arg VERSION=1.2.3 \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  -t myapp:1.2.3 .
```

### 6.4 Специализированные примеры

#### Dockerfile для Python приложения

```dockerfile
FROM python:3.11-slim

# Системные зависимости
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        build-essential \
        curl \
        libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Python оптимизации
ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1
ENV PIP_NO_CACHE_DIR=1
ENV PIP_DISABLE_PIP_VERSION_CHECK=1

# Создание пользователя
RUN useradd --create-home --shell /bin/bash app

WORKDIR /app

# Установка зависимостей
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Копирование кода
COPY --chown=app:app . .

USER app

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

#### Dockerfile для Go приложения

```dockerfile
# ==========================================
# Этап сборки
# ==========================================
FROM golang:1.21-alpine AS builder

# Установка git для модулей
RUN apk add --no-cache git

WORKDIR /app

# Копирование go.mod и go.sum для кэширования зависимостей
COPY go.mod go.sum ./
RUN go mod download

# Копирование исходного кода
COPY . .

# Сборка статического бинарника
RUN CGO_ENABLED=0 GOOS=linux go build \
    -ldflags='-w -s -extldflags "-static"' \
    -a -installsuffix cgo \
    -o main .

# ==========================================
# Продакшн образ
# ==========================================
FROM scratch

# Копирование CA сертификатов для HTTPS
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Копирование бинарника
COPY --from=builder /app/main /main

EXPOSE 8080

USER 1000

ENTRYPOINT ["/main"]
```

---

# Блок 4: Управление данными и сетями

## Глава 7: Volumes и управление данными

### 7.1 Проблема временности контейнеров

```
🐳 Контейнер запускается
📝 Приложение создает данные
💾 Данные сохраняются в контейнере
🛑 Контейнер останавливается
🗑️ Контейнер удаляется
💔 Данные теряются НАВСЕГДА!
```

### 7.2 Типы хранения данных

**Volumes (Рекомендуется):**
- Управляются Docker
- Лучшая производительность
- Переносимость между контейнерами

**Bind Mounts:**
- Прямое монтирование с хоста
- Зависимость от структуры хоста
- Подходит для разработки

**tmpfs mounts:**
- Хранение в памяти
- Временные данные
- Исчезают при остановке

### 7.3 Named Volumes

```bash
# Создание volume
docker volume create mydata
docker volume create postgres_data
docker volume create --driver local --opt type=tmpfs --opt device=tmpfs mytemp

# Просмотр volumes
docker volume ls
docker volume inspect mydata

# Использование в контейнерах
docker run -d \
  --name postgres \
  -v postgres_data:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# Резервное копирование данных
docker run --rm \
  -v postgres_data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/postgres_backup.tar.gz -C /data .

# Восстановление данных
docker run --rm \
  -v postgres_data:/data \
  -v $(pwd):/backup \
  alpine sh -c "cd /data && tar xzf /backup/postgres_backup.tar.gz --strip 1"

# Удаление volumes
docker volume rm mydata
docker volume prune  # Удаление неиспользуемых volumes
```

### 7.4 Bind Mounts

```bash
# Абсолютный путь (Linux/macOS)
docker run -d \
  --name nginx \
  -v /home/user/website:/usr/share/nginx/html \
  nginx

# Относительный путь
docker run -d \
  --name nginx \
  -v $(pwd)/website:/usr/share/nginx/html \
  nginx

# Windows (PowerShell)
docker run -d \
  --name nginx \
  -v ${PWD}/website:/usr/share/nginx/html \
  nginx

# Только для чтения
docker run -d \
  --name nginx \
  -v $(pwd)/website:/usr/share/nginx/html:ro \
  nginx
```

### 7.5 Практические сценарии с volumes

#### База данных с постоянным хранением

```bash
# Создание специального volume для PostgreSQL
docker volume create pgdata

# Запуск PostgreSQL с volume
docker run -d \
  --name postgres \
  -e POSTGRES_DB=myapp \
  -e POSTGRES_USER=developer \
  -e POSTGRES_PASSWORD=secret123 \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15-alpine

# Создание таблицы и данных
docker exec -it postgres psql -U developer -d myapp -c "
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);
INSERT INTO users (name, email) VALUES 
('John Doe', 'john@example.com'),
('Jane Smith', 'jane@example.com');
"

# Остановка и удаление контейнера
docker stop postgres
docker rm postgres

# Запуск нового контейнера с тем же volume
docker run -d \
  --name postgres-new \
  -e POSTGRES_DB=myapp \
  -e POSTGRES_USER=developer \
  -e POSTGRES_PASSWORD=secret123 \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15-alpine

# Проверка сохранности данных
docker exec -it postgres-new psql -U developer -d myapp -c "SELECT * FROM users;"
```

## Глава 8: Networks и сетевое взаимодействие

### 8.1 Типы сетей Docker

**bridge (по умолчанию):**
- Изолированная сеть
- NAT к хосту
- Используется по умолчанию

**host:**
- Использует сеть хоста
- Нет изоляции
- Максимальная производительность

**none:**
- Без сети
- Полная изоляция

**overlay:**
- Многохостовые сети
- Docker Swarm

### 8.2 Работа с сетями

```bash
# Просмотр сетей
docker network ls

# Детальная информация о сети
docker network inspect bridge

# Создание пользовательской сети
docker network create myapp-network
docker network create --driver bridge --subnet=172.20.0.0/16 custom-network

# Подключение контейнера к сети при запуске
docker run -d --name web --network myapp-network nginx

# Подключение существующего контейнера к сети
docker network connect myapp-network existing-container

# Отключение от сети
docker network disconnect myapp-network existing-container

# Удаление сети
docker network rm myapp-network
docker network prune  # Удаление неиспользуемых сетей
```

### 8.3 Связка Frontend + Backend + Database

```bash
# Создание сети
docker network create fullstack-network

# База данных
docker run -d \
  --name postgres \
  --network fullstack-network \
  -e POSTGRES_DB=myapp \
  -e POSTGRES_USER=api \
  -e POSTGRES_PASSWORD=secret \
  postgres:15-alpine

# Backend API
docker run -d \
  --name api \
  --network fullstack-network \
  -e DATABASE_URL=postgresql://api:secret@postgres:5432/myapp \
  myapi:latest

# Frontend (с проброшенным портом для доступа извне)
docker run -d \
  --name frontend \
  --network fullstack-network \
  -p 80:80 \
  -e API_URL=http://api:3000 \
  myfrontend:latest
```

### 8.4 Service Discovery и DNS

```bash
# Проверка DNS резолюции между контейнерами
docker exec -it frontend nslookup api

# Проверка подключения
docker exec -it frontend ping api
docker exec -it frontend curl http://api/health

# Просмотр сетевых интерфейсов контейнера
docker exec -it frontend ip addr show

# Просмотр таблицы маршрутизации
docker exec -it frontend ip route
```

### 8.5 Сетевые политики безопасности

```bash
# Создание изолированных сетей
docker network create --internal database-network
docker network create --internal cache-network

# Контейнер с доступом только к внутренним сетям
docker run -d \
  --name secure-api \
  --network database-network \
  --network cache-network \
  secure-api:latest

# Проверка изоляции (не должно работать)
docker exec -it secure-api curl https://google.com
```

---

# Блок 5: Docker Compose

## Глава 9: Основы Docker Compose

### 9.1 Проблемы управления множественными контейнерами

```bash
# Без Compose: много команд для одного приложения
docker network create myapp-network
docker volume create postgres_data
docker volume create redis_data

docker run -d --name postgres --network myapp-network \
  -v postgres_data:/var/lib/postgresql/data \
  -e POSTGRES_DB=myapp -e POSTGRES_USER=api -e POSTGRES_PASSWORD=secret \
  postgres:15-alpine

docker run -d --name redis --network myapp-network \
  -v redis_data:/data \
  redis:alpine

docker run -d --name api --network myapp-network \
  -e DATABASE_URL=postgresql://api:secret@postgres:5432/myapp \
  -e REDIS_URL=redis://redis:6379 \
  myapi:latest

docker run -d --name frontend --network myapp-network \
  -p 80:80 -e API_URL=http://api:3000 \
  myfrontend:latest
```

### 9.2 Docker Compose решение

```yaml
# docker-compose.yml - одна команда для всего
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: api
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:alpine
    volumes:
      - redis_data:/data

  api:
    image: myapi:latest
    environment:
      DATABASE_URL: postgresql://api:secret@postgres:5432/myapp
      REDIS_URL: redis://redis:6379
    depends_on:
      - postgres
      - redis

  frontend:
    image: myfrontend:latest
    ports:
      - "80:80"
    environment:
      API_URL: http://api:3000
    depends_on:
      - api

volumes:
  postgres_data:
  redis_data:

networks:
  default:
    driver: bridge
```

Запуск одной командой:
```bash
docker-compose up -d
```

### 9.3 Основные команды Docker Compose

```bash
# Запуск сервисов
docker-compose up                    # Запуск в foreground
docker-compose up -d                 # Запуск в background
docker-compose up --build            # Пересборка образов перед запуском
docker-compose up --scale api=3      # Масштабирование сервиса

# Остановка и удаление
docker-compose stop                  # Остановка сервисов
docker-compose down                  # Остановка и удаление контейнеров
docker-compose down -v               # + удаление volumes
docker-compose down --rmi all        # + удаление образов

# Управление сервисами
docker-compose start api             # Запуск остановленного сервиса
docker-compose stop api              # Остановка сервиса
docker-compose restart api           # Перезапуск сервиса
docker-compose pause api             # Приостановка сервиса
docker-compose unpause api           # Возобновление сервиса

# Просмотр информации
docker-compose ps                    # Статус сервисов
docker-compose logs                  # Логи всех сервисов
docker-compose logs -f api           # Следить за логами сервиса
docker-compose top                   # Процессы в контейнерах
docker-compose images               # Образы сервисов

# Выполнение команд
docker-compose exec api bash         # Интерактивная сессия
docker-compose exec api npm test     # Выполнение команды
docker-compose run --rm api npm install  # Разовое выполнение

# Валидация и отладка
docker-compose config               # Проверка синтаксиса
docker-compose config --services    # Список сервисов
docker-compose port api 3000        # Проброшенный порт
```

### 9.4 Полноценное веб-приложение

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - frontend
      - api
    networks:
      - web

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    environment:
      - REACT_APP_API_URL=http://localhost/api
    networks:
      - web

  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://api:secret@postgres:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - web
      - backend

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: api
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U api -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:alpine
    volumes:
      - redis_data:/data
    networks:
      - backend

volumes:
  postgres_data:
  redis_data:

networks:
  web:
    driver: bridge
  backend:
    driver: bridge
    internal: true
```

## Глава 10: Продвинутые возможности Compose

### 10.1 Множественные окружения

#### Базовая конфигурация (docker-compose.yml):
```yaml
version: '3.8'

x-common-variables: &common-variables
  NODE_ENV: ${NODE_ENV:-development}
  DATABASE_URL: postgresql://${DB_USER}:${DB_PASSWORD}@postgres:${DB_PORT:-5432}/${DB_NAME}
  REDIS_URL: redis://redis:6379

x-logging: &default-logging
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
      target: ${BUILD_TARGET:-development}
    environment:
      <<: *common-variables
    networks:
      - backend
    logging: *default-logging

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - backend
    logging: *default-logging

  redis:
    image: redis:alpine
    volumes:
      - redis_data:/data
    networks:
      - backend
    logging: *default-logging

volumes:
  postgres_data:
  redis_data:

networks:
  backend:
    driver: bridge
```

#### Development окружение (docker-compose.dev.yml):
```yaml
version: '3.8'

services:
  api:
    build:
      target: development
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "${API_PORT:-3000}:3000"
    environment:
      - NODE_ENV=development
      - DEBUG=api:*
      - NODEMON_WATCH=true
    command: ["npm", "run", "dev"]

  postgres:
    ports:
      - "${DB_PORT:-5432}:5432"

  redis:
    ports:
      - "6379:6379"

  mailhog:
    image: mailhog/mailhog
    ports:
      - "1025:1025"  # SMTP
      - "8025:8025"  # Web UI
    networks:
      - backend

  adminer:
    image: adminer
    ports:
      - "8080:8080"
    networks:
      - backend
    environment:
      ADMINER_DEFAULT_SERVER: postgres
```

#### Production окружение (docker-compose.prod.yml):
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - static_files:/usr/share/nginx/html:ro
    restart: unless-stopped
    networks:
      - frontend
    depends_on:
      - api

  api:
    build:
      target: production
    restart: unless-stopped
    environment:
      - NODE_ENV=production
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s
    deploy:
      replicas: 2
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
    networks:
      - frontend
      - backend

  postgres:
    restart: unless-stopped
    command: postgres -c max_connections=200 -c shared_buffers=256MB
    deploy:
      resources:
        limits:
          memory: 1G
          cpus: '1.0'
    networks:
      - backend

  redis:
    restart: unless-stopped
    command: redis-server --maxmemory 256mb --maxmemory-policy allkeys-lru
    networks:
      - backend

volumes:
  static_files:

networks:
  frontend:
    driver: bridge
```

### 10.2 Запуск разных окружений

```bash
# Development
docker-compose -f docker-compose.yml -f docker-compose.dev.yml --env-file .env.dev up -d

# Test
docker-compose -f docker-compose.yml -f docker-compose.test.yml --env-file .env.test up --abort-on-container-exit

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml --env-file .env.prod up -d
```

### 10.3 Profiles для условного запуска

```yaml
version: '3.8'

services:
  api:
    build: .
    # Всегда запускается

  postgres:
    image: postgres:15-alpine
    # Всегда запускается

  redis:
    image: redis:alpine
    profiles: ["cache"]  # Только с профилем cache

  monitoring:
    image: prom/prometheus
    profiles: ["monitoring"]  # Только с профилем monitoring

  debug-tools:
    image: nicolaka/netshoot
    profiles: ["debug"]  # Только с профилем debug
    command: ["sleep", "infinity"]
```

Запуск с профилями:
```bash
# Базовые сервисы
docker-compose up -d

# С кэшем
docker-compose --profile cache up -d

# Полная конфигурация
docker-compose --profile cache --profile monitoring --profile debug up -d
```

### 10.4 Масштабирование сервисов

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - api

  api:
    build: .
    environment:
      - NODE_ENV=production
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
```

Команды масштабирования:
```bash
# Запуск с определенным количеством реплик
docker-compose up -d --scale api=5

# Изменение количества реплик на лету
docker-compose up -d --scale api=3 --no-recreate
```

---

# Блок 6: Production-готовые решения

## Глава 11: Production конфигурации

### 11.1 Полный production stack

```yaml
version: '3.8'

x-restart-policy: &restart-policy
  restart: unless-stopped

x-healthcheck-defaults: &healthcheck-defaults
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 60s

x-logging: &logging
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "5"

services:
  traefik:
    image: traefik:v3.0
    <<: *restart-policy
    command:
      - "--api.dashboard=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.letsencrypt.acme.tlschallenge=true"
      - "--certificatesresolvers.letsencrypt.acme.email=admin@example.com"
      - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"  # Dashboard
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - traefik_certs:/letsencrypt
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.rule=Host(`traefik.example.com`)"
      - "traefik.http.routers.traefik.tls.certresolver=letsencrypt"
    logging: *logging

  api:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    <<: *restart-policy
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=${JWT_SECRET}
    networks:
      - traefik
      - backend
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.api.rule=Host(`api.example.com`)"
      - "traefik.http.routers.api.tls.certresolver=letsencrypt"
      - "traefik.http.services.api.loadbalancer.server.port=3000"
    healthcheck:
      <<: *healthcheck-defaults
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
        reservations:
          memory: 256M
          cpus: '0.25'
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
        order: start-first
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
    logging: *logging
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy

  postgres:
    image: postgres:15-alpine
    <<: *restart-policy
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./backups:/backups
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER} -d ${DB_NAME}"]
      interval: 10s
      timeout: 5s
      retries: 5
    command: |
      postgres
      -c max_connections=200
      -c shared_buffers=256MB
      -c effective_cache_size=1GB
      -c maintenance_work_mem=64MB
      -c checkpoint_completion_target=0.9
      -c wal_buffers=16MB
      -c default_statistics_target=100
    deploy:
      resources:
        limits:
          memory: 1G
          cpus: '1.0'
    logging: *logging

  redis:
    image: redis:alpine
    <<: *restart-policy
    command: redis-server --maxmemory 256mb --maxmemory-policy allkeys-lru --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
    deploy:
      resources:
        limits:
          memory: 300M
          cpus: '0.3'
    logging: *logging

  worker:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    <<: *restart-policy
    command: ["npm", "run", "worker"]
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=redis://redis:6379
    networks:
      - backend
    deploy:
      replicas: 2
      resources:
        limits:
          memory: 256M
          cpus: '0.3'
    logging: *logging
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy

  backup:
    image: postgres:15-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data:ro
      - ./backups:/backups
    environment:
      PGPASSWORD: ${DB_PASSWORD}
    command: |
      sh -c '
      while true; do
        sleep 86400  # 24 hours
        pg_dump -h postgres -U ${DB_USER} ${DB_NAME} | gzip > /backups/backup_$(date +%Y%m%d_%H%M%S).sql.gz
        find /backups -name "backup_*.sql.gz" -mtime +7 -delete
      done'
    networks:
      - backend
    depends_on:
      postgres:
        condition: service_healthy

volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local
  traefik_certs:
    driver: local

networks:
  traefik:
    external: true
  backend:
    driver: bridge
    internal: true
```

### 11.2 Мониторинг и логирование

#### Prometheus конфигурация (monitoring/prometheus.yml):
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'api'
    static_configs:
      - targets: ['api:3000']
    metrics_path: '/metrics'

  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']

  - job_name: 'redis'
    static_configs:
      - targets: ['redis-exporter:9121']

  - job_name: 'traefik'
    static_configs:
      - targets: ['traefik:8080']

rule_files:
  - "alert_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093
```

#### Добавление мониторинга в docker-compose:
```yaml
  prometheus:
    image: prom/prometheus
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--storage.tsdb.retention.time=200h'
      - '--web.enable-lifecycle'
    networks:
      - monitoring

  grafana:
    image: grafana/grafana
    volumes:
      - grafana_data:/var/lib/grafana
      - ./monitoring/grafana/provisioning:/etc/grafana/provisioning
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    ports:
      - "3000:3000"
    networks:
      - monitoring
```

## Глава 12: Реальные проекты и CI/CD

### 12.1 E-commerce платформа

```yaml
version: '3.8'

services:
  # API Gateway
  kong:
    image: kong:latest
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_USER: kong
      KONG_PG_PASSWORD: kong
    depends_on:
      - kong-database
    ports:
      - "8000:8000"  # API
      - "8001:8001"  # Admin API
    networks:
      - kong-net

  # Микросервисы
  user-service:
    build: ./services/user-service
    environment:
      - DATABASE_URL=postgresql://users:password@user-db:5432/users
      - JWT_SECRET=${JWT_SECRET}
    networks:
      - kong-net
      - user-net

  product-service:
    build: ./services/product-service
    environment:
      - DATABASE_URL=postgresql://products:password@product-db:5432/products
      - ELASTICSEARCH_URL=http://elasticsearch:9200
    networks:
      - kong-net
      - product-net

  order-service:
    build: ./services/order-service
    environment:
      - DATABASE_URL=postgresql://orders:password@order-db:5432/orders
      - REDIS_URL=redis://redis:6379
      - RABBITMQ_URL=amqp://rabbitmq:5672
    networks:
      - kong-net
      - order-net

  # Базы данных
  user-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: users
      POSTGRES_USER: users
      POSTGRES_PASSWORD: password
    volumes:
      - user_data:/var/lib/postgresql/data
    networks:
      - user-net

  product-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: products
      POSTGRES_USER: products
      POSTGRES_PASSWORD: password
    volumes:
      - product_data:/var/lib/postgresql/data
    networks:
      - product-net

  # Поиск
  elasticsearch:
    image: elasticsearch:8.8.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    networks:
      - product-net

  # Очереди
  rabbitmq:
    image: rabbitmq:3.11-management
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin
    ports:
      - "15672:15672"  # Management UI
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks:
      - order-net

volumes:
  user_data:
  product_data:
  elasticsearch_data:
  rabbitmq_data:

networks:
  kong-net:
  user-net:
    internal: true
  product-net:
    internal: true
  order-net:
    internal: true
```

### 12.2 CI/CD интеграция

#### GitHub Actions workflow (.github/workflows/deploy.yml):
```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run tests
        run: |
          docker-compose -f docker-compose.yml -f docker-compose.test.yml run --rm api npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build and push Docker image
        env:
          DOCKER_BUILDKIT: 1
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker build --target production -t myapp:${{ github.sha }} .
          docker tag myapp:${{ github.sha }} myapp:latest
          docker push myapp:${{ github.sha }}
          docker push myapp:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to server
        uses: appleboy/ssh-action@v0.1.5
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /app
            docker-compose pull
            docker-compose up -d --no-deps api worker
            docker system prune -f
```

### 12.3 Скрипты управления

#### deploy.sh:
```bash
#!/bin/bash

set -e

ENV=${1:-production}
ACTION=${2:-deploy}

case $ENV in
  dev|development)
    COMPOSE_FILES="-f docker-compose.yml -f docker-compose.dev.yml"
    ENV_FILE=".env.dev"
    ;;
  test|testing)
    COMPOSE_FILES="-f docker-compose.yml -f docker-compose.test.yml"
    ENV_FILE=".env.test"
    ;;
  staging)
    COMPOSE_FILES="-f docker-compose.yml -f docker-compose.staging.yml"
    ENV_FILE=".env.staging"
    ;;
  prod|production)
    COMPOSE_FILES="-f docker-compose.yml -f docker-compose.prod.yml"
    ENV_FILE=".env.prod"
    ;;
  *)
    echo "Unknown environment: $ENV"
    exit 1
    ;;
esac

echo "🚀 Deploying to $ENV environment..."

case $ACTION in
  deploy)
    echo "📦 Pulling latest images..."
    docker-compose $COMPOSE_FILES --env-file $ENV_FILE pull

    echo "🔧 Building services..."
    docker-compose $COMPOSE_FILES --env-file $ENV_FILE build

    echo "🏃 Starting services..."
    docker-compose $COMPOSE_FILES --env-file $ENV_FILE up -d

    echo "🏥 Checking health..."
    sleep 30
    docker-compose $COMPOSE_FILES --env-file $ENV_FILE ps
    ;;

  rollback)
    echo "⏪ Rolling back..."
    docker-compose $COMPOSE_FILES --env-file $ENV_FILE down
    ;;

  logs)
    docker-compose $COMPOSE_FILES --env-file $ENV_FILE logs -f
    ;;

  status)
    docker-compose $COMPOSE_FILES --env-file $ENV_FILE ps
    ;;

  *)
    echo "Unknown action: $ACTION"
    echo "Usage: $0 <env> <action>"
    echo "Environments: dev, test, staging, prod"
    echo "Actions: deploy, rollback, logs, status"
    exit 1
    ;;
esac

echo "✅ $ACTION completed for $ENV environment"
```

#### backup.sh:
```bash
#!/bin/bash

set -e

BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)

# Database backup
echo "📊 Backing up database..."
docker-compose exec -T postgres pg_dump -U $DB_USER $DB_NAME | gzip > "$BACKUP_DIR/db_backup_$DATE.sql.gz"

# Redis backup
echo "💾 Backing up Redis..."
docker-compose exec redis redis-cli --rdb /data/dump.rdb
docker cp "$(docker-compose ps -q redis):/data/dump.rdb" "$BACKUP_DIR/redis_backup_$DATE.rdb"

# Application data
echo "📁 Backing up volumes..."
docker run --rm \
  -v myapp_postgres_data:/data/postgres \
  -v myapp_redis_data:/data/redis \
  -v $BACKUP_DIR:/backup \
  alpine tar czf "/backup/volumes_backup_$DATE.tar.gz" -C /data .

# Cleanup old backups (keep 7 days)
find $BACKUP_DIR -name "*_backup_*.gz" -mtime +7 -delete
find $BACKUP_DIR -name "*_backup_*.rdb" -mtime +7 -delete

echo "✅ Backup completed: $DATE"
```

---

# Блок 7: Экспертные техники

## Глава 13: Оптимизация и безопасность

### 13.1 BuildKit и кэширование

```dockerfile
# syntax=docker/dockerfile:1

FROM node:18-alpine AS base
WORKDIR /app

# Кэширование mount для npm
FROM base AS deps
RUN --mount=type=cache,target=/root/.npm \
    npm ci --only=production

FROM base AS build
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci
COPY . .
RUN npm run build

FROM base AS runtime
COPY --from=deps /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY package*.json ./
CMD ["npm", "start"]
```

Сборка с BuildKit:
```bash
DOCKER_BUILDKIT=1 docker build --target runtime .
```

### 13.2 Distroless образы

```dockerfile
# Multi-stage для минимального размера
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o app

FROM gcr.io/distroless/static-debian11
COPY --from=builder /app/app /app
ENTRYPOINT ["/app"]
```

### 13.3 Сканирование уязвимостей

```bash
# Сканирование образа
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image myapp:latest

# Сканирование в CI/CD
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image --exit-code 1 --severity HIGH,CRITICAL myapp:latest
```

### 13.4 Подписывание образов

```bash
# Docker Content Trust
export DOCKER_CONTENT_TRUST=1
docker push myapp:latest

# Notary подпись
notary -s https://notary.docker.io -d ~/.docker/trust init myapp
notary -s https://notary.docker.io -d ~/.docker/trust publish myapp:latest
```

### 13.5 Секреты и безопасность

```yaml
version: '3.8'

services:
  api:
    build: .
    environment:
      - DATABASE_URL_FILE=/run/secrets/db_url
      - JWT_SECRET_FILE=/run/secrets/jwt_secret
    secrets:
      - db_url
      - jwt_secret
    user: "1001:1001"  # Непривилегированный пользователь

  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    secrets:
      - db_password

secrets:
  db_url:
    file: ./secrets/db_url.txt
  jwt_secret:
    file: ./secrets/jwt_secret.txt
  db_password:
    file: ./secrets/db_password.txt
```

### 13.6 Оптимизация размера образов

#### Лучшие практики:
```dockerfile
# 1. Использование alpine базовых образов
FROM node:18-alpine

# 2. Минимизация слоев
RUN apk add --no-cache curl wget \
    && npm ci --only=production \
    && npm cache clean --force \
    && rm -rf /tmp/*

# 3. Удаление ненужных файлов
RUN apt-get update \
    && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

# 4. Использование .dockerignore
# См. предыдущие примеры .dockerignore

# 5. Multi-stage builds для разделения build и runtime
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
CMD ["npm", "start"]
```

## Глава 14: Мониторинг и отладка

### 14.1 Отладочные контейнеры

```yaml
# docker-compose.debug.yml
version: '3.8'

services:
  debug:
    image: nicolaka/netshoot
    command: ["sleep", "infinity"]
    network_mode: "container:api"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    cap_add:
      - NET_ADMIN
      - SYS_PTRACE

  debug-standalone:
    image: nicolaka/netshoot
    command: ["sleep", "infinity"]
    networks:
      - backend
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    cap_add:
      - NET_ADMIN
      - SYS_PTRACE
```

Использование отладочного контейнера:
```bash
# Запуск отладочного контейнера
docker-compose -f docker-compose.yml -f docker-compose.debug.yml up -d debug

# Подключение к отладочному контейнеру
docker-compose exec debug bash

# Внутри отладочного контейнера доступны инструменты:
# - tcpdump для анализа трафика
# - nmap для сканирования портов
# - curl/wget для HTTP запросов
# - netstat для анализа соединений
# - iftop для мониторинга сети
```

### 14.2 Анализ производительности

```bash
# Анализ производительности контейнера
docker run --rm -it --pid container:myapp --cap-add SYS_PTRACE \
  brendangregg/perf-tools

# Профилирование с помощью htop
docker run --rm -it --pid container:myapp \
  terencehill/htop

# Мониторинг ресурсов
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}\t{{.BlockIO}}"

# Анализ использования дискового пространства
docker exec myapp du -sh /*
docker exec myapp df -h

# Анализ процессов в контейнере
docker exec myapp ps aux
docker top myapp
```

### 14.3 Логирование и мониторинг

#### Централизованное логирование с ELK Stack:
```yaml
version: '3.8'

services:
  app:
    build: .
    logging:
      driver: "fluentd"
      options:
        fluentd-address: fluentd:24224
        tag: "app.{{.Name}}"

  fluentd:
    image: fluentd:v1.16-1
    volumes:
      - ./fluentd/conf:/fluentd/etc
      - ./logs:/var/log/fluentd
    ports:
      - "24224:24224"
    depends_on:
      - elasticsearch

  elasticsearch:
    image: elasticsearch:8.8.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"

  kibana:
    image: kibana:8.8.0
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

volumes:
  elasticsearch_data:
```

#### Fluentd конфигурация (fluentd/conf/fluent.conf):
```xml
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

<match app.**>
  @type elasticsearch
  host elasticsearch
  port 9200
  index_name docker-logs
  type_name _doc
  <buffer>
    flush_interval 10s
  </buffer>
</match>
```

### 14.4 Мониторинг производительности

#### Prometheus + Grafana для мониторинга:
```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--storage.tsdb.retention.time=200h'
      - '--web.enable-lifecycle'

  grafana:
    image: grafana/grafana
    volumes:
      - grafana_data:/var/lib/grafana
      - ./monitoring/grafana/provisioning:/etc/grafana/provisioning
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
    ports:
      - "3000:3000"

  node-exporter:
    image: prom/node-exporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($|/)'
    ports:
      - "9100:9100"

  cadvisor:
    image: gcr.io/cadvisor/cadvisor
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8080:8080"

volumes:
  prometheus_data:
  grafana_data:
```

### 14.5 Отладка сетевых проблем

```bash
# Анализ сетевых соединений
docker exec myapp netstat -tlnp
docker exec myapp ss -tlnp

# Трассировка сетевого трафика
docker exec myapp tcpdump -i eth0 -n

# Тестирование подключения к другим сервисам
docker exec myapp ping postgres
docker exec myapp telnet postgres 5432
docker exec myapp curl -v http://api:3000/health

# Анализ DNS
docker exec myapp nslookup postgres
docker exec myapp dig postgres

# Просмотр сетевых интерфейсов
docker exec myapp ip addr show
docker exec myapp ip route show

# Анализ с помощью netshoot
docker run --rm -it --net container:myapp nicolaka/netshoot
```

### 14.6 Отладка проблем с volumes

```bash
# Просмотр содержимого volume
docker run --rm -v myapp_data:/data alpine ls -la /data

# Поиск файлов в volume
docker run --rm -v myapp_data:/data alpine find /data -name "*.log"

# Анализ использования дискового пространства в volume
docker run --rm -v myapp_data:/data alpine du -sh /data/*

# Копирование данных из volume
docker run --rm -v myapp_data:/data -v $(pwd):/backup alpine cp -r /data/* /backup/

# Восстановление данных в volume
docker run --rm -v myapp_data:/data -v $(pwd):/backup alpine cp -r /backup/* /data/

# Очистка volume
docker run --rm -v myapp_data:/data alpine sh -c "rm -rf /data/*"
```

### 14.7 Профилирование приложений

#### Node.js профилирование:
```dockerfile
FROM node:18-alpine

# Установка инструментов профилирования
RUN npm install -g clinic

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .

# Команды для профилирования
# docker exec myapp clinic doctor -- node app.js
# docker exec myapp clinic bubbleprof -- node app.js
# docker exec myapp clinic flame -- node app.js

CMD ["npm", "start"]
```

#### Python профилирование:
```dockerfile
FROM python:3.11-slim

# Установка инструментов профилирования
RUN pip install py-spy memory-profiler line-profiler

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

# Команды для профилирования
# docker exec myapp py-spy top --pid 1
# docker exec myapp python -m memory_profiler app.py

CMD ["python", "app.py"]
```

### 14.8 Автоматизированное тестирование production среды

#### Health checks и smoke tests:
```bash
#!/bin/bash
# smoke-test.sh

set -e

echo "🔍 Running smoke tests..."

# Проверка доступности сервисов
curl -f http://localhost/health || exit 1
curl -f http://localhost/api/health || exit 1

# Проверка базы данных
docker-compose exec -T postgres pg_isready -U $DB_USER -d $DB_NAME || exit 1

# Проверка Redis
docker-compose exec -T redis redis-cli ping | grep PONG || exit 1

# Проверка основных endpoints
curl -f -X POST http://localhost/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"test"}' || exit 1

echo "✅ All smoke tests passed!"
```

#### Автоматизированное тестирование в CI/CD:
```yaml
# .github/workflows/production-tests.yml
name: Production Health Check

on:
  schedule:
    - cron: '*/15 * * * *'  # Каждые 15 минут

jobs:
  health-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run health checks
        run: |
          curl -f https://api.example.com/health
          curl -f https://example.com/
          
      - name: Run smoke tests
        run: |
          ./scripts/smoke-test.sh
          
      - name: Notify on failure
        if: failure()
        uses: 8398a7/action-slack@v3
        with:
          status: failure
          text: "Production health check failed!"
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

---

## 🎓 Заключение курса

### Изученные навыки по блокам:

**Блок 1 - Введение в Docker:**
- ✅ Понимание проблем традиционной разработки
- ✅ Знание архитектуры Docker
- ✅ Установка и настройка Docker

**Блок 2 - Основы работы с Docker:**
- ✅ Управление образами и контейнерами
- ✅ Работа с портами, volumes, переменными окружения
- ✅ Практические сценарии применения

**Блок 3 - Создание образов:**
- ✅ Создание эффективных Dockerfile
- ✅ Многоэтапная сборка
- ✅ Оптимизация размера образов

**Блок 4 - Управление данными и сетями:**
- ✅ Постоянное хранение данных с volumes
- ✅ Сетевое взаимодействие между контейнерами
- ✅ Изоляция и безопасность

**Блок 5 - Docker Compose:**
- ✅ Оркестрация множественных контейнеров
- ✅ Управление различными окружениями
- ✅ Масштабирование и профили

**Блок 6 - Production-готовые решения:**
- ✅ Production конфигурации
- ✅ CI/CD интеграция
- ✅ Мониторинг и резервное копирование

**Блок 7 - Экспертные техники:**
- ✅ Оптимизация производительности
- ✅ Безопасность и сканирование уязвимостей
- ✅ Продвинутая отладка и мониторинг

### Рекомендации для дальнейшего развития:

1. **Практикуйтесь на реальных проектах** - применяйте полученные знания в своих проектах
2. **Изучите Kubernetes** - следующий шаг в оркестрации контейнеров
3. **Погрузитесь в DevOps практики** - GitOps, Infrastructure as Code
4. **Следите за новыми возможностями** - Docker постоянно развивается

### Полезные ресурсы:

- 📚 [Официальная документация Docker](https://docs.docker.com/)
- 🐙 [Docker Hub](https://hub.docker.com/) - репозиторий образов
- 💬 [Docker Community](https://www.docker.com/community/) - сообщество разработчиков
- 🛠️ [Awesome Docker](https://github.com/veggiemonk/awesome-docker) - полезные инструменты

**Удачи в вашем путешествии с Docker! 🐳**