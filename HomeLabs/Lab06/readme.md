## Лабораторная работа - Внедрение маршрутизации между виртуальными локальными сетями

#### Топология

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab06/files/Topo.png?raw=true "Схема сети")

#### Таблица адресации

|Устройство	|Интерфейс	|IP-адрес	|Маска подсети	|Шлюз по умолчанию|
| ------------- |:------------------:|------------- |------------- |-------------|
|R1	|G0/0/1.10	|192.168.10.1	|255.255.255.0	|—|
|R1	|G0/0/1.20	|192.168.20.1	|255.255.255.0	|—|
|R1	|G0/0/1.30	|192.168.30.1	|255.255.255.0	|—|
|R1	|G0/0/1.1000|	—	|—	|—|
|S1	|VLAN 10	|192.168.10.11	|255.255.255.0	|192.168.10.1|
|S2	|VLAN 10	|192.168.10.12	|255.255.255.0	|192.168.10.1|
|PC-A	|NIC	|192.168.20.3	|255.255.255.0	|192.168.20.1|
|PC-B	|NIC	|192.168.30.3	|255.255.255.0	|192.168.30.1|

#### Таблица VLAN

|VLAN	|Имя	|Назначенный интерфейс|
| ------------- |:------------------:|------------- |
|10	|Управление|	S1: VLAN 10 <br> S2: VLAN 10 |
|20	|Sales	|S1: F0/6|
|30	|Operations|	S2: F0/18|
|999	|Parking_Lot	|С1: F0/2-4, F0/7-24, G0/1-2 <br> С2: F0/2-17, F0/19-24, G0/1-2|
|1000	|Собственная	|—|

##

### Задачи
#### Часть 1. Создание сети и настройка основных параметров устройства.
##### Шаг 1. Создайте сеть согласно топологии.
##### Шаг 2. Настройте базовые параметры для маршрутизатора.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab06/files/Topo1.png?raw=true "Схема сети")

> a.	Подключитесь к маршрутизатору с помощью консоли и активируйте привилегированный режим EXEC. Откройте окно конфигурации.<br>
> b.	Войдите в режим конфигурации.<br>
> c.	Назначьте маршрутизатору имя устройства.<br>
> d.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.<br>
> e.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.<br>
> f.	Назначьте ciscoв качестве пароля консоли и включите вход в систему по паролю.<br>
> g.	Установите cisco в качестве пароля виртуального терминала и активируйте вход.<br>
> h.	Зашифруйте открытые пароли.<br>
> i.	Создайте баннер с предупреждением о запрете несанкционированного доступа к устройству.<br>
> j.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.<br>
> k.	Настройте на маршрутизаторе время.<br>

Router(config)#hostname R1<br>
R1(config)#no ip domain-lookup <br>
R1(config)#enable secret 0 class<br>
R1(config)#line con 0<br>
R1(config-line)#password cisco<br>
R1(config-line)#login<br>
R1(config-line)#exit<br>
R1(config)#line vty 0 15<br>
R1(config-line)#password cisco<br>
R1(config-line)#login<br>
R1(config-line)#exit<br>
R1(config)#service password-encryption<br>
R1(config)#banner motd ^C!!!!!!!!!!!!enter only for MA, no MAX or MAXIM!!!!!!^C<br>
R1(config)#clock timezone EKB 5 0<br>
R1(config)#clock ?<br>
  timezone  Configure time zone<br>

Задан только часовой пояс, остальные параметры установки времени недоступны.

##

#### Шаг 3. Настройте базовые параметры каждого коммутатора.

##### Настройки первого коммутатора
```
Switch(config)#hostname S1
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
S1(config)#clock timezone EKB 5 0
S1(config)#clock ?
  timezone  Configure time zone
```
Задан только часовой пояс, остальные параметры установки времени недоступны.

##### Настройки второго коммутатора

```
Switch(config)#hostname S2
S2(config)#no ip domain-lookup 
S2(config)#enable secret 0 class
S2(config)#line con 0
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
S2(config)#line vty 0 15
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
S2(config)#service password-encryption
S2(config)#banner motd ^C!!!!!!!!!!!!enter only for MA, no MAX or MAXIM!!!!!!^C
S2(config)#clock timezone EKB 5 0
S2(config)#clock ?
  timezone  Configure time zone
```
Задан только часовой пояс, остальные параметры установки времени недоступны.

##
#### Шаг 4. Настройте узлы ПК.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab06/files/IP_Config.png?raw=true "IP Config")

