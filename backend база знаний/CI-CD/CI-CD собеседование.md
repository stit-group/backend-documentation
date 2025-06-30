# CI/CD Interview Template для Backend Разработчика

## Инструкция по использованию
- **Целевая аудитория**: Middle/Senior Backend разработчики
- **Время интервью**: 2-3 часа (рекомендуется разбить на несколько сессий)
- **Формат**: Каждый основной вопрос сопровождается 5 углубляющими подвопросами
- **Оценка**: Фиксируйте ответы и уровень понимания (базовый/средний/продвинутый)

---

## 1. Основы CI/CD (10 вопросов)

### 1.1 Что такое CI/CD и в чем разница между Continuous Integration, Continuous Delivery и Continuous Deployment?
**Углубляющие вопросы:**
1. Какие проблемы решает CI/CD в разработке ПО?
2. Приведите пример ситуации, когда CD (Delivery) предпочтительнее CD (Deployment)
3. Какие метрики используются для оценки эффективности CI/CD процессов?
4. Как CI/CD влияет на культуру разработки в команде?
5. Какие антипаттерны CI/CD вы знаете и как их избежать?

### 1.2 Объясните концепцию "Pipeline as Code". Какие преимущества она дает?
**Углубляющие вопросы:**
1. Как версионировать CI/CD пайплайны и почему это важно?
2. Какие подходы к структурированию pipeline конфигураций вы знаете?
3. Как обеспечить переносимость пайплайнов между разными средами?
4. Какие инструменты для валидации pipeline конфигураций вы использовали?
5. Как организовать переиспользование общих частей пайплайнов?

### 1.3 Что такое "Shift Left" в контексте CI/CD?
**Углубляющие вопросы:**
1. Какие виды тестирования можно сдвинуть "влево" в pipeline?
2. Как интегрировать статический анализ кода в ранние стадии разработки?
3. Какие инструменты для pre-commit hooks вы знаете?
4. Как балансировать скорость обратной связи и глубину проверок?
5. Приведите пример метрики, которая показывает успешность "Shift Left" подхода

### 1.4 Объясните концепцию Infrastructure as Code (IaC) в контексте CI/CD
**Углубляющие вопросы:**
1. Какие инструменты IaC вы использовали и в чем их различия?
2. Как организовать testing инфраструктурного кода?
3. Какие паттерны для управления состоянием инфраструктуры вы знаете?
4. Как решать проблему drift detection в IaC?
5. Как интегрировать IaC пайплайны с application deployment пайплайнами?

### 1.5 Что такое GitOps и чем он отличается от традиционных подходов к CD?
**Углубляющие вопросы:**
1. Какие компоненты необходимы для реализации GitOps?
2. Как организовать multi-environment GitOps workflow?
3. Какие проблемы безопасности возникают при GitOps и как их решать?
4. Как реализовать rollback в GitOps модели?
5. Сравните GitOps с push-based deployment подходами

### 1.6 Объясните принцип "Build once, deploy anywhere"
**Углубляющие вопросы:**
1. Как этот принцип влияет на структуру артефактов сборки?
2. Какие проблемы возникают при нарушении этого принципа?
3. Как организовать environment-specific конфигурацию при соблюдении этого принципа?
4. Какие инструменты помогают реализовать этот подход?
5. Как версионировать и отслеживать артефакты через разные среды?

### 1.7 Что такое Feature Flags и как они интегрируются с CI/CD?
**Углубляющие вопросы:**
1. Какие типы feature flags существуют и когда каждый использовать?
2. Как организовать lifecycle management для feature flags?
3. Какие метрики собирать для feature flags?
4. Как feature flags влияют на стратегию тестирования?
5. Какие риски связаны с накоплением технического долга от feature flags?

### 1.8 Объясните концепцию Immutable Infrastructure
**Углубляющие вопросы:**
1. Какие преимущества дает immutable подход для CI/CD?
2. Как реализовать database migrations в immutable инфраструктуре?
3. Какие проблемы с persistent data возникают и как их решать?
4. Сравните mutable и immutable подходы с точки зрения безопасности
5. Как организовать debugging в immutable инфраструктуре?

