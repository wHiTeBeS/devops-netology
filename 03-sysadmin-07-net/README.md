# Домашнее задание к занятию "3.7. Компьютерные сети, лекция 2"

1. Проверьте список доступных сетевых интерфейсов на вашем компьютере. Какие команды есть для этого в Linux и в Windows?
Windows  
```commandline
C:\Users\wHiTe>ipconfig

Настройка протокола IP для Windows


Адаптер Ethernet Ethernet 4:

   Состояние среды. . . . . . . . : Среда передачи недоступна.
   DNS-суффикс подключения . . . . . :

Адаптер Ethernet Ethernet:

   Состояние среды. . . . . . . . : Среда передачи недоступна.
   DNS-суффикс подключения . . . . . :

Адаптер Ethernet Local:

   DNS-суффикс подключения . . . . . :
   IPv4-адрес. . . . . . . . . . . . : 100.64.0.122
   Маска подсети . . . . . . . . . . : 255.255.255.0
   Основной шлюз. . . . . . . . . : 100.64.0.1

Адаптер Ethernet VirtualBox Host-Only Network:

   DNS-суффикс подключения . . . . . :
   Локальный IPv6-адрес канала . . . : fe80::b434:dd79:4fbf:76e5%11
   IPv4-адрес. . . . . . . . . . . . : 192.168.56.1
   Маска подсети . . . . . . . . . . : 255.255.255.0
   Основной шлюз. . . . . . . . . :

Туннельный адаптер Teredo Tunneling Pseudo-Interface:

   DNS-суффикс подключения . . . . . :
   IPv6-адрес. . . . . . . . . . . . : 2001:0:284a:364:3cce:ad1:3c7a:e077
   Локальный IPv6-адрес канала . . . : fe80::3cce:ad1:3c7a:e077%27
   Основной шлюз. . . . . . . . . : ::
```
Linux
```bash
vagrant@vagrant:~$ ip l
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:b1:28:5d brd ff:ff:ff:ff:ff:ff
```

2. Какой протокол используется для распознавания соседа по сетевому интерфейсу? Какой пакет и команды есть в Linux для этого?
Протокол `LDP`, пакет `lldpd`, команда `lldpctl`

3. Какая технология используется для разделения L2 коммутатора на несколько виртуальных сетей? Какой пакет и команды есть в Linux для этого? Приведите пример конфига.
Технология `VLAN (Virtual LAN)`, пакет `vlan`, команда, например `ip`
```bash
vagrant@vagrant:~$ sudo ip link add link eth0 name enp1s0.100 type vlan id 100
vagrant@vagrant:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b1:28:5d brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
       valid_lft 85618sec preferred_lft 85618sec
    inet6 fe80::a00:27ff:feb1:285d/64 scope link
       valid_lft forever preferred_lft forever
3: enp1s0.100@eth0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 08:00:27:b1:28:5d brd ff:ff:ff:ff:ff:ff
```

5. Какие типы агрегации интерфейсов есть в Linux? Какие опции есть для балансировки нагрузки? Приведите пример конфига.  
На данный момент есть два типа реализации агрегации интерфейсов `teaming` и `bonding` 
Режимы bonding
```bash
vagrant@vagrant:~$ modinfo bonding | grep mode:
parm:           mode:Mode of operation; 0 for balance-rr, 1 for active-backup, 2 for balance-xor, 3 for broadcast, 4 for 802.3ad, 5 for balance-tlb, 6 for balance-alb (charp)
```
Политики для отказоустойчивости:
`active-backup` Политика активный-резервный. Только один сетевой интерфейс из объединённых будет активным. Другой интерфейс может стать активным, только в том случае, когда упадёт текущий активный интерфейс. 
`broadcast` Широковещательная политика. Передает всё на все сетевые интерфейсы. Эта политика применяется для отказоустойчивости.  
Остальные политики вместе с отказоусточивостью реализуют распределение нагрузки в разных вариациях.  
Пример конфига `/etc/network/interfaces`
```auto bond0
iface bond0 inet dhcp
   bond-slaves eth0 eth1
   bond-mode active-backup
   bond-miimon 100
   bond-primary eth0 eth1
  ```

