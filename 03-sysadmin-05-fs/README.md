# Домашнее задание к занятию "3.5. Файловые системы"

1. Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.
`sparse file` — файл, в котором последовательности нулевых байтов заменены на информацию об этих последовательностях (список дыр).
2. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?  
Нет, т.к. права доступа хранятся в метаданных объекта, а он один. С точки зрения ФС хардлинки это один и тот же объект с одинаковым inode  

3. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```bash
    Vagrant.configure("2") do |config|
      config.vm.box = "bento/ubuntu-20.04"
      config.vm.provider :virtualbox do |vb|
        lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
        lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
        vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
        vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
      end
    end
    ```

    Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.
    ```bash
    vagrant@vagrant:~$ sudo fdisk -l
    Disk /dev/loop0: 55.53 MiB, 58212352 bytes, 113696 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    
    
    Disk /dev/loop1: 55.45 MiB, 58130432 bytes, 113536 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    
    
    Disk /dev/loop2: 70.32 MiB, 73728000 bytes, 144000 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    
    
    Disk /dev/loop4: 44.65 MiB, 46804992 bytes, 91416 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    
    
    Disk /dev/loop5: 61.92 MiB, 64901120 bytes, 126760 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    
    
    Disk /dev/loop6: 67.83 MiB, 71106560 bytes, 138880 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    
    
    Disk /dev/sda: 64 GiB, 68719476736 bytes, 134217728 sectors
    Disk model: VBOX HARDDISK
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: gpt
    Disk identifier: B4F1CD46-1589-455C-BA21-5171874A019C
    
    Device       Start       End   Sectors Size Type
    /dev/sda1     2048      4095      2048   1M BIOS boot
    /dev/sda2     4096   2101247   2097152   1G Linux filesystem
    /dev/sda3  2101248 134215679 132114432  63G Linux filesystem
    
    
    Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
    Disk model: VBOX HARDDISK
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    
    
    Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
    Disk model: VBOX HARDDISK
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    
    
    Disk /dev/mapper/ubuntu--vg-ubuntu--lv: 31.51 GiB, 33822867456 bytes, 66060288 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    ```
4. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.
    ```bash
    vagrant@vagrant:~$ sudo fdisk /dev/sdb
    
    Welcome to fdisk (util-linux 2.34).
    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.
    
    Device does not contain a recognized partition table.
    Created a new DOS disklabel with disk identifier 0x3f639118.
    
    Command (m for help): n
    Partition type
       p   primary (0 primary, 0 extended, 4 free)
       e   extended (container for logical partitions)
    Select (default p): p
    Partition number (1-4, default 1): 
    First sector (2048-5242879, default 2048):
    Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5242879, default 5242879): +2G
    
    Created a new partition 1 of type 'Linux' and of size 2 GiB.
    
    Command (m for help): n
    Partition type
       p   primary (1 primary, 0 extended, 3 free)
       e   extended (container for logical partitions)
    Select (default p): p
    Partition number (2-4, default 2):
    First sector (4196352-5242879, default 4196352):
    Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242879, default 5242879):
    
    Created a new partition 2 of type 'Linux' and of size 511 MiB.
    
    Command (m for help): w
    The partition table has been altered.
    Calling ioctl() to re-read partition table.
    Syncing disks.
   ```
   ```bash 
    vagrant@vagrant:~$ sudo fdisk /dev/sdb -l
    Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
    Disk model: VBOX HARDDISK
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x48c1e9ba
    
    Device     Boot   Start     End Sectors  Size Id Type
    /dev/sdb1          2048 4196351 4194304    2G 83 Linux
    /dev/sdb2       4196352 5242879 1046528  511M 83 Linux
   ```
   
5. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.
    ```bash
    vagrant@vagrant:~$ sudo sfdisk -d /dev/sdb > sdb.dump | sudo sfdisk /dev/sdc < sdb.dump
    Checking that no-one is using this disk right now ... OK
    
    Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
    Disk model: VBOX HARDDISK
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    
    >>> Script header accepted.
    >>> Script header accepted.
    >>> Script header accepted.
    >>> Script header accepted.
    >>> Created a new DOS disklabel with disk identifier 0x48c1e9ba.
    /dev/sdc1: Created a new partition 1 of type 'Linux' and of size 2 GiB.
    /dev/sdc2: Created a new partition 2 of type 'Linux' and of size 511 MiB.
    /dev/sdc3: Done.
    
    New situation:
    Disklabel type: dos
    Disk identifier: 0x48c1e9ba
    
    Device     Boot   Start     End Sectors  Size Id Type
    /dev/sdc1          2048 4196351 4194304    2G 83 Linux
    /dev/sdc2       4196352 5242879 1046528  511M 83 Linux
    
    The partition table has been altered.
    Calling ioctl() to re-read partition table.
    Syncing disks. 
    ```
