[[docker]]

Если запустить БД локально, а сервис, который будет подключатся к этой БД, запустить в докер контейнере то по ==localhost== сервис не сможет достучаться до БД, ему нужно сообщить реальный локальный IP используемый на машине, т.к. внутренний DNS docker не знает что находится под элиасом ==localhost==
свой локальный адрес можно узнать через команду:
```shel
ip address
```
ответ:
```shel
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:4e:01:c5:bf:5b brd ff:ff:ff:ff:ff:ff
    altname enp0s31f6
    inet 172.16.50.148/24 brd 172.16.50.255 scope global dynamic noprefixroute eno1
       valid_lft 146966sec preferred_lft 146966sec
    inet6 fe80::5246:f208:dcb4:a6c0/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever

```
нам нужен адрес ==eno1== он будет отличаться от указанного но надо использовать его.