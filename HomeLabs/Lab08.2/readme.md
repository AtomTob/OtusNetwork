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
R1(config)#ipv6 route ::1/64 2001:db8:acad:2::2
R1(config)#int g0/0
R1(config-if)#ipv6 add 2001:db8:acad:2::1/64
R1(config-if)#ipv6 add fe80::1 link-local 
R1(config-if)#ex
R1(config)#int g0/1
R1(config-if)#ipv6 add 2001:db8:acad:1::1/64
R1(config-if)#ipv6 add fe80::1 link-local 

R2(config)#ipv6 route ::1/64 2001:db8:acad:2::1
R2(config)#int g0/0
R2(config-if)#ipv6 add 2001:db8:acad:2::2/64
R2(config-if)#ipv6 add fe80::2 link-local 
R2(config-if)#ex
R2(config)#int g0/1
R2(config-if)#ipv6 add 2001:db8:acad:3::1/64
R2(config-if)#ipv6 add fe80::1 link-local 
```

> c.	Убедитесь, что маршрутизация работает с помощью пинга адреса G0/0/1 R2 из R1

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab08.2/files/ping1.jpg?raw=true)

##
### Часть 2. Проверка назначения адреса SLAAC от R1

> Через несколько минут результаты команды ipconfig должны показать, что PC-A присвоил себе адрес из сети 2001:db8:1::/64.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab08.2/files/ipconf1.jpg?raw=true)

> Откуда взялась часть адреса с идентификатором хоста?

При использовании SLAAC для назначения адресов IPv6 хостам сервер DHCPv6 не используется, поэтому в этом случае для части адреса хоста используется МАС-адрес устройства.

##
### Часть 3. Настройка и проверка сервера DHCPv6 на R1
#### Шаг 1. Более подробно изучите конфигурацию PC-A.

> a.	Выполните команду __ipconfig /all__ на PC-A и посмотрите на результат.

```
C:\>IPCONFIG /ALL

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0030.F2CD.DAA3
   Link-local IPv6 Address.........: FE80::230:F2FF:FECD:DAA3
   IPv6 Address....................: 2001:DB8:ACAD:1:230:F2FF:FECD:DAA3
   Autoconfiguration IP Address....: 169.254.218.163
   Subnet Mask.....................: 255.255.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-45-55-23-14-00-30-F2-CD-DA-A3
   DNS Servers.....................: ::
                                     0.0.0.0
```

DNS-серверы отсутствуют.

##
#### Шаг 2. Настройте R1 для предоставления DHCPv6 без состояния для PC-A.

> a.	Создайте пул DHCP IPv6 на R1 с именем R1-STATELESS. В составе этого пула назначьте адрес DNS-сервера как 2001:db8:acad: :1, а имя домена — как stateless.com.

```
R1(config)#ipv6 dhcp pool R1-STATELESS
R1(config-dhcpv6)#dns-server 2001:db8:acad::254
R1(config-dhcpv6)#domain-name STATELESS.com
```

> b.	Настройте интерфейс G0/0/1 на R1, чтобы предоставить флаг конфигурации OTHER для локальной сети R1 и укажите только что созданный пул DHCP в качестве ресурса DHCP для этого интерфейса.

```
R1(config)#int g0/1
R1(config-if)#ipv6 nd other-config-flag
R1(config-if)#ipv6 dhcp server R1-STATELESS
```

> c.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.<br>
> d.	Перезапустите PC-A.<br>
> e.	Проверьте вывод ipconfig /all и обратите внимание на изменения.<br>

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab08.2/files/ipconf2.jpg?raw=true)

Появился DNS-суффикс и DNS-сервер.

> f.	Тестирование подключения с помощью пинга IP-адреса интерфейса G0/1 R2.

```
C:\>ping 2001:db8:acad:3::1

Pinging 2001:db8:acad:3::1 with 32 bytes of data:

Reply from 2001:DB8:ACAD:3::1: bytes=32 time=4ms TTL=254
Reply from 2001:DB8:ACAD:3::1: bytes=32 time=1ms TTL=254
Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254

Ping statistics for 2001:DB8:ACAD:3::1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 4ms, Average = 1ms
```

##
### Часть 4. Настройка сервера DHCPv6 с сохранением состояния на R1.

> a.	Создайте пул DHCPv6 на R1 для сети 2001:db8:acad:3:aaa::/80. Это предоставит адреса локальной сети, подключенной к интерфейсу G0/0/1 на R2. В составе пула задайте DNS-сервер 2001:db8:acad: :254 и задайте доменное имя STATEFUL.com.

```
R1(config)#ipv6 dhcp pool R2-STATEFUL
R1(config-dhcpv6)#address prefix 2001:db8:acad:3:aaa::/80
R1(config-dhcpv6)#dns-server 2001:db8:acad::254
R1(config-dhcpv6)#domain-name STATEFUL.com
```

> b.	Назначьте только что созданный пул DHCPv6 интерфейсу g0/0 на R1.

```
R1(config)#interface g0/0
R1(config-if)#ipv6 dhcp server R2-STATEFUL
```

##
### Часть 5. Настройка и проверка ретрансляции DHCPv6 на R2.
#### Шаг 1. Включите PC-B и проверьте адрес SLAAC, который он генерирует.

```
C:\>ipconfig /all
FastEthernet0 Connection:(default port)
   Connection-specific DNS Suffix..: 
   Physical Address................: 0007.EC5C.809D
   Link-local IPv6 Address.........: FE80::207:ECFF:FE5C:809D
   IPv6 Address....................: 2001:DB8:ACAD:3:207:ECFF:FE5C:809D
   Autoconfiguration IP Address....: 169.254.128.157
   Subnet Mask.....................: 255.255.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-92-E7-D4-05-00-07-EC-5C-80-9D
   DNS Servers.....................: ::
                                     0.0.0.0
```

#### Шаг 2. Настройте R2 в качестве агента DHCP-ретрансляции для локальной сети на G0/0/1.

> a.	Настройте команду __ipv6 dhcp relay__ на интерфейсе R2 G0/0/1, указав адрес назначения интерфейса G0/0/0 на R1. Также настройте команду __managed-config-flag__ .

```
R2(config)#int g0/1
R2(config-if)#ipv6 nd managed-config-flag
R2(config-if)#ipv6 dhcp relay destination 2001:db8:acad:2::1 g0/0
                        ^
% Invalid input detected at '^' marker.
```

## Т.к. команда dhcp relay неприменима в Cisco Packet Tracer, поэтому реализуем dhcp-сервер сразу на R2:

```
R2(config)#ipv6 dhcp pool R2-STATEFUL
R2(config-dhcpv6)#address prefix 2001:db8:acad:3:aaa::/80
R2(config-dhcpv6)#dns-server 2001:db8:acad::254
R2(config-dhcpv6)#domain-name STATEFUL.com
R2(config-dhcpv6)#ex
R2(config)#int g0/1
R2(config-if)#ipv6 nd managed-config-flag 
R2(config-if)#ipv6 dhcp server R2-STATEFUL
```

#### Шаг 3. Попытка получить адрес IPv6 из DHCPv6 на PC-B.

> a.	Перезапустите PC-B.
> b.	Откройте командную строку на PC-B и выполните команду ipconfig /all и проверьте выходные данные, чтобы увидеть результаты операции ретрансляции DHCPv6.

