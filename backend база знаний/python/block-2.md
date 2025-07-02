# 🔧 Блок 2: Объектно-ориентированное программирование

```
████████╗██╗  ██╗███████╗    ██████╗  ██████╗  ██████╗ 
╚══██╔══╝██║  ██║██╔════╝   ██╔═══██╗██╔═══██╗██╔══██╗
   ██║   ███████║█████╗     ██║   ██║██║   ██║██████╔╝
   ██║   ██╔══██║██╔══╝     ██║   ██║██║   ██║██╔═══╝ 
   ██║   ██║  ██║███████╗   ╚██████╔╝╚██████╔╝██║     
   ╚═╝   ╚═╝  ╚═╝╚══════╝    ╚═════╝  ╚═════╝ ╚═╝     
```

**⏱ Длительность:** 4-6 недель  
**🎯 Цель:** Освоить ООП парадигму и научиться создавать сложные программные структуры

---

## 📋 Содержание блока

```
┌─────────────────────────────────────────────────────────┐
│  2.1 Введение в ООП          │  2.5 Полиморфизм         │
│  2.2 Создание классов        │  2.6 Специальные методы  │
│  2.3 Инкапсуляция           │  2.7 Декораторы          │
│  2.4 Наследование           │  2.8 Модули и пакеты     │
└─────────────────────────────────────────────────────────┘
```

---

## 🎯 2.1 Введение в ООП

### Парадигмы программирования

```
┌─────────────────────────────────────────────────────────┐
│                ПАРАДИГМЫ ПРОГРАММИРОВАНИЯ                │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  📝 Процедурная      📦 Объектно-       🔧 Функциональная │
│     парадигма           ориентированная     парадигма     │
│                         парадигма                       │
│  ┌─────────────┐      ┌─────────────┐   ┌─────────────┐  │
│  │ def func1() │      │ class Car:  │   │ map()       │  │
│  │ def func2() │      │   methods   │   │ filter()    │  │
│  │ def func3() │      │   data      │   │ reduce()    │  │
│  └─────────────┘      └─────────────┘   └─────────────┘  │
│                                                         │
│  Данные отдельно     Данные + методы    Чистые функции  │
│  от функций          в одном объекте    без состояния   │
└─────────────────────────────────────────────────────────┘
```

### Основные принципы ООП

```
                    🏛️ ЧЕТЫРЕ СТОЛПА ООП
    ┌───────────┬───────────┬───────────┬───────────┐
    │ИНКАПСУЛЯЦИЯ│НАСЛЕДОВАНИЕ│ПОЛИМОРФИЗМ│АБСТРАКЦИЯ │
    │     🔒     │     🧬     │     🎭     │     🎨     │
    │  Скрытие   │ Получение  │ Одинаковый │ Выделение │
    │  данных    │ свойств от │ интерфейс  │ сущности  │
    │            │ родителя   │разных типов│ объектов  │
    └───────────┴───────────┴───────────┴───────────┘
```

### Класс vs Объект

```
┌─────────────────────────────────────────────────────────┐
│                    КЛАСС vs ОБЪЕКТ                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  📋 КЛАСС (Чертёж)           📦 ОБЪЕКТ (Экземпляр)      │
│                                                         │
│  class Car:                  car1 = Car()              │
│      def __init__(self):     car1.color = "red"        │
│          self.color = None   car1.start()              │
│          self.engine = False                            │
│                              car2 = Car()              │
│      def start(self):        car2.color = "blue"       │
│          self.engine = True  car2.start()              │
│                                                         │
│  🎯 Определяет структуру     🎯 Конкретная реализация   │
│     и поведение                с уникальными данными   │
└─────────────────────────────────────────────────────────┘
```

---

## 🔨 2.2 Создание классов

### Синтаксис определения класса

```python
# Базовая структура класса
class ClassName:
    """Документация класса"""
    
    # Атрибут класса (общий для всех экземпляров)
    class_attribute = "Общее значение"
    
    def __init__(self, parameter):
        """Конструктор - инициализация объекта"""
        self.instance_attribute = parameter  # Атрибут экземпляра
    
    def instance_method(self):
        """Метод экземпляра"""
        return self.instance_attribute
    
    @classmethod
    def class_method(cls):
        """Метод класса"""
        return cls.class_attribute
    
    @staticmethod
    def static_method():
        """Статический метод"""
        return "Независим от класса и экземпляра"
```

### Практический пример: Класс Автомобиль

```python
class Car:
    """Класс для представления автомобиля"""
    
    # Атрибут класса - общий для всех автомобилей
    wheels = 4
    total_cars = 0  # Счётчик созданных машин
    
    def __init__(self, brand, model, year, color="white"):
        """
        Инициализация нового автомобиля
        
        Args:
            brand (str): Марка автомобиля
            model (str): Модель автомобиля
            year (int): Год выпуска
            color (str): Цвет (по умолчанию белый)
        """
        # Атрибуты экземпляра
        self.brand = brand
        self.model = model
        self.year = year
        self.color = color
        self.engine_started = False
        self.speed = 0
        
        # Увеличиваем счётчик при создании
        Car.total_cars += 1
    
    def start_engine(self):
        """Запуск двигателя"""
        if not self.engine_started:
            self.engine_started = True
            return f"{self.brand} {self.model} - двигатель запущен! 🚗"
        return f"{self.brand} {self.model} - двигатель уже работает"
    
    def stop_engine(self):
        """Остановка двигателя"""
        if self.engine_started:
            self.engine_started = False
            self.speed = 0
            return f"{self.brand} {self.model} - двигатель остановлен"
        return f"{self.brand} {self.model} - двигатель уже выключен"
    
    def accelerate(self, speed_increase):
        """Ускорение автомобиля"""
        if self.engine_started:
            self.speed += speed_increase
            return f"Скорость увеличена до {self.speed} км/ч"
        return "Нельзя ускоряться с выключенным двигателем!"
    
    def get_info(self):
        """Получение информации об автомобиле"""
        status = "запущен" if self.engine_started else "выключен"
        return (f"🚗 {self.brand} {self.model} ({self.year})\n"
                f"   Цвет: {self.color}\n"
                f"   Двигатель: {status}\n"
                f"   Скорость: {self.speed} км/ч")
    
    @classmethod
    def get_total_cars(cls):
        """Получение общего количества созданных автомобилей"""
        return f"Всего создано автомобилей: {cls.total_cars}"
    
    @staticmethod
    def is_vintage(year):
        """Проверка, является ли автомобиль винтажным (старше 25 лет)"""
        from datetime import datetime
        current_year = datetime.now().year
        return current_year - year > 25

# Демонстрация использования
if __name__ == "__main__":
    # Создание объектов
    car1 = Car("Toyota", "Camry", 2020, "синий")
    car2 = Car("BMW", "X5", 2019)  # цвет по умолчанию
    car3 = Car("Ford", "Mustang", 1995, "красный")
    
    print("=== СОЗДАНИЕ АВТОМОБИЛЕЙ ===")
    print(car1.get_info())
    print("\n" + car2.get_info())
    print("\n" + car3.get_info())
    
    print(f"\n{Car.get_total_cars()}")
    
    print("\n=== ЗАПУСК И ДВИЖЕНИЕ ===")
    print(car1.start_engine())
    print(car1.accelerate(60))
    print(car1.accelerate(30))
    
    print(f"\n{car1.get_info()}")
    
    print(f"\n=== ПРОВЕРКА ВИНТАЖНОСТИ ===")
    print(f"Toyota Camry 2020 винтажная? {Car.is_vintage(2020)}")
    print(f"Ford Mustang 1995 винтажная? {Car.is_vintage(1995)}")
```

### Диаграмма структуры класса

```
                    📦 КЛАСС CAR
    ┌─────────────────────────────────────────────────┐
    │                                                 │
    │  🏷️  АТРИБУТЫ КЛАССА                            │
    │      wheels = 4                                 │
    │      total_cars = 0                             │
    │                                                 │
    │  🔧 МЕТОДЫ КЛАССА                               │
    │      @classmethod get_total_cars()              │
    │      @staticmethod is_vintage()                 │
    │                                                 │
    │  🏷️  АТРИБУТЫ ЭКЗЕМПЛЯРА                        │
    │      self.brand                                 │
    │      self.model                                 │
    │      self.year                                  │
    │      self.color                                 │
    │      self.engine_started                        │
    │      self.speed                                 │
    │                                                 │
    │  🔧 МЕТОДЫ ЭКЗЕМПЛЯРА                           │
    │      __init__()                                 │
    │      start_engine()                             │
    │      stop_engine()                              │
    │      accelerate()                               │
    │      get_info()                                 │
    └─────────────────────────────────────────────────┘
                           │
                           │ Создание экземпляров
                           ▼
         ┌──────────────┬──────────────┬──────────────┐
         │   car1       │    car2      │    car3      │
         │ Toyota Camry │   BMW X5     │Ford Mustang  │
         │ 2020, синий  │ 2019, белый  │1995, красный │
         └──────────────┴──────────────┴──────────────┘
```

---

## 🔒 2.3 Инкапсуляция

### Концепция сокрытия данных

```
┌─────────────────────────────────────────────────────────┐
│                   УРОВНИ ДОСТУПА                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  🟢 PUBLIC           🟡 PROTECTED         🔴 PRIVATE     │
│     attribute           _attribute          __attribute  │
│                                                         │
│  ┌─────────────┐    ┌─────────────┐     ┌─────────────┐ │
│  │ Доступен    │    │ Внутреннее  │     │ Строго      │ │
│  │ отовсюду    │    │ использо-   │     │ внутри      │ │
│  │             │    │ вание       │     │ класса      │ │
│  │ obj.attr    │    │ obj._attr   │     │ obj.__attr  │ │
│  │             │    │ (конвенция) │     │ (name       │ │
│  │             │    │             │     │ mangling)   │ │
│  └─────────────┘    └─────────────┘     └─────────────┘ │
└─────────────────────────────────────────────────────────┘
```

### Практический пример: Банковский счёт

