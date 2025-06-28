# Блок собеседования по PostgreSQL для Backend разработчиков

## 📋 Структура собеседования

### Уровни сложности:
- **Junior** (1-2 года опыта) - 🟢
- **Middle** (2-4 года опыта) - 🟡  
- **Senior** (4+ лет опыта) - 🔴

---

## 1. Основы PostgreSQL и СУБД

### 🟢 Junior уровень

**Вопрос:** Что такое PostgreSQL и в чем его основные отличия от других реляционных СУБД?

**Подсказки к ответу:**
- PostgreSQL - это открытая реляционная СУБД, известная своей масштабируемостью, надежностью и расширяемостью
- Поддержка ACID-транзакций
- Многоверсионная конкурентность (MVCC)
- Расширяемость через пользовательские типы данных и функции
- Поддержка как SQL, так и JSON-запросов

**Пример:**
```sql
-- PostgreSQL поддерживает расширенные типы данных
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    metadata JSONB, -- JSON с бинарным хранением
    tags TEXT[], -- Массивы
    price DECIMAL(10,2)
);
```

---

**Вопрос:** Что такое схемы (schemas) в PostgreSQL и зачем они нужны?

**Подсказки к ответу:**
- Схемы - это пространства имен для организации объектов базы данных в логические группы
- Рекомендуется создавать единую базу данных с множественными именованными схемами вместо множественных баз данных
- Позволяют многим пользователям использовать одну базу данных без конфликтов
- Упрощение управления доступом через GRANT

**Пример:**
```sql
-- Создание схем для разделения логики
CREATE SCHEMA auth;
CREATE SCHEMA billing;
CREATE SCHEMA analytics;

-- Создание таблиц в разных схемах
CREATE TABLE auth.users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL
);

CREATE TABLE billing.invoices (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES auth.users(id)
);
```

### 🟡 Middle уровень

**Вопрос:** Объясните концепцию MVCC в PostgreSQL и как она влияет на производительность.

**Подсказки к ответу:**
- Multi-Version Concurrency Control - механизм управления конкурентным доступом
- Каждая транзакция видит снимок данных на момент её начала
- Читатели не блокируют писателей и наоборот
- Необходимость в VACUUM для очистки старых версий строк
- VACUUM удаляет мертвые кортежи и освобождает место для хранения

**Пример проблемы:**
```sql
-- Проблема накопления мертвых кортежей
SELECT schemaname, tablename, n_dead_tup, n_live_tup,
       round(n_dead_tup::numeric / (n_live_tup + n_dead_tup) * 100, 2) as dead_ratio
FROM pg_stat_user_tables 
WHERE n_dead_tup > 0
ORDER BY dead_ratio DESC;
```

---

**Вопрос:** Какие новые возможности появились в PostgreSQL 16 и 17, актуальные для 2025 года?

**Подсказки к ответу:**
- PostgreSQL 16: улучшенная параллелизация запросов (FULL и RIGHT joins), оптимизация агрегатных функций
- PostgreSQL 17: улучшения B-tree индексов для IN-запросов, оптимизация VACUUM, новая система управления памятью
- Поддержка SQL/JSON стандарта с JSON_TABLE функцией
- Улучшенная логическая репликация с failover slots

### 🔴 Senior уровень

**Вопрос:** Как устроена логическая репликация в PostgreSQL 17 и какие улучшения были внесены?

**Подсказки к ответу:**
- Новые failover slots для бесшовного продолжения репликации после сбоя
- Сохранение слотов репликации и зависимостей подписок при обновлении версий
- Возможность использования hash-индексов на подписчике
- Параллельное применение больших транзакций

---

## 2. Проектирование схемы базы данных

### 🟢 Junior уровень

**Вопрос:** Объясните принципы нормализации. Приведите пример перехода от 1NF к 3NF.

**Подсказки к ответу:**
- 1NF: каждая колонка должна содержать только одно значение
- 2NF: каждая не-ключевая колонка должна зависеть от всего составного ключа
- 3NF: каждый элемент данных хранится только один раз

