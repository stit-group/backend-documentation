# Блок 5: Безопасность и production-ready решения

**⏱️ Продолжительность:** 4-6 недель  
**🎯 Цель:** Обеспечить безопасность микросервисной архитектуры и подготовить систему к продакшену

---

## 📋 Содержание блока

1. [Аутентификация и авторизация](#1-аутентификация-и-авторизация)
2. [Service-to-service security](#2-service-to-service-security)
3. [Network security](#3-network-security)
4. [Secrets management](#4-secrets-management)
5. [Performance и scalability](#5-performance-и-scalability)
6. [Production readiness](#6-production-readiness)
7. [Distributed tracing](#7-distributed-tracing)

---

## 1. Аутентификация и авторизация

### 🔐 Основные принципы безопасности

```
┌─────────────────────────────────────────────────────────────┐
│                    ПРИНЦИПЫ БЕЗОПАСНОСТИ                    │
├─────────────────────────────────────────────────────────────┤
│ 1. Аутентификация (Authentication) - КТО пользователь?      │
│ 2. Авторизация (Authorization) - ЧТО может делать?          │
│ 3. Аудит (Audit) - ЧТО было сделано?                       │
│ 4. Accounting (Учет) - КОГДА и СКОЛЬКО?                     │
└─────────────────────────────────────────────────────────────┘
```

------

### 🎫 JWT Tokens и их безопасное использование

#### Структура JWT токена:

```
Header.Payload.Signature
```

```json
// Header
{
  "alg": "HS256",
  "typ": "JWT"
}

// Payload
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022,
  "exp": 1516242622,
  "roles": ["user", "admin"]
}
```

#### ✅ Best Practices для JWT:

```
┌─────────────────────────────────────────────────────────────┐
│                     JWT BEST PRACTICES                      │
├─────────────────────────────────────────────────────────────┤
│ ✅ Используйте короткий срок жизни (15-30 минут)            │
│ ✅ Храните в httpOnly cookies для веб-приложений            │
│ ✅ Используйте refresh tokens для обновления                │
│ ✅ Включайте минимум необходимой информации                 │
│ ✅ Используйте сильные алгоритмы подписи (RS256, ES256)     │
│ ❌ НЕ храните в localStorage                                 │
│ ❌ НЕ включайте чувствительные данные                       │
└─────────────────────────────────────────────────────────────┘
```

------

### 🔄 OAuth 2.0 Flows

#### Authorization Code Flow (рекомендуемый):

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Client    │    │   Browser   │    │Auth Server  │    │  Resource   │
│ Application │    │             │    │             │    │   Server    │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
       │                  │                  │                  │
       │ 1. Redirect to   │                  │                  │
       │    auth server   │                  │                  │
       │─────────────────▶│                  │                  │
       │                  │ 2. User login    │                  │
       │                  │─────────────────▶│                  │
       │                  │ 3. Authorization │                  │
       │                  │    code          │                  │
       │                  │◀─────────────────│                  │
       │ 4. Authorization │                  │                  │
       │    code          │                  │                  │
       │◀─────────────────│                  │                  │
       │ 5. Exchange code │                  │                  │
       │    for tokens    │                  │                  │
       │─────────────────────────────────────▶│                  │
       │ 6. Access Token  │                  │                  │
       │    + Refresh     │                  │                  │
       │◀─────────────────────────────────────│                  │
       │ 7. API call with │                  │                  │
       │    access token  │                  │                  │
       │─────────────────────────────────────────────────────────▶│
       │ 8. Protected     │                  │                  │
       │    resource      │                  │                  │
       │◀─────────────────────────────────────────────────────────│
```

#### Практический пример настройки:

```yaml
# OAuth 2.0 Configuration
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ${GOOGLE_CLIENT_ID}
            client-secret: ${GOOGLE_CLIENT_SECRET}
            scope:
              - openid
              - profile
              - email
            redirect-uri: "{baseUrl}/oauth2/callback/{registrationId}"
        provider:
          google:
            authorization-uri: https://accounts.google.com/o/oauth2/v2/auth
            token-uri: https://www.googleapis.com/oauth2/v4/token
            user-info-uri: https://www.googleapis.com/oauth2/v3/userinfo
```

------

### 🔗 OpenID Connect

#### OIDC поток:

```
┌─────────────────────────────────────────────────────────────┐
│                    OIDC = OAuth 2.0 + Identity              │
├─────────────────────────────────────────────────────────────┤
│ OAuth 2.0: Authorization (что можно делать)                 │
│ OpenID Connect: Authentication (кто пользователь)           │
│                                                             │
│ ID Token (JWT) содержит:                                    │
│ - sub (subject) - уникальный ID пользователя                │
│ - email, name, picture - профиль пользователя               │
│ - aud (audience) - для кого предназначен токен              │
│ - iss (issuer) - кто выдал токен                            │
│ - exp (expiration) - когда истекает                         │
└─────────────────────────────────────────────────────────────┘
```

------

## 2. Service-to-service security

### 🔐 Mutual TLS (mTLS)

#### Архитектура mTLS:

```
┌─────────────────────────────────────────────────────────────┐
│                     mTLS ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Service A                              Service B           │
│  ┌─────────────┐                        ┌─────────────┐     │
│  │ Client Cert │◄─────── mTLS ─────────▶│ Server Cert │     │
│  │ Private Key │      Connection        │ Private Key │     │
│  └─────────────┘                        └─────────────┘     │
│         │                                       │           │
│         ▼                                       ▼           │
│  ┌─────────────┐                        ┌─────────────┐     │
│  │   CA Cert   │                        │   CA Cert   │     │
│  │ (validates  │                        │ (validates  │     │
│  │ Service B)  │                        │ Service A)  │     │
│  └─────────────┘                        └─────────────┘     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Настройка mTLS с помощью cert-manager:

```yaml
# Certificate Authority
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: ca-issuer
spec:
  ca:
    secretName: ca-key-pair

---
# Service Certificate
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: service-a-cert
spec:
  secretName: service-a-tls
  issuerRef:
    name: ca-issuer
    kind: ClusterIssuer
  dnsNames:
  - service-a.default.svc.cluster.local
  - service-a
```

------

### 🕸️ Service Mesh Security

#### Istio Security Architecture:

```
┌─────────────────────────────────────────────────────────────┐
│                    ISTIO SECURITY LAYERS                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │                  IDENTITY                               │ │
│ │ SPIFFE ID для каждого сервиса                           │ │
│ └─────────────────────────────────────────────────────────┘ │
│                          │                                 │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │                 COMMUNICATION                           │ │
│ │ Automatic mTLS между всеми сервисами                    │ │
│ └─────────────────────────────────────────────────────────┘ │
│                          │                                 │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │                AUTHORIZATION                            │ │
│ │ RBAC policies для контроля доступа                      │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Пример Istio Authorization Policy:

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: payment-service-policy
spec:
  selector:
    matchLabels:
      app: payment-service
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/order-service"]
  - to:
    - operation:
        methods: ["POST"]
        paths: ["/api/v1/payments"]
```

------

### 🔑 API Keys и Service Accounts

#### Service Account Architecture:

```
┌─────────────────────────────────────────────────────────────┐
│                  SERVICE ACCOUNT PATTERN                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Identity Provider                                           │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │                Service Accounts                         │ │
│ │ ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │ │
│ │ │ order-svc   │  │ payment-svc │  │ user-svc    │     │ │
│ │ │ ID: sa-001  │  │ ID: sa-002  │  │ ID: sa-003  │     │ │
│ │ │ Scope: ord* │  │ Scope: pay* │  │ Scope: usr* │     │ │
│ │ └─────────────┘  └─────────────┘  └─────────────┘     │ │
│ └─────────────────────────────────────────────────────────┘ │
│                          │                                 │
│                          ▼                                 │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │                    API Gateway                          │ │
│ │ Route + Validate + Rate Limit                           │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

------

## 3. Network Security

### 🔒 TLS Everywhere

#### TLS Termination Strategies:

```
┌─────────────────────────────────────────────────────────────┐
│                   TLS TERMINATION OPTIONS                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ 1. Edge Termination (Load Balancer)                        │
│ Client ──TLS──▶ LB ──HTTP──▶ Services                      │
│ ✅ Простота  ❌ Небезопасно внутри кластера                │
│                                                             │
│ 2. Re-encryption                                            │
│ Client ──TLS──▶ LB ──TLS──▶ Services                       │
│ ✅ Безопасно  ⚠️ Сложность управления сертификатами        │
│                                                             │
│ 3. Passthrough                                              │
│ Client ──TLS──────────────▶ Services                       │
│ ✅ Максимальная безопасность  ❌ Нет L7 features           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Nginx Ingress с TLS:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - api.example.com
    secretName: api-tls
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

------

### 🌐 Network Policies в Kubernetes

#### Deny All Policy (Default):

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

#### Allow Specific Communication:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-payment-to-db
spec:
  podSelector:
    matchLabels:
      app: payment-service
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: postgresql
    ports:
    - protocol: TCP
      port: 5432
```

------

## 4. Secrets Management

### 🔐 Centralized Secrets Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   SECRETS MANAGEMENT                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│           External Secrets Operator                        │
│         ┌─────────────────────────────────────────────────┐ │
│         │                                                 │ │
│  ┌──────▼──────┐  ┌──────────────┐  ┌──────────────┐     │ │
│  │    Vault    │  │ AWS Secrets  │  │ Azure KeyVault│     │ │
│  │             │  │   Manager    │  │              │     │ │
│  └──────┬──────┘  └──────┬───────┘  └──────┬───────┘     │ │
│         │                │                 │             │ │
│         └────────────────┼─────────────────┘             │ │
│                          │                               │ │
│         ┌────────────────▼─────────────────┐             │ │
│         │          Kubernetes              │             │ │
│         │        Secret Objects            │             │ │
│         └────────────────┬─────────────────┘             │ │
│                          │                               │ │
│         ┌────────────────▼─────────────────┐             │ │
│         │           Pods                   │             │ │
│         │    (Environment Variables)       │             │ │
│         └──────────────────────────────────┘             │ │
│                                                           │ │
└─────────────────────────────────────────────────────────────┘
```

------

### 🔧 HashiCorp Vault Integration

#### Vault Agent Configuration:

```hcl
# vault-agent.hcl
vault {
  address = "https://vault.example.com:8200"
}

auto_auth {
  method "kubernetes" {
    mount_path = "auth/kubernetes"
    config = {
      role = "payment-service"
    }
  }
}

template {
  source      = "/etc/vault/templates/database.tpl"
  destination = "/etc/secrets/database.json"
  perms       = 0640
}
```

#### Database Template:

```json
{{- with secret "database/creds/payment-service" }}
{
  "username": "{{ .Data.username }}",
  "password": "{{ .Data.password }}",
  "database": "payments",
  "host": "postgres.example.com",
  "port": 5432
}
{{- end }}
```

------

### 📊 Secrets Rotation Strategy

```
┌─────────────────────────────────────────────────────────────┐
│                   SECRETS ROTATION                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Phase 1: Preparation                                        │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ 1. Generate new secret                                  │ │
│ │ 2. Store in secrets manager                             │ │
│ │ 3. Update applications gradually                        │ │
│ └─────────────────────────────────────────────────────────┘ │
│                          │                                 │
│ Phase 2: Rotation                                           │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ 1. Applications use new secret                          │ │
│ │ 2. Monitor for failures                                 │ │
│ │ 3. Rollback if needed                                   │ │
│ └─────────────────────────────────────────────────────────┘ │
│                          │                                 │
│ Phase 3: Cleanup                                            │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ 1. Remove old secret                                    │ │
│ │ 2. Update documentation                                 │ │
│ │ 3. Audit rotation process                               │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

------

## 5. Performance и Scalability

### 📈 Horizontal vs Vertical Scaling

#### Scaling Strategies Comparison:

```
┌─────────────────────────────────────────────────────────────┐
│                   SCALING STRATEGIES                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Horizontal Scaling (Scale Out)                              │
│ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐                           │
│ │Pod 1│ │Pod 2│ │Pod 3│ │Pod 4│                           │
│ │2CPU │ │2CPU │ │2CPU │ │2CPU │                           │
│ │4GB  │ │4GB  │ │4GB  │ │4GB  │                           │
│ └─────┘ └─────┘ └─────┘ └─────┘                           │
│ ✅ Fault tolerance   ✅ Cost effective                     │
│ ✅ Better load dist  ❌ Complexity                         │
│                                                             │
│ Vertical Scaling (Scale Up)                                 │
│ ┌─────────────────┐                                        │
│ │      Pod 1      │                                        │
│ │      8CPU       │                                        │
│ │      16GB       │                                        │
│ └─────────────────┘                                        │
│ ✅ Simple setup    ❌ Single point of failure              │
│ ✅ Less network    ❌ Resource limits                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

------

### 🚀 Auto-scaling Configuration

#### Horizontal Pod Autoscaler:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: payment-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: payment-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
```

#### Vertical Pod Autoscaler:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: payment-service-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: payment-service
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: payment-service
      maxAllowed:
        cpu: 2
        memory: 4Gi
      minAllowed:
        cpu: 100m
        memory: 128Mi
```

------

### ⚡ Caching Strategies

#### Multi-Level Caching Architecture:

```
┌─────────────────────────────────────────────────────────────┐
│                   CACHING ARCHITECTURE                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Level 1: Application Cache (In-Memory)                      │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ Caffeine/GuavaCache - frequently accessed data         │ │
│ │ TTL: 5-15 minutes                                       │ │
│ └─────────────────────────────────────────────────────────┘ │
│                          │                                 │
│ Level 2: Distributed Cache (Redis)                          │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ Redis Cluster - shared between services                │ │
│ │ TTL: 1-24 hours                                         │ │
│ └─────────────────────────────────────────────────────────┘ │
│                          │                                 │
│ Level 3: CDN/Edge Cache                                     │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ CloudFlare/AWS CloudFront - static content             │ │
│ │ TTL: 24 hours - 30 days                                 │ │
│ └─────────────────────────────────────────────────────────┘ │
│                          │                                 │
│ Level 4: Database                                           │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ PostgreSQL with query result caching                   │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Redis Cache Configuration:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-config
data:
  redis.conf: |
    maxmemory 2gb
    maxmemory-policy allkeys-lru
    timeout 300
    tcp-keepalive 60
    save 900 1
    save 300 10
    save 60 10000
```

------

## 6. Production Readiness

### 🏥 Health Checks Implementation

#### Health Check Endpoints:

```
┌─────────────────────────────────────────────────────────────┐
│                   HEALTH CHECK TYPES                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ /health/live (Liveness Probe)                               │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ Проверяет, работает ли приложение                       │ │
│ │ Если fail -> рестарт pod                                │ │
│ │ Простая проверка (HTTP 200)                             │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
│ /health/ready (Readiness Probe)                             │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ Проверяет, готово ли приложение принимать трафик        │ │
│ │ Если fail -> исключение из Service                      │ │
│ │ Проверка зависимостей (DB, Redis, etc.)                 │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
│ /health/startup (Startup Probe)                             │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ Проверяет, запустилось ли приложение                    │ │
│ │ Блокирует другие probes до успешного запуска            │ │
│ │ Полезно для медленно стартующих приложений               │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Пример реализации:

```java
@RestController
public class HealthController {
    
    @Autowired
    private DatabaseHealthIndicator databaseHealth;
    
    @Autowired
    private RedisHealthIndicator redisHealth;
    
    @GetMapping("/health/live")
    public ResponseEntity<Map<String, String>> liveness() {
        return ResponseEntity.ok(Map.of("status", "UP"));
    }
    
    @GetMapping("/health/ready")
    public ResponseEntity<Map<String, Object>> readiness() {
        Map<String, Object> health = new HashMap<>();
        
        boolean isHealthy = true;
        
        // Check database
        try {
            databaseHealth.check();
            health.put("database", "UP");
        } catch (Exception e) {
            health.put("database", "DOWN");
            isHealthy = false;
        }
        
        // Check Redis
        try {
            redisHealth.check();
            health.put("redis", "UP");
        } catch (Exception e) {
            health.put("redis", "DOWN");
            isHealthy = false;
        }
        
        health.put("status", isHealthy ? "UP" : "DOWN");
        
        return ResponseEntity
            .status(isHealthy ? HttpStatus.OK : HttpStatus.SERVICE_UNAVAILABLE)
            .body(health);
    }
}
```

------

### 🔄 Graceful Shutdown

#### Shutdown Hook Implementation:

```java
@Component
public class GracefulShutdownHook {
    
    private static final Logger logger = LoggerFactory.getLogger(GracefulShutdownHook.class);
    
    @Autowired
    private ConnectionPoolManager connectionPool;
    
    @Autowired
    private MessageQueueManager messageQueue;
    
    @PreDestroy
    public void onShutdown() {
        logger.info("Начинаем graceful shutdown...");
        
        // 1. Перестаем принимать новые запросы
        logger.info("Останавливаем прием новых запросов");
        
        // 2. Ждем завершения текущих запросов
        logger.info("Ждем завершения активных запросов...");
        waitForActiveRequests();
        
        // 3. Закрываем подключения к БД
        logger.info("Закрываем подключения к базе данных");
        connectionPool.close();
        
        // 4. Закрываем подключения к message queue
        logger.info("Закрываем подключения к message queue");
        messageQueue.close();
        
        logger.info("Graceful shutdown завершен");
    }
    
    private void waitForActiveRequests() {
        int timeout = 30; // seconds
        while (hasActiveRequests() && timeout > 0) {
            try {
                Thread.sleep(1000);
                timeout--;
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }
}
```

#### Kubernetes Deployment Configuration:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
spec:
  template:
    spec:
      terminationGracePeriodSeconds: 60
      containers:
      - name: payment-service
        image: payment-service:latest
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 2
        startupProbe:
          httpGet:
            path: /health/startup
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 30
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15"]
```

------

## 7. Distributed Tracing

### 🔍 OpenTelemetry Architecture

```
┌─────────────────────────────────────────────────────────────┐
│              DISTRIBUTED TRACING ARCHITECTURE               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│ │ Service A   │    │ Service B   │    │ Service C   │     │
│ │             │    │             │    │             │     │
│ │ ┌─────────┐ │    │ ┌─────────┐ │    │ ┌─────────┐ │     │
│ │ │ OTel    │ │    │ │ OTel    │ │    │ │ OTel    │ │     │
│ │ │ Agent   │ │    │ │ Agent   │ │    │ │ Agent   │ │     │
│ │ └─────────┘ │    │ └─────────┘ │    │ └─────────┘ │     │
│ └─────────────┘    └─────────────┘    └─────────────┘     │
│       │                  │                  │             │
│       └──────────────────┼──────────────────┘             │
│                          │                                │
│              ┌─────────────────────────────┐               │
│              │    OTel Collector           │               │
│              │  (Batching, Filtering)      │               │
│              └─────────────────────────────┘               │
│                          │                                │
│       ┌──────────────────┼──────────────────┐             │
│       │                  │                  │             │
│ ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│ │   Jaeger    │    │ Prometheus  │    │    Logs     │     │
│ │  (Traces)   │    │ (Metrics)   │    │ (Logging)   │     │
│ └─────────────┘    └─────────────┘    └─────────────┘     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

------

### 📊 Jaeger Configuration

#### Jaeger All-in-One Deployment:

```yaml
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
        ports:
        - containerPort: 16686
        - containerPort: 14268
        env:
        - name: COLLECTOR_OTLP_ENABLED
          value: "true"
        - name: COLLECTOR_OTLP_HTTP_HOST_PORT
          value: ":4318"
        - name: COLLECTOR_OTLP_GRPC_HOST_PORT
          value: ":4317"
---
apiVersion: v1
kind: Service
metadata:
  name: jaeger
spec:
  selector:
    app: jaeger
  ports:
  - name: ui
    port: 16686
    targetPort: 16686
  - name: collector
    port: 14268
    targetPort: 14268
  - name: otlp-grpc
    port: 4317
    targetPort: 4317
  - name: otlp-http
    port: 4318
    targetPort: 4318
```

------

### 🎯 Correlation IDs

#### Correlation ID Flow:

```
┌─────────────────────────────────────────────────────────────┐
│                   CORRELATION ID FLOW                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ 1. Request arrives                                          │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ GET /api/orders                                         │ │
│ │ X-Correlation-ID: req-123-456-789                       │ │
│ └─────────────────────────────────────────────────────────┘ │
│                          │                                 │
│ 2. Order Service processes                                  │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ [req-123-456-789] Processing order request              │ │
│ │ [req-123-456-789] Calling payment service               │ │
│ └─────────────────────────────────────────────────────────┘ │
│                          │                                 │
│ 3. Payment Service call                                     │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ POST /api/payments                                      │ │
│ │ X-Correlation-ID: req-123-456-789                       │ │
│ │ [req-123-456-789] Processing payment                    │ │
│ └─────────────────────────────────────────────────────────┘ │
│                          │                                 │
│ 4. All logs correlated                                      │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ ElasticSearch query: X-Correlation-ID:req-123-456-789   │ │
│ │ Shows complete request flow across all services         │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Spring Boot Implementation:

```java
@Component
public class CorrelationIdFilter implements Filter {
    
    private static final String CORRELATION_ID_HEADER = "X-Correlation-ID";
    private static final String MDC_KEY = "correlationId";
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        
        String correlationId = httpRequest.getHeader(CORRELATION_ID_HEADER);
        if (correlationId == null || correlationId.isEmpty()) {
            correlationId = UUID.randomUUID().toString();
        }
        
        // Set in MDC for logging
        MDC.put(MDC_KEY, correlationId);
        
        // Add to response
        httpResponse.setHeader(CORRELATION_ID_HEADER, correlationId);
        
        try {
            chain.doFilter(request, response);
        } finally {
            MDC.remove(MDC_KEY);
        }
    }
}
```

------

## 🎯 Практические задания

### 📋 Задание 1: OAuth 2.0 Authentication

**Цель:** Настроить OAuth 2.0 аутентификацию для микросервисной архитектуры

**Задачи:**
1. Настроить OAuth 2.0 Provider (Keycloak или Auth0)
2. Создать Resource Server с JWT validation
3. Реализовать Authorization Code Flow
4. Настроить scopes и roles
5. Добавить refresh token rotation

**Критерии оценки:**
- ✅ Работающая аутентификация
- ✅ Proper JWT validation
- ✅ Secure token storage
- ✅ Error handling

------

### 📋 Задание 2: Health Checks и Monitoring

**Цель:** Реализовать comprehensive health checks и monitoring

**Задачи:**
1. Создать liveness, readiness, startup probes
2. Настроить Prometheus metrics
3. Создать Grafana dashboard
4. Реализовать graceful shutdown
5. Добавить alerting rules

**Критерии оценки:**
- ✅ Все типы health checks работают
- ✅ Metrics собираются корректно
- ✅ Dashboard показывает нужные метрики
- ✅ Alerts срабатывают при проблемах

------

### 📋 Задание 3: Distributed Tracing

**Цель:** Настроить distributed tracing с OpenTelemetry

**Задачи:**
1. Интегрировать OpenTelemetry в сервисы
2. Настроить Jaeger для trace collection
3. Реализовать correlation IDs
4. Создать custom spans для бизнес-операций
5. Настроить trace sampling

**Критерии оценки:**
- ✅ Traces отображаются в Jaeger
- ✅ Correlation IDs проходят через все сервисы
- ✅ Custom spans информативны
- ✅ Performance impact минимален

------

### 📋 Задание 4: Secrets Management

**Цель:** Настроить безопасное управление секретами

**Задачи:**
1. Развернуть HashiCorp Vault
2. Настроить Kubernetes authentication
3. Реализовать secrets rotation
4. Создать Vault policies
5. Интегрировать с External Secrets Operator

**Критерии оценки:**
- ✅ Vault работает и доступен
- ✅ Сервисы получают секреты из Vault
- ✅ Rotation работает автоматически
- ✅ Policies ограничивают доступ правильно

------

## ✅ Чек-лист готовности к продакшену

### 🔐 Безопасность
- [ ] Все коммуникации используют TLS
- [ ] Аутентификация настроена для всех API
- [ ] Авторизация на основе ролей реализована
- [ ] Секреты хранятся в специализированных хранилищах
- [ ] Network policies ограничивают трафик
- [ ] Vulnerability scanning настроен

### 📊 Monitoring & Observability
- [ ] Health checks для всех сервисов
- [ ] Metrics собираются в Prometheus
- [ ] Logs агрегируются централизованно
- [ ] Distributed tracing настроен
- [ ] Dashboards созданы в Grafana
- [ ] Alerting rules настроены

### 🚀 Performance & Scalability
- [ ] Auto-scaling настроен (HPA/VPA)
- [ ] Resource limits и requests заданы
- [ ] Caching strategy реализована
- [ ] Load balancing настроен
- [ ] Performance testing проведено

### 🔄 Reliability
- [ ] Graceful shutdown реализован
- [ ] Circuit breaker pattern используется
- [ ] Retry policies настроены
- [ ] Backup и disaster recovery план
- [ ] Chaos engineering testing проведено

------

## 📚 Дополнительные ресурсы

### 📖 Документация
- [OAuth 2.0 RFC](https://tools.ietf.org/html/rfc6749)
- [OpenID Connect Specification](https://openid.net/connect/)
- [Kubernetes Security](https://kubernetes.io/docs/concepts/security/)
- [Istio Security](https://istio.io/latest/docs/concepts/security/)

### 🛠️ Инструменты
- [HashiCorp Vault](https://www.vaultproject.io/)
- [Keycloak](https://www.keycloak.org/)
- [Jaeger](https://www.jaegertracing.io/)
- [OpenTelemetry](https://opentelemetry.io/)

### 📝 Best Practices
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [12 Factor App](https://12factor.net/)
- [Cloud Native Security](https://github.com/cncf/financial-user-group/tree/main/projects/k8s-security-best-practices)

------

## 🎓 Заключение

Безопасность и готовность к продакшену - это не разовая задача, а непрерывный процесс. Регулярно аудируйте вашу архитектуру, обновляйте security practices и мониторьте новые угрозы.

**Следующий блок:** Тестирование и качество - где мы рассмотрим комплексные стратегии тестирования микросервисных систем.