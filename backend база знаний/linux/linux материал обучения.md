# Linux для Backend разработчика: от нуля до эксперта

> **Цель курса:** Превратить backend разработчика в уверенного пользователя Linux, способного администрировать серверы и оптимизировать production окружения.

---

## 🎯 Модуль 1: Основы командной строки (2-3 недели)

### Зачем это нужно backend разработчику?
- **95% серверов** работают на Linux
- Debugging в production происходит через терминал
- Автоматизация deployments требует знания bash
- DevOps практики невозможны без CLI

### 📁 Файловая система Linux

**Концепция:** Все в Linux — это файл. Понимание структуры = понимание системы.

```bash
# Ключевые директории для backend разработчика
/home/user/          # Домашняя директория
/var/log/            # Логи приложений и системы  
/etc/                # Конфигурационные файлы
/opt/                # Установленные приложения
/tmp/                # Временные файлы
/usr/bin/            # Исполняемые файлы
```

**Навигация (must-know):**
```bash
pwd                  # Где я нахожусь?
ls -la              # Подробный список файлов
cd /var/log         # Перейти в директорию логов
find /etc -name "*.conf"  # Найти все конфиг файлы
```

### 🔧 Управление файлами

**Основные операции:**
```bash
# Создание и просмотр
touch app.log       # Создать файл
mkdir -p logs/app   # Создать структуру директорий
cat error.log       # Посмотреть содержимое
tail -f access.log  # Следить за логами в реальном времени

# Копирование и перемещение  
cp config.json config.backup.json
mv old_app.py archive/
rm -rf temp_files/
```

### 🔍 Поиск и фильтрация

**Поиск ошибок в логах:**
```bash
# Найти все ошибки за сегодня
grep "ERROR" /var/log/app.log | tail -20

# Найти IP адреса в access логах
grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' access.log | sort | uniq

# Подсчитать количество запросов по эндпоинтам
awk '{print $7}' access.log | sort | uniq -c | sort -nr
```

### ✏️ Работа с редакторами

**Vim (минимум для выживания):**
```bash
vim config.py
# i - режим вставки
# Esc - выход из режима вставки  
# :w - сохранить
# :q - выйти
# :wq - сохранить и выйти
# /search_term - поиск
```

### ✅ Чек-лист Модуля 1
- [ ] Могу навигировать по файловой системе без GUI
- [ ] Умею искать файлы и контент в файлах
- [ ] Могу редактировать файлы в терминале
- [ ] Понимаю права доступа к файлам (rwx)
- [ ] Умею работать с логами приложений

---

## ⚡ Модуль 2: Процессы и системные ресурсы (2 недели)

### Зачем это нужно?
- **Debugging производительности** приложений
- Мониторинг потребления ресурсов в production
- Управление background процессами
- Устранение "подвисших" процессов

### 📊 Мониторинг процессов

**Основные команды:**
```bash
# Посмотреть все процессы
ps aux | grep python    # Найти Python процессы
ps aux | head -20       # Топ 20 процессов

# Интерактивный мониторинг
top                     # Базовый мониторинг
htop                    # Улучшенная версия (нужно установить)

# Поиск процесса по порту
lsof -i :8000          # Кто использует порт 8000?
netstat -tulpn | grep :80  # Альтернативный способ
```

### 🎯 Управление процессами

**Запуск и остановка:**
```bash
# Запуск в фоне
python app.py &         # Запустить в background
nohup python app.py &   # Запустить с игнорированием hangup

# Управление процессами
jobs                    # Посмотреть background задачи
kill 1234              # Остановить процесс по PID
killall python         # Остановить все Python процессы
pkill -f "app.py"      # Остановить по имени файла
```

### 💾 Мониторинг ресурсов

**Память и диск:**
```bash
# Память
free -h                 # Использование RAM
cat /proc/meminfo       # Детальная информация о памяти

# Диск
df -h                   # Использование дискового пространства
du -sh /var/log/*       # Размер директорий в /var/log
du -sh . | sort -hr     # Самые большие директории

# I/O операции
iostat 1 5              # Статистика I/O каждую секунду, 5 раз
```

### 📋 Практический сценарий

**Проблема:** Приложение работает медленно
```bash
# 1. Проверяем нагрузку на CPU
top -p $(pgrep python)

# 2. Проверяем использование памяти
ps aux --sort=-%mem | head -10

# 3. Проверяем дисковое пространство  
df -h | grep -E "(Use%|/)"

# 4. Смотрим активность диска
iotop -o               # Показать только активные процессы
```

### ✅ Чек-лист Модуля 2
- [ ] Умею мониторить системные ресурсы (CPU, RAM, диск)
- [ ] Могу найти и остановить процессы
- [ ] Понимаю, как запускать процессы в фоне
- [ ] Умею диагностировать проблемы производительности
- [ ] Знаю, как читать системную статистику

---

## 🔐 Модуль 3: Пользователи и безопасность (1-2 недели)

### Зачем это нужно?
- **Безопасность production серверов**
- Ограничение доступа к критичным ресурсам
- Правильная настройка прав для приложений
- Аудит доступа к системе

### 👥 Управление пользователями

**Основы:**
```bash
# Текущий пользователь и группы
whoami                  # Кто я?
id                     # Мои права и группы
groups                 # Мои группы

# Создание пользователя для приложения
sudo useradd -r -s /bin/false appuser  # Системный пользователь
sudo usermod -aG docker appuser        # Добавить в группу docker
```

### 🔒 Права доступа

**Система прав rwx:**
```bash
# Просмотр прав
ls -la config.py        # -rw-r--r-- (644)
# r=4, w=2, x=1
# 644 = rw-r--r-- (владелец: чтение+запись, группа и остальные: только чтение)

# Изменение прав
chmod 600 secret.key    # Только владелец может читать/писать
chmod +x deploy.sh      # Добавить право на выполнение
chmod -R 755 /var/www   # Рекурсивно для директории

# Изменение владельца
sudo chown appuser:appuser /opt/myapp
sudo chown -R nginx:nginx /var/www/html
```

### 🔑 Работа с sudo

**Безопасное администрирование:**
```bash
# Временное повышение прав
sudo systemctl restart nginx
sudo tail /var/log/auth.log

# Переключение пользователя
sudo -u appuser python app.py  # Запустить от имени appuser
sudo -i                        # Стать root
```

### 🚪 SSH и аутентификация

**Настройка безопасного доступа:**
```bash
# Генерация SSH ключа
ssh-keygen -t rsa -b 4096 -C "deploy@myapp.com"

# Копирование ключа на сервер
ssh-copy-id user@server.com

# Безопасное подключение
ssh -i ~/.ssh/deploy_key user@server.com
```

### 🛡️ Практический сценарий безопасности

**Настройка прав для web приложения:**
```bash
# 1. Создаем пользователя для приложения
sudo useradd -r -d /opt/myapp -s /bin/bash appuser

# 2. Настраиваем права на файлы приложения
sudo chown -R appuser:appuser /opt/myapp
chmod -R 750 /opt/myapp          # Только владелец и группа
chmod 640 /opt/myapp/config.py   # Конфиг только для чтения

# 3. Настраиваем права на логи
sudo chown appuser:syslog /var/log/myapp.log
chmod 640 /var/log/myapp.log
```

