# Python Backend Developer: Полное руководство от нуля до эксперта

## 🎯 Введение: Ваш путь в backend разработку

### Почему Python доминирует в backend?

Python стал языком выбора для backend разработки не случайно. Рассмотрим статистику:

```
Рост использования Python в backend (2020-2024):
├── 2020: 31% проектов
├── 2021: 38% проектов  
├── 2022: 45% проектов
├── 2023: 52% проектов
└── 2024: 58% проектов
```

**🏢 Кто использует Python в backend:**
- **Instagram** - 500+ млн пользователей на Django
- **Spotify** - рекомендательная система на Python
- **Netflix** - микросервисы обработки контента
- **Uber** - система динамического ценообразования
- **Dropbox** - файловое хранилище и синхронизация

### Экосистема Python для backend

```
Python Backend Ecosystem
│
├── 🌐 Web Frameworks
│   ├── FastAPI (современный, быстрый)
│   ├── Django (полнофункциональный)
│   ├── Flask (минималистичный)
│   └── Starlette (асинхронный)
│
├── 🗄️ Databases & ORM
│   ├── SQLAlchemy (самый популярный ORM)
│   ├── Django ORM (встроенный в Django)
│   ├── Tortoise ORM (асинхронный)
│   └── Peewee (легковесный)
│
├── ⚡ Performance & Caching
│   ├── Redis (кэширование, сессии)
│   ├── Celery (фоновые задачи)
│   ├── Gunicorn (WSGI сервер)
│   └── Uvicorn (ASGI сервер)
│
└── 🛠️ Tools & Libraries
    ├── Pydantic (валидация данных)
    ├── Pytest (тестирование)
    ├── Alembic (миграции БД)
    └── Poetry (управление зависимостями)
```

### Карьерная траектория Python Backend разработчика

```
Уровень          Зарплата*         Обязанности
├── Trainee      $800-1500         Изучение основ, простые задачи
├── Junior       $1500-3000        CRUD API, работа с БД
├── Middle       $3000-5500        Архитектура, оптимизация
├── Senior       $5500-8500        Техлидство, сложные системы
└── Lead/Architect $8500+          Стратегия, менторство

* Зарплаты указаны для рынка СНГ в USD
```

---

## 📚 Фаза 1: Основы Python (4-6 недель)

### 🎯 Цели фазы:
- Освоить синтаксис Python
- Понять принципы структур данных
- Изучить объектно-ориентированное программирование
- Научиться работать с файлами и модулями

---

## 📅 Неделя 1-2: Базовый синтаксис и структуры данных

### Философия Python: Zen of Python

```python
import this
```

**Ключевые принципы для backend разработчика:**
1. **Читаемость имеет значение** - код читается чаще, чем пишется
2. **Простое лучше сложного** - избегайте overengineering
3. **Плоское лучше вложенного** - минимизируйте уровни вложенности
4. **Явное лучше неявного** - код должен быть понятным

### Типы данных в контексте backend

#### 1. Числовые типы для бизнес-логики

```python
# Финансовые расчеты - используйте Decimal для точности
from decimal import Decimal

# ❌ Неправильно - потеря точности
price = 19.99
tax_rate = 0.08
total = price * (1 + tax_rate)  # 21.589999999999996

# ✅ Правильно - точные вычисления
price = Decimal('19.99')
tax_rate = Decimal('0.08')
total = price * (1 + tax_rate)  # 21.59

# Работа с ID и счетчиками
user_id = 12345
page_number = 1
items_per_page = 20
offset = (page_number - 1) * items_per_page

# Проверка валидности
def validate_pagination(page: int, limit: int) -> tuple[int, int]:
    """Валидация пагинации"""
    page = max(1, page)  # Минимум 1
    limit = min(max(1, limit), 100)  # От 1 до 100
    offset = (page - 1) * limit
    return offset, limit
```

#### 2. Строки для обработки данных

```python
# URL и путь обработка
api_base = "https://api.example.com"
version = "v1"
endpoint = "users"
user_id = 123

# Формирование URL
url = f"{api_base}/{version}/{endpoint}/{user_id}"
# Результат: https://api.example.com/v1/users/123

# Валидация и очистка данных
def clean_username(username: str) -> str:
    """Очистка имени пользователя"""
    # Убираем лишние пробелы, приводим к нижнему регистру
    cleaned = username.strip().lower()
    
    # Проверяем на допустимые символы
    import re
    if not re.match(r'^[a-z0-9_]+$', cleaned):
        raise ValueError("Username содержит недопустимые символы")
    
    return cleaned

# Работа с многострочным текстом
email_template = """
Здравствуйте, {name}!

Спасибо за регистрацию на нашем сайте.
Ваш логин: {username}
Дата регистрации: {date}

С уважением,
Команда проекта
""".strip()

# Использование
formatted_email = email_template.format(
    name="Иван Петров",
    username="ivan_petrov", 
    date="2024-01-15"
)
```

#### 3. Списки для коллекций данных

```python
# Обработка списка пользователей
users = [
    {"id": 1, "name": "Alice", "role": "admin", "active": True},
    {"id": 2, "name": "Bob", "role": "user", "active": True},
    {"id": 3, "name": "Charlie", "role": "user", "active": False},
    {"id": 4, "name": "Diana", "role": "moderator", "active": True}
]

# Фильтрация активных пользователей
active_users = [user for user in users if user["active"]]

# Получение списка имен
names = [user["name"] for user in users]

# Группировка по ролям
from collections import defaultdict

users_by_role = defaultdict(list)
for user in users:
    users_by_role[user["role"]].append(user)

# Сортировка пользователей
sorted_users = sorted(users, key=lambda u: u["name"])

# Пагинация списка
def paginate_list(items: list, page: int, per_page: int) -> dict:
    """Пагинация списка элементов"""
    total = len(items)
    start = (page - 1) * per_page
    end = start + per_page
    
    return {
        "items": items[start:end],
        "total": total,
        "page": page,
        "per_page": per_page,
        "pages": (total + per_page - 1) // per_page
    }

# Использование
page_data = paginate_list(users, page=1, per_page=2)
```

#### 4. Словари для структурированных данных

```python
# Конфигурация приложения
app_config = {
    "database": {
        "host": "localhost",
        "port": 5432,
        "name": "myapp",
        "user": "postgres"
    },
    "redis": {
        "host": "localhost", 
        "port": 6379,
        "db": 0
    },
    "api": {
        "host": "0.0.0.0",
        "port": 8000,
        "debug": False
    }
}

# Безопасное получение вложенных значений
def get_nested_value(data: dict, path: str, default=None):
    """Получение значения по пути вида 'database.host'"""
    keys = path.split('.')
    value = data
    
    for key in keys:
        if isinstance(value, dict) and key in value:
            value = value[key]
        else:
            return default
    
    return value

# Использование
db_host = get_nested_value(app_config, "database.host", "localhost")
cache_ttl = get_nested_value(app_config, "cache.ttl", 3600)

# HTTP Response структуры
def success_response(data, message="Success"):
    """Стандартный успешный ответ"""
    return {
        "status": "success",
        "message": message,
        "data": data,
        "timestamp": datetime.now().isoformat()
    }

def error_response(message, code=400, details=None):
    """Стандартный ответ с ошибкой"""
    response = {
        "status": "error", 
        "message": message,
        "code": code,
        "timestamp": datetime.now().isoformat()
    }
    
    if details:
        response["details"] = details
        
    return response

# Кэш в памяти с TTL
class SimpleCache:
    def __init__(self):
        self._cache = {}
        self._expiry = {}
    
    def set(self, key: str, value, ttl: int = 3600):
        """Установить значение с TTL в секундах"""
        import time
        self._cache[key] = value
        self._expiry[key] = time.time() + ttl
    
    def get(self, key: str, default=None):
        """Получить значение, проверив TTL"""
        import time
        
        if key not in self._cache:
            return default
            
        if time.time() > self._expiry[key]:
            del self._cache[key]
            del self._expiry[key]
            return default
            
        return self._cache[key]
```

### Диаграмма: Структуры данных Python в backend

```
Backend Data Structures Usage
│
├── 📊 Числа (int, float, Decimal)
│   ├── ID пользователей и записей
│   ├── Счетчики и метрики  
│   ├── Финансовые расчеты (Decimal)
│   └── Временные метки (timestamp)
│
├── 📝 Строки (str)
│   ├── Пользовательский ввод
│   ├── URL и пути API
│   ├── Email и шаблоны
│   └── JSON строки
│
├── 📋 Списки (list)
│   ├── Коллекции записей из БД
│   ├── Списки пользователей/товаров
│   ├── Результаты поиска
│   └── Очереди задач
│
├── 🗂️ Словари (dict)
│   ├── JSON данные от клиента
│   ├── Конфигурация приложения
│   ├── HTTP заголовки
│   └── Кэш ключ-значение
│
└── 🔧 Множества (set)
    ├── Уникальные ID
    ├── Роли пользователей
    ├── Теги и категории
    └── Права доступа
```

---

## 📅 Неделя 3-4: Управляющие конструкции и функции

### Условные конструкции в backend логике

#### 1. Аутентификация и авторизация

```python
from enum import Enum
from typing import Optional

class UserRole(Enum):
    ADMIN = "admin"
    MODERATOR = "moderator" 
    USER = "user"
    GUEST = "guest"

class Permission:
    def __init__(self, user_role: UserRole, resource: str, action: str):
        self.user_role = user_role
        self.resource = resource
        self.action = action
    
    def can_access(self) -> bool:
        """Проверка прав доступа"""
        
        # Админ может все
        if self.user_role == UserRole.ADMIN:
            return True
        
        # Модератор может читать и редактировать, но не удалять
        if self.user_role == UserRole.MODERATOR:
            if self.action in ["read", "update"]:
                return True
            elif self.action == "delete" and self.resource in ["comments", "posts"]:
                return True
            else:
                return False
        
        # Обычный пользователь может только читать и редактировать свои данные
        if self.user_role == UserRole.USER:
            if self.action == "read":
                return True
            elif self.action in ["update", "delete"] and self.resource == "own_profile":
                return True
            else:
                return False
        
        # Гость может только читать публичные данные
        if self.user_role == UserRole.GUEST:
            return self.action == "read" and self.resource in ["posts", "comments"]
        
        return False

# Использование
def check_permission(user_role: str, resource: str, action: str) -> bool:
    permission = Permission(UserRole(user_role), resource, action)
    return permission.can_access()

# Примеры
print(check_permission("user", "own_profile", "update"))  # True
print(check_permission("user", "admin_panel", "read"))    # False
print(check_permission("moderator", "posts", "delete"))   # True
```

#### 2. Валидация данных

```python
import re
from datetime import datetime, date
from typing import Union, List

class ValidationError(Exception):
    """Кастомное исключение для ошибок валидации"""
    pass

class UserValidator:
    @staticmethod
    def validate_email(email: str) -> str:
        """Валидация email адреса"""
        if not email:
            raise ValidationError("Email обязателен")
        
        # Простая regex для email
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        if not re.match(pattern, email):
            raise ValidationError("Неверный формат email")
        
        return email.lower().strip()
    
    @staticmethod
    def validate_password(password: str) -> str:
        """Валидация пароля"""
        errors = []
        
        if len(password) < 8:
            errors.append("Пароль должен содержать минимум 8 символов")
        
        if not re.search(r'[A-Z]', password):
            errors.append("Пароль должен содержать заглавную букву")
        
        if not re.search(r'[a-z]', password):
            errors.append("Пароль должен содержать строчную букву")
        
        if not re.search(r'\d', password):
            errors.append("Пароль должен содержать цифру")
        
        if errors:
            raise ValidationError("; ".join(errors))
        
        return password
    
    @staticmethod
    def validate_age(birth_date: Union[str, date]) -> int:
        """Валидация возраста по дате рождения"""
        if isinstance(birth_date, str):
            try:
                birth_date = datetime.strptime(birth_date, "%Y-%m-%d").date()
            except ValueError:
                raise ValidationError("Неверный формат даты. Используйте YYYY-MM-DD")
        
        today = date.today()
        age = today.year - birth_date.year - ((today.month, today.day) < (birth_date.month, birth_date.day))
        
        if age < 0:
            raise ValidationError("Дата рождения не может быть в будущем")
        elif age < 13:
            raise ValidationError("Минимальный возраст: 13 лет")
        elif age > 120:
            raise ValidationError("Максимальный возраст: 120 лет")
        
        return age

def validate_user_data(data: dict) -> dict:
    """Комплексная валидация данных пользователя"""
    validated = {}
    errors = []
    
    # Валидация email
    try:
        validated['email'] = UserValidator.validate_email(data.get('email', ''))
    except ValidationError as e:
        errors.append(f"Email: {e}")
    
    # Валидация пароля
    try:
        validated['password'] = UserValidator.validate_password(data.get('password', ''))
    except ValidationError as e:
        errors.append(f"Пароль: {e}")
    
    # Валидация возраста
    try:
        validated['age'] = UserValidator.validate_age(data.get('birth_date', ''))
    except ValidationError as e:
        errors.append(f"Возраст: {e}")
    
    if errors:
        raise ValidationError("; ".join(errors))
    
    return validated
```

### Циклы для обработки данных

#### 1. Обработка больших наборов данных

```python
from typing import Iterator, Dict, Any
import json

def process_users_batch(users: List[Dict]) -> Dict[str, Any]:
    """Пакетная обработка пользователей"""
    stats = {
        "total": 0,
        "active": 0,
        "inactive": 0,
        "by_role": {},
        "errors": []
    }
    
    for user in users:
        stats["total"] += 1
        
        try:
            # Проверяем обязательные поля
            if not all(key in user for key in ['id', 'email', 'role']):
                stats["errors"].append(f"User {user.get('id', 'unknown')}: missing required fields")
                continue
            
            # Подсчет по статусу
            if user.get('is_active', False):
                stats["active"] += 1
            else:
                stats["inactive"] += 1
            
            # Подсчет по ролям
            role = user['role']
            stats["by_role"][role] = stats["by_role"].get(role, 0) + 1
            
        except Exception as e:
            stats["errors"].append(f"User {user.get('id', 'unknown')}: {str(e)}")
    
    return stats

# Генератор для обработки больших файлов
def read_large_json_file(filename: str) -> Iterator[Dict]:
    """Чтение большого JSON файла по частям"""
    with open(filename, 'r', encoding='utf-8') as file:
        data = json.load(file)
        
        # Если это список пользователей
        if isinstance(data, list):
            for item in data:
                yield item
        # Если это объект с пользователями
        elif 'users' in data:
            for item in data['users']:
                yield item

# Использование генератора
def process_large_dataset(filename: str, batch_size: int = 1000):
    """Обработка большого датасета батчами"""
    batch = []
    
    for user in read_large_json_file(filename):
        batch.append(user)
        
        # Обработка батча
        if len(batch) >= batch_size:
            stats = process_users_batch(batch)
            print(f"Обработан батч: {stats['total']} пользователей")
            batch = []  # Очищаем батч
    
    # Обработка последнего неполного батча
    if batch:
        stats = process_users_batch(batch)
        print(f"Обработан последний батч: {stats['total']} пользователей")
```

#### 2. Агрегация и группировка данных

```python
from collections import defaultdict, Counter
from datetime import datetime, timedelta

def analyze_user_activity(logs: List[Dict]) -> Dict[str, Any]:
    """Анализ активности пользователей"""
    
    # Группировка по пользователям
    user_stats = defaultdict(lambda: {
        "sessions": 0,
        "total_time": 0,
        "last_activity": None,
        "actions": Counter()
    })
    
    # Группировка по дням
    daily_stats = defaultdict(lambda: {
        "unique_users": set(),
        "total_sessions": 0,
        "popular_actions": Counter()
    })
    
    for log in logs:
        user_id = log['user_id']
        action = log['action']
        timestamp = datetime.fromisoformat(log['timestamp'])
        session_time = log.get('session_time', 0)
        
        # Статистика пользователя
        user_stats[user_id]["sessions"] += 1
        user_stats[user_id]["total_time"] += session_time
        user_stats[user_id]["actions"][action] += 1
        
        if (user_stats[user_id]["last_activity"] is None or 
            timestamp > user_stats[user_id]["last_activity"]):
            user_stats[user_id]["last_activity"] = timestamp
        
        # Дневная статистика
        day_key = timestamp.date().isoformat()
        daily_stats[day_key]["unique_users"].add(user_id)
        daily_stats[day_key]["total_sessions"] += 1
        daily_stats[day_key]["popular_actions"][action] += 1
    
    # Подготовка результата
    result = {
        "summary": {
            "total_users": len(user_stats),
            "total_sessions": sum(stats["sessions"] for stats in user_stats.values()),
            "avg_session_time": sum(stats["total_time"] for stats in user_stats.values()) / len(user_stats) if user_stats else 0
        },
        "top_users": [],
        "daily_breakdown": {}
    }
    
    # Топ активных пользователей
    sorted_users = sorted(
        user_stats.items(), 
        key=lambda x: x[1]["sessions"], 
        reverse=True
    )
    
    for user_id, stats in sorted_users[:10]:  # Топ 10
        result["top_users"].append({
            "user_id": user_id,
            "sessions": stats["sessions"],
            "total_time": stats["total_time"],
            "last_activity": stats["last_activity"].isoformat() if stats["last_activity"] else None,
            "top_actions": dict(stats["actions"].most_common(3))
        })
    
    # Дневная статистика
    for day, stats in daily_stats.items():
        result["daily_breakdown"][day] = {
            "unique_users": len(stats["unique_users"]),
            "total_sessions": stats["total_sessions"],
            "top_actions": dict(stats["popular_actions"].most_common(5))
        }
    
    return result
```

### Функции как строительные блоки

#### 1. Декораторы для backend функциональности

```python
import time
import functools
from typing import Callable, Any

def timer(func: Callable) -> Callable:
    """Декоратор для измерения времени выполнения"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        print(f"{func.__name__} выполнилась за {end_time - start_time:.4f} секунд")
        return result
    return wrapper

def retry(max_attempts: int = 3, delay: float = 1.0):
    """Декоратор для повторных попыток"""
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    if attempt < max_attempts - 1:
                        print(f"Попытка {attempt + 1} неудачна: {e}. Повтор через {delay} сек...")
                        time.sleep(delay)
                    else:
                        print(f"Все {max_attempts} попыток исчерпаны")
            
            raise last_exception
        return wrapper
    return decorator

def cache_result(ttl: int = 300):
    """Простой декоратор кэширования с TTL"""
    def decorator(func: Callable) -> Callable:
        cache = {}
        
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # Создаем ключ кэша
            cache_key = f"{func.__name__}:{hash(str(args) + str(sorted(kwargs.items())))}"
            current_time = time.time()
            
            # Проверяем кэш
            if cache_key in cache:
                result, timestamp = cache[cache_key]
                if current_time - timestamp < ttl:
                    print(f"Cache hit for {func.__name__}")
                    return result
            
            # Выполняем функцию и кэшируем результат
            result = func(*args, **kwargs)
            cache[cache_key] = (result, current_time)
            print(f"Cache miss for {func.__name__}")
            return result
            
        return wrapper
    return decorator

# Использование декораторов
@timer
@retry(max_attempts=3, delay=2.0)
@cache_result(ttl=600)
def fetch_user_data(user_id: int) -> Dict[str, Any]:
    """Получение данных пользователя с внешнего API"""
    import random
    
    # Симуляция возможной ошибки
    if random.random() < 0.3:
        raise ConnectionError("Временная ошибка сети")
    
    # Симуляция медленного API
    time.sleep(1)
    
    return {
        "id": user_id,
        "name": f"User {user_id}",
        "email": f"user{user_id}@example.com",
        "created_at": datetime.now().isoformat()
    }
```

#### 2. Функции высшего порядка для обработки данных

