# Git и Trunk-based Development: Полный курс от новичка до эксперта
## Структурированное руководство для backend разработчика

---

## 🎯 **Введение: Почему Git — это не просто инструмент, а философия разработки**

### Эволюция контроля версий

```
1970s-1980s: Ручное копирование файлов
    project_v1.zip
    project_v2_final.zip  
    project_v2_final_REALLY_FINAL.zip
    ↓ Проблемы: потеря данных, конфликты, хаос

1990s: Централизованные системы (CVS, SVN)
    [Dev1] ←→ [Central Server] ←→ [Dev2]
    ↓ Проблемы: единая точка отказа, медленная работа

2005: Git — распределенная система
    [Dev1 + History] ←→ [GitHub] ←→ [Dev2 + History]
    ✅ Решения: локальная история, быстрота, надежность
```

### Цели и результаты курса

**Уровень 1 (Блоки 1-2): Git Practitioner**
- Уверенная работа с локальным репозиторием
- Понимание архитектуры Git
- Решение 90% повседневных задач

**Уровень 2 (Блоки 3-4): GitHub Professional**  
- Эффективная командная работа
- Продвинутый code review
- Настройка CI/CD процессов

**Уровень 3 (Блоки 5-6): Trunk-based Expert**
- Современные практики разработки
- Архитектурное мышление
- Оптимизация workflow команды

**Уровень 4 (Блоки 7-8): Git Architect**
- Проектирование Git стратегий
- Менторство команды
- Решение сложных проблем

---

# **БЛОК 1: Git основы и архитектура**

## **Глава 1: Архитектура Git — Как это работает под капотом**

### Модель данных Git: Снимки против дельт

**Традиционные VCS (SVN, CVS):**
```
Файл A:  Версия 1 → [Дельта 1] → [Дельта 2] → [Дельта 3]
Файл B:  Версия 1 → [Дельта 1] → [Дельта 2] → [Дельта 3]
```

**Git модель:**
```
Коммит 1: [Снимок A1] [Снимок B1] [Снимок C1]
Коммит 2: [Снимок A2] [Ссылка B1] [Снимок C2]  
Коммит 3: [Ссылка A2] [Снимок B3] [Ссылка C2]
```

**Почему снимки лучше:**
- **Скорость** — не нужно вычислять дельты
- **Надежность** — каждый коммит самодостаточен
- **Простота** — легче понять и отладить

### Установка и первоначальная настройка

```bash
# Проверяем версию Git (минимум 2.23+)
git --version

# Глобальная настройка (делается один раз)
git config --global user.name "Иван Иванов"
git config --global user.email "ivan.ivanov@company.com"

# Настройка редактора по умолчанию
git config --global core.editor "code --wait"  # VS Code

# Настройка автопереносов строк (важно для команды!)
git config --global core.autocrlf input      # Linux/Mac

# Красивый вывод логов
git config --global alias.lg "log --oneline --graph --decorate --all"

# Проверяем настройки
git config --list
```

### Анатомия Git репозитория

```bash
# Создаем новый проект
mkdir my-backend-project
cd my-backend-project

# Инициализируем Git репозиторий
git init

# Смотрим что создалось
ls -la
# .git/ — здесь вся магия Git
```

**Структура .git директории:**
```
.git/
├── objects/     # База данных всех объектов (коммиты, файлы, деревья)
├── refs/        # Ссылки на коммиты (ветки, теги)
├── HEAD         # Указатель на текущую ветку
├── index        # Staging area (промежуточная область)
├── config       # Настройки репозитория
└── hooks/       # Скрипты для автоматизации
```

## **Глава 2: Три состояния файлов и базовые операции**

### Три состояния файлов: Основа понимания Git

```
Working Directory  →  Staging Area  →  Repository
   (рабочая)           (подготовка)     (история)
      
Файлы изменены    →  git add  →    git commit
   но не готовы      готовы к        сохранены
   к коммиту         коммиту        в истории
```

**Практический пример:**

```bash
# Создаем первый файл
echo "const express = require('express');" > server.js

# Проверяем статус
git status
# Output: server.js (untracked)

# Добавляем в staging
git add server.js
git status  
# Output: server.js (staged for commit)

# Делаем изменения в файле
echo "const app = express();" >> server.js
git status
# Output: server.js (staged + modified)

# Видим разницу между версиями
git diff              # Working vs Staging
git diff --staged     # Staging vs Repository

# Коммитим staged версию
git commit -m "Добавил базовую структуру Express сервера"

# Добавляем оставшиеся изменения
git add server.js
git commit -m "Добавил создание Express приложения"
```

### История коммитов: Навигация во времени

```bash
# Полная история
git log

# Компактная история  
git log --oneline

# История с графиком веток
git log --oneline --graph --all

# История конкретного файла
git log --oneline server.js

# Кто что менял в файле
git blame server.js

# Детали конкретного коммита
git show <commit-hash>
```

### .gitignore: Что НЕ должно попадать в репозиторий

```bash
# Создаем .gitignore для Node.js проекта
cat > .gitignore << 'EOF'
# Зависимости
node_modules/
npm-debug.log*

# Переменные окружения  
.env
.env.local
.env.production

# Логи
logs/
*.log

# Кэш и временные файлы
.cache/
dist/
build/
.tmp/

# IDE файлы
.vscode/
.idea/
*.swp
*.swo

# OS файлы
.DS_Store
Thumbs.db

# База данных (локальная)
*.sqlite
*.db
EOF
```

---

# **БЛОК 2: Ветвление и слияние**

## **Глава 3: Концепция веток в Git**

### Что такое ветка в Git: Концептуальное понимание

