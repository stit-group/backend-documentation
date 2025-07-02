# БЛОК 4: ПРОЕКТИРОВАНИЕ И АРХИТЕКТУРА
## Создание масштабируемых и надежных систем обмена сообщениями

---

### 🎯 **ЦЕЛЬ БЛОКА**
Освоить принципы проектирования сложных distributed систем с правильными архитектурными решениями для enterprise-уровня приложений.

**Продолжительность:** 4-5 недель  
**Уровень сложности:** ⭐⭐⭐⭐⭐

---

## **ГЛАВА 4.1: ENTERPRISE INTEGRATION PATTERNS (EIP)**
### 📚 Продолжительность: 1.5 недели

#### **Теоретическая основа**

Enterprise Integration Patterns — это набор проверенных решений для интеграции enterprise приложений через асинхронный обмен сообщениями.

```
┌─────────────────────────────────────────────────────────────┐
│                    EIP АРХИТЕКТУРА                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [App A] ──► [Message] ──► [Channel] ──► [Message] ──► [App B] │
│                              │                              │
│                              ▼                              │
│                         [Router/Filter]                     │
│                              │                              │
│                              ▼                              │
│                         [Transform]                         │
│                              │                              │
│                              ▼                              │
│                         [Endpoint]                          │
└─────────────────────────────────────────────────────────────┘
```

#### **Основные паттерны**

##### 1. **MESSAGE CHANNEL**
```
Назначение: Соединение двух приложений через messaging систему

┌─────────┐    Message     ┌─────────────┐    Message     ┌─────────┐
│Producer │ ────────────► │   Channel   │ ────────────► │Consumer │
└─────────┘               └─────────────┘               └─────────┘

Типы каналов:
├── Point-to-Point Channel (Queue)
├── Publish-Subscribe Channel (Topic)
├── Dead Letter Channel
└── Invalid Message Channel
```

##### 2. **MESSAGE ROUTER**
```
Назначение: Маршрутизация сообщений на основе содержимого

                    ┌─ [Channel A] ──► [Consumer A]
                    │
[Producer] ──► [Router] ─── [Channel B] ──► [Consumer B]
                    │
                    └─ [Channel C] ──► [Consumer C]

Критерии маршрутизации:
├── Message Type
├── Message Content
├── Message Header
└── Context Information
```

##### 3. **CONTENT-BASED ROUTER**
```
Пример маршрутизации заказов:

┌─────────────┐     ┌──────────────────┐
│   Order     │────►│ Content Router   │
│ Message     │     │                  │
└─────────────┘     │ if (region ==    │──► [EU Processing]
                    │     "EU")        │
                    │ else if (amount  │──► [VIP Processing]
                    │     > 10000)     │
                    │ else             │──► [Standard Processing]
                    └──────────────────┘
```

##### 4. **MESSAGE FILTER**
```
Назначение: Фильтрация нежелательных сообщений

[All Messages] ──► [Filter] ──► [Relevant Messages] ──► [Consumer]
                      │
                      ▼
                 [Discarded Messages]

Примеры фильтров:
├── Priority Filter (priority > 5)
├── Content Filter (contains "urgent")
├── Source Filter (from trusted sources)
└── Time Filter (created within last hour)
```

##### 5. **AGGREGATOR PATTERN**
```
Назначение: Сбор связанных сообщений в одно составное

[Message 1] ─┐
             ├─► [Aggregator] ──► [Composite Message] ──► [Consumer]
[Message 2] ─┘      │
                    │ Correlation ID
                    │ Timeout: 30s
                    │ Complete when: count = 3

Стратегии агрегации:
├── Count-based (собрать N сообщений)
├── Time-based (собрать за T времени)
├── Content-based (до получения End marker)
└── Custom logic
```

##### 6. **SPLITTER PATTERN**
```
Назначение: Разделение одного сообщения на несколько

[Composite Message] ──► [Splitter] ──► [Message 1] ──► [Consumer A]
                                   ├─► [Message 2] ──► [Consumer B]
                                   └─► [Message 3] ──► [Consumer C]

Примеры:
├── Order → OrderItems
├── Batch → Individual Records
├── File → Lines
└── Array → Elements
```

