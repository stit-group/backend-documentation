# System Design: Блок 4 - Микросервисы и распределенные системы

**Продолжительность:** 4-5 недель  
**Цель:** Освоить современные подходы к архитектуре сложных систем  
**Временные затраты:** 8-12 часов в неделю

---

## 📋 Обзор блока

```
┌─────────────────────────────────────────────────────────────┐
│                    БЛОК 4: СОДЕРЖАНИЕ                      │
├─────────────────────────────────────────────────────────────┤
│ Неделя 15-16: │ Микросервисная архитектура                │
│ Неделя 17:    │ API Management                            │
│ Неделя 18:    │ Event-driven архитектура                  │
│ Неделя 19:    │ Containerization и оркестрация            │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎯 Цели обучения

По завершении этого блока вы сможете:
- ✅ Проектировать микросервисную архитектуру
- ✅ Выбирать между монолитом и микросервисами
- ✅ Настраивать API Gateway и service discovery
- ✅ Проектировать event-driven системы
- ✅ Работать с контейнерами и оркестрацией

---

# 📅 НЕДЕЛЯ 15-16: Микросервисная архитектура

## 🎯 Глава 4.1: Монолит vs Микросервисы

### 📚 Теория (3 часа)

#### Эволюция архитектур

```
ЭВОЛЮЦИЯ СИСТЕМ
================

Монолит (2000-2010)
┌─────────────────────────────────┐
│          ПРИЛОЖЕНИЕ             │
│  ┌─────┐ ┌─────┐ ┌─────────┐   │
│  │ UI  │ │Logic│ │Database │   │
│  └─────┘ └─────┘ └─────────┘   │
└─────────────────────────────────┘

SOA (2005-2015)
┌─────────┐    ┌─────────┐    ┌─────────┐
│Service A│<-->│Service B│<-->│Service C│
└─────────┘    └─────────┘    └─────────┘
     │              │              │
     └──────────────┼──────────────┘
                    │
              ┌─────────┐
              │   ESB   │
              └─────────┘

Микросервисы (2010+)
     API Gateway
          │
    ┌─────┴─────┐
    │           │
┌───▼───┐   ┌───▼───┐   ┌───────┐
│Service│   │Service│   │Service│
│   A   │   │   B   │   │   C   │
└───┬───┘   └───┬───┘   └───┬───┘
    │           │           │
┌───▼───┐   ┌───▼───┐   ┌───▼───┐
│ DB-A  │   │ DB-B  │   │ DB-C  │
└───────┘   └───────┘   └───────┘
```

#### Conway's Law и организационные аспекты

> **Conway's Law**: "Организации проектируют системы, которые копируют структуру коммуникации в этой организации"

```
ВЛИЯНИЕ СТРУКТУРЫ КОМАНДЫ НА АРХИТЕКТУРУ
=======================================

Монолитная команда → Монолитная архитектура
┌─────────────────────────────────────────┐
│              КОМАНДА                    │
│  Frontend + Backend + Database + DevOps │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│            МОНОЛИТ                      │
└─────────────────────────────────────────┘

Распределенные команды → Микросервисы
┌──────────┐ ┌──────────┐ ┌──────────┐
│ Команда  │ │ Команда  │ │ Команда  │
│    A     │ │    B     │ │    C     │
└────┬─────┘ └────┬─────┘ └────┬─────┘
     │            │            │
     ▼            ▼            ▼
┌────────┐   ┌────────┐   ┌────────┐
│Сервис A│   │Сервис B│   │Сервис C│
└────────┘   └────────┘   └────────┘
```

#### Когда выбирать микросервисы

```
МАТРИЦА ПРИНЯТИЯ РЕШЕНИЙ
========================

              │ Простая │ Сложная │
              │ система │ система │
──────────────┼─────────┼─────────┤
Маленькая     │ МОНОЛИТ │ МОНОЛИТ │
команда (<10) │   ✅    │   ✅    │
──────────────┼─────────┼─────────┤
Большая       │ МОНОЛИТ │МИКРОСЕР-│
команда (>10) │   ✅    │  ВИСЫ ✅│
──────────────┴─────────┴─────────┘

КРИТЕРИИ ДЛЯ МИКРОСЕРВИСОВ:
---------------------------
✅ Команда > 10 человек
✅ Разные команды работают над разными частями
✅ Разные технологические требования
✅ Независимые циклы релизов
✅ Высокие требования к масштабированию
✅ Четкие границы доменов
```

#### Преимущества и недостатки

```
СРАВНЕНИЕ ПОДХОДОВ
==================

МОНОЛИТ                    │  МИКРОСЕРВИСЫ
──────────────────────────┼──────────────────────────
ПЛЮСЫ:                    │  ПЛЮСЫ:
+ Простота разработки     │  + Технологическое разнообразие
+ Простота тестирования   │  + Независимое масштабирование
+ Простота деплоя         │  + Изоляция отказов
+ Единая кодовая база     │  + Независимые релизы
+ ACID транзакции         │  + Команды владеют сервисами

МИНУСЫ:                   │  МИНУСЫ:
- Scaling bottlenecks     │  - Сложность интеграции
- Technology lock-in      │  - Distributed системы сложность
- Длинные циклы релизов   │  - Eventual consistency
- Single point of failure │  - Network latency
                          │  - Operational overhead
```

### 🛠️ Практика (3 часа)

#### Задание 1: Декомпозиция монолитного приложения

Рассмотрим монолитное приложение интернет-магазина:

```
ИСХОДНЫЙ МОНОЛИТ E-COMMERCE
===========================
┌─────────────────────────────────────────────────────────┐
│                    E-COMMERCE MONOLITH                  │
├─────────────────────────────────────────────────────────┤
│ User Management     │ Product Catalog    │ Orders       │
│ - Registration      │ - Product CRUD     │ - Order CRUD │
│ - Authentication    │ - Categories       │ - Order Status│
│ - User Profiles     │ - Search           │ - Payments   │
│ - Permissions       │ - Recommendations  │ - Shipping   │
├─────────────────────────────────────────────────────────┤
│ Inventory          │ Reviews & Ratings   │ Analytics    │
│ - Stock tracking   │ - User reviews      │ - User behavior│
│ - Reservations     │ - Rating system     │ - Sales reports│
│ - Suppliers        │ - Moderation        │ - Recommendations│
├─────────────────────────────────────────────────────────┤
│                    SHARED DATABASE                      │
└─────────────────────────────────────────────────────────┘
```

**Шаги декомпозиции:**

1. **Идентификация доменных границ:**

```
DOMAIN BOUNDARIES
=================

┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   USERS     │  │  CATALOG    │  │   ORDERS    │
│             │  │             │  │             │
│ Users       │  │ Products    │  │ Orders      │
│ Auth        │  │ Categories  │  │ Payments    │
│ Profiles    │  │ Search      │  │ Shipping    │
└─────────────┘  └─────────────┘  └─────────────┘

┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│ INVENTORY   │  │  REVIEWS    │  │ ANALYTICS   │
│             │  │             │  │             │
│ Stock       │  │ Reviews     │  │ Metrics     │
│ Reservations│  │ Ratings     │  │ Reports     │
│ Suppliers   │  │ Moderation  │  │ ML Models   │
└─────────────┘  └─────────────┘  └─────────────┘
```

2. **Результат микросервисной архитектуры:**

```
МИКРОСЕРВИСНАЯ АРХИТЕКТУРА
==========================

                     API Gateway
                         │
    ┌────────────────────┼────────────────────┐
    │                    │                    │
    ▼                    ▼                    ▼
┌──────────┐      ┌──────────┐      ┌──────────┐
│  User    │      │ Product  │      │  Order   │
│ Service  │      │ Service  │      │ Service  │
├──────────┤      ├──────────┤      ├──────────┤
│User DB   │      │Product DB│      │Order DB  │
└──────────┘      └──────────┘      └──────────┘

┌──────────┐      ┌──────────┐      ┌──────────┐
│Inventory │      │ Review   │      │Analytics │
│ Service  │      │ Service  │      │ Service  │
├──────────┤      ├──────────┤      ├──────────┤
│Inventory │      │Review DB │      │Analytics │
│   DB     │      │          │      │   DB     │
└──────────┘      └──────────┘      └──────────┘

          Event Bus (Kafka/RabbitMQ)
          ═══════════════════════════
```

#### Задание 2: Определение границ сервисов

**Критерии выделения сервисов:**

```
КРИТЕРИИ ДЕКОМПОЗИЦИИ
=====================

1. BUSINESS CAPABILITY
   ┌─────────────────┐
   │ Что делает?     │
   │ - User mgmt     │
   │ - Product mgmt  │
   │ - Order process │
   └─────────────────┘

2. DATA OWNERSHIP
   ┌─────────────────┐
   │ Кто владеет?    │
   │ - User data     │
   │ - Product data  │
   │ - Order data    │
   └─────────────────┘

3. TEAM OWNERSHIP
   ┌─────────────────┐
   │ Кто развивает?  │
   │ - User team     │
   │ - Catalog team  │
   │ - Order team    │
   └─────────────────┘

4. SCALING NEEDS
   ┌─────────────────┐
   │ Как масштабируется?│
   │ - High read     │
   │ - High write    │
   │ - CPU intensive │
   └─────────────────┘
```

---

## 🎯 Глава 4.2: Service Discovery и коммуникация

### 📚 Теория (2 часа)

#### Service Discovery паттерны

```
SERVICE DISCOVERY PATTERNS
===========================

1. CLIENT-SIDE DISCOVERY
========================
┌────────┐    1. Query    ┌─────────────┐
│ Client │──────────────→│Service      │
│        │               │Registry     │
│        │←──────────────│             │
└────┬───┘    2. Response└─────────────┘
     │ 3. Direct call
     ▼
┌────────┐
│Service │
│Instance│
└────────┘

Примеры: Netflix Eureka, Consul

2. SERVER-SIDE DISCOVERY
========================
┌────────┐               ┌─────────────┐
│ Client │──────────────→│Load         │
│        │  Request      │Balancer     │
└────────┘               │             │
                         │ ┌─────────┐ │
                         │ │Registry │ │
                         │ └─────────┘ │
                         └──────┬──────┘
                                │ Forward
                                ▼
                         ┌────────────┐
                         │Service     │
                         │Instance    │
                         └────────────┘

Примеры: AWS ELB, Kubernetes Service

3. SERVICE MESH
===============
┌────────┐               ┌────────────┐
│ Client │←─────────────→│   Proxy    │
│        │  All traffic  │  (Sidecar) │
└────────┘               └──────┬─────┘
                                │
                         ┌──────▼─────┐
                         │   Proxy    │
                         │  (Sidecar) │
                         └──────┬─────┘
                                │
                         ┌──────▼─────┐
                         │  Service   │
                         │  Instance  │
                         └────────────┘

Примеры: Istio, Linkerd, Consul Connect
```

#### Синхронная vs асинхронная коммуникация

```
ТИПЫ КОММУНИКАЦИИ
=================

СИНХРОННАЯ (Request-Response)
============================
Client           Service A            Service B
  │                 │                   │
  ├─ HTTP Request ─→│                   │
  │                 ├─ HTTP Request ────→│
  │                 │← Response ────────┤
  │← Response ──────┤                   │
  │                 │                   │

+ Простота реализации
+ Немедленный ответ
+ Легко debug
- Cascading failures
- Higher latency
- Tight coupling

АСИНХРОННАЯ (Event-driven)
==========================
Service A        Message Broker        Service B
  │                     │                   │
  ├─ Publish Event ────→│                   │
  │                     ├─ Deliver ────────→│
  │                     │                   │
  │                     │← Ack ─────────────┤
  │                     │                   │

