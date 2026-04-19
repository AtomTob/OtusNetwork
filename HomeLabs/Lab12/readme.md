## Лабораторная работа - Настройка NAT для IPv4

### Топология
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab12/files/topo1.jpg?raw=true)

##
### Таблица адресации

|Устройство	|Интерфейс|	IP-адрес	|Маска подсети |
| ------------- |:------------------:|------------- |------------- |
|R1	|G0/0/0	|209.165.200.230|	255.255.255.248|
|R1|	G0/0/1	|192.168.1.1|	255.255.255.0|
|R2	|G0/0/0	|209.165.200.225|	255.255.255.248|
|R2	|Lo1|	209.165.200.1	|255.255.255.224|
|S1|	VLAN 1	|192.168.1.11|	255.255.255.0|
|S2|	VLAN 1	|192.168.1.12|	255.255.255.0|
|PC-A|	NIC|	192.168.1.2|	255.255.255.0|
|PC-B	|NIC	|192.168.1.3	|255.255.255.0|

##
### Часть 1. Создание сети и настройка основных параметров устройства.
#### Шаг 1. Подключите кабели сети согласно приведенной топологии.
![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab12/files/topo2.jpg?raw=true)
#### Шаг 2. Произведите базовую настройку маршрутизаторов.
> a.	Назначьте маршрутизатору имя устройства. <br>
> b.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.<br>
> c.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.<br>
> d.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.<br>
> e.	Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.<br>
> f.	Зашифруйте открытые пароли.<br>
> g.	Создайте баннер с предупреждением о запрете несанкционированного доступа к устройству.<br>

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
```
> h.	Настройте IP-адресации интерфейса, как указано в таблице выше.<br>
> i.	Настройте маршрут по умолчанию. от R2 до  R1.<br>
> j.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.<br>

```
R1(config)#int g0/0
R1(config-if)#ip add 209.165.200.230 255.255.255.248
R1(config-if)#no shut
R1(config)#int g0/1 
R1(config-if)#ip add 192.168.1.1 255.255.255.0
R1(config-if)#no shut
R1(config)#ip route 0.0.0.0 0.0.0.0 209.165.200.225

R2(config)#int g 0/0
R2(config-if)#ip add 209.165.200.225 255.255.255.248
R2(config-if)#no shut
R2(config)#int l1
R2(config-if)#ip add 209.165.200.1 255.255.255.224
R2(config)#ip route
R2(config)#ip route 0.0.0.0 0.0.0.0 209.165.200.230
```
##
#### Шаг 3. Настройте базовые параметры каждого коммутатора.
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
S1(config)#ip domain-name home.local 
S1(config)#crypto key generate rsa general-keys modulus 2048 
S1(config)#ip ssh version 2 
S1(config)#username admin secret 5 $1$mERr$qJb.eHvBN7S590aq.dpRL. 
S1(config)#line vty 0 4 
S1(config-line)#transport input ssh 
S1(config-line)#login local 
S1(config-line)#exit
S1(config)#exit
S1#clock set 21:00:00 24 march 2026
S1(config)#int vl1
S1(config-if)#ip address 192.168.1.11 255.255.255.0
S1(config-if)#no shut
S1(config)#int ran Fa0/2-4, Fa0/7-24
S1(config-if-range)#shut

S2(config)#int vl1
S2(config-if)#ip address 192.168.1.12 255.255.255.0
S2(config-if)#no shut
S2(config)#int ran Fa0/2-17, Fa0/19-24
S2(config-if-range)#shut
```

##
### Часть 2. Настройка и проверка NAT для IPv4.
#### Шаг 1. Настройте NAT на R1, используя пул из трех адресов 209.165.200.226-209.165.200.228. 
> a.	Настройте простой список доступа, который определяет, какие хосты будут разрешены для трансляции. В этом случае все устройства в локальной сети R1 имеют право на трансляцию.
```
R1(config)#access-list 1 permit 192.168.1.0 0.0.0.255
```
> b.	Создайте пул NAT и укажите ему имя и диапазон используемых адресов.
```
R1(config)#ip nat pool PUBLIC_ACCESS 209.165.200.226 209.165.200.228 netmask 255.255.255.248 
```
> c.	Настройте перевод, связывая ACL и пул с процессом преобразования.
```
R1(config)#ip nat inside source list 1 pool PUBLIC_ACCESS
```
> d.	Задайте внутренний (inside) интерфейс. 
```
R1(config)#int g 0/1
R1(config-if)#ip nat inside
```
> e.	Определите внешний (outside) интерфейс.
```
R1(config)#int g0/0
R1(config-if)#ip nat outside
```

##
#### Шаг 2. Проверьте и проверьте конфигурацию. 
> a.	С PC-B, запустите эхо-запрос интерфейса Lo1 (209.165.200.1) на R2. <br>
Изначально запрос не проходил. В процессе трабл-шутинга было выявлено, что не хватало объявления шлюза по умолчанию по коммутаторах и маршрута на маршрутизаторах.
```
R1(config)#ip route 0.0.0.0 0.0.0.0 209.165.200.225
R2(config)#ip route 0.0.0.0 0.0.0.0 209.165.200.230
S2(config)#ip default-gateway 192.168.1.1
S1(config)#ip default-gateway 192.168.1.1

C:\>ping 209.165.200.1
Pinging 209.165.200.1 with 32 bytes of data:
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254

Ping statistics for 209.165.200.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```
На R1 отобразите таблицу NAT на R1 с помощью команды __show ip nat translations__. <br>
```
R1#sh ip nat tran
Pro  Inside global       Inside local       Outside local      Outside global
icmp 209.165.200.226:42  192.168.1.3:42     209.165.200.1:42   209.165.200.1:42
icmp 209.165.200.226:43  192.168.1.3:43     209.165.200.1:43   209.165.200.1:43
icmp 209.165.200.226:44  192.168.1.3:44     209.165.200.1:44   209.165.200.1:44
icmp 209.165.200.226:45  192.168.1.3:45     209.165.200.1:45   209.165.200.1:45
```
> __Во что был транслирован внутренний локальный адрес PC-B?__ <br>

В внутренний глобальный адрес 209.165.200.226

> __Какой тип адреса NAT является переведенным адресом?__ <br>

Dynamic NAT


> b.	С PC-A, запустите  эхо-запрос интерфейса Lo1 (209.165.200.1) на R2. Если эхо-запрос не прошел, выполните отладку. На R1 отобразите таблицу NAT на R1 с помощью команды show ip nat translations. <br><br>
```
C:\>ping 209.165.200.1

Pinging 209.165.200.1 with 32 bytes of data:

Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254

Ping statistics for 209.165.200.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