##
#### Часть 2. Создание сетей VLAN и назначение портов коммутатора
##### Шаг 1. Создайте сети VLAN на коммутаторах.

> a.	Создайте и назовите необходимые VLAN на каждом коммутаторе из таблицы выше. Откройте окно конфигурации<br>
> b.	Настройте интерфейс управления и шлюз по умолчанию на каждом коммутаторе, используя информацию об IP-адресе в таблице адресации. <br>
> c.	Назначьте все неиспользуемые порты коммутатора VLAN Parking_Lot, настройте их для статического режима доступа и административно деактивируйте их.<br>


Настроен коммутатор S1
```
S1(config)#vl 10
S1(config-vlan)#name Management
S1(config-vlan)#ex
S1(config)#vl 20
S1(config-vlan)#name Sales
S1(config-vlan)#ex
S1(config)#vl 999
S1(config-vlan)#name Parking_Lot
S1(config-vlan)#int vl 10
S1(config-if)#
%LINK-5-CHANGED: Interface Vlan10, changed state to up
S1(config-if)#ip add 192.168.10.11 255.255.255.0
S1(config-if)#ex
S1(config)#ip default-gateway 192.168.10.1
S1(config)#int ran F0/2-4, F0/7-24, G0/1-2
S1(config-if-range)#sw m a
S1(config-if-range)#sw a vl 999
S1(config-if-range)#sw noneg
S1(config-if-range)#shut
```

Настроен коммутатор S2
```
S2(config)#vlan 10
S2(config-vlan)#name Management
S2(config-vlan)#exit
S2(config)#vlan 30
S2(config-vlan)#name Operations
S2(config-vlan)#exit
S2(config)#vlan 999
S2(config-vlan)#name Parking_Lot
S2(config-vlan)#exit
S2(config)#int vl 10
S2(config-if)#
%LINK-5-CHANGED: Interface Vlan10, changed state to up
S2(config-if)#ip add 192.168.10.12 255.255.255.0
S2(config-if)#exit
S2(config)#ip default-gateway 192.168.10.1
S2(config)#int ran F0/2-17, F0/19-24, G0/1-2
S2(config-if-range)#sw m a
S2(config-if-range)#sw a vl 999
S2(config-if-range)#sw noneg
S2(config-if-range)#shut
```

##
#### Шаг 2. Назначьте сети VLAN соответствующим интерфейсам коммутатора.

> a.	Назначьте используемые порты соответствующей VLAN (указанной в таблице VLAN выше) и настройте их для режима статического доступа.

Настроен коммутатор S1
```
S1(config)#int f0/6
S1(config-if)#sw m a
S1(config-if)#sw a vl 20
```

Настроен коммутатор S2
```
S2(config)#int f0/18
S2(config-if)#sw m a
S2(config-if)#sw a vl 30
```

> b.	Убедитесь, что VLAN назначены на правильные интерфейсы.

Убеждаемся на примере коммутатора S1
```
S1#sh vl br

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/5
10   Management                       active    
20   Sales                            active    Fa0/6
999  Parking_Lot                      active    Fa0/2, Fa0/3, Fa0/4, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active  
```


Убеждаемся на примере коммутатора S2
```
S2#sh vl br

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1
10   Management                       active    
30   Operations                       active    Fa0/18
999  Parking_Lot                      active    Fa0/2, Fa0/3, Fa0/4, Fa0/5
                                                Fa0/6, Fa0/7, Fa0/8, Fa0/9
                                                Fa0/10, Fa0/11, Fa0/12, Fa0/13
                                                Fa0/14, Fa0/15, Fa0/16, Fa0/17
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24, Gig0/1, Gig0/2
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active 
```

##
#### Часть 3. Конфигурация магистрального канала стандарта 802.1Q между коммутаторами.
##### Шаг 1. Вручную настройте магистральный интерфейс F0/1 на коммутаторах S1 и S2.

> a.	Настройка статического транкинга на интерфейсе F0/1 для обоих коммутаторов.<br>
> b.	Установите native VLAN 1000 на обоих коммутаторах.<br>
> c.	Укажите, что VLAN 10, 20, 30 и 1000 могут проходить по транку.<br>
> d.	Проверьте транки, native VLAN и разрешенные VLAN через транк.<br>

