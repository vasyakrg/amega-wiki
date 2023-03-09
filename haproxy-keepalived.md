---
title: HAProxy + Keepalived
description: пример настройки двух балансеров и VIP-адреса через keepalived
published: true
date: 2023-03-09T06:49:24.018Z
tags: proxy, balancer, haproxy, keepalived
editor: markdown
dateCreated: 2023-03-09T06:47:35.291Z
---

# HAProxy + Keepalived
пример настройки двух балансеров и VIP-адреса через keepalived

**общая схема**

![monosnap-2023-03-09_13-47-24.jpg](/monosnap-2023-03-09_13-47-24.jpg)

адресация с моего стенда:

web1: 192.168.9.151
web2: 192.168.9.152

lb1: 192.168.9.153
lb2: 192.168.9.154

vip: 192.168.9.105

**на web1\web2**

```bash
apt update && apt install -y apache2
systemctl enable --now apache2
```

можно так же подменить странички в /var/www/html/index.html, что бы видеть, на каком сервере вы находитесь сейчас

**на lb1\lb2**

```bash
apt update && apt install -y haproxy keepalived
systemctl enable --now haproxy
systemctl enable --now keepalived
```

добавляем настройку для сети, что бы приложения могли создавать виртуальные адреса без привязки к настройкам в /etc/interface

`nano /etc/sysctl.conf`

```config
	...
  
net.ipv4.ip_nonlocal_bind=1
```

**конфиг для haproxy общий**

```config
	...
  
listen  stats
        bind    0.0.0.0:8989
        mode    http
        stats   enable
        stats   uri     /haproxy_stats
        stats   realm HAProxy\  Statistics
        stats   auth    admin:pass123
        stats   admin   if TRUE

frontend my-web
        bind 192.168.9.150:80
        default_backend my-web

backend my-web
        balance roundrobin
        server  web1    192.168.9.151:80        check
        server  web2    192.168.9.152:80        check
```

**конфиг для lb1**

```config
global_defs {
        router_id lb01
}

vrrp_script check_haproxy {
        script "/usr/bin/systemctl is-active --quiet haproxy"
        interval 2
        weight 2
}

vrrp_instance my-web {
        state MASTER
        interface eth0
        virtual_router_id 123
        priority 100
        advert_int 1
        authentication {
                auth_type PASS
                auth_pass myPass12
        }
        virtual_ipaddress {
                192.168.9.150
        }
        track_script {
                check_haproxy
        }
}
```

**конфиг для lb1**

```config
global_defs {
        router_id lb01
}

vrrp_script check_haproxy {
        script "/usr/bin/systemctl is-active --quiet haproxy"
        interval 2
        weight 2
}

vrrp_instance my-web {
        state BACKUP
        interface eth0
        virtual_router_id 123
        priority 99
        advert_int 1
        authentication {
                auth_type PASS
                auth_pass myPass12
        }
        virtual_ipaddress {
                192.168.9.150
        }
        track_script {
                check_haproxy
        }
}
```