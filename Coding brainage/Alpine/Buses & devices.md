Шины, к которым подключаются периферийные устройства.
Шина отвечает за обнаружение устройства и назначение ему уникального адреса (vendor id + uuid).
Далее контроль за устройством передается подсистеме, которая отвечает за шину. Подсистема выбирает конкретный драйвер, который будет отвечать за работу с этим устройством.

Вроде бы шины бывают двух основных типов:
- такие, на которых какой-то контроллер договаривается с девайсом о его адресе
- такие, которые впаяны наглухо в плату (GPIO в RPi), у них адреса записываются деревом в строку (device tree compatible string)

Конвенция адресов устройств:

```
alias usb:v046Dp0825d*dc*dsc*dp*ic*isc*ip*in* uvcvideo
```

Детализация
```
v046D   vendor ID       = 0x046D = Logitech (assigned by USB-IF)
p0825   product ID      = 0x0825 = specific Logitech webcam model (assigned by Logitech)
d*      device version  = any
dc*     device class    = any
ic*     interface class = any
isc*    interface sub   = any
ip*     interface proto = any
*       = wildcard
```

Вендоры устройств могут купить айдишник на форуме лол:
- **USB Implementers Forum** (USB-IF)
- Logitech `046D`
- Apple - `05AC`,
- ...

Номер класса девайса подчиняется стандарту (за стандарт отвечает тот же USB-IF):
- `0x03` = HID - human interface device
- `0x08` = Mass Storage
- `0x0E` = Video

Далее внутри диапазона адрес назначается самим вендором.