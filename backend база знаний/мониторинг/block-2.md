# Блок 2: Инструменты и технологии мониторинга

**Длительность:** 4-5 недель  
**🎯 Цель:** Освоить основные инструменты мониторинга и их применение

---

## Обзор блока

В этом блоке мы изучим основные категории инструментов мониторинга:
- 📊 **Системы мониторинга инфраструктуры**
- 📝 **Системы логирования и анализа** 
- 🚀 **APM и мониторинг приложений**
- ☁️ **Облачные и современные решения**

---

## Глава 2.1: Системы мониторинга инфраструктуры

### 🔍 Обзор инструментов

```
┌─────────────────────────────────────────────────────────────┐
│                 СИСТЕМЫ МОНИТОРИНГА                         │
├─────────────────┬─────────────────┬─────────────────────────┤
│    ZABBIX       │    NAGIOS       │      PROMETHEUS         │
│   (Enterprise)  │  (Classical)    │      (Modern)           │
└─────────────────┴─────────────────┴─────────────────────────┘
```

---

### 📈 Zabbix: Enterprise-решение

**Когда использовать:**
- Крупные корпоративные сети
- Смешанная инфраструктура (Linux/Windows/Network)
- Необходимость в готовых шаблонах
- Требования к enterprise-поддержке

**Архитектура Zabbix:**

```
┌─────────────────────────────────────────────────────────────┐
│                    ZABBIX АРХИТЕКТУРА                       │
└─────────────────────────────────────────────────────────────┘

    [Web Interface] ←→ [Zabbix Server] ←→ [Database]
           ↑                 ↑                ↑
           │                 │                │
    ┌──────┴──────┐   ┌──────┴──────┐  ┌──────┴──────┐
    │   Grafana   │   │   Proxy     │  │ MySQL/      │
    │  (Optional) │   │ (Optional)  │  │ PostgreSQL  │
    └─────────────┘   └─────────────┘  └─────────────┘
                            ↑
                    ┌───────┴───────┐
                    │               │
              [Zabbix Agent]   [SNMP Devices]
                    │               │
              [Linux/Windows]  [Switches/Routers]
```

**Пример конфигурации Zabbix Agent:**

```bash
# /etc/zabbix/zabbix_agentd.conf

Server=192.168.1.10
ServerActive=192.168.1.10
Hostname=web-server-01
RefreshActiveChecks=120

# Пользовательские параметры
UserParameter=mysql.ping,mysqladmin -uroot ping | grep -c alive
UserParameter=custom.disk.discovery,/etc/zabbix/scripts/disk_discovery.sh
UserParameter=nginx.status[*],/etc/zabbix/scripts/nginx_status.sh $1

# Логи для мониторинга
LogFile=/tmp/zabbix_agentd.log
LogFileSize=10
```

**Практическое задание 2.1.1:**
```bash
# Установка Zabbix Server на Ubuntu 20.04

# 1. Добавление репозитория
wget https://repo.zabbix.com/zabbix/5.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_5.4-1+ubuntu20.04_all.deb
sudo dpkg -i zabbix-release_5.4-1+ubuntu20.04_all.deb
sudo apt update

# 2. Установка компонентов
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent

# 3. Настройка базы данных
sudo mysql -uroot -p
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> quit;

# 4. Импорт схемы
zcat /usr/share/doc/zabbix-sql-scripts/mysql/create.sql.gz | mysql -uzabbix -p zabbix
```

---

### 🔧 Nagios/Icinga: Классические решения

**Особенности Nagios:**
- Плагинная архитектура
- Активные и пассивные проверки
- Гибкая система уведомлений
- Развитая экосистема плагинов

**Архитектура Nagios:**

