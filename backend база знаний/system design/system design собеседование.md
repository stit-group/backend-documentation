# Компактный гайд: System Design интервью для Backend разработчиков

*Структурированный подход к оценке архитектурных навыков за 60-90 минут*

---

## 🎯 Структура интервью (90 минут)

| Блок | Время | Фокус |
|------|-------|-------|
| **Warm-up** | 10 мин | Базовые концепции |
| **Core Design** | 40 мин | Практическое проектирование |
| **Deep Dive** | 30 мин | Специализированные темы |
| **Wrap-up** | 10 мин | Вопросы кандидата |

---

## 🔥 Core Questions (Основные вопросы)

### **1. Масштабируемость и архитектура** *(15 мин)*

**Базовый вопрос:** *Спроектируйте систему для обработки 1M RPS*

**🔍 Углубляющие вопросы:**
- Горизонтальное vs вертикальное масштабирование - когда что?
- Load balancing алгоритмы для вашего случая
- Bottlenecks и методы их предотвращения
- Auto-scaling стратегии

**💡 Что оценивать:**
- Понимание trade-offs
- Знание практических ограничений
- Системное мышление

---

### **2. Микросервисы и распределенные системы** *(15 мин)*

**Базовый вопрос:** *Как разбить монолит на микросервисы?*

**🔍 Углубляющие вопросы:**
- Database per service - плюсы/минусы
- Service communication: sync vs async
- Distributed transactions (Saga pattern)
- Service discovery и API Gateway роль

**💡 Что оценивать:**
- Понимание domain boundaries
- Знание паттернов микросервисов
- Опыт с distributed challenges

---

### **3. Данные и консистентность** *(10 мин)*

**Базовый вопрос:** *SQL vs NoSQL - как выбрать?*

**🔍 Углубляющие вопросы:**
- CAP theorem на практике
- Sharding стратегии
- ACID vs BASE
- Eventual consistency сценарии

**💡 Что оценивать:**
- Data modeling навыки
- Понимание consistency trade-offs
- Опыт с разными типами БД

---

## 🚀 Практические сценарии (40 минут)

*Выберите 1-2 сценария в зависимости от позиции*

### **Сценарий A: Social Feed** *(Instagram-like)*
**Требования:** 100M пользователей, real-time updates, media content

**Ключевые challenges:**
- Feed generation algorithm
- Image/video processing pipeline  
- Real-time notifications
- Content delivery optimization

**🔍 Deep dive topics:**
- Как handle viral content?
- Push vs Pull модель для feeds
- CDN strategy для media
- Caching layers design

---

### **Сценарий B: Real-time системы** *(Chat/Ride-sharing)*
**Требования:** Low latency, high concurrency, geo-data

**Ключевые challenges:**
- WebSocket connection management
- Message ordering и delivery guarantees
- Location-based services
- Real-time matching algorithms

**🔍 Deep dive topics:**
- Connection pooling strategies
- Geospatial indexing
- Event-driven architecture
- Conflict resolution

---

### **Сценарий C: E-commerce** *(Amazon-like)*
**Требования:** High availability, ACID transactions, search

**Ключевые challenges:**
- Inventory management
- Payment processing
- Search и recommendations
- Order processing pipeline

**🔍 Deep dive topics:**
- Distributed transactions
- Search relevance ranking
- Recommendation algorithms
- Fraud detection

---

## ⚡ Specialized Topics (30 минут)

*Выберите темы на основе опыта кандидата и требований позиции*

### **Отказоустойчивость** *(10 мин)*
- **Circuit Breaker pattern** - states и transitions
- **Retry strategies** - exponential backoff, jitter
- **Bulkhead isolation** - resource separation
- **Graceful degradation** - feature prioritization

### **Производительность** *(10 мин)*
- **Caching layers** - multi-level strategies
- **Database optimization** - indexing, query optimization
- **Connection pooling** - sizing и management
- **CDN integration** - cache invalidation strategies

### **Безопасность** *(10 мин)*
- **Authentication/Authorization** - OAuth2, JWT, RBAC
- **API security** - rate limiting, input validation
- **Data protection** - encryption at rest/transit
- **Secret management** - rotation strategies

---

## 🎯 Red Flags и Green Flags

### ❌ **Red Flags**
- Не учитывает trade-offs
- Overengineering простых задач
- Игнорирует non-functional requirements
- Нет понимания CAP theorem
- Не может объяснить выбор технологий

### ✅ **Green Flags**
- Задает clarifying questions
- Обсуждает multiple approaches
- Учитывает business constraints
- Знает когда не использовать сложные решения
- Понимает operational complexity

---

## 📊 Scoring Matrix

| Критерий | Weight | 1-2 (Weak) | 3-4 (Good) | 5 (Excellent) |
|----------|--------|------------|------------|---------------|
| **System Design** | 30% | Basic understanding | Solid design skills | Advanced architecture |
| **Scalability** | 25% | Single machine thinking | Understands scaling | Distributed systems expert |
| **Trade-offs** | 20% | No consideration | Some awareness | Deep analysis |
| **Communication** | 15% | Unclear explanation | Good communication | Excellent teaching |
| **Experience** | 10% | Theoretical only | Some practice | Production experience |

**Проходной балл:** 3.5+ для Senior позиций, 3.0+ для Middle

---

## 🛠️ Quick Reference

### **Обязательные концепции для проверки:**
- [ ] Load balancing algorithms
- [ ] Database partitioning/sharding
- [ ] Caching strategies (multi-level)
- [ ] CAP theorem implications
- [ ] Microservices communication patterns
- [ ] Failure handling patterns

### **Technology stack awareness:**
- [ ] Message queues (Kafka, RabbitMQ)
- [ ] Databases (SQL/NoSQL trade-offs)
- [ ] Caching (Redis, Memcached)
- [ ] Load balancers (nginx, HAProxy)
- [ ] Monitoring (metrics, logging, tracing)

---

## 💡 Interview Tips

### **Для интервьюера:**
1. **Start broad, then narrow** - сначала high-level design
2. **Push for specifics** - "Какой именно алгоритм?"
3. **Ask about failures** - "Что если этот компонент упадет?"
4. **Challenge assumptions** - "А если нагрузка вырастет в 10x?"
5. **Time management** - не давайте увязнуть в деталях

### **Структура ответа кандидата:**
1. **Clarifying questions** (5 мин)
2. **High-level architecture** (10 мин)  
3. **Component deep-dive** (15 мин)
4. **Scale & optimize** (10 мин)

### **Примеры follow-up вопросов:**
- "Как вы будете мониторить эту систему?"
- "Какие метрики важны для этого компонента?"
- "Как обеспечить zero-downtime deployment?"
- "Что происходит при network partition?"

---

## 🎪 Bonus: Quick Fire Round (5 минут)

*Быстрые вопросы для проверки breadth of knowledge:*

1. **Eventual consistency** - объясните за 30 секунд
2. **Database index** - когда создавать, когда избегать?
3. **HTTP/2 vs HTTP/1.1** - ключевые отличия?
4. **Docker vs VM** - use cases для каждого?
5. **Stateful vs Stateless** - плюсы/минусы?

---

*Этот гайд позволяет провести comprehensive оценку за 90 минут, покрывая все ключевые аспекты system design с возможностью адаптации под конкретную позицию и уровень кандидата.*