# PostgreSQL: Полное руководство для Backend разработчика
*От основ до экспертного уровня за 16 недель*

## Введение: Зачем PostgreSQL backend разработчику?

**PostgreSQL** - это не просто база данных, это мощная платформа для создания надежных backend систем:

- **ACID транзакции**: Гарантия целостности данных в любых условиях
- **Расширяемость**: JSON, массивы, custom типы, расширения
- **Производительность**: Развитая система индексирования и оптимизации
- **Надежность**: Проверено годами в enterprise окружении
- **Open Source**: Бесплатно, активное сообщество, постоянное развитие

---

# ФАЗА 1: ОСНОВЫ SQL И POSTGRESQL (Недели 1-3)

## Неделя 1: Фундаментальные концепции

### Понимание реляционных баз данных

#### Что такое СУБД и зачем она нужна
**СУБД (Система Управления Базами Данных)** решает ключевые проблемы backend разработки:

1. **Персистентность**: данные сохраняются между перезапусками
2. **Concurrent доступ**: множество пользователей одновременно
3. **ACID гарантии**: данные всегда в консистентном состоянии
4. **Масштабируемость**: от сотен до миллионов записей

#### Концепция таблиц, строк, столбцов
```sql
-- Таблица = структурированное хранилище
-- Строка = один объект (пользователь, заказ, товар)
-- Столбец = свойство объекта (имя, email, цена)

CREATE TABLE users (
    id INTEGER,           -- Столбец: уникальный идентификатор
    email VARCHAR(255),   -- Столбец: email пользователя
    name VARCHAR(100),    -- Столбец: имя пользователя
    created_at TIMESTAMP  -- Столбец: время создания
);
-- Каждая строка = один пользователь со всеми его свойствами
```

#### Первичные и внешние ключи
```sql
-- Первичный ключ (PRIMARY KEY) - уникальный идентификатор строки
CREATE TABLE users (
    id SERIAL PRIMARY KEY,        -- SERIAL = автоинкремент
    email VARCHAR(255) UNIQUE,    -- UNIQUE = уникальное значение
    name VARCHAR(100) NOT NULL    -- NOT NULL = обязательное поле
);

-- Внешний ключ (FOREIGN KEY) - связь между таблицами
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    total DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    
    -- Связь: каждый заказ принадлежит пользователю
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

#### Нормализация данных (1NF, 2NF, 3NF)
```sql
-- ❌ Не нормализованная таблица (плохо)
CREATE TABLE bad_orders (
    order_id INTEGER,
    customer_name VARCHAR(100),
    customer_email VARCHAR(255),
    customer_phone VARCHAR(20),
    product_names TEXT,           -- "Laptop, Mouse, Keyboard"
    product_prices TEXT,          -- "1000, 25, 75"
    total DECIMAL(10,2)
);

-- ✅ Нормализованные таблицы (хорошо)
-- 1NF: Атомарные значения (нет списков в одном поле)
-- 2NF: Убираем дублирование данных клиента
-- 3NF: Выносим продукты в отдельную таблицу

CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255),
    phone VARCHAR(20)
);

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    price DECIMAL(10,2)
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(id),
    total DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE order_items (
    order_id INTEGER REFERENCES orders(id),
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER,
    price DECIMAL(10,2),        -- Цена на момент заказа
    PRIMARY KEY (order_id, product_id)
);
```

### Установка и первоначальная настройка PostgreSQL

#### Основные команды psql
```bash
# Подключение к базе
psql -h localhost -U postgres -d mydb

# Основные команды в psql:
\l          # Список баз данных
\d          # Список таблиц
\d users    # Структура таблицы users
\q          # Выход
```

#### Создание первой базы данных
```sql
-- Создание базы данных
CREATE DATABASE ecommerce 
    WITH ENCODING 'UTF8' 
    LC_COLLATE='en_US.UTF-8' 
    LC_CTYPE='en_US.UTF-8';

-- Подключение к базе
\c ecommerce

-- Создание схемы (namespace для таблиц)
CREATE SCHEMA shop;
CREATE SCHEMA analytics;
```

### Основы DDL (Data Definition Language)

#### CREATE TABLE с различными типами данных
```sql
CREATE TABLE users (
    -- Числовые типы
    id BIGSERIAL PRIMARY KEY,           -- Автоинкремент (8 байт)
    age SMALLINT CHECK (age > 0),       -- Маленькие числа (2 байта)
    balance DECIMAL(15,2) DEFAULT 0,    -- Точные числа для денег
    
    -- Строковые типы
    email VARCHAR(255) UNIQUE NOT NULL, -- Ограниченная длина
    bio TEXT,                           -- Неограниченная длина
    status CHAR(1) DEFAULT 'A',         -- Фиксированная длина
    
    -- Дата и время
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    birth_date DATE,
    last_login TIME,
    
    -- Логический тип
    is_active BOOLEAN DEFAULT true,
    
    -- JSON типы
    preferences JSONB,                  -- Бинарный JSON (быстрее)
    metadata JSON                       -- Текстовый JSON
);
```

#### ALTER TABLE операции
```sql
-- Добавление столбца
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- Изменение типа данных
ALTER TABLE users ALTER COLUMN phone TYPE VARCHAR(25);

-- Добавление ограничения
ALTER TABLE users ADD CONSTRAINT check_email 
    CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');

-- Удаление столбца
ALTER TABLE users DROP COLUMN metadata;

-- Переименование столбца
ALTER TABLE users RENAME COLUMN bio TO description;
```

#### Ограничения (CONSTRAINTS)
```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    category_id INTEGER,
    sku VARCHAR(50) UNIQUE,
    
    -- Ограничения
    CONSTRAINT positive_price CHECK (price > 0),
    CONSTRAINT valid_sku CHECK (length(sku) >= 3),
    CONSTRAINT fk_category 
        FOREIGN KEY (category_id) REFERENCES categories(id) 
        ON DELETE SET NULL            -- При удалении категории
        ON UPDATE CASCADE             -- При изменении ID категории
);
```

---

## Неделя 2: Основные SQL операции

### DML (Data Manipulation Language)

#### INSERT: одиночные и множественные вставки
```sql
-- Одиночная вставка
INSERT INTO users (email, name, age) 
VALUES ('john@example.com', 'John Doe', 25);

-- Множественная вставка
INSERT INTO users (email, name, age) VALUES 
    ('alice@example.com', 'Alice Smith', 30),
    ('bob@example.com', 'Bob Johnson', 28),
    ('carol@example.com', 'Carol Williams', 35);

