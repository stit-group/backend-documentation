# PostgreSQL Technical Interview
## 50 вопросов с практическим фокусом для Backend разработчика

> **Структура:** 5 тематических блоков по 10 вопросов + практические задачи
> 
> **Время:** 60-90 минут
> 
> **Подход:** От базовых концепций к архитектурным решениям

---

## 🎯 Блок 1: SQL Fundamentals & Core Concepts (10 вопросов)

### 1. **Типы JOIN и их применение**
```sql
-- Объясните результат и когда использовать каждый тип
SELECT u.name, o.amount 
FROM users u 
LEFT JOIN orders o ON u.id = o.user_id;
```
**Follow-up:** Как JOIN с составными ключами влияет на производительность?

### 2. **ACID и MVCC в PostgreSQL**
**Вопрос:** Как PostgreSQL обеспечивает изоляцию транзакций без блокировки читающих операций?
**Практика:** Объясните поведение двух concurrent транзакций на уровне Repeatable Read.

### 3. **Window Functions vs GROUP BY**
```sql
-- Чем отличаются эти запросы и когда использовать каждый?
SELECT department, AVG(salary) FROM employees GROUP BY department;
SELECT name, salary, AVG(salary) OVER (PARTITION BY department) FROM employees;
```

### 4. **CTE vs Subqueries**
**Задача:** Перепишите рекурсивный запрос для обхода иерархии без CTE. Объясните trade-offs.

### 5. **Уровни изоляции транзакций**
**Сценарий:** У вас e-commerce приложение. Какой уровень изоляции выбрать для:
- Просмотра каталога товаров
- Оформления заказа
- Генерации отчетов

### 6. **UPSERT и Conflict Resolution**
```sql
INSERT INTO user_stats (user_id, login_count) 
VALUES (1, 1) 
ON CONFLICT (user_id) 
DO UPDATE SET login_count = user_stats.login_count + 1;
```
**Вопрос:** Какие альтернативы UPSERT и когда их использовать?

### 7. **JSON/JSONB Performance**
**Задача:** У вас таблица с JSONB колонкой user_preferences. Напишите эффективный запрос для поиска пользователей с определенными настройками.

### 8. **Deadlock Prevention**
**Сценарий:** Два concurrent процесса обновляют связанные таблицы. Как предотвратить deadlock на уровне кода?

### 9. **EXPLAIN Analysis**
```sql
EXPLAIN (ANALYZE, BUFFERS) 
SELECT * FROM orders o 
JOIN customers c ON o.customer_id = c.id 
WHERE o.created_at > NOW() - INTERVAL '30 days';
```
**Задача:** Интерпретируйте план и предложите оптимизации.

### 10. **Views vs Materialized Views**
**Кейс:** У вас сложный аналитический запрос, выполняющийся 30 секунд. Решение через views, materialized views или другие подходы?

---

## 🚀 Блок 2: Indexing & Query Optimization (10 вопросов)

### 11. **Типы индексов и их применение**
**Задача:** Для каждого сценария выберите оптимальный тип индекса:
- Полнотекстовый поиск в статьях
- Географические данные (latitude, longitude)
- Поиск по диапазону дат
- JSON атрибуты продуктов

### 12. **Composite Index Design**
```sql
-- Какой порядок колонок оптимален?
CREATE INDEX idx_orders_optimized ON orders (status, created_at, customer_id);
```
**Вопрос:** Объясните правило левого префикса и исключения из него.

### 13. **Partial Indexes**
**Практика:** Создайте partial index для таблицы заказов, где 95% заказов имеют статус 'completed'.

### 14. **Covering Indexes (INCLUDE)**
```sql
CREATE INDEX idx_orders_covering 
ON orders (customer_id) 
INCLUDE (amount, created_at);
```
**Вопрос:** Когда covering index эффективнее join'а с основной таблицей?

### 15. **Index Maintenance**
**Проблема:** После массивного UPDATE операции производительность SELECT упала. Диагностика и решение?

### 16. **Statistics и Query Planner**
**Сценарий:** Query planner выбирает Seq Scan вместо Index Scan для селективного запроса. Возможные причины и решения?

### 17. **Bitmap Scan Optimization**
**Вопрос:** Когда PostgreSQL использует Bitmap Index Scan и как это влияет на memory usage?

