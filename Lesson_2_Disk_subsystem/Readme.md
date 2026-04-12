# Занятие 2. Дисковая подсистема
Задание 
- Добавить в виртуальную машину несколько дисков
- Собрать RAID-0/1/5/10 на выбор
- Сломать и починить RAID
- Создать GPT таблицу, пять разделов и смонтировать их в системе.

Задание выполнялось в соответствии с методическими рекомендациями.

## Собрать RAID на выбор
Добавим в виртуальную маршину 6 дисков размером 1 Гб. Проверим, что диски отображаются в системе:
```
user@ubuntu-server:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   20G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1.8G  0 part /boot
└─sda3                      8:3    0 18.2G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   10G  0 lvm  /
sdb                         8:16   0    1G  0 disk
sdc                         8:32   0    1G  0 disk
sdd                         8:48   0    1G  0 disk
sde                         8:64   0    1G  0 disk
sdf                         8:80   0    1G  0 disk
sdg                         8:96   0    1G  0 disk
sr0                        11:0    1  3.2G  0 rom
```
Видим наши 6 дисков с именами sd{b,c,d,e,f,g}.

Исходя из того, что дисков у нас четное количество и их больше 4-х, то у нас есть возможность собрать из них RAID-10.
Диски новые и ранее не использовались для создания массива, поэтому занулять на них суперблоки нет необходимости, пропустим этот шаг.
Создадим массив:
```
user@ubuntu-server:~$ sudo mdadm --create --verbose /dev/md0 -l 10 -n 6 /dev/sd{b,c,d,e,f,g}
[sudo] password for user:
mdadm: layout defaults to n2
mdadm: layout defaults to n2
mdadm: chunk size defaults to 512K
mdadm: size set to 1046528K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

Проверим корректность создания массива:
```
user@ubuntu-server:~$ cat /proc/mdstat
Personalities : [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid10 sdg[5] sdf[4] sde[3] sdd[2] sdc[1] sdb[0]
      3139584 blocks super 1.2 512K chunks 2 near-copies [6/6] [UUUUUU]

unused devices: <none>

user@ubuntu-server:~$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sun Apr 12 10:47:42 2026
        Raid Level : raid10
        Array Size : 3139584 (2.99 GiB 3.21 GB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 6
     Total Devices : 6
       Persistence : Superblock is persistent

       Update Time : Sun Apr 12 10:50:23 2026
             State : clean
    Active Devices : 6
   Working Devices : 6
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : ubuntu-server:0  (local to host ubuntu-server)
              UUID : 41d1fbc3:c4bfdf0f:8e6134c9:917b4e49
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde
       4       8       80        4      active sync set-A   /dev/sdf
       5       8       96        5      active sync set-B   /dev/sdg
```
Массив собрался корректно - `State : clean`.

## Сломать и починить RAID
Искусственно нарушим работу массива, переведя один из дисков в статус fail:
```
user@ubuntu-server:~$ sudo mdadm /dev/md0 --fail /dev/sde
mdadm: set /dev/sde faulty in /dev/md0
```
Проверим как это отразилось на состоянии массива:
```
user@ubuntu-server:~$ sudo mdadm /dev/md0 --fail /dev/sde
mdadm: set /dev/sde faulty in /dev/md0
user@ubuntu-server:~$ cat /proc/mdstat
Personalities : [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid10 sdg[5] sdf[4] sde[3](F) sdd[2] sdc[1] sdb[0]
      3139584 blocks super 1.2 512K chunks 2 near-copies [6/5] [UUU_UU]

unused devices: <none>

user@ubuntu-server:~$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sun Apr 12 10:47:42 2026
        Raid Level : raid10
        Array Size : 3139584 (2.99 GiB 3.21 GB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 6
     Total Devices : 6
       Persistence : Superblock is persistent

       Update Time : Sun Apr 12 11:02:39 2026
             State : clean, degraded
    Active Devices : 5
   Working Devices : 5
    Failed Devices : 1
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : ubuntu-server:0  (local to host ubuntu-server)
              UUID : 41d1fbc3:c4bfdf0f:8e6134c9:917b4e49
            Events : 19

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       -       0        0        3      removed
       4       8       80        4      active sync set-A   /dev/sdf
       5       8       96        5      active sync set-B   /dev/sdg

       3       8       64        -      faulty   /dev/sde
```
Видим, что диск sde в составе массива помечен как `removed faulty`, статус массива изменился на `State : clean, degraded`
Удалим диск из массива
```
user@ubuntu-server:~$ mdadm /dev/md0 --remove /dev/sde 
mdadm: hot removed /dev/sde from /dev/md0 
```
Вернем диск sde в состав массива
```
user@ubuntu-server:~$ sudo mdadm /dev/md0 --add /dev/sde
mdadm: added /dev/sde
```
Проверим состояние массива
```
user@ubuntu-server:~$ cat /proc/mdstat
Personalities : [raid10] [raid0] [raid1] [raid6] [raid5] [raid4]
md0 : active raid10 sde[6] sdb[0] sdd[2] sdc[1] sdg[5] sdf[4]
      3139584 blocks super 1.2 512K chunks 2 near-copies [6/6] [UUUUUU]

unused devices: <none>

user@ubuntu-server:~$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sun Apr 12 10:47:42 2026
        Raid Level : raid10
        Array Size : 3139584 (2.99 GiB 3.21 GB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 6
     Total Devices : 6
       Persistence : Superblock is persistent

       Update Time : Sun Apr 12 11:38:33 2026
             State : clean
    Active Devices : 6
   Working Devices : 6
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : ubuntu-server:0  (local to host ubuntu-server)
              UUID : 41d1fbc3:c4bfdf0f:8e6134c9:917b4e49
            Events : 39

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       6       8       64        3      active sync set-B   /dev/sde
       4       8       80        4      active sync set-A   /dev/sdf
       5       8       96        5      active sync set-B   /dev/sdg
```
Видим, что массив вернулся в исходное состояние 'clean'. Непосредвенно статус 'rebuilding' зафиксировать в выводах не удалось, вероятно, восстановление произошло очень быстро.

## Создать GPT таблицу, пять разделов и смонтировать их в системе
Создаем раздел GPT на RAID
```
user@ubuntu-server:~$ sudo parted -s /dev/md127 mklabel gpt
```
Создаем партиции
```
user@ubuntu-server:~$ sudo parted /dev/md127 mkpart primary ext4 0% 20%
Information: You may need to update /etc/fstab.

user@ubuntu-server:~$ sudo parted /dev/md127 mkpart primary ext4 20% 40%
Information: You may need to update /etc/fstab.

user@ubuntu-server:~$ sudo parted /dev/md127 mkpart primary ext4 40% 60%
Information: You may need to update /etc/fstab.

user@ubuntu-server:~$ sudo parted /dev/md127 mkpart primary ext4 60% 80%
Information: You may need to update /etc/fstab.

user@ubuntu-server:~$ sudo parted /dev/md127 mkpart primary ext4 80% 100%
Information: You may need to update /etc/fstab.
```

Создадим файловые системы на этих разделах:
```
user@ubuntu-server:~$ sudo mkfs.ext4 /dev/md127p1
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 156672 4k blocks and 39200 inodes
Filesystem UUID: 300e30aa-d43c-4139-a2f1-af8fdc2587b6
Superblock backups stored on blocks:
        32768, 98304

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

user@ubuntu-server:~$ sudo mkfs.ext4 /dev/md127p2
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 157056 4k blocks and 39280 inodes
Filesystem UUID: 4eaae734-4319-4dad-bec7-ba4a9758451d
Superblock backups stored on blocks:
        32768, 98304

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

user@ubuntu-server:~$ sudo mkfs.ext4 /dev/md127p3
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 156672 4k blocks and 39200 inodes
Filesystem UUID: e0dea3e7-f4f7-43fb-9bac-d7e067e1dbea
Superblock backups stored on blocks:
        32768, 98304

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

user@ubuntu-server:~$ sudo mkfs.ext4 /dev/md127p4
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 157056 4k blocks and 39280 inodes
Filesystem UUID: 16f29d77-82d9-4885-97dd-baa49e84ebfc
Superblock backups stored on blocks:
        32768, 98304

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

user@ubuntu-server:~$ sudo mkfs.ext4 /dev/md127p5
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 156672 4k blocks and 39200 inodes
Filesystem UUID: 65b0ab41-1650-4ca5-9e2e-9f7a9dabe92c
Superblock backups stored on blocks:
        32768, 98304

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
```
Создадим каталоги и смонтируем в них разделы
```
user@ubuntu-server:~$ sudo mkdir -p /raid/part{1,2,3,4,5}

user@ubuntu-server:~$ ls -l /raid/
total 20
drwxr-xr-x 2 root root 4096 Apr 12 12:05 part1
drwxr-xr-x 2 root root 4096 Apr 12 12:05 part2
drwxr-xr-x 2 root root 4096 Apr 12 12:05 part3
drwxr-xr-x 2 root root 4096 Apr 12 12:05 part4
drwxr-xr-x 2 root root 4096 Apr 12 12:05 part5

user@ubuntu-server:~$ sudo mount /dev/md127p1 /raid/part1
user@ubuntu-server:~$ sudo mount /dev/md127p2 /raid/part2
user@ubuntu-server:~$ sudo mount /dev/md127p3 /raid/part3
user@ubuntu-server:~$ sudo mount /dev/md127p4 /raid/part4
user@ubuntu-server:~$ sudo mount /dev/md127p5 /raid/part5

user@ubuntu-server:~$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              387M  1.6M  386M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  9.8G  3.2G  6.1G  35% /
tmpfs                              1.9G     0  1.9G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          1.8G  210M  1.4G  13% /boot
tmpfs                              387M   12K  387M   1% /run/user/1000
/dev/md127p1                       586M   24K  543M   1% /raid/part1
/dev/md127p2                       587M   24K  544M   1% /raid/part2
/dev/md127p3                       586M   24K  543M   1% /raid/part3
/dev/md127p4                       587M   24K  544M   1% /raid/part4
/dev/md127p5                       586M   24K  543M   1% /raid/part5
```
