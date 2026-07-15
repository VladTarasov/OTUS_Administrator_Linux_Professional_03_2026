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
3. Создаём пользователей otusadm и otus: sudo 
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
6. Добавляем пользователя otusadm в группу admin и проверям, что у пользователя появилась группа:
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
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-134-generic aarch64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Wed Jul 15 07:11:03 PM UTC 2026

  System load:  0.0               Processes:               97
  Usage of /:   52.7% of 9.75GB   Users logged in:         0
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


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Could not chdir to home directory /home/otus: No such file or directory

$ whoami
otus


% ssh otusadm@192.168.1.77
otusadm@192.168.1.77's password: 
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-134-generic aarch64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Wed Jul 15 07:13:42 PM UTC 2026

  System load:  0.01              Processes:               97
  Usage of /:   52.7% of 9.75GB   Users logged in:         0
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


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Could not chdir to home directory /home/otusadm: No such file or directory

$ whoami
otusadm
```

