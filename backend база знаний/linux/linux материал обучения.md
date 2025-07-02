# Linux для Backend разработчика: От новичка до эксперта

## Введение

Linux — это не просто операционная система, это целая экосистема, на которой работает 96.3% из миллиона самых загруженных веб-серверов в мире. Каждый раз, когда вы открываете Instagram, заказываете еду или смотрите Netflix — вы взаимодействуете с Linux серверами.

### Почему Linux критичен для Backend разработчика?

```
Современная веб-архитектура:

Пользователь → CDN (Linux) → Load Balancer (Linux) → Web Server (Linux) → App Server (Linux) → Database (Linux)
    ↓             ↓              ↓                    ↓                   ↓                ↓
  Браузер     Nginx/Apache    HAProxy/Nginx      Docker/K8s        Python/Java      PostgreSQL/MySQL
```

**Что вы получите от изучения:**
- Способность развернуть и масштабировать любое приложение
- Навыки отладки проблем на production серверах
- Понимание того, как работают Docker, Kubernetes, облачные сервисы
- Автоматизацию развертывания и мониторинга
- Экспертизу в сетевых технологиях и безопасности

---

# Блок 1: Основы командной строки и файловой системы

## Глава 1: Философия Unix/Linux и файловая система

### Философия Unix/Linux

Linux следует философии Unix: "Делай одну вещь, но делай её хорошо". Каждая утилита решает конкретную задачу, но может объединяться с другими для решения сложных проблем.

```
Философия Unix в действии:

cat file.txt | grep "ERROR" | sort | uniq -c | sort -nr
  ↓           ↓              ↓      ↓          ↓
Читать    Фильтровать   Сортировать Убрать   Сортировать
файл      строки        строки      дубли    по числам
```

### Файловая система Linux — подробно

Linux организует всё как файлы в древовидной структуре. Понимание этой структуры критично для работы с серверами:

```
/                           (корень файловой системы)
├── bin/                    (основные исполняемые файлы: ls, cat, grep)
│   ├── ls → /bin/ls
│   ├── cat → /bin/cat
│   └── bash → /bin/bash
├── boot/                   (файлы загрузки системы)
│   ├── vmlinuz             (ядро Linux)
│   └── initrd.img          (начальный RAM диск)
├── dev/                    (файлы устройств)
│   ├── sda1                (жесткий диск, раздел 1)
│   ├── null                (виртуальное устройство "черная дыра")
│   └── random              (генератор случайных чисел)
├── etc/                    (конфигурационные файлы)
│   ├── nginx/
│   │   ├── nginx.conf      (главная конфигурация)
│   │   └── sites-enabled/  (активные сайты)
│   ├── passwd              (пользователи системы)
│   ├── hosts               (локальная DNS таблица)
│   └── crontab             (расписание задач)
├── home/                   (домашние папки пользователей)
│   ├── alice/
│   ├── bob/
│   └── developer/
│       ├── .bashrc         (настройки bash)
│       ├── .ssh/           (SSH ключи)
│       └── projects/
├── lib/                    (системные библиотеки)
├── media/                  (съемные носители)
├── mnt/                    (временные монтирования)
├── opt/                    (дополнительное ПО)
├── proc/                   (виртуальная FS с информацией о процессах)
│   ├── cpuinfo             (информация о CPU)
│   ├── meminfo             (информация о памяти)
│   └── 1234/               (папка процесса с PID 1234)
├── root/                   (домашняя папка root)
├── run/                    (runtime данные)
├── sbin/                   (системные исполняемые файлы)
├── srv/                    (данные сервисов)
├── sys/                    (виртуальная FS для взаимодействия с ядром)
├── tmp/                    (временные файлы, очищается при перезагрузке)
├── usr/                    (пользовательские программы и данные)
│   ├── bin/                (пользовательские команды)
│   ├── lib/                (библиотеки)
│   ├── local/              (локально установленное ПО)
│   └── share/              (общие данные)
└── var/                    (изменяемые данные)
    ├── log/                (логи системы и приложений)
    │   ├── nginx/          (логи веб-сервера)
    │   ├── auth.log        (аутентификация)
    │   └── syslog          (системные сообщения)
    ├── www/                (веб-контент)
    ├── lib/                (данные приложений)
    └── tmp/                (временные файлы приложений)
```

### Навигация по файловой системе

**Основные команды навигации с примерами:**

```bash
# Показать текущую директорию
pwd
# Результат: /home/developer/projects

# Перейти в папку (абсолютный путь)
cd /var/log/nginx
pwd
# Результат: /var/log/nginx

# Перейти в папку (относительный путь)
cd ../../www/html
pwd
# Результат: /var/www/html

# Специальные пути
cd ~           # домашняя папка текущего пользователя
cd -           # предыдущая папка
cd ../         # на один уровень вверх
cd ../../      # на два уровня вверх

# Список файлов с различными опциями
ls                    # простой список
ls -l                # детальный список
ls -la               # включая скрытые файлы
ls -lh               # размеры в читаемом виде
ls -lt               # сортировка по времени
ls -lS               # сортировка по размеру

# Пример вывода ls -la:
# drwxr-xr-x  2 user group  4096 Jan 15 10:30 documents
# -rw-r--r--  1 user group  1024 Jan 15 09:15 config.txt
# -rwxr-xr-x  1 user group  2048 Jan 14 16:45 script.sh
```

## Глава 2: Права доступа и управление файлами

### Права доступа — глубокое понимание

Система прав Linux основана на трех типах доступа для трех категорий пользователей:

```
Структура прав доступа:

-rwxr-xr--
│└┬┘└┬┘└┬┘
│ │  │  └── Другие пользователи (others): r-- (только чтение)
│ │  └───── Группа (group): r-x (чтение и выполнение)
│ └──────── Владелец (owner): rwx (все права)
└─────────── Тип файла: - (обычный файл), d (директория), l (ссылка)

Права в числовом виде:
r (read)    = 4
w (write)   = 2  
x (execute) = 1

rwx = 4+2+1 = 7
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4

Таким образом: -rwxr-xr-- = 754
```

**Практические примеры работы с правами:**

```bash
# Создание файла и изменение прав
touch script.sh
echo "#!/bin/bash" > script.sh
echo "echo 'Hello World'" >> script.sh

# Проверим права
ls -l script.sh
# -rw-r--r-- 1 user group 32 Jan 15 10:30 script.sh

# Файл не выполняется, добавим права
chmod +x script.sh
ls -l script.sh
# -rwxr-xr-x 1 user group 32 Jan 15 10:30 script.sh

# Теперь можно выполнить
./script.sh
# Hello World

# Различные способы установки прав
chmod 755 script.sh      # rwxr-xr-x
chmod u+x script.sh      # добавить выполнение владельцу
chmod g-w script.sh      # убрать запись группе
chmod o=r script.sh      # установить только чтение для других

# Рекурсивное изменение прав
chmod -R 644 /var/www/html/   # все файлы только для чтения
find /var/www -type d -exec chmod 755 {} \;   # папки доступны для обхода
find /var/www -type f -exec chmod 644 {} \;   # файлы только для чтения
```

### Создание и управление файлами и директориями

```bash
# Создание директорий
mkdir project
mkdir -p project/src/main/java    # создать всю иерархию
mkdir -p logs/{nginx,app,db}       # создать несколько папок

# Результирующая структура:
# project/
# ├── src/
# │   └── main/
# │       └── java/
# └── logs/
#     ├── nginx/
#     ├── app/
#     └── db/

# Создание файлов
touch app.py                       # пустой файл
touch {config,logs,data}.txt       # несколько файлов
echo "print('Hello')" > app.py     # файл с содержимым

# Копирование
cp app.py backup/                  # копировать файл
cp -r project/ backup/             # копировать папку рекурсивно
cp -a project/ backup/             # копировать с сохранением атрибутов

# Перемещение и переименование
mv app.py main.py                  # переименовать
mv main.py src/                    # переместить
mv src/ application/               # переименовать папку

# Удаление
rm file.txt                        # удалить файл
rm -r folder/                      # удалить папку
rm -rf folder/                     # принудительное удаление
rm *.tmp                          # удалить по маске
```

## Глава 3: Просмотр и поиск содержимого

### Просмотр содержимого файлов

```bash
# Полный просмотр
cat config.txt                     # весь файл сразу
cat file1.txt file2.txt           # несколько файлов

# Постраничный просмотр
less access.log                    # интерактивный просмотр
more access.log                    # простой постраничный просмотр

# Начало и конец файла
head -10 error.log                 # первые 10 строк
head -c 100 binary.dat             # первые 100 байт
tail -20 access.log                # последние 20 строк
tail -f error.log                  # следить за новыми строками

# Нумерация строк
cat -n script.py                   # с номерами строк
nl script.py                       # альтернативный способ

# Специальные символы
cat -A file.txt                    # показать все спецсимволы
cat -T file.txt                    # показать табы как ^I
```

### Поиск файлов и содержимого

**Команда find — мощный инструмент поиска:**

```bash
# Поиск по имени
find /var -name "*.log"                    # все .log файлы
find . -name "config*"                     # файлы начинающиеся с config
find /etc -iname "NGINX*"                  # регистронезависимый поиск

# Поиск по типу
find /usr -type f -name "python*"          # только файлы
find /var -type d -name "log*"             # только директории
find /tmp -type l                          # только символические ссылки

# Поиск по размеру
find /var/log -size +100M                  # файлы больше 100 МБ
find . -size -1k                           # файлы меньше 1 КБ
find /tmp -size +1G -size -2G               # файлы от 1 до 2 ГБ

# Поиск по времени
find /var/log -mtime -7                     # изменены за последние 7 дней
find /tmp -atime +30                        # не использовались 30+ дней
find . -newer reference_file.txt            # новее определенного файла

# Поиск по правам
find /usr/bin -perm 755                     # точные права 755
find /var -perm -644                        # минимум 644
find /tmp -perm /022                        # любой из битов записи для группы/других

# Выполнение команд над найденными файлами
find /var/log -name "*.log" -exec ls -lh {} \;     # показать детали
find /tmp -name "*.tmp" -delete                     # удалить найденные
find . -name "*.py" -exec grep -l "import flask" {} \;  # найти Python файлы с Flask

# Комбинированный поиск
find /var/log -name "*.log" -size +10M -mtime -1    # большие логи за сутки
```

**Команда grep — поиск внутри файлов:**

```bash
# Простой поиск
grep "error" /var/log/nginx/error.log       # найти строки с "error"
grep -i "Error" application.log             # регистронезависимый поиск
grep -v "INFO" app.log                      # все строки КРОМЕ содержащих "INFO"

# Контекст поиска
grep -A 3 "ERROR" app.log                   # показать 3 строки после
grep -B 2 "FATAL" app.log                   # показать 2 строки до
grep -C 5 "Exception" app.log               # показать 5 строк до и после

# Рекурсивный поиск
grep -r "database" /etc/                    # поиск по всем файлам в /etc/
grep -r --include="*.py" "import" ./        # только в Python файлах
grep -r --exclude="*.log" "password" ./     # исключить лог файлы

# Регулярные выражения
grep "^Error" app.log                       # строки начинающиеся с "Error"
grep "failed$" app.log                      # строки заканчивающиеся на "failed"
grep -E "error|warning|fatal" app.log       # несколько вариантов

# Полезная информация
grep -n "Exception" app.log                 # с номерами строк
grep -c "GET" access.log                    # количество совпадений
grep -l "import flask" *.py                 # только имена файлов
```

## Глава 4: Работа с текстом и потоками

### Обработка текста с awk и sed

**awk — мощный инструмент обработки текста:**

```bash
# Базовые операции
awk '{print $1}' access.log                 # первый столбец (IP адреса)
awk '{print $1, $7}' access.log             # IP и URL
awk '{print NF}' file.txt                   # количество полей в каждой строке
awk '{print NR, $0}' file.txt               # номер строки и содержимое

# Фильтрация
awk '$9 == 404' access.log                  # только 404 ошибки
awk '$10 > 1000' access.log                 # ответы больше 1000 байт
awk '/ERROR/ {print $1, $3}' app.log        # обработка строк с ERROR

# Суммирование и подсчет
awk '{sum += $10} END {print sum}' access.log           # сумма размеров ответов
awk '{count[$1]++} END {for(ip in count) print ip, count[ip]}' access.log  # подсчет по IP

# Форматирование вывода
awk '{printf "%-15s %10s\n", $1, $7}' access.log       # выровненный вывод
```

**sed — потоковый редактор:**

```bash
# Замена текста
sed 's/old/new/' file.txt                   # заменить первое вхождение в каждой строке
sed 's/old/new/g' file.txt                  # заменить все вхождения
sed 's/old/new/2' file.txt                  # заменить второе вхождение

# Удаление строк
sed '/pattern/d' file.txt                   # удалить строки содержащие pattern
sed '1d' file.txt                           # удалить первую строку
sed '$d' file.txt                           # удалить последнюю строку
sed '2,5d' file.txt                         # удалить строки с 2 по 5

# Вставка и добавление
sed '2i\New line' file.txt                  # вставить строку перед 2-й
sed '2a\New line' file.txt                  # добавить строку после 2-й
sed '$a\Last line' file.txt                 # добавить в конец

# Редактирование на месте
sed -i 's/old/new/g' file.txt               # изменить файл
sed -i.bak 's/old/new/g' file.txt           # создать backup
```

### Перенаправление ввода/вывода и пайпы

**Концепция потоков в Linux:**

```
Каждый процесс имеет 3 стандартных потока:

stdin  (0) ────→ [Процесс] ────→ stdout (1)
                     │
                     ↓
                  stderr (2)

stdin  - стандартный ввод (клавиатура)
stdout - стандартный вывод (экран)  
stderr - стандартный поток ошибок (экран)
```

**Перенаправление:**

```bash
# Перенаправление вывода
echo "Hello" > file.txt                     # перезаписать файл
echo "World" >> file.txt                    # добавить в конец
ls > files.txt 2> errors.txt               # stdout в один файл, stderr в другой
ls > all.txt 2>&1                          # объединить stdout и stderr

# Перенаправление ввода
wc -l < file.txt                            # прочитать из файла
sort < unsorted.txt > sorted.txt            # ввод из файла, вывод в файл

# Игнорирование вывода
command > /dev/null                         # игнорировать stdout
command 2> /dev/null                        # игнорировать stderr
command > /dev/null 2>&1                    # игнорировать всё
```

**Пайпы — соединение команд:**

```bash
# Базовые примеры
ls -la | grep "Jan"                         # файлы изменённые в январе
ps aux | grep nginx                         # процессы nginx
cat access.log | grep "POST" | wc -l        # количество POST запросов

# Сложные пайплайны
cat access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -10
# ↑ Топ-10 IP адресов по количеству запросов:
# 1. Извлечь IP адреса (первый столбец)
# 2. Отсортировать
# 3. Убрать дубликаты и подсчитать
# 4. Отсортировать по количеству (убывание)
# 5. Показать первые 10

# Анализ логов веб-сервера
cat access.log | grep " 404 " | awk '{print $7}' | sort | uniq -c | sort -nr
# ↑ Самые частые 404 ошибки

# Мониторинг системы в реальном времени
tail -f /var/log/syslog | grep -i error | while read line; do echo "$(date): $line"; done
# ↑ Отслеживание ошибок с временными метками
```

## Глава 5: Архивирование и сжатие

### tar — основной инструмент архивирования

```bash
# Создание архивов
tar -cf archive.tar folder/                 # создать tar архив
tar -czf archive.tar.gz folder/             # создать с gzip сжатием
tar -cjf archive.tar.bz2 folder/            # создать с bzip2 сжатием
tar -cJf archive.tar.xz folder/             # создать с xz сжатием

# Просмотр содержимого
tar -tf archive.tar                         # список файлов
tar -tvf archive.tar                        # детальный список

# Извлечение
tar -xf archive.tar                         # извлечь в текущую папку
tar -xf archive.tar -C /destination/        # извлечь в определенную папку
tar -xf archive.tar file.txt                # извлечь только определенный файл

# Обновление архива
tar -rf archive.tar newfile.txt             # добавить файл в существующий архив

# Практические примеры
tar -czf backup_$(date +%Y%m%d).tar.gz /home/user/  # backup с датой
tar -czf - folder/ | ssh user@server "cat > remote_backup.tar.gz"  # backup на удаленный сервер
```

