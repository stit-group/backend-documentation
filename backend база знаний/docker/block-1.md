# 📚 Блок 1: Основы контейнеризации
*Время изучения: 2-3 недели | 8-12 часов в неделю*

---

## 🎯 Цели блока
После изучения этого блока вы будете:
- Понимать фундаментальные концепции контейнеризации
- Знать историю и эволюцию технологий изоляции
- Уметь устанавливать и настраивать Docker
- Разбираться в архитектуре Docker и его компонентах
- Выполнять базовые операции с контейнерами и образами

---

## 📖 Глава 1.1: Введение в контейнеризацию
**Время изучения:** 3-4 дня

### 🤔 Что такое контейнеризация?

Контейнеризация - это технология виртуализации на уровне операционной системы, которая позволяет упаковать приложение и все его зависимости в изолированную среду, называемую контейнером.

```
┌─────────────────────────────────────────────────────────────┐
│                    ТРАДИЦИОННОЕ РАЗВЕРТЫВАНИЕ               │
├─────────────────────────────────────────────────────────────┤
│  Приложение A   │  Приложение B   │  Приложение C          │
│  ┌─────────────┐  │  ┌─────────────┐  │  ┌─────────────┐      │
│  │   Java 8    │  │  │   Python    │  │  │   Node.js   │      │
│  │   Tomcat    │  │  │   Django    │  │  │   Express   │      │
│  │   MySQL     │  │  │   PostgreSQL│  │  │   MongoDB   │      │
│  └─────────────┘  │  └─────────────┘  │  └─────────────┘      │
├─────────────────────────────────────────────────────────────┤
│              ОПЕРАЦИОННАЯ СИСТЕМА (Linux/Windows)           │
├─────────────────────────────────────────────────────────────┤
│                     ФИЗИЧЕСКОЕ ЖЕЛЕЗО                       │
└─────────────────────────────────────────────────────────────┘

                              ⬇️ ПРОБЛЕМЫ

❌ Конфликты зависимостей между приложениями
❌ "Работает у меня на машине" - проблемы переносимости
❌ Сложное управление различными версиями библиотек
❌ Неэффективное использование ресурсов
```

### 🏗️ Как контейнеризация решает эти проблемы?

```
┌─────────────────────────────────────────────────────────────┐
│                  КОНТЕЙНЕРИЗИРОВАННОЕ РАЗВЕРТЫВАНИЕ          │
├─────────────────────────────────────────────────────────────┤
│ ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│ │ КОНТЕЙНЕР A │  │ КОНТЕЙНЕР B │  │ КОНТЕЙНЕР C │           │
│ │┌───────────┐│  │┌───────────┐│  │┌───────────┐│           │
│ ││Приложение ││  ││Приложение ││  ││Приложение ││           │
│ ││   Java 8  ││  ││  Python   ││  ││  Node.js  ││           │
│ ││  Tomcat   ││  ││  Django   ││  ││  Express  ││           │
│ ││  Библиотеки││  ││ Библиотеки││  ││ Библиотеки││           │
│ │└───────────┘│  │└───────────┘│  │└───────────┘│           │
│ └─────────────┘  └─────────────┘  └─────────────┘           │
├─────────────────────────────────────────────────────────────┤
│            CONTAINER RUNTIME (Docker Engine)                │
├─────────────────────────────────────────────────────────────┤
│              ОПЕРАЦИОННАЯ СИСТЕМА (Linux)                   │
├─────────────────────────────────────────────────────────────┤
│                     ФИЗИЧЕСКОЕ ЖЕЛЕЗО                       │
└─────────────────────────────────────────────────────────────┘

                              ⬇️ ПРЕИМУЩЕСТВА

✅ Полная изоляция приложений и их зависимостей
✅ Одинаковое поведение на всех средах (dev, test, prod)
✅ Легкая масштабируемость и развертывание
✅ Эффективное использование ресурсов
```

### 🆚 Контейнеры vs Виртуальные машины

Понимание различий между контейнерами и виртуальными машинами критически важно:

```
┌─────────────────────────────────────────────────────────────┐
│                    ВИРТУАЛЬНЫЕ МАШИНЫ                       │
├─────────────────────────────────────────────────────────────┤
│ ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│ │     ВМ 1    │  │     ВМ 2    │  │     ВМ 3    │           │
│ │┌───────────┐│  │┌───────────┐│  │┌───────────┐│           │
│ ││Приложение ││  ││Приложение ││  ││Приложение ││           │
│ ││    ...    ││  ││    ...    ││  ││    ...    ││           │
│ │├───────────┤│  │├───────────┤│  │├───────────┤│           │
│ ││Guest OS   ││  ││Guest OS   ││  ││Guest OS   ││  ⭐ КАЖДАЯ ВМ│
│ ││(Linux)    ││  ││(Windows)  ││  ││(Linux)    ││   ИМЕЕТ   │
│ │└───────────┘│  │└───────────┘│  │└───────────┘│   СВОЮ ОС │
│ └─────────────┘  └─────────────┘  └─────────────┘           │
├─────────────────────────────────────────────────────────────┤
│                    ГИПЕРВИЗОР                               │
├─────────────────────────────────────────────────────────────┤
│              ХОСТОВАЯ ОПЕРАЦИОННАЯ СИСТЕМА                  │
├─────────────────────────────────────────────────────────────┤
│                     ФИЗИЧЕСКОЕ ЖЕЛЕЗО                       │
└─────────────────────────────────────────────────────────────┘

                                VS

┌─────────────────────────────────────────────────────────────┐
│                      КОНТЕЙНЕРЫ                             │
├─────────────────────────────────────────────────────────────┤
│ ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│ │ КОНТЕЙНЕР 1 │  │ КОНТЕЙНЕР 2 │  │ КОНТЕЙНЕР 3 │           │
│ │┌───────────┐│  │┌───────────┐│  │┌───────────┐│           │
│ ││Приложение ││  ││Приложение ││  ││Приложение ││  ⭐ ОБЩАЯ  │
│ ││Библиотеки ││  ││Библиотеки ││  ││Библиотеки ││   ОС ДЛЯ  │
│ ││   ...     ││  ││   ...     ││  ││   ...     ││   ВСЕХ    │
│ │└───────────┘│  │└───────────┘│  │└───────────┘│           │
│ └─────────────┘  └─────────────┘  └─────────────┘           │
├─────────────────────────────────────────────────────────────┤
│            CONTAINER RUNTIME (Docker)                       │
├─────────────────────────────────────────────────────────────┤
│              ХОСТОВАЯ ОПЕРАЦИОННАЯ СИСТЕМА                  │
├─────────────────────────────────────────────────────────────┤
│                     ФИЗИЧЕСКОЕ ЖЕЛЕЗО                       │
└─────────────────────────────────────────────────────────────┘
```

**Ключевые различия:**

| Характеристика | Виртуальные машины | Контейнеры |
|---|---|---|
| **Размер** | Гигабайты (включают полную ОС) | Мегабайты (только приложение) |
| **Время запуска** | Минуты | Секунды |
| **Изоляция** | Полная (аппаратный уровень) | Процесс-уровень |
| **Использование ресурсов** | Высокое (накладные расходы ОС) | Низкое |
| **Портативность** | Ограниченная | Высокая |

### 📜 История развития: от chroot до Docker

Понимание эволюции поможет оценить современные технологии:

```
1979: chroot
┌─────────────┐
│   chroot    │ ⭐ Изменение корневой директории
│   jail      │   для процесса
└─────────────┘

         ⬇️

2000: FreeBSD Jails
┌─────────────┐
│  FreeBSD    │ ⭐ Виртуализация на уровне ОС
│   Jails     │   Изоляция процессов, файлов, сети
└─────────────┘

         ⬇️

2004: Solaris Containers
┌─────────────┐
│  Solaris    │ ⭐ Zones - полноценные контейнеры
│ Containers  │   с управлением ресурсами
└─────────────┘

         ⬇️

2006: Process Containers → Control Groups (cgroups)
┌─────────────┐
│   cgroups   │ ⭐ Ограничение и учет ресурсов
│ (Google)    │   для групп процессов
└─────────────┘

         ⬇️

2008: LXC (Linux Containers)
┌─────────────┐
│     LXC     │ ⭐ Объединение cgroups + namespaces
│  (Linux)    │   Пользовательский интерфейс
└─────────────┘

         ⬇️

2013: Docker
┌─────────────┐
│   DOCKER    │ ⭐ Революция в контейнеризации!
│             │   - Простота использования
│             │   - Образы и слои
│             │   - Экосистема инструментов
└─────────────┘
```

### 🔍 Проблемы, которые решает контейнеризация