+ Loose coupling
+ Better resilience
+ Higher throughput
- Eventual consistency
- Complex debugging
- Message ordering issues
```

### 🛠️ Практика (3 часа)

#### Проектирование Service Mesh архитектуры

```
SERVICE MESH ARCHITECTURE
==========================

                Control Plane
        ┌─────────────────────────────┐
        │     Service Discovery       │
        │     Policy Management       │
        │     Certificate Authority   │
        │     Telemetry Collection    │
        └─────────────┬───────────────┘
                      │ Config & Control
         ┌────────────┼────────────┐
         │            │            │
         ▼            ▼            ▼
    ┌─────────┐  ┌─────────┐  ┌─────────┐
    │ Proxy A │  │ Proxy B │  │ Proxy C │
    │(Envoy)  │  │(Envoy)  │  │(Envoy)  │
    └────┬────┘  └────┬────┘  └────┬────┘
         │            │            │
    ┌────▼────┐  ┌────▼────┐  ┌────▼────┐
    │Service A│  │Service B│  │Service C│
    └─────────┘  └─────────┘  └─────────┘

ПРЕИМУЩЕСТВА SERVICE MESH:
- Автоматическое mTLS
- Traffic management
- Observability из коробки
- Circuit breaking
- Rate limiting
- Canary deployments
```

---

# 📅 НЕДЕЛЯ 17: API Management

## 🎯 Глава 4.3: API Gateway

### 📚 Теория (2 часа)

#### API Gateway responsibilities

```
API GATEWAY ФУНКЦИИ
===================

┌─────────────────────────────────────────────────┐
│                 API GATEWAY                     │
├─────────────────────────────────────────────────┤
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │
│ │   Routing   │ │    Auth     │ │Rate Limiting│ │
│ │             │ │             │ │             │ │
│ └─────────────┘ └─────────────┘ └─────────────┘ │
├─────────────────────────────────────────────────┤
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │
│ │Load Balance │ │  Caching    │ │ Monitoring  │ │
│ │             │ │             │ │             │ │
│ └─────────────┘ └─────────────┘ └─────────────┘ │
├─────────────────────────────────────────────────┤
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │
│ │Transformation│ │Circuit      │ │   Logging   │ │
│ │             │ │Breaker      │ │             │ │
│ └─────────────┘ └─────────────┘ └─────────────┘ │
└─────────────────────────────────────────────────┘
                          │
         ┌────────────────┼────────────────┐
         │                │                │
         ▼                ▼                ▼
    ┌─────────┐      ┌─────────┐      ┌─────────┐
    │Service A│      │Service B│      │Service C│
    └─────────┘      └─────────┘      └─────────┘
```

#### Rate Limiting алгоритмы

```
RATE LIMITING ALGORITHMS
========================

1. TOKEN BUCKET
===============
┌─────────────────┐    Add tokens
│   Token Bucket  │◄─── at fixed rate
│   [●●●●○○○○○○]  │
└─────────┬───────┘
          │ Request consumes token
          ▼
    ┌──────────┐
    │ Request  │
    └──────────┘

Pros: Allows bursts
Cons: More complex

2. LEAKY BUCKET
===============
         Requests
            │
    ┌───────▼───────┐
    │     Queue     │
    │  [R][R][R]    │
    └───────┬───────┘
            │ Process at
            │ constant rate
            ▼
      ┌──────────┐
      │Processing│
      └──────────┘

Pros: Smooth output
Cons: Can reject bursts

3. FIXED WINDOW
===============
Window 1    Window 2    Window 3
[●●●●●]     [●●○○○]     [○○○○○]
 0-60s      60-120s     120-180s

Pros: Simple to implement
Cons: Burst at window edges

4. SLIDING WINDOW
=================
Current time: 90s
Window: [30s ← 90s]
Requests in window: ●●●●○

Pros: Precise control
Cons: Memory intensive
```

### 🛠️ Практика (3 часа)

#### Проектирование API Gateway для микросервисов

```
API GATEWAY DESIGN
==================

External Clients
┌─────────┐ ┌─────────┐ ┌─────────┐
│Mobile   │ │Web App  │ │Partners │
│   App   │ │         │ │   API   │
└────┬────┘ └────┬────┘ └────┬────┘
     │           │           │
     └───────────┼───────────┘
                 │
            ┌────▼────┐
            │   CDN   │
            │ (Static)│
            └────┬────┘
                 │
         ┌───────▼───────┐
         │  API Gateway  │
         │               │
         │ ┌───────────┐ │
         │ │  Routing  │ │
         │ │   Rules   │ │
         │ └───────────┘ │
         └───┬───┬───┬───┘
             │   │   │
    ┌────────▼───▼───▼────────┐
    │     Service Mesh        │
    └┬─────────┬─────────────┬┘
     │         │             │
┌────▼────┐┌───▼────┐┌──────▼──┐
│ User    ││Product ││ Order   │
│Service  ││Service ││ Service │
└─────────┘└────────┘└─────────┘

