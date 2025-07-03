# Блок 3: Сверточные нейронные сети и компьютерное зрение

**⏱ Длительность:** 6-8 недель  
**🎯 Цель:** Освоить CNN и их применение для задач компьютерного зрения

---

## 🔍 Обзор блока

Сверточные нейронные сети (CNN) произвели революцию в области компьютерного зрения. В этом блоке мы изучим:
- Математические принципы свертки
- Классические и современные архитектуры CNN
- Transfer Learning и Fine-tuning
- Практические задачи компьютерного зрения
- Работу с реальными датасетами изображений

---

## 📖 Глава 3.1: Принципы свертки
**📅 Неделя 1**

### Теоретическая часть

#### Что такое свертка?

Свертка - это математическая операция, которая объединяет две функции для получения третьей функции. В контексте нейронных сетей, свертка применяется к изображениям для извлечения признаков.

```
Математическая формула свертки:
(f * g)(t) = ∫ f(τ)g(t-τ)dτ
```

Для дискретных данных (изображений):
```
(f * g)[m,n] = ∑∑ f[i,j] × g[m-i,n-j]
```

#### Визуализация операции свертки

```
Исходное изображение (5x5):        Фильтр (3x3):
┌─────────────────────┐            ┌─────────────┐
│ 1  0  1  0  1 │            │ 1  0 -1 │
│ 0  1  0  1  0 │    *       │ 1  0 -1 │
│ 1  0  1  0  1 │            │ 1  0 -1 │
│ 0  1  0  1  0 │            └─────────────┘
│ 1  0  1  0  1 │
└─────────────────────┘

Результат свертки (3x3):
┌─────────────┐
│ -1  0  1 │
│  1  0 -1 │
│ -1  0  1 │
└─────────────┘
```

#### Ключевые параметры свертки

**1. Padding (Заполнение)**
```
Без padding:                   С padding = 1:
┌─────────────────────┐        ┌─────────────────────────┐
│ 1  0  1  0  1 │        │ 0  0  0  0  0  0  0 │
│ 0  1  0  1  0 │        │ 0  1  0  1  0  1  0 │
│ 1  0  1  0  1 │   →    │ 0  0  1  0  1  0  0 │
│ 0  1  0  1  0 │        │ 0  1  0  1  0  1  0 │
│ 1  0  1  0  1 │        │ 0  0  1  0  1  0  0 │
└─────────────────────┘        │ 0  1  0  1  0  1  0 │
                               │ 0  0  0  0  0  0  0 │
                               └─────────────────────────┘
5x5 → 3x3 (уменьшение)         7x7 → 5x5 (сохранение размера)
```

**2. Stride (Шаг)**
```
Stride = 1:                    Stride = 2:
┌─────────────────────┐        ┌─────────────────────┐
│ [1  0  1] 0  1 │        │ [1  0  1] 0  1 │
│ [0  1  0] 1  0 │        │  0  1  0  1  0 │
│ [1  0  1] 0  1 │        │ [1  0  1] 0  1 │
│  0  1  0  1  0 │        │  0  1  0  1  0 │
│  1  0  1  0  1 │        │  1  0  1  0  1 │
└─────────────────────┘        └─────────────────────┘
     ↓                              ↓
    3x3                            2x2
```

#### Карты признаков (Feature Maps)

```
Входное изображение (RGB):
┌─────────────────────┐
│ Red Channel   │
│ ┌─────────────────┐   │
│ │ R R R R R │   │
│ │ R R R R R │   │
│ │ R R R R R │   │
│ └─────────────────┘   │
│                       │
│ Green Channel   │
│ ┌─────────────────┐   │
│ │ G G G G G │   │
│ │ G G G G G │   │
│ │ G G G G G │   │
│ └─────────────────┘   │
│                       │
│ Blue Channel    │
│ ┌─────────────────┐   │
│ │ B B B B B │   │
│ │ B B B B B │   │
│ │ B B B B B │   │
│ └─────────────────┘   │
└─────────────────────┘

                ↓ Свертка с 32 фильтрами

32 карты признаков:
┌─────────────────────────────────────────────────────┐
│ Map1  Map2  Map3  ...  Map30  Map31  Map32 │
│ ┌───┐ ┌───┐ ┌───┐      ┌───┐  ┌───┐  ┌───┐ │
│ │ F │ │ F │ │ F │      │ F │  │ F │  │ F │ │
│ │ F │ │ F │ │ F │ ...  │ F │  │ F │  │ F │ │
│ │ F │ │ F │ │ F │      │ F │  │ F │  │ F │ │
│ └───┘ └───┘ └───┘      └───┘  └───┘  └───┘ │
└─────────────────────────────────────────────────────┘
```

