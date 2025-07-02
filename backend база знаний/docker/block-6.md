# 📊 Блок 6: Мониторинг и production

*Превращаем контейнеры в наблюдаемую и оптимизированную production систему*

---

## 📋 Обзор блока

В этом блоке мы изучим, как превратить ваши контейнеризованные приложения в полноценные production-ready системы. Вы научитесь видеть "внутренности" ваших контейнеров, оптимизировать их производительность и готовить к реальным нагрузкам.

**Что вы изучите:**
- Централизованное логирование и анализ логов
- Системы мониторинга метрик и алертинг
- Профилирование и оптимизацию производительности
- Production-ready конфигурации
- Стратегии backup и disaster recovery

---

## 📝 Глава 6.1: Логирование и мониторинг

### Концепция наблюдаемости (Observability)

#### Три столпа наблюдаемости

```
Observability Triangle:
══════════════════════

                    ┌─────────────┐
                    │   METRICS   │
                    │ (Что?)      │
                    │ • CPU/RAM   │
                    │ • Requests  │
                    │ • Errors    │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │                         │
              ▼                         ▼
    ┌─────────────┐                ┌─────────────┐
    │    LOGS     │                │   TRACES    │
    │  (Почему?)  │                │   (Где?)    │
    │ • Events    │                │ • Request   │
    │ • Errors    │                │   Flow      │
    │ • Debug     │                │ • Latency   │
    └─────────────┘                └─────────────┘

Вместе они дают полную картину состояния системы!
```

#### Проблемы традиционного мониторинга контейнеров

```
Container Monitoring Challenges:
═══════════════════════════════

Традиционный мониторинг:
┌─────────────────────────────────────┐
│ Физический сервер                   │
│ ├── Процессы                        │
│ ├── Файлы логов (/var/log/)         │
│ ├── Системные метрики               │
│ └── Статичная конфигурация          │
└─────────────────────────────────────┘
        Простой мониторинг

Контейнерная среда:
┌─────────────────────────────────────┐
│ Host                                │
│ ├── Container 1 (появляется)        │ ← Эфемерные
│ ├── Container 2 (исчезает)          │ ← Динамичные
│ ├── Container 3 (масштабируется)    │ ← Множественные
│ └── Container N (мигрирует)         │ ← Распределенные
└─────────────────────────────────────┘
     Сложный мониторинг требуется!

Проблемы:
─────────
• Логи исчезают с контейнером
• Метрики нужно агрегировать
• Сетевая связность сложная
• Service discovery необходим
• Корреляция между сервисами
```

### ELK Stack для централизованного логирования

#### Архитектура ELK Stack

```
ELK Stack Architecture:
══════════════════════

Application Containers:
┌─────────┐ ┌─────────┐ ┌─────────┐
│  App 1  │ │  App 2  │ │  App N  │
│ (logs)  │ │ (logs)  │ │ (logs)  │
└────┬────┘ └────┬────┘ └────┬────┘
     │           │           │
     ▼           ▼           ▼
┌─────────────────────────────────┐
│         Log Drivers             │
│ • json-file                     │
│ • journald                      │
│ • syslog                        │
│ • fluentd                       │
└─────────────┬───────────────────┘
              │
              ▼
┌─────────────────────────────────┐
│         Logstash                │
│ • Parse & Transform             │
│ • Filter & Enrich               │
│ • Multiple Input Sources        │
│ • Multiple Output Targets       │
└─────────────┬───────────────────┘
              │
              ▼
┌─────────────────────────────────┐
│      Elasticsearch             │
│ • Index & Store Logs            │
│ • Full-text Search              │
│ • Distributed Storage           │
│ • Real-time Analytics           │
└─────────────┬───────────────────┘
              │
              ▼
┌─────────────────────────────────┐
│         Kibana                  │
│ • Visualization                 │
│ • Dashboards                    │
│ • Alerting                      │
│ • Log Analysis                  │
└─────────────────────────────────┘
```

#### Логирование стратегии для контейнеров

```
Container Logging Strategies:
════════════════════════════

1. Application-Level Logging:
┌─────────────────────────────────┐
│ FROM node:16                    │
│ COPY app.js .                   │
│ # App пишет в stdout/stderr     │
│ CMD ["node", "app.js"]          │
└─────────────────────────────────┘
              │
              ▼ Docker захватывает
┌─────────────────────────────────┐
│ /var/lib/docker/containers/     │
│ └── <container-id>/             │
│     └── <container-id>-json.log │
└─────────────────────────────────┘

2. Sidecar Pattern:
┌─────────────────────────────────┐
│ ┌─────────┐ ┌─────────────────┐ │
│ │   App   │ │   Log Forwarder │ │
│ │Container│→│   (Fluentd)     │→│ ELK
│ └─────────┘ └─────────────────┘ │
│     Shared Volume               │
└─────────────────────────────────┘

3. Host-Level Agents:
┌─────────────────────────────────┐
│ Host Machine                    │
│ ├── App Containers              │
│ └── Fluentd/Filebeat Agent ─────→ ELK
│     (монтирует Docker socket)   │
└─────────────────────────────────┘

4. Direct Shipping:
┌─────────────────────────────────┐
│ Application                     │
│ └── Logging Library ────────────→ ELK
│     (напрямую в Elasticsearch)  │
└─────────────────────────────────┘
```

