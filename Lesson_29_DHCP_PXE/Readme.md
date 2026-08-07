# Занятие 29. DHCP, PXE

## Цель домашнего задания
Отработать навыки установки и настройки DHCP, TFTP, PXE загрузчика и автоматической загрузки ОС


## Описание домашнего задания
1. Настроить загрузку по сети дистрибутива Ubuntu 24.
2. Установка должна проходить из HTTP-репозитория.
3. Настроить автоматическую установку c помощью файла user-data.
*4. Настроить автоматическую загрузку по сети дистрибутива Ubuntu 24 c использованием UEFI
Задания со звёздочкой выполняются по желанию.

## Выполнение
### 1. Настройка DHCP и TFTP-сервера

1) Проверяем текущий статус firewall, убедимся в отсутствии правил фильтрации либо скорректируем их:
```
root@pxeserver:~# ufw status
Status: inactive

root@pxeserver:~# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```
Правила фильтрации отсутствуют.
2) Обновляем кэш пакетов и устанавливаем утилиту dnsmasq
```
root@pxeserver:~# apt update
Hit:1 http://ru.archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://ru.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:3 http://ru.archive.ubuntu.com/ubuntu noble-backports InRelease
Hit:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
34 packages can be upgraded. Run 'apt list --upgradable' to see them.
root@pxeserver:~# apt install dnsmasq
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  dns-root-data dnsmasq-base
The following NEW packages will be installed:
  dns-root-data dnsmasq dnsmasq-base
0 upgraded, 3 newly installed, 0 to remove and 34 not upgraded.
Need to get 400 kB of archives.
After this operation, 955 kB of additional disk space will be used.
Do you want to continue? [Y/n]
Get:1 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 dnsmasq-base amd64 2.90-2ubuntu0.4 [376 kB]
Get:2 http://ru.archive.ubuntu.com/ubuntu noble-updates/universe amd64 dnsmasq all 2.90-2ubuntu0.4 [17.9 kB]
Get:3 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 dns-root-data all 2024071801~ubuntu0.24.04.1 [5,918 B]
Fetched 400 kB in 0s (3,391 kB/s)
Selecting previously unselected package dnsmasq-base.
(Reading database ... 88265 files and directories currently installed.)
Preparing to unpack .../dnsmasq-base_2.90-2ubuntu0.4_amd64.deb ...
Unpacking dnsmasq-base (2.90-2ubuntu0.4) ...
Selecting previously unselected package dnsmasq.
Preparing to unpack .../dnsmasq_2.90-2ubuntu0.4_all.deb ...
Unpacking dnsmasq (2.90-2ubuntu0.4) ...
Selecting previously unselected package dns-root-data.
Preparing to unpack .../dns-root-data_2024071801~ubuntu0.24.04.1_all.deb ...
Unpacking dns-root-data (2024071801~ubuntu0.24.04.1) ...
Setting up dnsmasq-base (2.90-2ubuntu0.4) ...
Setting up dns-root-data (2024071801~ubuntu0.24.04.1) ...
Setting up dnsmasq (2.90-2ubuntu0.4) ...
Created symlink /etc/systemd/system/multi-user.target.wants/dnsmasq.service → /usr/lib/systemd/system/dnsmasq.service.
Could not execute systemctl:  at /usr/bin/deb-systemd-invoke line 148.
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for dbus (1.14.10-4ubuntu4.1) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```
3) Создаём файл /etc/dnsmasq.d/pxe.conf и добавляем в него следующее содержимое:
```
root@pxeserver:/etc/dnsmasq.d# cat pxe.conf
#Указываем интерфейс в на котором будет работать DHCP/TFTP
interface=ens34
bind-interfaces

#Также указываем интерфейс и диапазон адресов, которые будут выдаваться по DHCP
dhcp-range=eth34,10.0.0.100,10.0.0.120

#Имя файла, с которого надо начинать загрузку для Legacy boot (этот пример рассматривается в методичке)
dhcp-boot=pxelinux.0

#Имена файлов, для UEFI-загрузки (не обязательно добавлять)
dhcp-match=set:efi-x86_64,option:client-arch,7
dhcp-boot=tag:efi-x86_64,bootx64.efi

#Включаем TFTP-сервер
enable-tftp

#Указываем каталог для TFTP-сервера
tftp-root=/srv/tftp/amd64
```
4) Создаём каталог для файлов TFTP-сервера
```
root@pxeserver:/etc/dnsmasq.d# mkdir -p /srv/tftp/
root@pxeserver:/etc/dnsmasq.d#
```
5) Cкачиваем файлы для сетевой установки Ubuntu 24.04 и распаковываем их в каталог /srv/tftp
```
root@pxeserver:~# wget https://releases.ubuntu.com/noble/ubuntu-24.04.4-netboot-amd64.tar.gz | tar xzvf - -C /srv/tftp
--2026-07-30 19:22:53--  https://releases.ubuntu.com/noble/ubuntu-24.04.4-netboot-amd64.tar.gz
Resolving releases.ubuntu.com (releases.ubuntu.com)... 185.125.190.40, 91.189.91.107, 91.189.91.109, ...
Connecting to releases.ubuntu.com (releases.ubuntu.com)|185.125.190.40|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 96330730 (92M) [application/x-gzip]
Saving to: ‘ubuntu-24.04.4-netboot-amd64.tar.gz’

ubuntu-24.04.4-netboot-amd64.t 100%[===================================================>]  91.87M  11.8MB/s    in 17s

2026-07-30 19:23:10 (5.43 MB/s) - ‘ubuntu-24.04.4-netboot-amd64.tar.gz’ saved [96330730/96330730]

root@pxeserver:~# tar -xzvf ubuntu-24.04.4-netboot-amd64.tar.gz -C /srv/tftp
./
./amd64/
./amd64/bootx64.efi
./amd64/grub/
./amd64/grub/grub.cfg
./amd64/ldlinux.c32
./amd64/linux
./amd64/pxelinux.0
./amd64/initrd
./amd64/pxelinux.cfg/
./amd64/pxelinux.cfg/default
./amd64/grubx64.efi

root@pxeserver:~# ls -l /srv/tftp/amd64/
total 96976
-rw-r--r-- 1 root root   966664 Apr  4  2024 bootx64.efi
drwxr-xr-x 2 root root     4096 Feb 10 00:38 grub
-rw-r--r-- 1 root root  2344840 Mar 28  2025 grubx64.efi
-rw-r--r-- 1 root root 79072248 Feb 10 00:38 initrd
-rw-r--r-- 1 root root   118676 Apr  8  2024 ldlinux.c32
-rw-r--r-- 1 root root 16738376 Feb 10 00:38 linux
-rw-r--r-- 1 root root    42392 Apr  8  2024 pxelinux.0
drwxr-xr-x 2 root root     4096 Feb 10 00:38 pxelinux.cfg
```
6) Настраиваем и поднимаем порт, на котором будут работать DHCP и TFTP сервера, отключаем DNS, для устранения конфликта с systemd-resolve и перезапускаем службу dnsmasq
```
root@pxeserver:~# ip link set ens34 up
root@pxeserver:~# ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
ens33            UP             00:0c:29:ae:0f:b6 <BROADCAST,MULTICAST,UP,LOWER_UP>
ens34            UP             00:50:56:22:a0:0b <BROADCAST,MULTICAST,UP,LOWER_UP>

user@pxeserver:/etc/netplan$ sudo nano /etc/netplan/00-installer-config.yaml
user@pxeserver:/etc/netplan$ cat /etc/netplan/00-installer-config.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens34:
      dhcp4: false
      addresses:
        - 10.0.0.21/24
user@pxeserver:/etc/netplan$ sudo netplan try
user@pxeserver:/etc/netplan$ sudo chmod 600 /etc/netplan/00-installer-config.yaml
user@pxeserver:/etc/netplan$ sudo netplan try
Do you want to keep these settings?


Press ENTER before the timeout to accept the new configuration


Changes will revert in 111 seconds
Configuration accepted.
user@pxeserver:/etc/netplan$ ip -br a
lo               UNKNOWN        127.0.0.1/8 ::1/128
ens33            UP             192.168.175.136/24 metric 100 fe80::20c:29ff:feae:fb6/64
ens34            UP             10.0.0.21/24 fe80::250:56ff:fe22:a00b/64
user@pxeserver:/etc/netplan$ sudo netplan apply

root@pxeserver:~# systemctl restart dnsmasq
user@pxeserver:~$ sudo systemctl restart dnsmasq
user@pxeserver:~$ sudo systemctl status dnsmasq
● dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server
     Loaded: loaded (/usr/lib/systemd/system/dnsmasq.service; enabled; preset: enabled)
     Active: active (running) since Sun 2026-08-02 10:14:51 UTC; 6s ago
    Process: 1335 ExecStartPre=/usr/share/dnsmasq/systemd-helper checkconfig (code=exited, status=0/SUCCESS)
    Process: 1341 ExecStart=/usr/share/dnsmasq/systemd-helper exec (code=exited, status=0/SUCCESS)
    Process: 1347 ExecStartPost=/usr/share/dnsmasq/systemd-helper start-resolvconf (code=exited, status=0/SUCCESS)
   Main PID: 1346 (dnsmasq)
      Tasks: 1 (limit: 9374)
     Memory: 740.0K (peak: 1.8M)
        CPU: 38ms
     CGroup: /system.slice/dnsmasq.service
             └─1346 /usr/sbin/dnsmasq -x /run/dnsmasq/dnsmasq.pid -u dnsmasq -r /run/dnsmasq/resolv.conf -7 /etc/dnsmasq.d>

Aug 02 10:14:51 pxeserver systemd[1]: Starting dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server...
Aug 02 10:14:51 pxeserver dnsmasq[1346]: started, version 2.90 DNS disabled
Aug 02 10:14:51 pxeserver dnsmasq[1346]: compile time options: IPv6 GNU-getopt DBus no-UBus i18n IDN2 DHCP DHCPv6 no-Lua T>
Aug 02 10:14:51 pxeserver dnsmasq-dhcp[1346]: DHCP, IP range 10.0.0.100 -- 10.0.0.120, lease time 1h
Aug 02 10:14:51 pxeserver dnsmasq-dhcp[1346]: DHCP, sockets bound exclusively to interface ens34
Aug 02 10:14:51 pxeserver dnsmasq-tftp[1346]: TFTP root is /srv/tftp/amd64
Aug 02 10:14:51 pxeserver systemd[1]: Started dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server.
user@pxeserver:~$ ss -tunlp | grep dnsmasq
user@pxeserver:~$ sudo ss -tunlp | grep dnsmasq
udp   UNCONN 0      0                         0.0.0.0%ens34:67        0.0.0.0:*    users:(("dnsmasq",pid=1346,fd=4))       
udp   UNCONN 0      0                             127.0.0.1:69        0.0.0.0:*    users:(("dnsmasq",pid=1346,fd=7))       
udp   UNCONN 0      0                             10.0.0.21:69        0.0.0.0:*    users:(("dnsmasq",pid=1346,fd=6))       
udp   UNCONN 0      0                                 [::1]:69           [::]:*    users:(("dnsmasq",pid=1346,fd=9))       
udp   UNCONN 0      0      [fe80::250:56ff:fe22:a00b]%ens34:69           [::]:*    users:(("dnsmasq",pid=1346,fd=8))   
```
Видим, что открыты сокеты для работы TFTP и DHCP серверов.

