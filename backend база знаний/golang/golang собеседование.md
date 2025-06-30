# Техническое интервью Go Backend разработчика
## 100 вопросов с постепенным углублением

---

## 🎯 Раздел 1: Основы Go (Вопросы 1-20)

### 1. Что такое Go и каковы его основные особенности?
**Ожидаемый ответ:** Компилируемый язык от Google, статическая типизация, garbage collector, встроенная поддержка concurrency, простой синтаксис.

### 2. Какие примитивные типы данных есть в Go?
**Ожидаемый ответ:** `bool`, `string`, числовые типы (`int`, `int8`-`int64`, `uint`, `uint8`-`uint64`, `float32`, `float64`, `complex64`, `complex128`), `byte` (alias для `uint8`), `rune` (alias для `int32`).

### 3. В чем разница между `var`, `:=` и `const`?
```go
var name string = "John"
name := "John"  // короткая запись
const PI = 3.14
```

### 4. Объясните zero values в Go
**Ожидаемый ответ:** `0` для чисел, `false` для bool, `""` для string, `nil` для указателей, слайсов, карт, каналов, функций и интерфейсов.

### 5. Что такое указатели в Go? Как они работают?
```go
var x int = 10
var p *int = &x
fmt.Println(*p) // 10
```

### 6. Какие есть циклы в Go?
**Ожидаемый ответ:** Только `for`, но в разных формах:
```go
for i := 0; i < 10; i++ {}
for condition {}
for {}
for key, value := range collection {}
```

### 7. Как работает `switch` в Go?
```go
switch x {
case 1:
    fmt.Println("one")
case 2, 3:
    fmt.Println("two or three")
default:
    fmt.Println("other")
}
```

### 8. Что такое `defer` и как он работает?
**Пример из кода:**
```go
func (pg *Postgres) Insert(ctx context.Context, query string, args ...any) (any, error) {
    ctx, span := pg.tracer.Start(ctx, "Postgres.Insert")
    defer span.End() // выполнится в конце функции
    
    // логика функции
}
```

### 9. Как обрабатываются ошибки в Go?
**Пример из кода:**
```go
func (r *RepoAmocrmSource) CreateAmocrmSource(ctx context.Context, ...) error {
    _, err := r.db.Insert(ctx, CreateAmocrmSource, args)
    if err != nil {
        r.logger.Error(ctx, err.Error())
        return err
    }
    return nil
}
```

### 10. Что такое пустой интерфейс `interface{}`?
**Ожидаемый ответ:** Может содержать значение любого типа. В Go 1.18+ заменен на `any`.

### 11. Как работает приведение типов в Go?
```go
var i interface{} = "hello"
s := i.(string)
s, ok := i.(string) // безопасное приведение
```

### 12. Что такое struct embedding?
```go
type Person struct {
    Name string
}

type Employee struct {
    Person  // встраивание
    ID int
}
```

### 13. В чем разница между методами с value и pointer receiver?
```go
func (p Person) GetName() string { return p.Name }     // value receiver
func (p *Person) SetName(name string) { p.Name = name } // pointer receiver
```

### 14. Как работает инициализация в Go?
**Ожидаемый ответ:** `init()` функции, порядок инициализации пакетов, различие между `var` блоками и `init()`.

### 15. Что такое type alias и type definition?
```go
type MyInt = int        // alias
type MyCustomInt int    // новый тип
```

### 16. Как работают вариативные функции?
```go
func sum(nums ...int) int {
    total := 0
    for _, num := range nums {
        total += num
    }
    return total
}
```

### 17. Что такое blank identifier `_`?
**Ожидаемый ответ:** Используется для игнорирования значений, которые не нужны.

### 18. Как работает область видимости в Go?
**Ожидаемый ответ:** Блочная область видимости, экспорт через заглавную букву.

### 19. Объясните понятие "shadowing" в Go
```go
var x = "global"
func main() {
    x := "local" // затеняет глобальную переменную
    fmt.Println(x) // "local"
}
```

### 20. Как работает `iota`?
```go
const (
    Monday = iota  // 0
    Tuesday        // 1
    Wednesday      // 2
)
```