```python
class BankAccount:
    """Банковский счёт с инкапсуляцией данных"""
    
    def __init__(self, account_number, initial_balance=0):
        # Публичные атрибуты
        self.account_number = account_number
        self.owner = None
        
        # Защищённые атрибуты (внутреннее использование)
        self._creation_date = self._get_current_date()
        self._transaction_history = []
        
        # Приватные атрибуты (строго внутри класса)
        self.__balance = initial_balance
        self.__pin = None
        self.__is_frozen = False
    
    @property
    def balance(self):
        """Получение баланса (только для чтения)"""
        if self.__is_frozen:
            return "Счёт заморожен"
        return self.__balance
    
    @property
    def pin(self):
        """PIN нельзя получить напрямую"""
        return "****"
    
    @pin.setter
    def pin(self, new_pin):
        """Установка PIN с валидацией"""
        if isinstance(new_pin, str) and len(new_pin) == 4 and new_pin.isdigit():
            self.__pin = new_pin
            self._add_transaction("PIN изменён")
        else:
            raise ValueError("PIN должен состоять из 4 цифр")
    
    def deposit(self, amount):
        """Пополнение счёта"""
        if self.__is_frozen:
            return "Операция невозможна: счёт заморожен"
        
        if amount > 0:
            self.__balance += amount
            self._add_transaction(f"Пополнение: +{amount}")
            return f"Счёт пополнен на {amount}. Баланс: {self.__balance}"
        else:
            return "Сумма должна быть положительной"
    
    def withdraw(self, amount, entered_pin):
        """Снятие средств с проверкой PIN"""
        if self.__is_frozen:
            return "Операция невозможна: счёт заморожен"
        
        if not self._verify_pin(entered_pin):
            return "Неверный PIN"
        
        if 0 < amount <= self.__balance:
            self.__balance -= amount
            self._add_transaction(f"Снятие: -{amount}")
            return f"Снято {amount}. Баланс: {self.__balance}"
        else:
            return "Недостаточно средств или неверная сумма"
    
    def _verify_pin(self, entered_pin):
        """Проверка PIN (защищённый метод)"""
        return self.__pin == entered_pin
    
    def _add_transaction(self, description):
        """Добавление записи в историю (защищённый метод)"""
        timestamp = self._get_current_date()
        self._transaction_history.append(f"{timestamp}: {description}")
    
    def _get_current_date(self):
        """Получение текущей даты (защищённый метод)"""
        from datetime import datetime
        return datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    
    def __freeze_account(self):
        """Заморозка счёта (приватный метод)"""
        self.__is_frozen = True
        self._add_transaction("Счёт заморожен")
    
    def get_transaction_history(self, entered_pin):
        """Получение истории транзакций"""
        if not self._verify_pin(entered_pin):
            return "Неверный PIN"
        
        if not self._transaction_history:
            return "История транзакций пуста"
        
        history = "\n=== ИСТОРИЯ ТРАНЗАКЦИЙ ===\n"
        for transaction in self._transaction_history[-10:]:  # Последние 10
            history += f"{transaction}\n"
        return history
    
    def get_account_info(self):
        """Общая информация о счёте"""
        status = "заморожен" if self.__is_frozen else "активен"
        return (f"💳 Счёт: {self.account_number}\n"
                f"   Владелец: {self.owner or 'Не указан'}\n"
                f"   Создан: {self._creation_date}\n"
                f"   Статус: {status}")

# Демонстрация инкапсуляции
if __name__ == "__main__":
    # Создание счёта
    account = BankAccount("123456789", 1000)
    account.owner = "Иван Петров"
    account.pin = "1234"
    
    print("=== ИНФОРМАЦИЯ О СЧЁТЕ ===")
    print(account.get_account_info())
    print(f"Баланс: {account.balance}")
    
    print("\n=== ОПЕРАЦИИ СО СЧЁТОМ ===")
    print(account.deposit(500))
    print(account.withdraw(200, "1234"))
    print(account.withdraw(100, "0000"))  # Неверный PIN
    
    print(f"\nТекущий баланс: {account.balance}")
    
    print("\n=== ИСТОРИЯ ТРАНЗАКЦИЙ ===")
    print(account.get_transaction_history("1234"))
    
    print("\n=== ДЕМОНСТРАЦИЯ ИНКАПСУЛЯЦИИ ===")
    # Попытка прямого доступа к приватным атрибутам
    print(f"PIN (через property): {account.pin}")
    
    # Защищённые атрибуты доступны, но не рекомендуется
    print(f"Дата создания: {account._creation_date}")
    
    # Приватные атрибуты недоступны напрямую
    try:
        print(account.__balance)  # Вызовет AttributeError
    except AttributeError as e:
        print(f"Ошибка доступа к приватному атрибуту: {e}")
    
    # Name mangling - Python изменяет имя приватного атрибута
    print(f"Реальное имя приватного атрибута: {dir(account)}")
```

### Свойства (Properties)

```python
class Temperature:
    """Класс для работы с температурой с автоматическими преобразованиями"""
    
    def __init__(self, celsius=0):
        self._celsius = celsius
    
    @property
    def celsius(self):
        """Температура в Цельсиях"""
        return self._celsius
    
    @celsius.setter
    def celsius(self, value):
        """Установка температуры в Цельсиях с валидацией"""
        if value < -273.15:
            raise ValueError("Температура не может быть ниже абсолютного нуля!")
        self._celsius = value
    
    @property
    def fahrenheit(self):
        """Температура в Фаренгейтах (только для чтения)"""
        return self._celsius * 9/5 + 32
    
    @fahrenheit.setter
    def fahrenheit(self, value):
        """Установка температуры через Фаренгейты"""
        celsius_value = (value - 32) * 5/9
        self.celsius = celsius_value  # Используем setter для валидации
    
    @property
    def kelvin(self):
        """Температура в Кельвинах (только для чтения)"""
        return self._celsius + 273.15
    
    @kelvin.setter
    def kelvin(self, value):
        """Установка температуры через Кельвины"""
        if value < 0:
            raise ValueError("Температура в Кельвинах не может быть отрицательной!")
        self.celsius = value - 273.15
    
    def __str__(self):
        return f"{self._celsius:.1f}°C"
    
    def __repr__(self):
        return f"Temperature({self._celsius})"

# Демонстрация свойств
temp = Temperature(25)
print(f"Температура: {temp}")
print(f"В Фаренгейтах: {temp.fahrenheit:.1f}°F")
print(f"В Кельвинах: {temp.kelvin:.1f}K")

# Изменение через разные свойства
temp.fahrenheit = 100  # Кипение воды
print(f"После установки 100°F: {temp}")

temp.kelvin = 273.15  # Замерзание воды
print(f"После установки 273.15K: {temp}")
```

---

## 🧬 2.4 Наследование

### Концепция наследования

```
                        🌳 ДЕРЕВО НАСЛЕДОВАНИЯ
    
                         ┌─────────────┐
                         │   Animal    │
                         │   (базовый) │
                         └──────┬──────┘
                                │
                    ┌───────────┼───────────┐
                    │                       │
            ┌─────────────┐         ┌─────────────┐
            │   Mammal    │         │    Bird     │
            │ (наследник) │         │ (наследник) │
            └──────┬──────┘         └──────┬──────┘
                   │                       │
         ┌─────────┼─────────┐            │
         │                   │            │
    ┌─────────┐      ┌─────────────┐ ┌─────────┐
    │   Dog   │      │    Cat      │ │  Eagle  │
    │(внучек) │      │  (внучек)   │ │(внучек) │
    └─────────┘      └─────────────┘ └─────────┘
```

### Базовый пример наследования

```python
# Базовый класс
class Animal:
    """Базовый класс для всех животных"""
    
    def __init__(self, name, age):
        self.name = name
        self.age = age
        self.energy = 100
    
    def eat(self, food_amount):
        """Питание животного"""
        self.energy = min(100, self.energy + food_amount)
        return f"{self.name} поел и восстановил энергию до {self.energy}"
    
    def sleep(self):
        """Сон животного"""
        self.energy = 100
        return f"{self.name} поспал и полностью восстановился"
    
    def make_sound(self):
        """Базовый звук (будет переопределён в наследниках)"""
        return f"{self.name} издаёт звук"
    
    def get_info(self):
        """Информация о животном"""
        return f"🐾 {self.name}, возраст: {self.age}, энергия: {self.energy}"

# Наследник - класс млекопитающих
class Mammal(Animal):
    """Класс млекопитающих"""
    
    def __init__(self, name, age, fur_color):
        super().__init__(name, age)  # Вызов конструктора родителя
        self.fur_color = fur_color
        self.body_temperature = 36.6
    
    def regulate_temperature(self):
        """Терморегуляция млекопитающих"""
        return f"{self.name} поддерживает температуру тела {self.body_temperature}°C"
    
    def get_info(self):
        """Расширенная информация для млекопитающих"""
        base_info = super().get_info()  # Получаем базовую информацию
        return f"{base_info}, цвет шерсти: {self.fur_color}"

# Наследник млекопитающего - собака
class Dog(Mammal):
    """Класс собак"""
    
    def __init__(self, name, age, fur_color, breed):
        super().__init__(name, age, fur_color)
        self.breed = breed
        self.loyalty = 100
        self.trained_commands = []
    
    def make_sound(self):
        """Переопределение звука для собаки"""
        return f"{self.name} лает: Гав-гав!"
    
    def bark(self, intensity="обычно"):
        """Специфичный метод для собак"""
        sounds = {
            "тихо": "гав",
            "обычно": "Гав-гав!",
            "громко": "ГАВ-ГАВ-ГАВ!"
        }
        return f"{self.name} лает {intensity}: {sounds.get(intensity, sounds['обычно'])}"
    
    def learn_command(self, command):
        """Обучение команде"""
        if command not in self.trained_commands:
            self.trained_commands.append(command)
            return f"{self.name} выучил команду '{command}'!"
        return f"{self.name} уже знает команду '{command}'"
    
    def execute_command(self, command):
        """Выполнение команды"""
        if command in self.trained_commands:
            self.energy -= 10
            responses = {
                "сидеть": f"{self.name} сел",
                "лежать": f"{self.name} лёг",
                "дать лапу": f"{self.name} дал лапу",
                "апорт": f"{self.name} принёс мячик"
            }
            return responses.get(command, f"{self.name} выполнил команду '{command}'")
        return f"{self.name} не знает команду '{command}'"
    
    def get_info(self):
        """Полная информация о собаке"""
        base_info = super().get_info()
        commands = ", ".join(self.trained_commands) if self.trained_commands else "нет"
        return (f"{base_info}, порода: {self.breed}\n"
                f"   Лояльность: {self.loyalty}, Команды: {commands}")

# Другой наследник млекопитающего - кот
class Cat(Mammal):
    """Класс котов"""
    
    def __init__(self, name, age, fur_color, indoor=True):
        super().__init__(name, age, fur_color)
        self.indoor = indoor
        self.independence = 80
        self.affection = 50
    
    def make_sound(self):
        """Переопределение звука для кота"""
        return f"{self.name} мяукает: Мяу!"
    
    def purr(self):
        """Специфичный метод для котов"""
        self.affection += 10
        return f"{self.name} мурлычет: мур-мур-мур 😸"
    
    def hunt(self):
        """Охота (только для уличных котов)"""
        if not self.indoor:
            self.energy -= 20
            return f"{self.name} поохотился на улице"
        return f"{self.name} домашний кот и не охотится"
    
    def climb(self):
        """Лазанье"""
        self.energy -= 5
        return f"{self.name} залез на высокое место"

# Другая ветка наследования - птицы
class Bird(Animal):
    """Класс птиц"""
    
    def __init__(self, name, age, wingspan):
        super().__init__(name, age)
        self.wingspan = wingspan
        self.can_fly = True
    
    def make_sound(self):
        """Звук птиц"""
        return f"{self.name} чирикает"
    
    def fly(self, distance):
        """Полёт"""
        if self.can_fly and self.energy >= 20:
            self.energy -= 20
            return f"{self.name} пролетел {distance} метров"
        elif not self.can_fly:
            return f"{self.name} не умеет летать"
        else:
            return f"{self.name} слишком устал для полёта"
    
    def get_info(self):
        """Информация о птице"""
        base_info = super().get_info()
        flight_status = "умеет летать" if self.can_fly else "не летает"
        return f"{base_info}, размах крыльев: {self.wingspan}см, {flight_status}"

# Демонстрация наследования
if __name__ == "__main__":
    # Создание животных
    dog = Dog("Бобик", 3, "коричневый", "Лабрадор")
    cat = Cat("Мурка", 2, "серый", indoor=True)
    bird = Bird("Чижик", 1, 15)
    
    print("=== ИНФОРМАЦИЯ О ЖИВОТНЫХ ===")
    print(dog.get_info())
    print(cat.get_info())
    print(bird.get_info())
    
    print("\n=== ЗВУКИ ЖИВОТНЫХ ===")
    print(dog.make_sound())
    print(cat.make_sound())
    print(bird.make_sound())
    
    print("\n=== СПЕЦИФИЧНЫЕ ДЕЙСТВИЯ ===")
    # Собака
    print(dog.learn_command("сидеть"))
    print(dog.learn_command("дать лапу"))
    print(dog.execute_command("сидеть"))
    print(dog.bark("громко"))
    
    # Кот
    print(cat.purr())
    print(cat.climb())
    print(cat.hunt())
    
    # Птица
    print(bird.fly(50))
    
    print("\n=== ОБЩИЕ МЕТОДЫ (НАСЛЕДУЕМЫЕ) ===")
    print(dog.eat(20))
    print(cat.sleep())
    print(bird.eat(15))
```

