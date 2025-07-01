# План изучения Kubernetes для Backend разработчиков

## Общая структура курса

**Продолжительность:** 12 недель (3 месяца)  
**Формат:** Теория + практические задания + проекты  
**Целевая аудитория:** Backend разработчики с опытом Docker  

---

## Неделя 1: Введение и основы контейнеризации

### Теоретическая часть (4 часа)
- [ ] **1.1** Проблемы масштабирования приложений
  - Ограничения традиционных деплойментов
  - Проблемы ручного управления серверами
  - Вызовы современных distributed систем

- [ ] **1.2** От Docker к Kubernetes
  - Возможности и ограничения Docker в продакшене
  - Что такое оркестрация контейнеров
  - Место Kubernetes в экосистеме

- [ ] **1.3** Ключевые преимущества Kubernetes
  - Автоматическое масштабирование
  - Самовосстановление
  - Декларативное управление
  - Портативность

### Практическая часть (4 часа)
- [ ] **Задание 1.1:** Установка kubectl и minikube
- [ ] **Задание 1.2:** Первый Pod из командной строки
- [ ] **Задание 1.3:** Сравнение Docker Compose и Kubernetes YAML
- [ ] **Задание 1.4:** Создание простейшего Deployment

### Проверочные вопросы
- Какие проблемы решает Kubernetes?
- В чем отличие между контейнеризацией и оркестрацией?
- Когда стоит использовать Kubernetes, а когда достаточно Docker?

---

## Неделя 2: Архитектура Kubernetes

### Теоретическая часть (5 часов)
- [ ] **2.1** Кластерная архитектура
  - Control Plane компоненты
  - Worker Nodes и их роль
  - Взаимодействие между компонентами

- [ ] **2.2** Control Plane в деталях
  - API Server как единая точка входа
  - etcd как хранилище состояния
  - Controller Manager и Scheduler

- [ ] **2.3** Worker Nodes
  - kubelet и управление Pod'ами
  - kube-proxy и сетевые функции
  - Container Runtime

### Практическая часть (4 часа)
- [ ] **Задание 2.1:** Изучение компонентов через kubectl
- [ ] **Задание 2.2:** Анализ логов системных компонентов
- [ ] **Задание 2.3:** Мониторинг состояния кластера
- [ ] **Задание 2.4:** Понимание API через kubectl describe

### Проверочные вопросы
- Какую роль играет каждый компонент Control Plane?
- Как происходит планирование Pod'ов на узлы?
- Что произойдет, если упадет API Server?

---

## Неделя 3: Основные объекты и ресурсы

### Теоретическая часть (6 часов)
- [ ] **3.1** Pod - минимальная единица развертывания
  - Концепция Pod'а и зачем группировать контейнеры
  - Sidecar pattern и примеры использования
  - Lifecycle Pod'а

- [ ] **3.2** ReplicaSet и управление репликами
  - Принцип работы и reconciliation loop
  - Селекторы и метки

- [ ] **3.3** Deployment - декларативные обновления
  - Rolling Update стратегия
  - Rollback и история изменений
  - Стратегии развертывания

- [ ] **3.4** Service - сетевой доступ
  - Типы Service (ClusterIP, NodePort, LoadBalancer)
  - Service Discovery через DNS
  - Endpoints и их управление

### Практическая часть (6 часов)
- [ ] **Задание 3.1:** Создание Pod с несколькими контейнерами
- [ ] **Задание 3.2:** Работа с ReplicaSet и масштабирование
- [ ] **Задание 3.3:** Deployment с различными стратегиями обновления
- [ ] **Задание 3.4:** Настройка Service для доступа к приложению
- [ ] **Проект 3.1:** Деплой простого веб-приложения с базой данных

### Проверочные вопросы
- Когда использовать Pod напрямую, а когда Deployment?
- Как происходит Rolling Update?
- Какой тип Service выбрать для разных сценариев?

---

## Неделя 4: Управление приложениями

