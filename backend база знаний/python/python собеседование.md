# Python Backend Interview Guide
## Компактное собеседование с практическим фокусом

---

## 🎯 Структура собеседования
**Время:** 60-90 минут | **Вопросов:** 12 основных + углубление | **Формат:** Теория + практика

### Уровни сложности:
- 🟢 **Junior (1-2 года)**: Вопросы 1-4, основы
- 🟡 **Middle (2-5 лет)**: Вопросы 1-8, углубление
- 🔴 **Senior (5+ лет)**: Все вопросы, архитектурные решения

---

## 1. Основы Python & Memory Management
**🟢 Базовый вопрос:**
Объясните разницу между `list`, `tuple` и `set`. Когда использовать каждый тип?

**🟡 Углубление:**
```python
# Что выведет этот код и почему?
a = [1, 2, 3]
b = a
c = a[:]
b.append(4)
c.append(5)
print(a, b, c)
```

**🔴 Эксперт:**
- Как работает сборщик мусора в Python (reference counting + cycle detection)?
- Что такое `__slots__` и когда его использовать?
- Объясните memory interning в Python

**💡 Практическое задание:**
Реализуйте LRU cache с ограниченной памятью без использования `functools.lru_cache`

---

## 2. Async/Await & Concurrency
**🟢 Базовый вопрос:**
В чем разница между синхронным и асинхронным кодом? Когда использовать `async/await`?

**🟡 Углубление:**
```python
import asyncio

async def fetch_data(delay):
    await asyncio.sleep(delay)
    return f"Data after {delay}s"

# Как запустить 3 корутины параллельно и получить результаты?
```

**🔴 Эксперт:**
- Объясните разницу между `asyncio.gather()`, `asyncio.wait()` и `asyncio.as_completed()`
- Что такое GIL и как обойти его ограничения?
- Как интегрировать синхронный код в асинхронное приложение?

**💡 Практическое задание:**
Напишите асинхронный web scraper, который обрабатывает 100 URL с ограничением в 10 одновременных соединений

---

## 3. Web Framework Architecture
**🟢 Базовый вопрос:**
Объясните архитектуру Django/FastAPI. Что такое middleware?

**🟡 Углубление:**
```python
# FastAPI: Как создать dependency injection для database session?
from fastapi import Depends

def get_db():
    # Ваша реализация
    pass

@app.get("/users/{user_id}")
def get_user(user_id: int, db = Depends(get_db)):
    # Использование db
    pass
```

**🔴 Эксперт:**
- Разница между WSGI и ASGI
- Как обрабатывать websockets в production
- Стратегии API versioning

**💡 Практическое задание:**
Спроектируйте REST API для блог-системы с аутентификацией, правами доступа и rate limiting

---

## 4. Database & ORM
**🟢 Базовый вопрос:**
Что такое ORM? Преимущества и недостатки по сравнению с raw SQL.

**🟡 Углубление:**
```python
# SQLAlchemy: Оптимизируйте этот код
users = session.query(User).all()
for user in users:
    print(user.posts)  # N+1 problem!
```

**🔴 Эксперт:**
- Database connection pooling
- Стратегии sharding и репликации
- ACID свойства и уровни изоляции

**💡 Практическое задание:**
Спроектируйте схему БД для e-commerce с оптимизацией под 1M+ пользователей

---

## 5. Testing & Code Quality
**🟢 Базовый вопрос:**
Разница между unit, integration и e2e тестами. Что такое мокинг?

**🟡 Углубление:**
```python
# Pytest: Как протестировать этот код?
class PaymentService:
    def __init__(self, bank_api):
        self.bank_api = bank_api
    
    def process_payment(self, amount, card):
        if amount <= 0:
            raise ValueError("Invalid amount")
        return self.bank_api.charge(amount, card)
```

**🔴 Эксперт:**
- Property-based testing с Hypothesis
- Contract testing для микросервисов
- Chaos engineering

**💡 Практическое задание:**
Напишите тесты для API endpoint с моками внешних сервисов

---

## 6. Performance & Optimization
**🟢 Базовый вопрос:**
Как профилировать Python приложение? Основные bottlenecks в web приложениях.

**🟡 Углубление:**
```python
# Какой код быстрее и почему?
# Вариант A
result = []
for i in range(1000000):
    if i % 2 == 0:
        result.append(i * 2)

# Вариант B
result = [i * 2 for i in range(1000000) if i % 2 == 0]

# Вариант C
result = list(map(lambda x: x * 2, filter(lambda x: x % 2 == 0, range(1000000))))
```

**🔴 Эксперт:**
- Caching strategies (Redis, Memcached)
- Database query optimization
- Load balancing и horizontal scaling

**💡 Практическое задание:**
Оптимизируйте медленный API endpoint, обрабатывающий большие данные

---

## 7. Security & Best Practices
**🟢 Базовый вопрос:**
Основные принципы безопасности web приложений (OWASP Top 10).

**🟡 Углубление:**
```python
# Найдите уязвимости в этом коде
@app.route('/user/<user_id>')
def get_user(user_id):
    query = f"SELECT * FROM users WHERE id = {user_id}"
    result = db.execute(query)
    return jsonify(result)
```

**🔴 Эксперт:**
- JWT vs Session authentication
- Rate limiting implementations
- Secure headers и CORS

**💡 Практическое задание:**
Реализуйте безопасную систему аутентификации с 2FA

---

## 8. Architecture & Patterns
**🟢 Базовый вопрос:**
Объясните принципы SOLID. Что такое dependency injection?

