В зону corp.ru добавляем запись fs A 192.168.10.20
```
node1# nano /etc/bind/corp.ru
```
```
fs        A       192.168.10.20
```
```
node1# named-checkconf -z

node1# csync2 -xv
```
На оба узла устанавливаем Samba
```
# apt install samba
```

Отключаем автоматический запуск сервиса
```
root@nodeN:~#

service smbd stop
service nmbd stop

systemctl disable smbd
systemctl disable nmbd
```
Публичный каталог доступный на запись создать на обоих узлах
```
# cat /etc/samba/smb.conf
```
```
[global]
   unix charset = UTF-8
   dos charset = cp866
   workgroup = CORPX
   security = user
   map to guest = Bad User
[pub_share]
   path = /disk2/samba
   guest ok = yes
   read only = no
   force user = games

```
```
# testparm
```

Настройка кластера
```
crm configure
```
```
crm(live)configure# primitive pr_srv_smbd systemd:smbd

crm(live)configure# primitive pr_ip_smbd ocf:heartbeat:IPaddr2 params ip=192.168.10.20 cidr_netmask=32 nic=eth1

crm(live)configure# group gr_ip_fs pr_fs_r0 pr_ip_smbd pr_srv_smbd
```

Тестирование
```
# watch cat /proc/drbd
# crm_mon
```
