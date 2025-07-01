# DevOps & Trunk-Based Development: От новичка до эксперта

## 🎯 Что вы получите

**За 24 недели:** Превратитесь из backend-разработчика в DevOps-эксперта с глубоким пониманием Trunk-Based Development.

**Результат:** Способность построить production-ready CI/CD pipeline с TBD практиками для любого проекта.

---

## 📋 Структура курса

```
Неделя 1-3   → DevOps культура + Git mastery
Неделя 4-5   → Trunk-Based Development
Неделя 6-8   → Continuous Integration  
Неделя 9-12  → Continuous Delivery/Deployment
Неделя 13-15 → Infrastructure & Operations
Неделя 16-17 → Monitoring & Observability
Неделя 18-19 → Security (DevSecOps)
Неделя 20-21 → Performance & Scaling
Неделя 22-23 → Advanced Practices
Неделя 24    → Capstone Project
```

---

# 🚀 Фаза 1: DevOps Фундаменты (3 недели)

## Неделя 1: DevOps Культура

### 💡 Концепция
DevOps — это не инструменты, это **культура быстрой доставки качественного ПО**.

**Основная идея:** Разрушить стену между разработкой и операциями.

```
БЫЛО:                    СТАЛО:
Dev → QA → Ops          Dev ↔ DevOps ↔ Ops
 ↓     ↓    ↓              ↓
Slow  Bugs  Fire         Fast + Reliable
```

### 🎯 DORA Метрики (ваш компас)

1. **Deployment Frequency** — как часто деплоим
2. **Lead Time** — от коммита до продакшена  
3. **MTTR** — время восстановления после сбоя
4. **Change Failure Rate** — % плохих деплоев

### ✅ Практика: Baseline Measurement

```bash
# Узнайте текущие метрики вашей команды
git log --oneline --since="1 month ago" | wc -l  # Коммитов в месяц
git log --pretty=format:"%h %ad" --date=short | head -10  # Частота
```

**Цель:** Элитные команды деплоят **несколько раз в день** с **MTTR < 1 часа**.

---

## Неделя 2-3: Git Mastery

### 💡 Концепция
Git — это не просто система контроля версий, это **основа всей DevOps философии**.

### 🛠 Продвинутые практики

#### Semantic Versioning
```
1.2.3
│ │ └── PATCH (bug fixes)
│ └──── MINOR (new features)  
└────── MAJOR (breaking changes)
```

#### Conventional Commits
```bash
feat: add user authentication
fix: resolve memory leak in payment service
docs: update API documentation
test: add integration tests for orders
```

#### Git Hooks для автоматизации
```bash
# .git/hooks/pre-commit
#!/bin/sh
npm test && npm run lint
```

### ✅ Чек-лист Git Excellence

- [ ] Защищена главная ветка (branch protection)
- [ ] Настроены pre-commit hooks
- [ ] Все коммиты следуют конвенции
- [ ] Code review обязателен для merge

---

# 🌿 Фаза 2: Trunk-Based Development (2 недели)

## Неделя 4: TBD Философия

### 💡 Концепция
**Trunk-Based Development** — все разработчики работают с одной главной веткой (trunk/main).

#### Почему TBD побеждает Git Flow?

```
Git Flow (медленно):          TBD (быстро):
                             
main ──────────────          main ●─●─●─●─●─●
 │                            │   │ │ │ │ │
 └─ feature ───┐              └─●─┘ │ │ │ │  
   └─ hotfix ──┘                └─●─┘ │ │
                                  └─●─┘ │
Merge Hell                          └─●─┘
```

### 🎯 Ключевые принципы

1. **Короткие ветки** — жизнь < 24 часов
2. **Маленькие коммиты** — 1-2 файла за раз
3. **Feature flags** — код в продакшене, фича выключена
4. **Быстрая интеграция** — merge минимум раз в день

### 🛠 Feature Flags Implementation

