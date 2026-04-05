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
```
vlan 20
 name Management
vlan 30
 name Operations
vlan 40
 name Sales
vlan 999
 name ParkingLot
vlan 1000
 name NativeVlan
```

> b.	Настройте интерфейс управления и шлюз по умолчанию на каждом коммутаторе, используя информацию об IP-адресе в таблице адресации.
```
S1(config)#int vl 20
S1(config-if)#
%LINK-5-CHANGED: Interface Vlan20, changed state to up
S1(config-if)#ip address 10.20.0.2 255.255.255.0
S1(config-if)#ex
S1(config)#ip default-gateway 10.20.0.1

S2(config)#int vl 20
S2(config-if)#
%LINK-5-CHANGED: Interface Vlan20, changed state to up
S2(config-if)#ip address 10.20.0.3 255.255.255.0
S2(config-if)#ex
S2(config)#ip default-gateway 10.20.0.1
```

> c.	Назначьте все неиспользуемые порты коммутатора VLAN Parking Lot, настройте их для статического режима доступа и административно деактивируйте их.
```
S1(config)#int ran F0/2-4, F0/7-24, G0/1-2
S1(config-if-range)#switchport mo acc
S1(config-if-range)#sw acc vl 999
S1(config-if-range)#shut

S2(config)#int ran F0/2-4, F0/6-17, F0/19-24, G0/1-2
S2(config-if-range)#switchport mo acc
S2(config-if-range)#sw acc vl 999
S2(config-if-range)#shut
```
##
#### Шаг 2. Назначьте сети VLAN соответствующим интерфейсам коммутатора.
> a.	Назначьте используемые порты соответствующей VLAN (указанной в таблице VLAN выше) и настройте их для режима статического доступа.
```
S1(config)#int f0/5
S1(config-if)#sw mo acc
S1(config-if)#sw acc vl 30

S2(config)#int f0/5
S2(config-if)#sw mo acc
S2(config-if)#sw acc vl 20
S2(config-if)#int f0/18
S2(config-if)#sw mo acc
S2(config-if)#sw acc vl 40
```

> b.	Выполните команду __show vlan brief__, чтобы убедиться, что сети VLAN назначены правильным интерфейсам.
```
S1#sh vl br
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/6
20   Management                       active    
30   Operations                       active    Fa0/5
40   Sales                            active    
999  ParkingLot                       active    Fa0/2, Fa0/3, Fa0/4, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
1000 NativeVlan                       active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active

S2#sh vl br
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1
20   Management                       active    Fa0/5
30   Operations                       active    
40   Sales                            active    Fa0/18
999  ParkingLot                       active    Fa0/2, Fa0/3, Fa0/4, Fa0/6
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
1000 NativeVlan                       active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active
```

##
### Часть 3. Настройте транки (магистральные каналы).
#### Шаг 1. Вручную настройте магистральный интерфейс F0/1.
> a.	Измените режим порта коммутатора на интерфейсе F0/1, чтобы принудительно создать магистральную связь. Не забудьте сделать это на обоих коммутаторах. <br>
> b.	В рамках конфигурации транка установите для native vlan значение 1000 на обоих коммутаторах. При настройке двух интерфейсов для разных собственных VLAN сообщения об ошибках могут отображаться временно. <br>
> c.	В качестве другой части конфигурации транка укажите, что VLAN 20, 30, 40 и 1000 разрешены в транке. <br>
```
S1(config)#int f0/1
S1(config-if)#sw mo tr
S1(config-if)#sw trunk native vl 1000
S1(config-if)#sw trunk all vl 20,30,40,1000

S2(config)#int f0/1
S2(config-if)#sw mo tr
S2(config-if)#sw tr nati vl 1000
S2(config-if)#sw trunk all vl 20,30,40,1000
```