**1. Проблема "Работает у меня на машине"**
```
Разработчик:                   Production сервер:
┌─────────────────┐           ┌─────────────────┐
│ Ubuntu 20.04    │    VS     │ CentOS 7        │
│ Python 3.9      │           │ Python 3.6      │
│ pip 21.0        │           │ pip 18.0        │
│ lib-ssl 1.1     │           │ lib-ssl 1.0     │
└─────────────────┘           └─────────────────┘
        ⬇️                            ⬇️
    ✅ Работает               ❌ Не работает!

С КОНТЕЙНЕРАМИ:
┌─────────────────┐           ┌─────────────────┐
│ Docker контейнер│    →      │ Docker контейнер│
│ Ubuntu 20.04    │           │ Ubuntu 20.04    │
│ Python 3.9      │           │ Python 3.9      │
│ pip 21.0        │           │ pip 21.0        │
│ lib-ssl 1.1     │           │ lib-ssl 1.1     │
└─────────────────┘           └─────────────────┘
        ⬇️                            ⬇️
    ✅ Работает               ✅ Работает!
```

**2. Управление зависимостями**
```
БЕЗ КОНТЕЙНЕРОВ:
┌─────────────────────────────────────────┐
│ Сервер с несколькими приложениями       │
├─────────────────────────────────────────┤
│ App A → нужен Python 2.7               │
│ App B → нужен Python 3.9               │
│ App C → нужен Python 3.6               │
│                                         │
│ ❌ КОНФЛИКТ ВЕРСИЙ!                     │
└─────────────────────────────────────────┘

С КОНТЕЙНЕРАМИ:
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│ Контейнер A │ │ Контейнер B │ │ Контейнер C │
│ Python 2.7  │ │ Python 3.9  │ │ Python 3.6  │
│ ✅ Изоляция │ │ ✅ Изоляция │ │ ✅ Изоляция │
└─────────────┘ └─────────────┘ └─────────────┘
```

**3. Масштабирование**
```
ТРАДИЦИОННЫЙ ПОДХОД:
Нагрузка растет → Купить новый сервер → Настроить → Развернуть
(Время: дни/недели)

С КОНТЕЙНЕРАМИ:
Нагрузка растет → docker run my-app → Готово!
(Время: секунды)

АВТОМАСШТАБИРОВАНИЕ:
┌─────────┐    Нагрузка ↑    ┌─────────┐┌─────────┐┌─────────┐
│ 1 копия │ ─────────────→   │ 3 копии ││ 3 копии ││ 3 копии │
└─────────┘                  └─────────┘└─────────┘└─────────┘
```

### 💡 Практическое задание
Создайте диаграмму сравнения традиционного развертывания и контейнеризации для вашего текущего проекта. Подумайте:
- Какие проблемы вы испытываете сейчас?
- Как контейнеры могли бы их решить?
- Какие преимущества это принесло бы вашей команде?

### 📝 Эссе на 500 слов: "Почему контейнеры изменили мир разработки"

**Структура для написания:**
1. **Введение** - Состояние разработки до контейнеров
2. **Основная часть** - Ключевые изменения:
   - DevOps практики
   - Микросервисы
   - Облачные технологии
   - CI/CD автоматизация
3. **Заключение** - Будущее с контейнерами

---

## 🔧 Глава 1.2: Установка и первое знакомство с Docker
**Время изучения:** 4-5 дней

### 🏗️ Архитектура Docker

Docker использует клиент-серверную архитектуру, которую важно понимать:

```
┌─────────────────────────────────────────────────────────────┐
│                    DOCKER АРХИТЕКТУРА                       │
└─────────────────────────────────────────────────────────────┘

┌─────────────────┐                    ┌──────────────────────┐
│                 │                    │                      │
│  DOCKER CLIENT  │◄──── REST API ────►│   DOCKER DAEMON      │
│                 │   (HTTP/Socket)    │     (dockerd)        │
│ ┌─────────────┐ │                    │                      │
│ │docker build │ │                    │ ┌──────────────────┐ │
│ │docker pull  │ │                    │ │                  │ │
│ │docker run   │ │                    │ │    КОНТЕЙНЕРЫ    │ │
│ │docker push  │ │                    │ │                  │ │
│ └─────────────┘ │                    │ │ ┌──┐ ┌──┐ ┌──┐  │ │
│                 │                    │ │ │C1│ │C2│ │C3│  │ │
└─────────────────┘                    │ │ └──┘ └──┘ └──┘  │ │
                                       │ └──────────────────┘ │
                                       │                      │
                                       │ ┌──────────────────┐ │
                                       │ │                  │ │
                                       │ │     ОБРАЗЫ       │ │
                                       │ │                  │ │
                                       │ │ [nginx] [ubuntu] │ │
                                       │ │ [python] [node]  │ │
                                       │ └──────────────────┘ │
                                       └──────────────────────┘

┌────────────────────────────────────────────────────────────┐
│                    DOCKER REGISTRY                         │
│                   (Docker Hub, ECR)                        │
│                                                            │
│  🐳 nginx    🐍 python    ⚛️ react    🚀 node            │
│  🗄️ postgres 📊 grafana   🔍 elasticsearch               │
└────────────────────────────────────────────────────────────┘
```

