# 🔒 Блок 4: Безопасность и управление доступом в Kubernetes

**⏱️ Продолжительность:** 2-3 недели  
**🎯 Цель:** Освоить принципы безопасности в Kubernetes  
**🎓 Уровень:** Средний → Продвинутый

---

## 📖 Введение в безопасность Kubernetes

Безопасность в Kubernetes — это не одна технология, а комплексная система защиты, построенная на принципах:

**🛡️ Многоуровневая защита (Defense in Depth)**
```
┌─────────────────────────────────────────┐
│           Cloud/Network Layer           │
├─────────────────────────────────────────┤
│           Cluster Layer                 │
├─────────────────────────────────────────┤
│           Node Layer                    │
├─────────────────────────────────────────┤
│           Pod/Container Layer           │
└─────────────────────────────────────────┘
```

**🔐 Принцип минимальных привилегий (Principle of Least Privilege)**
- Каждый компонент получает только необходимые права
- Регулярный аудит и пересмотр разрешений
- Временные токены вместо постоянных ключей

---

## 4.1 RBAC (Role-Based Access Control)

### 🎯 Концепция RBAC

RBAC в Kubernetes — это система авторизации, которая регулирует доступ к ресурсам кластера на основе ролей пользователей.

**Философия RBAC:**
- **"Кто?"** (Субъект) → **"Что может делать?"** (Роль) → **"С чем?"** (Ресурс)

---

### 🏗️ Архитектура RBAC

```
        ПОЛЬЗОВАТЕЛИ/СЕРВИСЫ
              │
              │ связывается через
              ▼
         РОЛИ (ROLES)
              │
              │ определяют права на
              ▼
      РЕСУРСЫ KUBERNETES
```

**Ключевые компоненты:**

1. **Субъекты (Subjects)**
   - Users (пользователи)
   - Groups (группы пользователей)
   - Service Accounts (сервисные учетные записи)

2. **Роли (Roles)**
   - **Role** — права в рамках namespace
   - **ClusterRole** — права на уровне всего кластера

3. **Привязки (Bindings)**
   - **RoleBinding** — связывает субъект с ролью в namespace
   - **ClusterRoleBinding** — связывает субъект с кластерной ролью

---

### 🎭 Типы ролей и их область действия

```
CLUSTER LEVEL                    NAMESPACE LEVEL
┌─────────────────┐             ┌─────────────────┐
│   ClusterRole   │◄────────────┤      Role       │
│                 │             │                 │
│ • Nodes         │             │ • Pods          │
│ • Namespaces    │             │ • Services      │
│ • PVs           │             │ • ConfigMaps    │
│ • CRDs          │             │ • Secrets       │
└─────────────────┘             └─────────────────┘
        │                               │
        ▼                               ▼
ClusterRoleBinding                RoleBinding
```

**Когда использовать Role vs ClusterRole:**

**Role** — используется когда:
- Нужно ограничить доступ в пределах одного namespace
- Работа с обычными ресурсами приложений (pods, services, configmaps)
- Команда разработчиков работает только со своим проектом

**ClusterRole** — используется когда:
- Нужен доступ к ресурсам всего кластера
- Работа с глобальными ресурсами (nodes, persistent volumes)
- Администрирование кластера

---

### 🎯 Принципы создания эффективных ролей

**1. Гранулярность разрешений**
```
ПЛОХО: verb: ["*"]
ХОРОШО: verb: ["get", "list", "watch"]

ПЛОХО: resources: ["*"]  
ХОРОШО: resources: ["pods", "services"]
```

**2. Использование resourceNames для точечного доступа**
```
Пример: доступ только к конкретному секрету
resources: ["secrets"]
resourceNames: ["my-app-secret"]
```

**3. Иерархия ролей**
```
    cluster-admin (все права)
           │
    ┌──────┴──────┐
    │             │
namespace-admin  view-only
    │             │
app-developer  auditor
```

---

### 🔍 Аудит и отладка RBAC

**Команды для диагностики:**

1. **Проверка прав текущего пользователя:**
   ```bash
   kubectl auth can-i <verb> <resource>
   kubectl auth can-i create pods
   kubectl auth can-i get secrets --namespace=production
   ```

