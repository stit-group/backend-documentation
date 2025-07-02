# 🔗 Блок 3: Взаимодействие между сервисами

---

## 📋 Обзор блока

**⏱️ Продолжительность:** 4-5 недель  
**🎯 Цель:** Изучить различные способы коммуникации между микросервисами и выбрать оптимальные решения  
**📊 Сложность:** ⭐⭐⭐⭐☆

---

## 🗺️ Карта обучения

```
Синхронная коммуникация → Асинхронная коммуникация → Event-driven архитектура
           ↓                        ↓                           ↓
    REST/GraphQL/gRPC        Message Queues/Kafka        Domain Events/CQRS
           ↓                        ↓                           ↓
          Паттерны интеграции ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ←
                    ↓
              Resilience и fault tolerance
```

---

## 🎯 Глава 1: Синхронная коммуникация

### 📖 Теоретическая основа

**Синхронная коммуникация** — это взаимодействие, при котором клиент отправляет запрос и ожидает ответа в режиме реального времени.

#### 🔄 Принципы синхронной коммуникации

```
┌─────────────┐    HTTP Request     ┌─────────────┐
│   Service   │ ──────────────────→ │   Service   │
│      A      │                     │      B      │
│             │ ←────────────────── │             │
└─────────────┘    HTTP Response    └─────────────┘
       ↑                                   ↑
   Блокируется                         Обрабатывает
   до получения                          запрос
    ответа
```

### 🛠️ REST API Design Principles

#### ✅ Основные принципы REST

| Принцип | Описание | Пример |
|---------|----------|--------|
| **Stateless** | Каждый запрос содержит всю необходимую информацию | JWT токены вместо сессий |
| **Uniform Interface** | Единообразный интерфейс для всех ресурсов | Стандартные HTTP методы |
| **Resource-based** | URL представляют ресурсы, а не действия | `/users/123` вместо `/getUser?id=123` |
| **HATEOAS** | Hypermedia as the Engine of Application State | Ссылки на связанные ресурсы |

#### 🎨 REST API Best Practices

```markdown
✅ Хорошо:
GET    /api/v1/users           # Получить всех пользователей
GET    /api/v1/users/123       # Получить пользователя по ID
POST   /api/v1/users           # Создать пользователя
PUT    /api/v1/users/123       # Обновить пользователя
DELETE /api/v1/users/123       # Удалить пользователя

❌ Плохо:
GET    /api/v1/getUsers
POST   /api/v1/createUser
POST   /api/v1/updateUser
POST   /api/v1/deleteUser
```

### 🚀 GraphQL для агрегации данных

#### 🎯 Преимущества GraphQL

```
┌─────────────────────────────────────────────────────────────┐
│                    REST vs GraphQL                         │
├─────────────────────────────────────────────────────────────┤
│ REST:                      │ GraphQL:                       │
│ ┌─────┐  ┌─────┐  ┌─────┐  │ ┌─────┐                        │
│ │ GET │  │ GET │  │ GET │  │ │Query│                        │
│ │Users│  │Posts│  │Tags │  │ │ All │                        │
│ └─────┘  └─────┘  └─────┘  │ └─────┘                        │
│    ↓        ↓        ↓     │    ↓                           │
│ 3 запроса              │ 1 запрос                      │
│ Over-fetching          │ Точные данные                 │
│ Under-fetching         │ Нет лишних полей              │
└─────────────────────────────────────────────────────────────┘
```

#### 📝 Пример GraphQL запроса

```graphql
query GetUserWithPosts($userId: ID!) {
  user(id: $userId) {
    id
    name
    email
    posts {
      id
      title
      createdAt
      comments {
        id
        content
        author {
          name
        }
      }
    }
  }
}
```

### ⚡ gRPC для high-performance коммуникации

#### 🔧 Архитектура gRPC