---

## 📊 Раздел 2: Массивы, слайсы, мапы (Вопросы 21-35)

### 21. В чем разница между массивом и слайсом?
**Ожидаемый ответ:** Массив - фиксированный размер, слайс - динамический. Массив передается по значению, слайс содержит указатель на массив.

### 22. Как устроен слайс внутри?
**Ожидаемый ответ:** Структура из трех полей: указатель на данные, длина, емкость.
```go
type slice struct {
    array unsafe.Pointer
    len   int
    cap   int
}
```

### 23. Что произойдет в этом коде?
```go
s1 := []int{1, 2, 3}
s2 := s1[1:2]
s2[0] = 999
fmt.Println(s1) // что выведет?
```
**Ответ:** `[1 999 3]` - слайсы разделяют один массив.

### 24. Когда происходит реаллокация слайса?
**Пример из кода:**
```go
// В middleware/03-metric.go
attrs := make([]attribute.KeyValue, 0, 8) // cap = 8
attrs = append(attrs, otellog.String(model.FileKey, fmt.Sprintf("%s:%d", file, line)))
// Если добавим больше 8 элементов, произойдет реаллокация
```

### 25. Как правильно скопировать слайс?
```go
src := []int{1, 2, 3}
dst := make([]int, len(src))
copy(dst, src)
// или
dst := append([]int(nil), src...)
```

### 26. Что такое nil slice и empty slice? В чем разница?
```go
var nilSlice []int        // nil slice
emptySlice := []int{}     // empty slice
emptySlice2 := make([]int, 0) // empty slice
```

### 27. Как работает `append` со слайсами?
**Анализ кода:**
```go
// Из telemetry/logger.go
attrs := make([]otellog.KeyValue, 0, 8)
attrs = append(attrs, otellog.String(model.FileKey, fmt.Sprintf("%s:%d", file, line)))
if len(extraParams) > 0 {
    attrs = append(attrs, otellog.Map(model.ExtraLogFieldsKey, extraParams...))
}
```

### 28. Как правильно удалить элемент из слайса?
```go
// Удаление элемента по индексу i
slice = append(slice[:i], slice[i+1:]...)
// Или с сохранением порядка
copy(slice[i:], slice[i+1:])
slice = slice[:len(slice)-1]
```

### 29. Что такое string slicing и как он работает?
```go
s := "hello"
sub := s[1:4] // "ell"
```

### 30. Как устроены мапы в Go?
**Ожидаемый ответ:** Хеш-таблица с открытой адресацией, bucket-based структура.

### 31. Что произойдет при обращении к несуществующему ключу?
```go
m := map[string]int{"a": 1}
value := m["b"]        // 0 (zero value)
value, ok := m["b"]    // 0, false
```

### 32. Как правильно проверить существование ключа в мапе?
**Пример из кода:**
```go
// В amocrm/handler.go
amocrmContactPhone, ok := body.Message.Receiver["phone"].(string)
if !ok {
    amocrmContactPhone = "Он нужен, только если менеджер пишет первым из интерфейса амо"
}
```

### 33. Потокобезопасны ли мапы в Go?
**Ожидаемый ответ:** Нет, нужна синхронизация или использование `sync.Map`.

### 34. Как правильно итерироваться по мапе?
```go
for key, value := range m {
    fmt.Printf("%s: %d\n", key, value)
}
// Порядок не гарантирован!
```

### 35. В чем проблема этого кода?
```go
var m map[string]int
m["key"] = 1 // runtime panic!
```
**Ответ:** Нужно инициализировать мапу: `m = make(map[string]int)`.

---

## 🚀 Раздел 3: Concurrency - Горутины и каналы (Вопросы 36-55)

### 36. Что такое горутины и чем они отличаются от потоков ОС?
**Ожидаемый ответ:** Легковесные потоки, управляемые Go runtime, мультиплексирование на OS threads, малый stack (2KB).

### 37. Как создать и запустить горутину?
**Пример из кода:**
```go
// В userbot/service.go
qrChanForExport := make(chan string)
go func() {
    defer span.End()
    qrCtx := context.Background()
    qrChan, _ := userbot.GetQRChannel(qrCtx)
    // ... логика в горутине
}()
```

