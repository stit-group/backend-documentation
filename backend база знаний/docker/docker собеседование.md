# Компактный Docker Interview Guide для Backend разработчика
## 50 ключевых вопросов + практические задания

---

## 🎯 Структура интервью (60-90 минут)

| Уровень | Время | Вопросы | Фокус |
|---------|-------|---------|-------|
| **Junior** | 20 мин | 1-15 | Основы, базовые команды |
| **Middle** | 25 мин | 16-35 | Production опыт, Compose |
| **Senior** | 30 мин | 36-50 | Архитектура, безопасность, troubleshooting |
| **Практика** | 15 мин | Кодинг | Dockerfile + задача |

---

## 📊 Быстрая оценка кандидата

### Must-Know чек-лист ✅
- [ ] Разница между образом и контейнером
- [ ] Базовые команды Docker CLI
- [ ] Понимание Dockerfile инструкций
- [ ] Volumes vs bind mounts
- [ ] Docker networks основы
- [ ] Production best practices

### Красные флаги 🚩
- Не знает разницу между CMD и ENTRYPOINT
- Использует latest тег в production
- Не понимает layer caching
- Запускает контейнеры от root без объяснения
- Не знает про .dockerignore

---

## JUNIOR LEVEL (1-15)

### 1. **Что такое Docker и какие проблемы он решает?**
**Углубляющие вопросы:**
- Чем отличается от виртуальных машин?
- Что означает "Build once, run anywhere"?
- Назовите альтернативы Docker

### 2. **Объясните разницу между Docker образом и контейнером**
**Углубляющие вопросы:**
- Как создается контейнер из образа?
- Можно ли из одного образа запустить несколько контейнеров?
- Что происходит с изменениями в контейнере?

### 3. **Основные команды Docker CLI**
**Практическое задание:**
```bash
# Объясните что делает каждая команда
docker pull nginx:alpine
docker run -d -p 8080:80 --name web nginx:alpine
docker exec -it web sh
docker logs web
docker stop web && docker rm web
```

### 4. **Что такое Dockerfile и основные инструкции?**
**Углубляющие вопросы:**
- Разница между RUN, CMD и ENTRYPOINT?
- Зачем нужен WORKDIR?
- Что делает EXPOSE?

### 5. **Объясните Docker слои (layers)**
**Углубляющие вопросы:**
- Как работает кеширование слоев?
- Что происходит при изменении одного слоя?
- Как оптимизировать количество слоев?

### 6. **Что такое .dockerignore?**
**Углубляющие вопросы:**
- Зачем нужен .dockerignore?
- Как он влияет на build context?

### 7. **Разница между ADD и COPY**
**Углубляющие вопросы:**
- Когда использовать ADD?
- Какие проблемы безопасности может создать ADD?

### 8. **Что такое Docker Hub?**
**Углубляющие вопросы:**
- Как публиковать образы?
- Что такое automated builds?

### 9. **Базовые типы Docker сетей**
**Углубляющие вопросы:**
- Разница между bridge, host, none?
- Как контейнеры общаются друг с другом?

### 10. **Что такое Docker volumes?**
**Углубляющие вопросы:**
- Разница между volumes и bind mounts?
- Когда данные сохраняются после удаления контейнера?

### 11. **Environment variables в Docker**
**Практическое задание:**
```dockerfile
# Объясните как работают env vars в этом Dockerfile
FROM node:alpine
ENV NODE_ENV=production
ARG VERSION=1.0.0
COPY package.json .
RUN npm install
CMD ["npm", "start"]
```

### 12. **Что такое port mapping?**
**Углубляющие вопросы:**
- Разница между EXPOSE и -p флагом?
- Можно ли мапить один порт на несколько контейнеров?

### 13. **Docker образы: тегирование и версионирование**
**Углубляющие вопросы:**
- Почему опасно использовать latest в production?
- Как правильно версионировать образы?

### 14. **Жизненный цикл контейнера**
**Углубляющие вопросы:**
- Какие состояния может иметь контейнер?
- Разница между stop и kill?

### 15. **Логи контейнеров**
**Углубляющие вопросы:**
- Где хранятся логи по умолчанию?
- Как настроить ротацию логов?

---

## MIDDLE LEVEL (16-35)

### 16. **Напишите оптимальный Dockerfile для Node.js приложения**
**Практическое задание:**
```
Требования:
- Multi-stage build
- Минимальный размер образа
- Не root пользователь
- Правильное кеширование dependencies
```

