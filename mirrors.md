---
title: Наши зеркала
description: 
published: true
date: 2022-09-05T15:48:26.156Z
tags: mirrors
editor: markdown
dateCreated: 2022-09-05T15:48:26.156Z
---

# Доступные зеркала

## mongoBD

`deb https://apt.amegahost.kz/repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse`

## Hashicorp

качаешь ключ
`curl -fsSL https://apt.amegahost.kz/apt.releases.hashicorp.com/gpg | sudo apt-key add -`

потом добавляешь репу
`echo "deb https://apt.amegahost.kz/apt.releases.hashicorp.com/ focal main" > /etc/apt/sources.list.d/vault.list`

ну и `apt update && apt install vault`