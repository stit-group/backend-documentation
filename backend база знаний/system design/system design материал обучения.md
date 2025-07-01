# System Design: От нуля до эксперта

> **Философия курса**: Понимание принципов важнее знания инструментов. Инструменты меняются, принципы остаются.

---

## **Фаза 1: Основы (4-6 недель)**

### **Неделя 1-2: Базовые концепции**

#### Что такое System Design?
**System Design** — это искусство проектирования крупномасштабных распределенных систем, способных обслуживать миллионы пользователей.

**Ключевая разница:**
- **Algorithm Design**: Как решить задачу эффективно?
- **System Design**: Как решить задачу для миллионов пользователей надежно?

#### Масштабируемость vs Производительность

```
Производительность: Как быстро система отвечает на один запрос?
Масштабируемость: Как система ведет себя при росте нагрузки?

1 пользователь  →  быстро ✓
1000 пользователей → быстро ✓  (хорошая производительность)
1M пользователей  →  медленно ✗

vs

1 пользователь     →  быстро ✓
1000 пользователей →  быстро ✓  
1M пользователей   →  быстро ✓  (хорошая масштабируемость)
```

#### Типы масштабирования

**Вертикальное масштабирование (Scale Up)**
```
Сервер 1.0: 2 CPU, 4GB RAM
        ↓ добавляем ресурсы
Сервер 2.0: 8 CPU, 32GB RAM
```
- ✅ Проще
- ❌ Есть предел
- ❌ Единая точка отказа

**Горизонтальное масштабирование (Scale Out)**
```
Сервер 1 ←→ Load Balancer ←→ Сервер 2
                ↕
             Сервер 3
```
- ✅ Неограниченное масштабирование
- ✅ Отказоустойчивость
- ❌ Сложность

#### Ключевые метрики

**Latency** — время отклика на запрос
```python
start = time.now()
response = process_request()
latency = time.now() - start  # миллисекунды
```

**Throughput** — количество запросов в секунду
```python
requests_per_second = total_requests / time_period
```

**Availability** — доступность системы
```
Availability = Uptime / (Uptime + Downtime)
99.9% = 43 минуты простоя в месяц
99.99% = 4 минуты простоя в месяц
```

---

### **Неделя 3-4: Архитектурные паттерны**

#### Монолитная архитектура

```
┌─────────────────────────────────┐
│         МОНОЛИТ                 │
│  ┌─────┐ ┌─────┐ ┌──────────┐   │
│  │ UI  │ │ API │ │ Business │   │
│  └─────┘ └─────┘ └──────────┘   │
│           ┌─────────────┐       │
│           │  Database   │       │
│           └─────────────┘       │
└─────────────────────────────────┘
```

**Плюсы:**
- Простота разработки и развертывания
- Легкое тестирование
- Высокая производительность внутренних вызовов

**Минусы:**
- Трудности с масштабированием команд
- Технологические ограничения
- Единая точка отказа

#### Микросервисная архитектура

```
┌──────────┐    ┌──────────┐    ┌──────────┐
│ Service A│    │ Service B│    │ Service C│
│    DB    │    │    DB    │    │    DB    │
└──────────┘    └──────────┘    └──────────┘
      ↕              ↕              ↕
         ┌─────────────────────┐
         │   API Gateway       │
         └─────────────────────┘
```

**Когда использовать:**
- Большие команды разработки
- Разные технологические требования
- Независимое масштабирование компонентов

**Пример декомпозиции:**
```python
# Монолит
class ECommerceApp:
    def create_user(self): pass
    def create_order(self): pass
    def process_payment(self): pass
    def manage_inventory(self): pass

# Микросервисы
class UserService:
    def create_user(self): pass

class OrderService:
    def create_order(self): pass

class PaymentService:
    def process_payment(self): pass
```

#### Event-Driven Architecture

```
Service A  →  [Event Bus]  →  Service B
   ↓              ↑              ↓
Event      →  [Storage]   ←  Subscribe
```