-- Вставка с возвратом данных
INSERT INTO users (email, name) 
VALUES ('newuser@example.com', 'New User')
RETURNING id, created_at;

-- Вставка из другой таблицы
INSERT INTO archived_users (email, name, age)
SELECT email, name, age 
FROM users 
WHERE last_login < NOW() - INTERVAL '1 year';

-- Вставка с конфликтами (UPSERT)
INSERT INTO users (email, name, age) 
VALUES ('existing@example.com', 'Updated Name', 26)
ON CONFLICT (email) 
DO UPDATE SET 
    name = EXCLUDED.name,
    age = EXCLUDED.age,
    updated_at = NOW();
```

#### SELECT: базовые запросы и фильтрация
```sql
-- Базовый SELECT
SELECT id, email, name FROM users;

-- Вычисляемые поля
SELECT 
    name,
    age,
    CASE 
        WHEN age < 18 THEN 'Minor'
        WHEN age < 65 THEN 'Adult' 
        ELSE 'Senior'
    END as age_group,
    EXTRACT(YEAR FROM created_at) as registration_year
FROM users;

-- Псевдонимы (aliases)
SELECT 
    u.name as user_name,
    u.email as user_email,
    COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name, u.email;
```

#### UPDATE: обновление данных с условиями
```sql
-- Простое обновление
UPDATE users 
SET last_login = NOW() 
WHERE email = 'john@example.com';

-- Обновление с подзапросом
UPDATE users 
SET status = 'VIP'
WHERE id IN (
    SELECT user_id 
    FROM orders 
    GROUP BY user_id 
    HAVING SUM(total) > 10000
);

-- Обновление с JOIN
UPDATE products 
SET price = price * 0.9
FROM categories c
WHERE products.category_id = c.id 
AND c.name = 'Electronics';

-- Условное обновление
UPDATE users 
SET 
    status = CASE 
        WHEN age < 18 THEN 'MINOR'
        WHEN age >= 65 THEN 'SENIOR'
        ELSE 'REGULAR'
    END,
    updated_at = NOW();
```

#### DELETE: удаление данных
```sql
-- Простое удаление
DELETE FROM users WHERE last_login < '2023-01-01';

-- Удаление с подзапросом
DELETE FROM products 
WHERE category_id IN (
    SELECT id FROM categories WHERE name = 'Discontinued'
);

-- Каскадное удаление (через FOREIGN KEY)
-- При удалении пользователя удаляются его заказы
DELETE FROM users WHERE id = 123;
```

### Работа с типами данных PostgreSQL

#### Числовые типы
```sql
CREATE TABLE financial_data (
    id SERIAL,                          -- 4 байта, до 2 млрд
    big_id BIGSERIAL,                   -- 8 байт, до 9 * 10^18
    small_count SMALLINT,               -- 2 байта, до 32k
    
    price DECIMAL(15,2),                -- Точное число для денег
    percentage NUMERIC(5,2),            -- 999.99% максимум
    scientific_value DOUBLE PRECISION,  -- Для научных вычислений
    simple_number INTEGER,              -- Целое число
    
    -- Примеры использования
    tax_rate DECIMAL(5,4),              -- 0.0825 (8.25%)
    weight REAL,                        -- Приблизительное число
    latitude DOUBLE PRECISION,          -- Координаты GPS
    longitude DOUBLE PRECISION
);
```

#### Строковые типы
```sql
CREATE TABLE text_examples (
    id SERIAL PRIMARY KEY,
    
    -- Фиксированная длина (дополняется пробелами)
    country_code CHAR(2),               -- 'US', 'RU'
    
    -- Переменная длина с ограничением
    email VARCHAR(255),                 -- До 255 символов
    phone VARCHAR(20),                  -- До 20 символов
    
    -- Неограниченная длина
    description TEXT,                   -- Любая длина
    article_content TEXT,
    
    -- Специальные ограничения
    status VARCHAR(10) CHECK (status IN ('active', 'inactive', 'pending'))
);

-- Операции со строками
SELECT 
    UPPER(name) as name_upper,          -- Верхний регистр
    LOWER(email) as email_lower,        -- Нижний регистр
    LENGTH(description) as desc_length, -- Длина строки
    SUBSTRING(phone FROM 1 FOR 3) as area_code,  -- Подстрока
    CONCAT(name, ' (', email, ')') as full_info,  -- Конкатенация
    name || ' - ' || email as alt_concat          -- Альтернативная конкатенация
FROM users;
```

#### Дата и время
```sql
CREATE TABLE datetime_examples (
    id SERIAL PRIMARY KEY,
    
    -- Только дата
    birth_date DATE,                    -- '1990-05-15'
    
    -- Только время
    opening_time TIME,                  -- '09:30:00'
    
    -- Дата и время без часового пояса
    created_at TIMESTAMP,               -- '2024-01-15 14:30:00'
    
    -- Дата и время с часовым поясом (рекомендуется)
    updated_at TIMESTAMPTZ DEFAULT NOW(),  -- '2024-01-15 14:30:00+03'
    
    -- Интервалы
    session_duration INTERVAL           -- '2 hours 30 minutes'
);

-- Операции с датами
SELECT 
    NOW() as current_time,
    CURRENT_DATE as today,
    CURRENT_TIME as now_time,
    
    -- Извлечение частей даты
    EXTRACT(YEAR FROM created_at) as year,
    EXTRACT(MONTH FROM created_at) as month,
    EXTRACT(DOW FROM created_at) as day_of_week,  -- 0=Sunday
    
    -- Арифметика с датами
    created_at + INTERVAL '1 month' as next_month,
    created_at - INTERVAL '7 days' as week_ago,
    AGE(birth_date) as age,
    
    -- Форматирование
    TO_CHAR(created_at, 'YYYY-MM-DD HH24:MI:SS') as formatted_date
FROM users;
```

#### JSON и JSONB
```sql
CREATE TABLE user_profiles (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    
    -- JSON (текстовое хранение)
    settings JSON,
    
    -- JSONB (бинарное хранение, быстрее)
    preferences JSONB,
    metadata JSONB
);

-- Вставка JSON данных
INSERT INTO user_profiles (user_id, preferences) VALUES 
(1, '{"theme": "dark", "language": "en", "notifications": {"email": true, "sms": false}}'),
(2, '{"theme": "light", "language": "ru", "features": ["premium", "beta"]}');

