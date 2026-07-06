# ДЗ №4  Пакети, сервіси та журнали

Студент: Ковальчук Назар

---

## Завдання 1. Менеджери пакетів

**1. Оновлення списку пакетів**

```bash
sudo apt update
```
```
Hit:1 https://download.docker.com/linux/ubuntu noble InRelease
Hit:2 https://mirror.hetzner.com/ubuntu/packages noble InRelease
Hit:3 https://mirror.hetzner.com/ubuntu/packages noble-updates InRelease
Hit:4 https://mirror.hetzner.com/ubuntu/packages noble-backports InRelease
Hit:5 https://mirror.hetzner.com/ubuntu/security noble-security InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
95 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

`apt update` оновлює списки доступних пакетів із репозиторіїв, але нічого не встановлює.

**2. Встановлення утиліти `tree`**

```bash
sudo apt install -y tree
```
```
The following NEW packages will be installed:
  tree
0 upgraded, 1 newly installed, 0 to remove and 95 not upgraded.
Need to get 47,4 kB of archives.
Get:1 .../tree_2.1.1-2ubuntu3.24.04.2_amd64.deb ...
Setting up tree (2.1.1-2ubuntu3.24.04.2) ...
```

`-y` відповідає «yes» на всі запити під час встановлення.

**3. Перевірка встановлення та версії**

```bash
tree --version
```
```
tree v2.1.1 © 1996 - 2023 by Steve Baker, Thomas Moore, Francesc Rocher, Florian Sesser, Kyosuke Tokoro
```

Також перевірив через `dpkg`:

```bash
dpkg -l tree | grep "^ii"
```
```
ii  tree   2.1.1-2ubuntu3.24.04.2   amd64   displays an indented directory tree, in color
```

`ii` означає, що пакет встановлений.

**4. Видалення пакету**

```bash
sudo apt remove -y tree
```
```
The following packages will be REMOVED:
  tree
0 upgraded, 0 newly installed, 1 to remove and 95 not upgraded.
Removing tree (2.1.1-2ubuntu3.24.04.2) ...
Processing triggers for man-db (2.12.0-4build2) ...
```

Пакет видалено.

---

## Завдання 2. Керування сервісами через systemctl

Працюватимемо із сервісом `cron` (планувальник завдань).

**1. Перевірка статусу сервісу `cron`**

```bash
sudo systemctl status cron
```
```
● cron.service - Regular background program processing daemon
     Loaded: loaded (/usr/lib/systemd/system/cron.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-02-06 06:06:55 UTC; 4 months 28 days ago
       Docs: man:cron(8)
   Main PID: 2468316 (cron)
      Tasks: 1 (limit: 18695)
     Memory: 576.0K (peak: 5.3M)
        CPU: 2min 58.473s
     CGroup: /system.slice/cron.service
             └─2468316 /usr/sbin/cron -f -P

Jul 06 00:25:01 staging-crm CRON[55462]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Jul 06 00:25:01 staging-crm CRON[55463]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1)
Jul 06 00:25:01 staging-crm CRON[55462]: pam_unix(cron:session): session closed for user root
```

Сервіс активний (`active (running)`) і вже в автозавантаженні (`enabled`).

**2. Зупинка сервісу**

```bash
sudo systemctl stop cron
systemctl is-active cron
```
```
inactive
```

Сервіс зупинено — `is-active` повертає `inactive`.

**3. Повторний запуск сервісу**

```bash
sudo systemctl start cron
systemctl is-active cron
```
```
active
```

**4. Додавання в автозавантаження**

```bash
sudo systemctl enable cron
systemctl is-enabled cron
```
```
Synchronizing state of cron.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable cron
enabled
```

`enable` робить так, що сервіс запускатиметься сам під час завантаження системи.

---

## Завдання 3. Робота з логами

**1. Останні 10 рядків `syslog`**

```bash
cd /var/log
tail -n 10 syslog
```
```
2026-07-06T00:29:28+00:00 staging-crm systemd[1]: Reloading...
2026-07-06T00:29:28+00:00 staging-crm systemd[1]: cloud-init-hotplugd.socket: Socket unit configuration has changed...
2026-07-06T00:29:28+00:00 staging-crm systemd[1]: Reloading finished in 302 ms.
2026-07-06T00:29:32+00:00 staging-crm fstrim[58109]: /: 2.5 GiB trimmed on /dev/disk/by-uuid/b9653418...
2026-07-06T00:29:32+00:00 staging-crm systemd[1]: fstrim.service: Deactivated successfully.
2026-07-06T00:29:32+00:00 staging-crm systemd[1]: Finished fstrim.service - Discard unused blocks on filesystems...
2026-07-06T00:30:04+00:00 staging-crm systemd[1]: Starting sysstat-collect.service - system activity accounting tool...
2026-07-06T00:30:04+00:00 staging-crm systemd[1]: sysstat-collect.service: Deactivated successfully.
2026-07-06T00:30:04+00:00 staging-crm systemd[1]: Finished sysstat-collect.service - system activity accounting tool.
```

`tail -n 10` виводить останні 10 рядків. У `syslog` видно системні події: перезавантаження юнітів, `fstrim`, `sysstat`.

**2. Перегляд помилок через `journalctl`**

```bash
sudo journalctl -p err -b
```
```
Feb 10 17:26:20 staging-crm sshd[104660]: error: maximum authentication attempts exceeded for invalid user guest from...
Feb 10 17:28:38 staging-crm sshd[105349]: error: maximum authentication attempts exceeded for invalid user administr...
Feb 10 17:35:47 staging-crm sshd[107670]: error: maximum authentication attempts exceeded for invalid user postgres from...
Feb 10 17:45:21 staging-crm sshd[110806]: error: maximum authentication attempts exceeded for root from 185.246.128...
Feb 10 17:53:36 staging-crm sshd[2468341]: error: beginning MaxStartups throttling
... (багато рядків — це спроби підбору пароля до SSH ззовні)
```

`-p err` — лише записи рівня помилки, `-b` — лише поточне завантаження системи. Тут помилки — це спроби brute-force входу по SSH.

**3. Записи про запуск/зупинку сервісу `cron`**

```bash
sudo journalctl -u cron --since today --no-pager
```
```
Jul 06 00:05:01 staging-crm CRON[48075]: pam_unix(cron:session): session closed for user root
Jul 06 00:15:01 staging-crm CRON[51288]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Jul 06 00:15:01 staging-crm CRON[51289]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1)
Jul 06 00:17:01 staging-crm CRON[51931]: (root) CMD (cd / && run-parts --report /etc/cron.hourly)
Jul 06 00:25:01 staging-crm CRON[55463]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1)
```

`-u cron` фільтрує логи лише сервісу `cron`, `--since today` — лише за сьогодні, `--no-pager` виводить без пейджера `less`. Видно сесії `cron` (відкриття/закриття) і виконані завдання.

---

## Завдання 4. Створення власного сервісу

**1. Створення bash-скрипта**

```bash
cat > ~/myscript.sh << 'EOF'
#!/bin/bash
LOGFILE="/root/myscript.log"
while true; do
    date "+%Y-%m-%d %H:%M:%S" >> "$LOGFILE"
    sleep 1