### Другие инструменты сжатия

```bash
# gzip/gunzip
gzip file.txt                               # сжать файл (создаст file.txt.gz)
gunzip file.txt.gz                          # распаковать
zcat file.txt.gz                            # просмотреть без распаковки

# zip/unzip
zip -r archive.zip folder/                  # создать zip архив
unzip archive.zip                           # распаковать
unzip -l archive.zip                        # список файлов в архиве

# Сравнение степени сжатия
ls -lh original_folder
tar -czf folder.tar.gz original_folder      # gzip
tar -cjf folder.tar.bz2 original_folder     # bzip2  
tar -cJf folder.tar.xz original_folder      # xz
ls -lh folder.tar.*
# Обычно: xz > bzip2 > gzip по степени сжатия
#         gzip > bzip2 > xz по скорости
```

---

# Блок 2: Процессы и системные ресурсы

## Глава 6: Понимание процессов в Linux

### Иерархия процессов

В Linux всё является процессом. Каждая программа, каждый сервис — это процесс с уникальным идентификатором (PID).

```
Иерархия процессов в Linux:

systemd (PID: 1) ──────────────── Главный процесс системы
├── NetworkManager (PID: 234)
├── sshd (PID: 456) ─── ssh сессии
│   ├── sshd (PID: 1001) ─── пользователь alice
│   └── sshd (PID: 1002) ─── пользователь bob
├── nginx (PID: 789)
│   ├── nginx worker (PID: 790)
│   ├── nginx worker (PID: 791)
│   └── nginx worker (PID: 792)
└── postgresql (PID: 567)
    ├── postgres writer (PID: 568)
    ├── postgres wal writer (PID: 569)
    └── postgres autovacuum (PID: 570)
```

### Просмотр и анализ процессов

**Команда ps — снимок процессов:**

```bash
# Базовые варианты
ps                                          # процессы текущего пользователя
ps aux                                      # все процессы системы
ps -ef                                      # альтернативный формат

# Пример вывода ps aux:
# USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
# root         1  0.0  0.4 168548  9012 ?        Ss   10:30   0:01 /sbin/init
# www-data   789  0.1  0.8  55968  17284 ?       S    10:31   0:00 nginx: master
# www-data   790  0.0  0.4  56420  8192 ?        S    10:31   0:00 nginx: worker

# Фильтрация процессов
ps aux | grep nginx                         # только nginx процессы
ps aux | grep -v grep                       # исключить сам grep
pgrep nginx                                 # только PID процессов nginx
pgrep -f "python.*app.py"                  # поиск по полной команде

# Дерево процессов
ps auxf                                     # в виде дерева
pstree                                      # простое дерево
pstree -p                                   # с PID
pstree nginx                                # дерево конкретного процесса
```

**Расшифровка статусов процессов:**

```
STAT колонка в ps aux:

D  - Непрерываемый сон (обычно I/O)
R  - Выполняется или готов к выполнению  
S  - Прерываемый сон (ожидает события)
T  - Остановлен
Z  - Зомби (завершился, но parent не прочитал exit status)

Дополнительные флаги:
<  - Высокий приоритет
N  - Низкий приоритет  
L  - Заблокированы страницы в памяти
s  - Лидер сессии
l  - Многопоточный
+  - В foreground группе процессов
```

### Интерактивный мониторинг с top и htop

**top — системный монитор:**

```bash
top
# Интерактивные команды:
# h - помощь
# q - выход
# M - сортировка по памяти
# P - сортировка по CPU
# k - убить процесс
# 1 - показать все CPU ядра
# f - выбрать поля для отображения

# Запуск с параметрами
top -p 1234,5678                           # мониторинг конкретных PID
top -u nginx                               # процессы конкретного пользователя
top -d 2                                   # обновление каждые 2 секунды
```

**htop — улучшенная версия top:**

```bash
htop
# Дополнительные возможности:
# - Цветной интерфейс
# - Мышь поддержка  
# - Горизонтальная прокрутка
# - Древовидный вид процессов
# - Встроенный поиск и фильтрация

# Горячие клавиши htop:
# F3 - поиск
# F4 - фильтр
# F5 - древовидный вид
# F6 - сортировка
# F9 - убить процесс
# F10 - выход
```

## Глава 7: Управление процессами

### Сигналы в Linux

```
Основные сигналы:

SIGTERM (15) - Вежливая просьба завершиться (по умолчанию)
SIGKILL (9)  - Принудительное завершение (нельзя перехватить)
SIGHUP (1)   - Перечитать конфигурацию
SIGSTOP (19) - Приостановить выполнение
SIGCONT (18) - Продолжить выполнение
SIGINT (2)   - Прерывание (Ctrl+C)
SIGQUIT (3)  - Quit (Ctrl+\)
SIGUSR1 (10) - Пользовательский сигнал 1
SIGUSR2 (12) - Пользовательский сигнал 2
```

### Команды управления процессами

```bash
# Завершение процессов
kill 1234                                  # послать SIGTERM процессу 1234
kill -9 1234                               # принудительно завершить (SIGKILL)
kill -HUP 1234                             # перечитать конфигурацию
killall nginx                              # завершить все процессы nginx
pkill -f "python.*app.py"                  # завершить по паттерну команды

# Запуск в фоне
nohup python app.py &                      # запустить в фоне, устойчиво к SIGHUP
python app.py &                            # просто в фоне
disown                                     # отвязать от текущей сессии

# Управление заданиями
jobs                                       # показать фоновые задания
fg %1                                      # перенести задание 1 на передний план
bg %1                                      # продолжить задание 1 в фоне
# Ctrl+Z                                   # приостановить текущий процесс
```

### Практические примеры

```bash
# Безопасная перезагрузка nginx
sudo nginx -t                              # проверить конфигурацию
sudo kill -HUP $(cat /var/run/nginx.pid)   # перечитать конфигурацию

# Мониторинг конкретного процесса
watch -n 1 "ps aux | grep python"          # обновлять каждую секунду

# Запуск долгой задачи
nohup python long_running_script.py > output.log 2>&1 &
echo $!                                    # PID запущенного процесса
```

## Глава 8: Мониторинг системных ресурсов

### Память системы

```bash
# Основная информация о памяти
free -h
#               total        used        free      shared  buff/cache   available
# Mem:           8.0G        2.1G        1.2G        125M        4.7G        5.5G
# Swap:          2.0G          0B        2.0G

# Расшифровка:
# total     - общий объем RAM
# used      - используется приложениями
# free      - свободная память  
# shared    - разделяемая память (tmpfs, etc.)
# buff/cache - буферы и кеш файловой системы
# available - доступная для новых приложений

# Детальная информация
cat /proc/meminfo | head -20

# Процессы по использованию памяти
ps aux --sort=-%mem | head -10

# Карта памяти процесса
pmap 1234                                  # карта памяти процесса 1234
pmap -x 1234                               # с дополнительной информацией
```

### Дисковое пространство

```bash
# Использование файловых систем
df -h
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda1        20G   15G  4.2G  79% /
# /dev/sda2       100G   45G   50G  48% /home
# tmpfs           4.0G     0  4.0G   0% /dev/shm

# Размер директорий
du -h /var/log                             # размер /var/log
du -sh /var/log/*                          # размер каждого файла/папки
du -sh * | sort -hr                        # сортировка по размеру

# Поиск больших файлов
find /var -type f -size +100M -exec ls -lh {} \; | sort -k5 -hr

# Анализ использования inode
df -i                                      # количество inode
find /var -type f | wc -l                  # количество файлов

# Самые большие файлы в системе
find / -type f -size +1G -exec ls -lh {} \; 2>/dev/null
```

### I/O статистика

```bash
# iostat - статистика I/O
iostat -x 1                                # каждую секунду
# Device            r/s     w/s    rkB/s    wkB/s
# sda              12.34    5.67   123.45    67.89

# iotop - топ процессов по I/O
sudo iotop

# Статистика дисков
cat /proc/diskstats

# Активность файловой системы  
sudo iotop -o                              # только активные процессы
sudo iotop -P                              # группировка по процессам
```

## Глава 9: Системная информация

### Информация о CPU

```bash
# Детальная информация о процессоре
lscpu
# Architecture:        x86_64
# CPU(s):              4
# Thread(s) per core:  2
# Core(s) per socket:  2
# Model name:          Intel Core i5-8250U

# Простая информация
cat /proc/cpuinfo | grep "model name" | head -1
nproc                                      # количество CPU ядер

# Загрузка CPU
top -bn1 | grep "Cpu(s)"
# %Cpu(s):  5.2 us,  2.1 sy,  0.0 ni, 92.1 id,  0.6 wa,  0.0 hi,  0.0 si,  0.0 st

# Средняя загрузка системы
uptime
# 14:30:15 up 2 days,  4:15,  3 users,  load average: 0.15, 0.25, 0.30

# Load average расшифровка:
# 0.15 - средняя загрузка за 1 минуту
# 0.25 - средняя загрузка за 5 минут  
# 0.30 - средняя загрузка за 15 минут
# Значения выше количества CPU ядер означают очередь процессов
```

### Информация об оборудовании

```bash
# PCI устройства
lspci                                      # все PCI устройства
lspci | grep -i network                    # сетевые карты
lspci | grep -i vga                        # видеокарты

# USB устройства
lsusb

# Блочные устройства (диски)
lsblk
# NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
# sda      8:0    0   20G  0 disk 
# ├─sda1   8:1    0    1G  0 part /boot
# └─sda2   8:2    0   19G  0 part /

# Подробная информация о дисках
sudo fdisk -l
sudo parted -l
```

### Сетевая активность (базовый обзор)

```bash
# Сетевые соединения
netstat -tuln                              # слушающие порты
netstat -tun                               # активные соединения
ss -tuln                                   # современная альтернатива netstat

# Статистика сетевых интерфейсов
cat /proc/net/dev
ip -s link show                            # статистика интерфейсов

# Что использует порт
lsof -i :80                                # что слушает порт 80
lsof -i :443                               # что слушает порт 443
sudo netstat -tlnp | grep :22              # что слушает SSH порт

# Сетевая активность по процессам
sudo lsof -i                               # все сетевые соединения
sudo lsof -i -P                            # с портами вместо названий сервисов
```

---

# Блок 3: Пользователи и безопасность

## Глава 10: Архитектура безопасности Linux

### Уровни безопасности в Linux

```
Уровни безопасности в Linux:

Пользователь (alice) ──→ Группы (developers, www-data) ──→ Права доступа
         ↓                           ↓                            ↓
    Аутентификация            Авторизация                 Файловые права
         ↓                           ↓                            ↓
    (/etc/passwd)           (/etc/group)              (rwxrwxrwx)
         ↓                           ↓                            ↓
        sudo                   Capability                   ACL
         ↓                           ↓                            ↓
    (/etc/sudoers)            (setcap)                (setfacl/getfacl)
```

### Управление пользователями

**Создание и настройка пользователей:**

```bash
# Создание нового пользователя
sudo useradd -m -s /bin/bash alice         # создать с домашней папкой и bash
sudo useradd -m -s /bin/bash -G sudo bob   # создать и добавить в группу sudo
sudo useradd -r -s /bin/false nginx        # системный пользователь без оболочки

# Установка пароля
sudo passwd alice                          # интерактивная установка пароля
echo "alice:newpassword" | sudo chpasswd   # установка через команду

# Изменение пользователя
sudo usermod -aG docker alice              # добавить в группу docker
sudo usermod -s /bin/zsh alice             # сменить оболочку на zsh
sudo usermod -L alice                      # заблокировать учетную запись
sudo usermod -U alice                      # разблокировать учетную запись

# Удаление пользователя
sudo userdel alice                         # удалить без домашней папки
sudo userdel -r alice                      # удалить с домашней папкой

# Информация о пользователях
id alice                                   # группы пользователя alice
whoami                                     # текущий пользователь
who                                        # кто сейчас в системе
w                                          # детальная информация о пользователях
last                                       # история входов в систему
```

**Файлы управления пользователями:**

```bash
# /etc/passwd - информация о пользователях
cat /etc/passwd | grep alice
# alice:x:1001:1001:Alice Smith,,,:/home/alice:/bin/bash
#   ↑   ↑  ↑    ↑    ↑               ↑           ↑
# имя пароль UID GID полное_имя  домашняя_папка оболочка

# /etc/shadow - зашифрованные пароли (только root)
sudo cat /etc/shadow | grep alice
# alice:$6$rounds=656000$salt$hash:18000:0:99999:7:::

# /etc/group - информация о группах
cat /etc/group | grep docker
# docker:x:999:alice,bob
#   ↑    ↑ ↑   ↑
# группа пароль GID участники

# /etc/gshadow - пароли групп
sudo cat /etc/gshadow | grep docker
```

## Глава 11: Управление группами и система sudo

### Управление группами

**Создание и настройка групп:**

```bash
# Создание группы
sudo groupadd developers                   # создать группу
sudo groupadd -g 2000 webdev               # создать с конкретным GID

# Добавление пользователей в группы
sudo gpasswd -a alice developers           # добавить alice в developers
sudo gpasswd -A alice developers           # сделать alice администратором группы
sudo gpasswd -d alice developers           # удалить alice из developers

# Альтернативные команды
sudo usermod -aG developers alice          # добавить в группу
newgrp developers                          # временно переключиться на группу

# Информация о группах
groups alice                               # группы пользователя alice
id -G alice                                # GID групп пользователя alice
getent group developers                    # информация о группе
```

### Система sudo

sudo позволяет выполнять команды от имени других пользователей (обычно root):

```
Архитектура sudo:

Пользователь ──→ sudo ──→ Проверка /etc/sudoers ──→ Выполнение команды
     ↓              ↓              ↓                         ↓
   alice         команда      права доступа            от имени root
```

**Настройка sudo:**

```bash
# Редактирование sudoers (ТОЛЬКО через visudo!)
sudo visudo

# Примеры настроек sudoers:
# Полные права для группы sudo
%sudo ALL=(ALL:ALL) ALL

# Пользователь может перезапускать nginx без пароля
alice ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx, /bin/systemctl reload nginx

# Группа может управлять веб-сервером
%webadmin ALL=(ALL) NOPASSWD: /usr/sbin/nginx, /bin/systemctl * nginx

# Запрет на конкретные команды
alice ALL=(ALL) ALL, !/bin/su, !/usr/bin/passwd root

# Ограничение по времени
alice ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx
Defaults:alice timestamp_timeout=5

# Использование sudo с различными опциями
sudo -l                                    # показать разрешенные команды
sudo -u alice command                      # выполнить от имени alice
sudo -g developers command                 # выполнить от имени группы developers
sudo -i                                    # полная оболочка root
sudo -s                                    # оболочка с сохранением переменных
```

## Глава 12: SSH и удаленный доступ

### SSH — основной способ безопасного управления серверами

```
SSH соединение:

Клиент ──→ [Интернет] ──→ SSH сервер (порт 22) ──→ Оболочка пользователя
   ↓                           ↓                        ↓
 ssh alice@server         sshd процесс              bash/zsh/fish
   ↓                           ↓                        ↓
Аутентификация            Проверка ключей          Выполнение команд
(пароль/ключ)             ~/.ssh/authorized_keys
```

### Генерация и управление SSH ключами

