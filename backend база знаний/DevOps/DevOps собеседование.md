# Оптимизированное интервью Backend разработчика: DevOps практики

## Структура интервью

**Время:** 90-120 минут  
**Формат:** 40 основных вопросов + практические кейсы  
**Оценка:** Динамическая по уровням сложности  

---

## Блиц-раунд (5 минут)
*Быстрая проверка базовых знаний*

1. Назовите 3 основные команды Docker
2. Что такое Pod в Kubernetes?
3. Основная цель CI/CD?
4. Что означает Infrastructure as Code?
5. Назовите 3 типа cloud сервисов

---

## Блок 1: Контейнеризация (10 вопросов)

### 🟢 Junior уровень

**1. Объясните разницу между Docker Image и Container**
- Как создается контейнер из образа?
- Что происходит при остановке контейнера?
- Приведите аналогию из реального мира

**2. Зачем нужен Dockerfile и какие основные инструкции вы знаете?**
- В чем разница между CMD и ENTRYPOINT?
- Когда использовать COPY vs ADD?
- Как оптимизировать размер образа?

### 🟡 Middle уровень

**3. Как организовать networking между контейнерами?**
- Какие типы сетей в Docker?
- Как контейнеры находят друг друга?
- Практика: опишите архитектуру для web-app + database

**4. Управление данными в Docker - volumes vs bind mounts**
- Когда использовать каждый подход?
- Как обеспечить персистентность данных?
- Сценарий: backup стратегия для базы данных в контейнере

### 🔴 Senior уровень

**5. Docker в production: безопасность и производительность**
- Почему не стоит запускать от root?
- Как ограничить ресурсы контейнера?
- Кейс: приложение потребляет слишком много памяти - план действий?

---

## Блок 2: Kubernetes (12 вопросов)

### 🟢 Junior уровень

**6. Основные компоненты Kubernetes и их роли**
- Что такое control plane?
- Объясните Pod, Service, Deployment
- Зачем нужны namespaces?

**7. Как деплоить приложение в Kubernetes?**
- Минимальный набор манифестов
- Что такое rolling update?
- Как проверить статус деплоя?

### 🟡 Middle уровень

**8. ConfigMaps vs Secrets - управление конфигурацией**
- Как монтировать конфигурацию в Pod?
- Стратегии обновления конфигурации
- Практика: как хранить database credentials?

**9. Автоскейлинг в Kubernetes**
- HPA vs VPA - когда что использовать?
- Какие метрики для автоскейлинга?
- Сценарий: пики нагрузки по расписанию

**10. Persistent Volumes и StatefulSets**
- Когда использовать StatefulSet vs Deployment?
- Как работает динамическое создание volumes?
- Кейс: миграция stateful приложения в Kubernetes

**11. Helm - пакетный менеджер**
- Структура Helm chart
- Как управлять зависимостями?
- Практика: создание chart для многокомпонентного приложения

### 🔴 Senior уровень

**12. Мониторинг и troubleshooting Kubernetes**
- Какие метрики кластера критичны?
- Подход к отладке проблем с Pod
- Инструменты для производственного мониторинга

**13. Безопасность Kubernetes**
- RBAC best practices
- Network Policies для микросервисов
- Pod Security Standards

**14. Kubernetes operators и CRDs**
- Когда создавать собственный operator?
- Примеры полезных operators
- Кейс: автоматизация backup процедур

**15. Стратегии deployment в production**
- Blue-Green vs Canary vs Rolling
- Как организовать multi-environment setup?
- GitOps подход с ArgoCD

**16. Производительность и масштабирование кластера**
- Cluster autoscaling стратегии
- Resource quotas и limits
- Кейс: оптимизация кластера для cost efficiency

**17. Service Mesh - Istio/Linkerd**
- Какие проблемы решает service mesh?
- Traffic management и security policies
- Когда service mesh избыточен?

---

## Блок 3: CI/CD (8 вопросов)

### 🟢 Junior уровень

**18. Принципы CI/CD и их важность**
- Этапы типичного pipeline
- Fail-fast принцип
- Как измерить эффективность процесса?

### 🟡 Middle уровень

**19. Дизайн эффективного CI pipeline**
- Стратегии тестирования в pipeline
- Параллелизация vs последовательность
- Практика: pipeline для микросервисной архитектуры

**20. Стратегии branching и deployment**
- Git Flow vs trunk-based development
- Feature flags в deployment
- Сценарий: hotfix в production

**21. Artifact management и версионирование**
- Стратегии версионирования
- Container registry best practices
- Кейс: rollback к предыдущей версии

### 🔴 Senior уровень

**22. Security в CI/CD (DevSecOps)**
- Интеграция security scanning
- Secrets management в pipeline
- Supply chain security

**23. Advanced CI/CD patterns**
- Deployment strategies для zero-downtime
- Pipeline as Code подходы
- Multi-cloud deployment challenges

**24. Monitoring и optimization CI/CD**
- DORA metrics
- Bottleneck identification
- Кейс: ускорение медленного pipeline

**25. Enterprise CI/CD**
- Governance и compliance
- Multi-team coordination
- Cost optimization strategies

---

## Блок 4: Infrastructure as Code (5 вопросов)

### 🟡 Middle уровень

**26. Terraform fundamentals**
- State management best practices
- Module organization
- Практика: multi-environment setup

**27. IaC testing и validation**
- Типы тестов для инфраструктуры
- Compliance as Code
- Drift detection strategies

### 🔴 Senior уровень

