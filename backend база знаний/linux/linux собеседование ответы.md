# Linux Interview Guide - Развернутые ответы

## 🔧 Основы системы

### 1. Процесс загрузки Linux от BIOS до login prompt

**Концепция:** Загрузка Linux проходит несколько последовательных этапов, каждый из которых передает управление следующему.

**Этапы загрузки:**
1. **BIOS/UEFI** - инициализация железа, поиск загрузочного устройства
2. **Bootloader (GRUB)** - загрузка ядра и initramfs в память
3. **Kernel** - инициализация драйверов, монтирование корневой FS
4. **Init system** - запуск systemd/init, базовых сервисов
5. **Login prompt** - готовность к работе пользователя

**Практическое задание - диагностика "Loading kernel":**
```bash
# Загрузка в GRUB rescue режим
grub rescue> ls
grub rescue> set root=(hd0,1)
grub rescue> linux /vmlinuz root=/dev/sda1
grub rescue> initrd /initrd.img
grub rescue> boot

# Анализ проблем загрузки
dmesg | grep -i "fail\|error"
journalctl -b | grep -i "fail"
```

**Углубление:** initramfs - временная корневая файловая система в RAM, содержащая минимальные драйверы для монтирования настоящей корневой FS. Восстановление через GRUB: `grub-install /dev/sda && update-grub`

---

### 2. Системные вызовы при выполнении `ls -la /home`

**Концепция:** Команда ls выполняет серию системных вызовов для взаимодействия с ядром и получения информации о файлах.

**Основные системные вызовы:**
1. `openat()` - открытие директории
2. `getdents64()` - чтение содержимого директории  
3. `lstat()` - получение метаданных каждого файла
4. `write()` - вывод результата

**Практическое задание:**
```bash
# Трассировка системных вызовов
strace -e trace=openat,getdents64,lstat,write ls -la /home

# Краткий вывод только файловых операций
strace -e trace=file ls -la /home 2>&1 | head -20

# Подсчет системных вызовов
strace -c ls -la /home
```

**Углубление:** Файловые дескрипторы: 0-stdin, 1-stdout, 2-stderr. Кеширование FS использует page cache для ускорения повторных обращений к одним и тем же данным.

---

### 3. Разница между процессом и потоком

**Концепция:** Процесс - независимая единица выполнения с собственным адресным пространством. Поток - легковесная единица выполнения внутри процесса, разделяющая память с другими потоками.

**Ключевые различия:**
- Процессы изолированы, потоки разделяют память
- Создание процесса дороже (fork), потока дешевле (clone)
- IPC между процессами сложнее, между потоками проще

**Практическое задание:**
```bash
# Найти все потоки процесса 1234
ps -T -p 1234
# или
ls -la /proc/1234/task/

# Показать потоки в htop
htop -H

# Информация о потоках через /proc
cat /proc/1234/status | grep Threads
```

**Углубление:** При fork() создается точная копия процесса с copy-on-write оптимизацией - реальное копирование памяти происходит только при записи.

---

### 4. Виртуальная память и swap

**Концепция:** Виртуальная память позволяет процессам использовать больше памяти, чем физически доступно. Swap - область на диске для временного хранения неактивных страниц памяти.

**Механизм работы:**
- MMU транслирует виртуальные адреса в физические
- Page fault при обращении к выгруженной странице
- Kernel решает какие страницы выгрузить в swap

**Практическое задание:**
```bash
# Диагностика нехватки памяти
free -h
cat /proc/meminfo | grep -E "MemTotal|MemAvailable|SwapTotal"

# Мониторинг swap активности  
vmstat 1 5
sar -S 1 5

# Проверка процессов по использованию памяти
ps aux --sort=-%mem | head -10
```

**Углубление:** Kernel начинает использовать swap когда свободной памяти становится мало. swappiness (0-100) контролирует агрессивность - по умолчанию 60.

---

### 5. Права доступа и setuid/setgid

**Концепция:** Система прав основана на владельце, группе и остальных (rwx). setuid/setgid позволяют запускать программы с правами владельца файла.

**Биты прав:**
- rwx для user, group, others
- setuid (4000) - выполнение с правами владельца
- setgid (2000) - выполнение с правами группы  
- sticky bit (1000) - только владелец может удалить

**Практическое задание - /usr/bin/passwd:**
```bash
ls -la /usr/bin/passwd
# -rwsr-xr-x root root passwd

# 's' в позиции user execute = setuid bit
# Позволяет обычному пользователю менять /etc/shadow
```

