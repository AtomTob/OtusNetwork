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

__Т.к. маршрут с порта Fa0/4 коммутатора S3 имеет меньшую стоимость(метрику) до корневого коммутатора и этот порт уже является корневым - 19 против 38__ <br>
__Т.к. другой порт по этому линку со стороны  коммутатора S2 не является корневым портом__ <br>


##

#### Часть 3:	Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов
##### Шаг 1:	Определите коммутатор с заблокированным портом.

Выполним команду __show spanning-tree__ на обоих коммутаторах некорневого моста. <br>
Как видим из примера, протокол __STP__ блокирует порт Fa0/2 на коммутаторе S3.

```
S2#sh spa
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
S3#sh spa
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

##
##### Шаг 2:	Изменим стоимость порта.

Помимо заблокированного порта, единственным активным портом на этом коммутаторе является порт, выделенный в качестве порта корневого моста. <br>
Уменьшим стоимость этого порта корневого моста до 18, выполнив команду __spanning-tree vlan 1 cost 18__ режима конфигурации интерфейса.

```
S3(config)#int f0/4
S3(config-if)#spanning-tree vlan 1 cost 18
```

##
##### Шаг 3:	Просмотрим изменения протокола spanning-tree.

Повторно выполним команду __show spanning-tree__ на обоих коммутаторах некорневого моста. <br>
Обратим внимание, что ранее заблокированный порт (S3 – F0/2) теперь является корневым портом, и протокол spanning-tree теперь блокирует порт на другом коммутаторе некорневого моста (S2 – F0/4).

```
S2#sh spa
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
Fa0/4            Altn BLK 19        128.4    P2p
```

```
S3#sh spa
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.96E4.5401
             Cost        18
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00D0.9768.33C5
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/4            Root FWD 18        128.4    P2p
```

> Почему протокол spanning-tree заменяет ранее заблокированный порт на назначенный порт и блокирует порт, который был назначенным портом на другом коммутаторе?

__Потому что стоимость маршрута из сегмента сети между S2 и S3 теперь меньше через коммутатор S3 - 18 против 19__

##
##### Шаг 4:	Удалить изменения стоимости порта.

> a.	Выполните команду __no spanning-tree vlan 1 cost 18__ режима конфигурации интерфейса, чтобы удалить запись стоимости, созданную ранее.

```
S3(config)#int f0/4
S3(config-if)#no spanning-tree vlan 1 cost 18
```

> b.	Повторно выполните команду show spanning-tree, чтобы подтвердить, что протокол STP сбросил порт на коммутаторе некорневого моста, вернув исходные настройки порта. <br>
> Протоколу STP требуется примерно 30 секунд, чтобы завершить процесс перевода порта.

Настойки портов вернулись в прежнее значение.


##

#### Часть 4:	Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов.
##### 

> a.	Включите порты F0/1 и F0/3 на всех коммутаторах.

На всех коммутаторах выполняется нижеследующий скрипт:
```
conf t
int ran f0/1,f0/3
no shut
end
```

> b.	Подождите 30 секунд, чтобы протокол STP завершил процесс перевода порта, после чего выполните команду show spanning-tree на коммутаторах некорневого моста.<br>
> Обратим внимание, что порт корневого моста переместился на порт с меньшим номером, связанный с коммутатором корневого моста, и заблокировал предыдущий порт корневого моста.

```
S2#sh spa
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.96E4.5401
             Cost        19
             Port        1(FastEthernet0/1)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     000C.85AB.3795
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/3            Desg FWD 19        128.3    P2p
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/1            Root FWD 19        128.1    P2p
Fa0/4            Desg FWD 19        128.4    P2p
```

```
S3#sh spa
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.96E4.5401
             Cost        19
             Port        3(FastEthernet0/3)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00D0.9768.33C5
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/3            Root FWD 19        128.3    P2p
Fa0/1            Altn BLK 19        128.1    P2p
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Altn BLK 19        128.4    P2p
```

> Какой порт выбран протоколом STP в качестве порта корневого моста на каждом коммутаторе некорневого моста?

__У коммутатора S2 корневой порт - Fa0/1__ <br>
__У коммутатора S3 корневой порт - Fa0/3__

> Почему протокол STP выбрал эти порты в качестве портов корневого моста на этих коммутаторах?

Приоритет порта по умолчанию - 128; поэтому протокол STP использовал номер порта для устранения петли. <br>
Поэтому протокол STP выбрал порт с меньшим номером Fa0/1 в качестве корневого порта для коммутатора S2 и заблокировал порт с большим номером, используя резервный путь к корневому мосту.


##
#### Вопросы для повторения

> 1.	Какое значение протокол STP использует первым после выбора корневого моста, чтобы определить выбор порта?

Стоимость пути к корневому мосту (суммарная стоимость всех каналов от конкретного порта до корневого) - выбирается путь с наименьшей стоимостью.

> 2.	Если первое значение на двух портах одинаково, какое следующее значение будет использовать протокол STP при выборе порта?

Используется Bridge ID, которое состоит из приоритета (по умолчанию 32768), номера Vlan и МАС-адреса. Приоритет можно изменить.

> 3.	Если оба значения на двух портах равны, каким будет следующее значение, которое использует протокол STP при выборе порта?

В таком случае используется сумма приоритета порта (по умолчанию - 128) и номером порта соседа (например, Fa0/1).


Файл проекта по лабораторной работе [здесь](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab07/files/Lab07.pkt)
