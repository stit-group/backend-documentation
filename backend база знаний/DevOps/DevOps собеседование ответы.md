# Справочник ответов: DevOps интервью для Backend разработчиков

## Блиц-раунд - Примерные ответы

**1. Назовите 3 основные команды Docker**
```bash
docker build -t myapp .    # Сборка образа
docker run -p 8080:80 myapp # Запуск контейнера
docker ps                  # Список запущенных контейнеров
```

**2. Что такое Pod в Kubernetes?**
Минимальная единица развертывания - группа из одного или нескольких контейнеров с общими ресурсами (сеть, storage).

**3. Основная цель CI/CD?**
Автоматизация процесса доставки кода от разработки до production с быстрой обратной связью.

---

## Блок 1: Контейнеризация

### 1. Docker Image vs Container

**Концепция:**
- **Image** - неизменяемый шаблон с инструкциями для создания контейнера
- **Container** - запущенный экземпляр образа с собственным состоянием

**Практический пример:**
```bash
# Создание образа
docker build -t myapp:v1 .

# Запуск нескольких контейнеров из одного образа
docker run --name app1 -p 8080:3000 myapp:v1
docker run --name app2 -p 8081:3000 myapp:v1
```

**Аналогия:** Image - это чертеж дома, Container - построенный дом

**Оценка уровней:**
- **Junior:** Знает базовые команды, понимает разницу
- **Middle:** Объясняет слои образов, lifecycle контейнеров
- **Senior:** Рассказывает про оптимизацию образов, security implications

### 2. Dockerfile оптимизация

**Базовый Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
USER node
CMD ["npm", "start"]
```

**Ключевые принципы:**
- Многоэтапная сборка для уменьшения размера
- Кэширование слоев (dependencies раньше кода)
- Минимальные base images (alpine)
- Не запускать от root

**Multi-stage пример:**
```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force
COPY --from=builder /app/dist ./dist
USER node
CMD ["npm", "start"]
```

### 3. Docker Networking

**Типы сетей:**
```bash
# Bridge (default) - контейнеры на одном хосте
docker network create myapp-network

# Host - контейнер использует сеть хоста
docker run --network host myapp

# None - без сетевого доступа
docker run --network none myapp
```

**Практический кейс - Web app + Database:**
```yaml
# docker-compose.yml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "8080:3000"
    environment:
      - DB_HOST=postgres
    depends_on:
      - postgres
    networks:
      - app-network

  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: myapp
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```

**Service Discovery:** Контейнеры обращаются друг к другу по имени сервиса

---

## Блок 2: Kubernetes

### 6. Основные компоненты K8s

**Control Plane:**
- **API Server** - точка входа для всех команд
- **etcd** - хранилище состояния кластера
- **Scheduler** - размещение Pod'ов на узлах
- **Controller Manager** - поддержание желаемого состояния

**Node:**
- **kubelet** - агент на каждом узле
- **kube-proxy** - сетевой прокси
- **Container Runtime** - Docker/containerd

**Базовые объекты:**
```yaml
# Pod - минимальная единица
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:v1
    ports:
    - containerPort: 3000

---
# Service - сетевой доступ
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 3000
  type: ClusterIP

---
# Deployment - управление репликами
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:v1
```

### 8. ConfigMaps vs Secrets

**ConfigMap для конфигурации:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app.properties: |
    debug=true
    feature.flag=enabled
  DATABASE_URL: "postgres://localhost:5432/myapp"

---
# Использование в Pod
spec:
  containers:
  - name: app
    image: myapp:v1
    envFrom:
    - configMapRef:
        name: app-config
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

**Secret для чувствительных данных:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: cG9zdGdyZXM=  # base64 encoded
  password: c2VjcmV0UGFzcw==

---
# Использование
spec:
  containers:
  - name: app
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
```

**Best practices:**
- Secrets в etcd шифруются at rest
- Используйте external secret operators (External Secrets, Vault)
- Ротация паролей через автоматизацию

### 9. Автоскейлинг