##### 7. **CORRELATION IDENTIFIER**
```
Назначение: Связывание сообщений в рамках одной бизнес-транзакции

Request:
┌─────────────────────────────────────┐
│ CorrelationId: "12345"              │
│ Message: "Process Order"            │
│ OrderId: "ORD-001"                  │
└─────────────────────────────────────┘
                    │
                    ▼
Response:
┌─────────────────────────────────────┐
│ CorrelationId: "12345"              │
│ Message: "Order Processed"          │
│ Status: "SUCCESS"                   │
└─────────────────────────────────────┘
```

#### **Практическое задание 4.1**

**Задача:** Реализовать систему обработки заказов с использованием EIP

```
Требования:
├── Content-Based Router для типов заказов
├── Message Filter для приоритетных заказов  
├── Aggregator для группировки по клиенту
├── Splitter для разделения на позиции
└── Correlation ID для отслеживания
```

**Пример реализации на Java/Spring Integration:**

```java
@Configuration
@EnableIntegration
public class OrderProcessingFlow {
    
    @Bean
    public IntegrationFlow orderFlow() {
        return IntegrationFlows
            .from("orders.input")
            .filter("headers['priority'] == 'HIGH'")
            .route("payload.orderType", r -> r
                .channelMapping("STANDARD", "standard.channel")
                .channelMapping("EXPRESS", "express.channel")
                .channelMapping("BULK", "bulk.channel"))
            .get();
    }
    
    @Bean
    public IntegrationFlow aggregatorFlow() {
        return IntegrationFlows
            .from("bulk.channel")
            .aggregate(a -> a
                .correlationStrategy(m -> m.getHeaders().get("customerId"))
                .releaseStrategy(g -> g.size() >= 10)
                .sendTimeout(30000))
            .channel("aggregated.orders")
            .get();
    }
}
```

---

## **ГЛАВА 4.2: EVENT SOURCING И CQRS**
### 📚 Продолжительность: 1.5 недели

#### **Event Sourcing - Основы**

Event Sourcing — это паттерн, где изменения состояния приложения сохраняются как последовательность событий.

```
ТРАДИЦИОННЫЙ ПОДХОД vs EVENT SOURCING

Традиционный:                    Event Sourcing:
┌─────────────┐                 ┌─────────────────────────────┐
│   Account   │                 │        Event Store          │
├─────────────┤                 ├─────────────────────────────┤
│ id: 123     │                 │ 1. AccountCreated           │
│ balance: 50 │   ◄────────     │ 2. DepositMade (+100)       │
│ status: ACT │                 │ 3. WithdrawalMade (-50)     │
└─────────────┘                 │ 4. StatusChanged (ACTIVE)   │
                                └─────────────────────────────┘
```

#### **Event Store Design**

```
┌─────────────────────────────────────────────────────────────┐
│                        EVENT STORE                          │
├─────────────────────────────────────────────────────────────┤
│ EventId │ AggregateId │ EventType │ EventData │ Timestamp   │
├─────────────────────────────────────────────────────────────┤
│ e1      │ acc-123     │ Created   │ {...}     │ 2025-01-01  │
│ e2      │ acc-123     │ Deposited │ {...}     │ 2025-01-02  │
│ e3      │ acc-123     │ Withdrawn │ {...}     │ 2025-01-03  │
└─────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────┐
│                    EVENT REPLAY                             │
│                                                             │
│ State = fold(events, initialState)                         │
│                                                             │
│ Account balance = 0 + 100 - 50 = 50                        │
└─────────────────────────────────────────────────────────────┘
```

#### **CQRS (Command Query Responsibility Segregation)**