### ✅ Чек-лист Модуля 3
- [ ] Понимаю систему прав доступа Linux (rwx, числовые права)
- [ ] Умею создавать и настраивать пользователей
- [ ] Могу работать с sudo безопасно
- [ ] Настроил SSH ключи для безопасного доступа
- [ ] Знаю принципы минимальных привилегий

---

## 🌐 Модуль 4: Сетевые технологии (2 недели)

### Зачем это нужно?
- **Debugging сетевых проблем** в production
- Настройка load balancers и reverse proxy
- Мониторинг сетевой активности приложений
- Конфигурация файрволов

### 🔍 Диагностика сети

**Основные инструменты:**
```bash
# Проверка подключения
ping google.com         # Базовая проверка
ping -c 4 192.168.1.1  # 4 пакета к локальному gateway

# Трассировка маршрута
traceroute google.com   # Путь пакетов до цели
mtr google.com          # Непрерывная трассировка

# DNS проверки
nslookup myapp.com      # Базовая DNS проверка
dig myapp.com           # Детальная DNS информация
dig @8.8.8.8 myapp.com  # Проверка через конкретный DNS
```

### 🔌 Мониторинг портов и соединений

**Активные соединения:**
```bash
# Слушающие порты
netstat -tulpn          # Все слушающие порты
ss -tulpn               # Современная альтернатива netstat
lsof -i                 # Открытые сетевые файлы

# Конкретные проверки
netstat -an | grep :80  # Кто слушает 80 порт
ss -tulpn | grep :3306  # MySQL соединения
lsof -i :8000           # Что использует порт 8000
```

### 🔧 Настройка сети

**Конфигурация интерфейсов:**
```bash
# Просмотр интерфейсов
ip addr show            # Все сетевые интерфейсы  
ip route show           # Таблица маршрутизации

# Статистика интерфейсов
cat /proc/net/dev       # Статистика сетевых интерфейсов
iftop                   # Мониторинг трафика в реальном времени
```

### 🛡️ Базовый файрвол

**iptables основы:**
```bash
# Просмотр правил
sudo iptables -L -n -v  # Все правила с подробностями

# Базовые правила для web сервера
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT   # SSH
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT   # HTTP  
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT  # HTTPS
sudo iptables -A INPUT -j DROP                       # Блокировать остальное

# UFW (упрощенный файрвол)
sudo ufw enable
sudo ufw allow 22/tcp
sudo ufw allow 80,443/tcp
```

### 📡 Анализ трафика

**tcpdump для отладки:**
```bash
# Захват HTTP трафика
sudo tcpdump -i any port 80 -A

# Мониторинг конкретного хоста
sudo tcpdump host 192.168.1.100

# Сохранение в файл для анализа
sudo tcpdump -w capture.pcap port 443
```

### 🔧 Практический сценарий

**Проблема:** API не отвечает с определенных IP
```bash
# 1. Проверяем, слушает ли приложение
ss -tulpn | grep :8000

# 2. Проверяем подключения к приложению
netstat -an | grep :8000 | grep ESTABLISHED

# 3. Проверяем файрвол
sudo iptables -L INPUT -n --line-numbers

# 4. Тестируем подключение изнутри
curl -v http://localhost:8000/health

# 5. Мониторим входящие соединения
sudo tcpdump -i any port 8000 -n
```

### ✅ Чек-лист Модуля 4
- [ ] Умею диагностировать сетевые проблемы
- [ ] Могу мониторить порты и соединения
- [ ] Понимаю основы DNS и умею его отлаживать
- [ ] Настроил базовый файрвол
- [ ] Могу анализировать сетевой трафик

---

## 🌍 Модуль 5: Веб-серверы и базы данных (2-3 недели)

### Зачем это нужно?
- **Production deployment** требует настройки веб-серверов
- Reverse proxy для масштабирования приложений  
- SSL/TLS для безопасности
- Оптимизация производительности БД

### 🚀 Nginx: современный веб-сервер

**Установка и базовая настройка:**
```bash
# Установка (Ubuntu/Debian)
sudo apt update && sudo apt install nginx

# Управление сервисом
sudo systemctl start nginx
sudo systemctl enable nginx    # Автозапуск
sudo systemctl status nginx    # Проверка статуса

# Проверка конфигурации
sudo nginx -t               # Тест конфигурации
sudo systemctl reload nginx  # Перезагрузка без остановки
```

**Конфигурация для backend приложения:**
```nginx
# /etc/nginx/sites-available/myapp
server {
    listen 80;
    server_name myapp.com;
    
    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    
    location /static/ {
        alias /var/www/myapp/static/;
        expires 30d;
    }
}
```

**Активация конфигурации:**
```bash
# Включение сайта
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

### 🔒 SSL/TLS с Let's Encrypt

**Автоматические сертификаты:**
```bash
# Установка Certbot
sudo apt install certbot python3-certbot-nginx

# Получение сертификата
sudo certbot --nginx -d myapp.com

# Автообновление
sudo crontab -e
# Добавить: 0 12 * * * /usr/bin/certbot renew --quiet
```

### 🗄️ PostgreSQL для production

**Установка и настройка:**
```bash
# Установка
sudo apt install postgresql postgresql-contrib

# Создание пользователя и БД
sudo -u postgres psql
CREATE USER myapp WITH PASSWORD 'secure_password';
CREATE DATABASE myapp OWNER myapp;
GRANT ALL PRIVILEGES ON DATABASE myapp TO myapp;
\q
```

**Настройка удаленного доступа:**
```bash
# Редактируем postgresql.conf
sudo nano /etc/postgresql/13/main/postgresql.conf
# listen_addresses = 'localhost'  # Только локальный доступ

# Настраиваем аутентификацию pg_hba.conf
sudo nano /etc/postgresql/13/main/pg_hba.conf
# local   myapp    myapp                     md5
# host    myapp    myapp    127.0.0.1/32     md5
```

### 📊 Мониторинг и логи

**Nginx логи:**
```bash
# Основные логи
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log

# Анализ access логов
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head -10
# Топ IP адресов

grep " 5[0-9][0-9] " /var/log/nginx/access.log | tail -20
# Последние 500 ошибки
```

**PostgreSQL мониторинг:**
```bash
# Активные соединения
sudo -u postgres psql -c "SELECT * FROM pg_stat_activity;"

# Размеры БД
sudo -u postgres psql -c "SELECT datname, pg_size_pretty(pg_database_size(datname)) FROM pg_database;"

# Медленные запросы (если включены в конфиге)
sudo tail -f /var/log/postgresql/postgresql-13-main.log
```

### 🔧 Практическая настройка production

**Полный пример деплоя Flask приложения:**
```bash
# 1. Подготовка приложения
sudo useradd -r -d /opt/myapp -s /bin/bash myapp
sudo mkdir -p /opt/myapp
sudo chown myapp:myapp /opt/myapp

