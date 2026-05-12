Драйвер-демон (всё сложнее, см ниже) - отслеживает факт подключения usb-устройств к машине.

```
# from the AI

Package name: eudev — chosen by the Alpine package maintainers. eudev is a fork of udev originally created by Gentoo developers to decouple the device manager from systemd. Alpine named their package eudev to make it explicit that this is the standalone fork, not systemd's udev.

Library name: libudev.so — kept by the eudev upstream authors deliberately. Since eudev's entire purpose is to be a drop-in replacement for udev, it ships the same library name, same symbols, same API. Anything that links against libudev.so — including pyudev — works without knowing or caring whether the underlying implementation is systemd's udev or eudev.

Service name: udev — also kept by eudev upstream for the same compatibility reason. The OpenRC init script ships as /etc/init.d/udev, not /etc/init.d/eudev. Software, documentation, and muscle memory that refers to "the udev service" all just work.

So the dependency picture
apk package:   eudev          ← Alpine's name for the fork
daemon binary: udevd           ← same name as systemd's udev
service:       udev            ← OpenRC service, same name as systemd's udev
library:       libudev.so      ← same name, same API, pyudev links against this
The confusing eudevd name you tried first doesn't exist anywhere — it was my mistake inferring a name that follows the pattern of the package name but doesn't actually exist in the codebase. The upstream authors never used it.

For your plan
The system dependency to document is simply eudev (the apk package). Everything else — the library name, the service name — is transparent once the package is installed.
```


Сервис:
Драйвер:
Библиотека:

---

Проверяем, что оно установлено
```bash
apk info -e eudev        # is the daemon package actually installed?
apk info -e eudev-libs   # this is probably what you have
```

Установка и добавление сервиса в [[openrc]]
```bash
apk add eudev
setup-devd udev          # configures openrc to use eudevd instead of mdev
rc-service eudevd start
```