```python
from typing import List, Callable, TypeVar, Optional

T = TypeVar('T')
U = TypeVar('U')

def pipeline(*functions: Callable) -> Callable:
    """Создание пайплайна обработки данных"""
    def process(data):
        result = data
        for func in functions:
            result = func(result)
        return result
    return process

def filter_and_map(
    data: List[T], 
    filter_func: Callable[[T], bool], 
    map_func: Callable[[T], U]
) -> List[U]:
    """Комбинированная фильтрация и преобразование"""
    return [map_func(item) for item in data if filter_func(item)]

def reduce_with_key(
    data: List[T], 
    key_func: Callable[[T], str], 
    reduce_func: Callable[[List[T]], U]
) -> Dict[str, U]:
    """Группировка по ключу и агрегация"""
    groups = {}
    
    for item in data:
        key = key_func(item)
        if key not in groups:
            groups[key] = []
        groups[key].append(item)
    
    return {key: reduce_func(items) for key, items in groups.items()}

# Пример использования с данными о заказах
orders = [
    {"id": 1, "user_id": 10, "amount": 100, "status": "completed", "date": "2024-01-01"},
    {"id": 2, "user_id": 11, "amount": 250, "status": "completed", "date": "2024-01-01"},
    {"id": 3, "user_id": 10, "amount": 75, "status": "pending", "date": "2024-01-02"},
    {"id": 4, "user_id": 12, "amount": 300, "status": "completed", "date": "2024-01-02"},
    {"id": 5, "user_id": 11, "amount": 150, "status": "cancelled", "date": "2024-01-03"},
]

# Функции для обработки
def is_completed(order: Dict) -> bool:
    return order["status"] == "completed"

def extract_amount(order: Dict) -> float:
    return order["amount"]

def sum_amounts(orders: List[Dict]) -> float:
    return sum(order["amount"] for order in orders)

def get_user_id(order: Dict) -> str:
    return f"user_{order['user_id']}"

def get_date(order: Dict) -> str:
    return order["date"]

# Создание пайплайна обработки
def process_completed_orders():
    # Только завершенные заказы -> суммы
    completed_amounts = filter_and_map(orders, is_completed, extract_amount)
    print(f"Суммы завершенных заказов: {completed_amounts}")
    
    # Группировка по пользователям
    revenue_by_user = reduce_with_key(
        [order for order in orders if is_completed(order)],
        get_user_id,
        sum_amounts
    )
    print(f"Выручка по пользователям: {revenue_by_user}")
    
    # Группировка по дням
    revenue_by_date = reduce_with_key(
        [order for order in orders if is_completed(order)],
        get_date,
        sum_amounts
    )
    print(f"Выручка по дням: {revenue_by_date}")

process_completed_orders()
```

---

## 📅 Неделя 5-6: ООП и работа с файлами

### Объектно-ориентированное программирование в backend

#### 1. Модели данных как классы

```python
from datetime import datetime, date
from typing import Optional, List, Dict, Any
from enum import Enum
import hashlib
import json

class UserStatus(Enum):
    ACTIVE = "active"
    INACTIVE = "inactive"
    SUSPENDED = "suspended"
    DELETED = "deleted"

class User:
    """Модель пользователя"""
    
    def __init__(self, email: str, password: str, name: str, birth_date: date = None):
        self.id: Optional[int] = None
        self.email = email
        self._password_hash = self._hash_password(password)
        self.name = name
        self.birth_date = birth_date
        self.status = UserStatus.ACTIVE
        self.created_at = datetime.utcnow()
        self.updated_at = datetime.utcnow()
        self.last_login: Optional[datetime] = None
        self._login_attempts = 0
        self._is_locked = False
    
    @staticmethod
    def _hash_password(password: str) -> str:
        """Хеширование пароля"""
        return hashlib.sha256(password.encode()).hexdigest()
    
    def verify_password(self, password: str) -> bool:
        """Проверка пароля"""
        if self._is_locked:
            raise ValueError("Аккаунт заблокирован")
        
        is_valid = self._password_hash == self._hash_password(password)
        
        if is_valid:
            self._login_attempts = 0
            self.last_login = datetime.utcnow()
        else:
            self._login_attempts += 1
            if self._login_attempts >= 5:
                self._is_locked = True
                raise ValueError("Аккаунт заблокирован после 5 неудачных попыток")
        
        return is_valid
    
    def change_password(self, old_password: str, new_password: str):
        """Смена пароля"""
        if not self.verify_password(old_password):
            raise ValueError("Неверный текущий пароль")
        
        self._password_hash = self._hash_password(new_password)
        self.updated_at = datetime.utcnow()
    
    def deactivate(self, reason: str = ""):
        """Деактивация пользователя"""
        self.status = UserStatus.INACTIVE
        self.updated_at = datetime.utcnow()
        # Логирование причины деактивации
        print(f"User {self.email} deactivated. Reason: {reason}")
    
    def to_dict(self, include_sensitive: bool = False) -> Dict[str, Any]:
        """Преобразование в словарь"""
        data = {
            "id": self.id,
            "email": self.email,
            "name": self.name,
            "status": self.status.value,
            "created_at": self.created_at.isoformat(),
            "updated_at": self.updated_at.isoformat(),
            "last_login": self.last_login.isoformat() if self.last_login else None
        }
        
        if self.birth_date:
            data["birth_date"] = self.birth_date.isoformat()
        
        if include_sensitive:
            data["_login_attempts"] = self._login_attempts
            data["_is_locked"] = self._is_locked
        
        return data
    
    @classmethod
    def from_dict(cls, data: Dict[str, Any]) -> 'User':
        """Создание объекта из словаря"""
        user = cls.__new__(cls)  # Создаем объект без вызова __init__
        
        user.id = data.get("id")
        user.email = data["email"]
        user._password_hash = data["password_hash"]
        user.name = data["name"]
        
        if "birth_date" in data and data["birth_date"]:
            user.birth_date = date.fromisoformat(data["birth_date"])
        else:
            user.birth_date = None
        
        user.status = UserStatus(data.get("status", "active"))
        user.created_at = datetime.fromisoformat(data["created_at"])
        user.updated_at = datetime.fromisoformat(data["updated_at"])
        
        if data.get("last_login"):
            user.last_login = datetime.fromisoformat(data["last_login"])
        else:
            user.last_login = None
        
        user._login_attempts = data.get("_login_attempts", 0)
        user._is_locked = data.get("_is_locked", False)
        
        return user
    
    def __str__(self) -> str:
        return f"User(id={self.id}, email={self.email}, status={self.status.value})"
    
    def __repr__(self) -> str:
        return (f"User(id={self.id}, email='{self.email}', name='{self.name}', "
                f"status={self.status.value}, created_at='{self.created_at}')")
```

#### 2. Наследование для расширения функциональности

```python
class AdminUser(User):
    """Администратор с расширенными правами"""
    
    def __init__(self, email: str, password: str, name: str, department: str = "IT"):
        super().__init__(email, password, name)
        self.department = department
        self.admin_level = 1
        self.can_manage_users = True
        self.can_access_logs = True
    
    def promote_user(self, user: User, new_status: UserStatus):
        """Изменение статуса пользователя"""
        if not self.can_manage_users:
            raise PermissionError("Недостаточно прав для управления пользователями")
        
        old_status = user.status
        user.status = new_status
        user.updated_at = datetime.utcnow()
        
        print(f"Admin {self.email} changed user {user.email} status from {old_status.value} to {new_status.value}")
    
    def view_user_details(self, user: User) -> Dict[str, Any]:
        """Просмотр детальной информации о пользователе"""
        if not self.can_access_logs:
            raise PermissionError("Недостаточно прав для просмотра детальной информации")
        
        return user.to_dict(include_sensitive=True)
    
    def to_dict(self, include_sensitive: bool = False) -> Dict[str, Any]:
        data = super().to_dict(include_sensitive)
        data.update({
            "department": self.department,
            "admin_level": self.admin_level,
            "can_manage_users": self.can_manage_users,
            "can_access_logs": self.can_access_logs,
            "user_type": "admin"
        })
        return data

class PremiumUser(User):
    """Премиум пользователь с дополнительными возможностями"""
    
    def __init__(self, email: str, password: str, name: str, subscription_plan: str = "basic"):
        super().__init__(email, password, name)
        self.subscription_plan = subscription_plan
        self.subscription_start = datetime.utcnow()
        self.subscription_end = None
        self.storage_limit_gb = self._get_storage_limit()
        self.api_rate_limit = self._get_api_rate_limit()
    
    def _get_storage_limit(self) -> int:
        """Определение лимита хранилища по тарифу"""
        limits = {
            "basic": 10,
            "premium": 100,
            "enterprise": 1000
        }
        return limits.get(self.subscription_plan, 1)
    
    def _get_api_rate_limit(self) -> int:
        """Определение лимита API запросов по тарифу"""
        limits = {
            "basic": 1000,
            "premium": 10000,
            "enterprise": 100000
        }
        return limits.get(self.subscription_plan, 100)
    
    def upgrade_subscription(self, new_plan: str):
        """Обновление подписки"""
        if new_plan == self.subscription_plan:
            return
        
        old_plan = self.subscription_plan
        self.subscription_plan = new_plan
        self.storage_limit_gb = self._get_storage_limit()
        self.api_rate_limit = self._get_api_rate_limit()
        self.updated_at = datetime.utcnow()
        
        print(f"User {self.email} upgraded from {old_plan} to {new_plan}")
    
    def is_subscription_active(self) -> bool:
        """Проверка активности подписки"""
        if self.subscription_end is None:
            return True
        return datetime.utcnow() < self.subscription_end
    
    def to_dict(self, include_sensitive: bool = False) -> Dict[str, Any]:
        data = super().to_dict(include_sensitive)
        data.update({
            "subscription_plan": self.subscription_plan,
            "subscription_start": self.subscription_start.isoformat(),
            "subscription_end": self.subscription_end.isoformat() if self.subscription_end else None,
            "storage_limit_gb": self.storage_limit_gb,
            "api_rate_limit": self.api_rate_limit,
            "user_type": "premium"
        })
        return data
```

#### 3. Система управления пользователями

```python
class UserManager:
    """Менеджер для управления пользователями"""
    
    def __init__(self):
        self._users: Dict[int, User] = {}
        self._email_index: Dict[str, int] = {}
        self._next_id = 1
    
    def create_user(self, email: str, password: str, name: str, 
                   user_type: str = "regular", **kwargs) -> User:
        """Создание нового пользователя"""
        
        # Проверка уникальности email
        if email in self._email_index:
            raise ValueError(f"Пользователь с email {email} уже существует")
        
        # Создание пользователя по типу
        if user_type == "admin":
            user = AdminUser(email, password, name, kwargs.get("department", "IT"))
        elif user_type == "premium":
            user = PremiumUser(email, password, name, kwargs.get("subscription_plan", "basic"))
        else:
            user = User(email, password, name, kwargs.get("birth_date"))
        
        # Присвоение ID и сохранение
        user.id = self._next_id
        self._users[user.id] = user
        self._email_index[email] = user.id
        self._next_id += 1
        
        print(f"Создан пользователь: {user}")
        return user
    
    def get_user_by_id(self, user_id: int) -> Optional[User]:
        """Получение пользователя по ID"""
        return self._users.get(user_id)
    
    def get_user_by_email(self, email: str) -> Optional[User]:
        """Получение пользователя по email"""
        user_id = self._email_index.get(email)
        return self._users.get(user_id) if user_id else None
    
    def authenticate(self, email: str, password: str) -> Optional[User]:
        """Аутентификация пользователя"""
        user = self.get_user_by_email(email)
        if user and user.verify_password(password):
            return user
        return None
    
    def get_users_by_status(self, status: UserStatus) -> List[User]:
        """Получение пользователей по статусу"""
        return [user for user in self._users.values() if user.status == status]
    
    def get_active_users(self, limit: int = None) -> List[User]:
        """Получение активных пользователей"""
        active_users = self.get_users_by_status(UserStatus.ACTIVE)
        return active_users[:limit] if limit else active_users
    
    def get_statistics(self) -> Dict[str, Any]:
        """Статистика пользователей"""
        total_users = len(self._users)
        
        status_counts = {}
        type_counts = {"regular": 0, "admin": 0, "premium": 0}
        
        for user in self._users.values():
            # Подсчет по статусам
            status = user.status.value
            status_counts[status] = status_counts.get(status, 0) + 1
            
            # Подсчет по типам
            if isinstance(user, AdminUser):
                type_counts["admin"] += 1
            elif isinstance(user, PremiumUser):
                type_counts["premium"] += 1
            else:
                type_counts["regular"] += 1
        
        return {
            "total_users": total_users,
            "by_status": status_counts,
            "by_type": type_counts,
            "last_created": max(self._users.values(), key=lambda u: u.created_at).created_at.isoformat() if self._users else None
        }
    
    def export_users(self, filename: str = "users_export.json"):
        """Экспорт пользователей в JSON"""
        users_data = []
        for user in self._users.values():
            user_dict = user.to_dict()
            # Добавляем хеш пароля для восстановления
            user_dict["password_hash"] = user._password_hash
            users_data.append(user_dict)
        
        with open(filename, 'w', encoding='utf-8') as f:
            json.dump(users_data, f, ensure_ascii=False, indent=2)
        
        print(f"Экспортировано {len(users_data)} пользователей в {filename}")
    
    def import_users(self, filename: str):
        """Импорт пользователей из JSON"""
        with open(filename, 'r', encoding='utf-8') as f:
            users_data = json.load(f)
        
        imported_count = 0
        for user_data in users_data:
            try:
                user = User.from_dict(user_data)
                
                # Проверка уникальности email
                if user.email in self._email_index:
                    print(f"Пользователь {user.email} уже существует, пропускаем")
                    continue
                
                # Присвоение нового ID если не указан
                if user.id is None or user.id in self._users:
                    user.id = self._next_id
                    self._next_id += 1
                else:
                    self._next_id = max(self._next_id, user.id + 1)
                
                self._users[user.id] = user
                self._email_index[user.email] = user.id
                imported_count += 1
                
            except Exception as e:
                print(f"Ошибка импорта пользователя: {e}")
        
        print(f"Импортировано {imported_count} пользователей из {filename}")

# Пример использования
def demo_user_management():
    """Демонстрация системы управления пользователями"""
    
    manager = UserManager()
    
    # Создание пользователей разных типов
    regular_user = manager.create_user("john@example.com", "password123", "John Doe")
    admin_user = manager.create_user("admin@example.com", "admin123", "Admin User", 
                                   user_type="admin", department="Security")
    premium_user = manager.create_user("premium@example.com", "premium123", "Premium User",
                                     user_type="premium", subscription_plan="enterprise")
    
    # Аутентификация
    authenticated = manager.authenticate("john@example.com", "password123")
    print(f"Аутентификация успешна: {authenticated is not None}")
    
    # Статистика
    stats = manager.get_statistics()
    print(f"Статистика: {stats}")
    
    # Экспорт/импорт
    manager.export_users("demo_users.json")
    
    return manager

# Запуск демо
if __name__ == "__main__":
    demo_user_management()
```

### Работа с файлами и конфигурацией

#### 1. Управление конфигурацией приложения

```python
import json
import yaml
import os
from pathlib import Path
from typing import Dict, Any, Optional
import configparser

class ConfigManager:
    """Менеджер конфигурации приложения"""
    
    def __init__(self, config_dir: str = "config"):
        self.config_dir = Path(config_dir)
        self.config_dir.mkdir(exist_ok=True)
        self._config_cache: Dict[str, Any] = {}
    
    def load_json_config(self, filename: str) -> Dict[str, Any]:
        """Загрузка JSON конфигурации"""
        config_path = self.config_dir / filename
        
        if not config_path.exists():
            print(f"Конфигурационный файл {config_path} не найден")
            return {}
        
        try:
            with open(config_path, 'r', encoding='utf-8') as f:
                config = json.load(f)
                self._config_cache[filename] = config
                return config
        except json.JSONDecodeError as e:
            print(f"Ошибка парсинга JSON в {filename}: {e}")
            return {}
    
    def save_json_config(self, filename: str, config: Dict[str, Any]):
        """Сохранение JSON конфигурации"""
        config_path = self.config_dir / filename
        
        with open(config_path, 'w', encoding='utf-8') as f:
            json.dump(config, f, ensure_ascii=False, indent=2)
        
        self._config_cache[filename] = config
        print(f"Конфигурация сохранена в {config_path}")
    
    def load_yaml_config(self, filename: str) -> Dict[str, Any]:
        """Загрузка YAML конфигурации"""
        config_path = self.config_dir / filename
        
        if not config_path.exists():
            print(f"Конфигурационный файл {config_path} не найден")
            return {}
        
        try:
            with open(config_path, 'r', encoding='utf-8') as f:
                config = yaml.safe_load(f)
                self._config_cache[filename] = config
                return config
        except yaml.YAMLError as e:
            print(f"Ошибка парсинга YAML в {filename}: {e}")
            return {}
    
    def load_env_config(self, filename: str = ".env") -> Dict[str, str]:
        """Загрузка переменных окружения из файла"""
        config_path = self.config_dir / filename
        env_vars = {}
        
        if config_path.exists():
            with open(config_path, 'r', encoding='utf-8') as f:
                for line in f:
                    line = line.strip()
                    if line and not line.startswith('#'):
                        key, value = line.split('=', 1)
                        env_vars[key.strip()] = value.strip()
        
        return env_vars
    
    def get_database_config(self) -> Dict[str, Any]:
        """Получение конфигурации базы данных"""
        # Сначала пробуем загрузить из файла
        db_config = self.load_json_config("database.json")
        
        # Затем переопределяем переменными окружения
        env_vars = self.load_env_config()
        
        # Приоритет у переменных окружения
        if "DATABASE_URL" in env_vars:
            # Парсинг URL вида postgresql://user:pass@host:port/db
            import re
            match = re.match(r'(\w+)://([^:]+):([^@]+)@([^:]+):(\d+)/(.+)', env_vars["DATABASE_URL"])
            if match:
                db_config.update({
                    "driver": match.group(1),
                    "user": match.group(2),
                    "password": match.group(3),
                    "host": match.group(4),
                    "port": int(match.group(5)),
                    "database": match.group(6)
                })
        
        # Индивидуальные переменные
        for env_key, config_key in [
            ("DB_HOST", "host"),
            ("DB_PORT", "port"),
            ("DB_NAME", "database"),
            ("DB_USER", "user"),
            ("DB_PASSWORD", "password")
        ]:
            if env_key in env_vars:
                db_config[config_key] = env_vars[env_key]
        
        return db_config
    
    def create_default_configs(self):
        """Создание конфигураций по умолчанию"""
        
        # База данных
        default_db_config = {
            "driver": "postgresql",
            "host": "localhost",
            "port": 5432,
            "database": "myapp",
            "user": "postgres",
            "password": "password",
            "pool_size": 10,
            "max_overflow": 20
        }
        self.save_json_config("database.json", default_db_config)
        
        # Redis
        default_redis_config = {
            "host": "localhost",
            "port": 6379,
            "db": 0,
            "password": null,
            "socket_timeout": 30,
            "connection_pool_size": 10
        }
        self.save_json_config("redis.json", default_redis_config)
        
        # API сервер
        default_api_config = {
            "host": "0.0.0.0",
            "port": 8000,
            "debug": false,
            "reload": false,
            "workers": 4,
            "cors": {
                "allowed_origins": ["*"],
                "allowed_methods": ["GET", "POST", "PUT", "DELETE"],
                "allowed_headers": ["*"]
            },
            "rate_limiting": {
                "enabled": true,
                "requests_per_minute": 60,
                "burst": 10
            }
        }
        self.save_json_config("api.json", default_api_config)
        
        # Логирование
        default_logging_config = {
            "version": 1,
            "disable_existing_loggers": false,
            "formatters": {
                "default": {
                    "format": "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
                },
                "json": {
                    "format": "{\"timestamp\":\"%(asctime)s\",\"logger\":\"%(name)s\",\"level\":\"%(levelname)s\",\"message\":\"%(message)s\"}"
                }
            },
            "handlers": {
                "console": {
                    "class": "logging.StreamHandler",
                    "level": "INFO",
                    "formatter": "default",
                    "stream": "ext://sys.stdout"
                },
                "file": {
                    "class": "logging.FileHandler",
                    "level": "DEBUG",
                    "formatter": "json",
                    "filename": "logs/app.log"
                }
            },
            "loggers": {
                "": {
                    "level": "INFO",
                    "handlers": ["console", "file"]
                }
            }
        }
        self.save_json_config("logging.json", default_logging_config)
        
        # .env файл
        env_content = """# Database
DATABASE_URL=postgresql://postgres:password@localhost:5432/myapp
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
DB_USER=postgres
DB_PASSWORD=password

# Redis
REDIS_URL=redis://localhost:6379/0

# API
API_HOST=0.0.0.0
API_PORT=8000
DEBUG=false
SECRET_KEY=your-secret-key-here

# External Services
EMAIL_SMTP_HOST=smtp.gmail.com
EMAIL_SMTP_PORT=587
EMAIL_USERNAME=your-email@gmail.com
EMAIL_PASSWORD=your-app-password

# Monitoring
SENTRY_DSN=your-sentry-dsn-here
ENABLE_METRICS=true
"""
        
        with open(self.config_dir / ".env", 'w', encoding='utf-8') as f:
            f.write(env_content)
        
        print("Созданы конфигурационные файлы по умолчанию")
```

#### 2. Система логирования

