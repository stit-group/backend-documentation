# Блок 4: Управление данными в микросервисах

```
⏱️ Продолжительность: 4-5 недель
🎯 Цель: Освоить стратегии работы с данными в распределенной архитектуре
📊 Сложность: ★★★★☆
```

---

## 🎯 Обзор блока

В микросервисной архитектуре управление данными становится одной из самых сложных задач. В отличие от монолитных приложений, где все данные находятся в одной базе данных, микросервисы требуют новых подходов к обеспечению консистентности, транзакций и интеграции данных.

```
┌─────────────────────────────────────────────────────────────┐
│                    ПРОБЛЕМЫ ДАННЫХ                          │
│                                                             │
│  Монолит                    →    Микросервисы               │
│  ┌─────────────┐                 ┌───┐ ┌───┐ ┌───┐         │
│  │ Приложение  │                 │ S1│ │ S2│ │ S3│         │
│  └─────────────┘                 └───┘ └───┘ └───┘         │
│  ┌─────────────┐                 ┌───┐ ┌───┐ ┌───┐         │
│  │ Одна БД     │                 │DB1│ │DB2│ │DB3│         │
│  └─────────────┘                 └───┘ └───┘ └───┘         │
│                                                             │
│  ✅ ACID транзакции             ❌ Distributed transactions  │
│  ✅ Простые JOIN'ы              ❌ Сложная агрегация        │
│  ✅ Консистентность             ❌ Eventual consistency     │
└─────────────────────────────────────────────────────────────┘
```

---

## 📚 Глава 1: Database per Service Pattern

### 🔑 Основные принципы

**Database per Service** - фундаментальный паттерн микросервисной архитектуры, который гласит: каждый микросервис должен владеть своими данными.

```
┌─────────────────────────────────────────────────────────────┐
│                 DATA OWNERSHIP PRINCIPLE                    │
│                                                             │
│  Service A          Service B          Service C            │
│  ┌─────────┐        ┌─────────┐        ┌─────────┐          │
│  │ Orders  │        │ Users   │        │ Catalog │          │
│  │ Service │        │ Service │        │ Service │          │
│  └─────────┘        └─────────┘        └─────────┘          │
│       │                   │                   │             │
│       ▼                   ▼                   ▼             │
│  ┌─────────┐        ┌─────────┐        ┌─────────┐          │
│  │ Orders  │        │ Users   │        │ Product │          │
│  │   DB    │        │   DB    │        │   DB    │          │
│  └─────────┘        └─────────┘        └─────────┘          │
│                                                             │
│  ❌ НЕ РАЗРЕШЕНО: Прямой доступ к чужой БД                  │
│  ✅ РАЗРЕШЕНО: Только через API сервиса                     │
└─────────────────────────────────────────────────────────────┘
```

### 🗄️ Polyglot Persistence

Каждый сервис может использовать наиболее подходящую для его задач базу данных:

```
┌─────────────────────────────────────────────────────────────┐
│                   POLYGLOT PERSISTENCE                      │
│                                                             │
│  User Service              Order Service         Analytics  │
│  ┌───────────┐             ┌───────────┐         ┌────────┐ │
│  │PostgreSQL │             │ MongoDB   │         │ Redis  │ │
│  │           │             │           │         │        │ │
│  │• ACID     │             │• Flexible │         │• Cache │ │
│  │• Relations│             │• Documents│         │• Speed │ │
│  │• Queries  │             │• Scaling  │         │• Memory│ │
│  └───────────┘             └───────────┘         └────────┘ │
│                                                             │
│  Notification              Logging Service       Search     │
│  ┌───────────┐             ┌───────────┐         ┌────────┐ │
│  │ RabbitMQ  │             │Elasticsearch│        │Elastic │ │
│  │           │             │           │         │ Search │ │
│  │• Messages │             │• Log Data │         │• Index │ │
│  │• Queues   │             │• Analysis │         │• Query │ │
│  └───────────┘             └───────────┘         └────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### ⚠️ Антипаттерн: Shared Database

```
┌─────────────────────────────────────────────────────────────┐
│                    ❌ SHARED DATABASE                        │
│                                                             │
│  Service A          Service B          Service C            │
│  ┌─────────┐        ┌─────────┐        ┌─────────┐          │
│  │ Orders  │        │ Users   │        │ Catalog │          │
│  │ Service │        │ Service │        │ Service │          │
│  └─────────┘        └─────────┘        └─────────┘          │
│       │                   │                   │             │
│       └───────────────────┼───────────────────┘             │
│                           ▼                                 │
│                   ┌─────────────┐                           │
│                   │  Shared DB  │                           │
│                   │             │                           │
│                   │ ❌ Coupling │                           │
│                   │ ❌ Scaling  │                           │
│                   │ ❌ Changes  │                           │
│                   └─────────────┘                           │
│                                                             │
│  ПРОБЛЕМЫ:                                                  │
│  • Сильная связанность сервисов                            │
│  • Сложности с независимым развертыванием                   │
│  • Bottleneck при масштабировании                           │
│  • Схема БД становится слишком сложной                     │
└─────────────────────────────────────────────────────────────┘
```

### 💡 Практическое задание

**Задача:** Декомпозиция монолитной БД интернет-магазина

```sql
-- ИСХОДНАЯ МОНОЛИТНАЯ СХЕМА
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(255),
    password_hash VARCHAR(255),
    name VARCHAR(255),
    address TEXT
);

CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    description TEXT,
    price DECIMAL(10,2),
    stock_quantity INT
);

CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT REFERENCES users(id),
    total_amount DECIMAL(10,2),
    status VARCHAR(50),
    created_at TIMESTAMP
);

CREATE TABLE order_items (
    id INT PRIMARY KEY,
    order_id INT REFERENCES orders(id),
    product_id INT REFERENCES products(id),
    quantity INT,
    price DECIMAL(10,2)
);
```

**Решение - разделение по bounded contexts:**

```
┌─────────────────────────────────────────────────────────────┐
│                    ДЕКОМПОЗИЦИЯ БД                          │
│                                                             │
│  User Service DB        Product Service DB    Order Service │
│  ┌─────────────────┐    ┌─────────────────┐   ┌──────────── │
│  │ users           │    │ products        │   │ orders      │
│  │ ├─ id          │    │ ├─ id          │   │ ├─ id       │
│  │ ├─ email       │    │ ├─ name        │   │ ├─ user_id  │
│  │ ├─ password    │    │ ├─ description │   │ ├─ total    │
│  │ ├─ name        │    │ ├─ price       │   │ ├─ status   │
│  │ └─ address     │    │ └─ stock       │   │ └─ created  │
│  └─────────────────┘    └─────────────────┘   │             │
│                                               │ order_items │
│                                               │ ├─ id       │
│                                               │ ├─ order_id │
│                                               │ ├─ prod_id  │
│                                               │ ├─ quantity │
│                                               │ └─ price    │
│                                               └──────────── │
└─────────────────────────────────────────────────────────────┘
```

---

## 📚 Глава 2: Distributed Transactions и альтернативы

### 🤔 Проблема распределенных транзакций

В монолитной архитектуре ACID транзакции решают проблему консистентности:

```
┌─────────────────────────────────────────────────────────────┐
│                    ACID ТРАНЗАКЦИИ                          │
│                                                             │
│  BEGIN TRANSACTION;                                         │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ 1. UPDATE accounts SET balance = balance - 100         │ │
│  │    WHERE user_id = 1;                                  │ │
│  │                                                        │ │
│  │ 2. UPDATE accounts SET balance = balance + 100         │ │
│  │    WHERE user_id = 2;                                  │ │
│  │                                                        │ │
│  │ 3. INSERT INTO transactions (from_user, to_user, ...)  │ │
│  │    VALUES (1, 2, ...);                                 │ │
│  └─────────────────────────────────────────────────────────┘ │
│  COMMIT; -- Все операции выполняются атомарно               │
│                                                             │
│  ✅ Либо все операции выполняются                           │
│  ✅ Либо ни одна не выполняется                             │
└─────────────────────────────────────────────────────────────┘
```

В микросервисной архитектуре это становится сложной проблемой:

```
┌─────────────────────────────────────────────────────────────┐
│                DISTRIBUTED TRANSACTION                      │
│                                                             │
│  Account Service        Payment Service      Audit Service  │
│  ┌─────────────┐        ┌─────────────┐      ┌───────────── │
│  │ UPDATE      │   1    │ INSERT      │  2   │ LOG         │
│  │ balance     │ -----> │ payment     │ ---> │ transaction │
│  │ WHERE...    │        │ record      │      │ WHERE...    │
│  └─────────────┘        └─────────────┘      └───────────── │
│                                                             │
│  ❓ Что если Payment Service недоступен?                    │
│  ❓ Что если Audit Service упал?                            │
│  ❓ Как откатить изменения в Account Service?               │
│                                                             │
│  Two-Phase Commit (2PC) - сложно и ненадежно               │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Phase 1: PREPARE (все готовы?)                         │ │
│  │ Phase 2: COMMIT (все выполняют)                        │ │
│  │                                                        │ │
│  │ Проблемы:                                              │ │
│  │ • Blocking protocol                                    │ │
│  │ • Single point of failure (координатор)               │ │
│  │ • Производительность                                  │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 🔄 CAP теорема на практике

```
┌─────────────────────────────────────────────────────────────┐
│                      CAP ТЕОРЕМА                            │
│                                                             │
│                    Consistency                              │
│                        ▲                                   │
│                       /│\                                  │
│                      / │ \                                 │
│                     /  │  \                                │
│                    /   │   \                               │
│                   /    │    \                              │
│                  /     │     \                             │
│                 /      │      \                            │
│                /       │       \                           │
│               /        │        \                          │
│              /         │         \                         │
│             /          │          \                        │
│    Availability ◄──────┼──────► Partition                  │
│                        │        Tolerance                  │
│                                                             │
│  В распределенной системе можно выбрать только 2 из 3:     │
│                                                             │
│  🔴 CP (Consistency + Partition Tolerance)                  │
│      Система остается консистентной, но может быть         │
│      недоступной при разделении сети                       │
│                                                             │
│  🟡 AP (Availability + Partition Tolerance)                 │
│      Система остается доступной, но может быть             │
│      временно неконсистентной                              │
│                                                             │
│  🔵 CA (Consistency + Availability)                         │
│      Возможно только в отсутствие разделения сети          │
│      (обычно в одном дата-центре)                          │
└─────────────────────────────────────────────────────────────┘
```

