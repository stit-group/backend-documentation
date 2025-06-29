# Техническое собеседование Python Senior Developer

## Структуры данных и типы Python

### 1. Изменяемые и неизменяемые типы
**Вопрос:** Объясните разницу между mutable и immutable типами в Python. Что произойдет с этим кодом?
```python
def modify_list(lst=[]):
    lst.append(1)
    return lst

a = modify_list()
b = modify_list()
print(a, b)  # Что выведет?
```

<details>
<summary>💡 Подсказка</summary>
Рассмотрите механизм создания объектов по умолчанию в функциях
</details>

**Углубление:**
- Как работает интернирование строк и чисел?
- Почему `id()` может возвращать одинаковые значения для разных объектов?
- Расскажите про copy vs deepcopy

---

### 2. Словари и их внутреннее устройство
**Вопрос:** Как работают словари в Python под капотом? Какая временная сложность операций?
```python
# Что быстрее и почему?
# Вариант 1
if key in dict:
    value = dict[key]

# Вариант 2  
try:
    value = dict[key]
except KeyError:
    value = None
```

<details>
<summary>💡 Подсказка</summary>
Подумайте про hash tables, collision resolution, и Python 3.7+ ordering guarantee
</details>

**Углубление:**
- Что такое hash collision и как Python их обрабатывает?
- Почему с Python 3.7 словари стали упорядоченными?
- Когда использовать dict vs OrderedDict vs ChainMap?

---

### 3. Генераторы и итераторы
**Вопрос:** В чем разница между этими реализациями? Когда что использовать?
```python
# Вариант 1
def numbers_list(n):
    return [i for i in range(n)]

# Вариант 2
def numbers_generator(n):
    for i in range(n):
        yield i

# Вариант 3
def numbers_iterator(n):
    return iter(range(n))
```

<details>
<summary>💡 Подсказка</summary>
Рассмотрите memory usage, lazy evaluation, и протокол итератора
</details>

**Углубление:**
- Что такое yield from и когда его использовать?
- Как работает send() у генераторов?
- Объясните разницу между __iter__ и __next__

---

## Многопоточность и конкурентность

### 4. GIL и его влияние
**Вопрос:** Объясните, что такое GIL. Когда многопоточность в Python эффективна, а когда нет?
```python
import threading
import time

counter = 0

def increment():
    global counter
    for _ in range(100000):
        counter += 1

# Будет ли результат всегда 200000?
threads = [threading.Thread(target=increment) for _ in range(2)]
for t in threads:
    t.start()
for t in threads:
    t.join()
print(counter)
```

<details>
<summary>💡 Подсказка</summary>
GIL защищает только интерпретатор, но не ваш код от race conditions
</details>

**Углубление:**
- Как работают CPU-bound vs I/O-bound задачи с GIL?
- Когда использовать threading vs multiprocessing vs asyncio?
- Что такое thread-local storage?

---

### 5. Race Conditions и синхронизация
**Вопрос:** Найдите и исправьте race condition в этом коде:
```python
import threading

class Counter:
    def __init__(self):
        self.value = 0
        
    def increment(self):
        temp = self.value
        temp += 1
        self.value = temp
        
    def get_value(self):
        return self.value

# Как сделать thread-safe?
```

<details>
<summary>💡 Подсказка</summary>
Рассмотрите Lock, RLock, Semaphore, Condition, Event
</details>

**Углубление:**
- В чем разница между Lock и RLock?
- Что такое deadlock и как его избежать?
- Когда использовать thread.local()?

---

## Асинхронность

### 6. Event Loop и корутины
**Вопрос:** Объясните разницу между этими подходами:
```python
import asyncio

# Вариант 1
async def fetch_data_sequential():
    result1 = await fetch_from_api1()
    result2 = await fetch_from_api2()
    return result1, result2

# Вариант 2
async def fetch_data_concurrent():
    task1 = asyncio.create_task(fetch_from_api1())
    task2 = asyncio.create_task(fetch_from_api2())
    return await task1, await task2

# Вариант 3
async def fetch_data_gather():
    return await asyncio.gather(
        fetch_from_api1(), 
        fetch_from_api2()
    )
```

