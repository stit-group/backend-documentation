# Блок 6: Тестирование и качество микросервисов

**⏱️ Продолжительность:** 3-4 недели  
**🎯 Цель:** Освоить стратегии тестирования распределенных систем  
**📈 Уровень сложности:** Продвинутый  

---

## 📋 Содержание блока

```
┌─────────────────────────────────────────────────────────────┐
│                 ТЕСТИРОВАНИЕ МИКРОСЕРВИСОВ                  │
├─────────────────────────────────────────────────────────────┤
│ 1. Пирамида тестирования для микросервисов                  │
│ 2. Contract testing                                         │
│ 3. Integration и E2E тестирование                          │
│ 4. Chaos engineering                                        │
│ 5. Performance testing                                      │
└─────────────────────────────────────────────────────────────┘
```

---

## 1. Пирамида тестирования для микросервисов

### 🏗️ Архитектура тестирования

```
                    ╔════════════════════╗
                    ║    E2E TESTS       ║ ← Медленно, дорого
                    ║      (GUI)         ║   Мало тестов
                    ╚════════════════════╝
                 ╔═══════════════════════════╗
                 ║   INTEGRATION TESTS       ║ ← Среднее время
                 ║   (API, Database)         ║   Умеренно
                 ╚═══════════════════════════╝
              ╔═════════════════════════════════╗
              ║        UNIT TESTS               ║ ← Быстро, дешево
              ║   (Business Logic)              ║   Много тестов
              ╚═════════════════════════════════╝
```

### 📊 Распределение тестов

| Тип теста | Процент | Время выполнения | Стоимость поддержки |
|-----------|---------|------------------|-------------------|
| Unit Tests | 70% | < 1 сек | Низкая |
| Integration Tests | 20% | 1-10 сек | Средняя |
| E2E Tests | 10% | 10+ сек | Высокая |

---

### 🔬 Unit Tests - Фундамент качества

**Цель:** Тестирование бизнес-логики в изоляции

```java
// ✅ Хороший unit test
@Test
void shouldCalculateDiscountForPremiumCustomer() {
    // Given
    Customer customer = new Customer("premium");
    Order order = new Order(BigDecimal.valueOf(100));
    
    // When
    BigDecimal discount = discountService.calculateDiscount(customer, order);
    
    // Then
    assertThat(discount).isEqualTo(BigDecimal.valueOf(10));
}
```

**Принципы написания:**
- **F**ast - быстрые
- **I**ndependent - независимые
- **R**epeatable - повторяемые
- **S**elf-validating - самопроверяющиеся
- **T**imely - своевременные

---

### 🔗 Integration Tests - Проверка взаимодействия

**Цель:** Тестирование интеграции с внешними системами

```java
@SpringBootTest
@Testcontainers
class UserServiceIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:13")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");
    
    @Test
    void shouldSaveAndRetrieveUser() {
        // Given
        User user = new User("john@example.com");
        
        // When
        User saved = userService.save(user);
        User retrieved = userService.findById(saved.getId());
        
        // Then
        assertThat(retrieved.getEmail()).isEqualTo("john@example.com");
    }
}
```

---

### 🌐 E2E Tests - Полный пользовательский путь

**Цель:** Тестирование критических пользовательских сценариев

```javascript
// Cypress E2E test
describe('Order Flow', () => {
  it('should complete order successfully', () => {
    cy.visit('/products')
    cy.get('[data-cy=add-to-cart]').first().click()
    cy.get('[data-cy=cart-icon]').click()
    cy.get('[data-cy=checkout]').click()
    cy.get('[data-cy=payment-form]').within(() => {
      cy.get('[name=cardNumber]').type('4111111111111111')
      cy.get('[name=cvv]').type('123')
    })
    cy.get('[data-cy=submit-order]').click()
    cy.get('[data-cy=order-success]').should('be.visible')
  })
})
```

---

## 2. Contract Testing

### 🤝 Consumer-Driven Contracts

**Проблема:** Как убедиться, что изменения в API не сломают потребителей?

```
┌─────────────────┐    HTTP/API    ┌─────────────────┐
│   Consumer      │ ◄─────────────► │   Provider      │
│   Service       │     Contract   │   Service       │
└─────────────────┘                └─────────────────┘
```

**Решение:** Contract Testing с Pact

---

### 📝 Pact Framework - Практика

**1. Consumer создает контракт:**

