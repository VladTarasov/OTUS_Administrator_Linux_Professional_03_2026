# Занятие 9. Systemd — создание unit-файла

## Домашнее задание
- Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default).
- Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта (https://gist.github.com/cea2k/1318020).
- Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно.

Создадим файл с конфигурацией для сервиса в директории /etc/default - из неё сервис будет брать необходимые переменные.
```
root@grub:/home/grub# touch /etc/default/watchlog
root@grub:/home/grub# nano /etc/default/watchlog
root@grub:/home/grub# cat /etc/default/watchlog
# Configuration file for my watchlog service
# Place it to /etc/default

# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log
```

Создадим файл /var/log/watchlog.log и запишем туда поизвольный текст, содержащий ключевое слово ‘ALERT’ 

```
root@grub:/home/grub# nano /var/log/watchlog.log
root@grub:/home/grub# cat /var/log/watchlog.log
PING localhost (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.014 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.061 ms
64 bytes from localhost (127.0.0.1): icmp_seq=3 ttl=64 time=0.061 ms
64 bytes from localhost (127.0.0.1): icmp_seq=4 ttl=64 time=0.060 ms

ALERT ALERT ALERT ALERT ALERT ALERT

ALERT ALERT ALERT ALERT ALERT ALERT
```

Создадим скрипт для поиска ключевого слова:
```
root@grub:/home/grub# cat > /opt/watchlog.sh
#!/bin/bash
WORD=$1
LOG=$2
DATE=`date`

if grep $WORD $LOG &> /dev/null
then
logger "$DATE: I found word, Master!"
else
exit 0
fi
```
Команда logger отправляет лог в системный журнал. # Каким образом?

Добавим права на запуск файла: 
```
root@grub:/home/grub# chmod +x /opt/watchlog.sh
```

Создадим systemd-юнит для сервиса: 
```
root@grub:/home/grub# cat > /etc/systemd/system/watchlog.service
[Unit]
Description=My watchlog service
[Service]
Type=oneshot
EnvironmentFile=/etc/default/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG
```

Создадим юнит для таймера: 

```
root@grub:/home/grub# cat > /etc/systemd/system/watchlog.timer
[Unit]
Description=Run watchlog script every 30 second
[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service
[Install]
WantedBy=multi-user.target
```

Запустим timer и убедимся в его работоспособности:
```

