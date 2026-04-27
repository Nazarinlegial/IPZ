# ДЗ №1  Основи Linux і робота в терміналі

Студент: Ковальчук Назар

---

## Завдання 1

**1. Вміст домашнього каталогу**

```bash
ls -la ~
```
```
total 64
drwx------ 10 root root 4096 Dec 21 12:43 .
drwxr-xr-x 23 root root 4096 Dec 10 21:40 ..
drwxr-xr-x  5 root root 4096 Apr 27 22:14 .antigravity-server
-rw-r--r--  1 root root 4865 Jan  5 01:48 .bash_history
-rw-r--r--  1 root root 3222 Dec 10 21:40 .bashrc
drwx------  6 root root 4096 Dec 21 12:43 .cache
-rw-r--r--  1 root root    0 Jan  5  2025 .cloud-locale-test.skip
drwx------  3 root root 4096 Dec 11 00:34 .docker
drwxr-xr-x  3 root root 4096 Dec 10 21:29 .gemini
drwxr-xr-x  3 root root 4096 Dec 21 12:43 .java
drwx------  2 root root 4096 Apr 27 22:14 .keychain
-rw-------  1 root root   20 Dec 11 00:12 .lesshst
drwxr-xr-x  3 root root 4096 Dec 10 21:39 .local
-rw-r--r--  1 root root  252 Dec 21 12:43 .profile
-rw-r--r--  1 root root  161 Apr 22  2024 .profile.bak
drwx------  2 root root 4096 Dec 10 21:50 .ssh
```

**2. Переходжу в Downloads, дивлюсь що там**

```bash
cd ~/Downloads
```
```
bash: cd: /root/Downloads: No such file or directory
```

Каталогу нема (на сервері його нема за замовчуванням), створюю і переходжу:

```bash
mkdir ~/Downloads
cd ~/Downloads
ls -la
```
```
total 8
drwxr-xr-x  2 root root 4096 Apr 27 22:20 .
drwx------ 11 root root 4096 Apr 27 22:20 ..
```

Порожній, тільки створився.

**3. Створюю порожній файл без touch**

```bash
> testfile.txt
```

Тут використовую оператор перенаправлення `>` без команди зліва — він просто створює порожній файл.

**4. Переглядаю вміст файлу**

```bash
echo "Hello, Linux!" > testfile.txt
cat testfile.txt
```
```
Hello, Linux!
```

Спочатку записав текст у файл через `echo`, потім переглянув через `cat`.

**5. Перехід у домашній каталог абсолютним шляхом**

```bash
cd /root
pwd
```
```
root@staging-crm:~# 
```

Абсолютний шлях - це повний шлях від кореня `/`.

**6. Перехід у домашній каталог відносним шляхом**

```bash
cd ~/Downloads   # спершу пішов кудись
cd ..            # повернувся на рівень вгору
pwd
```
```
root@staging-crm:~#
```

`..` - це відносний шлях до батьківського каталогу.

---

## Завдання 2

Виконав команди:
```bash
man ls
help cd
man cat
man man
cp --help
mv --help
```

`man ls`, `man cat`, `man man` - на сервері man-сторінки не повністю встановлені, тому man відкривався але з помилками:
```
[1]+  Stopped                 man ls
[2]+  Stopped                 man cat
man: can't resolve man7/groff_man.7
[3]+  Stopped                 man man
```

`help cd` - спрацював нормально, показав повну довідку по cd (це вбудована команда bash, тому help працює без man).

`cp --help` і `mv --help` - спрацювали без проблем, показали повний список ключів.

Щоб подивитись ключі ls і cat без man, використав `--help`:
```bash
ls --help
cat --help
```

**1. Який ключ ls показує приховані файли?**

`-a` (або `--all`). Показує файли що починаються з крапки. Є ще `-A` - те саме, тільки без `.` та `..`.

**2. Який ключ у cat нумерує рядки?**

`-n` (або `--number`). Є ще `-b` який нумерує тільки непорожні рядки.

```bash
cat -n testfile.txt
```
```
     1  Hello, Linux!
```

**3. Чим відрізняються man і --help?**

`man` - це окрема програма, яка відкриває повну документацію по команді. Там є і опис, і всі ключі, і приклади, і посилання на інші сторінки. Відкривається в інтерактивному режимі де можна скролити і шукати текст через `/`. Але на мінімальних серверних збірках man-сторінки можуть бути не встановлені (як у мене на staging-crm).

`--help` - це просто прапор самої команди, який виводить коротку довідку прямо в термінал. Зазвичай там тільки список основних ключів і формат виклику. Працює завжди, навіть якщо man не встановлений.

---

## Завдання 3

Міні-сценарій:

```bash
# створюю робочий каталог і заходжу в нього
mkdir -p ~/projects/demo
cd ~/projects/demo

# створюю файл з текстом
echo "Це демонстраційний файл" > notes.txt

# дивлюсь що у файлі
cat notes.txt

# дивлюсь що в каталозі
ls -la

# переходжу в /tmp
cd /tmp
pwd

# повертаюсь назад
cd ~/projects/demo
pwd

# дивлюсь довідку по mkdir
mkdir --help
```

Вивід:
```
Це демонстраційний файл

total 12
drwxr-xr-x 2 root root 4096 Apr 27 22:39 .
drwxr-xr-x 3 root root 4096 Apr 27 22:39 ..
-rw-r--r-- 1 root root   45 Apr 27 22:39 notes.txt

/tmp

/root/projects/demo

Usage: mkdir [OPTION]... DIRECTORY...
Create the DIRECTORY(ies), if they do not already exist.

Mandatory arguments to long options are mandatory for short options too.
  -m, --mode=MODE   set file mode (as in chmod), not a=rwx - umask
  -p, --parents     no error if existing, make parent directories as needed,
                    with their file modes unaffected by any -m option.
  -v, --verbose     print a message for each created directory
  -Z                   set SELinux security context of each created directory
                         to the default type
      --context[=CTX]  like -Z, or if CTX is specified then set the SELinux
                         or SMACK security context to CTX
      --help        display this help and exit
      --version     output version information and exit

GNU coreutils online help: <https://www.gnu.org/software/coreutils/>
Full documentation <https://www.gnu.org/software/coreutils/mkdir>
or available locally via: info '(coreutils) mkdir invocation'
```

---

## Бонус: аналоги для PowerShell (Windows)

Оскільки в мене Windows, додаю відповідники основних команд для PowerShell, бо в ентерпрайзі це досі актуально:

`ls` → `Get-ChildItem` (аліас `ls`, `dir`). Приховані файли: `ls -Force` замість `ls -a`.

`cd` → `Set-Location` (аліас `cd`). Працює так само, тільки шляхи через `\` замість `/`.

`pwd` → `Get-Location` (аліас `pwd`).

`cat` → `Get-Content` (аліас `cat`, `type`).

`mkdir` → `New-Item -ItemType Directory` (аліас `mkdir`).

Створити файл без touch: `$null > file.txt` або `New-Item file.txt`.

`echo "text" > file` → `Set-Content file.txt "text"`.

`cp` → `Copy-Item`, `mv` → `Move-Item`, `rm` → `Remove-Item`.

Довідка: замість `man` і `--help` є `Get-Help <cmdlet>`. Прапор `-Full` дає повну доку, `-Examples` — тільки приклади.

PowerShell-аліаси спеціально зроблені схожими на Linux-команди, але поводяться інакше. Наприклад `ls -la` в PowerShell не спрацює — треба `ls -Force`. Тому покладатись тільки на аліаси не варто, краще знати реальні cmdlet-и.