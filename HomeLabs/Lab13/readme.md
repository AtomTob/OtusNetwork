## Лабораторная работа - Настройка протоколов CDP, LLDP и NTP

### Топология
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab13/files/topo.png?raw=true)

##
### Таблица адресации

|Устройство|	Интерфейс|	IP-адрес	|Маска подсети	|Шлюз по умолчанию|
| ------------- |:------------------:|------------- |------------- |------------- |
|R1|	Loopback1	|172.16.1.1	|255.255.255.0|	—|
|R1|	G0/0/1|	10.22.0.1	|255.255.255.0|	—|
|S1|	SVI VLAN 1 |	10.22.0.2|	255.255.255.0	|10.22.0.1|
|S2|	SVI VLAN 1	|10.22.0.3|	255.255.255.0	|10.22.0.1|

##
### Часть 1. Создание сети и настройка основных параметров устройства.
#### Шаг 1. Создайте сеть согласно топологии.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab13/files/Topo2.jpg?raw=true)

#### Шаг 2. Настройте базовые параметры для маршрутизатора.
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
R1(config)#int g0/1
R1(config-if)#ip add 10.22.0.1 255.255.255.0
R1(config-if)#no shut
R1(config-if)#int l1
R1(config-if)#ip add 172.16.1.1 255.255.255.0
```

#### Шаг 3. Настройте базовые параметры каждого коммутатора.
```
Switch(config)#host S1
S1(config)#no ip domain-lookup
S1(config)#enable secret 0 class
S1(config)#line con 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#line vty 0 15
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#service password-encryption
S1(config)#banner motd ^C!!!!!!!!!!!!enter only for MA, no MAX or MAXIM!!!!!!^C
S1(config)#int ra Fa0/2-4, Fa0/6-24, Gig0/1, Gig0/2
S1(config-if-range)#shut
```

##
### Часть 2. Обнаружение сетевых ресурсов с помощью протокола CDP
> a.	На R1 используйте соответствующую команду show cdp, чтобы определить, сколько интерфейсов включено CDP, сколько из них включено и сколько отключено.
```
R1#sh cdp in 
Vlan1 is administratively down, line protocol is down
  Sending CDP packets every 60 seconds
  Holdtime is 180 seconds
GigabitEthernet0/0 is administratively down, line protocol is down
  Sending CDP packets every 60 seconds
  Holdtime is 180 seconds
GigabitEthernet0/1 is up, line protocol is up
  Sending CDP packets every 60 seconds
  Holdtime is 180 seconds
```

> Сколько интерфейсов участвует в объявлениях CDP? Какие из них активны?

Участвует 3 интерфейса, 1 активный.

> b.	На R1 используйте соответствующую команду show cdp, чтобы определить версию IOS, используемую на S1.
```
R1#show cdp entry  S1

Device ID: S1
Entry address(es): 
Platform: cisco 2960, Capabilities: Switch
Interface: GigabitEthernet0/1, Port ID (outgoing port): FastEthernet0/5
Holdtime: 149

Version :
Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE4, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2013 by Cisco Systems, Inc.
Compiled Wed 26-Jun-13 02:49 by mnguyen

advertisement version: 2
Duplex: full
```

> Какая версия IOS используется на  S1?

Cisco IOS Version 15.0(2)SE4


> c.	На S1 используйте соответствующую команду show cdp, чтобы определить, сколько пакетов CDP было выданных.
```
S1#show cdp traffic
            ^
% Invalid input detected at '^' marker.
	
S1#show cdp ?
  entry      Information for specific neighbor entry
  interface  CDP interface status and configuration
  neighbors  CDP neighbor entries
  <cr>
```
Неприменимо в CPT.

> d.	Настройте SVI для VLAN 1 на S1 и S2, используя IP-адреса, указанные в таблице адресации выше. <br>
> Настройте шлюз по умолчанию для каждого коммутатора на основе таблицы адресов. <br>
```
S1(config)#int vlan 1
S1(config-if)#ip address 10.22.0.2 255.255.255.0
S1(config-if)#no shut
S1(config)#ip default-gateway 10.22.0.1

S2(config)#int vlan 1
S2(config-if)#ip address 10.22.0.3 255.255.255.0
S2(config-if)#no shut
S2(config)#ip default-gateway 10.22.0.1
```


> e.	На R1 выполните команду show cdp entry S1 .
```
Device ID: S1
Entry address(es): 
  IP address : 10.22.0.2
