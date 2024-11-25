
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
    interface eth0

    virtual_router_id 1
    virtual_ipaddress {
        192.168.10.254 label eth0:1 192.168.123.2/24 dev eth0
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
    interface eth0

    virtual_router_id 1
    virtual_ipaddress {
        192.168.10.254 label eth0:1 192.168.123.2/24 dev eth0
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
