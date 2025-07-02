# 🏗️ Блок 4: Проектирование и архитектура мониторинга

---

## 📋 Обзор блока

**🎯 Главная цель:** Научиться создавать комплексные решения мониторинга для предприятий  
**⏱️ Продолжительность:** 4-5 недель  
**👥 Уровень:** Продвинутый  
**🔧 Предварительные требования:** Завершение блоков 1-3

---

### 🎖️ Ключевые компетенции, которые вы освоите:

```
┌─────────────────────────────────────────────────────────────┐
│ ✅ Enterprise архитектура мониторинга                        │
│ ✅ Масштабирование и производительность                      │
│ ✅ DevOps интеграция                                        │
│ ✅ Контейнеры и микросервисы                                │
└─────────────────────────────────────────────────────────────┘
```

---

## 📅 Глава 4.1: Enterprise архитектура мониторинга
### 🕐 Длительность: 8 дней

---

### 🎯 Цели обучения

По завершении этой главы вы будете уметь:
- Проектировать многоуровневую архитектуру мониторинга
- Создавать федеративные установки Prometheus
- Планировать disaster recovery для систем мониторинга
- Разрабатывать отказоустойчивые решения

---

### 📚 Теоретическая часть

#### 🏢 Принципы Enterprise архитектуры

```
         УРОВНИ ENTERPRISE МОНИТОРИНГА
    ─────────────────────────────────────────────
    │    Уровень 1: Бизнес метрики              │
    │    ┌─────────────────────────────────┐    │
    │    │ • Revenue per hour              │    │
    │    │ • User conversion rates         │    │
    │    │ • Customer satisfaction         │    │
    │    └─────────────────────────────────┘    │
    ─────────────────────────────────────────────
    │    Уровень 2: Application метрики         │
    │    ┌─────────────────────────────────┐    │
    │    │ • Response time                 │    │
    │    │ • Error rates                   │    │
    │    │ • Throughput                    │    │
    │    └─────────────────────────────────┘    │
    ─────────────────────────────────────────────
    │    Уровень 3: Infrastructure метрики      │
    │    ┌─────────────────────────────────┐    │
    │    │ • CPU, Memory, Disk             │    │
    │    │ • Network performance           │    │
    │    │ • Hardware health               │    │
    │    └─────────────────────────────────┘    │
    ─────────────────────────────────────────────
```

#### 🌐 Федеративная архитектура Prometheus

```
                    ФЕДЕРАТИВНАЯ УСТАНОВКА PROMETHEUS
    ───────────────────────────────────────────────────────────────────

    ┌─────────────────────────────────────────────────────────────────┐
    │                    ГЛОБАЛЬНЫЙ PROMETHEUS                        │
    │  ┌─────────────────────────────────────────────────────────┐    │
    │  │  • Агрегированные метрики                               │    │
    │  │  • Глобальные алерты                                   │    │
    │  │  │  • Долгосрочное хранение                            │    │
    │  └─────────────────────────────────────────────────────────┘    │
    └─────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    │               │               │
    ┌───────────────▼─────┐ ┌───────▼─────┐ ┌───────▼─────┐
    │  РЕГИОНАЛЬНЫЙ       │ │ РЕГИОНАЛЬНЫЙ │ │ РЕГИОНАЛЬНЫЙ │
    │  PROMETHEUS         │ │ PROMETHEUS   │ │ PROMETHEUS   │
    │  (Европа)           │ │ (Азия)       │ │ (США)        │
    └─────────────────────┘ └─────────────┘ └─────────────┘
             │                     │               │
        ┌────┼────┐           ┌────┼────┐     ┌────┼────┐
        │    │    │           │    │    │     │    │    │
       ▼    ▼    ▼           ▼    ▼    ▼     ▼    ▼    ▼
     Сервер1-3            Сервер4-6       Сервер7-9
```

---

### ⚙️ Архитектурные паттерны

