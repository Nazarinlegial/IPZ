# ДЗ Модуль 5  Мережева діагностика та SSH

Студент: Ковальчук Назар

---

## Завдання 1. Мережева діагностика

**1. IP-адреси та інтерфейси**

```bash
ip a
```
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 ...
    inet 127.0.0.1/8 scope host lo
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 state UP ...
    inet 46.62.225.252/32 metric 100 scope global dynamic eth0
    inet6 2a01:4f9:c013:49f2::1/64 scope global
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> state DOWN
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
24: br-3721dbce46da: <BROADCAST,MULTICAST,UP,LOWER_UP> state UP
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-3721dbce46da
... (ще ~15 інтерфейсів veth* — це мережеві інтерфейси docker-контейнерів)
```

Основний інтерфейс — `eth0` з публічною IPv4 `46.62.225.252` та IPv6 `2a01:4f9:c013:49f2::1`. Docker створює міст `docker0` (`172.17.0.1`) і користувацький міст `br-3721dbce46da` (`172.18.0.1`).

**2. Перевірка доступності публічного вузла**

```bash
ping -c 4 8.8.8.8
```
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=116 time=1.72 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=116 time=1.21 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=116 time=1.28 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=116 time=1.29 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 1.210/1.373/1.719/0.201 ms
```

Усі 4 пакети доставлено, `0% packet loss` — **доступ до інтернету є**.

**3. Перевірка listening-портів**

```bash
ss -tulpn
```
```
tcp  LISTEN  0  4096   0.0.0.0:22        0.0.0.0:*  users:(("sshd",pid=1695423))
tcp  LISTEN  0  4096   0.0.0.0:80        0.0.0.0:*  users:(("docker-proxy",pid=3126152))
tcp  LISTEN  0  4096   0.0.0.0:443       0.0.0.0:*  users:(("docker-proxy",pid=3126168))
tcp  LISTEN  0  4096   0.0.0.0:8000      0.0.0.0:*  users:(("docker-proxy",pid=1834946))
tcp  LISTEN  0  4096   0.0.0.0:5000      0.0.0.0:*  users:(("docker-proxy",pid=2926226))
tcp  LISTEN  0  4096   127.0.0.53%lo:53  0.0.0.0:*  users:(("systemd-resolve",pid=1695437))
... (java, node, language_server — на 127.0.0.1)
```

**Приклад сервісу, що слухає порт:** `sshd` на порту `:22` (SSH-сервер, приймає зовнішні підключення). Також `docker-proxy` пробросив порти `80/443/8000/5000` у контейнери, а `systemd-resolved` обслуговує локальний DNS на `:53`.

### Підсумок Завдання 1
- **IP інтерфейсу:** `eth0` → `46.62.225.252` (публічна); внутрішні — `docker0` `172.17.0.1`, `lo` `127.0.0.1`.
- **Доступ до інтернету:** ✅ так (`ping 8.8.8.8`, `0% packet loss`).
- **Сервіс на порту:** `sshd` на `:22`.

---

## Завдання 2. SSH-доступ з ключами та config

**1. Генерація SSH-ключа**

Ключ `~/.ssh/id_ed25519` вже існує, тому генерацію пропускаємо (умова ДЗ — «якщо ще не існує»). Перевіряю наявність і fingerprint:

```bash
ls -la ~/.ssh/id_ed25519*
ssh-keygen -lf ~/.ssh/id_ed25519.pub
```
```
-rw------- 1 nazar nazar 419 Dec  8  2025 /home/nazar/.ssh/id_ed25519
-rw-r--r-- 1 nazar nazar 110 Dec  8  2025 /home/nazar/.ssh/id_ed25519.pub
256 SHA256:xuELe6Kqz50ZOOs+QA+PtfMFIeVnuhKKaZx0LHEJ3T8 nazar.kovalchuk17@gmail.com (ED25519)
```

Алгоритм `ed25519` — сучасний, короткий і швидкий. Права `600` на приватний ключ — обов'язкові, інакше ssh його відкидає.

**2. Копіювання ключа на сервер**

```bash
ssh-copy-id root@46.62.225.252
```
```
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh root@46.62.225.252"
and check to make sure that only the key(s) you wanted were added.
```

