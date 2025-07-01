# Git и Trunk-based Development: Полный курс от новичка до эксперта
## Комплексное руководство для backend разработчика | 8 недель

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

### Почему именно Git победил?

**Статистика индустрии:**
- **87%** всех проектов на GitHub используют Git
- **100%** топ-100 IT компаний используют Git
- **Средняя зарплата** разработчика, владеющего Git на продвинутом уровне, на **25% выше**

**Философские принципы Git:**
1. **Целостность данных** — невозможно потерять информацию незаметно
2. **Распределенность** — каждый разработчик имеет полную историю
3. **Ветвление как основа** — создание веток занимает миллисекунды
4. **Неизменяемость истории** — то что закоммичено, остается навсегда

---

## 🎓 **Цели и результаты курса**

### Что вы получите через 8 недель:

**Уровень 1 (Недели 1-2): Git Practitioner**
- Уверенная работа с локальным репозиторием
- Понимание архитектуры Git
- Решение 90% повседневных задач

**Уровень 2 (Недели 3-4): GitHub Professional**  
- Эффективная командная работа
- Продвинутый code review
- Настройка CI/CD процессов

**Уровень 3 (Недели 5-6): Trunk-based Expert**
- Современные практики разработки
- Архитектурное мышление
- Оптимизация workflow команды

**Уровень 4 (Недели 7-8): Git Architect**
- Проектирование Git стратегий
- Менторство команды
- Решение сложных проблем

---

# **НЕДЕЛЯ 1-2: Git основы — Фундамент профессиональной разработки**

## **🎯 Цель недель:** Понять архитектуру Git и научиться работать локально

---

## **День 1-3: Архитектура Git — Как это работает под капотом**

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
# git config --global core.editor "vim"        # Vim

# Настройка автопереносов строк (важно для команды!)
git config --global core.autocrlf input      # Linux/Mac
# git config --global core.autocrlf true     # Windows

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

# Теперь создаем файлы и проверяем
mkdir node_modules
echo "test" > node_modules/test.js
echo "SECRET_KEY=123" > .env

git status
# .env и node_modules не показываются!
```

**Распространенная ошибка новичков:**
```bash
# ❌ НЕПРАВИЛЬНО - коммитим всё подряд
git add .
git commit -m "Добавил всё"

# ✅ ПРАВИЛЬНО - осознанно выбираем файлы  
git add server.js package.json
git commit -m "Добавил сервер и зависимости"
```

---

## **День 4-7: Ветвление — Суперсила Git для параллельной разработки**

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

# Добавляем маршруты
cat > routes/auth.js << 'EOF'
const express = require('express');
const AuthService = require('../auth');

const router = express.Router();
const authService = new AuthService();

router.post('/register', async (req, res) => {
    // Логика регистрации
});

router.post('/login', async (req, res) => {
    // Логика входа
});

module.exports = router;
EOF

mkdir -p routes
git add routes/auth.js
git commit -m "Добавил маршруты аутентификации"

# Проверяем историю ветки
git log --oneline
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

### Удаление веток: Наведение порядка

```bash
# Удаляем локальную ветку (после merge)
git branch -d feature/user-auth

# Принудительное удаление (если не было merge)
git branch -D feature/old-experiment

# Просматриваем оставшиеся ветки
git branch
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

**Плохие примеры:**
```bash
fix                 # Слишком общее
my-branch          # Не информативное  
test123            # Временное название
ivan-work          # Привязка к человеку
```

---

## **День 8-14: Удаленная работа — Синхронизация с командой**

### Концепция удаленных репозиториев

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

### Связывание локального репозитория с удаленным

```bash
# Если создали репозиторий локально
git init
git add .
git commit -m "Первый коммит"

# Добавляем связь с GitHub
git remote add origin https://github.com/username/my-project.git

# Отправляем код в GitHub
git push -u origin main
# -u устанавливает tracking между local main и origin/main
```

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

**Безопасная практика:**
```bash
# ✅ РЕКОМЕНДУЕТСЯ - контролируем процесс
git fetch origin
git log --oneline HEAD..origin/main  # Смотрим что изменилось
git merge origin/main

# ⚠️ ОСТОРОЖНО - автоматическое слияние
git pull origin main
```

### Разрешение merge конфликтов

**Конфликт возникает когда:**
- Два разработчика изменили одни и те же строки
- Один разработчик изменил файл, другой его удалил
- Разные изменения в одном участке кода

**Пример конфликта:**

```bash
# Разработчик A изменил config.js:
const config = {
    port: 3000,
    database: 'mongodb://localhost:27017/myapp'
};

# Разработчик B изменил config.js:
const config = {
    port: 8080,
    database: 'postgresql://localhost:5432/myapp'  
};

# При попытке merge Git покажет:
git merge feature/database-change
# Auto-merging config.js
# CONFLICT (content): Merge conflict in config.js
```

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

**Инструменты для разрешения конфликтов:**
```bash
# Настройка merge tool
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# Использование merge tool
git mergetool
```

### Продвинутые техники работы с remote

```bash
# Несколько удаленных репозиториев
git remote add upstream https://github.com/original/project.git
git remote add fork https://github.com/myusername/project.git

# Синхронизация с upstream
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

# Переименование remote
git remote rename origin gitlab
git remote rename upstream origin

# Изменение URL remote
git remote set-url origin https://github.com/newusername/project.git
```

---

## **🎯 Результат недель 1-2:**

