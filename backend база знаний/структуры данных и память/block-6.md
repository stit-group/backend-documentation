# Блок 6: Практическое применение структур данных

**⏱️ Длительность:** 3-4 недели  
**🎯 Цель:** Применить знания в реальных проектах и системах

---

## 📋 Содержание блока

```
┌─────────────────────────────────────────────────────────────┐
│                    БЛОК 6: ПРАКТИЧЕСКОЕ ПРИМЕНЕНИЕ          │
├─────────────────────────────────────────────────────────────┤
│ 6.1 Структуры данных в базах данных                        │
│ 6.2 Сетевое программирование                               │
│ 6.3 Real-time системы                                      │
│ 6.4 Большие данные                                         │
└─────────────────────────────────────────────────────────────┘
```

---

## 🗄️ Глава 6.1: Структуры данных в базах данных

### Теоретические основы

#### B+ деревья в СУБД

```
                    B+ дерево (порядок 3)
                         ┌─────┐
                         │  50 │
                         └─────┘
                         /     \
                    ┌─────┐   ┌─────┐
                    │  25 │   │  75 │
                    └─────┘   └─────┘
                    /     \   /     \
              ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
              │10,20│ │30,40│ │60,70│ │80,90│
              └─────┘ └─────┘ └─────┘ └─────┘
                 ↓       ↓       ↓       ↓
              [Data]  [Data]  [Data]  [Data]
```

**Ключевые свойства B+ деревьев:**
- Все данные хранятся в листьях
- Внутренние узлы содержат только ключи для навигации
- Листья связаны в двусвязный список
- Высота дерева минимальна для заданного количества элементов

#### LSM-деревья для NoSQL

```
┌─────────────────────────────────────────────────────────────┐
│                    LSM-дерево архитектура                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Memory (MemTable)     Disk (SSTables)                      │
│ ┌─────────────┐      ┌─────────────────────────────────┐    │
│ │    Level 0  │      │ Level 1  │ Level 2  │ Level 3  │    │
│ │             │      │          │          │          │    │
│ │ ┌─────────┐ │ ──►  │ ┌─────┐  │ ┌─────┐  │ ┌─────┐  │    │
│ │ │ B-Tree  │ │      │ │SST-1│  │ │SST-3│  │ │SST-5│  │    │
│ │ │         │ │      │ │     │  │ │     │  │ │     │  │    │
│ │ └─────────┘ │      │ └─────┘  │ └─────┘  │ └─────┘  │    │
│ └─────────────┘      │          │          │          │    │
│                      │ ┌─────┐  │ ┌─────┐  │ ┌─────┐  │    │
│                      │ │SST-2│  │ │SST-4│  │ │SST-6│  │    │
│                      │ └─────┘  │ └─────┘  │ └─────┘  │    │
│                      └─────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

**Преимущества LSM-деревьев:**
- Высокая скорость записи
- Оптимизация для SSD
- Компрессия данных
- Поддержка версионности

### Практическое задание 6.1: Микро-СУБД

```cpp
class MicroDatabase {
private:
    BPlusTree<int, Record> primaryIndex;
    HashMap<std::string, int> secondaryIndex;
    WAL writeAheadLog;
    
public:
    void insert(const Record& record) {
        // 1. Записать в WAL
        writeAheadLog.append(INSERT, record);
        
        // 2. Обновить индексы
        primaryIndex.insert(record.id, record);
        secondaryIndex.insert(record.name, record.id);
        
        // 3. Периодически flush на диск
        if (writeAheadLog.size() > FLUSH_THRESHOLD) {
            flush();
        }
    }
    
    Record* select(int id) {
        return primaryIndex.search(id);
    }
    
    std::vector<Record> selectByName(const std::string& name) {
        auto ids = secondaryIndex.find(name);
        std::vector<Record> result;
        for (int id : ids) {
            if (auto record = primaryIndex.search(id)) {
                result.push_back(*record);
            }
        }
        return result;
    }
};
```

---

## 🌐 Глава 6.2: Сетевое программирование

### Буферизация сетевого ввода-вывода

```
┌─────────────────────────────────────────────────────────────┐
│                  Архитектура сетевых буферов                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Application Layer                                           │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │           Ring Buffer (Input)                           │ │
│ │ ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐       │ │
│ │ │     │     │     │     │     │     │     │     │       │ │
│ │ └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘       │ │
│ │      ↑                                         ↑        │ │
│ │   read_pos                                write_pos     │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
│ Socket Layer                                                │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │ │
│ │ │  Socket 1   │  │  Socket 2   │  │  Socket N   │     │ │
│ │ │             │  │             │  │             │     │ │
│ │ │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │     │ │
│ │ │ │ Buffer  │ │  │ │ Buffer  │ │  │ │ Buffer  │ │     │ │
│ │ │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │     │ │
│ │ └─────────────┘  └─────────────┘  └─────────────┘     │ │
│ └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Пулы соединений

