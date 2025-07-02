# 📊 Блок 3: Практическая настройка мониторинга

---

**🎯 Цель блока:** Научиться устанавливать и настраивать системы мониторинга  
**⏱️ Длительность:** 5-6 недель  
**🏆 Результат:** Полнофункциональная система мониторинга инфраструктуры  

---

## 🔧 Глава 3.1: Prometheus + Grafana с нуля (10 дней)

### 📋 Обзор главы

```
День 1-2: Установка и базовая настройка Prometheus
День 3-4: Конфигурация targets и service discovery  
День 5-6: Установка и настройка Grafana
День 7-8: Создание первых дашбордов
День 9-10: Настройка пользователей и разрешений
```

---

### 🚀 День 1-2: Установка и базовая настройка Prometheus

#### Архитектура Prometheus

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Prometheus    │────│   Exporters     │────│    Targets      │
│     Server      │    │   (Метрики)     │    │   (Сервисы)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │              ┌─────────────────┐             │
         └──────────────│  AlertManager   │─────────────┘
                        │  (Уведомления)  │
                        └─────────────────┘
                                 │
                        ┌─────────────────┐
                        │     Grafana     │
                        │ (Визуализация)  │
                        └─────────────────┘
```

#### Установка Prometheus

**На Ubuntu/Debian:**
```bash
# Создание пользователя
sudo useradd --no-create-home --shell /bin/false prometheus

# Создание директорий
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

# Загрузка и установка
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
tar xvf prometheus-2.45.0.linux-amd64.tar.gz
sudo cp prometheus-2.45.0.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus-2.45.0.linux-amd64/promtool /usr/local/bin/

# Настройка прав
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
```

#### Базовый конфигурационный файл

**Файл: `/etc/prometheus/prometheus.yml`**
```yaml
global:
  scrape_interval: 15s        # Интервал сбора метрик
  evaluation_interval: 15s    # Интервал оценки правил

rule_files:
  # "first_rules.yml"
  # "second_rules.yml"

scrape_configs:
  # Мониторинг самого Prometheus
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  # Мониторинг Node Exporter
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

#### Systemd сервис

**Файл: `/etc/systemd/system/prometheus.service`**
```ini
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9090 \
    --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
```

---

### ⚙️ День 3-4: Конфигурация targets и service discovery

#### Статическая конфигурация

```yaml
scrape_configs:
  - job_name: 'web-servers'
    static_configs:
      - targets: 
          - 'web01:9100'
          - 'web02:9100'
          - 'web03:9100'
        labels:
          environment: 'production'
          team: 'web'
```

#### File-based Service Discovery

```
📁 /etc/prometheus/
├── prometheus.yml
├── targets/
│   ├── web-servers.yml
│   ├── databases.yml
│   └── monitoring.yml
```

**Файл: `/etc/prometheus/targets/web-servers.yml`**
```yaml
- targets:
    - 'web01.example.com:9100'
    - 'web02.example.com:9100'
  labels:
    environment: 'production'
    service: 'web'
    datacenter: 'dc1'

- targets:
    - 'web03.example.com:9100'
    - 'web04.example.com:9100'
  labels:
    environment: 'staging'
    service: 'web'
    datacenter: 'dc2'
```

**Конфигурация в prometheus.yml:**
```yaml
scrape_configs:
  - job_name: 'web-servers'
    file_sd_configs:
      - files:
          - '/etc/prometheus/targets/web-servers.yml'
        refresh_interval: 30s
```

#### Схема Service Discovery

```
┌─────────────────────────────────────────────────────────┐
│                    Service Discovery                    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │   Static    │  │ File-based  │  │  Consul/    │     │
│  │   Config    │  │ Discovery   │  │  K8s/AWS    │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│         │                │                │             │
│         └────────────────┼────────────────┘             │
│                          │                              │
│                   ┌─────────────┐                       │
│                   │  Prometheus │                       │
│                   │   Targets   │                       │
│                   └─────────────┘                       │
└─────────────────────────────────────────────────────────┘
```

