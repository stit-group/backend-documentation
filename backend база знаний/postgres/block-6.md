# Блок 6: Администрирование и безопасность PostgreSQL

> **🎯 Цель:** Научиться управлять PostgreSQL в продакшене  
> **⏱️ Продолжительность:** 2-3 недели  
> **📊 Сложность:** Продвинутый

---

## 📋 Содержание блока

```
┌─────────────────────────────────────────────────────────────────┐
│ 6.1 Пользователи, роли и права доступа              (4-5 дней) │
│ 6.2 Резервное копирование и восстановление          (4-5 дней) │
│ 6.3 Мониторинг и логирование                        (3-4 дня)  │
│ 6.4 Настройка производительности сервера            (4-5 дней) │
│ 6.5 Репликация и высокая доступность                (5-6 дней) │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6.1 Пользователи, роли и права доступа (4-5 дней)

### 🔐 Концепция ролей в PostgreSQL

В PostgreSQL **нет различия между пользователями и ролями** - все является ролями. Роль может:
- Подключаться к базе данных (если имеет атрибут LOGIN)
- Владеть объектами базы данных
- Наследовать права от других ролей

```
┌─────────────────────────────────────────────────────────────┐
│                    ИЕРАРХИЯ РОЛЕЙ                           │
│                                                             │
│         superuser (postgres)                                │
│                    │                                        │
│         ┌──────────┴──────────┐                            │
│    db_admin                app_admin                        │
│         │                      │                           │
│    ┌────┴────┐           ┌─────┴─────┐                     │
│  analyst  reporter    web_user    api_user                 │
└─────────────────────────────────────────────────────────────┘
```

### 🛠️ Создание пользователей и ролей

#### Создание роли без права входа (группа)
```sql
-- Создание группы ролей
CREATE ROLE developers;
CREATE ROLE analysts;
CREATE ROLE admins;
```

#### Создание пользователя с правом входа
```sql
-- Создание пользователя
CREATE ROLE john_dev 
    LOGIN 
    PASSWORD 'secure_password123'
    VALID UNTIL '2025-12-31';

-- Альтернативный синтаксис
CREATE USER jane_analyst 
    PASSWORD 'another_secure_pass'
    IN ROLE analysts;
```

#### Модификация ролей
```sql
-- Добавление прав
ALTER ROLE john_dev CREATEDB;
ALTER ROLE jane_analyst SET work_mem = '256MB';

-- Изменение пароля
ALTER ROLE john_dev PASSWORD 'new_password456';

-- Добавление роли в группу
GRANT developers TO john_dev;
```

### 📊 Права доступа (GRANT/REVOKE)

#### Основные типы прав
```
┌──────────────────────────────────────────────────────────────┐
│ SELECT   │ Чтение данных из таблиц/представлений              │
│ INSERT   │ Вставка новых строк                                │
│ UPDATE   │ Изменение существующих строк                       │
│ DELETE   │ Удаление строк                                     │
│ USAGE    │ Использование схем, последовательностей, типов     │
│ CREATE   │ Создание объектов в схеме/базе                     │
│ CONNECT  │ Подключение к базе данных                          │
│ EXECUTE  │ Выполнение функций/процедур                        │
└──────────────────────────────────────────────────────────────┘
```

#### Примеры назначения прав
```sql
-- Права на уровне базы данных
GRANT CONNECT ON DATABASE myapp TO developers;
GRANT CREATE ON DATABASE myapp TO admins;

-- Права на уровне схемы
GRANT USAGE ON SCHEMA public TO developers;
GRANT CREATE ON SCHEMA public TO developers;

-- Права на уровне таблицы
GRANT SELECT, INSERT, UPDATE ON users TO web_app;
GRANT SELECT ON users TO analysts;

-- Права на все таблицы в схеме
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_role;

-- Права по умолчанию для новых объектов
ALTER DEFAULT PRIVILEGES IN SCHEMA public 
    GRANT SELECT ON TABLES TO readonly_role;
```

#### Отзыв прав
```sql
-- Отзыв конкретных прав
REVOKE INSERT ON users FROM web_app;

-- Отзыв всех прав
REVOKE ALL ON users FROM john_dev;

