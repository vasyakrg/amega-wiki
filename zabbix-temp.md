---
title: Zabbix - температура
description: 
published: true
date: 2021-02-18T12:21:08.491Z
tags: zabbix, sensors, temperature
editor: markdown
dateCreated: 2021-02-18T12:21:08.491Z
---

# Мониторим температуру на гипервизорах

Имеем Proxmox гиперы с debian9\10 и внешний забикс
Задача - снимать в том числе показания средней температуры проца

Со стороны гипперов просаживаем сенсоры:

```
apt install lm-sensors
```

проверяем что возвращаются данные:
```
sensors
```

видим:
```
root@pve1:~# sensors
coretemp-isa-0000
Adapter: ISA adapter
Package id 0:  +33.0°C  (high = +80.0°C, crit = +100.0°C)
Core 0:        +31.0°C  (high = +80.0°C, crit = +100.0°C)
Core 1:        +32.0°C  (high = +80.0°C, crit = +100.0°C)
```

прямой наводкой, берем значение температуры:
```
sensors | grep Core | awk -F'[:+°]' '{avg+=$3}END{print avg/NR}'
```

должна вернуться цифра - например `35`

правим конфиг забикс агента (**он ведь уже стоял на гипперах**?)

```
nano /etc/zabbix/zabbix_agentd.conf
```

ищем необходимый параметр и правим на еденицу
`UnsafeUserParameters=1`

ниже прописываем переменную:
`UserParameter=pve-t.core0,sensors coretemp-isa-00000 | awk -F'[:+°]' '{if(max==""){max=$3}; if(max<$3) {max=$3};} END {print max}'`

выходим, перезапускаем агента:
```
systemctl restart zabbix-agent
```

# Со стороны zabbix

> Создаем элемент данных
{.is-success}

Открываем необходимый узел и перейдя в `Элементы данных` добавляем новый нажав `Создать элемент данных`.

Необходимо заполнить следующие поля:

- Имя — core0 Temperature;
- Ключ — pve-t.core0;
- Тип информации — Числовой (с плавающей точкой);
- Интервал обновления — 1m;
- Период хранения истории — 1w;
- Группы элементов данных — CPU.

![zabbix_temperature_sevo44-1.jpg](/zabbix/zabbix_temperature_sevo44-1.jpg)

> Добавление тригера
{.is-success}

Открываем необходимый узел и перейдя в `Тригеры` добавляем новый нажав `Создать тригер`.

Необходимо заполнить следующие поля:

- Имя тригера — pve-t core0 Temperature;
- Выражение — {pve-t:pve-t.core0.last()}>80.

![zabbix_temperature_sevo44-2.jpg](/zabbix/zabbix_temperature_sevo44-2.jpg)

Выражение формируется на вкладке открывающейся по кнопке `Добавить` рядом с полем `Выражение`.

![zabbix_temperature_sevo44-3.jpg](/zabbix/zabbix_temperature_sevo44-3.jpg)

> Добавление графика
{.is-success}


Открываем необходимый узел и перейдя в «Графики» добавляем новый нажав «Создать график«.

Какое количество графиков и настройки параметров отображения решите сами. 
на картинке мониторится не только CPU, но и диски и память (если сенсоры это возвращают)

![zabbix_temperature_sevo44-4.jpg](/zabbix/zabbix_temperature_sevo44-4.jpg)

По нажатию кнопки `Добавить` в параметре `Элемент данных` выбираем все необходимые элементы данных для отображения на графике.

> В результате график имеет следующий вид
{.is-success}

![zabbix_панель_2021-02-18_19-20-08.jpg](/zabbix/zabbix_панель_2021-02-18_19-20-08.jpg)

Подсмотрено (С) [тут](https://sevo44.ru/monitoring-temperatury-v-zabbix/)