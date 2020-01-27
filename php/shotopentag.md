---
title: PHP Parse error:  syntax error, unexpected end of file…
description: ошибка при переносе сайта с apache2 на nginx
published: true
date: 2020-01-27T03:31:26.338Z
tags: nginx, apache2, error, php, php-fpm
---

# Проблема
Столкнулся тут с проблемой переноса одного файла php с сервера apache2 на nginx
вылезает такое:

`PHP message: PHP Parse error:  syntax error, unexpected end of file ...`

указывая на последнюю строку файла.
Естественно код верный, но для проверки перенес файл на локальный docker php7.3-fpm - там работает, а вот на удаленном хостинге с php7.1-fpm - нет.

# Решение

Необходимо в файле php.ini поменять один единственный параметр - short_open_tag на On. Т.е. выглядеть это должно так:

> short_open_tag = On
{.is-success}


сам php.ini у вас находится там, от куда запускается php-fpm, посмотреть можно в настройках nginx в конфиге сайта.

в моем случае лежал тут - `/etc/php/7.1/fpm/php.ini`