**Пример денормализованной таблицы (нарушение 1NF):**
```sql
-- НЕПРАВИЛЬНО - нарушение 1NF
CREATE TABLE movies_bad (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    genres VARCHAR(500), -- "Action, Drama, Thriller"
    actors VARCHAR(1000) -- "Actor1, Actor2, Actor3"
);
```

**Пример нормализованной структуры (3NF):**
```sql
-- ПРАВИЛЬНО - соответствует 3NF
CREATE TABLE movies (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    release_year INTEGER
);

CREATE TABLE genres (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE movie_genres (
    movie_id INTEGER REFERENCES movies(id),
    genre_id INTEGER REFERENCES genres(id),
    PRIMARY KEY (movie_id, genre_id)
);
```

---

**Вопрос:** Когда следует использовать денормализацию и какие есть альтернативы?

**Подсказки к ответу:**
- Для случаев, когда множественные JOIN дороги и нужны быстрые операции чтения
- Денормализация может применяться для оптимизации read-heavy операций
- Альтернативы: материализованные представления, кэширование, индексы

### 🟡 Middle уровень

**Вопрос:** Как правильно выбирать типы данных в PostgreSQL? Приведите примеры оптимального выбора.

**Подсказки к ответу:**
- Использование подходящих типов данных обеспечивает эффективное хранение и обработку
- Выбор правильных типов данных влияет на место хранения, производительность и целостность данных

**Примеры оптимального выбора:**
```sql
CREATE TABLE users (
    -- UUID для распределенных систем
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- VARCHAR с ограничением вместо TEXT где возможно
    email VARCHAR(255) NOT NULL,
    
    -- DECIMAL для денежных значений (не FLOAT!)
    balance DECIMAL(15,2) DEFAULT 0.00,
    
    -- JSONB для полуструктурированных данных
    metadata JSONB,
    
    -- TIMESTAMP WITH TIME ZONE для временных меток
    created_at TIMESTAMPTZ DEFAULT NOW(),
    
    -- Массивы для списков значений
    tags TEXT[],
    
    -- Правильный размер для статуса
    status VARCHAR(20) CHECK (status IN ('active', 'inactive', 'suspended'))
);
```

---

**Вопрос:** Объясните стратегии именования в PostgreSQL. Почему это важно?

**Подсказки к ответу:**
- Использовать строчные буквы для имен объектов, так как PostgreSQL автоматически конвертирует неэкранированные идентификаторы
- Разделять слова подчеркиваниями (_) вместо camelCase
- Для внешних ключей использовать формат referenced_table_singular_form_id

**Примеры правильного именования:**
```sql
-- Таблицы - множественное число, snake_case
CREATE TABLE user_profiles (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Индексы с префиксом
CREATE INDEX idx_user_profiles_user_id ON user_profiles (user_id);
CREATE INDEX idx_user_profiles_created_at ON user_profiles (created_at);

-- Внешние ключи
ALTER TABLE user_profiles 
ADD CONSTRAINT fk_user_profiles_user_id 
FOREIGN KEY (user_id) REFERENCES users(id);
```

### 🔴 Senior уровень

**Вопрос:** Как спроектировать схему для системы с высокой нагрузкой? Какие техники масштабирования следует применить?

**Подсказки к ответу:**
- Партицирование таблиц (по времени, по хешу, по диапазону)
- Выбор между узкими и широкими таблицами
- Использование материализованных представлений
- Стратегическая денормализация
- Разделение на read/write реплики

**Пример партицирования:**
```sql
-- Партицирование по времени для логов
CREATE TABLE user_activities (
    id BIGSERIAL,
    user_id BIGINT NOT NULL,
    activity_type VARCHAR(50),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    data JSONB
) PARTITION BY RANGE (created_at);

-- Создание партиций по месяцам
CREATE TABLE user_activities_2025_01 
PARTITION OF user_activities 
FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');

CREATE TABLE user_activities_2025_02 
PARTITION OF user_activities 
FOR VALUES FROM ('2025-02-01') TO ('2025-03-01');

-- Автоматическое создание партиций через pg_partman
```

---

## 3. Индексы и оптимизация производительности

### 🟢 Junior уровень

**Вопрос:** Какие типы индексов существуют в PostgreSQL и когда их использовать?

