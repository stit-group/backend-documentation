# 📊 Блок 5: Мониторинг, логирование и отладка в Kubernetes

**⏱️ Продолжительность:** 2-3 недели  
**🎯 Цель:** Научиться поддерживать и диагностировать кластер  
**🎓 Уровень:** Средний → Продвинутый

---

## 📖 Введение в Observability

Observability (наблюдаемость) в Kubernetes — это способность понимать внутреннее состояние системы по ее внешним выходным данным.

**🔍 Три столпа Observability:**
```
┌─────────────────────────────────────────────────────┐
│                 OBSERVABILITY PILLARS              │
├─────────────────┬─────────────────┬─────────────────┤
│    METRICS      │      LOGS       │     TRACES      │
├─────────────────┼─────────────────┼─────────────────┤
│ Числовые данные │ Текстовые       │ Путь запроса    │
│ во времени      │ события         │ через систему   │
│                 │                 │                 │
│ • CPU/Memory    │ • Errors        │ • Latency       │
│ • Request rate  │ • Events        │ • Dependencies  │
│ • Error rate    │ • Debugging     │ • Bottlenecks   │
│ • Duration      │ • Audit         │ • Flow          │
└─────────────────┴─────────────────┴─────────────────┘
```

**🎯 Принципы эффективного мониторинга:**

1. **Проактивность vs Реактивность**
```
РЕАКТИВНЫЙ ПОДХОД:          ПРОАКТИВНЫЙ ПОДХОД:
Проблема → Обнаружение      Мониторинг → Предсказание → Превенция
    ↓                           ↓
Расследование                Automated Response
    ↓                           ↓
Устранение                   Minimal Impact
```

2. **SLI, SLO, SLA концепции**
```
SLA (Service Level Agreement)
├─ Внешние договоренности с клиентами
├─ 99.9% uptime = 8.76 часов downtime в год
└─ Бизнес-последствия при нарушении

SLO (Service Level Objective)  
├─ Внутренние цели команды
├─ 99.95% availability для critical services
└─ Error budget для экспериментов

SLI (Service Level Indicator)
├─ Фактические измерения
├─ Request success rate: 99.97%
└─ P99 latency: 150ms
```

---

## 5.1 Мониторинг с Prometheus и Grafana

### 🎯 Архитектура Prometheus

Prometheus — это система мониторинга, построенная на принципе pull-модели сбора метрик.

**🏗️ Компоненты экосистемы Prometheus:**
```
┌─────────────────────────────────────────────────────┐
│              PROMETHEUS ECOSYSTEM                   │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌─────────────┐    ┌─────────────────┐            │
│  │ Prometheus  │◄───│   Exporters     │            │
│  │   Server    │    │ • node-exporter │            │
│  │             │    │ • kube-state    │            │
│  └─────────────┘    │ • custom apps   │            │
│         │            └─────────────────┘            │
│         │                                           │
│         ▼                                           │
│  ┌─────────────┐    ┌─────────────────┐            │
│  │   Grafana   │    │  AlertManager   │            │
│  │ Dashboards  │    │  Notifications  │            │
│  └─────────────┘    └─────────────────┘            │
└─────────────────────────────────────────────────────┘
```

**🔄 Pull vs Push модели:**
```
PULL MODEL (Prometheus):
┌─────────────┐     HTTP GET     ┌─────────────┐
│ Prometheus  │ ←─────────────── │   Target    │
│   Server    │    /metrics      │ Application │
└─────────────┘                  └─────────────┘

Преимущества:
├─ Service discovery
├─ Health checking
├─ Target awareness
└─ Simpler debugging

PUSH MODEL (традиционный):
┌─────────────┐     HTTP POST    ┌─────────────┐
│ Monitoring  │ ←─────────────── │   Target    │
│   Server    │    metrics       │ Application │
└─────────────┘                  └─────────────┘

Проблемы:
├─ Configuration complexity  
├─ No health visibility
├─ Firewall issues
└─ Load balancing challenges
```

---

### 📊 Типы метрик в Prometheus

**1. Counter (счетчик)**
```
Характеристики:
├─ Монотонно возрастающее значение
├─ Сбрасывается только при рестарте
├─ Используется с функцией rate()
└─ Примеры: HTTP requests, errors, bytes sent

Пример метрики:
http_requests_total{method="GET", status="200"} 1547

PromQL запросы:
rate(http_requests_total[5m])          # Requests per second
increase(http_requests_total[1h])      # Total increase in 1 hour
```

**2. Gauge (измеритель)**
```
Характеристики:
├─ Может увеличиваться и уменьшаться
├─ Отражает текущее состояние
├─ Snapshot значения в момент времени
└─ Примеры: CPU usage, memory, temperature

Пример метрики:
cpu_usage_percent{instance="node-1"} 75.5

PromQL запросы:
avg(cpu_usage_percent)                 # Average CPU across instances
max(memory_usage_bytes)                # Peak memory usage
```

**3. Histogram (гистограмма)**
```
Характеристики:
├─ Распределение значений по buckets
├─ Автоматически создает _count, _sum, _bucket
├─ Позволяет вычислять квантили
└─ Примеры: request duration, response size

Пример метрик:
http_request_duration_seconds_bucket{le="0.1"} 100
http_request_duration_seconds_bucket{le="0.5"} 150
http_request_duration_seconds_bucket{le="1.0"} 200
http_request_duration_seconds_bucket{le="+Inf"} 200
http_request_duration_seconds_sum 95.5
http_request_duration_seconds_count 200

PromQL запросы:
histogram_quantile(0.95, http_request_duration_seconds_bucket)  # P95
rate(http_request_duration_seconds_sum[5m]) /                   # Average
rate(http_request_duration_seconds_count[5m])                   # duration
```