**Ветка = подвижный указатель на коммит**

```
main:     A ← B ← C
               ↑
             HEAD
```

**После создания ветки:**
```
main:     A ← B ← C
               ↑
feature:  A ← B ← C
               ↑
             HEAD
```

**После коммита в feature:**
```
main:     A ← B ← C

feature:  A ← B ← C ← D
                   ↑
                 HEAD
```

### Практическая работа с ветками

```bash
# Просматриваем текущие ветки
git branch
# * main

# Создаем новую ветку (но остаемся на main)
git branch feature/user-authentication

# Переключаемся на новую ветку
git checkout feature/user-authentication
# Или в новых версиях Git:
git switch feature/user-authentication

# Создаем и сразу переключаемся (самый частый случай)
git checkout -b feature/payment-integration
# Или:
git switch -c feature/payment-integration

# Видим все ветки (включая удаленные)
git branch -a
```

## **Глава 4: Разработка в ветках и слияние**

### Разработка в ветке: Изоляция изменений

```bash
# Работаем над аутентификацией
git switch -c feature/user-auth

# Создаем модуль аутентификации
cat > auth.js << 'EOF'
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');

class AuthService {
    async hashPassword(password) {
        return await bcrypt.hash(password, 10);
    }
    
    async comparePassword(password, hash) {
        return await bcrypt.compare(password, hash);
    }
    
    generateToken(userId) {
        return jwt.sign({ userId }, process.env.JWT_SECRET);
    }
}

module.exports = AuthService;
EOF

git add auth.js
git commit -m "Добавил базовый сервис аутентификации"
```

### Слияние веток: Интеграция изменений

```bash
# Возвращаемся на main
git switch main

# Проверяем что main не изменился
git log --oneline
# Наших коммитов нет!

# Сливаем feature ветку в main
git merge feature/user-auth

# Теперь проверяем историю
git log --oneline --graph
# Видим merge commit
```

**Типы слияния в Git:**

**1. Fast-forward merge (простое слияние):**
```
До:   main: A ← B
     feature: A ← B ← C ← D

После: main: A ← B ← C ← D
```

**2. Three-way merge (слияние с merge commit):**
```
До:   main: A ← B ← E
     feature: A ← B ← C ← D

После: main: A ← B ← E ← M
                  ← C ← D ←┘
```

### Стратегии именования веток

**Хорошие примеры:**
```bash
feature/user-authentication     # Новая функциональность
bugfix/login-validation-error   # Исправление бага
hotfix/critical-security-patch  # Критическое исправление
refactor/database-layer         # Рефакторинг кода
docs/api-documentation          # Обновление документации
```

---

# **БЛОК 3: Удаленная работа и синхронизация**

## **Глава 5: Концепция удаленных репозиториев**

### Удаленная работа — Синхронизация с командой

**Локальная разработка:**
```
[Ваш компьютер]
├── Working Directory
├── Staging Area  
└── Local Repository
```

**Командная разработка:**
```
[Разработчик 1]     [GitHub/GitLab]     [Разработчик 2]
Local Repository ←→ Remote Repository ←→ Local Repository
                       (origin)
```

### Клонирование существующего проекта

```bash
# Клонируем проект
git clone https://github.com/company/backend-api.git
cd backend-api

# Изучаем структуру
git remote -v
# origin  https://github.com/company/backend-api.git (fetch)
# origin  https://github.com/company/backend-api.git (push)

git branch -a
# * main
#   remotes/origin/HEAD -> origin/main
#   remotes/origin/main
#   remotes/origin/feature/user-management

# Получаем все ветки
git fetch origin

# Создаем локальную ветку на основе удаленной
git checkout -b feature/user-management origin/feature/user-management
```

## **Глава 6: Синхронизация и разрешение конфликтов**

### Workflow синхронизации с командой

**Ежедневный цикл работы:**

```bash
# 1. Утром - получаем последние изменения
git switch main
git pull origin main

# 2. Создаем ветку для новой задачи
git switch -c feature/api-rate-limiting

# 3. Разрабатываем функциональность
echo "Rate limiting middleware" > middleware/rateLimiter.js
git add middleware/rateLimiter.js
git commit -m "Добавил middleware для rate limiting"

# 4. Отправляем ветку в GitHub
git push origin feature/api-rate-limiting

# 5. Создаем Pull Request через GitHub UI
```

### Fetch vs Pull: Важная разница

```bash
# git fetch - получает изменения БЕЗ слияния
git fetch origin
git log origin/main    # Просматриваем что изменилось
git merge origin/main  # Сливаем когда готовы

# git pull = git fetch + git merge
git pull origin main   # Получает и сливает за одну команду
```

### Разрешение merge конфликтов

**Конфликт возникает когда:**
- Два разработчика изменили одни и те же строки
- Один разработчик изменил файл, другой его удалил
- Разные изменения в одном участке кода

**Содержимое файла с конфликтом:**
```javascript
const config = {
<<<<<<< HEAD
    port: 3000,
    database: 'mongodb://localhost:27017/myapp'
=======
    port: 8080,  
    database: 'postgresql://localhost:5432/myapp'
>>>>>>> feature/database-change
};
```

**Разрешение конфликта:**
```bash
# 1. Открываем файл и исправляем вручную
const config = {
    port: process.env.PORT || 3000,
    database: process.env.DATABASE_URL || 'mongodb://localhost:27017/myapp'
};

# 2. Отмечаем конфликт как разрешенный
git add config.js

# 3. Завершаем merge
git commit -m "Разрешил конфликт в config.js, добавил environment variables"
```

---

# **БЛОК 4: GitHub Workflow и командная разработка**

