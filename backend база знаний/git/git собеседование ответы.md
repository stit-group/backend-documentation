# Ответы на вопросы интервью Backend разработчика
## Git, GitHub, TDD - Развернутые ответы с примерами

---

## 🔧 БЛОК 1: GIT

### **1. Merge vs Rebase: объясните различия и применение**

**Merge** создает новый commit, объединяющий две ветки, сохраняя всю историю. **Rebase** перемещает commits одной ветки на вершину другой, переписывая историю.

**Ключевые различия:**
- Merge сохраняет контекст и временные метки
- Rebase создает линейную историю, но теряет информацию о том, когда реально происходила работа
- Merge безопаснее для shared веток, rebase лучше для feature веток

```bash
# Merge - безопасный, сохраняет историю
git checkout main
git merge feature-branch

# Rebase - переписывает историю, создает линейность
git checkout feature-branch
git rebase main
git checkout main
git merge feature-branch  # fast-forward merge
```

**Когда использовать:**
- **Merge**: для integration веток, публичных веток, когда важна история
- **Rebase**: для приведения в порядок feature веток перед merge, локальных commit'ов

**Golden rule**: Никогда не rebase публичные ветки, которые используют другие разработчики.

---

### **2. Git стратегии ветвления: выбор под проект**

**Git Flow** - сложная модель с release ветками, подходит для продуктов с четкими релизными циклами:
```bash
# Основные ветки: main, develop
# Вспомогательные: feature/, release/, hotfix/
git flow feature start new-feature
git flow feature finish new-feature
```

**GitHub Flow** - простая модель, подходит для continuous deployment:
```bash
# Только main ветка + feature ветки
git checkout -b feature/user-auth
# разработка -> PR -> merge в main -> deploy
```

**GitLab Flow** - комбинирует GitHub Flow с environment ветками:
```bash
# main -> staging -> production
# или main -> pre-production -> production
```

**Выбор стратегии:**
- **Стартап/Agile команда**: GitHub Flow
- **Enterprise с релизными циклами**: Git Flow  
- **Команда с staging окружениями**: GitLab Flow
- **Большая команда (20+ разработчиков)**: Git Flow или модифицированный GitLab Flow

---

### **3. Продвинутая навигация и восстановление в Git**

**Восстановление после git reset --hard:**
```bash
# Найти потерянные commits через reflog
git reflog
# commit 1234567 HEAD@{1}: commit: важная фича

# Восстановить
git reset --hard 1234567
# или создать новую ветку
git checkout -b recovery-branch 1234567
```

**Поиск бага с git bisect:**
```bash
# Автоматический поиск commit'а, который сломал код
git bisect start
git bisect bad                    # текущий commit сломан
git bisect good v1.0.0           # версия 1.0.0 работала

# Git будет предлагать commits для тестирования
git bisect good   # если текущий commit работает
git bisect bad    # если сломан
git bisect reset  # завершить поиск
```

**Восстановление удаленной ветки:**
```bash
# Если коллега удалил ветку, но она была у вас локально
git reflog --all | grep branch-name
git checkout -b recovered-branch commit-hash

# Если ветка была только на remote
git fsck --lost-found
# поиск среди dangling commits
```

---

### **4. Git hooks и автоматизация качества кода**

**Pre-commit hook для проверки качества:**
```bash
#!/bin/sh
# .git/hooks/pre-commit

# Проверка линтера
npm run lint
if [ $? -ne 0 ]; then
  echo "Lint errors found. Commit aborted."
  exit 1
fi

# Запуск unit тестов
npm test
if [ $? -ne 0 ]; then
  echo "Tests failed. Commit aborted."
  exit 1
fi

# Проверка на секреты
git diff --cached --name-only | xargs grep -l "password\|secret\|key" && {
  echo "Potential secrets found. Commit aborted."
  exit 1
}
```

**Распространение hooks на команду:**
```bash
# Создать shared hooks в отдельной папке
mkdir .githooks
# Настроить Git использовать эту папку
git config core.hooksPath .githooks
# Добавить в setup script проекта
echo "git config core.hooksPath .githooks" >> setup.sh
```