2. **Проверка прав другого пользователя:**
   ```bash
   kubectl auth can-i get pods --as=developer@company.com
   ```

3. **Просмотр всех привязок:**
   ```bash
   kubectl get rolebindings,clusterrolebindings --all-namespaces
   ```

**📊 Матрица анализа RBAC:**
```
┌──────────────┬───────┬─────────┬────────┬─────────┐
│  Пользователь│ Роль  │ Ресурс  │ Глагол │ Результат│
├──────────────┼───────┼─────────┼────────┼─────────├
│ developer    │ edit  │ pods    │ create │ ✅ ДА   │
│ developer    │ edit  │ secrets │ delete │ ❌ НЕТ  │  
│ admin        │ admin │ *       │ *      │ ✅ ДА   │
└──────────────┴───────┴─────────┴────────┴─────────┘
```

---

## 4.2 Service Accounts и аутентификация

### 🎯 Понимание Service Accounts

Service Account — это способ для подов и процессов внутри кластера аутентифицироваться в Kubernetes API.

**Различия между типами аккаунтов:**

```
┌─────────────────────────────────────────────────────┐
│                 ТИПЫ АККАУНТОВ                      │
├─────────────────────┬───────────────────────────────┤
│   USER ACCOUNTS     │      SERVICE ACCOUNTS         │
├─────────────────────┼───────────────────────────────┤
│ • Для людей         │ • Для приложений и сервисов   │
│ • Глобальные        │ • Привязаны к namespace       │
│ • Внешняя система   │ • Управляются Kubernetes      │  
│ • OIDC, LDAP        │ • JWT токены                  │
│ • Сертификаты       │ • Автоматическое обновление   │
└─────────────────────┴───────────────────────────────┘
```

---

### 🔐 Жизненный цикл Service Account Token

```
1. СОЗДАНИЕ ПОДА
   │
   │ Kubernetes автоматически:
   ▼
2. ПРИВЯЗКА SERVICE ACCOUNT
   │ (default или указанный)
   ▼
3. ГЕНЕРАЦИЯ JWT ТОКЕНА
   │ (с ограниченным временем жизни)
   ▼
4. МОНТИРОВАНИЕ В ПОД
   │ (/var/run/secrets/kubernetes.io/serviceaccount/)
   ▼
5. ИСПОЛЬЗОВАНИЕ ДЛЯ API ЗАПРОСОВ
   │
   ▼
6. АВТОМАТИЧЕСКОЕ ОБНОВЛЕНИЕ
   (при необходимости)
```

**Структура токена в поде:**
```
/var/run/secrets/kubernetes.io/serviceaccount/
├── token          # JWT токен для аутентификации
├── ca.crt         # Сертификат для проверки API сервера
└── namespace      # Текущий namespace
```

---

### 🎭 Типы аутентификации в Kubernetes

**1. Service Account Tokens (для подов)**
```
Формат JWT токена:
eyJhbGciOiJSUzI1NiIsImt...
│
├─ Header: алгоритм подписи
├─ Payload: claims (кто, когда, права)
└─ Signature: цифровая подпись
```

**2. OIDC (OpenID Connect) для пользователей**
```
┌─────────────┐    1. Аутентификация    ┌──────────────┐
│   Client    │──────────────────────► │ OIDC Provider│
│             │                        │ (Azure AD,   │
│             │◄──────────────────────  │  Google)     │
└─────────────┘    2. ID Token         └──────────────┘
       │
       │ 3. Запрос к K8s API с токеном
       ▼
┌─────────────┐
│ Kubernetes  │
│ API Server  │
└─────────────┘
```

**3. Webhook Token Authentication**
```
┌──────────────┐   Token   ┌─────────────┐   HTTP   ┌─────────────┐
│ Kubernetes   │──────────►│   Webhook   │─────────►│  External   │
│ API Server   │           │ Authenti-   │          │   Auth      │
│              │◄──────────│ cator       │◄─────────│  Service    │
└──────────────┘  Response └─────────────┘ Response └─────────────┘
```