> d.	Выполните команду __show interfaces trunk__ для проверки портов магистрали, собственной VLAN и разрешенных VLAN через магистраль.
```
S1#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/1       20,30,40,1000

Port        Vlans allowed and active in management domain
Fa0/1       20,30,40,1000

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       20,30,40,1000
```
##
#### Шаг 2. Вручную настройте магистральный интерфейс F0/5 на коммутаторе S1.
> a.	Настройте интерфейс S1 F0/5 с теми же параметрами транка, что и F0/1. Это транк до маршрутизатора.
> b.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.
> c.	Используйте команду show interfaces trunk для проверки настроек транка.

```
S1(config)#int f0/5
S1(config-if)#sw mo tr
S1(config-if)#sw tr al vl 20,30,40,1000
S1(config-if)#sw tr na vl 1000

S1#sh int tr
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1000
Port        Vlans allowed on trunk
Fa0/1       20,30,40,1000
Port        Vlans allowed and active in management domain
Fa0/1       20,30,40,1000
Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       20,30,40,1000
```
Данные не изменились, т.к. интерфейс на соседнем маршрутизаторе выключен.

##
### Часть 4.	Настройте маршрутизацию.
#### Шаг 1. Настройка маршрутизации между сетями VLAN на R1.
> a.	Активируйте интерфейс G0/0/1 на маршрутизаторе.<br>
> b.	Настройте подинтерфейсы для каждой VLAN, как указано в таблице IP-адресации. Все подинтерфейсы используют инкапсуляцию 802.1Q. Убедитесь, что подинтерфейс для собственной VLAN не имеет назначенного IP-адреса. Включите описание для каждого подинтерфейса.<br>
> c.	Настройте интерфейс Loopback 1 на R1 с адресацией из приведенной выше таблицы.<br>
> d.	С помощью команды __show ip interface brief__ проверьте конфигурацию подынтерфейса.<br>

```
R1(config)#int g0/1.20
R1(config-subif)#encapsulation dot1Q 20
R1(config-subif)#desc Management
R1(config-subif)#ip add 10.20.0.1 255.255.255.0 
R1(config-subif)#ex
R1(config)#int g0/1.30
R1(config-subif)#encapsulation dot1Q 30
R1(config-subif)#desc Operations
R1(config-subif)#ip add 10.30.0.1 255.255.255.0
R1(config-subif)#ex
R1(config)#int g0/1.40
R1(config-subif)#encapsulation dot1Q 40
R1(config-subif)#ip add 10.40.0.1 255.255.255.0
R1(config-subif)#ex
R1(config)#int g0/1.1000
R1(config-subif)#desc NativeVlan
R1(config-subif)#encap dot 1000 native
R1(config-subif)#ex
R1(config)#int l1
R1(config-if)#ip add 172.16.1.1 255.255.255.0

R1#sh ip int br
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0     unassigned      YES unset  administratively down down 
GigabitEthernet0/1     unassigned      YES unset  up                    up 
GigabitEthernet0/1.20  10.20.0.1       YES manual up                    up 
GigabitEthernet0/1.30  10.30.0.1       YES manual up                    up 
GigabitEthernet0/1.40  10.40.0.1       YES manual up                    up 
GigabitEthernet0/1.1000unassigned      YES unset  up                    up 
Loopback1              172.16.1.1      YES manual up                    up 
Vlan1                  unassigned      YES unset  administratively down down
```
##
#### Шаг 2. Настройка интерфейса R2 g0/0/1 с использованием адреса из таблицы и маршрута по умолчанию с адресом следующего перехода 10.20.0.1
```
R2(config)#int g0/1
R2(config-if)#ip add 10.20.0.4 255.255.255.0
R2(config-if)#no shut
R2(config)#ip route 0.0.0.0 0.0.0.0 10.20.0.1
```
##
### Часть 5. Настройте удаленный доступ.
#### Шаг 1. Настройте все сетевые устройства для базовой поддержки SSH.
> a.	Создайте локального пользователя с именем пользователя SSHadmin и зашифрованным паролем $cisco123! <br>
> b.	Используйте ccna-lab.com в качестве доменного имени.<br>
> c.	Генерируйте криптоключи с помощью 1024 битного модуля.<br>
> d.	Настройте первые пять линий VTY на каждом устройстве, чтобы поддерживать только SSH-соединения и с локальной аутентификацией.<br>