6. Сколько IP адресов в сети с маской /29 ? Сколько /29 подсетей можно получить из сети с маской /24. Приведите несколько примеров /29 подсетей внутри сети 10.10.10.0/24.
```bash
vagrant@vagrant:~$  ipcalc -b 10.10.10.0/29
Address:   10.10.10.0
Netmask:   255.255.255.248 = 29
Wildcard:  0.0.0.7
=>
Network:   10.10.10.0/29
HostMin:   10.10.10.1
HostMax:   10.10.10.6
Broadcast: 10.10.10.7
Hosts/Net: 6                     Class A, Private Internet
```
8 адресов - 6 для хостов, адрес сети и широковещательный адрес

```bash
vagrant@vagrant:~$ ipcalc -b 10.10.10.0/24
Address:   10.10.10.0
Netmask:   255.255.255.0 = 24
Wildcard:  0.0.0.255
=>
Network:   10.10.10.0/24
HostMin:   10.10.10.1
HostMax:   10.10.10.254
Broadcast: 10.10.10.255
Hosts/Net: 254
```
(254+2)/8 = 32 подсети с маской /29

```bash
vagrant@vagrant:~$ ipcalc -b 10.10.10.8/29
Address:   10.10.10.8
Netmask:   255.255.255.248 = 29
Wildcard:  0.0.0.7
=>
Network:   10.10.10.8/29
HostMin:   10.10.10.9
HostMax:   10.10.10.14
Broadcast: 10.10.10.15
Hosts/Net: 6                     Class A, Private Internet

vagrant@vagrant:~$ ipcalc -b 10.10.10.16/29
Address:   10.10.10.16
Netmask:   255.255.255.248 = 29
Wildcard:  0.0.0.7
=>
Network:   10.10.10.16/29
HostMin:   10.10.10.17
HostMax:   10.10.10.22
Broadcast: 10.10.10.23
Hosts/Net: 6                     Class A, Private Internet

vagrant@vagrant:~$ ipcalc -b 10.10.10.24/29
Address:   10.10.10.24
Netmask:   255.255.255.248 = 29
Wildcard:  0.0.0.7
=>
Network:   10.10.10.24/29
HostMin:   10.10.10.25
HostMax:   10.10.10.30
Broadcast: 10.10.10.31
Hosts/Net: 6                     Class A, Private Internet
```

7. Задача: вас попросили организовать стык между 2-мя организациями. Диапазоны 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 уже заняты. Из какой подсети допустимо взять частные IP адреса? Маску выберите из расчета максимум 40-50 хостов внутри подсети.
Можно использовать адреса из сети  100.64.0.0/10. Например
```bash
vagrant@vagrant:~$ ipcalc -b 100.64.0.0/26
Address:   100.64.0.0
Netmask:   255.255.255.192 = 26
Wildcard:  0.0.0.63
=>
Network:   100.64.0.0/26
HostMin:   100.64.0.1
HostMax:   100.64.0.62
Broadcast: 100.64.0.63
Hosts/Net: 62
```

