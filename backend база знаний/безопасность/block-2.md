# 🛡️ БЛОК 2: ТЕХНИЧЕСКАЯ ЗАЩИТА СИСТЕМ

---
```
 ╔═══════════════════════════════════════════════════════════════╗
 ║                    ТЕХНИЧЕСКАЯ ЗАЩИТА СИСТЕМ                 ║
 ║                                                               ║
 ║  Продолжительность: 3-4 месяца                              ║
 ║  Уровень: Intermediate                                        ║
 ║  Цель: Освоить технические средства и методы защиты ИС      ║
 ╚═══════════════════════════════════════════════════════════════╝
```
---

## 🎯 ЦЕЛИ И РЕЗУЛЬТАТЫ БЛОКА

**По завершению блока вы сможете:**
- ✅ Настраивать и администрировать криптографические системы
- ✅ Проектировать архитектуру сетевой безопасности
- ✅ Выполнять hardening операционных систем
- ✅ Защищать приложения и базы данных от атак
- ✅ Внедрять DevSecOps практики

---

## 📊 КАРТА НАВЫКОВ

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   КРИПТОГРАФИЯ  │────│ СЕТЕВАЯ ЗАЩИТА  │────│  ЗАЩИТА ОС      │
│                 │    │                 │    │                 │
│ • AES/RSA       │    │ • Firewalls     │    │ • Hardening     │
│ • TLS/SSL       │    │ • IDS/IPS       │    │ • Аудит         │
│ • PKI           │    │ • VPN           │    │ • Мониторинг    │
│ • ГОСТ          │    │ • VLAN          │    │ • Backup        │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │ ЗАЩИТА ПРИЛОЖЕНИЙ│
                    │                 │
                    │ • OWASP Top 10  │
                    │ • WAF           │
                    │ • API Security  │
                    │ • DevSecOps     │
                    └─────────────────┘
```

---

# 🔐 ГЛАВА 2.1: ОСНОВЫ КРИПТОГРАФИИ

---

## 📋 ПЛАН ГЛАВЫ (4 недели)

| Неделя | Тема | Практика |
|--------|------|----------|
| 1 | Симметричное шифрование | Реализация AES |
| 2 | Асимметричное шифрование | Настройка PKI |
| 3 | Хеширование и ЭЦП | Анализ TLS |
| 4 | Российские стандарты | ГОСТ криптография |

---

## 🧠 ТЕОРЕТИЧЕСКАЯ ЧАСТЬ

### 📖 2.1.1 Основы криптографии

**Криптография** — наука о методах обеспечения конфиденциальности, целостности и аутентичности данных.

```
┌─────────────────────────────────────────────────────────┐
│                    ТИПЫ КРИПТОГРАФИИ                    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  СИММЕТРИЧНАЯ           │         АСИММЕТРИЧНАЯ         │
│  ┌─────────────┐       │       ┌─────────────┐         │
│  │ Один ключ   │       │       │ Пара ключей │         │
│  │ ┌─────────┐ │       │       │ ┌─────────┐ │         │
│  │ │   AES   │ │       │       │ │   RSA   │ │         │
│  │ │   DES   │ │       │       │ │   ECC   │ │         │
│  │ │ ChaCha20│ │       │       │ │ DSA/ECDSA│ │         │
│  │ └─────────┘ │       │       │ └─────────┘ │         │
│  └─────────────┘       │       └─────────────┘         │
│                                                         │
│  Быстро                │         Медленно              │
│  Проблема распределения│         Безопасное             │
│  ключей                │         распределение          │
└─────────────────────────────────────────────────────────┘
```

### 🔑 2.1.2 Симметричное шифрование

**Advanced Encryption Standard (AES)**

```
Процесс шифрования AES:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ Открытый    │    │ Ключ        │    │ Зашифрован- │
│ текст       │───▶│ шифрования  │───▶│ ный текст   │
│ (128 бит)   │    │ (128/192/   │    │ (128 бит)   │
│             │    │  256 бит)   │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
```

**Режимы работы блочных шифров:**

| Режим | Описание | Использование |
|-------|----------|---------------|
| **ECB** | Electronic Codebook | ❌ Не рекомендуется |
| **CBC** | Cipher Block Chaining | ✅ Шифрование файлов |
| **CTR** | Counter Mode | ✅ Высокая производительность |
| **GCM** | Galois/Counter Mode | ✅ AEAD (TLS 1.3) |

---

### 🔐 2.1.3 Асимметричное шифрование

**RSA (Rivest-Shamir-Adleman)**

```
Генерация ключей RSA:
1. p, q - простые числа
2. n = p × q
3. φ(n) = (p-1)(q-1)
4. e: gcd(e, φ(n)) = 1
5. d: e × d ≡ 1 (mod φ(n))

