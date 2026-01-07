## Лабораторная работа. Базовая настройка коммутатора

### Задание
##### Часть 1. Проверка конфигурации коммутатора по умолчанию
##### Часть 2. Создание сети и настройка основных параметров устройства
- Настройте базовые параметры коммутатора.
- Настройте IP-адрес для ПК.
##### Часть 3. Проверка сетевых подключений
- Отобразите конфигурацию устройства.
- Протестируйте сквозное соединение, отправив эхо-запрос.
- Протестируйте возможности удаленного управления с помощью Telnet.



### Выполнение задания
##### Часть 1. Шаг 1.
Ответ на вопрос:
> Почему нужно использовать консольное подключение для первоначальной настройки коммутатора? Почему нельзя подключиться к коммутатору через Telnet или SSH?

Потому что в заводской (первоначальной) конфигурации отсутствуют настройки доступа по Telnet или SSH.

##### Часть 1. Шаг 2.
b. 
> Сколько интерфейсов FastEthernet имеется на коммутаторе 2960?

    24
    
> Сколько интерфейсов GigabitEthernet имеется на коммутаторе 2960?

    2
    
> Каков диапазон значений, отображаемых в vty-линиях?

    0
    0 - 4
    5 - 15
    
c. 
> Почему появляется это сообщение? Имеется ввиду startup-config is not present.

Потому что устройство находится в заводской конфигурации

d.
> Назначен ли IP-адрес сети VLAN 1?

Нет

> Какой MAC-адрес имеет SVI? Возможны различные варианты ответов.

000a.f32e.64ce

> Данный интерфейс включен?

Интерфейс находится в состоянии administratively down, т.е. выключен.

e + f.
> Изучите IP-свойства интерфейса SVI сети VLAN 1.Какие выходные данные вы видите?

Данные по настройке IP на интерфейсе Vlan 1 отсутствуют.

g. 
> Под управлением какой версии ОС Cisco IOS работает коммутатор? Как называется файл образа системы?

Версия IOS - 15.0(2)SE4. Файл образа - c2960-lanbasek9-mz.150-2.SE4.bin.

h.
> Switch# show interface f0/6.  Интерфейс включен или выключен? Что нужно сделать, чтобы включить интерфейс? Какой MAC-адрес у интерфейса? Какие настройки скорости и дуплекса заданы в интерфейсе?

По умолчанию интерфейс в состоянии down, при подключении хоста порт переъодит в состояние Up. Есть интерфейс в состоянии administratively down, то включить его необходимо командой no shutdown при конфигурировании интерфейса.

MAC-адрес - 0000.0c09.b106. Настройки скорости/дуплекса - Full-duplex, 100Mb/s

i. 
> Какое имя присвоено образу Cisco IOS?

2960-lanbasek9-mz.150-2.SE4.bin

##

##### Часть 2.
###### Шаг 1.
Введены все необходимые команды.
> Для чего нужна команда login?

Без команды login устройство не будет запрашивать пароль при попытке удалённого входа.

###### Шаг 2.
Выполнены все необходимые настройки.

##

##### Часть 3. Шаг 1.

###### Настройка коммутатора S1