9. Как проверить ARP таблицу в Linux, Windows? Как очистить ARP кеш полностью? Как из ARP таблицы удалить только один нужный IP?
Linux
Посмотреть, очистить
```bash
vagrant@vagrant:~$ ip neigh
10.0.2.3 dev eth0 lladdr 52:54:00:12:35:03 STALE
10.0.2.2 dev eth0 lladdr 52:54:00:12:35:02 REACHABLE
vagrant@vagrant:~$ sudo ip neigh flush all
vagrant@vagrant:~$ ip neigh
```
Удалить 1 запись
```bash
vagrant@vagrant:~$ ip neigh
10.0.2.3 dev eth0 lladdr 52:54:00:12:35:03 REACHABLE
10.0.2.2 dev eth0 lladdr 52:54:00:12:35:02 REACHABLE
vagrant@vagrant:~$ sudo ip neigh delete 10.0.2.3 dev eth0
vagrant@vagrant:~$ ip neigh
10.0.2.2 dev eth0 lladdr 52:54:00:12:35:02 REACHABLE 
```
Windows
Посмотреть, очистить
```commandline
C:\WINDOWS\system32>arp -a

Интерфейс: 192.168.56.1 --- 0xb
  адрес в Интернете      Физический адрес      Тип
  192.168.56.255        ff-ff-ff-ff-ff-ff     статический
  224.0.0.2             01-00-5e-00-00-02     статический
  224.0.0.251           01-00-5e-00-00-fb     статический
  239.192.152.143       01-00-5e-40-98-8f     статический
  239.255.255.250       01-00-5e-7f-ff-fa     статический

Интерфейс: 100.64.0.122 --- 0x12
  адрес в Интернете      Физический адрес      Тип
  100.64.0.1            a8-63-7d-d9-82-0c     динамический
  100.64.0.255          ff-ff-ff-ff-ff-ff     статический
  224.0.0.2             01-00-5e-00-00-02     статический
  224.0.0.251           01-00-5e-00-00-fb     статический
  224.0.0.252           01-00-5e-00-00-fc     статический
  224.0.0.253           01-00-5e-00-00-fd     статический
  239.192.152.143       01-00-5e-40-98-8f     статический
  239.255.255.250       01-00-5e-7f-ff-fa     статический
C:\WINDOWS\system32>arp -d *

C:\WINDOWS\system32>arp -a

Интерфейс: 100.64.0.122 --- 0x12
  адрес в Интернете      Физический адрес      Тип
  100.64.0.1            a8-63-7d-d9-82-0c     динамический
  224.0.0.2             01-00-5e-00-00-02     статический
```
Удалить 1 запись
```commandline
C:\WINDOWS\system32>arp -a

Интерфейс: 192.168.56.1 --- 0xb
  адрес в Интернете      Физический адрес      Тип
  192.168.56.255        ff-ff-ff-ff-ff-ff     статический
  224.0.0.251           01-00-5e-00-00-fb     статический
  239.192.152.143       01-00-5e-40-98-8f     статический
  239.255.255.250       01-00-5e-7f-ff-fa     статический

Интерфейс: 100.64.0.122 --- 0x12
  адрес в Интернете      Физический адрес      Тип
  100.64.0.1            a8-63-7d-d9-82-0c     динамический
  100.64.0.255          ff-ff-ff-ff-ff-ff     статический
  224.0.0.2             01-00-5e-00-00-02     статический
  224.0.0.251           01-00-5e-00-00-fb     статический
  224.0.0.252           01-00-5e-00-00-fc     статический
  239.192.152.143       01-00-5e-40-98-8f     статический
  239.255.255.250       01-00-5e-7f-ff-fa     статический
  
 C:\WINDOWS\system32>arp -d 192.168.56.255

C:\WINDOWS\system32>arp -a

Интерфейс: 192.168.56.1 --- 0xb
  адрес в Интернете      Физический адрес      Тип
  224.0.0.251           01-00-5e-00-00-fb     статический
  239.192.152.143       01-00-5e-40-98-8f     статический
  239.255.255.250       01-00-5e-7f-ff-fa     статический

Интерфейс: 100.64.0.122 --- 0x12
  адрес в Интернете      Физический адрес      Тип
  100.64.0.1            a8-63-7d-d9-82-0c     динамический
  100.64.0.255          ff-ff-ff-ff-ff-ff     статический
  224.0.0.2             01-00-5e-00-00-02     статический
  224.0.0.251           01-00-5e-00-00-fb     статический
  224.0.0.252           01-00-5e-00-00-fc     статический
  239.192.152.143       01-00-5e-40-98-8f     статический
  239.255.255.250       01-00-5e-7f-ff-fa     статический 
 ```