**Компоненты системы:**

**1. Docker Client (docker)**
- Интерфейс командной строки
- Отправляет команды Docker Daemon
- Может работать удаленно

**2. Docker Daemon (dockerd)**
- Основной процесс Docker
- Управляет образами, контейнерами, сетями
- Прослушивает API запросы

**3. Docker Registry**
- Хранилище Docker образов
- Docker Hub - публичный реестр
- Частные реестры (ECR, Harbor)

### 🖥️ Docker Desktop vs Docker Engine

```
┌─────────────────────────────────────────────────────────────┐
│                    DOCKER DESKTOP                           │
│                  (Windows/macOS)                            │
├─────────────────────────────────────────────────────────────┤
│ ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│ │     GUI     │  │  Settings   │  │  Dashboard  │          │
│ │ Management  │  │  Resources  │  │  Monitoring │          │
│ └─────────────┘  └─────────────┘  └─────────────┘          │
├─────────────────────────────────────────────────────────────┤
│                 DOCKER ENGINE                               │
├─────────────────────────────────────────────────────────────┤
│                ВИРТУАЛЬНАЯ МАШИНА                           │
│               (Linux VM на Windows/Mac)                     │
└─────────────────────────────────────────────────────────────┘

                              VS

┌─────────────────────────────────────────────────────────────┐
│                   DOCKER ENGINE                             │
│                    (Linux сервер)                           │
├─────────────────────────────────────────────────────────────┤
│ ┌─────────────────────────────────────────────────────────┐ │
│ │                Docker Daemon                            │ │
│ │                                                         │ │
│ │  Контейнеры  │  Образы  │  Сети  │  Тома               │ │
│ └─────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│                   LINUX KERNEL                              │
└─────────────────────────────────────────────────────────────┘
```

**Когда использовать что:**

**Docker Desktop** - идеален для:
- Разработки на Windows/macOS
- Обучения и экспериментов
- Локальной разработки
- GUI управления

**Docker Engine** - подходит для:
- Production серверов
- Linux окружений
- CI/CD пайплайнов
- Максимальной производительности

### ⚙️ Системные требования

**Минимальные требования:**
```
┌─────────────────────────────────────────┐
│             WINDOWS                     │
├─────────────────────────────────────────┤
│ • Windows 10/11 Pro, Enterprise        │
│ • WSL 2 включен                        │
│ • Hyper-V поддержка                     │
│ • 4GB RAM                               │
│ • Виртуализация в BIOS                  │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│              macOS                      │
├─────────────────────────────────────────┤
│ • macOS 10.15 или новее                │
│ • Intel или Apple Silicon              │
│ • 4GB RAM                               │
│ • VirtualBox (если не Desktop)          │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│              LINUX                      │
├─────────────────────────────────────────┤
│ • Ubuntu 18.04+, CentOS 7+, Debian 9+  │
│ • Kernel 3.10+                          │
│ • 2GB RAM                               │
│ • 64-bit архитектура                    │
└─────────────────────────────────────────┘
```

### 🛠️ Процесс установки

**Пошаговая установка на Ubuntu:**

