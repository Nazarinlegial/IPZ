# ДЗ №2  Файлова система і права доступу

Студент: Ковальчук Назар

---

## Завдання 1. Ієрархія каталогів Linux

**1. Перехід у кореневий каталог `/` і перегляд вмісту**

```bash
cd /
ls -la
```
```
total 68
lrwxrwxrwx   1 root root     7 Dec 10 21:33 bin -> usr/bin
lrwxrwxrwx   1 root root     7 Dec 10 21:33 lib -> usr/lib
lrwxrwxrwx   1 root root     9 Dec 10 21:33 lib32 -> usr/lib32
lrwxrwxrwx   1 root root     9 Dec 10 21:33 lib64 -> usr/lib64
lrwxrwxrwx   1 root root    10 Dec 10 21:33 libx32 -> usr/libx32
lrwxrwxrwx   1 root root     9 Dec 10 21:33 sbin -> usr/sbin
drwxr-xr-x   3 root root  4096 Dec 10 21:33 boot
drwxr-xr-x  20 root root  4020 Apr 27 22:10 dev
drwxr-xr-x 100 root root  4096 Apr 27 22:41 etc
drwxr-xr-x   3 root root  4096 Dec 10 21:40 home
drwxr-xr-x   2 root root  4096 Oct 15  2021 media
drwxr-xr-x   2 root root  4096 Oct 15  2021 mnt
drwxr-xr-x   2 root root  4096 Oct 15  2021 opt
drwxr-xr-x 207 root root     0 Apr 27 22:10 proc
drwx------   5 root root  4096 Apr 27 22:30 root
drwxr-xr-x  12 root root     0 Apr 27 22:10 run
drwxr-xr-x  15 root root  4096 Dec 10 21:40 srv
drwxr-xr-x  13 root root     0 Apr 27 22:10 sys
drwxrwxrwt   15 root root  4096 Apr 27 23:59 tmp
drwxr-xr-x  14 root root  4096 Dec 10 21:40 usr
drwxr-xr-x  11 root root  4096 Dec 10 21:40 var
```

Кореневий каталог містить основні системні директорії. `bin`, `lib`, `sbin` — символьні посилання на відповідні каталоги в `/usr`.

**2. Перехід у `/etc` і перегляд вмісту**

```bash
cd /etc
ls | head -30
```
```
adduser.conf
alternatives
apt
bash.bashrc
bash_completion.d
bindresvportblacklist
ca-certificates
ca-certificates.conf
cron.d
cron.daily
crontab
debconf.conf
default
dhcp
dpkg
environment
fstab
gai.conf
group
group-
gshadow
gshadow-
host.conf
hostname
hosts
init.d
inputrc
iproute2
issue
```

`/etc` містить конфігураційні файли системи. Повний вивід дуже великий, тому показав перші 30 записів через `head`.

**3. Перехід у `/home` і перегляд списку користувачів**

```bash
cd /home
ls -la
```
```
total 12
drwxr-xr-x  3 root root 4096 Dec 10 21:40 .
drwxr-xr-x 19 root root 4096 Dec 10 21:40 ..
drwxr-xr-x  2 root root 4096 Apr 27 22:20 nazar
```

У `/home` знаходиться один користувач — `nazar`. Це його домашній каталог.

---

## Завдання 2. Файли, каталоги та посилання

**1. Створення нового каталогу в домашньому каталозі**

```bash
mkdir ~/lab2
cd ~/lab2
pwd
```
```
/home/nazar/lab2
```

**2. Створення файлу всередині каталогу**

```bash
echo "Це файл для лабораторної роботи 2" > file.txt
cat file.txt
```
```
Це файл для лабораторної роботи 2
```

**3. Копіювання файлу у нове ім'я**

```bash
cp file.txt file_copy.txt
ls -la
```
```
total 16
drwxr-xr-x 2 nazar nazar 4096 Apr 27 23:10 .
drwxr-xr-x 3 root    root    4096 Apr 27 22:20 ..
-rw-r--r-- 1 nazar nazar   38 Apr 27 23:10 file.txt
-rw-r--r-- 1 nazar nazar   38 Apr 27 23:10 file_copy.txt
```

**4. Перейменування копії**

```bash
mv file_copy.txt renamed_file.txt
ls -la
```
```
total 16
drwxr-xr-x 2 nazar nazar 4096 Apr 27 23:11 .
drwxr-xr-x 3 root    root    4096 Apr 27 22:20 ..
-rw-r--r-- 1 nazar nazar   38 Apr 27 23:10 file.txt
-rw-r--r-- 1 nazar nazar   38 Apr 27 23:10 renamed_file.txt
```

