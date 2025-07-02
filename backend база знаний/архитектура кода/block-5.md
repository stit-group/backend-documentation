# Блок 5: Управление зависимостями и модульность

---

## 🎯 Цели блока

После изучения этого блока вы сможете:
- ✅ Применять принципы инверсии управления и внедрения зависимостей
- ✅ Создавать слабосвязанные, легко тестируемые системы
- ✅ Проектировать модульную архитектуру
- ✅ Эффективно управлять конфигурацией приложений
- ✅ Рефакторить legacy код

---

## 📚 Содержание

1. [Dependency Injection и IoC контейнеры](#1-dependency-injection-и-ioc-контейнеры)
2. [Модульность и разделение ответственности](#2-модульность-и-разделение-ответственности)
3. [Управление конфигурацией](#3-управление-конфигурацией)
4. [Тестирование архитектуры](#4-тестирование-архитектуры)
5. [Рефакторинг legacy кода](#5-рефакторинг-legacy-кода)

---

## 1. Dependency Injection и IoC контейнеры

### 🔍 Что такое Dependency Injection?

**Dependency Injection (DI)** — это паттерн проектирования, который реализует принцип Inversion of Control (IoC). Вместо того чтобы объект сам создавал свои зависимости, они передаются ему извне.

### ❌ Проблема: Тесная связанность

```java
// ПЛОХО: Тесная связанность
public class OrderService {
    private EmailService emailService;
    private PaymentService paymentService;
    
    public OrderService() {
        this.emailService = new EmailService(); // Жестко задана зависимость
        this.paymentService = new PaymentService(); // Жестко задана зависимость
    }
    
    public void processOrder(Order order) {
        paymentService.processPayment(order.getAmount());
        emailService.sendConfirmation(order.getCustomerEmail());
    }
}
```

**Проблемы этого подхода:**
- Сложно тестировать (нельзя подставить моки)
- Нарушается принцип DIP (зависимость от конкретных классов)
- Сложно изменить реализацию зависимостей

### ✅ Решение: Dependency Injection

```java
// ХОРОШО: Dependency Injection
public class OrderService {
    private final EmailService emailService;
    private final PaymentService paymentService;
    
    // Constructor Injection
    public OrderService(EmailService emailService, PaymentService paymentService) {
        this.emailService = emailService;
        this.paymentService = paymentService;
    }
    
    public void processOrder(Order order) {
        paymentService.processPayment(order.getAmount());
        emailService.sendConfirmation(order.getCustomerEmail());
    }
}
```

### 📊 Типы внедрения зависимостей

```
┌─────────────────────────────────────────────────────────────┐
│                    ТИПЫ DEPENDENCY INJECTION                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. CONSTRUCTOR INJECTION (Рекомендуется)                   │
│     ┌─────────────────────────────────────────────────┐     │
│     │ public MyClass(Dependency dep) {                │     │
│     │     this.dependency = dep;                      │     │
│     │ }                                               │     │
│     └─────────────────────────────────────────────────┘     │
│     ✅ Обязательные зависимости                             │
│     ✅ Immutable объекты                                    │
│     ✅ Fail-fast при отсутствии зависимостей               │
│                                                             │
│  2. SETTER INJECTION                                        │
│     ┌─────────────────────────────────────────────────┐     │
│     │ public void setDependency(Dependency dep) {     │     │
│     │     this.dependency = dep;                      │     │
│     │ }                                               │     │
│     └─────────────────────────────────────────────────┘     │
│     ✅ Опциональные зависимости                             │
│     ❌ Mutable объекты                                      │
│                                                             │
│  3. INTERFACE INJECTION                                     │
│     ┌─────────────────────────────────────────────────┐     │
│     │ interface Injectable {                          │     │
│     │     void injectDependency(Dependency dep);     │     │
│     │ }                                               │     │
│     └─────────────────────────────────────────────────┘     │
│     ❌ Редко используется                                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 🏗️ IoC Контейнеры

**IoC Container** — это фреймворк, который автоматически управляет созданием объектов и внедрением зависимостей.

```
┌──────────────────────────────────────────────────────────────┐
│                    IOC CONTAINER WORKFLOW                    │
└──────────────────────────────────────────────────────────────┘
                                
    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
    │   Config    │    │  Container  │    │   Client    │
    │             │    │             │    │             │
    └──────┬──────┘    └──────┬──────┘    └──────┬──────┘
           │                  │                  │
           │ 1. Register      │                  │
           │ Dependencies     │                  │
           ├─────────────────→│                  │
           │                  │                  │
           │                  │ 2. Request       │
           │                  │ Instance         │
           │                  │←─────────────────┤
           │                  │                  │
           │                  │ 3. Create &      │
           │                  │ Inject           │
           │                  │                  │
           │                  │ 4. Return        │
           │                  │ Instance         │
           │                  ├─────────────────→│
           │                  │                  │
```

### 💡 Пример с IoC контейнером (Spring)

```java
// Конфигурация контейнера
@Configuration
public class AppConfig {
    
    @Bean
    public EmailService emailService() {
        return new SmtpEmailService();
    }
    
    @Bean
    public PaymentService paymentService() {
        return new StripePaymentService();
    }
    
    @Bean
    public OrderService orderService(EmailService emailService, 
                                   PaymentService paymentService) {
        return new OrderService(emailService, paymentService);
    }
}

// Использование
@Component
public class OrderController {
    private final OrderService orderService;
    
    @Autowired
    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }
    
    @PostMapping("/orders")
    public ResponseEntity<String> createOrder(@RequestBody Order order) {
        orderService.processOrder(order);
        return ResponseEntity.ok("Order processed");
    }
}
```

### 🆚 Service Locator vs Dependency Injection

```
┌─────────────────────────────────────────────────────────────┐
│                SERVICE LOCATOR vs DI                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SERVICE LOCATOR (Антипаттерн)                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ public class OrderService {                         │    │
│  │     public void processOrder(Order order) {        │    │
│  │         EmailService email =                       │    │
│  │             ServiceLocator.get(EmailService.class);│    │
│  │         PaymentService payment =                   │    │
│  │             ServiceLocator.get(PaymentService.class);│   │
│  │         // использование сервисов                  │    │
│  │     }                                               │    │
│  │ }                                                   │    │
│  └─────────────────────────────────────────────────────┘    │
│  ❌ Скрытые зависимости                                     │
│  ❌ Сложно тестировать                                      │
│  ❌ Нарушает принцип явности                                │
│                                                             │
│  DEPENDENCY INJECTION (Рекомендуется)                       │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ public class OrderService {                         │    │
│  │     private final EmailService emailService;       │    │
│  │     private final PaymentService paymentService;   │    │
│  │                                                     │    │
│  │     public OrderService(EmailService email,        │    │
│  │                        PaymentService payment) {   │    │
│  │         this.emailService = email;                 │    │
│  │         this.paymentService = payment;             │    │
│  │     }                                               │    │
│  │ }                                                   │    │
│  └─────────────────────────────────────────────────────┘    │
│  ✅ Явные зависимости                                       │
│  ✅ Легко тестировать                                       │
│  ✅ Следует принципу явности                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 🎯 Практическое задание 1

**Задача:** Рефакторинг кода с использованием DI

```java
// Исходный код (требует рефакторинга)
public class UserService {
    private UserRepository userRepository;
    private EmailService emailService;
    private PasswordEncoder passwordEncoder;
    
    public UserService() {
        this.userRepository = new DatabaseUserRepository();
        this.emailService = new SmtpEmailService();
        this.passwordEncoder = new BCryptPasswordEncoder();
    }
    
    public User createUser(String email, String password) {
        String encodedPassword = passwordEncoder.encode(password);
        User user = new User(email, encodedPassword);
        User savedUser = userRepository.save(user);
        emailService.sendWelcomeEmail(savedUser.getEmail());
        return savedUser;
    }
}
```

**Ваша задача:**
1. Создайте интерфейсы для всех зависимостей
2. Примените Constructor Injection
3. Создайте конфигурацию для IoC контейнера
4. Напишите unit тест с моками

---

## 2. Модульность и разделение ответственности

### 🧩 Принципы модульного дизайна

**Модуль** — это самостоятельная единица программного обеспечения, которая инкапсулирует определенную функциональность и имеет четко определенный интерфейс.

### 📊 Coupling vs Cohesion

```
┌─────────────────────────────────────────────────────────────┐
│                    COUPLING & COHESION                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  COUPLING (Связанность) - ДОЛЖНА БЫТЬ НИЗКОЙ                │
│                                                             │
│  ┌─────────┐       ┌─────────┐       ┌─────────┐            │
│  │ Module A│──────→│ Module B│──────→│ Module C│            │
│  └─────────┘       └─────────┘       └─────────┘            │
│       ↓                 ↓                 ↓                 │
│   Тесная связь    Средняя связь    Слабая связь            │
│   (избегать)      (осторожно)      (стремиться)            │
│                                                             │
│  COHESION (Сплоченность) - ДОЛЖНА БЫТЬ ВЫСОКОЙ              │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                   MODULE                            │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐             │    │
│  │  │Function │  │Function │  │Function │             │    │
│  │  │    A    │  │    B    │  │    C    │             │    │
│  │  └─────────┘  └─────────┘  └─────────┘             │    │
│  │       ↑           ↑           ↑                     │    │
│  │   Все функции работают вместе для                   │    │
│  │   достижения одной цели                             │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 🏗️ Типы модулей

```
┌─────────────────────────────────────────────────────────────┐
│                      ТИПЫ МОДУЛЕЙ                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. FEATURE MODULES (По функциональности)                   │
│     ┌─────────────────────────────────────────────────┐     │
│     │ user-management/                                │     │
│     │ ├── controllers/                                │     │
│     │ ├── services/                                   │     │
│     │ ├── repositories/                               │     │
│     │ └── models/                                     │     │
│     └─────────────────────────────────────────────────┘     │
│                                                             │
│  2. LAYER MODULES (По слоям)                                │
│     ┌─────────────────────────────────────────────────┐     │
│     │ presentation/                                   │     │
│     │ business/                                       │     │
│     │ data-access/                                    │     │
│     │ infrastructure/                                 │     │
│     └─────────────────────────────────────────────────┘     │
│                                                             │
│  3. SHARED MODULES (Общие компоненты)                       │
│     ┌─────────────────────────────────────────────────┐     │
│     │ common/                                         │     │
│     │ ├── utils/                                      │     │
│     │ ├── validators/                                 │     │
│     │ ├── exceptions/                                 │     │
│     │ └── constants/                                  │     │
│     └─────────────────────────────────────────────────┘     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 📐 Модульная архитектура e-commerce системы

```
┌─────────────────────────────────────────────────────────────┐
│                 E-COMMERCE MODULES                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│  │   USER      │    │   PRODUCT   │    │   ORDER     │      │
│  │   MODULE    │    │   MODULE    │    │   MODULE    │      │
│  │             │    │             │    │             │      │
│  │ • Register  │    │ • Catalog   │    │ • Cart      │      │
│  │ • Login     │    │ • Search    │    │ • Checkout  │      │
│  │ • Profile   │    │ • Reviews   │    │ • Payment   │      │
│  └─────────────┘    └─────────────┘    └─────────────┘      │
│         │                   │                   │           │
│         └───────────────────┼───────────────────┘           │
│                             │                               │
│                   ┌─────────────┐                           │
│                   │   SHARED    │                           │
│                   │   MODULE    │                           │
│                   │             │                           │
│                   │ • Security  │                           │
│                   │ • Logging   │                           │
│                   │ • Config    │                           │
│                   │ • Utils     │                           │
│                   └─────────────┘                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 🔗 Управление зависимостями между модулями

```java
// ХОРОШО: Слабая связанность через интерфейсы
public interface UserNotificationService {
    void sendWelcomeEmail(String userEmail);
    void sendPasswordResetEmail(String userEmail, String resetToken);
}

// User Module
@Component
public class UserService {
    private final UserRepository userRepository;
    private final UserNotificationService notificationService;
    
    public UserService(UserRepository userRepository, 
                      UserNotificationService notificationService) {
        this.userRepository = userRepository;
        this.notificationService = notificationService;
    }
    
    public User createUser(CreateUserRequest request) {
        User user = userRepository.save(new User(request));
        notificationService.sendWelcomeEmail(user.getEmail());
        return user;
    }
}

// Notification Module - реализация интерфейса
@Component
public class EmailNotificationService implements UserNotificationService {
    @Override
    public void sendWelcomeEmail(String userEmail) {
        // реализация отправки email
    }
    
    @Override
    public void sendPasswordResetEmail(String userEmail, String resetToken) {
        // реализация отправки email
    }
}
```

### 🎯 Практическое задание 2

**Задача:** Проектирование модульной структуры

Спроектируйте модульную архитектуру для системы управления библиотекой со следующими требованиями:
- Управление книгами (добавление, поиск, категории)
- Управление пользователями (регистрация, профили)
- Система аренды (выдача, возврат, штрафы)
- Уведомления (email, SMS)
- Отчеты (статистика, аналитика)

**Требуется:**
1. Определить модули и их ответственности
2. Спроектировать интерфейсы взаимодействия
3. Показать диаграмму зависимостей
4. Создать структуру пакетов

---

## 3. Управление конфигурацией

### ⚙️ Принципы управления конфигурацией

**Конфигурация** — это параметры приложения, которые могут изменяться в зависимости от окружения (dev, test, prod) без перекомпиляции кода.

### 📊 Типы конфигурационных данных

```
┌─────────────────────────────────────────────────────────────┐
│                 ТИПЫ КОНФИГУРАЦИИ                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. ENVIRONMENT-SPECIFIC (Зависящие от окружения)           │
│     ┌─────────────────────────────────────────────────┐     │
│     │ • Database URLs                                 │     │
│     │ • API endpoints                                 │     │
│     │ • Cache settings                                │     │
│     │ • Feature flags                                 │     │
│     └─────────────────────────────────────────────────┘     │
│                                                             │
│  2. SECRETS (Секретные данные)                              │
│     ┌─────────────────────────────────────────────────┐     │
│     │ • Database passwords                            │     │
│     │ • API keys                                      │     │
│     │ • JWT secrets                                   │     │
│     │ • Certificates                                  │     │
│     └─────────────────────────────────────────────────┘     │
│                                                             │
│  3. APPLICATION SETTINGS (Настройки приложения)             │
│     ┌─────────────────────────────────────────────────┐     │
│     │ • Thread pool sizes                             │     │
│     │ • Timeout values                                │     │
│     │ • Cache TTL                                     │     │
│     │ • Batch sizes                                   │     │
│     └─────────────────────────────────────────────────┘     │
│                                                             │
│  4. BUSINESS RULES (Бизнес-правила)                         │
│     ┌─────────────────────────────────────────────────┐     │
│     │ • Tax rates                                     │     │
│     │ • Commission percentages                        │     │
│     │ • Business hours                                │     │
│     │ • Validation rules                              │     │
│     └─────────────────────────────────────────────────┘     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 🏗️ Иерархия конфигурации

```
┌─────────────────────────────────────────────────────────────┐
│               CONFIGURATION HIERARCHY                       │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                 PRIORITY                            │    │
│  │                                                     │    │
│  │  1. Command Line Arguments      (Highest)           │    │
│  │     java -jar app.jar --server.port=9090           │    │
│  │                                                     │    │
│  │  2. Environment Variables                           │    │
│  │     export SERVER_PORT=8080                        │    │
│  │                                                     │    │
│  │  3. External Config Files                           │    │
│  │     /etc/myapp/application.properties               │    │
│  │                                                     │    │
│  │  4. Profile-specific Properties                     │    │
│  │     application-prod.properties                     │    │
│  │                                                     │    │
│  │  5. Application Properties      (Lowest)            │    │
│  │     application.properties                          │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  Более высокий приоритет переопределяет более низкий        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 💾 Способы хранения конфигурации

```java
// 1. Properties файлы
// application.properties
server.port=8080
database.url=jdbc:mysql://localhost:3306/mydb
database.username=user
cache.ttl=3600

// 2. YAML файлы (более читаемые)
// application.yml
server:
  port: 8080
database:
  url: jdbc:mysql://localhost:3306/mydb
  username: user
cache:
  ttl: 3600

// 3. Environment Variables
// export DATABASE_URL=jdbc:mysql://prod-db:3306/mydb
// export DATABASE_USERNAME=prod_user

// 4. Внешние конфигурационные сервисы
// Spring Cloud Config, Consul, etcd
```

### 🔧 Паттерн Configuration Object

```java
// Конфигурационный класс
@ConfigurationProperties(prefix = "database")
public class DatabaseConfig {
    private String url;
    private String username;
    private String password;
    private int maxConnections = 10;
    private Duration connectionTimeout = Duration.ofSeconds(30);
    
    // getters and setters
    public String getUrl() { return url; }
    public void setUrl(String url) { this.url = url; }
    
    // validation
    @PostConstruct
    public void validate() {
        if (url == null || url.isEmpty()) {
            throw new IllegalStateException("Database URL must be configured");
        }
    }
}

// Использование
@Service
public class DatabaseService {
    private final DatabaseConfig databaseConfig;
    
    public DatabaseService(DatabaseConfig databaseConfig) {
        this.databaseConfig = databaseConfig;
    }
    
    public Connection getConnection() {
        return DriverManager.getConnection(
            databaseConfig.getUrl(),
            databaseConfig.getUsername(),
            databaseConfig.getPassword()
        );
    }
}
```

### 🔐 Безопасное управление секретами

```
┌─────────────────────────────────────────────────────────────┐
│                   SECRETS MANAGEMENT                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ❌ ПЛОХИЕ ПРАКТИКИ:                                        │
│     • Хранение в коде                                       │
│     • Коммит в VCS                                          │
│     • Открытый текст в конфигах                             │
│     • Логирование секретов                                  │
│                                                             │
│  ✅ ХОРОШИЕ ПРАКТИКИ:                                       │
│                                                             │
│  1. Environment Variables                                    │
│     ┌─────────────────────────────────────────────────┐     │
│     │ export DB_PASSWORD=secret123                    │     │
│     │ String password = System.getenv("DB_PASSWORD"); │     │
│     └─────────────────────────────────────────────────┘     │
│                                                             │
│  2. External Secret Stores                                  │
│     ┌─────────────────────────────────────────────────┐     │
│     │ • HashiCorp Vault                               │     │
│     │ • AWS Secrets Manager                           │     │
│     │ • Azure Key Vault                               │     │
│     │ • Kubernetes Secrets                            │     │
│     └─────────────────────────────────────────────────┘     │
│                                                             │
│  3. Encrypted Configuration                                 │
│     ┌─────────────────────────────────────────────────┐     │
│     │ • SOPS (Secrets OPerationS)                     │     │
│     │ • git-crypt                                     │     │
│     │ • Ansible Vault                                 │     │
│     └─────────────────────────────────────────────────┘     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 🎯 Практическое задание 3

**Задача:** Создание системы конфигурации

Создайте конфигурационную систему для микросервиса с поддержкой:
1. Разных окружений (dev, staging, prod)
2. Безопасного хранения секретов
3. Hot reload конфигурации
4. Валидации параметров

```java
// Ваша задача: реализовать ConfigurationManager
public interface ConfigurationManager {
    <T> T getConfig(String key, Class<T> type);
    <T> T getConfig(String key, Class<T> type, T defaultValue);
    void refreshConfiguration();
    void addConfigurationChangeListener(ConfigurationChangeListener listener);
}
```

---

## 4. Тестирование архитектуры

### 🧪 Пирамида тестирования

```
┌─────────────────────────────────────────────────────────────┐
│                    TESTING PYRAMID                          │
│                                                             │
│                        ┌─────┐                              │
│                        │ E2E │ - Дорогие, медленные         │
│                        └─────┘   Мало тестов               │
│                      ┌───────────┐                          │
│                      │Integration│ - Средние по стоимости   │
│                      │   Tests   │   Средние по скорости    │
│                      └───────────┘   Умеренное количество   │
│                  ┌───────────────────┐                      │
│                  │    Unit Tests     │ - Быстрые, дешевые   │
│                  │                   │   Много тестов       │
│                  └───────────────────┘                      │
│                                                             │
│  ПРИНЦИП: Больше быстрых unit тестов,                       │
│           меньше медленных E2E тестов                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 🎭 Тестовые дублеры (Test Doubles)

```java
// 1. STUB - возвращает заранее определенные данные
public class EmailServiceStub implements EmailService {
    @Override
    public boolean sendEmail(String to, String subject, String body) {
        return true; // всегда успешно
    }
}

// 2. MOCK - проверяет взаимодействия
@Test
public void shouldSendWelcomeEmailWhenUserCreated() {
    // Given
    EmailService emailMock = mock(EmailService.class);
    UserService userService = new UserService(userRepository, emailMock);
    
    // When
    User user = userService.createUser("test@example.com", "password");
    
    // Then
    verify(emailMock).sendEmail(
        eq("test@example.com"),
        eq("Welcome!"),
        anyString()
    );
}

// 3. SPY - частичное мокирование
@Test
public void shouldLogEmailSending() {
    // Given
    EmailService emailSpy = spy(new RealEmailService());
    doReturn(true).when(emailSpy).sendEmail(anyString(), anyString(), anyString());
    
    // When
    boolean result = emailSpy.sendEmail("test@example.com", "Subject", "Body");
    
    // Then
    assertTrue(result);
    verify(emailSpy).sendEmail(anyString(), anyString(), anyString());
}
```

### 🏗️ Тестирование с Dependency Injection

```java
// Тестируемый класс
@Service
public class OrderService {
    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    private final NotificationService notificationService;
    
    public OrderService(PaymentService paymentService,
                       InventoryService inventoryService,
                       NotificationService notificationService) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
        this.notificationService = notificationService;
    }
    
    public Order processOrder(OrderRequest request) {
        // Проверяем наличие товара
        if (!inventoryService.isAvailable(request.getProductId(), request.getQuantity())) {
            throw new OutOfStockException("Product not available");
        }
        
        // Обрабатываем платеж
        PaymentResult paymentResult = paymentService.processPayment(request.getPayment());
        if (!paymentResult.isSuccessful()) {
            throw new PaymentException("Payment failed");
        }
        
        // Резервируем товар
        inventoryService.reserve(request.getProductId(), request.getQuantity());
        
        // Создаем заказ
        Order order = new Order(request, paymentResult.getTransactionId());
        
        // Отправляем уведомление
        notificationService.sendOrderConfirmation(order);
        
        return order;
    }
}

// Unit тест
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    
    @Mock private PaymentService paymentService;
    @Mock private InventoryService inventoryService;
    @Mock private NotificationService notificationService;
    
    @InjectMocks private OrderService orderService;
    
    @Test
    void shouldProcessOrderSuccessfully() {
        // Given
        OrderRequest request = createOrderRequest();
        PaymentResult successfulPayment = new PaymentResult(true, "tx123");
        
        when(inventoryService.isAvailable("product1", 2)).thenReturn(true);
        when(paymentService.processPayment(any())).thenReturn(successfulPayment);
        
        // When
        Order result = orderService.processOrder(request);
        
        // Then
        assertThat(result).isNotNull();
        assertThat(result.getTransactionId()).isEqualTo("tx123");
        
        verify(inventoryService).reserve("product1", 2);
        verify(notificationService).sendOrderConfirmation(result);
    }
    
    @Test
    void shouldThrowExceptionWhenProductNotAvailable() {
        // Given
        OrderRequest request = createOrderRequest();
        when(inventoryService.isAvailable("product1", 2)).thenReturn(false);
        
        // When & Then
        assertThatThrownBy(() -> orderService.processOrder(request))
            .isInstanceOf(OutOfStockException.class)
            .hasMessage("Product not available");
        
        verify(paymentService, never()).processPayment(any());
        verify(inventoryService, never()).reserve(anyString(), anyInt());
    }
}
```

### 🏛️ Архитектурные тесты с ArchUnit

```java
// Тесты архитектурных правил
@AnalyzeClasses(packages = "com.example.myapp")
public class ArchitectureTest {
    
    @ArchTest
    static final ArchRule services_should_not_depend_on_controllers =
        noClasses()
            .that().resideInAPackage("..service..")
            .should().dependOnClassesThat()
            .resideInAPackage("..controller..");
    
    @ArchTest
    static final ArchRule repositories_should_only_be_accessed_by_services =
        classes()
            .that().resideInAPackage("..repository..")
            .should().onlyBeAccessed().byClassesThat()
            .resideInAPackage("..service..");
    
    @ArchTest
    static final ArchRule services_should_be_annotated_with_service =
        classes()
            .that().resideInAPackage("..service..")
            .and().areNotInterfaces()
            .should().beAnnotatedWith(Service.class);
    
    @ArchTest
    static final ArchRule no_cycles_in_packages =
        slices()
            .matching("com.example.myapp.(*)..")
            .should().beFreeOfCycles();
}
```

### 📊 Integration тесты

```java
// Integration тест с TestContainers
@SpringBootTest
@Testcontainers
class OrderServiceIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:13")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");
    
    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:6-alpine")
            .withExposedPorts(6379);
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private OrderRepository orderRepository;
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
        
        registry.add("spring.redis.host", redis::getHost);
        registry.add("spring.redis.port", redis::getFirstMappedPort);
    }
    
    @Test
    void shouldPersistOrderInDatabase() {
        // Given
        OrderRequest request = createOrderRequest();
        
        // When
        Order order = orderService.processOrder(request);
        
        // Then
        Optional<Order> savedOrder = orderRepository.findById(order.getId());
        assertThat(savedOrder).isPresent();
        assertThat(savedOrder.get().getStatus()).isEqualTo(OrderStatus.CONFIRMED);
    }
}
```

### 🎯 Практическое задание 4

**Задача:** Создание комплексной тестовой стратегии

Для следующего сервиса создайте полный набор тестов:

```java
@Service
public class ProductRecommendationService {
    private final ProductRepository productRepository;
    private final UserBehaviorService userBehaviorService;
    private final MLRecommendationEngine mlEngine;
    private final CacheService cacheService;
    
