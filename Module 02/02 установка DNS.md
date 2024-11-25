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
Заменяем
```
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
@         SOA     ns root.gate  1 1d 12h 1w 3h
          NS      ns

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
