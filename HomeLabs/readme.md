## Лабораторная работа. Реализация DHCPv4.

#### Топология

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab08.1/files/topo.jpg?raw=true "Схема сети")

#### Таблица адресации

|Устройство|	Интерфейс	|IP-адрес	|Маска подсети	|Шлюз по умолчанию|
| ------------- |:------------------:|------------- |------------- |------------- |
|R1	|G0/0/0	|10.0.0.1	|255.255.255.252|	—|
|R1|	G0/0/1|	—|	—|	—|
|R1	|G0/0/1.100	|—	|—	|—|
|R1	|G0/0/1.200	|—	|—	|—|
|R1|	G0/0/1.1000	|—	|—	|—|
|R2|	G0/0	|10.0.0.2	|255.255.255.252	|—|
|R2|	G0/0/1			|—	|—	|—|
|S1	|VLAN 200			|—	|—	|—|	
|S2	|VLAN 1				|—	|—	|—|
|PC-A|	NIC	|DHCP	|DHCP|	DHCP|
|PC-B	|NIC	|DHCP	|DHCP|	DHCP|

#### Таблица VLAN

|VLAN|	Имя|	Назначенный интерфейс|
| ------------- |------------------|------------- |
|1|	Нет	|S2: F0/18|
|100	|Клиенты	|S1: F0/6 |
|200	|Управление	|S1: VLAN 200  |
|999	|Parking_Lot	|S1: F0/1-4, F0/7-24, G0/1-2|
|1000	|Собственная	|—|

##

### Задачи
#### Часть 1.	Создание сети и настройка основных параметров устройства
##### Шаг 1.	Создание схемы адресации

> a.	Одна подсеть «Подсеть A», поддерживающая 58 хостов (клиентская VLAN на R1).<br>
> Запишите первый IP-адрес в таблице адресации для R1 G0/0/1.100.

Подсеть, удовлетворяющая таким условиям, имеет префикс /26 (размер 64 хоста > 58).<br>
Запишите первый IP-адрес в таблице адресации для R1 G0/0/1.100 - вычислим первый адрес сети:<br>

__IPv4-адрес первого узла в этой подсети__ <br>
192.168.1.00 000001 /26<br>
192.168.1.1 /26<br>

__IPv4-адрес последнего узла в этой подсети__ <br>
192.168.1.00 111110 /26<br>
192.168.1.62 /26<br>

> b.	Одна подсеть «Подсеть B», поддерживающая 28 хостов (управляющая VLAN на R1). <br>
> Запишите первый IP-адрес в таблице адресации для R1 G0/0/1.200. Запишите второй IP-адрес в таблице адресов для S1 VLAN 200 и введите соответствующий шлюз по умолчанию.<br>

Префикс сети, удовлетворяющий условиям - /27 (размер 32 хоста > 28).<br>
Т.к. последний занятый адрес из предыдущего примера - 192.168.1.63 (включая широковещательный адрес), то подсеть для этого примера начинается с адреса __192.168.1.64__ <br>

__IPv4-адрес первого узла в этой подсети__<br>
192.168.1.010 00001 /27<br>
192.168.1.65 /27<br>

__IPv4-адрес второго узла в этой подсети__<br>
192.168.1.010 00010 /27<br>
192.168.1.66 /27<br>

__IPv4-адрес последнего узла в этой подсети__<br>
192.168.1.010 11110 /27<br>
192.168.1.94 /27<br>

> c.	Одна подсеть «Подсеть C», поддерживающая 12 узлов (клиентская сеть на R2).<br>
> Запишите первый IP-адрес в таблице адресации для R2 G0/0/1.<br>

Префикс сети, удовлетворяющий условиям - /28 (размер 16 хостов > 12).<br>
Т.к. последний занятый адрес из предыдущего примера - 192.168.1.95 (включая широковещательный адрес), то подсеть для этого примера начинается с адреса __192.168.1.96__<br>

__IPv4-адрес первого узла в этой подсети__<br>
192.168.1.0110 0001 /28<br>
192.168.1.97 /28<br>

__IPv4-адрес последнего узла в этой подсети__<br>
192.168.1.0110 1110 /28<br>
192.168.1.110 /28<br>

#### Итоговая таблица маршрутизации

|Устройство	|Интерфейс	|IP-адрес|Маска подсети	|Шлюз по умолчанию|
| ------------- |:------------------:|------------- |------------- |------------- |
|R1	|G0/0/0	|10.0.0.1	|255.255.255.252|	-|
|R1 |G0/0/1|	-	|	-| -|
|R1 |G0/0/1.100|	192.168.1.1|	255.255.255.192	|-|
|R1 |G0/0/1.200	|192.168.1.65	|255.255.255.224	|-|
|R1 |G0/0/1.1000|	|-		|-| -|
|R2	|G0/0/0	|10.0.0.2|	255.255.255.252	|-|
|R2| G0/0/1|	192.168.1.97	|255.255.255.240|	-|
|S1	|VLAN 200	|192.168.1.66	|255.255.255.224	|192.168.1.65|
|S2|	VLAN 1|	192.168.1.98	|255.255.255.240	|192.168.1.97|
|PC-A|	NIC|	DHCP	|DHCP	|DHCP|
|PC-B|	NIC	|DHCP|	DHCP	|DHCP|