    public List<Product> getRecommendations(String userId, int limit) {
        // Проверяем кэш
        String cacheKey = "recommendations:" + userId;
        List<Product> cached = cacheService.get(cacheKey, List.class);
        if (cached != null) {
            return cached.subList(0, Math.min(limit, cached.size()));
        }
        
        // Получаем поведение пользователя
        UserBehavior behavior = userBehaviorService.getUserBehavior(userId);
        
        // Получаем рекомендации от ML движка
        List<String> productIds = mlEngine.recommend(behavior, limit * 2);
        
        // Загружаем продукты
        List<Product> products = productRepository.findByIdsAndAvailable(productIds);
        
        // Фильтруем и сортируем
        List<Product> recommendations = products.stream()
            .filter(Product::isActive)
            .sorted(Comparator.comparing(Product::getPopularityScore).reversed())
            .limit(limit)
            .collect(Collectors.toList());
        
        // Кэшируем результат
        cacheService.put(cacheKey, recommendations, Duration.ofMinutes(30));
        
        return recommendations;
    }
}
```

**Требуется создать:**
1. Unit тесты с моками
2. Integration тесты
3. Архитектурные тесты
4. Performance тесты

---

## 5. Рефакторинг legacy кода

### 🏚️ Проблемы legacy кода

```
┌─────────────────────────────────────────────────────────────┐
│                   LEGACY CODE PROBLEMS                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🔴 ТЕХНИЧЕСКИЕ ПРОБЛЕМЫ:                                   │
│     • Отсутствие тестов                                     │
│     • Сложная архитектура                                   │
│     • Тесная связанность                                    │
│     • Дублирование кода                                     │
│     • Устаревшие технологии                                 │
│                                                             │
│  🔴 БИЗНЕС-ПРОБЛЕМЫ:                                        │
│     • Медленная разработка                                  │
│     • Высокая стоимость изменений                           │
│     • Частые баги                                           │
│     • Сложность добавления функций                          │
│     • Риски при изменениях                                  │
│                                                             │
│  🔴 ЧЕЛОВЕЧЕСКИЕ ПРОБЛЕМЫ:                                  │
│     • Страх вносить изменения                               │
│     • Потеря знаний о системе                               │
│     • Демотивация команды                                   │
│     • Сложность онбординга                                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 🛠️ Стратегии рефакторинга