`ssh-copy-id` додає вміст `~/.ssh/id_ed25519.pub` у `/root/.ssh/authorized_keys` на сервері.

**3. Запис у `~/.ssh/config`**

```bash
cat ~/.ssh/config
```
```
Host crm-staging
    HostName 46.62.225.252
    User root
    Port 22
    IdentityFile ~/.ssh/id_ed25519
```

`Host crm-staging` — короткий аліас; `HostName` — IP сервера; `User` — логін; `IdentityFile` — який ключ використовувати для цього хоста.

**4. Підключення короткою командою**

```bash
ssh -o BatchMode=yes crm-staging "hostname && whoami && echo OK_NO_PASSWORD"
```
```
staging-crm
root
OK_NO_PASSWORD
```

**5. Перевірка входу без пароля**

Підключення `ssh crm-staging` спрацювало без запиту пароля — автентифікація пройшла по ключу `id_ed25519`. `BatchMode=yes` гарантує, що ssh узагалі не міг запитати пароль (інакше одразу помилка).

### Підсумок Завдання 2
- **Host у config:** `crm-staging`.
- **Вхід без пароля:** ✅ так, по SSH-ключу.

---

## Завдання 3. Копіювання файлів між машинами

**1. Створення локального тестового файлу**

```bash
mkdir -p ~/m5_local
echo "test" > ~/m5_local/test.txt
cat ~/m5_local/test.txt
```
```
test
```

**2. Передача файлу на сервер через scp**

```bash
ssh crm-staging "mkdir -p /root/m5_remote"
scp ~/m5_local/test.txt crm-staging:/root/m5_remote/test.txt
```
```
test.txt    100%    5     4.9KB/s   00:00
```

Перевірка на сервері:

```bash
ssh crm-staging "ls -la /root/m5_remote/"
```
```
total 12
drwxr-xr-x 2 root root 4096 Jul  7 01:15 .
drwx------ 9 root root 4096 Jul  6 01:00 ..
-rw-r--r-- 1 root root    5 Jul  7 01:15 test.txt
```

**3. Створення директорії для синхронізації та ще файлів локально**

Директорію `/root/m5_remote` на сервері вже створено (крок 2). Додаю файли локально:

```bash
echo "file1" > ~/m5_local/a.txt
echo "file2" > ~/m5_local/b.txt
ls ~/m5_local/
```
```
a.txt  b.txt  test.txt
```

**4. Синхронізація папки через rsync**

```bash
rsync -av ~/m5_local/ crm-staging:/root/m5_remote/
```
```
sending incremental file list
./
a.txt
b.txt

sent 320 bytes  received 77 bytes  264.67 bytes/sec
total size is 16  speedup is 0.04
```

Перевірка результату:

```bash
ssh crm-staging "ls -la /root/m5_remote/"
```
```
total 24
drwxr-xr-x 2 root root 4096 Jul  7 01:16 .
drwx------ 9 root root 4096 Jul  6 01:00 ..
-rw-r--r-- 1 root root    6 Jul  7 01:16 a.txt
-rw-r--r-- 1 root root    6 Jul  7 01:16 b.txt
-rw-r--r-- 1 root root    5 Jul  7 01:15 test.txt
```

`-a` (archive) — рекурсивно зі збереженням прав і часу, `-v` — вивід процесу. Завершальний слеш у `~/m5_local/` важливий: копіюється **вміст** папки, а не сама папка.

**5. Перевірка через sftp**

```bash
sftp -b - crm-staging << 'EOF'
ls m5_remote
bye
EOF
```
```
Connected to crm-staging.
sftp> ls m5_remote
m5_remote/a.txt    m5_remote/b.txt    m5_remote/test.txt
sftp> bye
```

`sftp -b -` читає команди зі stdin. Команда `ls m5_remote` підтверджує, що всі три файли (`a.txt`, `b.txt`, `test.txt`) присутні на сервері.

### Підсумок Завдання 3
- **Шлях до файлів на сервері:** `/root/m5_remote/` (`test.txt`, `a.txt`, `b.txt`).
- **Команди перевірки:** `ssh crm-staging "ls -la /root/m5_remote/"` та `sftp ... ls m5_remote`.
