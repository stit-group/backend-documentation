# System Design Interview: Подробные ответы с примерами кода

*Comprehensive гайд с практическими решениями для backend разработчиков*

---

## 🎯 1. Масштабируемость и архитектура

### **Вопрос: Спроектируйте систему для обработки 1M RPS**

#### **Архитектурное решение:**

```yaml
# Load Balancer Configuration (Nginx)
upstream backend {
    least_conn;
    server app1:8080 weight=3;
    server app2:8080 weight=3;
    server app3:8080 weight=2;
    keepalive 32;
}

server {
    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
}
```

#### **Auto-scaling logic:**

```python
# Auto-scaling decision engine
class AutoScaler:
    def __init__(self):
        self.cpu_threshold = 70
        self.memory_threshold = 80
        self.response_time_threshold = 500  # ms
    
    def should_scale_up(self, metrics):
        return (metrics.cpu > self.cpu_threshold or 
                metrics.memory > self.memory_threshold or
                metrics.avg_response_time > self.response_time_threshold)
    
    def calculate_instances(self, current_rps, target_rps_per_instance=1000):
        # Учитываем запас 20% для пиковых нагрузок
        needed_instances = int((current_rps * 1.2) / target_rps_per_instance)
        return max(needed_instances, 2)  # минимум 2 инстанса
```

#### **Trade-offs анализ:**

| Подход | Плюсы | Минусы | Когда использовать |
|--------|-------|--------|-------------------|
| **Horizontal scaling** | Fault tolerance, linear scale | Complexity, eventual consistency | High traffic, stateless apps |
| **Vertical scaling** | Simplicity, strong consistency | Single point of failure, cost | Monoliths, ACID requirements |
| **Hybrid approach** | Best of both worlds | Complex management | Enterprise applications |

---

## 🚀 2. Микросервисы и распределенные системы

### **Вопрос: Как разбить монолит на микросервисы?**

#### **Domain-driven подход:**

```python
# Service boundary definition
class OrderService:
    """Manages order lifecycle and business logic"""
    
    async def create_order(self, order_data):
        # 1. Validate order
        if not self.validate_order(order_data):
            raise OrderValidationError()
        
        # 2. Reserve inventory (sync call)
        inventory_reserved = await self.inventory_service.reserve_items(
            order_data.items
        )
        
        # 3. Process payment (async)
        payment_event = PaymentRequestEvent(
            order_id=order.id,
            amount=order.total_amount
        )
        await self.event_bus.publish(payment_event)
        
        # 4. Create order in pending state
        return await self.order_repository.create(order_data)

# Event-driven communication
class EventBus:
    async def publish(self, event):
        # Надежная доставка через message queue
        await self.message_queue.send(
            topic=event.type,
            message=event.serialize(),
            retry_policy=ExponentialBackoff(max_retries=3)
        )
```

#### **Service communication patterns:**

```python
# Saga pattern для distributed transactions
class OrderSaga:
    def __init__(self):
        self.steps = [
            ('inventory', 'reserve_items'),
            ('payment', 'charge_card'),
            ('shipping', 'create_shipment'),
            ('notification', 'send_confirmation')
        ]
    
    async def execute(self, order_data):
        completed_steps = []
        
        try:
            for service, action in self.steps:
                result = await self.call_service(service, action, order_data)
                completed_steps.append((service, action, result))
                
        except Exception as e:
            # Compensating actions в обратном порядке
            await self.rollback(completed_steps)
            raise SagaExecutionError(f"Failed at {service}.{action}")
    
    async def rollback(self, completed_steps):
        for service, action, result in reversed(completed_steps):
            compensating_action = f"cancel_{action}"
            await self.call_service(service, compensating_action, result)
```

#### **API Gateway pattern:**

```python
# Rate limiting and routing
class APIGateway:
    def __init__(self):
        self.rate_limiter = TokenBucket()
        self.circuit_breaker = CircuitBreaker()
    
    async def route_request(self, request):
        # 1. Authentication & Authorization
        user = await self.authenticate(request.headers['Authorization'])
        if not self.authorize(user, request.path):
            return Response(status=403)
        
        # 2. Rate limiting
        if not self.rate_limiter.allow_request(user.id):
            return Response(status=429)
        
        # 3. Service discovery and routing
        service = self.discover_service(request.path)
        
        # 4. Circuit breaker protection
        return await self.circuit_breaker.call(
            service.handle_request, request
        )
```