```
АРХИТЕКТУРА CQRS:

┌─────────────┐    Commands    ┌─────────────┐    Events    ┌─────────────┐
│   Client    │ ────────────► │   Command   │ ──────────► │    Event    │
│             │               │    Side     │             │    Store    │
└─────────────┘               └─────────────┘             └─────────────┘
        │                                                         │
        │                                                         │ Events
        │ Queries           ┌─────────────┐                       ▼
        └─────────────────► │    Query    │ ◄─────────── ┌─────────────┐
                           │    Side     │               │    Read     │
                           └─────────────┘               │   Models    │
                                                        └─────────────┘

Преимущества:
├── Независимое масштабирование чтения и записи
├── Оптимизированные модели для запросов
├── Eventual consistency
└── Audit trail из коробки
```

#### **Event Versioning и Migration**

```
ВЕРСИОНИРОВАНИЕ СОБЫТИЙ:

Version 1:                          Version 2:
┌─────────────────────────┐        ┌─────────────────────────┐
│ {                       │   →    │ {                       │
│   "type": "OrderPlaced" │        │   "type": "OrderPlaced" │
│   "orderId": "123"      │        │   "orderId": "123"      │
│   "amount": 100         │        │   "amount": 100         │
│ }                       │        │   "currency": "USD"     │ ← Новое поле
└─────────────────────────┘        │   "version": 2          │ ← Версия
                                   └─────────────────────────┘

СТРАТЕГИИ МИГРАЦИИ:
├── Upcast (преобразование при чтении)
├── Copy-Transform (создание новых событий)
├── Weak Schema (гибкая схема)
└── Event Migration (batch обновление)
```

#### **Snapshotting Strategies**

```
БЕЗ СНАПШОТОВ:
Event 1 → Event 2 → Event 3 → ... → Event 1000 → Current State
           (медленно при большом количестве событий)

СО СНАПШОТАМИ:
Snapshot (Event 900) → Event 901 → Event 902 → ... → Event 1000
                       (быстрое восстановление состояния)

┌─────────────────────────────────────────────────────────────┐
│                   SNAPSHOT STRATEGY                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ if (eventCount % 100 == 0) {                               │
│     saveSnapshot(aggregateId, currentState, eventVersion)  │
│ }                                                           │
│                                                             │
│ Восстановление:                                             │
│ 1. Загрузить последний снапшот                              │
│ 2. Применить события после снапшота                         │
│ 3. Получить актуальное состояние                            │
└─────────────────────────────────────────────────────────────┘
```

#### **Практическое задание 4.2**

**Задача:** Построить Event Sourced систему управления банковскими счетами

```
События:
├── AccountCreated
├── DepositMade  
├── WithdrawalMade
├── AccountClosed
└── InterestCalculated

Read Models:
├── AccountBalance (быстрый доступ к балансу)
├── TransactionHistory (история операций)
├── AccountStatement (выписка за период)
└── DailyReport (агрегированная отчетность)
```

---

## **ГЛАВА 4.3: DISTRIBUTED TRANSACTIONS И SAGA PATTERN**
### 📚 Продолжительность: 1 неделя

#### **Проблемы Distributed Transactions**

```
ПРОБЛЕМА TWO-PHASE COMMIT:

Phase 1 (Prepare):              Phase 2 (Commit):
                               
Coordinator ──► Service A       Coordinator ──► Service A
           ├──► Service B                  ├──► Service B  
           └──► Service C                  └──► Service C

Проблемы:
├── Блокирующий протокол
├── Single point of failure (Coordinator)
├── Долгие блокировки ресурсов
└── Не подходит для микросервисов
```

#### **Saga Pattern - Решение**

