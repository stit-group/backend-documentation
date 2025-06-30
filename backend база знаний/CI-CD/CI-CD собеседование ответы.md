# CI/CD Interview Template - Развернутая версия с примерами

## 📋 Инструкция
- **Время**: 90-120 минут  
- **Формат**: 30 основных вопросов + развернутые ответы для ключевых тем
- **Оценка**: 1-5 баллов (критерии указаны)
- **Структура**: Практические задачи + теоретические основы + примеры кода

---

## 🎯 Блок 1: Основы и Pipeline Design (8 вопросов)

### 1.1 Опишите идеальный CI/CD pipeline для веб-приложения с БД

**Эталонный ответ:**
```yaml
# .gitlab-ci.yml пример
stages:
  - validate
  - build
  - test
  - security
  - deploy-staging
  - integration-tests
  - deploy-production

validate:
  stage: validate
  script:
    - lint code
    - validate infrastructure code
    - check commit message format

build:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  only:
    - main
    - develop

database-migration-check:
  stage: test
  script:
    - ./scripts/check-migration-safety.sh
    - ./scripts/test-rollback-scenario.sh
```

**Ключевые принципы:**
- **Fail Fast**: Быстрые проверки первыми (lint, unit tests)
- **Parallel Execution**: Независимые стадии выполняются параллельно
- **Gates**: Автоматические и ручные gates перед production
- **Rollback Strategy**: Всегда должен быть план отката

**Углубление при хорошем ответе:**
- Как обеспечить zero-downtime deployments?
- Где разместить database migrations в pipeline?

**Оценка:**
- 🟢 5 баллов: Описывает стадии, gates, rollback strategy, database considerations
- 🟡 3 балла: Базовые стадии build→test→deploy, понимает основную логику
- 🔴 1 балл: Путается в терминологии, не видит картину целиком

### 1.2 У вас есть монорепо с 10 микросервисами. Как организовать CI/CD?

**Эталонный ответ:**
```yaml
# Selective building strategy
.detect-changes: &detect-changes
  - |
    if git diff --name-only $CI_COMMIT_BEFORE_SHA $CI_COMMIT_SHA | grep -q "^services/user-service/"; then
      export BUILD_USER_SERVICE=true
    fi

build-user-service:
  <<: *detect-changes
  script:
    - |
      if [ "$BUILD_USER_SERVICE" == "true" ]; then
        cd services/user-service
        docker build -t user-service:$CI_COMMIT_SHA .
      fi
  rules:
    - changes:
        - services/user-service/**/*
```

**Архитектурные решения:**
- **Change Detection**: Строить только измененные сервисы
- **Dependency Graph**: Учитывать зависимости между сервисами
- **Shared Libraries**: Отдельный pipeline для общих библиотек
- **Integration Testing**: Контрактное тестирование между сервисами

### 1.3 Команда жалуется, что CI pipeline занимает 45 минут. Ваши действия?

**Эталонный ответ:**
```bash
# Скрипт анализа производительности pipeline
#!/bin/bash
# analyze-pipeline-performance.sh

echo "Анализ времени выполнения стадий:"
gitlab-ci-analyzer --project-id=$PROJECT_ID --token=$TOKEN \
  --from-date="$(date -d '30 days ago' '+%Y-%m-%d')" \
  --group-by=stage

echo "Топ медленных джобов:"
gitlab-ci-analyzer --slowest-jobs --limit=10
```

**План оптимизации:**
1. **Параллелизация**: Разделить длинные стадии
2. **Кэширование**: Docker layer cache, dependency cache
3. **Incremental Builds**: Строить только измененное
4. **Resource Allocation**: Увеличить runners для критических стадий

### 1.5 Что такое "Infrastructure as Code" и зачем это нужно?

**Эталонный ответ:**
```hcl
# terraform/main.tf
resource "aws_ecs_cluster" "app_cluster" {
  name = "${var.environment}-app-cluster"
  
  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}

resource "aws_ecs_service" "app_service" {
  name            = "${var.environment}-app"
  cluster         = aws_ecs_cluster.app_cluster.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = var.app_replicas
  
  deployment_configuration {
    maximum_percent         = 200
    minimum_healthy_percent = 100
  }
}
```

