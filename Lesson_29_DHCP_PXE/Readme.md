# Занятие 29. DHCP, PXE

## Цель домашнего задания
Отработать навыки установки и настройки DHCP, TFTP, PXE загрузчика и автоматической загрузки


## Описание домашнего задания
1. Настроить загрузку по сети дистрибутива Ubuntu 24
2. Установка должна проходить из HTTP-репозитория.
3. Настроить автоматическую установку c помощью файла user-data
*4. Настроить автоматическую загрузку по сети дистрибутива Ubuntu 24 c использованием UEFI
Задания со звёздочкой выполняются по желанию

## Выполнение
1. Настройка DHCP и TFTP-сервера

1) Проверяем текущий статус firewall, убедимся в отсутствии правил фильтрации:
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

#Также указаваем интерфейс и range адресов которые будут выдаваться по DHCP
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
6) Поднимаем порт, на котором будут работать DHCP и TFTP сервера, отключаем DNS, для устранения конфликта с systemd-resolve и перезапускаем службу dnsmasq
```
root@pxeserver:~# ip link set ens34 up
root@pxeserver:~# ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
ens33            UP             00:0c:29:ae:0f:b6 <BROADCAST,MULTICAST,UP,LOWER_UP>
ens34            UP             00:50:56:22:a0:0b <BROADCAST,MULTICAST,UP,LOWER_UP>

root@pxeserver:~# systemctl restart dnsmasq
root@pxeserver:~# systemctl status dnsmasq
● dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server
     Loaded: loaded (/usr/lib/systemd/system/dnsmasq.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-07-30 19:33:06 UTC; 32s ago
    Process: 2593 ExecStartPre=/usr/share/dnsmasq/systemd-helper checkconfig (code=exited, status=0/SUCCESS)
    Process: 2598 ExecStart=/usr/share/dnsmasq/systemd-helper exec (code=exited, status=0/SUCCESS)
    Process: 2606 ExecStartPost=/usr/share/dnsmasq/systemd-helper start-resolvconf (code=exited, status=0/SUCCESS)
   Main PID: 2604 (dnsmasq)
      Tasks: 1 (limit: 9374)
     Memory: 744.0K (peak: 2.1M)
        CPU: 26ms
     CGroup: /system.slice/dnsmasq.service
             └─2604 /usr/sbin/dnsmasq -x /run/dnsmasq/dnsmasq.pid -u dnsmasq -r /run/dnsmasq/resolv.conf -7 /etc/dnsmasq.d>

Jul 30 19:33:06 pxeserver systemd[1]: Starting dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server...
Jul 30 19:33:06 pxeserver dnsmasq[2604]: started, version 2.90 DNS disabled
Jul 30 19:33:06 pxeserver dnsmasq[2604]: compile time options: IPv6 GNU-getopt DBus no-UBus i18n IDN2 DHCP DHCPv6 no-Lua T>
Jul 30 19:33:06 pxeserver dnsmasq-dhcp[2604]: DHCP, IP range 10.0.0.100 -- 10.0.0.120, lease time 1h
Jul 30 19:33:06 pxeserver dnsmasq-dhcp[2604]: DHCP, sockets bound exclusively to interface ens34
Jul 30 19:33:06 pxeserver dnsmasq-tftp[2604]: TFTP root is /srv/tftp/amd64
Jul 30 19:33:06 pxeserver systemd[1]: Started dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server.

```








