# Глубокое техническое собеседование по Kubernetes

## 🏗️ Архитектура и компоненты Control Plane

### Вопрос 1: Компоненты Control Plane
**Основной вопрос:** Расскажите о всех компонентах Control Plane и их взаимодействии.

**Подсказка:** Думайте о kube-apiserver, etcd, kube-scheduler, kube-controller-manager, cloud-controller-manager

**Углубление:**
- Как kube-apiserver обрабатывает HTTP-запросы?
- Какие типы данных хранит etcd и как происходит репликация?
- Какие алгоритмы использует scheduler для размещения Pod'ов?
- Сколько контроллеров входит в kube-controller-manager и как они работают?

### Вопрос 2: etcd и консистентность данных
**Основной вопрос:** Как etcd обеспечивает консистентность данных в кластере?

**Подсказка:** Raft consensus algorithm, quorum, leader election

**Углубление:**
- Что происходит при split-brain ситуации?
- Как настроить backup и restore etcd?
- Какие метрики etcd нужно мониторить?
- Как влияет latency сети на производительность etcd?

### Вопрос 3: API Server и аутентификация
**Основной вопрос:** Опишите процесс аутентификации и авторизации в kube-apiserver.

**Подсказка:** Authentication → Authorization → Admission Control

**Углубление:**
- Различия между ServiceAccount и User Account?
- Как работают Admission Controllers и какие встроенные есть?
- Что такое ValidatingAdmissionWebhook vs MutatingAdmissionWebhook?
- Как настроить OIDC интеграцию?

## 🔄 Рабочие компоненты (Worker Nodes)

### Вопрос 4: kubelet и Container Runtime
**Основной вопрос:** Как kubelet взаимодействует с container runtime?

**Подсказка:** CRI (Container Runtime Interface), containerd, cri-o

**Углубление:**
- Что изменилось после deprecation Docker в Kubernetes 1.24?
- Как работает Pod lifecycle и какие хуки существуют?
- Что такое Static Pods и когда их использовать?
- Как kubelet управляет ресурсами через cgroups?

### Вопрос 5: kube-proxy и сетевая маршрутизация
**Основной вопрос:** Какие режимы работы у kube-proxy и их различия?

**Подсказка:** userspace, iptables, ipvs, kernelspace

**Углубление:**
- Почему iptables режим стал default?
- Когда использовать IPVS режим и его преимущества?
- Как kube-proxy обновляет правила при изменении Service?
- Что такое sessionAffinity и как она реализована?

## 📦 Workload Resources

### Вопрос 6: Pod и контейнеры
**Основной вопрос:** Что происходит внутри Pod при его создании?

**Подсказка:** Pause container, shared namespaces, init containers

**Углубление:**
- Зачем нужен pause container?
- Как работают init containers и их use cases?
- Что такое sidecar containers и их отличие от init containers?
- Как Pod'ы разделяют сетевые ресурсы?

### Вопрос 7: Deployment vs StatefulSet vs DaemonSet
**Основной вопрос:** Когда использовать каждый тип workload ресурса?

**Подсказка:** Stateless vs Stateful приложения, rolling updates, persistent storage

**Углубление:**
- Как работает rolling update в Deployment?
- Что такое Headless Service и зачем он нужен для StatefulSet?
- Как StatefulSet гарантирует ordered deployment/scaling?
- Когда DaemonSet создает Pod на master нодах?

### Вопрос 8: Jobs и CronJobs
**Основной вопрос:** Как Kubernetes управляет batch workloads?

**Подсказка:** completions, parallelism, backoffLimit, restartPolicy

**Углубление:**
- Разница между OnFailure и Never restart policy?
- Как работает parallelism в Jobs?
- Что происходит с завершенными Job'ами?
- Как настроить timezone для CronJob?

## 🌐 Сетевая подсистема

### Вопрос 9: CNI и сетевая модель Kubernetes
**Основной вопрос:** Объясните сетевую модель Kubernetes и роль CNI.

**Подсказка:** Pod-to-Pod communication, Cluster networking, CNI plugins

**Углубление:**
- Требования к CNI plugin в Kubernetes?
- Различия между overlay и underlay сетями?
- Как работают популярные CNI: Calico, Flannel, Cilium, Weave?
- Что такое dual-stack networking?

### Вопрос 10: Services и их типы
**Основной вопрос:** Как работает каждый тип Service и их use cases?

**Подсказка:** ClusterIP, NodePort, LoadBalancer, ExternalName

**Углубление:**
- Как Service selector работает с endpoints?
- Что происходит с traffic при обновлении Service?
- Как работает service mesh интеграция?
- Что такое Headless Service и его применение?

### Вопрос 11: Ingress и Ingress Controllers
**Основной вопрос:** Как Ingress обеспечивает HTTP/HTTPS routing?