```
                    NAGIOS CORE ARCHITECTURE
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   Web UI    │    │ Config Files│    │   Plugins   │     │
│  │  (CGI/PHP)  │    │   (.cfg)    │    │ (check_*)   │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
│         │                   │                   │          │
│  ┌──────┴───────────────────┴───────────────────┴──────┐   │
│  │              NAGIOS CORE ENGINE                     │   │
│  │                                                     │   │
│  │  • Scheduler    • Event Handler   • Notifications  │   │
│  │  • State Engine • Command Processor                │   │
│  └─────────────────────────────────────────────────────┘   │
│                           │                                │
└───────────────────────────┼────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   [SSH Checks]      [SNMP Checks]     [HTTP Checks]
        │                  │                  │
   [Linux Hosts]    [Network Devices]   [Web Services]
```

**Пример конфигурации хоста в Nagios:**

```bash
# /etc/nagios/objects/servers.cfg

define host {
    host_name               web-server-01
    alias                   Web Server 01
    address                 192.168.1.100
    check_command           check-host-alive
    check_interval          5
    retry_interval          1
    max_check_attempts      3
    check_period            24x7
    process_perf_data       1
    retain_status_information   1
    contact_groups          admins
    notification_interval   60
    notification_period     24x7
    notification_options    d,u,r
}

define service {
    host_name               web-server-01
    service_description     HTTP
    check_command           check_http
    max_check_attempts      3
    check_interval          5
    retry_interval          1
    check_period            24x7
    notification_interval   60
    notification_period     24x7
    notification_options    w,u,c,r
    contact_groups          admins
}
```

---

### 🚀 Prometheus: Современная метрическая система

**Преимущества Prometheus:**
- Pull-модель сбора метрик
- Мощный язык запросов PromQL
- Встроенная база данных временных рядов
- Service Discovery
- Экосистема экспортеров

**Архитектура Prometheus:**

```
                    PROMETHEUS ECOSYSTEM
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│ ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐   │
│ │   Grafana   │  │ AlertManager│  │  Prometheus Server  │   │
│ │(Visualization)│ │(Alerting)   │  │                     │   │
│ └─────────────┘  └─────────────┘  │ • TSDB              │   │
│        │                │         │ • PromQL Engine     │   │
│        │                │         │ • Service Discovery │   │
│        └────────────────┼─────────│ • Scrape Manager    │   │
│                         │         └─────────────────────┘   │
│                         │                    │              │
└─────────────────────────┼────────────────────┼──────────────┘
                          │                    │
                          │              ┌─────┴─────┐
                          │              │           │
                          │        [Pull Metrics]    │
                          │              │           │
                    [Alert Rules]        │           │
                          │              │           │
                          ▼              ▼           ▼
                   ┌──────────────┐ ┌─────────┐ ┌─────────┐
                   │ Notification │ │Node     │ │Custom   │
                   │  Channels    │ │Exporter │ │Exporter │
                   │              │ │         │ │         │
                   │• Email       │ │[Metrics]│ │[Metrics]│
                   │• Slack       │ └─────────┘ └─────────┘
                   │• Webhook     │      │           │
                   └──────────────┘      │           │
                                    [Linux]    [Application]
```

**Пример конфигурации Prometheus:**

```yaml
# prometheus.yml

global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alert_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  # Мониторинг самого Prometheus
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Мониторинг Linux серверов
  - job_name: 'node'
    static_configs:
      - targets: 
        - 'web-server-01:9100'
        - 'db-server-01:9100'
        - 'app-server-01:9100'
    scrape_interval: 30s
    metrics_path: /metrics

  # Service Discovery для Kubernetes
  - job_name: 'kubernetes-apiservers'
    kubernetes_sd_configs:
    - role: endpoints
    scheme: https
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    relabel_configs:
    - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
      action: keep
      regex: default;kubernetes;https
```

**PromQL примеры запросов:**

```promql
# CPU загрузка по серверам
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Память использование
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Сетевой трафик
rate(node_network_receive_bytes_total[5m]) * 8

# Top 5 процессов по CPU
topk(5, rate(node_cpu_seconds_total[5m]))

# Прогноз заполнения диска на 4 часа
predict_linear(node_filesystem_avail_bytes[1h], 4*3600) < 0
```

---

### 📊 InfluxDB + Telegraf: Time-Series решение