##

##### Шаг 2.	Создайте сеть согласно топологии.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab08.1/files/topo1.jpg?raw=true)

##

##### Шаг 3.	Произведите базовую настройку маршрутизаторов.


```
hostname R1
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
clock set 15:00:00 22 feb 2026
copy running-config startup-config
```

##

##### Шаг 4.	Настройка маршрутизации между сетями VLAN на маршрутизаторе R1

> a.	Активируйте интерфейс G0/0/1 на маршрутизаторе.<br>
> b.	Настройте подинтерфейсы для каждой VLAN в соответствии с требованиями таблицы IP-адресации. Все субинтерфейсы используют инкапсуляцию 802.1Q и назначаются первый полезный адрес из вычисленного пула IP-адресов. Убедитесь, что подинтерфейсу для native VLAN не назначен IP-адрес. Включите описание для каждого подинтерфейса.<br>

```
R1(config)#int g0/1
R1(config-if)#no shut
R1(config)#int g0/1.100
R1(config-subif)#encapsulation dot1Q 100
R1(config-subif)#ip address 192.168.1.1 255.255.255.192
R1(config-subif)#description Clients
R1(config-subif)#ex

R1(config)#int g0/1.200
R1(config-subif)#encapsulation dot1Q 200
R1(config-subif)#ip address 192.168.1.65 255.255.255.224
R1(config-subif)#description MGMNT
R1(config-subif)#ex

R1(config)#int g0/1.1000
R1(config-subif)#encapsulation dot1Q 1000 native
R1(config-subif)#description NativeVLAN
```

> c.	Убедитесь, что вспомогательные интерфейсы работают.<br>

```
R1#sh int g0/1.100
GigabitEthernet0/1.100 is up, line protocol is up (connected)
  Hardware is PQUICC_FEC, address is 0060.4761.9302 (bia 0060.4761.9302)
  Internet address is 192.168.1.1/26
  MTU 1500 bytes, BW 100000 Kbit, DLY 100 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation 802.1Q Virtual LAN, Vlan ID 100
  ARP type: ARPA, ARP Timeout 04:00:00, 
  Last clearing of "show interface" counters never

R1#sh int g0/1.200
GigabitEthernet0/1.200 is up, line protocol is up (connected)
  Hardware is PQUICC_FEC, address is 0060.4761.9302 (bia 0060.4761.9302)
  Internet address is 192.168.1.65/27
  MTU 1500 bytes, BW 100000 Kbit, DLY 100 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation 802.1Q Virtual LAN, Vlan ID 200
  ARP type: ARPA, ARP Timeout 04:00:00, 
  Last clearing of "show interface" counters never

R1#sh int g0/1.1000
GigabitEthernet0/1.1000 is up, line protocol is up (connected)
  Hardware is PQUICC_FEC, address is 0060.4761.9302 (bia 0060.4761.9302)
  MTU 1500 bytes, BW 100000 Kbit, DLY 100 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation 802.1Q Virtual LAN, Vlan ID 1000
  ARP type: ARPA, ARP Timeout 04:00:00, 
  Last clearing of "show interface" counters never
```

##

##### Шаг 5.	Настройте G0/1 на R2, затем G0/0/0 и статическую маршрутизацию для обоих маршрутизаторов

> a.	Настройте G0/0/1 на R2 с первым IP-адресом подсети C, рассчитанным ранее.<br>

```
R2(config)#int g0/1
R2(config-if)#ip add 192.168.1.97 255.255.255.240
R2(config-if)#no shut
```

> b.	Настройте интерфейс G0/0/0 для каждого маршрутизатора на основе приведенной выше таблицы IP-адресации.<br>

```
R1(config)#int g0/0
R1(config-if)#ip add 10.0.0.1 255.255.255.252
R1(config-if)#no shut

R2(config)#int g0/0
R2(config-if)#ip add 10.0.0.2 255.255.255.252
R2(config-if)#no shut
```

> c.	Настройте маршрут по умолчанию на каждом маршрутизаторе, указываемом на IP-адрес G0/0/0 на другом маршрутизаторе.<br>

```
R1(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.2
R2(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.1
```

> d.	Убедитесь, что статическая маршрутизация работает с помощью пинга до адреса G0/0/1 R2 от R1.<br>

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab08.1/files/Ping01.jpg?raw=true)

