# Техническое интервью Go Backend разработчика
## 100 вопросов с развернутыми ответами

---

## 🎯 Раздел 1: Основы Go (Ответы 1-20)

### 1. Что такое Go и каковы его основные особенности?

**Развернутый ответ:**
Go (Golang) — компилируемый язык программирования, разработанный в Google в 2009 году. Основные особенности:

- **Статическая типизация** с выводом типов
- **Компиляция в машинный код** — быстрое выполнение
- **Garbage Collector** — автоматическое управление памятью
- **Встроенная поддержка concurrency** через горутины и каналы
- **Простой синтаксис** — минимализм и читаемость
- **Быстрая компиляция** — секунды даже для больших проектов
- **Кроссплатформенность** — один код для разных ОС

```go
// Пример простоты синтаксиса
package main

import "fmt"

func main() {
    message := "Hello, Go!" // автоматический вывод типа
    fmt.Println(message)
}
```

### 2. Какие примитивные типы данных есть в Go?

**Развернутый ответ:**
Go имеет богатый набор встроенных типов:

**Булевы:** `bool` (true/false)
**Строковые:** `string` (UTF-8)
**Целые числа:**
- Знаковые: `int`, `int8`, `int16`, `int32`, `int64`
- Беззнаковые: `uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr`
**Числа с плавающей точкой:** `float32`, `float64`
**Комплексные числа:** `complex64`, `complex128`
**Алиасы:** `byte` (uint8), `rune` (int32 для Unicode)

```go
var (
    active     bool    = true
    count      int     = 42
    percentage float64 = 98.6
    name       string  = "Go"
    symbol     rune    = '🚀'
)
```

### 3. В чем разница между `var`, `:=` и `const`?

**Развернутый ответ:**

**`var`** — объявление переменных с возможностью инициализации:
- Может использоваться на уровне пакета и функции
- Позволяет явно указать тип
- Автоматически инициализируется zero value

**`:=`** — короткое объявление с инициализацией:
- Только внутри функций
- Автоматический вывод типа
- Требует инициализации

**`const`** — объявление констант:
- Значение должно быть известно на этапе компиляции
- Неизменяемые значения

```go
var globalVar int              // zero value: 0
var initializedVar int = 10    // явная инициализация

func example() {
    localVar := 20             // короткое объявление
    const maxRetries = 3       // константа
    
    var dynamicVar int
    if condition {
        dynamicVar = 42
    }
}
```

### 4. Объясните zero values в Go

**Развернутый ответ:**
В Go каждый тип имеет нулевое значение (zero value) — значение по умолчанию для неинициализированных переменных:

- **Числовые типы**: `0`
- **bool**: `false`
- **string**: `""` (пустая строка)
- **Указатели, слайсы, мапы, каналы, функции, интерфейсы**: `nil`
- **Массивы и структуры**: zero value всех полей

```go
var (
    num    int         // 0
    active bool        // false
    name   string      // ""
    ptr    *int        // nil
    slice  []int       // nil
    m      map[string]int // nil
)

type User struct {
    ID   int    // 0
    Name string // ""
}
var user User // {ID: 0, Name: ""}
```

### 5. Что такое указатели в Go? Как они работают?

**Развернутый ответ:**
Указатели в Go содержат адрес памяти переменной. Go поддерживает указатели, но не арифметику указателей (как в C/C++).

**Операторы:**
- `&` — получить адрес переменной
- `*` — разыменование (получить значение по адресу)

```go
func pointerExample() {
    x := 42
    var p *int = &x    // p указывает на x
    
    fmt.Println(x)     // 42
    fmt.Println(p)     // адрес в памяти (например: 0xc000014098)
    fmt.Println(*p)    // 42 (значение по адресу)
    
    *p = 100           // изменяем значение x через указатель
    fmt.Println(x)     // 100
}

// Передача по ссылке
func increment(n *int) {
    *n++
}
```

### 6. Какие есть циклы в Go?

**Развернутый ответ:**
В Go есть только один тип цикла — `for`, но в разных формах:

**1. Классический цикл:**
```go
for i := 0; i < 10; i++ {
    fmt.Println(i)
}
```

**2. Цикл-условие (аналог while):**
```go
i := 0
for i < 10 {
    fmt.Println(i)
    i++
}
```

**3. Бесконечный цикл:**
```go
for {
    // бесконечный цикл
    if condition {
        break
    }
}
```

**4. Range цикл (для коллекций):**
```go
slice := []int{1, 2, 3}
for index, value := range slice {
    fmt.Printf("%d: %d\n", index, value)
}

m := map[string]int{"a": 1, "b": 2}
for key, value := range m {
    fmt.Printf("%s: %d\n", key, value)
}
```

### 7. Как работает `switch` в Go?

**Развернутый ответ:**
`switch` в Go более мощный, чем в других языках:

**Особенности:**
- Автоматический `break` (нет fallthrough по умолчанию)
- Можно использовать любые типы
- Множественные значения в case
- Switch без выражения (как if-else цепочка)

```go
// Обычный switch
func gradeSwitch(score int) string {
    switch score / 10 {
    case 10, 9:
        return "A"
    case 8:
        return "B"
    case 7:
        return "C"
    default:
        return "F"
    }
}

// Switch без выражения
func checkNumber(x int) {
    switch {
    case x < 0:
        fmt.Println("negative")
    case x == 0:
        fmt.Println("zero")
    case x > 0:
        fmt.Println("positive")
    }
}

// Type switch
func processValue(v interface{}) {
    switch v := v.(type) {
    case string:
        fmt.Printf("String: %s\n", v)
    case int:
        fmt.Printf("Integer: %d\n", v)
    default:
        fmt.Printf("Unknown type: %T\n", v)
    }
}
```

### 8. Что такое `defer` и как он работает?

**Развернутый ответ:**
`defer` откладывает выполнение функции до момента возврата из текущей функции. Отложенные вызовы выполняются в порядке LIFO (последний вошел — первый вышел).

**Применение:**
- Освобождение ресурсов
- Закрытие файлов/соединений
- Разблокировка мьютексов
- Трейсинг и логирование

```go
func resourceManagement() error {
    file, err := os.Open("data.txt")
    if err != nil {
        return err
    }
    defer file.Close() // гарантированно закроется, даже при ошибке
    
    mu.Lock()
    defer mu.Unlock() // автоматическая разблокировка
    
    // работа с ресурсами
    return processFile(file)
}

// Порядок выполнения defer
func deferOrder() {
    defer fmt.Println("1")
    defer fmt.Println("2")
    defer fmt.Println("3")
    fmt.Println("function body")
}
// Вывод: function body, 3, 2, 1
```

### 9. Как обрабатываются ошибки в Go?

**Развернутый ответ:**
Go использует явную обработку ошибок через возвращаемые значения, а не исключения:

**Принципы:**
- Ошибки — это значения типа `error`
- Явная проверка каждой ошибки
- Возврат ошибки как последнего значения

```go
// Стандартный паттерн
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

func example() error {
    result, err := divide(10, 0)
    if err != nil {
        return fmt.Errorf("calculation failed: %w", err)
    }
    
    fmt.Printf("Result: %f\n", result)
    return nil
}

// Кастомный тип ошибки
type ValidationError struct {
    Field string
    Value string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("invalid %s: %s", e.Field, e.Value)
}
```

### 10. Что такое пустой интерфейс `interface{}`?

**Развернутый ответ:**
Пустой интерфейс `interface{}` (с Go 1.18 — `any`) может содержать значение любого типа, так как все типы реализуют пустой интерфейс.

**Использование:**
- Generic-like поведение до появления дженериков
- Работа с неизвестными типами
- JSON обработка

```go
// any — алиас для interface{} с Go 1.18
func printAny(value any) {
    fmt.Printf("Value: %v, Type: %T\n", value, value)
}

func example() {
    printAny(42)
    printAny("hello")
    printAny([]int{1, 2, 3})
    printAny(map[string]int{"a": 1})
}

// Работа с JSON
func processJSON(data []byte) (any, error) {
    var result any
    err := json.Unmarshal(data, &result)
    return result, err
}
```

### 11. Как работает приведение типов в Go?

**Развернутый ответ:**
Go поддерживает два вида приведения типов:

**1. Type assertion** — для интерфейсов:
```go
var i interface{} = "hello"

// Небезопасное приведение (может panic)
s := i.(string)

// Безопасное приведение
s, ok := i.(string)
if ok {
    fmt.Println("String:", s)
}

// Type switch для множественных типов
switch v := i.(type) {
case string:
    fmt.Println("String:", v)
case int:
    fmt.Println("Int:", v)
default:
    fmt.Println("Unknown type")
}
```

**2. Type conversion** — между совместимыми типами:
```go
var i int = 42
var f float64 = float64(i)  // явное преобразование
var s string = string(rune(i)) // int to rune to string

// Между пользовательскими типами
type UserId int
type ProductId int

var uid UserId = 123
var pid ProductId = ProductId(uid) // явное преобразование
```

### 12. Что такое struct embedding?

**Развернутый ответ:**
Struct embedding позволяет встраивать один struct в другой, получая его поля и методы напрямую (композиция вместо наследования).

```go
type Person struct {
    Name string
    Age  int
}

func (p Person) Greet() string {
    return fmt.Sprintf("Hi, I'm %s", p.Name)
}

type Employee struct {
    Person   // встраивание
    Position string
    Salary   int
}

func (e Employee) Work() string {
    return fmt.Sprintf("%s is working as %s", e.Name, e.Position)
}

func example() {
    emp := Employee{
        Person:   Person{Name: "John", Age: 30},
        Position: "Developer",
        Salary:   100000,
    }
    
    // Доступ к полям встроенной структуры
    fmt.Println(emp.Name)      // напрямую
    fmt.Println(emp.Person.Name) // через явное указание
    
    // Доступ к методам
    fmt.Println(emp.Greet())   // метод Person
    fmt.Println(emp.Work())    // метод Employee
}
```

### 13. В чем разница между методами с value и pointer receiver?

**Развернутый ответ:**

**Value receiver** — работает с копией:
- Не может изменить оригинальный объект
- Вызывается на копии структуры
- Используется для read-only операций

**Pointer receiver** — работает с оригиналом:
- Может изменять поля структуры
- Более эффективен для больших структур
- Обязателен для методов, изменяющих состояние

```go
type Counter struct {
    count int
}

// Value receiver - не изменяет оригинал
func (c Counter) GetCount() int {
    return c.count
}

// Pointer receiver - изменяет оригинал
func (c *Counter) Increment() {
    c.count++
}

func (c *Counter) Reset() {
    c.count = 0
}

func example() {
    counter := Counter{count: 5}
    
    fmt.Println(counter.GetCount()) // 5
    counter.Increment()
    fmt.Println(counter.GetCount()) // 6
    
    // Go автоматически преобразует между & и *
    counterPtr := &counter
    counterPtr.Increment() // то же, что (*counterPtr).Increment()
}
```

### 14. Как работает инициализация в Go?

**Развернутый ответ:**
Инициализация в Go происходит в определенном порядке:

**1. Package-level переменные** инициализируются в порядке зависимостей
**2. init() функции** выполняются после инициализации переменных
**3. main() функция** выполняется в main пакете

```go
// Порядок инициализации переменных
var (
    a = b + c  // 3-й
    b = f()    // 2-й  
    c = 1      // 1-й
)

func f() int { return c + 1 }

func init() {
    // init функций может быть несколько
    fmt.Println("First init")
}

func init() {
    fmt.Println("Second init")
}

func main() {
    fmt.Println("main function")
}

// Порядок выполнения:
// 1. c = 1
// 2. b = f() = 2
// 3. a = b + c = 3
// 4. First init
// 5. Second init
// 6. main function
```

### 15. Что такое type alias и type definition?

**Развернутый ответ:**

**Type alias** создает новое имя для существующего типа:
**Type definition** создает новый тип на основе существующего:

```go
// Type alias - полностью взаимозаменяемы
type StringAlias = string
type IntAlias = int

// Type definition - новые типы
type UserId int
type ProductId int
type Email string

func processUser(id UserId) { /* ... */ }
func processProduct(id ProductId) { /* ... */ }

func example() {
    // Alias - взаимозаменяемы
    var s string = "hello"
    var sa StringAlias = s  // OK
    s = sa                  // OK
    
    // Definition - требует явного преобразования
    var userId UserId = 123
    var productId ProductId = 456
    
    // userId = productId        // Error!
    userId = UserId(productId)   // OK - явное преобразование
    
    // Методы можно добавлять только к новым типам
    func (e Email) IsValid() bool {
        return strings.Contains(string(e), "@")
    }
}
```

### 16. Как работают вариативные функции?

**Развернутый ответ:**
Вариативные функции принимают переменное количество аргументов одного типа:

```go
// Объявление вариативной функции
func sum(nums ...int) int {
    total := 0
    for _, num := range nums {
        total += num
    }
    return total
}

func printf(format string, args ...interface{}) {
    fmt.Printf(format, args...)
}

func example() {
    // Различные способы вызова
    fmt.Println(sum())           // 0
    fmt.Println(sum(1))          // 1
    fmt.Println(sum(1, 2, 3))    // 6
    
    // Передача слайса
    numbers := []int{1, 2, 3, 4, 5}
    fmt.Println(sum(numbers...)) // 15
    
    // Смешанные параметры
    printf("Name: %s, Age: %d\n", "John", 30)
}

// Реальный пример из кода
func logWithFields(message string, fields ...interface{}) {
    logger := getLogger()
    logger.WithFields(fields...).Info(message)
}
```

### 17. Что такое blank identifier `_`?

**Развернутый ответ:**
Blank identifier `_` используется для игнорирования значений, которые не нужны:

```go
// Игнорирование возвращаемых значений
_, err := fmt.Printf("Hello, World!")
if err != nil {
    return err
}

// Игнорирование в range
for _, value := range slice {
    process(value) // индекс не нужен
}

for index := range slice {
    processIndex(index) // значение не нужно
}

// Импорт пакета только для init()
import _ "database/sql/driver"

// Проверка реализации интерфейса на этапе компиляции
var _ io.Writer = (*bytes.Buffer)(nil)

// Игнорирование полей в struct
type Config struct {
    Host     string
    Port     int
    _        struct{} // запрет добавления полей
    Password string
}
```

### 18. Как работает область видимости в Go?

**Развернутый ответ:**
Go использует блочную область видимости и правило экспорта:

**Уровни видимости:**
1. **Universe scope** — встроенные типы и функции
2. **Package scope** — переменные уровня пакета
3. **File scope** — импорты
4. **Function scope** — параметры и локальные переменные
5. **Block scope** — внутри `{}`

**Экспорт:** заглавная буква делает идентификатор публичным

```go
package example

// Публичные (экспортируемые)
var PublicVar = "visible outside package"
type PublicStruct struct {
    PublicField    string
    privateField   int  // приватное поле
}

func PublicFunction() {}

// Приватные
var privateVar = "only in package"
func privateFunction() {}

func scopeExample() {
    var functionVar = "function scope"
    
    for i := 0; i < 10; i++ {
        var blockVar = "block scope"
        
        if i > 5 {
            var nestedBlockVar = "nested block"
            // все переменные доступны здесь
        }
        // nestedBlockVar недоступна здесь
    }
    // i и blockVar недоступны здесь
}
```

### 19. Объясните понятие "shadowing" в Go

**Развернутый ответ:**
Shadowing происходит, когда переменная во внутренней области видимости "затеняет" переменную с тем же именем из внешней области:

```go
var global = "global variable"

func shadowingExample() {
    var local = "function variable"
    
    fmt.Println(global) // "global variable"
    
    {
        var global = "block variable" // затеняет глобальную
        var local = "inner block"     // затеняет функциональную
        
        fmt.Println(global) // "block variable"
        fmt.Println(local)  // "inner block"
    }
    
    fmt.Println(global) // "global variable" (снова доступна)
    fmt.Println(local)  // "function variable"
}

// Опасное shadowing с коротким объявлением
func dangerousShadowing() error {
    var err error
    
    if condition {
        data, err := readData() // err затенена!
        process(data)
        // локальная err исчезает здесь
    }
    
    return err // возвращает оригинальную err (nil)
}

// Правильный вариант
func correctVersion() error {
    var err error
    
    if condition {
        var data []byte
        data, err = readData() // используем оригинальную err
        process(data)
    }
    
    return err
}
```

### 20. Как работает `iota`?

**Развернутый ответ:**
`iota` — предопределенный идентификатор для создания последовательности констант:

```go
// Простой счетчик
const (
    Sunday = iota  // 0
    Monday         // 1
    Tuesday        // 2
    Wednesday      // 3
)

// Пропуск значений
const (
    _ = iota      // 0 (игнорируем)
    KB = 1 << (10 * iota) // 1024
    MB                    // 1048576
    GB                    // 1073741824
)

// Группированные константы
const (
    Read = 1 << iota   // 1 (001)
    Write              // 2 (010)
    Execute            // 4 (100)
)

// Сброс iota в новом блоке const
const (
    Apple = iota   // 0
    Orange         // 1
)

const (
    Red = iota     // 0 (сброс)
    Green          // 1
    Blue           // 2
)

// Сложные выражения
const (
    First = iota * 2 + 1  // 1
    Second                // 3
    Third                 // 5
)

// Использование в реальном коде
type Status int

const (
    StatusPending Status = iota
    StatusRunning
    StatusCompleted
    StatusFailed
)

func (s Status) String() string {
    names := []string{"Pending", "Running", "Completed", "Failed"}
    if s < 0 || int(s) >= len(names) {
        return "Unknown"
    }
    return names[s]
}
```

---

## 📊 Раздел 2: Массивы, слайсы, мапы (Ответы 21-35)

### 21. В чем разница между массивом и слайсом?

**Развернутый ответ:**

**Массив:**
- Фиксированный размер, часть типа
- Передается по значению (копируется)
- Размер известен на этапе компиляции
- Хранится в стеке (если небольшой)

**Слайс:**
- Динамический размер
- Передается по ссылке (указатель на данные)
- Размер может изменяться в runtime
- Ссылается на массив в heap

```go
// Массивы
var arr1 [5]int                    // массив из 5 элементов
var arr2 [10]int                   // другой тип!
// arr1 = arr2                     // Error: разные типы

func modifyArray(arr [5]int) {
    arr[0] = 100 // изменяет копию
}

// Слайсы
var slice1 []int                   // nil slice
var slice2 = make([]int, 5)        // slice длиной 5
var slice3 = []int{1, 2, 3}        // slice литерал

func modifySlice(s []int) {
    s[0] = 100 // изменяет оригинальные данные
}

func example() {
    arr := [3]int{1, 2, 3}
    slice := []int{1, 2, 3}
    
    modifyArray(arr)   // arr не изменился
    modifySlice(slice) // slice[0] == 100
}
```

