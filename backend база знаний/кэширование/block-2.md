# Блок 2: Аппаратное кэширование
**⏱️ Продолжительность:** 2-3 недели  
**🎯 Цель:** Изучить кэширование на уровне процессора и памяти, научиться писать cache-friendly код

---

## 📖 Глава 1: Иерархия памяти

### Обзор иерархии памяти

Современные компьютеры используют многоуровневую иерархию памяти для баланса между скоростью, объемом и стоимостью.

```
                    ИЕРАРХИЯ ПАМЯТИ
────────────────────────────────────────────────────────────
                        ▲ Скорость
                        │ ▼ Объем  
                        │ ▼ Стоимость за байт

┌─────────────────┐ ← Регистры процессора
│   REGISTERS     │   • 32-64 регистра × 64 бита
│   ~0.25ns       │   • Встроены в процессор
│   ~256 байт     │   • Прямой доступ процессора
└─────────────────┘

┌─────────────────┐ ← L1 кэш (данные + инструкции)
│   L1 CACHE      │   • 32-64 KB (обычно разделен)
│   ~1ns          │   • L1d (данные) + L1i (инструкции)
│   32-64 KB      │   • Один кэш на ядро
└─────────────────┘

┌─────────────────┐ ← L2 кэш (объединенный)
│   L2 CACHE      │   • 256KB - 1MB
│   ~3-5ns        │   • Объединенные данные и инструкции
│   256KB-1MB     │   • Один кэш на ядро или пару ядер
└─────────────────┘

┌─────────────────┐ ← L3 кэш (последний уровень)
│   L3 CACHE      │   • 8-32 MB
│   ~10-20ns      │   • Разделяется между всеми ядрами
│   8-32 MB       │   • Last Level Cache (LLC)
└─────────────────┘

┌─────────────────┐ ← Основная память
│   MAIN MEMORY   │   • 8-128 GB (DDR4/DDR5)
│   ~100-300ns    │   • DRAM технология
│   8-128 GB      │   • Энергозависимая
└─────────────────┘

┌─────────────────┐ ← Дисковое хранилище
│   STORAGE       │   • 256GB - 8TB+
│   ~0.1-10ms     │   • SSD: ~0.1ms, HDD: ~10ms
│   256GB-8TB+    │   • Энергонезависимая
└─────────────────┘
```

### Характеристики каждого уровня

#### Регистры процессора
```
┌─────────────────────────────────────────────────────────┐
│                     РЕГИСТРЫ                            │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Общего назначения (x86-64):                             │
│ ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐       │
│ │ RAX │ RBX │ RCX │ RDX │ RSI │ RDI │ RSP │ RBP │       │
│ └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘       │
│                                                         │
│ Векторные регистры (AVX-512):                           │
│ ┌───────────────────────────────────────────────────┐   │
│ │               ZMM0 (512 бит)                      │   │
│ ├───────────────────────────────────────────────────┤   │
│ │               ZMM1 (512 бит)                      │   │
│ │               ...                                 │   │
│ │               ZMM31 (512 бит)                     │   │
│ └───────────────────────────────────────────────────┘   │
│                                                         │
│ Характеристики:                                         │
│ • Латентность: 0.25-0.5 ns                             │
│ • Пропускная способность: >1TB/s                       │
│ • Прямое управление компилятором                        │
└─────────────────────────────────────────────────────────┘

### Memory Barriers и их использование

```
┌─────────────────────────────────────────────────────────┐
│                   MEMORY BARRIERS                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Проблема: процессоры могут переупорядочивать операции   │
│ для оптимизации производительности                      │
│                                                         │
│ Пример проблемы (без барьеров):                         │
│                                                         │
│ // Thread 1                // Thread 2                 │
│ data = 42;                 while (!ready);              │
│ ready = true;              printf("%d", data);          │
│                                                         │
│ Проблема: процессор может поменять порядок записей:     │
│ ready = true;  ← выполнено первым!                      │
│ data = 42;     ← выполнено вторым                       │
│                                                         │
│ Результат: Thread 2 может увидеть ready=true,           │
│ но data еще не записан!                                 │
│                                                         │
│ Типы memory barriers:                                   │
│                                                         │
│ 1. COMPILER BARRIERS                                    │
│    Запрещают компилятору переупорядочивать код          │
│                                                         │
│    asm volatile("" : : : "memory");  // GCC             │
│    std::atomic_signal_fence();       // C++11          │
│                                                         │
│ 2. CPU BARRIERS                                         │
│    Управляют порядком выполнения на процессоре         │
│                                                         │
│    // x86-64                                            │
│    asm volatile("mfence" : : : "memory");  // Full      │
│    asm volatile("sfence" : : : "memory");  // Store     │
│    asm volatile("lfence" : : : "memory");  // Load      │
│                                                         │
│    // ARM                                               │
│    asm volatile("dmb sy" : : : "memory");  // Full      │
│    asm volatile("dmb st" : : : "memory");  // Store     │
│    asm volatile("dmb ld" : : : "memory");  // Load      │
│                                                         │
│ 3. ACQUIRE/RELEASE SEMANTICS                            │
│                                                         │
│    std::atomic<bool> ready{false};                      │
│    std::atomic<int> data{0};                            │
│                                                         │
│    // Thread 1                                          │
│    data.store(42, std::memory_order_relaxed);           │
│    ready.store(true, std::memory_order_release);        │
│                                                         │
│    // Thread 2                                          │
│    while (!ready.load(std::memory_order_acquire));      │
│    int value = data.load(std::memory_order_relaxed);    │
│                                                         │
│ Визуализация acquire/release:                           │
│                                                         │
│ Thread 1:                   Thread 2:                   │
│ ┌─────────────┐             ┌─────────────┐            │
│ │ data = 42   │             │             │            │
│ │     ↓       │             │    ↓        │            │
│ │ RELEASE     │────────────►│ ACQUIRE     │            │
│ │ ready=true  │  synchronize │ read ready  │            │
│ └─────────────┘             │     ↓       │            │
│                             │ read data   │            │
│                             │ (sees 42)   │            │
│                             └─────────────┘            │
│                                                         │
│ Гарантии:                                               │
│ • Все записи перед RELEASE видны после ACQUIRE          │
│ • Нет переупорядочивания через границу                  │
└─────────────────────────────────────────────────────────┘
```

---

## 🛠️ Практические задания

### Задание 1: Измерение латентности кэшей

**Цель:** Создать программу для измерения характеристик кэшей

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <stdint.h>

#define KB (1024)
#define MB (1024 * KB)

// Структура для результатов
struct cache_info {
    size_t size;
    double latency_ns;
    double bandwidth_gbps;
};

/**
 * Измерение латентности доступа к памяти
 * array_size - размер массива для тестирования
 * iterations - количество итераций
 */
double measure_latency(size_t array_size, int iterations) {
    // TODO: Реализовать измерение латентности
    // 1. Создать массив указанного размера
    // 2. Инициализировать паттерн pointer chasing
    // 3. Измерить время прохода по всему массиву
    // 4. Вернуть среднее время доступа к одному элементу
    
    uint8_t *array = malloc(array_size);
    if (!array) return -1;
    
    // Заполнение случайным паттерном доступа
    // для исключения prefetcher'а
    
    // Ваш код здесь...
    
    free(array);
    return 0.0; // placeholder
}

/**
 * Определение размеров кэшей по изменению латентности
 */
void detect_cache_sizes() {
    printf("Размер массива\tЛатентность (нс)\tПредполагаемый уровень\n");
    printf("─────────────────────────────────────────────────────────\n");
    
    for (size_t size = 1*KB; size <= 64*MB; size *= 2) {
        double latency = measure_latency(size, 1000000);
        
        const char* level = "Unknown";
        if (latency < 2.0) level = "L1 Cache";
        else if (latency < 8.0) level = "L2 Cache";
        else if (latency < 25.0) level = "L3 Cache";
        else level = "Main Memory";
        
        printf("%8zu KB\t%10.2f ns\t%s\n", 
               size/KB, latency, level);
    }
}

int main() {
    detect_cache_sizes();
    return 0;
}
```