---

### 🛡️ Best Practices для Service Accounts

**1. Принцип минимальных привилегий**
```
❌ ПЛОХО:
serviceAccountName: default
# использует стандартный SA со всеми правами

✅ ХОРОШО:
serviceAccountName: my-app-reader
# создан специально для приложения
```

**2. Отключение автоматического монтирования токенов**
```yaml
# На уровне Service Account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: no-token-sa
automountServiceAccountToken: false

# На уровне Pod
spec:
  automountServiceAccountToken: false
```

**3. Ротация токенов**
```
BOUND SERVICE ACCOUNT TOKENS:
├─ Время жизни: 1 час (по умолчанию)
├─ Привязаны к конкретному поду
├─ Автоматическое обновление
└─ Аудит использования
```

---

## 4.3 Pod Security Standards

### 🎯 Эволюция Pod Security

```
ИСТОРИЯ РАЗВИТИЯ БЕЗОПАСНОСТИ ПОДОВ:

2017-2021: Pod Security Policies (PSP)
    │ Проблемы: сложность, непредсказуемость
    ▼
2021-2022: Pod Security Standards (PSS)  
    │ Упрощение и стандартизация
    ▼
2022+: Pod Security Admission (PSA)
    │ Встроенный механизм контроля
```

---

### 🏗️ Три уровня Pod Security Standards

```
┌─────────────────────────────────────────────────────┐
│                 SECURITY LEVELS                     │
├─────────────┬─────────────────┬─────────────────────┤
│ PRIVILEGED  │    BASELINE     │     RESTRICTED      │
├─────────────┼─────────────────┼─────────────────────┤
│ Без         │ Минимальные     │ Строгие ограничения│
│ ограничений │ ограничения     │ по безопасности     │
│             │                 │                     │
│ • Legacy    │ • Известные     │ • Hardened          │
│   apps      │   privileged    │   applications      │
│ • System    │   escalations   │ • Defense in depth  │
│   pods      │ • Host access   │ • Compliance        │
└─────────────┴─────────────────┴─────────────────────┘
```

---

### 🔒 Детальный разбор уровней безопасности

**PRIVILEGED (Привилегированный)**
```
Разрешено ВСЕ:
├─ privileged: true
├─ hostNetwork: true  
├─ hostPID: true
├─ hostIPC: true
├─ Любые volume типы
└─ Запуск от root (UID 0)

Использование: системные поды, CNI, CSI драйверы
```

**BASELINE (Базовый)**
```
ЗАПРЕЩЕНО:
├─ privileged: true
├─ hostNetwork: true
├─ hostPID: true  
├─ hostIPC: true
├─ hostPath volumes
├─ hostPort mapping
├─ AppArmor: Unconfined
├─ SELinux: пользовательские типы
├─ /proc mount options
├─ Sysctl модификации
├─ privileged capabilities
└─ HostProcess containers (Windows)

РАЗРЕШЕНО: большинство обычных приложений
```

**RESTRICTED (Ограниченный)**
```
ВСЕ ОГРАНИЧЕНИЯ BASELINE +
├─ runAsNonRoot: true (обязательно)
├─ allowPrivilegeEscalation: false
├─ capabilities: только NET_BIND_SERVICE
├─ seccompProfile: RuntimeDefault или Localhost
├─ readOnlyRootFilesystem: true (рекомендуется)
└─ Ограниченные volume types
```

---

### 🎚️ Режимы работы Pod Security Admission

```
┌─────────────────────────────────────────────────────┐
│                 ENFORCEMENT MODES                   │
├─────────────┬─────────────────┬─────────────────────┤
│   ENFORCE   │      AUDIT      │        WARN         │
├─────────────┼─────────────────┼─────────────────────┤
│ Блокирует   │ Разрешает +     │ Разрешает +         │
│ нарушения   │ записывает в    │ предупреждение      │
│             │ audit log       │ в ответе            │
│             │                 │                     │
│ Production  │ Мониторинг      │ Development         │
│ окружения   │ соответствия    │ Testing             │
└─────────────┴─────────────────┴─────────────────────┘
```

