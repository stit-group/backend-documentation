# План изучения DevOps & TBD: Вопросы и структура

## 🎯 Общий подход к изучению

### Методика изучения:
1. **Теория** → **Практика** → **Рефлексия** → **Применение**
2. Каждую неделю: изучить концепции, выполнить практические задания, ответить на контрольные вопросы
3. Ведите дневник обучения с заметками и инсайтами
4. Создавайте практические проекты для закрепления

---

## 📚 Фаза 1: DevOps Фундаменты (недели 1-3)

### Неделя 1: DevOps Культура

**Ключевые вопросы для изучения:**
- Что такое DevOps и почему это культура, а не инструменты?
- Какие проблемы решает DevOps в традиционной разработке?
- Что такое DORA метрики и как их измерять?
- Как рассчитать базовые показатели команды?

**Контрольные вопросы:**
1. Объясните разницу между традиционным подходом Dev→QA→Ops и DevOps
2. Перечислите 4 DORA метрики и их целевые значения для элитных команд
3. Как измерить текущий Deployment Frequency вашей команды?
4. Что означает "сдвиг влево" (shift-left) в контексте DevOps?

**Практическое задание:**
- Проанализируйте Git историю проекта за последний месяц
- Рассчитайте базовые метрики (частота коммитов, время между релизами)
- Создайте dashboard с текущими показателями

### Неделя 2-3: Git Mastery

**Ключевые вопросы для изучения:**
- Что такое Semantic Versioning и зачем он нужен?
- Как правильно писать Conventional Commits?
- Какие Git hooks можно использовать для автоматизации?
- Как настроить branch protection и code review процесс?

**Контрольные вопросы:**
1. Приведите примеры правильных commit messages в формате Conventional Commits
2. Объясните структуру версии 2.3.1 в Semantic Versioning
3. Какие проверки должны выполняться в pre-commit хуке?
4. Как настроить автоматическое создание changelog из Git истории?

**Практическое задание:**
- Настройте pre-commit hooks для проекта
- Создайте GitHub/GitLab репозиторий с branch protection
- Внедрите Conventional Commits в команде

---

## 🌿 Фаза 2: Trunk-Based Development (недели 4-5)

### Неделя 4: TBD Философия

**Ключевые вопросы для изучения:**
- В чем отличие TBD от Git Flow и GitHub Flow?
- Почему короткие ветки лучше длинных?
- Что такое feature flags и как их использовать?
- Как организовать code review в TBD?

**Контрольные вопросы:**
1. Сравните TBD и Git Flow: преимущества и недостатки каждого подхода
2. Какова максимальная продолжительность жизни ветки в TBD?
3. Приведите пример использования feature flag в коде
4. Как избежать "merge hell" при работе с TBD?

**Практическое задание:**
- Реализуйте простую систему feature flags
- Создайте workflow для TBD в существующем проекте
- Проведите эксперимент: сравните время интеграции в TBD vs feature branches

### Неделя 5: Качество в TBD

**Ключевые вопросы для изучения:**
- Как обеспечить качество кода без долгих веток?
- Что такое "Definition of Done" для TBD?
- Как организовать быстрый code review?
- Какие автоматизированные проверки критически важны?

**Контрольные вопросы:**
1. Перечислите критерии качественного PR в TBD (размер, содержание, тесты)
2. Как backward compatibility связана с TBD?
3. Что должно проверяться автоматически перед merge?
4. Как измерить эффективность code review процесса?

**Практическое задание:**
- Создайте checklist для Definition of Done
- Настройте автоматические проверки качества кода
- Оптимизируйте процесс code review (время, критерии)

---

## 🔄 Фаза 3: Continuous Integration (недели 6-8)

### Неделя 6: CI Основы

**Ключевые вопросы для изучения:**
- Что такое CI и какие проблемы он решает?
- Как построить эффективную test pyramid?
- Какие метрики важны для CI pipeline?
- Как организовать параллельное выполнение тестов?

**Контрольные вопросы:**
1. Объясните принцип "fail fast" в контексте CI
2. Распределите тесты по уровням test pyramid для типичного веб-приложения
3. Какое время выполнения CI считается приемлемым?
4. Как измерить test coverage и какой уровень считается достаточным?