**4. Summary (сводка)**
```
Характеристики:
├─ Предварительно вычисленные квантили
├─ Меньше гибкости, но выше производительность
├─ Вычисления на стороне клиента
└─ Примеры: response time percentiles

Пример метрик:
http_request_duration_seconds{quantile="0.5"} 0.12
http_request_duration_seconds{quantile="0.9"} 0.38
http_request_duration_seconds{quantile="0.99"} 0.95
http_request_duration_seconds_sum 8953.332
http_request_duration_seconds_count 27892
```

---

### 🎯 Prometheus Operator и ServiceMonitor

**Kubernetes-native подход к мониторингу:**

```
┌─────────────────────────────────────────────────────┐
│            PROMETHEUS OPERATOR                      │
├─────────────────────────────────────────────────────┤
│                                                     │
│ ┌─────────────┐   manages   ┌─────────────────┐    │
│ │  Operator   │ ──────────► │   Prometheus    │    │
│ │             │             │    Instance     │    │
│ └─────────────┘             └─────────────────┘    │
│         │                            ▲              │
│         │ watches                    │              │
│         ▼                            │              │
│ ┌─────────────┐             ┌─────────────────┐    │
│ │ServiceMonitor│ ──────────► │ Scrape Config   │    │
│ │    CRDs     │  generates  │                 │    │
│ └─────────────┘             └─────────────────┘    │
└─────────────────────────────────────────────────────┘
```

**ServiceMonitor пример:**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app-monitor
  labels:
    app: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
  - port: metrics
    path: /metrics
    interval: 30s
    scrapeTimeout: 10s
    # Дополнительная конфигурация
    relabelings:
    - sourceLabels: [__meta_kubernetes_pod_name]
      targetLabel: pod_name
    - sourceLabels: [__meta_kubernetes_namespace]  
      targetLabel: kubernetes_namespace
```

**PrometheusRule для алертов:**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: my-app-alerts
spec:
  groups:
  - name: my-app.rules
    rules:
    - alert: HighErrorRate
      expr: |
        (
          rate(http_requests_total{status=~"5.."}[5m]) /
          rate(http_requests_total[5m])
        ) > 0.05
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "High error rate detected"
        description: "Error rate is {{ $value | humanizePercentage }} for {{ $labels.instance }}"
        
    - alert: HighMemoryUsage
      expr: |
        (
          container_memory_working_set_bytes{container!=""} /
          container_spec_memory_limit_bytes{container!=""} 
        ) > 0.8
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: "High memory usage detected"
        description: "Memory usage is above 80% for container {{ $labels.container }}"
```

---

### 📈 Grafana: визуализация и дашборды

**🎨 Типы визуализаций в Grafana:**

```
┌─────────────────────────────────────────────────────┐
│                VISUALIZATION TYPES                  │
├─────────────────┬─────────────────┬─────────────────┤
│   TIME SERIES   │     SINGLE      │    SPECIAL      │
├─────────────────┼─────────────────┼─────────────────┤
│ • Graph         │ • Stat          │ • Heatmap       │
│ • Area chart    │ • Gauge         │ • Histogram     │
│ • Bar gauge     │ • Big number    │ • Node graph    │
│ • State timeline│ • Progress bar  │ • Service map   │
└─────────────────┴─────────────────┴─────────────────┘
```

**🏗️ Структура эффективного дашборда:**

```
DASHBOARD HIERARCHY:
┌─────────────────────────────────────────────────────┐
│                 OVERVIEW LEVEL                      │
│ High-level SLIs: Availability, Latency, Throughput │
├─────────────────────────────────────────────────────┤
│                SERVICE LEVEL                        │
│ Service-specific metrics: Error rates, Dependencies│
├─────────────────────────────────────────────────────┤
│              INFRASTRUCTURE LEVEL                   │
│ Node metrics: CPU, Memory, Disk, Network           │
├─────────────────────────────────────────────────────┤
│               APPLICATION LEVEL                     │
│ Custom metrics: Business KPIs, Custom counters     │
└─────────────────────────────────────────────────────┘
```

**📊 PromQL запросы для Kubernetes:**

1. **Pod CPU Usage:**
```promql
# CPU usage per pod
rate(container_cpu_usage_seconds_total{container!="POD",container!=""}[5m])

# CPU usage as percentage of limit
rate(container_cpu_usage_seconds_total{container!="POD"}[5m]) / 
on(container,pod,namespace) 
container_spec_cpu_quota{container!="POD"} * 
container_spec_cpu_period{container!="POD"}
```

2. **Memory Usage:**
```promql
# Memory working set
container_memory_working_set_bytes{container!="POD",container!=""}

# Memory usage as percentage of limit  
container_memory_working_set_bytes{container!="POD"} /
on(container,pod,namespace) 
container_spec_memory_limit_bytes{container!="POD"} * 100
```

3. **Network I/O:**
```promql
# Network receive rate
rate(container_network_receive_bytes_total[5m])

# Network transmit rate
rate(container_network_transmit_bytes_total[5m])
```

4. **Disk Usage:**
```promql
# Filesystem usage percentage
(
  node_filesystem_size_bytes{fstype!="tmpfs"} - 
  node_filesystem_free_bytes{fstype!="tmpfs"}
) / 
node_filesystem_size_bytes{fstype!="tmpfs"} * 100
```

