# Интервью Backend разработчика: DevOps практики и инструменты

## Инструкция по проведению интервью

**Время проведения:** 2-3 часа  
**Структура:** 80 основных вопросов + 5 углубляющих к каждому  
**Уровень сложности:** От junior до senior  
**Оценка:** Каждый блок оценивается отдельно

---

## Блок 1: Контейнеризация и Docker (10 вопросов)

### 1. Что такое Docker и какие проблемы он решает?
1. Объясните разницу между виртуализацией и контейнеризацией
2. Какие альтернативы Docker вы знаете?
3. Как Docker влияет на безопасность приложений?
4. Опишите архитектуру Docker (daemon, client, registry)
5. Какие ограничения у Docker в production среде?

### 2. Объясните разницу между Docker Image и Container
1. Как происходит процес создания container из image?
2. Что такое слои (layers) в Docker image?
3. Как оптимизировать размер Docker image?
4. Объясните принцип Copy-on-Write в Docker
5. Как работает кеширование слоев при сборке image?

### 3. Что такое Dockerfile и какие основные инструкции вы знаете?
1. Объясните разницу между CMD и ENTRYPOINT
2. Когда использовать COPY, а когда ADD?
3. Как правильно структурировать Dockerfile для оптимальной сборки?
4. Что такое multi-stage builds и когда их применять?
5. Как передавать аргументы в Dockerfile во время сборки?

### 4. Как управлять данными в Docker контейнерах?
1. Объясните разницу между volumes, bind mounts и tmpfs
2. Когда использовать named volumes, а когда anonymous?
3. Как резервировать и восстанавливать данные из volumes?
4. Что происходит с данными при удалении контейнера?
5. Как организовать sharing данных между контейнерами?

### 5. Что такое Docker Compose и зачем он нужен?
1. Объясните структуру docker-compose.yml файла
2. Как управлять зависимостями между сервисами?
3. Как передавать переменные окружения через Compose?
4. Объясните networking в Docker Compose
5. Как масштабировать сервисы с помощью Compose?

### 6. Как настроить networking между Docker контейнерами?
1. Какие типы сетей поддерживает Docker?
2. Как контейнеры обнаруживают друг друга?
3. Объясните разницу между bridge и host networking
4. Как настроить custom network?
5. Как обеспечить сетевую изоляцию контейнеров?

### 7. Какие best practices для безопасности Docker контейнеров?
1. Почему не стоит запускать контейнеры от root пользователя?
2. Как сканировать Docker images на уязвимости?
3. Что такое Docker secrets и как их использовать?
4. Как ограничить ресурсы контейнера?
5. Какие инструменты для security scanning Docker images вы знаете?

### 8. Как мониторить производительность Docker контейнеров?
1. Какие метрики важно отслеживать для контейнеров?
2. Как использовать docker stats и docker logs?
3. Какие инструменты для мониторинга контейнеров вы знаете?
4. Как настроить централизованное логирование для контейнеров?
5. Как отлаживать проблемы производительности в контейнерах?

### 9. Опишите процесс deployment Docker приложения в production
1. Какие стратегии deployment для контейнеризованных приложений?
2. Как обеспечить zero-downtime deployment?
3. Что такое health checks и как их настроить?
4. Как управлять секретами в production?
5. Какие инструменты для оркестрации контейнеров вы знаете?

### 10. Как оптимизировать производительность Docker приложений?
1. Какие факторы влияют на производительность контейнеров?
2. Как правильно настроить ресурсы (CPU, память) для контейнера?
3. Что такое cgroups и как они работают?
4. Как оптимизировать время старта контейнера?
5. Какие инструменты для профилирования приложений в контейнерах?

---

## Блок 2: Kubernetes и оркестрация (15 вопросов)

### 11. Что такое Kubernetes и какие проблемы он решает?
1. Чем Kubernetes отличается от Docker Swarm?
2. Объясните архитектуру Kubernetes cluster
3. Что такое control plane и worker nodes?
4. Какие компоненты входят в master node?
5. Как Kubernetes обеспечивает высокую доступность?

