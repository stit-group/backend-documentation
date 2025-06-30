# Python Backend Interview - Развернутые ответы
## Подробные ответы с примерами кода

---

## 1. Основы Python & Memory Management

### 🟢 Базовый ответ: list, tuple, set
**list** - изменяемая упорядоченная коллекция. Используется когда нужно добавлять/удалять элементы.
**tuple** - неизменяемая упорядоченная коллекция. Используется для постоянных данных, ключей словарей.
**set** - изменяемая неупорядоченная коллекция уникальных элементов. Быстрый поиск и операции множеств.

```python
# Примеры использования
users = ['Alice', 'Bob']  # список для изменения
coordinates = (10, 20)    # tuple для постоянных данных
tags = {'python', 'web'}  # set для уникальных значений
```

### 🟡 Ответ на код:
```python
a = [1, 2, 3]  # Создаем список
b = a          # b указывает на тот же объект
c = a[:]       # c - новый список (копия)
b.append(4)    # Изменяем исходный список через b
c.append(5)    # Изменяем только копию
print(a, b, c) # [1, 2, 3, 4] [1, 2, 3, 4] [1, 2, 3, 5]
```

### 🔴 Экспертные вопросы:
**Сборщик мусора:** Python использует reference counting + cycle detector. Объекты удаляются когда счетчик ссылок = 0, циклические ссылки обрабатывает отдельный механизм.

**__slots__:** Ограничивает атрибуты класса, экономит память:
```python
class Point:
    __slots__ = ['x', 'y']  # Только эти атрибуты
```

---

## 2. Async/Await & Concurrency

### 🟢 Базовый ответ:
**Синхронный код** выполняется последовательно, блокируя выполнение.
**Асинхронный код** позволяет выполнять другие задачи во время ожидания I/O операций.

### 🟡 Практический пример:
```python
import asyncio

async def fetch_data(delay):
    await asyncio.sleep(delay)
    return f"Data after {delay}s"

# Параллельный запуск
async def main():
    tasks = [fetch_data(1), fetch_data(2), fetch_data(3)]
    results = await asyncio.gather(*tasks)
    return results

# Выполнится за 3 секунды, а не за 6
```

### 🔴 Экспертные концепции:
- **gather()** - ждет все задачи, останавливается при первой ошибке
- **wait()** - больше контроля, можно настроить условия завершения
- **as_completed()** - обрабатывает результаты по мере готовности

**GIL обход:** multiprocessing для CPU-bound задач, async для I/O-bound.

---

## 3. Web Framework Architecture

### 🟢 Базовый ответ:
**Django:** MTV (Model-Template-View), ORM, admin панель, монолитный подход.
**FastAPI:** ASGI, автогенерация документации, type hints, высокая производительность.
**Middleware** - промежуточный слой для обработки запросов/ответов.

### 🟡 Dependency Injection в FastAPI:
```python
from fastapi import Depends
from sqlalchemy.orm import Session

def get_db() -> Session:
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/users/{user_id}")
def get_user(user_id: int, db: Session = Depends(get_db)):
    return db.query(User).filter(User.id == user_id).first()
```

### 🔴 Экспертные знания:
**WSGI vs ASGI:** WSGI - синхронный протокол, ASGI - асинхронный с поддержкой WebSockets.

---

## 4. Database & ORM

### 🟢 Базовый ответ:
**ORM** - Object-Relational Mapping, работа с БД через объекты.
**Плюсы:** безопасность, портируемость, удобство.
**Минусы:** производительность, сложные запросы.

### 🟡 Решение N+1 проблемы:
```python
# Проблема N+1
users = session.query(User).all()  # 1 запрос
for user in users:
    print(user.posts)  # N запросов

# Решение - eager loading
users = session.query(User).options(joinedload(User.posts)).all()
# Или select_related в Django
users = User.objects.select_related('posts').all()
```

### 🔴 Экспертные концепции:
**Connection pooling:** Переиспользование соединений с БД для производительности.
**Sharding:** Горизонтальное разделение данных по разным серверам.

---

## 5. Testing & Code Quality

### 🟢 Базовый ответ:
- **Unit тесты** - тестируют отдельные функции/методы
- **Integration тесты** - взаимодействие компонентов
- **E2E тесты** - полный пользовательский сценарий
- **Мокинг** - замена зависимостей на фальшивые объекты

### 🟡 Тестирование с моками:
```python
import pytest
from unittest.mock import Mock

def test_payment_service():
    # Arrange
    mock_bank_api = Mock()
    mock_bank_api.charge.return_value = {'status': 'success'}
    service = PaymentService(mock_bank_api)
    
    # Act
    result = service.process_payment(100, 'card123')
    
    # Assert
    mock_bank_api.charge.assert_called_once_with(100, 'card123')
    assert result['status'] == 'success'

def test_invalid_amount():
    service = PaymentService(Mock())
    with pytest.raises(ValueError):
        service.process_payment(-10, 'card123')
```

---

## 6. Performance & Optimization

### 🟢 Базовый ответ:
**Профилирование:** cProfile, line_profiler, memory_profiler.
**Bottlenecks:** медленные запросы к БД, неэффективные алгоритмы, блокирующие операции.

### 🟡 Сравнение производительности:
```python
# Быстрее всего - list comprehension (вариант B)
# Причины:
# 1. Оптимизировано на уровне C
# 2. Меньше вызовов Python функций
# 3. Предвыделение памяти

# Вариант A - много append() вызовов
# Вариант C - много lambda вызовов (медленно)

# Измерение:
import timeit
timeit.timeit('[i*2 for i in range(1000) if i%2==0]', number=10000)
```

