---
title: Zabbix - контроль SSL
description: проверяем SSL сертификаты на живость по списку доменов
published: true
date: 2021-03-28T14:57:29.048Z
tags: zabbix, zabbix-agent, ssl, letsencrypt
editor: markdown
dateCreated: 2021-03-28T14:47:56.086Z
---

# Zabbix SSL мониторинг

> Проверка на живость с консоли
{.is-success}


```
openssl s_client -connect domain.ru:443 -servername domain.ru -tlsextdebug 2>/dev/null | openssl x509 -noout -dates 2>/dev/null | grep notAfter | cut -d'=' -f2
```

## Настройка со стороны агента

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

> Не забывайте подставлять значения своих доменов, чтобы наверняка быть уверенными в том, что они нормально отдают сертификаты. На выходе у вас должна быть только цифра с количеством дней актуальности ssl сертификата. Больше ничего. Это важно, иначе заббикс сервер будет выдавать ошибку.
{.is-info}


Связываем все наши скрипты с самим zabbix.
Создаем файл с расширением конфигурации заббикса:

```
mcedit /etc/zabbix/zabbix_agentd.d/ssl.conf
```

Содержимое:

```
UserParameter=ssl_https.discovery[*],/usr/lib/zabbix/externalscripts/disc_ssl_https.sh
UserParameter=ssl_https.expire[*],/usr/lib/zabbix/externalscripts/check_ssl_https.sh $1
```

Сохраняем все конфиги. Делаем пользователя zabbix владельцем всех наших скриптов.

```
chown -R zabbix. /usr/lib/zabbix/externalscripts/
systemctl restart zabbix-agent
```


Проверяем что работает (вторая команда - домен нужен из списка `ssl_https.txt`)

```
zabbix_agentd -t ssl_https.discovery
zabbix_agentd -t ssl_https.expire[mail.ru]
```

## Теперь со стороны Заббикса

- Качаем [шаблон](https://serveradmin.ru/files/zabbix/ssl_cert_expiration.xml) и импортируем его в заббикс в разделе Шаблоны
- Прикрепляйте этот шаблон к тому хосту, где вы настраивали скрипты и zabbix-agent. Дальше вам остается только подождать примерно 5 минут. Такой интервал установлен для автообнаружения.
- Еще примерно через 5 минут, вы получите информацию о сроке действия сертификатов указанных доменов.

> Итог:
{.is-success}


![2021-03-28_21-55-gdsch(1).png](/2021-03-28_21-55-gdsch(1).png)

> В шаблоне настроен триггер, который срабатывает, если время жизни сертификата становится меньше 30-ти дней. В принципе, этот параметр можно уменьшить и до 10-ти дней.
{.is-info}

(С) https://serveradmin.ru/monitoring-sroka-deystviya-ssl-sertifikata-v-zabbix/
