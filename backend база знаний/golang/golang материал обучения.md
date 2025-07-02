# Полный курс изучения Golang
## От новичка до эксперта в 25 главах

---

## 🎯 Что вас ждет

Этот курс проведет вас через полное изучение языка Go - от первой программы до создания production-ready сервисов. Каждая глава содержит теорию, практические примеры и задания для закрепления материала.

```
Новичок → Уверенный разработчик → Backend эксперт
   |              |                    |
3-4 недели    8-10 недель         18-24 недели
```

---

# БЛОК 1: ОСНОВЫ ЯЗЫКА
*Цель: Освоить синтаксис Go и базовые концепции программирования*

---

## Глава 1: Знакомство с Go и первые шаги

### Что такое Go и почему он важен

Go (или Golang) - это язык программирования, созданный в Google для решения современных задач разработки. Представьте Go как швейцарский нож программирования: простой, надежный и эффективный.

```
Преимущества Go:
┌─────────────────┐
│ Простота        │ ──┐
├─────────────────┤   │
│ Скорость        │   ├── Почему Go популярен
├─────────────────┤   │
│ Конкурентность  │   │
├─────────────────┤   │
│ Стабильность    │ ──┘
└─────────────────┘
```

### Установка и настройка

1. **Скачайте Go** с официального сайта golang.org
2. **Установите** следуя инструкциям для вашей ОС
3. **Проверьте установку**: откройте терминал и введите `go version`

### Ваша первая программа

```go
package main

import "fmt"

func main() {
    fmt.Println("Привет, мир Go!")
}
```

**Что происходит:**
- `package main` - говорит, что это главный пакет
- `import "fmt"` - подключаем библиотеку для вывода
- `func main()` - главная функция программы
- `fmt.Println()` - выводит текст на экран

---

## Глава 2: Переменные и типы данных

### Как компьютер хранит информацию

Представьте переменные как коробки с ярлыками, где вы храните разные типы вещей:

```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│ name: "Вася"│  │ age: 25     │  │ isActive:   │
│ (строка)    │  │ (число)     │  │ true        │
└─────────────┘  └─────────────┘  │ (булев)     │
                                  └─────────────┘
```

### Основные типы данных в Go

```go
// Объявление переменных
var name string = "Алексей"
var age int = 30
var isStudent bool = true
var height float64 = 175.5

// Короткая запись (Go сам определит тип)
city := "Москва"
temperature := -5.2
```

**Типы данных:**
- `string` - текст ("Привет", "Go")
- `int` - целые числа (1, 42, -10)
- `float64` - числа с дробной частью (3.14, -2.5)
- `bool` - истина или ложь (true, false)

---

## Глава 3: Управление программой

### Условия: если-то-иначе

Программа должна уметь принимать решения, как человек:

```
Если температура < 0
    ├── ДА → "Одевайся теплее"
    └── НЕТ → "Можно легко одеться"
```

```go
temperature := -5

if temperature < 0 {
    fmt.Println("На улице мороз!")
} else if temperature > 25 {
    fmt.Println("Жарко!")
} else {
    fmt.Println("Нормальная погода")
}
```

### Циклы: повторение действий

Когда нужно сделать что-то много раз:

```
Цикл for:
┌─────────────┐
│ Начало: i=1 │
├─────────────┤
│ Условие:    │ ──┐
│ i <= 5      │   │ Да
├─────────────┤   ▼
│ Действие:   │ ┌─────────────┐
│ печать i    │ │ Выполнить   │
├─────────────┤ │ код         │
│ Шаг: i++    │ └─────────────┘
└─────────────┘       │
      ▲               │
      └───────────────┘
```

```go
// Печатаем числа от 1 до 5
for i := 1; i <= 5; i++ {
    fmt.Println("Число:", i)
}
```

---

## Глава 4: Функции - строительные блоки

### Что такое функция

Функция - это как рецепт приготовления блюда. Вы даете ингредиенты (параметры), следуете рецепту (код функции), получаете готовое блюдо (результат).

```
Функция сложения:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ Вход: a, b  │ ──▶│ Обработка:  │ ──▶│ Выход:      │
│ (числа)     │    │ a + b       │    │ сумма       │
└─────────────┘    └─────────────┘    └─────────────┘
```

```go
// Функция для сложения двух чисел
func add(a int, b int) int {
    result := a + b
    return result
}

// Использование функции
sum := add(5, 3) // sum = 8
```

### Функции с несколькими возвращаемыми значениями

В Go функция может возвращать несколько значений одновременно:

```go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("деление на ноль")
    }
    return a / b, nil
}

result, err := divide(10, 2)
if err != nil {
    fmt.Println("Ошибка:", err)
} else {
    fmt.Println("Результат:", result)
}
```

---

## Глава 5: Коллекции данных

### Массивы и слайсы

Представьте массив как ряд пронумерованных ящиков:

