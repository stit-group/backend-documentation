# Компактное собеседование по Kubernetes для Backend разработчика

## 🎯 Структура интервью
- **Время**: 90-120 минут
- **Формат**: 5 блоков по 8 вопросов (40 вопросов всего)
- **Уровни**: Junior (⭐) → Middle (⭐⭐) → Senior (⭐⭐⭐)

---

## 1️⃣ Архитектура и Основы (20 минут)

### ⭐ Junior уровень

**1. Что такое Kubernetes и какие проблемы он решает?**
- Чем отличается от Docker/Docker Compose?
- Пример сценария где K8s избыточен

**2. Опишите основные компоненты кластера**
- Control Plane vs Worker Nodes
- Роль etcd, API Server, Scheduler
- Что произойдет если etcd недоступен?

### ⭐⭐ Middle уровень

**3. Объясните путь HTTP-запроса от клиента до Pod**
- Ingress → Service → Pod
- Роль kube-proxy в load balancing
- DNS resolution в кластере

**4. Как работает Kubernetes scheduler?**
- Алгоритм размещения Pods
- Node affinity vs Pod affinity
- Taints и tolerations

### ⭐⭐⭐ Senior уровень

**5. Сценарий: В кластере 3 master nodes, 2 вышли из строя. Что происходит?**
- etcd quorum и consensus
- Leader election процесс
- Стратегии восстановления

**6. Как обеспечить HA для Control Plane в production?**
- Multi-zone deployment
- Load balancing для API Server
- etcd backup/restore стратегии

**7. Объясните kubelet certificate rotation**
- Процесс обновления сертификатов
- Bootstrap токены
- Security implications

**8. Как отладить проблемы с сетевой связностью между nodes?**
- CNI debugging
- iptables rules анализ
- Network policies troubleshooting

---

## 2️⃣ Workloads и Networking (25 минут)

### ⭐ Junior уровень

**9. Чем Deployment отличается от StatefulSet?**
- Когда использовать каждый
- Примеры workloads для каждого типа
- Rolling update стратегии

**10. Объясните типы Services**
- ClusterIP vs NodePort vs LoadBalancer
- Headless services - когда нужны?
- External traffic policy

### ⭐⭐ Middle уровень

**11. Как работает Ingress и зачем он нужен?**
- Ingress Controller vs Ingress Resource
- SSL termination
- Path-based vs Host-based routing

**12. Сценарий: Pod не может подключиться к базе данных в другом namespace**
- Service discovery debugging
- DNS resolution проверка
- Network policies анализ

### ⭐⭐⭐ Senior уровень

**13. Как реализовать blue-green deployment без внешних инструментов?**
- Kubernetes-native подход
- Service selector switching
- Rollback стратегии

**14. Объясните микросегментацию с Network Policies**
- Ingress vs Egress rules
- Default deny политики
- Multi-tier application isolation

**15. Как работает Service Mesh (Istio/Linkerd)?**
- Sidecar pattern
- Traffic management vs Security
- Observability возможности

**16. Troubleshooting: Pods периодически теряют сетевую связность**
- CNI plugin issues
- iptables rules conflicts
- Network interface debugging

---

## 3️⃣ Storage и Configuration (20 минут)

### ⭐ Junior уровень

**17. Разница между emptyDir, hostPath и PersistentVolume?**
- Use cases для каждого типа
- Data persistence implications
- Security considerations

**18. ConfigMap vs Secret - когда что использовать?**
- Способы mount в Pods
- Encryption at rest
- Size limitations

### ⭐⭐ Middle уровень

**19. Как работает dynamic provisioning в Kubernetes?**
- Storage Classes роль
- PVC binding modes
- Volume expansion

**20. Сценарий: Приложение не может прочитать конфигурацию после обновления ConfigMap**
- Config reload mechanisms
- Immutable ConfigMaps
- Pod restart triggers

### ⭐⭐⭐ Senior уровень

**21. Как обеспечить zero-downtime rotation секретов?**
- External secret management (Vault, CSI)
- Projected volumes
- Application-level rotation

**22. StatefulSet storage management в production**
- Volume claim templates
- Orphaned volume handling
- Cross-AZ persistence

**23. CSI drivers - архитектура и troubleshooting**
- External components (provisioner, attacher)
- Volume snapshots
- Performance optimization

**24. Как организовать multi-environment конфигурации?**
- Kustomize vs Helm подходы
- Configuration drift prevention
- GitOps integration

---

## 4️⃣ Security и RBAC (15 минут)

### ⭐ Junior уровень

**25. Основы RBAC в Kubernetes**
- Role vs ClusterRole
- RoleBinding vs ClusterRoleBinding
- Service Accounts

**26. Pod Security Standards**
- Privileged vs Baseline vs Restricted
- Security Context настройки
- runAsNonRoot enforcement