```javascript
// Простейший feature flag
const features = {
  newPaymentFlow: process.env.FEATURE_NEW_PAYMENT === 'true'
};

function processPayment(order) {
  if (features.newPaymentFlow) {
    return newPaymentService.process(order);
  }
  return legacyPaymentService.process(order);
}
```

### ✅ TBD Workflow

```
1. git pull origin main
2. Создать мини-фичу (2-4 часа работы)
3. Написать тесты
4. git commit -m "feat: add payment validation"
5. git push + создать PR
6. Code review + merge
7. Повторить
```

---

## Неделя 5: Качество в TBD

### 💡 Концепция
В TBD **качество обеспечивается автоматизацией**, а не долгими ветками.

### 🛠 Pre-commit Pipeline

```yaml
# .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: tests
        name: Run tests
        entry: npm test
        language: system
      - id: lint
        name: Code linting
        entry: npm run lint
        language: system
```

### 🔍 Code Review для TBD

**Правило:** PR должен быть reviewable за **15 минут**.

```
✅ Хороший TBD PR:
- 50-200 строк кода
- 1 фича/фикс
- Понятный title
- Тесты included

❌ Плохой PR:
- 500+ строк
- 3 фичи сразу  
- "Fix stuff"
- Без тестов
```

### ✅ Definition of Done

- [ ] Тесты проходят
- [ ] Code coverage не упал
- [ ] Static analysis passed
- [ ] Feature flag готов (если нужен)
- [ ] Backward compatibility сохранена

---

# 🔄 Фаза 3: Continuous Integration (3 недели)

## Неделя 6: CI Основы

### 💡 Концепция
**Continuous Integration** — каждый коммит автоматически проверяется и собирается.

**Цель:** Найти проблемы за **минуты**, а не дни.

```
Developer commits → CI triggers → Tests run → Feedback in 5min
```

### 🛠 Базовый CI Pipeline

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm test
      - run: npm run build
```

### 📊 Test Pyramid

```
    /\     E2E Tests (few, slow, expensive)
   /  \    
  /____\   Integration Tests (some, medium)
 /      \  
/________\  Unit Tests (many, fast, cheap)
```

### ✅ CI Best Practices

- [ ] Тесты выполняются < 10 минут
- [ ] Fail fast при первой ошибке
- [ ] Параллельное выполнение тестов
- [ ] Кеширование dependencies
- [ ] Отчеты о coverage

---

## Неделя 7: Advanced CI

### 💡 Концепция
**Оптимизация CI** — это инвестиция в скорость команды.

### 🛠 Test Containers для Integration Tests

```javascript
// Изолированные тесты с реальной БД
const { GenericContainer } = require('testcontainers');

describe('User Service', () => {
  let container, db;
  
  beforeAll(async () => {
    container = await new GenericContainer('postgres:13')
      .withExposedPorts(5432)
      .withEnv('POSTGRES_PASSWORD', 'test')
      .start();
    
    db = await connectToDb(container.getMappedPort(5432));
  });
  
  test('creates user', async () => {
    const user = await userService.create({name: 'John'});
    expect(user.id).toBeDefined();
  });
});
```

### ⚡ Optimization Strategies

```yaml
# Параллельные job'ы
jobs:
  lint:
    runs-on: ubuntu-latest
    steps: [lint code]
  
  test-unit:
    runs-on: ubuntu-latest  
    steps: [run unit tests]
    
  test-integration:
    runs-on: ubuntu-latest
    steps: [run integration tests]
```

### 🏗 Monorepo CI

```
repo/
├── service-a/     → Trigger only if service-a changed
├── service-b/     → Trigger only if service-b changed  
└── shared/        → Trigger all if shared changed
```

---

## Неделя 8: CI + TBD Integration

### 💡 Концепция
CI в TBD должен обеспечивать **мгновенную обратную связь** для коротких веток.

### 🛠 Smart CI для TBD

```yaml
# Быстрая feedback loop
on:
  push:
    branches: [main]
  pull_request:

jobs:
  quick-feedback:  # 2-3 минуты
    steps:
      - run: npm run lint
      - run: npm run test:unit
      
  full-pipeline:   # 8-10 минут  
    needs: quick-feedback
    if: github.ref == 'refs/heads/main'
    steps:
      - run: npm run test:integration
      - run: npm run test:e2e
      - run: npm run build
```

### 🚨 Automated Rollback

```javascript
// Health check после деплоя
const healthCheck = async () => {
  const response = await fetch('/health');
  if (!response.ok) {
    throw new Error('Health check failed');
  }
};

// Если health check падает → rollback
```

### ✅ TBD CI Checklist

- [ ] PR checks завершаются за < 5 минут
- [ ] Main branch всегда deployable
- [ ] Автоматический rollback при сбоях
- [ ] Feature flags интегрированы в CI

---

# 🚀 Фаза 4: Continuous Delivery (4 недели)

## Неделя 9-10: CD Основы

### 💡 Концепция

**Continuous Delivery** — код всегда готов к деплою одним кликом.
**Continuous Deployment** — код деплоится автоматически.

```
CD Pipeline:
Code → Build → Test → Deploy to Staging → Deploy to Prod
                ↓
           All Automated
```

### 🛠 Environment Promotion

```yaml
# Multi-environment pipeline
stages:
  - build
  - deploy-dev
  - deploy-staging  
  - deploy-prod

deploy-dev:
  stage: deploy-dev
  script: deploy.sh dev
  
deploy-staging:
  stage: deploy-staging
  when: manual  # Требует подтверждения
  
deploy-prod:
  stage: deploy-prod
  when: manual
  only: [main]
```

### 📋 Deployment Strategies

#### Blue-Green Deployment
```
Blue (current) ←── Traffic ──→ Green (new)
     v1.0                        v1.1
     
1. Deploy v1.1 to Green
2. Test Green environment  
3. Switch traffic: Blue → Green
4. Keep Blue as rollback option
```

#### Canary Deployment
```
Traffic split:
90% → Stable version (v1.0)
10% → Canary version (v1.1)

Monitor metrics → Gradually increase canary %
```

### 🛠 Infrastructure as Code

```hcl
# terraform/main.tf
resource "aws_instance" "app" {
  ami           = "ami-12345"
  instance_type = "t3.micro"
  
  tags = {
    Name = "app-${var.environment}"
    Environment = var.environment
  }
}
```

---

## Неделя 11: Advanced Deployment

### 🛠 Database Migrations в CD

```sql
-- V1__create_users.sql (необратимая миграция)
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) NOT NULL
);

-- V2__add_name_column.sql (обратимо совместимая)
ALTER TABLE users ADD COLUMN name VARCHAR(255);
-- Старый код работает, новый использует name
```

**Правило:** Миграции должны быть **backward compatible**.

### 🔐 Secrets Management

```yaml
# CI/CD с secrets
deploy:
  script:
    - echo $DATABASE_URL | base64 -d > .env
    - docker build -t app .
    - docker run -d --env-file .env app
  variables:
    DATABASE_URL: 
      vault: production/db/url  # Из Vault
```

### 🩺 Health Checks

```javascript
// app/health.js
app.get('/health', (req, res) => {
  const checks = {
    database: await dbHealthCheck(),
    redis: await redisHealthCheck(),
    uptime: process.uptime()
  };
  
  const isHealthy = Object.values(checks).every(Boolean);
  res.status(isHealthy ? 200 : 503).json(checks);
});
```

---

## Неделя 12: CD для TBD

### 💡 Концепция
В TBD **каждый merge в main должен автоматически доходить до продакшена**.

### 🛠 Feature Flags в Production

```javascript
// Градуальный rollout с feature flags
const rolloutPercentage = await featureFlag.getRollout('new-checkout');

