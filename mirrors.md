---
title: Наши зеркала
description: 
published: true
date: 2024-04-08T04:47:45.574Z
tags: mirrors
editor: markdown
dateCreated: 2022-09-05T15:48:26.156Z
---

# Доступные зеркала

## Hashicorp

качаешь ключ
`curl -fsSL https://apt.amegahost.kz/apt.releases.hashicorp.com/gpg | apt-key add -`

потом добавляешь репу
`echo "deb https://apt.amegahost.kz/apt.releases.hashicorp.com/ jammy main" > /etc/apt/sources.list.d/vault.list`

ну и `apt update && apt install vault`

## Grafana Labs

добавляешь репу
`echo "deb https://apt.amegahost.kz/apt.grafana.com/ stable main" > /etc/apt/sources.list.d/grafana.list`