Натсройка пакета
```
node1# nano /etc/csync2.cfg
```
добавить
```


group dhcp
{
        host node1 node2;
       
        auto younger;  

         key /etc/csync2.key;
         include /etc/dhcp/dhcpd.general;

        action
        {
                pattern /etc/dhcp/*;
                exec "service dhcp restart";

                do-local;
                logfile "/var/log/csync2_action.log";
        }
}
```

Копируем Настройки на второй узел
```
node1# scn2 /etc/csync2.cfg
```
```
csync2 -xv
```