**Когда использовать:**
- Высокочастотные метрики
- IoT и сенсорные данные
- Потребность в SQL-подобных запросах
- Интеграция с Grafana

**Архитектура InfluxDB:**

```
                  TICK STACK ARCHITECTURE
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│ ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐   │
│ │   Chronograf│  │   Kapacitor │  │     InfluxDB        │   │
│ │ (Dashboard) │  │ (Alerting)  │  │   (Time Series DB)  │   │
│ └─────────────┘  └─────────────┘  └─────────────────────┘   │
│        │                │                     │             │
│        └────────────────┼─────────────────────┘             │
│                         │                                   │
│                         │                                   │
│                  ┌──────┴──────┐                            │
│                  │   Telegraf  │                            │
│                  │(Data Agent) │                            │
│                  └─────────────┘                            │
│                         │                                   │
└─────────────────────────┼───────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
   [System Metrics] [Application]    [Network SNMP]
        │                 │                 │
   [CPU/Memory]      [MySQL/Redis]     [Switches]
```

**Пример конфигурации Telegraf:**

```toml
# /etc/telegraf/telegraf.conf

[global_tags]
  environment = "production"
  datacenter = "us-west-2"

[agent]
  interval = "10s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "10s"
  flush_jitter = "0s"
  precision = ""
  hostname = ""
  omit_hostname = false

# InfluxDB output
[[outputs.influxdb]]
  urls = ["http://localhost:8086"]
  database = "telegraf"
  username = "admin"
  password = "password"

# Система
[[inputs.cpu]]
  percpu = true
  totalcpu = true
  collect_cpu_time = false
  report_active = false

[[inputs.disk]]
  ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]

[[inputs.diskio]]

[[inputs.mem]]

[[inputs.net]]

[[inputs.processes]]

[[inputs.swap]]

[[inputs.system]]

# MySQL мониторинг
[[inputs.mysql]]
  servers = ["root:password@tcp(localhost:3306)/"]
  metric_version = 2

# Nginx мониторинг
[[inputs.nginx]]
  urls = ["http://localhost/nginx_status"]

# Custom script
[[inputs.exec]]
  commands = ["/opt/scripts/check_application.sh"]
  timeout = "5s"
  data_format = "influx"
```

---

### 🔄 Сравнительная таблица инструментов

```
┌──────────────────────────────────────────────────────────────────────────┐
│                     СРАВНЕНИЕ РЕШЕНИЙ                                   │
├─────────────┬─────────────┬─────────────┬─────────────┬─────────────────┤
│  Критерий   │   Zabbix    │   Nagios    │ Prometheus  │    InfluxDB     │
├─────────────┼─────────────┼─────────────┼─────────────┼─────────────────┤
│ Сложность   │   Средняя   │   Высокая   │   Средняя   │     Низкая      │
│ внедрения   │             │             │             │                 │
├─────────────┼─────────────┼─────────────┼─────────────┼─────────────────┤
│ Масштаб-    │   Хорошее   │   Среднее   │  Отличное   │    Отличное     │
│ руемость    │             │             │             │                 │
├─────────────┼─────────────┼─────────────┼─────────────┼─────────────────┤
│ Визуали-    │   Базовая   │   Слабая    │   Внешняя   │    Хорошая      │
│ зация       │             │             │  (Grafana)  │  (Chronograf)   │
├─────────────┼─────────────┼─────────────┼─────────────┼─────────────────┤
│ Алертинг    │   Хороший   │  Отличный   │   Хороший   │     Хороший     │
│             │             │             │(AlertMngr)  │  (Kapacitor)    │
├─────────────┼─────────────┼─────────────┼─────────────┼─────────────────┤
│ Community   │   Большое   │  Огромное   │ Растущее    │     Среднее     │
├─────────────┼─────────────┼─────────────┼─────────────┼─────────────────┤
│ Стоимость   │ Free/Paid   │Free/Paid    │    Free     │   Free/Paid     │
├─────────────┼─────────────┼─────────────┼─────────────┼─────────────────┤
│ Лучше для   │ Enterprise  │ Infrastructure│ Microservices│ Time-series   │
│             │ Networks    │ Monitoring  │ & Cloud     │ & Analytics     │
└─────────────┴─────────────┴─────────────┴─────────────┴─────────────────┘
```

