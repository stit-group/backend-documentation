# Golang Backend Interview Guide
## Clean Architecture & DI Focus - Compact Edition

---

## 🎯 Interview Structure

**50 core questions** with practical examples and progressive difficulty. Focus on Clean Architecture, Dependency Injection, and real-world scenarios.

**Time allocation:** 60-90 minutes
- **Junior (0-2 years):** Questions 1-20
- **Middle (2-4 years):** Questions 1-35  
- **Senior (4+ years):** All questions

---

## 1. Language Fundamentals (10 questions)

### 1.1 Pointers and Memory Management
```go
func (pg *Postgres) Insert(ctx context.Context, query string, args ...any) (any, error) {
    var id any
    err := tx.QueryRow(ctx, query, args...).Scan(&id) // Why &id?
    return id, nil
}
```
**Key questions:**
- Why use pointer receiver `(pg *Postgres)`?
- When do pointers cause memory leaks?
- Explain Go's escape analysis

### 1.2 Interfaces and Duck Typing
```go
type IUserbotService interface {
    CreateUserbot(ctx context.Context, accountID int) (*bytes.Buffer, error)
    SendMessage(ctx context.Context, userbot, contact, text string) error
}
```
**Key questions:**
- How do interfaces enable DI?
- Interface segregation: split this interface into smaller ones
- Empty interface vs generics (Go 1.18+)

### 1.3 Error Handling Patterns
```go
func (r *RepoAmocrmSource) CreateAmocrmSource(ctx context.Context) error {
    _, err := r.db.Insert(ctx, query, args)
    if err != nil {
        r.logger.Error(ctx, err.Error())
        span.RecordError(err)
        return err // Should we wrap this error?
    }
    return nil
}
```
**Key questions:**
- When to wrap errors vs return as-is?
- `errors.Is()` vs `errors.As()` usage
- Should you log at every layer?

### 1.4 Context Usage Patterns
```go
func (pg *Postgres) CtxWithTx(ctx context.Context) (context.Context, error) {
    tx, err := pg.db.Begin(ctx)
    if err != nil {
        return nil, err
    }
    return context.WithValue(ctx, txKey{}, tx), nil
}
```
**Key questions:**
- Why use empty struct `txKey{}` as key?
- Context values vs function parameters - when to use what?
- How to handle context cancellation in DB operations?

### 1.5 Slices and Memory Sharing
```go
func appendProblem() {
    s1 := []int{1, 2, 3}
    s2 := append(s1, 4)
    s1[0] = 999
    fmt.Println(s2[0]) // What prints and why?
}
```

### 1.6 Struct Tags and Reflection
```go
type AmocrmSource struct {
    ID           int       `db:"id" json:"id"`
    PhoneNumber  string    `db:"phone" json:"phone" validate:"required,phone"`
    CreatedAt    time.Time `db:"created_at" json:"created_at"`
}
```
**Key questions:**
- How do struct tags work internally?
- Performance implications of reflection
- Custom validation tags

### 1.7 Method Sets and Receivers
```go
type Counter struct{ value int }

func (c Counter) Get() int      { return c.value }     // value receiver
func (c *Counter) Inc()         { c.value++ }          // pointer receiver

var c Counter
var p *Counter = &c
// Which method calls are valid? c.Inc(), p.Get(), etc.
```

### 1.8 Channels and Select
```go
func processWithTimeout(data chan string, timeout time.Duration) error {
    select {
    case result := <-data:
        return processResult(result)
    case <-time.After(timeout):
        return ErrTimeout
    }
}
```
**Key questions:**
- Buffered vs unbuffered channels
- Channel directions (`<-chan`, `chan<-`)
- Avoiding goroutine leaks

### 1.9 Defer and Panic Recovery
```go
func RecoverMiddleware(next func(evt interface{})) func(evt interface{}) {
    return func(evt interface{}) {
        defer func() {
            if r := recover(); r != nil {
                log.Printf("Panic: %v\n%s", r, debug.Stack())
            }
        }()
        next(evt)
    }
}
```

### 1.10 Type Assertions and Switches
```go
func handleEvent(evt interface{}) error {
    switch event := evt.(type) {
    case *MessageEvent:
        return handleMessage(event)
    case *JoinEvent:
        return handleJoin(event)
    default:
        return fmt.Errorf("unknown event type: %T", evt)
    }
}
```

