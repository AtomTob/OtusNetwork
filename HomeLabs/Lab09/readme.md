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
##
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
##
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
##
#### Шаг 4. Документирование и реализация функций безопасности порта.
> __a.	На S1, введите команду __show port-security interface f0/6__  для отображения настроек по умолчанию безопасности порта для интерфейса F0/6. Запишите свои ответы ниже.__


|Функция	|Настройка по умолчанию|
| ------------- |:------------------:|
|Защита портов	|Выключена (Disabled)|
|Максимальное количество записей MAC-адресов	|1|
|Режим проверки на нарушение безопасности	|Отключена (Shutdown) |
|Aging Time	|0 минут|
|Aging Type	|Absolute|
|Secure Static Address Aging	|Disabled|
|Sticky MAC Address	|0|

> b.	На S1 включите защиту порта на F0 / 6 со следующими настройками:<br>
> Максимальное количество записей MAC-адресов: 3<br>
> Режим безопасности: restrict<br>
> Aging time: 60 мин.<br>
> Aging type: неактивный<br>

```
S1(config)#int f0/6
S1(config-if)#sw port-sec 
S1(config-if)#sw port-sec max 3
S1(config-if)#sw port-se violation restrict 
S1(config-if)#sw port-sec aging time 60
```
> c.	Проверим настройки port-security на коммутаторе S1 на интерфейсе F0/6.
```
S1#show port-security interface f0/6
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Restrict
Aging Time                 : 60 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 3
Total MAC Addresses        : 0
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 0
Last Source Address:Vlan   : 0000.0000.0000:0
Security Violation Count   : 0
```
> d.	Включите безопасность порта для F0/18 на S2. Настройте каждый активный порт доступа таким образом, чтобы он автоматически добавлял адреса МАС, изученные на этом порту, в текущую конфигурацию.
```
S2(config)#int f0/18
S2(config-if)#sw port-sec
S2(config-if)#sw port-sec mac sticky
```
> __e.	Настройте следующие параметры безопасности порта на S2 F/18:__ <br>
> o	Максимальное количество записей MAC-адресов: 2<br>
> o	Тип безопасности: Protect<br>
> o	Aging time: 60 мин.<br>

```
S2(config)#int f0/18
S2(config-if)#sw port-sec maximum 2
S2(config-if)#sw port-sec violation protect
S2(config-if)#sw port-sec aging time 60
```
> f.	Проверка функции безопасности портов на S2 F0/18.
```
S2#sh port-sec int f0/18
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Protect
Aging Time                 : 60 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 2
Total MAC Addresses        : 1
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 1
Last Source Address:Vlan   : 0090.2BE8.99BC:10
Security Violation Count   : 0

S2#sh port-sec add
               Secure Mac Address Table
-----------------------------------------------------------------------------
Vlan    Mac Address       Type                          Ports   Remaining Age
                                                                   (mins)
----    -----------       ----                          -----   -------------
  10    0090.2BE8.99BC    SecureSticky                  Fa0/18       -
-----------------------------------------------------------------------------
Total Addresses in System (excluding one mac per port)     : 0
Max Addresses limit in System (excluding one mac per port) : 1024
```
##
#### Шаг 5. Реализовать безопасность DHCP snooping.
> a.	На S2 включите DHCP snooping и настройте DHCP snooping во VLAN 10.
```
S2(config)#ip dhcp snooping 
S2(config)#ip dhcp snooping vlan 10
```
> b.	Настройте магистральные порты на S2 как доверенные порты.
```
S2(config)#int f0/1
S2(config-if)#ip dhcp snooping trust
```
> c.	Ограничьте ненадежный порт Fa0/18 на S2 пятью DHCP-пакетами в секунду.
```
S2(config)#int f0/18
S2(config-if)#no ip dhcp snooping limit rate 5
```
> d.	Проверка DHCP Snooping на S2.
```
S2#sh ip dhcp snooping 
Switch DHCP snooping is enabled
DHCP snooping is configured on following VLANs:
10
Insertion of option 82 is enabled
Option 82 on untrusted port is not allowed
Verification of hwaddr field is enabled
Interface                  Trusted    Rate limit (pps)
-----------------------    -------    ----------------
FastEthernet0/1            yes        unlimited       
FastEthernet0/18           no         5   
```
> e.	В командной строке на PC-B освободите, а затем обновите IP-адрес.
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab09/files/iprenew.jpg?raw=true)

> f.	Проверьте привязку отслеживания DHCP с помощью команды __show ip dhcp snooping binding__.
```
S2#show ip dhcp snooping binding
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  -----------------
00:90:2B:E8:99:BC   192.168.10.10    0           dhcp-snooping  10    FastEthernet0/18
Total number of bindings: 1
```

