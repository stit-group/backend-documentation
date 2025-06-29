# Linux System Administration Interview - 2025

## Управление процессами и потоками

### 1. Чем отличается процесс от потока (thread) в Linux?
**Подсказка:** В Linux threads создаются через `clone()` system call и отображаются как LWP (Light Weight Process)

**Углубление:** 
- Объясните, что такое NLWP в выводе `ps -eLf`
- Почему в Linux потоки видны как отдельные процессы?
- Как посмотреть количество потоков процесса?

---

### 2. Как посмотреть все процессы с количеством потоков?
**Подсказка:** `ps -eLf` показывает все потоки, `ps -o pid,ppid,nlwp,comm` показывает количество потоков

**Углубление:**
- Что означает NLWP в выводе?
- Чем LWP отличается от PID?
- Как найти процессы с наибольшим количеством потоков?

---

### 3. Объясните состояния процессов в Linux
**Подсказка:** R (Running), S (Sleeping), D (Uninterruptible sleep), T (Stopped), Z (Zombie)

**Углубление:**
- Когда процесс попадает в состояние D и почему это опасно?
- Как избавиться от zombie процессов?
- Что такое TASK_KILLABLE состояние (появилось в новых ядрах)?

---

### 4. Как ограничить количество потоков для процесса?
**Подсказка:** `ulimit -u`, `/proc/sys/kernel/threads-max`, systemd limits

**Углубление:**
- Какие факторы влияют на максимальное количество потоков?
- Как настроить limits в systemd unit файлах?
- Что такое `RLIMIT_NPROC` vs `threads-max`?

---

## Systemd и управление сервисами

### 5. Как создать systemd service unit для многопоточного приложения?
**Подсказка:** Type=forking vs Type=simple, ExecStart, Restart policies

**Углубление:**
- Когда использовать Type=notify vs Type=simple?
- Как настроить restart policies для критичных сервисов?
- Что такое systemd socket activation?

---

### 6. Как запустить N экземпляров одного сервиса через systemd?
**Подсказка:** Template units с символом `@`, targets для группового управления

**Углубление:**
- Создайте service template с переменной %i
- Как создать target для управления группой сервисов?
- Чем PartOf отличается от WantedBy?

---

### 7. Как посмотреть и управлять cgroups v2 в современном Linux?
**Подсказка:** `systemctl show`, `systemd-cgtop`, `/sys/fs/cgroup/`

**Углубление:**
- Какие controllers доступны в cgroups v2?
- Как ограничить память и CPU для сервиса в systemd?
- Что изменилось в cgroups v2 по сравнению с v1?

---

### 8. Объясните процесс загрузки Linux с systemd
**Подсказка:** kernel → systemd (PID 1) → default.target → multi-user.target

**Углубление:**
- Что такое systemd targets и чем они отличаются от runlevels?
- Как systemd распараллеливает процесс загрузки?
- Объясните dependency resolution в systemd

---

## Мониторинг и производительность

### 9. Какие современные инструменты мониторинга процессов вы знаете?
**Подсказка:** htop, iotop, iftop, ss, lsof, systemd-cgtop, bpftrace

**Углубление:**
- Чем ss лучше netstat в 2025 году?
- Как использовать eBPF для мониторинга системы?
- Что такое BCC tools и bpftrace?

---

### 10. Как диагностировать высокую load average?
**Подсказка:** Проверить CPU usage, iowait, количество процессов в состоянии D

**Углубление:**
- Почему load average может быть высоким при низком CPU usage?
- Как найти процессы в uninterruptible sleep?
- Что такое pressure stall information (PSI)?

---

### 11. Как найти процесс, потребляющий много файловых дескрипторов?
**Подсказка:** `lsof`, `/proc/PID/fd/`, `ls -la /proc/*/fd | wc -l`

**Углубление:**
- Как настроить limits для файловых дескрипторов?
- В чем разница между soft и hard limits?
- Как мониторить утечки FD в production?

---

### 12. Объясните разницу между RSS, VSZ и PSS в памяти процессов
**Подсказка:** RSS - физическая память, VSZ - виртуальная, PSS - пропорциональная

**Углубление:**
- Что такое USS (Unique Set Size)?
- Как shared memory влияет на эти метрики?
- Когда использовать каждую метрику для анализа?

---

## Сети и безопасность

### 13. Как настроить iptables firewall с помощью systemd?
**Подсказка:** iptables-save/restore, systemd unit files, netfilter-persistent

**Углубление:**
- Как мигрировать с iptables на nftables?
- Что такое iptables-nft backend?
- Как интегрировать firewall rules с container orchestration?

---

### 14. Объясните socket activation в systemd
**Подсказка:** systemd может слушать сокеты и запускать сервисы по требованию

**Углубление:**
- Создайте .socket unit файл
- Как это помогает с производительностью?
- Какие типы сокетов поддерживает systemd?

---

### 15. Как настроить SELinux в современном Linux?
**Подсказка:** sestatus, setsebool, audit2allow, современные политики

