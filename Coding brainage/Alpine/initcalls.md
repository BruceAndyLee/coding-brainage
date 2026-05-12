kernel binary вызывает подряд несколько функций, которые были скомпилированы в сорсе ядра на верхнем уровне(?)

```
kernel
	-> .initcall sections
			core_initcall
			subsys_initcall
			device_initcall
			late_initcall
```


## `core_initcall`

## `subsys_initcall`

## `device_initcall`
Здесь запускается регистрация всех драйверов, которые встроены в ядро (не вынесены в отдельный модуль ядра в .ko-файл)
