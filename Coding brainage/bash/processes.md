
Запустить процесс и отвалиться от терминала, оставив скрипт работать
```bash
ssh root@192.168.1.12 "nohup ./some-script.sh"
```

Направить вывод (stdout) программы в файл
```bash
./some-script.sh > some-output-file.log
```

Направить поток ошибок процесса в его основной вывод (stderr -> stdout):
```bash
./some-script.sh > lol 2>&1
```

Запустить скрипт в бекграунде на удаленной машине
```bash
ssh <credentials> root@192.168.1.12 "cd /scripts/ && ./script.sh 2>&1 &"
```