# Ansible3
Практика

**Установка**

https://linuxopsys.com/topics/ansible-playbook-to-install-apache

```
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
```
**Посмотреть inventory**
```
ansible-inventory --list
```
**Запуск Ad-Hoc Команд**
```
ansible all -m ping - ping
ansible all -m setup - facts
ansible all -m shell -a 'uptime'
ansible all -m shell -a 'ls /etc | grep py' - через shell можно делать pipe, >, >> и т.д.
ansible all -m command -a 'ls /etc | grep py' - ERROR - не работает с pipe - для безопасности
ansible all -m copy -a "src=/root/hy.txt dest=/home/joos" - копируем файл
ansible all -m copy -a "src=/root/hy.txt dest=/home/joos mode=777"
ansible all -m file -a 'path=/home/joos/hy.txt state=absent' - удаляем файл
ansible all -m get_url -a 'url=https://pickimage.ru/wp-content/uploads/2020/08/rizhikot1.jpg dest=/home/joos' - качаем файл из интернета
ansible all -bK -m apt -a 'name=mc state=present' - ставим mc - -b = become - повышаем права - -K - спрашиваем пароль при повышении прав
ansible all -bK -m apt -a 'name=mc state=absent' - удаляем
ansible all -m uri -a 'url=https://ifconfig.me' - проверяем доступ к сайту - получаем заголовок
ansible all -m uri -a 'url=https://ifconfig.me return_content=yes' - получаем содержимое сайта
ansible all -bK -m apt -a 'name=apache2 state=latest' - ставим apache2
ansible all -bK -m service -a 'name=apache2 state=started enabled=yes' - запускаем и включаем запуск при старте apache2
ansible all -bK -m apt -a 'name=apache2 state=absent'
ansible all -m shell -a 'ls' -vvv - подробный вывод для анализа ошибок
ansible-doc -l - все модули
```
**Перенос переменных в group_vars**
```
mkdir group_vars - создаем директорию для переменных
nano group_vars/server1 - по названию группы создаем файл
file
---
ansible_user : joos - например имя пользователя при коннекте
file
```
**{"msg": "Missing sudo password"}**

https://www.ansiblepilot.com/articles/ansible-troubleshooting-missing-sudo-password/
```
visudo
joos  ALL=(ALL) NOPASSWD: ALL

sudo nano /etc/sudoers.d/joos
joos ALL=(ALL) NOPASSWD: ALL
```

**Playbook**

Test connection
```yml
---
- name: Test connection to my servers
  hosts: all
  become: yes

  tasks:
  - name: Ping my servers
    ping:
```
Install Apache
```yml
---
- name: Install Apache Web Server
  hosts: all
  become: yes
  tasks:
  - name: Install Apache Web Server
    apt: name=apache2 state=latest

  - name: Start Apache and Enable Service
    service: name=apache2 state=started enabled=yes

```
Install Apache and copy index.html
```yml
---
- name: Install Apache and Upload my Web Page
  hosts: all
  become: yes

  vars:
    source_file: ./index.html
    destin_file: /var/www/html

  tasks:
  - name: Install Apache Web Server
    apt: name=apache2 state=latest
  - name: Copy MyPage to Servers
    copy: src={{ source_file }} dest={{ destin_file }} mode=0555
    notify: Restart Apache
  - name: Start WebServer and make it enable
    service: name=apache2 state=started enabled=yes

  handlers:
  - name: Restart Apache
    service: name=apache2 state=restarted
```
**Debug, Set_fact, Register**
```yml
---
- name: My playbook
  hosts: all
  become: yes

  vars:
    message1: Privet
    message2: World
    secret: asdkasdlkjahsldjk

  tasks:
  - name: Print Secret
    debug:
      var: secret
  - debug:
      msg: 'Sekretnoe slovo: {{ secret }}'
  - debug:
      msg: "Vladelec Serverd -->{{ owner }}<--" # - owner - добавляем в group_vars/server1 - owner : Petya
  - set_fact: full_message="{{ message1 }} {{ message2 }} from {{ owner }}" # - собираем переменные
  - debug:
      var: full_message # - выводим переменные
  - debug:
      var: ansible_distribution # - переменные из facts в моем случае Debian
  - shell: uptime  # - если просто запустить, то выполнится на удаленном сервере и ничего не вернет, надо сохранить вывод
    register: results # - сохраняем вывод в results
  - debug:
      var: results.stdout # - выводим results и уменьшаем вывод через stdout
```
**Блоки и Условия – Block-When**