**Ожидаемые результаты:**
```
Размер массива    Латентность (нс)    Предполагаемый уровень
─────────────────────────────────────────────────────────
       4 KB              1.2 ns       L1 Cache
       8 KB              1.3 ns       L1 Cache
      16 KB              1.5 ns       L1 Cache
      32 KB              1.8 ns       L1 Cache
      64 KB              4.5 ns       L2 Cache
     128 KB              5.2 ns       L2 Cache
     256 KB              6.1 ns       L2 Cache
     512 KB              7.8 ns       L2 Cache
       1 MB             18.5 ns       L3 Cache
       2 MB             19.2 ns       L3 Cache
       4 MB             20.1 ns       L3 Cache
       8 MB             22.5 ns       L3 Cache
      16 MB            120.0 ns       Main Memory
      32 MB            125.0 ns       Main Memory
```

### Задание 2: Оптимизация умножения матриц

**Цель:** Реализовать и сравнить различные алгоритмы матричного умножения

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define MATRIX_SIZE 1024

typedef double matrix_t[MATRIX_SIZE][MATRIX_SIZE];

/**
 * Стандартное умножение матриц (ijk)
 */
void multiply_standard(matrix_t A, matrix_t B, matrix_t C) {
    for (int i = 0; i < MATRIX_SIZE; i++) {
        for (int j = 0; j < MATRIX_SIZE; j++) {
            C[i][j] = 0.0;
            for (int k = 0; k < MATRIX_SIZE; k++) {
                C[i][j] += A[i][k] * B[k][j];
            }
        }
    }
}

/**
 * Блочное умножение матриц
 */
void multiply_blocked(matrix_t A, matrix_t B, matrix_t C, int block_size) {
    // TODO: Реализовать блочный алгоритм
    // 1. Разбить матрицы на блоки размером block_size
    // 2. Выполнить умножение блоков
    // 3. Внутри блоков использовать стандартный алгоритм
    
    memset(C, 0, sizeof(matrix_t));
    
    for (int ii = 0; ii < MATRIX_SIZE; ii += block_size) {
        for (int jj = 0; jj < MATRIX_SIZE; jj += block_size) {
            for (int kk = 0; kk < MATRIX_SIZE; kk += block_size) {
                // Умножение блоков
                for (int i = ii; i < ii + block_size && i < MATRIX_SIZE; i++) {
                    for (int j = jj; j < jj + block_size && j < MATRIX_SIZE; j++) {
                        for (int k = kk; k < kk + block_size && k < MATRIX_SIZE; k++) {
                            C[i][j] += A[i][k] * B[k][j];
                        }
                    }
                }
            }
        }
    }
}

/**
 * Транспонированное умножение (для лучшего доступа к B)
 */
void multiply_transposed(matrix_t A, matrix_t B, matrix_t C) {
    // TODO: Реализовать алгоритм с транспозицией матрицы B
    // 1. Транспонировать матрицу B
    // 2. Изменить порядок доступа для лучшей локальности
    
    // Ваш код здесь...
}

/**
 * Бенчмарк различных алгоритмов
 */
void benchmark_algorithms() {
    matrix_t *A = malloc(sizeof(matrix_t));
    matrix_t *B = malloc(sizeof(matrix_t));
    matrix_t *C = malloc(sizeof(matrix_t));
    
    // Инициализация матриц
    for (int i = 0; i < MATRIX_SIZE; i++) {
        for (int j = 0; j < MATRIX_SIZE; j++) {
            (*A)[i][j] = (double)rand() / RAND_MAX;
            (*B)[i][j] = (double)rand() / RAND_MAX;
        }
    }
    
    printf("Алгоритм\t\tВремя (сек)\tGFLOPS\tУскорение\n");
    printf("────────────────────────────────────────────────────\n");
    
    // Стандартный алгоритм
    clock_t start = clock();
    multiply_standard(*A, *B, *C);
    clock_t end = clock();
    double time_standard = (double)(end - start) / CLOCKS_PER_SEC;
    double gflops_standard = (2.0 * MATRIX_SIZE * MATRIX_SIZE * MATRIX_SIZE) / (time_standard * 1e9);
    
    printf("Стандартный\t%.3f\t\t%.2f\t1.00x\n", time_standard, gflops_standard);
    
    // Блочные алгоритмы с разными размерами блоков
    int block_sizes[] = {32, 64, 128, 256};
    for (int i = 0; i < 4; i++) {
        start = clock();
        multiply_blocked(*A, *B, *C, block_sizes[i]);
        end = clock();
        double time_blocked = (double)(end - start) / CLOCKS_PER_SEC;
        double gflops_blocked = (2.0 * MATRIX_SIZE * MATRIX_SIZE * MATRIX_SIZE) / (time_blocked * 1e9);
        double speedup = time_standard / time_blocked;
        
        printf("Блочный %d\t\t%.3f\t\t%.2f\t%.2fx\n", 
               block_sizes[i], time_blocked, gflops_blocked, speedup);
    }
    
    free(A);
    free(B);
    free(C);
}
```

**Критерии оценки:**
- ✅ Правильная реализация всех алгоритмов
- ✅ Измерение производительности в GFLOPS
- ✅ Анализ влияния размера блока
- ✅ Объяснение результатов с точки зрения кэширования

### Задание 3: Исследование False Sharing

**Цель:** Продемонстрировать влияние false sharing и способы его устранения

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <unistd.h>

#define NUM_THREADS 4
#define ITERATIONS 10000000
#define CACHE_LINE_SIZE 64

// Проблемный вариант - false sharing
struct bad_counters {
    volatile long counter[NUM_THREADS];
};

// Хороший вариант - padding
struct good_counters {
    volatile long counter;
    char padding[CACHE_LINE_SIZE - sizeof(long)];
};

// Глобальные переменные для тестирования
struct bad_counters bad_data;
struct good_counters good_data[NUM_THREADS];

/**
 * Воркер для false sharing теста
 */
void* worker_bad(void* arg) {
    int thread_id = *(int*)arg;
    
    for (int i = 0; i < ITERATIONS; i++) {
        bad_data.counter[thread_id]++;
    }
    
    return NULL;
}

/**
 * Воркер для теста без false sharing
 */
void* worker_good(void* arg) {
    int thread_id = *(int*)arg;
    
    for (int i = 0; i < ITERATIONS; i++) {
        good_data[thread_id].counter++;
    }
    
    return NULL;
}

/**
 * Бенчмарк false sharing
 */
void benchmark_false_sharing() {
    pthread_t threads[NUM_THREADS];
    int thread_ids[NUM_THREADS];
    struct timespec start, end;
    
    printf("False Sharing Benchmark\n");
    printf("Threads: %d, Iterations per thread: %d\n\n", NUM_THREADS, ITERATIONS);
    
    // Тест с false sharing
    for (int i = 0; i < NUM_THREADS; i++) {
        bad_data.counter[i] = 0;
        thread_ids[i] = i;
    }
    
    clock_gettime(CLOCK_MONOTONIC, &start);
    
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_create(&threads[i], NULL, worker_bad, &thread_ids[i]);
    }
    
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }
    
    clock_gettime(CLOCK_MONOTONIC, &end);
    double time_bad = (end.tv_sec - start.tv_sec) + 
                      (end.tv_nsec - start.tv_nsec) / 1e9;
    
    // Тест без false sharing
    for (int i = 0; i < NUM_THREADS; i++) {
        good_data[i].counter = 0;
    }
    
    clock_gettime(CLOCK_MONOTONIC, &start);
    
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_create(&threads[i], NULL, worker_good, &thread_ids[i]);
    }
    
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }
    
    clock_gettime(CLOCK_MONOTONIC, &end);
    double time_good = (end.tv_sec - start.tv_sec) + 
                       (end.tv_nsec - start.tv_nsec) / 1e9;
    
    // Результаты
    printf("Результаты:\n");
    printf("С false sharing:    %.3f сек\n", time_bad);
    printf("Без false sharing:  %.3f сек\n", time_good);
    printf("Замедление:         %.2fx\n", time_bad / time_good);
    
    // Проверка корректности
    long total_bad = 0, total_good = 0;
    for (int i = 0; i < NUM_THREADS; i++) {
        total_bad += bad_data.counter[i];
        total_good += good_data[i].counter;
    }
    printf("\nПроверка корректности:\n");
    printf("Сумма (false sharing): %ld\n", total_bad);
    printf("Сумма (без false sharing): %ld\n", total_good);
    printf("Ожидаемая сумма: %ld\n", (long)NUM_THREADS * ITERATIONS);
}

int main() {
    benchmark_false_sharing();
    return 0;
}
```