**Технические навыки:**
✅ Понимание архитектуры Git на концептуальном уровне  
✅ Уверенная работа с локальным репозиторием  
✅ Создание и управление ветками  
✅ Синхронизация с удаленными репозиториями  
✅ Разрешение merge конфликтов  

**Практические результаты:**
✅ Настроенная рабочая среда Git  
✅ Локальный проект с историей коммитов  
✅ Связь с GitHub и первые Push/Pull операции  
✅ Опыт работы с ветками и слияниями  

**Следующий шаг:** Переход к командной работе через GitHub workflow

---

# **НЕДЕЛЯ 3-4: GitHub Workflow — Профессиональная командная разработка**

## **🎯 Цель недель:** Освоить современный workflow командной разработки через Pull Requests

---

## **День 15-21: Pull Requests — Основа качественной разработки**

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

**2. Атомарность изменений:**
```bash
# ✅ Одна ветка = одна логическая задача
git switch -c feature/password-reset

# Добавляем ТОЛЬКО функции сброса пароля
git add controllers/passwordReset.js
git add routes/password.js  
git add templates/passwordReset.html
git commit -m "Добавил функциональность сброса пароля"

# ❌ НЕ смешиваем разные задачи в одном PR
# git add userAuth.js passwordReset.js emailService.js database.js
```

**3. Качественное описание PR:**

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

## 📝 Тестовые данные
```json
{
  "email": "test@example.com",
  "newPassword": "newSecurePassword123!"
}
```

## ⚠️ Важные изменения
- Добавлена новая переменная окружения `RESET_TOKEN_SECRET`
- Требуется настройка SMTP для отправки email
- Изменена схема таблицы users (добавлено поле reset_token_expires)

## 🔗 Связанные задачи
Closes #123
Related to #456

## 📷 Скриншоты/Демо
[Если применимо - добавить скриншоты или GIF с демонстрацией]
```

### Создание Pull Request: Пошаговый процесс

```bash
# 1. Синхронизируемся с main
git switch main
git pull origin main

# 2. Создаем feature ветку
git switch -c feature/PROJ-123-password-reset

# 3. Разрабатываем функциональность
# Создаем контроллер для сброса пароля
cat > controllers/passwordResetController.js << 'EOF'
const jwt = require('jsonwebtoken');
const User = require('../models/User');
const EmailService = require('../services/EmailService');

class PasswordResetController {
    async forgotPassword(req, res) {
        try {
            const { email } = req.body;
            
            const user = await User.findByEmail(email);
            if (!user) {
                // Не раскрываем существование пользователя
                return res.status(200).json({ 
                    message: 'Если email существует, письмо было отправлено' 
                });
            }
            
            // Генерируем токен сброса
            const resetToken = jwt.sign(
                { userId: user.id, type: 'password_reset' },
                process.env.RESET_TOKEN_SECRET,
                { expiresIn: '15m' }
            );
            
            // Сохраняем токен в БД
            await user.setResetToken(resetToken);
            
            // Отправляем email
            await EmailService.sendPasswordReset(email, resetToken);
            
            res.status(200).json({ 
                message: 'Если email существует, письмо было отправлено' 
            });
            
        } catch (error) {
            console.error('Password reset error:', error);
            res.status(500).json({ error: 'Внутренняя ошибка сервера' });
        }
    }
    
    async resetPassword(req, res) {
        try {
            const { token, newPassword } = req.body;
            
            // Валидируем токен
            const decoded = jwt.verify(token, process.env.RESET_TOKEN_SECRET);
            
            if (decoded.type !== 'password_reset') {
                return res.status(400).json({ error: 'Неверный токен' });
            }
            
            const user = await User.findById(decoded.userId);
            if (!user || !user.isResetTokenValid(token)) {
                return res.status(400).json({ error: 'Токен недействителен' });
            }
            
            // Обновляем пароль
            await user.updatePassword(newPassword);
            await user.invalidateResetToken();
            
            res.status(200).json({ message: 'Пароль успешно изменен' });
            
        } catch (error) {
            if (error.name === 'JsonWebTokenError') {
                return res.status(400).json({ error: 'Неверный токен' });
            }
            
            console.error('Password reset error:', error);
            res.status(500).json({ error: 'Внутренняя ошибка сервера' });
        }
    }
}

module.exports = new PasswordResetController();
EOF

# 4. Добавляем маршруты
cat > routes/passwordReset.js << 'EOF'
const express = require('express');
const rateLimit = require('express-rate-limit');
const PasswordResetController = require('../controllers/passwordResetController');

const router = express.Router();

// Rate limiting для защиты от спама
const forgotPasswordLimiter = rateLimit({
    windowMs: 60 * 60 * 1000, // 1 час
    max: 3, // максимум 3 запроса в час
    message: 'Слишком много запросов. Попробуйте позже.'
});

router.post('/forgot-password', 
    forgotPasswordLimiter,
    PasswordResetController.forgotPassword
);

router.post('/reset-password', 
    PasswordResetController.resetPassword
);

module.exports = router;
EOF

# 5. Коммитим изменения
git add controllers/passwordResetController.js routes/passwordReset.js
git commit -m "Добавил контроллер и маршруты для сброса пароля

- Реализован безопасный процесс сброса пароля
- Добавлен rate limiting для защиты от спама  
- Использованы JWT токены с коротким TTL
- Добавлена валидация и обработка ошибок"

# 6. Пушим ветку в GitHub
git push origin feature/PROJ-123-password-reset