**Углубление:** sticky bit на директориях (как /tmp) позволяет удалять файлы только их владельцам. setuid на скрипты небезопасен - многие системы его игнорируют.

---

### 6. Файловые системы /proc и /sys

**Концепция:** /proc и /sys - виртуальные файловые системы, предоставляющие интерфейс к внутренним структурам ядра.

**/proc содержит:**
- Информацию о процессах (/proc/PID/)
- Системную информацию (/proc/cpuinfo, /proc/meminfo)
- Параметры ядра (/proc/sys/)

**/sys содержит:**
- Структурированную информацию об устройствах
- Параметры драйверов и подсистем

**Практическое задание:**
```bash
# Максимальное количество открытых файлов для процесса
cat /proc/1234/limits | grep "Max open files"
ulimit -n

# Общесистемный лимит
cat /proc/sys/fs/file-max
```

**Углубление:** /proc - старый интерфейс, часто неструктурированный текст. /sys - новый, иерархический интерфейс. /proc/cpuinfo показывает информацию о процессорах.

---

### 7. Сигналы в Linux

**Концепция:** Сигналы - механизм межпроцессного взаимодействия для уведомления процессов о событиях.

**Основные сигналы:**
- SIGTERM (15) - вежливая просьба завершиться
- SIGKILL (9) - принудительное завершение
- SIGINT (2) - прерывание (Ctrl+C)
- SIGHUP (1) - перезагрузка конфигурации

**Практическое задание:**
```bash
# kill -15 - корректное завершение (можно перехватить)
kill -TERM 1234

# kill -9 - принудительное (нельзя перехватить)  
kill -KILL 1234

# Ctrl+C отправляет SIGINT
```

**Углубление:** SIGKILL нельзя перехватить или игнорировать. Zombie-процессы - завершенные дочерние процессы, информация о которых еще не считана родителем.

---

### 8. Hard links и symbolic links

**Концепция:** Hard link - дополнительное имя для того же inode. Symbolic link - файл, содержащий путь к другому файлу.

**Различия:**
- Hard link: тот же inode, работает при удалении оригинала
- Symlink: отдельный inode, ломается при удалении цели

**Практическое задание:**
```bash
# Создание ссылок
echo "test content" > original.txt
ln original.txt hardlink.txt        # hard link
ln -s original.txt symlink.txt      # symbolic link

# Проверка inode
ls -li original.txt hardlink.txt symlink.txt

# Удаление оригинала
rm original.txt
cat hardlink.txt    # работает
cat symlink.txt     # ошибка
```

**Углубление:** Hard link на директорию создать нельзя (кроме . и ..) - это может создать циклы в файловой системе. inode - структура данных с метаинформацией о файле.

---

## 📁 Файловая система и управление

### 9. FHS (Filesystem Hierarchy Standard)

**Концепция:** FHS определяет стандартную структуру директорий в Unix-подобных системах.

**Основные директории:**
- `/bin`, `/usr/bin` - основные программы
- `/etc` - конфигурационные файлы
- `/var` - изменяемые данные (логи, кеши)
- `/opt` - дополнительное ПО
- `/usr/local` - локально установленное ПО

**Практическое задание:**
```bash
# Для собственного приложения:
# /opt - коммерческое/стороннее ПО
sudo mkdir /opt/myapp

# /usr/local - собственные сборки
sudo mkdir /usr/local/bin/myapp

# /home - пользовательские приложения
mkdir ~/bin/myapp
```

**Углубление:** /var часто на отдельном разделе - защита от переполнения логами. /tmp очищается при перезагрузке, /var/tmp - сохраняется.

---

### 10. Монтирование файловых систем

**Концепция:** Монтирование присоединяет файловую систему к дереву каталогов в определенной точке.

**Основы монтирования:**
- mount point - директория для присоединения
- /etc/fstab - постоянная конфигурация монтирования
- mount options определяют режим работы

**Практическое задание:**
```bash
# Монтирование USB диска
sudo mkdir /mnt/usb
sudo mount /dev/sdb1 /mnt/usb

# /etc/fstab структура:
# устройство точка_монтирования тип_FS опции dump pass
/dev/sda1 / ext4 defaults 0 1
/dev/sda2 /home ext4 defaults 0 2
```

