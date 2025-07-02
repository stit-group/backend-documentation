# PostgreSQL: Полное руководство для Backend разработчика
*Структурированный курс от основ до экспертного уровня*

## Введение: Зачем PostgreSQL backend разработчику?

**PostgreSQL** - это не просто база данных, это мощная платформа для создания надежных backend систем:

- **ACID транзакции**: Гарантия целостности данных в любых условиях
- **Расширяемость**: JSON, массивы, custom типы, расширения
- **Производительность**: Развитая система индексирования и оптимизации
- **Надежность**: Проверено годами в enterprise окружении
- **Open Source**: Бесплатно, активное сообщество, постоянное развитие

---

# БЛОК 1: ОСНОВЫ SQL И POSTGRESQL

## Глава 1: Фундаментальные концепции

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

## Глава 2: Основные SQL операции

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

## Глава 3: Расширенные SELECT запросы

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

# БЛОК 2: ПРОДВИНУТЫЙ SQL

## Глава 4: Объединения таблиц (JOINs)

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

## Глава 5: Функции окна (Window Functions)

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

## Глава 6: Общие табличные выражения (CTE)

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

# БЛОК 3: POSTGRESQL-СПЕЦИФИЧНЫЕ ВОЗМОЖНОСТИ

## Глава 7: Индексы и производительность

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
SELECT * FROM users WHERE tags @> ARRAY['premium'];
SELECT * FROM users WHERE tags && ARRAY['vip', 'premium'];

-- GIN для JSONB
CREATE INDEX idx_user_profiles_preferences ON user_profiles USING gin(preferences);

-- Поиск в JSONB
SELECT * FROM user_profiles WHERE preferences @> '{"theme": "dark"}';
SELECT * FROM user_profiles WHERE preferences ? 'notifications';
```

#### GiST индексы (геометрические и полнотекстовые)
```sql
-- GiST для геометрических данных
CREATE TABLE locations (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    coordinates POINT
);

CREATE INDEX idx_locations_coordinates ON locations USING gist(coordinates);

-- Поиск по расстоянию
SELECT name FROM locations 
WHERE coordinates <-> point(37.7749, -122.4194) < 10;

-- GiST для полнотекстового поиска (альтернатива GIN)
CREATE INDEX idx_products_search_gist ON products USING gist(to_tsvector('english', name));
```

#### Частичные индексы
```sql
-- Индекс только для активных пользователей
CREATE INDEX idx_users_active_email ON users(email) 
WHERE status = 'active';

-- Индекс только для больших заказов
CREATE INDEX idx_orders_large_total ON orders(created_at, user_id) 
WHERE total > 1000;

-- Индекс исключающий NULL
CREATE INDEX idx_users_last_login_not_null ON users(last_login) 
WHERE last_login IS NOT NULL;
```

### Анализ производительности запросов

#### EXPLAIN и EXPLAIN ANALYZE
```sql
-- Показать план выполнения
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- План с реальным временем выполнения
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';

-- Подробный анализ
EXPLAIN (ANALYZE, BUFFERS, VERBOSE) 
SELECT u.name, COUNT(o.id) 
FROM users u 
LEFT JOIN orders o ON u.id = o.user_id 
GROUP BY u.id, u.name 
HAVING COUNT(o.id) > 5;

-- Форматы вывода
EXPLAIN (FORMAT JSON) SELECT * FROM users WHERE age > 25;
EXPLAIN (FORMAT YAML) SELECT * FROM users WHERE age > 25;
EXPLAIN (FORMAT XML) SELECT * FROM users WHERE age > 25;
```

#### Интерпретация планов выполнения
```sql
-- Пример плохого плана (Seq Scan)
EXPLAIN ANALYZE 
SELECT * FROM orders WHERE user_id = 123;
/*
Seq Scan on orders  (cost=0.00..2125.00 rows=100 width=32) (actual time=0.123..24.567 rows=5 loops=1)
  Filter: (user_id = 123)
  Rows Removed by Filter: 99995
Planning Time: 0.234 ms
Execution Time: 24.789 ms
*/

-- Создаем индекс
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Хороший план (Index Scan)
EXPLAIN ANALYZE 
SELECT * FROM orders WHERE user_id = 123;
/*
Index Scan using idx_orders_user_id on orders  (cost=0.29..8.31 rows=1 width=32) (actual time=0.015..0.017 rows=5 loops=1)
  Index Cond: (user_id = 123)
Planning Time: 0.123 ms
Execution Time: 0.045 ms
*/
```

#### Статистика таблиц
```sql
-- Обновление статистики
ANALYZE users;
ANALYZE orders;

-- Автоматическое обновление (настройка)
-- В postgresql.conf:
-- autovacuum = on
-- autovacuum_analyze_scale_factor = 0.1

-- Просмотр статистики
SELECT 
    schemaname,
    tablename,
    n_live_tup as live_rows,
    n_dead_tup as dead_rows,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
ORDER BY n_live_tup DESC;
```

### Оптимизация запросов

#### Избегание проблемных паттернов
```sql
-- ❌ Плохо: функция в WHERE (индекс не используется)
SELECT * FROM users WHERE UPPER(email) = 'TEST@EXAMPLE.COM';

-- ✅ Хорошо: функциональный индекс
CREATE INDEX idx_users_email_upper ON users(UPPER(email));
SELECT * FROM users WHERE UPPER(email) = 'TEST@EXAMPLE.COM';

-- ❌ Плохо: LIKE с ведущим %
SELECT * FROM products WHERE name LIKE '%laptop%';

