# Интервью Backend разработчика: Git, GitHub, TDD
## Компактное руководство для глубокой оценки

### 📋 Инструкция по использованию

**Формат:** 40 ключевых вопросов (вместо 80) с усиленными углубляющими блоками
**Время:** 60-90 минут полного интервью  
**Подход:** Адаптивное углубление на основе ответов кандидата

**Система оценки:**
- **🟢 Junior (1-3 года):** Основной вопрос + 2-3 углубляющих
- **🟡 Middle (3-5 лет):** Основной вопрос + 4-5 углубляющих + практические сценарии  
- **🔴 Senior (5+ лет):** Все вопросы + архитектурные решения + менторинг

---

## 🔧 БЛОК 1: GIT (15 вопросов)

### **1. Merge vs Rebase: объясните различия и применение**
**🎯 Цель:** Оценка понимания workflow и влияния на историю коммитов

**Углубляющие вопросы:**
1. **Практический сценарий:** У вас feature-ветка с 5 коммитами, master обновился. Что выберете и почему?
2. **Golden rule of rebasing** - объясните и приведите пример нарушения
3. **Interactive rebase:** когда использовать и как squash коммиты безопасно?
4. **Конфликты:** В чем разница разрешения конфликтов при merge vs rebase?
5. **Командная работа:** Как принять решение о merge/rebase стратегии в команде?

---

### **2. Git стратегии ветвления: выбор под проект**
**🎯 Цель:** Понимание различных workflow и умение выбирать подходящий

**Углубляющие вопросы:**
1. **Git Flow vs GitHub Flow vs GitLab Flow** - когда какой использовать?
2. **Trunk-based development** - плюсы/минусы, когда подходит?
3. **Hotfix процесс** - как организовать в разных стратегиях?
4. **Команда 20+ разработчиков** - какую стратегию предложите и почему?
5. **Legacy проект vs greenfield** - как стратегия ветвления должна отличаться?

---

### **3. Продвинутая навигация и восстановление в Git**
**🎯 Цель:** Навыки работы с историей и аварийного восстановления

**Углубляющие вопросы:**
1. **Сценарий:** `git reset --hard HEAD~5` случайно выполнен. Как восстановить?
2. **reflog vs git log** - в чем разница и когда что использовать?
3. **Detached HEAD** - как попасть, зачем нужно, как выйти безопасно?
4. **git bisect** - автоматизация поиска бага, покажите на примере
5. **Удаленная ветка восстановление** - коллега удалил важную ветку, ваши действия?

---

### **4. Git hooks и автоматизация качества кода**
**🎯 Цель:** Понимание автоматизации процессов через Git

**Углубляющие вопросы:**
1. **Pre-commit хук** - создайте пример для проверки стиля кода и тестов
2. **Client vs server hooks** - различия, когда что использовать?
3. **Обход хуков** - можно ли и как предотвратить в командной работе?
4. **Shared hooks** - как распространить хуки на всю команду?
5. **Security hooks** - как предотвратить коммит секретов?

---

### **5. Производительность и оптимизация Git**
**🎯 Цель:** Понимание внутреннего устройства Git и оптимизации

**Углубляющие вопросы:**
1. **Большой репозиторий** (>1GB) - как оптимизировать клонирование и работу?
2. **git gc, packfiles, git filter-repo** - когда и как использовать?
3. **LFS vs submodules** - сравните подходы для больших файлов
4. **Monorepo в Git** - какие проблемы и как решать (sparse-checkout и др.)?
5. **Git protokols** (SSH vs HTTPS vs Git) - производительность и безопасность

---

### **6. Продвинутые Git операции**
**🎯 Цель:** Навыки сложных операций с Git

**Углубляющие вопросы:**
1. **Cherry-pick vs merge** - когда что использовать, какие проблемы создает cherry-pick?
2. **git worktree** - практические сценарии использования
3. **git stash advanced** - именованные stash, partial stash, stash на другую ветку
4. **git blame vs git log -p** - поиск изменений в коде, какой инструмент когда
5. **Переписывание истории** - безопасные практики, команды, предосторожности

---

### **7. Git в команде: конфликты и collaboration**
**🎯 Цель:** Навыки командной работы и решения конфликтов

