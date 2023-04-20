---
title: Proxmox твики
description: всяко разно про проксмокс
published: true
date: 2023-04-20T03:31:20.085Z
tags: proxmox, vm
editor: markdown
dateCreated: 2021-03-28T07:04:09.088Z
---

# Отправка уведомлений через mailgun

ставим зависимости
`apt install libsasl2-modules mailutils `

генерим в [mailgun](https://app.mailgun.com/app/sending/domains) отдельный почтовый ящик и через эхо прописываем настройки в файл
`echo "smtp.eu.mailgun.org <email@domain.ru>:<password>" > /etc/postfix/sasl_passwd`

пакуем
`postmap hash:/etc/postfix/sasl_passwd`
`chmod 600 /etc/postfix/sasl_passwd`

дописываем в конец файла
`nano /etc/postfix/main.cf`

настройки маилгана  
```bash
relayhost = smtp.eu.mailgun.org:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_security_options =
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_tls_CAfile = /etc/ssl/certs/Entrust_Root_Certification_Authority.pem
smtp_tls_session_cache_database = btree:/var/lib/postfix/smtp_tls_session_cache
smtp_tls_session_cache_timeout = 3600s
```

перезапускаем почту
`postfix reload`

тестируем
`echo "test message" | mail -s "test subject" youremail@gmail.com`

если почта пришла, ок.

> теперь самое важное
{.is-warning}

идем в настройки вашего пользователя в Proxmox, у меня это root и задаем ему ваш правильный email, у меня он почему-то оказался пустой, при том, что я его задавал при установке.

![2022-05-03_14-59-02(1).png](/2022-05-03_14-59-02(1).png)

теперь любое событие (например smartd который тестируем диски) будет приходит во время

# Темная тема

> не актуально, т.к. завезли в 7.4 версии
{.is-danger}


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
