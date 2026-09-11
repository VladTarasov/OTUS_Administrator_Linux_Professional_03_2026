# Занятие 24. Пользователи и группы. Авторизация и аутентификация 

**Домашнее задание**\
PAM

**Цель:**
- Научиться создавать пользователей и добавлять им ограничения.

🎯 Что нужно сделать?
- Ограничить доступ к системе для всех пользователей, кроме группы администраторов, в выходные дни (суббота и воскресенье), за исключением праздничных дней.

⭐️ **Задание повышенной сложности**

Предоставить определённому пользователю доступ к Docker и право перезапускать Docker-сервис.

Примечания:
- Работа выполняется на MacBook Air с процессором Apple Silicon M1 и гипервизором Virltual Box, виртуальная машина создается вручную, Vagrant не используется во избежание возможных проблем с совместимостью.

Настройка запрета доступа к системе для всех пользователей (кроме группы admin) в выходные дни (Праздники не учитываются):

1. Подключаемся созданной ВМ:
```
% ssh user@192.168.1.77
user@192.168.1.77's password: 
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-134-generic aarch64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Wed Jul 15 06:33:22 PM UTC 2026

  System load:  0.06              Processes:               96
  Usage of /:   52.5% of 9.75GB   Users logged in:         0
  Memory usage: 9%                IPv4 address for enp0s8: 192.168.1.77
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

29 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


Last login: Wed Jul 15 18:02:34 2026 from 192.168.1.71
```

2. Переходим в root-пользователя:
```
user@user:~$ sudo -i
[sudo] password for user: 
root@user:~# 
```  
3. Создаём пользователей otusadm и otus: 
```
root@user:~# useradd otusadm && useradd otus
root@user:~# 
```
4. Задаём для пользователей otusadm и otus пароли:
```
echo "otusadm:Otus@dm!" | chpasswd && echo "otus:Otus2026!" | chpasswd
```
5. Создаём группу admin:
```
groupadd -f admin
```
6. Добавляем пользователя otusadm в группу admin и проверяем, что у пользователя появилась группа:
```
root@user:~# usermod otusadm -a -G admin
root@user:~# exit
user@user:~$ su otusadm
Password: 
$ id
uid=1001(otusadm) gid=1001(otusadm) groups=1001(otusadm),1003(admin)
```

После создания пользователей проверим, что они могут подключаться по SSH к нашей ВМ с хостовой машины: 
```
% ssh otus@192.168.1.77
otus@192.168.1.77's password: 
...
$ whoami
otus


% ssh otusadm@192.168.1.77
otusadm@192.168.1.77's password: 
...
$ whoami
otusadm
```

7. Далее настроим правило, по которому все пользователи кроме тех, что указаны в группе admin, не смогут подключаться в выходные дни.
Выберем метод PAM-аутентификации. Так как у нас используется только ограничение по времени, то было бы логично использовать метод pam_time, однако, данный метод не работает с локальными группами пользователей, и получается, что использование данного метода добавит нам большое количество однообразных строк с разными пользователями. В текущей ситуации лучше написать небольшой скрипт контроля и использовать модуль pam_exec.

8. Создадим файл-скрипт /usr/local/bin/login.sh, ограничивающий доступ к системе с учетом праздничных, выходных дней и наличия у пользователя группы admin
```
#!/bin/bash
#Переменные
today="$(date +'%d %B')"
holidays=("01 January" "23 February" "08 March" "01 May" "09 May" "12 June" "04 November" "31 December")
match=false

#Цикл проверки праздничных дней
for item in "${holidays[@]}"; do
    if [ "$today" = "$item" ]; then
        match=true
        break
    fi
done

#Блок проверки выходных дней и наличия группы admin, а также завершения работы скрипта в случае праздничных дней
if [ "$match" = true ];
then
    exit 1
else
    if [ $(date +%a) = "Sat" ] || [ $(date +%a) = "Sun" ];
    then
        if getent group admin | grep -qw "$PAM_USER";
        then
            exit 0
        else
            exit 1
        fi
    else
        exit 0
    fi
fi
```

9. Добавим права на исполнение файла: 
```
root@user:~# chmod +x /usr/local/bin/login.sh
```

10. Укажем в файле /etc/pam.d/sshd модуль pam_exec и наш скрипт (вывод сокращен для краткости):
```
...
# Standard Un*x authentication.
@include common-auth
auth required pam_exec.so debug /usr/local/bin/login.sh
...
```