### Множественное наследование

```python
# Миксины для добавления функциональности
class FlyingMixin:
    """Миксин для добавления способности к полёту"""
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.altitude = 0
        self.max_altitude = 1000
    
    def take_off(self):
        """Взлёт"""
        if self.altitude == 0:
            self.altitude = 10
            return f"{self.name} взлетел на высоту {self.altitude}м"
        return f"{self.name} уже в воздухе"
    
    def land(self):
        """Посадка"""
        if self.altitude > 0:
            self.altitude = 0
            return f"{self.name} приземлился"
        return f"{self.name} уже на земле"
    
    def change_altitude(self, delta):
        """Изменение высоты"""
        new_altitude = max(0, min(self.max_altitude, self.altitude + delta))
        old_altitude = self.altitude
        self.altitude = new_altitude
        
        if delta > 0:
            return f"{self.name} поднялся с {old_altitude}м до {new_altitude}м"
        elif delta < 0:
            return f"{self.name} снизился с {old_altitude}м до {new_altitude}м"
        else:
            return f"{self.name} остался на высоте {self.altitude}м"

class SwimmingMixin:
    """Миксин для добавления способности к плаванию"""
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.depth = 0
        self.max_depth = 100
    
    def dive(self, target_depth):
        """Погружение"""
        if target_depth <= self.max_depth:
            self.depth = target_depth
            return f"{self.name} нырнул на глубину {self.depth}м"
        return f"{self.name} не может нырнуть так глубоко!"
    
    def surface(self):
        """Всплытие"""
        if self.depth > 0:
            self.depth = 0
            return f"{self.name} всплыл на поверхность"
        return f"{self.name} уже на поверхности"
    
    def swim(self, distance):
        """Плавание"""
        self.energy -= 10
        return f"{self.name} проплыл {distance}м на глубине {self.depth}м"

# Классы с множественным наследованием
class Duck(Bird, FlyingMixin, SwimmingMixin):
    """Утка - летает, плавает и ходит"""
    
    def __init__(self, name, age):
        # Важен порядок вызова super() при множественном наследовании
        super().__init__(name, age, wingspan=40)
        self.waterproof = True
    
    def make_sound(self):
        return f"{self.name} крякает: Кря-кря!"
    
    def get_info(self):
        base_info = super().get_info()
        return f"{base_info}, высота: {self.altitude}м, глубина: {self.depth}м"

class Penguin(Bird, SwimmingMixin):
    """Пингвин - плавает, но не летает"""
    
    def __init__(self, name, age):
        super().__init__(name, age, wingspan=30)
        self.can_fly = False  # Пингвины не летают
        self.max_depth = 500  # Глубоко ныряют
    
    def make_sound(self):
        return f"{self.name} издаёт звук пингвина"
    
    def slide_on_ice(self):
        """Скольжение по льду"""
        return f"{self.name} скользит по льду на животе!"

# Демонстрация множественного наследования
if __name__ == "__main__":
    duck = Duck("Дональд", 2)
    penguin = Penguin("Пингвиненок", 1)
    
    print("=== УТКА (летает и плавает) ===")
    print(duck.get_info())
    print(duck.make_sound())
    print(duck.take_off())
    print(duck.change_altitude(50))
    print(duck.land())
    print(duck.dive(5))
    print(duck.swim(20))
    print(duck.surface())
    
    print("\n=== ПИНГВИН (только плавает) ===")
    print(penguin.get_info())
    print(penguin.make_sound())
    print(penguin.fly(10))  # Не сможет
    print(penguin.dive(100))
    print(penguin.swim(50))
    print(penguin.slide_on_ice())
    
    print("\n=== MRO (Method Resolution Order) ===")
    print("Порядок разрешения методов для Duck:")
    for i, cls in enumerate(Duck.__mro__):
        print(f"  {i+1}. {cls.__name__}")
```

### Диаграмма MRO (Method Resolution Order)

```
                    MRO для класса Duck
    ┌─────────────────────────────────────────────────────┐
    │  1. Duck                                            │
    │  2. Bird                                            │
    │  3. FlyingMixin                                     │
    │  4. SwimmingMixin                                   │
    │  5. Animal                                          │
    │  6. object                                          │
    └─────────────────────────────────────────────────────┘
                           │
                    Порядок поиска методов
                           ▼
            При вызове super() Python ищет метод
            в следующем классе по этому списку
```

---

## 🎭 2.5 Полиморфизм

### Концепция полиморфизма

```
┌─────────────────────────────────────────────────────────┐
│                    ПОЛИМОРФИЗМ                          │
│              "Много форм одного поведения"              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  🎭 Один интерфейс → Разное поведение                   │
│                                                         │
│     animal.make_sound()                                 │
│           │                                             │
│      ┌────┼────┬────────────┬────────────┐              │
│      │         │            │            │              │
│   🐕 Dog    🐱 Cat      🐦 Bird     🦆 Duck            │
│   "Гав!"   "Мяу!"    "Чирик"     "Кря!"               │
│                                                         │
│  Один вызов → Разные результаты в зависимости от типа   │
└─────────────────────────────────────────────────────────┘
```

### Практический пример полиморфизма

```python
from abc import ABC, abstractmethod
import math

# Абстрактный базовый класс
class Shape(ABC):
    """Абстрактный класс для геометрических фигур"""
    
    def __init__(self, color="белый"):
        self.color = color
    
    @abstractmethod
    def area(self):
        """Абстрактный метод для вычисления площади"""
        pass
    
    @abstractmethod
    def perimeter(self):
        """Абстрактный метод для вычисления периметра"""
        pass
    
    def describe(self):
        """Общий метод описания фигуры"""
        return f"{self.__class__.__name__} {self.color} цвета"
    
    def paint(self, new_color):
        """Изменение цвета фигуры"""
        old_color = self.color
        self.color = new_color
        return f"Фигура перекрашена с {old_color} на {new_color}"

# Конкретные реализации
class Rectangle(Shape):
    """Прямоугольник"""
    
    def __init__(self, width, height, color="белый"):
        super().__init__(color)
        self.width = width
        self.height = height
    
    def area(self):
        """Площадь прямоугольника"""
        return self.width * self.height
    
    def perimeter(self):
        """Периметр прямоугольника"""
        return 2 * (self.width + self.height)
    
    def is_square(self):
        """Проверка, является ли прямоугольник квадратом"""
        return self.width == self.height
    
    def __str__(self):
        square_info = " (квадрат)" if self.is_square() else ""
        return (f"📐 Прямоугольник{square_info}: {self.width}×{self.height}, "
                f"цвет: {self.color}")

class Circle(Shape):
    """Круг"""
    
    def __init__(self, radius, color="белый"):
        super().__init__(color)
        self.radius = radius
    
    def area(self):
        """Площадь круга"""
        return math.pi * self.radius ** 2
    
    def perimeter(self):
        """Длина окружности"""
        return 2 * math.pi * self.radius
    
    def diameter(self):
        """Диаметр круга"""
        return 2 * self.radius
    
    def __str__(self):
        return f"⭕ Круг: радиус {self.radius}, цвет: {self.color}"

class Triangle(Shape):
    """Треугольник"""
    
    def __init__(self, side_a, side_b, side_c, color="белый"):
        super().__init__(color)
        self.side_a = side_a
        self.side_b = side_b
        self.side_c = side_c
        
        # Проверка возможности существования треугольника
        if not self._is_valid_triangle():
            raise ValueError("Невозможно построить треугольник с такими сторонами!")
    
    def _is_valid_triangle(self):
        """Проверка правила треугольника"""
        return (self.side_a + self.side_b > self.side_c and
                self.side_a + self.side_c > self.side_b and
                self.side_b + self.side_c > self.side_a)
    
    def area(self):
        """Площадь треугольника по формуле Герона"""
        s = self.perimeter() / 2  # Полупериметр
        return math.sqrt(s * (s - self.side_a) * (s - self.side_b) * (s - self.side_c))
    
    def perimeter(self):
        """Периметр треугольника"""
        return self.side_a + self.side_b + self.side_c
    
    def triangle_type(self):
        """Определение типа треугольника"""
        sides = sorted([self.side_a, self.side_b, self.side_c])
        
        if sides[0] == sides[1] == sides[2]:
            return "равносторонний"
        elif sides[0] == sides[1] or sides[1] == sides[2]:
            return "равнобедренный"
        else:
            return "разносторонний"
    
    def __str__(self):
        return (f"🔺 Треугольник ({self.triangle_type()}): "
                f"{self.side_a}-{self.side_b}-{self.side_c}, цвет: {self.color}")

# Полиморфные функции
def calculate_total_area(shapes):
    """Вычисление общей площади всех фигур"""
    total = 0
    print("=== ВЫЧИСЛЕНИЕ ПЛОЩАДЕЙ ===")
    
    for shape in shapes:
        area = shape.area()
        total += area
        print(f"{shape}: площадь = {area:.2f}")
    
    return total

def paint_all_shapes(shapes, color):
    """Покраска всех фигур в один цвет"""
    print(f"\n=== ПОКРАСКА В {color.upper()} ЦВЕТ ===")
    
    for shape in shapes:
        print(shape.paint(color))

def shape_comparison(shapes):
    """Сравнение характеристик фигур"""
    print("\n=== СРАВНЕНИЕ ФИГУР ===")
    
    if not shapes:
        return
    
    # Находим фигуру с максимальной площадью
    max_area_shape = max(shapes, key=lambda s: s.area())
    min_area_shape = min(shapes, key=lambda s: s.area())
    
    print(f"Наибольшая площадь: {max_area_shape} ({max_area_shape.area():.2f})")
    print(f"Наименьшая площадь: {min_area_shape} ({min_area_shape.area():.2f})")
    
    # Находим фигуру с максимальным периметром
    max_perimeter_shape = max(shapes, key=lambda s: s.perimeter())
    print(f"Наибольший периметр: {max_perimeter_shape} ({max_perimeter_shape.perimeter():.2f})")

# Демонстрация полиморфизма
if __name__ == "__main__":
    # Создание коллекции разных фигур
    shapes = [
        Rectangle(5, 3, "красный"),
        Circle(4, "синий"),
        Triangle(3, 4, 5, "зелёный"),
        Rectangle(4, 4, "жёлтый"),  # Квадрат
        Circle(2.5, "фиолетовый"),
        Triangle(6, 6, 6, "оранжевый")  # Равносторонний
    ]
    
    # Вывод информации о всех фигурах
    print("=== СПИСОК ФИГУР ===")
    for i, shape in enumerate(shapes, 1):
        print(f"{i}. {shape}")
    
    # Полиморфное вычисление площадей
    total_area = calculate_total_area(shapes)
    print(f"\nОбщая площадь всех фигур: {total_area:.2f}")
    
    # Полиморфная покраска
    paint_all_shapes(shapes, "розовый")
    
    # Сравнение фигур
    shape_comparison(shapes)
    
    # Демонстрация специфичных методов
    print("\n=== СПЕЦИФИЧНЫЕ МЕТОДЫ ===")
    for shape in shapes:
        if isinstance(shape, Rectangle):
            if shape.is_square():
                print(f"{shape} - это квадрат!")
        elif isinstance(shape, Circle):
            print(f"{shape} - диаметр: {shape.diameter():.2f}")
        elif isinstance(shape, Triangle):
            print(f"{shape} - тип: {shape.triangle_type()}")
    
    print("\n=== УТИНАЯ ТИПИЗАЦИЯ ===")
    # Все объекты с методами area() и perimeter() могут использоваться
    # одинаково, независимо от их типа
    
    def print_shape_info(shape):
        """Функция работает с любым объектом, имеющим нужные методы"""
        try:
            print(f"Объект: {shape.describe()}")
            print(f"Площадь: {shape.area():.2f}")
            print(f"Периметр: {shape.perimeter():.2f}")
            print("-" * 30)
        except AttributeError as e:
            print(f"Объект не поддерживает необходимые методы: {e}")
    
    # Тестируем с нашими фигурами
    for shape in shapes[:3]:
        print_shape_info(shape)
```