### 38. Что такое каналы и как они работают?
**Ожидаемый ответ:** Типизированные conduits для передачи данных между горутинами. Реализуют CSP модель.

### 39. В чем разница между буферизованными и небуферизованными каналами?
```go
unbuffered := make(chan int)     // блокирующий
buffered := make(chan int, 10)   // с буфером
```

### 40. Как закрыть канал и зачем это нужно?
**Пример из кода:**
```go
// В userbot/service.go
qrChan, _ := userbot.GetQRChannel(qrCtx)
for evt := range qrChan { // range автоматически завершится при закрытии канала
    if evt.Event == "code" {
        qrChanForExport <- evt.Code
    }
}
```

### 41. Что происходит при чтении из закрытого канала?
```go
ch := make(chan int, 2)
ch <- 1
ch <- 2
close(ch)

v1 := <-ch  // 1
v2 := <-ch  // 2
v3 := <-ch  // 0 (zero value)
v4, ok := <-ch // 0, false
```

### 42. Как работает `select` statement?
```go
select {
case msg1 := <-ch1:
    // обработка msg1
case msg2 := <-ch2:
    // обработка msg2
case <-time.After(5 * time.Second):
    // timeout
default:
    // неблокирующий случай
}
```

### 43. Что такое channel directions?
```go
func sender(ch chan<- int) {    // только отправка
    ch <- 42
}

func receiver(ch <-chan int) {  // только чтение
    value := <-ch
}
```

### 44. Как реализовать worker pool?
```go
func workerPool(jobs <-chan Job, results chan<- Result) {
    for job := range jobs {
        result := process(job)
        results <- result
    }
}

// Запуск workers
for w := 1; w <= numWorkers; w++ {
    go workerPool(jobs, results)
}
```

### 45. Что такое deadlock и как его избежать?
**Ожидаемый ответ:** Взаимная блокировка горутин. Избегать циклических зависимостей, использовать timeout, правильный порядок захвата ресурсов.

### 46. Как работает паттерн fan-in/fan-out?
```go
// Fan-out
func fanOut(in <-chan int, out1, out2 chan<- int) {
    for val := range in {
        select {
        case out1 <- val:
        case out2 <- val:
        }
    }
}

// Fan-in
func fanIn(in1, in2 <-chan int, out chan<- int) {
    for {
        select {
        case val := <-in1:
            out <- val
        case val := <-in2:
            out <- val
        }
    }
}
```

### 47. Что такое context и зачем он нужен?
**Пример из кода:**
```go
// В pg/pg.go
func (pg *Postgres) Insert(ctx context.Context, query string, args ...any) (any, error) {
    ctx, span := pg.tracer.Start(ctx, "Postgres.Insert",
        trace.WithSpanKind(trace.SpanKindClient),
    )
    defer span.End()
    // context передается для отмены операций и передачи значений
}
```

### 48. Как создать context с timeout?
```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
```

### 49. Как правильно передавать данные через context?
**Пример из кода:**
```go
// В middleware/01-trace.go
ctx = context.WithValue(ctx, model.TraceIDKey, spanCtx.TraceID().String())
ctx = context.WithValue(ctx, model.SpanIDKey, spanCtx.SpanID().String())

// Получение значения
traceID := ctx.Value(model.TraceIDKey).(string)
```

### 50. Что такое channel axioms (правила каналов)?
**Ожидаемый ответ:**
- Отправка в nil канал блокируется навсегда
- Чтение из nil канала блокируется навсегда
- Отправка в закрытый канал вызывает panic
- Чтение из закрытого канала возвращает zero value

### 51. Как реализовать graceful shutdown?
```go
func main() {
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    
    // запуск сервера в горутине
    go func() {
        if err := server.ListenAndServe(); err != nil {
            log.Fatal(err)
        }
    }()
    
    <-quit // ждем сигнал
    
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    server.Shutdown(ctx)
}
```

### 52. Что такое memory model в Go?
**Ожидаемый ответ:** Гарантии порядка выполнения операций в памяти между горутинами. Happens-before отношения.