```java
@ExtendWith(PactConsumerTestExt.class)
class UserServiceConsumerTest {
    
    @Pact(consumer = "user-service", provider = "profile-service")
    public RequestResponsePact createPact(PactDslWithProvider builder) {
        return builder
            .given("user exists")
            .uponReceiving("get user profile")
            .path("/users/123")
            .method("GET")
            .willRespondWith()
            .status(200)
            .body(new PactDslJsonBody()
                .stringType("name", "John Doe")
                .stringType("email", "john@example.com"))
            .toPact();
    }
}
```

**2. Provider проверяет контракт:**

```java
@Provider("profile-service")
@PactBroker(host = "localhost", port = "9292")
class ProfileServiceProviderTest {
    
    @State("user exists")
    public void userExists() {
        // Подготовка данных для теста
        userRepository.save(new User("123", "John Doe", "john@example.com"));
    }
    
    @TestTemplate
    @ExtendWith(PactVerificationInvocationContextProvider.class)
    void pactVerificationTestTemplate(PactVerificationContext context) {
        context.verifyInteraction();
    }
}
```

---

### 🔄 Workflow Contract Testing

```
┌─────────────────────────────────────────────────────────────┐
│                 CONTRACT TESTING WORKFLOW                   │
├─────────────────────────────────────────────────────────────┤
│                                                            │
│  1. Consumer пишет тест и генерирует контракт              │
│            ↓                                               │
│  2. Контракт публикуется в Pact Broker                    │
│            ↓                                               │
│  3. Provider скачивает и проверяет контракт                │
│            ↓                                               │
│  4. Результаты публикуются обратно                         │
│            ↓                                               │
│  5. Can-I-Deploy проверка перед деплоем                    │
│                                                            │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Integration и E2E тестирование

### 🐳 Test Containers - Реальная инфраструктура

**Преимущества:**
- Реальные базы данных вместо H2
- Консистентная среда для тестирования
- Изоляция тестов

```java
@SpringBootTest
@Testcontainers
class OrderServiceIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:13");
    
    @Container
    static KafkaContainer kafka = new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:latest"));
    
    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:6-alpine")
            .withExposedPorts(6379);
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.kafka.bootstrap-servers", kafka::getBootstrapServers);
        registry.add("spring.redis.host", redis::getHost);
    }
}
```

---

### 🌍 Управление тестовыми окружениями

```
┌─────────────────────────────────────────────────────────────┐
│                 TEST ENVIRONMENTS                           │
├─────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   Local     │  │   Staging   │  │ Production  │        │
│  │   Tests     │  │   Tests     │  │   Tests     │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
│         │                 │                 │             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │Test         │  │Integration  │  │Smoke        │        │
│  │Containers   │  │Environment  │  │Tests        │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                            │
└─────────────────────────────────────────────────────────────┘
```

---

### 🔄 Стратегия изоляции данных

**Проблема:** Тесты влияют друг на друга через общие данные

**Решения:**

1. **Database per test:**
```java
@Sql("/cleanup.sql")
@Sql("/test-data.sql")
@Test
void shouldProcessOrder() {
    // Test logic
}
```

2. **Transactional rollback:**
```java
@Transactional
@Rollback
@Test
void shouldSaveUser() {
    // Test logic - будет откачен
}
```

3. **Test data builders:**
```java
public class UserTestDataBuilder {
    private String email = "default@example.com";
    private String name = "Default User";
    
    public UserTestDataBuilder withEmail(String email) {
        this.email = email;
        return this;
    }
    
    public User build() {
        return new User(email, name);
    }
}
```

---

## 4. Chaos Engineering

### 🌪️ Принципы Chaos Engineering

**Цель:** Проверить устойчивость системы к неожиданным сбоям

```
┌─────────────────────────────────────────────────────────────┐
│                    CHAOS ENGINEERING                        │
├─────────────────────────────────────────────────────────────┤
│                                                            │
│  1. Гипотеза: Система выдержит отказ сервиса X             │
│             ↓                                               │
│  2. Эксперимент: Отключить сервис X                        │
│             ↓                                               │
│  3. Наблюдение: Мониторить поведение системы               │
│             ↓                                               │
│  4. Анализ: Подтвердилась ли гипотеза?                     │
│             ↓                                               │
│  5. Улучшение: Исправить найденные проблемы                │
│                                                            │
└─────────────────────────────────────────────────────────────┘
```

---

### 🐵 Chaos Monkey и Netflix Tools

**Chaos Monkey:** Случайно отключает инстансы в production

```yaml
# Chaos Monkey configuration
chaos:
  monkey:
    enabled: true
    watcher:
      enabled: true
    assaults:
      level: 1
      latencyActive: true
      latencyRangeStart: 1000
      latencyRangeEnd: 5000
      exceptionsActive: true
      killApplicationActive: true
