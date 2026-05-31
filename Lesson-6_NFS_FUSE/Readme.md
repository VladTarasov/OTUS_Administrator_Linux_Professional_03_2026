# Занятие 6. NFS, FUSE

**Цель домашнего задания**\
Научиться самостоятельно разворачивать сервис NFS и подключать к нему клиентов.

**Описание домашнего задания**\
Основная часть: 
- запустить 2 виртуальных машины (сервер NFS и клиента);
- на сервере NFS должна быть подготовлена и экспортирована директория; 
- в экспортированной директории должна быть поддиректория с именем upload с правами на запись в неё; 
- экспортированная директория должна автоматически монтироваться на клиенте при старте виртуальной машины (systemd, autofs или fstab — любым способом);
- монтирование и работа NFS на клиенте должна быть организована с использованием NFSv3.

Для самостоятельной реализации: 
- настроить аутентификацию через KERBEROS с использованием NFSv4.

## Создаём тестовые виртуальные машины 
Создаём 2 виртуальные машины с сетевыми интерфейсами, которые позволяют связь между ними. 
Далее будем называть ВМ с NFS сервером nfss (IP 192.168.1.73), а ВМ с клиентом nfsc (IP 192.168.1.74).

## Настраиваем сервер NFS
Заходим на сервер c NFS-сервером.
Дальнейшие действия выполняются от имени пользователя c повышенными привилегиями, разрешающими описанные действия. 
Установим пакет NFS-сервера "nfs-kernel-server":

