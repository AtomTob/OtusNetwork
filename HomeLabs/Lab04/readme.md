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


