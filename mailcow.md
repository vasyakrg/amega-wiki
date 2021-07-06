---
title: Mailcow - почтовый сервер
description: 
published: true
date: 2021-07-06T02:01:34.979Z
tags: mail, mailserver, mailcow
editor: markdown
dateCreated: 2021-07-06T02:01:34.979Z
---

# Mailcow
Свой почтовый сервер

> Установка
{.is-info}

```bash
cd /opt
sudo git clone https://github.com/mailcow/mailcow-dockerized
cd mailcow-dockerized
```

Затем запустите скрипт чтобы загрузить зависимости Mailcow и сгенерировать конфиг файл:

`sudo ./generate_config.sh`

Сначала нужно указать имя почтового сервера (host name), например mail.example.ru:

Укажите временную зону (Europe/Moscow).

Timezone [Etc/UTC]: Europe/Moscow

Мы сгенерировали конфиг `mailcow.conf` на основе которого будет собираться связка из docker-контейнеров. При необходимости можно изменить порты если они у вас уже заняты, например, другими докер контейнерами, а также изменить `hostname` (MAILCOW_HOSTNAME=mail.example.ru)

**Порты по умолчанию:**

SMTP_PORT=25
SMTPS_PORT=465
SUBMISSION_PORT=587
IMAP_PORT=143
IMAPS_PORT=993
POP_PORT=110
POPS_PORT=995
SIEVE_PORT=4190
DOVEADM_PORT=127.0.0.1:19991
HTTP=80
HTTPS=443

Запускаем установку всех связанных контейнеров командой:

`sudo docker-compose up -d`

Установка может занять от 5 до 15 минут. В конце установки вы увидим что-то похожее…

Creating mailcowdockerized_dockerapi-mailcow_1 ... done
Creating mailcowdockerized_olefy-mailcow_1 ... done
Creating mailcowdockerized_memcached-mailcow_1 ... done
Creating mailcowdockerized_watchdog-mailcow_1 ... done
Creating mailcowdockerized_unbound-mailcow_1 ... done
Creating mailcowdockerized_sogo-mailcow_1 ... done
Creating mailcowdockerized_ejabberd-mailcow_1 ... done
Creating mailcowdockerized_clamd-mailcow_1 ... done
Creating mailcowdockerized_solr-mailcow_1 ... done
Creating mailcowdockerized_redis-mailcow_1 ... done
Creating mailcowdockerized_php-fpm-mailcow_1 ... done
Creating mailcowdockerized_mysql-mailcow_1 ... done
Creating mailcowdockerized_nginx-mailcow_1 ... done
Creating mailcowdockerized_dovecot-mailcow_1 ... done
Creating mailcowdockerized_postfix-mailcow_1 ... done
Creating mailcowdockerized_acme-mailcow_1 ... done
Creating mailcowdockerized_netfilter-mailcow_1 ... done
Creating mailcowdockerized_rspamd-mailcow_1 ... done
Creating mailcowdockerized_ipv6nat-mailcow_1 ... done

Теперь вы можете получить доступ к веб интерфейсу mailcow https://${MAILCOW_HOSTNAME} с учетными данными по умолчанию `admin` \ `moohoo`

> Лечение буквы `Б` в именах
{.is-info}

```bash
cp -fv /usr/lib/GNUstep/SOGo/WebServerResources/js/Mailer.services.js{,.bak}
sed -i 's/,n.KEY_CODE.COMMA,n.KEY_CODE.SEMICOLON//' /usr/lib/GNUstep/SOGo/WebServerResources/js/Mailer.services.js
```

```bash
cp -fv /usr/lib/GNUstep/SOGo/WebServerResources/js/Mailer.services.js.map{,.bak}
sed -i 's/"COMMA","SEMICOLON",//' /usr/lib/GNUstep/SOGo/WebServerResources/js/Mailer.services.js.map
```

```bash
cp -fv /usr/lib/GNUstep/SOGo/WebServerResources/js/Mailer/MessageEditorController.js{,.bak}
sed -i -e '/this\.recipientSeparatorKeys.*\[$/,/\];$/c\this\.recipientSeparatorKeys = \[\\$mdConstant\.KEY_CODE\.ENTER,\\$mdConstant\.KEY_CODE\.TAB\\];' /usr/lib/GNUstep/SOGo/WebServerResources/js/Mailer/MessageEditorController.js
```

после чего `перезапускаем` Sogo через веб-панель

> Модуль для LDAP
{.is-info}

https://github.com/Programmierus/ldap-mailcow

```bash
ldap-mailcow:
    image: programmierus/ldap-mailcow
    network_mode: host
    container_name: mailcowcustomized_ldap-mailcow
    depends_on:
        - nginx-mailcow
    volumes:
        - ./data/ldap:/db:rw
        - ./data/conf/dovecot:/conf/dovecot:rw
        - ./data/conf/sogo:/conf/sogo:rw
    environment:
        - LDAP-MAILCOW_LDAP_URI=ldap(s)://dc.example.local
        - LDAP-MAILCOW_LDAP_BASE_DN=OU=Mail Users,DC=example,DC=local
        - LDAP-MAILCOW_LDAP_BIND_DN=CN=Bind DN,CN=Users,DC=example,DC=local
        - LDAP-MAILCOW_LDAP_BIND_DN_PASSWORD=BindPassword
        - LDAP-MAILCOW_API_HOST=https://mailcow.example.local
        - LDAP-MAILCOW_API_KEY=XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX
        - LDAP-MAILCOW_SYNC_INTERVAL=300
        - LDAP-MAILCOW_LDAP_FILTER=(&(objectClass=user)(objectCategory=person)(memberOf:1.2.840.113556.1.4.1941:=CN=Group,CN=Users,DC=example DC=local))
        - LDAP-MAILCOW_SOGO_LDAP_FILTER=objectClass='user' AND objectCategory='person' AND memberOf:1.2.840.113556.1.4.1941:='CN=Group,CN=Users,DC=example DC=local'
```

**спасибо** [(С)](https://winitpro.ru/index.php/2021/04/05/mailcow-pochtovyj-server-postfix-dovecot-sogo-v-docker/)