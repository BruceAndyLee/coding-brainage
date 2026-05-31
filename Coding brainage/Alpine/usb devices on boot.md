![[Drawing 2026-05-22 20.44.51.excalidraw]]
```
kernel enumerates USB device
    → creates /sys/bus/usb/devices/1-1.2
    → fires uevent (MODALIAS=usb:v...)
        → udev receives it
        → udev rule matches MODALIAS, calls modprobe usb-storage
            → usb-storage.ko loads
            → driver binds to device
            → kernel creates /sys/block/sda
            → kernel fires NEW uevent (SUBSYSTEM=block)
                → udev receives it
                → udev populates /dev/sda
                → udev writes /run/udev/data/b8:0
                    → pyudev can now read ID_FS_UUID etc.
```