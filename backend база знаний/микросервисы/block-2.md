# 🏗️ Блок 2: Технологический стек и инфраструктура микросервисов

═══════════════════════════════════════════════════════════════

**⏱️ Продолжительность:** 5-7 недель  
**🎯 Цель:** Освоить ключевые технологии для разработки и развертывания микросервисов  
**📚 Формат:** Теория + практические лабораторные работы

═══════════════════════════════════════════════════════════════

## 📋 Обзор блока

```
┌─────────────────────────────────────────────────────────────┐
│                    ТЕХНОЛОГИЧЕСКИЙ СТЕК                     │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   Docker    │  │ Kubernetes  │  │ API Gateway │         │
│  │ (Контейнеры)│  │(Оркестрация)│  │ (Маршрутиз.)│         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │Service Disc.│  │ Мониторинг  │  │   CI/CD     │         │
│  │(Обнаружение)│  │(Prometheus) │  │ (Автомат.)  │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

---

## 🐳 Глава 1: Контейнеризация с Docker

### ▶️ Теоретическая основа

**Что такое контейнеризация?**

Контейнеризация — это метод виртуализации на уровне операционной системы, который позволяет запускать приложения в изолированных пространствах, называемых контейнерами.

```
┌──────────────────────────────────────────────────────────┐
│                    ТРАДИЦИОННАЯ МОДЕЛЬ                   │
│                                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
│  │     App     │  │     App     │  │     App     │      │
│  ├─────────────┤  ├─────────────┤  ├─────────────┤      │
│  │Guest OS     │  │Guest OS     │  │Guest OS     │      │
│  ├─────────────┤  ├─────────────┤  ├─────────────┤      │
│  │ Hypervisor  │  │ Hypervisor  │  │ Hypervisor  │      │
│  └─────────────┘  └─────────────┘  └─────────────┘      │
│                    Host Operating System                 │
│                         Hardware                         │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│                    КОНТЕЙНЕРИЗАЦИЯ                       │
│                                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
│  │     App     │  │     App     │  │     App     │      │
│  │   + Deps    │  │   + Deps    │  │   + Deps    │      │
│  ├─────────────┤  ├─────────────┤  ├─────────────┤      │
│  │             Docker Engine / Container Runtime        │
│  ├──────────────────────────────────────────────────────┤
│  │                Host Operating System                 │
│  │                        Hardware                      │
└──────────────────────────────────────────────────────────┘
```

**Преимущества Docker для микросервисов:**

🔹 **Консистентность**: "Works on my machine" больше не проблема  
🔹 **Изоляция**: Каждый сервис работает в своем окружении  
🔹 **Портативность**: Один образ работает везде  
🔹 **Масштабируемость**: Легко создавать новые экземпляры  
🔹 **Ресурсоэффективность**: Меньше накладных расходов, чем у VM

### 🛠️ Практическая часть

#### Dockerfile Best Practices

```dockerfile
# ═══════════════════════════════════════════════════════════
# ПЛОХОЙ ПРИМЕР - НЕ ИСПОЛЬЗУЙТЕ!
# ═══════════════════════════════════════════════════════════
FROM node:latest
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["npm", "start"]

# ═══════════════════════════════════════════════════════════
# ХОРОШИЙ ПРИМЕР - BEST PRACTICES
# ═══════════════════════════════════════════════════════════

# 1. Используйте конкретную версию базового образа
FROM node:18.17.0-alpine AS base

# 2. Установите рабочую директорию
WORKDIR /app

# 3. Создайте пользователя без root-прав
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# 4. Сначала копируйте файлы зависимостей для кэширования
COPY package*.json ./
COPY --chown=nextjs:nodejs package*.json ./

# 5. Установите только production зависимости
RUN npm ci --only=production && npm cache clean --force

# ═══════════════════════════════════════════════════════════
# MULTI-STAGE BUILD
# ═══════════════════════════════════════════════════════════

# Стадия сборки
FROM base AS builder
RUN npm ci
COPY . .
RUN npm run build

# Продакшн стадия
FROM base AS runtime
COPY --from=builder --chown=nextjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nextjs:nodejs /app/public ./public

# 6. Смените пользователя на non-root
USER nextjs

# 7. Используйте EXPOSE для документирования портов
EXPOSE 3000

# 8. Используйте exec форму для CMD
CMD ["node", "dist/server.js"]

# 9. Добавьте healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```

#### Docker Compose для локальной разработки

```yaml
# ═══════════════════════════════════════════════════════════
# docker-compose.yml - Полный стек для разработки
# ═══════════════════════════════════════════════════════════

version: '3.8'

services:
  # ───────────────────────────────────────────────────────
  # API Gateway
  # ───────────────────────────────────────────────────────
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - user-service
      - order-service
      - product-service
    networks:
      - microservices-net

  # ───────────────────────────────────────────────────────
  # Микросервисы
  # ───────────────────────────────────────────────────────
  user-service:
    build: 
      context: ./user-service
      dockerfile: Dockerfile.dev
    environment:
      - NODE_ENV=development
      - DB_HOST=postgres-users
      - REDIS_HOST=redis
    volumes:
      - ./user-service:/app
      - /app/node_modules
    networks:
      - microservices-net
    depends_on:
      - postgres-users
      - redis

  order-service:
    build: 
      context: ./order-service
      dockerfile: Dockerfile.dev
    environment:
      - NODE_ENV=development
      - DB_HOST=postgres-orders
      - MESSAGE_BROKER=rabbitmq
    volumes:
      - ./order-service:/app
      - /app/node_modules
    networks:
      - microservices-net
    depends_on:
      - postgres-orders
      - rabbitmq

  product-service:
    build: 
      context: ./product-service
      dockerfile: Dockerfile.dev
    environment:
      - NODE_ENV=development
      - DB_HOST=mongo
    volumes:
      - ./product-service:/app
      - /app/node_modules
    networks:
      - microservices-net
    depends_on:
      - mongo

  # ───────────────────────────────────────────────────────
  # Базы данных
  # ───────────────────────────────────────────────────────
  postgres-users:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: users_db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password123
    volumes:
      - postgres-users-data:/var/lib/postgresql/data
    networks:
      - microservices-net

  postgres-orders:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: orders_db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password123
    volumes:
      - postgres-orders-data:/var/lib/postgresql/data
    networks:
      - microservices-net

  mongo:
    image: mongo:6
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password123
    volumes:
      - mongo-data:/data/db
    networks:
      - microservices-net

  # ───────────────────────────────────────────────────────
  # Инфраструктурные сервисы
  # ───────────────────────────────────────────────────────
  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    networks:
      - microservices-net

  rabbitmq:
    image: rabbitmq:3-management-alpine
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: password123
    ports:
      - "15672:15672"  # Management UI
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
    networks:
      - microservices-net

volumes:
  postgres-users-data:
  postgres-orders-data:
  mongo-data:
  redis-data:
  rabbitmq-data:

networks:
  microservices-net:
    driver: bridge
```