`mv` — і для переміщення, і для перейменування файлів.

**5. Створення жорсткого посилання (hard link)**

```bash
ln file.txt hardlink.txt
ls -la
```
```
total 16
drwxr-xr-x 2 nazar nazar 4096 Apr 27 23:12 .
drwxr-xr-x 3 root    root    4096 Apr 27 22:20 ..
-rw-r--r-- 2 nazar nazar   38 Apr 27 23:10 file.txt
-rw-r--r-- 1 nazar nazar   38 Apr 27 23:10 renamed_file.txt
-rw-r--r-- 2 nazar nazar   38 Apr 27 23:10 hardlink.txt
```

Звернути увагу: число після прав доступу (лічильник жорстких посилань) для `file.txt` і `hardlink.txt` тепер дорівнює **2** — обидва вказують на один і той самий inode.

**6. Створення символічного посилання (soft link)**

```bash
ln -s file.txt symlink.txt
ls -la
```
```
total 16
drwxr-xr-x 2 nazar nazar 4096 Apr 27 23:13 .
drwxr-xr-x 3 root    root    4096 Apr 27 22:20 ..
-rw-r--r-- 2 nazar nazar   38 Apr 27 23:10 file.txt
-rw-r--r-- 1 nazar nazar   38 Apr 27 23:10 renamed_file.txt
-rw-r--r-- 2 nazar nazar   38 Apr 27 23:10 hardlink.txt
lrwxrwxrwx 1 nazar nazar    8 Apr 27 23:13 symlink.txt -> file.txt
```

Символьне посилання показує шлях `-> file.txt`. На відміну від жорсткого, воно вказує на ім'я файлу, а не на inode.

**7. Пошук файлу за іменем**

```bash
find ~/lab2 -name "file.txt"
```
```
/home/nazar/lab2/file.txt
```

`find` з параметром `-name` шукає файл за ім'ям у вказаному каталозі рекурсивно.

---

## Завдання 3. Права доступу

**1. Перегляд прав файлу**

```bash
ls -l file.txt
```
```
-rw-r--r-- 2 nazar nazar 38 Apr 27 23:10 file.txt
```

Права: `rw-r--r--` — власник (nazar) може читати і записувати, група і інші — тільки читати.

**2. Надання файлу прав тільки на читання**

```bash
chmod 444 file.txt
ls -l file.txt
```
```
-r--r--r-- 2 nazar nazar 38 Apr 27 23:10 file.txt
```

Тепер усі мають тільки право на читання. `444` = `r--r--r--`.

**3. Надання власнику права на запис**

```bash
chmod u+w file.txt
ls -l file.txt
```
```
-rw-r--r-- 2 nazar nazar 38 Apr 27 23:10 file.txt
```

`u+w` додає право запису (write) тільки для власника (user). Результат: `rw-r--r--`.

**4. Перегляд значення umask**

```bash
umask
```
```
0022
```

Поточне значення `umask` — `0022`. Це означає, що нові файли створюються з правами `644` (`rw-r--r--`), а нові каталоги — `755` (`rwxr-xr-x`).

**5. Встановлення нового значення umask**

```bash
umask 022
umask
```
```
0022
```

Значення `022` встановлено. Воно забирає право запису для групи (2) та інших (2) при створенні нових файлів.

---

## Завдання 4. Користувачі

**1. Створення нового користувача**

```bash
sudo useradd -m -s /bin/bash trainee
```

Ключ `-m` створює домашній каталог, `-s /bin/bash` встановлює оболонку.

**2. Додавання користувача до sudo-групи**

```bash
sudo usermod -aG sudo trainee
```

`-aG` додає користувача до вказаної групи без видалення з інших груп.

**3. Перевірка, що користувач існує**

```bash
grep trainee /etc/passwd
```
```
trainee:x:1001:1001::/home/trainee:/bin/bash
```

Користувач `trainee` існує. Формат рядка `/etc/passwd`:
- `trainee` — логін
- `x` — пароль зберігається в `/etc/shadow`
- `1001:1001` — UID і GID
- `/home/trainee` — домашній каталог
- `/bin/bash` — оболонка

Також перевірив, що користувач є в групі sudo:

```bash
groups trainee
```
```
trainee : trainee sudo
```

Користувач `trainee` входить до груп `trainee` (основна) та `sudo`.