```
Массив чисел:
┌───┬───┬───┬───┬───┐
│ 1 │ 5 │ 3 │ 9 │ 2 │
└───┴───┴───┴───┴───┘
  0   1   2   3   4   ← индексы
```

```go
// Массив фиксированного размера
var numbers [5]int = [5]int{1, 5, 3, 9, 2}

// Слайс (динамический массив) - используется чаще
fruits := []string{"яблоко", "банан", "апельсин"}

// Добавление элементов в слайс
fruits = append(fruits, "груша")
```

### Maps (словари)

Map - это как телефонная книга, где по имени можно найти номер:

```
Map (словарь):
┌─────────────┬─────────────┐
│ Ключ        │ Значение    │
├─────────────┼─────────────┤
│ "Алексей"   │ 25          │
│ "Мария"     │ 30          │
│ "Иван"      │ 22          │
└─────────────┴─────────────┘
```

```go
// Создание map
ages := map[string]int{
    "Алексей": 25,
    "Мария":   30,
    "Иван":    22,
}

// Получение значения
age := ages["Мария"] // 30

// Добавление нового элемента
ages["Ольга"] = 28
```

---

## Глава 6: Работа с текстом

### Строки в Go

Строки в Go - это последовательности символов в кодировке UTF-8:

```
Строка "Привет":
┌─┬─┬─┬─┬─┬─┐
│П│р│и│в│е│т│
└─┴─┴─┴─┴─┴─┘
 0 1 2 3 4 5  ← позиции байт
```

```go
text := "Изучаем Go"

// Длина строки
length := len(text)

// Объединение строк
greeting := "Привет, " + "мир!"

// Проверка содержания
contains := strings.Contains(text, "Go") // true
```

### Полезные операции со строками

```go
import "strings"

text := "  Golang Programming  "

// Убрать пробелы
clean := strings.TrimSpace(text) // "Golang Programming"

// Разделить строку
words := strings.Split(clean, " ") // ["Golang", "Programming"]

// Привести к нижнему регистру
lower := strings.ToLower(clean) // "golang programming"
```

---

# БЛОК 2: ПРОДВИНУТЫЕ ВОЗМОЖНОСТИ
*Цель: Изучить уникальные особенности Go*

---

## Глава 7: Структуры - создаем свои типы

### Что такое структура

Структура - это способ объединить связанные данные в одну сущность, как паспорт объединяет информацию о человеке:

```
Структура Person:
┌─────────────────────┐
│ Name: "Алексей"     │
│ Age: 25             │
│ Email: "a@mail.ru"  │
│ IsActive: true      │
└─────────────────────┘
```

```go
// Определение структуры
type Person struct {
    Name     string
    Age      int
    Email    string
    IsActive bool
}

// Создание экземпляра
person := Person{
    Name:     "Алексей",
    Age:      25,
    Email:    "alex@example.com",
    IsActive: true,
}

fmt.Println(person.Name) // "Алексей"
```

### Методы структур

Методы позволяют структурам "уметь что-то делать":

```go
// Метод для структуры Person
func (p Person) Greet() string {
    return "Привет, меня зовут " + p.Name
}

// Метод, изменяющий структуру
func (p *Person) Birthday() {
    p.Age++
}

// Использование
person := Person{Name: "Анна", Age: 25}
fmt.Println(person.Greet()) // "Привет, меня зовут Анна"
person.Birthday()
fmt.Println(person.Age) // 26
```

---

## Глава 8: Указатели - адреса в памяти

### Зачем нужны указатели

Указатель - это как адрес дома. Вместо того чтобы переносить весь дом, мы можем просто передать адрес:

```
Переменная в памяти:
┌─────────────────┐
│ Адрес: 0x1040a124 │
│ Значение: 42      │
└─────────────────┘
       ▲
       │
┌─────────────────┐
│ Указатель:      │
│ 0x1040a124      │
└─────────────────┘
```

```go
number := 42

// Создание указателя
pointer := &number // pointer содержит адрес переменной number

// Получение значения по указателю
value := *pointer // value = 42

// Изменение значения через указатель
*pointer = 100
fmt.Println(number) // 100
```

### Когда использовать указатели

```go
// Функция без указателя (копирует значение)
func incrementCopy(x int) {
    x++ // изменяется только копия
}

// Функция с указателем (изменяет оригинал)
func incrementOriginal(x *int) {
    *x++ // изменяется оригинальная переменная
}

num := 5
incrementCopy(num)     // num остается 5
incrementOriginal(&num) // num становится 6
```

---

## Глава 9: Интерфейсы - контракты поведения

### Что такое интерфейс

Интерфейс определяет "что должно уметь", но не "как это делать". Как водительские права - они говорят, что вы можете водить, но не какую именно машину:

