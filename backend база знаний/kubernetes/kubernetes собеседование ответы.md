# Kubernetes Interview - Развернутые ответы

## 1️⃣ Архитектура и Основы

### ⭐ 1. Что такое Kubernetes и какие проблемы он решает?

**Краткий ответ:** Kubernetes - это платформа оркестрации контейнеров, которая автоматизирует развертывание, масштабирование и управление контейнеризованными приложениями.

**Ключевые проблемы которые решает:**
- **Автоматическое масштабирование** - HPA/VPA
- **Service Discovery** - автоматическое обнаружение сервисов
- **Load Balancing** - распределение нагрузки
- **Rolling Updates** - обновления без простоя
- **Self-healing** - автоматическое восстановление

**Отличия от Docker Compose:**
```yaml
# Docker Compose - один хост
version: '3'
services:
  web:
    image: nginx
    replicas: 3  # Только на одном хосте

# Kubernetes - кластер
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3  # Распределяется по всему кластеру
```

**Когда K8s избыточен:** Простые приложения, разработка на локальной машине, статические сайты.

---

### ⭐⭐ 3. Путь HTTP-запроса от клиента до Pod

**Поток запроса:**
1. **Client** → **Ingress Controller**
2. **Ingress** → **Service** 
3. **Service** → **Pod** (через kube-proxy)

```yaml
# 1. Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        backend:
          service:
            name: app-service
            port:
              number: 80

# 2. Service
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080

# 3. Deployment
apiVersion: apps/v1
kind: Deployment
spec:
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
```

**Роль kube-proxy:** Создает iptables правила для load balancing между Pods с помощью IPVS или iptables.

**DNS resolution:** `service-name.namespace.svc.cluster.local`

---

### ⭐⭐⭐ 5. Сценарий: 3 master nodes, 2 вышли из строя

**Что происходит:**
- **etcd теряет quorum** (нужно минимум 2 из 3)
- **Кластер становится read-only**
- **Новые Pods не создаются**
- **Существующие Pods продолжают работать**

**etcd quorum формула:** `(n/2) + 1`
- 3 nodes: минимум 2 активных
- 5 nodes: минимум 3 активных

**Стратегии восстановления:**
1. **Восстановить один из узлов** - автоматическое восстановление quorum
2. **Restore из backup:**
```bash
# Создание backup
etcdctl snapshot save backup.db

# Восстановление
etcdctl snapshot restore backup.db \
  --data-dir=/var/lib/etcd-new
```

---

## 2️⃣ Workloads и Networking

### ⭐ 9. Deployment vs StatefulSet

**Deployment - для stateless приложений:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

**StatefulSet - для stateful приложений:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  serviceName: "db-service"
  replicas: 3
  template:
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      storageClassName: "fast-ssd"
      resources:
        requests:
          storage: 10Gi
```

**Ключевые отличия:**
- **Stable network identity:** db-0, db-1, db-2
- **Ordered deployment/scaling:** последовательное создание
- **Persistent storage:** каждый Pod получает свой PVC

---

### ⭐⭐ 11. Как работает Ingress

**Ingress Controller vs Ingress Resource:**
- **Controller** - реальная реализация (nginx, traefik, istio)
- **Resource** - конфигурация маршрутизации

```yaml
# Ingress Resource
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt"
spec:
  tls:
  - hosts:
    - api.example.com
    secretName: api-tls
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /api/v1
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /api/v2
        pathType: Prefix
        backend:
          service:
            name: api-v2-service
            port:
              number: 80
```

**SSL termination** происходит на уровне Ingress Controller.

---

### ⭐⭐⭐ 14. Микросегментация с Network Policies

**Default deny policy:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

**Multi-tier application isolation:**
```yaml
# Frontend может обращаться только к Backend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-policy
spec:
  podSelector:
    matchLabels:
      tier: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          tier: backend
    ports:
    - protocol: TCP
      port: 8080

---
# Backend может обращаться только к Database
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
  egress:
  - to:
    - podSelector:
        matchLabels:
          tier: database
    ports:
    - protocol: TCP
      port: 5432
```

---

## 3️⃣ Storage и Configuration

### ⭐ 17. emptyDir vs hostPath vs PersistentVolume

**emptyDir** - временное хранилище:
```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    volumeMounts:
    - name: temp-storage
      mountPath: /tmp
  volumes:
  - name: temp-storage
    emptyDir:
      sizeLimit: 1Gi
```

**hostPath** - доступ к файловой системе хоста:
```yaml
volumes:
- name: host-logs
  hostPath:
    path: /var/log
    type: Directory
```

**PersistentVolume** - управляемое постоянное хранилище:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-storage
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: fast-ssd
  resources:
    requests:
      storage: 10Gi
```

**Безопасность:** hostPath может дать доступ к чувствительным файлам хоста.

---

### ⭐⭐⭐ 21. Zero-downtime rotation секретов

