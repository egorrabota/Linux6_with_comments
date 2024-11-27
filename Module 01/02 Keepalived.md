
Установка keepalived

```
apt install keepalived
```
Node1
Настройка
Node 1
```
nodeN# nano /etc/keepalived/keepalived.conf
```
```
vrrp_instance VI_1 {

    state MASTER
    interface eth1

    virtual_router_id 1
    virtual_ipaddress {
        192.168.10.254/24 label eth0:1
    }
    notify_backup "/usr/local/bin/vrrp.sh BACKUP"
    notify_master "/usr/local/bin/vrrp.sh MASTER"
}
```
Node 2
```
nodeN# nano /etc/keepalived/keepalived.conf
```
```
vrrp_instance VI_1 {

    state BACKUP
    interface eth1

    virtual_router_id 1
    virtual_ipaddress {
        192.168.10.254/24 label eth0:1
    }
    notify_backup "/usr/local/bin/vrrp.sh BACKUP"
    notify_master "/usr/local/bin/vrrp.sh MASTER"
}
```
```
nodeN# nano /usr/local/bin/vrrp.sh
```
```
#!/bin/sh

#echo $1 >> /tmp/vrrp.txt

case $1 in
    MASTER)
        ip route delete default
        ip route add default via 192.168.123.2
    ;;
    BACKUP)
        ip route delete default
        ip route add default via 192.168.10.254
    ;;
esac
```
```
nodeN# chmod +x /usr/local/bin/vrrp.sh
```

Запуск и мониторинг

```
# keepalived -t

# service keepalived restart

# watch "service keepalived status | cat"

# watch ipvsadm -L -n

# ipvsadm -L -n -c
```


> Целевое состояние:
  -  Доступный vIP (виртуальный адрес) в соответствии с keepalived.conf
  - `node1` keepalived - MASTER
  - `node2` keepalived - BACKUP
  - При отключении сетевого интерфейса на `node1` с консоли - `node2` переходит в MASTER и получает дефолтный маршрут в интернет.
  - После обратного включения `node1` - `node2` теряет MASTER вместе с маршрутом в интернет.