## **Глава 7: Pull Requests — Основа качественной разработки**

### Философия Pull Requests

**Традиционная разработка:**
```
Разработчик → Прямой Push в main → Проблемы в продакшене
```

**Современная разработка через PR:**
```
Разработчик → Feature Branch → Pull Request → Code Review → Merge в main
```

**Зачем нужны Pull Requests:**
1. **Качество кода** — обязательный code review перед merge
2. **Передача знаний** — команда изучает все изменения
3. **Документирование** — история причин и контекста изменений
4. **Автоматизация** — интеграция с CI/CD и тестами
5. **Безопасность** — предотвращение вредоносного кода

### Анатомия идеального Pull Request

**1. Правильное именование ветки:**
```bash
# ✅ Хорошие примеры
feature/PROJ-123-user-authentication
bugfix/PROJ-456-memory-leak-fix  
hotfix/PROJ-789-security-patch
refactor/PROJ-321-extract-database-layer

# ❌ Плохие примеры
fix
my-changes
test-branch
john-work
```

**2. Качественное описание PR:**

```markdown
## 📋 Описание задачи
Реализация функциональности сброса пароля для пользователей, которые забыли свой пароль.

## 🎯 Что сделано
- [ ] Добавлен эндпоинт POST /api/auth/forgot-password
- [ ] Добавлен эндпоинт POST /api/auth/reset-password  
- [ ] Реализована отправка email с токеном сброса
- [ ] Добавлена валидация токена сброса пароля
- [ ] Написаны unit тесты для новой функциональности
- [ ] Обновлена API документация

## 🔧 Техническая реализация
- Использован JWT токен с коротким TTL (15 минут)
- Токен одноразовый - инвалидируется после использования
- Email отправляется через существующий EmailService
- Добавлена rate limiting (3 запроса в час на email)

## 🧪 Как тестировать
1. Запустить проект: `npm run dev`
2. Отправить POST запрос на `/api/auth/forgot-password` с email
3. Проверить получение email с токеном
4. Использовать токен для сброса пароля через `/api/auth/reset-password`
5. Убедиться что старый пароль больше не работает

## ⚠️ Важные изменения
- Добавлена новая переменная окружения `RESET_TOKEN_SECRET`
- Требуется настройка SMTP для отправки email
- Изменена схема таблицы users (добавлено поле reset_token_expires)

## 🔗 Связанные задачи
Closes #123
Related to #456
```

## **Глава 8: Code Review и GitHub автоматизация**

### Code Review: Культура качественного кода

**Что проверять в Code Review:**

**1. Архитектура и дизайн:**
```javascript
// ❌ Плохо - нарушение Single Responsibility Principle
class UserController {
    async createUser(req, res) {
        // ... создание пользователя ...
        // ... отправка email ...  
        // ... логирование ...
        // ... аналитика ...
    }
}

// ✅ Хорошо - разделение ответственности
class UserController {
    constructor(userService, emailService, analyticsService) {
        this.userService = userService;
        this.emailService = emailService;
        this.analyticsService = analyticsService;
    }
    
    async createUser(req, res) {
        const user = await this.userService.create(req.body);
        await this.emailService.sendWelcome(user.email);
        this.analyticsService.track('user_created', user.id);
        res.status(201).json(user);
    }
}
```

### GitHub как платформа DevOps

**Базовый CI/CD workflow:**

```yaml
# .github/workflows/ci.yml
name: Continuous Integration

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run linter
      run: npm run lint
      
    - name: Run tests
      run: npm run test:coverage
      
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
```

### Branch Protection Rules: Защита main ветки

**Обязательные настройки для продакшена:**

```
Branch Protection для main:
☑️ Require a pull request before merging
  ☑️ Require approvals: 2
  ☑️ Dismiss stale PR approvals when new commits are pushed
  ☑️ Require review from code owners
  
☑️ Require status checks to pass before merging
  ☑️ Require branches to be up to date before merging
  Required checks:
    - CI Tests (Node 18.x)
    - Security Audit
    - Code Coverage > 80%
    
☑️ Require conversation resolution before merging
☑️ Include administrators (важно!)
☑️ Allow force pushes: Never
☑️ Allow deletions: Never
```

---

# **БЛОК 5: Trunk-based Development — Философия и принципы**

## **Глава 9: Эволюция стратегий ветвления**

### Философия и принципы Trunk-based Development

**1990s-2000s: Централизованные системы**
```
[Developers] → [Central Repository] → [Production]
               (Long development cycles)
```

**2000s-2010s: Git Flow**
```
main ────────────────────────────────────
  │                                     │
  └─ develop ─────────────────────────── ┘
       │                     │
       └─ feature/A ────────┘
       └─ feature/B ──────────────┘
```

**2010s-Present: Trunk-based Development**
```
main ─────●─────●─────●─────●─────●─────
          │     │     │     │     │
    feature/A ──┘     │     │     │
         feature/B ───┘     │     │
              feature/C ────┘     │
                   feature/D ─────┘
```

### Сравнительный анализ подходов

| Аспект | Git Flow | GitHub Flow | Trunk-based |
|--------|----------|-------------|-------------|
| **Сложность** | Высокая (5 типов веток) | Средняя (feature + main) | Низкая (main + короткие feature) |
| **Время интеграции** | Недели/месяцы | Дни/недели | Часы/дни |
| **Риск конфликтов** | Высокий | Средний | Низкий |
| **Скорость доставки** | Медленная | Средняя | Быстрая |
| **Подходит для** | Релизные продукты | Web приложения | SaaS, микросервисы |

### Основные принципы Trunk-based Development