**Практическое задание:**
- Создайте базовый CI pipeline для проекта
- Настройте параллельное выполнение тестов
- Добавьте отчеты о test coverage

### Неделя 7: Advanced CI

**Ключевые вопросы для изучения:**
- Что такое Test Containers и когда их использовать?
- Как оптимизировать время выполнения CI?
- Как организовать CI для монорепозитория?
- Какие стратегии кеширования эффективны?

**Контрольные вопросы:**
1. В чем преимущества Test Containers перед mock'ами для интеграционных тестов?
2. Перечислите 5 способов ускорить CI pipeline
3. Как организовать selective testing в монорепозитории?
4. Какие зависимости стоит кешировать в CI?

**Практическое задание:**
- Внедрите Test Containers в интеграционные тесты
- Оптимизируйте время выполнения существующего CI
- Настройте smart caching для dependencies

### Неделя 8: CI + TBD Integration

**Ключевые вопросы для изучения:**
- Как адаптировать CI для TBD workflow?
- Что такое "trunk-based CI" и как его настроить?
- Как организовать automated rollback?
- Какие проверки должны блокировать merge в main?

**Контрольные вопросы:**
1. Чем отличается CI стратегия для TBD от feature branch workflow?
2. Как обеспечить, что main branch всегда deployable?
3. Какие условия должны запускать automated rollback?
4. Как организовать quick feedback loop для разработчиков?

**Практическое задание:**
- Настройте двухуровневый CI (quick checks + full pipeline)
- Реализуйте automated rollback mechanism
- Создайте dashboard для отслеживания состояния main branch

---

## 🚀 Фаза 4: Continuous Delivery (недели 9-12)

### Неделя 9-10: CD Основы

**Ключевые вопросы для изучения:**
- В чем разница между Continuous Delivery и Continuous Deployment?
- Какие deployment strategies существуют?
- Что такое Infrastructure as Code и зачем он нужен?
- Как организовать environment promotion?

**Контрольные вопросы:**
1. Сравните Blue-Green и Canary deployment стратегии
2. Объясните принцип "infrastructure as code"
3. Как обеспечить consistency между средами разработки и production?
4. Какие проверки должны выполняться на каждой стадии deployment pipeline?

**Практическое задание:**
- Создайте multi-stage deployment pipeline
- Реализуйте Blue-Green deployment
- Напишите Infrastructure as Code для тестовой среды

### Неделя 11: Advanced Deployment

**Ключевые вопросы для изучения:**
- Как управлять database migrations в CD?
- Что такое backward compatible changes?
- Как организовать secrets management?
- Какие health checks критически важны?

**Контрольные вопросы:**
1. Приведите примеры backward compatible и breaking database changes
2. Как обеспечить zero-downtime deployment при изменении схемы БД?
3. Перечислите best practices для secrets management
4. Какие типы health checks должны быть в production приложении?

**Практическое задание:**
- Создайте систему для управления database migrations
- Настройте secrets management (HashiCorp Vault или аналог)
- Реализуйте comprehensive health checks

### Неделя 12: CD для TBD

**Ключевые вопросы для изучения:**
- Как организовать continuous deployment в TBD?
- Роль feature flags в production deployment
- Что такое progressive rollout?
- Как измерять success deployment в production?

**Контрольные вопросы:**
1. Объясните, как feature flags позволяют разделить код deployment и feature release
2. Что такое canary analysis и как ее автоматизировать?
3. Какие метрики указывают на необходимость rollback?
4. Как организовать A/B testing в рамках CD pipeline?

**Практическое задание:**
- Настройте автоматический deployment каждого merge в main
- Реализуйте progressive rollout с feature flags
- Создайте автоматическую canary analysis

---

## 🏗 Фаза 5: Infrastructure & Operations (недели 13-15)

### Неделя 13: Контейнеризация

**Ключевые вопросы для изучения:**
- Зачем нужны контейнеры и как они решают проблемы?
- Как создать оптимальный Dockerfile?
- Что такое multi-stage builds?
- Как организовать локальную разработку с контейнерами?

**Контрольные вопросы:**
1. Перечислите преимущества контейнеров перед виртуальными машинами
2. Объясните принцип multi-stage builds в Docker
3. Какие security best practices важны для контейнеров?
4. Как минимизировать размер Docker образа?