**Углубление:** bind mount (`mount --bind`) позволяет смонтировать директорию в другое место. autofs автоматически монтирует FS при обращении.

---

### 11. LVM (Logical Volume Manager)

**Концепция:** LVM добавляет слой абстракции между физическими дисками и файловыми системами, обеспечивая гибкость управления.

**Структура LVM:**
- PV (Physical Volume) - физические диски
- VG (Volume Group) - группа физических томов  
- LV (Logical Volume) - логические разделы

**Практическое задание:**
```bash
# Увеличение логического тома
# 1. Расширить VG (если нужно)
vgextend myvg /dev/sdc1

# 2. Расширить LV
lvextend -L +5G /dev/myvg/mylv

# 3. Расширить файловую систему
resize2fs /dev/myvg/mylv  # для ext4
xfs_growfs /mount/point   # для XFS
```

**Углубление:** LVM snapshots создают моментальные снимки тома. Преимущества: изменение размера на лету, перенос данных без размонтирования.

---

### 12. Файловые системы: ext4, XFS, Btrfs

**Концепция:** Разные ФС оптимизированы для различных задач и нагрузок.

**Сравнение:**
- **ext4**: надежная, универсальная, хорошая совместимость
- **XFS**: высокая производительность для больших файлов
- **Btrfs**: современные возможности (snapshots, compression)

**Практическое задание:**
```bash
# Для базы данных с частыми записями:
# XFS - лучший выбор из-за:
# - Оптимизации для больших файлов
# - Эффективной работы с concurrent I/O
# - Отложенного выделения пространства

mkfs.xfs -f /dev/sdb1
mount -o noatime,nodiratime /dev/sdb1 /var/lib/mysql
```

**Углубление:** Журналирование записывает изменения в журнал перед применением к данным. Btrfs checksums обеспечивают целостность данных.

---

### 13. Система квот (quotas)

**Концепция:** Квоты ограничивают использование дискового пространства и количества файлов пользователями и группами.

**Типы ограничений:**
- Block quota - ограничение по размеру
- Inode quota - ограничение по количеству файлов
- Soft limit - предупреждение
- Hard limit - жесткое ограничение

**Практическое задание:**
```bash
# Включение квот в /etc/fstab
/dev/sda2 /home ext4 defaults,usrquota,grpquota 0 2

# Создание файлов квот
quotacheck -cum /home
quotaon /home

# Установка лимита 1GB для пользователя
edquota -u username
# или
setquota -u username 1000000 1048576 0 0 /home
```

**Углубление:** Soft limit можно превышать в течение grace period. После истечения действует как hard limit.

---

### 14. ACL (Access Control Lists)

**Концепция:** ACL расширяют стандартную модель прав Unix, позволяя задавать права для конкретных пользователей и групп.

**Преимущества ACL:**
- Права для нескольких пользователей/групп
- Более гранулярный контроль
- Наследование прав

**Практическое задание:**
```bash
# Дать пользователю bob права на чтение
setfacl -m u:bob:r file.txt

# Проверить ACL
getfacl file.txt

# Удалить ACL
setfacl -x u:bob file.txt

# Рекурсивно для директории
setfacl -R -m u:bob:rx /path/to/dir
```

**Углубление:** `+` в `ls -la` указывает на наличие ACL. ACL работают поверх стандартных прав, дополняя их.

---

### 15. systemd

**Концепция:** systemd - современная система инициализации и управления сервисами с параллельным запуском и зависимостями.

**Ключевые концепции:**
- Units - базовые объекты (service, target, mount)
- Targets - группы units (аналог runlevels)
- Dependencies - зависимости между units

**Практическое задание:**
```bash
# Создание service для Node.js приложения
sudo tee /etc/systemd/system/myapp.service << EOF
[Unit]
Description=My Node.js App
After=network.target

[Service]
Type=simple
User=nodeuser
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/node app.js
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

# Активация сервиса
sudo systemctl daemon-reload
sudo systemctl enable myapp.service
sudo systemctl start myapp.service
```

**Углубление:** Targets заменили runlevels. Dependency resolution автоматически запускает зависимые сервисы в правильном порядке.

---

### 16. Управление пакетами

**Концепция:** Пакетные менеджеры автоматизируют установку, обновление и удаление программ, управляя зависимостями.

**Основные функции:**
- Установка/удаление пакетов
- Управление зависимостями
- Обновление системы
- Откат изменений

