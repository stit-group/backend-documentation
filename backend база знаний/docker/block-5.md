# 🚀 Блок 5: CI/CD и автоматизация с Docker

*Превращаем разработку в автоматизированный конвейер качества*

---

## 📋 Обзор блока

В этом блоке мы изучим, как Docker революционизирует процессы непрерывной интеграции и развертывания (CI/CD). Вы поймете, как контейнеры решают проблему "у меня работает" и создают предсказуемые, воспроизводимые среды для всего жизненного цикла разработки.

**Что вы изучите:**
- Интеграцию Docker в популярные CI/CD системы
- Автоматизацию сборки и тестирования
- Управление образами и реестрами
- Стратегии развертывания
- Оптимизацию процессов сборки

---

## 🎯 Глава 5.1: Docker в CI/CD пайплайнах

### Концептуальные основы

#### Что такое CI/CD в контексте контейнеров?

```
Традиционный подход:
Разработчик → Код → Сервер сборки → Тестовая среда → Production
    ↓              ↓                    ↓               ↓
Локальная       Другая ОС,         Еще одна        Совсем другая
  среда         зависимости        конфигурация    конфигурация

--------------------------------------------------------------

С Docker:
Разработчик → Dockerfile → Образ → Тот же образ → Тот же образ
    ↓              ↓          ↓           ↓            ↓
Контейнер      Сборка в   Тесты в    Staging в    Production в
локально       контейнере  контейнере  контейнере   контейнере
```

#### Преимущества контейнеризации в CI/CD

**1. Консистентность среды выполнения**
```
БЕЗ Docker:
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│  Dev (Mac)  │   │ CI (Linux)  │   │Prod (RHEL)  │
│ Node 14.x   │ → │ Node 16.x   │ → │ Node 12.x   │
│ Python 3.8  │   │ Python 3.9  │   │ Python 3.7  │
└─────────────┘   └─────────────┘   └─────────────┘
     Разные версии = Разные результаты

С Docker:
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│    Dev      │   │     CI      │   │    Prod     │
│ ┌─────────┐ │ → │ ┌─────────┐ │ → │ ┌─────────┐ │
│ │Container│ │   │ │Container│ │   │ │Container│ │
│ │Same Env │ │   │ │Same Env │ │   │ │Same Env │ │
│ └─────────┘ │   │ └─────────┘ │   │ └─────────┘ │
└─────────────┘   └─────────────┘   └─────────────┘
        Одинаковая среда = Предсказуемые результаты
```

**2. Изоляция зависимостей**
```
Проблема без контейнеров:
┌────────────────────────────────────┐
│         CI/CD Сервер               │
│  ┌──────┐ ┌──────┐ ┌──────┐       │
│  │Proj A│ │Proj B│ │Proj C│       │
│  │Node14│ │Node16│ │Python│       │
│  └──────┘ └──────┘ └──────┘       │
│    Конфликты зависимостей!         │
└────────────────────────────────────┘

Решение с Docker:
┌────────────────────────────────────┐
│         CI/CD Сервер               │
│ ┌────────┐ ┌────────┐ ┌────────┐  │
│ │┌──────┐│ │┌──────┐│ │┌──────┐│  │
│ ││Proj A││ ││Proj B││ ││Proj C││  │
│ ││Node14││ ││Node16││ ││Python││  │
│ │└──────┘│ │└──────┘│ │└──────┘│  │
│ │Container│ │Container│ │Container│  │
│ └────────┘ └────────┘ └────────┘  │
│     Полная изоляция!               │
└────────────────────────────────────┘
```

### Архитектура CI/CD с Docker

#### Типичный пайплайн

```
┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│   Source    │   │    Build    │   │    Test     │   │   Deploy    │
│   Control   │ → │   Docker    │ → │  Container  │ → │     To      │
│             │   │   Image     │   │             │   │ Production  │
└─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘
       ↓                 ↓                 ↓                 ↓
  Git Push          Dockerfile        docker run        docker push
  Webhook           docker build      npm test          kubectl apply
  Pull Request      Layer Cache       pytest            helm install
```

#### Многоступенчатый процесс