**1. Один главный поток разработки:**
```
main ветка - единственный источник истины
├── Все изменения попадают сюда
├── Всегда готова к релизу
└── Защищена от прямых коммитов
```

**2. Короткие feature ветки:**
```bash
# ✅ Правильно - ветка живет 1-3 дня
git switch -c feature/add-user-validation
# ... несколько коммитов ...
# PR создан и смержен за 1-2 дня

# ❌ Неправильно - ветка живет неделями
git switch -c feature/complete-user-system
# ... десятки коммитов за 2 недели ...
# Большой PR с множественными конфликтами
```

**3. Частые интеграции:**
```
Каждый разработчик интегрируется в main минимум раз в день
├── Утром: синхронизация с main
├── В течение дня: разработка в feature ветке  
└── Вечером: merge в main через PR
```

## **Глава 10: Преимущества и культурные изменения**

### Преимущества Trunk-based для Backend разработки

**1. Быстрая обратная связь:**
```
Традиционный подход:
Разработка (2 недели) → Интеграция (3 дня) → Обнаружение проблем → Исправления

Trunk-based:  
Разработка (1 день) → Интеграция (2 часа) → Быстрое обнаружение → Быстрое исправление
```

**2. Простота управления зависимостями:**
```javascript
// В Git Flow: сложные зависимости между ветками
feature/payment-api (зависит от) → develop
feature/user-auth (зависит от) → develop  
feature/order-system (зависит от payment + auth) → ???

// В Trunk-based: простые зависимости
feature/payment-api → main → feature/user-auth → main → feature/order-system
```

**3. Непрерывная доставка:**
```yaml
# main ветка всегда готова к деплою
on:
  push:
    branches: [main]
jobs:
  deploy:
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to production
        run: ./deploy.sh
```

### Культурные изменения в команде

**Требования к команде:**
1. **Высокий уровень автоматизации** — обязательные тесты и CI
2. **Дисциплина в размере PR** — не более 400 строк кода
3. **Быстрый code review** — максимум 24 часа на ответ
4. **Мониторинг продакшена** — быстрое обнаружение проблем

**Изменения в процессах:**
```
Старый процесс:
Планирование (неделя) → Разработка (2 недели) → Тестирование (неделя) → Релиз

Новый процесс:
Планирование (день) → Разработка (день) → Автотесты (минуты) → Релиз
```

---

# **БЛОК 6: Практические техники Trunk-based Development**

## **Глава 11: Стратегии разбиения больших задач**

### Практические техники Trunk-based Development

**Пример: Переработка системы аутентификации**

**❌ Традиционный подход (одна большая ветка):**
```bash
git switch -c feature/auth-rewrite
# 2 недели разработки...
# 50+ файлов изменено
# 2000+ строк кода
# Множественные конфликты при merge
```

**✅ Trunk-based подход (серия маленьких PR):**

```bash
# PR 1: Добавляем новую структуру БД (без изменения логики)
git switch -c feature/auth-db-schema
# Миграции для новых таблиц
# НЕ удаляем старые таблицы
git commit -m "Добавил новую схему БД для аутентификации"

# PR 2: Добавляем новые модели (параллельно со старыми)
git switch -c feature/auth-models-v2  
# Новые модели User, Session, Token
# Старые модели остаются работать
git commit -m "Добавил новые модели аутентификации"

# PR 3: Добавляем новые сервисы
git switch -c feature/auth-services-v2
# AuthServiceV2, TokenServiceV2
# Используют новые модели
git commit -m "Добавил новые сервисы аутентификации"

# PR 4: Добавляем новые эндпоинты (под feature flag)
git switch -c feature/auth-endpoints-v2
# /api/v2/auth/* эндпоинты
# Скрыты за feature flag
git commit -m "Добавил новые эндпоинты аутентификации"

# PR 5: Переключаем фронтенд на новые эндпоинты  
git switch -c feature/switch-to-auth-v2
# Включаем feature flag
# Фронтенд начинает использовать v2
git commit -m "Переключил фронтенд на новую аутентификацию"

# PR 6: Удаляем старый код
git switch -c feature/cleanup-auth-v1
# Удаляем старые эндпоинты, сервисы, модели
# Удаляем старые таблицы БД
git commit -m "Удалил старую систему аутентификации"
```

## **Глава 12: Feature Flags и Backward Compatibility**

### Feature Flags: Техническая реализация

**Простая система feature flags:**

```javascript
// config/featureFlags.js
class FeatureFlags {
    constructor() {
        this.flags = {
            // Статические флаги (из environment)
            newPaymentAPI: process.env.FEATURE_NEW_PAYMENT_API === 'true',
            advancedAnalytics: process.env.FEATURE_ADVANCED_ANALYTICS === 'true',
            
            // Динамические флаги (из БД)
            dynamicFlags: new Map()
        };
        
        this.loadDynamicFlags();
    }
    
    async loadDynamicFlags() {
        const flags = await FeatureFlagModel.findAll();
        flags.forEach(flag => {
            this.dynamicFlags.set(flag.name, flag.enabled);
        });
    }
    
    isEnabled(flagName, userId = null) {
        // Проверяем статический флаг
        if (this.flags.hasOwnProperty(flagName)) {
            return this.flags[flagName];
        }
        
        // Проверяем динамический флаг
        if (this.dynamicFlags.has(flagName)) {
            return this.checkUserSegment(flagName, userId);
        }
        
        return false;
    }
    
    checkUserSegment(flagName, userId) {
        const flag = this.dynamicFlags.get(flagName);
        
        // Полное включение/выключение
        if (typeof flag === 'boolean') {
            return flag;
        }
        
        // Процентное включение
        if (flag.percentage) {
            const hash = this.hashUserId(userId);
            return hash % 100 < flag.percentage;
        }
        
        // Список пользователей
        if (flag.userIds) {
            return flag.userIds.includes(userId);
        }
        
        return false;
    }
    
    hashUserId(userId) {
        // Простая хеш функция для консистентного разбиения
        let hash = 0;
        const str = String(userId);
        for (let i = 0; i < str.length; i++) {
            hash = ((hash << 5) - hash + str.charCodeAt(i)) & 0xffffffff;
        }
        return Math.abs(hash);
    }
}

module.exports = new FeatureFlags();
```

