## Лабораторная работа. Настройка IPv6-адресов на сетевых устройствах

#### Топология

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab04/files/topo.png?raw=true "Схема сети")

#### Таблица адресации

| Устройство	| Интерфейс	| IPv6-адрес	| Link local IPv6-адрес |	Длина префикса | Шлюз по умолчанию |
| ------------- |:------------------:|------------- |------------- |------------- |------------- |
| R1| 	G0/0/0| 	2001:db8:acad:a::1| 	fe80::1| 	64| 	—|
| R1| 	G0/0/1| 	2001:db8:acad:a::1| 	fe80::1| 	64| 	—|
| S1|	VLAN 1	|2001:db8:acad:1::b	| fe80::b	| 64|	—|
|PC-A|	NIC|	2001:db8:acad:1::3	|SLACC|	64	|fe80::1|
|PC-B	|NIC	|2001:db8:acad:a::3|	SLACC|	64|	fe80::1|

##

#### Задачи
##### Часть 1. Настройка топологии и конфигурация основных параметров маршрутизатора и коммутатора
##### Часть 2. Ручная настройка IPv6-адресов
##### Часть 3. Проверка сквозного соединения
##
#### Общие сведения/сценарий
Установлен шаблон dual-ipv4-and-ipv6 в качестве шаблона SDM по умолчанию на коммутаторе S1

```
S1#sh sdm prefer
 The current template is "dual-ipv4-and-ipv6 default" template.
 The selected template optimizes the resources in
 the switch to support this level of features for
 0 routed interfaces and 1024 VLANs.

  number of unicast mac addresses:                  4K
  number of IPv4 IGMP groups + multicast routes:    0.25K
  number of IPv4 unicast routes:                    0
  number of IPv6 multicast groups:                  0.375k
  number of directly-connected IPv6 addresses:      0
  number of indirect IPv6 unicast routes:           0
  number of IPv4 policy based routing aces:         0
  number of IPv4/MAC qos aces:                      0.125K
  number of IPv4/MAC security aces:                 0.375K
  number of IPv6 policy based routing aces:         0
  number of IPv6 qos aces:                          0.625k
  number of IPv6 security aces:                     0.125K
```
##

#### Часть 1.
В шагах 1 и 2 настроены коммутатор и роутер: настроено имя узла и основные параметры устройства.

##

#### Часть 2.
##### Шаг 1. Назначьте IPv6-адреса интерфейсам Ethernet на R1.
a. Назначены глобальные индивидуальные IPv6-адреса

b. 
```
R1#sh ipv6 int br
GigabitEthernet0/0/0       [up/up]
    FE80::206:2AFF:FE5A:9101
    2001:DB8:ACAD:A::1
GigabitEthernet0/0/1       [up/up]
    FE80::206:2AFF:FE5A:9102
    2001:DB8:ACAD:1::1
Vlan1                      [administratively down/down]
    unassigned

R1#sh ipv6 int g0/0/0
GigabitEthernet0/0/0 is up, line protocol is up
  IPv6 is enabled, link-local address is FE80::1
  No Virtual link-local address(es):
  Global unicast address(es):
    2001:DB8:ACAD:A::1, subnet is 2001:DB8:ACAD:A::/64
  Joined group address(es):
    FF02::1
    FF02::1:FF00:1
  MTU is 1500 bytes
  ICMP error messages limited to one every 100 milliseconds
  ICMP redirects are enabled
  ICMP unreachables are sent
  ND DAD is enabled, number of DAD attempts: 1
  ND reachable time is 30000 milliseconds
```
c. Введены локальные IPv6-адреса
```
R1#conf t
R1(config)#int g0/0/0
R1(config-if)#ipv6 address fe80::1 link-local
R1(config-if)#ex
R1(config)#int g0/0/1
R1(config-if)#ipv6 address fe80::1 link-local
```
d. Используем выбранную команду, чтобы убедиться, что локальный адрес связи изменен на fe80::1.  
```
R1#sh ipv6 int br
GigabitEthernet0/0/0       [up/up]
    FE80::1
    2001:DB8:ACAD:A::1
GigabitEthernet0/0/1       [up/up]
    FE80::1
    2001:DB8:ACAD:1::1
Vlan1                      [administratively down/down]
    unassigned


R1#sh ipv6 int g0/0/0
GigabitEthernet0/0/0 is up, line protocol is up
  IPv6 is enabled, link-local address is FE80::1
  No Virtual link-local address(es):
  Global unicast address(es):
    2001:DB8:ACAD:A::1, subnet is 2001:DB8:ACAD:A::/64
  Joined group address(es):
    FF02::1
    FF02::1:FF00:1
  MTU is 1500 bytes
  ICMP error messages limited to one every 100 milliseconds
  ICMP redirects are enabled
  ICMP unreachables are sent
  ND DAD is enabled, number of DAD attempts: 1
  ND reachable time is 30000 milliseconds
```