#### Структурированное логирование

```
Log Structure Evolution:
═══════════════════════

Unstructured Logs (плохо):
──────────────────────────
2023-07-02 10:15:30 User john_doe logged in from 192.168.1.100
2023-07-02 10:16:45 ERROR: Database connection failed
2023-07-02 10:17:12 Payment processed: $129.99 for order #12345

Structured Logs (хорошо):
────────────────────────
{
  "timestamp": "2023-07-02T10:15:30Z",
  "level": "INFO",
  "service": "auth-service",
  "event": "user_login",
  "user_id": "john_doe",
  "ip_address": "192.168.1.100",
  "session_id": "abc123",
  "trace_id": "xyz789"
}

{
  "timestamp": "2023-07-02T10:16:45Z",
  "level": "ERROR",
  "service": "database-service",
  "event": "connection_failed",
  "error_code": "CONN_TIMEOUT",
  "retry_count": 3,
  "trace_id": "xyz789"
}

Benefits of Structured Logs:
────────────────────────────
✅ Легкий поиск и фильтрация
✅ Автоматический парсинг
✅ Корреляция между сервисами
✅ Метрики из логов
✅ Алертинг на основе полей
```

### Prometheus и Grafana для метрик

#### Prometheus архитектура

```
Prometheus Ecosystem:
════════════════════

    Target Applications
    ┌─────────┐ ┌─────────┐ ┌─────────┐
    │  App 1  │ │  App 2  │ │  App N  │
    │:8080/   │ │:8081/   │ │:8082/   │
    │metrics  │ │metrics  │ │metrics  │
    └────┬────┘ └────┬────┘ └────┬────┘
         │           │           │
         ▼           ▼           ▼
    ┌─────────────────────────────────┐
    │        Prometheus Server        │
    │                                 │
    │ ┌─────────────┐ ┌─────────────┐ │
    │ │ Scraper     │ │ Time Series │ │
    │ │ (Pull)      │ │ Database    │ │
    │ └─────────────┘ └─────────────┘ │
    │ ┌─────────────┐ ┌─────────────┐ │
    │ │ Rules Engine│ │ Alert       │ │
    │ │             │ │ Manager     │ │
    │ └─────────────┘ └─────────────┘ │
    └─────────────┬───────────────────┘
                  │
                  ▼
    ┌─────────────────────────────────┐
    │           Grafana               │
    │ • Visualization                 │
    │ • Dashboards                    │
    │ • Alerting                      │
    │ • User Management               │
    └─────────────────────────────────┘
```

#### Типы метрик в Prometheus

```
Prometheus Metric Types:
═══════════════════════

1. Counter (Счетчик):
   Только растет (или сбрасывается)
   ┌─────────────────────────────────┐
   │ http_requests_total{method="GET"}│
   │ error_count_total               │
   │ bytes_processed_total           │
   └─────────────────────────────────┘
   
   График:    ↗
            ↗
          ↗
        ↗

2. Gauge (Измеритель):
   Может расти и убывать
   ┌─────────────────────────────────┐
   │ memory_usage_bytes              │
   │ active_connections              │
   │ temperature_celsius             │
   └─────────────────────────────────┘
   
   График:  ↗  ↘
           ↗    ↘
         ↗        ↘
       ↗            ↘

3. Histogram (Гистограмма):
   Распределение значений
   ┌─────────────────────────────────┐
   │ request_duration_seconds_bucket │
   │ {le="0.1"} 100                  │
   │ {le="0.5"} 250                  │
   │ {le="1.0"} 400                  │
   │ {le="+Inf"} 500                 │
   └─────────────────────────────────┘
   
   Buckets: [▓▓▓▓▓▓][▓▓▓][▓▓][▓]

4. Summary (Сводка):
   Квантили и общая статистика
   ┌─────────────────────────────────┐
   │ request_duration_seconds{       │
   │   quantile="0.5"} 0.05          │
   │   quantile="0.95"} 0.2          │
   │   quantile="0.99"} 0.5          │
   │ }                               │
   └─────────────────────────────────┘
```

#### Container-specific метрики

