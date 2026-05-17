У подсистемы может быть шина.
У подсистемы может не быть шины.

Разные шины могут работать с разными подсистемами:
```
шина - USB
  ├── usb-storage.ko  → block subsystem   (/dev/sda)
  ├── uvcvideo.ko     → v4l2 subsystem    (/dev/video0)
  ├── usbhid.ko       → input subsystem   (/dev/input/event0)
  ├── snd-usb-audio.ko→ ALSA subsystem    (/dev/snd/*)
  └── cdc_ether.ko    → net subsystem     (eth1, no /dev node)
```


Подсистемы, которые работают без шин, одним конечным устройством:
```
Subsystem       What it abstracts               
──────────────────────────────────────────────────────────────────
regulator       voltage/current regulators       power rails are
                                                 described in device tree
                                                 
clk (CCF)       clock tree and sources           clocks are SoC-
                Common Clock Framework           internal, described in DT
                
pinctrl         pin mux and GPIO config          pin registers live
                                                 in SoC address space

thermal         thermal zones + cooling          sensors come from
                                                 I2C/SPI/platform but all
                                                 register here

RTC             real-time clocks → /dev/rtcN     no — RTCs come from I2C,
                                                 SPI, platform, or on-chip,
                                                 all look identical here
LED             LED control                      
PWM             pulse-width modulation           ???
watchdog        hardware watchdog timers         ???
firmware        firmware loading/interface       manages UEFI, ACPI
                                                 runtime services
```

Подсистемы, у которых вообще нет устройств, только память и код:
```
crypto          crypto algorithms and            no — algorithms are just
                hardware accelerators            code or MMIO, not a bus
```