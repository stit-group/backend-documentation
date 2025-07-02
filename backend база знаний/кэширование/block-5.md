# 🔄 Блок 5: Распределенное кэширование

**⏱️ Продолжительность:** 3 недели  
**🎯 Цель:** Построить масштабируемые кэширующие системы для высоконагруженных приложений

---

## 📚 Глава 1: Консистентное хеширование

### 🎯 Введение

Представьте ситуацию: у вас есть 3 кэш-сервера и вы хотите равномерно распределить между ними 1000 ключей. Простое решение — использовать обычное хеширование:

```
server_id = hash(key) % number_of_servers
```

### ⚠️ Проблема обычного хеширования

```
Исходное состояние (3 сервера):
------------------------------------
Server 0: ключи 0, 3, 6, 9, ...
Server 1: ключи 1, 4, 7, 10, ...
Server 2: ключи 2, 5, 8, 11, ...
------------------------------------

После добавления 4-го сервера:
------------------------------------
Server 0: ключи 0, 4, 8, 12, ...
Server 1: ключи 1, 5, 9, 13, ...
Server 2: ключи 2, 6, 10, 14, ...
Server 3: ключи 3, 7, 11, 15, ...
------------------------------------
```

**Результат:** 75% ключей переместились на другие серверы! Это означает массовые cache miss'ы.

### 🔧 Решение: Консистентное хеширование

Консистентное хеширование решает эту проблему, минимизируя количество перемещений при изменении количества серверов.

#### Принцип работы:

```
        0°
        ↑
   315°   45°
      \ /
270° ← • → 90°
      / \
   225°   135°
        ↓
       180°

Кольцо хеширования (0-360°)
```

**Алгоритм:**
1. Размещаем серверы на кольце по хешу их идентификаторов
2. Размещаем ключи на кольце по их хешу
3. Каждый ключ обслуживается ближайшим сервером по часовой стрелке

#### Визуализация размещения:

```
        Server A (45°)
        ↑
   Key3   Key1
      \ /
Server C ← • → Key2
   (270°)    / \
   Key4   Key5
        ↓
    Server B (180°)
```

### 🎲 Virtual Nodes (Виртуальные узлы)

Для более равномерного распределения используются виртуальные узлы:

```
Без виртуальных узлов:
----------------------
Server A: 45°
Server B: 180°
Server C: 270°

Нагрузка неравномерная!

С виртуальными узлами (по 3 на сервер):
---------------------------------------
Server A: 45°, 120°, 300°
Server B: 60°, 180°, 330°  
Server C: 90°, 210°, 270°

Нагрузка равномерная!
```

### 💻 Практическая реализация

```python
import hashlib
import bisect

class ConsistentHash:
    def __init__(self, replicas=3):
        self.replicas = replicas
        self.ring = {}
        self.sorted_keys = []
    
    def _hash(self, key):
        return int(hashlib.md5(key.encode()).hexdigest(), 16)
    
    def add_node(self, node):
        for i in range(self.replicas):
            virtual_key = f"{node}:{i}"
            key = self._hash(virtual_key)
            self.ring[key] = node
            bisect.insort(self.sorted_keys, key)
    
    def remove_node(self, node):
        for i in range(self.replicas):
            virtual_key = f"{node}:{i}"
            key = self._hash(virtual_key)
            del self.ring[key]
            self.sorted_keys.remove(key)
    
    def get_node(self, key):
        if not self.ring:
            return None
        
        hash_key = self._hash(key)
        idx = bisect.bisect_right(self.sorted_keys, hash_key)
        if idx == len(self.sorted_keys):
            idx = 0
        return self.ring[self.sorted_keys[idx]]
```

### 📊 Сравнение эффективности

```
Сценарий: удаление 1 из 4 серверов

Обычное хеширование:
--------------------
Перемещено ключей: 75%
Cache miss rate: 75%

Консистентное хеширование:
--------------------------
Перемещено ключей: 25%
Cache miss rate: 25%

Выигрыш в 3 раза!
```

---

## 📚 Глава 2: Репликация и шардирование

### 🔄 Репликация данных

Репликация обеспечивает высокую доступность и отказоустойчивость кэша.

#### Master-Slave репликация:

```
    Client
      ↓
   [Master]     ← запись
      ↓
   ┌─────┬─────┐
   ↓     ↓     ↓
[Slave1][Slave2][Slave3]  ← чтение
```

**Характеристики:**
- ✅ Высокая производительность чтения
- ✅ Простота реализации  
- ❌ Единая точка отказа (master)
- ❌ Возможна потеря данных при падении master

#### Master-Master репликация:

```
[Master A] ↔ [Master B]
    ↑           ↑
    └─ Client ──┘
```

**Характеристики:**
- ✅ Нет единой точки отказа
- ✅ Балансировка записи
- ❌ Сложность разрешения конфликтов
- ❌ Сложность синхронизации

### 🗂️ Шардирование (Partitioning)

Разделение данных между несколькими узлами для горизонтального масштабирования.