```
CI/CD Pipeline Structure:
══════════════════════════════════════════════════════════════

Stage 1: Source
┌─────────────────────────────────────────────────────────────┐
│ • Git checkout                                              │
│ • Подготовка рабочего пространства                         │
│ • Проверка изменений                                        │
└─────────────────────────────────────────────────────────────┘
                            ↓
Stage 2: Build
┌─────────────────────────────────────────────────────────────┐
│ • docker build -t app:${BUILD_ID} .                        │
│ • Кеширование слоев                                         │
│ • Оптимизация размера                                       │
└─────────────────────────────────────────────────────────────┘
                            ↓
Stage 3: Test
┌─────────────────────────────────────────────────────────────┐
│ • Unit tests в контейнере                                   │
│ • Integration tests                                         │
│ • Security scanning                                         │
│ • Performance tests                                         │
└─────────────────────────────────────────────────────────────┘
                            ↓
Stage 4: Push
┌─────────────────────────────────────────────────────────────┐
│ • docker tag app:${BUILD_ID} registry/app:latest           │
│ • docker push registry/app:${BUILD_ID}                     │
│ • Обновление метаданных                                     │
└─────────────────────────────────────────────────────────────┘
                            ↓
Stage 5: Deploy
┌─────────────────────────────────────────────────────────────┐
│ • Развертывание в staging                                   │
│ • Smoke tests                                               │
│ • Production deployment                                     │
└─────────────────────────────────────────────────────────────┘
```

### Популярные CI/CD платформы с Docker

#### GitHub Actions

**Архитектура:**
```
GitHub Repository
       ↓
.github/workflows/ci.yml
       ↓
GitHub Runners (Ubuntu/Windows/macOS)
       ↓
Docker Engine
       ↓
Your Application Container
```

**Особенности:**
- Встроенная поддержка Docker
- Бесплатные публичные репозитории
- Marketplace с готовыми actions
- Matrix builds для разных платформ

#### GitLab CI/CD

**Архитектура:**
```
GitLab Repository
       ↓
.gitlab-ci.yml
       ↓
GitLab Runners
       ↓
Docker-in-Docker (DinD) или Docker Socket
       ↓
Container Registry
```

**Особенности:**
- Встроенный Container Registry
- Kubernetes integration
- Auto DevOps
- Security scanning

#### Jenkins

**Архитектура:**
```
Jenkins Master
       ↓
Pipeline (Jenkinsfile)
       ↓
Jenkins Agents/Slaves
       ↓
Docker Plugin
       ↓
Containers для каждого job
```

**Особенности:**
- Богатая экосистема плагинов
- Гибкость конфигурации
- Distributed builds
- Blue Ocean UI

### Стратегии тестирования в контейнерах

#### Уровни тестирования

```
Testing Pyramid в контейнерах:
                    ┌─────┐
                    │ E2E │ ← Browser tests в контейнерах
                    └─────┘
                  ┌─────────┐
                  │Integration│ ← API tests + DB в контейнерах
                  └─────────┘
              ┌─────────────────┐
              │   Unit Tests    │ ← Быстрые тесты в контейнере
              └─────────────────┘

──────────────────────────────────────────────────────────────

Container Strategy:
═══════════════════

Unit Tests:
┌─────────────────────────────────────┐
│ FROM node:16-alpine                 │
│ COPY package.json .                 │
│ RUN npm install                     │
│ COPY src/ src/                      │
│ RUN npm test                        │
└─────────────────────────────────────┘

Integration Tests:
┌─────────────────────────────────────┐
│ docker-compose:                     │
│   app:                              │
│     build: .                        │
│   database:                         │
│     image: postgres:13              │
│   tests:                            │
│     depends_on: [app, database]     │
│     command: npm run test:integration│
└─────────────────────────────────────┘

E2E Tests:
┌─────────────────────────────────────┐
│   app: (production-like)            │
│   selenium:                         │
│     image: selenium/standalone-chrome│
│   tests:                            │
│     command: npm run test:e2e       │
└─────────────────────────────────────┘
```

#### Тестовые данные и окружения