---

## ☸️ Глава 2: Оркестрация контейнеров с Kubernetes

### ▶️ Теоретическая основа

**Kubernetes Architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│                        KUBERNETES CLUSTER                   │
│                                                             │
│  ┌─────────────────┐              ┌─────────────────────────┐│
│  │   MASTER NODE   │              │      WORKER NODES       ││
│  │                 │              │                         ││
│  │ ┌─────────────┐ │              │ ┌─────────────────────┐ ││
│  │ │  API Server │ │◄────────────►│ │      kubelet        │ ││
│  │ └─────────────┘ │              │ └─────────────────────┘ ││
│  │ ┌─────────────┐ │              │ ┌─────────────────────┐ ││
│  │ │   etcd      │ │              │ │    kube-proxy       │ ││
│  │ └─────────────┘ │              │ └─────────────────────┘ ││
│  │ ┌─────────────┐ │              │ ┌─────────────────────┐ ││
│  │ │  Scheduler  │ │              │ │ Container Runtime   │ ││
│  │ └─────────────┘ │              │ │   (Docker/containerd)││
│  │ ┌─────────────┐ │              │ └─────────────────────┘ ││
│  │ │Controller   │ │              │                         ││
│  │ │Manager      │ │              │ ┌─────┐ ┌─────┐ ┌─────┐ ││
│  │ └─────────────┘ │              │ │Pod 1│ │Pod 2│ │Pod 3│ ││
│  └─────────────────┘              │ └─────┘ └─────┘ └─────┘ ││
│                                   └─────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

**Основные концепции Kubernetes:**

🔹 **Pod** - минимальная единица развертывания  
🔹 **Service** - абстракция для доступа к Pod'ам  
🔹 **Deployment** - декларативное управление Pod'ами  
🔹 **ConfigMap** - конфигурационные данные  
🔹 **Secret** - чувствительные данные  
🔹 **Ingress** - управление внешним доступом

### 🛠️ Практическая часть

#### Deployment микросервиса

```yaml
# ═══════════════════════════════════════════════════════════
# user-service-deployment.yaml
# ═══════════════════════════════════════════════════════════

apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  namespace: microservices
  labels:
    app: user-service
    version: v1.0.0
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
        version: v1.0.0
    spec:
      containers:
      - name: user-service
        image: myregistry/user-service:1.0.0
        ports:
        - containerPort: 3000
          name: http
        env:
        - name: NODE_ENV
          value: "production"
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: user-service-config
              key: database-host
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: user-service-secrets
              key: database-password
        
        # ───────────────────────────────────────────────────
        # Resource limits and requests
        # ───────────────────────────────────────────────────
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        
        # ───────────────────────────────────────────────────
        # Health checks
        # ───────────────────────────────────────────────────
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3

---
# ═══════════════════════════════════════════════════════════
# Service для внутренней коммуникации
# ═══════════════════════════════════════════════════════════

apiVersion: v1
kind: Service
metadata:
  name: user-service
  namespace: microservices
  labels:
    app: user-service
spec:
  selector:
    app: user-service
  ports:
  - port: 80
    targetPort: 3000
    protocol: TCP
    name: http
  type: ClusterIP

---
# ═══════════════════════════════════════════════════════════
# ConfigMap с конфигурацией
# ═══════════════════════════════════════════════════════════

apiVersion: v1
kind: ConfigMap
metadata:
  name: user-service-config
  namespace: microservices
data:
  database-host: "postgres-users.microservices.svc.cluster.local"
  database-port: "5432"
  database-name: "users_db"
  redis-host: "redis.microservices.svc.cluster.local"
  log-level: "info"

---
# ═══════════════════════════════════════════════════════════
# Secret с чувствительными данными
# ═══════════════════════════════════════════════════════════

apiVersion: v1
kind: Secret
metadata:
  name: user-service-secrets
  namespace: microservices
type: Opaque
data:
  database-password: cGFzc3dvcmQxMjM=  # base64: password123
  jwt-secret: bXlfc3VwZXJfc2VjcmV0X2tleQ==  # base64: my_super_secret_key
```

#### Ingress для внешнего доступа

```yaml
# ═══════════════════════════════════════════════════════════
# ingress.yaml - Входная точка для API
# ═══════════════════════════════════════════════════════════

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: microservices-ingress
  namespace: microservices
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - api.mycompany.com
    secretName: api-tls-secret
  rules:
  - host: api.mycompany.com
    http:
      paths:
      # ─────────────────────────────────────────────────────
      # User Service routes
      # ─────────────────────────────────────────────────────
      - path: /api/users(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
      
      # ─────────────────────────────────────────────────────
      # Order Service routes
      # ─────────────────────────────────────────────────────
      - path: /api/orders(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: order-service
            port:
              number: 80
      
      # ─────────────────────────────────────────────────────
      # Product Service routes
      # ─────────────────────────────────────────────────────
      - path: /api/products(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: product-service
            port:
              number: 80
```

---

## 🚪 Глава 3: API Gateway

### ▶️ Теоретическая основа

**Роль API Gateway в микросервисной архитектуре:**

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENT REQUESTS                      │
│    ┌─────────────┐ ┌─────────────┐ ┌─────────────┐         │
│    │Mobile Apps  │ │Web Browser  │ │Third-party  │         │
│    └─────────────┘ └─────────────┘ └─────────────┘         │
│              │              │              │               │
│              └──────────────┼──────────────┘               │
│                             │                               │
│              ┌──────────────▼──────────────┐               │
│              │         API GATEWAY         │               │
│              │                             │               │
│              │ ┌─────────────────────────┐ │               │
│              │ │    Authentication       │ │               │
│              │ │    Rate Limiting        │ │               │
│              │ │    Request Routing      │ │               │
│              │ │    Response Aggregation │ │               │
│              │ │    Load Balancing       │ │               │
│              │ │    SSL Termination      │ │               │
│              │ └─────────────────────────┘ │               │
│              └─────────────┬───────────────┘               │
│                           │                               │
│    ┌─────────────┐ ┌─────────────┐ ┌─────────────┐         │
│    │User Service │ │Order Service│ │Product Srv  │         │
│    └─────────────┘ └─────────────┘ └─────────────┘         │
│                      MICROSERVICES                         │
└─────────────────────────────────────────────────────────────┘
```

**Основные функции API Gateway:**

🔹 **Request Routing** - маршрутизация запросов к нужным сервисам  
🔹 **Authentication & Authorization** - проверка прав доступа  
🔹 **Rate Limiting** - ограничение частоты запросов  
🔹 **Load Balancing** - балансировка нагрузки  
🔹 **Response Aggregation** - агрегация ответов от нескольких сервисов  
🔹 **SSL Termination** - обработка SSL-соединений  
🔹 **Monitoring & Analytics** - сбор метрик и аналитики

### 🛠️ Практическая часть

#### Kong API Gateway Configuration

```yaml
# ═══════════════════════════════════════════════════════════
# kong-declarative-config.yaml
# ═══════════════════════════════════════════════════════════

