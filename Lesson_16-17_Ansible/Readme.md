# Занятия 16-17. Автоматизация администрированя. Ansible

## Домашнее задание
Первые шаги с Ansible

## Цель:
написать первые шаги с Ansible;


## Описание/Пошаговая инструкция выполнения домашнего задания:

🎯 Что нужно сделать?

**Основное задание**
Подготовить стенд на Vagrant как минимум с одним сервером. На этом сервере, используя Ansible, необходимо развернуть nginx со следующими условиями:
- необходимо использовать модуль yum/apt;
- конфигурационные файлы должны быть взяты из шаблона jinja2 с переменными;
- после установки nginx должен быть в режиме enabled в systemd;
- должен быть использован notify для старта nginx после установки;
- сайт должен слушать на нестандартном порту — 8080, для этого использовать переменные в Ansible.

**Дополнительное задание**
- Сделать все это с использованием Ansible роли.

### **Основное задание**
Подготовка стенда
- Развернуты 2 виртуальные машины Debian 12 в среде EVE-NG (ansible и nginx), объединенных в локальную сеть 172.17.135.0/24;
- На машине nginx создан пользователь ansible c возможностью выполнения команды sudo;
- На машине аnsible создан пользователь user c возможностью выполнения команды sudo, установлены необходимые для работы пакеты ansible;
- Настроено подключение по ssh-ключу с машины аnsible на машину nginx.

Схема стенда приведена на скриншоте ниже.

<img width="490" height="229" alt="Схема стенда" src="https://github.com/user-attachments/assets/ae3ab91d-0621-441a-aed2-e3b830d456f7" />

Создадим inventory 
```
% cat staging/hosts ~ ansible

[web]
nginx ansible_host=172.17.135.23
```
И конфигурационный файл, где зададим удаленного пользователя, его привилегии и путь к нашему файлу inventory
```
% cat ansible.cfg

[defaults]
interpreter_python = auto_silent
inventory = staging/hosts
remote_user = ansible
host_key_checking = false
retry_files_enabled = false
```
Проверим работу ansible c помощью ad-hoc команд:
- выполним ping до нашего хоста;
- запросим версию ядра;
- проверим статус службы nftable. (см. на скриншотах ниже)

<img width="474" height="144" alt="ansible ping" src="https://github.com/user-attachments/assets/f1749061-2de5-4738-a105-43776b5e0093" /> .

<img width="340" height="52" alt="uname -r" src="https://github.com/user-attachments/assets/28ee2e20-09a0-4f45-97e4-f210b9506044" /> .

<img width="490" height="145" alt="nftables status" src="https://github.com/user-attachments/assets/c5b689ab-a050-4917-baed-e4d20b2c9ac3" /> .


Приступим к написанию playbook для установки nginx. Будем писать его поэтапно.

Начнем с задач по обновлению кэша пакетов и установки пакета nginx.
```
% nano nginx.yaml
---
- name: NGINX | Install and configure NGINX
  hosts: nginx
  become: true

  tasks:
    - name: update
      apt:
        update_cache=yes
    - name: NGINX | Install NGINX
      apt:
        name: nginx
        state: latest
```
Запустим playbook и проверим его работу.

<img width="1656" height="234" alt="playbook install nginx" src="https://github.com/user-attachments/assets/e3adc8b4-2593-4a97-8cd4-ff55907820ee" />

Добавим в плейбук задачу по конфигурированию nginx. Конфигурацию зададим с помощью Jinja template, а также используем переменную для указания кастомного tcp порта для сокета nginx.
Пропишем теги для задач.
Template будет выглядеть следующим образом.
```
% cat templates/nginx.conf.j2
                                                                                                                                                                  
server {
	listen {{ nginx_listen_port }} default_server;
	server_name default_server;
	root /usr/share/nginx/html;

	location / {
	}
}
```

Playbook примет вид
```
---
- name: NGINX | Install and configure NGINX
  hosts: nginx
  become: true
  vars:
    nginx_listen_port: 8080

  tasks:
    - name: update
      apt:
        update_cache=yes

    - name: NGINX | Install NGINX
      apt:
        name: nginx
        state: latest

    - name: NGINX | Create NGINX config file from template
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/default
      tags:
        - nginx-configuration
```

Добавим handlers и notify для их вызова, с их помощью будет выполняться перезапуск службы и перечитывание конфигурации. Добавим тэги к задачам. 
Playbook примет следующий и уже окончательный вид
```
% nano nginx.yaml
---
- name: NGINX | Install and configure NGINX
  hosts: nginx
  become: true
  vars:
    nginx_listen_port: 8080

  tasks:
    - name: update
      apt:
        update_cache=yes
      tags:
        - update apt

    - name: NGINX | Install NGINX
      apt:
        name: nginx
        state: latest
      notify:
        - restart nginx
      tags:
        - nginx-pakage

    - name: NGINX | Create NGINX config file from template
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/default
		owner: root
        group: root
        mode: "0644"
      notify:
        - reload nginx
      tags:
        - nginx-configuration

  handlers:
    - name: restart nginx
      systemd:
        name: nginx
        state: restarted
        enabled: yes

    - name: reload nginx
      systemd:
        name: nginx
        state: reloaded
```

Запустим playbook и убедимся в доступности nginx

<img width="600" height="251" alt="playbook final and check nginx" src="https://github.com/user-attachments/assets/72fe4f58-f215-4682-80a8-0ca336773651" />

Как видно на скриншоте playbook отработал штатно и nginx доступен на порту 8080.