### 18. **Expression Indexes**
```sql
-- Оптимизируйте поиск без учета регистра
SELECT * FROM users WHERE LOWER(email) = 'user@example.com';
```

### 19. **Index Bloat и HOT Updates**
**Задача:** У вас таблица с частыми UPDATE операциями. Как минимизировать index bloat?

### 20. **Parallel Query Execution**
**Кейс:** Аналитический запрос на 10M записях выполняется медленно. Когда parallel execution поможет, а когда нет?

---

## 🔧 Блок 3: Advanced PostgreSQL Features (10 вопросов)

### 21. **Partitioning Strategy**
**Задача:** Спроектируйте партиционирование для таблицы логов (1TB, 100M записей в день). Учтите queries и maintenance.

### 22. **Custom Data Types**
**Практика:** Создайте range тип для рабочих часов (9:00-18:00) с валидацией пересечений.

### 23. **Stored Procedures vs Functions**
```plpgsql
-- Когда использовать процедуру, а когда функцию?
CREATE PROCEDURE transfer_money(from_account INT, to_account INT, amount DECIMAL);
CREATE FUNCTION calculate_interest(balance DECIMAL) RETURNS DECIMAL;
```

### 24. **Row Level Security (RLS)**
**Сценарий:** Multi-tenant приложение. Как обеспечить data isolation через RLS?

### 25. **Extensions Ecosystem**
**Вопрос:** Выберите extensions для:
- Полнотекстовый поиск с морфологией
- UUID generation
- Geographical queries
- Time series data

### 26. **LISTEN/NOTIFY Mechanism**
**Кейс:** Реализуйте real-time уведомления об изменениях в таблице без polling.

### 27. **Advisory Locks**
**Задача:** Предотвратите concurrent выполнение job'а без использования внешних инструментов.

### 28. **Foreign Data Wrappers (FDW)**
**Архитектура:** Как интегрировать PostgreSQL с внешними данными (API, другие БД)?

### 29. **Generated Columns**
```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    price DECIMAL,
    tax_rate DECIMAL,
    total_price DECIMAL GENERATED ALWAYS AS (price * (1 + tax_rate)) STORED
);
```
**Вопрос:** STORED vs VIRTUAL columns trade-offs?

### 30. **Logical Replication vs Triggers**
**Проблема:** Синхронизация данных между PostgreSQL и внешней системой. Сравните подходы.

---

## ⚡ Блок 4: Performance & Monitoring (10 вопросов)

### 31. **Memory Configuration**
**Задача:** Настройте PostgreSQL для сервера с 32GB RAM под OLTP нагрузку. Обоснуйте каждый параметр.

### 32. **WAL Configuration**
**Сценарий:** High-write нагрузка вызывает I/O bottleneck. Как оптимизировать WAL?

### 33. **Connection Pooling Strategy**
**Архитектура:** 1000+ concurrent users, microservices. Спроектируйте connection pooling.

### 34. **VACUUM Tuning**
```sql
-- Интерпретируйте результат и предложите решение
SELECT schemaname, tablename, n_dead_tup, n_live_tup,
       n_dead_tup::float / n_live_tup AS ratio
FROM pg_stat_user_tables 
WHERE n_dead_tup > 1000;
```

### 35. **Query Performance Analysis**
**Инструменты:** Сравните pg_stat_statements, auto_explain, и внешние APM. Когда использовать каждый?

### 36. **Checkpoint Optimization**
**Проблема:** Регулярные performance спады каждые 5 минут. Диагностика через checkpoint метрики.

### 37. **Lock Contention Resolution**
```sql
-- Найдите и решите проблему блокировок
SELECT blocked_locks.pid AS blocked_pid,
       blocking_locks.pid AS blocking_pid,
       blocked_activity.query AS blocked_statement
FROM pg_catalog.pg_locks blocked_locks...
```

### 38. **Bloat Detection и Remediation**
**Практика:** Напишите запрос для определения bloated таблиц и индексов. Стратегия очистки.

### 39. **I/O Optimization**
**Кейс:** Database на AWS EBS. Как оптимизировать для лучшего I/O throughput?

### 40. **Resource Monitoring**
**Задача:** Создайте dashboard для мониторинга PostgreSQL health. Ключевые метрики?