```
SAGA = Последовательность локальных транзакций с компенсацией

CHOREOGRAPHY SAGA:
┌─────────────────────────────────────────────────────────────┐
│                     ORDER SAGA                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ [Order Service] ──► OrderCreated ──► [Payment Service]      │
│        │                                     │              │
│        │                                     ▼              │
│        │                            PaymentProcessed ───────┤
│        │                                     │              │
│        │                                     ▼              │
│        │                            [Inventory Service] ────┤
│        │                                     │              │
│        │                                     ▼              │
│        │                            ItemsReserved ─────────┤
│        │                                     │              │
│        ▼                                     ▼              │
│ OrderCompleted ◄────── [Shipping Service] ──┘              │
└─────────────────────────────────────────────────────────────┘

ORCHESTRATION SAGA:
┌─────────────────────────────────────────────────────────────┐
│                 SAGA ORCHESTRATOR                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│        ┌─► [Payment Service]  ──┐                          │
│        │                        │                          │
│ [Orchestrator] ─► [Inventory] ──┼─► Success/Failure        │
│        │                        │                          │
│        └─► [Shipping Service] ──┘                          │
│                                                             │
│ Преимущества:                                               │
│ ├── Централизованная логика                                 │
│ ├── Легче тестировать                                       │
│ └── Лучший контроль                                         │
└─────────────────────────────────────────────────────────────┘
```

#### **Compensating Actions**

```
ПАТТЕРН КОМПЕНСАЦИИ:

Нормальный Flow:                 Компенсирующие действия:
┌─────────────────┐             ┌─────────────────┐
│ CreateOrder     │ ◄────────── │ CancelOrder     │
├─────────────────┤             ├─────────────────┤
│ ReserveItems    │ ◄────────── │ ReleaseItems    │  
├─────────────────┤             ├─────────────────┤
│ ProcessPayment  │ ◄────────── │ RefundPayment   │
├─────────────────┤             ├─────────────────┤
│ ShipOrder       │ ◄────────── │ CancelShipment  │
└─────────────────┘             └─────────────────┘

Правила компенсации:
├── Каждая транзакция должна иметь компенсацию
├── Компенсации выполняются в обратном порядке
├── Компенсации должны быть идемпотентными
└── Учитывать частичные сбои
```

#### **Saga State Management**

```
SAGA STATE MACHINE:

                    PaymentFailed
               ┌─────────────────────┐
               │                     ▼
    Start ──► Payment ──► Inventory ──► Shipping ──► Completed
               │            │           │
               │            │           │ ShippingFailed
               │            │           ▼
               │            │      CompensateInventory
               │            │           │
               │            │ InventoryFailed
               │            ▼           │
               │      CompensatePayment │
               │            │           │
               │            ▼           ▼
               └────────► Failed ◄──────┘

Состояния Saga:
├── STARTED
├── PAYMENT_PROCESSING  
├── PAYMENT_COMPLETED
├── INVENTORY_PROCESSING
├── SHIPPING_PROCESSING
├── COMPLETED
├── COMPENSATING
└── FAILED
```

#### **Практическое задание 4.3**

**Задача:** Реализовать Saga для обработки заказа электронной коммерции

```java
@Component
public class OrderSagaOrchestrator {
    
    @SagaStart
    public void handle(CreateOrderCommand command) {
        // 1. Создать заказ
        commandGateway.send(new ValidateOrderCommand(command.getOrderId()));
    }
    
    @SagaAssociationProperty("orderId")
    public void on(OrderValidatedEvent event) {
        // 2. Резервировать товары
        commandGateway.send(new ReserveItemsCommand(event.getOrderId()));
    }
    
    @SagaAssociationProperty("orderId")  
    public void on(ItemsReservedEvent event) {
        // 3. Обработать платеж
        commandGateway.send(new ProcessPaymentCommand(event.getOrderId()));
    }
    
    @SagaAssociationProperty("orderId")
    public void on(PaymentFailedEvent event) {
        // Компенсация: освободить товары
        commandGateway.send(new ReleaseItemsCommand(event.getOrderId()));
    }
}
```

---

## **ГЛАВА 4.4: МИКРОСЕРВИСНАЯ АРХИТЕКТУРА**
### 📚 Продолжительность: 0.5 недели

#### **Event-Driven Microservices**