Открытый ключ: (n, e)
Закрытый ключ: (n, d)
```

**Эллиптические кривые (ECC)**

```
┌─────────────────────────────────────┐
│        СРАВНЕНИЕ RSA vs ECC         │
├─────────────────┬───────────────────┤
│ Безопасность    │ Размер ключа      │
├─────────────────┼───────────────────┤
│ 80 бит          │ RSA: 1024 / ECC: 160 │
│ 112 бит         │ RSA: 2048 / ECC: 224 │
│ 128 бит         │ RSA: 3072 / ECC: 256 │
│ 192 бит         │ RSA: 7680 / ECC: 384 │
│ 256 бит         │ RSA: 15360/ ECC: 521 │
└─────────────────┴───────────────────┘
```

---

### 🏗️ 2.1.4 Инфраструктура открытых ключей (PKI)

```
                PKI АРХИТЕКТУРА
    ┌─────────────────────────────────────────┐
    │              ROOT CA                    │
    │         (Корневой центр)               │
    └──────────────┬──────────────────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
    ┌───▼────┐          ┌─────▼───┐
    │SUB CA 1│          │SUB CA 2 │
    │(Промеж.)│          │(Промеж.)│
    └───┬────┘          └─────┬───┘
        │                     │
   ┌────▼────┐            ┌───▼────┐
   │End Users│            │Servers │
   │  Certs  │            │ Certs  │
   └─────────┘            └────────┘
```

**Компоненты PKI:**
- 📋 **CA (Certificate Authority)** - центр сертификации
- 📋 **RA (Registration Authority)** - центр регистрации  
- 📋 **CRL (Certificate Revocation List)** - список отозванных сертификатов
- 📋 **OCSP (Online Certificate Status Protocol)** - онлайн проверка статуса

---

### 🇷🇺 2.1.5 Российские криптографические стандарты

**ГОСТ Р 34.10-2012 (Электронная подпись)**
```
Алгоритм на эллиптических кривых:
- Кривые: 256 и 512 бит
- Стандартные параметры: 
  * tc26-gost-3410-12-256-paramSetA
  * tc26-gost-3410-12-512-paramSetA
```

**ГОСТ Р 34.11-2012 (Стрибог)**
```
Хеш-функция:
- Выходные значения: 256 или 512 бит
- Используется совместно с ГОСТ Р 34.10-2012
```

**ГОСТ Р 34.12-2015 (Кузнечик)**
```
Блочный шифр:
- Размер блока: 128 бит
- Размер ключа: 256 бит
- Замена DES и ГОСТ 28147-89
```

---

## 🛠️ ПРАКТИЧЕСКИЕ ЗАДАНИЯ

### 💻 Задание 2.1.1: Реализация AES

**Цель:** Понять принципы работы симметричного шифрования

```python
# Пример реализации AES на Python
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import os

# Генерация ключа
key = os.urandom(32)  # 256-bit key
iv = os.urandom(16)   # 128-bit IV

# Создание шифра
cipher = Cipher(algorithms.AES(key), modes.CBC(iv))
encryptor = cipher.encryptor()

# Шифрование
plaintext = b"Секретное сообщение"
# Дополнение до кратного 16 байт (PKCS7)
padded = plaintext + b'\x00' * (16 - len(plaintext) % 16)
ciphertext = encryptor.update(padded) + encryptor.finalize()