if (Math.random() * 100 < rolloutPercentage) {
  return newCheckoutService.process(order);
}
return oldCheckoutService.process(order);
```

### 📊 A/B Testing Integration

```javascript
// Встроенное A/B тестирование
const variant = await abTest.getVariant('checkout-flow', userId);

switch(variant) {
  case 'A': return renderCheckoutA();
  case 'B': return renderCheckoutB();
  default: return renderCheckoutA();
}
```

### 🚨 Production Monitoring

```
Deploy → Monitor metrics → Auto-rollback if issues

Key metrics:
- Error rate < 1%
- Response time < 200ms  
- Success rate > 99.9%
```

### ✅ TBD + CD Workflow

```
1. Code → main branch
2. Automatic deploy to prod  
3. Feature flag controls exposure
4. Monitor metrics
5. Gradually increase flag %
6. Full rollout or rollback
```

---

# 🏗 Фаза 5: Infrastructure & Operations (3 недели)

## Неделя 13: Контейнеризация

### 💡 Концепция
**Контейнеры** решают проблему "у меня работает, у вас нет".

### 🛠 Оптимальный Dockerfile

```dockerfile
# Multi-stage build для минимального размера
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine AS runtime
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
USER node
CMD ["npm", "start"]
```

### 🔧 Docker Compose для разработки

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports: ["3000:3000"]
    environment:
      - DATABASE_URL=postgres://db:5432/app
    depends_on: [db]
    
  db:
    image: postgres:13
    environment:
      POSTGRES_DB: app
    volumes: ["postgres_data:/var/lib/postgresql/data"]
```

### ✅ Container Best Practices

- [ ] Образы < 100MB (Alpine base)
- [ ] Non-root user
- [ ] Health checks included
- [ ] Secrets не в образе
- [ ] Multi-stage builds

---

## Неделя 14: Kubernetes Orchestration

### 💡 Концепция
**Kubernetes** — автопилот для ваших контейнеров.

### 🛠 Basic Deployment

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: app
        image: myapp:latest
        ports: [containerPort: 3000]
        resources:
          requests: {memory: "128Mi", cpu: "100m"}
          limits: {memory: "256Mi", cpu: "200m"}
```

### 🔍 Health Checks в K8s

```yaml
livenessProbe:
  httpGet: {path: /health, port: 3000}
  initialDelaySeconds: 30
  periodSeconds: 10
  
readinessProbe:
  httpGet: {path: /ready, port: 3000}
  initialDelaySeconds: 5
  periodSeconds: 5
```

### ⚡ Auto-scaling

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef: {apiVersion: apps/v1, kind: Deployment, name: app}
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource: {name: cpu, target: {type: Utilization, averageUtilization: 70}}
```

---

## Неделя 15: Infrastructure as Code

### 💡 Концепция
**IaC** — инфраструктура описана в коде и управляется как код.

### 🛠 Terraform Example

```hcl
# main.tf
resource "aws_eks_cluster" "main" {
  name     = var.cluster_name
  role_arn = aws_iam_role.cluster.arn
  version  = "1.24"

  vpc_config {
    subnet_ids = var.subnet_ids
  }
}

output "cluster_endpoint" {
  value = aws_eks_cluster.main.endpoint
}
```

### 🔄 GitOps Workflow

```
Git Repo (Source of Truth)
    ↓
ArgoCD (Deployment Agent)  
    ↓
Kubernetes Cluster (Reality)

Any drift → Auto-sync back to Git state
```

### 🛠 ArgoCD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: backend-app
spec:
  source:
    repoURL: https://github.com/company/app-config
    path: kubernetes/
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated: {prune: true, selfHeal: true}
```

### ✅ IaC Best Practices

- [ ] State в remote backend
- [ ] Environments как переменные
- [ ] Модульная структура
- [ ] Plan перед apply
- [ ] Drift detection настроен

---

# 📊 Фаза 6: Monitoring & Observability (2 недели)

## Неделя 16: Metrics & Logging

### 💡 Концепция
**Нельзя управлять тем, что не измеряешь.**

### 📈 Key Metrics для Backend

```javascript
// Prometheus metrics в Node.js
const prometheus = require('prom-client');

const httpDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code']
});

app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpDuration
      .labels(req.method, req.route?.path, res.statusCode)
      .observe(duration);
  });
  
  next();
});
```

### 📝 Structured Logging

```javascript
// Правильное логирование
const winston = require('winston');

const logger = winston.createLogger({
  format: winston.format.json(),
  defaultMeta: { service: 'backend-api' },
  transports: [new winston.transports.Console()]
});

// В коде
logger.info('User created', {
  userId: user.id,
  email: user.email,
  duration: Date.now() - start,
  requestId: req.id
});
```

### 📊 Grafana Dashboard

```
┌─ Response Time ─┐  ┌─ Error Rate ─┐
│     150ms       │  │     0.5%     │
└─────────────────┘  └──────────────┘

┌─ Throughput ────────────────────────┐
│ ████████████████████████████████████│
│ 1,200 requests/min                  │
└─────────────────────────────────────┘

┌─ Database Pool ─┐  ┌─ Memory Usage ┐
│ Active: 5/20    │  │     78%       │
└─────────────────┘  └───────────────┘
```

---

## Неделя 17: Distributed Tracing

### 💡 Концепция
**Трейсинг** показывает путь запроса через микросервисы.

### 🛠 OpenTelemetry Integration

```javascript
// Автоматическое создание spans
const { NodeTracerProvider } = require('@opentelemetry/sdk-node');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');

const sdk = new NodeSDK({
  traceExporter: new JaegerExporter(),
  instrumentations: [getNodeAutoInstrumentations()]
});

sdk.start();
```

### 📊 Trace Visualization

```
Request: GET /api/orders/123
│
├─ api-gateway     [20ms]
│  └─ auth-check   [5ms]
│
├─ order-service   [100ms]  
│  ├─ db-query     [80ms]  ← Bottleneck!
│  └─ cache-set    [15ms]
│
└─ notification    [10ms]

Total: 135ms
```

### 🚨 Alerting Rules

```yaml
# Prometheus alerting rules
groups:
- name: backend-alerts
  rules:
  - alert: HighErrorRate
    expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
    for: 2m
    annotations:
      summary: "High error rate detected"
      
  - alert: HighLatency  
    expr: histogram_quantile(0.95, http_request_duration_seconds) > 0.5
    for: 5m
```

### ✅ Observability Checklist

- [ ] 4 Golden Signals мониторятся (latency, traffic, errors, saturation)
- [ ] Distributed tracing настроен
- [ ] Алерты настроены для SLI нарушений
- [ ] Dashboards для каждого сервиса
- [ ] Runbooks для каждого алерта

---

# 🔒 Фаза 7: Security (DevSecOps) (2 недели)

## Неделя 18: Security в Pipeline

### 💡 Концепция
**Shift-left security** — безопасность проверяется на каждом этапе, а не в конце.

### 🛠 Security Pipeline

```yaml
# Security checks в CI
security-scan:
  steps:
    - name: Secret scanning
      run: truffleHog --regex --entropy=False .
      
    - name: Dependency check  
      run: npm audit --audit-level=high
      
    - name: SAST scanning
      run: sonar-scanner
      
    - name: Container scanning
      run: trivy image myapp:latest
```

### 🔐 Secrets Management

```javascript
// НЕ ДЕЛАЙТЕ ТАК
const dbUrl = "postgres://user:password@localhost/db";

// ДЕЛАЙТЕ ТАК  
const dbUrl = process.env.DATABASE_URL;

// Или с Vault
const vault = require('node-vault')();
const secrets = await vault.read('secret/database');
const dbUrl = secrets.data.url;
```

### 🛡 Container Security

```dockerfile
# Security hardened Dockerfile
FROM node:18-alpine

