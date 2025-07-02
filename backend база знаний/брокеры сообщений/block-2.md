# БЛОК 2: ПОПУЛЯРНЫЕ РЕШЕНИЯ И ИХ ОСОБЕННОСТИ

---

## 🎯 **ЦЕЛЬ БЛОКА**
**Изучение основных брокеров сообщений и их сравнительный анализ**

**Продолжительность:** 4-5 недель  
**Результат:** Способность выбрать оптимальный брокер для конкретной задачи

---

## 📋 **СТРУКТУРА БЛОКА**

```
БЛОК 2
├── 2.1 Apache Kafka (1.5 недели)
├── 2.2 RabbitMQ (1.5 недели)  
├── 2.3 ActiveMQ & Amazon SQS (1 неделя)
├── 2.4 Redis Pub/Sub & Apache Pulsar (0.5 недели)
└── 2.5 Сравнительный анализ (0.5 недели)
```

---

## 🔥 **ГЛАВА 2.1: APACHE KAFKA**

### **Что такое Apache Kafka?**

Apache Kafka — это **распределенная платформа для потоковой передачи данных**, которая работает как высокопроизводительный, масштабируемый и отказоустойчивый брокер сообщений.

---

### **📊 Архитектура Kafka**

```
┌─────────────────────────────────────────────────────────────┐
│                    KAFKA CLUSTER                           │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   Broker 1  │  │   Broker 2  │  │   Broker 3  │        │
│  │             │  │             │  │             │        │
│  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │        │
│  │ │Topic A  │ │  │ │Topic A  │ │  │ │Topic B  │ │        │
│  │ │Part 0   │ │  │ │Part 1   │ │  │ │Part 0   │ │        │
│  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────┘
           ▲                               ▲
           │                               │
    ┌─────────────┐                ┌─────────────┐
    │  PRODUCER   │                │  CONSUMER   │
    │             │                │             │
    │ ┌─────────┐ │                │ ┌─────────┐ │
    │ │Send Msg │ │                │ │Read Msg │ │
    │ └─────────┘ │                │ └─────────┘ │
    └─────────────┘                └─────────────┘
```

---

### **🔑 Ключевые компоненты**

| Компонент | Описание | Роль |
|-----------|----------|------|
| **Broker** | Сервер Kafka | Хранит и передает сообщения |
| **Topic** | Категория сообщений | Логическое разделение данных |
| **Partition** | Часть топика | Физическое распределение и параллелизм |
| **Producer** | Отправитель | Публикует сообщения в топики |
| **Consumer** | Получатель | Читает сообщения из топиков |
| **Consumer Group** | Группа консьюмеров | Распределенное чтение |

---

### **⚙️ Kafka как Distributed Commit Log**

```
Topic: user-events
┌────────────────────────────────────────────────────────────┐
│ Partition 0: [msg1][msg2][msg3][msg4]...                  │
│ Partition 1: [msg5][msg6][msg7][msg8]...                  │  
│ Partition 2: [msg9][msg10][msg11][msg12]...               │
└────────────────────────────────────────────────────────────┘
             ▲      ▲      ▲      ▲
          offset:  0      1      2      3
```

**Принципы:**
- ✅ **Immutable log** — сообщения не изменяются
- ✅ **Ordered within partition** — порядок в рамках партиции
- ✅ **Durable storage** — данные сохраняются на диск
- ✅ **Configurable retention** — настраиваемое время хранения

---

### **🔧 Zookeeper и его роль**

```
                    ┌─────────────────┐
                    │   ZOOKEEPER     │
                    │   ENSEMBLE      │
                    │                 │
                    │ ┌─────────────┐ │
                    │ │ Metadata    │ │
                    │ │ Coordination│ │
                    │ │ Leader      │ │
                    │ │ Election    │ │
                    │ └─────────────┘ │
                    └─────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
  ┌───────────┐       ┌───────────┐       ┌───────────┐
  │ Kafka     │       │ Kafka     │       │ Kafka     │
  │ Broker 1  │       │ Broker 2  │       │ Broker 3  │
  │           │       │           │       │           │
  │ (Leader)  │       │(Follower) │       │(Follower) │
  └───────────┘       └───────────┘       └───────────┘
```