### Протоколы и структурная подтипизация

```python
from typing import Protocol

class Drawable(Protocol):
    """Протокол для объектов, которые можно рисовать"""
    
    def draw(self) -> str:
        """Метод для рисования объекта"""
        ...
    
    def get_position(self) -> tuple[float, float]:
        """Получение позиции объекта"""
        ...

class MovableDrawable(Protocol):
    """Протокол для объектов, которые можно рисовать и перемещать"""
    
    def draw(self) -> str:
        ...
    
    def move(self, dx: float, dy: float) -> None:
        ...
    
    def get_position(self) -> tuple[float, float]:
        ...

# Классы, реализующие протоколы (неявно)
class DrawableCircle:
    """Круг, который можно рисовать"""
    
    def __init__(self, x=0, y=0, radius=1):
        self.x = x
        self.y = y
        self.radius = radius
    
    def draw(self):
        return f"Рисую круг радиусом {self.radius} в точке ({self.x}, {self.y})"
    
    def get_position(self):
        return (self.x, self.y)
    
    def move(self, dx, dy):
        self.x += dx
        self.y += dy

class DrawableText:
    """Текст, который можно рисовать"""
    
    def __init__(self, text, x=0, y=0):
        self.text = text
        self.x = x
        self.y = y
    
    def draw(self):
        return f"Рисую текст '{self.text}' в точке ({self.x}, {self.y})"
    
    def get_position(self):
        return (self.x, self.y)
    
    def move(self, dx, dy):
        self.x += dx
        self.y += dy

# Функции, работающие с протоколами
def render_scene(drawables: list[Drawable]):
    """Отрисовка сцены с любыми рисуемыми объектами"""
    print("=== ОТРИСОВКА СЦЕНЫ ===")
    for drawable in drawables:
        print(drawable.draw())

def move_objects(objects: list[MovableDrawable], dx: float, dy: float):
    """Перемещение всех объектов"""
    print(f"\n=== ПЕРЕМЕЩЕНИЕ НА ({dx}, {dy}) ===")
    for obj in objects:
        old_pos = obj.get_position()
        obj.move(dx, dy)
        new_pos = obj.get_position()
        print(f"Объект перемещён с {old_pos} в {new_pos}")

# Демонстрация протоколов
if __name__ == "__main__":
    # Создание объектов
    circle = DrawableCircle(10, 20, 5)
    text = DrawableText("Привет, мир!", 0, 0)
    
    # Все объекты автоматически соответствуют протоколам
    drawables = [circle, text]
    
    render_scene(drawables)
    move_objects(drawables, 5, -3)
    render_scene(drawables)
```

---

## ⚡ 2.6 Специальные методы (Magic Methods)

### Обзор магических методов

```
┌─────────────────────────────────────────────────────────┐
│                  МАГИЧЕСКИЕ МЕТОДЫ                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  🎯 СОЗДАНИЕ И УНИЧТОЖЕНИЕ                              │
│     __new__    __init__    __del__                      │
│                                                         │
│  📝 ПРЕДСТАВЛЕНИЕ                                       │
│     __str__    __repr__    __format__                   │
│                                                         │
│  🔢 АРИФМЕТИЧЕСКИЕ ОПЕРАЦИИ                             │
│     __add__    __sub__    __mul__    __truediv__        │
│     __eq__     __lt__     __le__     __gt__             │
│                                                         │
│  📦 КОНТЕЙНЕРЫ                                          │
│     __len__    __getitem__    __setitem__               │
│     __contains__    __iter__    __next__                │
│                                                         │
│  🔧 КОНТЕКСТНЫЕ МЕНЕДЖЕРЫ                               │
│     __enter__    __exit__                               │
└─────────────────────────────────────────────────────────┘
```

### Практический пример: Класс Vector

