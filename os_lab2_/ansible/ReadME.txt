Кхм Кхм
inventory:
  Файл содержащий хосты и команды для хостов
nginx.yml:
  Плейбук для установки NGINX (Если уже установлен epel, то используйте комманду "ansible-playbook nginx.yml --tag packages"
nginx.conf.j2
Файл конфигурации
epel.yml
  Плейбук для установки epel ("ansible-playbook epel.yml")
ansible.cfg:
  Файл конфигурации
Vagrant
  Виртуальная машина ("vagrant up")



1# Процесс установки плейбука:
Ansible требует для своей работы Python 2.6 или выше. Если необходимо, установите Ansible (yum install или apt install) и убедитесь что он корректно работает. 
2# Подготовка:
Для начала работы нужно создать каталог ansible и поднять Vagrantfile 
Для подключения к хосту nginx нам необходимо передать множество параметров. 
Мы можем узнать эти параметры с помощью vagrant ssh-config. 
3# Файл inventory:
В папке ansible создается файл inventory. 
Сам inventory должен выглядеть примерно так:
----------------------
[webservers]
nginx ansible_host=127.0.0.1 ansible_port=2222 ansible_private_key_file=/home/artem/os_lab/.vagrant/machines/nginx/virtualbox/private_key
----------------------
В этом файле стоит изменить параметры ansible_port и ansible_privat_key_file. Данную информацию можно посмотреть с помощью комманды ssh-config. 
4# Файл конфигурации ansible.cfg:
Что бы в дальнейшем не использовать полный путь до файла inventory заппишем в файл следующие строки:
----------------------
inventory = inventory
remote_user= vagrant
host_key_checking = False
transport=smart
----------------------
Ныне же из файла inventory можно убрать "home/artem/os_lab/".
Проверяем работу хоста с помощью комманды "ansible -m ping nginx"
#5 Написание плейбука 
 Напишем простой Playbook который будет выполнять установку пакета epel-release. Создаем файл epel.yml со следующим содержимым:
----------------------
- name: Install EPEL Repo
  hosts: webservers
  become: true
  tasks:
    - name: Install EPEL Repo package from standard repo
      yum:
        name: epel-release
        state: present
----------------------

После чего запустите выполнение Playbook командой: ansible-playbook epel.yml 
Если возникли ошибки, значит в синтаксисе сделано что-то неправильно.
#6 Шаблон для файла конфигурации:
Далее создадим файл шаблона для конфига NGINX, имя файла nginx.conf.j2
----------------------
events {
 worker_connections 1024;
}
http {
 server {
   listen {{ nginx_listen_port }} default_server;
   server_name default_server;
   root /usr/share/nginx/html;
   location / {
   }
 }
}
---------------------
#7 Полноценный плейбук
В конце должен получиться файл nginx.yml:
---------------------
- name: Install nginx package from epel repo
  hosts: webservers
  become: true
  vars:
    nginx_listen_port: 8080
  tasks:
    - name: Install EPEL Repo package from standard repo
      yum:
        name: epel-release
        state: present
    - name: install nginx from repo
      yum:
        name: nginx
        state: latest
      notify:
        - restart ngihx
      tags:
         nginx-package
         packages
    - name: Create config file from template
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify:
        - restart nginx  
      tags:
        nginx-configuration
  handlers:
    - name: restart nginx
      systemd:
        name: nginx
        state: restarted
        enabled: yes 
---------------------
Теперь проверяем на работу Nginx c помощью команды: vagrant ssh -c "ip addr show" после выводв этой команды, нам нужно найти там публичный ip-адресс

Затем в браузере откройте страницу http://192.168.11.150:8080
192.168.11.150 - ваш публичный адрес

Если страница открылась - значит всё работает, если нет, то не всё работает ищите ошибки в исключениях
Держи кота
         ／＞　   フ
　　　　　| 　_　 _|
　 　　　／`ミ _x 彡
　　 　 /　　　 　 |
　　　 /　 ヽ　　 ﾉ
　／￣|　　 |　|　|
　| (￣ヽ＿_ヽ_)_ )
　＼二つ
