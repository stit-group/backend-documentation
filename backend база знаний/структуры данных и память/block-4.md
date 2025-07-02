# Блок 4: Оптимизация памяти и продвинутые техники

**⏱️ Длительность:** 3-4 недели  
**🎯 Цель:** Научиться оптимизировать использование памяти и повышать производительность

---

## Глава 4.1: Кэширование и локальность данных

### 📚 Теоретические основы

#### Иерархия памяти компьютера

```
┌─────────────────────────────────────────────────────────────┐
│                    ИЕРАРХИЯ ПАМЯТИ                          │
├─────────────────────────────────────────────────────────────┤
│ Регистры CPU      │ ~1 цикл     │ ~1 KB    │ Самая быстрая  │
├─────────────────────────────────────────────────────────────┤
│ L1 Cache          │ ~4 цикла    │ ~64 KB   │ ↑ Скорость     │
├─────────────────────────────────────────────────────────────┤
│ L2 Cache          │ ~10 циклов  │ ~1 MB    │ ↑              │
├─────────────────────────────────────────────────────────────┤
│ L3 Cache          │ ~40 циклов  │ ~30 MB   │ ↑              │
├─────────────────────────────────────────────────────────────┤
│ Оперативная память│ ~200 циклов │ ~16 GB   │ ↓              │
├─────────────────────────────────────────────────────────────┤
│ SSD               │ ~25,000     │ ~1 TB    │ ↓              │
├─────────────────────────────────────────────────────────────┤
│ HDD               │ ~10,000,000 │ ~10 TB   │ ↓ Медленнее    │
└─────────────────────────────────────────────────────────────┘
```

#### Принципы кэширования

**Spatial Locality (Пространственная локальность):**
```
Обращение к адресу X → высока вероятность обращения к X+1, X+2, ...

Память: [A][B][C][D][E][F][G][H]
         ↑
    Обратились к A
    
Кэш загружает: [A][B][C][D] - целую cache line (обычно 64 байта)
```

**Temporal Locality (Временная локальность):**
```
Обращение к адресу X → высока вероятность повторного обращения к X

Время:  t1    t2    t3    t4    t5
Данные: [A] → [B] → [A] → [C] → [A]
              ↑           ↑           ↑
          Cache hit   Cache hit   Cache hit
```

### 🔧 Cache-friendly программирование

#### Пример 1: Обход массива

**❌ Плохо (cache miss):**
```cpp
// Обход по столбцам - много cache miss'ов
int matrix[1000][1000];
int sum = 0;

for (int col = 0; col < 1000; col++) {
    for (int row = 0; row < 1000; row++) {
        sum += matrix[row][col];  // Прыжки по памяти!
    }
}
```

**✅ Хорошо (cache hit):**
```cpp
// Обход по строкам - использует spatial locality
int matrix[1000][1000];
int sum = 0;

for (int row = 0; row < 1000; row++) {
    for (int col = 0; col < 1000; col++) {
        sum += matrix[row][col];  // Последовательный доступ
    }
}
```

**Визуализация доступа к памяти:**
```
Плохой обход (по столбцам):
┌───┬───┬───┬───┐
│ 1 │ 4 │ 7 │...│ ← Cache line 1
├───┼───┼───┼───┤
│ 2 │ 5 │ 8 │...│ ← Cache line 2
├───┼───┼───┼───┤
│ 3 │ 6 │ 9 │...│ ← Cache line 3
└───┴───┴───┴───┘
Каждое обращение = новая cache line!

Хороший обход (по строкам):
┌───┬───┬───┬───┐
│ 1 │ 2 │ 3 │ 4 │ ← Cache line 1
├───┼───┼───┼───┤
│ 5 │ 6 │ 7 │ 8 │ ← Cache line 2
├───┼───┼───┼───┤
│ 9 │10 │11 │12 │ ← Cache line 3
└───┴───┴───┴───┘
Одна cache line = 4 элемента!
```

#### Пример 2: Structure of Arrays vs Array of Structures