-- Отзыв роли
REVOKE developers FROM john_dev;
```

### 🔒 Row Level Security (RLS)

RLS позволяет ограничить доступ к строкам таблицы на основе пользователя или роли.

```sql
-- Включение RLS для таблицы
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- Создание политики безопасности
CREATE POLICY user_documents ON documents
    FOR ALL TO users
    USING (owner_id = current_user_id());

-- Политика только для чтения
CREATE POLICY read_own_orders ON orders
    FOR SELECT TO customers
    USING (customer_id = get_current_customer_id());

-- Политика для администраторов (полный доступ)
CREATE POLICY admin_all_access ON documents
    FOR ALL TO admins
    USING (true);
```

### 🛡️ Лучшие практики безопасности

```
┌─────────────────────────────────────────────────────────────┐
│ ✅ РЕКОМЕНДУЕТСЯ                                            │
├─────────────────────────────────────────────────────────────┤
│ • Принцип минимальных привилегий                            │
│ • Использование групп ролей вместо прямых назначений        │
│ • Регулярная ротация паролей                                │
│ • Использование SSL соединений                              │
│ • Ограничение подключений по IP                             │
│ • Логирование всех подключений                              │
│                                                             │
│ ❌ НЕ РЕКОМЕНДУЕТСЯ                                          │
│ • Использование роли postgres для приложений                │
│ • Пустые или слабые пароли                                  │
│ • Широкие права доступа "на всякий случай"                  │
│ • Общие аккаунты для нескольких разработчиков               │
└─────────────────────────────────────────────────────────────┘
```

---

## 6.2 Резервное копирование и восстановление (4-5 дней)

### 💾 Типы резервного копирования

```
┌─────────────────────────────────────────────────────────────┐
│                 ТИПЫ BACKUP'ОВ                              │
│                                                             │
│  Логическое копирование    │    Физическое копирование      │
│  ┌─────────────────────┐   │   ┌─────────────────────┐      │
│  │ • pg_dump/pg_dumpall│   │   │ • pg_basebackup     │      │
│  │ • SQL команды       │   │   │ • Файлы данных      │      │
│  │ • Кроссплатформенно │   │   │ • WAL архивы        │      │
│  │ • Медленнее         │   │   │ • Быстрее           │      │
│  └─────────────────────┘   │   └─────────────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### 🗃️ pg_dump и pg_restore

#### Базовое использование pg_dump
```bash
# Полный дамп базы данных
pg_dump -h localhost -U postgres -d myapp > myapp_backup.sql

# Дамп с сжатием
pg_dump -h localhost -U postgres -d myapp -Fc > myapp_backup.dump

# Только схема (без данных)
pg_dump -h localhost -U postgres -d myapp --schema-only > schema_only.sql

# Только данные (без схемы)
pg_dump -h localhost -U postgres -d myapp --data-only > data_only.sql

# Исключение таблиц
pg_dump -h localhost -U postgres -d myapp --exclude-table=logs > myapp_no_logs.sql
```

#### Восстановление с pg_restore
```bash
# Восстановление из custom формата
pg_restore -h localhost -U postgres -d myapp_restored myapp_backup.dump

# Восстановление с очисткой
pg_restore -h localhost -U postgres -d myapp_restored --clean myapp_backup.dump

# Восстановление только схемы
pg_restore -h localhost -U postgres -d myapp_restored --schema-only myapp_backup.dump

# Параллельное восстановление
pg_restore -h localhost -U postgres -d myapp_restored -j 4 myapp_backup.dump
```

### 📁 Непрерывное архивирование (WAL)

WAL (Write-Ahead Logging) позволяет создавать непрерывные backup'ы и восстанавливать данные на любой момент времени.

#### Настройка WAL архивирования в postgresql.conf
```ini
# Включение архивирования
wal_level = replica
archive_mode = on
archive_command = 'cp %p /backup/wal_archive/%f'

# Настройка checkpoint'ов
checkpoint_timeout = 15min
checkpoint_completion_target = 0.9
```

#### Создание базового backup'а
```bash
# Создание базового backup'а
pg_basebackup -h localhost -U postgres -D /backup/base_backup -Ft -z -P

# С включением WAL файлов
pg_basebackup -h localhost -U postgres -D /backup/base_backup -Ft -z -P -W
```

### ⏰ Point-in-Time Recovery (PITR)

PITR позволяет восстановить базу данных на конкретный момент времени.