**28. Advanced IaC patterns**
- Terraform vs CloudFormation vs Pulumi
- State migration strategies
- Кейс: infrastructure refactoring без downtime

**29. Secrets и security в IaC**
- Sensitive data handling
- Integration с Vault/cloud secret managers
- Security scanning for infrastructure code

**30. IaC в enterprise окружении**
- Multi-account/subscription strategies
- Cost governance через IaC
- Collaboration patterns для teams

---

## Блок 5: Observability (5 вопросов)

### 🟡 Middle уровень

**31. Monitoring strategy design**
- Metrics, logs, traces - когда что использовать?
- SLI/SLO definition
- Практика: monitoring plan для e-commerce приложения

**32. Prometheus + Grafana ecosystem**
- PromQL для complex queries
- Alert manager configuration
- High availability setup

### 🔴 Senior уровень

**33. Distributed tracing и APM**
- OpenTelemetry implementation
- Performance bottleneck identification
- Кейс: debugging latency в микросервисах

**34. Log management scale**
- ELK vs cloud-native solutions
- Log retention и cost optimization
- Structured logging best practices

**35. Incident response и on-call**
- Effective alerting strategies
- Post-incident analysis process
- Chaos engineering integration

---

## Блок 6: Cloud & Security (5 вопросов)

### 🟡 Middle уровень

**36. Multi-cloud strategy**
- Cloud provider comparison
- Vendor lock-in mitigation
- Cost optimization across clouds

### 🔴 Senior уровень

**37. Cloud security architecture**
- Zero-trust implementation
- Identity и access management
- Compliance automation

**38. Disaster recovery и business continuity**
- RTO/RPO planning
- Multi-region deployment strategies
- Backup и restore automation

**39. FinOps и cost management**
- Cloud cost optimization techniques
- Reserved instances strategy
- Кейс: снижение cloud costs на 30%

**40. Enterprise cloud governance**
- Landing zone design
- Policy as Code implementation
- Multi-account security model

---

## Практические кейсы (выбрать 2-3 по времени)

### Кейс 1: Incident Response
*Сценарий:* Production приложение в Kubernetes недоступно. Пользователи жалуются на 500 ошибки.
- Последовательность диагностики
- Инструменты для investigation
- План восстановления
- Post-mortem процесс

### Кейс 2: Performance Optimization
*Сценарий:* Microservice показывает высокую latency при увеличении нагрузки.
- Monitoring setup для диагностики
- Возможные bottlenecks
- Scaling strategies
- Testing approach

### Кейс 3: Security Incident
*Сценарий:* Обнаружена уязвимость в одной из зависимостей production приложения.
- Immediate response plan
- Risk assessment
- Deployment strategy для fix
- Prevention measures

### Кейс 4: Migration Project
*Сценарий:* Миграция legacy приложения в cloud-native архитектуру.
- Assessment подход
- Migration strategy
- Risk mitigation
- Success metrics

---

## Система оценки

### Уровневые индикаторы:

#### 🟢 Junior (0-2 года)
- **Знания:** Понимает основные концепции
- **Практика:** Может выполнять задачи под руководством
- **Мышление:** Фокус на "как сделать"
- **Примеры ответов:** Перечисляет команды, описывает процедуры

#### 🟡 Middle (2-5 лет)
- **Знания:** Глубокое понимание инструментов и их взаимосвязей
- **Практика:** Самостоятельно проектирует решения
- **Мышление:** Фокус на "почему именно так"
- **Примеры ответов:** Объясняет trade-offs, предлагает альтернативы

#### 🔴 Senior (5+ лет)
- **Знания:** Архитектурное видение, понимание business impact
- **Практика:** Менторство, техническое лидерство
- **Мышление:** Фокус на "что это даст бизнесу"
- **Примеры ответов:** Связывает технические решения с бизнес-целями

### Критерии оценки:

**Отлично (8-10 баллов):**
- Полное понимание концепций
- Практические примеры из опыта
- Понимание trade-offs и альтернатив
- Может объяснить сложное простыми словами

**Хорошо (6-8 баллов):**
- Правильное понимание основ
- Некоторый практический опыт
- Базовое понимание ограничений

**Удовлетворительно (4-6 баллов):**
- Знает термины и основные концепции
- Ограниченный практический опыт
- Нуждается в руководстве

**Неудовлетворительно (0-4 балла):**
- Отсутствие понимания основ
- Нет практического опыта
- Не может объяснить концепции

### Итоговая оценка:
- **Senior:** 320+ баллов (80%+)
- **Middle:** 240-320 баллов (60-80%)
- **Junior:** 160-240 баллов (40-60%)
- **Не подходит:** <160 баллов (<40%)

---

## Рекомендации интервьюеру

### Подготовка:
1. Изучите резюме кандидата заранее
2. Подготовьте практические примеры из вашей компании
3. Настройте demo environment для hands-on задач

### Проведение:
1. **Warm-up:** Начните с простых вопросов для комфорта
2. **Адаптация:** Углубляйтесь или упрощайте в зависимости от ответов
3. **Практика:** Просите конкретные примеры из опыта
4. **Мышление:** Оценивайте подход к решению проблем
5. **Коммуникация:** Внимание на способность объяснять сложное

### Red flags:
- Не может объяснить базовые концепции
- Отсутствие практического опыта
- Неспособность признать незнание
- Нет понимания security или best practices
- Плохие коммуникативные навыки

### Green flags:
- Делится реальными кейсами из практики
- Понимает trade-offs решений
- Задает уточняющие вопросы
- Признает ограничения своих знаний
- Показывает continuous learning mindset