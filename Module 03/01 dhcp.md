Шаг 1. Установка ПО

Залогиниться на node1 и node2

Установка DHCP
```
node1:~# apt install isc-dhcp-server

```
Настройка Интерфейса

```
node1:~# nano /etc/default/isc-dhcp-server
```
```
INTERFACESv4="eth1"
```

Настройка Scope

```
node1# nano /etc/dhcp/dhcpd.conf

```
```
log-facility local7;

shared-network LAN1 {
  subnet 192.168.10.0 netmask 255.255.255.0 {
    range 192.168.10.101 192.168.10.128;
    option routers 192.168.10.1;
    option domain-name "corp.ru";
    option domain-name-servers 192.168.10.1, 192.168.10.10;
    default-lease-time 600;
    max-lease-time 7200;
  }
}
```

Проверка DHCP

```
# dhcpd -t
```

Перезапуск службы DHCP

```
# service isc-dhcp-server restart

# service isc-dhcp-server status
```