```
ШАГ 1: Подготовка системы
┌─────────────────────────────────────────┐
│ sudo apt-get update                     │
│ sudo apt-get install apt-transport-https│
│ sudo apt-get install ca-certificates    │
│ sudo apt-get install curl               │
│ sudo apt-get install gnupg-agent        │
│ sudo apt-get install software-properties-common│
└─────────────────────────────────────────┘

ШАГ 2: Добавление GPG ключа
┌─────────────────────────────────────────┐
│ curl -fsSL https://download.docker.com/linux/ubuntu/gpg │
│ sudo apt-key add -                      │
└─────────────────────────────────────────┘

ШАГ 3: Добавление репозитория
┌─────────────────────────────────────────┐
│ sudo add-apt-repository \               │
│ "deb [arch=amd64] https://download.docker.com/linux/ubuntu │
│ $(lsb_release -cs) stable"              │
└─────────────────────────────────────────┘

ШАГ 4: Установка Docker
┌─────────────────────────────────────────┐
│ sudo apt-get update                     │
│ sudo apt-get install docker-ce          │
│ sudo apt-get install docker-ce-cli      │
│ sudo apt-get install containerd.io      │
└─────────────────────────────────────────┘

ШАГ 5: Проверка установки
┌─────────────────────────────────────────┐
│ sudo docker --version                   │
│ sudo docker run hello-world             │
└─────────────────────────────────────────┘
```

### 🏁 Первые команды

После установки проверим, что все работает:

```
ПРОВЕРКА ВЕРСИИ:
┌─────────────────────────────────────────┐
│ $ docker --version                      │
│ Docker version 24.0.0, build 98fdcd7    │
└─────────────────────────────────────────┘

ИНФОРМАЦИЯ О СИСТЕМЕ:
┌─────────────────────────────────────────┐
│ $ docker info                           │
│                                         │
│ Client:                                 │
│  Version:    24.0.0                     │
│  API version: 1.43                      │
│                                         │
│ Server:                                 │
│  Containers: 2                          │
│  Running: 1                             │
│  Images: 5                              │
│  Storage Driver: overlay2               │
└─────────────────────────────────────────┘

ПЕРВЫЙ КОНТЕЙНЕР:
┌─────────────────────────────────────────┐
│ $ docker run hello-world                │
│                                         │
│ Hello from Docker!                      │
│ This message shows that your            │
│ installation appears to be working      │
│ correctly.                              │
└─────────────────────────────────────────┘
```

### 🔐 Настройка прав доступа (Linux)

По умолчанию Docker требует sudo. Для удобства добавим пользователя в группу:

```
ДОБАВЛЕНИЕ В ГРУППУ DOCKER:
┌─────────────────────────────────────────┐
│ sudo usermod -aG docker $USER           │
│                                         │
│ # Перелогиниться или выполнить:         │
│ newgrp docker                           │
│                                         │
│ # Проверить:                            │
│ docker run hello-world                  │
│ # Теперь без sudo!                      │
└─────────────────────────────────────────┘
```

### 🚀 Автозапуск Docker

```
ВКЛЮЧЕНИЕ АВТОЗАПУСКА:
┌─────────────────────────────────────────┐
│ # Systemd (Ubuntu, CentOS):             │
│ sudo systemctl enable docker            │
│ sudo systemctl start docker             │
│                                         │
│ # Проверка статуса:                     │
│ sudo systemctl status docker            │
└─────────────────────────────────────────┘
```

### ✅ Чек-лист проверки установки

После установки убедитесь, что:
- [ ] `docker --version` показывает версию
- [ ] `docker info` выводит информацию о системе
- [ ] `docker run hello-world` работает без ошибок
- [ ] Можете запускать команды без sudo (Linux)
- [ ] Docker автоматически запускается при загрузке системы

### 🎯 Практическое задание

1. **Установите Docker** на свою ОС согласно инструкции
2. **Запустите интерактивный контейнер Ubuntu:**
   ```bash
   docker run -it ubuntu bash
   ```
3. **Исследуйте контейнер:**
   - Выполните команды: `ls`, `ps aux`, `cat /etc/os-release`
   - Установите пакет: `apt update && apt install nano`
   - Создайте файл: `echo "Hello Docker" > test.txt`
   - Выйдите: `exit`
4. **Запустите контейнер снова** - что произошло с файлом?

---

## 🐳 Глава 1.3: Основные концепции Docker
**Время изучения:** 5-6 дней

### 📦 Images (Образы) - Основа всего

Образ Docker - это неизменяемый шаблон, содержащий все необходимое для запуска приложения:

```
┌─────────────────────────────────────────────────────────────┐
│                     DOCKER IMAGE                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🏗️ ОБРАЗ = РЕЦЕПТ ДЛЯ СОЗДАНИЯ КОНТЕЙНЕРА                 │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Операционная система (базовый слой)              │    │
│  │ • Системные библиотеки                             │    │
│  │ • Среда выполнения (Python, Node.js, Java)         │    │
│  │ • Зависимости приложения                           │    │
│  │ • Исходный код приложения                          │    │
│  │ • Конфигурационные файлы                           │    │
│  │ • Команда запуска                                  │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  📚 ОБРАЗ - READ-ONLY (неизменяемый)                       │
│  🚀 Из одного образа можно создать множество контейнеров   │
└─────────────────────────────────────────────────────────────┘
```

