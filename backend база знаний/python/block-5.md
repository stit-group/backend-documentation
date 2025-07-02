# ⚡ Блок 5: Продвинутые темы и оптимизация Python

> **Цель блока:** Освоить продвинутые концепции для написания эффективного и профессионального кода  
> **Длительность:** 6-8 недель  
> **Уровень:** Advanced  

---

## 🎯 Обзор блока

```
Новичок ──► Основы ──► ООП ──► Данные ──► Web ──► 🔥 ПРОДВИНУТЫЙ 🔥 ──► Специализация
                                                      ▲
                                                   Вы здесь!
```

### 📊 Структура обучения

```
Блок 5: Продвинутые темы (6-8 недель)
├── 5.1 Функциональное программирование      (1 неделя)
├── 5.2 Генераторы и итераторы              (1 неделя)  
├── 5.3 Контекстные менеджеры               (0.5 недели)
├── 5.4 Метаклассы и продвинутое ООП        (1.5 недели)
├── 5.5 Многопоточность и параллелизм       (1 неделя)
├── 5.6 Асинхронное программирование        (1.5 недели)
├── 5.7 Профилирование и оптимизация        (1 неделя)
├── 5.8 Паттерны проектирования            (1 неделя)
└── 5.9 Качество кода и инструменты         (0.5 недели)
```

---

## 📚 5.1 Функциональное программирование

### Теория: Основные концепции

```
Функциональное программирование (ФП)
│
├── Функции высшего порядка
├── Неизменяемость данных  
├── Чистые функции
├── Композиция функций
└── Ленивые вычисления
```

#### 🎯 Функции высшего порядка

**Функция высшего порядка** - это функция, которая:
- Принимает другие функции как аргументы
- Возвращает функцию как результат
- Или делает и то, и другое

```python
# ──────────────────────────────────────────────────────────
# Пример 1: Функция принимает другую функцию
# ──────────────────────────────────────────────────────────

def apply_operation(numbers, operation):
    """Применяет операцию к каждому числу в списке"""
    return [operation(num) for num in numbers]

def square(x):
    return x ** 2

def double(x):
    return x * 2

# Использование
numbers = [1, 2, 3, 4, 5]
squared = apply_operation(numbers, square)      # [1, 4, 9, 16, 25]
doubled = apply_operation(numbers, double)      # [2, 4, 6, 8, 10]

# ──────────────────────────────────────────────────────────
# Пример 2: Функция возвращает другую функцию
# ──────────────────────────────────────────────────────────

def create_multiplier(factor):
    """Создает функцию-умножитель"""
    def multiplier(x):
        return x * factor
    return multiplier

# Создаем специализированные функции
double = create_multiplier(2)
triple = create_multiplier(3)

print(double(5))    # 10
print(triple(5))    # 15
```

#### 🔧 Lambda функции

```python
# ──────────────────────────────────────────────────────────
# Lambda - анонимные функции
# ──────────────────────────────────────────────────────────

# Обычная функция
def add(x, y):
    return x + y

# Эквивалентная lambda
add_lambda = lambda x, y: x + y

# Использование в функциях высшего порядка
numbers = [1, 2, 3, 4, 5]

# Квадраты чисел
squares = list(map(lambda x: x**2, numbers))

# Четные числа
evens = list(filter(lambda x: x % 2 == 0, numbers))

# Сумма всех элементов
from functools import reduce
total = reduce(lambda x, y: x + y, numbers)

print(f"Квадраты: {squares}")    # [1, 4, 9, 16, 25]
print(f"Четные: {evens}")        # [2, 4]
print(f"Сумма: {total}")         # 15
```

#### 🧹 Чистые функции и побочные эффекты

```python
# ──────────────────────────────────────────────────────────
# ПЛОХО: Функция с побочными эффектами
# ──────────────────────────────────────────────────────────

counter = 0  # Глобальная переменная

def bad_increment():
    global counter
    counter += 1        # Изменяет глобальное состояние
    print(counter)      # Побочный эффект (вывод)
    return counter

# ──────────────────────────────────────────────────────────
# ХОРОШО: Чистая функция
# ──────────────────────────────────────────────────────────

def pure_increment(value):
    """Чистая функция - всегда возвращает одинаковый результат"""
    return value + 1

# Использование чистых функций
def calculate_statistics(numbers):
    """Чистая функция для вычисления статистики"""
    if not numbers:
        return {"mean": 0, "max": 0, "min": 0}
    
    return {
        "mean": sum(numbers) / len(numbers),
        "max": max(numbers),
        "min": min(numbers),
        "count": len(numbers)
    }

# Композиция чистых функций
def process_data(raw_data):
    cleaned = [x for x in raw_data if x is not None]
    stats = calculate_statistics(cleaned)
    return stats

data = [1, 2, None, 4, 5, None, 7]
result = process_data(data)
print(result)  # {'mean': 3.8, 'max': 7, 'min': 1, 'count': 5}
```

#### 🎪 Замыкания (Closures)