6. Соберите `mdadm` RAID1 на паре разделов 2 Гб.
    ```bash
    root@vagrant:/home/vagrant# mdadm --create --verbose /dev/md0 -l 1 -n 2 /dev/sd{b1,c1}
    mdadm: Note: this array has metadata at the start and
        may not be suitable as a boot device.  If you plan to
        store '/boot' on this device please ensure that
        your boot-loader understands md/v1.x metadata, or use
        --metadata=0.90
    mdadm: size set to 2094080K
    Continue creating array? y
    mdadm: Defaulting to version 1.2 metadata
    mdadm: array /dev/md0 started.
   ```
7. Соберите `mdadm` RAID0 на второй паре маленьких разделов.
    ```bash
    root@vagrant:/home/vagrant# mdadm --create --verbose /dev/md1 -l 0 -n 2 /dev/sd{b2,c2}
    mdadm: chunk size defaults to 512K
    mdadm: Defaulting to version 1.2 metadata
    mdadm: array /dev/md1 started. 
    ```
    Запишем конфигурацию в файл  

    ```bash
    root@vagrant:/home/vagrant# mdadm --detail --scan --verbose > /etc/mdadm/mdadm.conf
    root@vagrant:/home/vagrant# cat /etc/mdadm/mdadm.conf
    ARRAY /dev/md0 level=raid1 num-devices=2 metadata=1.2 name=vagrant:0 UUID=c79a84b6:153b97d0:95af6259:9c17ddfc
       devices=/dev/sdb1,/dev/sdc1
    ARRAY /dev/md1 level=raid0 num-devices=2 metadata=1.2 name=vagrant:1 UUID=0ff17563:aad4cc3a:efcb130c:a4aecb0a
       devices=/dev/sdb2,/dev/sdc2
    ```
8. Создайте 2 независимых PV на получившихся md-устройствах.
    ```bash
    root@vagrant:/home/vagrant# pvcreate /dev/md0 /dev/md1
    Physical volume "/dev/md0" successfully created.
    Physical volume "/dev/md1" successfully created.
    ```
9. Создайте общую volume-group на этих двух PV.
  ```bash
  root@vagrant:/home/vagrant# vgcreate vol_group1 /dev/md0 /dev/md1
      Volume group "vol_group1" successfully created
  root@vagrant:/home/vagrant# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               ubuntu-vg
  PV Size               <63.00 GiB / not usable 0
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              16127
  Free PE               8063
  Allocated PE          8064
  PV UUID               sDUvKe-EtCc-gKuY-ZXTD-1B1d-eh9Q-XldxLf

  --- Physical volume ---
  PV Name               /dev/md0
  VG Name               vol_group1
  PV Size               <2.00 GiB / not usable 0
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              511
  Free PE               511
  Allocated PE          0
  PV UUID               GfB7ae-14rt-3pa9-Lj4t-MyQc-PYtl-LqM5jf

  --- Physical volume ---
  PV Name               /dev/md1
  VG Name               vol_group1
  PV Size               1018.00 MiB / not usable 2.00 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              254
  Free PE               254
  Allocated PE          0
  PV UUID               trBg8A-Ldqd-y3Rw-P0vF-fcUq-JxfL-u0GP8P
  ```
13. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.
   ```bash
root@vagrant:/home/vagrant# lvcreate -L 100M -n logical_vol1 vol_group1 /dev/md1
  Logical volume "logical_vol1" created.
root@vagrant:/home/vagrant# lvdisplay
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/ubuntu-lv
  LV Name                ubuntu-lv
  VG Name                ubuntu-vg
  LV UUID                ftN15m-3lML-YH5x-R5P2-kLCd-kzW3-32dlqO
  LV Write Access        read/write
  LV Creation host, time ubuntu-server, 2021-12-19 19:37:44 +0000
  LV Status              available
  # open                 1
  LV Size                31.50 GiB
  Current LE             8064
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0

  --- Logical volume ---
  LV Path                /dev/vol_group1/logical_vol1
  LV Name                logical_vol1
  VG Name                vol_group1
  LV UUID                GOB6To-tO13-xcyc-ODu6-EPK5-g2XX-FBsdOE
  LV Write Access        read/write
  LV Creation host, time vagrant, 2022-04-07 19:50:47 +0000
  LV Status              available
  # open                 0
  LV Size                100.00 MiB
  Current LE             25
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     4096
  Block device           253:1
  ```
