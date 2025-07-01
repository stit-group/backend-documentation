# Go для Backend разработчика: Полное руководство от нуля до эксперта

> **Цель курса:** Пройти путь от полного новичка до уверенного Go backend разработчика за 3-4 месяца

## 🎯 Зачем изучать Go для backend?

### Диаграмма: Go vs другие языки для backend

```
Performance & Concurrency Comparison
┌─────────────────────────────────────────────────────────────┐
│ Language │ Performance │ Concurrency │ Memory │ Learning │   │
│          │    (1-10)   │   (1-10)    │ Usage  │  Curve   │   │
├─────────────────────────────────────────────────────────────┤
│ Go       │      9      │      10     │  Low   │  Easy    │ ✅ │
│ Node.js  │      6      │       7     │ Medium │  Easy    │   │
│ Python   │      4      │       5     │  High  │  Easy    │   │
│ Java     │      8      │       8     │  High  │  Hard    │   │
│ C++      │     10      │       6     │  Low   │  Hard    │   │
│ Rust     │     10      │       9     │  Low   │  Hard    │   │
└─────────────────────────────────────────────────────────────┘
```

### Архитектура Go программы

```
Go Program Architecture
┌─────────────────────────────────────────────────────────────┐
│                      Go Binary                              │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │   main()    │  │ Goroutines  │  │   Memory    │          │
│  │  Package    │  │   Manager   │  │   Manager   │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │  Standard   │  │   Custom    │  │  External   │          │
│  │  Library    │  │  Packages   │  │ Dependencies│          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
├─────────────────────────────────────────────────────────────┤
│                    Go Runtime                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ Scheduler   │  │     GC      │  │  Network    │          │
│  │ (M:N model) │  │ (Concurrent)│  │   Poller    │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────────────────────┘
```

**Go решает главные проблемы backend разработки:**
- **🚀 Высокая производительность** - компилируется в нативный код, выполняется без VM
- **⚡ Простота параллелизма** - горутины и каналы из коробки  
- **⏱️ Быстрая компиляция** - мгновенная сборка даже больших проектов (секунды vs минуты)
- **📚 Отличная стандартная библиотека** - HTTP-сервер, JSON, работа с БД без внешних зависимостей
- **🛡️ Статическая типизация** - меньше ошибок в runtime, лучше производительность
- **🔧 Простой деплой** - один исполняемый файл без зависимостей

---

## 📚 Этап 1: Основы языка Go (2-3 недели)

### Неделя 1: Синтаксис и базовые конструкции

#### 🔧 Настройка окружения

```bash
# Проверка установки
go version

# Создание нового проекта
mkdir my-backend-app
cd my-backend-app
go mod init github.com/username/my-backend-app

# Структура проекта
my-backend-app/
├── go.mod                 # Описание модуля и зависимостей
├── go.sum                 # Хеши зависимостей для безопасности
├── main.go               # Точка входа
├── internal/             # Приватные пакеты приложения
│   ├── handlers/         # HTTP обработчики
│   ├── services/         # Бизнес-логика
│   └── repository/       # Работа с данными
├── pkg/                  # Публичные пакеты
└── cmd/                  # Исполняемые файлы
```

#### 📦 Система пакетов Go

```
Package System Visualization
┌─────────────────────────────────────────────────────────────┐
│                     Go Module                               │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │   package   │  │   package   │  │   package   │          │
│  │    main     │  │   handlers  │  │   models    │          │
│  │             │  │             │  │             │          │
│  │ func main() │  │ func Handle │  │ type User   │          │
│  │             │  │             │  │             │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│         │                │                │                │
│         ▼                ▼                ▼                │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │               Exported Functions/Types                  │ │
│  │              (начинаются с заглавной)                   │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

```go
// main.go - точка входа приложения
package main

import (
    "fmt"
    "log"
    "net/http"
    
    // Локальные пакеты
    "github.com/username/my-backend-app/internal/handlers"
    "github.com/username/my-backend-app/internal/services"
    
    // Внешние зависимости
    "github.com/gorilla/mux"
)

func main() {
    // Инициализация сервисов
    userService := services.NewUserService()
    userHandler := handlers.NewUserHandler(userService)
    
    // Настройка роутера
    router := mux.NewRouter()
    router.HandleFunc("/users", userHandler.GetUsers).Methods("GET")
    
    fmt.Println("🚀 Backend server starting on :8080")
    log.Fatal(http.ListenAndServe(":8080", router))
}
```

**💡 Ключевая концепция:** Каждый файл принадлежит пакету. `main` - специальный пакет для исполняемых программ.

#### 🔢 Переменные и типы данных

### Система типов Go

```
Go Type System
┌─────────────────────────────────────────────────────────────┐
│                     Basic Types                             │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │   Numbers   │  │   Strings   │  │   Boolean   │          │
│  │             │  │             │  │             │          │
│  │ int, int64  │  │   string    │  │    bool     │          │
│  │ float64     │  │   rune      │  │             │          │
│  │ uint        │  │   byte      │  │             │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
├─────────────────────────────────────────────────────────────┤
│                   Composite Types                           │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │   Arrays    │  │   Slices    │  │    Maps     │          │
│  │ [5]int      │  │   []int     │  │ map[string] │          │
│  │             │  │             │  │    int      │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │   Structs   │  │ Interfaces  │  │  Channels   │          │
│  │ type User   │  │ type Reader │  │  chan int   │          │
│  │   struct    │  │ interface   │  │             │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────────────────────┘
```

```go
package main

import (
    "fmt"
    "time"
)

// Глобальные константы (лучшая практика - использовать для конфигурации)
const (
    APIVersion = "v1"
    MaxRetries = 3
    TimeoutSeconds = 30
)

func demonstrateTypes() {
    // Основные типы для backend разработки
    var userID int64 = 12345678901234567  // int64 для больших ID
    var accountBalance float64 = 1234.56   // float64 для денежных сумм
    var isActive bool = true               // bool для флагов
    var userName string = "john_doe"       // string для текстовых данных
    
    // Краткое объявление (предпочтительный способ)
    email := "john@example.com"
    createdAt := time.Now()
    
    // Нулевые значения (важно для проверок)
    var uninitializedString string    // ""
    var uninitializedInt int         // 0
    var uninitializedBool bool       // false
    var uninitializedPointer *string // nil
    
    fmt.Printf("User: %d, %s, %s, Balance: %.2f, Active: %v\n", 
        userID, userName, email, accountBalance, isActive)
    fmt.Printf("Created: %v\n", createdAt)
    fmt.Printf("Zero values: '%s', %d, %v, %v\n", 
        uninitializedString, uninitializedInt, uninitializedBool, uninitializedPointer)
}

// Пример типичной структуры данных для API
type APIUser struct {
    ID        int64     `json:"id"`
    Email     string    `json:"email"`
    Name      string    `json:"name"`
    Balance   float64   `json:"balance"`
    IsActive  bool      `json:"is_active"`
    CreatedAt time.Time `json:"created_at"`
}
```

**🎯 Практическая польза:** Строгая типизация предотвращает множество ошибок, которые возникают в динамически типизированных языках при работе с API данными.

#### 🔄 Управляющие конструкции

### Диаграмма потока выполнения

```
Control Flow in Go
┌─────────────────────────────────────────────────────────────┐
│                    Conditional Logic                        │
├─────────────────────────────────────────────────────────────┤
│              if condition {                                 │
│                  // code                                    │
│              } else if condition2 {                         │
│                  // code                                    │
│              } else {                                       │
│                  // code                                    │
│              }                                              │
├─────────────────────────────────────────────────────────────┤
│                     Switch Logic                            │
├─────────────────────────────────────────────────────────────┤
│              switch value {                                 │
│              case "GET":                                    │
│                  // handle GET                             │
│              case "POST":                                   │
│                  // handle POST                            │
│              default:                                       │
│                  // handle others                          │
│              }                                              │
├─────────────────────────────────────────────────────────────┤
│                      Loops                                  │
├─────────────────────────────────────────────────────────────┤
│              for i := 0; i < len(items); i++ {             │
│                  // index-based loop                       │
│              }                                              │
│                                                             │
│              for index, value := range items {             │
│                  // range-based loop                       │
│              }                                              │
│                                                             │
│              for condition {                                │
│                  // while-like loop                        │
│              }                                              │
└─────────────────────────────────────────────────────────────┘
```

```go
package main

import (
    "fmt"
    "net/http"
    "strings"
)

// Условные конструкции в контексте HTTP обработчика
func handleHTTPMethod(method string) string {
    // if-else для простых условий
    if method == "" {
        return "Method not specified"
    }
    
    // switch для множественного выбора (идеально для HTTP методов)
    switch strings.ToUpper(method) {
    case "GET":
        return "Reading data"
    case "POST":
        return "Creating data"
    case "PUT":
        return "Updating data"
    case "DELETE":
        return "Deleting data"
    case "PATCH":
        return "Partially updating data"
    default:
        return "Unsupported method"
    }
}

// Циклы для обработки коллекций данных
func processUsers() {
    users := []string{"alice@example.com", "bob@example.com", "charlie@example.com"}
    
    // Классический цикл for (редко используется)
    fmt.Println("Classic for loop:")
    for i := 0; i < len(users); i++ {
        fmt.Printf("User %d: %s\n", i+1, users[i])
    }
    
    // Range цикл (предпочтительный способ)
    fmt.Println("\nRange loop:")
    for index, email := range users {
        fmt.Printf("Index %d: %s\n", index, email)
    }
    
    // Range только по значениям (когда индекс не нужен)
    fmt.Println("\nValues only:")
    for _, email := range users {
        if strings.Contains(email, "@example.com") {
            fmt.Printf("Valid email: %s\n", email)
        }
    }
    
    // "Бесконечный" цикл с условием выхода
    counter := 0
    for {
        counter++
        if counter > 3 {
            break
        }
        fmt.Printf("Processing batch %d\n", counter)
    }
}

// Практический пример: валидация HTTP заголовков
func validateHeaders(headers http.Header) []string {
    var errors []string
    
    requiredHeaders := []string{"Content-Type", "Authorization"}
    
    for _, header := range requiredHeaders {
        if value := headers.Get(header); value == "" {
            errors = append(errors, fmt.Sprintf("Missing required header: %s", header))
        }
    }
    
    return errors
}
```

**💡 Ключевая концепция:** В Go только один тип цикла `for`, но он может работать в разных режимах. Range цикл - идиоматичный способ итерации.

#### ⚡ Функции и обработка ошибок

### Архитектура обработки ошибок в Go

```
Error Handling Philosophy
┌─────────────────────────────────────────────────────────────┐
│                    Go Error Model                           │
├─────────────────────────────────────────────────────────────┤
│   ┌─────────────────┐    ┌─────────────────┐               │
│   │   Happy Path    │    │   Error Path    │               │
│   │                 │    │                 │               │
│   │ Normal Return   │    │ Error Return    │               │
│   │ (value, nil)    │    │ (zero, error)   │               │
│   └─────────────────┘    └─────────────────┘               │
│           │                        │                       │
│           ▼                        ▼                       │
│   ┌─────────────────┐    ┌─────────────────┐               │
│   │  Continue       │    │  Handle Error   │               │
│   │  Execution      │    │  - Log          │               │
│   │                 │    │  - Retry        │               │
│   │                 │    │  - Return       │               │
│   │                 │    │  - Panic        │               │
│   └─────────────────┘    └─────────────────┘               │
├─────────────────────────────────────────────────────────────┤
│              Explicit > Implicit                            │
│         "Errors are values" - Rob Pike                      │
└─────────────────────────────────────────────────────────────┘
```

```go
package main

import (
    "errors"
    "fmt"
    "log"
    "regexp"
    "strings"
    "time"
)

// Пользовательские типы ошибок для лучшей диагностики
type ValidationError struct {
    Field   string
    Value   string
    Message string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("validation failed for field '%s': %s", e.Field, e.Message)
}

// Функция с множественным возвращением (идиоматично для Go)
func validateEmail(email string) (bool, error) {
    if email == "" {
        return false, ValidationError{
            Field:   "email",
            Value:   email,
            Message: "email cannot be empty",
        }
    }
    
    // Простая проверка email
    emailRegex := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
    if !emailRegex.MatchString(email) {
        return false, ValidationError{
            Field:   "email",
            Value:   email,
            Message: "invalid email format",
        }
    }
    
    return true, nil
}

// Функция с различными стратегиями обработки ошибок
func processUserRegistration(email, password string) error {
    // Проверка email
    if valid, err := validateEmail(email); err != nil {
        // Возвращаем ошибку валидации
        return fmt.Errorf("email validation failed: %w", err)
    } else if !valid {
        return errors.New("email validation failed for unknown reason")
    }
    
    // Проверка пароля
    if len(password) < 8 {
        return ValidationError{
            Field:   "password",
            Value:   "***",
            Message: "password must be at least 8 characters",
        }
    }
    
    // Симуляция сохранения в базу данных
    if err := saveUserToDB(email, password); err != nil {
        // Оборачиваем ошибку для добавления контекста
        return fmt.Errorf("failed to save user to database: %w", err)
    }
    
    log.Printf("User successfully registered: %s", email)
    return nil
}

// Симуляция функции работы с БД
func saveUserToDB(email, password string) error {
    // Симуляция сетевой задержки
    time.Sleep(10 * time.Millisecond)
    
    // Симуляция ошибки БД (10% вероятность)
    if strings.Contains(email, "error") {
        return errors.New("database connection failed")
    }
    
    return nil
}

// Разные стратегии обработки ошибок
func demonstrateErrorHandling() {
    testEmails := []string{
        "valid@example.com",
        "",
        "invalid-email",
        "error@example.com", // Триггерит ошибку БД
    }
    
    for _, email := range testEmails {
        fmt.Printf("\n--- Processing: %s ---\n", email)
        
        err := processUserRegistration(email, "password123")
        if err != nil {
            // Проверка типа ошибки
            var validationErr ValidationError
            if errors.As(err, &validationErr) {
                fmt.Printf("❌ Validation Error: %s\n", validationErr.Message)
            } else {
                fmt.Printf("❌ System Error: %v\n", err)
            }
        } else {
            fmt.Println("✅ User registered successfully")
        }
    }
}

// Функции для различных случаев возврата
func getUser(id int64) (*User, error) {
    if id <= 0 {
        return nil, errors.New("invalid user ID")
    }
    
    // Симуляция поиска пользователя
    return &User{ID: id, Email: "user@example.com"}, nil
}

func getUserCount() (int, error) {
    // Симуляция подсчета пользователей
    return 42, nil
}

func deleteUser(id int64) error {
    if id <= 0 {
        return errors.New("invalid user ID")
    }
    
    // Симуляция удаления
    return nil
}

type User struct {
    ID    int64  `json:"id"`
    Email string `json:"email"`
    Name  string `json:"name"`
}
```

**🎯 Практическая польза:** Явная обработка ошибок в Go предотвращает неожиданные падения сервера и позволяет создавать надежные системы.

**💡 Ключевые принципы:**
- Ошибки - это значения, не исключения
- Всегда проверяйте ошибки явно
- Оборачивайте ошибки для добавления контекста
- Используйте пользовательские типы ошибок для лучшей диагностики

---

### Неделя 2: Структуры данных и указатели

#### 📋 Слайсы - динамические массивы

### Внутреннее устройство слайсов

```
Slice Internal Structure
┌─────────────────────────────────────────────────────────────┐
│                      Slice Header                           │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │   Pointer   │  │   Length    │  │  Capacity   │          │
│  │      │      │  │     3       │  │     5       │          │
│  └──────┼──────┘  └─────────────┘  └─────────────┘          │
│         │                                                   │
│         ▼                                                   │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │           Underlying Array                              │ │
│  │  [0]  │  [1]  │  [2]  │  [3]  │  [4]  │ ...           │ │
│  │ "Go"  │ "is"  │ "fun" │  nil  │  nil  │               │ │
│  └─────────────────────────────────────────────────────────┘ │
│         ▲               ▲                                   │
│      Used part      Unused capacity                        │
└─────────────────────────────────────────────────────────────┘
```

```go
package main

import (
    "fmt"
    "strings"
)

// Демонстрация работы со слайсами в контексте backend
func demonstrateSlices() {
    // Создание слайсов различными способами
    
    // 1. Литерал слайса
    supportedMethods := []string{"GET", "POST", "PUT", "DELETE"}
    fmt.Printf("Supported HTTP methods: %v\n", supportedMethods)
    fmt.Printf("Length: %d, Capacity: %d\n", len(supportedMethods), cap(supportedMethods))
    
    // 2. Make функция (когда знаем примерный размер)
    userIDs := make([]int64, 0, 100) // length=0, capacity=100
    fmt.Printf("User IDs: %v, len=%d, cap=%d\n", userIDs, len(userIDs), cap(userIDs))
    
    // 3. Добавление элементов
    userIDs = append(userIDs, 1001, 1002, 1003)
    fmt.Printf("After append: %v, len=%d, cap=%d\n", userIDs, len(userIDs), cap(userIDs))
    
    // 4. Срезы (slicing)
    recentUsers := userIDs[1:3] // элементы с индекса 1 до 3 (не включая 3)
    fmt.Printf("Recent users: %v\n", recentUsers)
    
    // 5. Копирование слайсов (важно для избежания неожиданных изменений)
    originalTags := []string{"backend", "go", "api"}
    copiedTags := make([]string, len(originalTags))
    copy(copiedTags, originalTags)
    
    copiedTags[0] = "frontend" // Изменяем копию
    fmt.Printf("Original: %v\n", originalTags)
    fmt.Printf("Copied: %v\n", copiedTags)
}

// Практические применения слайсов в backend
type APIResponse struct {
    Data    []interface{} `json:"data"`
    Meta    ResponseMeta  `json:"meta"`
    Errors  []string      `json:"errors,omitempty"`
}

type ResponseMeta struct {
    Total  int `json:"total"`
    Page   int `json:"page"`
    Limit  int `json:"limit"`
}

// Функция для пагинации данных
func paginateUsers(users []User, page, limit int) ([]User, ResponseMeta) {
    total := len(users)
    start := (page - 1) * limit
    end := start + limit
    
    // Проверка границ
    if start >= total {
        return []User{}, ResponseMeta{Total: total, Page: page, Limit: limit}
    }
    
    if end > total {
        end = total
    }
    
    paginatedUsers := users[start:end]
    meta := ResponseMeta{
        Total: total,
        Page:  page,
        Limit: limit,
    }
    
    return paginatedUsers, meta
}

// Фильтрация данных
func filterActiveUsers(users []User) []User {
    var activeUsers []User
    
    for _, user := range users {
        if user.IsActive {
            activeUsers = append(activeUsers, user)
        }
    }
    
    return activeUsers
}

// Bulk операции
func validateEmails(emails []string) ([]string, []string) {
    var validEmails []string
    var invalidEmails []string
    
    for _, email := range emails {
        if strings.Contains(email, "@") && strings.Contains(email, ".") {
            validEmails = append(validEmails, email)
        } else {
            invalidEmails = append(invalidEmails, email)
        }
    }
    
    return validEmails, invalidEmails
}

// Демонстрация производительности слайсов
func demonstrateSlicePerformance() {
    // Предварительное выделение памяти для лучшей производительности
    userIDs := make([]int64, 0, 1000) // Заранее выделяем память на 1000 элементов
    
    for i := int64(1); i <= 1000; i++ {
        userIDs = append(userIDs, i)
    }
    
    fmt.Printf("Generated %d user IDs efficiently\n", len(userIDs))
}

type User struct {
    ID       int64  `json:"id"`
    Email    string `json:"email"`
    Name     string `json:"name"`
    IsActive bool   `json:"is_active"`
}
```

**🎯 Практическая польза:**
- **Пагинация** - слайсы идеальны для разбиения больших наборов данных
- **Фильтрация** - легко создавать подмножества данных
- **Bulk операции** - обработка множества элементов за раз
- **API responses** - гибкие структуры данных для JSON

#### 🗂️ Карты - key-value хранилище

### Архитектура карт в Go

```
Map Internal Structure (Hash Table)
┌─────────────────────────────────────────────────────────────┐
│                      Hash Table                             │
├─────────────────────────────────────────────────────────────┤
│     Key       │    Hash     │    Bucket    │    Value       │
├─────────────────────────────────────────────────────────────┤
│ "user:123"    │ hash("...") │ bucket[0]    │ User{...}      │
│ "user:456"    │ hash("...") │ bucket[1]    │ User{...}      │
│ "session:abc" │ hash("...") │ bucket[2]    │ Session{...}   │
├─────────────────────────────────────────────────────────────┤
│                    Collision Handling                       │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│  │   Bucket    │    │   Bucket    │    │   Bucket    │      │
│  │  [Entry1]   │    │  [Entry1]   │    │  [Entry1]   │      │
│  │  [Entry2]   │    │      │      │    │             │      │
│  │      │      │    │      ▼      │    │             │      │
│  │   Overflow  │    │   Next      │    │             │      │
│  └─────────────┘    └─────────────┘    └─────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Демонстрация различных применений карт в backend
func demonstrateMaps() {
    // 1. Кэш в памяти (самое частое применение)
    userCache := make(map[string]*User)
    
    // Добавление в кэш
    userCache["user:123"] = &User{ID: 123, Email: "john@example.com", Name: "John"}
    userCache["user:456"] = &User{ID: 456, Email: "jane@example.com", Name: "Jane"}
    
    // Проверка существования (критически важно!)
    if user, exists := userCache["user:123"]; exists {
        fmt.Printf("Found in cache: %s\n", user.Name)
    } else {
        fmt.Println("User not found in cache")
    }
    
    // Удаление из кэша
    delete(userCache, "user:456")
    
    // 2. Счетчики и статистика
    requestCounts := make(map[string]int)
    endpoints := []string{"/users", "/posts", "/users", "/comments", "/users"}
    
    for _, endpoint := range endpoints {
        requestCounts[endpoint]++
    }
    
    fmt.Println("Request statistics:")
    for endpoint, count := range requestCounts {
        fmt.Printf("%s: %d requests\n", endpoint, count)
    }
    
    // 3. Конфигурация приложения
    config := map[string]interface{}{
        "database_url":    "postgres://localhost:5432/myapp",
        "redis_url":       "redis://localhost:6379",
        "max_connections": 100,
        "debug_mode":      true,
        "allowed_origins": []string{"localhost:3000", "myapp.com"},
    }
    
    if dbURL, ok := config["database_url"].(string); ok {
        fmt.Printf("Database URL: %s\n", dbURL)
    }
}

// Thread-safe кэш с мьютексом
type SafeCache struct {
    mu    sync.RWMutex
    cache map[string]*User
}

func NewSafeCache() *SafeCache {
    return &SafeCache{
        cache: make(map[string]*User),
    }
}

func (c *SafeCache) Set(key string, user *User) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.cache[key] = user
}

func (c *SafeCache) Get(key string) (*User, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    user, exists := c.cache[key]
    return user, exists
}

func (c *SafeCache) Delete(key string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    delete(c.cache, key)
}

func (c *SafeCache) Size() int {
    c.mu.RLock()
    defer c.mu.RUnlock()
    return len(c.cache)
}

// Кэш с TTL (Time To Live)
type CacheEntry struct {
    User      *User
    ExpiresAt time.Time
}

type TTLCache struct {
    mu      sync.RWMutex
    entries map[string]*CacheEntry
}

func NewTTLCache() *TTLCache {
    cache := &TTLCache{
        entries: make(map[string]*CacheEntry),
    }
    
    // Запускаем горутину для очистки истекших записей
    go cache.cleanup()
    
    return cache
}

func (c *TTLCache) Set(key string, user *User, ttl time.Duration) {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    c.entries[key] = &CacheEntry{
        User:      user,
        ExpiresAt: time.Now().Add(ttl),
    }
}

func (c *TTLCache) Get(key string) (*User, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    
    entry, exists := c.entries[key]
    if !exists {
        return nil, false
    }
    
    if time.Now().After(entry.ExpiresAt) {
        return nil, false
    }
    
    return entry.User, true
}

func (c *TTLCache) cleanup() {
    ticker := time.NewTicker(1 * time.Minute)
    defer ticker.Stop()
    
    for range ticker.C {
        c.mu.Lock()
        for key, entry := range c.entries {
            if time.Now().After(entry.ExpiresAt) {
                delete(c.entries, key)
            }
        }
        c.mu.Unlock()
    }
}

// Группировка данных с помощью карт
func groupUsersByDomain(users []User) map[string][]User {
    groups := make(map[string][]User)
    
    for _, user := range users {
        // Извлекаем домен из email
        parts := strings.Split(user.Email, "@")
        if len(parts) == 2 {
            domain := parts[1]
            groups[domain] = append(groups[domain], user)
        }
    }
    
    return groups
}

// Индексирование для быстрого поиска
type UserIndex struct {
    byID    map[int64]*User
    byEmail map[string]*User
    byName  map[string][]*User // Имена могут повторяться
}

func NewUserIndex() *UserIndex {
    return &UserIndex{
        byID:    make(map[int64]*User),
        byEmail: make(map[string]*User),
        byName:  make(map[string][]*User),
    }
}

func (idx *UserIndex) AddUser(user *User) {
    idx.byID[user.ID] = user
    idx.byEmail[user.Email] = user
    idx.byName[user.Name] = append(idx.byName[user.Name], user)
}

func (idx *UserIndex) FindByID(id int64) *User {
    return idx.byID[id]
}

func (idx *UserIndex) FindByEmail(email string) *User {
    return idx.byEmail[email]
}

func (idx *UserIndex) FindByName(name string) []*User {
    return idx.byName[name]
}
```

**🎯 Практическая польза:**
- **Кэширование** - мгновенный доступ к часто используемым данным
- **Индексирование** - быстрый поиск по различным полям
- **Конфигурация** - гибкое хранение настроек приложения
- **Статистика** - подсчет событий и метрик

#### 🏗️ Структуры - модели данных

### Структуры в архитектуре приложения

```
Struct Usage in Application Architecture
┌─────────────────────────────────────────────────────────────┐
│                     Domain Models                           │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │    User     │  │    Post     │  │   Comment   │          │
│  │             │  │             │  │             │          │
│  │ - ID        │  │ - ID        │  │ - ID        │          │
│  │ - Email     │  │ - Title     │  │ - Content   │          │
│  │ - Name      │  │ - Content   │  │ - UserID    │          │
│  │ - CreatedAt │  │ - UserID    │  │ - PostID    │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
├─────────────────────────────────────────────────────────────┤
│                    Data Transfer Objects                    │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ UserRequest │  │UserResponse │  │  ErrorDTO   │          │
│  │             │  │             │  │             │          │
│  │ - Email     │  │ - ID        │  │ - Code      │          │
│  │ - Name      │  │ - Email     │  │ - Message   │          │
│  │ - Password  │  │ - Name      │  │ - Details   │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
├─────────────────────────────────────────────────────────────┤
│                  Configuration Structs                     │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ DatabaseCfg │  │  ServerCfg  │  │  RedisCfg   │          │
│  │             │  │             │  │             │          │
│  │ - Host      │  │ - Port      │  │ - URL       │          │
│  │ - Port      │  │ - Host      │  │ - Password  │          │
│  │ - Database  │  │ - ReadTimeout│ │ - Database  │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────────────────────┘
```

```go
package main