```python
import logging
import logging.config
from pathlib import Path
import json
from datetime import datetime
from typing import Dict, Any

class LogManager:
    """Менеджер системы логирования"""
    
    def __init__(self, config_manager: ConfigManager):
        self.config_manager = config_manager
        self.logs_dir = Path("logs")
        self.logs_dir.mkdir(exist_ok=True)
        self._setup_logging()
    
    def _setup_logging(self):
        """Настройка системы логирования"""
        logging_config = self.config_manager.load_json_config("logging.json")
        
        if logging_config:
            # Создаем директории для лог файлов
            for handler_name, handler_config in logging_config.get("handlers", {}).items():
                if "filename" in handler_config:
                    log_file = Path(handler_config["filename"])
                    log_file.parent.mkdir(parents=True, exist_ok=True)
            
            logging.config.dictConfig(logging_config)
        else:
            # Базовая настройка если нет конфигурации
            logging.basicConfig(
                level=logging.INFO,
                format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
                handlers=[
                    logging.StreamHandler(),
                    logging.FileHandler(self.logs_dir / "app.log")
                ]
            )
    
    def get_logger(self, name: str) -> logging.Logger:
        """Получение логгера по имени"""
        return logging.getLogger(name)
    
    def log_api_request(self, method: str, path: str, user_id: int = None, 
                       ip_address: str = None, response_code: int = None,
                       response_time: float = None):
        """Логирование API запроса"""
        logger = self.get_logger("api.requests")
        
        log_data = {
            "method": method,
            "path": path,
            "user_id": user_id,
            "ip_address": ip_address,
            "response_code": response_code,
            "response_time": response_time,
            "timestamp": datetime.utcnow().isoformat()
        }
        
        level = logging.INFO
        if response_code and response_code >= 400:
            level = logging.WARNING if response_code < 500 else logging.ERROR
        
        logger.log(level, json.dumps(log_data))
    
    def log_database_query(self, query: str, duration: float, 
                          rows_affected: int = None, error: str = None):
        """Логирование запросов к базе данных"""
        logger = self.get_logger("database.queries")
        
        log_data = {
            "query": query[:200] + "..." if len(query) > 200 else query,
            "duration": duration,
            "rows_affected": rows_affected,
            "error": error,
            "timestamp": datetime.utcnow().isoformat()
        }
        
        level = logging.INFO if error is None else logging.ERROR
        logger.log(level, json.dumps(log_data))
    
    def log_user_action(self, user_id: int, action: str, details: Dict[str, Any] = None):
        """Логирование действий пользователя"""
        logger = self.get_logger("user.actions")
        
        log_data = {
            "user_id": user_id,
            "action": action,
            "details": details or {},
            "timestamp": datetime.utcnow().isoformat()
        }
        
        logger.info(json.dumps(log_data))
    
    def analyze_logs(self, log_file: str = "logs/app.log") -> Dict[str, Any]:
        """Анализ лог файлов"""
        if not Path(log_file).exists():
            return {"error": "Log file not found"}
        
        stats = {
            "total_lines": 0,
            "by_level": {},
            "by_logger": {},
            "errors": [],
            "api_requests": {
                "total": 0,
                "by_status": {},
                "by_method": {}
            }
        }
        
        with open(log_file, 'r', encoding='utf-8') as f:
            for line in f:
                stats["total_lines"] += 1
                
                try:
                    # Попытка парсинга JSON лога
                    if line.strip().startswith('{'):
                        log_entry = json.loads(line.strip())
                        
                        # API запросы
                        if "method" in log_entry and "path" in log_entry:
                            stats["api_requests"]["total"] += 1
                            
                            method = log_entry["method"]
                            stats["api_requests"]["by_method"][method] = \
                                stats["api_requests"]["by_method"].get(method, 0) + 1
                            
                            if "response_code" in log_entry:
                                code = log_entry["response_code"]
                                stats["api_requests"]["by_status"][code] = \
                                    stats["api_requests"]["by_status"].get(code, 0) + 1
                    
                    # Обычные логи
                    else:
                        # Парсинг стандартного формата логов
                        if " - " in line:
                            parts = line.split(" - ")
                            if len(parts) >= 3:
                                level = parts[2].strip()
                                stats["by_level"][level] = stats["by_level"].get(level, 0) + 1
                                
                                logger_name = parts[1].strip()
                                stats["by_logger"][logger_name] = stats["by_logger"].get(logger_name, 0) + 1
                                
                                if level in ["ERROR", "CRITICAL"]:
                                    stats["errors"].append({
                                        "timestamp": parts[0].strip(),
                                        "logger": logger_name,
                                        "level": level,
                                        "message": " - ".join(parts[3:]).strip()
                                    })
                
                except Exception as e:
                    # Игнорируем ошибки парсинга
                    pass
        
        return stats

# Пример использования системы файлов и конфигурации
def demo_file_management():
    """Демонстрация работы с файлами и конфигурацией"""
    
    # Создание менеджеров
    config_manager = ConfigManager()
    log_manager = LogManager(config_manager)
    
    # Создание конфигураций по умолчанию
    config_manager.create_default_configs()
    
    # Получение конфигурации БД
    db_config = config_manager.get_database_config()
    print(f"Database config: {db_config}")
    
    # Логирование различных событий
    logger = log_manager.get_logger("demo")
    logger.info("Демонстрация системы логирования")
    
    log_manager.log_api_request("GET", "/api/users", user_id=123, response_code=200, response_time=0.045)
    log_manager.log_user_action(123, "login", {"ip": "192.168.1.1", "user_agent": "Mozilla/5.0"})
    
    # Анализ логов
    log_stats = log_manager.analyze_logs()
    print(f"Log statistics: {log_stats}")

if __name__ == "__main__":
    demo_file_management()

---

## 📚 Фаза 2: Backend-специфичные технологии (6-8 недель)

### 🎯 Цели фазы:
- Понять принципы HTTP и REST API
- Освоить современный веб-фреймворк (FastAPI)
- Изучить работу с базами данных и ORM
- Внедрить аутентификацию и безопасность

---

## 📅 Неделя 7-8: HTTP и веб-основы

### Протокол HTTP: фундамент веб-разработки

#### Анатомия HTTP запроса и ответа

```
HTTP Request Structure:
┌─────────────────────────────────────┐
│ GET /api/users/123 HTTP/1.1         │ ← Request Line
├─────────────────────────────────────┤
│ Host: api.example.com               │
│ Authorization: Bearer eyJ0eXAi...   │ ← Headers
│ Content-Type: application/json      │
│ User-Agent: MyApp/1.0               │
├─────────────────────────────────────┤
│                                     │ ← Empty line
├─────────────────────────────────────┤
│ {"filter": "active"}                │ ← Body (optional)
└─────────────────────────────────────┘

HTTP Response Structure:
┌─────────────────────────────────────┐
│ HTTP/1.1 200 OK                     │ ← Status Line
├─────────────────────────────────────┤
│ Content-Type: application/json      │
│ Content-Length: 156                 │ ← Headers
│ Cache-Control: max-age=300          │
│ X-Rate-Limit: 1000                  │
├─────────────────────────────────────┤
│                                     │ ← Empty line
├─────────────────────────────────────┤
│ {"id": 123, "name": "John"}         │ ← Body
└─────────────────────────────────────┘
```

#### HTTP методы в контексте CRUD операций

```python
class HTTPMethodHandler:
    """Обработчик HTTP методов для CRUD операций"""
    
    def __init__(self):
        self.users_db = {}  # Имитация базы данных
        self.next_id = 1
    
    def handle_get(self, path: str, query_params: dict = None) -> tuple[dict, int]:
        """
        GET - Получение данных (READ)
        Идемпотентный, безопасный метод
        """
        if path == "/users":
            # Получение списка пользователей с фильтрацией
            users = list(self.users_db.values())
            
            if query_params:
                # Фильтрация по статусу
                if "status" in query_params:
                    users = [u for u in users if u.get("status") == query_params["status"]]
                
                # Пагинация
                if "page" in query_params and "limit" in query_params:
                    page = int(query_params["page"])
                    limit = int(query_params["limit"])
                    start = (page - 1) * limit
                    users = users[start:start + limit]
            
            return {
                "users": users,
                "total": len(self.users_db),
                "page": query_params.get("page", 1) if query_params else 1
            }, 200
        
        elif path.startswith("/users/"):
            # Получение конкретного пользователя
            user_id = int(path.split("/")[-1])
            user = self.users_db.get(user_id)
            
            if user:
                return {"user": user}, 200
            else:
                return {"error": "User not found"}, 404
        
        else:
            return {"error": "Not found"}, 404
    
    def handle_post(self, path: str, data: dict) -> tuple[dict, int]:
        """
        POST - Создание ресурса (CREATE)
        Не идемпотентный метод
        """
        if path == "/users":
            # Валидация обязательных полей
            required_fields = ["name", "email"]
            for field in required_fields:
                if field not in data:
                    return {"error": f"Missing required field: {field}"}, 400
            
            # Проверка уникальности email
            for user in self.users_db.values():
                if user["email"] == data["email"]:
                    return {"error": "Email already exists"}, 409
            
            # Создание нового пользователя
            user = {
                "id": self.next_id,
                "name": data["name"],
                "email": data["email"],
                "status": data.get("status", "active"),
                "created_at": datetime.now().isoformat()
            }
            
            self.users_db[self.next_id] = user
            self.next_id += 1
            
            return {"user": user, "message": "User created successfully"}, 201
        
        else:
            return {"error": "Method not allowed"}, 405
    
    def handle_put(self, path: str, data: dict) -> tuple[dict, int]:
        """
        PUT - Полное обновление ресурса (UPDATE)
        Идемпотентный метод
        """
        if path.startswith("/users/"):
            user_id = int(path.split("/")[-1])
            
            if user_id not in self.users_db:
                return {"error": "User not found"}, 404
            
            # Валидация обязательных полей для полного обновления
            required_fields = ["name", "email", "status"]
            for field in required_fields:
                if field not in data:
                    return {"error": f"Missing required field: {field}"}, 400
            
            # Проверка уникальности email (кроме текущего пользователя)
            for uid, user in self.users_db.items():
                if uid != user_id and user["email"] == data["email"]:
                    return {"error": "Email already exists"}, 409
            
            # Полное обновление пользователя
            updated_user = {
                "id": user_id,
                "name": data["name"],
                "email": data["email"],
                "status": data["status"],
                "created_at": self.users_db[user_id]["created_at"],
                "updated_at": datetime.now().isoformat()
            }
            
            self.users_db[user_id] = updated_user
            
            return {"user": updated_user, "message": "User updated successfully"}, 200
        
        else:
            return {"error": "Method not allowed"}, 405
    
    def handle_patch(self, path: str, data: dict) -> tuple[dict, int]:
        """
        PATCH - Частичное обновление ресурса
        Не идемпотентный метод
        """
        if path.startswith("/users/"):
            user_id = int(path.split("/")[-1])
            
            if user_id not in self.users_db:
                return {"error": "User not found"}, 404
            
            user = self.users_db[user_id].copy()
            
            # Проверка email на уникальность, если он обновляется
            if "email" in data:
                for uid, existing_user in self.users_db.items():
                    if uid != user_id and existing_user["email"] == data["email"]:
                        return {"error": "Email already exists"}, 409
            
            # Частичное обновление
            for key, value in data.items():
                if key in ["name", "email", "status"]:
                    user[key] = value
            
            user["updated_at"] = datetime.now().isoformat()
            self.users_db[user_id] = user
            
            return {"user": user, "message": "User updated successfully"}, 200
        
        else:
            return {"error": "Method not allowed"}, 405
    
    def handle_delete(self, path: str) -> tuple[dict, int]:
        """
        DELETE - Удаление ресурса
        Идемпотентный метод
        """
        if path.startswith("/users/"):
            user_id = int(path.split("/")[-1])
            
            if user_id not in self.users_db:
                # Идемпотентность: повторное удаление не ошибка
                return {"message": "User already deleted"}, 200
            
            deleted_user = self.users_db.pop(user_id)
            
            return {
                "message": "User deleted successfully",
                "deleted_user": deleted_user
            }, 200
        
        else:
            return {"error": "Method not allowed"}, 405

# Демонстрация работы с HTTP методами
def demo_http_methods():
    """Демонстрация обработки HTTP методов"""
    
    handler = HTTPMethodHandler()
    
    print("=== HTTP Methods Demo ===\n")
    
    # POST - создание пользователей
    print("1. POST /users - Создание пользователей")
    response, status = handler.handle_post("/users", {
        "name": "Alice Smith",
        "email": "alice@example.com"
    })
    print(f"Status: {status}, Response: {response}\n")
    
    response, status = handler.handle_post("/users", {
        "name": "Bob Johnson", 
        "email": "bob@example.com",
        "status": "inactive"
    })
    print(f"Status: {status}, Response: {response}\n")
    
    # GET - получение данных
    print("2. GET /users - Получение всех пользователей")
    response, status = handler.handle_get("/users")
    print(f"Status: {status}, Response: {response}\n")
    
    print("3. GET /users/1 - Получение конкретного пользователя")
    response, status = handler.handle_get("/users/1")
    print(f"Status: {status}, Response: {response}\n")
    
    # PUT - полное обновление
    print("4. PUT /users/1 - Полное обновление пользователя")
    response, status = handler.handle_put("/users/1", {
        "name": "Alice Cooper",
        "email": "alice.cooper@example.com",
        "status": "active"
    })
    print(f"Status: {status}, Response: {response}\n")
    
    # PATCH - частичное обновление
    print("5. PATCH /users/2 - Частичное обновление пользователя")
    response, status = handler.handle_patch("/users/2", {
        "status": "active"
    })
    print(f"Status: {status}, Response: {response}\n")
    
    # DELETE - удаление
    print("6. DELETE /users/2 - Удаление пользователя")
    response, status = handler.handle_delete("/users/2")
    print(f"Status: {status}, Response: {response}\n")
    
    # Проверка идемпотентности DELETE
    print("7. DELETE /users/2 - Повторное удаление (идемпотентность)")
    response, status = handler.handle_delete("/users/2")
    print(f"Status: {status}, Response: {response}\n")

if __name__ == "__main__":
    demo_http_methods()
```

#### Коды состояния HTTP

```python
from enum import IntEnum

class HTTPStatus(IntEnum):
    """Коды состояния HTTP с пояснениями"""
    
    # 2xx Success
    OK = 200                    # GET: Успешное получение данных
    CREATED = 201              # POST: Ресурс успешно создан
    ACCEPTED = 202             # Запрос принят, но обработка не завершена
    NO_CONTENT = 204           # DELETE: Успешно удалено, нет содержимого
    
    # 3xx Redirection
    MOVED_PERMANENTLY = 301    # Ресурс перемещен навсегда
    FOUND = 302               # Временное перенаправление
    NOT_MODIFIED = 304        # Ресурс не изменился (кеширование)
    
    # 4xx Client Error
    BAD_REQUEST = 400         # Неверный формат запроса
    UNAUTHORIZED = 401        # Требуется аутентификация
    FORBIDDEN = 403           # Доступ запрещен
    NOT_FOUND = 404           # Ресурс не найден
    METHOD_NOT_ALLOWED = 405  # Метод не поддерживается
    CONFLICT = 409            # Конфликт (например, дублирование)
    UNPROCESSABLE_ENTITY = 422 # Ошибка валидации
    TOO_MANY_REQUESTS = 429   # Превышен лимит запросов
    
    # 5xx Server Error
    INTERNAL_SERVER_ERROR = 500 # Внутренняя ошибка сервера
    BAD_GATEWAY = 502          # Ошибка шлюза
    SERVICE_UNAVAILABLE = 503  # Сервис недоступен
    GATEWAY_TIMEOUT = 504      # Таймаут шлюза

class APIResponseBuilder:
    """Строитель стандартизированных API ответов"""
    
    @staticmethod
    def success(data=None, message="Success", status_code=HTTPStatus.OK):
        """Успешный ответ"""
        response = {
            "status": "success",
            "message": message,
            "timestamp": datetime.now().isoformat()
        }
        
        if data is not None:
            response["data"] = data
        
        return response, status_code
    
    @staticmethod
    def error(message, status_code=HTTPStatus.BAD_REQUEST, details=None, error_code=None):
        """Ответ с ошибкой"""
        response = {
            "status": "error",
            "message": message,
            "timestamp": datetime.now().isoformat()
        }
        
        if error_code:
            response["error_code"] = error_code
        
        if details:
            response["details"] = details
        
        return response, status_code
    
    @staticmethod
    def validation_error(errors: dict, message="Validation failed"):
        """Ошибка валидации"""
        return APIResponseBuilder.error(
            message=message,
            status_code=HTTPStatus.UNPROCESSABLE_ENTITY,
            details={"validation_errors": errors}
        )
    
    @staticmethod
    def not_found(resource="Resource", resource_id=None):
        """Ресурс не найден"""
        message = f"{resource} not found"
        if resource_id:
            message += f" (ID: {resource_id})"
        
        return APIResponseBuilder.error(
            message=message,
            status_code=HTTPStatus.NOT_FOUND,
            error_code="RESOURCE_NOT_FOUND"
        )
    
    @staticmethod
    def conflict(message, conflicting_field=None):
        """Конфликт ресурсов"""
        details = {}
        if conflicting_field:
            details["conflicting_field"] = conflicting_field
        
        return APIResponseBuilder.error(
            message=message,
            status_code=HTTPStatus.CONFLICT,
            error_code="RESOURCE_CONFLICT",
            details=details if details else None
        )

# Пример использования стандартизированных ответов
def demo_api_responses():
    """Демонстрация стандартизированных API ответов"""
    
    print("=== API Response Standards Demo ===\n")
    
    # Успешные ответы
    print("1. Успешное получение данных:")
    response, status = APIResponseBuilder.success(
        data={"id": 1, "name": "John Doe"},
        message="User retrieved successfully"
    )
    print(f"Status: {status}\nResponse: {json.dumps(response, indent=2)}\n")
    
    print("2. Успешное создание:")
    response, status = APIResponseBuilder.success(
        data={"id": 2, "name": "Jane Smith"},
        message="User created successfully",
        status_code=HTTPStatus.CREATED
    )
    print(f"Status: {status}\nResponse: {json.dumps(response, indent=2)}\n")
    
    # Ошибки клиента
    print("3. Ошибка валидации:")
    response, status = APIResponseBuilder.validation_error({
        "email": "Invalid email format",
        "age": "Must be between 18 and 100"
    })
    print(f"Status: {status}\nResponse: {json.dumps(response, indent=2)}\n")
    
    print("4. Ресурс не найден:")
    response, status = APIResponseBuilder.not_found("User", 999)
    print(f"Status: {status}\nResponse: {json.dumps(response, indent=2)}\n")
    
    print("5. Конфликт ресурсов:")
    response, status = APIResponseBuilder.conflict(
        "Email already exists",
        conflicting_field="email"
    )
    print(f"Status: {status}\nResponse: {json.dumps(response, indent=2)}\n")

if __name__ == "__main__":
    demo_api_responses()
```

### REST архитектура и принципы API дизайна

#### 6 принципов REST

```
REST Architectural Constraints:
│
├── 1. 🔄 Client-Server
│   └── Разделение ответственности между клиентом и сервером
│
├── 2. 🚫 Stateless  
│   └── Сервер не хранит состояние клиента между запросами
│
├── 3. 💾 Cacheable
│   └── Ответы должны быть явно помечены как кешируемые или нет
│
├── 4. 🏗️ Layered System
│   └── Архитектура может состоять из иерархических слоев
│
├── 5. 📝 Code on Demand (optional)
│   └── Сервер может отправлять исполняемый код клиенту
│
└── 6. 🔗 Uniform Interface
    ├── Resource identification (URI)
    ├── Resource manipulation through representations
    ├── Self-descriptive messages  
    └── HATEOAS (Hypermedia as the Engine of Application State)
```

#### Дизайн RESTful API

```python
class RESTfulAPIDesign:
    """Примеры хорошего дизайна RESTful API"""
    
    def __init__(self):
        self.base_url = "https://api.example.com/v1"
    
    def get_resource_urls(self):
        """Правильное именование ресурсов"""
        return {
            # ✅ Правильно: существительные во множественном числе
            "users": f"{self.base_url}/users",
            "posts": f"{self.base_url}/posts", 
            "comments": f"{self.base_url}/comments",
            "categories": f"{self.base_url}/categories",
            
            # ❌ Неправильно: глаголы, единственное число
            # "getUser": f"{self.base_url}/getUser",
            # "user": f"{self.base_url}/user",
            # "createPost": f"{self.base_url}/createPost"
        }
    
    def get_endpoint_examples(self):
        """Примеры правильных эндпоинтов"""
        return {
            # Коллекции
            "GET /users": "Получить список пользователей",
            "POST /users": "Создать нового пользователя",
            
            # Отдельные ресурсы
            "GET /users/123": "Получить пользователя с ID 123",
            "PUT /users/123": "Полностью обновить пользователя 123",
            "PATCH /users/123": "Частично обновить пользователя 123", 
            "DELETE /users/123": "Удалить пользователя 123",
            
            # Вложенные ресурсы
            "GET /users/123/posts": "Получить посты пользователя 123",
            "POST /users/123/posts": "Создать пост для пользователя 123",
            "GET /users/123/posts/456": "Получить пост 456 пользователя 123",
            
            # Фильтрация и поиск
            "GET /users?status=active": "Активные пользователи",
            "GET /users?role=admin&limit=10": "Первые 10 администраторов",
            "GET /posts?author=123&published=true": "Опубликованные посты автора 123",
            
            # Пагинация
            "GET /users?page=2&limit=20": "Вторая страница по 20 пользователей",
            "GET /posts?offset=40&limit=20": "Посты с 41 по 60",
            
            # Сортировка
            "GET /users?sort=created_at": "Сортировка по дате создания",
            "GET /posts?sort=-published_at,title": "По дате публикации убыв., потом по названию",
        }