### 🔄 Containers (Контейнеры) - Живые экземпляры

Контейнер - это запущенный экземпляр образа:

```
┌─────────────────────────────────────────────────────────────┐
│                  IMAGE → CONTAINER                          │
└─────────────────────────────────────────────────────────────┘

📦 ОБРАЗ (nginx)           🏃 КОНТЕЙНЕР (работающий nginx)
┌─────────────────┐       ┌──────────────────────────────────┐
│  nginx:latest   │  ──►  │ ┌──────────────────────────────┐ │
│                 │ run   │ │     Writable Layer           │ │
│ ┌─────────────┐ │       │ │ (логи, временные файлы)      │ │
│ │ Layer 4     │ │       │ └──────────────────────────────┘ │
│ │ nginx config│ │       │ ┌──────────────────────────────┐ │
│ ├─────────────┤ │       │ │        Image Layers          │ │
│ │ Layer 3     │ │       │ │      (read-only)             │ │
│ │ nginx binary│ │       │ │                              │ │
│ ├─────────────┤ │       │ │ Layer 4: nginx config        │ │
│ │ Layer 2     │ │       │ │ Layer 3: nginx binary        │ │
│ │ libraries   │ │       │ │ Layer 2: libraries           │ │
│ ├─────────────┤ │       │ │ Layer 1: Ubuntu base         │ │
│ │ Layer 1     │ │       │ └──────────────────────────────┘ │
│ │ Ubuntu base │ │       │                                  │
│ └─────────────┘ │       │ 🔗 PID: 1234                    │
│                 │       │ 🌐 IP: 172.17.0.2               │
│                 │       │ 📁 Volumes: /data               │
└─────────────────┘       └──────────────────────────────────┘
```

### 🍰 Слои образов (Image Layers)

Docker использует многослойную архитектуру для эффективного хранения:

```
┌─────────────────────────────────────────────────────────────┐
│                    СЛОЕВАЯ АРХИТЕКТУРА                      │
└─────────────────────────────────────────────────────────────┘

СОЗДАНИЕ ОБРАЗА:
                    Dockerfile                     Образ
┌─────────────────┐              ┌─────────────────────────────┐
│ FROM ubuntu     │─────────────►│ Layer 1: Ubuntu (64MB)     │
│ RUN apt update  │              ├─────────────────────────────┤
│ RUN apt install │─────────────►│ Layer 2: Updates (12MB)    │
│ COPY app.py     │              ├─────────────────────────────┤
│ CMD python app  │─────────────►│ Layer 3: App files (2MB)   │
└─────────────────┘              └─────────────────────────────┘

ПРЕИМУЩЕСТВА СЛОЕВ:
┌─────────────────────────────────────────────────────────────┐
│ 1. 🎯 КЕШИРОВАНИЕ                                           │
│    Изменили только app.py? Пересоздается только Layer 3!   │
│                                                             │
│ 2. 💾 ЭКОНОМИЯ МЕСТА                                        │
│    Layer 1 (Ubuntu) используется многими образами          │
│                                                             │
│ 3. ⚡ БЫСТРАЯ ЗАГРУЗКА                                      │
│    Скачиваются только новые слои                           │
└─────────────────────────────────────────────────────────────┘

СОВМЕСТНОЕ ИСПОЛЬЗОВАНИЕ:
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   nginx:latest  │  │  ubuntu:20.04   │  │   my-app:v1.0   │
├─────────────────┤  ├─────────────────┤  ├─────────────────┤
│ Layer 3: nginx  │  │ Layer 2: updates│  │ Layer 3: my-app │
├─────────────────┤  ├─────────────────┤  ├─────────────────┤
│ Layer 2: updates│◄─┤ Layer 1: Ubuntu │─►│ Layer 2: updates│
├─────────────────┤  └─────────────────┘  ├─────────────────┤
│ Layer 1: Ubuntu │◄─────────────────────►│ Layer 1: Ubuntu │
└─────────────────┘                       └─────────────────┘
        ⬆️                                        ⬆️
   ОБЩИЙ СЛОЙ                               ОБЩИЙ СЛОЙ
```

### 🔄 Жизненный цикл контейнера

