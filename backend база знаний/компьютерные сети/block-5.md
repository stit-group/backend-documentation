# 🔒 Блок 5: Безопасность и мониторинг сетей
**⏱️ Время изучения: 3-4 месяца**

---

## 🎯 Цели блока

К концу этого блока вы будете:
- Понимать современные угрозы и методы защиты сетей
- Уметь проектировать многоуровневые системы безопасности
- Владеть инструментами мониторинга и анализа трафика
- Способны реагировать на инциденты безопасности
- Знать принципы Zero Trust архитектуры

---

## 📚 Глава 5.1: Сетевая безопасность

### 5.1.1 Модель угроз и анализ рисков

#### Понимание ландшафта угроз

Современная сетевая безопасность начинается с понимания того, **ЧТО** мы защищаем, **ОТ КОГО** защищаем и **КАК** это делать эффективно.

```
Классификация угроз по источникам:
┌─────────────────────────────────────────────────┐
│                ВНЕШНИЕ УГРОЗЫ                   │
├─────────────────────────────────────────────────┤
│ • Хакеры-одиночки                              │
│ • Организованные группы                         │
│ • Государственные акторы (APT)                 │
│ • Конкуренты                                   │
└─────────────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────┐
│              ПЕРИМЕТР СЕТИ                      │
│    ┌─────────────────────────────────────┐      │
│    │         ВНУТРЕННИЕ УГРОЗЫ           │      │
│    │ • Недовольные сотрудники           │      │
│    │ • Неосторожные пользователи        │      │
│    │ • Инсайдеры с привилегиями         │      │
│    │ • Скомпрометированные аккаунты     │      │
│    └─────────────────────────────────────┘      │
└─────────────────────────────────────────────────┘
```

#### Модель STRIDE для анализа угроз

**STRIDE** - это методология Microsoft для систематического анализа угроз:

- **S**poofing (Подмена) - атакующий притворяется кем-то другим
- **T**ampering (Изменение) - несанкционированная модификация данных
- **R**epudiation (Отказ) - отрицание совершенных действий
- **I**nformation Disclosure (Раскрытие информации) - доступ к конфиденциальным данным
- **D**enial of Service (Отказ в обслуживании) - нарушение доступности
- **E**levation of Privilege (Повышение привилегий) - получение несанкционированных прав

#### Методология оценки рисков

```
Процесс оценки рисков:
────────────────────────────────────────────────

1. ИДЕНТИФИКАЦИЯ АКТИВОВ
   ┌──────────────────┐
   │ • Серверы        │
   │ • Данные         │ ──┐
   │ • Приложения     │   │
   │ • Сетевое ПО     │   │
   └──────────────────┘   │
                          │
2. АНАЛИЗ УЯЗВИМОСТЕЙ     │
   ┌──────────────────┐   │
   │ • CVE базы       │   │
   │ • Пентесты       │ ──┼─► РИСК = УГРОЗА × УЯЗВИМОСТЬ × ВОЗДЕЙСТВИЕ
   │ • Аудиты        │   │
   └──────────────────┘   │
                          │
3. ОЦЕНКА УГРОЗ           │
   ┌──────────────────┐   │
   │ • Threat Intel   │   │
   │ • Исторические   │ ──┘
   │   данные        │
   └──────────────────┘
```

### 5.1.2 VPN технологии

#### IPSec - Internet Protocol Security

IPSec работает на сетевом уровне и обеспечивает **прозрачную** защиту для приложений.

```
Архитектура IPSec:
──────────────────────────────────────────────────

┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   Клиент    │      │ IPSec Gateway│      │   Сервер    │
│             │      │             │      │             │
│ 192.168.1.10│◄────►│  VPN Router │◄────►│ 10.0.0.100  │
└─────────────┘      └─────────────┘      └─────────────┘
      │                      │                    │
      │                      │                    │
   Tunnel Mode          Policy Decision       Protected
      ESP                 Point (PDP)         Network
      AH                      │
                             │
                    ┌─────────────────┐
                    │ IKE (Phase 1)   │ ← Установка SA
                    │ • Аутентификация│
                    │ • Обмен ключами │
                    └─────────────────┘
                             │
                    ┌─────────────────┐
                    │ IKE (Phase 2)   │ ← Создание туннеля
                    │ • IPSec SA      │
                    │ • Параметры ESP │
                    └─────────────────┘
```