-- Запросы к JSON
SELECT 
    user_id,
    preferences->>'theme' as theme,                    -- Текстовое значение
    preferences->'notifications'->>'email' as email_notifications,
    (preferences->'notifications'->>'email')::boolean as email_bool,
    
    -- Проверка существования ключа
    preferences ? 'theme' as has_theme,
    preferences ? 'premium' as has_premium,
    
    -- Проверка на содержание
    preferences @> '{"theme": "dark"}' as is_dark_theme
FROM user_profiles;

-- Обновление JSON
UPDATE user_profiles 
SET preferences = preferences || '{"last_login": "2024-01-15T10:30:00Z"}'
WHERE user_id = 1;

-- Удаление ключа из JSON
UPDATE user_profiles 
SET preferences = preferences - 'temporary_key'
WHERE user_id = 1;
```

---

## Неделя 3: Расширенные SELECT запросы

### Сортировка и ограничения

#### ORDER BY (ASC, DESC)
```sql
-- Простая сортировка
SELECT name, email, created_at 
FROM users 
ORDER BY created_at DESC;  -- Новые пользователи сначала

-- Сортировка по нескольким полям
SELECT name, age, created_at 
FROM users 
ORDER BY age DESC, name ASC;  -- Сначала по возрасту, потом по имени

-- Сортировка с NULL значениями
SELECT name, last_login 
FROM users 
ORDER BY last_login DESC NULLS LAST;  -- NULL в конце

-- Сортировка по вычисляемому полю
SELECT 
    name,
    LENGTH(name) as name_length
FROM users 
ORDER BY LENGTH(name) DESC, name;

-- Сортировка по условию (CASE)
SELECT name, status 
FROM users 
ORDER BY 
    CASE status 
        WHEN 'active' THEN 1
        WHEN 'pending' THEN 2 
        WHEN 'inactive' THEN 3
        ELSE 4
    END;
```

#### LIMIT и OFFSET (пагинация)
```sql
-- Первые 10 записей
SELECT * FROM users ORDER BY id LIMIT 10;

-- Пагинация: страница 3 по 20 записей (записи 41-60)
SELECT * FROM users 
ORDER BY id 
LIMIT 20 OFFSET 40;

-- Более эффективная пагинация через курсор
SELECT * FROM users 
WHERE id > 1000  -- Последний ID с предыдущей страницы
ORDER BY id 
LIMIT 20;

-- Пагинация с общим количеством
SELECT 
    *,
    COUNT(*) OVER() as total_count  -- Общее количество записей
FROM users 
ORDER BY created_at DESC 
LIMIT 10 OFFSET 20;
```

#### DISTINCT
```sql
-- Уникальные значения
SELECT DISTINCT status FROM users;

-- Уникальные комбинации
SELECT DISTINCT status, age FROM users;

-- DISTINCT ON (PostgreSQL специфика)
SELECT DISTINCT ON (status) 
    status, name, created_at
FROM users 
ORDER BY status, created_at DESC;  -- Последний пользователь в каждом статусе
```

### Агрегатные функции

#### COUNT, SUM, AVG, MIN, MAX
```sql
-- Базовые агрегации
SELECT 
    COUNT(*) as total_users,              -- Общее количество
    COUNT(last_login) as active_users,    -- Количество с login (не NULL)
    COUNT(DISTINCT status) as status_count, -- Уникальные статусы
    
    AVG(age) as average_age,              -- Средний возраст
    MIN(created_at) as first_registration, -- Первая регистрация
    MAX(created_at) as last_registration,  -- Последняя регистрация
    
    SUM(CASE WHEN status = 'active' THEN 1 ELSE 0 END) as active_count
FROM users;

-- Агрегации с условиями
SELECT 
    COUNT(*) FILTER (WHERE age >= 18) as adults,
    COUNT(*) FILTER (WHERE age < 18) as minors,
    AVG(age) FILTER (WHERE status = 'active') as avg_active_age
FROM users;
```

#### GROUP BY и HAVING
```sql
-- Группировка по одному полю
SELECT 
    status,
    COUNT(*) as user_count,
    AVG(age) as avg_age
FROM users 
GROUP BY status
ORDER BY user_count DESC;

-- Группировка по нескольким полям
SELECT 
    status,
    EXTRACT(YEAR FROM created_at) as registration_year,
    COUNT(*) as count
FROM users 
GROUP BY status, EXTRACT(YEAR FROM created_at)
ORDER BY registration_year DESC, status;

-- HAVING для фильтрации групп
SELECT 
    status,
    COUNT(*) as user_count
FROM users 
GROUP BY status
HAVING COUNT(*) > 100  -- Только статусы с >100 пользователей
ORDER BY user_count DESC;

-- Сложная аналитика заказов
SELECT 
    u.name,
    COUNT(o.id) as order_count,
    SUM(o.total) as total_spent,
    AVG(o.total) as avg_order_value,
    MIN(o.created_at) as first_order,
    MAX(o.created_at) as last_order
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name
HAVING COUNT(o.id) > 0  -- Только пользователи с заказами
ORDER BY total_spent DESC;
```

### Фильтрация и условия

#### Операторы сравнения
```sql
-- Базовые операторы
SELECT * FROM products WHERE price > 100;
SELECT * FROM products WHERE price <= 50;
SELECT * FROM products WHERE price <> 0;  -- Не равно (можно также !=)

-- Комбинирование условий
SELECT * FROM products 
WHERE price BETWEEN 50 AND 200 
  AND category_id = 1 
  AND stock_quantity > 0;

-- Приоритет операторов (AND выполняется раньше OR)
SELECT * FROM products 
WHERE (category_id = 1 OR category_id = 2) 
  AND price > 100;
```

#### LIKE, ILIKE для поиска по шаблону
```sql
-- LIKE (чувствительный к регистру)
SELECT * FROM users WHERE name LIKE 'John%';      -- Начинается с 'John'
SELECT * FROM users WHERE email LIKE '%@gmail.com'; -- Заканчивается на '@gmail.com'
SELECT * FROM users WHERE name LIKE '%smith%';     -- Содержит 'smith'

-- ILIKE (нечувствительный к регистру, PostgreSQL специфика)
SELECT * FROM users WHERE name ILIKE '%JOHN%';     -- Найдет 'john', 'John', 'JOHN'