**Практическое задание:**
- Создайте оптимальный Dockerfile для приложения
- Настройте Docker Compose для локальной разработки
- Реализуйте multi-stage build

### Неделя 14: Kubernetes Orchestration

**Ключевые вопросы для изучения:**
- Что такое Kubernetes и какие проблемы он решает?
- Как работают основные объекты K8s (Pod, Deployment, Service)?
- Что такое health checks в Kubernetes?
- Как настроить auto-scaling?

**Контрольные вопросы:**
1. Объясните разницу между liveness и readiness probes
2. Как работает Horizontal Pod Autoscaler?
3. Что такое resource requests и limits?
4. Как организовать service discovery в Kubernetes?

**Практическое задание:**
- Развертайте приложение в Kubernetes
- Настройте HPA и health checks
- Создайте monitoring для кластера

### Неделя 15: Infrastructure as Code

**Ключевые вопросы для изучения:**
- Что такое GitOps и как он работает?
- Как управлять инфраструктурой через код?
- Что такое drift detection?
- Как организовать multi-environment инфраструктуру?

**Контрольные вопросы:**
1. Сравните imperative и declarative подходы к управлению инфраструктурой
2. Объясните принципы GitOps
3. Как обеспечить consistency между environments?
4. Что такое infrastructure drift и как его предотвратить?

**Практическое задание:**
- Создайте Infrastructure as Code для проекта
- Настройте GitOps workflow
- Реализуйте multi-environment управление

---

## 📊 Фаза 6: Monitoring & Observability (недели 16-17)

### Неделя 16: Metrics & Logging

**Ключевые вопросы для изучения:**
- Что такое 4 Golden Signals мониторинга?
- Как организовать structured logging?
- Какие метрики критически важны для backend приложения?
- Как создать эффективные dashboards?

**Контрольные вопросы:**
1. Перечислите 4 Golden Signals и объясните каждый
2. В чем преимущества structured logging перед plain text?
3. Какие бизнес-метрики важно отслеживать в production?
4. Как избежать "dashboard hell" - избытка метрик?

**Практическое задание:**
- Добавьте metrics в приложение (Prometheus)
- Настройте structured logging
- Создайте Grafana dashboard

### Неделя 17: Distributed Tracing

**Ключевые вопросы для изучения:**
- Что такое distributed tracing и зачем он нужен?
- Как работает OpenTelemetry?
- Что такое spans и traces?
- Как организовать alerting на основе метрик?

**Контрольные вопросы:**
1. Объясните разницу между logs, metrics и traces
2. Как distributed tracing помогает в debugging микросервисов?
3. Что такое sampling в tracing и почему он важен?
4. Какие алерты должны быть настроены для production системы?

**Практическое задание:**
- Внедрите distributed tracing
- Настройте alerting rules
- Создайте runbooks для алертов

---

## 🔒 Фаза 7: Security (DevSecOps) (недели 18-19)

### Неделя 18: Security в Pipeline

**Ключевые вопросы для изучения:**
- Что такое "shift-left security"?
- Какие виды security scanning нужны в CI/CD?
- Как организовать secrets management?
- Что такое SAST и DAST?

**Контрольные вопросы:**
1. Объясните концепцию "shift-left security"
2. Перечислите типы security scanning для CI/CD pipeline
3. Как предотвратить попадание секретов в код?
4. В чем разница между SAST и DAST?

**Практическое задание:**
- Добавьте security scanning в CI
- Настройте secrets management
- Создайте security policy для команды

### Неделя 19: Production Security

**Ключевые вопросы для изучения:**
- Как обеспечить network security в Kubernetes?
- Что такое runtime security?
- Как организовать security monitoring?
- Какие compliance требования важны?

**Контрольные вопросы:**
1. Объясните принцип zero-trust networking
2. Что такое Kubernetes Network Policies?
3. Как организовать security incident response?
4. Какие security headers должны быть настроены?

**Практическое задание:**
- Настройте network policies
- Реализуйте security monitoring
- Создайте incident response plan

---

## ⚡ Фаза 8: Performance & Scaling (недели 20-21)

### Неделя 20: Performance Testing