**Server-side hook для защиты main ветки:**
```bash
#!/bin/sh
# hooks/pre-receive на сервере

while read oldrev newrev refname; do
  branch=$(git rev-parse --symbolic --abbrev-ref $refname)
  if [ "$branch" = "main" ]; then
    echo "Direct push to main is not allowed"
    exit 1
  fi
done
```

---

### **5. Производительность и оптимизация Git**

**Оптимизация больших репозиториев:**
```bash
# Shallow clone для CI/CD
git clone --depth 1 https://github.com/user/repo.git

# Partial clone (Git 2.19+)
git clone --filter=blob:none https://github.com/user/repo.git

# Sparse checkout для monorepo
git config core.sparseCheckout true
echo "backend/*" > .git/info/sparse-checkout
git read-tree -u HEAD
```

**Очистка и оптимизация:**
```bash
# Очистка неиспользуемых объектов
git gc --aggressive --prune=now

# Удаление больших файлов из истории
git filter-repo --path large-file.zip --invert-paths

# Анализ размера репозитория
git count-objects -vH
```

**Git LFS для больших файлов:**
```bash
# Настройка LFS
git lfs install
git lfs track "*.zip"
git lfs track "*.pdf"
git add .gitattributes
git commit -m "Add LFS tracking"

# Проверка LFS файлов
git lfs ls-files
```

---

## 🐙 БЛОК 2: GITHUB

### **11. GitHub workflow и automation**

**Настройка PR template:**
```markdown
<!-- .github/pull_request_template.md -->
## Описание изменений
Brief description of changes

## Тип изменений
- [ ] Bug fix
- [ ] New feature  
- [ ] Breaking change
- [ ] Documentation update

## Checklist
- [ ] Код покрыт тестами
- [ ] Документация обновлена
- [ ] Self-review выполнен
- [ ] Нет конфликтов с main
```

**CODEOWNERS для обязательного review:**
```bash
# .github/CODEOWNERS
# Global owners
* @team-leads

# Frontend
/frontend/ @frontend-team
*.css @frontend-team

# Backend API
/api/ @backend-team @security-team
/database/ @backend-team @dba-team

# Infrastructure
/deploy/ @devops-team
Dockerfile @devops-team
```

**Branch protection rules (через API):**
```bash
curl -X PUT \
  https://api.github.com/repos/owner/repo/branches/main/protection \
  -H "Authorization: token $GITHUB_TOKEN" \
  -d '{
    "required_status_checks": {
      "strict": true,
      "contexts": ["ci/tests", "ci/lint"]
    },
    "enforce_admins": false,
    "required_pull_request_reviews": {
      "required_approving_review_count": 2,
      "dismiss_stale_reviews": true
    },
    "restrictions": null
  }'
```

---

### **12. GitHub Actions: CI/CD mastery**

**Matrix strategy для мультиплатформенного тестирования:**
```yaml
# .github/workflows/test.yml
name: Test Matrix
on: [push, pull_request]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [14, 16, 18, 20]
        exclude:
          - os: macos-latest
            node-version: 14
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm test
      env:
        NODE_ENV: test
```

**Conditional jobs с зависимостями:**
```yaml
jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      backend: ${{ steps.changes.outputs.backend }}
      frontend: ${{ steps.changes.outputs.frontend }}
    steps:
    - uses: dorny/paths-filter@v2
      id: changes
      with:
        filters: |
          backend:
            - 'backend/**'
          frontend:
            - 'frontend/**'

  test-backend:
    needs: changes
    if: ${{ needs.changes.outputs.backend == 'true' }}
    runs-on: ubuntu-latest
    steps:
    - name: Test backend
      run: npm run test:backend

  deploy:
    needs: [test-backend, test-frontend]
    if: always() && (needs.test-backend.result == 'success' || needs.test-backend.result == 'skipped')
    runs-on: ubuntu-latest
    steps:
    - name: Deploy
      run: echo "Deploying..."
```

---

### **13. GitHub Security и Enterprise**

**Security pipeline с Dependabot и CodeQL:**
```yaml
# .github/workflows/security.yml
name: Security Scan
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 2 * * 1'  # Еженедельно по понедельникам

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v2
      with:
        languages: javascript, python
    
    - name: Build
      run: npm run build
    
    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v2
    
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'fs'
        scan-ref: '.'
```