---

## 2. Dependency Injection & Architecture (15 questions)

### 2.1 Manual DI vs Containers
```go
// Current approach
func main() {
    cfg := config.New()
    tel := telemetry.New(cfg)
    db := postgres.New(tel, cfg.DB)
    
    amocrmRepo := repository.NewAmocrm(db, tel)
    amocrmService := service.NewAmocrm(amocrmRepo, tel)
    
    server.Run(amocrmService, tel, cfg.Port)
}
```
**Key questions:**
- Pros/cons of manual DI vs DI containers (wire, fx)
- How to handle circular dependencies?
- Dependency lifecycle management

### 2.2 Interface Design Principles
```go
// Too big?
type IAmocrmService interface {
    CreateSource(ctx context.Context, pipelineID int, phone string) error
    DeleteSource(ctx context.Context, token, subdomain string, id int) error
    FindByPhone(ctx context.Context, phone string) ([]*AmocrmSource, error)
    CreateChat(ctx context.Context, sourceID int, contactID int) error
    SendMessage(ctx context.Context, chatID int, text string) error
    ImportMessage(ctx context.Context, messageID string) error
}

// Better approach?
type SourceManager interface {
    CreateSource(ctx context.Context, pipelineID int, phone string) error
    DeleteSource(ctx context.Context, token, subdomain string, id int) error
    FindByPhone(ctx context.Context, phone string) ([]*AmocrmSource, error)
}

type MessageHandler interface {
    SendMessage(ctx context.Context, chatID int, text string) error
    ImportMessage(ctx context.Context, messageID string) error
}
```

### 2.3 Repository Pattern Implementation
```go
type IAmocrmRepo interface {
    Create(ctx context.Context, source *AmocrmSource) error
    FindByID(ctx context.Context, id int) (*AmocrmSource, error)
    FindByPhone(ctx context.Context, phone string) ([]*AmocrmSource, error)
    Update(ctx context.Context, id int, updates map[string]interface{}) error
    Delete(ctx context.Context, id int) error
}

type PostgresAmocrmRepo struct {
    db     IDB
    logger Logger
}
```
**Key questions:**
- Repository vs DAO pattern differences
- Should repository contain business logic?
- How to handle complex queries and joins?

### 2.4 Clean Architecture Layers
```
┌─────────────────────────────────────┐
│           Controllers               │ ← HTTP handlers, gRPC servers
├─────────────────────────────────────┤
│           Use Cases                 │ ← Business logic orchestration  
├─────────────────────────────────────┤
│           Entities                  │ ← Domain models and rules
├─────────────────────────────────────┤
│           Repositories              │ ← Data access abstractions
└─────────────────────────────────────┘
│           Infrastructure            │ ← DB, HTTP clients, etc.
└─────────────────────────────────────┘
```

### 2.5 Dependency Inversion in Practice
```go
// Service depends on abstractions, not concretions
type AmocrmService struct {
    repo   IAmocrmRepo     // ← abstraction
    client IAmocrmClient   // ← abstraction
    logger Logger          // ← abstraction
}

// Concretions implement abstractions
type PostgresRepo struct { /* implementation */ }
type HTTPClient struct { /* implementation */ }
```

### 2.6 Domain Entities vs DTOs
```go
// Domain Entity
type AmocrmSource struct {
    id       int
    phone    string
    pipeline int
    
    // Business methods
    func (a *AmocrmSource) CanReceiveMessages() bool
    func (a *AmocrmSource) Validate() error
}

// DTO for API
type CreateSourceRequest struct {
    Phone    string `json:"phone" validate:"required"`
    Pipeline int    `json:"pipeline_id" validate:"required"`
}

// DTO for DB
type AmocrmSourceDB struct {
    ID       int    `db:"id"`
    Phone    string `db:"phone"`
    Pipeline int    `db:"pipeline_id"`
}
```