### 2. Настройка Web-сервера
Для того, чтобы отдавать файл образа системы по HTTP нам потребуется настроенный веб-сервер. 
1) Устанавливаем Web-сервер apache2
```
root@pxeserver:~# apt install apache2
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  apache2-bin apache2-data apache2-utils libapr1t64 libaprutil1-dbd-sqlite3 libaprutil1-ldap libaprutil1t64 liblua5.4-0
  ssl-cert
Suggested packages:
  apache2-doc apache2-suexec-pristine | apache2-suexec-custom www-browser
The following NEW packages will be installed:
  apache2 apache2-bin apache2-data apache2-utils libapr1t64 libaprutil1-dbd-sqlite3 libaprutil1-ldap libaprutil1t64
  liblua5.4-0 ssl-cert
0 upgraded, 10 newly installed, 0 to remove and 34 not upgraded.
Need to get 2,095 kB of archives.
After this operation, 8,129 kB of additional disk space will be used.
Do you want to continue? [Y/n]
Get:1 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 libapr1t64 amd64 1.7.2-3.1ubuntu0.1 [108 kB]
Get:2 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 libaprutil1t64 amd64 1.6.3-1.1ubuntu7 [91.9 kB]
Get:3 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 libaprutil1-dbd-sqlite3 amd64 1.6.3-1.1ubuntu7 [11.2 kB]
Get:4 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 libaprutil1-ldap amd64 1.6.3-1.1ubuntu7 [9,116 B]
Get:5 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 liblua5.4-0 amd64 5.4.6-3build2 [166 kB]
Get:6 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 apache2-bin amd64 2.4.58-1ubuntu8.15 [1,338 kB]
Get:7 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 apache2-data all 2.4.58-1ubuntu8.15 [163 kB]
Get:8 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 apache2-utils amd64 2.4.58-1ubuntu8.15 [99.7 kB]
Get:9 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 apache2 amd64 2.4.58-1ubuntu8.15 [90.2 kB]
Get:10 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 ssl-cert all 1.1.2ubuntu1 [17.8 kB]
Fetched 2,095 kB in 0s (4,689 kB/s)
Preconfiguring packages ...
Selecting previously unselected package libapr1t64:amd64.
(Reading database ... 88314 files and directories currently installed.)
Preparing to unpack .../0-libapr1t64_1.7.2-3.1ubuntu0.1_amd64.deb ...
Unpacking libapr1t64:amd64 (1.7.2-3.1ubuntu0.1) ...
Selecting previously unselected package libaprutil1t64:amd64.
Preparing to unpack .../1-libaprutil1t64_1.6.3-1.1ubuntu7_amd64.deb ...
Unpacking libaprutil1t64:amd64 (1.6.3-1.1ubuntu7) ...
Selecting previously unselected package libaprutil1-dbd-sqlite3:amd64.
Preparing to unpack .../2-libaprutil1-dbd-sqlite3_1.6.3-1.1ubuntu7_amd64.deb ...
Unpacking libaprutil1-dbd-sqlite3:amd64 (1.6.3-1.1ubuntu7) ...
Selecting previously unselected package libaprutil1-ldap:amd64.
Preparing to unpack .../3-libaprutil1-ldap_1.6.3-1.1ubuntu7_amd64.deb ...
Unpacking libaprutil1-ldap:amd64 (1.6.3-1.1ubuntu7) ...
Selecting previously unselected package liblua5.4-0:amd64.
Preparing to unpack .../4-liblua5.4-0_5.4.6-3build2_amd64.deb ...
Unpacking liblua5.4-0:amd64 (5.4.6-3build2) ...
Selecting previously unselected package apache2-bin.
Preparing to unpack .../5-apache2-bin_2.4.58-1ubuntu8.15_amd64.deb ...
Unpacking apache2-bin (2.4.58-1ubuntu8.15) ...
Selecting previously unselected package apache2-data.
Preparing to unpack .../6-apache2-data_2.4.58-1ubuntu8.15_all.deb ...
Unpacking apache2-data (2.4.58-1ubuntu8.15) ...
Selecting previously unselected package apache2-utils.
Preparing to unpack .../7-apache2-utils_2.4.58-1ubuntu8.15_amd64.deb ...
Unpacking apache2-utils (2.4.58-1ubuntu8.15) ...
Selecting previously unselected package apache2.
Preparing to unpack .../8-apache2_2.4.58-1ubuntu8.15_amd64.deb ...
Unpacking apache2 (2.4.58-1ubuntu8.15) ...
Selecting previously unselected package ssl-cert.
Preparing to unpack .../9-ssl-cert_1.1.2ubuntu1_all.deb ...
Unpacking ssl-cert (1.1.2ubuntu1) ...
Setting up ssl-cert (1.1.2ubuntu1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/ssl-cert.service → /usr/lib/systemd/system/ssl-cert.service.
Setting up libapr1t64:amd64 (1.7.2-3.1ubuntu0.1) ...
Setting up liblua5.4-0:amd64 (5.4.6-3build2) ...
Setting up apache2-data (2.4.58-1ubuntu8.15) ...
Setting up libaprutil1t64:amd64 (1.6.3-1.1ubuntu7) ...
Setting up libaprutil1-ldap:amd64 (1.6.3-1.1ubuntu7) ...
Setting up libaprutil1-dbd-sqlite3:amd64 (1.6.3-1.1ubuntu7) ...
Setting up apache2-utils (2.4.58-1ubuntu8.15) ...
Setting up apache2-bin (2.4.58-1ubuntu8.15) ...
Setting up apache2 (2.4.58-1ubuntu8.15) ...
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
Created symlink /etc/systemd/system/multi-user.target.wants/apache2.service → /usr/lib/systemd/system/apache2.service.
Created symlink /etc/systemd/system/multi-user.target.wants/apache-htcacheclean.service → /usr/lib/systemd/system/apache-htcacheclean.service.
Processing triggers for ufw (0.36.2-6) ...
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.8) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```
2) Cоздаём каталог /srv/images в котором будут храниться iso-образы для установки по сети, переходим в него и загружаем iso-образ
```
root@pxeserver:~# mkdir /srv/images
root@pxeserver:~# cd /srv/images/
root@pxeserver:/srv/images# wget https://releases.ubuntu.com/noble/ubuntu-24.04.4-live-server-amd64.iso
--2026-08-04 18:20:05--  https://releases.ubuntu.com/noble/ubuntu-24.04.4-live-server-amd64.iso
Resolving releases.ubuntu.com (releases.ubuntu.com)... 91.189.91.107, 91.189.91.109, 185.125.190.37, ...
Connecting to releases.ubuntu.com (releases.ubuntu.com)|91.189.91.107|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3405469696 (3.2G) [application/x-iso9660-image]
Saving to: ‘ubuntu-24.04.4-live-server-amd64.iso’

ubuntu-24.04.4-live-server-amd 100%[===================================================>]   3.17G  8.78MB/s    in 2m 24s

2026-08-04 18:22:30 (22.6 MB/s) - ‘ubuntu-24.04.4-live-server-amd64.iso’ saved [3405469696/3405469696]
```
3) Cоздаём конфигурационный файл /etc/apache2/sites-available/ks-server.conf со следующим содержимым
```
root@pxeserver:/srv/images# cat > /etc/apache2/sites-available/ks-server.conf
# IP-адрес хоста и порт, на котором будет работать Web-сервер
<VirtualHost 10.0.0.21:80>
DocumentRoot /

# Указываем директорию /srv/images из которой будет загружаться iso-образ
<Directory /srv/images>
Options Indexes MultiViews
AllowOverride All
Require all granted
</Directory>
</VirtualHost>
```
4) Активируем конфигурацию ks-server в apache
```
root@pxeserver:/etc/apache2/sites-available# a2ensite ks-server.conf
Enabling site ks-server.
To activate the new configuration, you need to run:
  systemctl reload apache2
```
5) Вносим изменения в файл /srv/tftp/amd64/pxelinux.cfg/default 
```
root@pxeserver:/srv/images# nano /srv/tftp/amd64/pxelinux.cfg/default
DEFAULT install
LABEL install
  KERNEL linux
  INITRD initrd
  APPEND root=/dev/ram0 ramdisk_size=3000000 ip=dhcp iso-url=http://10.0.0.21/srv/images/ubuntu-24.04.4-live-server-amd64.iso autoinstall ---
```
В данном файле мы указываем что файлы linux и initrd будут забираться по tftp, а сам iso-образ ubuntu 24.04 будет скачиваться из нашего веб-сервера http://10.0.0.21/srv/images/ubuntu-24.04.4-live-server-amd64.iso 
Из-за того, что образ достаточно большой (3.2G) и он сначала загружается в ОЗУ, необходимо указать достаточный размер ОЗУ (root=/dev/ram0 ramdisk_size=3000000).
6) Перезагружаем web-сервер apache и проверяем его статус
```
root@pxeserver:/etc/apache2/sites-available# systemctl status apache2
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/apache2.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-08-04 18:55:56 UTC; 8s ago
       Docs: https://httpd.apache.org/docs/2.4/
    Process: 2854 ExecStart=/usr/sbin/apachectl start (code=exited, status=0/SUCCESS)
   Main PID: 2858 (apache2)
      Tasks: 55 (limit: 9374)
     Memory: 5.1M (peak: 5.6M)
        CPU: 21ms
     CGroup: /system.slice/apache2.service
             ├─2858 /usr/sbin/apache2 -k start
             ├─2860 /usr/sbin/apache2 -k start
             └─2861 /usr/sbin/apache2 -k start

Aug 04 18:55:56 pxeserver systemd[1]: Starting apache2.service - The Apache HTTP Server...
Aug 04 18:55:56 pxeserver systemd[1]: Started apache2.service - The Apache HTTP Server.

root@pxeserver:/etc/apache2/sites-available# ss -tunlp | grep :80
tcp   LISTEN 0      511                                   *:80              *:*    users:(("apache2",pid=2861,fd=4),("apach
```
На этом настройка Web-сервера завершена и на данный момент, если мы запустим ВМ pxeclient, то увидим загрузку по PXE 
и далее увидим загрузку iso-образа и откроется мастер установки ubuntu.