<details>
<summary>💡 Подсказка</summary>
Подумайте про cooperative multitasking и event loop scheduling
</details>

**Углубление:**
- Как работает event loop под капотом?
- В чем разница между await, asyncio.create_task() и asyncio.gather()?
- Что происходит при смешивании sync и async кода?

---

### 7. Контекстные менеджеры в asyncio
**Вопрос:** Реализуйте асинхронный контекстный менеджер для пула соединений:
```python
class AsyncConnectionPool:
    def __init__(self, max_connections=5):
        self.max_connections = max_connections
        # Ваша реализация
        
    async def __aenter__(self):
        # Ваша реализация
        pass
        
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        # Ваша реализация
        pass
```

<details>
<summary>💡 Подсказка</summary>
Используйте asyncio.Semaphore для ограничения количества соединений
</details>

**Углубление:**
- Как обрабатывать исключения в async контекстных менеджерах?
- В чем разница между contextlib.asynccontextmanager и ручной реализацией?

---

## Архитектура и проектирование

### 8. Dependency Injection
**Вопрос:** Рефакторите этот код, применив принципы DI:
```python
class EmailService:
    def send_email(self, to, subject, body):
        # Отправка через SMTP
        pass

class UserService:
    def __init__(self):
        self.email_service = EmailService()  # Жесткая зависимость
        
    def register_user(self, email, password):
        # Логика регистрации
        self.email_service.send_email(email, "Welcome", "Hello!")
```

<details>
<summary>💡 Подсказка</summary>
Используйте абстракции (Protocol/ABC) и инъекцию зависимостей через конструктор
</details>

**Углубление:**
- Какие есть способы реализации DI в Python?
- Как использовать typing.Protocol для определения интерфейсов?
- Сравните DI контейнеры: dependency-injector, inject, punq

---

### 9. Чистая архитектура
**Вопрос:** Спроектируйте архитектуру для системы управления заказами с учетом принципов Clean Architecture:

```
Entities: Order, Product, User
Use Cases: CreateOrder, UpdateOrderStatus, CalculateTotal
```

Покажите структуру папок и основные интерфейсы.

<details>
<summary>💡 Подсказка</summary>
Подумайте про layers: entities, use_cases, interface_adapters, frameworks
</details>

**Углубление:**
- Как обеспечить изоляцию слоев?
- Где размещать бизнес-правила?
- Как тестировать каждый слой изолированно?

---

### 10. Repository Pattern
**Вопрос:** Реализуйте Repository pattern с поддержкой разных источников данных:
```python
from abc import ABC, abstractmethod
from typing import List, Optional

class User:
    def __init__(self, id: int, name: str, email: str):
        self.id = id
        self.name = name
        self.email = email

class UserRepository(ABC):
    # Ваша реализация интерфейса
    pass

class DatabaseUserRepository(UserRepository):
    # Реализация для БД
    pass

class InMemoryUserRepository(UserRepository):
    # Реализация для тестов
    pass
```

<details>
<summary>💡 Подсказка</summary>
Подумайте про CRUD операции и Unit of Work pattern
</details>

**Углубление:**
- Как реализовать Unit of Work?
- Где размещать сложные запросы с join'ами?
- Как обрабатывать транзакции в repository?

---

## Продвинутые вопросы по Python

### 11. Метаклассы и дескрипторы
**Вопрос:** Объясните, как работает этот код:
```python
class ValidatedField:
    def __init__(self, validator):
        self.validator = validator
        
    def __set_name__(self, owner, name):
        self.name = name
        
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name)
        
    def __set__(self, obj, value):
        if not self.validator(value):
            raise ValueError(f"Invalid value for {self.name}")
        obj.__dict__[self.name] = value

class Person:
    name = ValidatedField(lambda x: isinstance(x, str) and len(x) > 0)
    age = ValidatedField(lambda x: isinstance(x, int) and x >= 0)
```

<details>
<summary>💡 Подсказка</summary>
Это дескриптор. Изучите протокол дескрипторов и __set_name__
</details>

**Углубление:**
- В чем разница между data и non-data дескрипторами?
- Как работает method resolution order (MRO)?
- Когда использовать метаклассы vs дескрипторы vs декораторы?