class HATEOASExample:
    """Пример реализации HATEOAS"""
    
    def get_user_with_links(self, user_id: int):
        """Пользователь с навигационными ссылками"""
        user = {
            "id": user_id,
            "name": "John Doe",
            "email": "john@example.com",
            "status": "active",
            "_links": {
                "self": {"href": f"/users/{user_id}"},
                "edit": {"href": f"/users/{user_id}", "method": "PUT"},
                "delete": {"href": f"/users/{user_id}", "method": "DELETE"},
                "posts": {"href": f"/users/{user_id}/posts"},
                "avatar": {"href": f"/users/{user_id}/avatar"},
                "deactivate": {
                    "href": f"/users/{user_id}/deactivate",
                    "method": "POST",
                    "condition": "status == 'active'"
                }
            }
        }
        
        return user
    
    def get_posts_collection_with_links(self, page=1, limit=10):
        """Коллекция постов с навигацией"""
        posts = [
            {"id": 1, "title": "First Post", "author_id": 123},
            {"id": 2, "title": "Second Post", "author_id": 124}
        ]
        
        # Добавляем ссылки к каждому посту
        for post in posts:
            post["_links"] = {
                "self": {"href": f"/posts/{post['id']}"},
                "author": {"href": f"/users/{post['author_id']}"},
                "comments": {"href": f"/posts/{post['id']}/comments"},
                "edit": {"href": f"/posts/{post['id']}", "method": "PUT"},
                "delete": {"href": f"/posts/{post['id']}", "method": "DELETE"}
            }
        
        # Навигация по коллекции
        collection = {
            "posts": posts,
            "pagination": {
                "page": page,
                "limit": limit,
                "total": 50,
                "pages": 5
            },
            "_links": {
                "self": {"href": f"/posts?page={page}&limit={limit}"},
                "first": {"href": f"/posts?page=1&limit={limit}"},
                "last": {"href": f"/posts?page=5&limit={limit}"},
                "create": {"href": "/posts", "method": "POST"}
            }
        }
        
        # Условные ссылки
        if page > 1:
            collection["_links"]["prev"] = {"href": f"/posts?page={page-1}&limit={limit}"}
        
        if page < 5:
            collection["_links"]["next"] = {"href": f"/posts?page={page+1}&limit={limit}"}
        
        return collection

# Валидация API дизайна
class APIDesignValidator:
    """Валидатор качества дизайна API"""
    
    def __init__(self):
        self.common_mistakes = {
            "verbs_in_urls": [
                "getUsers", "createUser", "updateUser", "deleteUser",
                "fetchPosts", "addComment", "removeTag"
            ],
            "singular_resources": [
                "user", "post", "comment", "category", "tag"
            ],
            "non_standard_methods": [
                "PATCH /users/123/activate",  # Должно быть POST
                "GET /users/create",          # Должно быть GET /users/new (если нужна форма)
                "POST /users/123/delete"      # Должно быть DELETE /users/123
            ]
        }
    
    def validate_endpoint(self, method: str, path: str) -> dict:
        """Валидация эндпоинта на соответствие REST принципам"""
        issues = []
        suggestions = []
        
        # Проверка на глаголы в URL
        path_parts = path.strip('/').split('/')
        for part in path_parts:
            if part in self.common_mistakes["verbs_in_urls"]:
                issues.append(f"Глагол '{part}' в URL нарушает принципы REST")
                suggestions.append(f"Используйте HTTP метод вместо '{part}' в URL")
        
        # Проверка множественного числа для ресурсов
        if len(path_parts) > 0:
            resource = path_parts[0]
            if resource in self.common_mistakes["singular_resources"]:
                issues.append(f"Ресурс '{resource}' должен быть во множественном числе")
                suggestions.append(f"Используйте '{resource}s' вместо '{resource}'")
        
        # Проверка соответствия метода и действия
        if method == "GET" and any(action in path for action in ["create", "add", "update", "delete"]):
            issues.append("GET запрос не должен содержать действия изменения")
            suggestions.append("Используйте POST/PUT/PATCH/DELETE для изменения данных")
        
        # Проверка правильности вложенности
        if len(path_parts) > 4:
            issues.append("Слишком глубокая вложенность ресурсов")
            suggestions.append("Рассмотрите возможность упрощения структуры")
        
        return {
            "endpoint": f"{method} {path}",
            "is_valid": len(issues) == 0,
            "issues": issues,
            "suggestions": suggestions
        }
    
    def suggest_improvements(self, endpoints: list) -> dict:
        """Анализ списка эндпоинтов и предложения улучшений"""
        results = []
        
        for method, path in endpoints:
            validation = self.validate_endpoint(method, path)
            results.append(validation)
        
        # Общая статистика
        total = len(results)
        valid = len([r for r in results if r["is_valid"]])
        
        return {
            "summary": {
                "total_endpoints": total,
                "valid_endpoints": valid,
                "invalid_endpoints": total - valid,
                "compliance_rate": f"{(valid/total)*100:.1f}%" if total > 0 else "0%"
            },
            "detailed_results": results
        }

# Демонстрация дизайна API
def demo_api_design():
    """Демонстрация принципов дизайна RESTful API"""
    
    print("=== RESTful API Design Demo ===\n")
    
    # Примеры хорошего дизайна
    design = RESTfulAPIDesign()
    print("1. Правильные URL ресурсов:")
    for name, url in design.get_resource_urls().items():
        print(f"   {name}: {url}")
    print()
    
    print("2. Примеры эндпоинтов:")
    for endpoint, description in design.get_endpoint_examples().items():
        print(f"   {endpoint:<30} - {description}")
    print()
    
    # HATEOAS пример
    hateoas = HATEOASExample()
    print("3. HATEOAS - Пользователь с навигационными ссылками:")
    user = hateoas.get_user_with_links(123)
    print(json.dumps(user, indent=2))
    print()
    
    # Валидация дизайна
    validator = APIDesignValidator()
    test_endpoints = [
        ("GET", "/users"),           # ✅ Правильно
        ("POST", "/users"),          # ✅ Правильно  
        ("GET", "/getUsers"),        # ❌ Глагол в URL
        ("POST", "/user"),           # ❌ Единственное число
        ("GET", "/users/123/posts/456/comments/789/likes")  # ❌ Слишком глубоко
    ]
    
    print("4. Валидация API дизайна:")
    analysis = validator.suggest_improvements(test_endpoints)
    
    print(f"Общая статистика: {analysis['summary']}")
    print("\nДетальный анализ:")
    for result in analysis['detailed_results']:
        print(f"  {result['endpoint']}: {'✅' if result['is_valid'] else '❌'}")
        for issue in result['issues']:
            print(f"    ⚠️ {issue}")
        for suggestion in result['suggestions']:
            print(f"    💡 {suggestion}")
        print()

if __name__ == "__main__":
    demo_api_design()
```

---

## 📅 Неделя 9-11: Веб-фреймворки - FastAPI

### Знакомство с FastAPI

#### Почему FastAPI?

```
FastAPI Advantages:
│
├── ⚡ Performance
│   ├── Один из самых быстрых Python фреймворков
│   ├── Основан на Starlette (асинхронность)
│   └── Pydantic для валидации (C extensions)
│
├── 🛠️ Developer Experience  
│   ├── Автоматическая документация (Swagger/OpenAPI)
│   ├── Автокомплит в IDE благодаря типам
│   ├── Автоматическая валидация данных
│   └── Минимум boilerplate кода
│
├── 🔄 Modern Python
│   ├── Python 3.7+ с type hints
│   ├── Async/await поддержка
│   ├── Pydantic модели
│   └── Dependency Injection
│
└── 📚 Standards Based
    ├── OpenAPI (Swagger) для документации
    ├── JSON Schema для валидации
    └── OAuth2, JWT из коробки
```

#### Ваше первое FastAPI приложение

```python
# main.py
from fastapi import FastAPI, HTTPException, Depends, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel, EmailStr, validator
from typing import List, Optional, Dict, Any
from datetime import datetime, date
import uvicorn

# Создание экземпляра приложения
app = FastAPI(
    title="User Management API",
    description="Comprehensive API for user management with authentication",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)

# Модели данных с Pydantic
class UserBase(BaseModel):
    """Базовая модель пользователя"""
    name: str
    email: EmailStr
    birth_date: Optional[date] = None
    
    @validator('name')
    def validate_name(cls, v):
        if len(v.strip()) < 2:
            raise ValueError('Имя должно содержать минимум 2 символа')
        return v.strip().title()
    
    @validator('birth_date')
    def validate_birth_date(cls, v):
        if v and v > date.today():
            raise ValueError('Дата рождения не может быть в будущем')
        return v

class UserCreate(UserBase):
    """Модель для создания пользователя"""
    password: str
    
    @validator('password')
    def validate_password(cls, v):
        if len(v) < 8:
            raise ValueError('Пароль должен содержать минимум 8 символов')
        if not any(c.isupper() for c in v):
            raise ValueError('Пароль должен содержать заглавную букву')
        if not any(c.islower() for c in v):
            raise ValueError('Пароль должен содержать строчную букву')
        if not any(c.isdigit() for c in v):
            raise ValueError('Пароль должен содержать цифру')
        return v

class UserUpdate(BaseModel):
    """Модель для обновления пользователя"""
    name: Optional[str] = None
    email: Optional[EmailStr] = None
    birth_date: Optional[date] = None
    is_active: Optional[bool] = None

class UserResponse(UserBase):
    """Модель ответа с данными пользователя"""
    id: int
    is_active: bool
    created_at: datetime
    updated_at: Optional[datetime] = None
    
    class Config:
        # Позволяет использовать ORM объекты
        from_attributes = True

class LoginRequest(BaseModel):
    """Модель для входа в систему"""
    email: EmailStr
    password: str

class TokenResponse(BaseModel):
    """Модель ответа с токеном"""
    access_token: str
    token_type: str = "bearer"
    expires_in: int = 3600

# Имитация базы данных
users_db: Dict[int, Dict[str, Any]] = {}
next_user_id = 1

# Имитация хранения токенов
active_tokens: Dict[str, int] = {}  # token -> user_id

# Утилиты для работы с паролями
import hashlib
import secrets

def hash_password(password: str) -> str:
    """Хеширование пароля"""
    salt = "your-secret-salt"
    return hashlib.sha256((password + salt).encode()).hexdigest()

def verify_password(password: str, hashed: str) -> bool:
    """Проверка пароля"""
    return hash_password(password) == hashed

def generate_token() -> str:
    """Генерация простого токена"""
    return secrets.token_urlsafe(32)

# Dependency injection для аутентификации
security = HTTPBearer()

def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)) -> Dict[str, Any]:
    """Получение текущего пользователя по токену"""
    token = credentials.credentials
    
    if token not in active_tokens:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or expired token",
            headers={"WWW-Authenticate": "Bearer"}
        )
    
    user_id = active_tokens[token]
    user = users_db.get(user_id)
    
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="User not found"
        )
    
    if not user["is_active"]:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="User account is disabled"
        )
    
    return user

# API эндпоинты

@app.get("/", summary="Root endpoint")
async def root():
    """Корневой эндпоинт"""
    return {
        "message": "User Management API",
        "version": "1.0.0",
        "docs": "/docs",
        "redoc": "/redoc"
    }

@app.post("/auth/register", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def register_user(user_data: UserCreate):
    """
    Регистрация нового пользователя
    
    - **name**: Имя пользователя (минимум 2 символа)
    - **email**: Email адрес (должен быть уникальным)
    - **password**: Пароль (минимум 8 символов, должен содержать заглавную букву, строчную букву и цифру)
    - **birth_date**: Дата рождения (опционально)
    """
    global next_user_id
    
    # Проверка уникальности email
    for user in users_db.values():
        if user["email"] == user_data.email:
            raise HTTPException(
                status_code=status.HTTP_409_CONFLICT,
                detail="Email already registered"
            )
    
    # Создание пользователя
    now = datetime.utcnow()
    user = {
        "id": next_user_id,
        "name": user_data.name,
        "email": user_data.email,
        "birth_date": user_data.birth_date,
        "password_hash": hash_password(user_data.password),
        "is_active": True,
        "created_at": now,
        "updated_at": None
    }
    
    users_db[next_user_id] = user
    next_user_id += 1
    
    # Возвращаем пользователя без пароля
    return UserResponse(**user)

@app.post("/auth/login", response_model=TokenResponse)
async def login(login_data: LoginRequest):
    """
    Вход в систему
    
    Возвращает JWT токен для аутентификации
    """
    # Поиск пользователя по email
    user = None
    for u in users_db.values():
        if u["email"] == login_data.email:
            user = u
            break
    
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid email or password"
        )
    
    # Проверка пароля
    if not verify_password(login_data.password, user["password_hash"]):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid email or password"
        )
    
    if not user["is_active"]:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Account is disabled"
        )
    
    # Генерация токена
    token = generate_token()
    active_tokens[token] = user["id"]
    
    return TokenResponse(access_token=token)

@app.get("/users/me", response_model=UserResponse)
async def get_current_user_profile(current_user: Dict = Depends(get_current_user)):
    """Получение профиля текущего пользователя"""
    return UserResponse(**current_user)

@app.get("/users", response_model=List[UserResponse])
async def get_users(
    skip: int = 0,
    limit: int = 100,
    is_active: Optional[bool] = None,
    current_user: Dict = Depends(get_current_user)
):
    """
    Получение списка пользователей
    
    - **skip**: Количество пропускаемых записей (для пагинации)
    - **limit**: Максимальное количество возвращаемых записей
    - **is_active**: Фильтр по статусу активности
    """
    users = list(users_db.values())
    
    # Фильтрация по статусу
    if is_active is not None:
        users = [u for u in users if u["is_active"] == is_active]
    
    # Пагинация
    users = users[skip:skip + limit]
    
    return [UserResponse(**user) for user in users]

@app.get("/users/{user_id}", response_model=UserResponse)
async def get_user(user_id: int, current_user: Dict = Depends(get_current_user)):
    """Получение пользователя по ID"""
    user = users_db.get(user_id)
    
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"User with ID {user_id} not found"
        )
    
    return UserResponse(**user)

@app.put("/users/{user_id}", response_model=UserResponse)
async def update_user(
    user_id: int,
    user_update: UserUpdate,
    current_user: Dict = Depends(get_current_user)
):
    """
    Обновление пользователя
    
    Пользователи могут обновлять только свои данные
    """
    # Проверка прав доступа
    if current_user["id"] != user_id:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="You can only update your own profile"
        )
    
    user = users_db.get(user_id)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"User with ID {user_id} not found"
        )
    
    # Проверка уникальности email при обновлении
    if user_update.email:
        for uid, u in users_db.items():
            if uid != user_id and u["email"] == user_update.email:
                raise HTTPException(
                    status_code=status.HTTP_409_CONFLICT,
                    detail="Email already exists"
                )
    
    # Обновление полей
    update_data = user_update.dict(exclude_unset=True)
    for field, value in update_data.items():
        user[field] = value
    
    user["updated_at"] = datetime.utcnow()
    users_db[user_id] = user
    
    return UserResponse(**user)

@app.delete("/users/{user_id}")
async def delete_user(user_id: int, current_user: Dict = Depends(get_current_user)):
    """Удаление пользователя (деактивация)"""
    # Проверка прав доступа
    if current_user["id"] != user_id:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="You can only delete your own account"
        )
    
    user = users_db.get(user_id)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"User with ID {user_id} not found"
        )
    
    # Деактивация вместо удаления
    user["is_active"] = False
    user["updated_at"] = datetime.utcnow()
    users_db[user_id] = user
    
    return {"message": "User account deactivated successfully"}

@app.post("/auth/logout")
async def logout(credentials: HTTPAuthorizationCredentials = Depends(security)):
    """Выход из системы"""
    token = credentials.credentials
    
    if token in active_tokens:
        del active_tokens[token]
    
    return {"message": "Logged out successfully"}

# Обработчики ошибок
@app.exception_handler(ValueError)
async def validation_exception_handler(request, exc):
    """Обработчик ошибок валидации"""
    return HTTPException(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        detail=str(exc)
    )

# Middleware для логирования запросов
@app.middleware("http")
async def log_requests(request, call_next):
    """Middleware для логирования HTTP запросов"""
    import time
    
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    
    print(f"{request.method} {request.url.path} - {response.status_code} - {process_time:.4f}s")
    
    return response

# Настройка CORS для фронтенда
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000", "http://localhost:8080"],  # React, Vue
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Запуск сервера
if __name__ == "__main__":
    uvicorn.run(
        "main:app",
        host="0.0.0.0",
        port=8000,
        reload=True,  # Автоперезагрузка при изменении кода
        log_level="info"
    )
```

#### Продвинутые возможности FastAPI

```python
# advanced_features.py
from fastapi import FastAPI, BackgroundTasks, UploadFile, File, Form
from fastapi.responses import FileResponse, HTMLResponse
from pydantic import BaseModel
import aiofiles
import asyncio
from pathlib import Path
import shutil

app = FastAPI(title="Advanced FastAPI Features")

# 1. Background Tasks - Фоновые задачи
def send_notification_email(email: str, message: str):
    """Имитация отправки email"""
    import time
    time.sleep(2)  # Имитация медленной операции
    print(f"Email sent to {email}: {message}")

@app.post("/send-notification/")
async def send_notification(
    email: str,
    message: str,
    background_tasks: BackgroundTasks
):
    """Отправка уведомления в фоне"""
    # Добавляем задачу в фон
    background_tasks.add_task(send_notification_email, email, message)
    
    # Сразу возвращаем ответ, не дожидаясь выполнения задачи
    return {"message": "Notification will be sent in background"}

# 2. File Upload - Загрузка файлов
@app.post("/upload-file/")
async def upload_file(file: UploadFile = File(...)):
    """Загрузка файла"""
    upload_dir = Path("uploads")
    upload_dir.mkdir(exist_ok=True)
    
    file_path = upload_dir / file.filename
    
    # Асинхронная запись файла
    async with aiofiles.open(file_path, 'wb') as f:
        content = await file.read()
        await f.write(content)
    
    return {
        "filename": file.filename,
        "content_type": file.content_type,
        "file_size": len(content),
        "saved_path": str(file_path)
    }

@app.post("/upload-multiple/")
async def upload_multiple_files(files: List[UploadFile] = File(...)):
    """Загрузка нескольких файлов"""
    upload_dir = Path("uploads")
    upload_dir.mkdir(exist_ok=True)
    
    uploaded_files = []
    
    for file in files:
        file_path = upload_dir / file.filename
        
        async with aiofiles.open(file_path, 'wb') as f:
            content = await file.read()
            await f.write(content)
        
        uploaded_files.append({
            "filename": file.filename,
            "size": len(content)
        })
    
    return {"uploaded_files": uploaded_files}

# 3. Form Data - Работа с формами
@app.post("/submit-form/")
async def submit_form(
    name: str = Form(...),
    email: str = Form(...),
    age: int = Form(...),
    resume: UploadFile = File(...)
):
    """Обработка формы с файлом"""
    return {
        "name": name,
        "email": email,
        "age": age,
        "resume_filename": resume.filename,
        "resume_size": resume.size
    }

# 4. Response Models - Различные типы ответов
@app.get("/download-file/{filename}")
async def download_file(filename: str):
    """Скачивание файла"""
    file_path = Path("uploads") / filename
    
    if not file_path.exists():
        raise HTTPException(status_code=404, detail="File not found")
    
    return FileResponse(
        path=file_path,
        filename=filename,
        media_type='application/octet-stream'
    )

@app.get("/health", response_class=HTMLResponse)
async def health_check():
    """Health check с HTML ответом"""
    html_content = """
    <html>
        <head>
            <title>Health Check</title>
        </head>
        <body>
            <h1>Service is running! ✅</h1>
            <p>All systems operational</p>
        </body>
    </html>
    """
    return html_content

# 5. Dependency Injection - Продвинутые зависимости
from fastapi import Depends

class DatabaseConnection:
    """Имитация подключения к БД"""
    def __init__(self):
        self.is_connected = True
    
    def execute_query(self, query: str):
        return f"Executed: {query}"

class UserService:
    """Сервис для работы с пользователями"""
    def __init__(self, db: DatabaseConnection):
        self.db = db
    
    def get_user_stats(self, user_id: int):
        query = f"SELECT stats FROM users WHERE id = {user_id}"
        return self.db.execute_query(query)