```
┌─────────────────────────────────────────────────────────────┐
│                    СХЕМА PITR                               │
│                                                             │
│  Base Backup    WAL Files         Target Time               │
│      │             │                  │                    │
│   ┌──▼──┐       ┌──▼──┐            ┌──▼──┐                 │
│   │00:00│──────▶│01:00│──────▶─────│02:30│                 │
│   └─────┘       └─────┘            └─────┘                 │
│                                                             │
│  Восстановление = Base Backup + WAL до 02:30               │
└─────────────────────────────────────────────────────────────┘
```

#### Процедура восстановления PITR
```bash
# 1. Остановить PostgreSQL
systemctl stop postgresql

# 2. Очистить data директорию
rm -rf /var/lib/postgresql/13/main/*

# 3. Восстановить базовый backup
tar -xf /backup/base_backup/base.tar.gz -C /var/lib/postgresql/13/main/

# 4. Создать recovery.conf
cat > /var/lib/postgresql/13/main/recovery.conf << EOF
restore_command = 'cp /backup/wal_archive/%f %p'
recovery_target_time = '2024-01-15 14:30:00'
recovery_target_action = 'promote'
EOF

# 5. Запустить PostgreSQL
systemctl start postgresql
```

### 🔄 Стратегии резервного копирования

#### Стратегия 3-2-1
```
┌─────────────────────────────────────────────────────────────┐
│                   ПРАВИЛО 3-2-1                             │
│                                                             │
│   3 копии данных  │  2 разных носителя  │  1 offsite копия  │
│                   │                     │                   │
│  ┌─────────────┐  │  ┌─────────────┐    │ ┌─────────────┐   │
│  │ Оригинал    │  │  │ Локальный   │    │ │ Облачное    │   │
│  │ в продакшене│  │  │ backup      │    │ │ хранилище   │   │
│  └─────────────┘  │  └─────────────┘    │ └─────────────┘   │
│                   │                     │                   │
│  ┌─────────────┐  │  ┌─────────────┐    │                   │
│  │ Локальная   │  │  │ Внешний     │    │                   │
│  │ копия       │  │  │ диск/лента  │    │                   │
│  └─────────────┘  │  └─────────────┘    │                   │
└─────────────────────────────────────────────────────────────┘
```

#### Пример скрипта для автоматического backup'а
```bash
#!/bin/bash
# backup_script.sh

# Переменные
DB_NAME="myapp"
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=7

# Создание backup'а
pg_dump -h localhost -U postgres -d $DB_NAME -Fc > "$BACKUP_DIR/backup_${DB_NAME}_${DATE}.dump"

# Проверка успешности
if [ $? -eq 0 ]; then
    echo "Backup успешно создан: backup_${DB_NAME}_${DATE}.dump"
    
    # Удаление старых backup'ов
    find $BACKUP_DIR -name "backup_${DB_NAME}_*.dump" -mtime +$RETENTION_DAYS -delete
    
    # Отправка в облако (опционально)
    # aws s3 cp "$BACKUP_DIR/backup_${DB_NAME}_${DATE}.dump" s3://my-backup-bucket/
else
    echo "Ошибка при создании backup'а!"
    exit 1
fi
```

---

## 6.3 Мониторинг и логирование (3-4 дня)

### 📊 Системные представления для мониторинга

PostgreSQL предоставляет множество системных представлений для мониторинга состояния сервера.

#### Ключевые представления pg_stat_*
```sql
-- Активность по базам данных
SELECT datname, numbackends, xact_commit, xact_rollback, 
       blks_read, blks_hit, temp_files, temp_bytes
FROM pg_stat_database;

-- Активность по таблицам
SELECT schemaname, tablename, seq_scan, seq_tup_read, 
       idx_scan, idx_tup_fetch, n_tup_ins, n_tup_upd, n_tup_del
FROM pg_stat_user_tables;

-- Использование индексов
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Текущие подключения и запросы
SELECT pid, usename, application_name, client_addr, state, 
       query_start, query
FROM pg_stat_activity
WHERE state = 'active';
```

### 📝 Настройка логирования

#### Основные параметры в postgresql.conf
```ini
# Что логировать
log_statement = 'all'              # none, ddl, mod, all
log_duration = on                  # Время выполнения запросов
log_min_duration_statement = 1000  # Запросы дольше 1 секунды

# Куда логировать
logging_collector = on
log_directory = 'log'
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_rotation_age = 1d
log_rotation_size = 100MB

# Формат логов
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '
log_timezone = 'UTC'
```