```python
# ──────────────────────────────────────────────────────────
# Замыкания - функции, которые "запоминают" переменные
# ──────────────────────────────────────────────────────────

def create_counter(start=0):
    """Создает счетчик с начальным значением"""
    count = start
    
    def increment(step=1):
        nonlocal count
        count += step
        return count
    
    def decrement(step=1):
        nonlocal count
        count -= step
        return count
    
    def reset():
        nonlocal count
        count = start
        return count
    
    def get_value():
        return count
    
    # Возвращаем словарь с методами
    return {
        'increment': increment,
        'decrement': decrement,
        'reset': reset,
        'value': get_value
    }

# ──────────────────────────────────────────────────────────
# Использование замыкания
# ──────────────────────────────────────────────────────────

counter1 = create_counter(10)
counter2 = create_counter(0)

print(counter1['increment']())     # 11
print(counter1['increment'](5))    # 16
print(counter1['value']())         # 16

print(counter2['increment']())     # 1
print(counter2['value']())         # 1

# Каждый счетчик независим!
```

### 🛠 Практическое задание 5.1

```python
"""
──────────────────────────────────────────────────────────
ЗАДАНИЕ: Создайте функциональный калькулятор
──────────────────────────────────────────────────────────

Требования:
1. Используйте функции высшего порядка
2. Реализуйте композицию функций
3. Все функции должны быть чистыми
4. Используйте замыкания для создания специализированных калькуляторов
"""

# Ваш код здесь...

def create_calculator():
    """Создает функциональный калькулятор"""
    
    # Базовые операции (чистые функции)
    operations = {
        'add': lambda x, y: x + y,
        'subtract': lambda x, y: x - y,
        'multiply': lambda x, y: x * y,
        'divide': lambda x, y: x / y if y != 0 else float('inf'),
        'power': lambda x, y: x ** y
    }
    
    def calculate(operation_name, x, y):
        """Выполняет указанную операцию"""
        if operation_name in operations:
            return operations[operation_name](x, y)
        raise ValueError(f"Неизвестная операция: {operation_name}")
    
    def chain_operations(*ops_and_values):
        """Выполняет цепочку операций"""
        result = ops_and_values[0]
        
        for i in range(1, len(ops_and_values), 2):
            operation = ops_and_values[i]
            value = ops_and_values[i + 1]
            result = calculate(operation, result, value)
        
        return result
    
    return calculate, chain_operations

# Пример использования
calc, chain = create_calculator()

result1 = calc('add', 5, 3)           # 8
result2 = calc('multiply', result1, 2) # 16
result3 = chain(10, 'add', 5, 'multiply', 2, 'subtract', 3)  # 27
```

---

## 🔄 5.2 Генераторы и итераторы

### Теория: Протокол итератора

```
Итерация в Python
│
├── Итерируемые объекты (Iterable)
│   ├── Реализуют __iter__()
│   └── Возвращают итератор
│
├── Итераторы (Iterator) 
│   ├── Реализуют __iter__() и __next__()
│   └── Поддерживают протокол итератора
│
└── Генераторы (Generator)
    ├── Специальный вид итератора
    ├── Используют yield
    └── Ленивые вычисления
```

#### 🎯 Создание итераторов

```python
# ──────────────────────────────────────────────────────────
# Ручное создание итератора
# ──────────────────────────────────────────────────────────

class CountDown:
    """Итератор для обратного отсчета"""
    
    def __init__(self, start):
        self.start = start
        self.current = start
    
    def __iter__(self):
        """Возвращает сам себя (итератор)"""
        return self
    
    def __next__(self):
        """Возвращает следующий элемент"""
        if self.current <= 0:
            raise StopIteration
        
        self.current -= 1
        return self.current + 1

# ──────────────────────────────────────────────────────────
# Использование итератора
# ──────────────────────────────────────────────────────────

countdown = CountDown(5)
for num in countdown:
    print(f"T-{num}")

# T-5
# T-4  
# T-3
# T-2
# T-1

# ──────────────────────────────────────────────────────────
# Итерируемый объект vs Итератор
# ──────────────────────────────────────────────────────────

class NumberSequence:
    """Итерируемый объект - можно использовать многократно"""
    
    def __init__(self, max_num):
        self.max_num = max_num
    
    def __iter__(self):
        """Возвращает НОВЫЙ итератор каждый раз"""
        return NumberIterator(self.max_num)

class NumberIterator:
    """Итератор для NumberSequence"""
    
    def __init__(self, max_num):
        self.max_num = max_num
        self.current = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current >= self.max_num:
            raise StopIteration
        self.current += 1
        return self.current

# Можно использовать многократно
sequence = NumberSequence(3)

for num in sequence:
    print(f"Первый проход: {num}")

for num in sequence:
    print(f"Второй проход: {num}")
```

#### ⚡ Генераторы с yield

```python
# ──────────────────────────────────────────────────────────
# Простой генератор
# ──────────────────────────────────────────────────────────

def count_down(start):
    """Генератор для обратного отсчета"""
    while start > 0:
        yield start
        start -= 1

# Использование
for num in count_down(5):
    print(f"T-{num}")

# ──────────────────────────────────────────────────────────
# Генератор для чтения больших файлов
# ──────────────────────────────────────────────────────────

def read_large_file(file_path, chunk_size=1024):
    """Читает файл по частям"""
    with open(file_path, 'r', encoding='utf-8') as file:
        while True:
            chunk = file.read(chunk_size)
            if not chunk:
                break
            yield chunk

# ──────────────────────────────────────────────────────────
# Генератор последовательности Фибоначчи
# ──────────────────────────────────────────────────────────

def fibonacci(n=None):
    """Генерирует числа Фибоначчи"""
    a, b = 0, 1
    count = 0
    
    while n is None or count < n:
        yield a
        a, b = b, a + b
        count += 1

# Первые 10 чисел Фибоначчи
fib_10 = list(fibonacci(10))
print(fib_10)  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

# Бесконечная последовательность (осторожно!)
fib_gen = fibonacci()
for i, num in enumerate(fib_gen):
    if i >= 5:  # Останавливаемся на 5-м элементе
        break
    print(num)
```

