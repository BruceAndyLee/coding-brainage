
Пример: распарсить логи и превратить каждую строчку нужного вида в объект, с которым можно работать.
Для этого нужно использовать группы в регулярных выражениях.

Матчим логи вот такого вида
```
[12670.115574] usb 1-1.3.3: Forcing device quirks to 0x80 by module parameter for testing purpose.
[12670.115580] usb 1-1.3.3: Please report required quirks to the linux-media mailing list.
[13707.310872] usb 1-1.3.1: USB disconnect, device number 22
```

Нас интересует временная метка, адрес usb-устройства и сообщение

Определяем regex
```python
dmesg_usb_regex = r'\[(\d+\.\d+)\] usb (\d-\d(\.\d){1,2}): (.*)'
# notice how there are nested groups lol
```

Используем питоновский модуль re для поиска совпадений внутри строки
```python
import re

log_line = "[12670.115574] usb 1-1.3.3: Forcing device quirks to 0x80 by module parameter for testing purpose."

result = re.search(log_line, dmesg_usb_regex)
if result:
	(timestamp, usb_full_addr, usb_stick_suffix, message) = result.groups()
```