---

### 12. Управление памятью
**Вопрос:** Объясните, что происходит с памятью в этом коде:
```python
import gc

class Node:
    def __init__(self, value):
        self.value = value
        self.children = []
        self.parent = None
        
    def add_child(self, child):
        child.parent = self
        self.children.append(child)

# Создаем циклическую ссылку
root = Node("root")
child = Node("child")
root.add_child(child)

del root, child
# Очистится ли память?
```

<details>
<summary>💡 Подсказка</summary>
Подумайте про reference counting, циклические ссылки и garbage collector
</details>

**Углубление:**
- Как работает сборщик мусора в Python?
- Что такое weak references и когда их использовать?
- Как профилировать использование памяти?

---

## Системный дизайн и масштабирование

### 13. Кэширование
**Вопрос:** Спроектируйте многоуровневое кэширование для API:
```python
# Требования:
# - L1: In-memory cache (быстрый, небольшой)
# - L2: Redis cache (средний, больший)  
# - L3: Database (медленный, полный)
# - Cache invalidation по TTL и по ключам
# - Метрики hit/miss ratio
```

<details>
<summary>💡 Подсказка</summary>
Используйте паттерн Cache-Aside с fallback стратегией
</details>

**Углубление:**
- Как обновлять cache при изменении данных?
- Cache stampede problem и способы решения
- Distributed caching и consistency проблемы

---

### 14. Event-Driven Architecture
**Вопрос:** Спроектируйте событийную архитектуру для e-commerce:
```
События: OrderCreated, PaymentProcessed, OrderShipped, OrderCancelled
Сервисы: OrderService, PaymentService, InventoryService, NotificationService
```

Как обеспечить eventual consistency и обработку ошибок?

<details>
<summary>💡 Подсказка</summary>
Рассмотрите Event Sourcing, CQRS, Saga pattern
</details>

**Углубление:**
- Как обрабатывать failed events?
- Ordering гарантии в distributed системах
- Event versioning и backward compatibility

---

### 15. Мониторинг и наблюдаемость
**Вопрос:** Реализуйте систему трассировки запросов:
```python
# Требования:
# - Каждый запрос должен иметь trace_id
# - Логи должны содержать trace_id
# - Метрики по времени выполнения
# - Передача trace_id между сервисами
```

Покажите реализацию с использованием OpenTelemetry.

<details>
<summary>💡 Подсказка</summary>
Используйте контекстные переменные и middleware
</details>

**Углубление:**
- Как семплировать трейсы в high-load системах?
- Correlation ID vs Trace ID
- Distributed tracing в микросервисах

---

## Производительность и оптимизация

### 16. Профилирование и оптимизация
**Вопрос:** Найдите узкие места в этом коде и оптимизируйте:
```python
def process_users(users):
    result = []
    for user in users:
        if is_active_user(user):  # Медленная проверка в БД
            profile = get_user_profile(user.id)  # N+1 problem
            if profile.score > 100:
                result.append({
                    'id': user.id,
                    'name': user.name,
                    'score': profile.score
                })
    return result
```

<details>
<summary>💡 Подсказка</summary>
Рассмотрите batch операции, кэширование, предварительную загрузку
</details>

**Углубление:**
- Какие инструменты использовать для профилирования?
- Memory profiling vs CPU profiling
- Как оптимизировать database queries?

---

### 17. Асинхронная обработка
**Вопрос:** Спроектируйте систему обработки файлов:
```
Требования:
- Загрузка файлов до 1GB
- Асинхронная обработка 
- Progress tracking
- Error handling и retry
- Rate limiting
```

<details>
<summary>💡 Подсказка</summary>
Используйте background tasks, message queues, chunked processing
</details>

**Углубление:**
- Celery vs dramatiq vs RQ - когда что выбирать?
- Как обрабатывать long-running tasks?
- Circuit breaker pattern для внешних API

---

## Безопасность

### 18. Аутентификация и авторизация
**Вопрос:** Реализуйте RBAC (Role-Based Access Control):
```python
# Требования:
# - Роли: admin, manager, user
# - Ресурсы: orders, products, users
# - Действия: create, read, update, delete
# - Иерархия ролей: admin > manager > user
```

