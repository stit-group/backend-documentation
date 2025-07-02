# БЛОК 5: ПРОИЗВОДИТЕЛЬНОСТЬ И МАСШТАБИРОВАНИЕ
## От оптимизации до enterprise-масштабов

---

### 🎯 **ЦЕЛЬ БЛОКА**
Освоить экспертные навыки оптимизации систем обмена сообщениями для обработки **миллионов сообщений в секунду** при минимальной задержке.

**Продолжительность:** 4-5 недель  
**Уровень сложности:** ⭐⭐⭐⭐⭐ (Экспертный)

---

## 📊 **ГЛАВА 5.1: МОНИТОРИНГ И МЕТРИКИ**
### Время изучения: 1 неделя

---

#### 🔍 **Key Performance Indicators (KPI)**

```
┌─────────────────────────────────────────────────────────────┐
│                    KAFKA METRICS HIERARCHY                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌───────────────┐    ┌─────────────────┐    ┌────────────┐ │
│  │   BROKER      │    │    PRODUCER     │    │  CONSUMER  │ │
│  │   METRICS     │    │    METRICS      │    │  METRICS   │ │
│  └───────────────┘    └─────────────────┘    └────────────┘ │
│         │                       │                     │     │
│    ┌────▼────┐              ┌───▼────┐           ┌────▼───┐ │
│    │Messages │              │Throughput│           │Lag     │ │
│    │/sec     │              │(msg/sec) │           │(ms)    │ │
│    └─────────┘              └─────────┘           └────────┘ │
│    ┌─────────┐              ┌─────────┐           ┌────────┐ │
│    │CPU/Memory│              │Latency  │           │Offset  │ │
│    │Usage    │              │(ms)     │           │Commit  │ │
│    └─────────┘              └─────────┘           └────────┘ │
│    ┌─────────┐              ┌─────────┐           ┌────────┐ │
│    │Disk I/O │              │Error    │           │Rebalance│ │
│    │(MB/s)   │              │Rate     │           │Time    │ │
│    └─────────┘              └─────────┘           └────────┘ │
└─────────────────────────────────────────────────────────────┘
```

#### 📈 **Критические метрики для мониторинга**

**Broker-уровень:**
```yaml
Critical_Metrics:
  Performance:
    - Messages_per_second: >100,000
    - Bytes_per_second: >100MB
    - Request_latency: <50ms
    - Network_utilization: <80%
  
  Resource_Usage:
    - CPU_utilization: <70%
    - Memory_usage: <80% 
    - Disk_usage: <85%
    - File_descriptors: <80% of limit
  
  Reliability:
    - Under_replicated_partitions: 0
    - Offline_partitions: 0
    - Leader_election_rate: <1/hour
    - ISR_shrink_rate: <1/hour
```

---

#### 🔧 **Настройка Prometheus + Grafana**

**1. Kafka JMX Exporter конфигурация:**
```yaml
# kafka-jmx-exporter.yml
rules:
  # Broker metrics
  - pattern: kafka.server<type=BrokerTopicMetrics, name=MessagesInPerSec><>Count
    name: kafka_server_messages_in_total
    type: COUNTER
    
  - pattern: kafka.server<type=BrokerTopicMetrics, name=BytesInPerSec><>Count  
    name: kafka_server_bytes_in_total
    type: COUNTER
    
  - pattern: kafka.network<type=RequestMetrics, name=RequestsPerSec, request=(.+)><>Count
    name: kafka_network_requests_total
    labels:
      request: "$1"
    type: COUNTER
```

**2. Docker Compose для мониторинга стека:**
```yaml
version: '3.8'
services:
  kafka:
    image: confluentinc/cp-kafka:latest
    environment:
      KAFKA_JMX_PORT: 9999
      KAFKA_JMX_HOSTNAME: localhost
    ports:
      - "9999:9999"
      
  jmx-exporter:
    image: sscaling/jmx-prometheus-exporter
    ports:
      - "8080:8080"
    volumes:
      - ./kafka-jmx-exporter.yml:/opt/jmx_exporter/config.yml
      
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
```

---

#### 📊 **Dashboard дизайн лучших практик**