**Преимущества IaC:**
- **Reproducibility**: Одинаковая инфраструктура везде
- **Version Control**: История изменений инфраструктуры
- **Testing**: Тестирование инфраструктуры как кода
- **Compliance**: Автоматическая проверка соответствия политикам

---

## 🔧 Блок 2: Автоматизация и Testing (7 вопросов)

### 2.3 Flaky tests убивают доверие к CI. Как решать?

**Эталонный ответ:**
```python
# scripts/flaky-test-detector.py
import pytest
import statistics

class FlakyTestDetector:
    def __init__(self):
        self.test_results = {}
    
    def record_test_result(self, test_name, passed, duration):
        if test_name not in self.test_results:
            self.test_results[test_name] = []
        
        self.test_results[test_name].append({
            'passed': passed,
            'duration': duration
        })
    
    def detect_flaky_tests(self, min_runs=10, failure_threshold=0.1):
        flaky_tests = []
        
        for test_name, results in self.test_results.items():
            if len(results) < min_runs:
                continue
                
            failure_rate = sum(1 for r in results if not r['passed']) / len(results)
            duration_variance = statistics.stdev([r['duration'] for r in results])
            
            if failure_rate > failure_threshold or duration_variance > 5.0:
                flaky_tests.append({
                    'test': test_name,
                    'failure_rate': failure_rate,
                    'duration_variance': duration_variance
                })
        
        return flaky_tests
```

**Стратегии борьбы:**
- **Quarantine**: Временное отключение flaky тестов
- **Retry Logic**: Умный retry с экспоненциальным backoff
- **Environment Isolation**: Каждый тест в чистом окружении
- **Test Data Management**: Предсказуемые тестовые данные

---

## 🐳 Блок 3: Контейнеризация и Kubernetes (8 вопросов)

### 3.1 Best practices для production-ready Dockerfile

**Эталонный ответ:**
```dockerfile
# Плохой пример
FROM ubuntu:latest
RUN apt-get update && apt-get install -y nodejs npm
COPY . /app
WORKDIR /app
RUN npm install
CMD ["node", "server.js"]

# Хороший пример
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

FROM node:18-alpine AS runtime
RUN addgroup -g 1001 -S nodejs && adduser -S nextjs -u 1001
WORKDIR /app
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --chown=nextjs:nodejs . .
USER nextjs
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
CMD ["node", "server.js"]
```

**Ключевые принципы:**
- **Multi-stage builds**: Минимизация размера финального образа
- **Non-root user**: Безопасность через ограничение привилегий
- **Layer optimization**: Кэширование слоев для быстрых билдов
- **Health checks**: Встроенная проверка здоровья контейнера

### 3.3 Kubernetes deployment strategies - сравните подходы

**Эталонный ответ:**
```yaml
# Rolling Update (default)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-rolling
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1

---
# Blue-Green Deployment
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
    version: blue  # Переключаем на green при деплое

---
# Canary Deployment с Istio
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: myapp-canary
spec:
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: myapp
        subset: v2
  - route:
    - destination:
        host: myapp
        subset: v1
      weight: 90
    - destination:
        host: myapp
        subset: v2
      weight: 10  # 10% трафика на новую версию
```

**Trade-offs:**
- **Rolling**: Минимум ресурсов, но возможны проблемы совместимости
- **Blue-Green**: Быстрый rollback, но двойные ресурсы
- **Canary**: Минимальный риск, но сложность настройки

### 3.4 GitOps в Kubernetes - архитектура решения

**Эталонный ответ:**
```yaml
# argocd-application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-production
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/company/k8s-manifests
    targetRevision: HEAD
    path: production/myapp
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

**Архитектура GitOps:**
- **Git as Source of Truth**: Все конфигурации в Git
- **Pull-based**: ArgoCD/Flux забирают изменения из Git
- **Declarative**: Описываем желаемое состояние
- **Observable**: Все изменения отслеживаются

---

## 🚀 Блок 4: Deployment и Production (7 вопросов)

### 4.1 Database migrations в zero-downtime deployment

**Эталонный ответ:**
```sql
-- Плохой подход (breaking change)
ALTER TABLE users DROP COLUMN old_email;

-- Хороший подход (backward compatible)
-- Step 1: Add new column (deploy v1.1)
ALTER TABLE users ADD COLUMN email VARCHAR(255);

-- Step 2: Backfill data (background job)
UPDATE users SET email = old_email WHERE email IS NULL;