<img width="874" height="513" alt="image" src="https://github.com/user-attachments/assets/79057985-6eb8-43f7-8fea-74ae879dd4d7" />
<img width="617" height="208" alt="image" src="https://github.com/user-attachments/assets/a392a102-04c7-4f0d-8e14-5249806d1560" />
Так как на хосте pxeclient используется 2 сетевых карты, загрузка может начаться с неправильного адреса, тогда мы получим вот такое сообщение.
<img width="827" height="544" alt="image" src="https://github.com/user-attachments/assets/39c8b659-cf3c-4f18-934b-f28343e056bb" />
В данной ситуации поможет перезапуск виртуальной машины с помощью VirtualBox, либо её удаление и повторная инициализация с помощью команды vagrant up. Конкретно в нашем случае был отключен второй сетевой интерфейс и выполнена перезагрузка машины.

### 3. Настройка автоматической установки Ubuntu 24.04 

Автоматизируем установку ОС (чтобы не пользоваться мастером установки вручную) 
1) cоздаём каталог для файлов с автоматической установкой
```
root@pxeserver:~# mkdir /srv/ks
```
2) создаём файл /srv/ks/user-data и добавляем в него следующее содержимое
```
root@pxeserver:~# cat > /srv/ks/user-data
#cloud-config
autoinstall:
  version: 1
  apt:
    fallback: offline-install
    preserve_sources_list: false
    geoip: false
  drivers:
    install: false
  identity:
    hostname: pxeclient
    password: $6$sJgo6Hg5zXBwkkI8$btrEoWAb5FxKhajagWR49XM4EAOfODr5bMrLOkGe3KkMYdsh7T3MU5mYwY2TIMJpVKckAwnZFs2ltUJ1abOZ.
    realname: otus
    username: otus
  kernel:
    package: linux-generic
  keyboard:
    layout: us
    variant: ""
    toggle: null
  network:
    version: 2
    ethernets:
      ens33:
        dhcp4: true
      ens34:
        dhcp4: true
  ssh:
    allow-pw: true
    authorized-keys: []
    install-server: true
  updates: security
```
3) создаём файл с метаданными /srv/ks/meta-data
```
root@pxeserver:~# touch /srv/ks/meta-data
```
Файл с метаданными хранит дополнительную информацию о хосте, не будем добавлять дополнительную информацию.

