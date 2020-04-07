---
title: Браузеры: Safari
description: тюнинг и фишки
published: true
date: 2020-04-07T01:20:16.152Z
tags: 
---

# Проблемы
> Переадресация http:// > https://
{.is-success}

Eсли браузер запомнил, что вы зашли через https://, а теперь вам надо снова http.
Решение не элегантное, т.к. очищается весь список про все сайты.
Но чиска кеша, удаление куков конкретного сайта не помогает, делаем через терминал:

`sudo killall nsurlstoraged`
`rm -f ~/Library/Cookies/HSTS.plist`
`launchctl start /System/Library/LaunchAgents/com.apple.nsurlstoraged.plist`


# Плагины
> adblock
{.is-success}

реально чистит страницы от мусора и рекламы

> Ployglot
{.is-success}


при выделение текста на странице (при условии что это простой текст) - происходит автоперед на настроенный язык. У меня с ENG в RU, например.