```
┌──────────────────────────────────────────────────────────────┐
│                     gRPC Architecture                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ ┌─────────────┐                          ┌─────────────┐    │
│ │   Client    │ ←──── Protocol Buffers ──→ │   Server    │    │
│ │ Application │         (Binary)           │ Application │    │
│ └─────────────┘                          └─────────────┘    │
│       ↑                                          ↑           │
│ ┌─────────────┐                          ┌─────────────┐    │
│ │ gRPC Client │ ←────── HTTP/2 Stream ────→ │ gRPC Server │    │
│ │    Stub     │         Connection         │   Handler   │    │
│ └─────────────┘                          └─────────────┘    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

#### 📊 Сравнение протоколов

| Характеристика | REST/JSON | GraphQL | gRPC |
|---------------|-----------|---------|------|
| **Производительность** | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Читаемость** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| **Типизация** | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Кэширование** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐ |
| **Поддержка браузеров** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |

---

## 🎯 Глава 2: Асинхронная коммуникация

### 📖 Основы асинхронного взаимодействия

```
┌─────────────┐                    ┌─────────────┐
│   Service   │ ──── Message ────→ │   Message   │
│      A      │                    │    Queue    │
│             │                    │             │
└─────────────┘                    └─────────────┘
       ↑                                  │
   Не блокируется                         │
   продолжает работу                      ↓
                                 ┌─────────────┐
                                 │   Service   │
                                 │      B      │
                                 │             │
                                 └─────────────┘
```

### 🐰 RabbitMQ: Основы работы с очередями

#### 🏗️ Архитектура RabbitMQ

```
Producer → Exchange → Queue → Consumer
    ↓         ↓        ↓        ↓
 Отправляет  Маршрут-  Хранит   Обрабатывает
 сообщения   изирует   сообщения сообщения
```

#### 📋 Типы Exchange в RabbitMQ

| Тип | Описание | Схема маршрутизации |
|-----|----------|-------------------|
| **Direct** | Точное совпадение routing key | `routing_key = queue_binding_key` |
| **Topic** | Паттерн-matching | `logs.*.error`, `logs.#` |
| **Fanout** | Broadcast всем очередям | Игнорирует routing key |
| **Headers** | Маршрутизация по заголовкам | Сложные правила фильтрации |

#### 🎨 Визуализация паттернов RabbitMQ

```
┌─────────────────────────────────────────────────────────────┐
│                    Work Queue Pattern                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Producer ──→ [===Queue===] ──→ Worker 1                     │
│                   │                                         │
│                   └──────────→ Worker 2                     │
│                   │                                         │
│                   └──────────→ Worker 3                     │
│                                                             │
│ ✅ Load balancing между workers                             │
│ ✅ Fault tolerance                                          │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                 Publish/Subscribe Pattern                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Producer ──→ [Exchange] ──→ Queue 1 ──→ Consumer 1          │
│                   │                                         │
│                   ├──────→ Queue 2 ──→ Consumer 2          │
│                   │                                         │
│                   └──────→ Queue 3 ──→ Consumer 3          │
│                                                             │
│ ✅ Broadcast сообщений                                     │
│ ✅ Децентрализованная обработка                            │
└─────────────────────────────────────────────────────────────┘
```

### 🌊 Apache Kafka: Event Streaming Platform

#### 🏛️ Архитектура Kafka

```
┌──────────────────────────────────────────────────────────────┐
│                    Kafka Architecture                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ ┌─────────────┐    ┌──────────────────────────────────┐     │
│ │  Producer   │───→│            Topic                 │     │
│ │             │    │  ┌─────┬─────┬─────┬─────┬─────┐  │     │
│ └─────────────┘    │  │ P0  │ P1  │ P2  │ P3  │ P4  │  │     │
│                    │  └─────┴─────┴─────┴─────┴─────┘  │     │
│ ┌─────────────┐    │         (Partitions)             │     │
│ │  Producer   │───→│                                  │     │
│ │             │    └──────────────────────────────────────┘     │
│ └─────────────┘                    │                        │
│                                    ↓                        │
│                    ┌─────────────┐ ┌─────────────┐         │
│                    │ Consumer    │ │ Consumer    │         │
│                    │ Group A     │ │ Group B     │         │
│                    │             │ │             │         │
│                    └─────────────┘ └─────────────┘         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

#### 📊 Kafka vs RabbitMQ

| Характеристика | RabbitMQ | Kafka |
|---------------|----------|-------|
| **Производительность** | 20k сообщений/сек | 1M+ сообщений/сек |
| **Персистентность** | Опционально | По умолчанию |
| **Ordering** | По очереди | По партиции |
| **Replay** | Нет | Да |
| **Сложность** | Низкая | Высокая |
| **Use Case** | Task queues | Event streaming |

### 🎯 Pub/Sub паттерн в деталях

#### 🔄 Event Flow диаграмма

```
┌─────────────────────────────────────────────────────────────┐
│                Event-Driven Communication                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ ┌─────────────┐   1. User Created   ┌─────────────────────┐ │
│ │    User     │ ─────────────────→  │    Event Bus        │ │
│ │  Service    │                     │   (Kafka/RabbitMQ)  │ │
│ └─────────────┘                     └─────────────────────┘ │
│                                              │               │
│                                              ↓               │
│ ┌─────────────┐   2. Process Event  ┌─────────────────────┐ │
│ │   Email     │ ←─────────────────  │    Event Router     │ │
│ │  Service    │                     │                     │ │
│ └─────────────┘                     └─────────────────────┘ │
│                                              │               │
│ ┌─────────────┐   3. Process Event           │               │
│ │ Analytics   │ ←────────────────────────────┘               │
│ │  Service    │                                             │
│ └─────────────┘                                             │
│                                                             │
│ ┌─────────────┐   4. Process Event                         │
│ │ Notification│ ←──────────────────────────────────────────┘ │
│ │  Service    │                                             │
│ └─────────────┘                                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎯 Глава 3: Event-driven архитектура

