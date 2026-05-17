Это менеджер сервисов, запущенных на устройстве.
Запускается ядром операционной системы после загрузки как самый первый процесс, лежит по адресу /sbin/init:
```
# ps

1 root      0:00 /sbin/init
2 root      0:00 [kthreadd]
3 root      0:00 [pool_workqueue_]
4 root      0:00 [kworker/R-rcu_g]
5 root      0:00 [kworker/R-rcu_p]
6 root      0:00 [kworker/R-slub_]
7 root      0:00 [kworker/R-netns]
9 root      0:00 [kworker/0:0H-ev]
11 root      0:00 [kworker/u8:0-ex]
12 root      0:00 [kworker/R-mm_pe]
13 root      0:00 [rcu_tasks_kthre]
14 root      0:00 [rcu_tasks_trace]
15 root      0:00 [ksoftirqd/0]
16 root      0:00 [rcu_preempt]
17 root      0:00 [migration/0]
18 root      0:00 [cpuhp/0]
19 root      0:00 [cpuhp/1]
...
```

Скрипты разбиваются на несколько слоев в папке /etc/runlevels
```
sysinit/
boot/
default/
nonetwork/
shutdown/
```

---

Во время работы устройства можно менять скрипты на разных слоях.
```toml
# меняем runtime: останавливаем и запускаем сервисы
rc-service <name> start|stop
rc-service --list # список всех запущенных сервисов

# меняем boot-конфиг: добавляем и удаляем сервисы,
# которые openrc будет запускать самостоятельно при запуске системы
rc-update add <service> <runlevel>
rc-update del <service> <runlevel> 
```

Например, мы хотим, чтобы запускался [[eudev]]-daemon (вместо mdev, всё сложно, см. [[eudev]])

Для начала смотрим текущий конфиг `sysinit`:
```bash
alpine-rpi:~> ls -la /etc/runlevels/sysinit/
total 0
drwxr-xr-x    2 root     root           140 Jan  1  1970 .
drwxr-xr-x    7 root     root           140 Jan  1  1970 ..
lrwxrwxrwx    1 root     root            17 Jan  1  1970 devfs -> /etc/init.d/devfs
lrwxrwxrwx    1 root     root            17 Jan  1  1970 dmesg -> /etc/init.d/dmesg
lrwxrwxrwx    1 root     root            21 Jan  1  1970 hwdrivers -> /etc/init.d/hwdrivers
lrwxrwxrwx    1 root     root            16 Jan  1  1970 mdev -> /etc/init.d/mdev <-- ПО УМОЛЧАНИЮ СИСТЕМА ИСПОЛЬЗУЕТ mdev
lrwxrwxrwx    1 root     root            19 Jan  1  1970 modloop -> /etc/init.d/modloop
```

Понимаем, что нам надо удалить mdev из `/etc/runlevels/sysinit`:
```bash
alpine-rpi:~> rc-update del mdev sysinit
* service mdev deleted from runlevel sysinit
```

Убеждаемся, что из sysinit пропал этот сервис:
```bash
alpine-rpi:~> ls -la /etc/runlevels/sysinit/
total 0
drwxr-xr-x    2 root     root           120 May 12 12:02 .
drwxr-xr-x    7 root     root           140 Jan  1  1970 ..
lrwxrwxrwx    1 root     root            17 Jan  1  1970 devfs -> /etc/init.d/devfs
lrwxrwxrwx    1 root     root            17 Jan  1  1970 dmesg -> /etc/init.d/dmesg
lrwxrwxrwx    1 root     root            21 Jan  1  1970 hwdrivers -> /etc/init.d/hwdrivers
lrwxrwxrwx    1 root     root            19 Jan  1  1970 modloop -> /etc/init.d/modloop
```
!!! При этом из активного списка сервисов этот сервис не пропал и сам сервис по-прежнему работает, потому что run-time состояние никто не трогал:
```bash
alpine-rpi:~> rc-service --list | grep mdev
mdev # ВОТ ОН
alipine-rpi:~> rc-service mdev status
* status: started
```

Далее надо добавить вместо него udev:
```bash
alpine-rpi:~> rc-update add udev sysinit
* service udev added to runlevel sysinit
```

Снова перепроверяем настройки `sysinit`:
```bash
alpine-rpi:~> ls -la /etc/runlevels/sysinit
total 0
drwxr-xr-x    2 root     root           140 May 12 12:07 .
drwxr-xr-x    7 root     root           140 Jan  1  1970 ..
lrwxrwxrwx    1 root     root            17 Jan  1  1970 devfs -> /etc/init.d/devfs
lrwxrwxrwx    1 root     root            17 Jan  1  1970 dmesg -> /etc/init.d/dmesg
lrwxrwxrwx    1 root     root            21 Jan  1  1970 hwdrivers -> /etc/init.d/hwdrivers
lrwxrwxrwx    1 root     root            19 Jan  1  1970 modloop -> /etc/init.d/modloop
lrwxrwxrwx    1 root     root            16 May 12 12:07 udev -> /etc/init.d/udev # ВОТ ОН ДОБАВИЛСЯ
```