# 7. Создаем PR через GitHub UI или CLI
gh pr create --title "Реализация сброса пароля" --body "Подробное описание см. в PR template"
```

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

**2. Безопасность:**
```javascript
// ❌ Опасно - SQL injection
const query = `SELECT * FROM users WHERE email = '${email}'`;

// ✅ Безопасно - параметризованный запрос
const query = 'SELECT * FROM users WHERE email = ?';
const result = await db.query(query, [email]);
```

**3. Производительность:**
```javascript
// ❌ Неэффективно - N+1 запросов
const users = await User.findAll();
for (const user of users) {
    user.posts = await Post.findByUserId(user.id);
}

// ✅ Эффективно - один запрос с JOIN
const users = await User.findAll({
    include: [Post]
});
```

**4. Тестируемость:**
```javascript
// ❌ Сложно тестировать - много зависимостей
async function processPayment(userId, amount) {
    const user = await User.findById(userId);
    const stripe = new Stripe(process.env.STRIPE_KEY);
    const payment = await stripe.charge(amount, user.cardToken);
    await EmailService.sendReceipt(user.email, payment);
    return payment;
}

// ✅ Легко тестировать - внедрение зависимостей
async function processPayment(userId, amount, { userRepository, paymentService, emailService }) {
    const user = await userRepository.findById(userId);
    const payment = await paymentService.charge(amount, user.cardToken);
    await emailService.sendReceipt(user.email, payment);
    return payment;
}
```

### Стратегии слияния Pull Requests

**1. Merge Commit (по умолчанию):**
```
main:     A ← B ← E ← M
             ↗     ↗
feature:    C ← D ←┘

История: A → B → C → D → E → M
```
**Плюсы:** Сохраняет полную историю  
**Минусы:** Сложная история, много merge коммитов

**2. Squash and Merge:**
```
main:     A ← B ← E ← S
               
feature:  C ← D (удаляется)

История: A → B → E → S
```
**Плюсы:** Чистая линейная история  
**Минусы:** Теряется детальная история feature ветки

**3. Rebase and Merge:**
```
main:     A ← B ← E ← C' ← D'

feature:  C ← D (удаляется)

История: A → B → E → C' → D'
```
**Плюсы:** Линейная история с сохранением коммитов  
**Минусы:** Изменяет хеши коммитов

**Рекомендации по выбору стратегии:**
- **Мелкие PR (1-3 коммита):** Squash and Merge
- **Крупные PR с логическими коммитами:** Rebase and Merge  
- **Важные feature ветки:** Merge Commit

---

## **День 22-28: GitHub как платформа DevOps**

### GitHub Issues: Управление задачами и багами

**Создание качественного Issue:**

```markdown
---
name: Bug Report
about: Сообщить о баге в приложении
title: '[BUG] Краткое описание проблемы'
labels: bug, needs-triage
assignees: ''
---

## 🐛 Описание бага
Четкое и краткое описание проблемы.

## 🔄 Шаги для воспроизведения
1. Перейти на страницу '/api/users'
2. Отправить POST запрос с данными '...'
3. Посмотреть на ответ
4. Увидеть ошибку

## ✅ Ожидаемое поведение
Что должно происходить.

## ❌ Фактическое поведение  
Что происходит на самом деле.

## 🖥️ Окружение
- OS: [e.g. Ubuntu 20.04]
- Node.js: [e.g. 18.17.0]
- Database: [e.g. PostgreSQL 14.2]
- Browser [если применимо]: [e.g. Chrome 91]

## 📎 Дополнительный контекст
Логи, скриншоты, и другая информация.

```

**Связывание Issues с Pull Requests:**
```bash
# В описании PR или коммита используем ключевые слова:
git commit -m "Исправил валидацию email

Fixes #123
Closes #456  
Resolves #789"

# GitHub автоматически закроет Issues при merge PR
```

### Автоматизация с GitHub Actions

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
        
  security:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Run security audit
      run: npm audit --audit-level high
      
  build:
    needs: [test, security]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v4
    - name: Build Docker image
      run: |
        docker build -t myapp:${{ github.sha }} .
        docker tag myapp:${{ github.sha }} myapp:latest
```

**Продвинутый workflow с деплоем:**

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1
        
    - name: Deploy to ECS
      run: |
        aws ecs update-service \
          --cluster production \
          --service backend-api \
          --force-new-deployment
        
    - name: Notify Slack
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        channel: '#deployments'
        webhook_url: ${{ secrets.SLACK_WEBHOOK }}
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

**Настройка через GitHub CLI:**
```bash
# Установка базовых правил
gh api repos/:owner/:repo/branches/main/protection \
  --method PUT \
  --field required_status_checks='{"strict":true,"contexts":["ci/tests"]}' \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{"required_approving_review_count":2}' \
  --field restrictions=null
```

### CODEOWNERS: Автоматическое назначение ревьюеров

```bash
# .github/CODEOWNERS
# Глобальные владельцы
* @team-leads @senior-devs

# Backend код
/src/controllers/ @backend-team @security-team
/src/models/ @backend-team @database-team
/migrations/ @database-team

# Инфраструктура
/docker/ @devops-team
/.github/ @devops-team @team-leads
/kubernetes/ @devops-team

# Документация
/docs/ @technical-writers @team-leads
README.md @team-leads

# Безопасность
/src/auth/ @security-team
/src/crypto/ @security-team
```

### Semantic Versioning и Release Management

**Автоматическое создание релизов:**