Проверим работу скрипта аутентификации при различных условиях.

Установим текущую дату в значение праздничного дня:
```
root@client:~# timedatectl set-time "2026-01-01"
root@client:~# date
Thu Jan  1 12:00:02 AM UTC 2026
root@client:~# journalctl -u ssh -f --since now
Jan 01 00:01:59 client sshd[27222]: pam_exec(sshd:auth): Calling /usr/local/bin/login.sh ...
Jan 01 00:01:59 client sshd[27220]: pam_exec(sshd:auth): /usr/local/bin/login.sh failed: exit code 1
Jan 01 00:02:00 client sshd[27220]: Failed password for otus from 192.168.31.92 port 54874 ssh2
Jan 01 00:02:03 client sshd[27220]: Connection closed by authenticating user otus 192.168.31.92 port 54874 [preauth]
Jan 01 00:02:12 client sshd[27226]: pam_exec(sshd:auth): Calling /usr/local/bin/login.sh ...
Jan 01 00:02:12 client sshd[27224]: pam_exec(sshd:auth): /usr/local/bin/login.sh failed: exit code 1
Jan 01 00:02:14 client sshd[27224]: Failed password for otusadm from 192.168.31.92 port 54875 ssh2
Jan 01 00:02:17 client sshd[27224]: Connection closed by authenticating user otusadm 192.168.31.92 port 54875 [preauth]
```
Скрипт успешно отработал, администратору и обычному пользователю не удалось подключиться в праздничный день.

Установим дату в значение соответствующее выходному дню и проверим подключение:
```
root@client:~# timedatectl set-time "2026-09-05"
root@client:~# date
Sat Sep  5 12:00:02 AM UTC 2026
root@client:~# journalctl -u ssh --since now
Sep 05 00:01:09 client sshd[27448]: pam_exec(sshd:auth): Calling /usr/local/bin/login.sh ...
Sep 05 00:01:09 client sshd[27446]: pam_exec(sshd:auth): /usr/local/bin/login.sh failed: exit code 1
Sep 05 00:01:11 client sshd[27446]: Failed password for otus from 192.168.31.92 port 54951 ssh2
Sep 05 00:01:13 client sshd[27446]: Connection closed by authenticating user otus 192.168.31.92 port 54951 [preauth]
Sep 05 00:01:22 client sshd[27455]: pam_exec(sshd:auth): Calling /usr/local/bin/login.sh ...
Sep 05 00:01:22 client sshd[27453]: pam_unix(sshd:account): account otusadm has password changed in future
Sep 05 00:01:22 client sshd[27453]: Accepted password for otusadm from 192.168.31.92 port 54952 ssh2
Sep 05 00:01:22 client sshd[27453]: pam_unix(sshd:session): session opened for user otusadm(uid=1002) by otusadm(uid=0)
```
Как видим администратору подключиться удалось, а обычному пользователю нет.

Проверим возможность подключения в рабочий будний день:
```
root@client:~# timedatectl set-time "2026-09-07"
root@client:~# date
Mon Sep  7 12:00:01 AM UTC 2026
root@client:~# journalctl -u ssh --since now
Sep 07 00:00:41 client sshd[27708]: pam_exec(sshd:auth): Calling /usr/local/bin/login.sh ...
Sep 07 00:00:41 client sshd[27706]: pam_unix(sshd:account): account otus has password changed in future
Sep 07 00:00:41 client sshd[27706]: Accepted password for otus from 192.168.31.92 port 54955 ssh2
Sep 07 00:00:41 client sshd[27706]: pam_unix(sshd:session): session opened for user otus(uid=1003) by otus(uid=0)
Sep 07 00:00:57 client sshd[27812]: pam_exec(sshd:auth): Calling /usr/local/bin/login.sh ...
Sep 07 00:00:57 client sshd[27803]: pam_unix(sshd:account): account otusadm has password changed in future
Sep 07 00:00:57 client sshd[27803]: Accepted password for otusadm from 192.168.31.92 port 54956 ssh2
Sep 07 00:00:57 client sshd[27803]: pam_unix(sshd:session): session opened for user otusadm(uid=1002) by otusadm(uid=0)
```
Администратор и пользователь успешно подключились к системе.