**Практическое задание:**
```bash
# Debian/Ubuntu - откат пакета
apt list --installed | grep package-name
apt install package-name=old-version

# Блокировка версии
apt-mark hold package-name

# CentOS/RHEL - откат
yum downgrade package-name
yum versionlock package-name
```

**Углубление:** Репозитории содержат метаданные пакетов и сами пакеты. Конфликты зависимостей решаются алгоритмами SAT-solver.

---

## 🔄 Процессы и производительность

### 17. Диагностика load average

**Концепция:** Load average показывает среднее количество процессов, ожидающих выполнения (в очереди планировщика или ожидающих I/O).

**Интерпретация:**
- Load = количество ядер - нормальная нагрузка
- Load > количество ядер - перегрузка
- Три значения: 1, 5, 15 минут

**Практическое задание:**
```bash
# Load average = 5.0 на 4-ядерной системе (перегрузка 125%)
# Диагностика причин:

# CPU-bound нагрузка
top -1  # смотрим %CPU процессов
mpstat 1 5  # загрузка по ядрам

# I/O-bound нагрузка  
iostat -x 1 5  # %util близко к 100%
iotop  # процессы с высоким I/O

# Сетевая нагрузка
iftop  # сетевой трафик по соединениям
```

**Углубление:** В top: us=user, sy=system, wa=wait I/O. CPU-bound показывает высокий us/sy, I/O-bound - высокий wa.

---

### 18. Анализ использования памяти

**Концепция:** Linux агрессивно кеширует данные в памяти. "Использованная" память включает кеши, которые могут быть освобождены.

**Ключевые метрики:**
- Total - общая память
- Available - доступная для приложений
- Buffers/Cache - кеши файловой системы

**Практическое задание:**
```bash
# free показывает мало available - не всегда проблема
free -h
#              total   used   free   shared  buff/cache   available
# Mem:          16Gi   2.1Gi  1.2Gi    234Mi       12Gi        13Gi

# available = 13Gi - реально доступно для приложений
# buff/cache будет автоматически освобожден при необходимости

# Проверка реального давления на память
cat /proc/pressure/memory
```

**Углубление:** Page cache ускоряет доступ к файлам. OOM killer срабатывает при критической нехватке памяти, убивая процессы с высоким oom_score.

---

### 19. cgroups (Control Groups)

**Концепция:** cgroups позволяют ограничивать и изолировать ресурсы (CPU, память, I/O) для групп процессов.

**Основные контроллеры:**
- memory - ограничение памяти
- cpu - ограничение CPU
- blkio - ограничение дискового I/O
- devices - контроль доступа к устройствам

**Практическое задание:**
```bash
# Ограничение процесса 512MB памяти (cgroups v1)
sudo mkdir /sys/fs/cgroup/memory/limited_group
echo 536870912 | sudo tee /sys/fs/cgroup/memory/limited_group/memory.limit_in_bytes
echo $$ | sudo tee /sys/fs/cgroup/memory/limited_group/cgroup.procs

# cgroups v2 (современный способ)
sudo mkdir /sys/fs/cgroup/limited
echo "memory.max 512M" | sudo tee /sys/fs/cgroup/limited/memory.max
echo $$ | sudo tee /sys/fs/cgroup/limited/cgroup.procs
```

**Углубление:** Docker использует cgroups для изоляции контейнеров. cgroups v2 предоставляет единый иерархический интерфейс.

---

### 20. Мониторинг дисковой подсистемы

**Концепция:** Производительность дисков характеризуется пропускной способностью, IOPS и временем отклика.

**Ключевые метрики:**
- %util - процент времени занятости диска
- await - среднее время отклика
- IOPS - операций в секунду

**Практическое задание:**
```bash
# Диагностика медленной системы
iostat -x 1 5
# Если %util близко к 100% - диск перегружен
# Высокий await - медленные операции

# Детальный анализ I/O
iotop -o  # процессы с активным I/O
lsof +D /slow/mount/point  # открытые файлы

# Тестирование производительности диска
fio --name=test --rw=randread --bs=4k --numjobs=1 --size=1G
```

**Углубление:** %util=100% не всегда означает насыщение для SSD. Queue depth показывает количество одновременных операций.

---

### 21. IPC (Inter-Process Communication)

**Концепция:** IPC позволяет процессам обмениваться данными и синхронизироваться.

**Методы IPC:**
- Pipes - каналы для передачи данных
- Shared memory - общая память
- Message queues - очереди сообщений
- Sockets - сетевое взаимодействие