print(f"Зашифровано: {ciphertext.hex()}")
```

**Задачи:**
1. ✅ Реализовать шифрование/дешифрование AES-256-CBC
2. ✅ Сравнить производительность различных режимов
3. ✅ Реализовать безопасное хранение ключей

---

### 🏗️ Задание 2.1.2: Настройка PKI

**Цель:** Создать собственную инфраструктуру открытых ключей

```bash
# Создание корневого CA
openssl genrsa -out ca-key.pem 4096
openssl req -new -x509 -days 3650 -key ca-key.pem -out ca.pem

# Создание промежуточного CA
openssl genrsa -out intermediate-key.pem 4096
openssl req -new -key intermediate-key.pem -out intermediate.csr
openssl x509 -req -in intermediate.csr -CA ca.pem -CAkey ca-key.pem \
  -CAcreateserial -out intermediate.pem -days 1825

# Создание пользовательского сертификата
openssl genrsa -out user-key.pem 2048
openssl req -new -key user-key.pem -out user.csr
openssl x509 -req -in user.csr -CA intermediate.pem \
  -CAkey intermediate-key.pem -CAcreateserial -out user.pem -days 365
```

---

### 🔍 Задание 2.1.3: Анализ TLS-трафика

**Цель:** Изучить работу протоколов TLS/SSL

```
TLS 1.3 Handshake:
┌─────────┐                    ┌─────────┐
│ Client  │                    │ Server  │
└────┬────┘                    └────┬────┘
     │                              │
     │ 1. ClientHello               │
     │ ────────────────────────────▶│
     │                              │
     │ 2. ServerHello + Certificate │
     │ ◀────────────────────────────│
     │                              │
     │ 3. Finished                  │
     │ ────────────────────────────▶│
     │                              │
     │ 4. Finished                  │
     │ ◀────────────────────────────│
     │                              │
     │ Application Data             │
     │ ◀───────────────────────────▶│
```

**Практика с Wireshark:**
1. ✅ Захват HTTPS трафика
2. ✅ Анализ Certificate Chain
3. ✅ Проверка использованных cipher suites
4. ✅ Выявление уязвимостей (слабые алгоритмы)

---

## 📊 КОНТРОЛЬНЫЕ ВОПРОСЫ

### 🧪 Тест на понимание

1. **Какой размер ключа AES эквивалентен RSA-2048 по стойкости?**
   - [ ] AES-128
   - [ ] AES-192  
   - [x] AES-256
   - [ ] AES-512

2. **Что НЕ обеспечивает цифровая подпись?**
   - [ ] Аутентичность
   - [ ] Целостность
   - [x] Конфиденциальность
   - [ ] Неотрекаемость

3. **Какой режим AES обеспечивает аутентифицированное шифрование?**
   - [ ] CBC
   - [ ] ECB
   - [ ] CTR
   - [x] GCM

---

## 📚 ДОПОЛНИТЕЛЬНЫЕ РЕСУРСЫ

### 📖 Рекомендуемая литература
- 📘 "Applied Cryptography" - Bruce Schneier
- 📘 "Cryptography Engineering" - Ferguson, Schneier, Kohno
- 📘 "Introduction to Modern Cryptography" - Katz, Lindell

### 🔗 Полезные ссылки
- 🌐 [NIST SP 800-57](https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final)
- 🌐 [RFC 8446 - TLS 1.3](https://tools.ietf.org/html/rfc8446)
- 🌐 [CryptoPals Challenges](https://cryptopals.com/)

---

# 🌐 ГЛАВА 2.2: СЕТЕВАЯ БЕЗОПАСНОСТЬ

---

## 📋 ПЛАН ГЛАВЫ (4 недели)

| Неделя | Тема | Практика |
|--------|------|----------|
| 1 | Модель OSI и TCP/IP | Анализ протоколов |
| 2 | Межсетевые экраны | Настройка iptables/pfSense |
| 3 | IDS/IPS системы | Развертывание Suricata |
| 4 | VPN и VLAN | Конфигурирование OpenVPN |

---

## 🧠 ТЕОРЕТИЧЕСКАЯ ЧАСТЬ

### 📖 2.2.1 Модель OSI и стек TCP/IP

```
МОДЕЛЬ OSI vs TCP/IP
┌─────────────────┐    ┌─────────────────┐
│ 7. Application  │    │                 │
├─────────────────┤    │   Application   │
│ 6. Presentation │    │                 │
├─────────────────┤    │                 │
│ 5. Session      │    │                 │
├─────────────────┤    ├─────────────────┤
│ 4. Transport    │    │   Transport     │
├─────────────────┤    ├─────────────────┤
│ 3. Network      │    │   Internet      │
├─────────────────┤    ├─────────────────┤
│ 2. Data Link    │    │                 │
├─────────────────┤    │   Link          │
│ 1. Physical     │    │                 │
└─────────────────┘    └─────────────────┘
```

**Угрозы на каждом уровне:**

| Уровень | Угрозы | Защита |
|---------|--------|--------|
| **Application** | SQL injection, XSS | WAF, Input validation |
| **Transport** | TCP hijacking, DoS | IPS, Rate limiting |
| **Network** | IP spoofing, Routing attacks | ACL, Routing security |
| **Data Link** | ARP spoofing, MAC flooding | Port security, 802.1X |
| **Physical** | Wiretapping, Jamming | Physical security, Encryption |

---

### 🛡️ 2.2.2 Межсетевые экраны (Firewalls)

```
ТИПЫ МЕЖСЕТЕВЫХ ЭКРАНОВ