**Конфигурация на уровне namespace:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: secure-namespace
  labels:
    # Обязательное соблюдение baseline
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/enforce-version: v1.28
    
    # Аудит на уровне restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/audit-version: v1.28
    
    # Предупреждения для restricted
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/warn-version: v1.28
```

---

### 🛡️ Security Context: детальная конфигурация

**Иерархия Security Context:**
```
POD SECURITY CONTEXT
├─ Общие настройки для всех контейнеров
├─ runAsUser: 1000
├─ runAsGroup: 1000  
├─ fsGroup: 1000
└─ supplementalGroups: [1000, 2000]
    │
    └─ CONTAINER SECURITY CONTEXT
       ├─ Переопределяет pod-level настройки
       ├─ allowPrivilegeEscalation: false
       ├─ runAsNonRoot: true
       ├─ readOnlyRootFilesystem: true
       └─ capabilities:
          ├─ drop: ["ALL"]
          └─ add: ["NET_BIND_SERVICE"]
```

**Практический пример secure пода:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  # Pod Security Context
  securityContext:
    runAsUser: 1000
    runAsGroup: 1000
    runAsNonRoot: true
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  
  containers:
  - name: app
    image: nginx:alpine
    # Container Security Context
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
        add:
        - NET_BIND_SERVICE
```

---

## 4.4 Admission Controllers

### 🎯 Концепция Admission Controllers

Admission Controllers — это плагины, которые перехватывают запросы к Kubernetes API после их аутентификации и авторизации, но до сохранения объектов в etcd.

**Место в архитектуре:**
```
┌─────────────┐   ┌──────────────┐   ┌─────────────────┐   ┌──────┐
│   Client    │──►│ Authentication│──►│   Authorization │──►│ etcd │
│  (kubectl)  │   │   Module     │   │     (RBAC)      │   │      │
└─────────────┘   └──────────────┘   └─────────────────┘   └──────┘
                                             │
                                             ▼
                                    ┌─────────────────┐
                                    │   ADMISSION     │
                                    │   CONTROLLERS   │
                                    └─────────────────┘
                                             │
                                             ├─ Mutating AC
                                             └─ Validating AC
```

---

### 🔄 Типы Admission Controllers

**1. Mutating Admission Controllers**
```
ФУНКЦИЯ: Изменяют объекты перед их сохранением

Примеры изменений:
├─ Добавление Service Account
├─ Инъекция sidecar контейнеров  
├─ Установка resource limits
├─ Добавление labels и annotations
└─ Конфигурация security context

Последовательность: FIFO (первый вошел - первый вышел)
```

**2. Validating Admission Controllers**
```
ФУНКЦИЯ: Проверяют объекты и могут отклонить запрос

Примеры проверок:
├─ Соответствие политикам безопасности
├─ Валидация resource quotas
├─ Проверка naming conventions
├─ Compliance требования
└─ Custom business rules

Последовательность: параллельно
```

---

### 🏗️ Встроенные Admission Controllers

**Ключевые встроенные контроллеры:**

```
┌─────────────────────────────────────────────────────┐
│            ВАЖНЫЕ ВСТРОЕННЫЕ КОНТРОЛЛЕРЫ            │
├─────────────────────┬───────────────────────────────┤
│   MUTATING          │         VALIDATING            │
├─────────────────────┼───────────────────────────────┤
│ ServiceAccount      │ NamespaceLifecycle           │
│ DefaultTolerations  │ LimitRanger                  │
│ DefaultStorageClass │ ResourceQuota                │
│ PodNodeSelector     │ PodSecurityPolicy (deprecated)│
│ PersistentVolume    │ NodeRestriction              │
│ Label               │ DenyServiceExternalIPs       │
└─────────────────────┴───────────────────────────────┘
```

**ServiceAccount Admission Controller:**
```
Автоматически:
├─ Добавляет service account к поду (если не указан)
├─ Монтирует API credentials как volume
├─ Создает image pull secrets
└─ Обеспечивает наличие default SA в namespace
```