### Задание 4: Профайлер cache miss'ов

**Цель:** Создать инструмент для анализа cache miss'ов

```bash
#!/bin/bash
# cache_profiler.sh - скрипт для профилирования кэша

if [ $# -eq 0 ]; then
    echo "Использование: $0 <программа> [аргументы]"
    exit 1
fi

PROGRAM="$1"
shift
ARGS="$@"

echo "Профилирование кэша для: $PROGRAM $ARGS"
echo "========================================"

# Проверка наличия perf
if ! command -v perf &> /dev/null; then
    echo "Ошибка: perf не найден. Установите linux-tools."
    exit 1
fi

# Основные метрики кэша
echo "1. Общая статистика кэша:"
perf stat -e cache-references,cache-misses,L1-dcache-loads,L1-dcache-load-misses,L1-icache-loads,L1-icache-load-misses \
    "$PROGRAM" $ARGS

echo -e "\n2. Детальная статистика по уровням:"
perf stat -e L1-dcache-loads,L1-dcache-load-misses,L1-dcache-stores,L1-dcache-store-misses,\
L1-icache-loads,L1-icache-load-misses,\
LLC-loads,LLC-load-misses,LLC-stores,LLC-store-misses \
    "$PROGRAM" $ARGS

echo -e "\n3. Анализ TLB (Translation Lookaside Buffer):"
perf stat -e dTLB-loads,dTLB-load-misses,dTLB-stores,dTLB-store-misses,\
iTLB-loads,iTLB-load-misses \
    "$PROGRAM" $ARGS

# Создание профиля для анализа hotspots
echo -e "\n4. Создание профиля (perf record)..."
perf record -e cache-misses "$PROGRAM" $ARGS

echo -e "\n5. Топ функций по cache miss'ам:"
perf report --stdio | head -20

# Cleanup
rm -f perf.data
```

**Пример использования:**
```bash
# Компиляция тестовой программы
gcc -O2 -o matrix_mult matrix_mult.c

# Профилирование
./cache_profiler.sh ./matrix_mult

# Ожидаемый вывод:
# ========================================
# 1. Общая статистика кэша:
#    cache-references:     124,567,890
#    cache-misses:          12,456,789  (10.00%)
#    L1-dcache-loads:      987,654,321
#    L1-dcache-load-misses: 45,678,901  (4.63%)
```

---

## ✅ Контрольные вопросы

### Теоретические вопросы:

1. **Объясните разницу между direct-mapped, set-associative и fully associative кэшами. Когда каждый тип предпочтителен?**

2. **Что такое протокол MESI и зачем он нужен? Опишите переходы между состояниями.**

3. **Что такое false sharing и почему это проблема для многопоточных приложений?**

4. **Объясните разницу между memory barriers, compiler barriers и acquire/release semantics.**

### Практические задачи:

1. **Расчет производительности кэша:**
   ```
   Дано:
   - L1 cache: 32KB, 8-way, латентность 1ns
   - L2 cache: 256KB, 8-way, латентность 5ns  
   - L3 cache: 8MB, 16-way, латентность 15ns
   - Memory: латентность 100ns
   - Программа: 90% L1 hits, 8% L2 hits, 1.5% L3 hits, 0.5% memory
   
   Найти среднее время доступа к данным.
   ```

2. **Анализ алгоритма:**
   ```c
   // Какие проблемы с кэшированием в этом коде?
   void process_matrix(int matrix[1000][1000]) {
       for (int j = 0; j < 1000; j++) {
           for (int i = 0; i < 1000; i++) {
               matrix[i][j] = matrix[i][j] * 2;
           }
       }
   }
   
   // Как его оптимизировать?
   ```

3. **Размер оптимального блока:**
   ```
   Для матричного умножения с кэшем L1 = 32KB:
   - Какой оптимальный размер блока для алгоритма C = A × B?
   - Сколько элементов double поместится в блок?
   - Обоснуйте свой расчет.
   ```

---

## 📚 Дополнительные материалы

### Инструменты для анализа кэша:

**1. Intel VTune Profiler**
- Графический интерфейс для анализа производительности
- Детальная информация о cache miss'ах
- Hotspot analysis и bottleneck detection

**2. Linux perf**
```bash
# Основные команды
perf stat -e cache-misses ./program
perf record -e cache-misses ./program
perf report

# Детальный анализ
perf mem record ./program
perf mem report
```

**3. Valgrind/Cachegrind**
```bash
# Симуляция кэша
valgrind --tool=cachegrind ./program

# Аннотация исходного кода
cg_annotate cachegrind.out.pid
```

**4. CPU-specific PMU events**
```bash
# Список доступных событий
perf list cache

# Специфичные для Intel
perf stat -e cpu/event=0x24,umask=0x21/ ./program
```

### Рекомендуемое чтение:

**Книги:**
- "Computer Architecture: A Quantitative Approach" - Hennessy & Patterson
- "What Every Programmer Should Know About Memory" - Ulrich Drepper
- "Systems Performance" - Brendan Gregg

**Статьи:**
- Intel® 64 and IA-32 Architectures Optimization Reference Manual
- ARM Cortex-A Series Programmer's Guide
- "Cache-Conscious Programming" research papers

**Онлайн ресурсы:**
- Intel Performance Analysis Guide
- Agner Fog's optimization manuals
- Linux kernel documentation on CPU caches

### Следующий блок:
**Блок 3: Кэширование в веб-разработке** - изучение браузерного кэширования, CDN, HTTP заголовков и серверных кэширующих стратегий.

---

## 🎯 Чек-лист завершения блока

- [ ] Понимаю иерархию памяти современных компьютеров
- [ ] Знаю архитектуру процессорных кэшей (direct-mapped, set-associative, fully associative)
- [ ] Умею писать cache-friendly код
- [ ] Понимаю блочные алгоритмы и их применение
- [ ] Знаю принципы работы prefetching
- [ ] Понимаю проблемы когерентности кэша и протоколы MESI/MOESI
- [ ] Умею избегать false sharing в многопоточном коде
- [ ] Знаю, когда и как использовать memory barriers
- [ ] Создал инструменты для профилирования кэша
- [ ] Оптимизировал алгоритмы с учетом аппаратного кэширования