┌─────────────────────────────────────────────────────────┐
│                PACKET FILTER                            │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │
│  │ IP src  │  │ IP dst  │  │  Port   │  │Protocol │   │
│  │ IP dst  │  │ Port    │  │ Action  │  │ Action  │   │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │
│  Уровень 3-4 OSI                                       │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│               STATEFUL FIREWALL                         │
│  ┌─────────────────┐  ┌─────────────────┐              │
│  │ Connection      │  │ State Table     │              │
│  │ Tracking        │  │ TCP States      │              │
│  │                 │  │ UDP Sessions    │              │
│  └─────────────────┘  └─────────────────┘              │
│  Уровень 3-4 OSI + Connection State                    │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│            APPLICATION FIREWALL                         │
│  ┌─────────────────┐  ┌─────────────────┐              │
│  │ Deep Packet     │  │ Protocol        │              │
│  │ Inspection      │  │ Analysis        │              │
│  │ (DPI)           │  │ HTTP/FTP/etc    │              │
│  └─────────────────┘  └─────────────────┘              │
│  Уровень 3-7 OSI                                       │
└─────────────────────────────────────────────────────────┘
```

**Правила iptables (Linux):**

```bash
# Базовая конфигурация
iptables -P INPUT DROP
iptables -P FORWARD DROP  
iptables -P OUTPUT ACCEPT

# Разрешить loopback
iptables -A INPUT -i lo -j ACCEPT

# Разрешить установленные соединения
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Разрешить SSH (осторожно!)
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Разрешить HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Логирование отброшенных пакетов
iptables -A INPUT -j LOG --log-prefix "DROPPED: "
```

---

### 🔍 2.2.3 Системы обнаружения вторжений (IDS/IPS)

```
IDS vs IPS АРХИТЕКТУРА

