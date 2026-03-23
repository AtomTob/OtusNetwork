## Лабораторная работа - Настройка протокола OSPFv2 для одной области

### Топология
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab10/files/topo1.jpg?raw=true)

### Таблица адресации

|Устройство|	Интерфейс	|IP-адрес	|Маска подсети|
| ------------- |:------------------:|------------- |------------- |
|R1|	G0/0/1|	10.53.0.1|	255.255.255.0|
|R1|	Loopback1|	172.16.1.1	|255.255.255.0|
|R2	|G0/0/1	|10.53.0.2|	255.255.255.0|
|R2	|Loopback1	|192.168.1.1|	255.255.255.0|

## 
### Часть 1. Создание сети и настройка основных параметров устройства
#### Шаг 1. Создайте сеть согласно топологии.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab10/files/topo2.jpg?raw=true)

#### Шаг 2. Произведите базовую настройку маршрутизаторов.

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
clock timezone EKB 5 0
ip domain-name home.local 
crypto key generate rsa general-keys modulus 2048 
ip ssh version 2 
username admin secret 5 $1$mERr$qJb.eHvBN7S590aq.dpRL. 
line vty 0 4 
transport input ssh 
login local 
exit
exit
clock set 18:00:00 22 march 2026
copy running-config startup-config
```

#### Шаг 3. Настройте базовые параметры каждого коммутатора.

Настроено по аналогии с маршрутизаторами

## 
### Часть 2. Настройка и проверка базовой работы протокола OSPFv2 для одной области
#### Шаг 1. Настройте адреса интерфейса и базового OSPFv2 на каждом маршрутизаторе.

> a.	Настройте адреса интерфейсов на каждом маршрутизаторе, как показано в таблице адресации выше.

```
R1(config)#int gigabitEthernet 0/1
R1(config-if)#ip address 10.53.0.1 255.255.255.0
R1(config-if)#no shut
R1(config)#int loopback 1
R1(config-if)#ip add 172.16.1.1 255.255.255.0

R2(config)#int gigabitEthernet 0/1
R2(config-if)#ip address 10.53.0.2 255.255.255.0
R2(config-if)#no shut
R2(config-if)#int loopback 1
R2(config-if)#ip add 192.168.1.1 255.255.255.0
```

> b.	Перейдите в режим конфигурации маршрутизатора OSPF, используя идентификатор процесса 56.<br>
> c.	Настройте статический идентификатор маршрутизатора для каждого маршрутизатора (1.1.1.1 для R1, 2.2.2.2 для R2).<br>
> d.	Настройте инструкцию сети для сети между R1 и R2, поместив ее в область 0.<br>
```
R1(config)#router ospf 56
R1(config-router)#router-id 1.1.1.1
R1(config)#int g 0/1
R1(config-if)#ip ospf 56 area 0

R2(config)#router ospf 56
R2(config-router)#router-id 2.2.2.2
R2(config-router)#int g 0/1
R2(config-if)#ip ospf 56 area 0
```

> e.	Только на R2 добавьте конфигурацию, необходимую для объявления сети Loopback 1 в область OSPF 0.
```
R2(config)#int loopback 1
R2(config-if)#ip ospf 56 area 0
```

> f.	Убедитесь, что OSPFv2 работает между маршрутизаторами. Выполните команду, чтобы убедиться, что R1 и R2 сформировали смежность.
```
R1#sh ip ospf neighbor 
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/BDR        00:00:35    10.53.0.2       GigabitEthernet0/1

R2#sh ip ospf neighbor 
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           1   FULL/DR         00:00:36    10.53.0.1       GigabitEthernet0/1
```

__Какой маршрутизатор является DR?__ <br>
Маршрутизатор R1. <br>
__Какой маршрутизатор является BDR?__ <br>
Маршрутизатор R2. <br>
__Каковы критерии отбора?__ <br>
Даже не смотря на то, что маршрутизатор R2 имеет более высокий идентификатор - 2.2.2.2, R1 выбран в качестве DR, так как его настройка производилась раньше, чем R2. <br>

> g.	На R1 выполните команду __show ip route ospf__ , чтобы убедиться, что сеть R2 Loopback1 присутствует в таблице маршрутизации. Обратите внимание, что поведение OSPF по умолчанию заключается в объявлении интерфейса обратной связи в качестве маршрута узла с использованием 32-битной маски.
```
R1#sh ip route ospf 
     192.168.1.0/32 is subnetted, 1 subnets