### Теоретическая часть (5 часов)
- [ ] **4.1** StatefulSet для stateful приложений
  - Отличия от Deployment
  - Ordered deployment и scaling
  - Stable network identity

- [ ] **4.2** DaemonSet для системных сервисов
  - Случаи использования
  - Node selection и tolerations

- [ ] **4.3** Job и CronJob
  - Batch processing в Kubernetes
  - Управление завершением задач
  - Расписание и ретраи

### Практическая часть (5 часов)
- [ ] **Задание 4.1:** Развертывание StatefulSet с PostgreSQL
- [ ] **Задание 4.2:** Создание DaemonSet для логирования
- [ ] **Задание 4.3:** Batch jobs и scheduled tasks
- [ ] **Проект 4.1:** Полноценное приложение с различными типами workloads

### Проверочные вопросы
- Когда использовать StatefulSet вместо Deployment?
- Как обеспечить порядок запуска в StatefulSet?
- Какие задачи лучше решать через Job/CronJob?

---

## Неделя 5: Конфигурация и секреты

### Теоретическая часть (4 часа)
- [ ] **5.1** ConfigMap для конфигурационных данных
  - Создание и использование ConfigMap
  - Подключение как переменные окружения и файлы
  - Immutable ConfigMaps

- [ ] **5.2** Secrets для чувствительной информации
  - Типы Secret (generic, docker-registry, TLS)
  - Безопасное хранение и использование
  - Encryption at rest

- [ ] **5.3** Лучшие практики конфигурации
  - Разделение по средам
  - Ротация секретов
  - External Secret Management

### Практическая часть (4 часа)
- [ ] **Задание 5.1:** Конфигурация приложения через ConfigMap
- [ ] **Задание 5.2:** Работа с различными типами Secret
- [ ] **Задание 5.3:** Обновление конфигурации без downtime
- [ ] **Проект 5.1:** Настройка multi-environment конфигурации

### Проверочные вопросы
- Какую информацию хранить в ConfigMap, а какую в Secret?
- Как обновить конфигурацию running приложения?
- Какие есть способы ротации секретов?

---

## Неделя 6: Сетевое взаимодействие

### Теоретическая часть (5 часов)
- [ ] **6.1** Kubernetes Networking Model
  - Принципы сетевой модели
  - Pod-to-Pod коммуникация
  - CNI (Container Network Interface)

- [ ] **6.2** Service Discovery и DNS
  - Внутренний DNS в Kubernetes
  - Naming conventions
  - Cross-namespace communication

- [ ] **6.3** Ingress для HTTP(S) трафика
  - Ingress Controllers
  - Routing rules и path-based routing
  - TLS termination

- [ ] **6.4** Network Policies
  - Microsegmentation
  - Ingress и Egress правила
  - Default deny policies

### Практическая часть (5 часов)
- [ ] **Задание 6.1:** Исследование сетевой модели
- [ ] **Задание 6.2:** Настройка Ingress с SSL
- [ ] **Задание 6.3:** Создание Network Policies
- [ ] **Проект 6.1:** Микросервисная архитектура с network isolation

### Проверочные вопросы
- Как Pod'ы находят друг друга?
- В чем разница между Service и Ingress?
- Как обеспечить network security в кластере?

---

## Неделя 7: Хранение данных

### Теоретическая часть (4 часа)
- [ ] **7.1** Типы storage в Kubernetes
  - Ephemeral vs Persistent storage
  - Volumes и их типы
  - Use cases для каждого типа

- [ ] **7.2** Persistent Volumes и Claims
  - PV/PVC модель
  - Storage Classes
  - Dynamic provisioning

- [ ] **7.3** StatefulSet с Persistent Storage
  - Volume Claim Templates
  - Data persistence для баз данных
  - Backup и restore стратегии

### Практическая часть (4 часа)
- [ ] **Задание 7.1:** Работа с различными типами volumes
- [ ] **Задание 7.2:** Настройка dynamic provisioning
- [ ] **Задание 7.3:** StatefulSet с persistent данными
- [ ] **Проект 7.1:** Stateful приложение с backup/restore

