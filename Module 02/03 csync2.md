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
        host node1.corp.local node2.corp.local;
       
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