**Концепция:**
- Сервисы общаются через события
- Слабая связанность
- Асинхронная обработка

```python
# Публикация события
event_bus.publish('user_created', {'user_id': 123})

# Подписка на событие
@event_bus.subscribe('user_created')
def send_welcome_email(data):
    email_service.send(data['user_id'])
```

---

### **Неделя 5-6: Сетевые основы**

#### TCP vs UDP

**TCP — надежная доставка**
```
Client ──── SYN ────→ Server
Client ←─── ACK ───── Server
Client ──── DATA ───→ Server
Client ←─── ACK ───── Server
```
- ✅ Гарантированная доставка
- ✅ Правильный порядок
- ❌ Больше задержка

**UDP — быстрая доставка**
```
Client ──── DATA ───→ Server
Client ──── DATA ───→ Server  (может потеряться)
```
- ✅ Минимальная задержка
- ❌ Нет гарантий доставки

**Применение:**
- TCP: веб-сайты, API, файлы
- UDP: видео-стрим, игры, DNS

#### Load Balancing

**Round Robin**
```
Request 1 → Server A
Request 2 → Server B  
Request 3 → Server C
Request 4 → Server A  (цикл)
```

**Weighted Round Robin**
```python
servers = [
    {'name': 'A', 'weight': 3},
    {'name': 'B', 'weight': 2},
    {'name': 'C', 'weight': 1}
]
# A получит 50% трафика, B - 33%, C - 17%
```

**Least Connections**
```
Server A: 10 активных соединений
Server B: 5 активных соединений   ← новый запрос
Server C: 15 активных соединений
```

---

## **Фаза 2: Хранение данных (6-8 недель)**

### **Неделя 7-9: Типы баз данных**

#### ACID свойства

**Atomicity** — все или ничего
```python
# Перевод денег - должен быть атомарным
def transfer_money(from_account, to_account, amount):
    # Либо обе операции успешны, либо обе откатываются
    withdraw(from_account, amount)  
    deposit(to_account, amount)
```

**Consistency** — данные остаются валидными
```python
# Баланс не может быть отрицательным
constraint: account.balance >= 0
```

**Isolation** — транзакции не влияют друг на друга
```
Transaction A: читает balance = 100
Transaction B: читает balance = 100
Transaction A: устанавливает balance = 90
Transaction B: устанавливает balance = 110
Результат: balance = 110 (не 100!)
```

**Durability** — данные сохраняются навсегда

#### SQL vs NoSQL

**Когда использовать SQL:**
```
✅ Сложные запросы с JOIN
✅ ACID транзакции
✅ Структурированные данные
✅ Отчеты и аналитика

Пример: Банковская система
```

**Когда использовать NoSQL:**
```
✅ Простые запросы
✅ Горизонтальное масштабирование
✅ Неструктурированные данные
✅ Высокая скорость записи

Пример: Социальная сеть
```

#### CAP теорема

```
    Consistency
        ▲
       /|\
      / | \
     /  |  \
    /   |   \
   ▼    |    ▼
Availability ← → Partition Tolerance
```

**Можно выбрать только 2 из 3:**

- **CP системы**: MongoDB, Redis
  - Жертвуют доступностью ради консистентности
- **AP системы**: Cassandra, DynamoDB  
  - Жертвуют консистентностью ради доступности
- **CA системы**: PostgreSQL, MySQL
  - Работают только без сетевых разделений

---

### **Неделя 10-12: Масштабирование БД**

#### Database Sharding

**Горизонтальное разделение данных**

```
Users 1-1000   → Shard A
Users 1001-2000 → Shard B  
Users 2001-3000 → Shard C
```

**Стратегии шардинга:**

**Range-based**
```python
def get_shard(user_id):
    if user_id <= 1000: return "shard_a"
    elif user_id <= 2000: return "shard_b"
    else: return "shard_c"
```