**❌ Array of Structures (AoS):**
```cpp
struct Particle {
    float x, y, z;     // position
    float vx, vy, vz;  // velocity
    float mass;
    int id;
};

Particle particles[1000];

// Обновляем только позиции
for (int i = 0; i < 1000; i++) {
    particles[i].x += particles[i].vx;
    particles[i].y += particles[i].vy;
    particles[i].z += particles[i].vz;
}
```

**Память выглядит так:**
```
[x,y,z,vx,vy,vz,mass,id][x,y,z,vx,vy,vz,mass,id][x,y,z,vx,vy,vz,mass,id]...
 └─────── 32 байта ──────┘
 
Cache line (64 байта) загружает только 2 структуры,
но используем только 24 байта из 64!
```

**✅ Structure of Arrays (SoA):**
```cpp
struct ParticleSystem {
    float x[1000], y[1000], z[1000];        // positions
    float vx[1000], vy[1000], vz[1000];     // velocities
    float mass[1000];
    int id[1000];
};

ParticleSystem particles;

// Обновляем только позиции
for (int i = 0; i < 1000; i++) {
    particles.x[i] += particles.vx[i];
    particles.y[i] += particles.vy[i];
    particles.z[i] += particles.vz[i];
}
```

**Память выглядит так:**
```
X массив: [x0,x1,x2,x3,x4,x5,x6,x7,x8,x9,x10,x11,x12,x13,x14,x15]...
           └────────── Cache line (64 байта) ──────────┘
           
Используем ВСЕ 64 байта cache line!
16 float'ов за раз вместо 6!
```

### 📊 Измерение производительности кэша

#### Инструмент для измерения cache miss'ов:

```cpp
#include <chrono>
#include <iostream>
#include <vector>

void benchmark_cache_locality() {
    const int SIZE = 1024 * 1024;  // 1M элементов
    std::vector<int> data(SIZE);
    
    // Инициализация
    for (int i = 0; i < SIZE; i++) {
        data[i] = i;
    }
    
    auto start = std::chrono::high_resolution_clock::now();
    
    // Последовательный доступ (cache-friendly)
    long sum1 = 0;
    for (int i = 0; i < SIZE; i++) {
        sum1 += data[i];
    }
    
    auto mid = std::chrono::high_resolution_clock::now();
    
    // Случайный доступ (cache-unfriendly)
    long sum2 = 0;
    for (int i = 0; i < SIZE; i++) {
        sum2 += data[(i * 16777213) % SIZE];  // Псевдослучайный индекс
    }
    
    auto end = std::chrono::high_resolution_clock::now();
    
    auto seq_time = std::chrono::duration_cast<std::chrono::microseconds>(mid - start);
    auto rand_time = std::chrono::duration_cast<std::chrono::microseconds>(end - mid);
    
    std::cout << "Последовательный доступ: " << seq_time.count() << " μs\n";
    std::cout << "Случайный доступ: " << rand_time.count() << " μs\n";
    std::cout << "Разница: " << (float)rand_time.count() / seq_time.count() << "x\n";
}
```

---

## Глава 4.2: Пулы памяти и custom allocators

### 📚 Проблемы стандартных аллокаторов

#### Фрагментация памяти

```
Начальное состояние кучи:
┌─────────────────────────────────────────────┐
│                СВОБОДНО                      │ 1MB
└─────────────────────────────────────────────┘

После множественных malloc/free:
┌──┬────┬──┬─────┬──┬──────┬──┬────────┬──┬───┐
│F │ U  │F │ U   │F │ U    │F │   U    │F │ U │
└──┴────┴──┴─────┴──┴──────┴──┴────────┴──┴───┘
F = Free (фрагмент), U = Used

Проблема: Много маленьких фрагментов, 
нельзя выделить один большой блок!
```

#### Object Pool (Пул объектов)

**Идея:** Заранее выделить блок памяти и управлять им самостоятельно

