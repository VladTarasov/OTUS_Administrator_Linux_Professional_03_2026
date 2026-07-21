# Занятие 27. Резервное копирование

## Домашнее задание
**Цель**\
Научиться настраивать резервное копирование с помощью утилиты Borg

**Описание**
1. Настроить стенд Vagrant с двумя виртуальными машинами: backup_server и client. (Студент самостоятельно настраивает Vagrant)
2. Настроить удаленный бэкап каталога /etc c сервера client при помощи borgbackup. Резервные копии должны соответствовать следующим критериям:\
  a) директория для резервных копий /var/backup. Это должна быть отдельная точка монтирования. В данном случае для демонстрации размер не принципиален, достаточно будет и 2GB; (Студент самостоятельно настраивает)\
  b) репозиторий для резервных копий должен быть зашифрован ключом или паролем - на усмотрение студента;\
  c) имя бэкапа должно содержать информацию о времени снятия бекапа;\
  d) глубина бекапа должна быть год, хранить можно по последней копии на конец месяца, кроме последних трех. Последние три месяца должны содержать копии на каждый день. Т.е. должна быть правильно настроена политика удаления старых бэкапов;\
  e) резервная копия снимается каждые 5 минут. Такой частый запуск в целях демонстрации;\
  f) написан скрипт для снятия резервных копий. Скрипт запускается из соответствующей Cron джобы, либо systemd timer-а - на усмотрение студента;\
  g) настроено логирование процесса бекапа. Для упрощения можно весь вывод перенаправлять в logger с соответствующим тегом. Если настроите не в syslog, то обязательна ротация логов.

## Выполнение
1. Настроить стенд Vagrant с двумя виртуальными машинами: backup_server и client. (Студент самостоятельно настраивает Vagrant)

Настроеный две виртуальные машины backup и client со следующими данными:
- 2 CPU
- 10G HDD
- 8 Gb RAM
- OS Ubuntu 22.04.5 LTS
- backup IP 192.168.175.134/24
- client IP 192.168.175.135/24

К машине сервер подключен дополнительный HDD 2 Gb для хранения бекапов.
Vagrant не использовался.

