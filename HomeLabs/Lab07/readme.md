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
             Address     0001.96E4.5401
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0001.96E4.5401
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/4            Desg FWD 19        128.4    P2p
```

```
S2#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.96E4.5401
             Cost        19
             Port        2(FastEthernet0/2)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     000C.85AB.3795
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Root FWD 19        128.2    P2p
Fa0/4            Desg FWD 19        128.4    P2p
```

```
S3#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.96E4.5401
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00D0.9768.33C5
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Root FWD 19        128.4    P2p
```

Из вывода команд видно, что корневым мостом является коммутатор S1.

Схема с ролями и состоянием (Sts) активных портов на каждом коммутаторе в топологии.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab07/files/Scheme1.jpg?raw=true)

##
#### С учетом выходных данных, поступающих с коммутаторов, ответьте на следующие вопросы.

> Какой коммутатор является корневым мостом?

__S1__

> Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста?

__Потому что у него меньшее, чем у остальных коммутаторов, значение МАС-адреса при одинаковом значении приоритета (ID Priority)__

> Какие порты на коммутаторе являются корневыми портами?

__У коммутатора S2 корневой порт - Fa0/2__ <br>
__У коммутатора S3 корневой порт - Fa0/4__

> Какие порты на коммутаторе являются назначенными портами? 

__У корневого коммутатора S1 назначенные порты - Fa0/2 и Fa0/4__ <br>
__У коммутатора S2 назначенный порт - Fa0/4__ <br>
__У коммутатора S3 назначенный порт отсутствует__ <br>

> Какой порт отображается в качестве альтернативного и в настоящее время заблокирован?

__Альтернативный, и поэтому заблокированный, порт Fa0/2 принадлежит коммутатору S3__

> Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?