e.	Сохранил текущую конфигурацию в файл загрузочной конфигурации.

##

##### Шаг 6.	Настройте базовые параметры каждого коммутатора.

Оба коммутатора настроены, согласно скрипту ниже, меняется лишь hostname и время:

```
hostname S2
no ip domain-lookup 
enable secret 0 class
line con 0
password cisco
logging synchronous
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
line vty 0 15
transport input ssh
login local
end
clock set 10:00:00 23 feb 2026
copy running-config startup-config
```

##

##### Шаг 7.	Создайте сети VLAN на коммутаторе S1.

> a.	Создайте необходимые VLAN на коммутаторе 1 и присвойте им имена из приведенной выше таблицы.
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#vlan 100
S1(config-vlan)#name Clients
S1(config-vlan)#vlan 200
S1(config-vlan)#name MGMNT
S1(config-vlan)#vlan 999
S1(config-vlan)#name Parking_Lot
S1(config-vlan)#vlan 1000
S1(config-vlan)#name NativeVLAN
```

> b.	Настройте и активируйте интерфейс управления на S1 (VLAN 200), используя второй IP-адрес из подсети, рассчитанный ранее. Кроме того установите шлюз по умолчанию на S1.

```
S1(config)#int vl 200
S1(config-if)#ip add 192.168.1.66 255.255.255.224
S1(config-if)#ex
S1(config)#ip default-gateway 192.168.1.65
```

> c.	Настройте и активируйте интерфейс управления на S2 (VLAN 1), используя второй IP-адрес из подсети, рассчитанный ранее. Кроме того, установите шлюз по умолчанию на S2

```
S2(config)#int vl 1
S2(config-if)#ip add 192.168.1.98 255.255.255.240
S2(config-if)#no shut
S2(config-if)#ex
S2(config)#ip default-gateway 192.168.1.97
```

> d.	Назначьте все неиспользуемые порты S1 VLAN Parking_Lot, настройте их для статического режима доступа и административно деактивируйте их. На S2 административно деактивируйте все неиспользуемые порты.

```
S1(config)#int ra f0/1-4, f0/7-24, g0/1-2
S1(config-if-range)#sw mode acc
S1(config-if-range)#sw acc vl 999
S1(config-if-range)#shut
```

```
S2(config)#int ra f0/1-4, f0/6-17, f0/19-24, g0/1-2
S2(config-if-range)#shut
```

##

##### Шаг 8.	Назначьте сети VLAN соответствующим интерфейсам коммутатора.

> a.	Назначьте используемые порты соответствующей VLAN (указанной в таблице VLAN выше) и настройте их для режима статического доступа.

```
S1(config)#int f0/6
S1(config-if)#sw mo acc
S1(config-if)#sw acc vl 100

S2(config)#int f0/18
S2(config-if)#sw mo acc
S2(config-if)#sw acc vl 1
```

> b.	Убедитесь, что VLAN назначены на правильные интерфейсы.

```
S1#sh vl br
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/5
100  Clients                          active    Fa0/6


S2#sh vl br
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/5, Fa0/6, Fa0/7, Fa0/8
                                                Fa0/9, Fa0/10, Fa0/11, Fa0/12
                                                Fa0/13, Fa0/14, Fa0/15, Fa0/16
                                                Fa0/17, Fa0/18, Fa0/19, Fa0/20
                                                Fa0/21, Fa0/22, Fa0/23, Fa0/24
                                                Gig0/1, Gig0/2
```

> __Почему интерфейс F0/5 указан в VLAN 1?__

Потому что Vlan 1 всегда присутствует на коммутаторе по умолчанию и все порты также изначально принадлежат именно этому Vlan.

##

##### Шаг 9.	Вручную настройте интерфейс S1 F0/5 в качестве транка 802.1Q.

> a.	Измените режим порта коммутатора, чтобы принудительно создать магистральный канал.

```
S1(config)#int f0/5
S1(config-if)#sw mo tru
```

> b.	В рамках конфигурации транка  установите для native  VLAN значение 1000.

```
S1(config-if)#sw tr native vlan 1000
```

> c.	В качестве другой части конфигурации магистрали укажите, что VLAN 100, 200 и 1000 могут проходить по транку.

```
S1(config-if)#sw tr all vl 100,200,1000
```

> d.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.<br>
> e.	Проверьте состояние транка.

```
S1#sh int tr
Port        Mode         Encapsulation  Status        Native vlan
Fa0/5       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/5       100,200,1000

