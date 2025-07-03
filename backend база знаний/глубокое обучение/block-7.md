# Блок 6: Промышленная разработка и deployment

**⏱ Длительность:** 4-6 недель  
**🎯 Цель:** Научиться внедрять модели в продакшн и работать с реальными проектами

---

## 📊 Структура блока

```
Блок 6: Промышленная разработка
├── 6.1 MLOps и версионирование (1 неделя)
├── 6.2 Оптимизация моделей (1 неделя)
├── 6.3 Развертывание моделей (1-2 недели)
├── 6.4 Мониторинг и A/B тестирование (1 неделя)
├── 6.5 Этика ИИ и безопасность (1 неделя)
└── 6.6 Создание портфолио (1 неделя)
```

---

## 🔧 Глава 6.1: MLOps и версионирование

### 📋 Концепция MLOps

**MLOps** = Machine Learning + DevOps + Data Engineering

```
Традиционная разработка          MLOps разработка
┌─────────────────────┐         ┌─────────────────────┐
│   Код → Тест → Деплой   │    →    │ Данные + Код + Модель │
└─────────────────────┘         └─────────────────────┘
                                           ↓
                                 ┌─────────────────────┐
                                 │ Мониторинг + Ретрейн │
                                 └─────────────────────┘
```

### 🎯 Основные принципы

**1. Версионирование всего:**
- ✅ Код (Git)
- ✅ Данные (DVC)
- ✅ Модели (MLflow)
- ✅ Окружение (Docker)

**2. Автоматизация:**
- ✅ CI/CD пайплайны
- ✅ Автоматическое тестирование
- ✅ Автоматический деплой

**3. Воспроизводимость:**
- ✅ Фиксированные seed
- ✅ Версии библиотек
- ✅ Параметры эксперимента

---

### 🛠 Настройка Git для ML

```bash
# .gitignore для ML проектов
*.pyc
__pycache__/
.jupyter/
.ipynb_checkpoints/
data/raw/          # Сырые данные не коммитим
models/large/      # Большие модели
logs/
.env
```

### 📦 DVC для версионирования данных

```bash
# Инициализация DVC
dvc init
dvc remote add -d storage s3://my-bucket/data

# Добавление данных под версионирование
dvc add data/train.csv
git add data/train.csv.dvc .gitignore
git commit -m "Add training data"

# Отправка данных в хранилище
dvc push
```

### 📊 MLflow для tracking экспериментов

```python
import mlflow
import mlflow.pytorch

# Начало эксперимента
mlflow.start_run()

# Логирование параметров
mlflow.log_param("learning_rate", 0.01)
mlflow.log_param("epochs", 10)

# Логирование метрик
mlflow.log_metric("accuracy", 0.95)
mlflow.log_metric("loss", 0.05)

# Сохранение модели
mlflow.pytorch.log_model(model, "model")

mlflow.end_run()
```

---

## ⚡ Глава 6.2: Оптимизация и квантизация моделей

### 🎯 Почему важна оптимизация?

```
Обычная модель           Оптимизированная модель
┌─────────────────┐      ┌─────────────────┐
│  Размер: 500MB  │  →   │  Размер: 50MB   │
│  Время: 100ms   │      │  Время: 10ms    │
│  Память: 2GB    │      │  Память: 200MB  │
└─────────────────┘      └─────────────────┘
```

### 🔍 Основные методы оптимизации

**1. Pruning (Обрезка)**
```
Полная нейронная сеть          Pruned сеть
┌─ ● ── ● ── ● ─┐              ┌─ ● ── ✗ ── ● ─┐
│   │    │    │  │              │   │         │  │
├─ ● ── ● ── ● ─┤          →   ├─ ● ── ● ── ● ─┤
│   │    │    │  │              │   │    │    │  │
└─ ● ── ● ── ● ─┘              └─ ● ── ● ── ✗ ─┘
```

**2. Quantization (Квантизация)**
```
Float32 (4 байта)              Int8 (1 байт)
┌─────────────────┐           ┌─────────────────┐
│  0.7834567891   │     →     │       127       │
│  -0.123456789   │           │       -20       │
└─────────────────┘           └─────────────────┘
```

**3. Knowledge Distillation**
```
Teacher Model (большая)        Student Model (маленькая)
┌─────────────────┐           ┌─────────────────┐
│     ResNet50    │   учит    │   MobileNet     │
│   (25M params)  │    →      │   (4M params)   │
└─────────────────┘           └─────────────────┘
```

### 💻 Практическая реализация