```
Docker Container Metrics:
════════════════════════

System Level Metrics:
┌─────────────────────────────────┐
│ CPU Metrics:                    │
│ ├── container_cpu_usage_seconds │
│ ├── container_cpu_system_seconds│
│ └── container_cpu_user_seconds  │
│                                 │
│ Memory Metrics:                 │
│ ├── container_memory_usage_bytes│
│ ├── container_memory_cache      │
│ ├── container_memory_rss        │
│ └── container_memory_swap       │
│                                 │
│ Network Metrics:                │
│ ├── container_network_receive_  │
│ │   bytes_total                 │
│ ├── container_network_transmit_ │
│ │   bytes_total                 │
│ └── container_network_receive_  │
│     packets_dropped_total       │
│                                 │
│ Disk I/O Metrics:               │
│ ├── container_fs_reads_total    │
│ ├── container_fs_writes_total   │
│ └── container_fs_usage_bytes    │
└─────────────────────────────────┘

Application Level Metrics:
┌─────────────────────────────────┐
│ Business Metrics:               │
│ ├── orders_processed_total      │
│ ├── user_registrations_total    │
│ └── revenue_total               │
│                                 │
│ Performance Metrics:            │
│ ├── request_duration_seconds    │
│ ├── database_query_duration     │
│ └── cache_hit_ratio             │
│                                 │
│ Error Metrics:                  │
│ ├── http_requests_total{        │
│ │   status="500"}               │
│ ├── database_connection_errors  │
│ └── authentication_failures     │
└─────────────────────────────────┘
```

### Service Discovery в контейнерной среде

#### Динамическое обнаружение сервисов

```
Service Discovery Challenge:
═══════════════════════════

Static Configuration (не работает):
┌─────────────────────────────────┐
│ prometheus.yml:                 │
│   - targets:                    │
│     - app1:8080                 │ ← Что если контейнер
│     - app2:8081                 │   перезапустился?
│     - app3:8082                 │ ← Что если масштабирование?
└─────────────────────────────────┘

Dynamic Service Discovery:
┌─────────────────────────────────┐
│ Docker Swarm Mode:              │
│   - targets:                    │
│     - tasks.app-service:8080    │ ← Автоматически находит
│                                 │   все instances
│ Kubernetes:                     │
│   - kubernetes_sd_configs:      │
│     - role: pod                 │ ← Обнаруживает все pods
│                                 │
│ Consul:                         │
│   - consul_sd_configs:          │
│     - server: consul:8500       │ ← Service registry
└─────────────────────────────────┘

Service Registration Flow:
─────────────────────────

Container Start
       ↓
Register in Service Registry
       ↓
Prometheus Discovers Target
       ↓
Starts Scraping Metrics
       ↓
Container Stop
       ↓
Deregister from Service Registry
       ↓
Prometheus Removes Target
```

#### Labeling стратегии

```
Prometheus Labeling Best Practices:
══════════════════════════════════

Хорошие Labels:
┌─────────────────────────────────┐
│ http_requests_total{            │
│   service="user-service",       │ ← Идентифицирует сервис
│   method="GET",                 │ ← HTTP метод
│   status="200",                 │ ← Код ответа
│   endpoint="/api/users",        │ ← Конкретный endpoint
│   version="v2.1.0"              │ ← Версия приложения
│ }                               │
└─────────────────────────────────┘

Плохие Labels (высокая кардинальность):
┌─────────────────────────────────┐
│ http_requests_total{            │
│   user_id="12345",              │ ← Уникальные пользователи
│   request_id="abc-def-123",     │ ← Уникальные запросы
│   timestamp="1625234567"        │ ← Временные метки
│ }                               │
│ ⚠️  Взрывает кардинальность!    │
└─────────────────────────────────┘

Label Hierarchy:
───────────────
Environment: prod/staging/dev
  ├── Cluster: us-east-1/eu-west-1
  │   ├── Service: user-service/order-service
  │   │   ├── Version: v1.0/v2.0
  │   │   │   └── Instance: pod-123/pod-456
  │   │   └── Method: GET/POST/PUT
  │   └── Status: 2xx/4xx/5xx
  └── Component: frontend/backend/database
```

---

## ⚡ Глава 6.2: Оптимизация производительности

### Профилирование контейнеров

#### Ресурсное потребление анализ

```
Container Resource Analysis:
═══════════════════════════

Memory Usage Breakdown:
┌─────────────────────────────────┐
│ Container Memory Layout:        │
│                                 │
│ ┌─────────────────────────────┐ │
│ │     Application Heap        │ │ ← Основная память приложения
│ │        (60-80%)             │ │
│ ├─────────────────────────────┤ │
│ │     System Libraries        │ │ ← Разделяемые библиотеки
│ │        (10-15%)             │ │
│ ├─────────────────────────────┤ │
│ │     OS Buffers/Cache        │ │ ← Файловые кеши
│ │        (5-10%)              │ │
│ ├─────────────────────────────┤ │
│ │     Container Overhead      │ │ ← Docker namespace overhead
│ │        (2-5%)               │ │
│ └─────────────────────────────┘ │
└─────────────────────────────────┘

CPU Usage Patterns:
──────────────────
Idle Application:     [░░░░░░░░░░] 5-10%
Normal Load:          [████░░░░░░] 40%
High Load:            [████████░░] 80%
CPU Throttling:       [██████████] 100% (проблема!)

I/O Patterns:
────────────
Read-heavy:  ↑↑↑↑↑ (высокий read IOPS)
Write-heavy: ↓↓↓↓↓ (высокий write IOPS)
Balanced:    ↑↓↑↓↑ (смешанная нагрузка)
```