---

## 💾 3. Данные и консистентность

### **Вопрос: SQL vs NoSQL - как выбрать?**

#### **Decision matrix:**

```python
class DatabaseSelector:
    def choose_database(self, requirements):
        score_sql = 0
        score_nosql = 0
        
        # ACID requirements
        if requirements.needs_transactions:
            score_sql += 3
        
        # Schema flexibility
        if requirements.schema_changes_frequent:
            score_nosql += 2
        
        # Scale requirements
        if requirements.read_scale > 10000:
            score_nosql += 2
        
        # Consistency requirements
        if requirements.consistency_level == 'strong':
            score_sql += 3
        elif requirements.consistency_level == 'eventual':
            score_nosql += 2
            
        return 'SQL' if score_sql > score_nosql else 'NoSQL'
```

#### **Sharding strategies:**

```python
# Hash-based sharding
class ShardManager:
    def __init__(self, shard_count=16):
        self.shard_count = shard_count
        self.shards = [f"shard_{i}" for i in range(shard_count)]
    
    def get_shard(self, user_id):
        # Consistent hashing
        shard_index = hash(str(user_id)) % self.shard_count
        return self.shards[shard_index]
    
    def redistribute_shards(self, new_shard_count):
        # Минимизируем количество перемещаемых данных
        migration_plan = []
        for old_shard in range(self.shard_count):
            new_shard = old_shard % new_shard_count
            if old_shard != new_shard:
                migration_plan.append((old_shard, new_shard))
        return migration_plan

# Range-based sharding для временных данных
class TimeBasedSharding:
    def get_shard(self, timestamp):
        # По месяцам для log данных
        year_month = timestamp.strftime('%Y_%m')
        return f"logs_{year_month}"
```

#### **CAP theorem в действии:**

```python
# Eventual consistency пример
class EventuallyConsistentCache:
    def __init__(self):
        self.local_cache = {}
        self.write_queue = asyncio.Queue()
        self.sync_interval = 30  # секунд
    
    async def write(self, key, value):
        # Записываем локально сразу (Available)
        self.local_cache[key] = value
        
        # Асинхронно синхронизируем (Eventually Consistent)
        await self.write_queue.put((key, value, time.time()))
        
    async def sync_worker(self):
        while True:
            try:
                key, value, timestamp = await self.write_queue.get()
                # Попытка записи в основное хранилище
                await self.persistent_store.write(key, value)
                
            except NetworkError:
                # Partition tolerance - продолжаем работать
                # Retry позже
                await asyncio.sleep(self.sync_interval)
                await self.write_queue.put((key, value, timestamp))
```

---

## ⚡ 4. Кэширование и производительность

### **Multi-level caching strategy:**

```python
class CacheManager:
    def __init__(self):
        self.l1_cache = {}  # In-memory
        self.l2_cache = RedisClient()  # Distributed
        self.l3_cache = CDNClient()  # Edge
    
    async def get(self, key):
        # L1: Application cache
        if key in self.l1_cache:
            return self.l1_cache[key]
        
        # L2: Redis cache
        value = await self.l2_cache.get(key)
        if value:
            self.l1_cache[key] = value  # Populate L1
            return value
        
        # L3: Database
        value = await self.database.get(key)
        if value:
            # Populate all cache levels
            self.l1_cache[key] = value
            await self.l2_cache.set(key, value, ttl=3600)
            return value
        
        return None
    
    async def invalidate(self, key):
        # Invalidate all levels
        self.l1_cache.pop(key, None)
        await self.l2_cache.delete(key)
        await self.l3_cache.purge(key)
```

#### **Cache warming strategy:**

```python
class CacheWarmer:
    async def warm_popular_content(self):
        # Получаем популярный контент из аналитики
        popular_items = await self.analytics.get_trending_items(limit=1000)
        
        # Параллельный прогрев
        tasks = [
            self.cache_manager.preload(item.id, item.data)
            for item in popular_items
        ]
        
        await asyncio.gather(*tasks, return_exceptions=True)
    
    async def predictive_warming(self, user_id):
        # ML-based предсказание что понадобится пользователю
        predictions = await self.ml_service.predict_user_interests(user_id)
        
        for item_id in predictions[:50]:  # Top 50 predictions
            await self.cache_manager.preload(item_id)
```