**Подсказка:** Ingress Controller, rules, backend services, TLS termination

**Углубление:**
- Различия между разными Ingress Controllers (nginx, traefik, istio)?
- Как настроить SSL/TLS termination?
- Что такое Ingress Classes?
- Как работает path-based и host-based routing?

### Вопрос 12: NetworkPolicies
**Основной вопрос:** Как NetworkPolicies обеспечивают микросегментацию?

**Подсказка:** ingress/egress rules, podSelector, namespaceSelector

**Углубление:**
- Что такое default deny policy?
- Как тестировать NetworkPolicies?
- Какие CNI поддерживают NetworkPolicies?
- Как NetworkPolicies влияют на DNS resolution?

## 💾 Управление хранилищем

### Вопрос 13: Volumes и их типы
**Основной вопрос:** Какие типы volumes существуют и их use cases?

**Подсказка:** emptyDir, hostPath, persistentVolume, configMap, secret

**Углубление:**
- Различия между volumes и persistentVolumes?
- Как работает volume mounting в multi-container Pod?
- Что происходит с данными при рестарте Pod?
- Как использовать projected volumes?

### Вопрос 14: Persistent Volumes и Storage Classes
**Основной вопрос:** Как работает динамическое выделение хранилища?

**Подсказка:** PV, PVC, StorageClass, dynamic provisioning, CSI

**Углубление:**
- Жизненный цикл PV (Binding, Using, Releasing, Reclaiming)?
- Что такое VolumeBindingMode и его влияние?
- Как работает CSI (Container Storage Interface)?
- Различия между ReadWriteOnce, ReadOnlyMany, ReadWriteMany?

### Вопрос 15: StatefulSet и persistent storage
**Основной вопрос:** Как StatefulSet управляет persistent storage?

**Подсказка:** volumeClaimTemplates, ordered deployment, stable network identity

**Углубление:**
- Что происходит с PVC при scaling down StatefulSet?
- Как работает rolling update в StatefulSet с volumes?
- Что такое OnDelete update strategy?
- Как backup данных в StatefulSet?

## 🎛️ Configuration Management

### Вопрос 16: ConfigMaps и Secrets
**Основной вопрос:** Лучшие практики работы с configuration data?

**Подсказка:** environment variables, volume mounts, immutable configs, encryption at rest

**Углубление:**
- Различия в безопасности между ConfigMap и Secret?
- Как работает automatic Secret rotation?
- Что такое immutable ConfigMaps/Secrets?
- Как использовать subPath в volume mounts?

### Вопрос 17: Environment Variables и их приоритет
**Основной вопрос:** Как Kubernetes резолвит environment variables?

**Подсказка:** env, envFrom, fieldRef, resourceFieldRef, secretKeyRef

**Углубление:**
- Порядок приоритета различных источников env vars?
- Как использовать downward API?
- Что происходит при конфликте имен переменных?
- Как debug environment variables в Pod?

## 🛡️ Безопасность

### Вопрос 18: RBAC (Role-Based Access Control)
**Основной вопрос:** Как настроить fine-grained доступ в кластере?

**Подсказка:** Role, ClusterRole, RoleBinding, ClusterRoleBinding, ServiceAccount

**Углубление:**
- Различия между Role и ClusterRole?
- Как работает privilege escalation prevention?
- Что такое aggregated ClusterRoles?
- Как audit RBAC permissions?

### Вопрос 19: Pod Security Standards
**Основной вопрос:** Как обеспечить безопасность Pod'ов на уровне кластера?

**Подсказка:** Pod Security Standards (Privileged, Baseline, Restricted), Security Context

**Углубление:**
- Что заменило Pod Security Policies в Kubernetes 1.25+?
- Как настроить Security Context на уровне Pod и Container?
- Что такое seccomp и AppArmor profiles?
- Как работает runAsNonRoot и runAsUser?

### Вопрос 20: Service Mesh и mTLS
**Основной вопрос:** Как service mesh интегрируется с Kubernetes security?

**Подсказка:** Istio, Linkerd, automatic mTLS, traffic policies

**Углубление:**
- Как работает automatic mTLS в service mesh?
- Что такое SPIFFE/SPIRE?
- Как service mesh влияет на NetworkPolicies?
- Различия в security моделях разных service mesh?

## 📊 Мониторинг и наблюдаемость

### Вопрос 21: Metrics и Prometheus интеграция
**Основной вопрос:** Как собираются метрики в Kubernetes?

**Подсказка:** metrics-server, Prometheus, custom metrics, HPA

**Углубление:**
- Различия между resource metrics и custom metrics?
- Как работает service discovery в Prometheus?
- Что такое metrics aggregation layer?
- Как настроить custom metrics для HPA?

