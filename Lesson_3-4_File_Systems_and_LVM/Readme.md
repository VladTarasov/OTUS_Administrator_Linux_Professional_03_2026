# Занятие 3-4. Файловые системы и LVM

Домашнее задание: Работа с LVM

Цель: создавать и управлять логическими томами в LVM.

Описание/Пошаговая инструкция выполнения домашнего задания

На виртуальной машине с Ubuntu 24.04 и LVM.

1. Уменьшить том под / до 8G.
2. Выделить том под /var - сделать в mirror.
3. Прописать монтирование в fstab. Попробовать с разными опциями и разными файловыми системами (на выбор).
4. Работа со снапшотами (на примере тома для /home):
- выделить том под /home;
- сгенерить файлы в /home/;
- снять снапшот c /home;
- удалить часть файлов из /home;
- восстановить оригинальный /home с помощью снапшота.

  Исходное состояние стенда

```
root@ubuntu-server:/home/user# hostnamectl
 Static hostname: ubuntu-server
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 0343a9fa267b4dacaddcc66c086e0168
         Boot ID: 1f83106d8a0449158c2ee163e70312d6
  Virtualization: vmware
Operating System: Ubuntu 24.04.4 LTS
          Kernel: Linux 6.13.2-061302-generic
    Architecture: x86-64
 Hardware Vendor: VMware, Inc.
  Hardware Model: VMware Virtual Platform
Firmware Version: 6.00
   Firmware Date: Thu 2020-11-12
    Firmware Age: 5y 5month 1w 2d

root@ubuntu-server:/home/user# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   20G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1.8G  0 part /boot
└─sda3                      8:3    0 18.2G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   10G  0 lvm  /
sdb                         8:16   0   10G  0 disk
sdc                         8:32   0    2G  0 disk
sdd                         8:48   0    1G  0 disk
sde                         8:64   0    1G  0 disk
sdf                         8:80   0    1G  0 disk
sdg                         8:96   0    1G  0 disk
sr0                        11:0    1  3.2G  0 rom

root@ubuntu-server:/home/user# df -hT
Filesystem                        Type   Size  Used Avail Use% Mounted on
tmpfs                             tmpfs  387M  1.5M  386M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv ext4   9.8G  4.1G  5.2G  44% /
tmpfs                             tmpfs  1.9G     0  1.9G   0% /dev/shm
tmpfs                             tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/sda2                         ext4   1.8G  210M  1.4G  13% /boot
tmpfs                             tmpfs  387M   12K  387M   1% /run/user/1000
```

## 1. Уменьшить том под / до 8G.
Эту часть можно выполнить разными способами, в данном примере мы будем 
уменьшать / до 8G без использования LiveCD.
Подготовим временный том для / раздела:

```
root@ubuntu-server:/home/user# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
root@ubuntu-server:/home/user# vgcreate vg_root /dev/sdb
  Volume group "vg_root" successfully created
root@ubuntu-server:/home/user# lvcreate -n lv_root -l +100%FREE /dev/vg_root
WARNING: ext4 signature detected on /dev/vg_root/lv_root at offset 1080. Wipe it? [y/n]: y
  Wiping ext4 signature on /dev/vg_root/lv_root.
  Logical volume "lv_root" created.
```

Создадим на нем файловую систему и смонтируем его, чтобы перенести туда данные:

```
root@ubuntu-server:/home/user# mkfs.ext4 /dev/vg_root/lv_root
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 2620416 4k blocks and 655360 inodes
Filesystem UUID: f5b4d647-99c6-4a3b-abcc-7baabce187dd
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

root@ubuntu-server:/home/user# mount /dev/vg_root/lv_root /mnt
```

Cкопируем все данные с / раздела в /mnt (часть вывода для краткости скрыта):

```
root@ubuntu-server:/home/user# rsync -avxHAX --progress / /mnt/
sending incremental file list
./

...

sent 3,683,023,258 bytes  received 2,905,539 bytes  30,089,214.67 bytes/sec
total size is 3,675,887,443  speedup is 1.00
root@ubuntu-server:/home/user#
```

Проверим результат копирования командой `ls /mnt`:

```
root@ubuntu-server:/home/user# ls /mnt
bin                cdrom      dev   lib                lost+found  opt   root  sbin.usr-is-merged  sys  var
bin.usr-is-merged  data       etc   lib64              media       proc  run   snap                tmp
boot               data-snap  home  lib.usr-is-merged  mnt         raid  sbin  srv                 usr
root@ubuntu-server:/home/user#
```