#### 🏗 Generator Expressions

```python
# ──────────────────────────────────────────────────────────
# Генераторные выражения - компактный синтаксис
# ──────────────────────────────────────────────────────────

# Обычный list comprehension (создает весь список в памяти)
squares_list = [x**2 for x in range(1000000)]  # Много памяти!

# Generator expression (ленивое вычисление)
squares_gen = (x**2 for x in range(1000000))   # Мало памяти!

# Использование
for square in squares_gen:
    if square > 100:
        print(f"Первый квадрат больше 100: {square}")
        break

# ──────────────────────────────────────────────────────────
# Практические примеры
# ──────────────────────────────────────────────────────────

# Обработка лог-файлов
def parse_log_lines(filename):
    """Парсит строки лог-файла"""
    with open(filename, 'r') as file:
        for line in file:
            line = line.strip()
            if line and not line.startswith('#'):
                parts = line.split()
                if len(parts) >= 3:
                    yield {
                        'timestamp': parts[0],
                        'level': parts[1],
                        'message': ' '.join(parts[2:])
                    }

# Фильтрация ошибок
def filter_errors(log_entries):
    """Фильтрует только ошибки"""
    for entry in log_entries:
        if entry['level'] in ['ERROR', 'CRITICAL']:
            yield entry

# Использование pipeline
# log_entries = parse_log_lines('app.log')
# errors = filter_errors(log_entries)
# for error in errors:
#     print(f"Ошибка в {error['timestamp']}: {error['message']}")
```

#### 🔄 Продвинутые техники с генераторами

```python
# ──────────────────────────────────────────────────────────
# Передача данных в генератор через send()
# ──────────────────────────────────────────────────────────

def accumulator():
    """Генератор-аккумулятор"""
    total = 0
    while True:
        value = yield total
        if value is not None:
            total += value

# Использование
acc = accumulator()
next(acc)  # Инициализация

print(acc.send(10))    # 10
print(acc.send(5))     # 15
print(acc.send(-3))    # 12

# ──────────────────────────────────────────────────────────
# Обработка исключений в генераторах
# ──────────────────────────────────────────────────────────

def robust_generator():
    """Генератор с обработкой исключений"""
    try:
        for i in range(10):
            try:
                yield i
            except ValueError as e:
                print(f"Получено исключение: {e}")
                yield f"Ошибка на {i}"
    finally:
        print("Генератор завершен")

# ──────────────────────────────────────────────────────────
# Цепочка генераторов (pipeline)
# ──────────────────────────────────────────────────────────

def numbers(n):
    """Генерирует числа от 0 до n"""
    for i in range(n):
        yield i

def squares(numbers_gen):
    """Возводит числа в квадрат"""
    for num in numbers_gen:
        yield num ** 2

def evens_only(numbers_gen):
    """Фильтрует только четные числа"""
    for num in numbers_gen:
        if num % 2 == 0:
            yield num

# Создание pipeline
pipeline = evens_only(squares(numbers(10)))
result = list(pipeline)
print(result)  # [0, 4, 16, 36, 64]

# Более элегантно с functools
from functools import reduce

def compose(*functions):
    """Композиция функций"""
    return reduce(lambda f, g: lambda x: f(g(x)), functions, lambda x: x)

# pipeline = compose(evens_only, squares)(numbers(10))
```

### 🛠 Практическое задание 5.2

```python
"""
──────────────────────────────────────────────────────────
ЗАДАНИЕ: Система обработки потока данных
──────────────────────────────────────────────────────────

Создайте систему генераторов для обработки потока данных:
1. Генератор чисел с конфигурируемым шагом
2. Фильтр по условию
3. Трансформатор данных
4. Аккумулятор результатов
"""

def number_stream(start=0, step=1, max_count=None):
    """Генерирует поток чисел"""
    current = start
    count = 0
    
    while max_count is None or count < max_count:
        yield current
        current += step
        count += 1

def filter_stream(stream, predicate):
    """Фильтрует поток по предикату"""
    for item in stream:
        if predicate(item):
            yield item

def transform_stream(stream, transformer):
    """Преобразует элементы потока"""
    for item in stream:
        yield transformer(item)

def accumulate_stream(stream, operation):
    """Накапливает результат по потоку"""
    accumulator = None
    for item in stream:
        if accumulator is None:
            accumulator = item
        else:
            accumulator = operation(accumulator, item)
        yield accumulator

# Пример использования
import operator

# Создаем pipeline
numbers = number_stream(1, 2, 10)           # 1, 3, 5, 7, 9, 11, 13, 15, 17, 19
filtered = filter_stream(numbers, lambda x: x > 5)  # 7, 9, 11, 13, 15, 17, 19
squared = transform_stream(filtered, lambda x: x**2)  # 49, 81, 121, 169, 225, 289, 361
sums = accumulate_stream(squared, operator.add)       # 49, 130, 251, 420, 645, 934, 1295

result = list(sums)
print(f"Результат: {result}")
```

---

## 🎭 5.3 Контекстные менеджеры

### Теория: Протокол контекстного менеджера

```
Контекстный менеджер
│
├── __enter__() - вход в контекст
├── __exit__()  - выход из контекста  
└── with statement - использование
```

#### 🎯 Создание контекстных менеджеров