**Dependabot configuration:**
```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
    open-pull-requests-limit: 5
    reviewers:
      - "security-team"
    assignees:
      - "backend-team"
    commit-message:
      prefix: "deps"
      include: "scope"
```

---

## 🧪 БЛОК 3: TDD

### **16. TDD fundamentals и philosophy**

**Red-Green-Refactor цикл на практике:**
```javascript
// RED: Пишем failing test
describe('UserService', () => {
  test('should create user with valid email', () => {
    const userService = new UserService();
    const userData = { name: 'John', email: 'john@example.com' };
    
    const user = userService.createUser(userData);
    
    expect(user.id).toBeDefined();
    expect(user.email).toBe('john@example.com');
    expect(user.createdAt).toBeInstanceOf(Date);
  });
});

// GREEN: Минимальная реализация
class UserService {
  createUser(userData) {
    return {
      id: Math.random().toString(36),
      ...userData,
      createdAt: new Date()
    };
  }
}

// REFACTOR: Улучшаем код
class UserService {
  constructor(idGenerator = generateId, dateProvider = () => new Date()) {
    this.generateId = idGenerator;
    this.getDate = dateProvider;
  }
  
  createUser(userData) {
    this.validateEmail(userData.email);
    
    return {
      id: this.generateId(),
      name: userData.name,
      email: userData.email,
      createdAt: this.getDate()
    };
  }
  
  private validateEmail(email) {
    if (!email.includes('@')) {
      throw new Error('Invalid email format');
    }
  }
}
```

**YAGNI в TDD - избегаем over-engineering:**
```javascript
// ❌ Плохо - слишком много функциональности сразу
test('user service should handle all possible scenarios', () => {
  // тест на 50 строк с множеством assertions
});

// ✅ Хорошо - один тест = одно поведение
test('should throw error for invalid email', () => {
  const userService = new UserService();
  
  expect(() => {
    userService.createUser({ email: 'invalid-email' });
  }).toThrow('Invalid email format');
});

test('should generate unique IDs for different users', () => {
  const userService = new UserService();
  
  const user1 = userService.createUser({ email: 'user1@test.com' });
  const user2 = userService.createUser({ email: 'user2@test.com' });
  
  expect(user1.id).not.toBe(user2.id);
});
```

---

### **17. Testing patterns и best practices**

**AAA vs Given-When-Then patterns:**
```javascript
// AAA (Arrange-Act-Assert) - для unit тестов
test('should calculate order total with discount', () => {
  // Arrange
  const order = new Order();
  order.addItem({ price: 100, quantity: 2 });
  const discount = new PercentageDiscount(10);
  
  // Act
  const total = order.calculateTotal(discount);
  
  // Assert
  expect(total).toBe(180);
});

// Given-When-Then - для BDD/integration тестов
describe('Order processing', () => {
  test('discounted order total calculation', () => {
    // Given an order with items and a discount
    const order = new Order();
    order.addItem(createItem({ price: 100, quantity: 2 }));
    const discount = createDiscount({ type: 'percentage', value: 10 });
    
    // When calculating the total
    const total = order.calculateTotal(discount);
    
    // Then the total should include the discount
    expect(total).toBe(180);
    expect(order.appliedDiscounts).toContain(discount);
  });
});
```

**Test data management - Factories vs Fixtures:**
```javascript
// Factory pattern - гибкость
class UserFactory {
  static create(overrides = {}) {
    return {
      id: generateId(),
      name: 'Test User',
      email: 'test@example.com',
      role: 'user',
      createdAt: new Date(),
      ...overrides
    };
  }
  
  static createAdmin(overrides = {}) {
    return this.create({ role: 'admin', ...overrides });
  }
}

// Использование
test('admin can delete users', () => {
  const admin = UserFactory.createAdmin();
  const user = UserFactory.create();
  
  const result = userService.deleteUser(user.id, admin);
  
  expect(result.success).toBe(true);
});

// Object Mother pattern - для сложных объектов
class OrderMother {
  static simple() {
    return new Order([
      ItemMother.book(),
      ItemMother.electronics()
    ]);
  }
  
  static withMultipleDiscounts() {
    const order = this.simple();
    order.addDiscount(DiscountMother.percentage(10));
    order.addDiscount(DiscountMother.fixed(5));
    return order;
  }
}
```