**ResourceQuota Admission Controller:**
```
Проверяет:
├─ Не превышена ли квота CPU/Memory
├─ Количество объектов (pods, services, etc.)
├─ Storage requests
└─ Блокирует создание при превышении лимитов
```

---

### 🎯 Custom Admission Webhooks

**Архитектура Webhook:**
```
┌──────────────┐  HTTP POST  ┌─────────────────┐  HTTP POST  ┌─────────────┐
│ Kubernetes   │────────────►│   Your Webhook  │────────────►│  External   │
│ API Server   │             │    Service      │             │  Validator  │
│              │◄────────────│                 │◄────────────│             │
└──────────────┘  Response   └─────────────────┘  Response   └─────────────┘
                                      │
                                      ▼
                              ┌─────────────────┐
                              │   Business      │
                              │     Logic       │
                              │   Database      │
                              └─────────────────┘
```

**Пример использования Mutating Webhook:**
```yaml
# Автоматическая инъекция sidecar
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingAdmissionWebhook
metadata:
  name: sidecar-injector
webhooks:
- name: sidecar.example.com
  clientConfig:
    service:
      name: sidecar-injector
      namespace: system
      path: /mutate
  rules:
  - operations: ["CREATE"]
    apiGroups: [""]
    apiVersions: ["v1"]
    resources: ["pods"]
  admissionReviewVersions: ["v1", "v1beta1"]
```

---

### 🛡️ Open Policy Agent (OPA) Gatekeeper

**OPA Gatekeeper = Kubernetes + Policy as Code**

```
┌─────────────────────────────────────────────────────┐
│                OPA GATEKEEPER                       │
├─────────────────────┬───────────────────────────────┤
│   CONSTRAINT        │      CONSTRAINT TEMPLATE      │
│   (Instances)       │         (Rules)               │  
├─────────────────────┼───────────────────────────────┤
│ • Конкретные правила│ • Rego policy language       │
│ • Параметры         │ • Reusable logic             │
│ • Scope (namespaces)│ • Validation + Mutation      │
└─────────────────────┴───────────────────────────────┘
```

**Пример: запрет привилегированных контейнеров**

1. **ConstraintTemplate (шаблон правила):**
```yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredsecuritycontext
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredSecurityContext
      validation:
        properties:
          runAsNonRoot:
            type: boolean
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredsecuritycontext
        
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.securityContext.runAsNonRoot
          msg := "Container must run as non-root user"
        }
```

2. **Constraint (применение правила):**
```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredSecurityContext
metadata:
  name: must-run-as-nonroot
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    namespaces: ["production", "staging"]
  parameters:
    runAsNonRoot: true
```

---

## 4.5 Политики безопасности

### 🎯 Комплексный подход к безопасности

Политики безопасности в Kubernetes — это не отдельные инструменты, а **системный подход** к обеспечению безопасности на всех уровнях.

```
┌─────────────────────────────────────────────────────┐
│           LAYERS OF SECURITY POLICIES              │
├─────────────────────┬───────────────────────────────┤
│    PREVENTIVE       │         DETECTIVE             │
├─────────────────────┼───────────────────────────────┤
│ Network Policies    │ Audit Logging                │
│ Pod Security        │ Runtime Monitoring           │
│ RBAC               │ Anomaly Detection            │
│ Resource Quotas     │ Compliance Scanning          │
│ Admission Control   │ Vulnerability Assessment     │
└─────────────────────┴───────────────────────────────┘
```

---

### 🌐 Network Policies: Микросегментация

**Концепция Network Policies:**
```
DEFAULT KUBERNETES NETWORKING:
┌─────┐    ┌─────┐    ┌─────┐
│ Pod │◄──►│ Pod │◄──►│ Pod │  ← Все могут общаться
│  A  │    │  B  │    │  C  │    со всеми
└─────┘    └─────┘    └─────┘

WITH NETWORK POLICIES:
┌─────┐    ┌─────┐    ┌─────┐
│ Pod │───►│ Pod │ ✗  │ Pod │  ← Ограниченная связь
│  A  │    │  B  │    │  C  │    по правилам
└─────┘    └─────┘    └─────┘
```

**Типы Network Policies:**