#### Performance profiling инструменты

```
Container Profiling Stack:
═════════════════════════

1. Runtime Profilers:
┌─────────────────────────────────┐
│ Application Level:              │
│ ├── Node.js: --inspect         │
│ ├── Java: JProfiler, VisualVM  │
│ ├── Python: cProfile, py-spy   │
│ ├── Go: go tool pprof          │
│ └── .NET: dotMemory, PerfView  │
└─────────────────────────────────┘

2. System Level:
┌─────────────────────────────────┐
│ Container Tools:                │
│ ├── docker stats               │ ← Базовые метрики
│ ├── ctop                       │ ← TUI для контейнеров
│ ├── cadvisor                   │ ← Google cAdvisor
│ └── sysdig                     │ ← Детальная аналитика
│                                │
│ Linux Tools:                   │
│ ├── perf                       │ ← CPU профилирование
│ ├── iotop                      │ ← I/O мониторинг
│ ├── htop                       │ ← Процессы
│ └── nethogs                    │ ← Сетевая активность
└─────────────────────────────────┘

3. Distributed Tracing:
┌─────────────────────────────────┐
│ Request Flow Tracing:           │
│ ├── Jaeger                     │ ← OpenTracing совместимый
│ ├── Zipkin                     │ ← Lightweight трейсинг
│ ├── OpenTelemetry              │ ← Современный стандарт
│ └── AWS X-Ray                  │ ← Облачное решение
└─────────────────────────────────┘
```

#### Memory optimization техники

```
Memory Optimization Strategies:
══════════════════════════════

1. Base Image Optimization:
┌─────────────────────────────────┐
│ ❌ ubuntu:latest    → 72MB       │
│ ✅ ubuntu:20.04     → 72MB       │
│ ✅ python:3.9-slim  → 45MB       │
│ ✅ python:3.9-alpine→ 23MB       │ ← 68% меньше!
│ ✅ distroless       → 12MB       │ ← 83% меньше!
└─────────────────────────────────┘

2. Multi-stage Build Optimization:
┌─────────────────────────────────┐
│ # Build stage                   │
│ FROM node:16 AS builder         │
│ COPY package*.json ./           │
│ RUN npm ci                      │ ← Все зависимости
│ COPY . .                        │
│ RUN npm run build               │
│                                 │
│ # Production stage              │
│ FROM node:16-alpine             │ ← Меньший образ
│ COPY --from=builder /dist ./    │ ← Только артефакты
│ RUN npm ci --only=production    │ ← Только prod deps
│ USER node                       │ ← Безопасность
└─────────────────────────────────┘

3. Runtime Memory Limits:
┌─────────────────────────────────┐
│ # Правильные лимиты             │
│ docker run -m 512m app          │
│                                 │
│ # JVM heap sizing               │
│ ENV JAVA_OPTS="-Xmx400m -Xms400m"│
│                                 │
│ # Node.js memory limit          │
│ ENV NODE_OPTIONS="--max-old-space-size=400"│
└─────────────────────────────────┘

Memory Usage Monitoring:
───────────────────────
Application Memory:    [████████░░] 400MB/512MB (78%)
System Memory:         [██░░░░░░░░] 100MB (RSS)
Cache Memory:          [███░░░░░░░] 150MB (можно освободить)
Available for Growth:  [░░██░░░░░░] 112MB

⚠️  Мониторьте OOMKilled events!
```

### Настройка лимитов ресурсов

#### CPU ограничения

```
CPU Resource Management:
═══════════════════════

CPU Shares vs CPU Limits:
┌─────────────────────────────────┐
│ CPU Shares (относительный вес): │
│ ┌─────────┐ ┌─────────┐        │
│ │Container│ │Container│        │
│ │A (1024) │ │B (512)  │        │
│ └─────────┘ └─────────┘        │
│     ⅔           ⅓              │ ← При нагрузке
│ (66.7%)     (33.3%)            │
└─────────────────────────────────┘

┌─────────────────────────────────┐
│ CPU Limits (абсолютные):        │
│ ┌─────────┐ ┌─────────┐        │
│ │Container│ │Container│        │
│ │A (1.0)  │ │B (0.5)  │        │
│ └─────────┘ └─────────┘        │
│   100%       50%               │ ← Максимум всегда
│   CPU        CPU               │
└─────────────────────────────────┘

CPU Throttling Effects:
──────────────────────
Without Limits:        [████████████] 12 cores
With 1 CPU Limit:      [████░░░░░░░░] 1 core (throttled)
With 0.5 CPU Limit:    [██░░░░░░░░░░] 0.5 core

⚠️  CPU throttling увеличивает latency!
```

