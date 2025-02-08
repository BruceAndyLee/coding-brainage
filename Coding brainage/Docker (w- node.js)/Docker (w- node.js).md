---
Status: In progress
tags:
- devops
- system
---
[https://www.youtube.com/watch?v=9zUHg7xjIqQ&ab_channel=freeCodeCamp.org](https://www.youtube.com/watch?v=9zUHg7xjIqQ&ab_channel=freeCodeCamp.org)

[https://bitjudo.com/blog/2014/03/13/building-efficient-dockerfiles-node-dot-js/](https://bitjudo.com/blog/2014/03/13/building-efficient-dockerfiles-node-dot-js/)

# Docker

## Cоздание кастомного образа (node image)

```Docker
# с docker-hub скачиваем собранный образ для работы с нодой 15 версии
FROM node:15

# Указываем основную папку внутри образа
# в этой папке будут запускаться все файлы-точки входа 
WORKDIR /app

# копируем в основную папку файл с конфигами проекта (package.json)
COPY package.json .
# в более явной форме можно было написать
# COPY package.json /app/
# но '.' благодаря предыдущей команде означает то же самое

# RUN - build-time докер-команда, то есть, она будет выполнена
# при вызове из консоли
# 
#  $ docker build <path/to/dockerfile>
#
RUN yarn install

# копируем ВСЕ файлы нашего проекта в рабочую папку образа
COPY . ./
# Заметим, что package.json уже был скопирован ранее.
# Это сделано для того, чтобы докер закешировал его и
# при повторных сборах и пересборах контейнера, не выполнял копирование
# package.json и yarn install без необходимости

# ничего не делает, скорее, просто дает читателю докерфайла понять,
# что надо запускать контейнер с проброской порта 3000
# (чтобы трафик с порта 3000 хост-машины шел на 3000 порт внутри контейнера)
EXPOSE 3000

# CMD - run-time команда, не будет вызвана при сборке контейнера,
# только при запуске
CMD ["node", "index.js"]
# или, чтобы использовать правильно настроенный скрипт запуска
CMD ["yarn", "dev"]
```

## Базовое управление контейнерами

`docker run -p 3000:3000 -d --name node-app 4e4c9c09c9cc` - запуск контейнера с указанием порта и imageID

`-p 3000:3000` - означает что весь трафик, приходящий на хост-машину на 3000й порт, будет направлен на 3000 порт внутри контейнера (который собой являет виртуальную машину с портами, на которых могут прослушивать запросы наш изолированный проект).

`-d` - означает detatch, то есть, освободить текущую консоль от логов контейнера

`—name node-app <imageID>` - создать контейнер из образа imageID с именем node-app

![[Screenshot_2022-04-01_at_18.00.04 1.png|Screenshot_2022-04-01_at_18.00.04 1.png]]

![[Screenshot_2022-04-01_at_18.00.17 1.png|Screenshot_2022-04-01_at_18.00.17 1.png]]

вот так оно выглядит в выводе docker ps

![[Screenshot_2022-04-01_at_18.01.48 1.png|Screenshot_2022-04-01_at_18.01.48 1.png]]

---

`docker run -v host_folder_location:in_container_folder -p 3000:3000 -d --name node-app node-app` - новый флаг `-v`, чтобы сказать докеру какие папки синхронизировать.

`host_folder_location` - путь до папки с нашим проектом, который редактируется в IDE

`in_container_folder` - путь до workdir внутри контейнера

Без использования переменных окружения:

`/Users/andrey/Documents/code/node-docker/:/app`

С переменными окружения (mac/linux):

`$(pwd):/app`

==**Проблема:**== этой команды в том, что она синзронизует папку ==**$(pwd)**== на локальной машине с папкой /app в контейнере, ПЕРЕТИРАЯ контент, созданный в папке **/app** build-time командами. Это означает, что если удалить с локальной машины папку node_modules (так как она не нужна, разработка-то не локальная), то синхронизация папок (bind mount) удалит ее из контейнера тоже.

==**Решение:**== добавить еще один том (volume). Тома - это описание некоторых путей и папок внутри контейнера. Предпочтение отдается более длинным (то есть, более специфичным) путям. Вот так выглядит исправленная команда:

`docker run -v %(pwd):/app -v /app/node_modules -p 3000:3000 -d --name node-app node-app`

Теперь путь /app/node_modules исключен из списка путей, которые будут обновляться через bind mount `-v %(pwd):/app`

==**Решение 2:**== можно сложить весь изменяемый в процессе разработки контент в папку **src** на локальной машине и синхронизировать именно её, чтобы не перетирать папку /app целиком. Тогда команда

  

==**СУПЕРПРОБЛЕМА:**== bind mount работает в обе стороны. Если создать файл внутри контейнера, в привязанной папке на хосте он тоже появится.

==**РЕШЕНИЕ:**== сделать bind mount read-only:

`-v $(pwd):/app:ro`

---

`docker rm node-app -f` - удалить работающий контейнер

`docker exec -it node-app bash` - залезть в файловую систему контейнера

`docker build -t node-app .` - собрать образ контейнера и дать ему имя **node-app**

---

При обновлении package.json можно пользоваться **docker-compose** утилитой с флагом -V

```Bash
docker-compose -f fileone.yml -f filetwo.yml -d --build -V
# этот флаг заставит команду обновить анонимные тома (anonymous volumes)
```

## Переменные окружения

Объявление дефолтного значения переменной окружения:

```Docker
ENV PORT 3000
```

Указание значение переменной окружения при запуске контейнера:

```Bash
docker run <..> --env PORT=4000 --env VARNAME="VARVAL"
```

Если переменных окружения несколько десятков, создаем файл ==.env:==

```Plain
PORT=4000
VARNAME="VARVAL"
<..>
```

запуск контейнера:

```Bash
# запуск со ссылкой на локальный .env файл
docker run <..> --env-file ./.env
```

## Скрипты для запуска контейнеров

Создаем файл `docker-compose.yml`:

```YAML
version: "3" # версия docker-compose утилиты
services:
  node-app:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - ./:/app
      - /app/node_modules
    environment:
      - PORT=3000
		# env_file:
    #   - ./.env
```

запуск такого файла в консоли командой docker-compose эквивалентен выполнению команды

```Bash
docker run -v %(pwd):/app -v /app/node_modules --env PORT=3000 -p 3000:3000 -d --name node-app node-app
```

```Bash
# Запуск записанной .yml конфигурации:
docker-compose up -d

# выключение
docker-compose down -v # -v чтобы удалить том
```

### ВАЖНО

Эта команда сама собирает образ (image), который сама же создает, если его нет. По умолчанию используется конкатенация `` `${project_folder_name}_${service-name}` ``

Где service-name это имя контейнера, указанное в docker-compose.yml.

Так как docker-compose не собирает образ, если он уже есть, то при внесении изменений docker-compose сам не поймет, что образ протух. Для этого надо использовать флаг —build (для пересборки образа)

```Bash
docker-compose up -d —build
```

## Запуск в разных окружениях

docker-compose умеет принимать разные файлы для сборки контейнера

.yml с общими настройками

```YAML
# docker-compose.yml
version: "3"
services:
  node-app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - PORT=3000
```

И два файла с настройками, специфичными для среды

```YAML
# docker-compose.prod.yml
version: "3"
services:
  node-app:
    build:
      context: .
      args:
        NODE_ENV: production
    environment:
      - NODE_ENV=production
    command: yarn start
```

```YAML
# docker-compose.dev.yml
version: "3"
services:
  node-app:
    build:
      context: .
      args:
        NODE_ENV: development
    volumes:
      - ./:/app:ro
      - /app/node_modules
    environment:
      - NODE_ENV=development
    command: yarn dev
```

Разбиение build команды на context и args дает нам возможность отдельно указать путь до Dockerfile (context) и аргументы (args), которые потом будут использоваться в Dockerfile, см. ниже.

Запуска той или иной среды разработки:

```Bash
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up -d --build
```

Теперь надо настроить Dockerfile так, чтобы он звал yarn правильно и не устанавливал devDependencies, когда `NODE_ENV=production`

```Docker
# ...
ARG NODE_ENV
RUN if [ "$NODE_ENV" = "development" ]; \
  then yarn install; \
  else yarn install --production=true; \
  fi
# обратные слэши отменяют перенос строки

# ...
```

## Контейнер mongo

Пример использования готового образа контейнера для базы (без сборки своего образа с помощью dockerfile):

```YAML
services:
	node-app: ...
	mongo:
    image: mongo
		# инфа о переменных указана на странице в docker-hub
    environment:
      - MONGO_INITDB_ROOT_USERNAME=sudo
      - MONGO_INITDB_ROOT_PASSWORD=1
		# чтобы сохранить тома с данными из базы
		volumes:
      - mongo-db:/data/db

# все именованные тома нужно задекларировать в корне docker-compose.yml
volumes:
 mongo-db:
```

В такой конфигурации уже нельзя юзать docker-compose <...> down -v, потому что так удалятся все тома, в том числе и наш именованный том с данными из БД.

Вместо этого, надо просто делать docker-compose down, запускать всё по-новой и выполнять:

```Bash
docker volume prune
# чтобы удалить все локальные НЕиспользуемые контейнеры 
```

  

## Сеть контейнеров

посмотреть на настройки контейнера (в том числе и на айпи адресс контейнера):

```Bash
docker inspect container_name (e.g node-docker_node-app_1)
```

Пример вывода (секция `networks`):

```JSON
"Networks": {
  "node-docker_default": {
    "IPAMConfig": null,
    "Links": null,
    "Aliases": [
      "7050ad99fc28",
      "node-app"
    ],
    "NetworkID": "2a77714df0a00b946c2f19b040fa8ceb928946daf6b1d0ff45ef6d015ca9ded2",
    "EndpointID": "42bb48f6fa7b83df087fd0483b0b71f79d0b6307da83dc8766fa2a8a5cea7036",
    "Gateway": "172.30.0.1",
    "IPAddress": "172.30.0.3",
    "IPPrefixLen": 16,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "MacAddress": "02:42:ac:1e:00:03",
    "DriverOpts": null
  }
}
```

Скорее всего IP адресс будет разный при каждом запуске контейнера, поэтому надо получать адрес контейнера с БД каждый раз динамически

Список сетей:

```Bash
docker network ls
```

при создании кастомной сетки в выводе будет запись:

```Bash
NETWORK ID     NAME                   DRIVER    SCOPE
8add55429476   node-docker_default    bridge    local
```

В случае с построением кастомных сетей из контейнеров контейнерам доступен аналог DNS севрера, то есть, сервиса, который позволяет контейнерам общаться друг с другом используя имя контейнера для вычисления IP адреса (который, как и в реальной жизни, может меняться)

Посмотреть на сетку:

```Bash
docker network inspect node-docker_default
```

Пример вывода:

```JSON
[
  {
    "Name": "node-docker_default",
    "Id": "8add554294761e8b0fb160b028aab3e8f6d9aede690f5c5b90b496881dda9823",
    "Created": "2022-04-04T12:19:38.10287543Z",
    "Scope": "local",
    "Driver": "bridge",
    "EnableIPv6": false,
    "IPAM": {
      "Driver": "default",
      "Options": null,
      "Config": [
        {
          "Subnet": "172.31.0.0/16",
          "Gateway": "172.31.0.1"
        }
      ]
    },
    "Internal": false,
    "Attachable": true,
    "Ingress": false,
    "ConfigFrom": {
      "Network": ""
    },
    "ConfigOnly": false,
    "Containers": {
      "285e1c628dd144f55a1ca035ed64abd1aed80479bad17d9662e5e1dbbe2822da": {
        "Name": "node-docker_node-app_1",
        "EndpointID": "589f1194db0cad3dcc86e632a3b442dba4fd4ace578f1ad47c57047dfa7da08c",
        "MacAddress": "02:42:ac:1f:00:03",
        "IPv4Address": "172.31.0.3/16",
        "IPv6Address": ""
      },
      "6fe16226696cebdfb688999410150ee26110c5dbfda5876ab2913b9cd250c01a": {
        "Name": "node-docker_mongo_1",
        "EndpointID": "6fa9167f937aad6948d9c4d44291246d0d1aabab824e8057ffbb46b38b50ab75",
        "MacAddress": "02:42:ac:1f:00:02",
        "IPv4Address": "172.31.0.2/16",
        "IPv6Address": ""
      }
    },
    "Options": {},
    "Labels": {
      "com.docker.compose.network": "default",
      "com.docker.compose.project": "node-docker",
      "com.docker.compose.version": "1.29.2"
    }
  }
]
```

## Использование process.env

  

## Команды

```Bash
# удалить все контейнеры
docker rm -f $(docker ps -a -q)

# удалить все локальные тома
docker volume rm $(docker volume ls -q)

# удалить все образы
docker rmi $(docker images -a -q)
```

  

# Node roadmap

![[Untitled 4.png|Untitled 4.png]]

  

  

  

```JavaScript
cmd + d - a
cmd + c - a
```