done
EOF
chmod +x ~/myscript.sh
cat ~/myscript.sh
```
```
#!/bin/bash
LOGFILE="/root/myscript.log"
while true; do
    date "+%Y-%m-%d %H:%M:%S" >> "$LOGFILE"
    sleep 1
done
```

Скрипт у нескінченному циклі дописує поточну дату у файл і чекає 1 секунду. `chmod +x` робить його виконуваним.

**2. Створення файлу сервісу**

```bash
sudo tee /etc/systemd/system/myscript.service > /dev/null << 'EOF'
[Unit]
Description=Date logging service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/root/myscript.sh
Restart=always

[Install]
WantedBy=multi-user.target
EOF
cat /etc/systemd/system/myscript.service
```
```
[Unit]
Description=Date logging service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/root/myscript.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

`ExecStart` вказує шлях до скрипта, `User=root` — від якого користувача запускати, `Restart=always` — перезапускати при падінні.

**3. Запуск сервісу**

```bash
sudo systemctl daemon-reload
sudo systemctl start myscript
sudo systemctl enable myscript
```
```
Created symlink /etc/systemd/system/multi-user.target.wants/myscript.service → /etc/systemd/system/myscript.service.
```

`daemon-reload` каже systemd перечитати юніт-файли, `enable` додає сервіс в автозавантаження.

**4. Перевірка статусу та запису у файл**

```bash
sudo systemctl status myscript
```
```
● myscript.service - Date logging service
     Loaded: loaded (/etc/systemd/system/myscript.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-07-06 00:23:11 UTC; 380ms ago
   Main PID: 54052 (myscript.sh)
      Tasks: 2 (limit: 18695)
     Memory: 556.0K (peak: 1.5M)
        CPU: 20ms
     CGroup: /system.slice/myscript.service
             ├─54052 /bin/bash /root/myscript.sh
             └─54058 sleep 1

Jul 06 00:23:11 staging-crm systemd[1]: Started myscript.service - Date logging service.
```

Сервіс активний. Перевіряю, що дані пишуться у файл:

```bash
sleep 5
tail -n 5 ~/myscript.log
```
```
2026-07-06 00:23:13
2026-07-06 00:23:14
2026-07-06 00:23:15
2026-07-06 00:23:16
2026-07-06 00:23:17
```

Нові рядки додаються щосекунди — сервіс працює коректно.