2. Настроить удаленный бэкап каталога /etc c сервера client при помощи borgbackup. Резервные копии должны соответствовать следующим критериям:\
a) директория для резервных копий /var/backup. Это должна быть отдельная точка монтирования. В данном случае для демонстрации размер не принципиален, достаточно будет и 2GB.
К серверу для целей резервного копирования подключен диск sdb объемом 2 Гб.
```
user@user:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0 20.4G  0 disk 
├─sda1                      8:1    0  973M  0 part /boot/efi
├─sda2                      8:2    0  1.8G  0 part /boot
└─sda3                      8:3    0 17.7G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0   10G  0 lvm  /
sdb                         8:16   0    2G  0 disk 
sr0                        11:0    1 1024M  0 rom  
```
Для гипотетических целей дальнейшго мастабирования будем использовать lvm:
- создадим physical volume
- создадим volume group
- создадим logical volume из 100% доступного места (2 Гб)
- создаим файловую систему ext4
- создадим директорию для хранения бекапов и примонтируем к ней logical volume
- настроим автоматическое монтирование в /etc/fstab
- перезагрузим сервер, убедимся в работоспособности автомонтирования
```
root@backup:~# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
root@backup:~# vgcreate backup /dev/sdb
  Volume group "backup" successfully created
root@backup:~# lvcreate -l 100%FREE -n etc backup
  Logical volume "etc" created.
root@backup:~# mkdir -p /var/backup
root@backup:~# mkfs.ext4 /dev/backup/etc
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 523264 4k blocks and 130816 inodes
Filesystem UUID: c97e3d68-f431-47d2-80b3-7c9471f92b38
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done 

root@backup:~# mount /dev/backup/etc /var/backup
root@backup:~# df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              196M  1.1M  195M   1% /run
efivarfs                           256K   13K  244K   6% /sys/firmware/efi/efivars
/dev/mapper/ubuntu--vg-ubuntu--lv  9.8G  5.2G  4.1G  56% /
tmpfs                              977M     0  977M   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          1.7G  198M  1.4G  13% /boot
/dev/sda1                          972M  6.4M  965M   1% /boot/efi
tmpfs                              196M   12K  196M   1% /run/user/1000
/dev/mapper/backup-etc             2.0G   24K  1.9G   1% /var/backup
root@backup:~# echo "`blkid | grep backup | awk '{print $2}'`
> ^C
root@backup:~# echo "`blkid | grep backup | awk '{print $2}'`"
UUID="c97e3d68-f431-47d2-80b3-7c9471f92b38"
root@backup:~# echo "`blkid | grep backup | awk '{print $2}'` /var/backup ext4 defaults 0 0" >> /etc/fstab
root@backup:~# cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-SKZl0OEyg7tfzRrdaGEUilqslg1VfUkEVFGXbYIk2PLCAQmKZK1wuX3dBNiYjrrF / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/08be98f1-0f8f-47fd-a815-8801a5d4c63e /boot ext4 defaults 0 1
# /boot/efi was on /dev/sda1 during curtin installation
/dev/disk/by-uuid/87CC-0B27 /boot/efi vfat defaults 0 1
/swap.img	none	swap	sw	0	0
UUID="c97e3d68-f431-47d2-80b3-7c9471f92b38" /var/backup ext4 defaults 0 0

root@backup:~# reboot

Broadcast message from root@user on pts/1 (Mon 2026-07-20 18:57:34 UTC):

The system will reboot now!

root@backup:~# Read from remote host 192.168.1.77: Connection reset by peer
Connection to 192.168.1.77 closed.
client_loop: send disconnect: Broken pipe

~ % ssh user@192.168.1.77
user@192.168.1.77's password: 
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-134-generic aarch64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon Jul 20 06:57:52 PM UTC 2026

  System load:  0.0               Processes:               102
  Usage of /:   53.1% of 9.75GB   Users logged in:         0
  Memory usage: 9%                IPv4 address for enp0s8: 192.168.1.77
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

29 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Mon Jul 20 18:36:28 2026 from 192.168.1.71
user@backup:~$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              196M  1.1M  195M   1% /run
efivarfs                           256K   15K  242K   6% /sys/firmware/efi/efivars
/dev/mapper/ubuntu--vg-ubuntu--lv  9.8G  5.2G  4.1G  57% /
tmpfs                              977M     0  977M   0% /dev/shm
```
Устанавливаем на client и backup сервере borgbackup
```
user@backup:~$ sudo -i
[sudo] password for user: 
root@backup:~# apt update
Hit:1 http://ports.ubuntu.com/ubuntu-ports noble InRelease
Get:2 http://ports.ubuntu.com/ubuntu-ports noble-updates InRelease [126 kB]
Get:3 http://ports.ubuntu.com/ubuntu-ports noble-backports InRelease [126 kB]
Get:4 http://ports.ubuntu.com/ubuntu-ports noble-security InRelease [126 kB]
Get:5 http://ports.ubuntu.com/ubuntu-ports noble-updates/main arm64 Packages [1,146 kB]
Get:6 http://ports.ubuntu.com/ubuntu-ports noble-updates/main Translation-en [272 kB]
Get:7 http://ports.ubuntu.com/ubuntu-ports noble-updates/main arm64 Components [178 kB]
Get:8 http://ports.ubuntu.com/ubuntu-ports noble-updates/restricted arm64 Packages [1,876 kB]
Get:9 http://ports.ubuntu.com/ubuntu-ports noble-updates/restricted Translation-en [286 kB]
Get:10 http://ports.ubuntu.com/ubuntu-ports noble-updates/universe arm64 Packages [1,666 kB]
Get:11 http://ports.ubuntu.com/ubuntu-ports noble-updates/universe Translation-en [328 kB]
Get:12 http://ports.ubuntu.com/ubuntu-ports noble-updates/universe arm64 Components [387 kB]
Get:13 http://ports.ubuntu.com/ubuntu-ports noble-backports/main arm64 Components [3,584 B]
Get:14 http://ports.ubuntu.com/ubuntu-ports noble-backports/universe arm64 Components [10.5 kB]
Get:15 http://ports.ubuntu.com/ubuntu-ports noble-security/main arm64 Packages [893 kB]
Get:16 http://ports.ubuntu.com/ubuntu-ports noble-security/main Translation-en [192 kB]
Get:17 http://ports.ubuntu.com/ubuntu-ports noble-security/main arm64 Components [41.9 kB]
Get:18 http://ports.ubuntu.com/ubuntu-ports noble-security/restricted arm64 Packages [1,769 kB]
Get:19 http://ports.ubuntu.com/ubuntu-ports noble-security/restricted Translation-en [268 kB]
Get:20 http://ports.ubuntu.com/ubuntu-ports noble-security/universe arm64 Packages [1,218 kB]
Get:21 http://ports.ubuntu.com/ubuntu-ports noble-security/universe Translation-en [233 kB]
Get:22 http://ports.ubuntu.com/ubuntu-ports noble-security/universe arm64 Components [76.3 kB]
Fetched 11.2 MB in 2s (6,518 kB/s)                                     
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
66 packages can be upgraded. Run 'apt list --upgradable' to see them.
root@backup:~# 
root@backup:~# 
root@backup:~# apt install borgbackup
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  python3-msgpack
Suggested packages:
  python3-pyfuse3 borgbackup-doc
The following NEW packages will be installed:
  borgbackup python3-msgpack
0 upgraded, 2 newly installed, 0 to remove and 66 not upgraded.
Need to get 887 kB of archives.
After this operation, 3,889 kB of additional disk space will be used.
Do you want to continue? [Y/n] 
Get:1 http://ports.ubuntu.com/ubuntu-ports noble/main arm64 python3-msgpack arm64 1.0.3-3build2 [77.9 kB]
Get:2 http://ports.ubuntu.com/ubuntu-ports noble/universe arm64 borgbackup arm64 1.2.8-1 [809 kB]
Fetched 887 kB in 0s (2,337 kB/s)   
Selecting previously unselected package python3-msgpack.
(Reading database ... 134445 files and directories currently installed.)
Preparing to unpack .../python3-msgpack_1.0.3-3build2_arm64.deb ...
Unpacking python3-msgpack (1.0.3-3build2) ...
Selecting previously unselected package borgbackup.
Preparing to unpack .../borgbackup_1.2.8-1_arm64.deb ...
Unpacking borgbackup (1.2.8-1) ...
Setting up python3-msgpack (1.0.3-3build2) ...
Setting up borgbackup (1.2.8-1) ...
Processing triggers for man-db (2.12.0-4build2) ...
Scanning processes...                                                                                     
Scanning processor microcode...                                                                           
Scanning linux images...                                                                                  

Running kernel seems to be up-to-date.

The processor microcode seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.


user@client:~$ sudo -i
[sudo] password for user: 
root@client:~# apt install borgbackup
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  python3-msgpack
Suggested packages:
  python3-pyfuse3 borgbackup-doc
The following NEW packages will be installed:
  borgbackup python3-msgpack
0 upgraded, 2 newly installed, 0 to remove and 34 not upgraded.
Need to get 887 kB of archives.
After this operation, 3,889 kB of additional disk space will be used.
Do you want to continue? [Y/n] 
Get:1 http://ports.ubuntu.com/ubuntu-ports noble/main arm64 python3-msgpack arm64 1.0.3-3build2 [77.9 kB]
Get:2 http://ports.ubuntu.com/ubuntu-ports noble/universe arm64 borgbackup arm64 1.2.8-1 [809 kB]
Fetched 887 kB in 0s (2,051 kB/s)   
Selecting previously unselected package python3-msgpack.
(Reading database ... 92150 files and directories currently installed.)
Preparing to unpack .../python3-msgpack_1.0.3-3build2_arm64.deb ...
Unpacking python3-msgpack (1.0.3-3build2) ...
Selecting previously unselected package borgbackup.
Preparing to unpack .../borgbackup_1.2.8-1_arm64.deb ...
Unpacking borgbackup (1.2.8-1) ...
Setting up python3-msgpack (1.0.3-3build2) ...
Setting up borgbackup (1.2.8-1) ...
Processing triggers for man-db (2.12.0-4build2) ...
Scanning processes...                                                                                     
Scanning processor microcode...                                                                           
Scanning linux images...                                                                                  

Running kernel seems to be up-to-date.

The processor microcode seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```
На сервере backup создаем пользователя borg и каталог /var/backup (каталог и диск ~2Gb создан и примонтированы в предыдущем шаге)
и назначаем на него права пользователя borg:
```
root@backup:~# useradd -m borg
root@backup:~# chown borg:borg /var/backup/
root@backup:/var/backup# ls -l /var/ | grep backup
drwxr-xr-x  3 borg borg   4096 Jul 20 18:48 backup
```
На сервер backup создаем каталог ~/.ssh/authorized keys в каталоге /home/borg
```
root@backup:/var/backup# su - borg
$ mkdir .ssh
$ touch .ssh/authorized_keys
$ chmod 700 .ssh
$ chmod 600 .ssh/authorized_keys
```
На сервере client:
```
root@client:~# ssh-keygen
Generating public/private ed25519 key pair.
Enter file in which to save the key (/root/.ssh/id_ed25519): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_ed25519
Your public key has been saved in /root/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:/n28UnRG2ThQKzQKFbW8106ROs0+0yEmpRhiEHLlAeM root@client
The key's randomart image is:
+--[ED25519 256]--+
|   . *+o..oo=o..o|
|    + + .. + ooo+|
|     E + .. +..= |
|      . . o oo=.+|
|        S. o.*.*o|
|       .    o.=oo|
|        .    o +o|
|         . .. o o|
|          . .o.. |
+----[SHA256]-----+
```
## Все дальнейшие действия будут проходить на client сервере.
Инициализируем репозиторий borg на backup сервере с client сервера:
```
root@client:~# borg init --encryption=repokey borg@192.168.1.77:/var/backup/
borg@192.168.1.77's password: 
Enter new passphrase: 
Enter same passphrase again: 
Do you want your passphrase to be displayed for verification? [yN]: n

By default repositories initialized with this version will produce security
errors if written to with an older version (up to and including Borg 1.0.8).

If you want to use these older versions, you can disable the check by running:
borg upgrade --disable-tam ssh://borg@192.168.1.77/var/backup

See https://borgbackup.readthedocs.io/en/stable/changes.html#pre-1-0-9-manifest-spoofing-vulnerability for details about the security implications.

IMPORTANT: you will need both KEY AND PASSPHRASE to access this repo!
If you used a repokey mode, the key is stored in the repo, but you should back it up separately.
Use "borg key export" to export the key, optionally in printable format.
Write down the passphrase. Store both at safe place(s).
```
Запускаем для проверки создание бэкапа (вывод сокращен):
```
root@client:~# borg create --stats --list borg@192.168.1.77:/var/backup/::"etc-{now:%Y-%m-%d_%H:%M:$S}" /etc
borg@192.168.1.77's password: 
Enter passphrase for key ssh://borg@192.168.1.77/var/backup:

...

------------------------------------------------------------------------------
Repository: ssh://borg@192.168.1.77/var/backup
Archive name: etc-2026-07-20_19:38:
Archive fingerprint: 3d4c92507d01bea0736befcc0daf598d014fb52ec6885495123d9b28d72cc547
Time (start): Mon, 2026-07-20 19:38:28
Time (end):   Mon, 2026-07-20 19:38:29
Duration: 0.57 seconds
Number of files: 850
Utilization of max. archive size: 0%
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
This archive:                2.30 MB              1.01 MB            976.43 kB
All archives:                2.30 MB              1.01 MB              1.05 MB

                       Unique chunks         Total chunks
Chunk index:                     808                  840
------------------------------------------------------------------------------
```
Смотрим, что у нас получилось
```
user@client:~$ borg list borg@192.168.1.77:/var/backup/
borg@192.168.1.77's password: 
Enter passphrase for key ssh://borg@192.168.1.77/var/backup: 
etc-2026-07-20_19:38:                Mon, 2026-07-20 19:38:28 [3d4c92507d01bea0736befcc0daf598d014fb52ec6885495123d9b28d72cc547]
etc-2026-07-21_18:51:05              Tue, 2026-07-21 18:51:12 [ea0b454aee9f9f1007279264de92f6fd5eab5e3a94dea2f299d430feb5623f51]
```
Смотрим список файлов (вывод сокращен)
```
user@client:~$ borg list borg@192.168.1.77:/var/backup/::etc-2026-07-21_18:51:05
borg@192.168.1.77's password: 
Enter passphrase for key ssh://borg@192.168.1.77/var/backup: 
drwxr-xr-x root   root          0 Mon, 2026-07-20 19:04:08 etc
lrwxrwxrwx root   root         27 Mon, 2026-07-20 18:39:51 etc/localtime -> /usr/share/zoneinfo/Etc/UTC
lrwxrwxrwx root   root         19 Tue, 2026-02-10 00:33:00 etc/mtab -> ../proc/self/mounts
lrwxrwxrwx root   root         21 Fri, 2026-02-06 07:23:01 etc/os-release -> ../usr/lib/os-release
lrwxrwxrwx root   root         39 Tue, 2026-02-10 00:33:25 etc/resolv.conf -> ../run/systemd/resolve/stub-resolv.conf
lrwxrwxrwx root   root         16 Tue, 2026-02-10 00:33:00 etc/vconsole.conf -> default/keyboard
lrwxrwxrwx root   root         23 Mon, 2024-02-26 12:58:31 etc/vtrgb -> /etc/alternatives/vtrgb
drwxr-xr-x root   root          0 Tue, 2026-02-10 00:41:41 etc/ModemManager
drwxr-xr-x root   root          0 Tue, 2026-02-10 00:41:41 etc/ModemManager/connection.d
drwxr-xr-x root   root          0 Tue, 2026-02-10 00:41:41 etc/ModemManager/fcc-unlock.d
...
```
Достаем файл из бекапа и проверяем его содержимое
```
user@client:~$ borg extract borg@192.168.1.77:/var/backup/::etc-2026-07-21_18:51:05 etc/hostname
borg@192.168.1.77's password: 
Enter passphrase for key ssh://borg@192.168.1.77/var/backup: 
user@client:~$ ls -l etc/hostname 
-rw-r--r-- 1 user user 7 Jul 20 18:38 etc/hostname
user@client:~$ cat etc/hostname
client
```
## Автоматизируем создание бэкапов с помощью systemd
Создаем сервис и таймер в каталоге /etc/systemd/system/
```
nano /etc/systemd/system/borg-backup.service
[Unit]
Description=Borg Backup

[Service]
Type=oneshot

# Парольная фраза
Environment="BORG_PASSPHRASE=Otus1234"

# Репозиторий
Environment=REPO=borg@192.168.11.160:/var/backup/

# Что бэкапим
Environment=BACKUP_TARGET=/etc

# Создание бэкапа
ExecStart=/bin/borg create --stats ${REPO}::etc-{now:%%Y-%%m-%%d_%%H:%%M:%%S} ${BACKUP_TARGET}

# Проверка бэкапа
ExecStart=/bin/borg check ${REPO}

# Очистка старых бэкапов
ExecStart=/bin/borg prune --keep-daily 90 --keep-monthly 12 --keep-yearly 1 ${REPO}


nano /etc/systemd/system/borg-backup.timer
[Unit]
Description=Borg Backup
[Timer]
OnUnitActiveSec=5min
[Install]
WantedBy=timers.target
```
Включаем и запускаем службу таймера
```
user@client:~$ sudo systemctl enable borg-backup.timer
Created symlink /etc/systemd/system/timers.target.wants/borg-backup.timer → /etc/systemd/system/borg-backup.timer.
user@client:~$ sudo systemctl start borg-backup.timer
user@client:~$ sudo systemctl status borg-backup.timer
● borg-backup.timer - Borg Backup
     Loaded: loaded (/etc/systemd/system/borg-backup.timer; enabled; preset: enabled)
     Active: active (elapsed) since Tue 2026-07-21 19:08:42 UTC; 17s ago
    Trigger: n/a
   Triggers: ● borg-backup.service

Jul 21 19:08:42 client systemd[1]: Started borg-backup.timer - Borg Backup.
```
Проверяем работу таймера
```