#### Операции пулинга (Pooling)

**Max Pooling:**
```
Исходная карта признаков (4x4):    Max Pooling 2x2:
┌─────────────────────────┐        ┌─────────────┐
│ 1  3  2  4 │        │ 3  4 │
│ 5  6  7  8 │   →    │ 6  8 │
│ 1  2  3  4 │        └─────────────┘
│ 5  6  7  8 │
└─────────────────────────┘
```

**Average Pooling:**
```
Исходная карта признаков (4x4):    Average Pooling 2x2:
┌─────────────────────────┐        ┌─────────────┐
│ 1  3  2  4 │        │3.75 5.25│
│ 5  6  7  8 │   →    │3.75 5.25│
│ 1  2  3  4 │        └─────────────┘
│ 5  6  7  8 │
└─────────────────────────┘
```

### Практическая часть

#### Задание 1: Реализация свертки с нуля

```python
import numpy as np
import matplotlib.pyplot as plt

def convolution2d(image, kernel, padding=0, stride=1):
    """
    Реализация 2D свертки
    """
    # Добавляем padding
    if padding > 0:
        image = np.pad(image, ((padding, padding), (padding, padding)), mode='constant')
    
    # Размеры
    image_h, image_w = image.shape
    kernel_h, kernel_w = kernel.shape
    
    # Размер выходного изображения
    output_h = (image_h - kernel_h) // stride + 1
    output_w = (image_w - kernel_w) // stride + 1
    
    # Инициализация выходного массива
    output = np.zeros((output_h, output_w))
    
    # Выполняем свертку
    for i in range(0, output_h):
        for j in range(0, output_w):
            y_start = i * stride
            y_end = y_start + kernel_h
            x_start = j * stride
            x_end = x_start + kernel_w
            
            # Поэлементное перемножение и суммирование
            output[i, j] = np.sum(image[y_start:y_end, x_start:x_end] * kernel)
    
    return output

# Пример использования
if __name__ == "__main__":
    # Создаем тестовое изображение
    image = np.array([
        [1, 0, 1, 0, 1],
        [0, 1, 0, 1, 0],
        [1, 0, 1, 0, 1],
        [0, 1, 0, 1, 0],
        [1, 0, 1, 0, 1]
    ])
    
    # Фильтр для детекции вертикальных границ
    edge_filter = np.array([
        [1, 0, -1],
        [1, 0, -1],
        [1, 0, -1]
    ])
    
    # Применяем свертку
    result = convolution2d(image, edge_filter)
    
    print("Исходное изображение:")
    print(image)
    print("\nФильтр:")
    print(edge_filter)
    print("\nРезультат свертки:")
    print(result)
```

#### Задание 2: Визуализация фильтров

```python
def create_filters():
    """
    Создание различных фильтров для детекции признаков
    """
    filters = {}
    
    # Фильтр для детекции вертикальных границ
    filters['vertical_edge'] = np.array([
        [1, 0, -1],
        [1, 0, -1],
        [1, 0, -1]
    ])
    
    # Фильтр для детекции горизонтальных границ
    filters['horizontal_edge'] = np.array([
        [1, 1, 1],
        [0, 0, 0],
        [-1, -1, -1]
    ])
    
    # Фильтр размытия
    filters['blur'] = np.array([
        [1, 1, 1],
        [1, 1, 1],
        [1, 1, 1]
    ]) / 9
    
    # Фильтр повышения резкости
    filters['sharpen'] = np.array([
        [0, -1, 0],
        [-1, 5, -1],
        [0, -1, 0]
    ])
    
    return filters

# Демонстрация работы фильтров
filters = create_filters()
for name, filter_kernel in filters.items():
    result = convolution2d(image, filter_kernel)
    print(f"\n{name}:")
    print(result)
```

### Домашнее задание

1. **Теоретическое задание:** Вычислить размер выходного изображения для:
   - Вход: 224×224, фильтр: 5×5, padding=2, stride=1
   - Вход: 128×128, фильтр: 3×3, padding=0, stride=2

2. **Практическое задание:** Реализовать функцию max_pooling2d с нуля

3. **Исследовательское задание:** Создать 5 различных фильтров и применить их к изображению

---

## 📖 Глава 3.2: Классические архитектуры CNN
**📅 Недели 2-3**

### Эволюция архитектур CNN

```
Временная линия развития CNN:
1998      2012      2014      2015      2016
 |         |         |         |         |
LeNet-5   AlexNet   VGGNet   ResNet   DenseNet
 |         |         |         |         |
 └─────────┼─────────┼─────────┼─────────┘
          Прорыв   Глубина  Остаточные
          в ImageNet         связи
```