# 2. Настройка systemd сервиса
sudo tee /etc/systemd/system/myapp.service > /dev/null <<EOF
[Unit]
Description=MyApp Flask Application
After=network.target

[Service]
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/venv/bin/python app.py
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# 3. Запуск сервиса
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp
```

### ✅ Чек-лист Модуля 5
- [ ] Настроил Nginx как reverse proxy
- [ ] Получил и настроил SSL сертификат
- [ ] Установил и сконфигурировал PostgreSQL
- [ ] Настроил мониторинг веб-сервера и БД
- [ ] Создал systemd сервис для приложения

---

## 🐳 Модуль 6: Контейнеризация с Docker (2 недели)

### Зачем это нужно?
- **Изоляция приложений** и их зависимостей
- Одинаковое окружение dev/staging/production
- Простое масштабирование и deployment
- Упрощение CI/CD процессов

### 🚀 Docker основы

**Установка и первые шаги:**
```bash
# Установка Docker (Ubuntu)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER  # Добавить пользователя в группу docker

# Проверка установки
docker --version
docker run hello-world
```

**Основные команды:**
```bash
# Работа с образами
docker images               # Список локальных образов
docker pull nginx:alpine    # Скачать образ
docker rmi nginx:alpine     # Удалить образ

# Работа с контейнерами
docker ps                   # Запущенные контейнеры
docker ps -a                # Все контейнеры
docker run -d -p 80:80 nginx     # Запустить контейнер
docker stop container_id     # Остановить контейнер
docker rm container_id       # Удалить контейнер
```

### 📦 Создание собственных образов

**Dockerfile для Python приложения:**
```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

# Копируем requirements и устанавливаем зависимости
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Копируем код приложения
COPY . .

# Создаем пользователя без root прав
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

CMD ["python", "app.py"]
```

**Сборка и запуск:**
```bash
# Сборка образа
docker build -t myapp:latest .

# Запуск с переменными окружения
docker run -d \
  --name myapp \
  -p 8000:8000 \
  -e DATABASE_URL=postgresql://user:pass@host:5432/db \
  myapp:latest
```

### 🔧 Docker Compose для сложных приложений

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://app:password@db:5432/appdb
    depends_on:
      - db
      - redis
    volumes:
      - ./logs:/app/logs

  db:
    image: postgres:13
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: app
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./backup:/backup
    ports:
      - "5432:5432"

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./static:/var/www/static
    depends_on:
      - app

volumes:
  postgres_data:
```

**Управление compose:**
```bash
# Запуск всех сервисов
docker-compose up -d

# Просмотр логов
docker-compose logs -f app

# Масштабирование
docker-compose up -d --scale app=3

# Остановка и очистка
docker-compose down
docker-compose down -v  # С удалением volumes
```

### 📊 Мониторинг контейнеров

**Основные команды мониторинга:**
```bash
# Ресурсы контейнеров
docker stats                    # Реальное время
docker stats --no-stream       # Один снимок

# Логи контейнеров
docker logs container_name      # Все логи
docker logs -f --tail 100 container_name  # Последние 100 строк

# Инспекция контейнера
docker inspect container_name   # Полная информация
docker exec -it container_name bash  # Войти в контейнер
```

### 🛠️ Production настройки

**Оптимизация для production:**
```bash
# Многоэтапная сборка для уменьшения размера образа
# Dockerfile.prod
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
USER node
CMD ["node", "server.js"]
```

**Health checks:**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1
```

### ⚡ Практический сценарий

**Развертывание полного стека:**
```bash
# 1. Создаем сеть для контейнеров
docker network create myapp-network

# 2. Запускаем базу данных
docker run -d \
  --name myapp-db \
  --network myapp-network \
  -e POSTGRES_DB=myapp \
  -e POSTGRES_USER=app \
  -e POSTGRES_PASSWORD=secretpassword \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:13

# 3. Запускаем приложение
docker run -d \
  --name myapp-backend \
  --network myapp-network \
  -e DATABASE_URL=postgresql://app:secretpassword@myapp-db:5432/myapp \
  myapp:latest

# 4. Запускаем Nginx
docker run -d \
  --name myapp-nginx \
  --network myapp-network \
  -p 80:80 \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf \
  nginx:alpine
```

### ✅ Чек-лист Модуля 6
- [ ] Установил Docker и понимаю основные команды
- [ ] Создал Dockerfile для своего приложения
- [ ] Настроил Docker Compose для мультиконтейнерного приложения
- [ ] Умею мониторить контейнеры и их ресурсы
- [ ] Настроил health checks и production оптимизации

---

## 🤖 Модуль 7: Автоматизация и DevOps (3 недели)

### Зачем это нужно?
- **Автоматизация рутинных задач** экономит время
- Reproducible deployments снижают ошибки
- Infrastructure as Code для версионирования инфраструктуры
- CI/CD для быстрой доставки фич

### 📜 Bash скриптинг

**Основы автоматизации:**
```bash
#!/bin/bash

# deploy.sh - скрипт деплоя приложения
set -e  # Остановка при ошибках

APP_NAME="myapp"
DEPLOY_DIR="/opt/$APP_NAME"
BACKUP_DIR="/backup/$APP_NAME"

# Логирование
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

# Создание бэкапа
backup_current() {
    if [ -d "$DEPLOY_DIR" ]; then
        log "Creating backup..."
        sudo cp -r "$DEPLOY_DIR" "$BACKUP_DIR/$(date +%Y%m%d_%H%M%S)"
    fi
}

# Деплой новой версии
deploy() {
    log "Starting deployment..."
    
    # Остановка сервиса
    sudo systemctl stop $APP_NAME
    
    # Обновление кода
    cd $DEPLOY_DIR
    git pull origin main
    
    # Установка зависимостей
    pip install -r requirements.txt
    
    # Миграции БД
    python manage.py migrate
    
    # Запуск сервиса
    sudo systemctl start $APP_NAME
    
    log "Deployment completed!"
}

# Проверка здоровья приложения
health_check() {
    log "Performing health check..."
    
    for i in {1..30}; do
        if curl -f http://localhost:8000/health > /dev/null 2>&1; then
            log "Health check passed!"
            return 0
        fi
        sleep 2
    done
    
    log "Health check failed!"
    return 1
}

# Основная логика
main() {
    backup_current
    deploy
    
    if ! health_check; then
        log "Rolling back..."
        # Логика отката
        exit 1
    fi
}

main "$@"
```

### ⏰ Планировщик задач (Cron)

**Автоматизация регулярных задач:**
```bash
# Редактирование crontab
crontab -e

# Примеры заданий
# Бэкап БД каждый день в 2:00
0 2 * * * /opt/scripts/backup_db.sh

# Очистка логов каждую неделю
0 3 * * 0 find /var/log/myapp -name "*.log" -mtime +7 -delete

# Проверка здоровья каждые 5 минут
*/5 * * * * curl -f http://localhost:8000/health || /opt/scripts/alert.sh