### 🎯 BASE vs ACID

```
┌─────────────────────────────────────────────────────────────┐
│                     ACID vs BASE                            │
│                                                             │
│  ACID (Traditional)              BASE (Microservices)       │
│  ┌─────────────────────┐         ┌─────────────────────┐     │
│  │ A - Atomicity       │         │ BA - Basically      │     │
│  │     Все или ничего  │         │      Available      │     │
│  │                     │         │      Система всегда │     │
│  │ C - Consistency     │         │      доступна       │     │
│  │     Все правила     │         │                     │     │
│  │     соблюдены       │         │ S - Soft state      │     │
│  │                     │         │     Состояние может │     │
│  │ I - Isolation       │         │     изменяться      │     │
│  │     Транзакции      │         │                     │     │
│  │     изолированы     │         │ E - Eventual        │     │
│  │                     │         │     consistency     │     │
│  │ D - Durability      │         │     Консистентность │     │
│  │     Данные сохранены│         │     со временем     │     │
│  └─────────────────────┘         └─────────────────────┘     │
│                                                             │
│  Подходит для:                   Подходит для:              │
│  • Банковские системы            • Социальные сети          │
│  • Критичные операции            • E-commerce               │
│  • Монолитные приложения         • Аналитика                │
│                                  • Микросервисы             │
└─────────────────────────────────────────────────────────────┘
```

### 💡 Практическое задание

**Задача:** Анализ требований к консистентности для e-commerce системы

**Сценарий:** Покупка товара в интернет-магазине

```
┌─────────────────────────────────────────────────────────────┐
│                    АНАЛИЗ КОНСИСТЕНТНОСТИ                   │
│                                                             │
│  Операция: Покупка товара                                   │
│                                                             │
│  1. Order Service: Создать заказ                            │
│  2. Inventory Service: Резервировать товар                  │
│  3. Payment Service: Списать деньги                        │
│  4. Shipping Service: Создать задачу на доставку           │
│  5. Notification Service: Отправить уведомления            │
│                                                             │
│  АНАЛИЗ ТРЕБОВАНИЙ:                                         │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Критичная консистентность (Strong):                    │ │
│  │ • Payment ↔ Inventory (деньги списаны = товар зарезерв)│ │
│  │ • Не можем продать больше, чем есть на складе          │ │
│  │                                                        │ │
│  │ Eventual consistency (допустимо):                      │ │
│  │ • Notification Service (можно отправить позже)         │ │
│  │ • Analytics Service (статистика может отставать)      │ │
│  │ • Shipping Service (можно создать задачу с задержкой) │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                             │
│  РЕШЕНИЕ: Hybrid подход                                     │
│  • Критичные операции - через Saga                         │
│  • Некритичные - через асинхронные события                 │
└─────────────────────────────────────────────────────────────┘
```

---

## 📚 Глава 3: Saga Pattern на практике

### 🎭 Что такое Saga Pattern

Saga - это паттерн для управления long-running transactions в распределенных системах, где каждая транзакция состоит из последовательности локальных транзакций.

```
┌─────────────────────────────────────────────────────────────┐
│                      SAGA PATTERN                           │
│                                                             │
│  Обычная транзакция              Saga транзакция            │
│  ┌─────────────────┐             ┌─────────────────────────┐ │
│  │ BEGIN           │             │ Step 1: Local TX        │ │
│  │ ├─ Operation 1  │             │ ├─ Success → Step 2     │ │
│  │ ├─ Operation 2  │    vs       │ ├─ Fail → Compensate   │ │
│  │ ├─ Operation 3  │             │                        │ │
│  │ COMMIT/ROLLBACK │             │ Step 2: Local TX        │ │
│  └─────────────────┘             │ ├─ Success → Step 3     │ │
│                                  │ ├─ Fail → Compensate    │ │
│  ✅ Атомарность                   │ 1 + 2                  │ │
│  ❌ Не работает                   │                        │ │
│     в микросервисах               │ Step 3: Local TX        │ │
│                                  │ ├─ Success → Complete   │ │
│                                  │ ├─ Fail → Compensate    │ │
│                                  │ 1 + 2 + 3              │ │
│                                  └─────────────────────────┘ │
│                                                             │
│                  🔄 Компенсирующие действия                 │
│                     вместо ROLLBACK                         │
└─────────────────────────────────────────────────────────────┘
```

### 🎼 Choreography-based Saga

Сервисы координируют себя сами через события:

```
┌─────────────────────────────────────────────────────────────┐
│                  CHOREOGRAPHY SAGA                          │
│                                                             │
│  Order Service    Payment Service    Inventory    Shipping  │
│  ┌───────────┐    ┌─────────────┐    ┌─────────┐  ┌───────┐ │
│  │1. Create  │    │             │    │         │  │       │ │
│  │   Order   │    │             │    │         │  │       │ │
│  │           │    │             │    │         │  │       │ │
│  │     ▼     │    │             │    │         │  │       │ │
│  │2. Publish │────┼────────────▶│    │         │  │       │ │
│  │OrderCreated    │             │    │         │  │       │ │
│  └───────────┘    │3. Process   │    │         │  │       │ │
│                   │   Payment   │    │         │  │       │ │
│                   │      ▼      │    │         │  │       │ │
│                   │4. Publish   │────┼────────▶│  │       │ │
│                   │PaymentSuccess    │         │  │       │ │
│                   └─────────────┘    │5.Reserve│  │       │ │
│                                      │ Inventory│  │       │ │
│                                      │    ▼     │  │       │ │
│                                      │6.Publish │──┼──────▶│ │
│                                      │ItemReserved │       │ │
│                                      └─────────┘  │7.Ship │ │
│                                                   └───────┘ │
│                                                             │
│  ✅ Простота                                                │
│  ✅ Низкая связанность                                      │
│  ❌ Сложно отслеживать flow                                 │
│  ❌ Циклические зависимости                                 │
└─────────────────────────────────────────────────────────────┘
```

### 🎯 Orchestration-based Saga

Центральный координатор управляет процессом:

```
┌─────────────────────────────────────────────────────────────┐
│                  ORCHESTRATION SAGA                         │
│                                                             │
│                    Saga Orchestrator                        │
│                    ┌─────────────────┐                      │
│                    │                 │                      │
│     ┌──────────────┼──── 1. Start ───┼──────────────┐       │
│     │              │     Saga        │              │       │
│     ▼              └─────────────────┘              ▼       │
│  Order Service                                 Payment      │
│  ┌───────────┐                               Service        │
│  │2. Create  │◄────── 3. Payment ──────────┐ ┌─────────┐   │
│  │   Order   │        Success              │ │4.Process│   │
│  └───────────┘                             │ │ Payment │   │
│                                            │ └─────────┘   │
│                                            │               │
│                ┌─────────────────┐         │               │
│                │ 5. Continue     │◄────────┘               │
│                │    Saga         │                         │
│                └─────────────────┘                         │
│                         │                                  │
│               ┌─────────┼─────────┐                        │
│               ▼                   ▼                        │
│        Inventory Service     Shipping Service              │
│        ┌─────────────┐       ┌─────────────┐               │
│        │6. Reserve   │       │8. Create    │               │
│        │   Items     │       │   Shipment  │               │
│        └─────────────┘       └─────────────┘               │
│                                                             │
│  ✅ Centralized control                                     │
│  ✅ Easy to monitor                                         │
│  ✅ Clear transaction boundaries                            │
│  ❌ Single point of failure                                 │
│  ❌ Higher coupling                                         │
└─────────────────────────────────────────────────────────────┘
```

### 🔄 Compensation Actions

Пример компенсирующих действий для заказа:

```
┌─────────────────────────────────────────────────────────────┐
│                  COMPENSATION ACTIONS                       │
│                                                             │
│  SUCCESS FLOW:                                              │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ 1. Create Order    → Order ID: #12345                  │ │
│  │ 2. Reserve Items   → 5 units of Product A reserved     │ │
│  │ 3. Process Payment → $100 charged to card *1234        │ │
│  │ 4. Ship Items      → Tracking #ABC123                  │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                             │
│  FAILURE SCENARIO (Payment fails):                         │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ 1. Create Order    → ✅ Order ID: #12345               │ │
│  │ 2. Reserve Items   → ✅ 5 units reserved               │ │
│  │ 3. Process Payment → ❌ Card declined                  │ │
│  │                                                        │ │
│  │ COMPENSATION FLOW:                                     │ │
│  │ 3. Cancel Payment  → ✅ No action needed              │ │
│  │ 2. Release Items   → ✅ 5 units back to inventory     │ │
│  │ 1. Cancel Order    → ✅ Order status = CANCELLED      │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                             │
│  COMPENSATION RULES:                                        │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ • Idempotent: можно выполнить несколько раз            │ │
│  │ • Semantic compensation: не просто undo                │ │
│  │ • Business logic aware: учитывает бизнес-правила       │ │
│  │                                                        │ │
│  │ Пример semantic compensation:                          │ │
│  │ Вместо: DELETE FROM accounts WHERE...                  │ │
│  │ Делаем: INSERT INTO accounts (type='REFUND')...        │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 💡 Практическое задание

**Задача:** Реализация Saga для системы бронирования отеля

**Сценарий:** Пользователь бронирует отель с автомобилем

```python
# Orchestration-based Saga implementation