_format_version: "3.0"

services:
  # ───────────────────────────────────────────────────────
  # User Service
  # ───────────────────────────────────────────────────────
  - name: user-service
    url: http://user-service.microservices.svc.cluster.local:80
    plugins:
      - name: rate-limiting
        config:
          minute: 100
          hour: 1000
      - name: key-auth
        config:
          key_names: ["X-API-Key"]
    routes:
      - name: users-route
        paths: ["/api/users"]
        methods: ["GET", "POST", "PUT", "DELETE"]

  # ───────────────────────────────────────────────────────
  # Order Service
  # ───────────────────────────────────────────────────────
  - name: order-service
    url: http://order-service.microservices.svc.cluster.local:80
    plugins:
      - name: rate-limiting
        config:
          minute: 200
          hour: 2000
      - name: jwt
        config:
          secret_is_base64: false
    routes:
      - name: orders-route
        paths: ["/api/orders"]
        methods: ["GET", "POST", "PUT", "DELETE"]

  # ───────────────────────────────────────────────────────
  # Product Service
  # ───────────────────────────────────────────────────────
  - name: product-service
    url: http://product-service.microservices.svc.cluster.local:80
    plugins:
      - name: cors
        config:
          origins: ["*"]
          methods: ["GET", "POST"]
      - name: response-transformer
        config:
          add:
            headers: ["X-Service:product-service"]
    routes:
      - name: products-route
        paths: ["/api/products"]
        methods: ["GET", "POST", "PUT", "DELETE"]

# ═══════════════════════════════════════════════════════════
# Global Plugins
# ═══════════════════════════════════════════════════════════
plugins:
  - name: prometheus
    config:
      per_consumer: true
  - name: zipkin
    config:
      http_endpoint: "http://zipkin:9411/api/v2/spans"
      sample_ratio: 0.1

# ═══════════════════════════════════════════════════════════
# Consumers (API Keys)
# ═══════════════════════════════════════════════════════════
consumers:
  - username: mobile-app
    keyauth_credentials:
      - key: mobile-app-key-123
  - username: web-app
    keyauth_credentials:
      - key: web-app-key-456
```

#### Custom API Gateway с Express.js

```javascript
// ═══════════════════════════════════════════════════════════
// custom-api-gateway.js
// ═══════════════════════════════════════════════════════════

const express = require('express');
const httpProxy = require('http-proxy-middleware');
const rateLimit = require('express-rate-limit');
const jwt = require('jsonwebtoken');
const redis = require('redis');

const app = express();
const redisClient = redis.createClient({ host: 'redis' });

// ───────────────────────────────────────────────────────
// Middleware Configuration
// ───────────────────────────────────────────────────────

// Rate limiting
const rateLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP',
  standardHeaders: true,
  legacyHeaders: false,
});

// JWT Authentication Middleware
const authenticateJWT = (req, res, next) => {
  const authHeader = req.headers.authorization;
  
  if (authHeader) {
    const token = authHeader.split(' ')[1];
    
    jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
      if (err) {
        return res.sendStatus(403);
      }
      req.user = user;
      next();
    });
  } else {
    res.sendStatus(401);
  }
};

// Request logging middleware
const requestLogger = (req, res, next) => {
  console.log(`${new Date().toISOString()} - ${req.method} ${req.path}`);
  next();
};

// ───────────────────────────────────────────────────────
// Service Discovery
// ───────────────────────────────────────────────────────

const services = {
  users: {
    target: 'http://user-service:3000',
    changeOrigin: true,
    pathRewrite: {
      '^/api/users': ''
    }
  },
  orders: {
    target: 'http://order-service:3000',
    changeOrigin: true,
    pathRewrite: {
      '^/api/orders': ''
    }
  },
  products: {
    target: 'http://product-service:3000',
    changeOrigin: true,
    pathRewrite: {
      '^/api/products': ''
    }
  }
};

// ───────────────────────────────────────────────────────
// Global Middleware
// ───────────────────────────────────────────────────────

app.use(express.json());
app.use(rateLimiter);
app.use(requestLogger);

// ───────────────────────────────────────────────────────
// Routes with Proxy
// ───────────────────────────────────────────────────────

// Public routes (no authentication required)
app.use('/api/products', httpProxy(services.products));

// Protected routes (authentication required)
app.use('/api/users', authenticateJWT, httpProxy(services.users));
app.use('/api/orders', authenticateJWT, httpProxy(services.orders));

// ───────────────────────────────────────────────────────
// Health Check and Metrics
// ───────────────────────────────────────────────────────

app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime()
  });
});

app.get('/metrics', async (req, res) => {
  // Implement metrics collection here
  const metrics = await collectMetrics();
  res.json(metrics);
});

// ───────────────────────────────────────────────────────
// Error Handling
// ───────────────────────────────────────────────────────

app.use((err, req, res, next) => {
  console.error('API Gateway Error:', err);
  res.status(500).json({
    error: 'Internal Server Error',
    timestamp: new Date().toISOString()
  });
});

// ───────────────────────────────────────────────────────
// Start Server
// ───────────────────────────────────────────────────────

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`API Gateway listening on port ${PORT}`);
});

async function collectMetrics() {
  // Implementation for metrics collection
  return {
    requests_total: 1000,
    response_time_avg: 150,
    error_rate: 0.05
  };
}
```

---

## 🔍 Глава 4: Service Discovery

### ▶️ Теоретическая основа

**Проблема Service Discovery:**

В микросервисной архитектуре сервисы динамически создаются и уничтожаются. Как один сервис может найти другой?

```
┌─────────────────────────────────────────────────────────────┐
│                    SERVICE DISCOVERY PATTERNS               │
│                                                             │
│  ┌──────────────────────────┐  ┌─────────────────────────┐   │
│  │    CLIENT-SIDE           │  │    SERVER-SIDE          │   │
│  │    DISCOVERY             │  │    DISCOVERY            │   │
│  │                          │  │                         │   │
│  │  ┌─────────┐             │  │  ┌─────────┐            │   │
│  │  │Service A│──┐          │  │  │Service A│            │   │
│  │  └─────────┘  │ ①Query   │  │  └─────────┘            │   │
│  │               ▼          │  │       │ ①Request        │   │
│  │  ┌─────────────────────┐ │  │       ▼                 │   │
│  │  │Service Registry     │ │  │  ┌─────────────────────┐│   │
│  │  │(Consul/Eureka)     │ │  │  │  Load Balancer      ││   │
│  │  └─────────────────────┘ │  │  │  (API Gateway)      ││   │
│  │               │          │  │  └─────────────────────┘│   │
│  │               │ ②Response│  │       │ ②Forward        │   │
│  │  ┌─────────┐  │          │  │       ▼                 │   │
│  │  │Service B│◄─┘          │  │  ┌─────────┐            │   │
│  │  └─────────┘             │  │  │Service B│            │   │
│  │               ③Direct    │  │  └─────────┘            │   │
│  │               Call       │  │                         │   │
│  └──────────────────────────┘  └─────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 🛠️ Практическая часть