---

### **18. Mocking и test doubles strategy**

**Типы test doubles и их применение:**
```javascript
// Stub - возвращает предопределенные значения
const paymentGatewayStub = {
  processPayment: () => ({ success: true, transactionId: '12345' })
};

// Mock - проверяет взаимодействия
const emailServiceMock = {
  sendEmail: jest.fn(),
  getCallCount: function() { return this.sendEmail.mock.calls.length; }
};

// Spy - отслеживает вызовы реального объекта
const userRepository = new UserRepository();
const userRepositorySpy = jest.spyOn(userRepository, 'save');

// Fake - упрощенная рабочая реализация
class InMemoryUserRepository {
  constructor() {
    this.users = new Map();
  }
  
  save(user) {
    this.users.set(user.id, user);
    return Promise.resolve(user);
  }
  
  findById(id) {
    return Promise.resolve(this.users.get(id));
  }
}

// Пример использования в тесте
test('should send welcome email after user registration', async () => {
  // Arrange
  const userRepo = new InMemoryUserRepository();
  const emailService = { sendEmail: jest.fn() };
  const userService = new UserService(userRepo, emailService);
  
  // Act
  await userService.registerUser({ email: 'test@example.com' });
  
  // Assert
  expect(emailService.sendEmail).toHaveBeenCalledWith({
    to: 'test@example.com',
    template: 'welcome'
  });
});
```

**Database testing strategies:**
```javascript
// In-memory database для unit тестов
beforeEach(() => {
  const db = new InMemoryDatabase();
  userRepository = new UserRepository(db);
});

// Test containers для integration тестов
beforeAll(async () => {
  container = await new GenericContainer("postgres:13")
    .withEnvironment({ POSTGRES_DB: "test" })
    .withExposedPorts(5432)
    .start();
    
  const connectionString = `postgresql://postgres@localhost:${container.getMappedPort(5432)}/test`;
  db = new Database(connectionString);
});

// Transaction rollback для изоляции
beforeEach(async () => {
  await db.beginTransaction();
});

afterEach(async () => {
  await db.rollback();
});
```

---

### **19. Testing pyramid и strategy**

**Пропорции в testing pyramid:**
```javascript
// Unit tests (70%) - быстрые, изолированные
describe('OrderCalculator', () => {
  test('calculates tax correctly', () => {
    const calculator = new OrderCalculator();
    const order = { subtotal: 100, taxRate: 0.1 };
    
    expect(calculator.calculateTax(order)).toBe(10);
  });
});

// Integration tests (20%) - взаимодействие компонентов
describe('Order API', () => {
  test('creates order with valid data', async () => {
    const response = await request(app)
      .post('/api/orders')
      .send({
        customerId: '123',
        items: [{ productId: '456', quantity: 2 }]
      })
      .expect(201);
    
    expect(response.body.order.total).toBeGreaterThan(0);
    
    // Проверяем, что заказ сохранен в базе
    const savedOrder = await orderRepository.findById(response.body.order.id);
    expect(savedOrder).toBeDefined();
  });
});

// E2E tests (10%) - полный пользовательский сценарий
describe('Order placement flow', () => {
  test('user can place order and receive confirmation', async () => {
    // Полный флоу через UI или API
    await loginAsUser('customer@example.com');
    await addProductToCart('product-123');
    await proceedToCheckout();
    await fillShippingInfo();
    await confirmPayment();
    
    await expect(page.locator('[data-testid="order-confirmation"]')).toBeVisible();
    await expect(page.locator('[data-testid="order-number"]')).toContainText(/ORD-\d+/);
  });
});
```

**Test coverage strategy:**
```javascript
// Настройка coverage с исключениями
module.exports = {
  collectCoverageFrom: [
    'src/**/*.{js,ts}',
    '!src/**/*.d.ts',
    '!src/**/*.test.{js,ts}',
    '!src/**/index.{js,ts}',
    '!src/migrations/**'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    },
    './src/services/': {
      branches: 90,
      functions: 95,
      lines: 95,
      statements: 95
    }
  }
};