**Hash-based**
```python
def get_shard(user_id):
    return f"shard_{hash(user_id) % 3}"
```

**Directory-based**
```python
# Отдельная таблица маппинга
shard_map = {
    "user_123": "shard_a",
    "user_456": "shard_b"
}
```

#### Master-Slave репликация

```
    ┌─────────┐
    │ Master  │ (запись)
    └─────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌─────────┐ ┌─────────┐
│ Slave 1 │ │ Slave 2 │ (чтение)
└─────────┘ └─────────┘
```

**Преимущества:**
- Распределение нагрузки на чтение
- Отказоустойчивость
- Географическое распределение

**Проблемы:**
- Задержка репликации
- Единая точка отказа (master)

---

### **Неделя 13-14: Специализированные хранилища**

#### In-Memory Databases

**Redis как кэш:**
```python
# Установка значения с TTL
redis.setex("user:123", 3600, user_data)

# Попытка получить из кэша
cached = redis.get("user:123")
if cached:
    return cached
else:
    data = database.get_user(123)
    redis.setex("user:123", 3600, data)
    return data
```

#### Search Engines

**Elasticsearch для полнотекстового поиска:**
```
Обычная БД: WHERE title LIKE '%design%'  (медленно)
             ↓
Search Engine: полнотекстовый индекс (быстро)

"system design" → [doc1, doc5, doc12]
```

#### Time-Series Databases

**Для метрик и мониторинга:**
```
timestamp, metric_name, value
2024-01-01 10:00:00, cpu_usage, 45.2
2024-01-01 10:01:00, cpu_usage, 47.1
2024-01-01 10:02:00, cpu_usage, 44.8
```

**Оптимизированы для:**
- Быстрая запись по времени
- Агрегация по временным интервалам
- Сжатие старых данных

---

## **Фаза 3: Распределенные системы (8-10 недель)**

### **Неделя 15-17: Основы распределенных систем**

#### Distributed System Challenges

**Сетевые задержки и разделения**
```
Service A ──────X──────→ Service B
         (сеть недоступна)

Что делать Service A?
- Ждать? (блокировка)
- Вернуть ошибку? (плохой UX)
- Использовать кэш? (устаревшие данные)
```

**Частичные отказы**
```
Request → Service A ✓ → Service B ✗ → Service C ?

Проблема: система находится в неопределенном состоянии
```

#### Consensus алгоритмы

**Raft — простой алгоритм консенсуса**
```
Leader Election:
Node A: голосует за себя
Node B: голосует за A
Node C: голосует за A
→ Node A становится лидером

Log Replication:
Leader A → запись 1 → Follower B, C
Leader A ← подтверждение ← Follower B, C
→ запись считается committed
```

#### Logical Clocks

**Проблема физических часов:**
```
Server A: 10:00:01 - событие X
Server B: 10:00:00 - событие Y

Какое событие произошло раньше?
Физические часы не синхронизированы!
```

**Lamport Timestamps:**
```python
class LogicalClock:
    def __init__(self):
        self.time = 0
    
    def tick(self):
        self.time += 1
        return self.time
    
    def update(self, received_time):
        self.time = max(self.time, received_time) + 1
```

---

### **Неделя 18-20: Надежность и отказоустойчивость**

#### Circuit Breaker Pattern

**Защита от каскадных отказов**

```
┌─────────────────┐
│ Service A       │
│  ┌──────────┐   │    ┌─────────────┐
│  │ Circuit  │───────→│ Service B   │
│  │ Breaker  │   │    │ (падает)    │
│  └──────────┘   │    └─────────────┘
└─────────────────┘
```

**Состояния:**
```python
class CircuitBreaker:
    CLOSED = "closed"      # нормальная работа
    OPEN = "open"          # блокировка запросов
    HALF_OPEN = "half_open" # тестирование

    def call(self, func):
        if self.state == self.OPEN:
            raise Exception("Circuit breaker is open")
        
        try:
            result = func()
            self.on_success()
            return result
        except Exception:
            self.on_failure()
            raise
```