### 1.9 Что такое Deployment Pipeline и из каких стадий он должен состоять?
**Углубляющие вопросы:**
1. Как определить оптимальное количество стадий в pipeline?
2. Какие критерии использовать для promotion между стадиями?
3. Как организовать параллельное выполнение стадий?
4. Какие gates и approvals встраивать в pipeline?
5. Как обеспечить observability всего pipeline?

### 1.10 Объясните концепцию "Everything as Code"
**Углубляющие вопросы:**
1. Что включает в себя "Everything as Code" помимо инфраструктуры?
2. Как версионировать и управлять изменениями в configuration as code?
3. Какие вызовы возникают при применении этого подхода к compliance и security?
4. Как организовать testing для различных типов "code"?
5. Какие инструменты помогают в реализации этой концепции?

---

## 2. Системы контроля версий и Git workflow (10 вопросов)

### 2.1 Объясните различия между Git Flow, GitHub Flow и GitLab Flow
**Углубляющие вопросы:**
1. В каких ситуациях каждый workflow наиболее подходящий?
2. Как каждый workflow влияет на CI/CD процессы?
3. Какие проблемы могут возникнуть при неправильном выборе workflow?
4. Как адаптировать workflow под специфику команды и продукта?
5. Какие инструменты автоматизации каждого workflow вы знаете?

### 2.2 Как организовать branching strategy для микросервисной архитектуры?
**Углубляющие вопросы:**
1. Как синхронизировать релизы зависимых сервисов?
2. Какие подходы к versioning API между сервисами в CI/CD?
3. Как организовать integration testing в multi-repo окружении?
4. Какие инструменты для управления mono-repo vs multi-repo вы знаете?
5. Как решать конфликты версий dependencies между сервисами?

### 2.3 Что такое Semantic Versioning и как его автоматизировать в CI/CD?
**Углубляющие вопросы:**
1. Как определить тип изменения (major/minor/patch) автоматически?
2. Какие conventional commit patterns вы используете?
3. Как организовать автоматическую генерацию changelog?
4. Как semantic versioning влияет на dependency management?
5. Какие инструменты для автоматизации versioning вы знаете?

### 2.4 Объясните стратегии merge vs rebase vs squash
**Углубляющие вопросы:**
1. Как каждая стратегия влияет на историю коммитов и debugging?
2. В каких случаях предпочтительна каждая стратегия?
3. Как настроить автоматические правила для каждой стратегии?
4. Какие проблемы возникают при неконсистентном использовании стратегий?
5. Как стратегии влияют на возможность rollback и hotfix?

### 2.5 Как организовать code review process в контексте CI/CD?
**Углубляющие вопросы:**
1. Какие автоматические проверки должны выполняться до code review?
2. Как интегрировать результаты CI в процесс review?
3. Какие метрики code review влияют на качество CI/CD?
4. Как организовать review для infrastructure и pipeline кода?
5. Какие инструменты для автоматизации code review процесса вы знаете?

### 2.6 Что такое Git hooks и как их использовать в CI/CD?
**Углубляющие вопросы:**
1. Какие типы hooks существуют и для чего каждый используется?
2. Как организовать централизованное управление hooks в команде?
3. Какие проверки стоит выносить в pre-commit vs CI pipeline?
4. Как обеспечить безопасность и производительность hooks?
5. Как тестировать и версионировать Git hooks?

### 2.7 Объясните концепцию trunk-based development
**Углубляющие вопросы:**
1. Какие практики необходимы для успешного trunk-based development?
2. Как организовать feature development без feature branches?
3. Какие инструменты CI/CD лучше всего подходят для этого подхода?
4. Как решать проблемы с нестабильным main branch?
5. Какие метрики использовать для оценки эффективности подхода?

### 2.8 Как организовать управление secrets в Git и CI/CD?
**Углубляющие вопросы:**
1. Какие типы secrets требуют разных подходов к управлению?
2. Как организовать rotation secrets в автоматизированном режиме?
3. Какие инструменты для secret management вы использовали?
4. Как обеспечить audit trail для использования secrets?
5. Как организовать least privilege access для secrets в CI/CD?