1. **Ingress Policies (входящий трафик)**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}  # Применяется ко всем подам
  policyTypes:
  - Ingress
  # Нет ingress правил = запрет всего входящего трафика
```

2. **Egress Policies (исходящий трафик)**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-app-egress
spec:
  podSelector:
    matchLabels:
      app: web-app
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  # Разрешен только исходящий трафик к БД на порт 5432
```

**Стратегии сегментации:**
```
СТРАТЕГИЯ "DEFAULT DENY":
├─ Запретить весь трафик по умолчанию
├─ Явно разрешить только необходимые соединения
├─ Принцип минимальных привилегий
└─ Легче контролировать и аудировать

СТРАТЕГИЯ "PROGRESSIVE LOCKDOWN":  
├─ Начать с мониторинга трафика
├─ Постепенно внедрять ограничения
├─ Тестировать каждое изменение
└─ Минимизировать риск нарушения сервиса
```

---

### 📊 Resource Quotas: контроль ресурсов

**Зачем нужны Resource Quotas:**
```
БЕЗ КВОТ:                    С КВОТАМИ:
┌─────────────┐             ┌─────────────┐
│ Namespace A │ 90% CPU     │ Namespace A │ 30% CPU
├─────────────┤             ├─────────────┤  
│ Namespace B │ 8% CPU      │ Namespace B │ 30% CPU
├─────────────┤             ├─────────────┤
│ Namespace C │ 2% CPU      │ Namespace C │ 30% CPU
└─────────────┘             └─────────────┘
    Неспра-                     Справед-
    ведливо                     ливо
```

**Типы квот:**

1. **Compute Resources (вычислительные ресурсы)**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    requests.cpu: "4"      # Сумма всех CPU requests
    requests.memory: 8Gi   # Сумма всех Memory requests
    limits.cpu: "8"        # Сумма всех CPU limits
    limits.memory: 16Gi    # Сумма всех Memory limits
```

2. **Object Count Quotas (количество объектов)**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-quota
spec:
  hard:
    pods: "10"                    # Максимум 10 подов
    services: "5"                 # Максимум 5 сервисов
    secrets: "10"                 # Максимум 10 секретов
    configmaps: "10"              # Максимум 10 ConfigMaps
    persistentvolumeclaims: "4"   # Максимум 4 PVC
```

3. **Storage Quotas (квоты на хранилище)**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: storage-quota
spec:
  hard:
    requests.storage: 100Gi              # Общий объем storage
    persistentvolumeclaims: "4"          # Количество PVC
    requests.storage/ssd: 50Gi           # Квота на SSD storage class
    requests.storage/hdd: 50Gi           # Квота на HDD storage class
```

---

### 🚫 PodDisruptionBudgets: обеспечение доступности

**Концепция PDB:**
```
БЕЗ PDB:                     С PDB:
Maintenance Event            Maintenance Event
┌─────┐ ┌─────┐ ┌─────┐     ┌─────┐ ┌─────┐ ┌─────┐
│Pod 1│ │Pod 2│ │Pod 3│     │Pod 1│ │Pod 2│ │Pod 3│
└─────┘ └─────┘ └─────┘     └─────┘ └─────┘ └─────┘
   ↓       ↓       ↓           ↓       ↓       ↓
  ❌      ❌      ❌         ❌      ✅      ✅
Все поды удалены            Минимум 2 пода работают
= Недоступность сервиса     = Сервис продолжает работать
```

**Стратегии PDB:**
```yaml
# Стратегия 1: минимальное количество доступных подов
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-app-pdb
spec:
  minAvailable: 2          # Минимум 2 пода должны быть доступны
  selector:
    matchLabels:
      app: web-app

# Стратегия 2: максимальное количество недоступных подов  
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: database-pdb
spec:
  maxUnavailable: 1        # Максимум 1 под может быть недоступен
  selector:
    matchLabels:
      app: database

# Стратегия 3: процентное соотношение
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: worker-pdb
spec:
  minAvailable: 50%        # Минимум 50% подов должны работать
  selector:
    matchLabels:
      app: worker