### 22. Как устроен слайс внутри?

**Развернутый ответ:**
Слайс состоит из трех полей:
- **ptr** — указатель на данные в массиве
- **len** — текущая длина слайса
- **cap** — емкость (максимальная длина без реаллокации)

```go
// Внутренняя структура слайса
type slice struct {
    array unsafe.Pointer // указатель на данные
    len   int            // длина
    cap   int            // емкость
}

func sliceInternals() {
    // Создание слайса
    s := make([]int, 3, 5) // len=3, cap=5
    s = []int{1, 2, 3}
    
    fmt.Printf("len=%d cap=%d\n", len(s), cap(s))
    
    // Подслайсы разделяют данные
    s1 := s[1:3]  // {2, 3}, len=2, cap=4
    s2 := s[:2]   // {1, 2}, len=2, cap=5
    
    s1[0] = 100   // изменяет оригинальный массив
    fmt.Println(s) // [1, 100, 3]
}

// Визуализация
func visualizeSlice() {
    arr := [8]int{0, 1, 2, 3, 4, 5, 6, 7}
    slice := arr[2:5]
    
    // slice указывает на arr[2:5]
    // ptr -> arr[2]
    // len = 3 (элементы 2, 3, 4)
    // cap = 6 (до конца массива)
}
```

### 23. Что произойдет в этом коде?

```go
s1 := []int{1, 2, 3}
s2 := s1[1:2]
s2[0] = 999
fmt.Println(s1) // что выведет?
```

**Развернутый ответ:**
Выведет `[1 999 3]`, потому что слайсы разделяют один базовый массив.

```go
func sliceSharing() {
    s1 := []int{1, 2, 3}
    s2 := s1[1:2]    // s2 = [2], но указывает на s1[1]
    
    // Изменение s2 влияет на s1
    s2[0] = 999
    fmt.Println(s1)  // [1 999 3]
    fmt.Println(s2)  // [999]
    
    // Как избежать:
    s3 := make([]int, len(s2))
    copy(s3, s2)     // s3 - независимая копия
    s3[0] = 777
    fmt.Println(s1)  // [1 999 3] (не изменился)
    fmt.Println(s3)  // [777]
}

// Еще один пример
func appendDanger() {
    original := []int{1, 2, 3, 4, 5}
    sub := original[0:3]  // [1, 2, 3], cap=5
    
    sub = append(sub, 999) // перезаписывает original[3]
    fmt.Println(original)  // [1 2 3 999 5]
}
```

### 24. Когда происходит реаллокация слайса?

**Развернутый ответ:**
Реаллокация происходит при `append`, когда `len` превышает `cap`:

**Стратегия роста:**
- При cap < 1024: удваивание
- При cap >= 1024: увеличение на 25%

```go
func reallocationDemo() {
    var s []int
    
    for i := 0; i < 20; i++ {
        oldCap := cap(s)
        s = append(s, i)
        newCap := cap(s)
        
        if newCap != oldCap {
            fmt.Printf("Reallocation: %d -> %d\n", oldCap, newCap)
        }
    }
    // Вывод: 0 -> 1, 1 -> 2, 2 -> 4, 4 -> 8, 8 -> 16
}

// Предотвращение реаллокаций
func efficientAppend() {
    // Плохо: множественные реаллокации
    var s []int
    for i := 0; i < 1000; i++ {
        s = append(s, i) // реаллокация несколько раз
    }
    
    // Хорошо: заранее известна емкость
    s2 := make([]int, 0, 1000) // cap=1000
    for i := 0; i < 1000; i++ {
        s2 = append(s2, i) // без реаллокаций
    }
}

// Потеря ссылки при реаллокации
func reallocationProblem() {
    s1 := make([]int, 0, 2)
    s1 = append(s1, 1, 2)
    
    s2 := s1        // s2 указывает на тот же массив
    s1 = append(s1, 3) // реаллокация! s1 указывает на новый массив
    
    s2[0] = 999     // изменяет старый массив
    fmt.Println(s1) // [1 2 3] (не изменился)
    fmt.Println(s2) // [999 2]
}
```

### 25. Как правильно скопировать слайс?

**Развернутый ответ:**
Есть несколько способов копирования слайсов:

```go
func copySlice() {
    src := []int{1, 2, 3, 4, 5}
    
    // Способ 1: copy() - самый эффективный
    dst1 := make([]int, len(src))
    copy(dst1, src)
    
    // Способ 2: append с nil
    dst2 := append([]int(nil), src...)
    
    // Способ 3: append с пустым слайсом
    dst3 := append([]int{}, src...)
    
    // Способ 4: range (менее эффективно)
    dst4 := make([]int, len(src))
    for i, v := range src {
        dst4[i] = v
    }
    
    // Проверка независимости
    dst1[0] = 999
    fmt.Println(src)  // [1 2 3 4 5] (не изменился)
    fmt.Println(dst1) // [999 2 3 4 5]
}

// Копирование подслайса
func copySubslice() {
    src := []int{1, 2, 3, 4, 5}
    
    // Копирование части
    sub := make([]int, 3)
    copy(sub, src[1:4]) // копирует [2, 3, 4]
    
    // copy возвращает количество скопированных элементов
    n := copy(sub, src)
    fmt.Printf("Copied %d elements\n", n)
}

// Глубокое копирование слайса слайсов
func deepCopySliceOfSlices() {
    src := [][]int{
        {1, 2}, {3, 4}, {5, 6},
    }
    
    dst := make([][]int, len(src))
    for i, inner := range src {
        dst[i] = append([]int(nil), inner...)
    }
}
```

### 26. Что такое nil slice и empty slice? В чем разница?

**Развернутый ответ:**

**Nil slice:**
- Не указывает на массив
- len=0, cap=0
- Равен nil

**Empty slice:**
- Указывает на пустой массив
- len=0, cap может быть > 0
- Не равен nil

```go
func nilVsEmpty() {
    // Nil slice
    var nilSlice []int
    
    // Empty slices
    emptySlice1 := []int{}
    emptySlice2 := make([]int, 0)
    emptySlice3 := make([]int, 0, 5)
    
    fmt.Printf("nilSlice: %v, len=%d, cap=%d, nil=%t\n", 
        nilSlice, len(nilSlice), cap(nilSlice), nilSlice == nil)
    // nilSlice: [], len=0, cap=0, nil=true
    
    fmt.Printf("emptySlice1: %v, len=%d, cap=%d, nil=%t\n",
        emptySlice1, len(emptySlice1), cap(emptySlice1), emptySlice1 == nil)
    // emptySlice1: [], len=0, cap=0, nil=false
    
    // Все ведут себя одинаково при append
    nilSlice = append(nilSlice, 1)
    emptySlice1 = append(emptySlice1, 1)
    
    // JSON маршалинг различается
    nilJSON, _ := json.Marshal(nilSlice)     // "null"
    emptyJSON, _ := json.Marshal(emptySlice1) // "[]"
}

// Предпочтительные паттерны
func preferredPatterns() {
    // Для возврата пустой коллекции предпочтительнее nil
    func getUsers() []User {
        if noUsers {
            return nil // предпочтительнее return []User{}
        }
        return users
    }
    
    // Для инициализации с известной емкостью
    items := make([]Item, 0, expectedSize)
}
```

### 27. Как работает `append` со слайсами?

**Развернутый ответ:**
`append` добавляет элементы в конец слайса, возвращая новый слайс:

```go
func appendBehavior() {
    // Базовое использование
    slice := []int{1, 2, 3}
    slice = append(slice, 4, 5)        // [1, 2, 3, 4, 5]
    slice = append(slice, []int{6, 7}...) // распаковка слайса
    
    // append всегда возвращает новый слайс (может быть тот же базовый массив)
    original := make([]int, 3, 10) // len=3, cap=10
    original[0], original[1], original[2] = 1, 2, 3
    
    result := append(original, 4) // использует тот же массив
    fmt.Printf("Same array: %t\n", &original[0] == &result[0]) // true
    
    // Когда происходит реаллокация
    small := []int{1, 2}
    fmt.Printf("Before: cap=%d\n", cap(small)) // cap=2
    
    small = append(small, 3) // реаллокация
    fmt.Printf("After: cap=%d\n", cap(small))  // cap=4
}

// Append к nil slice
func appendToNil() {
    var slice []int // nil slice
    slice = append(slice, 1, 2, 3) // работает!
    fmt.Println(slice) // [1, 2, 3]
}

// Опасности append
func appendDangers() {
    // Проблема 1: потеря изменений при реаллокации
    func modifySlice(s []int) {
        s = append(s, 999) // может создать новый массив
        s[0] = 777         // изменяет локальную копию
    }
    
    slice := []int{1, 2, 3}
    modifySlice(slice)
    fmt.Println(slice) // [1, 2, 3] - не изменился
    
    // Решение: возвращать слайс
    func correctModify(s []int) []int {
        s = append(s, 999)
        s[0] = 777
        return s
    }
}

// Эффективное использование append
func efficientAppend() {
    // Предварительная аллокация
    result := make([]int, 0, expectedSize)
    for _, item := range items {
        result = append(result, process(item))
    }
    
    // Batch append
    var batch []int
    for _, item := range items {
        batch = append(batch, item)
        if len(batch) >= batchSize {
            processBatch(batch)
            batch = batch[:0] // сброс длины, сохраняем емкость
        }
    }
}
```

### 28. Как правильно удалить элемент из слайса?

**Развернутый ответ:**
Есть несколько паттернов удаления элементов:

```go
// Удаление по индексу с сохранением порядка
func removeWithOrder(slice []int, index int) []int {
    if index < 0 || index >= len(slice) {
        return slice
    }
    return append(slice[:index], slice[index+1:]...)
}

// Удаление по индексу без сохранения порядка (быстрее)
func removeSwapLast(slice []int, index int) []int {
    if index < 0 || index >= len(slice) {
        return slice
    }
    slice[index] = slice[len(slice)-1] // заменяем последним
    return slice[:len(slice)-1]        // убираем последний
}

// Удаление нескольких элементов
func removeRange(slice []int, start, end int) []int {
    return append(slice[:start], slice[end:]...)
}

// Удаление по значению
func removeValue(slice []int, value int) []int {
    for i, v := range slice {
        if v == value {
            return removeWithOrder(slice, i)
        }
    }
    return slice
}

// Удаление всех вхождений значения
func removeAllValues(slice []int, value int) []int {
    result := slice[:0] // переиспользуем базовый массив
    for _, v := range slice {
        if v != value {
            result = append(result, v)
        }
    }
    return result
}

// Фильтрация с предикатом
func filter(slice []int, predicate func(int) bool) []int {
    result := slice[:0]
    for _, v := range slice {
        if predicate(v) {
            result = append(result, v)
        }
    }
    return result
}

func example() {
    nums := []int{1, 2, 3, 4, 5, 3, 6}
    
    // Удаляем элемент по индексу 2
    nums = removeWithOrder(nums, 2) // [1, 2, 4, 5, 3, 6]
    
    // Удаляем все тройки
    nums = removeAllValues(nums, 3) // [1, 2, 4, 5, 6]
    
    // Оставляем только четные
    nums = filter(nums, func(n int) bool { return n%2 == 0 })
}
```

### 29. Что такое string slicing и как он работает?

**Развернутый ответ:**
Строки в Go неизменяемы и состоят из байтов UTF-8. String slicing создает новую строку:

```go
func stringSlicing() {
    s := "Hello, 世界"
    
    // Слайсинг по байтам
    sub := s[0:5]   // "Hello" (5 байт)
    sub = s[7:]     // "世界" (6 байт - по 3 байта на символ)
    
    // Осторожно с Unicode!
    fmt.Println(len(s))           // 13 (байт)
    fmt.Println(utf8.RuneCountInString(s)) // 9 (символов)
    
    // Неправильно: может разрезать UTF-8 символ
    // bad := s[7:8] // может дать невалидный UTF-8
    
    // Правильная работа с Unicode
    runes := []rune(s)
    unicodeSafe := string(runes[7:9]) // "世界"
}

// Эффективная работа со строками
func stringOperations() {
    // Конкатенация - неэффективно для множественных операций
    result := ""
    for _, word := range words {
        result += word + " " // создает новую строку каждый раз
    }
    
    // Эффективно: strings.Builder
    var builder strings.Builder
    for _, word := range words {
        builder.WriteString(word)
        builder.WriteString(" ")
    }
    result = builder.String()
    
    // Или strings.Join
    result = strings.Join(words, " ")
}

// Поиск подстрок
func stringSearch() {
    text := "The quick brown fox"
    
    // Проверка префикса/суффикса
    if strings.HasPrefix(text, "The") {
        fmt.Println("Starts with 'The'")
    }
    
    // Поиск подстроки
    if index := strings.Index(text, "quick"); index != -1 {
        fmt.Printf("Found 'quick' at position %d\n", index)
    }
    
    // Разделение
    words := strings.Split(text, " ")
    fmt.Println(words) // ["The", "quick", "brown", "fox"]
}
```

### 30. Как устроены мапы в Go?

**Развернутый ответ:**
Мапы в Go реализованы как хеш-таблицы с открытой адресацией:

**Внутреннее устройство:**
- Bucket-based структура
- Каждый bucket содержит до 8 key-value пар
- При коллизиях используется overflow buckets
- Динамическое увеличение размера

```go
// Внутренняя структура (упрощенно)
type hmap struct {
    count     int    // количество элементов
    flags     uint8  // флаги состояния
    B         uint8  // log_2 числа buckets
    noverflow uint16 // overflow buckets
    hash0     uint32 // seed для hash функции
    buckets   unsafe.Pointer // массив buckets
    oldbuckets unsafe.Pointer // для incremental resizing
}

type bmap struct {
    tophash [bucketCnt]uint8  // высшие 8 бит хешей
    // keys и values хранятся отдельно для лучшего выравнивания
    keys   [bucketCnt]keytype
    values [bucketCnt]valuetype
    overflow *bmap            // указатель на overflow bucket
}

// Создание и инициализация
func mapOperations() {
    // Различные способы создания
    var m1 map[string]int                    // nil map
    m2 := make(map[string]int)               // пустая map
    m3 := make(map[string]int, 100)          // с начальной емкостью
    m4 := map[string]int{"a": 1, "b": 2}     // map literal
    
    // Проверка существования ключа
    value, ok := m2["key"]
    if ok {
        fmt.Printf("Value: %d\n", value)
    }
}
```

### 31. Что произойдет при обращении к несуществующему ключу?

**Развернутый ответ:**
При обращении к несуществующему ключу возвращается zero value типа значения:

```go
func mapAccess() {
    m := map[string]int{"a": 1, "b": 2}
    
    // Обращение к несуществующему ключу
    value := m["c"]        // 0 (zero value для int)
    fmt.Println(value)     // 0
    
    // Безопасная проверка существования
    value, ok := m["c"]
    if ok {
        fmt.Printf("Found: %d\n", value)
    } else {
        fmt.Println("Key not found")
    }
    
    // Различие между "ключ не существует" и "значение равно zero value"
    m["zero"] = 0
    
    fmt.Println(m["zero"])     // 0
    fmt.Println(m["missing"])  // 0 тоже!
    
    // Правильная проверка
    if val, exists := m["zero"]; exists {
        fmt.Printf("Key exists with value: %d\n", val)
    }
    
    if val, exists := m["missing"]; exists {
        fmt.Printf("Key exists with value: %d\n", val)
    } else {
        fmt.Println("Key does not exist")
    }
}

// Работа с разными типами
func differentValueTypes() {
    // Для указателей
    m1 := make(map[string]*int)
    ptr := m1["missing"] // nil
    
    // Для слайсов
    m2 := make(map[string][]int)
    slice := m2["missing"] // nil slice
    
    // Для структур
    type User struct { Name string; Age int }
    m3 := make(map[string]User)
    user := m3["missing"] // User{Name: "", Age: 0}
}
```

### 32. Как правильно проверить существование ключа в мапе?

**Развернутый ответ:**
Go предоставляет идиому "comma ok" для проверки существования ключей:

```go
func keyExistence() {
    m := map[string]int{
        "exists": 42,
        "zero":   0,
    }
    
    // Правильная проверка существования
    if value, ok := m["exists"]; ok {
        fmt.Printf("Key exists: %d\n", value)
    }
    
    // Проверка без использования значения
    if _, ok := m["exists"]; ok {
        fmt.Println("Key exists")
    }
    
    // Удаление ключа
    delete(m, "exists")
    if _, ok := m["exists"]; !ok {
        fmt.Println("Key deleted")
    }
}

// Паттерны проверки
func checkPatterns() {
    cache := make(map[string]string)
    
    // Паттерн 1: проверка и использование
    if value, found := cache["key"]; found {
        return value
    }
    
    // Паттерн 2: проверка существования для логики
    _, userExists := users["john"]
    _, adminExists := admins["john"]
    
    if userExists && !adminExists {
        promoteToAdmin("john")
    }
    
    // Паттерн 3: установка значения по умолчанию
    if _, ok := config["timeout"]; !ok {
        config["timeout"] = "30s"
    }
}

// Безопасная работа с вложенными картами
func nestedMaps() {
    data := make(map[string]map[string]int)
    
    // Небезопасно
    // data["outer"]["inner"] = 42 // panic если data["outer"] == nil
    
    // Безопасно
    if inner, ok := data["outer"]; ok {
        inner["inner"] = 42
    } else {
        data["outer"] = map[string]int{"inner": 42}
    }
    
    // Еще более безопасно с helper функцией
    func setNested(outer, inner string, value int) {
        if data[outer] == nil {
            data[outer] = make(map[string]int)
        }
        data[outer][inner] = value
    }
}
```

### 33. Потокобезопасны ли мапы в Go?

**Развернутый ответ:**
Обычные мапы в Go НЕ потокобезопасны. Одновременное чтение и запись вызывает fatal error:

```go
// НЕБЕЗОПАСНО: может вызвать fatal error
func unsafeMapAccess() {
    m := make(map[int]int)
    
    // Goroutine 1: запись
    go func() {
        for i := 0; i < 1000; i++ {
            m[i] = i
        }
    }()
    
    // Goroutine 2: чтение
    go func() {
        for i := 0; i < 1000; i++ {
            _ = m[i]
        }
    }()
    
    time.Sleep(time.Second)
    // Может вывести: fatal error: concurrent map read and map write
}

// Решение 1: Mutex
type SafeMap struct {
    mu sync.RWMutex
    m  map[string]int
}

func NewSafeMap() *SafeMap {
    return &SafeMap{
        m: make(map[string]int),
    }
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

func (sm *SafeMap) Delete(key string) {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    delete(sm.m, key)
}

// Решение 2: sync.Map (для специальных случаев)
func syncMapExample() {
    var sm sync.Map
    
    // Запись
    sm.Store("key", "value")
    
    // Чтение
    if value, ok := sm.Load("key"); ok {
        fmt.Printf("Value: %v\n", value)
    }
    
    // Удаление
    sm.Delete("key")
    
    // Итерация
    sm.Range(func(key, value interface{}) bool {
        fmt.Printf("%v: %v\n", key, value)
        return true // продолжить итерацию
    })
}

// Решение 3: Каналы для координации
func channelBasedMap() {
    type MapOp struct {
        key    string
        value  int
        result chan int
        op     string // "get", "set", "delete"
    }
    
    ops := make(chan MapOp)
    
    // Map manager goroutine
    go func() {
        m := make(map[string]int)
        for op := range ops {
            switch op.op {
            case "get":
                op.result <- m[op.key]
            case "set":
                m[op.key] = op.value
                close(op.result)
            case "delete":
                delete(m, op.key)
                close(op.result)
            }
        }
    }()
    
    // Использование
    getFromMap := func(key string) int {
        result := make(chan int)
        ops <- MapOp{key: key, result: result, op: "get"}
        return <-result
    }
}
```

### 34. Как правильно итерироваться по мапе?

**Развернутый ответ:**
Итерация по мапе в Go имеет особенности:

```go
func mapIteration() {
    m := map[string]int{
        "apple":  5,
        "banana": 3,
        "cherry": 8,
    }
    
    // Базовая итерация (порядок НЕ гарантирован!)
    for key, value := range m {
        fmt.Printf("%s: %d\n", key, value)
    }
    
    // Только ключи
    for key := range m {
        fmt.Printf("Key: %s\n", key)
    }
    
    // Только значения
    for _, value := range m {
        fmt.Printf("Value: %d\n", value)
    }
}

// Сортированная итерация
func sortedIteration() {
    m := map[string]int{
        "zebra": 1, "apple": 2, "banana": 3,
    }
    
    // Получаем ключи и сортируем
    keys := make([]string, 0, len(m))
    for key := range m {
        keys = append(keys, key)
    }
    sort.Strings(keys)
    
    // Итерируемся в отсортированном порядке
    for _, key := range keys {
        fmt.Printf("%s: %d\n", key, m[key])
    }
}

// Безопасная модификация во время итерации
func safeModification() {
    m := map[string]int{"a": 1, "b": 2, "c": 3}
    
    // НЕБЕЗОПАСНО: модификация во время итерации
    // for key := range m {
    //     if key == "b" {
    //         delete(m, key) // может пропустить элементы
    //     }
    // }
    
    // БЕЗОПАСНО: собираем ключи для удаления
    var toDelete []string
    for key, value := range m {
        if value%2 == 0 {
            toDelete = append(toDelete, key)
        }
    }
    
    // Удаляем после итерации
    for _, key := range toDelete {
        delete(m, key)
    }
}

// Итерация с контролем
func controlledIteration() {
    m := map[string]int{
        "one": 1, "two": 2, "three": 3, "four": 4,
    }
    
    // Прерывание итерации
    found := false
    for key, value := range m {
        if value == 3 {
            fmt.Printf("Found: %s\n", key)
            found = true
            break
        }
    }
    
    // Подсчет во время итерации
    count := 0
    sum := 0
    for _, value := range m {
        count++
        sum += value
        if count >= 2 { // ограничиваем количество
            break
        }
    }
}

// Параллельная обработка элементов мапы
func parallelProcessing() {
    m := map[string]int{
        "task1": 1, "task2": 2, "task3": 3,
    }
    
    var wg sync.WaitGroup
    
    for key, value := range m {
        wg.Add(1)
        go func(k string, v int) {
            defer wg.Done()
            // Обработка элемента
            processItem(k, v)
        }(key, value) // важно: копируем значения
    }
    
    wg.Wait()
}
```

### 35. В чем проблема этого кода?

```go
var m map[string]int
m["key"] = 1 // runtime panic!
```

**Развернутый ответ:**
Проблема в том, что попытка записи в nil map вызывает runtime panic:

```go
func mapInitialization() {
    // Проблема: nil map
    var m map[string]int
    fmt.Printf("m == nil: %t\n", m == nil) // true
    
    // Чтение из nil map возвращает zero value
    value := m["key"] // 0, но не panic
    fmt.Printf("Reading from nil map: %d\n", value)
    
    // Запись в nil map вызывает panic
    // m["key"] = 1 // panic: assignment to entry in nil map
    
    // Правильная инициализация
    m = make(map[string]int)
    m["key"] = 1 // OK
    
    // Или инициализация с литералом
    m2 := map[string]int{}
    m2["key"] = 1 // OK
    
    // Или с начальными значениями
    m3 := map[string]int{"key": 1}
    fmt.Println(m3["key"]) // 1
}

// Паттерны безопасной работы
func safeMapPatterns() {
    // Паттерн 1: lazy initialization
    var cache map[string]string
    
    func getFromCache(key string) string {
        if cache == nil {
            cache = make(map[string]string)
        }
        return cache[key]
    }
    
    func setToCache(key, value string) {
        if cache == nil {
            cache = make(map[string]string)
        }
        cache[key] = value
    }
    
    // Паттерн 2: проверка в начале функции
    func processMap(m map[string]int) error {
        if m == nil {
            return errors.New("map cannot be nil")
        }
        
        for key, value := range m {
            process(key, value)
        }
        return nil
    }
    
    // Паттерн 3: создание в конструкторе
    type Config struct {
        settings map[string]string
    }
    
    func NewConfig() *Config {
        return &Config{
            settings: make(map[string]string),
        }
    }
    
    func (c *Config) Set(key, value string) {
        c.settings[key] = value // безопасно
    }
}

// Различия между nil и пустой map
func nilVsEmptyMap() {
    var nilMap map[string]int
    emptyMap := make(map[string]int)
    
    fmt.Printf("nil map len: %d\n", len(nilMap))     // 0
    fmt.Printf("empty map len: %d\n", len(emptyMap)) // 0
    
    // Чтение работает в обоих случаях
    fmt.Printf("nil map read: %d\n", nilMap["key"])     // 0
    fmt.Printf("empty map read: %d\n", emptyMap["key"]) // 0
    
    // Проверка существования работает
    _, ok1 := nilMap["key"]   // false
    _, ok2 := emptyMap["key"] // false
    
    // Итерация работает (но ничего не выводит)
    for k, v := range nilMap {
        fmt.Printf("%s: %d\n", k, v) // не выполнится
    }
    
    // Только запись вызывает panic для nil map
    // nilMap["key"] = 1    // panic!
    emptyMap["key"] = 1     // OK
}
```

---

## 🚀 Раздел 3: Concurrency - Горутины и каналы (Ответы 36-55)

### 36. Что такое горутины и чем они отличаются от потоков ОС?

**Развернутый ответ:**
Горутины — это легковесные потоки, управляемые Go runtime:

**Отличия от OS threads:**
- **Размер стека**: 2KB (vs 2MB для OS thread)
- **Управление**: Go scheduler (vs OS scheduler)  
- **Создание**: очень быстрое (vs медленное)
- **Переключение контекста**: быстрое (vs медленное)
- **Количество**: миллионы (vs тысячи)

```go
func goroutineBasics() {
    // Создание горутины
    go func() {
        fmt.Println("Hello from goroutine!")
    }()
    
    // Горутина с параметрами
    name := "World"
    go func(n string) {
        fmt.Printf("Hello, %s!\n", n)
    }(name) // важно: передаем значение
    
    // Опасно: захват переменной цикла
    for i := 0; i < 3; i++ {
        go func() {
            fmt.Println(i) // может вывести 3, 3, 3
        }()
    }
    
    // Правильно
    for i := 0; i < 3; i++ {
        go func(id int) {
            fmt.Println(id) // выведет 0, 1, 2
        }(i)
    }
    
    time.Sleep(time.Second) // ждем завершения горутин
}

// Жизненный цикл горутины
func goroutineLifecycle() {
    // Горутина завершается когда:
    // 1. Функция возвращает значение
    // 2. Происходит panic (убивает всю программу)
    // 3. main функция завершается (убивает все горутины)
    
    done := make(chan bool)
    
    go func() {
        defer func() {
            done <- true // сигнализируем о завершении
        }()
        
        // работа горутины
        time.Sleep(500 * time.Millisecond)
        fmt.Println("Goroutine finished")
    }()
    
    <-done // ждем завершения
    fmt.Println("Main finished")
}

// Горутины vs OS threads производительность
func performanceComparison() {
    // Создание миллиона горутин (возможно)
    const numGoroutines = 1000000
    var wg sync.WaitGroup
    
    start := time.Now()
    
    for i := 0; i < numGoroutines; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            // минимальная работа
            _ = 1 + 1
        }()
    }
    
    wg.Wait()
    fmt.Printf("Created %d goroutines in %v\n", numGoroutines, time.Since(start))
    
    // Аналогичное с OS threads было бы невозможно
}
```

### 37. Как создать и запустить горутину?

**Развернутый ответ:**
Горутины создаются с помощью ключевого слова `go`:

```go
// Основные способы создания
func createGoroutines() {
    // 1. Анонимная функция
    go func() {
        fmt.Println("Anonymous goroutine")
    }()
    
    // 2. Именованная функция
    go namedFunction()
    
    // 3. Метод структуры
    worker := &Worker{id: 1}
    go worker.Process()
    
    // 4. С параметрами
    go processData("input", 42)
}

type Worker struct {
    id int
}

func (w *Worker) Process() {
    fmt.Printf("Worker %d processing\n", w.id)
}

func namedFunction() {
    fmt.Println("Named function goroutine")
}

func processData(input string, count int) {
    fmt.Printf("Processing %s %d times\n", input, count)
}

// Паттерны запуска горутин
func goroutinePatterns() {
    // Паттерн 1: Fire and forget
    go logEvent("User logged in")
    
    // Паттерн 2: Worker pool
    jobs := make(chan Job, 100)
    results := make(chan Result, 100)
    
    // Запускаем workers
    for w := 1; w <= 3; w++ {
        go worker(w, jobs, results)
    }
    
    // Паттерн 3: Координированный запуск
    var wg sync.WaitGroup
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            processTask(id)
        }(i)
    }
    wg.Wait()
    
    // Паттерн 4: С контекстом для отмены
    ctx, cancel := context.WithCancel(context.Background())
    go backgroundWorker(ctx)
    
    time.Sleep(5 * time.Second)
    cancel() // останавливаем worker
}

func worker(id int, jobs <-chan Job, results chan<- Result) {
    for job := range jobs {
        result := processJob(job)
        results <- result
    }
}

func backgroundWorker(ctx context.Context) {
    ticker := time.NewTicker(time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            doWork()
        case <-ctx.Done():
            fmt.Println("Worker stopped")
            return
        }
    }
}
```

### 38. Что такое каналы и как они работают?

**Развернутый ответ:**
Каналы — это типизированные conduits для передачи данных между горутинами, реализующие CSP (Communicating Sequential Processes):

```go
// Создание каналов
func channelBasics() {
    // Небуферизованный канал
    ch1 := make(chan int)
    
    // Буферизованный канал
    ch2 := make(chan string, 10)
    
    // Только для чтения/записи
    var readOnly <-chan int = ch1
    var writeOnly chan<- int = ch1
    
    // Отправка и получение
    go func() {
        ch1 <- 42        // отправка (блокируется до получения)
        ch2 <- "hello"   // отправка в буфер
    }()
    
    value := <-ch1       // получение (блокируется до отправки)
    message := <-ch2     // получение из буфера
    
    fmt.Printf("Received: %d, %s\n", value, message)
}

// Синхронизация через каналы
func channelSynchronization() {
    done := make(chan bool)
    
    go func() {
        time.Sleep(2 * time.Second)
        fmt.Println("Work completed")
        done <- true // сигнал о завершении
    }()
    
    fmt.Println("Waiting for work to complete...")
    <-done // блокируемся до получения сигнала
    fmt.Println("All done!")
}

// Передача данных
func dataTransfer() {
    numbers := make(chan int, 5)
    results := make(chan int, 5)
    
    // Producer
    go func() {
        defer close(numbers)
        for i := 1; i <= 5; i++ {
            numbers <- i
        }
    }()
    
    // Processor
    go func() {
        defer close(results)
        for num := range numbers {
            results <- num * num // возводим в квадрат
        }
    }()
    
    // Consumer
    for result := range results {
        fmt.Printf("Result: %d\n", result)
    }
}

// Неблокирующие операции
func nonBlockingOperations() {
    ch := make(chan int, 1)
    
    // Неблокирующая отправка
    select {
    case ch <- 42:
        fmt.Println("Sent successfully")
    default:
        fmt.Println("Channel full, couldn't send")
    }
    
    // Неблокирующее получение
    select {
    case value := <-ch:
        fmt.Printf("Received: %d\n", value)
    default:
        fmt.Println("No data available")
    }
}
```

### 39. В чем разница между буферизованными и небуферизованными каналами?

**Развернутый ответ:**

**Небуферизованный канал:**
- Синхронная передача данных
- Отправка блокируется до получения
- Получение блокируется до отправки
- Гарантирует синхронизацию

**Буферизованный канал:**
- Асинхронная передача (до заполнения буфера)
- Отправка блокируется только при полном буфере
- Получение блокируется только при пустом буфере

```go
func bufferingComparison() {
    // Небуферизованный - требует синхронизации
    unbuffered := make(chan int)
    
    go func() {
        fmt.Println("Sending to unbuffered...")
        unbuffered <- 1 // блокируется до получения
        fmt.Println("Sent to unbuffered")
    }()
    
    time.Sleep(100 * time.Millisecond) // горутина заблокирована
    value := <-unbuffered // разблокируем отправку
    fmt.Printf("Received from unbuffered: %d\n", value)
    
    // Буферизованный - позволяет асинхронность
    buffered := make(chan int, 2)
    
    fmt.Println("Sending to buffered...")
    buffered <- 1 // не блокируется
    buffered <- 2 // не блокируется
    fmt.Println("Sent to buffered (no blocking)")
    
    // buffered <- 3 // заблокировалось бы (буфер полон)
    
    fmt.Printf("Received: %d\n", <-buffered)
    fmt.Printf("Received: %d\n", <-buffered)
}

// Использование буферизации для производительности
func bufferPerformance() {
    const numMessages = 1000000
    
    // Небуферизованный - медленнее
    start := time.Now()
    unbuffered := make(chan int)
    
    go func() {
        for i := 0; i < numMessages; i++ {
            unbuffered <- i
        }
        close(unbuffered)
    }()
    
    for range unbuffered {
        // обработка
    }
    unbufferedTime := time.Since(start)
    
    // Буферизованный - быстрее
    start = time.Now()
    buffered := make(chan int, 1000) // буфер уменьшает синхронизацию
    
    go func() {
        for i := 0; i < numMessages; i++ {
            buffered <- i
        }
        close(buffered)
    }()
    
    for range buffered {
        // обработка
    }
    bufferedTime := time.Since(start)
    
    fmt.Printf("Unbuffered: %v, Buffered: %v\n", unbufferedTime, bufferedTime)
}

// Семантика буферизации
func bufferSemantics() {
    // Размер буфера = 0 (небуферизованный)
    sync := make(chan int)    // каждая операция требует синхронизации
    
    // Размер буфера = 1 (семафор)
    semaphore := make(chan int, 1) // может хранить одно значение
    
    // Размер буфера > 1 (очередь)
    queue := make(chan int, 10)    // может хранить 10 значений
    
    // Примеры использования
    
    // Семафор для ограничения доступа
    semaphore <- 1 // захватываем ресурс
    // критическая секция
    <-semaphore    // освобождаем ресурс
    
    // Очередь для batch обработки
    for i := 0; i < 5; i++ {
        queue <- i // заполняем очередь
    }
    
    // Обрабатываем batch
    for i := 0; i < 5; i++ {
        item := <-queue
        process(item)
    }
}
```

### 40. Как закрыть канал и зачем это нужно?

**Развернутый ответ:**
Закрытие канала сигнализирует получателям, что больше данных не будет:

```go
func channelClosing() {
    // Основы закрытия
    ch := make(chan int, 2)
    
    // Отправка данных
    ch <- 1
    ch <- 2
    
    // Закрытие канала
    close(ch) // только отправитель должен закрывать канал
    
    // Чтение из закрытого канала
    v1 := <-ch          // 1 (нормальное чтение)
    v2 := <-ch          // 2 (нормальное чтение)
    v3 := <-ch          // 0 (zero value)
    v4, ok := <-ch      // 0, false (канал закрыт и пуст)
    
    fmt.Printf("Values: %d, %d, %d, %d (ok=%t)\n", v1, v2, v3, v4, ok)
}

// Зачем нужно закрытие
func whyClose() {
    // 1. Завершение range цикла
    numbers := make(chan int)
    
    go func() {
        defer close(numbers) // важно: закрываем канал
        for i := 1; i <= 5; i++ {
            numbers <- i
        }
    }()
    
    // range завершится когда канал закроется
    for num := range numbers {
        fmt.Printf("Number: %d\n", num)
    }
    
    // 2. Сигнализация о завершении работы
    done := make(chan struct{})
    
    go func() {
        defer close(done) // сигнал о завершении
        // выполняем работу
        time.Sleep(time.Second)
    }()
    
    <-done // ждем завершения
    
    // 3. Graceful shutdown
    quit := make(chan struct{})
    
    go func() {
        // рабочий цикл
        for {
            select {
            case <-quit:
                fmt.Println("Worker stopping...")
                return
            default:
                // работа
                time.Sleep(100 * time.Millisecond)
            }
        }
    }()
    
    time.Sleep(500 * time.Millisecond)
    close(quit) // сигнал остановки
}

// Паттерны с закрытием
func closingPatterns() {
    // Паттерн 1: Fan-out с закрытием
    input := make(chan int)
    output1 := make(chan int)
    output2 := make(chan int)
    
    // Fan-out worker
    go func() {
        defer close(output1)
        defer close(output2)
        
        for value := range input {
            output1 <- value
            output2 <- value * 2
        }
    }()
    
    // Отправляем данные и закрываем
    go func() {
        defer close(input)
        for i := 1; i <= 3; i++ {
            input <- i
        }
    }()
    
    // Читаем до закрытия
    for v := range output1 {
        fmt.Printf("Output1: %d\n", v)
    }
    
    for v := range output2 {
        fmt.Printf("Output2: %d\n", v)
    }
    
    // Паттерн 2: Pipeline с закрытием
    stage1 := make(chan int)
    stage2 := make(chan int)
    
    // Stage 1
    go func() {
        defer close(stage1)
        for i := 1; i <= 5; i++ {
            stage1 <- i
        }
    }()
    
    // Stage 2
    go func() {
        defer close(stage2)
        for num := range stage1 {
            stage2 <- num * num
        }
    }()
    
    // Final consumer
    for result := range stage2 {
        fmt.Printf("Final result: %d\n", result)
    }
}

// Ошибки при закрытии
func closingErrors() {
    ch := make(chan int)
    
    // Ошибка 1: отправка в закрытый канал
    close(ch)
    // ch <- 1 // panic: send on closed channel
    
    // Ошибка 2: повторное закрытие
    // close(ch) // panic: close of closed channel
    
    // Ошибка 3: закрытие nil канала
    var nilCh chan int
    // close(nilCh) // panic: close of nil channel
    
    // Правильная проверка закрытия
    func safeSend(ch chan<- int, value int) (sent bool) {
        defer func() {
            if recover() != nil {
                sent = false
            }
        }()
        
        ch <- value
        return true
    }
}
```