┌─────────────────────────────────────┐
│                IDS                  │
│  ┌─────────┐    ┌─────────────────┐ │
│  │ Network │───▶│ Detection       │ │
│  │ Traffic │    │ Engine          │ │
│  │ Mirror  │    │                 │ │
│  └─────────┘    │ ┌─────────────┐ │ │
│                 │ │ Signatures  │ │ │
│                 │ │ Anomaly     │ │ │
│                 │ │ Behavioral  │ │ │
│                 │ └─────────────┘ │ │
│                 └─────────────────┘ │
│                          │          │
│                    ┌─────▼─────┐    │
│                    │ Alert/Log │    │
│                    └───────────┘    │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│                IPS                  │
│  ┌─────────┐    ┌─────────────────┐ │
│  │ Network │───▶│ Detection +     │ │
│  │ Traffic │    │ Prevention      │ │
│  │ Inline  │    │ Engine          │ │
│  └─────────┘    │                 │ │
│       │         │ ┌─────────────┐ │ │
│       │         │ │ Block/Drop  │ │ │
│       │         │ │ Reset       │ │ │
│       │         │ │ Redirect    │ │ │
│       │         │ └─────────────┘ │ │
│       │         └─────────────────┘ │
│       ▼                             │
│  ┌─────────┐                       │
│  │Protected│                       │
│  │ Network │                       │
│  └─────────┘                       │
└─────────────────────────────────────┘
```

**Типы обнаружения:**

1. **Signature-based (Сигнатурный)**
   ```
   Правило Suricata:
   alert tcp any any -> any 80 (
     msg:"Possible SQL injection"; 
     content:"union select";
     nocase; sid:1001;
   )
   ```

2. **Anomaly-based (Аномальный)**
   ```
   Статистический анализ:
   - Объем трафика
   - Паттерны соединений  
   - Временные аномалии
   - Поведенческие отклонения
   ```

3. **Behavioral-based (Поведенческий)**
   ```
   Machine Learning подходы:
   - Кластеризация трафика
   - Нейронные сети
   - Байесовские классификаторы
   ```

---

### 🔐 2.2.4 VPN технологии

```
ТИПЫ VPN СОЕДИНЕНИЙ

┌─────────────────────────────────────────────────────────┐
│                 SITE-TO-SITE VPN                        │
│                                                         │
│ ┌─────────────┐    ┌───────────┐    ┌─────────────┐    │
│ │ Office A    │    │ Internet  │    │ Office B    │    │
│ │ 192.168.1.0 │────│    VPN    │────│ 192.168.2.0 │    │
│ │     /24     │    │ Tunnel    │    │     /24     │    │
│ └─────────────┘    └───────────┘    └─────────────┘    │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│               REMOTE ACCESS VPN                         │
│                                                         │
│ ┌─────────────┐    ┌───────────┐    ┌─────────────┐    │
│ │ Remote User │    │ Internet  │    │ Corporate   │    │
│ │ (Laptop)    │────│    VPN    │────│ Network     │    │
│ │             │    │ Tunnel    │    │             │    │
│ └─────────────┘    └───────────┘    └─────────────┘    │
└─────────────────────────────────────────────────────────┘
```

**Протоколы VPN:**

| Протокол | Уровень | Безопасность | Производительность |
|----------|---------|--------------|-------------------|
| **IPSec** | L3 (IP) | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **OpenVPN** | L2/L3 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **WireGuard** | L3 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **PPTP** | L2 | ⭐ | ⭐⭐⭐⭐⭐ |
| **L2TP/IPSec** | L2 | ⭐⭐⭐⭐ | ⭐⭐⭐ |

---

### 🏢 2.2.5 Сегментация сетей и VLAN

```
СЕТЕВАЯ СЕГМЕНТАЦИЯ

┌─────────────────────────────────────────────────────────┐
│                 FLAT NETWORK (ПЛОХО)                    │
│                                                         │
│ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐       │
│ │ PC1 │─│ PC2 │─│ SRV │─│ DB  │─│ WEB │─│ DMZ │       │
│ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘       │
│    Все в одном broadcast домене                        │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│              SEGMENTED NETWORK (ХОРОШО)                 │
│                                                         │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐        │
│ │ USER VLAN   │ │ SERVER VLAN │ │  DMZ VLAN   │        │
│ │ 10.1.0.0/24 │ │ 10.2.0.0/24 │ │ 10.3.0.0/24 │        │
│ │             │ │             │ │             │        │
│ │ ┌─────────┐ │ │ ┌─────────┐ │ │ ┌─────────┐ │        │
│ │ │   PC1   │ │ │ │   DB    │ │ │ │   WEB   │ │        │
│ │ │   PC2   │ │ │ │   FILE  │ │ │ │   MAIL  │ │        │
│ │ └─────────┘ │ │ └─────────┘ │ │ └─────────┘ │        │
│ └─────────────┘ └─────────────┘ └─────────────┘        │
│        │               │               │                │
│        └───────────────┼───────────────┘                │
│                        │                                │
│            ┌───────────▼────────────┐                   │
│            │     ROUTER/FIREWALL    │                   │
│            │    Access Control      │                   │
│            └────────────────────────┘                   │
└─────────────────────────────────────────────────────────┘
```

**Конфигурация VLAN (Cisco):**

```
# Создание VLAN
Switch(config)# vlan 10
Switch(config-vlan)# name USERS
Switch(config-vlan)# exit