**Ключевые протоколы IPSec:**

1. **AH (Authentication Header)** - только аутентификация
2. **ESP (Encapsulating Security Payload)** - шифрование + аутентификация
3. **IKE (Internet Key Exchange)** - управление ключами

**Режимы работы:**
- **Transport Mode** - защищает только payload
- **Tunnel Mode** - защищает весь IP пакет (создает новый IP заголовок)

#### SSL/TLS VPN

Работает на прикладном уровне, не требует специального клиента.

```
SSL VPN vs IPSec:
──────────────────────────────────────────────────

SSL/TLS VPN                    IPSec VPN
─────────────────             ─────────────────
┌─────────────────┐           ┌─────────────────┐
│   Web Browser   │           │ IPSec Client    │
│                 │           │                 │
│ HTTPS Session   │           │ ESP/AH Tunnel   │
├─────────────────┤           ├─────────────────┤
│      TCP        │           │       IP        │
├─────────────────┤           ├─────────────────┤
│      IP         │           │    Ethernet     │
├─────────────────┤           ├─────────────────┤
│   Ethernet      │           │    Physical     │
└─────────────────┘           └─────────────────┘

Преимущества:                Преимущества:
• Простота развертывания      • Прозрачность для приложений
• Работа через NAT           • Высокая производительность
• Гранулярный доступ         • Широкая поддержка протоколов
```

#### WireGuard - современная VPN технология

WireGuard представляет новый подход к VPN с акцентом на **простоту** и **производительность**.

```
Принципы WireGuard:
──────────────────────────────────────────────────

1. КРИПТОГРАФИЧЕСКАЯ ПРОСТОТА
   ┌─────────────────────────────────────┐
   │ Curve25519 (ECDH)                  │ ← Обмен ключами
   │ ChaCha20 (шифрование)              │ ← Шифрование данных
   │ Poly1305 (аутентификация)          │ ← Проверка целостности
   │ BLAKE2s (хеширование)              │ ← Хеш-функция
   │ SipHash24 (хеш-таблицы)            │ ← Внутренние структуры
   └─────────────────────────────────────┘

2. АРХИТЕКТУРА "CRYPTOKEY ROUTING"
   Peer A ◄──────── Зашифрованный туннель ────────► Peer B
   Public Key:       через UDP                   Public Key:
   ABC123...                                     XYZ789...
   
   Каждый peer знает публичные ключи разрешенных собеседников
```

### 5.1.3 Firewalls и фильтрация трафика

#### Эволюция технологий firewall

```
Поколения Firewall:
──────────────────────────────────────────────────

1-е поколение: PACKET FILTER
┌─────────────────────────────────────────┐
│ IP src: 192.168.1.0/24 → ALLOW         │
│ IP dst: 10.0.0.0/8, Port 80 → ALLOW    │
│ IP dst: ANY, Port 22 → DENY            │
└─────────────────────────────────────────┘
Анализ: только заголовки пакетов

2-е поколение: STATEFUL INSPECTION
┌─────────────────────────────────────────┐
│          Connection Table               │
│ ┌─────────────────────────────────────┐ │
│ │ SRC → DST:PORT  STATE    TIME      │ │
│ │ A → B:80        ESTAB    120s      │ │
│ │ C → D:443       SYN_SENT 30s       │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
Анализ: состояние соединений

3-е поколение: APPLICATION LAYER
┌─────────────────────────────────────────┐
│     Deep Packet Inspection (DPI)       │
│ ┌─────────────────────────────────────┐ │
│ │ HTTP: блокировать *.exe файлы      │ │
│ │ SMTP: антивирус сканирование       │ │
│ │ P2P:  ограничение полосы           │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
Анализ: содержимое приложений
```

#### Next-Generation Firewall (NGFW)

NGFW объединяет традиционные функции с современными возможностями:

```
Компоненты NGFW:
──────────────────────────────────────────────────

┌─────────────────────────────────────────────────┐
│                   NGFW STACK                    │
├─────────────────────────────────────────────────┤
│ Application Identification & Control           │ ← Определение приложений
├─────────────────────────────────────────────────┤
│ User Identity Integration                       │ ← Привязка к пользователям
├─────────────────────────────────────────────────┤
│ Intrusion Prevention System (IPS)              │ ← Обнаружение атак
├─────────────────────────────────────────────────┤
│ Advanced Malware Protection                     │ ← Защита от вредоносов
├─────────────────────────────────────────────────┤
│ URL Filtering & Reputation                      │ ← Веб-фильтрация
├─────────────────────────────────────────────────┤
│ Traditional Firewall (Stateful)                │ ← Классические функции
└─────────────────────────────────────────────────┘
```

