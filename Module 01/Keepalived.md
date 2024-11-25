
Установка keepalived

```
apt install keepalived
```

Настройка

```
nodeN# nano /etc/keepalived/keepalived.conf
```
```
vrrp_instance VI_1 {

#    state MASTER
#    state BACKUP

    interface eth0
#    interface br0
    virtual_router_id 1
    virtual_ipaddress {
        192.168.X.254 label eth0:1
#        192.168.X.254 label br0:1
        172.16.1.X/24 dev eth1
#        172.16.2.X/24 dev eth2
    }
#    virtual_routes {
#        0.0.0.0/0 via 172.16.1.254 dev eth1
#    }
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
        ip route add default via 172.16.1.254

#        ip route add default via 172.16.1.254 table 101
#        ip route add default via 172.16.2.254 table 102
#        /root/select_isp.sh
    ;;
    BACKUP)
        ip route delete default
        ip route add default via 192.168.X.254
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