```python
# ──────────────────────────────────────────────────────────
# Классический контекстный менеджер
# ──────────────────────────────────────────────────────────

class TimeMeasure:
    """Измеряет время выполнения блока кода"""
    
    def __enter__(self):
        import time
        self.start_time = time.time()
        print("⏱ Начинаем измерение времени...")
        return self
    
    def __exit__(self, exc_type, exc_value, traceback):
        import time
        self.end_time = time.time()
        self.duration = self.end_time - self.start_time
        
        if exc_type is None:
            print(f"✅ Выполнено за {self.duration:.4f} секунд")
        else:
            print(f"❌ Ошибка за {self.duration:.4f} секунд: {exc_value}")
        
        # False означает, что исключение НЕ подавляется
        return False

# ──────────────────────────────────────────────────────────
# Использование
# ──────────────────────────────────────────────────────────

with TimeMeasure() as timer:
    # Некоторая долгая операция
    import time
    time.sleep(1)
    result = sum(range(1000000))

print(f"Результат: {result}")

# ──────────────────────────────────────────────────────────
# Контекстный менеджер для базы данных
# ──────────────────────────────────────────────────────────

class DatabaseConnection:
    """Управляет подключением к базе данных"""
    
    def __init__(self, db_path):
        self.db_path = db_path
        self.connection = None
    
    def __enter__(self):
        print(f"🔌 Подключаемся к {self.db_path}")
        # В реальности здесь был бы код подключения к БД
        self.connection = f"connection_to_{self.db_path}"
        return self.connection
    
    def __exit__(self, exc_type, exc_value, traceback):
        print(f"🔌 Закрываем подключение к {self.db_path}")
        # В реальности здесь был бы код закрытия соединения
        self.connection = None
        
        if exc_type is not None:
            print(f"❌ Произошла ошибка: {exc_value}")
            # Здесь можно добавить логику отката транзакции
        
        return False  # Не подавляем исключения

# Использование
with DatabaseConnection("users.db") as conn:
    print(f"Работаем с {conn}")
    # Здесь код работы с базой данных
```

#### 🛠 Декоратор @contextmanager

```python
# ──────────────────────────────────────────────────────────
# Упрощенное создание с помощью декоратора
# ──────────────────────────────────────────────────────────

from contextlib import contextmanager
import os
import tempfile

@contextmanager
def temporary_file(suffix='.tmp'):
    """Создает временный файл и удаляет его после использования"""
    
    # Код __enter__
    temp_file = tempfile.NamedTemporaryFile(mode='w', suffix=suffix, delete=False)
    temp_path = temp_file.name
    
    try:
        print(f"📄 Создан временный файл: {temp_path}")
        yield temp_file  # Возвращаем файл пользователю
        
    finally:
        # Код __exit__
        temp_file.close()
        if os.path.exists(temp_path):
            os.unlink(temp_path)
            print(f"🗑 Удален временный файл: {temp_path}")

# Использование
with temporary_file('.txt') as temp:
    temp.write("Это временные данные\n")
    temp.write("Они будут удалены автоматически\n")
    print(f"Записываем в {temp.name}")

# ──────────────────────────────────────────────────────────
# Контекстный менеджер для изменения рабочей директории
# ──────────────────────────────────────────────────────────

@contextmanager
def change_directory(path):
    """Временно меняет рабочую директорию"""
    original_path = os.getcwd()
    
    try:
        print(f"📁 Переходим из {original_path} в {path}")
        os.chdir(path)
        yield path
        
    finally:
        print(f"📁 Возвращаемся в {original_path}")
        os.chdir(original_path)

# ──────────────────────────────────────────────────────────
# Контекстный менеджер для настройки логирования
# ──────────────────────────────────────────────────────────

import logging

@contextmanager
def log_level(level):
    """Временно меняет уровень логирования"""
    logger = logging.getLogger()
    old_level = logger.level
    
    try:
        logger.setLevel(level)
        print(f"🔧 Уровень логирования изменен на {logging.getLevelName(level)}")
        yield logger
        
    finally:
        logger.setLevel(old_level)
        print(f"🔧 Уровень логирования восстановлен: {logging.getLevelName(old_level)}")

# Использование
with log_level(logging.DEBUG):
    logging.debug("Это debug сообщение")
    logging.info("Это info сообщение")
```

#### 🔄 Вложенные контекстные менеджеры

