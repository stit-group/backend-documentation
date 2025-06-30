# Docker Interview Guide - Развернутые ответы с примерами

## JUNIOR LEVEL (1-15)

### 1. **Что такое Docker и какие проблемы он решает?**

**Ответ:** Docker - это платформа контейнеризации, которая упаковывает приложение и все его зависимости в легковесный, переносимый контейнер.

**Решаемые проблемы:**
- **"Works on my machine"** - одинаковое окружение везде
- **Изоляция приложений** - каждое приложение в своем контейнере
- **Быстрое развертывание** - секунды вместо минут
- **Эффективное использование ресурсов** - легче виртуальных машин

**Отличия от VM:**
- Контейнеры используют ядро хоста (меньше ресурсов)
- Запуск за секунды vs минуты для VM
- Размер образов MB vs GB для VM

### 2. **Объясните разницу между Docker образом и контейнером**

**Ответ:**
- **Образ (Image)** - это read-only шаблон с файловой системой и метаданными
- **Контейнер** - это запущенный экземпляр образа с собственным процессом и изменяемым слоем

**Аналогия:** Образ = класс, Контейнер = объект класса

Из одного образа можно запустить множество контейнеров. Изменения в контейнере не влияют на образ.

### 3. **Основные команды Docker CLI**

```bash
# Жизненный цикл контейнера
docker run -d -p 8080:80 --name web nginx:alpine  # Запуск
docker stop web                                    # Остановка
docker start web                                   # Запуск остановленного
docker restart web                                 # Перезапуск
docker rm web                                      # Удаление

# Управление образами
docker pull nginx:alpine                          # Скачивание
docker build -t myapp .                          # Сборка
docker images                                    # Список образов
docker rmi nginx:alpine                          # Удаление образа

# Диагностика
docker ps                                        # Запущенные контейнеры
docker logs web                                  # Логи
docker exec -it web sh                          # Подключение к контейнеру
```

### 4. **Что такое Dockerfile и основные инструкции?**

**Ответ:** Dockerfile - текстовый файл с инструкциями для создания образа.

**Ключевые инструкции:**
```dockerfile
FROM node:alpine           # Базовый образ
WORKDIR /app               # Рабочая директория
COPY package.json .        # Копирование файлов
RUN npm install            # Выполнение команд при сборке
EXPOSE 3000               # Документирование порта
CMD ["npm", "start"]      # Команда по умолчанию
```

**Разница RUN vs CMD vs ENTRYPOINT:**
- `RUN` - выполняется при сборке образа
- `CMD` - команда по умолчанию, можно переопределить
- `ENTRYPOINT` - всегда выполняется, CMD становится аргументами

### 5. **Объясните Docker слои (layers)**

**Ответ:** Каждая инструкция в Dockerfile создает новый слой. Слои кешируются и переиспользуются.

**Принципы оптимизации:**
- Часто изменяемые файлы копировать в конце
- Группировать RUN команды
- Использовать .dockerignore

```dockerfile
# Плохо - dependencies пересобираются при каждом изменении кода
COPY . /app
RUN npm install

# Хорошо - кеш dependencies сохраняется
COPY package.json /app/
RUN npm install
COPY . /app
```

### 6. **Что такое .dockerignore?**

**Ответ:** Файл, исключающий файлы и директории из build context.

```dockerignore
node_modules
.git
.env
*.log
coverage/
```

**Преимущества:** Уменьшает размер build context, ускоряет сборку, повышает безопасность.

### 7. **Разница между ADD и COPY**

**Ответ:**
- `COPY` - просто копирует файлы/директории
- `ADD` - дополнительно распаковывает архивы и скачивает URL

```dockerfile
COPY app.tar.gz /app/          # Копирует архив как есть
ADD app.tar.gz /app/           # Копирует и распаковывает

# Рекомендация: используйте COPY, ADD только при необходимости
```

### 8. **Что такое Docker Hub?**

**Ответ:** Облачный реестр для хранения и распространения Docker образов.

**Основные функции:**
- Публичные и приватные репозитории
- Automated builds из GitHub/Bitbucket
- Webhook'и для CI/CD
- Vulnerability scanning

### 9. **Базовые типы Docker сетей**

**Ответ:**
- **bridge** (default) - изолированная сеть с NAT
- **host** - использует сеть хоста напрямую
- **none** - без сетевого интерфейса