# Dependency providers
def get_database():
    """Поставщик подключения к БД"""
    return DatabaseConnection()

def get_user_service(db: DatabaseConnection = Depends(get_database)):
    """Поставщик сервиса пользователей"""
    return UserService(db)

@app.get("/users/{user_id}/stats")
async def get_user_stats(
    user_id: int,
    user_service: UserService = Depends(get_user_service)
):
    """Получение статистики пользователя через DI"""
    stats = user_service.get_user_stats(user_id)
    return {"user_id": user_id, "stats": stats}

# 6. WebSocket support - Веб-сокеты
from fastapi import WebSocket, WebSocketDisconnect

class ConnectionManager:
    """Менеджер WebSocket соединений"""
    def __init__(self):
        self.active_connections: List[WebSocket] = []
    
    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)
    
    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)
    
    async def send_personal_message(self, message: str, websocket: WebSocket):
        await websocket.send_text(message)
    
    async def broadcast(self, message: str):
        for connection in self.active_connections:
            await connection.send_text(message)

manager = ConnectionManager()

@app.websocket("/ws/{client_id}")
async def websocket_endpoint(websocket: WebSocket, client_id: int):
    """WebSocket эндпоинт для реального времени"""
    await manager.connect(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            await manager.send_personal_message(f"You wrote: {data}", websocket)
            await manager.broadcast(f"Client #{client_id} says: {data}")
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast(f"Client #{client_id} left the chat")

# 7. Middleware - Кастомные middleware
import time
from starlette.middleware.base import BaseHTTPMiddleware

class ProcessTimeMiddleware(BaseHTTPMiddleware):
    """Middleware для измерения времени обработки"""
    async def dispatch(self, request, call_next):
        start_time = time.time()
        response = await call_next(request)
        process_time = time.time() - start_time
        response.headers["X-Process-Time"] = str(process_time)
        return response

app.add_middleware(ProcessTimeMiddleware)

# 8. Custom Exception Handlers
class CustomException(Exception):
    def __init__(self, name: str):
        self.name = name

@app.exception_handler(CustomException)
async def custom_exception_handler(request, exc):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something wrong"}
    )

@app.get("/trigger-error")
async def trigger_error():
    """Тестирование кастомного обработчика ошибок"""
    raise CustomException(name="test error")
```

---

## 📅 Неделя 12-14: Базы данных и ORM

### Основы реляционных баз данных

#### Структура базы данных для backend приложения

```sql
-- database_schema.sql
-- Создание схемы базы данных для системы управления пользователями

-- 1. Таблица пользователей
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(100) NOT NULL,
    birth_date DATE,
    is_active BOOLEAN DEFAULT TRUE,
    is_verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP WITH TIME ZONE
);

-- 2. Таблица ролей
CREATE TABLE roles (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 3. Связь пользователей и ролей (многие ко многим)
CREATE TABLE user_roles (
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    role_id INTEGER REFERENCES roles(id) ON DELETE CASCADE,
    assigned_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    assigned_by INTEGER REFERENCES users(id),
    PRIMARY KEY (user_id, role_id)
);

-- 4. Таблица постов
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    author_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    is_published BOOLEAN DEFAULT FALSE,
    published_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    views_count INTEGER DEFAULT 0,
    likes_count INTEGER DEFAULT 0
);

-- 5. Таблица комментариев
CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    post_id INTEGER REFERENCES posts(id) ON DELETE CASCADE,
    author_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    parent_id INTEGER REFERENCES comments(id) ON DELETE CASCADE, -- для вложенных комментариев
    is_approved BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 6. Таблица категорий
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    parent_id INTEGER REFERENCES categories(id) ON DELETE SET NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 7. Связь постов и категорий
CREATE TABLE post_categories (
    post_id INTEGER REFERENCES posts(id) ON DELETE CASCADE,
    category_id INTEGER REFERENCES categories(id) ON DELETE CASCADE,
    PRIMARY KEY (post_id, category_id)
);

-- 8. Таблица лайков
CREATE TABLE likes (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    post_id INTEGER REFERENCES posts(id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, post_id) -- Один пользователь не может лайкнуть пост дважды
);

-- 9. Индексы для оптимизации
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_active ON users(is_active);
CREATE INDEX idx_posts_author ON posts(author_id);
CREATE INDEX idx_posts_published ON posts(is_published, published_at);
CREATE INDEX idx_comments_post ON comments(post_id);
CREATE INDEX idx_comments_author ON comments(author_id);
CREATE INDEX idx_likes_user_post ON likes(user_id, post_id);

-- 10. Функция для автоматического обновления updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$ language plpgsql;

-- Триггеры для автоматического обновления timestamps
CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_posts_updated_at BEFORE UPDATE ON posts
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_comments_updated_at BEFORE UPDATE ON comments
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Вставка начальных данных
INSERT INTO roles (name, description) VALUES
('admin', 'Полный доступ к системе'),
('moderator', 'Модерация контента'),
('user', 'Обычный пользователь');

INSERT INTO categories (name, slug, description) VALUES
('Технологии', 'tech', 'Статьи о технологиях'),
('Программирование', 'programming', 'Статьи о программировании'),
('Python', 'python', 'Статьи о Python'),
('JavaScript', 'javascript', 'Статьи о JavaScript');
```

#### Диаграмма связей базы данных

```
Database Schema Relationships:
│
├── 👤 users (1) ──────────── (M) user_roles (M) ──────────── (1) 🔑 roles
│    │                                                            
│    ├── (1) ──────────── (M) posts ──────────── (M) post_categories (M) ──── (1) 📂 categories
│    │                      │                                                      │
│    │                      └── (1) ──────────── (M) comments                     └── (self-reference)
│    │                           │                  │
│    │                           └── (M) likes ─────┘
│    │                                │
│    └─────────────────────────────────┘
│
└── Key Relationships:
    ├── users ↔ roles: Many-to-Many (user_roles)
    ├── users → posts: One-to-Many (author)
    ├── users → comments: One-to-Many (author)  
    ├── users → likes: One-to-Many
    ├── posts ↔ categories: Many-to-Many (post_categories)
    ├── posts → comments: One-to-Many
    ├── posts → likes: One-to-Many
    └── comments → comments: Self-reference (nested)
```

### SQLAlchemy ORM: Современный подход к базам данных

#### Настройка SQLAlchemy с FastAPI

```python
# database.py
from sqlalchemy import create_engine, Column, Integer, String, Text, Boolean, DateTime, ForeignKey, Table
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, relationship, Session
from sqlalchemy.sql import func
from datetime import datetime
from typing import Generator
import os

# Конфигурация базы данных
DATABASE_URL = os.getenv(
    "DATABASE_URL", 
    "postgresql://postgres:password@localhost:5432/myapp"
)

# Создание движка базы данных
engine = create_engine(
    DATABASE_URL,
    pool_size=10,           # Размер пула соединений
    max_overflow=20,        # Максимальное переполнение
    pool_pre_ping=True,     # Проверка соединений перед использованием
    echo=True               # Логирование SQL запросов (только для разработки)
)

# Фабрика сессий
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Базовый класс для моделей
Base = declarative_base()

# Dependency для получения сессии БД
def get_db() -> Generator[Session, None, None]:
    """Генератор сессий базы данных"""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# Модели SQLAlchemy
class TimestampMixin:
    """Миксин для автоматических временных меток"""
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), server_default=func.now(), onupdate=func.now())

# Таблица связи пользователей и ролей (многие ко многим)
user_roles = Table(
    'user_roles',
    Base.metadata,
    Column('user_id', Integer, ForeignKey('users.id', ondelete='CASCADE'), primary_key=True),
    Column('role_id', Integer, ForeignKey('roles.id', ondelete='CASCADE'), primary_key=True),
    Column('assigned_at', DateTime(timezone=True), server_default=func.now()),
    Column('assigned_by', Integer, ForeignKey('users.id'))
)

# Таблица связи постов и категорий
post_categories = Table(
    'post_categories',
    Base.metadata,
    Column('post_id', Integer, ForeignKey('posts.id', ondelete='CASCADE'), primary_key=True),
    Column('category_id', Integer, ForeignKey('categories.id', ondelete='CASCADE'), primary_key=True)
)

class User(Base, TimestampMixin):
    """Модель пользователя"""
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String(255), unique=True, index=True, nullable=False)
    password_hash = Column(String(255), nullable=False)
    name = Column(String(100), nullable=False)
    birth_date = Column(DateTime(timezone=True), nullable=True)
    is_active = Column(Boolean, default=True, index=True)
    is_verified = Column(Boolean, default=False)
    last_login = Column(DateTime(timezone=True), nullable=True)
    
    # Relationships
    roles = relationship("Role", secondary=user_roles, back_populates="users")
    posts = relationship("Post", back_populates="author", cascade="all, delete-orphan")
    comments = relationship("Comment", back_populates="author", cascade="all, delete-orphan")
    likes = relationship("Like", back_populates="user", cascade="all, delete-orphan")
    
    def __repr__(self):
        return f"<User(id={self.id}, email='{self.email}', name='{self.name}')>"

class Role(Base, TimestampMixin):
    """Модель роли"""
    __tablename__ = 'roles'
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(50), unique=True, nullable=False)
    description = Column(Text, nullable=True)
    
    # Relationships
    users = relationship("User", secondary=user_roles, back_populates="roles")

class Category(Base, TimestampMixin):
    """Модель категории"""
    __tablename__ = 'categories'
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), unique=True, nullable=False)
    slug = Column(String(100), unique=True, nullable=False)
    description = Column(Text, nullable=True)
    parent_id = Column(Integer, ForeignKey('categories.id'), nullable=True)
    
    # Self-referential relationship для вложенных категорий
    parent = relationship("Category", remote_side=[id], back_populates="children")
    children = relationship("Category", back_populates="parent")
    
    # Relationship с постами
    posts = relationship("Post", secondary=post_categories, back_populates="categories")

class Post(Base, TimestampMixin):
    """Модель поста"""
    __tablename__ = 'posts'
    
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(255), nullable=False)
    content = Column(Text, nullable=False)
    author_id = Column(Integer, ForeignKey('users.id', ondelete='CASCADE'), nullable=False)
    is_published = Column(Boolean, default=False, index=True)
    published_at = Column(DateTime(timezone=True), nullable=True)
    views_count = Column(Integer, default=0)
    likes_count = Column(Integer, default=0)
    
    # Relationships
    author = relationship("User", back_populates="posts")
    categories = relationship("Category", secondary=post_categories, back_populates="posts")
    comments = relationship("Comment", back_populates="post", cascade="all, delete-orphan")
    likes = relationship("Like", back_populates="post", cascade="all, delete-orphan")
    
    def __repr__(self):
        return f"<Post(id={self.id}, title='{self.title[:30]}...', author_id={self.author_id})>"

class Comment(Base, TimestampMixin):
    """Модель комментария"""
    __tablename__ = 'comments'
    
    id = Column(Integer, primary_key=True, index=True)
    content = Column(Text, nullable=False)
    post_id = Column(Integer, ForeignKey('posts.id', ondelete='CASCADE'), nullable=False)
    author_id = Column(Integer, ForeignKey('users.id', ondelete='CASCADE'), nullable=False)
    parent_id = Column(Integer, ForeignKey('comments.id', ondelete='CASCADE'), nullable=True)
    is_approved = Column(Boolean, default=True)
    
    # Relationships
    post = relationship("Post", back_populates="comments")
    author = relationship("User", back_populates="comments")
    
    # Self-referential для вложенных комментариев
    parent = relationship("Comment", remote_side=[id], back_populates="replies")
    replies = relationship("Comment", back_populates="parent")

class Like(Base, TimestampMixin):
    """Модель лайка"""
    __tablename__ = 'likes'
    
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey('users.id', ondelete='CASCADE'), nullable=False)
    post_id = Column(Integer, ForeignKey('posts.id', ondelete='CASCADE'), nullable=False)
    
    # Relationships
    user = relationship("User", back_populates="likes")
    post = relationship("Post", back_populates="likes")
    
    __table_args__ = (
        # Уникальный индекс - один пользователь может лайкнуть пост только один раз
        Index('idx_unique_user_post_like', 'user_id', 'post_id', unique=True),
    )

# Создание таблиц
def create_tables():
    """Создание всех таблиц в базе данных"""
    Base.metadata.create_all(bind=engine)
    print("Таблицы созданы успешно")

# Repository pattern для работы с данными
class UserRepository:
    """Репозиторий для работы с пользователями"""
    
    def __init__(self, db: Session):
        self.db = db
    
    def create(self, email: str, password_hash: str, name: str, **kwargs) -> User:
        """Создание нового пользователя"""
        user = User(
            email=email,
            password_hash=password_hash,
            name=name,
            **kwargs
        )
        self.db.add(user)
        self.db.commit()
        self.db.refresh(user)
        return user
    
    def get_by_id(self, user_id: int) -> User:
        """Получение пользователя по ID"""
        return self.db.query(User).filter(User.id == user_id).first()
    
    def get_by_email(self, email: str) -> User:
        """Получение пользователя по email"""
        return self.db.query(User).filter(User.email == email).first()
    
    def get_all(self, skip: int = 0, limit: int = 100, is_active: bool = None) -> List[User]:
        """Получение списка пользователей с фильтрацией"""
        query = self.db.query(User)
        
        if is_active is not None:
            query = query.filter(User.is_active == is_active)
        
        return query.offset(skip).limit(limit).all()
    
    def update(self, user_id: int, **kwargs) -> User:
        """Обновление пользователя"""
        user = self.get_by_id(user_id)
        if user:
            for key, value in kwargs.items():
                if hasattr(user, key):
                    setattr(user, key, value)
            self.db.commit()
            self.db.refresh(user)
        return user
    
    def delete(self, user_id: int) -> bool:
        """Удаление пользователя"""
        user = self.get_by_id(user_id)
        if user:
            self.db.delete(user)
            self.db.commit()
            return True
        return False
    
    def search_by_name(self, name_pattern: str) -> List[User]:
        """Поиск пользователей по имени"""
        return self.db.query(User).filter(
            User.name.ilike(f"%{name_pattern}%")
        ).all()
    
    def get_with_roles(self, user_id: int) -> User:
        """Получение пользователя с ролями"""
        return self.db.query(User).options(
            joinedload(User.roles)
        ).filter(User.id == user_id).first()

class PostRepository:
    """Репозиторий для работы с постами"""
    
    def __init__(self, db: Session):
        self.db = db
    
    def create(self, title: str, content: str, author_id: int, **kwargs) -> Post:
        """Создание нового поста"""
        post = Post(
            title=title,
            content=content,
            author_id=author_id,
            **kwargs
        )
        self.db.add(post)
        self.db.commit()
        self.db.refresh(post)
        return post
    
    def get_by_id(self, post_id: int) -> Post:
        """Получение поста по ID с автором"""
        return self.db.query(Post).options(
            joinedload(Post.author),
            joinedload(Post.categories)
        ).filter(Post.id == post_id).first()
    
    def get_published(self, skip: int = 0, limit: int = 100) -> List[Post]:
        """Получение опубликованных постов"""
        return self.db.query(Post).options(
            joinedload(Post.author)
        ).filter(
            Post.is_published == True
        ).order_by(
            Post.published_at.desc()
        ).offset(skip).limit(limit).all()
    
    def get_by_author(self, author_id: int, skip: int = 0, limit: int = 100) -> List[Post]:
        """Получение постов автора"""
        return self.db.query(Post).filter(
            Post.author_id == author_id
        ).order_by(
            Post.created_at.desc()
        ).offset(skip).limit(limit).all()
    
    def get_by_category(self, category_id: int, skip: int = 0, limit: int = 100) -> List[Post]:
        """Получение постов по категории"""
        return self.db.query(Post).join(
            post_categories
        ).filter(
            post_categories.c.category_id == category_id,
            Post.is_published == True
        ).order_by(
            Post.published_at.desc()
        ).offset(skip).limit(limit).all()
    
    def search(self, query: str, skip: int = 0, limit: int = 100) -> List[Post]:
        """Поиск постов по заголовку и содержимому"""
        search_pattern = f"%{query}%"
        return self.db.query(Post).filter(
            (Post.title.ilike(search_pattern)) |
            (Post.content.ilike(search_pattern)),
            Post.is_published == True
        ).order_by(
            Post.published_at.desc()
        ).offset(skip).limit(limit).all()
    
    def increment_views(self, post_id: int):
        """Увеличение счетчика просмотров"""
        self.db.query(Post).filter(Post.id == post_id).update({
            Post.views_count: Post.views_count + 1
        })
        self.db.commit()
    
    def get_popular(self, days: int = 7, limit: int = 10) -> List[Post]:
        """Получение популярных постов за период"""
        from datetime import datetime, timedelta
        
        date_threshold = datetime.utcnow() - timedelta(days=days)
        
        return self.db.query(Post).filter(
            Post.published_at >= date_threshold,
            Post.is_published == True
        ).order_by(
            Post.likes_count.desc(),
            Post.views_count.desc()
        ).limit(limit).all()

# Сервисный слой для бизнес-логики
class UserService:
    """Сервис для работы с пользователями"""
    
    def __init__(self, db: Session):
        self.user_repo = UserRepository(db)
        self.db = db
    
    def create_user(self, email: str, password: str, name: str, **kwargs) -> User:
        """Создание пользователя с валидацией"""
        # Проверка уникальности email
        existing_user = self.user_repo.get_by_email(email)
        if existing_user:
            raise ValueError("Email уже используется")
        
        # Хеширование пароля
        password_hash = self._hash_password(password)
        
        # Создание пользователя
        user = self.user_repo.create(
            email=email,
            password_hash=password_hash,
            name=name,
            **kwargs
        )
        
        # Назначение роли по умолчанию
        self._assign_default_role(user)
        
        return user
    
    def authenticate(self, email: str, password: str) -> User:
        """Аутентификация пользователя"""
        user = self.user_repo.get_by_email(email)
        
        if not user:
            raise ValueError("Неверный email или пароль")
        
        if not self._verify_password(password, user.password_hash):
            raise ValueError("Неверный email или пароль")
        
        if not user.is_active:
            raise ValueError("Аккаунт заблокирован")
        
        # Обновление времени последнего входа
        self.user_repo.update(user.id, last_login=datetime.utcnow())
        
        return user
    
    def change_password(self, user_id: int, old_password: str, new_password: str):
        """Смена пароля"""
        user = self.user_repo.get_by_id(user_id)
        
        if not user:
            raise ValueError("Пользователь не найден")
        
        if not self._verify_password(old_password, user.password_hash):
            raise ValueError("Неверный текущий пароль")
        
        new_password_hash = self._hash_password(new_password)
        self.user_repo.update(user_id, password_hash=new_password_hash)
    
    def assign_role(self, user_id: int, role_name: str):
        """Назначение роли пользователю"""
        user = self.user_repo.get_by_id(user_id)
        role = self.db.query(Role).filter(Role.name == role_name).first()
        
        if not user:
            raise ValueError("Пользователь не найден")
        if not role:
            raise ValueError("Роль не найдена")
        
        if role not in user.roles:
            user.roles.append(role)
            self.db.commit()
    
    def _hash_password(self, password: str) -> str:
        """Хеширование пароля"""
        import hashlib
        salt = "your-secret-salt"
        return hashlib.sha256((password + salt).encode()).hexdigest()
    
    def _verify_password(self, password: str, hash: str) -> bool:
        """Проверка пароля"""
        return self._hash_password(password) == hash
    
    def _assign_default_role(self, user: User):
        """Назначение роли по умолчанию"""
        default_role = self.db.query(Role).filter(Role.name == "user").first()
        if default_role:
            user.roles.append(default_role)
            self.db.commit()

class PostService:
    """Сервис для работы с постами"""
    
    def __init__(self, db: Session):
        self.post_repo = PostRepository(db)
        self.db = db
    
    def create_post(self, title: str, content: str, author_id: int, 
                   category_ids: List[int] = None, publish: bool = False) -> Post:
        """Создание поста"""
        post_data = {
            "title": title,
            "content": content,
            "author_id": author_id
        }
        
        if publish:
            post_data["is_published"] = True
            post_data["published_at"] = datetime.utcnow()
        
        post = self.post_repo.create(**post_data)
        
        # Назначение категорий
        if category_ids:
            categories = self.db.query(Category).filter(
                Category.id.in_(category_ids)
            ).all()
            post.categories.extend(categories)
            self.db.commit()
        
        return post
    
    def publish_post(self, post_id: int, user_id: int) -> Post:
        """Публикация поста"""
        post = self.post_repo.get_by_id(post_id)
        
        if not post:
            raise ValueError("Пост не найден")
        
        if post.author_id != user_id:
            raise ValueError("Недостаточно прав для публикации")
        
        if post.is_published:
            raise ValueError("Пост уже опубликован")
        
        return self.post_repo.update(post_id, 
                                   is_published=True, 
                                   published_at=datetime.utcnow())
    
    def like_post(self, post_id: int, user_id: int) -> bool:
        """Лайк поста"""
        # Проверка существования лайка
        existing_like = self.db.query(Like).filter(
            Like.post_id == post_id,
            Like.user_id == user_id
        ).first()
        
        if existing_like:
            # Убираем лайк
            self.db.delete(existing_like)
            # Уменьшаем счетчик
            self.db.query(Post).filter(Post.id == post_id).update({
                Post.likes_count: Post.likes_count - 1
            })
            self.db.commit()
            return False
        else:
            # Добавляем лайк
            like = Like(post_id=post_id, user_id=user_id)
            self.db.add(like)
            # Увеличиваем счетчик
            self.db.query(Post).filter(Post.id == post_id).update({
                Post.likes_count: Post.likes_count + 1
            })
            self.db.commit()
            return True