### 17. **Multi-stage builds: концепция и применение**
**Углубляющие вопросы:**
- Как передавать артефакты между стадиями?
- Преимущества для production образов?

### 18. **Docker Compose основы**
**Практическое задание:**
```yaml
# Объясните и улучшите этот compose файл
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: password
```

### 19. **Зависимости между сервисами в Compose**
**Углубляющие вопросы:**
- Ограничения depends_on?
- Как дождаться готовности базы данных?

### 20. **Docker networks в production**
**Углубляющие вопросы:**
- Custom bridge networks vs default?
- Как изолировать сервисы?

### 21. **Volumes в production: стратегии и backup**
**Углубляющие вопросы:**
- Как backup Docker volumes?
- Named volumes vs bind mounts для production?

### 22. **Мониторинг ресурсов контейнеров**
**Углубляющие вопросы:**
- Как ограничить память и CPU?
- Что происходит при OOMKilled?

### 23. **Health checks в Docker**
**Практическое задание:**
```dockerfile
# Добавьте health check для web приложения
FROM nginx
COPY index.html /usr/share/nginx/html/
EXPOSE 80
# Ваш health check здесь
```

### 24. **Docker Registry: приватные образы**
**Углубляющие вопросы:**
- Как настроить аутентификацию?
- Pull secrets в production?

### 25. **Secrets в Docker**
**Углубляющие вопросы:**
- Проблемы с environment variables для секретов?
- Docker secrets vs external secret management?

### 26. **Restart policies**
**Углубляющие вопросы:**
- Разные типы restart policy?
- Интеграция с health checks?

### 27. **BuildKit vs legacy builder**
**Углубляющие вопросы:**
- Преимущества BuildKit?
- Build secrets и cache mounts?

### 28. **Оптимизация времени сборки**
**Углубляющие вопросы:**
- Layer caching стратегии?
- Параллельная сборка?

### 29. **Docker в CI/CD pipeline**
**Углубляющие вопросы:**
- Docker-in-Docker проблемы?
- Стратегии для automated testing?

### 30. **Compose override файлы**
**Углубляющие вопросы:**
- Разные окружения (dev/staging/prod)?
- Порядок загрузки файлов?

### 31. **Работа с базами данных в контейнерах**
**Углубляющие вопросы:**
- Data persistence стратегии?
- Init scripts и migrations?

### 32. **Docker образы: анализ и оптимизация**
**Практическое задание:**
```bash
# Проанализируйте образ и предложите оптимизации
docker history my-app:latest
docker run --rm -it my-app:latest sh
```

### 33. **Container security basics**
**Углубляющие вопросы:**
- Почему не стоит запускать от root?
- Как создать non-root пользователя?

### 34. **Scaling в Docker Compose**
**Углубляющие вопросы:**
- Ограничения scaling в Compose?
- Load balancing между instances?

### 35. **Troubleshooting: common issues**
**Практическое задание:**
```
Сценарий: Контейнер запускается, но приложение недоступно
Как будете диагностировать проблему?
```

---

## SENIOR LEVEL (36-50)

### 36. **Production-ready Docker setup**
**Углубляющие вопросы:**
- Настройки Docker daemon для production?
- Logging drivers и централизованные логи?

### 37. **Orchestration platforms сравнение**
**Углубляющие вопросы:**
- Docker Swarm vs Kubernetes: когда что выбрать?
- Migration стратегии между платформами?

### 38. **Zero-downtime deployments**
**Углубляющие вопросы:**
- Rolling updates реализация?
- Health check стратегии для deployments?

### 39. **Advanced networking: overlay networks**
**Углубляющие вопросы:**
- VXLAN и multi-host networking?
- Network encryption в production?

### 40. **Security: advanced topics**
**Практическое задание:**
```dockerfile
# Оцените безопасность этого Dockerfile и улучшите
FROM ubuntu:latest
RUN apt-get update && apt-get install -y curl
COPY app /app
USER root
CMD ["/app"]
```

### 41. **Resource management и optimization**
**Углубляющие вопросы:**
- CPU shares vs CPU limits?
- Memory pressure и swap handling?

### 42. **Storage в distributed environments**
**Углубляющие вопросы:**
- Distributed file systems для containers?
- CSI и cloud storage integration?

### 43. **Monitoring и observability**
**Углубляющие вопросы:**
- Metrics collection для container workloads?
- Distributed tracing в микросервисах?