#### Анализ медленных запросов
```sql
-- Включение расширения pg_stat_statements
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Топ медленных запросов
SELECT query, calls, total_time, mean_time, rows
FROM pg_stat_statements 
ORDER BY mean_time DESC 
LIMIT 10;

-- Сброс статистики
SELECT pg_stat_statements_reset();
```

### 📈 Мониторинг производительности

#### Проверка блокировок
```sql
-- Текущие блокировки
SELECT blocked_locks.pid AS blocked_pid,
       blocked_activity.usename AS blocked_user,
       blocking_locks.pid AS blocking_pid,
       blocking_activity.usename AS blocking_user,
       blocked_activity.query AS blocked_statement,
       blocking_activity.query AS current_statement_in_blocking_process
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity 
    ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
JOIN pg_catalog.pg_stat_activity blocking_activity 
    ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

#### Мониторинг использования памяти
```sql
-- Размеры баз данных
SELECT datname, pg_size_pretty(pg_database_size(datname)) AS size
FROM pg_database
ORDER BY pg_database_size(datname) DESC;

-- Размеры таблиц
SELECT schemaname, tablename, 
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 10;
```

---

## 6.4 Настройка производительности сервера (4-5 дней)

### ⚙️ Основные параметры postgresql.conf

```
┌─────────────────────────────────────────────────────────────┐
│               КАТЕГОРИИ ПАРАМЕТРОВ                          │
│                                                             │
│  Память          │  I/O            │  Планировщик          │
│  ┌─────────────┐ │ ┌─────────────┐ │ ┌─────────────┐       │
│  │shared_buffers│ │ │checkpoint_  │ │ │effective_   │       │
│  │work_mem     │ │ │segments     │ │ │cache_size   │       │
│  │maintenance_ │ │ │wal_buffers  │ │ │random_page_ │       │
│  │work_mem     │ │ │synchronous_ │ │ │cost         │       │
│  └─────────────┘ │ │commit       │ │ └─────────────┘       │
│                  │ └─────────────┘ │                       │
└─────────────────────────────────────────────────────────────┘
```

#### Настройки памяти
```ini
# Разделяемая память для кэша страниц (25% от RAM)
shared_buffers = 2GB

# Память для операций сортировки/хеширования (на соединение)
work_mem = 64MB

# Память для операций обслуживания (VACUUM, CREATE INDEX)
maintenance_work_mem = 512MB

# Максимальное количество подключений
max_connections = 200

# Эффективный размер кэша ОС (50-75% от RAM)
effective_cache_size = 6GB
```

#### Настройки I/O
```ini
# Контрольные точки
checkpoint_timeout = 15min
checkpoint_completion_target = 0.9
max_wal_size = 4GB
min_wal_size = 1GB

# WAL буферы
wal_buffers = 64MB

# Синхронизация записи
synchronous_commit = on
fsync = on

# Случайный I/O
random_page_cost = 1.1  # Для SSD
seq_page_cost = 1.0
```

### 🔧 Автонастройка с pgtune

pgtune - это инструмент для автоматической генерации оптимальных настроек PostgreSQL.

```bash
# Установка pgtune
pip install pgtune

# Генерация настроек
pgtune -i postgresql.conf -o postgresql_tuned.conf \
       --memory=8GB \
       --type=web \
       --connections=200
```

#### Типы рабочих нагрузок
```
┌─────────────────────────────────────────────────────────────┐
│                ТИПЫ НАГРУЗОК В PGTUNE                       │
│                                                             │
│  web        │ Веб-приложения (много коротких транзакций)    │
│  oltp       │ OLTP системы (много одновременных соединений) │
│  dw         │ Data Warehouse (аналитические запросы)        │
│  desktop    │ Десктопные приложения                         │
│  mixed      │ Смешанная нагрузка                            │
└─────────────────────────────────────────────────────────────┘
```

### 📊 Мониторинг системных ресурсов

#### Использование ресурсов сервера
```bash
# CPU нагрузка
top -p $(pgrep postgres)

# Память PostgreSQL процессов
ps aux | grep postgres | awk '{sum+=$6} END {print "Total Memory:", sum/1024, "MB"}'