# Демонстрация работы с ORM
def demo_sqlalchemy_orm():
    """Демонстрация работы SQLAlchemy ORM"""
    
    # Создание таблиц
    create_tables()
    
    # Получение сессии
    db = SessionLocal()
    
    try:
        print("=== SQLAlchemy ORM Demo ===\n")
        
        # Создание ролей
        admin_role = Role(name="admin", description="Администратор")
        user_role = Role(name="user", description="Обычный пользователь")
        
        db.add_all([admin_role, user_role])
        db.commit()
        
        # Сервисы
        user_service = UserService(db)
        post_service = PostService(db)
        
        # Создание пользователей
        print("1. Создание пользователей:")
        admin = user_service.create_user(
            email="admin@example.com",
            password="admin123",
            name="Admin User"
        )
        print(f"Создан администратор: {admin}")
        
        user = user_service.create_user(
            email="john@example.com", 
            password="password123",
            name="John Doe"
        )
        print(f"Создан пользователь: {user}")
        
        # Назначение роли администратора
        user_service.assign_role(admin.id, "admin")
        print(f"Назначена роль admin пользователю {admin.email}")
        
        # Создание категорий
        print("\n2. Создание категорий:")
        tech_category = Category(name="Технологии", slug="tech")
        python_category = Category(name="Python", slug="python")
        
        db.add_all([tech_category, python_category])
        db.commit()
        print("Созданы категории: Технологии, Python")
        
        # Создание постов
        print("\n3. Создание постов:")
        post1 = post_service.create_post(
            title="Введение в Python",
            content="Python - отличный язык для backend разработки...",
            author_id=user.id,
            category_ids=[python_category.id],
            publish=True
        )
        print(f"Создан пост: {post1}")
        
        post2 = post_service.create_post(
            title="Современные технологии",
            content="Обзор современных технологий в веб-разработке...",
            author_id=admin.id,
            category_ids=[tech_category.id]
        )
        print(f"Создан пост: {post2}")
        
        # Лайки
        print("\n4. Лайки:")
        liked = post_service.like_post(post1.id, admin.id)
        print(f"Администратор {'лайкнул' if liked else 'убрал лайк с'} поста")
        
        # Запросы с объединениями
        print("\n5. Сложные запросы:")
        
        # Получение постов с авторами
        posts_with_authors = db.query(Post).options(
            joinedload(Post.author)
        ).all()
        
        for post in posts_with_authors:
            print(f"Пост '{post.title}' автора {post.author.name}")
        
        # Статистика по пользователям
        user_stats = db.query(
            User.name,
            func.count(Post.id).label('posts_count'),
            func.sum(Post.likes_count).label('total_likes')
        ).outerjoin(Post).group_by(User.id, User.name).all()
        
        print("\nСтатистика пользователей:")
        for name, posts_count, total_likes in user_stats:
            print(f"{name}: {posts_count} постов, {total_likes or 0} лайков")
        
        # Поиск постов
        print("\n6. Поиск:")
        post_repo = PostRepository(db)
        found_posts = post_repo.search("Python")
        print(f"Найдено постов с 'Python': {len(found_posts)}")
        
    finally:
        db.close()

if __name__ == "__main__":
    demo_sqlalchemy_orm()
```

### Миграции базы данных с Alembic

#### Настройка Alembic

```python
# migrations/env.py
from logging.config import fileConfig
from sqlalchemy import engine_from_config, pool
from alembic import context
from database import Base, DATABASE_URL

# Конфигурация Alembic
config = context.config

# Настройка логирования
if config.config_file_name is not None:
    fileConfig(config.config_file_name)

# Метаданные для автогенерации
target_metadata = Base.metadata

def run_migrations_offline() -> None:
    """Запуск миграций в 'offline' режиме"""
    url = DATABASE_URL
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )

    with context.begin_transaction():
        context.run_migrations()

def run_migrations_online() -> None:
    """Запуск миграций в 'online' режиме"""
    configuration = config.get_section(config.config_ini_section)
    configuration['sqlalchemy.url'] = DATABASE_URL
    
    connectable = engine_from_config(
        configuration,
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection, 
            target_metadata=target_metadata
        )

        with context.begin_transaction():
            context.run_migrations()

if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

#### Примеры миграций

```python
# migrations/versions/001_initial_migration.py
"""Initial migration

Revision ID: 001
Revises: 
Create Date: 2024-01-15 10:00:00.000000

"""
from alembic import op
import sqlalchemy as sa

# revision identifiers
revision = '001'
down_revision = None
branch_labels = None
depends_on = None

def upgrade() -> None:
    """Применение миграции"""
    # Создание таблицы пользователей
    op.create_table('users',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('email', sa.String(length=255), nullable=False),
        sa.Column('password_hash', sa.String(length=255), nullable=False),
        sa.Column('name', sa.String(length=100), nullable=False),
        sa.Column('birth_date', sa.DateTime(timezone=True), nullable=True),
        sa.Column('is_active', sa.Boolean(), nullable=True),
        sa.Column('is_verified', sa.Boolean(), nullable=True),
        sa.Column('created_at', sa.DateTime(timezone=True), server_default=sa.text('now()'), nullable=True),
        sa.Column('updated_at', sa.DateTime(timezone=True), nullable=True),
        sa.Column('last_login', sa.DateTime(timezone=True), nullable=True),
        sa.PrimaryKeyConstraint('id')
    )
    op.create_index(op.f('ix_users_email'), 'users', ['email'], unique=True)
    op.create_index(op.f('ix_users_id'), 'users', ['id'], unique=False)
    op.create_index(op.f('ix_users_is_active'), 'users', ['is_active'], unique=False)

def downgrade() -> None:
    """Откат миграции"""
    op.drop_index(op.f('ix_users_is_active'), table_name='users')
    op.drop_index(op.f('ix_users_id'), table_name='users')
    op.drop_index(op.f('ix_users_email'), table_name='users')
    op.drop_table('users')

# migrations/versions/002_add_posts_table.py
"""Add posts table

Revision ID: 002
Revises: 001
Create Date: 2024-01-15 11:00:00.000000

"""
from alembic import op
import sqlalchemy as sa

revision = '002'
down_revision = '001'
branch_labels = None
depends_on = None

def upgrade() -> None:
    # Создание таблицы постов
    op.create_table('posts',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('title', sa.String(length=255), nullable=False),
        sa.Column('content', sa.Text(), nullable=False),
        sa.Column('author_id', sa.Integer(), nullable=False),
        sa.Column('is_published', sa.Boolean(), nullable=True),
        sa.Column('published_at', sa.DateTime(timezone=True), nullable=True),
        sa.Column('views_count', sa.Integer(), nullable=True),
        sa.Column('likes_count', sa.Integer(), nullable=True),
        sa.Column('created_at', sa.DateTime(timezone=True), server_default=sa.text('now()'), nullable=True),
        sa.Column('updated_at', sa.DateTime(timezone=True), nullable=True),
        sa.ForeignKeyConstraint(['author_id'], ['users.id'], ondelete='CASCADE'),
        sa.PrimaryKeyConstraint('id')
    )
    op.create_index(op.f('ix_posts_author_id'), 'posts', ['author_id'], unique=False)
    op.create_index(op.f('ix_posts_is_published'), 'posts', ['is_published'], unique=False)

def downgrade() -> None:
    op.drop_index(op.f('ix_posts_is_published'), table_name='posts')
    op.drop_index(op.f('ix_posts_author_id'), table_name='posts')
    op.drop_table('posts')

# migrations/versions/003_add_user_avatar_column.py
"""Add user avatar column

Revision ID: 003
Revises: 002
Create Date: 2024-01-16 09:00:00.000000

"""
from alembic import op
import sqlalchemy as sa

revision = '003'
down_revision = '002'
branch_labels = None
depends_on = None

def upgrade() -> None:
    # Добавление колонки для аватара
    op.add_column('users', sa.Column('avatar_url', sa.String(length=500), nullable=True))

def downgrade() -> None:
    op.drop_column('users', 'avatar_url')
```

#### Управление миграциями

```bash
# Команды для работы с миграциями

# Инициализация Alembic (один раз)
alembic init migrations

# Автогенерация миграции на основе изменений в моделях
alembic revision --autogenerate -m "Description of changes"

# Применение всех неприменённых миграций
alembic upgrade head

# Применение конкретной миграции
alembic upgrade 002

# Откат к предыдущей миграции
alembic downgrade -1

# Откат к конкретной миграции
alembic downgrade 001

# Просмотр текущей ревизии
alembic current

# Просмотр истории миграций
alembic history

# Проверка разности между моделями и БД
alembic check
```

### Оптимизация запросов и производительность

#### N+1 Problem и его решение

```python
# performance_optimization.py
from sqlalchemy.orm import joinedload, selectinload, subqueryload
from sqlalchemy import func, text
from typing import List
import time

class PerformanceOptimization:
    """Примеры оптимизации производительности"""
    
    def __init__(self, db: Session):
        self.db = db
    
    def demo_n_plus_one_problem(self):
        """Демонстрация проблемы N+1"""
        print("=== N+1 Problem Demo ===\n")
        
        # ❌ Плохо: N+1 проблема
        print("1. N+1 Problem (неэффективно):")
        start_time = time.time()
        
        # 1 запрос для получения постов
        posts = self.db.query(Post).limit(5).all()
        
        # N запросов для получения авторов (по одному на каждый пост)
        for post in posts:
            author_name = post.author.name  # Каждое обращение = отдельный запрос
            print(f"Post: {post.title[:30]}... by {author_name}")
        
        end_time = time.time()
        print(f"Время выполнения N+1: {end_time - start_time:.4f}s\n")
        
        # ✅ Хорошо: Eager Loading
        print("2. Eager Loading (эффективно):")
        start_time = time.time()
        
        # 1 запрос с JOIN для получения постов и авторов
        posts_with_authors = self.db.query(Post).options(
            joinedload(Post.author)
        ).limit(5).all()
        
        for post in posts_with_authors:
            author_name = post.author.name  # Данные уже загружены, нет доп. запросов
            print(f"Post: {post.title[:30]}... by {author_name}")
        
        end_time = time.time()
        print(f"Время выполнения Eager Loading: {end_time - start_time:.4f}s\n")
    
    def demo_loading_strategies(self):
        """Демонстрация стратегий загрузки"""
        print("=== Loading Strategies ===\n")
        
        # 1. Joined Load - JOIN в одном запросе
        print("1. Joined Load:")
        posts = self.db.query(Post).options(
            joinedload(Post.author),
            joinedload(Post.categories)
        ).limit(3).all()
        
        for post in posts:
            print(f"  {post.title} by {post.author.name}")
            print(f"  Categories: {[cat.name for cat in post.categories]}")
        print()
        
        # 2. Select In Load - отдельный запрос с IN
        print("2. Select In Load:")
        posts = self.db.query(Post).options(
            selectinload(Post.comments).selectinload(Comment.author)
        ).limit(3).all()
        
        for post in posts:
            print(f"  {post.title}")
            print(f"  Comments: {len(post.comments)}")
        print()
        
        # 3. Subquery Load - подзапрос
        print("3. Subquery Load:")
        users = self.db.query(User).options(
            subqueryload(User.posts)
        ).limit(3).all()
        
        for user in users:
            print(f"  {user.name}: {len(user.posts)} posts")
        print()
    
    def demo_query_optimization(self):
        """Демонстрация оптимизации запросов"""
        print("=== Query Optimization ===\n")
        
        # 1. Использование индексов
        print("1. Индексированные запросы:")
        
        # ✅ Хорошо: использует индекс на email
        user = self.db.query(User).filter(User.email == "john@example.com").first()
        
        # ✅ Хорошо: использует индекс на is_published
        published_posts = self.db.query(Post).filter(Post.is_published == True).count()
        print(f"Published posts: {published_posts}")
        
        # 2. Оптимизация с помощью SELECT
        print("\n2. Выборочная загрузка полей:")
        
        # Загружаем только нужные поля
        user_emails = self.db.query(User.email, User.name).filter(
            User.is_active == True
        ).all()
        
        for email, name in user_emails:
            print(f"  {name}: {email}")
        
        # 3. Агрегация на уровне БД
        print("\n3. Агрегация:")
        
        # Подсчет на уровне БД, а не в Python
        stats = self.db.query(
            func.count(Post.id).label('total_posts'),
            func.sum(Post.views_count).label('total_views'),
            func.avg(Post.likes_count).label('avg_likes')
        ).filter(Post.is_published == True).first()
        
        print(f"  Total posts: {stats.total_posts}")
        print(f"  Total views: {stats.total_views}")
        print(f"  Average likes: {stats.avg_likes:.2f}")
        
        # 4. Группировка и сортировка
        print("\n4. Группировка по авторам:")
        
        author_stats = self.db.query(
            User.name,
            func.count(Post.id).label('posts_count'),
            func.sum(Post.views_count).label('total_views')
        ).join(Post).group_by(User.id, User.name).order_by(
            func.count(Post.id).desc()
        ).limit(5).all()
        
        for name, posts_count, total_views in author_stats:
            print(f"  {name}: {posts_count} posts, {total_views or 0} views")
    
    def demo_pagination_optimization(self):
        """Оптимизация пагинации"""
        print("\n=== Pagination Optimization ===\n")
        
        # ❌ Плохо: OFFSET для больших значений
        print("1. Неэффективная пагинация (OFFSET):")
        start_time = time.time()
        
        # При больших OFFSET производительность падает
        page_5000 = self.db.query(Post).offset(5000).limit(20).all()
        
        end_time = time.time()
        print(f"OFFSET пагинация: {end_time - start_time:.4f}s")
        
        # ✅ Хорошо: Cursor-based pagination
        print("\n2. Эффективная пагинация (Cursor):")
        start_time = time.time()
        
        # Используем ID как курсор
        last_id = 5000
        cursor_page = self.db.query(Post).filter(
            Post.id > last_id
        ).order_by(Post.id).limit(20).all()
        
        end_time = time.time()
        print(f"Cursor пагинация: {end_time - start_time:.4f}s")
        
        # Пример функции для cursor pagination
        def get_posts_cursor(cursor_id: int = None, limit: int = 20):
            query = self.db.query(Post).filter(Post.is_published == True)
            
            if cursor_id:
                query = query.filter(Post.id > cursor_id)
            
            posts = query.order_by(Post.id).limit(limit + 1).all()
            
            has_more = len(posts) > limit
            if has_more:
                posts = posts[:-1]  # Убираем лишний элемент
            
            next_cursor = posts[-1].id if posts and has_more else None
            
            return {
                "posts": posts,
                "next_cursor": next_cursor,
                "has_more": has_more
            }
        
        result = get_posts_cursor(cursor_id=100, limit=10)
        print(f"Posts loaded: {len(result['posts'])}")
        print(f"Next cursor: {result['next_cursor']}")
        print(f"Has more: {result['has_more']}")
    
    def demo_bulk_operations(self):
        """Массовые операции"""
        print("\n=== Bulk Operations ===\n")
        
        # ❌ Плохо: по одной записи
        print("1. Неэффективные массовые операции:")
        start_time = time.time()
        
        # Обновление по одной записи
        for i in range(100):
            self.db.query(Post).filter(Post.id == i).update({
                Post.views_count: Post.views_count + 1
            })
        
        end_time = time.time()
        print(f"Поодиночке: {end_time - start_time:.4f}s")
        
        # ✅ Хорошо: массовое обновление
        print("\n2. Эффективные массовые операции:")
        start_time = time.time()
        
        # Массовое обновление одним запросом
        self.db.query(Post).filter(
            Post.id.between(1, 100)
        ).update({
            Post.views_count: Post.views_count + 1
        }, synchronize_session=False)
        
        end_time = time.time()
        print(f"Массово: {end_time - start_time:.4f}s")
        
        # Массовая вставка
        print("\n3. Массовая вставка:")
        start_time = time.time()
        
        # Подготовка данных
        bulk_posts = []
        for i in range(1000):
            bulk_posts.append({
                'title': f'Bulk Post {i}',
                'content': f'Content for bulk post {i}',
                'author_id': 1,
                'is_published': False
            })
        
        # Массовая вставка
        self.db.bulk_insert_mappings(Post, bulk_posts)
        self.db.commit()
        
        end_time = time.time()
        print(f"Массовая вставка 1000 записей: {end_time - start_time:.4f}s")
    
    def demo_raw_sql(self):
        """Использование сырого SQL для сложных запросов"""
        print("\n=== Raw SQL for Complex Queries ===\n")
        
        # Сложный аналитический запрос
        complex_query = text("""
            SELECT 
                u.name,
                COUNT(p.id) as posts_count,
                SUM(p.views_count) as total_views,
                AVG(p.likes_count) as avg_likes,
                MAX(p.published_at) as last_post_date,
                RANK() OVER (ORDER BY COUNT(p.id) DESC) as author_rank
            FROM users u
            LEFT JOIN posts p ON u.id = p.author_id AND p.is_published = true
            WHERE u.is_active = true
            GROUP BY u.id, u.name
            HAVING COUNT(p.id) > 0
            ORDER BY posts_count DESC, total_views DESC
            LIMIT 10
        """)
        
        result = self.db.execute(complex_query).fetchall()
        
        print("Топ авторов:")
        for row in result:
            print(f"  #{row.author_rank} {row.name}: {row.posts_count} posts, "
                  f"{row.total_views or 0} views, {row.avg_likes:.1f} avg likes")

# Демонстрация оптимизации
def demo_performance_optimization():
    """Демонстрация техник оптимизации"""
    db = SessionLocal()
    
    try:
        optimizer = PerformanceOptimization(db)
        
        # Создаем тестовые данные если их нет
        if db.query(Post).count() < 10:
            print("Создание тестовых данных...")
            
            # Создаем пользователей
            for i in range(5):
                user = User(
                    email=f"user{i}@example.com",
                    password_hash="hash",
                    name=f"User {i}"
                )
                db.add(user)
            
            db.commit()
            
            # Создаем посты
            users = db.query(User).all()
            for user in users:
                for j in range(10):
                    post = Post(
                        title=f"Post {j} by {user.name}",
                        content=f"Content of post {j}",
                        author_id=user.id,
                        is_published=True,
                        views_count=j * 10,
                        likes_count=j
                    )
                    db.add(post)
            
            db.commit()
            print("Тестовые данные созданы\n")
        
        # Запуск демонстраций
        optimizer.demo_n_plus_one_problem()
        optimizer.demo_loading_strategies()
        optimizer.demo_query_optimization()
        optimizer.demo_pagination_optimization()
        optimizer.demo_bulk_operations()
        optimizer.demo_raw_sql()
        
    finally:
        db.close()

if __name__ == "__main__":
    demo_performance_optimization()
```

---

## 📅 Неделя 15-16: Аутентификация и безопасность

### JWT (JSON Web Tokens) аутентификация

#### Структура и принципы JWT

```
JWT Structure:
┌─────────────────────────────────────────────────────────────┐
│ Header.Payload.Signature                                    │
├─────────────────────────────────────────────────────────────┤
│ eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.                      │ ← Header (Base64)
│ eyJ1c2VyX2lkIjoxMjMsImV4cCI6MTY0MjU4NzYwMH0.              │ ← Payload (Base64)  
│ SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c             │ ← Signature
└─────────────────────────────────────────────────────────────┘

Header (decoded):
{
  "typ": "JWT",
  "alg": "HS256"
}

Payload (decoded):
{
  "user_id": 123,
  "email": "user@example.com", 
  "roles": ["user"],
  "exp": 1642587600,  // expiration timestamp
  "iat": 1642501200,  // issued at timestamp
  "jti": "unique-id"  // JWT ID
}
```

#### Реализация JWT аутентификации

