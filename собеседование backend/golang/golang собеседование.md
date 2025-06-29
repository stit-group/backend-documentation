# Golang Backend Interview Guide
## Clean Architecture & Dependency Injection Focus

---

## 🎯 Структура интервью

Этот гайд содержит 80 основных вопросов с 5 углубляющими подвопросами к каждому. Особый акцент на Clean Architecture, DI и практические примеры из реального кода.

---

## 📋 Категории вопросов

1. **Основы языка** (15 вопросов)
2. **Интерфейсы и Dependency Injection** (15 вопросов) 
3. **Clean Architecture и SOLID** (15 вопросов)
4. **Concurrency и Goroutines** (15 вопросов)
5. **Типы данных и структуры** (10 вопросов)
6. **Тестирование и моки** (10 вопросов)

---

## 1. Основы языка (15 вопросов)

### 1.1 Что такое указатели в Go и когда их использовать?

**Пример из вашего кода:**
```go
func (pg *Postgres) Insert(ctx context.Context, query string, args ...any) (any, error) {
    var id any
    tx := pg.getTx(ctx)
    if tx != nil {
        err := tx.QueryRow(ctx, query, args...).Scan(&id)
        // ...
    }
    return id, nil
}
```

**Углубляющие вопросы:**
1. Почему в методе `Insert` используется `&id` при вызове `Scan()`?
2. Как Go управляет памятью при работе с указателями?
3. В чем разница между `var p *int` и `p := &variable`?
4. Когда использование указателей может привести к утечкам памяти?
5. Объясните, почему receiver `(pg *Postgres)` объявлен как указатель?

---

### 1.2 Объясните defer, panic и recover в Go

**Пример из вашего кода:**
```go
func RecoverMiddleware00(
    requestIDMiddleware01 func(evt interface{}),
) func(evt interface{}) {
    return func(evt interface{}) {
        defer func() {
            if r := recover(); r != nil {
                fmt.Println(string(debug.Stack()))
            }
        }()
        requestIDMiddleware01(evt)
    }
}
```

**Углубляющие вопросы:**
1. Почему в middleware используется `defer` для recover?
2. Что произойдет, если panic возникнет в defer функции?
3. Как правильно логировать panic в production системах?
4. Можно ли использовать recover без defer?
5. Какие альтернативы panic/recover существуют в Go?

---

### 1.3 Что такое контекст (context) и как его использовать?

**Пример из вашего кода:**
```go
func (pg *Postgres) CtxWithTx(ctx context.Context) (context.Context, error) {
    tx, err := pg.db.Begin(ctx)
    if err != nil {
        return nil, err
    }
    return context.WithValue(ctx, txKey{}, tx), nil
}
```

**Углубляющие вопросы:**
1. Почему для ключа контекста используется пустая структура `txKey{}`?
2. Как избежать коллизий ключей в context.WithValue?
3. Какие проблемы могут возникнуть при передаче большого количества данных через контекст?
4. Как правильно обрабатывать timeout в контексте?
5. Что лучше: передавать данные через контекст или параметры функции?

---

### 1.4 Объясните интерфейсы в Go

**Пример из вашего кода:**
```go
type IUserbotService interface {
    CreateUserbot(ctx context.Context, accountID int) (*bytes.Buffer, error)
    StartAll(ctx context.Context)
    SendMessage(ctx context.Context, userbotWaPhoneNumber, contactWaPhoneNumber, text string) error
}
```

**Углубляющие вопросы:**
1. Почему интерфейсы в Go называются "неявными"?
2. Как интерфейсы помогают в реализации Dependency Injection?
3. В чем преимущества маленьких интерфейсов vs больших?
4. Что такое "interface segregation" и как это применяется в вашем коде?
5. Как тестировать код, использующий интерфейсы?

---

### 1.5 Что такое слайсы и как они работают внутри?

**Пример:**
```go
// Покажите этот код кандидату
func appendExample() {
    s1 := []int{1, 2, 3}
    s2 := append(s1, 4)
    s1[0] = 999
    fmt.Println(s2[0]) // Что выведет?
}
```

**Углубляющие вопросы:**
1. Объясните внутреннее устройство slice (указатель, длина, емкость)?
2. Когда append создает новый underlying array?
3. Как избежать проблем с shared underlying array?
4. В чем разница между `var s []int` и `s := make([]int, 0)`?
5. Как правильно копировать слайсы?

---

### 1.6 Объясните работу с ошибками в Go

**Пример из вашего кода:**
```go
func (r *RepoAmocrmSource) CreateAmocrmSource(ctx context.Context, /*...*/) error {
    _, err := r.db.Insert(ctx, CreateAmocrmSource, args)
    if err != nil {
        r.logger.Error(ctx, err.Error())
        span.RecordError(err)
        span.SetStatus(codes.Error, err.Error())
        return err
    }
    return nil
}
```

**Углубляющие вопросы:**
1. Как правильно оборачивать ошибки для создания контекста?
2. Когда использовать `errors.Is()` vs `errors.As()`?
3. Как создавать custom error types?
4. Стоит ли логировать ошибки на каждом уровне?
5. Как обрабатывать ошибки в middleware chain?

---

### 1.7 Что такое методы и receivers в Go?

**Пример из вашего кода:**
```go
func (s *ServiceAmocrmSource) CreateAmocrmSource(ctx context.Context, 
    amocrmPipelineID int, userbotWaPhoneNumber string) error {
    // implementation
}
```

**Углубляющие вопросы:**
1. Когда использовать pointer receiver vs value receiver?
2. Как влияет тип receiver на производительность?
3. Можно ли определить методы для встроенных типов?
4. Что такое method sets в Go?
5. Как методы взаимодействуют с интерфейсами?

---

### 1.8 Объясните структуры и композицию в Go

**Пример из вашего кода:**
```go
type ServiceAmocrmSource struct {
    amocrmSourceRepo model.IAmocrmSourceRepo
    amocrmClient     model.IAmocrmClient
    logger           model.OtelLogger
    meter            metric.Meter
    tracer           trace.Tracer
}
```

