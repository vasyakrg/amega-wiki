---
title: Методы для работы с API
description: записная книжка по с API на ISPManager Business
published: true
date: 2020-02-09T09:32:51.899Z
tags: ispmanager, ispmgr, api, domain
---

# Работа с доменами с контейнера
Готовый контейнер для работы с записями конкретного домена - `vasyakrg/ispmgr-api`
Может пригодиться как для простого управления записями через скрипты, так и при использовании автозаписей в k8s (пример ниже)

что бы создавать\удалять записи в конкретном домене, нужно:
- знать свой сервер,
- иметь логин и пароль от учетки, которая имеет право работать с доменом

> работает так:
{.is-success}

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

### Замечания
- при создании проверяется, существует ли уже запись, и если существует - просто информирует, что запись уже есть
- при удалении (из-за ограниченности API сервиса ISP) наличие записи не проверяется
- в обоих случаях проверяется логин\пароль и корневой домен, ошибки выводятся, и не проверяется сам сервер - тут уж пишите без ошибок и проверяется сервер на его работоспособность (может быть доведу до ума позже)

# Работа с домена с кластера k8s
так же, можно использоваться сразу в кластере, создавая доменное имя при поднятии деплоймента

вот так может выглядеть джоба:

```
# kubetpl:syntax:go-template

apiVersion: batch/v1
kind: Job
metadata:
  name: ispmgr-dns-job-lb1
spec:
  ttlSecondsAfterFinished: 300
  template:
    spec:
      containers:
      - name: ispmgr-dns-api-call
        image: vasyakrg/ispmgr-api
        command: ["bash", "-c", "./ispmgr.sh {{ .DNS_API_COMMAND }}"]
        env:
        - name: DNS_SERVER
          valueFrom:
            secretKeyRef:
              name: dns-service-secrets
              key: dns-server
        - name: DNS_LOGIN
          valueFrom:
            secretKeyRef:
              name: dns-service-secrets
              key: dns-login
        - name: DNS_PASSWORD
          valueFrom:
            secretKeyRef:
              name: dns-service-secrets
              key: dns-password
        - name: DOMAIN_NAME
          value: {{ .DOMAIN_NAME }}
        - name: DNS_SETNAME
          value: {{ .SITE_NAME }}
        - name: DNS_SETIP
          value: {{ .LB_IP1 }}

      restartPolicy: Never
  backoffLimit: 2
---
```

соответственно, передаем `DNS_API_COMMAND`, которая принимает вид `create` или `delete`, а так же `все` переменные, по примеру запуска таска в контейнере.

я обычно использую две такие джобы, которые создают две записи по айпишникам `LB_IP1` и `LB_IP2`, которые смотря на две ноды нашего кластера
а при удалении сервиса, запускаю две новые джобы с переменной DNS_API_COMMAND=delete, что бы не плодить в домене куча мертвых записей.

проверить, что джоба на удаление отработалась можно простым методом ожидания
kubectl -n ${NS} wait --for=condition=complete --timeout=600s jobs/ispmgr-dns-job-lb1
после чего можно удалять весь деплой, либо же весь неймспейс полностью.