**Horizontal Pod Autoscaler (HPA):**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp-deployment
  minReplicas: 2
  maxReplicas: 10
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
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
```

**Vertical Pod Autoscaler (VPA):**
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: myapp-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp-deployment
  updatePolicy:
    updateMode: "Auto"  # Auto/Initial/Off
  resourcePolicy:
    containerPolicies:
    - containerName: app
      maxAllowed:
        memory: 2Gi
        cpu: 1000m
```

**Кейс - Scheduled scaling:**
```yaml
# CronJob для предиктивного скейлинга
apiVersion: batch/v1
kind: CronJob
metadata:
  name: scale-up-morning
spec:
  schedule: "0 8 * * 1-5"  # 8:00 в рабочие дни
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: scaler
            image: bitnami/kubectl
            command:
            - /bin/sh
            - -c
            - kubectl scale deployment myapp-deployment --replicas=20
          restartPolicy: OnFailure
```

### 11. Helm Charts

**Базовая структура chart:**
```
myapp/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── _helpers.tpl
└── charts/  # зависимости
```

**Chart.yaml:**
```yaml
apiVersion: v2
name: myapp
version: 1.0.0
appVersion: "v1.2.3"
dependencies:
- name: postgresql
  version: 12.x.x
  repository: https://charts.bitnami.com/bitnami
  condition: postgresql.enabled
```

**values.yaml:**
```yaml
replicaCount: 3
image:
  repository: myapp
  tag: v1.2.3
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  host: myapp.example.com

postgresql:
  enabled: true
  auth:
    database: myapp
```

**Template пример (deployment.yaml):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: 3000
        env:
        {{- if .Values.postgresql.enabled }}
        - name: DATABASE_URL
          value: "postgres://{{ .Values.postgresql.auth.username }}@{{ include "myapp.fullname" . }}-postgresql:5432/{{ .Values.postgresql.auth.database }}"
        {{- end }}
```

---

## Блок 3: CI/CD

### 19. Дизайн CI Pipeline

**Этапы pipeline:**
```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16, 18, 20]
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linting
      run: npm run lint
    
    - name: Run unit tests
      run: npm run test:unit
    
    - name: Run integration tests
      run: npm run test:integration
      env:
        DATABASE_URL: postgres://test:test@localhost:5432/testdb

  security:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Run security audit
      run: npm audit --audit-level=high
    
    - name: SAST scanning
      uses: github/codeql-action/analyze@v2

  build:
    needs: [test, security]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
    - uses: actions/checkout@v3
    
    - name: Build Docker image
      run: |
        docker build -t myapp:${{ github.sha }} .
        docker tag myapp:${{ github.sha }} myapp:latest
    
    - name: Push to registry
      run: |
        echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
        docker push myapp:${{ github.sha }}
        docker push myapp:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: production
    steps:
    - name: Deploy to Kubernetes
      run: |
        kubectl set image deployment/myapp myapp=myapp:${{ github.sha }}
        kubectl rollout status deployment/myapp
```

**Микросервисная архитектура - Monorepo подход:**
```yaml
# Определение изменившихся сервисов
- name: Detect changes
  id: changes
  uses: dorny/paths-filter@v2
  with:
    filters: |
      user-service:
        - 'services/user-service/**'
      order-service:
        - 'services/order-service/**'
      shared:
        - 'shared/**'

# Параллельная сборка только изменившихся сервисов
- name: Build user service
  if: steps.changes.outputs.user-service == 'true'
  run: |
    cd services/user-service
    docker build -t user-service:${{ github.sha }} .
```

### 20. Стратегии Deployment

**Blue-Green Deployment:**
```yaml
# blue-green-deploy.yml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp-rollout
spec:
  replicas: 5
  strategy:
    blueGreen:
      activeService: myapp-active
      previewService: myapp-preview
      autoPromotionEnabled: false
      scaleDownDelaySeconds: 30
      prePromotionAnalysis:
        templates:
        - templateName: success-rate
        args:
        - name: service-name
          value: myapp-preview
      postPromotionAnalysis:
        templates:
        - templateName: success-rate
        args:
        - name: service-name
          value: myapp-active
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:stable
```

**Canary Deployment:**
```yaml
strategy:
  canary:
    steps:
    - setWeight: 10
    - pause: {duration: 300s}
    - analysis:
        templates:
        - templateName: error-rate
        args:
        - name: service-name
          value: myapp-canary
    - setWeight: 50
    - pause: {duration: 600s}
    - setWeight: 100
    canaryService: myapp-canary
    stableService: myapp-stable
    trafficRouting:
      istio:
        virtualService:
          name: myapp-vs