```
Test Environment Management:
══════════════════════════

Development:
┌─────────────────┐    ┌─────────────────┐
│   App Container │    │ Test Database   │
│   (Hot Reload)  │ ←→ │   (Ephemeral)   │
└─────────────────┘    └─────────────────┘

Staging:
┌─────────────────┐    ┌─────────────────┐
│  App Container  │    │  Staging DB     │
│ (Production-like) ←→ │ (Persistent)    │
└─────────────────┘    └─────────────────┘

Testing:
┌─────────────────┐    ┌─────────────────┐
│  Test Runner    │    │  Mock Services  │
│   Container     │ ←→ │   Container     │
└─────────────────┘    └─────────────────┘
```

### Безопасность в CI/CD

#### Сканирование уязвимостей

```
Security Pipeline:
═══════════════════

Source Code → Build → Scan → Test → Deploy
     ↓           ↓      ↓      ↓       ↓
SAST Tools   Docker   Image  Runtime Security
(SonarQube)   Build   Scan   Tests   Monitoring
             ↓        ↓      ↓       ↓
          Dockerfile  Trivy   DAST   Runtime
          Linting    Snyk   Tools   Protection
```

#### Управление секретами

```
Secrets Management Strategy:
══════════════════════════

❌ НЕ ДЕЛАЙТЕ ТАК:
┌─────────────────────────────────────┐
│ FROM node:16                        │
│ ENV DB_PASSWORD=secret123           │ ← В образе!
│ ENV API_KEY=abc123def456            │ ← Видно всем!
└─────────────────────────────────────┘

✅ ПРАВИЛЬНЫЙ ПОДХОД:
┌─────────────────────────────────────┐
│ CI/CD Variables (encrypted):        │
│ - DB_PASSWORD                       │
│ - API_KEY                           │
│ - DOCKER_REGISTRY_PASSWORD          │
└─────────────────────────────────────┘
              ↓
    Инжекция в runtime
              ↓
┌─────────────────────────────────────┐
│ docker run -e DB_PASSWORD=$DB_PASS  │
│   -e API_KEY=$API_KEY app:latest    │
└─────────────────────────────────────┘
```

### Мониторинг CI/CD процессов

#### Метрики качества пайплайна

```
Pipeline Metrics Dashboard:
══════════════════════════

Build Performance:
┌─────────────────────────────────────┐
│ Build Time Trend     [Graph]        │
│ ┌─┐                                 │
│ │█│   ┌─┐                           │
│ │█│┌─┐│█│     ┌─┐                   │
│ │█││█││█│  ┌─┐│█│                   │
│ └─┘└─┘└─┘  └─┘└─┘                   │
│ Mon Tue Wed Thu Fri                 │
└─────────────────────────────────────┘

Success Rate:
┌─────────────────────────────────────┐
│ ✅ Successful: 95%                  │
│ ❌ Failed: 3%                       │
│ ⚠️  Unstable: 2%                   │
│                                     │
│ [████████████████████░░]            │
└─────────────────────────────────────┘

Container Metrics:
┌─────────────────────────────────────┐
│ • Image Size: 245MB (↓15MB)         │
│ • Layer Count: 8                    │
│ • Vulnerabilities: 2 Low            │
│ • Cache Hit Rate: 78%               │
└─────────────────────────────────────┘
```

---

## 🔧 Глава 5.2: Автоматизация сборки

### Docker BuildKit: Революция в сборке

#### Что такое BuildKit?

```
Traditional Docker Build vs BuildKit:
════════════════════════════════════

Classic Builder:
┌─────────────────────────────────────┐
│ Step 1: FROM node:16                │ ← Последовательно
│ Step 2: COPY package.json           │ ← Один за другим
│ Step 3: RUN npm install             │ ← Без параллелизма
│ Step 4: COPY src/                   │ ← Простое кеширование
└─────────────────────────────────────┘

BuildKit:
┌─────────────────────────────────────┐
│ ┌─Step 1─┐  ┌─Step 3─┐              │ ← Параллельно
│ │FROM    │  │COPY src│              │ ← Где возможно
│ │node:16 │  │        │              │
│ └────────┘  └────────┘              │
│ ┌─Step 2─────────────┐              │
│ │RUN npm install     │              │ ← Умное кеширование
│ │(with mount cache)  │              │
│ └────────────────────┘              │
└─────────────────────────────────────┘
```

#### Преимущества BuildKit

