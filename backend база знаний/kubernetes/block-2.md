# 🔧 Блок 2: Основные объекты Kubernetes

**⏱️ Продолжительность:** 3-4 недели  
**🎯 Цель:** Освоить работу с ключевыми ресурсами Kubernetes

---

## 2.1 Pods: создание, управление, жизненный цикл (5-6 дней)

### Что такое Pod и зачем он нужен

Pod (стручок) — это **минимальная единица развертывания** в Kubernetes. Это не просто контейнер, а группа контейнеров, которые:

- Всегда запускаются на одном узле (Node)
- Разделяют сетевое пространство имен (один IP-адрес)
- Разделяют storage volumes
- Живут и умирают вместе

```
Представьте Pod как "квартиру" для контейнеров:
┌─────────────────────────────────────┐
│             POD (квартира)          │
│  ┌─────────────┐  ┌─────────────┐   │
│  │ Container 1 │  │ Container 2 │   │
│  │             │  │             │   │
│  └─────────────┘  └─────────────┘   │
│         Общая сеть и storage        │
└─────────────────────────────────────┘
```

**Почему именно Pod, а не просто контейнер?**

1. **Atomic unit**: Если приложению нужно несколько тесно связанных процессов
2. **Shared resources**: Контейнеры могут общаться через localhost
3. **Co-location**: Гарантия, что связанные компоненты на одном узле
4. **Simplicity**: Упрощает сетевое взаимодействие и управление

### Анатомия Pod

```
        POD LIFECYCLE
┌─────────────────────────────────────┐
│  Pending → Running → Succeeded      │
│     ↓         ↓         ↓           │
│  Waiting   Ready    Terminated      │
│     ↓         ↓         ↓           │
│  Failed ←─────┴─────── Unknown      │
└─────────────────────────────────────┘
```

**Основные состояния Pod:**
- **Pending**: Pod принят системой, но контейнеры еще не созданы
- **Running**: Pod привязан к узлу и все контейнеры созданы
- **Succeeded**: Все контейнеры успешно завершились
- **Failed**: Все контейнеры завершились, минимум один с ошибкой
- **Unknown**: Состояние Pod не может быть определено

### Multi-container Pods и Sidecar Pattern

**Основные паттерны использования нескольких контейнеров:**

#### 1. Sidecar Pattern
```
Main Application Container + Helper Container
┌──────────────────────────────────────┐
│               POD                    │
│ ┌─────────────┐  ┌─────────────────┐ │
│ │   Web App   │  │  Log Collector  │ │
│ │             │  │   (Sidecar)     │ │
│ │ generates   │──│ collects &      │ │
│ │ logs        │  │ forwards logs   │ │
│ └─────────────┘  └─────────────────┘ │
│        Shared Volume (logs)          │
└──────────────────────────────────────┘
```

#### 2. Ambassador Pattern
```
Main Container + Proxy Container
┌──────────────────────────────────────┐
│               POD                    │
│ ┌─────────────┐  ┌─────────────────┐ │
│ │  Main App   │  │ Proxy/Ambassador│ │
│ │             │  │                 │ │
│ │ localhost:  │──│ handles         │ │
│ │ 8080        │  │ external comms  │ │
│ └─────────────┘  └─────────────────┘ │
└──────────────────────────────────────┘
```

#### 3. Adapter Pattern
```
Main Container + Adapter Container
┌──────────────────────────────────────┐
│               POD                    │
│ ┌─────────────┐  ┌─────────────────┐ │
│ │ Legacy App  │  │   Adapter       │ │
│ │ (old format)│  │ (transforms     │ │
│ │ outputs     │──│ to new format)  │ │
│ │ data        │  │                 │ │
│ └─────────────┘  └─────────────────┘ │
└──────────────────────────────────────┘
```

### Init Containers

Init Containers — это **специальные контейнеры**, которые запускаются и завершаются **до** запуска основных контейнеров приложения.

```
INIT CONTAINERS WORKFLOW
─────────────────────────────
Init Container 1 → Complete ✓
Init Container 2 → Complete ✓
Init Container 3 → Complete ✓
                ↓
    Main Containers Start
```

**Типичные применения:**
- Ожидание запуска зависимых сервисов
- Инициализация базы данных
- Клонирование Git репозитория
- Настройка permissions и конфигурации