#### Retry стратегии

**Exponential Backoff**
```python
import time
import random

def retry_with_backoff(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except Exception:
            if attempt == max_retries - 1:
                raise
            
            # Увеличиваем задержку экспоненциально
            delay = (2 ** attempt) + random.uniform(0, 1)
            time.sleep(delay)
```

**Jitter для избежания thundering herd:**
```
Retry 1: 1 сек + random(0-1)
Retry 2: 2 сек + random(0-1)  
Retry 3: 4 сек + random(0-1)
```

#### Health Checks

**Типы проверок:**
```python
class HealthCheck:
    def shallow_check(self):
        # Быстрая проверка - отвечает ли сервис
        return {"status": "ok"}
    
    def deep_check(self):
        # Глубокая проверка - доступна ли БД
        try:
            database.ping()
            return {"status": "ok", "database": "connected"}
        except:
            return {"status": "error", "database": "disconnected"}
```

**Load Balancer Integration:**
```
Load Balancer → GET /health → Service
                ← 200 OK ←   (здоровый сервис)
                ← 503 Error ← (больной сервис)
```

---

### **Неделя 21-23: Консистентность данных**

#### Saga Pattern

**Distributed Transactions без 2PC**

```
Order Service → Payment Service → Inventory Service
     ↓               ↓                   ↓
  Create           Charge             Reserve
   Order           Card              Products
```

**Если что-то пошло не так:**
```
Order Service ← Payment Service ← Inventory Service
     ↑               ↑                   ↑
   Cancel          Refund            Unreserve
   Order           Card              Products
```

**Пример кода:**
```python
class OrderSaga:
    def execute(self, order_data):
        try:
            order = self.create_order(order_data)
            payment = self.process_payment(order)
            inventory = self.reserve_inventory(order)
            return {"status": "success"}
        except PaymentError:
            self.cancel_order(order)
            raise
        except InventoryError:
            self.refund_payment(payment)
            self.cancel_order(order)
            raise
```

#### Event Sourcing

**Сохранение событий вместо состояния**

```
Традиционный подход:
User { id: 123, name: "John", email: "john@example.com" }

Event Sourcing:
Event 1: UserCreated(id=123, name="John")
Event 2: EmailChanged(id=123, email="john@example.com")
Event 3: NameChanged(id=123, name="John Doe")

Текущее состояние = применение всех событий
```

**Преимущества:**
- Полная история изменений
- Возможность воспроизведения состояния
- Аудит из коробки

#### CQRS

**Разделение чтения и записи**

```
Command Side (запись):
Client → Command → Write Model → Events

Query Side (чтение):
Events → Read Model ← Query ← Client
```

```python
# Command (изменение данных)
class CreateUserCommand:
    def execute(self, data):
        user = User.create(data)
        events.publish('user_created', user)

# Query (чтение данных)  
class UserQueryService:
    def get_user(self, user_id):
        return read_model.get_user(user_id)
```

---

### **Неделя 24-25: Асинхронная обработка**

#### Message Queues

**Очереди для надежной доставки**

```
Producer → [Queue] → Consumer

Producer может "упасть" - сообщение сохранено
Consumer обрабатывает в своем темпе
```

**Паттерны обработки:**
```python
# At least once delivery
def process_message(message):
    try:
        business_logic(message)
        message.ack()  # подтверждение после обработки
    except:
        message.nack()  # сообщение вернется в очередь

# Idempotent processing
def process_payment(message):
    payment_id = message.payment_id
    if not is_already_processed(payment_id):
        charge_card(message.amount)
        mark_as_processed(payment_id)
```

#### Pub/Sub Pattern

**Один ко многим**

```
Publisher → [Topic] → Subscriber A
                  → Subscriber B  
                  → Subscriber C
```

