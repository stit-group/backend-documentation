# Блок 2: Основы нейронных сетей и их реализация

---

**⏱ Длительность:** 5-7 недель  
**🎯 Цель:** Понять архитектуру и принципы работы базовых нейронных сетей  
**📊 Уровень сложности:** ⭐⭐⭐⭐☆

---

## 🎯 Что вы изучите в этом блоке

```
Входные знания          Процесс обучения          Выходные навыки
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│ • Математика    │────▶│ • Теория        │────▶│ • Реализация    │
│ • Python        │     │ • Практика      │     │ • Понимание     │
│ • Основы ML     │     │ • Проекты       │     │ • Опыт          │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

---

## 📚 Глава 2.1: Персептрон и многослойные сети

### 🏛️ История и основы

Нейронные сети начались с простой идеи - создать математическую модель нейрона:

```
Биологический нейрон          Искусственный нейрон
┌───────────────────┐        ┌───────────────────┐
│    Дендриты       │   →    │     Входы (x)     │
│       ↓           │        │       ↓           │
│    Тело клетки    │   →    │  Функция (Σwx+b)  │
│       ↓           │        │       ↓           │
│     Аксон         │   →    │ Активация f(z)    │
└───────────────────┘        └───────────────────┘
```

### 🔢 Математическая модель персептрона

**Персептрон** - это простейшая нейронная сеть, состоящая из одного нейрона:

```
Входы → Веса → Суммирование → Активация → Выход

x₁ ─────┐
         │ w₁
x₂ ─────┤     ┌─────┐    ┌─────────┐    ┌─────┐
         │ w₂  │  Σ  │────│  f(z)   │────│  y  │
x₃ ─────┤     └─────┘    └─────────┘    └─────┘
         │ w₃      ↑