**Практическое задание:**
```bash
# Создание named pipe
mkfifo /tmp/mypipe

# Процесс 1 (отправитель)
echo "Hello from process 1" > /tmp/mypipe &

# Процесс 2 (получатель)
cat /tmp/mypipe

# Удаление pipe
rm /tmp/mypipe
```

**Углубление:** Pipes - для последовательной передачи. Shared memory - для высокоскоростного обмена большими объемами. Sockets - для сетевого IPC.

---

### 22. Планировщик процессов

**Концепция:** Планировщик определяет, какой процесс получит процессорное время и на какой период.

**CFS (Completely Fair Scheduler):**
- Виртуальное время выполнения
- Красно-черное дерево для эффективности
- Nice values влияют на приоритет

**Практическое задание:**
```bash
# Повышение приоритета процесса
nice -n -10 command  # запуск с высоким приоритетом
renice -10 1234      # изменение приоритета существующего процесса

# nice values: от -20 (высший) до +19 (низший)
# По умолчанию = 0
```

**Углубление:** Real-time scheduling (SCHED_FIFO, SCHED_RR) имеет приоритет над обычными процессами. CFS обеспечивает справедливое распределение CPU времени.

---

### 23. Диагностика с strace и perf

**Концепция:** strace показывает системные вызовы процесса. perf анализирует производительность на уровне CPU и ядра.

**Применение:**
- strace - отладка программ, поиск проблем I/O
- perf - профилирование CPU, поиск узких мест

**Практическое задание:**
```bash
# Диагностика медленного процесса
# 1. Проверить системные вызовы
strace -T -p PID  # время выполнения каждого вызова
strace -c -p PID  # статистика вызовов

# 2. Профилирование CPU
perf top -p PID   # реальном времени
perf record -p PID
perf report       # детальный анализ

# 3. Анализ конкретных функций
perf stat -p PID  # счетчики производительности
```

**Углубление:** perf top показывает функции, потребляющие больше всего CPU. CPU profiling помогает найти hot spots в коде.

---

### 24. Namespaces

**Концепция:** Namespaces обеспечивают изоляцию ресурсов, позволяя процессам видеть только свое подмножество системных ресурсов.

**Типы namespaces:**
- PID - изоляция ID процессов  
- NET - сетевая изоляция
- MNT - изоляция файловых систем
- UTS - изоляция hostname
- IPC - изоляция IPC объектов

**Практическое задание:**
```bash
# Создание PID namespace
unshare --pid --fork --mount-proc bash
# В новом namespace PID bash будет 1

# Проверка namespace
ls -la /proc/self/ns/
readlink /proc/self/ns/pid

# Просмотр процессов в namespace
ps aux  # покажет только процессы этого namespace
```

**Углубление:** Контейнеры (Docker) используют комбинацию namespaces для изоляции. Каждый тип namespace изолирует определенный аспект системы.

---

## 🌐 Сеть и безопасность

### 25. TCP/IP стек в Linux

**Концепция:** Сетевой стек Linux реализует модель TCP/IP с четырьмя уровнями: Link, Network, Transport, Application.

**Путь пакета:**
1. Application → socket API
2. Transport layer (TCP/UDP) → добавление заголовков
3. Network layer (IP) → маршрутизация
4. Link layer → физическая передача

**Практическое задание:**
```bash
# Трассировка пути пакета
# 1. Приложение создает socket
ss -tulpn | grep :80

# 2. Ядро обрабатывает пакет
tcpdump -i eth0 -n host google.com

# 3. Маршрутизация
ip route get 8.8.8.8

# 4. Отправка через интерфейс
ethtool eth0  # статистика интерфейса
```

**Углубление:** netfilter - подсистема для фильтрации пакетов. Connection tracking отслеживает состояние соединений для NAT и firewalling.

---

### 26. iptables

**Концепция:** iptables - интерфейс для управления netfilter, позволяющий фильтровать, модифицировать и перенаправлять сетевые пакеты.

**Структура:**
- Tables: filter, nat, mangle, raw
- Chains: INPUT, OUTPUT, FORWARD
- Rules: критерии и действия

**Практическое задание:**
```bash
# Блокировка порта 22 кроме подсети 192.168.1.0/24
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP

# Сохранение правил
iptables-save > /etc/iptables/rules.v4

# Просмотр правил с номерами
iptables -L INPUT --line-numbers
```

