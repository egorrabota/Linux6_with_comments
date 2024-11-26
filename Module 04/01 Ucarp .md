Установка
На node 1
```
apt install ucarp
```

Настройка
```
nano /etc/network/interfaces
```
```
auto eth1
iface eth1 inet static
        address 192.168.10.1
        netmask 255.255.255.0

        ucarp-vid 1
        ucarp-vip 192.168.10.254
        ucarp-password secret

iface eth1:ucarp inet static
        address 192.168.10.254
        netmask 255.255.255.255
```

Настройка скрипта 
```
nano /usr/share/ucarp/vip-up
```
```
#!/bin/sh

/sbin/ifup $1:ucarp
```
```
nano /usr/share/ucarp/vip-down
```
```
#!/bin/sh

/sbin/ifdown $1:ucarp
```
На node 2
```
apt install ucarp
```

Настройка
```
nano /etc/network/interfaces
```
```
auto eth1
iface eth1 inet static
        address 192.168.10.2
        netmask 255.255.255.0

        ucarp-vid 1
        ucarp-vip 192.168.10.254
        ucarp-password secret

iface eth1:ucarp inet static
        address 192.168.10.254
        netmask 255.255.255.255
```

Настройка скрипта 
```
nano /usr/share/ucarp/vip-up
```
```
#!/bin/sh

/sbin/ifup $1:ucarp
```
```
nano /usr/share/ucarp/vip-down
```
```
#!/bin/sh

/sbin/ifdown $1:ucarp
```