```
Интерфейс "Транспорт":
┌─────────────────┐
│ Может:          │
│ - Ехать()       │
│ - Остановиться()│
└─────────────────┘
       ▲
       │ реализуют
   ┌───────┬───────┐
   │       │       │
┌──▼──┐ ┌──▼──┐ ┌──▼──┐
│ Авто│ │Велос│ │Автоб│
│     │ │ипед │ │ус   │
└─────┘ └─────┘ └─────┘
```

```go
// Определение интерфейса
type Writer interface {
    Write(data []byte) (int, error)
}

// Структура, реализующая интерфейс
type FileWriter struct {
    filename string
}

func (fw FileWriter) Write(data []byte) (int, error) {
    // логика записи в файл
    fmt.Printf("Записываем в файл %s: %s\n", fw.filename, string(data))
    return len(data), nil
}

// Использование
var w Writer = FileWriter{filename: "test.txt"}
w.Write([]byte("Привет!"))
```

---

## Глава 10: Обработка ошибок

### Философия обработки ошибок в Go

В Go ошибки - это обычные значения, которые нужно проверять:

```
Обычный подход:
┌─────────────┐
│ Функция     │ ──┐ Успех
├─────────────┤   │
│ Возвращает: │   ├── Результат
│ - Результат │   │
│ - Ошибка    │ ──┘ Ошибка (если есть)
└─────────────┘
```

```go
import (
    "errors"
    "fmt"
)

// Функция, которая может вернуть ошибку
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("деление на ноль недопустимо")
    }
    return a / b, nil
}

// Правильная обработка ошибок
result, err := divide(10, 2)
if err != nil {
    fmt.Println("Произошла ошибка:", err)
    return
}
fmt.Println("Результат:", result)
```

### Создание собственных типов ошибок

```go
// Собственный тип ошибки
type ValidationError struct {
    Field   string
    Message string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("ошибка в поле %s: %s", e.Field, e.Message)
}

// Функция валидации
func validateAge(age int) error {
    if age < 0 {
        return ValidationError{
            Field:   "age",
            Message: "возраст не может быть отрицательным",
        }
    }
    return nil
}
```

---

## Глава 11: Горутины и каналы - конкурентность

### Что такое горутины

Горутина - это легкий поток выполнения. Представьте ресторан с несколькими поварами, работающими одновременно:

```
Обычная программа:    Программа с горутинами:
┌─────────┐           ┌─────────┐ ┌─────────┐ ┌─────────┐
│ Задача 1│ ──▶       │ Задача 1│ │ Задача 2│ │ Задача 3│
│ Задача 2│ ──▶       │         │ │         │ │         │
│ Задача 3│ ──▶       │    ▼    │ │    ▼    │ │    ▼    │
└─────────┘           │ Выполн. │ │ Выполн. │ │ Выполн. │
  (по очереди)        │ парал.  │ │ парал.  │ │ парал.  │
                      └─────────┘ └─────────┘ └─────────┘
```

```go
import (
    "fmt"
    "time"
)

func printNumbers() {
    for i := 1; i <= 5; i++ {
        fmt.Printf("Число: %d\n", i)
        time.Sleep(1 * time.Second)
    }
}

func main() {
    // Запуск горутины
    go printNumbers() // выполняется параллельно
    
    // Основная программа продолжает работать
    fmt.Println("Программа запущена")
    
    // Ждем завершения горутины
    time.Sleep(6 * time.Second)
    fmt.Println("Программа завершена")
}
```

### Каналы - связь между горутинами

Каналы позволяют горутинам обмениваться данными безопасно:

```
Канал как трубопровод:
Горутина 1 ──┬── данные ──▶ Канал ──▶ данные ──┬── Горутина 2
             │                                 │
             └── отправитель                   └── получатель
```

```go
func worker(ch chan string) {
    // Отправляем данные в канал
    ch <- "Работа выполнена!"
}

func main() {
    // Создаем канал
    ch := make(chan string)
    
    // Запускаем горутину
    go worker(ch)
    
    // Получаем данные из канала
    message := <-ch
    fmt.Println(message) // "Работа выполнена!"
}
```

---

## Глава 12: Модули и пакеты

### Система модулей Go

Модуль - это коллекция связанных пакетов. Как библиотека состоит из глав, так модуль состоит из пакетов:

```
Проект (модуль):
├── go.mod (описание модуля)
├── main.go
├── utils/
│   ├── math.go
│   └── string.go
└── models/
    ├── user.go
    └── product.go
```

### Создание модуля

```bash
# Инициализация нового модуля
go mod init myproject

# Добавление зависимости
go get github.com/gorilla/mux
```

### Создание собственного пакета

```go
// файл utils/math.go
package utils

// Публичная функция (начинается с заглавной буквы)
func Add(a, b int) int {
    return a + b
}

// Приватная функция (начинается со строчной буквы)
func multiply(a, b int) int {
    return a * b
}
```