```

**Типы атак:**
- **Latency Assault** - добавляет задержки
- **Exception Assault** - бросает исключения
- **Kill Application** - убивает процесс
- **Memory Assault** - создает нагрузку на память

---

### 🔧 Gremlin Platform - Продвинутый Chaos Engineering

**Возможности:**
- **Network attacks:** Latency, packet loss, DNS failures
- **Resource attacks:** CPU, memory, disk exhaustion
- **State attacks:** Process killer, shutdown

```bash
# Пример Gremlin атаки
gremlin attack-container \
  --name "cpu-attack" \
  --container-id "abc123" \
  --attack-type "cpu" \
  --cpu-capacity 80 \
  --length 300
```

---

### 📊 Эксперименты и метрики

**Ключевые метрики для мониторинга:**

| Метрика | Норма | Критичный уровень |
|---------|-------|-------------------|
| Response Time | < 200ms | > 1000ms |
| Error Rate | < 0.1% | > 1% |
| Throughput | 1000 RPS | < 500 RPS |
| Availability | 99.9% | < 99% |

```
┌─────────────────────────────────────────────────────────────┐
│                   CHAOS EXPERIMENT                          │
├─────────────────────────────────────────────────────────────┤
│                                                            │
│  Before: ████████████████████████████████████████ 100%    │
│  During: ████████████████████████████████████████  90%    │
│  After:  ████████████████████████████████████████  99%    │
│                                                            │
│  ✅ System recovered within SLA                            │
│  ✅ No data loss occurred                                  │
│  ⚠️  Response time increased by 20%                        │
│                                                            │
└─────────────────────────────────────────────────────────────┘
```

---

## 5. Performance Testing

### 📈 Типы нагрузочного тестирования

```
┌─────────────────────────────────────────────────────────────┐
│                 PERFORMANCE TESTING TYPES                   │
├─────────────────────────────────────────────────────────────┤
│                                                            │
│  Load Testing     │██████████████████████████████│        │
│  (Expected load)  │         Normal Usage         │        │
│                                                            │
│  Stress Testing   │████████████████████████████████████│  │
│  (Beyond capacity)│        Breaking Point           │    │
│                                                            │
│  Spike Testing    │██████████████████████████████████│    │
│  (Sudden increase)│    ▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲    │
│                                                            │
│  Volume Testing   │██████████████████████████████████│    │
│  (Large datasets) │        Big Data Processing       │    │
│                                                            │
└─────────────────────────────────────────────────────────────┘
```

---

### 🎯 JMeter - Настройка нагрузочных тестов

**Структура теста:**
```
Test Plan
├── Thread Group (Users)
│   ├── HTTP Request (API calls)
│   ├── CSV Data Set (Test data)
│   └── Response Assertions
├── Listeners (Results)
│   ├── Summary Report
│   └── Response Times Graph
└── Configuration
    ├── HTTP Header Manager
    └── User Defined Variables
```

**Пример JMeter скрипта:**
```xml
<TestPlan>
  <ThreadGroup>
    <elementProp name="ThreadGroup.main_controller" elementType="LoopController">
      <boolProp name="LoopController.continue_forever">false</boolProp>
      <stringProp name="LoopController.loops">100</stringProp>
    </elementProp>
    <stringProp name="ThreadGroup.num_threads">50</stringProp>
    <stringProp name="ThreadGroup.ramp_time">10</stringProp>
  </ThreadGroup>
</TestPlan>
```

---

### ⚡ Gatling - Высокопроизводительное тестирование

**Преимущества Gatling:**
- Высокая производительность (async I/O)
- Scala DSL для сложных сценариев
- Детальные HTML отчеты
- CI/CD интеграция

```scala
import io.gatling.core.Predef._
import io.gatling.http.Predef._

class UserJourneySimulation extends Simulation {
  
  val httpProtocol = http
    .baseUrl("https://api.example.com")
    .acceptHeader("application/json")
  
  val scn = scenario("User Journey")
    .exec(
      http("Login")
        .post("/auth/login")
        .body(StringBody("""{"email":"user@example.com","password":"secret"}"""))
        .check(jsonPath("$.token").saveAs("authToken"))
    )
    .pause(1)
    .exec(
      http("Get Profile")
        .get("/user/profile")
        .header("Authorization", "Bearer ${authToken}")
        .check(status.is(200))
    )
  