#### 1️⃣ **Hub-and-Spoke модель**

```
                       ЦЕНТРАЛЬНЫЙ HUB
                    ┌─────────────────────┐
                    │   Central Storage   │
                    │   • InfluxDB        │
                    │   • Elasticsearch   │
                    │   • TimescaleDB     │
                    └─────────────────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
    ┌───▼───┐           ┌────▼────┐           ┌───▼───┐
    │Spoke 1│           │ Spoke 2 │           │Spoke 3│
    │DC East│           │DC West  │           │Cloud  │
    └───────┘           └─────────┘           └───────┘
```

#### 2️⃣ **Mesh архитектура**

```
    MESH АРХИТЕКТУРА ДЛЯ РАСПРЕДЕЛЕННОГО МОНИТОРИНГА
    ─────────────────────────────────────────────────
    
    Node A ←──────────→ Node B
      │  ╲              ╱  │
      │    ╲          ╱    │
      │      ╲      ╱      │
      │        ╲  ╱        │
      │          ╳          │
      │        ╱  ╲        │
      │      ╱      ╲      │
      │    ╱          ╲    │
      │  ╱              ╲  │
    Node D ←──────────→ Node C
    
    • Каждый узел может общаться с любым другим
    • Отказоустойчивость через redundancy
    • Автоматический failover
```

---

### 🛠️ Практические рекомендации

#### 📊 Планирование ресурсов

| Компонент | Метрики (на 1000 серверов) | Требования |
|-----------|---------------------------|------------|
| **Prometheus** | 500GB RAM, 16 CPU cores | SSD storage |
| **Grafana** | 8GB RAM, 4 CPU cores | Load balancer |
| **AlertManager** | 4GB RAM, 2 CPU cores | HA setup |
| **Storage** | 10TB+ долгосрочное | Backup strategy |

#### 🔒 Безопасность Enterprise уровня

```
    SECURITY LAYERS В ENTERPRISE МОНИТОРИНГЕ
    ─────────────────────────────────────────
    
    ┌─────────────────────────────────────────┐ ← Audit & Compliance
    │ ┌─────────────────────────────────────┐ │ ← Access Control
    │ │ ┌─────────────────────────────────┐ │ │ ← Encryption
    │ │ │ ┌─────────────────────────────┐ │ │ │ ← Network Security
    │ │ │ │      CORE MONITORING        │ │ │ │
    │ │ │ │        SYSTEM               │ │ │ │
    │ │ │ └─────────────────────────────┘ │ │ │
    │ │ └─────────────────────────────────┘ │ │
    │ └─────────────────────────────────────┘ │
    └─────────────────────────────────────────┘
```

---

### 💡 Практическое задание 4.1

**🎯 Задача:** Спроектировать архитектуру мониторинга для компании из 1000+ серверов

#### 📋 Требования к проекту:
- [ ] 3 дата-центра в разных регионах
- [ ] 1000+ физических и виртуальных серверов
- [ ] 500+ приложений и сервисов
- [ ] Требования к availability: 99.9%
- [ ] Retention период: 2 года
- [ ] Disaster recovery: RPO 1 час, RTO 4 часа

#### 📐 Deliverables:
1. **Архитектурная диаграмма** с компонентами и связями
2. **Capacity planning** с расчетом ресурсов
3. **High availability** план
4. **Disaster recovery** стратегия
5. **Security** план с ролями и доступами

---

## 📅 Глава 4.2: Масштабирование и производительность
### 🕐 Длительность: 8 дней

---

### 🎯 Цели обучения

По завершении этой главы вы будете уметь:
- Масштабировать Prometheus горизонтально
- Использовать Thanos для долгосрочного хранения
- Оптимизировать запросы PromQL
- Планировать ресурсы для больших систем

---

### 🚀 Горизонтальное масштабирование

#### 📈 Проблемы масштабирования Prometheus