### 12. Объясните основные объекты Kubernetes: Pod, Service, Deployment
1. Почему Pod является минимальной единицей развертывания?
2. Какие типы Services существуют в Kubernetes?
3. В чем разница между Deployment и ReplicaSet?
4. Как Service обнаруживает Pods?
5. Что происходит при обновлении Deployment?

### 13. Что такое Kubernetes namespaces и зачем они нужны?
1. Как namespace влияет на сетевую изоляцию?
2. Можно ли ограничить ресурсы на уровне namespace?
3. Как объекты в разных namespace взаимодействуют?
4. Какие default namespaces создаются в Kubernetes?
5. Как настроить RBAC для namespace?

### 14. Объясните систему управления конфигурацией в Kubernetes
1. В чем разница между ConfigMap и Secret?
2. Как монтировать ConfigMap как volume?
3. Как обновлять конфигурацию без перезапуска Pod?
4. Что такое Sealed Secrets?
5. Как управлять секретами в GitOps подходе?

### 15. Как работает networking в Kubernetes?
1. Что такое Kubernetes network model?
2. Объясните принцип работы kube-proxy
3. Какие CNI плагины вы знаете?
4. Что такое Network Policies?
5. Как настроить Ingress controller?

### 16. Что такое Persistent Volumes в Kubernetes?
1. Объясните разницу между PV, PVC и StorageClass
2. Какие типы storage поддерживает Kubernetes?
3. Как работает динамическое создание volumes?
4. Что такое volume snapshots?
5. Как обеспечить backup данных в Kubernetes?

### 17. Объясните стратегии развертывания в Kubernetes
1. В чем разница между Rolling Update и Recreate?
2. Что такое Blue-Green deployment в контексте Kubernetes?
3. Как реализовать Canary deployment?
4. Что такое readiness и liveness probes?
5. Как настроить автоматический rollback?

### 18. Как работает автоскейлинг в Kubernetes?
1. Объясните принцип работы Horizontal Pod Autoscaler
2. Что такое Vertical Pod Autoscaler?
3. Как работает Cluster Autoscaler?
4. Какие метрики можно использовать для автоскейлинга?
5. Как настроить custom metrics для HPA?

### 19. Что такое Helm и зачем он нужен?
1. Объясните структуру Helm chart
2. Что такое Values файлы и как их использовать?
3. Как управлять зависимостями между charts?
4. Что такое Helm hooks?
5. Как создать собственный Helm repository?

### 20. Как мониторить Kubernetes cluster?
1. Какие компоненты cluster нужно мониторить?
2. Как настроить мониторинг с Prometheus в Kubernetes?
3. Что такое kube-state-metrics?
4. Как мониторить performance приложений в Kubernetes?
5. Какие инструменты для troubleshooting Kubernetes вы знаете?

### 21. Объясните безопасность в Kubernetes
1. Что такое RBAC в Kubernetes?
2. Как работают ServiceAccounts?
3. Что такое Pod Security Standards?
4. Как настроить Network Policies для изоляции?
5. Какие инструменты для security scanning Kubernetes вы знаете?

### 22. Что такое Operators в Kubernetes?
1. Чем Operator отличается от обычного controller?
2. Что такое Custom Resource Definitions (CRD)?
3. Как создать простой Operator?
4. Какие популярные Operators вы знаете?
5. Как Operators помогают в управлении stateful приложениями?

### 23. Как управлять stateful приложениями в Kubernetes?
1. Что такое StatefulSet и когда его использовать?
2. Как работает ordered deployment в StatefulSet?
3. Что такое headless Service?
4. Как обеспечить stable network identity для Pods?
5. Как решать проблемы с данными в stateful приложениях?

### 24. Объясните GitOps подход в Kubernetes
1. Какие принципы GitOps?
2. Что такое ArgoCD и как он работает?
3. Как организовать Git repository для GitOps?
4. Как обрабатывать секреты в GitOps?
5. Какие преимущества и недостатки GitOps подхода?