**Углубляющие вопросы:**
1. **Сложный merge конфликт** - стратегии разрешения, инструменты, профилактика
2. **git blame vs git log --follow** - исследование истории изменений файла
3. **Конфигурация Git для команды** - что стандартизировать, как внедрить?
4. **Code review workflow** - как организовать через Git (без GitHub пока)?
5. **git attributes, gitignore templates** - организация для разных окружений

---

### **8. Git security и compliance**
**🎯 Цель:** Понимание безопасности и соответствия требованиям

**Углубляющие вопросы:**
1. **GPG подписи коммитов** - зачем, как настроить, как проверять?
2. **Секреты в Git истории** - как найти, как удалить безопасно?
3. **git-secrets, pre-commit сканеры** - настройка защиты от утечек
4. **Audit trail в Git** - как отследить все изменения и их авторов?
5. **Bare repository security** - настройка безопасного центрального репозитория

---

### **9. Git интеграции и tooling**
**🎯 Цель:** Интеграция Git в dev workflow

**Углубляющие вопросы:**
1. **IDE vs command line** - что когда использовать, плюсы/минусы?
2. **git aliases и custom commands** - примеры полезных, как создавать?
3. **git в CI/CD** - best practices, shallow clone, cache стратегии
4. **Git API и автоматизация** - libgit2, примеры автоматизации
5. **Миграция Git репозиториев** - как перенести с сохранением истории?

---

### **10. Специфические Git сценарии**
**🎯 Цель:** Решение нестандартных задач

**Углубляющие вопросы:**
1. **Крупный рефакторинг** - как организовать с минимальными конфликтами?
2. **git subtree vs submodules** - когда что использовать, миграция между ними
3. **Binary files в Git** - стратегии работы, LFS настройка
4. **Multiple remotes** - работа с несколькими удаленными репозиториями
5. **git bundle** - создание offline копий, использование для backup

---

## 🐙 БЛОК 2: GITHUB (12 вопросов)

### **11. GitHub workflow и automation**
**🎯 Цель:** Эффективная организация workflow на GitHub

**Углубляющие вопросы:**
1. **PR стратегия** - templates, labels, auto-assignment, branch protection
2. **Issues vs Discussions vs Projects** - когда что использовать, интеграция
3. **GitHub Flow vs Git Flow** на практике - адаптация под разные команды
4. **CODEOWNERS и review process** - настройка обязательных review
5. **GitHub Apps vs Actions** - когда создавать приложение vs использовать Actions

---

### **12. GitHub Actions: CI/CD mastery**
**🎯 Цель:** Продвинутое использование GitHub Actions

**Углубляющие вопросы:**
1. **Matrix strategy** - создайте workflow для тестирования на 3 ОС и 4 версиях языка
2. **Secrets management** - уровни secrets, ротация, best practices
3. **Self-hosted runners** - когда нужны, безопасность, масштабирование
4. **Workflow optimization** - кэширование, параллелизм, conditional jobs
5. **Custom Actions** - создание переиспользуемого Action для Marketplace

---

### **13. GitHub Security и Enterprise**
**🎯 Цель:** Безопасность и корпоративное использование

**Углубляющие вопросы:**
1. **Dependabot, CodeQL, Secret scanning** - настройка полного security pipeline
2. **GitHub Advanced Security** - ROI, внедрение в большой организации
3. **SSO, SAML, audit logs** - корпоративная безопасность и compliance
4. **Branch protection rules** - создайте политику для критически важного проекта
5. **GitHub Enterprise** vs Cloud - критерии выбора, миграция

---

### **14. GitHub продвинутые возможности**
**🎯 Цель:** Использование современных GitHub возможностей

**Углубляющие вопросы:**
1. **GitHub Packages** - настройка приватного registry, интеграция с CI/CD
2. **GitHub Copilot в команде** - внедрение, обучение, best practices
3. **Codespaces configuration** - dev containers, персонализация для команды
4. **GitHub API и GraphQL** - автоматизация процессов, custom integrations
5. **GitHub Insights** - метрики команды, улучшение процессов

---

### **15. Open Source и Community management**
**🎯 Цель:** Управление Open Source проектами

**Углубляющие вопросы:**
1. **Open Source project setup** - что должно быть в репозитории, governance
2. **Community building** - привлечение contributors, good first issues
3. **GitHub Sponsors и monetization** - стратегии финансирования проектов
4. **License management** - выбор лицензии, compliance tracking
5. **Maintainer workflow** - управление большим количеством contributors