**1. Параллельные сборки**
```
Dependency Graph Building:
═════════════════════════

              ┌─────────────┐
              │ Base Image  │
              │   (FROM)    │
              └──────┬──────┘
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
   ┌──────────┐ ┌─────────┐ ┌──────────┐
   │Dependencies│ │  Code   │ │ Config   │
   │  Install   │ │  Copy   │ │  Copy    │
   └──────────┘ └─────────┘ └──────────┘
          │          │          │
          └──────────┼──────────┘
                     ▼
              ┌─────────────┐
              │Final Image  │
              └─────────────┘

Parallel execution сокращает время сборки на 30-70%!
```

**2. Улучшенное кеширование**
```
Cache Hierarchy:
═══════════════

Local Build Cache:
┌─────────────────────────────────────┐
│ Layer 1: Base OS         [CACHED]   │
│ Layer 2: Dependencies    [CACHED]   │
│ Layer 3: App Code        [REBUILD]  │
│ Layer 4: Config          [REBUILD]  │
└─────────────────────────────────────┘

Remote Cache (Registry):
┌─────────────────────────────────────┐
│ my-registry.com/app:cache           │
│ • Shared между dev, CI, production  │
│ • Автоматическая инвалидация        │
│ • Сжатие и дедупликация             │
└─────────────────────────────────────┘

Mount Cache:
┌─────────────────────────────────────┐
│ RUN --mount=type=cache,target=/npm  │
│   npm install                       │
│ # Кеш npm между сборками!           │
└─────────────────────────────────────┘
```

### Оптимизация Dockerfile

#### Стратегии кеширования слоев

```
❌ Неэффективный Dockerfile:
═══════════════════════════
┌─────────────────────────────────────┐
│ FROM node:16                        │
│ COPY . .                            │ ← Весь код сразу
│ RUN npm install                     │ ← Пересборка при
│ RUN npm run build                   │   любом изменении
└─────────────────────────────────────┘

✅ Оптимизированный:
══════════════════
┌─────────────────────────────────────┐
│ FROM node:16                        │
│ COPY package*.json ./               │ ← Только зависимости
│ RUN npm ci --only=production        │ ← Кешируется долго
│ COPY src/ ./src/                    │ ← Код отдельно
│ RUN npm run build                   │ ← Быстрая пересборка
└─────────────────────────────────────┘

Cache Hit Scenarios:
─────────────────────
Изменение кода:      Layer 1,2,3 ✅ cached
                     Layer 4,5   🔄 rebuild

Изменение зависимостей: Layer 1,2 ✅ cached
                       Layer 3,4,5 🔄 rebuild
```

#### Multi-stage builds для оптимизации

```
Production-Ready Multi-Stage:
════════════════════════════

┌─ Build Stage ─────────────────────────────────────────┐
│ FROM node:16 AS builder                               │
│ WORKDIR /app                                          │
│ COPY package*.json ./                                 │
│ RUN npm ci                           # Все зависимости│
│ COPY src/ ./src/                                      │
│ COPY public/ ./public/                                │
│ RUN npm run build                    # Сборка для prod│
│ RUN npm prune --production           # Удаляем dev deps│
└───────────────────────────────────────────────────────┘
                           │
                           ▼ Copy artifacts
┌─ Production Stage ────────────────────────────────────┐
│ FROM node:16-alpine AS production    # Меньший образ  │
│ WORKDIR /app                                          │
│ COPY --from=builder /app/dist ./dist                  │
│ COPY --from=builder /app/node_modules ./node_modules  │
│ COPY --from=builder /app/package.json ./              │
│ EXPOSE 3000                                           │
│ CMD ["node", "dist/server.js"]                        │
└───────────────────────────────────────────────────────┘

Size Comparison:
───────────────
Builder stage:  1.2GB (with dev tools, source code)
Production:     145MB (only runtime essentials)
Reduction:      88% smaller! 
```

### Продвинутые техники BuildKit

#### Mount кеши

```
Build Cache Types:
═════════════════

1. Cache Mount (для пакетных менеджеров):
┌─────────────────────────────────────┐
│ RUN --mount=type=cache,target=/root/.npm \│
│     npm install                     │
│ # npm cache переживает пересборки!  │
└─────────────────────────────────────┘

2. Bind Mount (для локальной разработки):
┌─────────────────────────────────────┐
│ RUN --mount=type=bind,source=.,target=/src \│
│     --mount=type=cache,target=/cache \│
│     go build -cache=/cache -o /app /src│
└─────────────────────────────────────┘

3. Secret Mount (для приватных репозиториев):
┌─────────────────────────────────────┐
│ RUN --mount=type=secret,id=github_token \│
│     git clone https://$(cat /run/secrets/github_token)@github.com/private/repo.git│
└─────────────────────────────────────┘
```