### 🎭 Domain Events концепция

#### 📋 Что такое Domain Event?

**Domain Event** — это что-то важное, что произошло в домене и на что могут реагировать другие части системы.

```
┌─────────────────────────────────────────────────────────────┐
│                  Domain Event Lifecycle                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ ┌─────────────┐  1. Business Action  ┌─────────────────────┐ │
│ │   Domain    │ ─────────────────→   │   Domain Event      │ │
│ │   Model     │                      │   Generated         │ │
│ └─────────────┘                      └─────────────────────┘ │
│                                              │               │
│                                              ↓               │
│ ┌─────────────┐  2. Event Published  ┌─────────────────────┐ │
│ │   Event     │ ←─────────────────   │    Event Store      │ │
│ │  Handlers   │                      │   (Persistent)      │ │
│ └─────────────┘                      └─────────────────────┘ │
│       │                                      │               │
│       ↓                                      ↓               │
│ ┌─────────────┐  3. Side Effects     ┌─────────────────────┐ │
│ │  External   │                      │   Event History     │ │
│ │  Services   │                      │   (Audit Trail)     │ │
│ └─────────────┘                      └─────────────────────┘ │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### 🏗️ Event Storming процесс

**Event Storming** — это workshop техника для исследования домена через события.

```
┌─────────────────────────────────────────────────────────────┐
│                    Event Storming Board                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ 🟠 Events     🔵 Commands     🟡 Actors     🟢 Read Models  │
│                                                             │
│ Order        ←── Place Order ←── Customer ──→ Order Status  │
│ Placed                                                      │
│   │                                                         │
│   ↓                                                         │
│ Payment      ←── Process     ←── Payment ──→ Payment        │
│ Processed        Payment         Service     History        │
│   │                                                         │
│   ↓                                                         │
│ Item         ←── Reserve     ←── Inventory ─→ Stock         │
│ Reserved         Items           Service      Levels        │
│   │                                                         │
│   ↓                                                         │
│ Order        ←── Ship Order ←── Shipping ──→ Tracking       │
│ Shipped                          Service     Info           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 📚 Event Sourcing основы

#### 💾 Traditional vs Event Sourcing

```
┌─────────────────────────────────────────────────────────────┐
│              Traditional State Storage                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Database Table: accounts                                    │
│ ┌────┬─────────┬─────────────┬─────────────────────────────┐ │
│ │ ID │  Name   │   Balance   │       Last_Updated          │ │
│ ├────┼─────────┼─────────────┼─────────────────────────────┤ │
│ │ 1  │  John   │   $1,500    │    2024-03-15 10:30:00     │ │
│ │ 2  │  Alice  │   $2,300    │    2024-03-15 14:20:00     │ │
│ └────┴─────────┴─────────────┴─────────────────────────────┘ │
│                                                             │
│ ❌ История изменений потеряна                              │
│ ❌ Невозможно восстановить состояние на момент времени      │
│ ❌ Сложно отладить проблемы                                │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    Event Sourcing                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Event Store: account_events                                 │
│ ┌──────┬─────────────┬─────────────┬─────────────────────────┐ │
│ │ Seq  │    Type     │    Data     │      Timestamp          │ │
│ ├──────┼─────────────┼─────────────┼─────────────────────────┤ │
│ │  1   │  Created    │ {id:1, ...} │  2024-03-01 09:00:00   │ │
│ │  2   │  Deposited  │ {amount:500}│  2024-03-05 14:30:00   │ │
│ │  3   │  Withdrawn  │ {amount:100}│  2024-03-10 11:15:00   │ │
│ │  4   │  Deposited  │ {amount:300}│  2024-03-15 10:30:00   │ │
│ └──────┴─────────────┴─────────────┴─────────────────────────┘ │
│                              ↓                              │
│                  Current State: $700                        │
│                                                             │
│ ✅ Полная история изменений                                │
│ ✅ Time travel возможен                                    │
│ ✅ Audit trail из коробки                                  │
│ ✅ Воспроизведение состояния в любой момент                │
└─────────────────────────────────────────────────────────────┘
```