Переносим для разных систем в разные блоки и делаем общую команду when: ansible_os_family ==
```yml
---
- name: Install Apache and Upload my Web Page
  hosts: all
  become: yes

  vars:
    source_file: ./index.html
    destin_file: /var/www/html

  tasks:
  - name: Check and Print Linux-Family
    debug: var=ansible_os_family

  - block: # for Debian
    - name: Install Apache Web Server for Debian
      apt: name=apache2 state=latest
    - name: Copy MyPage to Servers
      copy: src={{ source_file }} dest={{ destin_file }} mode=0555
      notify: Restart Apache Debian
    - name: Start WebServer and make it enable for Debian
      service: name=apache2 state=started enabled=yes
    when: ansible_os_family == "Debian"

  - block: # for RedHat
    - name: Install Apache Web Server for RedHat
      yum: name=httpd state=latest
    - name: Copy MyPage to Servers
      copy: src={{ source_file }} dest={{ destin_file }} mode=0555
      notify: Restart Apache RedHat
    - name: Start WebServer and make it enable for RedHat
      service: name=httpd state=started enabled=yes
    when: ansible_os_family == "RedHat"

  handlers:
  - name: Restart Apache Debian
    service: name=apache2 state=restarted

  - name: Restart Apache RedHat
    service: name=httpd statr=restarted
```
**Циклы – Loop, With_Items, Until, With_fileglob**
```yml
---
- name: Loops Playbook
  hosts: all
  become: yes

  tasks:
  - name : Say Hello to All
    debug: msg="Hello {{ item }}"
    loop: # - в старых версиях with_items
      - "Vasya"
      - "Mascha"
      - "Igor"
  - name: Loop Until example
    shell: echo -n Z >> myfile.txt && cat myfile.txt
    register: output # - сохраняем вывод
    delay: 2 # - делаем переры 2 сек между попытками
    retries: 10 # -количество попыток - по умолчанию 3
    until: output.stdout.find("ZZZZ") == false  # - делаем пока не получим ZZZZ

  - name: Print Output
    debug:
      var: output.stdout # - печатаем output

  - name: Install many package # - ставим много пакетов
    apt: name={{ item }} state=latest
    loop:
      - mc
      - tree
      - sysstat
```
```
---yml
- name: Install Apache and Upload my Web Page
  hosts: all
  become: yes

  vars:
    source_folder: ./mysite
    destin_folder: /var/www/html

  tasks:
  - name: Check and Print Linux-Family
    debug: var=ansible_os_family

  - block: # for Debian

    - name: Install Apache Web Server for Debian
      apt: name=apache2 state=latest
    - name: Start WebServer and make it enable for Debian
      service: name=apache2 state=started enabled=yes
    when: ansible_os_family == "Debian"

  - block: # for RedHat

    - name: Install Apache Web Server for RedHat
      yum: name=httpd state=latest
    - name: Start WebServer and make it enable for RedHat
      service: name=httpd state=started enabled=yes
    when: ansible_os_family == "RedHat"

  - name: Copy MyPage to Servers
#    copy: src={{ source_folder }}/{{ item }} dest={{ destin_folder }} mode=0555 - первый вариант копирования пофайлово
#    loop:
#      - "index.html"
#      - "1.jpg"
#      - "2.jpg"
    copy: src={{ item }} dest={{ destin_folder }} mode=0555 # - второй вариант копирования - вся папка
    with_fileglob: "{{ source_folder }}/*.*"
    notify:
      - Restart Apache RedHat
      - Restart Apache Debian

  handlers:
  - name: Restart Apache Debian
    service: name=apache2 state=restarted
    when: ansible_os_family == "Debian"

  - name: Restart Apache RedHat
    service: name=httpd statr=restarted
    when: ansible_os_family == "RedHat"
```
**Шаблоны - Jinja Template**
```yml
---
- name: Install Apache and Upload my Web Page
  hosts: all
  become: yes

  vars:
    source_folder: ./website2
    destin_folder: /var/www/html

  tasks:
  - name: Check and Print Linux-Family
    debug: var=ansible_os_family

  - block: # for Debian

    - name: Install Apache Web Server for Debian
      apt: name=apache2 state=latest
    - name: Start WebServer and make it enable for Debian
      service: name=apache2 state=started enabled=yes
    when: ansible_os_family == "Debian"

  - name: Generate INDEX.HTML file
    template: src={{ source_folder }}/index.j2 dest={{ destin_folder }}/index.html mode=0555
    notify:
      - Restart Apache Debian

  handlers:
  - name: Restart Apache Debian
    service: name=apache2 state=restarted
```
```html
<HTML>

<HEAD>

<TITLE>ADV-IT</TITLE>

<BODY bgcolor="black" onLoad=Elastic()>
<CENTER>

<br><br><br><br>

<br><br><br><br>

<font color="green"><h2>Owner of this Server is: {{ owner }}</h2>

<font color="green"><h2>This Page was created with</h2>

<font color="gold"><H1 ID="elastic" ALIGN="Center">ANSIBLE-BLA-BLA</H1>

<font color="gold"><H1 ID="elastic" ALIGN="Center">Server Host Name : {{ ansible_hostname }}</H1>
<font color="gold"><H1 ID="elastic" ALIGN="Center">Server OS Family : {{ ansible_os_family }}</H1>
<font color="gold"><H1 ID="elastic" ALIGN="Center">IP Adress : {{ ansible_default_ipv4.address }}</H1>

</body>
</HTML>

```