```
root@nfss:/home/user# apt install nfs-kernel-server
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  keyutils libnfsidmap1 nfs-common rpcbind
Suggested packages:
  watchdog
The following NEW packages will be installed:
  keyutils libnfsidmap1 nfs-common nfs-kernel-server rpcbind
0 upgraded, 5 newly installed, 0 to remove and 35 not upgraded.
Need to get 569 kB of archives.
After this operation, 2,022 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libnfsidmap1 amd64 1:2.6.4-3ubuntu5.1 [48.3 kB]
Get:2 http://archive.ubuntu.com/ubuntu noble/main amd64 rpcbind amd64 1.2.6-7ubuntu2 [46.5 kB]
Get:3 http://archive.ubuntu.com/ubuntu noble/main amd64 keyutils amd64 1.6.3-3build1 [56.8 kB]
Get:4 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 nfs-common amd64 1:2.6.4-3ubuntu5.1 [248 kB]
Get:5 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 nfs-kernel-server amd64 1:2.6.4-3ubuntu5.1 [169 kB]
Fetched 569 kB in 1s (441 kB/s)
Selecting previously unselected package libnfsidmap1:amd64.
(Reading database ... 88584 files and directories currently installed.)
Preparing to unpack .../libnfsidmap1_1%3a2.6.4-3ubuntu5.1_amd64.deb ...
Unpacking libnfsidmap1:amd64 (1:2.6.4-3ubuntu5.1) ...
Selecting previously unselected package rpcbind.
Preparing to unpack .../rpcbind_1.2.6-7ubuntu2_amd64.deb ...
Unpacking rpcbind (1.2.6-7ubuntu2) ...
Selecting previously unselected package keyutils.
Preparing to unpack .../keyutils_1.6.3-3build1_amd64.deb ...
Unpacking keyutils (1.6.3-3build1) ...
Selecting previously unselected package nfs-common.
Preparing to unpack .../nfs-common_1%3a2.6.4-3ubuntu5.1_amd64.deb ...
Unpacking nfs-common (1:2.6.4-3ubuntu5.1) ...
Selecting previously unselected package nfs-kernel-server.
Preparing to unpack .../nfs-kernel-server_1%3a2.6.4-3ubuntu5.1_amd64.deb ...
Unpacking nfs-kernel-server (1:2.6.4-3ubuntu5.1) ...
Setting up libnfsidmap1:amd64 (1:2.6.4-3ubuntu5.1) ...
Setting up rpcbind (1.2.6-7ubuntu2) ...
Created symlink /etc/systemd/system/multi-user.target.wants/rpcbind.service → /usr/lib/systemd/system/rpcbind.service.
Created symlink /etc/systemd/system/sockets.target.wants/rpcbind.socket → /usr/lib/systemd/system/rpcbind.socket.
Setting up keyutils (1.6.3-3build1) ...
Setting up nfs-common (1:2.6.4-3ubuntu5.1) ...

Creating config file /etc/idmapd.conf with new version

Creating config file /etc/nfs.conf with new version
info: Selecting UID from range 100 to 999 ...

info: Adding system user `statd' (UID 111) ...
info: Adding new user `statd' (UID 111) with group `nogroup' ...
info: Not creating home directory `/var/lib/nfs'.
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-client.target → /usr/lib/systemd/system/nfs-client.target.
Created symlink /etc/systemd/system/remote-fs.target.wants/nfs-client.target → /usr/lib/systemd/system/nfs-client.target.
auth-rpcgss-module.service is a disabled or a static unit, not starting it.
nfs-idmapd.service is a disabled or a static unit, not starting it.
nfs-utils.service is a disabled or a static unit, not starting it.
proc-fs-nfsd.mount is a disabled or a static unit, not starting it.
rpc-gssd.service is a disabled or a static unit, not starting it.
rpc-statd-notify.service is a disabled or a static unit, not starting it.
rpc-statd.service is a disabled or a static unit, not starting it.
rpc-svcgssd.service is a disabled or a static unit, not starting it.
Setting up nfs-kernel-server (1:2.6.4-3ubuntu5.1) ...
Created symlink /etc/systemd/system/nfs-mountd.service.requires/fsidd.service → /usr/lib/systemd/system/fsidd.service.
Created symlink /etc/systemd/system/nfs-server.service.requires/fsidd.service → /usr/lib/systemd/system/fsidd.service.
Created symlink /etc/systemd/system/nfs-client.target.wants/nfs-blkmap.service → /usr/lib/systemd/system/nfs-blkmap.service.
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
nfs-mountd.service is a disabled or a static unit, not starting it.
nfsdcld.service is a disabled or a static unit, not starting it.

Creating config file /etc/exports with new version

Creating config file /etc/default/nfs-kernel-server with new version
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

Настройки сервера находятся в файле /etc/nfs.conf.  
Проверяем наличие слушающих портов 2049/udp, 2049/tcp,111/udp, 
111/tcp (не все они будут использоваться далее,  но их наличие 
сигнализирует о том, что необходимые сервисы готовы принимать 
внешние подключения): 

```
root@nfss:/home/user# ss -tnplu | grep -E ":111|:2049"
udp   UNCONN 0      0                               0.0.0.0:111        0.0.0.0:*    users:(("rpcbind",pid=1699,fd=5),("systemd",pid=1,fd=40))
udp   UNCONN 0      0                                  [::]:111           [::]:*    users:(("rpcbind",pid=1699,fd=7),("systemd",pid=1,fd=42))
tcp   LISTEN 0      4096                            0.0.0.0:111        0.0.0.0:*    users:(("rpcbind",pid=1699,fd=4),("systemd",pid=1,fd=39))
tcp   LISTEN 0      64                              0.0.0.0:2049       0.0.0.0:*                                             
tcp   LISTEN 0      4096                               [::]:111           [::]:*    users:(("rpcbind",pid=1699,fd=6),("systemd",pid=1,fd=41))
tcp   LISTEN 0      64                                 [::]:2049          [::]:*                                         
```

Создаём и настраиваем директорию, которая в 
будущем  станет доступна по сети (доступна для экспорта):

```
root@nfss:/home/user# mkdir -p /srv/share/upload
root@nfss:/home/user# chown -R nobody:nogroup /srv/share
root@nfss:/home/user# chmod 0777 /srv/share/upload
root@nfss:/home/user# ls -la /srv/share/
total 12
drwxr-xr-x 3 nobody nogroup 4096 May 29 19:27 .
drwxr-xr-x 3 root   root    4096 May 29 19:27 ..
drwxrwxrwx 2 nobody nogroup 4096 May 29 19:27 upload
root@nfss:/home/user# ls -la /srv/
total 12
drwxr-xr-x  3 root   root    4096 May 29 19:27 .
drwxr-xr-x 28 root   root    4096 May  3 10:44 ..
drwxr-xr-x  3 nobody nogroup 4096 May 29 19:27 share
```

Cоздаём в файле /etc/exports структуру, которая позволит экспортировать ранее созданную директорию:

```
root@nfss:/etc# echo -n "/srv/share 192.168.1.74/32(rw,sync,root_squash)" >> /etc/exports
root@nfss:/etc# cat /etc/exports
# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/srv/share 192.168.1.74/32(rw,sync,root_squash)
```

Экспортируем ранее созданную директорию (перезапустим процесс синхронизации таблицы экспортированных директорий):

```
root@nfss:/etc# exportfs -r
exportfs: /etc/exports [1]: Neither 'subtree_check' or 'no_subtree_check' specified for export "192.168.1.74/32:/srv/share".
  Assuming default behaviour ('no_subtree_check').
  NOTE: this default has changed since nfs-utils version 1.0.x