# Ротация логов
0 0 * * * logrotate -f /etc/logrotate.d/myapp
```

**Скрипт бэкапа БД:**
```bash
#!/bin/bash
# backup_db.sh

DB_NAME="myapp"
BACKUP_DIR="/backup/database"
DATE=$(date +%Y%m%d_%H%M%S)

# Создание бэкапа
pg_dump $DB_NAME | gzip > "$BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz"

# Удаление старых бэкапов (старше 30 дней)
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete

# Проверка размера бэкапа
BACKUP_SIZE=$(stat -c%s "$BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz")
if [ $BACKUP_SIZE -lt 1000 ]; then
    echo "ERROR: Backup too small!" | mail -s "Backup Error" admin@myapp.com
fi
```

### 🔧 Ansible для управления конфигурацией

**Установка и базовая настройка:**
```bash
# Установка Ansible
pip install ansible

# Создание inventory файла
# hosts.yml
all:
  children:
    web:
      hosts:
        web1.myapp.com:
        web2.myapp.com:
    db:
      hosts:
        db1.myapp.com:
```

**Простой playbook:**
```yaml
# site.yml
---
- name: Setup web servers
  hosts: web
  become: yes
  
  tasks:
    - name: Install required packages
      apt:
        name:
          - nginx
          - python3
          - python3-pip
        state: present
        update_cache: yes
    
    - name: Create app user
      user:
        name: myapp
        system: yes
        shell: /bin/bash
        home: /opt/myapp
    
    - name: Deploy application
      git:
        repo: https://github.com/mycompany/myapp.git
        dest: /opt/myapp
        version: main
      become_user: myapp
      notify: restart myapp
    
    - name: Install Python dependencies
      pip:
        requirements: /opt/myapp/requirements.txt
        virtualenv: /opt/myapp/venv
      become_user: myapp
    
    - name: Configure nginx
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/sites-available/myapp
      notify: reload nginx
    
    - name: Enable nginx site
      file:
        src: /etc/nginx/sites-available/myapp
        dest: /etc/nginx/sites-enabled/myapp
        state: link
      notify: reload nginx
  
  handlers:
    - name: restart myapp
      systemd:
        name: myapp
        state: restarted
    
    - name: reload nginx
      systemd:
        name: nginx
        state: reloaded
```

**Запуск playbook:**
```bash
# Тест подключения
ansible all -i hosts.yml -m ping

# Запуск playbook
ansible-playbook -i hosts.yml site.yml

# Запуск для определенной группы
ansible-playbook -i hosts.yml site.yml --limit web
```

### 🔄 CI/CD с GitHub Actions

**Базовый workflow:**
```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: mycompany/myapp

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: 3.9
    
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install pytest
    
    - name: Run tests
      run: pytest
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test

  build-and-deploy:
    needs: test
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Build Docker image
      run: |
        docker build -t ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest .
    
    - name: Deploy to server
      uses: appleboy/ssh-action@v0.1.5
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.SSH_KEY }}
        script: |
          cd /opt/myapp
          docker-compose pull
          docker-compose up -d
          docker system prune -f
```

### 📊 Мониторинг автоматизации

**Скрипт проверки статуса сервисов:**
```bash
#!/bin/bash
# check_services.sh

SERVICES=("nginx" "myapp" "postgresql" "redis")
WEBHOOK_URL="https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK"

for service in "${SERVICES[@]}"; do
    if ! systemctl is-active --quiet "$service"; then
        MESSAGE="🚨 Service $service is down on $(hostname)"
        
        # Отправка в Slack
        curl -X POST -H 'Content-type: application/json' \
            --data "{\"text\":\"$MESSAGE\"}" \
            "$WEBHOOK_URL"
        
        # Попытка перезапуска
        systemctl restart "$service"
        sleep 10
        
        if systemctl is-active --quiet "$service"; then
            MESSAGE="✅ Service $service restarted successfully"
            curl -X POST -H 'Content-type: application/json' \
                --data "{\"text\":\"$MESSAGE\"}" \
                "$WEBHOOK_URL"
        fi
    fi
done
```

### ✅ Чек-лист Модуля 7
- [ ] Написал bash скрипты для автоматизации деплоя
- [ ] Настроил cron задачи для регулярных операций
- [ ] Изучил основы Ansible для управления конфигурацией
- [ ] Настроил CI/CD pipeline
- [ ] Создал мониторинг автоматизированных процессов

---

## 📊 Модуль 8: Мониторинг и логирование (2 недели)

### Зачем это нужно?
- **Проактивное обнаружение проблем** до пользователей
- Анализ производительности и bottlenecks
- Debugging issues в production
- Capacity planning и оптимизация ресурсов

### 📈 Prometheus + Grafana

**Установка Prometheus:**
```yaml
# docker-compose.monitoring.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana

  node_exporter:
    image: prom/node-exporter:latest
    container_name: node_exporter
    ports:
      - "9100:9100"

volumes:
  prometheus_data:
  grafana_data:
```

**Конфигурация Prometheus:**
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['node_exporter:9100']

  - job_name: 'myapp'
    static_configs:
      - targets: ['host.docker.internal:8000']
    metrics_path: '/metrics'
```

### 🔍 Мониторинг приложений

**Добавление метрик в Python приложение:**
```python
# app.py с метриками
from prometheus_client import Counter, Histogram, generate_latest
from flask import Flask, Response
import time

app = Flask(__name__)

# Метрики
REQUEST_COUNT = Counter('app_requests_total', 'Total requests', ['method', 'endpoint'])
REQUEST_LATENCY = Histogram('app_request_duration_seconds', 'Request latency')

@app.before_request
def before_request():
    request.start_time = time.time()

@app.after_request
def after_request(response):
    request_latency = time.time() - request.start_time
    REQUEST_LATENCY.observe(request_latency)
    REQUEST_COUNT.labels(method=request.method, endpoint=request.endpoint).inc()
    return response

@app.route('/metrics')
def metrics():
    return Response(generate_latest(), mimetype='text/plain')

@app.route('/health')
def health():
    return {'status': 'healthy'}
```

### 📊 Ключевые метрики для мониторинга

**Системные метрики:**
```bash
# CPU utilization
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory utilization
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk utilization
100 - ((node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes)

# Network I/O
rate(node_network_receive_bytes_total[5m])
rate(node_network_transmit_bytes_total[5m])
```

**Application метрики:**
```bash
# Request rate
rate(app_requests_total[5m])

# Error rate
rate(app_requests_total{status=~"5.."}[5m]) / rate(app_requests_total[5m])

# Response time percentiles
histogram_quantile(0.95, rate(app_request_duration_seconds_bucket[5m]))

# Active connections
app_active_connections
```

### 🚨 Настройка алертов

**Alertmanager конфигурация:**
```yaml
# alertmanager.yml
global:
  smtp_smarthost: 'localhost:587'
  smtp_from: 'alerts@myapp.com'

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'

receivers:
- name: 'web.hook'
  email_configs:
  - to: 'admin@myapp.com'
    subject: '🚨 Alert: {{ .GroupLabels.alertname }}'
    body: |
      {{ range .Alerts }}
      Alert: {{ .Annotations.summary }}
      Description: {{ .Annotations.description }}
      {{ end }}
```