ROUTING CONFIGURATION:
======================
/api/v1/users/*     → User Service
/api/v1/products/*  → Product Service  
/api/v1/orders/*    → Order Service
/api/v1/inventory/* → Inventory Service
```

#### Rate Limiting и Security настройка

```
RATE LIMITING CONFIGURATION
============================

Global Limits:
┌─────────────────────────────┐
│ All APIs: 1000 req/min      │
└─────────────────────────────┘

Per-Service Limits:
┌─────────────────────────────┐
│ /users/*:    100 req/min    │
│ /products/*: 500 req/min    │
│ /orders/*:   200 req/min    │
└─────────────────────────────┘

Per-User Limits:
┌─────────────────────────────┐
│ Free tier:   10 req/min     │
│ Premium:     100 req/min    │
│ Enterprise:  1000 req/min   │
└─────────────────────────────┘

SECURITY LAYERS:
================
1. DDoS Protection
   ├── Rate Limiting
   ├── IP Blacklisting  
   └── Geographic Filtering

2. Authentication
   ├── JWT Validation
   ├── OAuth 2.0
   └── API Keys

3. Authorization
   ├── RBAC (Role-Based)
   ├── ABAC (Attribute-Based)
   └── Resource-Level Permissions
```

---

# 📅 НЕДЕЛЯ 18: Event-driven архитектура

## 🎯 Глава 4.4: Event-driven архитектура и message queues

### 📚 Теория (3 часа)

#### Event Sourcing и CQRS

```
EVENT SOURCING PATTERN
======================

Traditional State Storage:
┌─────────────────────────┐
│     Current State       │
│  ┌─────────────────┐   │
│  │ User: John      │   │
│  │ Email: j@e.com  │   │
│  │ Status: Active  │   │
│  └─────────────────┘   │
└─────────────────────────┘
❌ История изменений потеряна

Event Sourcing:
┌─────────────────────────┐
│       Event Store       │
│  ┌─────────────────┐   │
│  │ UserCreated     │   │
│  │ EmailChanged    │   │
│  │ UserActivated   │   │
│  └─────────────────┘   │
└─────────────────────────┘
           │
           ▼ Replay events
┌─────────────────────────┐
│     Current State       │
│  ┌─────────────────┐   │
│  │ User: John      │   │
│  │ Email: j@e.com  │   │
│  │ Status: Active  │   │
│  └─────────────────┘   │
└─────────────────────────┘
✅ Полная история + аудит

CQRS (Command Query Responsibility Segregation):
===============================================

Write Side (Commands)     Read Side (Queries)
┌─────────────────┐      ┌─────────────────┐
│   Commands      │      │    Queries      │
│                 │      │                 │
│ CreateUser      │      │ GetUser         │
│ UpdateEmail     │      │ GetUserList     │
│ DeactivateUser  │      │ SearchUsers     │
└─────┬───────────┘      └─────┬───────────┘
      │                        │
      ▼                        ▼
┌─────────────────┐      ┌─────────────────┐
│  Write Model    │      │   Read Model    │
│  (Normalized)   │      │  (Denormalized) │
│                 │      │                 │
│ Event Store     │─────→│ Projection DB   │
└─────────────────┘      └─────────────────┘
```

#### Message Brokers сравнение

```
MESSAGE BROKERS COMPARISON
==========================

APACHE KAFKA
============
┌─────────────────────────────────────────────┐
│                 Kafka Cluster               │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐       │
│  │Broker 1 │ │Broker 2 │ │Broker 3 │       │
│  └─────────┘ └─────────┘ └─────────┘       │
│                                             │
│  Topic: orders                              │
│  ┌───────────┬───────────┬───────────┐     │
│  │Partition 0│Partition 1│Partition 2│     │
│  └───────────┴───────────┴───────────┘     │
└─────────────────────────────────────────────┘

✅ High throughput (millions/sec)
✅ Durable storage
✅ Scalable partitioning
✅ Stream processing
❌ Complex setup
❌ At-least-once delivery

RABBITMQ
========
┌─────────────────────────────────────────────┐
│                RabbitMQ                     │
│                                             │
│  Producer → Exchange → Queue → Consumer     │
│                                             │
│  ┌─────────┐   ┌─────┐   ┌──────────┐      │
│  │Publisher│──→│     │──→│Subscriber│      │
│  └─────────┘   └─────┘   └──────────┘      │
└─────────────────────────────────────────────┘

✅ Exactly-once delivery
✅ Message routing flexibility  
✅ Easy to setup
✅ Management UI
❌ Lower throughput
❌ Limited horizontal scaling

AWS SQS/SNS
===========
     ┌─────────┐
     │   SNS   │ (Topics)
     │ (Pub/Sub)│
     └────┬────┘
          │ Fan-out
    ┌─────┼─────┐
    ▼     ▼     ▼
┌───────┐┌─────┐┌───────┐
│SQS Q1 ││Email││SQS Q2 │ 
└───────┘└─────┘└───────┘

✅ Fully managed
✅ Auto-scaling
✅ Dead letter queues
✅ Pay per use
❌ AWS vendor lock-in
❌ Limited message routing
```

#### Pub/Sub vs Point-to-Point

```
MESSAGING PATTERNS
==================

POINT-TO-POINT (Queue)
=====================
Producer → [Queue] → Consumer
                     (Only one consumer gets message)

Example: Order Processing
┌─────────┐    ┌──────────┐    ┌──────────┐
│Order    │───→│Order     │───→│Payment   │
│Service  │    │Queue     │    │Service   │
└─────────┘    └──────────┘    └──────────┘

PUB/SUB (Topic)
===============
                 ┌→ Consumer A
Publisher → Topic┼→ Consumer B  
                 └→ Consumer C
                 (All consumers get message)

Example: User Registration Event
┌─────────┐    ┌──────────────┐    ┌────────────┐
│User     │───→│Registration  │───→│Email       │
│Service  │    │Topic         │    │Service     │
└─────────┘    └──────┬───────┘    └────────────┘
                      │             ┌────────────┐
                      └────────────→│Analytics   │
                                   │Service     │
                                   └────────────┘
```

### 🛠️ Практика (3 часа)

#### Проектирование Event-driven системы для e-commerce

```
E-COMMERCE EVENT-DRIVEN ARCHITECTURE
====================================

                     Event Bus (Kafka)
          ┌─────────────────────────────────────┐
          │                                     │
    ┌─────▼─────┐  ┌─────────────┐  ┌──────▼───┐
    │   User    │  │   Order     │  │ Payment  │
    │  Service  │  │  Service    │  │ Service  │
    └─────┬─────┘  └─────┬───────┘  └──────┬───┘
          │              │                 │
          │              │                 │
    ┌─────▼─────┐  ┌─────▼───────┐  ┌──────▼───┐
    │Notification│  │ Inventory   │  │ Shipping │
    │  Service   │  │  Service    │  │ Service  │
    └───────────┘  └─────────────┘  └──────────┘

EVENT FLOW EXAMPLE: Order Processing
===================================

1. Order Created Event
   ┌─────────────────────────────────┐
   │ Event: OrderCreated             │
   │ Data: {                         │
   │   orderId: "12345",            │
   │   customerId: "user123",       │
   │   items: [...],                │
   │   totalAmount: 99.99           │
   │ }                              │
   └─────────────────────────────────┘

2. Multiple Services React:
   ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
   │ Inventory    │ │ Payment      │ │ Notification │
   │ Service      │ │ Service      │ │ Service      │
   │              │ │              │ │              │
   │ Reserve      │ │ Process      │ │ Send email   │
   │ Items        │ │ Payment      │ │ to customer  │
   └──────────────┘ └──────────────┘ └──────────────┘

3. Subsequent Events:
   - PaymentProcessed
   - InventoryReserved  
   - OrderConfirmed
   - ShippingScheduled
```

---

## 🎯 Глава 4.5: Distributed Transactions и Saga Pattern

### 📚 Теория (3 часа)

#### Проблемы распределенных транзакций

```
DISTRIBUTED TRANSACTION CHALLENGES
==================================

ACID в монолите:
┌─────────────────────────────────────────┐
│            SINGLE DATABASE              │
│  ┌─────────────────────────────────┐   │
│  │         TRANSACTION             │   │
│  │  1. Debit Account A             │   │
│  │  2. Credit Account B            │   │
│  │  3. Log Transaction             │   │
│  │                                 │   │
│  │  Atomicity: All or Nothing     │   │
│  │  Consistency: Valid state      │   │
│  │  Isolation: No interference    │   │
│  │  Durability: Persistent        │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘

Проблема в микросервисах:
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│  Account     │ │   Payment    │ │   Logging    │
│  Service     │ │   Service    │ │   Service    │
│              │ │              │ │              │
│ [Local TX]   │ │ [Local TX]   │ │ [Local TX]   │
└──────┬───────┘ └──────┬───────┘ └──────┬───────┘
       │                │                │
       └────────────────┼────────────────┘
                        │
              ❌ No global ACID
              ❌ Partial failures
              ❌ Inconsistent state
```

#### Two-Phase Commit проблемы

```
TWO-PHASE COMMIT (2PC)
======================

Phase 1: Prepare
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│Coordinator  │────→│ Service A   │     │ Service B   │
│             │     │             │     │             │
│"Prepare"    │     │ "Vote YES"  │     │ "Vote YES"  │
│             │←────│             │←────│             │
└─────────────┘     └─────────────┘     └─────────────┘

Phase 2: Commit
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│Coordinator  │────→│ Service A   │     │ Service B   │
│             │     │             │     │             │
│"Commit"     │     │ "ACK"       │     │ "ACK"       │
│             │←────│             │←────│             │
└─────────────┘     └─────────────┘     └─────────────┘

ПРОБЛЕМЫ 2PC:
=============
❌ Coordinator single point of failure
❌ Blocking protocol (locks resources)
❌ Network partitions cause issues
❌ Not suitable for high-scale systems
❌ Performance overhead
```

#### Saga Pattern

```
SAGA PATTERN
============

CHOREOGRAPHY (Event-driven)
===========================

Order → Payment → Inventory → Shipping
  │        │         │          │
  ▼        ▼         ▼          ▼
Success  Failure   Success   Success

If Payment fails:
Payment Service publishes PaymentFailed event
Order Service listens and cancels order

┌─────────────────────────────────────────────────┐
│              Event Timeline                     │
├─────────────────────────────────────────────────┤
│ 1. OrderCreated        → All services listen   │
│ 2. PaymentFailed       → Trigger compensations │
│ 3. OrderCancelled      → Clean up resources    │
└─────────────────────────────────────────────────┘

ORCHESTRATION (Command-driven)
==============================

       ┌─────────────────┐
       │     Saga        │
       │  Orchestrator   │
       └─────┬───────────┘
             │
    ┌────────┼────────┐
    │        │        │
    ▼        ▼        ▼
┌────────┐┌────────┐┌────────┐
│Order   ││Payment ││Shipping│
│Service ││Service ││Service │
└────────┘└────────┘└────────┘

Orchestrator controls the flow:
1. Create Order
2. Process Payment
3. If payment fails → Cancel Order
4. If payment succeeds → Schedule Shipping
```

### 🛠️ Практика (2 часа)

#### Проектирование Saga для процесса заказа

```
E-COMMERCE ORDER SAGA
=====================

HAPPY PATH:
===========
┌─────────────────────────────────────────────────────────┐
│                    Order Saga Flow                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ 1. CreateOrder     → Order Service    → OrderCreated    │
│ 2. ReserveItems    → Inventory Service → ItemsReserved  │
│ 3. ProcessPayment  → Payment Service  → PaymentProcessed│
│ 4. ScheduleShipping→ Shipping Service → ShippingScheduled│
│ 5. ConfirmOrder    → Order Service    → OrderConfirmed  │
│                                                         │
└─────────────────────────────────────────────────────────┘

FAILURE SCENARIOS:
==================

Scenario 1: Payment Fails
┌─────────────────────────────────────────────────────────┐
│ 1. CreateOrder     → Order Service    → OrderCreated    │
│ 2. ReserveItems    → Inventory Service → ItemsReserved  │
│ 3. ProcessPayment  → Payment Service  → PaymentFailed   │
│                                                         │
│ COMPENSATION ACTIONS:                                   │
│ 4. ReleaseItems    → Inventory Service → ItemsReleased  │
│ 5. CancelOrder     → Order Service    → OrderCancelled  │
└─────────────────────────────────────────────────────────┘

Scenario 2: Inventory Unavailable
┌─────────────────────────────────────────────────────────┐
│ 1. CreateOrder     → Order Service    → OrderCreated    │
│ 2. ReserveItems    → Inventory Service → ReservationFailed│
│                                                         │
│ COMPENSATION ACTIONS:                                   │
│ 3. CancelOrder     → Order Service    → OrderCancelled  │
└─────────────────────────────────────────────────────────┘

IMPLEMENTATION:
===============
{
  "sagaId": "saga-12345",
  "sagaType": "OrderProcessing",
  "steps": [
    {
      "stepId": 1,
      "action": "CreateOrder",
      "service": "order-service",
      "compensation": "CancelOrder",
      "status": "completed"
    },
    {
      "stepId": 2,
      "action": "ReserveItems", 
      "service": "inventory-service",
      "compensation": "ReleaseItems",
      "status": "completed"
    },
    {
      "stepId": 3,
      "action": "ProcessPayment",
      "service": "payment-service", 
      "compensation": "RefundPayment",
      "status": "failed"
    }
  ],
  "compensationRequired": true,
  "currentStep": 2
}
```

---

# 📅 НЕДЕЛЯ 19: Containerization и оркестрация

## 🎯 Глава 4.6: Docker и Kubernetes

### 📚 Теория (3 часа)

#### Контейнеризация vs Виртуализация

```
VIRTUALIZATION vs CONTAINERIZATION
===================================

TRADITIONAL VIRTUALIZATION:
============================
┌─────────────────────────────────────────────────┐
│                Host OS                          │
├─────────────────────────────────────────────────┤
│              Hypervisor                         │
├─────────────────┬───────────────┬───────────────┤
│   Guest OS      │   Guest OS    │   Guest OS    │
│ ┌─────────────┐ │ ┌───────────┐ │ ┌───────────┐ │
│ │    App A    │ │ │   App B   │ │ │   App C   │ │
│ └─────────────┘ │ └───────────┘ │ └───────────┘ │
└─────────────────┴───────────────┴───────────────┘

Resource overhead: ~15-20% per VM
Boot time: 1-2 minutes
Isolation: Strong

CONTAINERIZATION:
=================
┌─────────────────────────────────────────────────┐
│                Host OS                          │
├─────────────────────────────────────────────────┤
│           Container Runtime                     │
├─────────────────┬───────────────┬───────────────┤
│   Container A   │  Container B  │  Container C  │
│ ┌─────────────┐ │ ┌───────────┐ │ ┌───────────┐ │
│ │    App A    │ │ │   App B   │ │ │   App C   │ │
│ │    Libs     │ │ │   Libs    │ │ │   Libs    │ │
│ └─────────────┘ │ └───────────┘ │ └───────────┘ │
└─────────────────┴───────────────┴───────────────┘

Resource overhead: ~2-5%
Boot time: Seconds
Isolation: Process-level
```

#### Docker Ecosystem

```
DOCKER ECOSYSTEM
================

DOCKER ARCHITECTURE:
====================
┌─────────────────────────────────────────────────┐
│               Docker Client                     │
│              (docker CLI)                       │
└─────────────────┬───────────────────────────────┘
                  │ Docker API
┌─────────────────▼───────────────────────────────┐
│              Docker Daemon                      │
│                                                 │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │
│ │ Container 1 │ │ Container 2 │ │ Container 3 │ │
│ └─────────────┘ └─────────────┘ └─────────────┘ │
│                                                 │
│ ┌─────────────┐ ┌─────────────┐                │
│ │   Images    │ │   Volumes   │                │
│ └─────────────┘ └─────────────┘                │
└─────────────────────────────────────────────────┘

DOCKERFILE EXAMPLE:
===================
FROM node:16-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
EXPOSE 3000
CMD ["npm", "start"]

BUILD → RUN → SHIP:
===================
Developer → Docker Build → Docker Registry → Production
   │              │              │              │
   ▼              ▼              ▼              ▼
Dockerfile    Docker Image    Push/Pull     Container
```

#### Kubernetes архитектура

```
KUBERNETES ARCHITECTURE
=======================

CONTROL PLANE:
==============
┌─────────────────────────────────────────────────┐
│                Master Node                      │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │
│ │API Server   │ │  Scheduler  │ │   etcd      │ │
│ └─────────────┘ └─────────────┘ └─────────────┘ │
│ ┌─────────────┐ ┌─────────────┐                │
│ │Controller   │ │Cloud Control│                │
│ │Manager      │ │Manager      │                │
│ └─────────────┘ └─────────────┘                │
└─────────────────────────────────────────────────┘

WORKER NODES:
=============
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│   Worker Node   │ │   Worker Node   │ │   Worker Node   │
│ ┌─────────────┐ │ │ ┌─────────────┐ │ │ ┌─────────────┐ │
│ │   kubelet   │ │ │ │   kubelet   │ │ │ │   kubelet   │ │
│ └─────────────┘ │ │ └─────────────┘ │ │ └─────────────┘ │
│ ┌─────────────┐ │ │ ┌─────────────┐ │ │ ┌─────────────┐ │
│ │kube-proxy   │ │ │ │kube-proxy   │ │ │ │kube-proxy   │ │
│ └─────────────┘ │ │ └─────────────┘ │ │ └─────────────┘ │
│ ┌─────────────┐ │ │ ┌─────────────┐ │ │ ┌─────────────┐ │
│ │ Container   │ │ │ │ Container   │ │ │ │ Container   │ │
│ │  Runtime    │ │ │ │  Runtime    │ │ │ │  Runtime    │ │
│ └─────────────┘ │ │ └─────────────┘ │ │ └─────────────┘ │
└─────────────────┘ └─────────────────┘ └─────────────────┘

KUBERNETES OBJECTS:
===================
Pod (Smallest unit)
┌─────────────────────────────────┐
│              Pod                │
│ ┌─────────────┐ ┌─────────────┐ │
│ │Container A  │ │Container B  │ │
│ │(main app)   │ │(sidecar)    │ │
│ └─────────────┘ └─────────────┘ │
│        Shared Storage           │
│        Shared Network           │
└─────────────────────────────────┘

Service (Load balancer)
┌─────────────────────────────────┐
│            Service              │
│         (Load Balancer)         │
└─────────────┬───────────────────┘
              │
    ┌─────────┼─────────┐
    │         │         │
    ▼         ▼         ▼
┌─────────┐┌─────────┐┌─────────┐
│  Pod 1  ││  Pod 2  ││  Pod 3  │
└─────────┘└─────────┘└─────────┘

Deployment (Pod management)
┌─────────────────────────────────┐
│           Deployment            │
│  Desired State: 3 replicas     │
│  Update Strategy: Rolling       │
└─────────────┬───────────────────┘
              │ Manages
              ▼
┌─────────────────────────────────┐
│           ReplicaSet            │
│    Current: 3, Desired: 3      │
└─────────────┬───────────────────┘
              │ Creates
              ▼
    ┌─────────┼─────────┐
    │         │         │
    ▼         ▼         ▼
┌─────────┐┌─────────┐┌─────────┐
│  Pod 1  ││  Pod 2  ││  Pod 3  │
└─────────┘└─────────┘└─────────┘
```

### 🛠️ Практика (3 часа)

#### Проектирование Kubernetes deployment

```
MICROSERVICES KUBERNETES DEPLOYMENT
===================================

NAMESPACE ORGANIZATION:
=======================
┌─────────────────────────────────────────────────┐
│                 Cluster                         │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │
│ │    dev      │ │   staging   │ │    prod     │ │
│ │ namespace   │ │ namespace   │ │ namespace   │ │
│ └─────────────┘ └─────────────┘ └─────────────┘ │
└─────────────────────────────────────────────────┘

DEPLOYMENT MANIFEST EXAMPLE:
============================
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
      - name: user-service
        image: myregistry/user-service:v1.2.3
        ports:
        - containerPort: 8080
        env:
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: host
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
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

SERVICE MANIFEST:
=================
apiVersion: v1
kind: Service
metadata:
  name: user-service
  namespace: production
spec:
  selector:
    app: user-service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP

INGRESS CONFIGURATION:
======================
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
  namespace: production
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - api.mycompany.com
    secretName: api-tls
  rules:
  - host: api.mycompany.com
    http:
      paths:
      - path: /users
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
      - path: /products
        pathType: Prefix
        backend:
          service:
            name: product-service
            port:
              number: 80
```

#### Service Mesh с Istio

```
ISTIO SERVICE MESH
==================

ISTIO ARCHITECTURE:
===================
┌─────────────────────────────────────────────────┐
│                Control Plane                    │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │
│ │   Pilot     │ │   Citadel   │ │   Galley    │ │
│ │(Discovery & │ │(Certificate │ │(Config      │ │
│ │ Config)     │ │ Management) │ │ Validation) │ │
│ └─────────────┘ └─────────────┘ └─────────────┘ │
└─────────────────────────────────────────────────┘
                          │
         ┌────────────────┼────────────────┐
         │                │                │
         ▼                ▼                ▼
    ┌─────────┐      ┌─────────┐      ┌─────────┐
    │ Envoy   │      │ Envoy   │      │ Envoy   │
    │ Proxy   │      │ Proxy   │      │ Proxy   │
    ├─────────┤      ├─────────┤      ├─────────┤
    │Service A│      │Service B│      │Service C│
    └─────────┘      └─────────┘      └─────────┘

TRAFFIC MANAGEMENT:
===================
# Virtual Service for Canary Deployment
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: user-service
spec:
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: user-service
        subset: v2
      weight: 100
  - route:
    - destination:
        host: user-service
        subset: v1
      weight: 90
    - destination:
        host: user-service
        subset: v2
      weight: 10

# Destination Rule
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: user-service
spec:
  host: user-service
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2

SECURITY POLICIES:
==================
# Require mTLS between services
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT

# Authorization Policy
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: user-service-policy
  namespace: production
spec:
  selector:
    matchLabels:
      app: user-service
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/production/sa/api-gateway"]
  - to:
    - operation:
        methods: ["GET", "POST"]
```

---

# 🎯 Контрольные задания блока 4

## 📝 Масштабный проект: Микросервисная архитектура онлайн-магазина

### Требования к проекту:

```
ПРОЕКТ: E-COMMERCE МИКРОСЕРВИСЫ
===============================

ФУНКЦИОНАЛЬНЫЕ ТРЕБОВАНИЯ:
==========================
✅ Регистрация и аутентификация пользователей
✅ Каталог товаров с поиском и фильтрацией
✅ Корзина покупок
✅ Оформление заказов
✅ Обработка платежей
✅ Управление складом
✅ Уведомления (email, push)
✅ Система отзывов и рейтингов
✅ Админ-панель

НЕФУНКЦИОНАЛЬНЫЕ ТРЕБОВАНИЯ:
============================
✅ 10,000 одновременных пользователей
✅ 99.9% availability
✅ Время отклика < 200ms для 95% запросов
✅ Горизонтальное масштабирование
✅ Отказоустойчивость
✅ Безопасность (PCI DSS compliance)
✅ Мониторинг и логирование
```

### Архитектурное решение:

```
АРХИТЕКТУРА РЕШЕНИЯ
===================

External Layer:
┌─────────────────────────────────────────────────┐
│     CDN      │    API Gateway    │    Web App   │
└─────────────────────────────────────────────────┘

Service Mesh Layer:
┌─────────────────────────────────────────────────┐
│                  Istio Service Mesh             │
└─────────────────────────────────────────────────┘

Microservices Layer:
┌─────────┐┌─────────┐┌─────────┐┌─────────┐
│  User   ││Product  ││  Cart   ││ Order   │
│Service  ││Service  ││Service  ││Service  │
└─────────┘└─────────┘└─────────┘└─────────┘

┌─────────┐┌─────────┐┌─────────┐┌─────────┐
│Payment  ││Inventory││Shipping ││Notification│
│Service  ││Service  ││Service  ││Service  │
└─────────┘└─────────┘└─────────┘└─────────┘

┌─────────┐┌─────────┐┌─────────┐
│ Review  ││Analytics││  Admin  │
│Service  ││Service  ││Service  │
└─────────┘└─────────┘└─────────┘

Data Layer:
┌─────────┐┌─────────┐┌─────────┐┌─────────┐
│PostgreSQL││MongoDB  ││ Redis   ││Elasticsearch│
│(OLTP)   ││(Catalog)││(Cache)  ││(Search) │
└─────────┘└─────────┘└─────────┘└─────────┘

Message Layer:
┌─────────────────────────────────────────────────┐
│              Apache Kafka                       │
└─────────────────────────────────────────────────┘

Infrastructure Layer:
┌─────────────────────────────────────────────────┐
│              Kubernetes Cluster                 │
└─────────────────────────────────────────────────┘
```

## 📊 Сравнительный анализ: Монолит vs Микросервисы

### Сценарий анализа:

```
БИЗНЕС-КЕЙС: СТАРТАП E-COMMERCE
===============================

Параметры:
- Команда: 5 разработчиков
- Пользователи: 1,000 в месяц (начальная стадия)
- Рост: план 100,000+ через 2 года
- Бюджет: Ограниченный
- Time to market: Критичен

АНАЛИЗ РЕШЕНИЙ:
===============

МОНОЛИТ (Рекомендуется для старта):
===================================
┌─────────────────────────────────────────────────┐
│                 МОНОЛИТ                         │
│ ┌─────────────────────────────────────────────┐ │
│ │     Single Codebase                         │ │
│ │  ┌─────────┐ ┌─────────┐ ┌─────────┐       │ │
│ │  │   UI    │ │  Logic  │ │   DB    │       │ │
│ │  └─────────┘ └─────────┘ └─────────┘       │ │
│ └─────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘

ПЛЮСЫ для стартапа:
✅ Быстрая разработка (1-2 месяца vs 4-6)
✅ Простое тестирование
✅ Легкий деплой
✅ Один сервер на старте
✅ Легче debugging
✅ Меньше operational overhead

МИНУСЫ:
❌ Сложнее масштабировать позже
❌ Technology lock-in
❌ Один deployment для всех изменений

МИКРОСЕРВИСЫ (Для масштабирования):
===================================
Переход к микросервисам при:
- Команда > 15 человек
- 50,000+ активных пользователей
- Разные требования к масштабированию
- Несколько продуктовых команд

РЕКОМЕНДАЦИЯ:
=============
🎯 Начать с МОНОЛИТА
🎯 Спроектировать модульную архитектуру
🎯 Планировать migration к микросервисам
🎯 Выделить первые сервисы через 12-18 месяцев
```

---

# 🎯 Результат блока 4

По завершении блока 4 вы получите:

✅ **Понимание современных архитектурных паттернов**
- Микросервисы vs монолит trade-offs
- Event-driven архитектура
- Service mesh концепции

✅ **Практические навыки**
- Декомпозиция приложений на сервисы
- Проектирование API Gateway
- Настройка service discovery
- Event sourcing и CQRS
- Kubernetes deployments

✅ **Портфолио проектов**
- Микросервисная архитектура e-commerce
- Event-driven система обработки заказов
- Kubernetes deployment стратегия
- Service mesh конфигурация

✅ **Архитектурное мышление**
- Conway's Law применение
- Distributed systems trade-offs
- Saga pattern для транзакций
- Container orchestration

---

# 📚 Дополнительные ресурсы

## 🔗 Полезные ссылки

```
КНИГИ:
======
📖 "Building Microservices" - Sam Newman
📖 "Microservices Patterns" - Chris Richardson  
📖 "Kubernetes in Action" - Marko Lukša
📖 "Event Storming" - Alberto Brandolini

ДОКУМЕНТАЦИЯ:
=============
🔗 Kubernetes Official Docs
🔗 Istio Documentation
🔗 Apache Kafka Documentation
🔗 Docker Best Practices

ПРАКТИЧЕСКИЕ РЕСУРСЫ:
=====================
🛠️ Minikube для локальной разработки
🛠️ Docker Desktop
🛠️ Katacoda Kubernetes Playground
🛠️ Istio Getting Started Guide

БЛОГИ И СООБЩЕСТВА:
===================
📝 Microservices.io (Chris Richardson)
📝 Kubernetes Blog
📝 CNCF Blog
📝 High Scalability Blog
```

## 🎯 Следующие шаги

```
ПОДГОТОВКА К БЛОКУ 5:
=====================

Изучить заранее:
┌─────────────────────────────────────┐
│ • OAuth 2.0 и OpenID Connect       │
│ • TLS/SSL протоколы                 │
│ • OWASP Top 10                      │
│ • Threat modeling основы            │
└─────────────────────────────────────┘

Практические задания:
┌─────────────────────────────────────┐
│ • Настроить HTTPS для приложения    │
│ • Реализовать JWT аутентификацию    │
│ • Провести security audit          │
│ • Изучить penetration testing      │
└─────────────────────────────────────┘
```

---

# 🎓 Заключение блока 4

Поздравляем! Вы успешно освоили один из самых сложных и востребованных разделов современной системной архитектуры. 

## 🌟 Что вы теперь умеете:

```
КЛЮЧЕВЫЕ КОМПЕТЕНЦИИ:
=====================

🔧 АРХИТЕКТУРНЫЕ РЕШЕНИЯ:
   ├── Выбор между монолитом и микросервисами
   ├── Декомпозиция приложений на сервисы  
   ├── Проектирование API Gateway
   └── Service discovery настройка

🔧 EVENT-DRIVEN СИСТЕМЫ:
   ├── Event sourcing и CQRS
   ├── Message brokers выбор и настройка
   ├── Saga pattern реализация
   └── Distributed transactions handling

🔧 СОВРЕМЕННЫЕ ТЕХНОЛОГИИ:
   ├── Kubernetes orchestration
   ├── Service mesh (Istio)
   ├── Container best practices
   └── Infrastructure as Code

🔧 ПРАКТИЧЕСКИЕ НАВЫКИ:
   ├── Микросервисное приложение с нуля
   ├── CI/CD для микросервисов
   ├── Monitoring и observability
   └── Production-ready deployments
```

## 🚀 Готовность к индустрии:

После завершения этого блока вы готовы:
- 👨‍💼 Работать Senior/Staff инженером в современных компаниях
- 🏗️ Проектировать enterprise-level архитектуры
- 🎤 Проводить архитектурные интервью
- 📈 Масштабировать системы для миллионов пользователей
- 🔧 Внедрять DevOps практики и cloud-native решения

---

*Переходите к Блоку 5: "Безопасность и аутентификация" для изучения security-first подходов в архитектуре систем!*