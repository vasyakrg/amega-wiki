---
title: Zabbix - контроль SSL
description: проверяем SSL сертификаты на живость по списку доменов
published: true
date: 2021-03-28T14:47:56.086Z
tags: zabbix, zabbix-agent, ssl, letsencrypt
editor: markdown
dateCreated: 2021-03-28T14:47:56.086Z
---

# Zabbix SSL мониторинг

Проверка на живость с консоли

```
openssl s_client -connect domain.ru:443 -servername domain.ru -tlsextdebug 2>/dev/null | openssl x509 -noout -dates 2>/dev/null | grep notAfter | cut -d'=' -f2
```

Создаем файл списка, куда будем вписывать на каждой строчке по домену

```
touch /usr/lib/zabbix/externalscripts/ssl_https.txt
```

Дальше добавляем скрипты для автообнаружения этих доменов и передачи в zabbix

```
touch /usr/lib/zabbix/externalscripts/check_ssl_https.sh && chmod 0740 /usr/lib/zabbix/externalscripts/check_ssl_https.sh
```

Вписываем туда:

```
#!/bin/bash

JSON=$(for i in `cat /etc/zabbix/scripts/ssl_https.txt`; do printf "{\"{#DOMAIN_HTTPS}\":\"$i\"},"; done | sed 's/^\(.*\).$/\1/')
printf "{\"data\":["
printf "$JSON"
printf "]}"
```

Для проверки достаточно выполнить один из скриптов. На выходе должен быть вывод списка доменов в формате JSON.

```
{"data":[{"{#DOMAIN_HTTPS}":"serveradmin.ru"}]}
```

Пишем скрипт, который будут определять, сколько дней осталось до окончания срока действия сертификата. За основу он будут брать ту дату, что мы выводили на предыдущем шаге.

```
touch /usr/lib/zabbix/externalscripts/check_ssl_https.sh && chmod 0740 /usr/lib/zabbix/externalscripts/check_ssl_https.sh
```

Вписываем туда код:

```
#!/bin/bash

SERVER=$1
TIMEOUT=25
RETVAL=0
TIMESTAMP=`echo | date`
EXPIRE_DATE=`echo | openssl s_client -connect $SERVER:443 -servername $SERVER -tlsextdebug 2>/dev/null | openssl x509 -noout -dates 2>/dev/null | grep notAfter | cut -d'=' -f2`
EXPIRE_SECS=`date -d "${EXPIRE_DATE}" +%s`
EXPIRE_TIME=$(( ${EXPIRE_SECS} - `date +%s` ))
if test $EXPIRE_TIME -lt 0
then
RETVAL=0
else
RETVAL=$(( ${EXPIRE_TIME} / 24 / 3600 ))
fi

echo ${RETVAL}
```

Запускаем и проверяем, что работает:

```
/usr/lib/zabbix/externalscripts/check_ssl_https.sh mail.ru
```