**Правила алертов:**
```yaml
# alert_rules.yml
groups:
- name: example
  rules:
  - alert: HighCPUUsage
    expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 80
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage detected"
      description: "CPU usage is above 80% for more than 2 minutes"

  - alert: HighErrorRate
    expr: rate(app_requests_total{status=~"5.."}[5m]) / rate(app_requests_total[5m]) > 0.1
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "High error rate detected"
      description: "Error rate is above 10%"

  - alert: ServiceDown
    expr: up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Service is down"
      description: "{{ $labels.instance }} is down"
```

### 📋 Централизованное логирование

**ELK Stack с Docker:**
```yaml
# elk-stack.yml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data

  kibana:
    image: docker.elastic.co/kibana/kibana:7.15.0
    ports:
      - "5601:5601"
    environment:
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200
    depends_on:
      - elasticsearch

  logstash:
    image: docker.elastic.co/logstash/logstash:7.15.0
    ports:
      - "5044:5044"
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch

volumes:
  elasticsearch_data:
```

**Конфигурация Logstash:**
```ruby
# logstash.conf
input {
  beats {
    port => 5044
  }
}

filter {
  if [fields][log_type] == "nginx" {
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
    date {
      match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
    }
  }
  
  if [fields][log_type] == "app" {
    json {
      source => "message"
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
}
```

### 📝 Структурированное логирование

**Python приложение с structured logging:**
```python
import logging
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_record = {
            'timestamp': datetime.utcnow().isoformat(),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
        }
        
        if hasattr(record, 'user_id'):
            log_record['user_id'] = record.user_id
        if hasattr(record, 'request_id'):
            log_record['request_id'] = record.request_id
            
        return json.dumps(log_record)

# Настройка логгера
logger = logging.getLogger('myapp')
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)
logger.setLevel(logging.INFO)

# Использование
@app.before_request
def log_request():
    logger.info('Request started', extra={
        'user_id': get_current_user_id(),
        'request_id': request.headers.get('X-Request-ID'),
        'method': request.method,
        'path': request.path
    })
```

### 🔧 Практический мониторинг setup

**Скрипт для автоматической настройки:**
```bash
#!/bin/bash
# setup_monitoring.sh

# Создание директорий
mkdir -p monitoring/{prometheus,grafana,alertmanager}

# Настройка Prometheus
cat > monitoring/prometheus/prometheus.yml << EOF
global:
  scrape_interval: 15s

rule_files:
  - "alert_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['node_exporter:9100']
  
  - job_name: 'myapp'
    static_configs:
      - targets: ['host.docker.internal:8000']
EOF

# Запуск мониторинга
docker-compose -f monitoring/docker-compose.yml up -d

# Импорт дашбордов в Grafana
sleep 30
curl -X POST \
  http://admin:admin@localhost:3000/api/dashboards/db \
  -H 'Content-Type: application/json' \
  -d @monitoring/dashboards/system.json

echo "Monitoring setup completed!"
echo "Grafana: http://localhost:3000 (admin/admin)"
echo "Prometheus: http://localhost:9090"
```

### ✅ Чек-лист Модуля 8
- [ ] Настроил Prometheus для сбора метрик
- [ ] Создал дашборды в Grafana
- [ ] Настроил алерты для критических метрик
- [ ] Внедрил структурированное логирование
- [ ] Настроил централизованный сбор логов

---

## 🔒 Модуль 9: Безопасность в production (2 недели)

### Зачем это нужно?
- **Защита от кибератак** и утечек данных
- Соответствие требованиям безопасности (GDPR, PCI DSS)
- Аудит доступа и мониторинг подозрительной активности
- Backup и disaster recovery

### 🛡️ Hardening системы

**Базовые настройки безопасности:**
```bash
# Обновление системы
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y

# Настройка автоматических обновлений безопасности
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# Отключение ненужных сервисов
sudo systemctl disable telnet
sudo systemctl disable rsh
sudo systemctl disable rlogin

# Настройка SSH
sudo nano /etc/ssh/sshd_config
```

**Безопасная конфигурация SSH:**
```bash
# /etc/ssh/sshd_config
Port 2222                      # Нестандартный порт
PermitRootLogin no             # Запрет root login
PasswordAuthentication no      # Только ключи
PubkeyAuthentication yes       
MaxAuthTries 3                 # Ограничение попыток
ClientAliveInterval 300        # Таймаут сессии
ClientAliveCountMax 2
AllowUsers myuser              # Ограничение пользователей

# Перезапуск SSH
sudo systemctl restart sshd
```

### 🔥 Настройка файрвола

**UFW (Uncomplicated Firewall):**
```bash
# Базовая настройка
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Разрешить необходимые порты
sudo ufw allow 2222/tcp        # SSH
sudo ufw allow 80/tcp          # HTTP
sudo ufw allow 443/tcp         # HTTPS

# Ограничение по IP для админских портов
sudo ufw allow from 192.168.1.0/24 to any port 3000  # Grafana
sudo ufw allow from 10.0.0.0/8 to any port 9090      # Prometheus

# Защита от brute force
sudo ufw limit 2222/tcp

# Включение файрвола
sudo ufw enable
sudo ufw status verbose
```

**iptables для продвинутых правил:**
```bash
#!/bin/bash
# firewall_rules.sh

# Очистка существующих правил
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X

# Политики по умолчанию
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Разрешить loopback
iptables -I INPUT 1 -i lo -j ACCEPT

# Разрешить established соединения
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# SSH с защитой от brute force
iptables -A INPUT -p tcp --dport 2222 -m state --state NEW -m recent --set --name SSH
iptables -A INPUT -p tcp --dport 2222 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 --name SSH -j DROP
iptables -A INPUT -p tcp --dport 2222 -j ACCEPT

# Web трафик
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Сохранение правил
iptables-save > /etc/iptables/rules.v4
```

### 🕵️ Fail2ban для защиты от атак

**Установка и настройка:**
```bash
# Установка
sudo apt install fail2ban

# Конфигурация
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

**Настройка jail.local:**
```ini
[DEFAULT]
bantime = 3600          # Время блокировки (1 час)
findtime = 600          # Период анализа (10 минут)
maxretry = 3            # Максимум попыток