---

### 📈 День 5-6: Установка и настройка Grafana

#### Установка Grafana

```bash
# Добавление репозитория
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

# Установка
sudo apt-get update
sudo apt-get install grafana

# Запуск сервиса
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

#### Первичная настройка

```
🌐 URL: http://localhost:3000
👤 Логин: admin
🔑 Пароль: admin (требует смены при первом входе)
```

#### Добавление Prometheus как источника данных

```
Настройки → Data Sources → Add data source → Prometheus

┌─────────────────────────────────────────────┐
│              Prometheus Settings            │
├─────────────────────────────────────────────┤
│ Name: Prometheus                            │
│ URL: http://localhost:9090                  │
│ Access: Server (default)                    │
│ HTTP Method: GET                            │
└─────────────────────────────────────────────┘
```

---

### 📊 День 7-8: Создание первых дашбордов

#### Структура дашборда

```
┌─────────────────────────────────────────────────────────┐
│                    Dashboard Header                     │
│  📊 Server Monitoring    🔄 Refresh: 30s  📅 Last 1h  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │     CPU     │  │   Memory    │  │    Disk     │     │
│  │   Usage %   │  │   Usage %   │  │   Usage %   │     │
│  │     78%     │  │     45%     │  │     23%     │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│                                                         │
│  ┌─────────────────────────────────────────────────────┐ │
│  │            CPU Usage Over Time                      │ │
│  │  100% ┌─────────────────────────────────────────┐  │ │
│  │       │    ╭─╮                                  │  │ │
│  │   50% │   ╱   ╲    ╭─╮                         │  │ │
│  │       │  ╱     ╲  ╱   ╲                        │  │ │
│  │    0% └─────────────────────────────────────────┘  │ │
│  │         12:00   12:15   12:30   12:45   13:00      │ │
│  └─────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

#### Основные PromQL запросы для дашборда