```python
import math
from typing import Iterator

class Vector:
    """Математический вектор с перегруженными операторами"""
    
    def __init__(self, *components):
        """Инициализация вектора с компонентами"""
        if not components:
            raise ValueError("Вектор должен иметь хотя бы одну компоненту")
        self._components = list(components)
    
    # === ПРЕДСТАВЛЕНИЕ ОБЪЕКТА ===
    
    def __str__(self):
        """Строковое представление для пользователя"""
        components_str = ", ".join(str(comp) for comp in self._components)
        return f"Vector({components_str})"
    
    def __repr__(self):
        """Техническое представление для разработчика"""
        components_str = ", ".join(repr(comp) for comp in self._components)
        return f"Vector({components_str})"
    
    def __format__(self, format_spec):
        """Форматированный вывод"""
        if format_spec == "":
            return str(self)
        elif format_spec == "components":
            return f"[{', '.join(f'{comp:.2f}' for comp in self._components)}]"
        elif format_spec == "magnitude":
            return f"|{self}| = {self.magnitude():.2f}"
        else:
            return str(self)
    
    # === АРИФМЕТИЧЕСКИЕ ОПЕРАЦИИ ===
    
    def __add__(self, other):
        """Сложение векторов: v1 + v2"""
        if isinstance(other, Vector):
            if len(self) != len(other):
                raise ValueError("Векторы должны иметь одинаковую размерность")
            return Vector(*(a + b for a, b in zip(self._components, other._components)))
        elif isinstance(other, (int, float)):
            return Vector(*(comp + other for comp in self._components))
        return NotImplemented
    
    def __radd__(self, other):
        """Обратное сложение: число + вектор"""
        return self.__add__(other)
    
    def __sub__(self, other):
        """Вычитание векторов: v1 - v2"""
        if isinstance(other, Vector):
            if len(self) != len(other):
                raise ValueError("Векторы должны иметь одинаковую размерность")
            return Vector(*(a - b for a, b in zip(self._components, other._components)))
        elif isinstance(other, (int, float)):
            return Vector(*(comp - other for comp in self._components))
        return NotImplemented
    
    def __rsub__(self, other):
        """Обратное вычитание: число - вектор"""
        if isinstance(other, (int, float)):
            return Vector(*(other - comp for comp in self._components))
        return NotImplemented
    
    def __mul__(self, other):
        """Умножение вектора на скаляр: v * число"""
        if isinstance(other, (int, float)):
            return Vector(*(comp * other for comp in self._components))
        return NotImplemented
    
    def __rmul__(self, other):
        """Обратное умножение: число * вектор"""
        return self.__mul__(other)
    
    def __truediv__(self, other):
        """Деление вектора на скаляр: v / число"""
        if isinstance(other, (int, float)):
            if other == 0:
                raise ZeroDivisionError("Деление на ноль")
            return Vector(*(comp / other for comp in self._components))
        return NotImplemented
    
    def __neg__(self):
        """Унарный минус: -v"""
        return Vector(*(-comp for comp in self._components))
    
    def __pos__(self):
        """Унарный плюс: +v"""
        return Vector(*self._components)
    
    def __abs__(self):
        """Модуль вектора: abs(v)"""
        return self.magnitude()
    
    # === ОПЕРАЦИИ СРАВНЕНИЯ ===
    
    def __eq__(self, other):
        """Равенство векторов: v1 == v2"""
        if isinstance(other, Vector):
            return self._components == other._components
        return False
    
    def __ne__(self, other):
        """Неравенство векторов: v1 != v2"""
        return not self.__eq__(other)
    
    def __lt__(self, other):
        """Сравнение по длине: v1 < v2"""
        if isinstance(other, Vector):
            return self.magnitude() < other.magnitude()
        return NotImplemented
    
    def __le__(self, other):
        """Сравнение по длине: v1 <= v2"""
        if isinstance(other, Vector):
            return self.magnitude() <= other.magnitude()
        return NotImplemented
    
    def __gt__(self, other):
        """Сравнение по длине: v1 > v2"""
        if isinstance(other, Vector):
            return self.magnitude() > other.magnitude()
        return NotImplemented
    
    def __ge__(self, other):
        """Сравнение по длине: v1 >= v2"""
        if isinstance(other, Vector):
            return self.magnitude() >= other.magnitude()
        return NotImplemented
    
    # === КОНТЕЙНЕРНЫЕ ОПЕРАЦИИ ===
    
    def __len__(self):
        """Размерность вектора: len(v)"""
        return len(self._components)
    
    def __getitem__(self, index):
        """Доступ к компоненте: v[i]"""
        return self._components[index]
    
    def __setitem__(self, index, value):
        """Изменение компоненты: v[i] = value"""
        self._components[index] = value
    
    def __contains__(self, value):
        """Проверка содержания: value in v"""
        return value in self._components
    
    def __iter__(self):
        """Итерация по компонентам: for comp in v"""
        return iter(self._components)
    
    def __reversed__(self):
        """Обратная итерация: reversed(v)"""
        return reversed(self._components)
    
    # === ДОПОЛНИТЕЛЬНЫЕ МАТЕМАТИЧЕСКИЕ МЕТОДЫ ===
    
    def magnitude(self):
        """Вычисление длины (модуля) вектора"""
        return math.sqrt(sum(comp ** 2 for comp in self._components))
    
    def normalize(self):
        """Нормализация вектора (единичный вектор)"""
        mag = self.magnitude()
        if mag == 0:
            raise ValueError("Нельзя нормализовать нулевой вектор")
        return self / mag
    
    def dot(self, other):
        """Скалярное произведение векторов"""
        if not isinstance(other, Vector):
            raise TypeError("Скалярное произведение возможно только с другим вектором")
        if len(self) != len(other):
            raise ValueError("Векторы должны иметь одинаковую размерность")
        return sum(a * b for a, b in zip(self._components, other._components))
    
    def cross(self, other):
        """Векторное произведение (только для 3D векторов)"""
        if not isinstance(other, Vector):
            raise TypeError("Векторное произведение возможно только с другим вектором")
        if len(self) != 3 or len(other) != 3:
            raise ValueError("Векторное произведение определено только для 3D векторов")
        
        x1, y1, z1 = self._components
        x2, y2, z2 = other._components
        
        return Vector(
            y1 * z2 - z1 * y2,
            z1 * x2 - x1 * z2,
            x1 * y2 - y1 * x2
        )
    
    def angle(self, other):
        """Угол между векторами в радианах"""
        if not isinstance(other, Vector):
            raise TypeError("Угол можно вычислить только с другим вектором")
        
        dot_product = self.dot(other)
        magnitudes = self.magnitude() * other.magnitude()
        
        if magnitudes == 0:
            raise ValueError("Нельзя вычислить угол с нулевым вектором")
        
        cos_angle = dot_product / magnitudes
        # Ограничиваем значение для избежания ошибок округления
        cos_angle = max(-1, min(1, cos_angle))
        return math.acos(cos_angle)

# Демонстрация магических методов
if __name__ == "__main__":
    # Создание векторов
    v1 = Vector(3, 4)
    v2 = Vector(1, 2)
    v3 = Vector(2, 1, 2)
    
    print("=== СОЗДАНИЕ И ПРЕДСТАВЛЕНИЕ ===")
    print(f"v1 = {v1}")
    print(f"v2 = {v2}")
    print(f"v3 = {v3}")
    print(f"repr(v1) = {repr(v1)}")
    print(f"Форматирование v1: {v1:components}")
    print(f"Форматирование v1: {v1:magnitude}")
    
    print("\n=== АРИФМЕТИЧЕСКИЕ ОПЕРАЦИИ ===")
    print(f"v1 + v2 = {v1 + v2}")
    print(f"v1 - v2 = {v1 - v2}")
    print(f"v1 * 3 = {v1 * 3}")
    print(f"2 * v1 = {2 * v1}")
    print(f"v1 / 2 = {v1 / 2}")
    print(f"-v1 = {-v1}")
    print(f"abs(v1) = {abs(v1):.2f}")
    
    print("\n=== ОПЕРАЦИИ СРАВНЕНИЯ ===")
    print(f"v1 == v2: {v1 == v2}")
    print(f"v1 > v2: {v1 > v2}")
    print(f"len(v1): {len(v1)}")
    print(f"len(v3): {len(v3)}")
    
    print("\n=== КОНТЕЙНЕРНЫЕ ОПЕРАЦИИ ===")
    print(f"v1[0] = {v1[0]}")
    print(f"v1[1] = {v1[1]}")
    print(f"3 in v1: {3 in v1}")
    print(f"5 in v1: {5 in v1}")
    
    print("Итерация по v1:")
    for i, component in enumerate(v1):
        print(f"  компонента {i}: {component}")
    
    print("\n=== МАТЕМАТИЧЕСКИЕ ОПЕРАЦИИ ===")
    print(f"Длина v1: {v1.magnitude():.2f}")
    print(f"Нормализация v1: {v1.normalize():components}")
    print(f"Скалярное произведение v1·v2: {v1.dot(v2)}")
    
    # Векторное произведение для 3D
    v3d1 = Vector(1, 0, 0)
    v3d2 = Vector(0, 1, 0)
    print(f"Векторное произведение {v3d1} × {v3d2} = {v3d1.cross(v3d2)}")
    
    angle_rad = v1.angle(v2)
    angle_deg = math.degrees(angle_rad)
    print(f"Угол между v1 и v2: {angle_rad:.3f} рад ({angle_deg:.1f}°)")
```

### Контекстные менеджеры

```python
import os
import tempfile
import json
from pathlib import Path

class FileManager:
    """Контекстный менеджер для работы с файлами"""
    
    def __init__(self, filename, mode='r', encoding='utf-8', backup=False):
        self.filename = filename
        self.mode = mode
        self.encoding = encoding
        self.backup = backup
        self.file = None
        self.backup_path = None
    
    def __enter__(self):
        """Вход в контекст - открытие файла"""
        print(f"🔓 Открываем файл: {self.filename}")
        
        # Создание резервной копии если нужно
        if self.backup and 'w' in self.mode and Path(self.filename).exists():
            self.backup_path = f"{self.filename}.backup"
            import shutil
            shutil.copy2(self.filename, self.backup_path)
            print(f"💾 Создана резервная копия: {self.backup_path}")
        
        try:
            self.file = open(self.filename, self.mode, encoding=self.encoding)
            return self.file
        except Exception as e:
            print(f"❌ Ошибка открытия файла: {e}")
            raise
    
    def __exit__(self, exc_type, exc_value, traceback):
        """Выход из контекста - закрытие файла"""
        if self.file:
            self.file.close()
            print(f"🔒 Файл закрыт: {self.filename}")
        
        # Обработка исключений
        if exc_type is not None:
            print(f"❌ Произошла ошибка: {exc_type.__name__}: {exc_value}")
            
            # Восстановление из резервной копии при ошибке записи
            if self.backup_path and 'w' in self.mode:
                import shutil
                shutil.move(self.backup_path, self.filename)
                print(f"🔄 Восстановлен из резервной копии")
            
            return False  # Не подавляем исключение
        
        # Удаление резервной копии при успешном завершении
        if self.backup_path:
            os.remove(self.backup_path)
            print(f"🗑️  Резервная копия удалена")
        
        return False

class DatabaseTransaction:
    """Имитация транзакции базы данных"""
    
    def __init__(self, connection_string):
        self.connection_string = connection_string
        self.connection = None
        self.transaction_active = False
        self.operations = []
    
    def __enter__(self):
        """Начало транзакции"""
        print(f"🔗 Подключение к БД: {self.connection_string}")
        self.connection = f"connection_to_{self.connection_string}"
        self.transaction_active = True
        print("🚀 Транзакция начата")
        return self
    
    def __exit__(self, exc_type, exc_value, traceback):
        """Завершение транзакции"""
        if exc_type is None:
            # Нет ошибок - коммитим транзакцию
            print("✅ Коммит транзакции")
            print(f"   Выполнено операций: {len(self.operations)}")
            for op in self.operations:
                print(f"   - {op}")
        else:
            # Есть ошибка - откатываем транзакцию
            print("❌ Откат транзакции из-за ошибки")
            print(f"   Отменено операций: {len(self.operations)}")
        
        self.transaction_active = False
        print("🔌 Соединение с БД закрыто")
        return False  # Не подавляем исключения
    
    def execute(self, query):
        """Выполнение SQL запроса"""
        if not self.transaction_active:
            raise RuntimeError("Транзакция не активна")
        
        self.operations.append(query)
        print(f"📝 Выполнен запрос: {query}")

class TimedOperation:
    """Контекстный менеджер для измерения времени выполнения"""
    
    def __init__(self, operation_name="Операция"):
        self.operation_name = operation_name
        self.start_time = None
        self.end_time = None
    
    def __enter__(self):
        """Начало измерения времени"""
        import time
        self.start_time = time.time()
        print(f"⏱️  Начало: {self.operation_name}")
        return self
    
    def __exit__(self, exc_type, exc_value, traceback):
        """Конец измерения времени"""
        import time
        self.end_time = time.time()
        duration = self.end_time - self.start_time
        
        if exc_type is None:
            print(f"✅ Завершено: {self.operation_name} за {duration:.3f} сек")
        else:
            print(f"❌ Прервано: {self.operation_name} через {duration:.3f} сек")
        
        return False
    
    @property
    def duration(self):
        """Получение длительности операции"""
        if self.start_time and self.end_time:
            return self.end_time - self.start_time
        return None

# Демонстрация контекстных менеджеров
if __name__ == "__main__":
    print("=== ФАЙЛОВЫЙ МЕНЕДЖЕР ===")
    
    # Создание временного файла для демонстрации
    temp_file = "demo_file.txt"
    
    # Запись в файл с автоматическим бэкапом
    try:
        with FileManager(temp_file, 'w', backup=True) as f:
            f.write("Строка 1\n")
            f.write("Строка 2\n")
            f.write("Строка 3\n")
    except Exception as e:
        print(f"Ошибка записи: {e}")
    
    # Чтение файла
    try:
        with FileManager(temp_file, 'r') as f:
            content = f.read()
            print(f"Содержимое файла:\n{content}")
    except Exception as e:
        print(f"Ошибка чтения: {e}")
    
    print("\n=== ТРАНЗАКЦИЯ БД ===")
    
    # Успешная транзакция
    try:
        with DatabaseTransaction("postgresql://localhost:5432/mydb") as db:
            db.execute("INSERT INTO users (name) VALUES ('Alice')")
            db.execute("INSERT INTO users (name) VALUES ('Bob')")
            db.execute("UPDATE users SET active = true")
    except Exception as e:
        print(f"Ошибка транзакции: {e}")
    
    print("\n--- Транзакция с ошибкой ---")
    
    # Транзакция с ошибкой
    try:
        with DatabaseTransaction("postgresql://localhost:5432/mydb") as db:
            db.execute("INSERT INTO users (name) VALUES ('Charlie')")
            db.execute("INSERT INTO users (name) VALUES ('David')")
            raise ValueError("Искусственная ошибка!")
            db.execute("This won't execute")
    except Exception as e:
        print(f"Поймана ошибка: {e}")
    
    print("\n=== ИЗМЕРЕНИЕ ВРЕМЕНИ ===")
    
    # Быстрая операция
    with TimedOperation("Быстрые вычисления") as timer:
        result = sum(range(1000))
        print(f"Результат: {result}")
    
    # Медленная операция
    with TimedOperation("Медленные вычисления") as timer:
        import time
        time.sleep(0.1)  # Имитация медленной операции
        result = sum(range(100000))
        print(f"Результат: {result}")
    
    print(f"Последняя операция заняла: {timer.duration:.3f} сек")
    
    # Операция с ошибкой
    try:
        with TimedOperation("Операция с ошибкой"):
            import time
            time.sleep(0.05)
            raise RuntimeError("Что-то пошло не так!")
    except RuntimeError as e:
        print(f"Поймана ошибка: {e}")
    
    # Очистка временного файла
    if os.path.exists(temp_file):
        os.remove(temp_file)
        print(f"\n🗑️  Временный файл {temp_file} удалён")
```