**Углубляющие вопросы:**
1. Как реализована композиция в этой структуре?
2. Что такое embedding и как его использовать?
3. Как Go решает name conflicts при embedding?
4. В чем преимущества композиции над наследованием?
5. Как структуры помогают в реализации Clean Architecture?

---

### 1.9 Что такое пакеты и модули в Go?

**Пример из вашего go.mod:**
```go
module crmessenger-whatsapp

go 1.23.0

require (
    github.com/georgysavva/scany/v2 v2.1.4
    github.com/go-playground/validator/v10 v10.26.0
    // ...
)
```

**Углубляющие вопросы:**
1. Как организована структура пакетов в вашем проекте?
2. Что такое internal packages и когда их использовать?
3. Как управлять зависимостями с помощью go mod?
4. Что такое vendor директория?
5. Как правильно версионировать Go модули?

---

### 1.10 Объясните работу с JSON в Go

**Пример:**
```go
type CreateAmocrmSourceBody struct {
    UserbotWaPhoneNumber string `json:"userbot_wa_phone_number"`
    AmocrmPipelineID     int    `json:"amocrm_pipeline_id"`
}
```

**Углубляющие вопросы:**
1. Как работают struct tags в Go?
2. Что происходит при unmarshal в поля разных типов?
3. Как обрабатывать optional поля в JSON?
4. Что такое json.RawMessage и когда его использовать?
5. Как кастомизировать marshal/unmarshal процесс?

---

### 1.11 Что такое init функции в Go?

**Углубляющие вопросы:**
1. В каком порядке выполняются init функции?
2. Можно ли иметь несколько init функций в одном пакете?
3. Когда init функции полезны и когда их лучше избегать?
4. Как init функции влияют на тестирование?
5. Какие альтернативы init функциям существуют?

---

### 1.12 Объясните работу reflection в Go

**Углубляющие вопросы:**
1. Как работает пакет reflect?
2. Какие проблемы производительности связаны с reflection?
3. Как библиотеки ORM используют reflection?
4. Можно ли избежать reflection в вашем коде?
5. Как reflection влияет на type safety?

---

### 1.13 Что такое type assertions и type switches?

**Пример:**
```go
func processEvent(evt interface{}) {
    switch event := evt.(type) {
    case *events.Message:
        // handle message
    case *events.JoinedGroup:
        // handle group join
    default:
        // unknown event
    }
}
```

**Углубляющие вопросы:**
1. В чем разница между type assertion и type switch?
2. Как обрабатывать failed type assertions?
3. Когда использовать interface{} оправдано?
4. Как type switches помогают в polymorphism?
5. Какие альтернативы type switches существуют?

---

### 1.14 Объясните работу с каналами (channels)

**Углубляющие вопросы:**
1. В чем разница между buffered и unbuffered каналами?
2. Что такое channel direction и как его использовать?
3. Как каналы помогают в реализации pipeline pattern?
4. Что происходит при закрытии канала?
5. Как избежать deadlock при работе с каналами?

---

### 1.15 Что такое select statement в Go?

**Углубляющие вопросы:**
1. Как select работает с multiple channels?
2. Что такое default case в select?
3. Как реализовать timeout с помощью select?
4. Как select помогает в non-blocking operations?
5. Можно ли использовать select без каналов?

---

## 2. Интерфейсы и Dependency Injection (15 вопросов)

### 2.1 Как реализован Dependency Injection в вашем проекте?

**Пример из main.go:**
```go
func main() {
    cfg := config.New()
    tel := Telemetry.New(/* params */)
    db := PG.New(tel, cfg.DB.Username, /* other params */)
    
    amocrmClient := AmocrmClient.New(tel, cfg.Amocrm.BaseURL, /* params */)
    amocrmRepo := AmocrmSourceRepo.New(db, tel)
    amocrmSourceService := AmocrmSourceService.New(amocrmRepo, amocrmClient, tel)
    
    userbotService := UserbotService.New(userbotRepo, amocrmSourceService, waDeviceDB, tel, /* params */)
    
    server.Run(db, amocrmSourceService, userbotService, tel, cfg.CRMessenger.CRMessengerWhatsapp.Port)
}
```

**Углубляющие вопросы:**
1. Какие преимущества и недостатки manual DI vs DI containers?
2. Как бы вы рефакторили этот код для использования DI container?
3. Как тестировать компоненты с таким подходом к DI?
4. Какие проблемы могут возникнуть при росте количества зависимостей?
5. Как обеспечить правильный lifecycle зависимостей?

---

### 2.2 Объясните принцип Dependency Inversion

**Пример интерфейса:**
```go
type IAmocrmSourceService interface {
    CreateAmocrmSource(ctx context.Context, amocrmPipelineID int, userbotWaPhoneNumber string) error
    SendMessageFromAmocrmToContact(ctx context.Context, /* params */) error
    // ...
}
```

**Углубляющие вопросы:**
1. Как этот интерфейс помогает в тестировании?
2. Почему интерфейс определен в model пакете, а не в service?
3. Как бы вы разделили этот интерфейс согласно Interface Segregation Principle?
4. Какие проблемы могут возникнуть при изменении интерфейса?
5. Как версионировать интерфейсы в Go?

---

### 2.3 Как правильно проектировать интерфейсы в Go?

**Пример из вашего кода:**
```go
type IDB interface {
    Insert(ctx context.Context, query string, args ...any) (any, error)
    Select(ctx context.Context, query string, args ...any) (pgx.Rows, error)
    Delete(ctx context.Context, query string, args ...any) error
    Update(ctx context.Context, query string, args ...any) error
    
    CtxWithTx(ctx context.Context) (context.Context, error)
    CommitTx(ctx context.Context) error
    RollbackTx(ctx context.Context)
    
    CreateTable(ctx context.Context, query string) error
    DropTable(ctx context.Context, query string) error
}
```