**CPU Usage:**
```promql
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

**Memory Usage:**
```promql
100 * (1 - ((node_memory_MemAvailable_bytes) / (node_memory_MemTotal_bytes)))
```

**Disk Usage:**
```promql
100 - ((node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100)
```

**Network Traffic:**
```promql
irate(node_network_receive_bytes_total{device!="lo"}[5m])
```

#### Типы панелей и их применение

```
┌─────────────────────────────────────────────────────────┐
│                    Типы панелей                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  📊 Graph Panel      │  Временные ряды данных           │
│  📈 Stat Panel       │  Текущие значения метрик         │
│  📋 Table Panel      │  Табличное представление         │
│  🗺️  Worldmap Panel  │  Географическое распределение    │
│  📶 Bar Gauge        │  Индикаторы уровня              │
│  🎯 Gauge Panel      │  Круговые индикаторы            │
│  📝 Text Panel       │  Документация и описания        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

### 👥 День 9-10: Настройка пользователей и разрешений

#### Организационная структура

```
┌─────────────────────────────────────────────────────────┐
│                 Grafana Organization                    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  👑 Admin           │  Полные права                     │
│  👔 Editor          │  Создание/изменение дашбордов     │
│  👁️  Viewer          │  Только просмотр                  │
│                                                         │
│  🏢 Teams:                                              │
│    ├── Infrastructure Team                              │
│    ├── Development Team                                 │
│    ├── QA Team                                          │
│    └── Management                                       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### Настройка LDAP интеграции

**Файл: `/etc/grafana/ldap.toml`**
```toml
[[servers]]
host = "ldap.company.com"
port = 389
use_ssl = false
start_tls = false
ssl_skip_verify = false

bind_dn = "cn=admin,dc=company,dc=com"
bind_password = 'password'

search_filter = "(cn=%s)"
search_base_dns = ["dc=company,dc=com"]

[servers.attributes]
name = "givenName"
surname = "sn"
username = "cn"
member_of = "memberOf"
email =  "email"

[[servers.group_mappings]]
group_dn = "cn=admins,ou=groups,dc=company,dc=com"
org_role = "Admin"

[[servers.group_mappings]]
group_dn = "cn=developers,ou=groups,dc=company,dc=com"
org_role = "Editor"
```

---

## 🖥️ Глава 3.2: Мониторинг инфраструктуры (8 дней)

### 📋 План главы

```
День 1-2: Node Exporter для серверов Linux/Windows
День 3-4: SNMP мониторинг сетевого оборудования
День 5-6: Мониторинг виртуализации
День 7-8: Blackbox monitoring
```

---

### 🐧 День 1-2: Node Exporter для серверов

#### Установка Node Exporter

```bash
# Загрузка и установка
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.0/node_exporter-1.6.0.linux-amd64.tar.gz
tar xvf node_exporter-1.6.0.linux-amd64.tar.gz
sudo cp node_exporter-1.6.0.linux-amd64/node_exporter /usr/local/bin/

# Systemd сервис
sudo useradd --no-create-home --shell /bin/false node_exporter
```

#### Метрики Node Exporter

```
┌─────────────────────────────────────────────────────────┐
│                 Node Exporter Metrics                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  🖥️  CPU Metrics        │  node_cpu_seconds_total       │
│  💾 Memory Metrics      │  node_memory_*                │
│  💿 Disk Metrics        │  node_disk_*                  │
│  🌐 Network Metrics     │  node_network_*               │
│  📁 Filesystem Metrics  │  node_filesystem_*            │
│  🔥 Hardware Metrics    │  node_hwmon_*                 │
│  ⏰ Time Metrics        │  node_time_*                  │
│  📊 System Load        │  node_load*                   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### Windows Exporter

```powershell
# Загрузка Windows Exporter
Invoke-WebRequest -Uri "https://github.com/prometheus-community/windows_exporter/releases/download/v0.23.1/windows_exporter-0.23.1-amd64.msi" -OutFile "windows_exporter.msi"

# Установка как сервис
msiexec /i windows_exporter.msi ENABLED_COLLECTORS="cpu,cs,logical_disk,net,os,service,system,textfile" /quiet
```

---

### 🌐 День 3-4: SNMP мониторинг

#### Архитектура SNMP мониторинга

```
┌─────────────────────────────────────────────────────────┐
│              SNMP Monitoring Architecture               │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐ │
│  │  Prometheus │────│    SNMP     │────│   Network   │ │
│  │   Server    │    │  Exporter   │    │  Equipment  │ │
│  └─────────────┘    └─────────────┘    └─────────────┘ │
│                             │                          │
│                     ┌─────────────┐                    │
│                     │ snmp.yml    │                    │
│                     │ (MIB Config)│                    │
│                     └─────────────┘                    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### Установка SNMP Exporter

```bash
# Загрузка и установка
wget https://github.com/prometheus/snmp_exporter/releases/download/v0.21.0/snmp_exporter-0.21.0.linux-amd64.tar.gz
tar xvf snmp_exporter-0.21.0.linux-amd64.tar.gz
sudo cp snmp_exporter-0.21.0.linux-amd64/snmp_exporter /usr/local/bin/
```

#### Конфигурация для сетевого оборудования

**Файл: `/etc/prometheus/snmp.yml`**
```yaml
auths:
  public_v2:
    community: public
    security_level: noAuthNoPriv
    auth_protocol: MD5
    priv_protocol: DES
    version: 2

modules:
  # Cisco IOS
  cisco_ios:
    walk:
      - 1.3.6.1.2.1.1.3.0      # sysUpTime
      - 1.3.6.1.2.1.2.2.1.10   # ifInOctets
      - 1.3.6.1.2.1.2.2.1.16   # ifOutOctets
      - 1.3.6.1.4.1.9.9.13.1.3.1.3  # cpmCPUTotal5min
    
  # Generic Linux
  linux:
    walk:
      - 1.3.6.1.4.1.2021.10.1.3.1  # laLoad.1
      - 1.3.6.1.4.1.2021.4.5.0     # memTotalReal
      - 1.3.6.1.4.1.2021.4.6.0     # memAvailReal
```

---

### ☁️ День 5-6: Мониторинг виртуализации

#### VMware vSphere мониторинг

```yaml
# vmware_exporter конфигурация
scrape_configs:
  - job_name: 'vmware'
    static_configs:
      - targets: ['vcenter.company.com']
    metrics_path: '/metrics'
    params:
      module: [default]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: vmware-exporter:9272
```

#### Метрики виртуализации

```
┌─────────────────────────────────────────────────────────┐
│               Virtualization Metrics                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  🏢 Datacenter Level    │  Общие ресурсы дата-центра   │
│  🏗️  Cluster Level      │  Ресурсы кластера             │
│  🖥️  Host Level         │  Физические хосты            │
│  💻 VM Level           │  Виртуальные машины           │
│  💾 Datastore Level    │  Хранилища данных            │
│  🌐 Network Level      │  Виртуальные сети            │
│                                                         │
│  📊 Key Metrics:                                        │
│    ├── CPU Usage & Ready Time                           │
│    ├── Memory Usage & Ballooning                        │
│    ├── Storage Latency & IOPS                          │
│    ├── Network Throughput                               │
│    └── Resource Contention                              │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

### 🔍 День 7-8: Blackbox monitoring

#### Типы Blackbox проверок

```
┌─────────────────────────────────────────────────────────┐
│                 Blackbox Monitoring                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  🌐 HTTP/HTTPS         │  Веб-сервисы и API            │
│  📧 ICMP Ping          │  Доступность хостов            │
│  🔌 TCP/UDP            │  Порты и сервисы              │
│  📨 DNS                │  Разрешение имен              │
│  📮 SMTP               │  Почтовые серверы             │
│                                                         │
│  ✅ Проверяется:                                        │
│    ├── Доступность сервиса                             │
│    ├── Время отклика                                    │
│    ├── SSL сертификаты                                 │
│    ├── Содержимое страниц                              │
│    └── Коды состояния HTTP                             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### Конфигурация Blackbox Exporter

**Файл: `/etc/blackbox/blackbox.yml`**
```yaml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_http_versions: ["HTTP/1.1", "HTTP/2.0"]
      valid_status_codes: [200]
      method: GET
      fail_if_ssl: false
      fail_if_not_ssl: false
  
  http_post_2xx:
    prober: http
    timeout: 5s
    http:
      method: POST
      headers:
        Content-Type: application/json
      body: '{"test": "data"}'
  
  tcp_connect:
    prober: tcp
    timeout: 5s
  
  ping:
    prober: icmp
    timeout: 5s
    icmp:
      preferred_ip_protocol: ip4
```

---

## 🗄️ Глава 3.3: Мониторинг баз данных и приложений (8 дней)

### 📋 План главы

```
День 1-2: MySQL/PostgreSQL мониторинг
День 3-4: NoSQL и кэш-системы  
День 5-6: Веб-серверы и приложения
День 7-8: Очереди сообщений
```

---

### 🐬 День 1-2: MySQL/PostgreSQL мониторинг

#### MySQL Exporter Setup

```sql
-- Создание пользователя для мониторинга
CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'password';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';
GRANT SELECT ON performance_schema.* TO 'exporter'@'localhost';
FLUSH PRIVILEGES;
```

#### Ключевые метрики баз данных

```
┌─────────────────────────────────────────────────────────┐
│                Database Monitoring                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  🔄 Connections        │  Активные подключения          │
│  ⚡ Query Performance  │  Время выполнения запросов     │
│  🔒 Locks & Waits      │  Блокировки и ожидания         │
│  💾 Buffer Pool        │  Использование кэша            │
│  📊 Slow Queries       │  Медленные запросы             │
│  🔄 Replication        │  Состояние репликации          │
│  💿 Disk I/O           │  Дисковые операции             │
│  📈 Throughput         │  Пропускная способность        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### PostgreSQL мониторинг

```yaml
# PostgreSQL Exporter конфигурация
scrape_configs:
  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']
    scrape_interval: 30s
```

---

### 📊 День 3-4: NoSQL и кэш-системы

#### Redis мониторинг

```
┌─────────────────────────────────────────────────────────┐
│                  Redis Monitoring                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  📊 Memory Usage       │  Использование памяти          │
│  🔑 Keys Count         │  Количество ключей             │
│  ⚡ Commands/sec        │  Команды в секунду             │
│  🔄 Hit Rate           │  Коэффициент попадания          │
│  🌐 Connected Clients  │  Подключенные клиенты          │
│  💾 Persistence        │  RDB/AOF статус               │
│  🔄 Replication        │  Master/Slave синхронизация    │
│  🔥 Evicted Keys       │  Вытесненные ключи             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### MongoDB мониторинг

```javascript
// MongoDB custom metrics
db.runCommand({serverStatus: 1})

// Основные метрики:
// - connections.current
// - opcounters.insert/query/update/delete
// - mem.resident
// - locks.Global.acquireCount
```

---

### 🌐 День 5-6: Веб-серверы

#### Nginx мониторинг

```nginx
# nginx.conf - включение статистики
server {
    listen 8080;
    location /nginx_status {
        stub_status on;
        access_log off;
        allow 127.0.0.1;
        deny all;
    }
}
```

#### Apache мониторинг

```apache
# httpd.conf - включение mod_status
<Location "/server-status">
    SetHandler server-status
    Require local
</Location>

<Location "/server-info">
    SetHandler server-info
    Require local
</Location>
```

---

### 📨 День 7-8: Очереди сообщений

#### RabbitMQ мониторинг

```
┌─────────────────────────────────────────────────────────┐
│                RabbitMQ Monitoring                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  📨 Queue Depth        │  Размер очередей               │
│  ⚡ Message Rate        │  Скорость обработки            │
│  🔄 Consumer Count     │  Количество потребителей       │
│  💾 Memory Usage       │  Использование памяти          │
│  🌐 Connections        │  Подключения                   │
│  📊 Exchanges          │  Статистика обменников         │
│  🔒 Node Status        │  Состояние узлов кластера     │
│  ⏰ Message TTL        │  Время жизни сообщений         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 🚨 Глава 3.4: Алертинг и уведомления (8 дней)

### 📋 План главы

```
День 1-2: Настройка AlertManager
День 3-4: Правила алертинга
День 5-6: Интеграция с системами уведомлений
День 7-8: Эскалация и оптимизация
```

---

### ⚠️ День 1-2: Настройка AlertManager

#### Архитектура алертинга

```
┌─────────────────────────────────────────────────────────┐
│                Alert Flow Architecture                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐ │
│  │ Prometheus  │────│ AlertManager│────│ Notification│ │
│  │   Rules     │    │  Routing    │    │  Channels   │ │
│  └─────────────┘    └─────────────┘    └─────────────┘ │
│         │                   │                   │      │
│    ┌──────────┐         ┌─────────┐        ┌──────────┐│
│    │ Evaluate │         │ Group & │        │  Email   ││
│    │ Metrics  │         │ Throttle│        │  Slack   ││
│    └──────────┘         └─────────┘        │ PagerDuty││
│                                            └──────────┘│
└─────────────────────────────────────────────────────────┘
```

#### Конфигурация AlertManager

**Файл: `/etc/alertmanager/alertmanager.yml`**
```yaml
global:
  smtp_smarthost: 'localhost:587'
  smtp_from: 'alerts@company.com'

route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'default-receiver'
  routes:
  - match:
      severity: critical
    receiver: 'critical-alerts'
  - match:
      team: database
    receiver: 'dba-team'

receivers:
- name: 'default-receiver'
  email_configs:
  - to: 'admin@company.com'
    subject: 'Alert: {{ .GroupLabels.alertname }}'
    body: |
      {{ range .Alerts }}
      Alert: {{ .Annotations.summary }}
      Description: {{ .Annotations.description }}
      {{ end }}

- name: 'critical-alerts'
  slack_configs:
  - api_url: 'https://hooks.slack.com/services/...'
    channel: '#critical-alerts'
    title: 'Critical Alert'
    text: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
```

---

### 📏 День 3-4: Правила алертинга

#### Структура правил алертинга

**Файл: `/etc/prometheus/alert_rules.yml`**
```yaml
groups:
- name: infrastructure.rules
  rules:
  - alert: HighCPUUsage
    expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 5m
    labels:
      severity: warning
      team: infrastructure
    annotations:
      summary: "High CPU usage on {{ $labels.instance }}"
      description: "CPU usage is above 80% for more than 5 minutes. Current value: {{ $value }}%"

  - alert: LowDiskSpace
    expr: (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 < 10
    for: 2m
    labels:
      severity: critical
      team: infrastructure
    annotations:
      summary: "Low disk space on {{ $labels.instance }}"
      description: "Disk space is below 10%. Current value: {{ $value }}%"

  - alert: ServiceDown
    expr: up == 0
    for: 1m
    labels:
      severity: critical
      team: platform
    annotations:
      summary: "Service {{ $labels.job }} is down"
      description: "Service has been down for more than 1 minute"
```

#### Категории алертов

```
┌─────────────────────────────────────────────────────────┐
│                  Alert Categories                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  🔴 Critical         │  Требует немедленного действия   │
│     • Service Down   │  • Полная недоступность          │
│     • Data Loss      │  • Критическая ошибка            │
│                                                         │
│  🟡 Warning          │  Требует внимания                │
│     • High Load      │  • Деградация производительности │
│     • Disk Space     │  • Приближение к лимитам         │
│                                                         │
│  🔵 Info             │  Информационные уведомления      │
│     • Deployment     │  • Изменения конфигурации        │
│     • Maintenance    │  • Плановые работы               │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

### 📱 День 5-6: Интеграция с системами уведомлений

#### Каналы уведомлений

```
┌─────────────────────────────────────────────────────────┐
│              Notification Channels                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  📧 Email            │  Детальные отчеты               │
│  💬 Slack/Teams      │  Командные уведомления          │
│  📱 SMS/Push         │  Критические алерты             │
│  📞 PagerDuty        │  Эскалация и дежурства          │
│  🔗 Webhooks         │  Интеграция с ITSM              │
│  📊 Custom API       │  Собственные системы            │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### Slack интеграция

```yaml
slack_configs:
- api_url: 'https://hooks.slack.com/services/...'
  channel: '#alerts'
  username: 'AlertManager'
  icon_emoji: ':warning:'
  title: 'Alert: {{ .GroupLabels.alertname }}'
  text: |
    {{ range .Alerts }}
    *Alert:* {{ .Annotations.summary }}
    *Severity:* {{ .Labels.severity }}
    *Instance:* {{ .Labels.instance }}
    *Description:* {{ .Annotations.description }}
    {{ end }}
  actions:
  - type: button
    text: 'Runbook :notebook:'
    url: '{{ (index .Alerts 0).Annotations.runbook_url }}'
  - type: button
    text: 'Silence :no_bell:'
    url: '{{ .SilenceURL }}'
```

---

### 🔄 День 7-8: Эскалация и оптимизация

#### Схема эскалации

```
┌─────────────────────────────────────────────────────────┐
│                 Escalation Matrix                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Level 1 (0-15 min)  │  Team Slack Channel             │
│                      │  Email to team lead             │
│                                                         │
│  Level 2 (15-30 min) │  SMS to on-call engineer        │
│                      │  PagerDuty notification         │
│                                                         │
│  Level 3 (30-60 min) │  Phone call to manager          │
│                      │  Incident bridge activation     │
│                                                         │
│  Level 4 (60+ min)   │  Executive notification         │
│                      │  External vendor escalation     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### Оптимизация алертов

```yaml
# Inhibition rules - подавление дублирующих алертов
inhibit_rules:
- source_match:
    severity: 'critical'
  target_match:
    severity: 'warning'
  equal: ['alertname', 'instance']

# Time-based routing
route:
  routes:
  - match:
      severity: critical
    receiver: 'pager-duty'
    group_wait: 10s
    repeat_interval: 30m
    continue: true
  - match_re:
      time: '^(09|10|11|12|13|14|15|16|17):'
    receiver: 'business-hours'
  - receiver: 'after-hours'
```

---

## 📋 Практическое задание к Блоку 3

### 🎯 Итоговый проект: Комплексная система мониторинга

**Задача:** Развернуть полнофункциональную систему мониторинга для тестовой инфраструктуры

#### Инфраструктура для мониторинга:

```
┌─────────────────────────────────────────────────────────┐
│                Test Infrastructure                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  🖥️  Web Servers (3)    │  Nginx + PHP-FPM             │
│  🗄️  Database Server    │  MySQL/PostgreSQL            │
│  📨 Message Queue       │  RabbitMQ                     │
│  🔄 Load Balancer       │  HAProxy                      │
│  💾 Cache Server        │  Redis                        │
│  📊 Monitoring Stack    │  Prometheus + Grafana         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### Требования к проекту:

✅ **Инфраструктурный мониторинг:**
- Node Exporter на всех серверах
- SNMP мониторинг сетевого оборудования
- Blackbox monitoring критических сервисов

✅ **Мониторинг приложений:**
- Веб-серверы (Nginx, Apache)
- Базы данных (MySQL/PostgreSQL)
- Очереди сообщений (RabbitMQ)
- Кэш-серверы (Redis)

✅ **Визуализация:**
- Дашборды по категориям (инфраструктура, приложения, бизнес-метрики)
- Различные типы панелей (графики, таблицы, индикаторы)
- Темплейтинг и переменные

✅ **Алертинг:**
- Многоуровневая система алертов
- Интеграция с Slack и email
- Правила эскалации и подавления

#### Критерии оценки:

```
┌─────────────────────────────────────────────────────────┐
│                 Assessment Criteria                    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  🏗️  Architecture (25%)    │  Правильная архитектура    │
│  ⚙️  Configuration (25%)   │  Качество настройки        │
│  📊 Dashboards (25%)      │  Информативность дашбордов │
│  🚨 Alerting (25%)        │  Эффективность алертинга   │
│                                                         │
│  🎯 Проходной балл: 80%                                │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 📚 Дополнительные материалы

### 🔧 Полезные команды

```bash
# Prometheus
promtool check config prometheus.yml
promtool query instant 'up'
promtool test rules test.yml

# AlertManager  
amtool config check alertmanager.yml
amtool alert query
amtool silence add alertname=HighCPUUsage

# Grafana
grafana-cli admin reset-admin-password newpassword
grafana-cli plugins list-remote
```

### 📖 Рекомендуемая литература

- **"Prometheus: Up & Running"** - Brian Brazil
- **"Monitoring with Prometheus"** - James Turnbull  
- **"Grafana 8.x Complete Guide"** - Alan Donovan
- **"Site Reliability Engineering"** - Google

### 🔗 Полезные ссылки

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [PromQL Tutorial](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [AlertManager Documentation](https://prometheus.io/docs/alerting/latest/alertmanager/)

---

## ✅ Чек-лист завершения блока

- [ ] Prometheus установлен и настроен
- [ ] Grafana интегрирована с Prometheus
- [ ] Node Exporter развернут на серверах
- [ ] SNMP мониторинг настроен
- [ ] Blackbox monitoring работает
- [ ] Мониторинг БД и приложений настроен
- [ ] AlertManager настроен и протестирован
- [ ] Создана система эскалации алертов
- [ ] Дашборды созданы для всех компонентов
- [ ] Документация написана
- [ ] Команда обучена работе с системой

---

**🎯 Результат:** Полнофункциональная система мониторинга готова к production использованию!