```go
// файл main.go
package main

import (
    "fmt"
    "myproject/utils"
)

func main() {
    result := utils.Add(5, 3)
    fmt.Println(result) // 8
}
```

---

# БЛОК 3: WEB И HTTP РАЗРАБОТКА
*Цель: Освоить создание веб-сервисов и REST API*

---

## Глава 13: HTTP основы

### Что такое HTTP

HTTP - это протокол общения между клиентом (браузер, мобильное приложение) и сервером:

```
HTTP запрос-ответ:
┌─────────────┐     HTTP запрос      ┌─────────────┐
│   Клиент    │ ──────────────────▶ │   Сервер    │
│ (браузер)   │                     │   (Go app)  │
│             │ ◀────────────────── │             │
└─────────────┘     HTTP ответ      └─────────────┘
```

### Простейший HTTP сервер

```go
package main

import (
    "fmt"
    "net/http"
)

// Обработчик запросов
func helloHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Привет! Это мой первый сервер на Go!")
}

func main() {
    // Регистрируем обработчик
    http.HandleFunc("/", helloHandler)
    
    // Запускаем сервер на порту 8080
    fmt.Println("Сервер запущен на http://localhost:8080")
    http.ListenAndServe(":8080", nil)
}
```

### HTTP методы и статус коды

```
HTTP методы:
┌────────┬─────────────────────┐
│ GET    │ Получить данные     │
│ POST   │ Создать новые данные│
│ PUT    │ Обновить данные     │
│ DELETE │ Удалить данные      │
└────────┴─────────────────────┘

Статус коды:
┌─────┬──────────────────┐
│ 200 │ OK (успех)       │
│ 404 │ Не найдено       │
│ 500 │ Ошибка сервера   │
└─────┴──────────────────┘
```

---

## Глава 14: REST API и роутинг

### Что такое REST API

REST API - это набор правил для создания веб-сервисов, как меню в ресторане:

```
REST API для управления пользователями:
┌─────────────┬────────┬─────────────────────┐
│ Путь        │ Метод  │ Действие            │
├─────────────┼────────┼─────────────────────┤
│ /users      │ GET    │ Получить всех       │
│ /users      │ POST   │ Создать нового      │
│ /users/123  │ GET    │ Получить по ID      │
│ /users/123  │ PUT    │ Обновить по ID      │
│ /users/123  │ DELETE │ Удалить по ID       │
└─────────────┴────────┴─────────────────────┘
```

### Простое API для задач

```go
package main

import (
    "encoding/json"
    "net/http"
    "strconv"
    
    "github.com/gorilla/mux"
)

type Task struct {
    ID          int    `json:"id"`
    Title       string `json:"title"`
    Description string `json:"description"`
    Completed   bool   `json:"completed"`
}

var tasks []Task
var nextID = 1

// Получить все задачи
func getTasks(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(tasks)
}

// Создать новую задачу
func createTask(w http.ResponseWriter, r *http.Request) {
    var task Task
    json.NewDecoder(r.Body).Decode(&task)
    
    task.ID = nextID
    nextID++
    tasks = append(tasks, task)
    
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(task)
}

func main() {
    r := mux.NewRouter()
    
    r.HandleFunc("/tasks", getTasks).Methods("GET")
    r.HandleFunc("/tasks", createTask).Methods("POST")
    
    http.ListenAndServe(":8080", r)
}
```

---

## Глава 15: Работа с JSON

### JSON в веб-разработке

JSON - это формат обмена данными между клиентом и сервером:

```
Go структура ──▶ JSON ──▶ Клиент
                ▲
                │ Сериализация
                │ (Marshal)
                
Клиент ──▶ JSON ──▶ Go структура  
                    ▲
                    │ Десериализация
                    │ (Unmarshal)
```

### Преобразование структур в JSON и обратно

```go
type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

func main() {
    // Структура в JSON
    user := User{
        ID:    1,
        Name:  "Алексей",
        Email: "alex@example.com",
    }
    
    jsonData, err := json.Marshal(user)
    if err != nil {
        fmt.Println("Ошибка:", err)
        return
    }
    
    fmt.Println(string(jsonData))
    // {"id":1,"name":"Алексей","email":"alex@example.com"}
    
    // JSON в структуру
    jsonString := `{"id":2,"name":"Мария","email":"maria@example.com"}`
    var newUser User
    
    err = json.Unmarshal([]byte(jsonString), &newUser)
    if err != nil {
        fmt.Println("Ошибка:", err)
        return
    }
    
    fmt.Printf("%+v\n", newUser)
    // {ID:2 Name:Мария Email:maria@example.com}
}
```

---

## Глава 16: Middleware и шаблоны

### Что такое Middleware

Middleware - это промежуточное ПО, которое выполняется до или после основного обработчика:

```
HTTP запрос проходит через цепочку middleware:
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│ Запрос  │ ──▶│ Логгинг │ ──▶│ Аутент. │ ──▶│ Основной│
│         │    │         │    │         │    │обработч.│
└─────────┘    └─────────┘    └─────────┘    └─────────┘
```

```go
// Middleware для логирования
func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Printf("%s %s %s\n", r.RemoteAddr, r.Method, r.URL)
        next.ServeHTTP(w, r)
    })
}

// Использование middleware
func main() {
    r := mux.NewRouter()
    r.HandleFunc("/", homeHandler)
    
    // Применяем middleware ко всем роутам
    http.ListenAndServe(":8080", loggingMiddleware(r))
}
```

---

## Глава 17: WebSockets и real-time

### Что такое WebSockets

WebSocket создает постоянное соединение между клиентом и сервером для real-time общения:

```
HTTP vs WebSocket:
HTTP:
Клиент ──запрос──▶ Сервер
       ◀─ответ────

WebSocket:
Клиент ◀─────────▶ Сервер
       постоянное соединение
       ◀──данные──▶
```

### Простой чат на WebSockets

```go
package main

import (
    "log"
    "net/http"
    
    "github.com/gorilla/websocket"
)

var upgrader = websocket.Upgrader{
    CheckOrigin: func(r *http.Request) bool {
        return true // Разрешаем все origins
    },
}

var clients = make(map[*websocket.Conn]bool)
var broadcast = make(chan []byte)

func handleConnections(w http.ResponseWriter, r *http.Request) {
    // Обновляем HTTP соединение до WebSocket
    ws, err := upgrader.Upgrade(w, r, nil)
    if err != nil {
        log.Fatal(err)
    }
    defer ws.Close()
    
    // Регистрируем нового клиента
    clients[ws] = true
    
    for {
        // Читаем сообщение от клиента
        _, msg, err := ws.ReadMessage()
        if err != nil {
            delete(clients, ws)
            break
        }
        
        // Отправляем сообщение в канал для рассылки
        broadcast <- msg
    }
}

func handleMessages() {
    for {
        // Получаем сообщение из канала
        msg := <-broadcast
        
        // Отправляем всем подключенным клиентам
        for client := range clients {
            err := client.WriteMessage(websocket.TextMessage, msg)
            if err != nil {
                client.Close()
                delete(clients, client)
            }
        }
    }
}

func main() {
    http.HandleFunc("/ws", handleConnections)
    
    go handleMessages()
    
    log.Println("HTTP сервер запущен на :8080")
    err := http.ListenAndServe(":8080", nil)
    if err != nil {
        log.Fatal("ListenAndServe: ", err)
    }
}
```

---

## Глава 18: Тестирование

### Зачем нужны тесты

Тесты - это как проверка домашнего задания. Они убеждаются, что ваш код работает правильно:

```
Без тестов:           С тестами:
┌─────────────┐      ┌─────────────┐
│ Код         │ ──▶  │ Код         │ ──┐
│ Работает?   │      │ + Тесты     │   │
│ Неизвестно  │      │ = Уверенность│  ├── Качество ▲
└─────────────┘      └─────────────┘   │
     ▲                     ▲           │
     │ Много багов         │ Мало багов│
     ▼                     ▼           ▼
```

### Написание тестов

```go
// файл calculator.go
package main

func Add(a, b int) int {
    return a + b
}

func Multiply(a, b int) int {
    return a * b
}
```

```go
// файл calculator_test.go
package main

import "testing"

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    expected := 5
    
    if result != expected {
        t.Errorf("Add(2, 3) = %d; ожидали %d", result, expected)
    }
}

func TestMultiply(t *testing.T) {
    tests := []struct {
        a, b     int
        expected int
    }{
        {2, 3, 6},
        {5, 4, 20},
        {0, 10, 0},
    }
    
    for _, test := range tests {
        result := Multiply(test.a, test.b)
        if result != test.expected {
            t.Errorf("Multiply(%d, %d) = %d; ожидали %d", 
                test.a, test.b, result, test.expected)
        }
    }
}
```

### Тестирование HTTP обработчиков

```go
func TestGetTasks(t *testing.T) {
    // Создаем тестовый запрос
    req, err := http.NewRequest("GET", "/tasks", nil)
    if err != nil {
        t.Fatal(err)
    }
    
    // Создаем ResponseRecorder для записи ответа
    rr := httptest.NewRecorder()
    handler := http.HandlerFunc(getTasks)
    
    // Выполняем запрос
    handler.ServeHTTP(rr, req)
    
    // Проверяем статус код
    if status := rr.Code; status != http.StatusOK {
        t.Errorf("Неправильный статус код: получили %v, ожидали %v",
            status, http.StatusOK)
    }
}
```

---

# БЛОК 4: БАЗЫ ДАННЫХ И ВНЕШНИЕ СЕРВИСЫ
*Цель: Интеграция с базами данных и внешними системами*

---

## Глава 19: SQL базы данных