```yaml
# .github/workflows/release.yml
name: Create Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
        
    - name: Generate changelog
      run: |
        git log $(git describe --tags --abbrev=0 HEAD^)..HEAD \
          --pretty=format:"- %s" > changelog.txt
        
    - name: Create Release
      uses: actions/create-release@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        tag_name: ${{ github.ref }}
        release_name: Release ${{ github.ref }}
        body_path: changelog.txt
        draft: false
        prerelease: false
```

---

## **🎯 Результат недель 3-4:**

**Технические навыки:**
✅ Создание качественных Pull Requests с детальным описанием  
✅ Эффективное участие в Code Review процессах  
✅ Настройка и использование GitHub Actions для CI/CD  
✅ Конфигурирование Branch Protection Rules  
✅ Управление проектом через GitHub Issues  

**Процессные навыки:**
✅ Понимание современного DevOps workflow  
✅ Автоматизация проверок качества кода  
✅ Безопасная работа с продакшен кодом  
✅ Командная разработка с code review культурой  

**Следующий шаг:** Переход к изучению Trunk-based Development для оптимизации workflow

---

# **НЕДЕЛЯ 5-6: Trunk-based Development — Современный подход к ветвлению**

## **🎯 Цель недель:** Освоить философию и практики trunk-based development для ускорения доставки кода

---

## **День 29-35: Философия и принципы Trunk-based Development**

### Эволюция стратегий ветвления

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

**4. Feature Flags вместо долгих веток:**
```javascript
// Вместо долгой ветки используем флаги
function getUserProfile(userId) {
    if (featureFlags.newProfileAPI) {
        return getProfileV2(userId);
    }
    return getProfileV1(userId);
}
```

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

## **День 36-42: Практические техники Trunk-based Development**

### Стратегии разбиения больших задач

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

**Использование feature flags в коде:**

```javascript
// controllers/paymentController.js
const FeatureFlags = require('../config/featureFlags');
const PaymentServiceV1 = require('../services/paymentV1');
const PaymentServiceV2 = require('../services/paymentV2');

class PaymentController {
    async processPayment(req, res) {
        const { userId, amount, cardToken } = req.body;
        
        try {
            let result;
            
            if (FeatureFlags.isEnabled('newPaymentAPI', userId)) {
                // Новая реализация
                result = await PaymentServiceV2.process({
                    userId,
                    amount,
                    cardToken,
                    metadata: req.body.metadata
                });
                
                // Логируем использование новой версии
                console.log(`User ${userId} used Payment API v2`);
            } else {
                // Старая реализация
                result = await PaymentServiceV1.process(userId, amount, cardToken);
            }
            
            res.status(200).json(result);
            
        } catch (error) {
            // При ошибке в новой версии, откатываемся на старую
            if (FeatureFlags.isEnabled('newPaymentAPI', userId) && error.code === 'NEW_API_ERROR') {
                console.warn(`Payment API v2 failed for user ${userId}, falling back to v1`);
                const result = await PaymentServiceV1.process(userId, amount, cardToken);
                res.status(200).json(result);
            } else {
                throw error;
            }
        }
    }
}
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

**3. Deprecation стратегия:**
```javascript
// middleware/deprecationWarning.js
function deprecationWarning(endpoint, newEndpoint, sunsetDate) {
    return (req, res, next) => {
        res.setHeader('Warning', `299 - "Deprecated endpoint. Use ${newEndpoint} instead. Sunset date: ${sunsetDate}"`);
        res.setHeader('Sunset', sunsetDate);
        
        // Логируем использование deprecated API
        console.warn(`Deprecated endpoint ${endpoint} used by ${req.ip}`);
        
        next();
    };
}

// routes/api.js
router.get('/api/v1/users', 
    deprecationWarning('/api/v1/users', '/api/v2/users', '2024-12-31'),
    userController.getUsers
);
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

**Миграции с feature flags:**
```javascript
// migrations/addAdvancedUserMetadata.js
async function up(knex) {
    // Создаем таблицу только если фича включена
    if (process.env.FEATURE_ADVANCED_USER_METADATA === 'true') {
        await knex.schema.createTable('user_metadata', table => {
            table.increments('id');
            table.integer('user_id').references('id').inTable('users');
            table.json('metadata');
            table.timestamps();
        });
    }
}

async function down(knex) {
    await knex.schema.dropTableIfExists('user_metadata');
}
```

### Continuous Integration в Trunk-based

**Обязательные проверки перед merge:**

```yaml
# .github/workflows/trunk-based-ci.yml
name: Trunk-based CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  # Быстрые проверки (должны выполняться < 5 минут)
  fast-checks:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Lint code
      run: npm run lint
      
    - name: Check code formatting  
      run: npm run format:check
      
    - name: Type checking
      run: npm run type-check
      
    - name: Unit tests
      run: npm run test:unit
      timeout-minutes: 3
      
  # Интеграционные тесты (могут быть дольше)
  integration-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
          
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup test database
      run: npm run db:test:setup
      
    - name: Run integration tests
      run: npm run test:integration
      timeout-minutes: 10
      
  # Проверки безопасности
  security:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Security audit
      run: npm audit --audit-level high
      
    - name: Check for secrets
      uses: trufflesecurity/trufflehog@v3.63.2
      with:
        path: ./
        base: main
        head: HEAD
        
  # Проверка размера PR
  pr-size:
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
        
    - name: Check PR size
      run: |
        CHANGED_LINES=$(git diff --numstat origin/main...HEAD | awk '{sum += $1 + $2} END {print sum}')
        echo "Changed lines: $CHANGED_LINES"
        if [ $CHANGED_LINES -gt 400 ]; then
          echo "❌ PR too large! Changed lines: $CHANGED_LINES (max: 400)"
          echo "Please split into smaller PRs"
          exit 1
        fi
        echo "✅ PR size OK: $CHANGED_LINES lines"

  # Автоматический деплой в staging при push в main
  deploy-staging:
    needs: [fast-checks, integration-tests, security]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Deploy to staging
      run: |
        echo "Deploying to staging environment..."
        # Здесь код деплоя
        
    - name: Run smoke tests
      run: npm run test:smoke:staging
```