**Углубляющие вопросы:**
1. Не слишком ли большой этот интерфейс? Как его можно разделить?
2. Почему методы возвращают `pgx.Rows` вместо более абстрактного типа?
3. Как бы вы сделали этот интерфейс database-agnostic?
4. Стоит ли выделять транзакционные методы в отдельный интерфейс?
5. Как обрабатывать database-specific ошибки через такой интерфейс?

---

### 2.4 Как использовать интерфейсы для мокирования?

**Пример использования:**
```go
func TestAmocrmService(t *testing.T) {
    // Как бы вы создали мок для IAmocrmClient?
    mockClient := &MockAmocrmClient{}
    mockRepo := &MockAmocrmRepo{}
    
    service := AmocrmSourceService.New(mockRepo, mockClient, tel)
    // ...
}
```

**Углубляющие вопросы:**
1. Как генерировать моки автоматически в Go?
2. В чем преимущества testify/mock vs gomock?
3. Как тестировать взаимодействие между компонентами?
4. Стоит ли мокать все зависимости или использовать real implementations?
5. Как обеспечить, что моки соответствуют реальным implementation?

---

### 2.5 Объясните композицию интерфейсов

**Пример:**
```go
type ITelemetry interface {
    Logger() OtelLogger
    Tracer() trace.Tracer
    Meter() metric.Meter
}

type OtelLogger interface {
    Debug(ctx context.Context, message string, fields ...map[string]any)
    Info(ctx context.Context, message string, fields ...map[string]any)
    Warn(ctx context.Context, message string, fields ...map[string]any)
    Error(ctx context.Context, message string, fields ...map[string]any)
}
```

**Углубляющие вопросы:**
1. Как можно композировать эти интерфейсы?
2. Что такое embedding интерфейсов в Go?
3. Как композиция интерфейсов помогает в design?
4. Какие проблемы могут возникнуть при глубокой композиции?
5. Как документировать составные интерфейсы?

---

### 2.6 Как обеспечить type safety при использовании interface{}?

**Пример из middleware:**
```go
func TraceMiddleware02(/* params */) func(ctx context.Context, evt interface{}) error {
    return func(ctx context.Context, evt interface{}) error {
        switch event := evt.(type) {
        case *events.Message:
            // handle message
        case *events.JoinedGroup:
            // handle group join  
        default:
            return nil
        }
    }
}
```

**Углубляющие вопросы:**
1. Можно ли избежать interface{} в этом случае?
2. Как использовать generics для улучшения type safety?
3. Какие runtime проверки стоит добавить?
4. Как документировать ожидаемые типы для interface{}?
5. Какие альтернативы interface{} существуют?

---

### 2.7 Объясните паттерн Strategy через интерфейсы

**Пример применения:**
```go
type MessageSender interface {
    SendMessage(ctx context.Context, recipient, message string) error
}

type WhatsAppSender struct{}
type EmailSender struct{}
type SMSSender struct{}
```

**Углубляющие вопросы:**
1. Как реализовать выбор стратегии в runtime?
2. Как передавать конфигурацию для разных стратегий?
3. Как обрабатывать ошибки, специфичные для стратегии?
4. Можно ли комбинировать несколько стратегий?
5. Как тестировать систему с множественными стратегиями?

---

### 2.8 Как реализовать Factory pattern в Go?

**Пример:**
```go
type ServiceFactory interface {
    CreateAmocrmService() model.IAmocrmSourceService
    CreateUserbotService() model.IUserbotService
}
```

**Углубляющие вопросы:**
1. Когда Factory pattern полезен в Go?
2. Как передавать зависимости в Factory?
3. В чем разница между Factory и Builder pattern?
4. Как обеспечить singleton behavior в Factory?
5. Стоит ли использовать Factory для DI в Go?

---

### 2.9 Объясните интерфейсы как контракты между слоями

**Пример архитектуры:**
```
handlers -> services -> repositories
     \         |           /
      \        |          /
       \   interfaces    /
        \      |        /
         \     |       /
          \    |      /
            models
```

**Углубляющие вопросы:**
1. Как интерфейсы обеспечивают независимость слоев?
2. В каком пакете должны быть определены интерфейсы?
3. Как изменения в одном слое влияют на другие?
4. Стоит ли дублировать DTO между слоями?
5. Как обеспечить backward compatibility интерфейсов?

---

### 2.10 Как использовать empty interface эффективно?

**Углубляющие вопросы:**
1. Когда использование interface{} оправдано?
2. Как минимизировать использование interface{}?
3. Какие проблемы производительности связаны с interface{}?
4. Как interface{} влияет на garbage collection?
5. Какие альтернативы interface{} появились в Go 1.18+?

---

### 2.11 Объясните интерфейсы в контексте middleware pattern

**Пример из вашего кода:**
```go
func LoggerMiddleware04(
    tel model.ITelemetry,
    userbotWaPhoneNumber string,
    routerMiddleware05 func(ctx context.Context, evt interface{}) error,
) func(ctx context.Context, evt interface{}) error
```

**Углубляющие вопросы:**
1. Как можно типизировать middleware chain?
2. Какие интерфейсы помогут сделать middleware переиспользуемыми?
3. Как обеспечить правильный порядок middleware?
4. Можно ли использовать generics для middleware?
5. Как тестировать middleware chain?

---

### 2.12 Как правильно версионировать интерфейсы?

**Углубляющие вопросы:**
1. Что делать при breaking changes в интерфейсе?
2. Как добавлять новые методы без breaking changes?
3. Стоит ли создавать V2 интерфейсы?
4. Как миграция между версиями интерфейсов?
5. Какие best practices для API evolution в Go?

---

### 2.13 Объясните интерфейсы для error handling

**Пример:**
```go
type ValidationError interface {
    error
    Field() string
    Tag() string
}

type DatabaseError interface {
    error
    IsRetryable() bool
    Code() string
}
```