```cpp
template<typename T, size_t PoolSize>
class ObjectPool {
private:
    // Хранилище для объектов
    alignas(T) char storage[sizeof(T) * PoolSize];
    
    // Стек свободных объектов
    T* free_objects[PoolSize];
    size_t free_count;
    
public:
    ObjectPool() : free_count(PoolSize) {
        // Инициализируем стек свободных объектов
        for (size_t i = 0; i < PoolSize; i++) {
            free_objects[i] = reinterpret_cast<T*>(storage + i * sizeof(T));
        }
    }
    
    T* allocate() {
        if (free_count == 0) {
            return nullptr;  // Пул исчерпан
        }
        return free_objects[--free_count];
    }
    
    void deallocate(T* ptr) {
        if (free_count < PoolSize) {
            free_objects[free_count++] = ptr;
        }
    }
    
    template<typename... Args>
    T* construct(Args&&... args) {
        T* ptr = allocate();
        if (ptr) {
            new(ptr) T(std::forward<Args>(args)...);
        }
        return ptr;
    }
    
    void destroy(T* ptr) {
        if (ptr) {
            ptr->~T();
            deallocate(ptr);
        }
    }
};
```

**Визуализация работы Object Pool:**

```
Инициализация:
Storage:  [Obj1][Obj2][Obj3][Obj4][Obj5]
FreeList: [  5  ][  4  ][  3  ][  2  ][  1  ] ← free_count = 5
             ↑
           Top

После allocate():
Storage:  [Obj1][Obj2][Obj3][Obj4][Obj5]
           Used   Free   Free   Free   Free
FreeList: [  -  ][  4  ][  3  ][  2  ][  1  ] ← free_count = 4
                    ↑
                  Top

После deallocate(Obj1):
Storage:  [Obj1][Obj2][Obj3][Obj4][Obj5]
           Free   Free   Free   Free   Free
FreeList: [  1  ][  4  ][  3  ][  2  ][  1  ] ← free_count = 5
             ↑
           Top
```

#### Memory Pool (Пул памяти)

```cpp
class MemoryPool {
private:
    struct Block {
        Block* next;
    };
    
    char* memory;
    Block* free_head;
    size_t block_size;
    size_t block_count;
    
public:
    MemoryPool(size_t block_size, size_t block_count) 
        : block_size(std::max(block_size, sizeof(Block)))
        , block_count(block_count) {
        
        // Выделяем память для всех блоков
        memory = new char[this->block_size * block_count];
        
        // Создаем связный список свободных блоков
        free_head = reinterpret_cast<Block*>(memory);
        Block* current = free_head;
        
        for (size_t i = 0; i < block_count - 1; i++) {
            current->next = reinterpret_cast<Block*>(
                memory + (i + 1) * this->block_size
            );
            current = current->next;
        }
        current->next = nullptr;
    }
    
    void* allocate() {
        if (!free_head) {
            return nullptr;  // Нет свободных блоков
        }
        
        Block* allocated = free_head;
        free_head = free_head->next;
        return allocated;
    }
    
    void deallocate(void* ptr) {
        if (!ptr) return;
        
        Block* block = static_cast<Block*>(ptr);
        block->next = free_head;
        free_head = block;
    }
    
    ~MemoryPool() {
        delete[] memory;
    }
};
```

**Визуализация Memory Pool:**

```
Инициализация (block_size=16, count=5):

Memory: [Block0][Block1][Block2][Block3][Block4]
         │      │      │      │      │
         ▼      ▼      ▼      ▼      ▼
        Next   Next   Next   Next   NULL

free_head → Block0 → Block1 → Block2 → Block3 → Block4 → NULL

После allocate():
free_head → Block1 → Block2 → Block3 → Block4 → NULL
            (Block0 выдан пользователю)

После deallocate(Block0):
free_head → Block0 → Block1 → Block2 → Block3 → Block4 → NULL
            (Block0 возвращен в начало списка)
```

#### Stack Allocator (Стековый аллокатор)