#### Условные сборки

```
Dynamic Dockerfile Logic:
════════════════════════

Target-Based Building:
┌─────────────────────────────────────┐
│ FROM node:16 AS base                │
│ # Общие слои                        │
│                                     │
│ FROM base AS development            │
│ RUN npm install                     │ ← Dev зависимости
│ CMD ["npm", "run", "dev"]           │
│                                     │
│ FROM base AS production             │
│ RUN npm ci --only=production        │ ← Только prod
│ CMD ["npm", "start"]                │
└─────────────────────────────────────┘

Build Commands:
──────────────
docker build --target development .    # Для разработки
docker build --target production .     # Для продакшена
docker build .                         # Последний target (production)
```

### CI/CD оптимизации

#### Registry cache strategy

```
Multi-Registry Caching:
══════════════════════

Local Developer Machine:
┌─────────────────────────────────────┐
│ docker build                        │
│   --cache-from my-app:latest        │ ← Локальный кеш
│   --tag my-app:new .                │
└─────────────────────────────────────┘

CI/CD Pipeline:
┌─────────────────────────────────────┐
│ docker build                        │
│   --cache-from registry/my-app:cache│ ← Registry кеш
│   --cache-from registry/my-app:latest│ ← Fallback
│   --tag my-app:${BUILD_ID}          │
│   --push                            │
└─────────────────────────────────────┘

Cache Push Strategy:
┌─────────────────────────────────────┐
│ docker build                        │
│   --cache-to type=registry,ref=registry/my-app:cache,mode=max│
│   --cache-from type=registry,ref=registry/my-app:cache│
│   .                                 │
└─────────────────────────────────────┘
```

#### Parallel builds в CI

```
Matrix Build Strategy:
═════════════════════

Single Platform (Sequential):
Build → Test → Push
  ↓      ↓      ↓
 5min   3min   2min = 10min total

Multi-Platform (Parallel):
    ┌─ linux/amd64 ─┐
    │ Build → Test  │ ── Push
    └───────────────┘   ↗
Build                  ↗
    ┌─ linux/arm64 ─┐ ↗
    │ Build → Test  │──
    └───────────────┘
      6min total (40% faster!)

GitHub Actions Example:
─────────────────────
strategy:
  matrix:
    platform: [linux/amd64, linux/arm64]
    node-version: [14, 16, 18]
# 6 parallel jobs вместо последовательных!
```

---

## 📦 Глава 5.3: Реестры и развертывание

### Container Registry Ecosystem

#### Типы реестров

```
Registry Landscape:
══════════════════

Public Registries:
┌─────────────────────────────────────┐
│ Docker Hub          [docker.io]     │ ← Самый популярный
│ • 100+ млн образов                  │
│ • Official images                   │
│ • Автоматические сборки             │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│ GitHub Packages     [ghcr.io]       │ ← Интеграция с Git
│ • Приватные репозитории             │
│ • Встроенная безопасность           │
│ • Workflow интеграция               │
└─────────────────────────────────────┘

Cloud Registries:
┌─────────────────────────────────────┐
│ AWS ECR            [account.dkr.ecr]│
│ Google GCR         [gcr.io]         │
│ Azure ACR          [registry.azurecr]│
│ • Управляемые сервисы               │
│ • Интеграция с облачными платформами│
│ • Enterprise security               │
└─────────────────────────────────────┘

Private/On-Premise:
┌─────────────────────────────────────┐
│ Harbor             [Enterprise]     │
│ Nexus Repository   [Sonatype]       │
│ Docker Registry    [Open Source]    │
│ • Полный контроль                   │
│ • Соответствие требованиям          │
│ • Кастомные политики                │
└─────────────────────────────────────┘
```

#### Registry архитектура