### Проверочные вопросы
- Какой тип volume выбрать для разных сценариев?
- Как обеспечить persistence данных при restart Pod'ов?
- Что происходит с данными при удалении StatefulSet?

---

## Неделя 8: Мониторинг и отладка

### Теоретическая часть (4 часа)
- [ ] **8.1** Health Checks и Probes
  - Liveness, Readiness, Startup probes
  - HTTP, TCP, и Command проверки
  - Правильная настройка timeouts

- [ ] **8.2** Resource Management
  - Requests и Limits
  - QoS классы
  - Resource monitoring

- [ ] **8.3** Логирование в Kubernetes
  - Структурированные логи
  - Централизованное логирование
  - ELK/EFK стек

- [ ] **8.4** Troubleshooting
  - Систематический подход к диагностике
  - Общие проблемы и их решения
  - Debugging tools

### Практическая часть (6 часов)
- [ ] **Задание 8.1:** Настройка health checks
- [ ] **Задание 8.2:** Resource monitoring и optimization
- [ ] **Задание 8.3:** Centralized logging setup
- [ ] **Задание 8.4:** Практика troubleshooting
- [ ] **Проект 8.1:** Production-ready мониторинг

### Проверочные вопросы
- Как правильно настроить health checks?
- Что делать при OutOfMemory ошибках?
- Как найти причину CrashLoopBackOff?

---

## Неделя 9: Безопасность

### Теоретическая часть (5 часов)
- [ ] **9.1** Authentication и Authorization
  - Service Accounts
  - User management
  - OIDC integration

- [ ] **9.2** RBAC (Role-Based Access Control)
  - Roles и ClusterRoles
  - RoleBindings и принцип least privilege
  - Service Account permissions

- [ ] **9.3** Security Contexts
  - Pod и Container security contexts
  - Non-root users
  - Capabilities и privileges

- [ ] **9.4** Pod Security Standards
  - Privileged, Baseline, Restricted
  - Admission controllers
  - Security policies

### Практическая часть (5 часов)
- [ ] **Задание 9.1:** Настройка RBAC для приложения
- [ ] **Задание 9.2:** Security contexts и non-root containers
- [ ] **Задание 9.3:** Pod Security Standards implementation
- [ ] **Проект 9.1:** Security hardening existing application

### Проверочные вопросы
- Как ограничить доступ приложения к API?
- Почему важно запускать контейнеры от non-root пользователя?
- Какие security risks есть в Kubernetes по умолчанию?

---

## Неделя 10: Автоматизация и CI/CD

### Теоретическая часть (5 часов)
- [ ] **10.1** Helm - Package Manager
  - Charts и templates
  - Values и параметризация
  - Управление релизами

- [ ] **10.2** GitOps с ArgoCD
  - Принципы GitOps
  - Declarative configuration
  - Автоматическая синхронизация

- [ ] **10.3** CI/CD Pipelines
  - Integration с существующими CI системами
  - Building и pushing images
  - Automated testing в K8s

- [ ] **10.4** Deployment Strategies
  - Rolling Updates
  - Blue-Green deployments
  - Canary releases

### Практическая часть (6 часов)
- [ ] **Задание 10.1:** Создание Helm chart
- [ ] **Задание 10.2:** GitOps workflow setup
- [ ] **Задание 10.3:** CI/CD pipeline для K8s
- [ ] **Задание 10.4:** Advanced deployment strategies
- [ ] **Проект 10.1:** End-to-end автоматизация

### Проверочные вопросы
- Когда использовать Helm, а когда plain YAML?
- Как обеспечить rollback в GitOps модели?
- Какие метрики важны для Canary deployments?

---

## Неделя 11: Production-ready практики

### Теоретическая часть (4 часа)
- [ ] **11.1** Resource Management в продакшене
  - Правильная настройка requests/limits
  - Resource Quotas и LimitRanges
  - Capacity planning

- [ ] **11.2** High Availability
  - Multi-zone deployment
  - PodDisruptionBudgets
  - Cluster-level HA

