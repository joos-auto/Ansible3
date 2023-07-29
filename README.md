# Ansible3
Практика

**Установка**
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
root@anmaster:~# ansible all -bK -m apt -a 'name=mc state=absent' - удаляем
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

---
ansible_user : joos - например имя пользователя при коннекте
```