**Функции Zookeeper:**
- 🎯 **Broker discovery** — регистрация и обнаружение брокеров
- 🎯 **Topic configuration** — метаданные топиков и партиций
- 🎯 **Leader election** — выбор лидера для партиций
- 🎯 **Consumer group coordination** — управление группами

---

### **🌊 Kafka Streams и Kafka Connect**

#### **Kafka Streams**
```
Input Topic → [Kafka Streams App] → Output Topic
     │              │                    │
     │         ┌──────────┐               │
     │         │Transform │               │
     │         │ Filter   │               │
     │         │Aggregate │               │
     │         └──────────┘               │
     │                                    │
   Events                            Processed
                                      Events
```

#### **Kafka Connect**
```
┌─────────────┐    ┌──────────────┐    ┌─────────────┐
│  Database   │───▶│    Source    │───▶│    Kafka    │
│             │    │  Connector   │    │   Cluster   │
└─────────────┘    └──────────────┘    └─────────────┘
                                              │
┌─────────────┐    ┌──────────────┐          │
│   Search    │◀───│    Sink      │◀─────────┘
│   Engine    │    │  Connector   │
└─────────────┘    └──────────────┘
```

---

### **🛠️ Практическое задание: Установка и настройка Kafka**

#### **Шаг 1: Установка через Docker Compose**

```yaml
version: '3'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

#### **Шаг 2: Создание топика**
```bash
# Создание топика
kafka-topics --create --topic user-events \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1

# Просмотр топиков
kafka-topics --list --bootstrap-server localhost:9092
```

#### **Шаг 3: Producer и Consumer**
```bash
# Producer
kafka-console-producer --topic user-events \
  --bootstrap-server localhost:9092

# Consumer
kafka-console-consumer --topic user-events \
  --from-beginning \
  --bootstrap-server localhost:9092
```

---

## 🐰 **ГЛАВА 2.2: RABBITMQ**

### **Что такое RabbitMQ?**

RabbitMQ — это **message broker**, реализующий протокол **AMQP (Advanced Message Queuing Protocol)** и предоставляющий гибкие возможности маршрутизации сообщений.

---

### **📊 Архитектура RabbitMQ**

```
┌─────────────────────────────────────────────────────────────┐
│                     RABBITMQ BROKER                        │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    │
│  │  EXCHANGE   │    │  EXCHANGE   │    │  EXCHANGE   │    │
│  │   (direct)  │    │  (fanout)   │    │   (topic)   │    │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘    │
│         │                  │                  │           │
│    ┌────▼────┐        ┌────▼────┐        ┌────▼────┐      │
│    │ Queue A │        │ Queue B │        │ Queue C │      │
│    └─────────┘        └─────────┘        └─────────┘      │
└─────────────────────────────────────────────────────────────┘
        ▲                                           ▲
        │                                           │
┌───────────────┐                           ┌───────────────┐
│   PRODUCER    │                           │   CONSUMER    │
│               │                           │               │
│ ┌───────────┐ │                           │ ┌───────────┐ │
│ │ Publish   │ │                           │ │ Subscribe │ │
│ │ Message   │ │                           │ │& Consume  │ │
│ └───────────┘ │                           │ └───────────┘ │
└───────────────┘                           └───────────────┘
```

---

### **🔑 AMQP Протокол и компоненты**

| Компонент | Описание | Функция |
|-----------|----------|---------|
| **Producer** | Отправитель сообщений | Публикует сообщения в exchanges |
| **Exchange** | Маршрутизатор | Определяет, в какие очереди направить сообщение |
| **Queue** | Хранилище сообщений | Буферизует сообщения для consumers |
| **Consumer** | Получатель | Читает и обрабатывает сообщения |
| **Binding** | Связь | Соединяет exchange с queue по правилам |
| **Routing Key** | Ключ маршрутизации | Определяет путь сообщения |

---

### **🎯 Типы Exchanges**

#### **1. Direct Exchange**
```
Producer ──["user.created"]──▶ Direct Exchange
                                      │
                               ┌──────┼──────┐
                               │      │      │
                        [user.created] │ [user.updated]
                               │      │      │
                               ▼      │      ▼
                           Queue A    │  Queue B
                                      │
                                [user.deleted]
                                      │
                                      ▼
                                  Queue C