На всех устройствах выполнен скрипт:
```
username SSHadmin privilege 15 password $cisco123!
ip domain-name ccna-lab.com
crypto key generate rsa 
yes
1024
line vty 0 4
transport input ssh
login local
```
##
#### Шаг 2. Включите защищенные веб-службы с проверкой подлинности на R1.
> a.	Включите сервер HTTPS на R1. <br>
> b.	Настройте R1 для проверки подлинности пользователей, пытающихся подключиться к веб-серверу.<br>
```
R1(config)#ip http secure-server
% Invalid input detected at '^' marker.
R1(config)#ip http authentication local
% Invalid input detected at '^' marker.
```
Т.к. в PacketTracer применение данных команд на маршрутизаторе невозможно, добавим сервер к коммутатору S1, назначив ему адрес 10.30.0.11/24 на порту Fa0/7 vlan 30.
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab11/files/topo3.jpg?raw=true)

##
### Часть 6. Проверка подключения.
#### Шаг 1. Настройте узлы ПК.
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab11/files/ip.jpg?raw=true)

#### Шаг 2. Выполните следующие тесты. Эхо запрос должен пройти успешно.

| От	| Протокол	| Назначение| 
| ------------- |:------------------:|------------- |
| PC-A	| Ping	| 10.40.0.10| 
| PC-A	| Ping	| 10.20.0.1| 
| PC-B	| Ping| 10.30.0.10| 
| PC-B	| Ping	| 10.20.0.1| 
| PC-B	| Ping	| 172.16.1.1| 
| PC-B| 	HTTPS	| 10.20.0.1| 
| PC-B	| HTTPS	| 172.16.1.1| 
| PC-B	| SSH| 	10.20.0.1| 
| PC-B	| SSH	| 172.16.1.1| 

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab11/files/ping1.jpg?raw=true)

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab11/files/ping%202.jpg?raw=true)

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab11/files/https1.jpg?raw=true)

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab11/files/ssh1.jpg?raw=true)

##
### Часть 7.	Настройка и проверка списков контроля доступа (ACL).
#### Шаг 1. Проанализируйте требования к сети и политике безопасности для планирования реализации ACL.
#### Шаг 2. Разработка и применение расширенных списков доступа, которые будут соответствовать требованиям политики безопасности.

__Политика1.__ Сеть Sales не может использовать SSH в сети Management (но в другие сети SSH разрешен). <br>
```
R1(config)#ip access-list extended Rule1
R1(config-ext-nacl)#remark No SHH in Management from sales
R1(config-ext-nacl)#10 deny tcp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 eq 22
R1(config-ext-nacl)#50 permit tcp 10.40.0.0 0.0.0.255 any eq 22
```
##
__Политика 2.__ Сеть Sales не имеет доступа к IP-адресам в сети Management с помощью любого веб-протокола (HTTP/HTTPS). Сеть Sales также не имеет доступа к интерфейсам R1 с помощью любого веб-протокола. Разрешён весь другой веб-трафик (обратите внимание — Сеть Sales  может получить доступ к интерфейсу Loopback 1 на R1).
```
Сеть Sales не имеет доступа к IP-адресам в сети Management с помощью любого веб-протокола

R1(config-ext-nacl)#20 deny tcp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 eq 80
R1(config-ext-nacl)#30 deny tcp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 eq 443


Сеть Sales также не имеет доступа к интерфейсам R1 с помощью любого веб-протокола

R1(config-ext-nacl)#33 deny tcp 10.40.0.0 0.0.0.255 host 10.20.0.1 eq 80
R1(config-ext-nacl)#35 deny tcp 10.40.0.0 0.0.0.255 host 10.20.0.1 eq 443
R1(config-ext-nacl)#37 deny tcp 10.40.0.0 0.0.0.255 host 10.30.0.1 eq 80
R1(config-ext-nacl)#38 deny tcp 10.40.0.0 0.0.0.255 host 10.30.0.1 eq 443
R1(config-ext-nacl)#39 deny tcp 10.40.0.0 0.0.0.255 host 10.40.0.1 eq 80
R1(config-ext-nacl)#40 deny tcp 10.40.0.0 0.0.0.255 host 10.40.0.1 eq 443

Разрешён весь другой веб-трафик (обратите внимание — Сеть Sales  может получить доступ к интерфейсу Loopback 1 на R1)
51 permit tcp 10.40.0.0 0.0.0.255 any
```
##
__Политика 3.__ Сеть Sales не может отправлять эхо-запросы ICMP в сети Operations или Management. Разрешены эхо-запросы ICMP к другим адресатам. 
```
R1(config-ext-nacl)#43 deny icmp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 echo 
R1(config-ext-nacl)#44 deny icmp 10.40.0.0 0.0.0.255 10.30.0.0 0.0.0.255 echo
R1(config-ext-nacl)#55 permit icmp 10.40.0.0 0.0.0.255 any 
```