---

### 🚨 AlertManager: управление уведомлениями

**🏗️ Архитектура AlertManager:**

```
┌─────────────────────────────────────────────────────┐
│                 ALERT FLOW                          │
├─────────────────────────────────────────────────────┤
│                                                     │
│ ┌─────────────┐   fires    ┌─────────────────┐     │
│ │ Prometheus  │ ─────────► │  AlertManager   │     │
│ │    Rules    │   alerts   │                 │     │
│ └─────────────┘            └─────────────────┘     │
│                                     │               │
│                            ┌────────┼────────┐      │
│                            ▼        ▼        ▼      │
│                     ┌─────────┐ ┌──────┐ ┌──────┐  │
│                     │  Email  │ │Slack │ │ PD   │  │
│                     └─────────┘ └──────┘ └──────┘  │
└─────────────────────────────────────────────────────┘
```

**🔧 Конфигурация AlertManager:**

```yaml
# alertmanager.yml
global:
  smtp_smarthost: 'localhost:587'
  smtp_from: 'alerts@company.com'

# Маршрутизация алертов
route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 30s           # Ждать 30с перед отправкой группы
  group_interval: 5m        # Интервал между группами
  repeat_interval: 12h      # Повтор уведомления
  receiver: 'default'
  
  routes:
  # Критические алерты сразу на PagerDuty
  - match:
      severity: critical
    receiver: 'pagerduty'
    group_wait: 10s
    
  # Алерты инфраструктуры в отдельный канал
  - match_re:
      alertname: ^(NodeDown|DiskSpaceLow)$
    receiver: 'infrastructure-team'

# Получатели уведомлений
receivers:
- name: 'default'
  slack_configs:
  - api_url: 'https://hooks.slack.com/...'
    channel: '#alerts'
    title: 'Alert: {{ .GroupLabels.alertname }}'
    text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

- name: 'pagerduty'
  pagerduty_configs:
  - service_key: 'YOUR_PAGERDUTY_KEY'
    description: '{{ .GroupLabels.alertname }}: {{ .GroupLabels.instance }}'

- name: 'infrastructure-team'
  email_configs:
  - to: 'infra-team@company.com'
    subject: 'Infrastructure Alert: {{ .GroupLabels.alertname }}'
    body: |
      Alert Details:
      {{ range .Alerts }}
      - {{ .Annotations.summary }}
      - Instance: {{ .Labels.instance }}
      - Value: {{ .Annotations.value }}
      {{ end }}

# Подавление алертов
inhibit_rules:
- source_match:
    severity: 'critical'
  target_match:
    severity: 'warning'
  equal: ['alertname', 'cluster', 'service']
```

---

## 5.2 Централизованное логирование

### 🎯 Архитектура логирования в Kubernetes

**🏗️ Компоненты log pipeline:**

```
┌─────────────────────────────────────────────────────┐
│              LOGGING ARCHITECTURE                   │
├─────────────────────────────────────────────────────┤
│                                                     │
│ ┌─────────────┐    ┌─────────────────┐             │
│ │    PODS     │───►│   Log Agents    │             │
│ │  (stdout)   │    │ • Fluentd       │             │
│ │  (stderr)   │    │ • Fluent Bit    │             │
│ │  (files)    │    │ • Filebeat      │             │
│ └─────────────┘    └─────────────────┘             │
│                             │                       │
│                             ▼                       │
│                    ┌─────────────────┐             │
│                    │   Aggregation   │             │
│                    │ • Kafka         │             │
│                    │ • Redis         │             │
│                    │ • Logstash      │             │
│                    └─────────────────┘             │
│                             │                       │
│                             ▼                       │
│ ┌─────────────┐    ┌─────────────────┐             │
│ │   Storage   │◄───│   Processing    │             │
│ │ • Elastic   │    │ • Parsing       │             │
│ │ • Loki      │    │ • Enrichment    │             │
│ │ • BigQuery  │    │ • Filtering     │             │
│ └─────────────┘    └─────────────────┘             │
│         │                                           │
│         ▼                                           │
│ ┌─────────────┐                                    │
│ │Visualization│                                    │
│ │ • Kibana    │                                    │
│ │ • Grafana   │                                    │
│ └─────────────┘                                    │
└─────────────────────────────────────────────────────┘
```

---

### 📝 Fluentd vs Fluent Bit: выбор агента

**🔄 Сравнение log агентов:**

```
┌─────────────────────────────────────────────────────┐
│              FLUENTD vs FLUENT BIT                  │
├─────────────────────┬───────────────────────────────┤
│      FLUENTD        │         FLUENT BIT            │
├─────────────────────┼───────────────────────────────┤
│ • Ruby-based        │ • C-based                     │
│ • Богатая экосистема│ • Минимальное потребление     │
│ • 40MB+ RAM         │ • ~650KB RAM                  │
│ • Сложная обработка │ • Простая обработка           │
│ • Плагины на Ruby   │ • Встроенные processors       │
│ • Aggregator роль   │ • Edge agent роль             │
└─────────────────────┴───────────────────────────────┘
```

**🎯 Рекомендуемая архитектура:**
```
NODE LEVEL:                    CLUSTER LEVEL:
┌─────────────┐               ┌─────────────────┐
│ Fluent Bit  │──────────────►│    Fluentd      │
│ DaemonSet   │   forwards    │   Deployment    │
│             │               │                 │
│ • Collect   │               │ • Aggregate     │
│ • Parse     │               │ • Process       │
│ • Forward   │               │ • Route         │
└─────────────┘               │ • Output        │
                              └─────────────────┘
```