**Углубление:**
- Чем SELinux отличается от AppArmor?
- Как создать custom SELinux policy?
- Как SELinux интегрируется с containers?

---

## Файловые системы и хранение

### 16. Что нового в файловых системах Linux в 2025 году?
**Подсказка:** Btrfs snapshots, ZFS on Linux, ext4 с encryption, XFS improvements

**Углубление:**
- Сравните Btrfs и ZFS для production use
- Что такое fs-verity и fscrypt?
- Как работает Copy-on-Write в современных FS?

---

### 17. Как правильно настроить LVM для production?
**Подсказка:** Планирование, snapshots, monitoring, performance tuning

**Углубление:**
- Как создать автоматические LVM snapshots?
- Что такое LVM thin provisioning?
- Как мигрировать данные между PV без downtime?

---

### 18. Объясните современные методы резервного копирования в Linux
**Подсказка:** rsync, borgbackup, restic, filesystem snapshots

**Углубление:**
- Чем incremental backup отличается от differential?
- Как использовать deduplication в backups?
- Что такое immutable backups и зачем они нужны?

---

## Контейнеризация и виртуализация

### 19. Как systemd интегрируется с container runtimes?
**Подсказка:** systemd-nspawn, podman, container@.service

**Углубление:**
- Чем podman отличается от docker в плане systemd?
- Что такое systemd portable services?
- Как настроить rootless containers с systemd?

---

### 20. Объясните namespace isolation в Linux
**Подсказка:** PID, Network, Mount, UTS, IPC, User, Cgroup namespaces

**Углубление:**
- Как создать namespace вручную?
- Что такое nested namespaces?
- Как namespaces связаны с security?

---

## Автоматизация и конфигурация

### 21. Как автоматизировать задачи администрирования в 2025?
**Подсказка:** Ansible, Terraform, systemd timers вместо cron

**Углубление:**
- Чем systemd timers лучше cron?
- Как использовать Ansible для immutable infrastructure?
- Что такое GitOps подход в Linux admin?

---

### 22. Объясните cloud-init и его использование
**Подсказка:** Автоматическая конфигурация cloud instances

**Углубление:**
- Как создать custom cloud-init config?
- Интеграция с различными cloud providers
- Что такое cloud-config vs user-data?

---

### 23. Как настроить централизованное логирование с systemd?
**Подсказка:** journald, rsyslog, forwarding в ELK/Grafana

**Углубление:**
- Настройка журнала systemd для production
- Как форвардить journal logs в external systems?
- Structured logging и metadata в journald

---

## Производительность и тюнинг

### 24. Как настроить system tuning для высоких нагрузок?
**Подсказка:** sysctl parameters, CPU governors, I/O schedulers

**Углубление:**
- Какие sysctl параметры критичны для web servers?
- Как выбрать правильный I/O scheduler?
- Что такое CPU isolation и когда его использовать?

---

### 25. Объясните NUMA и её влияние на производительность
**Подсказка:** numactl, numa balancing, memory policies

**Углубление:**
- Как диагностировать NUMA-related problems?
- Настройка applications для NUMA awareness
- Что такое automatic NUMA balancing в kernel?

---

### 26. Как мониторить и оптимизировать дисковую подсистему?
**Подсказка:** iostat, iotop, fio, disk schedulers, filesystem tuning

**Углубление:**
- Какие метрики важны для storage performance?
- Как настроить SSD для максимальной производительности?
- Что такое block layer tuning?

---

## Безопасность и hardening

### 27. Какие современные практики hardening Linux вы знаете?
**Подсказка:** Principle of least privilege, mandatory access controls, audit

**Углубление:**
- Как настроить Linux Kernel Runtime Guard (LKRG)?
- Что такое kernel live patching?
- Современные методы защиты от ROP/JOP attacks

---

### 28. Объясните управление сертификатами в Linux
**Подсказка:** Let's Encrypt, certbot, cert-manager, PKI

**Углубление:**
- Автоматическое обновление сертификатов
- Integration с service discovery
- Certificate transparency и monitoring

---

### 29. Как настроить network security monitoring?
**Подсказка:** fail2ban, intrusion detection, network segmentation

**Углубление:**
- Современные IDS/IPS solutions для Linux
- Что такое eBPF-based security monitoring?
- Container network security

---

## Troubleshooting и диагностика

### 30. Опишите ваш подход к диагностике проблем производительности
**Подсказка:** Systematic approach, правильные инструменты, baseline measurements

**Углубление:**
- Создайте план диагностики медленного сервера
- Какие метрики собирать постоянно?
- Как использовать flame graphs для анализа?

---

## Дополнительные ресурсы для изучения

- **Brendan Gregg's Blog** - Performance analysis methodologies
- **Red Hat System Administration Guide** - Enterprise best practices  
- **Linux Performance Tools** - Современные инструменты мониторинга
- **systemd.io** - Официальная документация systemd
- **eBPF.io** - Современные возможности observability в Linux

---

*Собеседование составлено с учетом актуальных тенденций Linux администрирования в 2025 году, включая современные инструменты, безопасность и best practices.*