```
┌─────────────────────────────────────────────────────────────┐
│                   KAFKA CLUSTER OVERVIEW                    │
├─────────────────────────────────────────────────────────────┤
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────┐ │
│ │ Throughput  │ │   Latency   │ │ Error Rate  │ │ Uptime  │ │
│ │ 2.3M msg/s  │ │    45ms     │ │   0.01%     │ │ 99.9%   │ │
│ └─────────────┘ └─────────────┘ └─────────────┘ └─────────┘ │
├─────────────────────────────────────────────────────────────┤
│                     BROKER HEALTH                          │
│ ┌──────────────────────────────────────────────────────────┐ │
│ │        Broker-1    Broker-2    Broker-3                 │ │
│ │ CPU:     65%        72%         68%                     │ │
│ │ Memory:  78%        82%         75%                     │ │
│ │ Disk:    45%        52%         48%                     │ │
│ │ Status: [●GREEN]   [●GREEN]    [●GREEN]                 │ │
│ └──────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│                    CONSUMER LAG                            │
│ ┌──────────────────────────────────────────────────────────┐ │
│ │ orders-service:     ▁▁▁▃▃▃▇▇▇ 1.2K messages             │ │
│ │ payment-service:    ▁▁▁▁▁▁▃▃▃ 340 messages              │ │
│ │ analytics-service:  ▁▁▁▁▁▁▁▁▁ 0 messages                │ │
│ └──────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

**Пример Grafana запроса:**
```promql
# Throughput по топикам
rate(kafka_server_messages_in_total[5m]) * 60

# Consumer lag
kafka_consumer_lag_sum by (consumergroup, topic)

# 99th percentile latency
histogram_quantile(0.99, 
  rate(kafka_network_request_time_99th_percentile[5m])
)
```

---

#### 🚨 **Alerting стратегии**

**Критические алерты:**
```yaml
alerts:
  - alert: KafkaBrokerDown
    expr: up{job="kafka"} == 0
    for: 30s
    labels:
      severity: critical
    annotations:
      summary: "Kafka broker {{ $labels.instance }} is down"
      
  - alert: HighConsumerLag
    expr: kafka_consumer_lag > 10000
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "Consumer lag is high: {{ $value }} messages"
      
  - alert: DiskSpaceUsage
    expr: (1 - node_filesystem_avail_bytes / node_filesystem_size_bytes) > 0.85
    for: 1m
    labels:
      severity: critical
```

---

## ⚖️ **ГЛАВА 5.2: ПАРТИЦИРОВАНИЕ И РАСПРЕДЕЛЕНИЕ НАГРУЗКИ**
### Время изучения: 1.5 недели

---

#### 🔄 **Стратегии Partition Key**

```
┌─────────────────────────────────────────────────────────────┐
│              PARTITION KEY STRATEGIES                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐    │
│  │   Random    │     │ Round Robin │     │   Hashed    │    │
│  │             │     │             │     │             │    │
│  │ No Key      │────▶│ Sequential  │────▶│ Consistent  │    │
│  │ Distribution│     │ Distribution│     │ Distribution│    │
│  └─────────────┘     └─────────────┘     └─────────────┘    │
│        │                     │                     │        │
│        ▼                     ▼                     ▼        │
│  [●●●●●●●●]           [●●●○○○○○]           [●●○●●○●○]        │
│  Even Load            Uneven Load         Predictable      │
│  No Ordering          Some Ordering       Strong Ordering  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### 🎯 **Оптимальные стратегии по use-case**

**E-commerce Orders:**
```java
// ❌ Плохо: может создать hot partitions
String key = "order"; // все в одну партицию

// ✅ Хорошо: равномерное распределение
String key = customerId; // распределение по клиентам

// ✅ Отлично: учитывает locality  
String key = customerId + "_" + region; // + региональность
```

**Financial Transactions:**
```java
// ✅ Для FIFO по аккаунту
String key = accountId;

// ✅ Для глобального FIFO
String key = null; // single partition (осторожно с throughput!)

// ✅ Для daily settlement
String key = accountId + "_" + dateStr;
```

---

#### 🔥 **Решение проблемы Hot Partitions**