### Зачем нужны базы данных

База данных - это как огромная, хорошо организованная библиотека для хранения информации:

```
Файлы vs База данных:
Файлы:                  База данных:
┌─────────────┐        ┌─────────────┐
│ user1.txt   │        │ Таблица     │
│ user2.txt   │   ──▶  │ Users       │
│ user3.txt   │        │ ┌─────────┐ │
│ ...         │        │ │id│name │ │
└─────────────┘        │ │1 │Иван │ │
  Хаос                 │ │2 │Анна │ │
                       │ └─────────┘ │
                       └─────────────┘
                         Порядок
```

### Подключение к PostgreSQL

```go
package main

import (
    "database/sql"
    "fmt"
    "log"
    
    _ "github.com/lib/pq"
)

type User struct {
    ID    int
    Name  string
    Email string
}

func main() {
    // Строка подключения к базе данных
    connStr := "user=username dbname=mydb sslmode=disable"
    db, err := sql.Open("postgres", connStr)
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    
    // Проверяем соединение
    err = db.Ping()
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Println("Успешно подключились к базе данных!")
}
```

### CRUD операции

```go
// CREATE - создание пользователя
func createUser(db *sql.DB, name, email string) error {
    query := `INSERT INTO users (name, email) VALUES ($1, $2)`
    _, err := db.Exec(query, name, email)
    return err
}

// READ - получение пользователя
func getUser(db *sql.DB, id int) (*User, error) {
    user := &User{}
    query := `SELECT id, name, email FROM users WHERE id = $1`
    
    row := db.QueryRow(query, id)
    err := row.Scan(&user.ID, &user.Name, &user.Email)
    if err != nil {
        return nil, err
    }
    
    return user, nil
}

// UPDATE - обновление пользователя
func updateUser(db *sql.DB, id int, name, email string) error {
    query := `UPDATE users SET name = $1, email = $2 WHERE id = $3`
    _, err := db.Exec(query, name, email, id)
    return err
}

// DELETE - удаление пользователя
func deleteUser(db *sql.DB, id int) error {
    query := `DELETE FROM users WHERE id = $1`
    _, err := db.Exec(query, id)
    return err
}
```

---

## Глава 20: ORM и миграции

### Что такое ORM

ORM (Object-Relational Mapping) - это мост между объектами Go и таблицами базы данных:

```
Без ORM:                    С ORM:
┌─────────────┐            ┌─────────────┐
│ Go объект   │            │ Go объект   │
├─────────────┤            ├─────────────┤
│ SQL запросы │   ──▶      │ ORM         │ ──▶ База данных
├─────────────┤            ├─────────────┤
│ База данных │            │ Автомагия!  │
└─────────────┘            └─────────────┘
  Много кода                 Меньше кода
```

### Использование GORM

```go
package main

import (
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

type User struct {
    ID    uint   `gorm:"primaryKey"`
    Name  string
    Email string `gorm:"unique"`
}

func main() {
    // Подключение к базе данных
    dsn := "host=localhost user=username dbname=mydb sslmode=disable"
    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        panic("Не удалось подключиться к базе данных")
    }
    
    // Автоматическая миграция (создание таблиц)
    db.AutoMigrate(&User{})
    
    // Создание пользователя
    user := User{Name: "Алексей", Email: "alex@example.com"}
    db.Create(&user)
    
    // Поиск пользователя
    var foundUser User
    db.First(&foundUser, "email = ?", "alex@example.com")
    
    // Обновление
    db.Model(&foundUser).Update("Name", "Александр")
    
    // Удаление
    db.Delete(&foundUser)
}
```

---

## Глава 21: Redis и кэширование

### Зачем нужно кэширование

Кэш - это как блокнот с часто используемой информацией рядом с рабочим местом:

```
Без кэша:               С кэшем:
Запрос ──▶ База данных  Запрос ──▶ Кэш (быстро)
   ▲           │           │         │
   │           ▼           │         ▼ (если нет)
   └─── Медленно           └──▶ База данных
```

### Работа с Redis

```go
package main

import (
    "context"
    "encoding/json"
    "time"
    
    "github.com/go-redis/redis/v8"
)

var ctx = context.Background()

type CacheService struct {
    client *redis.Client
}

func NewCacheService() *CacheService {
    rdb := redis.NewClient(&redis.Options{
        Addr:     "localhost:6379",
        Password: "",
        DB:       0,
    })
    
    return &CacheService{client: rdb}
}

// Сохранение в кэш
func (c *CacheService) Set(key string, value interface{}, expiration time.Duration) error {
    json, err := json.Marshal(value)
    if err != nil {
        return err
    }
    
    return c.client.Set(ctx, key, json, expiration).Err()
}

// Получение из кэша
func (c *CacheService) Get(key string, dest interface{}) error {
    val, err := c.client.Get(ctx, key).Result()
    if err != nil {
        return err
    }
    
    return json.Unmarshal([]byte(val), dest)
}

// Использование
func main() {
    cache := NewCacheService()
    
    user := User{ID: 1, Name: "Алексей", Email: "alex@example.com"}
    
    // Сохраняем пользователя в кэш на 1 час
    cache.Set("user:1", user, time.Hour)
    
    // Получаем из кэша
    var cachedUser User
    err := cache.Get("user:1", &cachedUser)
    if err != nil {
        // Если в кэше нет, получаем из базы данных
        // и сохраняем в кэш
    }
}
```