class HotelBookingSaga:
    def __init__(self):
        self.state = "STARTED"
        self.booking_id = None
        self.hotel_reservation_id = None
        self.car_reservation_id = None
        self.payment_id = None
    
    async def execute(self, booking_request):
        try:
            # Step 1: Create booking
            self.booking_id = await self.booking_service.create_booking(
                booking_request
            )
            self.state = "BOOKING_CREATED"
            
            # Step 2: Reserve hotel room
            self.hotel_reservation_id = await self.hotel_service.reserve_room(
                booking_request.hotel_id,
                booking_request.check_in,
                booking_request.check_out
            )
            self.state = "HOTEL_RESERVED"
            
            # Step 3: Reserve car (if requested)
            if booking_request.car_required:
                self.car_reservation_id = await self.car_service.reserve_car(
                    booking_request.car_type,
                    booking_request.pickup_date
                )
                self.state = "CAR_RESERVED"
            
            # Step 4: Process payment
            self.payment_id = await self.payment_service.charge_card(
                booking_request.card_details,
                booking_request.total_amount
            )
            self.state = "PAYMENT_PROCESSED"
            
            # Step 5: Confirm all reservations
            await self.hotel_service.confirm_reservation(
                self.hotel_reservation_id
            )
            if self.car_reservation_id:
                await self.car_service.confirm_reservation(
                    self.car_reservation_id
                )
            
            self.state = "COMPLETED"
            return {"status": "success", "booking_id": self.booking_id}
            
        except Exception as e:
            await self.compensate()
            raise
    
    async def compensate(self):
        """Выполняем компенсирующие действия в обратном порядке"""
        if self.state == "PAYMENT_PROCESSED":
            await self.payment_service.refund(self.payment_id)
        
        if self.state in ["PAYMENT_PROCESSED", "CAR_RESERVED"]:
            if self.car_reservation_id:
                await self.car_service.cancel_reservation(
                    self.car_reservation_id
                )
        
        if self.state in ["PAYMENT_PROCESSED", "CAR_RESERVED", "HOTEL_RESERVED"]:
            if self.hotel_reservation_id:
                await self.hotel_service.cancel_reservation(
                    self.hotel_reservation_id
                )
        
        if self.booking_id:
            await self.booking_service.cancel_booking(self.booking_id)
        
        self.state = "COMPENSATED"
```

**Тестовые сценарии:**

```
┌─────────────────────────────────────────────────────────────┐
│                    ТЕСТОВЫЕ СЦЕНАРИИ                        │
│                                                             │
│  Сценарий 1: Happy Path                                     │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ ✅ Создание бронирования                               │ │
│  │ ✅ Резервирование отеля                                │ │
│  │ ✅ Резервирование автомобиля                           │ │
│  │ ✅ Успешная оплата                                     │ │
│  │ ✅ Подтверждение всех резерваций                       │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                             │
│  Сценарий 2: Отель недоступен                              │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ ✅ Создание бронирования                               │ │
│  │ ❌ Резервирование отеля (нет свободных номеров)        │ │
│  │ 🔄 Компенсация: отмена бронирования                   │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                             │
│  Сценарий 3: Ошибка оплаты                                 │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ ✅ Создание бронирования                               │ │
│  │ ✅ Резервирование отеля                                │ │
│  │ ✅ Резервирование автомобиля                           │ │
│  │ ❌ Ошибка оплаты (недостаточно средств)               │ │
│  │ 🔄 Компенсация: отмена автомобиля                     │ │
│  │ 🔄 Компенсация: отмена отеля                          │ │
│  │ 🔄 Компенсация: отмена бронирования                   │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## 📚 Глава 4: Event Sourcing и CQRS

### 🎬 Event Sourcing концепция

Event Sourcing - паттерн хранения данных в виде последовательности событий вместо текущего состояния.

```
┌─────────────────────────────────────────────────────────────┐
│                    TRADITIONAL vs EVENT SOURCING            │
│                                                             │
│  Traditional Storage          Event Sourcing                │
│  ┌─────────────────┐         ┌─────────────────────────────┐ │
│  │ Current State   │         │ Event Store                 │ │
│  │                 │         │                             │ │
│  │ Account: {      │         │ 1. AccountCreated           │ │
│  │   id: 123,      │   vs    │    {id: 123, balance: 0}    │ │
│  │   balance: 250, │         │                             │ │
│  │   owner: "John" │         │ 2. MoneyDeposited           │ │
│  │ }               │         │    {amount: 100}            │ │
│  └─────────────────┘         │                             │ │
│                              │ 3. MoneyDeposited           │ │
│  ❌ Потеря истории            │    {amount: 200}            │ │
│  ❌ Сложно восстановить       │                             │ │
│      прошлое состояние       │ 4. MoneyWithdrawn           │ │
│  ❌ Audit trail отдельно      │    {amount: 50}             │ │
│                              │                             │ │
│                              │ Current State = replay      │ │
│                              │ events 1-4 = balance: 250   │ │
│                              └─────────────────────────────┘ │
│                                                             │
│                              ✅ Полная история              │
│                              ✅ Audit trail встроен         │
│                              ✅ Temporal queries            │
│                              ✅ Легко дебажить              │
└─────────────────────────────────────────────────────────────┘
```

### 📊 CQRS (Command Query Responsibility Segregation)

CQRS разделяет операции чтения и записи:

```
┌─────────────────────────────────────────────────────────────┐
│                        CQRS PATTERN                         │
│                                                             │
│                      Client Application                     │
│                           │                                 │
│               ┌───────────┼───────────┐                     │
│               ▼                       ▼                     │
│        ┌─────────────┐          ┌─────────────┐             │
│        │  Commands   │          │   Queries   │             │
│        │             │          │             │             │
│        │ • Create    │          │ • Get user  │             │
│        │ • Update    │          │ • List      │             │
│        │ • Delete    │          │ • Search    │             │
│        └─────────────┘          └─────────────┘             │
│               │                       │                     │
│               ▼                       ▼                     │
│        ┌─────────────┐          ┌─────────────┐             │
│        │Write Model  │          │ Read Model  │             │
│        │             │          │             │             │
│        │Event Store  │          │ Projections │             │
│        │┌───────────┐│    sync  │┌───────────┐│             │
│        ││ Events    ││ ────────▶││ Views     ││             │
│        ││ Stream    ││          ││ Denorm    ││             │
│        │└───────────┘│          ││ Tables    ││             │
│        └─────────────┘          │└───────────┘│             │
│                                 └─────────────┘             │
│                                                             │
│  Write Side:                    Read Side:                  │
│  • Optimized для consistency    • Optimized для queries     │
│  • Complex business logic       • Simple, fast reads       │
│  • Normalized                   • Denormalized             │
│  • ACID transactions            • Eventually consistent     │
└─────────────────────────────────┘─────────────────────────────┘
```