---

## **🎯 Результат недель 5-6:**

**Концептуальные навыки:**
✅ Глубокое понимание принципов Trunk-based Development  
✅ Умение разбивать большие задачи на маленькие PR  
✅ Навыки работы с Feature Flags  
✅ Знание стратегий Backward Compatibility  

**Технические навыки:**
✅ Реализация системы Feature Flags  
✅ Безопасные Database Migrations  
✅ Настройка CI для Trunk-based workflow  
✅ Мониторинг размера и качества PR  

**Следующий шаг:** Применение всех знаний на практике в реальном проекте

---

# **НЕДЕЛЯ 7-8: Практическое применение — Настройка полноценного Trunk-based проекта**

## **🎯 Цель недель:** Создать с нуля полноценный проект с Trunk-based workflow

---

## **День 43-49: Настройка проекта с нуля**

### Архитектура современного backend проекта

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

# Инициализируем package.json с современными зависимостями
cat > package.json << 'EOF'
{
  "name": "backend-api",
  "version": "1.0.0",
  "description": "Modern backend API with trunk-based development",
  "main": "src/app.js",
  "scripts": {
    "start": "node src/app.js",
    "dev": "nodemon src/app.js",
    "test": "jest",
    "test:unit": "jest tests/unit",
    "test:integration": "jest tests/integration",
    "test:e2e": "jest tests/e2e",
    "test:coverage": "jest --coverage",
    "test:watch": "jest --watch",
    "lint": "eslint src/ tests/",
    "lint:fix": "eslint src/ tests/ --fix",
    "format": "prettier --write src/ tests/",
    "format:check": "prettier --check src/ tests/",
    "type-check": "tsc --noEmit",
    "db:migrate": "knex migrate:latest",
    "db:rollback": "knex migrate:rollback",
    "db:seed": "knex seed:run",
    "db:test:setup": "./scripts/setup-test-db.sh",
    "security:audit": "npm audit --audit-level high",
    "security:check": "snyk test",
    "build": "docker build -t backend-api .",
    "deploy:staging": "./scripts/deploy.sh staging",
    "deploy:production": "./scripts/deploy.sh production"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5",
    "helmet": "^7.0.0",
    "express-rate-limit": "^6.7.0",
    "express-validator": "^7.0.1",
    "bcrypt": "^5.1.0",
    "jsonwebtoken": "^9.0.0",
    "knex": "^2.4.2",
    "pg": "^8.11.0",
    "redis": "^4.6.7",
    "dotenv": "^16.1.4",
    "winston": "^3.9.0",
    "joi": "^17.9.2",
    "uuid": "^9.0.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.22",
    "jest": "^29.5.0",
    "supertest": "^6.3.3",
    "eslint": "^8.42.0",
    "prettier": "^2.8.8",
    "typescript": "^5.1.3",
    "@types/node": "^20.3.1",
    "husky": "^8.0.3",
    "lint-staged": "^13.2.2",
    "snyk": "^1.1156.0"
  },
  "engines": {
    "node": ">=18.0.0",
    "npm": ">=9.0.0"
  },
  "lint-staged": {
    "*.js": ["eslint --fix", "prettier --write"],
    "*.{json,md}": ["prettier --write"]
  }
}
EOF

# Устанавливаем зависимости
npm install
```

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

**Prettier конфигурация (.prettierrc):**
```json
{
  "semi": true,
  "trailingComma": "none", 
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "arrowParens": "avoid"
}
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

# Устанавливаем commitlint
npm install --save-dev @commitlint/cli @commitlint/config-conventional

# Конфигурация commitlint
cat > .commitlintrc.js << 'EOF'
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',     // новая функциональность
        'fix',      // исправление бага
        'docs',     // обновление документации
        'style',    // форматирование кода
        'refactor', // рефакторинг
        'test',     // добавление тестов
        'chore',    // обновление зависимостей, конфигураций
        'perf',     // улучшение производительности
        'ci',       // изменения CI/CD
        'revert'    // откат изменений
      ]
    ],
    'subject-max-length': [2, 'always', 72],
    'subject-case': [2, 'always', 'lower-case']
  }
};
EOF
```

### Настройка базового приложения

**Главный файл приложения (src/app.js):**
```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
require('dotenv').config();

const logger = require('./config/logger');
const errorHandler = require('./middleware/errorHandler');
const requestLogger = require('./middleware/requestLogger');
const featureFlags = require('./config/featureFlags');

// Импорт маршрутов
const authRoutes = require('./routes/auth');
const userRoutes = require('./routes/users');
const healthRoutes = require('./routes/health');

const app = express();

// Базовые middleware
app.use(helmet()); // Безопасность
app.use(cors()); // CORS
app.use(express.json({ limit: '10mb' })); // Парсинг JSON
app.use(express.urlencoded({ extended: true }));

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 минут
  max: 100, // максимум 100 запросов на IP
  message: 'Слишком много запросов с этого IP'
});
app.use(limiter);

// Логирование запросов
app.use(requestLogger);