### Resource Requests и Limits

Kubernetes позволяет указать **два типа ресурсных ограничений**:

#### Requests vs Limits
```
RESOURCE ALLOCATION
────────────────────────────────────
│    Node Available: 4 CPU, 8GB     │
├────────────────────────────────────┤
│ Pod A: Request 1 CPU, Limit 2 CPU │
│ Pod B: Request 1 CPU, Limit 1 CPU │
│ Pod C: Request 0.5 CPU, No Limit  │
└────────────────────────────────────┘
```

**Request** — минимальные ресурсы, которые гарантированы Pod
**Limit** — максимальные ресурсы, которые Pod может использовать

#### Quality of Service (QoS) классы:

1. **Guaranteed**: requests = limits для всех контейнеров
2. **Burstable**: есть requests, но limits > requests
3. **BestEffort**: нет ни requests, ни limits

```
QoS PRIORITY DURING EVICTION
─────────────────────────────
BestEffort (убираются первыми)
    ↓
Burstable (убираются вторыми)
    ↓
Guaranteed (убираются последними)
```

---

## 2.2 Deployments и ReplicaSets (5-6 дней)

### Проблема голых Pods

Если вы создаете Pod напрямую, то:
- При падении узла Pod исчезает навсегда
- Нет автоматического перезапуска
- Нет возможности масштабирования
- Нет управления версиями

```
NAKED POD PROBLEM
──────────────────
Pod dies → Gone forever ❌

DEPLOYMENT SOLUTION
──────────────────────
Pod dies → ReplicaSet creates new one ✅
```

### Что такое ReplicaSet

ReplicaSet — это контроллер, который **гарантирует**, что определенное количество replica Pod'ов всегда работает.

```
REPLICASET WORKFLOW
─────────────────────────────────
Desired State: 3 replicas
Current State: 2 replicas
Action: Create 1 more pod

┌─────────────────────────────────┐
│          ReplicaSet             │
│                                 │
│  ┌─────┐  ┌─────┐  ┌─────┐     │
│  │Pod 1│  │Pod 2│  │Pod 3│     │
│  └─────┘  └─────┘  └─────┘     │
│     ↑        ↑        ↑        │
│     └────────┼────────┘        │
│              │                 │
│       Label Selector           │
│       app=my-app               │
└─────────────────────────────────┘
```

### Deployment: Управление ReplicaSets

Deployment — это **высокоуровневый контроллер**, который управляет ReplicaSets и предоставляет:

- Декларативные обновления
- Rollback к предыдущим версиям
- Масштабирование
- Паузу и возобновление развертывания

```
DEPLOYMENT HIERARCHY
────────────────────────
Deployment
    ↓ manages
ReplicaSet (v1) → Pods (v1)
ReplicaSet (v2) → Pods (v2) ← current
ReplicaSet (v3) → (empty)   ← future
```

### Rolling Updates: плавное обновление

Rolling Update позволяет обновлять приложение **без простоя**:

```
ROLLING UPDATE PROCESS
──────────────────────────────────────

Step 1: Start with 3 pods (v1)
[Pod v1] [Pod v1] [Pod v1]

Step 2: Create 1 new pod (v2)
[Pod v1] [Pod v1] [Pod v1] [Pod v2]

Step 3: Terminate 1 old pod
[Pod v1] [Pod v1] [Pod v2]

Step 4: Create another new pod
[Pod v1] [Pod v1] [Pod v2] [Pod v2]

Step 5: Continue until all updated
[Pod v2] [Pod v2] [Pod v2]
```

**Ключевые параметры Rolling Update:**
- **maxUnavailable**: максимум недоступных Pod'ов во время обновления
- **maxSurge**: максимум дополнительных Pod'ов во время обновления

### Стратегии развертывания

#### 1. Recreate Strategy
```
RECREATE DEPLOYMENT
───────────────────────
[Pod v1] [Pod v1] [Pod v1]
    ↓ (kill all)
[ DOWNTIME PERIOD ]
    ↓ (create new)
[Pod v2] [Pod v2] [Pod v2]
```

