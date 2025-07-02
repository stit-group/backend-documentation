# DevOps & Trunk-Based Development: Структурированный обучающий курс

## 🎯 Цель курса
Превратиться из backend-разработчика в DevOps-эксперта с глубоким пониманием Trunk-Based Development и способностью построить production-ready CI/CD pipeline.

---

# Блок 1: Основы DevOps

## Глава 1: DevOps Культура и Философия

### Концепция DevOps
DevOps — это не инструменты, это **культура быстрой доставки качественного ПО**. Основная идея: разрушить стену между разработкой и операциями.

```
БЫЛО:                    СТАЛО:
Dev → QA → Ops          Dev ↔ DevOps ↔ Ops
 ↓     ↓    ↓              ↓
Slow  Bugs  Fire         Fast + Reliable
```

### DORA Метрики (компас развития)
1. **Deployment Frequency** — как часто деплоим
2. **Lead Time** — от коммита до продакшена  
3. **MTTR** — время восстановления после сбоя
4. **Change Failure Rate** — % плохих деплоев

**Цель:** Элитные команды деплоят **несколько раз в день** с **MTTR < 1 часа**.

### Практические задания
- Измерьте базовые метрики команды
- Создайте dashboard для отслеживания DORA метрик
- Проанализируйте частоту коммитов за месяц

## Глава 2: Git Mastery и Advanced Practices

### Продвинутые Git практики

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

### Чек-лист Git Excellence
- [ ] Защищена главная ветка (branch protection)
- [ ] Настроены pre-commit hooks
- [ ] Все коммиты следуют конвенции
- [ ] Code review обязателен для merge

---

# Блок 2: Trunk-Based Development

## Глава 3: TBD Философия и Принципы

### Концепция Trunk-Based Development
**Trunk-Based Development** — все разработчики работают с одной главной веткой (trunk/main).

#### Сравнение подходов
```
Git Flow (медленно):          TBD (быстро):
                             
main ──────────────          main ●─●─●─●─●─●
 │                            │   │ │ │ │ │
 └─ feature ───┐              └─●─┘ │ │ │ │  
   └─ hotfix ──┘                └─●─┘ │ │
                                  └─●─┘ │
Merge Hell                          └─●─┘
```

### Ключевые принципы TBD
1. **Короткие ветки** — жизнь < 24 часов
2. **Маленькие коммиты** — 1-2 файла за раз
3. **Feature flags** — код в продакшене, фича выключена
4. **Быстрая интеграция** — merge минимум раз в день

### Feature Flags Implementation
```javascript
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

## Глава 4: Качество в TBD

### Обеспечение качества через автоматизацию
В TBD **качество обеспечивается автоматизацией**, а не долгими ветками.

### Pre-commit Pipeline
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

### Code Review для TBD
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

### Definition of Done
- [ ] Тесты проходят
- [ ] Code coverage не упал
- [ ] Static analysis passed
- [ ] Feature flag готов (если нужен)
- [ ] Backward compatibility сохранена

---

# Блок 3: Continuous Integration

## Глава 5: CI Основы и Архитектура

### Концепция Continuous Integration
**Continuous Integration** — каждый коммит автоматически проверяется и собирается.

**Цель:** Найти проблемы за **минуты**, а не дни.

```
Developer commits → CI triggers → Tests run → Feedback in 5min
```

### Базовый CI Pipeline
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

### Test Pyramid
```
    /\     E2E Tests (few, slow, expensive)
   /  \    
  /____\   Integration Tests (some, medium)
 /      \  
/________\  Unit Tests (many, fast, cheap)
```

## Глава 6: Advanced CI Techniques

### Test Containers для Integration Tests
```javascript
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

### Optimization Strategies
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

## Глава 7: CI + TBD Integration

### Smart CI для TBD
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

### TBD CI Checklist
- [ ] PR checks завершаются за < 5 минут
- [ ] Main branch всегда deployable
- [ ] Автоматический rollback при сбоях
- [ ] Feature flags интегрированы в CI

---

# Блок 4: Continuous Delivery

## Глава 8: CD Основы и Deployment Strategies

### Концепция Continuous Delivery
**Continuous Delivery** — код всегда готов к деплою одним кликом.
**Continuous Deployment** — код деплоится автоматически.

```
CD Pipeline:
Code → Build → Test → Deploy to Staging → Deploy to Prod
                ↓
           All Automated
```

### Environment Promotion
```yaml
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
  when: manual
  
deploy-prod:
  stage: deploy-prod
  when: manual
  only: [main]
```