```
┌─────────────────────────────────────────────────────────────┐
│                REFACTORING STRATEGIES                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. STRANGLER FIG PATTERN                                   │
│     ┌─────────────────────────────────────────────────┐     │
│     │    Legacy System                                │     │
│     │  ┌─────────────────┐                            │     │
│     │  │   Module A      │ ← Gradually replace        │     │
│     │  └─────────────────┘                            │     │
│     │  ┌─────────────────┐                            │     │
│     │  │   Module B      │                            │     │
│     │  └─────────────────┘                            │     │
│     │                     ┌─────────────────┐         │     │
│     │                     │   New Module C  │         │     │
│     │                     └─────────────────┘         │     │
│     └─────────────────────────────────────────────────┘     │
│                                                             │
│  2. ANTI-CORRUPTION LAYER                                   │
│     ┌─────────────────────────────────────────────────┐     │
│     │  New System     │  ACL  │    Legacy System      │     │
│     │                 │       │                       │     │
│     │  Clean API  ←──→│ Adapt │←──→ Complex Legacy    │     │
│     │                 │       │     Interface         │     │
│     └─────────────────────────────────────────────────┘     │
│                                                             │
│  3. BRANCH BY ABSTRACTION                                   │
│     ┌─────────────────────────────────────────────────┐     │
│     │              Abstraction Layer                  │     │
│     │                      │                          │     │
│     │    ┌─────────────────┼─────────────────┐        │     │
│     │    │                 │                 │        │     │
│     │    ▼                 ▼                 ▼        │     │
│     │ Old Impl A      New Impl B        New Impl C    │     │
│     └─────────────────────────────────────────────────┘     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 🔧 Пример рефакторинга: от God Class к модульной структуре

```java
// ❌ ПЛОХО: God Class
public class OrderManager {
    private Connection dbConnection;
    private EmailSender emailSender;
    private PaymentGateway paymentGateway;
    private InventoryService inventoryService;
    
