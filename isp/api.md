---
title: Методы для работы с API
description: записная книжка по с API на ISPManager Business
published: true
date: 2020-02-09T09:21:23.687Z
tags: ispmanager, ispmgr, api, domain
---

# Работа с доменами
Готовый контейнер для работы с записями конкретного домена - vasyakrg/ispmgr-api
Может пригодиться как для простого управления записями через скрипты, так и при использовании автозаписей в k8s (пример ниже)

что бы создавать\удалять записи в конкретном домене, нужно:
знать свой сервер,
иметь логин и пароль от учетки, которая имеет право работать с доменом

работает так:
создаем конфиг файл .env с содержимым:

```
DNS_SERVER=isp.domain.ru
DNS_LOGIN=root
DNS_PASSWORD=password
DOMAIN_NAME=domain1.com
DNS_SETNAME=subdomain1
DNS_SETIP=5.5.5.5
```

> запускаем контейнер:
{.is-success}

чтобы создать
`docker run --rm -it --env-file=.env vasyakrg/ispmgr-api /bin/bash -c "./ispmgr.sh create"`

чтобы удалить:
`docker run --rm -it --env-file=.env vasyakrg/ispmgr-api /bin/bash -c "./ispmgr.sh delete"`