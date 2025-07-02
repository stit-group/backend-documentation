# 🚀 Блок 6: Продвинутые техники и оптимизации кэширования

---

## 📋 Содержание блока

**⏱️ Продолжительность:** 2-3 недели  
**🎯 Цель:** Освоить экспертные подходы к кэшированию и продвинутые алгоритмы  
**📊 Сложность:** Эксперт  

```
┌─────────────────────────────────────────────────────────────┐
│                    БЛОК 6: СТРУКТУРА                       │
├─────────────────────────────────────────────────────────────┤
│ Глава 1: Продвинутые алгоритмы замещения                   │
│ Глава 2: Вероятностные структуры данных                    │
│ Глава 3: Многоуровневое кэширование                        │
│ Глава 4: Мониторинг и профилирование                       │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔍 Глава 1: Продвинутые алгоритмы замещения

### 1.1 Ограничения классических алгоритмов

Классические алгоритмы LRU, LFU, FIFO имеют фундаментальные ограничения:

```
┌──────────────────────────────────────────────────────────────┐
│                   ПРОБЛЕМЫ LRU                               │
├──────────────────────────────────────────────────────────────┤
│ ❌ Плохо работает с циклическими паттернами доступа          │
│ ❌ Не учитывает частоту использования                        │
│ ❌ Может вытеснить "горячие" данные из-за временного всплеска│
│ ❌ O(1) операции требуют сложных структур данных            │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                   ПРОБЛЕМЫ LFU                               │
├──────────────────────────────────────────────────────────────┤
│ ❌ Новые элементы имеют низкий приоритет                     │
│ ❌ Старые счетчики "загрязняют" статистику                   │
│ ❌ Не адаптируется к изменению паттернов доступа            │
│ ❌ Сложность обновления счетчиков                           │
└──────────────────────────────────────────────────────────────┘
```

### 1.2 ARC (Adaptive Replacement Cache)

ARC объединяет преимущества LRU и LFU, адаптивно балансируя между ними.

```
                    ARC АРХИТЕКТУРА
    ┌─────────────────────────────────────────────────────┐
    │                 CACHE SIZE = c                      │
    ├──────────────────┬──────────────────────────────────┤
    │       T1         │               T2                 │
    │   (Recency)      │           (Frequency)            │
    │                  │                                  │
    │  ┌─────────────┐ │ ┌─────────────┐                 │
    │  │Recent Pages │ │ │Frequent Pages│                │
    │  │(LRU order)  │ │ │(LRU order)   │                │
    │  └─────────────┘ │ └─────────────┘                 │
    └──────────────────┴──────────────────────────────────┘
    ├─ Target: p ─────┤├── Target: c-p ──┤
    
    ┌─────────────────────────────────────────────────────┐
    │                GHOST LISTS                          │
    ├──────────────────┬──────────────────────────────────┤
    │       B1         │               B2                 │
    │ (Recently evicted│        (Frequently evicted       │
    │  from T1)        │         from T2)                 │
    └──────────────────┴──────────────────────────────────┘