**Отлично! Теперь вы понимаете аппаратное кэширование на экспертном уровне! 🚀**
```

#### L1 кэш (Level 1)
```
┌─────────────────────────────────────────────────────────┐
│                      L1 КЭШЫ                           │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Архитектура разделения:                                 │
│                                                         │
│   CPU Core                                              │
│ ┌─────────────────────────────────────────────────────┐ │
│ │ ┌─────────────┐    ┌─────────────┐                  │ │
│ │ │   L1i       │    │    L1d      │                  │ │
│ │ │(инструкции) │    │  (данные)   │                  │ │
│ │ │   32 KB     │    │   32 KB     │                  │ │
│ │ │   4-way     │    │   8-way     │                  │ │
│ │ └─────────────┘    └─────────────┘                  │ │
│ │           │              │                          │ │
│ │           └──────┬───────┘                          │ │
│ │                  │                                  │ │
│ │            ┌─────────────┐                          │ │
│ │            │  L2 Cache   │                          │ │
│ │            │   256 KB    │                          │ │
│ │            └─────────────┘                          │ │
│ └─────────────────────────────────────────────────────┘ │
│                                                         │
│ Характеристики L1:                                      │
│ • Размер: 32-64 KB каждый                               │
│ • Латентность: 1-2 такта (~1ns)                        │
│ • Ассоциативность: 4-8 way                             │
│ • Размер строки: 64 байта                               │
│ • Write policy: обычно write-through в L2              │
└─────────────────────────────────────────────────────────┘
```

#### L2 и L3 кэши
```
┌─────────────────────────────────────────────────────────┐
│                   L2 и L3 КЭШИ                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Многоядерная архитектура:                               │
│                                                         │
│ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐         │
│ │ Core 0  │ │ Core 1  │ │ Core 2  │ │ Core 3  │         │
│ │ ┌─────┐ │ │ ┌─────┐ │ │ ┌─────┐ │ │ ┌─────┐ │         │
│ │ │ L1  │ │ │ │ L1  │ │ │ │ L1  │ │ │ │ L1  │ │         │
│ │ └─────┘ │ │ └─────┘ │ │ └─────┘ │ │ └─────┘ │         │
│ │    │    │ │    │    │ │    │    │ │    │    │         │
│ │ ┌─────┐ │ │ ┌─────┐ │ │ ┌─────┐ │ │ ┌─────┐ │         │
│ │ │ L2  │ │ │ │ L2  │ │ │ │ L2  │ │ │ │ L2  │ │         │
│ │ │256KB│ │ │ │256KB│ │ │ │256KB│ │ │ │256KB│ │         │
│ │ └─────┘ │ │ └─────┘ │ │ └─────┘ │ │ └─────┘ │         │
│ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘         │
│      │           │           │           │              │
│      └───────────┬───────────┬───────────┘              │
│                  │           │                          │
│              ┌─────────────────────┐                     │
│              │      L3 Cache       │                     │
│              │      16-32 MB       │                     │
│              │    16-20 way        │                     │
│              │   (разделяемый)     │                     │
│              └─────────────────────┘                     │
│                         │                               │
│              ┌─────────────────────┐                     │
│              │    Main Memory      │                     │
│              │      8-128 GB       │                     │
│              └─────────────────────┘                     │
│                                                         │
│ L2 характеристики:                                      │
│ • Размер: 256KB - 1MB                                   │
│ • Латентность: 3-5 нс (~10-15 тактов)                  │
│ • Объединенный (данные + инструкции)                    │
│                                                         │
│ L3 характеристики:                                      │
│ • Размер: 8-32 MB                                       │
│ • Латентность: 10-20 нс (~30-60 тактов)                │
│ • Разделяется между всеми ядрами                        │
│ • Inclusive или Non-inclusive                           │
└─────────────────────────────────────────────────────────┘
```

### Принципы работы иерархии

#### Принцип включения (Inclusion)
```
┌─────────────────────────────────────────────────────────┐
│                ПОЛИТИКИ ВКЛЮЧЕНИЯ                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ 1. INCLUSIVE HIERARCHY (включающая)                     │
│    L3 ⊃ L2 ⊃ L1                                        │
│                                                         │
│    ┌─────────────────────────────────────────────────┐   │
│    │              L3 Cache                           │   │
│    │ ┌─────────────────────────────────────────────┐ │   │
│    │ │            L2 Cache                         │ │   │
│    │ │ ┌─────────────────────────────────────────┐ │ │   │
│    │ │ │          L1 Cache                       │ │ │   │
│    │ │ │ [A] [B] [C] [D]                         │ │ │   │
│    │ │ └─────────────────────────────────────────┘ │ │   │
│    │ │ [A] [B] [C] [D] [E] [F] [G] [H]             │ │   │
│    │ └─────────────────────────────────────────────┘ │   │
│    │ [A] [B] [C] [D] [E] [F] [G] [H] ... [P]         │   │
│    └─────────────────────────────────────────────────┘   │
│                                                         │
│ 2. NON-INCLUSIVE HIERARCHY (исключающая)                │
│    L1 ∩ L2 ∩ L3 = ∅ (нет дублирования)                │
│                                                         │
│    ┌─────────────────────────────────────────────────┐   │
│    │ L3: [I] [J] [K] [L] [M] [N] [O] [P]             │   │
│    └─────────────────────────────────────────────────┘   │
│    ┌─────────────────────────────────────────────────┐   │
│    │ L2: [E] [F] [G] [H]                             │   │
│    └─────────────────────────────────────────────────┘   │
│    ┌─────────────────────────────────────────────────┐   │
│    │ L1: [A] [B] [C] [D]                             │   │
│    └─────────────────────────────────────────────────┘   │
│                                                         │
│ 3. VICTIM CACHE (кэш жертв)                             │
│    Специальный буфер для недавно вытесненных данных     │
│                                                         │
│    ┌─────────────────────────────────────────────────┐   │
│    │ Main Cache: [A] [B] [C] [D]                     │   │
│    └─────────────────────────────────────────────────┘   │
│             ↓ (вытеснение)                              │
│    ┌─────────────────────────────────────────────────┐   │
│    │ Victim Cache: [E] [F]                           │   │
│    └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## 🏗️ Глава 2: Архитектура процессорных кэшей

### Организация кэш-памяти

#### Прямое отображение (Direct-Mapped)
```
┌─────────────────────────────────────────────────────────┐
│               DIRECT-MAPPED CACHE                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ 32-битный адрес памяти:                                 │
│ ┌─────────────┬─────────────┬─────────────┐             │
│ │ Tag (18)    │ Index (8)   │ Offset (6)  │             │
│ └─────────────┴─────────────┴─────────────┘             │
│  биты 31-14    биты 13-6     биты 5-0                   │
│                                                         │
│ Index определяет строку кэша:                           │
│ ┌─────┬─────────────┬─────────────────────────────────┐ │
│ │Index│   Valid     │        Tag        │    Data     │ │
│ ├─────┼─────────────┼───────────────────┼─────────────┤ │
│ │ 0   │     1       │    0x12345        │ [64 bytes]  │ │
│ │ 1   │     0       │    0x00000        │ [64 bytes]  │ │
│ │ 2   │     1       │    0xABCDE        │ [64 bytes]  │ │
│ │ ... │    ...      │     ...           │    ...      │ │
│ │255  │     1       │    0x67890        │ [64 bytes]  │ │
│ └─────┴─────────────┴───────────────────┴─────────────┘ │
│                                                         │
│ Пример поиска адреса 0x12345180:                        │
│ • Tag = 0x12345                                         │
│ • Index = 0x06 (строка 6)                              │
│ • Offset = 0x00 (байт 0 в строке)                      │
│                                                         │
│ Преимущества:                                           │
│ ✓ Простая реализация                                    │
│ ✓ Быстрый поиск                                         │
│ ✓ Низкие аппаратные затраты                             │
│                                                         │
│ Недостатки:                                             │
│ ✗ Конфликты отображения                                 │
│ ✗ Плохая производительность при конфликтах адресов     │
└─────────────────────────────────────────────────────────┘
```

