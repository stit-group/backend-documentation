# PostgreSQL Interview - Развернутые Ответы
## 50 ключевых вопросов с практическими решениями

---

## 🎯 Блок 1: SQL Fundamentals & Core Concepts

### 1. **Типы JOIN и их применение**

**Ответ:** PostgreSQL поддерживает несколько типов JOIN, каждый для разных сценариев:

- **INNER JOIN** - только совпадающие записи
- **LEFT JOIN** - все записи из левой таблицы + совпадения справа
- **RIGHT JOIN** - все записи из правой таблицы + совпадения слева  
- **FULL OUTER JOIN** - все записи из обеих таблиц
- **CROSS JOIN** - декартово произведение

```sql
-- LEFT JOIN - показать всех пользователей, даже без заказов
SELECT u.name, COALESCE(o.amount, 0) as order_amount
FROM users u 
LEFT JOIN orders o ON u.id = o.user_id;

-- Составные ключи влияют на производительность через кардинальность
SELECT c.name, p.title 
FROM customers c
JOIN purchases p ON c.id = p.customer_id AND c.region = p.region;
```

**Практический совет:** LEFT JOIN с составными ключами требует индекса на все колонки связи.

### 2. **ACID и MVCC в PostgreSQL**

**Ответ:** PostgreSQL использует MVCC (Multi-Version Concurrency Control) для обеспечения изоляции без блокировки читающих операций. Каждая строка имеет версии, читатели видят консистентный snapshot.

```sql
-- Транзакция 1
BEGIN;
SELECT balance FROM accounts WHERE id = 1; -- видит 1000
-- Пауза...

-- Транзакция 2 (concurrent)  
BEGIN;
UPDATE accounts SET balance = 500 WHERE id = 1;
COMMIT;

-- Транзакция 1 продолжается
SELECT balance FROM accounts WHERE id = 1; -- все еще видит 1000 (Repeatable Read)
COMMIT;
```

**Ключевое:** MVCC позволяет readers никогда не блокироваться writers, что критично для производительности.

### 3. **Window Functions vs GROUP BY**

**Ответ:** GROUP BY агрегирует строки, window functions сохраняют детализацию:

```sql
-- GROUP BY - теряем детали сотрудников
SELECT department, AVG(salary) 
FROM employees 
GROUP BY department;

-- Window function - сохраняем все строки + агрегат
SELECT name, salary, department,
       AVG(salary) OVER (PARTITION BY department) as dept_avg,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) as dept_rank
FROM employees;
```

**Практическое применение:** Window functions незаменимы для ranking, running totals, lag/lead анализа.

### 4. **CTE vs Subqueries**

**Ответ:** CTE обеспечивает читаемость и поддержку рекурсии, subqueries часто быстрее для простых случаев:

```sql
-- Рекурсивная CTE для иерархии
WITH RECURSIVE employee_hierarchy AS (
  SELECT id, name, manager_id, 1 as level
  FROM employees WHERE manager_id IS NULL
  
  UNION ALL
  
  SELECT e.id, e.name, e.manager_id, eh.level + 1
  FROM employees e
  JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM employee_hierarchy;

-- Альтернатива без CTE (только для фиксированной глубины)
SELECT e1.name as employee, e2.name as manager, e3.name as director
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.id  
LEFT JOIN employees e3 ON e2.manager_id = e3.id;
```

**Trade-off:** CTE материализуется один раз, что может быть дороже для больших наборов.

### 5. **Уровни изоляции транзакций**

**Ответ:** Выбор зависит от требований к консистентности vs производительности:

```sql
-- Read Committed - для каталога товаров
BEGIN ISOLATION LEVEL READ COMMITTED;
SELECT * FROM products WHERE available = true;

-- Repeatable Read - для оформления заказа  
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT quantity FROM inventory WHERE product_id = 123;
UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 123;

-- Serializable - для критичных отчетов
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT SUM(amount) FROM transactions WHERE date = CURRENT_DATE;
```

**Практический выбор:** Read Committed для 95% случаев, Repeatable Read для financial операций.