### 41. Что происходит при чтении из закрытого канала?

**Развернутый ответ:**
Чтение из закрытого канала имеет специальную семантику:

```go
func readFromClosedChannel() {
    ch := make(chan int, 3)
    
    // Отправляем данные
    ch <- 1
    ch <- 2
    ch <- 3
    
    // Закрываем канал
    close(ch)
    
    // Читаем оставшиеся данные
    fmt.Println(<-ch) // 1
    fmt.Println(<-ch) // 2
    fmt.Println(<-ch) // 3
    
    // Читаем из пустого закрытого канала
    fmt.Println(<-ch) // 0 (zero value для int)
    
    // Проверяем статус канала
    value, ok := <-ch
    fmt.Printf("Value: %d, OK: %t\n", value, ok) // Value: 0, OK: false
}

// Различные типы и zero values
func zeroValuesFromClosed() {
    // Для разных типов zero values различаются
    intCh := make(chan int)
    close(intCh)
    fmt.Printf("int: %d\n", <-intCh) // 0
    
    stringCh := make(chan string)
    close(stringCh)
    fmt.Printf("string: '%s'\n", <-stringCh) // ""
    
    boolCh := make(chan bool)
    close(boolCh)
    fmt.Printf("bool: %t\n", <-boolCh) // false
    
    sliceCh := make(chan []int)
    close(sliceCh)
    slice := <-sliceCh
    fmt.Printf("slice: %v (nil: %t)\n", slice, slice == nil) // [] (nil: true)
    
    ptrCh := make(chan *int)
    close(ptrCh)
    ptr := <-ptrCh
    fmt.Printf("pointer: %v (nil: %t)\n", ptr, ptr == nil) // <nil> (nil: true)
}

// Паттерны безопасного чтения
func safeReadingPatterns() {
    ch := make(chan int, 2)
    ch <- 1
    ch <- 2
    close(ch)
    
    // Паттерн 1: проверка с ok
    for {
        value, ok := <-ch
        if !ok {
            fmt.Println("Channel closed")
            break
        }
        fmt.Printf("Value: %d\n", value)
    }
    
    // Паттерн 2: range (автоматически завершается при закрытии)
    ch2 := make(chan int, 2)
    ch2 <- 10
    ch2 <- 20
    close(ch2)
    
    for value := range ch2 {
        fmt.Printf("Range value: %d\n", value)
    }
    // цикл автоматически завершается
    
    // Паттерн 3: select с проверкой
    ch3 := make(chan int)
    close(ch3)
    
    select {
    case value, ok := <-ch3:
        if ok {
            fmt.Printf("Received: %d\n", value)
        } else {
            fmt.Println("Channel closed in select")
        }
    default:
        fmt.Println("No data available")
    }
}

// Отличие от nil канала
func closedVsNilChannel() {
    // Nil канал
    var nilCh chan int
    
    // Закрытый канал
    closedCh := make(chan int)
    close(closedCh)
    
    // Поведение в select
    select {
    case <-nilCh:
        fmt.Println("Nil channel") // никогда не выполнится
    case <-closedCh:
        fmt.Println("Closed channel") // выполнится немедленно
    case <-time.After(100 * time.Millisecond):
        fmt.Println("Timeout")
    }
    
    // В цикле
    // for range nilCh {} // зависнет навсегда
    for range closedCh {} // завершится немедленно
}

// Практический пример: drain channel
func drainChannel() {
    ch := make(chan int, 10)
    
    // Заполняем канал
    for i := 0; i < 5; i++ {
        ch <- i
    }
    close(ch)
    
    // Полностью очищаем канал
    func drain(ch <-chan int) {
        for {
            select {
            case _, ok := <-ch:
                if !ok {
                    return // канал закрыт и пуст
                }
                // обрабатываем или игнорируем значение
            default:
                return // канал пуст
            }
        }
    }
    
    drain(ch)
    fmt.Println("Channel drained")
}
```

### 42. Как работает `select` statement?

**Развернутый ответ:**
`select` позволяет горутине ожидать несколько операций с каналами одновременно:

```go
func selectBasics() {
    ch1 := make(chan string)
    ch2 := make(chan string)
    
    go func() {
        time.Sleep(100 * time.Millisecond)
        ch1 <- "from ch1"
    }()
    
    go func() {
        time.Sleep(200 * time.Millisecond)
        ch2 <- "from ch2"
    }()
    
    // select выберет первый готовый case
    select {
    case msg1 := <-ch1:
        fmt.Println("Received:", msg1)
    case msg2 := <-ch2:
        fmt.Println("Received:", msg2)
    }
}

// Неблокирующие операции с default
func nonBlockingSelect() {
    ch := make(chan string, 1)
    
    // Неблокирующая отправка
    select {
    case ch <- "hello":
        fmt.Println("Sent successfully")
    default:
        fmt.Println("Channel full")
    }
    
    // Неблокирующее чтение
    select {
    case msg := <-ch:
        fmt.Println("Received:", msg)
    default:
        fmt.Println("No data available")
    }
}

// Timeout паттерн
func timeoutPattern() {
    ch := make(chan string)
    
    go func() {
        time.Sleep(2 * time.Second)
        ch <- "slow response"
    }()
    
    select {
    case result := <-ch:
        fmt.Println("Got result:", result)
    case <-time.After(1 * time.Second):
        fmt.Println("Timeout!")
    }
}

// Множественные операции
func multipleOperations() {
    input := make(chan int)
    output := make(chan int)
    quit := make(chan bool)
    
    go func() {
        for i := 0; i < 5; i++ {
            input <- i
            time.Sleep(100 * time.Millisecond)
        }
        close(input)
    }()
    
    for {
        select {
        case value, ok := <-input:
            if !ok {
                fmt.Println("Input channel closed")
                return
            }
            output <- value * 2
            
        case result := <-output:
            fmt.Printf("Processed: %d\n", result)
            
        case <-quit:
            fmt.Println("Quitting...")
            return
            
        default:
            // Выполняется если ни один case не готов
            time.Sleep(10 * time.Millisecond)
        }
    }
}

// Select в цикле для мультиплексирования
func selectMultiplexing() {
    ch1 := make(chan string)
    ch2 := make(chan string)
    done := make(chan bool)
    
    // Producer 1
    go func() {
        for i := 0; i < 3; i++ {
            ch1 <- fmt.Sprintf("ch1-%d", i)
            time.Sleep(300 * time.Millisecond)
        }
        close(ch1)
    }()
    
    // Producer 2
    go func() {
        for i := 0; i < 3; i++ {
            ch2 <- fmt.Sprintf("ch2-%d", i)
            time.Sleep(500 * time.Millisecond)
        }
        close(ch2)
    }()
    
    // Multiplexer
    go func() {
        openChannels := 2
        for openChannels > 0 {
            select {
            case msg, ok := <-ch1:
                if ok {
                    fmt.Println("From ch1:", msg)
                } else {
                    ch1 = nil // отключаем закрытый канал
                    openChannels--
                }
            case msg, ok := <-ch2:
                if ok {
                    fmt.Println("From ch2:", msg)
                } else {
                    ch2 = nil // отключаем закрытый канал
                    openChannels--
                }
            }
        }
        done <- true
    }()
    
    <-done
}

// Приоритетный select
func prioritySelect() {
    high := make(chan string, 10)
    low := make(chan string, 10)
    
    // Заполняем каналы
    high <- "high priority 1"
    high <- "high priority 2"
    low <- "low priority 1"
    low <- "low priority 2"
    
    for {
        select {
        case msg := <-high:
            fmt.Println("HIGH:", msg)
        default:
            // Только если high канал пуст, проверяем low
            select {
            case msg := <-low:
                fmt.Println("LOW:", msg)
            default:
                fmt.Println("No work available")
                return
            }
        }
    }
}

// Select с nil каналами
func selectWithNilChannels() {
    var ch1, ch2 chan string
    
    // nil каналы никогда не готовы в select
    select {
    case <-ch1: // никогда не выполнится
        fmt.Println("ch1")
    case <-ch2: // никогда не выполнится
        fmt.Println("ch2")
    default:
        fmt.Println("No channels ready") // выполнится
    }
    
    // Динамическое отключение каналов
    active1, active2 := true, true
    
    for active1 || active2 {
        var c1, c2 chan string
        
        if active1 {
            c1 = make(chan string, 1)
            c1 <- "message1"
        }
        if active2 {
            c2 = make(chan string, 1)
            c2 <- "message2"
        }
        
        select {
        case msg := <-c1:
            fmt.Println("From c1:", msg)
            active1 = false
        case msg := <-c2:
            fmt.Println("From c2:", msg)
            active2 = false
        }
    }
}
```

### 43. Что такое channel directions?

**Развернутый ответ:**
Channel directions ограничивают операции с каналами для безопасности типов:

```go
// Типы каналов по направлению
func channelDirections() {
    // Двунаправленный канал
    bidir := make(chan int)
    
    // Только для отправки
    var sendOnly chan<- int = bidir
    
    // Только для получения
    var receiveOnly <-chan int = bidir
    
    // sendOnly <- 42     // OK
    // receiveOnly <- 42  // Error: cannot send
    // <-sendOnly         // Error: cannot receive
    // <-receiveOnly      // OK
}

// Функции с направленными каналами
func producer(out chan<- int) {
    defer close(out)
    for i := 0; i < 5; i++ {
        out <- i
        time.Sleep(100 * time.Millisecond)
    }
}

func consumer(in <-chan int) {
    for value := range in {
        fmt.Printf("Consumed: %d\n", value)
    }
}

func pipeline() {
    numbers := make(chan int)
    
    go producer(numbers)  // может только отправлять
    consumer(numbers)     // может только получать
}

// Pipeline с обработкой
func processingPipeline() {
    // Stage 1: генерация
    func generate(out chan<- int) {
        defer close(out)
        for i := 1; i <= 10; i++ {
            out <- i
        }
    }
    
    // Stage 2: обработка
    func process(in <-chan int, out chan<- int) {
        defer close(out)
        for num := range in {
            out <- num * num
        }
    }
    
    // Stage 3: фильтрация
    func filter(in <-chan int, out chan<- int) {
        defer close(out)
        for num := range in {
            if num%2 == 0 {
                out <- num
            }
        }
    }
    
    // Создаем pipeline
    stage1 := make(chan int)
    stage2 := make(chan int)
    stage3 := make(chan int)
    
    go generate(stage1)
    go process(stage1, stage2)
    go filter(stage2, stage3)
    
    // Читаем результат
    for result := range stage3 {
        fmt.Printf("Result: %d\n", result)
    }
}

// Fan-out/Fan-in паттерны
func fanPatterns() {
    // Fan-out: один источник, много получателей
    func fanOut(in <-chan int, out1, out2, out3 chan<- int) {
        defer close(out1)
        defer close(out2) 
        defer close(out3)
        
        for value := range in {
            // Отправляем всем получателям
            out1 <- value
            out2 <- value
            out3 <- value
        }
    }
    
    // Fan-in: много источников, один получатель
    func fanIn(in1, in2, in3 <-chan int, out chan<- int) {
        var wg sync.WaitGroup
        
        sendFromChannel := func(ch <-chan int) {
            defer wg.Done()
            for value := range ch {
                out <- value
            }
        }
        
        wg.Add(3)
        go sendFromChannel(in1)
        go sendFromChannel(in2)
        go sendFromChannel(in3)
        
        go func() {
            wg.Wait()
            close(out)
        }()
    }
}

// Преобразование направлений
func channelConversion() {
    // Двунаправленный канал можно преобразовать в однонаправленный
    bidir := make(chan string)
    
    var sendOnly chan<- string = bidir  // OK
    var recvOnly <-chan string = bidir  // OK
    
    // Обратное преобразование невозможно
    // var bidir2 chan string = sendOnly // Error
    
    // Но можно использовать type assertion (небезопасно)
    // unsafeBidir := (chan string)(sendOnly) // небезопасно!
}

// Практический пример: Worker pool
func workerPool() {
    type Job struct {
        ID   int
        Data string
    }
    
    type Result struct {
        JobID int
        Value string
    }
    
    // Worker функция
    func worker(id int, jobs <-chan Job, results chan<- Result) {
        for job := range jobs {
            // Обработка задачи
            result := Result{
                JobID: job.ID,
                Value: fmt.Sprintf("Processed by worker %d: %s", id, job.Data),
            }
            results <- result
        }
    }
    
    // Создаем каналы
    jobs := make(chan Job, 100)
    results := make(chan Result, 100)
    
    // Запускаем workers
    const numWorkers = 3
    for w := 1; w <= numWorkers; w++ {
        go worker(w, jobs, results) // jobs только для чтения, results только для записи
    }
    
    // Отправляем задачи
    go func() {
        defer close(jobs)
        for i := 1; i <= 10; i++ {
            jobs <- Job{ID: i, Data: fmt.Sprintf("job_%d", i)}
        }
    }()
    
    // Собираем результаты
    for i := 0; i < 10; i++ {
        result := <-results
        fmt.Printf("Result: %+v\n", result)
    }
}
```

### 44. Как реализовать worker pool?

**Развернутый ответ:**
Worker pool — это паттерн для ограничения количества одновременно выполняющихся задач:

```go
// Базовый worker pool
func basicWorkerPool() {
    type Job struct {
        ID       int
        Payload  string
        Duration time.Duration
    }
    
    type Result struct {
        JobID     int
        Output    string
        Worker    int
        StartTime time.Time
        EndTime   time.Time
    }
    
    // Worker функция
    worker := func(workerID int, jobs <-chan Job, results chan<- Result) {
        for job := range jobs {
            startTime := time.Now()
            
            // Имитация работы
            time.Sleep(job.Duration)
            
            result := Result{
                JobID:     job.ID,
                Output:    fmt.Sprintf("Processed '%s'", job.Payload),
                Worker:    workerID,
                StartTime: startTime,
                EndTime:   time.Now(),
            }
            
            results <- result
        }
    }
    
    // Настройка pool
    const numWorkers = 3
    const numJobs = 10
    
    jobs := make(chan Job, numJobs)
    results := make(chan Result, numJobs)
    
    // Запуск workers
    for w := 1; w <= numWorkers; w++ {
        go worker(w, jobs, results)
    }
    
    // Отправка задач
    for j := 1; j <= numJobs; j++ {
        job := Job{
            ID:       j,
            Payload:  fmt.Sprintf("task_%d", j),
            Duration: time.Duration(rand.Intn(1000)) * time.Millisecond,
        }
        jobs <- job
    }
    close(jobs)
    
    // Сбор результатов
    for r := 1; r <= numJobs; r++ {
        result := <-results
        fmt.Printf("Job %d completed by worker %d in %v\n",
            result.JobID, result.Worker, result.EndTime.Sub(result.StartTime))
    }
}

// Продвинутый worker pool с graceful shutdown
func advancedWorkerPool() {
    type WorkerPool struct {
        workerCount int
        jobQueue    chan Job
        resultQueue chan Result
        quit        chan bool
        wg          sync.WaitGroup
    }
    
    func NewWorkerPool(workerCount, jobQueueSize int) *WorkerPool {
        return &WorkerPool{
            workerCount: workerCount,
            jobQueue:    make(chan Job, jobQueueSize),
            resultQueue: make(chan Result, jobQueueSize),
            quit:        make(chan bool),
        }
    }
    
    func (wp *WorkerPool) Start() {
        // Запускаем workers
        for i := 0; i < wp.workerCount; i++ {
            wp.wg.Add(1)
            go wp.worker(i + 1)
        }
    }
    
    func (wp *WorkerPool) worker(id int) {
        defer wp.wg.Done()
        
        for {
            select {
            case job := <-wp.jobQueue:
                // Обработка задачи
                startTime := time.Now()
                time.Sleep(job.Duration) // имитация работы
                
                result := Result{
                    JobID:     job.ID,
                    Output:    fmt.Sprintf("Worker %d processed: %s", id, job.Payload),
                    Worker:    id,
                    StartTime: startTime,
                    EndTime:   time.Now(),
                }
                
                select {
                case wp.resultQueue <- result:
                case <-wp.quit:
                    return
                }
                
            case <-wp.quit:
                return
            }
        }
    }
    
    func (wp *WorkerPool) Submit(job Job) bool {
        select {
        case wp.jobQueue <- job:
            return true
        case <-wp.quit:
            return false
        default:
            return false // queue full
        }
    }
    
    func (wp *WorkerPool) Results() <-chan Result {
        return wp.resultQueue
    }
    
    func (wp *WorkerPool) Stop() {
        close(wp.quit)
        wp.wg.Wait()
        close(wp.resultQueue)
    }
    
    // Использование
    pool := NewWorkerPool(3, 10)
    pool.Start()
    
    // Отправляем задачи
    go func() {
        for i := 1; i <= 5; i++ {
            job := Job{
                ID:       i,
                Payload:  fmt.Sprintf("task_%d", i),
                Duration: 500 * time.Millisecond,
            }
            if !pool.Submit(job) {
                fmt.Printf("Failed to submit job %d\n", i)
            }
        }
    }()
    
    // Читаем результаты
    go func() {
        for result := range pool.Results() {
            fmt.Printf("Completed: %s\n", result.Output)
        }
    }()
    
    time.Sleep(3 * time.Second)
    pool.Stop()
}

// Worker pool с контекстом
func contextWorkerPool() {
    type ContextJob struct {
        ctx context.Context
        Job
    }
    
    worker := func(ctx context.Context, id int, jobs <-chan ContextJob, results chan<- Result) {
        for {
            select {
            case job := <-jobs:
                // Проверяем контекст задачи
                select {
                case <-job.ctx.Done():
                    fmt.Printf("Worker %d: job %d cancelled\n", id, job.ID)
                    continue
                default:
                }
                
                // Выполняем с возможностью отмены
                done := make(chan Result, 1)
                go func() {
                    time.Sleep(job.Duration)
                    done <- Result{
                        JobID:  job.ID,
                        Output: fmt.Sprintf("Worker %d completed job %d", id, job.ID),
                        Worker: id,
                    }
                }()
                
                select {
                case result := <-done:
                    results <- result
                case <-job.ctx.Done():
                    fmt.Printf("Worker %d: job %d cancelled during execution\n", id, job.ID)
                case <-ctx.Done():
                    return
                }
                
            case <-ctx.Done():
                return
            }
        }
    }
    
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    
    jobs := make(chan ContextJob, 10)
    results := make(chan Result, 10)
    
    // Запускаем workers
    for i := 1; i <= 2; i++ {
        go worker(ctx, i, jobs, results)
    }
    
    // Отправляем задачи с разными контекстами
    for i := 1; i <= 3; i++ {
        jobCtx := ctx
        if i == 2 {
            // Задача с таймаутом
            var jobCancel context.CancelFunc
            jobCtx, jobCancel = context.WithTimeout(ctx, 100*time.Millisecond)
            defer jobCancel()
        }
        
        jobs <- ContextJob{
            ctx: jobCtx,
            Job: Job{
                ID:       i,
                Payload:  fmt.Sprintf("task_%d", i),
                Duration: 200 * time.Millisecond,
            },
        }
    }
    
    // Собираем результаты
    time.Sleep(time.Second)
}

// Динамический worker pool
func dynamicWorkerPool() {
    type DynamicPool struct {
        minWorkers    int
        maxWorkers    int
        currentWorkers int
        jobQueue      chan Job
        resultQueue   chan Result
        workerQueue   chan chan Job
        quit          chan bool
        mu            sync.RWMutex
    }
    
    func (dp *DynamicPool) addWorker() {
        dp.mu.Lock()
        defer dp.mu.Unlock()
        
        if dp.currentWorkers < dp.maxWorkers {
            go dp.worker()
            dp.currentWorkers++
            fmt.Printf("Added worker, current: %d\n", dp.currentWorkers)
        }
    }
    
    func (dp *DynamicPool) worker() {
        jobChannel := make(chan Job)
        
        defer func() {
            dp.mu.Lock()
            dp.currentWorkers--
            dp.mu.Unlock()
        }()
        
        for {
            // Регистрируем worker как доступный
            dp.workerQueue <- jobChannel
            
            select {
            case job := <-jobChannel:
                // Выполняем работу
                time.Sleep(job.Duration)
                dp.resultQueue <- Result{
                    JobID:  job.ID,
                    Output: fmt.Sprintf("Processed job %d", job.ID),
                }
                
            case <-dp.quit:
                return
            }
        }
    }
    
    func (dp *DynamicPool) dispatcher() {
        for {
            select {
            case job := <-dp.jobQueue:
                // Ищем доступного worker
                select {
                case workerChannel := <-dp.workerQueue:
                    workerChannel <- job
                default:
                    // Нет доступных workers, добавляем нового
                    dp.addWorker()
                    workerChannel := <-dp.workerQueue
                    workerChannel <- job
                }
                
            case <-dp.quit:
                return
            }
        }
    }
}
```