-- ✅ Хорошо: полнотекстовый поиск
CREATE INDEX idx_products_search ON products USING gin(to_tsvector('english', name));
SELECT * FROM products WHERE to_tsvector('english', name) @@ to_tsquery('laptop');

-- ❌ Плохо: OR условия
SELECT * FROM users WHERE status = 'active' OR status = 'pending';

-- ✅ Хорошо: IN список
SELECT * FROM users WHERE status IN ('active', 'pending');

-- ❌ Плохо: подзапрос в SELECT (выполняется для каждой строки)
SELECT 
    u.name,
    (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) as order_count
FROM users u;

-- ✅ Хорошо: JOIN с агрегацией
SELECT 
    u.name,
    COALESCE(o.order_count, 0) as order_count
FROM users u
LEFT JOIN (
    SELECT user_id, COUNT(*) as order_count 
    FROM orders 
    GROUP BY user_id
) o ON u.id = o.user_id;
```

#### Оптимизация JOIN операций
```sql
-- Правильный порядок JOIN (меньшие таблицы первыми)
SELECT u.name, p.name, oi.quantity
FROM order_items oi              -- Самая большая таблица
JOIN products p ON oi.product_id = p.id    -- Средняя таблица
JOIN orders o ON oi.order_id = o.id        -- Меньшая таблица
JOIN users u ON o.user_id = u.id           -- Самая маленькая таблица
WHERE o.created_at > '2024-01-01';

-- Использование EXISTS вместо IN для больших наборов
-- ❌ Медленно для больших результатов подзапроса
SELECT * FROM users 
WHERE id IN (SELECT user_id FROM orders WHERE total > 1000);

-- ✅ Быстрее
SELECT * FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id AND o.total > 1000);
```

### Управление индексами

#### Мониторинг использования индексов
```sql
-- Статистика использования индексов
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan as scans,
    idx_tup_read as tuples_read,
    idx_tup_fetch as tuples_fetched
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Неиспользуемые индексы
SELECT 
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) as size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;

-- Размеры индексов
SELECT 
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) as size
FROM pg_stat_user_indexes
ORDER BY pg_relation_size(indexrelid) DESC;
```

#### Обслуживание индексов
```sql
-- Пересборка индекса
REINDEX INDEX idx_users_email;

-- Пересборка всех индексов таблицы
REINDEX TABLE users;

-- Конкурентная пересборка (не блокирует таблицу)
CREATE INDEX CONCURRENTLY idx_users_email_new ON users(email);
DROP INDEX idx_users_email;
ALTER INDEX idx_users_email_new RENAME TO idx_users_email;

-- Удаление неиспользуемых индексов
DROP INDEX IF EXISTS idx_unused_index;
```

---

## Глава 8: Транзакции и параллелизм

### Основы транзакций

#### ACID свойства
```sql
-- Atomicity (Атомарность) - все или ничего
BEGIN;
    INSERT INTO accounts (name, balance) VALUES ('Alice', 1000);
    INSERT INTO accounts (name, balance) VALUES ('Bob', 500);
    -- Если одна команда упадет, откатятся обе
COMMIT;

-- Consistency (Согласованность) - данные остаются валидными
BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE name = 'Alice';
    UPDATE accounts SET balance = balance + 100 WHERE name = 'Bob';
    -- Общая сумма остается неизменной
COMMIT;

-- Isolation (Изолированность) - транзакции не видят промежуточных состояний
-- Durability (Долговечность) - сохраненные данные не теряются
```

#### Базовый синтаксис транзакций
```sql
-- Простая транзакция
BEGIN;
    INSERT INTO orders (user_id, total) VALUES (1, 100.00);
    INSERT INTO order_items (order_id, product_id, quantity) VALUES (LASTVAL(), 1, 2);
COMMIT;

-- Транзакция с откатом
BEGIN;
    UPDATE products SET stock = stock - 5 WHERE id = 1;
    -- Проверяем результат
    SELECT stock FROM products WHERE id = 1;
    -- Если что-то не так, откатываем
ROLLBACK;

-- Точки сохранения (Savepoints)
BEGIN;
    INSERT INTO users (email, name) VALUES ('user1@example.com', 'User 1');
    
    SAVEPOINT sp1;
    INSERT INTO users (email, name) VALUES ('user2@example.com', 'User 2');
    INSERT INTO users (email, name) VALUES ('invalid', 'Invalid User');  -- Может упасть
    
    ROLLBACK TO sp1;  -- Откатываемся только до savepoint
    INSERT INTO users (email, name) VALUES ('user3@example.com', 'User 3');
COMMIT;
```

#### Автоматические транзакции
```sql
-- PostgreSQL автоматически оборачивает каждую команду в транзакцию
INSERT INTO users (email, name) VALUES ('auto@example.com', 'Auto User');
-- Эквивалентно:
-- BEGIN;
-- INSERT INTO users (email, name) VALUES ('auto@example.com', 'Auto User');
-- COMMIT;

-- Отключение автокоммита (в psql)
\set AUTOCOMMIT off
INSERT INTO users (email, name) VALUES ('manual@example.com', 'Manual User');
-- Нужен явный COMMIT

-- Включение обратно
\set AUTOCOMMIT on
```

### Уровни изоляции

#### READ UNCOMMITTED
```sql
-- Самый низкий уровень изоляции
-- Может читать незакоммиченные данные (dirty reads)
BEGIN ISOLATION LEVEL READ UNCOMMITTED;
    SELECT balance FROM accounts WHERE name = 'Alice';
    -- Может увидеть изменения из других незавершенных транзакций
