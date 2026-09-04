# Занятие 33. Статическая и динамическая маршрутизация, OSPF
**Цель домашнего задания** \
Создать домашнюю сетевую лабораторию. Научится настраивать протокол OSPF в 
Linux-based системах. 

**Описание домашнего задания** 
1. Развернуть 3 виртуальные машины. 
2. Объединить их разными vlan.
3. Настроить OSPF между машинами на базе Quagga.
4. Изобразить ассиметричный роутинг.
5. Сделать один из линков "дорогим", но что бы при этом роутинг был симметричным.

Стенд подготовлен на базе VM Debian12, развернутых в EVE-NG.

## Подготовка виртуальных машин

Создадим 3 виртуальные машины и 3 хоста, и объединим их в следующую топологию.

<img width="484" height="658" alt="frr_topology_v2" src="https://github.com/user-attachments/assets/adbd99ab-8201-43d8-b7c7-709678e7eea7" />

```
 Static hostname: router1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: aa0b0e1dddcd46638047c9ebe3efcb70
         Boot ID: b89bd001586344039a908d35ea70430f
  Virtualization: kvm
Operating System: Debian GNU/Linux 12 (bookworm)
          Kernel: Linux 6.1.0-44-amd64
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: rel-1.11.1-0-g0551a4be2c-prebuilt.qemu-project.org
```

Предварительно на каждой VM
 - устанавливаем frr; 
 - включаем пересылку пакетов протокола ipv4;
 - включаем протокл OSPF в FRR; 
 - перезапустим службу и проверим ее статус.