---

## 🎨 2.7 Декораторы

### Концепция декораторов

```
┌─────────────────────────────────────────────────────────┐
│                     ДЕКОРАТОРЫ                          │
│              "Обёртки для функций"                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  📦 ИСХОДНАЯ ФУНКЦИЯ                                    │
│     def my_function():                                  │
│         return "Hello"                                  │
│                                                         │
│            ▼ Применение декоратора                      │
│                                                         │
│  🎁 ОБЁРНУТАЯ ФУНКЦИЯ                                   │
│     @decorator                                          │
│     def my_function():                                  │
│         return "Hello"                                  │
│                                                         │
│     ≡                                                   │
│                                                         │
│     my_function = decorator(my_function)                │
│                                                         │
│  🔄 Декоратор может добавить:                           │
│     • Логирование                                       │
│     • Проверку прав                                     │
│     • Кеширование                                       │
│     • Измерение времени                                 │
│     • Валидацию аргументов                              │
└─────────────────────────────────────────────────────────┘
```

### Создание декораторов

```python
import functools
import time
import threading
from typing import Any, Callable, Dict

# === БАЗОВЫЕ ДЕКОРАТОРЫ ===

def timer(func):
    """Декоратор для измерения времени выполнения функции"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        try:
            result = func(*args, **kwargs)
            return result
        finally:
            end_time = time.time()
            duration = end_time - start_time
            print(f"⏱️  {func.__name__} выполнялась {duration:.4f} сек")
    return wrapper

def logger(func):
    """Декоратор для логирования вызовов функций"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        args_str = ", ".join(repr(arg) for arg in args)
        kwargs_str = ", ".join(f"{k}={repr(v)}" for k, v in kwargs.items())
        all_args = ", ".join(filter(None, [args_str, kwargs_str]))
        
        print(f"📝 Вызов {func.__name__}({all_args})")
        
        try:
            result = func(*args, **kwargs)
            print(f"✅ {func.__name__} вернула: {repr(result)}")
            return result
        except Exception as e:
            print(f"❌ {func.__name__} вызвала исключение: {repr(e)}")
            raise
    return wrapper

def retry(max_attempts=3, delay=1):
    """Декоратор для повторных попыток выполнения функции"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            
            for attempt in range(max_attempts):
                try:
                    result = func(*args, **kwargs)
                    if attempt > 0:
                        print(f"✅ {func.__name__} успешно выполнена с {attempt + 1} попытки")
                    return result
                except Exception as e:
                    last_exception = e
                    if attempt < max_attempts - 1:
                        print(f"❌ Попытка {attempt + 1} неудачна: {e}. Повтор через {delay} сек...")
                        time.sleep(delay)
                    else:
                        print(f"❌ Все {max_attempts} попыток исчерпаны")
            
            raise last_exception
        return wrapper
    return decorator

def validate_types(**expected_types):
    """Декоратор для валидации типов аргументов"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # Получаем имена аргументов из сигнатуры функции
            import inspect
            sig = inspect.signature(func)
            bound_args = sig.bind(*args, **kwargs)
            bound_args.apply_defaults()
            
            # Проверяем типы
            for param_name, expected_type in expected_types.items():
                if param_name in bound_args.arguments:
                    value = bound_args.arguments[param_name]
                    if not isinstance(value, expected_type):
                        raise TypeError(
                            f"Аргумент '{param_name}' должен быть типа {expected_type.__name__}, "
                            f"получен {type(value).__name__}"
                        )
            
            return func(*args, **kwargs)
        return wrapper
    return decorator

def cache(maxsize=128):
    """Декоратор для кеширования результатов функции"""
    def decorator(func):
        cache_dict = {}
        access_order = []
        
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # Создаем ключ для кеша
            key = (args, tuple(sorted(kwargs.items())))
            
            # Проверяем наличие в кеше
            if key in cache_dict:
                print(f"💾 Кеш попадание для {func.__name__}")
                # Обновляем порядок доступа
                access_order.remove(key)
                access_order.append(key)
                return cache_dict[key]
            
            # Вычисляем результат
            print(f"🔄 Вычисление {func.__name__}")
            result = func(*args, **kwargs)
            
            # Добавляем в кеш
            cache_dict[key] = result
            access_order.append(key)
            
            # Удаляем старые записи если превышен размер
            while len(cache_dict) > maxsize:
                oldest_key = access_order.pop(0)
                del cache_dict[oldest_key]
            
            return result
        
        # Добавляем методы для управления кешем
        wrapper.cache_clear = lambda: (cache_dict.clear(), access_order.clear())
        wrapper.cache_info = lambda: {
            'size': len(cache_dict),
            'maxsize': maxsize,
            'keys': list(cache_dict.keys())
        }
        
        return wrapper
    return decorator

# === ДЕКОРАТОРЫ КЛАССОВ ===

def singleton(cls):
    """Декоратор для реализации паттерна Singleton"""
    instances = {}
    lock = threading.Lock()
    
    @functools.wraps(cls)
    def get_instance(*args, **kwargs):
        if cls not in instances:
            with lock:
                if cls not in instances:
                    instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    
    return get_instance

def add_methods(**methods):
    """Декоратор для добавления методов в класс"""
    def decorator(cls):
        for method_name, method_func in methods.items():
            setattr(cls, method_name, method_func)
        return cls
    return decorator

# === ПРОДВИНУТЫЕ ДЕКОРАТОРЫ ===

class RateLimiter:
    """Декоратор для ограничения частоты вызовов"""
    
    def __init__(self, max_calls=5, time_window=60):
        self.max_calls = max_calls
        self.time_window = time_window
        self.calls = []
        self.lock = threading.Lock()
    
    def __call__(self, func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            with self.lock:
                current_time = time.time()
                
                # Удаляем старые вызовы
                self.calls = [call_time for call_time in self.calls 
                             if current_time - call_time < self.time_window]
                
                # Проверяем лимит
                if len(self.calls) >= self.max_calls:
                    raise RuntimeError(
                        f"Превышен лимит вызовов: {self.max_calls} вызовов "
                        f"за {self.time_window} секунд"
                    )
                
                # Добавляем текущий вызов
                self.calls.append(current_time)
            
            return func(*args, **kwargs)
        return wrapper

# === ПРИМЕРЫ ИСПОЛЬЗОВАНИЯ ===

# Применение нескольких декораторов
@timer
@logger
@retry(max_attempts=2, delay=0.5)
@validate_types(x=int, y=int)
def divide(x, y):
    """Деление с обработкой ошибок"""
    if y == 0:
        raise ValueError("Деление на ноль!")
    return x / y

@cache(maxsize=3)
@timer
def fibonacci(n):
    """Вычисление числа Фибоначчи с кешированием"""
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

@singleton
class DatabaseConnection:
    """Одиночное подключение к базе данных"""
    
    def __init__(self):
        print("🔗 Создание подключения к БД...")
        self.connection_id = id(self)
    
    def query(self, sql):
        return f"Результат запроса '{sql}' через соединение {self.connection_id}"

@add_methods(
    greet=lambda self: f"Привет, я {self.name}!",
    age_in_days=lambda self: self.age * 365
)
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

@RateLimiter(max_calls=3, time_window=5)
def api_call(endpoint):
    """Имитация API вызова с ограничением частоты"""
    print(f"🌐 Вызов API: {endpoint}")
    return f"Данные от {endpoint}"

# Демонстрация декораторов
if __name__ == "__main__":
    print("=== БАЗОВЫЕ ДЕКОРАТОРЫ ===")
    
    # Успешное деление
    result = divide(10, 2)
    print(f"Результат: {result}\n")
    
    # Деление с ошибкой и повтором
    try:
        result = divide(10, 0)
    except ValueError as e:
        print(f"Финальная ошибка: {e}\n")
    
    # Валидация типов
    try:
        divide("10", 2)  # Неверный тип
    except TypeError as e:
        print(f"Ошибка типа: {e}\n")
    
    print("=== КЕШИРОВАНИЕ ===")
    
    # Демонстрация кеширования
    print("Вычисление fibonacci(5):")
    result1 = fibonacci(5)
    print(f"Результат: {result1}")
    
    print("\nПовторное вычисление fibonacci(5):")
    result2 = fibonacci(5)
    print(f"Результат: {result2}")
    
    print(f"\nИнформация о кеше: {fibonacci.cache_info()}")
    
    print("\n=== SINGLETON ===")
    
    # Демонстрация singleton
    db1 = DatabaseConnection()
    db2 = DatabaseConnection()
    
    print(f"db1 is db2: {db1 is db2}")
    print(db1.query("SELECT * FROM users"))
    
    print("\n=== ДОБАВЛЕНИЕ МЕТОДОВ ===")
    
    # Демонстрация добавления методов
    person = Person("Анна", 25)
    print(person.greet())
    print(f"Возраст в днях: {person.age_in_days()}")
    
    print("\n=== ОГРАНИЧЕНИЕ ЧАСТОТЫ ВЫЗОВОВ ===")
    
    # Демонстрация rate limiting
    try:
        for i in range(5):
            api_call(f"/users/{i}")
            time.sleep(0.1)
    except RuntimeError as e:
        print(f"Ошибка лимита: {e}")
    
    print("\nОжидание сброса лимита...")
    time.sleep(1)
    api_call("/reset")  # Должно работать
```

---

## 📦 2.8 Модули и пакеты

### Структура модулей и пакетов