COMMIT;
```

#### READ COMMITTED (по умолчанию)
```sql
-- Читает только закоммиченные данные
-- Но может видеть разные значения при повторном чтении
BEGIN ISOLATION LEVEL READ COMMITTED;
    SELECT balance FROM accounts WHERE name = 'Alice';  -- 1000
    -- Другая транзакция изменила и закоммитила
    SELECT balance FROM accounts WHERE name = 'Alice';  -- 900 (non-repeatable read)
COMMIT;
```

#### REPEATABLE READ
```sql
-- Гарантирует постоянство данных в рамках транзакции
BEGIN ISOLATION LEVEL REPEATABLE READ;
    SELECT balance FROM accounts WHERE name = 'Alice';  -- 1000
    -- Другая транзакция изменила и закоммитила
    SELECT balance FROM accounts WHERE name = 'Alice';  -- Все еще 1000
    -- Но может увидеть новые строки (phantom reads)
COMMIT;
```

#### SERIALIZABLE
```sql
-- Самый высокий уровень изоляции
-- Эмулирует последовательное выполнение транзакций
BEGIN ISOLATION LEVEL SERIALIZABLE;
    SELECT SUM(balance) FROM accounts;
    INSERT INTO accounts (name, balance) VALUES ('Charlie', 200);
    -- Если другая транзакция тоже работает с accounts,
    -- одна из них может быть отменена с ошибкой serialization
COMMIT;
```

### Блокировки

#### Типы блокировок
```sql
-- Явная блокировка строк
BEGIN;
    SELECT * FROM accounts WHERE name = 'Alice' FOR UPDATE;
    -- Строка заблокирована до конца транзакции
    UPDATE accounts SET balance = balance - 100 WHERE name = 'Alice';
COMMIT;

-- Блокировка для чтения
BEGIN;
    SELECT * FROM accounts WHERE name = 'Alice' FOR SHARE;
    -- Другие могут читать, но не могут изменять
COMMIT;

-- Неблокирующая блокировка
BEGIN;
    SELECT * FROM accounts WHERE name = 'Alice' FOR UPDATE NOWAIT;
    -- Если строка заблокирована, сразу ошибка
COMMIT;

-- Пропуск заблокированных строк
BEGIN;
    SELECT * FROM accounts FOR UPDATE SKIP LOCKED;
    -- Возвращает только незаблокированные строки
COMMIT;
```

#### Блокировки таблиц
```sql
-- Различные уровни блокировки таблиц
BEGIN;
    LOCK TABLE accounts IN ACCESS EXCLUSIVE MODE;
    -- Никто не может читать или писать
    TRUNCATE accounts;
COMMIT;

BEGIN;
    LOCK TABLE accounts IN SHARE MODE;
    -- Разрешает чтение, запрещает изменения
    SELECT COUNT(*) FROM accounts;
COMMIT;

-- Неблокирующая блокировка таблицы
BEGIN;
    LOCK TABLE accounts IN ACCESS SHARE MODE NOWAIT;
COMMIT;
```

### Deadlocks и их предотвращение

#### Пример deadlock
```sql
-- Транзакция 1:
BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE name = 'Alice';
    -- Ждет блокировку Bob (заблокирован транзакцией 2)
    UPDATE accounts SET balance = balance + 100 WHERE name = 'Bob';
COMMIT;

-- Транзакция 2 (выполняется одновременно):
BEGIN;
    UPDATE accounts SET balance = balance - 50 WHERE name = 'Bob';
    -- Ждет блокировку Alice (заблокирован транзакцией 1)
    UPDATE accounts SET balance = balance + 50 WHERE name = 'Alice';
COMMIT;
-- PostgreSQL автоматически обнаружит deadlock и откатит одну транзакцию
```

#### Стратегии предотвращения deadlocks
```sql
-- 1. Согласованный порядок блокировок
-- ✅ Всегда блокируем accounts в порядке ID
BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- Alice (меньший ID)
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- Bob (больший ID)
COMMIT;

-- 2. Использование SELECT FOR UPDATE
BEGIN;
    SELECT * FROM accounts WHERE name IN ('Alice', 'Bob') 
    ORDER BY id FOR UPDATE;  -- Блокируем в определенном порядке
    
    UPDATE accounts SET balance = balance - 100 WHERE name = 'Alice';
    UPDATE accounts SET balance = balance + 100 WHERE name = 'Bob';
COMMIT;

-- 3. Короткие транзакции
-- ❌ Длинная транзакция
BEGIN;
    SELECT * FROM accounts WHERE name = 'Alice' FOR UPDATE;
    -- Много сложной логики...
    UPDATE accounts SET balance = balance - 100 WHERE name = 'Alice';
COMMIT;

-- ✅ Короткая транзакция
-- Делаем вычисления вне транзакции
BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE name = 'Alice';
COMMIT;
```

### Мониторинг блокировок

#### Просмотр активных блокировок
```sql
-- Активные блокировки
SELECT 
    l.locktype,
    l.database,
    l.relation::regclass as table_name,
    l.page,
    l.tuple,
    l.mode,
    l.granted,
    a.query,
    a.pid,
    a.usename
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE NOT l.granted
ORDER BY l.relation, l.page, l.tuple;

-- Блокирующие и заблокированные запросы
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_statement,
    blocking_activity.query AS blocking_statement
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
    AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
    AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

#### Завершение блокирующих процессов
```sql
-- Мягкое завершение
SELECT pg_cancel_backend(pid) FROM pg_stat_activity WHERE pid = 12345;

-- Принудительное завершение
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid = 12345;

-- Завершение всех неактивных соединений
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE state = 'idle in transaction'
AND state_change < NOW() - INTERVAL '10 minutes';
```