  setUp(
    scn.inject(rampUsers(100) during(30 seconds))
  ).protocols(httpProtocol)
}
```

---

### 🔍 Анализ узких мест

**Профилирование производительности:**

```
┌─────────────────────────────────────────────────────────────┐
│                    BOTTLENECK ANALYSIS                      │
├─────────────────────────────────────────────────────────────┤
│                                                            │
│  Database Queries        │████████████████████████████│60%│
│  External API calls      │████████████████████       │45%│
│  JSON Serialization      │████████████               │30%│
│  Business Logic          │████████                   │20%│
│  Network Latency         │██████                     │15%│
│  Memory Allocation       │████                       │10%│
│                                                            │
└─────────────────────────────────────────────────────────────┘
```

**Техники оптимизации:**

1. **Database optimization:**
   - Добавление индексов
   - Оптимизация запросов
   - Connection pooling

2. **Caching strategies:**
   - Redis для часто запрашиваемых данных
   - CDN для статических ресурсов
   - Application-level caching

3. **Asynchronous processing:**
   - Message queues для тяжелых операций
   - Event-driven architecture
   - Reactive programming

---

## 📋 Практические задания

### 🎯 Задание 1: Создание пирамиды тестов

**Цель:** Создать комплексную стратегию тестирования для микросервиса

**Шаги:**
1. Создать unit тесты для всех бизнес-методов
2. Настроить integration тесты с TestContainers
3. Написать несколько E2E тестов для критических путей
4. Настроить coverage reporting

**Критерии успеха:**
- Unit test coverage > 80%
- Integration тесты покрывают все внешние интеграции
- E2E тесты проверяют основные пользовательские сценарии

---

### 🎯 Задание 2: Contract Testing с Pact

**Цель:** Настроить contract testing между двумя сервисами

**Шаги:**
1. Создать consumer тест с Pact
2. Настроить provider verification
3. Настроить Pact Broker
4. Интегрировать в CI/CD pipeline

**Критерии успеха:**
- Контракты автоматически проверяются при изменениях
- Can-I-Deploy проверка работает корректно
- Все breaking changes отлавливаются

---

### 🎯 Задание 3: Chaos Engineering эксперимент

**Цель:** Проверить устойчивость системы к сбоям

**Шаги:**
1. Определить критические компоненты системы
2. Создать гипотезы об устойчивости
3. Провести контролируемые эксперименты
4. Проанализировать результаты и улучшить систему

**Критерии успеха:**
- Система корректно обрабатывает сбои
- Время восстановления в пределах SLA
- Пользователи не замечают деградации

---

### 🎯 Задание 4: Performance Testing Suite

**Цель:** Создать комплексный набор нагрузочных тестов

**Шаги:**
1. Определить performance requirements
2. Создать тесты в Gatling или JMeter
3. Настроить мониторинг во время тестов
4. Создать performance CI/CD pipeline

**Критерии успеха:**
- Все performance требования выполнены
- Bottlenecks идентифицированы и устранены
- Тесты интегрированы в CI/CD

---

## ✅ Итоговый результат блока

После завершения этого блока вы будете уметь:

**🎯 Стратегическое планирование тестирования:**
- Создавать комплексную стратегию тестирования микросервисов
- Правильно распределять типы тестов в пирамиде
- Балансировать скорость, надежность и стоимость тестов

**🤝 Contract Testing:**
- Настраивать consumer-driven contracts
- Предотвращать breaking changes между сервисами
- Интегрировать contract testing в CI/CD

**🧪 Комплексное тестирование:**
- Создавать надежные integration тесты
- Использовать TestContainers для реальной инфраструктуры
- Писать эффективные E2E тесты

**🌪️ Chaos Engineering:**
- Проводить контролируемые эксперименты
- Выявлять слабые места в архитектуре
- Повышать устойчивость системы

**📊 Performance Testing:**
- Создавать нагрузочные тесты
- Анализировать performance bottlenecks
- Оптимизировать производительность системы

**🔄 Automation & CI/CD:**
- Автоматизировать все типы тестов
- Интегрировать тестирование в deployment pipeline
- Обеспечивать быструю обратную связь для разработчиков

---

## 🚀 Следующие шаги

Переходите к **Блоку 7: Эксплуатация и оптимизация**, где изучите:
- Observability и мониторинг
- Troubleshooting распределенных систем
- Deployment strategies
- Organizational aspects

---

**📚 Полезные ресурсы:**
- [Test Pyramid - Martin Fowler](https://martinfowler.com/articles/practical-test-pyramid.html)
- [Pact Documentation](https://docs.pact.io/)
- [Chaos Engineering Principles](https://principlesofchaos.org/)
- [Performance Testing Guide](https://www.guru99.com/performance-testing.html)