### 2.9 Что такое monorepo и какие особенности CI/CD для него?
**Углубляющие вопросы:**
1. Как организовать selective building в monorepo?
2. Какие стратегии dependency management в monorepo?
3. Как масштабировать CI/CD для больших monorepo?
4. Какие инструменты для monorepo management вы знаете?
5. Как организовать release management в monorepo?

### 2.10 Объясните стратегии для hotfix и emergency deployments
**Углубляющие вопросы:**
1. Как организовать expedited pipeline для критических исправлений?
2. Какие автоматические проверки можно пропустить для hotfix?
3. Как обеспечить traceability hotfix через все environments?
4. Какие post-deployment процедуры необходимы после hotfix?
5. Как предотвратить накопление технического долга от hotfix?

---

## 3. Автоматизация сборки и тестирования (15 вопросов)

### 3.1 Объясните пирамиду тестирования в контексте CI/CD
**Углубляющие вопросы:**
1. Как распределить время выполнения между уровнями тестирования?
2. Какие типы тестов можно распараллелить и как?
3. Как организовать test data management для разных уровней?
4. Какие метрики использовать для оценки эффективности каждого уровня?
5. Как адаптировать пирамиду тестирования для микросервисов?

### 3.2 Что такое Contract Testing и как его интегрировать в CI/CD?
**Углубляющие вопросы:**
1. Какие инструменты для contract testing вы знаете?
2. Как организовать consumer-driven contract testing?
3. Как решать breaking changes в contracts?
4. Как интегрировать contract testing с API versioning?
5. Какие метрики собирать для contract testing?

### 3.3 Как организовать parallel execution тестов в CI/CD?
**Углубляющие вопросы:**
1. Какие стратегии для splitting test suites вы знаете?
2. Как решать проблемы с flaky tests в параллельном окружении?
3. Как организовать test isolation при параллельном выполнении?
4. Какие инструменты для test orchestration вы использовали?
5. Как оптимизировать resource utilization при параллельном тестировании?

### 3.4 Объясните концепцию Test Doubles (mocks, stubs, fakes)
**Углубляющие вопросы:**
1. Когда использовать каждый тип test doubles?
2. Как организовать shared test doubles между командами?
3. Какие проблемы возникают при чрезмерном использовании mocks?
4. Как поддерживать актуальность test doubles?
5. Как интегрировать test doubles с contract testing?

### 3.5 Что такое Mutation Testing и как его применять?
**Углубляющие вопросы:**
1. Какие инструменты для mutation testing вы знаете?
2. Как интегрировать mutation testing в CI pipeline?
3. Какие метрики mutation testing наиболее важны?
4. Как балансировать время выполнения и coverage mutation testing?
5. В каких случаях mutation testing не подходит?

### 3.6 Как организовать Performance Testing в CI/CD?
**Углубляющие вопросы:**
1. Какие типы performance тестов можно автоматизировать?
2. Как создать stable test environment для performance testing?
3. Какие метрики собирать и как их интерпретировать?
4. Как организовать performance budgets и alerts?
5. Какие инструменты для performance testing в CI/CD вы использовали?

### 3.7 Объясните подходы к Database Testing в CI/CD
**Углубляющие вопросы:**
1. Как организовать test data provisioning для database tests?
2. Какие стратегии для database schema testing существуют?
3. Как тестировать database migrations в CI/CD?
4. Как организовать performance testing для database queries?
5. Какие инструменты для database testing automation вы знаете?

### 3.8 Что такое Shift-Right Testing и как его реализовать?
**Углубляющие вопросы:**
1. Какие виды testing можно сдвинуть в production?
2. Как организовать monitoring-based testing?
3. Какие инструменты для chaos engineering вы знаете?
4. Как организовать canary testing и A/B testing?
5. Как собирать и анализировать feedback от production testing?