Cконфигурируем grub для того, чтобы при загрузиться из нового /. 

```
root@ubuntu-server:/home/user# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
root@ubuntu-server:/home/user#
```

Сымитируем текущий root, сделаем в него chroot и обновим grub:

```
root@ubuntu-server:/home/user# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
root@ubuntu-server:/home/user# chroot /mnt/
root@ubuntu-server:/# grub-mkconfig -o /boot/grub/grub.cfg
Sourcing file `/etc/default/grub'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.13.2-061302-generic
Found initrd image: /boot/initrd.img-6.13.2-061302-generic
Found linux image: /boot/vmlinuz-6.8.0-110-generic
Found initrd image: /boot/initrd.img-6.8.0-110-generic
Found linux image: /boot/vmlinuz-6.8.0-107-generic
Found initrd image: /boot/initrd.img-6.8.0-107-generic
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
```

Обновим образ initrd

```
root@ubuntu-server:/# update-initramfs -u
update-initramfs: Generating /boot/initrd.img-6.13.2-061302-generic
root@ubuntu-server:/#
```

Перезагружаемся, чтобы работать с новым разделом

```
root@ubuntu-server:/# reboot
Running in chroot, ignoring request.
root@ubuntu-server:/# exit
exit
root@ubuntu-server:/home/user# reboot

Broadcast message from root@ubuntu-server on pts/1 (Thu 2026-04-23 08:54:16 UTC):

The system will reboot now!

root@ubuntu-server:/home/user#
Remote side unexpectedly closed network connection

```

Посмотрим картину с дисками после перезагрузки:

```
root@ubuntu-server:/home/user# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   20G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1.8G  0 part /boot
└─sda3                      8:3    0 18.2G  0 part
  └─ubuntu--vg-ubuntu--lv 252:1    0   10G  0 lvm
sdb                         8:16   0   10G  0 disk
└─vg_root-lv_root         252:0    0   10G  0 lvm  /
sdc                         8:32   0    2G  0 disk
sdd                         8:48   0    1G  0 disk
sde                         8:64   0    1G  0 disk
sdf                         8:80   0    1G  0 disk
sdg                         8:96   0    1G  0 disk
sr0                        11:0    1  3.2G  0 rom
```

Теперь нам нужно изменить размер старой VG и вернуть на него рут. Для этого 
удаляем старый LV размером в 40G и создаём новый на 8G:

```
root@ubuntu-server:/home/user# lvremove /dev/ubuntu-vg/ubuntu-lv
Do you really want to remove and DISCARD active logical volume ubuntu-vg/ubuntu-lv? [y/n]: y
  Logical volume "ubuntu-lv" successfully removed.
root@ubuntu-server:/home/user# lvcreate -n ubuntu-vg/ubuntu-lv -L 8G /dev/ubuntu-vg
WARNING: ext4 signature detected on /dev/ubuntu-vg/ubuntu-lv at offset 1080. Wipe it? [y/n]: y
  Wiping ext4 signature on /dev/ubuntu-vg/ubuntu-lv.
  Logical volume "ubuntu-lv" created.
```

Создадим на нем файловую систему, смонтируем его и перенесем туда данные:

```
root@ubuntu-server:/home/user# mkfs.ext4 /dev/ubuntu-vg/ubuntu-lv
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 2097152 4k blocks and 524288 inodes
Filesystem UUID: 4ef37afb-96c5-4d68-bf79-8e13fef27dfa
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

root@ubuntu-server:/home/user# mount /dev/ubuntu-vg/ubuntu-lv /mnt
root@ubuntu-server:/home/user# rsync -avxHAX --progress / /mnt/
...
sent 3,708,635,314 bytes  received 2,905,617 bytes  78,137,703.81 bytes/sec
total size is 3,701,490,674  speedup is 1.00
```

Так же как в первый раз cконфигурируем grub:

```
root@ubuntu-server:/home/user# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
root@ubuntu-server:/home/user# chroot /mnt/
root@ubuntu-server:/# grub-mkconfig -o /boot/grub/grub.cfg
Sourcing file `/etc/default/grub'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.13.2-061302-generic
Found initrd image: /boot/initrd.img-6.13.2-061302-generic
Found linux image: /boot/vmlinuz-6.8.0-110-generic
Found initrd image: /boot/initrd.img-6.8.0-110-generic
Found linux image: /boot/vmlinuz-6.8.0-107-generic
Found initrd image: /boot/initrd.img-6.8.0-107-generic
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
root@ubuntu-server:/# update-initramfs -u
update-initramfs: Generating /boot/initrd.img-6.13.2-061302-generic
W: Couldn't identify type of root file system for fsck hook
root@ubuntu-server:/#
```