```cpp
class StackAllocator {
private:
    char* memory;
    size_t size;
    size_t offset;
    
public:
    StackAllocator(size_t size) : size(size), offset(0) {
        memory = new char[size];
    }
    
    void* allocate(size_t bytes) {
        // Выравнивание по границе машинного слова
        size_t aligned_bytes = (bytes + 7) & ~7;
        
        if (offset + aligned_bytes > size) {
            return nullptr;  // Недостаточно места
        }
        
        void* result = memory + offset;
        offset += aligned_bytes;
        return result;
    }
    
    // Откат к предыдущему состоянию
    void reset(size_t new_offset = 0) {
        offset = new_offset;
    }
    
    // Получить текущее состояние (для отката)
    size_t get_marker() const {
        return offset;
    }
    
    ~StackAllocator() {
        delete[] memory;
    }
};
```

**Визуализация Stack Allocator:**

```
Начальное состояние:
┌─────────────────────────────────────────┐
│                СВОБОДНО                  │
└─────────────────────────────────────────┘
offset = 0

После allocate(100):
┌──────────┬──────────────────────────────┐
│ ЗАНЯТО   │           СВОБОДНО            │
│ 100 байт │                              │
└──────────┴──────────────────────────────┘
offset = 100

После allocate(200):
┌──────────┬─────────────┬─────────────────┐
│ ЗАНЯТО   │   ЗАНЯТО    │    СВОБОДНО     │
│ 100 байт │  200 байт   │                │
└──────────┴─────────────┴─────────────────┘
offset = 300

После reset(100):
┌──────────┬──────────────────────────────┐
│ ЗАНЯТО   │           СВОБОДНО            │
│ 100 байт │      (200 байт освобождено)  │
└──────────┴──────────────────────────────┘
offset = 100
```

### 🎯 Практическое задание

**Задача:** Реализовать игровую систему частиц с оптимизированным управлением памятью

```cpp
class ParticleSystem {
private:
    struct Particle {
        float x, y, z;           // позиция
        float vx, vy, vz;        // скорость
        float life;              // время жизни
        float size;              // размер
    };
    
    ObjectPool<Particle, 10000> particle_pool;
    std::vector<Particle*> active_particles;
    
public:
    void spawn_particle(float x, float y, float z) {
        Particle* p = particle_pool.construct();
        if (p) {
            p->x = x; p->y = y; p->z = z;
            p->vx = (rand() % 200 - 100) / 100.0f;
            p->vy = (rand() % 200 - 100) / 100.0f;
            p->vz = (rand() % 200 - 100) / 100.0f;
            p->life = 5.0f;
            p->size = 1.0f;
            
            active_particles.push_back(p);
        }
    }
    
    void update(float dt) {
        for (auto it = active_particles.begin(); it != active_particles.end();) {
            Particle* p = *it;
            
            // Обновляем физику
            p->x += p->vx * dt;
            p->y += p->vy * dt;
            p->z += p->vz * dt;
            p->life -= dt;
            
            if (p->life <= 0) {
                // Частица умерла - возвращаем в пул
                particle_pool.destroy(p);
                it = active_particles.erase(it);
            } else {
                ++it;
            }
        }
    }
};
```

---

## Глава 4.3: Компрессия данных и битовые операции

### 📚 Bit arrays и bit fields

#### Bit Array (Битовый массив)

```cpp
class BitArray {
private:
    std::vector<uint64_t> data;
    size_t bit_count;
    
    static const size_t BITS_PER_WORD = 64;
    
public:
    BitArray(size_t bits) : bit_count(bits) {
        size_t words = (bits + BITS_PER_WORD - 1) / BITS_PER_WORD;
        data.resize(words, 0);
    }
    
    void set(size_t index) {
        if (index >= bit_count) return;
        
        size_t word_index = index / BITS_PER_WORD;
        size_t bit_index = index % BITS_PER_WORD;
        
        data[word_index] |= (1ULL << bit_index);
    }
    
    void clear(size_t index) {
        if (index >= bit_count) return;
        
        size_t word_index = index / BITS_PER_WORD;
        size_t bit_index = index % BITS_PER_WORD;
        
        data[word_index] &= ~(1ULL << bit_index);
    }
    
    bool test(size_t index) const {
        if (index >= bit_count) return false;
        
        size_t word_index = index / BITS_PER_WORD;
        size_t bit_index = index % BITS_PER_WORD;
        
        return (data[word_index] & (1ULL << bit_index)) != 0;
    }
};
```