> Какие группы многоадресной рассылки назначены интерфейсу G0/0?

Группа многоадресной рассылки для всех узлов - FF02::1
Многоадресная группа запрошенных узлов - FF02::1:FF00:1

##

##### Шаг 2. Активация IPv6-маршрутизации на R1.
a.
```
C:\>ipconfig /all
   Connection-specific DNS Suffix..: 
   Physical Address................: 0090.2B36.A7AA
   Link-local IPv6 Address.........: FE80::290:2BFF:FE36:A7AA
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
```
> Назначен ли индивидуальный IPv6-адрес сетевой интерфейсной карте (NIC) на PC-B?

Нет

b. Активируем IPv6-маршрутизацию на R1 с помощью команды IPv6 unicast-routing.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab04/files/ipv6%20route.png?raw=true "Применение команды")

Проверим получение IPv6-адреса компьютером PC-B
```
C:\>ipconfig /all
   Connection-specific DNS Suffix..: 
   Physical Address................: 0090.2B36.A7AA
   Link-local IPv6 Address.........: FE80::290:2BFF:FE36:A7AA
   IPv6 Address....................: 2001:DB8:ACAD:A:290:2BFF:FE36:A7AA
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
```

> Почему PC-B получил глобальный префикс маршрутизации и идентификатор подсети, которые вы настроили на R1?


Т.к. после применения команды unicast-routing интерфейсы маршрутизатора могут получать и распространять глобальные IPv6-префиксы, потому что являются частью группы многоадрессной рассылки для всех маршрутизаторов FF02::2.

##

##### Шаг 3. Назначим IPv6-адреса интерфейсу управления (SVI) на S1.
a/b.
```
S1(config)#int vl 1
S1(config-if)#ipv6 address 2001:db8:acad:1::b/64
S1(config-if)#ipv6 address fe80::b link-local
S1#sh ipv6 int vl 1
Vlan1 is up, line protocol is up
  IPv6 is enabled, link-local address is FE80::B
  No Virtual link-local address(es):
  Global unicast address(es):
    2001:DB8:ACAD:1::B, subnet is 2001:DB8:ACAD:1::/64
  Joined group address(es):
    FF02::1
    FF02::1:FF00:B
```

##

##### Шаг 4. Назначим компьютерам статические IPv6-адреса.
a.
Настройки IPv6 PC-A


![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab04/files/PC-A%20IPv6.png?raw=true "PC-A")

Настройки IPv6 PC-B


![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab04/files/PC-B%20IPv6.png?raw=true "PC-B")



##

#### Часть 3. Проверка сквозного подключения

a. С PC-A отправьте эхо-запрос на FE80::1. Это локальный адрес канала, назначенный G0/1 на R1.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab04/files/ping%20fe80-pca.png?raw=true "Эхо с РС-а на FE80::1")

b. Отправьте эхо-запрос на интерфейс управления S1 с PC-A.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab04/files/ping%20s1-pca.png?raw=true "Эхо PC-A на S1")

c. Введите команду tracert на PC-A, чтобы проверить наличие сквозного подключения к PC-B.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab04/files/tracert%20pca-pcb.png?raw=true "Tracert c PC-A to PC-B")

d. С PC-B отправьте эхо-запрос на PC-A.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab04/files/ping%20pcb-pca.png?raw=true "Echo PC-b to PC-A")

e. С PC-B отправьте эхо-запрос на локальный адрес канала G0/0 на R1.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab04/files/ping%20pcb-fe80.png?raw=true "Ping PC-B to FE80")

##

#### Вопросы для повторения

> #### 1. Почему обоим интерфейсам Ethernet на R1 можно назначить один и тот же локальный адрес канала — FE80::1?

Т.к. локальные пакеты не покидают локальные сети, то разрешается/допускается использовать один и тот же локальный адрес на интерфейсах, которые связаны с другими локальными сетями.

> #### 2.	Какой идентификатор подсети в индивидуальном IPv6-адресе 2001:db8:acad::aaaa:1234/64?

/64 - Первые 64 бита определяют сеть и подсеть.

##### Исходный адрес

2001:db8:acad::aaaa:1234/64

##### Полный адрес 

2001:0db8:acad:0000:0000:0000:aaaa:1234 

##### Первые 64 бита, определяющие сеть и подсеть

2001:0db8:acad:0000

##### Сокращаем 0000 на ::, а также убираем лидирующие 0

2001:db8:acad::

##### ИТОГ

2001:db8:acad::/64

##

Файл проекта по лабораторной работе [здесь](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab04/files/Lab_IPv6.pkt)