import (
    "encoding/json"
    "fmt"
    "time"
    "database/sql/driver"
    "strings"
)

// Базовая модель для всех сущностей
type BaseModel struct {
    ID        int64     `json:"id" db:"id"`
    CreatedAt time.Time `json:"created_at" db:"created_at"`
    UpdatedAt time.Time `json:"updated_at" db:"updated_at"`
}

// Пользователь - основная модель
type User struct {
    BaseModel
    Email     string    `json:"email" db:"email" validate:"required,email"`
    Name      string    `json:"name" db:"name" validate:"required,min=2,max=50"`
    Password  string    `json:"-" db:"password_hash"` // "-" скрывает поле в JSON
    IsActive  bool      `json:"is_active" db:"is_active"`
    Role      UserRole  `json:"role" db:"role"`
    Profile   *Profile  `json:"profile,omitempty"` // Pointer для optional поля
}

// Enum для роли пользователя
type UserRole string

const (
    RoleUser  UserRole = "user"
    RoleAdmin UserRole = "admin"
    RoleMod   UserRole = "moderator"
)

// Реализация интерфейса для работы с БД
func (r UserRole) Value() (driver.Value, error) {
    return string(r), nil
}

func (r *UserRole) Scan(value interface{}) error {
    if value == nil {
        *r = RoleUser
        return nil
    }
    if s, ok := value.(string); ok {
        *r = UserRole(s)
        return nil
    }
    return fmt.Errorf("cannot scan %T into UserRole", value)
}

// Профиль пользователя (отдельная таблица)
type Profile struct {
    UserID   int64  `json:"user_id" db:"user_id"`
    Avatar   string `json:"avatar" db:"avatar"`
    Bio      string `json:"bio" db:"bio"`
    Website  string `json:"website" db:"website"`
    Location string `json:"location" db:"location"`
}

// Методы для бизнес-логики
func (u *User) IsValid() bool {
    return u.Email != "" && u.Name != "" && len(u.Name) >= 2
}

func (u *User) CanModerate() bool {
    return u.Role == RoleAdmin || u.Role == RoleMod
}

func (u *User) GetDisplayName() string {
    if u.Name != "" {
        return u.Name
    }
    return strings.Split(u.Email, "@")[0]
}

// DTO для создания пользователя
type CreateUserRequest struct {
    Email    string `json:"email" validate:"required,email"`
    Name     string `json:"name" validate:"required,min=2,max=50"`
    Password string `json:"password" validate:"required,min=8"`
}

// DTO для ответа API
type UserResponse struct {
    ID        int64     `json:"id"`
    Email     string    `json:"email"`
    Name      string    `json:"name"`
    IsActive  bool      `json:"is_active"`
    Role      UserRole  `json:"role"`
    CreatedAt time.Time `json:"created_at"`
    Profile   *Profile  `json:"profile,omitempty"`
}

// Конвертация между моделями
func (u *User) ToResponse() UserResponse {
    return UserResponse{
        ID:        u.ID,
        Email:     u.Email,
        Name:      u.Name,
        IsActive:  u.IsActive,
        Role:      u.Role,
        CreatedAt: u.CreatedAt,
        Profile:   u.Profile,
    }
}

func (req *CreateUserRequest) ToUser() *User {
    return &User{
        BaseModel: BaseModel{
            CreatedAt: time.Now(),
            UpdatedAt: time.Now(),
        },
        Email:    req.Email,
        Name:     req.Name,
        Password: req.Password, // В реальности нужно хешировать
        IsActive: true,
        Role:     RoleUser,
    }
}

// Структура для пагинации
type PaginatedResponse struct {
    Data       interface{}    `json:"data"`
    Pagination PaginationMeta `json:"pagination"`
}

type PaginationMeta struct {
    Page       int `json:"page"`
    Limit      int `json:"limit"`
    Total      int `json:"total"`
    TotalPages int `json:"total_pages"`
    HasNext    bool `json:"has_next"`
    HasPrev    bool `json:"has_prev"`
}

func NewPaginatedResponse(data interface{}, page, limit, total int) PaginatedResponse {
    totalPages := (total + limit - 1) / limit
    
    return PaginatedResponse{
        Data: data,
        Pagination: PaginationMeta{
            Page:       page,
            Limit:      limit,
            Total:      total,
            TotalPages: totalPages,
            HasNext:    page < totalPages,
            HasPrev:    page > 1,
        },
    }
}

// Конфигурация приложения
type Config struct {
    Server   ServerConfig   `json:"server"`
    Database DatabaseConfig `json:"database"`
    Redis    RedisConfig    `json:"redis"`
    JWT      JWTConfig      `json:"jwt"`
    Email    EmailConfig    `json:"email"`
}

type ServerConfig struct {
    Host         string        `json:"host" env:"SERVER_HOST" default:"localhost"`
    Port         int           `json:"port" env:"SERVER_PORT" default:"8080"`
    ReadTimeout  time.Duration `json:"read_timeout" env:"SERVER_READ_TIMEOUT" default:"30s"`
    WriteTimeout time.Duration `json:"write_timeout" env:"SERVER_WRITE_TIMEOUT" default:"30s"`
    IdleTimeout  time.Duration `json:"idle_timeout" env:"SERVER_IDLE_TIMEOUT" default:"60s"`
}

type DatabaseConfig struct {
    Host         string `json:"host" env:"DB_HOST" default:"localhost"`
    Port         int    `json:"port" env:"DB_PORT" default:"5432"`
    User         string `json:"user" env:"DB_USER" default:"postgres"`
    Password     string `json:"-" env:"DB_PASSWORD"`
    Name         string `json:"name" env:"DB_NAME" default:"myapp"`
    SSLMode      string `json:"ssl_mode" env:"DB_SSL_MODE" default:"disable"`
    MaxOpenConns int    `json:"max_open_conns" env:"DB_MAX_OPEN_CONNS" default:"25"`
    MaxIdleConns int    `json:"max_idle_conns" env:"DB_MAX_IDLE_CONNS" default:"5"`
}

func (db DatabaseConfig) DSN() string {
    return fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=%s",
        db.Host, db.Port, db.User, db.Password, db.Name, db.SSLMode)
}

type RedisConfig struct {
    URL      string `json:"url" env:"REDIS_URL" default:"redis://localhost:6379"`
    Password string `json:"-" env:"REDIS_PASSWORD"`
    Database int    `json:"database" env:"REDIS_DB" default:"0"`
}

type JWTConfig struct {
    Secret         string        `json:"-" env:"JWT_SECRET"`
    AccessExpiry   time.Duration `json:"access_expiry" env:"JWT_ACCESS_EXPIRY" default:"15m"`
    RefreshExpiry  time.Duration `json:"refresh_expiry" env:"JWT_REFRESH_EXPIRY" default:"7d"`
    Issuer         string        `json:"issuer" env:"JWT_ISSUER" default:"myapp"`
}

type EmailConfig struct {
    Host     string `json:"host" env:"EMAIL_HOST"`
    Port     int    `json:"port" env:"EMAIL_PORT" default:"587"`
    Username string `json:"username" env:"EMAIL_USERNAME"`
    Password string `json:"-" env:"EMAIL_PASSWORD"`
    From     string `json:"from" env:"EMAIL_FROM"`
}

// Embedded структуры для общих полей
type Timestamped struct {
    CreatedAt time.Time `json:"created_at" db:"created_at"`
    UpdatedAt time.Time `json:"updated_at" db:"updated_at"`
}

type SoftDeleted struct {
    DeletedAt *time.Time `json:"deleted_at,omitempty" db:"deleted_at"`
}

// Пример использования embedded структур
type Post struct {
    ID       int64  `json:"id" db:"id"`
    Title    string `json:"title" db:"title"`
    Content  string `json:"content" db:"content"`
    UserID   int64  `json:"user_id" db:"user_id"`
    Status   string `json:"status" db:"status"`
    
    Timestamped  // Встраиваем поля времени
    SoftDeleted  // Встраиваем soft delete
}

func (p *Post) IsPublished() bool {
    return p.Status == "published" && p.DeletedAt == nil
}

// Демонстрация работы со структурами
func demonstrateStructs() {
    // Создание пользователя
    user := &User{
        BaseModel: BaseModel{
            ID:        1,
            CreatedAt: time.Now(),
            UpdatedAt: time.Now(),
        },
        Email:    "john@example.com",
        Name:     "John Doe",
        IsActive: true,
        Role:     RoleUser,
        Profile: &Profile{
            Bio:      "Backend developer",
            Location: "San Francisco",
        },
    }
    
    // Сериализация в JSON
    jsonData, _ := json.MarshalIndent(user, "", "  ")
    fmt.Printf("User JSON:\n%s\n", jsonData)
    
    // Использование методов
    fmt.Printf("Display name: %s\n", user.GetDisplayName())
    fmt.Printf("Can moderate: %v\n", user.CanModerate())
    fmt.Printf("Is valid: %v\n", user.IsValid())
    
    // Конвертация в DTO
    response := user.ToResponse()
    responseJSON, _ := json.MarshalIndent(response, "", "  ")
    fmt.Printf("Response JSON:\n%s\n", responseJSON)
}
```

**🎯 Практическая польза:**
- **Типизированные данные** - предотвращение ошибок на этапе компиляции
- **JSON теги** - автоматическая сериализация/десериализация для API
- **Database теги** - маппинг на таблицы БД
- **Validation теги** - автоматическая валидация входящих данных
- **Методы** - инкапсуляция бизнес-логики

#### 🎯 Указатели и управление памятью

### Память в Go: Stack vs Heap

```
Memory Management in Go
┌─────────────────────────────────────────────────────────────┐
│                        Stack                                │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │   func1()   │  │   func2()   │  │   func3()   │          │
│  │             │  │             │  │             │          │
│  │ local vars  │  │ local vars  │  │ local vars  │          │
│  │ parameters  │  │ parameters  │  │ parameters  │          │
│  │             │  │             │  │             │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│         ▲               ▲               ▲                  │
│      Current         Previous        Stack                 │
│      Frame            Frame          Growth                │
├─────────────────────────────────────────────────────────────┤
│                         Heap                                │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │   *User     │  │   []byte    │  │   map[...]  │          │
│  │             │  │             │  │             │          │
│  │    ID: 1    │  │ Large Data  │  │ Key: Value  │          │
│  │ Email: ...  │  │   Buffer    │  │    ...      │          │
│  │             │  │             │  │             │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│         ▲               ▲               ▲                  │
│    Garbage           Escape           Dynamic              │
│    Collected        Analysis         Allocation            │
└─────────────────────────────────────────────────────────────┘
```

```go
package main

import (
    "fmt"
    "runtime"
    "unsafe"
)

// Демонстрация указателей в контексте backend разработки
func demonstratePointers() {
    // 1. Создание значения и указателя
    user := User{
        ID:    1,
        Email: "john@example.com",
        Name:  "John Doe",
    }
    
    userPtr := &user // Получаем указатель на user
    
    fmt.Printf("Value: %+v\n", user)
    fmt.Printf("Pointer: %p\n", userPtr)
    fmt.Printf("Value via pointer: %+v\n", *userPtr)
    
    // 2. Передача по значению vs по указателю
    updateUserByValue(user)
    fmt.Printf("After update by value: %s\n", user.Name) // Не изменился
    
    updateUserByPointer(&user)
    fmt.Printf("After update by pointer: %s\n", user.Name) // Изменился
    
    // 3. Нулевые указатели (nil)
    var nilUser *User
    fmt.Printf("Nil pointer: %v\n", nilUser)
    
    // Безопасная проверка на nil
    if nilUser != nil {
        fmt.Printf("User name: %s\n", nilUser.Name)
    } else {
        fmt.Println("User is nil")
    }
}

func updateUserByValue(u User) {
    u.Name = "Updated Name" // Изменяется только копия
}

func updateUserByPointer(u *User) {
    u.Name = "Updated Name" // Изменяется оригинал
}

// Практическое применение указателей в API
type UpdateUserRequest struct {
    Email *string `json:"email,omitempty"`     // nil = не обновлять
    Name  *string `json:"name,omitempty"`      // nil = не обновлять  
    IsActive *bool `json:"is_active,omitempty"` // nil = не обновлять
}

func updateUser(user *User, req UpdateUserRequest) {
    // Обновляем только переданные поля
    if req.Email != nil {
        user.Email = *req.Email
    }
    
    if req.Name != nil {
        user.Name = *req.Name
    }
    
    if req.IsActive != nil {
        user.IsActive = *req.IsActive
    }
}

// Вспомогательные функции для создания указателей
func StringPtr(s string) *string {
    return &s
}

func BoolPtr(b bool) *bool {
    return &b
}

func IntPtr(i int) *int {
    return &i
}

// Пример использования в JSON API
func demonstratePartialUpdate() {
    user := &User{
        ID:       1,
        Email:    "old@example.com",
        Name:     "Old Name",
        IsActive: false,
    }
    
    fmt.Printf("Before update: %+v\n", user)
    
    // Частичное обновление - обновляем только email и статус
    updateReq := UpdateUserRequest{
        Email:    StringPtr("new@example.com"),
        IsActive: BoolPtr(true),
        // Name не передаем - останется старое значение
    }
    
    updateUser(user, updateReq)
    fmt.Printf("After partial update: %+v\n", user)
}

// Эффективность памяти: указатели vs значения
type LargeStruct struct {
    Data [1000]int // 8KB данных
    ID   int64
    Name string
}

func processLargeStructByValue(ls LargeStruct) {
    // Копируется 8KB+ данных при каждом вызове
    fmt.Printf("Processing large struct with ID: %d\n", ls.ID)
}

func processLargeStructByPointer(ls *LargeStruct) {
    // Передается только 8 байт (размер указателя)
    fmt.Printf("Processing large struct with ID: %d\n", ls.ID)
}

func demonstrateMemoryEfficiency() {
    large := LargeStruct{ID: 1, Name: "Test"}
    
    var m1, m2 runtime.MemStats
    
    // Измеряем память до операций
    runtime.GC()
    runtime.ReadMemStats(&m1)
    
    // 1000 вызовов по значению (много копирований)
    for i := 0; i < 1000; i++ {
        processLargeStructByValue(large)
    }
    
    runtime.GC()
    runtime.ReadMemStats(&m2)
    fmt.Printf("Memory used (by value): %d bytes\n", m2.TotalAlloc-m1.TotalAlloc)
    
    // Сброс счетчиков
    runtime.ReadMemStats(&m1)
    
    // 1000 вызовов по указателю (только указатели)
    for i := 0; i < 1000; i++ {
        processLargeStructByPointer(&large)
    }
    
    runtime.GC()
    runtime.ReadMemStats(&m2)
    fmt.Printf("Memory used (by pointer): %d bytes\n", m2.TotalAlloc-m1.TotalAlloc)
}

// Связанные списки с указателями
type ListNode struct {
    Value int
    Next  *ListNode
}

type LinkedList struct {
    Head *ListNode
    Size int
}

func (ll *LinkedList) Add(value int) {
    newNode := &ListNode{
        Value: value,
        Next:  ll.Head,
    }
    ll.Head = newNode
    ll.Size++
}

func (ll *LinkedList) Print() {
    current := ll.Head
    for current != nil {
        fmt.Printf("%d -> ", current.Value)
        current = current.Next
    }
    fmt.Println("nil")
}

// Escape analysis - когда переменные "убегают" в heap
func createUser() *User {
    // user "убегает" в heap, потому что возвращается указатель
    user := User{
        ID:    1,
        Email: "escape@example.com",
        Name:  "Escaped User",
    }
    return &user // Возвращаем указатель на локальную переменную
}

func demonstrateEscapeAnalysis() {
    // user создается в heap, не в stack
    user := createUser()
    fmt.Printf("Escaped user: %+v\n", user)
    
    // Можем проверить размер указателя
    fmt.Printf("Pointer size: %d bytes\n", unsafe.Sizeof(user))
    fmt.Printf("User struct size: %d bytes\n", unsafe.Sizeof(*user))
}

// Пулы объектов для уменьшения нагрузки на GC
type UserPool struct {
    pool chan *User
}

func NewUserPool(size int) *UserPool {
    return &UserPool{
        pool: make(chan *User, size),
    }
}

func (p *UserPool) Get() *User {
    select {
    case user := <-p.pool:
        return user
    default:
        return &User{} // Создаем новый, если пул пуст
    }
}

func (p *UserPool) Put(user *User) {
    // Очищаем пользователя перед возвратом в пул
    *user = User{}
    
    select {
    case p.pool <- user:
        // Успешно вернули в пул
    default:
        // Пул полон, позволяем GC очистить объект
    }
}
```

**🎯 Практическая польза указателей:**
- **Экономия памяти** - избегаем копирования больших структур
- **Модификация данных** - изменяем оригинальные объекты
- **Частичные обновления** - обновляем только нужные поля в API
- **Производительность** - меньше работы для GC

**💡 Правила использования:**
- Используйте указатели для больших структур (>100 байт)
- Всегда проверяйте на nil перед использованием
- Используйте указатели для optional полей в API
- Возвращайте указатели из функций для изменяемых объектов

#### 🔌 Интерфейсы - гибкая архитектура

### Архитектура интерфейсов в Go

```
Interface Architecture Pattern
┌─────────────────────────────────────────────────────────────┐
│                    Interface Layer                          │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │   Reader    │  │   Writer    │  │   Closer    │          │
│  │ interface   │  │ interface   │  │ interface   │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│         ▲               ▲               ▲                  │
│         │               │               │                  │
├─────────┼───────────────┼───────────────┼─────────────────┤
│         │               │               │                  │
│    Implementation   Implementation   Implementation        │
│         │               │               │                  │
│         ▼               ▼               ▼                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │    File     │  │   Network   │  │   Database  │          │
│  │             │  │   Socket    │  │ Connection  │          │
│  │ Read()      │  │ Write()     │  │ Close()     │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────────────────────┘

Repository Pattern with Interfaces
┌─────────────────────────────────────────────────────────────┐
│                  Business Logic                             │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐ │
│  │           UserService                                   │ │
│  │  depends on → UserRepository interface                 │ │
│  └─────────────────────────────────────────────────────────┘ │
│                           │                                 │
│                           ▼                                 │
├─────────────────────────────────────────────────────────────┤
│                  Interface Layer                            │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐ │
│  │              UserRepository                             │ │
│  │  Create(user *User) error                              │ │
│  │  GetByID(id int64) (*User, error)                      │ │
│  │  Update(user *User) error                              │ │
│  │  Delete(id int64) error                                │ │
│  └─────────────────────────────────────────────────────────┘ │
│                           ▲                                 │
│                           │                                 │
├─────────────────────────────────────────────────────────────┤
│                 Implementation Layer                        │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ PostgresRepo│  │  RedisRepo  │  │  MockRepo   │          │
│  │             │  │             │  │             │          │
│  │ implements  │  │ implements  │  │ implements  │          │
│  │UserRepository│  │UserRepository│  │UserRepository│          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────────────────────┘
```

```go
package main

import (
    "context"
    "errors"
    "fmt"
    "io"
    "log"
    "strings"
    "time"
)

// Базовые интерфейсы Go (встроенные)
func demonstrateBuiltinInterfaces() {
    // io.Reader - читает данные
    var reader io.Reader = strings.NewReader("Hello, Go interfaces!")
    
    data := make([]byte, 10)
    n, err := reader.Read(data)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Read %d bytes: %s\n", n, string(data[:n]))
    
    // io.Writer - записывает данные
    var writer io.Writer = &strings.Builder{}
    writer.Write([]byte("Writing to interface"))
    
    // Type assertion для получения конкретного типа
    if builder, ok := writer.(*strings.Builder); ok {
        fmt.Printf("Written: %s\n", builder.String())
    }
}

// Определение интерфейсов для backend архитектуры

// UserRepository - интерфейс для работы с пользователями
type UserRepository interface {
    Create(ctx context.Context, user *User) error
    GetByID(ctx context.Context, id int64) (*User, error)
    GetByEmail(ctx context.Context, email string) (*User, error)
    Update(ctx context.Context, user *User) error
    Delete(ctx context.Context, id int64) error
    List(ctx context.Context, offset, limit int) ([]*User, error)
}

// CacheRepository - интерфейс для кэширования
type CacheRepository interface {
    Set(ctx context.Context, key string, value interface{}, ttl time.Duration) error
    Get(ctx context.Context, key string, dest interface{}) error
    Delete(ctx context.Context, key string) error
    Exists(ctx context.Context, key string) (bool, error)
}

// EmailService - интерфейс для отправки email
type EmailService interface {
    SendWelcome(ctx context.Context, user *User) error
    SendPasswordReset(ctx context.Context, user *User, token string) error
    SendNotification(ctx context.Context, email, subject, body string) error
}

// Logger - интерфейс для логирования
type Logger interface {
    Debug(msg string, fields ...Field)
    Info(msg string, fields ...Field)
    Warn(msg string, fields ...Field)
    Error(msg string, fields ...Field)
    Fatal(msg string, fields ...Field)
}

type Field struct {
    Key   string
    Value interface{}
}

// Реализация PostgreSQL репозитория
type PostgresUserRepository struct {
    db Database // Еще один интерфейс!
}

type Database interface {
    Query(ctx context.Context, query string, args ...interface{}) (Rows, error)
    QueryRow(ctx context.Context, query string, args ...interface{}) Row
    Exec(ctx context.Context, query string, args ...interface{}) (Result, error)
    Begin(ctx context.Context) (Transaction, error)
}

type Rows interface {
    Next() bool
    Scan(dest ...interface{}) error
    Close() error
}

type Row interface {
    Scan(dest ...interface{}) error
}

type Result interface {
    LastInsertId() (int64, error)
    RowsAffected() (int64, error)
}

type Transaction interface {
    Database
    Commit() error
    Rollback() error
}

func NewPostgresUserRepository(db Database) UserRepository {
    return &PostgresUserRepository{db: db}
}

func (r *PostgresUserRepository) Create(ctx context.Context, user *User) error {
    query := `
        INSERT INTO users (email, name, password_hash, is_active, role, created_at, updated_at)
        VALUES ($1, $2, $3, $4, $5, $6, $7)
        RETURNING id`
    
    now := time.Now()
    err := r.db.QueryRow(ctx, query, 
        user.Email, user.Name, user.Password, user.IsActive, user.Role, now, now,
    ).Scan(&user.ID)
    
    if err != nil {
        return fmt.Errorf("failed to create user: %w", err)
    }
    
    user.CreatedAt = now
    user.UpdatedAt = now
    return nil
}

func (r *PostgresUserRepository) GetByID(ctx context.Context, id int64) (*User, error) {
    user := &User{}
    query := `
        SELECT id, email, name, is_active, role, created_at, updated_at
        FROM users WHERE id = $1 AND deleted_at IS NULL`
    
    err := r.db.QueryRow(ctx, query, id).Scan(
        &user.ID, &user.Email, &user.Name, &user.IsActive, 
        &user.Role, &user.CreatedAt, &user.UpdatedAt,
    )
    
    if err != nil {
        return nil, fmt.Errorf("failed to get user by id %d: %w", id, err)
    }
    
    return user, nil
}

func (r *PostgresUserRepository) GetByEmail(ctx context.Context, email string) (*User, error) {
    user := &User{}
    query := `
        SELECT id, email, name, is_active, role, created_at, updated_at
        FROM users WHERE email = $1 AND deleted_at IS NULL`
    
    err := r.db.QueryRow(ctx, query, email).Scan(
        &user.ID, &user.Email, &user.Name, &user.IsActive, 
        &user.Role, &user.CreatedAt, &user.UpdatedAt,
    )
    
    if err != nil {
        return nil, fmt.Errorf("failed to get user by email %s: %w", email, err)
    }
    
    return user, nil
}

func (r *PostgresUserRepository) Update(ctx context.Context, user *User) error {
    user.UpdatedAt = time.Now()
    query := `
        UPDATE users 
        SET email = $2, name = $3, is_active = $4, role = $5, updated_at = $6
        WHERE id = $1 AND deleted_at IS NULL`
    
    result, err := r.db.Exec(ctx, query, 
        user.ID, user.Email, user.Name, user.IsActive, user.Role, user.UpdatedAt,
    )
    
    if err != nil {
        return fmt.Errorf("failed to update user: %w", err)
    }
    
    rowsAffected, _ := result.RowsAffected()
    if rowsAffected == 0 {
        return errors.New("user not found or already deleted")
    }
    
    return nil
}

func (r *PostgresUserRepository) Delete(ctx context.Context, id int64) error {
    // Soft delete
    query := `UPDATE users SET deleted_at = $2 WHERE id = $1 AND deleted_at IS NULL`
    
    result, err := r.db.Exec(ctx, query, id, time.Now())
    if err != nil {
        return fmt.Errorf("failed to delete user: %w", err)
    }
    
    rowsAffected, _ := result.RowsAffected()
    if rowsAffected == 0 {
        return errors.New("user not found or already deleted")
    }
    
    return nil
}

func (r *PostgresUserRepository) List(ctx context.Context, offset, limit int) ([]*User, error) {
    query := `
        SELECT id, email, name, is_active, role, created_at, updated_at
        FROM users 
        WHERE deleted_at IS NULL
        ORDER BY created_at DESC
        LIMIT $1 OFFSET $2`
    
    rows, err := r.db.Query(ctx, query, limit, offset)
    if err != nil {
        return nil, fmt.Errorf("failed to list users: %w", err)
    }
    defer rows.Close()
    
    var users []*User
    for rows.Next() {
        user := &User{}
        err := rows.Scan(
            &user.ID, &user.Email, &user.Name, &user.IsActive,
            &user.Role, &user.CreatedAt, &user.UpdatedAt,
        )
        if err != nil {
            return nil, fmt.Errorf("failed to scan user: %w", err)
        }
        users = append(users, user)
    }
    
    return users, nil
}

// Redis реализация для кэширования
type RedisUserRepository struct {
    client RedisClient
    logger Logger
}

type RedisClient interface {
    Set(ctx context.Context, key string, value interface{}, expiration time.Duration) error
    Get(ctx context.Context, key string) (string, error)
    Del(ctx context.Context, keys ...string) error
    Exists(ctx context.Context, keys ...string) (int64, error)
}

func NewRedisUserRepository(client RedisClient, logger Logger) UserRepository {
    return &RedisUserRepository{
        client: client,
        logger: logger,
    }
}

func (r *RedisUserRepository) Create(ctx context.Context, user *User) error {
    return errors.New("redis repository does not support create operations")
}