**Углубляющие вопросы:**
1. Как использовать type assertions для специфичных ошибок?
2. Когда создавать custom error interfaces?
3. Как композировать error interfaces?
4. Стоит ли интерфейсы ошибок наследовать от error?
5. Как обрабатывать wrapped errors с интерфейсами?

---

### 2.14 Как интерфейсы помогают в plugin architecture?

**Углубляющие вопросы:**
1. Как реализовать plugin system в Go?
2. Какие интерфейсы нужны для hot-swappable компонентов?
3. Как обеспечить безопасность при загрузке plugins?
4. Стоит ли использовать reflection для plugin discovery?
5. Какие альтернативы plugin architecture существуют?

---

### 2.15 Объясните performance implications интерфейсов

**Углубляющие вопросы:**
1. Как интерфейсы влияют на производительность?
2. Что такое method table lookup?
3. Когда compiler может заинлайнить вызовы через интерфейс?
4. Как измерить overhead интерфейсов?
5. В каких случаях стоит избегать интерфейсов ради производительности?

---

## 3. Clean Architecture и SOLID (15 вопросов)

### 3.1 Объясните архитектуру вашего проекта

**Структура проекта:**
```
crmessenger-whatsapp/
├── internal/
│   ├── controller/     # Presentation Layer
│   ├── service/       # Business Logic Layer  
│   ├── repo/          # Data Access Layer
│   └── model/         # Entities & Interfaces
├── infrastructure/    # External Dependencies
└── pkg/              # Shared Utilities
```

**Углубляющие вопросы:**
1. Как обеспечивается независимость слоев в этой архитектуре?
2. Почему интерфейсы определены в model пакете?
3. Как данная структура следует принципам Clean Architecture?
4. Какие проблемы могут возникнуть при росте проекта?
5. Как бы вы добавили новый тип контроллера (gRPC) в эту архитектуру?

---

### 3.2 Объясните принцип Single Responsibility

**Пример сервиса:**
```go
type ServiceAmocrmSource struct {
    amocrmSourceRepo model.IAmocrmSourceRepo
    amocrmClient     model.IAmocrmClient
    logger           model.OtelLogger
    meter            metric.Meter
    tracer           trace.Tracer
}

func (s *ServiceAmocrmSource) CreateAmocrmSource(ctx context.Context, /*...*/) error
func (s *ServiceAmocrmSource) SendMessageFromAmocrmToContact(ctx context.Context, /*...*/) error
func (s *ServiceAmocrmSource) SendMessageFromContactToAmocrm(ctx context.Context, /*...*/) error
```

**Углубляющие вопросы:**
1. Соблюдает ли этот сервис принцип SRP?
2. Какие обязанности можно выделить в отдельные сервисы?
3. Как разделить CRUD операции и бизнес-логику?
4. Стоит ли выносить телеметрию в отдельный concern?
5. Как обеспечить cohesion при разделении ответственностей?

---

### 3.3 Как применяется принцип Open/Closed?

**Пример middleware chain:**
```go
func IncludeMiddleware(userbot *whatsmeow.Client, userbotWaPhoneNumber string) {
    routerMiddleware05 := whatsappMiddleware.RouterMiddleware05(/* handlers */)
    loggerMiddleware04 := whatsappMiddleware.LoggerMiddleware04(tel, userbotWaPhoneNumber, routerMiddleware05)
    metricMiddleware03 := whatsappMiddleware.MetricMiddleware03(tel, userbotWaPhoneNumber, loggerMiddleware04)
    // ...
}
```

**Углубляющие вопросы:**
1. Как добавить новый middleware без изменения существующего кода?
2. Какой паттерн использован для композиции middleware?
3. Как обеспечить правильный порядок middleware?
4. Можно ли сделать middleware configurable?
5. Как тестировать цепочку middleware?

---

### 3.4 Объясните применение Liskov Substitution Principle

**Пример интерфейса:**
```go
type IDB interface {
    Insert(ctx context.Context, query string, args ...any) (any, error)
    Select(ctx context.Context, query string, args ...any) (pgx.Rows, error)
    // ...
}

type Postgres struct { /* postgres implementation */ }
type MySQL struct { /* mysql implementation */ }
```

**Углубляющие вопросы:**
1. Что нужно гарантировать при реализации IDB для MySQL?
2. Как обеспечить одинаковое поведение при ошибках?
3. Какие preconditions и postconditions должны соблюдаться?
4. Как тестировать LSP compliance?
5. Что делать с database-specific features?

---

### 3.5 Как реализован Interface Segregation Principle?

**Пример больших интерфейсов:**
```go
type IAmocrmSourceService interface {
    CreateAmocrmSource(ctx context.Context, amocrmPipelineID int, userbotWaPhoneNumber string) error
    DeleteAmocrmSource(ctx context.Context, amocrmToken, amocrmSubdomain string, amocrmSourceID int) error
    AmocrmSourceByUserbotWaPhoneNumber(ctx context.Context, userbotWaPhoneNumber string) ([]*AmocrmSource, error)
    NewChat(ctx context.Context, /*...*/) (int, string, string, int, error)
    ImportMessageFromUserbotToAmocrm(ctx context.Context, /*...*/) error
    SendMessageFromAmocrmToContact(ctx context.Context, /*...*/) error
    SendMessageFromContactToAmocrm(ctx context.Context, /*...*/) error
}
```

**Углубляющие вопросы:**
1. Как разделить этот интерфейс на более специфичные?
2. Какие клиенты используют какие методы?
3. Как группировать методы по функциональности?
4. Стоит ли создавать композитный интерфейс?
5. Как рефакторинг повлияет на существующий код?

---

### 3.6 Объясните Dependency Inversion в контексте слоев

**Пример зависимости:**
```go
// Service зависит от абстракции, а не от конкретной реализации
type ServiceAmocrmSource struct {
    amocrmSourceRepo model.IAmocrmSourceRepo  // абстракция
    amocrmClient     model.IAmocrmClient      // абстракция
}

// Конкретные реализации в других пакетах
type RepoAmocrmSource struct { /* postgres implementation */ }
type ClientAmocrm struct { /* HTTP client implementation */ }
```