##
#### Шаг 6. Реализация PortFast и BPDU Guard.
> a.	Настройте PortFast на всех портах доступа, которые используются на обоих коммутаторах.<br>
> b.	Включите защиту BPDU на портах доступа VLAN 10 S1 и S2, подключенных к PC-A и PC-B.

```
S1(config)#spanning-tree portfast bpduguard default 
S1(config)#int f0/6
S1(config-if)#spann portfast 
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION
%Portfast has been configured on FastEthernet0/6 but will only
have effect when the interface is in a non-trunking mode.
S1(config-if)#spanning-tree bpduguard enable


S2(config)#spanning-tree portfast bpduguard default 
S2(config)#int f0/18
S2(config-if)#spanning-tree portfast 
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION
%Portfast has been configured on FastEthernet0/18 but will only
have effect when the interface is in a non-trunking mode.
S2(config-if)#spanning-tree bpduguard enable
```

> c.	Убедитесь, что защита BPDU и PortFast включены на соответствующих портах.
```
S1#show spanning-tree interface f0/6 detail
Port 6 (FastEthernet0/6) of VLAN0010 is designated forwarding
  Port path cost 19, Port priority 128, Port Identifier 128.6
  Designated root has priority 32778, address 0009.7CAB.7B45
  Designated bridge has priority 32778, address 0009.7CAB.7B45
  Designated port id is 128.6, designated path cost 19
  Timers: message age 16, forward delay 0, hold 0
  Number of transitions to forwarding state: 1
  The port is in the portfast mode
  Link type is point-to-point by default
  ```
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab09/files/bpdus1.jpg?raw=true)

```
S2#show spanning-tree interface f0/18 detail
Port 18 (FastEthernet0/18) of VLAN0010 is designated forwarding
  Port path cost 19, Port priority 128, Port Identifier 128.18
  Designated root has priority 32778, address 0009.7CAB.7B45
  Designated bridge has priority 32778, address 0060.2FEA.B7A9
  Designated port id is 128.18, designated path cost 19
  Timers: message age 16, forward delay 0, hold 0
  Number of transitions to forwarding state: 1
  The port is in the portfast mode
  Link type is point-to-point by default
```
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab09/files/bpdu2.jpg?raw=true)

##
#### Шаг 7. Проверьте наличие сквозного ⁪подключения.
> Проверьте PING свзяь между всеми устройствами в таблице IP-адресации. В случае сбоя проверки связи может потребоваться отключить брандмауэр на хостах.


![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab09/files/ping_s1.jpg?raw=true)

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab09/files/ping_s2.jpg?raw=true)

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab09/files/ping_pca.jpg?raw=true)

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab09/files/ping_pcb.jpg?raw=true)

##
### Вопросы для повторения.

> 1.	С точки зрения безопасности порта на S2, почему нет значения таймера для оставшегося возраста в минутах, когда было сконфигурировано динамическое обучение - sticky?

В режиме динамического изучения sticky изученный МАС-адрес передается в рабочей конфигурации и запись становится статичной.<br>
Соответственно, таймер устаревания не используется для статической записи.


##
> 2.	Что касается безопасности порта на S2, если вы загружаете скрипт текущей конфигурации на S2, почему порту 18 на PC-B никогда не получит IP-адрес через DHCP?

Если я правильно понял вопрос, то при копировании нижеследующего скрипта настроек с порта F0/18:
```
 description To_PC-B
 switchport access vlan 10
 ip dhcp snooping limit rate 5
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky 
 switchport port-security violation protect 
 switchport port-security mac-address sticky 0090.2BE8.99BC
 switchport port-security aging time 60
 spanning-tree portfast
 spanning-tree bpduguard enable
```
в команде __switchport port-security mac-address sticky 0090.2BE8.99BC__ жестко задан МАС-адрес подключенного устройства.<br>
При загрузке скрипта в чистую конфигурацию велика вероятность, что у подключенного устройства МАС-адрес будет совершенно другой.<br>
Соответственно, коммутатор заблокирует порт, как не отвечающих политикам безопасности и хост не получит IP-адрес.

> 3.	Что касается безопасности порта, в чем разница между типом абсолютного устаревания и типом устаревание по неактивности?

При абсолютном устаревании адреса на порту будут удаляться при истечении установленного времени устаревания без каки-либо доп.условий.<br>
Тогда как при устаревании по неактивности адреса удаляются только если неактивны в течение заданного периода.

Файл проекта по лабораторной работе [здесь](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab09/files/lab09.pkt)
