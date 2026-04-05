# **Занятие 1. С чего начинается Linux**

**Цель домашнего задания**\
Научиться обновлять ядро в ОС Linux.

**Описание домашнего задания**
1) Запустить ВМ c Ubuntu.
2) Обновить ядро ОС на новейшую стабильную версию из mainline-репозитория.
3) Оформить отчет в README-файле в GitHub-репозитории.

**Дополнительное задание:**\
Собрать ядро самостоятельно из исходных кодов.

**Основная часть**\
Подключаемся по ssh к виртуальной машине.
Перед работами проверим текущую версию ядра:
```
user@ubuntu-server:~$ uname -r
6.8.0-107-generic
user@ubuntu-server:~/kernel$  ls -al /boot
total 98464
drwxr-xr-x  4 root root     4096 Apr  5 05:53 .
drwxr-xr-x 23 root root     4096 Apr  5 05:48 ..
-rw-r--r--  1 root root   287601 Mar 13 13:27 config-6.8.0-107-generic
drwxr-xr-x  5 root root     4096 Apr  5 05:51 grub
lrwxrwxrwx  1 root root       28 Apr  5 05:50 initrd.img -> initrd.img-6.8.0-107-generic
-rw-r--r--  1 root root 76332373 Apr  5 05:53 initrd.img-6.8.0-107-generic
lrwxrwxrwx  1 root root       28 Apr  5 05:50 initrd.img.old -> initrd.img-6.8.0-107-generic
drwx------  2 root root    16384 Apr  5 05:49 lost+found
-rw-------  1 root root  9125925 Mar 13 13:27 System.map-6.8.0-107-generic
lrwxrwxrwx  1 root root       25 Apr  5 05:50 vmlinuz -> vmlinuz-6.8.0-107-generic
-rw-------  1 root root 15042952 Mar 13 17:46 vmlinuz-6.8.0-107-generic
lrwxrwxrwx  1 root root       25 Apr  5 05:50 vmlinuz.old -> vmlinuz-6.8.0-107-generic
```
Уточняем арихтектуру системы:
```
user@ubuntu-server:~$ uname -p
x86_64
```
Выберем из репозитория https://kernel.ubuntu.com/mainline/ версию для обновления.
Будем обновляться до версии 6.13.2.
<img width="1608" height="564" alt="image" src="https://github.com/user-attachments/assets/7ffc11e2-e7b9-4d12-a782-8423a58502e9" />