### 25. Как отлаживать проблемы в Kubernetes?
1. Какие команды kubectl для troubleshooting?
2. Как анализировать logs в Kubernetes?
3. Что делать, если Pod не запускается?
4. Как отлаживать проблемы с networking?
5. Какие инструменты для debugging Kubernetes приложений?

---

## Блок 3: CI/CD и автоматизация (15 вопросов)

### 26. Что такое CI/CD и какие принципы лежат в основе?
1. В чем разница между Continuous Integration и Continuous Deployment?
2. Что такое Continuous Delivery?
3. Какие преимущества дает CI/CD процесс?
4. Какие этапы обычно включает CI/CD pipeline?
5. Как измерить эффективность CI/CD процесса?

### 27. Объясните основы Jenkins и его архитектуру
1. Что такое Jenkins Master и Agent архитектура?
2. Как настроить distributed builds в Jenkins?
3. Что такое Jenkins Pipeline и чем отличается от Freestyle jobs?
4. Как управлять Jenkins конфигурацией как кодом?
5. Какие альтернативы Jenkins вы знаете?

### 28. Как создать эффективный CI pipeline?
1. Какие стадии должны быть в типичном CI pipeline?
2. Как организовать тестирование в CI pipeline?
3. Что такое fail-fast принцип?
4. Как обеспечить reproducible builds?
5. Как параллелизировать выполнение CI jobs?

### 29. Объясните стратегии branching для CI/CD
1. Как Git Flow влияет на CI/CD процесс?
2. Что такое trunk-based development?
3. Как организовать CI/CD для feature branches?
4. Что такое merge vs rebase стратегии?
5. Как handle merge conflicts в автоматическом режиме?

### 30. Как настроить автоматическое тестирование в CI/CD?
1. Какие типы тестов должны быть в CI pipeline?
2. Что такое test pyramid и как его применить?
3. Как организовать integration тесты в CI?
4. Что такое contract testing?
5. Как handle flaky tests в CI?

### 31. Объясните подходы к deployment automation
1. Какие стратегии deployment вы знаете?
2. Что такое immutable infrastructure?
3. Как реализовать zero-downtime deployment?
4. Что такое feature flags и как их использовать?
5. Как организовать rollback стратегию?

### 32. Что такое GitLab CI/CD и его особенности?
1. Как устроен .gitlab-ci.yml файл?
2. Что такое GitLab Runners?
3. Как организовать multi-stage pipeline в GitLab?
4. Что такое GitLab environments?
5. Как настроить auto-scaling runners?

### 33. Объясните GitHub Actions workflow
1. Как создать GitHub Actions workflow?
2. Что такое Actions marketplace?
3. Как организовать secrets management в GitHub Actions?
4. Что такое matrix builds?
5. Как создать собственный custom action?

### 34. Как интегрировать security testing в CI/CD?
1. Что такое DevSecOps подход?
2. Какие виды security тестов должны быть в pipeline?
3. Что такое SAST и DAST?
4. Как автоматизировать dependency vulnerability scanning?
5. Как handle security issues в CI/CD процессе?

### 35. Объясните artifact management в CI/CD
1. Что такое build artifacts и как их управлять?
2. Какие artifact repositories вы знаете?
3. Как версионировать artifacts?
4. Что такое semantic versioning?
5. Как организовать cleanup старых artifacts?

### 36. Как мониторить CI/CD pipeline?
1. Какие метрики важно отслеживать в CI/CD?
2. Что такое DORA metrics?
3. Как измерить deployment frequency?
4. Что такое lead time и как его оптимизировать?
5. Как handle pipeline failures и alerts?

### 37. Объясните Infrastructure as Code в контексте CI/CD
1. Как интегрировать Terraform в CI/CD pipeline?
2. Что такое plan/apply стратегия?
3. Как handle state management в automated deployments?
4. Что такое drift detection?
5. Как организовать multi-environment deployments?

### 38. Как организовать CD для микросервисов?
1. Какие challenges при deployment микросервисов?
2. Что такое service mesh и как он помогает в deployment?
3. Как организовать dependency management между сервисами?
4. Что такое distributed tracing в контексте deployment?
5. Как handle database migrations в микросервисной архитектуре?

