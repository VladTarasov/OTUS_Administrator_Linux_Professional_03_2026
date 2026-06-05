# Занятие 8. Загрузка системы

## Задание
- Включить отображение меню Grub.
- Попасть в систему без пароля несколькими способами.
- Установить систему с LVM, после чего переименовать VG.

## Включить отображение меню Grub

Отредактируем конфигурационный файл загрузкчика /etc/default/grub комментируем строку, скрывающую меню и ставим задержку для выбора пункта меню в 10 секунд.
```
GRUB_DEFAULT=0
#GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=10
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT=""
GRUB_CMDLINE_LINUX=""
```
Обновляем конфигурацию загрузчика и перезагружаемся для проверки.
```
root@grub:/home/grub# nano /etc/default/grub
root@grub:/home/grub# update-grub
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.15.0-181-generic
Found initrd image: /boot/initrd.img-5.15.0-181-generic
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
done
root@grub:/home/grub# reboot
root@grub:/home/grub#
Remote side unexpectedly closed network connection
```

При загрузке в окне виртуальной машины видим меню загрузчика:
<img width="637" height="541" alt="image" src="https://github.com/user-attachments/assets/1fb7ff76-1d96-46b1-954e-1898ef2ccb08" />

## Попасть в систему без пароля несколькими способами

При загрузке виртуальной машины в окне выбора ядра для загрузки нажмем e - в данном контексте edit. Попадаем в окно, где мы можем изменить параметры загрузки:

<img width="641" height="536" alt="image" src="https://github.com/user-attachments/assets/39e712df-234b-4a2a-ae4b-f52873174d84" />

**Способ 1. init=/bin/bash** 

В конце строки, начинающейся с linux, добавляем 'init=/bin/bash' и нажимаем сtrl-x для 
загрузки в системы 

<img width="647" height="549" alt="image" src="https://github.com/user-attachments/assets/88bb0da7-77bc-4cf8-8663-2000c6817032" />

В целом на этом все, Мы попали в систему. 

<img width="806" height="661" alt="image" src="https://github.com/user-attachments/assets/e315f2a6-0dc3-4b69-b18d-9199ac696baa" />

Но есть один нюанс. Рутовая файловая система при этом монтируется в режиме Read-Only. Перемонтироуем
ее в режим Read-Write: 


<img width="802" height="652" alt="image" src="https://github.com/user-attachments/assets/ac4f851b-cf5f-454c-9c43-c6c1b4293d82" />

После чего можно убедимся, записав данные в любой файл или прочитав вывод 
команды 'mount':

<img width="505" height="60" alt="image" src="https://github.com/user-attachments/assets/b197a324-c170-4fec-a35d-889c90181333" />

**Способ 2. Recovery mode**

В меню загрузчика на первом уровне выбрать второй пункт (Advanced options…), 

<img width="636" height="537" alt="image" src="https://github.com/user-attachments/assets/e92bec03-eb25-4d31-84f8-1d1d455a5931" />

далее загрузить пункт меню с указанием "recovery mod" в названии. Попадаем вьщгте меню режима восстановления. 

<img width="644" height="537" alt="image" src="https://github.com/user-attachments/assets/7c23528f-b2bd-4ed7-9f25-6a7724c1af85" />

<img width="707" height="354" alt="image" src="https://github.com/user-attachments/assets/e8233106-8ef5-4b67-bcda-21da2d84f2a6" />

В этом меню сначала включаем поддержку сети (network) для того, чтобы файловая система перемонтировалась в режим read/write (либо это можно сделать вручную). 

<img width="720" height="393" alt="image" src="https://github.com/user-attachments/assets/3c1c0aef-b977-4c37-9e9d-eef328d776c3" />

Далее выбираем пункт root и попадаем в консоль с пользователем root. Если вы ранее устанавливали пароль для пользователя root (по умолчанию его нет), то необходимо его ввести. В этой консоли можно производить любые манипуляции с системой. 

<img width="722" height="436" alt="image" src="https://github.com/user-attachments/assets/88323188-89a2-417e-9ddc-23f347df4fa7" />




