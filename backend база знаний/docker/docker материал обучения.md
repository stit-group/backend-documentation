# 🐳 Docker для Backend разработчика: Полный курс от новичка к эксперту

## 📖 Содержание курса

1. [Введение и мотивация](#введение)
2. [Модуль 1: Основы Docker](#модуль-1-основы-docker)
3. [Модуль 2: Dockerfile мастерство](#модуль-2-dockerfile-мастерство)
4. [Модуль 3: Управление данными и сетями](#модуль-3-управление-данными-и-сетями)
5. [Модуль 4: Docker Compose основы](#модуль-4-docker-compose-основы)
6. [Модуль 5: Docker Compose продвинутое](#модуль-5-docker-compose-продвинутое)
7. [Модуль 6: Production и реальные проекты](#модуль-6-production-и-реальные-проекты)
8. [Бонус: Экспертные техники](#бонус-экспертные-техники)

---

## 🎯 Введение

### Проблемы традиционной разработки

```
🏠 Локальная машина     →     🔧 Тестовый сервер     →     🚀 Продакшн
Python 3.9              →     Python 3.8             →     Python 3.7
Ubuntu 20.04            →     CentOS 7               →     RHEL 8
MySQL 8.0               →     MySQL 5.7              →     PostgreSQL 13
```

**Результат:** "У меня работает!" → "На тесте падает" → "В продакшне не запускается"

### Docker решение

```
📦 Контейнер с приложением
├── Точно такая же ОС
├── Точно такие же зависимости  
├── Точно такие же настройки
└── Гарантированно работает везде
```

### Сравнение: Виртуальные машины vs Docker

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

### Архитектура Docker

```mermaid
graph TB
    A[Docker Client] -->|docker run| B[Docker Daemon]
    B --> C[Images]
    B --> D[Containers]
    B --> E[Networks]
    B --> F[Volumes]
    
    C --> G[Docker Hub]
    C --> H[Local Registry]
    
    subgraph "Docker Host"
        B
        C
        D
        E
        F
    end
```

---

## 📚 Модуль 1: Основы Docker (1 неделя)

### День 1-2: Установка и первое знакомство

#### Установка Docker

**Ubuntu/Debian:**
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

**macOS:**
```bash
# Установка через Homebrew
brew install --cask docker

# Или скачать Docker Desktop с официального сайта
```

**Windows:**
```bash
# Скачать Docker Desktop с официального сайта
# Требует WSL2 для лучшей производительности
```

#### Проверка установки

```bash
# Проверка версии
docker --version
# Docker version 24.0.5, build ced0996

# Проверка информации о системе
docker info

# Первый запуск
docker run hello-world
```

#### Жизненный цикл контейнера

```
📥 PULL IMAGE    →    🏗️ CREATE        →    ▶️ START       →    ⏸️ STOP        →    🗑️ REMOVE
docker pull           docker create        docker start       docker stop        docker rm
     ↓                      ↓                   ↓                  ↓                  ↓
Скачивание образа    Создание контейнера   Запуск процесса    Остановка          Удаление
```

### День 3-4: Работа с образами и контейнерами

#### Основные команды для образов

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

#### Работа с контейнерами

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

#### Работа с запущенными контейнерами

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

### День 5-7: Практические сценарии

#### Сценарий 1: Веб-сервер с кастомным контентом

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

#### Сценарий 2: База данных PostgreSQL

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

#### Сценарий 3: Redis для кэширования

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

#### Отладка и диагностика

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

#### Практическое задание: Многоконтейнерное приложение

Создайте связку из трех контейнеров:
1. **Frontend**: nginx для статики
2. **Backend**: простой API сервер
3. **Database**: PostgreSQL

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

# Backend (предполагаем, что у вас есть простое API)
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

## 🔨 Модуль 2: Dockerfile мастерство (1 неделя)

### День 1-2: Основы создания образов

#### Анатомия Dockerfile

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

#### Выбор базового образа

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

#### Инструкции Dockerfile подробно

**COPY vs ADD:**
```dockerfile
# COPY - простое копирование (рекомендуется)
COPY src/ /app/src/
COPY package*.json ./

# ADD - копирование с дополнительными возможностями
ADD https://example.com/file.tar.gz /tmp/  # Скачивание URL
ADD archive.tar.gz /app/                   # Автораспаковка архивов
```

**RUN оптимизация:**
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

**CMD vs ENTRYPOINT:**
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

#### Пример docker-entrypoint.sh

```bash
#!/bin/bash
set -e

# Ожидание базы данных
if [ "$DATABASE_URL" ]; then
    echo "Waiting for database..."
    while ! nc -z db 5432; do
        sleep 1
    done
    echo "Database is ready!"
fi

# Миграции
if [ "$RUN_MIGRATIONS" = "true" ]; then
    echo "Running migrations..."
    npm run migrate
fi

# Запуск основной команды
exec "$@"
```

### День 3-4: Продвинутые техники

#### Многоэтапная сборка (Multi-stage builds)

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

#### Оптимизация кэширования слоев

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

#### .dockerignore для оптимизации

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

#### ARG и ENV переменные

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

### День 5-7: Специализированные примеры

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

#### Dockerfile для Java приложения

```dockerfile
# ==========================================
# Этап сборки
# ==========================================
FROM maven:3.9-openjdk-17 AS builder

WORKDIR /app

# Копирование pom.xml для кэширования зависимостей
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Копирование исходного кода и сборка
COPY src ./src
RUN mvn clean package -DskipTests

# ==========================================
# Продакшн образ
# ==========================================
FROM openjdk:17-jre-slim

# Создание пользователя
RUN groupadd -r spring && useradd -r -g spring spring

WORKDIR /app

# Копирование JAR файла
COPY --from=builder /app/target/*.jar app.jar

# JVM оптимизации
ENV JAVA_OPTS="-Xmx512m -Xms256m -XX:+UseContainerSupport"

RUN chown spring:spring app.jar
USER spring

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

#### Практическое задание: Создание образа для микросервиса

Создайте Dockerfile для Node.js микросервиса со следующими требованиями:

1. Использует Node.js 18 Alpine
2. Многоэтапная сборка
3. Запускается от имени непривилегированного пользователя
4. Включает healthcheck
5. Оптимизирован для production
6. Размер образа < 200MB

```dockerfile
# Ваш Dockerfile здесь
```

---

## 💾 Модуль 3: Управление данными и сетями (1 неделя)

### День 1-3: Volumes - управление данными

#### Проблема временности контейнеров

```
🐳 Контейнер запускается
📝 Приложение создает данные
💾 Данные сохраняются в контейнере
🛑 Контейнер останавливается
🗑️ Контейнер удаляется
💔 Данные теряются НАВСЕГДА!
```

#### Типы хранения данных

```mermaid
graph TB
    A[Типы хранения в Docker] --> B[Volumes]
    A --> C[Bind Mounts]
    A --> D[tmpfs mounts]
    
    B --> E[Управляются Docker]
    B --> F[Лучшая производительность]
    B --> G[Переносимость между контейнерами]
    
    C --> H[Прямое монтирование с хоста]
    C --> I[Зависимость от структуры хоста]
    C --> J[Подходит для разработки]
    
    D --> K[Хранение в памяти]
    D --> L[Временные данные]
    D --> M[Исчезают при остановке]
```

#### Named Volumes

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

#### Bind Mounts

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

#### Практические сценарии с volumes

**Сценарий 1: База данных с постоянным хранением**

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

**Сценарий 2: Разработка с live reload**

```bash
# Структура проекта
mkdir -p myapp/{src,public}
echo "console.log('Hello from Node.js!');" > myapp/src/app.js
echo "<h1>Hello World!</h1>" > myapp/public/index.html

# Создание package.json
cat > myapp/package.json << EOF
{
  "name": "myapp",
  "version": "1.0.0",
  "main": "src/app.js",
  "scripts": {
    "dev": "nodemon src/app.js"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.0"
  }
}
EOF

# Запуск с bind mount для разработки
docker run -it --rm \
  --name dev-container \
  -v $(pwd)/myapp:/app \
  -w /app \
  -p 3000:3000 \
  node:18-alpine \
  sh -c "npm install && npm run dev"
```

**Сценарий 3: Логи и мониторинг**

```bash
# Создание volume для логов
docker volume create app_logs

# Запуск приложения с логами в volume
docker run -d \
  --name myapp \
  -v app_logs:/var/log/app \
  myapp:latest

# Запуск анализатора логов
docker run -d \
  --name log-analyzer \
  -v app_logs:/logs:ro \
  fluentd:latest

# Просмотр логов через отдельный контейнер
docker run --rm \
  -v app_logs:/logs \
  alpine tail -f /logs/app.log
```

### День 4-7: Networks - сетевое взаимодействие

#### Типы сетей Docker

```mermaid
graph TB
    A[Docker Networks] --> B[bridge]
    A --> C[host]
    A --> D[none]
    A --> E[overlay]
    
    B --> F[Изолированная сеть]
    B --> G[NAT к хосту]
    B --> H[По умолчанию]
    
    C --> I[Использует сеть хоста]
    C --> J[Нет изоляции]
    C --> K[Максимальная производительность]
    
    D --> L[Без сети]
    D --> M[Полная изоляция]
    
    E --> N[Многохостовые сети]
    E --> O[Docker Swarm]
```

#### Работа с сетями

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

#### Практические примеры сетевого взаимодействия

**Пример 1: Связка Frontend + Backend + Database**

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

**Пример 2: Микросервисная архитектура**

```bash
# Создание сетей для разных сервисов
docker network create frontend-network
docker network create backend-network
docker network create database-network

# Database сервисы
docker run -d \
  --name postgres \
  --network database-network \
  -e POSTGRES_DB=users \
  postgres:15-alpine

docker run -d \
  --name redis \
  --network database-network \
  redis:alpine

# Backend сервисы
docker run -d \
  --name user-service \
  --network backend-network \
  --network database-network \
  -e DATABASE_URL=postgresql://postgres:5432/users \
  -e REDIS_URL=redis://redis:6379 \
  user-service:latest

docker run -d \
  --name auth-service \
  --network backend-network \
  --network database-network \
  -e REDIS_URL=redis://redis:6379 \
  auth-service:latest

# API Gateway
docker run -d \
  --name api-gateway \
  --network frontend-network \
  --network backend-network \
  -p 80:80 \
  -e USER_SERVICE_URL=http://user-service:3000 \
  -e AUTH_SERVICE_URL=http://auth-service:3001 \
  api-gateway:latest

# Frontend
docker run -d \
  --name frontend \
  --network frontend-network \
  -p 8080:80 \
  -e API_URL=http://api-gateway \
  frontend:latest
```

#### Service Discovery и DNS

```bash
# Проверка DNS резолюции между контейнерами
docker exec -it frontend nslookup api-gateway

# Проверка подключения
docker exec -it frontend ping api-gateway
docker exec -it frontend curl http://api-gateway/health

# Просмотр сетевых интерфейсов контейнера
docker exec -it frontend ip addr show

# Просмотр таблицы маршрутизации
docker exec -it frontend ip route
```

#### Сетевые политики безопасности

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

#### Мониторинг сетевого трафика

```bash
# Просмотр статистики сети контейнера
docker stats --format "table {{.Container}}\t{{.NetIO}}"

# Анализ трафика с помощью tcpdump
docker run --rm --net container:myapp \
  nicolaka/netshoot tcpdump -i eth0

# Сканирование портов
docker run --rm --net container:myapp \
  nicolaka/netshoot nmap -p 1-1000 localhost
```

---

## 🏗️ Модуль 4: Docker Compose основы (1 неделя)

### День 1-2: Введение в Docker Compose

#### Проблемы управления множественными контейнерами

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

#### Docker Compose решение

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

#### Основные команды Docker Compose

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

### День 3-4: Конфигурация сервисов

#### Полная структура docker-compose.yml

```yaml
version: '3.8'

# Определение сервисов
services:
  
  # Веб-сервер
  nginx:
    image: nginx:alpine
    container_name: myapp_nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
      - static_files:/usr/share/nginx/html
    networks:
      - frontend
    depends_on:
      - api
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s

  # API сервис
  api:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        - NODE_ENV=production
        - VERSION=1.0.0
      target: production
    container_name: myapp_api
    restart: unless-stopped
    environment:
      - NODE_ENV=production
      - PORT=3000
      - DATABASE_URL=postgresql://${DB_USER}:${DB_PASSWORD}@postgres:5432/${DB_NAME}
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=${JWT_SECRET}
    env_file:
      - .env
    volumes:
      - ./uploads:/app/uploads
      - app_logs:/var/log/app
    networks:
      - frontend
      - backend
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
        reservations:
          memory: 256M
          cpus: '0.25'

  # База данных
  postgres:
    image: postgres:15-alpine
    container_name: myapp_postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_INITDB_ARGS: "--encoding=UTF8 --locale=C"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER} -d ${DB_NAME}"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          memory: 1G
          cpus: '1.0'

  # Кэш
  redis:
    image: redis:alpine
    container_name: myapp_redis
    restart: unless-stopped
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "--raw", "incr", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

  # Воркеры для фоновых задач
  worker:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    restart: unless-stopped
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://${DB_USER}:${DB_PASSWORD}@postgres:5432/${DB_NAME}
      - REDIS_URL=redis://redis:6379
    command: ["npm", "run", "worker"]
    volumes:
      - app_logs:/var/log/app
    networks:
      - backend
    depends_on:
      - postgres
      - redis
    deploy:
      replicas: 2

# Volumes для постоянного хранения
volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local
  app_logs:
    driver: local
  static_files:
    driver: local

# Сети для изоляции сервисов
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # Без доступа к интернету
```

#### Переменные окружения

**.env файл:**
```bash
# Database
DB_NAME=myapp
DB_USER=api
DB_PASSWORD=super_secret_password

# Redis
REDIS_PASSWORD=redis_secret

# Application
JWT_SECRET=your_jwt_secret_here
API_PORT=3000
NODE_ENV=production

# External APIs
STRIPE_SECRET_KEY=sk_test_...
SENDGRID_API_KEY=SG.xxx

# Monitoring
SENTRY_DSN=https://xxx@sentry.io/xxx
```

**Использование переменных:**
```yaml
services:
  api:
    environment:
      - DATABASE_URL=postgresql://${DB_USER}:${DB_PASSWORD}@postgres:5432/${DB_NAME}
      - PORT=${API_PORT:-3000}  # Значение по умолчанию
      - DEBUG=${DEBUG:-false}
    env_file:
      - .env
      - .env.local  # Локальные переопределения
```

#### Зависимости между сервисами

```yaml
services:
  api:
    depends_on:
      # Простая зависимость
      - postgres
      - redis
      
  api-with-health:
    depends_on:
      # Зависимость с проверкой здоровья
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
      migration:
        condition: service_completed_successfully

  migration:
    image: myapi:latest
    command: ["npm", "run", "migrate"]
    depends_on:
      postgres:
        condition: service_healthy
    restart: "no"  # Выполнить один раз
```

### День 5-7: Практические примеры

#### Полноценное веб-приложение

**Структура проекта:**
```
myproject/
├── docker-compose.yml
├── .env
├── nginx/
│   └── nginx.conf
├── api/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
└── scripts/
    ├── init.sql
    └── wait-for-it.sh
```

**docker-compose.yml:**
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

**nginx.conf:**
```nginx
events {
    worker_connections 1024;
}

http {
    upstream api {
        server api:3000;
    }

    upstream frontend {
        server frontend:80;
    }

    server {
        listen 80;
        
        # API прокси
        location /api/ {
            proxy_pass http://api/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
        
        # Frontend
        location / {
            proxy_pass http://frontend/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

#### Скрипт для разработки

**start-dev.sh:**
```bash
#!/bin/bash

# Проверка .env файла
if [ ! -f .env ]; then
    echo "Создание .env файла из шаблона..."
    cp .env.example .env
fi

# Сборка и запуск для разработки
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up --build -d

echo "Приложение запущено!"
echo "Frontend: http://localhost:3000"
echo "API: http://localhost:3001"
echo "Adminer: http://localhost:8080"

# Просмотр логов
docker-compose logs -f
```

**docker-compose.dev.yml (override для разработки):**
```yaml
version: '3.8'

services:
  frontend:
    build:
      target: development
    volumes:
      - ./frontend/src:/app/src
      - ./frontend/public:/app/public
    environment:
      - REACT_APP_API_URL=http://localhost:3001
    ports:
      - "3000:3000"
    command: ["npm", "run", "dev"]

  api:
    build:
      target: development
    volumes:
      - ./api/src:/app/src
    environment:
      - NODE_ENV=development
      - DEBUG=api:*
    ports:
      - "3001:3000"
    command: ["npm", "run", "dev"]

  adminer:
    image: adminer
    restart: always
    ports:
      - "8080:8080"
    networks:
      - backend
```

---

## 🚀 Модуль 5: Docker Compose продвинутое (1 неделя)

### День 1-3: Множественные окружения

#### Стратегия окружений

```
📁 Структура конфигураций
├── docker-compose.yml           # Базовая конфигурация
├── docker-compose.dev.yml       # Разработка
├── docker-compose.test.yml      # Тестирование
├── docker-compose.staging.yml   # Staging
├── docker-compose.prod.yml      # Production
├── .env.example                 # Шаблон переменных
├── .env.dev                     # Переменные для разработки
├── .env.test                    # Переменные для тестов
├── .env.staging                 # Переменные для staging
└── .env.prod                    # Переменные для production
```

#### Базовая конфигурация

**docker-compose.yml:**
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

#### Development окружение

**docker-compose.dev.yml:**
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

#### Test окружение

**docker-compose.test.yml:**
```yaml
version: '3.8'

services:
  api:
    build:
      target: test
    environment:
      - NODE_ENV=test
      - DATABASE_URL=postgresql://test:test@postgres-test:5432/test_db
    command: ["npm", "run", "test"]
    depends_on:
      postgres-test:
        condition: service_healthy

  postgres-test:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: test_db
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
    tmpfs:
      - /var/lib/postgresql/data  # В памяти для скорости
    networks:
      - test
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U test -d test_db"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis-test:
    image: redis:alpine
    tmpfs:
      - /data
    networks:
      - test

networks:
  test:
    driver: bridge
```

#### Production окружение

**docker-compose.prod.yml:**
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
    logging:
      driver: "fluentd"
      options:
        fluentd-address: localhost:24224
        tag: api.{{.Name}}

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

  worker:
    build:
      target: production
    restart: unless-stopped
    command: ["npm", "run", "worker"]
    deploy:
      replicas: 2
    networks:
      - backend

  monitoring:
    image: prom/prometheus
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    networks:
      - monitoring

volumes:
  static_files:

networks:
  frontend:
    driver: bridge
  monitoring:
    driver: bridge
```

#### Запуск разных окружений

```bash
# Development
docker-compose -f docker-compose.yml -f docker-compose.dev.yml --env-file .env.dev up -d

# Test
docker-compose -f docker-compose.yml -f docker-compose.test.yml --env-file .env.test up --abort-on-container-exit

# Staging
docker-compose -f docker-compose.yml -f docker-compose.staging.yml --env-file .env.staging up -d

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml --env-file .env.prod up -d
```

### День 4-7: Продвинутые возможности

#### Profiles для условного запуска

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

# С мониторингом
docker-compose --profile monitoring up -d

# Полная конфигурация
docker-compose --profile cache --profile monitoring --profile debug up -d
```

#### Масштабирование сервисов

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

#### Продвинутые healthchecks

```yaml
version: '3.8'

services:
  api:
    build: .
    healthcheck:
      test: |
        curl -f http://localhost:3000/health || exit 1
        curl -f http://localhost:3000/ready || exit 1
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s

  postgres:
    image: postgres:15-alpine
    healthcheck:
      test: |
        pg_isready -U $POSTGRES_USER -d $POSTGRES_DB &&
        psql -U $POSTGRES_USER -d $POSTGRES_DB -c "SELECT 1" > /dev/null
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  redis:
    image: redis:alpine
    healthcheck:
      test: |
        redis-cli ping &&
        redis-cli info replication | grep role:master
      interval: 10s
      timeout: 3s
      retries: 5

  elasticsearch:
    image: elasticsearch:8.8.0
    healthcheck:
      test: |
        curl -s http://localhost:9200/_cluster/health | grep -q '"status":"green"' ||
        curl -s http://localhost:9200/_cluster/health | grep -q '"status":"yellow"'
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 60s
```

#### Секреты и безопасность

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

#### Мониторинг и логирование

```yaml
version: '3.8'

x-logging: &default-logging
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"

services:
  api:
    build: .
    logging:
      driver: "fluentd"
      options:
        fluentd-address: fluentd:24224
        tag: "api.{{.Name}}"
    depends_on:
      - fluentd

  fluentd:
    image: fluentd:v1.16-1
    volumes:
      - ./fluentd/conf:/fluentd/etc
      - ./logs:/var/log/fluentd
    ports:
      - "24224:24224"
    networks:
      - logging

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

volumes:
  prometheus_data:
  grafana_data:

networks:
  logging:
    driver: bridge
  monitoring:
    driver: bridge
```

---

## 🎯 Модуль 6: Production и реальные проекты (1 неделя)

### День 1-3: Production-ready конфигурация

#### Полный production stack

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

  frontend:
    image: nginx:alpine
    <<: *restart-policy
    volumes:
      - ./dist:/usr/share/nginx/html:ro
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.frontend.rule=Host(`example.com`)"
      - "traefik.http.routers.frontend.tls.certresolver=letsencrypt"
    healthcheck:
      <<: *healthcheck-defaults
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost/"]
    logging: *logging

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

#### CI/CD интеграция

**GitHub Actions workflow:**
```yaml
# .github/workflows/deploy.yml
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

#### Monitoring и логирование

**monitoring/prometheus.yml:**
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

**Grafana dashboards configuration:**
```yaml
# monitoring/grafana/provisioning/dashboards/dashboard.yml
apiVersion: 1

providers:
  - name: 'default'
    orgId: 1
    folder: ''
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /etc/grafana/provisioning/dashboards
```

### День 4-7: Реальные проекты и кейсы

#### Кейс 1: E-commerce платформа

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

  payment-service:
    build: ./services/payment-service
    environment:
      - DATABASE_URL=postgresql://payments:password@payment-db:5432/payments
      - STRIPE_SECRET_KEY=${STRIPE_SECRET_KEY}
    networks:
      - kong-net
      - payment-net

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

  # Кэш
  redis:
    image: redis:alpine
    volumes:
      - redis_data:/data
    networks:
      - order-net

  # Frontend
  web-app:
    build: ./frontend
    ports:
      - "80:80"
    environment:
      - API_URL=http://kong:8000
    depends_on:
      - kong

volumes:
  user_data:
  product_data:
  order_data:
  payment_data:
  elasticsearch_data:
  rabbitmq_data:
  redis_data:

networks:
  kong-net:
  user-net:
    internal: true
  product-net:
    internal: true
  order-net:
    internal: true
  payment-net:
    internal: true
```

#### Кейс 2: Data Pipeline

```yaml
version: '3.8'

services:
  # Data Sources
  postgres-source:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: source_data
      POSTGRES_USER: etl
      POSTGRES_PASSWORD: password
    volumes:
      - source_data:/var/lib/postgresql/data

  # ETL Pipeline
  airflow-webserver:
    image: apache/airflow:2.6.0
    environment:
      - AIRFLOW__CORE__EXECUTOR=CeleryExecutor
      - AIRFLOW__DATABASE__SQL_ALCHEMY_CONN=postgresql+psycopg2://airflow:airflow@postgres:5432/airflow
      - AIRFLOW__CELERY__RESULT_BACKEND=db+postgresql://airflow:airflow@postgres:5432/airflow
      - AIRFLOW__CELERY__BROKER_URL=redis://:@redis:6379/0
    ports:
      - "8080:8080"
    volumes:
      - ./dags:/opt/airflow/dags
      - ./logs:/opt/airflow/logs
      - ./plugins:/opt/airflow/plugins
    depends_on:
      - postgres
      - redis

  airflow-scheduler:
    image: apache/airflow:2.6.0
    command: scheduler
    environment:
      - AIRFLOW__CORE__EXECUTOR=CeleryExecutor
      - AIRFLOW__DATABASE__SQL_ALCHEMY_CONN=postgresql+psycopg2://airflow:airflow@postgres:5432/airflow
      - AIRFLOW__CELERY__RESULT_BACKEND=db+postgresql://airflow:airflow@postgres:5432/airflow
      - AIRFLOW__CELERY__BROKER_URL=redis://:@redis:6379/0
    volumes:
      - ./dags:/opt/airflow/dags
      - ./logs:/opt/airflow/logs
      - ./plugins:/opt/airflow/plugins

  # Data Warehouse
  clickhouse:
    image: clickhouse/clickhouse-server
    ports:
      - "8123:8123"
      - "9000:9000"
    volumes:
      - clickhouse_data:/var/lib/clickhouse
    environment:
      CLICKHOUSE_DB: analytics
      CLICKHOUSE_USER: default
      CLICKHOUSE_PASSWORD: password

  # Analytics
  superset:
    image: apache/superset
    ports:
      - "8088:8088"
    environment:
      - SUPERSET_SECRET_KEY=your-secret-key
    volumes:
      - superset_data:/app/superset_home

volumes:
  source_data:
  clickhouse_data:
  superset_data:
```

#### Скрипты управления

**deploy.sh:**
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
    # Логика для отката к предыдущей версии
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

**backup.sh:**
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

## 🏆 Бонус: Экспертные техники

### Оптимизация производительности

#### BuildKit и кэширование

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

#### Distroless образы

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

### Безопасность

#### Сканирование уязвимостей

```bash
# Сканирование образа
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image myapp:latest

# Сканирование в CI/CD
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image --exit-code 1 --severity HIGH,CRITICAL myapp:latest
```

#### Подписывание образов

```bash
# Docker Content Trust
export DOCKER_CONTENT_TRUST=1
docker push myapp:latest

# Notary подпись
notary -s https://notary.docker.io -d ~/.docker/trust init myapp
notary -s https://notary.docker.io -d ~/.docker/trust publish myapp:latest
```

### Продвинутая отладка

#### Отладочные контейнеры

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
```

#### Анализ производительности

```bash
# Анализ производительности контейнера
docker run --rm -it --pid container:myapp --cap-add SYS_PTRACE \
  brendangregg/perf-tools

# Профилирование с помощью htop
docker run --rm -it --pid container:myapp \
  terencehill/htop
```

### Заключение

Поздравляем! Вы прошли полный курс Docker для backend разработчиков. Теперь вы обладаете знаниями и навыками для:

🎯 **Базового уровня:**
- Контейнеризации любых приложений
- Создания эффективных Dockerfile
- Управления данными и сетями
- Использования Docker Compose

🚀 **Продвинутого уровня:**
- Настройки production окружений
- Масштабирования приложений
- Мониторинга и логирования
- CI/CD интеграции

💎 **Экспертного уровня:**
- Оптимизации производительности
- Обеспечения безопасности
- Отладки сложных проблем
- Управления большими проектами

**Следующие шаги:**
1. Практикуйтесь на реальных проектах
2. Изучите Kubernetes для оркестрации
3. Погрузитесь в DevOps практики
4. Следите за новыми возможностями Docker

Удачи в вашем путешествии с Docker! 🐳