```bash
# Генерация ключевой пары
ssh-keygen -t rsa -b 4096 -C "alice@company.com"
ssh-keygen -t ed25519 -C "alice@company.com"      # более современный алгоритм

# Ключи сохраняются в ~/.ssh/:
# ~/.ssh/id_rsa      - приватный ключ (НИКОГДА не передавать!)
# ~/.ssh/id_rsa.pub  - публичный ключ (можно передавать)

# Копирование публичного ключа на сервер
ssh-copy-id alice@server.com                      # автоматическое копирование
ssh-copy-id -i ~/.ssh/specific_key.pub alice@server.com

# Ручное копирование
cat ~/.ssh/id_rsa.pub | ssh alice@server.com "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Настройка прав доступа на сервере
chmod 700 ~/.ssh                                  # права на папку .ssh
chmod 600 ~/.ssh/authorized_keys                  # права на файл ключей
```

### Подключение и настройка SSH

**Подключение по SSH:**

```bash
# Базовые подключения
ssh alice@server.com                              # подключение с паролем/ключом
ssh -p 2222 alice@server.com                      # нестандартный порт
ssh -i ~/.ssh/specific_key alice@server.com       # конкретный ключ

# Выполнение команд удаленно
ssh alice@server.com "ls -la"                     # выполнить команду и выйти
ssh alice@server.com "cd /var/log && tail -f nginx/access.log"

# Туннелирование (проброс портов)
ssh -L 8080:localhost:80 alice@server.com         # локальный порт 8080 -> удаленный 80
ssh -R 9090:localhost:3000 alice@server.com       # удаленный порт 9090 -> локальный 3000
ssh -D 1080 alice@server.com                      # SOCKS proxy через SSH

# Передача файлов
scp file.txt alice@server.com:/home/alice/         # копировать файл на сервер
scp -r folder/ alice@server.com:/home/alice/       # копировать папку
scp alice@server.com:/var/log/app.log ./           # скачать файл с сервера

# rsync для синхронизации
rsync -avz folder/ alice@server.com:/backup/       # синхронизация с сжатием
rsync -avz --delete local/ remote:/backup/         # с удалением лишних файлов
```

**Конфигурация SSH клиента:**

```bash
# ~/.ssh/config - конфигурация клиента
cat > ~/.ssh/config << EOF
# Сервер разработки
Host dev
    HostName dev.company.com
    User alice
    Port 2222
    IdentityFile ~/.ssh/dev_key

# Продакшн сервер
Host prod  
    HostName prod.company.com
    User deploy
    IdentityFile ~/.ssh/prod_key
    StrictHostKeyChecking yes

# Все серверы компании
Host *.company.com
    User alice
    Compression yes
    ServerAliveInterval 60
EOF

chmod 600 ~/.ssh/config

# Теперь можно подключаться просто:
ssh dev                                            # вместо ssh -p 2222 alice@dev.company.com
ssh prod                                           # вместо ssh deploy@prod.company.com
```

**Настройка SSH сервера:**

```bash
# /etc/ssh/sshd_config - основные настройки безопасности
sudo nano /etc/ssh/sshd_config

# Рекомендуемые настройки безопасности:
Port 2222                                         # нестандартный порт
PermitRootLogin no                                # запрет входа под root
PasswordAuthentication no                         # только ключи, без паролей
PubkeyAuthentication yes                          # разрешить ключи
AuthorizedKeysFile .ssh/authorized_keys           # файл с ключами
MaxAuthTries 3                                    # максимум попыток входа
ClientAliveInterval 300                           # тайм-аут неактивных соединений
ClientAliveCountMax 2                             # количество проверок

# Ограничение пользователей
AllowUsers alice bob                              # только эти пользователи
DenyUsers guest                                   # запретить guest
AllowGroups ssh-users                             # только группа ssh-users

# Применение настроек
sudo systemctl restart sshd
sudo systemctl status sshd                        # проверить статус
```

## Глава 13: Расширенные права доступа и аудит

### Access Control Lists (ACL)

ACL позволяют устанавливать более гибкие права доступа:

```bash
# Проверка поддержки ACL
mount | grep acl                                   # поддерживает ли файловая система ACL

# Просмотр ACL
getfacl file.txt
# # file: file.txt
# # owner: alice
# # group: developers
# user::rw-
# user:bob:r--
# group::r--
# mask::r--
# other::---

# Установка ACL
setfacl -m u:bob:rw file.txt                      # дать bob права на чтение и запись
setfacl -m g:developers:rx folder/                # дать группе права на чтение и выполнение
setfacl -m o::--- file.txt                        # запретить доступ остальным

# Рекурсивная установка ACL
setfacl -R -m u:bob:rw folder/                     # рекурсивно для всех файлов
setfacl -d -m u:bob:rw folder/                     # по умолчанию для новых файлов

# Удаление ACL
setfacl -x u:bob file.txt                          # удалить ACL для bob
setfacl -b file.txt                                # удалить все ACL

# Копирование ACL
getfacl source_file | setfacl --set-file=- target_file
```

### Аудит и мониторинг безопасности

**Мониторинг входов в систему:**

```bash
# Последние входы
last                                               # история входов
last alice                                         # входы конкретного пользователя
lastb                                              # неудачные попытки входа (требует sudo)

# Текущие сессии
who                                                # кто сейчас в системе
w                                                  # детальная информация
users                                              # просто список пользователей

# Анализ логов аутентификации
sudo tail -f /var/log/auth.log                    # мониторинг в реальном времени
sudo grep "Failed password" /var/log/auth.log     # неудачные попытки входа
sudo grep "Accepted" /var/log/auth.log            # успешные входы

# SSH специфичные логи
sudo grep "sshd" /var/log/auth.log | grep "Failed"
sudo grep "sshd" /var/log/auth.log | grep "Accepted"
```

**Мониторинг sudo активности:**

```bash
# Логи sudo
sudo tail -f /var/log/auth.log | grep sudo
sudo grep "sudo.*COMMAND" /var/log/auth.log       # выполненные команды через sudo

# Детальные логи sudo (если включено в sudoers)
# Defaults logfile="/var/log/sudo.log"
sudo tail -f /var/log/sudo.log
```

---

# Блок 4: Сетевые технологии

## Глава 14: Сетевая модель Linux

### Сетевой стек Linux

Linux реализует полный стек TCP/IP с богатыми возможностями настройки и мониторинга:

```
Сетевой стек Linux:

Приложение (HTTP, SSH, FTP, DNS)
    ↓
Сокеты (Berkeley Sockets API)
    ↓
Транспортный уровень (TCP, UDP, SCTP)
    ↓
Сетевой уровень (IP, ICMP, IGMP)
    ↓
Канальный уровень (Ethernet, WiFi)
    ↓
Физический уровень (кабель, радио)

Пример HTTP запроса:
Browser → socket(AF_INET, SOCK_STREAM) → TCP → IP → Ethernet → Network
```

### Сетевые интерфейсы и адресация

**Типы сетевых интерфейсов:**

```
Физические интерфейсы:
eth0, eth1    - Ethernet интерфейсы
wlan0, wlan1  - WiFi интерфейсы
enp0s3        - новое именование (en = ethernet, p0s3 = PCI slot)

Виртуальные интерфейсы:
lo            - loopback (127.0.0.1)
docker0       - Docker bridge
br0           - bridge интерфейс
tun0, tap0    - VPN туннели
veth*         - виртуальные ethernet пары
```

**Просмотр и настройка интерфейсов:**

```bash
# Современные команды (iproute2)
ip addr show                                       # все интерфейсы
ip addr show eth0                                  # конкретный интерфейс
ip link show                                       # физические свойства интерфейсов

# Пример вывода ip addr show:
# 2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP
#    link/ether 08:00:27:46:75:05 brd ff:ff:ff:ff:ff:ff
#    inet 192.168.1.100/24 brd 192.168.1.255 scope global dynamic eth0
#    inet6 fe80::a00:27ff:fe46:7505/64 scope link

# Классические команды (устаревшие, но всё ещё используются)
ifconfig                                           # все интерфейсы
ifconfig eth0                                      # конкретный интерфейс

# Управление интерфейсами
sudo ip link set eth0 up                           # включить интерфейс
sudo ip link set eth0 down                         # выключить интерфейс
sudo ip link set eth0 mtu 9000                     # изменить MTU

# Добавление/удаление IP адресов
sudo ip addr add 192.168.1.200/24 dev eth0        # добавить IP
sudo ip addr del 192.168.1.200/24 dev eth0        # удалить IP

# Псевдонимы интерфейсов (алиасы)
sudo ip addr add 192.168.1.201/24 dev eth0 label eth0:0
ifconfig eth0:0                                    # просмотр алиаса
```

## Глава 15: DNS и разрешение имён

### DNS преобразует человеко-читаемые доменные имена в IP-адреса

```
DNS запрос (example.com):

Клиент → /etc/hosts → локальный DNS кеш → DNS сервер → корневые серверы → .com серверы → example.com серверы
   ↓         ↓              ↓               ↓              ↓                ↓               ↓
локальная локальный      systemd        8.8.8.8     корневые NS      .com NS       example.com NS
проверка  файл hosts   resolved                                                           ↓
                                                                                    93.184.216.34
```

### Конфигурация DNS

```bash
# /etc/hosts - локальная таблица имён
cat /etc/hosts
127.0.0.1       localhost
192.168.1.10    database.local db
192.168.1.20    cache.local redis
192.168.1.30    app1.local app2.local

# /etc/resolv.conf - DNS серверы
cat /etc/resolv.conf
nameserver 8.8.8.8
nameserver 1.1.1.1
search company.local
options timeout:2 attempts:3

# systemd-resolved (современные Ubuntu)
systemd-resolve --status
resolvectl status                                  # новая команда

# Настройка DNS через systemd-resolved
sudo nano /etc/systemd/resolved.conf
[Resolve]
DNS=8.8.8.8 1.1.1.1
FallbackDNS=8.8.4.4 1.0.0.1
Domains=company.local
```

### Диагностика DNS

```bash
# nslookup - базовые DNS запросы
nslookup google.com
nslookup google.com 8.8.8.8                       # запрос к конкретному DNS серверу

# dig - мощный инструмент DNS диагностики
dig google.com                                     # A запись
dig google.com MX                                  # MX записи (почта)
dig google.com NS                                  # NS записи (имя серверы)
dig google.com TXT                                 # TXT записи
dig @8.8.8.8 google.com                           # запрос к конкретному серверу

# Подробный dig запрос
dig +trace google.com                              # полная трассировка DNS запроса
dig +short google.com                              # только IP адрес
dig -x 8.8.8.8                                     # обратный DNS (PTR запись)

# host - простой инструмент
host google.com                                    # базовый запрос
host -t MX google.com                              # MX записи
host -a google.com                                 # все записи

# Кеш DNS
sudo systemd-resolve --flush-caches                # очистить кеш DNS
sudo resolvectl flush-caches                       # новая команда

# Мониторинг DNS запросов
sudo tcpdump -i any port 53                        # захват DNS трафика
dig +trace google.com | grep -E "^[a-zA-Z]"        # трассировка DNS резолвинга
```

## Глава 16: Маршрутизация

### Таблица маршрутизации определяет, куда отправлять сетевые пакеты

```
Концепция маршрутизации:

Пакет → Проверка таблицы маршрутизации → Выбор интерфейса → Отправка

Таблица маршрутизации:
Сеть назначения    Шлюз          Интерфейс    Метрика
192.168.1.0/24     0.0.0.0       eth0         0          (локальная сеть)
10.0.0.0/16        192.168.1.10  eth0         10         (корпоративная сеть)
0.0.0.0/0          192.168.1.1   eth0         100        (маршрут по умолчанию)
```

### Просмотр и управление маршрутами

```bash
# Просмотр таблицы маршрутизации
ip route show                                      # современная команда
route -n                                           # классическая команда
netstat -rn                                        # альтернативный вывод

# Пример вывода ip route show:
# default via 192.168.1.1 dev eth0 proto dhcp metric 100
# 192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100 metric 100

# Добавление маршрутов
sudo ip route add 10.0.0.0/16 via 192.168.1.10    # статический маршрут
sudo ip route add default via 192.168.1.1          # маршрут по умолчанию
sudo ip route add 172.16.0.0/12 dev tun0           # через интерфейс

# Удаление маршрутов
sudo ip route del 10.0.0.0/16                      # удалить конкретный маршрут
sudo ip route del default                          # удалить маршрут по умолчанию

# Маршруты с метриками
sudo ip route add 0.0.0.0/0 via 192.168.1.1 metric 100    # основной шлюз
sudo ip route add 0.0.0.0/0 via 192.168.1.2 metric 200    # резервный шлюз
```

### Диагностика маршрутизации

```bash
# Трассировка маршрута
traceroute google.com                              # показать путь к google.com
tracepath google.com                               # альтернативная команда
mtr google.com                                     # интерактивная трассировка

# Пример вывода traceroute:
# 1  192.168.1.1 (192.168.1.1)  1.234 ms  1.123 ms  1.456 ms
# 2  10.0.0.1 (10.0.0.1)  15.234 ms  14.123 ms  16.456 ms
# 3  203.0.113.1 (203.0.113.1)  25.234 ms  24.123 ms  26.456 ms

# Проверка достижимости
ping -c 4 google.com                               # 4 пакета к google.com
ping -c 4 -I eth0 google.com                      # через конкретный интерфейс
ping6 -c 4 google.com                             # IPv6 ping

# Детальная диагностика
ip route get 8.8.8.8                              # как система маршрутизирует к 8.8.8.8
# 8.8.8.8 via 192.168.1.1 dev eth0 src 192.168.1.100 uid 1000
```

## Глава 17: Анализ сетевых соединений

### Просмотр активных соединений

```bash
# netstat - классический инструмент
netstat -tuln                                      # слушающие порты TCP/UDP
netstat -tun                                       # активные соединения
netstat -tulnp                                     # с процессами (нужен sudo для чужих процессов)

# Расшифровка флагов netstat:
# -t: TCP соединения
# -u: UDP соединения  
# -l: только слушающие порты
# -n: числовой вид (не резолвить имена)
# -p: показать PID/имя процесса

# ss - современная замена netstat
ss -tuln                                           # слушающие порты
ss -tun                                            # активные соединения
ss -tlnp                                           # слушающие TCP порты с процессами

# Пример вывода ss -tlnp:
# State    Recv-Q Send-Q Local Address:Port  Peer Address:Port Process
# LISTEN   0      128    0.0.0.0:22           0.0.0.0:*     users:(("sshd",pid=1234,fd=3))
# LISTEN   0      128    0.0.0.0:80           0.0.0.0:*     users:(("nginx",pid=5678,fd=6))

# Фильтрация соединений
ss state listening                                 # только слушающие
ss sport = :80                                     # порт 80
ss dport = :443                                    # удаленный порт 443
ss src 192.168.1.100                              # с конкретного IP
ss dst 8.8.8.8                                    # к конкретному IP

# Статистика соединений
ss -s                                              # общая статистика
# Total: 1024 (kernel 1536)
# TCP:   12 (estab 3, closed 5, orphaned 0, synrecv 0, timewait 4/0), ports 0
```

### lsof для сетевой диагностики

```bash
# Все сетевые соединения
sudo lsof -i                                       # все сетевые файлы
sudo lsof -i -P                                    # с портами вместо названий сервисов

# Фильтрация по протоколу
sudo lsof -i tcp                                   # только TCP
sudo lsof -i udp                                   # только UDP

# Фильтрация по порту
sudo lsof -i :80                                   # что использует порт 80
sudo lsof -i :443                                  # что использует порт 443
sudo lsof -i :22                                   # SSH соединения

# Фильтрация по процессу
sudo lsof -i -p 1234                              # сетевые соединения процесса 1234
sudo lsof -i -c nginx                              # соединения nginx

# Фильтрация по статусу
sudo lsof -i -s TCP:LISTEN                         # слушающие TCP порты
sudo lsof -i -s TCP:ESTABLISHED                    # установленные соединения

# Комбинированные фильтры
sudo lsof -i TCP:80 -s TCP:LISTEN                  # что слушает TCP порт 80
sudo lsof -i @192.168.1.100                       # соединения с конкретным IP
```

## Глава 18: Файрвол и фильтрация трафика

### Linux имеет встроенную систему фильтрации пакетов на уровне ядра