---

## 🧪 БЛОК 3: TDD (13 вопросов)

### **16. TDD fundamentals и philosophy**
**🎯 Цель:** Глубокое понимание TDD принципов

**Углубляющие вопросы:**
1. **Red-Green-Refactor** - покажите на живом примере, типичные ошибки
2. **TDD vs Test-After** - конкретные преимущества, когда TDD не подходит?
3. **YAGNI в TDD** - как избежать over-engineering при TDD?
4. **TDD adoption** - как внедрить в команду, преодоление сопротивления
5. **TDD metrics** - как измерить эффективность TDD в проекте?

---

### **17. Testing patterns и best practices**
**🎯 Цель:** Продвинутые техники написания тестов

**Углубляющие вопросы:**
1. **AAA vs Given-When-Then** - когда что использовать, примеры
2. **Test naming conventions** - как называть тесты для максимальной ясности?
3. **Test data management** - fixtures, factories, builders - что когда?
4. **Test isolation** - как обеспечить независимость тестов?
5. **Flaky tests** - причины, профилактика, исправление

---

### **18. Mocking и test doubles strategy**
**🎯 Цель:** Эффективное использование test doubles

**Углубляющие вопросы:**
1. **Mock vs Stub vs Fake vs Spy** - конкретные примеры использования каждого
2. **Over-mocking problem** - как избежать, когда mocking вредит?
3. **Database testing** - in-memory vs real DB vs mocks, trade-offs
4. **API testing** - contract testing, mocking external services
5. **Time и randomness** - как тестировать код с текущим временем и случайностью?

---

### **19. Testing pyramid и strategy**
**🎯 Цель:** Архитектура тестирования в проекте

**Углубляющие вопросы:**
1. **Unit vs Integration vs E2E** - соотношение, как определить границы?
2. **Test coverage** - целевые проценты, mutation testing, coverage holes
3. **Testing in microservices** - consumer contracts, service virtualization
4. **Performance testing integration** - как включить в TDD workflow?
5. **Testing pyramid evolution** - как меняется с архитектурой проекта?

---

### **20. Async и complex scenarios testing**
**🎯 Цель:** Тестирование сложных сценариев

**Углубляющие вопросы:**
1. **Async code testing** - Promises, async/await, event-driven код
2. **Concurrency testing** - race conditions, deadlocks, как тестировать?
3. **Error handling testing** - exception paths, error recovery
4. **State machine testing** - как тестировать сложную бизнес-логику?
5. **Integration with external systems** - third-party APIs, message queues

---

### **21. TDD в различных контекстах**
**🎯 Цель:** Адаптация TDD под разные типы проектов

**Углубляющие вопросы:**
1. **Legacy code** - characterization tests, seam injection, refactoring strategy
2. **API-first development** - contract-driven TDD, OpenAPI integration
3. **Event-driven architecture** - testing events, sagas, eventual consistency
4. **Security testing** - как включить security в TDD workflow?
5. **Infrastructure as Code** - TDD для terraform, ansible и других IaC

---

### **22. Testing tools и automation**
**🎯 Цель:** Выбор и использование инструментов тестирования

**Углубляющие вопросы:**
1. **Testing framework comparison** - критерии выбора для вашего стека
2. **Property-based testing** - когда использовать, генерация test cases
3. **Snapshot testing** - плюсы/минусы, поддержка актуальности
4. **Visual regression testing** - интеграция в pipeline, handling changes
5. **Test automation in CI/CD** - parallel execution, test sharding, reporting

---

### **23. Advanced testing techniques**
**🎯 Цель:** Современные подходы к тестированию

**Углубляющие вопросы:**
1. **BDD integration** - Gherkin, living documentation, business collaboration
2. **Contract testing** - Pact, consumer-driven contracts для микросервисов
3. **Chaos engineering** - как интегрировать с testing strategy?
4. **A/B testing** - infrastructure для feature flags и testing
5. **Testing in production** - canary deployments, monitoring, observability

---

### **24. Team practices и culture**
**🎯 Цель:** Культура тестирования в команде