// API маршруты
app.use('/health', healthRoutes);
app.use('/api/auth', authRoutes);
app.use('/api/users', userRoutes);

// API документация (в development режиме)
if (process.env.NODE_ENV === 'development') {
  const swaggerUi = require('swagger-ui-express');
  const swaggerDocument = require('../docs/api/swagger.json');
  app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument));
}

// 404 для несуществующих маршрутов
app.use('*', (req, res) => {
  res.status(404).json({
    error: 'Маршрут не найден',
    path: req.originalUrl
  });
});

// Обработчик ошибок (должен быть последним)
app.use(errorHandler);

// Запуск сервера
const PORT = process.env.PORT || 3000;
const server = app.listen(PORT, () => {
  logger.info(`Сервер запущен на порту ${PORT}`);
  logger.info(`Окружение: ${process.env.NODE_ENV}`);
  logger.info(`Feature flags: ${JSON.stringify(featureFlags.getAllFlags())}`);
});

// Graceful shutdown
process.on('SIGTERM', () => {
  logger.info('SIGTERM получен, завершаем сервер gracefully');
  server.close(() => {
    logger.info('Сервер завершен');
    process.exit(0);
  });
});

process.on('SIGINT', () => {
  logger.info('SIGINT получен, завершаем сервер gracefully');
  server.close(() => {
    logger.info('Сервер завершен');
    process.exit(0);
  });
});

module.exports = app;
```

**Конфигурация логирования (src/config/logger.js):**
```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { 
    service: 'backend-api',
    version: process.env.npm_package_version || '1.0.0'
  },
  transports: [
    // Файл для ошибок
    new winston.transports.File({ 
      filename: 'logs/error.log', 
      level: 'error',
      maxsize: 5242880, // 5MB
      maxFiles: 5
    }),
    // Файл для всех логов
    new winston.transports.File({ 
      filename: 'logs/combined.log',
      maxsize: 5242880, // 5MB
      maxFiles: 5
    })
  ]
});

// В development режиме логируем в консоль
if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.combine(
      winston.format.colorize(),
      winston.format.simple()
    )
  }));
}

module.exports = logger;
```

**Система Feature Flags (src/config/featureFlags.js):**
```javascript
const logger = require('./logger');

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
      if (value) {
        logger.info(`Feature flag enabled: ${key}`);
      }
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

    if (context.environment && flagName === 'EXPERIMENTAL_CACHE') {
      // Экспериментальный кеш только в staging/development
      return ['staging', 'development'].includes(context.environment);
    }

    return baseFlag;
  }

  getAllFlags() {
    return Object.fromEntries(this.flags);
  }

  // Метод для динамического включения/выключения флагов (для тестов)
  setFlag(flagName, value) {
    this.flags.set(flagName, value);
    logger.info(`Feature flag ${flagName} set to ${value}`);
  }
}

module.exports = new FeatureFlags();
```

### Docker конфигурация

**Dockerfile для production:**
```dockerfile
# Многоэтапная сборка для оптимизации размера
FROM node:18-alpine AS base

# Устанавливаем рабочую директорию
WORKDIR /app

# Копируем package files для кеширования слоев
COPY package*.json ./

# Production dependencies
FROM base AS dependencies
RUN npm ci --only=production && npm cache clean --force

# Development dependencies (для сборки)
FROM base AS build-deps
RUN npm ci

# Сборка приложения
FROM build-deps AS build
COPY . .
RUN npm run lint
RUN npm run test:unit
# Здесь могут быть дополнительные шаги сборки

# Production image
FROM node:18-alpine AS production

# Создаем пользователя для безопасности
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

# Устанавливаем рабочую директорию
WORKDIR /app

# Копируем production dependencies
COPY --from=dependencies --chown=nodejs:nodejs /app/node_modules ./node_modules

# Копируем исходный код
COPY --chown=nodejs:nodejs . .

# Создаем директорию для логов
RUN mkdir -p logs && chown nodejs:nodejs logs

# Устанавливаем пользователя
USER nodejs

# Открываем порт
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node ./scripts/health-check.js

# Запускаем приложение
CMD ["node", "src/app.js"]
```

**Docker Compose для разработки:**
```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: 
      context: .
      target: build-deps  # Используем development зависимости
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://postgres:password@db:5432/backend_api
      - REDIS_URL=redis://redis:6379
      - FEATURE_DETAILED_LOGGING=true
      - FEATURE_EXPERIMENTAL_CACHE=true
    volumes:
      - .:/app
      - /app/node_modules  # Предотвращаем перезапись node_modules
    depends_on:
      - db
      - redis
    command: npm run dev

  db:
    image: postgres:14-alpine
    environment:
      - POSTGRES_DB=backend_api
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./migrations:/docker-entrypoint-initdb.d

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  test-db:
    image: postgres:14-alpine
    environment:
      - POSTGRES_DB=backend_api_test
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - "5433:5432"
    tmpfs:
      - /var/lib/postgresql/data  # Используем память для тестовой БД

volumes:
  postgres_data:
  redis_data:
```

---

## **День 50-56: Командная работа и процессы**

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

## 📷 Скриншоты/Демо

<!-- Если применимо, добавьте скриншоты или GIF -->

## ⚠️ Breaking Changes

<!-- Опишите изменения, которые могут сломать существующую функциональность -->

- [ ] Нет breaking changes
- [ ] Есть breaking changes (описаны ниже)

## 📝 Дополнительные заметки

<!-- Любая дополнительная информация для ревьюеров -->
```

