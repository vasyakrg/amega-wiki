---
title: Настройка почтовых клиентов
description: Imap - способ, который настоятельно рекомендуется использовать
published: true
date: 2020-01-06T14:37:21.362Z
tags: mail, mailserver, keriomailserver, kerio
---

# Настройка по протоколу IMAP
1. Запустите программу и нажмите в окне приветствия кнопку Пропустить это и использовать мою существующую почту.
2. В окне Настройка имеющейся у вас учётной записи электронной почты укажите следующие параметры учетной записи:
3. Ваше имя — имя пользователя (например, «Алиса Литл»);
4. Адрес эл. почты — ваш почтовый адрес (например, «mylogin@\<domain>»);
5. Пароль — ваш пароль, выданный ранее администраторами ГК Амега.


> В детальных настройках нужно указать следующее:
{.is-success}

Настройка производится вручную, указываются следующие параметры серверов электронной почты:
**Входящая почта**
Протокол — IMAP;
Имя сервера — mail.\<domain>
Порт — 993;
SSL — SSL/TLS;
Аутентификация — Обычный пароль (Plain text).

**Исходящая почта**
Имя сервера — mail.\<domain>
Порт — 465;
SSL — SSL/TLS;
Аутентификация — Обычный пароль (Plain text).

> Где \<domain> - ваш корпоративный домен, например gkamega.com
{.is-info}