---

## Глава 9: Расширенные типы данных PostgreSQL

### Массивы

#### Создание и работа с массивами
```sql
-- Создание таблицы с массивами
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255),
    tags TEXT[],
    ratings INTEGER[],
    metadata JSONB
);

-- Вставка данных с массивами
INSERT INTO articles (title, tags, ratings) VALUES 
    ('PostgreSQL Tutorial', ARRAY['database', 'sql', 'tutorial'], ARRAY[5, 4, 5, 4]),
    ('Advanced PostgreSQL', ARRAY['database', 'advanced'], ARRAY[5, 5, 4]),
    ('JSON in PostgreSQL', ARRAY['database', 'json', 'nosql'], ARRAY[4, 5, 5]);

-- Альтернативный синтаксис
INSERT INTO articles (title, tags, ratings) VALUES 
    ('Array Operations', '{"arrays", "postgresql"}', '{3, 4, 5, 4}');
```

#### Операции с массивами
```sql
-- Доступ к элементам (индексы начинаются с 1)
SELECT 
    title,
    tags[1] as first_tag,
    tags[2] as second_tag,
    array_length(tags, 1) as tag_count
FROM articles;

-- Изменение элементов
UPDATE articles 
SET tags[1] = 'databases'
WHERE id = 1;

-- Добавление элементов
UPDATE articles 
SET tags = array_append(tags, 'beginner')
WHERE title = 'PostgreSQL Tutorial';

-- Удаление элементов
UPDATE articles 
SET tags = array_remove(tags, 'nosql')
WHERE id = 3;

-- Конкатенация массивов
UPDATE articles 
SET tags = tags || ARRAY['popular']
WHERE array_length(ratings, 1) > 3;
```

#### Поиск в массивах
```sql
-- Поиск по содержанию элемента
SELECT title, tags
FROM articles
WHERE 'database' = ANY(tags);

-- Поиск пересечения массивов
SELECT title, tags
FROM articles
WHERE tags && ARRAY['database', 'advanced'];

-- Поиск включения массива
SELECT title, tags
FROM articles
WHERE tags @> ARRAY['database', 'sql'];

-- Поиск по позиции элемента
SELECT title, tags
FROM articles
WHERE array_position(tags, 'database') = 1;

-- Агрегация массивов
SELECT 
    array_agg(DISTINCT unnest) as all_unique_tags
FROM (
    SELECT unnest(tags) FROM articles
) t;
```

### JSONB и расширенные JSON операции

#### Создание и индексирование JSONB
```sql
-- Таблица с JSONB данными
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    details JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Индексы для JSONB
CREATE INDEX idx_products_details_gin ON products USING gin(details);
CREATE INDEX idx_products_brand ON products USING btree((details->>'brand'));
CREATE INDEX idx_products_price ON products USING btree(((details->>'price')::numeric));

-- Вставка JSONB данных
INSERT INTO products (name, details) VALUES 
    ('Laptop Pro', '{"brand": "TechCorp", "price": 1299.99, "specs": {"cpu": "Intel i7", "ram": 16, "storage": "512GB SSD"}, "tags": ["laptop", "professional"]}'),
    ('Gaming Mouse', '{"brand": "GamerCorp", "price": 79.99, "specs": {"dpi": 12000, "buttons": 8}, "tags": ["mouse", "gaming"]}'),
    ('Monitor 4K', '{"brand": "DisplayCorp", "price": 549.99, "specs": {"size": 27, "resolution": "4K", "panel": "IPS"}, "tags": ["monitor", "4k"]}');
```

#### Расширенные JSONB операции
```sql
-- Извлечение данных разными способами
SELECT 
    name,
    details->>'brand' as brand,
    (details->>'price')::numeric as price,
    details->'specs'->>'cpu' as cpu,
    details->'tags' as tags_json,
    jsonb_array_elements_text(details->'tags') as individual_tags
FROM products;

-- Фильтрация по JSONB
SELECT name, details->>'price' as price
FROM products
WHERE (details->>'price')::numeric > 100;

SELECT name, details->>'brand' as brand
FROM products
WHERE details @> '{"brand": "TechCorp"}';

-- Поиск в массивах внутри JSONB
SELECT name, details->'tags' as tags
FROM products
WHERE details->'tags' ? 'gaming';

-- Проверка существования путей
SELECT name, details
FROM products
WHERE details ? 'specs' AND details->'specs' ? 'cpu';
```

#### Модификация JSONB
```sql
-- Добавление/обновление полей
UPDATE products 
SET details = details || '{"warranty": "2 years"}'
WHERE id = 1;

-- Обновление вложенных полей
UPDATE products 
SET details = jsonb_set(details, '{specs, ram}', '32')
WHERE name = 'Laptop Pro';

-- Удаление полей
UPDATE products 
SET details = details - 'warranty'
WHERE id = 1;

-- Удаление вложенных полей
UPDATE products 
SET details = details #- '{specs, storage}'
WHERE name = 'Laptop Pro';

-- Добавление элементов в массив
UPDATE products 
SET details = jsonb_set(
    details, 
    '{tags}', 
    (details->'tags') || '"premium"'::jsonb
)
WHERE (details->>'price')::numeric > 1000;
```