```
    ОГРАНИЧЕНИЯ SINGLE-NODE PROMETHEUS
    ──────────────────────────────────────
    
    ┌─────────────────────────────────────┐
    │           PROMETHEUS                │
    │  ┌─────────────────────────────┐    │
    │  │      TSDB Storage           │    │ ← Ограничение по дисковому пространству
    │  │      (Local Disk)           │    │
    │  └─────────────────────────────┘    │
    │  ┌─────────────────────────────┐    │
    │  │      Memory (RAM)           │    │ ← Ограничение по памяти
    │  │      (Active series)        │    │
    │  └─────────────────────────────┘    │
    │  ┌─────────────────────────────┐    │
    │  │      CPU Usage              │    │ ← Ограничение по вычислениям
    │  │      (Query processing)     │    │
    │  └─────────────────────────────┘    │
    └─────────────────────────────────────┘
    
    ЛИМИТЫ:
    • ~10M активных метрик
    • ~100GB RAM
    • ~500 queries/sec
```

#### 🔄 Решения для масштабирования

```
    ВАРИАНТЫ МАСШТАБИРОВАНИЯ PROMETHEUS
    ──────────────────────────────────────
    
    1. ФУНКЦИОНАЛЬНОЕ РАЗДЕЛЕНИЕ
    ────────────────────────────
    Prometheus-1    Prometheus-2    Prometheus-3
    (Infrastructure)  (Applications)   (Business)
         │                │              │
         ▼                ▼              ▼
    Servers, Network   APIs, Services   KPIs, Metrics
    
    2. ГЕОГРАФИЧЕСКОЕ РАЗДЕЛЕНИЕ
    ────────────────────────────
    Prometheus-EU   Prometheus-US   Prometheus-ASIA
         │              │               │
         ▼              ▼               ▼
    European DC     American DC      Asian DC
    
    3. FEDERATED SETUP
    ─────────────────
           Global Prometheus
                  │
         ┌────────┼────────┐
         ▼        ▼        ▼
      Prom-1   Prom-2   Prom-3
```

---

### 🗄️ Thanos: Долгосрочное хранение

#### 🏗️ Архитектура Thanos

```
    THANOS ARCHITECTURE OVERVIEW
    ─────────────────────────────────────────────────────────────
    
    ┌─────────────────────────────────────────────────────────┐
    │                    THANOS QUERY                         │
    │              (Unified Query Interface)                  │
    └─────────────────┬───────────────────┬───────────────────┘
                      │                   │
    ┌─────────────────▼─────────────┐   ┌─▼───────────────────┐
    │        THANOS STORE           │   │   THANOS SIDECAR    │
    │     (Historical Data)         │   │  (Real-time Data)   │
    │                               │   │                     │
    │  ┌─────────────────────────┐  │   │ ┌─────────────────┐ │
    │  │    Object Storage       │  │   │ │   Prometheus    │ │
    │  │   • S3/GCS/Azure        │  │   │ │     TSDB        │ │
    │  │   • Long-term storage   │  │   │ └─────────────────┘ │
    │  └─────────────────────────┘  │   └─────────────────────┘
    └───────────────────────────────┘
                      │
    ┌─────────────────▼─────────────┐
    │       THANOS COMPACT          │
    │    (Data Compaction &         │
    │     Downsampling)             │
    └───────────────────────────────┘
```

#### 📊 Преимущества Thanos

```
    БЕНЕФИТЫ ИСПОЛЬЗОВАНИЯ THANOS
    ─────────────────────────────
    
    ✅ НЕОГРАНИЧЕННОЕ ХРАНЕНИЕ
    ┌────────────────────────────────┐
    │ Local: 2 недели                │ ──┐
    └────────────────────────────────┘   │
    ┌────────────────────────────────┐   ├──► THANOS
    │ S3: Годы истории               │ ──┘
    └────────────────────────────────┘
    
    ✅ GLOBAL VIEW
    Prometheus-1 ──┐
    Prometheus-2 ──┼──► Единый интерфейс запросов
    Prometheus-3 ──┘
    
    ✅ DOWNSAMPLING
    Raw data (5m) ──► 1h resolution ──► 1d resolution
    
    ✅ DEDUPLICATION
    Duplicate series ──► Unique series
```