**Диагностика:**
```bash
# Анализ распределения по партициям
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --describe --group analytics-group

# Просмотр размеров партиций
kafka-log-dirs.sh --bootstrap-server localhost:9092 \
  --topic-list orders --describe
```

**Визуализация hot partition:**
```
┌─────────────────────────────────────────────────────────────┐
│                  PARTITION LOAD DISTRIBUTION                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Partition 0: ████████████████████ 89% (HOT!) 🔥            │
│ Partition 1: ███ 12%                                       │
│ Partition 2: ██ 8%                                         │
│ Partition 3: ██ 9%                                         │
│ Partition 4: ███ 11%                                       │
│ Partition 5: ██ 7%                                         │
│                                                             │
│ 🚨 Problem: Single large customer dominates partition 0     │
│ 💡 Solution: Use composite key or custom partitioner        │
└─────────────────────────────────────────────────────────────┘
```

**Custom Partitioner решение:**
```java
public class BalancedPartitioner implements Partitioner {
    private final AtomicInteger counter = new AtomicInteger(0);
    
    @Override
    public int partition(String topic, Object key, byte[] keyBytes,
                        Object value, byte[] valueBytes, Cluster cluster) {
        
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();
        
        if (key == null) {
            return counter.getAndIncrement() % numPartitions;
        }
        
        // Для known hot keys используем round-robin
        if (isHotKey(key.toString())) {
            return counter.getAndIncrement() % numPartitions;
        }
        
        // Стандартный hash для остальных
        return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
    }
    
    private boolean isHotKey(String key) {
        // Логика определения hot keys
        return KNOWN_HOT_CUSTOMERS.contains(key);
    }
}
```

---

#### ⚖️ **Load Balancing алгоритмы**

**Consumer Group балансировка:**
```
┌─────────────────────────────────────────────────────────────┐
│                CONSUMER GROUP REBALANCING                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ BEFORE REBALANCING:                                         │
│ Consumer-1: [P0, P1, P2, P3, P4, P5] ← Overloaded          │
│ Consumer-2: [] ← Idle                                       │
│ Consumer-3: [] ← Idle                                       │
│                                                             │
│ ────────────────── REBALANCE ──────────────────             │
│                                                             │
│ AFTER REBALANCING:                                          │
│ Consumer-1: [P0, P1] ← Balanced                            │
│ Consumer-2: [P2, P3] ← Balanced                            │  
│ Consumer-3: [P4, P5] ← Balanced                            │
│                                                             │
│ ✅ Improved throughput: 3x                                  │
│ ✅ Reduced latency: ~70%                                    │
└─────────────────────────────────────────────────────────────┘
```

**Sticky Assignment стратегия:**
```java
// Consumer configuration для минимизации rebalancing
Properties props = new Properties();
props.put("partition.assignment.strategy", 
    "org.apache.kafka.clients.consumer.StickyAssignor");
props.put("session.timeout.ms", "30000");
props.put("heartbeat.interval.ms", "10000");
props.put("max.poll.interval.ms", "300000");
```

---

## 🚀 **ГЛАВА 5.3: THROUGHPUT И LATENCY ОПТИМИЗАЦИЯ**
### Время изучения: 1.5 недели

---

#### ⚡ **Batch Processing vs Real-time Trade-offs**

```
┌─────────────────────────────────────────────────────────────┐
│              BATCH SIZE IMPACT ON PERFORMANCE              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Throughput (msg/sec)    Latency (ms)                       │
│     ▲                      ▲                               │
│ 1M  │ ●                100  │                               │
│     │   ●●               80  │   ●                           │
│ 800K│     ●●●            60  │     ●●                        │
│     │        ●●●         40  │       ●●●                     │
│ 600K│           ●●●      20  │          ●●●                  │
│     │              ●●●    0  │─────────────●●●──────▶        │
│ 400K└─────────────────●──     1   16   64  256  1024        │
│     1   16   64  256  1024          Batch Size              │
│           Batch Size                                        │
│                                                             │
│ 💡 Sweet Spot: batch.size = 64-256 для большинства случаев  │
└─────────────────────────────────────────────────────────────┘
```

