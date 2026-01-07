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