---

### ⚡ Оптимизация PromQL

#### 🎯 Лучшие практики запросов

```promql
# ❌ ПЛОХО: Inefficient query
rate(http_requests_total[5m]) > 0.1

# ✅ ХОРОШО: More specific query
rate(http_requests_total{job="web-server", status!="404"}[5m]) > 0.1

# ❌ ПЛОХО: Expensive aggregation
sum(rate(http_requests_total[5m])) by (instance)

# ✅ ХОРОШО: Pre-aggregated metric
sum(rate(http_requests_total[5m])) by (job)
```

#### 📊 Performance Guidelines

| 🔍 **Аспект** | ⚠️ **Проблема** | ✅ **Решение** |
|---------------|-----------------|----------------|
| **Time Range** | `[1d]` в rate() | `[5m]` максимум |
| **Labels** | Без фильтров | Конкретные значения |
| **Aggregation** | `by (instance)` | `by (job)` |
| **Functions** | Nested functions | Простые выражения |

---

### 💡 Практическое задание 4.2

**🎯 Задача:** Настроить Thanos для федеративной установки Prometheus

#### 📋 Компоненты для развертывания:
- [ ] 3 Prometheus instance с Thanos Sidecar
- [ ] Thanos Query для unified interface
- [ ] Thanos Store для object storage
- [ ] Thanos Compact для downsamplings
- [ ] Object storage (MinIO/S3)

#### 📊 Метрики для мониторинга:
- Query latency и throughput
- Storage usage и compaction status
- Deduplication effectiveness
- Downsampling ratios

---

## 📅 Глава 4.3: DevOps интеграция
### 🕐 Длительность: 8 дней

---

### 🎯 Цели обучения

По завершении этой главы вы будете уметь:
- Использовать Infrastructure as Code для мониторинга
- Интегрировать мониторинг в CI/CD пайплайны
- Применять GitOps подход
- Автоматизировать создание дашбордов и алертов

---

### 🏗️ Infrastructure as Code

#### 🛠️ Terraform для мониторинга

```hcl
# Пример Terraform конфигурации для Prometheus
resource "kubernetes_deployment" "prometheus" {
  metadata {
    name = "prometheus"
    namespace = var.monitoring_namespace
  }
  
  spec {
    replicas = var.prometheus_replicas
    
    selector {
      match_labels = {
        app = "prometheus"
      }
    }
    
    template {
      metadata {
        labels = {
          app = "prometheus"
        }
      }
      
      spec {
        container {
          name  = "prometheus"
          image = "prom/prometheus:${var.prometheus_version}"
          
          resources {
            requests = {
              memory = "2Gi"
              cpu    = "1000m"
            }
            limits = {
              memory = "4Gi"
              cpu    = "2000m"
            }
          }
        }
      }
    }
  }
}
```

#### 📱 Ansible Playbooks

```yaml
---
- name: Deploy Monitoring Stack
  hosts: monitoring_servers
  become: yes
  
  vars:
    prometheus_version: "2.40.0"
    grafana_version: "9.3.0"
    
  tasks:
    - name: Create monitoring directories
      file:
        path: "{{ item }}"
        state: directory
        mode: '0755'
      loop:
        - /opt/prometheus
        - /opt/grafana
        - /var/lib/prometheus
        
    - name: Deploy Prometheus configuration
      template:
        src: prometheus.yml.j2
        dest: /opt/prometheus/prometheus.yml
        backup: yes
      notify: restart prometheus
      
    - name: Start Prometheus service
      docker_container:
        name: prometheus
        image: "prom/prometheus:{{ prometheus_version }}"
        ports:
          - "9090:9090"
        volumes:
          - "/opt/prometheus:/etc/prometheus"
          - "/var/lib/prometheus:/prometheus"
        restart_policy: always
```