```
┌─────────────────────────────────────────────────────────┐
│                СТРУКТУРА ПРОЕКТА                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  my_project/                                            │
│  ├── main.py                    # Главный файл          │
│  ├── config.py                  # Конфигурация         │
│  ├── utils/                     # Пакет утилит         │
│  │   ├── __init__.py           # Инициализация пакета  │
│  │   ├── helpers.py            # Вспомогательные ф-ции │
│  │   └── validators.py         # Валидаторы            │
│  ├── models/                    # Пакет моделей        │
│  │   ├── __init__.py                                   │
│  │   ├── user.py               # Модель пользователя   │
│  │   └── product.py            # Модель продукта       │
│  ├── services/                  # Пакет сервисов       │
│  │   ├── __init__.py                                   │
│  │   ├── auth_service.py       # Сервис аутентификации │
│  │   └── data_service.py       # Сервис данных         │
│  └── tests/                     # Пакет тестов         │
│      ├── __init__.py                                   │
│      ├── test_models.py        # Тесты моделей         │
│      └── test_services.py      # Тесты сервисов        │
└─────────────────────────────────────────────────────────┘
```

### Создание модулей

```python
# === utils/helpers.py ===
"""
Модуль вспомогательных функций
"""

import re
import hashlib
import secrets
from typing import List, Dict, Any, Optional
from datetime import datetime, timedelta

__version__ = "1.0.0"
__author__ = "Python Developer"

# Константы модуля
DEFAULT_PASSWORD_LENGTH = 12
EMAIL_PATTERN = re.compile(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,})

def generate_password(length: int = DEFAULT_PASSWORD_LENGTH, 
                     include_symbols: bool = True) -> str:
    """
    Генерация безопасного пароля
    
    Args:
        length: Длина пароля
        include_symbols: Включать ли специальные символы
    
    Returns:
        str: Сгенерированный пароль
    
    Examples:
        >>> password = generate_password(8, False)
        >>> len(password)
        8
        >>> generate_password(0)
        Traceback (most recent call last):
        ...
        ValueError: Длина пароля должна быть больше 0
    """
    if length <= 0:
        raise ValueError("Длина пароля должна быть больше 0")
    
    # Наборы символов
    lowercase = "abcdefghijklmnopqrstuvwxyz"
    uppercase = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    digits = "0123456789"
    symbols = "!@#$%^&*()_+-=[]{}|;:,.<>?"
    
    # Формируем алфавит
    alphabet = lowercase + uppercase + digits
    if include_symbols:
        alphabet += symbols
    
    # Генерируем пароль
    password = ''.join(secrets.choice(alphabet) for _ in range(length))
    return password

def hash_password(password: str, salt: Optional[str] = None) -> tuple[str, str]:
    """
    Хеширование пароля с солью
    
    Args:
        password: Пароль для хеширования
        salt: Соль (если не указана, генерируется автоматически)
    
    Returns:
        tuple: (hash, salt)
    """
    if salt is None:
        salt = secrets.token_hex(16)
    
    # Создаем hash с солью
    password_hash = hashlib.pbkdf2_hmac(
        'sha256',
        password.encode('utf-8'),
        salt.encode('utf-8'),
        100000  # iterations
    )
    
    return password_hash.hex(), salt

def verify_password(password: str, password_hash: str, salt: str) -> bool:
    """
    Проверка пароля
    
    Args:
        password: Введенный пароль
        password_hash: Сохраненный хеш
        salt: Соль
    
    Returns:
        bool: True если пароль верный
    """
    computed_hash, _ = hash_password(password, salt)
    return computed_hash == password_hash

def format_file_size(size_bytes: int) -> str:
    """
    Форматирование размера файла в человекочитаемый вид
    
    Args:
        size_bytes: Размер в байтах
    
    Returns:
        str: Отформатированный размер
    
    Examples:
        >>> format_file_size(1024)
        '1.0 KB'
        >>> format_file_size(1048576)
        '1.0 MB'
    """
    if size_bytes == 0:
        return "0 B"
    
    size_names = ["B", "KB", "MB", "GB", "TB"]
    i = 0
    size = float(size_bytes)
    
    while size >= 1024.0 and i < len(size_names) - 1:
        size /= 1024.0
        i += 1
    
    return f"{size:.1f} {size_names[i]}"

def chunk_list(lst: List[Any], chunk_size: int) -> List[List[Any]]:
    """
    Разбиение списка на части заданного размера
    
    Args:
        lst: Исходный список
        chunk_size: Размер части
    
    Returns:
        List[List]: Список частей
    
    Examples:
        >>> chunk_list([1, 2, 3, 4, 5], 2)
        [[1, 2], [3, 4], [5]]
    """
    if chunk_size <= 0:
        raise ValueError("Размер части должен быть больше 0")
    
    return [lst[i:i + chunk_size] for i in range(0, len(lst), chunk_size)]

def deep_merge_dicts(dict1: Dict[str, Any], dict2: Dict[str, Any]) -> Dict[str, Any]:
    """
    Глубокое слияние словарей
    
    Args:
        dict1: Первый словарь
        dict2: Второй словарь
    
    Returns:
        Dict: Объединенный словарь
    """
    result = dict1.copy()
    
    for key, value in dict2.items():
        if (key in result and 
            isinstance(result[key], dict) and 
            isinstance(value, dict)):
            result[key] = deep_merge_dicts(result[key], value)
        else:
            result[key] = value
    
    return result

class Timer:
    """Контекстный менеджер для измерения времени"""
    
    def __init__(self, description: str = "Операция"):
        self.description = description
        self.start_time = None
        self.duration = None
    
    def __enter__(self):
        self.start_time = datetime.now()
        return self
    
    def __exit__(self, exc_type, exc_value, traceback):
        end_time = datetime.now()
        self.duration = end_time - self.start_time
        print(f"{self.description} заняла: {self.duration.total_seconds():.3f} сек")

# === utils/validators.py ===
"""
Модуль валидаторов данных
"""

import re
from typing import Any, List, Callable
from datetime import datetime

def is_valid_email(email: str) -> bool:
    """Проверка корректности email адреса"""
    if not isinstance(email, str):
        return False
    return bool(EMAIL_PATTERN.match(email))

def is_valid_phone(phone: str, country_code: str = "RU") -> bool:
    """
    Проверка корректности номера телефона
    
    Args:
        phone: Номер телефона
        country_code: Код страны
    
    Returns:
        bool: True если номер корректный
    """
    # Упрощенная проверка для демонстрации
    patterns = {
        "RU": r'^(\+7|8)[\s\-]?\(?\d{3}\)?[\s\-]?\d{3}[\s\-]?\d{2}[\s\-]?\d{2},
        "US": r'^(\+1|1)?[\s\-]?\(?\d{3}\)?[\s\-]?\d{3}[\s\-]?\d{4}
    }
    
    pattern = patterns.get(country_code)
    if not pattern:
        return False
    
    return bool(re.match(pattern, phone))

def validate_age(age: int, min_age: int = 0, max_age: int = 150) -> bool:
    """Валидация возраста"""
    return isinstance(age, int) and min_age <= age <= max_age

class ValidationError(Exception):
    """Исключение для ошибок валидации"""
    pass

class Validator:
    """Класс для создания цепочек валидации"""
    
    def __init__(self, value: Any):
        self.value = value
        self.errors = []
    
    def required(self, message: str = "Поле обязательно для заполнения"):
        """Проверка на обязательность"""
        if self.value is None or self.value == "":
            self.errors.append(message)
        return self
    
    def min_length(self, length: int, message: str = None):
        """Проверка минимальной длины"""
        if message is None:
            message = f"Минимальная длина: {length}"
        
        if isinstance(self.value, str) and len(self.value) < length:
            self.errors.append(message)
        return self
    
    def max_length(self, length: int, message: str = None):
        """Проверка максимальной длины"""
        if message is None:
            message = f"Максимальная длина: {length}"
        
        if isinstance(self.value, str) and len(self.value) > length:
            self.errors.append(message)
        return self
    
    def email(self, message: str = "Некорректный email адрес"):
        """Проверка email"""
        if self.value and not is_valid_email(self.value):
            self.errors.append(message)
        return self
    
    def custom(self, validator_func: Callable[[Any], bool], 
              message: str = "Значение не прошло валидацию"):
        """Пользовательская валидация"""
        if self.value and not validator_func(self.value):
            self.errors.append(message)
        return self
    
    def validate(self) -> List[str]:
        """Возврат списка ошибок"""
        return self.errors
    
    def is_valid(self) -> bool:
        """Проверка валидности"""
        return len(self.errors) == 0
    
    def raise_if_invalid(self):
        """Вызов исключения при наличии ошибок"""
        if not self.is_valid():
            raise ValidationError("; ".join(self.errors))

# === utils/__init__.py ===
"""
Пакет утилит

Предоставляет вспомогательные функции и классы для работы с:
- Паролями и хешированием
- Валидацией данных
- Форматированием
- Измерением времени
"""

# Импорты для удобного доступа
from .helpers import (
    generate_password,
    hash_password,
    verify_password,
    format_file_size,
    chunk_list,
    deep_merge_dicts,
    Timer
)

from .validators import (
    is_valid_email,
    is_valid_phone,
    validate_age,
    ValidationError,
    Validator
)

# Метаданные пакета
__version__ = "1.0.0"
__author__ = "Python Developer"

# Определяем что экспортируется при from utils import *
__all__ = [
    # Helpers
    'generate_password',
    'hash_password', 
    'verify_password',
    'format_file_size',
    'chunk_list',
    'deep_merge_dicts',
    'Timer',
    
    # Validators
    'is_valid_email',
    'is_valid_phone',
    'validate_age',
    'ValidationError',
    'Validator'
]

def get_package_info():
    """Информация о пакете"""
    return {
        'name': 'utils',
        'version': __version__,
        'author': __author__,
        'modules': ['helpers', 'validators'],
        'functions': len(__all__)
    }

# === models/user.py ===
"""
Модель пользователя
"""

from datetime import datetime
from typing import Optional, List, Dict, Any
from utils import hash_password, verify_password, ValidationError, Validator

class User:
    """Модель пользователя системы"""
    
    # Классовые атрибуты
    _users_count = 0
    _active_users = set()
    
    def __init__(self, username: str, email: str, password: str, 
                 first_name: str = "", last_name: str = ""):
        """
        Инициализация пользователя
        
        Args:
            username: Имя пользователя
            email: Email адрес
            password: Пароль
            first_name: Имя
            last_name: Фамилия
        """
        # Валидация входных данных
        self._validate_user_data(username, email, password)
        
        # Приватные атрибуты
        self._id = User._users_count + 1
        self._created_at = datetime.now()
        self._password_hash = None
        self._password_salt = None
        self._is_active = True
        self._last_login = None
        
        # Публичные атрибуты
        self.username = username
        self.email = email
        self.first_name = first_name
        self.last_name = last_name
        self.roles = []
        self.metadata = {}
        
        # Устанавливаем пароль
        self.set_password(password)
        
        # Обновляем счетчики
        User._users_count += 1
        User._active_users.add(self._id)
    
    @staticmethod
    def _validate_user_data(username: str, email: str, password: str):
        """Валидация данных пользователя"""
        # Валидация username
        Validator(username)\
            .required("Имя пользователя обязательно")\
            .min_length(3, "Имя пользователя должно быть не менее 3 символов")\
            .max_length(50, "Имя пользователя должно быть не более 50 символов")\
            .raise_if_invalid()
        
        # Валидация email
        Validator(email)\
            .required("Email обязателен")\
            .email()\
            .raise_if_invalid()
        
        # Валидация password
        Validator(password)\
            .required("Пароль обязателен")\
            .min_length(6, "Пароль должен быть не менее 6 символов")\
            .raise_if_invalid()
    
    @property
    def id(self) -> int:
        """ID пользователя (только для чтения)"""
        return self._id
    
    @property
    def created_at(self) -> datetime:
        """Дата создания (только для чтения)"""
        return self._created_at
    
    @property
    def is_active(self) -> bool:
        """Статус активности"""
        return self._is_active
    
    @property
    def full_name(self) -> str:
        """Полное имя пользователя"""
        return f"{self.first_name} {self.last_name}".strip()
    
    @property
    def last_login(self) -> Optional[datetime]:
        """Время последнего входа"""
        return self._last_login
    
    def set_password(self, password: str):
        """Установка нового пароля"""
        Validator(password)\
            .required("Пароль обязателен")\
            .min_length(6, "Пароль должен быть не менее 6 символов")\
            .raise_if_invalid()
        
        self._password_hash, self._password_salt = hash_password(password)
    
    def check_password(self, password: str) -> bool:
        """Проверка пароля"""
        if not self._password_hash or not self._password_salt:
            return False
        return verify_password(password, self._password_hash, self._password_salt)
    
    def login(self, password: str) -> bool:
        """Вход в систему"""
        if not self._is_active:
            raise RuntimeError("Пользователь деактивирован")
        
        if self.check_password(password):
            self._last_login = datetime.now()
            return True
        return False
    
    def deactivate(self):
        """Деактивация пользователя"""
        self._is_active = False
        User._active_users.discard(self._id)
    
    def activate(self):
        """Активация пользователя"""
        self._is_active = True
        User._active_users.add(self._id)
    
    def add_role(self, role: str):
        """Добавление роли"""
        if role not in self.roles:
            self.roles.append(role)
    
    def remove_role(self, role: str):
        """Удаление роли"""
        if role in self.roles:
            self.roles.remove(role)
    
    def has_role(self, role: str) -> bool:
        """Проверка наличия роли"""
        return role in self.roles
    
    def update_metadata(self, **kwargs):
        """Обновление метаданных"""
        self.metadata.update(kwargs)
    
    def to_dict(self, include_sensitive: bool = False) -> Dict[str, Any]:
        """Преобразование в словарь"""
        data = {
            'id': self._id,
            'username': self.username,
            'email': self.email,
            'first_name': self.first_name,
            'last_name': self.last_name,
            'full_name': self.full_name,
            'is_active': self._is_active,
            'created_at': self._created_at.isoformat(),
            'last_login': self._last_login.isoformat() if self._last_login else None,
            'roles': self.roles.copy(),
            'metadata': self.metadata.copy()
        }
        
        if include_sensitive:
            data.update({
                'password_hash': self._password_hash,
                'password_salt': self._password_salt
            })
        
        return data
    
    @classmethod
    def from_dict(cls, data: Dict[str, Any]) -> 'User':
        """Создание пользователя из словаря"""
        # Создаем временный пароль (будет заменен если есть хеш)
        user = cls(
            username=data['username'],
            email=data['email'],
            password='temporary_password',
            first_name=data.get('first_name', ''),
            last_name=data.get('last_name', '')
        )
        
        # Восстанавливаем сохраненные данные
        if 'password_hash' in data and 'password_salt' in data:
            user._password_hash = data['password_hash']
            user._password_salt = data['password_salt']
        
        user.roles = data.get('roles', [])
        user.metadata = data.get('metadata', {})
        
        if 'is_active' in data:
            if data['is_active']:
                user.activate()
            else:
                user.deactivate()
        
        return user
    
    @classmethod
    def get_user_count(cls) -> int:
        """Получение общего количества пользователей"""
        return cls._users_count
    
    @classmethod
    def get_active_user_count(cls) -> int:
        """Получение количества активных пользователей"""
        return len(cls._active_users)
    
    def __str__(self):
        status = "активен" if self._is_active else "неактивен"
        return f"User({self.username}, {self.email}, {status})"
    
    def __repr__(self):
        return (f"User(id={self._id}, username='{self.username}', "
                f"email='{self.email}', active={self._is_active})")
    
    def __eq__(self, other):
        if isinstance(other, User):
            return self._id == other._id
        return False
    
    def __hash__(self):
        return hash(self._id)

# === Демонстрация использования модулей ===

if __name__ == "__main__":
    print("=== ДЕМОНСТРАЦИЯ МОДУЛЕЙ И ПАКЕТОВ ===")
    
    # Импорт из пакета utils
    from utils import generate_password, Timer, Validator, ValidationError
    from models.user import User
    
    print("\n=== ИНФОРМАЦИЯ О ПАКЕТЕ ===")
    import utils
    print(utils.get_package_info())
    
    print("\n=== ИСПОЛЬЗОВАНИЕ УТИЛИТ ===")
    
    # Генерация пароля
    password = generate_password(12, True)
    print(f"Сгенерированный пароль: {password}")
    
    # Валидация данных
    print("\n--- Валидация email ---")
    emails = ["test@example.com", "invalid-email", "user@domain.org"]
    for email in emails:
        try:
            Validator(email).required().email().raise_if_invalid()
            print(f"✅ {email} - корректный")
        except ValidationError as e:
            print(f"❌ {email} - {e}")
    
    print("\n=== РАБОТА С ПОЛЬЗОВАТЕЛЯМИ ===")
    
    # Создание пользователей
    try:
        user1 = User("alice", "alice@example.com", "secure123", "Алиса", "Иванова")
        user2 = User("bob", "bob@example.com", "password456", "Боб", "Петров")
        
        print(f"Создан пользователь: {user1}")
        print(f"Создан пользователь: {user2}")
        
        # Добавление ролей
        user1.add_role("admin")
        user1.add_role("moderator")
        user2.add_role("user")
        
        print(f"\nРоли {user1.username}: {user1.roles}")
        print(f"Роли {user2.username}: {user2.roles}")
        
        # Вход в систему
        print(f"\n--- Попытка входа для {user1.username} ---")
        if user1.login("secure123"):
            print("✅ Успешный вход")
        else:
            print("❌ Неверный пароль")
        
        # Неверный пароль
        if user1.login("wrong_password"):
            print("✅ Успешный вход")
        else:
            print("❌ Неверный пароль")
        
        # Информация о пользователях
        print(f"\nВсего пользователей: {User.get_user_count()}")
        print(f"Активных пользователей: {User.get_active_user_count()}")
        
        # Деактивация пользователя
        user2.deactivate()
        print(f"\nПосле деактивации {user2.username}:")
        print(f"Активных пользователей: {User.get_active_user_count()}")
        
        # Сериализация в словарь
        print(f"\n--- Данные пользователя {user1.username} ---")
        user_data = user1.to_dict()
        for key, value in user_data.items():
            print(f"{key}: {value}")
        
    except ValidationError as e:
        print(f"Ошибка валидации: {e}")
    
    print("\n=== ИЗМЕРЕНИЕ ВРЕМЕНИ ===")
    
    # Использование Timer из utils
    with Timer("Создание 1000 пользователей"):
        users = []
        for i in range(1000):
            try:
                user = User(f"user{i}", f"user{i}@test.com", "password123")
                users.append(user)
            except ValidationError:
                pass  # Игнорируем дубликаты
    
    print(f"Создано пользователей: {len(users)}")
    print(f"Общий счетчик пользователей: {User.get_user_count()}")
```