func (r *RedisUserRepository) GetByID(ctx context.Context, id int64) (*User, error) {
    key := fmt.Sprintf("user:%d", id)
    
    data, err := r.client.Get(ctx, key)
    if err != nil {
        return nil, fmt.Errorf("failed to get user from redis: %w", err)
    }
    
    var user User
    if err := json.Unmarshal([]byte(data), &user); err != nil {
        return nil, fmt.Errorf("failed to unmarshal user data: %w", err)
    }
    
    r.logger.Debug("User retrieved from cache", Field{Key: "user_id", Value: id})
    return &user, nil
}

// Mock реализация для тестирования
type MockUserRepository struct {
    users map[int64]*User
    nextID int64
}

func NewMockUserRepository() UserRepository {
    return &MockUserRepository{
        users: make(map[int64]*User),
        nextID: 1,
    }
}

func (m *MockUserRepository) Create(ctx context.Context, user *User) error {
    user.ID = m.nextID
    m.nextID++
    user.CreatedAt = time.Now()
    user.UpdatedAt = user.CreatedAt
    
    m.users[user.ID] = user
    return nil
}

func (m *MockUserRepository) GetByID(ctx context.Context, id int64) (*User, error) {
    user, exists := m.users[id]
    if !exists {
        return nil, errors.New("user not found")
    }
    
    // Возвращаем копию, чтобы избежать изменений
    userCopy := *user
    return &userCopy, nil
}

func (m *MockUserRepository) GetByEmail(ctx context.Context, email string) (*User, error) {
    for _, user := range m.users {
        if user.Email == email {
            userCopy := *user
            return &userCopy, nil
        }
    }
    return nil, errors.New("user not found")
}

func (m *MockUserRepository) Update(ctx context.Context, user *User) error {
    if _, exists := m.users[user.ID]; !exists {
        return errors.New("user not found")
    }
    
    user.UpdatedAt = time.Now()
    m.users[user.ID] = user
    return nil
}

func (m *MockUserRepository) Delete(ctx context.Context, id int64) error {
    if _, exists := m.users[id]; !exists {
        return errors.New("user not found")
    }
    
    delete(m.users, id)
    return nil
}

func (m *MockUserRepository) List(ctx context.Context, offset, limit int) ([]*User, error) {
    var users []*User
    i := 0
    
    for _, user := range m.users {
        if i >= offset && len(users) < limit {
            userCopy := *user
            users = append(users, &userCopy)
        }
        i++
    }
    
    return users, nil
}

// Композитный репозиторий (кэш + база данных)
type CachedUserRepository struct {
    primary UserRepository  // Основной репозиторий (БД)
    cache   UserRepository  // Кэш репозиторий
    logger  Logger
}

func NewCachedUserRepository(primary, cache UserRepository, logger Logger) UserRepository {
    return &CachedUserRepository{
        primary: primary,
        cache:   cache,
        logger:  logger,
    }
}

func (c *CachedUserRepository) GetByID(ctx context.Context, id int64) (*User, error) {
    // Сначала проверяем кэш
    user, err := c.cache.GetByID(ctx, id)
    if err == nil {
        c.logger.Debug("User found in cache", Field{Key: "user_id", Value: id})
        return user, nil
    }
    
    // Если в кэше нет, идем в основную БД
    user, err = c.primary.GetByID(ctx, id)
    if err != nil {
        return nil, err
    }
    
    // Сохраняем в кэш для следующих запросов
    if cacheErr := c.cache.Create(ctx, user); cacheErr != nil {
        c.logger.Warn("Failed to cache user", 
            Field{Key: "user_id", Value: id},
            Field{Key: "error", Value: cacheErr},
        )
    }
    
    c.logger.Debug("User loaded from primary storage", Field{Key: "user_id", Value: id})
    return user, nil
}

// Для остальных методов используем primary репозиторий
func (c *CachedUserRepository) Create(ctx context.Context, user *User) error {
    return c.primary.Create(ctx, user)
}

func (c *CachedUserRepository) GetByEmail(ctx context.Context, email string) (*User, error) {
    return c.primary.GetByEmail(ctx, email)
}

func (c *CachedUserRepository) Update(ctx context.Context, user *User) error {
    // Обновляем в основной БД
    if err := c.primary.Update(ctx, user); err != nil {
        return err
    }
    
    // Инвалидируем кэш
    if err := c.cache.Delete(ctx, user.ID); err != nil {
        c.logger.Warn("Failed to invalidate cache", 
            Field{Key: "user_id", Value: user.ID},
            Field{Key: "error", Value: err},
        )
    }
    
    return nil
}

func (c *CachedUserRepository) Delete(ctx context.Context, id int64) error {
    // Удаляем из основной БД
    if err := c.primary.Delete(ctx, id); err != nil {
        return err
    }
    
    // Удаляем из кэша
    if err := c.cache.Delete(ctx, id); err != nil {
        c.logger.Warn("Failed to delete from cache", 
            Field{Key: "user_id", Value: id},
            Field{Key: "error", Value: err},
        )
    }
    
    return nil
}

func (c *CachedUserRepository) List(ctx context.Context, offset, limit int) ([]*User, error) {
    return c.primary.List(ctx, offset, limit)
}

// Пустой интерфейс для динамических данных
func handleDynamicData(data interface{}) {
    switch v := data.(type) {
    case string:
        fmt.Printf("String data: %s\n", v)
    case int:
        fmt.Printf("Integer data: %d\n", v)
    case *User:
        fmt.Printf("User data: %s (%s)\n", v.Name, v.Email)
    case []interface{}:
        fmt.Printf("Array with %d elements\n", len(v))
    default:
        fmt.Printf("Unknown data type: %T\n", v)
    }
}

// Демонстрация работы с интерфейсами
func demonstrateInterfaces() {
    ctx := context.Background()
    
    // Создаем разные реализации
    mockRepo := NewMockUserRepository()
    
    // Используем интерфейс - не важно, какая реализация
    user := &User{
        Email: "test@example.com",
        Name:  "Test User",
        IsActive: true,
        Role: RoleUser,
    }
    
    // Создаем пользователя
    if err := mockRepo.Create(ctx, user); err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("Created user with ID: %d\n", user.ID)
    
    // Получаем пользователя
    retrievedUser, err := mockRepo.GetByID(ctx, user.ID)
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("Retrieved user: %+v\n", retrievedUser)
    
    // Можем легко менять реализацию
    var repo UserRepository = mockRepo // Тип interface{}
    
    // Позже можем заменить на PostgreSQL или Redis
    // repo = NewPostgresUserRepository(db)
    // repo = NewRedisUserRepository(redisClient, logger)
    
    users, err := repo.List(ctx, 0, 10)
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("Found %d users\n", len(users))
}
```

**🎯 Практическая польза интерфейсов:**
- **Тестируемость** - легко заменять реальные компоненты на mock'и
- **Гибкость** - можно менять реализации без изменения кода
- **Разделение ответственности** - четкие границы между слоями
- **Композиция** - комбинирование разных реализаций (кэш + БД)

**💡 Принципы проектирования:**
- Интерфейсы определяет потребитель, не производитель
- Делайте интерфейсы минимальными и специфичными
- Принцип "Program to interfaces, not implementations"
- Используйте композицию для комбинирования функциональности

---

### Неделя 3: Параллелизм и тестирование

#### 🚀 Горутины - легковесные потоки

### Модель параллелизма в Go

```
Go Concurrency Model (M:N Threading)
┌─────────────────────────────────────────────────────────────┐
│                    OS Threads (M)                           │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │   Thread    │  │   Thread    │  │   Thread    │          │
│  │     1       │  │     2       │  │     3       │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│         ▲               ▲               ▲                  │
│         │               │               │                  │
├─────────┼───────────────┼───────────────┼─────────────────┤
│         │      Go Scheduler (P)         │                  │
├─────────┼───────────────┼───────────────┼─────────────────┤
│         ▼               ▼               ▼                  │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                Goroutines (G)                          │ │
│  │  ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐     │ │
│  │  │G1 │ │G2 │ │G3 │ │G4 │ │G5 │ │G6 │ │G7 │ │G8 │ ... │ │
│  │  └───┘ └───┘ └───┘ └───┘ └───┘ └───┘ └───┘ └───┘     │ │
│  └─────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│               Goroutine Lifecycle                           │
├─────────────────────────────────────────────────────────────┤
│  Created → Runnable → Running → Blocked → Runnable → Dead   │
│     │         │          │         │         │        │   │
│     ▼         ▼          ▼         ▼         ▼        ▼   │
│   go()    Scheduled   Executing  I/O Wait  Ready    Exit  │
└─────────────────────────────────────────────────────────────┘

HTTP Server with Goroutines
┌─────────────────────────────────────────────────────────────┐
│                    HTTP Server                              │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────────────────────────────┐  │
│  │   Accept    │    │           Handler Pool             │  │
│  │   Loop      │────┤                                     │  │
│  │             │    │  ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐     │  │
│  └─────────────┘    │  │ H1│ │ H2│ │ H3│ │ H4│ │ H5│ ... │  │
│         │           │  └───┘ └───┘ └───┘ └───┘ └───┘     │  │
│         ▼           └─────────────────────────────────────────┘  │
│  ┌─────────────┐              Each handler is a goroutine    │
│  │ New Request │                                              │
│  │      │      │                                              │
│  │      ▼      │                                              │
│  │go handleReq │                                              │
│  └─────────────┘                                              │
└─────────────────────────────────────────────────────────────┘
```

```go
package main

import (
    "context"
    "fmt"
    "log"
    "math/rand"
    "net/http"
    "runtime"
    "sync"
    "time"
)

// Базовое использование горутин
func demonstrateBasicGoroutines() {
    fmt.Printf("Starting with %d goroutines\n", runtime.NumGoroutine())
    
    // Запуск горутин
    for i := 1; i <= 5; i++ {
        go func(id int) {
            fmt.Printf("Goroutine %d starting\n", id)
            time.Sleep(time.Duration(rand.Intn(1000)) * time.Millisecond)
            fmt.Printf("Goroutine %d finished\n", id)
        }(i) // Передаем i как параметр, чтобы избежать closure проблем
    }
    
    // Ждем немного, чтобы горутины запустились
    time.Sleep(2 * time.Second)
    fmt.Printf("Ending with %d goroutines\n", runtime.NumGoroutine())
}

// WaitGroup для ожидания завершения горутин
func demonstrateWaitGroup() {
    var wg sync.WaitGroup
    
    tasks := []string{"task1", "task2", "task3", "task4", "task5"}
    
    for _, task := range tasks {
        wg.Add(1) // Увеличиваем счетчик
        
        go func(taskName string) {
            defer wg.Done() // Уменьшаем счетчик при завершении
            
            fmt.Printf("Processing %s\n", taskName)
            
            // Имитация работы
            time.Sleep(time.Duration(rand.Intn(1000)) * time.Millisecond)
            
            fmt.Printf("Completed %s\n", taskName)
        }(task)
    }
    
    fmt.Println("Waiting for all tasks to complete...")
    wg.Wait() // Ждем, пока счетчик не станет 0
    fmt.Println("All tasks completed!")
}

// Worker Pool Pattern для ограничения количества одновременных задач
type Job struct {
    ID   int
    Data string
}

type Result struct {
    Job    Job
    Output string
    Error  error
}

func worker(id int, jobs <-chan Job, results chan<- Result) {
    for job := range jobs {
        fmt.Printf("Worker %d processing job %d\n", id, job.ID)
        
        // Имитация работы
        time.Sleep(time.Duration(rand.Intn(500)) * time.Millisecond)
        
        result := Result{
            Job:    job,
            Output: fmt.Sprintf("Processed by worker %d: %s", id, job.Data),
        }
        
        // Имитация ошибки (10% вероятность)
        if rand.Float32() < 0.1 {
            result.Error = fmt.Errorf("worker %d failed to process job %d", id, job.ID)
        }
        
        results <- result
    }
}

func demonstrateWorkerPool() {
    const numWorkers = 3
    const numJobs = 10
    
    // Создаем каналы
    jobs := make(chan Job, numJobs)
    results := make(chan Result, numJobs)
    
    // Запускаем workers
    for w := 1; w <= numWorkers; w++ {
        go worker(w, jobs, results)
    }
    
    // Отправляем задачи
    for j := 1; j <= numJobs; j++ {
        jobs <- Job{
            ID:   j,
            Data: fmt.Sprintf("data-%d", j),
        }
    }
    close(jobs) // Закрываем канал задач
    
    // Собираем результаты
    for r := 1; r <= numJobs; r++ {
        result := <-results
        if result.Error != nil {
            fmt.Printf("❌ Job %d failed: %v\n", result.Job.ID, result.Error)
        } else {
            fmt.Printf("✅ Job %d: %s\n", result.Job.ID, result.Output)
        }
    }
}

// HTTP сервер с горутинами
func handleUserRequest(w http.ResponseWriter, r *http.Request) {
    // Каждый HTTP запрос обрабатывается в отдельной горутине
    goroutineID := getGoroutineID()
    fmt.Printf("Handling request in goroutine %d\n", goroutineID)
    
    // Имитация обработки
    time.Sleep(100 * time.Millisecond)
    
    fmt.Fprintf(w, "Hello from goroutine %d at %s", goroutineID, time.Now().Format(time.RFC3339))
}

func getGoroutineID() int {
    // Упрощенная функция для демонстрации
    return int(rand.Int31n(10000))
}

// Graceful shutdown с горутинами
func demonstrateHTTPServer() {
    http.HandleFunc("/user", handleUserRequest)
    
    server := &http.Server{
        Addr:    ":8080",
        Handler: nil,
    }
    
    // Запускаем сервер в горутине
    go func() {
        fmt.Println("Server starting on :8080")
        if err := server.ListenAndServe(); err != http.ErrServerClosed {
            log.Fatal("Server failed:", err)
        }
    }()
    
    // Имитируем работу
    time.Sleep(5 * time.Second)
    
    // Graceful shutdown
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    if err := server.Shutdown(ctx); err != nil {
        log.Fatal("Server shutdown failed:", err)
    }
    
    fmt.Println("Server stopped gracefully")
}

// Pipeline pattern с горутинами
func generateNumbers(max int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for i := 1; i <= max; i++ {
            out <- i
            time.Sleep(10 * time.Millisecond) // Имитация медленной генерации
        }
    }()
    return out
}

func squareNumbers(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- n * n
            time.Sleep(5 * time.Millisecond) // Имитация обработки
        }
    }()
    return out
}

func filterEven(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            if n%2 == 0 {
                out <- n
            }
        }
    }()
    return out
}

func demonstratePipeline() {
    // Создаем pipeline: generate → square → filter
    numbers := generateNumbers(10)
    squared := squareNumbers(numbers)
    evenSquares := filterEven(squared)
    
    // Читаем результаты
    fmt.Println("Even squares:")
    for result := range evenSquares {
        fmt.Printf("%d ", result)
    }
    fmt.Println()
}

// Fan-out/Fan-in pattern
func fanOut(in <-chan Job, numWorkers int) []<-chan Result {
    channels := make([]<-chan Result, numWorkers)
    
    for i := 0; i < numWorkers; i++ {
        out := make(chan Result)
        channels[i] = out
        
        go func() {
            defer close(out)
            for job := range in {
                // Имитация обработки
                time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
                
                result := Result{
                    Job:    job,
                    Output: fmt.Sprintf("Processed: %s", job.Data),
                }
                out <- result
            }
        }()
    }
    
    return channels
}

func fanIn(channels ...<-chan Result) <-chan Result {
    out := make(chan Result)
    var wg sync.WaitGroup
    
    // Запускаем горутину для каждого входного канала
    for _, ch := range channels {
        wg.Add(1)
        go func(c <-chan Result) {
            defer wg.Done()
            for result := range c {
                out <- result
            }
        }(ch)
    }
    
    // Закрываем выходной канал, когда все входные закрыты
    go func() {
        wg.Wait()
        close(out)
    }()
    
    return out
}

func demonstrateFanOutFanIn() {
    // Создаем задачи
    jobs := make(chan Job, 10)
    for i := 1; i <= 10; i++ {
        jobs <- Job{ID: i, Data: fmt.Sprintf("task-%d", i)}
    }
    close(jobs)
    
    // Fan-out: распределяем работу между 3 воркерами
    workerChannels := fanOut(jobs, 3)
    
    // Fan-in: собираем результаты от всех воркеров
    results := fanIn(workerChannels...)
    
    // Читаем результаты
    for result := range results {
        fmt.Printf("Result: %s\n", result.Output)
    }
}

// Race condition и как его избежать
type Counter struct {
    mu    sync.Mutex
    value int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value++
}

func (c *Counter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.value
}

func demonstrateRaceCondition() {
    // Небезопасный счетчик
    unsafeCounter := 0
    
    // Безопасный счетчик
    safeCounter := &Counter{}
    
    var wg sync.WaitGroup
    
    // Запускаем 100 горутин, каждая увеличивает счетчики 1000 раз
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for j := 0; j < 1000; j++ {
                unsafeCounter++ // Race condition!
                safeCounter.Increment() // Thread-safe
            }
        }()
    }
    
    wg.Wait()
    
    fmt.Printf("Expected: 100000\n")
    fmt.Printf("Unsafe counter: %d\n", unsafeCounter) // Вероятно меньше 100000
    fmt.Printf("Safe counter: %d\n", safeCounter.Value()) // Точно 100000
}

// Context для отмены горутин
func longRunningTask(ctx context.Context, id int) {
    ticker := time.NewTicker(500 * time.Millisecond)
    defer ticker.Stop()
    
    for {
        select {
        case <-ctx.Done():
            fmt.Printf("Task %d cancelled: %v\n", id, ctx.Err())
            return
        case <-ticker.C:
            fmt.Printf("Task %d working...\n", id)
        }
    }
}

func demonstrateContext() {
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()
    
    // Запускаем несколько долгих задач
    for i := 1; i <= 3; i++ {
        go longRunningTask(ctx, i)
    }
    
    // Ждем завершения по таймауту
    <-ctx.Done()
    fmt.Println("All tasks cancelled")
    
    // Даем время горутинам завершиться
    time.Sleep(100 * time.Millisecond)
}
```

**🎯 Практическая польза горутин:**
- **Обработка HTTP запросов** - каждый запрос в отдельной горутине
- **Параллельная обработка данных** - worker pools для масштабируемости
- **Pipeline обработка** - этапы обработки данных в разных горутинах
- **Background задачи** - периодические задачи, очистка кэша

**💡 Лучшие практики:**
- Всегда используйте WaitGroup или каналы для синхронизации
- Избегайте race conditions с помощью мьютексов или каналов
- Используйте context для отмены и таймаутов
- Ограничивайте количество горутин с помощью worker pools

#### 📡 Каналы - безопасная передача данных

### Типы каналов и их использование

```
Channel Types and Operations
┌─────────────────────────────────────────────────────────────┐
│                    Unbuffered Channel                       │
├─────────────────────────────────────────────────────────────┤
│  Goroutine A                     Goroutine B                │
│  ┌─────────────┐                 ┌─────────────┐            │
│  │   Send      │←─────────────────│   Receive   │            │
│  │  ch <- val  │   Synchronous   │  val := <-ch│            │
│  └─────────────┘    Communication └─────────────┘            │
│         │                                │                  │
│         ▼                                ▼                  │
│    Blocks until                     Blocks until            │
│    receiver ready                   sender ready            │
├─────────────────────────────────────────────────────────────┤
│                     Buffered Channel                        │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐ │
│  │           Buffer (capacity = 3)                        │ │
│  │  ┌─────┐  ┌─────┐  ┌─────┐                              │ │
│  │  │ V1  │  │ V2  │  │ V3  │                              │ │
│  │  └─────┘  └─────┘  └─────┘                              │ │
│  └─────────────────────────────────────────────────────────┘ │
│         ▲                                ▲                  │
│      Send here                      Receive here           │
│   (blocks when full)              (blocks when empty)      │
├─────────────────────────────────────────────────────────────┤
│                   Channel Directions                        │
├─────────────────────────────────────────────────────────────┤
│  chan int        │  Bidirectional (send + receive)         │
│  chan<- int      │  Send-only channel                      │
│  <-chan int      │  Receive-only channel                   │
└─────────────────────────────────────────────────────────────┘
```

```go
package main

import (
    "fmt"
    "log"
    "math/rand"
    "sync"
    "time"
)

// Базовая работа с каналами
func demonstrateBasicChannels() {
    // Unbuffered канал (синхронный)
    ch := make(chan string)
    
    go func() {
        time.Sleep(1 * time.Second)
        ch <- "Hello from goroutine!"
    }()
    
    message := <-ch // Блокируется до получения значения
    fmt.Println("Received:", message)
    
    // Buffered канал (асинхронный)
    bufferedCh := make(chan int, 3)
    
    // Можем отправить 3 значения без блокировки
    bufferedCh <- 1
    bufferedCh <- 2
    bufferedCh <- 3
    
    // Читаем значения
    for i := 0; i < 3; i++ {
        value := <-bufferedCh
        fmt.Printf("Received: %d\n", value)
    }
    
    // Проверка свойств канала
    fmt.Printf("Channel length: %d, capacity: %d\n", len(bufferedCh), cap(bufferedCh))
}

// Направленные каналы для типобезопасности
func producer(out chan<- int) { // send-only
    for i := 1; i <= 5; i++ {
        out <- i
        time.Sleep(100 * time.Millisecond)
    }
    close(out)
}

func consumer(in <-chan int) { // receive-only
    for value := range in {
        fmt.Printf("Consumed: %d\n", value)
    }
}

func demonstrateDirectionalChannels() {
    ch := make(chan int)
    
    go producer(ch)
    consumer(ch)
}

// Закрытие каналов и проверка статуса
func demonstrateChannelClosing() {
    ch := make(chan int, 5)
    
    // Отправляем данные
    go func() {
        for i := 1; i <= 5; i++ {
            ch <- i
        }
        close(ch) // Закрываем канал после отправки всех данных
    }()
    
    // Способ 1: проверка с помощью ok
    for {
        value, ok := <-ch
        if !ok {
            fmt.Println("Channel closed")
            break
        }
        fmt.Printf("Received: %d\n", value)
    }
    
    // Способ 2: использование range (автоматически завершается при закрытии)
    ch2 := make(chan string, 3)
    ch2 <- "first"
    ch2 <- "second"
    ch2 <- "third"
    close(ch2)
    
    for value := range ch2 {
        fmt.Printf("Range received: %s\n", value)
    }
}

// Request-Response pattern с каналами
type Request struct {
    ID       int
    Data     string
    Response chan string // Канал для ответа
}

func requestHandler(requests <-chan Request) {
    for req := range requests {
        // Имитация обработки
        time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
        
        response := fmt.Sprintf("Processed request %d: %s", req.ID, req.Data)
        req.Response <- response
        close(req.Response)
    }
}

func demonstrateRequestResponse() {
    requests := make(chan Request, 10)
    
    // Запускаем обработчик
    go requestHandler(requests)
    
    var wg sync.WaitGroup
    
    // Отправляем несколько запросов
    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            // Создаем канал для ответа
            responseCh := make(chan string)
            
            // Отправляем запрос
            requests <- Request{
                ID:       id,
                Data:     fmt.Sprintf("data-%d", id),
                Response: responseCh,
            }
            
            // Ждем ответ
            response := <-responseCh
            fmt.Printf("Got response: %s\n", response)
        }(i)
    }
    
    wg.Wait()
    close(requests)
}

// Semaphore pattern для ограничения параллелизма
type Semaphore chan struct{}

func NewSemaphore(capacity int) Semaphore {
    return make(Semaphore, capacity)
}

func (s Semaphore) Acquire() {
    s <- struct{}{}
}

func (s Semaphore) Release() {
    <-s
}

func demonstrateSemaphore() {
    // Ограничиваем до 2 одновременных операций
    sem := NewSemaphore(2)
    var wg sync.WaitGroup
    
    for i := 1; i <= 10; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            sem.Acquire()
            defer sem.Release()
            
            fmt.Printf("Task %d started\n", id)
            time.Sleep(1 * time.Second) // Имитация работы
            fmt.Printf("Task %d completed\n", id)
        }(i)
    }
    
    wg.Wait()
}

// Notification pattern
type EventBus struct {
    subscribers map[string][]chan Event
    mu          sync.RWMutex
}

type Event struct {
    Type string
    Data interface{}
}

func NewEventBus() *EventBus {
    return &EventBus{
        subscribers: make(map[string][]chan Event),
    }
}

func (eb *EventBus) Subscribe(eventType string) <-chan Event {
    eb.mu.Lock()
    defer eb.mu.Unlock()
    
    ch := make(chan Event, 10)
    eb.subscribers[eventType] = append(eb.subscribers[eventType], ch)
    return ch
}

func (eb *EventBus) Publish(event Event) {
    eb.mu.RLock()
    defer eb.mu.RUnlock()
    
    for _, ch := range eb.subscribers[event.Type] {
        // Неблокирующая отправка
        select {
        case ch <- event:
        default:
            log.Printf("Subscriber channel full, dropping event: %+v", event)
        }
    }
}

func demonstrateEventBus() {
    bus := NewEventBus()
    
    // Подписчик 1
    userEvents := bus.Subscribe("user")
    go func() {
        for event := range userEvents {
            fmt.Printf("User subscriber got: %+v\n", event)
        }
    }()
    
    // Подписчик 2
    orderEvents := bus.Subscribe("order")
    go func() {
        for event := range orderEvents {
            fmt.Printf("Order subscriber got: %+v\n", event)
        }
    }()
    
    // Публикуем события
    bus.Publish(Event{Type: "user", Data: "User registered"})
    bus.Publish(Event{Type: "order", Data: "Order created"})
    bus.Publish(Event{Type: "user", Data: "User logged in"})
    
    time.Sleep(100 * time.Millisecond)
}

// Circuit Breaker pattern с каналами
type CircuitBreaker struct {
    maxFailures int
    failures    int
    lastFailure time.Time
    timeout     time.Duration
    state       State
    mu          sync.Mutex
}

type State int

const (
    Closed State = iota
    Open
    HalfOpen
)

func NewCircuitBreaker(maxFailures int, timeout time.Duration) *CircuitBreaker {
    return &CircuitBreaker{
        maxFailures: maxFailures,
        timeout:     timeout,
        state:       Closed,
    }
}

func (cb *CircuitBreaker) Call(fn func() error) error {
    cb.mu.Lock()
    defer cb.mu.Unlock()
    
    // Проверяем состояние
    if cb.state == Open {
        if time.Since(cb.lastFailure) > cb.timeout {
            cb.state = HalfOpen
            cb.failures = 0
        } else {
            return fmt.Errorf("circuit breaker is open")
        }
    }
    
    // Выполняем функцию
    err := fn()
    
    if err != nil {
        cb.failures++
        cb.lastFailure = time.Now()
        
        if cb.failures >= cb.maxFailures {
            cb.state = Open
        }
        return err
    }
    
    // Успех - сбрасываем счетчик
    cb.failures = 0
    cb.state = Closed
    return nil
}