    public void processOrder(Order order) {
        // Валидация (смешана с бизнес-логикой)
        if (order.getItems().isEmpty()) {
            throw new RuntimeException("Order is empty");
        }
        
        // Проверка инвентаря (прямой SQL)
        try {
            Statement stmt = dbConnection.createStatement();
            for (OrderItem item : order.getItems()) {
                ResultSet rs = stmt.executeQuery(
                    "SELECT quantity FROM inventory WHERE product_id = " + item.getProductId()
                );
                if (rs.next() && rs.getInt("quantity") < item.getQuantity()) {
                    throw new RuntimeException("Not enough inventory");
                }
            }
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
        
        // Обработка платежа (смешана с другой логикой)
        PaymentRequest paymentRequest = new PaymentRequest();
        paymentRequest.setAmount(order.getTotalAmount());
        paymentRequest.setCardNumber(order.getPaymentInfo().getCardNumber());
        PaymentResult result = paymentGateway.processPayment(paymentRequest);
        
        if (!result.isSuccessful()) {
            throw new RuntimeException("Payment failed");
        }
        
        // Обновление базы данных (прямой SQL)
        try {
            PreparedStatement stmt = dbConnection.prepareStatement(
                "INSERT INTO orders (customer_id, total_amount, status) VALUES (?, ?, ?)"
            );
            stmt.setLong(1, order.getCustomerId());
            stmt.setBigDecimal(2, order.getTotalAmount());
            stmt.setString(3, "CONFIRMED");
            stmt.executeUpdate();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
        
        // Отправка email (прямо в основной логике)
        String emailBody = "Your order #" + order.getId() + " has been confirmed.";
        emailSender.sendEmail(order.getCustomerEmail(), "Order Confirmation", emailBody);
    }
}
```

```java
// ✅ ХОРОШО: Рефакторинг в модульную структуру

// 1. Валидация вынесена в отдельный класс
@Component
public class OrderValidator {
    public void validate(Order order) {
        if (order.getItems().isEmpty()) {
            throw new InvalidOrderException("Order must contain at least one item");
        }
        
        if (order.getCustomerId() == null) {
            throw new InvalidOrderException("Customer ID is required");
        }
        
        // другие валидации...
    }
}

// 2. Работа с инвентарем через репозиторий
@Repository
public class InventoryRepository {
    private final JdbcTemplate jdbcTemplate;
    
    public InventoryRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    
    public boolean isAvailable(String productId, int quantity) {
        String sql = "SELECT quantity FROM inventory WHERE product_id = ?";
        Integer available = jdbcTemplate.queryForObject(sql, Integer.class, productId);
        return available != null && available >= quantity;
    }
    
    public void reserve(String productId, int quantity) {
        String sql = "UPDATE inventory SET quantity = quantity - ? WHERE product_id = ?";
        jdbcTemplate.update(sql, quantity, productId);
    }
}

// 3. Сервис для работы с платежами
@Service
public class PaymentService {
    private final PaymentGateway paymentGateway;
    
    public PaymentService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }
    
    public PaymentResult processPayment(PaymentInfo paymentInfo, BigDecimal amount) {
        PaymentRequest request = PaymentRequest.builder()
            .amount(amount)
            .cardNumber(paymentInfo.getCardNumber())
            .expiryDate(paymentInfo.getExpiryDate())
            .cvv(paymentInfo.getCvv())
            .build();
            
        return paymentGateway.processPayment(request);
    }
}

// 4. Репозиторий для заказов
@Repository
public class OrderRepository {
    private final JdbcTemplate jdbcTemplate;
    