---

### 🔧 Конфигурация Fluent Bit

**DaemonSet конфигурация:**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
  namespace: logging
spec:
  selector:
    matchLabels:
      name: fluent-bit
  template:
    metadata:
      labels:
        name: fluent-bit
    spec:
      containers:
      - name: fluent-bit
        image: fluent/fluent-bit:latest
        ports:
        - containerPort: 2020
        volumeMounts:
        - name: varlog
          mountPath: /var/log
          readOnly: true
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: config
          mountPath: /fluent-bit/etc/
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      - name: config
        configMap:
          name: fluent-bit-config
```

**ConfigMap с парсингом:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush         1
        Log_Level     info
        Daemon        off
        Parsers_File  parsers.conf
    
    [INPUT]
        Name              tail
        Path              /var/log/containers/*.log
        Parser            docker
        Tag               kube.*
        Refresh_Interval  5
        Mem_Buf_Limit     50MB
        Skip_Long_Lines   On
    
    [FILTER]
        Name                kubernetes
        Match               kube.*
        Kube_URL            https://kubernetes.default.svc:443
        Kube_CA_File        /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        Kube_Token_File     /var/run/secrets/kubernetes.io/serviceaccount/token
        Merge_Log           On
        K8S-Logging.Parser  On
        K8S-Logging.Exclude Off
    
    [OUTPUT]
        Name  es
        Match *
        Host  elasticsearch.logging.svc.cluster.local
        Port  9200
        Index fluentbit
        Type  _doc

  parsers.conf: |
    [PARSER]
        Name   docker
        Format json
        Time_Key time
        Time_Format %Y-%m-%dT%H:%M:%S.%L
    
    [PARSER]
        Name        nginx
        Format      regex
        Regex       ^(?<remote>[^ ]*) (?<host>[^ ]*) (?<user>[^ ]*) \[(?<time>[^\]]*)\] "(?<method>\S+)(?: +(?<path>[^\"]*?)(?: +\S*)?)?" (?<code>[^ ]*) (?<size>[^ ]*)(?: "(?<referer>[^\"]*)" "(?<agent>[^\"]*)")?$
        Time_Key    time
        Time_Format %d/%b/%Y:%H:%M:%S %z
```

---

### 🔍 Structured Logging Best Practices

**📋 Принципы структурированного логирования:**

```
НЕСТРУКТУРИРОВАННЫЕ ЛОГИ:
2024-01-15 10:30:45 ERROR User john.doe failed to login from 192.168.1.100

СТРУКТУРИРОВАННЫЕ ЛОГИ:
{
  "timestamp": "2024-01-15T10:30:45.123Z",
  "level": "ERROR",
  "message": "User login failed",
  "user_id": "john.doe",
  "source_ip": "192.168.1.100",
  "event_type": "authentication",
  "request_id": "req-12345",
  "service": "auth-service",
  "namespace": "production"
}
```

**🏷️ Стандартные поля для Kubernetes:**
```json
{
  "timestamp": "2024-01-15T10:30:45.123Z",
  "level": "INFO|WARN|ERROR|DEBUG",
  "message": "Human readable message",
  
  // Kubernetes metadata
  "kubernetes": {
    "namespace": "production",
    "pod_name": "web-app-7d4b8c9f5-xyz12",
    "container_name": "web-app",
    "node_name": "worker-node-1",
    "labels": {
      "app": "web-app",
      "version": "v1.2.3"
    }
  },
  
  // Request tracing
  "trace_id": "64f8a2b1-4c5d-4e1f-8a5c-123456789abc",
  "span_id": "7a2b8c9d",
  "request_id": "req-98765",
  
  // Business context
  "user_id": "user-12345",
  "session_id": "sess-67890",
  "operation": "create_order",
  
  // Technical details
  "duration_ms": 245,
  "status_code": 200,
  "error_code": null
}
```

---

### 📊 Log Retention и Storage

**🗄️ Стратегии хранения логов:**

```
┌─────────────────────────────────────────────────────┐
│              LOG RETENTION STRATEGY                 │
├─────────────────┬─────────────────┬─────────────────┤
│   HOT STORAGE   │   WARM STORAGE  │  COLD STORAGE   │
├─────────────────┼─────────────────┼─────────────────┤
│ Last 7 days     │ 7-30 days       │ 30+ days        │
│ SSD, fast query │ Cheaper SSD     │ Object storage  │
│ Real-time alerts│ Investigation   │ Compliance      │
│ High cost       │ Medium cost     │ Low cost        │
│ Elasticsearch   │ Elasticsearch   │ S3, GCS         │
└─────────────────┴─────────────────┴─────────────────┘
```