#### IDS/IPS - системы обнаружения и предотвращения вторжений

```
IDS vs IPS Positioning:
──────────────────────────────────────────────────

NETWORK-BASED IDS (NIDS):
Internet ──► Router ──► Switch ──┬─► Internal Network
                                │
                                ├─► IDS Sensor
                                    (пассивное прослушивание)
                                    │
                                    ▼
                               Security Center
                               (алерты и анализ)

NETWORK-BASED IPS (NIPS):
Internet ──► Router ──► IPS ──► Switch ──► Internal Network
                        │
                        ▼
                   Блокирование
                   атак в реальном
                   времени
```

**Методы обнаружения:**

1. **Signature-based** - сравнение с базой известных паттернов атак
2. **Anomaly-based** - выявление отклонений от нормального поведения
3. **Behavioral-based** - анализ поведения пользователей и систем

### 5.1.4 Zero Trust Architecture

#### Принципы Zero Trust

Zero Trust основывается на принципе **"Never trust, always verify"**.

```
Traditional Perimeter vs Zero Trust:
──────────────────────────────────────────────────

TRADITIONAL PERIMETER SECURITY:
┌─────────────────────────────────────────────┐
│                TRUSTED ZONE                 │
│  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐        │
│  │User │  │App  │  │DB   │  │File │        │
│  │     │  │     │  │     │  │Srv  │        │
│  └─────┘  └─────┘  └─────┘  └─────┘        │
└─────────────────────────────────────────────┘
│◄────────── Implicit Trust ──────────►│
                │
            Firewall
                │
┌─────────────────────────────────────────────┐
│             UNTRUSTED ZONE                  │
│              (Internet)                     │
└─────────────────────────────────────────────┘

ZERO TRUST ARCHITECTURE:
   User ←─┬─→ Policy Engine ←─┬─→ Application
          │                  │
          │    ┌─────────────┴─────────────┐
          │    │     Policy Decision       │
          │    │     • Identity           │
          │    │     • Device State       │
          │    │     • Location           │
          │    │     • Time              │
          │    │     • Risk Score        │
          │    └─────────────┬─────────────┘
          │                  │
          └──► Enforcement ◄─┘
               Point
```

#### Компоненты Zero Trust

```
Zero Trust Control Plane:
──────────────────────────────────────────────────

1. POLICY ENGINE (PE)
   ┌─────────────────────────────────────────┐
   │ • Анализ запросов доступа              │
   │ • Применение политик                   │
   │ • Вычисление risk score                │
   └─────────────────────────────────────────┘
                    │
                    ▼
2. POLICY ADMINISTRATOR (PA)
   ┌─────────────────────────────────────────┐
   │ • Перевод решений в команды            │
   │ • Конфигурация PEP                     │
   │ • Мониторинг соответствия              │
   └─────────────────────────────────────────┘
                    │
                    ▼
3. POLICY ENFORCEMENT POINTS (PEP)
   ┌─────────────────────────────────────────┐
   │ • Gateways                             │
   │ • Proxy серверы                        │
   │ • VPN концентраторы                    │
   │ • Cloud security services             │
   └─────────────────────────────────────────┘
```

---

## 📚 Глава 5.2: Аутентификация и контроль доступа

### 5.2.1 Network Access Control (NAC) и 802.1X

#### Понимание 802.1X

802.1X - это стандарт IEEE для **port-based authentication** в сетях.

```
Архитектура 802.1X:
──────────────────────────────────────────────────

Supplicant          Authenticator         Authentication Server
(Клиент)           (Коммутатор)              (RADIUS)
    │                     │                      │
    │                     │                      │
    │ 1. EAP Start        │                      │
    │◄────────────────────│                      │
    │                     │                      │
    │ 2. EAP Identity     │                      │
    │────────────────────►│                      │
    │                     │ 3. RADIUS Access-Req │
    │                     │─────────────────────►│
    │                     │                      │
    │ 4. EAP Challenge    │ 4. RADIUS Challenge  │
    │◄────────────────────│◄─────────────────────│
    │                     │                      │
    │ 5. EAP Response     │ 5. RADIUS Access-Req │
    │────────────────────►│─────────────────────►│
    │                     │                      │
    │ 6. EAP Success      │ 6. RADIUS Accept     │
    │◄────────────────────│◄─────────────────────│
    │                     │                      │
    │ 7. Port Authorized  │                      │
    │◄═══════════════════►│                      │
```