Platform: cisco 2960, Capabilities: Switch
Interface: GigabitEthernet0/1, Port ID (outgoing port): FastEthernet0/5
Holdtime: 144

Version :
Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE4, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2013 by Cisco Systems, Inc.
Compiled Wed 26-Jun-13 02:49 by mnguyen

advertisement version: 2
Duplex: full
```

> Какие дополнительные сведения доступны теперь?

Доступна информация по интерфейсам коммутатора S1, назначенному адресу SVI.

> f.	Отключить CDP глобально на всех устройствах.
```
R1(config)#no cdp run
S1(config)#no cdp run
S2(config)#no cdp run
```

##
### Часть 3. Обнаружение сетевых ресурсов с помощью протокола LLDP
> a.	Введите соответствующую команду lldp, чтобы включить LLDP на всех устройствах в топологии.
```
R1(config)#lldp run
S1(config)#lldp run
S2(config)#lldp run
```

> b.	На S1 выполните соответствующую команду lldp, чтобы предоставить подробную информацию о S2.
```
S1#show lldp entry S2
             ^
% Invalid input detected at '^' marker.

S1#show lldp neighbors detail 
------------------------------------------------
Chassis id: 0000.0C08.3B01
Port id: Fa0/1
Port Description: FastEthernet0/1
System Name: S2
System Description:
Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE4, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2013 by Cisco Systems, Inc.
Compiled Wed 26-Jun-13 02:49 by mnguyen
Time remaining: 90 seconds
System Capabilities: B
Enabled Capabilities: B
Management Addresses - not advertised
Auto Negotiation - supported, enabled
Physical media capabilities:
    100baseT(FD)
    100baseT(HD)
    1000baseT(HD)
Media Attachment Unit type: 10
```

> Что такое chassis ID для коммутатора S2?

Это МАС-адрес интерфейса F0/1 коммутатора S2, по которому организовано подключение к коммутатору S1.
```
S2#sh int f0/1
FastEthernet0/1 is up, line protocol is up (connected)
  Hardware is Lance, address is 0000.0c08.3b01 (bia 0000.0c08.3b01)
 BW 100000 Kbit, DLY 1000 usec,
     reliability 255/255, txload 1/255, rxload 1/255
```

> c.	Соединитесь через консоль на всех устройствах и используйте команды LLDP, необходимые для отображения топологии физической сети только из выходных данных команды show.
```
R1#sh lldp neighbors 
Capability codes:
    (R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device
    (W) WLAN Access Point, (P) Repeater, (S) Station, (O) Other
Device ID           Local Intf     Hold-time  Capability      Port ID
S1                  Gig0/1         120        B               Fa0/5
Total entries displayed: 1


S1#sh lldp nei
Capability codes:
    (R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device
    (W) WLAN Access Point, (P) Repeater, (S) Station, (O) Other
Device ID           Local Intf     Hold-time  Capability      Port ID
S2                  Fa0/1          120        B               Fa0/1
R1                  Fa0/5          120        R               Gig0/1
Total entries displayed: 2


S2#sh lldp nei
Capability codes:
    (R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device
    (W) WLAN Access Point, (P) Repeater, (S) Station, (O) Other
Device ID           Local Intf     Hold-time  Capability      Port ID
S1                  Fa0/1          120        B               Fa0/1
Total entries displayed: 1
```

##
### Часть 4. Настройка NTP.
#### Шаг 1. Выведите на экран текущее время.
```
R1#sh clo
*1:56:12.938 UTC Mon Mar 1 1993
```
| Дата	| Время	| Часовой пояс| 	Источник времени| 
| ------------- |:------------------:|------------- |------------- |	
|01-03-1993 | 01:56:12.938 | UTC (+0) | Зашитая логика | 

#### Шаг 2. Установите время.
```
R1#clock set 23:00:00 apr 19 2026
```

#### Шаг 3. Настройте главный сервер NTP.
```
R1(config)#ntp master 4
```

#### Шаг 4. Настройте клиент NTP.
> a.	Выполните соответствующую команду на S1 и S2, чтобы просмотреть настроенное время. Запишите текущее время,  в следующей таблице.

| Коммутатор | Дата	| Время	| Часовой пояс| 	
| -------------   | ------------- |:------------------:|------------- |
|  S1  |01-03-1993 | 0:28:23.184 | UTC (+0) |
|  S2  |01-03-1993 | 0:29:9.519 | UTC (+0) |