### 45. Что такое deadlock и как его избежать?

**Развернутый ответ:**
Deadlock — взаимная блокировка горутин, ожидающих друг друга:

```go
// Классические примеры deadlock
func deadlockExamples() {
    // Пример 1: блокировка на небуферизованном канале
    func simpleDeadlock() {
        ch := make(chan int)
        ch <- 1 // блокируется навсегда, нет получателя
        // fatal error: all goroutines are asleep - deadlock!
    }
    
    // Пример 2: циклическое ожидание каналов
    func cyclicDeadlock() {
        ch1 := make(chan int)
        ch2 := make(chan int)
        
        go func() {
            ch1 <- 1  // горутина 1 отправляет в ch1
            <-ch2     // горутина 1 ждет из ch2
        }()
        
        go func() {
            ch2 <- 2  // горутина 2 отправляет в ch2
            <-ch1     // горутина 2 ждет из ch1
        }()
        
        time.Sleep(time.Second) // deadlock - горутины ждут друг друга
    }
    
    // Пример 3: deadlock с мьютексами
    func mutexDeadlock() {
        var mu1, mu2 sync.Mutex
        
        go func() {
            mu1.Lock()
            time.Sleep(100 * time.Millisecond)
            mu2.Lock() // ждет mu2
            mu2.Unlock()
            mu1.Unlock()
        }()
        
        go func() {
            mu2.Lock()
            time.Sleep(100 * time.Millisecond)
            mu1.Lock() // ждет mu1
            mu1.Unlock()
            mu2.Unlock()
        }()
        
        time.Sleep(time.Second) // deadlock
    }
}

// Методы избежания deadlock
func avoidDeadlock() {
    // Метод 1: использование буферизованных каналов
    func useBufferedChannels() {
        ch := make(chan int, 1) // буфер предотвращает блокировку
        ch <- 1                 // не блокируется
        value := <-ch           // получаем значение
        fmt.Println(value)
    }
    
    // Метод 2: правильный порядок блокировок
    func consistentLockOrder() {
        var mu1, mu2 sync.Mutex
        
        lockInOrder := func(first, second *sync.Mutex) {
            first.Lock()
            defer first.Unlock()
            second.Lock()
            defer second.Unlock()
            // критическая секция
        }
        
        go lockInOrder(&mu1, &mu2)
        go lockInOrder(&mu1, &mu2) // тот же порядок - нет deadlock
    }
    
    // Метод 3: timeout для операций
    func useTimeout() {
        ch := make(chan int)
        
        select {
        case ch <- 1:
            fmt.Println("Sent successfully")
        case <-time.After(100 * time.Millisecond):
            fmt.Println("Timeout - potential deadlock avoided")
        }
    }
    
    // Метод 4: неблокирующие операции с default
    func useDefault() {
        ch := make(chan int)
        
        select {
        case ch <- 1:
            fmt.Println("Sent successfully")
        default:
            fmt.Println("Channel not ready, doing something else")
        }
    }
}

// Обнаружение deadlock паттернов
func deadlockDetection() {
    // Паттерн 1: проверка готовности канала
    func isChannelReady(ch chan int) bool {
        select {
        case ch <- 0:
            <-ch // убираем тестовое значение
            return true
        default:
            return false
        }
    }
    
    // Паттерн 2: мониторинг горутин
    func monitorGoroutines() {
        initial := runtime.NumGoroutine()
        
        // запускаем операцию
        done := make(chan bool)
        go func() {
            defer func() { done <- true }()
            // потенциально блокирующая операция
            time.Sleep(100 * time.Millisecond)
        }()
        
        select {
        case <-done:
            fmt.Println("Operation completed")
        case <-time.After(500 * time.Millisecond):
            current := runtime.NumGoroutine()
            if current > initial {
                fmt.Printf("Potential deadlock: %d goroutines stuck\n", current-initial)
            }
        }
    }
    
    // Паттерн 3: graceful shutdown с context
    func gracefulShutdown() {
        ctx, cancel := context.WithCancel(context.Background())
        defer cancel()
        
        // Операции с возможностью отмены
        done := make(chan bool)
        
        go func() {
            defer func() { done <- true }()
            
            // Долгая операция с проверкой context
            for i := 0; i < 100; i++ {
                select {
                case <-ctx.Done():
                    fmt.Println("Operation cancelled")
                    return
                default:
                    time.Sleep(10 * time.Millisecond)
                }
            }
        }()
        
        select {
        case <-done:
            fmt.Println("Operation completed normally")
        case <-time.After(200 * time.Millisecond):
            fmt.Println("Operation taking too long, cancelling...")
            cancel()
            <-done // ждем завершения после отмены
        }
    }
}

// Практические решения deadlock проблем
func practicalSolutions() {
    // Решение 1: Producer-Consumer с буферизацией
    func producerConsumer() {
        buffer := make(chan int, 10) // буфер предотвращает deadlock
        
        // Producer
        go func() {
            defer close(buffer)
            for i := 0; i < 20; i++ {
                buffer <- i
            }
        }()
        
        // Consumer
        for value := range buffer {
            fmt.Printf("Consumed: %d\n", value)
            time.Sleep(10 * time.Millisecond)
        }
    }
    
    // Решение 2: Coordinator горутина
    func coordinator() {
        requests := make(chan string)
        responses := make(chan string)
        
        // Coordinator предотвращает прямые deadlock между клиентами
        go func() {
            for request := range requests {
                // Обработка запроса
                response := fmt.Sprintf("Processed: %s", request)
                responses <- response
            }
            close(responses)
        }()
        
        // Клиенты
        go func() {
            requests <- "request1"
        }()
        
        go func() {
            requests <- "request2"
        }()
        
        close(requests)
        
        // Получаем ответы
        for response := range responses {
            fmt.Println(response)
        }
    }
    
    // Решение 3: Resource pool
    func resourcePool() {
        type Resource struct {
            id int
        }
        
        pool := make(chan *Resource, 3)
        
        // Инициализируем pool
        for i := 1; i <= 3; i++ {
            pool <- &Resource{id: i}
        }
        
        // Функция для безопасного использования ресурса
        useResource := func(name string) {
            select {
            case resource := <-pool:
                fmt.Printf("%s acquired resource %d\n", name, resource.id)
                
                // Используем ресурс
                time.Sleep(100 * time.Millisecond)
                
                // Возвращаем в pool
                pool <- resource
                fmt.Printf("%s released resource %d\n", name, resource.id)
                
            case <-time.After(200 * time.Millisecond):
                fmt.Printf("%s: timeout acquiring resource\n", name)
            }
        }
        
        // Множественные пользователи
        var wg sync.WaitGroup
        for i := 1; i <= 5; i++ {
            wg.Add(1)
            go func(id int) {
                defer wg.Done()
                useResource(fmt.Sprintf("User%d", id))
            }(i)
        }
        
        wg.Wait()
    }
}
```

### 46. Как работает паттерн fan-in/fan-out?

**Развернутый ответ:**
Fan-out распределяет работу между несколькими горутинами, fan-in собирает результаты:

```go
// Fan-out: один источник -> много получателей
func fanOut() {
    input := make(chan int)
    output1 := make(chan int)
    output2 := make(chan int)
    output3 := make(chan int)
    
    // Распределитель (fan-out)
    go func() {
        defer close(output1)
        defer close(output2)
        defer close(output3)
        
        for value := range input {
            // Отправляем всем workers
            go func(v int) { output1 <- v }(value)
            go func(v int) { output2 <- v }(value)
            go func(v int) { output3 <- v }(value)
        }
    }()
    
    // Workers
    worker := func(id int, in <-chan int) {
        for value := range in {
            fmt.Printf("Worker %d processing %d\n", id, value)
            time.Sleep(100 * time.Millisecond)
        }
    }
    
    go worker(1, output1)
    go worker(2, output2)
    go worker(3, output3)
    
    // Отправляем данные
    go func() {
        defer close(input)
        for i := 1; i <= 5; i++ {
            input <- i
        }
    }()
    
    time.Sleep(2 * time.Second)
}

// Fan-in: много источников -> один получатель
func fanIn() {
    // Функция fan-in для объединения каналов
    merge := func(channels ...<-chan int) <-chan int {
        out := make(chan int)
        var wg sync.WaitGroup
        
        // Для каждого входного канала
        for _, ch := range channels {
            wg.Add(1)
            go func(c <-chan int) {
                defer wg.Done()
                for value := range c {
                    out <- value
                }
            }(ch)
        }
        
        // Закрываем выходной канал когда все источники завершены
        go func() {
            wg.Wait()
            close(out)
        }()
        
        return out
    }
    
    // Создаем источники данных
    source1 := make(chan int)
    source2 := make(chan int)
    source3 := make(chan int)
    
    // Запускаем producers
    go func() {
        defer close(source1)
        for i := 1; i <= 3; i++ {
            source1 <- i * 10
            time.Sleep(100 * time.Millisecond)
        }
    }()
    
    go func() {
        defer close(source2)
        for i := 1; i <= 3; i++ {
            source2 <- i * 100
            time.Sleep(150 * time.Millisecond)
        }
    }()
    
    go func() {
        defer close(source3)
        for i := 1; i <= 3; i++ {
            source3 <- i * 1000
            time.Sleep(200 * time.Millisecond)
        }
    }()
    
    // Объединяем все источники
    merged := merge(source1, source2, source3)
    
    // Читаем объединенные данные
    for value := range merged {
        fmt.Printf("Merged value: %d\n", value)
    }
}

// Комбинированный fan-out/fan-in pipeline
func fanOutInPipeline() {
    type Job struct {
        ID   int
        Data int
    }
    
    type Result struct {
        JobID  int
        Output int
    }
    
    // Stage 1: Generator
    generator := func() <-chan Job {
        out := make(chan Job)
        go func() {
            defer close(out)
            for i := 1; i <= 10; i++ {
                out <- Job{ID: i, Data: i * 10}
            }
        }()
        return out
    }
    
    // Stage 2: Fan-out to workers
    fanOutToWorkers := func(input <-chan Job, numWorkers int) []<-chan Result {
        outputs := make([]<-chan Result, numWorkers)
        
        for i := 0; i < numWorkers; i++ {
            out := make(chan Result)
            outputs[i] = out
            
            go func(workerID int, jobs <-chan Job, results chan<- Result) {
                defer close(results)
                for job := range jobs {
                    // Имитация работы
                    time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
                    
                    result := Result{
                        JobID:  job.ID,
                        Output: job.Data * job.Data,
                    }
                    results <- result
                }
            }(i, input, out)
        }
        
        return outputs
    }
    
    // Stage 3: Fan-in results
    fanInResults := func(inputs []<-chan Result) <-chan Result {
        out := make(chan Result)
        var wg sync.WaitGroup
        
        for _, input := range inputs {
            wg.Add(1)
            go func(in <-chan Result) {
                defer wg.Done()
                for result := range in {
                    out <- result
                }
            }(input)
        }
        
        go func() {
            wg.Wait()
            close(out)
        }()
        
        return out
    }
    
    // Создаем pipeline
    jobs := generator()
    workerOutputs := fanOutToWorkers(jobs, 3)
    results := fanInResults(workerOutputs)
    
    // Собираем результаты
    for result := range results {
        fmt.Printf("Job %d result: %d\n", result.JobID, result.Output)
    }
}

// Практический пример: параллельная обработка веб-запросов
func parallelWebRequests() {
    urls := []string{
        "https://httpbin.org/delay/1",
        "https://httpbin.org/delay/2", 
        "https://httpbin.org/delay/1",
    }
    
    type Request struct {
        URL string
        ID  int
    }
    
    type Response struct {
        ID     int
        URL    string
        Status int
        Error  error
    }
    
    // Fan-out requests
    requestChan := make(chan Request)
    responseChan := make(chan Response)
    
    // Worker pool
    const numWorkers = 3
    for i := 0; i < numWorkers; i++ {
        go func(workerID int) {
            client := &http.Client{Timeout: 5 * time.Second}
            
            for req := range requestChan {
                resp, err := client.Get(req.URL)
                
                response := Response{
                    ID:  req.ID,
                    URL: req.URL,
                }
                
                if err != nil {
                    response.Error = err
                } else {
                    response.Status = resp.StatusCode
                    resp.Body.Close()
                }
                
                responseChan <- response
            }
        }(i)
    }
    
    // Отправляем запросы
    go func() {
        defer close(requestChan)
        for i, url := range urls {
            requestChan <- Request{URL: url, ID: i}
        }
    }()
    
    // Собираем ответы
    for i := 0; i < len(urls); i++ {
        response := <-responseChan
        if response.Error != nil {
            fmt.Printf("Request %d failed: %v\n", response.ID, response.Error)
        } else {
            fmt.Printf("Request %d completed: %d\n", response.ID, response.Status)
        }
    }
}
```

### 47. Что такое context и зачем он нужен?

**Развернутый ответ:**
Context передает deadline, сигналы отмены и значения между границами API и горутинами:

```go
// Основные функции context
func contextBasics() {
    // 1. Background context - корневой контекст
    ctx := context.Background()
    
    // 2. TODO context - когда неясно какой контекст использовать
    ctx = context.TODO()
    
    // 3. WithCancel - для отмены операций
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel() // всегда вызывайте cancel
    
    // 4. WithTimeout - автоматическая отмена по времени
    ctx, cancel = context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    // 5. WithDeadline - отмена в определенное время
    deadline := time.Now().Add(10 * time.Second)
    ctx, cancel = context.WithDeadline(context.Background(), deadline)
    defer cancel()
    
    // 6. WithValue - передача значений
    ctx = context.WithValue(ctx, "userID", 12345)
}

// Отмена операций
func cancellationExample() {
    ctx, cancel := context.WithCancel(context.Background())
    
    go func() {
        // Имитируем долгую операцию
        for i := 0; i < 10; i++ {
            select {
            case <-ctx.Done():
                fmt.Printf("Operation cancelled: %v\n", ctx.Err())
                return
            default:
                fmt.Printf("Working... step %d\n", i)
                time.Sleep(500 * time.Millisecond)
            }
        }
        fmt.Println("Operation completed")
    }()
    
    // Отменяем через 2 секунды
    time.Sleep(2 * time.Second)
    cancel()
    
    time.Sleep(1 * time.Second) // ждем завершения
}

// Timeout операций
func timeoutExample() {
    // Функция с поддержкой context
    doWork := func(ctx context.Context) error {
        // Имитация работы с проверкой контекста
        for i := 0; i < 5; i++ {
            select {
            case <-ctx.Done():
                return ctx.Err() // context.DeadlineExceeded или context.Canceled
            default:
                time.Sleep(time.Second)
                fmt.Printf("Work step %d completed\n", i+1)
            }
        }
        return nil
    }
    
    // Операция с таймаутом
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()
    
    if err := doWork(ctx); err != nil {
        if errors.Is(err, context.DeadlineExceeded) {
            fmt.Println("Operation timed out")
        } else if errors.Is(err, context.Canceled) {
            fmt.Println("Operation was cancelled")
        }
    } else {
        fmt.Println("Operation completed successfully")
    }
}

// Передача значений через context
func contextValues() {
    type key string
    
    const (
        userIDKey    key = "userID"
        requestIDKey key = "requestID"
    )
    
    // Создаем контекст со значениями
    ctx := context.WithValue(context.Background(), userIDKey, 12345)
    ctx = context.WithValue(ctx, requestIDKey, "req-abc-123")
    
    // Функция, использующая значения из контекста
    processRequest := func(ctx context.Context) {
        userID := ctx.Value(userIDKey)
        requestID := ctx.Value(requestIDKey)
        
        if userID != nil {
            fmt.Printf("Processing request for user: %v\n", userID)
        }
        
        if requestID != nil {
            fmt.Printf("Request ID: %v\n", requestID)
        }
        
        // Безопасное извлечение с type assertion
        if uid, ok := userID.(int); ok {
            fmt.Printf("User ID as int: %d\n", uid)
        }
    }
    
    processRequest(ctx)
}

// Context в HTTP серверах
func httpContextExample() {
    handler := func(w http.ResponseWriter, r *http.Request) {
        // Контекст автоматически создается для каждого запроса
        ctx := r.Context()
        
        // Добавляем значения в контекст
        ctx = context.WithValue(ctx, "startTime", time.Now())
        
        // Создаем контекст с таймаутом для внешних вызовов
        ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
        defer cancel()
        
        // Имитация внешнего API вызова
        if err := callExternalAPI(ctx); err != nil {
            if errors.Is(err, context.DeadlineExceeded) {
                http.Error(w, "External API timeout", http.StatusRequestTimeout)
                return
            }
            http.Error(w, "External API error", http.StatusInternalServerError)
            return
        }
        
        w.WriteHeader(http.StatusOK)
        w.Write([]byte("Success"))
    }
    
    callExternalAPI := func(ctx context.Context) error {
        // Имитация внешнего вызова с поддержкой отмены
        done := make(chan error, 1)
        
        go func() {
            // Имитация долгого запроса
            time.Sleep(3 * time.Second)
            done <- nil
        }()
        
        select {
        case err := <-done:
            return err
        case <-ctx.Done():
            return ctx.Err()
        }
    }
}

// Context propagation в микросервисах
func microserviceContext() {
    type Service struct {
        name string
    }
    
    // Каждый сервис передает контекст дальше
    func (s *Service) ProcessRequest(ctx context.Context, data string) error {
        // Добавляем трассировку
        ctx = context.WithValue(ctx, "service", s.name)
        
        // Проверяем, не отменен ли запрос
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
        }
        
        fmt.Printf("[%s] Processing: %s\n", s.name, data)
        
        // Имитация работы
        time.Sleep(100 * time.Millisecond)
        
        // Вызов следующего сервиса
        if s.name == "service1" {
            nextService := &Service{name: "service2"}
            return nextService.ProcessRequest(ctx, data)
        }
        
        return nil
    }
    
    // Создаем запрос с таймаутом
    ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
    defer cancel()
    
    service := &Service{name: "service1"}
    if err := service.ProcessRequest(ctx, "important data"); err != nil {
        fmt.Printf("Request failed: %v\n", err)
    }
}
```

### 48. Как создать context с timeout?

**Развернутый ответ:**
Context с timeout автоматически отменяется через заданное время:

```go
func timeoutContexts() {
    // Основные способы создания timeout context
    
    // 1. WithTimeout - относительное время
    ctx1, cancel1 := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel1()
    
    // 2. WithDeadline - абсолютное время
    deadline := time.Now().Add(10 * time.Second)
    ctx2, cancel2 := context.WithDeadline(context.Background(), deadline)
    defer cancel2()
    
    // Использование timeout context
    doWorkWithTimeout := func(ctx context.Context, workDuration time.Duration) error {
        done := make(chan struct{})
        
        go func() {
            defer close(done)
            time.Sleep(workDuration) // имитация работы
        }()
        
        select {
        case <-done:
            return nil // работа завершена
        case <-ctx.Done():
            return ctx.Err() // таймаут или отмена
        }
    }
    
    // Тест с успешным завершением
    fmt.Println("Testing successful completion...")
    err := doWorkWithTimeout(ctx1, 1*time.Second)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Println("Work completed successfully")
    }
    
    // Тест с таймаутом
    fmt.Println("Testing timeout...")
    ctx3, cancel3 := context.WithTimeout(context.Background(), 1*time.Second)
    defer cancel3()
    
    err = doWorkWithTimeout(ctx3, 3*time.Second)
    if err != nil {
        if errors.Is(err, context.DeadlineExceeded) {
            fmt.Println("Work timed out as expected")
        } else {
            fmt.Printf("Unexpected error: %v\n", err)
        }
    }
}

// Каскадные timeout'ы
func cascadingTimeouts() {
    // Родительский контекст с большим таймаутом
    parentCtx, parentCancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer parentCancel()
    
    // Дочерний контекст с меньшим таймаутом
    childCtx, childCancel := context.WithTimeout(parentCtx, 2*time.Second)
    defer childCancel()
    
    // Операция будет отменена через 2 секунды (меньший таймаут)
    startTime := time.Now()
    
    select {
    case <-childCtx.Done():
        elapsed := time.Since(startTime)
        fmt.Printf("Child context cancelled after %v: %v\n", elapsed, childCtx.Err())
    case <-parentCtx.Done():
        elapsed := time.Since(startTime)
        fmt.Printf("Parent context cancelled after %v: %v\n", elapsed, parentCtx.Err())
    }
}

// Практические паттерны с timeout
func practicalTimeoutPatterns() {
    // Паттерн 1: HTTP клиент с timeout
    func makeHTTPRequest(url string, timeout time.Duration) error {
        ctx, cancel := context.WithTimeout(context.Background(), timeout)
        defer cancel()
        
        req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
        if err != nil {
            return err
        }
        
        client := &http.Client{}
        resp, err := client.Do(req)
        if err != nil {
            return err
        }
        defer resp.Body.Close()
        
        fmt.Printf("Response status: %s\n", resp.Status)
        return nil
    }
    
    // Паттерн 2: База данных с timeout
    func queryDatabase(query string, timeout time.Duration) error {
        ctx, cancel := context.WithTimeout(context.Background(), timeout)
        defer cancel()
        
        // Имитация DB запроса
        done := make(chan error, 1)
        
        go func() {
            // Симуляция DB операции
            time.Sleep(1 * time.Second)
            done <- nil
        }()
        
        select {
        case err := <-done:
            if err == nil {
                fmt.Println("Database query completed")
            }
            return err
        case <-ctx.Done():
            fmt.Println("Database query timed out")
            return ctx.Err()
        }
    }
    
    // Паттерн 3: Retry с timeout
    func retryWithTimeout(operation func() error, maxRetries int, timeout time.Duration) error {
        ctx, cancel := context.WithTimeout(context.Background(), timeout)
        defer cancel()
        
        for attempt := 1; attempt <= maxRetries; attempt++ {
            select {
            case <-ctx.Done():
                return fmt.Errorf("retry timeout after %d attempts: %w", attempt-1, ctx.Err())
            default:
            }
            
            err := operation()
            if err == nil {
                fmt.Printf("Operation succeeded on attempt %d\n", attempt)
                return nil
            }
            
            fmt.Printf("Attempt %d failed: %v\n", attempt, err)
            
            if attempt < maxRetries {
                // Exponential backoff
                backoff := time.Duration(attempt) * 100 * time.Millisecond
                
                select {
                case <-time.After(backoff):
                case <-ctx.Done():
                    return ctx.Err()
                }
            }
        }
        
        return fmt.Errorf("operation failed after %d attempts", maxRetries)
    }
    
    // Тестирование паттернов
    err := queryDatabase("SELECT * FROM users", 2*time.Second)
    if err != nil {
        fmt.Printf("DB error: %v\n", err)
    }
    
    // Retry пример
    failingOperation := func() error {
        if rand.Float64() < 0.7 { // 70% вероятность неудачи
            return errors.New("random failure")
        }
        return nil
    }
    
    err = retryWithTimeout(failingOperation, 5, 3*time.Second)
    if err != nil {
        fmt.Printf("Retry failed: %v\n", err)
    }
}

// Advanced timeout управление
func advancedTimeoutManagement() {
    // Динамический timeout на основе нагрузки
    func dynamicTimeout(baseTimeout time.Duration, load float64) time.Duration {
        // Увеличиваем timeout при высокой нагрузке
        multiplier := 1.0 + load
        return time.Duration(float64(baseTimeout) * multiplier)
    }
    
    // Timeout с прогресс индикатором
    func operationWithProgress(ctx context.Context) error {
        const totalSteps = 10
        progressChan := make(chan int, totalSteps)
        
        go func() {
            defer close(progressChan)
            for i := 1; i <= totalSteps; i++ {
                select {
                case <-ctx.Done():
                    return
                default:
                    time.Sleep(200 * time.Millisecond)
                    progressChan <- i
                }
            }
        }()
        
        for {
            select {
            case progress, ok := <-progressChan:
                if !ok {
                    fmt.Println("Operation completed 100%")
                    return nil
                }
                fmt.Printf("Progress: %d%%\n", progress*10)
                
            case <-ctx.Done():
                fmt.Printf("Operation cancelled at some progress level\n")
                return ctx.Err()
            }
        }
    }
    
    // Тестирование
    currentLoad := 0.5 // 50% нагрузка
    timeout := dynamicTimeout(2*time.Second, currentLoad)
    
    ctx, cancel := context.WithTimeout(context.Background(), timeout)
    defer cancel()
    
    err := operationWithProgress(ctx)
    if err != nil {
        fmt.Printf("Operation error: %v\n", err)
    }
}
```

### 49. Как правильно передавать данные через context?

**Развернутый ответ:**
Context values следует использовать для request-scoped данных, а не для передачи параметров функций:

```go
// Правильное использование context values
func contextValuesGoodPractices() {
    // Определяем типизированные ключи
    type contextKey string
    
    const (
        userIDKey     contextKey = "userID"
        requestIDKey  contextKey = "requestID"
        correlationKey contextKey = "correlationID"
        traceIDKey    contextKey = "traceID"
    )
    
    // Создание контекста с типизированными значениями
    createRequestContext := func(userID int, requestID string) context.Context {
        ctx := context.Background()
        ctx = context.WithValue(ctx, userIDKey, userID)
        ctx = context.WithValue(ctx, requestIDKey, requestID)
        ctx = context.WithValue(ctx, correlationKey, generateCorrelationID())
        return ctx
    }
    
    // Безопасное извлечение значений
    getUserID := func(ctx context.Context) (int, bool) {
        userID, ok := ctx.Value(userIDKey).(int)
        return userID, ok
    }
    
    getRequestID := func(ctx context.Context) (string, bool) {
        requestID, ok := ctx.Value(requestIDKey).(string)
        return requestID, ok
    }
    
    // Использование
    ctx := createRequestContext(12345, "req-abc-123")
    
    if userID, ok := getUserID(ctx); ok {
        fmt.Printf("User ID: %d\n", userID)
    }
    
    if requestID, ok := getRequestID(ctx); ok {
        fmt.Printf("Request ID: %s\n", requestID)
    }
}

// Что НЕ следует передавать через context
func contextAntiPatterns() {
    // ПЛОХО: передача обычных параметров функции
    func badExample(ctx context.Context) {
        // Не делайте так!
        ctx = context.WithValue(ctx, "database", db)
        ctx = context.WithValue(ctx, "config", config)
        ctx = context.WithValue(ctx, "logger", logger)
        
        processData(ctx) // передаем зависимости через context
    }
    
    // ХОРОШО: передача через параметры
    func goodExample(ctx context.Context, db Database, config Config, logger Logger) {
        processDataCorrectly(ctx, db, config, logger)
    }
    
    // ПЛОХО: изменяемые данные в context
    func mutableDataBad(ctx context.Context) {
        data := make(map[string]interface{})
        ctx = context.WithValue(ctx, "mutableData", data)
        
        // Проблема: несколько горутин могут изменять data
        go func() {
            data["key1"] = "value1" // race condition!
        }()
        
        go func() {
            data["key2"] = "value2" // race condition!
        }()
    }
    
    // ХОРОШО: неизменяемые данные
    func immutableDataGood(ctx context.Context) {
        readOnlyData := map[string]string{
            "version": "1.0",
            "region":  "us-east-1",
        }
        ctx = context.WithValue(ctx, "metadata", readOnlyData)
        // Безопасно: данные только для чтения
    }
}

// Паттерны для request-scoped данных
func requestScopedPatterns() {
    // Middleware pattern для добавления значений
    func authMiddleware(next http.HandlerFunc) http.HandlerFunc {
        return func(w http.ResponseWriter, r *http.Request) {
            // Извлекаем токен из заголовка
            token := r.Header.Get("Authorization")
            
            // Валидируем и получаем user ID
            userID, err := validateToken(token)
            if err != nil {
                http.Error(w, "Unauthorized", http.StatusUnauthorized)
                return
            }
            
            // Добавляем user ID в контекст
            ctx := context.WithValue(r.Context(), "userID", userID)
            r = r.WithContext(ctx)
            
            next(w, r)
        }
    }
    
    func loggingMiddleware(next http.HandlerFunc) http.HandlerFunc {
        return func(w http.ResponseWriter, r *http.Request) {
            // Генерируем request ID
            requestID := generateRequestID()
            
            // Добавляем в контекст
            ctx := context.WithValue(r.Context(), "requestID", requestID)
            r = r.WithContext(ctx)
            
            // Логируем начало запроса
            fmt.Printf("[%s] %s %s\n", requestID, r.Method, r.URL.Path)
            
            next(w, r)
        }
    }
    
    // Handler использует данные из контекста
    func profileHandler(w http.ResponseWriter, r *http.Request) {
        ctx := r.Context()
        
        // Извлекаем user ID
        userID, ok := ctx.Value("userID").(int)
        if !ok {
            http.Error(w, "User ID not found", http.StatusInternalServerError)
            return
        }
        
        // Извлекаем request ID для логирования
        requestID, _ := ctx.Value("requestID").(string)
        
        fmt.Printf("[%s] Getting profile for user %d\n", requestID, userID)
        
        // Логика получения профиля...
        profile := getUserProfile(ctx, userID)
        json.NewEncoder(w).Encode(profile)
    }
}

// Трассировка и мониторинг через context
func tracingThroughContext() {
    type TraceInfo struct {
        TraceID string
        SpanID  string
        Baggage map[string]string
    }
    
    // Middleware для трассировки
    func tracingMiddleware(next http.HandlerFunc) http.HandlerFunc {
        return func(w http.ResponseWriter, r *http.Request) {
            // Создаем или извлекаем trace информацию
            traceID := r.Header.Get("X-Trace-ID")
            if traceID == "" {
                traceID = generateTraceID()
            }
            
            spanID := generateSpanID()
            
            trace := TraceInfo{
                TraceID: traceID,
                SpanID:  spanID,
                Baggage: make(map[string]string),
            }
            
            // Добавляем в контекст
            ctx := context.WithValue(r.Context(), "trace", trace)
            r = r.WithContext(ctx)
            
            // Добавляем заголовки ответа
            w.Header().Set("X-Trace-ID", traceID)
            w.Header().Set("X-Span-ID", spanID)
            
            next(w, r)
        }
    }
    
    // Функция для создания child span
    func createChildSpan(ctx context.Context, operationName string) context.Context {
        if trace, ok := ctx.Value("trace").(TraceInfo); ok {
            childTrace := TraceInfo{
                TraceID: trace.TraceID,
                SpanID:  generateSpanID(),
                Baggage: trace.Baggage, // наследуем baggage
            }
            
            fmt.Printf("Starting span: %s (trace: %s, span: %s)\n", 
                operationName, childTrace.TraceID, childTrace.SpanID)
            
            return context.WithValue(ctx, "trace", childTrace)
        }
        
        return ctx
    }
    
    // Использование в сервисных функциях
    func databaseQuery(ctx context.Context, query string) error {
        ctx = createChildSpan(ctx, "database.query")
        
        if trace, ok := ctx.Value("trace").(TraceInfo); ok {
            fmt.Printf("[%s:%s] Executing query: %s\n", 
                trace.TraceID, trace.SpanID, query)
        }
        
        // Имитация DB запроса
        time.Sleep(100 * time.Millisecond)
        return nil
    }
    
    func externalAPICall(ctx context.Context, endpoint string) error {
        ctx = createChildSpan(ctx, "http.client")
        
        // Передаем trace ID в исходящих запросах
        if trace, ok := ctx.Value("trace").(TraceInfo); ok {
            fmt.Printf("[%s:%s] Calling external API: %s\n", 
                trace.TraceID, trace.SpanID, endpoint)
            
            // В реальном коде добавили бы заголовок X-Trace-ID
        }
        
        time.Sleep(200 * time.Millisecond)
        return nil
    }
}

// Вспомогательные функции
func generateCorrelationID() string {
    return fmt.Sprintf("corr-%d", time.Now().UnixNano())
}

func generateRequestID() string {
    return fmt.Sprintf("req-%d", time.Now().UnixNano())
}

func generateTraceID() string {
    return fmt.Sprintf("trace-%d", time.Now().UnixNano())
}

func generateSpanID() string {
    return fmt.Sprintf("span-%d", time.Now().UnixNano())
}

func validateToken(token string) (int, error) {
    // Имитация валидации токена
    if token == "" {
        return 0, errors.New("empty token")
    }
    return 12345, nil // возвращаем mock user ID
}

func getUserProfile(ctx context.Context, userID int) map[string]interface{} {
    return map[string]interface{}{
        "id":   userID,
        "name": "John Doe",
        "email": "john@example.com",
    }
}
```

### 50. Что такое channel axioms (правила каналов)?

**Развернутый ответ:**
Channel axioms — фундаментальные правила поведения каналов в Go:

```go
// Основные правила каналов
func channelAxioms() {
    fmt.Println("=== Channel Axioms Demo ===")
    
    // Правило 1: Отправка в nil канал блокируется навсегда
    func nilChannelSend() {
        var ch chan int // nil канал
        
        go func() {
            fmt.Println("Attempting to send to nil channel...")
            ch <- 1 // заблокируется навсегда
            fmt.Println("This will never print")
        }()
        
        time.Sleep(100 * time.Millisecond)
        fmt.Println("Send to nil channel is blocked (as expected)")
    }
    
    // Правило 2: Чтение из nil канала блокируется навсегда
    func nilChannelReceive() {
        var ch chan int // nil канал
        
        go func() {
            fmt.Println("Attempting to receive from nil channel...")
            <-ch // заблокируется навсегда
            fmt.Println("This will never print")
        }()
        
        time.Sleep(100 * time.Millisecond)
        fmt.Println("Receive from nil channel is blocked (as expected)")
    }
    
    // Правило 3: Отправка в закрытый канал вызывает panic
    func closedChannelSend() {
        ch := make(chan int, 1)
        close(ch)
        
        defer func() {
            if r := recover(); r != nil {
                fmt.Printf("Panic caught: %v\n", r)
            }
        }()
        
        fmt.Println("Attempting to send to closed channel...")
        ch <- 1 // panic: send on closed channel
    }
    
    // Правило 4: Чтение из закрытого канала возвращает zero value
    func closedChannelReceive() {
        ch := make(chan int, 2)
        ch <- 1
        ch <- 2
        close(ch)
        
        fmt.Printf("Reading from closed channel: %d\n", <-ch) // 1
        fmt.Printf("Reading from closed channel: %d\n", <-ch) // 2
        fmt.Printf("Reading from closed channel: %d\n", <-ch) // 0 (zero value)
        
        // С проверкой статуса
        value, ok := <-ch
        fmt.Printf("Value: %d, OK: %t\n", value, ok) // 0, false
    }
    
    // Правило 5: Закрытие nil канала вызывает panic
    func closeNilChannel() {
        var ch chan int // nil канал
        
        defer func() {
            if r := recover(); r != nil {
                fmt.Printf("Panic caught: %v\n", r)
            }
        }()
        
        fmt.Println("Attempting to close nil channel...")
        close(ch) // panic: close of nil channel
    }
    
    // Правило 6: Повторное закрытие канала вызывает panic
    func doubleClose() {
        ch := make(chan int)
        close(ch)
        
        defer func() {
            if r := recover(); r != nil {
                fmt.Printf("Panic caught: %v\n", r)
            }
        }()
        
        fmt.Println("Attempting to close already closed channel...")
        close(ch) // panic: close of closed channel
    }
    
    // Демонстрация всех правил
    fmt.Println("\n1. Nil channel operations:")
    nilChannelSend()
    nilChannelReceive()
    
    fmt.Println("\n2. Closed channel operations:")
    closedChannelSend()
    closedChannelReceive()
    
    fmt.Println("\n3. Closing operations:")
    closeNilChannel()
    doubleClose()
}

// Практическое применение axioms
func practicalChannelAxioms() {
    // Использование nil каналов для отключения в select
    func selectWithNilChannels() {
        ch1 := make(chan string, 1)
        ch2 := make(chan string, 1)
        
        ch1 <- "message1"
        ch2 <- "message2"
        
        // Симулируем отключение каналов
        for i := 0; i < 3; i++ {
            select {
            case msg := <-ch1:
                fmt.Printf("Received from ch1: %s\n", msg)
                ch1 = nil // отключаем ch1 в select
                
            case msg := <-ch2:
                fmt.Printf("Received from ch2: %s\n", msg)
                ch2 = nil // отключаем ch2 в select
                
            default:
                fmt.Println("No more messages")
                return
            }
        }
    }
    
    // Безопасное закрытие каналов
    func safeChannelClose() {
        type SafeChannel struct {
            ch     chan int
            closed bool
            mu     sync.Mutex
        }
        
        func (sc *SafeChannel) Send(value int) bool {
            sc.mu.Lock()
            defer sc.mu.Unlock()
            
            if sc.closed {
                return false
            }
            
            select {
            case sc.ch <- value:
                return true
            default:
                return false // канал полон
            }
        }
        
        func (sc *SafeChannel) Close() {
            sc.mu.Lock()
            defer sc.mu.Unlock()
            
            if !sc.closed {
                close(sc.ch)
                sc.closed = true
            }
        }
        
        // Использование
        safeCh := &SafeChannel{
            ch: make(chan int, 2),
        }
        
        fmt.Println("Safe send:", safeCh.Send(1))  // true
        fmt.Println("Safe send:", safeCh.Send(2))  // true
        
        safeCh.Close()
        
        fmt.Println("Safe send after close:", safeCh.Send(3)) // false
        safeCh.Close() // безопасно - не вызовет panic
    }
    
    // Graceful shutdown паттерн
    func gracefulShutdown() {
        work := make(chan int, 10)
        done := make(chan struct{})
        quit := make(chan struct{})
        
        // Worker
        go func() {
            defer close(done)
            
            for {
                select {
                case job, ok := <-work:
                    if !ok {
                        // Канал закрыт, завершаем работу
                        fmt.Println("Work channel closed, worker stopping")
                        return
                    }
                    fmt.Printf("Processing job: %d\n", job)
                    time.Sleep(100 * time.Millisecond)
                    
                case <-quit:
                    // Сигнал немедленной остановки
                    fmt.Println("Quit signal received, worker stopping")
                    return
                }
            }
        }()
        
        // Отправляем работу
        for i := 1; i <= 3; i++ {
            work <- i
        }
        
        // Graceful shutdown - закрываем канал работы
        close(work)
        
        // Ждем завершения
        <-done
        fmt.Println("Worker finished gracefully")
    }
    
    selectWithNilChannels()
    safeChannelClose()
    gracefulShutdown()
}

// Channel направления и axioms
func channelDirectionsAxioms() {
    // Правила для направленных каналов
    
    // Send-only канал
    func sendOnlyExample(ch chan<- int) {
        ch <- 42
        // <-ch // Compilation error: cannot receive from send-only channel
        // close(ch) // OK: можно закрывать send-only канал
    }
    
    // Receive-only канал
    func receiveOnlyExample(ch <-chan int) {
        <-ch
        // ch <- 42 // Compilation error: cannot send to receive-only channel
        // close(ch) // Compilation error: cannot close receive-only channel
    }
    
    // Producer-Consumer с правильными направлениями
    func producerConsumer() {
        ch := make(chan int, 3)
        
        // Producer - может только отправлять
        producer := func(out chan<- int) {
            defer close(out) // Producer закрывает канал
            for i := 1; i <= 5; i++ {
                out <- i
            }
        }
        
        // Consumer - может только получать
        consumer := func(in <-chan int) {
            for value := range in { // range автоматически завершится при закрытии
                fmt.Printf("Consumed: %d\n", value)
            }
        }
        
        go producer(ch)
        consumer(ch)
    }
}
```

---

## 🔒 Раздел 4: Синхронизация (Ответы 56-70)

### 56. Что такое data race и как его обнаружить?

**Развернутый ответ:**
Data race происходит когда две или более горутин одновременно обращаются к одной переменной, и хотя бы одна из них записывает:

```go
// Пример data race
func dataRaceExample() {
    var counter int
    var wg sync.WaitGroup
    
    // Запускаем 1000 горутин, каждая увеличивает counter
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            counter++ // DATA RACE: одновременная запись без синхронизации
        }()
    }
    
    wg.Wait()
    fmt.Printf("Final counter (with race): %d\n", counter) // результат непредсказуем
}

// Обнаружение с помощью race detector
func raceDetection() {
    // Запуск с race detector: go run -race main.go
    // Или при сборке: go build -race
    
    var shared int
    
    go func() {
        shared = 1 // запись
    }()
    
    go func() {
        fmt.Println(shared) // чтение
    }()
    
    time.Sleep(100 * time.Millisecond)
    // Race detector выдаст предупреждение
}

// Исправленная версия без data race
func fixedVersion() {
    var counter int
    var mu sync.Mutex
    var wg sync.WaitGroup
    
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            mu.Lock()
            counter++ // защищено мьютексом
            mu.Unlock()
        }()
    }
    
    wg.Wait()
    fmt.Printf("Final counter (race-free): %d\n", counter) // всегда 1000
}

// Типичные места data race
func commonRaceConditions() {
    // 1. Неправильное использование map
    func mapRace() {
        m := make(map[int]int)
        
        // Несколько горутин записывают в мапу
        for i := 0; i < 10; i++ {
            go func(id int) {
                m[id] = id * id // RACE: concurrent map writes
            }(i)
        }
        
        time.Sleep(100 * time.Millisecond)
    }
    
    // 2. Захват переменной цикла
    func loopVariableRace() {
        var wg sync.WaitGroup
        
        for i := 0; i < 5; i++ {
            wg.Add(1)
            go func() {
                defer wg.Done()
                fmt.Println(i) // RACE: все горутины читают одну переменную i
            }()
        }
        
        wg.Wait()
    }
    
    // 3. Slice append race
    func sliceRace() {
        var slice []int
        var wg sync.WaitGroup
        
        for i := 0; i < 10; i++ {
            wg.Add(1)
            go func(val int) {
                defer wg.Done()
                slice = append(slice, val) // RACE: append не atomic
            }(i)
        }
        
        wg.Wait()
        fmt.Printf("Slice length: %d\n", len(slice)) // непредсказуемо
    }
}

// Инструменты для обнаружения races
func raceDetectionTools() {
    // 1. Встроенный race detector
    // go run -race program.go
    // go test -race
    // go build -race
    
    // 2. Статический анализ
    // go vet program.go
    
    // 3. Программная проверка
    func checkDataRace() {
        // Используем runtime для обнаружения
        defer func() {
            if r := recover(); r != nil {
                fmt.Printf("Panic detected (possible race): %v\n", r)
            }
        }()
        
        // код с потенциальным race
    }
}
```

### 57. Как работает `sync.Mutex`?

**Развернутый ответ:**
Mutex (mutual exclusion) обеспечивает эксклюзивный доступ к ресурсу:

```go
// Основы Mutex
func mutexBasics() {
    var mu sync.Mutex
    var counter int
    
    // Критическая секция
    increment := func() {
        mu.Lock()         // захват блокировки
        defer mu.Unlock() // освобождение (даже при panic)
        counter++
    }
    
    var wg sync.WaitGroup
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            increment()
        }()
    }
    
    wg.Wait()
    fmt.Printf("Counter: %d\n", counter) // всегда 1000
}

// Mutex в структурах
func mutexInStructs() {
    type SafeCounter struct {
        mu    sync.Mutex
        value int
    }
    
    func (c *SafeCounter) Increment() {
        c.mu.Lock()
        defer c.mu.Unlock()
        c.value++
    }
    
    func (c *SafeCounter) Value() int {
        c.mu.Lock()
        defer c.mu.Unlock()
        return c.value
    }
    
    // Использование
    counter := &SafeCounter{}
    
    var wg sync.WaitGroup
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            counter.Increment()
        }()
    }
    
    wg.Wait()
    fmt.Printf("Safe counter: %d\n", counter.Value())
}

// Проблемы с Mutex
func mutexProblems() {
    // 1. Deadlock
    func deadlockExample() {
        var mu1, mu2 sync.Mutex
        
        go func() {
            mu1.Lock()
            time.Sleep(100 * time.Millisecond)
            mu2.Lock() // может заблокироваться
            mu2.Unlock()
            mu1.Unlock()
        }()
        
        go func() {
            mu2.Lock()
            time.Sleep(100 * time.Millisecond)
            mu1.Lock() // может заблокироваться
            mu1.Unlock()
            mu2.Unlock()
        }()
        
        time.Sleep(500 * time.Millisecond)
    }
    
    // 2. Забыли разблокировать
    func forgotUnlock() {
        var mu sync.Mutex
        var data int
        
        badFunction := func() {
            mu.Lock()
            if data < 0 {
                return // ОШИБКА: не разблокировали!
            }
            data++
            mu.Unlock()
        }
        
        // Правильная версия с defer
        goodFunction := func() {
            mu.Lock()
            defer mu.Unlock() // всегда выполнится
            
            if data < 0 {
                return // безопасно
            }
            data++
        }
    }
    
    // 3. Копирование mutex
    func copyMutex() {
        type Counter struct {
            mu    sync.Mutex
            value int
        }
        
        c1 := Counter{}
        c2 := c1 // ОШИБКА: копируем mutex!
        
        // go vet обнаружит эту ошибку
        // Правильно: использовать указатели
        c3 := &Counter{}
        c4 := c3 // OK: копируем указатель
    }
}

// Продвинутые паттерны с Mutex
func advancedMutexPatterns() {
    // 1. Try-lock паттерн
    func tryLock() {
        var mu sync.Mutex
        
        // Go не имеет TryLock, но можно эмулировать
        type TryMutex struct {
            sync.Mutex
            ch chan struct{}
        }
        
        func NewTryMutex() *TryMutex {
            return &TryMutex{
                ch: make(chan struct{}, 1),
            }
        }
        
        func (tm *TryMutex) TryLock() bool {
            select {
            case tm.ch <- struct{}{}:
                return true
            default:
                return false
            }
        }
        
        func (tm *TryMutex) Unlock() {
            <-tm.ch
        }
        
        // Использование
        tryMu := NewTryMutex()
        if tryMu.TryLock() {
            defer tryMu.Unlock()
            // критическая секция
            fmt.Println("Lock acquired")
        } else {
            fmt.Println("Could not acquire lock")
        }
    }
    
    // 2. Timeout lock
    func timeoutLock() {
        type TimeoutMutex struct {
            ch chan struct{}
        }
        
        func NewTimeoutMutex() *TimeoutMutex {
            return &TimeoutMutex{
                ch: make(chan struct{}, 1),
            }
        }
        
        func (tm *TimeoutMutex) LockTimeout(timeout time.Duration) bool {
            select {
            case tm.ch <- struct{}{}:
                return true
            case <-time.After(timeout):
                return false
            }
        }
        
        func (tm *TimeoutMutex) Unlock() {
            <-tm.ch
        }
        
        // Использование
        timeoutMu := NewTimeoutMutex()
        if timeoutMu.LockTimeout(100 * time.Millisecond) {
            defer timeoutMu.Unlock()
            fmt.Println("Lock acquired within timeout")
        } else {
            fmt.Println("Lock timeout")
        }
    }
    
    // 3. Mutex с контекстом
    func contextMutex() {
        type ContextMutex struct {
            ch chan struct{}
        }
        
        func NewContextMutex() *ContextMutex {
            return &ContextMutex{
                ch: make(chan struct{}, 1),
            }
        }
        
        func (cm *ContextMutex) LockContext(ctx context.Context) error {
            select {
            case cm.ch <- struct{}{}:
                return nil
            case <-ctx.Done():
                return ctx.Err()
            }
        }
        
        func (cm *ContextMutex) Unlock() {
            <-cm.ch
        }
        
        // Использование
        ctx, cancel := context.WithTimeout(context.Background(), 200*time.Millisecond)
        defer cancel()
        
        ctxMu := NewContextMutex()
        if err := ctxMu.LockContext(ctx); err == nil {
            defer ctxMu.Unlock()
            fmt.Println("Lock acquired with context")
        } else {
            fmt.Printf("Failed to acquire lock: %v\n", err)
        }
    }
}
```

### 58. В чем разница между `Mutex` и `RWMutex`?

**Развернутый ответ:**
RWMutex позволяет множественное чтение, но эксклюзивную запись:

```go
// Сравнение Mutex и RWMutex
func mutexVsRWMutex() {
    data := make(map[string]int)
    
    // 1. Обычный Mutex - эксклюзивный доступ
    type MutexMap struct {
        mu sync.Mutex
        m  map[string]int
    }
    
    func (mm *MutexMap) Read(key string) int {
        mm.mu.Lock()         // блокирует ВСЕ операции
        defer mm.mu.Unlock()
        return mm.m[key]
    }
    
    func (mm *MutexMap) Write(key string, value int) {
        mm.mu.Lock()         // блокирует ВСЕ операции
        defer mm.mu.Unlock()
        mm.m[key] = value
    }
    
    // 2. RWMutex - читатели не блокируют друг друга
    type RWMutexMap struct {
        mu sync.RWMutex
        m  map[string]int
    }
    
    func (rwm *RWMutexMap) Read(key string) int {
        rwm.mu.RLock()         // множественные читатели OK
        defer rwm.mu.RUnlock()
        return rwm.m[key]
    }
    
    func (rwm *RWMutexMap) Write(key string, value int) {
        rwm.mu.Lock()          // эксклюзивная запись
        defer rwm.mu.Unlock()
        rwm.m[key] = value
    }
}

// Производительность RWMutex vs Mutex
func performanceComparison() {
    const (
        numReaders = 10
        numWrites  = 1000
        numReads   = 10000
    )
    
    data := make(map[int]int)
    for i := 0; i < 100; i++ {
        data[i] = i
    }
    
    // Тест с Mutex
    func testMutex() time.Duration {
        var mu sync.Mutex
        var wg sync.WaitGroup
        
        start := time.Now()
        
        // Читатели
        for i := 0; i < numReaders; i++ {
            wg.Add(1)
            go func() {
                defer wg.Done()
                for j := 0; j < numReads/numReaders; j++ {
                    mu.Lock()
                    _ = data[j%100]
                    mu.Unlock()
                }
            }()
        }
        
        // Писатель
        wg.Add(1)
        go func() {
            defer wg.Done()
            for i := 0; i < numWrites; i++ {
                mu.Lock()
                data[i%100] = i
                mu.Unlock()
            }
        }()
        
        wg.Wait()
        return time.Since(start)
    }
    
    // Тест с RWMutex
    func testRWMutex() time.Duration {
        var rwmu sync.RWMutex
        var wg sync.WaitGroup
        
        start := time.Now()
        
        // Читатели
        for i := 0; i < numReaders; i++ {
            wg.Add(1)
            go func() {
                defer wg.Done()
                for j := 0; j < numReads/numReaders; j++ {
                    rwmu.RLock()
                    _ = data[j%100]
                    rwmu.RUnlock()
                }
            }()
        }
        
        // Писатель
        wg.Add(1)
        go func() {
            defer wg.Done()
            for i := 0; i < numWrites; i++ {
                rwmu.Lock()
                data[i%100] = i
                rwmu.Unlock()
            }
        }()
        
        wg.Wait()
        return time.Since(start)
    }
    
    mutexTime := testMutex()
    rwMutexTime := testRWMutex()
    
    fmt.Printf("Mutex time: %v\n", mutexTime)
    fmt.Printf("RWMutex time: %v\n", rwMutexTime)
    fmt.Printf("RWMutex speedup: %.2fx\n", float64(mutexTime)/float64(rwMutexTime))
}

// Когда использовать RWMutex
func whenToUseRWMutex() {
    // RWMutex эффективен когда:
    // 1. Много читателей, мало писателей
    // 2. Критические секции чтения долгие
    // 3. Данные читаются часто
    
    // Пример: кэш с редкими обновлениями
    type Cache struct {
        mu   sync.RWMutex
        data map[string]interface{}
    }
    
    func NewCache() *Cache {
        return &Cache{
            data: make(map[string]interface{}),
        }
    }
    
    func (c *Cache) Get(key string) (interface{}, bool) {
        c.mu.RLock()         // множественные читатели
        defer c.mu.RUnlock()
        
        value, exists := c.data[key]
        return value, exists
    }
    
    func (c *Cache) Set(key string, value interface{}) {
        c.mu.Lock()          // эксклюзивная запись
        defer c.mu.Unlock()
        
        c.data[key] = value
    }
    
    func (c *Cache) GetOrCompute(key string, compute func() interface{}) interface{} {
        // Сначала пробуем прочитать
        c.mu.RLock()
        if value, exists := c.data[key]; exists {
            c.mu.RUnlock()
            return value
        }
        c.mu.RUnlock()
        
        // Если нет, вычисляем и записываем
        c.mu.Lock()
        defer c.mu.Unlock()
        
        // Double-check после получения write lock
        if value, exists := c.data[key]; exists {
            return value
        }
        
        value := compute()
        c.data[key] = value
        return value
    }
    
    // Использование
    cache := NewCache()
    
    // Много читателей
    var wg sync.WaitGroup
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            key := fmt.Sprintf("key_%d", id%10)
            
            value := cache.GetOrCompute(key, func() interface{} {
                time.Sleep(10 * time.Millisecond) // имитация вычисления
                return fmt.Sprintf("computed_value_%s", key)
            })
            
            fmt.Printf("Goroutine %d got: %v\n", id, value)
        }(i)
    }
    
    wg.Wait()
}

// Проблемы с RWMutex
func rwMutexProblems() {
    // 1. Writer starvation
    func writerStarvation() {
        var rwmu sync.RWMutex
        var data int
        
        // Много читателей
        for i := 0; i < 10; i++ {
            go func() {
                for {
                    rwmu.RLock()
                    _ = data
                    time.Sleep(1 * time.Millisecond)
                    rwmu.RUnlock()
                }
            }()
        }
        
        // Писатель может долго ждать
        go func() {
            for {
                rwmu.Lock()
                data++
                fmt.Printf("Writer updated data to: %d\n", data)
                rwmu.Unlock()
                time.Sleep(100 * time.Millisecond)
            }
        }()
        
        time.Sleep(1 * time.Second)
    }
    
    // 2. Lock upgrade невозможен
    func lockUpgradeIssue() {
        var rwmu sync.RWMutex
        var data map[string]int = make(map[string]int)
        
        // НЕПРАВИЛЬНО: нельзя обновить read lock до write lock
        badFunction := func(key string) {
            rwmu.RLock()
            if _, exists := data[key]; !exists {
                // rwmu.Lock() // DEADLOCK! Уже держим RLock
                // data[key] = 1
                // rwmu.Unlock()
            }
            rwmu.RUnlock()
        }
        
        // ПРАВИЛЬНО: сначала освобождаем read lock
        goodFunction := func(key string) {
            rwmu.RLock()
            _, exists := data[key]
            rwmu.RUnlock()
            
            if !exists {
                rwmu.Lock()
                // Double-check pattern
                if _, exists := data[key]; !exists {
                    data[key] = 1
                }
                rwmu.Unlock()
            }
        }
    }
}
```