**Углубляющие вопросы:**
1. Как высокоуровневые модули избегают зависимости от низкоуровневых?
2. Где должны быть определены абстракции?
3. Как это влияет на тестируемость?
4. Какие проблемы решает инверсия зависимостей?
5. Как обеспечить стабильность абстракций?

---

### 3.7 Как организованы слои в Clean Architecture?

**Пример слоев:**
```go
// Presentation Layer (controllers)
func SendMessageFromAmocrm(tel model.ITelemetry, amocrmSourceService model.IAmocrmSourceService) echo.HandlerFunc

// Business Logic Layer (services)  
func (s *ServiceAmocrmSource) SendMessageFromAmocrmToContact(ctx context.Context, /*...*/) error

// Data Access Layer (repositories)
func (r *RepoAmocrmSource) CreateAmocrmSource(ctx context.Context, /*...*/) error

// Infrastructure Layer
func (c *ClientAmocrm) CreateSource(ctx context.Context, /*...*/) (int, error)
```

**Углубляющие вопросы:**
1. Как обеспечивается направление зависимостей внутрь?
2. Какие данные передаются между слоями?
3. Где находится domain logic в этой архитектуре?
4. Как обрабатываются cross-cutting concerns?
5. Стоит ли выделять отдельный domain layer?

---

### 3.8 Объясните Entity и Value Objects

**Пример моделей:**
```go
type AmocrmSource struct {
    AmocrmSourceID       int       `db:"amocrm_source_id"`
    UserbotWaPhoneNumber string    `db:"userbot_wa_phone_number"`
    AmocrmPipelineID     int       `db:"amocrm_pipeline_id"`
    AmocrmExternalID     string    `db:"amocrm_external_id"`
    AmocrmScopeID        string    `db:"amocrm_scope_id"`
    CreatedAt            time.Time `db:"created_at"`
    UpdatedAt            time.Time `db:"updated_at"`
}
```

**Углубляющие вопросы:**
1. Является ли AmocrmSource entity или value object?
2. Какие бизнес-правила должны быть в этой структуре?
3. Как добавить методы поведения к этой модели?
4. Стоит ли разделять DB модели и domain модели?
5. Как обеспечить immutability для value objects?

---

### 3.9 Как реализованы Use Cases в вашей архитектуре?

**Пример use case:**
```go
func (s *ServiceAmocrmSource) SendMessageFromAmocrmToContact(ctx context.Context,
    text, amocrmExternalID, amocrmChatID, amocrmMessageID, amocrmContactName, contactWaPhoneNumber string,
    sendMessage func(ctx context.Context, userbotWaPhoneNumber, contactWaPhoneNumber, text string) error,
) error {
    // 1. Получить источник по external ID
    amocrmSource, err := s.amocrmSourceRepo.AmocrmSourceByAmocrmExternalID(ctx, amocrmExternalID)
    
    // 2. Найти или создать чат
    chat, err := s.amocrmSourceRepo.ChatByAmocrmChatID(ctx, amocrmChatID)
    
    // 3. Отправить сообщение через WhatsApp
    err = sendMessage(ctx, amocrmSource[0].UserbotWaPhoneNumber, contact[0].WaPhoneNumber, text)
    
    // 4. Сохранить сообщение в БД
    err = s.amocrmSourceRepo.CreateMessage(ctx, chat[0].ID, amocrmMessageID, "manager", text)
    
    return nil
}
```

**Углубляющие вопросы:**
1. Как выделить отдельные use cases из сервисов?
2. Где должна быть бизнес-логика валидации?
3. Как обрабатывать транзакции в use cases?
4. Стоит ли создавать отдельные структуры для каждого use case?
5. Как тестировать сложные use cases?

---

### 3.10 Объясните роль Repositories в Clean Architecture

**Пример repository:**
```go
type RepoAmocrmSource struct {
    db     model.IDB
    logger model.OtelLogger
    meter  metric.Meter
    tracer trace.Tracer
}

func (r *RepoAmocrmSource) CreateAmocrmSource(ctx context.Context, /*...*/) error {
    _, err := r.db.Insert(ctx, CreateAmocrmSource, args)
    return err
}
```

**Углубляющие вопросы:**
1. Что такое Repository pattern и как он реализован?
2. Как repository абстрагирует persistence layer?
3. Стоит ли repository содержать business logic?
4. Как обрабатывать сложные запросы через repository?
5. В чем разница между Repository и DAO pattern?

---

### 3.11 Как обеспечивается тестируемость архитектуры?

**Пример тестирования:**
```go
func TestServiceAmocrmSource_CreateAmocrmSource(t *testing.T) {
    // Mock dependencies
    mockRepo := &MockAmocrmSourceRepo{}
    mockClient := &MockAmocrmClient{}
    mockTel := &MockTelemetry{}
    
    service := amocrm.New(mockRepo, mockClient, mockTel)
    
    // Setup expectations
    mockClient.On("CreateSource", mock.Anything).Return(123, nil)
    mockRepo.On("CreateAmocrmSource", mock.Anything).Return(nil)
    
    // Execute
    err := service.CreateAmocrmSource(ctx, 1, "123456789")
    
    // Assert
    assert.NoError(t, err)
    mockClient.AssertExpectations(t)
}
```

**Углубляющие вопросы:**
1. Как архитектура облегчает unit testing?
2. Какие слои нужно мокировать?
3. Как тестировать integration между слоями?
4. Стоит ли использовать test doubles для всех зависимостей?
5. Как обеспечить test isolation?

---

### 3.12 Объясните обработку ошибок в Clean Architecture