**Подсказки к ответу:**
- B-tree индексы - тип по умолчанию, эффективны для равенства и диапазонных запросов
- Hash индексы - полезны для операций равенства, но не подходят для диапазонных запросов
- GIN и GiST индексы - специализированные для полнотекстового поиска и геометрических типов данных

**Примеры использования:**
```sql
-- B-tree (по умолчанию) для обычных запросов
CREATE INDEX idx_users_email ON users (email);

-- Составной индекс для сложных запросов
CREATE INDEX idx_orders_user_status ON orders (user_id, status);

-- Частичный индекс для конкретных условий
CREATE INDEX idx_active_users ON users (email) WHERE status = 'active';

-- GIN индекс для JSONB
CREATE INDEX idx_users_metadata ON users USING gin (metadata);

-- Полнотекстовый поиск
CREATE INDEX idx_products_search ON products USING gin (to_tsvector('english', name || ' ' || description));
```

---

**Вопрос:** Как анализировать производительность запросов? Что показывает EXPLAIN?

**Подсказки к ответу:**
- EXPLAIN показывает время, потраченное на локальное чтение и запись блоков I/O
- Новые опции в PostgreSQL 17: SERIALIZE и MEMORY для анализа времени конвертации данных и использования памяти
- Анализ cost, rows, actual time
- Выявление узких мест через Seq Scan, Nested Loop

**Пример анализа:**
```sql
-- Базовый анализ плана выполнения
EXPLAIN (ANALYZE, BUFFERS) 
SELECT u.email, p.name 
FROM users u 
JOIN profiles p ON u.id = p.user_id 
WHERE u.status = 'active';

-- Анализ с детализацией по памяти (PostgreSQL 17)
EXPLAIN (ANALYZE, BUFFERS, MEMORY) 
SELECT * FROM large_table 
WHERE complex_condition = 'value';
```

### 🟡 Middle уровень

**Вопрос:** Что такое индекс покрытия (covering index) и как его использовать в PostgreSQL?

**Подсказки к ответу:**
- Индекс, содержащий все необходимые для запроса колонки
- Избегает обращения к основной таблице (Index-Only Scan)
- Использование INCLUDE для дополнительных колонок

**Пример:**
```sql
-- Обычный индекс - требует обращения к таблице
CREATE INDEX idx_orders_user_id ON orders (user_id);

-- Покрывающий индекс - содержит все нужные данные
CREATE INDEX idx_orders_covering 
ON orders (user_id) 
INCLUDE (total_amount, created_at, status);

-- Этот запрос будет использовать Index-Only Scan
SELECT total_amount, created_at, status 
FROM orders 
WHERE user_id = 123;
```

---

**Вопрос:** Какие улучшения производительности появились в PostgreSQL 17 для индексов?

**Подсказки к ответу:**
- Значительные улучшения B-tree индексов для запросов с большими IN-списками или ANY условиями
- Сокращение количества сканирований индекса, уменьшение нагрузки на CPU и буферы
- BRIN индексы теперь поддерживают параллельное построение

### 🔴 Senior уровень

**Вопрос:** Как оптимизировать производительность для high-concurrency систем в PostgreSQL 17?

**Подсказки к ответу:**
- Использование PgBouncer для поддержания оптимального уровня соединений
- Мониторинг конкуренции блокировок
- Улучшения производительности при высокой конкуренции WAL-записей
- Настройка connection pooling через PgBouncer или Pgpool-II

**Пример конфигурации пула соединений:**
```python
# PgBouncer configuration для высокой нагрузки
[databases]
myapp = host=localhost port=5432 dbname=myapp

[pgbouncer]
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 25
max_db_connections = 100
```

---

## 4. Конфигурация и настройка производительности

### 🟢 Junior уровень

**Вопрос:** Какие основные параметры конфигурации PostgreSQL влияют на производительность?

**Подсказки к ответу:**
- shared_buffers: 25-40% от общей RAM для выделенных серверов БД
- work_mem: обычно 4-16MB на соединение
- maintenance_work_mem: 256MB-1GB для операций обслуживания

**Пример базовой настройки:**
```sql
-- postgresql.conf основные параметры
shared_buffers = '2GB'              -- 25% от 8GB RAM
work_mem = '8MB'                    -- для сортировки и хеш-операций
maintenance_work_mem = '512MB'      -- для VACUUM, CREATE INDEX
effective_cache_size = '6GB'       -- оценка кэша ОС
```