```python
import torch
import torch.nn.utils.prune as prune

# Pruning - удаление 20% весов
def prune_model(model, amount=0.2):
    for name, module in model.named_modules():
        if isinstance(module, torch.nn.Linear):
            prune.l1_unstructured(module, name='weight', amount=amount)
    return model

# Квантизация модели
def quantize_model(model):
    model.qconfig = torch.quantization.get_default_qconfig('fbgemm')
    torch.quantization.prepare(model, inplace=True)
    
    # Калибровка на данных
    with torch.no_grad():
        for data, _ in calibration_loader:
            model(data)
    
    torch.quantization.convert(model, inplace=True)
    return model
```

---

## 🚀 Глава 6.3: Развертывание моделей

### 🏗 Архитектура развертывания

```
Клиент (Web/Mobile)
        ↓
┌─────────────────┐
│   Load Balancer  │
└─────────────────┘
        ↓
┌─────────────────┐
│   API Gateway    │
└─────────────────┘
        ↓
┌─────────────────┐
│   ML Service     │
│   (FastAPI)      │
└─────────────────┘
        ↓
┌─────────────────┐
│   Model Store    │
│   (Redis/S3)     │
└─────────────────┘
```

### 🔧 FastAPI для ML API

```python
from fastapi import FastAPI, File, UploadFile
import torch
import numpy as np
from PIL import Image

app = FastAPI()

# Загрузка модели при старте
model = torch.load('model.pth')
model.eval()

@app.post("/predict")
async def predict(file: UploadFile = File(...)):
    # Предобработка изображения
    image = Image.open(file.file)
    image = preprocess(image)
    
    # Предсказание
    with torch.no_grad():
        prediction = model(image)
        confidence = torch.softmax(prediction, dim=1)
    
    return {
        "class": int(prediction.argmax()),
        "confidence": float(confidence.max())
    }

@app.get("/health")
async def health_check():
    return {"status": "healthy"}
```

### 🐳 Docker для контейнеризации

```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### ☁️ Развертывание в облаке

```yaml
# kubernetes-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ml-service
  template:
    metadata:
      labels:
        app: ml-service
    spec:
      containers:
      - name: ml-service
        image: ml-service:latest
        ports:
        - containerPort: 8000
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
```

---

## 📈 Глава 6.4: Мониторинг и A/B тестирование

### 🔍 Что мониторить?

```
Система мониторинга ML
├── Инфраструктура
│   ├── CPU, Memory, GPU
│   ├── Латентность API
│   └── Пропускная способность
├── Качество данных
│   ├── Schema validation
│   ├── Data drift
│   └── Outlier detection
└── Качество модели
    ├── Accuracy drift
    ├── Prediction distribution
    └── Business metrics
```

### 📊 Детекция Data Drift

```python
import numpy as np
from scipy import stats

def detect_data_drift(reference_data, current_data, threshold=0.05):
    """
    Детекция drift с помощью KS-test
    """
    drift_detected = {}
    
    for feature in reference_data.columns:
        ks_stat, p_value = stats.ks_2samp(
            reference_data[feature], 
            current_data[feature]
        )
        
        drift_detected[feature] = {
            'drift': p_value < threshold,
            'p_value': p_value,
            'ks_statistic': ks_stat
        }
    
    return drift_detected
```

### 🧪 A/B тестирование ML моделей

```
A/B тест структура
┌─────────────────────────────────────┐
│              Трафик                 │
└─────────────────────────────────────┘
                 ↓
        ┌─────────────────┐
        │   Splitter      │
        │   (50/50)       │
        └─────────────────┘
         ↓               ↓
┌─────────────────┐ ┌─────────────────┐
│   Model A       │ │   Model B       │
│   (Baseline)    │ │   (New)         │
└─────────────────┘ └─────────────────┘
         ↓               ↓
┌─────────────────┐ ┌─────────────────┐
│   Metrics A     │ │   Metrics B     │
│   CTR: 2.1%     │ │   CTR: 2.3%     │
└─────────────────┘ └─────────────────┘
```

### 📊 Метрики для мониторинга

```python
import prometheus_client
from prometheus_client import Counter, Histogram, Gauge

# Создание метрик
REQUEST_COUNT = Counter('ml_requests_total', 'Total requests')
REQUEST_LATENCY = Histogram('ml_request_duration_seconds', 'Request latency')
MODEL_ACCURACY = Gauge('ml_model_accuracy', 'Current model accuracy')

# Использование в коде
@REQUEST_LATENCY.time()
def predict(data):
    REQUEST_COUNT.inc()
    # ... логика предсказания
    return prediction
```

---

## 🛡️ Глава 6.5: Этика ИИ и безопасность

### ⚖️ Источники bias в ML

```
Источники предвзятости
┌─────────────────────────────────────┐
│           Данные                    │
├─────────────────────────────────────┤
│ • Историческая предвзятость         │
│ • Недопредставленность групп        │
│ • Сбор данных в неравных условиях   │
└─────────────────────────────────────┘
                 ↓