---

### 🔄 CI/CD Integration

#### 🏗️ GitLab CI Pipeline

```yaml
# .gitlab-ci.yml для мониторинга
stages:
  - validate
  - deploy
  - test
  - promote

variables:
  PROMETHEUS_VERSION: "2.40.0"
  GRAFANA_VERSION: "9.3.0"

validate_config:
  stage: validate
  image: prom/prometheus:${PROMETHEUS_VERSION}
  script:
    - promtool check config prometheus/prometheus.yml
    - promtool check rules prometheus/rules/*.yml
  artifacts:
    reports:
      junit: validation_report.xml

deploy_staging:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl apply -f k8s/staging/ -n monitoring-staging
    - kubectl rollout status deployment/prometheus -n monitoring-staging
  environment:
    name: staging
    url: https://prometheus-staging.company.com
  only:
    - develop

monitoring_tests:
  stage: test
  image: python:3.9
  script:
    - pip install pytest requests
    - python -m pytest tests/monitoring/
  artifacts:
    reports:
      junit: test_results.xml

deploy_production:
  stage: promote
  image: bitnami/kubectl:latest
  script:
    - kubectl apply -f k8s/production/ -n monitoring
    - kubectl rollout status deployment/prometheus -n monitoring
  environment:
    name: production
    url: https://prometheus.company.com
  when: manual
  only:
    - main
```

#### 📊 Monitoring Deployments

```
    CI/CD МОНИТОРИНГ PIPELINE
    ─────────────────────────────────────────
    
    Code Push ──► GitLab CI ──► Deploy ──► Monitor
         │             │           │          │
         ▼             ▼           ▼          ▼
    ┌─────────┐ ┌─────────┐ ┌──────────┐ ┌─────────┐
    │ Git     │ │ Build   │ │ K8s      │ │ Grafana │
    │ Webhook │ │ & Test  │ │ Deploy   │ │ Alert   │
    └─────────┘ └─────────┘ └──────────┘ └─────────┘
         │             │           │          │
         ▼             ▼           ▼          ▼
    Auto-trigger   Unit Tests   Health Check  Dashboard
```

---

### 🤖 GitOps подход

#### 🔄 ArgoCD для мониторинга

```yaml
# ArgoCD Application для мониторинга
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: monitoring-stack
  namespace: argocd
spec:
  project: monitoring
  
  source:
    repoURL: https://github.com/company/monitoring-config
    targetRevision: main
    path: environments/production
    
    helm:
      valueFiles:
        - values-production.yaml
        
  destination:
    server: https://kubernetes.default.svc
    namespace: monitoring
    
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
      
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
```

#### 📂 GitOps структура репозитория

```
monitoring-gitops/
├── apps/
│   ├── prometheus/
│   │   ├── base/
│   │   └── overlays/
│   │       ├── staging/
│   │       └── production/
│   ├── grafana/
│   └── alertmanager/
├── environments/
│   ├── staging/
│   │   ├── kustomization.yaml
│   │   └── values.yaml
│   └── production/
│       ├── kustomization.yaml
│       └── values.yaml
└── charts/
    └── monitoring-stack/
```

---

### 💡 Практическое задание 4.3

**🎯 Задача:** Создать CI/CD пайплайн для автоматического развертывания мониторинга

#### 📋 Требования:
- [ ] Terraform для infrastructure provisioning
- [ ] Ansible для configuration management
- [ ] GitLab CI/CD pipeline
- [ ] Automated testing мониторинга
- [ ] GitOps workflow с ArgoCD
- [ ] Rollback механизм