```cpp
class ConnectionPool {
private:
    std::queue<std::unique_ptr<Connection>> available;
    std::unordered_set<Connection*> inUse;
    std::mutex poolMutex;
    std::condition_variable cv;
    size_t maxSize;
    
public:
    class ConnectionGuard {
    private:
        ConnectionPool* pool;
        std::unique_ptr<Connection> conn;
        
    public:
        ConnectionGuard(ConnectionPool* p, std::unique_ptr<Connection> c) 
            : pool(p), conn(std::move(c)) {}
        
        ~ConnectionGuard() {
            pool->returnConnection(std::move(conn));
        }
        
        Connection* operator->() { return conn.get(); }
    };
    
    ConnectionGuard getConnection() {
        std::unique_lock<std::mutex> lock(poolMutex);
        
        // Ждем доступное соединение
        cv.wait(lock, [this] { return !available.empty(); });
        
        auto conn = std::move(available.front());
        available.pop();
        
        Connection* rawPtr = conn.get();
        inUse.insert(rawPtr);
        
        return ConnectionGuard(this, std::move(conn));
    }
    
private:
    void returnConnection(std::unique_ptr<Connection> conn) {
        std::lock_guard<std::mutex> lock(poolMutex);
        
        Connection* rawPtr = conn.get();
        inUse.erase(rawPtr);
        available.push(std::move(conn));
        
        cv.notify_one();
    }
};
```

### Асинхронное программирование

```
┌─────────────────────────────────────────────────────────────┐
│                  Event Loop Architecture                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │                   Event Queue                           │ │
│ │ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐       │ │
│ │ │ I/O │ │Timer│ │ I/O │ │ I/O │ │Timer│ │ I/O │  ...  │ │
│ │ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘       │ │
│ └─────────────────────────────────────────────────────────┘ │
│                            ↓                                │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │                  Event Loop                             │ │
│ │                                                         │ │
│ │ while (true) {                                          │ │
│ │     event = eventQueue.pop();                          │ │
│ │     switch (event.type) {                              │ │
│ │         case IO_READ: handleRead(event); break;        │ │
│ │         case IO_WRITE: handleWrite(event); break;      │ │
│ │         case TIMER: handleTimer(event); break;         │ │
│ │     }                                                   │ │
│ │ }                                                       │ │
│ └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## ⚡ Глава 6.3: Real-time системы

### Структуры данных для игр

```
┌─────────────────────────────────────────────────────────────┐
│                     Game Engine Memory                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Stack Allocator (Frame-based)                              │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ Frame N   │ Frame N+1 │ Frame N+2 │        Free       │ │
│ │ ┌─────┐   │ ┌─────┐   │ ┌─────┐   │                   │ │
│ │ │Temp │   │ │Temp │   │ │Temp │   │                   │ │
│ │ │Data │   │ │Data │   │ │Data │   │                   │ │
│ │ └─────┘   │ └─────┘   │ └─────┘   │                   │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
│ Object Pool (Entities)                                      │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐       │ │
│ │ │ Obj │ │ Obj │ │ Obj │ │ Obj │ │ Obj │ │ Obj │  ...  │ │
│ │ │  1  │ │  2  │ │  3  │ │  4  │ │  5  │ │  6  │       │ │
│ │ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘       │ │
│ │   ↑       ↑       ↑                                    │ │
│ │ Active  Active  Active                                 │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
│ Spatial Hash (Collision Detection)                         │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │   0,0   │   1,0   │   2,0   │   3,0   │   4,0   │     │ │
│ │ ┌─────┐ │ ┌─────┐ │ ┌─────┐ │ ┌─────┐ │ ┌─────┐ │     │ │
│ │ │ ●   │ │ │  ●  │ │ │     │ │ │  ●  │ │ │     │ │     │ │
│ │ └─────┘ │ └─────┘ │ └─────┘ │ └─────┘ │ └─────┘ │     │ │
│ └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Embedded системы и ограничения памяти