**Визуализация BitArray:**

```
Хранение 100 бит в BitArray:

Word 0 (биты 0-63):
┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┐
│0│1│0│1│1│0│1│0│0│1│1│1│0│0│1│0│1│1│0│1│0│0│1│1│1│0│1│0│1│1│0│1│
└─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┘
 0 1 2 3 4 5 6 7 8 9...                                     62 63

Word 1 (биты 64-99):
┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┐
│1│0│1│1│0│1│0│0│1│1│1│0│0│1│0│1│1│0│1│0│0│1│1│1│0│1│0│1│ │ │ │ │
└─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┘
64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91

set(67): Word 1, bit 3
data[1] |= (1ULL << 3)  // Устанавливаем 3-й бит в 1
```

#### Битовые поля для компактного хранения

```cpp
// Обычная структура - 16 байт
struct GameObjectNormal {
    bool is_visible;      // 1 байт (+ 3 байта padding)
    bool is_solid;        // 1 байт (+ 3 байта padding)
    bool is_animated;     // 1 байт (+ 3 байта padding)
    uint8_t health;       // 1 байт (+ 3 байта padding)
};

// Оптимизированная структура - 4 байта
struct GameObjectCompact {
    uint32_t is_visible : 1;    // 1 бит
    uint32_t is_solid : 1;      // 1 бит
    uint32_t is_animated : 1;   // 1 бит
    uint32_t health : 8;        // 8 бит (0-255)
    uint32_t object_type : 4;   // 4 бита (16 типов)
    uint32_t level : 4;         // 4 бита (16 уровней)
    uint32_t unused : 12;       // 12 бит резерв
};
```

**Визуализация битовых полей:**

```
GameObjectCompact (32 бита = 4 байта):

┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┐
│ unused (12 бит)       │level│object_type│  health (8)   │A│S│V│
│                       │ (4) │    (4)    │               │n│o│i│
│                       │     │           │               │i│l│s│
└─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┘
31                    20  19  16  15      8   7    4  3  2  1  0

Экономия: 16 байт → 4 байта = 75% экономии памяти!
```

### 🔧 SIMD оптимизации

#### Векторизация операций

```cpp
#include <immintrin.h>  // SSE/AVX

// Обычное сложение массивов
void add_arrays_scalar(const float* a, const float* b, float* result, size_t count) {
    for (size_t i = 0; i < count; i++) {
        result[i] = a[i] + b[i];
    }
}

// SIMD версия (обрабатывает 8 float за раз)
void add_arrays_simd(const float* a, const float* b, float* result, size_t count) {
    const size_t simd_count = count & ~7;  // Кратно 8
    
    for (size_t i = 0; i < simd_count; i += 8) {
        __m256 va = _mm256_load_ps(&a[i]);     // Загружаем 8 float
        __m256 vb = _mm256_load_ps(&b[i]);     // Загружаем 8 float
        __m256 vresult = _mm256_add_ps(va, vb); // Складываем все 8 за раз
        _mm256_store_ps(&result[i], vresult);   // Сохраняем результат
    }
    
    // Обрабатываем оставшиеся элементы
    for (size_t i = simd_count; i < count; i++) {
        result[i] = a[i] + b[i];
    }
}
```

**Визуализация SIMD операций:**

```
Скалярная обработка:
a[0] + b[0] = result[0]  ← 1 операция
a[1] + b[1] = result[1]  ← 1 операция  
a[2] + b[2] = result[2]  ← 1 операция
...
Итого: N операций

SIMD обработка (AVX):
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│a[0] │a[1] │a[2] │a[3] │a[4] │a[5] │a[6] │a[7] │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
       +       +       +       +       +       +       +       +
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│b[0] │b[1] │b[2] │b[3] │b[4] │b[5] │b[6] │b[7] │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
       =       =       =       =       =       =       =       =
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│r[0] │r[1] │r[2] │r[3] │r[4] │r[5] │r[6] │r[7] │ ← 1 операция!
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘

Итого: N/8 операций (ускорение в ~8 раз)
```