### 2.7 Error Handling Across Layers
```go
// Domain errors
type ErrSourceNotFound struct { ID int }
func (e ErrSourceNotFound) Error() string { return fmt.Sprintf("source %d not found", e.ID) }

// Infrastructure errors  
type ErrDatabaseConnection struct { Cause error }
func (e ErrDatabaseConnection) Error() string { return "database connection failed" }

// Service layer
func (s *AmocrmService) GetSource(ctx context.Context, id int) (*AmocrmSource, error) {
    source, err := s.repo.FindByID(ctx, id)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, ErrSourceNotFound{ID: id}  // Transform infra error to domain error
        }
        return nil, fmt.Errorf("failed to get source: %w", err)
    }
    return source, nil
}
```

### 2.8 Testing with Mocks
```go
//go:generate mockgen -source=interfaces.go -destination=mocks/mock_repo.go

func TestAmocrmService_CreateSource(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()
    
    mockRepo := mocks.NewMockIAmocrmRepo(ctrl)
    mockClient := mocks.NewMockIAmocrmClient(ctrl)
    
    service := NewAmocrmService(mockRepo, mockClient, logger)
    
    // Setup expectations
    mockClient.EXPECT().
        CreateSource(gomock.Any(), gomock.Any()).
        Return(123, nil)
    
    mockRepo.EXPECT().
        Create(gomock.Any(), gomock.Any()).
        Return(nil)
    
    // Execute and assert
    err := service.CreateSource(ctx, 1, "123456789")
    assert.NoError(t, err)
}
```

### 2.9 Configuration and Environment
```go
type Config struct {
    Database DatabaseConfig `yaml:"database"`
    Amocrm   AmocrmConfig   `yaml:"amocrm"`
    Server   ServerConfig   `yaml:"server"`
}

func NewConfig() (*Config, error) {
    cfg := &Config{}
    
    // Load from file
    if data, err := os.ReadFile("config.yaml"); err == nil {
        yaml.Unmarshal(data, cfg)
    }
    
    // Override with env vars
    if dbURL := os.Getenv("DATABASE_URL"); dbURL != "" {
        cfg.Database.URL = dbURL
    }
    
    return cfg, cfg.Validate()
}
```

### 2.10 Middleware Pattern
```go
type Middleware func(http.Handler) http.Handler

func ChainMiddleware(h http.Handler, middlewares ...Middleware) http.Handler {
    for i := len(middlewares) - 1; i >= 0; i-- {
        h = middlewares[i](h)
    }
    return h
}

func LoggingMiddleware(logger Logger) Middleware {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()
            next.ServeHTTP(w, r)
            logger.Info("request", "method", r.Method, "path", r.URL.Path, "duration", time.Since(start))
        })
    }
}
```

### 2.11-2.15 Rapid-Fire Architecture Questions:
- **2.11** How to implement CQRS in Go?
- **2.12** Event-driven architecture with channels vs message queues?
- **2.13** Microservices communication patterns (sync vs async)?
- **2.14** Database transaction management across layers?
- **2.15** API versioning strategies in Clean Architecture?

---

## 3. Concurrency & Performance (15 questions)

### 3.1 Goroutine Management and Leaks
```go
func (s *ServiceUserbot) CreateUserbot(ctx context.Context, accountID int) (*bytes.Buffer, error) {
    qrChan := make(chan string)
    
    go func() {  // Potential goroutine leak?
        defer close(qrChan)
        
        client := whatsmeow.NewClient(device, nil)
        qrCodeChan, _ := client.GetQRChannel(ctx)
        
        for evt := range qrCodeChan {
            if evt.Event == "code" {
                qrChan <- evt.Code
                return  // Goroutine exits here
            }
        }
    }()
    
    select {
    case qrCode := <-qrChan:
        return generateQRImage(qrCode), nil
    case <-ctx.Done():
        return nil, ctx.Err()  // But what about the goroutine?
    }
}
```
**Key questions:**
- How to prevent goroutine leaks?
- Proper context propagation to child goroutines
- When to use `defer close(channel)`?

### 3.2 Race Conditions and Synchronization
```go
type ConnectionPool struct {
    connections map[string]*Connection  // Race condition!
}

func (p *ConnectionPool) Get(key string) *Connection {
    return p.connections[key]  // Unsafe read
}

func (p *ConnectionPool) Set(key string, conn *Connection) {
    p.connections[key] = conn  // Unsafe write
}

// Fix with mutex or sync.Map?
```