#### Типы EAP методов

```
EAP Methods Comparison:
──────────────────────────────────────────────────

┌─────────────┬──────────────┬──────────────┬──────────────┐
│   Method    │  Credentials │   Security   │  Use Case    │
├─────────────┼──────────────┼──────────────┼──────────────┤
│ EAP-MD5     │ Password     │ Weak         │ Legacy only  │
├─────────────┼──────────────┼──────────────┼──────────────┤
│ EAP-TLS     │ Certificates │ Very Strong  │ High Security│
├─────────────┼──────────────┼──────────────┼──────────────┤
│ PEAP        │ Password     │ Strong       │ Enterprise   │
├─────────────┼──────────────┼──────────────┼──────────────┤
│ EAP-TTLS    │ Various      │ Strong       │ Flexible     │
├─────────────┼──────────────┼──────────────┼──────────────┤
│ EAP-FAST    │ PAC + Pwd    │ Strong       │ Cisco        │
└─────────────┴──────────────┴──────────────┴──────────────┘
```

#### Dynamic VLAN Assignment

```
NAC Workflow с Dynamic VLAN:
──────────────────────────────────────────────────

1. Подключение устройства
   Device ──► Switch Port (Access VLAN 999 - Quarantine)

2. 802.1X аутентификация
   Device ◄──► Switch ◄──► RADIUS Server
                           │
                           ▼
                      User Database
                      • User: john@company.com
                      • Group: Engineering
                      • VLAN: 100

3. Успешная авторизация
   Device ──► Switch Port (Access VLAN 100 - Engineering)
   
4. Политики доступа применены
   ┌─────────────────────────────────────────┐
   │ VLAN 100 Policies:                     │
   │ • Internet: Allow                      │
   │ • File Servers: Allow                  │
   │ • HR Systems: Deny                     │
   │ • Guest Network: Deny                  │
   └─────────────────────────────────────────┘
```

### 5.2.2 RADIUS и TACACS+

#### RADIUS - Remote Authentication Dial-In User Service

```
RADIUS Packet Flow:
──────────────────────────────────────────────────

NAS (Network Access Server)     RADIUS Server
        │                            │
        │ 1. Access-Request          │
        │───────────────────────────►│
        │   • Username               │
        │   • Password (encrypted)   │
        │   • NAS-IP-Address         │
        │                            │
        │ 2. Access-Challenge        │
        │◄───────────────────────────│
        │   • Challenge String       │
        │                            │
        │ 3. Access-Request          │
        │───────────────────────────►│
        │   • Challenge Response     │
        │                            │
        │ 4. Access-Accept/Reject    │
        │◄───────────────────────────│
        │   • Service-Type           │
        │   • VLAN Assignment        │
        │   • Session-Timeout        │
```

#### TACACS+ vs RADIUS

```
Protocol Comparison:
──────────────────────────────────────────────────

RADIUS                          TACACS+
────────────────────────        ────────────────────────
┌─────────────────────┐          ┌─────────────────────┐
│ AAA в одном пакете  │          │ Раздельные AAA      │
│                     │          │                     │
│ UDP Transport       │          │ TCP Transport       │
│ (порты 1812/1813)   │          │ (порт 49)           │
│                     │          │                     │
│ Шифрование только   │          │ Полное шифрование  │
│ пароля              │          │ всего пакета        │
│                     │          │                     │
│ Стандарт RFC        │          │ Cisco проприетарный │
└─────────────────────┘          └─────────────────────┘

         │                                  │
         ▼                                  ▼
   
Лучше для:                        Лучше для:
• Доступ к сети                   • Управление устройствами
• VPN аутентификация              • Command authorization
• Wi-Fi аутентификация            • Детальный accounting
```

### 5.2.3 Certificate Management и PKI

#### Public Key Infrastructure (PKI)