**Index Lifecycle Management (ILM):**
```json
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_size": "50gb",
            "max_age": "1d"
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "allocate": {
            "number_of_replicas": 0
          },
          "set_priority": {
            "priority": 50
          }
        }
      },
      "cold": {
        "min_age": "30d",
        "actions": {
          "allocate": {
            "number_of_replicas": 0
          },
          "set_priority": {
            "priority": 0
          }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

---

## 5.3 Трассировка и APM

### 🎯 Distributed Tracing концепции

**🔍 Анатомия распределенной трассировки:**

```
┌─────────────────────────────────────────────────────┐
│              DISTRIBUTED TRACE                      │
├─────────────────────────────────────────────────────┤
│                                                     │
│ TRACE ID: abc123def456                              │
│                                                     │
│ ┌─ SPAN: Frontend (100ms) ──────────────────────┐   │
│ │                                               │   │
│ │  ┌─ SPAN: API Gateway (80ms) ──────────────┐  │   │
│ │  │                                         │  │   │
│ │  │  ┌─ SPAN: Auth Service (20ms) ───┐     │  │   │
│ │  │  └─────────────────────────────────┘     │  │   │
│ │  │                                         │  │   │
│ │  │  ┌─ SPAN: User Service (40ms) ─────────┐│  │   │
│ │  │  └─────────────────────────────────────┘│  │   │
│ │  │                                         │  │   │
│ │  │  ┌─ SPAN: Database (15ms) ────┐        │  │   │
│ │  │  └─────────────────────────────┘        │  │   │
│ │  └─────────────────────────────────────────┘  │   │
│ └───────────────────────────────────────────────┘   │
│                                                     │
│ Timeline: 0ms    25ms    50ms    75ms    100ms      │
└─────────────────────────────────────────────────────┘
```

**📊 Ключевые метрики трассировки:**

```
PERFORMANCE METRICS:
├─ Latency Distribution
│  ├─ P50: 150ms (median)
│  ├─ P95: 500ms 
│  ├─ P99: 1200ms
│  └─ P99.9: 3000ms
│
├─ Error Rate Analysis
│  ├─ Service-level errors
│  ├─ Dependency failures
│  └─ Timeout patterns
│
└─ Throughput Metrics
   ├─ Requests per second
   ├─ Service saturation
   └─ Bottleneck identification
```

---

### 🛠️ OpenTelemetry: стандарт наблюдаемости

**🏗️ Архитектура OpenTelemetry:**

```
┌─────────────────────────────────────────────────────┐
│               OPENTELEMETRY STACK                   │
├─────────────────────────────────────────────────────┤
│                                                     │
│ ┌─────────────┐   ┌─────────────────┐              │
│ │Application  │──►│ OTel SDK/Agent  │              │
│ │   Code      │   │ • Auto-instru   │              │
│ │             │   │ • Manual instru │              │
│ └─────────────┘   └─────────────────┘              │
│                            │                        │
│                            ▼                        │
│                   ┌─────────────────┐              │
│                   │ OTel Collector  │              │
│                   │ • Receive       │              │
│                   │ • Process       │              │
│                   │ • Export        │              │
│                   └─────────────────┘              │
│                            │                        │
│              ┌─────────────┼─────────────┐          │
│              ▼             ▼             ▼          │
│      ┌─────────────┐ ┌──────────┐ ┌─────────────┐  │
│      │   Jaeger    │ │Prometheus│ │  Backend    │  │
│      │  (traces)   │ │(metrics) │ │  (logs)     │  │
│      └─────────────┘ └──────────┘ └─────────────┘  │
└─────────────────────────────────────────────────────┘
```

**🔧 OpenTelemetry Collector конфигурация:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: otel-collector-config
data:
  config.yaml: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318
      jaeger:
        protocols:
          grpc:
            endpoint: 0.0.0.0:14250
          thrift_http:
            endpoint: 0.0.0.0:14268
            
    processors:
      batch:
        timeout: 1s
        send_batch_size: 1024
      
      resource:
        attributes:
        - key: environment
          value: production
          action: upsert
      
      # Sampling для снижения нагрузки
      probabilistic_sampler:
        sampling_percentage: 1.0
        
    exporters:
      jaeger:
        endpoint: jaeger-collector:14250
        tls:
          insecure: true
          
      prometheus:
        endpoint: "0.0.0.0:8889"
        
      logging:
        loglevel: debug
        
    service:
      pipelines:
        traces:
          receivers: [otlp, jaeger]
          processors: [batch, resource, probabilistic_sampler]
          exporters: [jaeger, logging]
          
        metrics:
          receivers: [otlp]
          processors: [batch, resource]
          exporters: [prometheus]
```

---

### 🔍 Jaeger: система трассировки

**🏗️ Компоненты Jaeger:**

```
┌─────────────────────────────────────────────────────┐
│                JAEGER ARCHITECTURE                  │
├─────────────────────────────────────────────────────┤
│                                                     │
│ ┌─────────────┐    ┌─────────────────┐             │
│ │  Jaeger     │───►│  Jaeger Agent   │             │
│ │  Client     │    │  (DaemonSet)    │             │
│ │  (in app)   │    │                 │             │
│ └─────────────┘    └─────────────────┘             │
│                             │                       │
│                             ▼                       │
│                    ┌─────────────────┐             │
│                    │ Jaeger Collector│             │
│                    │  (Deployment)   │             │
│                    │ • Validation    │             │
│                    │ • Processing    │             │
│                    │ • Storage       │             │
│                    └─────────────────┘             │
│                             │                       │
│                             ▼                       │
│ ┌─────────────┐    ┌─────────────────┐             │
│ │   Jaeger    │◄───│    Storage      │             │
│ │   Query     │    │ • Elasticsearch │             │
│ │   (UI)      │    │ • Cassandra     │             │
│ └─────────────┘    │ • Memory        │             │
│                    └─────────────────┘             │
└─────────────────────────────────────────────────────┘
```