#### Горизонтальное шардирование:

```
Данные пользователей:
--------------------
User ID: 1-1000   → Shard A
User ID: 1001-2000 → Shard B  
User ID: 2001-3000 → Shard C
User ID: 3001-4000 → Shard D
```

#### Шардирование по hash'у:

```
hash(key) % num_shards = shard_id

Example:
--------
hash("user:123") % 4 = 1 → Shard B
hash("user:456") % 4 = 3 → Shard D
```

### 🔧 Стратегии Failover

#### Автоматический Failover:

```
Нормальная работа:
-----------------
Client → Master A (активный)
         Master B (standby)

После отказа Master A:
---------------------
Client → Master B (активный)
         Master A (недоступен)

Время переключения: 1-5 секунд
```

#### Graceful Degradation:

```
Все 4 шарда работают:
--------------------
Performance: 100%
Capacity: 100%

1 шард недоступен:
-----------------
Performance: 75%
Capacity: 75%
Strategy: перенаправление на другие шарды
```

### 💾 Recovery стратегии

#### Warm Standby:

```
[Primary] → sync → [Standby]
    ↓               ↓
  Hot cache      Warm cache
  (все данные)   (часть данных)

Recovery time: секунды
```

#### Cold Standby:

```
[Primary] → backup → [Storage]
                        ↓
                   [New instance]
                        ↓
                   Восстановление

Recovery time: минуты/часы
```

---

## 📚 Глава 3: Паттерны распределенного кэширования

### 🔄 Cache-Aside в распределенной среде

```
Application Logic:
------------------

1. Чтение:
   ┌─ get(key) → Cache Cluster
   │              ↓
   │         [Node A][Node B][Node C]
   │              ↓
   │         cache miss/hit
   │              ↓
   │         if miss: → Database
   │              ↓
   │         put(key, value) → Cache

2. Запись:
   ┌─ save(data) → Database
   │              ↓
   │         invalidate(key) → Cache Cluster
```

#### Проблемы в распределенной среде:

```
Race Condition:
--------------
Thread 1: invalidate(key) → [Node A]
Thread 2: get(key) → miss → DB → put(key, old_value)

Результат: устаревшие данные в кэше!

Решение: версионирование
```

### 🔄 Write-Through кэширование

```
Write-Through Pattern:
---------------------

Application
    ↓ write(key, value)
Cache Cluster
    ↓ (синхронно)
Database

Преимущества:
✅ Консистентность данных
✅ Всегда актуальный кэш

Недостатки:
❌ Высокая латентность записи
❌ Нагрузка на кэш при каждой записи
```

### 🔄 Write-Behind кэширование

```
Write-Behind Pattern:
--------------------

Application
    ↓ write(key, value)
Cache Cluster
    ↓ (асинхронно, батчами)
Database

Асинхронная запись:
------------------
Cache Buffer: [key1, key2, key3, ...]
               ↓ (каждые 5 сек)
Database: batch_write([key1, key2, key3])
```

#### Реализация Write-Behind:

```python
import asyncio
from collections import defaultdict

class WriteBehindCache:
    def __init__(self, flush_interval=5):
        self.cache = {}
        self.dirty_keys = set()
        self.flush_interval = flush_interval
        self.running = True
        
    async def put(self, key, value):
        self.cache[key] = value
        self.dirty_keys.add(key)
        
    async def flush_worker(self):
        while self.running:
            if self.dirty_keys:
                # Batch write to database
                await self.flush_to_db(list(self.dirty_keys))
                self.dirty_keys.clear()
            await asyncio.sleep(self.flush_interval)
```

### 🔄 Read-Through кэширование

```
Read-Through Pattern:
--------------------

Application
    ↓ get(key)
Cache Cluster
    ↓ (если miss)
Database Loader
    ↓
Cache (автоматически)

Transparent Loading:
-------------------
def get(key):
    value = cache.get(key)
    if value is None:
        value = database.load(key)
        cache.put(key, value)
    return value
```

---

## 📚 Глава 4: Проблемы консистентности

### ⚖️ CAP теорема и кэширование

```
CAP Triangle:
            Consistency
                ▲
               / \
              /   \
             /     \
            /       \
Availability ◄─────► Partition
                   Tolerance

В распределенных кэшах нужно выбрать 2 из 3!
```

#### Примеры выбора:

```
CP (Consistency + Partition Tolerance):
--------------------------------------
Redis Cluster в строгом режиме
- Сильная консистентность
- Доступность может страдать при разделении

AP (Availability + Partition Tolerance):
---------------------------------------
Memcached кластер
- Высокая доступность
- Eventual consistency

CA (Consistency + Availability):
-------------------------------
Невозможно в распределенной среде!
```

### 🕰️ Eventual Consistency

```
Временная шкала Eventual Consistency:
------------------------------------

T0: Write(key=X, value=1) → Node A
T1: Node A → sync → Node B ❌ (network issue)
T2: Read(key=X) from Node B → returns old value
T3: Network recovered
T4: Node A → sync → Node B ✅
T5: Read(key=X) from Node B → returns new value

Период несогласованности: T1-T4
```