```
$ curl https://deb.frrouting.org/frr/keys.asc > frr_key.asc
$ gpg --import frr_key.asc
$ mv frr_key.asc /etc/apt/trusted.gpg.d
$ echo deb https://deb.frrouting.org/frr $(lsb_release -s -c) frr-stable > /etc/apt/sources.list.d/frr.list
$ apt update
$ apt install frr frr-pythontools
$ sysctl net.ipv4.conf.all.forwarding=1
$ systemctl restart frr

root@router1:~# cat /etc/frr/daemons | grep ospfd
ospfd=yes
root@router2:~# cat /etc/frr/daemons | grep ospfd
ospfd=yes
root@router3:~# cat /etc/frr/daemons | grep ospfd
ospfd=yes

root@router1:~# systemctl status frr
● frr.service - FRRouting
     Loaded: loaded (/lib/systemd/system/frr.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-08-20 22:10:53 MSK; 8s ago
       Docs: https://frrouting.readthedocs.io/en/latest/setup.html
    Process: 785 ExecStart=/usr/lib/frr/frrinit.sh start (code=exited, status=0/SUCCESS)
   Main PID: 796 (watchfrr)
     Status: "FRR Operational"
      Tasks: 10 (limit: 4635)
     Memory: 21.9M
        CPU: 255ms
     CGroup: /system.slice/frr.service
             ├─796 /usr/lib/frr/watchfrr -d mgmtd zebra ospfd staticd
             ├─807 /usr/lib/frr/mgmtd -d -F traditional -A 127.0.0.1
             ├─809 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 90000000
             ├─814 /usr/lib/frr/ospfd -d -F traditional -A 127.0.0.1
             └─817 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1

Aug 20 22:10:53 router1 frrinit.sh[841]: [841|watchfrr] sending configuration
Aug 20 22:10:53 router1 watchfrr[796]: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
Aug 20 22:10:53 router1 frrinit.sh[841]: [841|watchfrr] done
Aug 20 22:10:53 router1 watchfrr[796]: [QDG3Y-BY5TN] mgmtd state -> up : connect succeeded
Aug 20 22:10:53 router1 frrinit.sh[785]: Started watchfrr.
Aug 20 22:10:53 router1 watchfrr[796]: [QDG3Y-BY5TN] zebra state -> up : connect succeeded
Aug 20 22:10:53 router1 watchfrr[796]: [QDG3Y-BY5TN] ospfd state -> up : connect succeeded
Aug 20 22:10:53 router1 watchfrr[796]: [QDG3Y-BY5TN] staticd state -> up : connect succeeded
Aug 20 22:10:53 router1 watchfrr[796]: [KWE5Q-QNGFC] all daemons up, doing startup-complete notify
Aug 20 22:10:53 router1 systemd[1]: Started frr.service - FRRouting.

root@router2:~# systemctl status frr
● frr.service - FRRouting
     Loaded: loaded (/lib/systemd/system/frr.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-08-20 22:10:53 MSK; 8s ago
       Docs: https://frrouting.readthedocs.io/en/latest/setup.html
    Process: 764 ExecStart=/usr/lib/frr/frrinit.sh start (code=exited, status=0/SUCCESS)
   Main PID: 774 (watchfrr)
     Status: "FRR Operational"
      Tasks: 10 (limit: 4635)
     Memory: 21.8M
        CPU: 224ms
     CGroup: /system.slice/frr.service
             ├─774 /usr/lib/frr/watchfrr -d mgmtd zebra ospfd staticd
             ├─785 /usr/lib/frr/mgmtd -d -F traditional -A 127.0.0.1
             ├─787 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 90000000
             ├─792 /usr/lib/frr/ospfd -d -F traditional -A 127.0.0.1
             └─795 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1

Aug 20 22:10:53 router2 frrinit.sh[819]: [819|watchfrr] done
Aug 20 22:10:53 router2 zebra[787]: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
Aug 20 22:10:53 router2 frrinit.sh[799]: [799|zebra] done
Aug 20 22:10:53 router2 watchfrr[774]: [QDG3Y-BY5TN] mgmtd state -> up : connect succeeded
Aug 20 22:10:53 router2 watchfrr[774]: [QDG3Y-BY5TN] zebra state -> up : connect succeeded
Aug 20 22:10:53 router2 watchfrr[774]: [QDG3Y-BY5TN] ospfd state -> up : connect succeeded
Aug 20 22:10:53 router2 watchfrr[774]: [QDG3Y-BY5TN] staticd state -> up : connect succeeded
Aug 20 22:10:53 router2 frrinit.sh[764]: Started watchfrr.
Aug 20 22:10:53 router2 watchfrr[774]: [KWE5Q-QNGFC] all daemons up, doing startup-complete notify
Aug 20 22:10:53 router2 systemd[1]: Started frr.service - FRRouting.

root@router3:~# systemctl status frr
● frr.service - FRRouting
     Loaded: loaded (/lib/systemd/system/frr.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-08-20 22:10:53 MSK; 8s ago
       Docs: https://frrouting.readthedocs.io/en/latest/setup.html
    Process: 706 ExecStart=/usr/lib/frr/frrinit.sh start (code=exited, status=0/SUCCESS)
   Main PID: 716 (watchfrr)
     Status: "FRR Operational"
      Tasks: 10 (limit: 4635)
     Memory: 21.8M
        CPU: 243ms
     CGroup: /system.slice/frr.service
             ├─716 /usr/lib/frr/watchfrr -d mgmtd zebra ospfd staticd
             ├─727 /usr/lib/frr/mgmtd -d -F traditional -A 127.0.0.1
             ├─729 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 90000000
             ├─734 /usr/lib/frr/ospfd -d -F traditional -A 127.0.0.1
             └─737 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1

Aug 20 22:10:53 router3 zebra[729]: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
Aug 20 22:10:53 router3 frrinit.sh[741]: [741|zebra] done
Aug 20 22:10:53 router3 frrinit.sh[756]: [756|staticd] done
Aug 20 22:10:53 router3 watchfrr[716]: [QDG3Y-BY5TN] mgmtd state -> up : connect succeeded
Aug 20 22:10:53 router3 watchfrr[716]: [QDG3Y-BY5TN] zebra state -> up : connect succeeded
Aug 20 22:10:53 router3 watchfrr[716]: [QDG3Y-BY5TN] ospfd state -> up : connect succeeded
Aug 20 22:10:53 router3 watchfrr[716]: [QDG3Y-BY5TN] staticd state -> up : connect succeeded
Aug 20 22:10:53 router3 watchfrr[716]: [KWE5Q-QNGFC] all daemons up, doing startup-complete notify
Aug 20 22:10:53 router3 frrinit.sh[706]: Started watchfrr.
Aug 20 22:10:53 router3 systemd[1]: Started frr.service - FRRouting.
```