### Backward Compatibility: Безопасные изменения API

**Принципы безопасных изменений:**

**1. Аддитивные изменения (всегда безопасны):**
```javascript
// ✅ Добавление новых полей в ответ
// Старое API
{
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com"
}

// Новое API (backward compatible)
{
    "id": 123,
    "name": "John Doe", 
    "email": "john@example.com",
    "avatar": "https://...",      // новое поле
    "preferences": {...}          // новое поле
}
```

**2. Версионирование через заголовки:**
```javascript
// middleware/apiVersioning.js
function apiVersioning(req, res, next) {
    const version = req.headers['api-version'] || '1.0';
    req.apiVersion = version;
    
    // Устанавливаем версию ответа
    res.setHeader('api-version', version);
    
    next();
}

// controllers/userController.js
async getUser(req, res) {
    const user = await UserService.findById(req.params.id);
    
    if (req.apiVersion >= '2.0') {
        // Возвращаем расширенную версию
        return res.json({
            ...user,
            profile: await UserProfileService.getProfile(user.id),
            preferences: await UserPreferencesService.get(user.id)
        });
    }
    
    // Возвращаем базовую версию
    res.json(user);
}
```

### Database Migrations в Trunk-based подходе

**Многоэтапная миграция для безопасности:**

```sql
-- Этап 1: Добавляем новую колонку (nullable)
-- migration_001_add_user_phone.sql
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
-- ✅ Безопасно: не ломает существующий код

-- Этап 2: Заполняем данные (в отдельном PR)  
-- migration_002_populate_user_phone.sql
UPDATE users 
SET phone = REGEXP_REPLACE(legacy_contact, '[^0-9]', '') 
WHERE phone IS NULL AND legacy_contact IS NOT NULL;
-- ✅ Безопасно: только добавляем данные

-- Этап 3: Делаем колонку обязательной (в отдельном PR)
-- migration_003_make_phone_required.sql  
ALTER TABLE users ALTER COLUMN phone SET NOT NULL;
-- ✅ Безопасно: данные уже заполнены

-- Этап 4: Удаляем старую колонку (в отдельном PR)
-- migration_004_remove_legacy_contact.sql
ALTER TABLE users DROP COLUMN legacy_contact;
-- ✅ Безопасно: код уже не использует старую колонку
```

---

# **БЛОК 7: Настройка полноценного проекта**

## **Глава 13: Архитектура современного backend проекта**

### Практическое применение — Настройка полноценного Trunk-based проекта

**Структура репозитория:**
```
backend-api/
├── .github/                          # GitHub конфигурации
│   ├── workflows/                    # CI/CD пайплайны
│   │   ├── ci.yml                   # Основной CI
│   │   ├── deploy.yml               # Деплой workflow
│   │   └── pr-checks.yml            # Дополнительные проверки PR
│   ├── ISSUE_TEMPLATE/              # Шаблоны для Issues
│   │   ├── bug_report.md
│   │   ├── feature_request.md
│   │   └── config.yml
│   ├── pull_request_template.md     # Шаблон для PR
│   └── CODEOWNERS                   # Автоматические ревьюеры
├── src/                             # Исходный код
│   ├── controllers/                 # HTTP контроллеры
│   ├── services/                    # Бизнес логика
│   ├── models/                      # Модели данных
│   ├── middleware/                  # Express middleware
│   ├── routes/                      # Маршрутизация
│   ├── config/                      # Конфигурации
│   ├── utils/                       # Утилиты
│   └── app.js                       # Главный файл приложения
├── tests/                           # Тесты
│   ├── unit/                        # Unit тесты
│   ├── integration/                 # Интеграционные тесты
│   ├── e2e/                         # End-to-end тесты
│   └── fixtures/                    # Тестовые данные
├── migrations/                      # Database миграции
├── docs/                           # Документация
│   ├── api/                        # API документация
│   ├── deployment/                 # Инструкции по деплою
│   └── development/                # Руководство для разработчиков
├── docker/                         # Docker конфигурации
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── docker-compose.prod.yml
├── scripts/                        # Скрипты автоматизации
│   ├── setup.sh                    # Настройка окружения
│   ├── test.sh                     # Запуск тестов
│   └── deploy.sh                   # Скрипт деплоя
├── .env.example                    # Пример переменных окружения
├── .gitignore                      # Git ignore правила
├── .eslintrc.js                    # ESLint конфигурация
├── .prettierrc                     # Prettier конфигурация
├── jest.config.js                  # Jest конфигурация
├── package.json                    # NPM dependencies
├── README.md                       # Документация проекта
└── CHANGELOG.md                    # История изменений
```

### Инициализация проекта с современными практиками

```bash
# Создаем новый проект
mkdir backend-api && cd backend-api
git init

# Настраиваем базовую структуру
mkdir -p src/{controllers,services,models,middleware,routes,config,utils}
mkdir -p tests/{unit,integration,e2e,fixtures}
mkdir -p .github/{workflows,ISSUE_TEMPLATE}
mkdir -p docs/{api,deployment,development}
mkdir -p docker scripts migrations
```