#### 2. Rolling Update Strategy (по умолчанию)
```
ROLLING UPDATE
──────────────────────
[Pod v1] [Pod v1] [Pod v1]
    ↓ (gradual replacement)
[Pod v2] [Pod v2] [Pod v2]
```

#### 3. Blue-Green (реализуется через Service)
```
BLUE-GREEN DEPLOYMENT
─────────────────────────
Blue Environment:  [Pod v1] [Pod v1] [Pod v1]
Green Environment: [Pod v2] [Pod v2] [Pod v2]
                        ↓
Service switch: Blue → Green
```

### Rollback: откат к предыдущей версии

Kubernetes хранит **историю** всех Deployment'ов:

```
DEPLOYMENT HISTORY
──────────────────────────
Revision 1: image:app:v1.0
Revision 2: image:app:v1.1  ← current
Revision 3: image:app:v1.2  (failed)
                ↓
         Rollback to v1.1
```

### Horizontal Pod Autoscaler (HPA)

HPA автоматически **масштабирует** количество Pod'ов на основе метрик:

```
HPA SCALING LOGIC
─────────────────────────────────
Current CPU: 80%
Target CPU: 50%
Current Replicas: 3
Desired Replicas: 80% / 50% * 3 = 4.8 → 5

┌─────────────────────────────────┐
│               HPA               │
│                                 │
│ Target: 50% CPU                 │
│ Min replicas: 2                 │
│ Max replicas: 10                │
│                                 │
│        ↓ scales ↓               │
│                                 │
│          Deployment             │
│     ┌─────┐  ┌─────┐  ┌─────┐  │
│     │Pod 1│  │Pod 2│  │Pod 3│  │
│     └─────┘  └─────┘  └─────┘  │
└─────────────────────────────────┘
```

**Метрики для HPA:**
- CPU utilization
- Memory utilization
- Custom metrics (через Prometheus adapter)
- External metrics (очереди, внешние API)

---

## 2.3 Services: сетевое взаимодействие (4-5 дней)

### Проблема сетевого взаимодействия в Kubernetes

Pod'ы в Kubernetes:
- Имеют **эфемерные IP-адреса** (меняются при пересоздании)
- Могут находиться на **разных узлах**
- **Масштабируются** (количество меняется)

```
PODS NETWORKING PROBLEM
───────────────────────────────────
Pod 1 (IP: 10.1.1.1) ←─┐
Pod 2 (IP: 10.1.1.2) ←─┤ How to connect?
Pod 3 (IP: 10.1.1.3) ←─┘ IPs change!
```

### Что такое Service

Service — это **стабильная точка входа** для группы Pod'ов. Он предоставляет:
- Стабильный IP-адрес и DNS имя
- Load balancing между Pod'ами
- Service discovery

```
SERVICE ABSTRACTION
─────────────────────────────────────
           Client
             ↓
┌─────────────────────────────────┐
│          Service                │
│     Stable IP: 10.0.0.100       │
│     DNS: my-app.default.svc     │
└─────────────────────────────────┘
             ↓ (load balance)
┌─────────────────────────────────┐
│    Pod 1     Pod 2     Pod 3    │
│ 10.1.1.1   10.1.1.2   10.1.1.3 │
└─────────────────────────────────┘
```

### Типы Services

#### 1. ClusterIP (по умолчанию)
Доступен **только внутри кластера**:

```
CLUSTERIP SERVICE
─────────────────────────────────
┌─────────────────────────────────┐
│           Cluster               │
│                                 │
│  Pod A ──→ Service ──→ Pod B    │
│            (ClusterIP)          │
│                                 │
│  ✅ Internal access             │
│  ❌ External access             │
└─────────────────────────────────┘
```

#### 2. NodePort
Открывает порт на **всех узлах кластера**:

```
NODEPORT SERVICE
─────────────────────────────────────
External Client
       ↓
Node 1:30080 ──┐
Node 2:30080 ──┤ ─→ Service ─→ Pods
Node 3:30080 ──┘

Port Range: 30000-32767
```

#### 3. LoadBalancer
Создает **внешний балансировщик нагрузки** (в облаке):

```
LOADBALANCER SERVICE
────────────────────────────────────
External Client
       ↓
Cloud Load Balancer (External IP)
       ↓
┌─────────────────────────────────┐
│          Cluster                │
│                                 │
│  Service ──→ Pods               │
└─────────────────────────────────┘
```