**Пример из вашего кода:**
```go
func (s *ServiceAmocrmSource) CreateAmocrmSource(ctx context.Context, /*...*/) error {
    amocrmSourceID, err := s.amocrmClient.CreateSource(ctx, /*...*/)
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, err.Error())
        return err // domain error or infrastructure error?
    }
    
    err = s.amocrmSourceRepo.CreateAmocrmSource(ctx, /*...*/)
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, err.Error())
        return err
    }
    
    return nil
}
```

**Углубляющие вопросы:**
1. Как различать domain errors от infrastructure errors?
2. На каком слое должна происходить трансформация ошибок?
3. Как передавать контекст ошибки между слоями?
4. Стоит ли создавать domain-specific error types?
5. Где логировать ошибки в многослойной архитектуре?

---

### 3.13 Как реализовать Command Query Responsibility Segregation (CQRS)?

**Углубляющие вопросы:**
1. Как разделить read и write операции в вашем сервисе?
2. Какие преимущества CQRS в микросервисной архитектуре?
3. Как обеспечить консистентность данных при CQRS?
4. Стоит ли использовать разные модели для команд и запросов?
5. Как event sourcing сочетается с CQRS?

---

### 3.14 Объясните роль Domain Events

**Пример потенциального события:**
```go
type MessageSentEvent struct {
    MessageID            string
    FromUserbot          string  
    ToContact            string
    Text                 string
    AmocrmConversationID string
    Timestamp            time.Time
}
```

**Углубляющие вопросы:**
1. Как domain events помогают в декаплинге?
2. Где должны генерироваться domain events?
3. Как обеспечить доставку events?
4. Стоит ли использовать event store?
5. Как тестировать logic с domain events?

---

### 3.15 Как масштабировать Clean Architecture?

**Углубляющие вопросы:**
1. Как разбить монолит на микросервисы сохраняя архитектуру?
2. Какие bounded contexts можно выделить в вашем проекте?
3. Как обеспечить communication между сервисами?
4. Стоит ли дублировать common interfaces между сервисами?
5. Как версионировать API между сервисами?

---

## 4. Concurrency и Goroutines (15 вопросов)

### 4.1 Объясните горутины и планировщик Go

**Пример из вашего кода:**
```go
func (s *ServiceUserbot) CreateUserbot(ctx context.Context, accountID int) (*bytes.Buffer, error) {
    device := s.waDeviceDB.NewDevice()
    userbot := whatsmeow.NewClient(device, nil)

    qrChanForExport := make(chan string)
    go func() {  // горутина для обработки QR кода
        defer span.End()
        qrCtx := context.Background()
        qrChan, _ := userbot.GetQRChannel(qrCtx)

        err := userbot.Connect()
        if err != nil {
            s.logger.Error(ctx, err.Error())
            return
        }

        for evt := range qrChan {
            if evt.Event == "code" {
                qrChanForExport <- evt.Code
            } else {
                // обработка успешной авторизации
            }
        }
    }()
    
    qrData := <-qrChanForExport  // блокирующее чтение
    // ...
}
```

**Углубляющие вопросы:**
1. Как Go scheduler управляет горутинами?
2. Что такое M:N threading model в Go?
3. Какие проблемы могут возникнуть с этой горутиной?
4. Как обеспечить graceful shutdown горутины?
5. Когда горутины могут блокировать OS threads?

---

### 4.2 Как избежать race conditions?

**Пример потенциальной проблемы:**
```go
type ServiceUserbot struct {
    userbots map[string]*whatsmeow.Client  // concurrent access?
    // ...
}

func (s *ServiceUserbot) SendMessage(ctx context.Context, userbotWaPhoneNumber, contactWaPhoneNumber, text string) error {
    userbot := s.userbots[userbotWaPhoneNumber]  // race condition?
    // ...
}

func (s *ServiceUserbot) CreateUserbot(ctx context.Context, accountID int) (*bytes.Buffer, error) {
    // ...
    s.userbots[device.GetJID().User] = userbot  // race condition?
    // ...
}
```

**Углубляющие вопросы:**
1. Как детектировать race conditions в этом коде?
2. Какие способы синхронизации можно использовать?
3. Как использовать sync.RWMutex для оптимизации?
4. Когда использовать channels вместо mutex?
5. Как race detector помогает в разработке?

---

### 4.3 Объясните работу с каналами в producer-consumer pattern

**Пример реализации:**
```go
func ProcessMessages(messagesChan <-chan *events.Message, workerCount int) {
    for i := 0; i < workerCount; i++ {
        go func(workerID int) {
            for message := range messagesChan {
                // обработка сообщения
                processMessage(message)
            }
        }(i)
    }
}
```

**Углубляющие вопросы:**
1. Как правильно закрывать каналы в этом паттерне?
2. Что происходит при панике в одном из workers?
3. Как ограничить количество горутин при высокой нагрузке?
4. Как обеспечить graceful shutdown workers?
5. Стоит ли использовать buffered channels?

---

### 4.4 Как реализовать worker pool pattern?

**Углубляющие вопросы:**
1. Как создать ограниченный pool горутин?
2. Как балансировать между throughput и resource usage?
3. Как обрабатывать долго выполняющиеся задачи?
4. Что делать с failed jobs?
5. Как мониторить здоровье worker pool?

---

### 4.5 Объясните context и cancellation

**Пример из вашего кода:**
```go
func (pg *Postgres) Insert(ctx context.Context, query string, args ...any) (any, error) {
    var id any
    tx := pg.getTx(ctx)
    if tx != nil {
        ctx, span := pg.tracer.Start(ctx, "Postgres.TxInsert", trace.WithSpanKind(trace.SpanKindClient))
        defer span.End()

        err := tx.QueryRow(ctx, query, args...).Scan(&id)  // может быть отменен через context
        if err != nil {
            return 0, err
        }
    }
    return id, nil
}
```

**Углубляющие вопросы:**
1. Как context propagation работает в горутинах?
2. Что происходит с child contexts при отмене parent context?
3. Как правильно обрабатывать context.Done()?
4. Когда использовать context.WithTimeout vs context.WithDeadline?
5. Как тестировать код с context cancellation?