```
Архитектура Netfilter/iptables:

Сетевой пакет → Netfilter hooks → iptables правила → Решение (ACCEPT/DROP/REJECT)
       ↓              ↓                ↓                    ↓
   входящий/        NF_PRE_ROUTING   таблицы:         разрешить/
   исходящий/       NF_LOCAL_IN      - filter         блокировать/
   проходящий       NF_FORWARD       - nat            отклонить
                   NF_LOCAL_OUT      - mangle
                   NF_POST_ROUTING   - raw
```

### iptables - основы

```bash
# Просмотр текущих правил
sudo iptables -L                                   # все цепочки
sudo iptables -L INPUT                             # цепочка INPUT
sudo iptables -L -n                                # без резолвинга имён
sudo iptables -L -v                                # с статистикой
sudo iptables -L --line-numbers                    # с номерами правил

# Основные цепочки:
# INPUT   - входящий трафик для локальной системы
# OUTPUT  - исходящий трафик от локальной системы  
# FORWARD - проходящий трафик (маршрутизация)

# Базовые правила
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT           # разрешить SSH
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT           # разрешить HTTP
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT          # разрешить HTTPS
sudo iptables -A INPUT -i lo -j ACCEPT                       # разрешить loopback
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT  # связанные соединения

# Блокировка трафика
sudo iptables -A INPUT -s 192.168.1.100 -j DROP             # блокировать IP
sudo iptables -A INPUT -p tcp --dport 3306 -j DROP           # блокировать MySQL порт
sudo iptables -A INPUT -j DROP                               # блокировать всё остальное

# Удаление правил
sudo iptables -D INPUT 3                                     # удалить правило номер 3
sudo iptables -D INPUT -p tcp --dport 80 -j ACCEPT           # удалить конкретное правило
sudo iptables -F                                             # очистить все правила
sudo iptables -F INPUT                                       # очистить цепочку INPUT

# Политики по умолчанию
sudo iptables -P INPUT DROP                                  # запретить входящий трафик по умолчанию
sudo iptables -P OUTPUT ACCEPT                               # разрешить исходящий трафик
sudo iptables -P FORWARD DROP                                # запретить проходящий трафик
```

### UFW - упрощённый файрвол

```bash
# Включение UFW
sudo ufw enable                                              # включить файрвол
sudo ufw --force enable                                      # без подтверждения

# Базовые правила
sudo ufw allow ssh                                           # разрешить SSH
sudo ufw allow 22/tcp                                        # альтернативный синтаксис
sudo ufw allow 80/tcp                                        # HTTP
sudo ufw allow 443/tcp                                       # HTTPS
sudo ufw allow from 192.168.1.0/24                          # разрешить подсеть

# Запрет трафика
sudo ufw deny 3306                                           # заблокировать MySQL
sudo ufw deny from 203.0.113.100                            # заблокировать IP

# Сложные правила
sudo ufw allow from 192.168.1.0/24 to any port 3306         # MySQL только из локальной сети
sudo ufw allow in on eth0 to any port 80                    # HTTP только на eth0
sudo ufw limit ssh                                           # ограничить SSH (rate limiting)

# Управление правилами
sudo ufw status                                              # текущий статус
sudo ufw status numbered                                     # с номерами правил
sudo ufw delete 3                                            # удалить правило номер 3
sudo ufw reset                                               # сбросить все правила

# Логирование
sudo ufw logging on                                          # включить логирование
sudo ufw logging high                                        # детальное логирование
tail -f /var/log/ufw.log                                     # просмотр логов
```

## Глава 19: Мониторинг сетевого трафика

### tcpdump - захват пакетов

```bash
# Базовый захват
sudo tcpdump -i eth0                                         # захват на интерфейсе eth0
sudo tcpdump -i any                                          # захват на всех интерфейсах
sudo tcpdump -c 100                                          # захватить 100 пакетов

# Фильтрация по протоколу
sudo tcpdump -i eth0 tcp                                     # только TCP
sudo tcpdump -i eth0 udp                                     # только UDP
sudo tcpdump -i eth0 icmp                                    # только ICMP (ping)

# Фильтрация по порту
sudo tcpdump -i eth0 port 80                                 # трафик на порт 80
sudo tcpdump -i eth0 src port 22                             # исходящий с порта 22
sudo tcpdump -i eth0 dst port 443                            # входящий на порт 443

# Фильтрация по IP
sudo tcpdump -i eth0 host 8.8.8.8                           # трафик с/на 8.8.8.8
sudo tcpdump -i eth0 src 192.168.1.100                      # от конкретного IP
sudo tcpdump -i eth0 dst 192.168.1.200                      # к конкретному IP
sudo tcpdump -i eth0 net 192.168.1.0/24                     # трафик подсети

# Комбинированные фильтры
sudo tcpdump -i eth0 tcp and port 80                        # TCP на порт 80
sudo tcpdump -i eth0 host 8.8.8.8 and port 53               # DNS к 8.8.8.8
sudo tcpdump -i eth0 '(tcp and port 80) or (tcp and port 443)'  # HTTP/HTTPS

# Детальный анализ
sudo tcpdump -i eth0 -v                                     # подробный вывод
sudo tcpdump -i eth0 -x                                     # hex дамп
sudo tcpdump -i eth0 -X                                     # hex + ASCII
sudo tcpdump -i eth0 -A                                     # ASCII содержимое

# Сохранение в файл
sudo tcpdump -i eth0 -w capture.pcap                        # сохранить в файл
sudo tcpdump -r capture.pcap                                # читать из файла
sudo tcpdump -r capture.pcap host 192.168.1.100             # фильтрация при чтении
```

### Анализ производительности сети

```bash
# iftop - топ соединений по трафику
sudo iftop                                                  # интерактивный мониторинг
sudo iftop -i eth0                                          # на конкретном интерфейсе
sudo iftop -n                                               # без резолвинга имён

# nethogs - топ процессов по сетевому трафику
sudo nethogs                                                # все интерфейсы
sudo nethogs eth0                                           # конкретный интерфейс

# iperf3 - тестирование пропускной способности
# На сервере:
iperf3 -s                                                   # режим сервера
# На клиенте:
iperf3 -c server_ip                                         # тест к серверу
iperf3 -c server_ip -t 30                                   # тест 30 секунд
iperf3 -c server_ip -P 4                                    # 4 параллельных потока

# nload - простой мониторинг трафика
nload                                                       # график трафика
nload eth0                                                  # конкретный интерфейс

# bwm-ng - мониторинг пропускной способности
bwm-ng                                                      # простой вывод
bwm-ng -o csv                                               # CSV формат
```

---

# Блок 5: Веб-серверы и базы данных

## Глава 20: Nginx - глубокое погружение

### Nginx — высокопроизводительный веб-сервер, reverse proxy и load balancer

```
Архитектура Nginx:

Master процесс (управление)
├── Worker процесс 1 (обработка запросов)
├── Worker процесс 2 (обработка запросов)
└── Worker процесс N (обработка запросов)

Каждый worker может обрабатывать тысячи соединений одновременно
благодаря event-driven архитектуре (epoll/kqueue).
```

### Установка и базовая настройка

```bash
# Установка (Ubuntu/Debian)
sudo apt update
sudo apt install nginx

# Установка (CentOS/RHEL)
sudo yum install epel-release
sudo yum install nginx

# Управление сервисом
sudo systemctl start nginx                                  # запуск
sudo systemctl stop nginx                                   # остановка
sudo systemctl restart nginx                                # перезапуск
sudo systemctl reload nginx                                 # перезагрузка конфигурации
sudo systemctl enable nginx                                 # автозапуск
sudo systemctl status nginx                                 # статус

# Проверка конфигурации
sudo nginx -t                                               # проверить синтаксис
sudo nginx -T                                               # показать полную конфигурацию
nginx -v                                                    # версия nginx
nginx -V                                                    # версия и модули
```

### Структура конфигурации Nginx

```
/etc/nginx/
├── nginx.conf                 (главная конфигурация)
├── conf.d/                    (дополнительные конфигурации)
├── sites-available/           (доступные сайты)
├── sites-enabled/             (активные сайты - симлинки)
├── modules-available/         (доступные модули)
├── modules-enabled/           (активные модули)
└── snippets/                  (переиспользуемые фрагменты)
```

### Главная конфигурация nginx.conf

```nginx
# /etc/nginx/nginx.conf
user www-data;                                              # пользователь для worker процессов
worker_processes auto;                                      # количество worker процессов (= CPU cores)
pid /run/nginx.pid;                                         # файл с PID главного процесса

# Events блок - настройки соединений
events {
    worker_connections 1024;                                # максимум соединений на worker
    use epoll;                                               # метод обработки событий (Linux)
    multi_accept on;                                         # принимать несколько соединений сразу
}

# HTTP блок - основные настройки
http {
    ##
    # Базовые настройки
    ##
    sendfile on;                                             # эффективная передача файлов
    tcp_nopush on;                                           # оптимизация TCP пакетов
    tcp_nodelay on;                                          # отключить алгоритм Nagle
    keepalive_timeout 65;                                    # время жизни keep-alive соединений
    types_hash_max_size 2048;                              # размер хеш-таблицы MIME типов
    
    include /etc/nginx/mime.types;                          # MIME типы файлов
    default_type application/octet-stream;                  # тип по умолчанию

    ##
    # SSL настройки
    ##
    ssl_protocols TLSv1.2 TLSv1.3;                         # разрешённые SSL протоколы
    ssl_prefer_server_ciphers on;                           # предпочитать серверные шифры
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384;

    ##
    # Логирование
    ##
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    
    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log;

    ##
    # Gzip сжатие
    ##
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;

    ##
    # Виртуальные хосты
    ##
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

### Конфигурация виртуального хоста

```nginx
# /etc/nginx/sites-available/myapp
server {
    listen 80;                                              # слушать порт 80
    listen [::]:80;                                         # IPv6
    server_name myapp.example.com www.myapp.example.com;    # доменные имена
    
    # Перенаправление на HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;                                   # HTTPS с HTTP/2
    listen [::]:443 ssl http2;                              # IPv6 HTTPS
    server_name myapp.example.com www.myapp.example.com;
    
    # SSL сертификаты
    ssl_certificate /etc/ssl/certs/myapp.crt;
    ssl_certificate_key /etc/ssl/private/myapp.key;
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozTLS:10m;
    ssl_session_tickets off;
    
    # Корневая папка
    root /var/www/myapp/public;
    index index.html index.htm index.php;
    
    # Общие настройки безопасности
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    # Основная локация
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    # PHP обработка
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;
    }
    
    # Статические файлы с кешированием
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1M;
        add_header Cache-Control "public, immutable";
        add_header X-Content-Type-Options nosniff;
    }
    
    # Запретить доступ к системным файлам
    location ~ /\. {
        deny all;
    }
    
    location ~ ^/(\.user.ini|\.htaccess|\.git|\.svn|\.project|LICENSE|README.md)$ {
        deny all;
    }
    
    # Логи для конкретного сайта
    access_log /var/log/nginx/myapp_access.log;
    error_log /var/log/nginx/myapp_error.log;
}
```

### Nginx как reverse proxy

```nginx
# /etc/nginx/sites-available/api-proxy
upstream backend {
    least_conn;                                             # алгоритм балансировки
    server 127.0.0.1:3000 weight=3;                        # сервер с весом 3
    server 127.0.0.1:3001 weight=2;                        # сервер с весом 2
    server 127.0.0.1:3002 backup;                          # резервный сервер
}

