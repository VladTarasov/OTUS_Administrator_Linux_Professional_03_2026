# Занятие 5. ZFS

## Цели домашнего задания

Научится самостоятельно устанавливать ZFS, настраивать пулы, изучить основные возможности ZFS. 

## Описание домашнего задания

1. Определить алгоритм с наилучшим сжатием:
- Определить какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);
- создать 4 файловых системы на каждой применить свой алгоритм сжатия;
- для сжатия использовать либо текстовый файл, либо группу файлов.
2. Определить настройки пула.
С помощью команды zfs import собрать pool ZFS.
Командами zfs определить настройки:
    - размер хранилища;
    - тип pool;
    - значение recordsize;
    - какое сжатие используется;
    - какая контрольная сумма используется.
3. Работа со снапшотами:
- скопировать файл из удаленной директории;

## Создаем виртуальную машину 
Создаём виртуальную машину с Ubuntu 24.04 Server. Помимо 
системного диска добавляем 8 дополнительных дисков по 512 MB. 
Заходим на сервер по SSH. Дальнейшие действия выполняются от 
пользователя root. Переходим в root пользователя: sudo -i 

## 1. Определение алгоритма с наилучшим сжатием
Смотрим список всех дисков, которые есть в виртуальной машине: 

```
user@ubuntu24:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   20G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1.8G  0 part /boot
└─sda3                      8:3    0 18.2G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   10G  0 lvm  /
sdb                         8:16   0  512M  0 disk
sdc                         8:32   0  512M  0 disk
sdd                         8:48   0  512M  0 disk
sde                         8:64   0  512M  0 disk
sdf                         8:80   0  512M  0 disk
sdg                         8:96   0  512M  0 disk
sdh                         8:112  0  512M  0 disk
sdi                         8:128  0  512M  0 disk
sr0                        11:0    1  3.2G  0 rom
```

Установим пакет утилит для ZFS:

```
root@ubuntu24:~# apt install zfsutils-linux
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libnvpair3linux libuutil3linux libzfs4linux libzpool5linux zfs-zed
Suggested packages:
  nfs-kernel-server samba-common-bin zfs-initramfs | zfs-dracut
The following NEW packages will be installed:
  libnvpair3linux libuutil3linux libzfs4linux libzpool5linux zfs-zed zfsutils-linux
0 upgraded, 6 newly installed, 0 to remove and 35 not upgraded.
Need to get 2,355 kB of archives.
After this operation, 7,399 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libnvpair3linux amd64 2.2.2-0ubuntu9.4 [62.1 kB]
Get:2 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libuutil3linux amd64 2.2.2-0ubuntu9.4 [53.2 kB]
Get:3 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libzfs4linux amd64 2.2.2-0ubuntu9.4 [224 kB]
Get:4 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libzpool5linux amd64 2.2.2-0ubuntu9.4 [1,397 kB]
Get:5 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 zfsutils-linux amd64 2.2.2-0ubuntu9.4 [551 kB]
Get:6 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 zfs-zed amd64 2.2.2-0ubuntu9.4 [67.9 kB]
Fetched 2,355 kB in 8s (308 kB/s)
Selecting previously unselected package libnvpair3linux.
(Reading database ... 88211 files and directories currently installed.)
Preparing to unpack .../0-libnvpair3linux_2.2.2-0ubuntu9.4_amd64.deb ...
Unpacking libnvpair3linux (2.2.2-0ubuntu9.4) ...
Selecting previously unselected package libuutil3linux.
Preparing to unpack .../1-libuutil3linux_2.2.2-0ubuntu9.4_amd64.deb ...
Unpacking libuutil3linux (2.2.2-0ubuntu9.4) ...
Selecting previously unselected package libzfs4linux.
Preparing to unpack .../2-libzfs4linux_2.2.2-0ubuntu9.4_amd64.deb ...
Unpacking libzfs4linux (2.2.2-0ubuntu9.4) ...
Selecting previously unselected package libzpool5linux.
Preparing to unpack .../3-libzpool5linux_2.2.2-0ubuntu9.4_amd64.deb ...
Unpacking libzpool5linux (2.2.2-0ubuntu9.4) ...
Selecting previously unselected package zfsutils-linux.
Preparing to unpack .../4-zfsutils-linux_2.2.2-0ubuntu9.4_amd64.deb ...
Unpacking zfsutils-linux (2.2.2-0ubuntu9.4) ...
Selecting previously unselected package zfs-zed.
Preparing to unpack .../5-zfs-zed_2.2.2-0ubuntu9.4_amd64.deb ...
Unpacking zfs-zed (2.2.2-0ubuntu9.4) ...
Setting up libnvpair3linux (2.2.2-0ubuntu9.4) ...
Setting up libuutil3linux (2.2.2-0ubuntu9.4) ...
Setting up libzpool5linux (2.2.2-0ubuntu9.4) ...
Setting up libzfs4linux (2.2.2-0ubuntu9.4) ...
Setting up zfsutils-linux (2.2.2-0ubuntu9.4) ...
insmod /lib/modules/6.8.0-111-generic/kernel/zfs/spl.ko.zst
insmod /lib/modules/6.8.0-111-generic/kernel/zfs/zfs.ko.zst
Created symlink /etc/systemd/system/zfs-import.target.wants/zfs-import-cache.service → /usr/lib/systemd/system/zfs-import-cache.service.
Created symlink /etc/systemd/system/zfs.target.wants/zfs-import.target → /usr/lib/systemd/system/zfs-import.target.
Created symlink /etc/systemd/system/zfs-mount.service.wants/zfs-load-module.service → /usr/lib/systemd/system/zfs-load-module.service.
Created symlink /etc/systemd/system/zfs.target.wants/zfs-load-module.service → /usr/lib/systemd/system/zfs-load-module.service.
Created symlink /etc/systemd/system/zfs.target.wants/zfs-mount.service → /usr/lib/systemd/system/zfs-mount.service.
Created symlink /etc/systemd/system/zfs.target.wants/zfs-share.service → /usr/lib/systemd/system/zfs-share.service.
Created symlink /etc/systemd/system/zfs-volumes.target.wants/zfs-volume-wait.service → /usr/lib/systemd/system/zfs-volume-wait.service.
Created symlink /etc/systemd/system/zfs.target.wants/zfs-volumes.target → /usr/lib/systemd/system/zfs-volumes.target.
Created symlink /etc/systemd/system/multi-user.target.wants/zfs.target → /usr/lib/systemd/system/zfs.target.
zfs-import-scan.service is a disabled or a static unit, not starting it.
Setting up zfs-zed (2.2.2-0ubuntu9.4) ...
Created symlink /etc/systemd/system/zed.service → /usr/lib/systemd/system/zfs-zed.service.
Created symlink /etc/systemd/system/zfs.target.wants/zfs-zed.service → /usr/lib/systemd/system/zfs-zed.service.
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.7) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```