### 53. Как работает `go` statement?
**Ожидаемый ответ:** Создает новую горутину, функция выполняется асинхронно, аргументы вычисляются в текущей горутине.

### 54. Что происходит с паникой в горутине?
```go
go func() {
    panic("something went wrong") // убьет всю программу
}()

// Правильно:
go func() {
    defer func() {
        if r := recover(); r != nil {
            log.Printf("Recovered: %v", r)
        }
    }()
    panic("something went wrong")
}()
```

### 55. Как реализовать timeout для операции?
```go
select {
case result := <-longRunningOperation():
    return result
case <-time.After(5 * time.Second):
    return errors.New("timeout")
}
```

---

## 🔒 Раздел 4: Синхронизация (Вопросы 56-70)

### 56. Что такое data race и как его обнаружить?
**Ожидаемый ответ:** Одновременный доступ к памяти из разных горутин без синхронизации. Детектируется флагом `-race`.

### 57. Как работает `sync.Mutex`?
**Пример из кода:**
```go
// В userbot/service.go - потенциальная проблема
type ServiceUserbot struct {
    userbots map[string]*whatsmeow.Client // нужна синхронизация!
}

// Правильно:
type ServiceUserbot struct {
    mu       sync.RWMutex
    userbots map[string]*whatsmeow.Client
}

func (s *ServiceUserbot) addUserbot(key string, client *whatsmeow.Client) {
    s.mu.Lock()
    defer s.mu.Unlock()
    s.userbots[key] = client
}
```

### 58. В чем разница между `Mutex` и `RWMutex`?
**Ожидаемый ответ:** `RWMutex` позволяет множественное чтение, но эксклюзивную запись.

### 59. Что такое `sync.WaitGroup` и как его использовать?
```go
var wg sync.WaitGroup

for i := 0; i < 10; i++ {
    wg.Add(1)
    go func(id int) {
        defer wg.Done()
        // работа
    }(i)
}

wg.Wait()
```

### 60. Как работает `sync.Once`?
```go
var once sync.Once
var config *Config

func GetConfig() *Config {
    once.Do(func() {
        config = loadConfig() // выполнится только один раз
    })
    return config
}
```

### 61. Что такое `sync.Pool` и зачем он нужен?
```go
var bufferPool = sync.Pool{
    New: func() interface{} {
        return make([]byte, 1024)
    },
}

func processData() {
    buf := bufferPool.Get().([]byte)
    defer bufferPool.Put(buf)
    // использование buf
}
```

### 62. Как работают атомарные операции?
```go
import "sync/atomic"

var counter int64

func increment() {
    atomic.AddInt64(&counter, 1)
}

func getCounter() int64 {
    return atomic.LoadInt64(&counter)
}
```

### 63. Что такое happens-before relationship?
**Ожидаемый ответ:** Гарантии порядка выполнения операций в памяти. Основа memory model Go.

### 64. Как правильно синхронизировать доступ к мапе?
```go
type SafeMap struct {
    mu sync.RWMutex
    m  map[string]int
}

func (sm *SafeMap) Get(key string) (int, bool) {
    sm.mu.RLock()
    defer sm.mu.RUnlock()
    val, ok := sm.m[key]
    return val, ok
}

func (sm *SafeMap) Set(key string, val int) {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    sm.m[key] = val
}
```

### 65. Что такое `sync.Map` и когда его использовать?
**Ожидаемый ответ:** Потокобезопасная мапа, оптимизированная для случаев с редкими записями и частыми чтениями.

### 66. Как работает `sync.Cond`?
```go
var mu sync.Mutex
var cond = sync.NewCond(&mu)
var ready bool

func waiter() {
    mu.Lock()
    for !ready {
        cond.Wait()
    }
    mu.Unlock()
    // продолжить работу
}

func setter() {
    mu.Lock()
    ready = true
    cond.Broadcast()
    mu.Unlock()
}
```

### 67. В чем проблема этого кода?
```go
var mu sync.Mutex
var data map[string]int

func getValue(key string) int {
    mu.Lock()
    defer mu.Unlock()
    return data[key] // Что если data == nil?
}
```