```
Docker Registry v2 Architecture:
═══════════════════════════════

Client (docker CLI)
         │
         ▼ HTTPS/JSON API
┌─────────────────┐
│  Registry API   │
│   (Port 5000)   │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐    ┌─────────────────┐
│   Blob Store    │    │   Metadata      │
│  (Layer Data)   │    │   Database      │
│                 │    │ (Manifest, Tags)│
└─────────────────┘    └─────────────────┘
          │                      │
          ▼                      ▼
┌─────────────────┐    ┌─────────────────┐
│ Storage Backend │    │     Redis/      │
│ • Local FS      │    │   PostgreSQL    │
│ • S3 / GCS      │    │                 │
│ • Azure Blob    │    │                 │
└─────────────────┘    └─────────────────┘
```

### Стратегии управления версиями

#### Semantic Versioning для образов

```
Container Versioning Strategy:
═════════════════════════════

Git Flow → Container Tags:
┌─────────────────────────────────────┐
│ main branch      → latest, stable   │
│ develop branch   → dev, nightly     │
│ feature/auth     → feature-auth     │
│ release/v2.1     → v2.1.0-rc1       │
│ hotfix/security  → v2.0.1-hotfix    │
└─────────────────────────────────────┘

Tag Examples:
────────────
myapp:latest              ← Последняя стабильная
myapp:v2.1.0              ← Конкретная версия
myapp:v2.1                ← Minor версия
myapp:v2                  ← Major версия
myapp:2021-03-15          ← По дате
myapp:abc123f             ← Git commit SHA
myapp:pr-42               ← Pull request build

Multi-Architecture Tags:
───────────────────────
myapp:v2.1.0              ← Manifest list
├── linux/amd64           ← Intel/AMD 64-bit
├── linux/arm64           ← ARM 64-bit (Apple M1)
├── linux/arm/v7          ← ARM 32-bit
└── windows/amd64         ← Windows containers
```

#### Lifecycle управление

```
Image Lifecycle Management:
══════════════════════════

Development Cycle:
┌─────────────────────────────────────┐
│ Feature Branch                      │
│ ├── myapp:feature-auth-build-123    │ ← Эфемерные
│ ├── myapp:feature-auth-build-124    │
│ └── myapp:feature-auth-build-125    │
└─────────────────────────────────────┘
              │ Автоудаление через 7 дней
              ▼
┌─────────────────────────────────────┐
│ Release                             │
│ ├── myapp:v2.1.0-rc1                │ ← Кандидаты
│ ├── myapp:v2.1.0-rc2                │
│ └── myapp:v2.1.0                    │ ← Stable
└─────────────────────────────────────┘
              │ Хранятся долгосрочно
              ▼
┌─────────────────────────────────────┐
│ Production                          │
│ ├── myapp:v2.0.0 (deprecated)      │ ← Устаревшие
│ ├── myapp:v2.1.0 (current)         │ ← Текущая
│ └── myapp:latest → v2.1.0           │ ← Алиас
└─────────────────────────────────────┘

Retention Policies:
──────────────────
• Feature builds: 7 дней
• Release candidates: 30 дней
• Stable releases: 2 года
• Security patches: Бессрочно
```

### Стратегии развертывания

#### Blue-Green Deployment

```
Blue-Green Deployment Strategy:
══════════════════════════════

Step 1: Blue Environment (Current)
┌─────────────────────────────────────┐
│        Load Balancer                │
│             │                       │
│             ▼ 100% traffic          │
│   ┌─────────────────┐               │
│   │ Blue Environment│               │
│   │   app:v1.0      │               │
│   └─────────────────┘               │
│                                     │
│   ┌─────────────────┐               │
│   │Green Environment│ ← Standby     │
│   │   (empty)       │               │
│   └─────────────────┘               │
└─────────────────────────────────────┘

Step 2: Deploy to Green
┌─────────────────────────────────────┐
│        Load Balancer                │
│             │                       │
│             ▼ 100% traffic          │
│   ┌─────────────────┐               │
│   │ Blue Environment│               │
│   │   app:v1.0      │               │
│   └─────────────────┘               │
│                                     │
│   ┌─────────────────┐               │
│   │Green Environment│ ← Deploy v2.0 │
│   │   app:v2.0      │   Test        │
│   └─────────────────┘               │
└─────────────────────────────────────┘

Step 3: Switch Traffic
┌─────────────────────────────────────┐
│        Load Balancer                │
│             │                       │
│             ▼ 100% traffic          │
│   ┌─────────────────┐               │
│   │ Blue Environment│ ← Standby     │
│   │   app:v1.0      │               │
│   └─────────────────┘               │
│                                     │
│   ┌─────────────────┐               │
│   │Green Environment│ ← Active      │
│   │   app:v2.0      │               │
│   └─────────────────┘               │
└─────────────────────────────────────┘

Benefits:
- Мгновенный rollback
- Zero downtime
- Полное тестирование перед переключением
- Простота автоматизации
```