### LeNet-5 (1998) - Пионер CNN

```
Архитектура LeNet-5:
Input (32x32x1) → Conv1 (5x5) → Pool1 (2x2) → Conv2 (5x5) → Pool2 (2x2) → FC1 (120) → FC2 (84) → Output (10)
     ↓              ↓           ↓           ↓           ↓         ↓         ↓         ↓
   32×32×1      → 28×28×6   → 14×14×6   → 10×10×16  → 5×5×16  → 400×1   → 120×1   → 84×1   → 10×1

Детальная структура:
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    LeNet-5 Architecture                                             │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Input Image    │ Conv Layer 1   │ SubSample 1    │ Conv Layer 2   │ SubSample 2    │ Fully Connected │
│ 32×32×1        │ 28×28×6        │ 14×14×6        │ 10×10×16       │ 5×5×16         │ 120 → 84 → 10  │
│ ┌─────────────┐ │ ┌─────────────┐ │ ┌─────────────┐ │ ┌─────────────┐ │ ┌─────────────┐ │ ┌─────────────┐ │
│ │ Handwritten │ │ │ 6 Feature   │ │ │ 6 Feature   │ │ │ 16 Feature  │ │ │ 16 Feature  │ │ │ Classification│ │
│ │ Digit       │ │ │ Maps        │ │ │ Maps        │ │ │ Maps        │ │ │ Maps        │ │ │ Vector      │ │
│ │ (grayscale) │ │ │ (5×5 conv)  │ │ │ (2×2 pool)  │ │ │ (5×5 conv)  │ │ │ (2×2 pool)  │ │ │             │ │
│ └─────────────┘ │ └─────────────┘ │ └─────────────┘ │ └─────────────┘ │ └─────────────┘ │ └─────────────┘ │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

**Ключевые особенности LeNet-5:**
- Первая успешная CNN для распознавания рукописных цифр
- Использует таnh активацию
- Subsampling вместо max pooling
- Всего ~60,000 параметров

### AlexNet (2012) - Прорыв в глубоком обучении

```
Архитектура AlexNet:
Input (227x227x3) → Conv1 → Pool1 → Conv2 → Pool2 → Conv3 → Conv4 → Conv5 → Pool3 → FC1 → FC2 → FC3
     ↓              ↓       ↓       ↓       ↓       ↓       ↓       ↓       ↓       ↓     ↓     ↓
  227×227×3    → 55×55×96 → 27×27×96 → 27×27×256 → 13×13×256 → 13×13×384 → 13×13×384 → 13×13×256 → 6×6×256 → 4096 → 4096 → 1000

Детальная структура:
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                   AlexNet Architecture                                              │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Layer    │ Type      │ Size        │ Filters │ Stride │ Padding │ Activation │ Params     │ Output    │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Input    │ Image     │ 227×227×3   │ -       │ -      │ -       │ -          │ 0          │ 227×227×3 │
│ Conv1    │ Conv      │ 11×11×96    │ 96      │ 4      │ 0       │ ReLU       │ 34,848     │ 55×55×96  │
│ Pool1    │ MaxPool   │ 3×3         │ -       │ 2      │ 0       │ -          │ 0          │ 27×27×96  │
│ Conv2    │ Conv      │ 5×5×256     │ 256     │ 1      │ 2       │ ReLU       │ 614,656    │ 27×27×256 │
│ Pool2    │ MaxPool   │ 3×3         │ -       │ 2      │ 0       │ -          │ 0          │ 13×13×256 │
│ Conv3    │ Conv      │ 3×3×384     │ 384     │ 1      │ 1       │ ReLU       │ 885,120    │ 13×13×384 │
│ Conv4    │ Conv      │ 3×3×384     │ 384     │ 1      │ 1       │ ReLU       │ 1,327,488  │ 13×13×384 │
│ Conv5    │ Conv      │ 3×3×256     │ 256     │ 1      │ 1       │ ReLU       │ 884,992    │ 13×13×256 │
│ Pool3    │ MaxPool   │ 3×3         │ -       │ 2      │ 0       │ -          │ 0          │ 6×6×256   │
│ FC1      │ Dense     │ 4096        │ -       │ -      │ -       │ ReLU       │ 37,752,832 │ 4096      │
│ FC2      │ Dense     │ 4096        │ -       │ -      │ -       │ ReLU       │ 16,781,312 │ 4096      │
│ FC3      │ Dense     │ 1000        │ -       │ -      │ -       │ Softmax    │ 4,097,000  │ 1000      │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
Total Parameters: 62,378,248
```

**Ключевые инновации AlexNet:**
- Использование ReLU вместо tanh
- Dropout для предотвращения переобучения
- Аугментация данных
- GPU вычисления
- Local Response Normalization

### VGGNet (2014) - Сила простоты

```
Архитектура VGG-16:
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    VGG-16 Architecture                                              │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Block 1:  │ Input (224×224×3) → Conv(3×3×64) → Conv(3×3×64) → MaxPool(2×2) → 112×112×64            │
│ Block 2:  │ Conv(3×3×128) → Conv(3×3×128) → MaxPool(2×2) → 56×56×128                                │
│ Block 3:  │ Conv(3×3×256) → Conv(3×3×256) → Conv(3×3×256) → MaxPool(2×2) → 28×28×256                │
│ Block 4:  │ Conv(3×3×512) → Conv(3×3×512) → Conv(3×3×512) → MaxPool(2×2) → 14×14×512                │
│ Block 5:  │ Conv(3×3×512) → Conv(3×3×512) → Conv(3×3×512) → MaxPool(2×2) → 7×7×512                  │
│ Classifier:│ FC(4096) → FC(4096) → FC(1000) → Softmax                                                 │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘

Визуализация размеров:
224×224×3 → 224×224×64 → 112×112×64 → 112×112×128 → 56×56×128 → 56×56×256 → 28×28×256 → 28×28×512 → 14×14×512 → 14×14×512 → 7×7×512 → 4096 → 4096 → 1000
    ↓            ↓            ↓            ↓           ↓           ↓           ↓           ↓           ↓           ↓         ↓      ↓      ↓
  Input      Conv+Conv     MaxPool     Conv+Conv    MaxPool   Conv+Conv+Conv MaxPool  Conv+Conv+Conv MaxPool Conv+Conv+Conv MaxPool  FC1    FC2    FC3
```

**Принципы VGG:**
- Только 3×3 фильтры
- Удвоение числа фильтров после каждого pooling
- Простая и единообразная архитектура
- Глубина 16-19 слоев

### ResNet (2015) - Революция остаточных связей

```
Проблема исчезающих градиентов:
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                            Глубокие сети без skip connections                                       │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Input → Layer1 → Layer2 → ... → Layer50 → ... → Layer152 → Output                                   │
│   ↓        ↓        ↓               ↓               ↓         ↓                                      │
│ Gradient backpropagation: ∇L/∂x₁₅₂ → ∇L/∂x₅₀ → ... → ∇L/∂x₂ → ∇L/∂x₁                           │
│                                                                                                     │
│ Градиент уменьшается экспоненциально: ∇L/∂x₁ ≈ ∇L/∂x₁₅₂ × ∏(∂xᵢ/∂xᵢ₋₁)                        │
│                                                        i=2                                          │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘

Residual Block (основной блок ResNet):
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                  Residual Block                                                     │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│           Input x                                                                                   │
│              │                                                                                      │
│              ├─────────────────────────────────────────────────────────────────────┐               │
│              │                                                                     │               │
│              ▼                                                                     │               │
│        Weight Layer 1                                                             │               │
│        Conv 3×3, ReLU                                                             │               │
│              │                                                                     │               │
│              ▼                                                                     │               │
│        Weight Layer 2                                                             │               │
│        Conv 3×3                                                                   │               │
│              │                                                                     │               │
│              ▼                                                                     │               │
│             [+] ←──────────────────────────────────────────────────────────────────┘               │
│              │                                                                                      │
│              ▼                                                                                      │
│            ReLU                                                                                     │
│              │                                                                                      │
│              ▼                                                                                      │
│         Output F(x) + x                                                                             │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘

Математически: H(x) = F(x) + x, где F(x) - остаточная функция
```

**Архитектура ResNet-50:**
```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                  ResNet-50 Architecture                                             │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Stage      │ Output Size │ Residual Block                                        │ Layers            │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Conv1      │ 112×112     │ 7×7 conv, 64, stride 2                              │ 1                 │
│ Conv2_x    │ 56×56       │ 3×3 maxpool, stride 2                               │ 1                 │
│            │             │ [1×1 conv, 64]                                       │                   │
│            │             │ [3×3 conv, 64] × 3                                   │ 9                 │
│            │             │ [1×1 conv, 256]                                      │                   │
│ Conv3_x    │ 28×28       │ [1×1 conv, 128]                                      │                   │
│            │             │ [3×3 conv, 128] × 4                                  │ 12                │
│            │             │ [1×1 conv, 512]                                      │                   │
│ Conv4_x    │ 14×14       │ [1×1 conv, 256]                                      │                   │
│            │             │ [3×3 conv, 256] × 6                                  │ 18                │
│            │             │ [1×1 conv, 1024]                                     │                   │
│ Conv5_x    │ 7×7         │ [1×1 conv, 512]                                      │                   │
│            │             │ [3×3 conv, 512] × 3                                  │ 9                 │
│            │             │ [1×1 conv, 2048]                                     │                   │
│ Classifier │ 1×1         │ average pool, 1000-d fc, softmax                     │ 2                 │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Total Layers: 50                                                                                    │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### Практическая реализация

