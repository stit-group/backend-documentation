# План обучения System Design
*От основ до продвинутой архитектуры распределенных систем*

---

## 📚 Уровень 1: Фундаментальные концепции (2-3 недели)

### Неделя 1: Базовые принципы
- **Что такое System Design** - цели, задачи, scope
- **Вертикальное vs горизонтальное масштабирование** - когда применять
- **Load Balancing** - алгоритмы Round Robin, Weighted, Least Connections
- **Базовые метрики** - latency, throughput, availability
- **Single Point of Failure** - выявление и устранение

### Неделя 2: Хранение данных
- **SQL vs NoSQL** - критерии выбора
- **ACID свойства** - практическое значение
- **Database indexes** - типы, когда использовать
- **Нормализация vs денормализация** - trade-offs
- **Connection pooling** - зачем нужен, как настраивать

### Неделя 3: Сетевое взаимодействие
- **HTTP/HTTPS протоколы** - methods, status codes, headers
- **REST API принципы** - idempotency, stateless
- **Синхронное vs асинхронное взаимодействие** - когда что выбирать
- **Polling vs WebSockets** - real-time коммуникация
- **DNS и CDN** - как работают, зачем нужны

---

## 🏗️ Уровень 2: Архитектурные паттерны (3-4 недели)

### Неделя 4: Кэширование
- **Уровни кэширования** - browser, CDN, application, database
- **Cache patterns** - Cache-aside, Write-through, Write-behind
- **Cache invalidation** - TTL, manual, event-based
- **Distributed caching** - Redis, Memcached сравнение
- **Cache stampede** - проблема и решения

### Неделя 5: Микросервисы
- **Монолит vs микросервисы** - когда мигрировать
- **Domain-driven design** - bounded contexts
- **Service communication** - REST, gRPC, message queues
- **API Gateway** - функции, паттерны
- **Service discovery** - статический vs динамический

### Неделя 6: Обработка данных
- **Message queues** - FIFO, priorities, delivery guarantees
- **Event-driven architecture** - publisher/subscriber паттерн
- **Stream processing** - real-time vs batch
- **Data pipelines** - ETL процессы
- **Idempotency** - обеспечение в распределенных системах

### Неделя 7: Безопасность
- **Authentication vs Authorization** - разница и реализация
- **OAuth 2.0 и JWT** - flows, best practices
- **Rate limiting** - алгоритмы, distributed rate limiting
- **Input validation** - sanitization, injection attacks
- **HTTPS и TLS** - handshake, certificates

---

## ⚡ Уровень 3: Масштабируемость и надежность (4-5 недель)

### Неделя 8: Database scaling
- **Read replicas** - master-slave, читабельные реплики
- **Database partitioning** - horizontal vs vertical
- **Sharding strategies** - key-based, directory-based, range-based
- **Distributed transactions** - 2PC, Saga pattern
- **Database consistency models** - strong, eventual, weak

### Неделя 9: CAP theorem и консистентность
- **CAP theorem** - практические implications
- **ACID vs BASE** - когда выбирать каждый подход
- **Consistency patterns** - strong, eventual, weak consistency
- **Conflict resolution** - last-write-wins, vector clocks
- **Distributed consensus** - Raft, Byzantine fault tolerance

### Неделя 10: Отказоустойчивость
- **Circuit Breaker pattern** - states, timeout strategies
- **Retry mechanisms** - exponential backoff, jitter
- **Bulkhead isolation** - resource separation
- **Graceful degradation** - priority-based feature disabling
- **Health checks** - shallow vs deep checks

### Неделя 11: Мониторинг и observability
- **Logging strategies** - structured logging, log levels
- **Metrics collection** - push vs pull models
- **Distributed tracing** - correlation IDs, span creation
- **Alerting** - threshold-based, anomaly detection
- **SLA/SLO/SLI** - определение и измерение

### Неделя 12: Deployment и DevOps
- **Blue-green deployment** - zero-downtime strategies
- **Canary releases** - gradual rollout
- **Feature flags** - dynamic configuration
- **Container orchestration** - Docker, Kubernetes basics
- **Infrastructure as Code** - reproducible environments

---

## 🎯 Уровень 4: Продвинутые темы (4-6 недель)

### Неделя 13-14: Специализированные системы
- **Search systems** - indexing, ranking algorithms
- **Recommendation engines** - collaborative filtering, content-based
- **Real-time systems** - low-latency requirements
- **Geospatial systems** - location-based services
- **Time-series databases** - metrics, monitoring data

### Неделя 15-16: Производительность
- **Performance profiling** - CPU, memory, I/O bottlenecks
- **Database query optimization** - execution plans, index tuning
- **Network optimization** - connection pooling, keep-alive
- **Memory management** - garbage collection, memory leaks
- **Compression** - data compression strategies

### Неделя 17-18: Архитектурные решения
- **Event sourcing** - immutable event log
- **CQRS** - command query responsibility segregation
- **Serverless architecture** - functions as a service
- **Edge computing** - bringing computation closer to users
- **Multi-tenant architecture** - shared vs isolated resources

---

## 🛠️ Практические задания по неделям

### Базовый уровень
- **Неделя 2:** Спроектировать простой URL shortener
- **Неделя 3:** Добавить кэширование и rate limiting

### Средний уровень  
- **Неделя 6:** Спроектировать chat application
- **Неделя 7:** Добавить authentication и message persistence
- **Неделя 9:** Спроектировать distributed cache система

### Продвинутый уровень
- **Неделя 12:** Спроектировать social media feed
- **Неделя 14:** Спроектировать ride-sharing service
- **Неделя 16:** Спроектировать e-commerce platform
- **Неделя 18:** Спроектировать video streaming service

---

## 📖 Ресурсы для изучения

### Обязательные книги
- "Designing Data-Intensive Applications" - Martin Kleppmann
- "System Design Interview" - Alex Xu
- "Building Microservices" - Sam Newman

### Онлайн ресурсы
- High Scalability blog
- AWS Architecture Center
- Google Cloud Architecture Framework
- Microsoft Azure Well-Architected Framework

### Практика
- LeetCode System Design
- System Design Primer (GitHub)
- Case studies крупных компаний

---

## ✅ Чек-лист готовности

### После уровня 1
- [ ] Объясняю разницу между SQL и NoSQL
- [ ] Понимаю основы HTTP и REST API
- [ ] Знаю что такое load balancing
- [ ] Могу спроектировать simple web application

### После уровня 2
- [ ] Понимаю микросервисную архитектуру
- [ ] Знаю паттерны кэширования
- [ ] Разбираюсь в message queues
- [ ] Могу спроектировать API gateway

### После уровня 3
- [ ] Понимаю CAP theorem на практике
- [ ] Знаю стратегии database sharding
- [ ] Разбираюсь в failure handling patterns
- [ ] Могу спроектировать highly available system

### После уровня 4
- [ ] Знаю specialized system patterns
- [ ] Понимаю performance optimization
- [ ] Разбираюсь в advanced architectural patterns
- [ ] Готов к senior-level system design интервью

---

## 🎯 Финальная цель

После прохождения плана вы сможете:
- Спроектировать масштабируемую систему для миллионов пользователей
- Обосновать архитектурные решения с учетом trade-offs
- Решать проблемы производительности и надежности
- Уверенно проходить system design интервью

**Общее время обучения:** 18-20 недель при 10-15 часах в неделю