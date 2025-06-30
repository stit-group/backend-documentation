# CI/CD Interview Template - Компактная версия

## 📋 Инструкция
- **Время**: 90-120 минут  
- **Формат**: 30 основных вопросов + сценарии углубления
- **Оценка**: 1-5 баллов (критерии указаны)
- **Структура**: Практические задачи + теоретические основы

---

## 🎯 Блок 1: Основы и Pipeline Design (8 вопросов)

### 1.1 Опишите идеальный CI/CD pipeline для веб-приложения с БД
**Углубление при хорошем ответе:**
- Как обеспечить zero-downtime deployments?
- Где разместить database migrations в pipeline?

**Оценка:**
- 🟢 5 баллов: Описывает стадии, gates, rollback strategy, database considerations
- 🟡 3 балла: Базовые стадии build→test→deploy, понимает основную логику
- 🔴 1 балл: Путается в терминологии, не видит картину целиком

### 1.2 У вас есть монорепо с 10 микросервисами. Как организовать CI/CD?
**Углубление:**
- Selective building vs rebuild all - когда что использовать?
- Как решать dependency hell между сервисами?

### 1.3 Команда жалуется, что CI pipeline занимает 45 минут. Ваши действия?
**Углубление:**
- Какие метрики собирать для анализа?
- Приоритизация оптимизаций - с чего начать?

### 1.4 Объясните разницу между GitFlow и trunk-based development
**Углубление:**
- Когда каждый подход предпочтительнее?
- Как feature flags влияют на выбор стратегии?

### 1.5 Что такое "Infrastructure as Code" и зачем это нужно?
**Углубление:**
- Как тестировать infrastructure code?
- Terraform vs Ansible vs CloudFormation - когда что?

### 1.6 Опишите стратегию branching для критического production приложения
**Углубление:**
- Как организовать hotfix workflow?
- Защита main branch - какие правила?

### 1.7 Что включает в себя "Shift Left Security"?
**Углубление:**
- Какие security проверки можно автоматизировать?
- SAST vs DAST - в какой момент pipeline?

### 1.8 Как обеспечить reproducible builds?
**Углубление:**
- Lock files, docker images, dependency management
- Как детектировать non-reproducible builds?

---

## 🔧 Блок 2: Автоматизация и Testing (7 вопросов)

### 2.1 Пирамида тестирования - как применить в CI/CD?
**Углубление:**
- Сколько времени тратить на каждый уровень?
- Как организовать parallel execution?

**Оценка:**
- 🟢 5 баллов: Понимает соотношение unit/integration/e2e, timing, parallel strategies
- 🟡 3 балла: Знает уровни, понимает базовые принципы
- 🔴 1 балл: Путает типы тестов

### 2.2 Contract testing - что это и зачем нужно?
**Углубление:**
- Consumer-driven contracts на практике
- Integration с API versioning

### 2.3 Flaky tests убивают доверие к CI. Как решать?
**Углубление:**
- Стратегии обнаружения flaky tests
- Quarantine vs immediate fix подходы

### 2.4 Как организовать test data management?
**Углубление:**
- Test fixtures vs synthetic data vs production snapshots
- GDPR compliance для test data

### 2.5 Performance testing в CI - возможно ли?
**Углубление:**
- Какие метрики отслеживать?
- Performance budgets и regression detection

### 2.6 End-to-end тесты для микросервисов - как не сойти с ума?
**Углубление:**
- Service virtualization strategies
- Test environment management

### 2.7 Mutation testing - когда оправдано?
**Углубление:**
- ROI от mutation testing
- Integration в существующий pipeline

---

## 🐳 Блок 3: Контейнеризация и Kubernetes (8 вопросов)

### 3.1 Best practices для production-ready Dockerfile
**Углубление:**
- Multi-stage builds для оптимизации
- Security scanning integration

**Оценка:**
- 🟢 5 баллов: Layer optimization, security, multi-stage, base image management
- 🟡 3 балла: Понимает основы, знает COPY vs ADD
- 🔴 1 балл: Базовое понимание контейнеров

### 3.2 Container registry strategy для enterprise
**Углубление:**
- Image promotion между environments
- Vulnerability scanning workflow

### 3.3 Kubernetes deployment strategies - сравните подходы
**Углубление:**
- Rolling vs Blue-Green vs Canary - trade-offs
- Automated rollback triggers

### 3.4 GitOps в Kubernetes - архитектура решения
**Углубление:**
- ArgoCD vs Flux vs Jenkins X
- Multi-cluster management

