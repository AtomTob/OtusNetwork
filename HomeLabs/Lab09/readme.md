## Лабораторная работа - Конфигурация безопасности коммутатора

### Топология
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab09/files/topo1.jpg?raw=true)

### Таблица адресации

|Устройство|	interface/vlan	|IP-адрес	|Маска подсети|
| ------------- |:------------------:|------------- |------------- |
|R1	|G0/0/1	|192.168.10.1	|255.255.255.0|
|R1	|Loopback 0 |	10.10.1.1|	255.255.255.0|
|S1|	VLAN 10|	192.168.10.201	|255.255.255.0|
|S2	|VLAN 10	|192.168.10.202	|255.255.255.0|
|PC A|	NIC|	DHCP|	255.255.255.0|
|PC B	|NIC	|DHCP	|255.255.255.0|

## 
### Часть 1. Настройка основного сетевого устройства
#### Шаг 1. Создайте сеть.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab09/files/topo2.jpg?raw=true)

#### Шаг 2. Настройте маршрутизатор R1.
> a.	Загружен следующий конфигурационный скрипт на R1.<br>
> b.	Проверьте текущую конфигурацию на R1, используя следующую команду:

```
R1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0     unassigned      YES NVRAM  administratively down down 
GigabitEthernet0/1     192.168.10.1    YES manual up                    up 
Loopback0              10.10.1.1       YES manual up                    up 
Vlan1                  unassigned      YES NVRAM  administratively down down
```

> c.	Убедился, что IP-адресация и интерфейсы находятся в состоянии up / up (устранение неполадок не требуется).
##
#### Шаг 3. Настройка и проверка основных параметров коммутатора
> a.	Настройте имя хоста для коммутаторов S1 и S2. Откройте окно конфигурации.<br>
> b.	Запретите нежелательный поиск в DNS.<br>
> c.	Настройте описания интерфейса для портов, которые используются в S1 и S2.<br>
> d.	Установите для шлюза по умолчанию для VLAN управления значение 192.168.10.1 на обоих коммутаторах.<br>

```
Switch(config)#hostname S1
S1(config)#no ip domain-lookup
S1(config)#ip default-gateway 192.168.10.1
S1(config)#int f0/1
S1(config-if)#desc
S1(config-if)#description To_S2
S1(config-if)#int f0/6
S1(config-if)#description To_PC-A

Switch(config)#host S2
S2(config)#no ip domain-lookup 
S2(config)#ip default-gateway 192.168.10.1
S2(config)#int f0/1
S2(config-if)#desc To_S1
S2(config-if)#int f0/18
S2(config-if)#desc To_PC-B
```

### Часть 2. Настройка сетей VLAN на коммутаторах.
#### Шаг 1. Сконфигруриуйте VLAN 10.
```
S1(config)#vl 10
S1(config-vlan)#name Management

S2(config)#vl 10
S2(config-vlan)#name Management
```

#### Шаг 2. Сконфигруриуйте SVI для VLAN 10.
```
S1(config)#int vl 10
%LINK-5-CHANGED: Interface Vlan10, changed state to up
S1(config-if)#ip add 192.168.10.201 255.255.255.0
S1(config-if)#desc Management

S2(config)#int vl 10
%LINK-5-CHANGED: Interface Vlan10, changed state to up
S2(config-if)#ip add 192.168.10.202 255.255.255.0
S2(config-if)#desc Management
```
#### Шаг 3. Настройте VLAN 333 с именем Native на S1 и S2.
```
S1(config)#vl 333
S1(config-vlan)#name Native

S1(config)#vl 333
S1(config-vlan)#name Native
```
#### Шаг 4. Настройте VLAN 999 с именем ParkingLot на S1 и S2.
```
vl 999
name ParkingLot
```