---

### 4.6 Как обеспечить thread safety для shared state?

**Пример:**
```go
type ConnectionPool struct {
    mu          sync.RWMutex
    connections map[string]*Connection
    maxSize     int
}

func (p *ConnectionPool) Get(key string) (*Connection, bool) {
    p.mu.RLock()
    defer p.mu.RUnlock()
    conn, exists := p.connections[key]
    return conn, exists
}

func (p *ConnectionPool) Set(key string, conn *Connection) error {
    p.mu.Lock()
    defer p.mu.Unlock()
    
    if len(p.connections) >= p.maxSize {
        return errors.New("pool is full")
    }
    
    p.connections[key] = conn
    return nil
}
```

**Углубляющие вопросы:**
1. Когда использовать sync.Mutex vs sync.RWMutex?
2. Как избежать deadlocks при multiple locks?
3. Стоит ли использовать sync.Map для concurrent access?
4. Как тестировать thread safety?
5. Какие альтернативы mutex существуют в Go?

---

### 4.7 Объясните паттерн Fan-in/Fan-out

**Пример реализации:**
```go
// Fan-out: распределение задач
func fanOut(input <-chan Task, workerCount int) []<-chan Result {
    outputs := make([]<-chan Result, workerCount)
    
    for i := 0; i < workerCount; i++ {
        output := make(chan Result)
        outputs[i] = output
        
        go func(out chan<- Result) {
            defer close(out)
            for task := range input {
                result := processTask(task)
                out <- result
            }
        }(output)
    }
    
    return outputs
}

// Fan-in: объединение результатов
func fanIn(inputs ...<-chan Result) <-chan Result {
    output := make(chan Result)
    var wg sync.WaitGroup
    
    for _, input := range inputs {
        wg.Add(1)
        go func(in <-chan Result) {
            defer wg.Done()
            for result := range in {
                output <- result
            }
        }(input)
    }
    
    go func() {
        wg.Wait()
        close(output)
    }()
    
    return output
}
```

**Углубляющие вопросы:**
1. Когда полезен Fan-out pattern?
2. Как обеспечить правильное закрытие каналов?
3. Что происходит при панике в одном из workers?
4. Как ограничить memory usage при большом количестве workers?
5. Стоит ли использовать buffered channels в Fan-in?

---

### 4.8 Как реализовать timeout для операций?

**Пример:**
```go
func (c *ClientAmocrm) requestWithTimeout(ctx context.Context, url string, timeout time.Duration) (*http.Response, error) {
    client := &http.Client{
        Timeout: timeout,
    }
    
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, err
    }
    
    return client.Do(req)
}

// Альтернативный подход с select
func processWithTimeout(data interface{}, timeout time.Duration) error {
    done := make(chan error, 1)
    
    go func() {
        done <- processData(data)
    }()
    
    select {
    case err := <-done:
        return err
    case <-time.After(timeout):
        return errors.New("operation timed out")
    }
}
```

**Углубляющие вопросы:**
1. В чем разница между этими подходами к timeout?
2. Что происходит с горутиной после timeout?
3. Как избежать goroutine leaks при timeout?
4. Когда использовать context.WithTimeout vs time.After?
5. Как обрабатывать частичные результаты при timeout?

---

### 4.9 Объясните synchronization primitives в Go

**Углубляющие вопросы:**
1. Когда использовать sync.Once?
2. Как работает sync.WaitGroup и его ограничения?
3. Что такое sync.Cond и когда его использовать?
4. Как sync.Pool помогает в memory management?
5. В чем разница между sync.Mutex и atomic operations?

---

### 4.10 Как обрабатывать панику в горутинах?

**Пример:**
```go
func safeWorker(work func()) {
    defer func() {
        if r := recover(); r != nil {
            log.Printf("Worker panic recovered: %v\n%s", r, debug.Stack())
        }
    }()
    
    work()
}

func WorkerPool(jobs <-chan func(), workerCount int) {
    for i := 0; i < workerCount; i++ {
        go func() {
            for job := range jobs {
                safeWorker(job)
            }
        }()
    }
}
```

**Углубляющие вопросы:**
1. Как панику в горутине влияет на main goroutine?
2. Стоит ли всегда восстанавливаться от паники?
3. Как правильно логировать панику?
4. Что делать с состоянием после recovery?
5. Как тестировать recovery logic?

---

### 4.11 Объясните memory model Go

**Углубляющие вопросы:**
1. Что такое happens-before relation в Go?
2. Как каналы обеспечивают memory synchronization?
3. Когда нужны memory barriers в Go?
4. Как init functions влияют на memory model?
5. Что такое data races и как их избежать?

---

### 4.12 Как реализовать rate limiting с горутинами?

**Пример:**
```go
type RateLimiter struct {
    rate   int
    burst  int
    tokens chan struct{}
}

func NewRateLimiter(rate, burst int) *RateLimiter {
    rl := &RateLimiter{
        rate:   rate,
        burst:  burst,
        tokens: make(chan struct{}, burst),
    }
    
    // Заполняем bucket
    for i := 0; i < burst; i++ {
        rl.tokens <- struct{}{}
    }
    
    // Пополняем tokens с заданной частотой
    go func() {
        ticker := time.NewTicker(time.Second / time.Duration(rate))
        defer ticker.Stop()
        
        for range ticker.C {
            select {
            case rl.tokens <- struct{}{}:
            default:
                // bucket полный
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
```

**Углубляющие вопросы:**
1. Как реализовать blocking rate limiter?
2. Что такое token bucket vs leaky bucket?
3. Как обеспечить fairness между горутинами?
4. Как настроить rate limiter для burst traffic?
5. Стоит ли использовать готовые библиотеки?

---

### 4.13 Как обеспечить graceful shutdown?

