---
title: Zalando возвращаем мастер к жизни
description: 
published: true
date: 2024-09-28T03:56:39.577Z
tags: k8s, psql, zalando
editor: markdown
dateCreated: 2024-09-28T03:48:14.444Z
---

# Zalando-operator возвращаем мастер к жизни

Кластер рассыпался, лидера нет, оба пытаются стартовать и забутстрапится с лидера.

Суть: был себе кластер patroni из двух реплик, собранный через Zalando Operator и живущий в k8s.
ноды потрясло, поды подвисли, админ запустил reinit реплики с пода-мастера, но он зафейлился, т.к. мастер тоже был в read-only. Далее админ перезапустил под мастера, после чего последний написал, что мастера в кластере нет и он запускаться отказывается.

Итог: реплика в режиме бутстрапа ждет лидера, лидер не стартует, т.к. lock: none.

> Действия
{.is-success}


1. выбрать правильного "бывшего" лидера из подов, проверить, что в `/home/postgres/pgdata/pgroot/data` реально есть данные.
2. заходим в его под и выполняем по очереди:

> patronictl pause
> sv stop patroni
> su -- postgres
> rm standby.signal
> pg_resetwal -D /home/postgres/pgdata/pgroot/data -f
> pg_ctl start -D /home/postgres/pgdata/pgroot/data
> exit
> patronictl resume
> sv start patroni
{.is-info}


 
3. после успешного запуска (проверяем это в логе пода) оператор обязан повесить на под лейбл `spilo-role=master`
4. далее, проваливаемся в него и проверяем что остальные поды забутстрапились. если тупят - реинитим их по очереди, дожидаясь что они так же встали в строй.

> команды в помощь
{.is-success}


`patronictl list`
`patronictl reinit <cluster_name> <replica_name>`