func demonstrateCircuitBreaker() {
    cb := NewCircuitBreaker(3, 2*time.Second)
    
    // Функция, которая иногда падает
    unreliableService := func() error {
        if rand.Float32() < 0.7 { // 70% вероятность ошибки
            return fmt.Errorf("service unavailable")
        }
        return nil
    }
    
    // Тестируем circuit breaker
    for i := 1; i <= 10; i++ {
        err := cb.Call(unreliableService)
        if err != nil {
            fmt.Printf("Call %d failed: %v\n", i, err)
        } else {
            fmt.Printf("Call %d succeeded\n", i)
        }
        time.Sleep(500 * time.Millisecond)
    }
}

// Rate Limiter с каналами
type RateLimiter struct {
    tokens chan struct{}
    ticker *time.Ticker
}

func NewRateLimiter(rate int, burst int) *RateLimiter {
    rl := &RateLimiter{
        tokens: make(chan struct{}, burst),
        ticker: time.NewTicker(time.Second / time.Duration(rate)),
    }
    
    // Заполняем начальными токенами
    for i := 0; i < burst; i++ {
        rl.tokens <- struct{}{}
    }
    
    // Добавляем токены по расписанию
    go func() {
        for range rl.ticker.C {
            select {
            case rl.tokens <- struct{}{}:
            default:
                // Bucket полон, пропускаем токен
            }
        }
    }()
    
    return rl
}

func (rl *RateLimiter) Allow() bool {
    select {
    case <-rl.tokens:
        return true
    default:
        return false
    }
}

func (rl *RateLimiter) Wait() {
    <-rl.tokens
}

func (rl *RateLimiter) Stop() {
    rl.ticker.Stop()
}

func demonstrateRateLimiter() {
    // 5 запросов в секунду, burst до 10
    rl := NewRateLimiter(5, 10)
    defer rl.Stop()
    
    // Быстро отправляем 15 запросов
    for i := 1; i <= 15; i++ {
        if rl.Allow() {
            fmt.Printf("Request %d: ✅ Allowed\n", i)
        } else {
            fmt.Printf("Request %d: ❌ Rate limited\n", i)
        }
    }
    
    // Ждем и пробуем еще раз
    fmt.Println("Waiting 2 seconds...")
    time.Sleep(2 * time.Second)
    
    for i := 16; i <= 20; i++ {
        if rl.Allow() {
            fmt.Printf("Request %d: ✅ Allowed\n", i)
        } else {
            fmt.Printf("Request %d: ❌ Rate limited\n", i)
        }
    }
}

// Timeout pattern с каналами
func makeRequestWithTimeout(url string, timeout time.Duration) (string, error) {
    result := make(chan string, 1)
    errors := make(chan error, 1)
    
    // Запускаем запрос в горутине
    go func() {
        // Имитация HTTP запроса
        time.Sleep(time.Duration(rand.Intn(2000)) * time.Millisecond)
        
        if rand.Float32() < 0.8 { // 80% успеха
            result <- fmt.Sprintf("Response from %s", url)
        } else {
            errors <- fmt.Errorf("request failed")
        }
    }()
    
    // Ждем либо результат, либо таймаут
    select {
    case res := <-result:
        return res, nil
    case err := <-errors:
        return "", err
    case <-time.After(timeout):
        return "", fmt.Errorf("request timed out after %v", timeout)
    }
}

func demonstrateTimeout() {
    urls := []string{
        "http://api1.example.com",
        "http://api2.example.com",
        "http://api3.example.com",
    }
    
    for _, url := range urls {
        response, err := makeRequestWithTimeout(url, 1*time.Second)
        if err != nil {
            fmt.Printf("❌ %s: %v\n", url, err)
        } else {
            fmt.Printf("✅ %s: %s\n", url, response)
        }
    }
}
```

**🎯 Практическая польза каналов:**
- **Безопасная коммуникация** - избежание race conditions
- **Структурирование параллелизма** - четкие паттерны взаимодействия
- **Контроль потока** - rate limiting, circuit breaker, семафоры
- **Event-driven архитектура** - pub/sub паттерны

**💡 Принципы использования:**
- "Don't communicate by sharing memory; share memory by communicating"
- Закрывайте каналы только на стороне отправителя
- Используйте направленные каналы для типобезопасности
- Буферизованные каналы для производительности, небуферизованные для синхронизации

---

## 🌐 Этап 2: Backend-специфичные технологии (3-4 недели)

### Неделя 4: HTTP серверы и API

#### 🌍 HTTP сервер архитектура

### Архитектура HTTP сервера в Go

```
HTTP Server Architecture
┌─────────────────────────────────────────────────────────────┐
│                    HTTP Request Lifecycle                   │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│  │   Client    │────│  Network    │────│    Server   │      │
│  │             │    │   Stack     │    │   Socket    │      │
│  └─────────────┘    └─────────────┘    └─────────────┘      │
│         │                                      │            │
│         ▼                                      ▼            │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │               Go HTTP Server                            │ │
│  │                                                         │ │
│  │  ┌─────────────────────────────────────────────────────┐│ │
│  │  │                ServeMux Router                      ││ │
│  │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐             ││ │
│  │  │  │  /api/  │  │ /users/ │  │ /health │             ││ │
│  │  │  │ handler │  │ handler │  │ handler │             ││ │
│  │  │  └─────────┘  └─────────┘  └─────────┘             ││ │
│  │  └─────────────────────────────────────────────────────┘│ │
│  │                         │                               │ │
│  │                         ▼                               │ │
│  │  ┌─────────────────────────────────────────────────────┐│ │
│  │  │              Middleware Chain                       ││ │
│  │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐             ││ │
│  │  │  │ Logging │→ │  CORS   │→ │  Auth   │→ Handler   ││ │
│  │  │  └─────────┘  └─────────┘  └─────────┘             ││ │
│  │  └─────────────────────────────────────────────────────┘│ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │               Request Processing                        │ │
│  │                                                         │ │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │ │
│  │  │ Parse   │→ │Validate │→ │Business │→ │Response │   │ │
│  │  │Request  │  │  Data   │  │ Logic   │  │ Format  │   │ │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘

Goroutine Per Request Model
┌─────────────────────────────────────────────────────────────┐
│                    Connection Pool                          │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ Connection  │  │ Connection  │  │ Connection  │          │
│  │     1       │  │     2       │  │     3       │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│         │               │               │                  │
│         ▼               ▼               ▼                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ Goroutine   │  │ Goroutine   │  │ Goroutine   │          │
│  │     A       │  │     B       │  │     C       │          │
│  │             │  │             │  │             │          │
│  │ Handle Req  │  │ Handle Req  │  │ Handle Req  │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────────────────────┘
```

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strconv"
    "strings"
    "time"
    
    "github.com/gorilla/mux"
)

// Базовый HTTP сервер
func demonstrateBasicHTTPServer() {
    // Простейший обработчик
    http.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, World! Time: %s", time.Now().Format(time.RFC3339))
    })
    
    // Обработчик с параметрами URL
    http.HandleFunc("/user/", func(w http.ResponseWriter, r *http.Request) {
        // Извлекаем ID из URL path
        path := strings.TrimPrefix(r.URL.Path, "/user/")
        if path == "" {
            http.Error(w, "User ID required", http.StatusBadRequest)
            return
        }
        
        userID, err := strconv.ParseInt(path, 10, 64)
        if err != nil {
            http.Error(w, "Invalid user ID", http.StatusBadRequest)
            return
        }
        
        // Возвращаем JSON ответ
        response := map[string]interface{}{
            "user_id": userID,
            "name":    fmt.Sprintf("User %d", userID),
            "status":  "active",
        }
        
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(response)
    })
    
    fmt.Println("Server starting on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}

// Продвинутый HTTP сервер с конфигурацией
type HTTPServer struct {
    server   *http.Server
    router   *mux.Router
    userSvc  UserService
    logger   Logger
}

func NewHTTPServer(addr string, userSvc UserService, logger Logger) *HTTPServer {
    router := mux.NewRouter()
    
    server := &http.Server{
        Addr:         addr,
        Handler:      router,
        ReadTimeout:  15 * time.Second,
        WriteTimeout: 15 * time.Second,
        IdleTimeout:  60 * time.Second,
    }
    
    httpServer := &HTTPServer{
        server:  server,
        router:  router,
        userSvc: userSvc,
        logger:  logger,
    }
    
    httpServer.setupRoutes()
    httpServer.setupMiddleware()
    
    return httpServer
}

func (s *HTTPServer) setupRoutes() {
    // API v1 routes
    api := s.router.PathPrefix("/api/v1").Subrouter()
    
    // User routes
    users := api.PathPrefix("/users").Subrouter()
    users.HandleFunc("", s.handleCreateUser).Methods("POST")
    users.HandleFunc("", s.handleListUsers).Methods("GET")
    users.HandleFunc("/{id:[0-9]+}", s.handleGetUser).Methods("GET")
    users.HandleFunc("/{id:[0-9]+}", s.handleUpdateUser).Methods("PUT")
    users.HandleFunc("/{id:[0-9]+}", s.handleDeleteUser).Methods("DELETE")
    
    // Auth routes
    auth := api.PathPrefix("/auth").Subrouter()
    auth.HandleFunc("/login", s.handleLogin).Methods("POST")
    auth.HandleFunc("/logout", s.handleLogout).Methods("POST")
    auth.HandleFunc("/refresh", s.handleRefreshToken).Methods("POST")
    
    // Health check
    s.router.HandleFunc("/health", s.handleHealth).Methods("GET")
    s.router.HandleFunc("/ready", s.handleReadiness).Methods("GET")
    
    // Metrics endpoint
    s.router.HandleFunc("/metrics", s.handleMetrics).Methods("GET")
}

func (s *HTTPServer) setupMiddleware() {
    // Применяем middleware в правильном порядке
    s.router.Use(s.loggingMiddleware)
    s.router.Use(s.corsMiddleware)
    s.router.Use(s.rateLimitMiddleware)
    
    // Auth middleware только для защищенных роутов
    api := s.router.PathPrefix("/api/v1").Subrouter()
    api.Use(s.authMiddleware)
}

// Middleware для логирования
func (s *HTTPServer) loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        
        // Оборачиваем ResponseWriter для захвата статуса
        lrw := &loggingResponseWriter{
            ResponseWriter: w,
            statusCode:     http.StatusOK,
        }
        
        next.ServeHTTP(lrw, r)
        
        duration := time.Since(start)
        
        s.logger.Info("HTTP request",
            Field{Key: "method", Value: r.Method},
            Field{Key: "path", Value: r.URL.Path},
            Field{Key: "status", Value: lrw.statusCode},
            Field{Key: "duration", Value: duration.Milliseconds()},
            Field{Key: "ip", Value: r.RemoteAddr},
            Field{Key: "user_agent", Value: r.UserAgent()},
        )
    })
}

type loggingResponseWriter struct {
    http.ResponseWriter
    statusCode int
}

func (lrw *loggingResponseWriter) WriteHeader(code int) {
    lrw.statusCode = code
    lrw.ResponseWriter.WriteHeader(code)
}

// CORS middleware
func (s *HTTPServer) corsMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Access-Control-Allow-Origin", "*")
        w.Header().Set("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS")
        w.Header().Set("Access-Control-Allow-Headers", "Content-Type, Authorization")
        
        if r.Method == "OPTIONS" {
            w.WriteHeader(http.StatusOK)
            return
        }
        
        next.ServeHTTP(w, r)
    })
}

// Rate limiting middleware
func (s *HTTPServer) rateLimitMiddleware(next http.Handler) http.Handler {
    // Простейший rate limiter на основе IP
    clients := make(map[string]*RateLimiter)
    var mu sync.RWMutex
    
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        ip := r.RemoteAddr
        
        mu.RLock()
        limiter, exists := clients[ip]
        mu.RUnlock()
        
        if !exists {
            mu.Lock()
            limiter = NewRateLimiter(100, 10) // 100 req/sec, burst 10
            clients[ip] = limiter
            mu.Unlock()
        }
        
        if !limiter.Allow() {
            http.Error(w, "Rate limit exceeded", http.StatusTooManyRequests)
            return
        }
        
        next.ServeHTTP(w, r)
    })
}

// Authentication middleware
func (s *HTTPServer) authMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Пропускаем публичные endpoints
        if strings.HasSuffix(r.URL.Path, "/login") || 
           strings.HasSuffix(r.URL.Path, "/health") ||
           strings.HasSuffix(r.URL.Path, "/ready") {
            next.ServeHTTP(w, r)
            return
        }
        
        authHeader := r.Header.Get("Authorization")
        if authHeader == "" {
            http.Error(w, "Authorization header required", http.StatusUnauthorized)
            return
        }
        
        // Проверяем Bearer token
        parts := strings.Split(authHeader, " ")
        if len(parts) != 2 || parts[0] != "Bearer" {
            http.Error(w, "Invalid authorization header", http.StatusUnauthorized)
            return
        }
        
        token := parts[1]
        userID, err := s.validateJWT(token)
        if err != nil {
            http.Error(w, "Invalid token", http.StatusUnauthorized)
            return
        }
        
        // Добавляем user ID в контекст
        ctx := context.WithValue(r.Context(), "userID", userID)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

// User handlers
func (s *HTTPServer) handleCreateUser(w http.ResponseWriter, r *http.Request) {
    var req CreateUserRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        s.respondError(w, "Invalid JSON", http.StatusBadRequest)
        return
    }
    
    // Валидация
    if err := s.validateCreateUserRequest(req); err != nil {
        s.respondError(w, err.Error(), http.StatusBadRequest)
        return
    }
    
    // Создание пользователя
    user, err := s.userSvc.CreateUser(r.Context(), req)
    if err != nil {
        s.logger.Error("Failed to create user", Field{Key: "error", Value: err})
        s.respondError(w, "Internal server error", http.StatusInternalServerError)
        return
    }
    
    s.respondJSON(w, user.ToResponse(), http.StatusCreated)
}

func (s *HTTPServer) handleGetUser(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    userID, err := strconv.ParseInt(vars["id"], 10, 64)
    if err != nil {
        s.respondError(w, "Invalid user ID", http.StatusBadRequest)
        return
    }
    
    user, err := s.userSvc.GetUserByID(r.Context(), userID)
    if err != nil {
        if err == ErrUserNotFound {
            s.respondError(w, "User not found", http.StatusNotFound)
            return
        }
        s.logger.Error("Failed to get user", 
            Field{Key: "user_id", Value: userID},
            Field{Key: "error", Value: err},
        )
        s.respondError(w, "Internal server error", http.StatusInternalServerError)
        return
    }
    
    s.respondJSON(w, user.ToResponse(), http.StatusOK)
}

func (s *HTTPServer) handleListUsers(w http.ResponseWriter, r *http.Request) {
    // Парсим query parameters для пагинации
    page, _ := strconv.Atoi(r.URL.Query().Get("page"))
    if page < 1 {
        page = 1
    }
    
    limit, _ := strconv.Atoi(r.URL.Query().Get("limit"))
    if limit < 1 || limit > 100 {
        limit = 20
    }
    
    offset := (page - 1) * limit
    
    users, total, err := s.userSvc.ListUsers(r.Context(), offset, limit)
    if err != nil {
        s.logger.Error("Failed to list users", Field{Key: "error", Value: err})
        s.respondError(w, "Internal server error", http.StatusInternalServerError)
        return
    }
    
    // Конвертируем в response DTOs
    userResponses := make([]UserResponse, len(users))
    for i, user := range users {
        userResponses[i] = user.ToResponse()
    }
    
    // Создаем пагинированный ответ
    response := NewPaginatedResponse(userResponses, page, limit, total)
    s.respondJSON(w, response, http.StatusOK)
}

func (s *HTTPServer) handleUpdateUser(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    userID, err := strconv.ParseInt(vars["id"], 10, 64)
    if err != nil {
        s.respondError(w, "Invalid user ID", http.StatusBadRequest)
        return
    }
    
    // Проверяем права доступа
    currentUserID := r.Context().Value("userID").(int64)
    if currentUserID != userID {
        // Дополнительная проверка прав админа
        if !s.isAdmin(r.Context(), currentUserID) {
            s.respondError(w, "Access denied", http.StatusForbidden)
            return
        }
    }
    
    var req UpdateUserRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        s.respondError(w, "Invalid JSON", http.StatusBadRequest)
        return
    }
    
    user, err := s.userSvc.UpdateUser(r.Context(), userID, req)
    if err != nil {
        if err == ErrUserNotFound {
            s.respondError(w, "User not found", http.StatusNotFound)
            return
        }
        s.logger.Error("Failed to update user",
            Field{Key: "user_id", Value: userID},
            Field{Key: "error", Value: err},
        )
        s.respondError(w, "Internal server error", http.StatusInternalServerError)
        return
    }
    
    s.respondJSON(w, user.ToResponse(), http.StatusOK)
}

func (s *HTTPServer) handleDeleteUser(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    userID, err := strconv.ParseInt(vars["id"], 10, 64)
    if err != nil {
        s.respondError(w, "Invalid user ID", http.StatusBadRequest)
        return
    }
    
    // Проверяем права доступа (только админы)
    currentUserID := r.Context().Value("userID").(int64)
    if !s.isAdmin(r.Context(), currentUserID) {
        s.respondError(w, "Access denied", http.StatusForbidden)
        return
    }
    
    err = s.userSvc.DeleteUser(r.Context(), userID)
    if err != nil {
        if err == ErrUserNotFound {
            s.respondError(w, "User not found", http.StatusNotFound)
            return
        }
        s.logger.Error("Failed to delete user",
            Field{Key: "user_id", Value: userID},
            Field{Key: "error", Value: err},
        )
        s.respondError(w, "Internal server error", http.StatusInternalServerError)
        return
    }
    
    w.WriteHeader(http.StatusNoContent)
}

// Auth handlers
func (s *HTTPServer) handleLogin(w http.ResponseWriter, r *http.Request) {
    var req LoginRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        s.respondError(w, "Invalid JSON", http.StatusBadRequest)
        return
    }
    
    // Аутентификация
    user, err := s.userSvc.AuthenticateUser(r.Context(), req.Email, req.Password)
    if err != nil {
        if err == ErrInvalidCredentials {
            s.respondError(w, "Invalid credentials", http.StatusUnauthorized)
            return
        }
        s.logger.Error("Failed to authenticate user",
            Field{Key: "email", Value: req.Email},
            Field{Key: "error", Value: err},
        )
        s.respondError(w, "Internal server error", http.StatusInternalServerError)
        return
    }
    
    // Генерируем JWT токены
    accessToken, err := s.generateAccessToken(user.ID)
    if err != nil {
        s.logger.Error("Failed to generate access token", Field{Key: "error", Value: err})
        s.respondError(w, "Internal server error", http.StatusInternalServerError)
        return
    }
    
    refreshToken, err := s.generateRefreshToken(user.ID)
    if err != nil {
        s.logger.Error("Failed to generate refresh token", Field{Key: "error", Value: err})
        s.respondError(w, "Internal server error", http.StatusInternalServerError)
        return
    }
    
    response := LoginResponse{
        User:         user.ToResponse(),
        AccessToken:  accessToken,
        RefreshToken: refreshToken,
        ExpiresIn:    3600, // 1 час
    }
    
    s.respondJSON(w, response, http.StatusOK)
}

// Health check handlers
func (s *HTTPServer) handleHealth(w http.ResponseWriter, r *http.Request) {
    health := map[string]interface{}{
        "status":    "healthy",
        "timestamp": time.Now(),
        "version":   "1.0.0",
    }
    
    s.respondJSON(w, health, http.StatusOK)
}

func (s *HTTPServer) handleReadiness(w http.ResponseWriter, r *http.Request) {
    checks := map[string]string{
        "database": "ok",
        "redis":    "ok",
    }
    
    // Проверяем подключения
    if err := s.userSvc.HealthCheck(r.Context()); err != nil {
        checks["database"] = "failed"
        s.respondJSON(w, map[string]interface{}{
            "status": "not ready",
            "checks": checks,
        }, http.StatusServiceUnavailable)
        return
    }
    
    s.respondJSON(w, map[string]interface{}{
        "status": "ready",
        "checks": checks,
    }, http.StatusOK)
}

// Utility methods
func (s *HTTPServer) respondJSON(w http.ResponseWriter, data interface{}, status int) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    
    if err := json.NewEncoder(w).Encode(data); err != nil {
        s.logger.Error("Failed to encode JSON response", Field{Key: "error", Value: err})
    }
}

func (s *HTTPServer) respondError(w http.ResponseWriter, message string, status int) {
    response := map[string]interface{}{
        "error":   message,
        "status":  status,
        "timestamp": time.Now(),
    }
    
    s.respondJSON(w, response, status)
}

func (s *HTTPServer) validateCreateUserRequest(req CreateUserRequest) error {
    if req.Email == "" {
        return fmt.Errorf("email is required")
    }
    if req.Name == "" {
        return fmt.Errorf("name is required")
    }
    if len(req.Password) < 8 {
        return fmt.Errorf("password must be at least 8 characters")
    }
    return nil
}

func (s *HTTPServer) validateJWT(token string) (int64, error) {
    // Simplified JWT validation - в реальности используйте jwt-go
    if token == "valid-token" {
        return 1, nil
    }
    return 0, fmt.Errorf("invalid token")
}

func (s *HTTPServer) isAdmin(ctx context.Context, userID int64) bool {
    user, err := s.userSvc.GetUserByID(ctx, userID)
    if err != nil {
        return false
    }
    return user.Role == RoleAdmin
}

func (s *HTTPServer) generateAccessToken(userID int64) (string, error) {
    // Simplified token generation
    return fmt.Sprintf("access-token-%d", userID), nil
}

func (s *HTTPServer) generateRefreshToken(userID int64) (string, error) {
    // Simplified token generation
    return fmt.Sprintf("refresh-token-%d", userID), nil
}

// Graceful shutdown
func (s *HTTPServer) Start() error {
    s.logger.Info("Starting HTTP server", Field{Key: "addr", Value: s.server.Addr})
    return s.server.ListenAndServe()
}

func (s *HTTPServer) Stop(ctx context.Context) error {
    s.logger.Info("Stopping HTTP server")
    return s.server.Shutdown(ctx)
}

// Request/Response DTOs
type CreateUserRequest struct {
    Email    string `json:"email"`
    Name     string `json:"name"`
    Password string `json:"password"`
}

type LoginRequest struct {
    Email    string `json:"email"`
    Password string `json:"password"`
}

type LoginResponse struct {
    User         UserResponse `json:"user"`
    AccessToken  string       `json:"access_token"`
    RefreshToken string       `json:"refresh_token"`
    ExpiresIn    int          `json:"expires_in"`
}

// Errors
var (
    ErrUserNotFound       = fmt.Errorf("user not found")
    ErrInvalidCredentials = fmt.Errorf("invalid credentials")
)

// User service interface
type UserService interface {
    CreateUser(ctx context.Context, req CreateUserRequest) (*User, error)
    GetUserByID(ctx context.Context, id int64) (*User, error)
    ListUsers(ctx context.Context, offset, limit int) ([]*User, int, error)
    UpdateUser(ctx context.Context, id int64, req UpdateUserRequest) (*User, error)
    DeleteUser(ctx context.Context, id int64) error
    AuthenticateUser(ctx context.Context, email, password string) (*User, error)
    HealthCheck(ctx context.Context) error
}

// File upload handler
func (s *HTTPServer) handleFileUpload(w http.ResponseWriter, r *http.Request) {
    // Ограничиваем размер файла (10MB)
    r.ParseMultipartForm(10 << 20)
    
    file, handler, err := r.FormFile("upload")
    if err != nil {
        s.respondError(w, "Error retrieving file", http.StatusBadRequest)
        return
    }
    defer file.Close()
    
    // Валидация типа файла
    allowedTypes := map[string]bool{
        "image/jpeg": true,
        "image/png":  true,
        "image/gif":  true,
    }
    
    contentType := handler.Header.Get("Content-Type")
    if !allowedTypes[contentType] {
        s.respondError(w, "File type not allowed", http.StatusBadRequest)
        return
    }
    
    // Здесь был бы код сохранения файла
    s.logger.Info("File uploaded",
        Field{Key: "filename", Value: handler.Filename},
        Field{Key: "size", Value: handler.Size},
        Field{Key: "content_type", Value: contentType},
    )
    
    response := map[string]interface{}{
        "message":  "File uploaded successfully",
        "filename": handler.Filename,
        "size":     handler.Size,
    }
    
    s.respondJSON(w, response, http.StatusOK)
}

// Streaming response example
func (s *HTTPServer) handleStreamingData(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    w.Header().Set("Transfer-Encoding", "chunked")
    
    flusher, ok := w.(http.Flusher)
    if !ok {
        s.respondError(w, "Streaming not supported", http.StatusInternalServerError)
        return
    }
    
    encoder := json.NewEncoder(w)
    
    for i := 1; i <= 10; i++ {
        data := map[string]interface{}{
            "id":        i,
            "message":   fmt.Sprintf("Chunk %d", i),
            "timestamp": time.Now(),
        }
        
        if err := encoder.Encode(data); err != nil {
            s.logger.Error("Failed to encode streaming data", Field{Key: "error", Value: err})
            return
        }
        
        flusher.Flush()
        time.Sleep(500 * time.Millisecond)
    }
}

// WebSocket upgrade (если нужен real-time)
func (s *HTTPServer) handleWebSocket(w http.ResponseWriter, r *http.Request) {
    // Здесь был бы код для WebSocket upgrade
    // Используя gorilla/websocket или аналогичную библиотеку
    s.respondError(w, "WebSocket not implemented", http.StatusNotImplemented)
}

// Metrics handler для мониторинга
func (s *HTTPServer) handleMetrics(w http.ResponseWriter, r *http.Request) {
    metrics := map[string]interface{}{
        "goroutines":    runtime.NumGoroutine(),
        "memory_alloc":  getMemStats().Alloc,
        "memory_sys":    getMemStats().Sys,
        "gc_cycles":     getMemStats().NumGC,
        "uptime":        time.Since(startTime).Seconds(),
    }
    
    s.respondJSON(w, metrics, http.StatusOK)
}

var startTime = time.Now()

func getMemStats() runtime.MemStats {
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    return m
}

// Пример запуска сервера
func main() {
    // В реальном приложении эти зависимости создавались бы через DI
    logger := &ConsoleLogger{}
    userRepo := NewMockUserRepository()
    userSvc := NewUserService(userRepo, logger)
    
    server := NewHTTPServer(":8080", userSvc, logger)
    
    // Graceful shutdown
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    
    go func() {
        if err := server.Start(); err != http.ErrServerClosed {
            log.Fatal("Server failed to start:", err)
        }
    }()
    
    // Ждем сигнал завершения
    // В реальном приложении здесь был бы обработчик сигналов
    time.Sleep(30 * time.Second)
    
    // Graceful shutdown
    shutdownCtx, shutdownCancel := context.WithTimeout(ctx, 30*time.Second)
    defer shutdownCancel()
    
    if err := server.Stop(shutdownCtx); err != nil {
        log.Fatal("Server failed to stop gracefully:", err)
    }
    
    log.Println("Server stopped gracefully")
}

// Console logger implementation for demo
type ConsoleLogger struct{}

func (c *ConsoleLogger) Debug(msg string, fields ...Field) {
    log.Printf("DEBUG: %s %v", msg, fields)
}

func (c *ConsoleLogger) Info(msg string, fields ...Field) {
    log.Printf("INFO: %s %v", msg, fields)
}

func (c *ConsoleLogger) Warn(msg string, fields ...Field) {
    log.Printf("WARN: %s %v", msg, fields)
}

func (c *ConsoleLogger) Error(msg string, fields ...Field) {
    log.Printf("ERROR: %s %v", msg, fields)
}

func (c *ConsoleLogger) Fatal(msg string, fields ...Field) {
    log.Fatalf("FATAL: %s %v", msg, fields)
}
```