# I/O статистика
iostat -x 1

# Использование дискового пространства
df -h /var/lib/postgresql/
```

#### Мониторинг с помощью SQL
```sql
-- Статистика кэша буферов
SELECT 
    c.relname,
    count(*) AS buffers,
    round(100.0 * count(*) / (SELECT setting FROM pg_settings WHERE name='shared_buffers')::integer,1) AS percent
FROM pg_class c
INNER JOIN pg_buffercache b ON b.relfilenode = c.relfilenode
INNER JOIN pg_database d ON (b.reldatabase = d.oid AND d.datname = current_database())
GROUP BY c.relname
ORDER BY buffers DESC
LIMIT 10;

-- Hit ratio для разных объектов
SELECT 
    'Tables' as object_type,
    sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) * 100 as hit_ratio
FROM pg_statio_user_tables
UNION ALL
SELECT 
    'Indexes' as object_type,
    sum(idx_blks_hit) / (sum(idx_blks_hit) + sum(idx_blks_read)) * 100 as hit_ratio
FROM pg_statio_user_indexes;
```

---

## 6.5 Репликация и высокая доступность (5-6 дней)

### 🔄 Потоковая репликация

Потоковая репликация позволяет создать точную копию основного сервера в реальном времени.

```
┌─────────────────────────────────────────────────────────────┐
│                 СХЕМА РЕПЛИКАЦИИ                            │
│                                                             │
│  Master (Primary)           Slave (Standby)                 │
│  ┌─────────────────┐       ┌─────────────────┐             │
│  │                 │ WAL   │                 │             │
│  │   Application   │──────▶│   Read-only     │             │
│  │                 │stream │   Queries       │             │
│  │  Read + Write   │       │                 │             │
│  └─────────────────┘       └─────────────────┘             │
└─────────────────────────────────────────────────────────────┘
```

#### Настройка Master сервера
```ini
# postgresql.conf на Master
wal_level = replica
max_wal_senders = 3
max_replication_slots = 3
synchronous_standby_names = 'standby1'  # Для синхронной репликации
```

```ini
# pg_hba.conf на Master
# Разрешить подключения для репликации
host replication replicator 192.168.1.100/24 md5
```

#### Создание пользователя для репликации
```sql
-- На Master сервере
CREATE ROLE replicator REPLICATION LOGIN PASSWORD 'replica_password';
```

#### Настройка Standby сервера
```bash
# Создание базового backup'а для Standby
pg_basebackup -h master_server -D /var/lib/postgresql/13/main \
              -U replicator -P -W -R

# Файл recovery.conf (PostgreSQL < 12) или postgresql.conf (PostgreSQL >= 12)
# standby.signal файл должен существовать
touch /var/lib/postgresql/13/main/standby.signal
```

```ini
# postgresql.conf на Standby
primary_conninfo = 'host=master_server port=5432 user=replicator password=replica_password'
hot_standby = on
```

### ⚡ Синхронная и асинхронная репликация

#### Различия между типами репликации
```
┌─────────────────────────────────────────────────────────────┐
│              СИНХРОННАЯ vs АСИНХРОННАЯ                      │
│                                                             │
│  Синхронная                 │  Асинхронная                  │
│  ┌─────────────────────────┐│ ┌─────────────────────────┐   │
│  │ ✅ Нет потери данных    ││ │ ⚡ Высокая производ.     │   │
│  │ ❌ Медленнее           ││ │ ❌ Возможна потеря данных│   │
│  │ ❌ Зависит от Standby   ││ │ ✅ Не зависит от Standby │   │
│  └─────────────────────────┘│ └─────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

#### Настройка синхронной репликации
```ini
# На Master
synchronous_standby_names = 'FIRST 1 (standby1, standby2)'
synchronous_commit = on
```

### 🔄 Логическая репликация

Логическая репликация позволяет реплицировать отдельные таблицы или части данных.

#### Настройка публикации (Publisher)
```sql
-- Включение логической репликации
ALTER SYSTEM SET wal_level = logical;
-- Перезапуск сервера требуется

-- Создание публикации
CREATE PUBLICATION my_publication FOR TABLE users, orders;

-- Публикация всех таблиц
CREATE PUBLICATION all_tables FOR ALL TABLES;
```