#### 4. ExternalName
Создает **CNAME запись** для внешнего сервиса:

```
EXTERNALNAME SERVICE
─────────────────────────────────────
Pod requests: my-db.default.svc
       ↓ (CNAME)
External DB: db.example.com:5432
```

### Service Discovery и DNS

Kubernetes автоматически создает **DNS записи** для каждого Service:

```
DNS RESOLUTION
──────────────────────────────────────
Service Name: my-app
Namespace: production
Cluster Domain: cluster.local

Full DNS Name:
my-app.production.svc.cluster.local

Short Forms (from same namespace):
- my-app
- my-app.production
```

### Endpoints и EndpointSlices

Service **не содержит** Pod'ы напрямую. Вместо этого:

```
SERVICE → ENDPOINTS → PODS
─────────────────────────────────
Service (my-app)
    ↓ (selector: app=my-app)
Endpoints Object
    ↓ (contains IP list)
Pod 1: 10.1.1.1:8080
Pod 2: 10.1.1.2:8080
Pod 3: 10.1.1.3:8080
```

**EndpointSlices** — это новая версия Endpoints, которая:
- Лучше масштабируется
- Поддерживает больше протоколов
- Более эффективна для больших кластеров

### Headless Services

Headless Service **не имеет ClusterIP** и возвращает IP-адреса всех Pod'ов:

```
HEADLESS SERVICE
──────────────────────────────────
DNS Query: my-app.default.svc
Response: 
  10.1.1.1  (Pod 1)
  10.1.1.2  (Pod 2)  
  10.1.1.3  (Pod 3)

Use case: StatefulSets, где нужен 
доступ к конкретным Pod'ам
```

---

## 2.4 ConfigMaps и Secrets (3-4 дня)

### Проблема конфигурации в контейнерах

**Традиционный подход**:
- Конфигурация зашита в образ контейнера
- Разные образы для разных сред (dev, staging, prod)
- Сложно менять конфигурацию без пересборки

```
TRADITIONAL CONFIG PROBLEM
──────────────────────────────
App Image v1.0-dev   (config for dev)
App Image v1.0-staging (config for staging)  
App Image v1.0-prod (config for prod)

Result: 3 different images! ❌
```

**Kubernetes подход**:
```
KUBERNETES CONFIG SOLUTION
───────────────────────────────
App Image v1.0 (no config)
        +
ConfigMap/Secret (external config)
        ↓
Runtime Configuration ✅
```

### ConfigMaps: несекретная конфигурация

ConfigMap хранит **конфигурационные данные** в виде пар ключ-значение:

```
CONFIGMAP STRUCTURE
──────────────────────────────
ConfigMap: app-config
├── database.host = "db.example.com"
├── database.port = "5432"
├── log.level = "info"
└── feature.enabled = "true"
```

### Способы подключения ConfigMaps

#### 1. Environment Variables
```
CONFIGMAP AS ENV VARS
──────────────────────────────────
ConfigMap → Environment Variables
              ↓
Container sees:
- DATABASE_HOST=db.example.com
- DATABASE_PORT=5432
- LOG_LEVEL=info
```

#### 2. Volume Mounts
```
CONFIGMAP AS VOLUME
─────────────────────────────────
ConfigMap → Volume → Files
              ↓
Container sees files:
/config/database.host
/config/database.port
/config/log.level
```

#### 3. Command Line Arguments
```
CONFIGMAP AS ARGS
──────────────────────────────
ConfigMap values → Command args
                    ↓
Container starts with:
app --db-host=db.example.com --log-level=info
```

### Secrets: секретная информация

Secrets похожи на ConfigMaps, но предназначены для **чувствительных данных**:

```
SECRETS vs CONFIGMAPS
─────────────────────────────────
ConfigMap:           Secret:
- Database host      - Database password
- Log level          - API keys
- Feature flags      - TLS certificates
- App settings       - OAuth tokens
```

### Типы Secrets

#### 1. Opaque (generic)
Произвольные пользовательские данные:
```
SECRET: database-credentials
├── username = "admin"
└── password = "super-secret-123"
```