O       192.168.1.1 [110/2] via 10.53.0.2, 00:14:17, GigabitEthernet0/1
```

> h.	Запустите Ping до  адреса интерфейса R2 Loopback 1 из R1. Выполнение команды ping должно быть успешным.
```
R1#ping 192.168.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/2 ms
```
##
### Часть 3. Оптимизация и проверка конфигурации OSPFv2 для одной области
#### Шаг 1. Реализация различных оптимизаций на каждом маршрутизаторе.

> a.	На R1 настройте приоритет OSPF интерфейса G0/0/1 на 50, чтобы убедиться, что R1 является назначенным маршрутизатором.
```
R1(config)#int g0/1
R1(config-if)#ip ospf priority 50
R1#clear ip ospf process 
Reset ALL OSPF processes? [no]: yes
19:59:17: %OSPF-5-ADJCHG: Process 56, Nbr 2.2.2.2 on GigabitEthernet0/1 from FULL to DOWN, Neighbor Down: Adjacency forced to reset
19:59:17: %OSPF-5-ADJCHG: Process 56, Nbr 2.2.2.2 on GigabitEthernet0/1 from FULL to DOWN, Neighbor Down: Interface down or detached
19:59:19: %OSPF-5-ADJCHG: Process 56, Nbr 2.2.2.2 on GigabitEthernet0/1 from LOADING to FULL, Loading Done

R1#sh ip ospf neighbor 
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/BDR        00:00:34    10.53.0.2       GigabitEthernet0/1
```

> b.	Настройте таймеры OSPF на G0/0/1 каждого маршрутизатора для таймера приветствия, составляющего 30 секунд.
```
R1(config)#int g0/1
R1(config-if)#ip ospf hello-interval 30
20:52:18: %OSPF-5-ADJCHG: Process 56, Nbr 2.2.2.2 on GigabitEthernet0/1 from FULL to DOWN, Neighbor Down: Dead timer expired
20:52:18: %OSPF-5-ADJCHG: Process 56, Nbr 2.2.2.2 on GigabitEthernet0/1 from FULL to DOWN, Neighbor Down: Interface down or detached

R1#sh ip ospf neighbor 
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/BDR        00:00:29    10.53.0.2       GigabitEthernet0/1

R2(config)#int g0/1
R2(config-if)#ip ospf hello-interval 30
20:49:22: %OSPF-5-ADJCHG: Process 56, Nbr 1.1.1.1 on GigabitEthernet0/1 from FULL to DOWN, Neighbor Down: Dead timer expired
20:49:22: %OSPF-5-ADJCHG: Process 56, Nbr 1.1.1.1 on GigabitEthernet0/1 from FULL to DOWN, Neighbor Down: Interface down or detached
20:49:52: %OSPF-5-ADJCHG: Process 56, Nbr 1.1.1.1 on GigabitEthernet0/1 from LOADING to FULL, Loading Done

R2#sh ip ospf nei
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1          50   FULL/DR         00:00:27    10.53.0.1       GigabitEthernet0/1
```

> c.	На R1 настройте статический маршрут по умолчанию, который использует интерфейс Loopback 1 в качестве интерфейса выхода. Затем распространите маршрут по умолчанию в OSPF. Обратите внимание на сообщение консоли после установки маршрута по умолчанию.
```
R1(config)#ip route 0.0.0.0 0.0.0.0 loopback 1
%Default route without gateway, if not a point-to-point interface, may impact performance

R1(config)#router ospf 56
R1(config-router)#default-information originate
```
> d.	Добавьте конфигурацию, необходимую для OSPF для обработки R2 Loopback 1 как сети точка-точка. Это приводит к тому, что OSPF объявляет Loopback 1 использует маску подсети интерфейса.
```
R2(config)#int l1
R2(config-if)#ip ospf network point-to-point 
```
> e.	Только на R2 добавьте конфигурацию, необходимую для предотвращения отправки объявлений OSPF в сеть Loopback 1.
```
R2(config)#rout osp 56
R2(config-router)#passive-interface l1
```
> f.	Измените базовую пропускную способность для маршрутизаторов. После этой настройки перезапустите OSPF с помощью команды clear ip ospf process . Обратите внимание на сообщение консоли после установки новой опорной полосы пропускания.
```
R1(config)#router ospf 56
R1(config-router)#auto-cost reference-bandwidth 1000
% OSPF: Reference bandwidth is changed.
        Please ensure reference bandwidth is consistent across all routers.

