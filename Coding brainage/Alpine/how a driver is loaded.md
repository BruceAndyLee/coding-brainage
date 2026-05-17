ОС разбита на подсистемы [[subsystem]].
Каждая подсистема предоставляет любому драйверу интерфейс, которым драйвер может пользоваться.
Система отвечает за то, какой драйвер будет отвечать за работу конкретного подключенного устройства.

Как происходит соотнесение драйвера и устройства:
- система знает о том, какие ШИНЫ в ней есть
- именно шина отвечает за обнаружение устройства и за его адрес (адрес запечен в устройство или его назначает шина?) - match & probe
- зарегистрированный девайс шиной девайс - это комбинация `vendor ID + uuid`
- далее подсистема, которая отвечает за эту шину, соотносит эту комбинацию `vendor ID + device uuid` с драйвером, доступным в системе
	- в частности, если драйвер не загружен, грузит его с помощью [[modprobe]] - ИЗ ЮЗЕР-СПЕЙСА ЛОЛ ПРИКИНЬТЕ
	- драйвер мог быть уже загружен на этапе initcall, если он вписан в ядро до компиляции
	- драйвер мог быть загружен как часть завершающего этапа перед запуском приложения
	  `modprobe uvcvideo`


## Примеры команд для исследования системы

Посмотреть таблицу перевода девайсов в драйвера:
```bash
alpine-rpi:~> ls -la /lib/modules/$(uname -r)/modules/alias | grep uvcvideo
station-9b24:/extra# cat /lib/modules/$(uname -r)/modules.alias | grep uvcvideo
alias usb:v*p*d*dc*dsc*dp*ic0Eisc01ip01in* uvcvideo
alias usb:v*p*d*dc*dsc*dp*ic0Eisc01ip00in* uvcvideo
alias usb:v8086p0B5Cd*dc*dsc*dp*ic0Eisc01ip00in* uvcvideo
...
alias usb:v19ABp1000d00*dc*dsc*dp*ic0Eisc01ip00in* uvcvideo
alias usb:v19ABp1000d01[0-1]*dc*dsc*dp*ic0Eisc01ip00in* uvcvideo
alias usb:v19ABp1000d012[0-6]dc*dsc*dp*ic0Eisc01ip00in* uvcvideo
...
```

Посмотреть, какие драйверы доступны в системе:
```bash
alpine-rpi:~> ls -la /lib/modules/$(uname -r)/kernel
drwxr-xr-x    3 root     root            28 Jun 18  2024 arch
drwxr-xr-x    4 root     root          1007 Jun 18  2024 crypto
drwxr-xr-x   49 root     root           657 Jun 18  2024 drivers
drwxr-xr-x   31 root     root           421 Jun 18  2024 fs
drwxr-xr-x    2 root     root            33 Jun 18  2024 kernel
drwxr-xr-x    6 root     root           185 Jun 18  2024 lib
drwxr-xr-x    2 root     root            51 Jun 18  2024 mm
drwxr-xr-x   36 root     root           499 Jun 18  2024 net
drwxr-xr-x    3 root     root            27 Jun 18  2024 security
drwxr-xr-x    6 root     root            64 Jun 18  2024 sound

alpine-rpi:~> ls -la /lib/modules/$(uname -r)/kernel/lib
```

Посмотреть, какие драйвера встроены в систему:
```bash
alpine-rpi:~> cat /lib/modules/$(uname -r)/modules.builtin | wc -l
411 # характерное количество

alpine-rpi:~> cat /lib/modules/$(uname -r)/modules.builtin | grep usb
...
kernel/drivers/usb/common/usb-common.ko
kernel/drivers/usb/core/usbcore.ko
...

```