#### Настройка подписки (Subscriber)
```sql
-- Создание подписки
CREATE SUBSCRIPTION my_subscription 
    CONNECTION 'host=publisher_host dbname=mydb user=replication_user password=password' 
    PUBLICATION my_publication;

-- Проверка статуса подписки
SELECT * FROM pg_subscription;
SELECT * FROM pg_stat_subscription;
```

### 🚀 Переключение на резервный сервер (Failover)

#### Ручное переключение
```bash
# На Standby сервере
# 1. Остановить репликацию
pg_ctl promote -D /var/lib/postgresql/13/main

# 2. Убедиться, что сервер работает в режиме read-write
psql -c "SELECT pg_is_in_recovery();"  # Должно вернуть false
```

#### Автоматическое переключение с Patroni
```yaml
# patroni.yml
scope: postgres-cluster
namespace: /db/
name: node1

restapi:
  listen: 0.0.0.0:8008
  connect_address: 192.168.1.10:8008

etcd:
  hosts: 192.168.1.10:2379,192.168.1.11:2379,192.168.1.12:2379

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 30
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
      parameters:
        wal_level: replica
        hot_standby: "on"
        max_wal_senders: 3
        max_replication_slots: 3

postgresql:
  listen: 0.0.0.0:5432
  connect_address: 192.168.1.10:5432
  data_dir: /var/lib/postgresql/13/main
  pgpass: /tmp/pgpass
  authentication:
    replication:
      username: replicator
      password: replica_password
    superuser:
      username: postgres
      password: postgres_password
```

### 🏗️ Кластерные решения

#### Сравнение решений для HA
```
┌─────────────────────────────────────────────────────────────┐
│                   HA РЕШЕНИЯ                                │
│                                                             │
│  Patroni          │ Stolon          │ Repmgr               │
│  ┌──────────────┐ │ ┌──────────────┐│ ┌──────────────┐     │
│  │• etcd/consul │ │ │• etcd        ││ │• Простота    │     │
│  │• Автофейловер│ │ │• Kubernetes  ││ │• Легкая      │     │
│  │• REST API    │ │ │• Cloud ready ││ │  настройка   │     │
│  └──────────────┘ │ └──────────────┘│ └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎯 Практические задания блока

### Задание 1: Настройка безопасности
1. Создайте иерархию ролей для веб-приложения
2. Настройте Row Level Security для таблицы пользователей
3. Создайте политики доступа к данным

### Задание 2: Резервное копирование
1. Настройте автоматическое создание backup'ов
2. Реализуйте PITR с восстановлением на заданное время
3. Создайте скрипт для проверки целостности backup'ов

### Задание 3: Мониторинг
1. Настройте логирование медленных запросов
2. Создайте дашборд для мониторинга производительности
3. Настройте алерты на критические события

### Задание 4: Репликация
1. Настройте потоковую репликацию master-slave
2. Проведите тестирование failover процедуры
3. Настройте мониторинг задержки репликации

---

## ✅ Результат блока

После изучения этого блока вы будете уметь:

- **Управлять пользователями и правами доступа** в PostgreSQL
- **Создавать и восстанавливать backup'ы** различными способами
- **Настраивать мониторинг и логирование** для production среды
- **Оптимизировать производительность сервера** через конфигурацию
- **Настраивать репликацию и обеспечивать высокую доступность**

### 🎓 Ключевые навыки
```
┌─────────────────────────────────────────────────────────────┐
│ ✅ Администрирование PostgreSQL в продакшене                │
│ ✅ Обеспечение безопасности и контроля доступа              │
│ ✅ Планирование и реализация стратегий backup'а             │
│ ✅ Мониторинг и диагностика проблем производительности      │
│ ✅ Настройка высокодоступных кластеров                      │
└─────────────────────────────────────────────────────────────┘
```

---

## 📚 Дополнительные ресурсы

- **Документация PostgreSQL:** [Administration](https://www.postgresql.org/docs/current/admin.html)
- **pgTune:** [Онлайн конфигуратор](https://pgtune.leopard.in.ua/)
- **Patroni:** [Документация по HA кластерам](https://patroni.readthedocs.io/)
- **pgBackRest:** [Продвинутое резервное копирование](https://pgbackrest.org/)

**Следующий блок:** [Блок 7: Экспертные возможности PostgreSQL] 🚀