### 3.3 Worker Pool Pattern
```go
type WorkerPool struct {
    jobs    chan Job
    results chan Result
    workers int
    wg      sync.WaitGroup
}

func (wp *WorkerPool) Start(ctx context.Context) {
    for i := 0; i < wp.workers; i++ {
        wp.wg.Add(1)
        go wp.worker(ctx)
    }
}

func (wp *WorkerPool) worker(ctx context.Context) {
    defer wp.wg.Done()
    for {
        select {
        case job, ok := <-wp.jobs:
            if !ok {
                return
            }
            result := job.Process()
            wp.results <- result
        case <-ctx.Done():
            return
        }
    }
}
```

### 3.4 Fan-in/Fan-out Pattern
```go
func Pipeline(input <-chan Task) <-chan Result {
    // Fan-out to 3 workers
    workers := 3
    channels := make([]<-chan Result, workers)
    
    for i := 0; i < workers; i++ {
        ch := make(chan Result)
        channels[i] = ch
        
        go func(out chan<- Result) {
            defer close(out)
            for task := range input {
                out <- processTask(task)
            }
        }(ch)
    }
    
    // Fan-in results
    return merge(channels...)
}
```

### 3.5 Context and Cancellation
```go
func (c *HTTPClient) RequestWithTimeout(ctx context.Context, url string) (*Response, error) {
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()
    
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, err
    }
    
    resp, err := c.client.Do(req)
    if err != nil {
        // How to distinguish timeout vs other errors?
        return nil, err
    }
    
    return resp, nil
}
```

### 3.6-3.10 Concurrency Scenarios:
- **3.6** Rate limiting with channels vs sync primitives
- **3.7** Graceful shutdown coordination
- **3.8** Panic recovery in goroutines
- **3.9** Memory consistency and happens-before
- **3.10** Channel patterns: pipeline, broadcast, multiplexing

### 3.11-3.15 Performance Questions:
- **3.11** Goroutine vs OS thread overhead
- **3.12** When to use sync.Pool for object reuse?
- **3.13** Channel buffer sizing strategies
- **3.14** Lock-free programming with atomic operations
- **3.15** Profiling concurrent applications (go tool pprof)

---

## 4. Advanced Topics (10 questions)

### 4.1 Database Connection Management
```go
type DB struct {
    *sql.DB
    maxOpenConns    int
    maxIdleConns    int
    connMaxLifetime time.Duration
}

func (db *DB) CtxWithTx(ctx context.Context) (context.Context, error) {
    tx, err := db.BeginTx(ctx, nil)
    if err != nil {
        return nil, err
    }
    return context.WithValue(ctx, txKey{}, tx), nil
}

func (db *DB) CommitTx(ctx context.Context) error {
    tx := getTxFromCtx(ctx)
    if tx == nil {
        return errors.New("no transaction in context")
    }
    return tx.Commit()
}
```
**Key questions:**
- Connection pooling best practices
- Transaction management across service boundaries
- Handling distributed transactions

### 4.2 HTTP Client Optimization
```go
type HTTPClient struct {
    client *http.Client
    baseURL string
}

func NewHTTPClient(timeout time.Duration) *HTTPClient {
    return &HTTPClient{
        client: &http.Client{
            Timeout: timeout,
            Transport: &http.Transport{
                MaxIdleConns:        100,
                MaxIdleConnsPerHost: 10,
                IdleConnTimeout:     90 * time.Second,
            },
        },
    }
}
```

### 4.3 Metrics and Observability
```go
type MetricsMiddleware struct {
    requestDuration prometheus.HistogramVec
    requestsTotal   prometheus.CounterVec
}

func (m *MetricsMiddleware) Handler(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        
        recorder := &statusRecorder{ResponseWriter: w, status: 200}
        next.ServeHTTP(recorder, r)
        
        duration := time.Since(start)
        
        m.requestDuration.WithLabelValues(r.Method, r.URL.Path).Observe(duration.Seconds())
        m.requestsTotal.WithLabelValues(r.Method, r.URL.Path, fmt.Sprintf("%d", recorder.status)).Inc()
    })
}
```