server {
    listen 80;
    server_name api.example.com;
    
    location / {
        proxy_pass http://backend;
        
        # Заголовки для backend
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Port $server_port;
        
        # Настройки таймаутов
        proxy_connect_timeout 30s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
        
        # Буферизация
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
        proxy_busy_buffers_size 8k;
        
        # Кеширование (опционально)
        proxy_cache backend_cache;
        proxy_cache_valid 200 302 10m;
        proxy_cache_valid 404 1m;
        proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
    }
    
    # WebSocket поддержка
    location /ws {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
    
    # Health check endpoint
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}

# Кеш зона
proxy_cache_path /var/cache/nginx/backend levels=1:2 keys_zone=backend_cache:10m inactive=60m;
```

## Глава 21: SSL/TLS и Let's Encrypt

### Получение бесплатного SSL сертификата

```bash
# Установка Certbot
sudo apt install certbot python3-certbot-nginx             # Ubuntu/Debian
sudo yum install certbot python3-certbot-nginx             # CentOS/RHEL

# Получение сертификата для Nginx
sudo certbot --nginx -d example.com -d www.example.com

# Ручное получение сертификата
sudo certbot certonly --webroot -w /var/www/html -d example.com

# Получение wildcard сертификата (требует DNS challenge)
sudo certbot certonly --manual --preferred-challenges dns -d *.example.com

# Автоматическое обновление
sudo crontab -e
# Добавить строку:
0 12 * * * /usr/bin/certbot renew --quiet && systemctl reload nginx

# Тестирование обновления
sudo certbot renew --dry-run

# Просмотр сертификатов
sudo certbot certificates

# Отзыв сертификата
sudo certbot revoke --cert-path /etc/letsencrypt/live/example.com/cert.pem
```

### Настройка SSL в Nginx

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    # Пути к сертификатам Let's Encrypt
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
    # Современные SSL настройки
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-CHACHA20-POLY1305;
    ssl_prefer_server_ciphers off;
    
    # SSL оптимизация
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozTLS:10m;
    ssl_session_tickets off;
    
    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
    
    # Другие заголовки безопасности
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'";
}
```

## Глава 22: PostgreSQL - администрирование и оптимизация

### Установка и настройка PostgreSQL

```bash
# Установка (Ubuntu/Debian)
sudo apt update
sudo apt install postgresql postgresql-contrib

# Установка (CentOS/RHEL)
sudo yum install postgresql-server postgresql-contrib
sudo postgresql-setup initdb

# Управление сервисом
sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo systemctl status postgresql

# Переключение на пользователя postgres
sudo -i -u postgres

# Создание базы данных и пользователя
createdb myapp
createuser --interactive myuser
# Ответить на вопросы:
# Shall the new role be a superuser? (y/n) n
# Shall the new role be allowed to create databases? (y/n) y
# Shall the new role be allowed to create more new roles? (y/n) n

# Установка пароля
psql
postgres=# \password myuser
postgres=# \q

# Альтернативный способ создания пользователя
psql -c "CREATE USER myuser WITH PASSWORD 'secretpassword';"
psql -c "CREATE DATABASE myapp OWNER myuser;"
psql -c "GRANT ALL PRIVILEGES ON DATABASE myapp TO myuser;"
```

### Конфигурация PostgreSQL

```bash
# Основные файлы конфигурации
/etc/postgresql/13/main/postgresql.conf              # основная конфигурация
/etc/postgresql/13/main/pg_hba.conf                  # аутентификация и доступ

# Редактирование postgresql.conf
sudo nano /etc/postgresql/13/main/postgresql.conf
```

```ini
# postgresql.conf - основные настройки
listen_addresses = '*'                                     # слушать все интерфейсы
port = 5432                                                # порт PostgreSQL

# Память
shared_buffers = 256MB                                     # разделяемая память
effective_cache_size = 1GB                                 # кеш ОС
maintenance_work_mem = 64MB                                # память для обслуживания
work_mem = 4MB                                             # память для запросов

# Соединения
max_connections = 200                                      # максимум соединений
superuser_reserved_connections = 3                        # резерв для суперпользователя

# WAL (Write-Ahead Logging)
wal_level = replica                                        # уровень логирования WAL
max_wal_size = 1GB                                         # максимальный размер WAL
min_wal_size = 80MB                                        # минимальный размер WAL

# Логирование
logging_collector = on                                     # включить сборщик логов
log_directory = 'log'                                      # папка логов
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'           # формат имени файла
log_min_messages = warning                                 # минимальный уровень логов
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '

# Автовакуум
autovacuum = on                                            # включить автовакуум
autovacuum_max_workers = 3                                 # количество worker процессов
```

### Настройка доступа pg_hba.conf

```bash
sudo nano /etc/postgresql/13/main/pg_hba.conf
```

```
# pg_hba.conf - контроль доступа
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# Локальные соединения
local   all             postgres                                peer
local   all             all                                     md5

# IPv4 локальные соединения
host    all             all             127.0.0.1/32            md5
host    all             all             192.168.1.0/24          md5    # локальная сеть

# IPv6 локальные соединения  
host    all             all             ::1/128                 md5

# Удаленные соединения (осторожно!)
host    myapp           myuser          0.0.0.0/0               md5    # только для myapp

# SSL соединения
hostssl myapp           myuser          0.0.0.0/0               md5

# После изменений перезагрузить конфигурацию
sudo systemctl reload postgresql
```

### Работа с PostgreSQL

```bash
# Подключение к базе данных
psql -h localhost -U myuser -d myapp
psql -h localhost -U postgres                             # как суперпользователь

# Основные команды psql
\l                                                         # список баз данных
\c myapp                                                   # подключиться к базе myapp
\dt                                                        # список таблиц
\d table_name                                              # структура таблицы
\du                                                        # список пользователей
\q                                                         # выйти

# SQL запросы
SELECT version();                                          # версия PostgreSQL
SELECT current_database();                                 # текущая база данных
SELECT current_user;                                       # текущий пользователь

# Создание таблицы
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

# Вставка данных
INSERT INTO users (username, email, password_hash) 
VALUES ('alice', 'alice@example.com', 'hashed_password');

# Создание индексов
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
```

### Backup и восстановление PostgreSQL

```bash
# Backup одной базы данных
pg_dump -h localhost -U myuser myapp > myapp_backup.sql
pg_dump -h localhost -U myuser -Fc myapp > myapp_backup.dump    # сжатый формат

# Backup всех баз данных
pg_dumpall -h localhost -U postgres > all_databases.sql

# Backup с сжатием
pg_dump -h localhost -U myuser myapp | gzip > myapp_backup.sql.gz

# Восстановление из SQL файла
psql -h localhost -U myuser myapp < myapp_backup.sql

# Восстановление из dump файла
pg_restore -h localhost -U myuser -d myapp myapp_backup.dump

# Восстановление с пересозданием базы
dropdb myapp
createdb myapp
psql -h localhost -U myuser myapp < myapp_backup.sql

# Автоматический backup через cron
sudo crontab -e
# Добавить строку:
0 2 * * * pg_dump -h localhost -U myuser myapp | gzip > /backup/myapp_$(date +\%Y\%m\%d).sql.gz
```

---

# Блок 6: Контейнеризация

## Глава 23: Docker - глубокое понимание

### Архитектура Docker

Docker революционизировал развертывание приложений, обеспечивая согласованность между средами разработки и production.

```
Архитектура Docker:

Docker Client (docker CLI)
        ↓
Docker Daemon (dockerd)
        ↓
┌─────────────────────────────┐
│        Docker Host          │
│  ┌─────────┐ ┌─────────┐    │
│  │Container│ │Container│    │
│  │    A    │ │    B    │    │
│  └─────────┘ └─────────┘    │
│       ↑           ↑         │
│   ┌─────────┐ ┌─────────┐    │
│   │ Image A │ │ Image B │    │
│   └─────────┘ └─────────┘    │
└─────────────────────────────┘

Контейнеры используют общее ядро Linux, но изолированы через:
- Namespaces (изоляция процессов, сети, файловой системы)
- Cgroups (ограничение ресурсов)
- Union FS (слоистая файловая система)
```

### Установка Docker

```bash
# Удаление старых версий
sudo apt remove docker docker-engine docker.io containerd runc

# Установка зависимостей
sudo apt update
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release

# Добавление официального GPG ключа Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Добавление репозитория
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Установка Docker Engine
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io

# Добавление пользователя в группу docker
sudo usermod -aG docker $USER
newgrp docker                                              # применить группу без перелогина

# Проверка установки
docker --version
docker run hello-world
```

### Основные команды Docker

```bash
# Работа с образами
docker images                                             # список локальных образов
docker search nginx                                       # поиск образов в Docker Hub
docker pull nginx:latest                                  # скачать образ
docker pull nginx:1.21-alpine                            # конкретная версия
docker rmi nginx:latest                                   # удалить образ
docker image prune                                        # удалить неиспользуемые образы

# Запуск контейнеров
docker run nginx                                          # запустить в foreground
docker run -d nginx                                       # запустить в background (daemon)
docker run -d --name webserver nginx                      # с именем
docker run -d -p 8080:80 nginx                           # проброс портов
docker run -d -p 127.0.0.1:8080:80 nginx                # привязка к localhost
docker run -d -v /host/path:/container/path nginx         # монтирование volume

# Управление контейнерами
docker ps                                                 # запущенные контейнеры
docker ps -a                                              # все контейнеры
docker stop container_id                                  # остановить контейнер
docker start container_id                                 # запустить остановленный
docker restart container_id                               # перезапустить
docker rm container_id                                    # удалить контейнер
docker rm -f container_id                                 # принудительно удалить

# Взаимодействие с контейнерами
docker exec -it container_id bash                         # войти в контейнер
docker exec container_id ls -la                           # выполнить команду
docker logs container_id                                  # логи контейнера
docker logs -f container_id                               # следить за логами
docker logs --tail 100 container_id                       # последние 100 строк

# Информация о контейнерах
docker inspect container_id                               # детальная информация
docker stats                                              # статистика ресурсов
docker top container_id                                   # процессы в контейнере
```

## Глава 24: Создание Docker образов

### Dockerfile - описание образа

```dockerfile
# Комментарий: Dockerfile для Python веб-приложения

# Базовый образ
FROM python:3.9-slim

# Метаданные
LABEL maintainer="developer@company.com"
LABEL version="1.0"
LABEL description="My Python Web Application"

# Переменные окружения
ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1
ENV APP_HOME=/app

# Создание пользователя (безопасность)
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Установка системных зависимостей
RUN apt-get update && apt-get install -y \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Создание рабочей директории
WORKDIR $APP_HOME

# Копирование файла зависимостей
COPY requirements.txt .

# Установка Python зависимостей
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# Копирование исходного кода
COPY . .

# Изменение владельца файлов
RUN chown -R appuser:appuser $APP_HOME

# Переключение на непривилегированного пользователя
USER appuser

# Открытие порта
EXPOSE 8000

# Проверка здоровья
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# Команда по умолчанию
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:application"]
```

### Многоэтапная сборка (Multi-stage build)

```dockerfile
# Этап 1: Сборка
FROM node:16-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

# Этап 2: Production
FROM nginx:alpine

# Копирование только необходимых файлов из предыдущего этапа
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Сборка и тегирование образов

```bash
# Простая сборка
docker build -t myapp .

# Сборка с тегом
docker build -t myapp:v1.0 .
docker build -t myapp:latest .

# Сборка с контекстом
docker build -t myapp -f Dockerfile.prod .

# Сборка с аргументами
docker build --build-arg ENV=production -t myapp .

# Пример Dockerfile с аргументами:
# ARG ENV=development
# ENV ENVIRONMENT=$ENV

# Просмотр истории образа
docker history myapp:latest

# Анализ размера слоев
docker image inspect myapp:latest
```

## Глава 25: Docker Compose - управление многоконтейнерными приложениями

### docker-compose.yml для веб-приложения

```yaml
version: '3.8'

services:
  # Веб-приложение
  web:
    build: 
      context: .
      dockerfile: Dockerfile
      args:
        - ENV=production
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/myapp
      - REDIS_URL=redis://redis:6379/0
      - DEBUG=false
    volumes:
      - ./app:/app                                         # для разработки
      - static_volume:/app/static                          # статические файлы
      - media_volume:/app/media                            # загруженные файлы
    depends_on:
      - db
      - redis
    restart: unless-stopped
    networks:
      - frontend
      - backend

  # База данных PostgreSQL
  db:
    image: postgres:13-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_INITDB_ARGS: "--encoding=UTF-8"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    ports:
      - "5432:5432"                                        # только для разработки
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    restart: unless-stopped
    networks:
      - backend

  # Redis для кеширования
  redis:
    image: redis:6-alpine
    command: redis-server --appendonly yes --requirepass mypassword
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    restart: unless-stopped
    networks:
      - backend

  # Nginx reverse proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
      - static_volume:/var/www/static:ro
      - media_volume:/var/www/media:ro
    depends_on:
      - web
    restart: unless-stopped
    networks:
      - frontend

# Volumes для постоянного хранения данных
volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local
  static_volume:
    driver: local
  media_volume:
    driver: local

# Сети для изоляции сервисов
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

### Команды Docker Compose

```bash
# Основные команды
docker-compose up                                          # запуск в foreground
docker-compose up -d                                       # запуск в background
docker-compose up --build                                  # пересобрать образы
docker-compose up --scale web=3                            # масштабирование

# Управление сервисами
docker-compose start                                       # запустить остановленные сервисы
docker-compose stop                                        # остановить сервисы
docker-compose restart                                     # перезапустить сервисы
docker-compose pause web                                   # приостановить сервис
docker-compose unpause web                                 # возобновить сервис

# Просмотр информации
docker-compose ps                                          # статус сервисов
docker-compose logs                                        # логи всех сервисов
docker-compose logs -f web                                 # логи конкретного сервиса
docker-compose top                                         # процессы в сервисах

# Выполнение команд
docker-compose exec web bash                               # войти в контейнер web
docker-compose exec db psql -U user -d myapp               # подключиться к БД
docker-compose run --rm web python manage.py migrate      # разовая команда

# Очистка
docker-compose down                                        # остановить и удалить контейнеры
docker-compose down -v                                     # удалить также volumes
docker-compose down --rmi all                              # удалить также образы

# Конфигурация
docker-compose config                                      # проверить конфигурацию
docker-compose config --services                           # список сервисов
```

## Глава 26: Docker сети и volumes

### Управление сетями Docker

```bash
# Просмотр сетей
docker network ls

# Создание сети
docker network create mynetwork                           # bridge сеть
docker network create --driver bridge mynetwork           # явно указать driver
docker network create --driver overlay mynetwork          # для Docker Swarm

# Подключение контейнера к сети
docker run -d --name web --network mynetwork nginx
docker network connect mynetwork existing_container

# Отключение от сети
docker network disconnect mynetwork container_name

# Информация о сети
docker network inspect mynetwork

# Удаление сети
docker network rm mynetwork
docker network prune                                      # удалить неиспользуемые сети
```

### Типы сетей Docker

```
bridge (по умолчанию)
├── Изолированная сеть для контейнеров на одном хосте
├── Контейнеры могут общаться по имени
└── NAT для доступа во внешнюю сеть

host
├── Контейнер использует сеть хоста напрямую
├── Нет изоляции сети
└── Максимальная производительность

none
├── Контейнер без сетевого интерфейса
├── Полная изоляция
└── Для специальных случаев

overlay
├── Для Docker Swarm кластера
├── Связывает контейнеры на разных хостах
└── Шифрование трафика между хостами
```

### Управление volumes

```bash
# Просмотр volumes
docker volume ls

# Создание volume
docker volume create myvolume
docker volume create --driver local --opt type=nfs myvolume

# Использование volume
docker run -v myvolume:/data nginx
docker run --mount source=myvolume,target=/data nginx      # новый синтаксис

# Информация о volume
docker volume inspect myvolume

# Удаление volume
docker volume rm myvolume
docker volume prune                                       # удалить неиспользуемые volumes

# Backup volume
docker run --rm -v myvolume:/source -v $(pwd):/backup alpine tar czf /backup/myvolume.tar.gz -C /source .

# Restore volume
docker run --rm -v myvolume:/target -v $(pwd):/backup alpine tar xzf /backup/myvolume.tar.gz -C /target
```

---

# Блок 7: Автоматизация и DevOps

## Глава 27: Bash скриптинг для автоматизации

### Bash скрипты — основа автоматизации в Linux

Они позволяют автоматизировать рутинные задачи и создавать сложные deployment пайплайны.

### Структура bash скрипта

```bash
#!/bin/bash
# Deployment script for web application
# Usage: ./deploy.sh [environment] [version]
# Example: ./deploy.sh production v1.2.3

# Настройка обработки ошибок
set -euo pipefail                                          # остановка при ошибке, неопределенных переменных
IFS=\n\t'                                                # безопасное разделение строк

# Константы
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly APP_NAME="mywebapp"
readonly LOG_FILE="/var/log/${APP_NAME}/deploy.log"
readonly BACKUP_DIR="/backup/${APP_NAME}"
readonly DEPLOY_USER="deploy"

# Функция логирования
log() {
    local level=$1
    shift
    local message="$*"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[${timestamp}] [${level}] ${message}" | tee -a "${LOG_FILE}"
}

# Функция проверки зависимостей
check_dependencies() {
    local deps=("docker" "docker-compose" "git" "curl")
    
    for dep in "${deps[@]}"; do
        if ! command -v "$dep" &> /dev/null; then
            log "ERROR" "Dependency $dep is not installed"
            exit 1
        fi
    done
    
    log "INFO" "All dependencies are satisfied"
}

# Функция создания backup
create_backup() {
    local backup_name="${APP_NAME}_$(date +%Y%m%d_%H%M%S)"
    local backup_path="${BACKUP_DIR}/${backup_name}"
    
    log "INFO" "Creating backup: ${backup_name}"
    
    mkdir -p "${backup_path}"
    
    # Backup кода
    if [ -d "/opt/${APP_NAME}" ]; then
        cp -r "/opt/${APP_NAME}" "${backup_path}/code"
    fi
    
    # Backup базы данных
    if docker ps | grep -q "${APP_NAME}_db"; then
        docker exec "${APP_NAME}_db" pg_dump -U postgres myapp > "${backup_path}/database.sql"
    fi
    
    # Создание архива
    tar -czf "${backup_path}.tar.gz" -C "${BACKUP_DIR}" "${backup_name}"
    rm -rf "${backup_path}"
    
    log "INFO" "Backup created: ${backup_path}.tar.gz"
    
    # Очистка старых backup (старше 30 дней)
    find "${BACKUP_DIR}" -name "*.tar.gz" -mtime +30 -delete
}

# Функция развертывания
deploy_application() {
    local environment=$1
    local version=$2
    local app_dir="/opt/${APP_NAME}"
    
    log "INFO" "Starting deployment of ${APP_NAME} ${version} to ${environment}"
    
    # Переход в директорию приложения
    cd "${app_dir}"
    
    # Обновление кода
    log "INFO" "Updating code to version ${version}"
    git fetch origin
    git checkout "${version}"
    git pull origin "${version}"
    
    # Обновление зависимостей
    if [ -f "requirements.txt" ]; then
        log "INFO" "Updating Python dependencies"
        pip install -r requirements.txt
    fi
    
    if [ -f "package.json" ]; then
        log "INFO" "Updating Node.js dependencies"
        npm ci --production
    fi
    
    # Сборка Docker образов
    log "INFO" "Building Docker images"
    docker-compose build --no-cache
    
    # Запуск миграций базы данных
    log "INFO" "Running database migrations"
    docker-compose run --rm web python manage.py migrate
    
    # Перезапуск сервисов
    log "INFO" "Restarting services"
    docker-compose down
    docker-compose up -d
    
    # Проверка здоровья сервисов
    health_check
    
    log "INFO" "Deployment completed successfully"
}

# Функция проверки здоровья
health_check() {
    local max_attempts=30
    local attempt=1
    
    log "INFO" "Performing health check"
    
    while [ $attempt -le $max_attempts ]; do
        if curl -f -s http://localhost:8000/health > /dev/null; then
            log "INFO" "Health check passed"
            return 0
        fi
        
        log "WARN" "Health check failed, attempt ${attempt}/${max_attempts}"
        sleep 10
        ((attempt++))
    done
    
    log "ERROR" "Health check failed after ${max_attempts} attempts"
    return 1
}

# Главная функция
main() {
    local environment=${1:-staging}
    local version=${2:-main}
    
    # Проверка прав доступа
    if [ "$(whoami)" != "${DEPLOY_USER}" ]; then
        log "ERROR" "This script must be run as ${DEPLOY_USER} user"
        exit 1
    fi
    
    # Создание директории для логов
    mkdir -p "$(dirname "${LOG_FILE}")"
    
    log "INFO" "Starting deployment process"
    
    # Проверка зависимостей
    check_dependencies
    
    # Создание backup
    create_backup
    
    # Развертывание
    if deploy_application "${environment}" "${version}"; then
        log "INFO" "Deployment completed successfully"
    else
        log "ERROR" "Deployment failed"
        exit 1
    fi
}

# Обработка сигналов
trap 'log "ERROR" "Script interrupted"; exit 1' INT TERM

# Запуск главной функции
main "$@"
```

### Полезные bash функции и паттерны

```bash
# Проверка существования файла/директории
if [ -f "/path/to/file" ]; then
    echo "File exists"
fi

if [ -d "/path/to/directory" ]; then
    echo "Directory exists"
fi

# Работа с массивами
servers=("web1.example.com" "web2.example.com" "web3.example.com")

for server in "${servers[@]}"; do
    echo "Deploying to ${server}"
    ssh "${server}" "systemctl restart nginx"
done

# Параллельное выполнение
for server in "${servers[@]}"; do
    ssh "${server}" "systemctl restart nginx" &
done
wait  # дождаться завершения всех фоновых задач

# Работа с файлами построчно
while IFS= read -r line; do
    echo "Processing: ${line}"
done < input.txt

# Генерация случайных данных
random_password=$(openssl rand -base64 32)
random_id=$(uuidgen)

# Работа с датами
current_date=$(date '+%Y-%m-%d')
timestamp=$(date '+%Y-%m-%d %H:%M:%S')
unix_timestamp=$(date +%s)

# Валидация email
validate_email() {
    local email=$1
    if [[ $email =~ ^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$ ]]; then
        return 0
    else
        return 1
    fi
}

# Retry функция
retry() {
    local max_attempts=$1
    shift
    local cmd=("$@")
    local attempt=1
    
    while [ $attempt -le $max_attempts ]; do
        if "${cmd[@]}"; then
            return 0
        fi
        
        echo "Attempt $attempt failed, retrying..."
        ((attempt++))
        sleep 5
    done
    
    echo "All $max_attempts attempts failed"
    return 1
}

# Использование: retry 3 curl -f http://example.com
```

## Глава 28: Управление задачами с Cron

### Cron позволяет автоматически выполнять задачи по расписанию

```bash
# Редактирование crontab
crontab -e                                                # для текущего пользователя
sudo crontab -e                                           # для root
sudo crontab -u username -e                               # для конкретного пользователя

# Просмотр crontab
crontab -l                                                # текущий пользователь
sudo crontab -l                                           # root
sudo crontab -u username -l                               # конкретный пользователь

# Удаление crontab
crontab -r                                                # удалить все задачи

# Формат crontab:
# минута час день_месяца месяц день_недели команда
# 0-59   0-23    1-31      1-12    0-7     (0 и 7 = воскресенье)
```

### Примеры cron задач

```bash
# Backup базы данных каждый день в 2:00
0 2 * * * /usr/local/bin/backup-database.sh

# Очистка логов каждое воскресенье в 3:00
0 3 * * 0 find /var/log -name "*.log" -mtime +7 -delete

# Проверка сервисов каждые 5 минут
*/5 * * * * /usr/local/bin/check-services.sh

# Перезапуск сервиса каждую неделю
0 4 * * 1 systemctl restart myservice

# Отправка отчета каждый первый день месяца
0 9 1 * * /usr/local/bin/monthly-report.sh

# Мониторинг дискового пространства каждый час
0 * * * * df -h | grep -E '9[0-9]%|100%' && echo "Disk space warning" | mail -s "Disk Alert" admin@company.com

# Переменные окружения в crontab
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=admin@company.com

# Резервное копирование с ротацией
0 1 * * * /usr/local/bin/backup.sh && find /backup -name "*.tar.gz" -mtime +30 -delete

# Синхронизация времени
0 3 * * * ntpdate -s time.nist.gov

# Мониторинг SSL сертификатов
0 6 * * * /usr/local/bin/check-ssl-expiry.sh
```

### Скрипт проверки сервисов

```bash
#!/bin/bash
# /usr/local/bin/check-services.sh

SERVICES=("nginx" "postgresql" "redis" "docker")
ALERT_EMAIL="admin@company.com"
LOG_FILE="/var/log/service-monitor.log"

for service in "${SERVICES[@]}"; do
    if ! systemctl is-active --quiet "$service"; then
        echo "$(date): $service is not running" >> "$LOG_FILE"
        echo "$service is down on $(hostname)" | mail -s "Service Alert" "$ALERT_EMAIL"
        
        # Попытка перезапуска
        systemctl restart "$service"
        sleep 10
        
        if systemctl is-active --quiet "$service"; then
            echo "$(date): $service restarted successfully" >> "$LOG_FILE"
        else
            echo "$(date): Failed to restart $service" >> "$LOG_FILE"
        fi
    fi
done
```

## Глава 29: Ansible для управления конфигурацией

### Ansible позволяет автоматизировать настройку и управление множеством серверов

```
Архитектура Ansible:

Control Node (Ansible Controller)
├── Inventory (список серверов)
├── Playbooks (инструкции)
├── Roles (переиспользуемые компоненты)
└── Modules (команды для выполнения)
        ↓
    SSH соединения
        ↓
Managed Nodes (целевые серверы)
├── Server 1
├── Server 2
└── Server N
```

### Установка Ansible

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ansible

# CentOS/RHEL
sudo yum install epel-release
sudo yum install ansible

# pip
pip install ansible

# Проверка установки
ansible --version
```

### Inventory файл

```ini
# /etc/ansible/hosts или inventory/hosts

# Группы серверов
[webservers]
web1.example.com
web2.example.com
web3.example.com

[databases]
db1.example.com
db2.example.com

[loadbalancers]
lb1.example.com

# Переменные для групп
[webservers:vars]
ansible_user=deploy
ansible_ssh_private_key_file=~/.ssh/deploy_key
http_port=80
max_clients=200

[databases:vars]
ansible_user=postgres
postgresql_version=13

# Все серверы production
[production:children]
webservers
databases
loadbalancers
```

### Простые ad-hoc команды

```bash
# Ping всех серверов
ansible all -m ping

# Выполнение команды
ansible webservers -m command -a "uptime"
ansible webservers -m shell -a "df -h | grep -v tmpfs"

# Управление сервисами
ansible webservers -m service -a "name=nginx state=restarted"
ansible databases -m service -a "name=postgresql state=started enabled=yes"

# Управление пакетами
ansible webservers -m apt -a "name=nginx state=present" --become
ansible webservers -m yum -a "name=httpd state=latest" --become

# Копирование файлов
ansible webservers -m copy -a "src=/local/file dest=/remote/file" --become
ansible webservers -m file -a "path=/tmp/test state=directory mode=0755" --become

# Сбор фактов о системе
ansible webservers -m setup | grep ansible_distribution
ansible webservers -m setup -a "filter=ansible_memory_mb"
```

### Playbook для настройки веб-сервера

```yaml
# webserver-setup.yml
---
- name: Setup Web Server
  hosts: webservers
  become: yes
  vars:
    nginx_version: "1.20.*"
    domain_name: "{{ inventory_hostname }}"
    
  tasks:
    - name: Update package cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
      when: ansible_os_family == "Debian"
    
    - name: Install Nginx
      package:
        name: nginx
        state: present
    
    - name: Install additional packages
      package:
        name: "{{ item }}"
        state: present
      loop:
        - ufw
        - certbot
        - python3-certbot-nginx
    
    - name: Start and enable Nginx
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Configure firewall
      ufw:
        rule: allow
        port: "{{ item }}"
        proto: tcp
      loop:
        - '22'
        - '80'
        - '443'
    
    - name: Enable firewall
      ufw:
        state: enabled
        policy: deny
        direction: incoming
    
    - name: Create document root
      file:
        path: "/var/www/{{ domain_name }}"
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'
    
    - name: Copy index.html template
      template:
        src: index.html.j2
        dest: "/var/www/{{ domain_name }}/index.html"
        owner: www-data
        group: www-data
        mode: '0644'
      notify: restart nginx
    
    - name: Copy Nginx configuration
      template:
        src: nginx-site.conf.j2
        dest: "/etc/nginx/sites-available/{{ domain_name }}"
        owner: root
        group: root
        mode: '0644'
      notify: restart nginx
    
    - name: Enable site
      file:
        src: "/etc/nginx/sites-available/{{ domain_name }}"
        dest: "/etc/nginx/sites-enabled/{{ domain_name }}"
        state: link
      notify: restart nginx
    
    - name: Remove default site
      file:
        path: /etc/nginx/sites-enabled/default
        state: absent
      notify: restart nginx
    
    - name: Test Nginx configuration
      command: nginx -t
      changed_when: false
    
    - name: Obtain SSL certificate
      shell: >
        certbot --nginx -d {{ domain_name }} 
        --non-interactive --agree-tos 
        --email admin@{{ domain_name }}
      args:
        creates: "/etc/letsencrypt/live/{{ domain_name }}/fullchain.pem"
    
    - name: Setup certificate renewal
      cron:
        name: "Renew SSL certificates"
        minute: "0"
        hour: "2"
        job: "certbot renew --quiet && systemctl reload nginx"
  
  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
    
    - name: reload nginx
      service:
        name: nginx
        state: reloaded
```

### Запуск playbooks

```bash
# Основные команды
ansible-playbook webserver-setup.yml                      # запуск playbook
ansible-playbook -i inventory/hosts webserver-setup.yml   # с конкретным inventory
ansible-playbook webserver-setup.yml --limit webservers   # только для группы webservers
ansible-playbook webserver-setup.yml --tags nginx         # только задачи с тегом nginx

# Проверка без выполнения
ansible-playbook webserver-setup.yml --check              # dry run
ansible-playbook webserver-setup.yml --diff               # показать изменения

# Отладка
ansible-playbook webserver-setup.yml -v                   # verbose
ansible-playbook webserver-setup.yml -vvv                 # очень verbose

# Выполнение конкретных задач
ansible-playbook webserver-setup.yml --step               # пошаговое выполнение
ansible-playbook webserver-setup.yml --start-at-task="Install Nginx"
```

---

# Блок 8: Мониторинг и логирование

## Глава 30: Prometheus и Grafana

### Prometheus — система мониторинга и алертинга, Grafana — платформа для визуализации метрик

```
Архитектура мониторинга:

Приложения → Exporters → Prometheus → Grafana
     ↓           ↓           ↓          ↓
  Метрики   Преобразование  Хранение  Визуализация
             в формат       временных    и алерты
             Prometheus      рядов
```

### Установка Prometheus

```bash
# Создание пользователя
sudo useradd --no-create-home --shell /bin/false prometheus

# Создание директорий
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus

# Скачивание и установка
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.30.3/prometheus-2.30.3.linux-amd64.tar.gz
tar xvf prometheus-2.30.3.linux-amd64.tar.gz
cd prometheus-2.30.3.linux-amd64

# Копирование файлов
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool

# Копирование конфигурации
sudo cp -r consoles /etc/prometheus
sudo cp -r console_libraries /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

### Конфигурация Prometheus

```yaml
# /etc/prometheus/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "rules/*.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - localhost:9093

scrape_configs:
  # Prometheus сам себя
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Node Exporter для системных метрик
  - job_name: 'node'
    static_configs:
      - targets: 
        - 'web1.example.com:9100'
        - 'web2.example.com:9100'
        - 'db1.example.com:9100'

  # Nginx метрики
  - job_name: 'nginx'
    static_configs:
      - targets: ['localhost:9113']

  # PostgreSQL метрики
  - job_name: 'postgres'
    static_configs:
      - targets: ['localhost:9187']

  # Application метрики
  - job_name: 'myapp'
    static_configs:
      - targets: ['localhost:8000']
    metrics_path: '/metrics'
    scrape_interval: 5s

  # Docker метрики
  - job_name: 'docker'
    static_configs:
      - targets: ['localhost:9323']

  # Blackbox exporter для проверки доступности
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - https://example.com
        - https://api.example.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9115
```

### Node Exporter для системных метрик

```bash
# Установка Node Exporter
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz
tar xvf node_exporter-1.2.2.linux-amd64.tar.gz
sudo cp node_exporter-1.2.2.linux-amd64/node_exporter /usr/local/bin/

# Создание пользователя
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

### Установка Grafana

```bash
# Добавление репозитория
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

# Установка
sudo apt-get update
sudo apt-get install grafana

# Запуск
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
sudo systemctl status grafana-server

# Доступ: http://localhost:3000 (admin/admin)
```

### Правила алертинга Prometheus

```yaml
# /etc/prometheus/rules/alerts.yml
groups:
  - name: system_alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage detected"
          description: "CPU usage is above 80% for more than 5 minutes"

      - alert: HighMemoryUsage
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 90
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High memory usage detected"
          description: "Memory usage is above 90% for more than 5 minutes"

      - alert: DiskSpaceLow
        expr: 100 - ((node_filesystem_avail_bytes{mountpoint="/"} * 100) / node_filesystem_size_bytes{mountpoint="/"}) > 85
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Low disk space"
          description: "Disk usage is above 85%"

      - alert: ServiceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Service is down"
          description: "Service {{ $labels.job }} is down"

  - name: application_alerts
    rules:
      - alert: HighResponseTime
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 0.5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High response time"
          description: "95th percentile response time is above 500ms"

      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate"
          description: "Error rate is above 10%"
```

## Глава 31: ELK Stack для логирования

### ELK Stack (Elasticsearch, Logstash, Kibana) обеспечивает централизованное логирование и анализ

```
Архитектура ELK:

Приложения → Filebeat → Logstash → Elasticsearch → Kibana
     ↓          ↓          ↓           ↓            ↓
   Логи    Сбор логов  Обработка   Индексация   Визуализация
           с серверов   и парсинг   и хранение   и поиск
```

### Docker Compose для ELK Stack

```yaml
# elk-stack.yml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.0
    container_name: elasticsearch
    environment:
      - node.name=elasticsearch
      - cluster.name=elk-cluster
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms2g -Xmx2g"
      - xpack.security.enabled=false
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
      - "9300:9300"
    networks:
      - elk

  logstash:
    image: docker.elastic.co/logstash/logstash:7.15.0
    container_name: logstash
    volumes:
      - ./logstash/config:/usr/share/logstash/config:ro
      - ./logstash/pipeline:/usr/share/logstash/pipeline:ro
    ports:
      - "5044:5044"
      - "5000:5000/tcp"
      - "5000:5000/udp"
      - "9600:9600"
    environment:
      LS_JAVA_OPTS: "-Xmx1g -Xms1g"
    networks:
      - elk
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:7.15.0
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      ELASTICSEARCH_URL: http://elasticsearch:9200
      ELASTICSEARCH_HOSTS: '["http://elasticsearch:9200"]'
    networks:
      - elk
    depends_on:
      - elasticsearch

  filebeat:
    image: docker.elastic.co/beats/filebeat:7.15.0
    container_name: filebeat
    user: root
    volumes:
      - ./filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /var/log:/var/log:ro
    networks:
      - elk
    depends_on:
      - logstash

volumes:
  elasticsearch_data:
    driver: local

networks:
  elk:
    driver: bridge
```

### Конфигурация Logstash

```ruby
# logstash/pipeline/logstash.conf
input {
  beats {
    port => 5044
  }
  
  tcp {
    port => 5000
    codec => json
  }
  
  udp {
    port => 5000
    codec => json
  }
}

filter {
  # Nginx access logs
  if [fileset][module] == "nginx" and [fileset][name] == "access" {
    grok {
      match => { 
        "message" => '%{IPORHOST:remote_addr} - %{DATA:remote_user} \[%{HTTPDATE:time_local}\] "%{WORD:method} %{DATA:request} HTTP/%{NUMBER:http_version}" %{INT:status} %{INT:body_bytes_sent} "%{DATA:http_referer}" "%{DATA:http_user_agent}"'
      }
    }
    
    date {
      match => [ "time_local", "dd/MMM/yyyy:HH:mm:ss Z" ]
    }
    
    mutate {
      convert => {
        "status" => "integer"
        "body_bytes_sent" => "integer"
      }
    }
  }
  
  # Application logs
  if [fields][service] == "myapp" {
    if [message] =~ /^\{.*\}$/ {
      json {
        source => "message"
      }
    }
    
    # Добавление геоинформации по IP
    if [remote_addr] {
      geoip {
        source => "remote_addr"
        target => "geoip"
      }
    }
  }
  
  # Docker logs
  if [container][name] {
    mutate {
      add_field => { "service" => "%{[container][name]}" }
    }
  }
  
  # Обработка уровней логирования
  if [level] {
    mutate {
      uppercase => [ "level" ]
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
  
  # Дебаг вывод
  stdout {
    codec => rubydebug
  }
}
```

### Конфигурация Filebeat

```yaml
# filebeat/filebeat.yml
filebeat.inputs:
  # Системные логи
  - type: log
    enabled: true
    paths:
      - /var/log/syslog
      - /var/log/auth.log
    fields:
      service: system
      
  # Nginx логи
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/access.log
    fields:
      service: nginx
      log_type: access
      
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/error.log
    fields:
      service: nginx
      log_type: error
      
  # Docker логи
  - type: docker
    containers.ids:
      - '*'
    processors:
      - add_docker_metadata:
          host: "unix:///var/run/docker.sock"

# Модули
filebeat.modules:
  - module: nginx
    access:
      enabled: true
      var.paths: ["/var/log/nginx/access.log"]
    error:
      enabled: true
      var.paths: ["/var/log/nginx/error.log"]
      
  - module: system
    syslog:
      enabled: true
    auth:
      enabled: true

# Output к Logstash
output.logstash:
  hosts: ["logstash:5044"]

# Настройки логирования
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
  permissions: 0644

# Мониторинг
monitoring.enabled: true
```

## Глава 32: Мониторинг производительности в реальном времени

### Комплексный мониторинг с Netdata

```bash
# Установка Netdata (мониторинг в реальном времени)
bash <(curl -Ss https://my-netdata.io/kickstart.sh)

# Или установка из пакетов
sudo apt install netdata                                  # Ubuntu/Debian

# Конфигурация /etc/netdata/netdata.conf
[global]
    hostname = web-server-01
    history = 3600                                         # история данных в секундах
    update every = 1                                       # интервал обновления

[web]
    bind to = 127.0.0.1:19999                             # только локальный доступ
    
# Доступ через nginx proxy
# /etc/nginx/sites-available/netdata
server {
    listen 80;
    server_name monitoring.example.com;
    
    location / {
        proxy_pass http://127.0.0.1:19999;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# Netdata дает метрики по:
# - CPU, память, диски, сеть
# - Процессы и сервисы
# - Nginx, Apache, MySQL, PostgreSQL
# - Docker контейнеры
# - Системные вызовы и прерывания
```

### Настройка алертов в Netdata

```bash
# /etc/netdata/health.d/cpu_usage.conf
alarm: cpu_usage_high
    on: system.cpu
lookup: average -3m unaligned of user,system,softirq,irq,guest
  units: %
  every: 10s
   warn: $this > 80
   crit: $this > 95
  delay: down 15m multiplier 1.5 max 1h
   info: average CPU utilization over the last 3 minutes
     to: sysadmin

# /etc/netdata/health.d/memory_usage.conf
alarm: ram_usage_high
    on: system.ram
lookup: average -1m unaligned of used
calc: $this * 100 / ($this + $free + $buffers + $cached)
units: %
every: 10s
warn: $this > 90
crit: $this > 95
delay: down 15m multiplier 1.5 max 1h
info: system RAM usage
to: sysadmin

# Настройка уведомлений через Slack
# /etc/netdata/health_alarm_notify.conf
SEND_SLACK="YES"
SLACK_WEBHOOK_URL="https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK"
DEFAULT_RECIPIENT_SLACK="monitoring"
```

---

# Блок 9: Безопасность в production

## Глава 33: Hardening системы

### Hardening — процесс укрепления безопасности системы путем минимизации поверхности атаки

### Обновления безопасности

```bash
# Автоматические обновления безопасности (Ubuntu/Debian)
sudo apt install unattended-upgrades apt-listchanges

# Конфигурация
sudo dpkg-reconfigure -plow unattended-upgrades

# Настройка /etc/apt/apt.conf.d/50unattended-upgrades
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}";
    "${distro_id}:${distro_codename}-security";
    "${distro_id}ESMApps:${distro_codename}-apps-security";
    "${distro_id}ESM:${distro_codename}-infra-security";
};

Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Remove-New-Unused-Dependencies "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "false";
Unattended-Upgrade::Automatic-Reboot-Time "02:00";

# Мониторинг обновлений
sudo cat /var/log/unattended-upgrades/unattended-upgrades.log
```

### Отключение ненужных сервисов

```bash
# Просмотр всех сервисов
systemctl list-unit-files --type=service --state=enabled

# Отключение потенциально опасных сервисов
sudo systemctl disable telnet.socket
sudo systemctl disable rsh.socket
sudo systemctl disable rlogin.socket
sudo systemctl disable vsftpd
sudo systemctl disable apache2                             # если используется nginx

# Проверка открытых портов
sudo netstat -tulnp
sudo ss -tulnp

# Закрытие неиспользуемых портов через firewall
sudo ufw deny 23/tcp                                       # telnet
sudo ufw deny 21/tcp                                       # ftp
sudo ufw deny 512/tcp                                      # rexec
sudo ufw deny 513/tcp                                      # rlogin
sudo ufw deny 514/tcp                                      # rsh
```

### Настройка SSH для максимальной безопасности

```bash
# /etc/ssh/sshd_config - рекомендуемые настройки безопасности

# Изменение порта (security through obscurity)
Port 2222

# Запрет root login
PermitRootLogin no

# Только ключевая аутентификация
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

# Ограничение пользователей
AllowUsers alice bob charlie
DenyUsers guest test

# Ограничение групп
AllowGroups ssh-users sudo

# Таймауты и ограничения
MaxAuthTries 3
MaxSessions 2
MaxStartups 3
LoginGraceTime 30
ClientAliveInterval 300
ClientAliveCountMax 2

# Запрет пустых паролей
PermitEmptyPasswords no

# Запрет X11 forwarding если не нужен
X11Forwarding no

# Запрет TCP forwarding если не нужен
AllowTcpForwarding no

# Запрет Agent forwarding
AllowAgentForwarding no

# Использование сильных алгоритмов шифрования
Protocol 2
Ciphers aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-sha2-256,hmac-sha2-512
KexAlgorithms diffie-hellman-group14-sha256,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512

# Баннер предупреждения
Banner /etc/ssh/banner.txt

# Применение изменений
sudo systemctl restart sshd
```

## Глава 34: Fail2ban - защита от брутфорс атак

### Fail2ban анализирует логи и автоматически блокирует IP адреса с подозрительной активностью

### Установка и базовая настройка

```bash
# Установка
sudo apt install fail2ban                                  # Ubuntu/Debian
sudo yum install epel-release && sudo yum install fail2ban # CentOS/RHEL

# Основные файлы конфигурации
/etc/fail2ban/fail2ban.conf                                # основная конфигурация
/etc/fail2ban/jail.conf                                    # настройки jail (не редактировать!)
/etc/fail2ban/jail.local                                   # локальные настройки
/etc/fail2ban/filter.d/                                    # фильтры логов
/etc/fail2ban/action.d/                                    # действия при блокировке
```

### Конфигурация jail.local

```ini
# /etc/fail2ban/jail.local
[DEFAULT]
# Время блокировки (в секундах)
bantime = 3600

# Время анализа логов (в секундах)
findtime = 600

# Максимальное количество попыток
maxretry = 3

# Игнорируемые IP адреса
ignoreip = 127.0.0.1/8 ::1 192.168.1.0/24 10.0.0.0/8

# Email уведомления
destemail = admin@company.com
sender = fail2ban@company.com
mta = sendmail

# Действие по умолчанию
action = %(action_mwl)s

[sshd]
enabled = true
port = ssh,2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600

[nginx-http-auth]
enabled = true
filter = nginx-http-auth
port = http,https
logpath = /var/log/nginx/error.log
maxretry = 3

[nginx-noscript]
enabled = true
port = http,https
filter = nginx-noscript
logpath = /var/log/nginx/access.log
maxretry = 6

[nginx-badbots]
enabled = true
port = http,https
filter = nginx-badbots
logpath = /var/log/nginx/access.log
maxretry = 2

[nginx-noproxy]
enabled = true
port = http,https
filter = nginx-noproxy
logpath = /var/log/nginx/access.log
maxretry = 2

[postfix]
enabled = true
port = smtp,465,submission
filter = postfix
logpath = /var/log/mail.log

[mysql]
enabled = true
port = 3306
filter = mysqld-auth
logpath = /var/log/mysql/error.log
maxretry = 3

[postgresql]
enabled = true
port = 5432
filter = postgresql
logpath = /var/log/postgresql/postgresql-*.log
maxretry = 3
```

### Управление Fail2ban

```bash
# Статус сервиса
sudo systemctl status fail2ban

# Статус всех jail
sudo fail2ban-client status

# Статус конкретного jail
sudo fail2ban-client status sshd

# Просмотр заблокированных IP
sudo fail2ban-client get sshd banip

# Разблокировка IP
sudo fail2ban-client set sshd unbanip 203.0.113.100

# Блокировка IP вручную
sudo fail2ban-client set sshd banip 203.0.113.100

# Перезагрузка конфигурации
sudo fail2ban-client reload

# Перезапуск конкретного jail
sudo fail2ban-client restart sshd

# Логи Fail2ban
sudo tail -f /var/log/fail2ban.log
```

## Глава 35: Аудит безопасности и backup стратегии

### Lynis - комплексный аудит безопасности

```bash
# Установка Lynis
sudo apt install lynis                                     # Ubuntu/Debian
sudo yum install lynis                                     # CentOS/RHEL

# Или скачивание последней версии
cd /tmp
wget https://downloads.cisofy.com/lynis/lynis-3.0.6.tar.gz
tar xzf lynis-3.0.6.tar.gz
cd lynis

# Запуск аудита
sudo ./lynis audit system

# Просмотр отчета
sudo cat /var/log/lynis.log

# Проверка только определенных категорий
sudo ./lynis audit system --tests SSH,FILE,NETW

# Тихий режим для скриптов
sudo ./lynis audit system --quiet
```

### Комплексная backup стратегия

```bash
#!/bin/bash
# /usr/local/bin/backup-system.sh
# Полная система резервного копирования

set -euo pipefail

# Конфигурация
BACKUP_ROOT="/backup"
RETENTION_DAYS=30
RETENTION_WEEKS=12
RETENTION_MONTHS=12
LOG_FILE="/var/log/backup.log"
NOTIFICATION_EMAIL="admin@company.com"

# Функция логирования
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOG_FILE"
}

# Создание структуры backup
create_backup_structure() {
    local backup_date=$(date +%Y-%m-%d)
    local backup_time=$(date +%H%M%S)
    
    DAILY_DIR="$BACKUP_ROOT/daily/$backup_date"
    WEEKLY_DIR="$BACKUP_ROOT/weekly/$(date +%Y-W%V)"
    MONTHLY_DIR="$BACKUP_ROOT/monthly/$(date +%Y-%m)"
    
    mkdir -p "$DAILY_DIR" "$WEEKLY_DIR" "$MONTHLY_DIR"
    
    # Текущий backup
    CURRENT_BACKUP="$DAILY_DIR/$backup_time"
    mkdir -p "$CURRENT_BACKUP"
}

# Backup конфигурационных файлов
backup_configs() {
    log "Backing up configuration files"
    
    tar -czf "$CURRENT_BACKUP/configs.tar.gz" \
        /etc \
        /var/spool/cron \
        /usr/local/etc \
        2>/dev/null || true
        
    log "Configuration backup completed"
}

# Backup домашних директорий
backup_home_dirs() {
    log "Backing up home directories"
    
    tar -czf "$CURRENT_BACKUP/home.tar.gz" \
        --exclude="*/tmp/*" \
        --exclude="*/.cache/*" \
        --exclude="*/Downloads/*" \
        /home \
        2>/dev/null || true
        
    log "Home directories backup completed"
}

# Backup баз данных
backup_databases() {
    log "Backing up databases"
    
    # PostgreSQL
    if systemctl is-active --quiet postgresql; then
        sudo -u postgres pg_dumpall > "$CURRENT_BACKUP/postgresql.sql"
        gzip "$CURRENT_BACKUP/postgresql.sql"
    fi
    
    # MySQL
    if systemctl is-active --quiet mysql; then
        mysqldump --all-databases --single-transaction --routines --triggers > "$CURRENT_BACKUP/mysql.sql"
        gzip "$CURRENT_BACKUP/mysql.sql"
    fi
    
    # Redis
    if systemctl is-active --quiet redis; then
        cp /var/lib/redis/dump.rdb "$CURRENT_BACKUP/redis.rdb"
    fi
    
    log "Database backup completed"
}

# Главная функция
main() {
    log "Starting backup process"
    
    create_backup_structure
    backup_configs
    backup_home_dirs
    backup_databases
    
    log "Backup completed successfully"
}

# Запуск
main "$@"
```

### Автоматизация backup через cron

```bash
# /etc/cron.d/backup-system
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=admin@company.com

# Ежедневный backup в 2:00
0 2 * * * root /usr/local/bin/backup-system.sh

# Проверка backup'ов каждый час
0 * * * * root /usr/local/bin/check-backup-health.sh

# Тестовое восстановление раз в неделю
0 4 * * 0 root /usr/local/bin/test-restore.sh
```

---

# Блок 10: Performance tuning и практические сценарии

## Глава 36: Оптимизация системных параметров

### Linux предоставляет множество параметров для тюнинга производительности под конкретные рабочие нагрузки

### Настройка параметров ядра (sysctl)

```bash
# Просмотр текущих параметров
sysctl -a | grep -E "(net|vm|kernel)" | head -20

# Временное изменение параметра
sudo sysctl net.core.somaxconn=65535

# Постоянные настройки в /etc/sysctl.conf или /etc/sysctl.d/99-custom.conf
```

```ini
# /etc/sysctl.d/99-performance.conf
# Высокопроизводительные сетевые настройки для web-серверов

# === Сетевые параметры ===

# Размер очереди входящих соединений
net.core.somaxconn = 65535

# Размер буфера для сетевых устройств
net.core.netdev_max_backlog = 30000

# Размеры сокетных буферов
net.core.rmem_default = 262144
net.core.rmem_max = 16777216
net.core.wmem_default = 262144
net.core.wmem_max = 16777216

# TCP буферы
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216

# TCP congestion control (BBR для высокой пропускной способности)
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr

# TCP настройки для высокой нагрузки
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_intvl = 30
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 2000000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_no_metrics_save = 1

# Увеличение диапазона локальных портов
net.ipv4.ip_local_port_range = 1024 65535

# === Параметры виртуальной памяти ===

# Агрессивность swap (0-100, меньше = реже swap)
vm.swappiness = 1

# Процент грязных страниц для начала записи на диск
vm.dirty_background_ratio = 5
vm.dirty_ratio = 10

# Время хранения грязных страниц (сотые доли секунды)
vm.dirty_expire_centisecs = 3000
vm.dirty_writeback_centisecs = 500

# === Файловая система ===

# Максимальное количество открытых файлов
fs.file-max = 2097152

# Максимальное количество inotify watches
fs.inotify.max_user_watches = 524288
fs.inotify.max_user_instances = 256

# === Безопасность ===

# Защита от SYN flood атак
net.ipv4.tcp_syncookies = 1

# Игнорирование ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0

# Защита от IP spoofing
net.ipv4.conf.all.rp_filter = 1

# === Ядро ===

# Размер разделяемой памяти
kernel.shmmax = 68719476736
kernel.shmall = 4294967296

# Максимальное количество процессов
kernel.pid_max = 4194304

# Применение настроек
# sudo sysctl -p /etc/sysctl.d/99-performance.conf
```

### Оптимизация лимитов системы

```bash
# /etc/security/limits.conf - лимиты для пользователей
# <domain> <type> <item> <value>

# Лимиты для всех пользователей
* soft nofile 65535
* hard nofile 65535
* soft nproc 32768
* hard nproc 32768

# Лимиты для конкретных пользователей/групп
www-data soft nofile 100000
www-data hard nofile 100000

postgres soft nofile 8192
postgres hard nofile 8192

# Память для core dumps
* soft core unlimited

# Размер стека
* soft stack 8192
* hard stack 32768

# Проверка текущих лимитов
ulimit -a                                                  # все лимиты
ulimit -n                                                  # файловые дескрипторы
ulimit -u                                                  # процессы

# Временное изменение лимитов
ulimit -n 100000                                           # увеличить лимит файлов
```

## Глава 37: Анализ производительности

### CPU профилирование с perf

```bash
# Установка perf
sudo apt install linux-tools-common linux-tools-generic    # Ubuntu/Debian
sudo yum install perf                                       # CentOS/RHEL

# Системное профилирование
sudo perf top                                               # топ функций в реальном времени
sudo perf top -p $(pgrep nginx)                            # профилирование конкретного процесса

# Запись профиля
sudo perf record -g ./my_application                       # запись с call graph
sudo perf record -F 99 -a -g -- sleep 30                  # запись всей системы 30 секунд

# Анализ записанного профиля
sudo perf report                                            # интерактивный анализ
sudo perf report --stdio                                   # текстовый отчет

# CPU события
sudo perf stat ./my_application                            # статистика выполнения
sudo perf stat -e cache-misses,cache-references ./app     # конкретные события
```

### Анализ памяти

```bash
# Использование памяти процессом
pmap -x $(pgrep nginx)                                     # карта памяти процесса
cat /proc/$(pgrep nginx)/smaps                             # детальная информация

# Анализ утечек памяти с valgrind
sudo apt install valgrind
valgrind --tool=memcheck --leak-check=full ./my_app

# Профилирование памяти с massif
valgrind --tool=massif ./my_app
ms_print massif.out.* | head -50

# Анализ через /proc/meminfo
watch -n 1 'cat /proc/meminfo | head -20'

# Статистика по слабам (slab allocator)
sudo cat /proc/slabinfo | head -20
sudo slabtop                                               # топ потребителей slab памяти
```

### I/O анализ

```bash
# iostat для анализа дисковой активности
iostat -x 1                                               # каждую секунду
iostat -x -d 1 10                                         # 10 раз каждую секунду

# Важные метрики iostat:
# %util - процент времени занятости диска
# await - среднее время ожидания запроса
# r/s, w/s - операции чтения/записи в секунду
# rkB/s, wkB/s - KB/s чтения/записи

# iotop для мониторинга I/O по процессам
sudo iotop -o                                             # только активные процессы
sudo iotop -a                                             # накопленный I/O

# Тестирование производительности диска
# Последовательное чтение
sudo dd if=/dev/sda of=/dev/null bs=1M count=1000

# Последовательная запись
sudo dd if=/dev/zero of=/tmp/testfile bs=1M count=1000 oflag=direct

# Случайный I/O с fio
sudo apt install fio
fio --name=random-rw --ioengine=libaio --iodepth=64 --rw=randrw --bs=4k --direct=1 --size=1G --numjobs=4 --runtime=60 --group_reporting
```

## Глава 38: Практические сценарии решения проблем

### Сценарий 1: Диагностика высокой нагрузки

**Проблема:** Сервер работает медленно, высокий load average.

```bash
# Шаг 1: Общий обзор системы
top                                                        # общая картина
htop                                                       # более удобный интерфейс
uptime                                                     # load average

# Шаг 2: Анализ загрузки CPU
iostat -x 1 5                                            # I/O wait
vmstat 1 5                                                # статистика виртуальной памяти
sar -u 1 5                                                # детальная статистика CPU

# Если высокий %iowait - проблема с дисками
# Если высокий %sys - много системных вызовов
# Если высокий %user - приложения потребляют CPU

# Шаг 3: Поиск процессов-потребителей
ps aux --sort=-%cpu | head -10                           # топ по CPU
ps aux --sort=-%mem | head -10                           # топ по памяти

# Детальный анализ процесса
strace -p PID                                             # системные вызовы
lsof -p PID                                               # открытые файлы
pmap -x PID                                               # использование памяти

# Шаг 4: Анализ сетевой активности
netstat -i                                                # статистика интерфейсов
ss -tuln | wc -l                                         # количество соединений
ss -s                                                     # сводная статистика

# Шаг 5: Проверка дискового пространства
df -h                                                     # свободное место
du -sh /var/log/* | sort -hr                            # размер логов
lsof +L1                                                  # удаленные но открытые файлы

# Шаг 6: Анализ логов
tail -f /var/log/syslog                                  # системные сообщения
journalctl -f                                            # systemd логи
tail -f /var/log/nginx/error.log                        # ошибки веб-сервера
```

### Сценарий 2: Проблемы с сетью

**Проблема:** Сайт недоступен или работает медленно.

```bash
# Шаг 1: Базовая диагностика подключения
ping -c 4 google.com                                     # доступность интернета
ping -c 4 localhost                                      # локальная сеть
curl -I http://localhost                                 # HTTP ответ сервера

# Шаг 2: Проверка сетевых интерфейсов
ip addr show                                             # IP адреса интерфейсов
ip route show                                            # таблица маршрутизации
netstat -i                                               # статистика интерфейсов

# Шаг 3: Проверка слушающих портов
ss -tlnp | grep :80                                     # кто слушает порт 80
ss -tlnp | grep :443                                    # кто слушает порт 443
nmap localhost                                           # сканирование портов

# Шаг 4: Анализ сетевого трафика
sudo tcpdump -i eth0 port 80                           # HTTP трафик
sudo tcpdump -i eth0 host 8.8.8.8                      # трафик к DNS серверу
sudo iftop                                               # мониторинг трафика

# Шаг 5: DNS диагностика
dig google.com                                          # DNS запрос
nslookup google.com                                     # альтернативный DNS запрос
cat /etc/resolv.conf                                    # настройки DNS

# Шаг 6: Проверка firewall
sudo ufw status                                          # статус UFW
sudo iptables -L -n                                     # правила iptables
sudo iptables -L -n -v                                  # с счетчиками пакетов

# Шаг 7: Диагностика конкретного сервиса
systemctl status nginx                                  # статус веб-сервера
nginx -t                                                 # проверка конфигурации nginx
curl -v http://localhost                                # подробный HTTP запрос
```

### Сценарий 3: Проблемы с дисковым пространством

**Проблема:** Заканчивается место на диске.

```bash
# Шаг 1: Обзор использования дисков
df -h                                                    # общее использование
df -i                                                    # использование inodes

# Шаг 2: Поиск больших файлов и директорий
du -sh /* | sort -hr | head -10                        # топ директорий по размеру
find / -type f -size +1G 2>/dev/null | head -10       # файлы больше 1GB
find /var/log -type f -size +100M                      # большие лог файлы

# Шаг 3: Анализ конкретных директорий
du -sh /var/log/* | sort -hr                           # размер логов
du -sh /home/* | sort -hr                              # размер домашних папок
du -sh /opt/* | sort -hr                               # установленные приложения

# Шаг 4: Поиск удаленных но открытых файлов
lsof +L1                                                # файлы удалены но все еще открыты
lsof | grep "(deleted)"                                # альтернативный способ

# Шаг 5: Очистка места
# Логи
sudo journalctl --vacuum-time=7d                       # оставить журналы за 7 дней
sudo logrotate -f /etc/logrotate.conf                  # принудительная ротация логов

# Temporary файлы
sudo find /tmp -type f -atime +7 -delete               # удалить старые temp файлы
sudo find /var/tmp -type f -atime +30 -delete          # удалить старые var/tmp файлы

# Package cache
sudo apt clean                                          # очистить кеш пакетов (Debian/Ubuntu)
sudo yum clean all                                      # очистить кеш пакетов (RHEL/CentOS)

# Docker
docker system prune -a                                  # удалить неиспользуемые образы и контейнеры
docker volume prune                                     # удалить неиспользуемые volumes
```

---

# Заключение и следующие шаги

## Путь, который вы прошли

Поздравляем! Вы прошли путь от основ Linux до экспертного уровня. Теперь у вас есть глубокие знания в:

### 🔧 Основные навыки
- **Командная строка** — автоматизация повседневных задач через bash скрипты
- **Файловая система** — понимание структуры и эффективная работа с файлами
- **Права доступа** — настройка безопасности на уровне файлов и пользователей
- **Процессы** — мониторинг, управление и оптимизация системных ресурсов

### 🌐 Сетевые технологии
- **TCP/IP стек** — глубокое понимание сетевых протоколов
- **DNS и маршрутизация** — настройка и диагностика сетевой инфраструктуры
- **Файрволы** — защита сервера с помощью iptables и UFW
- **Мониторинг трафика** — анализ сетевой активности и решение проблем

### 🚀 Инфраструктура и DevOps
- **Веб-серверы** — настройка и оптимизация Nginx для production
- **Базы данных** — администрирование PostgreSQL и оптимизация производительности
- **Контейнеризация** — Docker и Docker Compose для современного развертывания
- **Автоматизация** — Ansible для управления конфигурацией множества серверов

### 📊 Мониторинг и безопасность
- **Prometheus + Grafana** — профессиональный мониторинг инфраструктуры
- **ELK Stack** — централизованное логирование и анализ
- **Hardening системы** — укрепление безопасности production серверов
- **Fail2ban** — автоматическая защита от атак

### ⚡ Performance tuning
- **Системные параметры** — тюнинг ядра Linux для высоких нагрузок
- **Анализ производительности** — профилирование CPU, памяти и I/O
- **Оптимизация стека** — настройка веб-сервера и базы данных
- **Диагностика проблем** — систематический подход к решению проблем

---

## Что дальше?

Теперь, когда у вас есть прочная основа в Linux, рекомендуем развиваться в следующих направлениях:

### 🔄 Оркестрация контейнеров
**Kubernetes** — следующий естественный шаг после Docker:
```bash
# Изучите основы Kubernetes
kubectl get nodes
kubectl get pods
kubectl apply -f deployment.yaml

# Понимание концепций:
# - Pods, Services, Deployments
# - Ingress, ConfigMaps, Secrets  
# - Helm для управления пакетами
# - Мониторинг с Prometheus Operator
```

### ☁️ Облачные платформы
Современная инфраструктура работает в облаке:
```bash
# AWS
aws ec2 describe-instances
aws s3 ls
aws eks update-kubeconfig --cluster my-cluster

# Google Cloud Platform  
gcloud compute instances list
gcloud container clusters get-credentials my-cluster

# Azure
az vm list
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

### 🏗️ Infrastructure as Code
Управление инфраструктурой через код:
```hcl
# Terraform example
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1d0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
  }
}

# Изучите также:
# - Terraform для создания инфраструктуры
# - Ansible для настройки серверов  
# - Pulumi как современную альтернативу
```

### 🔐 Продвинутая безопасность
Углубление в DevSecOps:
```bash
# Сканирование уязвимостей
trivy image nginx:latest
clair-scanner nginx:latest

# Secrets management
vault write secret/myapp/config db_password=secretpassword
kubectl create secret generic mysecret --from-literal=password=secretpassword

# Compliance as Code
inspec exec compliance-profile
```

### 📈 SRE (Site Reliability Engineering)
Обеспечение надежности больших систем:
```yaml
# SLI/SLO определения
sli:
  - name: "API availability"
    description: "Percentage of successful API requests"
    query: "rate(http_requests_total{status!~'5..'}[5m]) / rate(http_requests_total[5m])"
    
slo:
  - name: "99.9% API availability"
    target: 99.9
    time_window: "30d"
```

### 🤖 GitOps и CI/CD
Автоматизация всего жизненного цикла приложений:
```yaml
# GitHub Actions workflow
name: Deploy to Production
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f k8s/
          kubectl rollout status deployment/app
```

---

## Ресурсы для продолжения обучения

### 📚 Книги
- **"Linux Bible"** by Christopher Negus — всеобъемлющий справочник
- **"UNIX and Linux System Administration Handbook"** — практическое администрирование
- **"Site Reliability Engineering"** by Google — SRE принципы и практики
- **"Kubernetes: Up and Running"** — глубокое погружение в K8s
- **"Terraform: Up & Running"** — Infrastructure as Code

### 🌐 Онлайн ресурсы
- **Linux Academy / A Cloud Guru** — курсы по Linux и облачным технологиям
- **Kubernetes Documentation** — официальная документация K8s
- **DigitalOcean Tutorials** — практические руководства
- **Red Hat Learning** — сертификационные курсы
- **Linux Foundation Training** — официальные курсы Linux

### 🏆 Сертификации
Подтвердите свои знания профессиональными сертификатами:

**Linux сертификации:**
- LPIC-1, LPIC-2, LPIC-3 (LPI)
- RHCSA, RHCE (Red Hat)
- CompTIA Linux+

**Cloud сертификации:**
- AWS Solutions Architect
- Google Cloud Professional
- Azure Solutions Architect

**DevOps сертификации:**
- Certified Kubernetes Administrator (CKA)
- Certified Kubernetes Application Developer (CKAD)
- Docker Certified Associate

---

## Финальные советы

### 🎯 Практика — ключ к успеху
- **Создавайте проекты** — теория без практики бесполезна
- **Ломайте и чините** — лучший способ изучить диагностику
- **Автоматизируйте всё** — если делаете что-то дважды, напишите скрипт
- **Читайте логи** — они расскажут вам всю правду о системе

### 🤝 Сообщество
- **Участвуйте в open source** — вносите вклад в Linux проекты
- **Задавайте вопросы** — Stack Overflow, Reddit, форумы
- **Делитесь знаниями** — блог, выступления, менторство
- **Следите за трендами** — Hacker News, /r/sysadmin, технические блоги

### 📈 Непрерывное обучение
- **Технологии меняются быстро** — будьте готовы учиться постоянно
- **Изучайте смежные области** — сети, безопасность, разработка
- **Экспериментируйте** — домашняя лаборатория или cloud playground
- **Документируйте опыт** — создавайте личную базу знаний

### 🚀 От новичка к эксперту
Помните путь, который вы прошли:
```
Новичок → Знание команд → Понимание системы → Автоматизация → Экспертиза
   ↓            ↓              ↓               ↓           ↓
Страх      Уверенность    Глубокое         Эффективность  Архитектурное
перед      в базовых      понимание        в решении      мышление и
терминалом операциях      взаимосвязей     проблем        стратегическое
                                                          планирование
```

### 💡 Последнее напутствие

Linux — это не просто операционная система, это **философия**:
- **"Делай одну вещь хорошо"** — принцип Unix
- **"Всё является файлом"** — унифицированный интерфейс  
- **"Маленькие программы, работающие вместе"** — модульность
- **"Выбор важнее удобства"** — гибкость конфигурации

Вы теперь часть сообщества, которое поддерживает **96.3% топ-миллиона веб-серверов** в мире. Каждый раз, когда кто-то смотрит YouTube, заказывает еду или использует WhatsApp — они взаимодействуют с Linux системами, которые настроены такими экспертами, как вы.

**Linux — это ваш инструмент для изменения мира. Используйте его мудро! 🐧**

---

*Удачи в вашем путешествии по миру Linux и backend разработки!*