### 🎯 CQRS (Command Query Responsibility Segregation)

#### ⚖️ CQRS Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    CQRS Pattern                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│            Commands (Write Side)                           │
│ ┌─────────────┐    ┌─────────────┐    ┌─────────────────┐   │
│ │   Client    │───→│  Command    │───→│   Write Model   │   │
│ │             │    │  Handler    │    │  (Event Store)  │   │
│ └─────────────┘    └─────────────┘    └─────────────────┘   │
│                                               │             │
│                                               ↓             │
│                                    ┌─────────────────┐      │
│                                    │     Events      │      │
│                                    │   Published     │      │
│                                    └─────────────────┘      │
│                                               │             │
│                                               ↓             │
│            Queries (Read Side)                              │
│ ┌─────────────┐    ┌─────────────┐    ┌─────────────────┐   │
│ │   Client    │←───│   Query     │←───│   Read Models   │   │
│ │             │    │  Handler    │    │  (Projections)  │   │
│ └─────────────┘    └─────────────┘    └─────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### 📊 CQRS Benefits

| Аспект | Преимущество |
|--------|-------------|
| **Масштабируемость** | Независимое масштабирование read/write |
| **Производительность** | Оптимизированные read models |
| **Сложность** | Разделение бизнес-логики |
| **Гибкость** | Множественные представления данных |

---

## 🎯 Глава 4: Паттерны интеграции

### 🌊 Saga Pattern для Distributed Transactions

#### 🎭 Choreography vs Orchestration

```
┌─────────────────────────────────────────────────────────────┐
│                  Choreography Saga                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ 1. Order Service ──→ "Order Created" ──→ Event Bus         │
│                                                ↓            │
│ 2. Payment Service ←─── Listens ←──────────────┘            │
│         │                                                   │
│         ↓                                                   │
│ 3. "Payment Processed" ──→ Event Bus                       │
│                                  ↓                          │
│ 4. Inventory Service ←── Listens ┘                         │
│         │                                                   │
│         ↓                                                   │
│ 5. "Items Reserved" ──→ Event Bus                          │
│                              ↓                              │
│ 6. Shipping Service ←─ Listens                             │
│                                                             │
│ ✅ Децентрализованное управление                           │
│ ❌ Сложно отслеживать состояние                            │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                  Orchestration Saga                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│              ┌─────────────────────┐                       │
│              │   Saga Manager      │                       │
│              │   (Orchestrator)    │                       │
│              └─────────────────────┘                       │
│                    │  │  │  │                              │
│          ┌─────────┘  │  │  └─────────┐                    │
│          ↓            ↓  ↓            ↓                    │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │   Payment   │ │ Inventory   │ │  Shipping   │           │
│ │   Service   │ │  Service    │ │   Service   │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
│                                                             │
│ ✅ Централизованное управление                             │
│ ✅ Легко отслеживать состояние                             │
│ ❌ Single point of failure                                 │
└─────────────────────────────────────────────────────────────┘
```

#### 🔄 Saga State Machine