**Углубляющие вопросы:**
1. **Code review для тестов** - что проверять, стандарты качества
2. **Test-first culture** - как привить, метрики adoption
3. **Testing documentation** - living docs, test как спецификация
4. **Pair/mob programming** - TDD в collaborative coding
5. **Testing training** - как обучать junior разработчиков TDD?

---

### **25. Continuous testing и DevOps**
**🎯 Цель:** Интеграция тестирования в DevOps процессы

**Углубляющие вопросы:**
1. **Shift-left testing** - раннее обнаружение проблем, prevention vs detection
2. **Test data management** - production-like data, privacy, GDPR compliance
3. **Testing environments** - staging, preview environments, infrastructure
4. **Quality gates** - автоматические критерии для deploy, rollback triggers
5. **Testing metrics и KPIs** - lead time, MTTR, defect escape rate, ROI

---

### **26. Testing architecture decisions**
**🎯 Цель:** Архитектурные решения в тестировании

**Углубляющие вопросы:**
1. **Test architecture** - слои, зависимости, переиспользование компонентов
2. **Test execution strategy** - local vs cloud, resource management
3. **Test result analysis** - trend analysis, failure categorization
4. **Testing cost optimization** - compute costs, time optimization
5. **Scale testing** - как тестировать high-load системы?

---

### **27. Future of testing**
**🎯 Цель:** Понимание трендов и будущего тестирования

**Углубляющие вопросы:**
1. **AI in testing** - генерация тестов, auto-healing, intelligent test selection
2. **Testing cloud-native apps** - containers, serverless, distributed systems
3. **Observability vs testing** - как monitoring дополняет тesting strategy?
4. **Testing for ML/AI systems** - data testing, model validation, bias detection
5. **Low-code/no-code testing** - будущее автоматизации тестирования

---

### **28. Problem-solving scenarios**
**🎯 Цель:** Практическое применение знаний

**Углубляющие вопросы:**
1. **Дизайн тестовой стратегии** для нового проекта с нуля - что учесть?
2. **Debug failing test** - подход к диагностике нестабильных тестов
3. **Test performance optimization** - медленные тесты, как ускорить?
4. **Testing complex business rules** - многоуровневая валидация, edge cases
5. **Testing migration scenarios** - данные, схемы, backward compatibility

---

## 📊 Система оценки и подведение итогов

### **Критерии оценки по уровням:**

#### 🟢 **Junior Backend Developer (1-3 года)**
- **Git:** Основные команды, простой workflow, понимание merge/rebase
- **GitHub:** PR процесс, basic Actions, Issues
- **TDD:** Red-Green-Refactor, unit тесты, AAA pattern

#### 🟡 **Middle Backend Developer (3-5 лет)** 
- **Git:** Сложные сценарии восстановления, hooks, стратегии ветвления
- **GitHub:** Advanced Actions, security features, team collaboration
- **TDD:** Integration тесты, mocking стратегии, testing pyramid

#### 🔴 **Senior Backend Developer (5+ лет)**
- **Git:** Архитектурные решения, performance, enterprise scenarios
- **GitHub:** Enterprise features, automation, процессы в масштабе
- **TDD:** Testing strategy, культура тестирования, complex scenarios

### **Итоговая оценка:**
- **Excellent (90-100%):** Все уровни + ментроинг + инновации
- **Good (70-89%):** Соответствует уровню + некоторые продвинутые темы  
- **Satisfactory (50-69%):** Базовый уровень с пробелами
- **Needs Improvement (<50%):** Существенные пробелы в знаниях

---

## ✅ Финальные рекомендации

### **Для интервьюера:**
1. **Адаптируйтесь** - начинайте с основных вопросов, углубляйтесь по мере успешных ответов
2. **Практические примеры** - просите код, конкретные команды, real-world сценарии
3. **Мышление важнее знаний** - оценивайте подход к решению проблем
4. **Честность ценится** - "не знаю, но разберусь" лучше чем неверный ответ

### **Красные флаги:**
- ❌ Теоретические знания без практического применения
- ❌ Неумение объяснить простыми словами
- ❌ Отсутствие понимания команды/процессов  
- ❌ Нежелание признавать пробелы в знаниях

### **Зеленые флаги:**
- ✅ Практические примеры из реального опыта
- ✅ Понимание trade-offs и альтернативных решений
- ✅ Вопросы о контексте и требованиях
- ✅ Стремление к continuous learning