#### Агрегация и аналитика JSONB
```sql
-- Статистика по брендам
SELECT 
    details->>'brand' as brand,
    COUNT(*) as product_count,
    AVG((details->>'price')::numeric) as avg_price,
    MIN((details->>'price')::numeric) as min_price,
    MAX((details->>'price')::numeric) as max_price
FROM products
GROUP BY details->>'brand'
ORDER BY avg_price DESC;

-- Популярные теги
SELECT 
    tag,
    COUNT(*) as frequency
FROM products,
     jsonb_array_elements_text(details->'tags') as tag
GROUP BY tag
ORDER BY frequency DESC;

-- Агрегация объектов
SELECT jsonb_agg(
    jsonb_build_object(
        'name', name,
        'brand', details->>'brand',
        'price', details->>'price'
    )
) as products_summary
FROM products
WHERE (details->>'price')::numeric < 500;
```

### Пользовательские типы данных

#### Создание составных типов
```sql
-- Определение составного типа
CREATE TYPE address_type AS (
    street VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(50),
    zip_code VARCHAR(20),
    country VARCHAR(50)
);

CREATE TYPE contact_info AS (
    email VARCHAR(255),
    phone VARCHAR(20),
    address address_type
);

-- Использование составных типов
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    contact contact_info,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Вставка данных
INSERT INTO customers (name, contact) VALUES 
    ('John Doe', ROW('john@example.com', '+1-555-0123', ROW('123 Main St', 'New York', 'NY', '10001', 'USA')));

-- Доступ к полям составного типа
SELECT 
    name,
    (contact).email,
    (contact.address).city,
    (contact.address).country
FROM customers;
```

#### Создание перечислений (ENUM)
```sql
-- Создание ENUM типа
CREATE TYPE order_status AS ENUM ('pending', 'processing', 'shipped', 'delivered', 'cancelled');
CREATE TYPE priority_level AS ENUM ('low', 'medium', 'high', 'critical');

-- Использование ENUM
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    status order_status DEFAULT 'pending',
    priority priority_level DEFAULT 'medium',
    created_at TIMESTAMP DEFAULT NOW()
);

-- Вставка и запросы
INSERT INTO orders (customer_id, status, priority) VALUES 
    (1, 'pending', 'high'),
    (2, 'processing', 'medium'),
    (3, 'shipped', 'low');

-- Сравнение ENUM значений (учитывает порядок)
SELECT * FROM orders WHERE status > 'pending';  -- processing, shipped, delivered
SELECT * FROM orders WHERE priority >= 'medium';  -- medium, high, critical

-- Изменение ENUM (добавление значений)
ALTER TYPE order_status ADD VALUE 'returned' AFTER 'delivered';
```

#### Создание доменов
```sql
-- Домены - типы с ограничениями
CREATE DOMAIN email_type AS VARCHAR(255)
    CHECK (VALUE ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,});

CREATE DOMAIN positive_price AS NUMERIC(10,2)
    CHECK (VALUE > 0);

CREATE DOMAIN phone_type AS VARCHAR(20)
    CHECK (VALUE ~ '^\+?[\d\s\-\(\)]+);

-- Использование доменов
CREATE TABLE validated_customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email email_type NOT NULL,
    phone phone_type,
    credit_limit positive_price
);

-- Попытка вставить невалидные данные вызовет ошибку
INSERT INTO validated_customers (name, email, phone, credit_limit) VALUES 
    ('Valid User', 'user@example.com', '+1-555-0123', 5000.00);

-- Это вызовет ошибку
-- INSERT INTO validated_customers (name, email, credit_limit) VALUES 
--     ('Invalid User', 'not-an-email', -100.00);
```

### Полнотекстовый поиск

#### Основы полнотекстового поиска
```sql
-- Создание таблицы для поиска
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255),
    content TEXT,
    search_vector tsvector
);

-- Создание индекса для поиска
CREATE INDEX idx_documents_search ON documents USING gin(search_vector);

-- Вставка данных с автоматическим созданием поискового вектора
INSERT INTO documents (title, content, search_vector) VALUES 
    ('PostgreSQL Tutorial', 'Learn PostgreSQL database management system', 
     to_tsvector('english', 'PostgreSQL Tutorial Learn PostgreSQL database management system')),
    ('Advanced SQL Queries', 'Complex JOIN operations and window functions in SQL',
     to_tsvector('english', 'Advanced SQL Queries Complex JOIN operations and window functions in SQL')),
    ('Database Design Principles', 'Normalization, indexing, and performance optimization',
     to_tsvector('english', 'Database Design Principles Normalization, indexing, and performance optimization'));

-- Автоматическое обновление поискового вектора
CREATE OR REPLACE FUNCTION update_search_vector() RETURNS TRIGGER AS $
BEGIN
    NEW.search_vector := to_tsvector('english', COALESCE(NEW.title, '') || ' ' || COALESCE(NEW.content, ''));
    RETURN NEW;
END;
$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_update_search_vector
    BEFORE INSERT OR UPDATE ON documents
    FOR EACH ROW EXECUTE FUNCTION update_search_vector();
```

#### Поисковые запросы
```sql
-- Простой поиск
SELECT title, content
FROM documents
WHERE search_vector @@ to_tsquery('english', 'PostgreSQL');

-- Поиск с несколькими словами (AND)
SELECT title, content
FROM documents
WHERE search_vector @@ to_tsquery('english', 'PostgreSQL & database');

-- Поиск с альтернативами (OR)
SELECT title, content
FROM documents
WHERE search_vector @@ to_tsquery('english', 'PostgreSQL | MySQL');

-- Исключение слов (NOT)
SELECT title, content
FROM documents
WHERE search_vector @@ to_tsquery('english', 'database & !PostgreSQL');

-- Поиск фраз
SELECT title, content
FROM documents
WHERE search_vector @@ phraseto_tsquery('english', 'database management system');

-- Нечеткий поиск
SELECT title, content
FROM documents
WHERE search_vector @@ websearch_to_tsquery('english', 'PostgreSQL tutorial');
```