Switch(config)# vlan 20  
Switch(config-vlan)# name SERVERS
Switch(config-vlan)# exit

# Настройка портов
Switch(config)# interface range fa0/1-10
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10

Switch(config)# interface range fa0/11-20
Switch(config-if-range)# switchport mode access  
Switch(config-if-range)# switchport access vlan 20

# Настройка trunk порта
Switch(config)# interface gi0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20
```

---

## 🛠️ ПРАКТИЧЕСКИЕ ЗАДАНИЯ

### 💻 Задание 2.2.1: Настройка межсетевого экрана

**Цель:** Создать правила фильтрации трафика

```bash
#!/bin/bash
# Скрипт настройки iptables для веб-сервера

# Очистка существующих правил
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X

# Установка политик по умолчанию
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Разрешить loopback
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Разрешить установленные соединения
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# SSH доступ (ограничить по IP)
iptables -A INPUT -p tcp -s 192.168.1.100 --dport 22 -j ACCEPT

# HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# DNS
iptables -A INPUT -p udp --dport 53 -j ACCEPT
iptables -A INPUT -p tcp --dport 53 -j ACCEPT

# Защита от DoS атак
iptables -A INPUT -p tcp --dport 80 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT

# Блокировка ping flood
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT

# Логирование
iptables -A INPUT -j LOG --log-prefix "IPTABLES-DROPPED: " --log-level 4

# Сохранение правил
iptables-save > /etc/iptables/rules.v4
```

**Задачи:**
1. ✅ Настроить базовые правила фильтрации
2. ✅ Реализовать защиту от DDoS
3. ✅ Настроить логирование безопасности
4. ✅ Создать скрипты автоматизации

---

### 🔍 Задание 2.2.2: Развертывание IDS/IPS

**Цель:** Настроить систему обнаружения вторжений

```yaml
# docker-compose.yml для Suricata + ELK
version: '3.7'
services:
  suricata:
    image: jasonish/suricata:latest
    container_name: suricata
    cap_add:
      - NET_ADMIN
      - SYS_NICE
    network_mode: host
    volumes:
      - ./suricata:/etc/suricata
      - ./logs:/var/log/suricata
    command: >
      suricata -c /etc/suricata/suricata.yaml -i eth0
      
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
      
  logstash:
    image: docker.elastic.co/logstash/logstash:7.15.0
    container_name: logstash
    volumes:
      - ./logstash:/usr/share/logstash/pipeline
      - ./logs:/var/log/suricata
    ports:
      - "5044:5044"
      
  kibana:
    image: docker.elastic.co/kibana/kibana:7.15.0
    container_name: kibana
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - "5601:5601"
```

**Конфигурация Suricata правил:**

```yaml
# suricata.yaml (основные настройки)
%YAML 1.1
---
vars:
  address-groups:
    HOME_NET: "[192.168.0.0/16,10.0.0.0/8,172.16.0.0/12]"
    EXTERNAL_NET: "!$HOME_NET"
    
  port-groups:
    HTTP_PORTS: "80"
    SHELLCODE_PORTS: "!80"
    
default-rule-path: /etc/suricata/rules
rule-files:
  - emerging-threats.rules
  - custom.rules
  
outputs:
  - eve-log:
      enabled: yes
      filetype: regular
      filename: eve.json
      types:
        - alert
        - http
        - dns
        - tls
