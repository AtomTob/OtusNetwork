## Лабораторная работа. Доступ к сетевым устройствам по протоколу SSH

#### Топология

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab05/files/Screenshot_1.png?raw=true "Схема сети")

#### Таблица адресации

|Устройство	|Интерфейс	|IP-адрес	|Маска подсети	|Шлюз по умолчанию|
| ------------- |:------------------:|------------- |------------- |-------------|
|R1	|G0/0/1	|192.168.1.1	|255.255.255.0	|—|
|S1	|VLAN 1	|192.168.1.11	|255.255.255.0	|192.168.1.1|
|PC-A	|NIC	|192.168.1.3	|255.255.255.0	|192.168.1.1|

##

### Задачи
#### Часть 1. Настройка основных параметров устройств
##### Шаг 1. Создайте сеть согласно топологии

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab05/files/Jpg01.png?raw=true "Схема сети")

##
##### Шаг 2. Выполните инициализацию и перезагрузку маршрутизатора и коммутатора.

На маршрутизаторе на схеме первоначальная конфигурация.

##
##### Шаг 3. Настройте маршрутизатор.
> a.	Подключитесь к маршрутизатору с помощью консоли и активируйте привилегированный режим EXEC.

> b.	Войдите в режим конфигурации.

> c.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

no ip domain-lookup

> d.	Назначьте **class** в качестве зашифрованного пароля привилегированного режима EXEC.

Router(config)#enable secret 0 class

> e.	Назначьте **cisco** в качестве пароля консоли и включите вход в систему по паролю.

Router(config)#line con 0<br>
Router(config-line)# password 7 0822455D0A16<br>
Router(config-line)# login<br>

> f.	Назначьте ciscoв качестве пароля VTY и включите вход в систему по паролю.

Router(config)#line vty 0 15<br>
Router(config-line)# password 7 0822455D0A16<br>
Router(config-line)# login<br>

> g.	Зашифруйте открытые пароли.

Все пароли вводились хешами, за исключением в привилегированный режим EXEC.<br>
Но при необходимости можно использовать команду __Router(config)#service password-encryption__.

> h.	Создайте баннер, который предупреждает о запрете несанкционированного доступа.

Router(config)#banner motd !!!!!!!!!!!!enter only for MA, no MAX or MAXIM!!!!!!

> i.	Настройте и активируйте на маршрутизаторе интерфейс G0/0/1, используя информацию, приведенную в таблице адресации.

Router(config)#int g0/1<br>
Router(config-if)#ip address 192.168.1.1 255.255.255.0<br>
Router(config-if)#no shutdown<br>

> j.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.

Router#wr<br>
Building configuration...<br>
[OK]

##
##### Шаг 4. Настройте компьютер PC-A.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab05/files/ConfigPC0.png?raw=true "Настройка РС")

##
##### Шаг 5. Проверьте подключение к сети.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab05/files/Ping_R1.png?raw=true "Проверка подключения")

##
#### Часть 2. Настройка маршрутизатора для доступа по протоколу SSH
##### Шаг 1. Настройте аутентификацию устройств.

> a.	Задайте имя устройства.<br>
> b.	Задайте домен для устройства.

Router(config)#hostname R1<br>
R1(config)#ip domain-name home.local

##
##### Шаг 2. Создайте ключ шифрования с указанием его длины.

R1(config)#crypto key generate rsa general-keys modulus 2048<br>
The name for the keys will be: R1.home.local

% The key modulus size is 2048 bits<br>
% Generating 2048 bit RSA keys, keys will be non-exportable...[OK]<br>
*Mar 1 1:11:46.86: %SSH-5-ENABLED: SSH 1.99 has been enabled<br>

R1(config)#ip ssh version 2

##
##### Шаг 3. Создайте имя пользователя в локальной базе учетных записей.

> Настройте имя пользователя, используя admin в качестве имени пользователя и Adm1nP@55 в качестве пароля.

R1(config)#username admin privilege 15 secret Adm1nP@55 

##
##### Шаг 4. Активируйте протокол SSH на линиях VTY.

> a.	Активируйте протоколы Telnet и SSH на входящих линиях VTY с помощью команды transport input.<br>
> b.	Измените способ входа в систему таким образом, чтобы использовалась проверка пользователей по локальной базе учетных записей.