### 🔴 Стратегии оптимизации:
- **Caching:** Redis для сессий, Memcached для вычислений
- **Database:** индексы, query optimization, read replicas
- **Load balancing:** nginx, горизонтальное масштабирование

---

## 7. Security & Best Practices

### 🟢 OWASP Top 10 основы:
1. Injection атаки (SQL, NoSQL, OS)
2. Broken authentication
3. Sensitive data exposure
4. XML External Entities (XXE)
5. Broken access control

### 🟡 Уязвимости в коде:
```python
# УЯЗВИМЫЙ КОД:
@app.route('/user/<user_id>')
def get_user(user_id):
    query = f"SELECT * FROM users WHERE id = {user_id}"  # SQL Injection!
    result = db.execute(query)
    return jsonify(result)  # Возможна утечка данных

# БЕЗОПАСНЫЙ КОД:
@app.route('/user/<int:user_id>')  # Валидация типа
def get_user(user_id):
    query = "SELECT id, name, email FROM users WHERE id = %s"
    result = db.execute(query, (user_id,))  # Параметризированный запрос
    return jsonify(result)
```

### 🔴 Продвинутая безопасность:
**JWT vs Session:** JWT - stateless, хорошо для микросервисов. Session - server-side, легче отзывать.

---

## 8. Architecture & Patterns

### 🟢 SOLID принципы:
- **S** - Single Responsibility (одна ответственность)
- **O** - Open/Closed (открыт для расширения, закрыт для изменения)
- **L** - Liskov Substitution (подтипы заменяемы)
- **I** - Interface Segregation (много специфичных интерфейсов)
- **D** - Dependency Inversion (зависимость от абстракций)

### 🟡 Рефакторинг с паттернами:
```python
# Применяем Strategy, Observer, Repository паттерны
class DiscountStrategy:
    def apply(self, total, user): pass

class PremiumDiscount(DiscountStrategy):
    def apply(self, total, user):
        return total * 0.9 if user.is_premium else total

class OrderProcessor:
    def __init__(self, discount_strategy, notifier, repository):
        self.discount_strategy = discount_strategy
        self.notifier = notifier
        self.repository = repository
    
    def process(self, order):
        self._validate(order)
        total = self._calculate_total(order)
        total = self.discount_strategy.apply(total, order.user)
        self.notifier.send(order.user, f"Order total: {total}")
        return self.repository.save(order)
```

---

## 9. DevOps & Deployment

### 🟢 Docker основы:
**Docker** - контейнеризация приложений для консистентного развертывания.
**CI/CD** - автоматизация тестирования, сборки и развертывания.

### 🟡 Оптимизация Dockerfile:
```dockerfile
# Оптимизированный вариант:
FROM python:3.11-slim
WORKDIR /app

# Копируем requirements отдельно для кеширования
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Копируем код в конце
COPY . .

# Не root пользователь
RUN useradd -m appuser
USER appuser

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

---

## 10. Monitoring & Observability

### 🟢 Что логировать:
- Ошибки и исключения
- Медленные операции (>100ms)
- Бизнес-события (регистрация, покупки)
- Security события (неудачные логины)

### 🟡 Правильное логирование:
```python
import logging
import time
from functools import wraps

logger = logging.getLogger(__name__)

def monitor_performance(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        try:
            result = func(*args, **kwargs)
            logger.info(f"{func.__name__} completed in {time.time() - start_time:.2f}s")
            return result
        except Exception as e:
            logger.error(f"{func.__name__} failed: {e}", exc_info=True)
            # Отправить в систему мониторинга
            raise
    return wrapper

@monitor_performance
def process_payment(user_id, amount):
    logger.info(f"Processing payment: user={user_id}, amount={amount}")
    # ... логика обработки
```

---

## 11. Data Processing & APIs

### 🟢 REST принципы:
- **Stateless** - каждый запрос независим
- **HTTP методы** - GET (чтение), POST (создание), PUT (обновление), DELETE (удаление)
- **Статус коды** - 200 (OK), 201 (Created), 404 (Not Found), 500 (Server Error)

### 🟡 Обработка больших файлов:
```python
import pandas as pd

def process_large_csv(file_path):
    # Читаем по частям (chunks)
    chunk_size = 10000
    for chunk in pd.read_csv(file_path, chunksize=chunk_size):
        # Обрабатываем каждый chunk
        processed = chunk.groupby('category').sum()
        # Сохраняем результат
        processed.to_sql('results', engine, if_exists='append')
```

---

## 12. System Design

### 🔴 Twitter-like система:

**Архитектура:**
```
Load Balancer → API Gateway → Microservices
                           ↓
User Service | Tweet Service | Timeline Service
                           ↓
     Database Cluster (Sharded)
                           ↓
        Cache Layer (Redis)
                           ↓
         CDN для медиа
```

**Ключевые решения:**
- **Sharding по user_id** для tweets
- **Fan-out модель** для timeline (push vs pull)
- **Redis для timeline cache**
- **Message queue** для асинхронной обработки
- **CDN** для изображений/видео

**Масштабирование:**
- Read replicas для чтения
- Horizontal sharding для записи
- Микросервисная архитектура
- Async processing для тяжелых операций

---

## 💡 Практические советы для интервью

### Для кандидатов:
1. **Думайте вслух** - объясняйте свой мыслительный процесс
2. **Задавайте вопросы** - уточняйте требования
3. **Начинайте просто** - потом усложняйте
4. **Обсуждайте trade-offs** - почему выбрали именно это решение

### Для интервьюеров:
1. **Оценивайте процесс**, не только результат
2. **Помогайте наводящими вопросами**
3. **Обращайте внимание на code style**
4. **Проверяйте понимание принципов**, не заучивание

---

*Помните: хорошее собеседование - это диалог, где обе стороны узнают друг друга лучше.*