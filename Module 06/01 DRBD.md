Установка и настройка


# 1. Схема стенда

# 2.Создание виртуальных машин в виртуализации

Шаг 1. Импорт VM

* выключение двух node 1-2

* Импорт 2виртуальных машин с Debian 


* В action выбрать import Virtual Machine...
```
2 машины без графики
```


Шаг 2. Переименование вируальных машин

```
Переименовать  в Node3
Переименовать  в Node4

```

Шаг 4. Подключение сетевых карт в виртуальных машинах



Открыть настройки node 3-4

1. Добавить Network Adapter
2. 1 network adapter подключить к External network
3. 2 network adapter подключить к Internal network

Подключиться к Node3 по SSH
```
User: root

Password: Pa$$w0rd
```
Настроить сетевые адреса
```
# nano /etc/network/interfaces
```
Проверить и добавить

```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
address 192.168.10.1/24

```

изменить имя компьютера
```
hostnamectl set-hostname  node1.corp.ru

```
Настройка hosts

```bash
# nano /etc/hosts
```
```
127.0.0.1               localhost
127.0.1.1                node1.corp.ru node1

192.168.10.1             node1.corp.ru node1
192.168.10.2            node2.corp.ru node2
```

Редактирование Resolv
```
nano /etc/resolv.conf
```
```
search corp.ru
nameserver 8.8.8.8
```

перезапустить сетевые адаптеры
```
systemctl restart networking
```

Подключиться к node 4
```
User: root

Password: Pa$$w0rd
```
Настроить сетевые адреса
```
# nano /etc/network/interfaces
```
Проверить и добавить

```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
address 192.168.10.2/24


```

изменить имя компьютера
```
hostnamectl set-hostname  node2.corp.ru

```
Настройка hosts
```
# nano /etc/hosts
```
```
127.0.0.1               localhost
127.0.1.1                node2.corp.ru node2

192.168.10.1             node1.corp.ru node1
192.168.10.2             node2.corp.ru node2
```

Редактирование Resolv
```
nano /etc/resolv.conf
```
```
search corp.ru
nameserver 8.8.8.8
```

перезапустить сетевые адаптеры
```
systemctl restart networking
```
Настройка праймери DNS сервиса 

Подключиться к node 1

Устанвока DNS
```
node1:~# apt install bind9 -y

```
Проверка сервиса
применяем только при перенастройки интерфейсов
```
systemctl status bind9
```
Проверка состовляемой конфигурации

```
Cat /etc/bind/named.conf
```

Настройка сервера перенаправляющего запросы на DNS cервер провайдера

```
# nano /etc/bind/named.conf.options
```
```
        forwarders {
                8.8.8.8;
        };
       dnssec-validation no;
       listen-on { 192.168.10.1; 127.0.0.1; };
       listen-on-v6 { ::; };
       

```
Проверка конфигурации

```
# named-checkconf -z
```

Рестарт службы DNS

```
systemctl restart bind9
```
Тестирование пересылки

```
nslookup -1=A ya.ru 127.0.0.1
```

Правим настройки DNS
```
nano /etc/resolv.conf
```
```
domain corp.ru
nameserver 127.0.0.1
```

Настройка Мастер сервера

```
nano /etc/bind/named.conf.local
```
```
zone "corp.ru" {
        type master;
        file "/etc/bind/corp.ru";
};
```

создание файла zone
```
nano /etc/bind/corp.ru
```
```
$TTL      3h
@         SOA     node1 root.node1  1 1d 12h 1w 3h
          NS      node1

ns        A       192.168.10.1
ns        A       192.168.10.2

$GENERATE 1-9 node$ A 192.168.10.$

gate      A       192.168.10.254

```
Проверка
```
node1# named-checkconf -z
```
Устанавливаем на второй узел 
```
node1# ssn2 apt install bind9
```
Перезапуск DNS

```
# rndc reload
```

node 2
```
nano /etc/resolv.conf
```
```
domain corp.ru
nameserver 127.0.0.1
```
   
Установка 

Проверям настройку времени на обоих узлах
```
node1# date
node1# ssn2 date
```
Настраиваем одну часовую зону
```
node1# timedatectl set-timezone Europe/Moscow
node1# ssn2 timedatectl set-timezone Europe/Moscow
```

Устнавливем пакет CSYNC2
```
node1# apt install csync2
node1# ssn2 apt install csync2

```

Генерируем ключ 
```
node1# csync2 -k /etc/csync2.key
node1# scp /etc/csync2.key node2:/etc/
```

Натсройка пакета
```
node1# nano /etc/csync2.cfg
```
Заменяем содержимое
```
nossl * *;

group dns
{
        host node1 node2;
       
        auto younger;  

         key /etc/csync2.key;
         include /etc/bind/;

        action
        {
                pattern /etc/bind/*;
                exec "service bind9 restart";

                do-local;
                logfile "/var/log/csync2_action.log";
        }
}
```

