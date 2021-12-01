---
title: Proxmox твики
description: всяко разно про проксмокс
published: true
date: 2021-12-01T17:01:12.624Z
tags: proxmox, vm
editor: markdown
dateCreated: 2021-03-28T07:04:09.088Z
---

# Proxmox твики

## Темная тема

![2021-03-28_14.02.54.jpg](/2021-03-28_14.02.54.jpg)

Установка:
```
wget https://raw.githubusercontent.com/Weilbyte/PVEDiscordDark/master/PVEDiscordDark.sh && bash PVEDiscordDark.sh install
```

Далее нужно добавить скрипт в apt хук, что бы при обновление пакетов тема не затиралась:

идем в /etc/apt/apt.conf.d
и в последнем файле (обычно это **99needrestart**) прописываем так:
```
# это нужно добавить

DPkg::Post-Invoke {"curl https://raw.githubusercontent.com/Weilbyte/PVEDiscordDark/master/PVEDiscordDark.sh -o PVEDiscordDark.sh && chmod +x PVEDiscordDark.sh && bash PVEDiscordDark.sh
 install; rm PVEDiscordDark.sh"; };

# это уже было тут
DPkg::Post-Invoke {"test -x /usr/lib/needrestart/apt-pinvoke && /usr/lib/needrestart/apt-pinvoke || true"; };
```