**Event Broadcasting:**
```python
# Публикация события
event_bus.publish('user_registered', {
    'user_id': 123,
    'email': 'user@example.com'
})

# Подписчики
@subscribe('user_registered')
def send_welcome_email(event):
    email_service.send_welcome(event['email'])

@subscribe('user_registered') 
def create_user_profile(event):
    profile_service.create(event['user_id'])
```

#### Dead Letter Queues

**Обработка проблемных сообщений**

```
Message → [Main Queue] → Consumer
                   ↓ (failed 3 times)
                [Dead Letter Queue]
                   ↓
               Manual Review
```

---

## **Фаза 4: Производительность и масштабирование (6-8 недель)**

### **Неделя 26-28: Кэширование**

#### Уровни кэширования

```
Browser Cache
       ↓
   CDN Cache  
       ↓
Reverse Proxy Cache
       ↓
Application Cache
       ↓
Database Cache
       ↓
   Database
```

#### Cache Patterns

**Cache-Aside (Lazy Loading)**
```python
def get_user(user_id):
    # Проверяем кэш
    user = cache.get(f"user:{user_id}")
    if user:
        return user
    
    # Если нет в кэше - идем в БД
    user = database.get_user(user_id)
    cache.set(f"user:{user_id}", user, ttl=3600)
    return user
```

**Write-Through**
```python
def update_user(user_id, data):
    # Обновляем БД и кэш одновременно
    database.update_user(user_id, data)
    cache.set(f"user:{user_id}", data, ttl=3600)
```

**Write-Behind (Write-Back)**
```python
def update_user(user_id, data):
    # Обновляем только кэш
    cache.set(f"user:{user_id}", data, ttl=3600)
    # БД обновится асинхронно позже
    async_queue.add_task('update_db', user_id, data)
```

#### Cache Invalidation

**Самая сложная проблема в computer science:**

```python
# Time-based expiration
cache.set("key", value, ttl=3600)  # истекает через час

# Event-based invalidation
@listen('user_updated')
def invalidate_user_cache(event):
    cache.delete(f"user:{event.user_id}")

# Cache tags
cache.set("user:123", data, tags=["user", "profile"])
cache.invalidate_tags(["user"])  # удалит все с тегом user
```

---

### **Неделя 29-31: Оптимизация производительности**

#### Database Optimization

**Query Optimization**
```sql
-- Медленно
SELECT * FROM users WHERE created_at > '2024-01-01';

-- Быстро (с индексом на created_at)
CREATE INDEX idx_users_created_at ON users(created_at);
SELECT id, name FROM users WHERE created_at > '2024-01-01';
```

**Connection Pooling**
```python
# Плохо - создаем соединение для каждого запроса
def get_user(user_id):
    conn = database.connect()  # медленно!
    result = conn.query(f"SELECT * FROM users WHERE id = {user_id}")
    conn.close()
    return result

# Хорошо - переиспользуем соединения
connection_pool = ConnectionPool(size=10)

def get_user(user_id):
    with connection_pool.get_connection() as conn:
        return conn.query(f"SELECT * FROM users WHERE id = {user_id}")
```

#### Batching

**Группировка операций**
```python
# Плохо - N+1 проблема
def get_user_posts(user_ids):
    result = []
    for user_id in user_ids:
        posts = database.query(f"SELECT * FROM posts WHERE user_id = {user_id}")
        result.append(posts)
    return result

# Хорошо - batch запрос
def get_user_posts(user_ids):
    return database.query(f"SELECT * FROM posts WHERE user_id IN ({','.join(user_ids)})")
```

#### Rate Limiting

**Защита от злоупотреблений**

**Token Bucket Algorithm:**
```python
class TokenBucket:
    def __init__(self, capacity, refill_rate):
        self.capacity = capacity
        self.tokens = capacity
        self.refill_rate = refill_rate
        self.last_refill = time.time()
    
    def consume(self, tokens=1):
        self._refill()
        if self.tokens >= tokens:
            self.tokens -= tokens
            return True
        return False
    
    def _refill(self):
        now = time.time()
        elapsed = now - self.last_refill
        self.tokens = min(self.capacity, 
                         self.tokens + elapsed * self.refill_rate)
        self.last_refill = now
```