```bash
# Создание custom network
docker network create mynetwork
docker run --network mynetwork nginx
```

### 10. **Что такое Docker volumes?**

**Ответ:**
- **Volumes** - управляются Docker, хранятся в /var/lib/docker/volumes/
- **Bind mounts** - привязка к конкретному пути на хосте

```bash
# Named volume
docker run -v mydata:/data nginx

# Bind mount
docker run -v /host/path:/container/path nginx
```

**Volumes переживают удаление контейнера, filesystem контейнера - нет.**

### 11. **Environment variables в Docker**

```dockerfile
# Build-time переменные
ARG VERSION=1.0.0

# Runtime переменные
ENV NODE_ENV=production
ENV PORT=3000

# Использование ARG в ENV
ENV APP_VERSION=${VERSION}
```

```bash
# Передача переменных при запуске
docker run -e NODE_ENV=development -e PORT=8080 myapp
```

### 12. **Что такое port mapping?**

**Ответ:**
- `EXPOSE` - документирует порт, не публикует его
- `-p` флаг - мапит порт хоста на порт контейнера

```bash
docker run -p 8080:80 nginx        # host:8080 -> container:80
docker run -p 127.0.0.1:8080:80 nginx  # Только localhost
```

### 13. **Docker образы: тегирование и версионирование**

**Ответы:**
```bash
# Правильное тегирование
docker build -t myapp:1.2.3 .
docker build -t myapp:1.2 .
docker build -t myapp:1 .
docker build -t myapp:latest .

# Проблемы с latest
# - Неопределенность версии
# - Проблемы с кешированием
# - Сложность rollback
```

**Лучшие практики:** Используйте семантическое версионирование, избегайте latest в production.

### 14. **Жизненный цикл контейнера**

**Состояния:**
- **Created** - создан, но не запущен
- **Running** - активно выполняется
- **Paused** - приостановлен
- **Stopped** - остановлен (exit code сохранен)
- **Removed** - удален

```bash
docker stop container    # Graceful shutdown (SIGTERM -> SIGKILL)
docker kill container    # Принудительное завершение (SIGKILL)
```

### 15. **Логи контейнеров**

```bash
docker logs container              # Все логи
docker logs -f container          # Follow (real-time)
docker logs --tail 100 container  # Последние 100 строк
docker logs --since 1h container  # За последний час
```

**Логи хранятся в:** `/var/lib/docker/containers/{container-id}/{container-id}-json.log`

---

## MIDDLE LEVEL (16-35)

### 16. **Оптимальный Dockerfile для Node.js приложения**

```dockerfile
# Multi-stage build
FROM node:alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:alpine AS runtime
# Создание non-root пользователя
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY --chown=nextjs:nodejs . .
USER nextjs
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
CMD ["npm", "start"]
```

### 17. **Multi-stage builds**

**Преимущества:**
- Уменьшение размера финального образа
- Разделение build и runtime окружений
- Лучшая безопасность (нет build tools в production)

```dockerfile
# Пример для Go приложения
FROM golang:alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o main .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
COPY --from=builder /app/main /main
CMD ["/main"]
```

### 18. **Docker Compose основы**

```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/myapp
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped
    
  db:
    image: postgres:13-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 30s
      timeout: 10s
      retries: 5
    secrets:
      - db_password

volumes:
  postgres_data:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### 19. **Зависимости между сервисами в Compose**

**Проблема:** `depends_on` только ждет запуска контейнера, а не готовности сервиса.

**Решения:**
```yaml
# 1. Health checks
depends_on:
  db:
    condition: service_healthy

# 2. Init containers
init: true

# 3. Wait scripts
command: ["./wait-for-it.sh", "db:5432", "--", "npm", "start"]
```

### 20. **Docker networks в production**

```yaml
# Изоляция сервисов
version: '3.8'
services:
  frontend:
    networks:
      - frontend-network
      
  backend:
    networks:
      - frontend-network
      - backend-network
      
  database:
    networks:
      - backend-network

networks:
  frontend-network:
  backend-network:
    internal: true  # Нет доступа к интернету
```

### 21. **Volumes в production**

```bash
# Backup volume
docker run --rm -v mydata:/data -v $(pwd):/backup alpine \
  tar czf /backup/backup.tar.gz -C /data .