```
PKI Hierarchy:
──────────────────────────────────────────────────

                 Root CA
               (Offline, Secure)
                     │
                     │ Подписывает
                     ▼
               Intermediate CA
              (Online, Operational)
                     │
            ┌────────┼────────┐
            │        │        │
            ▼        ▼        ▼
      User Certs  Server   Device
                  Certs    Certs

Certificate Fields:
┌─────────────────────────────────────────┐
│ Subject: CN=user@company.com           │
│ Issuer: CN=Company Intermediate CA     │
│ Serial Number: 0x1234567890ABCDEF      │
│ Valid From: 2024-01-01                 │
│ Valid To: 2025-01-01                   │
│ Public Key: RSA 2048-bit               │
│ Signature: SHA-256 with RSA            │
│ Extensions:                            │
│   • Key Usage: Digital Signature      │
│   • Extended Key Usage: Client Auth    │
│   • Subject Alternative Name: DNS     │
└─────────────────────────────────────────┘
```

#### Certificate Lifecycle Management

```
Certificate Lifecycle:
──────────────────────────────────────────────────

1. REQUEST
   User/Device ──► Certificate Authority
                   • CSR (Certificate Signing Request)
                   • Identity verification
                   
2. ISSUANCE
   CA ──► User/Device
         • Signed certificate
         • Root CA certificate chain
         
3. DEPLOYMENT
   Certificate installed on:
   • User devices
   • Servers
   • Network equipment
   
4. VALIDATION
   Regular checks:
   • Certificate expiration
   • Revocation status (CRL/OCSP)
   • Key compromise indicators
   
5. RENEWAL/REVOCATION
   • Automatic renewal before expiry
   • Immediate revocation if compromised
   • CRL/OCSP updates
```

### 5.2.4 Multi-Factor Authentication (MFA)

#### Факторы аутентификации

```
Authentication Factors:
──────────────────────────────────────────────────

SOMETHING YOU KNOW          SOMETHING YOU HAVE
(Knowledge Factor)          (Possession Factor)
┌─────────────────┐         ┌─────────────────┐
│ • Пароли        │         │ • Смарт-карты   │
│ • PIN коды      │         │ • Токены        │
│ • Секретные     │         │ • Мобильные     │
│   вопросы       │         │   приложения    │
└─────────────────┘         └─────────────────┘
        │                           │
        └─────────┬─────────────────┘
                  │
        ┌─────────▼─────────┐
        │ SOMETHING YOU ARE │
        │ (Inherence Factor)│
        │ • Отпечатки      │
        │ • Радужка глаза  │
        │ • Голос          │
        │ • Поведение      │
        └───────────────────┘
```

#### SAML и федеративная аутентификация

```
SAML SSO Flow:
──────────────────────────────────────────────────

User                Service Provider (SP)      Identity Provider (IdP)
 │                         │                          │
 │ 1. Доступ к ресурсу    │                          │
 │────────────────────────►│                          │
 │                         │ 2. Redirect to IdP       │
 │                         │◄─────────────────────────│
 │ 3. Redirect to IdP      │                          │
 │─────────────────────────┼─────────────────────────►│
 │                         │                          │
 │ 4. Аутентификация       │                          │
 │◄────────────────────────┼──────────────────────────│
 │                         │                          │
 │ 5. SAML Response        │                          │
 │◄────────────────────────┼──────────────────────────│
 │                         │                          │
 │ 6. Доступ к ресурсу     │                          │
 │────────────────────────►│                          │
 │                         │                          │
 │ 7. Предоставление       │                          │
 │    ресурса              │                          │
 │◄────────────────────────│                          │
```

---

## 📚 Глава 5.3: Мониторинг и анализ

### 5.3.1 Wireshark и анализ трафика

#### Архитектура анализа пакетов

```
Packet Capture Architecture:
──────────────────────────────────────────────────

Physical Network
      │
      ▼
┌─────────────────┐
│  Network TAP    │ ◄─── Пассивное снятие копии трафика
│  или            │
│  Mirror Port    │ ◄─── Копирование через коммутатор
└─────────────────┘
      │
      ▼
┌─────────────────┐
│ Packet Capture  │
│ Buffer          │ ◄─── Временное хранение пакетов
└─────────────────┘
      │
      ▼
┌─────────────────┐
│ Protocol        │
│ Decoders        │ ◄─── Разбор протоколов
└─────────────────┘
      │
      ▼
┌─────────────────┐
│ Analysis        │
│ Engine          │ ◄─── Анализ и фильтрация
└─────────────────┘
```