### 3.9 Как организовать Security Testing в CI/CD pipeline?
**Углубляющие вопросы:**
1. Какие типы security тестов можно автоматизировать?
2. Как интегрировать SAST и DAST в pipeline?
3. Как организовать dependency scanning и vulnerability management?
4. Какие compliance требования влияют на security testing?
5. Как балансировать security и скорость delivery?

### 3.10 Объясните концепцию Test Environment Management
**Углубляющие вопросы:**
1. Как организовать on-demand test environments?
2. Какие стратегии для test data management существуют?
3. Как организовать environment parity между test и production?
4. Какие инструменты для test environment automation вы использовали?
5. Как оптимизировать стоимость test environments?

### 3.11 Что такое Property-Based Testing и где его применять?
**Углубляющие вопросы:**
1. Какие инструменты для property-based testing вы знаете?
2. Как генерировать эффективные test cases автоматически?
3. В каких областях property-based testing наиболее эффективен?
4. Как интегрировать property-based testing в CI pipeline?
5. Как интерпретировать результаты property-based тестов?

### 3.12 Как организовать End-to-End Testing в микросервисной архитектуре?
**Углубляющие вопросы:**
1. Какие стратегии для E2E testing в distributed systems?
2. Как организовать test service discovery и configuration?
3. Как решать проблемы с test data consistency в E2E тестах?
4. Какие инструменты для E2E testing микросервисов вы использовали?
5. Как оптимизировать время выполнения E2E тестов?

### 3.13 Объясните концепцию Test Impact Analysis
**Углубляющие вопросы:**
1. Как определить какие тесты запускать при конкретных изменениях?
2. Какие инструменты для test impact analysis существуют?
3. Как построить dependency graph для эффективного анализа?
4. Какие метрики использовать для оценки эффективности подхода?
5. Как обеспечить safety при использовании test selection?

### 3.14 Как организовать Visual Testing в CI/CD?
**Углубляющие вопросы:**
1. Какие инструменты для visual regression testing вы знаете?
2. Как решать проблемы с cross-browser и cross-platform различиями?
3. Как организовать approval workflow для visual changes?
4. Какие стратегии для managing visual test assets?
5. Как интегрировать visual testing с design systems?

### 3.15 Что такое Synthetic Monitoring и как его интегрировать с CI/CD?
**Углубляющие вопросы:**
1. Какие типы synthetic tests можно автоматизировать?
2. Как организовать continuous synthetic monitoring?
3. Какие метрики собирать от synthetic monitoring?
4. Как интегрировать результаты synthetic monitoring с deployment decisions?
5. Какие инструменты для synthetic monitoring вы использовали?

---

## 4. Контейнеризация и оркестрация (15 вопросов)

### 4.1 Объясните best practices для написания Dockerfile в контексте CI/CD
**Углубляющие вопросы:**
1. Как оптимизировать layer caching для быстрой сборки?
2. Какие security практики применять при создании Docker images?
3. Как организовать multi-stage builds для разных environments?
4. Какие инструменты для Dockerfile linting и security scanning вы знаете?
5. Как управлять base image updates в CI/CD pipeline?

### 4.2 Что такое distroless images и когда их использовать?
**Углубляющие вопросы:**
1. Какие преимущества и недостатки distroless images?
2. Как отлаживать приложения в distroless containers?
3. Какие альтернативы distroless images существуют?
4. Как организовать security scanning для distroless images?
5. Как migration на distroless влияет на CI/CD процессы?

### 4.3 Объясните стратегии Container Image Versioning и Tagging
**Углубляющие вопросы:**
1. Какие naming conventions для tags рекомендуете?
2. Как организовать image promotion между environments?
3. Какие стратегии для cleanup старых images?
4. Как решать проблемы с image immutability?
5. Как интегрировать image scanning в versioning workflow?

### 4.4 Как организовать Container Registry management?
**Углубляющие вопросы:**
1. Какие стратегии для multi-region registry replication?
2. Как организовать access control и security для registry?
3. Какие подходы к registry high availability?
4. Как организовать vulnerability scanning в registry?
5. Какие метрики собирать от container registry?