---

## ✅ Практическое задание 2.1: Установка и настройка Zabbix

### Цель
Установить и настроить полнофункциональную систему мониторинга Zabbix для тестового сервера.

### Шаги выполнения

#### 1. Подготовка окружения
```bash
# Обновление системы
sudo apt update && sudo apt upgrade -y

# Установка необходимых пакетов
sudo apt install wget curl gnupg2 software-properties-common -y
```

#### 2. Установка и настройка MySQL
```bash
# Установка MySQL
sudo apt install mysql-server -y

# Настройка безопасности
sudo mysql_secure_installation

# Создание базы данных для Zabbix
sudo mysql -u root -p
```

```sql
CREATE DATABASE zabbix CHARACTER SET utf8 COLLATE utf8_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'StrongPassword123!';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

#### 3. Установка Zabbix
```bash
# Добавление репозитория Zabbix
wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-1+ubuntu20.04_all.deb
sudo dpkg -i zabbix-release_6.0-1+ubuntu20.04_all.deb
sudo apt update

# Установка Zabbix сервера, веб-интерфейса и агента
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

#### 4. Настройка Zabbix сервера
```bash
# Импорт начальной схемы
zcat /usr/share/doc/zabbix-sql-scripts/mysql/create.sql.gz | mysql -uzabbix -p zabbix

# Редактирование конфигурации сервера
sudo nano /etc/zabbix/zabbix_server.conf
```

```ini
# Основные настройки в zabbix_server.conf
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=StrongPassword123!

# Настройки производительности
StartPollers=10
StartTrappers=5
StartPingers=5
StartDiscoverers=3
StartHTTPPollers=3
```

#### 5. Настройка веб-интерфейса
```bash
# Настройка PHP для Zabbix
sudo nano /etc/zabbix/apache.conf
```

```apache
# Настройки в apache.conf
php_value max_execution_time 300
php_value memory_limit 128M
php_value post_max_size 16M
php_value upload_max_filesize 2M
php_value max_input_time 300
php_value max_input_vars 10000
php_value always_populate_raw_post_data -1
php_value date.timezone Europe/Moscow
```

#### 6. Запуск сервисов
```bash
# Перезапуск и включение автозапуска
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2

# Проверка статуса
sudo systemctl status zabbix-server
sudo systemctl status zabbix-agent
```

#### 7. Веб-настройка
1. Открыть браузер: `http://your-server-ip/zabbix`
2. Следовать мастеру установки
3. Войти с учетными данными: `Admin` / `zabbix`

#### 8. Создание хоста для мониторинга
```bash
# Настройка Zabbix агента на целевом сервере
sudo nano /etc/zabbix/zabbix_agentd.conf
```

```ini
# Конфигурация агента
Server=YOUR_ZABBIX_SERVER_IP
ServerActive=YOUR_ZABBIX_SERVER_IP
Hostname=target-server-01
RefreshActiveChecks=120

# Пользовательские параметры
UserParameter=custom.cpu.load,cat /proc/loadavg | awk '{print $1}'
UserParameter=custom.memory.usage,free | grep Mem | awk '{printf("%.2f", ($3/$2)*100)}'
```

### Результат
После выполнения задания у вас будет:
- Работающий Zabbix сервер
- Настроенный веб-интерфейс
- Мониторинг минимум одного хоста
- Базовые графики и триггеры

---

## 📚 Дополнительные материалы

### Полезные ссылки
- [Документация Zabbix](https://www.zabbix.com/documentation/current/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Nagios Core Documentation](https://www.nagios.org/documentation/)
- [InfluxDB Documentation](https://docs.influxdata.com/)

### Рекомендуемые шаблоны для изучения
- Zabbix templates на GitHub
- Prometheus exporters
- Nagios Exchange
- Telegraf input plugins

---

*Следующая глава: **2.2 Системы логирования и анализа***