```python
# auth.py
import jwt
from datetime import datetime, timedelta
from passlib.context import CryptContext
from fastapi import HTTPException, status, Depends
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from typing import Optional, List, Dict, Any
import secrets
from enum import Enum

# Конфигурация безопасности
SECRET_KEY = "your-super-secret-key-change-in-production"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30
REFRESH_TOKEN_EXPIRE_DAYS = 7

# Настройка хеширования паролей
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# Схема безопасности
security = HTTPBearer()

class TokenType(Enum):
    ACCESS = "access"
    REFRESH = "refresh"

class AuthenticationError(Exception):
    """Ошибки аутентификации"""
    pass

class AuthorizationError(Exception):
    """Ошибки авторизации"""
    pass

class PasswordManager:
    """Менеджер паролей"""
    
    @staticmethod
    def hash_password(password: str) -> str:
        """Хеширование пароля"""
        return pwd_context.hash(password)
    
    @staticmethod
    def verify_password(plain_password: str, hashed_password: str) -> bool:
        """Проверка пароля"""
        return pwd_context.verify(plain_password, hashed_password)
    
    @staticmethod
    def generate_reset_token() -> str:
        """Генерация токена для сброса пароля"""
        return secrets.token_urlsafe(32)

class JWTManager:
    """Менеджер JWT токенов"""
    
    def __init__(self, secret_key: str = SECRET_KEY, algorithm: str = ALGORITHM):
        self.secret_key = secret_key
        self.algorithm = algorithm
    
    def create_access_token(self, data: Dict[str, Any], expires_delta: Optional[timedelta] = None) -> str:
        """Создание access токена"""
        to_encode = data.copy()
        
        if expires_delta:
            expire = datetime.utcnow() + expires_delta
        else:
            expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
        
        to_encode.update({
            "exp": expire,
            "iat": datetime.utcnow(),
            "type": TokenType.ACCESS.value,
            "jti": secrets.token_urlsafe(16)  # JWT ID для отзыва токенов
        })
        
        encoded_jwt = jwt.encode(to_encode, self.secret_key, algorithm=self.algorithm)
        return encoded_jwt
    
    def create_refresh_token(self, data: Dict[str, Any]) -> str:
        """Создание refresh токена"""
        to_encode = data.copy()
        expire = datetime.utcnow() + timedelta(days=REFRESH_TOKEN_EXPIRE_DAYS)
        
        to_encode.update({
            "exp": expire,
            "iat": datetime.utcnow(),
            "type": TokenType.REFRESH.value,
            "jti": secrets.token_urlsafe(16)
        })
        
        encoded_jwt = jwt.encode(to_encode, self.secret_key, algorithm=self.algorithm)
        return encoded_jwt
    
    def verify_token(self, token: str, token_type: TokenType = TokenType.ACCESS) -> Dict[str, Any]:
        """Проверка и декодирование токена"""
        try:
            payload = jwt.decode(token, self.secret_key, algorithms=[self.algorithm])
            
            # Проверка типа токена
            if payload.get("type") != token_type.value:
                raise AuthenticationError("Invalid token type")
            
            # Проверка обязательных полей
            if "user_id" not in payload:
                raise AuthenticationError("Invalid token payload")
            
            return payload
            
        except jwt.ExpiredSignatureError:
            raise AuthenticationError("Token has expired")
        except jwt.JWTError:
            raise AuthenticationError("Invalid token")
    
    def refresh_access_token(self, refresh_token: str) -> str:
        """Обновление access токена по refresh токену"""
        try:
            payload = self.verify_token(refresh_token, TokenType.REFRESH)
            
            # Создаем новый access токен с теми же данными
            new_token_data = {
                "user_id": payload["user_id"],
                "email": payload.get("email"),
                "roles": payload.get("roles", [])
            }
            
            return self.create_access_token(new_token_data)
            
        except AuthenticationError:
            raise AuthenticationError("Invalid refresh token")

# Хранилище отозванных токенов (в продакшне используйте Redis)
revoked_tokens = set()

class TokenBlacklist:
    """Черный список токенов"""
    
    @staticmethod
    def revoke_token(jti: str):
        """Отзыв токена"""
        revoked_tokens.add(jti)
    
    @staticmethod
    def is_token_revoked(jti: str) -> bool:
        """Проверка отзыва токена"""
        return jti in revoked_tokens
    
    @staticmethod
    def clear_expired_tokens():
        """Очистка истекших токенов (должна запускаться периодически)"""
        # В продакшне это должно быть реализовано с TTL в Redis
        pass

class AuthService:
    """Сервис аутентификации"""
    
    def __init__(self, user_service, jwt_manager: JWTManager = None):
        self.user_service = user_service
        self.jwt_manager = jwt_manager or JWTManager()
        self.password_manager = PasswordManager()
    
    def register_user(self, email: str, password: str, name: str, **kwargs) -> Dict[str, Any]:
        """Регистрация нового пользователя"""
        # Проверка существования пользователя
        existing_user = self.user_service.get_by_email(email)
        if existing_user:
            raise AuthenticationError("Email already registered")
        
        # Валидация пароля
        self._validate_password(password)
        
        # Создание пользователя
        hashed_password = self.password_manager.hash_password(password)
        user = self.user_service.create_user(
            email=email,
            password_hash=hashed_password,
            name=name,
            **kwargs
        )
        
        # Генерация токенов
        token_data = {
            "user_id": user.id,
            "email": user.email,
            "roles": [role.name for role in user.roles]
        }
        
        access_token = self.jwt_manager.create_access_token(token_data)
        refresh_token = self.jwt_manager.create_refresh_token(token_data)
        
        return {
            "user": user,
            "access_token": access_token,
            "refresh_token": refresh_token,
            "token_type": "bearer"
        }
    
    def authenticate_user(self, email: str, password: str) -> Dict[str, Any]:
        """Аутентификация пользователя"""
        # Получение пользователя
        user = self.user_service.get_by_email(email)
        if not user:
            raise AuthenticationError("Invalid email or password")
        
        # Проверка пароля
        if not self.password_manager.verify_password(password, user.password_hash):
            raise AuthenticationError("Invalid email or password")
        
        # Проверка активности аккаунта
        if not user.is_active:
            raise AuthenticationError("Account is disabled")
        
        # Обновление времени последнего входа
        self.user_service.update_last_login(user.id)
        
        # Генерация токенов
        token_data = {
            "user_id": user.id,
            "email": user.email,
            "roles": [role.name for role in user.roles]
        }
        
        access_token = self.jwt_manager.create_access_token(token_data)
        refresh_token = self.jwt_manager.create_refresh_token(token_data)
        
        return {
            "user": user,
            "access_token": access_token,
            "refresh_token": refresh_token,
            "token_type": "bearer"
        }
    
    def refresh_token(self, refresh_token: str) -> Dict[str, Any]:
        """Обновление access токена"""
        try:
            new_access_token = self.jwt_manager.refresh_access_token(refresh_token)
            
            return {
                "access_token": new_access_token,
                "token_type": "bearer"
            }
            
        except AuthenticationError as e:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail=str(e)
            )
    
    def logout_user(self, token: str):
        """Выход пользователя (отзыв токена)"""
        try:
            payload = self.jwt_manager.verify_token(token)
            jti = payload.get("jti")
            
            if jti:
                TokenBlacklist.revoke_token(jti)
            
        except AuthenticationError:
            # Игнорируем ошибки при выходе
            pass
    
    def _validate_password(self, password: str):
        """Валидация пароля"""
        if len(password) < 8:
            raise AuthenticationError("Password must be at least 8 characters long")
        
        if not any(c.isupper() for c in password):
            raise AuthenticationError("Password must contain at least one uppercase letter")
        
        if not any(c.islower() for c in password):
            raise AuthenticationError("Password must contain at least one lowercase letter")
        
        if not any(c.isdigit() for c in password):
            raise AuthenticationError("Password must contain at least one digit")

# Dependency для получения текущего пользователя
def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    user_service=Depends(get_user_service)  # Зависимость от сервиса пользователей
) -> User:
    """Получение текущего пользователя из JWT токена"""
    
    jwt_manager = JWTManager()
    
    try:
        # Извлечение токена
        token = credentials.credentials
        
        # Проверка токена
        payload = jwt_manager.verify_token(token)
        
        # Проверка отзыва токена
        jti = payload.get("jti")
        if jti and TokenBlacklist.is_token_revoked(jti):
            raise AuthenticationError("Token has been revoked")
        
        # Получение пользователя
        user_id = payload.get("user_id")
        user = user_service.get_by_id(user_id)
        
        if not user:
            raise AuthenticationError("User not found")
        
        if not user.is_active:
            raise AuthenticationError("User account is disabled")
        
        return user
        
    except AuthenticationError as e:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail=str(e),
            headers={"WWW-Authenticate": "Bearer"}
        )

# Dependency для проверки ролей
def require_roles(*required_roles: str):
    """Декоратор для проверки ролей пользователя"""
    def role_checker(current_user: User = Depends(get_current_user)) -> User:
        user_roles = [role.name for role in current_user.roles]
        
        if not any(role in user_roles for role in required_roles):
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail=f"Required roles: {', '.join(required_roles)}"
            )
        
        return current_user
    
    return role_checker

# Dependency для проверки конкретных разрешений
def require_permission(permission: str):
    """Декоратор для проверки конкретных разрешений"""
    def permission_checker(current_user: User = Depends(get_current_user)) -> User:
        # Логика проверки разрешений (можно расширить)
        user_roles = [role.name for role in current_user.roles]
        
        # Простая логика разрешений
        permissions = {
            "admin": ["read", "write", "delete", "manage_users"],
            "moderator": ["read", "write", "moderate_content"],
            "user": ["read", "write_own"]
        }
        
        user_permissions = []
        for role in user_roles:
            user_permissions.extend(permissions.get(role, []))
        
        if permission not in user_permissions:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail=f"Permission '{permission}' required"
            )
        
        return current_user
    
    return permission_checker

# Примеры использования в эндпоинтах
from fastapi import FastAPI

app = FastAPI()

@app.post("/auth/register")
async def register(
    email: str,
    password: str,
    name: str,
    auth_service: AuthService = Depends(get_auth_service)
):
    """Регистрация нового пользователя"""
    try:
        result = auth_service.register_user(email, password, name)
        return {
            "message": "User registered successfully",
            "access_token": result["access_token"],
            "refresh_token": result["refresh_token"],
            "token_type": result["token_type"]
        }
    except AuthenticationError as e:
        raise HTTPException(status_code=400, detail=str(e))

@app.post("/auth/login")
async def login(
    email: str,
    password: str,
    auth_service: AuthService = Depends(get_auth_service)
):
    """Вход в систему"""
    try:
        result = auth_service.authenticate_user(email, password)
        return {
            "access_token": result["access_token"],
            "refresh_token": result["refresh_token"],
            "token_type": result["token_type"]
        }
    except AuthenticationError as e:
        raise HTTPException(status_code=401, detail=str(e))

@app.post("/auth/refresh")
async def refresh_token(
    refresh_token: str,
    auth_service: AuthService = Depends(get_auth_service)
):
    """Обновление access токена"""
    return auth_service.refresh_token(refresh_token)

@app.post("/auth/logout")
async def logout(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    auth_service: AuthService = Depends(get_auth_service)
):
    """Выход из системы"""
    auth_service.logout_user(credentials.credentials)
    return {"message": "Logged out successfully"}

@app.get("/users/me")
async def get_my_profile(current_user: User = Depends(get_current_user)):
    """Получение собственного профиля"""
    return {
        "id": current_user.id,
        "email": current_user.email,
        "name": current_user.name,
        "roles": [role.name for role in current_user.roles]
    }

@app.get("/admin/users")
async def get_all_users(
    current_user: User = Depends(require_roles("admin")),
    user_service = Depends(get_user_service)
):
    """Получение всех пользователей (только админы)"""
    users = user_service.get_all()
    return {"users": users}

@app.delete("/admin/users/{user_id}")
async def delete_user(
    user_id: int,
    current_user: User = Depends(require_permission("manage_users")),
    user_service = Depends(get_user_service)
):
    """Удаление пользователя (требует разрешение manage_users)"""
    success = user_service.delete_user(user_id)
    if success:
        return {"message": "User deleted successfully"}
    else:
        raise HTTPException(status_code=404, detail="User not found")
```

### Безопасность API

#### Защита от основных уязвимостей

```python
# security.py
from fastapi import Request, HTTPException, status
from fastapi.middleware.base import BaseHTTPMiddleware
from starlette.responses import JSONResponse
import time
import hashlib
from collections import defaultdict, deque
from datetime import datetime, timedelta
from typing import Dict, Optional
import re
import html

class SecurityConfig:
    """Конфигурация безопасности"""
    
    # Rate Limiting
    RATE_LIMIT_REQUESTS = 100  # запросов
    RATE_LIMIT_WINDOW = 3600   # за час
    RATE_LIMIT_BURST = 10      # burst лимит
    
    # Password Policy
    MIN_PASSWORD_LENGTH = 8
    REQUIRE_UPPERCASE = True
    REQUIRE_LOWERCASE = True
    REQUIRE_DIGITS = True
    REQUIRE_SPECIAL_CHARS = True
    
    # Session Security
    SESSION_TIMEOUT = 3600     # 1 час
    MAX_LOGIN_ATTEMPTS = 5     # максимум попыток входа
    LOCKOUT_DURATION = 1800    # блокировка на 30 минут
    
    # Content Security
    MAX_REQUEST_SIZE = 10 * 1024 * 1024  # 10MB
    ALLOWED_FILE_TYPES = {'.jpg', '.jpeg', '.png', '.gif', '.pdf', '.doc', '.docx'}

class RateLimiter:
    """Ограничитель скорости запросов"""
    
    def __init__(self):
        self.requests = defaultdict(deque)  # IP -> список временных меток
        self.blocked_ips = {}  # IP -> время разблокировки
    
    def is_allowed(self, ip_address: str) -> bool:
        """Проверка разрешения на запрос"""
        now = time.time()
        
        # Проверка блокировки
        if ip_address in self.blocked_ips:
            if now < self.blocked_ips[ip_address]:
                return False
            else:
                del self.blocked_ips[ip_address]
        
        # Очистка старых запросов
        window_start = now - SecurityConfig.RATE_LIMIT_WINDOW
        requests = self.requests[ip_address]
        
        while requests and requests[0] < window_start:
            requests.popleft()
        
        # Проверка лимита
        if len(requests) >= SecurityConfig.RATE_LIMIT_REQUESTS:
            # Блокируем IP
            self.blocked_ips[ip_address] = now + SecurityConfig.LOCKOUT_DURATION
            return False
        
        # Проверка burst лимита
        burst_window = now - 60  # последняя минута
        recent_requests = sum(1 for req_time in requests if req_time > burst_window)
        
        if recent_requests >= SecurityConfig.RATE_LIMIT_BURST:
            return False
        
        # Добавляем текущий запрос
        requests.append(now)
        return True
    
    def get_retry_after(self, ip_address: str) -> Optional[int]:
        """Получение времени до разблокировки"""
        if ip_address in self.blocked_ips:
            return int(self.blocked_ips[ip_address] - time.time())
        return None

class InputValidator:
    """Валидатор входных данных"""
    
    @staticmethod
    def sanitize_html(text: str) -> str:
        """Очистка HTML"""
        return html.escape(text)
    
    @staticmethod
    def validate_email(email: str) -> bool:
        """Валидация email"""
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}
        return re.match(pattern, email) is not None
    
    @staticmethod
    def validate_sql_injection(text: str) -> bool:
        """Проверка на SQL инъекции"""
        dangerous_patterns = [
            r'(\b(SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|EXEC)\b)',
            r'(\bUNION\b.*\bSELECT\b)',
            r'(\b(OR|AND)\b.*[\'"].*[\'"])',
            r'[\'"];.*--',
            r'(\bSCRIPT\b)',
        ]
        
        text_upper = text.upper()
        for pattern in dangerous_patterns:
            if re.search(pattern, text_upper, re.IGNORECASE):
                return False
        return True
    
    @staticmethod
    def validate_xss(text: str) -> bool:
        """Проверка на XSS"""
        dangerous_patterns = [
            r'<script[^>]*>.*?</script>',
            r'javascript:',
            r'on\w+\s*=',
            r'<iframe[^>]*>',
            r'<object[^>]*>',
            r'<embed[^>]*>',
        ]
        
        for pattern in dangerous_patterns:
            if re.search(pattern, text, re.IGNORECASE):
                return False
        return True
    
    @staticmethod
    def validate_file_upload(filename: str, content: bytes) -> tuple[bool, str]:
        """Валидация загружаемых файлов"""
        # Проверка расширения
        file_extension = '.' + filename.split('.')[-1].lower()
        if file_extension not in SecurityConfig.ALLOWED_FILE_TYPES:
            return False, f"File type {file_extension} not allowed"
        
        # Проверка размера
        if len(content) > SecurityConfig.MAX_REQUEST_SIZE:
            return False, "File too large"
        
        # Проверка magic bytes для изображений
        if file_extension in {'.jpg', '.jpeg', '.png', '.gif'}:
            magic_bytes = {
                '.jpg': [b'\xff\xd8\xff'],
                '.jpeg': [b'\xff\xd8\xff'],
                '.png': [b'\x89\x50\x4e\x47'],
                '.gif': [b'\x47\x49\x46\x38']
            }
            
            file_magic = magic_bytes.get(file_extension, [])
            if not any(content.startswith(magic) for magic in file_magic):
                return False, "Invalid file format"
        
        return True, "File is valid"

class LoginAttemptTracker:
    """Отслеживание попыток входа"""
    
    def __init__(self):
        self.attempts = defaultdict(list)  # email -> список попыток
        self.locked_accounts = {}  # email -> время разблокировки
    
    def record_attempt(self, email: str, success: bool) -> bool:
        """Запись попытки входа"""
        now = datetime.utcnow()
        
        # Проверка блокировки
        if email in self.locked_accounts:
            if now < self.locked_accounts[email]:
                return False  # Аккаунт заблокирован
            else:
                del self.locked_accounts[email]
                self.attempts[email] = []
        
        if success:
            # Успешный вход - очищаем попытки
            self.attempts[email] = []
            return True
        else:
            # Неудачная попытка
            self.attempts[email].append(now)
            
            # Очистка старых попыток (старше 1 часа)
            hour_ago = now - timedelta(hours=1)
            self.attempts[email] = [
                attempt for attempt in self.attempts[email] 
                if attempt > hour_ago
            ]
            
            # Проверка блокировки
            if len(self.attempts[email]) >= SecurityConfig.MAX_LOGIN_ATTEMPTS:
                self.locked_accounts[email] = now + timedelta(
                    seconds=SecurityConfig.LOCKOUT_DURATION
                )
                return False
            
            return True
    
    def is_locked(self, email: str) -> bool:
        """Проверка блокировки аккаунта"""
        if email in self.locked_accounts:
            return datetime.utcnow() < self.locked_accounts[email]
        return False
    
    def get_remaining_attempts(self, email: str) -> int:
        """Получение оставшихся попыток"""
        if self.is_locked(email):
            return 0
        
        return max(0, SecurityConfig.MAX_LOGIN_ATTEMPTS - len(self.attempts[email]))

class SecurityMiddleware(BaseHTTPMiddleware):
    """Middleware для безопасности"""
    
    def __init__(self, app):
        super().__init__(app)
        self.rate_limiter = RateLimiter()
        self.input_validator = InputValidator()
    
    async def dispatch(self, request: Request, call_next):
        # Получение IP адреса
        ip_address = self.get_client_ip(request)
        
        # Проверка rate limiting
        if not self.rate_limiter.is_allowed(ip_address):
            retry_after = self.rate_limiter.get_retry_after(ip_address)
            return JSONResponse(
                status_code=429,
                content={"detail": "Too many requests"},
                headers={"Retry-After": str(retry_after)} if retry_after else {}
            )
        
        # Проверка размера запроса
        content_length = request.headers.get("content-length")
        if content_length and int(content_length) > SecurityConfig.MAX_REQUEST_SIZE:
            return JSONResponse(
                status_code=413,
                content={"detail": "Request too large"}
            )
        
        # Добавление security headers
        response = await call_next(request)
        self.add_security_headers(response)
        
        return response
    
    def get_client_ip(self, request: Request) -> str:
        """Получение IP адреса клиента"""
        # Проверяем заголовки прокси
        forwarded_for = request.headers.get("X-Forwarded-For")
        if forwarded_for:
            return forwarded_for.split(",")[0].strip()
        
        real_ip = request.headers.get("X-Real-IP")
        if real_ip:
            return real_ip
        
        return request.client.host
    
    def add_security_headers(self, response):
        """Добавление заголовков безопасности"""
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"
        response.headers["X-XSS-Protection"] = "1; mode=block"
        response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
        response.headers["Content-Security-Policy"] = "default-src 'self'"
        response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"

class SecureAuthService(AuthService):
    """Безопасный сервис аутентификации"""
    
    def __init__(self, user_service, jwt_manager: JWTManager = None):
        super().__init__(user_service, jwt_manager)
        self.login_tracker = LoginAttemptTracker()
        self.input_validator = InputValidator()
    
    def authenticate_user(self, email: str, password: str, ip_address: str = None) -> Dict[str, Any]:
        """Безопасная аутентификация"""
        
        # Валидация входных данных
        if not self.input_validator.validate_email(email):
            raise AuthenticationError("Invalid email format")
        
        if not self.input_validator.validate_sql_injection(email):
            raise AuthenticationError("Invalid input")
        
        # Проверка блокировки аккаунта
        if self.login_tracker.is_locked(email):
            remaining_attempts = self.login_tracker.get_remaining_attempts(email)
            raise AuthenticationError(
                f"Account locked due to too many failed attempts. "
                f"Remaining attempts: {remaining_attempts}"
            )
        
        try:
            # Попытка аутентификации
            result = super().authenticate_user(email, password)
            
            # Запись успешной попытки
            self.login_tracker.record_attempt(email, success=True)
            
            # Логирование успешного входа
            self._log_security_event("LOGIN_SUCCESS", {
                "email": email,
                "ip_address": ip_address,
                "timestamp": datetime.utcnow()
            })
            
            return result
            
        except AuthenticationError as e:
            # Запись неудачной попытки
            self.login_tracker.record_attempt(email, success=False)
            
            # Логирование неудачной попытки
            self._log_security_event("LOGIN_FAILED", {
                "email": email,
                "ip_address": ip_address,
                "error": str(e),
                "remaining_attempts": self.login_tracker.get_remaining_attempts(email),
                "timestamp": datetime.utcnow()
            })
            
            raise e
    
    def register_user(self, email: str, password: str, name: str, **kwargs) -> Dict[str, Any]:
        """Безопасная регистрация"""
        
        # Валидация входных данных
        if not self.input_validator.validate_email(email):
            raise AuthenticationError("Invalid email format")
        
        if not self.input_validator.validate_sql_injection(email):
            raise AuthenticationError("Invalid email")
        
        if not self.input_validator.validate_xss(name):
            raise AuthenticationError("Invalid name")
        
        # Очистка имени от HTML
        name = self.input_validator.sanitize_html(name)
        
        # Усиленная валидация пароля
        self._validate_strong_password(password)
        
        result = super().register_user(email, password, name, **kwargs)
        
        # Логирование регистрации
        self._log_security_event("USER_REGISTERED", {
            "email": email,
            "name": name,
            "timestamp": datetime.utcnow()
        })
        
        return result
    
    def _validate_strong_password(self, password: str):
        """Усиленная валидация пароля"""
        errors = []
        
        if len(password) < SecurityConfig.MIN_PASSWORD_LENGTH:
            errors.append(f"Password must be at least {SecurityConfig.MIN_PASSWORD_LENGTH} characters")
        
        if SecurityConfig.REQUIRE_UPPERCASE and not any(c.isupper() for c in password):
            errors.append("Password must contain at least one uppercase letter")
        
        if SecurityConfig.REQUIRE_LOWERCASE and not any(c.islower() for c in password):
            errors.append("Password must contain at least one lowercase letter")
        
        if SecurityConfig.REQUIRE_DIGITS and not any(c.isdigit() for c in password):
            errors.append("Password must contain at least one digit")
        
        if SecurityConfig.REQUIRE_SPECIAL_CHARS and not any(c in "!@#$%^&*()_+-=[]{}|;:,.<>?" for c in password):
            errors.append("Password must contain at least one special character")
        
        # Проверка на общие пароли
        common_passwords = {
            "password", "123456", "12345678", "qwerty", "abc123",
            "password123", "admin", "letmein", "welcome", "monkey"
        }
        
        if password.lower() in common_passwords:
            errors.append("Password is too common")
        
        if errors:
            raise AuthenticationError("; ".join(errors))
    
    def _log_security_event(self, event_type: str, details: Dict[str, Any]):
        """Логирование событий безопасности"""
        import json
        
        log_entry = {
            "event_type": event_type,
            "details": details,
            "timestamp": datetime.utcnow().isoformat()
        }
        
        # В продакшне отправляйте в SIEM систему
        print(f"SECURITY_LOG: {json.dumps(log_entry)}")

# HTTPS и TLS конфигурация
class HTTPSConfig:
    """Конфигурация HTTPS"""
    
    @staticmethod
    def get_ssl_config():
        """Конфигурация SSL для продакшна"""
        return {
            "ssl_keyfile": "/path/to/private.key",
            "ssl_certfile": "/path/to/certificate.crt",
            "ssl_version": "TLSv1.2",
            "ssl_ciphers": "ECDHE+AESGCM:ECDHE+CHACHA20:DHE+AESGCM:DHE+CHACHA20:!aNULL:!MD5:!DSS",
            "ssl_verify_mode": "CERT_REQUIRED"
        }

# Настройка CORS
from fastapi.middleware.cors import CORSMiddleware

def setup_cors(app: FastAPI, allowed_origins: List[str] = None):
    """Настройка CORS"""
    
    if allowed_origins is None:
        # В продакшне указывайте конкретные домены
        allowed_origins = [
            "https://yourdomain.com",
            "https://www.yourdomain.com"
        ]
    
    app.add_middleware(
        CORSMiddleware,
        allow_origins=allowed_origins,
        allow_credentials=True,
        allow_methods=["GET", "POST", "PUT", "DELETE"],
        allow_headers=["Authorization", "Content-Type"],
        expose_headers=["X-Total-Count"],
        max_age=86400  # 24 hours
    )

# Демонстрация безопасности
def demo_security_features():
    """Демонстрация функций безопасности"""
    
    print("=== Security Features Demo ===\n")
    
    # Rate Limiter
    print("1. Rate Limiter:")
    rate_limiter = RateLimiter()
    
    test_ip = "192.168.1.100"
    for i in range(12):
        allowed = rate_limiter.is_allowed(test_ip)
        print(f"  Request {i+1}: {'✅ Allowed' if allowed else '❌ Blocked'}")
        if not allowed:
            retry_after = rate_limiter.get_retry_after(test_ip)
            print(f"    Retry after: {retry_after} seconds")
            break
    
    # Input Validator
    print("\n2. Input Validation:")
    validator = InputValidator()
    
    test_inputs = [
        ("valid@email.com", "Valid email"),
        ("invalid-email", "Invalid email"),
        ("SELECT * FROM users", "SQL Injection attempt"),
        ("<script>alert('xss')</script>", "XSS attempt"),
        ("Normal text content", "Safe content")
    ]
    
    for test_input, description in test_inputs:
        email_valid = validator.validate_email(test_input)
        sql_safe = validator.validate_sql_injection(test_input)
        xss_safe = validator.validate_xss(test_input)
        
        print(f"  {description}:")
        print(f"    Email valid: {email_valid}")
        print(f"    SQL safe: {sql_safe}")
        print(f"    XSS safe: {xss_safe}")
    
    # Login Attempt Tracker
    print("\n3. Login Attempt Tracking:")
    tracker = LoginAttemptTracker()
    
    test_email = "test@example.com"
    for i in range(7):
        success = i == 6  # Последняя попытка успешная
        allowed = tracker.record_attempt(test_email, success)
        remaining = tracker.get_remaining_attempts(test_email)
        
        print(f"  Attempt {i+1}: {'Success' if success else 'Failed'}")
        print(f"    Allowed: {allowed}")
        print(f"    Remaining attempts: {remaining}")
        
        if tracker.is_locked(test_email):
            print(f"    Account locked!")
            break

if __name__ == "__main__":
    demo_security_features()
```

