---
title: Кеширующий ДНС
description: свой днс сервер для локальной сети
published: true
date: 2020-06-08T14:18:14.599Z
tags: dns, cache-dns, bind9
editor: markdown
---

# Вводная
Используем Ubuntu 18.04

сервер - `192.168.0.2`
клиент - `192.168.0.3`

- Ставим пакеты на сервер

```
sudo apt-get update
sudo apt-get install bind9 bind9utils bind9-doc
```

- Все коифиги bind9 лежат в `/etc/bind`

- Настриваем `named.conf.options`

```
acl "trusted" {
    localhost;
    192.168.0.2;    # ns1 - can be set to localhost
    192.168.0.63;  # host1
};

options {
        directory "/var/cache/bind";
        recursion yes;                 # enables resursive queries
        allow-query { trusted; };  # allows recursive queries from "trusted" clients

        forward only;

        forwarders {
            8.8.8.8;
            8.8.4.4;
        };

        dnssec-enable yes;
        dnssec-validation yes;

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```

- Проверяем, что нигде не опечатались, командой
```
named-checkconf
```

- Перезапускаем сервис
```
service bind9 restart
```

- открываем лог на чтение 

(прервать можно через Ctrl+C)

```
tail -f /var/log/syslog
```


## Во второй консоли открываем клиента

- правим etc/resolv.conf

```
#nameserver 192.168.9.1
nameserver 192.168.0.2
```

- проверяем
```
dig linuxfoundation.org
```

- смотрим на 
```
;; Query time: 36 msec
```

- еще раз запускаем 
```
dig linuxfoundation.org
```

- и 
```
;; Query time: 1 msec
```

> нужный результат получен.
{.is-success}


## На последок
что бы клиент не забыл настроек (а `etc/resolv.conf` перезатрется после перезагрузки), идем в 
`/etc/network/interfaces` и добавляем `dns-nameservers 192.168.0.2`

должно получится что-то типа этого
```
. . .
iface eth0 inet static
address 111.111.111.111
netmask 255.255.255.0
gateway 111.111.0.1
dns-nameservers 192.168.0.2
. . .
```

перезапускаем сетевой сервис 
`service networking restart`

done.