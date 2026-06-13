# Занятие 9. Systemd — создание unit-файла

## Домашнее задание
- Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default).
- Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта (https://gist.github.com/cea2k/1318020).
- Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно.

## Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default).

Создадим файл с конфигурацией для сервиса в директории /etc/default - из неё сервис будет брать необходимые переменные.
```
root@grub:/home/grub# cat > /etc/default/watchlog
# Configuration file for my watchlog service
# Place it to /etc/default

# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log
```

Создадим файл /var/log/watchlog.log и запишем туда поизвольный текст, содержащий ключевое слово ‘ALERT’ 

```
root@grub:/home/grub# cat > /var/log/watchlog.log
PING localhost (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.014 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.061 ms
64 bytes from localhost (127.0.0.1): icmp_seq=3 ttl=64 time=0.061 ms
64 bytes from localhost (127.0.0.1): icmp_seq=4 ttl=64 time=0.060 ms

ALERT
```

Создадим скрипт для поиска ключевого слова:
```
root@grub:/home/grub# cat > /opt/watchlog.sh
WORD="ALERT"
LOG="/var/log/watchlog.log"
DATE=`date`

if grep $WORD $LOG &> /dev/null;
then
        logger "$DATE: I found word, Master!";
else
exit 0
fi
```
Команда logger отправляет лог в системный журнал. # Каким образом?

Добавим права на запуск файла: 
```
root@grub:/home/grub# chmod +x /opt/watchlog.sh
```

Создадим systemd-юнит для сервиса и запустим его: 
```
root@grub:/home/grub# cat > /etc/systemd/system/watchlog.service
[Unit]
Description=My watchlog service
[Service]
Type=oneshot
EnvironmentFile=/etc/default/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG

root@grub:/home/grub# systemctl start watchlog.service
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
root@grub:/home/grub# tail -n 100 /var/log/syslog | grep word
Jun 13 10:48:45 grub root: Sat Jun 13 10:48:45 AM UTC 2026: I found word, Master!
Jun 13 10:49:27 grub root: Sat Jun 13 10:49:27 AM UTC 2026: I found word, Master!
Jun 13 10:50:03 grub root: Sat Jun 13 10:50:03 AM UTC 2026: I found word, Master!
Jun 13 10:50:50 grub root: Sat Jun 13 10:50:50 AM UTC 2026: I found word, Master!
Jun 13 10:52:03 grub root: Sat Jun 13 10:52:03 AM UTC 2026: I found word, Master!
```

## Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта
