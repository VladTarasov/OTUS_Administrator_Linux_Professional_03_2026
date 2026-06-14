# Занятие 9. Systemd — создание unit-файла

## Домашнее задание
- Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default).
- Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта (https://gist.github.com/cea2k/1318020).
- Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно.

## Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default).

Создадим файл с конфигурацией для сервиса в директории /etc/default - из неё сервис будет брать необходимые переменные.
```
root@grub:/home/grub# cat > /etc/default/watchlog
# Configuration file for my watchlog service
# Place it to /etc/default

# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log
```

Создадим файл /var/log/watchlog.log и запишем туда поизвольный текст, содержащий ключевое слово ‘ALERT’ 

```
root@grub:/home/grub# cat > /var/log/watchlog.log
PING localhost (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.014 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.061 ms
64 bytes from localhost (127.0.0.1): icmp_seq=3 ttl=64 time=0.061 ms
64 bytes from localhost (127.0.0.1): icmp_seq=4 ttl=64 time=0.060 ms

ALERT
```

Создадим скрипт для поиска ключевого слова:
```
root@grub:/home/grub# cat > /opt/watchlog.sh
WORD="ALERT"
LOG="/var/log/watchlog.log"
DATE=`date`

if grep $WORD $LOG &> /dev/null;
then
        logger "$DATE: I found word, Master!";
else
exit 0
fi
```
Команда logger отправляет лог в системный журнал. # Каким образом?

Добавим права на запуск файла: 
```
root@grub:/home/grub# chmod +x /opt/watchlog.sh
```

Создадим systemd-юнит для сервиса и запустим его: 
```
root@grub:/home/grub# cat > /etc/systemd/system/watchlog.service
[Unit]
Description=My watchlog service
[Service]
Type=oneshot
EnvironmentFile=/etc/default/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG

root@grub:/home/grub# systemctl start watchlog.service
```

Создадим юнит для таймера: 

```
root@grub:/home/grub# cat > /etc/systemd/system/watchlog.timer
[Unit]
Description=Run watchlog script every 30 second
[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service
[Install]
WantedBy=multi-user.target
```

Запустим timer и убедимся в его работоспособности:
```
root@grub:/home/grub# tail -n 100 /var/log/syslog | grep word
Jun 13 10:48:45 grub root: Sat Jun 13 10:48:45 AM UTC 2026: I found word, Master!
Jun 13 10:49:27 grub root: Sat Jun 13 10:49:27 AM UTC 2026: I found word, Master!
Jun 13 10:50:03 grub root: Sat Jun 13 10:50:03 AM UTC 2026: I found word, Master!
Jun 13 10:50:50 grub root: Sat Jun 13 10:50:50 AM UTC 2026: I found word, Master!
Jun 13 10:52:03 grub root: Sat Jun 13 10:52:03 AM UTC 2026: I found word, Master!
```

## Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта

Устанавливаем spawn-fcgi и необходимые для него пакеты:
```
root@grub:/home/grub#  apt install spawn-fcgi php php-cgi php-cli apache2 libapache2-mod-fcgid -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  apache2-bin apache2-data apache2-utils bzip2 libapache2-mod-php8.1 libapr1 libaprutil1 libaprutil1-dbd-sqlite3
  libaprutil1-ldap liblua5.3-0 mailcap mime-support php-common php8.1 php8.1-cgi php8.1-cli php8.1-common php8.1-opcache
  php8.1-readline ssl-cert
Suggested packages:
  apache2-doc apache2-suexec-pristine | apache2-suexec-custom www-browser bzip2-doc php-pear
The following NEW packages will be installed:
  apache2 apache2-bin apache2-data apache2-utils bzip2 libapache2-mod-fcgid libapache2-mod-php8.1 libapr1 libaprutil1
  libaprutil1-dbd-sqlite3 libaprutil1-ldap liblua5.3-0 mailcap mime-support php php-cgi php-cli php-common php8.1
  php8.1-cgi php8.1-cli php8.1-common php8.1-opcache php8.1-readline spawn-fcgi ssl-cert
0 upgraded, 26 newly installed, 0 to remove and 68 not upgraded.
Need to get 9,159 kB of archives.
After this operation, 41.1 MB of additional disk space will be used.
Get:1 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libapr1 amd64 1.7.0-8ubuntu0.22.04.2 [108 kB]
Get:2 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libaprutil1 amd64 1.6.1-5ubuntu4.22.04.2 [92.8 kB]
Get:3 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libaprutil1-dbd-sqlite3 amd64 1.6.1-5ubuntu4.22.04.2 [11.3 kB]
Get:4 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libaprutil1-ldap amd64 1.6.1-5ubuntu4.22.04.2 [9,170 B]
Get:5 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 liblua5.3-0 amd64 5.3.6-1build1 [140 kB]
Get:6 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 mailcap all 3.70+nmu1ubuntu1 [23.8 kB]
Get:7 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 mime-support all 3.66 [3,696 B]
Get:8 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 bzip2 amd64 1.0.8-5build1 [34.8 kB]
Get:9 http://ru.archive.ubuntu.com/ubuntu jammy/universe amd64 libapache2-mod-fcgid amd64 1:2.3.9-4 [64.9 kB]
Get:10 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 php-common all 2:92ubuntu1 [12.4 kB]
Get:11 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 php8.1-common amd64 8.1.2-1ubuntu2.24 [1,130 kB]
Get:12 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 php8.1-opcache amd64 8.1.2-1ubuntu2.24 [365 kB]
Get:13 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 php8.1-readline amd64 8.1.2-1ubuntu2.24 [13.6 kB]
Get:14 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 php8.1-cli amd64 8.1.2-1ubuntu2.24 [1,835 kB]
Get:15 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libapache2-mod-php8.1 amd64 8.1.2-1ubuntu2.24 [1,766 kB]
Get:16 http://security.ubuntu.com/ubuntu jammy-security/main amd64 apache2-bin amd64 2.4.52-1ubuntu4.21 [1,361 kB]
Get:17 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 php8.1-cgi amd64 8.1.2-1ubuntu2.24 [1,786 kB]
Get:18 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 php8.1 all 8.1.2-1ubuntu2.24 [9,164 B]
Get:19 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 php all 2:8.1+92ubuntu1 [2,756 B]
Get:20 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 php-cgi all 2:8.1+92ubuntu1 [3,280 B]
Get:21 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 php-cli all 2:8.1+92ubuntu1 [3,234 B]
Get:22 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 ssl-cert all 1.1.2 [17.4 kB]
Get:23 http://ru.archive.ubuntu.com/ubuntu jammy/universe amd64 spawn-fcgi amd64 1.6.4-2 [14.9 kB]
Get:24 http://security.ubuntu.com/ubuntu jammy-security/main amd64 apache2-data all 2.4.52-1ubuntu4.21 [165 kB]
Get:25 http://security.ubuntu.com/ubuntu jammy-security/main amd64 apache2-utils amd64 2.4.52-1ubuntu4.21 [89.3 kB]
Get:26 http://security.ubuntu.com/ubuntu jammy-security/main amd64 apache2 amd64 2.4.52-1ubuntu4.21 [97.9 kB]
Fetched 9,159 kB in 2s (5,164 kB/s)
Preconfiguring packages ...
Selecting previously unselected package libapr1:amd64.
(Reading database ... 75441 files and directories currently installed.)
Preparing to unpack .../00-libapr1_1.7.0-8ubuntu0.22.04.2_amd64.deb ...
Unpacking libapr1:amd64 (1.7.0-8ubuntu0.22.04.2) ...
Selecting previously unselected package libaprutil1:amd64.
Preparing to unpack .../01-libaprutil1_1.6.1-5ubuntu4.22.04.2_amd64.deb ...
Unpacking libaprutil1:amd64 (1.6.1-5ubuntu4.22.04.2) ...
Selecting previously unselected package libaprutil1-dbd-sqlite3:amd64.
Preparing to unpack .../02-libaprutil1-dbd-sqlite3_1.6.1-5ubuntu4.22.04.2_amd64.deb ...
Unpacking libaprutil1-dbd-sqlite3:amd64 (1.6.1-5ubuntu4.22.04.2) ...
Selecting previously unselected package libaprutil1-ldap:amd64.
Preparing to unpack .../03-libaprutil1-ldap_1.6.1-5ubuntu4.22.04.2_amd64.deb ...
Unpacking libaprutil1-ldap:amd64 (1.6.1-5ubuntu4.22.04.2) ...
Selecting previously unselected package liblua5.3-0:amd64.
Preparing to unpack .../04-liblua5.3-0_5.3.6-1build1_amd64.deb ...
Unpacking liblua5.3-0:amd64 (5.3.6-1build1) ...
Selecting previously unselected package apache2-bin.
Preparing to unpack .../05-apache2-bin_2.4.52-1ubuntu4.21_amd64.deb ...
Unpacking apache2-bin (2.4.52-1ubuntu4.21) ...
Selecting previously unselected package apache2-data.
Preparing to unpack .../06-apache2-data_2.4.52-1ubuntu4.21_all.deb ...
Unpacking apache2-data (2.4.52-1ubuntu4.21) ...
Selecting previously unselected package apache2-utils.
Preparing to unpack .../07-apache2-utils_2.4.52-1ubuntu4.21_amd64.deb ...
Unpacking apache2-utils (2.4.52-1ubuntu4.21) ...
Selecting previously unselected package mailcap.
Preparing to unpack .../08-mailcap_3.70+nmu1ubuntu1_all.deb ...
Unpacking mailcap (3.70+nmu1ubuntu1) ...
Selecting previously unselected package mime-support.
Preparing to unpack .../09-mime-support_3.66_all.deb ...
Unpacking mime-support (3.66) ...
Selecting previously unselected package apache2.
Preparing to unpack .../10-apache2_2.4.52-1ubuntu4.21_amd64.deb ...
Unpacking apache2 (2.4.52-1ubuntu4.21) ...
Selecting previously unselected package bzip2.
Preparing to unpack .../11-bzip2_1.0.8-5build1_amd64.deb ...
Unpacking bzip2 (1.0.8-5build1) ...
Selecting previously unselected package libapache2-mod-fcgid.
Preparing to unpack .../12-libapache2-mod-fcgid_1%3a2.3.9-4_amd64.deb ...
Unpacking libapache2-mod-fcgid (1:2.3.9-4) ...
Selecting previously unselected package php-common.
Preparing to unpack .../13-php-common_2%3a92ubuntu1_all.deb ...
Unpacking php-common (2:92ubuntu1) ...
Selecting previously unselected package php8.1-common.
Preparing to unpack .../14-php8.1-common_8.1.2-1ubuntu2.24_amd64.deb ...
Unpacking php8.1-common (8.1.2-1ubuntu2.24) ...
Selecting previously unselected package php8.1-opcache.
Preparing to unpack .../15-php8.1-opcache_8.1.2-1ubuntu2.24_amd64.deb ...
Unpacking php8.1-opcache (8.1.2-1ubuntu2.24) ...
Selecting previously unselected package php8.1-readline.
Preparing to unpack .../16-php8.1-readline_8.1.2-1ubuntu2.24_amd64.deb ...
Unpacking php8.1-readline (8.1.2-1ubuntu2.24) ...
Selecting previously unselected package php8.1-cli.
Preparing to unpack .../17-php8.1-cli_8.1.2-1ubuntu2.24_amd64.deb ...
Unpacking php8.1-cli (8.1.2-1ubuntu2.24) ...
Selecting previously unselected package libapache2-mod-php8.1.
Preparing to unpack .../18-libapache2-mod-php8.1_8.1.2-1ubuntu2.24_amd64.deb ...
Unpacking libapache2-mod-php8.1 (8.1.2-1ubuntu2.24) ...
Selecting previously unselected package php8.1-cgi.
Preparing to unpack .../19-php8.1-cgi_8.1.2-1ubuntu2.24_amd64.deb ...
Unpacking php8.1-cgi (8.1.2-1ubuntu2.24) ...
Selecting previously unselected package php8.1.
Preparing to unpack .../20-php8.1_8.1.2-1ubuntu2.24_all.deb ...
Unpacking php8.1 (8.1.2-1ubuntu2.24) ...
Selecting previously unselected package php.
Preparing to unpack .../21-php_2%3a8.1+92ubuntu1_all.deb ...
Unpacking php (2:8.1+92ubuntu1) ...
Selecting previously unselected package php-cgi.
Preparing to unpack .../22-php-cgi_2%3a8.1+92ubuntu1_all.deb ...
Unpacking php-cgi (2:8.1+92ubuntu1) ...
Selecting previously unselected package php-cli.
Preparing to unpack .../23-php-cli_2%3a8.1+92ubuntu1_all.deb ...
Unpacking php-cli (2:8.1+92ubuntu1) ...
Selecting previously unselected package ssl-cert.
Preparing to unpack .../24-ssl-cert_1.1.2_all.deb ...
Unpacking ssl-cert (1.1.2) ...
Selecting previously unselected package spawn-fcgi.
Preparing to unpack .../25-spawn-fcgi_1.6.4-2_amd64.deb ...
Unpacking spawn-fcgi (1.6.4-2) ...
Setting up php-common (2:92ubuntu1) ...
Created symlink /etc/systemd/system/timers.target.wants/phpsessionclean.timer → /lib/systemd/system/phpsessionclean.timer.
Setting up php8.1-common (8.1.2-1ubuntu2.24) ...

Creating config file /etc/php/8.1/mods-available/calendar.ini with new version

Creating config file /etc/php/8.1/mods-available/ctype.ini with new version

Creating config file /etc/php/8.1/mods-available/exif.ini with new version

Creating config file /etc/php/8.1/mods-available/fileinfo.ini with new version

Creating config file /etc/php/8.1/mods-available/ffi.ini with new version

Creating config file /etc/php/8.1/mods-available/ftp.ini with new version

Creating config file /etc/php/8.1/mods-available/gettext.ini with new version

Creating config file /etc/php/8.1/mods-available/iconv.ini with new version

Creating config file /etc/php/8.1/mods-available/pdo.ini with new version

Creating config file /etc/php/8.1/mods-available/phar.ini with new version

Creating config file /etc/php/8.1/mods-available/posix.ini with new version

Creating config file /etc/php/8.1/mods-available/shmop.ini with new version

Creating config file /etc/php/8.1/mods-available/sockets.ini with new version

Creating config file /etc/php/8.1/mods-available/sysvmsg.ini with new version

Creating config file /etc/php/8.1/mods-available/sysvsem.ini with new version

Creating config file /etc/php/8.1/mods-available/sysvshm.ini with new version

Creating config file /etc/php/8.1/mods-available/tokenizer.ini with new version
Setting up libapr1:amd64 (1.7.0-8ubuntu0.22.04.2) ...
Setting up bzip2 (1.0.8-5build1) ...
Setting up ssl-cert (1.1.2) ...
Setting up liblua5.3-0:amd64 (5.3.6-1build1) ...
Setting up php8.1-readline (8.1.2-1ubuntu2.24) ...

Creating config file /etc/php/8.1/mods-available/readline.ini with new version
Setting up apache2-data (2.4.52-1ubuntu4.21) ...
Setting up spawn-fcgi (1.6.4-2) ...
Setting up mailcap (3.70+nmu1ubuntu1) ...
Setting up php8.1-opcache (8.1.2-1ubuntu2.24) ...

Creating config file /etc/php/8.1/mods-available/opcache.ini with new version
Setting up libaprutil1:amd64 (1.6.1-5ubuntu4.22.04.2) ...
Setting up mime-support (3.66) ...
Setting up libaprutil1-ldap:amd64 (1.6.1-5ubuntu4.22.04.2) ...
Setting up libaprutil1-dbd-sqlite3:amd64 (1.6.1-5ubuntu4.22.04.2) ...
Setting up php8.1-cli (8.1.2-1ubuntu2.24) ...
update-alternatives: using /usr/bin/php8.1 to provide /usr/bin/php (php) in auto mode
update-alternatives: using /usr/bin/phar8.1 to provide /usr/bin/phar (phar) in auto mode
update-alternatives: using /usr/bin/phar.phar8.1 to provide /usr/bin/phar.phar (phar.phar) in auto mode

Creating config file /etc/php/8.1/cli/php.ini with new version
Setting up php-cli (2:8.1+92ubuntu1) ...
update-alternatives: using /usr/bin/php.default to provide /usr/bin/php (php) in auto mode
update-alternatives: using /usr/bin/phar.default to provide /usr/bin/phar (phar) in auto mode
update-alternatives: using /usr/bin/phar.phar.default to provide /usr/bin/phar.phar (phar.phar) in auto mode
Setting up php8.1-cgi (8.1.2-1ubuntu2.24) ...
update-alternatives: using /usr/bin/php-cgi8.1 to provide /usr/bin/php-cgi (php-cgi) in auto mode
update-alternatives: using /usr/lib/cgi-bin/php8.1 to provide /usr/lib/cgi-bin/php (php-cgi-bin) in auto mode

Creating config file /etc/php/8.1/cgi/php.ini with new version
Setting up php-cgi (2:8.1+92ubuntu1) ...
update-alternatives: using /usr/bin/php-cgi.default to provide /usr/bin/php-cgi (php-cgi) in auto mode
update-alternatives: using /usr/lib/cgi-bin/php.default to provide /usr/lib/cgi-bin/php (php-cgi-bin) in auto mode
Setting up apache2-utils (2.4.52-1ubuntu4.21) ...
Setting up apache2-bin (2.4.52-1ubuntu4.21) ...
Setting up php8.1 (8.1.2-1ubuntu2.24) ...
Setting up libapache2-mod-php8.1 (8.1.2-1ubuntu2.24) ...
Package apache2 is not configured yet. Will defer actions by package libapache2-mod-php8.1.

Creating config file /etc/php/8.1/apache2/php.ini with new version
No module matches
Setting up php (2:8.1+92ubuntu1) ...
Setting up libapache2-mod-fcgid (1:2.3.9-4) ...
Package apache2 is not configured yet. Will defer actions by package libapache2-mod-fcgid.
Setting up apache2 (2.4.52-1ubuntu4.21) ...
Enabling module mpm_event.
Enabling module authz_core.
Enabling module authz_host.
Enabling module authn_core.
Enabling module auth_basic.
Enabling module access_compat.
Enabling module authn_file.
Enabling module authz_user.
Enabling module alias.
Enabling module dir.
Enabling module autoindex.
Enabling module env.
Enabling module mime.
Enabling module negotiation.
Enabling module setenvif.
Enabling module filter.
Enabling module deflate.
Enabling module status.
Enabling module reqtimeout.
Enabling conf charset.
Enabling conf localized-error-pages.
Enabling conf other-vhosts-access-log.
Enabling conf security.
Enabling conf serve-cgi-bin.
Enabling site 000-default.
info: Switch to mpm prefork for package libapache2-mod-php8.1
Module mpm_event disabled.
Enabling module mpm_prefork.
info: Executing deferred 'a2enmod php8.1' for package libapache2-mod-php8.1
Enabling module php8.1.
info: Executing deferred 'a2enmod fcgid' for package libapache2-mod-fcgid
Enabling module fcgid.
Created symlink /etc/systemd/system/multi-user.target.wants/apache2.service → /lib/systemd/system/apache2.service.
Created symlink /etc/systemd/system/multi-user.target.wants/apache-htcacheclean.service → /lib/systemd/system/apache-htcacheclean.service.
Processing triggers for ufw (0.36.1-4ubuntu0.1) ...
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for libc-bin (2.35-0ubuntu3.13) ...
Processing triggers for php8.1-cli (8.1.2-1ubuntu2.24) ...
Processing triggers for php8.1-cgi (8.1.2-1ubuntu2.24) ...
Processing triggers for libapache2-mod-php8.1 (8.1.2-1ubuntu2.24) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```

Cоздадим файл с настройками для будущего сервиса в файле /etc/spawn-fcgi/fcgi.conf.
```
root@grub:/home/grub# cat > /etc/spawn-fcgi/fcgi.conf
# You must set some working options before the "spawn-fcgi" service
will work.
# If SOCKET points to a file, then this file is cleaned up by the
init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u www-data -g www-data -s $SOCKET -S -M 0600 -C 32 -F 1 --
/usr/bin/php-cgi"
```