#### Ранжирование результатов поиска
```sql
-- Поиск с ранжированием
SELECT 
    title,
    content,
    ts_rank(search_vector, to_tsquery('english', 'PostgreSQL & database')) as rank
FROM documents
WHERE search_vector @@ to_tsquery('english', 'PostgreSQL & database')
ORDER BY rank DESC;

-- Weighted ranking (разные веса для заголовка и содержимого)
SELECT 
    title,
    content,
    ts_rank_cd(
        setweight(to_tsvector('english', title), 'A') ||
        setweight(to_tsvector('english', content), 'B'),
        to_tsquery('english', 'PostgreSQL')
    ) as rank
FROM documents
WHERE to_tsvector('english', title || ' ' || content) @@ to_tsquery('english', 'PostgreSQL')
ORDER BY rank DESC;

-- Выделение найденных слов
SELECT 
    title,
    ts_headline(
        'english',
        content,
        to_tsquery('english', 'PostgreSQL & database'),
        'StartSel = <mark>, StopSel = </mark>'
    ) as highlighted_content
FROM documents
WHERE search_vector @@ to_tsquery('english', 'PostgreSQL & database');
```

---

## Глава 10: Функции, процедуры и триггеры

### Пользовательские функции

#### Создание простых функций
```sql
-- Простая функция на SQL
CREATE OR REPLACE FUNCTION calculate_age(birth_date DATE)
RETURNS INTEGER AS $
    SELECT EXTRACT(YEAR FROM AGE(birth_date))::INTEGER;
$ LANGUAGE sql;

-- Использование функции
SELECT name, calculate_age(birth_date) as age
FROM customers
WHERE birth_date IS NOT NULL;

-- Функция с несколькими параметрами
CREATE OR REPLACE FUNCTION format_name(first_name VARCHAR, last_name VARCHAR)
RETURNS VARCHAR AS $
    SELECT CASE 
        WHEN first_name IS NOT NULL AND last_name IS NOT NULL THEN
            first_name || ' ' || last_name
        WHEN first_name IS NOT NULL THEN
            first_name
        WHEN last_name IS NOT NULL THEN
            last_name
        ELSE
            'Unknown'
    END;
$ LANGUAGE sql;
```

#### Функции на PL/pgSQL
```sql
-- Функция с условной логикой
CREATE OR REPLACE FUNCTION get_customer_tier(customer_id INTEGER)
RETURNS VARCHAR AS $
DECLARE
    total_orders INTEGER;
    total_spent DECIMAL;
    tier VARCHAR;
BEGIN
    -- Получаем статистику клиента
    SELECT 
        COUNT(*),
        COALESCE(SUM(total), 0)
    INTO total_orders, total_spent
    FROM orders
    WHERE user_id = customer_id;
    
    -- Определяем tier
    IF total_spent > 10000 OR total_orders > 50 THEN
        tier := 'VIP';
    ELSIF total_spent > 5000 OR total_orders > 20 THEN
        tier := 'Gold';
    ELSIF total_spent > 1000 OR total_orders > 5 THEN
        tier := 'Silver';
    ELSE
        tier := 'Bronze';
    END IF;
    
    RETURN tier;
END;
$ LANGUAGE plpgsql;

-- Функция с циклом
CREATE OR REPLACE FUNCTION generate_fibonacci(n INTEGER)
RETURNS INTEGER[] AS $
DECLARE
    result INTEGER[] := ARRAY[0, 1];
    i INTEGER;
BEGIN
    IF n <= 0 THEN
        RETURN ARRAY[]::INTEGER[];
    ELSIF n = 1 THEN
        RETURN ARRAY[0];
    ELSIF n = 2 THEN
        RETURN result;
    END IF;
    
    FOR i IN 3..n LOOP
        result := array_append(result, result[i-1] + result[i-2]);
    END LOOP;
    
    RETURN result;
END;
$ LANGUAGE plpgsql;
```

#### Функции, возвращающие таблицы
```sql
-- Функция, возвращающая набор строк
CREATE OR REPLACE FUNCTION get_top_customers(limit_count INTEGER DEFAULT 10)
RETURNS TABLE(
    customer_id INTEGER,
    customer_name VARCHAR,
    total_orders BIGINT,
    total_spent DECIMAL,
    tier VARCHAR
) AS $
BEGIN
    RETURN QUERY
    SELECT 
        u.id,
        u.name,
        COUNT(o.id),
        COALESCE(SUM(o.total), 0),
        get_customer_tier(u.id)
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    GROUP BY u.id, u.name
    ORDER BY COALESCE(SUM(o.total), 0) DESC
    LIMIT limit_count;
END;
$ LANGUAGE plpgsql;

-- Использование табличной функции
SELECT * FROM get_top_customers(5);

-- Функция с динамическим SQL
CREATE OR REPLACE FUNCTION get_table_stats(table_name TEXT)
RETURNS TABLE(
    column_name TEXT,
    data_type TEXT,
    is_nullable TEXT,
    column_default TEXT
) AS $
BEGIN
    RETURN QUERY EXECUTE format('
        SELECT 
            column_name::TEXT,
            data_type::TEXT,
            is_nullable::TEXT,
            column_default::TEXT
        FROM information_schema.columns 
        WHERE table_name = %L
        ORDER BY ordinal_position',
        table_name
    );
END;
$ LANGUAGE plpgsql;
```