### 4.4 Custom JSON Marshaling
```go
type FlexibleTime struct {
    time.Time
}

func (ft *FlexibleTime) UnmarshalJSON(data []byte) error {
    formats := []string{
        time.RFC3339,
        "2006-01-02 15:04:05",
        "2006-01-02",
    }
    
    str := strings.Trim(string(data), `"`)
    
    for _, format := range formats {
        if t, err := time.Parse(format, str); err == nil {
            ft.Time = t
            return nil
        }
    }
    
    return fmt.Errorf("unable to parse time: %s", str)
}
```

### 4.5 Generics in Business Logic
```go
// Go 1.18+
type Repository[T any, ID comparable] interface {
    Create(ctx context.Context, entity T) (ID, error)
    GetByID(ctx context.Context, id ID) (T, error)
    Update(ctx context.Context, id ID, entity T) error
    Delete(ctx context.Context, id ID) error
    List(ctx context.Context, filter Filter) ([]T, error)
}

type PostgresRepository[T any, ID comparable] struct {
    db        *sql.DB
    tableName string
}

func (r *PostgresRepository[T, ID]) Create(ctx context.Context, entity T) (ID, error) {
    // Generic implementation
}
```

### 4.6-4.10 Expert Level:
- **4.6** Memory profiling and optimization
- **4.7** Custom binary protocols vs JSON/gRPC
- **4.8** Plugin architecture with Go modules
- **4.9** Code generation tools (protobuf, sqlc, etc.)
- **4.10** Go module versioning and backwards compatibility

---

## 🚀 Practical Exercises

### Exercise 1: Refactor to Clean Architecture (20 min)
Given a monolithic handler, separate it into proper layers:

```go
// Bad: Everything in handler
func CreateUser(w http.ResponseWriter, r *http.Request) {
    var req CreateUserRequest
    json.NewDecoder(r.Body).Decode(&req)
    
    // Validation
    if req.Email == "" {
        http.Error(w, "email required", 400)
        return
    }
    
    // Business logic
    hashedPassword := bcrypt.GenerateFromPassword(req.Password, 10)
    
    // Database
    db := getDBConnection()
    _, err := db.Exec("INSERT INTO users (email, password) VALUES (?, ?)", req.Email, hashedPassword)
    
    // Response
    json.NewEncoder(w).Encode(map[string]string{"status": "created"})
}
```

### Exercise 2: Fix Concurrency Issues (15 min)
```go
type Cache struct {
    data map[string]interface{}
}

func (c *Cache) Get(key string) interface{} {
    return c.data[key]
}

func (c *Cache) Set(key string, value interface{}) {
    c.data[key] = value
}

func (c *Cache) Delete(key string) {
    delete(c.data, key)
}
```

### Exercise 3: Design API with Proper Error Handling (10 min)
Design a REST API endpoint that:
- Validates input
- Handles business logic errors
- Manages database errors  
- Returns proper HTTP status codes
- Logs appropriately

---

## 📊 Evaluation Rubric

### Junior Developer (0-2 years)
- ✅ Understands basic Go syntax and concepts
- ✅ Can explain pointers, slices, maps
- ✅ Basic error handling patterns
- ✅ Simple goroutine usage
- ✅ Interface basics

### Middle Developer (2-4 years)  
- ✅ All junior requirements
- ✅ Clean Architecture principles
- ✅ Dependency injection patterns
- ✅ Concurrency patterns and race conditions
- ✅ Testing strategies with mocks
- ✅ Performance considerations

### Senior Developer (4+ years)
- ✅ All middle requirements
- ✅ System design and scalability
- ✅ Advanced concurrency patterns
- ✅ Performance optimization
- ✅ Observability and monitoring
- ✅ Code review and mentoring capabilities

---

## 🎯 Red Flags to Watch For

1. **Can't explain pointer receivers vs value receivers**
2. **Doesn't understand interface{} vs generics trade-offs**
3. **Creates goroutines without considering lifecycle**
4. **Logs errors at every layer without context**
5. **Uses big interfaces violating ISP**
6. **Can't identify race conditions in simple code**
7. **Doesn't understand dependency direction in Clean Architecture**
8. **Copies slices incorrectly causing memory issues**

---

*This guide focuses on practical, real-world scenarios rather than theoretical knowledge. Adjust question difficulty based on candidate level and role requirements.*