### 🟡 Middle уровень

**Вопрос:** Как настроить autovacuum для оптимальной производительности?

**Подсказки к ответу:**
- Обеспечение регулярного autovacuum для предотвращения раздувания таблиц
- Увеличен default vacuum_buffer_usage_limit до 2MB в PostgreSQL 17
- Настройка параметров под нагрузку системы

**Пример настройки:**
```sql
-- Настройка autovacuum
ALTER SYSTEM SET autovacuum_vacuum_scale_factor = 0.1;
ALTER SYSTEM SET autovacuum_analyze_scale_factor = 0.05;
ALTER SYSTEM SET autovacuum_vacuum_cost_limit = 2000;
ALTER SYSTEM SET autovacuum_max_workers = 6;

-- Для конкретной таблицы с высокой нагрузкой
ALTER TABLE high_activity_table SET (
    autovacuum_vacuum_scale_factor = 0.05,
    autovacuum_analyze_scale_factor = 0.02
);
```

### 🔴 Senior уровень

**Вопрос:** Как мониторить производительность PostgreSQL и какие метрики наиболее важны в 2025 году?

**Подсказки к ответу:**
- pg_stat_io - новый источник ключевых I/O метрик для детального анализа
- pg_wait_events система представлений для анализа ожиданий активных сессий
- pg_stat_checkpointer - новое системное представление для статистики checkpoint'ов

**Пример мониторинга:**
```sql
-- Анализ I/O статистики (PostgreSQL 16+)
SELECT backend_type, io_context, io_object, 
       reads, writes, extends, hits
FROM pg_stat_io 
ORDER BY reads + writes DESC;

-- Мониторинг ожиданий (PostgreSQL 17+)
SELECT wait_event_type, wait_event, 
       COUNT(*) as waiting_sessions
FROM pg_stat_activity a
JOIN pg_wait_events w ON a.wait_event = w.name
WHERE state = 'active'
GROUP BY wait_event_type, wait_event;

-- Анализ эффективности кэша
SELECT schemaname, tablename, 
       heap_blks_read, heap_blks_hit,
       round(heap_blks_hit::numeric / (heap_blks_hit + heap_blks_read) * 100, 2) as cache_hit_ratio
FROM pg_statio_user_tables 
WHERE heap_blks_read > 0
ORDER BY cache_hit_ratio;
```

---

## 5. Безопасность и управление доступом

### 🟢 Junior уровень

**Вопрос:** Как правильно настроить аутентификацию и авторизацию в PostgreSQL?

**Подсказки к ответу:**
- Использование сильных паролей и ролевого контроля доступа
- Использование переменных окружения для строк подключения
- Настройка pg_hba.conf для контроля подключений

**Пример настройки ролей:**
```sql
-- Создание ролей для разделения доступа
CREATE ROLE app_read;
CREATE ROLE app_write;
CREATE ROLE app_admin;

-- Предоставление прав на схему
GRANT USAGE ON SCHEMA app TO app_read, app_write;
GRANT SELECT ON ALL TABLES IN SCHEMA app TO app_read;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA app TO app_write;

-- Создание пользователя приложения
CREATE USER myapp_user WITH PASSWORD 'strong_password_here';
GRANT app_write TO myapp_user;
```

### 🟡 Middle уровень

**Вопрос:** Что такое Row Level Security (RLS) и как его применять?

**Подсказки к ответу:**
- Контроль доступа на уровне строк таблицы
- Политики безопасности для разных ролей
- Применение для мультитенантных приложений

**Пример использования:**
```sql
-- Включение RLS для таблицы
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- Политика: пользователи видят только свои документы
CREATE POLICY user_documents ON documents
    FOR ALL TO app_users
    USING (user_id = current_setting('app.current_user_id')::integer);

-- Политика для администраторов: видят всё
CREATE POLICY admin_all_documents ON documents
    FOR ALL TO app_admins
    USING (true);
```

### 🔴 Senior уровень

**Вопрос:** Как обеспечить безопасность в production среде PostgreSQL?