```

Проверяем экспортированную директорию следующей командой (запросим текущий статус таблицы экспортированных директорий):

```
root@nfss:/etc# exportfs -s
/srv/share  192.168.1.74/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```

## Настраиваем клиент NFS

Заходим на сервер с клиентом. 
Дальнейшие действия выполняются от имени пользователя, имеющего повышенные привилегии, разрешающие описанные действия.  
Установим пакет с NFS-клиентом "nfs-common":

```
root@nfsc:/home/user# apt install nfs-common
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  keyutils libnfsidmap1 rpcbind
Suggested packages:
  watchdog
The following NEW packages will be installed:
  keyutils libnfsidmap1 nfs-common rpcbind
0 upgraded, 4 newly installed, 0 to remove and 34 not upgraded.
Need to get 400 kB of archives.
After this operation, 1,416 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 libnfsidmap1 amd64 1:2.6.4-3ubuntu5.1 [48.3 kB]
Get:2 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 rpcbind amd64 1.2.6-7ubuntu2 [46.5 kB]
Get:3 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 keyutils amd64 1.6.3-3build1 [56.8 kB]
Get:4 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 nfs-common amd64 1:2.6.4-3ubuntu5.1 [248 kB]
Fetched 400 kB in 1s (462 kB/s)
Selecting previously unselected package libnfsidmap1:amd64.
(Reading database ... 164807 files and directories currently installed.)
Preparing to unpack .../libnfsidmap1_1%3a2.6.4-3ubuntu5.1_amd64.deb ...
Unpacking libnfsidmap1:amd64 (1:2.6.4-3ubuntu5.1) ...
Selecting previously unselected package rpcbind.
Preparing to unpack .../rpcbind_1.2.6-7ubuntu2_amd64.deb ...
Unpacking rpcbind (1.2.6-7ubuntu2) ...
Selecting previously unselected package keyutils.
Preparing to unpack .../keyutils_1.6.3-3build1_amd64.deb ...
Unpacking keyutils (1.6.3-3build1) ...
Selecting previously unselected package nfs-common.
Preparing to unpack .../nfs-common_1%3a2.6.4-3ubuntu5.1_amd64.deb ...
Unpacking nfs-common (1:2.6.4-3ubuntu5.1) ...
Setting up libnfsidmap1:amd64 (1:2.6.4-3ubuntu5.1) ...
Setting up rpcbind (1.2.6-7ubuntu2) ...
Created symlink /etc/systemd/system/multi-user.target.wants/rpcbind.service → /usr/lib/systemd/system/rpcbind.service.
Created symlink /etc/systemd/system/sockets.target.wants/rpcbind.socket → /usr/lib/systemd/system/rpcbind.socket.
Setting up keyutils (1.6.3-3build1) ...
Setting up nfs-common (1:2.6.4-3ubuntu5.1) ...

Creating config file /etc/idmapd.conf with new version

Creating config file /etc/nfs.conf with new version
info: Selecting UID from range 100 to 999 ...

info: Adding system user `statd' (UID 111) ...
info: Adding new user `statd' (UID 111) with group `nogroup' ...
info: Not creating home directory `/var/lib/nfs'.
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-client.target → /usr/lib/systemd/system/nfs-client.target.
Created symlink /etc/systemd/system/remote-fs.target.wants/nfs-client.target → /usr/lib/systemd/system/nfs-client.target.
auth-rpcgss-module.service is a disabled or a static unit, not starting it.
nfs-idmapd.service is a disabled or a static unit, not starting it.
nfs-utils.service is a disabled or a static unit, not starting it.
proc-fs-nfsd.mount is a disabled or a static unit, not starting it.
rpc-gssd.service is a disabled or a static unit, not starting it.
rpc-statd-notify.service is a disabled or a static unit, not starting it.
rpc-statd.service is a disabled or a static unit, not starting it.
rpc-svcgssd.service is a disabled or a static unit, not starting it.
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

Настраиваем монтирование удаленной (сетевой) директории при старте системы: 

```
root@nfsc:/home/user# echo "192.168.1.73:/srv/share/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0">> /etc/fstab
root@nfsc:/home/user# cat /etc/fstab
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
UUID="db1fef13-f2af-48be-b499-ff2a89088de0" /home ext4 defaults 0 0
UUID="68ba2c7c-044e-49ac-a086-7a635ec30c73" /var ext4 defaults 0 0
192.168.1.73:/srv/share/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0