Не перезагружаемся и не выходим из под chroot - мы можем заодно перенести 
/var.

## 2. Выделить том под /var - сделать в mirror.

На свободных дисках создаем зеркало: 

```
root@ubuntu-server:/# pvcreate /dev/sd{c,d}
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.
root@ubuntu-server:/# vgcreate vg_var /dev/sd{c,d}
  Volume group "vg_var" successfully created
root@ubuntu-server:/# lvcreate -L 950M -m1 -n lv_var_mirror vg_var
  Rounding up size to full physical extent 952.00 MiB
  Logical volume "lv_var_mirror" created.
```

Создаем на нем ФС и перемещаем туда /var:

```
root@ubuntu-server:/# mkfs.ext4 /dev/vg_var/lv_var_mirror
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 243712 4k blocks and 60928 inodes
Filesystem UUID: 68ba2c7c-044e-49ac-a086-7a635ec30c73
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

root@ubuntu-server:/# mount /dev/vg_var/lv_var_mirror /mnt
root@ubuntu-server:/# cp -aR /var/* /mnt
root@ubuntu-server:/#
```

На всякий случай сохраняем содержимое старого var

```
root@ubuntu-server:/# mount /dev/vg_var/lv_var_mirror /mnt
root@ubuntu-server:/# cp -aR /var/* /mnt
root@ubuntu-server:/# mkdir /tmp/oldvar && mv /var/* /tmp/oldvar
root@ubuntu-server:/#
```

монтируем новый var в каталог /var: 

```
root@ubuntu-server:/# umount /mnt
root@ubuntu-server:/# mount /dev/vg_var/lv_var_mirror /var
root@ubuntu-server:/#
```
## 3. Прописать монтирование в fstab. Попробовать с разными опциями и разными файловыми системами (на выбор).

Правим fstab для автоматического монтирования /var (выбираем файловую систему ext4):

```
root@ubuntu-server:/# cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-otFM9gmDElqIHV4swTlidgej8nqxC8V5ypWhHadU9RUNNGlNF7LXaW7ZjW07er3U / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/1d3bb7d8-43c8-4f82-807b-575013604971 /boot ext4 defaults 0 1

root@ubuntu-server:/# echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab

root@ubuntu-server:~# cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-otFM9gmDElqIHV4swTlidgej8nqxC8V5ypWhHadU9RUNNGlNF7LXaW7ZjW07er3U / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/1d3bb7d8-43c8-4f82-807b-575013604971 /boot ext4 defaults 0 1
 /var ext4 defaults 0 0
UUID="68ba2c7c-044e-49ac-a086-7a635ec30c73" /var ext4 defaults 0 0
```

Перезагружаемся в новый (уменьшенный root) и удаляем временную Volume Group: 

