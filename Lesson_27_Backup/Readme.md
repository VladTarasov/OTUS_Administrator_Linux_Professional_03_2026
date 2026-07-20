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
