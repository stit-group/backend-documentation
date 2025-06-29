# Собеседование по Golang: 25 глубоких вопросов для Senior разработчика

Данные вопросы направлены на проверку глубокого понимания Go, его архитектуры, многопоточности, планировщика и принципов чистой архитектуры. Каждый вопрос содержит подсказки для более глубокого изучения.

## 🔥 Базовые структуры данных и их тонкости

### 1. В чем принципиальная разница между array, slice и map с точки зрения внутреннего устройства?

**Ожидаемый ответ:**
- **Array**: фиксированного размера, значения хранятся подряд в памяти, передается по значению
- **Slice**: дескриптор содержит указатель на underlying array, длину и capacity. При append может происходить реаллокация
- **Map**: hash table с bucket'ами, не thread-safe, при чтении из nil map - zero value, при записи - panic

**Подсказки для углубления:**
- Что происходит при `copy(dst, src)` для slice?
- Как работает `append` при превышении capacity? Какова стратегия роста?
- Почему `map[string]int{}` безопасно для чтения, но небезопасно для записи из разных goroutine?

### 2. Объясните zero values в Go. Почему это важно для архитектуры?

**Ожидаемый ответ:**
Zero values позволяют создавать полезные структуры без явной инициализации:
```go
var mu sync.Mutex // готов к использованию
var buf bytes.Buffer // готов к использованию
var users []User // готов к использованию с append
```

**Подсказки для углубления:**
- Как спроектировать API, чтобы zero value был полезен?
- Что будет, если обратиться к методу на nil pointer?
- Приведите примеры типов из стандартной библиотеки, где zero value полезен

### 3. Что такое escape analysis? Как он влияет на производительность?

**Ожидаемый ответ:**
Escape analysis определяет, должна ли переменная размещаться в stack или heap:
- Stack: быстрое выделение/освобождение, локальность данных
- Heap: требует garbage collection, медленнее

**Подсказки для углубления:**
- Как проверить, что переменная "ускользает" на heap? (`go build -gcflags="-m"`)
- Когда interface{} заставляет значение попасть на heap?
- Как pointer возвращаемый из функции влияет на escape analysis?

## 🚀 Многопоточность и планировщик

### 4. Объясните модель планировщика Go (G-M-P). Почему именно такая архитектура?

**Ожидаемый ответ:**
- **G (Goroutine)**: легковесная задача (~2KB stack)
- **M (Machine)**: OS thread
- **P (Processor)**: логический процессор, имеет local run queue

Планировщик решает проблемы:
- Work stealing между P
- Отвязка от kernel scheduler
- Кооперативная многозадачность с preemption (с Go 1.14)

**Подсказки для углубления:**
- Что происходит при системном вызове? Как P отвязывается от M?
- Как работает work stealing? Когда P крадет работу у другого P?
- Что такое sysmon goroutine и зачем она нужна?
- Как GOMAXPROCS влияет на количество P?

### 5. Что такое race condition? Покажите пример и объясните, как его обнаружить и исправить.

**Ожидаемый ответ:**
```go
// Плохо - race condition
var counter int
go func() { counter++ }()
go func() { counter++ }()

// Хорошо - с мьютексом
var mu sync.Mutex
var counter int
go func() { mu.Lock(); counter++; mu.Unlock() }()

// Хорошо - атомарные операции
var counter int64
go func() { atomic.AddInt64(&counter, 1) }()

// Обнаружение: go run -race main.go
```

**Подсказки для углубления:**
- В чем разница между data race и race condition?
- Когда использовать `sync/atomic` вместо mutex?
- Как race detector работает внутри? Почему он увеличивает потребление памяти?
- Что такое happens-before relationship?

### 6. Объясните все способы синхронизации в Go и когда какой использовать.

**Ожидаемый ответ:**
- **Channels**: "Don't communicate by sharing memory; share memory by communicating"
- **sync.Mutex/RWMutex**: защита критических секций
- **sync/atomic**: простые атомарные операции
- **sync.WaitGroup**: ожидание завершения группы goroutine
- **sync.Once**: выполнение кода только один раз
- **context.Context**: передача сигналов отмены

**Подсказки для углубления:**
- Когда buffered channel лучше unbuffered?
- В чем разница между `sync.RWMutex.RLock()` и `Lock()`?
- Как работает `select` с multiple channels? Random selection?
- Что происходит при закрытии channel? Можно ли читать из закрытого channel?

### 7. Что происходит при блокировке goroutine? Как это влияет на планировщик?