#### 🔧 **Producer Performance Tuning**

**Высокий Throughput конфигурация:**
```java
Properties props = new Properties();

// ===== THROUGHPUT OPTIMIZATION =====
props.put("batch.size", 65536);           // 64KB batches
props.put("linger.ms", 10);              // Wait 10ms for batch
props.put("compression.type", "lz4");     // Fast compression
props.put("buffer.memory", 67108864);     // 64MB buffer
props.put("max.in.flight.requests.per.connection", 5);

// ===== NETWORK OPTIMIZATION =====
props.put("send.buffer.bytes", 131072);   // 128KB
props.put("receive.buffer.bytes", 65536); // 64KB

// ===== PARALLELISM =====
props.put("max.request.size", 1048576);   // 1MB max message
```

**Низкая Latency конфигурация:**
```java
Properties props = new Properties();

// ===== LATENCY OPTIMIZATION =====
props.put("batch.size", 1);              // No batching
props.put("linger.ms", 0);               // Send immediately
props.put("compression.type", "none");    // No compression overhead
props.put("acks", "1");                  // Don't wait for all replicas

// ===== QUICK FAILURES =====
props.put("request.timeout.ms", 1000);   // Fail fast
props.put("delivery.timeout.ms", 5000);  // Quick timeout
```

---

#### 🎛️ **Consumer Performance Tuning**

**Конфигурация для максимального throughput:**
```java
Properties props = new Properties();

// ===== FETCH OPTIMIZATION =====
props.put("fetch.min.bytes", 50000);     // Wait for 50KB
props.put("fetch.max.wait.ms", 500);     // Or 500ms max
props.put("max.partition.fetch.bytes", 1048576); // 1MB per partition
props.put("max.poll.records", 5000);     // Process 5K records/poll

// ===== MEMORY OPTIMIZATION =====
props.put("receive.buffer.bytes", 262144); // 256KB
props.put("send.buffer.bytes", 131072);    // 128KB

// ===== SESSION MANAGEMENT =====
props.put("session.timeout.ms", 30000);   // 30s session
props.put("heartbeat.interval.ms", 10000); // 10s heartbeat
```

**Обработка в несколько потоков:**
```java
public class MultiThreadConsumer {
    private final ExecutorService executor;
    private final int numThreads;
    
    public void start() {
        for (int i = 0; i < numThreads; i++) {
            executor.submit(() -> {
                try (KafkaConsumer<String, String> consumer = createConsumer()) {
                    while (!Thread.currentThread().isInterrupted()) {
                        ConsumerRecords<String, String> records = 
                            consumer.poll(Duration.ofMillis(1000));
                        
                        // Параллельная обработка внутри batch
                        records.forEach(record -> 
                            processAsync(record));
                    }
                }
            });
        }
    }
    
    private void processAsync(ConsumerRecord<String, String> record) {
        CompletableFuture.supplyAsync(() -> {
            // Бизнес логика
            return processMessage(record.value());
        }).exceptionally(ex -> {
            log.error("Failed to process message", ex);
            return null;
        });
    }
}
```

---

#### 📊 **Benchmark результаты**

**Сравнение различных конфигураций:**
```
┌─────────────────────────────────────────────────────────────┐
│                PERFORMANCE BENCHMARK RESULTS               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Configuration        Throughput    Latency     CPU Usage   │
│ ─────────────────────────────────────────────────────────  │
│ Default              100K msg/s    45ms        45%         │
│ High Throughput      850K msg/s    95ms        78%         │
│ Low Latency          320K msg/s    8ms         65%         │
│ Balanced             450K msg/s    25ms        58%         │
│ Compressed (LZ4)     720K msg/s    52ms        71%         │
│ Compressed (GZIP)    580K msg/s    78ms        82%         │
│                                                             │
│ 🏆 Winner by use-case:                                      │
│ • Real-time trading: Low Latency config                    │
│ • Analytics pipeline: High Throughput config               │
│ • General purpose: Balanced config                         │
└─────────────────────────────────────────────────────────────┘
```

---

#### 🔍 **Memory Management оптимизация**