#### Memory ограничения и OOM

```
Memory Limit Behavior:
═════════════════════

Memory Usage Timeline:
┌─────────────────────────────────┐
│ Memory Limit: 512MB             │
│                                 │
│ Usage: [████████░░] 400MB ✅    │ ← Нормальная работа
│ Usage: [█████████░] 450MB ✅    │ ← Близко к лимиту
│ Usage: [██████████] 512MB ⚠️    │ ← На лимите
│ Usage: [████████████] 600MB ❌  │ ← OOMKilled!
└─────────────────────────────────┘

OOM Kill Process:
────────────────
1. Container превышает memory limit
2. Kernel OOM killer активируется
3. Процесс с highest OOM score убивается
4. Container перезапускается (если restart policy)
5. Приложение теряет состояние

Memory Management Strategies:
────────────────────────────
┌─────────────────────────────────┐
│ Conservative Approach:          │
│ Request: 256MB                  │ ← Гарантированная память
│ Limit:   512MB                  │ ← Максимум
│ Ratio:   1:2                    │ ← Запас для всплесков
│                                 │
│ Aggressive Approach:            │
│ Request: 400MB                  │ ← Типичное потребление
│ Limit:   450MB                  │ ← Жесткий лимит
│ Ratio:   1:1.125                │ ← Минимальный запас
└─────────────────────────────────┘
```

#### I/O и сетевые оптимизации

```
I/O Performance Optimization:
════════════════════════════

Storage Performance Tiers:
┌─────────────────────────────────┐
│ Type        │ IOPS  │ Latency   │
│─────────────┼───────┼───────────│
│ tmpfs       │∞      │ <1μs      │ ← В памяти
│ NVMe SSD    │100k+  │ 100μs     │ ← Локальный SSD
│ SATA SSD    │20k    │ 200μs     │ ← Стандартный SSD
│ Network SSD │10k    │ 1ms       │ ← EBS, Persistent Disk
│ HDD         │200    │ 10ms      │ ← Традиционные диски
└─────────────────────────────────┘

Volume Types for Different Workloads:
────────────────────────────────────
Database Data:     Persistent Volume (SSD)
Temporary Files:   tmpfs mount
Log Files:         Host bind mount
Config Files:      ConfigMap/Secret
Cache Data:        emptyDir (SSD)

Network Optimization:
────────────────────
┌─────────────────────────────────┐
│ Container Network Performance: │
│                                │
│ Host Network:                  │ ← Максимальная производительность
│ ├── Прямой доступ к host НIC   │   но без изоляции
│ └── Latency: ~1μs             │
│                                │
│ Bridge Network:                │ ← Баланс производительности/изоляции
│ ├── docker0 bridge            │
│ └── Latency: ~10μs            │
│                                │
│ Overlay Network:               │ ← Для multi-host
│ ├── VXLAN encapsulation       │
│ └── Latency: ~50μs            │
└─────────────────────────────────┘

Connection Pooling Impact:
─────────────────────────
Without Pooling:     [━━━━━━━━━━] 1000 connections
With Pooling (10):   [━━░░░░░░░░] 10 connections
Resource Reduction:  90% fewer connections!
```

### Application-level оптимизации

#### Runtime оптимизации по языкам

```
Language-Specific Optimizations:
═══════════════════════════════

Java Applications:
┌─────────────────────────────────┐
│ JVM Tuning:                     │
│ ├── -Xmx400m                    │ ← Heap размер
│ ├── -XX:+UseG1GC               │ ← Garbage collector
│ ├── -XX:MaxGCPauseMillis=200   │ ← GC pause target
│ ├── -XX:+UseContainerSupport   │ ← Container awareness
│ └── -Djava.security.egd=       │ ← Энтропия
│     file:/dev/./urandom        │
│                                │
│ Spring Boot Optimizations:     │
│ ├── Lazy initialization        │
│ ├── Exclude unused autocfg     │
│ └── Native compilation (GraalVM)│
└─────────────────────────────────┘

Node.js Applications:
┌─────────────────────────────────┐
│ V8 Engine Tuning:               │
│ ├── --max-old-space-size=400   │ ← Heap лимит
│ ├── --optimize-for-size        │ ← Меньше памяти
│ ├── --gc-interval=100          │ ← GC интервал
│ └── --trace-gc                 │ ← GC мониторинг
│                                │
│ Application Optimizations:     │
│ ├── Cluster mode               │ ← Multi-process
│ ├── Keep-alive connections     │ ← Connection reuse
│ ├── Response compression       │ ← gzip/deflate
│ └── Static asset caching       │ ← CDN integration
└─────────────────────────────────┘

Python Applications:
┌─────────────────────────────────┐
│ CPython Optimizations:         │
│ ├── PYTHONOPTIMIZE=1           │ ← Bytecode optimization
│ ├── PYTHONDONTWRITEBYTECODE=1  │ ← No .pyc files
│ ├── PYTHONUNBUFFERED=1         │ ← Immediate stdout
│ └── MALLOC_ARENA_MAX=2         │ ← Memory arenas
│                                │
│ Alternative Runtimes:          │
│ ├── PyPy (JIT compilation)     │ ← 2-10x faster
│ ├── Gunicorn workers          │ ← Multi-process
│ └── uvloop (asyncio)          │ ← Faster event loop
└─────────────────────────────────┘
```

