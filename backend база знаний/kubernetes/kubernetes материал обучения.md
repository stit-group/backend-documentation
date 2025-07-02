# Полное руководство по Kubernetes для Backend разработчиков

## Блок 1: Основы и введение в Kubernetes

### Глава 1: Введение - Зачем нужен Kubernetes?

#### Проблемы масштабирования приложений

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

#### Ключевые преимущества Kubernetes

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

### Глава 2: От Docker к Kubernetes

#### Docker: Основа контейнеризации

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

#### Ограничения Docker в продакшене

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

#### Kubernetes как оркестратор

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

## Блок 2: Архитектура и компоненты

### Глава 3: Архитектура Kubernetes

#### Кластерная архитектура

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

#### Control Plane: Мозг кластера

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

#### Worker Nodes: Исполнители

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

### Глава 4: Основные объекты и ресурсы

#### Pod: Минимальная единица развертывания

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

#### ReplicaSet: Управление репликами

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

#### Deployment: Декларативные обновления

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

#### Service: Сетевой доступ к приложениям

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

## Блок 3: Управление приложениями

### Глава 5: Специализированные контроллеры

#### StatefulSet: Для stateful приложений

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

#### DaemonSet: По одному Pod'у на узел

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

#### Job и CronJob: Задачи и расписание

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

### Глава 6: Конфигурация и секреты

#### ConfigMap: Конфигурационные данные

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

#### Secrets: Чувствительные данные

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

## Блок 4: Сетевое взаимодействие и хранение

### Глава 7: Сетевое взаимодействие

#### Kubernetes Networking Model

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

#### Service Discovery

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

#### Ingress: HTTP(S) роутинг

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

#### Network Policies: Сетевая безопасность

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
```

### Глава 8: Хранение данных

#### Типы хранилищ в Kubernetes

```
Ephemeral (временные):        Persistent (постоянные):
├── emptyDir                  ├── PersistentVolume
├── hostPath                  ├── Cloud storage (EBS, GCE PD)
└── configMap/secret          └── Network storage (NFS, Ceph)
```

#### Volumes: Базовые типы

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

#### Persistent Volumes (PV) и Claims (PVC)

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

#### Storage Classes: Динамическое выделение

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

## Блок 5: Мониторинг и безопасность

### Глава 9: Мониторинг и отладка

#### Health Checks: Проверки состояния

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

#### Мониторинг ресурсов

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

#### Логирование

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

#### Отладка проблем

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

**3. Exec в контейнер:**
```bash
# Интерактивная сессия
kubectl exec -it my-pod -- /bin/bash

# Выполнение команды
kubectl exec my-pod -- ps aux
kubectl exec my-pod -- netstat -tulpn
kubectl exec my-pod -- env
```

### Глава 10: Безопасность

#### Authentication и Authorization

**Модель безопасности Kubernetes:**
```
Request → Authentication → Authorization → Admission Control → API Server
    ↓           ↓              ↓               ↓
 "Кто?"    "Кто такой?"   "Что можно?"   "Валидация"
```

#### Service Accounts

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

#### RBAC (Role-Based Access Control)

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

#### Security Contexts

**Pod Security Context:**
```yaml
spec:
  securityContext:
    runAsUser: 1000          # Запуск от пользователя 1000
    runAsGroup: 1000         # Группа 1000
    runAsNonRoot: true       # Запрет запуска от root
    fsGroup: 2000            # Группа для volumes
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

## Блок 6: Автоматизация и CI/CD

### Глава 11: Helm - Package Manager для Kubernetes

#### Зачем нужен Helm

**Проблемы без Helm:**
```
Без Helm:                     С Helm:
├── 10+ YAML файлов           ├── 1 команда установки
├── Ручное управление         ├── Версионирование релизов
├── Сложность обновлений      ├── Откат одной командой
└── Дублирование конфигов     └── Параметризация шаблонов
```

#### Структура Helm Chart

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

#### Команды Helm

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

### Глава 12: GitOps с ArgoCD

#### Принципы GitOps

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

#### ArgoCD Application

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

#### Структура GitOps репозитория

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

### Глава 13: Автомасштабирование

#### Horizontal Pod Autoscaler (HPA)

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

#### Vertical Pod Autoscaler (VPA)

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

## Блок 7: Production-ready практики и продвинутые темы

### Глава 14: Production-ready практики

#### Resource Management

**Правильная настройка ресурсов:**
```yaml
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

#### High Availability

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

#### Мониторинг и Observability

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
```

### Глава 15: Продвинутые темы

#### Custom Resources и Operators

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

#### Service Mesh (Istio)

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
```

#### Multi-tenancy

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
```

### Глава 16: Практические примеры и заключение

#### Полный пример: Веб-приложение с базой данных

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
  volumeClaimTemplates:
  - metadata:
      name: postgres-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 20Gi
```

#### Ключевые принципы Kubernetes

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

#### Путь к экспертизе

**Новичок (Блоки 1-2):**
- ✅ Понимание базовых концепций (Pod, Service, Deployment)
- ✅ Умение создавать простые приложения
- ✅ Работа с kubectl
- ✅ Понимание YAML манифестов

**Практик (Блоки 3-5):**
- ✅ Сложные деплойменты с базами данных
- ✅ Настройка мониторинга и логирования
- ✅ Понимание сетей и storage
- ✅ Базовые принципы безопасности
- ✅ CI/CD интеграция

**Эксперт (Блоки 6-7):**
- ✅ Кастомные ресурсы и операторы
- ✅ Service Mesh
- ✅ Multi-cluster management
- ✅ Performance tuning
- ✅ Disaster recovery
- ✅ Compliance и governance

*Kubernetes - это не просто инструмент, это платформа для построения cloud-native приложений. Понимание его принципов откроет путь к современной разработке и эксплуатации distributed систем. Успехов в изучении!* 🚀