```
root@ubuntu-server:/# exit
exit
root@ubuntu-server:/home/user# reboot

Broadcast message from root@ubuntu-server on pts/1 (Thu 2026-04-23 09:44:57 UTC):

The system will reboot now!

root@ubuntu-server:/home/user#

root@ubuntu-server:~# lsblk
NAME                            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                               8:0    0   20G  0 disk
├─sda1                            8:1    0    1M  0 part
├─sda2                            8:2    0  1.8G  0 part /boot
└─sda3                            8:3    0 18.2G  0 part
  └─ubuntu--vg-ubuntu--lv       252:0    0    8G  0 lvm  /
sdb                               8:16   0   10G  0 disk
sdc                               8:32   0    2G  0 disk
├─vg_var-lv_var_mirror_rmeta_0  252:1    0    4M  0 lvm
│ └─vg_var-lv_var_mirror        252:5    0  952M  0 lvm  /var
└─vg_var-lv_var_mirror_rimage_0 252:2    0  952M  0 lvm
  └─vg_var-lv_var_mirror        252:5    0  952M  0 lvm  /var
sdd                               8:48   0    1G  0 disk
sde                               8:64   0    1G  0 disk
├─vg_var-lv_var_mirror_rmeta_1  252:3    0    4M  0 lvm
│ └─vg_var-lv_var_mirror        252:5    0  952M  0 lvm  /var
└─vg_var-lv_var_mirror_rimage_1 252:4    0  952M  0 lvm
  └─vg_var-lv_var_mirror        252:5    0  952M  0 lvm  /var
sdf                               8:80   0    1G  0 disk
sdg                               8:96   0    1G  0 disk
sr0                              11:0    1  3.2G  0 rom

root@ubuntu-server:~# df -hT
Filesystem                        Type   Size  Used Avail Use% Mounted on
tmpfs                             tmpfs  387M  1.6M  386M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv ext4   7.8G  3.1G  4.3G  42% /
tmpfs                             tmpfs  1.9G     0  1.9G   0% /dev/shm
tmpfs                             tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/mapper/vg_var-lv_var_mirror  ext4   919M  548M  308M  65% /var
/dev/sda2                         ext4   1.8G  306M  1.4G  19% /boot
tmpfs                             tmpfs  387M   12K  387M   1% /run/user/1000

root@ubuntu-server:/home/user# lvs
  LV            VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  ubuntu-lv     ubuntu-vg -wi-ao----   8.00g
  lv_root       vg_root   -wi-a----- <10.00g
  lv_var_mirror vg_var    rwi-a-r--- 952.00m                                    100.00

root@ubuntu-server:/home/user# vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  ubuntu-vg   1   1   0 wz--n-  18.22g 10.22g
  vg_root     1   1   0 wz--n- <10.00g     0
  vg_var      2   1   0 wz--n-   2.99g  1.12g

root@ubuntu-server:/home/user# pvs
  PV         VG        Fmt  Attr PSize    PFree
  /dev/sda3  ubuntu-vg lvm2 a--    18.22g 10.22g
  /dev/sdb   vg_root   lvm2 a--   <10.00g     0
  /dev/sdc   vg_var    lvm2 a--    <2.00g  1.06g
  /dev/sdd   vg_var    lvm2 a--  1020.00m 64.00m

root@ubuntu-server:/home/user# lvremove /dev/vg_root/lv_root
Do you really want to remove and DISCARD active logical volume vg_root/lv_root? [y/n]: y
  Logical volume "lv_root" successfully removed.

root@ubuntu-server:/home/user# vgremove /dev/vg_root
  Volume group "vg_root" successfully removed

root@ubuntu-server:/home/user# pvremove /dev/sdb
  Labels on physical volume "/dev/sdb" successfully wiped.

root@ubuntu-server:/home/user# lsblk
NAME                            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                               8:0    0   20G  0 disk
├─sda1                            8:1    0    1M  0 part
├─sda2                            8:2    0  1.8G  0 part /boot
└─sda3                            8:3    0 18.2G  0 part
  └─ubuntu--vg-ubuntu--lv       252:1    0    8G  0 lvm  /
sdb                               8:16   0   10G  0 disk
sdc                               8:32   0    2G  0 disk
├─vg_var-lv_var_mirror_rmeta_0  252:2    0    4M  0 lvm
│ └─vg_var-lv_var_mirror        252:6    0  952M  0 lvm
└─vg_var-lv_var_mirror_rimage_0 252:3    0  952M  0 lvm
  └─vg_var-lv_var_mirror        252:6    0  952M  0 lvm
sdd                               8:48   0    1G  0 disk
├─vg_var-lv_var_mirror_rmeta_1  252:4    0    4M  0 lvm
│ └─vg_var-lv_var_mirror        252:6    0  952M  0 lvm
└─vg_var-lv_var_mirror_rimage_1 252:5    0  952M  0 lvm
  └─vg_var-lv_var_mirror        252:6    0  952M  0 lvm
sde                               8:64   0    1G  0 disk
sdf                               8:80   0    1G  0 disk
sdg                               8:96   0    1G  0 disk
sr0                              11:0    1  3.2G  0 rom

root@ubuntu-server:/home/user# lvs
  LV            VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  ubuntu-lv     ubuntu-vg -wi-ao----   8.00g
  lv_var_mirror vg_var    rwi-a-r--- 952.00m                                    100.00

root@ubuntu-server:/home/user# vgs
  VG        #PV #LV #SN Attr   VSize  VFree
  ubuntu-vg   1   1   0 wz--n- 18.22g 10.22g
  vg_var      2   1   0 wz--n-  2.99g  1.12g

root@ubuntu-server:/home/user# pvs
  PV         VG        Fmt  Attr PSize    PFree
  /dev/sda3  ubuntu-vg lvm2 a--    18.22g 10.22g
  /dev/sdc   vg_var    lvm2 a--    <2.00g  1.06g
  /dev/sdd   vg_var    lvm2 a--  1020.00m 64.00m
```