-- Step 3: Update application to use new column (deploy v1.2)
-- Step 4: Remove old column (deploy v1.3, after rollback window)
ALTER TABLE users DROP COLUMN old_email;
```

```bash
#!/bin/bash
# scripts/safe-migration.sh
set -e

echo "Проверка backward compatibility миграции..."

# Тест: старая версия приложения работает с новой схемой
docker run --rm myapp:previous-version test-database-compatibility

# Тест: rollback scenario
echo "Тестирование сценария отката..."
./test-rollback-migration.sh

echo "Миграция безопасна для zero-downtime deployment"
```

**Стратегии safe migrations:**
- **Additive Changes**: Только добавление колонок/таблиц
- **Multi-step Process**: Разбивка breaking changes на этапы
- **Backward Compatibility**: Поддержка старой и новой схемы
- **Automated Testing**: Проверка rollback scenarios

### 4.2 Feature flags architecture и lifecycle

**Эталонный ответ:**
```javascript
// feature-flags/client.js
class FeatureFlagClient {
  constructor(config) {
    this.config = config;
    this.cache = new Map();
  }
  
  async isEnabled(flagName, context = {}) {
    const cacheKey = `${flagName}:${JSON.stringify(context)}`;
    
    if (this.cache.has(cacheKey)) {
      return this.cache.get(cacheKey);
    }
    
    const result = await this.evaluate(flagName, context);
    this.cache.set(cacheKey, result);
    
    // Метрики для отслеживания использования
    this.trackFlagUsage(flagName, result, context);
    
    return result;
  }
  
  async evaluate(flagName, context) {
    const flag = await this.fetchFlag(flagName);
    
    // Проверка targeting rules
    for (const rule of flag.rules) {
      if (this.matchesRule(rule, context)) {
        return rule.enabled;
      }
    }
    
    return flag.defaultValue;
  }
  
  trackFlagUsage(flagName, result, context) {
    // Отправка метрик для анализа и cleanup
    this.metrics.increment('feature_flag.evaluation', {
      flag: flagName,
      result: result.toString(),
      user_segment: context.userSegment
    });
  }
}
```

**Lifecycle management:**
- **Creation**: Clear naming, documentation, cleanup date
- **Rollout**: Gradual percentage rollout
- **Monitoring**: Usage metrics, error rates
- **Cleanup**: Automated removal of obsolete flags

---

## 📊 Практические сценарии с решениями

### Сценарий 1: Crisis Management
> "Production упал после деплоя 2 часа назад. Rollback не работает, БД была мигрирована. Ваши действия по шагам?"

**Пошаговое решение:**

```bash
#!/bin/bash
# incident-response-runbook.sh

echo "=== INCIDENT RESPONSE PLAYBOOK ==="
echo "Время начала: $(date)"

# Step 1: Остановить дальнейшие деплои
echo "1. Блокировка pipeline..."
gitlab-ci-pipelines cancel --project-id=$PROJECT_ID --ref=main

# Step 2: Оценка состояния
echo "2. Проверка состояния сервисов..."
kubectl get pods -l app=myapp --sort-by=.metadata.creationTimestamp
kubectl logs -l app=myapp --since=2h --tail=100

# Step 3: Database rollback strategy
echo "3. Проверка возможности DB rollback..."
if ./scripts/check-db-rollback-safety.sh; then
  echo "DB rollback безопасен"
  ./scripts/rollback-database.sh
else
  echo "DB rollback небезопасен, нужен forward fix"
  ./scripts/apply-hotfix-migration.sh
fi

# Step 4: Application rollback или hotfix
echo "4. Восстановление приложения..."
if [ "$DB_ROLLBACK_SAFE" == "true" ]; then
  kubectl rollout undo deployment/myapp
else
  # Deploy hotfix version
  kubectl set image deployment/myapp app=myapp:hotfix-$HOTFIX_VERSION
fi

# Step 5: Мониторинг восстановления
echo "5. Мониторинг восстановления..."
kubectl rollout status deployment/myapp --timeout=300s

echo "=== INCIDENT RESOLVED ==="
echo "Время завершения: $(date)"
```

### Сценарий 2: Scale Challenge  
> "Команда выросла с 5 до 50 разработчиков. CI время увеличилось до 2 часов, конфликты merge каждый день. План действий?"

**План оптимизации:**

```yaml
# .gitlab-ci.yml - оптимизированная версия
stages:
  - fast-feedback
  - build
  - test-parallel
  - integration
  - deploy