```

**Feature Flags подход:**
```javascript
// В приложении
const featureFlag = process.env.FEATURE_NEW_CHECKOUT || false;

if (featureFlag && userSegment === 'beta') {
  return newCheckoutFlow(request);
} else {
  return legacyCheckoutFlow(request);
}
```

---

## Блок 4: Infrastructure as Code

### 26. Terraform Fundamentals

**Базовая структура проекта:**
```
terraform/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
├── modules/
│   ├── vpc/
│   ├── eks/
│   └── rds/
└── shared/
    └── backend.tf
```

**Модуль VPC:**
```hcl
# modules/vpc/main.tf
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "${var.environment}-vpc"
    Environment = var.environment
  }
}

resource "aws_subnet" "private" {
  count             = length(var.private_subnet_cidrs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidrs[count.index]
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "${var.environment}-private-${count.index + 1}"
    Type = "private"
  }
}

# outputs.tf
output "vpc_id" {
  value = aws_vpc.main.id
}

output "private_subnet_ids" {
  value = aws_subnet.private[*].id
}
```

**Использование модуля:**
```hcl
# environments/prod/main.tf
terraform {
  backend "s3" {
    bucket = "mycompany-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "us-west-2"
  }
}

module "vpc" {
  source = "../../modules/vpc"
  
  environment           = "prod"
  vpc_cidr             = "10.0.0.0/16"
  private_subnet_cidrs = ["10.0.1.0/24", "10.0.2.0/24"]
}

module "eks" {
  source = "../../modules/eks"
  
  cluster_name     = "prod-cluster"
  vpc_id          = module.vpc.vpc_id
  subnet_ids      = module.vpc.private_subnet_ids
  node_group_size = {
    desired = 3
    max     = 10
    min     = 1
  }
}
```

**State management:**
```hcl
# Remote state для совместной работы
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

# Data source для чтения state другого проекта
data "terraform_remote_state" "vpc" {
  backend = "s3"
  config = {
    bucket = "terraform-state-bucket"
    key    = "vpc/terraform.tfstate"
    region = "us-west-2"
  }
}
```

### 29. Secrets в IaC

**AWS Secrets Manager интеграция:**
```hcl
# Создание секрета
resource "aws_secretsmanager_secret" "db_password" {
  name = "${var.environment}-db-password"
  
  tags = {
    Environment = var.environment
  }
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id = aws_secretsmanager_secret.db_password.id
  secret_string = jsonencode({
    username = "admin"
    password = random_password.db_password.result
  })
}

resource "random_password" "db_password" {
  length  = 16
  special = true
}

# Использование в RDS
resource "aws_db_instance" "main" {
  allocated_storage = 20
  engine           = "postgres"
  engine_version   = "15.3"
  instance_class   = "db.t3.micro"
  
  manage_master_user_password = true
  master_username            = "admin"
  
  tags = {
    Environment = var.environment
  }
}
```

**Vault Provider:**
```hcl
provider "vault" {
  address = "https://vault.example.com"
  token   = var.vault_token
}

data "vault_generic_secret" "db_creds" {
  path = "secret/database/${var.environment}"
}

resource "kubernetes_secret" "db_secret" {
  metadata {
    name = "db-credentials"
  }
  
  data = {
    username = data.vault_generic_secret.db_creds.data["username"]
    password = data.vault_generic_secret.db_creds.data["password"]
  }
}
```

---

## Блок 5: Observability

### 31. Monitoring Strategy

**Three Pillars of Observability:**

**1. Metrics (Prometheus + Grafana):**
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
- job_name: 'kubernetes-pods'
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
    action: keep
    regex: true
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
    action: replace
    target_label: __metrics_path__
    regex: (.+)

- job_name: 'node-exporter'
  static_configs:
  - targets: ['localhost:9100']
```