#### Реализация ResNet Block на PyTorch

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class BasicBlock(nn.Module):
    """Базовый блок ResNet"""
    expansion = 1
    
    def __init__(self, in_channels, out_channels, stride=1, downsample=None):
        super(BasicBlock, self).__init__()
        
        # Первый слой свертки
        self.conv1 = nn.Conv2d(in_channels, out_channels, kernel_size=3, 
                              stride=stride, padding=1, bias=False)
        self.bn1 = nn.BatchNorm2d(out_channels)
        
        # Второй слой свертки
        self.conv2 = nn.Conv2d(out_channels, out_channels, kernel_size=3, 
                              stride=1, padding=1, bias=False)
        self.bn2 = nn.BatchNorm2d(out_channels)
        
        # Skip connection
        self.downsample = downsample
        
    def forward(self, x):
        identity = x
        
        # Основной путь
        out = self.conv1(x)
        out = self.bn1(out)
        out = F.relu(out)
        
        out = self.conv2(out)
        out = self.bn2(out)
        
        # Skip connection
        if self.downsample is not None:
            identity = self.downsample(x)
        
        out += identity  # Остаточное соединение
        out = F.relu(out)
        
        return out

class ResNet(nn.Module):
    """Упрощенная реализация ResNet"""
    
    def __init__(self, block, layers, num_classes=1000):
        super(ResNet, self).__init__()
        
        self.in_channels = 64
        
        # Начальный слой
        self.conv1 = nn.Conv2d(3, 64, kernel_size=7, stride=2, padding=3, bias=False)
        self.bn1 = nn.BatchNorm2d(64)
        self.maxpool = nn.MaxPool2d(kernel_size=3, stride=2, padding=1)
        
        # Residual layers
        self.layer1 = self._make_layer(block, 64, layers[0])
        self.layer2 = self._make_layer(block, 128, layers[1], stride=2)
        self.layer3 = self._make_layer(block, 256, layers[2], stride=2)
        self.layer4 = self._make_layer(block, 512, layers[3], stride=2)
        
        # Классификатор
        self.avgpool = nn.AdaptiveAvgPool2d((1, 1))
        self.fc = nn.Linear(512 * block.expansion, num_classes)
        
    def _make_layer(self, block, out_channels, blocks, stride=1):
        downsample = None
        if stride != 1 or self.in_channels != out_channels * block.expansion:
            downsample = nn.Sequential(
                nn.Conv2d(self.in_channels, out_channels * block.expansion,
                         kernel_size=1, stride=stride, bias=False),
                nn.BatchNorm2d(out_channels * block.expansion),
            )
        
        layers = []
        layers.append(block(self.in_channels, out_channels, stride, downsample))
        self.in_channels = out_channels * block.expansion
        for _ in range(1, blocks):
            layers.append(block(self.in_channels, out_channels))
        
        return nn.Sequential(*layers)
    
    def forward(self, x):
        # Начальные слои
        x = self.conv1(x)
        x = self.bn1(x)
        x = F.relu(x)
        x = self.maxpool(x)
        
        # Residual blocks
        x = self.layer1(x)
        x = self.layer2(x)
        x = self.layer3(x)
        x = self.layer4(x)
        
        # Классификатор
        x = self.avgpool(x)
        x = torch.flatten(x, 1)
        x = self.fc(x)
        
        return x

def resnet18(num_classes=1000):
    return ResNet(BasicBlock, [2, 2, 2, 2], num_classes)

def resnet34(num_classes=1000):
    return ResNet(BasicBlock, [3, 4, 6, 3], num_classes)

# Пример использования
if __name__ == "__main__":
    model = resnet18(num_classes=10)  # для CIFAR-10
    x = torch.randn(1, 3, 224, 224)
    output = model(x)
    print(f"Input shape: {x.shape}")
    print(f"Output shape: {output.shape}")