В общем и целом, по первым трем политикам сформирован следующий ACL:
```
R1#sh acc
Extended IP access list Rule1
    10 deny tcp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 eq 22
    20 deny tcp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 eq www
    30 deny tcp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 eq 443
    33 deny tcp 10.40.0.0 0.0.0.255 host 10.20.0.1 eq www
    35 deny tcp 10.40.0.0 0.0.0.255 host 10.20.0.1 eq 443
    37 deny tcp 10.40.0.0 0.0.0.255 host 10.30.0.1 eq www
    38 deny tcp 10.40.0.0 0.0.0.255 host 10.30.0.1 eq 443
    39 deny tcp 10.40.0.0 0.0.0.255 host 10.40.0.1 eq www
    40 deny tcp 10.40.0.0 0.0.0.255 host 10.40.0.1 eq 443
    43 deny icmp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 echo
    44 deny icmp 10.40.0.0 0.0.0.255 10.30.0.0 0.0.0.255 echo
    50 permit tcp 10.40.0.0 0.0.0.255 any eq 22
    51 permit tcp 10.40.0.0 0.0.0.255 any
    55 permit icmp 10.40.0.0 0.0.0.255 any
```

Для того, чтобы политики отрабатывались, необходимо применить ACL на сабинтерфейс сети Sales:
```
R1(config)#int g0/1.40
R1(config-subif)#ip access-group Rule1 in
```
##
__Политика 4:__ Cеть Operations  не может отправлять ICMP эхозапросы в сеть Sales. Разрешены эхо-запросы ICMP к другим адресатам. 
```
R1(config)#ip access-list extended Operations
R1(config-ext-nacl)#10 deny icmp 10.30.0.0 0.0.0.255 10.40.0.0 0.0.0.255 echo 
R1(config-ext-nacl)#20 permit icmp 10.30.0.0 0.0.0.255 any echo

R1(config)#int g0/1.30
R1(config-subif)#ip access-group Operations in
```
##
#### Шаг 3. Убедитесь, что политики безопасности применяются развернутыми списками доступа.

 |Фото | От	 |Протокол	 |Назначение	 |Результат |
| ------------- |:------------------:|------------- |------------- |------------- |
 |1  |PC-A	 |Ping |	10.40.0.10 |	Сбой |
 |1 | PC-A |	Ping	 |10.20.0.1	 |Успех |
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab11/files/foto1.jpg?raw=true)


  |Фото | От	 |Протокол	 |Назначение	 |Результат |
| ------------- |:------------------:|------------- |------------- |------------- |
 |2 | PC-B |	Ping	 |10.30.0.10	 |Сбой |
 |2 | PC-B |	Ping |	10.20.0.1	 |Сбой |
 |2  |PC-B |	Ping	 |172.16.1.1 |	Успех |
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab11/files/foto2.jpg?raw=true)