**Application metrics в коде:**
```javascript
// Node.js с Prometheus клиентом
const promClient = require('prom-client');

// Metrics
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code']
});

const activeConnections = new promClient.Gauge({
  name: 'active_connections',
  help: 'Number of active connections'
});

// Middleware
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestDuration
      .labels(req.method, req.route?.path || req.path, res.statusCode)
      .observe(duration);
  });
  
  next();
});
```

**2. Logs (ELK Stack):**
```yaml
# filebeat.yml
filebeat.inputs:
- type: container
  paths:
    - /var/log/containers/*.log
  processors:
  - add_kubernetes_metadata:
      host: ${NODE_NAME}
      matchers:
      - logs_path:
          logs_path: "/var/log/containers/"

output.logstash:
  hosts: ["logstash:5044"]

# logstash.conf
input {
  beats {
    port => 5044
  }
}

filter {
  if [kubernetes][container][name] == "myapp" {
    json {
      source => "message"
    }
    
    date {
      match => [ "timestamp", "ISO8601" ]
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "k8s-logs-%{+YYYY.MM.dd}"
  }
}
```

**Structured logging:**
```javascript
const winston = require('winston');

const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console()
  ]
});

// Usage
logger.info('User login attempt', {
  userId: user.id,
  email: user.email,
  ip: req.ip,
  userAgent: req.get('User-Agent'),
  traceId: req.headers['x-trace-id']
});
```

**3. Traces (Jaeger + OpenTelemetry):**
```javascript
// OpenTelemetry setup
const { NodeSDK } = require('@opentelemetry/sdk-node');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');
const { JaegerExporter } = require('@opentelemetry/exporter-jaeger');

const sdk = new NodeSDK({
  traceExporter: new JaegerExporter({
    endpoint: 'http://jaeger:14268/api/traces',
  }),
  instrumentations: [getNodeAutoInstrumentations()]
});

// Manual tracing
const opentelemetry = require('@opentelemetry/api');

async function processOrder(orderId) {
  const tracer = opentelemetry.trace.getTracer('order-service');
  
  return tracer.startActiveSpan('process-order', async (span) => {
    span.setAttributes({
      'order.id': orderId,
      'service.name': 'order-service'
    });
    
    try {
      // Business logic
      const order = await fetchOrder(orderId);
      const payment = await processPayment(order);
      
      span.setStatus({ code: opentelemetry.SpanStatusCode.OK });
      return payment;
    } catch (error) {
      span.recordException(error);
      span.setStatus({ 
        code: opentelemetry.SpanStatusCode.ERROR,
        message: error.message 
      });
      throw error;
    } finally {
      span.end();
    }
  });
}
```

### 32. SLI/SLO Definition

**Service Level Indicators:**
```yaml
# Prometheus recording rules
groups:
- name: sli_rules
  rules:
  # Availability SLI
  - record: sli:availability:ratio_rate5m
    expr: |
      (
        sum(rate(http_requests_total{job="myapp",code!~"5.."}[5m]))
        /
        sum(rate(http_requests_total{job="myapp"}[5m]))
      )
  
  # Latency SLI
  - record: sli:latency:p99_5m
    expr: |
      histogram_quantile(0.99,
        sum(rate(http_request_duration_seconds_bucket{job="myapp"}[5m])) by (le)
      )
  
  # Error rate SLI
  - record: sli:error_rate:ratio_rate5m
    expr: |
      (
        sum(rate(http_requests_total{job="myapp",code=~"5.."}[5m]))
        /
        sum(rate(http_requests_total{job="myapp"}[5m]))
      )
```

**Alerting rules:**
```yaml
groups:
- name: slo_alerts
  rules:
  # SLO breach: 99.9% availability over 1 hour
  - alert: AvailabilitySLOBreach
    expr: |
      (
        1 - avg_over_time(sli:availability:ratio_rate5m[1h])
      ) > 0.001
    for: 2m
    labels:
      severity: critical
      slo: availability
    annotations:
      summary: "Availability SLO breach detected"
      description: "Availability is {{ $value | humanizePercentage }} over the last hour"

  # Error budget burn rate
  - alert: ErrorBudgetBurnRateHigh
    expr: |
      (
        avg_over_time(sli:error_rate:ratio_rate5m[5m]) > (0.001 * 14.4)
        and
        avg_over_time(sli:error_rate:ratio_rate5m[1h]) > (0.001 * 6)
      )
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "High error budget burn rate"
```

