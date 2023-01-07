---
title: Swap файл на ZFS
description: 
published: true
date: 2023-01-07T04:42:24.161Z
tags: proxmox, swap
editor: markdown
dateCreated: 2023-01-07T04:42:24.161Z
---

# Подключаем swap-файл на ZFS в Proxmox


## To set swap on a zfs drive:

1.
zfs create -V 8G -b $(getconf PAGESIZE) -o logbias=throughput -o sync=always -o primarycache=metadata -o com.sun:auto-snapshot=false rpool/swap

2.
mkswap -f /dev/zvol/rpool/swap

3.
swapon /dev/zvol/rpool/swap

## добавляем в /etc/fstab

1.
/dev/zvol/rpool/swap none swap discard 0 0

2.
mount -a