#### Ключевые техники анализа

**1. Display Filters (фильтры отображения)**

```
Примеры фильтров Wireshark:
──────────────────────────────────────────────────

Базовые фильтры:
• ip.addr == 192.168.1.100        # Любой трафик с/на IP
• tcp.port == 80                  # HTTP трафик
• http.request.method == "GET"     # HTTP GET запросы

Сложные условия:
• tcp.flags.syn == 1 and tcp.flags.ack == 0    # TCP SYN пакеты
• ip.src == 10.0.0.0/8 and not icmp           # Трафик из 10.x.x.x кроме ping
• dns.qry.name contains "malware"              # DNS запросы с "malware"

Временные фильтры:
• frame.time >= "2024-01-01 00:00:00"         # Пакеты после даты
• tcp.time_delta > 1                          # Медленные TCP ответы
```

**2. Follow Streams (следование потокам)**

Позволяет видеть полный диалог между клиентом и сервером.

```
Stream Analysis:
──────────────────────────────────────────────────

TCP Stream Example:
┌─────────────────────────────────────────────┐
│ Client → Server: HTTP Request               │
│ GET /index.html HTTP/1.1                   │
│ Host: www.example.com                      │
│ User-Agent: Mozilla/5.0...                 │
├─────────────────────────────────────────────┤
│ Server → Client: HTTP Response             │
│ HTTP/1.1 200 OK                           │
│ Content-Type: text/html                    │
│ Content-Length: 1234                       │
│                                            │
│ <html>...</html>                          │
└─────────────────────────────────────────────┘
```

#### Performance Analysis с Wireshark

```
Network Performance Metrics:
──────────────────────────────────────────────────

1. LATENCY ANALYSIS
   ┌─────────────────────────────────────────┐
   │ TCP Handshake Timing:                  │
   │ • SYN → SYN+ACK: Network RTT          │
   │ • SYN+ACK → ACK: Processing delay     │
   │ • Total handshake time                │
   └─────────────────────────────────────────┘

2. THROUGHPUT ANALYSIS
   ┌─────────────────────────────────────────┐
   │ Window Size Tracking:                  │
   │ • TCP window scaling                   │
   │ • Window full conditions               │
   │ • Bandwidth utilization               │
   └─────────────────────────────────────────┘

3. LOSS DETECTION
   ┌─────────────────────────────────────────┐
   │ TCP Issues:                           │
   │ • Duplicate ACKs                      │
   │ • Fast retransmissions               │
   │ • Out of order packets               │
   └─────────────────────────────────────────┘
```

### 5.3.2 NetFlow/sFlow анализ

#### Понимание Flow-based мониторинга

Flow - это последовательность пакетов с одинаковыми характеристиками.

```
Flow Definition:
──────────────────────────────────────────────────

NetFlow v5 Flow Key (5-tuple):
┌─────────────────────────────────────────┐
│ • Source IP Address                     │
│ • Destination IP Address                │
│ • Source Port                          │
│ • Destination Port                      │
│ • Protocol (TCP/UDP/ICMP...)           │
└─────────────────────────────────────────┘
      │
      ▼
Flow Record:
┌─────────────────────────────────────────┐
│ Flow Key + Metadata:                   │
│ • Start/End timestamps                 │
│ • Packet count                         │
│ • Byte count                          │
│ • TCP flags                           │
│ • Input/Output interface              │
│ • Next hop IP                         │
│ • AS numbers                          │
└─────────────────────────────────────────┘
```

#### NetFlow vs sFlow

```
Sampling Comparison:
──────────────────────────────────────────────────

NetFlow (Deterministic):           sFlow (Statistical):
─────────────────────────          ─────────────────────────

Every Flow Tracked                 Packet Sampling
┌─────────────────┐                ┌─────────────────┐
│ Flow 1: 100%    │                │ Packet 1: ✓     │
│ Flow 2: 100%    │                │ Packet 2: ✗     │
│ Flow 3: 100%    │                │ Packet 3: ✗     │
│ Flow 4: 100%    │                │ Packet 4: ✓     │
└─────────────────┘                └─────────────────┘

Преимущества:                      Преимущества:
• Точная статистика               • Низкая нагрузка на CPU
• Детальная информация            • Масштабируемость
• Security анализ                 • Мультивендорность

Недостатки:                       Недостатки:
• Высокая нагрузка на CPU         • Статистическая точность
• Ограниченная масштабируемость   • Менее детальная информация
```

