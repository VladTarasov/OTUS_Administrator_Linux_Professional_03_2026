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

root@ubuntu24:~# zpool status
  pool: otus1
 state: ONLINE
config:

        NAME        STATE     READ WRITE CKSUM
        otus1       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdb     ONLINE       0     0     0
            sdc     ONLINE       0     0     0

errors: No known data errors

  pool: otus2
 state: ONLINE
config:

        NAME        STATE     READ WRITE CKSUM
        otus2       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdd     ONLINE       0     0     0
            sde     ONLINE       0     0     0

errors: No known data errors

  pool: otus3
 state: ONLINE
config:

        NAME        STATE     READ WRITE CKSUM
        otus3       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdf     ONLINE       0     0     0
            sdg     ONLINE       0     0     0

errors: No known data errors

  pool: otus4
 state: ONLINE
config:

        NAME        STATE     READ WRITE CKSUM
        otus4       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdh     ONLINE       0     0     0
            sdi     ONLINE       0     0     0

errors: No known data errors
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

**Выводы:**
- Наиболее эффективным алгоритмом сжатия оказался gzip-9.
- Без сжатия отработал алгоритм zle, что логично, исходя из его предназначения - сжатие ранее сжатых данных (видео, аудио, фото и архивы).

## 2.  Определение настроек пула и файловой системы

Скачиваем архив в домашний каталог: 

```
root@ubuntu24:~# wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download'
--2026-05-03 10:32:52--  https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download
Resolving drive.usercontent.google.com (drive.usercontent.google.com)... 64.233.164.132, 2a00:1450:4010:c07::84
Connecting to drive.usercontent.google.com (drive.usercontent.google.com)|64.233.164.132|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7275140 (6.9M) [application/octet-stream]
Saving to: ‘archive.tar.gz’

archive.tar.gz                  100%[====================================================>]   6.94M  6.67MB/s    in 1.0s

2026-05-03 10:32:58 (6.67 MB/s) - ‘archive.tar.gz’ saved [7275140/7275140]

root@ubuntu24:~# ls -l
total 7108
-rw-r--r-- 1 root root 7275140 Dec  6  2023 archive.tar.gz
```

Разархивируем его:

```
root@ubuntu24:~# tar -xzvf archive.tar.gz
zpoolexport/
zpoolexport/filea
zpoolexport/fileb
root@ubuntu24:~# ls -l
total 7112
-rw-r--r-- 1 root root 7275140 Dec  6  2023 archive.tar.gz
drwxr-xr-x 2 root root    4096 May 15  2020 zpoolexport
root@ubuntu24:~# ls -l zpoolexport
total 1024008
-rw-r--r-- 1 root root 524288000 May 15  2020 filea
-rw-r--r-- 1 root root 524288000 May 15  2020 fileb
```

Проверим, возможно ли импортировать данный каталог в пул:

```
root@ubuntu24:~# zpool import -d zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
status: Some supported features are not enabled on the pool.
        (Note that they may be intentionally disabled if the
        'compatibility' property is set.)
 action: The pool can be imported using its name or numeric identifier, though
        some features will not be available without an explicit 'zpool upgrade'.
 config:

        otus                         ONLINE
          mirror-0                   ONLINE
            /root/zpoolexport/filea  ONLINE
            /root/zpoolexport/fileb  ONLINE
```

Данный вывод показывает нам имя пула, тип raid и его состав.  
Сделаем импорт данного пула к нам в ОС: 

```
root@ubuntu24:~# zpool import -d zpoolexport/ otus

root@ubuntu24:~# zpool status otus
  pool: otus
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
        The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
        the pool may no longer be accessible by software that does not support
        the features. See zpool-features(7) for details.
config:

        NAME                         STATE     READ WRITE CKSUM
        otus                         ONLINE       0     0     0
          mirror-0                   ONLINE       0     0     0
            /root/zpoolexport/filea  ONLINE       0     0     0
            /root/zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors
```

Далее определим параметры пула otus:

```
root@ubuntu24:~# zpool get all otus
NAME  PROPERTY                       VALUE                          SOURCE
otus  size                           480M                           -
otus  capacity                       0%                             -
otus  altroot                        -                              default
otus  health                         ONLINE                         -
otus  guid                           6554193320433390805            -
otus  version                        -                              default
otus  bootfs                         -                              default
otus  delegation                     on                             default
otus  autoreplace                    off                            default
otus  cachefile                      -                              default
otus  failmode                       wait                           default
otus  listsnapshots                  off                            default
otus  autoexpand                     off                            default
otus  dedupratio                     1.00x                          -
otus  free                           478M                           -
otus  allocated                      2.09M                          -
otus  readonly                       off                            -
otus  ashift                         0                              default
otus  comment                        -                              default
otus  expandsize                     -                              -
otus  freeing                        0                              -
otus  fragmentation                  0%                             -
otus  leaked                         0                              -
otus  multihost                      off                            default
otus  checkpoint                     -                              -
otus  load_guid                      17182652176502306074           -
otus  autotrim                       off                            default
otus  compatibility                  off                            default
otus  bcloneused                     0                              -
otus  bclonesaved                    0                              -
otus  bcloneratio                    1.00x                          -
otus  feature@async_destroy          enabled                        local
otus  feature@empty_bpobj            active                         local
otus  feature@lz4_compress           active                         local
otus  feature@multi_vdev_crash_dump  enabled                        local
otus  feature@spacemap_histogram     active                         local
otus  feature@enabled_txg            active                         local
otus  feature@hole_birth             active                         local
otus  feature@extensible_dataset     active                         local
otus  feature@embedded_data          active                         local
otus  feature@bookmarks              enabled                        local
otus  feature@filesystem_limits      enabled                        local
otus  feature@large_blocks           enabled                        local
otus  feature@large_dnode            enabled                        local
otus  feature@sha512                 enabled                        local
otus  feature@skein                  enabled                        local
otus  feature@edonr                  enabled                        local
otus  feature@userobj_accounting     active                         local
otus  feature@encryption             enabled                        local
otus  feature@project_quota          active                         local
otus  feature@device_removal         enabled                        local
otus  feature@obsolete_counts        enabled                        local
otus  feature@zpool_checkpoint       enabled                        local
otus  feature@spacemap_v2            active                         local
otus  feature@allocation_classes     enabled                        local
otus  feature@resilver_defer         enabled                        local
otus  feature@bookmark_v2            enabled                        local
otus  feature@redaction_bookmarks    disabled                       local
otus  feature@redacted_datasets      disabled                       local
otus  feature@bookmark_written       disabled                       local
otus  feature@log_spacemap           disabled                       local
otus  feature@livelist               disabled                       local
otus  feature@device_rebuild         disabled                       local
otus  feature@zstd_compress          disabled                       local
otus  feature@draid                  disabled                       local
otus  feature@zilsaxattr             disabled                       local
otus  feature@head_errlog            disabled                       local
otus  feature@blake3                 disabled                       local
otus  feature@block_cloning          disabled                       local
otus  feature@vdev_zaps_v2           disabled                       local
```

И параметры файловой системы для пула otus:

```
root@ubuntu24:~# zfs get all otus
NAME  PROPERTY              VALUE                  SOURCE
otus  type                  filesystem             -
otus  creation              Fri May 15  4:00 2020  -
otus  used                  2.04M                  -
otus  available             350M                   -
otus  referenced            24K                    -
otus  compressratio         1.00x                  -
otus  mounted               yes                    -
otus  quota                 none                   default
otus  reservation           none                   default
otus  recordsize            128K                   local
otus  mountpoint            /otus                  default
otus  sharenfs              off                    default
otus  checksum              sha256                 local
otus  compression           zle                    local
otus  atime                 on                     default
otus  devices               on                     default
otus  exec                  on                     default
otus  setuid                on                     default
otus  readonly              off                    default
otus  zoned                 off                    default
otus  snapdir               hidden                 default
otus  aclmode               discard                default
otus  aclinherit            restricted             default
otus  createtxg             1                      -
otus  canmount              on                     default
otus  xattr                 on                     default
otus  copies                1                      default
otus  version               5                      -
otus  utf8only              off                    -
otus  normalization         none                   -
otus  casesensitivity       sensitive              -
otus  vscan                 off                    default
otus  nbmand                off                    default
otus  sharesmb              off                    default
otus  refquota              none                   default
otus  refreservation        none                   default
otus  guid                  14592242904030363272   -
otus  primarycache          all                    default
otus  secondarycache        all                    default
otus  usedbysnapshots       0B                     -
otus  usedbydataset         24K                    -
otus  usedbychildren        2.01M                  -
otus  usedbyrefreservation  0B                     -
otus  logbias               latency                default
otus  objsetid              54                     -
otus  dedup                 off                    default
otus  mlslabel              none                   default
otus  sync                  standard               default
otus  dnodesize             legacy                 default
otus  refcompressratio      1.00x                  -
otus  written               24K                    -
otus  logicalused           1020K                  -
otus  logicalreferenced     12K                    -
otus  volmode               default                default
otus  filesystem_limit      none                   default
otus  snapshot_limit        none                   default
otus  filesystem_count      none                   default
otus  snapshot_count        none                   default
otus  snapdev               hidden                 default
otus  acltype               off                    default
otus  context               none                   default
otus  fscontext             none                   default
otus  defcontext            none                   default
otus  rootcontext           none                   default
otus  relatime              on                     default
otus  redundant_metadata    all                    default
otus  overlay               on                     default
otus  encryption            off                    default
otus  keylocation           none                   default
otus  keyformat             none                   default
otus  pbkdf2iters           0                      default
otus  special_small_blocks  0                      default
```

С помощью фильров для команды get уточним значения конкретных параметров: 

Размер: 

```
root@ubuntu24:~# zfs get available otus
NAME  PROPERTY   VALUE  SOURCE
otus  available  350M   -
```

Тип:  

```
root@ubuntu24:~# zfs get readonly otus
NAME  PROPERTY  VALUE   SOURCE
otus  readonly  off     default
```

По значению off можем понять, что система доступна для чтения и записи.

Значение recordsize: 

```
root@ubuntu24:~# zfs get recordsize otus
NAME  PROPERTY    VALUE    SOURCE
otus  recordsize  128K     local
```

Тип сжатия (или параметр отключения): 

```
root@ubuntu24:~# zfs get compression otus
NAME  PROPERTY     VALUE           SOURCE
otus  compression  zle             local
```

Тип хэш-функции для контрольной суммы: 

```
root@ubuntu24:~#  zfs get checksum otus
NAME  PROPERTY  VALUE      SOURCE
otus  checksum  sha256     local
```

## 3. Работа со снапшотами

Скачаем файл, указанный в задании:

```
root@ubuntu24:~# wget -O otus_task2.file --no-check-certificate 'https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download'
--2026-05-03 18:36:56--  https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download
Resolving drive.usercontent.google.com (drive.usercontent.google.com)... 64.233.164.132, 2a00:1450:4010:c07::84
Connecting to drive.usercontent.google.com (drive.usercontent.google.com)|64.233.164.132|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5432736 (5.2M) [application/octet-stream]
Saving to: ‘otus_task2.file’

otus_task2.file                 100%[====================================================>]   5.18M  8.22MB/s    in 0.6s

2026-05-03 18:36:59 (8.22 MB/s) - ‘otus_task2.file’ saved [5432736/5432736]

root@ubuntu24:~# ls -l
total 12424
-rw-r--r-- 1 root root 7275140 Dec  6  2023 archive.tar.gz
-rw-r--r-- 1 root root 5432736 Dec  6  2023 otus_task2.file
-rw-r--r-- 1 root root     917 May  3 18:35 wget-log
drwxr-xr-x 2 root root    4096 May 15  2020 zpoolexport
```