Настроим IP-адреса на интерфейсах маршрутизаторов в соответствии со схемой.

```
router1# show running-config

interface ens4
 description =router2_ens4=
 ip address 10.0.10.1/30
exit
!
interface ens5
 description =router3_ens5=
 ip address 10.0.11.1/30
exit
!
interface ens6
 description =net1=
 ip address 192.168.10.254/24
exit
!

router1# show int bri
Interface       Status  VRF             Addresses
---------       ------  ---             ---------
ens3            up      default         172.17.135.221/24
                                        fe80::521d:8ff:fe07:0/64
ens4            up      default         10.0.10.1/30
                                        fe80::521d:8ff:fe07:1/64
ens5            up      default         10.0.11.1/30
                                        fe80::521d:8ff:fe07:2/64
ens6            up      default         192.168.10.254/24
                                        fe80::521d:8ff:fe07:3/64
lo              up      default

interface ens4
 description =router1_ens4=
 ip address 10.0.10.2/30
exit
!
interface ens5
 description =router3_ens5=
 ip address 10.0.12.2/30
exit
!
interface ens6
 description =net2=
 ip address 192.168.20.254/24
exit
!

router2# show int bri
Interface       Status  VRF             Addresses
---------       ------  ---             ---------
ens3            up      default         172.17.135.226/24
                                        fe80::521d:8ff:fe01:0/64
ens4            up      default         10.0.10.2/30
                                        fe80::521d:8ff:fe01:1/64
ens5            up      default         10.0.12.2/30
                                        fe80::521d:8ff:fe01:2/64
ens6            up      default         192.168.20.254/24
                                        fe80::521d:8ff:fe01:3/64
lo              up      default

interface ens4
 description =router2_ens5=
 ip address 10.0.12.1/30
exit
!
interface ens5
 description =router1_ens5=
 ip address 10.0.11.2/30
exit
!
interface ens6
 description =net3=
 ip address 192.168.30.254/24
exit

router3# show int bri
Interface       Status  VRF             Addresses
---------       ------  ---             ---------
ens3            up      default         172.17.135.127/24
                                        fe80::521d:8ff:fe02:0/64
ens4            up      default         10.0.12.1/30
                                        fe80::521d:8ff:fe02:1/64
ens5            up      default         10.0.11.2/30
                                        fe80::521d:8ff:fe02:2/64
ens6            up      default         192.168.30.254/24
                                        fe80::521d:8ff:fe02:3/64
lo              up      default
```

1. Запустим процесс OSPF на каждом маршрутизаторе и настроим интерфейсы ens4-6 в качестве участников этого процесса.
Магистральные интерфейсы поместим в backbone area 0, а интерфейсы в подсетях хостов в standart area 1.

