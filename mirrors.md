---
title: Наши зеркала
description: 
published: true
date: 2024-10-16T09:57:04.035Z
tags: mirrors
editor: markdown
dateCreated: 2022-09-05T15:48:26.156Z
---

# Доступные зеркала

## Docker-hub

🤦Since Docker is a US company, we must comply with US export control regulations.
Доброе утро, Docker-hub 🙂

```
'docker login': denied: <html><body><h1>403 Forbidden</h1>
```

🚀 PS. Решение проблемы такое: в `/etc/docker/daemon.json` (если его нет, создаем) добавляем:

```
{
    "registry-mirrors": ["https://mirror.gcr.io", "https://daocloud.io", "https://c.163.com/", "https://registry.docker-cn.com"]
}
```

перезапускаем сервис: `systemctl reload docker`

## Hashicorp

качаешь ключ
`curl -fsSL https://apt.amegahost.kz/apt.releases.hashicorp.com/gpg | apt-key add -`

потом добавляешь репу
`echo "deb https://apt.amegahost.kz/apt.releases.hashicorp.com/ jammy main" > /etc/apt/sources.list.d/vault.list`

ну и `apt update && apt install vault`

## Grafana Labs

качаешь ключ
`wget -q -O - https://apt.amegahost.kz/apt.grafana.com/gpg.key | gpg --dearmor | tee /etc/apt/keyrings/grafana.gpg > /dev/null`

добавляешь репу
`echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.amegahost.kz/apt.grafana.com/ stable main" > /etc/apt/sources.list.d/grafana.list`

## Elastic co

качаешь ключ
`wget -q -O - https://apt.amegahost.kz/artifacts.elastic.co/elasticsearch-keyring.gpg | gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg > /dev/null`

добавляешь репу
`echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://apt.amegahost.kz/artifacts.elastic.co/packages/8.x/apt stable main" > /etc/apt/sources.list.d/elasticsearch.list`