root@nfsc:/home/user# systemctl daemon-reload
root@nfsc:/home/user# systemctl restart remote-fs.target
```

Отметим, что в данном случае происходит автоматическая генерация  systemd units в каталоге /run/systemd/generator/, которые производят монтирование при первом обращении к каталогу /mnt/. 
Переходим в директорию /mnt/ и проверяем успешность монтирования:

```
root@nfsc:/mnt# mount | grep mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=71,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=30033)
192.168.1.73:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=524288,wsize=524288,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=192.168.1.73,mountvers=3,mountport=55538,mountproto=udp,local_lock=none,addr=192.168.1.73)
```

Проверка работоспособности:  
● Заходим на сервер; 
```
ssh user@192.168.1.73
```
● Переходим в каталог /srv/share/upload; 
```
root@nfss:/etc# cd /srv/share/upload
root@nfss:/srv/share/upload#
```
● Создаём тестовый файл "check_file"; 
```
root@nfss:/srv/share/upload# touch check_file
root@nfss:/srv/share/upload# ls
check_file
```
● Заходим на клиент; 
```
ssh user@192.168.1.74
```
● Заходим в каталог /mnt/upload;  
```
root@nfsc:/mnt# cd /mnt/upload
root@nfsc:/mnt/upload#
```
● Проверяем наличие ранее созданного файла; 
```
root@nfsc:/mnt/upload# ls
check_file
```
● Создаём тестовый файл touch client_file;  
```
root@nfsc:/mnt/upload# touch client_file
```
● Проверяем, что файл успешно создан. Так же видим, что нам доступен на чтение файл, созданный на сервере.
```
root@nfsc:/mnt/upload# ls
check_file  client_file
root@nfsc:/mnt/upload# cat check_file
Test
```
Проверки прошли успешно, следовательно проблем с правами доступа нет.

Проверяем клиент:  
1. перезагружаем клиент;
```
root@nfsc:/mnt/upload# reboot

Broadcast message from root@nfsc on pts/1 (Sun 2026-05-31 11:49:17 UTC):

The system will reboot now!

root@nfsc:/mnt/upload#
Remote side unexpectedly closed network connection
```
2. заходим на клиент;
```
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.13.2-061302-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sun May 31 11:49:54 AM UTC 2026

  System load:    0.63              Processes:              272
  Usage of /home: 10.6% of 1.90GB   Users logged in:        0
  Memory usage:   9%                IPv4 address for ens33: 192.168.1.74
  Swap usage:     0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

32 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


Last login: Sun May 31 11:25:24 2026 from 192.168.1.72
```
3. заходим в каталог /mnt/upload;

```
root@nfsc:/home/user# cd /mnt/upload/
root@nfsc:/mnt/upload#
```
4. проверяем наличие ранее созданных файлов - файлы в наличии.
```
root@nfsc:/mnt/upload# ls -la
total 12
drwxrwxrwx 2 nobody nogroup 4096 May 29 20:02 .
drwxr-xr-x 3 nobody nogroup 4096 May 29 19:27 ..
-rw-r--r-- 1 root   root       5 May 31 11:41 check_file
-rw-r--r-- 1 nobody nogroup    0 May 29 20:02 client_file
```

Проверяем сервер:  
1. заходим на сервер в отдельном окне терминала; 
```
ssh user@192.168.1.73

Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-111-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sun May 31 11:52:53 AM UTC 2026

  System load:  0.0               Processes:              438
  Usage of /:   38.4% of 9.75GB   Users logged in:        1
  Memory usage: 11%               IPv4 address for ens33: 192.168.1.73
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

35 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


Last login: Sun May 31 11:25:14 2026 from 192.168.1.72