```

#### **2. Fanout Exchange**
```
Producer ──[any key]──▶ Fanout Exchange
                              │
                    ┌─────────┼─────────┐
                    │         │         │
                    ▼         ▼         ▼
                Queue A   Queue B   Queue C
                    │         │         │
                    ▼         ▼         ▼
               Consumer1  Consumer2  Consumer3
```

#### **3. Topic Exchange**
```
Producer ──["user.profile.updated"]──▶ Topic Exchange
                                            │
                                 ┌──────────┼──────────┐
                                 │          │          │
                           [user.#]    [*.profile.*]  [user.profile.#]
                                 │          │          │
                                 ▼          ▼          ▼
                             Queue A    Queue B    Queue C
```

**Wildcard patterns:**
- `*` (star) — заменяет ровно одно слово
- `#` (hash) — заменяет ноль или более слов

#### **4. Headers Exchange**
```
Producer ──[headers: {type: "email", priority: "high"}]──▶ Headers Exchange
                                                                │
                                              ┌─────────────────┼─────────────────┐
                                              │                 │                 │
                                    [type: "email"]    [priority: "high"]   [type: "sms"]
                                              │                 │                 │
                                              ▼                 ▼                 ▼
                                          Queue A           Queue B           Queue C
```

---

### **🔧 Bindings и Routing Keys**

```
┌─────────────┐  routing_key:  ┌─────────────┐  binding_key:  ┌─────────────┐
│  Producer   │ "user.created" │  Exchange   │ "user.created" │   Queue     │
│             ├───────────────▶│  (direct)   ├───────────────▶│             │
└─────────────┘                └─────────────┘                └─────────────┘

Правило: routing_key === binding_key → сообщение доставляется
```

---

### **🏠 Virtual Hosts и пользователи**

```
┌─────────────────────────────────────────────────────────────┐
│                    RABBITMQ SERVER                         │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐    ┌─────────────────┐                │
│  │   VHost: /app1  │    │   VHost: /app2  │                │
│  │                 │    │                 │                │
│  │ ┌─────────────┐ │    │ ┌─────────────┐ │                │
│  │ │ Exchanges   │ │    │ │ Exchanges   │ │                │
│  │ │ Queues      │ │    │ │ Queues      │ │                │
│  │ │ Bindings    │ │    │ │ Bindings    │ │                │
│  │ └─────────────┘ │    │ └─────────────┘ │                │
│  └─────────────────┘    └─────────────────┘                │
│                                                             │
│  Users & Permissions:                                       │
│  ┌─────────────────────────────────────────────────────────┤
│  │ admin:  /app1 (configure, write, read)                 │
│  │ app1:   /app1 (write, read)                             │
│  │ app2:   /app2 (write, read)                             │
│  └─────────────────────────────────────────────────────────┤
└─────────────────────────────────────────────────────────────┘
```

---

### **🔗 Clustering и High Availability**

#### **Cluster Architecture**
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Node 1     │    │   Node 2    │    │   Node 3    │
│             │    │             │    │             │
│ ┌─────────┐ │    │ ┌─────────┐ │    │ ┌─────────┐ │
│ │Queue A  │ │◀──▶│ │Queue A  │ │◀──▶│ │Queue A  │ │
│ │(Master) │ │    │ │(Mirror) │ │    │ │(Mirror) │ │
│ └─────────┘ │    │ └─────────┘ │    │ └─────────┘ │
│             │    │             │    │             │
│ ┌─────────┐ │    │ ┌─────────┐ │    │ ┌─────────┐ │
│ │Queue B  │ │◀──▶│ │Queue B  │ │◀──▶│ │Queue B  │ │
│ │(Mirror) │ │    │ │(Master) │ │    │ │(Mirror) │ │
│ └─────────┘ │    │ └─────────┘ │    │ └─────────┘ │
└─────────────┘    └─────────────┘    └─────────────┘
```

#### **Queue Mirroring Policies**
```bash
# Создание HA policy
rabbitmqctl set_policy ha-all "^ha\." '{"ha-mode":"all"}'

# Создание policy с specific nodes
rabbitmqctl set_policy ha-two "^two\." \
  '{"ha-mode":"exactly","ha-params":2,"ha-sync-mode":"automatic"}'
```

---

### **🛠️ Практическое задание: Настройка RabbitMQ**

#### **Шаг 1: Установка через Docker**
```yaml
version: '3'
services:
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin123
```

#### **Шаг 2: Создание Exchange и Queue**
```bash
# Объявление exchange
rabbitmqadmin declare exchange name=user_events type=topic

# Объявление queue
rabbitmqadmin declare queue name=user_notifications

# Создание binding
rabbitmqadmin declare binding source=user_events \
  destination=user_notifications \
  routing_key="user.#"
```

#### **Шаг 3: Publisher и Consumer**

**Publisher (Python):**
```python
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.basic_publish(
    exchange='user_events',
    routing_key='user.profile.updated',
    body='{"user_id": 123, "action": "profile_update"}'
)
```

**Consumer (Python):**
```python
def callback(ch, method, properties, body):
    print(f"Received: {body}")
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_consume(
    queue='user_notifications',
    on_message_callback=callback
)
```

---

## ⚡ **ГЛАВА 2.3: APACHE ACTIVEMQ И AMAZON SQS**

### **Apache ActiveMQ**

#### **Что такое ActiveMQ?**
Apache ActiveMQ — это **Java-based message broker**, поддерживающий множество протоколов (JMS, AMQP, STOMP, MQTT, WebSocket).

```
┌─────────────────────────────────────────────────────────────┐
│                   ACTIVEMQ BROKER                          │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    │
│  │    JMS      │    │    AMQP     │    │    STOMP    │    │
│  │  Protocol   │    │  Protocol   │    │  Protocol   │    │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘    │
│         │                  │                  │           │
│    ┌────▼─────────────────────────────────────▼────┐      │
│    │            Message Store                      │      │
│    │  ┌─────────┐  ┌─────────┐  ┌─────────┐      │      │
│    │  │Queue A  │  │Queue B  │  │Topic C  │      │      │
│    │  └─────────┘  └─────────┘  └─────────┘      │      │
│    └───────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

#### **JMS (Java Message Service)**
```
┌─────────────┐                    ┌─────────────┐
│   JMS       │    JMS Queue       │    JMS      │
│ Producer    ├───────────────────▶│  Consumer   │
└─────────────┘                    └─────────────┘

┌─────────────┐                    ┌─────────────┐
│   JMS       │                    │    JMS      │
│ Publisher   │    JMS Topic       │ Subscriber1 │
│             ├───────┬───────────▶│             │
└─────────────┘       │            └─────────────┘
                      │            ┌─────────────┐
                      │            │    JMS      │
                      └───────────▶│ Subscriber2 │
                                   └─────────────┘
```

#### **Network Connectors**
```
┌─────────────┐    Network     ┌─────────────┐    Network     ┌─────────────┐
│  Broker A   │   Connector    │  Broker B   │   Connector    │  Broker C   │
│ (New York)  ├───────────────▶│  (London)   ├───────────────▶│  (Tokyo)    │
│             │                │             │                │             │
│ ┌─────────┐ │                │ ┌─────────┐ │                │ ┌─────────┐ │
│ │Queue X  │ │                │ │Queue X  │ │                │ │Queue X  │ │
│ └─────────┘ │                │ └─────────┘ │                │ └─────────┘ │
└─────────────┘                └─────────────┘                └─────────────┘

Producer ──▶ Broker A ──▶ Message forwarded to other brokers ──▶ Consumers
```

---

### **Amazon SQS**

#### **Что такое Amazon SQS?**
Amazon Simple Queue Service — это **полностью управляемый сервис очередей сообщений** в облаке AWS.

```
┌─────────────────────────────────────────────────────────────┐
│                      AWS CLOUD                             │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┤
│  │                  SQS SERVICE                           │
│  │                                                        │
│  │  ┌─────────────┐    ┌─────────────┐                   │
│  │  │ Standard    │    │ FIFO Queue  │                   │
│  │  │ Queue       │    │             │                   │
│  │  │             │    │ ┌─────────┐ │                   │
│  │  │ ┌─────────┐ │    │ │Ordered  │ │                   │
│  │  │ │At-least │ │    │ │Exactly  │ │                   │
│  │  │ │once     │ │    │ │once     │ │                   │
│  │  │ │delivery │ │    │ │delivery │ │                   │
│  │  │ └─────────┘ │    │ └─────────┘ │                   │
│  │  └─────────────┘    └─────────────┘                   │
│  └─────────────────────────────────────────────────────────┤
└─────────────────────────────────────────────────────────────┘
           ▲                               ▲
           │                               │
    ┌─────────────┐                ┌─────────────┐
    │  Producer   │                │  Consumer   │
    │ Application │                │ Application │
    └─────────────┘                └─────────────┘
```

#### **Standard vs FIFO очереди**

| Характеристика | Standard Queue | FIFO Queue |
|----------------|----------------|------------|
| **Throughput** | Неограниченный | 300 TPS (или 3000 с batching) |
| **Delivery** | At-least-once | Exactly-once |
| **Ordering** | Best-effort | Строгий порядок |
| **Deduplication** | Нет | Есть (5 минут) |
| **Price** | Дешевле | Дороже |

#### **Visibility Timeout**
```
Timeline: ──────────────────────────────────────────────────▶

Message in Queue:     [████████████████████████████████████]
                           ▲                        ▲
                       Consumer                Message
                       receives               visible again
                       message                (if not deleted)
                           │                        │
                           └────────────────────────┘
                           Visibility Timeout Period
                              (e.g., 30 seconds)
```

---

### **🛠️ Практическое задание: Работа с ActiveMQ и SQS**

#### **ActiveMQ Setup**
```xml
<!-- activemq.xml configuration -->
<broker xmlns="http://activemq.apache.org/schema/core" 
        brokerName="localhost" 
        dataDirectory="${activemq.data}">
    
    <destinationPolicy>
        <policyMap>
            <policyEntries>
                <policyEntry topic=">" >
                    <pendingMessageLimitStrategy>
                        <constantPendingMessageLimitStrategy limit="1000"/>
                    </pendingMessageLimitStrategy>
                </policyEntry>
            </policyEntries>
        </policyMap>
    </destinationPolicy>

    <transportConnectors>
        <transportConnector name="openwire" 
                          uri="tcp://0.0.0.0:61616?maximumConnections=1000"/>
    </transportConnectors>
</broker>
```

#### **AWS SQS с Boto3**
```python
import boto3

# Создание SQS клиента
sqs = boto3.client('sqs', region_name='us-east-1')

# Создание FIFO очереди
response = sqs.create_queue(
    QueueName='my-orders.fifo',
    Attributes={
        'FifoQueue': 'true',
        'ContentBasedDeduplication': 'true',
        'VisibilityTimeoutSeconds': '60'
    }
)

# Отправка сообщения
sqs.send_message(
    QueueUrl=queue_url,
    MessageBody='Order processed',
    MessageGroupId='orders',
    MessageDeduplicationId='order-123'
)
```

---

## 🚀 **ГЛАВА 2.4: REDIS PUB/SUB И APACHE PULSAR**

### **Redis Pub/Sub**

#### **Что такое Redis как Message Broker?**
Redis — это **in-memory структура данных**, которая также предоставляет функциональность **publish/subscribe**.

```
┌─────────────────────────────────────────────────────────────┐
│                     REDIS SERVER                           │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┤
│  │                 PUB/SUB ENGINE                         │
│  │                                                        │
│  │  Channel: "user:events"                                │
│  │  ┌─────────────────────────────────────────────────────┤
│  │  │ Subscribers:                                        │
│  │  │ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐   │
│  │  │ │Subscriber 1 │ │Subscriber 2 │ │Subscriber 3 │   │
│  │  │ └─────────────┘ └─────────────┘ └─────────────┘   │
│  │  └─────────────────────────────────────────────────────┤
│  │                                                        │
│  │  Pattern: "user:*"                                     │
│  │  ┌─────────────────────────────────────────────────────┤
│  │  │ Pattern Subscribers:                                │
│  │  │ ┌─────────────┐ ┌─────────────┐                   │
│  │  │ │Subscriber A │ │Subscriber B │                   │
│  │  │ └─────────────┘ └─────────────┘                   │
│  │  └─────────────────────────────────────────────────────┤
│  └─────────────────────────────────────────────────────────┤
└─────────────────────────────────────────────────────────────┘
```

#### **Особенности Redis Pub/Sub**
- ✅ **Fire-and-forget** — сообщения не сохраняются
- ✅ **Low latency** — очень быстрая доставка
- ❌ **No persistence** — сообщения теряются при отключении
- ❌ **No acknowledgments** — нет гарантий доставки

#### **Redis Streams (альтернатива)**
```
Stream: user-events
┌────────────────────────────────────────────────────────────┐
│ 1609459200000-0: {user: "john", action: "login"}           │
│ 1609459201000-0: {user: "jane", action: "logout"}          │
│ 1609459202000-0: {user: "bob", action: "purchase"}         │
│ 1609459203000-0: {user: "alice", action: "register"}       │
└────────────────────────────────────────────────────────────┘
         ▲                    ▲                    ▲
    Consumer A            Consumer B          Consumer C
   (Group: web)         (Group: web)      (Group: analytics)
```

---

### **Apache Pulsar**

#### **Что такое Apache Pulsar?**
Apache Pulsar — это **cloud-native distributed messaging platform** с поддержкой **multi-tenancy** и **geo-replication**.

```
┌─────────────────────────────────────────────────────────────┐
│                   PULSAR CLUSTER                           │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┤
│  │              BROKER LAYER                              │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐      │
│  │  │  Broker 1   │ │  Broker 2   │ │  Broker 3   │      │
│  │  │ (Stateless) │ │ (Stateless) │ │ (Stateless) │      │
│  │  └─────────────┘ └─────────────┘ └─────────────┘      │
│  └─────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┤
│  │              STORAGE LAYER                             │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐      │
│  │  │  BookKeeper │ │  BookKeeper │ │  BookKeeper │      │
│  │  │   Bookie 1  │ │   Bookie 2  │ │   Bookie 3  │      │
│  │  └─────────────┘ └─────────────┘ └─────────────┘      │
│  └─────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┤
│  │            COORDINATION LAYER                          │
│  │                  ┌─────────────┐                       │
│  │                  │  ZooKeeper  │                       │
│  │                  │   Cluster   │                       │
│  │                  └─────────────┘                       │
│  └─────────────────────────────────────────────────────────┤
└─────────────────────────────────────────────────────────────┘
```

#### **Multi-tenancy в Pulsar**
```
Cluster: pulsar-prod
├── Tenant: company-a
│   ├── Namespace: company-a/marketing
│   │   ├── Topic: user-events
│   │   └── Topic: campaign-metrics
│   └── Namespace: company-a/sales
│       ├── Topic: orders
│       └── Topic: payments
└── Tenant: company-b
    ├── Namespace: company-b/analytics
    │   └── Topic: clickstream
    └── Namespace: company-b/notifications
        └── Topic: email-queue
```

#### **Geo-replication**
```
┌─────────────────┐    Replication   ┌─────────────────┐
│  Cluster USA    │◀────────────────▶│  Cluster EU     │
│                 │                  │                 │
│ Topic: orders   │                  │ Topic: orders   │
│ (Partitioned)   │                  │ (Partitioned)   │
└─────────────────┘                  └─────────────────┘
         ▲                                     ▲
         │                                     │
    ┌─────────────┐                      ┌─────────────┐
    │   US        │                      │    EU       │
    │ Producers   │                      │ Producers   │
    │& Consumers  │                      │& Consumers  │
    └─────────────┘                      └─────────────┘
```

---

### **🔄 Сравнение с Kafka**

| Характеристика | Apache Kafka | Apache Pulsar |
|----------------|--------------|---------------|
| **Architecture** | Monolithic brokers | Layered (compute + storage) |
| **Storage** | Local disk | BookKeeper (distributed) |
| **Multi-tenancy** | Limited | Native support |
| **Geo-replication** | MirrorMaker | Built-in |
| **Message retention** | Time/size based | Until acknowledged |
| **Rebalancing** | Stop-the-world | Instant |
| **Protocol** | Custom binary | Custom binary + HTTP |

---

### **🛠️ Практическое задание: Redis Pub/Sub и Pulsar**

#### **Redis Pub/Sub пример**
```python
import redis

# Publisher
r = redis.Redis(host='localhost', port=6379, db=0)
r.publish('user:events', '{"user": "john", "action": "login"}')

# Subscriber
pubsub = r.pubsub()
pubsub.subscribe('user:events')

for message in pubsub.listen():
    if message['type'] == 'message':
        print(f"Received: {message['data']}")
```

#### **Pulsar пример**
```python
import pulsar

# Producer
client = pulsar.Client('pulsar://localhost:6650')
producer = client.create_producer('persistent://public/default/my-topic')

producer.send('Hello Pulsar'.encode('utf-8'))

# Consumer
consumer = client.subscribe('persistent://public/default/my-topic',
                          subscription_name='my-subscription')

while True:
    msg = consumer.receive()
    print(f"Received: {msg.data()}")
    consumer.acknowledge(msg)
```

---

## 📊 **ГЛАВА 2.5: СРАВНИТЕЛЬНЫЙ АНАЛИЗ И ВЫБОР РЕШЕНИЯ**

### **🎯 Критерии выбора брокера**

#### **Матрица принятия решений**

| Критерий | Вес | Kafka | RabbitMQ | ActiveMQ | SQS | Redis | Pulsar |
|----------|-----|-------|----------|----------|-----|-------|---------|
| **Throughput** | 25% | 9 | 7 | 6 | 8 | 8 | 9 |
| **Latency** | 20% | 7 | 8 | 7 | 6 | 9 | 8 |
| **Durability** | 20% | 9 | 8 | 8 | 9 | 3 | 9 |
| **Ease of Use** | 15% | 6 | 8 | 7 | 9 | 9 | 7 |
| **Scalability** | 10% | 9 | 6 | 5 | 9 | 7 | 9 |
| **Community** | 5% | 9 | 8 | 7 | 8 | 8 | 6 |
| **Cost** | 5% | 8 | 7 | 8 | 6 | 9 | 7 |

#### **Weighted Score Calculation:**
```
Kafka Score    = (9×0.25 + 7×0.20 + 9×0.20 + 6×0.15 + 9×0.10 + 9×0.05 + 8×0.05) = 8.0
RabbitMQ Score = (7×0.25 + 8×0.20 + 8×0.20 + 8×0.15 + 6×0.10 + 8×0.05 + 7×0.05) = 7.6
Pulsar Score   = (9×0.25 + 8×0.20 + 9×0.20 + 7×0.15 + 9×0.10 + 6×0.05 + 7×0.05) = 8.2
SQS Score      = (8×0.25 + 6×0.20 + 9×0.20 + 9×0.15 + 9×0.10 + 8×0.05 + 6×0.05) = 7.9
```

---

### **🎯 Рекомендации по использованию**

#### **Используйте Kafka когда:**
```
✅ Высокий throughput (100K+ messages/sec)
✅ Stream processing и real-time analytics  
✅ Event sourcing и audit logs
✅ Long-term storage сообщений
✅ Ecosystem интеграция (Confluent, Streams, Connect)

❌ Сложность setup и management
❌ Требует ZooKeeper
❌ Limited routing capabilities
```

#### **Используйте RabbitMQ когда:**
```
✅ Гибкая маршрутизация сообщений
✅ Enterprise integration patterns
✅ Protocol diversity (AMQP, STOMP, MQTT)
✅ Rich management UI
✅ Strong consistency requirements

❌ Limited throughput по сравнению с Kafka
❌ Single point of failure (без clustering)
❌ Memory-based storage limits
```

#### **Используйте SQS когда:**
```
✅ AWS-native applications
✅ Managed service (no ops overhead)
✅ Auto-scaling requirements
✅ Cost-effective для small-medium workloads
✅ FIFO guarantees нужны

❌ Vendor lock-in
❌ Limited throughput (FIFO)
❌ No complex routing
❌ Higher latency vs self-hosted
```

#### **Используйте Pulsar когда:**
```
✅ Multi-tenant environments
✅ Geo-distributed applications
✅ Cloud-native architecture
✅ Instant rebalancing нужен
✅ Future-proof solution

❌ Smaller community
❌ More complex architecture
❌ Less tooling ecosystem
❌ Newer platform (less mature)
```

---

### **💰 Total Cost of Ownership (TCO)**

#### **Стоимостная модель**
```
┌─────────────────────────────────────────────────────────────┐
│                      TCO COMPONENTS                        │
├─────────────────────────────────────────────────────────────┤
│  Infrastructure Costs:                                     │
│  ├─ Compute (CPU, Memory)                                  │
│  ├─ Storage (Disk space, IOPS)                             │
│  ├─ Network (Bandwidth, Transfer)                          │
│  └─ Cloud provider fees                                    │
│                                                             │
│  Operational Costs:                                        │
│  ├─ DevOps engineer time                                   │
│  ├─ Monitoring и alerting                                  │
│  ├─ Backup и disaster recovery                             │
│  └─ Security compliance                                    │
│                                                             │
│  Development Costs:                                        │
│  ├─ Learning curve                                         │
│  ├─ Integration time                                       │
│  ├─ Testing и debugging                                    │
│  └─ Maintenance и updates                                  │
└─────────────────────────────────────────────────────────────┘
```

#### **Cost Comparison (примерные цифры для 1M msgs/day):**

| Broker | Infrastructure | Operations | Development | Total/Month |
|--------|----------------|------------|-------------|-------------|
| **Kafka** | $200 | $500 | $300 | $1000 |
| **RabbitMQ** | $150 | $400 | $200 | $750 |
| **SQS** | $100 | $50 | $100 | $250 |
| **Pulsar** | $250 | $600 | $400 | $1250 |

---

### **🔧 Decision Tree для выбора**

```
START: Выбор Message Broker
    │
    ├─ Throughput > 50K msgs/sec?
    │   ├─ YES ──▶ Multi-tenancy важна?
    │   │           ├─ YES ──▶ PULSAR
    │   │           └─ NO ───▶ KAFKA
    │   └─ NO
    │       │
    │       ├─ AWS ecosystem?
    │       │   ├─ YES ──▶ Managed service?
    │       │   │           ├─ YES ──▶ SQS
    │       │   │           └─ NO ───▶ Self-hosted
    │       │   └─ NO
    │       │       │
    │       │       ├─ Complex routing?
    │       │       │   ├─ YES ──▶ RABBITMQ
    │       │       │   └─ NO
    │       │       │       │
    │       │       │       ├─ Low latency critical?
    │       │       │       │   ├─ YES ──▶ REDIS
    │       │       │       │   └─ NO ───▶ ACTIVEMQ
    │       │       │       │
    │       │       │       └─ Java ecosystem?
    │       │       │           ├─ YES ──▶ ACTIVEMQ
    │       │       │           └─ NO ───▶ RABBITMQ
```

---

## 🎯 **ИТОГИ БЛОКА 2**

### **✅ Что вы изучили:**

1. **Apache Kafka** — лидер для high-throughput event streaming
2. **RabbitMQ** — гибкий брокер с rich routing capabilities  
3. **ActiveMQ/SQS** — enterprise и cloud-managed решения
4. **Redis/Pulsar** — specialized use cases и next-gen платформы
5. **Decision framework** для выбора оптимального решения

### **🛠️ Практические навыки:**

- Установка и настройка различных брокеров
- Создание producers и consumers
- Понимание архитектурных различий
- Ability to make informed decisions

### **📈 Следующие шаги:**

Переходите к **Блоку 3: Практическая разработка**, где вы углубитесь в создание production-ready приложений с выбранными брокерами!

---

## 📚 **Дополнительные ресурсы**

### **Документация:**
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [RabbitMQ Tutorials](https://www.rabbitmq.com/getstarted.html)
- [AWS SQS Developer Guide](https://docs.aws.amazon.com/sqs/)
- [Apache Pulsar Documentation](https://pulsar.apache.org/docs/)

### **Практические лаборатории:**
- Confluent Platform Quickstart
- RabbitMQ Management Plugin
- AWS SQS Console Hands-on
- Pulsar Standalone Mode

### **Бенчмарки:**
- [Kafka vs Pulsar Performance](https://streamnative.io/kafka-vs-pulsar/)
- [RabbitMQ Performance](https://www.rabbitmq.com/blog/2012/04/25/rabbitmq-performance-measurements-part-2/)

---

**Время на изучение блока: 4-5 недель**  
**Следующий блок: Практическая разработка** 🚀