### 4.5 Объясните Kubernetes Deployment strategies
**Углубляющие вопросы:**
1. Сравните Rolling, Blue-Green, и Canary deployments в K8s
2. Как реализовать custom deployment strategies?
3. Как организовать automated rollback при проблемах?
4. Какие метрики использовать для оценки успешности deployment?
5. Как интегрировать deployment strategies с monitoring?

### 4.6 Что такое GitOps в контексте Kubernetes?
**Углубляющие вопросы:**
1. Какие инструменты для GitOps в K8s вы использовали?
2. Как организовать multi-cluster GitOps?
3. Какие проблемы безопасности в GitOps и как их решать?
4. Как организовать secret management в GitOps workflow?
5. Как debugging GitOps deployments?

### 4.7 Объясните концепцию Helm Charts и их роль в CI/CD
**Углубляющие вопросы:**
1. Как организовать templating и configuration management в Helm?
2. Какие best practices для Helm chart testing?
3. Как версионировать и release Helm charts?
4. Какие альтернативы Helm для K8s package management?
5. Как интегрировать Helm с GitOps workflows?

### 4.8 Как организовать Service Mesh в CI/CD pipeline?
**Углубляющие вопросы:**
1. Какие service mesh решения вы знаете и их различия?
2. Как service mesh влияет на deployment strategies?
3. Как организовать traffic routing и canary deployments через service mesh?
4. Какие observability возможности дает service mesh?
5. Как debugging network issues в service mesh?

### 4.9 Что такое Kubernetes Operators и как их использовать в CI/CD?
**Углубляющие вопросы:**
1. Как создать custom operator для своего приложения?
2. Какие existing operators полезны для CI/CD workflows?
3. Как тестировать Kubernetes operators?
4. Как организовать lifecycle management для operators?
5. Какие метрики собирать от operators?

### 4.10 Объясните подходы к Configuration Management в Kubernetes
**Углубляющие вопросы:**
1. Сравните ConfigMaps, Secrets, и external config management
2. Как организовать configuration updates без downtime?
3. Какие инструменты для configuration templating в K8s?
4. Как обеспечить configuration validation?
5. Как организовать environment-specific configurations?

### 4.11 Как организовать Resource Management и Scaling в CI/CD?
**Углубляющие вопросы:**
1. Как настроить HPA и VPA для приложений?
2. Какие стратегии для resource quotas и limits?
3. Как организовать cluster autoscaling?
4. Какие метрики использовать для scaling decisions?
5. Как тестировать scaling behavior в CI/CD?

### 4.12 Что такое Pod Security Standards и как их применять?
**Углубляющие вопросы:**
1. Какие уровни Pod Security Standards существуют?
2. Как интегрировать security policies в deployment pipeline?
3. Какие инструменты для policy validation вы знаете?
4. Как организовать exceptions и overrides для security policies?
5. Как monitoring и alerting на нарушения security policies?

### 4.13 Объясните Network Policies в Kubernetes
**Углубляющие вопросы:**
1. Как создать effective network segmentation?
2. Какие инструменты для network policy testing?
3. Как debugging network connectivity issues?
4. Как организовать network policies для multi-tenant clusters?
5. Как network policies интегрируются с service mesh?

### 4.14 Как организовать Persistent Storage в CI/CD workflows?
**Углубляющие вопросы:**
1. Какие storage classes и provisioners использовать?
2. Как организовать backup и restore для persistent data?
3. Как тестировать storage-dependent applications?
4. Какие стратегии для data migration между environments?
5. Как monitoring storage performance и availability?

### 4.15 Что такое Custom Resource Definitions (CRDs) и их применение?
**Углубляющие вопросы:**
1. Как создать и версионировать CRDs?
2. Какие validation strategies для custom resources?
3. Как организовать migration между версиями CRDs?
4. Как debugging issues с custom resources?
5. Какие best practices для CRD naming и structure?

---

## 5. Deployment стратегии (10 вопросов)