#### Множественно-ассоциативные кэши
```
┌─────────────────────────────────────────────────────────┐
│              N-WAY SET-ASSOCIATIVE CACHE                │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ 4-way Set-Associative (пример):                         │
│                                                         │
│ 32-битный адрес:                                        │
│ ┌─────────────┬─────────────┬─────────────┐             │
│ │ Tag (20)    │ Set (6)     │ Offset (6)  │             │
│ └─────────────┴─────────────┴─────────────┘             │
│                                                         │
│ Структура кэша:                                         │
│                Way 0    Way 1    Way 2    Way 3         │
│ ┌─────┬─────────────┬─────────────┬─────────────┬─────┐ │
│ │Set 0│V│Tag│Data  │V│Tag│Data  │V│Tag│Data  │V│..│  │ │
│ ├─────┼─────────────┼─────────────┼─────────────┼─────┤ │
│ │Set 1│1│0x1│[64B] │0│0x0│[64B] │1│0x5│[64B] │1│..│  │ │
│ ├─────┼─────────────┼─────────────┼─────────────┼─────┤ │
│ │Set 2│1│0x2│[64B] │1│0x3│[64B] │0│0x0│[64B] │1│..│  │ │
│ ├─────┼─────────────┼─────────────┼─────────────┼─────┤ │
│ │ ... │ ... │ ...  │ ... │ ...  │ ... │ ...  │ ..│..│  │ │
│ ├─────┼─────────────┼─────────────┼─────────────┼─────┤ │
│ │Set63│1│0xF│[64B] │1│0xA│[64B] │1│0xB│[64B] │0│..│  │ │
│ └─────┴─────────────┴─────────────┴─────────────┴─────┘ │
│                                                         │
│ Процесс поиска:                                         │
│ 1. Set index → выбор строки (Set)                      │
│ 2. Параллельное сравнение тегов во всех Ways            │
│ 3. Выбор подходящего Way (если есть совпадение)        │
│ 4. Извлечение данных по offset                          │
│                                                         │
│ Преимущества по сравнению с Direct-Mapped:              │
│ ✓ Меньше конфликтов отображения                         │
│ ✓ Лучший hit rate                                       │
│ ✓ Более гибкое размещение данных                        │
│                                                         │
│ Недостатки:                                             │
│ ✗ Сложнее аппаратура                                    │
│ ✗ Больше энергопотребление                              │
│ ✗ Нужен алгоритм замещения                              │
└─────────────────────────────────────────────────────────┘
```

#### Полностью ассоциативные кэши
```
┌─────────────────────────────────────────────────────────┐
│              FULLY ASSOCIATIVE CACHE                    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ 32-битный адрес:                                        │
│ ┌─────────────────────────┬─────────────┐               │
│ │      Tag (26)           │ Offset (6)  │               │
│ └─────────────────────────┴─────────────┘               │
│                                                         │
│ Структура кэша (любая строка может содержать любые      │
│ данные):                                                │
│                                                         │
│ ┌─────┬───┬──────────┬─────────────────────────────────┐ │
│ │Line │ V │   Tag    │             Data               │ │
│ ├─────┼───┼──────────┼─────────────────────────────────┤ │
│ │  0  │ 1 │0x1234567 │ [────── 64 bytes ──────]      │ │
│ │  1  │ 1 │0xABCDEF0 │ [────── 64 bytes ──────]      │ │
│ │  2  │ 0 │0x0000000 │ [────── 64 bytes ──────]      │ │
│ │  3  │ 1 │0x9876543 │ [────── 64 bytes ──────]      │ │
│ │ ... │...│   ...    │           ...                 │ │
│ │ 63  │ 1 │0x1111111 │ [────── 64 bytes ──────]      │ │
│ └─────┴───┴──────────┴─────────────────────────────────┘ │
│                                                         │
│ Процесс поиска:                                         │
│ ┌─────────────────────────────────────────────────────┐ │
│ │ 1. Параллельное сравнение со ВСЕМИ тегами          │ │
│ │ 2. Если совпадение найдено → HIT                   │ │
│ │ 3. Если нет совпадений → MISS                      │ │
│ │ 4. При MISS → выбор жертвы по алгоритму LRU        │ │
│ └─────────────────────────────────────────────────────┘ │
│                                                         │
│ CAM (Content Addressable Memory) реализация:            │
│                                                         │
│ Input Tag: 0x1234567                                    │
│      ↓                                                  │
│ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐        │
│ │Comparator│ │Comparator│ │Comparator│ │Comparator│       │
│ │    0    │ │    1    │ │    2    │ │    3    │        │
│ └─────────┘ └─────────┘ └─────────┘ └─────────┘        │
│      │            │            │            │           │
│   Match=1      Match=0      Match=0      Match=0        │
│      │            │            │            │           │
│      └────────────┼────────────┼────────────┘           │
│                   │            │                        │
│              Hit Signal    Line Select                  │
│                                                         │
│ Преимущества:                                           │
│ ✓ Максимальная гибкость размещения                      │
│ ✓ Наилучший hit rate                                    │
│ ✓ Нет конфликтов отображения                            │
│                                                         │
│ Недостатки:                                             │
│ ✗ Очень дорогая аппаратура                              │
│ ✗ Высокое энергопотребление                             │
│ ✗ Сложная схема сравнения                               │
│ ✗ Применим только для малых кэшей                       │
└─────────────────────────────────────────────────────────┘
```

### Алгоритмы замещения в аппаратных кэшах

#### CLOCK Algorithm (аппаратная реализация)
```
┌─────────────────────────────────────────────────────────┐
│                   CLOCK ALGORITHM                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Кольцевая структура с указателем:                       │
│                                                         │
│           ┌─────────┐                                   │
│        ┌──│ Block 0 │◄─── Clock Hand                    │
│        │  │ Use=1   │                                   │
│        │  └─────────┘                                   │
│        │                                                │
│  ┌─────────┐                             ┌─────────┐    │
│  │ Block 7 │                             │ Block 1 │    │
│  │ Use=0   │                             │ Use=0   │    │
│  └─────────┘                             └─────────┘    │
│        │                                        │       │
│  ┌─────────┐                             ┌─────────┐    │
│  │ Block 6 │                             │ Block 2 │    │
│  │ Use=1   │                             │ Use=1   │    │
│  └─────────┘                             └─────────┘    │
│        │                                        │       │
│        │  ┌─────────┐     ┌─────────┐          │       │
│        └──│ Block 5 │     │ Block 3 │──────────┘       │
│           │ Use=0   │     │ Use=1   │                  │
│           └─────────┘     └─────────┘                  │
│                  │               │                     │
│                  │  ┌─────────┐  │                     │
│                  └──│ Block 4 │──┘                     │
│                     │ Use=0   │                        │
│                     └─────────┘                        │
│                                                         │
│ Алгоритм замещения:                                     │
│ ┌─────────────────────────────────────────────────────┐ │
│ │ 1. Clock hand указывает на текущий блок            │ │
│ │ 2. Если Use bit = 0 → выбираем для замещения       │ │
│ │ 3. Если Use bit = 1 → сбрасываем в 0, идем дальше  │ │
│ │ 4. Поворачиваем clock hand по часовой стрелке      │ │
│ │ 5. Повторяем до нахождения блока с Use=0           │ │
│ └─────────────────────────────────────────────────────┘ │
│                                                         │
│ Пример выполнения (поиск жертвы):                       │
│ Состояние: Hand на Block 0                             │
│ Block 0: Use=1 → сброс в 0, Hand → Block 1             │
│ Block 1: Use=0 → ВЫБРАН для замещения!                 │
│                                                         │
│ Аппаратная реализация Use bit:                         │
│ • Устанавливается в 1 при каждом обращении            │
│ • Периодически сбрасывается таймером                   │
│ • Или сбрасывается алгоритмом Clock                    │
└─────────────────────────────────────────────────────────┘
```

---

## 💻 Глава 3: Cache-friendly программирование

### Оптимизация доступа к массивам