**Подсказки к ответу:**
- SSL/TLS шифрование соединений
- Новая опция sslnegotiation=direct для прямого TLS handshake
- Аудит изменений данных
- Создание триггеров аудита для отслеживания изменений
- Регулярные обновления безопасности

**Пример системы аудита:**
```sql
-- Создание таблицы аудита
CREATE TABLE audit_log (
    id BIGSERIAL PRIMARY KEY,
    table_name TEXT NOT NULL,
    operation TEXT NOT NULL,
    user_name TEXT NOT NULL DEFAULT current_user,
    changed_at TIMESTAMPTZ DEFAULT NOW(),
    old_values JSONB,
    new_values JSONB
);

-- Функция аудита
CREATE OR REPLACE FUNCTION audit_trigger_function()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_log (table_name, operation, old_values, new_values)
    VALUES (
        TG_TABLE_NAME,
        TG_OP,
        CASE WHEN TG_OP = 'DELETE' THEN row_to_json(OLD) ELSE NULL END,
        CASE WHEN TG_OP = 'INSERT' THEN row_to_json(NEW) 
             WHEN TG_OP = 'UPDATE' THEN row_to_json(NEW) 
             ELSE NULL END
    );
    RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql;
```

---

## 6. Резервное копирование и восстановление

### 🟢 Junior уровень

**Вопрос:** Какие типы резервного копирования существуют в PostgreSQL?

**Подсказки к ответу:**
- Логические и физические бэкапы, каждый предоставляет различные варианты восстановления
- pg_dump для логических бэкапов
- pg_basebackup для физических бэкапов
- pg_basebackup теперь поддерживает инкрементальные бэкапы

**Примеры команд:**
```bash
# Логический бэкап базы данных
pg_dump -h localhost -U postgres -d myapp > myapp_backup.sql

# Физический бэкап кластера
pg_basebackup -h localhost -D /backup/base -U postgres -P -W

# Инкрементальный бэкап (PostgreSQL 17+)
pg_basebackup -h localhost -D /backup/incremental --incremental=/backup/base -P
```

### 🟡 Middle уровень

**Вопрос:** Как настроить Point-in-Time Recovery (PITR)?

**Подсказки к ответу:**
- Настройка WAL архивирования
- Использование pg_basebackup для базовой копии
- Восстановление до определенного времени

**Пример настройки:**
```sql
-- postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'cp %p /backup/wal/%f'
max_wal_senders = 3

-- Восстановление до определенного времени
-- recovery.conf (или postgresql.conf в новых версиях)
restore_command = 'cp /backup/wal/%f %p'
recovery_target_time = '2025-01-15 14:30:00'
```

### 🔴 Senior уровень

**Вопрос:** Как спроектировать стратегию бэкапов для enterprise системы?

**Подсказки к ответу:**
- Регулярные полные и инкрементальные бэкапы
- Проверка целостности бэкапов и практика процедур восстановления
- Использование Barman или pgBackRest
- Географически распределенные бэкапы

---

## 7. Мониторинг и диагностика

### 🟡 Middle уровень

**Вопрос:** Какие системные представления PostgreSQL наиболее важны для мониторинга?

**Подсказки к ответу:**
- pg_stat_activity - активные сессии
- pg_stat_database - статистика по базам данных
- pg_stat_io - новые I/O метрики в PostgreSQL 16+
- pg_locks - информация о блокировках

**Примеры запросов мониторинга:**
```sql
-- Долго выполняющиеся запросы
SELECT pid, usename, application_name, state, 
       now() - query_start as duration, query
FROM pg_stat_activity 
WHERE state != 'idle' 
  AND now() - query_start > interval '5 minutes'
ORDER BY duration DESC;

-- Анализ блокировок
SELECT blocked_locks.pid AS blocked_pid,
       blocking_locks.pid AS blocking_pid,
       blocked_activity.usename AS blocked_user,
       blocking_activity.usename AS blocking_user,
       blocked_activity.query AS blocked_statement
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks ON blocking_locks.locktype = blocked_locks.locktype
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted AND blocking_locks.granted;

-- Размеры таблиц и индексов
SELECT schemaname, tablename,
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size,
       pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) as table_size,
       pg_size_pretty(pg_indexes_size(schemaname||'.'||tablename)) as indexes_size
FROM pg_tables 
WHERE schemaname NOT IN ('information_schema', 'pg_catalog')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

### 🔴 Senior уровень

**Вопрос:** Как выявить и решить проблемы производительности в production системе?

**Подсказки к ответу:**
- Анализ wait events через pg_wait_events (PostgreSQL 17)
- Мониторинг checkpoint активности через pg_stat_checkpointer
- Использование pg_stat_statements для анализа самых затратных запросов
- Выявление проблем с autovacuum

**Пример комплексной диагностики:**
```sql
-- Топ самых медленных запросов
SELECT query, calls, total_exec_time, mean_exec_time, 
       rows, 100.0 * shared_blks_hit / nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