### 44. **Performance troubleshooting**
**Практическое задание:**
```
Сценарий: Приложение в контейнере работает медленно
Какие метрики будете анализировать?
Какие инструменты использовать?
```

### 45. **Custom network drivers**
**Углубляющие вопросы:**
- Когда нужны custom drivers?
- Macvlan vs bridge для specific use cases?

### 46. **Advanced Dockerfile techniques**
**Углубляющие вопросы:**
- BuildKit cache mounts и секреты?
- Cross-platform builds?

### 47. **Capacity planning для containers**
**Углубляющие вопросы:**
- Resource calculation методики?
- Bin packing и scheduling optimization?

### 48. **Security scanning и compliance**
**Углубляющие вопросы:**
- Image vulnerability scanning automation?
- CIS benchmarks для Docker?

### 49. **Disaster recovery для containerized apps**
**Углубляющие вопросы:**
- Backup стратегии для distributed applications?
- RTO/RPO considerations?

### 50. **Architecture design: микросервисы в production**
**Комплексное задание:**
```
Спроектируйте Docker-based решение для:
- E-commerce платформа
- 10 микросервисов
- 100k+ requests/day
- Multi-region deployment

Включите:
- Service discovery
- Load balancing
- Data persistence
- Monitoring
- Security
```

---

## 🛠 Практические задания

### Задание 1: Dockerfile Challenge (15 минут)
```
Создайте production-ready Dockerfile для Python FastAPI приложения:

Требования:
- Multi-stage build
- Размер образа < 100MB
- Non-root пользователь
- Health check
- Оптимизация кеширования dependencies

Приложение:
- requirements.txt с dependencies
- main.py с FastAPI app
- Порт 8000
```

### Задание 2: Docker Compose Architecture (10 минут)
```
Спроектируйте docker-compose.yml для:
- Web приложение (Python/Node.js)
- PostgreSQL база данных
- Redis cache
- Nginx reverse proxy

Требования:
- Разные сети для изоляции
- Persistent volumes
- Environment variables
- Health checks
- Development overrides
```

### Задание 3: Troubleshooting Scenario (10 минут)
```
Проблема: После деплоя новой версии приложения:
- Контейнер запускается успешно
- Health check проходит
- Но пользователи получают 502 ошибки

docker logs показывает:
"Application started on port 8000"
"Database connection established"

Как будете диагностировать?
```

---

## 🎯 Критерии оценки

### Junior (15-25 правильных ответов)
**Должен знать:**
- Основы Docker концепций
- Базовые команды CLI
- Простой Dockerfile
- Docker Compose basics

**Может делать:**
- Запускать готовые образы
- Писать простые Dockerfile
- Использовать volumes и networks
- Работать с готовыми compose файлами

### Middle (30-40 правильных ответов)
**Должен знать:**
- Production best practices
- Multi-stage builds
- Security basics
- Monitoring и логирование

**Может делать:**
- Оптимизировать образы
- Настраивать CI/CD с Docker
- Решать стандартные production задачи
- Troubleshoot базовые проблемы

### Senior (40+ правильных ответов)
**Должен знать:**
- Advanced networking и storage
- Orchestration platforms
- Performance optimization
- Security advanced topics

**Может делать:**
- Проектировать container архитектуру
- Решать сложные performance проблемы
- Внедрять security best practices
- Mentoring команды по Docker

---

## 💡 Советы интервьюеру

### ✅ Хорошие признаки:
- Упоминает security considerations без подсказок
- Объясняет trade-offs разных подходов
- Приводит примеры из реального опыта
- Задает уточняющие вопросы о requirements

### ⚠️ Предупреждающие знаки:
- Использует latest тег везде
- Не знает про layer caching
- Игнорирует security аспекты
- Не может объяснить разницу между dev и prod setup

### 🎭 Адаптация под роль:
- **DevOps фокус**: Больше вопросов по orchestration, monitoring, CI/CD
- **Backend фокус**: Акцент на application containerization, databases, APIs
- **Full-stack фокус**: Включить frontend containerization, nginx configuration

---

## 📚 Дополнительные ресурсы

**Для кандидата:**
- Docker official documentation
- Best practices guides
- Security benchmarks

**Для интервьюера:**
- Обновлять вопросы раз в полгода
- Адаптировать под конкретный tech stack
- Включать real-world сценарии из вашей компании