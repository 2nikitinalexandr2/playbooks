* создаемфайл конфигурации vagrant для 64-битного образа виртуальной машины с ubuntu trusty tahr
```
vagrant init ubuntu/trusty64
```
* загружаем её
```
vagrant up
```
* зайдем по ssh на нашу машину
```
vagrant ssh
```
* вывести детали ssh-подключения
```
vagrant ssh-config
```
```
Host default
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/user/playbooks/.vagrant/machines/default/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL
```
* в Vagrant 1.7 изменился порядок работы с приватными ssh ключами.
Начиная с этой версии Vagrant генерирует новый приватный ключ для каждой машины

* убедимся что можем начать новый ssh сеанс

```
ssh vagrant@127.0.0.1 -p 2222 -i /home/user/playbooks/.vagrant/machines/default/virtualbox/private_key
```

* Передать информацию о серверах в Ansible можно в файле реестра
* Каждому серверу должно быть присвоено имя для идентификации в Ansible
* С этой целью можно использовать имя хоста или выбрать другой псевдоним
* С именем также должны определяться дополнительные параметры подключения
* Присвоим серверу псевдоним testserver
* Файл hosts будет служить реестром

* проверить способность ansible подключиться к серверу, используем утилиту ansible
* попросим ansible установить соединение с сервером testserver, указанным в файле hosts и вызвать модуль ping:
```
ansible testserver -i hosts -m ping
```
```
testserver | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```
* если ansible сообщит об ошибке, добавьте команду флаг -vvv, чтобы получить больше информации об ошибке

* ansible.cfg - в нем определяются: 
  - местоположение файла реестра(inventory/hosts) - при использовании Vagrant
  - имя пользователя SSH(remote_user)
  * приватный ключ(private_key_file) 