### 39. Объясните подходы к configuration management в CI/CD
1. Как управлять конфигурацией для разных environments?
2. Что такое externalized configuration?
3. Как handle secrets в CI/CD pipeline?
4. Что такое configuration drift?
5. Как обеспечить config validation в pipeline?

### 40. Как оптимизировать производительность CI/CD pipeline?
1. Какие bottlenecks могут быть в CI/CD процессе?
2. Как использовать caching для ускорения builds?
3. Что такое incremental builds?
4. Как параллелизировать testing и deployment?
5. Как measure и improve pipeline performance?

---

## Блок 4: Инфраструктура как код (10 вопросов)

### 41. Что такое Infrastructure as Code и его преимущества?
1. Какие проблемы решает IaC подход?
2. В чем разница между declarative и imperative IaC?
3. Какие инструменты IaC вы знаете?
4. Что такое infrastructure drift?
5. Как обеспечить версионирование инфраструктуры?

### 42. Объясните основы Terraform
1. Что такое Terraform provider и resource?
2. Как работает Terraform state?
3. Что такое Terraform modules?
4. Объясните terraform plan и apply процесс
5. Как handle terraform state в команде?

### 43. Как организовать структуру Terraform проекта?
1. Как разделить код на environments?
2. Что такое workspace в Terraform?
3. Как организовать shared modules?
4. Как handle dependencies между resources?
5. Какие best practices для структуры Terraform кода?

### 44. Объясните Terraform state management
1. Где должен храниться Terraform state?
2. Что такое state locking?
3. Как handle state в CI/CD pipeline?
4. Что делать при corruption state файла?
5. Как migrate resources между state файлами?

### 45. Что такое Ansible и как он работает?
1. Чем Ansible отличается от других CM инструментов?
2. Что такое Ansible playbooks и roles?
3. Как работает Ansible inventory?
4. Что такое idempotency в Ansible?
5. Как handle secrets в Ansible?

### 46. Объясните CloudFormation (AWS)
1. Что такое CloudFormation template?
2. Как работают CloudFormation stacks?
3. Что такое nested stacks?
4. Как handle updates в CloudFormation?
5. Чем CloudFormation отличается от Terraform?

### 47. Как тестировать Infrastructure as Code?
1. Какие виды тестов нужны для IaC?
2. Что такое infrastructure testing tools?
3. Как организовать unit тесты для Terraform?
4. Что такое compliance testing?
5. Как интегрировать IaC тестирование в CI/CD?

### 48. Объясните подходы к secrets management в IaC
1. Как handle sensitive data в Terraform?
2. Что такое Vault и как его интегрировать?
3. Как использовать cloud-native secrets managers?
4. Что такое encryption at rest и in transit?
5. Как audit access к секретам?

### 49. Как организовать multi-environment infrastructure?
1. Как structure код для dev/staging/prod?
2. Что такое environment promotion strategy?
3. Как handle configuration differences между environments?
4. Что такое infrastructure pipeline?
5. Как ensure consistency между environments?

### 50. Какие best practices для Infrastructure as Code?
1. Как organize и structure IaC repositories?
2. Что такое code review процесс для infrastructure?
3. Как handle infrastructure documentation?
4. Какие naming conventions использовать?
5. Как ensure security в IaC коде?

---

## Блок 5: Мониторинг и логирование (10 вопросов)

### 51. Объясните принципы мониторинга современных приложений
1. Что такое observability и чем отличается от monitoring?
2. Какие три pillars of observability?
3. Что такое SLI, SLO и SLA?
4. Как выбрать правильные метрики для мониторинга?
5. Что такое monitoring pyramid?

### 52. Что такое Prometheus и как он работает?
1. Объясните pull-based модель Prometheus
2. Что такое PromQL?
3. Как работает service discovery в Prometheus?
4. Что такое alerting rules?
5. Как обеспечить high availability Prometheus?

