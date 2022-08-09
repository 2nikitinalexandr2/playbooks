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

* ansible.cfg(по умолчанию хранится в /etc/ansible/hosts) - в нем определяются: 
  - местоположение файла реестра(inventory/hosts) - при использовании Vagrant
  - имя пользователя SSH(remote_user)
  * приватный ключ(private_key_file) 



* В нашем примере конфигурации проверка SSH-ключей хоста отключена
* Это удобно при работе с Vagrant, в противном случае необходимо вносить изменения в файл ~/.ssh/known_hosts каждый раз, когда удаляется имеющийся или создается новый Vagrant-сервер.
* Однако отключение проверки ключей несет определенные риски

* С настройками по умолчанию отпадает необходимость указывать имя пользователя и файл с ключами SSH в файле hosts.

* Можно запустить ansible без аргумента `-i hostname`
```
ansible testserver -m ping
```
* получил ошибку:
```
testserver | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: no such identity: /home/user/playbooks/home/user/playbooks/.vagrant/machines/default/virtualbox/private_key: No such file or directory\r\nvagrant@127.0.0.1: Permission denied (publickey,password).",
    "unreachable": true
}
```
* нужно поменять путь в файле ansible.cfg
```
/home/user/playbooks/home/user/playbooks/.vagrant/machines/default/virtualbox/private_key:
```
* на
```
.vagrant/machines/default/virtualbox/private_key:
```
* проверить вермя работы сервера с момента последнего запуска с помощью модуля `command`
* при запуске модуля необходимо указать аргумент `-a` с запускаемой командой
```
ansible testserver -m command -a uptime
```
```
testserver | CHANGED | rc=0 >>
 09:31:47 up 15:33,  1 user,  load average: 0.00, 0.01, 0.05
```
* модуль `command` настолько часто используется, что сделан модулем по умолчанию,
* то есть его имя можно опустить в команде
```
ansible testserver -a uptime
```
* если команда в аргументе -a содержит пробелы, ее необходимо заключить в кавычки, 
* чтобы командная оболочка передала всю строку как единый аргумент. Например:
* извлечение нескольких строк последних строк из журнала `/var/log/dmseg`
```
ansible testserver -a "tail /var/log/dmesg"
```
```
testserver | CHANGED | rc=0 >>
[    6.250928] type=1400 audit(1659943007.223:6): apparmor="STATUS" operation="profile_replace" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-client.action" pid=937 comm="apparmor_parser"
[    6.250934] type=1400 audit(1659943007.223:7): apparmor="STATUS" operation="profile_replace" profile="unconfined" name="/usr/lib/connman/scripts/dhclient-script" pid=937 comm="apparmor_parser"
[    6.258340] type=1400 audit(1659943007.231:8): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/sbin/tcpdump" pid=940 comm="apparmor_parser"
[    6.298822] vboxvideo: Unknown symbol drm_open (err 0)
[    6.298829] vboxvideo: Unknown symbol drm_poll (err 0)
[    6.298835] vboxvideo: Unknown symbol drm_pci_init (err 0)
[    6.298842] vboxvideo: Unknown symbol drm_ioctl (err 0)
[    6.298848] vboxvideo: Unknown symbol drm_mmap (err 0)
[    6.298853] vboxvideo: Unknown symbol drm_pci_exit (err 0)
[    6.298859] vboxvideo: Unknown symbol drm_release (err 0)
```
* Чтобы выполнить команду с привелегиями root,
* нужно передать параметр -b. В этом случае Ansible выполнит команду от лица пользователя `root`
* Например для доступа к `/var/log/syslog` требуются привелегии root:
```
ansible testserver -b -a "tail /var/log/syslog"
```
```
testserver | CHANGED | rc=0 >>
Aug  9 09:34:17 vagrant-ubuntu-trusty-64 puppet-agent[1114]: Could not request certificate: getaddrinfo: Name or service not known
Aug  9 09:36:17 vagrant-ubuntu-trusty-64 puppet-agent[1114]: Could not request certificate: getaddrinfo: Name or service not known
Aug  9 09:37:29 vagrant-ubuntu-trusty-64 ansible-command: Invoked with creates=None executable=None _uses_shell=False strip_empty_ends=True _raw_params=uptime removes=None argv=None warn=True chdir=None stdin_add_newline=True stdin=None
Aug  9 09:38:17 vagrant-ubuntu-trusty-64 puppet-agent[1114]: Could not request certificate: getaddrinfo: Name or service not known
Aug  9 09:40:17 vagrant-ubuntu-trusty-64 puppet-agent[1114]: Could not request certificate: getaddrinfo: Name or service not known
Aug  9 09:41:26 vagrant-ubuntu-trusty-64 ansible-command: Invoked with creates=None executable=None _uses_shell=False strip_empty_ends=True _raw_params=tail /var/log/dmesg removes=None argv=None warn=True chdir=None stdin_add_newline=True stdin=None
Aug  9 09:42:17 vagrant-ubuntu-trusty-64 puppet-agent[1114]: Could not request certificate: getaddrinfo: Name or service not known
Aug  9 09:44:17 vagrant-ubuntu-trusty-64 puppet-agent[1114]: Could not request certificate: getaddrinfo: Name or service not known
Aug  9 09:50:17 vagrant-ubuntu-trusty-64 puppet-agent[1114]: message repeated 3 times: [ Could not request certificate: getaddrinfo: Name or service not known]
Aug  9 09:50:45 vagrant-ubuntu-trusty-64 ansible-command: Invoked with creates=None executable=None _uses_shell=False strip_empty_ends=True _raw_params=tail /var/log/syslog removes=None argv=None warn=True chdir=None stdin_add_newline=True stdin=None
```
* как видно Ansible 