**Sliding Window:**
```python
def rate_limit(user_id, limit=100, window=3600):
    key = f"rate_limit:{user_id}"
    current_requests = redis.zcount(key, time.time() - window, time.time())
    
    if current_requests >= limit:
        return False
    
    redis.zadd(key, {str(uuid.uuid4()): time.time()})
    redis.expire(key, window)
    return True
```

---

### **Неделя 32-33: Мониторинг и наблюдаемость**

#### Три столпа наблюдаемости

**Metrics — что происходит**
```python
# Счетчики
request_count.increment()
error_count.increment()

# Гистограммы  
response_time.observe(0.245)

# Гауges
active_connections.set(42)
```

**Logs — контекст событий**
```python
logger.info("User login attempt", extra={
    "user_id": 123,
    "ip_address": "192.168.1.1",
    "user_agent": "Mozilla/5.0...",
    "success": True
})
```

**Traces — путь запроса**
```
GET /api/users/123
├── auth_service.validate_token (10ms)
├── user_service.get_user (25ms)
│   └── database.query (20ms)
└── response_serialization (5ms)
Total: 40ms
```

#### SLI/SLO/SLA

**SLI (Service Level Indicator)** — что измеряем
```
Availability = successful_requests / total_requests
Latency = request_duration < 100ms  
Error Rate = error_requests / total_requests
```

**SLO (Service Level Objective)** — цели
```
Availability: 99.9% за месяц
Latency: 95% запросов < 100ms
Error Rate: < 0.1% запросов
```

**SLA (Service Level Agreement)** — контракт
```
Если availability < 99.9%, то:
- Возврат 10% от месячной платы
- Публичный отчет о причинах
```

#### Error Budgets

**Сколько ошибок мы можем себе позволить**
```python
# SLO: 99.9% availability
# Error budget: 0.1% = 43 минуты простоя в месяц

class ErrorBudget:
    def __init__(self, slo_percentage):
        self.slo = slo_percentage
        self.budget = 1 - slo_percentage
    
    def consume(self, downtime_minutes):
        monthly_budget = 30 * 24 * 60 * self.budget  # 43 минуты
        return downtime_minutes / monthly_budget
```

---

## **Фаза 5: Продвинутые темы (6-8 недель)**

### **Неделя 34-36: Безопасность**

#### Authentication vs Authorization

```
Authentication: "Кто ты?"
User → presents credentials → System
     ← identity confirmed  ←

Authorization: "Что ты можешь делать?"
Authenticated User → requests action → System
                  ← permission check ←
```

#### JWT Tokens

**Stateless аутентификация**
```
Header: {"alg": "HS256", "typ": "JWT"}
Payload: {"user_id": 123, "exp": 1640995200}
Signature: HMACSHA256(base64(header) + "." + base64(payload), secret)
```

```python
def verify_token(token):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
        if payload['exp'] < time.time():
            raise Exception("Token expired")
        return payload['user_id']
    except:
        raise Exception("Invalid token")
```

**Преимущества JWT:**
- Stateless (не нужно хранить сессии)
- Можно передавать данные в токене
- Подходит для микросервисов

**Недостатки:**
- Нельзя отозвать до истечения
- Размер больше чем session ID

#### API Security

**Rate Limiting**
```python
@rate_limit(requests_per_minute=60)
def api_endpoint():
    pass
```

**Input Validation**
```python
def create_user(data):
    # Валидация обязательна!
    if not isinstance(data.get('email'), str):
        raise ValidationError("Invalid email")
    
    if len(data.get('password', '')) < 8:
        raise ValidationError("Password too short")
```