```python
# ──────────────────────────────────────────────────────────
# Несколько контекстных менеджеров
# ──────────────────────────────────────────────────────────

@contextmanager
def performance_monitor(operation_name):
    """Мониторинг производительности"""
    import time
    import psutil
    
    # Начальные метрики
    start_time = time.time()
    start_memory = psutil.Process().memory_info().rss / 1024 / 1024  # MB
    
    print(f"🚀 Начинаем '{operation_name}'")
    print(f"📊 Память: {start_memory:.1f} MB")
    
    try:
        yield {
            'start_time': start_time,
            'start_memory': start_memory
        }
        
    finally:
        end_time = time.time()
        end_memory = psutil.Process().memory_info().rss / 1024 / 1024
        
        duration = end_time - start_time
        memory_diff = end_memory - start_memory
        
        print(f"✅ '{operation_name}' завершено")
        print(f"⏱ Время: {duration:.3f} сек")
        print(f"📊 Память: {end_memory:.1f} MB ({memory_diff:+.1f} MB)")

# ──────────────────────────────────────────────────────────
# Использование вложенных менеджеров
# ──────────────────────────────────────────────────────────

with performance_monitor("Обработка данных"):
    with temporary_file('.csv') as temp_file:
        # Генерируем тестовые данные
        for i in range(100000):
            temp_file.write(f"{i},{i**2},{i**3}\n")
        
        temp_file.flush()  # Убеждаемся, что данные записаны
        
        # Читаем и обрабатываем
        temp_file.seek(0)
        lines = temp_file.readlines()
        
        print(f"📝 Обработано {len(lines)} строк")

# ──────────────────────────────────────────────────────────
# Создание составного контекстного менеджера
# ──────────────────────────────────────────────────────────

from contextlib import ExitStack

@contextmanager 
def data_processing_context(temp_suffix='.tmp', log_level=logging.INFO):
    """Комбинированный контекст для обработки данных"""
    
    with ExitStack() as stack:
        # Настраиваем логирование
        logger = stack.enter_context(log_level(log_level))
        
        # Создаем временный файл
        temp_file = stack.enter_context(temporary_file(temp_suffix))
        
        # Мониторим производительность
        perf_data = stack.enter_context(performance_monitor("Обработка данных"))
        
        yield {
            'logger': logger,
            'temp_file': temp_file,
            'performance': perf_data
        }

# Использование
with data_processing_context('.json', logging.DEBUG) as ctx:
    ctx['temp_file'].write('{"test": "data"}')
    ctx['logger'].debug("Данные записаны в временный файл")
```

### 🛠 Практическое задание 5.3

```python
"""
──────────────────────────────────────────────────────────
ЗАДАНИЕ: Контекстный менеджер для ресурсов
──────────────────────────────────────────────────────────

Создайте контекстный менеджер ResourcePool для управления 
ограниченными ресурсами (например, подключения к БД, файлы и т.д.)
"""

from contextlib import contextmanager
import threading
import time
from queue import Queue, Empty

class ResourcePool:
    """Пул ресурсов с ограничением количества"""
    
    def __init__(self, create_resource_func, max_resources=5):
        self.create_resource = create_resource_func
        self.max_resources = max_resources
        self.available_resources = Queue()
        self.total_created = 0
        self.lock = threading.Lock()
    
    def acquire_resource(self, timeout=None):
        """Получает ресурс из пула"""
        try:
            # Пытаемся взять существующий ресурс
            resource = self.available_resources.get_nowait()
            print(f"♻️  Переиспользуем ресурс: {resource}")
            return resource
            
        except Empty:
            # Создаем новый ресурс, если можем
            with self.lock:
                if self.total_created < self.max_resources:
                    resource = self.create_resource()
                    self.total_created += 1
                    print(f"🆕 Создан новый ресурс: {resource} (всего: {self.total_created})")
                    return resource
            
            # Ждем освобождения ресурса
            print(f"⏳ Ожидание свободного ресурса...")
            try:
                resource = self.available_resources.get(timeout=timeout)
                print(f"✅ Получен ресурс после ожидания: {resource}")
                return resource
            except Empty:
                raise TimeoutError("Превышено время ожидания ресурса")
    
    def release_resource(self, resource):
        """Возвращает ресурс в пул"""
        self.available_resources.put(resource)
        print(f"↩️  Ресурс возвращен в пул: {resource}")
    
    @contextmanager
    def get_resource(self, timeout=10):
        """Контекстный менеджер для автоматического управления ресурсом"""
        resource = None
        try:
            resource = self.acquire_resource(timeout)
            yield resource
        finally:
            if resource:
                self.release_resource(resource)

# ──────────────────────────────────────────────────────────
# Пример использования с "базой данных"
# ──────────────────────────────────────────────────────────

def create_db_connection():
    """Имитирует создание подключения к БД"""
    import uuid
    connection_id = str(uuid.uuid4())[:8]
    time.sleep(0.1)  # Имитируем задержку подключения
    return f"DB_Connection_{connection_id}"

# Создаем пул подключений
db_pool = ResourcePool(create_db_connection, max_resources=3)

# Использование
with db_pool.get_resource() as conn1:
    print(f"🔧 Используем {conn1} для SELECT запроса")
    time.sleep(1)
    
    with db_pool.get_resource() as conn2:
        print(f"🔧 Используем {conn2} для INSERT запроса")
        time.sleep(0.5)

print("✅ Все ресурсы возвращены в пул")
```

---

## 🧬 5.4 Метаклассы и продвинутое ООП

### Теория: Метаклассы

```
Иерархия классов в Python
│
├── type (метакласс по умолчанию)
│   ├── object (базовый класс)
│   │   ├── MyClass
│   │   └── MyInstance ──► instance of MyClass
│   └── MyMetaclass ──► creates MyClass
│
└── "Классы это объекты, которые создают объекты"
```

#### 🎯 Понимание метаклассов

