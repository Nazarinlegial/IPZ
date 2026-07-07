# ДЗ Модуль 6  Bash-скрипт: бекап логів (Варіант A)

Студент: Ковальчук Назар

Варіант: **A** — скрипт бекапу логів.

---

## Код скрипта `backup.sh`

```bash
#!/bin/bash
# backup.sh — архівування логів
# Використання: ./backup.sh <log_dir> <backup_dir>

# 1. Перевірка аргументів: рівно 2, обидва — існуючі каталоги
if [ "$#" -ne 2 ]; then
    echo "Usage: ./backup.sh <log_dir> <backup_dir>"
    exit 1
fi
LOG_DIR="$1"
BACKUP_DIR="$2"
if [ ! -d "$LOG_DIR" ] || [ ! -d "$BACKUP_DIR" ]; then
    echo "Usage: ./backup.sh <log_dir> <backup_dir>"
    exit 1
fi

# 2. Захист від паралельного запуску через lock-файл
LOCK_FILE="/tmp/backup.lock"
if [ -f "$LOCK_FILE" ]; then
    echo "Backup already running"
    exit 0
fi
touch "$LOCK_FILE"
trap 'rm -f "$LOCK_FILE"' EXIT

# 3. Створення архіву: ім'я з датою і часом
TIMESTAMP=$(date "+%Y-%m-%d_%H-%M")
ARCHIVE="$BACKUP_DIR/logs_backup_${TIMESTAMP}.tar.gz"

# 4. Перевірка результату архівування
if tar -czf "$ARCHIVE" -C "$LOG_DIR" . ; then
    echo "Backup created: $ARCHIVE"
else
    echo "Backup failed"
    exit 2
fi
```

## Опис, що відбувається

Скрипт `backup.sh` стискає всі файли з каталогу логів у `.tar.gz` архів з датою і часом у назві та кладе його в каталог бекапів.

1. **Перевірка аргументів** — має бути рівно 2, обидва існуючі каталоги (`-d`). Інакше виводиться `Usage` і `exit 1`.
2. **Захист від паралельного запуску** — lock-файл `/tmp/backup.lock`. Якщо існує — `Backup already running` + `exit 0`. Інакше створюється lock і ставиться `trap`, що видалить його при будь-якому виході (успіх, помилка, Ctrl+C).
3. **Створення архіву** — `tar -czf` з ім'ям `logs_backup_YYYY-MM-DD_HH-MM.tar.gz`. `-C "$LOG_DIR" .` архівує вміст каталогу без повного шляху.
4. **Перевірка результату** — при помилці `tar` виводиться `Backup failed` + `exit 2`; при успіху — повний шлях до архіву.

---

## Перевірка з різними аргументами

Перед запусками скрипт зроблено виконуваним: `chmod +x backup.sh`. Каталоги: `logs/` (з `app.log`, `error.log`), `backup/`.

**1. Без аргументів**

```bash
./backup.sh
```
```
Usage: ./backup.sh <log_dir> <backup_dir>
exit code = 1
```

**2. Один аргумент**

```bash
./backup.sh logs
```
```
Usage: ./backup.sh <log_dir> <backup_dir>
exit code = 1
```

**3. Неіснуючий каталог**

```bash
./backup.sh /no/such/dir backup
```
```
Usage: ./backup.sh <log_dir> <backup_dir>
exit code = 1
```

**4. Файл замість каталогу**

```bash
./backup.sh logs/app.log backup
```
```
Usage: ./backup.sh <log_dir> <backup_dir>
exit code = 1
```

**5. Коректний запуск**

```bash
./backup.sh logs backup
```
```
Backup created: backup/logs_backup_2026-07-07_22-22.tar.gz
exit code = 0
```

Перевірка вмісту архіву:

```bash
tar -tzf backup/logs_backup_2026-07-07_22-22.tar.gz
```
```
./
./app.log
./error.log
```

**6. Паралельний запуск (lock-файл)**

```bash
touch /tmp/backup.lock
./backup.sh logs backup
```
```
Backup already running
exit code = 0
```

Усі перевірки пройшли коректно: помилкові аргументи дають `Usage` + код `1`, успішний запуск створює архів (код `0`), наявний lock-файл блокує повторний запуск.