// Mutation testing для проверки качества тестов
// package.json
{
  "scripts": {
    "test:mutation": "stryker run"
  }
}
```

---

### **20. Async и complex scenarios testing**

**Тестирование асинхронного кода:**
```javascript
// Promises и async/await
test('should process async operation', async () => {
  const userService = new UserService();
  
  const user = await userService.createUserAsync({
    email: 'test@example.com'
  });
  
  expect(user.id).toBeDefined();
});

// Event-driven код
test('should emit user created event', (done) => {
  const eventBus = new EventBus();
  const userService = new UserService(eventBus);
  
  eventBus.on('user.created', (user) => {
    expect(user.email).toBe('test@example.com');
    done();
  });
  
  userService.createUser({ email: 'test@example.com' });
});

// Timeout и retry logic
test('should retry failed operations', async () => {
  const failingService = {
    call: jest.fn()
      .mockRejectedValueOnce(new Error('Network error'))
      .mockRejectedValueOnce(new Error('Network error'))
      .mockResolvedValueOnce({ success: true })
  };
  
  const retryService = new RetryService(failingService, { maxRetries: 3 });
  
  const result = await retryService.callWithRetry();
  
  expect(result.success).toBe(true);
  expect(failingService.call).toHaveBeenCalledTimes(3);
});
```

**Тестирование race conditions:**
```javascript
test('should handle concurrent user creation', async () => {
  const userService = new UserService();
  const email = 'test@example.com';
  
  // Запускаем одновременно
  const promises = [
    userService.createUser({ email }),
    userService.createUser({ email }),
    userService.createUser({ email })
  ];
  
  const results = await Promise.allSettled(promises);
  
  // Только один должен успешно создаться
  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');
  
  expect(successful).toHaveLength(1);
  expect(failed).toHaveLength(2);
  expect(failed[0].reason.message).toContain('already exists');
});
```

---

### **21. TDD в различных контекстах**

**Legacy code - характеризационные тесты:**
```javascript
// Сначала создаем тест, описывающий текущее поведение
test('legacy calculateDiscount behavior', () => {
  // Неизвестная логика, но нужно зафиксировать поведение
  const result = legacyCalculateDiscount(100, 'SUMMER', new Date('2023-07-15'));
  
  // Запускаем и записываем результат
  expect(result).toBe(15); // Whatever the current output is
});

// Затем рефакторим с уверенностью
class DiscountCalculator {
  calculate(amount, code, date) {
    if (code === 'SUMMER' && this.isSummerPeriod(date)) {
      return amount * 0.15;
    }
    return 0;
  }
  
  private isSummerPeriod(date) {
    const month = date.getMonth() + 1;
    return month >= 6 && month <= 8;
  }
}
```

**Event-driven architecture testing:**
```javascript
test('should handle order saga', async () => {
  const eventStore = new InMemoryEventStore();
  const saga = new OrderSaga(eventStore);
  
  // Given: order created event
  await eventStore.append('order-123', [
    new OrderCreatedEvent({ orderId: 'order-123', amount: 100 })
  ]);
  
  // When: payment processed
  await saga.handle(new PaymentProcessedEvent({ 
    orderId: 'order-123', 
    success: true 
  }));
  
  // Then: should emit inventory reserved event
  const events = await eventStore.getEvents('order-123');
  const inventoryEvent = events.find(e => e.type === 'InventoryReserved');
  
  expect(inventoryEvent).toBeDefined();
  expect(inventoryEvent.orderId).toBe('order-123');
});
```

---

### **22. Testing tools и automation**

**Property-based testing:**
```javascript
import fc from 'fast-check';

test('reverse of reverse should equal original', () => {
  fc.assert(fc.property(
    fc.array(fc.integer()),
    (arr) => {
      const reversed = reverse(reverse(arr));
      expect(reversed).toEqual(arr);
    }
  ));
});