### Deployment Strategies

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

## Глава 9: Advanced Deployment Practices

### Database Migrations в CD
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

### Secrets Management
```yaml
deploy:
  script:
    - echo $DATABASE_URL | base64 -d > .env
    - docker build -t app .
    - docker run -d --env-file .env app
  variables:
    DATABASE_URL: 
      vault: production/db/url
```

### Health Checks
```javascript
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

## Глава 10: CD для TBD

### Feature Flags в Production
```javascript
const rolloutPercentage = await featureFlag.getRollout('new-checkout');

if (Math.random() * 100 < rolloutPercentage) {
  return newCheckoutService.process(order);
}
return oldCheckoutService.process(order);
```

### A/B Testing Integration
```javascript
const variant = await abTest.getVariant('checkout-flow', userId);

switch(variant) {
  case 'A': return renderCheckoutA();
  case 'B': return renderCheckoutB();
  default: return renderCheckoutA();
}
```

### TBD + CD Workflow
```
1. Code → main branch
2. Automatic deploy to prod  
3. Feature flag controls exposure
4. Monitor metrics
5. Gradually increase flag %
6. Full rollout or rollback
```

---

# Блок 5: Infrastructure & Operations

## Глава 11: Контейнеризация и Docker

### Оптимальный Dockerfile
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

### Docker Compose для разработки
```yaml
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

### Container Best Practices
- [ ] Образы < 100MB (Alpine base)
- [ ] Non-root user
- [ ] Health checks included
- [ ] Secrets не в образе
- [ ] Multi-stage builds

## Глава 12: Kubernetes Orchestration

### Basic Deployment
```yaml
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

### Health Checks в K8s
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

### Auto-scaling
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

## Глава 13: Infrastructure as Code

### Terraform Example
```hcl
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

### GitOps Workflow
```
Git Repo (Source of Truth)
    ↓
ArgoCD (Deployment Agent)  
    ↓
Kubernetes Cluster (Reality)

Any drift → Auto-sync back to Git state
```

### ArgoCD Application
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

---

# Блок 6: Monitoring & Observability

## Глава 14: Metrics и Logging

### Key Metrics для Backend
```javascript
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

### Structured Logging
```javascript
const winston = require('winston');

const logger = winston.createLogger({
  format: winston.format.json(),
  defaultMeta: { service: 'backend-api' },
  transports: [new winston.transports.Console()]
});

logger.info('User created', {
  userId: user.id,
  email: user.email,
  duration: Date.now() - start,
  requestId: req.id
});
```

## Глава 15: Distributed Tracing

### OpenTelemetry Integration
```javascript
const { NodeTracerProvider } = require('@opentelemetry/sdk-node');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');

const sdk = new NodeSDK({
  traceExporter: new JaegerExporter(),
  instrumentations: [getNodeAutoInstrumentations()]
});

sdk.start();
```

### Trace Visualization
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

### Alerting Rules
```yaml
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

---

# Блок 7: Security (DevSecOps)

## Глава 16: Security в Pipeline

### Security Pipeline
```yaml
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

### Secrets Management
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

### Container Security
```dockerfile
FROM node:18-alpine

# Создать non-root пользователя
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# Копировать файлы с правильными правами
COPY --chown=nextjs:nodejs . .

# Переключиться на non-root
USER nextjs
```

## Глава 17: Production Security

### Network Security
```yaml
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
```

### Runtime Security
```javascript
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"]
    }
  },
  hsts: {maxAge: 31536000}
}));

app.use(rateLimit({
  windowMs: 15 * 60 * 1000, // 15 минут
  max: 100 // максимум 100 запросов
}));
```

---

# Блок 8: Performance & Scaling

## Глава 18: Performance Testing

### Load Testing в CI
```javascript
import http from 'k6/http';
import { check } from 'k6';