#### Влияние порядка обхода на производительность
```
┌─────────────────────────────────────────────────────────┐
│            PATTERNS OF MEMORY ACCESS                    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Двумерный массив в памяти (row-major order):            │
│                                                         │
│ int matrix[4][4] в памяти:                              │
│ ┌────┬────┬────┬────┬────┬────┬────┬────┐               │
│ │[0,0]│[0,1]│[0,2]│[0,3]│[1,0]│[1,1]│[1,2]│[1,3]│      │
│ └────┴────┴────┴────┴────┴────┴────┴────┘               │
│ ┌────┬────┬────┬────┬────┬────┬────┬────┐               │
│ │[2,0]│[2,1]│[2,2]│[2,3]│[3,0]│[3,1]│[3,2]│[3,3]│      │
│ └────┴────┴────┴────┴────┴────┴────┴────┘               │
│                                                         │
│ ХОРОШО - Row-major обход (cache-friendly):              │
│ for (i = 0; i < N; i++)                                 │
│     for (j = 0; j < N; j++)                             │
│         sum += matrix[i][j];                            │
│                                                         │
│ Паттерн доступа: →→→→→→→→→→→→→→→→                     │
│ ┌────┬────┬────┬────┐                                   │
│ │ 1→ │ 2→ │ 3→ │ 4→ │  ← одна cache line              │
│ ├────┼────┼────┼────┤                                   │
│ │ 5→ │ 6→ │ 7→ │ 8→ │  ← следующая cache line         │
│ └────┴────┴────┴────┘                                   │
│                                                         │
│ ПЛОХО - Column-major обход (cache-unfriendly):          │
│ for (j = 0; j < N; j++)                                 │
│     for (i = 0; i < N; i++)                             │
│         sum += matrix[i][j];                            │
│                                                         │
│ Паттерн доступа: ↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓                     │
│ ┌────┬────┬────┬────┐                                   │
│ │ 1  │ 5  │ 9  │13  │  ← разные cache lines            │
│ ├────┼────┼────┼────┤                                   │
│ │ 2  │ 6  │10  │14  │  ← разные cache lines            │
│ ├────┼────┼────┼────┤                                   │
│ │ 3  │ 7  │11  │15  │  ← разные cache lines            │
│ ├────┼────┼────┼────┤                                   │
│ │ 4  │ 8  │12  │16  │  ← разные cache lines            │
│ └────┴────┴────┴────┘                                   │
│                                                         │
│ Результаты бенчмарка (1000x1000 матрица):               │
│ ┌─────────────────┬─────────────┬─────────────┐         │
│ │    Паттерн      │    Время    │ Cache misses│         │
│ ├─────────────────┼─────────────┼─────────────┤         │
│ │ Row-major       │   45ms      │     15%     │         │
│ │ Column-major    │  180ms      │     85%     │         │
│ │ Разница         │   4x        │    5.7x     │         │
│ └─────────────────┴─────────────┴─────────────┘         │
└─────────────────────────────────────────────────────────┘
```