FROM pg_stat_statements 
ORDER BY mean_exec_time DESC 
LIMIT 10;

-- Анализ checkpoint'ов (PostgreSQL 17)
SELECT checkpoints_timed, checkpoints_req, 
       checkpoint_write_time, checkpoint_sync_time,
       buffers_checkpoint, buffers_clean, buffers_backend
FROM pg_stat_checkpointer;

-- Проблемы с autovacuum
SELECT schemaname, tablename, n_dead_tup, n_live_tup,
       last_vacuum, last_autovacuum, last_analyze, last_autoanalyze
FROM pg_stat_user_tables 
WHERE n_dead_tup > 10000 
   OR last_autovacuum < now() - interval '1 day'
ORDER BY n_dead_tup DESC;
```

---

## 8. Репликация и высокая доступность

### 🟡 Middle уровень

**Вопрос:** Какие типы репликации поддерживает PostgreSQL?

**Подсказки к ответу:**
- Streaming репликация (физическая)
- Логическая репликация для селективной репликации данных
- Синхронная и асинхронная репликация
- Новые failover slots в PostgreSQL 17

**Пример настройки streaming репликации:**
```sql
-- На primary сервере postgresql.conf
wal_level = replica
max_wal_senders = 3
wal_keep_size = '1GB'
synchronous_standby_names = 'standby1'

-- pg_hba.conf на primary
host replication replicator 192.168.1.2/32 md5

-- На standby сервере
standby_mode = 'on'
primary_conninfo = 'host=192.168.1.1 port=5432 user=replicator password=secret'
```

### 🔴 Senior уровень

**Вопрос:** Что нового в логической репликации PostgreSQL 17 и как это влияет на архитектуру HA?

**Подсказки к ответу:**
- pg_upgrade теперь сохраняет слоты логической репликации и состояние подписок
- Включение failover логических слотов через sync_replication_slots
- Поддержка двунаправленной логической репликации
- Оптимизация производительности при множественных подтранзакциях

**Пример настройки failover slots:**
```sql
-- Создание logical replication slot с failover
SELECT pg_create_logical_replication_slot(
    'my_failover_slot', 
    'pgoutput', 
    false, -- temporary
    false, -- two_phase  
    true   -- failover
);

-- Настройка синхронизации failover slots
ALTER SYSTEM SET sync_replication_slots = on;
SELECT pg_reload_conf();

-- Создание подписки с failover поддержкой
CREATE SUBSCRIPTION my_subscription
CONNECTION 'host=primary.example.com dbname=mydb user=replicator'
PUBLICATION my_publication
WITH (
    enabled = true,
    create_slot = true,
    slot_name = 'my_failover_slot',
    failover = true
);
```

---

## 9. Работа с JSON и современные типы данных

### 🟡 Middle уровень

**Вопрос:** В чем разница между JSON и JSONB? Какие новые JSON возможности появились в PostgreSQL 17?

**Подсказки к ответу:**
- JSONB - бинарное представление JSON, поддерживает индексирование
- PostgreSQL 17 добавил JSON_TABLE для конвертации JSON в табличное представление
- Новые SQL/JSON конструкторы и функции запросов

**Примеры использования:**
```sql
-- Создание таблицы с JSONB
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    attributes JSONB
);

-- Индекс для JSONB
CREATE INDEX idx_products_attributes ON products USING gin (attributes);