**Шаблон Bug Report (.github/ISSUE_TEMPLATE/bug_report.md):**
```markdown
---
name: 🐛 Bug Report
about: Сообщить о баге в приложении
title: '[BUG] '
labels: ['bug', 'needs-triage']
assignees: ''
---

## 🐛 Описание бага

<!-- Четкое и краткое описание проблемы -->

## 🔄 Шаги для воспроизведения

1. Перейти на '...'
2. Нажать на '...'
3. Прокрутить до '...'
4. Увидеть ошибку

## ✅ Ожидаемое поведение

<!-- Что должно происходить -->

## ❌ Фактическое поведение

<!-- Что происходит на самом деле -->

## 📱 Окружение

**Backend:**
- OS: [e.g. Ubuntu 20.04, macOS 12.0]
- Node.js: [e.g. 18.17.0]
- Database: [e.g. PostgreSQL 14.2]
- Version: [e.g. 1.2.3]

**Client (если применимо):**
- Browser: [e.g. Chrome 115, Safari 16]
- Device: [e.g. iPhone 12, Desktop]

## 📋 Логи

<!-- Вставьте соответствующие логи -->

```
Вставьте логи здесь
```

## 📎 Дополнительный контекст

<!-- Скриншоты, файлы, или другая информация -->

## 🎯 Приоритет

- [ ] 🔥 Критический (блокирует продакшен)
- [ ] ⚠️ Высокий (влияет на основную функциональность)  
- [ ] 📊 Средний (влияет на пользователей)
- [ ] 📝 Низкий (косметические изменения)
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

### Политики качества кода

**Настройка Branch Protection через GitHub CLI:**
```bash
#!/bin/bash
# scripts/setup-branch-protection.sh

REPO="organization/backend-api"
BRANCH="main"

echo "Настройка Branch Protection для $REPO:$BRANCH"

# Базовые правила защиты
gh api repos/$REPO/branches/$BRANCH/protection \
  --method PUT \
  --field required_status_checks='{
    "strict": true,
    "contexts": [
      "ci/lint-and-format",
      "ci/unit-tests", 
      "ci/integration-tests",
      "ci/security-audit",
      "ci/coverage-check",
      "ci/pr-size-check"
    ]
  }' \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{
    "required_approving_review_count": 2,
    "dismiss_stale_reviews": true,
    "require_code_owner_reviews": true,
    "require_last_push_approval": true
  }' \
  --field restrictions=null \
  --field required_conversation_resolution=true \
  --field allow_force_pushes=false \
  --field allow_deletions=false

echo "✅ Branch Protection настроен успешно"

# Дополнительные настройки репозитория
gh api repos/$REPO \
  --method PATCH \
  --field allow_squash_merge=true \
  --field allow_merge_commit=false \
  --field allow_rebase_merge=true \
  --field delete_branch_on_merge=true \
  --field allow_auto_merge=true

echo "✅ Настройки merge стратегий обновлены"
```

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
          
          **Tips for large PRs:**
          - Split into logical commits
          - Create separate PRs for refactoring
          - Use feature flags for incomplete features
          `;
          
          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: body
          });

  code-quality-metrics:
    runs-on: ubuntu-latest
    name: Code Quality Metrics
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
        
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run complexity analysis
      run: |
        npx complexity-report --format json --output complexity.json src/
        
    - name: Analyze code complexity
      uses: actions/github-script@v7
      with:
        script: |
          const fs = require('fs');
          
          try {
            const complexity = JSON.parse(fs.readFileSync('complexity.json', 'utf8'));
            
            const highComplexity = complexity.functions.filter(fn => fn.complexity > 10);
            
            if (highComplexity.length > 0) {
              const body = `## ⚠️ High Complexity Functions Detected
              
              The following functions have high cyclomatic complexity and should be refactored:
              
              ${highComplexity.map(fn => 
                `- **${fn.name}** (${fn.file}:${fn.line}) - Complexity: ${fn.complexity}`
              ).join('\n')}
              
              **Recommendations:**
              - Break down large functions into smaller ones
              - Extract complex logic into separate functions
              - Consider using early returns to reduce nesting
              - Add unit tests for complex functions
              `;
              
              github.rest.issues.createComment({
                issue_number: context.issue.number,
                owner: context.repo.owner,
                repo: context.repo.repo,
                body: body
              });
            }
          } catch (error) {
            console.log('No complexity report generated or parsing failed');
          }

  dependency-check:
    runs-on: ubuntu-latest
    name: Dependency Security Check
    steps:
    - uses: actions/checkout@v4
    
    - name: Run Snyk security check
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      with:
        args: --severity-threshold=medium --fail-on=all
        
    - name: Check for dependency updates
      run: |
        npx npm-check-updates --format json > ncu-output.json
        
    - name: Comment on outdated dependencies
      uses: actions/github-script@v7
      with:
        script: |
          const fs = require('fs');
          
          try {
            const updates = JSON.parse(fs.readFileSync('ncu-output.json', 'utf8'));
            
            if (Object.keys(updates).length > 0) {
              const body = `## 📦 Outdated Dependencies
              
              Consider updating the following dependencies:
              
              ${Object.entries(updates).map(([pkg, version]) => 
                `- **${pkg}**: ${version}`
              ).join('\n')}
              
              Run \`npm update\` to update these dependencies.
              `;
              
              github.rest.issues.createComment({
                issue_number: context.issue.number,
                owner: context.repo.owner,
                repo: context.repo.repo,
                body: body
              });
            }
          } catch (error) {
            console.log('No dependency updates needed or checking failed');
          }
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