---

## 🎯 Итоговые результаты Блока 2

### ✅ Чему вы научились:

```
┌─────────────────────────────────────────────────────────┐
│                   ДОСТИЖЕНИЯ                            │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  🏗️  СОЗДАНИЕ КЛАССОВ                                   │
│     ✓ Определение атрибутов и методов                   │
│     ✓ Конструкторы и деструкторы                        │
│     ✓ Методы класса и статические методы                │
│                                                         │
│  🔒 ИНКАПСУЛЯЦИЯ                                        │
│     ✓ Приватные и защищённые атрибуты                   │
│     ✓ Свойства (properties)                             │
│     ✓ Геттеры и сеттеры                                 │
│                                                         │
│  🧬 НАСЛЕДОВАНИЕ                                        │
│     ✓ Базовые и производные классы                      │
│     ✓ Множественное наследование                        │
│     ✓ MRO (Method Resolution Order)                     │
│                                                         │
│  🎭 ПОЛИМОРФИЗМ                                         │
│     ✓ Переопределение методов                           │
│     ✓ Абстрактные классы                                │
│     ✓ Протоколы и утиная типизация                      │
│                                                         │
│  ⚡ СПЕЦИАЛЬНЫЕ МЕТОДЫ                                  │
│     ✓ Магические методы (__str__, __add__, etc.)        │
│     ✓ Контекстные менеджеры                             │
│     ✓ Итераторы и генераторы                           │
│                                                         │
│  🎨 ДЕКОРАТОРЫ                                          │
│     ✓ Функциональные декораторы                         │
│     ✓ Декораторы классов                                │
│     ✓ Декораторы с параметрами                          │
│                                                         │
│  📦 МОДУЛИ И ПАКЕТЫ                                     │
│     ✓ Создание модулей                                  │
│     ✓ Организация пакетов                               │
│     ✓ Управление импортами                              │
└─────────────────────────────────────────────────────────┘
```

### 🛠 Практические проекты для закрепления:

1. **Система управления библиотекой** - Создайте иерархию классов для книг, авторов, читателей с полной функциональностью
2. **Игра "Крестики-нолики" с ИИ** - Используйте ООП для создания игрового движка с разными типами игроков
3. **Банковская система** - Разработайте классы для различных типов счетов с инкапсуляцией и безопасностью
4. **Файловый менеджер** - Создайте систему для работы с файлами используя контекстные менеджеры и декораторы

### 📚 Рекомендуемые ресурсы для углубления:

- **Книги**: "Effective Python" Brett Slatkin, "Architecture Patterns with Python"
- **Практика**: Участие в open-source проектах на GitHub
- **Задачи**: LeetCode OOP problems, Design Patterns implementations

---

**🎓 Поздравляем! Вы освоили объектно-ориентированное программирование в Python!**

*Теперь вы готовы создавать сложные, масштабируемые и поддерживаемые приложения используя принципы ООП.*

---

**📅 Следующий блок:** Работа с данными и файлами  
**🎯 Подготовка:** Повторите основы работы с файлами и изучите JSON формат