#### Consul Configuration

```yaml
# ═══════════════════════════════════════════════════════════
# consul-deployment.yaml
# ═══════════════════════════════════════════════════════════

apiVersion: apps/v1
kind: Deployment
metadata:
  name: consul
  namespace: microservices
spec:
  replicas: 3
  selector:
    matchLabels:
      app: consul
  template:
    metadata:
      labels:
        app: consul
    spec:
      containers:
      - name: consul
        image: consul:1.15
        ports:
        - containerPort: 8500
          name: ui
        - containerPort: 8600
          name: dns
        env:
        - name: CONSUL_BIND_INTERFACE
          value: "eth0"
        command:
        - "consul"
        - "agent"
        - "-server"
        - "-bootstrap-expect=3"
        - "-ui"
        - "-bind=0.0.0.0"
        - "-client=0.0.0.0"
        - "-retry-join=consul"
        volumeMounts:
        - name: consul-data
          mountPath: /consul/data
      volumes:
      - name: consul-data
        emptyDir: {}

---
apiVersion: v1
kind: Service
metadata:
  name: consul
  namespace: microservices
spec:
  selector:
    app: consul
  ports:
  - port: 8500
    targetPort: 8500
    name: ui
  - port: 8600
    targetPort: 8600
    name: dns
    protocol: UDP
```

#### Service Registration в Node.js

```javascript
// ═══════════════════════════════════════════════════════════
// service-registry.js - Утилита для регистрации сервиса
// ═══════════════════════════════════════════════════════════

const consul = require('consul')({
  host: process.env.CONSUL_HOST || 'consul',
  port: process.env.CONSUL_PORT || 8500
});

class ServiceRegistry {
  constructor(serviceName, servicePort, serviceHost) {
    this.serviceName = serviceName;
    this.servicePort = servicePort;
    this.serviceHost = serviceHost || 'localhost';
    this.serviceId = `${serviceName}-${serviceHost}-${servicePort}`;
  }

  // ───────────────────────────────────────────────────────
  // Регистрация сервиса в Consul
  // ───────────────────────────────────────────────────────
  async register() {
    const serviceDefinition = {
      id: this.serviceId,
      name: this.serviceName,
      address: this.serviceHost,
      port: this.servicePort,
      tags: [
        'microservice',
        `version-${process.env.SERVICE_VERSION || '1.0.0'}`,
        `environment-${process.env.NODE_ENV || 'development'}`
      ],
      check: {
        http: `http://${this.serviceHost}:${this.servicePort}/health`,
        interval: '10s',
        timeout: '3s',
        deregistercriticalserviceafter: '30s'
      }
    };

    try {
      await consul.agent.service.register(serviceDefinition);
      console.log(`✅ Service ${this.serviceName} registered successfully`);
      
      // Graceful shutdown handling
      process.on('SIGINT', () => this.deregister());
      process.on('SIGTERM', () => this.deregister());
      
    } catch (error) {
      console.error('❌ Failed to register service:', error);
      throw error;
    }
  }

  // ───────────────────────────────────────────────────────
  // Удаление сервиса из реестра
  // ───────────────────────────────────────────────────────
  async deregister() {
    try {
      await consul.agent.service.deregister(this.serviceId);
      console.log(`✅ Service ${this.serviceName} deregistered successfully`);
      process.exit(0);
    } catch (error) {
      console.error('❌ Failed to deregister service:', error);
      process.exit(1);
    }
  }

  // ───────────────────────────────────────────────────────
  // Поиск других сервисов
  // ───────────────────────────────────────────────────────
  async discover(serviceName) {
    try {
      const result = await consul.health.service({
        service: serviceName,
        passing: true
      });

      const services = result.map(entry => ({
        id: entry.Service.ID,
        address: entry.Service.Address,
        port: entry.Service.Port,
        tags: entry.Service.Tags
      }));

      console.log(`🔍 Found ${services.length} instances of ${serviceName}`);
      return services;
    } catch (error) {
      console.error(`❌ Failed to discover service ${serviceName}:`, error);
      return [];
    }
  }

  // ───────────────────────────────────────────────────────
  // Load balancing (Round Robin)
  // ───────────────────────────────────────────────────────
  async getServiceInstance(serviceName) {
    const services = await this.discover(serviceName);
    
    if (services.length === 0) {
      throw new Error(`No healthy instances of ${serviceName} found`);
    }

    // Simple round-robin load balancing
    const index = Math.floor(Math.random() * services.length);
    return services[index];
  }
}

module.exports = ServiceRegistry;
```

#### HTTP Client с Service Discovery

```javascript
// ═══════════════════════════════════════════════════════════
// service-client.js - HTTP клиент с автоматическим discovery
// ═══════════════════════════════════════════════════════════

const axios = require('axios');
const ServiceRegistry = require('./service-registry');

class ServiceClient {
  constructor() {
    this.registry = new ServiceRegistry();
    this.cache = new Map(); // Cache для service instances
    this.cacheTimeout = 30000; // 30 seconds
  }

  // ───────────────────────────────────────────────────────
  // HTTP запрос к другому микросервису
  // ───────────────────────────────────────────────────────
  async request(serviceName, path, options = {}) {
    const instance = await this.getServiceInstance(serviceName);
    const url = `http://${instance.address}:${instance.port}${path}`;
    
    const requestOptions = {
      ...options,
      timeout: 5000, // 5 second timeout
      headers: {
        'Content-Type': 'application/json',
        'X-Request-ID': this.generateRequestId(),
        ...options.headers
      }
    };

    try {
      console.log(`🌐 Making request to ${serviceName}: ${url}`);
      const response = await axios(url, requestOptions);
      return response.data;
    } catch (error) {
      console.error(`❌ Request to ${serviceName} failed:`, error.message);
      
      // Invalidate cache on error
      this.cache.delete(serviceName);
      
      throw new Error(`Service ${serviceName} unavailable: ${error.message}`);
    }
  }

  // ───────────────────────────────────────────────────────
  // Получение instance сервиса с кэшированием
  // ───────────────────────────────────────────────────────
  async getServiceInstance(serviceName) {
    const cached = this.cache.get(serviceName);
    
    if (cached && Date.now() - cached.timestamp < this.cacheTimeout) {
      return cached.instance;
    }

    const instance = await this.registry.getServiceInstance(serviceName);
    
    this.cache.set(serviceName, {
      instance,
      timestamp: Date.now()
    });

    return instance;
  }

  // ───────────────────────────────────────────────────────
  // Convenience methods
  // ───────────────────────────────────────────────────────
  async get(serviceName, path, params = {}) {
    return this.request(serviceName, path, {
      method: 'GET',
      params
    });
  }