#### Применения Flow анализа

```
Flow Analytics Use Cases:
──────────────────────────────────────────────────

1. CAPACITY PLANNING
   ┌─────────────────────────────────────────┐
   │ Interface Utilization Trends:          │
   │                                        │
   │ GigE 0/1  ████████░░ 80% (peak)       │
   │ GigE 0/2  ██████░░░░ 60% (average)    │
   │ GigE 0/3  ███░░░░░░░ 30% (low)        │
   │                                        │
   │ Prediction: GigE 0/1 needs upgrade    │
   │ in 6 months at current growth rate    │
   └─────────────────────────────────────────┘

2. SECURITY MONITORING
   ┌─────────────────────────────────────────┐
   │ Anomaly Detection:                     │
   │                                        │
   │ • Unusual traffic patterns            │
   │ • DDoS attack signatures              │
   │ • Data exfiltration (large outbound)  │
   │ • Scanning activity (many dst ports)  │
   │ • Botnet communication patterns       │
   └─────────────────────────────────────────┘

3. APPLICATION PERFORMANCE
   ┌─────────────────────────────────────────┐
   │ Top Talkers Analysis:                  │
   │                                        │
   │ Source IP    → Destination  | Bytes    │
   │ 10.1.1.100   → 172.16.1.5 | 2.3 GB   │
   │ 192.168.1.50 → 10.0.0.100 | 1.8 GB   │
   │ 10.1.1.200   → 8.8.8.8    | 1.2 GB   │
   └─────────────────────────────────────────┘
```

### 5.3.3 SIEM и безопасность

#### Security Information and Event Management

```
SIEM Architecture:
──────────────────────────────────────────────────

Data Sources                Collection Layer         Processing Layer
┌─────────────┐             ┌─────────────┐          ┌─────────────┐
│ Firewalls   │────────────►│ Log         │─────────►│ Correlation │
│ IDS/IPS     │             │ Collectors  │          │ Engine      │
│ Servers     │             │             │          │             │
│ Workstations│             │ • Syslog    │          │ • Rules     │
│ Applications│             │ • SNMP      │          │ • Patterns  │
│ Network     │             │ • APIs      │          │ • ML/AI     │
│ Devices     │             │ • Agents    │          │             │
└─────────────┘             └─────────────┘          └─────────────┘
                                   │                        │
                                   ▼                        ▼
                            ┌─────────────┐          ┌─────────────┐
                            │ Normalization│          │ Alerting &  │
                            │ & Parsing   │          │ Dashboards  │
                            └─────────────┘          └─────────────┘
```

#### Event Correlation Rules

```
Correlation Examples:
──────────────────────────────────────────────────

1. MULTIPLE FAILED LOGINS
   Rule: 
   IF (failed_login_count > 5) 
   AND (time_window < 5_minutes) 
   AND (same_source_ip)
   THEN generate_alert("Brute Force Attack")

2. PRIVILEGE ESCALATION
   Rule:
   IF (user_login) 
   AND (privilege_change within 10_minutes)
   AND (admin_access_granted)
   THEN generate_alert("Potential Privilege Escalation")

3. DATA EXFILTRATION
   Rule:
   IF (outbound_traffic > baseline * 3)
   AND (outside_business_hours)
   AND (sensitive_file_access_logged)
   THEN generate_alert("Possible Data Exfiltration")
```

#### Threat Intelligence Integration

```
Threat Intel Workflow:
──────────────────────────────────────────────────

External Feeds              Internal Processing        Response Actions
┌─────────────┐             ┌─────────────┐           ┌─────────────┐
│ Commercial  │────────────►│ IOC         │──────────►│ Automated   │
│ Feeds       │             │ Database    │           │ Blocking    │
│             │             │             │           │             │
│ Open Source │             │ • IPs       │           │ Manual      │
│ Intelligence│             │ • Domains   │           │ Investigation│
│             │             │ • Hashes    │           │             │
│ Government  │             │ • URLs      │           │ Threat      │
│ Feeds       │             │ • Patterns  │           │ Hunting     │
└─────────────┘             └─────────────┘           └─────────────┘
                                   │
                                   ▼
                            ┌─────────────┐
                            │ Enrichment  │
                            │ • Context   │
                            │ • Reputation│
                            │ • History   │
                            └─────────────┘
```