# Создать non-root пользователя
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# Копировать файлы с правильными правами
COPY --chown=nextjs:nodejs . .

# Переключиться на non-root
USER nextjs

# Read-only filesystem
COPY --from=builder --chown=nextjs:nodejs /app .
```

---

## Неделя 19: Production Security

### 🛠 Network Security

```yaml
# Kubernetes Network Policies
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-netpol
spec:
  podSelector:
    matchLabels: {app: backend}
  policyTypes: [Ingress, Egress]
  ingress:
  - from:
    - podSelector:
        matchLabels: {app: frontend}
    ports: [protocol: TCP, port: 3000]
  egress:
  - to:
    - podSelector:
        matchLabels: {app: database}
```

### 🔍 Runtime Security

```javascript
// Security middleware
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"]
    }
  },
  hsts: {maxAge: 31536000}
}));

// Rate limiting
app.use(rateLimit({
  windowMs: 15 * 60 * 1000, // 15 минут
  max: 100 // максимум 100 запросов
}));
```

### 📋 Security Checklist

- [ ] Dependency scanning в CI
- [ ] Container images сканируются
- [ ] Secrets в Vault, не в коде
- [ ] Network policies настроены
- [ ] HTTPS everywhere
- [ ] Security headers настроены

---

# ⚡ Фаза 8: Performance & Scaling (2 недели)

## Неделя 20: Performance Testing

### 💡 Концепция
**Performance regression** — когда новый код делает приложение медленнее.

### 🛠 Load Testing в CI

```javascript
// k6 performance test
import http from 'k6/http';
import { check } from 'k6';