**Deployment example:**
```yaml
# Jaeger All-in-One для development
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jaeger
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jaeger
  template:
    metadata:
      labels:
        app: jaeger
    spec:
      containers:
      - name: jaeger
        image: jaegertracing/all-in-one:latest
        env:
        - name: COLLECTOR_ZIPKIN_HTTP_PORT
          value: "9411"
        ports:
        - containerPort: 5775
          protocol: UDP
        - containerPort: 6831
          protocol: UDP
        - containerPort: 6832
          protocol: UDP
        - containerPort: 5778
          protocol: TCP
        - containerPort: 16686
          protocol: TCP
        - containerPort: 14268
          protocol: TCP
        - containerPort: 14250
          protocol: TCP
        - containerPort: 9411
          protocol: TCP
```

---

### 📈 Application Performance Monitoring

**🎯 Key Performance Indicators (KPIs):**

```
┌─────────────────────────────────────────────────────┐
│                 APM METRICS                         │
├─────────────────┬─────────────────┬─────────────────┤
│    GOLDEN       │    BUSINESS     │   TECHNICAL     │
│   SIGNALS       │    METRICS      │    METRICS      │
├─────────────────┼─────────────────┼─────────────────┤
│ • Latency       │ • Conversion    │ • CPU Usage     │
│ • Throughput    │ • Revenue       │ • Memory Usage  │
│ • Errors        │ • User Activity │ • Disk I/O      │
│ • Saturation    │ • Cart Size     │ • Network I/O   │
└─────────────────┴─────────────────┴─────────────────┘
```

**🔍 Synthetic Monitoring:**
```yaml
# Synthetic tests для проверки сервисов
apiVersion: batch/v1
kind: CronJob
metadata:
  name: synthetic-test
spec:
  schedule: "*/5 * * * *"  # каждые 5 минут
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: test
            image: curlimages/curl:latest
            command:
            - /bin/sh
            - -c
            - |
              # Health check
              RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" http://my-app/health)
              if [ $RESPONSE -ne 200 ]; then
                echo "Health check failed: $RESPONSE"
                exit 1
              fi
              
              # Performance test
              LATENCY=$(curl -s -o /dev/null -w "%{time_total}" http://my-app/api/users)
              echo "API latency: ${LATENCY}s"
              
              # Functional test
              curl -s -X POST -H "Content-Type: application/json" \
                -d '{"test": "data"}' \
                http://my-app/api/test
          restartPolicy: Never
```

---

## 5.4 Отладка подов и сервисов

### 🎯 Systematic Troubleshooting Approach

**🔍 Troubleshooting Workflow:**

```
┌─────────────────────────────────────────────────────┐
│            KUBERNETES TROUBLESHOOTING               │
├─────────────────────────────────────────────────────┤
│                                                     │
│ 1. GATHER INFORMATION                               │
│    ├─ kubectl get pods -o wide                      │
│    ├─ kubectl describe pod <name>                   │
│    └─ kubectl logs <pod> --previous                 │
│                                                     │
│ 2. CHECK RESOURCES                                  │
│    ├─ Resource requests/limits                      │
│    ├─ Node capacity                                 │
│    └─ Storage availability                          │
│                                                     │
│ 3. VERIFY NETWORKING                                │
│    ├─ Service endpoints                             │
│    ├─ DNS resolution                                │
│    └─ Network policies                              │
│                                                     │
│ 4. EXAMINE CONFIGURATION                            │
│    ├─ ConfigMaps/Secrets                           │
│    ├─ Environment variables                         │
│    └─ Volume mounts                                 │
│                                                     │
│ 5. TEST CONNECTIVITY                                │
│    ├─ Pod-to-pod communication                      │
│    ├─ External dependencies                         │
│    └─ Load balancer health                          │
└─────────────────────────────────────────────────────┘
```

---

### 🐛 Common Pod Issues и их решение

**1. ImagePullBackOff / ErrImagePull**
```bash
# Диагностика
kubectl describe pod <pod-name>

Возможные причины:
├─ Неправильное имя образа
├─ Образ не существует в registry
├─ Отсутствуют права на pull
├─ Registry недоступен
└─ imagePullSecrets неправильно настроены

# Решение
kubectl create secret docker-registry my-secret \
  --docker-server=registry.example.com \
  --docker-username=myuser \
  --docker-password=mypass \
  --docker-email=email@example.com

# В pod spec
spec:
  imagePullSecrets:
  - name: my-secret
```

**2. CrashLoopBackOff**
```bash
# Получить логи предыдущего контейнера
kubectl logs <pod-name> --previous

# Проверить exit code
kubectl describe pod <pod-name>

Анализ проблем:
├─ Exit Code 0: Нормальное завершение
├─ Exit Code 1: Общая ошибка
├─ Exit Code 125: Docker daemon error
├─ Exit Code 126: Контейнер не исполняемый
├─ Exit Code 127: Контейнер не найден
└─ Exit Code 137: SIGKILL (OOM Killer)
```

**3. Pending Pods**
```bash
# Проверить события
kubectl describe pod <pod-name>

Причины Pending:
├─ Insufficient CPU/Memory
├─ Node selector не совпадает
├─ Taints/Tolerations
├─ PVC не может быть смонтирован
└─ Pod Security constraints

# Проверить node capacity
kubectl describe nodes
kubectl top nodes
```

---

### 🌐 Network Troubleshooting

**🔧 Debugging Network Issues:**

```bash
# 1. Проверить Service endpoints
kubectl get endpoints <service-name>

# 2. Проверить DNS resolution
kubectl run debug --image=busybox -it --rm -- /bin/sh
nslookup <service-name>.<namespace>.svc.cluster.local

# 3. Test connectivity
kubectl exec -it <pod> -- wget -qO- http://<service>:<port>/health

# 4. Проверить iptables rules (на node)
sudo iptables -t nat -L | grep <service-name>

# 5. Проверить kube-proxy logs
kubectl logs -n kube-system -l k8s-app=kube-proxy
```