```
МИКРОСЕРВИСЫ С MESSAGING:

┌─────────────┐    Events    ┌─────────────┐    Events    ┌─────────────┐
│   User      │ ──────────► │   Order     │ ──────────► │  Payment    │
│  Service    │             │  Service    │             │  Service    │
└─────────────┘             └─────────────┘             └─────────────┘
      │                           │                           │
      │ Domain Events             │ Domain Events             │
      ▼                           ▼                           ▼
┌─────────────┐             ┌─────────────┐             ┌─────────────┐
│   Message   │             │   Message   │             │   Message   │
│   Broker    │ ◄───────────┤   Broker    │ ◄───────────┤   Broker    │
└─────────────┘             └─────────────┘             └─────────────┘

Принципы:
├── Loose coupling через события
├── Autonomous deployment
├── Domain-driven design
└── Event-first thinking
```

#### **Domain Events Design**

```
DOMAIN EVENT STRUCTURE:

┌─────────────────────────────────────────────────────────────┐
│                    DOMAIN EVENT                             │
├─────────────────────────────────────────────────────────────┤
│ EventId: UUID                                               │
│ EventType: "OrderPlaced"                                    │
│ AggregateId: "order-123"                                    │
│ AggregateType: "Order"                                      │
│ Version: 1                                                  │
│ Timestamp: 2025-07-02T10:00:00Z                            │
│ UserId: "user-456"                                          │
│ Data: {                                                     │
│   orderId: "order-123",                                     │
│   customerId: "customer-789",                               │
│   items: [...],                                             │
│   totalAmount: 150.00                                       │
│ }                                                           │
└─────────────────────────────────────────────────────────────┘

Event Categories:
├── Command Events (намерения)
├── Domain Events (факты)
├── Integration Events (между bounded contexts)
└── System Events (технические)
```

#### **Bounded Contexts Integration**

```
INTEGRATION PATTERNS:

1. SHARED KERNEL:
┌─────────────┐     ┌─────────────┐
│  Service A  │────►│ Shared Lib  │◄────┤  Service B  │
└─────────────┘     └─────────────┘     └─────────────┘

2. CUSTOMER/SUPPLIER:
┌─────────────┐     Events     ┌─────────────┐
│ Upstream    │ ──────────────►│ Downstream  │
│ Service     │                │ Service     │
└─────────────┘                └─────────────┘

3. CONFORMIST:
┌─────────────┐     API/Events  ┌─────────────┐
│ Our Service │ ──────────────►│ External    │
│             │                │ Service     │
└─────────────┘                └─────────────┘

4. ANTI-CORRUPTION LAYER:
┌─────────────┐     ┌─────────┐     ┌─────────────┐
│ Our Service │────►│   ACL   │────►│ Legacy      │
│             │     │         │     │ System      │
└─────────────┘     └─────────┘     └─────────────┘
```

---

## **ГЛАВА 4.5: ORDERING И IDEMPOTENCY**
### 📚 Продолжительность: 0.5 недели

#### **Message Ordering Guarantees**

```
ORDERING LEVELS:

1. NO ORDERING:
Producer ──► [M1, M2, M3] ──► Consumer может получить [M3, M1, M2]

2. PARTITION ORDERING:
┌─────────────────────────────────────────┐
│               TOPIC                     │
├─────────────────────────────────────────┤
│ Partition 0: [M1] → [M3] → [M5]        │
│ Partition 1: [M2] → [M4] → [M6]        │  
│ Partition 2: [M7] → [M8] → [M9]        │
└─────────────────────────────────────────┘
   Порядок гарантирован внутри partition

3. GLOBAL ORDERING:
Single Partition: [M1] → [M2] → [M3] → [M4]
                   (медленно, не масштабируется)
```

#### **Partition Key Design**

```
СТРАТЕГИИ ПАРТИЦИРОВАНИЯ:

1. По User ID:
   key = userId → обеспечивает порядок для каждого пользователя

2. По Entity ID:  
   key = orderId → все события заказа в одной партиции

3. По Time Window:
   key = timestamp / timeWindow → временные группы

4. Композитный ключ:
   key = customerId + region → баланс между порядком и распределением

┌─────────────────────────────────────────────────────────────┐
│                 PARTITION ASSIGNMENT                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ hash(partitionKey) % numberOfPartitions = targetPartition  │
│                                                             │
│ Примеры:                                                    │
│ hash("user123") % 3 = 1 → Partition 1                     │
│ hash("user456") % 3 = 0 → Partition 0                     │
│ hash("user789") % 3 = 2 → Partition 2                     │
└─────────────────────────────────────────────────────────────┘
```

