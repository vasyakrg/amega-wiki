---
title: Ubiquiti - починка cloudkey
description: Как починить cloudkey в связке с роутером и прописать правильные хосты
published: true
date: 2020-03-13T09:47:47.870Z
tags: cloudkey, ubiquiti
---

# Глава 1.                     
---

                                         Болезнь




Не получается зайти на 150.9 (manager), пользователи жалуются коровку что не работают ваучеры. 



# Глава 2.                     
---

                                          Осмотр

Приходим на коворк никого не слушая, идем в серверную.
С собой надо иметь Ethernet hub  и ноутбук
Видим за первым шкафом 


###  Шлюз ![шлюз.png](/шлюз.png)


                                        и




###  Cloud key ![keyy.png](/keyy.png)

 Он запитан через POE блок ![poe.png](/poe.png)

Глава 3
---

                                               Лечение


Берем скрепку и сбрасываем обе железки ( имеются ПОДПИСАННЫЕ кнопки «reset» )
Выдергиваем WAN и LAN из Шлюза 
Выдергиваем LAN из POE блока
Объединяем LAN Шлюза и Lan Cloud Key в hub

Тут мы видим, что: Lan Шлюза, ноутбук, Cloud Key подключены в  hub.
![общ_сис.png](/общ_сис.png)

# ставим софт

                          

Качаем программу с сайта.  https://www.ui.com/download/unifi 
![ссыль_проги.png](/ссыль_проги.png)

Скачали. Поставили. Заходим. Если всё сбросили и подключили правильно, то видим окно с активной кнопкой. 
![кнопка.png](/кнопка.png)


При подключении все получают 1ю сетку.
Заходим на 192.168.1.1 (Шлюз) видим список устройств. 

После того как нашли наш Cloud Key надо зайти в менеджер и видим 
![настр.png](/настр.png)
Нажимаем «восстановить из предыдущей резервной копии» и выбираем с ноута бэкап старой система, которая работала. Ждем минут 5-12 пока встанет. 

 Заходим на восстановленное железо. 
Заходим на шлюз. Заполняем поля WAN с файлика
![wan_с_файлика.png](/wan_с_файлика.png)

# Настройка WAN
 убираем DHCP во вкладке LAN прописываем сеть 150.1 
Подключаем Шлюз и cloud key обратно в систему. 
Ждем пока всё поднимается. после того всё поднимется проверяем в 150.9 в менеджере. 

после чего идем в SSH и крутим настройки для выхода окна ввода ваучера


# Настройка SSH

командная строка
ssh admin@192.168.150.1
pass Tp9JACMvTeaCgi 67 (возможно генерируется после сброса)
cat /etc/hosts - посмотреть файл
sudo vi /etc/hosts - открывает файл в редакторе vi (гореть им в аду)

чтобы перейти в режим редактирования жмем i
прописываем необходимые данные, например 192.168.150.9   wifi.iqwork.me
для завершения редактирования жмем ESC
команда :w - записать в файл, :wq - записать и выйти,  :q! - выйти

cat /etc/hosts - проверяем