Понимание состояний контейнера критически важно:

```
┌─────────────────────────────────────────────────────────────┐
│                ЖИЗНЕННЫЙ ЦИКЛ КОНТЕЙНЕРА                    │
└─────────────────────────────────────────────────────────────┘

                    docker run
       📦 IMAGE ─────────────────► 🏃 RUNNING
                                      │
                                      │ docker stop
                                      ▼
                                  ⏸️ STOPPED
                                      │
                   docker start       │ docker rm
                       ▲              ▼
                       └──────── 🗑️ DELETED

ДЕТАЛЬНЫЕ СОСТОЯНИЯ:
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  CREATED     ──► RUNNING ──► PAUSED ──► STOPPED ──► DELETED │
│     ▲            │   ▲        │          │   ▲         ▲    │
│     │            │   │        │          │   │         │    │
│  docker         │   │    docker       │   │      docker │   │
│  create         │   │    pause        │   │      start  │   │
│                 │   │                 │   │             │   │
│              docker │             docker │         docker  │
│              stop   │             unpause │         rm      │
│                     │                     │                 │
│                  docker                docker               │
│                  kill                  restart              │
│                                                             │
└─────────────────────────────────────────────────────────────┘

КОМАНДЫ УПРАВЛЕНИЯ:
┌─────────────────────────────────────────────────────────────┐
│ docker create  → Создать, но не запускать                  │
│ docker start   → Запустить остановленный контейнер         │
│ docker stop    → Graceful остановка (SIGTERM → SIGKILL)    │
│ docker kill    → Принудительная остановка (SIGKILL)        │
│ docker restart → stop + start                              │
│ docker pause   → Заморозить процессы (SIGSTOP)             │
│ docker unpause → Разморозить процессы (SIGCONT)            │
│ docker rm      → Удалить остановленный контейнер           │
└─────────────────────────────────────────────────────────────┘
```

### 🌐 Docker Hub - Центральный реестр

Docker Hub - это облачный сервис для хранения и распространения образов:

```
┌─────────────────────────────────────────────────────────────┐
│                      DOCKER HUB                             │
│                   hub.docker.com                            │
└─────────────────────────────────────────────────────────────┘

🏢 ОФИЦИАЛЬНЫЕ ОБРАЗЫ:
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   ubuntu        │  │     nginx       │  │    postgres     │
│   ⭐ Verified   │  │   ⭐ Verified   │  │  ⭐ Verified    │
│   📊 1B+ pulls  │  │   📊 1B+ pulls  │  │  📊 100M+ pulls │
└─────────────────┘  └─────────────────┘  └─────────────────┘

👥 СООБЩЕСТВО:
┌─────────────────────────────────────────────────────────────┐
│ username/repository-name                                    │
│                                                             │
│ Примеры:                                                    │
│ • bitnami/nginx                                             │
│ • grafana/grafana                                           │
│ • jenkins/jenkins                                           │
└─────────────────────────────────────────────────────────────┘

🏷️ СИСТЕМА ТЕГОВ:
┌─────────────────────────────────────────────────────────────┐
│ nginx:latest      ← Последняя стабильная версия            │
│ nginx:1.21        ← Конкретная версия                      │
│ nginx:1.21-alpine ← Версия + вариант (Alpine Linux)        │
│ nginx:stable      ← Стабильный релиз                       │
│ nginx:mainline    ← Разрабатываемая версия                 │
└─────────────────────────────────────────────────────────────┘

СТРУКТУРА ТЕГА:
[registry/]namespace/repository[:tag]
│          │         │          │
│          │         │          └─ По умолчанию: latest
│          │         └─ Имя образа
│          └─ Пользователь/организация
└─ По умолчанию: docker.io (Docker Hub)

Примеры:
• nginx = docker.io/library/nginx:latest
• ubuntu:20.04 = docker.io/library/ubuntu:20.04
• myuser/myapp:v1.0 = docker.io/myuser/myapp:v1.0
```

### 📊 Copy-on-Write (CoW) - Эффективность хранения

Docker использует CoW для экономии места и быстродействия:

```
┌─────────────────────────────────────────────────────────────┐
│                    COPY-ON-WRITE                            │
└─────────────────────────────────────────────────────────────┘

БЕЗ CoW (традиционные ВМ):
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│      ВМ 1       │  │      ВМ 2       │  │      ВМ 3       │
├─────────────────┤  ├─────────────────┤  ├─────────────────┤
│ OS Copy (2GB)   │  │ OS Copy (2GB)   │  │ OS Copy (2GB)   │
│ App A (100MB)   │  │ App B (200MB)   │  │ App C (150MB)   │
└─────────────────┘  └─────────────────┘  └─────────────────┘
     2.1 GB             2.2 GB             2.15 GB
                    ИТОГО: 6.45 GB 🔺

С CoW (Docker):
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  Контейнер 1    │  │  Контейнер 2    │  │  Контейнер 3    │
├─────────────────┤  ├─────────────────┤  ├─────────────────┤
│ W: App A (100MB)│  │ W: App B (200MB)│  │ W: App C (150MB)│
│ R: ────────────►│  │ R: ────────────►│  │ R: ────────────►│
└─────────────────┘  └─────────────────┘  └─────────────────┘
                                │
                                ▼
                    ┌─────────────────────┐
                    │   ОБЩИЙ БАЗОВЫЙ     │
                    │   ОБРАЗ (2GB)       │
                    │   (read-only)       │
                    └─────────────────────┘
                    ИТОГО: 2.45 GB ✅

КАК РАБОТАЕТ CoW:
┌─────────────────────────────────────────────────────────────┐
│ 1. 📖 ЧТЕНИЕ: Читаем из общего слоя                        │
│                                                             │
│ 2. ✏️ ЗАПИСЬ: Копируем файл в writable layer контейнера    │
│                                                             │
│ 3. 🔄 ПОСЛЕДУЮЩИЕ ОПЕРАЦИИ: Работаем с копией              │
└─────────────────────────────────────────────────────────────┘
```

### 🎯 Практическое задание: Исследование Docker Hub

1. **Поиск образов:**
   ```bash
   docker search nginx
   docker search python
   ```

2. **Анализ популярных образов:**
   - Найдите топ-5 образов Node.js
   - Изучите их описания и теги
   - Сравните размеры (alpine vs regular)

3. **Загрузка и исследование:**
   ```bash
   docker pull nginx
   docker pull nginx:alpine
   docker images  # Сравните размеры
   ```

4. **Анализ слоев:**
   ```bash
   docker history nginx
   docker inspect nginx
   ```

### 📋 Проект недели: Простой веб-сервер

**Цель:** Создать HTML страницу и подать её через nginx контейнер

**Шаги:**
1. Создайте HTML файл с личной информацией
2. Запустите nginx контейнер
3. Скопируйте файл в контейнер
4. Проверьте работу в браузере
5. Исследуйте логи и процессы

**Команды для выполнения:**
```bash
# Создание HTML
echo "<h1>Привет от $(whoami)!</h1>" > index.html

# Запуск nginx
docker run -d --name my-web -p 8080:80 nginx

# Копирование файла
docker cp index.html my-web:/usr/share/nginx/html/

# Проверка
curl http://localhost:8080

# Исследование
docker logs my-web
docker exec my-web ps aux
```

---

## 🎓 Резюме блока

### ✅ Что мы изучили:

1. **Концепции контейнеризации:**
   - Проблемы традиционного развертывания
   - Решения через контейнеры
   - Сравнение с виртуальными машинами

2. **Архитектуру Docker:**
   - Client-Server модель
   - Daemon и CLI
   - Registry система

3. **Основные компоненты:**
   - Images (образы) и их слои
   - Containers (контейнеры) и их состояния
   - Docker Hub и система тегов

4. **Практические навыки:**
   - Установка и настройка
   - Базовые команды
   - Работа с образами

### 🎯 Контрольные вопросы:

1. В чем основное отличие контейнера от виртуальной машины?
2. Что такое слой образа и как работает Copy-on-Write?
3. Какие состояния может иметь контейнер?
4. Как образы связаны с контейнерами?
5. Что означает тег "latest" и почему его не рекомендуют в production?

### 📈 Готовность к следующему блоку:

Вы готовы к изучению Блока 2, если можете:
- [ ] Объяснить преимущества контейнеризации
- [ ] Установить Docker на любую ОС
- [ ] Запускать и управлять контейнерами
- [ ] Находить образы в Docker Hub
- [ ] Понимать архитектуру Docker

---

## 🚀 Что дальше?

В **Блоке 2** мы углубимся в:
- Создание собственных образов с Dockerfile
- Оптимизацию размера образов
- Продвинутое управление контейнерами
- Multi-stage builds
- Лучшие практики разработки

**Готовы продолжить путешествие в мир Docker? 🐳**