#### 🔧 Компоненты пайплайна:
1. **Config validation** (promtool, yamllint)
2. **Infrastructure deployment** (Terraform)
3. **Application deployment** (Helm/Kustomize)
4. **Health checks** (automated tests)
5. **Monitoring integration** (deployment metrics)

---

## 📅 Глава 4.4: Контейнеры и микросервисы
### 🕐 Длительность: 8 дней

---

### 🎯 Цели обучения

По завершении этой главы вы будете уметь:
- Мониторить Docker контейнеры
- Настраивать полный мониторинг Kubernetes
- Работать с service mesh мониторингом
- Внедрять distributed tracing

---

### 🐳 Docker мониторинг

#### 📊 Основные метрики контейнеров

```
    КОНТЕЙНЕР МЕТРИКИ OVERVIEW
    ─────────────────────────────────────
    
    ┌─────────────────────────────────┐
    │           CONTAINER             │
    │  ┌─────────────────────────┐    │
    │  │      CPU METRICS        │    │ ← cpu_usage_percent
    │  │  • Usage percentage     │    │   cpu_throttling_data
    │  │  • Throttling data      │    │
    │  └─────────────────────────┘    │
    │  ┌─────────────────────────┐    │
    │  │     MEMORY METRICS      │    │ ← memory_usage_bytes
    │  │  • Usage bytes          │    │   memory_limit_bytes
    │  │  • Limit bytes          │    │   memory_cache
    │  │  • Cache usage          │    │
    │  └─────────────────────────┘    │
    │  ┌─────────────────────────┐    │
    │  │     NETWORK METRICS     │    │ ← network_rx_bytes
    │  │  • RX/TX bytes          │    │   network_tx_bytes
    │  │  • Packet counts        │    │   network_errors
    │  └─────────────────────────┘    │
    │  ┌─────────────────────────┐    │
    │  │      DISK METRICS       │    │ ← disk_usage_bytes
    │  │  • Read/Write bytes     │    │   disk_io_ops
    │  │  • IO operations        │    │
    │  └─────────────────────────┘    │
    └─────────────────────────────────┘
```

#### 🔧 cAdvisor конфигурация

```yaml
# Docker Compose для cAdvisor
version: '3.8'
services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    privileged: true
    devices:
      - /dev/kmsg
    command:
      - '--housekeeping_interval=10s'
      - '--docker_only=true'
      - '--disable_metrics=disk,network,tcp,udp,percpu,sched,process'
```

---

### ☸️ Kubernetes мониторинг

#### 🏗️ Полная архитектура K8s мониторинга

```
    KUBERNETES MONITORING STACK
    ─────────────────────────────────────────────────────────
    
    ┌─────────────────────────────────────────────────────┐
    │                   GRAFANA                           │
    │              (Visualization)                        │
    └─────────────────┬───────────────────────────────────┘
                      │
    ┌─────────────────▼───────────────────────────────────┐
    │                PROMETHEUS                           │
    │             (Metrics Storage)                       │
    └┬────────────┬─────────────┬─────────────┬──────────┘
     │            │             │             │
     ▼            ▼             ▼             ▼
┌─────────┐ ┌──────────┐ ┌─────────────┐ ┌─────────────┐
│ Node    │ │ kube-    │ │ Application │ │ Custom      │
│Exporter │ │state-    │ │ Exporters   │ │ Exporters   │
│         │ │metrics   │ │             │ │             │
└─────────┘ └──────────┘ └─────────────┘ └─────────────┘
     │            │             │             │
     ▼            ▼             ▼             ▼
┌─────────┐ ┌──────────┐ ┌─────────────┐ ┌─────────────┐
│Hardware │ │K8s API   │ │Pods/        │ │Business     │
│Resources│ │Objects   │ │Services     │ │Metrics      │
└─────────┘ └──────────┘ └─────────────┘ └─────────────┘
```

#### 📊 Ключевые компоненты