```python
# ──────────────────────────────────────────────────────────
# Все в Python - объекты, включая классы
# ──────────────────────────────────────────────────────────

class MyClass:
    pass

# Класс - это тоже объект!
print(f"Тип MyClass: {type(MyClass)}")           # <class 'type'>
print(f"MyClass это объект: {isinstance(MyClass, object)}")  # True

# Создание класса динамически
DynamicClass = type('DynamicClass', (), {'attr': 'value'})
print(f"Динамический класс: {DynamicClass}")     # <class '__main__.DynamicClass'>
print(f"Атрибут: {DynamicClass.attr}")           # value

# ──────────────────────────────────────────────────────────
# Метакласс через функцию
# ──────────────────────────────────────────────────────────

def debug_metaclass(name, bases, namespace):
    """Метакласс, который добавляет отладочную информацию"""
    
    print(f"🏗 Создаем класс '{name}'")
    print(f"📋 Базовые классы: {bases}")
    print(f"📝 Атрибуты: {list(namespace.keys())}")
    
    # Добавляем метод отладки к каждому классу
    def debug_info(self):
        return f"<{self.__class__.__name__} object at {hex(id(self))}>"
    
    namespace['debug_info'] = debug_info
    namespace['_created_by'] = 'debug_metaclass'
    
    # Создаем класс
    cls = type(name, bases, namespace)
    print(f"✅ Класс '{name}' создан\n")
    
    return cls

# Использование метакласса
class DebuggedClass(metaclass=debug_metaclass):
    def __init__(self, value):
        self.value = value
    
    def get_value(self):
        return self.value

# Тестирование
obj = DebuggedClass(42)
print(obj.debug_info())      # <DebuggedClass object at 0x...>
print(obj._created_by)       # debug_metaclass
```

#### 🏗 Метаклассы как классы

```python
# ──────────────────────────────────────────────────────────
# Метакласс в виде класса
# ──────────────────────────────────────────────────────────

class SingletonMeta(type):
    """Метакласс для создания Singleton объектов"""
    
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        """Вызывается при создании экземпляра"""
        if cls not in cls._instances:
            print(f"🆕 Создаем первый экземпляр {cls.__name__}")
            cls._instances[cls] = super().__call__(*args, **kwargs)
        else:
            print(f"♻️  Возвращаем существующий экземпляр {cls.__name__}")
        
        return cls._instances[cls]

class DatabaseConnection(metaclass=SingletonMeta):
    """Класс для подключения к БД (Singleton)"""
    
    def __init__(self, host="localhost"):
        self.host = host
        self.connected = False
        print(f"🔌 Инициализация подключения к {host}")
    
    def connect(self):
        self.connected = True
        print(f"✅ Подключено к {self.host}")
    
    def disconnect(self):
        self.connected = False
        print(f"❌ Отключено от {self.host}")

# Тестирование Singleton
db1 = DatabaseConnection("server1")
db2 = DatabaseConnection("server2")  # Тот же объект!

print(f"db1 is db2: {db1 is db2}")   # True
print(f"Host: {db1.host}")           # server1 (не изменился!)

# ──────────────────────────────────────────────────────────
# Метакласс для автоматической регистрации классов
# ──────────────────────────────────────────────────────────

class RegistryMeta(type):
    """Метакласс для автоматической регистрации классов"""
    
    registry = {}
    
    def __new__(mcs, name, bases, namespace):
        # Создаем класс
        cls = super().__new__(mcs, name, bases, namespace)
        
        # Регистрируем класс
        if name != 'RegisteredBase':  # Исключаем базовый класс
            mcs.registry[name.lower()] = cls
            print(f"📝 Зарегистрирован класс: {name}")
        
        return cls
    
    @classmethod
    def get_class(mcs, name):
        """Получает класс по имени"""
        return mcs.registry.get(name.lower())
    
    @classmethod
    def list_classes(mcs):
        """Возвращает все зарегистрированные классы"""
        return list(mcs.registry.keys())

class RegisteredBase(metaclass=RegistryMeta):
    """Базовый класс для регистрируемых классов"""
    pass

# Наследники автоматически регистрируются
class User(RegisteredBase):
    def __init__(self, name):
        self.name = name

class Product(RegisteredBase):
    def __init__(self, title, price):
        self.title = title
        self.price = price

class Order(RegisteredBase):
    def __init__(self, user, products):
        self.user = user
        self.products = products

# Использование реестра
print(f"📋 Зарегистрированные классы: {RegistryMeta.list_classes()}")

# Создание объектов через реестр
UserClass = RegistryMeta.get_class('user')
user = UserClass("Алиса")
print(f"👤 Создан пользователь: {user.name}")
```

#### 🔍 Дескрипторы

```python
# ──────────────────────────────────────────────────────────
# Дескрипторы - управление доступом к атрибутам
# ──────────────────────────────────────────────────────────

class ValidatedAttribute:
    """Дескриптор для валидации атрибутов"""
    
    def __init__(self, validator=None, default=None):
        self.validator = validator
        self.default = default
        self.name = None  # Будет установлено автоматически
    
    def __set_name__(self, owner, name):
        """Вызывается при присвоении дескриптора атрибуту класса"""
        self.name = name
        self.private_name = f'_{name}'
    
    def __get__(self, instance, owner):
        """Вызывается при чтении атрибута"""
        if instance is None:
            return self
        
        return getattr(instance, self.private_name, self.default)
    
    def __set__(self, instance, value):
        """Вызывается при записи атрибута"""
        if self.validator:
            if not self.validator(value):
                raise ValueError(f"Некорректное значение для {self.name}: {value}")
        
        setattr(instance, self.private_name, value)
    
    def __delete__(self, instance):
        """Вызывается при удалении атрибута"""
        delattr(instance, self.private_name)

# ──────────────────────────────────────────────────────────
# Специализированные дескрипторы
# ──────────────────────────────────────────────────────────

class PositiveNumber(ValidatedAttribute):
    """Дескриптор для положительных чисел"""
    
    def __init__(self, default=0):
        super().__init__(
            validator=lambda x: isinstance(x, (int, float)) and x > 0,
            default=default
        )

class Email(ValidatedAttribute):
    """Дескриптор для email адресов"""
    
    def __init__(self):
        import re
        email_pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        super().__init__(
            validator=lambda x: isinstance(x, str) and re.match(email_pattern, x),
            default=""
        )

class NonEmptyString(ValidatedAttribute):
    """Дескриптор для непустых строк"""
    
    def __init__(self, min_length=1):
        self.min_length = min_length
        super().__init__(
            validator=lambda x: isinstance(x, str) and len(x.strip()) >= min_length,
            default=""
        )

# ──────────────────────────────────────────────────────────
# Использование дескрипторов
# ──────────────────────────────────────────────────────────

class User:
    """Класс пользователя с валидированными атрибутами"""
    
    name = NonEmptyString(min_length=2)
    email = Email()
    age = PositiveNumber()
    balance = PositiveNumber(default=0.0)
    
    def __init__(self, name, email, age):
        self.name = name
        self.email = email
        self.age = age
    
    def __repr__(self):
        return f"User(name='{self.name}', email='{self.email}', age={self.age})"

# Тестирование
try:
    user = User("Алиса", "alice@example.com", 25)
    print(f"✅ Пользователь создан: {user}")
    
    user.balance = 100.50
    print(f"💰 Баланс: {user.balance}")
    
    # Это вызовет ошибку
    user.age = -5  # ValueError!
    
except ValueError as e:
    print(f"❌ Ошибка валидации: {e}")
```