**JVM tuning для Kafka:**
```bash
# Broker JVM settings
export KAFKA_HEAP_OPTS="-Xmx6g -Xms6g"
export KAFKA_JVM_PERFORMANCE_OPTS="
  -server 
  -XX:+UseG1GC 
  -XX:MaxGCPauseMillis=20 
  -XX:+DisableExplicitGC 
  -XX:+UseStringDeduplication
  -Djava.awt.headless=true"

# OS-level optimization  
echo 'vm.swappiness=1' >> /etc/sysctl.conf
echo 'vm.dirty_ratio=80' >> /etc/sysctl.conf
echo 'vm.dirty_background_ratio=5' >> /etc/sysctl.conf
```

**Page Cache оптимизация:**
```
┌─────────────────────────────────────────────────────────────┐
│                  KAFKA MEMORY ARCHITECTURE                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ ┌─────────────────── RAM (32GB) ────────────────────┐       │
│ │                                                   │       │
│ │ ┌────────────┐  ┌─────────────────────────────────┐ │     │
│ │ │    JVM     │  │         OS Page Cache           │ │     │
│ │ │   Heap     │  │                                 │ │     │
│ │ │   (6GB)    │  │    Kafka Log Segments          │ │     │
│ │ │            │  │         (24GB)                  │ │     │
│ │ │  ┌─────┐   │  │                                 │ │     │
│ │ │  │Kafka│   │  │ ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┐ │ │     │
│ │ │  │Objs │   │  │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │     │
│ │ │  └─────┘   │  │ └─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┘ │ │     │
│ │ └────────────┘  └─────────────────────────────────┘ │     │
│ └───────────────────────────────────────────────────────┘   │
│                           │                                 │
│ 💡 Key: Kafka leverages OS page cache for zero-copy reads  │
│ 🚀 Result: Reads serve directly from memory, not disk      │
└─────────────────────────────────────────────────────────────┘
```

---

## 📈 **ГЛАВА 5.4: ГОРИЗОНТАЛЬНОЕ МАСШТАБИРОВАНИЕ**
### Время изучения: 1 неделя

---

#### 🏗️ **Cluster Expansion стратегии**

**Поэтапное расширение кластера:**
```
┌─────────────────────────────────────────────────────────────┐
│                  CLUSTER SCALING TIMELINE                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Phase 1: Initial Setup (3 brokers)                         │
│ ┌─────┐    ┌─────┐    ┌─────┐                              │
│ │ B-1 │────│ B-2 │────│ B-3 │                              │
│ └─────┘    └─────┘    └─────┘                              │
│   RF=3       RF=3       RF=3                               │
│                                                             │
│ Phase 2: Load Growth → Add Brokers (6 brokers)             │
│ ┌─────┐    ┌─────┐    ┌─────┐                              │
│ │ B-1 │────│ B-2 │────│ B-3 │                              │
│ └─────┘    └─────┘    └─────┘                              │
│    │          │          │                                 │
│ ┌─────┐    ┌─────┐    ┌─────┐                              │
│ │ B-4 │────│ B-5 │────│ B-6 │                              │
│ └─────┘    └─────┘    └─────┘                              │
│                                                             │
│ Phase 3: Partition Reassignment                            │
│ [████████████████████████████] 100% Complete               │
│                                                             │
│ ✅ Capacity: 2x                                             │
│ ✅ Fault tolerance: Improved                                │
│ ✅ Load distribution: Optimized                             │
└─────────────────────────────────────────────────────────────┘
```

**Пошаговый процесс добавления брокеров:**

```bash
# 1. Добавить новые brokers в кластер
cat > server-4.properties << EOF
broker.id=4
listeners=PLAINTEXT://kafka-4:9092
log.dirs=/var/kafka-logs-4
zookeeper.connect=zk-1:2181,zk-2:2181,zk-3:2181
EOF

# 2. Запустить новый broker
kafka-server-start.sh server-4.properties

# 3. Создать план реассignмента
cat > topics-to-move.json << EOF
{
  "topics": [
    {"topic": "orders"},
    {"topic": "payments"}, 
    {"topic": "analytics"}
  ],
  "version": 1
}
EOF

# 4. Сгенерировать план
kafka-reassign-partitions.sh --bootstrap-server localhost:9092 \
  --topics-to-move-json-file topics-to-move.json \
  --broker-list "1,2,3,4,5,6" \
  --generate

# 5. Выполнить реассignмент
kafka-reassign-partitions.sh --bootstrap-server localhost:9092 \
  --reassignment-json-file reassignment.json \
  --execute

# 6. Проверить статус
kafka-reassign-partitions.sh --bootstrap-server localhost:9092 \
  --reassignment-json-file reassignment.json \
  --verify
```