```
┌─────────────────────────────────────────────────────────────┐
│                  Order Saga States                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ ┌─────────────┐  Success   ┌─────────────┐  Success        │
│ │   Started   │ ────────→  │  Payment    │ ────────→       │
│ └─────────────┘            │ Processing  │                 │
│       │                    └─────────────┘                 │
│       │ Failure                   │ Failure                │
│       ↓                           ↓                        │
│ ┌─────────────┐            ┌─────────────┐                 │
│ │   Failed    │            │  Payment    │                 │
│ │             │            │ Cancelled   │                 │
│ └─────────────┘            └─────────────┘                 │
│                                                             │
│ ┌─────────────┐  Success   ┌─────────────┐  Success        │
│ │ Inventory   │ ────────→  │  Shipping   │ ────────→       │
│ │ Reserved    │            │ Processing  │                 │
│ └─────────────┘            └─────────────┘                 │
│       ↑                           │                        │
│       │                           ↓                        │
│ ┌─────────────┐            ┌─────────────┐                 │
│ │ Compensate  │            │  Completed  │                 │
│ │ Inventory   │            │             │                 │
│ └─────────────┘            └─────────────┘                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 📦 Outbox Pattern

#### 📋 Проблема и решение

```
┌─────────────────────────────────────────────────────────────┐
│                   Problem: Dual Write                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ ┌─────────────┐   1. Update DB    ┌─────────────────────┐   │
│ │  Service    │ ─────────────────→│     Database        │   │
│ │             │                   │                     │   │
│ │             │   2. Send Event   │                     │   │
│ │             │ ─────────────────→│ ❌ What if this     │   │
│ │             │                   │    fails?           │   │
│ └─────────────┘                   └─────────────────────┘   │
│                                                             │
│ ❌ Inconsistency между DB и Event Bus                      │
│ ❌ Lost events                                             │
│ ❌ Duplicate events                                        │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                 Solution: Outbox Pattern                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ ┌─────────────┐                 ┌─────────────────────────┐ │
│ │  Service    │ ──── 1 TX ────→ │      Database           │ │
│ │             │                 │  ┌─────────────────────┐│ │
│ │             │                 │  │  Business Tables   ││ │
│ │             │                 │  └─────────────────────┘│ │
│ │             │                 │  ┌─────────────────────┐│ │
│ │             │                 │  │   Outbox Table     ││ │
│ │             │                 │  │  (Events to send)  ││ │
│ │             │                 │  └─────────────────────┘│ │
│ └─────────────┘                 └─────────────────────────┘ │
│                                          │                  │
│                                          ↓                  │
│ ┌─────────────┐                 ┌─────────────────────────┐ │
│ │  Message    │ ←── Polling ──  │   Outbox Publisher      │ │
│ │   Broker    │                 │    (Background Job)     │ │
│ └─────────────┘                 └─────────────────────────┘ │
│                                                             │
│ ✅ Transactional consistency                               │
│ ✅ At-least-once delivery                                  │
│ ✅ No lost events                                          │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎯 Глава 5: Resilience и Fault Tolerance

### ⚡ Circuit Breaker Pattern

#### 🔌 Circuit Breaker States

```
┌─────────────────────────────────────────────────────────────┐
│                Circuit Breaker States                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    ┌─────────────┐                         │
│               ┌───→│   CLOSED    │←──┐                      │
│               │    │             │   │                      │
│               │    │ ✅ Normal   │   │ Success threshold    │
│               │    │ operation   │   │ reached              │
│               │    └─────────────┘   │                      │
│               │           │          │                      │
│               │ Timeout   │ Failure  │                      │
│               │           │ threshold│                      │
│               │           ↓          │                      │
│    ┌─────────────┐    ┌─────────────┐                      │
│    │ HALF-OPEN   │    │    OPEN     │                      │
│    │             │    │             │                      │
│    │ 🔄 Testing  │    │ ❌ Fail     │                      │
│    │ recovery    │    │ fast        │                      │
│    └─────────────┘    └─────────────┘                      │
│           │                   │                            │
│           └───────────────────┘                            │
│                  Timeout                                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### 🎨 Circuit Breaker Implementation

```javascript
class CircuitBreaker {
  constructor(service, options = {}) {
    this.service = service;
    this.failureThreshold = options.failureThreshold || 5;
    this.timeout = options.timeout || 60000;
    this.state = 'CLOSED';
    this.failureCount = 0;
    this.lastFailureTime = null;
  }