```
router1# show running-config ospfd
Building configuration...

Current configuration:
!
frr version 10.7.0
frr defaults traditional
hostname router1
service integrated-vtysh-config
!
interface ens4
 description =router2_ens4=
 ip ospf area 0
 ip ospf network point-to-point
exit
!
interface ens5
 description =router3_ens5=
 ip ospf area 0
 ip ospf network point-to-point
exit
!
interface ens6
 description =net1=
 ip ospf area 1
 ip ospf passive
exit
!
router ospf
 ospf router-id 1.1.1.1
exit
!

router2# show run ospfd
Building configuration...

Current configuration:
!
frr version 10.7.0
frr defaults traditional
hostname router2
service integrated-vtysh-config
!
interface ens4
 description =router1_ens4=
 ip ospf area 0
 ip ospf network point-to-point
exit
!
interface ens5
 description =router3_ens5=
 ip ospf area 0
 ip ospf network point-to-point
exit
!
interface ens6
 description =net2=
 ip ospf area 1
 ip ospf passive
exit
!
router ospf
 ospf router-id 2.2.2.2
exit
!
end

router3# show run ospfd
Building configuration...

Current configuration:
!
frr version 10.7.0
frr defaults traditional
hostname router3
service integrated-vtysh-config
!
interface ens4
 description =router2_ens5=
 ip ospf area 0
 ip ospf network point-to-point
exit
!
interface ens5
 description =router1_ens5=
 ip ospf area 0
 ip ospf network point-to-point
exit
!
interface ens6
 description =net3=
 ip ospf area 1
 ip ospf passive
exit
!
router ospf
 ospf router-id 3.3.3.3
exit
!
end
```

Проверим отношения соседства и таблицу маршрутизации.
```
router1# show ip ospf nei

Neighbor ID     Pri State           Up Time         Dead Time Address         Interface                        RXmtL RqstL DBsmL
2.2.2.2           1 Full/-          12m02s            38.111s 10.0.10.2       ens4:10.0.10.1                       0     0     0
3.3.3.3           1 Full/-          1m20s             30.632s 10.0.11.2       ens5:10.0.11.1                       0     0     0

router1# show ip route ospf
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

IPv4 unicast VRF default:
O>* 10.0.12.0/30 [110/20] via 10.0.10.2, ens4, weight 1, 00:01:38
  *                       via 10.0.11.2, ens5, weight 1, 00:01:38
O>* 192.168.20.0/24 [110/20] via 10.0.10.2, ens4, weight 1, 00:06:58
O>* 192.168.30.0/24 [110/20] via 10.0.11.2, ens5, weight 1, 00:01:38

router2# show ip ospf nei

Neighbor ID     Pri State           Up Time         Dead Time Address         Interface                        RXmtL RqstL DBsmL
1.1.1.1           1 Full/-          12m06s            33.148s 10.0.10.1       ens4:10.0.10.2                       0     0     0
3.3.3.3           1 Full/-          1m33s             36.580s 10.0.12.1       ens5:10.0.12.2                       0     0     0

router2# show ip route ospf
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

IPv4 unicast VRF default:
O>* 10.0.11.0/30 [110/20] via 10.0.10.1, ens4, weight 1, 00:02:22
  *                       via 10.0.12.1, ens5, weight 1, 00:02:22
O>* 192.168.10.0/24 [110/20] via 10.0.10.1, ens4, weight 1, 00:04:51
O>* 192.168.30.0/24 [110/20] via 10.0.12.1, ens5, weight 1, 00:02:10

router3# show ip ospf nei

Neighbor ID     Pri State           Up Time         Dead Time Address         Interface                        RXmtL RqstL DBsmL
2.2.2.2           1 Full/-          1m17s             32.795s 10.0.12.2       ens4:10.0.12.1                       0     0     0
1.1.1.1           1 Full/-          1m08s             31.742s 10.0.11.1       ens5:10.0.11.2                       0     0     0

router3# show ip route ospf
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

IPv4 unicast VRF default:
O>* 10.0.10.0/30 [110/20] via 10.0.11.1, ens5, weight 1, 00:02:15
  *                       via 10.0.12.2, ens4, weight 1, 00:02:15
O>* 192.168.10.0/24 [110/20] via 10.0.11.1, ens5, weight 1, 00:02:23
O>* 192.168.20.0/24 [110/20] via 10.0.12.2, ens4, weight 1, 00:02:27
```