export let options = {
  stages: [
    { duration: '2m', target: 100 },
    { duration: '5m', target: 100 },
    { duration: '2m', target: 200 },
    { duration: '5m', target: 200 },
    { duration: '2m', target: 0 },
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],
    http_req_failed: ['rate<0.1'],
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

### Caching Strategy
```javascript
app.get('/api/user/:id', async (req, res) => {
  const { id } = req.params;
  
  // L1: Memory cache
  let user = memoryCache.get(`user:${id}`);
  if (user) return res.json(user);
  
  // L2: Redis cache  
  user = await client.get(`user:${id}`);
  if (user) {
    memoryCache.set(`user:${id}`, user, 300);
    return res.json(JSON.parse(user));
  }
  
  // L3: Database
  user = await db.findUser(id);
  await client.setex(`user:${id}`, 3600, JSON.stringify(user));
  memoryCache.set(`user:${id}`, user, 300);
  
  res.json(user);
});
```

## Глава 19: Auto-scaling и Circuit Breakers

### Kubernetes HPA
```yaml
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
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
```

### Circuit Breaker Pattern
```javascript
const CircuitBreaker = require('opossum');

const options = {
  timeout: 3000,
  errorThresholdPercentage: 50,
  resetTimeout: 30000
};

const breaker = new CircuitBreaker(callExternalAPI, options);
breaker.fallback(() => 'Service temporarily unavailable');

app.get('/api/data', async (req, res) => {
  try {
    const data = await breaker.fire(req.params);
    res.json(data);
  } catch (err) {
    res.status(503).json({ error: err.message });
  }
});
```

---

# Блок 9: Advanced Practices

## Глава 20: Enterprise TBD

### Микросервисы + TBD
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

### Cross-team Coordination
```yaml
# Dependency management в TBD
{
  "dependencies": {
    "@company/auth-types": "^1.2.0",
    "@company/payment-sdk": "^2.1.0"
  }
}
```

### TBD Metrics для команды
```javascript
const tbdMetrics = {
  branchLifetime: '4.2 hours avg',
  mergeFrequency: '3.5 merges/day',
  buildSuccess: '96%',
  codeReviewTime: '45 min avg',
  featureFlagCoverage: '78%'
};
```

## Глава 21: Continuous Improvement

### SPACE Framework
```
S - Satisfaction and well-being
P - Performance  
A - Activity
C - Communication and collaboration
E - Efficiency and flow
```

### Chaos Engineering
```javascript
const chaos = require('chaos-monkey');

if (process.env.NODE_ENV === 'staging') {
  chaos.configure({
    probability: 0.01,
    attacks: [
      'kill-process',
      'slow-requests', 
      'memory-leak',
      'network-failure'
    ]
  });
}
```

### Value Stream Mapping
```
Idea → Code → Build → Test → Deploy → Monitor
  ↓      ↓      ↓       ↓       ↓        ↓
 2d     4h    10m     20m     5m      24/7

Total Lead Time: 2.5 days
Processing Time: 39 minutes  
Efficiency: 39m / 2.5d = 1.1%  ← Opportunity!
```

---

# Блок 10: Capstone Project

## Глава 22: Production-Ready Platform

### Архитектура решения
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

### Deliverables

#### 1. Code Repository
- Trunk-based workflow настроен
- Feature flags implemented
- Comprehensive test suite

#### 2. CI/CD Pipeline
- < 10 min build time
- Automated security scanning
- Multi-environment deployment

#### 3. Infrastructure
- Kubernetes cluster
- Infrastructure as Code
- Auto-scaling configured

#### 4. Monitoring & Observability
- Metrics dashboard
- Distributed tracing
- Alerting rules

#### 5. Documentation
- Runbooks для операций
- Developer onboarding guide
- Architecture decision records

### Success Criteria

**DORA Metrics:**
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

# 🎓 Результаты обучения

## Полученные навыки

### Core DevOps Skills
- TBD implementation и leadership
- Production-ready CI/CD pipelines
- Kubernetes orchestration
- Infrastructure as Code
- Comprehensive monitoring
- Security integration
- Performance optimization

### Leadership & Process
- DevOps transformation leadership
- Team coaching в TBD practices
- Metrics-driven decision making
- Continuous improvement culture

## Карьерные возможности
- **DevOps Engineer** ($80k-150k+)
- **Platform Engineer** ($100k-180k+)  
- **Site Reliability Engineer** ($120k-200k+)
- **Cloud Architect** ($130k-220k+)
- **DevOps Consultant** ($150k-300k+)

## Дальнейшее развитие

### Специализация
- Cloud platforms (AWS/Azure/GCP)
- Service Mesh (Istio/Linkerd)
- GitOps (ArgoCD/Flux)

### Leadership
- DevOps transformation
- Platform teams building  
- Developer experience

### Cutting-edge
- ML/AI Operations (MLOps)
- WebAssembly platforms
- Edge computing

---

**Помните:** DevOps — это не про инструменты, это про людей. Ваша миссия: помочь командам достичь скорости и надежности через культуру continuous improvement!