**🎯 Практическая польза HTTP серверов:**
- **Масштабируемость** - горутины позволяют обрабатывать тысячи одновременных запросов
- **Middleware** - модульная архитектура для добавления функциональности
- **Типобезопасность** - структуры для request/response предотвращают ошибки
- **Мониторинг** - встроенные endpoints для health checks и метрик

**💡 Архитектурные принципы:**
- Разделение ответственности между слоями
- Middleware для cross-cutting concerns
- Graceful shutdown для стабильности
- Comprehensive error handling и логирование

---

### Неделя 5: Работа с базами данных

#### 🗄️ Database архитектура в Go

### Database Connection Architecture

```
Database Connection Architecture
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                        │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │   Service   │  │   Service   │  │   Service   │          │
│  │    Layer    │  │    Layer    │  │    Layer    │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│         │               │               │                  │
│         ▼               ▼               ▼                  │
├─────────────────────────────────────────────────────────────┤
│                  Repository Layer                           │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │    User     │  │    Order    │  │   Product   │          │
│  │ Repository  │  │ Repository  │  │ Repository  │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│         │               │               │                  │
│         ▼               ▼               ▼                  │
├─────────────────────────────────────────────────────────────┤
│                 Database Layer                              │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐ │
│  │              Connection Pool                            │ │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │ │
│  │  │  Conn   │  │  Conn   │  │  Conn   │  │  Conn   │   │ │
│  │  │    1    │  │    2    │  │    3    │  │   ...   │   │ │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │ │
│  └─────────────────────────────────────────────────────────┘ │
│                           │                                 │
│                           ▼                                 │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                PostgreSQL Database                     │ │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐                │ │
│  │  │  users  │  │ orders  │  │products │                │ │
│  │  │  table  │  │  table  │  │  table  │                │ │
│  │  └─────────┘  └─────────┘  └─────────┘                │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘

Transaction Flow
┌─────────────────────────────────────────────────────────────┐
│                    Transaction Lifecycle                    │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│  │    BEGIN    │ ─→ │  EXECUTE    │ ─→ │   COMMIT    │      │
│  │             │    │  QUERIES    │    │     OR      │      │
│  │             │    │             │    │  ROLLBACK   │      │
│  └─────────────┘    └─────────────┘    └─────────────┘      │
│         │                 │                     │           │
│         ▼                 ▼                     ▼           │
│  Start transaction   Execute SQL       Apply changes        │
│  Get connection      within context    or revert all        │
└─────────────────────────────────────────────────────────────┘
```

```go
package main

import (
    "context"
    "database/sql"
    "fmt"
    "log"
    "time"

    _ "github.com/lib/pq" // PostgreSQL driver
    "github.com/jmoiron/sqlx"
)

// Database configuration
type DatabaseConfig struct {
    Host         string
    Port         int
    User         string
    Password     string
    Database     string
    SSLMode      string
    MaxOpenConns int
    MaxIdleConns int
    MaxLifetime  time.Duration
}

func (cfg DatabaseConfig) DSN() string {
    return fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=%s",
        cfg.Host, cfg.Port, cfg.User, cfg.Password, cfg.Database, cfg.SSLMode)
}

// Database connection wrapper
type Database struct {
    db     *sqlx.DB
    config DatabaseConfig
}

func NewDatabase(config DatabaseConfig) (*Database, error) {
    db, err := sqlx.Connect("postgres", config.DSN())
    if err != nil {
        return nil, fmt.Errorf("failed to connect to database: %w", err)
    }

    // Настройка connection pool
    db.SetMaxOpenConns(config.MaxOpenConns)
    db.SetMaxIdleConns(config.MaxIdleConns)
    db.SetConnMaxLifetime(config.MaxLifetime)

    // Проверка подключения
    if err := db.Ping(); err != nil {
        return nil, fmt.Errorf("failed to ping database: %w", err)
    }

    return &Database{
        db:     db,
        config: config,
    }, nil
}

func (d *Database) Close() error {
    return d.db.Close()
}

func (d *Database) HealthCheck(ctx context.Context) error {
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()
    
    return d.db.PingContext(ctx)
}

func (d *Database) Stats() sql.DBStats {
    return d.db.Stats()
}

// Модели данных с database тегами
type User struct {
    ID        int64     `db:"id" json:"id"`
    Email     string    `db:"email" json:"email"`
    Name      string    `db:"name" json:"name"`
    Password  string    `db:"password_hash" json:"-"`
    IsActive  bool      `db:"is_active" json:"is_active"`
    Role      string    `db:"role" json:"role"`
    CreatedAt time.Time `db:"created_at" json:"created_at"`
    UpdatedAt time.Time `db:"updated_at" json:"updated_at"`
    DeletedAt *time.Time `db:"deleted_at" json:"deleted_at,omitempty"`
}

type Order struct {
    ID         int64     `db:"id" json:"id"`
    UserID     int64     `db:"user_id" json:"user_id"`
    TotalAmount float64   `db:"total_amount" json:"total_amount"`
    Status     string    `db:"status" json:"status"`
    CreatedAt  time.Time `db:"created_at" json:"created_at"`
    UpdatedAt  time.Time `db:"updated_at" json:"updated_at"`
    
    // Связанные данные
    User  *User        `json:"user,omitempty"`
    Items []OrderItem  `json:"items,omitempty"`
}

type OrderItem struct {
    ID        int64   `db:"id" json:"id"`
    OrderID   int64   `db:"order_id" json:"order_id"`
    ProductID int64   `db:"product_id" json:"product_id"`
    Quantity  int     `db:"quantity" json:"quantity"`
    Price     float64 `db:"price" json:"price"`
}

// Repository interface
type UserRepository interface {
    Create(ctx context.Context, user *User) error
    GetByID(ctx context.Context, id int64) (*User, error)
    GetByEmail(ctx context.Context, email string) (*User, error)
    Update(ctx context.Context, user *User) error
    Delete(ctx context.Context, id int64) error
    List(ctx context.Context, filters UserFilters) ([]*User, error)
    Count(ctx context.Context, filters UserFilters) (int, error)
}

type UserFilters struct {
    IsActive *bool
    Role     string
    Search   string
    Limit    int
    Offset   int
}

// PostgreSQL repository implementation
type PostgresUserRepository struct {
    db *Database
}

func NewPostgresUserRepository(db *Database) UserRepository {
    return &PostgresUserRepository{db: db}
}

func (r *PostgresUserRepository) Create(ctx context.Context, user *User) error {
    query := `
        INSERT INTO users (email, name, password_hash, is_active, role, created_at, updated_at)
        VALUES (:email, :name, :password_hash, :is_active, :role, :created_at, :updated_at)
        RETURNING id`
    
    // Устанавливаем временные метки
    now := time.Now()
    user.CreatedAt = now
    user.UpdatedAt = now
    
    // Используем NamedQuery для работы со структурой
    rows, err := r.db.db.NamedQuery(query, user)
    if err != nil {
        return fmt.Errorf("failed to create user: %w", err)
    }
    defer rows.Close()
    
    if rows.Next() {
        err = rows.Scan(&user.ID)
        if err != nil {
            return fmt.Errorf("failed to scan user ID: %w", err)
        }
    }
    
    return nil
}

func (r *PostgresUserRepository) GetByID(ctx context.Context, id int64) (*User, error) {
    user := &User{}
    query := `
        SELECT id, email, name, password_hash, is_active, role, created_at, updated_at, deleted_at
        FROM users 
        WHERE id = $1 AND deleted_at IS NULL`
    
    err := r.db.db.GetContext(ctx, user, query, id)
    if err != nil {
        if err == sql.ErrNoRows {
            return nil, ErrUserNotFound
        }
        return nil, fmt.Errorf("failed to get user by id %d: %w", id, err)
    }
    
    return user, nil
}

func (r *PostgresUserRepository) GetByEmail(ctx context.Context, email string) (*User, error) {
    user := &User{}
    query := `
        SELECT id, email, name, password_hash, is_active, role, created_at, updated_at, deleted_at
        FROM users 
        WHERE email = $1 AND deleted_at IS NULL`
    
    err := r.db.db.GetContext(ctx, user, query, email)
    if err != nil {
        if err == sql.ErrNoRows {
            return nil, ErrUserNotFound
        }
        return nil, fmt.Errorf("failed to get user by email %s: %w", email, err)
    }
    
    return user, nil
}

func (r *PostgresUserRepository) Update(ctx context.Context, user *User) error {
    user.UpdatedAt = time.Now()
    
    query := `
        UPDATE users 
        SET email = :email, name = :name, password_hash = :password_hash, 
            is_active = :is_active, role = :role, updated_at = :updated_at
        WHERE id = :id AND deleted_at IS NULL`
    
    result, err := r.db.db.NamedExecContext(ctx, query, user)
    if err != nil {
        return fmt.Errorf("failed to update user: %w", err)
    }
    
    rowsAffected, err := result.RowsAffected()
    if err != nil {
        return fmt.Errorf("failed to get rows affected: %w", err)
    }
    
    if rowsAffected == 0 {
        return ErrUserNotFound
    }
    
    return nil
}

func (r *PostgresUserRepository) Delete(ctx context.Context, id int64) error {
    // Soft delete
    query := `UPDATE users SET deleted_at = $2 WHERE id = $1 AND deleted_at IS NULL`
    
    result, err := r.db.db.ExecContext(ctx, query, id, time.Now())
    if err != nil {
        return fmt.Errorf("failed to delete user: %w", err)
    }
    
    rowsAffected, err := result.RowsAffected()
    if err != nil {
        return fmt.Errorf("failed to get rows affected: %w", err)
    }
    
    if rowsAffected == 0 {
        return ErrUserNotFound
    }
    
    return nil
}

func (r *PostgresUserRepository) List(ctx context.Context, filters UserFilters) ([]*User, error) {
    var users []*User
    
    // Строим динамический запрос
    query := `
        SELECT id, email, name, password_hash, is_active, role, created_at, updated_at, deleted_at
        FROM users 
        WHERE deleted_at IS NULL`
    
    args := []interface{}{}
    argIndex := 1
    
    // Добавляем фильтры
    if filters.IsActive != nil {
        query += fmt.Sprintf(" AND is_active = $%d", argIndex)
        args = append(args, *filters.IsActive)
        argIndex++
    }
    
    if filters.Role != "" {
        query += fmt.Sprintf(" AND role = $%d", argIndex)
        args = append(args, filters.Role)
        argIndex++
    }
    
    if filters.Search != "" {
        query += fmt.Sprintf(" AND (name ILIKE $%d OR email ILIKE $%d)", argIndex, argIndex+1)
        searchPattern := "%" + filters.Search + "%"
        args = append(args, searchPattern, searchPattern)
        argIndex += 2
    }
    
    // Сортировка и пагинация
    query += " ORDER BY created_at DESC"
    
    if filters.Limit > 0 {
        query += fmt.Sprintf(" LIMIT $%d", argIndex)
        args = append(args, filters.Limit)
        argIndex++
    }
    
    if filters.Offset > 0 {
        query += fmt.Sprintf(" OFFSET $%d", argIndex)
        args = append(args, filters.Offset)
    }
    
    err := r.db.db.SelectContext(ctx, &users, query, args...)
    if err != nil {
        return nil, fmt.Errorf("failed to list users: %w", err)
    }
    
    return users, nil
}

func (r *PostgresUserRepository) Count(ctx context.Context, filters UserFilters) (int, error) {
    query := `SELECT COUNT(*) FROM users WHERE deleted_at IS NULL`
    
    args := []interface{}{}
    argIndex := 1
    
    // Те же фильтры, что и в List
    if filters.IsActive != nil {
        query += fmt.Sprintf(" AND is_active = $%d", argIndex)
        args = append(args, *filters.IsActive)
        argIndex++
    }
    
    if filters.Role != "" {
        query += fmt.Sprintf(" AND role = $%d", argIndex)
        args = append(args, filters.Role)
        argIndex++
    }
    
    if filters.Search != "" {
        query += fmt.Sprintf(" AND (name ILIKE $%d OR email ILIKE $%d)", argIndex, argIndex+1)
        searchPattern := "%" + filters.Search + "%"
        args = append(args, searchPattern, searchPattern)
    }
    
    var count int
    err := r.db.db.GetContext(ctx, &count, query, args...)
    if err != nil {
        return 0, fmt.Errorf("failed to count users: %w", err)
    }
    
    return count, nil
}

// Transaction support
type Transactional interface {
    WithTransaction(ctx context.Context, fn func(ctx context.Context, tx *sqlx.Tx) error) error
}

func (d *Database) WithTransaction(ctx context.Context, fn func(ctx context.Context, tx *sqlx.Tx) error) error {
    tx, err := d.db.BeginTxx(ctx, nil)
    if err != nil {
        return fmt.Errorf("failed to begin transaction: %w", err)
    }
    
    defer func() {
        if p := recover(); p != nil {
            tx.Rollback()
            panic(p)
        }
    }()
    
    if err := fn(ctx, tx); err != nil {
        if rollbackErr := tx.Rollback(); rollbackErr != nil {
            return fmt.Errorf("failed to rollback transaction: %v (original error: %w)", rollbackErr, err)
        }
        return err
    }
    
    if err := tx.Commit(); err != nil {
        return fmt.Errorf("failed to commit transaction: %w", err)
    }
    
    return nil
}

// Service layer with transaction support
type UserService struct {
    userRepo UserRepository
    db       *Database
    logger   Logger
}

func NewUserService(userRepo UserRepository, db *Database, logger Logger) *UserService {
    return &UserService{
        userRepo: userRepo,
        db:       db,
        logger:   logger,
    }
}

func (s *UserService) CreateUserWithOrder(ctx context.Context, email, name string, orderAmount float64) (*User, *Order, error) {
    var user *User
    var order *Order
    
    err := s.db.WithTransaction(ctx, func(ctx context.Context, tx *sqlx.Tx) error {
        // Создаем пользователя
        user = &User{
            Email:    email,
            Name:     name,
            IsActive: true,
            Role:     "user",
        }
        
        // В транзакции нам нужен репозиторий, работающий с tx
        txUserRepo := NewTransactionalUserRepository(tx)
        if err := txUserRepo.Create(ctx, user); err != nil {
            return fmt.Errorf("failed to create user: %w", err)
        }
        
        // Создаем заказ
        order = &Order{
            UserID:      user.ID,
            TotalAmount: orderAmount,
            Status:      "pending",
            CreatedAt:   time.Now(),
            UpdatedAt:   time.Now(),
        }
        
        txOrderRepo := NewTransactionalOrderRepository(tx)
        if err := txOrderRepo.Create(ctx, order); err != nil {
            return fmt.Errorf("failed to create order: %w", err)
        }
        
        return nil
    })
    
    if err != nil {
        s.logger.Error("Failed to create user with order",
            Field{Key: "email", Value: email},
            Field{Key: "error", Value: err},
        )
        return nil, nil, err
    }
    
    return user, order, nil
}

// Transactional repository
type TransactionalUserRepository struct {
    tx *sqlx.Tx
}

func NewTransactionalUserRepository(tx *sqlx.Tx) *TransactionalUserRepository {
    return &TransactionalUserRepository{tx: tx}
}

func (r *TransactionalUserRepository) Create(ctx context.Context, user *User) error {
    query := `
        INSERT INTO users (email, name, password_hash, is_active, role, created_at, updated_at)
        VALUES ($1, $2, $3, $4, $5, $6, $7)
        RETURNING id`
    
    now := time.Now()
    user.CreatedAt = now
    user.UpdatedAt = now
    
    err := r.tx.QueryRowContext(ctx, query, 
        user.Email, user.Name, user.Password, user.IsActive, user.Role, now, now,
    ).Scan(&user.ID)
    
    if err != nil {
        return fmt.Errorf("failed to create user in transaction: %w", err)
    }
    
    return nil
}

// Order repository for transaction example
type OrderRepository interface {
    Create(ctx context.Context, order *Order) error
    GetByID(ctx context.Context, id int64) (*Order, error)
    GetByUserID(ctx context.Context, userID int64) ([]*Order, error)
}

type TransactionalOrderRepository struct {
    tx *sqlx.Tx
}

func NewTransactionalOrderRepository(tx *sqlx.Tx) *TransactionalOrderRepository {
    return &TransactionalOrderRepository{tx: tx}
}

func (r *TransactionalOrderRepository) Create(ctx context.Context, order *Order) error {
    query := `
        INSERT INTO orders (user_id, total_amount, status, created_at, updated_at)
        VALUES ($1, $2, $3, $4, $5)
        RETURNING id`
    
    err := r.tx.QueryRowContext(ctx, query,
        order.UserID, order.TotalAmount, order.Status, order.CreatedAt, order.UpdatedAt,
    ).Scan(&order.ID)
    
    if err != nil {
        return fmt.Errorf("failed to create order in transaction: %w", err)
    }
    
    return nil
}

// Query builder для сложных запросов
type QueryBuilder struct {
    query  string
    args   []interface{}
    argNum int
}

func NewQueryBuilder(baseQuery string) *QueryBuilder {
    return &QueryBuilder{
        query:  baseQuery,
        args:   make([]interface{}, 0),
        argNum: 1,
    }
}

func (qb *QueryBuilder) Where(condition string, args ...interface{}) *QueryBuilder {
    if strings.Contains(qb.query, "WHERE") {
        qb.query += " AND "
    } else {
        qb.query += " WHERE "
    }
    
    // Заменяем ? на $N
    for i := 0; i < len(args); i++ {
        condition = strings.Replace(condition, "?", fmt.Sprintf("$%d", qb.argNum), 1)
        qb.argNum++
    }
    
    qb.query += condition
    qb.args = append(qb.args, args...)
    
    return qb
}

func (qb *QueryBuilder) OrderBy(column, direction string) *QueryBuilder {
    qb.query += fmt.Sprintf(" ORDER BY %s %s", column, direction)
    return qb
}

func (qb *QueryBuilder) Limit(limit int) *QueryBuilder {
    qb.query += fmt.Sprintf(" LIMIT $%d", qb.argNum)
    qb.args = append(qb.args, limit)
    qb.argNum++
    return qb
}

func (qb *QueryBuilder) Offset(offset int) *QueryBuilder {
    qb.query += fmt.Sprintf(" OFFSET $%d", qb.argNum)
    qb.args = append(qb.args, offset)
    qb.argNum++
    return qb
}

func (qb *QueryBuilder) Build() (string, []interface{}) {
    return qb.query, qb.args
}

// Использование query builder
func (r *PostgresUserRepository) SearchUsers(ctx context.Context, search string, isActive *bool, limit, offset int) ([]*User, error) {
    qb := NewQueryBuilder("SELECT id, email, name, is_active, role, created_at, updated_at FROM users")
    
    qb.Where("deleted_at IS NULL")
    
    if search != "" {
        qb.Where("(name ILIKE ? OR email ILIKE ?)", "%"+search+"%", "%"+search+"%")
    }
    
    if isActive != nil {
        qb.Where("is_active = ?", *isActive)
    }
    
    qb.OrderBy("created_at", "DESC").Limit(limit).Offset(offset)
    
    query, args := qb.Build()
    
    var users []*User
    err := r.db.db.SelectContext(ctx, &users, query, args...)
    if err != nil {
        return nil, fmt.Errorf("failed to search users: %w", err)
    }
    
    return users, nil
}

// Connection pooling monitoring
func (d *Database) LogStats() {
    stats := d.db.Stats()
    log.Printf("DB Pool Stats: Open=%d InUse=%d Idle=%d WaitCount=%d WaitDuration=%v MaxIdleClosed=%d MaxLifetimeClosed=%d",
        stats.OpenConnections,
        stats.InUse,
        stats.Idle,
        stats.WaitCount,
        stats.WaitDuration,
        stats.MaxIdleClosed,
        stats.MaxLifetimeClosed,
    )
}

// Prepared statements для производительности
type PreparedStatements struct {
    getUserByID    *sqlx.Stmt
    getUserByEmail *sqlx.Stmt
    createUser     *sqlx.Stmt
}

func NewPreparedStatements(db *sqlx.DB) (*PreparedStatements, error) {
    getUserByID, err := db.Preparex("SELECT id, email, name, is_active, role, created_at, updated_at FROM users WHERE id = $1 AND deleted_at IS NULL")
    if err != nil {
        return nil, err
    }
    
    getUserByEmail, err := db.Preparex("SELECT id, email, name, is_active, role, created_at, updated_at FROM users WHERE email = $1 AND deleted_at IS NULL")
    if err != nil {
        return nil, err
    }
    
    createUser, err := db.Preparex("INSERT INTO users (email, name, is_active, role, created_at, updated_at) VALUES ($1, $2, $3, $4, $5, $6) RETURNING id")
    if err != nil {
        return nil, err
    }
    
    return &PreparedStatements{
        getUserByID:    getUserByID,
        getUserByEmail: getUserByEmail,
        createUser:     createUser,
    }, nil
}

func (ps *PreparedStatements) Close() error {
    ps.getUserByID.Close()
    ps.getUserByEmail.Close()
    ps.createUser.Close()
    return nil
}

// Migration system
type Migration struct {
    Version int
    Name    string
    Up      string
    Down    string
}

var migrations = []Migration{
    {
        Version: 1,
        Name:    "create_users_table",
        Up: `
            CREATE TABLE users (
                id BIGSERIAL PRIMARY KEY,
                email VARCHAR(255) UNIQUE NOT NULL,
                name VARCHAR(255) NOT NULL,
                password_hash VARCHAR(255) NOT NULL,
                is_active BOOLEAN DEFAULT true,
                role VARCHAR(50) DEFAULT 'user',
                created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
                updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
                deleted_at TIMESTAMP WITH TIME ZONE
            );
            
            CREATE INDEX idx_users_email ON users(email);
            CREATE INDEX idx_users_deleted_at ON users(deleted_at);
        `,
        Down: `DROP TABLE users;`,
    },
    {
        Version: 2,
        Name:    "create_orders_table",
        Up: `
            CREATE TABLE orders (
                id BIGSERIAL PRIMARY KEY,
                user_id BIGINT NOT NULL REFERENCES users(id),
                total_amount DECIMAL(10,2) NOT NULL,
                status VARCHAR(50) DEFAULT 'pending',
                created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
                updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
            );
            
            CREATE INDEX idx_orders_user_id ON orders(user_id);
            CREATE INDEX idx_orders_status ON orders(status);
        `,
        Down: `DROP TABLE orders;`,
    },
}

func RunMigrations(db *Database) error {
    // Создаем таблицу миграций
    _, err := db.db.Exec(`
        CREATE TABLE IF NOT EXISTS schema_migrations (
            version INTEGER PRIMARY KEY,
            name VARCHAR(255) NOT NULL,
            applied_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
        )
    `)
    if err != nil {
        return fmt.Errorf("failed to create migrations table: %w", err)
    }
    
    // Получаем примененные миграции
    appliedVersions := make(map[int]bool)
    rows, err := db.db.Query("SELECT version FROM schema_migrations")
    if err != nil {
        return fmt.Errorf("failed to get applied migrations: %w", err)
    }
    defer rows.Close()
    
    for rows.Next() {
        var version int
        if err := rows.Scan(&version); err != nil {
            return fmt.Errorf("failed to scan migration version: %w", err)
        }
        appliedVersions[version] = true
    }
    
    // Применяем неприменённые миграции
    for _, migration := range migrations {
        if appliedVersions[migration.Version] {
            continue
        }
        
        log.Printf("Applying migration %d: %s", migration.Version, migration.Name)
        
        err := db.WithTransaction(context.Background(), func(ctx context.Context, tx *sqlx.Tx) error {
            // Выполняем миграцию
            if _, err := tx.Exec(migration.Up); err != nil {
                return fmt.Errorf("failed to execute migration %d: %w", migration.Version, err)
            }
            
            // Записываем в таблицу миграций
            _, err := tx.Exec("INSERT INTO schema_migrations (version, name) VALUES ($1, $2)", 
                migration.Version, migration.Name)
            if err != nil {
                return fmt.Errorf("failed to record migration %d: %w", migration.Version, err)
            }
            
            return nil
        })
        
        if err != nil {
            return err
        }
        
        log.Printf("Migration %d applied successfully", migration.Version)
    }
    
    return nil
}

// Пример использования
func main() {
    config := DatabaseConfig{
        Host:         "localhost",
        Port:         5432,
        User:         "postgres",
        Password:     "password",
        Database:     "myapp",
        SSLMode:      "disable",
        MaxOpenConns: 25,
        MaxIdleConns: 5,
        MaxLifetime:  time.Hour,
    }
    
    db, err := NewDatabase(config)
    if err != nil {
        log.Fatal("Failed to connect to database:", err)
    }
    defer db.Close()
    
    // Применяем миграции
    if err := RunMigrations(db); err != nil {
        log.Fatal("Failed to run migrations:", err)
    }
    
    // Создаем репозиторий
    logger := &ConsoleLogger{}
    userRepo := NewPostgresUserRepository(db)
    userService := NewUserService(userRepo, db, logger)
    
    ctx := context.Background()
    
    // Тестируем создание пользователя с заказом в транзакции
    user, order, err := userService.CreateUserWithOrder(ctx, "test@example.com", "Test User", 99.99)
    if err != nil {
        log.Fatal("Failed to create user with order:", err)
    }
    
    log.Printf("Created user %d with order %d", user.ID, order.ID)
    
    // Периодически логируем статистику пула соединений
    ticker := time.NewTicker(30 * time.Second)
    defer ticker.Stop()
    
    for range ticker.C {
        db.LogStats()
    }
}
```

**🎯 Практическая польза работы с БД:**
- **Connection pooling** - эффективное использование ресурсов
- **Транзакции** - обеспечение целостности данных
- **Prepared statements** - защита от SQL injection и производительность
- **Query builders** - динамическое построение сложных запросов
- **Migrations** - версионирование схемы базы данных

**💡 Лучшие практики:**
- Всегда используйте контексты для отмены долгих запросов
- Разделяйте read и write операции для масштабируемости
- Мониторьте статистику connection pool
- Используйте индексы для производительности

---

### Неделя 6: ORM и продвинутая работа с БД

#### 🔧 GORM - мощный ORM для Go

### GORM Architecture & Features