-- JSON_TABLE в PostgreSQL 17
SELECT jt.*
FROM products p,
     JSON_TABLE(
         p.attributes,
         ' COLUMNS (
             brand VARCHAR(50) PATH '$.brand',
             price DECIMAL(10,2) PATH '$.price',
             tags TEXT[] PATH '$.tags'
         )
     ) AS jt
WHERE p.id = 1;

-- Новые SQL/JSON функции
SELECT JSON_EXISTS(attributes, '$.price' RETURNING BOOLEAN),
       JSON_VALUE(attributes, '$.brand' RETURNING VARCHAR(50)),
       JSON_QUERY(attributes, '$.features' RETURNING JSONB)
FROM products;
```

### 🔴 Senior уровень

**Вопрос:** Как оптимизировать работу с JSONB в высоконагруженных системах?

**Подсказки к ответу:**
- Использование частичных индексов для JSONB
- Выбор между GIN и GiST индексами
- Оптимизация запросов с JSON операторами

**Пример оптимизации:**
```sql
-- Частичный GIN индекс для конкретных ключей
CREATE INDEX idx_products_brand 
ON products USING gin ((attributes->'brand')) 
WHERE attributes ? 'brand';

-- Индекс для диапазонных запросов по числовым значениям в JSON
CREATE INDEX idx_products_price 
ON products ((attributes->>'price')::decimal) 
WHERE attributes ? 'price';

-- Оптимизированный запрос
SELECT * FROM products 
WHERE attributes @> '{"category": "electronics"}'
  AND (attributes->>'price')::decimal BETWEEN 100 AND 500;
```

---

## 10. Практические задачи и кейсы

### 🟡 Middle уровень

**Практическая задача:** Спроектируйте схему для блог-платформы с требованиями:
- Пользователи могут создавать посты
- Посты могут иметь теги и категории
- Система комментариев с возможностью ответов
- Лайки для постов и комментариев

**Подсказки к решению:**
```sql
-- Базовая схема
CREATE SCHEMA blog;

CREATE TABLE blog.users (
    id BIGSERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE blog.categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL
);