export let options = {
  stages: [
    { duration: '2m', target: 100 },  // Ramp up
    { duration: '5m', target: 100 },  // Stay at 100 users  
    { duration: '2m', target: 200 },  // Ramp to 200 users
    { duration: '5m', target: 200 },  // Stay at 200
    { duration: '2m', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% of requests < 500ms
    http_req_failed: ['rate<0.1'],    // Error rate < 10%
  },
};

export default function() {
  let response = http.get('https://api.example.com/users');
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
}
```

### 📊 Performance Monitoring

```javascript
// APM integration
const apm = require('elastic-apm-node').start();

app.get('/slow-endpoint', async (req, res) => {
  const span = apm.startSpan('database-query');
  
  const result = await db.query('SELECT * FROM large_table');
  
  span?.end();
  res.json(result);
});
```

### 🗄 Caching Strategy

```javascript
// Multi-level caching
const redis = require('redis');
const client = redis.createClient();

app.get('/api/user/:id', async (req, res) => {
  const { id } = req.params;
  
  // L1: Memory cache
  let user = memoryCache.get(`user:${id}`);
  if (user) return res.json(user);
  
  // L2: Redis cache  
  user = await client.get(`user:${id}`);
  if (user) {
    memoryCache.set(`user:${id}`, user, 300); // 5min
    return res.json(JSON.parse(user));
  }
  
  // L3: Database
  user = await db.findUser(id);
  await client.setex(`user:${id}`, 3600, JSON.stringify(user)); // 1hr
  memoryCache.set(`user:${id}`, user, 300);
  
  res.json(user);
});
```

---

## Неделя 21: Auto-scaling

### 💡 Концепция
**Elastic infrastructure** — ресурсы масштабируются автоматически под нагрузку.

### 🛠 Kubernetes HPA

```yaml
# Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  minReplicas: 2
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource  
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
    scaleDown:
      stabilizationWindowSeconds: 300
```

### 🔧 Circuit Breaker Pattern

```javascript
const CircuitBreaker = require('opossum');

const options = {
  timeout: 3000,        // Таймаут 3 сек
  errorThresholdPercentage: 50, // 50% ошибок
  resetTimeout: 30000   // Пытаться снова через 30 сек
};

const breaker = new CircuitBreaker(callExternalAPI, options);

breaker.fallback(() => 'Service temporarily unavailable');

// Использование
app.get('/api/data', async (req, res) => {
  try {
    const data = await breaker.fire(req.params);
    res.json(data);
  } catch (err) {
    res.status(503).json({ error: err.message });
  }
});
```

### 💰 Cost Optimization

```yaml
# Vertical Pod Autoscaler для оптимизации ресурсов
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: backend-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: backend
      maxAllowed:
        cpu: 1
        memory: 2Gi
```

---

# 🚀 Фаза 9: Advanced Practices (2 недели)

## Неделя 22: Enterprise TBD

### 💡 Концепция
**Scaling TBD** для больших команд требует дополнительных практик.

### 🏗 Микросервисы + TBD

```
Monorepo Structure:
├── services/
│   ├── auth-service/     ← Команда A
│   ├── payment-service/  ← Команда B  
│   └── order-service/    ← Команда C
├── shared/
│   ├── types/           ← Общие типы
│   └── utils/           ← Утилиты
└── .github/workflows/   ← Shared CI/CD
```

### 🛠 Cross-team Coordination

```yaml
# Dependency management в TBD
# services/order-service/package.json
{
  "dependencies": {
    "@company/auth-types": "^1.2.0",    # Только major.minor
    "@company/payment-sdk": "^2.1.0"    # Backward compatible
  }
}
```

### 📊 TBD Metrics для команды

```javascript
// Metrics dashboard для TBD
const tbdMetrics = {
  branchLifetime: '4.2 hours avg',     // Цель: < 24h
  mergeFrequency: '3.5 merges/day',    // Цель: > 1/day  
  buildSuccess: '96%',                 // Цель: > 95%
  codeReviewTime: '45 min avg',        // Цель: < 2h
  featureFlagCoverage: '78%'           // Цель: > 80%
};
```

---

## Неделя 23: Continuous Improvement

### 💡 Концепция
**DevOps — это journey, не destination.** Постоянное улучшение процессов.

### 📈 SPACE Framework

```
S - Satisfaction and well-being
P - Performance  
A - Activity
C - Communication and collaboration
E - Efficiency and flow
```

### 🛠 Chaos Engineering

```javascript
// Chaos Monkey для Node.js
const chaos = require('chaos-monkey');

if (process.env.NODE_ENV === 'staging') {
  chaos.configure({
    probability: 0.01,  // 1% вероятность хаоса
    attacks: [
      'kill-process',
      'slow-requests', 
      'memory-leak',
      'network-failure'
    ]
  });
}
```

### 🔄 Value Stream Mapping

```
Idea → Code → Build → Test → Deploy → Monitor
  ↓      ↓      ↓       ↓       ↓        ↓
 2d     4h    10m     20m     5m      24/7

Total Lead Time: 2.5 days
Processing Time: 39 minutes  
Efficiency: 39m / 2.5d = 1.1%  ← Opportunity!
```

### ✅ Maturity Assessment

```
DevOps Maturity Levels:

Level 1 - Initial (Manual)
[ ] Manual deployments
[ ] No automated tests  
[ ] Long-lived branches

Level 2 - Managed (Basic CI/CD)  
[ ] Automated builds
[ ] Some tests automated
[ ] Feature branches

Level 3 - Defined (Advanced CI/CD)
[ ] Trunk-based development
[ ] Deployment automation  
[ ] Infrastructure as Code

Level 4 - Quantitatively Managed  
[ ] Metrics-driven decisions
[ ] Performance monitoring
[ ] Continuous optimization  

Level 5 - Optimizing (Elite)
[ ] Continuous experimentation  
[ ] Self-healing systems
[ ] Full observability
```

---

# 🎯 Фаза 10: Capstone Project (1 неделя)

## Неделя 24: Финальный проект

### 🎯 Задача
Создать **production-ready платформу** с полным TBD + DevOps workflow.

### 🛠 Архитектура решения

```
GitHub Repo (TBD)
    ↓
GitHub Actions (CI/CD)
    ↓  
Kubernetes Cluster
    ├── API Gateway (Nginx/Traefik)
    ├── Backend Services (Node.js/Python)  
    ├── Database (PostgreSQL)
    ├── Cache (Redis)
    └── Monitoring (Prometheus/Grafana)
```

### 📋 Deliverables

1. **Code Repository**
   - Trunk-based workflow настроен
   - Feature flags implemented
   - Comprehensive test suite

2. **CI/CD Pipeline**  
   - < 10 min build time
   - Automated security scanning
   - Multi-environment deployment

3. **Infrastructure**
   - Kubernetes cluster (local/cloud)
   - Infrastructure as Code
   - Auto-scaling configured

4. **Monitoring & Observability**
   - Metrics dashboard
   - Distributed tracing
   - Alerting rules

5. **Documentation**
   - Runbooks для операций
   - Developer onboarding guide
   - Architecture decision records

### 🎖 Success Criteria

**DORA Metrics achieved:**
- [ ] Deployment frequency: Daily
- [ ] Lead time: < 1 hour  
- [ ] MTTR: < 30 minutes
- [ ] Change failure rate: < 15%

**TBD Metrics:**
- [ ] Branch lifetime: < 24 hours
- [ ] Merge frequency: > 1/day
- [ ] Feature flag coverage: > 80%

**Technical Excellence:**
- [ ] Test coverage: > 80%
- [ ] Build time: < 10 minutes
- [ ] Zero-downtime deployments
- [ ] Automated rollback

---

# 🎓 Graduation & Next Steps

## 🏆 Вы достигли статуса DevOps Expert!

### 💪 Ваши новые суперсилы:

- **TBD Mastery** — можете внедрить TBD в любой команде
- **CI/CD Expertise** — строите надежные пайплайны с нуля  
- **Infrastructure Skills** — управляете Kubernetes как профи
- **Observability** — видите что происходит в production
- **Security Mindset** — встраиваете безопасность в процессы

### 🚀 Карьерные возможности:

- **DevOps Engineer** ($80k-150k+)
- **Platform Engineer** ($100k-180k+)  
- **Site Reliability Engineer** ($120k-200k+)
- **Cloud Architect** ($130k-220k+)
- **DevOps Consultant** ($150k-300k+)

### 📚 Дальнейшее развитие:

1. **Специализация:**
   - Cloud platforms (AWS/Azure/GCP)
   - Service Mesh (Istio/Linkerd)
   - GitOps (ArgoCD/Flux)

2. **Leadership:**
   - DevOps transformation
   - Platform teams building  
   - Developer experience

3. **Cutting-edge:**
   - ML/AI Operations (MLOps)
   - WebAssembly platforms
   - Edge computing

---

## 📈 Continuous Learning Path

### Месяц 7-12: Углубление
- Cloud provider certification
- Advanced Kubernetes (CKA/CKAD)
- Service Mesh mastery
- Advanced security practices

### Год 2: Экспертиза  
- Архитектурные паттерны
- Multi-cloud strategies
- Performance engineering
- Team leadership

### Год 3+: Innovation
- Emerging technologies
- Conference speaking
- Open source contribution
- Mentoring others

---

## 🎯 Final Wisdom

> **"DevOps — это не про инструменты, это про людей."**

**Помните:**
- Автоматизируйте все, что можно автоматизировать
- Измеряйте все, что важно
- Улучшайте процессы постоянно
- Делитесь знаниями с командой
- **Trunk-Based Development + DevOps = Скорость + Надежность**

**Ваша миссия теперь:** Помочь другим командам достичь такого же уровня excellence!

---

*Поздравляем с завершением курса! Вы прошли путь от новичка до DevOps эксперта. Теперь идите и меняйте мир разработки к лучшему! 🚀*