---

## 🔧 Глава 6.3: Production-ready настройки

### Health checks и liveness probes

#### Типы health checks

```
Health Check Types:
══════════════════

Docker Native Health Checks:
┌─────────────────────────────────┐
│ HEALTHCHECK --interval=30s \    │
│   --timeout=3s \                │
│   --start-period=5s \           │
│   --retries=3 \                 │
│   CMD curl -f http://localhost/ │
│     || exit 1                   │
└─────────────────────────────────┘

Health Check Levels:
───────────────────
┌─────────────────────────────────┐
│ 1. Shallow Health Check:        │
│    ✅ Процесс работает           │
│    ✅ Порт слушает               │
│    ❌ Не проверяет зависимости   │
│                                 │
│ 2. Deep Health Check:           │
│    ✅ Database connectivity      │
│    ✅ External services         │
│    ✅ Internal state           │
│    ❌ Может быть медленным      │
│                                 │
│ 3. Business Health Check:       │
│    ✅ Критичные функции         │
│    ✅ Data consistency          │
│    ✅ Performance thresholds    │
│    ❌ Комплексная логика        │
└─────────────────────────────────┘

Health Check State Machine:
──────────────────────────
   Starting
      │
      ▼ (start-period)
   Healthy ←──→ Unhealthy
      ▲           │
      │           ▼ (retries exceeded)
      └────── Restarting
```

#### Graceful shutdown

```
Graceful Shutdown Process:
═════════════════════════

Signal Handling Flow:
┌─────────────────────────────────┐
│ 1. SIGTERM received             │ ← Kubernetes/Docker
│    ├── Stop accepting new       │   посылает сигнал
│    │   requests                 │
│    ├── Finish current requests  │
│    └── Close connections        │
│                                 │
│ 2. Grace period (30s default)   │ ← Время на завершение
│    ├── Complete in-flight       │
│    │   operations              │
│    ├── Save state if needed     │
│    └── Cleanup resources        │
│                                 │
│ 3. SIGKILL (if timeout)         │ ← Принудительное завершение
│    └── Process terminated       │
└─────────────────────────────────┘

Implementation Examples:
──────────────────────

Node.js:
┌─────────────────────────────────┐
│ process.on('SIGTERM', () => {   │
│   console.log('Shutting down'); │
│   server.close(() => {          │
│     process.exit(0);            │
│   });                           │
│ });                             │
│                                 │
│ // Graceful timeout             │
│ setTimeout(() => {              │
│   process.exit(1);              │
│ }, 30000);                      │
└─────────────────────────────────┘

Java Spring Boot:
┌─────────────────────────────────┐
│ @PreDestroy                     │
│ public void onShutdown() {      │
│   // Cleanup logic              │
│   connectionPool.close();       │
│   scheduler.shutdown();         │
│ }                               │
│                                 │
│ server.shutdown=graceful        │
│ spring.lifecycle.timeout-       │
│   per-shutdown-phase=30s        │
└─────────────────────────────────┘

Shutdown Timeline:
─────────────────
T+0s:  SIGTERM received
T+1s:  Stop accepting new requests
T+2s:  Begin draining connections
T+15s: Complete critical operations
T+25s: Cleanup resources
T+30s: Process exits gracefully
       (or SIGKILL if hanging)
```

### Secrets management

#### Secrets security model

```
Secrets Management Hierarchy:
════════════════════════════

❌ Worst Practices:
┌─────────────────────────────────┐
│ ENV PASSWORD=secret123          │ ← В Dockerfile
│ docker run -e PASS=secret app   │ ← В истории команд
│ ConfigMap: password: secret     │ ← Plain text в кластере
└─────────────────────────────────┘

✅ Better Practices:
┌─────────────────────────────────┐
│ Docker Secrets:                 │
│ ├── Encrypted at rest           │
│ ├── Encrypted in transit        │
│ ├── Only on manager nodes       │
│ └── Mounted as tmpfs            │
│                                 │
│ Kubernetes Secrets:             │
│ ├── Base64 encoded (not secure!)│
│ ├── RBAC controlled             │
│ ├── Can be encrypted at rest    │
│ └── Namespace isolated          │
└─────────────────────────────────┘

✅ Best Practices:
┌─────────────────────────────────┐
│ External Secret Managers:       │
│ ├── HashiCorp Vault             │
│ ├── AWS Secrets Manager         │
│ ├── Azure Key Vault             │
│ ├── Google Secret Manager       │
│ └── Bank-grade encryption       │
│                                 │
│ Integration Patterns:           │
│ ├── Init containers fetch       │
│ ├── Sidecar containers refresh  │
│ ├── CSI drivers mount           │
│ └── Application SDK integration │
└─────────────────────────────────┘
```