### ⭐⭐ Middle уровень

**27. Как ограничить права Pod'а на minimum necessary?**
- RBAC best practices
- Network policies
- Resource limits

**28. Sценарий: Pod получает ошибку "forbidden" при обращении к API**
- RBAC debugging
- kubectl auth can-i
- Service Account permissions

### ⭐⭐⭐ Senior уровень

**29. Admission Controllers в production**
- Validating vs Mutating webhooks
- OPA Gatekeeper policies
- Custom admission controllers

**30. Container image security**
- Image scanning integration
- Signed images verification
- Supply chain security

**31. Runtime security monitoring**
- Behavioral analysis
- Anomaly detection
- Incident response

**32. Multi-tenant кластер security**
- Namespace isolation
- Resource quotas enforcement
- Cross-tenant communication prevention

---

## 5️⃣ Production Operations (30 минут)

### ⭐ Junior уровень

**33. Какие метрики критичны для мониторинга кластера?**
- Cluster health indicators
- Application metrics
- Resource utilization

**34. Основные команды для troubleshooting**
- kubectl logs, describe, get events
- Pod debugging commands
- Network connectivity testing

### ⭐⭐ Middle уровень

**35. Horizontal Pod Autoscaler настройка**
- CPU vs Memory vs Custom metrics
- Scale-up/down behavior
- HPA debugging

**36. Сценарий: Pod застрял в Terminating состоянии**
- Finalizers анализ
- Graceful shutdown
- Force deletion последствия

**37. Backup и disaster recovery стратегии**
- etcd backup automation
- Persistent data backup
- Cross-region DR

### ⭐⭐⭐ Senior уровень

**38. Cluster upgrade в production**
- Rolling upgrade стратегии
- API deprecation handling
- Rollback procedures

**39. Performance optimization на scale**
- etcd tuning
- API server optimization
- Node optimization

**40. Cost optimization стратегии**
- Resource right-sizing
- Spot instances integration
- Multi-zone cost analysis

---

## 🔧 Практические задания (выберите 2-3)

### Задание 1: YAML Review (10 минут)
```yaml
# Кандидат должен найти и исправить ошибки в манифесте
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: web-app  # Ошибка: не совпадает с selector
    spec:
      containers:
      - name: web
        image: nginx:latest  # Ошибка: latest tag
        resources:
          limits:
            memory: "128Mi"
            cpu: "100m"
          requests:
            memory: "256Mi"  # Ошибка: requests > limits
            cpu: "50m"
```

### Задание 2: Troubleshooting Scenario (15 минут)
**Проблема**: "Pods создаются, но недоступны через Service"
- Кандидат должен задать правильные вопросы
- Предложить план диагностики
- Объяснить возможные причины

### Задание 3: Архитектурный дизайн (15 минут)
**Задача**: Спроектировать Kubernetes setup для:
- Microservices приложение (3 сервиса)
- PostgreSQL база данных
- Redis cache
- Требования: HA, monitoring, security

---

## 📊 Система оценки

### ⭐ Junior (1-2 года опыта)
- ✅ Знает основные концепции
- ✅ Умеет создавать базовые манифесты
- ✅ Понимает основы troubleshooting
- ❌ Ограниченный production опыт

### ⭐⭐ Middle (2-4 года опыта)
- ✅ Глубокое понимание архитектуры
- ✅ Опыт production deployments
- ✅ Умеет проектировать решения
- ✅ Advanced troubleshooting skills

### ⭐⭐⭐ Senior (4+ года опыта)
- ✅ Экспертные знания внутренних механизмов
- ✅ Опыт работы с large-scale кластерами
- ✅ Security и compliance expertise
- ✅ Mentoring и knowledge sharing

## 🎯 Критерии принятия решения

| Позиция | Обязательные навыки | Желательные навыки |
|---------|-------------------|-------------------|
| **Junior** | Блоки 1-2 (⭐ уровень) | Базовое понимание остальных блоков |
| **Middle** | Блоки 1-3 (⭐⭐ уровень) | Блоки 4-5 (⭐ уровень) |
| **Senior** | Все блоки (⭐⭐ уровень) | 60%+ от ⭐⭐⭐ вопросов |

## 💡 Рекомендации интервьюеру

1. **Начинайте с комфортного уровня** - если кандидат junior, не мучайте senior вопросами
2. **Просите примеры из опыта** - "Расскажите случай когда вы..."
3. **Оценивайте мышление** - как кандидат подходит к решению проблем
4. **Обращайте внимание на security awareness** - критично для production
5. **Проверяйте soft skills** - способность объяснить сложные концепции

**Время распределения:**
- Теоретические вопросы: 70 минут
- Практические задания: 30-40 минут
- Вопросы кандидата: 10 минут