#### Блочные алгоритмы для матричных операций
```
┌─────────────────────────────────────────────────────────┐
│              MATRIX MULTIPLICATION BLOCKING              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Стандартный алгоритм (плохо для кэша):                  │
│ for (i = 0; i < N; i++)                                 │
│     for (j = 0; j < N; j++)                             │
│         for (k = 0; k < N; k++)                         │
│             C[i][j] += A[i][k] * B[k][j];               │
│                                                         │
│ Проблема: матрица B обходится по столбцам               │
│                                                         │
│ Блочный алгоритм (cache-friendly):                      │
│                                                         │
│ #define BLOCK_SIZE 64  // подбирается под L1 cache     │
│                                                         │
│ for (ii = 0; ii < N; ii += BLOCK_SIZE)                  │
│   for (jj = 0; jj < N; jj += BLOCK_SIZE)                │
│     for (kk = 0; kk < N; kk += BLOCK_SIZE)              │
│       // Умножение блоков                              │
│       for (i = ii; i < min(ii+BLOCK_SIZE, N); i++)      │
│         for (j = jj; j < min(jj+BLOCK_SIZE, N); j++)    │
│           for (k = kk; k < min(kk+BLOCK_SIZE, N); k++)  │
│             C[i][j] += A[i][k] * B[k][j];               │
│                                                         │
│ Визуализация блочного умножения:                        │
│                                                         │
│     Matrix A        Matrix B        Matrix C            │
│ ┌─────┬─────┐   ┌─────┬─────┐   ┌─────┬─────┐          │
│ │ A11 │ A12 │ × │ B11 │ B12 │ = │ C11 │ C12 │          │
│ ├─────┼─────┤   ├─────┼─────┤   ├─────┼─────┤          │
│ │ A21 │ A22 │   │ B21 │ B22 │   │ C21 │ C22 │          │
│ └─────┴─────┘   └─────┴─────┘   └─────┴─────┘          │
│                                                         │
│ C11 = A11×B11 + A12×B21  (блоки остаются в кэше)       │
│                                                         │
│ Размер блока для оптимальной производительности:        │
│ ┌─────────────────────────────────────────────────────┐ │
│ │ Block Size = √(Cache_Size / 3)                      │ │
│ │                                                     │ │
│ │ Для L1 cache 32KB:                                  │ │
│ │ Block Size = √(32768 / 3) ≈ 104 элементов          │ │
│ │ Для double: 104 × 8 байт = 832 байта на строку     │ │
│ │ Оптимальный размер: 64×64 или 128×128               │ │
│ └─────────────────────────────────────────────────────┘ │
│                                                         │
│ Результаты оптимизации:                                 │
│ ┌─────────────────┬─────────────┬─────────────────────┐ │
│ │   Алгоритм      │   Время     │    Ускорение       │ │
│ ├─────────────────┼─────────────┼─────────────────────┤ │
│ │ Стандартный     │   2.4s      │        1x          │ │
│ │ Блочный 64×64   │   0.8s      │        3x          │ │
│ │ Блочный 128×128 │   0.6s      │        4x          │ │
│ │ + SIMD          │   0.2s      │       12x          │ │
│ └─────────────────┴─────────────┴─────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

### Prefetching и его влияние

#### Аппаратный prefetching
```
┌─────────────────────────────────────────────────────────┐
│                HARDWARE PREFETCHING                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Типы аппаратных prefetcher'ов:                          │
│                                                         │
│ 1. STREAM PREFETCHER                                    │
│    Обнаруживает последовательный доступ                │
│                                                         │
│    Паттерн доступа: 0x1000, 0x1040, 0x1080, 0x10C0...  │
│    Шаг: +64 байта (размер cache line)                  │
│                                                         │
│    ┌─────────────────────────────────────────────────┐ │
│    │ Обнаружен паттерн → Запрос следующих lines:     │ │
│    │ 0x1100, 0x1140, 0x1180, 0x11C0                 │ │
│    └─────────────────────────────────────────────────┘ │
│                                                         │
│ 2. STRIDE PREFETCHER                                    │
│    Обнаруживает доступ с постоянным шагом              │
│                                                         │
│    Паттерн: 0x1000, 0x1100, 0x1200, 0x1300...          │
│    Шаг: +256 байт                                      │
│                                                         │
│    ┌─────────────────────────────────────────────────┐ │
│    │ Предсказание: 0x1400, 0x1500, 0x1600...        │ │
│    └─────────────────────────────────────────────────┘ │
│                                                         │
│ 3. INDIRECT PREFETCHER                                  │
│    Обнаруживает pointer chasing                        │
│                                                         │
│    struct node *p = head;                              │
│    while (p) {                                         │
│        process(p->data);                               │
│        p = p->next;  ← предсказывается                 │
│    }                                                   │
│                                                         │
│ Эффективность prefetching:                              │
│                                                         │
│    Performance ↑                                       │
│         │                                              │
│    100% │    ████                                      │
│         │   ██████                                     │
│     80% │  ████████                                    │
│         │ ██████████                                   │
│     60% │████████████ ← Оптимальная зона              │
│         │████████████                                 │
│     40% │████████████                                 │
│         │████████████                                 │
│     20% │████████████                                 │
│         └────────────────────────► Prefetch Distance  │
│         0   2   4   6   8  10  12                     │
│                                                         │
│         ↑            ↑             ↑                   │
│    Too little   Optimal      Too much                  │
│    (late)       zone          (pollution)              │
└─────────────────────────────────────────────────────────┘
```

#### Программный prefetching
```
┌─────────────────────────────────────────────────────────┐
│               SOFTWARE PREFETCHING                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Компиляторные директивы и intrinsics:                   │
│                                                         │
│ // GCC builtin                                          │
│ __builtin_prefetch(addr, rw, locality);                 │
│ //                 ↑    ↑   ↑                          │
│ //              адрес  R/W  уровень                     │
│                                                         │
│ // Intel intrinsics                                     │
│ #include <xmmintrin.h>                                  │
│ _mm_prefetch(addr, _MM_HINT_T0);  // в L1              │
│ _mm_prefetch(addr, _MM_HINT_T1);  // в L2              │
│ _mm_prefetch(addr, _MM_HINT_T2);  // в L3              │
│ _mm_prefetch(addr, _MM_HINT_NTA); // non-temporal      │
│                                                         │
│ Пример оптимизации цикла:                               │
│                                                         │
│ // БЕЗ prefetch                                         │
│ for (i = 0; i < N; i++) {                              │
│     result += expensive_function(array[i]);            │
│ }                                                       │
│                                                         │
│ // С prefetch                                           │
│ for (i = 0; i < N; i++) {                              │
│     // Предзагружаем данные на несколько итераций      │
│     // вперед                                          │
│     if (i + PREFETCH_DISTANCE < N) {                   │
│         __builtin_prefetch(&array[i + PREFETCH_DISTANCE], │
│                           0, 3);                       │
│     }                                                  │
│     result += expensive_function(array[i]);            │
│ }                                                       │
│                                                         │
│ Оптимальное расстояние prefetch:                        │
│ ┌─────────────────────────────────────────────────────┐ │
│ │ PREFETCH_DISTANCE = Memory_Latency / Loop_Time      │ │
│ │                                                     │ │
│ │ Пример:                                             │ │
│ │ Memory_Latency = 300 cycles                         │ │
│ │ Loop_Time = 50 cycles                               │ │
│ │ PREFETCH_DISTANCE = 300/50 = 6 итераций            │ │
│ └─────────────────────────────────────────────────────┘ │
│                                                         │
│ Non-temporal prefetch для больших данных:               │
│                                                         │
│ // Данные используются только один раз                  │
│ for (i = 0; i < HUGE_N; i++) {                         │
│     _mm_prefetch(&huge_array[i + 8], _MM_HINT_NTA);    │
│     process_once(huge_array[i]);                       │
│ }                                                       │
│ // NTA = Not To All cache levels                       │
│ // Избегает загрязнения кэша                           │
└─────────────────────────────────────────────────────────┘
```

---

## 🔄 Глава 4: Когерентность кэша

### Проблема когерентности в многопроцессорных системах

```
┌─────────────────────────────────────────────────────────┐
│                CACHE COHERENCE PROBLEM                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Начальное состояние (X = 5 в памяти):                   │
│                                                         │
│ ┌─────────────┐              ┌─────────────┐            │
│ │   CPU 0     │              │   CPU 1     │            │
│ │ ┌─────────┐ │              │ ┌─────────┐ │            │
│ │ │ Cache 0 │ │              │ │ Cache 1 │ │            │
│ │ │    -    │ │              │ │    -    │ │            │
│ │ └─────────┘ │              │ └─────────┘ │            │
│ └─────────────┘              └─────────────┘            │
│       │                              │                 │
│       └──────────────┬───────────────┘                 │
│                      │                                 │
│              ┌─────────────┐                           │
│              │   Memory    │                           │
│              │    X = 5    │                           │
│              └─────────────┘                           │
│                                                         │
│ Шаг 1: CPU 0 читает X                                  │
│ ┌─────────────┐              ┌─────────────┐            │
│ │   CPU 0     │              │   CPU 1     │            │
│ │ ┌─────────┐ │              │ ┌─────────┐ │            │
│ │ │ Cache 0 │ │              │ │ Cache 1 │ │            │
│ │ │  X = 5  │ │              │ │    -    │ │            │
│ │ └─────────┘ │              │ └─────────┘ │            │
│ └─────────────┘              └─────────────┘            │
│                                                         │
│ Шаг 2: CPU 1 читает X                                  │
│ ┌─────────────┐              ┌─────────────┐            │
│ │   CPU 0     │              │   CPU 1     │            │
│ │ ┌─────────┐ │              │ ┌─────────┐ │            │
│ │ │ Cache 0 │ │              │ │ Cache 1 │ │            │
│ │ │  X = 5  │ │              │ │  X = 5  │ │            │
│ │ └─────────┘ │              │ └─────────┘ │            │
│ └─────────────┘              └─────────────┘            │
│                                                         │
│ Шаг 3: CPU 0 записывает X = 10                         │
│ ┌─────────────┐              ┌─────────────┐            │
│ │   CPU 0     │              │   CPU 1     │            │
│ │ ┌─────────┐ │              │ ┌─────────┐ │            │
│ │ │ Cache 0 │ │              │ │ Cache 1 │ │            │
│ │ │  X = 10 │ │              │ │  X = 5  │ │ ← ПРОБЛЕМА!│
│ │ └─────────┘ │              │ └─────────┘ │            │
│ └─────────────┘              └─────────────┘            │
│                                                         │
│ Результат: CPU 1 видит старое значение X = 5           │
│ Требуется протокол когерентности!                       │
└─────────────────────────────────────────────────────────┘
```

### Протокол MESI

```
┌─────────────────────────────────────────────────────────┐
│                     MESI PROTOCOL                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Четыре состояния cache line:                            │
│                                                         │
│ M = Modified   (Грязная, эксклюзивная)                  │
│ E = Exclusive  (Чистая, эксклюзивная)                   │
│ S = Shared     (Разделяемая между процессорами)         │
│ I = Invalid    (Недействительная)                       │
│                                                         │
│ Диаграмма состояний:                                    │
│                                                         │
│                    Local Read                           │
│             ┌──────────────────────────┐                │
│             ▼                          │                │
│    ┌─────────────┐ Local Write ┌─────────────┐          │
│    │      E      │─────────────►│      M      │          │
│    │ (Exclusive) │              │ (Modified)  │          │
│    └─────────────┘              └─────────────┘          │
│             │                          │                │
│             │ Remote Read             │ Eviction/      │
│             ▼                          │ Remote Read    │
│    ┌─────────────┐                     ▼                │
│    │      S      │◄────────────────────┘                │
│    │  (Shared)   │ Write-back                           │
│    └─────────────┘                                      │
│             │                                           │
│             │ Local/Remote Write                        │
│             ▼                                           │
│    ┌─────────────┐                                      │
│    │      I      │                                      │
│    │ (Invalid)   │                                      │
│    └─────────────┘                                      │
│                                                         │
│ Пример выполнения протокола:                            │
│                                                         │
│ Начальное состояние: все кэши пусты                     │
│ ┌─────────┬─────────┬─────────┬─────────────────────────┐│
│ │  CPU    │ Action  │ State   │      Описание          ││
│ ├─────────┼─────────┼─────────┼─────────────────────────┤│
│ │ CPU 0   │ Read X  │ E       │ Exclusive, loaded from ││
│ │         │         │         │ memory                 ││
│ ├─────────┼─────────┼─────────┼─────────────────────────┤│
│ │ CPU 1   │ Read X  │ S       │ CPU 0: E→S, CPU 1: S  ││
│ │         │         │         │ Shared между кэшами    ││
│ ├─────────┼─────────┼─────────┼─────────────────────────┤│
│ │ CPU 0   │ Write X │ M       │ CPU 0: S→M, CPU 1: S→I││
│ │         │         │         │ CPU 0 инвалидирует    ││
│ │         │         │         │ копию в CPU 1          ││
│ ├─────────┼─────────┼─────────┼─────────────────────────┤│
│ │ CPU 1   │ Read X  │ S       │ CPU 0: M→S (write-back)││
│ │         │         │         │ CPU 1: I→S, оба Shared││
│ └─────────┴─────────┴─────────┴─────────────────────────┘│
└─────────────────────────────────────────────────────────┘
```

### Протокол MOESI

```
┌─────────────────────────────────────────────────────────┐
│                    MOESI PROTOCOL                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Пять состояний (расширение MESI):                       │
│                                                         │
│ M = Modified   (Грязная, эксклюзивная)                  │
│ O = Owned      (Ответственная за поставку данных)       │
│ E = Exclusive  (Чистая, эксклюзивная)                   │
│ S = Shared     (Разделяемая)                            │
│ I = Invalid    (Недействительная)                       │
│                                                         │
│ Ключевое улучшение - состояние O (Owned):               │
│                                                         │
│ ┌─────────────────────────────────────────────────────┐ │
│ │ В MESI: при переходе M→S требуется write-back       │ │
│ │ в память, даже если другой CPU сразу считает        │ │
│ │                                                     │ │
│ │ В MOESI: M→O, данные остаются в кэше и поставляются│ │
│ │ напрямую другим процессорам (cache-to-cache        │ │
│ │ transfer)                                           │ │
│ └─────────────────────────────────────────────────────┘ │
│                                                         │
│ Пример оптимизации:                                     │
│                                                         │
│ ┌─────────┬─────────┬─────────┬─────────────────────────┐│
│ │ Step    │ CPU 0   │ CPU 1   │       Действие          ││
│ ├─────────┼─────────┼─────────┼─────────────────────────┤│
│ │ 1       │ Read X  │   -     │ CPU 0: I→E (from mem)  ││
│ ├─────────┼─────────┼─────────┼─────────────────────────┤│
│ │ 2       │Write X=10│   -     │ CPU 0: E→M             ││
│ ├─────────┼─────────┼─────────┼─────────────────────────┤│
│ │ 3       │ X=10(O) │ Read X  │ CPU 0: M→O, CPU 1: I→S ││
│ │         │         │ X=10(S) │ Cache-to-cache transfer ││
│ ├─────────┼─────────┼─────────┼─────────────────────────┤│
│ │ 4       │ X=10(O) │ Read X  │ CPU 0 поставляет       ││
│ │         │ X=10(S) │ X=10(S) │ данные напрямую        ││
│ └─────────┴─────────┴─────────┴─────────────────────────┘│
│                                                         │
│ Преимущества MOESI:                                     │
│ ✓ Меньше обращений к памяти                             │
│ ✓ Быстрее cache-to-cache transfers                      │
│ ✓ Лучшая производительность для read-heavy workloads    │
│                                                         │
│ Использует: AMD процессоры, некоторые ARM               │
└─────────────────────────────────────────────────────────┘
```

### False Sharing

```
┌─────────────────────────────────────────────────────────┐
│                     FALSE SHARING                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Проблема: разные переменные в одной cache line          │
│                                                         │
│ Память (одна cache line = 64 байта):                    │
│ ┌────────────────────────────────────────────────────┐  │
│ │  var_a   │  var_b   │  var_c   │  var_d   │ ...   │  │
│ │ (8 байт) │ (8 байт) │ (8 байт) │ (8 байт) │       │  │
│ └────────────────────────────────────────────────────┘  │
│                                                         │
│ Сценарий false sharing:                                 │
│                                                         │
│ Thread 1: постоянно пишет в var_a                       │
│ Thread 2: постоянно пишет в var_b                       │
│                                                         │
│ ┌─────────────┐              ┌─────────────┐            │
│ │   CPU 0     │              │   CPU 1     │            │
│ │ ┌─────────┐ │              │ ┌─────────┐ │            │
│ │ │ Cache 0 │ │              │ │ Cache 1 │ │            │
│ │ │ [a,b,c,d]│ │              │ │ [a,b,c,d]│ │            │
│ │ └─────────┘ │              │ └─────────┘ │            │
│ └─────────────┘              └─────────────┘            │
│                                                         │
│ Последовательность событий:                             │
│ 1. CPU 0 изменяет var_a → cache line переходит в M     │
│ 2. CPU 1 изменяет var_b → нужна копия cache line       │
│ 3. CPU 0 отдает cache line (M→S)                       │
│ 4. CPU 1 получает и изменяет (S→M)                     │
│ 5. CPU 0 опять хочет изменить var_a → miss!            │
│                                                         │
│ ▼ РЕЗУЛЬТАТ: постоянная инвалидация кэшей!              │
│                                                         │
│ Пример проблемного кода:                                │
│                                                         │
│ struct {                                                │
│     int counter1;  // Используется Thread 1            │
│     int counter2;  // Используется Thread 2            │
│ } shared_data;                                          │
│                                                         │
│ // Thread 1                                             │
│ while (running) {                                       │
│     shared_data.counter1++;  // False sharing!         │
│ }                                                       │
│                                                         │
│ // Thread 2                                             │
│ while (running) {                                       │
│     shared_data.counter2++;  // False sharing!         │
│ }                                                       │
│                                                         │
│ Измерение производительности:                           │
│ ┌─────────────────┬─────────────┬─────────────────────┐ │
│ │   Сценарий      │Operations/s │   Penalty           │ │
│ ├─────────────────┼─────────────┼─────────────────────┤ │
│ │ Без false shar. │ 100M ops/s  │        -            │ │
│ │ С false sharing │  10M ops/s  │       10x           │ │
│ │ С padding       │  95M ops/s  │       5%            │ │
│ └─────────────────┴─────────────┴─────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