```
2. перезагружаем сервер; 
```
user@nfss:~$ sudo reboot
[sudo] password for user:

Broadcast message from root@nfss on pts/3 (Sun 2026-05-31 11:53:43 UTC):

The system will reboot now!

user@nfss:~$ Connection to 192.168.1.73 closed by remote host.
Connection to 192.168.1.73 closed.
```
3. заходим на сервер; 
```
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-111-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sun May 31 11:54:00 AM UTC 2026

  System load:  0.6               Processes:              479
  Usage of /:   38.5% of 9.75GB   Users logged in:        0
  Memory usage: 11%               IPv4 address for ens33: 192.168.1.73
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

35 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


Last login: Sun May 31 11:52:54 2026 from 192.168.1.72
```
4. проверяем наличие ранее созданных файлов в каталоге /srv/share/upload/; 
```
root@nfss:/home/user# ls -la /srv/share/upload/
total 12
drwxrwxrwx 2 nobody nogroup 4096 May 29 20:02 .
drwxr-xr-x 3 nobody nogroup 4096 May 29 19:27 ..
-rw-r--r-- 1 root   root       5 May 31 11:41 check_file
-rw-r--r-- 1 nobody nogroup    0 May 29 20:02 client_file
```
5. проверяем таблицу экспорта exportfs -s; 
```
root@nfss:/home/user# exportfs -s
/srv/share  192.168.1.74/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```
6. проверяем работу RPC showmount -a 192.168.50.10.
```
root@nfss:/home/user# showmount -a 192.168.1.73
All mount points on 192.168.1.73:
192.168.1.74:/srv/share
```

Проверяем клиент:  
1. возвращаемся на клиент; 
2. перезагружаем клиент; 
```
root@nfsc:/mnt/upload# reboot

Broadcast message from root@nfsc on pts/1 (Sun 2026-05-31 11:58:53 UTC):

The system will reboot now!

root@nfsc:/mnt/upload#
Remote side unexpectedly closed network connection
```
3. заходим на клиент; 
```
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.13.2-061302-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sun May 31 11:49:54 AM UTC 2026

  System load:    0.63              Processes:              272
  Usage of /home: 10.6% of 1.90GB   Users logged in:        0
  Memory usage:   9%                IPv4 address for ens33: 192.168.1.74
  Swap usage:     0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

32 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


Last login: Sun May 31 11:49:55 2026 from 192.168.1.72
```
4. проверяем работу RPC showmount -a 192.168.50.10; 
```
root@nfsc:/home/user# showmount -a 192.168.1.73
All mount points on 192.168.1.73:
192.168.1.74:/srv/share
```
5. заходим в каталог /mnt/upload; 
```
root@nfsc:/home/user# cd /mnt/upload
root@nfsc:/mnt/upload#
```
6. проверяем статус монтирования mount | grep mnt; 
```
root@nfsc:/mnt/upload# mount | grep mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=71,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=15848)
192.168.1.73:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=524288,wsize=524288,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=192.168.1.73,mountvers=3,mountport=44669,mountproto=udp,local_lock=none,addr=192.168.1.73)
```
7. проверяем наличие ранее созданных файлов - файлы в наличии; 
```
root@nfsc:/mnt/upload# ls -la
total 12
drwxrwxrwx 2 nobody nogroup 4096 May 29 20:02 .
drwxr-xr-x 3 nobody nogroup 4096 May 29 19:27 ..
-rw-r--r-- 1 root   root       5 May 31 11:41 check_file
-rw-r--r-- 1 nobody nogroup    0 May 29 20:02 client_file
```
8. создаём финальный тестовый файл touch final_check; 
```
root@nfsc:/mnt/upload# touch final_check
root@nfsc:/mnt/upload#
```
9. проверяем, что файл успешно создан.
```
root@nfsc:/mnt/upload# ls -la
total 12
drwxrwxrwx 2 nobody nogroup 4096 May 31 12:02 .
drwxr-xr-x 3 nobody nogroup 4096 May 29 19:27 ..
-rw-r--r-- 1 root   root       5 May 31 11:41 check_file
-rw-r--r-- 1 nobody nogroup    0 May 29 20:02 client_file
-rw-r--r-- 1 nobody nogroup    0 May 31 12:02 final_check
```

Вышеописанные проверки прошли успешно, следовательно демонстрационный стенд работоспособен и готов к работе.