## **Глава 14: Конфигурация качества кода и инструментов**

### Конфигурация качества кода

**ESLint конфигурация (.eslintrc.js):**
```javascript
module.exports = {
  env: {
    node: true,
    es2022: true,
    jest: true
  },
  extends: [
    'eslint:recommended'
  ],
  parserOptions: {
    ecmaVersion: 2022,
    sourceType: 'module'
  },
  rules: {
    // Качество кода
    'no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    'no-console': ['warn', { allow: ['warn', 'error'] }],
    'prefer-const': 'error',
    'no-var': 'error',
    
    // Стиль кода
    'indent': ['error', 2],
    'quotes': ['error', 'single'],
    'semi': ['error', 'always'],
    'comma-dangle': ['error', 'never'],
    
    // Безопасность
    'no-eval': 'error',
    'no-implied-eval': 'error',
    'no-new-func': 'error',
    
    // Производительность
    'no-await-in-loop': 'warn',
    'require-atomic-updates': 'error'
  },
  overrides: [
    {
      files: ['tests/**/*.js'],
      rules: {
        'no-console': 'off' // Разрешаем console.log в тестах
      }
    }
  ]
};
```

**Jest конфигурация (jest.config.js):**
```javascript
module.exports = {
  testEnvironment: 'node',
  roots: ['<rootDir>/src', '<rootDir>/tests'],
  testMatch: [
    '**/tests/**/*.test.js',
    '**/tests/**/*.spec.js'
  ],
  collectCoverageFrom: [
    'src/**/*.js',
    '!src/app.js', // Исключаем entry point
    '!src/config/**', // Исключаем конфигурации
    '!**/node_modules/**'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  setupFilesAfterEnv: ['<rootDir>/tests/setup.js'],
  testTimeout: 10000,
  verbose: true
};
```

### Git hooks с Husky

```bash
# Инициализируем Husky
npx husky install
npm pkg set scripts.prepare="husky install"

# Добавляем pre-commit хук
npx husky add .husky/pre-commit "npx lint-staged"

# Добавляем commit-msg хук для проверки формата коммитов
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit ${1}'
```

### Система Feature Flags

```javascript
// src/config/featureFlags.js
class FeatureFlags {
  constructor() {
    this.flags = new Map();
    this.loadFlags();
  }

  loadFlags() {
    // Загружаем флаги из переменных окружения
    const envFlags = {
      // API функциональность
      NEW_USER_API: process.env.FEATURE_NEW_USER_API === 'true',
      ADVANCED_AUTH: process.env.FEATURE_ADVANCED_AUTH === 'true',
      PAYMENT_SYSTEM: process.env.FEATURE_PAYMENT_SYSTEM === 'true',
      
      // Мониторинг и аналитика
      DETAILED_LOGGING: process.env.FEATURE_DETAILED_LOGGING === 'true',
      PERFORMANCE_METRICS: process.env.FEATURE_PERFORMANCE_METRICS === 'true',
      
      // Экспериментальные функции
      EXPERIMENTAL_CACHE: process.env.FEATURE_EXPERIMENTAL_CACHE === 'true',
      BETA_FEATURES: process.env.FEATURE_BETA_FEATURES === 'true'
    };

    // Сохраняем в Map для быстрого доступа
    Object.entries(envFlags).forEach(([key, value]) => {
      this.flags.set(key, value);
    });
  }

  isEnabled(flagName, context = {}) {
    // Проверяем базовый флаг
    const baseFlag = this.flags.get(flagName);
    if (!baseFlag) {
      return false;
    }

    // Дополнительная логика для контекстных флагов
    if (context.userId && flagName === 'BETA_FEATURES') {
      // Включаем бета функции только для определенных пользователей
      const betaUsers = process.env.BETA_USER_IDS?.split(',') || [];
      return betaUsers.includes(String(context.userId));
    }

    return baseFlag;
  }

  getAllFlags() {
    return Object.fromEntries(this.flags);
  }
}

module.exports = new FeatureFlags();
```

---

# **БЛОК 8: Командная работа и процессы**

## **Глава 15: GitHub Templates и автоматизация**

### GitHub Templates для стандартизации

**Шаблон Pull Request (.github/pull_request_template.md):**
```markdown
## 📋 Описание изменений

<!-- Опишите что было сделано и зачем -->

## 🎯 Тип изменений

- [ ] 🐛 Исправление бага (bugfix)
- [ ] ✨ Новая функциональность (feature)  
- [ ] 💄 Изменения UI/UX (style)
- [ ] ♻️ Рефакторинг кода (refactor)
- [ ] ⚡ Улучшение производительности (perf)
- [ ] ✅ Добавление тестов (test)
- [ ] 📝 Обновление документации (docs)
- [ ] 🔧 Изменения конфигурации (chore)

## 🧪 Как тестировать

<!-- Подробные инструкции по тестированию изменений -->

1. Шаг 1
2. Шаг 2
3. Ожидаемый результат

## ✅ Чеклист

### Разработчик
- [ ] Код прошел self-review
- [ ] Добавлены/обновлены тесты
- [ ] Все тесты проходят локально
- [ ] Обновлена документация (если нужно)
- [ ] Добавлены комментарии для сложной логики

### CI/CD
- [ ] ✅ Все проверки CI проходят
- [ ] ✅ Code coverage не упал
- [ ] ✅ Security audit проходит
- [ ] ✅ PR size < 400 строк

## 🔗 Связанные задачи

<!-- Используйте ключевые слова для автоматического закрытия Issues -->
Closes #123
Related to #456

## ⚠️ Breaking Changes

<!-- Опишите изменения, которые могут сломать существующую функциональность -->

- [ ] Нет breaking changes
- [ ] Есть breaking changes (описаны ниже)

## 📝 Дополнительные заметки

<!-- Любая дополнительная информация для ревьюеров -->
```