После перезагрузки видим уменьшенный root и смонтированный /var.

## 4. Работа со снапшотами (на примере тома для /home)
Выделяем том под /home по тому же принципу что делали для /var: 

```
root@ubuntu-server:~# lvcreate -n LogVol_Home -L 2G /dev/ubuntu-vg
  Logical volume "LogVol_Home" created.

root@ubuntu-server:~# lvs
  LV            VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol_Home   ubuntu-vg -wi-a-----   2.00g
  ubuntu-lv     ubuntu-vg -wi-ao----   8.00g
  lv_var_mirror vg_var    rwi-aor--- 952.00m                                    100.00

root@ubuntu-server:~# mkfs.ext4 /dev/ubuntu-vg/LogVol_Home
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 524288 4k blocks and 131072 inodes
Filesystem UUID: db1fef13-f2af-48be-b499-ff2a89088de0
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

root@ubuntu-server:~# mount /dev/ubuntu-vg/LogVol_Home /mnt/

root@ubuntu-server:~# df -hT
Filesystem                         Type   Size  Used Avail Use% Mounted on
tmpfs                              tmpfs  387M  1.6M  386M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  ext4   7.8G  3.1G  4.3G  42% /
tmpfs                              tmpfs  1.9G     0  1.9G   0% /dev/shm
tmpfs                              tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/mapper/vg_var-lv_var_mirror   ext4   919M  546M  310M  64% /var
/dev/sda2                          ext4   1.8G  306M  1.4G  19% /boot
tmpfs                              tmpfs  387M   12K  387M   1% /run/user/1000
/dev/mapper/ubuntu--vg-LogVol_Home ext4   2.0G   24K  1.8G   1% /mnt

root@ubuntu-server:~# cp -aR /home/* /mnt/

root@ubuntu-server:~# mkdir /tmp/home_old
root@ubuntu-server:~# cp -aR /home/* /tmp/home_old/
root@ubuntu-server:~# rm -rf /home/*

root@ubuntu-server:~# umount /mnt
root@ubuntu-server:~# mount /dev/ubuntu-vg/LogVol_Home /home/
root@ubuntu-server:~# df -hT
Filesystem                         Type   Size  Used Avail Use% Mounted on
tmpfs                              tmpfs  387M  1.6M  386M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  ext4   7.8G  3.1G  4.3G  42% /
tmpfs                              tmpfs  1.9G     0  1.9G   0% /dev/shm
tmpfs                              tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/mapper/vg_var-lv_var_mirror   ext4   919M  546M  310M  64% /var
/dev/sda2                          ext4   1.8G  306M  1.4G  19% /boot
tmpfs                              tmpfs  387M   12K  387M   1% /run/user/1000
/dev/mapper/ubuntu--vg-LogVol_Home ext4   2.0G  208M  1.6G  12% /home
```

Правим fstab для автоматического монтирования /home: 

```
root@ubuntu-server:~# echo "`blkid | grep Home | awk '{print $2}'` /home ext4 defaults 0 0" >> /etc/fstab
root@ubuntu-server:~# cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-otFM9gmDElqIHV4swTlidgej8nqxC8V5ypWhHadU9RUNNGlNF7LXaW7ZjW07er3U / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/1d3bb7d8-43c8-4f82-807b-575013604971 /boot ext4 defaults 0 1
UUID="68ba2c7c-044e-49ac-a086-7a635ec30c73" /var ext4 defaults 0 0
UUID="db1fef13-f2af-48be-b499-ff2a89088de0" /home ext4 defaults 0 0
```

Генерируем файлы в /home/