Port        Vlans allowed and active in management domain
Fa0/5       100,200,1000

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/5       100,200,1000
```

> __Какой IP-адрес был бы у ПК, если бы он был подключен к сети с помощью DHCP?__

Т.к. на текущий момент DHCP-сервер в сети отсутствует, то адрес ПК был бы __169.254.30.117__

##
### Часть 2.	Настройка и проверка двух серверов DHCPv4 на R1
#### Шаг 1.	Настройте R1 с пулами DHCPv4 для двух поддерживаемых подсетей. Ниже приведен только пул DHCP для подсети A

> a.	Исключите первые пять используемых адресов из каждого пула адресов.

```
R1(config)#ip dhcp excluded-address 192.168.1.2 192.168.1.6
R1(config)#ip dhcp excluded-address 192.168.1.99 192.168.1.103
```

> b.	Создайте пул DHCP (используйте уникальное имя для каждого пула).

```
R1(config)#ip dhcp pool FirstNet
R1(config)#ip dhcp pool SecondNet
```

> c.	Укажите сеть, поддерживающую этот DHCP-сервер.<br>
> d.	В качестве имени домена укажите CCNA-lab.com.<br>
> e.	Настройте соответствующий шлюз по умолчанию для каждого пула DHCP.<br>
> f.	Настройте время аренды на 2 дня 12 часов и 30 минут. __Команда неприменима в примере__ <br>

```
R1(dhcp-config)#network 192.168.1.0 255.255.255.192
R1(dhcp-config)#domain-name CCNA-lab.com
R1(dhcp-config)#default-router 192.168.1.1
R1(dhcp-config)#lease 2 12 30
                ^
% Invalid input detected at '^' marker.
```
 
> g.	Затем настройте второй пул DHCPv4, используя имя пула R2_Client_LAN и вычислите сеть, маршрутизатор по умолчанию, и используйте то же имя домена и время аренды, что и предыдущий пул DHCP.

```
R1(config)#ip dhcp pool R2_Client_LAN
R1(dhcp-config)#network 192.168.1.96 255.255.255.240
R1(dhcp-config)#domain-name CCNA-lab.com
R1(dhcp-config)#default-router 192.168.1.97
```

##
#### Шаг 3.	Проверка конфигурации сервера DHCPv4

> a.	Чтобы просмотреть сведения о пуле, выполните команду __show ip dhcp pool__.

```
R1#show ip dhcp pool 

Pool FirstNet :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 62
 Leased addresses               : 1
 Excluded addresses             : 2
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.1.1          192.168.1.1      - 192.168.1.62      1    / 2     / 62

Pool R2_Client_LAN :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 14
 Leased addresses               : 0
 Excluded addresses             : 2
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.1.97         192.168.1.97     - 192.168.1.110     0    / 2     / 14
```

> b.	Выполните команду __show ip dhcp bindings__ для проверки установленных назначений адресов DHCP.

```
R1#show ip dhcp binding
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.7      0090.213A.1E75           --                     Automatic
```

> c.	Выполните команду __show ip dhcp server statistics__ для проверки сообщений DHCP.

Неприменимо, возможно, ограничение Cisco Packet Tracer.

##
#### Шаг 4.	Попытка получить IP-адрес от DHCP на PC-A.

> a.	Из командной строки компьютера PC-A выполните команду ip config /all.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab08.1/files/ipconf1.jpg?raw=true)

> b.	После завершения процесса обновления выполните команду ip config для просмотра новой информации об IP-адресе.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab08.1/files/ipconf2.jpg?raw=true)

> c.	Проверьте подключение с помощью пинга IP-адреса интерфейса R0 G0/0/1.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab08.1/files/ping02.jpg?raw=true)

##
### Часть 3.	Настройка и проверка DHCP-ретрансляции на R2
#### Шаг 1.	Настройка R2 в качестве агента DHCP-ретрансляции для локальной сети на G0/0/1

> a.	Настройте команду ip helper-address на G0/0/1, указав IP-адрес G0/0/0 R1.

```
R2(config)#int g0/1
R2(config-if)#ip helper-address 10.0.0.1
```

##
#### Шаг 2.	Попытка получить IP-адрес от DHCP на PC-B

> a.	Из командной строки компьютера PC-B выполните команду ip config /all.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab08.1/files/ipconf3.jpg?raw=true)

> b.	После завершения процесса обновления выполните команду ip config для просмотра новой информации об IP-адресе.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab08.1/files/ipconf4.jpg?raw=true)

> c.	Проверьте подключение с помощью пинга IP-адреса интерфейса R1 G0/0/1.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab08.1/files/ping03.jpg?raw=true)

> d.	Выполните __show ip dhcp binding__ для R1 для проверки назначений адресов в DHCP.

```
R1#show ip dhcp binding
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.7      0090.213A.1E75           --                     Automatic
192.168.1.104    000C.CF45.2227           --                     Automatic
```

> e.	Выполните команду show ip dhcp server statistics для проверки сообщений DHCP.

Неприменимо, возможно, ограничение Cisco Packet Tracer.

Файл проекта по лабораторной работе [здесь](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab08.1/files/Lab8.1.pkt).
