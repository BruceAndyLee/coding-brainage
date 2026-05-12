Добавить локальный или глобальный репозиторий в настройки apk, чтобы установить зависимость
```bash
vim /etc/apk/repositories
```

В настройках apk могут быть включены только локальные репозитории:
```
/media/boot/apks
```

Добавляем url community-репозитория
```toml
/media/boot/apks
https://dl-cdn.alpinelinux.org/alpine/v3.20/main
https://dl-cdn.alpinelinux.org/alpine/v3.20/community
```

После чего обновляем конфиг apk:
```
~# apk update

fetch https://dl-cdn.alpinelinux.org/alpine/v3.20/main/aarch64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.20/community/aarch64/APKINDEX.tar.gz
v3.20.10-36-g436ca7554c9 [https://dl-cdn.alpinelinux.org/alpine/v3.20/main]
v3.20.10-36-g436ca7554c9 [https://dl-cdn.alpinelinux.org/alpine/v3.20/community]
OK: 24087 distinct packages available
```

Теперь можно будет установить любой глобально-доступный пакет
```
apk add vim
```
