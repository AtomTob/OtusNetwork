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