  async post(serviceName, path, data = {}) {
    return this.request(serviceName, path, {
      method: 'POST',
      data
    });
  }

  async put(serviceName, path, data = {}) {
    return this.request(serviceName, path, {
      method: 'PUT',
      data
    });
  }

  async delete(serviceName, path) {
    return this.request(serviceName, path, {
      method: 'DELETE'
    });
  }

  generateRequestId() {
    return `req_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}

module.exports = ServiceClient;
```

---

## 📊 Глава 5: Мониторинг и логирование

### ▶️ Теоретическая основа

**Observability Stack:**

```
┌─────────────────────────────────────────────────────────────┐
│                    OBSERVABILITY PILLARS                    │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   METRICS   │  │    LOGS     │  │   TRACES    │         │
│  │             │  │             │  │             │         │
│  │ Prometheus  │  │   ELK       │  │   Jaeger    │         │
│  │ Grafana     │  │ Fluentd     │  │  Zipkin     │         │
│  │             │  │             │  │             │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐ │
│  │                APPLICATION LAYER                      │ │
│  │                                                       │ │
│  │  [Service A] ──► [Service B] ──► [Service C]         │ │
│  │       │              │              │                │ │
│  │       ▼              ▼              ▼                │ │
│  │  ┌─────────────────────────────────────────────────┐ │ │
│  │  │            INSTRUMENTATION                      │ │ │
│  │  │  • Metrics collection                           │ │ │
│  │  │  • Structured logging                           │ │ │
│  │  │  • Distributed tracing                          │ │ │
│  │  └─────────────────────────────────────────────────┘ │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 🛠️ Практическая часть

#### Prometheus + Grafana Setup

```yaml
# ═══════════════════════════════════════════════════════════
# monitoring-stack.yaml
# ═══════════════════════════════════════════════════════════

apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s
    
    rule_files:
      - "alert_rules.yml"
    
    alerting:
      alertmanagers:
        - static_configs:
            - targets:
              - alertmanager:9093
    
    scrape_configs:
      # ─────────────────────────────────────────────────────
      # Prometheus self-monitoring
      # ─────────────────────────────────────────────────────
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']
      
      # ─────────────────────────────────────────────────────
      # Kubernetes API server
      # ─────────────────────────────────────────────────────
      - job_name: 'kubernetes-apiservers'
        kubernetes_sd_configs:
        - role: endpoints
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        relabel_configs:
        - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
          action: keep
          regex: default;kubernetes;https
      
      # ─────────────────────────────────────────────────────
      # Microservices discovery
      # ─────────────────────────────────────────────────────
      - job_name: 'microservices'
        kubernetes_sd_configs:
        - role: endpoints
        relabel_configs:
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
          action: keep
          regex: true
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
          action: replace
          target_label: __metrics_path__
          regex: (.+)
        - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
          action: replace
          regex: ([^:]+)(?::\d+)?;(\d+)
          replacement: $1:$2
          target_label: __address__

  alert_rules.yml: |
    groups:
    - name: microservices
      rules:
      - alert: ServiceDown
        expr: up == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Service {{ $labels.instance }} is down"
          description: "{{ $labels.instance }} has been down for more than 2 minutes."
      
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} for {{ $labels.instance }}"

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:v2.40.0
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: config
          mountPath: /etc/prometheus
        - name: storage
          mountPath: /prometheus
        args:
          - '--config.file=/etc/prometheus/prometheus.yml'
          - '--storage.tsdb.path=/prometheus'
          - '--web.console.libraries=/etc/prometheus/console_libraries'
          - '--web.console.templates=/etc/prometheus/consoles'
          - '--storage.tsdb.retention.time=15d'
          - '--web.enable-lifecycle'
      volumes:
      - name: config
        configMap:
          name: prometheus-config
      - name: storage
        emptyDir: {}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:9.0.0
        ports:
        - containerPort: 3000
        env:
        - name: GF_SECURITY_ADMIN_PASSWORD
          value: "admin123"
        volumeMounts:
        - name: grafana-storage
          mountPath: /var/lib/grafana
      volumes:
      - name: grafana-storage
        emptyDir: {}
```

#### Application Instrumentation

```javascript
// ═══════════════════════════════════════════════════════════
// metrics.js - Prometheus metrics для Node.js приложения
// ═══════════════════════════════════════════════════════════

const promClient = require('prom-client');

// ───────────────────────────────────────────────────────
// Metrics Configuration
// ───────────────────────────────────────────────────────

// Enable default metrics collection
promClient.collectDefaultMetrics({
  timeout: 5000,
  gcDurationBuckets: [0.001, 0.01, 0.1, 1, 2, 5]
});

// Custom metrics
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.3, 0.5, 0.7, 1, 3, 5, 7, 10]
});

const httpRequestTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

const activeConnections = new promClient.Gauge({
  name: 'active_connections',
  help: 'Number of active connections',
  labelNames: ['type']
});

const businessMetrics = new promClient.Counter({
  name: 'business_events_total',
  help: 'Total number of business events',
  labelNames: ['event_type', 'status']
});

// ───────────────────────────────────────────────────────
// Middleware для Express.js
// ───────────────────────────────────────────────────────

function prometheusMiddleware() {
  return (req, res, next) => {
    const start = Date.now();
    
    // Increment active connections
    activeConnections.inc({ type: 'http' });
    
    res.on('finish', () => {
      const duration = (Date.now() - start) / 1000;
      const route = req.route ? req.route.path : req.path;
      
      // Record metrics
      httpRequestDuration
        .labels(req.method, route, res.statusCode)
        .observe(duration);
      
      httpRequestTotal
        .labels(req.method, route, res.statusCode)
        .inc();
      
      // Decrement active connections
      activeConnections.dec({ type: 'http' });
    });
    
    next();
  };
}

// ───────────────────────────────────────────────────────
// Business Metrics Helpers
// ───────────────────────────────────────────────────────

function recordBusinessEvent(eventType, status = 'success') {
  businessMetrics.labels(eventType, status).inc();
}

// ───────────────────────────────────────────────────────
// Metrics Endpoint
// ───────────────────────────────────────────────────────

function getMetrics() {
  return promClient.register.metrics();
}

module.exports = {
  prometheusMiddleware,
  recordBusinessEvent,
  getMetrics,
  httpRequestDuration,
  httpRequestTotal,
  activeConnections,
  businessMetrics
};
```

#### Structured Logging

```javascript
// ═══════════════════════════════════════════════════════════
// logger.js - Structured logging с Winston
// ═══════════════════════════════════════════════════════════

const winston = require('winston');
const { v4: uuidv4 } = require('uuid');