xₙ ─────┘           b (bias)
```

**Математическая формула:**
```
z = w₁x₁ + w₂x₂ + ... + wₙxₙ + b = Σwᵢxᵢ + b
y = f(z)
```

### 🧮 Функции активации

```
Ступенчатая функция      Сигмоида              ReLU
┌─────────────────┐     ┌─────────────────┐    ┌─────────────────┐
│      1 │        │     │        ╭─────    │    │      ╱          │
│        │        │     │      ╱           │    │    ╱            │
│────────┼────────│     │    ╱             │    │  ╱              │
│      0 │        │     │  ╱               │    │╱________________│
└─────────────────┘     └─────────────────┘    └─────────────────┘
f(z) = {1 если z≥0     f(z) = 1/(1+e^(-z))    f(z) = max(0,z)
        {0 если z<0
```

### 🏗️ Многослойный персептрон (MLP)

```
Входной слой    Скрытый слой    Выходной слой
                                              
x₁ ───┐         ┌─── h₁ ───┐         ┌─── y₁
      │         │          │         │
x₂ ───┼─────────┼─── h₂ ───┼─────────┼─── y₂
      │         │          │         │
x₃ ───┼─────────┼─── h₃ ───┼─────────┼─── y₃
      │         │          │         │
x₄ ───┘         └─── h₄ ───┘         └─── y₄

Полносвязная архитектура - каждый нейрон соединен с каждым нейроном следующего слоя
```

### 💻 Практическое задание 2.1: Реализация персептрона

```python
import numpy as np
import matplotlib.pyplot as plt

class Perceptron:
    def __init__(self, n_inputs, learning_rate=0.01):
        """
        Инициализация персептрона
        
        Args:
            n_inputs: количество входных признаков
            learning_rate: скорость обучения
        """
        self.weights = np.random.randn(n_inputs) * 0.01
        self.bias = 0
        self.learning_rate = learning_rate
        self.training_history = []
    
    def activation(self, z):
        """Ступенчатая функция активации"""
        return np.where(z >= 0, 1, 0)
    
    def forward(self, X):
        """Прямое распространение"""
        z = np.dot(X, self.weights) + self.bias
        return self.activation(z)
    
    def fit(self, X, y, epochs=100):
        """Обучение персептрона"""
        for epoch in range(epochs):
            errors = 0
            for i in range(len(X)):
                # Прямое распространение
                prediction = self.forward(X[i])
                
                # Вычисление ошибки
                error = y[i] - prediction
                
                # Обновление весов и смещения
                self.weights += self.learning_rate * error * X[i]
                self.bias += self.learning_rate * error
                
                errors += int(error != 0)
            
            self.training_history.append(errors)
            
            if errors == 0:
                print(f"Сходимость достигнута на эпохе {epoch + 1}")
                break
    
    def plot_training_history(self):
        """Визуализация процесса обучения"""
        plt.figure(figsize=(10, 6))
        plt.plot(self.training_history)
        plt.title('Количество ошибок в процессе обучения')
        plt.xlabel('Эпоха')
        plt.ylabel('Количество ошибок')
        plt.grid(True)
        plt.show()

# Пример использования для задачи XOR
# (Внимание: простой персептрон НЕ может решить XOR!)
```

### 🔍 Ключевые понятия главы 2.1

| Понятие | Описание |
|---------|----------|
| **Персептрон** | Простейшая нейронная сеть из одного нейрона |
| **Веса (weights)** | Параметры, определяющие силу связи между нейронами |
| **Смещение (bias)** | Параметр, позволяющий сдвигать функцию активации |
| **Линейная разделимость** | Возможность разделить классы прямой линией |
| **Универсальная теорема аппроксимации** | MLP может аппроксимировать любую непрерывную функцию |

---

## 🎭 Глава 2.2: Функции активации

### 🌟 Роль функций активации

```
Без активации           С активацией
┌─────────────────┐    ┌─────────────────┐
│  Линейная       │    │  Нелинейная     │
│  комбинация     │    │  комбинация     │
│     ↓           │    │     ↓           │
│  Линейная       │    │  Сложная        │
│  функция        │    │  функция        │
└─────────────────┘    └─────────────────┘
```

**Зачем нужны функции активации?**
- Вносят нелинейность в модель
- Позволяют решать сложные задачи
- Контролируют выходные значения нейронов

### 📊 Основные функции активации

#### 1. **Sigmoid (Сигмоида)**
```
Формула: f(x) = 1/(1 + e^(-x))
График:     1 ┌─────────────────
              │        ╭─────
              │      ╱
              │    ╱
              │  ╱
            0 └╱─────────────────
             -∞  -2  0  2  +∞

Плюсы:  • Гладкая функция
        • Значения в диапазоне (0,1)
        • Интерпретируемые как вероятности

Минусы: • Проблема затухающих градиентов
        • Не центрирована относительно нуля
        • Вычислительно затратная
```

#### 2. **Tanh (Гиперболический тангенс)**
```
Формула: f(x) = (e^x - e^(-x))/(e^x + e^(-x))
График:     1 ┌─────────────────
              │      ╭─────
              │    ╱
              │  ╱
            0 ├╱─────────────────
              │╱
              │  ╲
           -1 └─────────────────
             -∞  -2  0  2  +∞

Плюсы:  • Центрирована относительно нуля
        • Более сильные градиенты чем sigmoid
        • Значения в диапазоне (-1,1)

Минусы: • Все еще проблема затухающих градиентов
        • Вычислительно затратная
```

#### 3. **ReLU (Rectified Linear Unit)**
```
Формула: f(x) = max(0, x)
График:       ┌─────────────────
              │      ╱
              │    ╱
              │  ╱
            0 ├╱─────────────────
              │
              │
              └─────────────────
             -∞  -2  0  2  +∞

Плюсы:  • Простая и быстрая
        • Не насыщается для положительных значений
        • Разреженность активаций

Минусы: • "Мертвые" нейроны (dying ReLU)
        • Не дифференцируема в нуле
```

#### 4. **Leaky ReLU**
```
Формула: f(x) = max(αx, x), где α ≈ 0.01
График:       ┌─────────────────
              │      ╱
              │    ╱
              │  ╱
            0 ├╱─────────────────
              │╱
              │
              └─────────────────
             -∞  -2  0  2  +∞

Плюсы:  • Решает проблему dying ReLU
        • Быстрые вычисления
        • Позволяет небольшие отрицательные значения

Минусы: • Дополнительный гиперпараметр α
```

### 🔬 Производные функций активации

```
Функция     │ Производная        │ Особенности
────────────┼───────────────────┼─────────────────────
Sigmoid     │ f'(x) = f(x)(1-f(x))│ Максимум 0.25
Tanh        │ f'(x) = 1 - f(x)²   │ Максимум 1
ReLU        │ f'(x) = {1 if x>0   │ Константа или 0
            │         {0 if x≤0   │
Leaky ReLU  │ f'(x) = {1 if x>0   │ Всегда > 0
            │         {α if x≤0   │
```

### 🧪 Практическое задание 2.2: Сравнение функций активации

```python
import numpy as np
import matplotlib.pyplot as plt

class ActivationFunctions:
    @staticmethod
    def sigmoid(x):
        """Сигмоида"""
        return 1 / (1 + np.exp(-np.clip(x, -500, 500)))
    
    @staticmethod
    def sigmoid_derivative(x):
        """Производная сигмоиды"""
        s = ActivationFunctions.sigmoid(x)
        return s * (1 - s)
    
    @staticmethod
    def tanh(x):
        """Гиперболический тангенс"""
        return np.tanh(x)
    
    @staticmethod
    def tanh_derivative(x):
        """Производная tanh"""
        return 1 - np.tanh(x)**2
    
    @staticmethod
    def relu(x):
        """ReLU"""
        return np.maximum(0, x)
    
    @staticmethod
    def relu_derivative(x):
        """Производная ReLU"""
        return (x > 0).astype(float)
    
    @staticmethod
    def leaky_relu(x, alpha=0.01):
        """Leaky ReLU"""
        return np.where(x > 0, x, alpha * x)
    
    @staticmethod
    def leaky_relu_derivative(x, alpha=0.01):
        """Производная Leaky ReLU"""
        return np.where(x > 0, 1, alpha)
    
    @staticmethod
    def plot_comparison():
        """Сравнение функций активации"""
        x = np.linspace(-5, 5, 1000)
        
        fig, ((ax1, ax2), (ax3, ax4)) = plt.subplots(2, 2, figsize=(15, 10))
        
        # Функции активации
        ax1.plot(x, ActivationFunctions.sigmoid(x), 'b-', label='Sigmoid')
        ax1.plot(x, ActivationFunctions.tanh(x), 'r-', label='Tanh')
        ax1.plot(x, ActivationFunctions.relu(x), 'g-', label='ReLU')
        ax1.plot(x, ActivationFunctions.leaky_relu(x), 'm-', label='Leaky ReLU')
        ax1.set_title('Функции активации')
        ax1.legend()
        ax1.grid(True)
        
        # Производные
        ax2.plot(x, ActivationFunctions.sigmoid_derivative(x), 'b-', label='Sigmoid')
        ax2.plot(x, ActivationFunctions.tanh_derivative(x), 'r-', label='Tanh')
        ax2.plot(x, ActivationFunctions.relu_derivative(x), 'g-', label='ReLU')
        ax2.plot(x, ActivationFunctions.leaky_relu_derivative(x), 'm-', label='Leaky ReLU')
        ax2.set_title('Производные функций активации')
        ax2.legend()
        ax2.grid(True)
        
        # Проблема затухающих градиентов
        layers = np.arange(1, 21)
        sigmoid_grad = 0.25 ** layers  # Максимальный градиент сигмоиды
        tanh_grad = 1.0 ** layers      # Максимальный градиент tanh
        relu_grad = np.ones_like(layers)  # ReLU сохраняет градиент
        
        ax3.semilogy(layers, sigmoid_grad, 'b-', label='Sigmoid')
        ax3.semilogy(layers, tanh_grad, 'r-', label='Tanh')
        ax3.semilogy(layers, relu_grad, 'g-', label='ReLU')
        ax3.set_title('Затухание градиентов по слоям')
        ax3.set_xlabel('Номер слоя')
        ax3.set_ylabel('Градиент (log scale)')
        ax3.legend()
        ax3.grid(True)
        
        plt.tight_layout()
        plt.show()

# Запуск сравнения
ActivationFunctions.plot_comparison()
```

### 🎯 Выбор функции активации

```
Задача                  │ Рекомендуемая функция
────────────────────────┼─────────────────────
Скрытые слои (общий)    │ ReLU / Leaky ReLU
Выходной слой (бинарная)│ Sigmoid
Выходной слой (мульти)  │ Softmax
Регрессия              │ Линейная / ReLU
Автоэнкодеры           │ Sigmoid / Tanh
```

---

## 🔄 Глава 2.3: Алгоритм обратного распространения ошибки

### 🧠 Суть алгоритма

**Обратное распространение (Backpropagation)** - это алгоритм для обучения многослойных нейронных сетей.

```
Прямое распространение (Forward Pass)
┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐
│Input│───▶│ L1  │───▶│ L2  │───▶│Output│
└─────┘    └─────┘    └─────┘    └─────┘
                                    │
                                    ▼
                               ┌─────────┐
                               │  Loss   │
                               └─────────┘
                                    │
                                    ▼
Обратное распространение (Backward Pass)
┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐
│∂L/∂W│◀───│∂L/∂W│◀───│∂L/∂W│◀───│∂L/∂y│
└─────┘    └─────┘    └─────┘    └─────┘
```

### 📚 Математические основы

#### 1. **Цепное правило дифференцирования**

Если у нас есть композиция функций: `z = f(g(x))`, то:
```
dz/dx = (dz/dg) × (dg/dx)
```

#### 2. **Применение к нейронным сетям**

Для сети с функцией потерь `L` и весами `W`:
```
∂L/∂W = (∂L/∂y) × (∂y/∂z) × (∂z/∂W)

где:
- ∂L/∂y - градиент функции потерь по выходу
- ∂y/∂z - производная функции активации
- ∂z/∂W - градиент по весам
```

### 🔢 Пошаговый алгоритм

```
Шаг 1: Прямое распространение
──────────────────────────────
Входной слой: a⁽⁰⁾ = x
Для каждого слоя l = 1, 2, ..., L:
  z⁽ˡ⁾ = W⁽ˡ⁾a⁽ˡ⁻¹⁾ + b⁽ˡ⁾
  a⁽ˡ⁾ = f(z⁽ˡ⁾)

Шаг 2: Вычисление ошибки
────────────────────────
δ⁽ᴸ⁾ = ∇ₐL ⊙ f'(z⁽ᴸ⁾)

Шаг 3: Обратное распространение
────────────────────────────────
Для каждого слоя l = L-1, L-2, ..., 1:
  δ⁽ˡ⁾ = ((W⁽ˡ⁺¹⁾)ᵀδ⁽ˡ⁺¹⁾) ⊙ f'(z⁽ˡ⁾)

Шаг 4: Вычисление градиентов
────────────────────────────
∂L/∂W⁽ˡ⁾ = δ⁽ˡ⁾(a⁽ˡ⁻¹⁾)ᵀ
∂L/∂b⁽ˡ⁾ = δ⁽ˡ⁾

Шаг 5: Обновление параметров
────────────────────────────
W⁽ˡ⁾ = W⁽ˡ⁾ - η∂L/∂W⁽ˡ⁾
b⁽ˡ⁾ = b⁽ˡ⁾ - η∂L/∂b⁽ˡ⁾
```

### 📊 Вычислительный граф

```
Пример простой сети:
x ─── W₁ ─── z₁ ─── σ ─── a₁ ─── W₂ ─── z₂ ─── σ ─── a₂ ─── L
      │              │              │              │
      │              │              │              │
Прямой │              │              │              │ Обратный
проход │              │              │              │ проход
      │              │              │              │
      ▼              ▼              ▼              ▼
    ∂L/∂W₁ ◀─── ∂L/∂z₁ ◀─── ∂L/∂a₁ ◀─── ∂L/∂z₂ ◀─── ∂L/∂a₂
```

### 💻 Практическое задание 2.3: Реализация backpropagation

```python
import numpy as np
import matplotlib.pyplot as plt

class NeuralNetwork:
    def __init__(self, layers, learning_rate=0.01):
        """
        Создание нейронной сети
        
        Args:
            layers: список с количеством нейронов в каждом слое
            learning_rate: скорость обучения
        """
        self.layers = layers
        self.learning_rate = learning_rate
        self.weights = []
        self.biases = []
        
        # Инициализация весов и смещений
        for i in range(len(layers) - 1):
            w = np.random.randn(layers[i], layers[i+1]) * np.sqrt(2.0 / layers[i])
            b = np.zeros((1, layers[i+1]))
            self.weights.append(w)
            self.biases.append(b)
    
    def sigmoid(self, x):
        """Сигмоида"""
        return 1 / (1 + np.exp(-np.clip(x, -500, 500)))
    
    def sigmoid_derivative(self, x):
        """Производная сигмоиды"""
        s = self.sigmoid(x)
        return s * (1 - s)
    
    def forward(self, X):
        """
        Прямое распространение
        
        Args:
            X: входные данные
            
        Returns:
            activations: активации каждого слоя
            z_values: линейные комбинации каждого слоя
        """
        activations = [X]
        z_values = []
        
        for i in range(len(self.weights)):
            z = np.dot(activations[-1], self.weights[i]) + self.biases[i]
            z_values.append(z)
            a = self.sigmoid(z)
            activations.append(a)
        
        return activations, z_values
    
    def backward(self, X, y, activations, z_values):
        """
        Обратное распространение
        
        Args:
            X: входные данные
            y: целевые значения
            activations: активации каждого слоя
            z_values: линейные комбинации каждого слоя
        """
        m = X.shape[0]
        
        # Вычисление ошибки выходного слоя
        delta = activations[-1] - y
        
        # Обратное распространение ошибки
        deltas = [delta]
        
        for i in range(len(self.weights) - 1, 0, -1):
            delta = np.dot(delta, self.weights[i].T) * self.sigmoid_derivative(z_values[i-1])
            deltas.append(delta)
        
        deltas.reverse()
        
        # Обновление весов и смещений
        for i in range(len(self.weights)):
            self.weights[i] -= self.learning_rate * np.dot(activations[i].T, deltas[i]) / m
            self.biases[i] -= self.learning_rate * np.sum(deltas[i], axis=0, keepdims=True) / m
    
    def train(self, X, y, epochs=1000):
        """
        Обучение сети
        
        Args:
            X: входные данные
            y: целевые значения
            epochs: количество эпох
        """
        losses = []
        
        for epoch in range(epochs):
            # Прямое распространение
            activations, z_values = self.forward(X)
            
            # Вычисление функции потерь
            loss = np.mean((activations[-1] - y) ** 2)
            losses.append(loss)
            
            # Обратное распространение
            self.backward(X, y, activations, z_values)
            
            if epoch % 100 == 0:
                print(f'Epoch {epoch}, Loss: {loss:.4f}')
        
        return losses
    
    def predict(self, X):
        """Предсказание"""
        activations, _ = self.forward(X)
        return activations[-1]
    
    def plot_training_curve(self, losses):
        """Визуализация процесса обучения"""
        plt.figure(figsize=(10, 6))
        plt.plot(losses)
        plt.title('Кривая обучения')
        plt.xlabel('Эпоха')
        plt.ylabel('Функция потерь')
        plt.yscale('log')
        plt.grid(True)
        plt.show()

# Тестирование на данных XOR
X_xor = np.array([[0, 0], [0, 1], [1, 0], [1, 1]])
y_xor = np.array([[0], [1], [1], [0]])

# Создание и обучение сети
nn = NeuralNetwork([2, 4, 1], learning_rate=1.0)
losses = nn.train(X_xor, y_xor, epochs=5000)

# Проверка результатов
predictions = nn.predict(X_xor)
print("\nРезультаты:")
for i in range(len(X_xor)):
    print(f"Input: {X_xor[i]}, Target: {y_xor[i][0]}, Prediction: {predictions[i][0]:.4f}")

# Визуализация процесса обучения
nn.plot_training_curve(losses)
```

### 🎯 Ключевые моменты главы 2.3

| Концепция | Важность | Описание |
|-----------|----------|----------|
| **Цепное правило** | ⭐⭐⭐⭐⭐ | Основа для вычисления градиентов |
| **Вычислительный граф** | ⭐⭐⭐⭐☆ | Представление операций для автоматического дифференцирования |
| **Затухающие градиенты** | ⭐⭐⭐⭐☆ | Проблема глубоких сетей |
| **Инициализация весов** | ⭐⭐⭐⭐☆ | Критично для успешного обучения |

---

## 📉 Глава 2.4: Функции потерь и метрики качества

### 🎯 Роль функций потерь

```
Обучение нейронной сети = Минимизация функции потерь

Предсказание → Функция потерь → Градиенты → Обновление весов
     ↑                                              ↓
     └──────────────── Итерация ──────────────────┘
```

### 📊 Основные функции потерь

#### 1. **Средняя квадратичная ошибка (MSE)**

```
Формула: MSE = 1/n Σ(yᵢ - ŷᵢ)²

Применение: Регрессия
Особенности: • Чувствительна к выбросам
            • Гладкая функция
            • Квадратично штрафует большие ошибки

График функции потерь:
     ↑ Loss
     │    ╱│╲
     │  ╱  │  ╲
     │ ╱   │   ╲
     │╱    │    ╲
─────┼─────┼─────┼────→ (y - ŷ)
     │     0
```

#### 2. **Средняя абсолютная ошибка (MAE)**

```
Формула: MAE = 1/n Σ|yᵢ - ŷᵢ|

Применение: Регрессия (устойчивая к выбросам)
Особенности: • Менее чувствительна к выбросам
            • Недифференцируема в нуле
            • Линейно штрафует ошибки

График функции потерь:
     ↑ Loss
     │     ╱│╲
     │   ╱  │  ╲
     │ ╱    │    ╲
─────┼─────┼─────┼────→ (y - ŷ)
     │     0
```

#### 3. **Кросс-энтропия (Cross-Entropy)**

```
Бинарная: L = -[y log(ŷ) + (1-y) log(1-ŷ)]

Многоклассовая: L = -Σ yᵢ log(ŷᵢ)

Применение: Классификация
Особенности: • Сильно штрафует уверенные неправильные предсказания
            • Работает с вероятностями
            • Выпуклая функция

График (бинарный случай):
     ↑ Loss
     │ ╱
     │╱
     │╲
     │ ╲____
─────┼─────┼────→ ŷ
     0     1
```

### 📈 Метрики качества

#### Классификация

```
Матрица ошибок (Confusion Matrix):
                 Предсказание
                 │  0  │  1  │
Истинные    ─────┼─────┼─────┤
значения     0   │ TN  │ FP  │
             1   │ FN  │ TP  │
```

**Основные метрики:**
- **Accuracy** = (TP + TN) / (TP + TN + FP + FN)
- **Precision** = TP / (TP + FP)
- **Recall** = TP / (TP + FN)
- **F1-Score** = 2 × (Precision × Recall) / (Precision + Recall)

#### Регрессия

```
Метрика              │ Формула                    │ Интерпретация
─────────────────────┼────────────────────────────┼─────────────────
MSE                  │ 1/n Σ(y - ŷ)²              │ Средняя кв. ошибка
RMSE                 │ √(1/n Σ(y - ŷ)²)           │ В единицах целевой переменной
MAE                  │ 1/n Σ|y - ŷ|               │ Средняя абс. ошибка
R²                   │ 1 - SS_res/SS_tot          │ Доля объясненной дисперсии
```

### 💻 Практическое задание 2.4: Реализация функций потерь

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.metrics import confusion_matrix, classification_report
import seaborn as sns

class LossFunctions:
    @staticmethod
    def mse(y_true, y_pred):
        """Средняя квадратичная ошибка"""
        return np.mean((y_true - y_pred) ** 2)
    
    @staticmethod
    def mse_derivative(y_true, y_pred):
        """Производная MSE"""
        return 2 * (y_pred - y_true) / len(y_true)
    
    @staticmethod
    def mae(y_true, y_pred):
        """Средняя абсолютная ошибка"""
        return np.mean(np.abs(y_true - y_pred))
    
    @staticmethod
    def mae_derivative(y_true, y_pred):
        """Производная MAE"""
        return np.sign(y_pred - y_true) / len(y_true)
    
    @staticmethod
    def binary_cross_entropy(y_true, y_pred):
        """Бинарная кросс-энтропия"""
        epsilon = 1e-15  # для численной стабильности
        y_pred = np.clip(y_pred, epsilon, 1 - epsilon)
        return -np.mean(y_true * np.log(y_pred) + (1 - y_true) * np.log(1 - y_pred))
    
    @staticmethod
    def binary_cross_entropy_derivative(y_true, y_pred):
        """Производная бинарной кросс-энтропии"""
        epsilon = 1e-15
        y_pred = np.clip(y_pred, epsilon, 1 - epsilon)
        return (y_pred - y_true) / (y_pred * (1 - y_pred) * len(y_true))
    
    @staticmethod
    def categorical_cross_entropy(y_true, y_pred):
        """Категориальная кросс-энтропия"""
        epsilon = 1e-15
        y_pred = np.clip(y_pred, epsilon, 1 - epsilon)
        return -np.mean(np.sum(y_true * np.log(y_pred), axis=1))
    
    @staticmethod
    def visualize_loss_functions():
        """Визуализация функций потерь"""
        # Данные для визуализации
        y_true = 1.0  # истинное значение
        y_pred = np.linspace(0.01, 0.99, 100)
        
        # Вычисление потерь
        bce_loss = []
        mse_loss = []
        mae_loss = []
        
        for pred in y_pred:
            bce_loss.append(LossFunctions.binary_cross_entropy(
                np.array([y_true]), np.array([pred])
            ))
            mse_loss.append(LossFunctions.mse(
                np.array([y_true]), np.array([pred])
            ))
            mae_loss.append(LossFunctions.mae(
                np.array([y_true]), np.array([pred])
            ))
        
        plt.figure(figsize=(15, 5))
        
        # Binary Cross-Entropy
        plt.subplot(1, 3, 1)
        plt.plot(y_pred, bce_loss, 'b-', linewidth=2)
        plt.axvline(x=y_true, color='r', linestyle='--', label='True value')
        plt.title('Binary Cross-Entropy Loss')
        plt.xlabel('Predicted Value')
        plt.ylabel('Loss')
        plt.legend()
        plt.grid(True)
        
        # MSE
        plt.subplot(1, 3, 2)
        plt.plot(y_pred, mse_loss, 'g-', linewidth=2)
        plt.axvline(x=y_true, color='r', linestyle='--', label='True value')
        plt.title('Mean Squared Error Loss')
        plt.xlabel('Predicted Value')
        plt.ylabel('Loss')
        plt.legend()
        plt.grid(True)
        
        # MAE
        plt.subplot(1, 3, 3)
        plt.plot(y_pred, mae_loss, 'm-', linewidth=2)
        plt.axvline(x=y_true, color='r', linestyle='--', label='True value')
        plt.title('Mean Absolute Error Loss')
        plt.xlabel('Predicted Value')
        plt.ylabel('Loss')
        plt.legend()
        plt.grid(True)
        
        plt.tight_layout()
        plt.show()

class Metrics:
    @staticmethod
    def accuracy(y_true, y_pred):
        """Точность"""
        return np.mean(y_true == y_pred)
    
    @staticmethod
    def precision(y_true, y_pred, average='binary'):
        """Точность (precision)"""
        if average == 'binary':
            tp = np.sum((y_true == 1) & (y_pred == 1))
            fp = np.sum((y_true == 0) & (y_pred == 1))
            return tp / (tp + fp) if (tp + fp) > 0 else 0
        else:
            # Многоклассовая точность
            classes = np.unique(y_true)
            precisions = []
            for cls in classes:
                tp = np.sum((y_true == cls) & (y_pred == cls))
                fp = np.sum((y_true != cls) & (y_pred == cls))
                prec = tp / (tp + fp) if (tp + fp) > 0 else 0
                precisions.append(prec)
            return np.mean(precisions)
    
    @staticmethod
    def recall(y_true, y_pred, average='binary'):
        """Полнота (recall)"""
        if average == 'binary':
            tp = np.sum((y_true == 1) & (y_pred == 1))
            fn = np.sum((y_true == 1) & (y_pred == 0))
            return tp / (tp + fn) if (tp + fn) > 0 else 0
        else:
            # Многоклассовая полнота
            classes = np.unique(y_true)
            recalls = []
            for cls in classes:
                tp = np.sum((y_true == cls) & (y_pred == cls))
                fn = np.sum((y_true == cls) & (y_pred != cls))
                rec = tp / (tp + fn) if (tp + fn) > 0 else 0
                recalls.append(rec)
            return np.mean(recalls)
    
    @staticmethod
    def f1_score(y_true, y_pred, average='binary'):
        """F1-мера"""
        prec = Metrics.precision(y_true, y_pred, average)
        rec = Metrics.recall(y_true, y_pred, average)
        return 2 * prec * rec / (prec + rec) if (prec + rec) > 0 else 0
    
    @staticmethod
    def confusion_matrix_plot(y_true, y_pred, classes=None):
        """Визуализация матрицы ошибок"""
        cm = confusion_matrix(y_true, y_pred)
        
        plt.figure(figsize=(8, 6))
        sns.heatmap(cm, annot=True, fmt='d', cmap='Blues',
                   xticklabels=classes, yticklabels=classes)
        plt.title('Confusion Matrix')
        plt.xlabel('Predicted')
        plt.ylabel('True')
        plt.show()
        
        return cm

# Демонстрация
LossFunctions.visualize_loss_functions()

# Пример использования метрик
np.random.seed(42)
y_true = np.random.randint(0, 2, 100)
y_pred = np.random.randint(0, 2, 100)

print("Метрики классификации:")
print(f"Accuracy: {Metrics.accuracy(y_true, y_pred):.3f}")
print(f"Precision: {Metrics.precision(y_true, y_pred):.3f}")
print(f"Recall: {Metrics.recall(y_true, y_pred):.3f}")
print(f"F1-Score: {Metrics.f1_score(y_true, y_pred):.3f}")
```

### 🎯 Выбор функции потерь

```
Задача                    │ Функция потерь           │ Метрики
──────────────────────────┼─────────────────────────┼────────────────
Бинарная классификация    │ Binary Cross-Entropy    │ Accuracy, F1, AUC
Многоклассовая классифик. │ Categorical Cross-Entropy│ Accuracy, F1-macro
Регрессия                 │ MSE / MAE               │ RMSE, MAE, R²
Несбалансированные данные │ Focal Loss              │ F1, Precision, Recall
```

---

## 🔧 Глава 2.5: Регуляризация и нормализация

### 🎯 Проблема переобучения

```
Недообучение    Оптимальная модель    Переобучение
┌─────────────┐  ┌─────────────────┐  ┌─────────────┐
│  Простая    │  │   Сбалансир.    │  │   Сложная   │
│  модель     │  │    модель       │  │   модель    │
│             │  │                 │  │             │
│ Высокий     │  │ Низкий bias     │  │ Низкий bias │
│ bias        │  │ Низкий variance │  │ Высокий     │
│ Низкий      │  │                 │  │ variance    │
│ variance    │  │                 │  │             │
└─────────────┘  └─────────────────┘  └─────────────┘
```

### 🛡️ Методы регуляризации

#### 1. **L1 и L2 регуляризация**

```
L1 регуляризация (Lasso):
Loss = MSE + λ Σ|wᵢ|

L2 регуляризация (Ridge):
Loss = MSE + λ Σwᵢ²

Elastic Net:
Loss = MSE + λ₁ Σ|wᵢ| + λ₂ Σwᵢ²

Эффект на веса:
L1: │    ╱│╲         L2: │      ╱│╲
    │  ╱  │  ╲           │    ╱  │  ╲
    │ ╱   │   ╲          │  ╱    │    ╲
────┼────┼────┼───      ────┼────┼────┼───
    │    0               │    0
    
L1 → разреженные веса   L2 → малые веса
```

#### 2. **Dropout**

```
Обучение                    Инференс
┌─────────────────────┐    ┌─────────────────────┐
│  o    o    o    o   │    │  o    o    o    o   │
│  │    │    │    │   │    │  │    │    │    │   │
│  o    ×    o    ×   │    │  o    o    o    o   │
│  │    ×    │    ×   │    │  │    │    │    │   │
│  o    ×    o    o   │    │  o    o    o    o   │
└─────────────────────┘    └─────────────────────┘
50% нейронов отключены     Все нейроны активны

Математически:
r ~ Bernoulli(p)
y = r * x / p  (масштабирование)
```

#### 3. **Batch Normalization**

```
Проблема: Internal Covariate Shift
┌─────────────────────────────────────────────────────┐
│ Слой 1    Слой 2    Слой 3    Слой 4    Слой 5    │
│   │        │         │         │         │        │
│   ▼        ▼         ▼         ▼         ▼        │
│ Норм.   Сдвиг      Сдвиг     Сдвиг     Сдвиг     │
│распред. распред.  распред.  распред.  распред.    │
└─────────────────────────────────────────────────────┘

Решение Batch Normalization:
μ = 1/m Σxᵢ                    # среднее по батчу
σ² = 1/m Σ(xᵢ - μ)²            # дисперсия по батчу
x̂ᵢ = (xᵢ - μ) / √(σ² + ε)      # нормализация
yᵢ = γx̂ᵢ + β                   # масштабирование и сдвиг
```

### 📊 Сравнение методов нормализации

```
Batch Norm     │ Layer Norm     │ Instance Norm  │ Group Norm
───────────────┼────────────────┼────────────────┼────────────────
Норм. по батчу │ Норм. по слою  │ Норм. по канал │ Норм. по группе
               │                │                │
B×H×W×C        │ B×H×W×C        │ B×H×W×C        │ B×H×W×C
   ↓           │    ↓           │    ↓           │    ↓
Среднее по     │ Среднее по     │ Среднее по     │ Среднее по
H×W×C          │ H×W×C          │ H×W            │ подгруппе каналов
```

### 💻 Практическое задание 2.5: Реализация регуляризации

```python
import numpy as np
import matplotlib.pyplot as plt

class RegularizedNeuralNetwork:
    def __init__(self, layers, learning_rate=0.01, dropout_rate=0.5, 
                 l1_reg=0.0, l2_reg=0.01, use_batch_norm=True):
        """
        Нейронная сеть с регуляризацией
        
        Args:
            layers: архитектура сети
            learning_rate: скорость обучения
            dropout_rate: вероятность отключения нейрона
            l1_reg: коэффициент L1 регуляризации
            l2_reg: коэффициент L2 регуляризации
            use_batch_norm: использовать batch normalization
        """
        self.layers = layers
        self.learning_rate = learning_rate
        self.dropout_rate = dropout_rate
        self.l1_reg = l1_reg
        self.l2_reg = l2_reg
        self.use_batch_norm = use_batch_norm
        
        # Инициализация параметров
        self.weights = []
        self.biases = []
        self.gamma = []  # для batch norm
        self.beta = []   # для batch norm
        
        for i in range(len(layers) - 1):
            # Xavier/He инициализация
            if i == 0:
                w = np.random.randn(layers[i], layers[i+1]) * np.sqrt(2.0 / layers[i])
            else:
                w = np.random.randn(layers[i], layers[i+1]) * np.sqrt(2.0 / layers[i])
            
            b = np.zeros((1, layers[i+1]))
            
            self.weights.append(w)
            self.biases.append(b)
            
            # Параметры batch normalization
            if self.use_batch_norm and i < len(layers) - 2:  # не для выходного слоя
                self.gamma.append(np.ones((1, layers[i+1])))
                self.beta.append(np.zeros((1, layers[i+1])))
    
    def relu(self, x):
        """ReLU активация"""
        return np.maximum(0, x)
    
    def relu_derivative(self, x):
        """Производная ReLU"""
        return (x > 0).astype(float)
    
    def softmax(self, x):
        """Softmax активация"""
        # Для численной стабильности вычитаем максимум
        exp_x = np.exp(x - np.max(x, axis=1, keepdims=True))
        return exp_x / np.sum(exp_x, axis=1, keepdims=True)
    
    def dropout(self, x, rate, training=True):
        """Dropout регуляризация"""
        if not training:
            return x
        
        keep_prob = 1 - rate
        mask = np.random.binomial(1, keep_prob, size=x.shape) / keep_prob
        return x * mask
    
    def forward(self, X, training=True):
        """Прямое распространение"""
        activations = [X]
        z_values = []
        dropout_masks = []
        
        for i in range(len(self.weights)):
            # Линейная комбинация
            z = np.dot(activations[-1], self.weights[i]) + self.biases[i]
            z_values.append(z)
            
            # Активация
            if i < len(self.weights) - 1:  # скрытые слои
                a = self.relu(z)
                # Dropout для скрытых слоев
                if training:
                    a = self.dropout(a, self.dropout_rate, training)
            else:  # выходной слой
                a = self.softmax(z)
            
            activations.append(a)
        
        return activations, z_values
    
    def compute_loss(self, y_true, y_pred):
        """Вычисление функции потерь"""
        # Categorical Cross-Entropy
        epsilon = 1e-15
        y_pred = np.clip(y_pred, epsilon, 1 - epsilon)
        ce_loss = -np.mean(np.sum(y_true * np.log(y_pred), axis=1))
        
        # L2 регуляризация
        l2_loss = 0
        for w in self.weights:
            l2_loss += np.sum(w ** 2)
        
        total_loss = ce_loss + self.l2_reg * l2_loss
        return total_loss, ce_loss
    
    def backward(self, X, y, activations, z_values):
        """Обратное распространение"""
        m = X.shape[0]
        
        # Градиенты
        dW = [np.zeros_like(w) for w in self.weights]
        db = [np.zeros_like(b) for b in self.biases]
        
        # Ошибка выходного слоя
        delta = activations[-1] - y
        
        # Обратное распространение
        for i in range(len(self.weights) - 1, -1, -1):
            # Градиенты для весов и смещений
            dW[i] = np.dot(activations[i].T, delta) / m + self.l2_reg * self.weights[i]
            db[i] = np.sum(delta, axis=0, keepdims=True) / m
            
            # Распространение ошибки на предыдущий слой
            if i > 0:
                delta = np.dot(delta, self.weights[i].T) * self.relu_derivative(z_values[i-1])
        
        return dW, db
    
    def adam_update(self, dW, db):
        """Обновление параметров с помощью Adam optimizer"""
        self.t += 1
        
        for i in range(len(self.weights)):
            # Обновление моментов для весов
            self.m_w[i] = self.beta1 * self.m_w[i] + (1 - self.beta1) * dW[i]
            self.v_w[i] = self.beta2 * self.v_w[i] + (1 - self.beta2) * (dW[i] ** 2)
            
            # Коррекция смещения
            m_w_corrected = self.m_w[i] / (1 - self.beta1 ** self.t)
            v_w_corrected = self.v_w[i] / (1 - self.beta2 ** self.t)
            
            # Обновление весов
            self.weights[i] -= self.learning_rate * m_w_corrected / (np.sqrt(v_w_corrected) + self.epsilon)
            
            # Обновление моментов для смещений
            self.m_b[i] = self.beta1 * self.m_b[i] + (1 - self.beta1) * db[i]
            self.v_b[i] = self.beta2 * self.v_b[i] + (1 - self.beta2) * (db[i] ** 2)
            
            # Коррекция смещения
            m_b_corrected = self.m_b[i] / (1 - self.beta1 ** self.t)
            v_b_corrected = self.v_b[i] / (1 - self.beta2 ** self.t)
            
            # Обновление смещений
            self.biases[i] -= self.learning_rate * m_b_corrected / (np.sqrt(v_b_corrected) + self.epsilon)
    
    def accuracy(self, y_true, y_pred):
        """Вычисление точности"""
        predictions = np.argmax(y_pred, axis=1)
        true_labels = np.argmax(y_true, axis=1)
        return np.mean(predictions == true_labels)
    
    def train(self, X_train, y_train, X_val, y_val, epochs=100, batch_size=32):
        """Обучение нейронной сети"""
        n_batches = len(X_train) // batch_size
        
        print("Начинаем обучение...")
        print(f"Количество эпох: {epochs}")
        print(f"Размер батча: {batch_size}")
        print(f"Количество батчей за эпоху: {n_batches}")
        print("-" * 50)
        
        for epoch in range(epochs):
            epoch_loss = 0
            epoch_accuracy = 0
            
            # Перемешивание данных
            indices = np.random.permutation(len(X_train))
            X_train_shuffled = X_train[indices]
            y_train_shuffled = y_train[indices]
            
            # Обучение по батчам
            for batch in range(n_batches):
                start_idx = batch * batch_size
                end_idx = start_idx + batch_size
                
                X_batch = X_train_shuffled[start_idx:end_idx]
                y_batch = y_train_shuffled[start_idx:end_idx]
                
                # Прямое распространение
                activations, z_values = self.forward(X_batch, training=True)
                
                # Вычисление потерь
                loss, ce_loss = self.compute_loss(y_batch, activations[-1])
                epoch_loss += loss
                
                # Точность
                acc = self.accuracy(y_batch, activations[-1])
                epoch_accuracy += acc
                
                # Обратное распространение
                dW, db = self.backward(X_batch, y_batch, activations, z_values)
                
                # Обновление параметров
                self.adam_update(dW, db)
            
            # Средние значения за эпоху
            avg_loss = epoch_loss / n_batches
            avg_accuracy = epoch_accuracy / n_batches
            
            # Валидация
            val_activations, _ = self.forward(X_val, training=False)
            val_loss, _ = self.compute_loss(y_val, val_activations[-1])
            val_accuracy = self.accuracy(y_val, val_activations[-1])
            
            # Сохранение истории
            self.train_losses.append(avg_loss)
            self.train_accuracies.append(avg_accuracy)
            self.val_losses.append(val_loss)
            self.val_accuracies.append(val_accuracy)
            
            # Вывод прогресса
            if epoch % 10 == 0 or epoch == epochs - 1:
                print(f"Эпоха {epoch:3d}: "
                      f"Train Loss: {avg_loss:.4f}, Train Acc: {avg_accuracy:.3f}, "
                      f"Val Loss: {val_loss:.4f}, Val Acc: {val_accuracy:.3f}")
        
        print("-" * 50)
        print("Обучение завершено!")
        print(f"Финальная точность на валидации: {self.val_accuracies[-1]:.3f}")
    
    def predict(self, X):
        """Предсказание"""
        activations, _ = self.forward(X, training=False)
        return activations[-1]
    
    def plot_training_history(self):
        """Визуализация процесса обучения"""
        fig, ((ax1, ax2), (ax3, ax4)) = plt.subplots(2, 2, figsize=(15, 10))
        
        # Функция потерь
        ax1.plot(self.train_losses, 'b-', label='Train Loss')
        ax1.plot(self.val_losses, 'r-', label='Validation Loss')
        ax1.set_title('Функция потерь')
        ax1.set_xlabel('Эпоха')
        ax1.set_ylabel('Потеря')
        ax1.legend()
        ax1.grid(True)
        
        # Точность
        ax2.plot(self.train_accuracies, 'b-', label='Train Accuracy')
        ax2.plot(self.val_accuracies, 'r-', label='Validation Accuracy')
        ax2.set_title('Точность')
        ax2.set_xlabel('Эпоха')
        ax2.set_ylabel('Точность')
        ax2.legend()
        ax2.grid(True)
        
        # Функция потерь (логарифмическая шкала)
        ax3.semilogy(self.train_losses, 'b-', label='Train Loss')
        ax3.semilogy(self.val_losses, 'r-', label='Validation Loss')
        ax3.set_title('Функция потерь (log scale)')
        ax3.set_xlabel('Эпоха')
        ax3.set_ylabel('Потеря (log)')
        ax3.legend()
        ax3.grid(True)
        
        # Разность между train и validation accuracy
        overfitting = np.array(self.train_accuracies) - np.array(self.val_accuracies)
        ax4.plot(overfitting, 'g-', label='Overfitting Gap')
        ax4.axhline(y=0, color='k', linestyle='--', alpha=0.5)
        ax4.set_title('Переобучение (Train Acc - Val Acc)')
        ax4.set_xlabel('Эпоха')
        ax4.set_ylabel('Разность точности')
        ax4.legend()
        ax4.grid(True)
        
        plt.tight_layout()
        plt.show()
    
    def plot_confusion_matrix(self, X_test, y_test):
        """Матрица ошибок"""
        predictions = self.predict(X_test)
        pred_labels = np.argmax(predictions, axis=1)
        true_labels = np.argmax(y_test, axis=1)
        
        # Вычисление матрицы ошибок
        cm = np.zeros((10, 10))
        for i in range(len(pred_labels)):
            cm[true_labels[i], pred_labels[i]] += 1
        
        # Нормализация по строкам
        cm_normalized = cm / cm.sum(axis=1, keepdims=True)
        
        # Визуализация
        plt.figure(figsize=(10, 8))
        sns.heatmap(cm_normalized, annot=True, fmt='.2f', cmap='Blues',
                   xticklabels=range(10), yticklabels=range(10))
        plt.title('Матрица ошибок (нормализованная)')
        plt.xlabel('Предсказанный класс')
        plt.ylabel('Истинный класс')
        plt.show()
        
        return cm
    
    def visualize_weights(self):
        """Визуализация весов первого слоя"""
        # Веса первого слоя можно интерпретировать как изображения
        weights = self.weights[0].T  # Транспонируем для правильной формы
        
        # Визуализация первых 16 нейронов
        fig, axes = plt.subplots(4, 4, figsize=(10, 10))
        for i in range(16):
            ax = axes[i // 4, i % 4]
            weight_img = weights[i].reshape(28, 28)
            ax.imshow(weight_img, cmap='RdBu', vmin=-np.max(np.abs(weight_img)), 
                     vmax=np.max(np.abs(weight_img)))
            ax.set_title(f'Нейрон {i+1}')
            ax.axis('off')
        
        plt.suptitle('Веса первого слоя (детекторы признаков)')
        plt.tight_layout()
        plt.show()
    
    def test_predictions(self, X_test, y_test, n_samples=10):
        """Тестирование предсказаний на случайных примерах"""
        indices = np.random.choice(len(X_test), n_samples, replace=False)
        
        fig, axes = plt.subplots(2, 5, figsize=(15, 6))
        
        for i, idx in enumerate(indices):
            ax = axes[i // 5, i % 5]
            
            # Показываем изображение
            image = X_test[idx].reshape(28, 28)
            ax.imshow(image, cmap='gray')
            
            # Предсказание
            prediction = self.predict(X_test[idx:idx+1])
            pred_class = np.argmax(prediction)
            pred_prob = prediction[0, pred_class]
            
            # Истинный класс
            true_class = np.argmax(y_test[idx])
            
            # Цвет заголовка: зеленый если правильно, красный если неправильно
            color = 'green' if pred_class == true_class else 'red'
            ax.set_title(f'True: {true_class}, Pred: {pred_class} ({pred_prob:.2f})', 
                        color=color)
            ax.axis('off')
        
        plt.suptitle('Примеры предсказаний')
        plt.tight_layout()
        plt.show()

# Функция для загрузки и подготовки данных MNIST
def load_mnist_data():
    """Загрузка и подготовка данных MNIST"""
    print("Загрузка данных MNIST...")
    
    # Загрузка данных
    mnist = fetch_openml('mnist_784', version=1, as_frame=False)
    X, y = mnist.data, mnist.target.astype(int)
    
    # Нормализация данных
    X = X / 255.0
    
    # One-hot кодирование для меток
    y_onehot = np.zeros((len(y), 10))
    for i, label in enumerate(y):
        y_onehot[i, label] = 1
    
    # Разделение на train/test
    X_train, X_test, y_train, y_test = train_test_split(
        X, y_onehot, test_size=0.2, random_state=42, stratify=y
    )
    
    # Разделение train на train/validation
    X_train, X_val, y_train, y_val = train_test_split(
        X_train, y_train, test_size=0.2, random_state=42, stratify=np.argmax(y_train, axis=1)
    )
    
    print(f"Размер обучающей выборки: {X_train.shape}")
    print(f"Размер валидационной выборки: {X_val.shape}")
    print(f"Размер тестовой выборки: {X_test.shape}")
    
    return X_train, X_val, X_test, y_train, y_val, y_test

# Демонстрация работы
def demonstrate_mnist_training():
    """Демонстрация обучения на MNIST"""
    # Загрузка данных
    X_train, X_val, X_test, y_train, y_val, y_test = load_mnist_data()
    
    # Создание и обучение модели
    model = MNISTNeuralNetwork()
    model.train(X_train, y_train, X_val, y_val, epochs=50, batch_size=128)
    
    # Оценка на тестовых данных
    test_predictions = model.predict(X_test)
    test_accuracy = model.accuracy(y_test, test_predictions)
    print(f"\nТочность на тестовых данных: {test_accuracy:.3f}")
    
    # Визуализация результатов
    model.plot_training_history()
    model.plot_confusion_matrix(X_test, y_test)
    model.visualize_weights()
    model.test_predictions(X_test, y_test)
    
    return model

# Запуск демонстрации
# model = demonstrate_mnist_training()
```

---

## 🎯 Итоговые результаты Блока 2

### ✅ Что вы достигли

```
📊 Теоретические знания:
├── Понимание архитектуры нейронных сетей
├── Знание функций активации и их свойств
├── Глубокое понимание backpropagation
├── Знание функций потерь и метрик
└── Методы регуляризации и нормализации

💻 Практические навыки:
├── Реализация нейронной сети с нуля
├── Обучение модели на реальных данных
├── Достижение высокой точности (95%+)
├── Визуализация процесса обучения
└── Анализ результатов

🛠️ Инструменты:
├── NumPy для вычислений
├── Matplotlib для визуализации
├── Gradient descent оптимизация
├── Adam optimizer
└── Регуляризация L1/L2 + Dropout
```

### 🏆 Финальная оценка

| Критерий | Оценка | Комментарий |
|----------|--------|-------------|
| **Теория** | ⭐⭐⭐⭐⭐ | Полное понимание основ |
| **Практика** | ⭐⭐⭐⭐⭐ | Рабочая реализация с нуля |
| **Результат** | ⭐⭐⭐⭐⭐ | Точность 95%+ на MNIST |
| **Код** | ⭐⭐⭐⭐☆ | Чистый, документированный |

### 🚀 Готовность к следующему блоку

```
✅ Понимание основ нейронных сетей
✅ Навыки реализации с нуля
✅ Опыт обучения на реальных данных
✅ Знание методов регуляризации
✅ Умение анализировать результаты

➡️ Готов к изучению CNN (Блок 3)
```

---

## 📚 Дополнительные материалы

### 🔗 Рекомендуемые ресурсы

**Книги:**
- Neural Networks and Deep Learning (Michael Nielsen)
- Deep Learning (Ian Goodfellow, Yoshua Bengio, Aaron Courville)
- Pattern Recognition and Machine Learning (Christopher Bishop)

**Онлайн курсы:**
- CS231n: Convolutional Neural Networks for Visual Recognition
- Andrew Ng's Deep Learning Specialization
- Fast.ai Practical Deep Learning

**Инструменты:**
- TensorFlow/Keras
- PyTorch
- JAX
- Weights & Biases для tracking

### 🎯 Практические задания для закрепления

1. **Реализация других архитектур:**
   - Автоэнкодер для сжатия данных
   - Многоклассовый классификатор для CIFAR-10
   - Регрессионная сеть для Boston Housing

2. **Эксперименты с гиперпараметрами:**
   - Сравнение разных оптимизаторов
   - Влияние размера батча на обучение
   - Optimal learning rate scheduling

3. **Продвинутые техники:**
   - Batch Normalization реализация
   - Различные инициализации весов
   - Early stopping и learning rate decay

---

## 🎓 Заключение

Поздравляем! Вы успешно завершили **Блок 2: Основы нейронных сетей и их реализация**. 

Теперь у вас есть:
- **Глубокое понимание** того, как работают нейронные сети
- **Практические навыки** создания и обучения моделей
- **Опыт работы** с реальными данными
- **Готовность** к изучению более сложных архитектур

В следующем блоке мы перейдем к **сверточным нейронным сетям (CNN)** и их применению в компьютерном зрении. Вы узнаете, как обрабатывать изображения и создавать системы распознавания объектов.

```
🎯 Следующий блок: CNN и компьютерное зрение
📅 Продолжительность: 6-8 недель
🚀 Уровень сложности: ⭐⭐⭐⭐☆
```

**Удачи в дальнейшем изучении глубокого обучения!** 🚀
    
    def sigmoid(self, x):
        """Сигмоида"""
        return 1 / (1 + np.exp(-np.clip(x, -500, 500)))
    
    def sigmoid_derivative(self, x):
        """Производная сигмоиды"""
        s = self.sigmoid(x)
        return s * (1 - s)
    
    def batch_norm(self, x, gamma, beta, epsilon=1e-8):
        """Batch normalization"""
        mu = np.mean(x, axis=0, keepdims=True)
        sigma2 = np.var(x, axis=0, keepdims=True)
        x_norm = (x - mu) / np.sqrt(sigma2 + epsilon)
        return gamma * x_norm + beta, mu, sigma2
    
    def dropout(self, x, rate, training=True):
        """Dropout регуляризация"""
        if not training:
            return x
        
        keep_prob = 1 - rate
        mask = np.random.binomial(1, keep_prob, size=x.shape)
        return x * mask / keep_prob
    
    def forward(self, X, training=True):
        """Прямое распространение"""
        activations = [X]
        z_values = []
        dropout_masks = []
        batch_norm_params = []
        
        for i in range(len(self.weights)):
            # Линейная комбинация
            z = np.dot(activations[-1], self.weights[i]) + self.biases[i]
            z_values.append(z)
            
            # Batch normalization (кроме выходного слоя)
            if self.use_batch_norm and i < len(self.weights) - 1:
                z_norm, mu, sigma2 = self.batch_norm(z, self.gamma[i], self.beta[i])
                batch_norm_params.append((mu, sigma2))
                z = z_norm
            
            # Активация
            if i < len(self.weights) - 1:  # скрытые слои
                a = self.relu(z)
            else:  # выходной слой
                a = self.sigmoid(z)
            
            # Dropout (кроме выходного слоя)
            if training and i < len(self.weights) - 1:
                a = self.dropout(a, self.dropout_rate, training)
                dropout_masks.append(np.random.binomial(1, 1-self.dropout_rate, size=a.shape))
            
            activations.append(a)
        
        return activations, z_values, dropout_masks, batch_norm_params
    
    def compute_loss(self, y_true, y_pred):
        """Вычисление функции потерь с регуляризацией"""
        # Основная потеря
        epsilon = 1e-15
        y_pred = np.clip(y_pred, epsilon, 1 - epsilon)
        ce_loss = -np.mean(y_true * np.log(y_pred) + (1 - y_true) * np.log(1 - y_pred))
        
        # L1 регуляризация
        l1_loss = 0
        if self.l1_reg > 0:
            for w in self.weights:
                l1_loss += np.sum(np.abs(w))
        
        # L2 регуляризация
        l2_loss = 0
        if self.l2_reg > 0:
            for w in self.weights:
                l2_loss += np.sum(w ** 2)
        
        total_loss = ce_loss + self.l1_reg * l1_loss + self.l2_reg * l2_loss
        return total_loss, ce_loss, l1_loss, l2_loss
    
    def train(self, X, y, epochs=1000, validation_data=None):
        """Обучение с регуляризацией"""
        train_losses = []
        val_losses = []
        
        for epoch in range(epochs):
            # Прямое распространение
            activations, z_values, dropout_masks, batch_norm_params = self.forward(X, training=True)
            
            # Вычисление потерь
            total_loss, ce_loss, l1_loss, l2_loss = self.compute_loss(y, activations[-1])
            train_losses.append(total_loss)
            
            # Обратное распространение (упрощенная версия)
            # ... (реализация обратного распространения с учетом регуляризации)
            
            # Валидация
            if validation_data is not None:
                X_val, y_val = validation_data
                val_activations, _, _, _ = self.forward(X_val, training=False)
                val_loss, _, _, _ = self.compute_loss(y_val, val_activations[-1])
                val_losses.append(val_loss)
            
            if epoch % 100 == 0:
                print(f'Epoch {epoch}: Total Loss = {total_loss:.4f}, '
                      f'CE Loss = {ce_loss:.4f}, L1 = {l1_loss:.4f}, L2 = {l2_loss:.4f}')
        
        return train_losses, val_losses
    
    def plot_regularization_comparison(self):
        """Сравнение эффекта регуляризации"""
        # Создание синтетических данных
        np.random.seed(42)
        X = np.random.randn(100, 2)
        y = (X[:, 0] + X[:, 1] > 0).astype(int).reshape(-1, 1)
        
        # Модели с разной регуляризацией
        models = {
            'Без регуляризации': {'l1_reg': 0, 'l2_reg': 0, 'dropout_rate': 0},
            'L1 регуляризация': {'l1_reg': 0.01, 'l2_reg': 0, 'dropout_rate': 0},
            'L2 регуляризация': {'l1_reg': 0, 'l2_reg': 0.01, 'dropout_rate': 0},
            'Dropout': {'l1_reg': 0, 'l2_reg': 0, 'dropout_rate': 0.5}
        }
        
        plt.figure(figsize=(15, 10))
        
        for i, (name, params) in enumerate(models.items(), 1):
            plt.subplot(2, 2, i)
            
            # Создание модели
            model = RegularizedNeuralNetwork(
                [2, 10, 1], 
                learning_rate=0.01,
                **params
            )
            
            # Обучение
            losses, _ = model.train(X, y, epochs=500)
            
            # Визуализация
            plt.plot(losses)
            plt.title(f'{name}')
            plt.xlabel('Эпоха')
            plt.ylabel('Потеря')
            plt.grid(True)
        
        plt.tight_layout()
        plt.show()

# Демонстрация
model = RegularizedNeuralNetwork([2, 10, 5, 1])
model.plot_regularization_comparison()
```

### 🎯 Рекомендации по использованию

```
Метод              │ Когда использовать              │ Гиперпараметры
───────────────────┼─────────────────────────────────┼─────────────────
L1 регуляризация   │ Нужна разреженность весов      │ λ = 0.001-0.1
L2 регуляризация   │ Универсальная регуляризация    │ λ = 0.0001-0.01
Dropout           │ Большие сети, переобучение     │ p = 0.2-0.5
Batch Norm        │ Глубокие сети, нестабильность  │ ε = 1e-8, momentum = 0.9
```

---

## 🏆 Глава 2.6: Финальный проект блока

### 🎯 Задача: Создание нейронной сети с нуля

**Цель:** Реализовать полносвязную нейронную сеть на чистом NumPy и обучить её на датасете MNIST.

**Требования:**
- Архитектура: 784 → 128 → 64 → 10
- Функции активации: ReLU для скрытых слоев, Softmax для выходного
- Функция потерь: Categorical Cross-Entropy
- Регуляризация: L2 + Dropout
- Оптимизатор: Adam
- Целевая точность: 95%+

### 💻 Полная реализация

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import fetch_openml
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
import seaborn as sns

class MNISTNeuralNetwork:
    def __init__(self):
        """Инициализация нейронной сети для MNIST"""
        self.layers = [784, 128, 64, 10]
        self.learning_rate = 0.001
        self.dropout_rate = 0.3
        self.l2_reg = 0.0001
        
        # Инициализация весов (He initialization)
        self.weights = []
        self.biases = []
        
        for i in range(len(self.layers) - 1):
            w = np.random.randn(self.layers[i], self.layers[i+1]) * np.sqrt(2.0 / self.layers[i])
            b = np.zeros((1, self.layers[i+1]))
            self.weights.append(w)
            self.biases.append(b)
        
        # Параметры для Adam optimizer
        self.m_w = [np.zeros_like(w) for w in self.weights]
        self.v_w = [np.zeros_like(w) for w in self.weights]
        self.m_b = [np.zeros_like(b) for b in self.biases]
        self.v_b = [np.zeros_like(b) for b in self.biases]
        self.beta1 = 0.9
        self.beta2 = 0.999
        self.epsilon = 1e-8
        self.t = 0  # временной шаг
        
        # История обучения
        self.train_losses = []
        self.train_accuracies = []
        self.val_losses = []
        self.val_accuracies = []
    
    def relu(self, x):
        """ReLU активация"""
        return np.maximum(0, x)
    
    def relu_derivative(self, x):
        """Производная ReLU"""
        return (x > 0).astype(float)