### 5.1 Сравните Blue-Green, Canary, и Rolling deployments
**Углубляющие вопросы:**
1. Какие критерии использовать для выбора стратегии deployment?
2. Как организовать automated decision making для rollback?
3. Какие метрики собирать для каждой стратегии?
4. Как каждая стратегия влияет на database migrations?
5. Какие инструменты для реализации каждой стратегии вы знаете?

### 5.2 Объясните концепцию Feature Toggles в deployment process
**Углубляющие вопросы:**
1. Как организовать dynamic configuration для feature toggles?
2. Какие типы feature toggles существуют?
3. Как testing strategy меняется с feature toggles?
4. Как организовать monitoring для feature toggle usage?
5. Какие риски связаны с feature toggles и как их минимизировать?

### 5.3 Что такое Database Migration strategies в CI/CD?
**Углубляющие вопросы:**
1. Как организовать backward-compatible database changes?
2. Какие подходы к zero-downtime database migrations?
3. Как тестировать database migrations в CI pipeline?
4. Как организовать rollback для database changes?
5. Какие инструменты для database migration automation вы знаете?

### 5.4 Объясните концепцию Immutable Deployments
**Углубляющие вопросы:**
1. Как реализовать immutable infrastructure для deployments?
2. Какие преимущества immutable deployments для rollback?
3. Как организовать configuration management в immutable deployments?
4. Какие challenges с persistent data в immutable deployments?
5. Как debugging в immutable deployment environments?

### 5.5 Как организовать Multi-Region deployments?
**Углубляющие вопросы:**
1. Какие стратегии для cross-region deployment orchestration?
2. Как организовать data consistency между регионами?
3. Какие подходы к traffic routing между регионами?
4. Как организовать disaster recovery для multi-region deployments?
5. Какие метрики собирать для multi-region deployments?

### 5.6 Что такое Progressive Delivery?
**Углубляющие вопросы:**
1. Как автоматизировать progression rules для delivery?
2. Какие инструменты для progressive delivery вы знаете?
3. Как интегрировать business metrics в progression decisions?
4. Как организовать automated rollback в progressive delivery?
5. Какие patterns для progressive delivery в микросервисах?

### 5.7 Объясните Deployment Pipelines для микросервисов
**Углубляющие вопросы:**
1. Как организовать service dependency management в deployment?
2. Какие стратегии для coordinated vs independent deployments?
3. Как тестировать service interactions при deployment?
4. Как организовать deployment orchestration для сложных dependencies?
5. Какие инструменты для microservices deployment orchestration?

### 5.8 Как организовать Environment Promotion strategies?
**Углубляющие вопросы:**
1. Какие validation gates между environments?
2. Как организовать automated promotion criteria?
3. Как обеспечить environment parity?
4. Какие метрики использовать для promotion decisions?
5. Как организовать manual overrides в promotion process?

### 5.9 Что такое Deployment Windows и как их оптимизировать?
**Углубляющие вопросы:**
1. Как определить optimal deployment windows?
2. Какие стратегии для reducing deployment time?
3. Как организовать emergency deployments outside windows?
4. Как balance deployment frequency и stability?
5. Какие инструменты для deployment scheduling вы знаете?

### 5.10 Объясните Rollback strategies и их автоматизацию
**Углубляющие вопросы:**
1. Какие типы rollback существуют и когда каждый использовать?
2. Как организовать automated rollback triggers?
3. Какие данные сохранять для effective rollback?
4. Как тестировать rollback procedures?
5. Как communicate rollback events в команде?

---

## 6. Мониторинг и логирование (10 вопросов)

### 6.1 Объясните Three Pillars of Observability в контексте CI/CD
**Углубляющие вопросы:**
1. Как интегрировать metrics, logs, и traces в CI/CD pipeline?
2. Какие корреляции между тремя pillars наиболее важны?
3. Как организовать unified observability strategy?
4. Какие инструменты для каждого pillar вы использовали?
5. Как observability влияет на deployment decisions?