```

### Сравнение архитектур

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              Сравнение классических CNN архитектур                                 │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Модель    │ Год  │ Слои │ Параметры │ Top-1 Acc │ Особенности                                       │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ LeNet-5   │ 1998 │ 5    │ 60K       │ ~99% (MNIST)│ Первая CNN, простая архитектура                │
│ AlexNet   │ 2012 │ 8    │ 62M       │ 57.1%     │ ReLU, Dropout, GPU, прорыв в ImageNet            │
│ VGG-16    │ 2014 │ 16   │ 138M      │ 71.3%     │ Только 3×3 фильтры, простота                     │
│ VGG-19    │ 2014 │ 19   │ 144M      │ 71.1%     │ Еще глубже, но не лучше                          │
│ ResNet-50 │ 2015 │ 50   │ 26M       │ 76.0%     │ Skip connections, очень глубокие сети             │
│ ResNet-152│ 2015 │ 152  │ 60M       │ 77.8%     │ Самая глубокая на тот момент                      │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### Домашнее задание

1. **Реализация VGG-блока:** Создайте функцию для создания VGG-блока
2. **Анализ параметров:** Подсчитайте количество параметров в каждом слое ResNet-18
3. **Эксперимент:** Сравните время обучения ResNet-18 vs VGG-16 на CIFAR-10

---

## 📖 Глава 3.3: Transfer Learning и Fine-tuning
**📅 Неделя 4**

### Концепция Transfer Learning

Transfer Learning - это метод машинного обучения, при котором модель, обученная на одной задаче, адаптируется для решения связанной задачи.

```
Традиционный подход:
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ Задача A                                    │ Задача B                                             │
├─────────────────────────────────────────────┼─────────────────────────────────────────────────────┤
│ Данные A → Модель A → Результат A          │ Данные B → Модель B → Результат B                   │
│ (ImageNet)  (ResNet-50)  (1000 классов)   │ (Кошки/Собаки) (Новая сеть) (2 класса)             │
└─────────────────────────────────────────────┴─────────────────────────────────────────────────────┘

Transfer Learning:
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ Задача A                                    │ Задача B                                             │
├─────────────────────────────────────────────┼─────────────────────────────────────────────────────┤
│ Данные A → Модель A → Результат A          │ Предобученная модель A → Fine-tuning → Результат B    │
│ (ImageNet)  (ResNet-50)  (1000 классов)   │ (Заморозить веса) + (Новый классификатор) (2 класса) │
└─────────────────────────────────────────────┴─────────────────────────────────────────────────────┘
```

### Иерархия признаков в CNN

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                             Иерархия признаков в глубоких CNN                                      │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Слой        │ Что изучает                    │ Универсальность │ Специфичность                      │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Первые      │ Границы, углы, текстуры        │ Очень высокая   │ Общие для всех изображений         │
│ слои        │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓         │                 │                                    │
│             │                                 │                 │                                    │
│ Средние     │ Формы, паттерны, комбинации    │ Средняя         │ Зависит от домена                  │
│ слои        │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓         │                 │                                    │
│             │                                 │                 │                                    │
│ Последние   │ Сложные объекты, семантика     │ Низкая          │ Очень специфично для задачи       │
│ слои        │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓         │                 │                                    │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘

Визуализация извлеченных признаков:
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ Слой 1:  [/] [\\] [|] [-] [○] [△]  ← Простые границы и формы                                    │
│ Слой 2:  [◐] [◑] [◒] [◓] [⬢] [⬡]  ← Комбинации границ                                         │
│ Слой 3:  [👁] [🔲] [🔳] [⚪] [🔴] [🔵]  ← Более сложные паттерны                              │
│ Слой 4:  [🐱] [🐶] [🚗] [🏠] [🌳] [👤]  ← Объекты и части объектов                           │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### Стратегии Transfer Learning

#### 1. Feature Extraction (Извлечение признаков)

```python
import torch
import torch.nn as nn
import torchvision.models as models

# Загружаем предобученную модель
model = models.resnet50(pretrained=True)

# Замораживаем все параметры
for param in model.parameters():
    param.requires_grad = False

# Заменяем последний слой для новой задачи
num_classes = 2  # кошки vs собаки
model.fc = nn.Linear(model.fc.in_features, num_classes)

# Только параметры нового слоя будут обучаться
print("Параметры для обучения:")
for name, param in model.named_parameters():
    if param.requires_grad:
        print(f"  {name}: {param.shape}")
```

#### 2. Fine-tuning (Тонкая настройка)

```python
# Загружаем предобученную модель
model = models.resnet50(pretrained=True)

# Заменяем последний слой
num_classes = 2
model.fc = nn.Linear(model.fc.in_features, num_classes)