```cpp
// Компактная структура для микроконтроллера
struct CompactSensor {
    uint16_t temperature : 12;  // 0-4095 (0.1°C precision)
    uint16_t humidity : 10;     // 0-1023 (0.1% precision)
    uint16_t pressure : 14;     // 0-16383 (0.01 hPa precision)
    uint8_t battery : 8;        // 0-255 (battery level)
    uint8_t status : 4;         // Status flags
    uint8_t reserved : 4;       // Reserved for future use
} __attribute__((packed));

// Circular buffer для сенсорных данных
template<typename T, size_t SIZE>
class CircularBuffer {
private:
    T buffer[SIZE];
    volatile size_t head = 0;
    volatile size_t tail = 0;
    volatile bool full = false;
    
public:
    void push(const T& item) {
        buffer[head] = item;
        
        if (full) {
            tail = (tail + 1) % SIZE;
        }
        
        head = (head + 1) % SIZE;
        full = head == tail;
    }
    
    bool pop(T& item) {
        if (empty()) return false;
        
        item = buffer[tail];
        full = false;
        tail = (tail + 1) % SIZE;
        
        return true;
    }
    
    bool empty() const {
        return (!full && (head == tail));
    }
};
```

---

## 📊 Глава 6.4: Большие данные

### Map-Reduce парадигма

```
┌─────────────────────────────────────────────────────────────┐
│                    MapReduce Pipeline                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Input Data                                                  │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ File1.txt │ File2.txt │ File3.txt │ File4.txt │ ...     │ │
│ └─────────────────────────────────────────────────────────┘ │
│                            ↓                                │
│ Map Phase                                                   │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ Mapper1   │ Mapper2   │ Mapper3   │ Mapper4   │ ...     │ │
│ │ ┌─────┐   │ ┌─────┐   │ ┌─────┐   │ ┌─────┐   │         │ │
│ │ │K1,V1│   │ │K2,V2│   │ │K1,V3│   │ │K3,V4│   │         │ │
│ │ │K2,V5│   │ │K3,V6│   │ │K2,V7│   │ │K1,V8│   │         │ │
│ │ └─────┘   │ └─────┘   │ └─────┘   │ └─────┘   │         │ │
│ └─────────────────────────────────────────────────────────┘ │
│                            ↓                                │
│ Shuffle & Sort                                              │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ K1: [V1, V3, V8]  │ K2: [V5, V7]  │ K3: [V6, V4]      │ │
│ └─────────────────────────────────────────────────────────┘ │
│                            ↓                                │
│ Reduce Phase                                                │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ Reducer1      │ Reducer2      │ Reducer3      │ ...     │ │
│ │ K1 → Result1  │ K2 → Result2  │ K3 → Result3  │         │ │
│ └─────────────────────────────────────────────────────────┘ │
│                            ↓                                │
│ Output                                                      │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ part-00000    │ part-00001    │ part-00002    │ ...     │ │
│ └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Стриминговые алгоритмы

```cpp
// Count-Min Sketch для оценки частоты элементов
class CountMinSketch {
private:
    static constexpr int HASH_FUNCTIONS = 4;
    static constexpr int BUCKETS = 1000;
    
    int table[HASH_FUNCTIONS][BUCKETS];
    std::vector<std::function<int(const std::string&)>> hashFunctions;
    
public:
    CountMinSketch() {
        // Инициализация таблицы нулями
        memset(table, 0, sizeof(table));
        
        // Создание хеш-функций
        for (int i = 0; i < HASH_FUNCTIONS; i++) {
            hashFunctions.push_back([i](const std::string& s) {
                return std::hash<std::string>{}(s + std::to_string(i)) % BUCKETS;
            });
        }
    }
    
    void add(const std::string& item) {
        for (int i = 0; i < HASH_FUNCTIONS; i++) {
            int bucket = hashFunctions[i](item);
            table[i][bucket]++;
        }
    }
    