---

#### 🌍 **Multi-region Deployment**

**Cross-datacenter репликация:**
```
┌─────────────────────────────────────────────────────────────┐
│              MULTI-REGION KAFKA ARCHITECTURE               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│     US-EAST-1                           EU-WEST-1          │
│ ┌─────────────────┐                 ┌─────────────────┐     │
│ │   DC-Primary    │◄────────────────┤   DC-Secondary  │     │
│ │                 │  MirrorMaker 2.0│                 │     │
│ │ ┌─────┐ ┌─────┐ │                 │ ┌─────┐ ┌─────┐ │     │
│ │ │ B-1 │ │ B-2 │ │                 │ │ B-4 │ │ B-5 │ │     │
│ │ └─────┘ └─────┘ │                 │ └─────┘ └─────┘ │     │
│ │ ┌─────┐ ┌─────┐ │                 │ ┌─────┐ ┌─────┐ │     │
│ │ │ B-3 │ │ZK-1 │ │                 │ │ B-6 │ │ZK-2 │ │     │
│ │ └─────┘ └─────┘ │                 │ └─────┘ └─────┘ │     │
│ └─────────────────┘                 └─────────────────┘     │
│         │                                   │               │
│    ┌────▼────┐                         ┌────▼────┐          │
│    │Local    │                         │Local    │          │
│    │Apps     │                         │Apps     │          │
│    └─────────┘                         └─────────┘          │
│                                                             │
│ 🔄 Replication Lag: <100ms                                 │
│ 🌐 Network: Dedicated line 10Gbps                          │
│ 📊 Consistency: Eventual (async replication)               │
└─────────────────────────────────────────────────────────────┘
```

**MirrorMaker 2.0 конфигурация:**
```properties
# mm2.properties
clusters = primary, secondary
primary.bootstrap.servers = us-east-1-kafka:9092
secondary.bootstrap.servers = eu-west-1-kafka:9092

# Replication flows
primary->secondary.enabled = true
secondary->primary.enabled = false

# Topics to replicate
primary->secondary.topics = orders.*, payments.*, analytics.*

# Replication settings
replication.factor = 3
sync.topic.acls.enabled = true
emit.checkpoints.interval.seconds = 5
emit.heartbeats.interval.seconds = 1

# Performance tuning
tasks.max = 10
primary->secondary.producer.batch.size = 65536
primary->secondary.producer.linger.ms = 10
```

---

#### 📊 **Capacity Planning**

**Расчет требований к ресурсам:**

```python
# Kafka Capacity Calculator
class KafkaCapacityPlanner:
    def __init__(self):
        self.safety_factor = 1.5  # 50% запас
        
    def calculate_requirements(self, 
                             messages_per_second: int,
                             avg_message_size_bytes: int,
                             retention_days: int,
                             replication_factor: int = 3):
        
        # Throughput calculations
        bytes_per_second = messages_per_second * avg_message_size_bytes
        bytes_per_day = bytes_per_second * 86400
        
        # Storage calculations  
        raw_storage_gb = (bytes_per_day * retention_days) / (1024**3)
        total_storage_gb = raw_storage_gb * replication_factor * self.safety_factor
        
        # Network calculations
        network_mbps = (bytes_per_second * 8) / (1024**2)
        
        # Memory calculations (page cache)
        memory_gb = min(total_storage_gb * 0.1, 64)  # 10% of data or 64GB max
        
        return {
            'throughput_mbps': round(network_mbps, 2),
            'storage_gb': round(total_storage_gb, 2),
            'memory_gb': round(memory_gb, 2),
            'recommended_brokers': max(3, int(total_storage_gb / 1000))
        }

# Пример расчета
planner = KafkaCapacityPlanner()
requirements = planner.calculate_requirements(
    messages_per_second=1_000_000,
    avg_message_size_bytes=1024,
    retention_days=7,
    replication_factor=3
)
```