### Вопрос 22: Logging архитектура
**Основной вопрос:** Лучшие практики централизованного логирования?

**Подсказка:** node-level logging, cluster-level logging, sidecar pattern, log rotation

**Углубление:**
- Различия между logging архитектурами?
- Как работает kubectl logs под капотом?
- Что происходит с логами при рестарте контейнера?
- Как настроить structured logging?

### Вопрос 23: Health Checks и Probes
**Основной вопрос:** Как Kubernetes определяет здоровье приложений?

**Подсказка:** livenessProbe, readinessProbe, startupProbe, probe handlers

**Углубление:**
- Когда использовать каждый тип probe?
- Как настроить probe timeouts и intervals?
- Что происходит при failure разных probes?
- Как gRPC health checks работают в Kubernetes?

## ⚖️ Автомасштабирование

### Вопрос 24: Horizontal Pod Autoscaler (HPA)
**Основной вопрос:** Как HPA принимает решения о масштабировании?

**Подсказка:** target utilization, metrics API, scaling algorithms, cooldown periods

**Углубление:**
- Как рассчитывается desired replica count?
- Что такое behavior policies в HPA v2?
- Как работает scale-down stabilization?
- Различия между resource и custom metrics scaling?

### Вопрос 25: Vertical Pod Autoscaler (VPA)
**Основной вопрос:** Когда и как использовать VPA?

**Подсказка:** resource recommendations, update modes, interaction with HPA

**Углубление:**
- Различия между VPA update modes?
- Можно ли использовать VPA и HPA одновременно?
- Как VPA влияет на Pod disruption?
- Что такое VPA recommender и его алгоритмы?

### Вопрос 26: Cluster Autoscaler
**Основной вопрос:** Как кластер автоматически масштабирует ноды?

**Подсказка:** node groups, scale-up/scale-down triggers, unschedulable Pods

**Углубление:**
- Как Cluster Autoscaler выбирает какую ноду удалить?
- Что такое node group priorities?
- Как работает integration с cloud providers?
- Что происходит с Pod'ами при scale-down?

## 🎯 Управление ресурсами

### Вопрос 27: Resource Requests и Limits
**Основной вопрос:** Как Kubernetes управляет CPU и память?

**Подсказка:** QoS classes (Guaranteed, Burstable, BestEffort), resource quotas, limit ranges

**Углубление:**
- Как определяется QoS class Pod'а?
- Различия в поведении CPU vs Memory limits?
- Что происходит при OOMKilled?
- Как работает CPU throttling?

### Вопрос 28: ResourceQuotas и LimitRanges
**Основной вопрос:** Как ограничить потребление ресурсов на уровне namespace?

**Подсказка:** resource quotas, limit ranges, default requests/limits

**Углубление:**
- Различия между ResourceQuota и LimitRange?
- Как quota enforcement работает?
- Что такое PriorityClass и как она влияет на scheduling?
- Как работает resource scoping в quotas?

### Вопрос 29: Node Affinity и Pod Affinity
**Основной вопрос:** Как управлять размещением Pod'ов на нодах?

**Подсказка:** node affinity, pod affinity/anti-affinity, taints and tolerations, topology spread constraints

**Углубление:**
- Различия между required и preferred affinity?
- Как работает pod anti-affinity для high availability?
- Что такое topology spread constraints и их use cases?
- Как влияют taints на scheduling process?

## 🔄 Lifecycle Management

### Вопрос 30: Rolling Updates и Rollbacks
**Основной вопрос:** Как Kubernetes обеспечивает zero-downtime deployments?

**Подсказка:** rolling update strategy, maxUnavailable, maxSurge, revision history

**Углубление:**
- Как работает rolling update algorithm?
- Что происходит при failed rolling update?
- Как настроить canary deployments нативно?
- Различия между Recreate и RollingUpdate strategies?

### Вопрос 31: Pod Disruption Budgets
**Основной вопрос:** Как защитить приложения от voluntary disruptions?

**Подсказка:** PDB, minAvailable, maxUnavailable, voluntary vs involuntary disruptions

**Углубление:**
- Различия между voluntary и involuntary disruptions?
- Как PDB влияет на node maintenance?
- Что происходит при конфликте PDB с node scaling?
- Как PDB работает с StatefulSets?

### Вопрос 32: Finalizers и Garbage Collection
**Основной вопрос:** Как Kubernetes управляет cleanup ресурсов?

**Подсказка:** finalizers, owner references, garbage collector, cascading deletion

**Углубление:**
- Как работает garbage collection в Kubernetes?
- Что такое foreground vs background cascading deletion?
- Как finalizers могут блокировать deletion?
- Что происходит с orphaned objects?

## 🔧 Операционные аспекты