[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log

[nginx-http-auth]
enabled = true
filter = nginx-http-auth
logpath = /var/log/nginx/error.log

[nginx-limit-req]
enabled = true
filter = nginx-limit-req
logpath = /var/log/nginx/error.log
```

**Мониторинг fail2ban:**
```bash
# Статус
sudo fail2ban-client status

# Статус конкретного jail
sudo fail2ban-client status sshd

# Разблокировка IP
sudo fail2ban-client set sshd unbanip 192.168.1.100
```

### 🔐 Управление сертификатами

**Let's Encrypt автоматизация:**
```bash
#!/bin/bash
# ssl_renewal.sh

DOMAINS=("myapp.com" "api.myapp.com" "admin.myapp.com")
WEBHOOK_URL="https://hooks.slack.com/services/YOUR/WEBHOOK"

for domain in "${DOMAINS[@]}"; do
    # Проверка срока действия сертификата
    EXPIRY=$(echo | openssl s_client -servername $domain -connect $domain:443 2>/dev/null | openssl x509 -noout -dates | grep notAfter | cut -d= -f2)
    EXPIRY_TIMESTAMP=$(date -d "$EXPIRY" +%s)
    CURRENT_TIMESTAMP=$(date +%s)
    DAYS_LEFT=$(( ($EXPIRY_TIMESTAMP - $CURRENT_TIMESTAMP) / 86400 ))
    
    if [ $DAYS_LEFT -lt 30 ]; then
        echo "Certificate for $domain expires in $DAYS_LEFT days. Renewing..."
        
        if certbot renew --cert-name $domain --nginx; then
            MESSAGE="✅ SSL certificate for $domain renewed successfully"
        else
            MESSAGE="❌ Failed to renew SSL certificate for $domain"
        fi
        
        # Уведомление в Slack
        curl -X POST -H 'Content-type: application/json' \
            --data "{\"text\":\"$MESSAGE\"}" \
            "$WEBHOOK_URL"
    fi
done
```

### 🔍 Аудит безопасности

**Скрипт аудита системы:**
```bash
#!/bin/bash
# security_audit.sh

REPORT_FILE="/tmp/security_audit_$(date +%Y%m%d).txt"

echo "Security Audit Report - $(date)" > $REPORT_FILE
echo "=================================" >> $REPORT_FILE

# Проверка обновлений
echo -e "\n[UPDATES]" >> $REPORT_FILE
apt list --upgradable 2>/dev/null | grep -v "WARNING" >> $REPORT_FILE

# Проверка открытых портов
echo -e "\n[OPEN PORTS]" >> $REPORT_FILE
netstat -tulpn | grep LISTEN >> $REPORT_FILE

# Проверка неудачных входов
echo -e "\n[FAILED LOGINS - Last 24h]" >> $REPORT_FILE
grep "Failed password" /var/log/auth.log | grep "$(date --date='yesterday' '+%b %d')" >> $REPORT_FILE

# Проверка sudo активности
echo -e "\n[SUDO ACTIVITY - Last 24h]" >> $REPORT_FILE
grep "sudo:" /var/log/auth.log | grep "$(date '+%b %d')" >> $REPORT_FILE

# Проверка файлов с SUID
echo -e "\n[SUID FILES]" >> $REPORT_FILE
find / -perm -4000 -type f 2>/dev/null >> $REPORT_FILE

# Проверка пользователей с пустыми паролями
echo -e "\n[USERS WITH EMPTY PASSWORDS]" >> $REPORT_FILE
awk -F: '($2 == "") {print $1}' /etc/shadow >> $REPORT_FILE

# Отправка отчета
mail -s "Security Audit Report" admin@myapp.com < $REPORT_FILE
```

### 💾 Backup стратегии

**Автоматический backup скрипт:**
```bash
#!/bin/bash
# backup.sh

BACKUP_ROOT="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

# Функция backup базы данных
backup_database() {
    local db_name=$1
    local backup_dir="$BACKUP_ROOT/database"
    
    mkdir -p $backup_dir
    
    echo "Backing up database: $db_name"
    pg_dump $db_name | gzip > "$backup_dir/${db_name}_${DATE}.sql.gz"
    
    if [ $? -eq 0 ]; then
        echo "Database backup completed: ${db_name}_${DATE}.sql.gz"
    else
        echo "Database backup failed for: $db_name"
        return 1
    fi
}

# Функция backup приложения
backup_application() {
    local app_dir=$1
    local backup_dir="$BACKUP_ROOT/application"
    
    mkdir -p $backup_dir
    
    echo "Backing up application: $app_dir"
    tar -czf "$backup_dir/app_${DATE}.tar.gz" -C $(dirname $app_dir) $(basename $app_dir) \
        --exclude='*.log' --exclude='__pycache__' --exclude='.git'
    
    if [ $? -eq 0 ]; then
        echo "Application backup completed: app_${DATE}.tar.gz"
    else
        echo "Application backup failed"
        return 1
    fi
}

# Функция очистки старых backup'ов
cleanup_old_backups() {
    echo "Cleaning up backups older than $RETENTION_DAYS days"
    find $BACKUP_ROOT -name "*.gz" -mtime +$RETENTION_DAYS -delete
    find $BACKUP_ROOT -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete
}

# Основная логика
main() {
    echo "Starting backup process at $(date)"
    
    # Backup базы данных
    backup_database "myapp"
    
    # Backup приложения
    backup_application "/opt/myapp"
    
    # Очистка старых backup'ов
    cleanup_old_backups
    
    # Sync в облако (опционально)
    if command -v aws &> /dev/null; then
        aws s3 sync $BACKUP_ROOT s3://myapp-backups/$(hostname)/
    fi
    
    echo "Backup process completed at $(date)"
}

main 2>&1 | tee -a /var/log/backup.log
```

### 🚨 Мониторинг безопасности

**Интеграция с ELK для анализа логов:**
```ruby
# logstash-security.conf
input {
  file {
    path => "/var/log/auth.log"
    type => "auth"
  }
  file {
    path => "/var/log/fail2ban.log"
    type => "fail2ban"
  }
}

filter {
  if [type] == "auth" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:timestamp} %{HOSTNAME:hostname} %{WORD:program}(?:\[%{POSINT:pid}\])?: %{GREEDYDATA:message}" }
    }
    
    if "Failed password" in [message] {
      mutate {
        add_tag => ["failed_login"]
      }
      grok {
        match => { "message" => "Failed password for %{USERNAME:failed_user} from %{IP:source_ip}" }
      }
    }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "security-logs-%{+YYYY.MM.dd}"
  }
}
```

### ✅ Чек-лист Модуля 9
- [ ] Выполнил hardening операционной системы
- [ ] Настроил файрвол с минимальными правами
- [ ] Установил и настроил fail2ban
- [ ] Автоматизировал обновление SSL сертификатов
- [ ] Создал систему backup'ов и disaster recovery
- [ ] Настроил мониторинг безопасности и алерты

---

## ⚡ Модуль 10: Performance tuning (1-2 недели)

### Зачем это нужно?
- **Оптимизация ресурсов** для снижения затрат
- Улучшение отзывчивости приложений
- Подготовка к высоким нагрузкам
- Эффективное использование инфраструктуры

### 🔧 Оптимизация ядра Linux

**Системные параметры производительности:**
```bash
# /etc/sysctl.conf - основные оптимизации

# Сетевые оптимизации
net.core.rmem_max = 134217728
net.core.wmem_max = 134217728
net.ipv4.tcp_rmem = 4096 65536 134217728
net.ipv4.tcp_wmem = 4096 65536 134217728
net.core.netdev_max_backlog = 5000
net.core.somaxconn = 65535
net.ipv4.tcp_congestion_control = bbr
net.ipv4.tcp_slow_start_after_idle = 0