### 6.2 Что такое SLI, SLO, и SLA и как их применять в CI/CD?
**Углубляющие вопросы:**
1. Как определить meaningful SLIs для CI/CD процессов?
2. Как автоматизировать SLO monitoring и alerting?
3. Как SLOs влияют на deployment strategies?
4. Какие error budgets использовать для CI/CD?
5. Как reporting SLA compliance автоматически?

### 6.3 Как организовать Distributed Tracing для микросервисов?
**Углубляющие вопросы:**
1. Какие tracing frameworks и protocols вы знаете?
2. Как организовать trace sampling strategies?
3. Как correlate traces с deployment events?
4. Какие performance implications от distributed tracing?
5. Как использовать tracing для debugging deployment issues?

### 6.4 Объясните Log Aggregation и Analysis strategies
**Углубляющие вопросы:**
1. Какие подходы к structured logging в CI/CD?
2. Как организовать log retention policies?
3. Какие security considerations для log management?
4. Как automated log analysis для detecting issues?
5. Какие инструменты для log aggregation вы использовали?

### 6.5 Что такое Application Performance Monitoring (APM)?
**Углубляющие вопросы:**
1. Как интегрировать APM в deployment pipeline?
2. Какие key performance indicators отслеживать?
3. Как APM data влияет на rollback decisions?
4. Какие APM инструменты для разных языков программирования?
5. Как correlate APM data с infrastructure metrics?

### 6.6 Как организовать Infrastructure Monitoring в CI/CD?
**Углубляющие вопросы:**
1. Какие infrastructure metrics критичны для CI/CD?
2. Как monitoring container и orchestration platforms?
3. Какие alerting strategies для infrastructure events?
4. Как automated infrastructure scaling на основе metrics?
5. Какие инструменты для infrastructure monitoring вы знаете?

### 6.7 Объясните концепцию Synthetic Monitoring
**Углубляющие вопросы:**
1. Какие типы synthetic tests создавать?
2. Как интегрировать synthetic monitoring с deployment pipeline?
3. Какие geographic distribution strategies для synthetic monitoring?
4. Как correlate synthetic results с real user experience?
5. Какие инструменты для synthetic monitoring вы использовали?

### 6.8 Что такое Chaos Engineering и как его применять?
**Углубляющие вопросы:**
1. Как интегрировать chaos experiments в CI/CD?
2. Какие типы failure injection полезны?
3. Как automated chaos testing в staging environments?
4. Какие safety mechanisms для chaos experiments?
5. Как measure system resilience через chaos engineering?

### 6.9 Как организовать Security Monitoring в CI/CD?
**Углубляющие вопросы:**
1. Какие security events отслеживать в CI/CD pipeline?
2. Как интегрировать SIEM с CI/CD processes?
3. Какие automated responses на security incidents?
4. Как monitoring compliance violations?
5. Какие инструменты для security monitoring вы знаете?

### 6.10 Объясните Alerting strategies и Incident Response
**Углубляющие вопросы:**
1. Как создать effective alerting rules без alert fatigue?
2. Какие escalation policies для разных типов alerts?
3. Как automated incident response procedures?
4. Как correlate alerts с deployment events?
5. Как measure и improve incident response times?

---

## 7. Безопасность в CI/CD (10 вопросов)

### 7.1 Объясните концепцию DevSecOps и Shift-Left Security
**Углубляющие вопросы:**
1. Как интегрировать security practices в early stages разработки?
2. Какие automated security checks включать в CI pipeline?
3. Как balance security и developer productivity?
4. Какие культурные изменения необходимы для DevSecOps?
5. Какие метрики использовать для measuring security posture?

### 7.2 Что такое SAST, DAST, и IAST и когда каждый использовать?
**Углубляющие вопросы:**
1. Как интегрировать каждый тип scanning в CI/CD pipeline?
2. Какие false positive strategies для каждого типа?
3. Как prioritize security findings от разных scanners?
4. Какие инструменты для каждого типа scanning вы знаете?
5. Как automated remediation для security vulnerabilities?