---

## Глава 4.4: Профилирование и отладка

### 📊 Инструменты профилирования

#### Простой профайлер времени

```cpp
class Timer {
private:
    std::chrono::high_resolution_clock::time_point start_time;
    std::string name;
    
public:
    Timer(const std::string& name) : name(name) {
        start_time = std::chrono::high_resolution_clock::now();
    }
    
    ~Timer() {
        auto end_time = std::chrono::high_resolution_clock::now();
        auto duration = std::chrono::duration_cast<std::chrono::microseconds>(
            end_time - start_time
        );
        std::cout << name << ": " << duration.count() << " μs\n";
    }
};

#define PROFILE_SCOPE(name) Timer timer(name)
#define PROFILE_FUNCTION() Timer timer(__FUNCTION__)

// Использование:
void expensive_function() {
    PROFILE_FUNCTION();
    
    {
        PROFILE_SCOPE("Memory allocation");
        std::vector<int> big_vector(1000000);
    }
    
    {
        PROFILE_SCOPE("Computation");
        for (int i = 0; i < 1000000; i++) {
            // некоторые вычисления
        }
    }
}
```

#### Профайлер памяти

```cpp
class MemoryProfiler {
private:
    struct AllocationInfo {
        size_t size;
        std::string file;
        int line;
        std::chrono::time_point<std::chrono::steady_clock> timestamp;
    };
    
    std::unordered_map<void*, AllocationInfo> allocations;
    std::mutex mutex;
    size_t total_allocated = 0;
    size_t peak_allocated = 0;
    
public:
    static MemoryProfiler& instance() {
        static MemoryProfiler instance;
        return instance;
    }
    
    void record_allocation(void* ptr, size_t size, const char* file, int line) {
        std::lock_guard<std::mutex> lock(mutex);
        
        allocations[ptr] = {
            size, 
            file, 
            line, 
            std::chrono::steady_clock::now()
        };
        
        total_allocated += size;
        peak_allocated = std::max(peak_allocated, total_allocated);
    }
    
    void record_deallocation(void* ptr) {
        std::lock_guard<std::mutex> lock(mutex);
        
        auto it = allocations.find(ptr);
        if (it != allocations.end()) {
            total_allocated -= it->second.size;
            allocations.erase(it);
        }
    }
    
    void report_leaks() {
        std::lock_guard<std::mutex> lock(mutex);
        
        if (!allocations.empty()) {
            std::cout << "=== MEMORY LEAKS DETECTED ===\n";
            for (const auto& [ptr, info] : allocations) {
                std::cout << "Leak: " << info.size << " bytes at " 
                         << info.file << ":" << info.line << "\n";
            }
        }
        
        std::cout << "Peak memory usage: " << peak_allocated << " bytes\n";
        std::cout << "Current memory usage: " << total_allocated << " bytes\n";
    }
};

// Макросы для отслеживания
#define TRACKED_NEW(size) \
    ([&](size_t s) { \
        void* ptr = malloc(s); \
        MemoryProfiler::instance().record_allocation(ptr, s, __FILE__, __LINE__); \
        return ptr; \
    })(size)

#define TRACKED_DELETE(ptr) \
    do { \
        MemoryProfiler::instance().record_deallocation(ptr); \
        free(ptr); \
    } while(0)
```

### 🔍 Поиск узких мест

#### CPU профилирование с помощью sampling