# Файловая система
fs.file-max = 2097152
fs.nr_open = 1048576

# Память
vm.swappiness = 10
vm.dirty_ratio = 15
vm.dirty_background_ratio = 5
vm.overcommit_memory = 1

# Применение изменений
sudo sysctl -p
```

### 🚀 Оптимизация веб-сервера

**Nginx production конфигурация:**
```nginx
# /etc/nginx/nginx.conf
user www-data;
worker_processes auto;                    # По числу CPU ядер
worker_rlimit_nofile 65535;
pid /run/nginx.pid;

events {
    worker_connections 4096;              # Максимум соединений на worker
    use epoll;                           # Эффективная модель событий
    multi_accept on;                     # Принимать несколько соединений
}

http {
    # Базовые оптимизации
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 30;
    keepalive_requests 100;
    
    # Gzip сжатие
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/javascript
        application/xml+rss
        application/json;
    
    # Кэширование
    open_file_cache max=10000 inactive=30s;
    open_file_cache_valid 60s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;
    
    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    
    upstream backend {
        least_conn;                      # Балансировка по нагрузке
        server 127.0.0.1:8001 weight=3;
        server 127.0.0.1:8002 weight=3;
        server 127.0.0.1:8003 weight=2;
        keepalive 32;                    # Постоянные соединения
    }
    
    server {
        listen 80;
        server_name myapp.com;
        
        # Security headers
        add_header X-Frame-Options DENY;
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        
        location /api/ {
            limit_req zone=api burst=20 nodelay;
            
            proxy_pass http://backend;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            
            # Timeouts
            proxy_connect_timeout 5s;
            proxy_send_timeout 10s;
            proxy_read_timeout 10s;
        }
        
        location /static/ {
            alias /var/www/static/;
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }
}
```

### 🗄️ Оптимизация PostgreSQL

**Основные параметры производительности:**
```sql
-- postgresql.conf оптимизации для 8GB RAM сервера

-- Память
shared_buffers = 2GB                    -- 25% от RAM
effective_cache_size = 6GB              -- 75% от RAM
work_mem = 64MB                         -- Для сортировок
maintenance_work_mem = 512MB            -- Для VACUUM, CREATE INDEX

-- Checkpoint и WAL
wal_buffers = 16MB
checkpoint_completion_target = 0.9
max_wal_size = 2GB
min_wal_size = 1GB

-- Параллелизм
max_worker_processes = 8
max_parallel_workers_per_gather = 4
max_parallel_workers = 8
max_parallel_maintenance_workers = 4

-- Соединения
max_connections = 200
```

**Мониторинг производительности БД:**
```sql
-- Топ медленных запросов
SELECT query, calls, total_time, mean_time, rows
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;

-- Использование индексов
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Размеры таблиц
SELECT schemaname, tablename,
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
FROM pg_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Активные соединения
SELECT state, count(*)
FROM pg_stat_activity
WHERE state IS NOT NULL
GROUP BY state;
```

### 📊 Профилирование приложений

**Python профилирование с cProfile:**
```python
# performance_monitor.py
import cProfile
import pstats
import functools
from flask import request, g
import time

def profile_endpoint(func):
    """Декоратор для профилирования эндпоинтов"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        if request.args.get('profile'):
            pr = cProfile.Profile()
            pr.enable()
            
            result = func(*args, **kwargs)
            
            pr.disable()
            stats = pstats.Stats(pr)
            stats.sort_stats('cumulative')
            
            # Сохранение результатов
            profile_file = f"/tmp/profile_{func.__name__}_{int(time.time())}.prof"
            stats.dump_stats(profile_file)
            
            return result
        else:
            return func(*args, **kwargs)
    return wrapper

# Middleware для измерения времени запросов
@app.before_request
def before_request():
    g.start_time = time.time()

@app.after_request
def after_request(response):
    diff = time.time() - g.start_time
    if diff > 1.0:  # Логировать медленные запросы
        logger.warning(f"Slow request: {request.path} took {diff:.2f}s")
    return response

# Использование
@app.route('/api/data')
@profile_endpoint
def get_data():
    return expensive_operation()
```

**Системное профилирование с perf:**
```bash
# Установка perf
sudo apt install linux-tools-common linux-tools-generic

# Профилирование CPU
sudo perf record -g python app.py
sudo perf report

# Профилирование конкретного процесса
sudo perf top -p $(pgrep python)

# Анализ cache misses
sudo perf stat -e cache-references,cache-misses python app.py

# Flame graph для визуализации
git clone https://github.com/brendangregg/FlameGraph
sudo perf record -F 99 -g python app.py
sudo perf script | ./FlameGraph/stackcollapse-perf.pl | ./FlameGraph/flamegraph.pl > flamegraph.svg
```

### 🔍 Анализ bottlenecks

**Комплексный скрипт диагностики:**
```bash
#!/bin/bash
# performance_check.sh

echo "=== PERFORMANCE ANALYSIS REPORT ==="
echo "Generated: $(date)"
echo

# CPU анализ
echo "[CPU UTILIZATION]"
top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}'
echo

echo "[TOP CPU PROCESSES]"
ps aux --sort=-%cpu | head -6
echo

# Память
echo "[MEMORY UTILIZATION]"
free -h
echo

echo "[TOP MEMORY PROCESSES]"
ps aux --sort=-%mem | head -6
echo

# Дисковая активность
echo "[DISK I/O]"
iostat -x 1 1 | grep -E "(Device|sda|nvme)"
echo

echo "[DISK USAGE]"
df -h | grep -E "(Filesystem|/dev/)"
echo

# Сетевая активность
echo "[NETWORK CONNECTIONS]"
netstat -an | awk '/^tcp/ {count[$ NF]++} END {for (state in count) print state, count[state]}'
echo

echo "[NETWORK I/O]"
cat /proc/net/dev | grep -E "(eth0|ens|enp)" | awk '{print $1, "RX:", $2, "TX:", $10}'
echo

# Анализ логов ошибок
echo "[RECENT ERRORS (last 100 lines)]"
tail -100 /var/log/nginx/error.log | grep -E "(error|warn)" | tail -5
echo

# Проверка swap
echo "[SWAP USAGE]"
swapon --show
echo

# Load average
echo "[SYSTEM LOAD]"
uptime
```

### ⚙️ Оптимизация приложений

**Кэширование с Redis:**
```python
# cache_manager.py
import redis
import json
import functools
from typing import Any, Optional

class CacheManager:
    def __init__(self, host='localhost', port=6379, db=0):
        self.redis_client = redis.Redis(host=host, port=port, db=db, decode_responses=True)
    
    def cache_result(self, ttl: int = 3600):
        """Декоратор для кэширования результатов функций"""
        def decorator(func):
            @functools.wraps(func)
            def wrapper(*args, **kwargs):
                # Создание ключа кэша
                cache_key = f"{func.__name__}:{hash(str(args) + str(sorted(kwargs.items())))}"
                
                # Попытка получить из кэша
                cached_result = self.redis_client.get(cache_key)
                if cached_result:
                    return json.loads(cached_result)
                
                # Выполнение функции и кэширование результата
                result = func(*args, **kwargs)
                self.redis_client.setex(cache_key, ttl, json.dumps(result))
                
                return result
            return wrapper
        return decorator
    
    def invalidate_pattern(self, pattern: str):
        """Удаление кэша по шаблону"""
        keys = self.redis_client.keys(pattern)
        if keys:
            self.redis_client.delete(*keys)

# Использование
cache = CacheManager()

@cache.cache_result(ttl=1800)  # 30 минут
def get_user_profile(user_id: int):
    # Тяжелый запрос к БД
    return database.query_user_profile(user_id)

@cache.cache_result(ttl=300)   # 5 минут
def get_dashboard_stats():
    # Агрегация данных
    return calculate_complex_stats()
```

**Connection pooling для БД:**
```python
# db_pool.py
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool
import os

# Настройка connection pool
engine = create_engine(
    os.getenv('DATABASE_URL'),
    poolclass=QueuePool,
    pool_size=20,                    # Базовый размер пула
    max_overflow=30,                 # Дополнительные соединения
    pool_pre_ping=True,              # Проверка соединений
    pool_recycle=3600,               # Пересоздание через час
    echo=False                       # Отключить SQL логи в production
)

# Мониторинг пула соединений
def get_pool_status():
    pool = engine.pool
    return {
        'pool_size': pool.size(),
        'checked_in': pool.checkedin(),
        'checked_out': pool.checkedout(),
        'overflow': pool.overflow(),
        'invalid': pool.invalid()
    }
```

### 📈 Мониторинг производительности

**Дашборд метрик производительности:**
```bash
# Prometheus метрики для мониторинга производительности

# Время ответа приложения
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Throughput (запросов в секунду)
rate(http_requests_total[5m])

# Error rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])

# Database connection pool utilization
db_connections_active / db_connections_max

# Cache hit rate
redis_keyspace_hits / (redis_keyspace_hits + redis_keyspace_misses)

# System load
node_load1

# Memory utilization
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```

### 🔧 Автоматическая оптимизация

**Скрипт автоматической настройки:**
```bash
#!/bin/bash
# auto_tune.sh

# Определение ресурсов системы
TOTAL_RAM=$(free -m | awk 'NR==2{printf "%.0f", $2/1024}')
CPU_CORES=$(nproc)

echo "System specs: ${TOTAL_RAM}GB RAM, ${CPU_CORES} CPU cores"

# Настройка PostgreSQL
configure_postgresql() {
    local shared_buffers=$((TOTAL_RAM * 256))  # 25% от RAM в MB
    local effective_cache=$((TOTAL_RAM * 768)) # 75% от RAM в MB
    
    cat >> /etc/postgresql/*/main/postgresql.conf << EOF

# Auto-tuned settings
shared_buffers = ${shared_buffers}MB
effective_cache_size = ${effective_cache}MB
work_mem = 64MB
maintenance_work_mem = 512MB
max_worker_processes = $CPU_CORES
max_parallel_workers = $CPU_CORES
EOF
    
    systemctl restart postgresql
}

# Настройка Nginx
configure_nginx() {
    local worker_connections=$((CPU_CORES * 1024))
    
    sed -i "s/worker_processes.*/worker_processes $CPU_CORES;/" /etc/nginx/nginx.conf
    sed -i "s/worker_connections.*/worker_connections $worker_connections;/" /etc/nginx/nginx.conf
    
    systemctl reload nginx
}

# Настройка системных параметров
configure_system() {
    # Оптимизация файловых дескрипторов
    echo "* soft nofile 65535" >> /etc/security/limits.conf
    echo "* hard nofile 65535" >> /etc/security/limits.conf
    
    # Сетевые оптимизации
    cat >> /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 5000
net.ipv4.tcp_congestion_control = bbr
vm.swappiness = 10
EOF
    
    sysctl -p
}

# Выполнение настроек
if [ "$EUID" -ne 0 ]; then
    echo "This script must be run as root"
    exit 1
fi

configure_system
configure_nginx
configure_postgresql

echo "Auto-tuning completed. Please restart services if needed."
```

### ✅ Финальный чек-лист: Linux Expert

**Системное администрирование:**
- [ ] Могу оптимизировать ядро Linux для production нагрузок
- [ ] Настроил высокопроизводительный веб-сервер (Nginx)
- [ ] Оптимизировал базу данных для конкретной нагрузки
- [ ] Умею профилировать приложения и находить bottlenecks

**Мониторинг и анализ:**
- [ ] Настроил комплексный мониторинг производительности
- [ ] Создал алерты для критических метрик производительности
- [ ] Умею анализировать системные логи и метрики
- [ ] Могу проводить capacity planning

**Автоматизация:**
- [ ] Автоматизировал процессы оптимизации
- [ ] Создал скрипты для регулярного анализа производительности
- [ ] Настроил автоматическое масштабирование при необходимости

---

## 🎯 Заключение: Путь к Linux экспертизе

### Что вы освоили

За 10 модулей вы прошли путь от новичка до опытного Linux администратора:

1. **Командная строка** - основа всей работы в Linux
2. **Процессы и ресурсы** - понимание работы системы
3. **Безопасность** - защита production окружений
4. **Сети** - диагностика и настройка сетевых сервисов
5. **Веб-серверы и БД** - основа современных приложений
6. **Контейнеризация** - современный подход к развертыванию
7. **Автоматизация** - DevOps практики и CI/CD
8. **Мониторинг** - наблюдаемость и алертинг
9. **Безопасность в production** - защита от угроз
10. **Performance tuning** - оптимизация для высоких нагрузок

### Дальнейшее развитие

**Специализация по направлениям:**
- **SRE (Site Reliability Engineering)** - фокус на надежности систем
- **DevOps Engineer** - автоматизация и CI/CD
- **Security Engineer** - специализация на информационной безопасности
- **Platform Engineer** - создание платформ для разработчиков

**Продвинутые темы:**
- Kubernetes и оркестрация контейнеров
- Infrastructure as Code (Terraform, CloudFormation)
- Service Mesh (Istio, Linkerd)
- Observability (OpenTelemetry, Jaeger)

### Практические советы

1. **Практикуйтесь регулярно** - создайте домашнюю лабораторию
2. **Читайте документацию** - man pages ваши лучшие друзья
3. **Участвуйте в сообществах** - Stack Overflow, Reddit r/sysadmin
4. **Изучайте инциденты** - анализируйте postmortem отчеты крупных компаний
5. **Автоматизируйте все** - если делаете что-то дважды, напишите скрипт

**Полезные ресурсы:**
- The Linux Documentation Project (tldp.org)
- Red Hat Enterprise Linux Documentation
- ArchWiki (отличные гайды даже для других дистрибутивов)
- Linux Performance Tools (brendangregg.com)

Теперь вы обладаете знаниями и навыками для уверенной работы с Linux в production окружениях. Продолжайте практиковаться и изучать новые технологии!

**Happy Linux-ing! 🐧**