#### **Idempotent Consumers**

```
ПРОБЛЕМА DUPLICATE PROCESSING:

┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Producer   │────►│   Broker    │────►│  Consumer   │
└─────────────┘     └─────────────┘     └─────────────┘
                           │                    │
                           │ Network failure    │
                           │ Retry              │
                           ▼                    ▼
                    [Same Message]      [Processed Again]

РЕШЕНИЯ:

1. Idempotency Key:
┌─────────────────────────────────────┐
│ Message:                            │
│ {                                   │
│   "idempotencyKey": "uuid-12345",   │
│   "amount": 100,                    │
│   "operation": "transfer"           │
│ }                                   │
└─────────────────────────────────────┘

2. Database Unique Constraint:
CREATE TABLE processed_messages (
    idempotency_key VARCHAR(255) PRIMARY KEY,
    processed_at TIMESTAMP,
    result JSON
);

3. State-based Idempotency:
if (currentState.allows(operation)) {
    executeOperation();
    updateState();
}
```

#### **Duplicate Detection Strategies**

```
СТРАТЕГИИ ОБНАРУЖЕНИЯ ДУБЛИКАТОВ:

1. MESSAGE DEDUPLICATION:
┌─────────────────────────────────────────────────────────────┐
│                    DEDUPLICATION                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ ┌─────────────┐     ┌─────────────┐     ┌─────────────┐    │
│ │  Message    │────►│    Cache    │────►│  Consumer   │    │
│ │    ID       │     │  (Recent    │     │             │    │
│ │             │     │   IDs)      │     │             │    │
│ └─────────────┘     └─────────────┘     └─────────────┘    │
│                           │                                 │
│                           │ If ID exists → Skip             │
│                           │ If ID new → Process             │
└─────────────────────────────────────────────────────────────┘

2. BLOOM FILTER:
┌─────────────────────────────────────────────────────────────┐
│ Bloom Filter (вероятностная структура):                     │
│                                                             │
│ + Очень быстрая проверка                                    │
│ + Низкое потребление памяти                                 │
│ - Ложноположительные результаты возможны                    │
│ - Ложноотрицательные невозможны                             │
│                                                             │
│ if (bloomFilter.mightContain(messageId)) {                 │
│     // Проверить в основном хранилище                      │
│ } else {                                                    │
│     // Определенно новое сообщение                          │
│ }                                                           │
└─────────────────────────────────────────────────────────────┘

3. TIME-BASED DEDUPLICATION:
┌─────────────────────────────────────────────────────────────┐
│ Sliding Window Approach:                                    │
│                                                             │
│ [Time: 10:00] [Time: 10:01] [Time: 10:02] [Time: 10:03]   │
│      │              │              │              │        │
│      ▼              ▼              ▼              ▼        │
│ Store IDs      Store IDs      Store IDs      Store IDs     │
│                                                             │
│ Cleanup old IDs every minute                               │
│ Keep deduplication window = 5 minutes                      │
└─────────────────────────────────────────────────────────────┘
```

#### **Практическое задание 4.5**

**Задача:** Реализовать идемпотентный обработчик платежей

```java
@Component
public class PaymentProcessor {
    
    @Autowired
    private PaymentRepository paymentRepository;
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    @EventHandler
    public void handle(PaymentRequestedEvent event) {
        String idempotencyKey = event.getIdempotencyKey();
        
        // 1. Проверить, был ли уже обработан
        if (isAlreadyProcessed(idempotencyKey)) {
            log.info("Payment already processed: {}", idempotencyKey);
            return;
        }
        
        // 2. Пометить как обрабатываемый
        markAsProcessing(idempotencyKey);
        
        try {
            // 3. Обработать платеж
            PaymentResult result = processPayment(event);
            
            // 4. Сохранить результат
            saveResult(idempotencyKey, result);
            
        } catch (Exception e) {
            // 5. Очистить блокировку при ошибке
            clearProcessingMark(idempotencyKey);
            throw e;
        }
    }
}
```

