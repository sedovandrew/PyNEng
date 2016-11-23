### Ad Hoc команды

Ad-hoc команды - это возможность запустить какое-то действие Ansible из командной строки.

Такой вариант подходит в тех случаях, когда вам надо просто что-то проверить, например, работу модуля.
Или просто выполнить какое-то разовое действие, которое вы не хотите сохранять.

В любом случае, это простой и быстрый способ начать использовать Ansible.

Для начала, нам нужно создать в локальном каталоге инвентарный файл. Назовем его myhosts:
```
[cisco-routers]
192.168.100.1
192.168.100.2
192.168.100.3

[cisco-switches]
192.168.100.100
```

> Если вы подключаетесь к этим устройствам первый раз, то сначала подключитесь к ним вручную, чтобы ключи устройств были сохранены локально. В Ansible есть возможность отключить эту первоначальную проверку ключей. В разделе о конфигурационном файле мы посмотрим как это делать (такой вариант может понадобиться, если вам надо будет подключаться к большому количеству устройств).

Попробуем выполнить ad-hoc команду:
```
$ ansible cisco-routers -i myhosts -m raw -a "sh ip int br" -u cisco --ask-pass
```

Разберемся с параметрами команды:
* ```cisco-routers``` - группа устройств, к которым нужно применить действия
 * эта группа должна существовать в инвентарном файле
 * это может быть конкретное имя или адрес
 * если нужно указать все хосты из файла, можно использовать значение all или *
 * Ansible поддерживает более сложные варианты указания хостов, с регулярными выражениями и разными шаблонами. Подробнее об этом в [документации](http://docs.ansible.com/ansible/intro_patterns.html)
* ```-i myhosts``` - параметр -i позволяет указать инвентарный файл
 * позже, мы посмотрим как настроить Ansible, чтобы не нужно было указывать инвентарный файл в командной строке
* ```-m raw -a "sh ip int br"``` - параметр ```-m raw``` означает, что мы хотим использовать модуль raw
 * этот модуль позволяет отправлять команды в SSH сессии, но при этом не загружает на хост модуль Python. То есть, этот модуль просто отправляет указанную команду как строку и всё
 * ```-a "sh ip int br"``` - параметр ```-a``` позволяет указать команду, которую мы хотим отправить
* ```-u cisco``` - будем подключаться используя пользователя с именем cisco
* ```--ask-pass``` - параметр который нужно указать, чтобы аутентификация была по паролю, а не по ключам


Результат выполнения будет таким:
```
$ ansible cisco-routers -i myhosts -m raw -a "sh ip int br" -u cisco --ask-pass
SSH password:
192.168.100.1 | FAILED | rc=0 >>
to use the 'ssh' connection type with passwords, you must install the sshpass program

192.168.100.2 | FAILED | rc=0 >>
to use the 'ssh' connection type with passwords, you must install the sshpass program

192.168.100.3 | FAILED | rc=0 >>
to use the 'ssh' connection type with passwords, you must install the sshpass program
```

Ошибка значит, что нужно установить программу sshpass. Эта особенность возникает только тогда, когда мы используем аутентификацию по паролю.

Устанаваливаем sshpass:
```
$ sudo apt-get install sshpass
```

И повторяем команду:
```
$ ansible cisco-routers -i myhosts -m raw -a "sh ip int br" -u cisco --ask-pass
SSH password:
192.168.100.1 | SUCCESS | rc=0 >>

Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                192.168.100.1   YES NVRAM  up                    up
Ethernet0/1                192.168.200.1   YES NVRAM  up                    up
Ethernet0/2                unassigned      YES NVRAM  administratively down down
Ethernet0/3                unassigned      YES NVRAM  administratively down down
Connection to 192.168.100.1 closed by remote host.
Shared connection to 192.168.100.1 closed.


192.168.100.2 | SUCCESS | rc=0 >>

Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                192.168.100.2   YES manual up                    up
Ethernet0/1                unassigned      YES unset  administratively down down
Ethernet0/2                unassigned      YES unset  administratively down down
Ethernet0/3                unassigned      YES unset  administratively down down
Shared connection to 192.168.100.2 closed.


192.168.100.3 | SUCCESS | rc=0 >>

Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                192.168.100.3   YES manual up                    up
Ethernet0/1                unassigned      YES unset  administratively down down
Ethernet0/2                unassigned      YES unset  administratively down down
Ethernet0/3                unassigned      YES unset  administratively down down
Connection to 192.168.100.3 closed by remote host.
Shared connection to 192.168.100.3 closed.
```

Теперь всё прошло успешно. Команда выполнилась и мы видим вывод с каждого устройства.

Аналогичным образом можно попробовать выполнять и другие команды и/или на других комбинациях устройств.

На этом базовое знакомство с Ansible заканчивается. Теперь нам нужно разобраться с тем как управлять настройками Ansible и как создавать файлы со сценариями (playbook).