```
GORM Architecture Overview
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                        │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │   Models    │  │  Services   │  │ Controllers │          │
│  │             │  │             │  │             │          │
│  │ User        │  │ UserService │  │ UserHandler │          │
│  │ Order       │  │ OrderSvc    │  │ OrderHandler│          │
│  │ Product     │  │ ProductSvc  │  │ ProductHndlr│          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│         │               │               │                  │
│         ▼               ▼               ▼                  │
├─────────────────────────────────────────────────────────────┤
│                     GORM Layer                              │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                 GORM Features                           │ │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │ │
│  │  │  Auto   │  │  Hooks  │  │  Eager  │  │  Query  │   │ │
│  │  │Migration│  │& Events │  │Loading  │  │Builder  │   │ │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │ │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │ │
│  │  │Validaton│  │Relations│  │  Soft   │  │  JSON   │   │ │
│  │  │   Tags  │  │ (1:1,1:N)│  │ Delete  │  │  Tags   │   │ │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │ │
│  └─────────────────────────────────────────────────────────┘ │
│                           │                                 │
│                           ▼                                 │
├─────────────────────────────────────────────────────────────┤
│                   Database Driver                           │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ PostgreSQL  │  │    MySQL    │  │   SQLite    │          │
│  │   Driver    │  │   Driver    │  │   Driver    │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────────────────────┘

Model Relationships Visualization
┌─────────────────────────────────────────────────────────────┐
│                    Relationship Types                       │
├─────────────────────────────────────────────────────────────┤
│  One-to-One (1:1)                                          │
│  ┌─────────────┐         ┌─────────────┐                   │
│  │    User     │────────▶│   Profile   │                   │
│  │             │  has    │             │                   │
│  │ ID          │  one    │ UserID (FK) │                   │
│  │ Email       │         │ Avatar      │                   │
│  └─────────────┘         └─────────────┘                   │
├─────────────────────────────────────────────────────────────┤
│  One-to-Many (1:N)                                         │
│  ┌─────────────┐         ┌─────────────┐                   │
│  │    User     │────────▶│    Order    │                   │
│  │             │  has    │             │                   │
│  │ ID          │  many   │ UserID (FK) │                   │
│  │ Email       │         │ Amount      │                   │
│  └─────────────┘         └─────────────┘                   │
├─────────────────────────────────────────────────────────────┤
│  Many-to-Many (N:M)                                        │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐       │
│  │    User     │  │  user_roles  │  │    Role     │       │
│  │             │  │              │  │             │       │
│  │ ID          │  │ UserID (FK)  │  │ ID          │       │
│  │ Email       │  │ RoleID (FK)  │  │ Name        │       │
│  └─────────────┘  └──────────────┘  └─────────────┘       │
└─────────────────────────────────────────────────────────────┘
```

```go
package main

import (
    "context"
    "errors"
    "fmt"
    "log"
    "time"

    "gorm.io/driver/postgres"
    "gorm.io/gorm"
    "gorm.io/gorm/logger"
    "gorm.io/gorm/clause"
)

// GORM модели с тегами и отношениями
type User struct {
    ID        uint           `gorm:"primaryKey" json:"id"`
    CreatedAt time.Time      `json:"created_at"`
    UpdatedAt time.Time      `json:"updated_at"`
    DeletedAt gorm.DeletedAt `gorm:"index" json:"deleted_at,omitempty"`
    
    // Основные поля
    Email     string `gorm:"uniqueIndex;not null;size:255" json:"email" validate:"required,email"`
    Name      string `gorm:"not null;size:100" json:"name" validate:"required,min=2,max=100"`
    Password  string `gorm:"not null;size:255" json:"-"` // Скрыто в JSON
    IsActive  bool   `gorm:"default:true" json:"is_active"`
    LastLogin *time.Time `json:"last_login,omitempty"`
    
    // Отношения
    Profile Profile `gorm:"foreignKey:UserID" json:"profile,omitempty"`
    Orders  []Order `gorm:"foreignKey:UserID" json:"orders,omitempty"`
    Roles   []Role  `gorm:"many2many:user_roles;" json:"roles,omitempty"`
    
    // Вычисляемые поля (не сохраняются в БД)
    OrderCount int `gorm:"-" json:"order_count,omitempty"`
    FullName   string `gorm:"-" json:"full_name,omitempty"`
}

// Хуки GORM для автоматических действий
func (u *User) BeforeCreate(tx *gorm.DB) error {
    // Валидация перед созданием
    if u.Email == "" {
        return errors.New("email is required")
    }
    if len(u.Password) < 8 {
        return errors.New("password must be at least 8 characters")
    }
    return nil
}

func (u *User) AfterCreate(tx *gorm.DB) error {
    // Создаем профиль автоматически
    profile := Profile{
        UserID: u.ID,
        Avatar: "default-avatar.png",
    }
    return tx.Create(&profile).Error
}

func (u *User) AfterFind(tx *gorm.DB) error {
    // Вычисляемые поля после загрузки
    u.FullName = u.Name
    return nil
}

// Профиль пользователя (1:1 отношение)
type Profile struct {
    ID       uint   `gorm:"primaryKey" json:"id"`
    UserID   uint   `gorm:"uniqueIndex;not null" json:"user_id"`
    Avatar   string `gorm:"size:255" json:"avatar"`
    Bio      string `gorm:"type:text" json:"bio"`
    Website  string `gorm:"size:255" json:"website"`
    Location string `gorm:"size:100" json:"location"`
    
    // Настройки пользователя в JSON
    Settings ProfileSettings `gorm:"type:jsonb" json:"settings"`
    
    CreatedAt time.Time `json:"created_at"`
    UpdatedAt time.Time `json:"updated_at"`
    
    // Обратная связь
    User User `gorm:"foreignKey:UserID" json:"user,omitempty"`
}

type ProfileSettings struct {
    Theme         string `json:"theme"`
    Notifications bool   `json:"notifications"`
    Language      string `json:"language"`
}

// Заказ (1:N отношение с User)
type Order struct {
    ID          uint           `gorm:"primaryKey" json:"id"`
    CreatedAt   time.Time      `json:"created_at"`
    UpdatedAt   time.Time      `json:"updated_at"`
    DeletedAt   gorm.DeletedAt `gorm:"index" json:"deleted_at,omitempty"`
    
    UserID      uint           `gorm:"not null;index" json:"user_id"`
    TotalAmount float64        `gorm:"type:decimal(10,2);not null" json:"total_amount"`
    Status      OrderStatus    `gorm:"type:varchar(20);default:'pending'" json:"status"`
    
    // Отношения
    User  User        `gorm:"foreignKey:UserID" json:"user,omitempty"`
    Items []OrderItem `gorm:"foreignKey:OrderID" json:"items,omitempty"`
}

type OrderStatus string

const (
    OrderStatusPending    OrderStatus = "pending"
    OrderStatusProcessing OrderStatus = "processing"
    OrderStatusShipped    OrderStatus = "shipped"
    OrderStatusDelivered  OrderStatus = "delivered"
    OrderStatusCancelled  OrderStatus = "cancelled"
)

// Товар в заказе
type OrderItem struct {
    ID        uint    `gorm:"primaryKey" json:"id"`
    OrderID   uint    `gorm:"not null;index" json:"order_id"`
    ProductID uint    `gorm:"not null;index" json:"product_id"`
    Quantity  int     `gorm:"not null;check:quantity > 0" json:"quantity"`
    Price     float64 `gorm:"type:decimal(10,2);not null" json:"price"`
    
    // Отношения
    Order   Order   `gorm:"foreignKey:OrderID" json:"order,omitempty"`
    Product Product `gorm:"foreignKey:ProductID" json:"product,omitempty"`
}

// Товар
type Product struct {
    ID          uint           `gorm:"primaryKey" json:"id"`
    CreatedAt   time.Time      `json:"created_at"`
    UpdatedAt   time.Time      `json:"updated_at"`
    DeletedAt   gorm.DeletedAt `gorm:"index" json:"deleted_at,omitempty"`
    
    Name        string  `gorm:"not null;size:255" json:"name"`
    Description string  `gorm:"type:text" json:"description"`
    Price       float64 `gorm:"type:decimal(10,2);not null" json:"price"`
    SKU         string  `gorm:"uniqueIndex;size:100" json:"sku"`
    Stock       int     `gorm:"default:0" json:"stock"`
    
    // JSON поле для атрибутов
    Attributes ProductAttributes `gorm:"type:jsonb" json:"attributes"`
    
    // M:N отношение с категориями
    Categories []Category `gorm:"many2many:product_categories;" json:"categories,omitempty"`
}

type ProductAttributes struct {
    Color  string   `json:"color"`
    Size   string   `json:"size"`
    Weight float64  `json:"weight"`
    Tags   []string `json:"tags"`
}

// Категория
type Category struct {
    ID          uint           `gorm:"primaryKey" json:"id"`
    Name        string         `gorm:"not null;size:100" json:"name"`
    Slug        string         `gorm:"uniqueIndex;size:100" json:"slug"`
    Description string         `gorm:"type:text" json:"description"`
    CreatedAt   time.Time      `json:"created_at"`
    UpdatedAt   time.Time      `json:"updated_at"`
    DeletedAt   gorm.DeletedAt `gorm:"index" json:"deleted_at,omitempty"`
    
    // Самоссылающаяся связь для иерархии
    ParentID *uint      `gorm:"index" json:"parent_id,omitempty"`
    Parent   *Category  `gorm:"foreignKey:ParentID" json:"parent,omitempty"`
    Children []Category `gorm:"foreignKey:ParentID" json:"children,omitempty"`
    
    // M:N отношение с продуктами
    Products []Product `gorm:"many2many:product_categories;" json:"products,omitempty"`
}

// Роль (M:N отношение с User)
type Role struct {
    ID          uint           `gorm:"primaryKey" json:"id"`
    Name        string         `gorm:"uniqueIndex;not null;size:50" json:"name"`
    Description string         `gorm:"size:255" json:"description"`
    CreatedAt   time.Time      `json:"created_at"`
    UpdatedAt   time.Time      `json:"updated_at"`
    
    // M:N отношение с пользователями
    Users []User `gorm:"many2many:user_roles;" json:"users,omitempty"`
}

// GORM Database wrapper
type GormDB struct {
    db *gorm.DB
}

func NewGormDB(dsn string) (*GormDB, error) {
    // Настройка логгера GORM
    gormLogger := logger.New(
        log.New(log.Writer(), "\r\n", log.LstdFlags),
        logger.Config{
            SlowThreshold:             200 * time.Millisecond,
            LogLevel:                  logger.Info,
            IgnoreRecordNotFoundError: true,
            Colorful:                  true,
        },
    )
    
    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{
        Logger: gormLogger,
        NowFunc: func() time.Time {
            return time.Now().UTC()
        },
    })
    
    if err != nil {
        return nil, fmt.Errorf("failed to connect to database: %w", err)
    }
    
    // Настройка connection pool
    sqlDB, err := db.DB()
    if err != nil {
        return nil, fmt.Errorf("failed to get database instance: %w", err)
    }
    
    sqlDB.SetMaxOpenConns(25)
    sqlDB.SetMaxIdleConns(5)
    sqlDB.SetConnMaxLifetime(time.Hour)
    
    return &GormDB{db: db}, nil
}

func (gdb *GormDB) AutoMigrate() error {
    return gdb.db.AutoMigrate(
        &User{},
        &Profile{},
        &Order{},
        &OrderItem{},
        &Product{},
        &Category{},
        &Role{},
    )
}

// Repository с GORM
type GormUserRepository struct {
    db *GormDB
}

func NewGormUserRepository(db *GormDB) *GormUserRepository {
    return &GormUserRepository{db: db}
}

func (r *GormUserRepository) Create(ctx context.Context, user *User) error {
    result := r.db.db.WithContext(ctx).Create(user)
    if result.Error != nil {
        return fmt.Errorf("failed to create user: %w", result.Error)
    }
    return nil
}

func (r *GormUserRepository) GetByID(ctx context.Context, id uint) (*User, error) {
    var user User
    result := r.db.db.WithContext(ctx).
        Preload("Profile").           // Eager loading профиля
        Preload("Orders").           // Eager loading заказов
        Preload("Roles").            // Eager loading ролей
        First(&user, id)
    
    if result.Error != nil {
        if errors.Is(result.Error, gorm.ErrRecordNotFound) {
            return nil, ErrUserNotFound
        }
        return nil, fmt.Errorf("failed to get user: %w", result.Error)
    }
    
    return &user, nil
}

func (r *GormUserRepository) GetByEmail(ctx context.Context, email string) (*User, error) {
    var user User
    result := r.db.db.WithContext(ctx).
        Where("email = ?", email).
        Preload("Profile").
        Preload("Roles").
        First(&user)
    
    if result.Error != nil {
        if errors.Is(result.Error, gorm.ErrRecordNotFound) {
            return nil, ErrUserNotFound
        }
        return nil, fmt.Errorf("failed to get user by email: %w", result.Error)
    }
    
    return &user, nil
}

func (r *GormUserRepository) Update(ctx context.Context, user *User) error {
    result := r.db.db.WithContext(ctx).Save(user)
    if result.Error != nil {
        return fmt.Errorf("failed to update user: %w", result.Error)
    }
    
    if result.RowsAffected == 0 {
        return ErrUserNotFound
    }
    
    return nil
}

func (r *GormUserRepository) Delete(ctx context.Context, id uint) error {
    result := r.db.db.WithContext(ctx).Delete(&User{}, id)
    if result.Error != nil {
        return fmt.Errorf("failed to delete user: %w", result.Error)
    }
    
    if result.RowsAffected == 0 {
        return ErrUserNotFound
    }
    
    return nil
}

// Сложные запросы с GORM
func (r *GormUserRepository) SearchUsers(ctx context.Context, query UserSearchQuery) ([]User, int64, error) {
    var users []User
    var total int64
    
    db := r.db.db.WithContext(ctx).Model(&User{})
    
    // Фильтры
    if query.Search != "" {
        db = db.Where("name ILIKE ? OR email ILIKE ?", "%"+query.Search+"%", "%"+query.Search+"%")
    }
    
    if query.IsActive != nil {
        db = db.Where("is_active = ?", *query.IsActive)
    }
    
    if query.Role != "" {
        db = db.Joins("JOIN user_roles ON users.id = user_roles.user_id").
            Joins("JOIN roles ON user_roles.role_id = roles.id").
            Where("roles.name = ?", query.Role)
    }
    
    if !query.CreatedAfter.IsZero() {
        db = db.Where("created_at > ?", query.CreatedAfter)
    }
    
    // Подсчет общего количества
    if err := db.Count(&total).Error; err != nil {
        return nil, 0, fmt.Errorf("failed to count users: %w", err)
    }
    
    // Получение данных с пагинацией
    err := db.Preload("Profile").
        Preload("Roles").
        Order("created_at DESC").
        Limit(query.Limit).
        Offset(query.Offset).
        Find(&users).Error
    
    if err != nil {
        return nil, 0, fmt.Errorf("failed to search users: %w", err)
    }
    
    return users, total, nil
}

type UserSearchQuery struct {
    Search       string
    IsActive     *bool
    Role         string
    CreatedAfter time.Time
    Limit        int
    Offset       int
}

// Работа с отношениями
func (r *GormUserRepository) GetUserWithOrders(ctx context.Context, userID uint, orderStatus OrderStatus) (*User, error) {
    var user User
    
    query := r.db.db.WithContext(ctx).Preload("Profile")
    
    if orderStatus != "" {
        // Фильтруем заказы по статусу
        query = query.Preload("Orders", "status = ?", orderStatus)
    } else {
        query = query.Preload("Orders")
    }
    
    // Загружаем элементы заказов с товарами
    query = query.Preload("Orders.Items").Preload("Orders.Items.Product")
    
    result := query.First(&user, userID)
    if result.Error != nil {
        if errors.Is(result.Error, gorm.ErrRecordNotFound) {
            return nil, ErrUserNotFound
        }
        return nil, fmt.Errorf("failed to get user with orders: %w", result.Error)
    }
    
    return &user, nil
}

// Агрегация и статистика
func (r *GormUserRepository) GetUserStatistics(ctx context.Context, userID uint) (*UserStatistics, error) {
    var stats UserStatistics
    
    // Подсчет заказов
    err := r.db.db.WithContext(ctx).
        Model(&Order{}).
        Where("user_id = ?", userID).
        Count(&stats.TotalOrders).Error
    if err != nil {
        return nil, fmt.Errorf("failed to count orders: %w", err)
    }
    
    // Сумма заказов
    err = r.db.db.WithContext(ctx).
        Model(&Order{}).
        Where("user_id = ? AND status = ?", userID, OrderStatusDelivered).
        Select("COALESCE(SUM(total_amount), 0)").
        Scan(&stats.TotalSpent).Error
    if err != nil {
        return nil, fmt.Errorf("failed to sum order amounts: %w", err)
    }
    
    // Последний заказ
    var lastOrder Order
    err = r.db.db.WithContext(ctx).
        Where("user_id = ?", userID).
        Order("created_at DESC").
        First(&lastOrder).Error
    if err != nil && !errors.Is(err, gorm.ErrRecordNotFound) {
        return nil, fmt.Errorf("failed to get last order: %w", err)
    }
    if err == nil {
        stats.LastOrderDate = &lastOrder.CreatedAt
    }
    
    return &stats, nil
}

type UserStatistics struct {
    TotalOrders   int64      `json:"total_orders"`
    TotalSpent    float64    `json:"total_spent"`
    LastOrderDate *time.Time `json:"last_order_date"`
}

// Bulk операции
func (r *GormUserRepository) BulkUpdateActiveStatus(ctx context.Context, userIDs []uint, isActive bool) error {
    result := r.db.db.WithContext(ctx).
        Model(&User{}).
        Where("id IN ?", userIDs).
        Update("is_active", isActive)
    
    if result.Error != nil {
        return fmt.Errorf("failed to bulk update users: %w", result.Error)
    }
    
    return nil
}

func (r *GormUserRepository) BulkCreate(ctx context.Context, users []User) error {
    // Батчевая вставка для производительности
    batchSize := 100
    
    for i := 0; i < len(users); i += batchSize {
        end := i + batchSize
        if end > len(users) {
            end = len(users)
        }
        
        batch := users[i:end]
        if err := r.db.db.WithContext(ctx).Create(&batch).Error; err != nil {
            return fmt.Errorf("failed to bulk create users batch %d-%d: %w", i, end, err)
        }
    }
    
    return nil
}

// Транзакции с GORM
func (r *GormUserRepository) TransferUserData(ctx context.Context, fromUserID, toUserID uint) error {
    return r.db.db.WithContext(ctx).Transaction(func(tx *gorm.DB) error {
        // Проверяем существование пользователей
        var fromUser, toUser User
        if err := tx.First(&fromUser, fromUserID).Error; err != nil {
            return fmt.Errorf("from user not found: %w", err)
        }
        if err := tx.First(&toUser, toUserID).Error; err != nil {
            return fmt.Errorf("to user not found: %w", err)
        }
        
        // Переносим заказы
        if err := tx.Model(&Order{}).
            Where("user_id = ?", fromUserID).
            Update("user_id", toUserID).Error; err != nil {
            return fmt.Errorf("failed to transfer orders: %w", err)
        }
        
        // Деактивируем старого пользователя
        if err := tx.Model(&fromUser).Update("is_active", false).Error; err != nil {
            return fmt.Errorf("failed to deactivate user: %w", err)
        }
        
        return nil
    })
}

// Raw SQL запросы для сложной аналитики
func (r *GormUserRepository) GetTopCustomers(ctx context.Context, limit int) ([]CustomerStats, error) {
    var customers []CustomerStats
    
    query := `
        SELECT 
            u.id,
            u.name,
            u.email,
            COUNT(o.id) as order_count,
            COALESCE(SUM(o.total_amount), 0) as total_spent,
            AVG(o.total_amount) as avg_order_value
        FROM users u
        LEFT JOIN orders o ON u.id = o.user_id AND o.deleted_at IS NULL
        WHERE u.deleted_at IS NULL
        GROUP BY u.id, u.name, u.email
        HAVING COUNT(o.id) > 0
        ORDER BY total_spent DESC
        LIMIT ?
    `
    
    err := r.db.db.WithContext(ctx).Raw(query, limit).Scan(&customers).Error
    if err != nil {
        return nil, fmt.Errorf("failed to get top customers: %w", err)
    }
    
    return customers, nil
}

type CustomerStats struct {
    ID              uint    `json:"id"`
    Name            string  `json:"name"`
    Email           string  `json:"email"`
    OrderCount      int     `json:"order_count"`
    TotalSpent      float64 `json:"total_spent"`
    AvgOrderValue   float64 `json:"avg_order_value"`
}

// Scopes для переиспользования запросов
func (r *GormUserRepository) ActiveUsers() func(db *gorm.DB) *gorm.DB {
    return func(db *gorm.DB) *gorm.DB {
        return db.Where("is_active = ?", true)
    }
}

func (r *GormUserRepository) WithRole(role string) func(db *gorm.DB) *gorm.DB {
    return func(db *gorm.DB) *gorm.DB {
        return db.Joins("JOIN user_roles ON users.id = user_roles.user_id").
            Joins("JOIN roles ON user_roles.role_id = roles.id").
            Where("roles.name = ?", role)
    }
}

func (r *GormUserRepository) CreatedInRange(start, end time.Time) func(db *gorm.DB) *gorm.DB {
    return func(db *gorm.DB) *gorm.DB {
        return db.Where("created_at BETWEEN ? AND ?", start, end)
    }
}

// Использование scopes
func (r *GormUserRepository) GetActiveAdmins(ctx context.Context) ([]User, error) {
    var users []User
    
    err := r.db.db.WithContext(ctx).
        Scopes(r.ActiveUsers(), r.WithRole("admin")).
        Preload("Profile").
        Preload("Roles").
        Find(&users).Error
    
    if err != nil {
        return nil, fmt.Errorf("failed to get active admins: %w", err)
    }
    
    return users, nil
}

// Уведомления с хуками
func (u *User) AfterUpdate(tx *gorm.DB) error {
    // Отправляем уведомление при изменении статуса
    if tx.Statement.Changed("is_active") {
        // Здесь была бы отправка уведомления
        log.Printf("User %d status changed to active: %v", u.ID, u.IsActive)
    }
    return nil
}

// Пример использования GORM
func demonstrateGORM() {
    dsn := "host=localhost user=postgres password=password dbname=testdb port=5432 sslmode=disable"
    
    db, err := NewGormDB(dsn)
    if err != nil {
        log.Fatal("Failed to connect to database:", err)
    }
    
    // Автомиграция
    if err := db.AutoMigrate(); err != nil {
        log.Fatal("Failed to migrate:", err)
    }
    
    repo := NewGormUserRepository(db)
    ctx := context.Background()
    
    // Создание пользователя с ролями
    user := &User{
        Email:    "admin@example.com",
        Name:     "Admin User",
        Password: "hashedpassword",
        IsActive: true,
        Roles: []Role{
            {Name: "admin", Description: "Administrator"},
            {Name: "user", Description: "Regular user"},
        },
    }
    
    if err := repo.Create(ctx, user); err != nil {
        log.Fatal("Failed to create user:", err)
    }
    
    log.Printf("Created user with ID: %d", user.ID)
    
    // Поиск пользователей
    query := UserSearchQuery{
        Search:   "admin",
        IsActive: &[]bool{true}[0],
        Limit:    10,
        Offset:   0,
    }
    
    users, total, err := repo.SearchUsers(ctx, query)
    if err != nil {
        log.Fatal("Failed to search users:", err)
    }
    
    log.Printf("Found %d users (total: %d)", len(users), total)
    
    // Статистика пользователя
    stats, err := repo.GetUserStatistics(ctx, user.ID)
    if err != nil {
        log.Fatal("Failed to get user statistics:", err)
    }
    
    log.Printf("User statistics: %+v", stats)
    
    // Топ клиенты
    topCustomers, err := repo.GetTopCustomers(ctx, 5)
    if err != nil {
        log.Fatal("Failed to get top customers:", err)
    }
    
    log.Printf("Top customers: %+v", topCustomers)
}
```

**🎯 Практическая польза GORM:**
- **Автоматические миграции** - схема БД синхронизируется с моделями
- **Отношения** - простая работа с foreign keys и joins
- **Хуки** - автоматические действия при изменении данных
- **Eager loading** - избежание N+1 проблемы
- **Scopes** - переиспользование частей запросов

**💡 Продвинутые возможности:**
- JSON поля для гибких атрибутов
- Soft delete для сохранения истории
- Bulk операции для производительности
- Raw SQL для сложной аналитики
- Transaction support для целостности данных

---

### Неделя 7: Архитектурные паттерны

#### 🏛️ Clean Architecture в Go

### Clean Architecture Layers

```
Clean Architecture Visualization
┌─────────────────────────────────────────────────────────────┐
│                    External Interfaces                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │     Web     │  │     CLI     │  │     API     │          │
│  │   Server    │  │   Commands  │  │   Gateway   │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│         │               │               │                  │
│         ▼               ▼               ▼                  │
├─────────────────────────────────────────────────────────────┤
│                Interface Adapters                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ Controllers │  │ Presenters  │  │  Gateways   │          │
│  │             │  │             │  │             │          │
│  │ HTTP        │  │ JSON/XML    │  │ External    │          │
│  │ Handlers    │  │ Formatters  │  │ APIs        │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│         │               │               │                  │
│         ▼               ▼               ▼                  │
├─────────────────────────────────────────────────────────────┤
│                Application Business Rules                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │   Use Cases │  │ Interactors │  │  Services   │          │
│  │             │  │             │  │             │          │
│  │ Application │  │ Workflow    │  │ Application │          │
│  │ Logic       │  │ Orchestration│ │ Services    │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│         │               │               │                  │
│         ▼               ▼               ▼                  │
├─────────────────────────────────────────────────────────────┤
│               Enterprise Business Rules                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │  Entities   │  │   Domain    │  │  Business   │          │
│  │             │  │   Models    │  │    Rules    │          │
│  │ Core        │  │             │  │             │          │
│  │ Business    │  │ Domain      │  │ Policies    │          │
│  │ Logic       │  │ Logic       │  │             │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────────────────────┘

Dependency Direction
┌─────────────────────────────────────────────────────────────┐
│                     Dependency Rule                         │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐                                            │
│  │   Outer     │ ────────┐                                  │
│  │   Layers    │         │                                  │
│  └─────────────┘         │                                  │
│                          ▼                                  │
│  ┌─────────────┐    ┌─────────────┐                         │
│  │ Interface   │ ──▶│ Application │                         │
│  │ Adapters    │    │   Layer     │                         │
│  └─────────────┘    └─────────────┘                         │
│                          │                                  │
│                          ▼                                  │
│                    ┌─────────────┐                          │
│                    │  Enterprise │                          │
│                    │   Business  │                          │
│                    │    Rules    │                          │
│                    └─────────────┘                          │
│                                                             │
│  Dependencies point INWARD only!                           │
│  Inner layers never depend on outer layers                 │
└─────────────────────────────────────────────────────────────┘
```