---

## 🛡️ 5. Отказоустойчивость

### **Circuit Breaker pattern:**

```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, timeout=60):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.failure_count = 0
        self.last_failure_time = None
        self.state = 'CLOSED'  # CLOSED, OPEN, HALF_OPEN
    
    async def call(self, func, *args, **kwargs):
        if self.state == 'OPEN':
            if time.time() - self.last_failure_time > self.timeout:
                self.state = 'HALF_OPEN'
            else:
                raise CircuitBreakerOpenError()
        
        try:
            result = await func(*args, **kwargs)
            
            # Success - reset failure count
            if self.state == 'HALF_OPEN':
                self.state = 'CLOSED'
            self.failure_count = 0
            
            return result
            
        except Exception as e:
            self.failure_count += 1
            self.last_failure_time = time.time()
            
            if self.failure_count >= self.failure_threshold:
                self.state = 'OPEN'
            
            raise e
```

#### **Retry with exponential backoff:**

```python
class RetryManager:
    async def execute_with_retry(self, func, max_retries=3, base_delay=1):
        for attempt in range(max_retries + 1):
            try:
                return await func()
                
            except TransientError as e:
                if attempt == max_retries:
                    raise e
                
                # Exponential backoff с jitter
                delay = base_delay * (2 ** attempt)
                jitter = random.uniform(0.1, 0.9)
                await asyncio.sleep(delay * jitter)
                
            except PermanentError:
                # Не ретраим permanent ошибки
                raise
```

#### **Graceful degradation:**

```python
class FeatureToggle:
    def __init__(self):
        self.features = {
            'recommendations': {'enabled': True, 'fallback': 'popular_items'},
            'real_time_notifications': {'enabled': True, 'fallback': 'email_digest'},
            'advanced_search': {'enabled': True, 'fallback': 'basic_search'}
        }
    
    async def get_recommendations(self, user_id):
        if self.is_enabled('recommendations'):
            try:
                return await self.ml_service.get_recommendations(user_id)
            except ServiceUnavailable:
                # Fallback to simple logic
                return await self.get_popular_items()
        else:
            return await self.get_popular_items()
    
    def is_enabled(self, feature):
        return self.features.get(feature, {}).get('enabled', False)
```

---

## 📊 6. Мониторинг и метрики

### **Application metrics:**

```python
class MetricsCollector:
    def __init__(self):
        self.counters = defaultdict(int)
        self.timers = defaultdict(list)
        self.gauges = defaultdict(float)
    
    @contextmanager
    def time_operation(self, operation_name):
        start_time = time.time()
        try:
            yield
        finally:
            duration = time.time() - start_time
            self.timers[operation_name].append(duration)
            
            # SLA monitoring
            if duration > self.get_sla_threshold(operation_name):
                self.counters[f"{operation_name}_sla_violations"] += 1
    
    async def export_metrics(self):
        return {
            'counters': dict(self.counters),
            'timers_p95': {
                name: np.percentile(times, 95) 
                for name, times in self.timers.items()
            },
            'gauges': dict(self.gauges)
        }

# Usage example
@metrics.time_operation('user_authentication')
async def authenticate_user(token):
    # Implementation
    pass
```

#### **Health checks:**

```python
class HealthChecker:
    def __init__(self):
        self.checks = {
            'database': self.check_database,
            'redis': self.check_redis,
            'external_api': self.check_external_services
        }
    
    async def health_status(self):
        results = {}
        overall_healthy = True
        
        for name, check_func in self.checks.items():
            try:
                await asyncio.wait_for(check_func(), timeout=5.0)
                results[name] = {'status': 'healthy', 'response_time': '< 5s'}
            except Exception as e:
                results[name] = {'status': 'unhealthy', 'error': str(e)}
                overall_healthy = False
        
        return {
            'status': 'healthy' if overall_healthy else 'degraded',
            'checks': results,
            'timestamp': datetime.utcnow().isoformat()
        }
```

---

## 🎪 7. Практические сценарии