```cpp
class SamplingProfiler {
private:
    struct Sample {
        std::string function_name;
        std::chrono::time_point<std::chrono::steady_clock> timestamp;
    };
    
    std::vector<Sample> samples;
    std::atomic<bool> profiling{false};
    std::thread sampling_thread;
    
    void sampling_loop() {
        while (profiling) {
            // Получаем стек вызовов (упрощенная версия)
            samples.push_back({get_current_function(), std::chrono::steady_clock::now()});
            std::this_thread::sleep_for(std::chrono::milliseconds(1));
        }
    }
    
    std::string get_current_function() {
        // В реальности нужно получать стек вызовов
        // Здесь упрощенная версия
        return "unknown_function";
    }
    
public:
    void start_profiling() {
        profiling = true;
        sampling_thread = std::thread(&SamplingProfiler::sampling_loop, this);
    }
    
    void stop_profiling() {
        profiling = false;
        if (sampling_thread.joinable()) {
            sampling_thread.join();
        }
    }
    
    void generate_report() {
        std::unordered_map<std::string, int> function_counts;
        
        for (const auto& sample : samples) {
            function_counts[sample.function_name]++;
        }
        
        std::vector<std::pair<std::string, int>> sorted_functions(
            function_counts.begin(), function_counts.end()
        );
        
        std::sort(sorted_functions.begin(), sorted_functions.end(),
                 [](const auto& a, const auto& b) { return a.second > b.second; });
        
        std::cout << "=== CPU PROFILING REPORT ===\n";
        for (const auto& [func, count] : sorted_functions) {
            float percentage = (float)count / samples.size() * 100.0f;
            std::cout << func << ": " << percentage << "% (" << count << " samples)\n";
        }
    }
};
```

### 📈 Визуализация производительности

```
Пример отчета профилировщика:

=== PERFORMANCE REPORT ===

Hotspots (most time consuming):
┌─────────────────────────────────────────────────────────────┐
│ matrix_multiply()     ████████████████████████ 45.2% (2.1s) │
│ vector_normalize()    ██████████████ 28.7% (1.3s)           │
│ collision_detect()    ███████ 15.1% (0.7s)                  │
│ render_particles()    ██ 8.9% (0.4s)                        │
│ input_processing()    ▌ 2.1% (0.1s)                         │
└─────────────────────────────────────────────────────────────┘

Memory usage over time:
     MB
  100 ┤                                          ╭─╮
   90 ┤                                      ╭───╯ ╰─╮
   80 ┤                              ╭───────╯       ╰─╮
   70 ┤                      ╭───────╯                 ╰─╮
   60 ┤              ╭───────╯                           ╰─╮
   50 ┤      ╭───────╯                                     ╰─╮
   40 ┤  ╭───╯                                               ╰─
   30 ┤╭─╯
   20 ┼╯
      └┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬
       0    10    20    30    40    50    60    70    80    90  sec

Cache miss rate:
L1: 2.3% ████████████████████████████████████████████████████
L2: 8.7% ████████████████████████████████████████████████████
L3: 23.1% ████████████████████████████████████████████████████
```

---

## 🎯 Практические задания

### Задание 1: Оптимизация матричного умножения

Реализуйте и сравните производительность различных подходов:

1. **Наивный алгоритм** (O(n³))
2. **Cache-friendly версия** (блочное умножение)
3. **SIMD оптимизированная версия**

### Задание 2: Система управления памятью для игры

Создайте complete memory management system для 2D игры:

1. **Object pools** для игровых объектов
2. **Stack allocator** для временных данных кадра
3. **Memory tracker** для отладки утечек

### Задание 3: Высокопроизводительный парсер JSON

Оптимизируйте парсер JSON с учетом:

1. **Минимизации allocations**
2. **Cache-friendly доступа к данным**
3. **SIMD оптимизаций для числовых данных**

---

## ✅ Контрольные вопросы

1. Объясните разницу между spatial и temporal locality
2. Почему Structure of Arrays быстрее Array of Structures?
3. Когда использовать Object Pool, а когда Memory Pool?
4. Как битовые поля помогают экономить память?
5. Какие метрики важны при профилировании производительности?

---

## 📚 Дополнительные ресурсы

- **Intel Optimization Manual** - детали по оптимизации под x86
- **Agner Fog's optimization manuals** - практические советы
- **"Computer Systems: A Programmer's Perspective"** - глубокое понимание архитектуры
- **Chandler Carruth's CppCon talks** - современные техники оптимизации

**Следующий блок:** Специализированные структуры данных →