R1(config)#line vty 0 15<br>
R1(config-line)#transport input all<br>
R1(config-line)#login local 

> Т.к. в Cisco Packet Tracer не применяется команда активации обоих протоколов, то включаем все, что некорретно с точки зрения сетевой безопасности.

##
##### Шаг 5. Сохраните текущую конфигурацию в файл загрузочной конфигурации.

R1#wr<br>
Building configuration...<br>
[OK]

##
##### Шаг 6. Установите соединение с маршрутизатором по протоколу SSH.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab05/files/SSH_Connect.png?raw=true "Проверка подключения")

Подключение установлено.


##
#### Часть 3. Настройка коммутатора для доступа по протоколу SSH
##### Шаг 1. Настройте основные параметры коммутатора.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab05/files/PC_SW.png?raw=true "PC_SW")

> a.	Подключитесь к коммутатору с помощью консольного подключения и активируйте привилегированный режим EXEC.<br>
> b.	Войдите в режим конфигурации.<br>
> c.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.<br>
> d.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.<br>
> e.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.<br>
> f.	Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.<br>
> g.	Зашифруйте открытые пароли.<br>
> h.	Создайте баннер, который предупреждает о запрете несанкционированного доступа.<br>

Switch(config)#no ip domain-lookup<br>
Switch(config)#enable secret 0 class<br>
Switch(config)#line con 0<br>
Switch(config-line)#password cisco<br>
Switch(config-line)#login<br>
Switch(config-line)#exit<br>
Switch(config)#line vty 0 15<br>
Switch(config-line)#password cisco<br>
Switch(config-line)#login<br>
Switch(config-line)#exit<br>
Switch(config)#service password-encryption<br>
Switch(config)#banner motd !!!!!!!!!!!!enter only for MA, no MAX or MAXIM!!!!!!<br>

> i.	Настройте и активируйте на коммутаторе интерфейс VLAN 1, используя информацию, приведенную в таблице адресации.

Switch(config)#int vl 1<br>
Switch(config-if)#ip add 192.168.1.11 255.255.255.0<br>
Switch(config-if)#no shut<br>

> j.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.

Switch#wr<br>
Building configuration...<br>
[OK]<br>

##
##### Шаг 2. Настройте коммутатор для соединения по протоколу SSH.

> a.	Настройте имя устройства, как указано в таблице адресации.<br>
> b.	Задайте домен для устройства.<br>
> c.	Создайте ключ шифрования с указанием его длины.<br>
> d.	Создайте имя пользователя в локальной базе учетных записей.<br>
> e.	Активируйте протоколы Telnet и SSH на линиях VTY.<br>
> f.	Измените способ входа в систему таким образом, чтобы использовалась проверка пользователей по локальной базе учетных записей.<br>

Switch(config)#hostname S1<br>
S1(config)#ip domain-name home.local<br>
S1(config)#crypto key generate rsa general-keys modulus 2048<br>
The name for the keys will be: S1.home.local<br>
% The key modulus size is 2048 bits<br>
% Generating 2048 bit RSA keys, keys will be non-exportable...[OK]<br>
*Mar 1 2:45:29.412: %SSH-5-ENABLED: SSH 1.99 has been enabled<br>
S1(config)#ip ssh version 2<br>
S1(config)#username admin privilege 15 secret Adm1nP@55<br>
S1(config)#line vty 0 15<br>
S1(config-line)#transport input all<br>
S1(config-line)#login local <br>

##
##### Шаг 3. Установите соединение с коммутатором по протоколу SSH.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab05/files/SSH_Connect2.png?raw=true "SSH Connect 2")

Подключение установлено.

##
#### Часть 4. Настройка протокола SSH с использованием интерфейса командной строки (CLI) коммутатора
##### Шаг 1. Посмотрите доступные параметры для клиента SSH в Cisco IOS.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab05/files/ssh%20command.png?raw=true "SSH Command")

##
##### Шаг 2. Установите с коммутатора S1 соединение с маршрутизатором R1 по протоколу SSH.

