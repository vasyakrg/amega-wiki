---
title: Уведомления от ssh в telegram
description: При входе на сервер по ssh, админу отправляется уведомление в телегу
published: true
date: 2021-03-15T11:21:42.087Z
tags: telegram, ssh, server, notify
editor: markdown
dateCreated: 2021-03-15T11:19:32.654Z
---

# Уведомления админа при входе по ssh на сервер

- @BotFather - подскажет как работать с ботами
- @myidbot - подскажет личный ID админа


проверить можно дернув ссылку:
нужно будет указать свой `id`, в примере это: 999
`api_token`: 1111:ABC-DEF1234g

```bash
https://api.telegram.org/bot1111:ABC-DEF1234g/sendMessage?chat_id=999&text=Работает?
```

Нужно создать скрипт в папке profile.d.
Скрипт назове ssh-to-telegram, он будет срабатывать с каждым входом в систему по SSH.
Пишем в терминале сервера Unraid:
`nano /etc/profile.d/ssh-to-telegram.sh`

- USERID="999"
- KEY="1111:ABC-DEF1234g"

```bash
USERID="999"
KEY="1111:ABC-DEF1234g"
TIMEOUT="10"
URL="https://api.telegram.org/bot$KEY/sendMessage"
DATE_EXEC="$(date "+%d %B %Y %H:%M")"
if [ -n "$SSH_CLIENT" ]; then
        IP=$(awk '{print $1}' <<< $SSH_CLIENT)
        PORT=$(awk '{print $3}' <<< $SSH_CLIENT)
        HOSTNAME=$(hostname -f)
        IPADDR=$(hostname -I | awk '{print $1}')
        TEXT="$DATE_EXEC
        Вход пользователя ${USER} по ssh на $HOSTNAME ($IPADDR)
        С $IP через порт $PORT"
        curl -s --max-time $TIMEOUT -d "chat_id=$USERID&disable_web_page_preview=1&text=$TEXT" $URL > /dev/null
fi
```

созраняем, выходим, проверяем, еще раз заходя в новую сессию