**External secret management с CSI:**
```yaml
apiVersion: v1
kind: SecretProviderClass
metadata:
  name: app-secrets
spec:
  provider: vault
  parameters:
    vaultAddress: "https://vault.example.com"
    roleName: "app-role"
    objects: |
      - objectName: "database-password"
        secretPath: "secret/data/app"
        secretKey: "password"
```

**Projected volumes для rotation:**
```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    volumeMounts:
    - name: secrets
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secrets
    projected:
      sources:
      - secret:
          name: app-secret
      - configMap:
          name: app-config
```

**Application-level rotation:** Приложение должно отслеживать изменения файлов и перезагружать конфигурацию.

---

## 4️⃣ Security и RBAC

### ⭐ 25. Основы RBAC

**Role vs ClusterRole:**
```yaml
# Role - права в namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "create", "update", "delete"]

# ClusterRole - права на весь кластер  
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
```

**RoleBinding:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: development
subjects:
- kind: User
  name: john@company.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

---

### ⭐⭐⭐ 29. Admission Controllers в production

**OPA Gatekeeper policy:**
```yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        properties:
          labels:
            type: array
            items:
              type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels
        
        violation[{"msg": msg}] {
          required := input.parameters.labels
          provided := input.review.object.metadata.labels
          missing := required[_]
          not provided[missing]
          msg := sprintf("Missing required label: %v", [missing])
        }

---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: must-have-env
spec:
  match:
    kinds:
    - apiGroups: ["apps"]
      kinds: ["Deployment"]
  parameters:
    labels: ["environment", "team", "version"]
```

---

## 5️⃣ Production Operations

### ⭐⭐ 35. Horizontal Pod Autoscaler

**HPA с custom metrics:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 2
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "100"
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

**Debugging HPA:**
```bash
# Проверить статус HPA
kubectl describe hpa app-hpa

# Проверить metrics server
kubectl top pods
kubectl top nodes

# Проверить custom metrics
kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1
```

---

### ⭐⭐⭐ 38. Cluster upgrade в production

**Rolling upgrade стратегия:**
1. **Backup etcd** перед началом
2. **Upgrade control plane** по одному узлу
3. **Upgrade worker nodes** с drain/uncordon

```bash
# 1. Drain node
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data

# 2. Upgrade kubelet and kubeadm
apt-mark unhold kubelet kubeadm kubectl
apt-get update && apt-get install -y kubelet=1.28.x kubeadm=1.28.x
apt-mark hold kubelet kubeadm kubectl

# 3. Upgrade node
kubeadm upgrade node

# 4. Restart kubelet
systemctl daemon-reload
systemctl restart kubelet

# 5. Uncordon node
kubectl uncordon node-1
```

**API deprecation handling:**
```bash
# Найти устаревшие API
kubectl api-resources --api-group=extensions

# Проверить использование deprecated APIs
kubectl get events --field-selector reason=FailedCreate
```

---

### ⭐⭐⭐ 39. Performance optimization на scale

**etcd tuning:**
```yaml
# etcd configuration
--quota-backend-bytes=8589934592  # 8GB
--max-request-bytes=33554432      # 32MB
--grpc-keepalive-min-time=30s
--grpc-keepalive-interval=30s
--grpc-keepalive-timeout=5s
```

**API server optimization:**
```yaml
--max-requests-inflight=3000
--max-mutating-requests-inflight=1000
--request-timeout=300s
--etcd-compaction-interval=300s
```

**Node optimization:**
```bash
# Увеличить лимиты для kubelet
echo 'fs.inotify.max_user_watches=1048576' >> /etc/sysctl.conf
echo 'fs.inotify.max_user_instances=8192' >> /etc/sysctl.conf

# Kernel tuning
echo 'net.core.somaxconn=32768' >> /etc/sysctl.conf
echo 'net.ipv4.ip_local_port_range=1024 65535' >> /etc/sysctl.conf
```

---

## 🔧 Практические сценарии

### Troubleshooting: Pod не может подключиться к Service

**План диагностики:**
1. **Проверить Service endpoints:**
```bash
kubectl get endpoints service-name
kubectl describe service service-name
```

2. **Проверить DNS resolution:**
```bash
kubectl run debug --image=busybox --rm -it -- nslookup service-name.namespace.svc.cluster.local
```

3. **Проверить Network Policies:**
```bash
kubectl describe networkpolicy -n namespace
```

4. **Проверить kube-proxy:**
```bash
# На node
iptables -t nat -L | grep service-name
```

**Частые причины:**
- Неправильные селекторы в Service
- Network Policy блокирует трафик  
- DNS проблемы (CoreDNS)
- kube-proxy не работает

---

## 💡 Ключевые принципы для собеседования

1. **Безопасность прежде всего** - всегда думайте о security implications
2. **Production-ready подход** - учитывайте мониторинг, логирование, backup
3. **Объясняйте компромиссы** - каждое решение имеет trade-offs
4. **Приводите примеры из опыта** - реальные случаи из практики
5. **Думайте о масштабе** - как решение поведет себя при росте нагрузки