**Ожидаемый ответ:**
При блокировке на:
- **Channel operations**: goroutine переходит в waiting state
- **System calls**: M отвязывается от P, создается новый M
- **Mutex**: goroutine встает в очередь ожидания

Планировщик переключается на следующую goroutine в run queue.

**Подсказки для углубления:**
- Что такое netpoller и как он работает с network I/O?
- Как Go обрабатывает blocking system calls vs non-blocking?
- Что произойдет, если все M заблокированы?

## 🏗️ Архитектура и проектирование

### 8. Объясните принципы SOLID на примере Go кода. Как интерфейсы помогают их соблюдать?

**Ожидаемый ответ:**
```go
// Interface Segregation - маленькие интерфейсы
type Reader interface { Read([]byte) (int, error) }
type Writer interface { Write([]byte) (int, error) }

// Dependency Inversion - зависимость от абстракции
type UserService struct {
    repo UserRepository // интерфейс, а не конкретная реализация
}

// Open/Closed - расширение через композицию
type LoggingWriter struct {
    Writer // встраивание интерфейса
    logger Logger
}
```

**Подсказки для углубления:**
- Как пустой интерфейс `interface{}` нарушает принципы SOLID?
- Почему в Go принято определять интерфейсы на стороне потребителя?
- Как использовать встраивание (embedding) для расширения функциональности?

### 9. Что такое Dependency Injection в Go? Покажите разные подходы.

**Ожидаемый ответ:**
```go
// Constructor injection
func NewUserService(repo UserRepository, logger Logger) *UserService {
    return &UserService{repo: repo, logger: logger}
}

// Interface injection
type UserService struct {
    repo UserRepository
}

// DI Container (например, wire, dig)
type Container struct {
    userRepo UserRepository
    userService UserService
}
```

**Подсказки для углубления:**
- В чем проблема Service Locator pattern?
- Как библиотеки типа `google/wire` генерируют код?
- Когда использовать DI container, а когда manual wiring?

### 10. Как бы вы спроектировали микросервисную архитектуру на Go?

**Ожидаемый ответ:**
```go
// Слоистая архитектура
// domain/ - бизнес-логика и сущности
// usecase/ - сценарии использования
// infrastructure/ - внешние зависимости
// interface/ - HTTP, gRPC handlers

// Пример структуры сервиса
type UserService interface {
    CreateUser(ctx context.Context, user User) error
}

type userService struct {
    repo UserRepository
    eventBus EventBus
}
```

**Подсказки для углубления:**
- Как обеспечить observability (metrics, traces, logs)?
- Как реализовать graceful shutdown?
- Как обрабатывать distributed transactions (Saga pattern)?
- Как версионировать API между сервисами?

## 🔧 Продвинутые темы

### 11. Объясните memory model Go. Что такое happens-before?

**Ожидаемый ответ:**
Memory model определяет гарантии видимости изменений между goroutine:
- Channel operations создают happens-before edges
- Mutex Lock/Unlock создают happens-before
- sync.Once гарантирует happens-before

**Подсказки для углубления:**
- Что такое memory reordering и как процессор оптимизирует код?
- Почему этот код может не работать как ожидается без синхронизации?
```go
var a, b int
go func() { a = 1; b = 1 }()
go func() { for b == 0 {} fmt.Println(a) }()
```

### 12. Как работает Garbage Collector в Go? Какие есть trade-offs?

**Ожидаемый ответ:**
- Триколорный concurrent mark-and-sweep
- Stop-the-world паузы минимизированы (< 100μs цель)
- Trade-off: throughput vs latency
- GC triggered по GOGC (default 100%)

**Подсказки для углубления:**
- Что такое write barriers и зачем они нужны?
- Как уменьшить GC pressure? (object pooling, avoiding allocations)
- Когда использовать `sync.Pool`?
- Как настроить GC для конкретного workload?

### 13. Объясните reflection в Go. Когда его использовать и какие у него проблемы?

**Ожидаемый ответ:**
```go
// Reflection для неизвестных типов во время компиляции
func PrintValue(v interface{}) {
    rv := reflect.ValueOf(v)
    rt := reflect.TypeOf(v)
    
    switch rv.Kind() {
    case reflect.Struct:
        // iterate fields
    }
}
```

Проблемы:
- Производительность
- Type safety теряется
- Сложность отладки

**Подсказки для углубления:**
- Как работают JSON marshal/unmarshal внутри?
- Что такое struct tags и как их читать?
- В чем разница между `Value` и `Type` в reflect?

### 14. Как правильно обрабатывать ошибки в Go? Покажите разные паттерны.