```yaml
# kube-state-metrics Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-state-metrics
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kube-state-metrics
  template:
    metadata:
      labels:
        app: kube-state-metrics
    spec:
      containers:
      - name: kube-state-metrics
        image: k8s.gcr.io/kube-state-metrics/kube-state-metrics:v2.7.0
        ports:
        - containerPort: 8080
          name: http-metrics
        - containerPort: 8081
          name: telemetry
        resources:
          requests:
            memory: 100Mi
            cpu: 100m
          limits:
            memory: 200Mi
            cpu: 200m
```

#### 🎯 Критические метрики K8s

| **Категория** | **Метрика** | **Описание** |
|---------------|-------------|--------------|
| **Cluster** | `kube_node_status_ready` | Статус готовности нод |
| **Pods** | `kube_pod_container_status_restarts_total` | Перезапуски контейнеров |
| **Resources** | `kube_pod_container_resource_requests` | Запрошенные ресурсы |
| **Network** | `container_network_receive_bytes_total` | Сетевой трафик |

---

### 🕸️ Service Mesh мониторинг

#### 🔄 Istio мониторинг

```
    ISTIO SERVICE MESH OBSERVABILITY
    ─────────────────────────────────────────────
    
    ┌─────────────────────────────────────────┐
    │                JAEGER                   │ ← Distributed Tracing
    │           (Trace Analysis)              │
    └─────────────────────────────────────────┘
    
    ┌─────────────────────────────────────────┐
    │               GRAFANA                   │ ← Metrics Visualization
    │          (Service Dashboards)           │
    └─────────────────────────────────────────┘
    
    ┌─────────────────────────────────────────┐
    │              PROMETHEUS                 │ ← Metrics Storage
    │           (Istio Metrics)               │
    └─────────────────────────────────────────┘
                          │
    ┌─────────────────────▼───────────────────┐
    │             ENVOY PROXIES               │ ← Data Collection
    │        (Sidecar containers)             │
    │                                         │
    │  Service A ←──── Service B ←──── Service C
    │     │               │              │
    │   Envoy           Envoy          Envoy
    └─────────────────────────────────────────┘
```

#### 📊 Golden Signals для микросервисов

```promql
# Request Rate (запросы в секунду)
sum(rate(istio_requests_total[5m])) by (destination_service_name)

# Error Rate (процент ошибок)
sum(rate(istio_requests_total{response_code!~"2.."}[5m])) 
/ 
sum(rate(istio_requests_total[5m])) * 100

# Request Duration (latency)
histogram_quantile(0.95, 
  sum(rate(istio_request_duration_milliseconds_bucket[5m])) 
  by (destination_service_name, le)
)

# Saturation (утилизация ресурсов)
rate(container_cpu_usage_seconds_total[5m]) / 
on(pod) kube_pod_container_resource_limits_cpu_cores
```

---

### 🔍 Distributed Tracing

#### 📈 Jaeger архитектура

```
    JAEGER DISTRIBUTED TRACING ARCHITECTURE
    ────────────────────────────────────────────────
    
    ┌─────────────────────────────────────────────┐
    │              JAEGER UI                      │ ← Query Interface
    │          (Trace Visualization)              │
    └─────────────────────────────────────────────┘
                          │
    ┌─────────────────────▼───────────────────────┐
    │             JAEGER QUERY                    │ ← Query Service
    │            (API Gateway)                    │
    └─────────────────────────────────────────────┘
                          │
    ┌─────────────────────▼───────────────────────┐
    │            STORAGE BACKEND                  │ ← Trace Storage
    │      (Elasticsearch/Cassandra)              │
    └─────────────────────────────────────────────┘
                          │
    ┌─────────────────────▼───────────────────────┐
    │           JAEGER COLLECTOR                  │ ← Trace Collection
    │          (Span Processing)                  │
    └─────────────────────────────────────────────┘
                          │
    ┌─────────────────────▼───────────────────────┐
    │            JAEGER AGENT                     │ ← Local Agent
    │         (Buffer & Forward)                  │
    └─────────────────────────────────────────────┘
                          │
    ┌─────────────────────▼───────────────────────┐
    │            APPLICATION                      │ ← Instrumentation
    │         (Tracing SDKs)                      │
    └─────────────────────────────────────────────┘
```