### 3.5 Helm charts в CI/CD pipeline
**Углубление:**
- Chart testing strategies
- Templating vs configuration management

### 3.6 Service mesh в deployment pipeline
**Углубление:**
- Traffic routing для canary deployments
- Observability integration

### 3.7 Kubernetes secrets management
**Углубление:**
- External secret operators
- Secret rotation automation

### 3.8 Resource management и scaling
**Углубление:**
- HPA vs VPA strategies
- Cost optimization practices

---

## 🚀 Блок 4: Deployment и Production (7 вопросов)

### 4.1 Database migrations в zero-downtime deployment
**Углубление:**
- Backward compatibility strategies
- Rollback scenarios для schema changes

**Оценка:**
- 🟢 5 баллов: Понимает backward compatibility, multi-step migrations, rollback strategies
- 🟡 3 балла: Знает основные принципы, может предложить решение
- 🔴 1 балл: Не видит проблемы или предлагает downtime

### 4.2 Feature flags architecture и lifecycle
**Углубление:**
- Technical debt от feature flags
- A/B testing integration

### 4.3 Multi-region deployment strategy
**Углубление:**
- Data consistency между регионами
- Disaster recovery procedures

### 4.4 Progressive delivery implementation
**Углубление:**
- Automated promotion criteria
- Business metrics integration

### 4.5 Environment promotion strategy
**Углубление:**
- Environment parity challenges
- Configuration drift detection

### 4.6 Incident response в контексте deployments
**Углубление:**
- Automated rollback triggers
- Communication workflows

### 4.7 Deployment windows vs continuous deployment
**Углубление:**
- Risk assessment frameworks
- Emergency deployment procedures

---

## 📊 Практические сценарии (углубленная оценка)

### Сценарий 1: Crisis Management
> "Production упал после деплоя 2 часа назад. Rollback не работает, БД была мигрирована. Ваши действия по шагам?"

**Ожидаемое обсуждение:**
- Incident response procedures
- Root cause analysis approach
- Communication strategy
- Prevention measures

### Сценарий 2: Scale Challenge  
> "Команда выросла с 5 до 50 разработчиков. CI время увеличилось до 2 часов, конфликты merge каждый день. План действий?"

**Ожидаемое обсуждение:**
- Pipeline optimization strategies
- Branching strategy changes
- Infrastructure scaling
- Process improvements

### Сценарий 3: Compliance Requirement
> "Нужно внедрить SOX compliance - все изменения должны быть auditable, segregation of duties. Как адаптировать существующий CI/CD?"

**Ожидаемое обсуждение:**
- Audit trail implementation
- Approval workflows
- Access control strategies
- Documentation automation

---

## 🎯 Критерии итоговой оценки

### Senior Level (22-30 баллов)
- **Архитектурное мышление**: Видит систему целиком, понимает trade-offs
- **Практический опыт**: Конкретные примеры решения сложных проблем  
- **Проактивность**: Предлагает улучшения, думает о метриках
- **Лидерство**: Может внедрять изменения, обучать команду

### Middle Level (15-21 балл)
- **Технические знания**: Понимает инструменты и практики
- **Problem solving**: Может решать стандартные задачи
- **Базовый опыт**: Работал с основными CI/CD инструментами
- **Потенциал роста**: Готов изучать новое

### Junior Level (8-14 баллов)
- **Основы**: Понимает базовые концепции
- **Теоретические знания**: Знает популярные инструменты
- **Мотивация**: Интерес к автоматизации и DevOps практикам

---

## 📝 Рекомендации по проведению

### Подготовка (10 минут)
- Ознакомьтесь с резюме кандидата
- Выберите 20-25 вопросов из списка
- Подготовьте 2-3 практических сценария

### Структура интервью
1. **Разминка** (10 мин): Опыт, текущие проекты
2. **Основные блоки** (60-80 мин): Выбранные вопросы + углубление
3. **Практические сценарии** (20-30 мин): 1-2 сценария
4. **Вопросы кандидата** (10 мин)

### Red Flags 🚩
- Не может объяснить trade-offs решений
- Фокус только на инструментах, а не на процессах
- Нет понимания business impact от CI/CD
- Не видит проблем безопасности

### Green Flags ✅  
- Приводит конкретные примеры из опыта
- Думает о метриках и измерениях
- Понимает человеческий фактор в процессах
- Предлагает улучшения существующих решений

---

*Этот шаблон покрывает все ключевые области CI/CD при значительно меньшем объеме. Фокус на практических навыках и архитектурном мышлении дает более точную оценку кандидата.*