Настройки и их проверка коммутатора S1
```
S1(config)#int f0/1
S1(config-if)#sw m tr
S1(config-if)#sw noneg
S1(config-if)#sw tr nat vl 1000
S1(config-if)#sw tr al vl 10,20,999

S1#
S1#sh int tr
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/1       10,20,999

Port        Vlans allowed and active in management domain
Fa0/1       10,20,999

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       999
```

Настройки и их проверка коммутатора S2
```
S2(config)#int f0/1
S2(config-if)#sw m tr
S2(config-if)#sw noneg
S2(config-if)#sw tr nat vl 1000
S2(config-if)#sw tr al vl 10,30,999

S2#sh int tr
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/1       10,30,999

Port        Vlans allowed and active in management domain
Fa0/1       10,30,999

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       10,30,999
```

##
#### Шаг 2. Вручную настройте магистральный интерфейс F0/5 на коммутаторе S1.

> a.	Настройте интерфейс S1 F0/5 с теми же параметрами транка, что и F0/1. Это транк до маршрутизатора.<br>
> b.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.<br>
> c.	Проверка транкинга.<br>

```
S1(config)#int f0/5
S1(config-if)#sw m tr
S1(config-if)#sw noneg
S1(config-if)#sw tr nat vl 1000
S1(config-if)#sw tr al vl 10,20,999
S1(config-if)#^Z
S1#
%SYS-5-CONFIG_I: Configured from console by console
sh int tr
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/1       10,20,999

Port        Vlans allowed and active in management domain
Fa0/1       10,20,999

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       10,20,999
```

> Что произойдет, если G0/0/1 на R1 будет отключен?

Порт Fa0/5 не появится в списке транков (собственно, что мы и наблюдаем, т.к. по умолчанию порты выключены и транк не поднимается).

##
#### Часть 4. Настройка маршрутизации между сетями VLAN
##### Шаг 1. Настройте маршрутизатор.

> a.	При необходимости активируйте интерфейс G0/0/1 на маршрутизаторе.

```
R1(config)#int g0/1
R1(config-if)#no shut
```

> b.	Настройте подинтерфейсы для каждой VLAN, как указано в таблице IP-адресации. Все подинтерфейсы используют инкапсуляцию 802.1Q. Убедитесь, что подинтерфейсу для native VLAN не назначен IP-адрес. Включите описание для каждого подинтерфейса.
```
R1(config)#int g0/1.10
R1(config-subif)#encap dot 10
R1(config-subif)#desc SubInterface10
R1(config-subif)#ip add 192.168.10.1 255.255.255.0

R1(config)#int g0/1.20
R1(config-subif)#encap dot 20
R1(config-subif)#desc SubInterface20
R1(config-subif)#ip add 192.168.20.1 255.255.255.0

R1(config)#int g0/1.30
R1(config-subif)#enc dot 30
R1(config-subif)#desc SubInterface30
R1(config-subif)#ip add 192.168.30.1 255.255.255.0
R1(config-subif)#ex
R1(config)#int g0/1.20

R1(config)#int g0/1.1000
R1(config-subif)#desc NativeVLAN_1000
R1(config-subif)#encap dot 1000 native


R1#sh run
!
interface GigabitEthernet0/1.1000
 description NativeVLAN_1000
 encapsulation dot1Q 1000 native
 no ip address
!
```

> c.	Убедитесь, что вспомогательные интерфейсы работают.

```
R1#sh ip int br
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0     unassigned      YES unset  administratively down down 
GigabitEthernet0/1     unassigned      YES unset  up                    up 
GigabitEthernet0/1.10  192.168.10.1    YES manual up                    up 
GigabitEthernet0/1.20  192.168.20.1    YES manual up                    up 
GigabitEthernet0/1.30  192.168.30.1    YES manual up                    up 
GigabitEthernet0/1.1000unassigned      YES unset  up                    up 
Vlan1                  unassigned      YES unset  administratively down down
```

##
#### Часть 5. Проверьте, работает ли маршрутизация между VLAN
##### Шаг 1. Выполните следующие тесты с PC-A. Все должно быть успешно.

> a.	Отправьте эхо-запрос с PC-A на шлюз по умолчанию.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab06/files/Ping_Gate1.png?raw=true "Ping Gateway1")

> b.	Отправьте эхо-запрос с PC-A на PC-B.

#### Чтобы я не делал - запросы до РС-В не проходят, прошу помощи в решении.

> c.	Отправьте команду ping с компьютера PC-A на коммутатор S2.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab06/files/Ping_S2.png?raw=true)