```go
package main

import (
    "context"
    "errors"
    "fmt"
    "time"
)

// ==================== ENTERPRISE BUSINESS RULES (DOMAIN) ====================

// Domain Entities - самый внутренний слой
type User struct {
    ID        UserID    `json:"id"`
    Email     Email     `json:"email"`
    Name      string    `json:"name"`
    Password  Password  `json:"-"`
    IsActive  bool      `json:"is_active"`
    CreatedAt time.Time `json:"created_at"`
    UpdatedAt time.Time `json:"updated_at"`
}

// Value Objects для type safety
type UserID uint64
type Email string
type Password string

// Domain validation в value objects
func NewEmail(email string) (Email, error) {
    if email == "" {
        return "", errors.New("email cannot be empty")
    }
    // Здесь была бы полная валидация email
    if len(email) < 5 || !contains(email, "@") {
        return "", errors.New("invalid email format")
    }
    return Email(email), nil
}

func NewPassword(password string) (Password, error) {
    if len(password) < 8 {
        return "", errors.New("password must be at least 8 characters")
    }
    // Здесь было бы хеширование пароля
    return Password("hashed_" + password), nil
}

func (e Email) String() string {
    return string(e)
}

func (p Password) String() string {
    return string(p)
}

// Domain methods - бизнес-логика в entities
func (u *User) Activate() error {
    if u.IsActive {
        return errors.New("user is already active")
    }
    u.IsActive = true
    u.UpdatedAt = time.Now()
    return nil
}

func (u *User) Deactivate() error {
    if !u.IsActive {
        return errors.New("user is already inactive")
    }
    u.IsActive = false
    u.UpdatedAt = time.Now()
    return nil
}

func (u *User) ChangeEmail(newEmail Email) error {
    if u.Email == newEmail {
        return errors.New("new email is the same as current")
    }
    u.Email = newEmail
    u.UpdatedAt = time.Now()
    return nil
}

// Domain specifications для бизнес-правил
type UserSpecification interface {
    IsSatisfiedBy(user *User) bool
}

type ActiveUserSpecification struct{}

func (s ActiveUserSpecification) IsSatisfiedBy(user *User) bool {
    return user.IsActive
}

type RecentUserSpecification struct {
    Days int
}

func (s RecentUserSpecification) IsSatisfiedBy(user *User) bool {
    return time.Since(user.CreatedAt) <= time.Duration(s.Days)*24*time.Hour
}

// Domain Events
type DomainEvent interface {
    EventName() string
    OccurredAt() time.Time
}

type UserActivatedEvent struct {
    UserID    UserID    `json:"user_id"`
    Email     Email     `json:"email"`
    Timestamp time.Time `json:"timestamp"`
}

func (e UserActivatedEvent) EventName() string {
    return "user.activated"
}

func (e UserActivatedEvent) OccurredAt() time.Time {
    return e.Timestamp
}

type UserDeactivatedEvent struct {
    UserID    UserID    `json:"user_id"`
    Email     Email     `json:"email"`
    Timestamp time.Time `json:"timestamp"`
}

func (e UserDeactivatedEvent) EventName() string {
    return "user.deactivated"
}

func (e UserDeactivatedEvent) OccurredAt() time.Time {
    return e.Timestamp
}

// ==================== APPLICATION BUSINESS RULES ====================

// Repository interfaces (defined in domain, implemented in infrastructure)
type UserRepository interface {
    Save(ctx context.Context, user *User) error
    FindByID(ctx context.Context, id UserID) (*User, error)
    FindByEmail(ctx context.Context, email Email) (*User, error)
    FindAll(ctx context.Context, spec UserSpecification) ([]*User, error)
    Delete(ctx context.Context, id UserID) error
}

// External service interfaces
type EmailService interface {
    SendWelcomeEmail(ctx context.Context, email Email, name string) error
    SendActivationEmail(ctx context.Context, email Email, name string) error
}

type EventPublisher interface {
    Publish(ctx context.Context, event DomainEvent) error
}

// Use Cases - application logic
type CreateUserUseCase struct {
    userRepo      UserRepository
    emailService  EmailService
    eventPublisher EventPublisher
}

func NewCreateUserUseCase(
    userRepo UserRepository,
    emailService EmailService,
    eventPublisher EventPublisher,
) *CreateUserUseCase {
    return &CreateUserUseCase{
        userRepo:       userRepo,
        emailService:   emailService,
        eventPublisher: eventPublisher,
    }
}

type CreateUserRequest struct {
    Email    string `json:"email"`
    Name     string `json:"name"`
    Password string `json:"password"`
}

type CreateUserResponse struct {
    UserID UserID `json:"user_id"`
    Email  Email  `json:"email"`
    Name   string `json:"name"`
}

func (uc *CreateUserUseCase) Execute(ctx context.Context, req CreateUserRequest) (*CreateUserResponse, error) {
    // Валидация входных данных
    email, err := NewEmail(req.Email)
    if err != nil {
        return nil, fmt.Errorf("invalid email: %w", err)
    }
    
    password, err := NewPassword(req.Password)
    if err != nil {
        return nil, fmt.Errorf("invalid password: %w", err)
    }
    
    if req.Name == "" {
        return nil, errors.New("name is required")
    }
    
    // Проверка уникальности email
    existingUser, err := uc.userRepo.FindByEmail(ctx, email)
    if err == nil && existingUser != nil {
        return nil, errors.New("user with this email already exists")
    }
    
    // Создание пользователя
    user := &User{
        ID:        UserID(generateID()), // Здесь был бы генератор ID
        Email:     email,
        Name:      req.Name,
        Password:  password,
        IsActive:  false, // Требуется активация
        CreatedAt: time.Now(),
        UpdatedAt: time.Now(),
    }
    
    // Сохранение
    if err := uc.userRepo.Save(ctx, user); err != nil {
        return nil, fmt.Errorf("failed to save user: %w", err)
    }
    
    // Отправка welcome email
    if err := uc.emailService.SendWelcomeEmail(ctx, user.Email, user.Name); err != nil {
        // Логируем ошибку, но не останавливаем процесс
        fmt.Printf("Failed to send welcome email: %v\n", err)
    }
    
    return &CreateUserResponse{
        UserID: user.ID,
        Email:  user.Email,
        Name:   user.Name,
    }, nil
}

// Activate User Use Case
type ActivateUserUseCase struct {
    userRepo       UserRepository
    emailService   EmailService
    eventPublisher EventPublisher
}

func NewActivateUserUseCase(
    userRepo UserRepository,
    emailService EmailService,
    eventPublisher EventPublisher,
) *ActivateUserUseCase {
    return &ActivateUserUseCase{
        userRepo:       userRepo,
        emailService:   emailService,
        eventPublisher: eventPublisher,
    }
}

func (uc *ActivateUserUseCase) Execute(ctx context.Context, userID UserID) error {
    // Получаем пользователя
    user, err := uc.userRepo.FindByID(ctx, userID)
    if err != nil {
        return fmt.Errorf("user not found: %w", err)
    }
    
    // Применяем бизнес-логику
    if err := user.Activate(); err != nil {
        return fmt.Errorf("failed to activate user: %w", err)
    }
    
    // Сохраняем изменения
    if err := uc.userRepo.Save(ctx, user); err != nil {
        return fmt.Errorf("failed to save user: %w", err)
    }
    
    // Публикуем domain event
    event := UserActivatedEvent{
        UserID:    user.ID,
        Email:     user.Email,
        Timestamp: time.Now(),
    }
    
    if err := uc.eventPublisher.Publish(ctx, event); err != nil {
        // Логируем ошибку, но не возвращаем ее
        fmt.Printf("Failed to publish user activated event: %v\n", err)
    }
    
    // Отправляем уведомление
    if err := uc.emailService.SendActivationEmail(ctx, user.Email, user.Name); err != nil {
        fmt.Printf("Failed to send activation email: %v\n", err)
    }
    
    return nil
}

// Get User Use Case
type GetUserUseCase struct {
    userRepo UserRepository
}

func NewGetUserUseCase(userRepo UserRepository) *GetUserUseCase {
    return &GetUserUseCase{userRepo: userRepo}
}

func (uc *GetUserUseCase) Execute(ctx context.Context, userID UserID) (*User, error) {
    user, err := uc.userRepo.FindByID(ctx, userID)
    if err != nil {
        return nil, fmt.Errorf("user not found: %w", err)
    }
    
    return user, nil
}

// List Users Use Case with specifications
type ListUsersUseCase struct {
    userRepo UserRepository
}

func NewListUsersUseCase(userRepo UserRepository) *ListUsersUseCase {
    return &ListUsersUseCase{userRepo: userRepo}
}

type ListUsersQuery struct {
    OnlyActive   bool
    RecentDays   int
}

func (uc *ListUsersUseCase) Execute(ctx context.Context, query ListUsersQuery) ([]*User, error) {
    var spec UserSpecification
    
    if query.OnlyActive && query.RecentDays > 0 {
        // Комбинируем спецификации
        spec = &AndSpecification{
            Left:  &ActiveUserSpecification{},
            Right: &RecentUserSpecification{Days: query.RecentDays},
        }
    } else if query.OnlyActive {
        spec = &ActiveUserSpecification{}
    } else if query.RecentDays > 0 {
        spec = &RecentUserSpecification{Days: query.RecentDays}
    }
    
    users, err := uc.userRepo.FindAll(ctx, spec)
    if err != nil {
        return nil, fmt.Errorf("failed to find users: %w", err)
    }
    
    return users, nil
}

// Composite specification pattern
type AndSpecification struct {
    Left, Right UserSpecification
}

func (s *AndSpecification) IsSatisfiedBy(user *User) bool {
    return s.Left.IsSatisfiedBy(user) && s.Right.IsSatisfiedBy(user)
}

type OrSpecification struct {
    Left, Right UserSpecification
}

func (s *OrSpecification) IsSatisfiedBy(user *User) bool {
    return s.Left.IsSatisfiedBy(user) || s.Right.IsSatisfiedBy(user)
}

// ==================== INTERFACE ADAPTERS ====================

// HTTP Controllers
type UserController struct {
    createUserUC   *CreateUserUseCase
    activateUserUC *ActivateUserUseCase
    getUserUC      *GetUserUseCase
    listUsersUC    *ListUsersUseCase
}

func NewUserController(
    createUserUC *CreateUserUseCase,
    activateUserUC *ActivateUserUseCase,
    getUserUC *GetUserUseCase,
    listUsersUC *ListUsersUseCase,
) *UserController {
    return &UserController{
        createUserUC:   createUserUC,
        activateUserUC: activateUserUC,
        getUserUC:      getUserUC,
        listUsersUC:    listUsersUC,
    }
}

// HTTP DTOs (отличаются от domain entities)
type CreateUserHTTPRequest struct {
    Email    string `json:"email"`
    Name     string `json:"name"`
    Password string `json:"password"`
}

type UserHTTPResponse struct {
    ID        uint64    `json:"id"`
    Email     string    `json:"email"`
    Name      string    `json:"name"`
    IsActive  bool      `json:"is_active"`
    CreatedAt time.Time `json:"created_at"`
}

func (c *UserController) CreateUser(ctx context.Context, httpReq CreateUserHTTPRequest) (*UserHTTPResponse, error) {
    // Конвертируем HTTP request в use case request
    req := CreateUserRequest{
        Email:    httpReq.Email,
        Name:     httpReq.Name,
        Password: httpReq.Password,
    }
    
    // Выполняем use case
    resp, err := c.createUserUC.Execute(ctx, req)
    if err != nil {
        return nil, err
    }
    
    // Конвертируем response в HTTP response
    return &UserHTTPResponse{
        ID:        uint64(resp.UserID),
        Email:     resp.Email.String(),
        Name:      resp.Name,
        IsActive:  false,
        CreatedAt: time.Now(),
    }, nil
}

func (c *UserController) ActivateUser(ctx context.Context, userIDStr string) error {
    // Парсим ID из строки
    userID, err := parseUserID(userIDStr)
    if err != nil {
        return fmt.Errorf("invalid user ID: %w", err)
    }
    
    return c.activateUserUC.Execute(ctx, userID)
}

func (c *UserController) GetUser(ctx context.Context, userIDStr string) (*UserHTTPResponse, error) {
    userID, err := parseUserID(userIDStr)
    if err != nil {
        return nil, fmt.Errorf("invalid user ID: %w", err)
    }
    
    user, err := c.getUserUC.Execute(ctx, userID)
    if err != nil {
        return nil, err
    }
    
    return &UserHTTPResponse{
        ID:        uint64(user.ID),
        Email:     user.Email.String(),
        Name:      user.Name,
        IsActive:  user.IsActive,
        CreatedAt: user.CreatedAt,
    }, nil
}

// Repository Implementation (Infrastructure Layer)
type PostgresUserRepository struct {
    db Database // database interface
}

func NewPostgresUserRepository(db Database) UserRepository {
    return &PostgresUserRepository{db: db}
}

func (r *PostgresUserRepository) Save(ctx context.Context, user *User) error {
    query := `
        INSERT INTO users (id, email, name, password_hash, is_active, created_at, updated_at)
        VALUES ($1, $2, $3, $4, $5, $6, $7)
        ON CONFLICT (id) DO UPDATE SET
            email = EXCLUDED.email,
            name = EXCLUDED.name,
            password_hash = EXCLUDED.password_hash,
            is_active = EXCLUDED.is_active,
            updated_at = EXCLUDED.updated_at
    `
    
    _, err := r.db.ExecContext(ctx, query,
        uint64(user.ID),
        user.Email.String(),
        user.Name,
        user.Password.String(),
        user.IsActive,
        user.CreatedAt,
        user.UpdatedAt,
    )
    
    if err != nil {
        return fmt.Errorf("failed to save user: %w", err)
    }
    
    return nil
}

func (r *PostgresUserRepository) FindByID(ctx context.Context, id UserID) (*User, error) {
    query := `
        SELECT id, email, name, password_hash, is_active, created_at, updated_at
        FROM users
        WHERE id = $1
    `
    
    var user User
    var emailStr, passwordStr string
    var userIDUint uint64
    
    err := r.db.QueryRowContext(ctx, query, uint64(id)).Scan(
        &userIDUint,
        &emailStr,
        &user.Name,
        &passwordStr,
        &user.IsActive,
        &user.CreatedAt,
        &user.UpdatedAt,
    )
    
    if err != nil {
        return nil, fmt.Errorf("user not found: %w", err)
    }
    
    // Конвертируем в domain types
    user.ID = UserID(userIDUint)
    user.Email = Email(emailStr)
    user.Password = Password(passwordStr)
    
    return &user, nil
}

func (r *PostgresUserRepository) FindByEmail(ctx context.Context, email Email) (*User, error) {
    query := `
        SELECT id, email, name, password_hash, is_active, created_at, updated_at
        FROM users
        WHERE email = $1
    `
    
    var user User
    var emailStr, passwordStr string
    var userIDUint uint64
    
    err := r.db.QueryRowContext(ctx, query, email.String()).Scan(
        &userIDUint,
        &emailStr,
        &user.Name,
        &passwordStr,
        &user.IsActive,
        &user.CreatedAt,
        &user.UpdatedAt,
    )
    
    if err != nil {
        return nil, fmt.Errorf("user not found: %w", err)
    }
    
    user.ID = UserID(userIDUint)
    user.Email = Email(emailStr)
    user.Password = Password(passwordStr)
    
    return &user, nil
}

func (r *PostgresUserRepository) FindAll(ctx context.Context, spec UserSpecification) ([]*User, error) {
    query := `
        SELECT id, email, name, password_hash, is_active, created_at, updated_at
        FROM users
        ORDER BY created_at DESC
    `
    
    rows, err := r.db.QueryContext(ctx, query)
    if err != nil {
        return nil, fmt.Errorf("failed to query users: %w", err)
    }
    defer rows.Close()
    
    var users []*User
    
    for rows.Next() {
        var user User
        var emailStr, passwordStr string
        var userIDUint uint64
        
        err := rows.Scan(
            &userIDUint,
            &emailStr,
            &user.Name,
            &passwordStr,
            &user.IsActive,
            &user.CreatedAt,
            &user.UpdatedAt,
        )
        
        if err != nil {
            return nil, fmt.Errorf("failed to scan user: %w", err)
        }
        
        user.ID = UserID(userIDUint)
        user.Email = Email(emailStr)
        user.Password = Password(passwordStr)
        
        // Применяем спецификацию
        if spec == nil || spec.IsSatisfiedBy(&user) {
            users = append(users, &user)
        }
    }
    
    return users, nil
}

func (r *PostgresUserRepository) Delete(ctx context.Context, id UserID) error {
    query := `DELETE FROM users WHERE id = $1`
    
    result, err := r.db.ExecContext(ctx, query, uint64(id))
    if err != nil {
        return fmt.Errorf("failed to delete user: %w", err)
    }
    
    rowsAffected, err := result.RowsAffected()
    if err != nil {
        return fmt.Errorf("failed to get rows affected: %w", err)
    }
    
    if rowsAffected == 0 {
        return errors.New("user not found")
    }
    
    return nil
}

// Email Service Implementation
type SMTPEmailService struct {
    host     string
    port     int
    username string
    password string
}

func NewSMTPEmailService(host string, port int, username, password string) EmailService {
    return &SMTPEmailService{
        host:     host,
        port:     port,
        username: username,
        password: password,
    }
}

func (s *SMTPEmailService) SendWelcomeEmail(ctx context.Context, email Email, name string) error {
    // Здесь была бы реальная отправка email
    fmt.Printf("Sending welcome email to %s (name: %s)\n", email.String(), name)
    return nil
}

func (s *SMTPEmailService) SendActivationEmail(ctx context.Context, email Email, name string) error {
    fmt.Printf("Sending activation email to %s (name: %s)\n", email.String(), name)
    return nil
}

// Event Publisher Implementation
type InMemoryEventPublisher struct {
    handlers map[string][]func(DomainEvent)
}

func NewInMemoryEventPublisher() EventPublisher {
    return &InMemoryEventPublisher{
        handlers: make(map[string][]func(DomainEvent)),
    }
}

func (p *InMemoryEventPublisher) Publish(ctx context.Context, event DomainEvent) error {
    eventName := event.EventName()
    
    handlers, exists := p.handlers[eventName]
    if !exists {
        return nil
    }
    
    for _, handler := range handlers {
        go handler(event) // Асинхронная обработка
    }
    
    return nil
}

func (p *InMemoryEventPublisher) Subscribe(eventName string, handler func(DomainEvent)) {
    p.handlers[eventName] = append(p.handlers[eventName], handler)
}

// ==================== MAIN (COMPOSITION ROOT) ====================

// Dependency Injection Container
type Container struct {
    userRepo       UserRepository
    emailService   EmailService
    eventPublisher EventPublisher
    
    createUserUC   *CreateUserUseCase
    activateUserUC *ActivateUserUseCase
    getUserUC      *GetUserUseCase
    listUsersUC    *ListUsersUseCase
    
    userController *UserController
}

func NewContainer(db Database) *Container {
    // Infrastructure
    userRepo := NewPostgresUserRepository(db)
    emailService := NewSMTPEmailService("smtp.example.com", 587, "user", "pass")
    eventPublisher := NewInMemoryEventPublisher()
    
    // Use Cases
    createUserUC := NewCreateUserUseCase(userRepo, emailService, eventPublisher)
    activateUserUC := NewActivateUserUseCase(userRepo, emailService, eventPublisher)
    getUserUC := NewGetUserUseCase(userRepo)
    listUsersUC := NewListUsersUseCase(userRepo)
    
    // Controllers
    userController := NewUserController(createUserUC, activateUserUC, getUserUC, listUsersUC)
    
    return &Container{
        userRepo:       userRepo,
        emailService:   emailService,
        eventPublisher: eventPublisher,
        createUserUC:   createUserUC,
        activateUserUC: activateUserUC,
        getUserUC:      getUserUC,
        listUsersUC:    listUsersUC,
        userController: userController,
    }
}

func (c *Container) UserController() *UserController {
    return c.userController
}

// Utility functions
func generateID() uint64 {
    return uint64(time.Now().UnixNano())
}

func parseUserID(s string) (UserID, error) {
    // Простая реализация для примера
    if s == "1" {
        return UserID(1), nil
    }
    return UserID(0), errors.New("invalid user ID")
}

func contains(s, substr string) bool {
    return len(s) > 0 && len(substr) > 0 && 
           len(s) >= len(substr) && 
           findSubstring(s, substr)
}

func findSubstring(s, substr string) bool {
    for i := 0; i <= len(s)-len(substr); i++ {
        if s[i:i+len(substr)] == substr {
            return true
        }
    }
    return false
}

// Database interface для абстракции
type Database interface {
    QueryContext(ctx context.Context, query string, args ...interface{}) (Rows, error)
    QueryRowContext(ctx context.Context, query string, args ...interface{}) Row
    ExecContext(ctx context.Context, query string, args ...interface{}) (Result, error)
}

type Rows interface {
    Next() bool
    Scan(dest ...interface{}) error
    Close() error
}

type Row interface {
    Scan(dest ...interface{}) error
}

type Result interface {
    RowsAffected() (int64, error)
}

// Пример использования Clean Architecture
func demonstrateCleanArchitecture() {
    // Создаем mock database для демонстрации
    db := &MockDatabase{}
    
    // Создаем контейнер зависимостей
    container := NewContainer(db)
    
    // Получаем controller
    userController := container.UserController()
    
    ctx := context.Background()
    
    // Создаем пользователя
    createReq := CreateUserHTTPRequest{
        Email:    "john@example.com",
        Name:     "John Doe",
        Password: "password123",
    }
    
    userResp, err := userController.CreateUser(ctx, createReq)
    if err != nil {
        fmt.Printf("Error creating user: %v\n", err)
        return
    }
    
    fmt.Printf("Created user: %+v\n", userResp)
    
    // Активируем пользователя
    err = userController.ActivateUser(ctx, "1")
    if err != nil {
        fmt.Printf("Error activating user: %v\n", err)
        return
    }
    
    fmt.Println("User activated successfully")
    
    // Получаем пользователя
    user, err := userController.GetUser(ctx, "1")
    if err != nil {
        fmt.Printf("Error getting user: %v\n", err)
        return
    }
    
    fmt.Printf("Retrieved user: %+v\n", user)
}

// Mock Database для тестирования
type MockDatabase struct {
    users map[uint64]*User
}

func (m *MockDatabase) QueryContext(ctx context.Context, query string, args ...interface{}) (Rows, error) {
    return &MockRows{}, nil
}

func (m *MockDatabase) QueryRowContext(ctx context.Context, query string, args ...interface{}) Row {
    return &MockRow{
        user: &User{
            ID:        UserID(1),
            Email:     Email("john@example.com"),
            Name:      "John Doe",
            Password:  Password("hashed_password123"),
            IsActive:  true,
            CreatedAt: time.Now(),
            UpdatedAt: time.Now(),
        },
    }
}

func (m *MockDatabase) ExecContext(ctx context.Context, query string, args ...interface{}) (Result, error) {
    return &MockResult{rowsAffected: 1}, nil
}

type MockRows struct {
    closed bool
}

func (m *MockRows) Next() bool {
    return false
}

func (m *MockRows) Scan(dest ...interface{}) error {
    return nil
}

func (m *MockRows) Close() error {
    m.closed = true
    return nil
}

type MockRow struct {
    user *User
}

func (m *MockRow) Scan(dest ...interface{}) error {
    if len(dest) >= 7 {
        if id, ok := dest[0].(*uint64); ok {
            *id = uint64(m.user.ID)
        }
        if email, ok := dest[1].(*string); ok {
            *email = m.user.Email.String()
        }
        if name, ok := dest[2].(*string); ok {
            *name = m.user.Name
        }
        if password, ok := dest[3].(*string); ok {
            *password = m.user.Password.String()
        }
        if isActive, ok := dest[4].(*bool); ok {
            *isActive = m.user.IsActive
        }
        if createdAt, ok := dest[5].(*time.Time); ok {
            *createdAt = m.user.CreatedAt
        }
        if updatedAt, ok := dest[6].(*time.Time); ok {
            *updatedAt = m.user.UpdatedAt
        }
    }
    return nil
}

type MockResult struct {
    rowsAffected int64
}

func (m *MockResult) RowsAffected() (int64, error) {
    return m.rowsAffected, nil
}
```

**🎯 Преимущества Clean Architecture:**
- **Независимость от фреймворков** - бизнес-логика не зависит от веб-фреймворков
- **Тестируемость** - каждый слой можно тестировать изолированно
- **Независимость от БД** - можно менять БД без изменения бизнес-логики
- **Независимость от UI** - можно добавить CLI, gRPC, GraphQL без изменений
- **Независимость от внешних сервисов** - легко мокать email, payment и др.

**💡 Ключевые принципы:**
- Dependency Inversion Principle - зависимости направлены внутрь
- Single Responsibility Principle - каждый компонент имеет одну ответственность
- Open/Closed Principle - расширяем функциональность без изменения существующего кода
- Interface Segregation - маленькие, специфичные интерфейсы

---

## 🚀 Этап 3: Продвинутые backend технологии (4-5 недель)

### Неделя 8: API Design и документация

#### 🏗️ RESTful API Design Principles

### REST API Design Patterns

```
REST API Design Hierarchy
┌─────────────────────────────────────────────────────────────┐
│                     API Design Layers                       │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                API Documentation                        │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │ │
│  │  │   OpenAPI   │  │  Postman    │  │   Insomnia  │     │ │
│  │  │   Swagger   │  │ Collections │  │    Tests    │     │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘     │ │
│  └─────────────────────────────────────────────────────────┘ │
│                           │                                 │
│                           ▼                                 │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                API Gateway Layer                        │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │ │
│  │  │    Rate     │  │    Auth     │  │   Request   │     │ │
│  │  │  Limiting   │  │ Validation  │  │ Validation  │     │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘     │ │
│  └─────────────────────────────────────────────────────────┘ │
│                           │                                 │
│                           ▼                                 │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                 HTTP Layer                              │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │ │
│  │  │   Headers   │  │    Status   │  │   Content   │     │ │
│  │  │ Management  │  │    Codes    │  │Negotiation  │     │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘     │ │
│  └─────────────────────────────────────────────────────────┘ │
│                           │                                 │
│                           ▼                                 │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │               Resource Layer                            │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │ │
│  │  │  Resources  │  │    CRUD     │  │ Relationship│     │ │
│  │  │   Design    │  │ Operations  │  │  Handling   │     │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘     │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘

HTTP Methods & Status Codes
┌─────────────────────────────────────────────────────────────┐
│                    HTTP Methods Usage                       │
├─────────────────────────────────────────────────────────────┤
│  Method  │    Purpose    │   Idempotent  │    Safe    │     │
├─────────────────────────────────────────────────────────────┤
│   GET    │ Retrieve data │      Yes      │    Yes     │     │
│   POST   │ Create new    │      No       │    No      │     │
│   PUT    │ Replace/Create│      Yes      │    No      │     │
│   PATCH  │ Partial update│      No       │    No      │     │
│  DELETE  │ Remove        │      Yes      │    No      │     │
│  HEAD    │ Get headers   │      Yes      │    Yes     │     │
│ OPTIONS  │ Get methods   │      Yes      │    Yes     │     │
├─────────────────────────────────────────────────────────────┤
│                   Status Code Categories                    │
├─────────────────────────────────────────────────────────────┤
│  2xx     │ Success       │ 200 OK, 201 Created, 204 No Content │
│  3xx     │ Redirection   │ 301 Moved, 304 Not Modified      │
│  4xx     │ Client Error  │ 400 Bad Request, 404 Not Found   │
│  5xx     │ Server Error  │ 500 Internal, 503 Unavailable   │
└─────────────────────────────────────────────────────────────┘
```

