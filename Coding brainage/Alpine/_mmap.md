---

kanban-plugin: board

---

## dump

- [ ] rc-service ~ systemctl
	
	for starting daemon-services in the background of the system
- [ ] после того, как бутлоадер загружает ОС в память, первый процесс, который запускает ядро - это /sbin/init, который как раз и есть openrc (или альтернативное решение для других дистро?)
- [ ] openrc - штука, которая запускает сервисы в OS после запуска машины
	
	сервисы разделены на слои, в рамках слоев сервисы могут блокировать друг друга, а могут запускаться параллельно.
- [ ] в папке /etc/runlevels хранятся скрипты для запуска сервисов, разложенные по папкам-слоям:
	
	```
	sysinit/
	boot/
	default/
	nonetwork/
	shutdown/
	```
- [ ] в каждой такой папке лежат симлинки на плоский список скриптов в /etc/init.d/
- [ ] каждый скрипт в init.d - это bash, в котором определены стандартные ручки, которые openrc может позвать, чтобы определить порядок загрузки модулей (скрипты - зависимости)


## drivers

- [ ] [[eudev]]


## OS startup order

- [ ] declare initcall sections (C code functions) that will be called at system boot
- [ ] initcalls - run subsystem-functions
- [ ] pid1 - run the openrc (other service manager process)


## скорее software

- [ ] [[initcalls]]
- [ ] [[subsystem]]
- [ ] [[openrc]]
- [ ] [[netlink]]
- [ ] [[runtime file-systems]]
- [ ] [[modprobe]]
- [ ] [[kobject]]
- [ ] [[uevent]]


## скорее hardware

- [ ] [[Buses & devices]]


## explainers

- [ ] [[how a driver is loaded]]


## subsystems

- [ ] [[ss overview]]
- [ ] [[block subsystem]]
- [ ] [[v4l2 subsystem]]
- [ ] [[input subsystem]]
- [ ] [[ALSA subsystem]]
- [ ] [[net subsystem]]




%% kanban:settings
```
{"kanban-plugin":"board","list-collapse":[false,false,false,false,false,false,false]}
```
%%