### 68. Как избежать deadlock при использовании нескольких мьютексов?
**Ожидаемый ответ:** Всегда захватывать мьютексы в одинаковом порядке, использовать timeout, минимизировать время удержания.

### 69. Что такое lock contention и как его уменьшить?
**Ожидаемый ответ:** Соревнование за блокировки. Уменьшить через sharding, RWMutex, атомарные операции, lock-free структуры.

### 70. Как правильно обработать панику с mutex?
```go
func riskyOperation() {
    mu.Lock()
    defer func() {
        mu.Unlock()
        if r := recover(); r != nil {
            // обработка паники
        }
    }()
    // опасная операция
}
```

---

## ⚙️ Раздел 5: Runtime, планировщик, GC (Вопросы 71-85)

### 71. Как работает планировщик Go (G-M-P модель)?
**Ожидаемый ответ:** 
- G (Goroutine) - горутина
- M (Machine) - OS thread
- P (Processor) - логический процессор
- Work-stealing алгоритм

### 72. Что такое GOMAXPROCS и как он влияет на производительность?
```go
import "runtime"

func init() {
    runtime.GOMAXPROCS(runtime.NumCPU()) // по умолчанию
}
```

### 73. Как работает garbage collector в Go?
**Ожидаемый ответ:** Concurrent mark-and-sweep, триколорный алгоритм, write barriers, STW фазы минимизированы.

### 74. Что такое GC pacer и как он работает?
**Ожидаемый ответ:** Контролирует частоту запуска GC на основе GOGC переменной и allocation rate.

### 75. Как настроить garbage collector?
```go
import "runtime/debug"

// GOGC=100 по умолчанию (удваивает heap перед GC)
debug.SetGCPercent(50) // более агрессивная сборка
```

### 76. Что такое stack growth в Go?
**Ожидаемый ответ:** Динамическое увеличение стека горутины, начинается с 2KB, может расти до 1GB.

### 77. Как работает escape analysis?
```go
func createLocal() *int {
    x := 42
    return &x // x "убегает" на heap
}

func createHeap() {
    x := make([]int, 1000000) // большой слайс на heap
    _ = x
}
```

### 78. Что показывает `go build -gcflags="-m"`?
**Ожидаемый ответ:** Результаты escape analysis - какие переменные размещаются на heap.

### 79. Как работает preemption в Go?
**Ожидаемый ответ:** 
- Cooperative до Go 1.14
- Asynchronous preemption с Go 1.14+
- На основе signals

### 80. Что такое memory layout в Go?
**Ожидаемый ответ:** Heap, stack, global data, структура объектов в памяти, pointer alignment.

### 81. Как работает write barrier в GC?
**Ожидаемый ответ:** Отслеживает изменения указателей во время concurrent mark фазы.

### 82. Что такое finalizers в Go?
```go
import "runtime"

func createResource() {
    resource := acquireResource()
    runtime.SetFinalizer(resource, (*Resource).Close)
}
```

### 83. Как работает `runtime.KeepAlive`?
```go
func unsafeOperation(ptr unsafe.Pointer) {
    // работа с указателем
    runtime.KeepAlive(originalObject) // гарантирует, что объект не будет собран
}
```

### 84. Что показывает `runtime.ReadMemStats`?
**Пример применения:**
```go
// В telemetry можно добавить мониторинг памяти
var m runtime.MemStats
runtime.ReadMemStats(&m)
fmt.Printf("Alloc = %d KB", bToKb(m.Alloc))
fmt.Printf("HeapAlloc = %d KB", bToKb(m.HeapAlloc))
```

### 85. Как оптимизировать работу с GC?
**Ожидаемый ответ:** 
- Минимизировать аллокации
- Использовать object pooling
- Избегать pointer-heavy структур
- Batch operations

---

## 🏗️ Раздел 6: Продвинутые темы (Вопросы 86-100)