# Быстрая обратная связь (< 5 минут)
lint-and-format:
  stage: fast-feedback
  script:
    - make lint
    - make format-check
  cache:
    key: lint-cache
    paths:
      - .lint-cache/

unit-tests:
  stage: fast-feedback
  parallel:
    matrix:
      - SERVICE: [user-service, order-service, payment-service]
  script:
    - cd services/$SERVICE
    - npm test -- --coverage
  coverage: '/Coverage: \d+\.\d+%/'

# Параллельное тестирование
integration-tests:
  stage: test-parallel
  parallel: 5
  script:
    - ./scripts/run-integration-tests.sh $CI_NODE_INDEX $CI_NODE_TOTAL
```

**Процессные улучшения:**
- **Trunk-based Development**: Короткоживущие feature branches
- **Pre-commit Hooks**: Быстрая проверка перед push
- **Parallel Testing**: Разделение тестов по runners
- **Incremental Builds**: Кэширование и smart rebuilds

---

## 🎯 Дополнительные примеры кода

### Automated Rollback на основе метрик

```python
# monitoring/auto-rollback.py
import time
from prometheus_client.parser import text_string_to_metric_families

class AutoRollbackMonitor:
    def __init__(self, deployment_name, thresholds):
        self.deployment = deployment_name
        self.thresholds = thresholds
        self.baseline_metrics = {}
    
    def monitor_deployment(self, duration_minutes=30):
        start_time = time.time()
        
        while time.time() - start_time < duration_minutes * 60:
            current_metrics = self.get_current_metrics()
            
            if self.should_rollback(current_metrics):
                self.trigger_rollback()
                return False
            
            time.sleep(30)  # Check every 30 seconds
        
        return True  # Deployment successful
    
    def should_rollback(self, metrics):
        # Error rate check
        if metrics['error_rate'] > self.thresholds['max_error_rate']:
            return True
        
        # Response time check
        if metrics['p95_response_time'] > self.thresholds['max_response_time']:
            return True
        
        # Memory usage check
        if metrics['memory_usage'] > self.thresholds['max_memory_usage']:
            return True
        
        return False
    
    def trigger_rollback(self):
        print(f"Triggering automatic rollback for {self.deployment}")
        os.system(f"kubectl rollout undo deployment/{self.deployment}")
```

### Contract Testing для микросервисов

```javascript
// tests/contract-tests.js
const { Pact } = require('@pact-foundation/pact');
const { UserService } = require('../services/user-service');

const provider = new Pact({
  consumer: 'OrderService',
  provider: 'UserService',
  port: 1234,
});

describe('User Service Contract', () => {
  before(() => provider.setup());
  after(() => provider.finalize());

  it('should get user by ID', async () => {
    await provider.addInteraction({
      state: 'user with ID 123 exists',
      uponReceiving: 'a request for user 123',
      withRequest: {
        method: 'GET',
        path: '/users/123',
        headers: {
          'Accept': 'application/json',
        },
      },
      willRespondWith: {
        status: 200,
        headers: {
          'Content-Type': 'application/json',
        },
        body: {
          id: 123,
          name: 'John Doe',
          email: 'john@example.com',
        },
      },
    });

    const userService = new UserService('http://localhost:1234');
    const user = await userService.getUser(123);
    
    expect(user.id).to.equal(123);
    expect(user.name).to.equal('John Doe');
  });
});
```

---

## 📝 Рекомендации по использованию шаблона

### Для интервьюера:
- Начните с базовых вопросов, затем углубляйтесь в детали
- Просите кандидата объяснить trade-offs решений
- Обращайте внимание на практический опыт vs теоретические знания
- Используйте примеры кода для проверки понимания

### Для кандидата:
- Изучите примеры кода и понимайте принципы за ними
- Готовьтесь объяснить не только "как", но и "почему"
- Практикуйтесь в объяснении сложных концепций простыми словами
- Подготовьте примеры из своего опыта для каждой области

---

*Этот развернутый шаблон обеспечивает глубокую техническую оценку с практическими примерами, сохраняя фокус на ключевых компетенциях CI/CD инженера.*