4) в конфигурации веб-сервера добавим каталог /srv/ks идентично каталогу /srv/images
```
root@pxeserver:/srv/ks# nano /etc/apache2/sites-available/ks-server.conf
root@pxeserver:/srv/ks# cat /etc/apache2/sites-available/ks-server.conf
# IP-адрес хоста и порт, на котором будет работать Web-сервер
<VirtualHost 10.0.0.21:80>
DocumentRoot /
</VirtualHost>

# Указываем директорию /srv/images из которой будет загружаться iso-образ
<Directory /srv/images>
Options Indexes MultiViews
AllowOverride All
Require all granted
</Directory>

# Указываем директорию /srv/ks из которой будет загружаться файл автонастройки ОС при установке
<Directory /srv/ks>
Options Indexes MultiViews
AllowOverride All
Require all granted
</Directory>
```
5) в файле /srv/tftp/amd64/pxelinux.cfg/default добавляем параметры автоматической установки ОС
```
root@pxeserver:/srv/ks# cat /srv/tftp/amd64/pxelinux.cfg/default
DEFAULT install
LABEL install
  KERNEL linux
  INITRD initrd
  APPEND root=/dev/ram0 ramdisk_size=4000000 ip=dhcp cloud-config-url=http://10.0.0.21/srv/ks/user-data iso-url=http://10.0.0.21/srv/images/ubuntu-24.04.4-live-server-amd64.iso autoinstall ds=nocloud-net;s=http://10.0.0.21/
```
6) перезапускаем службы dnsmasq и apache2 и проверяем их статус
```
root@pxeserver:/srv/ks# systemctl restart dnsmasq
root@pxeserver:/srv/ks# systemctl status dnsmasq
dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server
     Loaded: loaded (/usr/lib/systemd/system/dnsmasq.service; enabled; preset: enabled)
     Active: active (running) since Wed 2026-08-05 20:52:53 UTC; 6s ago
    Process: 1923 ExecStartPre=/usr/share/dnsmasq/systemd-helper checkconfig (code=exited, status=0/SUCCESS)
    Process: 1929 ExecStart=/usr/share/dnsmasq/systemd-helper exec (code=exited, status=0/SUCCESS)
    Process: 1937 ExecStartPost=/usr/share/dnsmasq/systemd-helper start-resolvconf (code=exited, status=0/SUCCESS)
   Main PID: 1935 (dnsmasq)
      Tasks: 1 (limit: 9374)
     Memory: 744.0K (peak: 2.1M)
        CPU: 29ms
     CGroup: /system.slice/dnsmasq.service
             └─1935 /usr/sbin/dnsmasq -x /run/dnsmasq/dnsmasq.pid -u dnsmasq -r /run/dnsmasq/resolv.conf -7 /etc/dnsmasq.d>

Aug 05 20:52:53 pxeserver systemd[1]: Starting dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server...
Aug 05 20:52:53 pxeserver dnsmasq[1935]: started, version 2.90 DNS disabled
Aug 05 20:52:53 pxeserver dnsmasq[1935]: compile time options: IPv6 GNU-getopt DBus no-UBus i18n IDN2 DHCP DHCPv6 no-Lua T>
Aug 05 20:52:53 pxeserver dnsmasq-dhcp[1935]: DHCP, IP range 10.0.0.100 -- 10.0.0.120, lease time 1h
Aug 05 20:52:53 pxeserver dnsmasq-dhcp[1935]: DHCP, sockets bound exclusively to interface ens34
Aug 05 20:52:53 pxeserver dnsmasq-tftp[1935]: TFTP root is /srv/tftp/amd64
Aug 05 20:52:53 pxeserver systemd[1]: Started dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server.

root@pxeserver:/srv/ks# systemctl restart apache2
root@pxeserver:/srv/ks# systemctl status apache2
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/apache2.service; enabled; preset: enabled)
     Active: active (running) since Wed 2026-08-05 20:53:56 UTC; 7s ago
       Docs: https://httpd.apache.org/docs/2.4/
    Process: 1955 ExecStart=/usr/sbin/apachectl start (code=exited, status=0/SUCCESS)
   Main PID: 1958 (apache2)
      Tasks: 55 (limit: 9374)
     Memory: 5.2M (peak: 5.5M)
        CPU: 22ms
     CGroup: /system.slice/apache2.service
             ├─1958 /usr/sbin/apache2 -k start
             ├─1960 /usr/sbin/apache2 -k start
             └─1961 /usr/sbin/apache2 -k start

Aug 05 20:53:56 pxeserver systemd[1]: Starting apache2.service - The Apache HTTP Server...
Aug 05 20:53:56 pxeserver systemd[1]: Started apache2.service - The Apache HTTP Server.
```

На этом настройка автоматической установки завершена. Теперь можно перезапустить ВМ pxeclient и мы должны увидеть автоматическую установку. 

После успешной установки выключаем ВМ и в её настройках ставим запуск ВМ из диска Далее, после запуска нашей ВМ мы сможем залогиниться под пользователем otus. 
На этом настройка автоматической установки завершена. 