test('sorted array should be ordered', () => {
  fc.assert(fc.property(
    fc.array(fc.integer()),
    (arr) => {
      const sorted = sort(arr);
      
      // Property: каждый элемент <= следующего
      for (let i = 0; i < sorted.length - 1; i++) {
        expect(sorted[i]).toBeLessThanOrEqual(sorted[i + 1]);
      }
      
      // Property: содержит те же элементы
      expect(sorted.sort()).toEqual(arr.sort());
    }
  ));
});
```

**Contract testing с Pact:**
```javascript
// Consumer test
const { Pact } = require('@pact-foundation/pact');

const provider = new Pact({
  consumer: 'OrderService',
  provider: 'UserService'
});

describe('User API contract', () => {
  beforeAll(() => provider.setup());
  afterAll(() => provider.finalize());

  test('should get user by ID', async () => {
    await provider.addInteraction({
      state: 'user exists',
      uponReceiving: 'a request for user',
      withRequest: {
        method: 'GET',
        path: '/users/123'
      },
      willRespondWith: {
        status: 200,
        body: {
          id: '123',
          name: 'John Doe',
          email: 'john@example.com'
        }
      }
    });

    const userService = new UserService(provider.mockService.baseUrl);
    const user = await userService.getUser('123');
    
    expect(user.id).toBe('123');
  });
});
```

---

### **23. Advanced testing techniques**

**BDD с Cucumber/Gherkin:**
```gherkin
# features/order.feature
Feature: Order processing
  As a customer
  I want to place orders
  So that I can buy products

  Scenario: Successful order placement
    Given I am a registered customer
    And I have items in my cart
    When I proceed to checkout
    And I provide valid payment information
    Then my order should be created
    And I should receive order confirmation
    And my cart should be empty

  Scenario Outline: Order with different discounts
    Given I have <item_count> items worth <total> in my cart
    When I apply discount code "<code>"
    Then my total should be <final_total>
    
    Examples:
      | item_count | total | code    | final_total |
      | 2          | 100   | SAVE10  | 90          |
      | 5          | 250   | BULK20  | 200         |
```

```javascript
// steps/order.steps.js
const { Given, When, Then } = require('@cucumber/cucumber');

Given('I am a registered customer', async function() {
  this.customer = await CustomerFactory.create();
  this.auth = await AuthService.login(this.customer.email);
});

When('I proceed to checkout', async function() {
  this.order = await OrderService.createOrder({
    customerId: this.customer.id,
    items: this.cartItems
  });
});

Then('my order should be created', function() {
  expect(this.order.id).toBeDefined();
  expect(this.order.status).toBe('created');
});
```

**Testing с feature flags:**
```javascript
describe('New checkout flow', () => {
  beforeEach(() => {
    FeatureFlags.enable('new_checkout_flow');
  });
  
  test('should use new payment processor', async () => {
    const orderService = new OrderService();
    const order = await orderService.processOrder(orderData);
    
    expect(order.paymentProcessor).toBe('stripe-v2');
  });
  
  test('should fall back to old flow when disabled', async () => {
    FeatureFlags.disable('new_checkout_flow');
    
    const orderService = new OrderService();
    const order = await orderService.processOrder(orderData);
    
    expect(order.paymentProcessor).toBe('paypal-legacy');
  });
});
```

---

## 📊 Заключение

### **Ключевые принципы качественного тестирования:**

1. **Тесты как документация** - тест должен четко описывать ожидаемое поведение
2. **Быстрая обратная связь** - большинство тестов должны выполняться за секунды
3. **Независимость тестов** - каждый тест должен работать изолированно
4. **Читаемость** - тесты читают чаще, чем пишут
5. **Покрытие поведения, а не кода** - 100% покрытие не гарантирует качество

### **Практические рекомендации:**

- Начинайте с простых unit тестов, постепенно добавляя интеграционные
- Инвестируйте в хорошие test utilities и helpers
- Автоматизируйте запуск тестов в CI/CD
- Регулярно рефакторите тесты вместе с кодом
- Используйте mutation testing для проверки качества тестов

### **Красные флаги в тестировании:**

- Тесты, которые часто ломаются без изменения логики
- Медленные тесты, которые разработчики пропускают
- Тесты, которые тестируют implementation details вместо поведения
- Отсутствие тестов для критически важной функциональности
- Моки, которые не соответствуют реальному API