  async call(...args) {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > this.timeout) {
        this.state = 'HALF-OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }

    try {
      const result = await this.service(...args);
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }

  onFailure() {
    this.failureCount++;
    this.lastFailureTime = Date.now();
    
    if (this.failureCount >= this.failureThreshold) {
      this.state = 'OPEN';
    }
  }
}
```

### ⏱️ Timeout и Retry Policies

#### 🔄 Exponential Backoff

```
┌─────────────────────────────────────────────────────────────┐
│                Exponential Backoff Pattern                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Attempt 1: ──×── Wait 1s  ─→ Attempt 2: ──×── Wait 2s  ─→  │
│                                                             │
│ Attempt 3: ──×── Wait 4s  ─→ Attempt 4: ──×── Wait 8s  ─→  │
│                                                             │
│ Attempt 5: ──×── Wait 16s ─→ Give Up                       │
│                                                             │
│ Base delay: 1s                                              │
│ Multiplier: 2                                               │
│ Max attempts: 5                                             │
│ Jitter: ±20% (prevents thundering herd)                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 🚧 Bulkhead Pattern

#### 🏗️ Resource Isolation

```
┌─────────────────────────────────────────────────────────────┐
│                    Bulkhead Pattern                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │                Application                              │ │
│ ├─────────────────────────────────────────────────────────┤ │
│ │ ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐ │ │
│ │ │  Critical   │ │  Standard   │ │     Batch           │ │ │
│ │ │  Requests   │ │  Requests   │ │   Processing        │ │ │
│ │ │             │ │             │ │                     │ │ │
│ │ │ Pool: 20    │ │ Pool: 30    │ │   Pool: 10          │ │ │
│ │ │ threads     │ │ threads     │ │   threads           │ │ │
│ │ └─────────────┘ └─────────────┘ └─────────────────────┘ │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
│ ✅ Failure isolation                                       │
│ ✅ Performance isolation                                   │
│ ✅ Resource allocation control                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 📋 Практические задания

### 🚀 Задание 1: REST API с Error Handling

**Цель:** Создать REST API с правильной обработкой ошибок

```markdown
📋 Требования:
- [ ] CRUD операции для User entity
- [ ] Валидация входных данных
- [ ] Structured error responses
- [ ] HTTP status codes
- [ ] API versioning
- [ ] Request/Response logging

🎯 Критерии оценки:
- Соответствие REST принципам
- Качество error handling
- Документация API (OpenAPI/Swagger)
```

### 🌊 Задание 2: Kafka Event Processing

**Цель:** Настроить асинхронную коммуникацию через Kafka

```markdown
📋 Требования:
- [ ] Producer для Order events
- [ ] Consumer groups для разных сервисов
- [ ] Dead letter queue для failed messages
- [ ] Event schema evolution
- [ ] Monitoring и alerting

🎯 Критерии оценки:
- Правильная конфигурация Kafka
- Обработка сбоев
- Производительность
```

### ⚡ Задание 3: Circuit Breaker Implementation

**Цель:** Реализовать Circuit Breaker pattern

```markdown
📋 Требования:
- [ ] Circuit breaker library/implementation
- [ ] Configurable thresholds
- [ ] Metrics collection
- [ ] Health check endpoint
- [ ] Graceful degradation

🎯 Критерии оценки:
- Корректность работы states
- Monitoring capabilities
- Integration testing
```

### 🌊 Задание 4: Event-driven Workflow

**Цель:** Создать event-driven workflow

```markdown
📋 Требования:
- [ ] Order processing saga
- [ ] Compensation actions
- [ ] Event sourcing for audit
- [ ] CQRS read models
- [ ] Workflow visualization

🎯 Критерии оценки:
- Consistency гарантии
- Error recovery
- Performance
```

---

## 📊 Итоговая оценка блока

### ✅ Чек-лист знаний

| Тема | Понимание | Практика |
|------|-----------|----------|
| REST API Design | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| GraphQL | ⭐⭐⭐⭐☆ | ⭐⭐⭐☆☆ |
| gRPC | ⭐⭐⭐☆☆ | ⭐⭐☆☆☆ |
| Message Queues | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐☆ |
| Event Sourcing | ⭐⭐⭐⭐☆ | ⭐⭐⭐☆☆ |
| CQRS | ⭐⭐⭐⭐☆ | ⭐⭐⭐☆☆ |
| Saga Pattern | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐☆ |
| Circuit Breaker | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐☆ |

### 🎯 Следующие шаги

После завершения этого блока вы должны:

✅ **Понимать** различные способы коммуникации между сервисами  
✅ **Уметь выбирать** подходящий паттерн для конкретной задачи  
✅ **Реализовывать** resilient коммуникацию  
✅ **Проектировать** event-driven архитектуры  

---

## 🔗 Полезные ресурсы

### 📚 Документация
- [REST API Design Guide](https://restfulapi.net/)
- [GraphQL Documentation](https://graphql.org/learn/)
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [RabbitMQ Tutorials](https://www.rabbitmq.com/tutorials/tutorial-one-python.html)

### 🛠️ Инструменты
- [Postman](https://www.postman.com/) - API testing
- [Kafka Tool](http://www.kafkatool.com/) - Kafka GUI
- [Insomnia](https://insomnia.rest/) - API client
- [AsyncAPI](https://www.asyncapi.com/) - Event-driven API documentation

### 📖 Книги
- "Enterprise Integration Patterns" - Gregor Hohpe
- "Designing Event-Driven Systems" - Ben Stopford
- "Microservices Patterns" - Chris Richardson

---

**🎉 Готовы к следующему блоку? Переходим к управлению данными в микросервисах!**