```

**Принцип работы ARC:**

1. **Адаптивный параметр p** - определяет границу между T1 и T2
2. **Четыре списка:**
   - T1: недавно использованные страницы
   - T2: часто использованные страницы  
   - B1: "призраки" недавно вытесненных из T1
   - B2: "призраки" недавно вытесненных из T2

```python
class ARCCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.p = 0  # Адаптивный параметр
        
        # Четыре списка ARC
        self.T1 = OrderedDict()  # Recent cache
        self.T2 = OrderedDict()  # Frequent cache
        self.B1 = OrderedDict()  # Recent ghosts
        self.B2 = OrderedDict()  # Frequent ghosts
        
    def get(self, key):
        # HIT в T1 -> перемещаем в T2
        if key in self.T1:
            value = self.T1.pop(key)
            self.T2[key] = value
            return value
            
        # HIT в T2 -> перемещаем в конец T2
        if key in self.T2:
            value = self.T2.pop(key)
            self.T2[key] = value
            return value
            
        return None  # MISS
        
    def put(self, key, value):
        if key in self.B1:
            # Увеличиваем p (больше места для T1)
            self.p = min(self.capacity, self.p + max(1, len(self.B2)//len(self.B1)))
            self._replace(key)
            self.B1.pop(key)
            self.T2[key] = value
            
        elif key in self.B2:
            # Уменьшаем p (больше места для T2)
            self.p = max(0, self.p - max(1, len(self.B1)//len(self.B2)))
            self._replace(key)
            self.B2.pop(key)
            self.T2[key] = value
            
        else:
            # Новый элемент
            if len(self.T1) + len(self.T2) >= self.capacity:
                self._replace(key)
            
            # Добавляем в T1
            self.T1[key] = value
```

**Преимущества ARC:**

```
✅ Автоматически адаптируется к паттернам доступа
✅ Объединяет преимущества LRU и LFU
✅ Гарантированно не хуже LRU по производительности
✅ Эффективная реализация с O(1) операциями
✅ Отличная производительность на реальных workload'ах
```

### 1.3 CLOCK алгоритм и его вариации

CLOCK - это аппроксимация LRU с меньшими вычислительными затратами.

```
                    CLOCK АЛГОРИТМ
    ┌─────────────────────────────────────────────────────┐
    │                    Cache Slots                      │
    │                                                     │
    │     ┌───┐     ┌───┐     ┌───┐     ┌───┐            │
    │  ┌─→│ 1 │────→│ 0 │────→│ 1 │────→│ 0 │────┐       │
    │  │  └───┘     └───┘     └───┘     └───┘    │       │
    │  │   Page A    Page B    Page C    Page D   │       │
    │  │                                          │       │
    │  └──────────────────────────────────────────┘       │
    │              Clock Hand                              │
    └─────────────────────────────────────────────────────┘
    
    Reference Bit: 1 = Recently used, 0 = Not recently used
```

**Алгоритм CLOCK:**

```python
class ClockCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.slots = [None] * capacity
        self.reference_bits = [0] * capacity
        self.clock_hand = 0
        
    def _advance_clock(self):
        """Продвигаем стрелку часов"""
        while self.reference_bits[self.clock_hand] == 1:
            self.reference_bits[self.clock_hand] = 0
            self.clock_hand = (self.clock_hand + 1) % self.capacity
        
        # Нашли слот с reference_bit = 0
        victim_slot = self.clock_hand
        self.clock_hand = (self.clock_hand + 1) % self.capacity
        return victim_slot
        
    def get(self, key):
        for i, (stored_key, value) in enumerate(self.slots):
            if stored_key == key:
                self.reference_bits[i] = 1  # Устанавливаем бит
                return value
        return None
        
    def put(self, key, value):
        # Ищем свободный слот
        for i, slot in enumerate(self.slots):
            if slot is None:
                self.slots[i] = (key, value)
                self.reference_bits[i] = 1
                return
                
        # Все слоты заняты - используем CLOCK
        victim_slot = self._advance_clock()
        self.slots[victim_slot] = (key, value)
        self.reference_bits[victim_slot] = 1
```

**Enhanced CLOCK (CLOCK-Pro):**

```
┌─────────────────────────────────────────────────────────────┐
│                    CLOCK-Pro STRUCTURE                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  HOT PAGES          COLD PAGES           GHOST PAGES        │
│  ┌─────────┐       ┌─────────┐         ┌─────────┐         │
│  │Recently │       │Recently │         │Recently │         │
│  │ + Freq  │       │accessed │         │evicted  │         │
│  │accessed │       │only     │         │pages    │         │
│  └─────────┘       └─────────┘         └─────────┘         │
│       ↑                 ↑                   ↑               │
│   Resident           Resident          Non-resident        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 1.4 Machine Learning подходы

Современные подходы используют ML для предсказания будущих обращений.

```
                ML-BASED CACHE REPLACEMENT
    ┌─────────────────────────────────────────────────────┐
    │                 FEATURES                            │
    ├─────────────────────────────────────────────────────┤
    │ • Recency (время последнего доступа)               │
    │ • Frequency (частота обращений)                     │
    │ • Access pattern (паттерн доступа)                  │
    │ • Time of day (время суток)                        │
    │ • User behavior (поведение пользователя)           │
    │ • Content type (тип контента)                       │
    └─────────────────────────────────────────────────────┘
                            ↓
    ┌─────────────────────────────────────────────────────┐
    │              ML MODEL                               │
    │  ┌─────────────────────────────────────────────┐    │
    │  │    Neural Network / Random Forest          │    │
    │  │           / Gradient Boosting              │    │
    │  └─────────────────────────────────────────────┘    │
    └─────────────────────────────────────────────────────┘
                            ↓
    ┌─────────────────────────────────────────────────────┐
    │             PREDICTION                              │
    │  Probability of re-access within time window       │
    └─────────────────────────────────────────────────────┘
```

**Пример ML-кэша:**

```python
import numpy as np
from sklearn.ensemble import RandomForestRegressor

class MLCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache = {}
        self.model = RandomForestRegressor(n_estimators=100)
        self.access_history = []
        self.features = {}
        
    def _extract_features(self, key):
        """Извлекаем признаки для ключа"""
        now = time.time()
        
        if key not in self.features:
            self.features[key] = {
                'first_access': now,
                'last_access': now,
                'access_count': 0,
                'access_intervals': []
            }
        
        stats = self.features[key]
        
        return [
            now - stats['last_access'],  # recency
            stats['access_count'],       # frequency
            np.mean(stats['access_intervals']) if stats['access_intervals'] else 0,
            now % 86400,  # time of day
            len(self.cache),  # current cache size
        ]
        
    def _update_model(self):
        """Обновляем модель на основе истории"""
        if len(self.access_history) < 100:
            return
            
        # Подготавливаем данные для обучения
        X, y = [], []
        for record in self.access_history[-1000:]:
            features = record['features']
            reaccess_time = record['reaccess_time']
            X.append(features)
            y.append(reaccess_time)
            
        self.model.fit(X, y)
        
    def _predict_utility(self, key):
        """Предсказываем полезность элемента"""
        features = self._extract_features(key)
        
        try:
            prediction = self.model.predict([features])[0]
            return 1.0 / (prediction + 1)  # Чем меньше время до переобращения, тем выше utility
        except:
            # Fallback к простой эвристике
            return self.features[key]['access_count']
```

---

## 📊 Глава 2: Оптимизация с помощью вероятностных структур

### 2.1 Bloom фильтры для кэширования

Bloom фильтры позволяют быстро проверить отсутствие элемента в кэше.

```
                    BLOOM FILTER STRUCTURE
    ┌─────────────────────────────────────────────────────┐
    │            Bit Array (m bits)                      │
    │  0  1  0  0  1  0  1  1  0  0  1  0  0  1  0  1    │
    │  ↑     ↑        ↑     ↑        ↑           ↑       │
    │  │     │        │     │        │           │       │
    └──┼─────┼────────┼─────┼────────┼───────────┼───────┘
       │     │        │     │        │           │
    ┌──┴─────┴────────┴─────┴────────┴───────────┴───────┐
    │              Hash Functions                         │
    │  h1(x)  h2(x)  h3(x)  h4(x)  h5(x)  h6(x)         │
    └─────────────────────────────────────────────────────┘
                            ↑
                      Input Element
```

**Применение в кэшировании:**

```python
class BloomFilterCache:
    def __init__(self, capacity, bloom_size=10000, num_hash=7):
        self.cache = {}
        self.capacity = capacity
        
        # Bloom filter для быстрой проверки
        self.bloom_bits = [0] * bloom_size
        self.bloom_size = bloom_size
        self.num_hash = num_hash
        
    def _hash_functions(self, key):
        """Генерируем несколько хеш-функций"""
        hashes = []
        key_bytes = str(key).encode()
        
        for i in range(self.num_hash):
            hash_val = hash(key_bytes + str(i).encode())
            hashes.append(abs(hash_val) % self.bloom_size)
            
        return hashes
        
    def _add_to_bloom(self, key):
        """Добавляем ключ в bloom filter"""
        for pos in self._hash_functions(key):
            self.bloom_bits[pos] = 1
            
    def _might_exist(self, key):
        """Проверяем, может ли ключ существовать"""
        for pos in self._hash_functions(key):
            if self.bloom_bits[pos] == 0:
                return False  # Точно НЕ существует
        return True  # Возможно существует
        
    def get(self, key):
        # Быстрая проверка с помощью Bloom filter
        if not self._might_exist(key):
            return None  # Точно нет в кэше
            
        # Медленная проверка в реальном кэше
        return self.cache.get(key)
        
    def put(self, key, value):
        self.cache[key] = value
        self._add_to_bloom(key)
        
        if len(self.cache) > self.capacity:
            # Eviction logic here
            pass
```

**Преимущества Bloom фильтров:**

```
┌─────────────────────────────────────────────────────────────┐
│                    BLOOM FILTER BENEFITS                   │
├─────────────────────────────────────────────────────────────┤
│ ✅ O(1) проверка наличия элемента                          │
│ ✅ Экономия памяти (1-2 бита на элемент)                   │
│ ✅ Нет ложноотрицательных результатов                       │
│ ✅ Быстрое исключение отсутствующих ключей                  │
│ ✅ Подходит для распределенных систем                       │
├─────────────────────────────────────────────────────────────┤
│ ⚠️  Возможны ложноположительные результаты                 │
│ ⚠️  Нельзя удалять элементы (нужен Counting Bloom Filter)  │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Count-Min Sketch для частотного анализа

Count-Min Sketch оценивает частоту элементов с минимальным использованием памяти.

```
                COUNT-MIN SKETCH STRUCTURE
    ┌─────────────────────────────────────────────────────┐
    │  Hash Function 1                                    │
    │  [5] [2] [8] [1] [3] [7] [4] [6] [9] [0] [2]       │
    ├─────────────────────────────────────────────────────┤
    │  Hash Function 2                                    │
    │  [3] [7] [1] [9] [5] [2] [8] [4] [6] [0] [1]       │
    ├─────────────────────────────────────────────────────┤
    │  Hash Function 3                                    │
    │  [8] [4] [6] [2] [7] [5] [1] [9] [3] [0] [4]       │
    ├─────────────────────────────────────────────────────┤
    │  Hash Function 4                                    │
    │  [2] [9] [3] [7] [1] [8] [5] [0] [4] [6] [3]       │
    └─────────────────────────────────────────────────────┘
    
    Query frequency of element X:
    min(count[h1(X)], count[h2(X)], count[h3(X)], count[h4(X)])
```

**Применение для LFU кэширования:**

```python
class CountMinSketchLFU:
    def __init__(self, capacity, width=1000, depth=4):
        self.capacity = capacity
        self.cache = {}
        
        # Count-Min Sketch параметры
        self.width = width
        self.depth = depth
        self.sketch = [[0] * width for _ in range(depth)]
        
        # Генерируем hash функции
        self.hash_funcs = []
        for i in range(depth):
            a = random.randint(1, 1000000)
            b = random.randint(0, 1000000)
            self.hash_funcs.append(lambda x, a=a, b=b: (a * hash(x) + b) % width)
            
    def _increment_frequency(self, key):
        """Увеличиваем счетчик частоты для ключа"""
        for i, hash_func in enumerate(self.hash_funcs):
            pos = hash_func(key)
            self.sketch[i][pos] += 1
            
    def _get_frequency(self, key):
        """Получаем оценку частоты ключа"""
        frequencies = []
        for i, hash_func in enumerate(self.hash_funcs):
            pos = hash_func(key)
            frequencies.append(self.sketch[i][pos])
        return min(frequencies)  # Count-Min оценка
        
    def get(self, key):
        if key in self.cache:
            self._increment_frequency(key)
            return self.cache[key]
        return None
        
    def put(self, key, value):
        if key in self.cache:
            self.cache[key] = value
            self._increment_frequency(key)
            return
            
        if len(self.cache) >= self.capacity:
            # Находим элемент с минимальной частотой
            min_freq = float('inf')
            victim_key = None
            
            for cache_key in self.cache:
                freq = self._get_frequency(cache_key)
                if freq < min_freq:
                    min_freq = freq
                    victim_key = cache_key
                    
            del self.cache[victim_key]
            
        self.cache[key] = value
        self._increment_frequency(key)
```

### 2.3 HyperLogLog для кардинальности

HyperLogLog оценивает количество уникальных элементов с фиксированным использованием памяти.

```
                HYPERLOGLOG STRUCTURE
    ┌─────────────────────────────────────────────────────┐
    │  Hash(element) = 101011001110010101...              │
    │                  └─┬─┘└───────┬─────────┘          │
    │                 bucket     leading zeros            │
    │                (b bits)    (remaining bits)         │
    └─────────────────────────────────────────────────────┘
    
    ┌─────────────────────────────────────────────────────┐
    │              BUCKET ARRAY                           │
    │  Bucket 0: max_zeros = 3                           │
    │  Bucket 1: max_zeros = 5                           │
    │  Bucket 2: max_zeros = 2                           │
    │  ...                                                │
    │  Bucket 2^b-1: max_zeros = 4                       │
    └─────────────────────────────────────────────────────┘
    
    Estimated cardinality = 2^b * harmonic_mean(2^max_zeros)
```

**Применение для мониторинга кэша:**

```python
class CacheWithCardinality:
    def __init__(self, capacity):
        self.cache = {}
        self.capacity = capacity
        self.hll = HyperLogLog(precision=10)  # 2^10 = 1024 buckets
        
    def get(self, key):
        if key in self.cache:
            return self.cache[key]
        return None
        
    def put(self, key, value):
        # Добавляем в HyperLogLog для подсчета уникальных ключей
        self.hll.add(key)
        
        self.cache[key] = value
        
        if len(self.cache) > self.capacity:
            # Eviction logic
            pass
            
    def get_unique_keys_estimate(self):
        """Оценка количества уникальных ключей за все время"""
        return self.hll.cardinality()
        
    def get_cache_diversity(self):
        """Оценка разнообразия данных в кэше"""
        total_unique = self.hll.cardinality()
        current_size = len(self.cache)
        return current_size / total_unique if total_unique > 0 else 0
```

---

## 🏗️ Глава 3: Многоуровневое кэширование

### 3.1 Иерархическая архитектура

```
                MULTI-LEVEL CACHE HIERARCHY
    ┌─────────────────────────────────────────────────────┐
    │                   L1 CACHE                          │
    │  ┌─────────────────────────────────────────────┐    │
    │  │ Size: 64KB | Latency: 1-2 cycles          │    │
    │  │ Algorithm: LRU | Hit Rate: 95%             │    │
    │  └─────────────────────────────────────────────┘    │
    └─────────────────────────────────────────────────────┘
                            ↓ MISS
    ┌─────────────────────────────────────────────────────┐
    │                   L2 CACHE                          │
    │  ┌─────────────────────────────────────────────┐    │
    │  │ Size: 2MB | Latency: 10-20 cycles          │    │
    │  │ Algorithm: ARC | Hit Rate: 85%              │    │
    │  └─────────────────────────────────────────────┘    │
    └─────────────────────────────────────────────────────┘
                            ↓ MISS
    ┌─────────────────────────────────────────────────────┐
    │                   L3 CACHE                          │
    │  ┌─────────────────────────────────────────────┐    │
    │  │ Size: 100MB | Latency: 100 cycles          │    │
    │  │ Algorithm: CLOCK | Hit Rate: 70%            │    │
    │  └─────────────────────────────────────────────┘    │
    └─────────────────────────────────────────────────────┘
                            ↓ MISS
    ┌─────────────────────────────────────────────────────┐
    │                  MAIN MEMORY                        │
    │  ┌─────────────────────────────────────────────┐    │
    │  │ Size: Unlimited | Latency: 1000+ cycles    │    │
    │  └─────────────────────────────────────────────┘    │
    └─────────────────────────────────────────────────────┘
```

### 3.2 Инклюзивное vs Эксклюзивное кэширование

```
                    INCLUSIVE CACHING
    ┌─────────────────────────────────────────────────────┐
    │  L1: [A] [B] [C] [D]                               │
    │  L2: [A] [B] [C] [D] [E] [F] [G] [H]               │
    │  L3: [A] [B] [C] [D] [E] [F] [G] [H] [I] [J] ...   │
    └─────────────────────────────────────────────────────┘
    
    ✅ Простая инвалидация (снизу вверх)
    ✅ Гарантированная консистентность
    ❌ Дублирование данных
    ❌ Неэффективное использование памяти
    
    
                    EXCLUSIVE CACHING
    ┌─────────────────────────────────────────────────────┐
    │  L1: [A] [B] [C] [D]                               │
    │  L2: [E] [F] [G] [H]                               │
    │  L3: [I] [J] [K] [L] [M] [N] [O] [P]               │
    └─────────────────────────────────────────────────────┘
    
    ✅ Максимальное использование памяти
    ✅ Больше уникальных данных
    ❌ Сложная миграция между уровнями
    ❌ Более сложная реализация
```

### 3.3 Реализация многоуровневого кэша

```python
class MultiLevelCache:
    def __init__(self, levels_config):
        """
        levels_config = [
            {'size': 100, 'algorithm': 'LRU', 'latency': 1},
            {'size': 1000, 'algorithm': 'ARC', 'latency': 10},
            {'size': 10000, 'algorithm': 'CLOCK', 'latency': 100},
        ]
        """
        self.levels = []
        self.stats = {'hits': [0] * len(levels_config), 'misses': [0] * len(levels_config)}
        
        for config in levels_config:
            if config['algorithm'] == 'LRU':
                cache = LRUCache(config['size'])
            elif config['algorithm'] == 'ARC':
                cache = ARCCache(config['size'])
            elif config['algorithm'] == 'CLOCK':
                cache = ClockCache(config['size'])
            else:
                raise ValueError(f"Unknown algorithm: {config['algorithm']}")
                
            self.levels.append({
                'cache': cache,
                'latency': config['latency'],
                'algorithm': config['algorithm']
            })
            
    def get(self, key):
        total_latency = 0
        
        # Проходим по уровням от быстрого к медленному
        for i, level in enumerate(self.levels):
            total_latency += level['latency']
            value = level['cache'].get(key)
            
            if value is not None:
                # HIT на уровне i
                self.stats['hits'][i] += 1
                
                # Продвигаем элемент на верхние уровни (inclusive policy)
                for j in range(i):
                    self.levels[j]['cache'].put(key, value)
                    
                return {
                    'value': value,
                    'latency': total_latency,
                    'hit_level': i
                }
            else:
                self.stats['misses'][i] += 1
                
        return None  # MISS на всех уровнях
        
    def put(self, key, value):
        # Добавляем на все уровни (inclusive policy)
        for level in self.levels:
            level['cache'].put(key, value)
            
    def get_statistics(self):
        """Статистика по уровням"""
        stats = {}
        for i, level in enumerate(self.levels):
            total_requests = self.stats['hits'][i] + self.stats['misses'][i]
            hit_rate = self.stats['hits'][i] / total_requests if total_requests > 0 else 0
            
            stats[f'L{i+1}'] = {
                'algorithm': level['algorithm'],
                'hit_rate': hit_rate,
                'hits': self.stats['hits'][i],
                'misses': self.stats['misses'][i],
                'latency': level['latency']
            }
            
        return stats
```

### 3.4 Оптимизация между уровнями

```python
class SmartMultiLevelCache:
    def __init__(self, levels_config):
        self.levels = self._init_levels(levels_config)
        self.prefetch_queue = Queue()
        self.access_patterns = defaultdict(list)
        
    def _analyze_access_pattern(self, key):
        """Анализируем паттерн доступа для предсказания"""
        now = time.time()
        self.access_patterns[key].append(now)
        
        # Оставляем только последние 10 обращений
        if len(self.access_patterns[key]) > 10:
            self.access_patterns[key] = self.access_patterns[key][-10:]
            
    def _predict_next_access(self, key):
        """Предсказываем следующее обращение"""
        accesses = self.access_patterns[key]
        if len(accesses) < 2:
            return None
            
        intervals = [accesses[i] - accesses[i-1] for i in range(1, len(accesses))]
        avg_interval = sum(intervals) / len(intervals)
        
        return accesses[-1] + avg_interval
        
    def _should_prefetch(self, key, current_level):
        """Решаем, нужно ли делать prefetch"""
        next_access = self._predict_next_access(key)
        if next_access is None:
            return False
            
        time_to_next = next_access - time.time()
        
        # Если следующее обращение скоро, и элемент не в быстрых уровнях
        return 0 < time_to_next < 60 and current_level > 0
        
    def get(self, key):
        self._analyze_access_pattern(key)
        
        for i, level in enumerate(self.levels):
            value = level['cache'].get(key)
            if value is not None:
                # Умный prefetch на верхние уровни
                if self._should_prefetch(key, i):
                    for j in range(i):
                        self.levels[j]['cache'].put(key, value)
                        
                return value
                
        return None
```

---

## 📈 Глава 4: Мониторинг и профилирование

### 4.1 Ключевые метрики кэша

```
                    CACHE PERFORMANCE METRICS
    ┌─────────────────────────────────────────────────────┐
    │                 BASIC METRICS                       │
    ├─────────────────────────────────────────────────────┤
    │ Hit Rate = Hits / (Hits + Misses)                   │
    │ Miss Rate = Misses / (Hits + Misses)                │
    │ Hit Ratio = Hit Rate * 100%                         │
    └─────────────────────────────────────────────────────┘
    
    ┌─────────────────────────────────────────────────────┐
    │               LATENCY METRICS                       │
    ├─────────────────────────────────────────────────────┤
    │ Average Hit Latency                                 │
    │ Average Miss Latency                                │
    │ P50, P95, P99 Response Times                        │
    │ Throughput (Operations/Second)                      │
    └─────────────────────────────────────────────────────┘
    
    ┌─────────────────────────────────────────────────────┐
    │              EFFICIENCY METRICS                     │
    ├─────────────────────────────────────────────────────┤
    │ Memory Utilization = Used / Allocated               │
    │ Eviction Rate = Evictions / Time                    │
    │ Write/Read Ratio                                    │
    │ Cost per Operation                                  │
    └─────────────────────────────────────────────────────┘
```

### 4.2 Продвинутые метрики

```python
class AdvancedCacheMetrics:
    def __init__(self):
        self.metrics = {
            'hits': 0,
            'misses': 0,
            'evictions': 0,
            'writes': 0,
            'hit_latencies': [],
            'miss_latencies': [],
            'size_over_time': [],
            'access_patterns': defaultdict(list)
        }
        
        # Sliding window для real-time метрик
        self.window_size = 1000
        self.recent_operations = deque(maxlen=self.window_size)
        
    def record_hit(self, key, latency):
        """Записываем информацию о cache hit"""
        now = time.time()
        self.metrics['hits'] += 1
        self.metrics['hit_latencies'].append(latency)
        self.metrics['access_patterns'][key].append(now)
        
        self.recent_operations.append({
            'type': 'hit',
            'key': key,
            'latency': latency,
            'timestamp': now
        })
        
    def record_miss(self, key, latency):
        """Записываем информацию о cache miss"""
        now = time.time()
        self.metrics['misses'] += 1
        self.metrics['miss_latencies'].append(latency)
        
        self.recent_operations.append({
            'type': 'miss',
            'key': key,
            'latency': latency,
            'timestamp': now
        })
        
    def get_hit_rate(self, window_seconds=None):
        """Вычисляем hit rate за заданное окно времени"""
        if window_seconds is None:
            total = self.metrics['hits'] + self.metrics['misses']
            return self.metrics['hits'] / total if total > 0 else 0
            
        # Hit rate за последние window_seconds секунд
        now = time.time()
        cutoff = now - window_seconds
        
        hits = misses = 0
        for op in self.recent_operations:
            if op['timestamp'] >= cutoff:
                if op['type'] == 'hit':
                    hits += 1
                else:
                    misses += 1
                    
        total = hits + misses
        return hits / total if total > 0 else 0
        
    def get_temporal_locality(self):
        """Оценка временной локальности"""
        localities = []
        
        for key, accesses in self.metrics['access_patterns'].items():
            if len(accesses) < 2:
                continue
                
            intervals = [accesses[i] - accesses[i-1] for i in range(1, len(accesses))]
            avg_interval = sum(intervals) / len(intervals)
            
            # Чем меньше интервал, тем выше локальность
            locality = 1.0 / (1.0 + avg_interval)
            localities.append(locality)
            
        return sum(localities) / len(localities) if localities else 0
        
    def get_hot_keys(self, top_n=10):
        """Получаем самые популярные ключи"""
        key_counts = {key: len(accesses) 
                     for key, accesses in self.metrics['access_patterns'].items()}
        
        return sorted(key_counts.items(), key=lambda x: x[1], reverse=True)[:top_n]
        
    def get_performance_report(self):
        """Генерируем полный отчет о производительности"""
        total_ops = self.metrics['hits'] + self.metrics['misses']
        
        if total_ops == 0:
            return "No operations recorded"
            
        hit_rate = self.metrics['hits'] / total_ops
        
        # Latency статистика
        all_latencies = self.metrics['hit_latencies'] + self.metrics['miss_latencies']
        hit_latencies = self.metrics['hit_latencies']
        miss_latencies = self.metrics['miss_latencies']
        
        report = f"""
╔══════════════════════════════════════════════════════════════╗
║                    CACHE PERFORMANCE REPORT                 ║
╠══════════════════════════════════════════════════════════════╣
║ Hit Rate:           {hit_rate:.2%}                            ║
║ Total Operations:   {total_ops:,}                            ║
║ Hits:              {self.metrics['hits']:,}                  ║
║ Misses:            {self.metrics['misses']:,}                ║
║ Evictions:         {self.metrics['evictions']:,}             ║
╠══════════════════════════════════════════════════════════════╣
║                      LATENCY ANALYSIS                       ║
╠══════════════════════════════════════════════════════════════╣
║ Avg Hit Latency:   {np.mean(hit_latencies):.2f}ms           ║
║ Avg Miss Latency:  {np.mean(miss_latencies):.2f}ms          ║
║ P95 Latency:       {np.percentile(all_latencies, 95):.2f}ms ║
║ P99 Latency:       {np.percentile(all_latencies, 99):.2f}ms ║
╠══════════════════════════════════════════════════════════════╣
║                      ACCESS PATTERNS                        ║
╠══════════════════════════════════════════════════════════════╣
║ Temporal Locality: {self.get_temporal_locality():.3f}       ║
║ Unique Keys:       {len(self.metrics['access_patterns']):,}  ║
║ Recent Hit Rate:   {self.get_hit_rate(300):.2%} (5min)      ║
╚══════════════════════════════════════════════════════════════╝
        """
        
        return report
```

### 4.3 Система мониторинга в реальном времени

```python
class RealTimeCacheMonitor:
    def __init__(self, cache, update_interval=1.0):
        self.cache = cache
        self.update_interval = update_interval
        self.metrics = AdvancedCacheMetrics()
        
        # Prometheus metrics (если используется)
        try:
            from prometheus_client import Counter, Histogram, Gauge
            self.hit_counter = Counter('cache_hits_total', 'Total cache hits')
            self.miss_counter = Counter('cache_misses_total', 'Total cache misses')
            self.latency_histogram = Histogram('cache_latency_seconds', 'Cache operation latency')
            self.size_gauge = Gauge('cache_size_current', 'Current cache size')
        except ImportError:
            pass
            
        # Запускаем мониторинг в отдельном потоке
        self.monitoring_thread = threading.Thread(target=self._monitor_loop, daemon=True)
        self.monitoring_thread.start()
        
    def _monitor_loop(self):
        """Основной цикл мониторинга"""
        while True:
            try:
                self._collect_metrics()
                self._detect_anomalies()
                time.sleep(self.update_interval)
            except Exception as e:
                print(f"Monitoring error: {e}")
                
    def _collect_metrics(self):
        """Собираем текущие метрики"""
        if hasattr(self.cache, 'get_stats'):
            stats = self.cache.get_stats()
            
            # Обновляем Prometheus метрики
            if hasattr(self, 'size_gauge'):
                self.size_gauge.set(len(self.cache))
                
    def _detect_anomalies(self):
        """Обнаруживаем аномалии в работе кэша"""
        current_hit_rate = self.metrics.get_hit_rate(window_seconds=60)
        
        # Аномально низкий hit rate
        if current_hit_rate < 0.5:
            self._alert(f"Low hit rate detected: {current_hit_rate:.2%}")
            
        # Высокая латентность
        recent_latencies = [op['latency'] for op in self.metrics.recent_operations 
                           if op['timestamp'] > time.time() - 60]
        
        if recent_latencies:
            avg_latency = sum(recent_latencies) / len(recent_latencies)
            if avg_latency > 100:  # 100ms threshold
                self._alert(f"High latency detected: {avg_latency:.2f}ms")
                
    def _alert(self, message):
        """Отправляем алерт"""
        timestamp = time.strftime('%Y-%m-%d %H:%M:%S')
        print(f"[ALERT {timestamp}] {message}")
        
        # Здесь можно добавить отправку в Slack, email, etc.
```

### 4.4 A/B тестирование кэширующих стратегий

```python
class CacheABTest:
    def __init__(self, strategy_a, strategy_b, traffic_split=0.5):
        self.strategy_a = strategy_a
        self.strategy_b = strategy_b
        self.traffic_split = traffic_split
        
        self.metrics_a = AdvancedCacheMetrics()
        self.metrics_b = AdvancedCacheMetrics()
        
    def get(self, key):
        """Направляем трафик на разные стратегии"""
        use_strategy_a = hash(key) % 100 < (self.traffic_split * 100)
        
        start_time = time.time()
        
        if use_strategy_a:
            result = self.strategy_a.get(key)
            latency = (time.time() - start_time) * 1000
            
            if result is not None:
                self.metrics_a.record_hit(key, latency)
            else:
                self.metrics_a.record_miss(key, latency)
        else:
            result = self.strategy_b.get(key)
            latency = (time.time() - start_time) * 1000
            
            if result is not None:
                self.metrics_b.record_hit(key, latency)
            else:
                self.metrics_b.record_miss(key, latency)
                
        return result
        
    def get_test_results(self):
        """Сравниваем результаты A/B теста"""
        hit_rate_a = self.metrics_a.get_hit_rate()
        hit_rate_b = self.metrics_b.get_hit_rate()
        
        latency_a = np.mean(self.metrics_a.metrics['hit_latencies'] + 
                           self.metrics_a.metrics['miss_latencies'])
        latency_b = np.mean(self.metrics_b.metrics['hit_latencies'] + 
                           self.metrics_b.metrics['miss_latencies'])
        
        improvement = (hit_rate_b - hit_rate_a) / hit_rate_a * 100
        
        return {
            'strategy_a': {
                'hit_rate': hit_rate_a,
                'avg_latency': latency_a,
                'total_ops': len(self.metrics_a.recent_operations)
            },
            'strategy_b': {
                'hit_rate': hit_rate_b,
                'avg_latency': latency_b,
                'total_ops': len(self.metrics_b.recent_operations)
            },
            'improvement': f"{improvement:+.2f}%",
            'confidence': self._calculate_confidence(hit_rate_a, hit_rate_b)
        }
        
    def _calculate_confidence(self, rate_a, rate_b):
        """Вычисляем статистическую значимость различий"""
        # Упрощенный расчет z-score для разности пропорций
        n_a = len(self.metrics_a.recent_operations)
        n_b = len(self.metrics_b.recent_operations)
        
        if n_a < 30 or n_b < 30:
            return "Insufficient data"
            
        pooled_rate = (rate_a * n_a + rate_b * n_b) / (n_a + n_b)
        se = math.sqrt(pooled_rate * (1 - pooled_rate) * (1/n_a + 1/n_b))
        
        if se == 0:
            return "Unable to calculate"
            
        z_score = abs(rate_b - rate_a) / se
        
        if z_score > 2.58:
            return "99% confident"
        elif z_score > 1.96:
            return "95% confident"
        elif z_score > 1.65:
            return "90% confident"
        else:
            return "Not significant"
```

---

## 🛠️ Практические задания

### Задание 1: Реализация ARC алгоритма

```python
# Реализуйте полную версию ARC кэша с детальным логированием
class DetailedARCCache:
    def __init__(self, capacity):
        # Ваша реализация здесь
        pass
        
    def get(self, key):
        # Логируйте все операции перемещения между списками
        pass
        
    def put(self, key, value):
        # Реализуйте адаптивную логику изменения параметра p
        pass
        
    def get_debug_info(self):
        # Верните детальную информацию о состоянии всех списков
        pass
```

### Задание 2: Bloom Filter оптимизация

```python
# Создайте систему кэширования с Bloom Filter для минимизации
# обращений к медленному storage
class OptimizedBloomCache:
    def __init__(self, cache_size, bloom_size, slow_storage):
        # Интегрируйте Bloom Filter с реальным кэшем
        pass
        
    def adaptive_bloom_resize(self):
        # Реализуйте адаптивное изменение размера Bloom Filter
        # на основе false positive rate
        pass
```

### Задание 3: Многоуровневая система с ML

```python
# Создайте систему кэширования с машинным обучением для предсказания
class MLMultiLevelCache:
    def __init__(self, levels):
        # Интегрируйте ML модель для предсказания optimal placement
        pass
        
    def train_placement_model(self):
        # Обучите модель на истории доступа для оптимального размещения
        pass
        
    def predict_optimal_level(self, key, access_pattern):
        # Предскажите оптимальный уровень для размещения данных
        pass
```

### Задание 4: Система мониторинга

```python
# Создайте dashboard для real-time мониторинга кэша
class CacheDashboard:
    def __init__(self):
        # Реализуйте веб-интерфейс для мониторинга
        pass
        
    def generate_performance_charts(self):
        # Создайте графики hit rate, latency, throughput
        pass
        
    def detect_performance_degradation(self):
        # Автоматическое обнаружение проблем с производительностью
        pass
```

---

## ✅ Результаты блока

После изучения этого блока вы будете уметь:

- ✅ Реализовывать и оптимизировать продвинутые алгоритмы замещения
- ✅ Использовать вероятностные структуры данных для оптимизации кэша
- ✅ Проектировать и реализовывать многоуровневые кэширующие системы
- ✅ Создавать системы мониторинга и профилирования кэша
- ✅ Проводить A/B тестирование различных стратегий кэширования
- ✅ Применять машинное обучение для оптимизации кэширования

## 📚 Дополнительные материалы

- **Research Papers:**
  - "ARC: A Self-Tuning, Low Overhead Replacement Cache" (Megiddo & Modha)
  - "CLOCK-Pro: An Effective Improvement of the CLOCK Replacement" 
  - "Learning Memory Access Patterns" (Google Research)

- **Инструменты:**
  - Intel VTune Profiler для анализа cache miss'ов
  - Perf для профилирования кэша Linux
  - Redis modules для экспериментов
  - Prometheus + Grafana для мониторинга

---

## 🎯 Переход к следующему блоку

Этот блок заложил основы экспертного уровня кэширования. В следующем блоке мы изучим применение этих техник в современных архитектурах: микросервисах, облачных платформах и AI/ML системах.