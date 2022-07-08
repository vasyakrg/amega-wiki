---
title: Kafka
description: Kafka твики
published: true
date: 2022-07-08T07:28:01.718Z
tags: kafka
editor: markdown
dateCreated: 2022-07-08T07:28:01.718Z
---

# Kafka работа с топиками

> Общее
{.is-success}

### посмотреть настройки брокеров
`bin/kafka-configs.sh --bootstrap-server kafka-kafka-brokers:9092 --describe --entity-type brokers --entity-name 0`

### просмотреть список топиков (в контейнере zookeeper)
`bin/kafka-topics.sh --bootstrap-server kafka-kafka-brokers:9092 --list` 

> Диагностика
{.is-success}


### вывести у кого не хватает ISR
`bin/kafka-topics.sh --bootstrap-server kafka-kafka-brokers:9092 --under-min-isr-partitions --describe`

### вывести у кого не хватает партиций (если 0 - скорее всего старые и пустые)
`bin/kafka-topics.sh --bootstrap-server kafka-kafka-brokers:9092 --replicated-partitions —list`

### просмотреть очередь консьюмеров в группе
`bin/kafka-consumer-groups.sh --bootstrap-server kafka-kafka-brokers:9092 --group theclub-comsumer-id --describe`

> Работа с топиком
{.is-success}


### получить данные по топику
`bin/kafka-topics.sh --bootstrap-server kafka-kafka-brokers:9092 --describe --topic <topic_name>`

### прочитать содержимое топика
`bin/kafka-console-consumer.sh --bootstrap-server kafka-kafka-brokers:9092 --topic <topic_name> --from-beginning`

### удалить топик
`bin/kafka-topics.sh --bootstrap-server kafka-kafka-brokers:9092 --delete --topic <topic_name>`

### для удаления, перед процедурой, надо выставить в server.properties свойство
`delete.topic.enable=true`
на strimzi-operator включено везде по дефолту = 1

### получить начальный офсет
`bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list kafka-kafka-brokers:9092 --topic top-lak-source-topic --time -2`

### получить конечный офсет
`bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list kafka-kafka-brokers:9092 --topic top-lak-source-topic --time -1`

### нагенерить пустых записей с цифрами в топик
`/usr/bin/shuf -i 1-1000 -n 30000 | tee -a /tmp/kafka-random-data | bin/kafka-console-producer.sh --bootstrap-server=kafka-kafka-brokers:9092 --topic top-lak-source-topic` 
и вводить..

### либо же нагенерить башем
`for i in {1..10};do echo $i | bin/kafka-console-producer.sh --bootstrap-server=kafka-kafka-brokers:9092 --topic top-lak-source-topic; done;`

### создать сообщение
`bin/kafka-console-producer.sh --broker-list kafka-kafka-brokers:9092 --topic top-lak-source-topic`

### ввалить данные из файла в топик
`cat /tmp/cat.txt | bin/kafka-console-producer.sh --broker-list kafka-kafka-brokers:9092 --topic top-lak-source-topic`

### сдвинуть офсет на нужное число (shift-by)
`bin/kafka-consumer-groups.sh --bootstrap-server kafka-kafka-brokers:9092 --group top-lak-comsumer-id --reset-offsets --shift-by 67437 --topic top-lak-source-topic --execute`

> Переезд топиков
{.is-success}

### между топиками
`kafkacat -b localhost:9092 -C -t source-topic -K: -e -o beginning | kafkacat -b localhost:9092 -P -t target-topic -K: `


### между кластерами
`kafkacat -b localhost:9092 -C -t source-topic -K: -e -o beginning | kafkacat -b localhost:9093 -P -t source-topic -K: `

> обслуживание кластера
{.is-success}

### лечим большое латести у зукипера пересозданием реплики:
`kubectl annotate pod kafka-zookeeper-2 strimzi.io/delete-pod-and-pvc=true -n kafka`