---

## 🚀 Блок 2: Indexing & Query Optimization

### 11. **Типы индексов и их применение**

**Ответ:** Каждый тип индекса оптимизирован под конкретные операции:

```sql
-- B-tree (по умолчанию) - для равенства и диапазонов
CREATE INDEX idx_orders_date ON orders(created_at);

-- GIN - для полнотекстового поиска
CREATE INDEX idx_articles_search ON articles USING GIN(to_tsvector('english', content));

-- GiST - для географических данных
CREATE INDEX idx_locations_geo ON locations USING GIST(coordinates);

-- Hash - только для равенства (редко используется)
CREATE INDEX idx_user_email ON users USING HASH(email);

-- BRIN - для очень больших таблиц с естественной сортировкой
CREATE INDEX idx_logs_time ON logs USING BRIN(timestamp);
```

**Выбор индекса:** GIN для JSON/arrays, GiST для геометрии, B-tree для всего остального.

### 12. **Composite Index Design**

**Ответ:** Порядок колонок критичен - от наиболее селективной к наименее селективной:

```sql
-- Правильный порядок для WHERE status = 'pending' AND created_at > '2024-01-01'
CREATE INDEX idx_orders_optimized ON orders (status, created_at, customer_id);

-- Индекс используется для:
-- ✅ WHERE status = 'pending'
-- ✅ WHERE status = 'pending' AND created_at > '...'  
-- ✅ WHERE status = 'pending' AND created_at > '...' AND customer_id = 1
-- ❌ WHERE created_at > '...' (пропущена первая колонка)
```

**Правило левого префикса:** Индекс работает только если используются колонки слева направо без пропусков.

### 13. **Partial Indexes**

**Ответ:** Partial indexes индексируют только подмножество строк, экономя место и время:

```sql
-- Индекс только для активных заказов (5% от общего объема)
CREATE INDEX idx_orders_active ON orders (customer_id, created_at) 
WHERE status IN ('pending', 'processing');

-- Для soft-delete паттерна
CREATE INDEX idx_users_active ON users (email) 
WHERE deleted_at IS NULL;

-- Для булевых полей с перекосом
CREATE INDEX idx_urgent_tickets ON tickets (created_at) 
WHERE is_urgent = true;
```

**Преимущества:** Меньший размер индекса, быстрее maintenance, точнее статистика.

### 14. **Covering Indexes (INCLUDE)**

**Ответ:** INCLUDE добавляет колонки только в leaf nodes, избегая table lookup:

```sql
-- Index-only scan возможен
CREATE INDEX idx_orders_covering 
ON orders (customer_id) 
INCLUDE (amount, created_at, status);

-- Запрос использует только индекс
SELECT amount, created_at, status 
FROM orders 
WHERE customer_id = 123;
```

**Когда эффективнее:** При частых запросах небольшого количества дополнительных колонок.

---

## 🔧 Блок 3: Advanced PostgreSQL Features

### 21. **Partitioning Strategy**

**Ответ:** Для таблицы логов оптимально range partitioning по времени:

```sql
-- Основная таблица
CREATE TABLE logs (
    id BIGSERIAL,
    timestamp TIMESTAMPTZ NOT NULL,
    level VARCHAR(10),
    message TEXT,
    user_id INTEGER
) PARTITION BY RANGE (timestamp);

-- Партиции по дням
CREATE TABLE logs_2024_06_30 PARTITION OF logs
FOR VALUES FROM ('2024-06-30') TO ('2024-07-01');

-- Автоматическое создание партиций
CREATE OR REPLACE FUNCTION create_daily_partition(table_date DATE)
RETURNS void AS $$
BEGIN
    EXECUTE format('CREATE TABLE IF NOT EXISTS logs_%s PARTITION OF logs
                   FOR VALUES FROM (%L) TO (%L)',
                   to_char(table_date, 'YYYY_MM_DD'),
                   table_date,
                   table_date + interval '1 day');
END;
$$ LANGUAGE plpgsql;
```

