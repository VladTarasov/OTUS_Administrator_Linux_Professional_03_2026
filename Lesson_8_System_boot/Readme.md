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