```

**Пользовательские правила:**

```bash
# custom.rules
alert tcp $EXTERNAL_NET any -> $HOME_NET $HTTP_PORTS (
  msg:"Possible SQL Injection Attack"; 
  content:"union"; nocase; 
  content:"select"; nocase; distance:0; within:10;
  sid:1000001; rev:1;
)

alert tcp $EXTERNAL_NET any -> $HOME_NET $HTTP_PORTS (
  msg:"Possible XSS Attack"; 
  content:"<script"; nocase;
  sid:1000002; rev:1;
)

alert tcp $EXTERNAL_NET any -> $HOME_NET 22 (
  msg:"SSH Brute Force Attack"; 
  flow:to_server; flags:S; 
  threshold:type both, track by_src, count 5, seconds 60;
  sid:1000003; rev:1;
)
```

---

### 🔐 Задание 2.2.3: Настройка OpenVPN

**Цель:** Создать безопасное VPN соединение

```bash
# Генерация сертификатов (Easy-RSA)
cd /etc/openvpn/easy-rsa
./easyrsa init-pki
./easyrsa build-ca nopass
./easyrsa gen-req server nopass
./easyrsa sign-req server server
./easyrsa gen-dh
openvpn --genkey --secret ta.key

# Генерация клиентского сертификата
./easyrsa gen-req client1 nopass
./easyrsa sign-req client client1
```

```bash
# server.conf
port 1194
proto udp
dev tun

ca ca.crt
cert server.crt
key server.key
dh dh.pem
tls-auth ta.key 0

server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt

# Маршруты для клиентов
push "route 192.168.1.0 255.255.255.0"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"

keepalive 10 120
tls-version-min 1.2
cipher AES-256-GCM
auth SHA256
comp-lzo

user nobody
group nogroup
persist-key
persist-tun

status openvpn-status.log
log openvpn.log
verb 3
```

```bash
# client.ovpn
client
dev tun
proto udp
remote your-server-ip 1194
resolv-retry infinite
nobind
persist-key
persist-tun

ca ca.crt
cert client1.crt
key client1.key
tls-auth ta.key 1

cipher AES-256-GCM
auth SHA256
comp-lzo
verb 3
```

---

## 📊 КОНТРОЛЬНЫЕ ВОПРОСЫ

### 🧪 Тест на понимание

1. **Какой тип firewall работает на уровне приложений?**
   - [ ] Packet filter
   - [ ] Stateful firewall
   - [x] Application firewall
   - [ ] Circuit-level gateway

2. **Что означает "fail-open" в контексте IPS?**
   - [x] Трафик проходит при отказе системы
   - [ ] Трафик блокируется при отказе системы
   - [ ] Система перезагружается
   - [ ] Генерируется alert

3. **Какой протокол VPN считается наиболее производительным?**
   - [ ] OpenVPN
   - [ ] IPSec
   - [x] WireGuard
   - [ ] PPTP

---

## 📚 ПОЛЕЗНЫЕ КОМАНДЫ

### 🐧 Linux Network Security

```bash
# Мониторинг сетевых соединений
netstat -tulpn
ss -tulpn
lsof -i

# Анализ трафика
tcpdump -i eth0 -w capture.pcap
tcpdump -r capture.pcap 'port 80'

# Тестирование портов
nmap -sS -O target
nmap -sV -p- target

# Проверка правил iptables
iptables -L -n -v --line-numbers
iptables -t nat -L -n -v

# Мониторинг логов
tail -f /var/log/syslog | grep iptables
journalctl -u iptables -f
```

### 🔧 Отладка сети

```bash
# Трассировка маршрута
traceroute google.com
mtr google.com

# Проверка DNS
dig google.com
nslookup google.com

# Тестирование скорости
iperf3 -s  # сервер
iperf3 -c server_ip  # клиент

# Проверка сертификатов
openssl s_client -connect google.com:443 -showcerts
```

---

**🎯 Следующая глава:** [2.3 Защита операционных систем](#-глава-23-защита-операционных-систем)

---