---

## Практические кейсы - Примеры решений

### Кейс 1: Production Incident Response

**Сценарий:** Kubernetes приложение возвращает 500 ошибки

**Пошаговая диагностика:**
```bash
# 1. Проверка общего состояния
kubectl get pods -n production
kubectl get events -n production --sort-by='.lastTimestamp'

# 2. Анализ конкретного Pod
kubectl describe pod myapp-deployment-xyz -n production
kubectl logs myapp-deployment-xyz -n production --previous

# 3. Проверка ресурсов
kubectl top pods -n production
kubectl get hpa -n production

# 4. Проверка сети и сервисов
kubectl get svc,endpoints -n production
kubectl exec -it myapp-deployment-xyz -n production -- netstat -tlnp

# 5. Проверка зависимостей
kubectl get pods -l app=postgres -n production
kubectl logs -l app=postgres -n production --tail=100
```

**Monitoring queries:**
```promql
# Error rate spike
increase(http_requests_total{job="myapp",code=~"5.."}[5m])

# Resource utilization
container_memory_usage_bytes{pod=~"myapp.*"} / container_spec_memory_limit_bytes

# Database connections
pg_stat_activity_count{state="active"}
```

**Recovery план:**
1. **Immediate:** Scale down to known good replicas
2. **Rollback:** Revert to previous working version
3. **Investigation:** Analyze logs and metrics
4. **Post-mortem:** Document and improve

### Кейс 2: Performance Optimization

**Сценарий:** Microservice high latency под нагрузкой

**Диагностика:**
```yaml
# HPA с custom metrics
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 50
  metrics:
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "100"
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70

# Pod Disruption Budget
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 70%
  selector:
    matchLabels:
      app: myapp
```

**Load testing:**
```yaml
# K6 load test
apiVersion: batch/v1
kind: Job
metadata:
  name: load-test
spec:
  template:
    spec:
      containers:
      - name: k6
        image: grafana/k6
        command: ["k6", "run", "--vus", "100", "--duration", "10m", "/scripts/test.js"]
        volumeMounts:
        - name: test-script
          mountPath: /scripts
      volumes:
      - name: test-script
        configMap:
          name: k6-script
      restartPolicy: Never
```

**Optimization стратегии:**
- Connection pooling
- Caching layers (Redis)
- Database query optimization
- Async processing (message queues)

---

## Оценка качества ответов

### Green Flags (хорошие ответы):

**Junior уровень:**
- Знает основные команды и концепции
- Может объяснить назначение инструментов
- Понимает базовый workflow

**Middle уровень:**
- Приводит практические примеры из опыта
- Понимает trade-offs решений
- Знает best practices и security concerns
- Может спроектировать решение end-to-end

**Senior уровень:**
- Связывает технические решения с бизнес-целями
- Понимает архитектурные patterns
- Опыт troubleshooting в production
- Знает enterprise concerns (compliance, governance)
- Может менторить команду

### Red Flags (плохие ответы):

- Не может объяснить зачем нужен инструмент
- Только теоретические знания без практики
- Не понимает security implications
- Копирует конфигурации без понимания
- Не может адаптировать решение под контекст
- Нет понимания monitoring и observability

### Дополнительные вопросы для углубления:

1. **"Расскажите о real case из вашего опыта"**
2. **"Что бы вы сделали по-другому?"**
3. **"Какие альтернативы рассматривали?"**
4. **"Как измеряли успех решения?"**
5. **"С какими проблемами столкнулись?"**

---

## Заключение

Этот справочник поможет:
- **Интервьюерам:** Понять глубину знаний кандидата
- **Кандидатам:** Подготовиться к техническим вопросам
- **Командам:** Стандартизировать процесс найма

**Помните:** Лучшие кандидаты не только знают инструменты, но и понимают, когда и зачем их применять.