**Углубление:** Tables: filter (фильтрация), nat (трансляция адресов), mangle (модификация пакетов). NAT изменяет адреса в заголовках пакетов.

---

### 27. Настройка и защита SSH

**Концепция:** SSH обеспечивает безопасное удаленное подключение через шифрование. Правильная настройка критична для безопасности.

**Основы безопасности:**
- Отключение password authentication
- Использование ключей
- Изменение порта
- Ограничение пользователей

**Практическое задание:**
```bash
# Настройка SSH key authentication
# На клиенте:
ssh-keygen -t ed25519 -C "user@example.com"
ssh-copy-id user@server

# На сервере (/etc/ssh/sshd_config):
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
Port 2222

# Перезапуск SSH
systemctl restart sshd
```

**Углубление:** SSH agent forwarding позволяет использовать локальные ключи на удаленном сервере. Port forwarding создает туннели для безопасной передачи трафика.

---

### 28. DNS диагностика

**Концепция:** DNS транслирует доменные имена в IP-адреса. Проблемы с DNS часто являются причиной недоступности сайтов.

**Компоненты DNS:**
- Resolver - клиентская библиотека
- Cache - локальное кеширование
- Recursive/Authoritative серверы

**Практическое задание:**
```bash
# Диагностика DNS проблем
# 1. Проверка разрешения имени
nslookup google.com
dig google.com

# 2. Проверка конфигурации
cat /etc/resolv.conf
cat /etc/nsswitch.conf

# 3. Очистка кеша
sudo systemctl flush-dns  # systemd-resolved
sudo /etc/init.d/nscd restart  # nscd

# 4. Трассировка DNS запросов
dig +trace google.com
```

**Углубление:** DNS caching ускоряет повторные запросы. /etc/nsswitch.conf определяет порядок разрешения имен (файлы, DNS, etc).

---

### 29. SELinux/AppArmor

**Концепция:** Mandatory Access Control (MAC) системы добавляют дополнительный уровень безопасности поверх стандартных прав Unix.

**SELinux режимы:**
- Enforcing - блокирует нарушения
- Permissive - логирует, но не блокирует  
- Disabled - отключен

**Практическое задание:**
```bash
# Диагностика SELinux проблемы
# Приложение не может записать файл
ausearch -m AVC -ts recent  # поиск нарушений

# Проверка контекста файла
ls -Z /path/to/file
ps -eZ | grep httpd

# Исправление контекста
restorecon -R /var/www/html
setsebool -P httpd_can_network_connect on

# Временное отключение для диагностики
setenforce 0
```

**Углубление:** SELinux contexts определяют права доступа (user:role:type:level). Enforcing/permissive mode контролируют применение политик.

---

### 30. SSL/TLS в Linux

**Концепция:** SSL/TLS обеспечивает шифрование и аутентификацию для сетевых соединений. Сертификаты подтверждают подлинность серверов.

**Компоненты:**
- Private/Public keys - криптографические ключи
- Certificate - цифровой сертификат
- CA (Certificate Authority) - центр сертификации

**Практическое задание:**
```bash
# Настройка nginx с Let's Encrypt
# 1. Установка certbot
apt install certbot python3-certbot-nginx

# 2. Получение сертификата
certbot --nginx -d example.com

# 3. Автообновление
echo "0 12 * * * /usr/bin/certbot renew --quiet" | crontab

# Проверка сертификата
openssl s_client -connect example.com:443 -servername example.com
```

**Углубление:** Certificate chain включает промежуточные сертификаты до корневого CA. SNI позволяет использовать несколько SSL сертификатов на одном IP.

---

### 31. Система логирования

**Концепция:** Логирование записывает события системы и приложений для мониторинга, отладки и безопасности.

**Компоненты:**
- rsyslog - основной демон логирования
- Facilities - источники сообщений (mail, auth, etc)
- Priorities - уровни важности (debug, info, warn, error)

**Практическое задание:**
```bash
# Настройка удаленного логирования
# На клиенте (/etc/rsyslog.conf):
*.* @@logserver.example.com:514

# На сервере:
module(load="imudp")
input(type="imudp" port="514")

# Перезапуск rsyslog
systemctl restart rsyslog

# Тестирование
logger -p mail.info "Test message"
```

**Углубление:** rsyslog может фильтровать, маршрутизировать и форматировать логи. Facilities группируют источники, priorities определяют важность.