### OAuth 2.0 и внешние провайдеры

#### Интеграция с Google OAuth

```python
# oauth.py
from fastapi import FastAPI, HTTPException, Depends, Request
from fastapi.responses import RedirectResponse
import httpx
import jwt
from typing import Dict, Optional
import secrets
from urllib.parse import urlencode

class OAuthConfig:
    """Конфигурация OAuth провайдеров"""
    
    # Google OAuth
    GOOGLE_CLIENT_ID = "your-google-client-id"
    GOOGLE_CLIENT_SECRET = "your-google-client-secret"
    GOOGLE_REDIRECT_URI = "http://localhost:8000/auth/google/callback"
    GOOGLE_SCOPE = "openid email profile"
    
    # GitHub OAuth
    GITHUB_CLIENT_ID = "your-github-client-id"
    GITHUB_CLIENT_SECRET = "your-github-client-secret"
    GITHUB_REDIRECT_URI = "http://localhost:8000/auth/github/callback"
    GITHUB_SCOPE = "user:email"

class OAuthProvider:
    """Базовый класс для OAuth провайдеров"""
    
    def __init__(self, client_id: str, client_secret: str, redirect_uri: str):
        self.client_id = client_id
        self.client_secret = client_secret
        self.redirect_uri = redirect_uri
        self.state_storage = {}  # В продакшне используйте Redis
    
    def generate_state(self) -> str:
        """Генерация state для защиты от CSRF"""
        state = secrets.token_urlsafe(32)
        self.state_storage[state] = True
        return state
    
    def verify_state(self, state: str) -> bool:
        """Проверка state"""
        return self.state_storage.pop(state, False)
    
    def get_authorization_url(self) -> str:
        """Получение URL для авторизации"""
        raise NotImplementedError
    
    async def exchange_code_for_token(self, code: str, state: str) -> Dict:
        """Обмен кода на токен"""
        raise NotImplementedError
    
    async def get_user_info(self, access_token: str) -> Dict:
        """Получение информации о пользователе"""
        raise NotImplementedError

class GoogleOAuthProvider(OAuthProvider):
    """Google OAuth провайдер"""
    
    def __init__(self):
        super().__init__(
            OAuthConfig.GOOGLE_CLIENT_ID,
            OAuthConfig.GOOGLE_CLIENT_SECRET,
            OAuthConfig.GOOGLE_REDIRECT_URI
        )
        self.auth_url = "https://accounts.google.com/o/oauth2/v2/auth"
        self.token_url = "https://oauth2.googleapis.com/token"
        self.user_info_url = "https://www.googleapis.com/oauth2/v2/userinfo"
    
    def get_authorization_url(self) -> str:
        """URL для авторизации через Google"""
        state = self.generate_state()
        
        params = {
            "client_id": self.client_id,
            "redirect_uri": self.redirect_uri,
            "scope": OAuthConfig.GOOGLE_SCOPE,
            "response_type": "code",
            "state": state,
            "access_type": "offline",  # Для получения refresh token
            "prompt": "consent"
        }
        
        return f"{self.auth_url}?{urlencode(params)}"
    
    async def exchange_code_for_token(self, code: str, state: str) -> Dict:
        """Обмен кода на токен Google"""
        if not self.verify_state(state):
            raise HTTPException(status_code=400, detail="Invalid state parameter")
        
        token_data = {
            "client_id": self.client_id,
            "client_secret": self.client_secret,
            "code": code,
            "grant_type": "authorization_code",
            "redirect_uri": self.redirect_uri
        }
        
        async with httpx.AsyncClient() as client:
            response = await client.post(self.token_url, data=token_data)
            
            if response.status_code != 200:
                raise HTTPException(status_code=400, detail="Failed to exchange code for token")
            
            return response.json()
    
    async def get_user_info(self, access_token: str) -> Dict:
        """Получение информации о пользователе Google"""
        headers = {"Authorization": f"Bearer {access_token}"}
        
        async with httpx.AsyncClient() as client:
            response = await client.get(self.user_info_url, headers=headers)
            
            if response.status_code != 200:
                raise HTTPException(status_code=400, detail="Failed to get user info")
            
            return response.json()

class GitHubOAuthProvider(OAuthProvider):
    """GitHub OAuth провайдер"""
    
    def __init__(self):
        super().__init__(
            OAuthConfig.GITHUB_CLIENT_ID,
            OAuthConfig.GITHUB_CLIENT_SECRET,
            OAuthConfig.GITHUB_REDIRECT_URI
        )
        self.auth_url = "https://github.com/login/oauth/authorize"
        self.token_url = "https://github.com/login/oauth/access_token"
        self.user_info_url = "https://api.github.com/user"
        self.user_emails_url = "https://api.github.com/user/emails"
    
    def get_authorization_url(self) -> str:
        """URL для авторизации через GitHub"""
        state = self.generate_state()
        
        params = {
            "client_id": self.client_id,
            "redirect_uri": self.redirect_uri,
            "scope": OAuthConfig.GITHUB_SCOPE,
            "state": state
        }
        
        return f"{self.auth_url}?{urlencode(params)}"
    
    async def exchange_code_for_token(self, code: str, state: str) -> Dict:
        """Обмен кода на токен GitHub"""
        if not self.verify_state(state):
            raise HTTPException(status_code=400, detail="Invalid state parameter")
        
        token_data = {
            "client_id": self.client_id,
            "client_secret": self.client_secret,
            "code": code
        }
        
        headers = {"Accept": "application/json"}
        
        async with httpx.AsyncClient() as client:
            response = await client.post(self.token_url, data=token_data, headers=headers)
            
            if response.status_code != 200:
                raise HTTPException(status_code=400, detail="Failed to exchange code for token")
            
            return response.json()
    
    async def get_user_info(self, access_token: str) -> Dict:
        """Получение информации о пользователе GitHub"""
        headers = {"Authorization": f"token {access_token}"}
        
        async with httpx.AsyncClient() as client:
            # Получение основной информации
            user_response = await client.get(self.user_info_url, headers=headers)
            if user_response.status_code != 200:
                raise HTTPException(status_code=400, detail="Failed to get user info")
            
            user_data = user_response.json()
            
            # Получение email (может быть приватным)
            emails_response = await client.get(self.user_emails_url, headers=headers)
            if emails_response.status_code == 200:
                emails = emails_response.json()
                primary_email = next((email["email"] for email in emails if email["primary"]), None)
                if primary_email:
                    user_data["email"] = primary_email
            
            return user_data

class OAuthService:
    """Сервис для работы с OAuth"""
    
    def __init__(self, user_service, auth_service):
        self.user_service = user_service
        self.auth_service = auth_service
        self.google_provider = GoogleOAuthProvider()
        self.github_provider = GitHubOAuthProvider()
    
    async def authenticate_with_google(self, code: str, state: str) -> Dict:
        """Аутентификация через Google"""
        # Обмен кода на токен
        token_data = await self.google_provider.exchange_code_for_token(code, state)
        access_token = token_data["access_token"]
        
        # Получение информации о пользователе
        user_info = await self.google_provider.get_user_info(access_token)
        
        # Поиск или создание пользователя
        user = await self._find_or_create_user(
            email=user_info["email"],
            name=user_info["name"],
            provider="google",
            provider_id=user_info["id"],
            avatar_url=user_info.get("picture")
        )
        
        # Генерация JWT токенов
        return self._generate_tokens(user)
    
    async def authenticate_with_github(self, code: str, state: str) -> Dict:
        """Аутентификация через GitHub"""
        # Обмен кода на токен
        token_data = await self.github_provider.exchange_code_for_token(code, state)
        access_token = token_data["access_token"]
        
        # Получение информации о пользователе
        user_info = await self.github_provider.get_user_info(access_token)
        
        # Поиск или создание пользователя
        user = await self._find_or_create_user(
            email=user_info.get("email"),
            name=user_info.get("name") or user_info["login"],
            provider="github",
            provider_id=str(user_info["id"]),
            avatar_url=user_info.get("avatar_url")
        )
        
        # Генерация JWT токенов
        return self._generate_tokens(user)
    
    async def _find_or_create_user(self, email: str, name: str, provider: str, 
                                 provider_id: str, avatar_url: str = None) -> User:
        """Поиск или создание пользователя OAuth"""
        
        # Поиск существующего пользователя по email
        user = self.user_service.get_by_email(email)
        
        if user:
            # Обновление информации OAuth
            self.user_service.update_oauth_info(
                user.id,
                provider=provider,
                provider_id=provider_id,
                avatar_url=avatar_url
            )
            return user
        else:
            # Создание нового пользователя
            return self.user_service.create_oauth_user(
                email=email,
                name=name,
                provider=provider,
                provider_id=provider_id,
                avatar_url=avatar_url
            )
    
    def _generate_tokens(self, user: User) -> Dict:
        """Генерация JWT токенов"""
        token_data = {
            "user_id": user.id,
            "email": user.email,
            "roles": [role.name for role in user.roles]
        }
        
        access_token = self.auth_service.jwt_manager.create_access_token(token_data)
        refresh_token = self.auth_service.jwt_manager.create_refresh_token(token_data)
        
        return {
            "access_token": access_token,
            "refresh_token": refresh_token,
            "token_type": "bearer",
            "user": user
        }

# OAuth эндпоинты
app = FastAPI()

@app.get("/auth/google")
async def google_auth():
    """Перенаправление на Google OAuth"""
    google_provider = GoogleOAuthProvider()
    auth_url = google_provider.get_authorization_url()
    return RedirectResponse(url=auth_url)

@app.get("/auth/google/callback")
async def google_callback(
    code: str,
    state: str,
    oauth_service: OAuthService = Depends(get_oauth_service)
):
    """Callback для Google OAuth"""
    try:
        result = await oauth_service.authenticate_with_google(code, state)
        
        # В реальном приложении перенаправьте на фронтенд с токенами
        return {
            "message": "Authentication successful",
            "access_token": result["access_token"],
            "refresh_token": result["refresh_token"]
        }
        
    except Exception as e:
        raise HTTPException(status_code=400, detail=str(e))

@app.get("/auth/github")
async def github_auth():
    """Перенаправление на GitHub OAuth"""
    github_provider = GitHubOAuthProvider()
    auth_url = github_provider.get_authorization_url()
    return RedirectResponse(url=auth_url)

@app.get("/auth/github/callback")
async def github_callback(
    code: str,
    state: str,
    oauth_service: OAuthService = Depends(get_oauth_service)
):
    """Callback для GitHub OAuth"""
    try:
        result = await oauth_service.authenticate_with_github(code, state)
        
        return {
            "message": "Authentication successful",
            "access_token": result["access_token"],
            "refresh_token": result["refresh_token"]
        }
        
    except Exception as e:
        raise HTTPException(status_code=400, detail=str(e))

# Расширение модели пользователя для OAuth
class OAuthUser(User):
    """Пользователь с OAuth информацией"""
    
    oauth_provider = Column(String(50), nullable=True)
    oauth_provider_id = Column(String(100), nullable=True)
    avatar_url = Column(String(500), nullable=True)
    is_oauth_user = Column(Boolean, default=False)
    
    __table_args__ = (
        Index('idx_oauth_provider', 'oauth_provider', 'oauth_provider_id', unique=True),
    )

# Демонстрация OAuth
def demo_oauth_flow():
    """Демонстрация OAuth потока"""
    print("=== OAuth Flow Demo ===\n")
    
    # Google OAuth
    print("1. Google OAuth URL generation:")
    google_provider = GoogleOAuthProvider()
    google_url = google_provider.get_authorization_url()
    print(f"Google Auth URL: {google_url}\n")
    
    # GitHub OAuth
    print("2. GitHub OAuth URL generation:")
    github_provider = GitHubOAuthProvider()
    github_url = github_provider.get_authorization_url()
    print(f"GitHub Auth URL: {github_url}\n")
    
    print("3. OAuth Flow Steps:")
    print("   1. User clicks on OAuth login button")
    print("   2. User redirected to provider (Google/GitHub)")
    print("   3. User authorizes application")
    print("   4. Provider redirects back with authorization code")
    print("   5. Backend exchanges code for access token")
    print("   6. Backend fetches user info from provider")
    print("   7. Backend creates/updates user in database")
    print("   8. Backend generates JWT tokens")
    print("   9. User is logged in")

if __name__ == "__main__":
    demo_oauth_flow()
```

---

## 🚀 Практическое задание

Создайте полноценный API для системы управления задачами с использованием всех изученных концепций:

### Требования к проекту:

1. **FastAPI приложение** с автоматической документацией
2. **SQLAlchemy ORM** с PostgreSQL
3. **JWT аутентификация** с ролями
4. **CRUD операции** для пользователей и задач
5. **Валидация данных** с Pydantic
6. **Безопасность**: rate limiting, input validation
7. **Тестирование** с pytest
8. **Docker** контейнеризация

### Структура базы данных:
- Users (пользователи)
- Tasks (задачи) 
- Categories (категории)
- Comments (комментарии к задачам)

### Основные эндпоинты:
```
POST /auth/register     - Регистрация
POST /auth/login        - Вход
GET  /users/me          - Профиль текущего пользователя
GET  /tasks             - Список задач с фильтрацией
POST /tasks             - Создание задачи
PUT  /tasks/{id}        - Обновление задачи
DELETE /tasks/{id}      - Удаление задачи
POST /tasks/{id}/comments - Добавление комментария
```

Этот проект станет основой вашего портфолио и покажет понимание всех ключевых аспектов backend разработки на Python!

---

## 📚 Что дальше?

В следующей фазе мы изучим:
- **Асинхронное программирование** с async/await
- **Кэширование** с Redis
- **Фоновые задачи** с Celery
- **Тестирование** API
- **Мониторинг** и логирование
- **Развертывание** в продакшн

Продолжайте практиковаться и создавать проекты! 🎯