# Устанавливаем разные learning rates для разных частей модели
optimizer = torch.optim.Adam([
    {'params': model.conv1.parameters(), 'lr': 1e-5},      # Очень низкий lr
    {'params': model.layer1.parameters(), 'lr': 1e-5},     # Очень низкий lr
    {'params': model.layer2.parameters(), 'lr': 1e-4},     # Низкий lr
    {'params': model.layer3.parameters(), 'lr': 1e-4},     # Низкий lr
    {'params': model.layer4.parameters(), 'lr': 1e-3},     # Средний lr
    {'params': model.fc.parameters(), 'lr': 1e-2}          # Высокий lr
])
```

#### 3. Gradual Unfreezing (Постепенное размораживание)

```python
def unfreeze_layers(model, num_layers_to_unfreeze):
    """Размораживает последние num_layers_to_unfreeze слоев"""
    all_layers = list(model.children())
    
    # Замораживаем все слои
    for layer in all_layers:
        for param in layer.parameters():
            param.requires_grad = False
    
    # Размораживаем последние слои
    for layer in all_layers[-num_layers_to_unfreeze:]:
        for param in layer.parameters():
            param.requires_grad = True

# Пример использования
model = models.resnet50(pretrained=True)
model.fc = nn.Linear(model.fc.in_features, 2)

# Этап 1: Обучаем только классификатор
unfreeze_layers(model, 1)
# ... обучение 10 эпох ...

# Этап 2: Размораживаем последние 2 слоя
unfreeze_layers(model, 2)
# ... обучение еще 10 эпох с меньшим lr ...
```

### Выбор стратегии

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                      Матрица выбора стратегии Transfer Learning                                     │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                    │ Мало данных          │ Много данных                                            │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Данные похожи      │ Feature Extraction   │ Fine-tuning с низким lr                                │
│ на исходные        │ ┌─────────────────────┐ │ ┌─────────────────────────────────────────────────────┐ │
│                    │ │ • Заморозить веса  │ │ │ • Разморозить все слои                             │ │
│                    │ │ • Обучить только   │ │ │ • Использовать очень низкий learning rate          │ │
│                    │ │   классификатор    │ │ │ • Возможно gradual unfreezing                      │ │
│                    │ └─────────────────────┘ │ └─────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Данные сильно      │ Feature Extraction   │ Fine-tuning с высоким lr                               │
│ отличаются         │ + Fine-tuning        │ ┌─────────────────────────────────────────────────────┐ │
│                    │ ┌─────────────────────┐ │ │ • Разморозить все слои                             │ │
│                    │ │ • Начать с заморозки│ │ │ • Использовать стандартный learning rate           │ │
│                    │ │ • Потом разморозить │ │ │ • Возможно обучение с нуля                         │ │
│                    │ │   последние слои   │ │ │                                                     │ │
│                    │ └─────────────────────┘ │ └─────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### Практический пример: Классификация медицинских изображений

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torchvision import datasets, transforms, models
from torch.utils.data import DataLoader
import matplotlib.pyplot as plt

class MedicalImageClassifier:
    def __init__(self, num_classes=2, model_name='resnet50'):
        self.num_classes = num_classes
        self.model_name = model_name
        self.model = self._create_model()
        self.device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
        self.model.to(self.device)
        
    def _create_model(self):
        """Создание модели с transfer learning"""
        if self.model_name == 'resnet50':
            model = models.resnet50(pretrained=True)
            num_features = model.fc.in_features
            model.fc = nn.Linear(num_features, self.num_classes)
        elif self.model_name == 'vgg16':
            model = models.vgg16(pretrained=True)
            num_features = model.classifier[6].in_features
            model.classifier[6] = nn.Linear(num_features, self.num_classes)
        
        return model
    
    def freeze_feature_extractor(self):
        """Заморозка весов feature extractor"""
        for param in self.model.parameters():
            param.requires_grad = False
        
        # Размораживаем только классификатор
        if self.model_name == 'resnet50':
            for param in self.model.fc.parameters():
                param.requires_grad = True
        elif self.model_name == 'vgg16':
            for param in self.model.classifier.parameters():
                param.requires_grad = True
    
    def unfreeze_all(self):
        """Размораживание всех весов"""
        for param in self.model.parameters():
            param.requires_grad = True
    
    def train_epoch(self, dataloader, optimizer, criterion):
        """Обучение одной эпохи"""
        self.model.train()
        running_loss = 0.0
        correct = 0
        total = 0
        
        for inputs, labels in dataloader:
            inputs, labels = inputs.to(self.device), labels.to(self.device)
            
            optimizer.zero_grad()
            outputs = self.model(inputs)
            loss = criterion(outputs, labels)
            loss.backward()
            optimizer.step()
            
            running_loss += loss.item()
            _, predicted = torch.max(outputs.data, 1)
            total += labels.size(0)
            correct += (predicted == labels).sum().item()
        
        return running_loss / len(dataloader), 100 * correct / total
    
    def validate(self, dataloader, criterion):
        """Валидация"""
        self.model.eval()
        running_loss = 0.0
        correct = 0
        total = 0
        
        with torch.no_grad():
            for inputs, labels in dataloader:
                inputs, labels = inputs.to(self.device), labels.to(self.device)
                
                outputs = self.model(inputs)
                loss = criterion(outputs, labels)
                
                running_loss += loss.item()
                _, predicted = torch.max(outputs.data, 1)
                total += labels.size(0)
                correct += (predicted == labels).sum().item()
        
        return running_loss / len(dataloader), 100 * correct / total
    
    def two_stage_training(self, train_loader, val_loader, num_epochs_stage1=10, num_epochs_stage2=10):
        """Двухэтапное обучение"""
        criterion = nn.CrossEntropyLoss()
        
        # Этап 1: Обучение только классификатора
        print("Этап 1: Обучение классификатора...")
        self.freeze_feature_extractor()
        optimizer = optim.Adam(filter(lambda p: p.requires_grad, self.model.parameters()), 
                              lr=0.001)
        
        for epoch in range(num_epochs_stage1):
            train_loss, train_acc = self.train_epoch(train_loader, optimizer, criterion)
            val_loss, val_acc = self.validate(val_loader, criterion)
            
            print(f'Epoch {epoch+1}/{num_epochs_stage1}:')
            print(f'  Train Loss: {train_loss:.4f}, Train Acc: {train_acc:.2f}%')
            print(f'  Val Loss: {val_loss:.4f}, Val Acc: {val_acc:.2f}%')
        
        # Этап 2: Fine-tuning всей модели
        print("\nЭтап 2: Fine-tuning всей модели...")
        self.unfreeze_all()
        optimizer = optim.Adam(self.model.parameters(), lr=0.0001)  # Меньший lr
        
        for epoch in range(num_epochs_stage2):
            train_loss, train_acc = self.train_epoch(train_loader, optimizer, criterion)
            val_loss, val_acc = self.validate(val_loader, criterion)
            
            print(f'Epoch {epoch+1}/{num_epochs_stage2}:')
            print(f'  Train Loss: {train_loss:.4f}, Train Acc: {train_acc:.2f}%')
            print(f'  Val Loss: {val_loss:.4f}, Val Acc: {val_acc:.2f}%')

# Пример использования
if __name__ == "__main__":
    # Трансформации для данных
    transform = transforms.Compose([
        transforms.Resize((224, 224)),
        transforms.ToTensor(),
        transforms.Normalize(mean=[0.485, 0.456, 0.406], 
                           std=[0.229, 0.224, 0.225])
    ])
    
    # Создание классификатора
    classifier = MedicalImageClassifier(num_classes=2, model_name='resnet50')
    
    # Здесь должна быть загрузка данных
    # train_loader = DataLoader(train_dataset, batch_size=32, shuffle=True)
    # val_loader = DataLoader(val_dataset, batch_size=32, shuffle=False)
    
    # classifier.two_stage_training(train_loader, val_loader)
```

