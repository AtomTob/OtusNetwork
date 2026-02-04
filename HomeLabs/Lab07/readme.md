## Лабораторная работа. Развертывание коммутируемой сети с резервными каналами

#### Топология

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab07/files/topo.jpg?raw=true "Схема сети")

#### Таблица адресации

|Устройство	|Интерфейс	|IP-адрес	|Маска подсети|
| ------------- |:------------------:|------------- |------------- |
|S1	|VLAN 1	|192.168.1.1	|255.255.255.0|
|S2	|VLAN 1	|192.168.1.2	|255.255.255.0|
|S3	|VLAN 1	|192.168.1.3	|255.255.255.0|

##

### Задачи
#### Часть 1:	Создание сети и настройка основных параметров устройства.
##### Шаг 1:	Создайте сеть согласно топологии.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab07/files/topo1.jpg?raw=true)

##
##### Шаг 2:	Выполните инициализацию и перезагрузку коммутаторов.

##### Шаг 3:	Настройте базовые параметры каждого коммутатора.

Все 3 коммутатора настроены, согласно скрипту ниже, меняется лишь hostname, ip-адрес и время:

```
hostname S1
no ip domain-lookup 
enable secret 0 class
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
int vl 1
ip add 192.168.1.1 255.255.255.0
no shut
end
clock set 20:00:00 04 feb 2026
copy running-config startup-config
```

##
##### Шаг 4:	Проверьте связь.

Проверьте способность компьютеров обмениваться эхо-запросами.

> Успешно ли выполняется эхо-запрос от коммутатора S1 на коммутатор S2?<br>
> Успешно ли выполняется эхо-запрос от коммутатора S1 на коммутатор S3?<br>

Да<br>
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab07/files/ping_from_s1.jpg?raw=true)

> Успешно ли выполняется эхо-запрос от коммутатора S2 на коммутатор S3?

Да<br>
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab07/files/ping_from_s2.jpg?raw=true)

##

#### Часть 2:	Определение корневого моста
##### Шаг 1:	Отключите все порты на коммутаторах.
##### Шаг 2:	Настройте подключенные порты в качестве транковых.
##### Шаг 3:	Включите порты F0/2 и F0/4 на всех коммутаторах.

Для всех 3-х шагов выполняется одинаковый скрипт на каждом коммутаторе

```
int ran f0/1-24
shut
ex
int ran f0/1-4
sw m trunk
ex
int ran f0/2,f0/4
no shut
```

##
##### Шаг 4:	Отобразите данные протокола spanning-tree.

```
S1#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.C9D8.3CD1
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0001.C9D8.3CD1
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/4            Desg FWD 19        128.4    P2p
```

```
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.C9D8.3CD1
             Cost        19
             Port        2(FastEthernet0/2)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00D0.BCD5.844D
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Root FWD 19        128.2    P2p
Fa0/4            Desg FWD 19        128.4    P2p
```

```
S3#sh sp
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.C9D8.3CD1
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00E0.8FE5.B405
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Root FWD 19        128.4    P2p
```

Выведем на всех 3-х коммутаторах результат команды __show spanning-tree__.
Из вывода команд видно, что корневым мостом является коммутатор S1.