#### 2. kubernetes.io/dockerconfigjson
Credentials для Docker registry:
```
SECRET: docker-registry-creds
└── .dockerconfigjson = {"auths": {...}}
```

#### 3. kubernetes.io/tls
TLS сертификаты:
```
SECRET: tls-certificate
├── tls.crt = (certificate)
└── tls.key = (private key)
```

#### 4. kubernetes.io/service-account-token
Токены Service Account:
```
SECRET: sa-token
├── token = (JWT token)
├── ca.crt = (CA certificate)
└── namespace = "default"
```

### Base64 кодирование в Secrets

**Важно**: Secrets в Kubernetes **НЕ зашифрованы**, а только закодированы в Base64!

```
SECRET ENCODING (NOT ENCRYPTION!)
─────────────────────────────────────
Plain text: "password123"
Base64:     "cGFzc3dvcmQxMjM="

⚠️  Base64 ≠ Encryption
⚠️  Anyone with access can decode
```

### Best Practices для Secrets

#### 1. RBAC ограничения
```
RBAC FOR SECRETS
──────────────────────────────
User/Service Account
       ↓ (limited permissions)
Only specific Secrets
       ↓ (principle of least privilege)
Application Pods
```

#### 2. Encryption at Rest
```
ENCRYPTION AT REST
────────────────────────────
Secret (plain) → etcd (encrypted)
                    ↑
               EncryptionConfig
```

#### 3. External Secret Management
```
EXTERNAL SECRET STORE
─────────────────────────────────
HashiCorp Vault
AWS Secrets Manager  ←→ Kubernetes
Azure Key Vault           Secret
GCP Secret Manager
```

#### 4. Secret Rotation
```
SECRET ROTATION WORKFLOW
────────────────────────────
1. Generate new secret
2. Update Secret object
3. Rolling restart pods
4. Revoke old secret
```

### Управление жизненным циклом конфигурации

```
CONFIG LIFECYCLE
──────────────────────────────────
Development:
├── ConfigMap: app-config-dev
└── Secret: db-secret-dev

Staging:
├── ConfigMap: app-config-staging  
└── Secret: db-secret-staging

Production:
├── ConfigMap: app-config-prod
└── Secret: db-secret-prod
```

---

## 2.5 Namespaces: организация ресурсов (2-3 дня)

### Что такое Namespace

Namespace — это **виртуальное разделение** кластера, которое позволяет:
- Логически изолировать ресурсы
- Применять политики доступа (RBAC)
- Устанавливать ресурсные квоты
- Организовать multi-tenancy

```
CLUSTER WITH NAMESPACES
─────────────────────────────────────
┌─────────────────────────────────┐
│           Cluster               │
│                                 │
│  ┌──────────┐  ┌──────────┐    │
│  │   dev    │  │ staging  │    │
│  │ namespace│  │namespace │    │
│  │          │  │          │    │
│  │ Pod A    │  │ Pod A    │    │ ← Same name, different namespaces
│  │ Service B│  │ Service B│    │
│  └──────────┘  └──────────┘    │
│                                 │
│  ┌──────────┐  ┌──────────┐    │
│  │   prod   │  │ kube-sys │    │
│  │ namespace│  │namespace │    │
│  └──────────┘  └──────────┘    │
└─────────────────────────────────┘
```

### Системные Namespaces

Kubernetes создает несколько **системных namespaces**:

```
SYSTEM NAMESPACES
─────────────────────────────────
default
├── Default namespace для пользовательских ресурсов
└── Используется, если namespace не указан

kube-system  
├── Системные компоненты Kubernetes
├── kube-dns, kube-proxy, etcd
└── ⚠️ Не трогайте без необходимости!

kube-public
├── Ресурсы, доступные всем пользователям
└── Обычно используется для cluster info

kube-node-lease
├── Node heartbeat объекты
└── Для улучшения производительности
```

### DNS и Namespaces

**DNS resolution** в Kubernetes учитывает namespaces:

```
DNS RESOLUTION WITH NAMESPACES
──────────────────────────────────────
Service: my-app
Namespace: production

Full DNS name:
my-app.production.svc.cluster.local

From same namespace:
└── my-app

From different namespace:
└── my-app.production

Cross-namespace communication:
frontend (web namespace) 
    ↓
backend.api.svc.cluster.local
```