### 86. Как работает reflection в Go?
**Пример из кода (потенциальное использование):**
```go
// В middleware для динамической обработки событий
func processEvent(evt interface{}) {
    v := reflect.ValueOf(evt)
    t := reflect.TypeOf(evt)
    
    switch t {
    case reflect.TypeOf(&events.Message{}):
        // обработка сообщения
    case reflect.TypeOf(&events.JoinedGroup{}):
        // обработка присоединения к группе
    }
}
```

### 87. Что такое unsafe пакет и когда его использовать?
```go
import "unsafe"

// Конвертация string в []byte без копирования (опасно!)
func stringToBytes(s string) []byte {
    return unsafe.Slice(unsafe.StringData(s), len(s))
}
```

### 88. Как работают generics в Go 1.18+?
```go
// Типизированная мапа с мьютексом
type SafeMap[K comparable, V any] struct {
    mu sync.RWMutex
    m  map[K]V
}

func (sm *SafeMap[K, V]) Get(key K) (V, bool) {
    sm.mu.RLock()
    defer sm.mu.RUnlock()
    val, ok := sm.m[key]
    return val, ok
}
```

### 89. Как оптимизировать производительность в Go?
**Анализ кода:**
```go
// В telemetry/logger.go - возможная оптимизация
func (l *OtelLogger) extractExtraParams(fields map[string]interface{}) []otellog.KeyValue {
    extraAttrs := make([]otellog.KeyValue, 0, len(fields)) // pre-allocate capacity
    // ...
}
```

### 90. Что такое build constraints/tags?
```go
//go:build !windows
// +build !windows

package unix_specific

// код только для не-Windows систем
```

### 91. Как работает `go:generate`?
```go
//go:generate stringer -type=Status
type Status int

const (
    StatusActive Status = iota
    StatusInactive
)
```

### 92. Как правильно работать с JSON в Go?
**Пример из кода:**
```go
// В amocrm/model.go
type TextMessageFromAmocrmBody struct {
    AccountID string `json:"account_id"`
    Message   struct {
        Receiver     map[string]any `json:"receiver"`
        Conversation struct {
            ID string `json:"id"`
        } `json:"conversation"`
    } `json:"message"`
}
```

### 93. Как реализовать middleware pattern?
**Пример из кода:**
```go
// В middleware/05-router.go
func RouterMiddleware05(
    messageHandler func(ctx context.Context, event *events.Message) error,
    joinedGroupHandler func(ctx context.Context, event *events.JoinedGroup) error,
) func(ctx context.Context, evt interface{}) error {
    return func(ctx context.Context, evt interface{}) error {
        switch event := evt.(type) {
        case *events.Message:
            return messageHandler(ctx, event)
        case *events.JoinedGroup:
            return joinedGroupHandler(ctx, event)
        }
        return nil
    }
}
```

### 94. Как работать с базами данных в Go?
**Анализ кода из pg/pg.go:**
```go
func (pg *Postgres) Insert(ctx context.Context, query string, args ...any) (any, error) {
    var id any
    tx := pg.getTx(ctx)
    if tx != nil {
        // транзакционная вставка
        err := tx.QueryRow(ctx, query, args...).Scan(&id)
        return id, err
    } else {
        // обычная вставка
        err := pg.db.QueryRow(ctx, query, args...).Scan(&id)
        return id, err
    }
}
```

### 95. Как правильно структурировать Go проект?
**Анализ структуры проекта:**
```
crmessenger-whatsapp/
├── cmd/                    # main приложения
├── internal/              # приватный код
│   ├── app/              # конфигурация приложения
│   ├── controller/       # обработчики
│   ├── service/          # бизнес-логика
│   ├── repo/             # работа с данными
│   └── model/            # модели данных
├── infrastructure/       # внешние сервисы
├── pkg/                  # публичные библиотеки
└── main.go
```

### 96. Как реализовать graceful shutdown?
**Пример для проекта:**
```go
func main() {
    // ... инициализация
    
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    
    go func() {
        server.Run(db, amocrmSourceService, userbotService, tel, cfg.CRMessenger.CRMessengerWhatsapp.Port)
    }()
    
    <-quit
    
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    // Shutdown telemetry
    if err := tel.Shutdown(ctx); err != nil {
        log.Printf("Telemetry shutdown error: %v", err)
    }
}
```