-- Специальные символы
SELECT * FROM products WHERE sku LIKE 'ABC-___';   -- _ = один любой символ
SELECT * FROM products WHERE name LIKE '%\%%';     -- Поиск символа %

-- Регулярные выражения (PostgreSQL)
SELECT * FROM users WHERE email ~ '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$';
SELECT * FROM users WHERE phone ~* '^\+7';         -- ~* = нечувствительный к регистру
```

#### IN, NOT IN
```sql
-- IN для списка значений
SELECT * FROM products WHERE category_id IN (1, 3, 5);
SELECT * FROM users WHERE status IN ('active', 'pending');

-- NOT IN
SELECT * FROM products WHERE category_id NOT IN (1, 3, 5);

-- IN с подзапросом
SELECT * FROM users 
WHERE id IN (
    SELECT user_id FROM orders WHERE total > 1000
);

-- Осторожно с NULL в NOT IN!
-- Если в подзапросе есть NULL, NOT IN может вернуть пустой результат
SELECT * FROM users 
WHERE id NOT IN (
    SELECT user_id FROM orders WHERE user_id IS NOT NULL
);
```

#### BETWEEN
```sql
-- Числовые диапазоны
SELECT * FROM orders WHERE total BETWEEN 100 AND 500;

-- Даты
SELECT * FROM orders 
WHERE created_at BETWEEN '2024-01-01' AND '2024-01-31';

-- BETWEEN включает границы (>= и <=)
SELECT * FROM products WHERE price BETWEEN 10 AND 20;  -- price >= 10 AND price <= 20

-- Исключение границ
SELECT * FROM products WHERE price > 10 AND price < 20;
```

#### IS NULL, IS NOT NULL
```sql
-- Поиск NULL значений
SELECT * FROM users WHERE last_login IS NULL;       -- Никогда не логинились
SELECT * FROM users WHERE phone IS NOT NULL;        -- Есть телефон

-- NULL в вычислениях
SELECT 
    name,
    age,
    COALESCE(phone, 'No phone') as phone_display,    -- Замена NULL
    NULLIF(bio, '') as bio_cleaned,                  -- Пустая строка → NULL
    CASE 
        WHEN last_login IS NULL THEN 'Never logged in'
        ELSE 'Has logged in'
    END as login_status
FROM users;

-- Сортировка с NULL
SELECT name, last_login 
FROM users 
ORDER BY last_login DESC NULLS LAST;  -- NULL в конце
```

---

# ФАЗА 2: ПРОДВИНУТЫЙ SQL (Недели 4-7)

## Недели 4-5: Объединения таблиц (JOINs)

### Концепция JOIN операций

JOIN позволяет объединять данные из нескольких таблиц на основе связей между ними. Это основа работы с нормализованными данными.

#### Подготовка тестовых данных
```sql
-- Создадим структуру для примеров
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    budget DECIMAL(10,2)
);

CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department_id INTEGER,
    salary DECIMAL(10,2),
    hire_date DATE
);

-- Тестовые данные
INSERT INTO departments (name, budget) VALUES 
    ('IT', 500000),
    ('Sales', 300000),
    ('HR', 150000);

INSERT INTO employees (name, department_id, salary, hire_date) VALUES 
    ('Alice Johnson', 1, 75000, '2023-01-15'),
    ('Bob Smith', 1, 85000, '2022-03-10'),
    ('Carol Davis', 2, 65000, '2023-06-01'),
    ('David Wilson', 2, 70000, '2022-11-20'),
    ('Eve Brown', NULL, 60000, '2024-01-10');  -- Сотрудник без отдела
```

### Типы JOIN операций

#### INNER JOIN: пересечение
```sql
-- INNER JOIN возвращает только записи, имеющие соответствие в обеих таблицах
SELECT 
    e.name as employee_name,
    e.salary,
    d.name as department_name,
    d.budget
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
-- Результат: Eve Brown не попадет в результат (нет отдела)

-- Краткая форма записи (без INNER)
SELECT e.name, d.name 
FROM employees e
JOIN departments d ON e.department_id = d.id;

-- JOIN с дополнительными условиями
SELECT e.name, d.name, e.salary
FROM employees e
JOIN departments d ON e.department_id = d.id 
WHERE e.salary > 70000 AND d.budget > 200000;
```

#### LEFT JOIN: все из левой таблицы
```sql
-- LEFT JOIN возвращает ВСЕ записи из левой таблицы
-- + соответствующие из правой (или NULL)
SELECT 
    e.name as employee_name,
    COALESCE(d.name, 'No Department') as department_name,
    e.salary
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
-- Результат: Eve Brown будет с department_name = NULL

-- Поиск "сирот" (записи без связей)
SELECT e.name, e.salary
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
WHERE d.id IS NULL;  -- Сотрудники без отдела
```

#### RIGHT JOIN: все из правой таблицы
```sql
-- RIGHT JOIN возвращает ВСЕ записи из правой таблицы
SELECT 
    COALESCE(e.name, 'No employees') as employee_name,
    d.name as department_name,
    d.budget
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
-- Результат: показать все отделы, даже если в них нет сотрудников

-- Отделы без сотрудников
SELECT d.name, d.budget
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id
WHERE e.id IS NULL;
```

#### FULL OUTER JOIN: все записи
```sql
-- FULL OUTER JOIN возвращает ВСЕ записи из обеих таблиц
SELECT 
    COALESCE(e.name, 'No employee') as employee_name,
    COALESCE(d.name, 'No department') as department_name,
    e.salary,
    d.budget
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;
-- Результат: и сотрудники без отделов, и отделы без сотрудников
```

#### CROSS JOIN: декартово произведение
```sql
-- CROSS JOIN создает все возможные комбинации
SELECT 
    e.name as employee,
    d.name as department
FROM employees e
CROSS JOIN departments d;
-- Результат: каждый сотрудник со каждым отделом

-- Практическое применение: создание таблицы расписания
SELECT 
    d.day_of_week,
    t.time_slot
FROM (VALUES ('Monday'), ('Tuesday'), ('Wednesday')) d(day_of_week)
CROSS JOIN (VALUES ('09:00'), ('10:00'), ('11:00')) t(time_slot);
```

### Продвинутые техники JOIN

#### Самообъединения (Self JOIN)
```sql
-- Иерархия сотрудников (подчиненные и руководители)
ALTER TABLE employees ADD COLUMN manager_id INTEGER;

UPDATE employees SET manager_id = 1 WHERE id IN (3, 4);  -- Alice - менеджер