### Resource Quotas: ограничение ресурсов

ResourceQuota позволяет **ограничить** использование ресурсов в namespace:

```
RESOURCE QUOTA EXAMPLE
──────────────────────────────────
Namespace: development
ResourceQuota: dev-quota

Limits:
├── CPU requests: 10 cores
├── Memory requests: 20Gi
├── CPU limits: 20 cores
├── Memory limits: 40Gi
├── Pods: 50
├── Services: 10
├── Secrets: 20
└── ConfigMaps: 30
```

### LimitRange: ограничения по умолчанию

LimitRange устанавливает **ограничения по умолчанию** для ресурсов:

```
LIMITRANGE EXAMPLE
──────────────────────────────────
Namespace: development
LimitRange: dev-limits

Pod defaults:
├── CPU request: 100m
├── CPU limit: 500m  
├── Memory request: 128Mi
└── Memory limit: 512Mi

Container limits:
├── Min CPU: 50m
├── Max CPU: 2000m
├── Min Memory: 64Mi
└── Max Memory: 2Gi
```

### Network Policies между Namespaces

По умолчанию **весь трафик разрешен** между namespaces. NetworkPolicy может это изменить:

```
NETWORK ISOLATION
──────────────────────────────────────
Without NetworkPolicy:
frontend namespace ←→ backend namespace ✅
backend namespace  ←→ database namespace ✅

With NetworkPolicy:
frontend namespace ←→ backend namespace ✅
backend namespace  ←→ database namespace ✅
frontend namespace ←X→ database namespace ❌
```

### Организационные паттерны

#### 1. По средам (Environment-based)
```
ENVIRONMENT NAMESPACES
──────────────────────────
├── development
├── staging  
├── production
└── testing
```

#### 2. По командам (Team-based)
```
TEAM NAMESPACES
──────────────────────────
├── team-frontend
├── team-backend
├── team-data
└── team-platform
```

#### 3. По приложениям (Application-based)
```
APPLICATION NAMESPACES
──────────────────────────
├── ecommerce-web
├── ecommerce-api
├── ecommerce-payment
└── ecommerce-analytics
```

#### 4. Multi-tenant
```
MULTI-TENANT NAMESPACES
──────────────────────────
├── tenant-company-a
├── tenant-company-b
├── tenant-company-c
└── shared-services
```

---

## 🏆 Полученные знания после Блока 2

После изучения этого блока вы будете понимать:

### Концептуальные знания:
- **Архитектурные принципы** объектов Kubernetes
- **Жизненный цикл** Pod'ов и контейнеров
- **Паттерны** использования multi-container Pod'ов
- **Стратегии развертывания** и управления версиями
- **Сетевую модель** и service discovery
- **Управление конфигурацией** и секретами
- **Организацию ресурсов** через namespaces

### Практические навыки:
- Создание и управление Pod'ами
- Настройка Deployment'ов с rolling updates
- Масштабирование через HPA
- Конфигурирование Services всех типов
- Работа с ConfigMaps и Secrets
- Организация multi-tenant кластеров

### Понимание взаимосвязей:
```
KUBERNETES OBJECTS RELATIONSHIPS
─────────────────────────────────────

Namespace
    ↓ contains
Deployment
    ↓ manages
ReplicaSet  
    ↓ creates
Pods ←──────────── Service (selects)
 ↓                    ↑
ConfigMap/Secret ─────┘
 ↓ (mounted as)
Volumes
```

### Готовность к следующему блоку:
- Понимание основ для изучения сетей и storage
- Базовые знания для настройки Ingress
- Фундамент для изучения StatefulSets
- Основа для углубления в безопасность

---

## 📚 Практические задания для закрепления

### Задание 1: Многоуровневое приложение
Разверните приложение состоящее из:
- Frontend (React app)
- Backend API (Node.js/Python)
- Database (PostgreSQL)
- Redis для кеширования

### Задание 2: Автоматическое масштабирование
Настройте HPA для одного из компонентов с нагрузочным тестированием.

### Задание 3: Конфигурация через ConfigMaps
Организуйте конфигурацию для разных сред (dev/staging/prod) используя ConfigMaps и Secrets.

---

**Следующий блок**: Сетевое взаимодействие и хранение данных 🌐