### CODEOWNERS для автоматического review

**.github/CODEOWNERS:**
```bash
# Глобальные владельцы - получают уведомления о всех PR
* @team-leads @senior-backend-devs

# Backend архитектура
/src/controllers/ @backend-team @api-team
/src/services/ @backend-team @business-logic-team  
/src/models/ @backend-team @database-team

# База данных
/migrations/ @database-team @senior-backend-devs
/src/config/database.js @database-team

# Безопасность  
/src/auth/ @security-team @backend-team
/src/middleware/security/ @security-team
/src/middleware/auth/ @security-team @backend-team

# DevOps и инфраструктура
/docker/ @devops-team @team-leads
/.github/ @devops-team @team-leads
/scripts/ @devops-team
Dockerfile @devops-team

# API документация
/docs/api/ @api-team @technical-writers
swagger.json @api-team

# Конфигурации
/src/config/ @senior-backend-devs @devops-team
package.json @team-leads
.eslintrc.js @code-quality-team
jest.config.js @qa-team

# Тесты
/tests/ @qa-team @backend-team
/tests/integration/ @qa-team @backend-team @devops-team
/tests/e2e/ @qa-team @frontend-team

# Feature flags
/src/config/featureFlags.js @product-team @team-leads

# Критически важные файлы (требуют 2+ approvals)
README.md @team-leads
CHANGELOG.md @team-leads  
.env.example @devops-team @team-leads
```

## **Глава 16: Автоматизация качества и процедуры экстренных ситуаций**

### Автоматизация качества PR

**GitHub Action для проверки размера PR:**
```yaml
# .github/workflows/pr-quality-checks.yml
name: PR Quality Checks

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  pr-size-check:
    runs-on: ubuntu-latest
    name: Check PR Size
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
        
    - name: Get PR size
      id: pr-size
      run: |
        # Подсчитываем изменения
        ADDED=$(git diff --numstat origin/${{ github.base_ref }}...HEAD | awk '{sum += $1} END {print sum}')
        DELETED=$(git diff --numstat origin/${{ github.base_ref }}...HEAD | awk '{sum += $2} END {print sum}')
        FILES=$(git diff --name-only origin/${{ github.base_ref }}...HEAD | wc -l)
        TOTAL=$((ADDED + DELETED))
        
        echo "added=$ADDED" >> $GITHUB_OUTPUT
        echo "deleted=$DELETED" >> $GITHUB_OUTPUT
        echo "files=$FILES" >> $GITHUB_OUTPUT
        echo "total=$TOTAL" >> $GITHUB_OUTPUT
        
    - name: Comment PR size
      uses: actions/github-script@v7
      with:
        script: |
          const { added, deleted, files, total } = ${{ steps.pr-size.outputs }};
          
          let status = '✅';
          let message = 'PR size is optimal';
          
          if (total > 800) {
            status = '❌';
            message = 'PR is too large! Consider splitting into smaller PRs';
          } else if (total > 400) {
            status = '⚠️';  
            message = 'PR is getting large. Consider if it can be split';
          }
          
          const body = `## 📊 PR Size Analysis ${status}
          
          **${message}**
          
          | Metric | Count | Status |
          |--------|-------|--------|
          | Files changed | ${files} | ${files > 20 ? '⚠️' : '✅'} |
          | Lines added | ${added} | ${added > 300 ? '⚠️' : '✅'} |
          | Lines deleted | ${deleted} | ✅ |
          | Total changes | ${total} | ${total > 400 ? (total > 800 ? '❌' : '⚠️') : '✅'} |
          
          ### Guidelines:
          - 🎯 **Optimal**: < 400 lines changed
          - ⚠️ **Acceptable**: 400-800 lines changed  
          - ❌ **Too large**: > 800 lines changed
          `;
          
          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: body
          });
```

### Стратегия откатов и hotfixes

**Процедура экстренного hotfix:**
```bash
#!/bin/bash
# scripts/hotfix-procedure.sh

set -e

ISSUE_NUMBER=$1
DESCRIPTION=$2

if [ -z "$ISSUE_NUMBER" ] || [ -z "$DESCRIPTION" ]; then
    echo "Usage: ./hotfix-procedure.sh <issue-number> <description>"
    echo "Example: ./hotfix-procedure.sh 123 'Fix critical memory leak'"
    exit 1
fi

echo "🚨 Starting hotfix procedure for issue #$ISSUE_NUMBER"

# 1. Убеждаемся что мы на актуальном main
git checkout main
git pull origin main

# 2. Создаем hotfix ветку
BRANCH_NAME="hotfix/ISSUE-$ISSUE_NUMBER-$(echo $DESCRIPTION | tr ' ' '-' | tr '[:upper:]' '[:lower:]')"
git checkout -b "$BRANCH_NAME"

echo "✅ Created hotfix branch: $BRANCH_NAME"
echo ""
echo "Next steps:"
echo "1. Make your critical fix"
echo "2. Commit with: git commit -m 'hotfix: $DESCRIPTION'"
echo "3. Push: git push origin $BRANCH_NAME"
echo "4. Create PR with [HOTFIX] label"
echo "5. After merge, tag release: git tag v1.x.x"
echo ""
echo "⚠️  Remember:"
echo "- Keep changes minimal"
echo "- Add tests if possible"  
echo "- Update CHANGELOG.md"
echo "- Notify team in #critical-fixes channel"
```