- [ ] **11.3** Автомасштабирование
  - Horizontal Pod Autoscaler (HPA)
  - Vertical Pod Autoscaler (VPA)
  - Cluster Autoscaler

- [ ] **11.4** Backup и Disaster Recovery
  - Velero для cluster backup
  - Database backup strategies
  - Recovery planning

### Практическая часть (6 часов)
- [ ] **Задание 11.1:** Production resource configuration
- [ ] **Задание 11.2:** HA setup и testing
- [ ] **Задание 11.3:** Autoscaling configuration
- [ ] **Задание 11.4:** Backup и restore procedures
- [ ] **Проект 11.1:** Production-ready deployment

### Проверочные вопросы
- Как рассчитать resource requirements?
- Что такое blast radius и как его минимизировать?
- Как тестировать disaster recovery procedures?

---

## Неделя 12: Продвинутые темы и специализация

### Теоретическая часть (4 часа)
- [ ] **12.1** Custom Resources и Operators
  - CRD создание и использование
  - Operator pattern
  - Popular operators (Prometheus, Cert-Manager)

- [ ] **12.2** Service Mesh (Istio basics)
  - Traffic management
  - Security policies
  - Observability

- [ ] **12.3** Multi-tenancy
  - Namespace isolation
  - Resource sharing
  - Security boundaries

- [ ] **12.4** Advanced Scheduling
  - Node affinity и anti-affinity
  - Taints и tolerations
  - Priority classes

### Практическая часть (6 часов)
- [ ] **Задание 12.1:** Создание простого CRD
- [ ] **Задание 12.2:** Istio service mesh setup
- [ ] **Задание 12.3:** Multi-tenant cluster configuration
- [ ] **Финальный проект:** Комплексное приложение с использованием всех изученных концепций

### Проверочные вопросы
- Когда создавать собственные CRD?
- Какие преимущества дает Service Mesh?
- Как обеспечить изоляцию между tenants?

---

## Итоговая оценка и сертификация

### Финальный проект (20 часов)
**Цель:** Создать production-ready приложение демонстрирующее все ключевые концепции

**Требования:**
- [ ] Микросервисная архитектура (минимум 3 сервиса)
- [ ] Stateful компоненты (база данных)
- [ ] Proper resource management
- [ ] Security hardening
- [ ] Monitoring и logging
- [ ] CI/CD pipeline
- [ ] HA configuration
- [ ] Backup/restore procedures
- [ ] Documentation

### Подготовка к сертификации
- [ ] **CKAD preparation:** Practice exams и hands-on labs
- [ ] **Mock exams:** Симуляция реальных экзаменационных условий
- [ ] **Time management:** Практика решения задач за ограниченное время

---

## Дополнительные ресурсы для самостоятельного изучения

### Обязательная литература
- [ ] Kubernetes Official Documentation
- [ ] "Kubernetes in Action" - Marko Lukša
- [ ] "Kubernetes Up & Running" - Brendan Burns

### Практические платформы
- [ ] Katacoda Kubernetes scenarios
- [ ] Play with Kubernetes
- [ ] KillerCoda hands-on labs

### Community участие
- [ ] Kubernetes Slack channels
- [ ] Local meetups посещение
- [ ] Open source contributions

---

## Метрики успеха

### Технические навыки
- [ ] Уверенная работа с kubectl
- [ ] Создание production-ready манифестов
- [ ] Troubleshooting и debugging
- [ ] Security best practices применение
- [ ] CI/CD integration

### Практические результаты
- [ ] 3+ реальных проекта в портфолио
- [ ] Сертификация CKAD/CKA
- [ ] Contributions в open source проекты
- [ ] Презентация на meetup/конференции

### Карьерный рост
- [ ] Позиция с использованием Kubernetes
- [ ] Ментoring junior разработчиков
- [ ] Архитектурные решения участие
- [ ] Technical leadership в команде

---

*Этот план предназначен для интеграции с планами изучения Linux, Git и мониторинга для создания комплексной программы подготовки DevOps/Platform Engineer специалистов.*