    int estimate(const std::string& item) {
        int minCount = INT_MAX;
        for (int i = 0; i < HASH_FUNCTIONS; i++) {
            int bucket = hashFunctions[i](item);
            minCount = std::min(minCount, table[i][bucket]);
        }
        return minCount;
    }
};
```

### Распределенные хеш-таблицы

```
┌─────────────────────────────────────────────────────────────┐
│                Consistent Hashing Ring                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                            Node A                           │
│                              ●                             │
│                          hash(A)                           │
│                         /         \                        │
│                        /           \                       │
│                   Key1 ●             ● Key2                │
│                      /                 \                   │
│                     /                   \                  │
│                    /                     \                 │
│               Node D                       Node B           │
│                  ●                         ●               │
│              hash(D)                   hash(B)             │
│                   \                     /                   │
│                    \                   /                    │
│                     \                 /                     │
│                   Key4 ●             ● Key3                │
│                         \           /                       │
│                          \         /                        │
│                            Node C                           │
│                              ●                             │
│                          hash(C)                           │
│                                                             │
│ При добавлении нового узла:                                 │
│ - Только часть ключей перемещается                         │
│ - Остальные узлы не затрагиваются                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎯 Практические задания блока 6

### 📝 Задание 6.1: Микро-СУБД с собственным движком хранения

**Цель:** Создать упрощенную базу данных с B+ деревьями для индексации

**Требования:**
- Поддержка CRUD операций
- Первичные и вторичные индексы
- Транзакции и Write-Ahead Logging
- Оптимизация для дисковых операций

**Ключевые компоненты:**
```cpp
class StorageEngine {
    PageManager pageManager;
    BPlusTree primaryIndex;
    std::unordered_map<std::string, SecondaryIndex> secondaryIndexes;
    TransactionManager transactionManager;
    WAL writeAheadLog;
};
```

### 🎮 Задание 6.2: Multiplayer игра с синхронизацией состояния

**Цель:** Реализовать сетевую игру с эффективной синхронизацией

**Требования:**
- Spatial hashing для коллизий
- Delta compression для сетевых пакетов
- Lag compensation и prediction
- Efficient memory management

**Архитектура:**
```
Client ←→ [Ring Buffer] ←→ Network ←→ [Connection Pool] ←→ Server
   ↑                                                         ↓
[Entity Pool]                                        [Spatial Hash]
   ↑                                                         ↓
[Component System]                                  [Game State]
```

### 🌐 Задание 6.3: Распределенный кэш

**Цель:** Создать масштабируемый кэш с consistent hashing

**Требования:**
- Consistent hashing для распределения
- Репликация для надежности
- LRU eviction policy
- Мониторинг и метрики

**Компоненты:**
```cpp
class DistributedCache {
    ConsistentHashRing hashRing;
    std::unordered_map<std::string, CacheNode> nodes;
    ReplicationManager replicationManager;
    MetricsCollector metrics;
};
```

---

## 📚 Дополнительные материалы

### Рекомендуемая литература:
- "Designing Data-Intensive Applications" - Martin Kleppmann
- "High Performance MySQL" - Baron Schwartz
- "Game Programming Patterns" - Robert Nystrom
- "Streaming Systems" - Tyler Akidau

### Полезные ресурсы:
- Papers We Love (distributed systems)
- High Scalability blog
- The Architecture of Open Source Applications
- Database internals documentation

---

## ✅ Критерии оценки

### Теоретические знания (40%):
- ✅ Понимание архитектуры СУБД
- ✅ Знание сетевых протоколов и паттернов
- ✅ Принципы real-time систем
- ✅ Алгоритмы для больших данных

### Практические навыки (50%):
- ✅ Реализация storage engine
- ✅ Оптимизация сетевого кода
- ✅ Профилирование и debugging
- ✅ Масштабирование систем

### Исследовательская работа (10%):
- ✅ Анализ существующих решений
- ✅ Бенчмарки и сравнения
- ✅ Предложения по улучшению

---

## 🚀 Результаты блока

После завершения блока 6 вы будете:

**🎯 Уметь:**
- Проектировать высокопроизводительные системы хранения данных
- Оптимизировать сетевое взаимодействие и управление памятью
- Создавать real-time системы с жесткими ограничениями
- Работать с большими данными и распределенными системами

**💼 Иметь портфолио:**
- Функциональную СУБД с собственным движком
- Multiplayer игру с сетевой синхронизацией
- Масштабируемый распределенный кэш

**🔄 Быть готовым к:**
- Архитектурным решениям на уровне системы
- Оптимизации производительности критических компонентов
- Работе с высоконагруженными системами
- Переходу к экспертному уровню (Блок 7)

---

*Следующий блок: **Блок 7 - Экспертный уровень** 🎓*