// ───────────────────────────────────────────────────────
// Logger Configuration
// ───────────────────────────────────────────────────────

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json(),
    winston.format.printf((info) => {
      return JSON.stringify({
        timestamp: info.timestamp,
        level: info.level,
        message: info.message,
        service: process.env.SERVICE_NAME || 'unknown-service',
        version: process.env.SERVICE_VERSION || '1.0.0',
        traceId: info.traceId,
        spanId: info.spanId,
        userId: info.userId,
        correlationId: info.correlationId,
        ...info.metadata
      });
    })
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({
      filename: 'logs/error.log',
      level: 'error'
    }),
    new winston.transports.File({
      filename: 'logs/combined.log'
    })
  ]
});

// ───────────────────────────────────────────────────────
// Context Middleware для Express
// ───────────────────────────────────────────────────────

function correlationMiddleware() {
  return (req, res, next) => {
    // Generate or extract correlation ID
    req.correlationId = req.headers['x-correlation-id'] || uuidv4();
    req.traceId = req.headers['x-trace-id'] || uuidv4();
    
    // Add to response headers
    res.setHeader('x-correlation-id', req.correlationId);
    res.setHeader('x-trace-id', req.traceId);
    
    // Create child logger with context
    req.logger = logger.child({
      correlationId: req.correlationId,
      traceId: req.traceId,
      userId: req.user?.id,
      method: req.method,
      url: req.url,
      userAgent: req.headers['user-agent']
    });
    
    // Log incoming request
    req.logger.info('Incoming request', {
      method: req.method,
      url: req.url,
      headers: req.headers,
      body: req.body
    });
    
    next();
  };
}

// ───────────────────────────────────────────────────────
// Request/Response Logging
// ───────────────────────────────────────────────────────

function requestLoggingMiddleware() {
  return (req, res, next) => {
    const start = Date.now();
    
    // Capture original json method
    const originalJson = res.json;
    res.json = function(data) {
      // Log response
      req.logger.info('Outgoing response', {
        statusCode: res.statusCode,
        duration: Date.now() - start,
        responseSize: JSON.stringify(data).length
      });
      
      return originalJson.call(this, data);
    };
    
    res.on('finish', () => {
      const duration = Date.now() - start;
      
      if (res.statusCode >= 400) {
        req.logger.error('Request failed', {
          statusCode: res.statusCode,
          duration,
          error: res.statusMessage
        });
      } else {
        req.logger.info('Request completed', {
          statusCode: res.statusCode,
          duration
        });
      }
    });
    
    next();
  };
}

// ───────────────────────────────────────────────────────
// Error Logging
// ───────────────────────────────────────────────────────

function errorHandler() {
  return (err, req, res, next) => {
    const logger = req.logger || winston;
    
    logger.error('Unhandled error', {
      error: {
        message: err.message,
        stack: err.stack,
        name: err.name
      },
      request: {
        method: req.method,
        url: req.url,
        headers: req.headers,
        body: req.body
      }
    });
    
    res.status(500).json({
      error: 'Internal Server Error',
      correlationId: req.correlationId
    });
  };
}

module.exports = {
  logger,
  correlationMiddleware,
  requestLoggingMiddleware,
  errorHandler
};
```

---

## 🚀 Глава 6: CI/CD для микросервисов

### ▶️ Теоретическая основа

**CI/CD Pipeline для микросервисов:**

```
┌─────────────────────────────────────────────────────────────┐
│                        CI/CD PIPELINE                       │
│                                                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐       │
│  │  Code   │  │ Build & │  │  Test   │  │ Deploy  │       │
│  │ Commit  │─►│  Test   │─►│  Stage  │─►│   Prod  │       │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘       │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐ │
│  │                 PER-SERVICE PIPELINE                  │ │
│  │                                                       │ │
│  │  user-service/     order-service/    product-service/│ │
│  │  ├── Dockerfile   ├── Dockerfile    ├── Dockerfile   │ │
│  │  ├── .gitlab-ci   ├── .github/      ├── Jenkinsfile │ │
│  │  └── k8s/         └── k8s/          └── k8s/         │ │
│  │                                                       │ │
│  │  Independent deployments, separate versions          │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 🛠️ Практическая часть

#### GitHub Actions Workflow

```yaml
# ═══════════════════════════════════════════════════════════
# .github/workflows/microservice-ci-cd.yml
# ═══════════════════════════════════════════════════════════

name: Microservice CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ───────────────────────────────────────────────────────
  # Detect Changes (для mono-repo)
  # ───────────────────────────────────────────────────────
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      user-service: ${{ steps.changes.outputs.user-service }}
      order-service: ${{ steps.changes.outputs.order-service }}
      product-service: ${{ steps.changes.outputs.product-service }}
    steps:
    - uses: actions/checkout@v3
    - uses: dorny/paths-filter@v2
      id: changes
      with:
        filters: |
          user-service:
            - 'services/user-service/**'
          order-service:
            - 'services/order-service/**'
          product-service:
            - 'services/product-service/**'

  # ───────────────────────────────────────────────────────
  # User Service Pipeline
  # ───────────────────────────────────────────────────────
  user-service:
    needs: detect-changes
    if: ${{ needs.detect-changes.outputs.user-service == 'true' }}
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        cache-dependency-path: services/user-service/package-lock.json

    # ─────────────────────────────────────────────────────
    # Install dependencies and run tests
    # ─────────────────────────────────────────────────────
    - name: Install dependencies
      working-directory: ./services/user-service
      run: npm ci

    - name: Run linting
      working-directory: ./services/user-service
      run: npm run lint

    - name: Run unit tests
      working-directory: ./services/user-service
      run: npm run test:unit

    - name: Run integration tests
      working-directory: ./services/user-service
      run: npm run test:integration

    # ─────────────────────────────────────────────────────
    # Security scanning
    # ─────────────────────────────────────────────────────
    - name: Run security audit
      working-directory: ./services/user-service
      run: npm audit --audit-level high

    - name: Run Snyk security scan
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      with:
        args: --severity-threshold=high
        command: test

    # ─────────────────────────────────────────────────────
    # Build and push Docker image
    # ─────────────────────────────────────────────────────
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Log in to Container Registry
      uses: docker/login-action@v2
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v4
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/user-service
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha,prefix={{branch}}-
          type=raw,value=latest,enable={{is_default_branch}}

    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: ./services/user-service
        platforms: linux/amd64,linux/arm64
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

    # ─────────────────────────────────────────────────────
    # Deploy to staging
    # ─────────────────────────────────────────────────────
    - name: Deploy to staging
      if: github.ref == 'refs/heads/develop'
      uses: azure/k8s-deploy@v1
      with:
        manifests: |
          services/user-service/k8s/staging/
        images: |
          ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/user-service:${{ github.sha }}
        kubeconfig: ${{ secrets.KUBE_CONFIG_STAGING }}

    # ─────────────────────────────────────────────────────
    # Deploy to production
    # ─────────────────────────────────────────────────────
    - name: Deploy to production
      if: github.ref == 'refs/heads/main'
      uses: azure/k8s-deploy@v1
      with:
        manifests: |
          services/user-service/k8s/production/
        images: |
          ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/user-service:${{ github.sha }}
        kubeconfig: ${{ secrets.KUBE_CONFIG_PROD }}

  # ───────────────────────────────────────────────────────
  # End-to-End Tests
  # ───────────────────────────────────────────────────────
  e2e-tests:
    needs: [user-service, order-service, product-service]
    if: always() && (needs.user-service.result == 'success' || needs.order-service.result == 'success' || needs.product-service.result == 'success')
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        cache-dependency-path: e2e-tests/package-lock.json

    - name: Install dependencies
      working-directory: ./e2e-tests
      run: npm ci

    - name: Run E2E tests against staging
      working-directory: ./e2e-tests
      env:
        API_BASE_URL: https://staging-api.mycompany.com
        TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
        TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
      run: npm run test:e2e

    - name: Upload test results
      if: always()
      uses: actions/upload-artifact@v3
      with:
        name: e2e-test-results
        path: e2e-tests/reports/
```