-- Поиск пар руководитель-подчиненный
SELECT 
    manager.name as manager_name,
    employee.name as employee_name,
    employee.salary
FROM employees employee
JOIN employees manager ON employee.manager_id = manager.id;

-- Сравнение с коллегами
SELECT 
    e1.name,
    e1.salary,
    e2.name as colleague,
    e2.salary as colleague_salary
FROM employees e1
JOIN employees e2 ON e1.department_id = e2.department_id 
WHERE e1.id <> e2.id  -- Исключаем самого себя
ORDER BY e1.name, e2.salary DESC;
```

#### Множественные JOIN
```sql
-- Сложная структура: сотрудники → отделы → проекты
CREATE TABLE projects (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department_id INTEGER REFERENCES departments(id)
);

CREATE TABLE employee_projects (
    employee_id INTEGER REFERENCES employees(id),
    project_id INTEGER REFERENCES projects(id),
    hours_per_week INTEGER,
    PRIMARY KEY (employee_id, project_id)
);

-- Многотабличный JOIN
SELECT 
    e.name as employee_name,
    d.name as department_name,
    p.name as project_name,
    ep.hours_per_week
FROM employees e
JOIN departments d ON e.department_id = d.id
JOIN employee_projects ep ON e.id = ep.employee_id
JOIN projects p ON ep.project_id = p.id
WHERE ep.hours_per_week > 20
ORDER BY e.name, p.name;
```

#### JOIN с условиями в ON vs WHERE
```sql
-- Условие в ON (влияет на объединение)
SELECT e.name, d.name, e.salary
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id AND d.budget > 200000;
-- Результат: сотрудники из "бедных" отделов получат d.name = NULL

-- Условие в WHERE (фильтрует результат)
SELECT e.name, d.name, e.salary
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id 
WHERE d.budget > 200000;
-- Результат: исключаются сотрудники из "бедных" отделов

-- Практический пример: активные заказы с деталями
SELECT 
    o.id as order_id,
    u.name as customer_name,
    p.name as product_name,
    oi.quantity
FROM orders o
JOIN users u ON o.user_id = u.id
LEFT JOIN order_items oi ON o.id = oi.order_id AND oi.quantity > 0  -- ON условие
JOIN products p ON oi.product_id = p.id
WHERE o.status = 'active';  -- WHERE условие
```

### Подзапросы (Subqueries)

#### Скалярные подзапросы
```sql
-- Подзапрос возвращает одно значение
SELECT 
    name,
    salary,
    (SELECT AVG(salary) FROM employees) as avg_salary,
    salary - (SELECT AVG(salary) FROM employees) as salary_diff
FROM employees;

-- Коррелированный скалярный подзапрос
SELECT 
    e.name,
    e.salary,
    (SELECT AVG(salary) 
     FROM employees e2 
     WHERE e2.department_id = e.department_id) as dept_avg_salary
FROM employees e;
```

#### Подзапросы в WHERE
```sql
-- EXISTS: проверка существования
SELECT name, salary
FROM employees e
WHERE EXISTS (
    SELECT 1 FROM employee_projects ep 
    WHERE ep.employee_id = e.id
);  -- Сотрудники, участвующие в проектах

-- NOT EXISTS: проверка отсутствия
SELECT name, salary
FROM employees e
WHERE NOT EXISTS (
    SELECT 1 FROM employee_projects ep 
    WHERE ep.employee_id = e.id
);  -- Сотрудники без проектов

-- IN с подзапросом
SELECT name, salary
FROM employees 
WHERE department_id IN (
    SELECT id FROM departments WHERE budget > 300000
);

-- Сравнение с подзапросом
SELECT name, salary
FROM employees 
WHERE salary > (
    SELECT AVG(salary) FROM employees
);  -- Зарплата выше средней

-- ALL и ANY
SELECT name, salary
FROM employees 
WHERE salary > ALL (
    SELECT salary FROM employees WHERE department_id = 2
);  -- Зарплата больше всех в отделе Sales

SELECT name, salary
FROM employees 
WHERE salary > ANY (
    SELECT salary FROM employees WHERE department_id = 1
);  -- Зарплата больше хотя бы одного в IT
```

#### Подзапросы в FROM (производные таблицы)
```sql
-- Подзапрос как временная таблица
SELECT 
    dept_stats.department_name,
    dept_stats.employee_count,
    dept_stats.avg_salary,
    dept_stats.total_payroll
FROM (
    SELECT 
        d.name as department_name,
        COUNT(e.id) as employee_count,
        AVG(e.salary) as avg_salary,
        SUM(e.salary) as total_payroll
    FROM departments d
    LEFT JOIN employees e ON d.id = e.department_id
    GROUP BY d.id, d.name
) dept_stats
WHERE dept_stats.employee_count > 1;

-- Ранжирование через подзапрос
SELECT 
    ranked_employees.name,
    ranked_employees.salary,
    ranked_employees.rank
FROM (
    SELECT 
        name,
        salary,
        ROW_NUMBER() OVER (ORDER BY salary DESC) as rank
    FROM employees
) ranked_employees
WHERE ranked_employees.rank <= 3;  -- Топ-3 по зарплате
```

---

## Неделя 6: Функции окна (Window Functions)

### Концепция оконных функций

**Window Functions** выполняют вычисления над набором строк, связанных с текущей строкой, но в отличие от GROUP BY не схлопывают результат в одну строку.

**Основные компоненты:**
- `OVER()` - определяет "окно" для вычислений
- `PARTITION BY` - разбивает данные на группы
- `ORDER BY` - определяет порядок в окне
- `ROWS/RANGE` - определяет границы окна

### Основы оконных функций

#### Базовый синтаксис OVER
```sql
-- Простейший пример: нумерация строк
SELECT 
    name,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- Сравнение с агрегатными функциями
SELECT 
    name,
    salary,
    COUNT(*) OVER () as total_employees,           -- Общее количество
    AVG(salary) OVER () as avg_salary,             -- Средняя зарплата
    salary - AVG(salary) OVER () as salary_diff    -- Отклонение от средней
FROM employees;
```

#### PARTITION BY: группировка в окнах
```sql
-- Разбивка по отделам
SELECT 
    name,
    department_id,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) as dept_rank,
    COUNT(*) OVER (PARTITION BY department_id) as dept_size,
    AVG(salary) OVER (PARTITION BY department_id) as dept_avg_salary