**Результат планирования:**
```
┌─────────────────────────────────────────────────────────────┐
│                  CAPACITY PLANNING RESULTS                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Input Parameters:                                           │
│ • Messages/sec: 1,000,000                                   │
│ • Message size: 1KB                                         │
│ • Retention: 7 days                                         │
│ • Replication: 3x                                           │
│                                                             │
│ ──────────────────────────────────────────────────────────  │
│                                                             │
│ Resource Requirements:                                      │
│ • Network: 7,813 Mbps (10 Gbps recommended)                │
│ • Storage: 31.5 TB (35 TB recommended)                     │
│ • Memory: 32 GB per broker                                  │
│ • Brokers: 6 minimum (9 recommended)                       │
│                                                             │
│ ──────────────────────────────────────────────────────────  │
│                                                             │
│ Scaling Timeline:                                           │
│ Phase 1 (0-6 months):   6 brokers                          │
│ Phase 2 (6-12 months):  9 brokers                          │
│ Phase 3 (12+ months):   12 brokers                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎯 **ПРАКТИЧЕСКИЕ ЗАДАНИЯ**

### **Задание 1: Настройка комплексного мониторинга**
**Время:** 2 дня

1. Развернуть Kafka кластер из 3 брокеров
2. Настроить Prometheus + Grafana стек
3. Создать comprehensive dashboard
4. Настроить критические алерты
5. Провести load testing и анализ метрик

**Критерии успеха:**
- ✅ Dashboard показывает все ключевые метрики
- ✅ Алерты срабатывают корректно  
- ✅ Отчет с анализом bottlenecks

---

### **Задание 2: Оптимизация hot partition**
**Время:** 1 день

1. Создать синтетический hot partition сценарий
2. Реализовать custom partitioner
3. Провести A/B тестирование
4. Измерить улучшения производительности

**Ожидаемый результат:**
- 📈 Равномерное распределение нагрузки
- ⚡ Улучшение throughput на 40%+

---

### **Задание 3: Multi-region setup**
**Время:** 2 дня

1. Развернуть кластеры в двух "регионах" (Docker networks)
2. Настроить MirrorMaker 2.0 репликацию
3. Провести failover тестирование
4. Измерить replication lag

**Deliverables:**
- 📋 Документация по disaster recovery
- 📊 Метрики репликации
- 🔄 Процедуры failover

---

## ✅ **РЕЗУЛЬТАТЫ БЛОКА**

После успешного завершения блока вы будете уметь:

### **Экспертные навыки:**
- 🔍 Проектировать comprehensive мониторинг для enterprise
- ⚖️ Решать проблемы неравномерного распределения нагрузки  
- 🚀 Настраивать системы для миллионов сообщений/сек
- 🌐 Проектировать multi-region deployment
- 📊 Проводить capacity planning для любых нагрузок

### **Измеримые улучшения:**
- **Throughput:** До 10x improvement через tuning
- **Latency:** Снижение с 100ms до <10ms для critical путей
- **Availability:** 99.99% uptime через proper scaling
- **Cost:** 30-50% экономия через оптимизацию ресурсов

---

## 📚 **ДОПОЛНИТЕЛЬНЫЕ РЕСУРСЫ**

### **Книги и статьи:**
- "Kafka: The Definitive Guide" - Chapter 7: Performance
- "Designing Data-Intensive Applications" - Chapter 11
- Confluent Performance Tuning Guide

### **Tools для практики:**
- **kafka-perf-test.sh** - нагрузочное тестирование
- **Kafka Manager** - UI для управления
- **Cruise Control** - автоматическая балансировка
- **Burrow** - мониторинг consumer lag

### **Community ресурсы:**
- Confluent Community Slack #performance
- Apache Kafka Users mailing list
- KafkaJS Performance Guide
- Kafka Streams Performance Tuning

---

**🚀 Следующий блок:** [Блок 6: Безопасность и надежность →]()