#### GitOps с ArgoCD

```yaml
# ═══════════════════════════════════════════════════════════
# argocd-application.yaml - GitOps deployment
# ═══════════════════════════════════════════════════════════

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: microservices-app
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  
  # ───────────────────────────────────────────────────────
  # Source configuration
  # ───────────────────────────────────────────────────────
  source:
    repoURL: https://github.com/mycompany/microservices-config
    targetRevision: main
    path: environments/production
    
    # Helm configuration
    helm:
      valueFiles:
        - values.yaml
        - values-production.yaml
      parameters:
        - name: userService.image.tag
          value: "v1.2.3"
        - name: orderService.image.tag
          value: "v1.1.5"
        - name: productService.image.tag
          value: "v2.0.1"

  # ───────────────────────────────────────────────────────
  # Destination configuration
  # ───────────────────────────────────────────────────────
  destination:
    server: https://kubernetes.default.svc
    namespace: microservices

  # ───────────────────────────────────────────────────────
  # Sync policy
  # ───────────────────────────────────────────────────────
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m

  # ───────────────────────────────────────────────────────
  # Health checks
  # ───────────────────────────────────────────────────────
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
  
  revisionHistoryLimit: 10

---
# ═══════════════════════════════════════════════════════════
# Application Project для изоляции
# ═══════════════════════════════════════════════════════════

apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: microservices-project
  namespace: argocd
spec:
  description: Microservices project
  
  # ───────────────────────────────────────────────────────
  # Allowed sources
  # ───────────────────────────────────────────────────────
  sourceRepos:
    - 'https://github.com/mycompany/microservices-config'
    - 'https://charts.helm.sh/stable'
  
  # ───────────────────────────────────────────────────────
  # Allowed destinations
  # ───────────────────────────────────────────────────────
  destinations:
    - namespace: microservices
      server: https://kubernetes.default.svc
    - namespace: monitoring
      server: https://kubernetes.default.svc
  
  # ───────────────────────────────────────────────────────
  # Cluster resource whitelist
  # ───────────────────────────────────────────────────────
  clusterResourceWhitelist:
    - group: ''
      kind: Namespace
    - group: rbac.authorization.k8s.io
      kind: ClusterRole
    - group: rbac.authorization.k8s.io
      kind: ClusterRoleBinding
  
  # ───────────────────────────────────────────────────────
  # Namespace resource whitelist
  # ───────────────────────────────────────────────────────
  namespaceResourceWhitelist:
    - group: ''
      kind: Service
    - group: ''
      kind: ConfigMap
    - group: ''
      kind: Secret
    - group: apps
      kind: Deployment
    - group: networking.k8s.io
      kind: Ingress

  roles:
    - name: developer
      description: Developer role
      policies:
        - p, proj:microservices-project:developer, applications, get, microservices-project/*, allow
        - p, proj:microservices-project:developer, applications, sync, microservices-project/*, allow
      groups:
        - microservices-developers
```

#### Deployment Strategies

```yaml
# ═══════════════════════════════════════════════════════════
# canary-deployment.yaml - Canary deployment с Flagger
# ═══════════════════════════════════════════════════════════

apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: user-service-canary
  namespace: microservices
spec:
  # ───────────────────────────────────────────────────────
  # Deployment reference
  # ───────────────────────────────────────────────────────
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: user-service

  # ───────────────────────────────────────────────────────
  # HPA reference для автомасштабирования
  # ───────────────────────────────────────────────────────
  autoscalerRef:
    apiVersion: autoscaling/v2
    kind: HorizontalPodAutoscaler
    name: user-service

  # ───────────────────────────────────────────────────────
  # Service configuration
  # ───────────────────────────────────────────────────────
  service:
    port: 80
    targetPort: 3000
    gateways:
    - public-gateway.istio-system.svc.cluster.local
    hosts:
    - api.mycompany.com
    trafficPolicy:
      tls:
        mode: DISABLE

  # ───────────────────────────────────────────────────────
  # Canary analysis configuration
  # ───────────────────────────────────────────────────────
  analysis:
    # ─────────────────────────────────────────────────────
    # Analysis timing
    # ─────────────────────────────────────────────────────
    interval: 1m
    threshold: 10
    maxWeight: 50
    stepWeight: 5
    
    # ─────────────────────────────────────────────────────
    # Success/failure criteria
    # ─────────────────────────────────────────────────────
    metrics:
    - name: request-success-rate
      thresholdRange:
        min: 99
      interval: 1m
    - name: request-duration
      thresholdRange:
        max: 500
      interval: 1m
    - name: error-rate
      thresholdRange:
        max: 1
      interval: 1m
    
    # ─────────────────────────────────────────────────────
    # Webhook tests
    # ─────────────────────────────────────────────────────
    webhooks:
    - name: integration-tests
      url: http://test-runner.microservices.svc.cluster.local/
      timeout: 30s
      metadata:
        type: bash
        cmd: "curl -sd 'test' http://user-service-canary/api/health"
    
    - name: load-test
      url: http://test-runner.microservices.svc.cluster.local/
      timeout: 5m
      metadata:
        type: bash
        cmd: "hey -z 5m -q 10 -c 2 http://user-service-canary/api/users"

---
# ═══════════════════════════════════════════════════════════
# blue-green-deployment.yaml - Blue-Green с Argo Rollouts
# ═══════════════════════════════════════════════════════════

apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: order-service-rollout
  namespace: microservices
spec:
  replicas: 5
  strategy:
    blueGreen:
      # ─────────────────────────────────────────────────────
      # Traffic routing
      # ─────────────────────────────────────────────────────
      activeService: order-service-active
      previewService: order-service-preview
      
      # ─────────────────────────────────────────────────────
      # Promotion timing
      # ─────────────────────────────────────────────────────
      autoPromotionEnabled: false
      scaleDownDelaySeconds: 30
      prePromotionAnalysis:
        templates:
        - templateName: order-service-analysis
        args:
        - name: service-name
          value: order-service-preview
      
      # ─────────────────────────────────────────────────────
      # Post-promotion validation
      # ─────────────────────────────────────────────────────
      postPromotionAnalysis:
        templates:
        - templateName: order-service-analysis
        args:
        - name: service-name
          value: order-service-active

  selector:
    matchLabels:
      app: order-service

  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
      - name: order-service
        image: myregistry/order-service:latest
        ports:
        - containerPort: 3000
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"

---
# ═══════════════════════════════════════════════════════════
# Analysis Template для тестирования
# ═══════════════════════════════════════════════════════════

apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: order-service-analysis
  namespace: microservices
spec:
  args:
  - name: service-name
  
  metrics:
  # ─────────────────────────────────────────────────────
  # Success rate analysis
  # ─────────────────────────────────────────────────────
  - name: success-rate
    interval: 60s
    count: 5
    successCondition: result[0] >= 0.95
    provider:
      prometheus:
        address: http://prometheus.monitoring.svc.cluster.local:9090
        query: |
          sum(rate(http_requests_total{service="{{args.service-name}}",status!~"5.."}[5m])) /
          sum(rate(http_requests_total{service="{{args.service-name}}"}[5m]))
  
  # ─────────────────────────────────────────────────────
  # Response time analysis
  # ─────────────────────────────────────────────────────
  - name: avg-response-time
    interval: 60s
    count: 5
    successCondition: result[0] <= 300
    provider:
      prometheus:
        address: http://prometheus.monitoring.svc.cluster.local:9090
        query: |
          histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket{service="{{args.service-name}}"}[5m])) by (le)
          ) * 1000
```

