---
title: Rocketchat-LDAP
description: 
published: true
date: 2021-10-10T03:58:24.817Z
tags: 
editor: markdown
dateCreated: 2021-10-10T03:57:42.973Z
---

# Rocket-chat

> Удаление всех LDAP юзеров с базы
{.is-success}


идем в контейнер slack_db
`docker exec -it slack_db /bin/bash`

запускаем
`mongo`

в ней переключаемся на коллекцию
`use rocketchat;`

и грохаем юзеров
`db.users.remove({"ldap": true});`

> Настройки LDAP со стороны RocketChat
{.is-success}


Юзеры лежат в `Domain > Users`

* `Base DN`=`CN=Users,DC=domain,DC=com`
* `User DN`=`cn=slack,ou=systems,dc=domain,dc=com` (юзер `slack` лежит в отдельной OU Systems)
* `Pass`=password
* Поле "Имя пользователя" = `sAMAccountName`
* Поле уникального идентификатора = `objectGUID,ibm-entryUUID,GUID,dominoUNID,nsuniqueId,uidNumber`
* Домен по умолчанию = `domain.com`
* Карта пользовательских данных = `{"displayName":"name", "mail":"email"}`
* Фильтр Группы Пользователей = `(&(sAMAccountName=#{username})(memberOf=CN=#{groupName},OU=rocket,DC=domain,DC=com))`
* LDAP Группы BaseDN = `DC=domain,DC=com`
* Фильтр = `(objectclass=user)`
* Область = `sub`
* Поле поиска = `sAMAccountName`