CREATE TABLE blog.posts (
    id BIGSERIAL PRIMARY KEY,
    title VARCHAR(500) NOT NULL,
    content TEXT NOT NULL,
    author_id BIGINT NOT NULL REFERENCES blog.users(id),
    category_id INTEGER REFERENCES blog.categories(id),
    published_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE blog.tags (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE blog.post_tags (
    post_id BIGINT REFERENCES blog.posts(id),
    tag_id INTEGER REFERENCES blog.tags(id),
    PRIMARY KEY (post_id, tag_id)
);

CREATE TABLE blog.comments (
    id BIGSERIAL PRIMARY KEY,
    post_id BIGINT NOT NULL REFERENCES blog.posts(id),
    author_id BIGINT NOT NULL REFERENCES blog.users(id),
    parent_id BIGINT REFERENCES blog.comments(id), -- для ответов
    content TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE blog.likes (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES blog.users(id),
    target_type VARCHAR(20) NOT NULL CHECK (target_type IN ('post', 'comment')),
    target_id BIGINT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE (user_id, target_type, target_id)
);

-- Индексы для производительности
CREATE INDEX idx_posts_author_published ON blog.posts (author_id, published_at) WHERE published_at IS NOT NULL;
CREATE INDEX idx_comments_post_id ON blog.comments (post_id);
CREATE INDEX idx_comments_parent_id ON blog.comments (parent_id) WHERE parent_id IS NOT NULL;
CREATE INDEX idx_likes_target ON blog.likes (target_type, target_id);
```

### 🔴 Senior уровень

**Практическая задача:** Оптимизируйте медленный запрос для аналитической системы:

```sql
-- Медленный запрос
SELECT u.username, 
       COUNT(DISTINCT p.id) as posts_count,
       COUNT(DISTINCT c.id) as comments_count,
       AVG(l.like_count) as avg_likes_per_post
FROM users u
LEFT JOIN posts p ON u.id = p.author_id AND p.published_at > '2024-01-01'
LEFT JOIN comments c ON p.id = c.post_id
LEFT JOIN (
    SELECT target_id, COUNT(*) as like_count 
    FROM likes 
    WHERE target_type = 'post' 
    GROUP BY target_id
) l ON p.id = l.target_id
WHERE u.created_at > '2023-01-01'
GROUP BY u.id, u.username
HAVING COUNT(DISTINCT p.id) > 0
ORDER BY posts_count DESC;
```

**Решение с оптимизацией:**
```sql
-- Создание материализованного представления для статистики
CREATE MATERIALIZED VIEW user_statistics AS
SELECT 
    u.id as user_id,
    u.username,
    COALESCE(ps.posts_count, 0) as posts_count,
    COALESCE(cs.comments_count, 0) as comments_count,
    COALESCE(ls.avg_likes_per_post, 0) as avg_likes_per_post,
    NOW() as calculated_at
FROM users u
LEFT JOIN (
    SELECT author_id, COUNT(*) as posts_count
    FROM posts 
    WHERE published_at > '2024-01-01'
    GROUP BY author_id
) ps ON u.id = ps.author_id
LEFT JOIN (
    SELECT p.author_id, COUNT(c.id) as comments_count
    FROM posts p
    JOIN comments c ON p.id = c.post_id
    WHERE p.published_at > '2024-01-01'
    GROUP BY p.author_id
) cs ON u.id = cs.author_id
LEFT JOIN (
    SELECT p.author_id, AVG(l.like_count) as avg_likes_per_post
    FROM posts p
    JOIN (
        SELECT target_id, COUNT(*) as like_count
        FROM likes 
        WHERE target_type = 'post'
        GROUP BY target_id
    ) l ON p.id = l.target_id
    WHERE p.published_at > '2024-01-01'
    GROUP BY p.author_id
) ls ON u.id = ls.author_id
WHERE u.created_at > '2023-01-01';

-- Индекс для материализованного представления
CREATE INDEX idx_user_statistics_posts_count ON user_statistics (posts_count DESC);

-- Автоматическое обновление через триггер или cron
CREATE OR REPLACE FUNCTION refresh_user_statistics()
RETURNS void AS $
BEGIN
    REFRESH MATERIALIZED VIEW CONCURRENTLY user_statistics;
END;
$ LANGUAGE plpgsql;
```

---

## 📝 Дополнительные вопросы для углубленной проверки

### Вопросы на знание специфики PostgreSQL 2025

1. **Новые возможности PostgreSQL 17:**
   - Объясните улучшения в системе управления памятью для VACUUM
   - Как работают новые failover slots в логической репликации?
   - Что изменилось в производительности B-tree индексов?

2. **Оптимизация для современных нагрузок:**
   - Как настроить PostgreSQL для микросервисной архитектуры?
   - Стратегии масштабирования read-heavy приложений
   - Интеграция с облачными платформами

3. **Мониторинг и диагностика:**
   - Использование pg_stat_io для анализа производительности
   - Настройка мониторинга для containerized окружений
   - Автоматизация задач обслуживания

### Критерии оценки ответов

**Excellent (90-100%):**
- Полное понимание концепций
- Знание современных возможностей PostgreSQL 17
- Способность предложить оптимальные решения
- Понимание trade-offs различных подходов

**Good (70-89%):**
- Хорошее понимание основных концепций
- Знание базовых принципов оптимизации
- Способность решать практические задачи

**Satisfactory (50-69%):**
- Базовое понимание PostgreSQL
- Знание основных команд и конфигурации
- Требуется дополнительное обучение для senior позиций

**Needs Improvement (<50%):**
- Недостаточное понимание ключевых концепций
- Неспособность решать практические задачи
- Требуется значительное обучение

---

## 🎯 Рекомендации для подготовки

### Для Junior разработчиков:
- Изучите официальную документацию PostgreSQL
- Практикуйтесь в проектировании простых схем
- Освойте базовые команды EXPLAIN и мониторинга

### Для Middle разработчиков:
- Углубите знания в области оптимизации запросов
- Изучите особенности различных типов индексов
- Практикуйтесь в настройке производительности

### Для Senior разработчиков:
- Следите за новыми возможностями PostgreSQL 17
- Изучите лучшие практики для high-load систем
- Освойте продвинутые техники мониторинга и диагностики

---

*Этот блок собеседования актуален для 2025 года и учитывает последние возможности PostgreSQL 16-17, современные best practices и требования к backend разработчикам.*