<details>
<summary>💡 Подсказка</summary>
Используйте декораторы, permission classes, middleware
</details>

**Углубление:**
- RBAC vs ABAC (Attribute-Based Access Control)
- JWT tokens: статeless vs stateful сессии
- OAuth 2.0 / OpenID Connect интеграция

---

### 19. Безопасность данных
**Вопрос:** Какие проблемы безопасности есть в этом коде?
```python
def get_user_data(user_id):
    query = f"SELECT * FROM users WHERE id = {user_id}"
    return database.execute(query)

def process_file_upload(file_content, filename):
    with open(f"uploads/{filename}", "wb") as f:
        f.write(file_content)
    return f"File saved as {filename}"

def authenticate_user(username, password):
    user = get_user_by_username(username)
    if user and user.password == password:
        return user
    return None
```

<details>
<summary>💡 Подсказка</summary>
SQL injection, path traversal, plaintext passwords
</details>

**Углубление:**
- Как правильно хэшировать пароли?
- Rate limiting и brute force protection
- Input validation и sanitization

---

## Тестирование

### 20. Стратегии тестирования
**Вопрос:** Напишите тесты для этого сервиса:
```python
class OrderService:
    def __init__(self, db: Database, payment: PaymentService, 
                 notification: NotificationService):
        self.db = db
        self.payment = payment
        self.notification = notification
    
    async def create_order(self, user_id: int, items: List[OrderItem]) -> Order:
        # Проверка пользователя
        # Создание заказа
        # Обработка платежа
        # Отправка уведомления
        pass
```

Покажите unit, integration и contract тесты.

<details>
<summary>💡 Подсказка</summary>
Используйте mocks, fixtures, test databases, pact testing
</details>

**Углубление:**
- Тестирование асинхронного кода
- Property-based testing с hypothesis
- Mutation testing для оценки качества тестов

---

## Системные вопросы

### 21. Конфигурация приложений
**Вопрос:** Спроектируйте систему конфигурации для микросервиса:
```
Требования:
- Environment-specific настройки (dev/staging/prod)
- Секреты (passwords, API keys)
- Feature flags
- Hot reload конфигурации
- Validation настроек
```

<details>
<summary>💡 Подсказка</summary>
Pydantic settings, environment variables, config files, external config services
</details>

**Углубление:**
- 12-factor app principles
- Secret management (Vault, AWS Secrets Manager)
- Configuration as Code

---

### 22. Логирование и мониторинг
**Вопрос:** Реализуйте structured logging:
```python
# Требования:
# - JSON формат логов
# - Correlation ID в каждом логе
# - Разные уровни логирования
# - Sanitization чувствительных данных
# - Интеграция с ELK stack
```

<details>
<summary>💡 Подсказка</summary>
structlog, contextual logging, log aggregation
</details>

**Углубление:**
- Centralized vs distributed logging
- Log sampling в high-load системах
- Alerting на основе логов

---

### 23. Graceful Shutdown
**Вопрос:** Реализуйте graceful shutdown для web-приложения:
```python
# Требования:
# - Завершение текущих запросов
# - Остановка background tasks
# - Закрытие соединений с БД
# - Cleanup ресурсов
# - Health check endpoints
```

<details>
<summary>💡 Подсказка</summary>
Signal handlers, async context managers, health checks
</details>

**Углубление:**
- Container orchestration (Kubernetes) integration
- Rolling deployments
- Circuit breaker patterns

---

## Архитектурные паттерны

### 24. Microservices Communication
**Вопрос:** Сравните способы коммуникации между микросервисами:
- Синхронные HTTP calls
- Асинхронные message queues  
- Event streaming
- gRPC

Когда какой подход использовать?

<details>
<summary>💡 Подсказка</summary>
Рассмотрите latency, reliability, coupling, data consistency
</details>

**Углубление:**
- Service mesh и traffic management
- API versioning strategies
- Distributed transactions и Saga pattern

---

### 25. CQRS и Event Sourcing
**Вопрос:** Спроектируйте CQRS архитектуру для banking системы:
```
Commands: TransferMoney, DepositMoney, WithdrawMoney
Queries: GetAccountBalance, GetTransactionHistory
Events: MoneyTransferred, MoneyDeposited, MoneyWithdrawn
```