```

---

### 🔐 Secrets Management Best Practices

**Иерархия управления секретами:**
```
┌─────────────────────────────────────────────────────┐
│              SECRETS MANAGEMENT                     │
├─────────────────────┬───────────────────────────────┤
│    AT REST          │        IN TRANSIT             │
├─────────────────────┼───────────────────────────────┤
│ etcd encryption     │ TLS everywhere               │
│ External key mgmt   │ Service mesh mTLS            │
│ Sealed secrets      │ Pod-to-pod encryption        │
│ CSI secret drivers  │ Network policies             │
└─────────────────────┴───────────────────────────────┘
```

**1. Encryption at Rest**
```yaml
# Конфигурация шифрования etcd
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources:
  - secrets
  providers:
  - aescbc:
      keys:
      - name: key1
        secret: <32-byte key>
  - identity: {}  # fallback для незашифрованных данных
```

**2. External Secrets Operator**
```yaml
# Интеграция с AWS Secrets Manager
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets-manager
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        secretRef:
          accessKeyID:
            name: aws-creds
            key: access-key-id
---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: database-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: database-secret
    creationPolicy: Owner
  data:
  - secretKey: username
    remoteRef:
      key: prod/database
      property: username
  - secretKey: password
    remoteRef:
      key: prod/database
      property: password
```

**3. Runtime Security с Secrets**
```yaml
# Безопасное использование secrets в поде
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    image: myapp:latest
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: database-secret
          key: password
    volumeMounts:
    - name: tls-certs
      mountPath: /etc/ssl/certs
      readOnly: true
  volumes:
  - name: tls-certs
    secret:
      secretName: tls-secret
      defaultMode: 0400  # Только владелец может читать
```

---

## 🎯 Практические рекомендации по безопасности

### 📋 Security Checklist для Production

**🔍 Аудит безопасности кластера:**
```
□ RBAC настроен с принципом минимальных привилегий
□ Pod Security Standards включены (минимум Baseline)
□ Network Policies ограничивают трафик между подами
□ Resource Quotas предотвращают resource exhaustion
□ Secrets зашифрованы в etcd
□ Admission Controllers настроены для политик безопасности
□ Audit logging включен и настроен
□ Node security patching автоматизирован
□ Image scanning интегрирован в CI/CD
□ Runtime security monitoring настроен
```

**🚨 Индикаторы компрометации (IoC):**
```
СЕТЕВАЯ АКТИВНОСТЬ:
├─ Необычные исходящие соединения
├─ Соединения с известными вредоносными IP
├─ Высокий объем DNS запросов
└─ Криптомайнинг трафик

ПОВЕДЕНИЕ ПОДОВ:
├─ Pods с привилегированными правами
├─ Необычное потребление ресурсов  
├─ Выполнение команд в runtime
└─ Модификация системных файлов

RBAC НАРУШЕНИЯ:
├─ Частые отказы в доступе
├─ Использование deprecated API
├─ Новые ServiceAccounts с широкими правами
└─ Изменения в ClusterRoles
```

---

## 🎓 Заключение блока

После изучения этого блока вы должны понимать:

✅ **Архитектуру безопасности Kubernetes** и принципы defense in depth  
✅ **RBAC систему** и умение создавать гранулярные роли  
✅ **Service Accounts** и различные методы аутентификации  
✅ **Pod Security Standards** и их практическое применение  
✅ **Admission Controllers** и создание custom policies  
✅ **Комплексные политики безопасности** для production-окружений

**🎯 Следующий шаг:** Применить полученные знания на практике, создав secure by default кластер с comprehensive security policies.

---

## 📚 Дополнительные ресурсы

**Документация:**
- [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/)
- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [RBAC Documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

**Инструменты:**
- [OPA Gatekeeper](https://open-policy-agent.github.io/gatekeeper/)
- [Falco Runtime Security](https://falco.org/)
- [kube-bench](https://github.com/aquasecurity/kube-bench) для CIS benchmarks

**Практика:**
- [Kubernetes Goat](https://madhuakula.com/kubernetes-goat/) - vulnerable cluster для практики
- [OWASP Kubernetes Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html)