## **🎯 Финальный результат недель 7-8:**

**Проектные навыки:**
✅ Настройка современного backend проекта с нуля  
✅ Конфигурация всех инструментов качества кода  
✅ Автоматизация проверок и деплоя  
✅ Документирование процессов и процедур  

**Командные навыки:**
✅ Создание шаблонов для Issues и PR  
✅ Настройка автоматического code review  
✅ Процедуры для экстренных ситуаций  
✅ Мониторинг качества и метрик  

---

# **🎓 ЗАКЛЮЧЕНИЕ: Путь от новичка к эксперту завершен**

## **Достигнутые навыки за 8 недель**

### **Уровень 1: Git Mastery** ✅
- **Архитектурное понимание**: Как Git хранит данные и управляет историей
- **Локальная работа**: Уверенное использование всех основных команд
- **Ветвление**: Эффективное создание, слияние и управление ветками
- **Решение конфликтов**: Быстрое и корректное разрешение merge конфликтов

### **Уровень 2: GitHub Professional** ✅  
- **Pull Requests**: Создание качественных PR с детальным описанием
- **Code Review**: Конструктивное участие в процессе проверки кода
- **GitHub Actions**: Настройка CI/CD пайплайнов для автоматизации
- **Project Management**: Использование Issues, Projects, Milestones

### **Уровень 3: Trunk-based Expert** ✅
- **Философия**: Глубокое понимание принципов trunk-based development
- **Feature Flags**: Техническая реализация и стратегическое использование
- **Backward Compatibility**: Безопасные изменения API и базы данных
- **Непрерывная интеграция**: Частые интеграции без нарушения продакшена

### **Уровень 4: Git Architect** ✅
- **Проектирование workflow**: Настройка процессов для команды любого размера
- **Автоматизация качества**: Системы проверок, метрик и мониторинга
- **Процедуры экстренных ситуаций**: Hotfixes, rollbacks, disaster recovery
- **Менторство**: Способность обучать и направлять других разработчиков

---

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

---

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

---

## **Следующие шаги развития**

### **Углубление Git экспертизы:**
- **Git Internals** — изучение внутреннего устройства Git на уровне объектов
- **Advanced Git** — rebase, cherry-pick, bisect, submodules, worktrees
- **Git Performance** — оптимизация больших репозиториев, LFS для бинарных файлов

### **Расширение DevOps навыков:**
- **Container Orchestration** — Kubernetes, Docker Swarm
- **Infrastructure as Code** — Terraform, CloudFormation
- **Monitoring & Observability** — Prometheus, Grafana, ELK Stack
- **Security** — SAST, DAST, dependency scanning, secrets management

### **Leadership и процессы:**
- **Engineering Management** — управление техническими командами
- **System Design** — проектирование распределенных систем
- **Technical Writing** — создание качественной технической документации

---

## **Ресурсы для дальнейшего изучения**

### **Книги:**
- 📚 **"Pro Git" by Scott Chacon** — полное руководство по Git
- 📚 **"Continuous Delivery" by Jez Humble** — принципы непрерывной доставки
- 📚 **"The Phoenix Project" by Gene Kim** — DevOps культура и практики
- 📚 **"Building Microservices" by Sam Newman** — архитектура современных систем

### **Онлайн ресурсы:**
- 🌐 **[git-scm.com](https://git-scm.com/)** — официальная документация Git
- 🌐 **[trunkbaseddevelopment.com](https://trunkbaseddevelopment.com/)** — подробно о trunk-based development
- 🌐 **[GitHub Skills](https://skills.github.com/)** — интерактивные курсы GitHub
- 🌐 **[GitLab Learn](https://about.gitlab.com/learn/)** — DevOps best practices

### **Практические инструменты:**
- 🛠️ **[learngitbranching.js.org](https://learngitbranching.js.org/)** — интерактивное изучение Git
- 🛠️ **[gitexplorer.com](https://gitexplorer.com/)** — поиск нужных Git команд
- 🛠️ **[github.com/k88hudson/git-flight-rules](https://github.com/k88hudson/git-flight-rules)** — решения сложных Git ситуаций

---

## **Финальное слово**

**Поздравляем!** За 8 недель вы прошли путь от новичка до эксперта Git и trunk-based development. Это не просто технические навыки — это **философия профессиональной разработки**, которая делает вас ценным участником любой команды.

**Помните:**
- **Git — это не просто инструмент**, это основа современной разработки ПО
- **Trunk-based development — это не просто workflow**, это культура качества и скорости
- **Непрерывное обучение** — ключ к росту в IT индустрии

**Ваше новое техническое мышление:**
- 🎯 **Атомарность** — каждое изменение должно быть логически завершенным
- ⚡ **Скорость интеграции** — чем быстрее код попадает в main, тем меньше проблем
- 🔍 **Качество через автоматизацию** — машины проверяют, люди думают
- 📊 **Данные для решений** — метрики и мониторинг направляют развитие

**Вы готовы:**
- Настроить Git workflow в любой команде
- Менторить других разработчиков
- Предлагать и внедрять процессные улучшения
- Быть техническим лидером в области качества кода

**Время инвестиций: 56 часов**  
**Полученная экспертность: Пожизненная**  
**Влияние на карьеру: Бесценное**

> *"Лучшие разработчики не просто пишут код — они создают процессы, которые позволяют командам писать отличный код быстро и безопасно."*

**Удачи в вашем дальнейшем профессиональном развитии! 🚀**