#### Способы избежать False Sharing

```
┌─────────────────────────────────────────────────────────┐
│              AVOIDING FALSE SHARING                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ 1. PADDING (выравнивание по cache line)                │
│                                                         │
│ // Проблемный код                                       │
│ struct bad_struct {                                     │
│     int counter1;    // 4 байта                        │
│     int counter2;    // 4 байта - в той же cache line! │
│ };                                                      │
│                                                         │
│ // Решение с padding                                    │
│ struct good_struct {                                    │
│     int counter1;                                       │
│     char padding[60];  // Заполнение до 64 байт        │
│     int counter2;      // Теперь в отдельной cache line│
│ };                                                      │
│                                                         │
│ // Альтернатива с атрибутами компилятора                │
│ struct aligned_struct {                                 │
│     int counter1;                                       │
│     int counter2 __attribute__((aligned(64)));          │
│ };                                                      │
│                                                         │
│ 2. CACHE LINE SIZE MACRO                                │
│                                                         │
│ #define CACHE_LINE_SIZE 64                              │
│ #define CACHE_ALIGNED __attribute__((aligned(CACHE_LINE_SIZE)))│
│                                                         │
│ struct optimized_struct {                               │
│     int counter1 CACHE_ALIGNED;                         │
│     int counter2 CACHE_ALIGNED;                         │
│ };                                                      │
│                                                         │
│ 3. ОТДЕЛЬНЫЕ ПЕРЕМЕННЫЕ                                 │
│                                                         │
│ // Вместо структуры                                     │
│ static int counter1 CACHE_ALIGNED;                      │
│ static int counter2 CACHE_ALIGNED;                      │
│                                                         │
│ 4. THREAD-LOCAL STORAGE                                 │
│                                                         │
│ __thread int local_counter;  // Каждый поток свой      │
│                                                         │
│ // Периодическая синхронизация                          │
│ void sync_counters() {                                  │
│     atomic_add(&global_counter, local_counter);         │
│     local_counter = 0;                                  │
│ }                                                       │
│                                                         │
│ Визуализация решения:                                   │
│                                                         │
│ ДО (false sharing):                                     │
│ ┌──────────────────────────────────────────────────┐    │
│ │ counter1 │ counter2 │ other_data │ ...           │    │
│ │(Thread1) │(Thread2) │            │               │    │
│ └──────────────────────────────────────────────────┘    │
│   ← Cache Line 1 (64 байта) →                          │
│                                                         │
│ ПОСЛЕ (padding):                                        │
│ ┌──────────────────────────────────────────────────┐    │
│ │ counter1 │ padding...                            │    │
│ │(Thread1) │                                       │    │
│ └──────────────────────────────────────────────────┘    │
│   ← Cache Line 1 →                                      │
│ ┌──────────────────────────────────────────────────┐    │
│ │ counter2 │ other_data │ ...                      │    │
│ │(Thread2) │            │                          │    │
│ └──────────────────────────────────────────────────┘    │
│   ← Cache Line 2 →                                      │
└─────────────────────────────────────────────────────────┘