---

## 🎯 Практические задания

### 📝 Задание 1: Контейнеризация приложения

**Цель:** Создать production-ready Docker образ для микросервиса

**Задачи:**
1. Создайте Dockerfile с multi-stage build
2. Оптимизируйте размер образа (цель: <100MB для Node.js app)
3. Добавьте health check
4. Настройте non-root пользователя
5. Создайте docker-compose.yml для локальной разработки

**Критерии оценки:**
- ✅ Образ собирается без ошибок
- ✅ Размер образа оптимален
- ✅ Health check работает корректно
- ✅ Все сервисы запускаются через docker-compose

### 📝 Задание 2: Kubernetes Deployment

**Цель:** Развернуть микросервис в Kubernetes с полной конфигурацией

**Задачи:**
1. Создайте Deployment с 3 репликами
2. Настройте Service для внутренней коммуникации
3. Добавьте ConfigMap и Secret
4. Настройте Ingress для внешнего доступа
5. Реализуйте HorizontalPodAutoscaler

**Критерии оценки:**
- ✅ Pod'ы успешно запускаются
- ✅ Service доступен внутри кластера
- ✅ Ingress корректно маршрутизирует трафик
- ✅ Автомасштабирование работает

### 📝 Задание 3: Мониторинг Setup

**Цель:** Настроить comprehensive мониторинг для микросервисов

**Задачи:**
1. Развернуть Prometheus и Grafana
2. Добавить instrumentation в приложение
3. Создать dashboard в Grafana
4. Настроить алерты
5. Реализовать structured logging

**Критерии оценки:**
- ✅ Метрики собираются корректно
- ✅ Dashboard отображает ключевые показатели
- ✅ Алерты настроены правильно
- ✅ Логи структурированы и коррелированы

### 📝 Задание 4: CI/CD Pipeline

**Цель:** Создать полноценный CI/CD pipeline

**Задачи:**
1. Настройте GitHub Actions workflow
2. Добавьте автоматические тесты
3. Реализуйте security scanning
4. Настройте автоматический деплой в staging
5. Добавьте manual approval для production

**Критерии оценки:**
- ✅ Pipeline выполняется без ошибок
- ✅ Тесты проходят успешно
- ✅ Security scan не находит критических уязвимостей
- ✅ Деплой работает корректно

---

## ✅ Чек-лист готовности

### 🐳 Docker & Контейнеризация
- [ ] Понимаю принципы контейнеризации
- [ ] Могу создать оптимизированный Dockerfile
- [ ] Умею работать с multi-stage builds
- [ ] Знаю best practices для Docker security
- [ ] Могу настроить docker-compose для разработки

### ☸️ Kubernetes
- [ ] Понимаю архитектуру Kubernetes
- [ ] Могу создать Deployment, Service, ConfigMap
- [ ] Умею работать с Ingress и networking
- [ ] Знаю принципы resource management
- [ ] Понимаю, как работают health checks

### 🚪 API Gateway
- [ ] Понимаю роль API Gateway в микросервисах
- [ ] Могу настроить Kong или custom gateway
- [ ] Умею реализовать authentication/authorization
- [ ] Знаю паттерны rate limiting и load balancing
- [ ] Понимаю принципы SSL termination

### 🔍 Service Discovery
- [ ] Понимаю проблемы service discovery
- [ ] Знаю различия между client-side и server-side discovery
- [ ] Могу настроить Consul или альтернативы
- [ ] Умею реализовать health checks
- [ ] Понимаю принципы load balancing

### 📊 Мониторинг и логирование
- [ ] Знаю three pillars of observability
- [ ] Могу настроить Prometheus и Grafana
- [ ] Умею добавить instrumentation в приложение
- [ ] Понимаю принципы structured logging
- [ ] Знаю, как настроить алерты

### 🚀 CI/CD
- [ ] Понимаю принципы GitOps
- [ ] Могу создать CI/CD pipeline
- [ ] Знаю deployment strategies (blue-green, canary)
- [ ] Умею настроить automated testing
- [ ] Понимаю принципы security scanning

---

## 📚 Дополнительные ресурсы

### 📖 Книги
- "Kubernetes: Up and Running" by Kelsey Hightower
- "Docker: Deep Dive" by Nigel Poulton
- "Prometheus: Up & Running" by Brian Brazil

### 🎥 Курсы
- Kubernetes Official Training
- Docker Certified Associate
- Prometheus and Grafana courses на Udemy

### 🛠️ Инструменты для практики
- **Minikube** - локальный Kubernetes
- **Kind** - Kubernetes in Docker
- **K3s** - легковесный Kubernetes
- **Docker Desktop** - полноценная среда разработки

### 🌐 Онлайн ресурсы
- Kubernetes Documentation
- Docker Official Documentation  
- Prometheus Documentation
- CNCF Landscape

═══════════════════════════════════════════════════════════════

## 🎉 Заключение блока

После завершения этого блока вы будете обладать всеми необходимыми навыками для работы с технологической инфраструктурой микросервисов. Вы сможете:

🔹 **Контейнеризировать** любое приложение с best practices  
🔹 **Развертывать** сервисы в Kubernetes кластере  
🔹 **Настраивать** API Gateway для маршрутизации трафика  
🔹 **Реализовывать** service discovery для динамического окружения  
🔹 **Мониторить** состояние системы и диагностировать проблемы  
🔹 **Автоматизировать** процессы развертывания через CI/CD

Переходим к следующему блоку: **"Взаимодействие между сервисами"** 🚀