**Пример:**
```go
func (s *Server) Run(ctx context.Context) error {
    ctx, cancel := context.WithCancel(ctx)
    defer cancel()
    
    // Запускаем workers
    errChan := make(chan error, 1)
    
    go func() {
        errChan <- s.httpServer.ListenAndServe()
    }()
    
    go func() {
        errChan <- s.messageProcessor.Start(ctx)
    }()
    
    // Ждем сигнал завершения или ошибку
    select {
    case err := <-errChan:
        return err
    case <-ctx.Done():
        // Graceful shutdown
        shutdownCtx, shutdownCancel := context.WithTimeout(context.Background(), 30*time.Second)
        defer shutdownCancel()
        
        var wg sync.WaitGroup
        
        wg.Add(1)
        go func() {
            defer wg.Done()
            s.httpServer.Shutdown(shutdownCtx)
        }()
        
        wg.Add(1)
        go func() {
            defer wg.Done()
            s.messageProcessor.Stop(shutdownCtx)
        }()
        
        wg.Wait()
        return ctx.Err()
    }
}
```

**Углубляющие вопросы:**
1. Как координировать shutdown multiple services?
2. Что делать с in-flight requests при shutdown?
3. Как обеспечить timeout для graceful shutdown?
4. Стоит ли сохранять state при аварийном завершении?
5. Как тестировать graceful shutdown logic?

---

### 4.14 Объясните pipeline pattern с горутинами

**Пример:**
```go
// Stage 1: чтение данных
func readData(ctx context.Context) <-chan RawData {
    out := make(chan RawData)
    go func() {
        defer close(out)
        for {
            select {
            case <-ctx.Done():
                return
            default:
                data := fetchRawData()
                out <- data
            }
        }
    }()
    return out
}

// Stage 2: обработка данных
func processData(ctx context.Context, in <-chan RawData) <-chan ProcessedData {
    out := make(chan ProcessedData)
    go func() {
        defer close(out)
        for {
            select {
            case <-ctx.Done():
                return
            case data, ok := <-in:
                if !ok {
                    return
                }
                processed := transform(data)
                out <- processed
            }
        }
    }()
    return out
}

// Stage 3: сохранение данных
func saveData(ctx context.Context, in <-chan ProcessedData) {
    go func() {
        for {
            select {
            case <-ctx.Done():
                return
            case data, ok := <-in:
                if !ok {
                    return
                }
                store(data)
            }
        }
    }()
}
```

**Углубляющие вопросы:**
1. Как обеспечить backpressure в pipeline?
2. Что делать если один stage медленнее других?
3. Как обрабатывать ошибки в pipeline?
4. Стоит ли использовать buffered channels между stages?
5. Как масштабировать отдельные stages?

---

### 4.15 Как оптимизировать производительность горутин?

**Углубляющие вопросы:**
1. Сколько горутин оптимально создавать?
2. Как pool горутин влияет на производительность?
3. Когда горутины становятся bottleneck?
4. Как профилировать concurrent приложения?
5. Стоит ли ограничивать количество горутин по количеству CPU cores?

---

## 5. Типы данных и структуры (10 вопросов)

### 5.1 Объясните struct tags и их использование

**Пример из вашего кода:**
```go
type AmocrmSource struct {
    AmocrmSourceID       int       `db:"amocrm_source_id"`
    UserbotWaPhoneNumber string    `db:"userbot_wa_phone_number"`
    AmocrmPipelineID     int       `db:"amocrm_pipeline_id"`
    AmocrmExternalID     string    `db:"amocrm_external_id"`
    AmocrmScopeID        string    `db:"amocrm_scope_id"`
    CreatedAt            time.Time `db:"created_at"`
    UpdatedAt            time.Time `db:"updated_at"`
}

type CreateAmocrmSourceBody struct {
    UserbotWaPhoneNumber string `json:"userbot_wa_phone_number"`
    AmocrmPipelineID     int    `json:"amocrm_pipeline_id"`
}
```

**Углубляющие вопросы:**
1. Как парсятся struct tags в runtime?
2. Можно ли использовать несколько тегов для одного поля?
3. Как создать custom struct tag validator?
4. Влияют ли struct tags на производительность?
5. Как обрабатывать невалидные struct tags?

---

### 5.2 Объясните embedding и композицию

**Пример рефакторинга:**
```go
// Текущий подход
type RepoAmocrmSource struct {
    db     model.IDB
    logger model.OtelLogger
    meter  metric.Meter
    tracer trace.Tracer
}

// Альтернатива с embedding
type BaseRepo struct {
    db     model.IDB
    logger model.OtelLogger
    meter  metric.Meter
    tracer trace.Tracer
}

type RepoAmocrmSource struct {
    BaseRepo  // embedded
}
```

**Углубляющие вопросы:**
1. Когда использовать embedding vs explicit fields?
2. Как решается name conflict при embedding?
3. Как embedding влияет на интерфейсы?
4. Можно ли embed интерфейсы в структуры?
5. Как тестировать структуры с embedding?

---

### 5.3 Работа с JSON и custom marshaling

**Пример кастомной сериализации:**
```go
type Timestamp time.Time

func (t Timestamp) MarshalJSON() ([]byte, error) {
    return json.Marshal(time.Time(t).Unix())
}

func (t *Timestamp) UnmarshalJSON(data []byte) error {
    var unix int64
    if err := json.Unmarshal(data, &unix); err != nil {
        return err
    }
    *t = Timestamp(time.Unix(unix, 0))
    return nil
}
```

**Углубляющие вопросы:**
1. Когда нужна кастомная JSON сериализация?
2. Как обрабатывать nullable поля в JSON?
3. Что такое json.RawMessage и когда его использовать?
4. Как валидировать JSON при unmarshal?
5. Стоит ли использовать codegen для JSON?

---

### 5.4 Объясните maps и их внутреннее устройство

**Пример из вашего кода:**
```go
type ServiceUserbot struct {
    userbots map[string]*whatsmeow.Client  // concurrent access issue
    // ...
}
```

**Углубляющие вопросы:**
1.