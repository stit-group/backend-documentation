# Блок собеседования по Git для Backend разработчиков (TBD фокус)

## 🎯 Цель блока
Оценить понимание Git в контексте **Trunk-Based Development** и современных практик CI/CD 2025 года.

---

## 📋 Структура собеседования

### 1. Базовые концепции Git (5-7 минут)

#### ❓ Вопрос 1.1: Основы версионного контроля
**Вопрос:** Объясните разницу между Git и другими VCS (например, SVN). Почему Git стал стандартом?

**💡 Подсказка к ответу:**
- **Распределённость** vs централизованность
- **Локальные операции** (коммиты, ветки, история)
- **Снапшоты** вместо дельт файлов
- **Криптографическая целостность** (SHA-1 хеши)
- **Эффективное ветвление** и слияние

**✅ Ожидаемый ответ:**
Git - распределённая система, где каждый разработчик имеет полную копию репозитория. Это позволяет работать офлайн, делать локальные коммиты и ветки без обращения к серверу. Git хранит снапшоты состояния файлов, а не изменения, что делает операции быстрее и надёжнее.

#### ❓ Вопрос 1.2: Три дерева Git
**Вопрос:** Объясните "Three Trees of Git". Как работает staging area?

**💡 Подсказка к ответу:**
- **Working Directory** - рабочая копия
- **Staging Area (Index)** - подготовленные изменения
- **Repository (.git directory)** - история коммитов

**✅ Ожидаемый ответ:**
Working Directory содержит текущие файлы, Staging Area подготавливает изменения для коммита (позволяет выборочно включать файлы), Repository хранит историю. Эта архитектура позволяет контролировать, что именно попадёт в коммит.

---

### 2. Trunk-Based Development (10-15 минут)

#### ❓ Вопрос 2.1: Философия TBD
**Вопрос:** Что такое Trunk-Based Development? Чем отличается от GitFlow?

**💡 Подсказка к ответу:**
- **Одна основная ветка** (main/master) vs множественные долгоживущие ветки
- **Частые интеграции** (минимум раз в день)
- **Короткоживущие feature branches** (<1 дня) или прямые коммиты
- **Готовность к релизу** main ветки всегда

**🔍 Глубокий ответ:**
```
TBD vs GitFlow:

GitFlow:
├── master (production)
├── develop (integration)
├── feature/* (long-lived)
├── release/* (stabilization)
└── hotfix/* (emergency)

TBD:
├── main (always deployable)
├── short-lived features (hours/1 day max)
└── release branches (optional, created just-in-time)
```

**✅ Ожидаемый ответ:**
TBD - методология, где все разработчики интегрируют изменения в одну основную ветку (trunk) минимум раз в день. Отличается от GitFlow отсутствием долгоживущих веток, что уменьшает merge conflicts и упрощает CI/CD.

#### ❓ Вопрос 2.2: Преимущества и вызовы TBD
**Вопрос:** Какие преимущества даёт TBD для backend команд? Какие требует дисциплины?

**💡 Подсказка к ответу:**
**Преимущества:**
- Быстрая интеграция и обратная связь
- Меньше merge conflicts
- Лучшая поддержка CI/CD
- Упрощённая модель веток

**Требует:**
- Надёжные автотесты
- Feature flags/toggles
- Культура малых изменений
- Быстрые code review

**🔍 Практический пример:**
```bash
# Плохо - крупные изменения
git commit -m "Rewrite user authentication system"

# Хорошо - атомарные изменения
git commit -m "Add password validation utility"
git commit -m "Update user model with new auth fields"
git commit -m "Implement OAuth2 token validation"
```

#### ❓ Вопрос 2.3: Feature Flags в TBD
**Вопрос:** Как использовать feature flags в TBD для безопасного деплоя?

**💡 Подсказка к ответу:**
- **Скрытие недоготовых фич** в production
- **A/B тестирование**
- **Поэтапный rollout**
- **Быстрый rollback** без deploy

**🔍 Пример кода:**
```javascript
// Feature flag example
const isNewPaymentSystemEnabled = featureFlag('new_payment_system');

if (isNewPaymentSystemEnabled) {
    return processPaymentV2(data);
} else {
    return processPaymentV1(data);
}
```

---

### 3. Практические команды Git (8-10 минут)

#### ❓ Вопрос 3.1: Ежедневный workflow TBD
**Вопрос:** Опишите типичный день разработчика в TBD. Какие Git команды используются?

**💡 Подсказка к ответу:**
```bash
# Утром - синхронизация
git pull origin main
git status

# Работа с малыми изменениями
git add -p  # частичное добавление
git commit -m "descriptive message"

# Частые push'ы
git push origin main

# Или через short-lived branch
git checkout -b fix/user-validation
# работа...
git push origin fix/user-validation
# создание PR, быстрый review, merge
```

#### ❓ Вопрос 3.2: Продвинутые команды
**Вопрос:** Как использовать `git rebase` в контексте TBD? Когда `merge` vs `rebase`?

**💡 Подсказка к ответу:**
- **Rebase для clean history** перед merge в main
- **Interactive rebase** для squash commits
- **Merge** сохраняет контекст веток
- **В TBD предпочтителен rebase** для линейной истории

**🔍 Практические команды:**
```bash
# Rebase feature branch перед merge
git checkout feature/api-endpoint
git rebase main

# Interactive rebase для cleanup
git rebase -i HEAD~3

# Squash commits в один
git rebase -i HEAD~3
# заменить 'pick' на 'squash' для объединяемых коммитов
```

