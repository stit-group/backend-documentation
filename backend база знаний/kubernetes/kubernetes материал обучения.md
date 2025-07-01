# Полное руководство по Kubernetes для Backend разработчиков

## Содержание

1. [Введение: Зачем нужен Kubernetes?](#введение-зачем-нужен-kubernetes)
2. [От Docker к Kubernetes](#от-docker-к-kubernetes)
3. [Архитектура Kubernetes](#архитектура-kubernetes)
4. [Основные объекты и ресурсы](#основные-объекты-и-ресурсы)
5. [Управление приложениями](#управление-приложениями)
6. [Конфигурация и секреты](#конфигурация-и-секреты)
7. [Сетевое взаимодействие](#сетевое-взаимодействие)
8. [Хранение данных](#хранение-данных)
9. [Мониторинг и отладка](#мониторинг-и-отладка)
10. [Безопасность](#безопасность)
11. [Автоматизация и CI/CD](#автоматизация-и-cicd)
12. [Production-ready практики](#production-ready-практики)
13. [Продвинутые темы](#продвинутые-темы)

---

## Введение: Зачем нужен Kubernetes?

### Проблемы масштабирования приложений

Представьте: у вас есть успешное backend приложение на Python/Node.js/Go. Сначала оно работало на одном сервере, потом понадобилось больше серверов. Теперь у вас:

- 10+ серверов с разными конфигурациями
- Проблемы с балансировкой нагрузки
- Сложности с деплоем обновлений
- Проблемы с отказоустойчивостью
- Ручное управление каждым сервером

**Kubernetes решает эти проблемы:**

```
Было:                           Стало:
┌─────────┐ ┌─────────┐        ┌─────────────────────────┐
│ Сервер1 │ │ Сервер2 │   →    │    Kubernetes Cluster   │
│ App v1  │ │ App v2  │        │  ┌─────┐ ┌─────┐ ┌─────┐│
└─────────┘ └─────────┘        │  │ Pod │ │ Pod │ │ Pod ││
                               │  └─────┘ └─────┘ └─────┘│
Ручное управление              │   Автоматическое        │
                               │   управление            │
                               └─────────────────────────┘
```

### Ключевые преимущества Kubernetes

**1. Автоматическое масштабирование**
- Горизонтальное: добавление новых экземпляров приложения
- Вертикальное: увеличение ресурсов существующих экземпляров
- Автоматическое: на основе нагрузки CPU/памяти

**2. Самовосстановление**
- Автоматический перезапуск упавших контейнеров
- Замена неработающих узлов
- Health checks и автоматическая изоляция проблемных экземпляров

**3. Декларативное управление**
```yaml
# Вы описываете "что хотите":
replicas: 3
image: myapp:v2

# Kubernetes обеспечивает "как это сделать":
# - Создает 3 экземпляра
# - Обновляет образы
# - Поддерживает желаемое состояние
```

**4. Портативность**
- Одинаково работает в облаке, on-premise, гибридных средах
- Избежание vendor lock-in
- Миграция между провайдерами без изменения кода

---

## От Docker к Kubernetes

### Docker: Основа контейнеризации

Прежде чем углубляться в Kubernetes, важно понимать Docker:

```dockerfile
# Dockerfile для backend приложения
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

**Docker решает проблему "у меня работает":**
- Инкапсуляция приложения и его зависимостей
- Консистентность между dev/test/prod средами
- Быстрое развертывание

### Ограничения Docker в продакшене

```bash
# Проблемы при масштабировании:
docker run -d --name app1 myapp:latest
docker run -d --name app2 myapp:latest
docker run -d --name app3 myapp:latest

# Вопросы:
# - Как распределить нагрузку между app1, app2, app3?
# - Что если app2 упадет?
# - Как обновить все экземпляры одновременно?
# - Как масштабировать автоматически?
```

### Kubernetes как оркестратор

Kubernetes берет контейнеры Docker и добавляет:

```
Docker Container → Kubernetes Pod → Kubernetes Deployment → Service
     ↓                    ↓                    ↓              ↓
  Изолированная      Группа связанных     Управление        Сетевой
  среда выполнения   контейнеров         репликами         доступ
```

**Пример трансформации:**

```yaml
# Docker Compose (для разработки)
version: '3'
services:
  app:
    image: myapp:latest
    ports:
      - "3000:3000"
    replicas: 3

# Kubernetes Deployment (для продакшена)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
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
        image: myapp:latest
        ports:
        - containerPort: 3000
```

---

## Архитектура Kubernetes

### Кластерная архитектура

```
                    Kubernetes Cluster
    ┌─────────────────────────────────────────────────────────┐
    │                 Control Plane                           │
    │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐   │
    │  │API Server│ │  etcd    │ │Controller│ │Scheduler │   │
    │  │          │ │          │ │ Manager  │ │          │   │
    │  └──────────┘ └──────────┘ └──────────┘ └──────────┘   │
    └─────────────────────────────────────────────────────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
    ┌─────────▼─────────┐ ┌──────▼──────┐ ┌─────────▼─────────┐
    │    Worker Node    │ │Worker Node  │ │   Worker Node     │
    │ ┌─────┐ ┌─────┐   │ │ ┌─────┐     │ │ ┌─────┐ ┌─────┐   │
    │ │ Pod │ │ Pod │   │ │ │ Pod │     │ │ │ Pod │ │ Pod │   │
    │ └─────┘ └─────┘   │ │ └─────┘     │ │ └─────┘ └─────┘   │
    │   kubelet         │ │  kubelet    │ │   kubelet         │
    │   kube-proxy      │ │  kube-proxy │ │   kube-proxy      │
    └───────────────────┘ └─────────────┘ └───────────────────┘
```

### Control Plane: Мозг кластера

**API Server** - единая точка входа
```bash
# Все команды идут через API Server:
kubectl get pods                    # → API Server
kubectl apply -f deployment.yaml    # → API Server
kubectl logs my-pod                 # → API Server → kubelet
```

**etcd** - распределенная база данных состояния
```
etcd хранит:
├── Конфигурации всех объектов
├── Текущее состояние кластера
├── Метаданные и политики безопасности
└── Network и Storage конфигурации
```

**Controller Manager** - поддерживает желаемое состояние
```go
// Псевдокод контроллера
for {
    currentState := getCurrentState()
    desiredState := getDesiredState()
    
    if currentState != desiredState {
        reconcile(currentState, desiredState)
    }
    
    sleep(reconciliationInterval)
}
```

**Scheduler** - размещает Pod'ы на узлах
```
Scheduler учитывает:
├── Ресурсы узлов (CPU, память)
├── Affinity/Anti-affinity правила
├── Taints и Tolerations
└── Политики размещения
```

### Worker Nodes: Исполнители

**kubelet** - агент на каждом узле
```bash
# kubelet отвечает за:
# - Запуск контейнеров
# - Мониторинг health checks
# - Отчеты в API Server
# - Управление volumes
```

**kube-proxy** - сетевой прокси
```
kube-proxy обеспечивает:
├── Load balancing между Pod'ами
├── Service discovery
├── Network Address Translation (NAT)
└── Firewall rules
```

**Container Runtime** - выполняет контейнеры
```
Поддерживаемые runtime:
├── Docker (через dockershim, deprecated)
├── containerd (рекомендуется)
├── CRI-O
└── Kata Containers (для безопасности)
```

---

## Основные объекты и ресурсы

### Pod: Минимальная единица развертывания

**Что такое Pod:**
```
Pod = Группа тесно связанных контейнеров
├── Общая сеть (localhost)
├── Общие volumes
├── Единый lifecycle
└── Обычно 1 основной контейнер + helper'ы
```

**Простейший Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: app
    image: nginx:1.21
    ports:
    - containerPort: 80
```

**Pod с несколькими контейнерами (sidecar pattern):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-with-logging
spec:
  containers:
  - name: web-app
    image: myapp:latest
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: logs
      mountPath: /var/log
      
  - name: log-shipper
    image: fluent/fluent-bit:latest
    volumeMounts:
    - name: logs
      mountPath: /var/log
      readOnly: true
      
  volumes:
  - name: logs
    emptyDir: {}
```

**Когда использовать Pod напрямую:**
- Отладка и тестирование
- Одноразовые задачи
- В 99% случаев лучше использовать Deployment

### ReplicaSet: Управление репликами

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: nginx:1.21
```

**Как работает ReplicaSet:**
```
Желаемое состояние: 3 реплики
Текущее состояние: 2 реплики

ReplicaSet Controller:
├── Обнаруживает несоответствие
├── Создает недостающий Pod
└── Мониторит до достижения 3 реплик
```

### Deployment: Декларативные обновления

**Почему Deployment лучше ReplicaSet:**
```
ReplicaSet: Управляет репликами
Deployment: Управляет ReplicaSet + стратегии обновления
```

**Базовый Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

**Rolling Update стратегия:**
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Максимум +1 Pod во время обновления
      maxUnavailable: 0  # Минимум доступных Pod'ов
```

**Как происходит обновление:**
```
Исходное состояние: nginx:1.20 (3 реплики)
Целевое состояние:  nginx:1.21 (3 реплики)

Шаг 1: Создается 1 новый Pod (nginx:1.21)
├── Всего Pod'ов: 4 (3 старых + 1 новый)
└── Ждем готовности нового Pod'а

Шаг 2: Удаляется 1 старый Pod (nginx:1.20)
├── Всего Pod'ов: 3 (2 старых + 1 новый)

Шаг 3-4: Повторяем процесс
└── Результат: 3 новых Pod'а (nginx:1.21)
```

### Service: Сетевой доступ к приложениям

**Проблема без Service:**
```
Pod'ы имеют динамические IP:
Pod-1: 10.244.1.5  ← может измениться при перезапуске
Pod-2: 10.244.1.8  ← может быть на другом узле
Pod-3: 10.244.2.3  ← как к ним обращаться стабильно?
```

**Service как стабильная точка доступа:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP  # По умолчанию
```

**Типы Service:**

**1. ClusterIP (внутренний доступ):**
```yaml
type: ClusterIP
# Доступ только внутри кластера
# Получает внутренний IP: 10.96.0.1
```

**2. NodePort (доступ извне через порт узла):**
```yaml
type: NodePort
ports:
- port: 80
  targetPort: 80
  nodePort: 30080  # Порт на каждом узле
# Доступ: http://node-ip:30080
```

**3. LoadBalancer (облачный балансировщик):**
```yaml
type: LoadBalancer
# Создает внешний LoadBalancer (AWS ALB, GCP LB)
# Получает публичный IP
```

**4. ExternalName (DNS алиас):**
```yaml
type: ExternalName
externalName: database.company.com
# Перенаправляет запросы на внешний сервис
```

---

## Управление приложениями

### StatefulSet: Для stateful приложений

**Отличия от Deployment:**
```
Deployment:               StatefulSet:
├── Pod'ы взаимозаменяемы ├── Каждый Pod уникален
├── Случайные имена       ├── Предсказуемые имена
├── Любой порядок запуска ├── Упорядоченный запуск
└── Временные данные      └── Persistent storage
```

**Пример StatefulSet для базы данных:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "password"
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

**Результат:**
```
Pod'ы создаются по порядку:
├── mysql-0 (первый, master)
├── mysql-1 (после готовности mysql-0)
└── mysql-2 (после готовности mysql-1)

Каждый получает:
├── Уникальное DNS имя: mysql-0.mysql.default.svc.cluster.local
├── Отдельный PVC: data-mysql-0, data-mysql-1, data-mysql-2
└── Стабильную идентичность при перезапуске
```

### DaemonSet: По одному Pod'у на узел

**Использование:**
- Логирование (Fluentd, Filebeat)
- Мониторинг (Node Exporter)
- Сетевые компоненты (CNI)
- Сбор метрик узлов

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /host/sys
          readOnly: true
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
      hostNetwork: true
      hostPID: true
```

### Job и CronJob: Задачи и расписание

**Job для одноразовых задач:**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-migration
spec:
  completions: 1
  parallelism: 1
  template:
    spec:
      containers:
      - name: migration
        image: myapp:migrate
        command: ["python", "migrate.py"]
      restartPolicy: Never
  backoffLimit: 3
```

**CronJob для задач по расписанию:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-job
spec:
  schedule: "0 2 * * *"  # Каждый день в 2:00
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: postgres:13
            command:
            - sh
            - -c
            - pg_dump $DATABASE_URL > /backup/$(date +%Y%m%d).sql
            env:
            - name: DATABASE_URL
              value: "postgresql://user:pass@db:5432/mydb"
          restartPolicy: OnFailure
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
```

---

## Конфигурация и секреты

### ConfigMap: Конфигурационные данные

**Проблема без ConfigMap:**
```dockerfile
# Хардкод в образе - плохо
ENV DATABASE_URL=postgresql://localhost/prod
ENV API_KEY=secret123
ENV DEBUG=false

# Результат: разные образы для dev/test/prod
```

**Решение с ConfigMap:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://db:5432/myapp"
  debug_level: "info"
  feature_flags: "new_ui=true,cache=false"
  # Можно хранить целые файлы:
  nginx.conf: |
    server {
        listen 80;
        location / {
            proxy_pass http://backend:8080;
        }
    }
```

**Использование ConfigMap в Pod'е:**

**1. Как переменные окружения:**
```yaml
spec:
  containers:
  - name: app
    image: myapp:latest
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    # Или загрузить все ключи:
    envFrom:
    - configMapRef:
        name: app-config
```

**2. Как файлы:**
```yaml
spec:
  containers:
  - name: nginx
    image: nginx:latest
    volumeMounts:
    - name: config
      mountPath: /etc/nginx/nginx.conf
      subPath: nginx.conf
  volumes:
  - name: config
    configMap:
      name: app-config
```

### Secrets: Чувствительные данные

**Создание Secret:**
```bash
# Из командной строки:
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secretpassword

# Из файла:
kubectl create secret generic ssl-secret \
  --from-file=tls.crt=path/to/cert.crt \
  --from-file=tls.key=path/to/cert.key
```

**Secret в YAML (base64 encoded):**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=     # admin
  password: c2VjcmV0UGFzcw==  # secretPass
```

**Использование Secret:**
```yaml
spec:
  containers:
  - name: app
    image: myapp:latest
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    volumeMounts:
    - name: ssl-certs
      mountPath: /etc/ssl/certs
      readOnly: true
  volumes:
  - name: ssl-certs
    secret:
      secretName: ssl-secret
```

**Типы Secret:**
```yaml
# Generic (произвольные данные)
type: Opaque

# Docker registry credentials
type: kubernetes.io/dockerconfigjson

# TLS сертификаты
type: kubernetes.io/tls

# SSH ключи
type: kubernetes.io/ssh-auth

# Service Account токены
type: kubernetes.io/service-account-token
```

### Лучшие практики конфигурации

**1. Разделение по средам:**
```
configs/
├── base/
│   ├── configmap.yaml
│   └── deployment.yaml
├── dev/
│   └── config-override.yaml
├── staging/
│   └── config-override.yaml
└── prod/
    └── config-override.yaml
```

**2. Immutable ConfigMaps:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-v2
immutable: true  # Нельзя изменить, только пересоздать
data:
  version: "2.0"
```

**3. Обновление конфигурации:**
```bash
# Изменение ConfigMap не перезапускает Pod'ы автоматически
kubectl patch deployment myapp -p \
  '{"spec":{"template":{"metadata":{"annotations":{"configVersion":"2"}}}}}'
```

---

## Сетевое взаимодействие

### Kubernetes Networking Model

**Основные принципы:**
```
1. Каждый Pod получает уникальный IP
2. Pod'ы могут общаться напрямую без NAT
3. Узлы могут общаться с Pod'ами без NAT
4. IP, который видит Pod изнутри = IP, который видят другие
```

**Сетевая архитектура:**
```
┌─────────────────────────────────────────┐
│                Cluster                  │
│  ┌─────────────┐    ┌─────────────────┐ │
│  │    Node 1   │    │     Node 2      │ │
│  │ Pod Network │    │   Pod Network   │ │
│  │ 10.244.1.0  │    │   10.244.2.0   │ │
│  │             │    │                 │ │
│  │ ┌─────┐     │    │     ┌─────┐     │ │
│  │ │Pod A│     │◄──►│     │Pod B│     │ │
│  │ │10.1 │     │    │     │10.5 │     │ │
│  │ └─────┘     │    │     └─────┘     │ │
│  └─────────────┘    └─────────────────┘ │
└─────────────────────────────────────────┘
```

### CNI (Container Network Interface)

**Популярные CNI плагины:**

**Flannel (простой):**
```yaml
# Простая overlay сеть
# Хорошо для начинающих
# Ограниченные возможности политик
```

**Calico (продвинутый):**
```yaml
# BGP-based networking
# Network Policies
# Высокая производительность
```

**Cilium (современный):**
```yaml
# eBPF-based
# Service Mesh интеграция
# Продвинутые security policies
```

### Service Discovery

**DNS в Kubernetes:**
```
Service: my-service в namespace: my-namespace

Доступен по адресам:
├── my-service (внутри того же namespace)
├── my-service.my-namespace
├── my-service.my-namespace.svc
└── my-service.my-namespace.svc.cluster.local

Pod: my-pod в namespace: my-namespace
├── my-pod.my-service.my-namespace.svc.cluster.local
```

**Пример использования:**
```go
// В приложении можно использовать DNS имена
db_connection := "postgresql://postgres:5432/mydb"
redis_url := "redis://redis-service:6379"
api_endpoint := "http://user-service.api.svc.cluster.local:8080"
```

### Ingress: HTTP(S) роутинг

**Проблема с Service NodePort/LoadBalancer:**
```
Каждый сервис → отдельный LoadBalancer → $$$ за каждый IP
```

**Решение с Ingress:**
```
1 LoadBalancer → Ingress Controller → множество сервисов
```

**Простой Ingress:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

**Продвинутый Ingress с путями:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  tls:
  - hosts:
    - api.example.com
    secretName: api-tls
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /api/v1(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: api-v1-service
            port:
              number: 8080
      - path: /api/v2(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: api-v2-service
            port:
              number: 8080
```

**Результат:**
```
https://api.example.com/api/v1/users → api-v1-service:8080/users
https://api.example.com/api/v2/posts → api-v2-service:8080/posts
```

### Network Policies: Сетевая безопасность

**По умолчанию: все разрешено**
```
Pod A → Pod B ✅
Pod B → Pod C ✅
External → Pod A ✅
```

**Deny All политика:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

**Разрешить только определенные соединения:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-netpol
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  - to: []  # DNS queries
    ports:
    - protocol: UDP
      port: 53
```

**Результат:**
```
✅ frontend → api:8080
✅ api → database:5432
✅ api → DNS:53
❌ api → external-service
❌ other-app → api
```

---

## Хранение данных

### Типы хранилищ в Kubernetes

```
Ephemeral (временные):        Persistent (постоянные):
├── emptyDir                  ├── PersistentVolume
├── hostPath                  ├── Cloud storage (EBS, GCE PD)
└── configMap/secret          └── Network storage (NFS, Ceph)
```

### Volumes: Базовые типы

**emptyDir - временное хранилище:**
```yaml
spec:
  containers:
  - name: app
    volumeMounts:
    - name: cache
      mountPath: /tmp/cache
  - name: log-collector
    volumeMounts:
    - name: cache
      mountPath: /logs
  volumes:
  - name: cache
    emptyDir:
      sizeLimit: 1Gi
```

**hostPath - файлы узла:**
```yaml
volumes:
- name: docker-sock
  hostPath:
    path: /var/run/docker.sock
    type: Socket
# Используется для мониторинга Docker
```

**configMap и secret как volumes:**
```yaml
volumes:
- name: config
  configMap:
    name: app-config
    items:
    - key: app.properties
      path: application.properties
- name: certs
  secret:
    secretName: tls-secret
    defaultMode: 0400
```

### Persistent Volumes (PV) и Claims (PVC)

**Концепция:**
```
Developer создает PVC     →  "Мне нужно 10GB storage"
Kubernetes находит PV     →  "Есть подходящий PV"
Pod использует PVC        →  "Монтирую volume по пути"
```

**PersistentVolume (создает админ):**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-pv
spec:
  capacity:
    storage: 20Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: fast-ssd
  hostPath:
    path: /data/postgres
```

**PersistentVolumeClaim (создает разработчик):**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 15Gi
  storageClassName: fast-ssd
```

**Использование в Pod:**
```yaml
spec:
  containers:
  - name: postgres
    image: postgres:13
    volumeMounts:
    - name: data
      mountPath: /var/lib/postgresql/data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: postgres-pvc
```

### Storage Classes: Динамическое выделение

**Проблема статических PV:**
```
Нужно 100 баз данных → 100 PV создавать вручную
```

**Решение - StorageClass:**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "3000"
  encrypted: "true"
reclaimPolicy: Delete
allowVolumeExpansion: true
```

**PVC с динамическим созданием:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: fast-ssd
  resources:
    requests:
      storage: 50Gi
```

**Что происходит автоматически:**
```
1. PVC создается
2. StorageClass provisioner создает PV автоматически
3. PV привязывается к PVC
4. Pod может использовать PVC
```

### Access Modes

```yaml
accessModes:
- ReadWriteOnce   # RWO: один узел, чтение+запись
- ReadOnlyMany    # ROX: много узлов, только чтение  
- ReadWriteMany   # RWX: много узлов, чтение+запись
- ReadWriteOncePod # RWOP: один Pod (Kubernetes 1.22+)
```

**Выбор режима:**
```
База данных        → ReadWriteOnce
Статические файлы  → ReadOnlyMany
Общий cache        → ReadWriteMany
```

### StatefulSet с Persistent Storage

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-cluster
spec:
  serviceName: mysql
  replicas: 3
  template:
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 100Gi
```

**Результат:**
```
Pod'ы:                    PVC:                     PV:
mysql-cluster-0    →     data-mysql-cluster-0 →   auto-generated-pv-001
mysql-cluster-1    →     data-mysql-cluster-1 →   auto-generated-pv-002
mysql-cluster-2    →     data-mysql-cluster-2 →   auto-generated-pv-003
```

---

## Мониторинг и отладка

### Health Checks: Проверки состояния

**Типы проверок:**
```
Liveness Probe:   "Жив ли контейнер?"
Readiness Probe:  "Готов ли принимать трафик?"
Startup Probe:    "Запустился ли контейнер?" (для медленного старта)
```

**HTTP проверки:**
```yaml
spec:
  containers:
  - name: app
    image: myapp:latest
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      successThreshold: 1
      failureThreshold: 3
```

**TCP и Command проверки:**
```yaml
livenessProbe:
  tcpSocket:
    port: 3306
  initialDelaySeconds: 30

readinessProbe:
  exec:
    command:
    - /bin/sh
    - -c
    - "pg_isready -U postgres"
```

**Что происходит при сбое:**
```
Liveness Probe failed:
├── Контейнер перезапускается
├── Pod статус: Running → Restarting
└── Счетчик restarts увеличивается

Readiness Probe failed:
├── Pod исключается из Service endpoints
├── Трафик не направляется на этот Pod
└── Pod остается запущенным
```

### Мониторинг ресурсов

**Resource Requests и Limits:**
```yaml
spec:
  containers:
  - name: app
    resources:
      requests:        # Гарантированные ресурсы
        cpu: 100m      # 0.1 CPU core
        memory: 128Mi  # 128 MiB
      limits:          # Максимальные ресурсы
        cpu: 500m      # 0.5 CPU core  
        memory: 512Mi  # 512 MiB
```

**QoS классы:**
```
Guaranteed:     requests = limits (высший приоритет)
Burstable:      requests < limits
BestEffort:     нет requests/limits (lowest приоритет)
```

**Мониторинг использования:**
```bash
# Ресурсы узлов
kubectl top nodes

# Ресурсы Pod'ов
kubectl top pods

# Детальная информация
kubectl describe node worker-1
kubectl describe pod my-app-123
```

### Логирование

**Просмотр логов:**
```bash
# Логи текущего контейнера
kubectl logs my-pod

# Логи предыдущего контейнера (после restart)
kubectl logs my-pod --previous

# Логи конкретного контейнера в multi-container Pod
kubectl logs my-pod -c sidecar-container

# Следить за логами в реальном времени
kubectl logs -f my-pod

# Логи за последний час
kubectl logs my-pod --since=1h

# Последние 50 строк
kubectl logs my-pod --tail=50
```

**Структурированное логирование:**
```go
// Хорошая практика - JSON логи
log.Info("Request processed", map[string]interface{}{
    "method": "GET",
    "path": "/api/users",
    "duration": "150ms",
    "status": 200,
    "user_id": "12345",
})
```

**Централизованное логирование (ELK/EFK стек):**
```
Application → stdout/stderr → Node Agent → Elasticsearch → Kibana
                ↓
            Log Shipper
            (Fluentd/Fluent Bit)
```

### Отладка проблем

**Пошаговая диагностика:**

**1. Проверка статуса:**
```bash
kubectl get pods
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl describe pod my-problematic-pod
```

**2. Проверка ресурсов:**
```bash
kubectl top pods
kubectl describe node
# Поиск: OutOfMemory, DiskPressure, PIDPressure
```

**3. Проверка сети:**
```bash
# Проверка Service endpoints
kubectl get endpoints my-service

# Проверка DNS
kubectl exec -it my-pod -- nslookup kubernetes.default

# Проверка доступности
kubectl exec -it my-pod -- wget -qO- http://other-service:8080/health
```

**4. Exec в контейнер:**
```bash
# Интерактивная сессия
kubectl exec -it my-pod -- /bin/bash

# Выполнение команды
kubectl exec my-pod -- ps aux
kubectl exec my-pod -- netstat -tulpn
kubectl exec my-pod -- env
```

**5. Копирование файлов:**
```bash
# Из Pod'а на локальную машину
kubectl cp my-pod:/var/log/app.log ./app.log

# На Pod из локальной машины  
kubectl cp ./config.json my-pod:/etc/app/
```

### Общие проблемы и решения

**ImagePullBackOff:**
```bash
# Проверка образа
kubectl describe pod my-pod
# Часто: неправильное имя образа, отсутствие доступа к registry

# Решение:
# 1. Проверить имя образа
# 2. Проверить imagePullSecrets
# 3. Проверить доступность registry
```

**CrashLoopBackOff:**
```bash
# Контейнер постоянно падает
kubectl logs my-pod --previous
# Часто: ошибка в приложении, неправильная конфигурация

# Решение:
# 1. Проверить логи
# 2. Проверить health checks
# 3. Проверить ресурсы (requests/limits)
```

**Pending состояние:**
```bash
kubectl describe pod my-pod
# Часто: нет ресурсов на узлах, проблемы с tolerations/affinity

# Решение:
# 1. Проверить ресурсы узлов: kubectl describe nodes
# 2. Проверить taints: kubectl describe node | grep Taints
# 3. Масштабировать кластер
```

---

## Безопасность

### Authentication и Authorization

**Модель безопасности Kubernetes:**
```
Request → Authentication → Authorization → Admission Control → API Server
    ↓           ↓              ↓               ↓
 "Кто?"    "Кто такой?"   "Что можно?"   "Валидация"
```

**Методы аутентификации:**
```
1. Service Accounts (для Pod'ов)
2. X.509 сертификаты (для пользователей)  
3. Bearer tokens
4. OpenID Connect (OIDC)
5. External auth (LDAP, AD)
```

### Service Accounts

**Создание Service Account:**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: api-service-account
  namespace: default
automountServiceAccountToken: true
```

**Использование в Pod:**
```yaml
spec:
  serviceAccountName: api-service-account
  containers:
  - name: api
    image: myapi:latest
```

**Что получает Pod:**
```bash
# Внутри Pod'а доступны:
ls /var/run/secrets/kubernetes.io/serviceaccount/
├── token      # JWT токен для API
├── ca.crt     # CA certificate
└── namespace  # текущий namespace

# Использование из приложения:
curl -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
     --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
     https://kubernetes.default.svc/api/v1/namespaces/default/pods
```

### RBAC (Role-Based Access Control)

**Концепция:**
```
Subject (кто?)  +  Verb (что делать?)  +  Resource (с чем?)  =  Permission
    ↓                     ↓                      ↓
ServiceAccount         get, list             pods, services
    User               create, update        deployments
    Group              delete                secrets
```

**Role для namespace:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "create", "update"]
```

**ClusterRole для всего кластера:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["metrics.k8s.io"]
  resources: ["nodes", "pods"]
  verbs: ["get", "list"]
```

**RoleBinding - привязка Role к Subject:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: api-service-account
  namespace: default
- kind: User
  name: jane@company.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**ClusterRoleBinding:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: monitoring-binding
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: monitoring
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

### Security Contexts

**Pod Security Context:**
```yaml
spec:
  securityContext:
    runAsUser: 1000          # Запуск от пользователя 1000
    runAsGroup: 1000         # Группа 1000
    runAsNonRoot: true       # Запрет запуска от root
    fsGroup: 2000            # Группа для volumes
    supplementalGroups: [3000, 4000]
  containers:
  - name: app
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL                # Удалить все capabilities
        add:
        - NET_BIND_SERVICE   # Добавить только необходимые
```

**Практический пример безопасного контейнера:**
```yaml
spec:
  containers:
  - name: secure-app
    image: myapp:latest
    securityContext:
      runAsNonRoot: true
      runAsUser: 10001
      readOnlyRootFilesystem: true
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: cache
      mountPath: /var/cache/app
  volumes:
  - name: tmp
    emptyDir: {}
  - name: cache
    emptyDir: {}
```

### Pod Security Standards

**Pod Security Standards (замена Pod Security Policies):**

**Privileged** - без ограничений
**Baseline** - минимальные ограничения
**Restricted** - максимальные ограничения

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: secure-apps
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

**Что блокирует Restricted режим:**
```yaml
# ❌ Запрещено:
- hostNetwork: true
- hostPID: true
- hostIPC: true
- privileged: true
- runAsUser: 0

# ✅ Требуется:
- runAsNonRoot: true
- readOnlyRootFilesystem: true
- allowPrivilegeEscalation: false
- capabilities.drop: ["ALL"]
```

### Secrets Management

**Типы секретов:**
```yaml
# Docker registry
kubectl create secret docker-registry regcred \
  --docker-server=myregistry.io \
  --docker-username=user \
  --docker-password=pass \
  --docker-email=user@company.com

# TLS сертификаты
kubectl create secret tls tls-secret \
  --cert=path/to/cert.crt \
  --key=path/to/cert.key

# SSH ключи
kubectl create secret generic ssh-key \
  --from-file=ssh-privatekey=~/.ssh/id_rsa \
  --from-file=ssh-publickey=~/.ssh/id_rsa.pub
```

**Лучшие практики для секретов:**

**1. Encryption at rest:**
```bash
# Включение шифрования etcd
kube-apiserver --encryption-provider-config=/etc/kubernetes/encryption.yaml
```

**2. Ротация секретов:**
```bash
# Автоматическая ротация
kubectl patch secret my-secret -p='{"data":{"password":"'$(echo -n newpass | base64)'"}}'

# Обновление Deployment для использования нового секрета
kubectl patch deployment myapp -p \
  '{"spec":{"template":{"metadata":{"annotations":{"secret-version":"2"}}}}}'
```

**3. External Secret Management:**
```yaml
# Интеграция с AWS Secrets Manager, HashiCorp Vault, etc.
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-backend
spec:
  provider:
    vault:
      server: "https://vault.company.com"
      path: "secret"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "myapp"
```

### Network Security

**Default Deny Network Policy:**
```yaml
# Запретить весь трафик по умолчанию
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

**Микросегментация:**
```yaml
# Разрешить только необходимые соединения
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-policy
spec:
  podSelector:
    matchLabels:
      tier: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
    - namespaceSelector:
        matchLabels:
          name: monitoring
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          tier: database
    ports:
    - protocol: TCP
      port: 5432
  - to: []  # DNS
    ports:
    - protocol: UDP
      port: 53
```

---

## Автоматизация и CI/CD

### Helm: Package Manager для Kubernetes

**Зачем нужен Helm:**
```
Без Helm:                     С Helm:
├── 10+ YAML файлов           ├── 1 команда установки
├── Ручное управление         ├── Версионирование релизов
├── Сложность обновлений      ├── Откат одной командой
└── Дублирование конфигов     └── Параметризация шаблонов
```

**Структура Helm Chart:**
```
mychart/
├── Chart.yaml              # Метаданные чарта
├── values.yaml             # Значения по умолчанию
├── charts/                 # Зависимости
├── templates/              # Kubernetes шаблоны
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── _helpers.tpl        # Вспомогательные шаблоны
└── tests/                  # Тесты чарта
```

**Chart.yaml:**
```yaml
apiVersion: v2
name: myapp
description: A Helm chart for MyApp
type: application
version: 0.1.0
appVersion: "1.0"
dependencies:
- name: postgresql
  version: 12.1.2
  repository: https://charts.bitnami.com/bitnami
```

**values.yaml:**
```yaml
image:
  repository: myapp
  tag: "latest"
  pullPolicy: IfNotPresent

replicaCount: 3

service:
  type: ClusterIP
  port: 80
  targetPort: 8080

ingress:
  enabled: true
  className: nginx
  hosts:
  - host: myapp.example.com
    paths:
    - path: /
      pathType: Prefix

resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"  
    cpu: "500m"

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
```

**templates/deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
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
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: {{ .Values.service.targetPort }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
```

**Команды Helm:**
```bash
# Создание чарта
helm create myapp

# Проверка шаблонов
helm template myapp ./myapp

# Установка
helm install myrelease ./myapp

# Обновление
helm upgrade myrelease ./myapp --values prod-values.yaml

# Откат
helm rollback myrelease 1

# Список релизов
helm list

# Удаление
helm uninstall myrelease
```

### GitOps с ArgoCD

**Принципы GitOps:**
```
1. Git как источник истины
2. Декларативное описание инфраструктуры
3. Автоматическая синхронизация
4. Обозримость изменений через Git
```

**Архитектура GitOps:**
```
Developer → Git Repository → ArgoCD → Kubernetes Cluster
    ↓            ↓              ↓           ↓
 git push    YAML configs   Sync Agent   Apply changes
```

**ArgoCD Application:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/company/k8s-manifests
    targetRevision: HEAD
    path: apps/myapp/overlays/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp-prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

**Структура GitOps репозитория:**
```
k8s-manifests/
├── apps/
│   └── myapp/
│       ├── base/                    # Базовые манифесты
│       │   ├── deployment.yaml
│       │   ├── service.yaml
│       │   └── kustomization.yaml
│       └── overlays/               # Окружения
│           ├── dev/
│           │   ├── kustomization.yaml
│           │   └── patch-replica.yaml
│           ├── staging/
│           └── prod/
└── infrastructure/
    ├── ingress-controller/
    ├── monitoring/
    └── cert-manager/
```

### CI/CD Pipeline

**Полный CI/CD процесс:**
```mermaid
graph LR
    A[Code Push] --> B[CI: Build & Test]
    B --> C[Build Docker Image]
    C --> D[Push to Registry]
    D --> E[Update GitOps Repo]
    E --> F[ArgoCD Sync]
    F --> G[Deploy to K8s]
    G --> H[Health Checks]
    H --> I[Notify Team]
```

**GitHub Actions пример:**
```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    - name: Login to Docker Registry
      uses: docker/login-action@v2
      with:
        registry: myregistry.io
        username: ${{ secrets.REGISTRY_USERNAME }}
        password: ${{ secrets.REGISTRY_PASSWORD }}
    
    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: myregistry.io/myapp:${{ github.sha }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
    
    - name: Update GitOps repo
      run: |
        git clone https://github.com/company/k8s-manifests.git
        cd k8s-manifests
        sed -i "s/image: myregistry.io\/myapp:.*/image: myregistry.io\/myapp:${{ github.sha }}/" \
          apps/myapp/base/deployment.yaml
        git add .
        git commit -m "Update myapp to ${{ github.sha }}"
        git push
```

### Стратегии развертывания

**Rolling Update (по умолчанию):**
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%        # Максимум +25% Pod'ов
      maxUnavailable: 25%  # Максимум -25% Pod'ов
```

**Recreate (простое обновление):**
```yaml
spec:
  strategy:
    type: Recreate
# Все старые Pod'ы удаляются, затем создаются новые
# Простое, но с downtime
```

**Blue-Green Deployment:**
```bash
# 1. Создаем Green версию
kubectl apply -f green-deployment.yaml

# 2. Ждем готовности Green
kubectl wait --for=condition=available deployment/myapp-green

# 3. Переключаем Service на Green
kubectl patch service myapp -p '{"spec":{"selector":{"version":"green"}}}'

# 4. Удаляем Blue версию
kubectl delete deployment myapp-blue
```

**Canary Deployment с Flagger:**
```yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: myapp
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  service:
    port: 80
    targetPort: 8080
  analysis:
    interval: 1m
    threshold: 5
    maxWeight: 50
    stepWeight: 10
    metrics:
    - name: request-success-rate
      threshold: 99
      interval: 1m
    - name: request-duration
      threshold: 500
      interval: 1m
```

### Автомасштабирование

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
    name: myapp
  minReplicas: 3
  maxReplicas: 20
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
  - type: Pods
    pods:
      metric:
        name: custom_metric
      target:
        type: AverageValue
        averageValue: "100"
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 100
        periodSeconds: 60
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
    name: myapp
  updatePolicy:
    updateMode: "Auto"  # Off, Initial, Auto
  resourcePolicy:
    containerPolicies:
    - containerName: app
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 4Gi
      controlledResources: ["cpu", "memory"]
```

**Cluster Autoscaler:**
```yaml
# Автоматическое масштабирование узлов
# Настраивается на уровне облачного провайдера
# AWS: Auto Scaling Groups
# GCP: Managed Instance Groups
# Azure: Virtual Machine Scale Sets

apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  template:
    spec:
      containers:
      - image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.21.0
        name: cluster-autoscaler
        command:
        - ./cluster-autoscaler
        - --v=4
        - --stderrthreshold=info
        - --cloud-provider=aws
        - --skip-nodes-with-local-storage=false
        - --expander=least-waste
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/my-cluster
```

---

## Production-ready практики

### Resource Management

**Правильная настройка ресурсов:**
```yaml
# Антипаттерн: без requests/limits
spec:
  containers:
  - name: app
    image: myapp:latest
    # ❌ Проблемы:
    # - Нет гарантий ресурсов
    # - Может "съесть" весь узел
    # - Непредсказуемая производительность

# ✅ Правильно: с requests и limits
spec:
  containers:
  - name: app
    image: myapp:latest
    resources:
      requests:
        cpu: 100m      # Гарантированно 0.1 CPU
        memory: 256Mi  # Гарантированно 256 MiB
      limits:
        cpu: 500m      # Максимум 0.5 CPU
        memory: 512Mi  # Максимум 512 MiB
```

**Расчет ресурсов:**
```bash
# Мониторинг текущего использования
kubectl top pods
kubectl top nodes

# Детальная информация
kubectl describe node worker-1 | grep -A 5 "Allocated resources"

# Правило thumb:
# requests = средняя нагрузка
# limits = пиковая нагрузка + 20-30% запас
```

**Resource Quotas для namespace:**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: production
spec:
  hard:
    requests.cpu: "10"        # Всего CPU requests
    requests.memory: 20Gi     # Всего memory requests
    limits.cpu: "20"          # Всего CPU limits
    limits.memory: 40Gi       # Всего memory limits
    persistentvolumeclaims: "10"  # Количество PVC
    pods: "50"                # Максимум Pod'ов
    services: "10"            # Максимум Services
```

### High Availability

**Multi-zone развертывание:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 6
  template:
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - myapp
              topologyKey: topology.kubernetes.io/zone
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - myapp
            topologyKey: kubernetes.io/hostname
```

**PodDisruptionBudget:**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2  # или maxUnavailable: 1
  selector:
    matchLabels:
      app: myapp
```

**Результат:**
```
# Kubernetes гарантирует:
# - Минимум 2 Pod'а всегда доступны
# - Блокирует eviction если остается меньше 2
# - Учитывается при обновлениях узлов
```

### Мониторинг и Observability

**Prometheus + Grafana стек:**
```yaml
# ServiceMonitor для Prometheus
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: myapp-metrics
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

**Пример метрик в приложении:**
```go
// Go приложение с метриками
package main

import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    httpRequests = prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total HTTP requests",
        },
        []string{"method", "endpoint", "status"},
    )
    
    httpDuration = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name: "http_request_duration_seconds",
            Help: "HTTP request duration",
        },
        []string{"method", "endpoint"},
    )
)

func init() {
    prometheus.MustRegister(httpRequests)
    prometheus.MustRegister(httpDuration)
}

// Middleware для сбора метрик
func metricsMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        
        next.ServeHTTP(w, r)
        
        duration := time.Since(start).Seconds()
        httpDuration.WithLabelValues(r.Method, r.URL.Path).Observe(duration)
        httpRequests.WithLabelValues(r.Method, r.URL.Path, "200").Inc()
    })
}
```

**Grafana Dashboard для Kubernetes:**
```json
{
  "dashboard": {
    "title": "Kubernetes Application Metrics",
    "panels": [
      {
        "title": "Pod CPU Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(container_cpu_usage_seconds_total{pod=~\"myapp-.*\"}[5m])",
            "legendFormat": "{{pod}}"
          }
        ]
      },
      {
        "title": "Pod Memory Usage", 
        "type": "graph",
        "targets": [
          {
            "expr": "container_memory_usage_bytes{pod=~\"myapp-.*\"}",
            "legendFormat": "{{pod}}"
          }
        ]
      },
      {
        "title": "HTTP Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{endpoint}}"
          }
        ]
      }
    ]
  }
}
```

**Алерты в Prometheus:**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: myapp-alerts
spec:
  groups:
  - name: myapp.rules
    rules:
    - alert: PodCrashLooping
      expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: "Pod is crash looping"
        description: "Pod {{ $labels.pod }} is restarting frequently"
        
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: "High error rate detected"
        
    - alert: PodMemoryUsageHigh
      expr: container_memory_usage_bytes / container_spec_memory_limit_bytes > 0.9
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Pod memory usage is high"
```

### Backup и Disaster Recovery

**Velero для backup кластера:**
```bash
# Установка Velero
kubectl apply -f https://github.com/vmware-tanzu/velero/releases/download/v1.9.0/velero-v1.9.0-linux-amd64.tar.gz

# Настройка backup в S3
velero install \
    --provider aws \
    --plugins velero/velero-plugin-for-aws:v1.5.0 \
    --bucket my-backup-bucket \
    --secret-file ./credentials-velero \
    --backup-location-config region=us-west-2
```

**Создание backup:**
```yaml
apiVersion: velero.io/v1
kind: Backup
metadata:
  name: daily-backup
spec:
  includedNamespaces:
  - production
  - staging
  excludedResources:
  - events
  - events.events.k8s.io
  storageLocation: default
  ttl: 720h0m0s  # 30 дней
```

**Restore из backup:**
```bash
# Список backup'ов
velero backup get

# Восстановление
velero restore create --from-backup daily-backup-20231201
```

**Database backup в Kubernetes:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: postgres-backup
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: postgres-backup
            image: postgres:13
            command:
            - /bin/bash
            - -c
            - |
              PGPASSWORD=$POSTGRES_PASSWORD pg_dump \
                -h $POSTGRES_HOST \
                -U $POSTGRES_USER \
                -d $POSTGRES_DB \
                | gzip > /backup/backup-$(date +%Y%m%d-%H%M%S).sql.gz
              
              # Загрузка в S3
              aws s3 cp /backup/backup-$(date +%Y%m%d-%H%M%S).sql.gz \
                s3://my-db-backups/postgres/
            env:
            - name: POSTGRES_HOST
              value: postgres-service
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: username
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: password
            volumeMounts:
            - name: backup-storage
              mountPath: /backup
          volumes:
          - name: backup-storage
            emptyDir: {}
          restartPolicy: OnFailure
```

### Performance Optimization

**Оптимизация образов:**
```dockerfile
# ❌ Плохо: большой образ
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y python3 python3-pip
COPY requirements.txt .
RUN pip3 install -r requirements.txt
COPY . .
CMD ["python3", "app.py"]

# ✅ Лучше: multi-stage build
FROM python:3.9-slim as builder
COPY requirements.txt .
RUN pip install --user -r requirements.txt

FROM python:3.9-slim
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]

# ✅ Еще лучше: distroless
FROM gcr.io/distroless/python3
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["app.py"]
```

**Node Affinity для оптимизации размещения:**
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: node-type
            operator: In
            values:
            - compute-optimized
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - us-west-2a
```

**Taints и Tolerations:**
```bash
# Назначение taint на узел
kubectl taint nodes worker-1 dedicated=database:NoSchedule

# Удаление taint
kubectl taint nodes worker-1 dedicated=database:NoSchedule-
```

```yaml
# Pod с toleration
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "database"
    effect: "NoSchedule"
  nodeSelector:
    node-type: "database"
```

---

## Продвинутые темы

### Custom Resources и Operators

**Зачем нужны CRD:**
```
Kubernetes знает о:          Мы хотим управлять:
├── Pod, Service, Deployment ├── Database
├── ConfigMap, Secret        ├── Certificate
└── Ingress, PV             └── Application (как единое целое)

CRD позволяет расширить API Kubernetes
```

**Простой CRD пример:**
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.example.com
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              engine:
                type: string
                enum: ["postgresql", "mysql"]
              version:
                type: string
              storage:
                type: string
              replicas:
                type: integer
                minimum: 1
                maximum: 5
          status:
            type: object
            properties:
              phase:
                type: string
              endpoint:
                type: string
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
```

**Использование Custom Resource:**
```yaml
apiVersion: example.com/v1
kind: Database
metadata:
  name: user-db
spec:
  engine: postgresql
  version: "13"
  storage: "100Gi"
  replicas: 2
```

**Operator паттерн:**
```go
// Псевдокод Operator'а
func (r *DatabaseReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    // 1. Получить желаемое состояние
    var database Database
    if err := r.Get(ctx, req.NamespacedName, &database); err != nil {
        return ctrl.Result{}, err
    }
    
    // 2. Получить текущее состояние
    currentDeployment := &appsv1.Deployment{}
    err := r.Get(ctx, types.NamespacedName{
        Name: database.Name + "-postgres",
        Namespace: database.Namespace,
    }, currentDeployment)
    
    // 3. Сравнить и привести к желаемому состоянию
    if errors.IsNotFound(err) {
        // Создать новый Deployment
        deployment := r.createPostgresDeployment(&database)
        return ctrl.Result{}, r.Create(ctx, deployment)
    }
    
    // 4. Обновить статус
    database.Status.Phase = "Running"
    database.Status.Endpoint = fmt.Sprintf("%s:5432", database.Name)
    return ctrl.Result{}, r.Status().Update(ctx, &database)
}
```

### Service Mesh (Istio)

**Проблемы микросервисов без Service Mesh:**
```
Service A → Service B: Как обеспечить:
├── Mutual TLS?
├── Circuit breaker?
├── Retry policy?
├── Load balancing?
├── Observability?
└── Access control?

Каждый сервис должен реализовывать это сам
```

**Service Mesh решение:**
```
Service A → Envoy Proxy → Envoy Proxy → Service B
              ↓                ↓
         Control Plane    Control Plane
         (Istiod)         (Istiod)
```

**Установка Istio:**
```bash
# Скачивание Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.18.0
export PATH=$PWD/bin:$PATH

# Установка
istioctl install --set values.defaultRevision=default

# Включение автоматического sidecar injection
kubectl label namespace default istio-injection=enabled
```

**Traffic Management:**
```yaml
# VirtualService для маршрутизации
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
        weight: 90
    - destination:
        host: reviews
        subset: v2
        weight: 10

---
# DestinationRule для load balancing
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    loadBalancer:
      simple: LEAST_CONN
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      connectionPool:
        tcp:
          maxConnections: 10
        http:
          http1MaxPendingRequests: 10
          maxRequestsPerConnection: 2
      circuitBreaker:
        consecutiveErrors: 3
        interval: 30s
        baseEjectionTime: 30s
```

**Security Policies:**
```yaml
# PeerAuthentication для mTLS
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
spec:
  mtls:
    mode: STRICT

---
# AuthorizationPolicy для доступа
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-frontend
spec:
  selector:
    matchLabels:
      app: backend
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/frontend"]
  - to:
    - operation:
        methods: ["GET", "POST"]
```

### Kubernetes Operators

**Популярные Operators:**

**Prometheus Operator:**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: prometheus
spec:
  serviceAccountName: prometheus
  serviceMonitorSelector:
    matchLabels:
      team: frontend
  ruleSelector:
    matchLabels:
      role: alert-rules
  resources:
    requests:
      memory: 400Mi
  enableAdminAPI: false
```

**Cert-Manager для автоматических сертификатов:**
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@company.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx

---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: myapp-tls
spec:
  secretName: myapp-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - myapp.example.com
```

**PostgreSQL Operator:**
```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: postgres-cluster
spec:
  instances: 3
  postgresql:
    parameters:
      max_connections: "200"
      shared_buffers: "256MB"
      effective_cache_size: "1GB"
  bootstrap:
    initdb:
      database: app
      owner: app
      secret:
        name: postgres-credentials
  storage:
    size: 100Gi
    storageClass: fast-ssd
  monitoring:
    enabled: true
```

### Advanced Scheduling

**Pod Priority и Preemption:**
```yaml
# PriorityClass
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
description: "High priority class for critical workloads"

---
# Pod с приоритетом
spec:
  priorityClassName: high-priority
  containers:
  - name: critical-app
    image: myapp:latest
```

**Результат:** Pod'ы с высоким приоритетом могут вытеснить Pod'ы с низким приоритетом

**Advanced Node Affinity:**
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/arch
            operator: In
            values: ["amd64"]
          - key: node.kubernetes.io/instance-type
            operator: In
            values: ["m5.large", "m5.xlarge"]
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values: ["us-west-2a"]
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values: ["myapp"]
        topologyKey: kubernetes.io/hostname
```

### Multi-tenancy

**Namespace-based изоляция:**
```yaml
# Namespace для команды
apiVersion: v1
kind: Namespace
metadata:
  name: team-alpha
  labels:
    team: alpha
    environment: production

---
# ResourceQuota для команды
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-alpha-quota
  namespace: team-alpha
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    persistentvolumeclaims: "10"
    count/deployments.apps: "20"
    count/services: "10"

---
# LimitRange для default'ов
apiVersion: v1
kind: LimitRange
metadata:
  name: team-alpha-limits
  namespace: team-alpha
spec:
  limits:
  - default:
      cpu: 500m
      memory: 512Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    type: Container
```

**Network isolation между командами:**
```yaml
# Deny all для namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: team-alpha
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

---
# Разрешить только внутри namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-namespace
  namespace: team-alpha
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: team-alpha
```

---

## Практические примеры и кейсы

### Полный пример: Веб-приложение с базой данных

**Архитектура:**
```
Internet → Ingress → Frontend Service → Frontend Pods
                         ↓
                    Backend Service → Backend Pods
                         ↓
                    Database Service → PostgreSQL StatefulSet
```

**1. Namespace и базовые ресурсы:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: webapp
  labels:
    environment: production

---
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
  namespace: webapp
type: Opaque
data:
  username: cG9zdGdyZXM=  # postgres
  password: c3VwZXJzZWNyZXQ=  # supersecret
  database: bXlhcHA=  # myapp
```

**2. PostgreSQL StatefulSet:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: webapp
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:13
        env:
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: username
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        - name: POSTGRES_DB
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: database
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
        livenessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - postgres
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - postgres
          initialDelaySeconds: 5
          periodSeconds: 5
  volumeClaimTemplates:
  - metadata:
      name: postgres-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 20Gi

---
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: webapp
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
  clusterIP: None  # Headless service для StatefulSet
```

**3. Backend API:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: backend-config
  namespace: webapp
data:
  DATABASE_URL: "postgresql://postgres:supersecret@postgres:5432/myapp"
  REDIS_URL: "redis://redis:6379"
  LOG_LEVEL: "info"
  API_PORT: "8080"

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
        version: v1
    spec:
      containers:
      - name: backend
        image: mycompany/backend:v1.2.0
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: backend-config
              key: DATABASE_URL
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: backend-config
              key: LOG_LEVEL
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        securityContext:
          runAsNonRoot: true
          runAsUser: 1000
          readOnlyRootFilesystem: true
          allowPrivilegeEscalation: false

---
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: webapp
spec:
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 8080
```

**4. Frontend:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: mycompany/frontend:v1.2.0
        ports:
        - containerPort: 80
        env:
        - name: API_URL
          value: "http://backend/api"
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: webapp
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
```

**5. Ingress:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
  namespace: webapp
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: webapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80
```

**6. HPA для автомасштабирования:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
  namespace: webapp
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  minReplicas: 3
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
```

### Пример миграции с Docker Compose

**Было (docker-compose.yml):**
```yaml
version: '3.8'
services:
  web:
    image: myapp:latest
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
      
  db:
    image: postgres:13
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      
  redis:
    image: redis:6-alpine
    
volumes:
  postgres_data:
```

**Стало (Kubernetes):**
```yaml
# ConfigMap заменяет environment
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_URL: "postgresql://postgres:5432/myapp"
  REDIS_URL: "redis://redis:6379"

---
# Secret для чувствительных данных
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  db-password: cGFzc3dvcmQ=

---
# StatefulSet вместо простого контейнера для БД
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 1
  # ... (как в примере выше)

---
# Deployment для stateless приложения
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3  # Масштабирование!
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: myapp:latest
        ports:
        - containerPort: 3000
        envFrom:
        - configMapRef:
            name: app-config
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: db-password

---
# Service для network доступа
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 3000
  type: LoadBalancer  # Внешний доступ
```

---

## Заключение и дальнейшее изучение

### Ключевые принципы Kubernetes

**1. Декларативность над императивностью:**
```bash
# ❌ Императивный подход
kubectl run nginx --image=nginx
kubectl scale deployment nginx --replicas=3
kubectl expose deployment nginx --port=80

# ✅ Декларативный подход
kubectl apply -f nginx-deployment.yaml
# Файл описывает желаемое состояние, K8s делает остальное
```

**2. Контроллеры и reconciliation loops:**
```
Желаемое состояние (YAML) ← → Текущее состояние (кластер)
                     ↑               ↓
                 Controller следит и исправляет отклонения
```

**3. Всё как код (Infrastructure as Code):**
```
Инфраструктура → Git → CI/CD → Kubernetes
     ↓              ↓      ↓         ↓
   YAML files   Version   Auto   Deployment
               Control   Deploy
```

### Путь к экспертизе

**Новичок (0-3 месяца):**
- ✅ Понимание базовых концепций (Pod, Service, Deployment)
- ✅ Умение создавать простые приложения
- ✅ Работа с kubectl
- ✅ Понимание YAML манифестов

**Практик (3-6 месяцев):**
- ✅ Сложные деплойменты с базами данных
- ✅ Настройка мониторинга и логирования
- ✅ Понимание сетей и storage
- ✅ Базовые принципы безопасности
- ✅ CI/CD интеграция

**Эксперт (6+ месяцев):**
- ✅ Кастомные ресурсы и операторы
- ✅ Service Mesh
- ✅ Multi-cluster management
- ✅ Performance tuning
- ✅ Disaster recovery
- ✅ Compliance и governance

### Следующие шаги

**1. Практические проекты:**
```
Week 1-2:  Деплой простого веб-приложения
Week 3-4:  Добавление базы данных и мониторинга
Week 5-6:  Настройка CI/CD pipeline
Week 7-8:  Реализация high availability
Week 9-10: Security hardening
Week 11-12: Performance optimization
```

**2. Сертификация:**
- **CKAD** (Certified Kubernetes Application Developer) - для разработчиков
- **CKA** (Certified Kubernetes Administrator) - для админов
- **CKS** (Certified Kubernetes Security Specialist) - для security

**3. Специализация:**
```
DevOps Engineer:        Platform Engineer:       Site Reliability Engineer:
├── CI/CD automation    ├── Multi-tenancy       ├── Monitoring & alerting
├── Infrastructure code ├── Developer tools     ├── Incident response
├── Release management  ├── Self-service        ├── Capacity planning
└── Environment mgmt    └── Internal platforms  └── Performance tuning
```

**4. Продвинутые технологии:**
- **Service Mesh**: Istio, Linkerd, Consul Connect
- **GitOps**: ArgoCD, Flux, Jenkins X
- **Observability**: Prometheus, Grafana, Jaeger, OpenTelemetry
- **Security**: Falco, OPA/Gatekeeper, Twistlock/Prisma

### Полезные ресурсы

**Документация:**
- [Kubernetes Official Docs](https://kubernetes.io/docs/)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/)

**Практика:**
- [Katacoda Kubernetes](https://katacoda.com/kubernetes)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)
- [KillerCoda](https://killercoda.com/kubernetes)

**Книги:**
- "Kubernetes in Action" by Marko Lukša
- "Kubernetes Up & Running" by Brendan Burns
- "The DevOps Handbook" by Gene Kim

**Community:**
- [Kubernetes Slack](https://kubernetes.slack.com/)
- [CNCF Community](https://community.cncf.io/)
- [KubeCon + CloudNativeCon](https://www.cncf.io/kubecon-cloudnativecon-events/)

---

*Kubernetes - это не просто инструмент, это платформа для построения cloud-native приложений. Понимание его принципов откроет путь к современной разработке и эксплуатации distributed систем. Успехов в изучении!* 🚀