### **Social Feed Generation:**

```python
class FeedGenerator:
    def __init__(self):
        self.fan_out_threshold = 1000  # followers
    
    async def generate_feed(self, user_id, limit=50):
        # Hybrid pull/push approach
        user = await self.user_service.get_user(user_id)
        
        if user.followers_count > self.fan_out_threshold:
            # Pull model для популярных пользователей
            return await self.pull_feed(user_id, limit)
        else:
            # Push model для обычных пользователей
            return await self.get_precomputed_feed(user_id, limit)
    
    async def pull_feed(self, user_id, limit):
        # Получаем список подписок
        following = await self.get_following(user_id)
        
        # Параллельно получаем последние посты
        tasks = [
            self.get_recent_posts(followed_id, limit=10)
            for followed_id in following[:100]  # Ограничиваем
        ]
        
        all_posts = await asyncio.gather(*tasks)
        
        # Merge и сортировка по времени
        merged_posts = sorted(
            [post for posts in all_posts for post in posts],
            key=lambda x: x.created_at,
            reverse=True
        )
        
        return merged_posts[:limit]
```

### **Real-time Chat System:**

```python
class ChatServer:
    def __init__(self):
        self.connections = {}  # user_id -> websocket
        self.rooms = defaultdict(set)  # room_id -> {user_ids}
    
    async def handle_message(self, websocket, message):
        data = json.loads(message)
        
        if data['type'] == 'join_room':
            await self.join_room(websocket, data['room_id'], data['user_id'])
        
        elif data['type'] == 'send_message':
            await self.broadcast_message(data['room_id'], data)
    
    async def broadcast_message(self, room_id, message):
        # Получаем всех участников комнаты
        participants = self.rooms[room_id]
        
        # Параллельная отправка
        tasks = []
        for user_id in participants:
            if user_id in self.connections:
                websocket = self.connections[user_id]
                tasks.append(self.send_safe(websocket, message))
        
        await asyncio.gather(*tasks, return_exceptions=True)
        
        # Persist message для offline пользователей
        await self.message_store.save(message)
    
    async def send_safe(self, websocket, message):
        try:
            await websocket.send(json.dumps(message))
        except ConnectionClosed:
            # Cleanup disconnected user
            await self.cleanup_connection(websocket)
```

---

## 🚨 Типичные ошибки и решения

### **1. Overengineering:**
```python
# ❌ Плохо: Premature optimization
class UserService:
    def __init__(self):
        self.cache = RedisCluster(nodes=50)
        self.database = ShardedDatabase(shards=100)
        self.event_bus = KafkaCluster(brokers=20)

# ✅ Хорошо: Start simple, scale when needed
class UserService:
    def __init__(self):
        self.database = PostgreSQL()
        self.cache = Redis()  # Single instance
    
    def get_user(self, user_id):
        # Простое кэширование
        cached = self.cache.get(f"user:{user_id}")
        if cached:
            return json.loads(cached)
        
        user = self.database.get_user(user_id)
        self.cache.setex(f"user:{user_id}", 3600, json.dumps(user))
        return user
```

### **2. Ignoring CAP theorem:**
```python
# ❌ Плохо: Expecting strong consistency everywhere
async def transfer_money(from_account, to_account, amount):
    # Это не будет работать в distributed системе
    await self.accounts_db.decrease_balance(from_account, amount)
    await self.accounts_db.increase_balance(to_account, amount)

# ✅ Хорошо: Use transactions or sagas
async def transfer_money(from_account, to_account, amount):
    async with self.database.transaction():
        await self.accounts_db.decrease_balance(from_account, amount)
        await self.accounts_db.increase_balance(to_account, amount)
```

---

## 🎯 Ключевые takeaways

1. **Всегда начинайте с простого решения** - преждевременная оптимизация корень зла
2. **Учитывайте trade-offs** - нет серебряной пули, все решения имеют компромиссы  
3. **Думайте о failure modes** - что произойдет когда что-то сломается?
4. **Мониторинг с самого начала** - нельзя улучшить то, что не измеряешь
5. **Знайте свои numbers** - latency, throughput, storage requirements

**Помните:** Лучший system design - тот, который решает бизнес-задачи с минимальной complexity!