```go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
    "strconv"
    "strings"
    "time"
    
    "github.com/gorilla/mux"
    "github.com/go-playground/validator/v10"
)

// ==================== API MODELS & DTOs ====================

// Standard API Response wrapper
type APIResponse struct {
    Success   bool        `json:"success"`
    Data      interface{} `json:"data,omitempty"`
    Error     *APIError   `json:"error,omitempty"`
    Meta      *MetaData   `json:"meta,omitempty"`
    Timestamp time.Time   `json:"timestamp"`
}

type APIError struct {
    Code    string                 `json:"code"`
    Message string                 `json:"message"`
    Details map[string]interface{} `json:"details,omitempty"`
}

type MetaData struct {
    Pagination *PaginationMeta `json:"pagination,omitempty"`
    Total      int             `json:"total,omitempty"`
    RequestID  string          `json:"request_id,omitempty"`
}

type PaginationMeta struct {
    Page       int  `json:"page"`
    Limit      int  `json:"limit"`
    Total      int  `json:"total"`
    TotalPages int  `json:"total_pages"`
    HasNext    bool `json:"has_next"`
    HasPrev    bool `json:"has_prev"`
}

// User resource DTOs
type UserCreateRequest struct {
    Email    string  `json:"email" validate:"required,email,max=255"`
    Name     string  `json:"name" validate:"required,min=2,max=100"`
    Password string  `json:"password" validate:"required,min=8,max=128"`
    Role     *string `json:"role,omitempty" validate:"omitempty,oneof=user admin moderator"`
}

type UserUpdateRequest struct {
    Email    *string `json:"email,omitempty" validate:"omitempty,email,max=255"`
    Name     *string `json:"name,omitempty" validate:"omitempty,min=2,max=100"`
    IsActive *bool   `json:"is_active,omitempty"`
    Role     *string `json:"role,omitempty" validate:"omitempty,oneof=user admin moderator"`
}

type UserResponse struct {
    ID        uint64    `json:"id"`
    Email     string    `json:"email"`
    Name      string    `json:"name"`
    IsActive  bool      `json:"is_active"`
    Role      string    `json:"role"`
    CreatedAt time.Time `json:"created_at"`
    UpdatedAt time.Time `json:"updated_at"`
    Links     UserLinks `json:"_links"`
}

// HATEOAS Links
type UserLinks struct {
    Self     Link `json:"self"`
    Orders   Link `json:"orders,omitempty"`
    Profile  Link `json:"profile,omitempty"`
    Activate Link `json:"activate,omitempty"`
}

type Link struct {
    Href   string `json:"href"`
    Method string `json:"method,omitempty"`
    Type   string `json:"type,omitempty"`
}

// Query parameters
type UserListQuery struct {
    Page     int    `query:"page" validate:"min=1"`
    Limit    int    `query:"limit" validate:"min=1,max=100"`
    Search   string `query:"search" validate:"max=255"`
    Role     string `query:"role" validate:"omitempty,oneof=user admin moderator"`
    IsActive *bool  `query:"is_active"`
    Sort     string `query:"sort" validate:"omitempty,oneof=name email created_at updated_at"`
    Order    string `query:"order" validate:"omitempty,oneof=asc desc"`
}

func (q *UserListQuery) SetDefaults() {
    if q.Page == 0 {
        q.Page = 1
    }
    if q.Limit == 0 {
        q.Limit = 20
    }
    if q.Sort == "" {
        q.Sort = "created_at"
    }
    if q.Order == "" {
        q.Order = "desc"
    }
}

// ==================== API CONTROLLER ====================

type UserAPIController struct {
    userService UserService
    validator   *validator.Validate
    baseURL     string
}

func NewUserAPIController(userService UserService, baseURL string) *UserAPIController {
    return &UserAPIController{
        userService: userService,
        validator:   validator.New(),
        baseURL:     baseURL,
    }
}

// GET /api/v1/users
func (c *UserAPIController) ListUsers(w http.ResponseWriter, r *http.Request) {
    // Parse query parameters
    query := &UserListQuery{}
    if err := c.parseQuery(r, query); err != nil {
        c.respondError(w, http.StatusBadRequest, "INVALID_QUERY", err.Error(), nil)
        return
    }
    
    query.SetDefaults()
    
    // Validate query parameters
    if err := c.validator.Struct(query); err != nil {
        c.respondValidationError(w, err)
        return
    }
    
    // Get users from service
    users, total, err := c.userService.ListUsers(r.Context(), UserFilters{
        Search:   query.Search,
        Role:     query.Role,
        IsActive: query.IsActive,
        Sort:     query.Sort,
        Order:    query.Order,
        Limit:    query.Limit,
        Offset:   (query.Page - 1) * query.Limit,
    })
    
    if err != nil {
        c.respondError(w, http.StatusInternalServerError, "SERVICE_ERROR", 
            "Failed to retrieve users", map[string]interface{}{"internal_error": err.Error()})
        return
    }
    
    // Convert to response DTOs
    userResponses := make([]UserResponse, len(users))
    for i, user := range users {
        userResponses[i] = c.userToResponse(user)
    }
    
    // Create pagination metadata
    totalPages := (total + query.Limit - 1) / query.Limit
    paginationMeta := &PaginationMeta{
        Page:       query.Page,
        Limit:      query.Limit,
        Total:      total,
        TotalPages: totalPages,
        HasNext:    query.Page < totalPages,
        HasPrev:    query.Page > 1,
    }
    
    c.respondSuccess(w, http.StatusOK, userResponses, &MetaData{
        Pagination: paginationMeta,
        Total:      total,
        RequestID:  c.getRequestID(r),
    })
}

// POST /api/v1/users
func (c *UserAPIController) CreateUser(w http.ResponseWriter, r *http.Request) {
    var req UserCreateRequest
    
    // Parse JSON body
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        c.respondError(w, http.StatusBadRequest, "INVALID_JSON", 
            "Invalid JSON format", map[string]interface{}{"parse_error": err.Error()})
        return
    }
    
    // Validate request
    if err := c.validator.Struct(&req); err != nil {
        c.respondValidationError(w, err)
        return
    }
    
    // Create user through service
    user, err := c.userService.CreateUser(r.Context(), CreateUserRequest{
        Email:    req.Email,
        Name:     req.Name,
        Password: req.Password,
        Role:     c.valueOrDefault(req.Role, "user"),
    })
    
    if err != nil {
        // Handle known business errors
        switch err {
        case ErrEmailAlreadyExists:
            c.respondError(w, http.StatusConflict, "EMAIL_EXISTS", 
                "User with this email already exists", nil)
        case ErrInvalidPassword:
            c.respondError(w, http.StatusBadRequest, "INVALID_PASSWORD", 
                "Password does not meet requirements", nil)
        default:
            c.respondError(w, http.StatusInternalServerError, "SERVICE_ERROR", 
                "Failed to create user", map[string]interface{}{"internal_error": err.Error()})
        }
        return
    }
    
    // Set Location header
    location := fmt.Sprintf("%s/api/v1/users/%d", c.baseURL, user.ID)
    w.Header().Set("Location", location)
    
    c.respondSuccess(w, http.StatusCreated, c.userToResponse(user), &MetaData{
        RequestID: c.getRequestID(r),
    })
}

// GET /api/v1/users/{id}
func (c *UserAPIController) GetUser(w http.ResponseWriter, r *http.Request) {
    // Extract ID from URL
    vars := mux.Vars(r)
    idStr := vars["id"]
    
    userID, err := strconv.ParseUint(idStr, 10, 64)
    if err != nil {
        c.respondError(w, http.StatusBadRequest, "INVALID_USER_ID", 
            "User ID must be a valid number", nil)
        return
    }
    
    // Get user from service
    user, err := c.userService.GetUserByID(r.Context(), userID)
    if err != nil {
        if err == ErrUserNotFound {
            c.respondError(w, http.StatusNotFound, "USER_NOT_FOUND", 
                fmt.Sprintf("User with ID %d not found", userID), nil)
        } else {
            c.respondError(w, http.StatusInternalServerError, "SERVICE_ERROR", 
                "Failed to retrieve user", map[string]interface{}{"internal_error": err.Error()})
        }
        return
    }
    
    // Check for conditional requests
    if c.handleConditionalRequest(w, r, user.UpdatedAt) {
        return
    }
    
    c.respondSuccess(w, http.StatusOK, c.userToResponse(user), &MetaData{
        RequestID: c.getRequestID(r),
    })
}

// PUT /api/v1/users/{id}
func (c *UserAPIController) UpdateUser(w http.ResponseWriter, r *http.Request) {
    // Extract ID from URL
    vars := mux.Vars(r)
    idStr := vars["id"]
    
    userID, err := strconv.ParseUint(idStr, 10, 64)
    if err != nil {
        c.respondError(w, http.StatusBadRequest, "INVALID_USER_ID", 
            "User ID must be a valid number", nil)
        return
    }
    
    var req UserUpdateRequest
    
    // Parse JSON body
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        c.respondError(w, http.StatusBadRequest, "INVALID_JSON", 
            "Invalid JSON format", map[string]interface{}{"parse_error": err.Error()})
        return
    }
    
    // Validate request
    if err := c.validator.Struct(&req); err != nil {
        c.respondValidationError(w, err)
        return
    }
    
    // Update user through service
    user, err := c.userService.UpdateUser(r.Context(), userID, UpdateUserRequest{
        Email:    req.Email,
        Name:     req.Name,
        IsActive: req.IsActive,
        Role:     req.Role,
    })
    
    if err != nil {
        switch err {
        case ErrUserNotFound:
            c.respondError(w, http.StatusNotFound, "USER_NOT_FOUND", 
                fmt.Sprintf("User with ID %d not found", userID), nil)
        case ErrEmailAlreadyExists:
            c.respondError(w, http.StatusConflict, "EMAIL_EXISTS", 
                "User with this email already exists", nil)
        default:
            c.respondError(w, http.StatusInternalServerError, "SERVICE_ERROR", 
                "Failed to update user", map[string]interface{}{"internal_error": err.Error()})
        }
        return
    }
    
    c.respondSuccess(w, http.StatusOK, c.userToResponse(user), &MetaData{
        RequestID: c.getRequestID(r),
    })
}

// DELETE /api/v1/users/{id}
func (c *UserAPIController) DeleteUser(w http.ResponseWriter, r *http.Request) {
    // Extract ID from URL
    vars := mux.Vars(r)
    idStr := vars["id"]
    
    userID, err := strconv.ParseUint(idStr, 10, 64)
    if err != nil {
        c.respondError(w, http.StatusBadRequest, "INVALID_USER_ID", 
            "User ID must be a valid number", nil)
        return
    }
    
    // Delete user through service
    err = c.userService.DeleteUser(r.Context(), userID)
    if err != nil {
        if err == ErrUserNotFound {
            c.respondError(w, http.StatusNotFound, "USER_NOT_FOUND", 
                fmt.Sprintf("User with ID %d not found", userID), nil)
        } else {
            c.respondError(w, http.StatusInternalServerError, "SERVICE_ERROR", 
                "Failed to delete user", map[string]interface{}{"internal_error": err.Error()})
        }
        return
    }
    
    // Return 204 No Content for successful deletion
    w.WriteHeader(http.StatusNoContent)
}

// PATCH /api/v1/users/{id}/activate
func (c *UserAPIController) ActivateUser(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    idStr := vars["id"]
    
    userID, err := strconv.ParseUint(idStr, 10, 64)
    if err != nil {
        c.respondError(w, http.StatusBadRequest, "INVALID_USER_ID", 
            "User ID must be a valid number", nil)
        return
    }
    
    err = c.userService.ActivateUser(r.Context(), userID)
    if err != nil {
        switch err {
        case ErrUserNotFound:
            c.respondError(w, http.StatusNotFound, "USER_NOT_FOUND", 
                fmt.Sprintf("User with ID %d not found", userID), nil)
        case ErrUserAlreadyActive:
            c.respondError(w, http.StatusConflict, "USER_ALREADY_ACTIVE", 
                "User is already active", nil)
        default:
            c.respondError(w, http.StatusInternalServerError, "SERVICE_ERROR", 
                "Failed to activate user", map[string]interface{}{"internal_error": err.Error()})
        }
        return
    }
    
    // Get updated user to return
    user, err := c.userService.GetUserByID(r.Context(), userID)
    if err != nil {
        c.respondError(w, http.StatusInternalServerError, "SERVICE_ERROR", 
            "User activated but failed to retrieve updated data", nil)
        return
    }
    
    c.respondSuccess(w, http.StatusOK, c.userToResponse(user), &MetaData{
        RequestID: c.getRequestID(r),
    })
}

// HEAD /api/v1/users/{id}
func (c *UserAPIController) HeadUser(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    idStr := vars["id"]
    
    userID, err := strconv.ParseUint(idStr, 10, 64)
    if err != nil {
        w.WriteHeader(http.StatusBadRequest)
        return
    }
    
    user, err := c.userService.GetUserByID(r.Context(), userID)
    if err != nil {
        if err == ErrUserNotFound {
            w.WriteHeader(http.StatusNotFound)
        } else {
            w.WriteHeader(http.StatusInternalServerError)
        }
        return
    }
    
    // Set headers without body
    w.Header().Set("Content-Type", "application/json")
    w.Header().Set("Last-Modified", user.UpdatedAt.Format(time.RFC1123))
    w.Header().Set("ETag", fmt.Sprintf(`"%d-%d"`, user.ID, user.UpdatedAt.Unix()))
    w.WriteHeader(http.StatusOK)
}

// OPTIONS /api/v1/users
func (c *UserAPIController) OptionsUsers(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Allow", "GET, POST, OPTIONS")
    w.Header().Set("Access-Control-Allow-Methods", "GET, POST, OPTIONS")
    w.Header().Set("Access-Control-Allow-Headers", "Content-Type, Authorization")
    w.WriteHeader(http.StatusOK)
}

// OPTIONS /api/v1/users/{id}
func (c *UserAPIController) OptionsUser(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Allow", "GET, PUT, DELETE, PATCH, HEAD, OPTIONS")
    w.Header().Set("Access-Control-Allow-Methods", "GET, PUT, DELETE, PATCH, HEAD, OPTIONS")
    w.Header().Set("Access-Control-Allow-Headers", "Content-Type, Authorization, If-Modified-Since, If-None-Match")
    w.WriteHeader(http.StatusOK)
}

// ==================== HELPER METHODS ====================

func (c *UserAPIController) userToResponse(user *User) UserResponse {
    baseURL := c.baseURL
    userURL := fmt.Sprintf("%s/api/v1/users/%d", baseURL, user.ID)
    
    links := UserLinks{
        Self: Link{
            Href:   userURL,
            Method: "GET",
            Type:   "application/json",
        },
        Orders: Link{
            Href:   fmt.Sprintf("%s/orders", userURL),
            Method: "GET",
            Type:   "application/json",
        },
        Profile: Link{
            Href:   fmt.Sprintf("%s/profile", userURL),
            Method: "GET",
            Type:   "application/json",
        },
    }
    
    // Conditional links based on state
    if !user.IsActive {
        links.Activate = Link{
            Href:   fmt.Sprintf("%s/activate", userURL),
            Method: "PATCH",
            Type:   "application/json",
        }
    }
    
    return UserResponse{
        ID:        user.ID,
        Email:     user.Email,
        Name:      user.Name,
        IsActive:  user.IsActive,
        Role:      user.Role,
        CreatedAt: user.CreatedAt,
        UpdatedAt: user.UpdatedAt,
        Links:     links,
    }
}

func (c *UserAPIController) respondSuccess(w http.ResponseWriter, status int, data interface{}, meta *MetaData) {
    response := APIResponse{
        Success:   true,
        Data:      data,
        Meta:      meta,
        Timestamp: time.Now().UTC(),
    }
    
    c.writeJSONResponse(w, status, response)
}

func (c *UserAPIController) respondError(w http.ResponseWriter, status int, code, message string, details map[string]interface{}) {
    response := APIResponse{
        Success: false,
        Error: &APIError{
            Code:    code,
            Message: message,
            Details: details,
        },
        Timestamp: time.Now().UTC(),
    }
    
    c.writeJSONResponse(w, status, response)
}

func (c *UserAPIController) respondValidationError(w http.ResponseWriter, err error) {
    details := make(map[string]interface{})
    
    if validationErrors, ok := err.(validator.ValidationErrors); ok {
        fieldErrors := make(map[string]string)
        for _, fieldError := range validationErrors {
            fieldErrors[strings.ToLower(fieldError.Field())] = c.getValidationErrorMessage(fieldError)
        }
        details["field_errors"] = fieldErrors
    }
    
    c.respondError(w, http.StatusBadRequest, "VALIDATION_ERROR", 
        "Request validation failed", details)
}

func (c *UserAPIController) getValidationErrorMessage(err validator.FieldError) string {
    switch err.Tag() {
    case "required":
        return "This field is required"
    case "email":
        return "Must be a valid email address"
    case "min":
        return fmt.Sprintf("Must be at least %s characters", err.Param())
    case "max":
        return fmt.Sprintf("Must be no more than %s characters", err.Param())
    case "oneof":
        return fmt.Sprintf("Must be one of: %s", err.Param())
    default:
        return "Invalid value"
    }
}

func (c *UserAPIController) writeJSONResponse(w http.ResponseWriter, status int, data interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    
    if err := json.NewEncoder(w).Encode(data); err != nil {
        // Log error but don't send another response
        fmt.Printf("Failed to encode JSON response: %v\n", err)
    }
}

func (c *UserAPIController) parseQuery(r *http.Request, dest interface{}) error {
    // Simplified query parsing - in production use gorilla/schema or similar
    query := r.URL.Query()
    
    if q, ok := dest.(*UserListQuery); ok {
        if page := query.Get("page"); page != "" {
            if p, err := strconv.Atoi(page); err == nil {
                q.Page = p
            }
        }
        if limit := query.Get("limit"); limit != "" {
            if l, err := strconv.Atoi(limit); err == nil {
                q.Limit = l
            }
        }
        q.Search = query.Get("search")
        q.Role = query.Get("role")
        q.Sort = query.Get("sort")
        q.Order = query.Get("order")
        
        if isActive := query.Get("is_active"); isActive != "" {
            if active, err := strconv.ParseBool(isActive); err == nil {
                q.IsActive = &active
            }
        }
    }
    
    return nil
}

func (c *UserAPIController) handleConditionalRequest(w http.ResponseWriter, r *http.Request, lastModified time.Time) bool {
    // Handle If-Modified-Since
    if modifiedSince := r.Header.Get("If-Modified-Since"); modifiedSince != "" {
        if t, err := time.Parse(time.RFC1123, modifiedSince); err == nil {
            if lastModified.Before(t.Add(time.Second)) {
                w.WriteHeader(http.StatusNotModified)
                return true
            }
        }
    }
    
    // Handle If-None-Match (ETag)
    if noneMatch := r.Header.Get("If-None-Match"); noneMatch != "" {
        etag := fmt.Sprintf(`"%d"`, lastModified.Unix())
        if noneMatch == etag {
            w.WriteHeader(http.StatusNotModified)
            return true
        }
    }
    
    // Set response headers
    w.Header().Set("Last-Modified", lastModified.Format(time.RFC1123))
    w.Header().Set("ETag", fmt.Sprintf(`"%d"`, lastModified.Unix()))
    
    return false
}

func (c *UserAPIController) getRequestID(r *http.Request) string {
    if id := r.Header.Get("X-Request-ID"); id != "" {
        return id
    }
    return fmt.Sprintf("req_%d", time.Now().UnixNano())
}

func (c *UserAPIController) valueOrDefault(value *string, defaultValue string) string {
    if value != nil {
        return *value
    }
    return defaultValue
}

// ==================== API VERSIONING ====================

type APIVersionMiddleware struct {
    defaultVersion string
    supportedVersions map[string]bool
}

func NewAPIVersionMiddleware(defaultVersion string, supportedVersions []string) *APIVersionMiddleware {
    supported := make(map[string]bool)
    for _, version := range supportedVersions {
        supported[version] = true
    }
    
    return &APIVersionMiddleware{
        defaultVersion:    defaultVersion,
        supportedVersions: supported,
    }
}

func (m *APIVersionMiddleware) Middleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        version := m.extractVersion(r)
        
        if !m.supportedVersions[version] {
            http.Error(w, fmt.Sprintf("API version %s is not supported", version), 
                http.StatusBadRequest)
            return
        }
        
        // Add version to context or headers
        r.Header.Set("X-API-Version", version)
        w.Header().Set("X-API-Version", version)
        
        next.ServeHTTP(w, r)
    })
}

func (m *APIVersionMiddleware) extractVersion(r *http.Request) string {
    // Method 1: URL path versioning (/api/v1/users)
    if strings.HasPrefix(r.URL.Path, "/api/v") {
        parts := strings.Split(r.URL.Path, "/")
        if len(parts) >= 3 && strings.HasPrefix(parts[2], "v") {
            return parts[2]
        }
    }
    
    // Method 2: Header versioning
    if version := r.Header.Get("API-Version"); version != "" {
        return version
    }
    
    // Method 3: Accept header versioning
    if accept := r.Header.Get("Accept"); accept != "" {
        if strings.Contains(accept, "version=") {
            // Parse application/json;version=v1
            parts := strings.Split(accept, "version=")
            if len(parts) > 1 {
                return strings.Split(parts[1], ";")[0]
            }
        }
    }
    
    return m.defaultVersion
}

// ==================== CONTENT NEGOTIATION ====================

type ContentNegotiationMiddleware struct{}

func NewContentNegotiationMiddleware() *ContentNegotiationMiddleware {
    return &ContentNegotiationMiddleware{}
}

func (m *ContentNegotiationMiddleware) Middleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Check Accept header
        accept := r.Header.Get("Accept")
        
        if accept != "" && !strings.Contains(accept, "application/json") && 
           !strings.Contains(accept, "*/*") {
            http.Error(w, "Not Acceptable: only application/json is supported", 
                http.StatusNotAcceptable)
            return
        }
        
        // Check Content-Type for requests with body
        if r.Method == "POST" || r.Method == "PUT" || r.Method == "PATCH" {
            contentType := r.Header.Get("Content-Type")
            if contentType != "" && !strings.HasPrefix(contentType, "application/json") {
                http.Error(w, "Unsupported Media Type: only application/json is supported", 
                    http.StatusUnsupportedMediaType)
                return
            }
        }
        
        next.ServeHTTP(w, r)
    })
}

// ==================== ROUTER SETUP ====================

func SetupAPIRoutes(controller *UserAPIController) *mux.Router {
    r := mux.NewRouter()
    
    // Middleware
    versionMiddleware := NewAPIVersionMiddleware("v1", []string{"v1", "v2"})
    contentMiddleware := NewContentNegotiationMiddleware()
    
    r.Use(versionMiddleware.Middleware)
    r.Use(contentMiddleware.Middleware)
    
    // API v1 routes
    v1 := r.PathPrefix("/api/v1").Subrouter()
    
    // Users collection
    v1.HandleFunc("/users", controller.ListUsers).Methods("GET")
    v1.HandleFunc("/users", controller.CreateUser).Methods("POST")
    v1.HandleFunc("/users", controller.OptionsUsers).Methods("OPTIONS")
    
    // Individual user
    v1.HandleFunc("/users/{id:[0-9]+}", controller.GetUser).Methods("GET")
    v1.HandleFunc("/users/{id:[0-9]+}", controller.UpdateUser).Methods("PUT")
    v1.HandleFunc("/users/{id:[0-9]+}", controller.DeleteUser).Methods("DELETE")
    v1.HandleFunc("/users/{id:[0-9]+}", controller.HeadUser).Methods("HEAD")
    v1.HandleFunc("/users/{id:[0-9]+}", controller.OptionsUser).Methods("OPTIONS")
    
    // User actions
    v1.HandleFunc("/users/{id:[0-9]+}/activate", controller.ActivateUser).Methods("PATCH")
    
    return r
}

// Error definitions
var (
    ErrEmailAlreadyExists = fmt.Errorf("email already exists")
    ErrInvalidPassword    = fmt.Errorf("invalid password")
    ErrUserAlreadyActive  = fmt.Errorf("user already active")
)

// Mock service interfaces for compilation
type UserService interface {
    ListUsers(ctx context.Context, filters UserFilters) ([]*User, int, error)
    CreateUser(ctx context.Context, req CreateUserRequest) (*User, error)
    GetUserByID(ctx context.Context, id uint64) (*User, error)
    UpdateUser(ctx context.Context, id uint64, req UpdateUserRequest) (*User, error)
    DeleteUser(ctx context.Context, id uint64) error
    ActivateUser(ctx context.Context, id uint64) error
}

type UserFilters struct {
    Search   string
    Role     string
    IsActive *bool
    Sort     string
    Order    string
    Limit    int
    Offset   int
}

type CreateUserRequest struct {
    Email    string
    Name     string
    Password string
    Role     string
}

type UpdateUserRequest struct {
    Email    *string
    Name     *string
    IsActive *bool
    Role     *string
}

type User struct {
    ID        uint64
    Email     string
    Name      string
    IsActive  bool
    Role      string
    CreatedAt time.Time
    UpdatedAt time.Time
}
```

**🎯 Преимущества правильного API дизайна:**
- **Предсказуемость** - стандартные HTTP методы и коды ответов
- **Самодокументируемость** - HATEOAS ссылки показывают доступные действия
- **Версионирование** - поддержка нескольких версий API
- **Валидация** - комплексная проверка входящих данных
- **Обработка ошибок** - структурированные и информативные ошибки

**💡 Ключевые принципы REST API:**
- Ресурсы идентифицируются URL
- Используйте HTTP методы по назначению
- Stateless - каждый запрос независим
- Кэшируемость через HTTP заголовки
- Layered system - возможность использования прокси, кэшей

---

Это завершает детальный обзор HTTP серверов и API дизайна в Go. Материал покрывает от базовых серверов до enterprise-level архитектур с middleware, validation, versioning и HATEOAS. Хотели бы вы продолжить с остальными неделями или углубиться в какую-то конкретную тему?