14. Создайте `mkfs.ext4` ФС на получившемся LV.
```bash
root@vagrant:/home/vagrant# mkfs.ext4 /dev/vol_group1/logical_vol1
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done  
```
15. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.
```bash
root@vagrant:/home/vagrant# mount /dev/vol_group1/logical_vol1 /tmp/lv1/ 
```
16. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.
```bash
    root@vagrant:/home/vagrant# wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/lv1/test.gz
    --2022-04-07 19:57:17--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
    Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
    Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 22340714 (21M) [application/octet-stream]
    Saving to: ‘/tmp/lv1/test.gz’
    
    /tmp/lv1/test.gz          100%[=====================================>]  21.31M  3.13MB/s    in 8.0s
    
    2022-04-07 19:57:25 (2.66 MB/s) - ‘/tmp/lv1/test.gz’ saved [22340714/22340714] 
```
17. Прикрепите вывод `lsblk`.
```bash
root@vagrant:/home/vagrant# lsblk
NAME                          MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                           7:0    0 55.5M  1 loop  /snap/core18/2344
loop1                           7:1    0 55.4M  1 loop  /snap/core18/2128
loop2                           7:2    0 70.3M  1 loop  /snap/lxd/21029
loop4                           7:4    0 44.7M  1 loop  /snap/snapd/15314
loop5                           7:5    0 61.9M  1 loop  /snap/core20/1405
loop6                           7:6    0 67.8M  1 loop  /snap/lxd/22753
sda                             8:0    0   64G  0 disk
├─sda1                          8:1    0    1M  0 part
├─sda2                          8:2    0    1G  0 part  /boot
└─sda3                          8:3    0   63G  0 part
  └─ubuntu--vg-ubuntu--lv     253:0    0 31.5G  0 lvm   /
sdb                             8:16   0  2.5G  0 disk
├─sdb1                          8:17   0    2G  0 part
│ └─md0                         9:0    0    2G  0 raid1
└─sdb2                          8:18   0  511M  0 part
  └─md1                         9:1    0 1018M  0 raid0
    └─vol_group1-logical_vol1 253:1    0  100M  0 lvm   /tmp/lv1
sdc                             8:32   0  2.5G  0 disk
├─sdc1                          8:33   0    2G  0 part
│ └─md0                         9:0    0    2G  0 raid1
└─sdc2                          8:34   0  511M  0 part
  └─md1                         9:1    0 1018M  0 raid0
    └─vol_group1-logical_vol1 253:1    0  100M  0 lvm   /tmp/lv1 
```
18. Протестируйте целостность файла:

     ```bash
     root@vagrant:~# gzip -t /tmp/new/test.gz
     root@vagrant:~# echo $?
     0
     ```
Тестируем
```bash
root@vagrant:/home/vagrant#  gzip -t /tmp/lv1/test.gz
root@vagrant:/home/vagrant# echo $?
0
```
19. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.
```bash
root@vagrant:/home/vagrant# pvmove -n /dev/vol_group1/logical_vol1 /dev/md1 /dev/md0
  /dev/md1: Moved: 16.00%
  /dev/md1: Moved: 100.00% 
```
20. Сделайте `--fail` на устройство в вашем RAID1 md.
```bash
root@vagrant:/home/vagrant# mdadm /dev/md0 -f /dev/sdc1
mdadm: set /dev/sdc1 faulty in /dev/md0
```
21. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.
```bash
root@vagrant:/home/vagrant# dmesg | grep raid1
[ 3094.433716] md/raid1:md0: not clean -- starting background reconstruction
[ 3094.433718] md/raid1:md0: active with 2 out of 2 mirrors
[ 4439.948394] md/raid1:md0: Disk failure on sdc1, disabling device.
               md/raid1:md0: Operation continuing on 1 devices.
```
22. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

     ```bash
     root@vagrant:~# gzip -t /tmp/new/test.gz
     root@vagrant:~# echo $?
     0
     ```
Проверяем
```bash
root@vagrant:/home/vagrant# gzip -t /tmp/lv1/test.gz
root@vagrant:/home/vagrant# echo $?
0
```
23. Погасите тестовый хост, `vagrant destroy`.
```commandline
D:\vagrant>vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
```
 
 