### Хранимые процедуры

#### Создание процедур
```sql
-- Процедура для обновления статистики клиентов
CREATE OR REPLACE PROCEDURE update_customer_statistics()
LANGUAGE plpgsql AS $
DECLARE
    customer_record RECORD;
    stats_updated INTEGER := 0;
BEGIN
    -- Создаем временную таблицу для статистики
    CREATE TEMP TABLE IF NOT EXISTS temp_customer_stats (
        customer_id INTEGER,
        total_orders INTEGER,
        total_spent DECIMAL,
        avg_order_value DECIMAL,
        last_order_date DATE
    );
    
    -- Очищаем временную таблицу
    TRUNCATE temp_customer_stats;
    
    -- Вычисляем статистику для каждого клиента
    FOR customer_record IN 
        SELECT DISTINCT user_id FROM orders
    LOOP
        INSERT INTO temp_customer_stats
        SELECT 
            customer_record.user_id,
            COUNT(*),
            SUM(total),
            AVG(total),
            MAX(created_at::DATE)
        FROM orders
        WHERE user_id = customer_record.user_id;
        
        stats_updated := stats_updated + 1;
    END LOOP;
    
    -- Выводим результат
    RAISE NOTICE 'Updated statistics for % customers', stats_updated;
    
    -- Можем обновить основную таблицу
    -- UPDATE customers SET ... FROM temp_customer_stats WHERE ...
END;
$;

-- Вызов процедуры
CALL update_customer_statistics();
```

#### Процедуры с параметрами
```sql
-- Процедура для создания отчета
CREATE OR REPLACE PROCEDURE generate_sales_report(
    start_date DATE,
    end_date DATE,
    OUT total_revenue DECIMAL,
    OUT total_orders INTEGER,
    OUT avg_order_value DECIMAL
)
LANGUAGE plpgsql AS $
BEGIN
    SELECT 
        COALESCE(SUM(total), 0),
        COUNT(*),
        COALESCE(AVG(total), 0)
    INTO total_revenue, total_orders, avg_order_value
    FROM orders
    WHERE created_at::DATE BETWEEN start_date AND end_date;
    
    RAISE NOTICE 'Report generated for period % to %', start_date, end_date;
    RAISE NOTICE 'Total Revenue: %, Orders: %, Avg Order: %', 
                 total_revenue, total_orders, avg_order_value;
END;
$;

-- Вызов процедуры с OUTPUT параметрами
DO $
DECLARE
    revenue DECIMAL;
    orders INTEGER;
    avg_value DECIMAL;
BEGIN
    CALL generate_sales_report('2024-01-01', '2024-01-31', revenue, orders, avg_value);
    RAISE NOTICE 'Results: Revenue=%, Orders=%, Average=%', revenue, orders, avg_value;
END;
$;
```

### Триггеры

#### Триггеры для аудита
```sql
-- Создаем таблицу аудита
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    table_name VARCHAR(50),
    operation VARCHAR(10),
    old_data JSONB,
    new_data JSONB,
    changed_by VARCHAR(100),
    changed_at TIMESTAMP DEFAULT NOW()
);

-- Функция для аудита
CREATE OR REPLACE FUNCTION audit_trigger_function()
RETURNS TRIGGER AS $
BEGIN
    IF TG_OP = 'DELETE' THEN
        INSERT INTO audit_log (table_name, operation, old_data, changed_by)
        VALUES (TG_TABLE_NAME, TG_OP, row_to_json(OLD), user);
        RETURN OLD;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_log (table_name, operation, old_data, new_data, changed_by)
        VALUES (TG_TABLE_NAME, TG_OP, row_to_json(OLD), row_to_json(NEW), user);
        RETURN NEW;
    ELSIF TG_OP = 'INSERT' THEN
        INSERT INTO audit_log (table_name, operation, new_data, changed_by)
        VALUES (TG_TABLE_NAME, TG_OP, row_to_json(NEW), user);
        RETURN NEW;
    END IF;
    RETURN NULL;
END;
$ LANGUAGE plpgsql;

-- Создаем триггеры для разных таблиц
CREATE TRIGGER users_audit_trigger
    AFTER INSERT OR UPDATE OR DELETE ON users
    FOR EACH ROW EXECUTE FUNCTION audit_trigger_function();

CREATE TRIGGER orders_audit_trigger
    AFTER INSERT OR UPDATE OR DELETE ON orders
    FOR EACH ROW EXECUTE FUNCTION audit_trigger_function();
```

#### Триггеры для валидации данных
```sql
-- Функция для валидации заказов
CREATE OR REPLACE FUNCTION validate_order()
RETURNS TRIGGER AS $
BEGIN
    -- Проверяем, что сумма заказа положительная
    IF NEW.total <= 0 THEN
        RAISE EXCEPTION 'Order total must be positive, got %', NEW.total;
    END IF;
    
    -- Проверяем, что пользователь существует и активен
    IF NOT EXISTS (
        SELECT 1 FROM users 
        WHERE id = NEW.user_id AND status = 'active'
    ) THEN
        RAISE EXCEPTION 'User % is not active or does not exist', NEW.user_id;
    END IF;
    
    -- Автоматически устанавливаем дату создания
    IF NEW.created_at IS NULL THEN
        NEW.created_at := NOW();
    END IF;
    
    RETURN NEW;
END;
$ LANGUAGE plpgsql;

-- Создаем триггер валидации
CREATE TRIGGER validate_order_trigger
    BEFORE INSERT OR UPDATE ON orders
    FOR EACH ROW EXECUTE FUNCTION validate_order();
```