### 🗄️ Event Store Implementation

Пример структуры Event Store:

```sql
-- Event Store Schema
CREATE TABLE event_store (
    event_id UUID PRIMARY KEY,
    aggregate_id VARCHAR(255) NOT NULL,
    aggregate_type VARCHAR(100) NOT NULL,
    event_type VARCHAR(100) NOT NULL,
    event_data JSONB NOT NULL,
    event_version INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    
    -- Optimistic concurrency control
    UNIQUE(aggregate_id, event_version)
);

-- Snapshots для производительности
CREATE TABLE snapshots (
    aggregate_id VARCHAR(255) PRIMARY KEY,
    aggregate_type VARCHAR(100) NOT NULL,
    snapshot_data JSONB NOT NULL,
    version INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Индексы для быстрого поиска
CREATE INDEX idx_events_aggregate ON event_store(aggregate_id);
CREATE INDEX idx_events_type ON event_store(aggregate_type);
CREATE INDEX idx_events_created ON event_store(created_at);
```

### 📈 Projections и Read Models

```
┌─────────────────────────────────────────────────────────────┐
│                      PROJECTIONS                            │
│                                                             │
│  Event Stream                  Projections                  │
│  ┌─────────────────┐          ┌─────────────────────────────┐ │
│  │ UserCreated     │          │ Users View                  │ │
│  │ {id, name, ...} │ ────────▶│ ┌─────────────────────────┐ │ │
│  ├─────────────────┤          │ │ SELECT * FROM users     │ │ │
│  │ EmailChanged    │ ────────▶│ │ WHERE status='active'   │ │ │
│  │ {id, newEmail}  │          │ └─────────────────────────┘ │ │
│  ├─────────────────┤          └─────────────────────────────┘ │
│  │ UserDeactivated │ ────────▶┌─────────────────────────────┐ │
│  │ {id, reason}    │          │ User Statistics             │ │
│  ├─────────────────┤          │ ┌─────────────────────────┐ │ │
│  │ OrderPlaced     │ ────────▶│ │ Total Users: 1,250      │ │ │
│  │ {userId, ...}   │          │ │ Active: 1,180           │ │ │
│  ├─────────────────┤          │ │ Deactivated: 70         │ │ │
│  │ PaymentMade     │          │ │ Orders this month: 450  │ │ │
│  │ {userId, ...}   │          │ └─────────────────────────┘ │ │
│  └─────────────────┘          └─────────────────────────────┘ │
│                                                             │
│  ТИПЫ PROJECTIONS:                                          │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ 1. Simple Views                                         │ │
│  │    • Прямая проекция событий в таблицы                  │ │
│  │    • Users, Products, Orders                            │ │
│  │                                                         │ │
│  │ 2. Aggregated Views                                     │ │
│  │    • Статистика и метрики                               │ │
│  │    • Daily sales, User counts                           │ │
│  │                                                         │ │
│  │ 3. Complex Joins                                        │ │
│  │    • Денормализованные данные                           │ │
│  │    • Order with Customer and Product details            │ │
│  │                                                         │ │
│  │ 4. Search Indexes                                       │ │
│  │    • Elasticsearch, Solr                                │ │
│  │    • Full-text search, faceting                         │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 🔄 Event Versioning

События должны развиваться со временем:

```
┌─────────────────────────────────────────────────────────────┐
│                    EVENT VERSIONING                         │
│                                                             │
│  Version 1 (Initial)           Version 2 (Added fields)     │
│  ┌─────────────────────┐       ┌─────────────────────────┐   │
│  │ UserCreated v1      │       │ UserCreated v2          │   │
│  │ {                   │  →    │ {                       │   │
│  │   "id": "123",      │       │   "id": "123",          │   │
│  │   "name": "John",   │       │   "name": "John",       │   │
│  │   "email": "..."    │       │   "email": "...",       │   │
│  │ }                   │       │   "age": 30,            │   │
│  └─────────────────────┘       │   "preferences": {...}  │   │
│                                │ }                       │   │
│                                └─────────────────────────┘   │
│                                                             │
│  СТРАТЕГИИ ВЕРСИОНИРОВАНИЯ:                                 │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ 1. Weak Schema (рекомендуется)                         │ │
│  │    • Добавление полей не ломает совместимость           │ │
│  │    • Optional fields с default values                  │ │
│  │    • Graceful degradation                              │ │
│  │                                                        │ │
│  │ 2. Event Upcasting                                     │ │
│  │    • При чтении старых событий конвертируем их          │ │
│  │    • V1 → V2 migration on-the-fly                     │ │
│  │                                                        │ │
│  │ 3. Multiple Event Handlers                             │ │
│  │    • Разные обработчики для разных версий              │ │
│  │    • Постепенный переход                               │ │
│  │                                                        │ │
│  │ 4. Copy and Replace                                    │ │
│  │    • Новые типы событий для breaking changes           │ │ │
│  │    • UserCreated → UserCreatedV2                       │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 💡 Практическое задание