---

## Глава 22: Внешние API

### Работа с HTTP клиентами

Часто приложения должны общаться с внешними сервисами:

```
Ваше приложение:
┌─────────────────┐     HTTP запросы     ┌─────────────────┐
│ Go сервер       │ ────────────────────▶│ Внешний API     │
│                 │                      │ (GitHub, Slack) │
│                 │ ◀──────────────────── │                 │
└─────────────────┘     HTTP ответы      └─────────────────┘
```

### HTTP клиент с таймаутами

```go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
    "time"
)

type GitHubUser struct {
    Login     string `json:"login"`
    Name      string `json:"name"`
    Followers int    `json:"followers"`
}

func getGitHubUser(username string) (*GitHubUser, error) {
    // Создаем HTTP клиент с таймаутом
    client := &http.Client{
        Timeout: 10 * time.Second,
    }
    
    url := fmt.Sprintf("https://api.github.com/users/%s", username)
    resp, err := client.Get(url)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf("GitHub API вернул статус: %d", resp.StatusCode)
    }
    
    var user GitHubUser
    err = json.NewDecoder(resp.Body).Decode(&user)
    if err != nil {
        return nil, err
    }
    
    return &user, nil
}

func main() {
    user, err := getGitHubUser("octocat")
    if err != nil {
        fmt.Println("Ошибка:", err)
        return
    }
    
    fmt.Printf("Пользователь: %s (%s), Подписчиков: %d\n", 
        user.Name, user.Login, user.Followers)
}
```

---

# БЛОК 5: ПРОДАКШН И ЭКСПЕРТИЗА
*Цель: Подготовка к production и углубленные знания*

---

## Глава 23: Логирование и мониторинг

### Структурированное логирование

Хорошие логи - это как дневник программы, который помогает понять, что происходило:

```
Плохие логи:           Хорошие логи:
"Ошибка!"             {"level":"error", "msg":"database connection failed", 
                       "error":"timeout", "timestamp":"2024-01-15T10:30:00Z"}
"Пользователь"        {"level":"info", "msg":"user logged in", 
                       "user_id":123, "ip":"192.168.1.1"}
```

### Использование структурированного логгера

```go
package main

import (
    "github.com/sirupsen/logrus"
)

func setupLogger() *logrus.Logger {
    logger := logrus.New()
    
    // Устанавливаем JSON формат
    logger.SetFormatter(&logrus.JSONFormatter{})
    
    // Устанавливаем уровень логирования
    logger.SetLevel(logrus.InfoLevel)
    
    return logger
}

func main() {
    logger := setupLogger()
    
    // Различные уровни логирования
    logger.Debug("Отладочная информация")
    logger.Info("Информационное сообщение")
    logger.Warn("Предупреждение")
    logger.Error("Ошибка")
    
    // Структурированные логи с полями
    logger.WithFields(logrus.Fields{
        "user_id": 123,
        "action":  "login",
        "ip":      "192.168.1.1",
    }).Info("Пользователь вошел в систему")
    
    // Логирование ошибок с контекстом
    err := someFunction()
    if err != nil {
        logger.WithFields(logrus.Fields{
            "error":    err.Error(),
            "function": "someFunction",
        }).Error("Произошла ошибка")
    }
}
```

---

## Глава 24: Оптимизация и профилирование

### Профилирование производительности

Профилирование показывает, где ваша программа тратит время и память:

```
Профилирование как рентген программы:
┌─────────────────┐
│ Функция A: 40%  │ ←── Узкое место!
├─────────────────┤
│ Функция B: 30%  │
├─────────────────┤
│ Функция C: 20%  │
├─────────────────┤
│ Остальное: 10%  │
└─────────────────┘
```

### Встроенный профайлер Go

```go
package main

import (
    _ "net/http/pprof" // Подключаем профайлер
    "net/http"
    "log"
)

func expensiveFunction() {
    // Имитация тяжелой работы
    for i := 0; i < 1000000; i++ {
        _ = i * i
    }
}

func handler(w http.ResponseWriter, r *http.Request) {
    expensiveFunction()
    w.Write([]byte("Готово!"))
}

func main() {
    http.HandleFunc("/", handler)
    
    // Профайлер доступен по адресу /debug/pprof/
    log.Println("Сервер запущен на :8080")
    log.Println("Профайлер доступен на http://localhost:8080/debug/pprof/")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### Оптимизация памяти

```go
// Неэффективно: создается много временных строк
func inefficientConcat(strs []string) string {
    result := ""
    for _, s := range strs {
        result += s // Каждая операция создает новую строку
    }
    return result
}