**Стратегия maintenance:** Автоматическое создание будущих партиций, удаление старых через retention policy.

### 22. **Row Level Security (RLS)**

**Ответ:** RLS обеспечивает data isolation на уровне строк для multi-tenant приложений:

```sql
-- Включаем RLS
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- Политика для tenant isolation
CREATE POLICY tenant_isolation ON documents
    FOR ALL TO app_user
    USING (tenant_id = current_setting('app.current_tenant_id')::INTEGER);

-- В приложении устанавливаем контекст
SET app.current_tenant_id = 123;
SELECT * FROM documents; -- видит только свой tenant
```

**Преимущества:** Защита на уровне БД, невозможно "забыть" проверить tenant_id в коде.

### 26. **LISTEN/NOTIFY Mechanism**

**Ответ:** Асинхронные уведомления без polling для real-time features:

```sql
-- Trigger для уведомлений
CREATE OR REPLACE FUNCTION notify_order_change()
RETURNS trigger AS $$
BEGIN
    PERFORM pg_notify('order_updates', 
        json_build_object('order_id', NEW.id, 'status', NEW.status)::text);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Устанавливаем trigger
CREATE TRIGGER order_update_trigger
    AFTER UPDATE ON orders
    FOR EACH ROW
    EXECUTE FUNCTION notify_order_change();

-- В приложении слушаем
LISTEN order_updates;
```

**Применение:** Real-time dashboards, cache invalidation, microservices coordination.

---

## ⚡ Блок 4: Performance & Monitoring

### 31. **Memory Configuration**

**Ответ:** Для OLTP на 32GB сервере базовая конфигурация:

```sql
-- postgresql.conf основные параметры
shared_buffers = 8GB              -- 25% от RAM
effective_cache_size = 24GB       -- 75% от RAM  
work_mem = 64MB                   -- для сортировок и hash joins
maintenance_work_mem = 2GB        -- для VACUUM, CREATE INDEX
random_page_cost = 1.1            -- для SSD
effective_io_concurrency = 200    -- для SSD

-- Connection settings
max_connections = 200
shared_preload_libraries = 'pg_stat_statements'
```

**Обоснование:** shared_buffers не больше 25% чтобы не конкурировать с OS cache, work_mem зависит от количества concurrent queries.

### 34. **VACUUM Tuning**

**Ответ:** VACUUM критичен для производительности, особенно для UPDATE-heavy нагрузки:

```sql
-- Анализ необходимости VACUUM
SELECT schemaname, tablename, 
       n_dead_tup, n_live_tup,
       n_dead_tup::float / NULLIF(n_live_tup, 0) AS dead_ratio,
       last_vacuum, last_autovacuum
FROM pg_stat_user_tables 
WHERE n_dead_tup > 1000
ORDER BY dead_ratio DESC;

-- Настройка autovacuum
ALTER TABLE high_update_table SET (
  autovacuum_vacuum_scale_factor = 0.1,    -- vacuum при 10% dead tuples
  autovacuum_analyze_scale_factor = 0.05,  -- analyze при 5% changes
  autovacuum_vacuum_cost_delay = 10        -- снижаем нагрузку на I/O
);

-- Принудительный VACUUM с детальной информацией
VACUUM (VERBOSE, ANALYZE) table_name;
```

**Проблемы:** Высокий dead_ratio указывает на недостаточный autovacuum или transaction wraparound.

### 37. **Lock Contention Resolution**

**Ответ:** Диагностика и решение проблем с блокировками:

```sql
-- Поиск заблокированных запросов
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_statement,
    blocking_activity.query AS blocking_statement,
    blocked_activity.application_name AS blocked_application
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity 
  ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
  ON blocking_locks.locktype = blocked_locks.locktype
  AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
  AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
JOIN pg_catalog.pg_stat_activity blocking_activity 
  ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;

-- Принудительное завершение блокирующего запроса
SELECT pg_terminate_backend(blocking_pid);
```

**Профилактика:** Короткие транзакции, правильный порядок блокировки таблиц, использование advisory locks.

