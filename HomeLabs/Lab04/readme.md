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

#### Общие сведения/сценарий
Установлен шаблон dual-ipv4-and-ipv6 в качестве шаблона SDM по умолчанию на коммутаторе S1

```
S1#show sdm prefer
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


##### Шаг 1.


![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab04/files/topo2.png?raw=true "Схема проекта")