**SQL Injection Prevention**
```python
# Плохо
query = f"SELECT * FROM users WHERE id = {user_id}"

# Хорошо
query = "SELECT * FROM users WHERE id = %s"
cursor.execute(query, (user_id,))
```

#### Encryption

**At Rest** — шифрование хранимых данных
```python
# Шифрование чувствительных полей
encrypted_ssn = encrypt(user.ssn, encryption_key)
database.save({'user_id': user.id, 'ssn': encrypted_ssn})
```

**In Transit** — HTTPS для всех API
```
Client ←──── HTTPS/TLS ────→ Server
      (зашифрованное соединение)
```

---

### **Неделя 37-39: Развертывание и операции**

#### Blue-Green Deployments

**Zero-downtime развертывание**

```
Load Balancer
     ↓
┌─────────────┐
│Blue (старая)│ ← 100% трафика
│ version 1.0 │
└─────────────┘

┌─────────────┐  
│Green (новая)│ ← разворачиваем версию 2.0
│ version 2.0 │
└─────────────┘
```

**После тестирования:**
```
Load Balancer
     ↓
┌─────────────┐
│Blue (старая)│ ← 0% трафика  
│ version 1.0 │
└─────────────┘

┌─────────────┐
│Green (новая)│ ← 100% трафика
│ version 2.0 │
└─────────────┘
```

#### Canary Releases

**Постепенное развертывание**

```python
def route_traffic():
    if user_in_canary_group():
        return "new_version"  # 5% пользователей
    else:
        return "old_version"  # 95% пользователей
```

**Мониторинг канарейки:**
```python
if canary_error_rate > threshold:
    rollback_canary()
else:
    increase_canary_traffic()
```

#### Feature Flags

**Управление функциональностью**
```python
def new_checkout_flow():
    if feature_flag("new_checkout", user.id):
        return new_checkout_implementation()
    else:
        return old_checkout_implementation()
```

**Градуальный rollout:**
```python
# Включаем для 10% пользователей
feature_config = {
    "new_checkout": {
        "enabled": True,
        "percentage": 10,
        "user_groups": ["beta_testers"]
    }
}
```

---

### **Неделя 40-41: Специализированные системы**

#### Real-time Systems

**WebSocket для мгновенной связи**
```
Client ←──── persistent connection ────→ Server
        (bidirectional communication)
```

```python
# Чат-система
class ChatServer:
    def __init__(self):
        self.connections = {}
    
    def on_message(self, user_id, message):
        # Broadcast всем подключенным пользователям
        for conn in self.connections.values():
            conn.send(f"{user_id}: {message}")
```

**Server-Sent Events для уведомлений**
```python
def notification_stream(user_id):
    while True:
        notifications = get_user_notifications(user_id)
        for notification in notifications:
            yield f"data: {json.dumps(notification)}\n\n"
        time.sleep(1)
```

#### Stream Processing

**Обработка потоков данных в реальном времени**

```
Data Source → Stream Processor → Output
   (logs)    → (filter, map,   → (alerts)
              aggregate)
```

**Концепции:**
```python
# Windowing - группировка по времени
def count_events_per_minute(stream):
    return stream.window(duration=60).count()

# Stateful processing
def detect_anomaly(stream):
    return stream.scan(lambda acc, event: 
                      update_model(acc, event))
```

#### Batch Processing

**ETL пайплайны**
```
Extract → Transform → Load

Data Sources → Processing → Data Warehouse
(databases,   (clean,      (for analytics)
 APIs, files)  aggregate)
```

```python
def daily_user_stats():
    # Extract
    user_events = read_from_log_files()
    
    # Transform  
    aggregated = group_by_user(user_events)
    stats = calculate_metrics(aggregated)
    
    # Load
    write_to_warehouse(stats)
```

---

## **Фаза 6: Практическое применение (4-6 недель)**

### **Неделя 42-44: Системные интервью**

#### Подход к решению