Копируем Настройки на второй узел
```
node1# scn2 /etc/csync2.cfg
```

Удалем все файлы конфигурации DNS с node2
```
node1# ssn2 rm -r /etc/bind
```

Синхронизация

```
node1# csync2 -xv
```
Ошибок не должно быть

Проверка работы DNS
```
node1# host node1 node1
node1# host node2 node2
```
Настройка на DNS
```
nodeN# nano /etc/resolv.conf
```
```
search corp.local
nameserver 192.168.X.1
nameserver 192.168.X.2
```


Подключаем к обоим узлам по дополнительному диску

Установка и базовая настройка
на обе ноды
```
apt install heartbeat
```
```
node# nano /etc/corosync/corosync.conf
```
```
totem {
        version: 2
        cluster_name: debian
        crypto_cipher: none
        crypto_hash: none
}

quorum {
       provider: corosync_votequorum
       two_node: 1
}

nodelist {
        node {
                name: node1
                nodeid: 1
                ring0_addr: node1
        }
        node {
                name: node2
                nodeid: 2
                ring0_addr: node2
        }
}
```
```
corosync -t
```
Перезапуск
```
service corosync restart
```


Проверка кластера
```
crm_mon -1
```


Управление конфигурацией
```
node1# apt install crmsh
```
Подготовка узлов

Добавляем жесткие диски, создаем точки монтирования

```
root@nodeN:~# lsblk

root@nodeN:~# mkdir /disk2
```
В Debian для сообщений используется postfix на обе ноды
```
# apt install postfix
```

на каждом узле отдельно
```
root@nodeN:~# apt install drbd-utils
```
```
root@nodeN:~# nano /etc/drbd.d/r0.res
```
```
resource r0 {
        syncer { rate 1000M; }
        handlers {
                split-brain "/usr/lib/drbd/notify-split-brain.sh root";
        }
        on node1.corp.ru {
                device /dev/drbd0;
                disk /dev/sdb;
                address 192.168.X.1:7788;
                meta-disk internal;
        }
        on node2.corp.ru {
                device /dev/drbd0;
                disk /dev/sdb;
                address 192.168.X.2:7788;
                meta-disk internal;
        }
}
```

```
root@nodeN:~# drbdadm create-md r0
```
Инициализация одновременно на обеих консолях node 3 и node 4
```
root@nodeN:~# service drbd start
```
```
root@nodeN:~# ps ax | grep drbd
```

Подключаемся не первый узел. Назначаем главный диск
```
root@node1:~# drbdadm -- --overwrite-data-of-peer primary r0
```

Мониторинг
```
root@nodeN:~# cat /proc/drbd
root@nodeN:~# watch cat /proc/drbd

root@nodeN:~# drbdadm sh-resources

root@nodeN:~# drbdadm cstate r0

root@nodeN:~# drbdadm dstate r0

root@nodeN:~# drbdadm role r0
```

Форматирование раздела

```
root@node1:~# mkfs.ext4 /dev/drbd0

root@node1:~# mount /dev/drbd0 /disk2

root@node1:~# mount | grep ext
```
```
cp /etc/passwd /disk2
```

Смена ролей узлов

Дождаться синхронизации
```
root@nodeN:~# cat /proc/drbd
```
отмантировать каталог
```
umount /disk2
```
Как синхронизация пройдет произвести переключение
```
node1# drbdadm secondary r0
node2# drbdadm primary r0
```
Монтируем 
```
node2# mount /dev/drbd0 /disk2
```
Кодготовка к кластеру

```
node2# umount /disk2
node2# drbdadm secondary r0
```

На обоих узлах останавливем службу

```
root@nodeN:~# systemctl stop drbd.service
```
Настройка кластера
```
crm configure
```
```
primitive pr_drbd_r0 ocf:linbit:drbd params drbd_resource="r0"
ms ms_drbd_r0 pr_drbd_r0 meta master-max="1" master-node-max="1" clone-max="2" clone-node-max="1" notify="true"
primitive pr_fs_r0 ocf:heartbeat:Filesystem params device="/dev/drbd0" directory="/disk2" fstype="ext4"
colocation col_fs_on_drbd inf: pr_fs_r0 ms_drbd_r0:Master
order or_fs_after_drbd Mandatory: ms_drbd_r0:promote pr_fs_r0:start
commit
quit
```
Проверяем
```
node1# crm status
```
Подключим ресурс к первой ноде 
```
crm configure location cli-prefer-pr_fs_r0 pr_fs_r0 role=Started inf: node1
```
Проверяем
```
node1# crm status
```
```
ls /disk2
```