#### Стратегии минимизации несогласованности:

```
1. Read Repair:
--------------
Client reads from multiple nodes
If values differ → trigger repair

2. Anti-Entropy:
---------------
Background process синхронизирует узлы

3. Hinted Handoff:
-----------------
Временное хранение записей для недоступных узлов
```

### ⚔️ Conflict Resolution

#### Last Write Wins (LWW):

```
Timeline:
--------
T1: Node A writes value=100, timestamp=T1
T2: Node B writes value=200, timestamp=T2
T3: Sync occurs

Result: value=200 (T2 > T1)

Проблема: потеря данных при неточных часах!
```

#### Vector Clocks:

```
Vector Clock Example:
--------------------
Node A: [A:1, B:0, C:0] → value=100
Node B: [A:1, B:1, C:0] → value=200

При синхронизации:
- Node B's clock полностью > Node A's clock
- Node B wins: value=200
```

#### CRDT (Conflict-free Replicated Data Types):

```
G-Counter (Grow-only Counter):
-----------------------------
Node A: {A: 5, B: 3, C: 2} = total: 10
Node B: {A: 4, B: 4, C: 2} = total: 10

Merge: {A: max(5,4), B: max(3,4), C: max(2,2)}
     = {A: 5, B: 4, C: 2} = total: 11
```

---

## 🛠️ Практические задания

### Задание 1: Реализация консистентного хеширования

```python
# Дополните реализацию ConsistentHash
# Добавьте методы:
# - get_load_distribution() - возвращает распределение нагрузки
# - rebalance() - перебалансировка при добавлении узлов

class AdvancedConsistentHash(ConsistentHash):
    def get_load_distribution(self):
        # TODO: Реализовать анализ распределения
        pass
    
    def rebalance(self):
        # TODO: Реализовать перебалансировку
        pass
```

### Задание 2: Distributed Cache с автоматическим шардированием

```python
# Создайте класс DistributedCache который:
# 1. Автоматически шардирует данные между узлами
# 2. Поддерживает репликацию
# 3. Обрабатывает отказы узлов

class DistributedCache:
    def __init__(self, nodes, replication_factor=2):
        # TODO: Инициализация
        pass
    
    async def get(self, key):
        # TODO: Получение с учетом шардирования
        pass
    
    async def put(self, key, value):
        # TODO: Запись с репликацией
        pass
    
    async def handle_node_failure(self, failed_node):
        # TODO: Обработка отказа узла
        pass
```

### Задание 3: Система мониторинга

```python
# Создайте систему мониторинга которая отслеживает:
# - Hit/miss ratio по узлам
# - Распределение нагрузки
# - Задержки репликации
# - Доступность узлов

class CacheMonitor:
    def __init__(self, cache_cluster):
        # TODO: Инициализация мониторинга
        pass
    
    def get_metrics(self):
        # TODO: Сбор метрик
        pass
    
    def alert_on_issues(self):
        # TODO: Система алертов
        pass
```

### Задание 4: Нагрузочное тестирование

```python
# Создайте benchmark для тестирования:
# - Производительности при разном количестве узлов
# - Поведения при отказах
# - Эффективности консистентного хеширования

async def benchmark_distributed_cache():
    # TODO: Комплексный бенчмарк
    pass
```

---

## 📊 Ключевые метрики

### Performance Metrics:

```
Throughput:
----------
Requests per second (RPS)
Target: > 100,000 RPS

Latency:
-------
P50: < 1ms
P95: < 5ms  
P99: < 10ms

Consistency:
-----------
Max inconsistency window: < 100ms
Conflict resolution time: < 50ms
```

### Availability Metrics:

```
Uptime:
------
Target: 99.99% (52 minutes downtime/year)

MTTR (Mean Time To Recovery):
----------------------------
Target: < 30 seconds

Data Durability:
---------------
Target: 99.999999999% (11 9's)
```

---

## ✅ Результат блока

После изучения этого блока вы сможете:

- 🎯 Проектировать распределенные кэширующие системы
- ⚖️ Выбирать правильные trade-off между консистентностью и доступностью  
- 🔧 Реализовывать консистентное хеширование и шардирование
- 🛡️ Обеспечивать отказоустойчивость через репликацию
- 📊 Мониторить и оптимизировать производительность
- 🔄 Решать проблемы консистентности в распределенных системах

---

## 📚 Дополнительные ресурсы

### Статьи:
- "Consistent Hashing and Random Trees" - Karger et al.
- "Dynamo: Amazon's Highly Available Key-value Store"
- "The CAP Theorem" - Eric Brewer

### Инструменты:
- Redis Cluster
- Apache Ignite  
- Hazelcast
- Memcached

### Open Source проекты:
- Consistent hashing implementations
- Distributed cache benchmarks
- Monitoring tools для кэшей