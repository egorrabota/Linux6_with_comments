Настройка доступа по протоколу ssh пользователя root с node1 на node2

```
node1# ping node2
```
```
node1# ssh-keygen
...
Enter passphrase (empty for no passphrase): Пароль на ключ пустой!!!
...
```
```
node1# ssh-copy-id node2
```

Копируем настройки с node 1 на node 2

```
node1# 

node1# ssh node2 apt update
node1# ssh node2 apt install keepalived
scp /etc/keepalived/keepalived.conf node2:/etc/keepalived/keepalived.conf
scp /usr/local/bin/vrrp.sh node2:/usr/local/bin/vrrp.sh
```
Перенастроить keepalived

```
node2# nano /etc/keepalived/keepalived.conf
```
```
...
#    state MASTER
    state BACKUP
...
```

Упростим вызов команд за счет Alias
```
node1# nano .bashrc
```
Добавить в конец

```
alias ssn2='ssh node2'

scn2() {
        scp $1 node2:$1
}
```

Применим настройки
```
source .bashrc
```