---

## 🏗️ Блок 5: Architecture & Scaling

### 41. **Read Replica Architecture**

**Ответ:** Для 80/20 read/write нагрузки оптимальна master-replica архитектура:

```sql
-- Настройка streaming replication
-- На мастере (postgresql.conf)
wal_level = replica
max_wal_senders = 3
wal_keep_size = '1GB'
archive_mode = on
archive_command = 'cp %p /archive/%f'

-- Создание replica user
CREATE USER replica_user REPLICATION LOGIN PASSWORD 'secure_password';

-- На реплике - инициализация
pg_basebackup -h master_host -D /var/lib/postgresql/data -U replica_user -v -P

-- В приложении - разделение читающей и пишущей нагрузки
// Write operations
const writeDB = new Pool({ host: 'master.db.company.com' });

// Read operations  
const readDB = new Pool({ host: 'replica.db.company.com' });
```

**Архитектурные решения:** Load balancer для read queries, connection pooling, monitoring lag.

### 45. **Backup & Recovery Strategy**

**Ответ:** Комплексная стратегия backup для enterprise требований:

```bash
# Базовый backup с архивацией WAL
pg_basebackup -D /backup/base -Ft -z -P

# Continuous archiving для PITR
archive_command = 'rsync %p backup-server:/archive/%f'

# Point-in-time recovery
pg_ctl stop
rm -rf /var/lib/postgresql/data/*
tar -xf /backup/base.tar.gz -C /var/lib/postgresql/data/
# Создаем recovery.signal
echo "restore_command = 'cp /archive/%f %p'" > /var/lib/postgresql/data/recovery.conf
echo "recovery_target_time = '2024-06-30 14:30:00'" >> /var/lib/postgresql/data/recovery.conf
pg_ctl start
```

```sql
-- Верификация backup
SELECT pg_start_backup('daily_backup', false, false);
-- ... copy files ...  
SELECT pg_stop_backup(false, true);

-- Проверка возможности восстановления
SELECT pg_is_in_recovery();
SELECT pg_last_wal_receive_lsn(), pg_last_wal_replay_lsn();
```

**Компоненты стратегии:** Automated backups, cross-region replication, регулярное тестирование recovery procedures.

### 47. **Connection Pooling at Scale**

**Ответ:** Для microservices архитектуры нужна многоуровневая стратегия:

```yaml
# pgbouncer конфигурация
[databases]
myapp = host=localhost port=5432 dbname=myapp

[pgbouncer]  
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 25
reserve_pool_size = 5
server_idle_timeout = 600
```

```javascript
// В каждом микросервисе - ограниченный pool
const pool = new Pool({
  host: 'pgbouncer.internal',
  port: 6432,
  database: 'myapp',
  max: 10,                    // Максимум 10 соединений на сервис
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});
```

**Архитектура:** PgBouncer как промежуточный слой, мониторинг pool utilization, graceful degradation при перегрузке.

---

## 💡 Ключевые Принципы Optimization

### **Performance First Principles:**
1. **Measure first** - используйте EXPLAIN ANALYZE перед оптимизацией
2. **Index wisely** - не создавайте индексы "на всякий случай"  
3. **Normalize carefully** - иногда денормализация улучшает производительность
4. **Monitor continuously** - pg_stat_statements + custom metrics

### **Troubleshooting Workflow:**
1. **Identify bottleneck** - CPU, I/O, или блокировки?
2. **Check recent changes** - schema, configuration, data volume
3. **Analyze query patterns** - новые queries или изменение планов?
4. **Apply targeted fixes** - не меняйте все сразу

### **Production Considerations:**
- **Connection limits** - всегда настраивайте max_connections под нагрузку
- **Memory tuning** - shared_buffers + effective_cache_size = основа
- **WAL configuration** - критично для write-heavy приложений
- **Monitoring setup** - метрики важнее красивых dashboard'ов

---

*Документ обновлен: Июнь 2025*
*Для production использования всегда тестируйте в staging окружении*