┌─────────────────────────────────────┐
│          Алгоритм                   │
├─────────────────────────────────────┤
│ • Неправильная функция потерь       │
│ • Неучет групповой справедливости   │
│ • Прокси-переменные                 │
└─────────────────────────────────────┘
                 ↓
┌─────────────────────────────────────┐
│        Интерпретация                │
├─────────────────────────────────────┤
│ • Неправильная интерпретация        │
│ • Игнорирование ограничений         │
│ • Автоматизация решений             │
└─────────────────────────────────────┘
```

### 🔍 Методы обнаружения bias

```python
def calculate_fairness_metrics(y_true, y_pred, sensitive_attr):
    """
    Расчет метрик справедливости
    """
    metrics = {}
    
    # Demographic parity
    pred_rate_group_0 = np.mean(y_pred[sensitive_attr == 0])
    pred_rate_group_1 = np.mean(y_pred[sensitive_attr == 1])
    metrics['demographic_parity'] = abs(pred_rate_group_0 - pred_rate_group_1)
    
    # Equalized odds
    tpr_group_0 = recall_score(y_true[sensitive_attr == 0], y_pred[sensitive_attr == 0])
    tpr_group_1 = recall_score(y_true[sensitive_attr == 1], y_pred[sensitive_attr == 1])
    metrics['equalized_odds'] = abs(tpr_group_0 - tpr_group_1)
    
    return metrics
```

### 🛡️ Защита от adversarial атак

```
Adversarial атака
┌─────────────────┐     ┌─────────────────┐
│  Исходное       │     │  Измененное     │
│  изображение    │  →  │  изображение    │
│  (кот)          │     │  (собака)       │
└─────────────────┘     └─────────────────┘
       ↓                        ↓
┌─────────────────┐     ┌─────────────────┐
│  Предсказание:  │     │  Предсказание:  │
│  "кот" 99%      │     │  "собака" 95%   │
└─────────────────┘     └─────────────────┘
```

---

## 🎨 Глава 6.6: Создание портфолио

### 📁 Структура проекта

```
ml-portfolio/
├── README.md                  # Главная страница
├── projects/
│   ├── image-classification/
│   │   ├── README.md
│   │   ├── notebooks/
│   │   ├── src/
│   │   └── models/
│   ├── nlp-sentiment/
│   └── recommendation-system/
├── blog/                      # Статьи и объяснения
├── assets/                    # Изображения, GIF
└── docs/                      # Документация
```

### 📝 Шаблон README для проекта

```markdown
# 🎯 Название проекта

## 📊 Описание задачи
Краткое описание бизнес-задачи и данных

## 🔧 Технологии
- Python, PyTorch, FastAPI
- Docker, Kubernetes
- MLflow, DVC

## 📈 Результаты
- Accuracy: 95.2%
- F1-score: 0.94
- Latency: 15ms

## 🚀 Как запустить
```bash
docker run -p 8000:8000 project-name
```

## 📊 Демо
![Demo GIF](demo.gif)
```

### 🎯 Ключевые проекты для портфолио

```
Обязательные проекты
├── 1. Классификация изображений
│   └── CNN, Transfer Learning, Деплой
├── 2. NLP проект
│   └── Transformers, API, Мониторинг
├── 3. Временные ряды
│   └── LSTM, Прогнозирование, Визуализация
├── 4. Recommendation System
│   └── Collaborative Filtering, A/B тест
├── 5. End-to-end ML pipeline
│   └── MLOps, CI/CD, Kubernetes
└── 6. Исследовательский проект
    └── Новая архитектура, Эксперименты
```

---

## 🏆 Итоговый результат блока

### ✅ Технические навыки
- **MLOps:** Версионирование, CI/CD, мониторинг
- **Деплой:** Docker, Kubernetes, облачные платформы
- **Оптимизация:** Pruning, quantization, ONNX
- **Безопасность:** Fairness, adversarial attacks

### 🎯 Практические достижения
- Полноценное приложение в продакшене
- Система мониторинга качества модели
- Портфолио из 6+ проектов на GitHub
- Готовность к техническим собеседованиям

### 📚 Дополнительные материалы
- [MLOps Guide](https://ml-ops.org/)
- [Model Deployment Patterns](https://martinfowler.com/articles/ml-architecture-patterns.html)
- [Responsible AI Practices](https://ai.google/responsibilities/responsible-ai-practices/)

---

## 🎓 Следующие шаги
1. **Создать GitHub Portfolio** - Демонстрация проектов
2. **Подготовиться к собеседованиям** - Практика алгоритмов
3. **Изучить специализацию** - Computer Vision, NLP, MLOps
4. **Участвовать в соревнованиях** - Kaggle, DrivenData
5. **Читать актуальные статьи** - Papers With Code, ArXiv

**Поздравляем! Вы готовы к карьере ML Engineer! 🚀**