### 53. Объясните Grafana и visualization
1. Как создавать dashboard в Grafana?
2. Что такое templating в Grafana?
3. Как настроить alerting в Grafana?
4. Что такое Grafana provisioning?
5. Как организовать team collaboration в Grafana?

### 54. Что такое ELK Stack и его компоненты?
1. Объясните роль каждого компонента ELK
2. Как работает log parsing в Logstash?
3. Что такое Elasticsearch indices и sharding?
4. Как создавать visualizations в Kibana?
5. Какие альтернативы ELK stack вы знаете?

### 55. Как организовать централизованное логирование?
1. Какие подходы к log collection существуют?
2. Что такое structured logging?
3. Как handle log rotation и retention?
4. Что такое log levels и как их использовать?
5. Как ensure log security и privacy?

### 56. Объясните application performance monitoring (APM)
1. Что такое distributed tracing?
2. Как работают APM agents?
3. Что такое synthetic monitoring?
4. Как monitor user experience?
5. Какие APM инструменты вы знаете?

### 57. Как мониторить инфраструктуру?
1. Какие infrastructure метрики важно отслеживать?
2. Что такое infrastructure monitoring tools?
3. Как monitor cloud resources?
4. Что такое capacity planning?
5. Как handle infrastructure alerts?

### 58. Объясните подходы к alerting
1. Как создавать effective alerts?
2. Что такое alert fatigue и как его избежать?
3. Как организовать escalation policies?
4. Что такое on-call rotations?
5. Как measure alerting effectiveness?

### 59. Что такое chaos engineering и мониторинг?
1. Как мониторинг помогает в chaos engineering?
2. Что такое failure detection metrics?
3. Как monitor system resilience?
4. Что такое blast radius monitoring?
5. Как use monitoring для post-incident analysis?

### 60. Как оптимизировать monitoring систему?
1. Какие costs associated с monitoring?
2. Как reduce monitoring overhead?
3. Что такое sampling strategies?
4. Как handle high-cardinality metrics?
5. Как ensure monitoring system scalability?

---

## Блок 6: Облачные платформы и сервисы (10 вопросов)

### 61. Объясните основные облачные модели (IaaS, PaaS, SaaS)
1. Какие преимущества и недостатки каждой модели?
2. Что такое serverless computing?
3. Как выбрать подходящую облачную модель?
4. Что такое multi-cloud и hybrid cloud стратегии?
5. Какие cost optimization стратегии для облака?

### 62. Основы Amazon Web Services (AWS)
1. Какие core services AWS вы знаете?
2. Что такое AWS regions и availability zones?
3. Как работает AWS IAM?
4. Что такое AWS VPC?
5. Как организовать AWS account structure?

### 63. Что такое AWS ECS и EKS?
1. В чем разница между ECS и EKS?
2. Как deploy контейнеры в AWS?
3. Что такое Fargate?
4. Как настроить load balancing для контейнеров?
5. Как monitor контейнеры в AWS?

### 64. Объясните AWS Lambda и serverless архитектуру
1. Когда использовать Lambda functions?
2. Что такое cold start problem?
3. Как organize serverless applications?
4. Что такое event-driven architecture?
5. Как handle state в serverless приложениях?

### 65. Основы Microsoft Azure
1. Какие key services Azure вы знаете?
2. Что такое Azure Resource Groups?
3. Как работает Azure Active Directory?
4. Что такое Azure DevOps?
5. Как интегрировать on-premises с Azure?

### 66. Что такое Google Cloud Platform (GCP)?
1. Какие unique features GCP?
2. Что такое Google Kubernetes Engine?
3. Как работает Google Cloud Functions?
4. Что такое BigQuery и когда его использовать?
5. Как organize projects в GCP?

### 67. Как обеспечить безопасность в облаке?
1. Что такое shared responsibility model?
2. Как implement identity and access management?
3. Что такое cloud security posture management?
4. Как encrypt data in cloud?
5. Какие compliance требования в облаке?

### 68. Объясните подходы к cloud networking
1. Как organize virtual networks в облаке?
2. Что такое cloud load balancers?
3. Как implement hybrid connectivity?
4. Что такое CDN и как его использовать?
5. Как handle DNS в cloud environments?