### Часть 3. Настройки безопасности коммутатора.
####  Шаг 1. Релизация магистральных соединений 802.1Q.
> a.	Настройте все магистральные порты Fa0/1 на обоих коммутаторах для использования VLAN 333 в качестве native VLAN.
```
int f0/1
sw mo tr
sw tr nat vl 333
```
> b.	Убедитесь, что режим транкинга успешно настроен на всех коммутаторах.
```
S1#sh int tr
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      333

Port        Vlans allowed on trunk
Fa0/1       1-1005

Port        Vlans allowed and active in management domain
Fa0/1       1,10,333,999

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       1,10,333,999

S2#sh int tr
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      333

Port        Vlans allowed on trunk
Fa0/1       1-1005

Port        Vlans allowed and active in management domain
Fa0/1       1,10,333,999

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       1,10,333,999
```
> c.	Отключить согласование DTP F0/1 на S1 и S2. 
```
int f0/1
sw noneg
```
> d.	Проверьте с помощью команды __show interfaces__.
```
S1#show interfaces f0/1 switchport | include Negotiation
Negotiation of Trunking: Off

S2#show interfaces f0/1 switchport | include Negotiation
Negotiation of Trunking: Off
```
#### Шаг 2. Настройка портов доступа
> a.	На S1 настройте F0/5 и F0/6 в качестве портов доступа и свяжите их с VLAN 10.
```
S1(config)#int ran f0/5-6
S1(config-if-range)#sw mo acc
S1(config-if-range)#sw acc vl 10
```
> b.	На S2 настройте порт доступа Fa0/18 и свяжите его с VLAN 10.
```
S2(config)#int f0/18
S2(config-if)#sw mo acc
S2(config-if)#sw acc vl 10
```
#### Шаг 3. Безопасность неиспользуемых портов коммутатора.
> a.	На S1 и S2 переместите неиспользуемые порты из VLAN 1 в VLAN 999 и отключите неиспользуемые порты.
```
S1(config)#int ran f0/2-4,fa0/7-24,g0/1-2
S1(config-if-range)#sw mo acc
S1(config-if-range)#sw acc vl 999

S2(config)#int ran f0/2-17,f0/19-24,g0/1-2
S2(config-if-range)#sw mo acc
S2(config-if-range)#sw acc vl 999
S2(config-if-range)#shut
```
> b.	Убедитесь, что неиспользуемые порты отключены и связаны с VLAN 999, введя команду  
```
S1#sh int sta
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1     To_S2              connected    trunk      auto    auto  10/100BaseTX
Fa0/2                        disabled 999        auto    auto  10/100BaseTX
Fa0/3                        disabled 999        auto    auto  10/100BaseTX
Fa0/4                        disabled 999        auto    auto  10/100BaseTX
Fa0/5                        connected    10         auto    auto  10/100BaseTX
Fa0/6     To_PC-A            connected    10         auto    auto  10/100BaseTX
Fa0/7                        disabled 999        auto    auto  10/100BaseTX
Fa0/8                        disabled 999        auto    auto  10/100BaseTX
Fa0/9                        disabled 999        auto    auto  10/100BaseTX
Fa0/10                       disabled 999        auto    auto  10/100BaseTX
Fa0/11                       disabled 999        auto    auto  10/100BaseTX
Fa0/12                       disabled 999        auto    auto  10/100BaseTX
Fa0/13                       disabled 999        auto    auto  10/100BaseTX
Fa0/14                       disabled 999        auto    auto  10/100BaseTX
Fa0/15                       disabled 999        auto    auto  10/100BaseTX
Fa0/16                       disabled 999        auto    auto  10/100BaseTX
Fa0/17                       disabled 999        auto    auto  10/100BaseTX
Fa0/18                       disabled 999        auto    auto  10/100BaseTX
Fa0/19                       disabled 999        auto    auto  10/100BaseTX
Fa0/20                       disabled 999        auto    auto  10/100BaseTX
Fa0/21                       disabled 999        auto    auto  10/100BaseTX
Fa0/22                       disabled 999        auto    auto  10/100BaseTX
Fa0/23                       disabled 999        auto    auto  10/100BaseTX
Fa0/24                       disabled 999        auto    auto  10/100BaseTX
Gig0/1                       disabled 999        auto    auto  10/100BaseTX
Gig0/2                       disabled 999        auto    auto  10/100BaseTX

S2#sh int sta
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1     To_S1              connected    trunk      auto    auto  10/100BaseTX
Fa0/2                        disabled 999        auto    auto  10/100BaseTX
Fa0/3                        disabled 999        auto    auto  10/100BaseTX
Fa0/4                        disabled 999        auto    auto  10/100BaseTX
Fa0/5                        disabled 999        auto    auto  10/100BaseTX
Fa0/6                        disabled 999        auto    auto  10/100BaseTX
Fa0/7                        disabled 999        auto    auto  10/100BaseTX
Fa0/8                        disabled 999        auto    auto  10/100BaseTX
Fa0/9                        disabled 999        auto    auto  10/100BaseTX
Fa0/10                       disabled 999        auto    auto  10/100BaseTX
Fa0/11                       disabled 999        auto    auto  10/100BaseTX
Fa0/12                       disabled 999        auto    auto  10/100BaseTX
Fa0/13                       disabled 999        auto    auto  10/100BaseTX
Fa0/14                       disabled 999        auto    auto  10/100BaseTX
Fa0/15                       disabled 999        auto    auto  10/100BaseTX
Fa0/16                       disabled 999        auto    auto  10/100BaseTX
Fa0/17                       disabled 999        auto    auto  10/100BaseTX
Fa0/18    To_PC-B            connected    10         auto    auto  10/100BaseTX
Fa0/19                       disabled 999        auto    auto  10/100BaseTX
Fa0/20                       disabled 999        auto    auto  10/100BaseTX
Fa0/21                       disabled 999        auto    auto  10/100BaseTX
Fa0/22                       disabled 999        auto    auto  10/100BaseTX
Fa0/23                       disabled 999        auto    auto  10/100BaseTX
Fa0/24                       disabled 999        auto    auto  10/100BaseTX
Gig0/1                       disabled 999        auto    auto  10/100BaseTX
Gig0/2                       disabled 999        auto    auto  10/100BaseTX
```
#### Шаг 4. Документирование и реализация функций безопасности порта.
> a.	На S1, введите команду __show port-security interface f0/6__  для отображения настроек по умолчанию безопасности порта для интерфейса F0/6. Запишите свои ответы ниже.


|Функция	|Настройка по умолчанию|
| ------------- |:------------------:|
|Защита портов	|Выключена (Disabled)|
|Максимальное количество записей MAC-адресов	|1|
|Режим проверки на нарушение безопасности	|Отключена (Shutdown) |
|Aging Time	|0 минут|
|Aging Type	|Absolute|
|Secure Static Address Aging	|Disabled|
|Sticky MAC Address	|0|