R1#clear ip ospf process 
Reset ALL OSPF processes? [no]: yes
04:46:14: %OSPF-5-ADJCHG: Process 56, Nbr 2.2.2.2 on GigabitEthernet0/1 from FULL to DOWN, Neighbor Down: Adjacency forced to reset
04:46:14: %OSPF-5-ADJCHG: Process 56, Nbr 2.2.2.2 on GigabitEthernet0/1 from FULL to DOWN, Neighbor Down: Interface down or detached
04:46:37: %OSPF-5-ADJCHG: Process 56, Nbr 2.2.2.2 on GigabitEthernet0/1 from LOADING to FULL, Loading Done


R2(config)#router ospf 56
R2(config-router)#auto-cost reference-bandwidth 1000
% OSPF: Reference bandwidth is changed.
        Please ensure reference bandwidth is consistent across all routers.

R2#clear ip ospf pro
Reset ALL OSPF processes? [no]: yes
04:47:11: %OSPF-5-ADJCHG: Process 56, Nbr 1.1.1.1 on GigabitEthernet0/1 from FULL to DOWN, Neighbor Down: Adjacency forced to reset
04:47:11: %OSPF-5-ADJCHG: Process 56, Nbr 1.1.1.1 on GigabitEthernet0/1 from FULL to DOWN, Neighbor Down: Interface down or detached
04:47:38: %OSPF-5-ADJCHG: Process 56, Nbr 1.1.1.1 on GigabitEthernet0/1 from LOADING to FULL, Loading Done
```

##
### Шаг 2. Убедитесь, что оптимизация OSPFv2 реализовалась.
> a.	Выполните команду show ip ospf interface g0/0/1 на R1 и убедитесь, что приоритет интерфейса установлен равным 50, а временные интервалы — Hello 30, Dead 120, а тип сети по умолчанию — Broadcast
```
R1#sh ip os int g0/1
GigabitEthernet0/1 is up, line protocol is up
  Internet address is 10.53.0.1/24, Area 0
  Process ID 56, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 10
  Transmit Delay is 1 sec, State DR, Priority 50
  Designated Router (ID) 1.1.1.1, Interface address 10.53.0.1
  Backup Designated Router (ID) 2.2.2.2, Interface address 10.53.0.2
  Timer intervals configured, Hello 30, Dead 40, Wait 40, Retransmit 5
    Hello due in 00:00:00
  Index 1/1, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1
    Adjacent with neighbor 2.2.2.2  (Backup Designated Router)
  Suppress hello for 0 neighbor(s)
```
__P.S. В данном случае после смены Hello-интервала, в следствие ограничений Cisco Packet Tracer, не перечитался Dead-интервал. Все остальные параметры соответствуют.__

> b.	На R1 выполните команду __show ip route ospf__, чтобы убедиться, что сеть R2 Loopback1 присутствует в таблице маршрутизации. Обратите внимание на разницу в метрике между этим выходным и предыдущим выходным. Также обратите внимание, что маска теперь составляет 24 бита, в отличие от 32 битов, ранее объявленных.
```
R1#show ip route ospf
O    192.168.1.0 [110/10] via 10.53.0.2, 00:12:26, GigabitEthernet0/1
```
> c.	Введите команду __show ip route ospf__ на маршрутизаторе R2. Единственная информация о маршруте OSPF должна быть распространяемый по умолчанию маршрут R1.
```
R2#show ip route ospf
O*E2 0.0.0.0/0 [110/1] via 10.53.0.1, 00:17:22, GigabitEthernet0/1
```
> d.	Запустите Ping до адреса интерфейса R1 Loopback 1 из R2. Выполнение команды ping должно быть успешным.
```
R2#ping 172.16.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.16.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/1/5 ms
```

##
#### Почему стоимость OSPF для маршрута по умолчанию отличается от стоимости OSPF в R1 для сети 192.168.1.0/24?
Маршрут, обозначенный Е2, является внешним и, соответственно, его стоимость не меняется и равна 1.<br>
А стоимость сети 192.168.1.0/24 для R1 вычисляется, в зависимости от пропуской стоимости линков.


Файл проекта по лабораторной работе [здесь](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab10/files/Lab_10.pkt)
