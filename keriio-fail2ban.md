---
title: Kerio Connect - советы
description: светы и твики по настройке
published: true
date: 2021-02-11T07:47:44.665Z
tags: mailserver, kerio, connect
editor: markdown
dateCreated: 2021-02-11T07:47:44.665Z
---

# Защита
Что бы предварительно защитить сервер от подбороов паролей и сканированиия по ящикам делаем:

- ставим фильтр `apt install faill2ban`
- идем в настройки `/etc/fail2ban/jail.conf`
- ищем фильтр [kerio] и правим его вот так:

```
[kerio]
enabled  = true
port     = 25,465,587,110,995,143,993,4040,80,443
logpath  = /store/logs/security.log
bantime  = 14400
findtime = 20m
maxretry = 5
```

не забудьте проверить путь до лога `/store/logs/security.log`
сверху в конфиге в переменной `ignoreip = 127.0.0.1/8` дописываем через запятую все сети и\или адреса для исключения.
туда же надо добавить адрес забикс-сервера (любого вашего мониторинга), т.к. он быстро залочится.

- конфиг `/etc/fail2ban/filter.d/kerio.conf` должен выглядеть вот так:

```
# Fail2ban filter for kerio

[Definition]

failregex =     SMTP Spam attack detected from <HOST>,
                IP address <HOST> found in DNS blacklist
                Relay attempt from IP address <HOST>
                Attempt to deliver to unknown recipient .*,.*, IP address <HOST>
                Invalid password for user .* Attempt from IP address <HOST>
                User .* doesn’t exist. Attempt from IP address <HOST>
                Failed POP3 login from <HOST>, user .*
                Failed IMAP login from <HOST>, user .*
                Failed SMTP login from <HOST>
                SMTP: User .* doesn’t exist. Attempt from IP address <HOST>

ignoreregex =

[Init]
datepattern = ^\[%%d/%%b/%%Y %%H:%%M:%%S\]
```

- после чего перезапускаем демон - `service fail2ban restart`

в реальном времени можно смотреть как работает бан 
- `tail -f /var/log/fail2ban.log `