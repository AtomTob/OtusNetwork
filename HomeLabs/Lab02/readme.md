## Лабораторная работа. Просмотр таблицы MAC-адресов коммутатора

### Задание
#### Часть 1. Создание и настройка сети

Настроены базовые параметры и коммутаторов и компьютеров.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab02/files/Screenshot_1.png?raw=true "Схема сети")

##

#### Часть 2. Изучение таблицы МАС-адресов коммутатора
##### Шаг 1.

a.
> MAC-адрес компьютера PC-A:

   Physical Address................: 0030.A349.E855
   Link-local IPv6 Address.........: FE80::230:A3FF:FE49:E855

> MAC-адрес компьютера PC-B:

   Physical Address................: 0001.64B0.3448
   Link-local IPv6 Address.........: FE80::201:64FF:FEB0:3448

b. 
> МАС-адрес коммутатора S1 FastEthernet 0/1:

Address is 0001.c9de.0901 (bia 0001.c9de.0901)

> МАС-адрес коммутатора S2 FastEthernet 0/1:

Address is 0001.6345.0101 (bia 0001.6345.0101)

##

##### Шаг 2.

b.
```
S2#sh mac address-table 
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----

   1    0001.c9de.0901    DYNAMIC     Fa0/1
   1    0030.a349.e855    DYNAMIC     Fa0/1
```

> Какие МАС-адреса записаны в таблице? С какими портами коммутатора они сопоставлены и каким устройствам принадлежат?

В таблице указаны МАС-адреса коммутатора S1 и компьютера PC-A

> Если вы не записали МАС-адреса сетевых устройств в шаге 1, как можно определить, каким устройствам принадлежат МАС-адреса, используя только выходные данные команды show mac address-table? Работает ли это решение в любой ситуации?

При выводе команды указаны порты, на которых были получены МАС-адреса. Если несколько МАС-адресов не связаны с одним портом, то в этом случае возможно определить устройства в соответствии с портом.

##

##### Шаг 3.

b.
```
S2#clear mac address-table dynamic 
S2#sh mac address-table 
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----

   1    0001.c9de.0901    DYNAMIC     Fa0/1
```

> Указаны ли в таблице МАС-адресов адреса для VLAN 1?

Да

> Указаны ли другие МАС-адреса?

Нет

> Появились ли в таблице МАС-адресов новые адреса?

Нет

##

##### Шаг 4.

a.
```
C:\>arp -a
No ARP Entries Found
```

> Не считая адресов многоадресной и широковещательной рассылки, сколько пар IP- и МАС-адресов устройств было получено через протокол ARP?

Какие-либо записи отсутствуют

b.
```
C:\> ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time<1ms TTL=128
Reply from 192.168.1.1: bytes=32 time<1ms TTL=128
Reply from 192.168.1.1: bytes=32 time<1ms TTL=128
Reply from 192.168.1.1: bytes=32 time<1ms TTL=128

Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\> ping 192.168.1.11

Pinging 192.168.1.11 with 32 bytes of data:

Request timed out.
Reply from 192.168.1.11: bytes=32 time<1ms TTL=255
Reply from 192.168.1.11: bytes=32 time<1ms TTL=255
Reply from 192.168.1.11: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.11:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\> ping 192.168.1.12

Pinging 192.168.1.12 with 32 bytes of data:

Request timed out.
Reply from 192.168.1.12: bytes=32 time<1ms TTL=255
Reply from 192.168.1.12: bytes=32 time<1ms TTL=255
Reply from 192.168.1.12: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.12:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```
> От всех ли устройств получены ответы?

Да

c. 
```
S2#sh mac address-table 
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----

   1    0001.64b0.3448    DYNAMIC     Fa0/18
   1    0001.c9de.0901    DYNAMIC     Fa0/1
   1    0005.5e31.4ca7    DYNAMIC     Fa0/1
   1    0030.a349.e855    DYNAMIC     Fa0/1
```

> Добавил ли коммутатор в таблицу МАС-адресов дополнительные МАС-адреса? Если да, то какие адреса и устройства?

Появились адреса компьютеров PC-A, PC-B 

> На компьютере PC-B откройте командную строку и еще раз введите команду arp -a. Появились ли в ARP-кэше компьютера PC-B дополнительные записи для всех сетевых устройств, которым были отправлены эхо-запросы?
```
C:\>arp -a
  Internet Address      Physical Address      Type
  192.168.1.1           0030.a349.e855        dynamic
  192.168.1.11          0005.5e31.4ca7        dynamic
  192.168.1.12          0060.47ac.084a        dynamic
```

В ARP-кэше появились записи для всех устройств, которым были отправлены эхо-запросы.

##### 	Вопрос для повторения
> В сетях Ethernet данные передаются на устройства по соответствующим МАС-адресам. Для этого коммутаторы и компьютеры динамически создают ARP-кэш и таблицы МАС-адресов. Если компьютеров в сети немного, эта процедура выглядит достаточно простой. Какие сложности могут возникнуть в крупных сетях?

Может возрастать нагрузка на сети в следствие большого количества широковещательных запросов.


Файл проекта по лабораторной работе [здесь](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab02/files/Lab%2002.pkt)