#### Rolling Deployment

```
Rolling Deployment Strategy:
═══════════════════════════

Initial State (3 instances of v1.0):
┌─────┐ ┌─────┐ ┌─────┐
│ v1.0│ │ v1.0│ │ v1.0│
└─────┘ └─────┘ └─────┘
   ▲       ▲       ▲
   │       │       │
 ┌─────────────────────┐
 │   Load Balancer     │ ← 100% traffic
 └─────────────────────┘

Step 1: Replace first instance
┌─────┐ ┌─────┐ ┌─────┐
│ v2.0│ │ v1.0│ │ v1.0│ ← 33% v2.0
└─────┘ └─────┘ └─────┘   67% v1.0
   ▲       ▲       ▲
   │       │       │
 ┌─────────────────────┐
 │   Load Balancer     │
 └─────────────────────┘

Step 2: Replace second instance
┌─────┐ ┌─────┐ ┌─────┐
│ v2.0│ │ v2.0│ │ v1.0│ ← 67% v2.0
└─────┘ └─────┘ └─────┘   33% v1.0
   ▲       ▲       ▲
   │       │       │
 ┌─────────────────────┐
 │   Load Balancer     │
 └─────────────────────┘

Step 3: Replace last instance
┌─────┐ ┌─────┐ ┌─────┐
│ v2.0│ │ v2.0│ │ v2.0│ ← 100% v2.0
└─────┘ └─────┘ └─────┘
   ▲       ▲       ▲
   │       │       │
 ┌─────────────────────┐
 │   Load Balancer     │
 └─────────────────────┘

Benefits:
- Постепенное обновление
- Меньше ресурсов
- Раннее обнаружение проблем
- Автоматический rollback при ошибках
```

#### Canary Deployment

```
Canary Deployment Strategy:
══════════════════════════

Phase 1: 5% Canary Traffic
┌─────────────────────────────────────┐
│           Load Balancer             │
│              /    \                 │
│          95% /      \ 5%            │
│             ▼        ▼              │
│   ┌──────────────┐ ┌──────────────┐ │
│   │ Stable v1.0  │ │ Canary v2.0  │ │
│   │ (19 instances)│ │ (1 instance) │ │
│   └──────────────┘ └──────────────┘ │
└─────────────────────────────────────┘
            │              │
            ▼              ▼
    ┌─────────────┐ ┌─────────────┐
    │  Metrics    │ │  Metrics    │
    │ Error: 0.1% │ │Error: 0.1%  │ ← Мониторинг
    │ Latency:50ms│ │Latency:52ms │
    └─────────────┘ └─────────────┘

Phase 2: 25% Canary (если успешно)
┌─────────────────────────────────────┐
│           Load Balancer             │
│              /    \                 │
│          75% /      \ 25%           │
│             ▼        ▼              │
│   ┌──────────────┐ ┌──────────────┐ │
│   │ Stable v1.0  │ │ Canary v2.0  │ │
│   │ (15 instances)│ │ (5 instances)│ │
│   └──────────────┘ └──────────────┘ │
└─────────────────────────────────────┘

Phase 3: 100% (если все метрики ОК)
┌─────────────────────────────────────┐
│           Load Balancer             │
│                │                    │
│             100% ▼                  │
│      ┌──────────────────┐           │
│      │   New Stable     │           │
│      │     v2.0         │           │
│      │  (20 instances)  │           │
│      └──────────────────┘           │
└─────────────────────────────────────┘

Automated Decision Making:
─────────────────────────
if error_rate > 1% OR latency > 100ms:
    → Rollback canary
    → Alert team
else if success_metrics_for > 10min:
    → Increase canary traffic
    → Continue monitoring
```

### Enterprise Registry Features

#### Security и Compliance