**Задача:** Реализация Event Store для системы управления заказами

```python
# Event Store Implementation

from dataclasses import dataclass
from typing import List, Optional, Any
from datetime import datetime
import json
import uuid

@dataclass
class Event:
    event_id: str
    aggregate_id: str
    aggregate_type: str
    event_type: str
    event_data: dict
    event_version: int
    created_at: datetime

class EventStore:
    def __init__(self):
        self.events: List[Event] = []
        self.snapshots: dict = {}
    
    def append_events(self, aggregate_id: str, aggregate_type: str, 
                     events: List[dict], expected_version: int):
        """Append events to stream with optimistic concurrency control"""
        
        # Check current version
        current_version = self.get_current_version(aggregate_id)
        if current_version != expected_version:
            raise ConcurrencyException(
                f"Expected version {expected_version}, "
                f"but current is {current_version}"
            )
        
        # Append new events
        for i, event_data in enumerate(events):
            event = Event(
                event_id=str(uuid.uuid4()),
                aggregate_id=aggregate_id,
                aggregate_type=aggregate_type,
                event_type=event_data['type'],
                event_data=event_data['data'],
                event_version=current_version + i + 1,
                created_at=datetime.now()
            )
            self.events.append(event)
    
    def get_events(self, aggregate_id: str, 
                   from_version: int = 0) -> List[Event]:
        """Get events for aggregate from specific version"""
        return [
            event for event in self.events
            if event.aggregate_id == aggregate_id 
            and event.event_version > from_version
        ]
    
    def get_current_version(self, aggregate_id: str) -> int:
        """Get current version of aggregate"""
        events = [
            event for event in self.events
            if event.aggregate_id == aggregate_id
        ]
        return max([event.event_version for event in events], default=0)

# Order Aggregate Example
class Order:
    def __init__(self, order_id: str):
        self.order_id = order_id
        self.items = []
        self.status = "CREATED"
        self.total_amount = 0.0
        self.version = 0
        self.uncommitted_events = []
    
    def add_item(self, product_id: str, quantity: int, price: float):
        """Add item to order"""
        if self.status != "CREATED":
            raise Exception("Cannot add items to processed order")
        
        event_data = {
            'type': 'ItemAdded',
            'data': {
                'product_id': product_id,
                'quantity': quantity,
                'price': price
            }
        }
        self.apply_event(event_data)
        self.uncommitted_events.append(event_data)
    
    def confirm_order(self):
        """Confirm order for processing"""
        if self.status != "CREATED":
            raise Exception("Order already processed")
        
        if not self.items:
            raise Exception("Cannot confirm empty order")
        
        event_data = {
            'type': 'OrderConfirmed',
            'data': {
                'total_amount': self.total_amount
            }
        }
        self.apply_event(event_data)
        self.uncommitted_events.append(event_data)
    
    def apply_event(self, event_data: dict):
        """Apply event to aggregate state"""
        event_type = event_data['type']
        data = event_data['data']
        
        if event_type == 'ItemAdded':
            self.items.append({
                'product_id': data['product_id'],
                'quantity': data['quantity'],
                'price': data['price']
            })
            self.total_amount += data['quantity'] * data['price']
        
        elif event_type == 'OrderConfirmed':
            self.status = "CONFIRMED"
        
        self.version += 1
    
    @classmethod
    def from_events(cls, order_id: str, events: List[Event]):
        """Rebuild aggregate from events"""
        order = cls(order_id)
        for event in events:
            event_data = {
                'type': event.event_type,
                'data': event.event_data
            }
            order.apply_event(event_data)
        return order

# Read Model Projections
class OrderProjection:
    def __init__(self):
        self.orders_view = {}  # Simple in-memory view
        self.order_statistics = {
            'total_orders': 0,
            'confirmed_orders': 0,
            'total_revenue': 0.0
        }
    
    def handle_event(self, event: Event):
        """Handle events and update read models"""
        if event.event_type == 'ItemAdded':
            # Update orders view
            if event.aggregate_id not in self.orders_view:
                self.orders_view[event.aggregate_id] = {
                    'order_id': event.aggregate_id,
                    'items': [],
                    'total_amount': 0.0,
                    'status': 'CREATED'
                }
            
            self.orders_view[event.aggregate_id]['items'].append(
                event.event_data
            )
            self.orders_view[event.aggregate_id]['total_amount'] += (
                event.event_data['quantity'] * event.event_data['price']
            )
        
        elif event.event_type == 'OrderConfirmed':
            # Update order status
            self.orders_view[event.aggregate_id]['status'] = 'CONFIRMED'
            
            # Update statistics
            self.order_statistics['confirmed_orders'] += 1
            self.order_statistics['total_revenue'] += (
                event.event_data['total_amount']
            )
    
    def get_order(self, order_id: str) -> Optional[dict]:
        """Get order view"""
        return self.orders_view.get(order_id)
    
    def get_statistics(self) -> dict:
        """Get order statistics"""
        return self.order_statistics

# Usage Example
def example_usage():
    event_store = EventStore()
    projection = OrderProjection()
    
    # Create and modify order
    order = Order("order-123")
    order.add_item("product-1", 2, 50.0)
    order.add_item("product-2", 1, 30.0)
    order.confirm_order()
    
    # Save events
    event_store.append_events(
        aggregate_id=order.order_id,
        aggregate_type="Order",
        events=order.uncommitted_events,
        expected_version=0
    )
    
    # Update projections
    events = event_store.get_events(order.order_id)
    for event in events:
        projection.handle_event(event)
    
    # Query read model
    order_view = projection.get_order("order-123")
    print(f"Order view: {order_view}")
    
    stats = projection.get_statistics()
    print(f"Statistics: {stats}")
    
    # Rebuild aggregate from events (for command side)
    rebuilt_order = Order.from_events("order-123", events)
    print(f"Rebuilt order status: {rebuilt_order.status}")
```

