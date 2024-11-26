На Node1
```
nano /etc/network/interfaces
```
```
auto eth0
iface eth0 inet manual
        up ip link set eth0 up

auto eth1
iface eth0 inet static
        address 192.168.10.1
        netmask 255.255.255.0
        gateway 192.168.10.1
```

```
node1# nano /etc/keepalived/keepalived.conf
```
```
vrrp_instance VI_1 {

  state MASTER

    interface eth1
    virtual_router_id 1
    virtual_ipaddress {
        192.168.123.121/24 dev eth0
        192.168.10.254/24 dev eth1
    }
    virtual_routes {
        0.0.0.0/0 via 192.168.123.2 dev eth0
    }
    notify_backup "/usr/local/bin/vrrp.sh BACKUP"
    notify_master "/usr/local/bin/vrrp.sh MASTER"
}
```
На Node2
```
nano /etc/network/interfaces
```
```
auto eth0
iface eth0 inet manual
        up ip link set eth0 up

auto eth1
iface eth0 inet static
        address 192.168.10.2
        netmask 255.255.255.0
        gateway 192.168.10.1
```

```
node2# nano /etc/keepalived/keepalived.conf
```
```
vrrp_instance VI_1 {

  state BACKUP

    interface eth1
    virtual_router_id 1
    virtual_ipaddress {
        192.168.123.121/24 dev eth0
        192.168.10.254/24 dev eth1
    }
    virtual_routes {
        0.0.0.0/0 via 192.168.123.2 dev eth0
    }
    notify_backup "/usr/local/bin/vrrp.sh BACKUP"
    notify_master "/usr/local/bin/vrrp.sh MASTER"
}
```