**Автоматический откат через GitHub Actions:**
```yaml
# .github/workflows/auto-rollback.yml
name: Auto Rollback on Critical Failure

on:
  deployment_status:
    types: [failure]

jobs:
  rollback:
    if: github.event.deployment_status.state == 'failure'
    runs-on: ubuntu-latest
    environment: production
    
    steps:
    - name: Get previous successful deployment
      id: get-previous
      uses: actions/github-script@v7
      with:
        script: |
          const deployments = await github.rest.repos.listDeployments({
            owner: context.repo.owner,
            repo: context.repo.repo,
            environment: 'production'
          });
          
          const successful = deployments.data.find(d => 
            d.sha !== context.payload.deployment.sha &&
            d.environment === 'production'
          );
          
          return successful?.sha || null;
          
    - name: Trigger rollback
      if: steps.get-previous.outputs.result
      run: |
        echo "Rolling back to SHA: ${{ steps.get-previous.outputs.result }}"
        
        # Здесь код для отката деплоя
        # Например, откат в Kubernetes:
        # kubectl rollout undo deployment/backend-api
        
    - name: Create rollback issue
      uses: actions/github-script@v7
      with:
        script: |
          github.rest.issues.create({
            owner: context.repo.owner,
            repo: context.repo.repo,
            title: `🚨 Automatic Rollback Triggered - ${new Date().toISOString()}`,
            body: `## Rollback Summary
            
            **Trigger**: Deployment failure in production
            **Failed SHA**: ${context.payload.deployment.sha}
            **Rolled back to**: ${{ steps.get-previous.outputs.result || 'Unknown' }}
            **Time**: ${new Date().toISOString()}
            
            ## Next Steps
            - [ ] Investigate failure cause
            - [ ] Fix identified issues  
            - [ ] Create hotfix if needed
            - [ ] Re-deploy when ready
            
            ## Monitoring
            - Check application logs
            - Verify service health
            - Monitor error rates
            `,
            labels: ['critical', 'rollback', 'production']
          });
```

---

# **🎓 ЗАКЛЮЧЕНИЕ: Путь от новичка к эксперту завершен**

## **Достигнутые навыки по блокам**

### **Блоки 1-2: Git Mastery** ✅
- **Архитектурное понимание**: Как Git хранит данные и управляет историей
- **Локальная работа**: Уверенное использование всех основных команд
- **Ветвление**: Эффективное создание, слияние и управление ветками
- **Решение конфликтов**: Быстрое и корректное разрешение merge конфликтов

### **Блоки 3-4: GitHub Professional** ✅  
- **Pull Requests**: Создание качественных PR с детальным описанием
- **Code Review**: Конструктивное участие в процессе проверки кода
- **GitHub Actions**: Настройка CI/CD пайплайнов для автоматизации
- **Project Management**: Использование Issues, Projects, Milestones

### **Блоки 5-6: Trunk-based Expert** ✅
- **Философия**: Глубокое понимание принципов trunk-based development
- **Feature Flags**: Техническая реализация и стратегическое использование
- **Backward Compatibility**: Безопасные изменения API и базы данных
- **Непрерывная интеграция**: Частые интеграции без нарушения продакшена

### **Блоки 7-8: Git Architect** ✅
- **Проектирование workflow**: Настройка процессов для команды любого размера
- **Автоматизация качества**: Системы проверок, метрик и мониторинга
- **Процедуры экстренных ситуаций**: Hotfixes, rollbacks, disaster recovery
- **Менторство**: Способность обучать и направлять других разработчиков

## **Практические результаты курса**

### **Готовые решения для использования:**
✅ **Настроенный проект** с современной архитектурой и всеми best practices  
✅ **GitHub templates** для Issues и Pull Requests  
✅ **CI/CD пайплайны** для автоматизации тестирования и деплоя  
✅ **Branch protection rules** для защиты продакшена  
✅ **Feature flags система** для безопасного выкатывания функциональности  
✅ **Процедуры hotfix** для экстренных ситуаций  

### **Навыки командной работы:**
✅ **Эффективная коммуникация** через PR описания и code review  
✅ **Планирование задач** с разбиением на маленькие итерации  
✅ **Качественное документирование** изменений и процессов  
✅ **Проактивный подход** к предотвращению проблем  

## **Влияние на карьеру**

### **Immediate Impact (сразу после курса):**
- **Повышение продуктивности** — быстрее решение повседневных задач с Git
- **Снижение стресса** — уверенность в работе с любыми Git ситуациями  
- **Улучшение качества кода** — систематический подход к review и тестированию

### **Medium-term Benefits (3-6 месяцев):**
- **Лидерство в команде** — становитесь go-to экспертом по Git вопросам
- **Процессные улучшения** — можете предложить и внедрить лучшие практики
- **Менторские возможности** — помощь junior разработчикам в освоении Git

### **Long-term Career Growth (6+ месяцев):**
- **Tech Lead готовность** — способность спроектировать и настроить workflow для команды
- **DevOps компетенции** — понимание CI/CD и автоматизации процессов
- **Архитектурное мышление** — видение полного цикла разработки от кода до продакшена

**Поздравляем!** За 8 блоков и 16 глав вы прошли путь от новичка до эксперта Git и trunk-based development. Это **философия профессиональной разработки**, которая делает вас ценным участником любой команды.

**Вы готовы:**
- Настроить Git workflow в любой команде
- Менторить других разработчиков
- Предлагать и внедрять процессные улучшения
- Быть техническим лидером в области качества кода

**Удачи в вашем дальнейшем профессиональном развитии! 🚀**