**Результат выполнения:**

```
┌─────────────────────────────────────────────────────────────┐
│                    РЕЗУЛЬТАТ ВЫПОЛНЕНИЯ                     │
│                                                             │
│  Event Store содержит:                                      │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ 1. ItemAdded: {product_id: "product-1", quantity: 2}   │ │
│  │ 2. ItemAdded: {product_id: "product-2", quantity: 1}   │ │
│  │ 3. OrderConfirmed: {total_amount: 130.0}               │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                             │
│  Order View (Read Model):                                   │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ {                                                       │ │
│  │   "order_id": "order-123",                              │ │
│  │   "items": [                                            │ │
│  │     {"product_id": "product-1", "quantity": 2},        │ │
│  │     {"product_id": "product-2", "quantity": 1}         │ │
│  │   ],                                                    │ │
│  │   "total_amount": 130.0,                                │ │
│  │   "status": "CONFIRMED"                                 │ │
│  │ }                                                       │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                             │
│  Statistics:                                                │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ • Total orders: 1                                       │ │
│  │ • Confirmed orders: 1                                   │ │
│  │ • Total revenue: $130.00                                │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## ✅ Итоговое задание блока

### 🎯 Комплексное задание: Система электронной коммерции

Создайте систему управления данными для интернет-магазина, которая включает:

1. **Database per Service pattern**
2. **Saga для обработки заказов**
3. **Event Sourcing для критичных операций**
4. **CQRS для оптимизации чтения**

**Требования:**

```
┌─────────────────────────────────────────────────────────────┐
│                    АРХИТЕКТУРА СИСТЕМЫ                      │
│                                                             │
│  Services:                                                  │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ 1. User Service                                         │ │
│  │    • PostgreSQL для профилей пользователей              │ │
│  │    • CRUD операции                                      │ │
│  │                                                         │ │
│  │ 2. Catalog Service                                      │ │
│  │    • MongoDB для продуктов и категорий                  │ │
│  │    • Elasticsearch для поиска                           │ │
│  │                                                         │ │
│  │ 3. Inventory Service                                    │ │
│  │    • Redis для быстрого доступа к остаткам              │ │
│  │    • PostgreSQL для audit trail                        │ │
│  │                                                         │ │
│  │ 4. Order Service                                        │ │
│  │    • Event Store для заказов                            │ │
│  │    • CQRS pattern                                       │ │
│  │                                                         │ │
│  │ 5. Payment Service                                      │ │
│  │    • PostgreSQL для транзакций                          │ │
│  │    • Event Sourcing для audit                           │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                             │
│  Saga Flows:                                                │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Order Processing Saga:                                  │ │
│  │ 1. Create Order → 2. Reserve Inventory →               │ │
│  │ 3. Process Payment → 4. Confirm Order                  │ │
│  │                                                         │ │
│  │ Compensation Actions:                                   │ │
│  │ • Release inventory if payment fails                    │ │
│  │ • Refund payment if shipping fails                      │ │
│  │ • Cancel order if any step fails                       │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 📋 Критерии оценки

```
┌─────────────────────────────────────────────────────────────┐
│                    КРИТЕРИИ ОЦЕНКИ                          │
│                                                             │
│  ✅ Базовый уровень (50 баллов):                            │
│  • Каждый сервис имеет свою БД                              │
│  • Реализован простой Saga pattern                          │
│  • Нет прямого доступа между БД сервисов                    │
│                                                             │
│  ✅ Продвинутый уровень (75 баллов):                        │
│  • Event Sourcing для Order Service                         │
│  • CQRS с read models                                       │
│  • Proper compensation actions                              │
│  • Optimistic concurrency control                          │
│                                                             │
│  ✅ Экспертный уровень (100 баллов):                        │
│  • Event versioning strategy                                │
│  • Multiple projections                                     │
│  • Performance optimization                                 │
│  • Monitoring и observability                               │
│  • Complete error handling                                  │
└─────────────────────────────────────────────────────────────┘
```

### 🚀 Следующие шаги

После завершения этого блока вы должны:

- ✅ Понимать принципы управления данными в микросервисах
- ✅ Уметь проектировать Saga для complex workflows
- ✅ Применять Event Sourcing и CQRS где это оправдано
- ✅ Обеспечивать eventual consistency

**Следующий блок:** Безопасность и production-ready решения