#### 🔗 Множественное наследование и MRO

```python
# ──────────────────────────────────────────────────────────
# Method Resolution Order (MRO) - порядок разрешения методов
# ──────────────────────────────────────────────────────────

class A:
    def method(self):
        print("Метод A")
        return "A"

class B(A):
    def method(self):
        print("Метод B")
        result = super().method()  # Вызывает A.method()
        return f"B -> {result}"

class C(A):
    def method(self):
        print("Метод C")
        result = super().method()  # Вызывает A.method()
        return f"C -> {result}"

class D(B, C):
    def method(self):
        print("Метод D")
        result = super().method()  # Вызывает B.method()
        return f"D -> {result}"

# Анализ MRO
print("🔍 MRO для класса D:")
for i, cls in enumerate(D.__mro__):
    print(f"  {i}: {cls.__name__}")

# MRO: D -> B -> C -> A -> object

obj = D()
result = obj.method()
print(f"\n📝 Результат: {result}")

# ──────────────────────────────────────────────────────────
# Миксины - классы для переиспользования функциональности
# ──────────────────────────────────────────────────────────

class TimestampMixin:
    """Миксин для добавления временных меток"""
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        from datetime import datetime
        self.created_at = datetime.now()
        self.updated_at = self.created_at
    
    def touch(self):
        """Обновляет временную метку"""
        from datetime import datetime
        self.updated_at = datetime.now()

class JsonMixin:
    """Миксин для JSON сериализации"""
    
    def to_json(self):
        """Конвертирует объект в JSON"""
        import json
        from datetime import datetime
        
        data = {}
        for key, value in self.__dict__.items():
            if isinstance(value, datetime):
                data[key] = value.isoformat()
            else:
                data[key] = value
        
        return json.dumps(data, indent=2)
    
    @classmethod
    def from_json(cls, json_str):
        """Создает объект из JSON"""
        import json
        from datetime import datetime
        
        data = json.loads(json_str)
        
        # Преобразуем datetime строки обратно
        for key, value in data.items():
            if isinstance(value, str) and 'T' in value:
                try:
                    data[key] = datetime.fromisoformat(value.replace('Z', '+00:00'))
                except ValueError:
                    pass
        
        return cls(**data)

class ValidationMixin:
    """Миксин для валидации данных"""
    
    def validate(self):
        """Валидирует объект"""
        errors = []
        
        # Проверяем обязательные поля
        required_fields = getattr(self, '_required_fields', [])
        for field in required_fields:
            if not hasattr(self, field) or getattr(self, field) is None:
                errors.append(f"Поле '{field}' обязательно")
        
        if errors:
            raise ValueError(f"Ошибки валидации: {'; '.join(errors)}")
        
        return True

# ──────────────────────────────────────────────────────────
# Использование миксинов
# ──────────────────────────────────────────────────────────

class Product(TimestampMixin, JsonMixin, ValidationMixin):
    """Продукт с автоматическими возможностями"""
    
    _required_fields = ['name', 'price']
    
    def __init__(self, name, price, description=""):
        self.name = name
        self.price = price
        self.description = description
        super().__init__()  # Инициализируем миксины
    
    def __repr__(self):
        return f"Product(name='{self.name}', price={self.price})"

# Тестирование
product = Product("Ноутбук", 50000, "Игровой ноутбук")

print(f"📦 Продукт: {product}")
print(f"📅 Создан: {product.created_at}")

# Валидация
product.validate()  # OK

# JSON сериализация
json_data = product.to_json()
print(f"\n📄 JSON:\n{json_data}")

# Обновление временной метки
import time
time.sleep(1)
product.touch()
print(f"\n📅 Обновлен: {product.updated_at}")
```

### 🛠 Практическое задание 5.4