<details>
<summary>💡 Подсказка</summary>
Separate read/write models, event store, projections
</details>

**Углубление:**
- Event versioning и schema evolution
- Snapshotting для оптимизации
- Eventual consistency challenges

---

## Финальные архитектурные вопросы

### 26. Database Design
**Вопрос:** Спроектируйте схему БД для социальной сети:
```
Entities: User, Post, Comment, Like, Follow
Requirements:
- Поддержка миллионов пользователей
- Быстрая лента новостей
- Поиск по постам
- Аналитика
```

<details>
<summary>💡 Подсказка</summary>
Denormalization, indexing, sharding, read replicas
</details>

**Углубление:**
- Шардинг strategies
- CAP theorem implications  
- Polyglot persistence

---

### 27. Performance at Scale
**Вопрос:** Как оптимизировать Python приложение для 10k RPS?
```
Текущие проблемы:
- Медленные database queries
- CPU-intensive computations
- Memory leaks
- GIL limitations
```

<details>
<summary>💡 Подсказка</summary>
Caching, async/await, connection pooling, profiling, horizontal scaling
</details>

**Углубление:**
- Load balancing strategies
- Database optimization techniques
- Когда переходить на другие языки?

---

### 28. Disaster Recovery
**Вопрос:** Спроектируйте disaster recovery план:
```
Requirements:
- RTO (Recovery Time Objective): 15 минут
- RPO (Recovery Point Objective): 5 минут
- Multi-region deployment
- Database backups
- Monitoring и alerting
```

<details>
<summary>💡 Подсказка</summary>
Backup strategies, replication, failover, chaos engineering
</details>

**Углубление:**
- Blue-green vs canary deployments
- Database migration strategies
- Testing disaster recovery procedures

---

## Практическое задание

### 29. Системный дизайн: URL Shortener
**Вопрос:** Спроектируйте систему сокращения URL (как bit.ly):

**Requirements:**
- 100M URLs создается в день
- 10:1 read/write ratio
- URL expiration
- Analytics
- Custom aliases
- Rate limiting

**Покажите:**
- High-level архитектуру
- Database schema
- API design
- Caching strategy
- Sharding approach

<details>
<summary>💡 Подсказка</summary>
Base62 encoding, consistent hashing, bloom filters, analytics pipeline
</details>

**Углубление:**
- Как обрабатывать hot keys?
- Geo-distributed deployment
- A/B testing infrastructure

---

### 30. Code Review
**Вопрос:** Проведите code review этого pull request:

```python
# Новый endpoint для получения пользователей
@app.route('/api/users')
def get_users():
    page = int(request.args.get('page', 1))
    users = []
    
    # Get all users from database
    connection = sqlite3.connect('users.db')
    cursor = connection.cursor()
    query = f"SELECT * FROM users LIMIT {(page-1)*10}, 10"
    results = cursor.execute(query).fetchall()
    
    for row in results:
        user_data = {
            'id': row[0],
            'name': row[1], 
            'email': row[2],
            'password': row[3],  # Include password for frontend
            'created_at': row[4]
        }
        users.append(user_data)
    
    connection.close()
    return jsonify(users)
```

**Найдите проблемы и предложите улучшения.**

<details>
<summary>💡 Подсказка</summary>
Security, performance, error handling, code organization, best practices
</details>

**Углубление:**
- Как структурировать code review процесс?
- Автоматизация через linting и static analysis
- Security scanning в CI/CD

---

## Заключение

Это собеседование покрывает:
- ✅ Основы Python и структуры данных
- ✅ Многопоточность и асинхронность  
- ✅ Архитектурные паттерны и принципы
- ✅ Системный дизайн и масштабирование
- ✅ Безопасность и производительность
- ✅ Практические навыки разработки

**Оценка кандидата:**
- **Junior:** 1-10 вопросов (основы Python, простая архитектура)
- **Middle:** 1-20 вопросов (+ concurrency, patterns, testing)  
- **Senior:** Все вопросы (+ system design, leadership, code review)

**Время:** 2-3 часа для полного собеседования, можно разбить на несколько сессий.