**Ключевые вопросы для изучения:**
- Как интегрировать performance testing в CI/CD?
- Что такое performance regression?
- Какие виды нагрузочного тестирования существуют?
- Как организовать effective caching?

**Контрольные вопросы:**
1. Объясните разницу между load, stress и spike testing
2. Как предотвратить performance regression?
3. Какие caching стратегии наиболее эффективны?
4. Как измерить performance в production?

**Практическое задание:**
- Добавьте performance tests в CI
- Реализуйте multi-level caching
- Настройте APM monitoring

### Неделя 21: Auto-scaling

**Ключевые вопросы для изучения:**
- Как работает auto-scaling в Kubernetes?
- Что такое circuit breaker pattern?
- Как оптимизировать costs в cloud?
- Какие patterns помогают в scaling?

**Контрольные вопросы:**
1. Объясните работу HPA и VPA в Kubernetes
2. Когда использовать circuit breaker pattern?
3. Как балансировать performance и cost?
4. Что такое bulkhead pattern?

**Практическое задание:**
- Настройте auto-scaling
- Реализуйте circuit breaker
- Оптимизируйте resource utilization

---

## 🚀 Фаза 9: Advanced Practices (недели 22-23)

### Неделя 22: Enterprise TBD

**Ключевые вопросы для изучения:**
- Как масштабировать TBD для больших команд?
- Что такое монорепозиторий и когда его использовать?
- Как организовать cross-team coordination?
- Какие метрики важны для enterprise TBD?

**Контрольные вопросы:**
1. Сравните монорепозиторий и мультирепозиторий подходы
2. Как организовать dependency management в TBD?
3. Какие процессы нужны для coordination между командами?
4. Как измерить эффективность TBD на enterprise уровне?

**Практическое задание:**
- Создайте enterprise TBD strategy
- Настройте cross-team metrics
- Разработайте governance процессы

### Неделя 23: Continuous Improvement

**Ключевые вопросы для изучения:**
- Что такое SPACE framework?
- Как организовать chaos engineering?
- Что такое value stream mapping?
- Как измерить DevOps maturity?

**Контрольные вопросы:**
1. Объясните компоненты SPACE framework
2. Зачем нужен chaos engineering в production?
3. Как создать value stream map для процесса разработки?
4. Какие уровни DevOps maturity существуют?

**Практическое задание:**
- Проведите value stream mapping
- Настройте chaos engineering
- Оцените DevOps maturity команды

---

## 🎯 Фаза 10: Capstone Project (неделя 24)

### Финальный проект

**Ключевые вопросы для проекта:**
- Как спроектировать production-ready систему?
- Как обеспечить все аспекты DevOps в одном проекте?
- Как измерить success проекта?
- Что такое sustainable architecture?

**Контрольные вопросы:**
1. Какие архитектурные решения принимались и почему?
2. Как достигнуты целевые DORA метрики?
3. Какие lessons learned получены в процессе?
4. Как проект можно улучшить в будущем?

**Итоговый проект должен включать:**
- Полный TBD workflow
- Production-ready CI/CD
- Comprehensive monitoring
- Security best practices
- Performance optimization
- Documentation и runbooks

---

## 📈 Рекомендации по изучению

### Еженедельный план:
1. **Понедельник-Вторник**: Изучить теорию, прочитать материал
2. **Среда-Четверг**: Выполнить практические задания
3. **Пятница**: Ответить на контрольные вопросы, рефлексия
4. **Выходные**: Дополнительная практика, подготовка к следующей неделе

### Методы изучения:
- **Active Learning**: Не просто читать, а практиковать
- **Teach Back**: Объяснить изученное коллеге или записать в блог
- **Peer Review**: Обсуждать решения с другими
- **Continuous Reflection**: Регулярно анализировать прогресс

### Критерии успеха:
- Можете объяснить концепцию простыми словами
- Можете применить на практике
- Можете адаптировать под разные контексты
- Можете обучить других

---

## 🎓 Заключение

Этот план предоставляет структурированный подход к изучению DevOps и TBD. Помните:

- **Consistency over Intensity** - лучше заниматься понемногу каждый день
- **Practice over Theory** - больше практики, меньше теории
- **Reflection over Consumption** - анализировать изученное важнее потребления контента
- **Teaching over Learning** - обучение других закрепляет знания

Удачи в изучении! 🚀