> a.	Чтобы подключиться к маршрутизатору R1 по протоколу SSH, введите команду –l admin. Это позволит вам войти в систему под именем admin. При появлении приглашения введите в качестве пароля Adm1nP@55

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab05/files/SSH_Connect3.png?raw=true "SSH Connect 3")

> b.	Чтобы вернуться к коммутатору S1, не закрывая сеанс SSH с маршрутизатором R1, нажмите комбинацию клавиш Ctrl+Shift+6. Отпустите клавиши Ctrl+Shift+6 и нажмите x. Отображается приглашение привилегированного режима EXEC коммутатора.<br>
> c.	Чтобы вернуться к сеансу SSH на R1, нажмите клавишу Enter в пустой строке интерфейса командной строки. Чтобы увидеть окно командной строки маршрутизатора, нажмите клавишу Enter еще раз.<br>

__Выполнение данных действий не применяется в данном режиме по неизвестным мне причинам :(<br>
Наверное, стоит перейти на EVE-NG...__

> d.	Чтобы завершить сеанс SSH на маршрутизаторе R1, введите в командной строке маршрутизатора команду exit.

![alt-текст](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab05/files/SSH_Exit.png?raw=true "SSH Exit")

##
### Вопрос для повторения

> Как предоставить доступ к сетевому устройству нескольким пользователям, у каждого из которых есть собственное имя пользователя?

Создать несколько пользователей, использую команду __username__.<br>
Настроить доступ по SSH.

## 	Сводная таблица по интерфейсам маршрутизаторов

|Модель маршрутизатора	|Интерфейс Ethernet № 1|	Интерфейс Ethernet № 2|	Последовательный интерфейс № 1|	Последовательный интерфейс № 2|
| ------------- |:------------------:|------------- |------------- |-------------|
|1 800	|Fast Ethernet 0/0 (F0/0)|	Fast Ethernet 0/1 (F0/1)|	Serial 0/0/0 (S0/0/0)|	Serial 0/0/1 (S0/0/1)|
|1900|	Gigabit Ethernet 0/0 (G0/0)	|Gigabit Ethernet 0/1 (G0/1)	|Serial 0/0/0 (S0/0/0)	|Serial 0/0/1 (S0/0/1)|
|2801|	Fast Ethernet 0/0 (F0/0)|	Fast Ethernet 0/1 (F0/1)|	Serial 0/1/0 (S0/1/0)|	Serial 0/1/1 (S0/1/1)|
|2811|	Fast Ethernet 0/0 (F0/0)|	Fast Ethernet 0/1 (F0/1)|	Serial 0/0/0 (S0/0/0)	|Serial 0/0/1 (S0/0/1)|
|2900|	Gigabit Ethernet 0/0 (G0/0)|	Gigabit Ethernet 0/1 (G0/1)|	Serial 0/0/0 (S0/0/0)|	Serial 0/0/1 (S0/0/1)|
|4221|	Gigabit Ethernet 0/0/0 (G0/0/0)|	Gigabit Ethernet 0/0/1 (G0/0/1)|	Serial 0/1/0 (S0/1/0)|	Serial 0/1/1 (S0/1/1)|
|4300|	Gigabit Ethernet 0/0/0 (G0/0/0)|	Gigabit Ethernet 0/0/1 (G0/0/1)|	Serial 0/1/0 (S0/1/0)|	Serial 0/1/1 (S0/1/1)|

__Примечание.__ Чтобы определить конфигурацию маршрутизатора, можно посмотреть на интерфейсы и установить тип маршрутизатора и количество его интерфейсов. Перечислить все комбинации конфигураций для каждого класса маршрутизаторов невозможно. Эта таблица содержит идентификаторы для возможных комбинаций интерфейсов Ethernet и последовательных интерфейсов на устройстве. Другие типы интерфейсов в таблице не представлены, хотя они могут присутствовать в данном конкретном маршрутизаторе. В качестве примера можно привести интерфейс ISDN BRI. Строка в скобках — это официальное сокращение, которое можно использовать в командах Cisco IOS для обозначения интерфейса.

Файл проекта по лабораторной работе [здесь](https://github.com/AtomTob/OtusNetwork/blob/main/HomeLabs/Lab05/files/Lab05.pkt)