### 59. Что такое `sync.WaitGroup` и как его использовать?

**Развернутый ответ:**
WaitGroup ждет завершения группы горутин:

```go
// Основы WaitGroup
func waitGroupBasics() {
    var wg sync.WaitGroup
    
    // Запускаем несколько горутин
    for i := 1; i <= 5; i++ {
        wg.Add(1) // увеличиваем счетчик на 1
        
        go func(id int) {
            defer wg.Done() // уменьшаем счетчик при завершении
            
            fmt.Printf("Goroutine %d starting\n", id)
            time.Sleep(time.Duration(id) * 100 * time.Millisecond)
            fmt.Printf("Goroutine %d finished\n", id)
        }(i)
    }
    
    fmt.Println("Waiting for all goroutines to finish...")
    wg.Wait() // ждем пока счетчик не станет 0
    fmt.Println("All goroutines finished!")
}

// Правильные паттерны использования
func waitGroupPatterns() {
    // Паттерн 1: Add перед запуском горутины
    func correctPattern() {
        var wg sync.WaitGroup
        
        for i := 0; i < 3; i++ {
            wg.Add(1) // ПРАВИЛЬНО: Add перед go
            go func(id int) {
                defer wg.Done()
                time.Sleep(100 * time.Millisecond)
                fmt.Printf("Task %d completed\n", id)
            }(i)
        }
        
        wg.Wait()
    }
    
    // Паттерн 2: WaitGroup в структуре
    type WorkerPool struct {
        wg       sync.WaitGroup
        jobQueue chan Job
    }
    
    type Job struct {
        ID   int
        Data string
    }
    
    func (wp *WorkerPool) Start(numWorkers int) {
        for i := 0; i < numWorkers; i++ {
            wp.wg.Add(1)
            go wp.worker(i)
        }
    }
    
    func (wp *WorkerPool) worker(id int) {
        defer wp.wg.Done()
        
        for job := range wp.jobQueue {
            fmt.Printf("Worker %d processing job %d: %s\n", id, job.ID, job.Data)
            time.Sleep(100 * time.Millisecond)
        }
    }
    
    func (wp *WorkerPool) Wait() {
        close(wp.jobQueue) // закрываем канал для завершения workers
        wp.wg.Wait()       // ждем завершения всех workers
    }
    
    // Использование
    pool := &WorkerPool{
        jobQueue: make(chan Job, 10),
    }
    
    pool.Start(3) // запускаем 3 workers
    
    // Отправляем задачи
    for i := 1; i <= 5; i++ {
        pool.jobQueue <- Job{ID: i, Data: fmt.Sprintf("task_%d", i)}
    }
    
    pool.Wait() // ждем завершения всех задач
}

// Ошибки при использовании WaitGroup
func waitGroupErrors() {
    // Ошибка 1: Add внутри горутины
    func wrongAddPlacement() {
        var wg sync.WaitGroup
        
        for i := 0; i < 3; i++ {
            go func(id int) {
                wg.Add(1) // НЕПРАВИЛЬНО: может вызвать race condition
                defer wg.Done()
                time.Sleep(100 * time.Millisecond)
            }(i)
        }
        
        wg.Wait() // может завершиться до того, как все Add вызовутся
    }
    
    // Ошибка 2: Копирование WaitGroup
    func copyWaitGroup() {
        var wg1 sync.WaitGroup
        wg2 := wg1 // НЕПРАВИЛЬНО: копируем WaitGroup
        
        wg1.Add(1)
        go func() {
            defer wg1.Done()
            time.Sleep(100 * time.Millisecond)
        }()
        
        wg2.Wait() // будет ждать навсегда, так как wg2 - это копия
    }
    
    // Ошибка 3: Повторное использование WaitGroup
    func reuseWaitGroup() {
        var wg sync.WaitGroup
        
        // Первое использование
        wg.Add(1)
        go func() {
            defer wg.Done()
            time.Sleep(50 * time.Millisecond)
        }()
        wg.Wait()
        
        // Повторное использование - ОПАСНО
        // Если счетчик стал отрицательным, будет panic
        wg.Add(1)
        go func() {
            defer wg.Done()
            time.Sleep(50 * time.Millisecond)
        }()
        wg.Wait()
    }
}

// Продвинутые паттерны с WaitGroup
func advancedWaitGroupPatterns() {
    // Паттерн 1: Ограниченный параллелизм
    func limitedParallelism() {
        const maxGoroutines = 3
        tasks := make([]int, 20)
        for i := range tasks {
            tasks[i] = i + 1
        }
        
        var wg sync.WaitGroup
        semaphore := make(chan struct{}, maxGoroutines)
        
        for _, task := range tasks {
            wg.Add(1)
            
            go func(taskID int) {
                defer wg.Done()
                
                semaphore <- struct{}{} // захватываем семафор
                defer func() { <-semaphore }() // освобождаем семафор
                
                fmt.Printf("Processing task %d\n", taskID)
                time.Sleep(200 * time.Millisecond)
                fmt.Printf("Task %d completed\n", taskID)
            }(task)
        }
        
        wg.Wait()
        fmt.Println("All tasks completed")
    }
    
    // Паттерн 2: Pipeline с WaitGroup
    func pipeline() {
        // Stage 1: числа
        numbers := make(chan int, 10)
        
        // Stage 2: квадраты
        squares := make(chan int, 10)
        
        // Stage 3: результат
        results := make(chan int, 10)
        
        var wg sync.WaitGroup
        
        // Generator
        wg.Add(1)
        go func() {
            defer wg.Done()
            defer close(numbers)
            
            for i := 1; i <= 10; i++ {
                numbers <- i
            }
        }()
        
        // Squarer
        wg.Add(1)
        go func() {
            defer wg.Done()
            defer close(squares)
            
            for num := range numbers {
                squares <- num * num
            }
        }()
        
        // Filter (только четные)
        wg.Add(1)
        go func() {
            defer wg.Done()
            defer close(results)
            
            for square := range squares {
                if square%2 == 0 {
                    results <- square
                }
            }
        }()
        
        // Consumer
        go func() {
            for result := range results {
                fmt.Printf("Even square: %d\n", result)
            }
        }()
        
        wg.Wait() // ждем завершения всех stages
        time.Sleep(100 * time.Millisecond) // даем время consumer'у
    }
    
    // Паттерн 3: Graceful shutdown
    func gracefulShutdown() {
        var wg sync.WaitGroup
        quit := make(chan struct{})
        
        // Запускаем workers
        for i := 1; i <= 3; i++ {
            wg.Add(1)
            go func(workerID int) {
                defer wg.Done()
                
                ticker := time.NewTicker(100 * time.Millisecond)
                defer ticker.Stop()
                
                for {
                    select {
                    case <-ticker.C:
                        fmt.Printf("Worker %d is working...\n", workerID)
                    case <-quit:
                        fmt.Printf("Worker %d shutting down\n", workerID)
                        return
                    }
                }
            }(i)
        }
        
        // Симулируем работу
        time.Sleep(500 * time.Millisecond)
        
        // Сигнализируем о завершении
        fmt.Println("Initiating shutdown...")
        close(quit)
        
        // Ждем завершения всех workers
        wg.Wait()
        fmt.Println("All workers stopped")
    }
    
    limitedParallelism()
    pipeline()
    gracefulShutdown()
}

// WaitGroup с таймаутом
func waitGroupWithTimeout() {
    var wg sync.WaitGroup
    
    // Функция для ожидания с таймаутом
    waitTimeout := func(wg *sync.WaitGroup, timeout time.Duration) bool {
        c := make(chan struct{})
        go func() {
            defer close(c)
            wg.Wait()
        }()
        
        select {
        case <-c:
            return false // завершилось вовремя
        case <-time.After(timeout):
            return true // таймаут
        }
    }
    
    // Запускаем долгую операцию
    wg.Add(1)
    go func() {
        defer wg.Done()
        time.Sleep(2 * time.Second) // долгая операция
    }()
    
    // Ждем с таймаутом
    if waitTimeout(&wg, 1*time.Second) {
        fmt.Println("Operation timed out")
    } else {
        fmt.Println("Operation completed")
    }
}
```

### 60. Как работает `sync.Once`?

**Развернутый ответ:**
Once гарантирует выполнение функции только один раз, даже при вызове из множественных горутин:

```go
// Основы sync.Once
func onceBasics() {
    var once sync.Once
    var result string
    
    // Функция, которая должна выполниться только один раз
    initFunction := func() {
        fmt.Println("Initializing...")
        time.Sleep(100 * time.Millisecond) // имитация дорогой операции
        result = "initialized"
        fmt.Println("Initialization complete")
    }
    
    var wg sync.WaitGroup
    
    // Запускаем несколько горутин
    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            fmt.Printf("Goroutine %d calling Do\n", id)
            once.Do(initFunction) // выполнится только один раз
            fmt.Printf("Goroutine %d got result: %s\n", id, result)
        }(i)
    }
    
    wg.Wait()
    // Вывод покажет, что "Initializing..." напечатается только один раз
}

// Типичные применения sync.Once
func onceUseCases() {
    // 1. Singleton паттерн
    type Database struct {
        connection string
    }
    
    var (
        dbInstance *Database
        dbOnce     sync.Once
    )
    
    func GetDatabase() *Database {
        dbOnce.Do(func() {
            fmt.Println("Creating database connection...")
            time.Sleep(50 * time.Millisecond) // имитация подключения
            dbInstance = &Database{connection: "connected"}
        })
        return dbInstance
    }
    
    // Тестируем singleton
    var wg sync.WaitGroup
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            db := GetDatabase()
            fmt.Printf("Goroutine %d got DB: %p\n", id, db)
        }(i)
    }
    wg.Wait()
    // Все горутины получат один и тот же экземпляр
    
    // 2. Ленивая инициализация конфигурации
    type Config struct {
        Settings map[string]string
    }
    
    var (
        config     *Config
        configOnce sync.Once
    )
    
    func loadConfig() *Config {
        configOnce.Do(func() {
            fmt.Println("Loading configuration...")
            config = &Config{
                Settings: map[string]string{
                    "host": "localhost",
                    "port": "8080",
                },
            }
        })
        return config
    }
    
    // 3. Регистрация обработчиков
    var (
        handlersRegistered bool
        registerOnce       sync.Once
    )
    
    func registerHandlers() {
        registerOnce.Do(func() {
            fmt.Println("Registering HTTP handlers...")
            // регистрация маршрутов
            handlersRegistered = true
        })
    }
}

// Продвинутые паттерны с Once
func advancedOncePatterns() {
    // 1. Once с возвратом ошибки
    type OnceWithError struct {
        once sync.Once
        err  error
    }
    
    func (o *OnceWithError) Do(f func() error) error {
        o.once.Do(func() {
            o.err = f()
        })
        return o.err
    }
    
    // Использование
    var initOnce OnceWithError
    
    initializeService := func() error {
        fmt.Println("Initializing service...")
        // Имитация потенциальной ошибки
        if rand.Float64() < 0.3 {
            return errors.New("initialization failed")
        }
        return nil
    }
    
    var wg sync.WaitGroup
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            err := initOnce.Do(initializeService)
            fmt.Printf("Goroutine %d: init error = %v\n", id, err)
        }(i)
    }
    wg.Wait()
    
    // 2. Once с результатом
    type OnceValue[T any] struct {
        once  sync.Once
        value T
    }
    
    func (o *OnceValue[T]) Do(f func() T) T {
        o.once.Do(func() {
            o.value = f()
        })
        return o.value
    }
    
    // Использование с дженериками (Go 1.18+)
    var expensiveComputation OnceValue[int]
    
    compute := func() int {
        fmt.Println("Performing expensive computation...")
        time.Sleep(100 * time.Millisecond)
        return 42
    }
    
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            result := expensiveComputation.Do(compute)
            fmt.Printf("Goroutine %d got result: %d\n", id, result)
        }(i)
    }
    wg.Wait()
    
    // 3. Множественные Once для разных ресурсов
    type ResourceManager struct {
        dbOnce     sync.Once
        cacheOnce  sync.Once
        loggerOnce sync.Once
        
        db     *Database
        cache  *Cache
        logger *Logger
    }
    
    func (rm *ResourceManager) GetDB() *Database {
        rm.dbOnce.Do(func() {
            fmt.Println("Initializing database...")
            rm.db = &Database{connection: "db_connected"}
        })
        return rm.db
    }
    
    func (rm *ResourceManager) GetCache() *Cache {
        rm.cacheOnce.Do(func() {
            fmt.Println("Initializing cache...")
            rm.cache = &Cache{data: make(map[string]interface{})}
        })
        return rm.cache
    }
    
    func (rm *ResourceManager) GetLogger() *Logger {
        rm.loggerOnce.Do(func() {
            fmt.Println("Initializing logger...")
            rm.logger = &Logger{level: "INFO"}
        })
        return rm.logger
    }
}

// Ошибки при использовании Once
func onceErrors() {
    // Ошибка 1: Панический вызов Once.Do
    func panicInOnce() {
        var once sync.Once
        
        defer func() {
            if r := recover(); r != nil {
                fmt.Printf("Recovered from panic: %v\n", r)
            }
        }()
        
        once.Do(func() {
            panic("something went wrong")
        })
        
        // Повторный вызов не выполнится, даже после panic
        once.Do(func() {
            fmt.Println("This will not execute")
        })
    }
    
    // Ошибка 2: Копирование Once
    func copyOnce() {
        var once1 sync.Once
        once2 := once1 // копируем Once
        
        once1.Do(func() {
            fmt.Println("Called on once1")
        })
        
        // once2 - это копия, независимая от once1
        once2.Do(func() {
            fmt.Println("Called on once2") // выполнится!
        })
    }
    
    // Ошибка 3: Рекурсивный вызов Once.Do
    func recursiveOnce() {
        var once sync.Once
        
        var f func()
        f = func() {
            fmt.Println("In function f")
            once.Do(f) // DEADLOCK! Рекурсивный вызов
        }
        
        // once.Do(f) // это вызовет deadlock
    }
    
    panicInOnce()
    copyOnce()
    // recursiveOnce() // закомментировано, чтобы избежать deadlock
}

// Реальный пример: инициализация HTTP клиента
func httpClientOnce() {
    var (
        httpClient *http.Client
        clientOnce sync.Once
    )
    
    func getHTTPClient() *http.Client {
        clientOnce.Do(func() {
            fmt.Println("Creating HTTP client...")
            httpClient = &http.Client{
                Timeout: 30 * time.Second,
                Transport: &http.Transport{
                    MaxIdleConns:        100,
                    MaxIdleConnsPerHost: 10,
                    IdleConnTimeout:     90 * time.Second,
                },
            }
        })
        return httpClient
    }
    
    // Использование в множественных горутинах
    var wg sync.WaitGroup
    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            client := getHTTPClient()
            fmt.Printf("Goroutine %d got HTTP client: %p\n", id, client)
            
            // Делаем запрос (в реальном коде)
            // resp, err := client.Get("https://example.com")
        }(i)
    }
    wg.Wait()
}

// Вспомогательные типы для примеров
type Cache struct {
    data map[string]interface{}
}

type Logger struct {
    level string
}
```

*Продолжение со следующими разделами...*