```
root@ubuntu-server:~# touch /home/file{1..20}
root@ubuntu-server:~# ll /home/
total 28
drwxr-xr-x  4 root root  4096 May  1 17:05 ./
drwxr-xr-x 26 root root  4096 Apr 22 14:42 ../
-rw-r--r--  1 root root     0 May  1 17:05 file1
-rw-r--r--  1 root root     0 May  1 17:05 file10
-rw-r--r--  1 root root     0 May  1 17:05 file11
-rw-r--r--  1 root root     0 May  1 17:05 file12
-rw-r--r--  1 root root     0 May  1 17:05 file13
-rw-r--r--  1 root root     0 May  1 17:05 file14
-rw-r--r--  1 root root     0 May  1 17:05 file15
-rw-r--r--  1 root root     0 May  1 17:05 file16
-rw-r--r--  1 root root     0 May  1 17:05 file17
-rw-r--r--  1 root root     0 May  1 17:05 file18
-rw-r--r--  1 root root     0 May  1 17:05 file19
-rw-r--r--  1 root root     0 May  1 17:05 file2
-rw-r--r--  1 root root     0 May  1 17:05 file20
-rw-r--r--  1 root root     0 May  1 17:05 file3
-rw-r--r--  1 root root     0 May  1 17:05 file4
-rw-r--r--  1 root root     0 May  1 17:05 file5
-rw-r--r--  1 root root     0 May  1 17:05 file6
-rw-r--r--  1 root root     0 May  1 17:05 file7
-rw-r--r--  1 root root     0 May  1 17:05 file8
-rw-r--r--  1 root root     0 May  1 17:05 file9
drwx------  2 root root 16384 May  1 16:56 lost+found/
drwxr-x---  5 user user  4096 May  1 16:41 user/
```

Создаем snapshot /home:

```
root@ubuntu-server:~# lvcreate -L 100MB -s -n home_snap /dev/ubuntu-vg/LogVol_Home
  Logical volume "home_snap" created.
```

Удалим часть файлов

```
root@ubuntu-server:~# rm -f /home/file{11..20}
root@ubuntu-server:~# ll /home/
total 28
drwxr-xr-x  4 root root  4096 May  1 17:13 ./
drwxr-xr-x 26 root root  4096 Apr 22 14:42 ../
-rw-r--r--  1 root root     0 May  1 17:05 file1
-rw-r--r--  1 root root     0 May  1 17:05 file10
-rw-r--r--  1 root root     0 May  1 17:05 file2
-rw-r--r--  1 root root     0 May  1 17:05 file3
-rw-r--r--  1 root root     0 May  1 17:05 file4
-rw-r--r--  1 root root     0 May  1 17:05 file5
-rw-r--r--  1 root root     0 May  1 17:05 file6
-rw-r--r--  1 root root     0 May  1 17:05 file7
-rw-r--r--  1 root root     0 May  1 17:05 file8
-rw-r--r--  1 root root     0 May  1 17:05 file9
drwx------  2 root root 16384 May  1 16:56 lost+found/
drwxr-x---  5 user user  4096 May  1 16:41 user/
```

Выполним восстановление из snapshot'a:

```
root@ubuntu-server:~# lvconvert --merge /dev/ubuntu-vg/home_snap
  Merging of volume ubuntu-vg/home_snap started.
  ubuntu-vg/LogVol_Home: Merged: 100.00%
root@ubuntu-server:~# mount /dev/ubuntu-vg/LogVol_Home /home/
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
root@ubuntu-server:~# ls -al /home
total 28
drwxr-xr-x  4 root root  4096 May  1 17:05 .
drwxr-xr-x 26 root root  4096 Apr 22 14:42 ..
-rw-r--r--  1 root root     0 May  1 17:05 file1
-rw-r--r--  1 root root     0 May  1 17:05 file10
-rw-r--r--  1 root root     0 May  1 17:05 file11
-rw-r--r--  1 root root     0 May  1 17:05 file12
-rw-r--r--  1 root root     0 May  1 17:05 file13
-rw-r--r--  1 root root     0 May  1 17:05 file14
-rw-r--r--  1 root root     0 May  1 17:05 file15
-rw-r--r--  1 root root     0 May  1 17:05 file16
-rw-r--r--  1 root root     0 May  1 17:05 file17
-rw-r--r--  1 root root     0 May  1 17:05 file18
-rw-r--r--  1 root root     0 May  1 17:05 file19
-rw-r--r--  1 root root     0 May  1 17:05 file2
-rw-r--r--  1 root root     0 May  1 17:05 file20
-rw-r--r--  1 root root     0 May  1 17:05 file3
-rw-r--r--  1 root root     0 May  1 17:05 file4
-rw-r--r--  1 root root     0 May  1 17:05 file5
-rw-r--r--  1 root root     0 May  1 17:05 file6
-rw-r--r--  1 root root     0 May  1 17:05 file7
-rw-r--r--  1 root root     0 May  1 17:05 file8
-rw-r--r--  1 root root     0 May  1 17:05 file9
drwx------  2 root root 16384 May  1 16:56 lost+found
drwxr-x---  5 user user  4096 May  1 16:41 user
```
Файлы 11-20 восстановлены.