### 69. Как управлять costs в облаке?
1. Какие cost optimization стратегии?
2. Что такое reserved instances?
3. Как implement cost monitoring?
4. Что такое cloud financial management?
5. Как handle cost allocation в multi-team environment?

### 70. Объясните cloud migration strategies
1. Какие подходы к cloud migration существуют?
2. Что такое lift-and-shift migration?
3. Как assess applications для migration?
4. Что такое cloud-native refactoring?
5. Как handle data migration к облаку?

---

## Блок 7: Безопасность и DevSecOps (10 вопросов)

### 71. Что такое DevSecOps и его принципы?
1. Как интегрировать security в DevOps процесс?
2. Что такое shift-left security?
3. Какие security practices должны быть automated?
4. Как balance security и delivery speed?
5. Что такое security as code?

### 72. Объясните подходы к vulnerability management
1. Как scan code на уязвимости?
2. Что такое dependency scanning?
3. Как handle security advisories?
4. Что такое penetration testing в DevOps?
5. Как prioritize security vulnerabilities?

### 73. Как обеспечить безопасность CI/CD pipeline?
1. Какие security risks в CI/CD процессе?
2. Как secure build environment?
3. Что такое supply chain security?
4. Как implement secrets scanning?
5. Как secure deployment process?

### 74. Объясните container security
1. Какие security risks у контейнеров?
2. Как scan container images на уязвимости?
3. Что такое runtime security для контейнеров?
4. Как implement least privilege для контейнеров?
5. Что такое container isolation?

### 75. Что такое identity and access management (IAM)?
1. Как implement principle of least privilege?
2. Что такое role-based access control?
3. Как handle service-to-service authentication?
4. Что такое zero-trust architecture?
5. Как audit access permissions?

### 76. Объясните secrets management
1. Как store и rotate secrets?
2. Что такое secrets lifecycle management?
3. Как integrate secrets managers с applications?
4. Что такое secret zero problem?
5. Как audit secrets access?

### 77. Как implement security monitoring?
1. Что такое security information and event management (SIEM)?
2. Как detect security incidents?
3. Что такое threat hunting?
4. Как implement incident response?
5. Что такое security orchestration?

### 78. Объясните compliance и governance
1. Какие compliance frameworks важны для DevOps?
2. Как implement policy as code?
3. Что такое continuous compliance?
4. Как handle audit requirements?
5. Что такое risk management в DevOps?

### 79. Как secure микросервисы архитектуру?
1. Какие unique security challenges микросервисов?
2. Что такое service mesh security?
3. Как implement API security?
4. Что такое mutual TLS?
5. Как handle distributed authentication?

### 80. Объясните incident response в DevOps среде
1. Как organize incident response team?
2. Что такое runbooks и playbooks?
3. Как implement blameless post-mortems?
4. Что такое chaos engineering для security?
5. Как learn от security incidents?

---

## Критерии оценки

### Уровни компетенции:

**Junior (1-2 года опыта):**
- Понимание базовых концепций
- Знание основных инструментов
- Способность работать под руководством

**Middle (2-5 лет опыта):**
- Глубокое понимание инструментов
- Способность проектировать решения
- Опыт troubleshooting

**Senior (5+ лет опыта):**
- Архитектурное мышление
- Способность обучать других
- Понимание business impact

### Система оценки:
- **Отлично (4-5 баллов):** Полный и точный ответ с примерами
- **Хорошо (3-4 балла):** Правильный ответ с небольшими недочетами
- **Удовлетворительно (2-3 балла):** Базовое понимание темы
- **Неудовлетворительно (0-2 балла):** Отсутствие понимания

### Рекомендации по проведению:
1. Начинайте с простых вопросов для создания комфортной атмосферы
2. Углубляйтесь в зависимости от ответов кандидата
3. Просите привести практические примеры
4. Обращайте внимание на способность объяснять сложные концепции
5. Оценивайте не только знания, но и подход к решению проблем