```python
"""
──────────────────────────────────────────────────────────
ЗАДАНИЕ: ORM-подобная система с метаклассами
──────────────────────────────────────────────────────────

Создайте упрощенную ORM систему используя метаклассы и дескрипторы:
1. Метакласс для автоматической генерации SQL
2. Дескрипторы для типов полей
3. Методы для CRUD операций
"""

# Дескрипторы полей
class Field:
    """Базовый класс для полей модели"""
    
    def __init__(self, field_type, primary_key=False, nullable=True, default=None):
        self.field_type = field_type
        self.primary_key = primary_key
        self.nullable = nullable
        self.default = default
        self.name = None
    
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__.get(self.name, self.default)
    
    def __set__(self, instance, value):
        if value is None and not self.nullable:
            raise ValueError(f"Поле {self.name} не может быть None")
        
        if value is not None and not isinstance(value, self.field_type):
            try:
                value = self.field_type(value)
            except (ValueError, TypeError):
                raise ValueError(f"Неверный тип для поля {self.name}")
        
        instance.__dict__[self.name] = value

class IntegerField(Field):
    def __init__(self, primary_key=False, nullable=True, default=None):
        super().__init__(int, primary_key, nullable, default)

class StringField(Field):
    def __init__(self, max_length=255, nullable=True, default=""):
        super().__init__(str, nullable=nullable, default=default)
        self.max_length = max_length
    
    def __set__(self, instance, value):
        super().__set__(instance, value)
        if value and len(value) > self.max_length:
            raise ValueError(f"Длина поля {self.name} превышает {self.max_length}")

class FloatField(Field):
    def __init__(self, nullable=True, default=0.0):
        super().__init__(float, nullable=nullable, default=default)

# Метакласс для модели
class ModelMeta(type):
    """Метакласс для создания моделей ORM"""
    
    def __new__(mcs, name, bases, namespace):
        # Собираем поля модели
        fields = {}
        for key, value in namespace.items():
            if isinstance(value, Field):
                fields[key] = value
        
        # Добавляем поля в класс
        namespace['_fields'] = fields
        namespace['_table_name'] = name.lower()
        
        # Ищем первичный ключ
        primary_key = None
        for field_name, field in fields.items():
            if field.primary_key:
                primary_key = field_name
                break
        
        namespace['_primary_key'] = primary_key
        
        # Создаем класс
        cls = super().__new__(mcs, name, bases, namespace)
        
        print(f"🗄 Создана модель '{name}' с полями: {list(fields.keys())}")
        
        return cls

class Model(metaclass=ModelMeta):
    """Базовый класс для всех моделей"""
    
    def __init__(self, **kwargs):
        # Устанавливаем значения полей
        for field_name in self._fields:
            value = kwargs.get(field_name, self._fields[field_name].default)
            setattr(self, field_name, value)
    
    def save(self):
        """Имитация сохранения в БД"""
        if hasattr(self, self._primary_key) and getattr(self, self._primary_key):
            print(f"💾 UPDATE {self._table_name} SET {self._get_update_sql()}")
        else:
            print(f"💾 INSERT INTO {self._table_name} {self._get_insert_sql()}")
        return self
    
    def delete(self):
        """Имитация удаления из БД"""
        if self._primary_key and getattr(self, self._primary_key):
            print(f"🗑 DELETE FROM {self._table_name} WHERE {self._primary_key} = {getattr(self, self._primary_key)}")
    
    def _get_insert_sql(self):
        """Генерирует SQL для INSERT"""
        fields = []
        values = []
        
        for field_name, field in self._fields.items():
            if field.primary_key and getattr(self, field_name) is None:
                continue  # Пропускаем автоинкремент
            
            fields.append(field_name)
            value = getattr(self, field_name)
            if isinstance(value, str):
                values.append(f"'{value}'")
            else:
                values.append(str(value))
        
        return f"({', '.join(fields)}) VALUES ({', '.join(values)})"
    
    def _get_update_sql(self):
        """Генерирует SQL для UPDATE"""
        updates = []
        
        for field_name, field in self._fields.items():
            if field.primary_key:
                continue
            
            value = getattr(self, field_name)
            if isinstance(value, str):
                updates.append(f"{field_name} = '{value}'")
            else:
                updates.append(f"{field_name} = {value}")
        
        pk_value = getattr(self, self._primary_key)
        return f"{', '.join(updates)} WHERE {self._primary_key} = {pk_value}"
    
    def __repr__(self):
        field_values = []
        for field_name in self._fields:
            value = getattr(self, field_name)
            field_values.append(f"{field_name}={value}")
        
        return f"{self.__class__.__name__}({', '.join(field_values)})"

# ──────────────────────────────────────────────────────────
# Использование ORM
# ──────────────────────────────────────────────────────────

class User(Model):
    id = IntegerField(primary_key=True)
    name = StringField(max_length=100, nullable=False)
    email = StringField(max_length=255)
    age = IntegerField(nullable=True)
    balance = FloatField(default=0.0)

class Product(Model):
    id = IntegerField(primary_key=True)
    title = StringField(max_length=200, nullable=False)
    price = FloatField(nullable=False)
    description = StringField(max_length=1000, default="")

# Тестирование
print("=" * 60)

# Создание пользователя
user = User(name="Алиса", email="alice@example.com", age=25, balance=1000.0)
print(f"👤 {user}")
user.save()

# Обновление пользователя
user.id = 1  # Имитируем назначение ID после INSERT
user.balance = 1500.0
user.save()

# Создание продукта
product = Product(title="Ноутбук", price=50000.0, description="Игровой ноутбук")
print(f"📦 {product}")
product.save()

print("=" * 60)
```

---

*Продолжение следует... В следующих частях мы рассмотрим остальные темы блока 5: многопоточность, асинхронное программирование, профилирование, паттерны проектирования и инструменты качества кода.*