FROM employees
ORDER BY department_id, dept_rank;

-- Процентили внутри групп
SELECT 
    name,
    department_id,
    salary,
    PERCENT_RANK() OVER (PARTITION BY department_id ORDER BY salary) as salary_percentile
FROM employees;
```

### Ранжирующие функции

#### ROW_NUMBER(), RANK(), DENSE_RANK()
```sql
-- Различие между функциями ранжирования
SELECT 
    name,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) as row_num,    -- 1,2,3,4,5...
    RANK() OVER (ORDER BY salary DESC) as rank,             -- 1,2,2,4,5... (пропускает)
    DENSE_RANK() OVER (ORDER BY salary DESC) as dense_rank  -- 1,2,2,3,4... (не пропускает)
FROM employees;

-- Практический пример: топ-3 в каждом отделе
WITH ranked_employees AS (
    SELECT 
        name,
        department_id,
        salary,
        DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) as rank
    FROM employees
)
SELECT name, department_id, salary
FROM ranked_employees 
WHERE rank <= 3;
```

#### NTILE(): разбивка на процентили
```sql
-- Разбивка сотрудников на квартили по зарплате
SELECT 
    name,
    salary,
    NTILE(4) OVER (ORDER BY salary) as salary_quartile,
    CASE NTILE(4) OVER (ORDER BY salary)
        WHEN 1 THEN 'Bottom 25%'
        WHEN 2 THEN 'Lower Middle 25%'
        WHEN 3 THEN 'Upper Middle 25%'
        WHEN 4 THEN 'Top 25%'
    END as salary_category
FROM employees;

-- Децили для более детального анализа
SELECT 
    name,
    salary,
    NTILE(10) OVER (ORDER BY salary) as salary_decile
FROM employees;
```

### Агрегатные функции как оконные

#### SUM() OVER, COUNT() OVER
```sql
-- Накопительные суммы и проценты
SELECT 
    name,
    salary,
    SUM(salary) OVER (ORDER BY salary DESC) as cumulative_payroll,
    ROUND(
        100.0 * SUM(salary) OVER (ORDER BY salary DESC) / 
        SUM(salary) OVER (),
        2
    ) as cumulative_percent
FROM employees
ORDER BY salary DESC;

-- Скользящие средние
SELECT 
    name,
    hire_date,
    salary,
    AVG(salary) OVER (
        ORDER BY hire_date 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) as rolling_avg_3
FROM employees
ORDER BY hire_date;
```

#### Скользящие окна (ROWS BETWEEN)
```sql
-- Типы границ окон
SELECT 
    name,
    hire_date,
    salary,
    
    -- Все предыдущие строки + текущая
    SUM(salary) OVER (
        ORDER BY hire_date 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as cumulative_sum,
    
    -- 2 предыдущие + текущая + 1 следующая
    AVG(salary) OVER (
        ORDER BY hire_date 
        ROWS BETWEEN 2 PRECEDING AND 1 FOLLOWING
    ) as centered_avg,
    
    -- Только следующие строки
    MIN(salary) OVER (
        ORDER BY hire_date 
        ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING
    ) as min_future_salary
FROM employees
ORDER BY hire_date;
```

### Специальные оконные функции

#### LAG() и LEAD(): доступ к соседним строкам
```sql
-- Сравнение с предыдущими и следующими значениями
SELECT 
    name,
    hire_date,
    salary,
    LAG(salary) OVER (ORDER BY hire_date) as prev_salary,
    LEAD(salary) OVER (ORDER BY hire_date) as next_salary,
    
    -- Изменение зарплаты
    salary - LAG(salary) OVER (ORDER BY hire_date) as salary_change,
    
    -- Процентное изменение
    ROUND(
        100.0 * (salary - LAG(salary) OVER (ORDER BY hire_date)) / 
        NULLIF(LAG(salary) OVER (ORDER BY hire_date), 0),
        2
    ) as salary_change_percent
FROM employees
ORDER BY hire_date;

-- LAG с offset и default
SELECT 
    name,
    salary,
    LAG(salary, 2, 0) OVER (ORDER BY salary) as salary_2_positions_back
FROM employees;
```

#### FIRST_VALUE() и LAST_VALUE()
```sql
-- Сравнение с минимальными и максимальными значениями в группе
SELECT 
    name,
    department_id,
    salary,
    FIRST_VALUE(salary) OVER (
        PARTITION BY department_id 
        ORDER BY salary DESC
    ) as highest_salary_in_dept,
    
    LAST_VALUE(salary) OVER (
        PARTITION BY department_id 
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) as lowest_salary_in_dept,
    
    -- Разница с максимальной зарплатой в отделе
    FIRST_VALUE(salary) OVER (
        PARTITION BY department_id 
        ORDER BY salary DESC
    ) - salary as gap_from_top
FROM employees
ORDER BY department_id, salary DESC;
```

### Практические применения оконных функций

#### Анализ продаж по периодам
```sql
-- Предположим, у нас есть таблица продаж
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    sale_date DATE,
    amount DECIMAL(10,2),
    product_category VARCHAR(50)
);