### 7.3 Как организовать Secret Management в CI/CD?
**Углубляющие вопросы:**
1. Какие типы secrets требуют разного management?
2. Как automated secret rotation в CI/CD workflows?
3. Какие access control patterns для secrets?
4. Как audit secret usage и detect misuse?
5. Какие инструменты для secret management вы использовали?

### 7.4 Объясните Container Security в CI/CD pipeline
**Углубляющие вопросы:**
1. Как организовать image vulnerability scanning?
2. Какие runtime security controls для containers?
3. Как security benchmarks для container configurations?
4. Какие network security policies для containers?
5. Как detect и respond на container security incidents?

### 7.5 Что такое Supply Chain Security и как его обеспечить?
**Углубляющие вопросы:**
1. Как verify integrity CI/CD pipeline components?
2. Какие dependency scanning strategies использовать?
3. Как organization software bill of materials (SBOM)?
4. Какие signing и verification strategies для artifacts?
5. Как detect и mitigate supply chain attacks?

### 7.6 Как организовать Identity and Access Management (IAM) в CI/CD?
**Углубляющие вопросы:**
1. Какие принципы least privilege в CI/CD workflows?
2. Как automated credential lifecycle management?
3. Какие authentication strategies для different CI/CD components?
4. Как audit access patterns и detect anomalies?
5. Как integrate with external identity providers?

### 7.7 Объясните Compliance as Code концепцию
**Углубляющие вопросы:**
1. Как automated compliance checking в CI/CD?
2. Какие compliance frameworks можно кодифицировать?
3. Как generate compliance reports автоматически?
4. Какие tools для policy as code вы знаете?
5. Как maintain compliance во время rapid deployments?

### 7.8 Что такое Infrastructure Security Scanning?
**Углубляющие вопросы:**
1. Как scan Infrastructure as Code для security issues?
2. Какие runtime infrastructure security controls?
3. Как cloud security posture management в CI/CD?
4. Какие network security scanning strategies?
5. Как automated remediation для infrastructure security issues?

### 7.9 Как организовать Secure Artifact Management?
**Углубляющие вопросы:**
1. Как ensure artifact integrity через pipeline?
2. Какие signing strategies для different artifact types?
3. Как организовать artifact vulnerability tracking?
4. Какие access controls для artifact repositories?
5. Как artifact provenance tracking и auditing?

### 7.10 Объясните Security Incident Response в CI/CD context
**Углубляющие вопросы:**
1. Как detect security incidents в CI/CD pipeline?
2. Какие automated response procedures для security events?
3. Как coordinate incident response с deployment activities?
4. Какие forensic capabilities для CI/CD security incidents?
5. Как post-incident improvement processes для security?

---

## Заключение и оценка

### Критерии оценки ответов:

**Базовый уровень (1-2 балла):**
- Понимает основные концепции
- Может объяснить базовую функциональность
- Знает популярные инструменты

**Средний уровень (3-4 балла):**
- Понимает взаимосвязи между компонентами
- Может предложить решения для типовых проблем
- Знает best practices и их обоснование

**Продвинутый уровень (5 баллов):**
- Глубокое понимание архитектурных решений
- Может спроектировать комплексные решения
- Понимает trade-offs и может обосновать выбор
- Знает продвинутые техники и паттерны

### Дополнительные вопросы для углубления:

1. **Опыт реализации**: "Расскажите о самом сложном CI/CD проекте, который вы реализовывали"
2. **Troubleshooting**: "Как бы вы debug медленно работающий CI pipeline?"
3. **Масштабирование**: "Как адаптировать CI/CD для команды из 100+ разработчиков?"
4. **Инновации**: "Какие новые trends в CI/CD вы считаете перспективными?"
5. **Культура**: "Как внедрить CI/CD практики в команду, которая их не использует?"

### Рекомендации по проведению интервью:

- Начинайте с базовых вопросов и углубляйтесь в зависимости от ответов
- Просите конкретные примеры из опыта кандидата
- Обращайте внимание на способность объяснить trade-offs
- Оценивайте не только знания, но и практический опыт
- Адаптируйте сложность вопросов под уровень позиции