---

### 32. fail2ban и защита от брутфорса

**Концепция:** fail2ban автоматически блокирует IP-адреса, которые показывают злонамеренное поведение (например, множественные неудачные попытки входа).

**Принцип работы:**
- Мониторинг лог-файлов
- Обнаружение паттернов атак
- Автоматическое создание iptables правил

**Практическое задание:**
```bash
# Установка и настройка fail2ban
apt install fail2ban

# Создание локальной конфигурации (/etc/fail2ban/jail.local):
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600

# Запуск и проверка
systemctl enable fail2ban
systemctl start fail2ban
fail2ban-client status sshd
```

**Углубление:** Jails - конфигурации для конкретных сервисов. fail2ban интегрируется с iptables для создания правил блокировки.

---

## 📜 Bash и автоматизация

### 33. Переменные и expansion в bash

**Концепция:** Bash поддерживает различные типы переменных и множество способов их расширения (expansion).

**Типы expansion:**
- Parameter expansion - ${var}
- Command substitution - $(command)
- Arithmetic expansion - $((expr))
- Brace expansion - {a,b,c}

**Практическое задание:**
```bash
#!/bin/bash
# Скрипт с default values

DB_HOST=${1:-localhost}
DB_PORT=${2:-5432}
DB_NAME=${3:-myapp}

echo "Connecting to ${DB_HOST}:${DB_PORT}/${DB_NAME}"

# Parameter expansion примеры:
FILE="/path/to/file.txt"
echo ${FILE##*/}     # file.txt (basename)
echo ${FILE%.*}      # /path/to/file (без расширения)
echo ${FILE:-default} # значение по умолчанию
```

**Углубление:** $* объединяет аргументы в одну строку, $@ сохраняет их как отдельные элементы. Parameter expansion позволяет манипулировать строками внутри переменных.

---

### 34. Pipes, redirection и command substitution

**Концепция:** Bash предоставляет мощные механизмы для связывания команд и управления потоками данных.

**Основные операторы:**
- | - pipe (передача вывода)
- >, >> - перенаправление вывода
- <, << - перенаправление ввода
- $() - command substitution

**Практическое задание:**
```bash
# Найти файлы > 100MB, сохранить список и ошибки отдельно
find / -type f -size +100M 2>/tmp/errors.log 1>/tmp/large_files.log

# Альтернативный способ с отдельной обработкой
find / -type f -size +100M > >(tee /tmp/large_files.log) 2> >(tee /tmp/errors.log >&2)

# Command substitution
CURRENT_DATE=$(date +%Y-%m-%d)
DISK_USAGE=$(df -h / | awk 'NR==2 {print $5}')
```

**Углубление:** Process substitution <() создает временный файл с выводом команды. Named pipes (mkfifo) позволяют межпроцессное взаимодействие.

---

### 35. Обработка ошибок в bash

**Концепция:** Правильная обработка ошибок делает скрипты надежными и предсказуемыми.

**Механизмы обработки:**
- Exit codes - коды возврата команд
- set options - изменение поведения shell
- trap - обработчики сигналов

**Практическое задание:**
```bash
#!/bin/bash
set -euo pipefail  # exit on error, undefined vars, pipe failures

LOG_FILE="/var/log/myscript.log"

# Функция логирования
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Trap для очистки при ошибке
cleanup() {
    log "ERROR: Script failed on line $1"
    # Очистка временных файлов
    rm -f /tmp/script_temp.*
}
trap 'cleanup $LINENO' ERR

# Пример с проверкой ошибок
if ! mkdir -p /tmp/mydir; then
    log "ERROR: Cannot create directory"
    exit 1
fi

log "Script completed successfully"
```

**Углубление:** `set -euo pipefail` включает строгий режим. trap handlers выполняются при получении сигналов или ошибках.

---

### 36. Регулярные выражения

**Концепция:** Регулярные выражения - мощный инструмент для поиска и обработки текстовых паттернов.

**Основные метасимволы:**
- . - любой символ
- * - 0 или более повторений
- + - 1 или более повторений
- ? - 0 или 1 повторение
- [] - класс символов

**Практическое задание:**
```bash
# Поиск IP адресов в логе и подсчет запросов
LOG_FILE="/var/log/apache2/access.log"

# Regex для IP адреса
IP_REGEX="[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}"

# Извлечение и подсчет
grep -oE "$IP_REGEX" "$LOG_FILE" | sort | uniq -c | sort -nr | head -10

# Альтернативный способ с awk
awk '{print $1}' "$LOG_FILE" | sort | uniq -c | sort -nr | head -10
```