### 97. Как работать с OpenTelemetry в Go?
**Анализ кода из telemetry/telemetry.go:**
```go
func (t *Telemetry) setupTracing(ctx context.Context, res *resource.Resource) error {
    exporter, err := otlptracegrpc.New(ctx, otlptracegrpc.WithGRPCConn(conn))
    if err != nil {
        return fmt.Errorf("failed to create trace exporter: %w", err)
    }

    var sampler sdktrace.Sampler
    if t.environment == "prod" {
        sampler = sdktrace.TraceIDRatioBased(6) // 6% sampling
    } else {
        sampler = sdktrace.AlwaysSample()
    }
    
    t.tracerProvider = sdktrace.NewTracerProvider(
        sdktrace.WithBatcher(exporter),
        sdktrace.WithResource(res),
        sdktrace.WithSampler(sampler),
    )
}
```

### 98. Как тестировать Go код?
```go
func TestUserbotService_CreateUserbot(t *testing.T) {
    // Arrange
    mockRepo := &MockUserbotRepo{}
    service := NewUserbotService(mockRepo, tel)
    
    // Act
    result, err := service.CreateUserbot(context.Background(), 123)
    
    // Assert
    assert.NoError(t, err)
    assert.NotNil(t, result)
    mockRepo.AssertExpectations(t)
}
```

### 99. Как профилировать Go приложения?
```go
import _ "net/http/pprof"

func main() {
    go func() {
        log.Println(http.ListenAndServe("localhost:6060", nil))
    }()
    
    // основное приложение
}

// go tool pprof http://localhost:6060/debug/pprof/profile
```

### 100. Каковы best practices для Go разработки?
**На основе анализа кода:**

1. **Error Handling**: Всегда проверяйте ошибки
```go
if err != nil {
    span.RecordError(err)
    span.SetStatus(codes.Error, err.Error())
    return err
}
```

2. **Context Propagation**: Передавайте context везде
```go
func (s *ServiceUserbot) CreateUserbot(ctx context.Context, accountID int) (*bytes.Buffer, error)
```

3. **Dependency Injection**: Используйте интерфейсы
```go
type IUserbotService interface {
    CreateUserbot(ctx context.Context, accountID int) (*bytes.Buffer, error)
    SendMessage(ctx context.Context, userbotWaPhoneNumber, contactWaPhoneNumber, text string) error
}
```

4. **Observability**: Трейсинг, метрики, логирование
```go
ctx, span := s.tracer.Start(ctx, "ServiceUserbot.CreateUserbot")
defer span.End()
```

5. **Configuration**: Используйте environment variables
```go
type Config struct {
    DB struct {
        Username string
        Password string
        Host     string
    }
}
```

---

## 🎯 Критерии оценки ответов

### Junior (1-40 вопросов)
- Знание основ синтаксиса
- Понимание типов данных
- Базовая работа с горутинами и каналами

### Middle (41-70 вопросов)
- Глубокое понимание concurrency
- Знание sync примитивов
- Опыт проектирования архитектуры

### Senior (71-100 вопросов)
- Понимание внутреннего устройства Go
- Оптимизация производительности
- Best practices и паттерны

---

## 💡 Дополнительные практические задачи

### Задача 1: Race Condition
```go
// Найдите и исправьте race condition
type Counter struct {
    value int
}

func (c *Counter) Increment() {
    c.value++
}

func (c *Counter) Value() int {
    return c.value
}
```

### Задача 2: Deadlock
```go
// Почему этот код может привести к deadlock?
func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)
    
    go func() {
        ch1 <- 1
        <-ch2
    }()
    
    go func() {
        ch2 <- 2
        <-ch1
    }()
    
    time.Sleep(time.Second)
}
```

### Задача 3: Memory Leak
```go
// Найдите потенциальную утечку памяти
func processEvents() {
    events := make(chan Event, 1000)
    
    go func() {
        for event := range events {
            process(event)
        }
    }()
    
    // events канал никогда не закрывается
}
```

Этот набор вопросов охватывает все ключевые аспекты Go разработки с реальными примерами из вашего кода и поможет оценить уровень кандидата от Junior до Senior.
