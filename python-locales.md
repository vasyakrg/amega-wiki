---
title: Русский язык в Python + Docker
description: Как установить русскую локаль в докере для питона
published: true
date: 2022-11-25T02:56:10.105Z
tags: docker, python
editor: markdown
dateCreated: 2022-11-25T02:56:10.105Z
---

# Русская локаль в питоне в докере

Небольшая заметка про питон и русский язык в докере.

Просто прописать в коде - толку нет:

`locale.setlocale(locale.LC_TIME, ('ru_RU', 'UTF-8'))`

Один фиг в контейнере на проде день рисуется как *Wednesday*.

Необходимо просадить в контейнер также musl-locales, ну и собрать его по ходу дела.

```dockerfile
ARG DOCKER_BASEIMAGE
FROM ${DOCKER_BASEIMAGE}

ARG DOCKER_MAINTAINER
LABEL maintainer="${DOCKER_MAINTAINER:-vasyakrg@gmail.com}"

ENV MUSL_LOCALE_DEPS cmake make musl-dev gcc gettext-dev libintl
ENV MUSL_LOCPATH /usr/share/i18n/locales/musl

RUN apk add --no-cache $MUSL_LOCALE_DEPS python3 python3-dev && apk add cmd:pip3 && pip3 install --upgrade pip \
    && wget https://gitlab.com/rilian-la-te/musl-locales/-/archive/master/musl-locales-master.zip \
    && unzip musl-locales-master.zip \
      && cd musl-locales-master \
      && cmake -DLOCALE_PROFILE=OFF -D CMAKE_INSTALL_PREFIX:PATH=/usr . && make && make install \
      && cd .. && rm -r musl-locales-master

ENV PYTHONIOENCODING=utf-8

WORKDIR /iqworkbot
COPY app/ .

RUN pip3 --no-cache-dir install -r requirements.txt

CMD [ "/usr/bin/python3", "./start.py" ]

```