**🟡 Углубление:**
```python
# Рефакторинг: примените паттерны для улучшения кода
class OrderProcessor:
    def process(self, order):
        # validate order
        if not order.items:
            raise ValueError("Empty order")
        
        # calculate total
        total = sum(item.price * item.quantity for item in order.items)
        
        # apply discount
        if order.user.is_premium:
            total *= 0.9
        
        # send email
        send_email(order.user.email, f"Order total: {total}")
        
        # save to db
        db.save(order)
        return total
```

**🔴 Эксперт:**
- Microservices vs Monolith
- Event-driven architecture
- Domain-driven design (DDD)

**💡 Практическое задание:**
Спроектируйте архитектуру для высоконагруженного сервиса (1M+ RPS)

---

## 9. DevOps & Deployment
**🟢 Базовый вопрос:**
Что такое Docker? Основы CI/CD pipeline.

**🟡 Углубление:**
```dockerfile
# Оптимизируйте этот Dockerfile
FROM python:3.11
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

**🔴 Эксперт:**
- Kubernetes concepts (pods, services, deployments)
- Blue-green deployments
- Infrastructure as Code

**💡 Практическое задание:**
Настройте deployment pipeline с автотестами и rollback

---

## 10. Monitoring & Observability
**🟢 Базовый вопрос:**
Что логировать в production приложении? Метрики для мониторинга.

**🟡 Углубление:**
```python
# Добавьте proper logging и metrics
def process_payment(user_id, amount):
    try:
        user = get_user(user_id)
        result = payment_gateway.charge(user.card, amount)
        return result
    except Exception as e:
        # Как правильно обработать?
        pass
```

**🔴 Эксперт:**
- Distributed tracing
- SLI/SLO/SLA concepts
- Alerting strategies

---

## 11. Data Processing & APIs
**🟢 Базовый вопрос:**
REST API design principles. HTTP статус коды.

**🟡 Углубление:**
```python
# Как обработать этот large dataset эффективно?
def process_large_csv(file_path):
    # Файл 10GB, RAM 8GB
    # Нужно обработать каждую строку и сохранить результат
    pass
```

**🔴 Эксперт:**
- GraphQL vs REST
- Message queues (Celery, RabbitMQ)
- Stream processing

---

## 12. System Design (Senior Level)
**🔴 Архитектурный вопрос:**
Спроектируйте систему как Twitter/Instagram с нуля:
- Пользователи: 100M
- Посты в день: 10M
- Read/Write ratio: 100:1

**Ключевые аспекты:**
- Database design & sharding
- Caching strategy
- CDN для медиа
- Real-time updates
- Scaling strategy

---

## 📊 Система оценки

### Junior (40-60 баллов)
- ✅ Знание основ Python
- ✅ Понимание web разработки
- ✅ Базовые SQL и testing
- ✅ Может решить простые задачи

### Middle (60-80 баллов)
- ✅ Глубокое знание Python
- ✅ Опыт с фреймворками
- ✅ Database optimization
- ✅ Testing best practices
- ✅ Может спроектировать модуль

### Senior (80+ баллов)
- ✅ Экспертные знания
- ✅ Архитектурное мышление
- ✅ Performance optimization
- ✅ Security awareness
- ✅ Может спроектировать систему

---

## 🚀 Практические задания (выберите 1-2)

### Backend Task
Создайте REST API для URL shortener (как bit.ly):
- Создание коротких ссылок
- Редирект по короткой ссылке
- Статистика переходов
- Rate limiting
- База данных на выбор

### Architecture Task
Спроектируйте chat application:
- Real-time messaging
- Группы и приватные чаты
- Файлы и медиа
- Online presence
- Push notifications
- Масштабирование на 1M пользователей

### Performance Task
Оптимизируйте медленный API:
```python
# Дан медленный endpoint - найдите bottlenecks и оптимизируйте
@app.get("/analytics/report")
def get_analytics_report(start_date: str, end_date: str):
    users = db.session.query(User).all()
    result = []
    for user in users:
        user_stats = {
            'user_id': user.id,
            'orders': [],
            'total_spent': 0
        }
        orders = db.session.query(Order).filter(
            Order.user_id == user.id,
            Order.created_at.between(start_date, end_date)
        ).all()
        for order in orders:
            order_items = db.session.query(OrderItem).filter(
                OrderItem.order_id == order.id
            ).all()
            order_total = sum(item.price * item.quantity for item in order_items)
            user_stats['orders'].append({
                'order_id': order.id,
                'total': order_total
            })
            user_stats['total_spent'] += order_total
        result.append(user_stats)
    return result
```

---

## 💼 Soft Skills Questions

1. **Опыт проектов**: Расскажите о самом сложном проекте
2. **Problem solving**: Как решали техническую проблему в production
3. **Code review**: Как проводите/получаете feedback по коду
4. **Learning**: Как изучаете новые технологии
5. **Team work**: Опыт работы в команде, конфликты, менторство

---

## 📝 Interview Checklist

### Подготовка интервьюера
- [ ] Просмотреть резюме кандидата
- [ ] Выбрать вопросы по уровню
- [ ] Подготовить практическое задание
- [ ] Настроить среду для кодинга

### Во время интервью
- [ ] Дать кандидату выбор языка/инструментов
- [ ] Поощрять вопросы и обсуждение
- [ ] Оценивать мыслительный процесс, не только результат
- [ ] Дать время на рефакторинг и улучшение

### После интервью
- [ ] Записать детальный feedback
- [ ] Оценить технические и soft skills
- [ ] Обсудить с командой
- [ ] Дать обратную связь кандидату

---

*Помните: цель собеседования - понять, как кандидат думает и решает проблемы, а не проверить знание всех деталей Python наизусть.*