Скачиваем пакеты на виртуальную машину:
```
user@ubuntu-server:~$ mkdir kernel && cd kernel
user@ubuntu-server:~/kernel$ wget https://kernel.ubuntu.com/mainline/v6.13.2/amd64/linux-headers-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb
--2026-04-05 06:58:48--  https://kernel.ubuntu.com/mainline/v6.13.2/amd64/linux-headers-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb
Resolving kernel.ubuntu.com (kernel.ubuntu.com)... 185.125.189.76, 185.125.189.74, 185.125.189.75
Connecting to kernel.ubuntu.com (kernel.ubuntu.com)|185.125.189.76|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3694072 (3.5M) [application/x-debian-package]
Saving to: ‘linux-headers-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb’

linux-headers-6.13.2-061302-gen 100%[====================================================>]   3.52M  6.13MB/s    in 0.6s

2026-04-05 06:58:49 (6.13 MB/s) - ‘linux-headers-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb’ saved [3694072/3694072]

user@ubuntu-server:~/kernel$  wget https://kernel.ubuntu.com/mainline/v6.13.2/amd64/linux-headers-6.13.2-061302_6.13.2-061302.202502081010_all.deb
--2026-04-05 06:59:24--  https://kernel.ubuntu.com/mainline/v6.13.2/amd64/linux-headers-6.13.2-061302_6.13.2-061302.202502081010_all.deb
Resolving kernel.ubuntu.com (kernel.ubuntu.com)... 185.125.189.75, 185.125.189.76, 185.125.189.74
Connecting to kernel.ubuntu.com (kernel.ubuntu.com)|185.125.189.75|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 13875326 (13M) [application/x-debian-package]
Saving to: ‘linux-headers-6.13.2-061302_6.13.2-061302.202502081010_all.deb’

linux-headers-6.13.2-061302_6.1 100%[====================================================>]  13.23M  15.4MB/s    in 0.9s

2026-04-05 06:59:25 (15.4 MB/s) - ‘linux-headers-6.13.2-061302_6.13.2-061302.202502081010_all.deb’ saved [13875326/13875326]

user@ubuntu-server:~/kernel$ wget https://kernel.ubuntu.com/mainline/v6.13.2/amd64/linux-image-unsigned-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb
--2026-04-05 06:59:44--  https://kernel.ubuntu.com/mainline/v6.13.2/amd64/linux-image-unsigned-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb
Resolving kernel.ubuntu.com (kernel.ubuntu.com)... 185.125.189.74, 185.125.189.75, 185.125.189.76
Connecting to kernel.ubuntu.com (kernel.ubuntu.com)|185.125.189.74|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 15677632 (15M) [application/x-debian-package]
Saving to: ‘linux-image-unsigned-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb’

linux-image-unsigned-6.13.2-061 100%[====================================================>]  14.95M  16.6MB/s    in 0.9s

2026-04-05 06:59:45 (16.6 MB/s) - ‘linux-image-unsigned-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb’ saved [15677632/15677632]

user@ubuntu-server:~/kernel$ wget https://kernel.ubuntu.com/mainline/v6.13.2/amd64/linux-modules-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb
--2026-04-05 06:59:59--  https://kernel.ubuntu.com/mainline/v6.13.2/amd64/linux-modules-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb
Resolving kernel.ubuntu.com (kernel.ubuntu.com)... 185.125.189.76, 185.125.189.74, 185.125.189.75
Connecting to kernel.ubuntu.com (kernel.ubuntu.com)|185.125.189.76|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 184576192 (176M) [application/x-debian-package]
Saving to: ‘linux-modules-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb’

linux-modules-6.13.2-061302-gen 100%[====================================================>] 176.03M  7.80MB/s    in 17s

2026-04-05 07:00:17 (10.4 MB/s) - ‘linux-modules-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb’ saved [184576192/184576192]
```
Устанавливаем все пакеты сразу:
```
user@ubuntu-server:~/kernel$  sudo dpkg -i *.deb
[sudo] password for user:
Selecting previously unselected package linux-headers-6.13.2-061302.
(Reading database ... 87540 files and directories currently installed.)
Preparing to unpack linux-headers-6.13.2-061302_6.13.2-061302.202502081010_all.deb ...
Unpacking linux-headers-6.13.2-061302 (6.13.2-061302.202502081010) ...
Selecting previously unselected package linux-headers-6.13.2-061302-generic.
Preparing to unpack linux-headers-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb ...
Unpacking linux-headers-6.13.2-061302-generic (6.13.2-061302.202502081010) ...
Selecting previously unselected package linux-image-unsigned-6.13.2-061302-generic.
Preparing to unpack linux-image-unsigned-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb ...
Unpacking linux-image-unsigned-6.13.2-061302-generic (6.13.2-061302.202502081010) ...
Selecting previously unselected package linux-modules-6.13.2-061302-generic.
Preparing to unpack linux-modules-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb ...
Unpacking linux-modules-6.13.2-061302-generic (6.13.2-061302.202502081010) ...
Setting up linux-headers-6.13.2-061302 (6.13.2-061302.202502081010) ...
Setting up linux-headers-6.13.2-061302-generic (6.13.2-061302.202502081010) ...
Setting up linux-modules-6.13.2-061302-generic (6.13.2-061302.202502081010) ...
Setting up linux-image-unsigned-6.13.2-061302-generic (6.13.2-061302.202502081010) ...
I: /boot/vmlinuz is now a symlink to vmlinuz-6.13.2-061302-generic
I: /boot/initrd.img is now a symlink to initrd.img-6.13.2-061302-generic
Processing triggers for linux-image-unsigned-6.13.2-061302-generic (6.13.2-061302.202502081010) ...
/etc/kernel/postinst.d/initramfs-tools:
update-initramfs: Generating /boot/initrd.img-6.13.2-061302-generic
/etc/kernel/postinst.d/zz-update-grub:
Sourcing file `/etc/default/grub'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.13.2-061302-generic
Found initrd image: /boot/initrd.img-6.13.2-061302-generic
Found linux image: /boot/vmlinuz-6.8.0-107-generic
Found initrd image: /boot/initrd.img-6.8.0-107-generic
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
```
Проверяем, что ядро появилось в /boot.
```
user@ubuntu-server:~/kernel$ ls -al /boot
total 207212
drwxr-xr-x  4 root root     4096 Apr  5 10:12 .
drwxr-xr-x 23 root root     4096 Apr  5 05:48 ..
-rw-r--r--  1 root root   294303 Feb  8  2025 config-6.13.2-061302-generic
-rw-r--r--  1 root root   287601 Mar 13 13:27 config-6.8.0-107-generic
drwxr-xr-x  5 root root     4096 Apr  5 10:12 grub
lrwxrwxrwx  1 root root       32 Apr  5 10:12 initrd.img -> initrd.img-6.13.2-061302-generic
-rw-r--r--  1 root root 85471604 Apr  5 10:12 initrd.img-6.13.2-061302-generic
-rw-r--r--  1 root root 76332373 Apr  5 05:53 initrd.img-6.8.0-107-generic
lrwxrwxrwx  1 root root       28 Apr  5 05:50 initrd.img.old -> initrd.img-6.8.0-107-generic
drwx------  2 root root    16384 Apr  5 05:49 lost+found
-rw-------  1 root root  9934398 Feb  8  2025 System.map-6.13.2-061302-generic
-rw-------  1 root root  9125925 Mar 13 13:27 System.map-6.8.0-107-generic
lrwxrwxrwx  1 root root       29 Apr  5 10:12 vmlinuz -> vmlinuz-6.13.2-061302-generic
-rw-------  1 root root 15647232 Feb  8  2025 vmlinuz-6.13.2-061302-generic
-rw-------  1 root root 15042952 Mar 13 17:46 vmlinuz-6.8.0-107-generic
lrwxrwxrwx  1 root root       25 Apr  5 05:50 vmlinuz.old -> vmlinuz-6.8.0-107-generic
```
Перезагружаем виртуальную машину.
После загрузки проверяем текущию версию ядра, видим новую версию:
```
user@ubuntu-server:~/kernel$ sudo reboot

Broadcast message from root@ubuntu-server on pts/1 (Sun 2026-04-05 10:15:51 UTC):

The system will reboot now!user@ubuntu-server:~$ uname -r
6.13.2-061302-generic
```