// Эффективно: используем strings.Builder
func efficientConcat(strs []string) string {
    var builder strings.Builder
    for _, s := range strs {
        builder.WriteString(s)
    }
    return builder.String()
}

// Использование пула объектов для переиспользования
var stringBuilderPool = sync.Pool{
    New: func() interface{} {
        return &strings.Builder{}
    },
}

func pooledConcat(strs []string) string {
    builder := stringBuilderPool.Get().(*strings.Builder)
    defer func() {
        builder.Reset()
        stringBuilderPool.Put(builder)
    }()
    
    for _, s := range strs {
        builder.WriteString(s)
    }
    return builder.String()
}
```

---

## Глава 25: Деплой и DevOps

### Контейнеризация с Docker

Docker упаковывает ваше приложение в контейнер - как посылку со всем необходимым:

```
Без Docker:               С Docker:
┌─────────────────┐      ┌─────────────────┐
│ Сервер 1        │      │ Контейнер       │
│ - Go 1.19       │      │ - Go 1.20       │ ──┐
│ - Зависимости   │ ──▶  │ - Приложение    │   │ Работает везде
│ - Конфигурация  │      │ - Зависимости   │   │ одинаково
└─────────────────┘      └─────────────────┘ ──┘
```

### Dockerfile для Go приложения

```dockerfile
# Многоэтапная сборка
FROM golang:1.20-alpine AS builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

# Финальный образ
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/

COPY --from=builder /app/main .

EXPOSE 8080
CMD ["./main"]
```

### Graceful Shutdown

```go
package main

import (
    "context"
    "fmt"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
)

func main() {
    // Создаем HTTP сервер
    srv := &http.Server{
        Addr:    ":8080",
        Handler: setupRoutes(),
    }
    
    // Запускаем сервер в горутине
    go func() {
        fmt.Println("Сервер запущен на :8080")
        if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            fmt.Printf("Ошибка сервера: %v\n", err)
        }
    }()
    
    // Ожидаем сигнал завершения
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    
    fmt.Println("Завершаем сервер...")
    
    // Graceful shutdown с таймаутом
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    if err := srv.Shutdown(ctx); err != nil {
        fmt.Printf("Принудительное завершение сервера: %v\n", err)
    }
    
    fmt.Println("Сервер завершен")
}

func setupRoutes() http.Handler {
    mux := http.NewServeMux()
    mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello, World!"))
    })
    return mux
}
```

---

## 🎯 Финальный проект: Система управления задачами

После изучения всех глав создайте комплексный проект:

### Архитектура системы

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Веб-клиент      │    │ API Gateway     │    │ Микросервисы    │
│ (React/Vue)     │◀──▶│ (Go + Gin)      │◀──▶│ - Пользователи  │
└─────────────────┘    └─────────────────┘    │ - Задачи        │
                                              │ - Уведомления   │
                                              └─────────────────┘
           ▲                       ▲                    ▲
           │                       │                    │
           ▼                       ▼                    ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Redis           │    │ PostgreSQL      │    │ Мониторинг      │
│ (кэш, сессии)   │    │ (основные       │    │ (Prometheus)    │
│                 │    │ данные)         │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Основные компоненты

1. **API для управления пользователями**
   - Регистрация и аутентификация
   - JWT токены
   - Ролевая модель

2. **API для управления задачами**
   - CRUD операции
   - Фильтрация и поиск
   - Категории и приоритеты

3. **Система уведомлений**
   - Email уведомления
   - WebSocket для real-time обновлений

4. **Кэширование и оптимизация**
   - Redis для часто запрашиваемых данных
   - Database connection pooling

5. **Мониторинг и логирование**
   - Структурированные логи
   - Метрики производительности
   - Health checks

---

## 📚 Что дальше?

Поздравляем! Вы прошли путь от новичка до эксперта Go разработки. Теперь вы умеете:

✅ Писать чистый и эффективный код на Go  
✅ Создавать веб-сервисы и REST API  
✅ Работать с базами данных и внешними сервисами  
✅ Использовать конкурентность и горутины  
✅ Тестировать и оптимизировать приложения  
✅ Деплоить в production  

### Направления для роста:

1. **Углубленное изучение**
   - Kubernetes и оркестрация
   - gRPC и микросервисы
   - Распределенные системы

2. **Специализация**
   - DevOps и инфраструктура
   - Высоконагруженные системы
   - Blockchain и криптография

3. **Сообщество**
   - Участие в Open Source проектах
   - Техническое писательство
   - Менторство новичков

**Удачи в вашем путешествии с Go! 🚀**