-- Анализ трендов продаж
SELECT 
    sale_date,
    product_category,
    amount,
    
    -- Скользящая средняя за 7 дней
    AVG(amount) OVER (
        PARTITION BY product_category 
        ORDER BY sale_date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as rolling_avg_7d,
    
    -- Сравнение с предыдущим днем
    amount - LAG(amount) OVER (
        PARTITION BY product_category 
        ORDER BY sale_date
    ) as day_over_day_change,
    
    -- Процентиль продаж в категории
    PERCENT_RANK() OVER (
        PARTITION BY product_category 
        ORDER BY amount
    ) as percentile_in_category,
    
    -- Максимальная продажа за весь период в категории
    MAX(amount) OVER (PARTITION BY product_category) as category_max
FROM sales
ORDER BY product_category, sale_date;
```

---

## Неделя 7: Общие табличные выражения (CTE)

### Концепция CTE

**Common Table Expressions (CTE)** - это именованные временные результирующие наборы, которые существуют только в рамках выполнения одного запроса. CTE делают сложные запросы более читаемыми и позволяют разбить логику на этапы.

**Преимущества CTE:**
- Улучшают читаемость сложных запросов
- Позволяют переиспользовать результаты в одном запросе
- Поддерживают рекурсию
- Могут заменить сложные подзапросы

### Простые CTE

#### Синтаксис WITH
```sql
-- Базовый синтаксис CTE
WITH department_stats AS (
    SELECT 
        department_id,
        COUNT(*) as employee_count,
        AVG(salary) as avg_salary,
        SUM(salary) as total_payroll
    FROM employees
    GROUP BY department_id
)
SELECT 
    d.name as department_name,
    ds.employee_count,
    ds.avg_salary,
    ds.total_payroll,
    d.budget,
    (d.budget - ds.total_payroll) as budget_remaining
FROM department_stats ds
JOIN departments d ON ds.department_id = d.id
WHERE ds.employee_count > 1;
```

#### Множественные CTE в одном запросе
```sql
-- Несколько CTE, использующих друг друга
WITH 
-- CTE 1: Статистика по отделам
department_stats AS (
    SELECT 
        department_id,
        COUNT(*) as employee_count,
        AVG(salary) as avg_salary,
        SUM(salary) as total_payroll
    FROM employees
    GROUP BY department_id
),
-- CTE 2: Общая статистика
company_stats AS (
    SELECT 
        AVG(avg_salary) as company_avg_salary,
        SUM(total_payroll) as company_total_payroll
    FROM department_stats
),
-- CTE 3: Сравнительный анализ
department_comparison AS (
    SELECT 
        ds.*,
        cs.company_avg_salary,
        ds.avg_salary - cs.company_avg_salary as avg_salary_diff,
        ROUND(100.0 * ds.total_payroll / cs.company_total_payroll, 2) as payroll_percent
    FROM department_stats ds
    CROSS JOIN company_stats cs
)
-- Финальный запрос
SELECT 
    d.name as department_name,
    dc.employee_count,
    ROUND(dc.avg_salary, 2) as avg_salary,
    ROUND(dc.company_avg_salary, 2) as company_avg,
    ROUND(dc.avg_salary_diff, 2) as diff_from_company_avg,
    dc.payroll_percent || '%' as payroll_share
FROM department_comparison dc
JOIN departments d ON dc.department_id = d.id
ORDER BY dc.avg_salary DESC;
```

#### CTE vs подзапросы
```sql
-- ❌ Сложный запрос с повторяющимися подзапросами (плохо)
SELECT 
    e.name,
    e.salary,
    (SELECT AVG(salary) FROM employees e2 WHERE e2.department_id = e.department_id) as dept_avg,
    e.salary - (SELECT AVG(salary) FROM employees e2 WHERE e2.department_id = e.department_id) as diff_from_dept_avg,
    (SELECT COUNT(*) FROM employees e2 WHERE e2.department_id = e.department_id) as dept_size
FROM employees e;

-- ✅ Тот же запрос с CTE (хорошо)
WITH department_averages AS (
    SELECT 
        department_id,
        AVG(salary) as avg_salary,
        COUNT(*) as employee_count
    FROM employees
    GROUP BY department_id
)
SELECT 
    e.name,
    e.salary,
    da.avg_salary as dept_avg,
    e.salary - da.avg_salary as diff_from_dept_avg,
    da.employee_count as dept_size
FROM employees e
JOIN department_averages da ON e.department_id = da.department_id;
```

### Рекурсивные CTE

#### Понимание рекурсии в SQL
Рекурсивные CTE состоят из двух частей:
1. **Anchor query** (якорный запрос) - начальное условие
2. **Recursive query** (рекурсивный запрос) - ссылается на само CTE

#### Структура рекурсивных CTE
```sql
WITH RECURSIVE cte_name AS (
    -- Якорный запрос (начальные данные)
    SELECT initial_columns
    FROM initial_table
    WHERE initial_condition
    
    UNION ALL
    
    -- Рекурсивный запрос
    SELECT recursive_columns
    FROM some_table
    JOIN cte_name ON join_condition
    WHERE recursive_condition
)
SELECT * FROM cte_name;
```

#### Иерархии организации
```sql
-- Создадим иерархическую структуру сотрудников
ALTER TABLE employees ADD COLUMN manager_id INTEGER;

UPDATE employees SET manager_id = NULL WHERE id = 1;  -- CEO
UPDATE employees SET manager_id = 1 WHERE id IN (2, 3);  -- Подчиненные CEO
UPDATE employees SET manager_id = 2 WHERE id = 4;  -- Подчиненный Bob
UPDATE employees SET manager_id = 3 WHERE id = 5;  -- Подчиненный Carol

-- Рекурсивный CTE для построения иерархии
WITH RECURSIVE employee_hierarchy AS (
    -- Якорный запрос: топ-менеджеры (без руководителя)
    SELECT 
        id,
        name,
        manager_id,
        salary,
        0 as level,
        name as path,
        ARRAY[id] as id_path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Рекурсивный запрос: подчиненные
    SELECT 
        e.id,
        e.name,
        e.manager_id,
        e.salary,
        eh.level + 1,
        eh.path || ' -> ' || e.name,
        eh.id_path || e.id
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT 
    REPEAT('  ', level) || name as indented_name,
    level,
    salary,
    path as hierarchy_path
FROM employee_hierarchy
ORDER BY id_path;
```

#### Генерация последовательностей
```sql
-- Генерация дат для отчетов
WITH RECURSIVE date_series AS (
    -- Якорный запрос: начальная дата
    SELECT DATE '2024-01-01' as report_date
    
    UNION ALL
    
    -- Рекурсивный запрос: следующие даты
    SELECT report_date + INTERVAL '1 day'
    FROM date_series
    WHERE report_date < DATE '2024-01-31'
)
SELECT 
    report_date,
    EXTRACT(DOW FROM report_date) as day_of_week,
    CASE EXTRACT(DOW FROM report_date)
        WHEN 0 THEN 'Sunday'
        WHEN 1 THEN 'Monday'
        WHEN 2 THEN 'Tuesday'
        WHEN 3 THEN 'Wednesday'
        WHEN 4 THEN 'Thursday'
        WHEN 5 THEN 'Friday'
        WHEN 6 THEN 'Saturday'
    END as day_name
FROM date_series
ORDER BY report_date;

-- Числовая последовательность
WITH RECURSIVE numbers AS (
    SELECT 1 as n
    UNION ALL
    SELECT n + 1
    FROM numbers
    WHERE n < 100
)
SELECT n, n * n as squared, n * n * n as cubed
FROM numbers
WHERE n <= 10;
```

### Практические применения CTE

#### Анализ продаж с накопительными итогами
```sql
-- Создадим таблицу продаж для примера
CREATE TABLE monthly_sales (
    month DATE PRIMARY KEY,
    revenue DECIMAL(10,2)
);

INSERT INTO monthly_sales VALUES 
    ('2024-01-01', 150000),
    ('2024-02-01', 180000),
    ('2024-03-01', 165000),
    ('2024-04-01', 195000),
    ('2024-05-01', 210000);

-- CTE для расчета накопительных итогов и трендов
WITH sales_analysis AS (
    SELECT 
        month,
        revenue,
        LAG(revenue) OVER (ORDER BY month) as prev_month_revenue,
        SUM(revenue) OVER (ORDER BY month) as cumulative_revenue,
        AVG(revenue) OVER (ORDER BY month) as avg_revenue_to_date
    FROM monthly_sales
),
sales_growth AS (
    SELECT 
        *,
        CASE 
            WHEN prev_month_revenue IS NULL THEN NULL
            ELSE ROUND(
                100.0 * (revenue - prev_month_revenue) / prev_month_revenue, 
                2
            )
        END as month_over_month_growth,
        revenue - avg_revenue_to_date as deviation_from_avg
    FROM sales_analysis
)
SELECT 
    TO_CHAR(month, 'YYYY-MM') as month,
    revenue,
    cumulative_revenue,
    month_over_month_growth || '%' as growth_rate,
    CASE 
        WHEN month_over_month_growth > 10 THEN 'High Growth'
        WHEN month_over_month_growth > 0 THEN 'Growth'
        WHEN month_over_month_growth > -10 THEN 'Slight Decline'
        ELSE 'Significant Decline'
    END as growth_category
FROM sales_growth
ORDER BY month;
```

#### Многоуровневые агрегации
```sql
-- CTE для анализа структуры заказов
WITH order_details AS (
    -- Детали заказов с вычисляемыми полями
    SELECT 
        o.id as order_id,
        o.user_id,
        o.created_at,
        oi.product_id,
        oi.quantity,
        oi.price,
        oi.quantity * oi.price as line_total,
        p.name as product_name,
        p.category_id
    FROM orders o
    JOIN order_items oi ON o.id = oi.order_id
    JOIN products p ON oi.product_id = p.id
),
order_summaries AS (
    -- Агрегация по заказам
    SELECT 
        order_id,
        user_id,
        created_at,
        COUNT(*) as items_count,
        SUM(quantity) as total_quantity,
        SUM(line_total) as order_total,
        STRING_AGG(product_name, ', ') as products_list
    FROM order_details
    GROUP BY order_id, user_id, created_at
),
user_summaries AS (
    -- Агрегация по пользователям
    SELECT 
        user_id,
        COUNT(*) as orders_count,
        SUM(order_total) as lifetime_value,
        AVG(order_total) as avg_order_value,
        MIN(created_at) as first_order_date,
        MAX(created_at) as last_order_date
    FROM order_summaries
    GROUP BY user_id
)
-- Финальный отчет с классификацией клиентов
SELECT 
    u.name as customer_name,
    us.orders_count,
    us.lifetime_value,
    us.avg_order_value,
    us.first_order_date,
    us.last_order_date,
    CASE 
        WHEN us.lifetime_value > 10000 THEN 'VIP'
        WHEN us.lifetime_value > 5000 THEN 'High Value'
        WHEN us.lifetime_value > 1000 THEN 'Regular'
        ELSE 'New'
    END as customer_segment,
    CASE 
        WHEN us.last_order_date > NOW() - INTERVAL '30 days' THEN 'Active'
        WHEN us.last_order_date > NOW() - INTERVAL '90 days' THEN 'At Risk'
        ELSE 'Inactive'
    END as activity_status
FROM user_summaries us
JOIN users u ON us.user_id = u.id
ORDER BY us.lifetime_value DESC;
```

---

# ФАЗА 3: POSTGRESQL-СПЕЦИФИЧНЫЕ ВОЗМОЖНОСТИ (Недели 8-10)

## Неделя 8: Индексы и производительность

### Концепция индексов

**Индекс** - это структура данных, которая улучшает скорость операций поиска в таблице за счет создания быстрого пути к данным. Это как алфавитный указатель в книге.

**Когда нужны индексы:**
- Колонки в WHERE условиях
- Колонки в JOIN условиях  
- Колонки в ORDER BY
- Колонки с UNIQUE ограничениями
- Внешние ключи

**Стоимость индексов:**
- Дополнительное место на диске
- Замедление INSERT/UPDATE/DELETE операций
- Необходимость обслуживания

### Типы индексов в PostgreSQL

#### B-tree индексы (по умолчанию)
```sql
-- Простой B-tree индекс
CREATE INDEX idx_users_email ON users(email);

-- Композитный (многоколоночный) индекс
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);

-- Порядок колонок в композитном индексе важен!
-- Хорошо работает для:
-- WHERE user_id = 123
-- WHERE user_id = 123 AND created_at > '2024-01-01'
-- 
-- Плохо работает для:
-- WHERE created_at > '2024-01-01' (без user_id)

-- Индекс с сортировкой
CREATE INDEX idx_products_price_desc ON products(price DESC);

-- Индекс с NULL обработкой
CREATE INDEX idx_users_last_login ON users(last_login DESC NULLS LAST);

-- Функциональный индекс
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
-- Позволяет эффективно искать: WHERE LOWER(email) = 'user@example.com'

-- Выражения в индексах
CREATE INDEX idx_orders_total_with_tax ON orders((total * 1.2));
-- Для запросов: WHERE total * 1.2 > 1000
```

#### Hash индексы
```sql
-- Hash индексы только для точного равенства
CREATE INDEX idx_products_sku_hash ON products USING hash(sku);

-- Хорошо для: WHERE sku = 'ABC123'
-- Плохо для: WHERE sku LIKE 'ABC%' или WHERE sku > 'ABC'

-- Hash индексы занимают меньше места, но менее универсальны
```

#### GIN индексы (для составных значений)
```sql
-- GIN для полнотекстового поиска
CREATE INDEX idx_products_search ON products USING gin(to_tsvector('english', name));

-- Поиск по тексту
SELECT * FROM products 
WHERE to_tsvector('english', name) @@ to_tsquery('laptop & gaming');

-- GIN для массивов
ALTER TABLE users ADD COLUMN tags TEXT[];
CREATE INDEX idx_users_tags ON users USING gin(tags);

-- Поиск в массивах