```
Enterprise Registry Security:
════════════════════════════

Multi-Layer Security:
┌─────────────────────────────────────┐
│ 1. Authentication & Authorization   │
│   ├── LDAP/AD Integration           │
│   ├── RBAC (Role-Based Access)      │
│   ├── Team/Project Permissions      │
│   └── API Token Management          │
└─────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│ 2. Image Security Scanning          │
│   ├── CVE Database Updates          │
│   ├── License Compliance Check      │
│   ├── Malware Detection             │
│   └── Policy Enforcement            │
└─────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│ 3. Content Trust & Signing          │
│   ├── Docker Content Trust (DCT)    │
│   ├── Notary Integration            │
│   ├── Image Signing Verification    │
│   └── Supply Chain Security         │
└─────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│ 4. Audit & Compliance               │
│   ├── Access Logs                   │
│   ├── Pull/Push Tracking            │
│   ├── Compliance Reports            │
│   └── Retention Policies            │
└─────────────────────────────────────┘
```

#### High Availability архитектура

```
HA Registry Architecture:
════════════════════════

                Internet
                    │
                    ▼
            ┌─────────────┐
            │Load Balancer│
            └─────┬───────┘
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
 ┌──────────┐┌──────────┐┌──────────┐
 │Registry  ││Registry  ││Registry  │
 │Instance 1││Instance 2││Instance 3│
 └────┬─────┘└────┬─────┘└────┬─────┘
      │           │           │
      └─────────┬─┴─────┬─────┘
                │       │
                ▼       ▼
        ┌─────────────────────┐
        │  Shared Storage     │
        │  (S3/GCS/Azure)     │
        └─────────────────────┘
                │
                ▼
        ┌─────────────────────┐
        │    Database         │
        │   (PostgreSQL       │
        │    with Replica)    │
        └─────────────────────┘

Benefits:
─────────
• Zero downtime maintenance
• Geographic distribution
• Automatic failover
• Horizontal scaling
• Disaster recovery
```

---

## 📈 Заключение блока

### Ключевые навыки, которые вы освоили

```
CI/CD Mastery Checklist:
═══════════════════════

✅ Pipeline Design
  ├── Multi-stage builds
  ├── Parallel execution
  ├── Cache optimization
  └── Security integration

✅ Build Automation
  ├── BuildKit features
  ├── Multi-arch builds
  ├── Registry management
  └── Performance tuning

✅ Deployment Strategies
  ├── Blue-Green deployments
  ├── Rolling updates
  ├── Canary releases
  └── Rollback procedures

✅ Production Readiness
  ├── Monitoring integration
  ├── Security scanning
  ├── Compliance requirements
  └── Disaster recovery
```

### Практические результаты

После завершения этого блока вы сможете:

1. **Создавать production-ready CI/CD пайплайны** с использованием любой популярной платформы
2. **Оптимизировать время сборки** на 50-80% с помощью продвинутых техник кеширования
3. **Безопасно управлять образами** через весь жизненный цикл разработки
4. **Реализовывать стратегии развертывания** без простоя сервисов
5. **Настраивать мониторинг и алерты** для всех этапов пайплайна

### Переход к следующему блоку

В **Блоке 6: Мониторинг и production** мы углубимся в:
- Централизованное логирование с ELK Stack
- Мониторинг метрик с Prometheus и Grafana
- Оптимизацию производительности контейнеров
- Production-ready конфигурации
- Disaster recovery стратегии

---

### 💡 Практические рекомендации

```
Best Practices Summary:
══════════════════════

🚀 Performance
  • Используйте BuildKit для всех сборок
  • Оптимизируйте порядок команд в Dockerfile
  • Применяйте multi-stage builds
  • Настройте registry caching

🔒 Security
  • Сканируйте образы на уязвимости
  • Используйте secrets management
  • Не включайте секреты в образы
  • Регулярно обновляйте базовые образы

📊 Monitoring
  • Мониторьте время сборки
  • Отслеживайте размер образов
  • Логируйте все изменения
  • Настройте алерты на неудачи

🔄 Automation
  • Автоматизируйте тестирование
  • Используйте GitOps подход
  • Настройте automatic rollbacks
  • Документируйте процессы
```

Теперь вы готовы к следующему этапу изучения Docker - производственному мониторингу и оптимизации!