#### ❓ Вопрос 3.3: Разрешение конфликтов
**Вопрос:** Стратегии минимизации и разрешения merge conflicts в TBD?

**💡 Подсказка к ответу:**
**Минимизация:**
- Частые pull/rebase
- Малые изменения
- Разделение зон ответственности
- Communication в команде

**Разрешение:**
```bash
git status  # посмотреть конфликты
# редактировать файлы
git add resolved_file.js
git commit
```

---

### 4. CI/CD интеграция (5-7 минут)

#### ❓ Вопрос 4.1: Pipeline в TBD
**Вопрос:** Как выглядит идеальный CI/CD pipeline для TBD? Какие этапы критичны?

**💡 Подсказка к ответу:**
```yaml
# Пример GitHub Actions для TBD
name: TBD Pipeline
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Run tests
        run: npm test
      
      - name: Check code coverage
        run: npm run coverage
      
      - name: Lint code
        run: npm run lint

  deploy:
    if: github.ref == 'refs/heads/main'
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to staging
        run: deploy-to-staging.sh
```

**✅ Критичные этапы:**
- Быстрые автотесты (< 10 минут)
- Code quality checks
- Автоматический deploy в staging
- Feature flag validation

#### ❓ Вопрос 4.2: Качество кода в TBD
**Вопрос:** Как обеспечить качество кода при частых интеграциях?

**💡 Подсказка к ответу:**
- **Pre-commit hooks** (lint, format)
- **Быстрая code review** (< 2 часов)
- **Comprehensive test suite** (unit, integration, e2e)
- **Static analysis tools** (SonarQube, ESLint)
- **Pair programming** или mob programming

---

### 5. Сложные сценарии (7-10 минут)

#### ❓ Вопрос 5.1: Восстановление после ошибок
**Вопрос:** В main попал багованный коммит, который сломал production. Как быстро исправить в TBD?

**💡 Подсказка к ответу:**
```bash
# Опция 1: Revert (предпочтительно в TBD)
git revert <bad-commit-hash>
git push origin main

# Опция 2: Hotfix через feature flag
# Быстро отключить проблемную функцию

# Опция 3: Forward fix
# Быстрый коммит с исправлением
```

**🎯 В TBD предпочтителен "fix forward" подход**

#### ❓ Вопрос 5.2: Управление релизами
**Вопрос:** Как управлять релизами в TBD без release branches?

**💡 Подсказка к ответу:**
- **Continuous deployment** из main
- **Feature flags** для управления фичами
- **Blue-green deployment**
- **Canary releases**
- **Tags** для маркировки релизов

**🔍 Стратегии релиза:**
```bash
# Тегирование релиза
git tag -a v1.2.3 -m "Release v1.2.3"
git push origin v1.2.3

# Release branch только если необходимо
git checkout -b release/v1.2.3
# cherry-pick только critical fixes
```

#### ❓ Вопрос 5.3: Мониторинг и откат
**Вопрос:** Как организовать мониторинг и быстрый откат в TBD environment?

**💡 Подсказка к ответу:**
- **Real-time monitoring** (метрики, логи, алерты)
- **Health checks** в каждом сервисе
- **Automated rollback** при падении метрик
- **Circuit breakers** для критичных сервисов

---

## 🎯 Критерии оценки

### ⭐ Junior (1-2 года опыта)
- Понимает основы Git
- Знает базовые команды
- Понимает концепцию TBD
- Может работать с простыми конфликтами

### ⭐⭐ Middle (2-4 года опыта)
- Глубоко понимает TBD vs GitFlow
- Уверенно использует rebase, cherry-pick
- Понимает CI/CD интеграцию
- Может настроить простые автоматизации

### ⭐⭐⭐ Senior (4+ лет опыта)
- Проектирует Git workflow для команды
- Внедряет best practices TBD
- Настраивает сложные CI/CD pipeline
- Решает архитектурные проблемы версионного контроля

---

## 📚 Дополнительные ресурсы

### Документация 2025
- [Trunk-Based Development](https://trunkbaseddevelopment.com/) - официальный сайт
- [Atlassian TBD Guide](https://www.atlassian.com/continuous-delivery/continuous-integration/trunk-based-development) - практические рекомендации
- [AWS Git Best Practices](https://docs.aws.amazon.com/prescriptive-guidance/latest/choosing-git-branch-approach/best-practices-for-git-based-development.html) - enterprise подход

### Ключевые практики 2025
Современные команды все чаще переходят на TBD для поддержки CI/CD и быстрой доставки ПО. Основной фокус - на коротких циклах интеграции и автоматизации.

### Инструменты
- **Feature Flags:** LaunchDarkly, Split.io, ConfigCat
- **CI/CD:** GitHub Actions, GitLab CI, Jenkins, CircleCI
- **Code Quality:** SonarQube, CodeClimate, DeepCode

---

## ⚡ Быстрая шпаргалка команд

```bash
# Ежедневный TBD workflow
git pull origin main                    # Синхронизация
git checkout -b feature/short-task      # Короткая ветка
git add -p && git commit -m "msg"      # Атомарный коммит
git rebase main                        # Подготовка к merge
git push origin feature/short-task     # Push для PR
# После merge в main через PR
git checkout main && git pull origin main
git branch -d feature/short-task       # Удаление ветки

# Экстренные ситуации
git revert HEAD                        # Откат последнего коммита
git reflog                            # Восстановление потерянных коммитов
git reset --hard HEAD~1               # Жёсткий откат (локально)
```