#### 🏷️ Span tags и logs

```json
{
  "traceID": "7af7a2ca57",
  "spanID": "1ed38d9",
  "operationName": "HTTP GET /api/users",
  "startTime": 1609459200000000,
  "duration": 150000,
  "tags": {
    "http.method": "GET",
    "http.url": "/api/users",
    "http.status_code": 200,
    "component": "http",
    "span.kind": "server"
  },
  "logs": [
    {
      "timestamp": 1609459200050000,
      "fields": {
        "event": "SQL query started",
        "query": "SELECT * FROM users"
      }
    }
  ]
}
```

---

### 💡 Практическое задание 4.4

**🎯 Задача:** Настроить полный мониторинг Kubernetes кластера

#### 📋 Компоненты для развертывания:
- [ ] Prometheus Operator
- [ ] Node Exporter DaemonSet
- [ ] kube-state-metrics
- [ ] Grafana с готовыми дашбордами
- [ ] AlertManager с правилами
- [ ] Jaeger для distributed tracing
- [ ] Service mesh (Istio) мониторинг

#### 🎯 Дашборды для создания:
1. **Cluster Overview** - общее состояние кластера
2. **Node Dashboard** - мониторинг нод
3. **Pod Dashboard** - состояние подов
4. **Service Mesh** - метрики Istio
5. **Application Performance** - APM метрики

#### 📊 Алерты для настройки:
- Node NotReady
- High CPU/Memory usage
- Pod CrashLooping
- High error rate в service mesh
- Storage space низкий

---

## 🏆 Итоговая оценка блока 4

### ✅ Чек-лист освоенных навыков:

```
┌─────────────────────────────────────────────────────────┐
│ □ Проектирование enterprise архитектуры мониторинга     │
│ □ Настройка федеративных установок Prometheus           │
│ □ Планирование disaster recovery                        │
│ □ Горизонтальное масштабирование систем мониторинга     │
│ □ Использование Thanos для долгосрочного хранения       │
│ □ Оптимизация PromQL запросов                          │
│ □ Infrastructure as Code для мониторинга                │
│ □ CI/CD интеграция и GitOps workflow                   │
│ □ Полный мониторинг Kubernetes кластера                │
│ □ Service mesh мониторинг и distributed tracing        │
└─────────────────────────────────────────────────────────┘
```

### 📊 Критерии оценки:

| **Уровень** | **Критерии** | **% выполнения** |
|-------------|--------------|------------------|
| **🥉 Базовый** | Понимание концепций enterprise архитектуры | 60-70% |
| **🥈 Продвинутый** | Самостоятельная настройка масштабируемых решений | 70-85% |
| **🥇 Экспертный** | Проектирование комплексных архитектур с нуля | 85-100% |

---

## 📚 Дополнительные ресурсы

### 📖 Обязательная литература:
- "Prometheus: Up & Running" - Brian Brazil
- "Kubernetes Patterns" - Bilgin Ibryam
- "Building Microservices" - Sam Newman

### 🔗 Полезные ссылки:
- [Prometheus Federation Guide](https://prometheus.io/docs/prometheus/latest/federation/)
- [Thanos Documentation](https://thanos.io/tip/thanos/quick-tutorial.md)
- [Kubernetes Monitoring Best Practices](https://kubernetes.io/docs/concepts/cluster-administration/system-metrics/)

### 🎓 Сертификации:
- Prometheus Certified Associate (PCA)
- Certified Kubernetes Administrator (CKA)
- Istio Certified Associate (ICA)

---

**🎯 Следующий шаг:** Переход к Блоку 5 - "Анализ данных и оптимизация"