**Ожидаемый ответ:**
```go
// Wrapping errors (Go 1.13+)
if err != nil {
    return fmt.Errorf("failed to process user %d: %w", userID, err)
}

// Custom error types
type ValidationError struct {
    Field string
    Value interface{}
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("validation failed for %s: %v", e.Field, e.Value)
}

// Sentinel errors
var ErrNotFound = errors.New("not found")

// Error checking
if errors.Is(err, ErrNotFound) { /* handle */ }
if errors.As(err, &validationErr) { /* handle */ }
```

**Подсказки для углубления:**
- Когда использовать panic/recover?
- Как организовать error handling в больших проектах?
- В чем проблема с `err.Error() == "some text"`?

### 15. Что такое context.Context? Как его правильно использовать?

**Ожидаемый ответ:**
```go
// Передача контекста через все слои
func (s *UserService) CreateUser(ctx context.Context, user User) error {
    // Проверка отмены
    select {
    case <-ctx.Done():
        return ctx.Err()
    default:
    }
    
    // Передача в следующий слой
    return s.repo.Save(ctx, user)
}

// Timeout context
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
defer cancel()
```

**Подсказки для углубления:**
- Когда использовать `context.WithValue()`? Что в него класть?
- Как context помогает с graceful shutdown?
- Что происходит при отмене родительского context?

## 🎯 Архитектурные задачи

### 16. Спроектируйте rate limiter. Какие алгоритмы можете предложить?

**Ожидаемый ответ:**
```go
// Token bucket
type TokenBucket struct {
    tokens    int64
    capacity  int64
    refillRate int64
    mu        sync.Mutex
    lastRefill time.Time
}

// Sliding window
type SlidingWindow struct {
    windows map[int64]int64
    mu      sync.RWMutex
}

// Distributed rate limiter с Redis
type RedisRateLimiter struct {
    client redis.Client
}
```

**Подсказки для углубления:**
- В чем разница между fixed window и sliding window?
- Как реализовать distributed rate limiting?
- Как обрабатывать burst traffic?

### 17. Как бы вы реализовали connection pool для базы данных?

**Ожидаемый ответ:**
```go
type ConnectionPool struct {
    connections chan *Connection
    factory     func() (*Connection, error)
    maxOpen     int
    maxIdle     int
    mu          sync.Mutex
    closed      bool
}

func (p *ConnectionPool) Get(ctx context.Context) (*Connection, error) {
    select {
    case conn := <-p.connections:
        return conn, nil
    case <-ctx.Done():
        return nil, ctx.Err()
    default:
        return p.factory()
    }
}
```

**Подсказки для углубления:**
- Как обрабатывать connection timeout?
- Что делать с broken connections?
- Как реализовать health check для connections?

### 18. Реализуйте circuit breaker pattern.

**Ожидаемый ответ:**
```go
type CircuitBreaker struct {
    maxRequests  uint32
    interval     time.Duration
    timeout      time.Duration
    
    mutex    sync.Mutex
    state    State
    counts   Counts
    expiry   time.Time
}

type State int
const (
    StateClosed State = iota
    StateHalfOpen
    StateOpen
)
```

**Подсказки для углубления:**
- Как выбрать пороговые значения?
- Как интегрировать с metrics/monitoring?
- В чем разница между circuit breaker и timeout?

### 19. Как реализовать graceful shutdown веб-сервера?

**Ожидаемый ответ:**
```go
func main() {
    srv := &http.Server{Addr: ":8080", Handler: handler}
    
    // Канал для сигналов ОС
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    
    go func() {
        if err := srv.ListenAndServe(); err != nil {
            log.Printf("Server error: %v", err)
        }
    }()
    
    <-quit
    log.Println("Shutting down server...")
    
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    if err := srv.Shutdown(ctx); err != nil {
        log.Printf("Server forced to shutdown: %v", err)
    }
}
```

**Подсказки для углубления:**
- Как обрабатывать long-running requests?
- Что делать с background goroutines?
- Как дождаться завершения всех connections?

## 🧪 Тестирование и отладка

### 20. Как тестировать concurrent код? Покажите примеры.

**Ожидаемый ответ:**
```go
func TestConcurrentMap(t *testing.T) {
    m := NewSafeMap()
    
    var wg sync.WaitGroup
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            m.Set(i, i*2)
            val, ok := m.Get(i)
            assert.True(t, ok)
            assert.Equal(t, i*2, val)
        }(i)
    }
    wg.Wait()
}

// Race detection
// go test -race
```

**Подсказки для углубления:**
- Как использовать table-driven tests для concurrent кода?
- Что такое property-based testing?
- Как мокать time.Now() в тестах?