a.
```
Current configuration : 1317 bytes
!
version 15.0
no service timestamps log datetime msec
no service timestamps debug datetime msec
service password-encryption
!
hostname S1
!
enable secret 5 $1$mERr$hx5rVt7rPNoS4wqbXKX7m0
!
no ip domain-lookup
!
spanning-tree mode pvst
spanning-tree extend system-id
!
interface FastEthernet0/1
!
interface FastEthernet0/2
!
interface FastEthernet0/3
!
interface FastEthernet0/4
!
interface FastEthernet0/5
!
interface FastEthernet0/6
!
interface FastEthernet0/7
!
interface FastEthernet0/8
!
interface FastEthernet0/9
!
interface FastEthernet0/10
!
interface FastEthernet0/11
!
interface FastEthernet0/12
!
interface FastEthernet0/13
!
interface FastEthernet0/14
!
interface FastEthernet0/15
!
interface FastEthernet0/16
!
interface FastEthernet0/17
!
interface FastEthernet0/18
!
interface FastEthernet0/19
!
interface FastEthernet0/20
!
interface FastEthernet0/21
!
interface FastEthernet0/22
!
interface FastEthernet0/23
!
interface FastEthernet0/24
!
interface GigabitEthernet0/1
!
interface GigabitEthernet0/2
!
interface Vlan1
 ip address 192.168.1.2 255.255.255.0
!
banner motd ^C Unauthorized access is strictly prohibited. ^C
!
line con 0
 password 7 0822455D0A16
 logging synchronous
 login
!
line vty 0 4
 password 7 0822455D0A16
 login
 transport input telnet
line vty 5 15
 password 7 0822455D0A16
 login
!
end
```

b.
```
S1#sh int vl 1
Vlan1 is up, line protocol is up
  Hardware is CPU Interface, address is 000a.f32e.64ce (bia 000a.f32e.64ce)
  Internet address is 192.168.1.2/24
  MTU 1500 bytes, BW 100000 Kbit, DLY 1000000 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 21:40:21, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/75/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     1682 packets input, 530955 bytes, 0 no buffer
     Received 0 broadcasts (0 IP multicast)
     0 runts, 0 giants, 0 throttles
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     563859 packets output, 0 bytes, 0 underruns
     0 output errors, 23 interface resets
     0 output buffer failures, 0 output buffers swapped out
```
> Какова полоса пропускания этого интерфейса?

100 МБит/сек

###### Шаг 2.
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab01/files/Screenshot_3.png?raw=true)

###### Шаг 3.
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab01/files/Screenshot_1.png?raw=true)


##### Вопросы для повторения
> 1. Зачем необходимо настраивать пароль VTY для коммутатора?

Для защиты от несанкционированного доступа

> 2.	Что нужно сделать, чтобы пароли не отправлялись в незашифрованном виде?

Использовать зашифрованные протоколы подключения (SSH вместо Telnet)

