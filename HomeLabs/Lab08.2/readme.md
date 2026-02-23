## Лабраторная работа - Настройка DHCPv6.

### Топология
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab08.1/files/topo.jpg?raw=true)

### Таблица адресации

| Устройство	| Интерфейс	| IPv6-адрес| 
| ------------- |:------------------:|------------- |
| R1| 	G0/0/0| 	2001:db8:acad:2::1/64| 
| 	| | 	fe80::1| 
| | 	G0/0/1| 	2001:db8:acad:1::1/64| 
| 	| | 	fe80::1| 
| R2	| G0/0/0	| 2001:db8:acad:2::2/64| 
| 	| | 	fe80::2| 
| 	| G0/0/1	| 2001:db8:acad:3::1/64| 
| | | 	fe80::1| 
| PC-A	| NIC	| DHCP| 
| PC-B| 	NIC	| DHCP| 

##
### Часть 1. Создание сети и настройка основных параметров устройства.
#### Шаг 1. Создайте сеть согласно топологии.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab08.1/files/topo1.jpg?raw=true)

#### Шаг 2. Настройте базовые параметры каждого коммутатора. (необязательно)

Оба коммутатора настроены, согласно скрипту ниже, меняется лишь hostname, отключаемые порты и время:

```
hostname S1
no ip domain-lookup 
enable secret 0 class
int ran f0/1-4, f0/7-24,g0/1-2
shut
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
clock set 19:00:00 23 feb 2026
copy running-config startup-config
```
##
#### Шаг 3. Произведите базовую настройку маршрутизаторов.

Оба маршрутизатора настроены, согласно скрипту ниже, меняется лишь hostname и время:

```
ipv6 unicast-routing
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
end
clock set 19:00:00 23 feb 2026
copy running-config startup-config
```

##
#### Шаг 4. Настройка интерфейсов и маршрутизации для обоих маршрутизаторов.

> a.	Настройте интерфейсы G0/0/0 и G0/1 на R1 и R2 с адресами IPv6, указанными в таблице выше.<br>
> b.	Настройте маршрут по умолчанию на каждом маршрутизаторе, который указывает на IP-адрес G0/0/0 на другом маршрутизаторе.

```
R1(config)#int g0/0
R1(config-if)#ipv6 add 2001:db8:acad:2::1/64
R1(config-if)#ipv6 add fe80::1 link-local 
R1(config-if)#ex
R1(config)#int g0/1
R1(config-if)#ipv6 add 2001:db8:acad:1::1/64
R1(config-if)#ipv6 add fe80::1 link-local 

R2(config)#int g0/0
R2(config-if)#ipv6 add 2001:db8:acad:2::2/64
R2(config-if)#ipv6 add fe80::2 link-local 
R2(config-if)#ex
R2(config)#int g0/1
R2(config-if)#ipv6 add 2001:db8:acad:3::1/64
R2(config-if)#ipv6 add fe80::1 link-local 
```