**Углубление:** BRE (Basic) vs ERE (Extended) регулярные выражения. Lookahead/lookbehind поддерживаются в PCRE, но не в стандартных Unix утилитах.

---

### 37. sed и awk для обработки текста

**Концепция:** sed - потоковый редактор для простых преобразований. awk - язык программирования для сложной обработки текста.

**sed основы:**
- s/pattern/replacement/ - замена
- d - удаление строк
- p - вывод строк

**awk основы:**
- Обработка по полям
- Встроенные переменные (NF, NR, FS)
- Ассоциативные массивы

**Практическое задание:**
```bash
# Преобразование CSV в JSON с awk
cat data.csv | awk -F',' '
BEGIN { 
    print "[" 
    first = 1
}
NR > 1 {
    if (!first) print ","
    printf "  {\"name\":\"%s\", \"age\":%s, \"city\":\"%s\"}", $1, $2, $3
    first = 0
}
END { 
    print "\n]" 
}'

# sed для замены в файлах
sed -i 's/old_domain\.com/new_domain.com/g' /etc/nginx/sites-available/*
```

**Углубление:** Pattern space в sed - буфер для текущей строки. Ассоциативные массивы в awk позволяют группировать и агрегировать данные.

---

### 38. cron и systemd timers

**Концепция:** Планировщики задач позволяют автоматически выполнять команды по расписанию.

**cron формат:**
```
# min hour day month weekday command
# *   *    *   *     *       command
```

**systemd timers преимущества:**
- Лучшее логирование
- Зависимости от других сервисов
- Более гибкие настройки времени

**Практическое задание:**
```bash
# cron backup скрипт (каждую ночь в 2:30)
echo "30 2 * * * /usr/local/bin/backup.sh" | crontab

# systemd timer
# /etc/systemd/system/backup.timer
[Unit]
Description=Daily backup

[Timer]
OnCalendar=daily 2:30
Persistent=true

[Install]
WantedBy=timers.target

# /etc/systemd/system/backup.service
[Unit]
Description=Backup service

[Service]
ExecStart=/usr/local/bin/backup.sh

# Активация
systemctl enable backup.timer
systemctl start backup.timer
```

**Углубление:** systemd timers не зависят от crond и предоставляют лучшую интеграцию с systemd. cron использует отдельное окружение с ограниченными переменными.

---

### 39. Configuration Management

**Концепция:** Infrastructure as Code позволяет управлять конфигурацией серверов через код, обеспечивая повторяемость и версионирование.

**Принципы:**
- Idempotence - повторное выполнение безопасно
- Declarative - описание желаемого состояния
- Version control - хранение в git

**Практическое задание:**
```yaml
# Ansible playbook для nginx
---
- hosts: webservers
  become: yes
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Copy nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: restart nginx

    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

**Углубление:** Idempotence означает, что повторное выполнение даст тот же результат. Версионирование инфраструктуры через git позволяет откатывать изменения.

---

### 40. Контейнеры и Docker

**Концепция:** Контейнеры обеспечивают изоляцию приложений на уровне ОС, используя ядро хост-системы.

**Docker компоненты:**
- Image - неизменяемый шаблон
- Container - работающий экземпляр
- Dockerfile - инструкции для сборки
- Registry - хранилище образов

**Практическое задание:**
```dockerfile
# Multi-stage Dockerfile для Node.js
# Build stage
FROM node:16-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage  
FROM node:16-alpine
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001
WORKDIR /app
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --chown=nextjs:nodejs . .
USER nextjs
EXPOSE 3000
CMD ["node", "server.js"]
```

**Углубление:** Docker networking создает изолированные сети для контейнеров. Container runtime (containerd, CRI-O) отвечает за выполнение контейнеров.

---

## 🎯 Заключение

Это руководство покрывает все ключевые области Linux знаний для backend разработчиков. Каждый вопрос включает теоретическую основу, практические примеры и углубленные детали для различных уровней подготовки.

**Ключевые принципы для успешного интервью:**
- Объясняйте концепции простыми словами
- Приводите практические примеры из опыта
- Показывайте понимание взаимосвязей между компонентами
- Демонстрируйте навыки troubleshooting
- Знайте современные практики и инструменты