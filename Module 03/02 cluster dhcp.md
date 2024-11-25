Настройка отказоусточивого решения
node1
```
nano /etc/dhcp/dhcpd.general
```
```
ddns-update-style none;

log-facility local7;

subnet 192.168.10.0 netmask 255.255.255.0 {
  pool {
    failover peer "dhcp";
    range 192.168.10.128 192.168.10.228;
  }
  option routers 192.168.10.254;
  option domain-name "corp.ru";
  option domain-name-servers 192.168.X.1, 192.168.10.2;
  default-lease-time 600;
  max-lease-time 7200;
}

```
```
nano /etc/dhcp/dhcpd.conf
```
```
failover peer "dhcp" {
  primary;
  address 192.168.10.1;
  port 519;
  peer address 192.168.10.2;
  peer port 520;
  max-response-delay 60;
  max-unacked-updates 10;
  mclt 600;
  split 128;
  load balance max seconds 3;
}

include "/etc/dhcp/dhcpd.general";
```

node 2

```
nano /etc/dhcp/dhcpd.general
```
```
ddns-update-style none;

log-facility local7;

subnet 192.168.10.0 netmask 255.255.255.0 {
  pool {
    failover peer "dhcp";
    range 192.168.10.128 192.168.10.228;
  }
  option routers 192.168.10.254;
  option domain-name "corp.ru";
  option domain-name-servers 192.168.X.1, 192.168.10.2;
  default-lease-time 600;
  max-lease-time 7200;
}

```
```
nano /etc/dhcp/dhcpd.conf
```
```
failover peer "dhcp" {
  secondary;
  address 192.168.X.2;
  port 520;
  peer address 192.168.X.1;
  peer port 519;
  max-response-delay 60;
  max-unacked-updates 10;
  load balance max seconds 3;
}

include "/etc/dhcp/dhcpd.general";
```

Проверка конфигурации и запуск node1 и node2
```
# dhcpd -t

# service isc-dhcp-server restart

# service isc-dhcp-server status
```

Статистика DHCP сервера

```
# apt install dhcpd-pools

# dhcpd-pools

# dhcpd-pools -l /var/lib/dhcp/dhcpd.leases -c /etc/dhcp/dhcpd.conf
```