**🚫 Network Policy Debugging:**
```yaml
# Test pod для проверки connectivity
apiVersion: v1
kind: Pod
metadata:
  name: network-test
  labels:
    app: test
spec:
  containers:
  - name: netshoot
    image: nicolaka/netshoot
    command: ["/bin/sleep", "3600"]
    
---
# Temporary allow-all policy для тестирования
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-test
spec:
  podSelector:
    matchLabels:
      app: test
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - {}
  egress:
  - {}
```

---

### 📊 Performance Debugging

**🔍 Resource Analysis:**

```bash
# 1. Top команды для анализа resource usage
kubectl top pods --containers
kubectl top nodes

# 2. Metrics server query
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods

# 3. cAdvisor metrics (на node)
curl http://localhost:4194/metrics

# 4. Проверить OOM kills
dmesg | grep -i "killed process"
journalctl -u kubelet | grep -i oom
```

**⚡ Performance Optimization:**
```yaml
# Resource tuning пример
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
      limits:
        memory: "512Mi"    # 2x requests
        cpu: "500m"        # Burst capability
    
    # JVM tuning для Java приложений
    env:
    - name: JAVA_OPTS
      value: |
        -Xmx400m 
        -XX:+UseContainerSupport
        -XX:MaxRAMPercentage=75.0
        -XX:+UseG1GC
        -XX:MaxGCPauseMillis=200
```

---

### 🛠️ Advanced Debugging Tools

**1. kubectl debug command (1.18+)**
```bash
# Создать debugging pod с общими инструментами
kubectl debug <pod-name> -it --image=ubuntu --share-processes --copy-to=debug-pod

# Добавить sidecar container к существующему поду
kubectl debug <pod-name> -it --image=busybox --target=<container-name>

# Debug node через privileged pod
kubectl debug node/<node-name> -it --image=ubuntu
```

**2. Ephemeral containers (1.23+)**
```bash
# Добавить ephemeral container к running pod
kubectl debug <pod-name> -it --image=busybox --target=<container>

# Пример для prod debugging без изменения оригинального пода
kubectl debug production-app-123 -it \
  --image=nicolaka/netshoot \
  --target=app-container
```

**3. Профилирование приложений**
```bash
# Go applications с pprof
kubectl port-forward <pod> 6060:6060
go tool pprof http://localhost:6060/debug/pprof/profile

# Java applications с JFR
kubectl exec <pod> -- jcmd <pid> JFR.start duration=60s filename=profile.jfr
kubectl cp <pod>:/path/to/profile.jfr ./profile.jfr
```

---

## 5.5 Health Checks и Probes

### 🎯 Типы Probes в Kubernetes

**🏥 Три типа health checks:**

```
┌─────────────────────────────────────────────────────┐
│                  KUBERNETES PROBES                  │
├─────────────────┬─────────────────┬─────────────────┤
│   LIVENESS      │   READINESS     │    STARTUP      │
├─────────────────┼─────────────────┼─────────────────┤
│ Когда убить     │ Когда принимать │ Когда начать    │
│ контейнер       │ трафик          │ другие проверки │
│                 │                 │                 │
│ • Deadlock      │ • Initialization│ • Slow start    │
│ • Memory leak   │ • Dependencies  │ • Legacy apps   │
│ • Corruption    │ • Load balancer │ • Migration     │
└─────────────────┴─────────────────┴─────────────────┘
```

**⏱️ Lifecycle взаимодействия probes:**
```
POD STARTUP SEQUENCE:
┌─────────────────────────────────────────────────────┐
│                                                     │
│ 1. Container starts                                 │
│    │                                                │
│    ▼                                                │
│ 2. Startup Probe (если настроен)                   │
│    │ ── Проверяется до успеха                       │
│    │ ── Блокирует другие probes                     │
│    ▼                                                │
│ 3. Liveness + Readiness Probes                     │
│    │ ── Работают параллельно                        │
│    │ ── Продолжают всю жизнь пода                   │
│    ▼                                                │
│ 4. Pod Ready для получения трафика                 │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

### 🔧 Конфигурация Probes

**HTTP Health Check пример:**
```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: web-app
    image: nginx
    ports:
    - containerPort: 80
    
    # Liveness Probe - перезапуск при сбое
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: health-check
      initialDelaySeconds: 30    # Ждать 30с после старта
      periodSeconds: 10          # Проверять каждые 10с
      timeoutSeconds: 5          # Timeout на запрос
      failureThreshold: 3        # 3 неудачи = restart
      successThreshold: 1        # 1 успех = healthy
    
    # Readiness Probe - исключение из Service
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      timeoutSeconds: 3
      failureThreshold: 3
      successThreshold: 1
    
    # Startup Probe - для медленно стартующих приложений
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      initialDelaySeconds: 0
      periodSeconds: 10
      timeoutSeconds: 3
      failureThreshold: 30       # 30 * 10s = 5 минут на старт
      successThreshold: 1
```

**TCP и Command Probes:**
```yaml
# TCP Socket probe
livenessProbe:
  tcpSocket:
    port: 5432
  initialDelaySeconds: 15
  periodSeconds: 10

# Command execution probe  
readinessProbe:
  exec:
    command:
    - /bin/sh
    - -c
    - "pg_isready -U $POSTGRES_USER -d $POSTGRES_DB"
  initialDelaySeconds: 5
  periodSeconds: 5