### 5.3.4 Threat Hunting

#### Proactive vs Reactive Security

```
Security Approaches:
──────────────────────────────────────────────────

REACTIVE (Traditional):
Incident occurs ──► Detection ──► Response ──► Recovery
                     │               │
                  Signature      Containment
                   Based         & Cleanup

PROACTIVE (Threat Hunting):
Hypothesis ──► Hunt ──► Discover ──► Investigate ──► Improve
     │          │         │            │             │
  Threat      Search    Anomalies   Root Cause    New Rules
  Intel       Patterns   Found      Analysis      & IOCs
```

#### Hunting Methodologies

```
Hunt Process Framework:
──────────────────────────────────────────────────

1. HYPOTHESIS FORMATION
   ┌─────────────────────────────────────────┐
   │ Based on:                              │
   │ • Threat intelligence                   │
   │ • Attack frameworks (MITRE ATT&CK)     │
   │ • Environmental knowledge              │
   │ • Recent incidents                     │
   └─────────────────────────────────────────┘

2. DATA COLLECTION
   ┌─────────────────────────────────────────┐
   │ Sources:                               │
   │ • Network logs (NetFlow, DNS, Proxy)   │
   │ • Host logs (System, Security, Apps)   │
   │ • Security tools (SIEM, IDS, EDR)     │
   │ • Threat intelligence feeds           │
   └─────────────────────────────────────────┘

3. ANALYSIS TECHNIQUES
   ┌─────────────────────────────────────────┐
   │ Methods:                               │
   │ • Statistical analysis                 │
   │ • Pattern matching                     │
   │ • Behavioral analysis                  │
   │ • Timeline reconstruction              │
   │ • Pivot and drill-down                 │
   └─────────────────────────────────────────┘
```

---

## 🎯 Практические задания

### Задание 1: Анализ инцидента безопасности

**Сценарий:** В вашей сети обнаружена подозрительная активность - множественные неудачные попытки входа, за которыми следует успешная аутентификация и аномальный сетевой трафик.

**Ваши действия:**
1. Определите источники логов для анализа
2. Создайте timeline событий
3. Проведите анализ трафика
4. Определите scope инцидента
5. Разработайте план containment

### Задание 2: Настройка 802.1X

**Цель:** Настроить port-based authentication для корпоративной сети.

**Компоненты:**
- Коммутаторы с поддержкой 802.1X
- RADIUS сервер (FreeRADIUS)
- Certificate Authority
- Клиентские устройства

**Требования:**
- Dynamic VLAN assignment
- Quarantine VLAN для неавторизованных устройств
- Guest network для посетителей

### Задание 3: Анализ NetFlow данных

**Задача:** Проанализировать NetFlow данные за неделю и выявить:
- Top talkers (источники трафика)
- Аномальные паттерны
- Potential security threats
- Capacity planning recommendations

---

## 📋 Контрольные вопросы

### Базовый уровень:
1. Объясните разницу между IDS и IPS
2. Какие компоненты входят в PKI инфраструктуру?
3. Что такое Zero Trust и его основные принципы?
4. Чем отличается NetFlow от sFlow?

### Продвинутый уровень:
1. Как проектировать defense-in-depth архитектуру?
2. Какие метрики использовать для threat hunting?
3. Как интегрировать threat intelligence в SIEM?
4. Методы обнаружения advanced persistent threats (APT)?

### Экспертный уровень:
1. Разработайте стратегию перехода к Zero Trust
2. Создайте методологию threat hunting для вашей организации
3. Спроектируйте SIEM архитектуру для enterprise сети
4. Оцените эффективность текущих security controls

---

## 🚀 Следующие шаги

После успешного завершения этого блока вы готовы к:

**Блок 6: Современные технологии и облака**
- Software-Defined Networking (SDN)
- Облачные сетевые сервисы
- Автоматизация и программирование сетей
- Container и microservices networking

**Профессиональные цели:**
- Security Architect позиции
- SOC Analyst/Manager роли
- Network Security Engineer позиции
- Консалтинг по информационной безопасности

**Рекомендуемые сертификации:**
- CISSP (Certified Information Systems Security Professional)
- CCNP Security
- GCIH (GIAC Certified Incident Handler)
- CEH (Certified Ethical Hacker)