## Лабораторная работа - Настройка и проверка расширенных списков контроля доступа.

### Топология
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab11/files/topo1.png?raw=true)

##
### Таблица адресации

|Устройство|	Интерфейс|	IP-адрес	|Маска подсети| 	Шлюз по умолчанию|
| ------------- |:------------------:|------------- |------------- |------------- |
|R1	|G0/0/1	|—	|—|	—|
|R1	|G0/0/1.20|	10.20.0.1	|255.255.255.0|	—|
|R1|	G0/0/1.30|	10.30.0.1|	255.255.255.0|	—|
|R1	|G0/0/1.40|	10.40.0.1|	255.255.255.0|	—|
|R1	|G0/0/1.1000	|—	|—|	—|
|R1|	Loopback1|	172.16.1.1|255.255.255.0|	—|
|R2	|G0/0/1	|10.20.0.4|	255.255.255.0|	—|
|S1	|VLAN 20	|10.20.0.2	|255.255.255.0|	10.20.0.1|
|S2|	VLAN 20|	10.20.0.3|	255.255.255.0|	10.20.0.1|
|PC-A|	NIC|	10.30.0.10|	255.255.255.0	|10.30.0.1|
|PC-B|	NIC	|10.40.0.10	|255.255.255.0	|10.40.0.1|

##
### Таблица VLAN

|VLAN	|Имя	|Назначенный интерфейс|
| ------------- |:------------------:|------------- |
|20	|Management	|S2: F0/5|
|30|	Operations	|S1: F0/6|
|40	|Sales	|S2: F0/18|
|999	|ParkingLot	|S1: F0/2-4, F0/7-24, G0/1-2|
|999	|ParkingLot	|S2: F0/2-4, F0/6-17, F0/19-24, G0/1-2|
|1000|	Собственная|	—|

## 
### Часть 1. Создание сети и настройка основных параметров устройства.
#### Шаг 1. Создайте сеть согласно топологии.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab11/files/topo2.jpg?raw=true)

##
#### Шаг 2. Произведите базовую настройку маршрутизаторов.
```
hostname R2
no ip domain-lookup
enable secret 0 class
line con 0
password cisco
login
exit
line vty 0 15
password cisco
login
exit
service password-encryption
banner motd ^C!!!!!!!!!!!!enter only for MA, no MAX or MAXIM!!!!!!^C
clock timezone EKB 5 0
ip domain-name home.local 
crypto key generate rsa general-keys modulus 2048 
ip ssh version 2 
username admin secret 5 $1$mERr$qJb.eHvBN7S590aq.dpRL. 
line vty 0 4 
transport input ssh 
login local 
exit
exit
clock set 21:00:00 24 march 2026
copy running-config startup-config
```
#### Шаг 3. Настройте базовые параметры каждого коммутатора.
Настроено по аналогии с маршрутизаторами

## 
### Часть 2. Настройка сетей VLAN на коммутаторах.
##### Шаг 1. Создайте сети VLAN на коммутаторах.
> a.	Создайте необходимые VLAN и назовите их на каждом коммутаторе из приведенной выше таблицы.

