Домашняя работа по теме "Архитектура сетей"

Задание:

Дано

https://github.com/erlong15/otus-linux/tree/network
(ветка network)
Vagrantfile с начальным построением сети

    inetRouter
    centralRouter
    centralServer
    тестировалось на virtualbox
    Планируемая архитектура
    построить следующую архитектуру
    Сеть office1
    192.168.2.0/26 - dev
    192.168.2.64/26 - test servers
    192.168.2.128/26 - managers
    192.168.2.192/26 - office hardware
    Сеть office2
    192.168.1.0/25 - dev
    192.168.1.128/26 - test servers
    192.168.1.192/26 - office hardware
    Сеть central
    192.168.0.0/28 - directors
    192.168.0.32/28 - office hardware
    192.168.0.64/26 - wifi
    ```
    Office1 ---\
    ----> Central --IRouter --> internet
    Office2----/
    ```
    Итого должны получится следующие сервера
    inetRouter
    centralRouter
    office1Router
    office2Router
    centralServer
    office1Server
    office2Server
    Теоретическая часть
    Найти свободные подсети
    Посчитать сколько узлов в каждой подсети, включая свободные
    Указать broadcast адрес для каждой подсети
    проверить нет ли ошибок при разбиении
    Практическая часть
    Соединить офисы в сеть согласно схеме и настроить роутинг
    Все сервера и роутеры должны ходить в инет черз inetRouter
    Все сервера должны видеть друг друга
    у всех новых серверов отключить дефолт на нат (eth0), который вагрант поднимает для связи
    при нехватке сетевых интервейсов добавить по несколько адресов на интерфейс

Ход работы:

Исходя из задания можно нарисовать следующую схему требующейся сети:

![Strukture of test stend](https://github.com/DmitryV81/HW18_Networking/blob/main/pictures/structure.jpg)

Выполним подсчет сетей, определим количество хостов в каждой подсети и broadcast address:


![IP Range of test stend](https://github.com/DmitryV81/HW18_Networking/blob/main/pictures/ip_range.JPG)


Далее автоматизируем процесс настройки стенда с помощью ansible и проверим результат.

1. ip r и вывод traceroute с сервера office1Server:
```
root@office1Server:~# ip r
default via 192.168.2.129 dev enp0s8 proto static
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15
10.0.2.2 dev enp0s3 proto dhcp scope link src 10.0.2.15 metric 100
192.168.2.128/26 dev enp0s8 proto kernel scope link src 192.168.2.130
192.168.50.0/24 dev enp0s19 proto kernel scope link src 192.168.50.21
root@office1Server:~# traceroute otus.ru
traceroute to otus.ru (188.114.99.236), 30 hops max, 60 byte packets
 1  _gateway (192.168.2.129)  0.844 ms  0.781 ms  0.747 ms
 2  192.168.255.9 (192.168.255.9)  1.097 ms  1.062 ms  1.020 ms
 3  192.168.255.1 (192.168.255.1)  1.960 ms  1.885 ms  1.843 ms
 4  * * *
 5  * * *
 6  * * *
 7  * * *
 8  * * *
 9  * 188.114.99.236 (188.114.99.236)  81.941 ms  81.862 ms

```
2. ip r и вывод traceroute с сервера office2Server
```
root@office2Server:~# ip r
default via 192.168.1.1 dev eth1
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15
192.168.1.0/25 dev eth1 proto kernel scope link src 192.168.1.2
192.168.50.0/24 dev eth2 proto kernel scope link src 192.168.50.31
root@office2Server:~# traceroute otus.ru
traceroute to otus.ru (188.114.98.236), 30 hops max, 60 byte packets
 1  _gateway (192.168.1.1)  0.794 ms  0.728 ms  0.689 ms
 2  192.168.255.5 (192.168.255.5)  1.700 ms  1.651 ms  1.418 ms
 3  192.168.255.1 (192.168.255.1)  1.860 ms  1.804 ms  1.758 ms
 4  * * *
 5  * * *
 6  * * *
 7  * * *
 8  * * *
 9  * * *
10  * * *
11  * * *
12  * * 188.114.98.236 (188.114.98.236)  73.585 ms

```
3. ip r и вывод traceroute с сервера centralServer
```
[root@centralServer ~]# ip r
default via 192.168.0.1 dev eth1 proto static metric 101
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100
192.168.0.0/28 dev eth1 proto kernel scope link src 192.168.0.2 metric 101
192.168.50.0/24 dev eth2 proto kernel scope link src 192.168.50.12 metric 102
[root@centralServer ~]# traceroute otus.ru
traceroute to otus.ru (188.114.99.236), 30 hops max, 60 byte packets
 1  gateway (192.168.0.1)  1.208 ms  1.105 ms  0.922 ms
 2  192.168.255.1 (192.168.255.1)  1.395 ms  1.367 ms  1.332 ms
 3  * * *
 4  * * *
 5  * * *
 6  87.236.154.202 (87.236.154.202)  76.187 ms  77.736 ms  77.655 ms
 7  * * *
 8  * * *
 9  * * *
10  * * *
11  * * *
12  * * *
13  * * 188.114.99.236 (188.114.99.236)  75.926 ms
```