# Restore volume
docker run --rm -v mydata:/data -v $(pwd):/backup alpine \
  tar xzf /backup/backup.tar.gz -C /data
```

### 22. **Мониторинг ресурсов контейнеров**

```yaml
# Resource limits
services:
  app:
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
        reservations:
          memory: 256M
          cpus: '0.25'
```

```bash
# Мониторинг
docker stats
docker top container
```

### 23. **Health checks в Docker**

```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

```yaml
# В compose
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

### 24. **Docker Registry: приватные образы**

```bash
# Login к registry
docker login registry.company.com

# Tag и push
docker tag myapp:latest registry.company.com/myapp:latest
docker push registry.company.com/myapp:latest
```

### 25. **Secrets в Docker**

**Проблемы с ENV:**
- Видны в `docker inspect`
- Сохраняются в shell history
- Передаются дочерним процессам

**Решения:**
```yaml
# Docker secrets
secrets:
  db_password:
    external: true
    
services:
  app:
    secrets:
      - db_password
```

### 26. **Restart policies**

```bash
# Типы restart policy
docker run --restart=no            # Никогда не перезапускать
docker run --restart=always        # Всегда перезапускать
docker run --restart=on-failure:3  # При ошибке, максимум 3 раза
docker run --restart=unless-stopped # Кроме ручной остановки
```

---

## SENIOR LEVEL (36-50)

### 36. **Production-ready Docker setup**

**Настройки daemon.json:**
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2",
  "live-restore": true,
  "userland-proxy": false,
  "default-ulimits": {
    "nofile": {
      "Hard": 64000,
      "Name": "nofile",
      "Soft": 64000
    }
  }
}
```

### 37. **Orchestration platforms**

**Docker Swarm vs Kubernetes:**

| Критерий | Docker Swarm | Kubernetes |
|----------|-------------|------------|
| Сложность | Простой | Сложный |
| Экосистема | Ограниченная | Обширная |
| Scaling | Базовый | Продвинутый |
| Use case | Простые приложения | Enterprise |

### 40. **Security: advanced topics**

```dockerfile
# Улучшенная безопасность
FROM node:alpine

# Обновление системы
RUN apk update && apk upgrade && apk add --no-cache dumb-init

# Non-root пользователь
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001 -G nodejs

WORKDIR /app
COPY --chown=nextjs:nodejs package*.json ./
RUN npm ci --only=production && npm cache clean --force

COPY --chown=nextjs:nodejs . .

USER nextjs

# Минимальные привилегии
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "server.js"]
```

### 43. **Monitoring и observability**

```yaml
# Пример с Prometheus
version: '3.8'
services:
  app:
    image: myapp
    labels:
      - "prometheus.io/scrape=true"
      - "prometheus.io/port=9090"
      - "prometheus.io/path=/metrics"
    
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
```

### 50. **Architecture design: микросервисы**

```yaml
# E-commerce архитектура
version: '3.8'
services:
  # API Gateway
  gateway:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    networks:
      - frontend
      
  # Микросервисы
  user-service:
    image: company/user-service:latest
    environment:
      - DATABASE_URL=postgresql://user:pass@user-db:5432/users
    networks:
      - frontend
      - user-backend
    deploy:
      replicas: 3
      
  product-service:
    image: company/product-service:latest
    networks:
      - frontend
      - product-backend
    deploy:
      replicas: 2
      
  # Базы данных
  user-db:
    image: postgres:13-alpine
    environment:
      POSTGRES_DB: users
    volumes:
      - user-data:/var/lib/postgresql/data
    networks:
      - user-backend
      
  # Service Discovery
  consul:
    image: consul:latest
    ports:
      - "8500:8500"
    networks:
      - frontend
      
  # Monitoring
  prometheus:
    image: prom/prometheus
    networks:
      - monitoring
      
networks:
  frontend:
  user-backend:
    internal: true
  product-backend:
    internal: true
  monitoring:
    
volumes:
  user-data:
  product-data:
```

---

## 🎯 Ключевые принципы для Senior уровня

1. **Security First**: Всегда non-root, минимальные образы, сканирование уязвимостей
2. **Observability**: Логи, метрики, трейсинг для всех компонентов
3. **Resilience**: Health checks, restart policies, graceful shutdown
4. **Scalability**: Stateless дизайн, горизонтальное масштабирование
5. **Resource Management**: Лимиты, мониторинг, capacity planning