**1. Requirements Clarification (5 мин)**
```
- Функциональные требования: что система должна делать?
- Нефункциональные: RPS, latency, availability
- Масштаб: сколько пользователей, данных?
```

**2. Capacity Estimation (5 мин)**
```python
# Пример: Twitter
daily_active_users = 200_000_000
tweets_per_day = 400_000_000  # 2 tweets/user
tweets_per_second = tweets_per_day / (24 * 3600) = ~4600 TPS

# Storage
avg_tweet_size = 280 bytes
daily_storage = tweets_per_day * avg_tweet_size = 112 GB/day
yearly_storage = 112 * 365 = ~40 TB/year
```

**3. System Design (20 мин)**
```
High-level architecture:
Client → Load Balancer → API Gateway → Services → Databases
```

**4. Deep Dive (15 мин)**
- Выберите компоненты для детального разбора
- Обсудите trade-offs
- Покажите знание паттернов

#### Trade-offs Analysis

**Consistency vs Availability**
```
Банковская система: Выбираем Consistency
- Лучше показать ошибку, чем неправильный баланс

Социальная сеть: Выбираем Availability  
- Пользователь может увидеть старый пост
```

**Latency vs Throughput**
```
Gaming: Низкий latency важнее
- 50ms vs 100ms критично для игры

Batch processing: Высокий throughput важнее
- Можем ждать, но хотим обработать много данных
```

---

### **Неделя 45-47: Изучение реальных систем**

#### Netflix Architecture

**Проблема:** Масштабирование видео-стриминга

**Решение:**
```
CDN (глобальное кэширование)
    ↓
Microservices (сотни сервисов)
    ↓
Chaos Engineering (тестирование отказов)
```

**Ключевые паттерны:**
- Circuit Breaker для отказоустойчивости
- Eventual Consistency для глобального каталога
- A/B тестирование алгоритмов рекомендаций

#### Uber Architecture

**Проблема:** Real-time геолокация и матчинг

**Решение:**
```
Geospatial Database (H3 hexagons)
    ↓
Real-time Stream Processing  
    ↓
Event-driven Architecture
```

**Innovations:**
- Geohashing для быстрого поиска водителей
- Supply-demand prediction models
- Dynamic pricing algorithms

#### WhatsApp Architecture

**Проблема:** Миллиарды сообщений в реальном времени

**Решение:**
```
Erlang/OTP (actor model)
    ↓
FreeBSD optimization
    ↓ 
Minimalist approach
```

**Lessons:**
- Простота лучше сложности
- Правильный выбор технологии критичен
- Оптимизация на уровне ОС может дать огромный выигрыш

---

## **Ключевые принципы System Design**

### **1. Масштабируемость**
```
Думайте о росте с первого дня
Horizontal scaling > Vertical scaling
Stateless services проще масштабировать
```

### **2. Надежность**
```
Assume failures will happen
Graceful degradation
Circuit breakers everywhere
```

### **3. Простота**
```
Start simple, add complexity when needed
Microservices - не silver bullet
Монолит может быть правильным выбором
```

### **4. Trade-offs**
```
Нет идеальных решений
Каждое решение имеет цену
Понимайте свои ограничения
```

### **5. Измеряйте**
```
You can't improve what you don't measure
SLIs/SLOs/SLAs
Real User Monitoring
```

---

## **Следующие шаги**

### **Практика**
1. Решайте system design задачи еженедельно
2. Анализируйте архитектуры популярных сервисов
3. Читайте engineering блоги крупных компаний

### **Углубление**
1. Изучите конкретные технологии (Kafka, Redis, etc.)
2. Практикуйтесь на реальных проектах
3. Участвуйте в архитектурных обсуждениях в команде

### **Ресурсы**
- High Scalability blog
- AWS/GCP/Azure whitepapers  
- Company engineering blogs
- System design interview books

**Помните:** System Design - это не про технологии, это про принципы и trade-offs. Технологии меняются, принципы остаются.