#### Триггеры для автоматического обновления
```sql
-- Функция для обновления счетчиков
CREATE OR REPLACE FUNCTION update_user_stats()
RETURNS TRIGGER AS $
DECLARE
    user_id_to_update INTEGER;
BEGIN
    -- Определяем, какого пользователя обновлять
    IF TG_OP = 'DELETE' THEN
        user_id_to_update := OLD.user_id;
    ELSE
        user_id_to_update := NEW.user_id;
    END IF;
    
    -- Обновляем статистику пользователя
    UPDATE users 
    SET 
        total_orders = (
            SELECT COUNT(*) FROM orders WHERE user_id = user_id_to_update
        ),
        total_spent = (
            SELECT COALESCE(SUM(total), 0) FROM orders WHERE user_id = user_id_to_update
        ),
        updated_at = NOW()
    WHERE id = user_id_to_update;
    
    IF TG_OP = 'DELETE' THEN
        RETURN OLD;
    ELSE
        RETURN NEW;
    END IF;
END;
$ LANGUAGE plpgsql;

-- Создаем триггер для автоматического обновления статистики
CREATE TRIGGER update_user_stats_trigger
    AFTER INSERT OR UPDATE OR DELETE ON orders
    FOR EACH ROW EXECUTE FUNCTION update_user_stats();

-- Триггер для автоматического обновления поля updated_at
CREATE OR REPLACE FUNCTION update_modified_time()
RETURNS TRIGGER AS $
BEGIN
    NEW.updated_at := NOW();
    RETURN NEW;
END;
$ LANGUAGE plpgsql;

-- Применяем к разным таблицам
CREATE TRIGGER update_users_modified_time
    BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_modified_time();

CREATE TRIGGER update_products_modified_time
    BEFORE UPDATE ON products
    FOR EACH ROW EXECUTE FUNCTION update_modified_time();
```

#### Управление триггерами
```sql
-- Просмотр всех триггеров
SELECT 
    t.trigger_name,
    t.event_manipulation,
    t.event_object_table,
    t.action_timing,
    t.action_statement
FROM information_schema.triggers t
WHERE t.trigger_schema = 'public'
ORDER BY t.event_object_table, t.trigger_name;

-- Отключение триггера
ALTER TABLE orders DISABLE TRIGGER validate_order_trigger;

-- Включение триггера
ALTER TABLE orders ENABLE TRIGGER validate_order_trigger;

-- Отключение всех триггеров на таблице
ALTER TABLE orders DISABLE TRIGGER ALL;

-- Удаление триггера
DROP TRIGGER IF EXISTS validate_order_trigger ON orders;

-- Удаление функции триггера
DROP FUNCTION IF EXISTS validate_order();
```

### Обработка ошибок и исключений

#### Базовая обработка ошибок
```sql
CREATE OR REPLACE FUNCTION safe_divide(a NUMERIC, b NUMERIC)
RETURNS NUMERIC AS $
BEGIN
    IF b = 0 THEN
        RAISE EXCEPTION 'Division by zero is not allowed';
    END IF;
    
    RETURN a / b;
EXCEPTION
    WHEN division_by_zero THEN
        RAISE NOTICE 'Attempted division by zero, returning NULL';
        RETURN NULL;
    WHEN OTHERS THEN
        RAISE NOTICE 'Unexpected error: %', SQLERRM;
        RETURN NULL;
END;
$ LANGUAGE plpgsql;
```

#### Продвинутая обработка ошибок
```sql
CREATE OR REPLACE FUNCTION process_order_safely(
    p_user_id INTEGER,
    p_total DECIMAL,
    p_items JSONB
)
RETURNS INTEGER AS $
DECLARE
    new_order_id INTEGER;
    item JSONB;
    error_msg TEXT;
BEGIN
    -- Начинаем транзакцию внутри функции
    BEGIN
        -- Создаем заказ
        INSERT INTO orders (user_id, total, status)
        VALUES (p_user_id, p_total, 'pending')
        RETURNING id INTO new_order_id;
        
        -- Добавляем товары в заказ
        FOR item IN SELECT * FROM jsonb_array_elements(p_items)
        LOOP
            INSERT INTO order_items (order_id, product_id, quantity, price)
            VALUES (
                new_order_id,
                (item->>'product_id')::INTEGER,
                (item->>'quantity')::INTEGER,
                (item->>'price')::DECIMAL
            );
        END LOOP;
        
        -- Проверяем целостность заказа
        IF NOT EXISTS (SELECT 1 FROM order_items WHERE order_id = new_order_id) THEN
            RAISE EXCEPTION 'Order must contain at least one item';
        END IF;
        
        RETURN new_order_id;
        
    EXCEPTION
        WHEN foreign_key_violation THEN
            error_msg := 'Invalid user_id or product_id: ' || SQLERRM;
            RAISE NOTICE '%', error_msg;
            RETURN -1;
        WHEN check_violation THEN
            error_msg := 'Data validation failed: ' || SQLERRM;
            RAISE NOTICE '%', error_msg;
            RETURN -2;
        WHEN OTHERS THEN
            error_msg := 'Unexpected error processing order: ' || SQLERRM;
            RAISE NOTICE '%', error_msg;
            RETURN -3;
    END;
END;
$ LANGUAGE plpgsql;
```

---

Это завершает структурированное руководство по PostgreSQL. Материал теперь организован в 3 блока и 10 глав с непрерывной нумерацией, охватывая путь от основ SQL до продвинутых возможностей PostgreSQL для backend разработчиков.