    public OrderRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    
    public Order save(Order order) {
        String sql = "INSERT INTO orders (customer_id, total_amount, status) VALUES (?, ?, ?) RETURNING id";
        Long id = jdbcTemplate.queryForObject(sql, Long.class,
            order.getCustomerId(), order.getTotalAmount(), order.getStatus().toString());
        order.setId(id);
        return order;
    }
}

// 5. Сервис уведомлений
@Service
public class NotificationService {
    private final EmailSender emailSender;
    
    public NotificationService(EmailSender emailSender) {
        this.emailSender = emailSender;
    }
    
    public void sendOrderConfirmation(Order order) {
        String subject = "Order Confirmation #" + order.getId();
        String body = buildOrderConfirmationEmail(order);
        emailSender.sendEmail(order.getCustomerEmail(), subject, body);
    }
    
    private String buildOrderConfirmationEmail(Order order) {
        // построение тела письма...
        return "Your order #" + order.getId() + " has been confirmed.";
    }
}

// 6. Главный сервис - координирует работу
@Service
@Transactional
public class OrderService {
    private final OrderValidator orderValidator;
    private final InventoryRepository inventoryRepository;
    private final PaymentService paymentService;
    private final OrderRepository orderRepository;
    private final NotificationService notificationService;
    
    public OrderService(OrderValidator orderValidator,
                       InventoryRepository inventoryRepository,
                       PaymentService paymentService,
                       OrderRepository orderRepository,
                       NotificationService notificationService) {
        this.orderValidator = orderValidator;
        this.inventoryRepository = inventoryRepository;
        this.paymentService = paymentService;
        this.orderRepository = orderRepository;
        this.notificationService = notificationService;
    }
    