Из представленной выше информации видим, что у каждого маршрутизатора установлены отношения смежности с двумя соседними маршрутизаторами, а в таблице маршрутизации присутствуют маршруты к подсетям хостов. Маршрутизация симметричная - так трафик от хоста 1 до хоста 2 в прямую и обратную сторону будет направлен через хост 2.

<img width="600" height="352" alt="ping and trace to 192 168 20 1" src="https://github.com/user-attachments/assets/d0953a15-80ed-47d4-82ce-8691e8c38678" />

<img width="600" height="235" alt="ping and trace to 192 168 10 1" src="https://github.com/user-attachments/assets/a3808c87-62d0-4beb-8d0e-1fd47c35177b" />


2. Создадим ситуацию **ассиметричной маршрутизации**, когда обратный трафик от хоста 2 к хосту 1 пойдет через router 3. Для этого увелчим стоимость линка ens4 на router2.
Также включим ассиметричнцю маршрутизацию.

```
root@router1:~# sysctl net.ipv4.conf.all.rp_filter=1
net.ipv4.conf.all.rp_filter = 1
root@router1:~# sysctl -p
net.ipv4.ip_forward = 1

router2# show run ospfd
Building configuration...

Current configuration:
!
frr version 10.7.0
frr defaults traditional
hostname router2
service integrated-vtysh-config
!
interface ens4
 description =router1_ens4=
 ip ospf area 0
 ip ospf cost 100
 ip ospf network point-to-point
exit
!
interface ens5
 description =router3_ens5=
 ip ospf area 0
 ip ospf network point-to-point
exit
!
interface ens6
 description =net2=
 ip ospf area 1
 ip ospf passive
exit
!
router ospf
 ospf router-id 2.2.2.2
exit
!
end
router2# show ip route ospf
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

IPv4 unicast VRF default:
O   10.0.10.0/30 [110/30] via 10.0.12.1, ens5, weight 1, 00:00:28
O>* 10.0.11.0/30 [110/20] via 10.0.12.1, ens5, weight 1, 00:00:28
O>* 192.168.10.0/24 [110/30] via 10.0.12.1, ens5, weight 1, 00:00:28
O>* 192.168.30.0/24 [110/20] via 10.0.12.1, ens5, weight 1, 00:04:51

Криншот "ping and trace to 192.168.10.1 via router3" 
```
3. Для восстановления симметричной маршрутизации на router1 для интерфейса ens4 также увеличим стоимость.
Результат проверим с помощью трассировки с хоста 1 до хоста 2 и хоста 2 к хосту 1.
```
router1# show run
Building configuration...

Current configuration:
!
frr version 10.7.0
frr defaults traditional
hostname router1
log syslog informational
service integrated-vtysh-config
!
interface ens4
 description =router2_ens4=
 ip address 10.0.10.1/30
 ip ospf area 0
 ip ospf cost 100
 ip ospf network point-to-point
exit
!
interface ens5
 description =router3_ens5=
 ip address 10.0.11.1/30
 ip ospf area 0
 ip ospf network point-to-point
exit
!
interface ens6
 description =net1=
 ip address 192.168.10.254/24
 ip ospf area 1
 ip ospf passive
exit
!
router ospf
 ospf router-id 1.1.1.1
exit
!
end
```

<img width="600" height="138" alt="trace to 192 168 20 1 simmetric" src="https://github.com/user-attachments/assets/22a8295e-b1c0-47a5-ba97-e66a4b6efce9" />

<img width="584" height="117" alt="trace to 192 168 10 1 simmetric" src="https://github.com/user-attachments/assets/982273fd-e2ef-46a5-a555-27b03de10234" />


Видим, что прямой и обратный трафик следует через маршрутизатор 3.

Выводы:
- В ходе выполнения домашнего задания с помощью FRR на виртуальных машинах под управлением Debian12 была настроена сеть с динамической маршрутизацией по протоколу OSPF. 
- Продемонстрирован механизм управления процессом выбора кратчайшего пути посредсвом изменения стоимости интерфейсов составляющих этот путь.