#### Secrets rotation

```
Secret Rotation Strategies:
══════════════════════════

Manual Rotation (не масштабируется):
┌─────────────────────────────────┐
│ 1. Generate new secret          │
│ 2. Update secret store          │
│ 3. Restart all applications     │ ← Downtime!
│ 4. Verify connectivity          │
│ 5. Remove old secret            │
└─────────────────────────────────┘

Automated Rotation:
┌─────────────────────────────────┐
│ Secret Manager                  │
│ ├── Rotation policy (30 days)   │
│ ├── Automatic generation        │
│ └── Gradual rollout             │
│                                 │
│ Application Support:            │
│ ├── Watch for secret changes    │
│ ├── Hot reload configuration    │
│ ├── Graceful connection switch  │
│ └── Fallback to previous secret │
└─────────────────────────────────┘

Blue-Green Secret Rotation:
──────────────────────────
┌─────────────────────────────────┐
│ Phase 1: Both secrets active    │
│ ├── Old secret: prod traffic    │
│ └── New secret: ready           │
│                                 │
│ Phase 2: Gradual migration      │
│ ├── Old secret: 75% traffic     │
│ └── New secret: 25% traffic     │
│                                 │
│ Phase 3: Complete migration     │
│ ├── Old secret: deactivated     │
│ └── New secret: 100% traffic    │
└─────────────────────────────────┘
```

### Backup и disaster recovery

#### Stateful applications backup

```
Container Data Backup Strategies:
════════════════════════════════

Volume Backup Approaches:
┌─────────────────────────────────┐
│ 1. Snapshot-based Backup:      │
│    ┌─────────────┐              │
│    │ Container   │              │
│    │     │       │              │
│    │     ▼       │              │
│    │ Volume ─────┼──── Snapshot │
│    │ (Running)   │      (Point  │
│    └─────────────┘       in time)│
│                                 │
│ 2. Application-consistent:      │
│    ┌─────────────┐              │
│    │   App       │              │
│    │ 1.Quiesce ──┼─┐            │
│    │ 2.Snapshot ─┼─┼─── Backup  │
│    │ 3.Resume ───┼─┘            │
│    └─────────────┘              │
│                                 │
│ 3. Continuous Replication:      │
│    ┌─────────────┐ ┌──────────┐ │
│    │ Primary ────┼→│ Replica  │ │
│    │ Volume      │ │ Volume   │ │
│    └─────────────┘ └──────────┘ │
└─────────────────────────────────┘

Database Backup Patterns:
────────────────────────
┌─────────────────────────────────┐
│ PostgreSQL:                     │
│ ├── pg_dump (logical)           │
│ ├── pg_basebackup (physical)    │
│ └── WAL shipping               │
│                                 │
│ MySQL:                          │
│ ├── mysqldump (logical)         │
│ ├── Percona XtraBackup         │
│ └── Binary log replication     │
│                                 │
│ MongoDB:                        │
│ ├── mongodump (logical)         │
│ ├── Filesystem snapshots       │
│ └── Replica set backups        │
└─────────────────────────────────┘
```

#### Disaster recovery planning

```
Disaster Recovery Levels:
════════════════════════

RTO/RPO Matrix:
┌─────────────────────────────────┐
│      │ RPO    │ RTO     │ Cost  │
│──────┼────────┼─────────┼───────│
│ Tier 1│ <1min  │ <5min   │ High  │ ← Mission critical
│ Tier 2│ <1hr   │ <1hr    │ Med   │ ← Important apps
│ Tier 3│ <24hr  │ <24hr   │ Low   │ ← Non-critical
│ Tier 4│ <week  │ <week   │ Min   │ ← Archive/backup
└─────────────────────────────────┘

RPO = Recovery Point Objective (data loss)
RTO = Recovery Time Objective (downtime)

Multi-Region DR Architecture:
────────────────────────────
Primary Region (Active):
┌─────────────────────────────────┐
│ ┌──────────┐ ┌──────────┐      │
│ │   App    │ │ Database │      │
│ │Containers│ │ Primary  │      │
│ └──────────┘ └─────┬────┘      │
└────────────────────┼────────────┘
                     │ Replication
                     ▼
DR Region (Standby):
┌─────────────────────────────────┐
│ ┌──────────┐ ┌──────────┐      │
│ │   App    │ │ Database │      │
│ │ (Scaled  │ │ Replica  │      │
│ │  to 0)   │ │ (Read    │      │
│ └──────────┘ │  Only)   │      │
└─────────────────────────────────┘

Failover Process:
────────────────
1. Primary region failure detected
2. Promote DR database to primary
3. Scale up application containers
4. Update DNS to point to DR region
5. Verify service functionality
6. Communicate status to stakeholders

Typical Failover Timeline:
   Detection: 1-5 minutes
   DNS Update: 1-10 minutes
   App Startup: 2-15 minutes
   Verification: 5-10 minutes
   Total RTO: 9-40 minutes
```