    public Order processOrder(Order order) {
        // 1. Валидация
        orderValidator.validate(order);
        
        // 2. Проверка доступности товаров
        validateInventoryAvailability(order);
        
        // 3. Обработка платежа
        PaymentResult paymentResult = paymentService.processPayment(
            order.getPaymentInfo(), order.getTotalAmount()
        );
        
        if (!paymentResult.isSuccessful()) {
            throw new PaymentException("Payment failed: " + paymentResult.getErrorMessage());
        }
        
        // 4. Резервирование товаров
        reserveInventory(order);
        
        // 5. Сохранение заказа
        order.setStatus(OrderStatus.CONFIRMED);
        order.setPaymentTransactionId(paymentResult.getTransactionId());
        Order savedOrder = orderRepository.save(order);
        
        // 6. Отправка уведомления
        notificationService.sendOrderConfirmation(savedOrder);
        
        return savedOrder;
    }
    
    private void validateInventoryAvailability(Order order) {
        for (OrderItem item : order.getItems()) {
            if (!inventoryRepository.isAvailable(item.getProductId(), item.getQuantity())) {
                throw new InsufficientInventoryException(
                    "Not enough inventory for product: " + item.getProductId()
                );
            }
        }
    }
    
    private void reserveInventory(Order order) {
        for (OrderItem item : order.getItems()) {
            inventoryRepository.reserve(item.getProductId(), item.getQuantity());
        }
    }
}
```

### 📋 Пошаговый план рефакторинга

```
┌─────────────────────────────────────────────────────────────┐
│                REFACTORING PLAN                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ФАЗА 1: ПОДГОТОВКА (1-2 недели)                            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ ✅ Создать comprehensive тесты для существующего    │    │
│  │    функционала                                      │    │
│  │ ✅ Настроить мониторинг и метрики                   │    │
│  │ ✅ Проанализировать зависимости                     │    │
│  │ ✅ Создать план миграции                            │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  ФАЗА 2: ИЗОЛЯЦИЯ (2-3 недели)                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ ✅ Выделить интерфейсы для ключевых компонентов     │    │
│  │ ✅ Создать Anti-Corruption Layer                    │    │
│  │ ✅ Добавить абстракции для внешних зависимостей     │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  ФАЗА 3: ДЕКОМПОЗИЦИЯ (4-6 недель)                          │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ ✅ Выделить отдельные сервисы                       │    │
│  │ ✅ Переместить бизнес-логику из контроллеров        │    │
│  │ ✅ Разделить данные и поведение                     │    │
│  │ ✅ Применить DI для управления зависимостями        │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  ФАЗА 4: ОПТИМИЗАЦИЯ (2-3 недели)                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ ✅ Удалить дублирование кода                        │    │
│  │ ✅ Оптимизировать производительность                │    │
│  │ ✅ Добавить кэширование где необходимо              │    │
│  │ ✅ Улучшить обработку ошибок                        │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 🎯 Практическое задание 5

**Задача:** Рефакторинг legacy системы

Дан legacy код системы управления библиотекой. Проведите его рефакторинг:

```java
// Legacy код (требует рефакторинга)
public class LibraryManager {
    private Connection connection;
    
    public void borrowBook(int userId, int bookId) throws Exception {
        // Проверка пользователя
        PreparedStatement userStmt = connection.prepareStatement(
            "SELECT * FROM users WHERE id = ? AND status = 'ACTIVE'"
        );
        userStmt.setInt(1, userId);
        ResultSet userRs = userStmt.executeQuery();
        if (!userRs.next()) {
            throw new Exception("User not found or inactive");
        }
        
        // Проверка книги
        PreparedStatement bookStmt = connection.prepareStatement(
            "SELECT * FROM books WHERE id = ? AND available = true"
        );
        bookStmt.setInt(1, bookId);
        ResultSet bookRs = bookStmt.executeQuery();
        if (!bookRs.next()) {
            throw new Exception("Book not available");
        }
        
        // Проверка лимита
        PreparedStatement countStmt = connection.prepareStatement(
            "SELECT COUNT(*) FROM borrowings WHERE user_id = ? AND returned_at IS NULL"
        );
        countStmt.setInt(1, userId);
        ResultSet countRs = countStmt.executeQuery();
        countRs.next();
        if (countRs.getInt(1) >= 5) {
            throw new Exception("User has reached borrowing limit");
        }
        
        // Создание записи о выдаче
        PreparedStatement borrowStmt = connection.prepareStatement(
            "INSERT INTO borrowings (user_id, book_id, borrowed_at, due_date) VALUES (?, ?, NOW(), DATE_ADD(NOW(), INTERVAL 14 DAY))"
        );
        borrowStmt.setInt(1, userId);
        borrowStmt.setInt(2, bookId);
        borrowStmt.executeUpdate();
        
        // Обновление статуса книги
        PreparedStatement updateBookStmt = connection.prepareStatement(
            "UPDATE books SET available = false WHERE id = ?"
        );
        updateBookStmt.setInt(1, bookId);
        updateBookStmt.executeUpdate();
        
        // Отправка email (захардкожено)
        String userEmail = userRs.getString("email");
        String bookTitle = bookRs.getString("title");
        System.out.println("Email sent to " + userEmail + " about borrowing " + bookTitle);
    }
}
```

**Требования к рефакторингу:**
1. Разделить на отдельные сервисы и репозитории
2. Применить Dependency Injection
3. Добавить валидацию
4. Создать доменные модели
5. Написать unit тесты
6. Добавить обработку ошибок

---

## 📝 Итоговое задание блока

### 🎯 Финальный проект: Система управления задачами

Создайте систему управления задачами (Task Management System) с применением всех изученных принципов:

**Функциональные требования:**
- Создание, редактирование, удаление задач
- Назначение задач пользователям
- Отслеживание статуса выполнения
- Уведомления о дедлайнах
- Комментарии к задачам
- Отчеты по проектам

**Архитектурные требования:**
1. **Модульность:** Разделить на модули (User, Task, Project, Notification)
2. **DI:** Использовать внедрение зависимостей
3. **Конфигурация:** Поддержка разных окружений
4. **Тестирование:** Unit, Integration, архитектурные тесты
5. **Качество кода:** Следовать принципам чистого кода

**Техническое задание:**
```java
// Основные интерфейсы для реализации

public interface TaskService {
    Task createTask(CreateTaskRequest request);
    Task updateTask(Long taskId, UpdateTaskRequest request);
    void deleteTask(Long taskId);
    Task assignTask(Long taskId, Long userId);
    List<Task> getTasksByProject(Long projectId);
    List<Task> getTasksByUser(Long userId);
}

public interface NotificationService {
    void sendTaskAssignmentNotification(Task task, User user);
    void sendDeadlineReminder(Task task);
    void sendTaskCompletedNotification(Task task);
}

public interface ReportService {
    ProjectReport generateProjectReport(Long projectId, DateRange dateRange);
    UserProductivityReport generateUserReport(Long userId, DateRange dateRange);
}
```

**Критерии оценки:**
- ✅ Правильное применение DI (20%)
- ✅ Модульная архитектура (20%)
- ✅ Качество тестов (20%)
- ✅ Управление конфигурацией (15%)
- ✅ Читаемость и структура кода (15%)
- ✅ Документация и диаграммы (10%)

---

## 📚 Дополнительные материалы

### 📖 Рекомендуемая литература
- "Clean Code" - Robert Martin
- "Dependency Injection in .NET" - Mark Seemann
- "Working Effectively with Legacy Code" - Michael Feathers
- "Building Microservices" - Sam Newman

### 🔗 Полезные ссылки
- [Spring Framework Dependency Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans)
- [Google Guice Documentation](https://github.com/google/guice/wiki)
- [ArchUnit Documentation](https://www.archunit.org/userguide/html/000_Index.html)
- [Testcontainers](https://www.testcontainers.org/)

### 🛠️ Инструменты
- **DI Контейнеры:** Spring, Guice, Dagger, CDI
- **Тестирование:** JUnit 5, Mockito, TestContainers, ArchUnit
- **Конфигурация:** Spring Boot Config, Typesafe Config
- **Мониторинг:** Micrometer, Prometheus, Grafana

---

## ✅ Чек-лист завершения блока

### Теоретические знания:
- [ ] Понимаю принципы IoC и DI
- [ ] Знаю типы внедрения зависимостей
- [ ] Понимаю разницу между coupling и cohesion
- [ ] Умею проектировать модульную архитектуру
- [ ] Знаю стратегии управления конфигурацией
- [ ] Понимаю пирамиду тестирования
- [ ] Знаю паттерны рефакторинга legacy кода

### Практические навыки:
- [ ] Умею настраивать DI контейнер
- [ ] Могу создавать модульную структуру проекта
- [ ] Пишу unit тесты с моками
- [ ] Умею рефакторить тесно связанный код
- [ ] Создаю архитектурные тесты
- [ ] Управляю конфигурацией приложения
- [ ] Применяю стратегии работы с legacy кодом

### Проектные компетенции:
- [ ] Могу спроектировать слабосвязанную систему
- [ ] Умею выбрать подходящую стратегию рефакторинга
- [ ] Создаю тестируемую архитектуру
- [ ] Документирую архитектурные решения
- [ ] Планирую поэтапную модернизацию системы

---

## 🎓 Следующие шаги

После завершения этого блока переходите к:
- **Блок 6:** Производительность и масштабируемость
- **Блок 7:** Практика и экспертиза

Успехов в изучении архитектуры кода! 🚀

---

*Этот материал является частью курса "Архитектура кода от новичка до эксперта". Полная программа включает 7 блоков и рассчитана на 8-12 месяцев изучения.*