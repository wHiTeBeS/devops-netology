# Домашнее задание к занятию "3.3. Операционные системы, лекция 1"

1. Какой системный вызов делает команда `cd`? В прошлом ДЗ мы выяснили, что `cd` не является самостоятельной  программой, это `shell builtin`, поэтому запустить `strace` непосредственно на `cd` не получится. Тем не менее, вы можете запустить `strace` на `/bin/bash -c 'cd /tmp'`. В этом случае вы увидите полный список системных вызовов, которые делает сам `bash` при старте. Вам нужно найти тот единственный, который относится именно к `cd`.  

Системный вызов`chdir`  

2. Попробуйте использовать команду `file` на объекты разных типов на файловой системе. Например:
    ```bash
    vagrant@netology1:~$ file /dev/tty
    /dev/tty: character special (5/0)
    vagrant@netology1:~$ file /dev/sda
    /dev/sda: block special (8/0)
    vagrant@netology1:~$ file /bin/bash
    /bin/bash: ELF 64-bit LSB shared object, x86-64
    ```
    Используя `strace` выясните, где находится база данных `file` на основании которой она делает свои догадки.  
```bash
vagrant@vagrant:~$ strace -e openat file /dev/sda
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libmagic.so.1", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/liblzma.so.5", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbz2.so.1.0", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libz.so.1", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpthread.so.0", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/magic", O_RDONLY) = 3
openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3
openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 3
/dev/sda: block special (8/0)
+++ exited with 0 +++ 

vagrant@vagrant:~$ cat /etc/magic
# Magic local data for file(1) command.
# Insert here your local magic data. Format is described in magic(5).
```
База данных находится в `/usr/share/misc/magic.mgc`, что подтверждает `man file`

3. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).
```bash
vagrant@vagrant:~$ ping ya.ru >> ping.log & rm ping.log
[1] 2025
vagrant@vagrant:~$ sudo lsof +L1
COMMAND  PID    USER   FD   TYPE DEVICE SIZE/OFF NLINK    NODE NAME
ping    2025 vagrant    1w   REG  253,0      326     0 1048599 /home/vagrant/ping.log (deleted)
vagrant@vagrant:~$ sudo truncate -s0 /proc/2025/fd/1
vagrant@vagrant:~$ sudo lsof +L1
COMMAND  PID    USER   FD   TYPE DEVICE SIZE/OFF NLINK    NODE NAME
ping    2025 vagrant    1w   REG  253,0      140     0 1048599 /home/vagrant/ping.log (deleted) 
```
`ping.log` уменьшился  

4. Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?

Зомби-процессы не потребляют ресурсы так как по факту уже завершились. В списке процессов отображаются потому что их родтельский процесс не сделал системный вызов `wait()`

5. В iovisor BCC есть утилита `opensnoop`:
    ```bash
    root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
    /usr/sbin/opensnoop-bpfcc
    ```
    На какие файлы вы увидели вызовы группы `open` за первую секунду работы утилиты? Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04. Дополнительные [сведения по установке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).

```bash
vagrant@vagrant:~$ sudo apt-get install bpfcc-tools
...
vagrant@vagrant:~$ sudo /usr/sbin/opensnoop-bpfcc
PID    COMM               FD ERR PATH
859    vminfo              6   0 /var/run/utmp
629    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
629    dbus-daemon        15   0 /usr/share/dbus-1/system-services
629    dbus-daemon        -1   2 /lib/dbus-1/system-services
629    dbus-daemon        15   0 /var/lib/snapd/dbus-1/system-services/
2418   systemd-udevd      14   0 /sys/fs/cgroup/unified/system.slice/systemd-udevd.service/cgroup.procs
2418   systemd-udevd      14   0 /sys/fs/cgroup/unified/system.slice/systemd-udevd.service/cgroup.threads
```

6. Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc`, где можно узнать версию ядра и релиз ОС.
```bash
vagrant@vagrant:~$ strace uname -a
...
uname({sysname="Linux", nodename="vagrant", ...}) = 0
```
```bash
vagrant@vagrant:~$ man 2 uname
``` 
Part of the utsname information is also accessible  via  /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.

7. Чем отличается последовательность команд через `;` и через `&&` в bash? Например:
    ```bash
    root@netology1:~# test -d /tmp/some_dir; echo Hi
    Hi
    root@netology1:~# test -d /tmp/some_dir && echo Hi
    root@netology1:~#
    ```
; команды выполняются последовательно не зависимо от успеха или ошибки других команд.   
&& следующая команда выполнится только при условии успешного завершения предыдущей.  

    Есть ли смысл использовать в bash `&&`, если применить `set -e`?  
`set -e` указывает оболочке выйти, если команда дает ненулевой статус выхода. Проще говоря, оболочка завершает работу при сбое команды.
Следовательно использовать `&&` не имеет смысла
```bash
vagrant@vagrant:~$ test -d /tmp/some_dir && echo Hi
vagrant@vagrant:~$ echo $?
1
vagrant@vagrant:~$ echo hi && test -d /tmp/some_dir
hi
vagrant@vagrant:~$ echo $?
1
```

8. Из каких опций состоит режим bash `set -euxo pipefail` и почему его хорошо было бы использовать в сценариях?
 
-e - прекращает выполнение скрипта если команда завершилась ошибкой, выводит в stderr строку с ошибкой. Обойти эту проверку можно добавив в пайплайн к команде true: mycommand | true.  
-u - прекращает выполнение скрипта, если встретилась несуществующая переменная.  
-x - выводит выполняемые команды в stdout перед выполненинем.  
set -o pipefail - прекращает выполнение скрипта, даже если одна из частей пайпа завершилась ошибкой.   

Режим хорошо использовать в сценариях для отладки, улучшит информативность вывода данных об ошибках, повысит безопасность  

9. Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В `man ps` ознакомьтесь (`/PROCESS STATE CODES`) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).
 ```bash
vagrant@vagrant:~$ ps -eo stat | cut -c1 | sort | uniq -c
     49 I
      1 R
     66 S
 ```

> For BSD formats and when the stat keyword is used, additional characters may
       be displayed:

               <    high-priority (not nice to other users)
               N    low-priority (nice to other users)
               L    has pages locked into memory (for real-time and custom IO)
               s    is a session leader
               l    is multi-threaded (using CLONE_THREAD, like NPTL pthreads do)
               +    is in the foreground process group