### Вопрос 33: Backup и Disaster Recovery
**Основной вопрос:** Стратегии backup и recovery для Kubernetes кластера?

**Подсказка:** etcd backup, application data backup, cluster state restoration

**Углубление:**
- Как создать consistent backup всего кластера?
- Различия между application-level и infrastructure-level backup?
- Как тестировать disaster recovery procedures?
- Что такое cross-region cluster replication?

### Вопрос 34: Cluster Upgrades
**Основной вопрос:** Как безопасно обновить Kubernetes кластер?

**Подсказка:** control plane upgrade, node upgrade, version skew policy, drain и cordon

**Углубление:**
- Какая последовательность upgrade компонентов?
- Что такое version skew policy?
- Как работает node drain процесс?
- Различия между in-place и blue-green cluster upgrades?

### Вопрос 35: Troubleshooting
**Основной вопрос:** Методология диагностики проблем в Kubernetes?

**Подсказка:** kubectl debug commands, event analysis, resource status, logs correlation

**Углубление:**
- Как debug network connectivity issues?
- Что проверить при постоянных Pod restarts?
- Как диагностировать performance problems?
- Какие tools использовать для cluster-wide analysis?

## 🏗️ Advanced Topics

### Вопрос 36: Custom Resources и Operators
**Основной вопрос:** Как расширить API Kubernetes custom resources?

**Подсказка:** CRD, custom controllers, operator pattern, controller-runtime

**Углубление:**
- Различия между aggregated API и CRDs?
- Как написать efficient controller?
- Что такое operator lifecycle management?
- Как обеспечить backward compatibility CRDs?

### Вопрос 37: Webhooks и API Extensions
**Основной вопрос:** Как интегрировать external systems с Kubernetes API?

**Подсказка:** admission webhooks, conversion webhooks, API aggregation

**Углубление:**
- Как обеспечить high availability webhooks?
- Что происходит при webhook failure?
- Как работает webhook certificate management?
- Различия между mutating и validating webhooks?

### Вопрос 38: Multi-tenancy
**Основной вопрос:** Стратегии изоляции в shared Kubernetes кластере?

**Подсказка:** namespace isolation, network policies, resource quotas, hierarchical namespaces

**Углубление:**
- Soft vs hard multi-tenancy models?
- Как обеспечить complete tenant isolation?
- Что такое virtual clusters (vcluster)?
- Как sharing cluster resources между tenants?

### Вопрос 39: GitOps и CI/CD Integration
**Основной вопрос:** Лучшие практики GitOps с Kubernetes?

**Подсказка:** ArgoCD, Flux, declarative configuration, drift detection

**Углубление:**
- Различия между push и pull-based deployments?
- Как обеспечить secrets management в GitOps?
- Что такое progressive delivery patterns?
- Как интегрировать testing в GitOps pipeline?

### Вопрос 40: Edge Computing и IoT
**Основной вопрос:** Как Kubernetes адаптируется для edge computing?

**Подсказка:** K3s, MicroK8s, KubeEdge, resource constraints, intermittent connectivity

**Углубление:**
- Различия между full Kubernetes и lightweight distributions?
- Как работать с intermittent network connectivity?
- Что такое edge-cloud hybrid architectures?
- Как управлять updates в edge environments?

---

## 🎯 Практические сценарии

### Сценарий 1: Production Incident
Ваше production приложение внезапно начало возвращать 503 ошибки. HPA показывает максимальное количество реплик, но Pod'ы постоянно перезапускаются. Опишите пошаговую диагностику.

### Сценарий 2: Resource Optimization
Команда жалуется на высокие cloud costs. Кластер имеет 100+ микросервисов с различными resource requirements. Как оптимизировать использование ресурсов?

### Сценарий 3: Security Incident
Обнаружена уязвимость в одном из base images. Как быстро обновить все затронутые workloads минимизируя downtime?

### Сценарий 4: Multi-region Setup
Нужно развернуть application в multiple regions для disaster recovery. Как обеспечить data consistency и failover?

### Сценарий 5: Performance Issues
Приложение показывает high latency в production, но работает нормально в staging. Кластеры идентичны. Как диагностировать?

---

## 💡 Дополнительные области для углубления

1. **Service Mesh** - Istio, Linkerd, Consul Connect
2. **Serverless** - Knative, OpenFaaS, Kubeless
3. **AI/ML Workloads** - Kubeflow, Seldon, KServe
4. **Batch Processing** - Volcano, Argo Workflows
5. **Policy Management** - OPA/Gatekeeper, Kyverno
6. **Cost Optimization** - KubeCost, resource right-sizing
7. **Chaos Engineering** - Chaos Monkey, Litmus
8. **Security Scanning** - Falco, Twistlock, Aqua

Каждая область содержит множество технических нюансов и лучших практик, достойных отдельного глубокого изучения.