##### 	Приложение А. Инициализация и перезагрузка коммутатора
```
User Access Verification

Password: 

S1>en
Password: 
S1#sh flash
Directory of flash:/

    1  -rw-     4670455          <no date>  2960-lanbasek9-mz.150-2.SE4.bin
    2  -rw-        1342          <no date>  config.text

64016384 bytes total (59344587 bytes free)
S1#erase startup-config 
Erasing the nvram filesystem will remove all configuration files! Continue? [confirm]
[OK]
Erase of nvram: complete
%SYS-7-NV_BLOCK_INIT: Initialized the geometry of nvram
S1#reload
Proceed with reload? [confirm]
C2960 Boot Loader (C2960-HBOOT-M) Version 12.2(25r)FX, RELEASE SOFTWARE (fc4)
Cisco WS-C2960-24TT (RC32300) processor (revision C0) with 21039K bytes of memory.
2960-24TT starting...
Base ethernet MAC Address: 000A.F32E.64CE
Xmodem file system is available.
Initializing Flash...
flashfs[0]: 1 files, 0 directories
flashfs[0]: 0 orphaned files, 0 orphaned directories
flashfs[0]: Total bytes: 64016384
flashfs[0]: Bytes used: 4670455
flashfs[0]: Bytes available: 59345929
flashfs[0]: flashfs fsck took 1 seconds.
...done Initializing Flash.

Boot Sector Filesystem (bs:) installed, fsid: 3
Parameter Block Filesystem (pb:) installed, fsid: 4


Loading "flash:/2960-lanbasek9-mz.150-2.SE4.bin"...
########################################################################## [OK]
Smart Init is enabled
smart init is sizing iomem
                  TYPE      MEMORY_REQ
                TOTAL:      0x00000000
Rounded IOMEM up to: 0Mb.
Using 6 percent iomem. [0Mb/512Mb]

              Restricted Rights Legend
Use, duplication, or disclosure by the Government is
subject to restrictions as set forth in subparagraph
(c) of the Commercial Computer Software - Restricted
Rights clause at FAR sec. 52.227-19 and subparagraph
(c) (1) (ii) of the Rights in Technical Data and Computer
Software clause at DFARS sec. 252.227-7013.
           cisco Systems, Inc.
           170 West Tasman Drive
           San Jose, California 95134-1706
Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE4, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2013 by Cisco Systems, Inc.
Compiled Wed 26-Jun-13 02:49 by mnguyen
Initializing flashfs...
fsck: Disable shadow buffering due to heap fragmentation.
flashfs[2]: 2 files, 1 directories
flashfs[2]: 0 orphaned files, 0 orphaned directories
flashfs[2]: Total bytes: 32514048
flashfs[2]: Bytes used: 11952128
flashfs[2]: Bytes available: 20561920
flashfs[2]: flashfs fsck took 2 seconds.
flashfs[2]: Initialization complete....done Initializing flashfs.
Checking for Bootloader upgrade..
Boot Loader upgrade not required (Stage 2)
POST: CPU MIC register Tests : Begin
POST: CPU MIC register Tests : End, Status Passed
POST: PortASIC Memory Tests : Begin
POST: PortASIC Memory Tests : End, Status Passed
POST: CPU MIC interface Loopback Tests : Begin
POST: CPU MIC interface Loopback Tests : End, Status Passed
POST: PortASIC RingLoopback Tests : Begin
POST: PortASIC RingLoopback Tests : End, Status Passed
POST: PortASIC CAM Subsystem Tests : Begin
POST: PortASIC CAM Subsystem Tests : End, Status Passed
POST: PortASIC Port Loopback Tests : Begin
POST: PortASIC Port Loopback Tests : End, Status Passed
Waiting for Port download...Complete

This product contains cryptographic features and is subject to United
States and local country laws governing import, export, transfer and
use. Delivery of Cisco cryptographic products does not imply
third-party authority to import, export, distribute or use encryption.
Importers, exporters, distributors and users are responsible for
compliance with U.S. and local country laws. By using this product you
agree to comply with applicable laws and regulations. If you are unable
to comply with U.S. and local laws, return this product immediately.
A summary of U.S. laws governing Cisco cryptographic products may be found at:
http://www.cisco.com/wwl/export/crypto/tool/stqrg.html
If you require further assistance please contact us by sending email to
export@cisco.com.
cisco WS-C2960-24TT-L (PowerPC405) processor (revision B0) with 65536K bytes of memory.
Processor board ID FOC1010X104
Last reset from power-on
1 Virtual Ethernet interface
24 FastEthernet interfaces
2 Gigabit Ethernet interfaces
The password-recovery mechanism is enabled.
64K bytes of flash-simulated non-volatile configuration memory.
Base ethernet MAC Address       : 00:0A:F3:2E:64:CE
Motherboard assembly number     : 73-10390-03
Power supply part number        : 341-0097-02
Motherboard serial number       : FOC10093R12
Power supply serial number      : AZS1007032H
Model revision number           : B0
Motherboard revision number     : B0
Model number                    : WS-C2960-24TT-L
System serial number            : FOC1010X104
Top Assembly Part Number        : 800-27221-02
Top Assembly Revision Number    : A0
Version ID                      : V02
CLEI Code Number                : COM3L00BRA
Hardware Board Revision Number  : 0x01

Switch Ports Model              SW Version            SW Image
------ ----- -----              ----------            ----------
*    1 26    WS-C2960-24TT-L    15.0(2)SE4            C2960-LANBASEK9-M

Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE4, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2013 by Cisco Systems, Inc.
Compiled Wed 26-Jun-13 02:49 by mnguyen



Press RETURN to get started!


%LINK-5-CHANGED: Interface FastEthernet0/6, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/6, changed state to up


Switch>en
Switch#
```
