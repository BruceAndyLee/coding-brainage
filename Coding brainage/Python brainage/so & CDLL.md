Питон - это фреймворк для C.
То есть, на С пишется специализированный прикладной код, который решает конкретные задачи.
На питоне пишется обвязка, которая решает тривиальные инфраструктурные задачи:
- валидация данных
- чтение yaml/json/txt-файлов с конфигами
- организация доступа к системе через внешние API
- вызов того самого специализированного кода на С там, где важно быстродействие

Как добиться такой интеграции:
- код на C должен быть поделен на main.c - который нужен для запуска, и на `module-name.c|h` - это код, который решает задачу
- в папке с модулем, который будет dll-кой, нужно будет создать header-файл с декларациями функций, которые используют `external`
- вынесенный в отдельный модуль прикладной код компилируется с флагом position-independent-code (-fpic) и превращается в shared-object - линкабельную динамическую библиотеку
- в питон импортируется модуль `ctypes` и с помощью класса `CDLL` создается объект-обертка над сишным кодом, внутри которого лежат все функции в виде методов
- код в питоне уточняет типизацию аргументов и возвращаемого значения каждого метода, который нужен на уровне питона
- код в питоне использует helper-функции из модуля `ctypes` для правильного конструирования аргументов, которые передаются в C-функции

Справка с демонстрацией всего упомянутого выше:

header.h
```C
#ifndef stats_h__
#define stats_h__

extern void load_program_from_file(char filename[]);
extern void print_stats();

#endif stats_h__
```

компиляция
```bash
gcc -c -fpic modules/stats/stats.c
# stats.o - is produced
```

конвертация в shared-object (.so)
```bash
gcc -shared -o stats.so stats.o
# stats.so - is produced
```

Импортирование `ctypes` и подгрузка динамической библиотеки
```python
import ctypes as ct

# load & setup typing
stats_dll = ct.CDLL("stats.so")
stats_dll.load_program_from_file.argtypes = [ct.c_char_p]  # aka char* / char str[]
stats_dll.load_program_from_file.restype = ct.c_voidp

# invoke
stats_dll.load_program_from_file(b"program-file-to-get-stats-for.txt")
stats_dll.print_stats()
```