---

## 🏗️ Блок 5: Architecture & Scaling (10 вопросов)

### 41. **Read Replica Architecture**
**Проблема:** 80% read, 20% write нагрузка. Спроектируйте масштабируемую архитектуру.

### 42. **Failover Strategy**
**Сценарий:** Primary server failure. Как обеспечить RPO < 1 minute, RTO < 5 minutes?

### 43. **Logical Replication Design**
**Задача:** Миграция с minimal downtime между двумя PostgreSQL кластерами.

### 44. **Sharding Approaches**
**Архитектура:** 100M+ users, geographical distribution. Horizontal scaling стратегия?

### 45. **Backup & Recovery Strategy**
**Требования:** 
- Point-in-time recovery за последние 30 дней
- Backup verification
- Cross-region disaster recovery

### 46. **High Availability с Patroni**
**Практика:** Настройте 3-node PostgreSQL кластер с автоматическим failover.

### 47. **Connection Pooling at Scale**
**Проблема:** 50+ microservices, каждый с connection pool. Как избежать connection exhaustion?

### 48. **Data Lifecycle Management**
**Кейс:** Time-series данные, 1TB/месяц прироста. Архивирование и purging стратегия?

### 49. **Multi-Region Setup**
**Требования:**
- Active-Passive replication
- Disaster recovery
- Compliance requirements

### 50. **PostgreSQL vs Alternatives**
**Архитектурное решение:** Когда выбрать PostgreSQL, а когда рассмотреть MongoDB, Cassandra, или MySQL?

---

## 🛠️ Практические Задачи

### **Задача A: Performance Investigation**
```
Проблема: Медленные запросы в production
Данные: pg_stat_statements показывает high mean_time
Требуется: План диагностики и оптимизации
```

### **Задача B: Schema Design**
```
Спроектируйте схему для системы бронирования:
- Hotels, Rooms, Bookings
- Temporal data (availability)
- Concurrent booking prevention
- Efficient queries для поиска
```

### **Задача C: Migration Planning**
```
Миграция 500GB PostgreSQL database:
- Zero-downtime requirement
- Version upgrade (12 → 15)
- Schema changes
- Rollback strategy
```

---

## 📊 Система Оценки

### **Junior (20-30 вопросов)**
- ✅ SQL fundamentals
- ✅ Basic indexing
- ✅ Transaction basics
- ⚠️ Limited optimization knowledge

### **Middle (30-40 вопросов)**
- ✅ Advanced SQL
- ✅ Index optimization
- ✅ Performance basics
- ✅ Some architecture understanding

### **Senior (40-45 вопросов)**
- ✅ Deep PostgreSQL knowledge
- ✅ Complex optimization
- ✅ Architecture design
- ✅ Production troubleshooting

### **Lead/Principal (45+ вопросов)**
- ✅ Expert-level knowledge
- ✅ Strategic decision making
- ✅ Cross-system integration
- ✅ Team guidance capability

---

## 💡 Interview Tips

### **Для интервьюера:**
1. **Начинайте с практических сценариев** - это покажет реальный опыт
2. **Углубляйтесь в зависимости от ответов** - адаптируйте сложность
3. **Просите объяснить trade-offs** - это показывает понимание архитектуры
4. **Обращайте внимание на troubleshooting подход** - как кандидат решает проблемы

### **Для кандидата:**
1. **Думайте вслух** - объясняйте свой мыслительный процесс
2. **Приводите примеры из опыта** - конкретные кейсы из практики
3. **Не бойтесь сказать "не знаю"** - лучше честность, чем выдумки
4. **Задавайте уточняющие вопросы** - это показывает системное мышление

---

## 🔗 Дополнительные Ресурсы

**Документация:**
- [PostgreSQL Official Docs](https://www.postgresql.org/docs/)
- [PostgreSQL Performance Tuning](https://wiki.postgresql.org/wiki/Performance_Optimization)

**Инструменты:**
- pg_stat_statements
- pgbench
- explain.depesz.com
- pgtune.leopard.in.ua

**Мониторинг:**
- Datadog PostgreSQL integration
- Prometheus + Grafana
- pgbadger для анализа логов

---

*Последнее обновление: Июнь 2025*