### Домашнее задание

1. **Эксперимент с различными стратегиями:** Сравните Feature Extraction vs Fine-tuning на датасете CIFAR-10
2. **Анализ слоев:** Визуализируйте активации различных слоев предобученной модели
3. **Создание пайплайна:** Реализуйте полный пайплайн transfer learning для собственного датасета

---

## 📚 Дополнительные материалы

### Рекомендованная литература
- "Deep Learning" by Ian Goodfellow, Yoshua Bengio, Aaron Courville (Глава 9)
- "Hands-On Machine Learning" by Aurélien Géron (Глава 14)
- CS231n Stanford: Convolutional Neural Networks for Visual Recognition

### Полезные ресурсы
- [Papers With Code - Computer Vision](https://paperswithcode.com/area/computer-vision)
- [PyTorch Vision Models](https://pytorch.org/vision/stable/models.html)
- [Kaggle Competitions - Computer Vision](https://www.kaggle.com/competitions)

### Практические задания
1. Реализация CNN с нуля на NumPy
2. Создание классификатора изображений с Transfer Learning
3. Участие в Kaggle соревновании по компьютерному зрению

---

**🎯 Результат блока:** После завершения этого блока вы будете понимать принципы работы CNN, сможете реализовывать современные архитектуры, применять transfer learning и решать реальные задачи компьютерного зрения с достижением высокой точности на практических датасетах.