---

## **🏆 ИТОГОВОЕ ПРАКТИЧЕСКОЕ ЗАДАНИЕ БЛОКА 4**

### **Комплексный проект: Event-Driven E-Commerce Platform**

**Задача:** Спроектировать и реализовать платформу электронной коммерции с использованием всех изученных паттернов.

#### **Архитектурные требования:**

```
МИКРОСЕРВИСЫ:
├── User Service (управление пользователями)
├── Product Service (каталог товаров)  
├── Order Service (обработка заказов)
├── Payment Service (обработка платежей)
├── Inventory Service (управление складом)
├── Shipping Service (доставка)
├── Notification Service (уведомления)
└── Analytics Service (аналитика)

ПАТТЕРНЫ ДЛЯ РЕАЛИЗАЦИИ:
├── Event Sourcing для Order Service
├── CQRS для Product Service (каталог + поиск)
├── Saga для обработки заказов
├── EIP для маршрутизации уведомлений
└── Idempotency для всех критичных операций
```

#### **События системы:**

```
USER EVENTS:
├── UserRegistered
├── UserProfileUpdated
└── UserDeactivated

PRODUCT EVENTS:  
├── ProductCreated
├── ProductUpdated
├── ProductDiscontinued
└── PriceChanged

ORDER EVENTS:
├── OrderCreated
├── OrderValidated
├── OrderCancelled
├── OrderCompleted
└── OrderShipped

PAYMENT EVENTS:
├── PaymentRequested
├── PaymentProcessed
├── PaymentFailed
└── RefundIssued
```

#### **Ожидаемые результаты:**

```
ТЕХНИЧЕСКАЯ РЕАЛИЗАЦИЯ:
├── Working code на выбранном языке
├── Docker Compose для локального запуска
├── Конфигурация брокера сообщений
├── Миграции для Event Store
└── Unit и Integration тесты

ДОКУМЕНТАЦИЯ:
├── Архитектурная диаграмма
├── Event Storming результаты  
├── API документация
├── Deployment guide
└── Troubleshooting guide

ДЕМОНСТРАЦИЯ:
├── Сценарий полного жизненного цикла заказа
├── Обработка сбоев и компенсации
├── Мониторинг и метрики
└── Performance тестирование
```

---

## **✅ РЕЗУЛЬТАТЫ ИЗУЧЕНИЯ БЛОКА 4**

После завершения этого блока вы будете способны:

### **Технические навыки:**
- ✅ Применять Enterprise Integration Patterns в реальных проектах
- ✅ Проектировать Event Sourced системы с CQRS
- ✅ Реализовывать Saga паттерн для distributed transactions
- ✅ Архитектурировать event-driven микросервисы
- ✅ Обеспечивать ordering и idempotency в распределенных системах

### **Архитектурное мышление:**
- ✅ Понимание trade-offs различных архитектурных решений
- ✅ Способность выбирать подходящие паттерны для конкретных задач
- ✅ Проектирование для отказоустойчивости и масштабируемости
- ✅ Domain-driven подход к дизайну событий

### **Готовность к следующему блоку:**
- ✅ Понимание архитектурных принципов для оптимизации производительности
- ✅ Базис для изучения monitoring и observability
- ✅ Фундамент для enterprise-уровня security требований

---

**📚 Дополнительные материалы для изучения:**
- Martin Fowler: "Patterns of Enterprise Application Architecture"
- Greg Young: "CQRS and Event Sourcing" talks
- Chris Richardson: "Microservices Patterns" book
- Sam Newman: "Building Microservices" book

**🔗 Полезные ресурсы:**
- Event Store documentation
- Axon Framework (Java)
- NEventStore (.NET)
- Eventuate platform
- Saga pattern implementations