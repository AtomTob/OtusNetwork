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
R1(config)#banner motd !!!!!!!!!!!!enter only for MA, no MAX or MAXIM!!!!!!<br>
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
S1(config)#banner motd !!!!!!!!!!!!enter only for MA, no MAX or MAXIM!!!!!!
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
S2(config)#banner motd !!!!!!!!!!!!enter only for MA, no MAX or MAXIM!!!!!!
S2(config)#clock timezone EKB 5 0
S2(config)#clock ?
  timezone  Configure time zone
```
Задан только часовой пояс, остальные параметры установки времени недоступны.