```

---

### 💡 Best Practices для Health Checks

**🎯 Design Principles:**

```
ЭФФЕКТИВНЫЕ HEALTH CHECKS:
├─ Быстрые (< 1 секунда)
├─ Легковесные (минимум CPU/Memory)
├─ Специфичные (проверяют реальную функциональность)
├─ Детерминированные (стабильные результаты)
└─ Безопасные (не меняют состояние)

ПЛОХИЕ ПРАКТИКИ:
├─ Проверка внешних зависимостей в liveness
├─ Тяжелые операции (полная проверка БД)
├─ Недетерминированные тесты
├─ Сайд-эффекты при проверке
└─ Слишком агрессивные таймауты
```

**🏥 Health Check Endpoints design:**

```go
// Пример Go HTTP health checks
func healthHandler(w http.ResponseWriter, r *http.Request) {
    // Liveness: проверяем только внутреннее состояние
    if !app.IsAlive() {
        http.Error(w, "Service not alive", http.StatusServiceUnavailable)
        return
    }
    
    w.WriteHeader(http.StatusOK)
    json.NewEncoder(w).Encode(map[string]string{
        "status": "UP",
        "timestamp": time.Now().ISO8601(),
        "version": app.Version,
    })
}

func readinessHandler(w http.ResponseWriter, r *http.Request) {
    checks := map[string]string{}
    healthy := true
    
    // Database connectivity
    if err := db.Ping(); err != nil {
        checks["database"] = "DOWN: " + err.Error()
        healthy = false
    } else {
        checks["database"] = "UP"
    }
    
    // Cache connectivity  
    if err := cache.Ping(); err != nil {
        checks["cache"] = "DOWN: " + err.Error()
        healthy = false
    } else {
        checks["cache"] = "UP"
    }
    
    status := "UP"
    httpStatus := http.StatusOK
    if !healthy {
        status = "DOWN"
        httpStatus = http.StatusServiceUnavailable
    }
    
    w.WriteHeader(httpStatus)
    json.NewEncoder(w).Encode(map[string]interface{}{
        "status": status,
        "checks": checks,
        "timestamp": time.Now().ISO8601(),
    })
}
```

---

### 🚨 Graceful Shutdown Patterns

**⚰️ Правильное завершение приложений:**

```yaml
# Pod с graceful shutdown
apiVersion: v1
kind: Pod
spec:
  terminationGracePeriodSeconds: 60  # Время на graceful shutdown
  containers:
  - name: app
    image: myapp:latest
    
    # Preemptive scaling down
    lifecycle:
      preStop:
        exec:
          command:
          - /bin/sh
          - -c
          - |
            # Снять с load balancer
            curl -X POST http://localhost:8080/admin/stop-accepting-requests
            # Дождаться завершения текущих запросов
            sleep 15
            # Graceful shutdown
            kill -TERM 1
    
    # Readiness probe для исключения из LB
    readinessProbe:
      httpGet:
        path: /health/ready
        port: 8080
      periodSeconds: 1
      failureThreshold: 1  # Быстро исключить из LB
```

**🔄 Graceful Shutdown Implementation (Go):**
```go
package main

import (
    "context"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
)

func main() {
    server := &http.Server{Addr: ":8080"}
    
    // Graceful shutdown
    go func() {
        sigint := make(chan os.Signal, 1)
        signal.Notify(sigint, os.Interrupt, syscall.SIGTERM)
        <-sigint
        
        // Shutdown with timeout
        ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
        defer cancel()
        
        log.Println("Shutting down server...")
        if err := server.Shutdown(ctx); err != nil {
            log.Printf("Server shutdown error: %v", err)
        }
    }()
    
    // Start server
    if err := server.ListenAndServe(); err != http.ErrServerClosed {
        log.Printf("Server error: %v", err)
    }
    
    log.Println("Server stopped")
}
```

---

## 🎓 Заключение блока

После изучения этого блока вы должны понимать:

✅ **Архитектуру мониторинга** с Prometheus и Grafana  
✅ **Централизованное логирование** и structured logging  
✅ **Distributed tracing** и APM концепции  
✅ **Systematic troubleshooting** подходы  
✅ **Health checks** и graceful shutdown patterns  
✅ **Performance optimization** для production

**🎯 Следующий шаг:** Применить полученные знания для создания comprehensive observability stack в production-окружении.

---

## 📚 Практические задания

**🔬 Лабораторные работы:**

1. **Развернуть monitoring stack:**
   - Prometheus + Grafana + AlertManager
   - Настроить custom metrics для приложения
   - Создать дашборды для SLI/SLO

2. **Настроить централизованное логирование:**
   - ELK Stack или Loki + Grafana
   - Structured logging в приложении
   - Log parsing и alerting

3. **Внедрить distributed tracing:**
   - OpenTelemetry instrumentation
   - Jaeger deployment
   - Performance optimization по traces

4. **Создать comprehensive troubleshooting guide:**
   - Runbook для типичных проблем
   - Automation для диагностики
   - Emergency response procedures

---

## 📖 Дополнительные ресурсы

**Документация:**
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)

**Инструменты:**
- [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus)
- [Jaeger Operator](https://github.com/jaegertracing/jaeger-operator)
- [Fluentd](https://www.fluentd.org/) / [Fluent Bit](https://fluentbit.io/)

**Практика:**
- [Prometheus Monitoring Mixins](https://monitoring.mixins.dev/)
- [SRE Workbook](https://sre.google/workbook/table-of-contents/) от Googlelfdf