### 21. Как отлаживать deadlock и performance проблемы?

**Ожидаемый ответ:**
```go
// Deadlock detection
go func() {
    log.Println(http.ListenAndServe("localhost:6060", nil))
}()
// curl http://localhost:6060/debug/pprof/goroutine?debug=1

// CPU profiling
f, _ := os.Create("cpu.prof")
pprof.StartCPUProfile(f)
defer pprof.StopCPUProfile()

// Memory profiling
f, _ := os.Create("mem.prof")
pprof.WriteHeapProfile(f)
f.Close()
```

**Подсказки для углубления:**
- Как читать goroutine stack traces?
- Что показывает flame graph?
- Как использовать `go tool trace`?

## 🔬 Специфичные вопросы

### 22. В чем разница между этими объявлениями?

```go
var users []User
users := []User{}
users := make([]User, 0)
users := make([]User, 0, 10)
```

**Ожидаемый ответ:**
- `var users []User` - nil slice, len=0, cap=0
- `users := []User{}` - empty slice, len=0, cap=0, но не nil
- `users := make([]User, 0)` - аналогично предыдущему
- `users := make([]User, 0, 10)` - len=0, cap=10, преаллокация памяти

**Подсказки для углубления:**
- Влияет ли это на JSON marshaling?
- Какой способ лучше для производительности?
- Что происходит при `append` к nil slice?

### 23. Объясните этот код. Что он выведет?

```go
func main() {
    var wg sync.WaitGroup
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            fmt.Printf("%d ", i)
        }()
    }
    wg.Wait()
}
```

**Ожидаемый ответ:**
Вероятно выведет "5 5 5 5 5" из-за замыкания на переменную `i`.

Исправления:
```go
// Вариант 1: передача параметра
go func(i int) { fmt.Printf("%d ", i) }(i)

// Вариант 2: локальная переменная
i := i
go func() { fmt.Printf("%d ", i) }()
```

**Подсказки для углубления:**
- Что такое closure и как они работают в Go?
- Как избежать подобных ошибок?
- Что будет, если использовать `for range`?

### 24. Как работают эти интерфейсы?

```go
type Reader interface {
    Read([]byte) (int, error)
}

type Writer interface {
    Write([]byte) (int, error)
}

type ReadWriter interface {
    Reader
    Writer
}
```

**Ожидаемый ответ:**
- Композиция интерфейсов через embedding
- `ReadWriter` требует реализации обоих методов
- Можно присвоить `ReadWriter` к `Reader` или `Writer`

**Подсказки для углубления:**
- Что такое type assertion и type switch?
- Как проверить, реализует ли тип интерфейс во время компиляции?
- В чем разница между `interface{}` и `any`?

### 25. Спроектируйте систему конфигурации для микросервиса.

**Ожидаемый ответ:**
```go
type Config struct {
    Database DatabaseConfig `json:"database"`
    Server   ServerConfig   `json:"server"`
    Redis    RedisConfig    `json:"redis"`
}

// Источники конфигурации:
// 1. Default values
// 2. Config file (JSON/YAML)
// 3. Environment variables
// 4. Command line flags
// 5. Remote config (etcd, consul)

func LoadConfig() (*Config, error) {
    cfg := &Config{
        // Default values
    }
    
    // Load from file
    if err := loadFromFile(cfg); err != nil {
        return nil, err
    }
    
    // Override with env vars
    if err := loadFromEnv(cfg); err != nil {
        return nil, err
    }
    
    // Validate
    if err := cfg.Validate(); err != nil {
        return nil, err
    }
    
    return cfg, nil
}
```

**Подсказки для углубления:**
- Как обеспечить hot reload конфигурации?
- Как секюрно хранить secrets?
- Как валидировать конфигурацию?
- Как организовать конфигурацию для разных окружений?

---

## 🎯 Критерии оценки

**Junior/Middle уровень:**
- Понимание основ синтаксиса Go
- Знание goroutines и channels
- Базовое понимание интерфейсов

**Senior уровень:**
- Глубокое понимание планировщика Go
- Умение проектировать архитектуру
- Знание race conditions и способов их избежания
- Понимание performance implications

**Tech Lead уровень:**
- Способность принимать архитектурные решения
- Знание trade-offs разных подходов
- Умение наставлять команду
- Понимание экосистемы Go и tooling

---

*Помните: цель собеседования не "завалить" кандидата, а понять уровень его экспертизы и способность решать реальные задачи. Давайте подсказки и направляйте разговор в нужное русло.*