### Compliance и security

#### Security scanning integration

```
Container Security Pipeline:
═══════════════════════════

Build-time Security:
┌─────────────────────────────────┐
│ 1. Base Image Scanning:         │
│    ├── Known vulnerabilities    │
│    ├── License compliance       │
│    └── Configuration issues     │
│                                 │
│ 2. Dependency Scanning:         │
│    ├── Application libraries    │
│    ├── OS packages             │
│    └── Transitive dependencies  │
│                                 │
│ 3. Static Code Analysis:        │
│    ├── Security patterns        │
│    ├── Hardcoded secrets        │
│    └── Dangerous practices      │
└─────────────────────────────────┘

Runtime Security:
┌─────────────────────────────────┐
│ 1. Runtime Behavior Monitoring: │
│    ├── Process activity         │
│    ├── Network connections      │
│    ├── File system access       │
│    └── System call patterns     │
│                                 │
│ 2. Network Security:            │
│    ├── Traffic encryption       │
│    ├── Network policies         │
│    ├── Ingress/egress control   │
│    └── Service mesh security    │
│                                 │
│ 3. Access Control:              │
│    ├── RBAC enforcement         │
│    ├── Pod security policies    │
│    ├── Service accounts         │
│    └── Privilege escalation     │
└─────────────────────────────────┘

Security Scanning Tools:
───────────────────────
┌─────────────────────────────────┐
│ Open Source:                    │
│ ├── Trivy (Aqua Security)       │
│ ├── Grype (Anchore)            │
│ ├── Clair (CoreOS)             │
│ └── Snyk (freemium)            │
│                                 │
│ Commercial:                     │
│ ├── Twistlock/Prisma Cloud      │
│ ├── Aqua Security Platform      │
│ ├── Sysdig Secure              │
│ └── Qualys Container Security   │
└─────────────────────────────────┘
```

---

## 🎯 Заключение блока

### Production Readiness Checklist

```
Production Readiness Assessment:
═══════════════════════════════

🔍 Observability:
┌─────────────────────────────────┐
│ ✅ Structured logging           │
│ ✅ Metrics collection           │
│ ✅ Distributed tracing          │
│ ✅ Alerting rules               │
│ ✅ Dashboards created           │
│ ✅ Runbooks documented          │
└─────────────────────────────────┘

⚡ Performance:
┌─────────────────────────────────┐
│ ✅ Resource limits set          │
│ ✅ Load testing completed       │
│ ✅ Performance baseline         │
│ ✅ Horizontal scaling setup     │
│ ✅ Cache strategies implement   │
│ ✅ Database optimization        │
└─────────────────────────────────┘

🔒 Security:
┌─────────────────────────────────┐
│ ✅ Image vulnerability scan     │
│ ✅ Secrets properly managed     │
│ ✅ Network policies configured  │
│ ✅ Access controls implemented  │
│ ✅ Security monitoring active   │
│ ✅ Compliance requirements met  │
└─────────────────────────────────┘

🛡️  Reliability:
┌─────────────────────────────────┐
│ ✅ Health checks configured     │
│ ✅ Graceful shutdown implement  │
│ ✅ Backup strategy active       │
│ ✅ Disaster recovery tested     │
│ ✅ Circuit breakers in place    │
│ ✅ Chaos engineering practices  │
└─────────────────────────────────┘
```

### Ключевые навыки освоенные

После завершения этого блока вы можете:

1. **Построить comprehensive observability stack** с логированием, метриками и трейсингом
2. **Оптимизировать производительность** контейнеров на всех уровнях - от образов до runtime
3. **Настроить production-grade security** с автоматическим сканированием и secrets management
4. **Реализовать надежные backup и DR стратегии** для stateful приложений
5. **Мониторить и алертить** на все критичные аспекты работы системы

### Переход к экспертному уровню

В **Блоке 7: Экспертный уровень** мы изучим:
- Продвинутые техники контейнеризации
- Архитектурные паттерны для сложных систем
- Container runtime экосистему
- Будущее технологий контейнеризации
- Подготовку к профессиональной сертификации

---

### 💡 Production Wisdom

```
Production Lessons Learned:
══════════════════════════

🎯 Monitoring Philosophy:
"Monitor what matters to users, not just what's easy to measure"

📊 Metrics Strategy:
"Start with business metrics, then drill down to technical metrics"

🔧 Performance Approach:
"Optimize for the bottleneck, not the average case"

🛡️  Security Mindset:
"Security is not a feature, it's a foundational requirement"

🔄 Reliability Principle:
"Build for failure, not just for success"

📚 Documentation Rule:
"If it's not documented, it doesn't exist in production"
```

Теперь вы готовы стать экспертом Docker и справляться с любыми production challenges! 🚀