Создаём 4 пула из двух дисков в режиме RAID 1:

```
root@ubuntu24:~# zpool create otus1 mirror /dev/sdb /dev/sdc
root@ubuntu24:~# zpool create otus2 mirror /dev/sdd /dev/sde
root@ubuntu24:~# zpool create otus3 mirror /dev/sdf /dev/sdg
root@ubuntu24:~# zpool create otus4 mirror /dev/sdh /dev/sdi
```

Проверяем информацию о пулах:

```
root@ubuntu24:~# zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus1   480M   110K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus2   480M   114K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus3   480M   111K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus4   480M   110K   480M        -         -     0%     0%  1.00x    ONLINE  -
```

Настроим разные алгоритмы сжатия для созданных пулов:

```
root@ubuntu24:~# zfs set compression=lzjb otus1
root@ubuntu24:~# zfs set compression=lz4 otus2
root@ubuntu24:~# zfs set compression=gzip-9 otus3
root@ubuntu24:~# zfs set compression=zle otus4
```

Проверим, что все файловые системы имеют разные методы сжатия:

```
root@ubuntu24:~# zfs get all | grep compression
otus1  compression           lzjb                   local
otus2  compression           lz4                    local
otus3  compression           gzip-9                 local
otus4  compression           zle                    local
```

Скачаем один и тот же текстовый файл во все пулы:
```
root@ubuntu24:~# for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
--2026-05-03 09:47:59--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41235281 (39M) [text/plain]
Saving to: ‘/otus1/pg2600.converter.log’

pg2600.converter.log            100%[====================================================>]  39.32M   740KB/s    in 42s

2026-05-03 09:48:48 (949 KB/s) - ‘/otus1/pg2600.converter.log’ saved [41235281/41235281]

--2026-05-03 09:48:48--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41235281 (39M) [text/plain]
Saving to: ‘/otus2/pg2600.converter.log’

pg2600.converter.log            100%[====================================================>]  39.32M  4.63MB/s    in 8.7s

2026-05-03 09:49:01 (4.53 MB/s) - ‘/otus2/pg2600.converter.log’ saved [41235281/41235281]

--2026-05-03 09:49:01--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41235281 (39M) [text/plain]
Saving to: ‘/otus3/pg2600.converter.log’

pg2600.converter.log            100%[====================================================>]  39.32M   672KB/s    in 22s

2026-05-03 09:49:27 (1.78 MB/s) - ‘/otus3/pg2600.converter.log’ saved [41235281/41235281]

--2026-05-03 09:49:27--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41235281 (39M) [text/plain]
Saving to: ‘/otus4/pg2600.converter.log’

pg2600.converter.log            100%[====================================================>]  39.32M   466KB/s    in 36s

2026-05-03 09:50:06 (1.09 MB/s) - ‘/otus4/pg2600.converter.log’ saved [41235281/41235281]
```

Проверим, что файл был скачан во все пулы:

```
root@ubuntu24:~# ls -l /otus*
/otus1:
total 22126
-rw-r--r-- 1 root root 41235281 May  2 07:31 pg2600.converter.log

/otus2:
total 18019
-rw-r--r-- 1 root root 41235281 May  2 07:31 pg2600.converter.log

/otus3:
total 10972
-rw-r--r-- 1 root root 41235281 May  2 07:31 pg2600.converter.log

/otus4:
total 40298
-rw-r--r-- 1 root root 41235281 May  2 07:31 pg2600.converter.log
```

Уже на этом этапе видно, что самый оптимальный метод сжатия используется в пуле otus3. 
Проверим, сколько места занимает один и тот же файл в разных пулах и оценим степень его сжатия:

```
root@ubuntu24:~# zfs list
NAME    USED  AVAIL  REFER  MOUNTPOINT
otus1  21.8M   330M  21.6M  /otus1
otus2  17.7M   334M  17.6M  /otus2
otus3  10.9M   341M  10.7M  /otus3
otus4  39.5M   312M  39.4M  /otus4

root@ubuntu24:~# zfs get all | grep compressratio | grep -v ref
otus1  compressratio         1.82x                  -
otus2  compressratio         2.23x                  -
otus3  compressratio         3.66x                  -
otus4  compressratio         1.00x                  -
```

Вывод:
- Наиболее эффективным алгоритмом сжатия оказался gzip-9.
- Без сжатия отработал алгоритм zle, что логично, исходя из его предназначения - сжатие ранее сжатых данных (видео, аудио, фото и архивы).

