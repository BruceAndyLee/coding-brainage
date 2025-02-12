---
Status: Completed
tags:
- chore
---
## SemVer

[https://semver.org/lang/ru/](https://semver.org/lang/ru/)

### Версионирование

Версия пакета формируется из трех чисел: **Major:minor:patch**

- **major** — внесение несовместимых с предыдущей версией изменений
- **minor** — новая функциональность, совместимая с предыдущей версией
- **patch** — исправление ошибок, незначительные улучшения

**Диапазоны версий:**

- `*` — любая версия (аналогично пустой строке)
- `<1.0.0` — любая версия, которая меньше ==**1.0.0**==
- `<=1.0.0` — любая версия, которая меньше или равна ==**1.0.0**==
- `>1.0.0` — любая версия, которая больше ==**1.0.0**==
- `>=1.0.0` — любая версия, которая больше или равна ==**1.0.0**==
- `=1.0.0` — только версия ==**1.0.0**== (оператор `"="` можно опустить)
- `>=1.0.0 <2.0.0` — больше или равно ==**1.0.0**== и меньше ==**2.0.0**==
- `1.0.0-2.0.0` — набор версий включительно
- `^1.0.0` — минорные и патчевые релизы первой мажорной версии (==**>=1.0.0 <2.0.0**==)
- `~1.0.0` — только патчевые релизы минорной версии (==**>=1.0.0 <1.1.0**==)

## Управление зависимостями

Формально, пакетные менеджеры придерживаются принципа инверсии зависимостей. Это позволяет реализациям-проектам зависеть от минималистичных абстракций-зависимостей. Примерно так же это происходит с классами, которые друг у друга наследуют, или реализуют те или иные интерфейсы в ООП.

**Транзитивные зависимости**: зависимости зависимостей, могут пересекаться и пложить проблемы разрешения версий.

![[Untitled 3.png|Untitled 3.png]]

  

## Yarn

- Проверка установки, версия **yarn**
    
    ```Bash
    # ---------------------------------------------
    # проверка установки, версия
    yarn --version | -v
    
    # обновление версии
    yarn set version latest
    
    # справка / справка о команде
    yarn help
    yarn help [command-name]
    # ---------------------------------------------
    ```
    
- Инициализация проекта
    
    ```Bash
    # ---------------------------------------------
    yarn init
    
    # auto инициализация
    yarn init --yes | -y
    
    # поле "private": true в package.json
    yarn init --private | -p
    
    # auto + private
    yarn init -yp
    # ---------------------------------------------
    ```
    
- Установка и добавление зависимостей
    
    ```Bash
    # ---------------------------------------------
    # установка зависимостей по сформированному package.json
    yarn
    # или
    yarn install
    
    # тихая установка
    yarn install --silent | -s
    
    # проверка установки
    yarn --check-files
    
    # принудительная переустановка зависимостей
    yarn install --force
    
    # установка только продакшн пакетов (???)
    yarn install --production | --prod
    # ---------------------------------------------
    
    
    # ---------------------------------------------
    # добавление зависимости в проект
    yarn add [package-name]
    yarn add [package-name@version]
    
    # пример
    yarn add express
    
    # тихая установка
    yarn add --silent [package-name@version]
    # или
    yarn add -s [package-name@version]
    
    # добавление зависимости для разработки
    # (не будет установлена в production режиме)
    yarn add --dev | -D [package-name]
    
    # пример
    yarn add -D nodemon
    
    # обновление зависимости
    yarn upgrade [package-name]
    
    # удаление зависимости
    yarn remove [package-name]
    # ---------------------------------------------
    ```
    
- `$ yarn install`
    - "resolution step":
        - First we load the entries stored within the lockfile, then based on those data and the current state of the project (that it figures out by reading the manifest files, aka `package.json`) the core runs an internal algorithm to find out which entries are missing.
        - For each of those missing entries, it queries the plugins using the [`Resolver`](https://github.com/yarnpkg/berry/blob/master/packages/yarnpkg-core/sources/Resolver.ts) interface, and asks them whether they would know about a package that would match the given descriptor ([`supportsDescriptor`](https://github.com/yarnpkg/berry/blob/master/packages/yarnpkg-core/sources/Resolver.ts#L54)) and its exact identity ([`getCandidates`](https://github.com/yarnpkg/berry/blob/master/packages/yarnpkg-core/sources/Resolver.ts#L114)) and transitive dependency list ([`resolve`](https://github.com/yarnpkg/berry/blob/master/packages/yarnpkg-core/sources/Resolver.ts#L123)).
        - Once it has obtained a new list of package metadata, the core starts a new resolution pass on the transitive dependencies of the newly added packages. This will be repeated until it figures out that all packages from the dependency tree now have their metadata stored within the lockfile.
        - Finally, once every package range from the dependency tree has been resolved into metadata, the core builds the tree in memory one last time in order to generate what we call "virtual packages". In short, those virtual packages are split instances of the same base package - we use them to disambiguate all packages that list peer dependencies, whose dependency set would change depending on their location in the dependency tree (consult [this lexicon entry](https://yarnpkg.com/advanced/lexicon#virtualpackages) for more information).
    - "fetch step”:
        - Now that we have the exact set of packages that make up our dependency tree, we iterate over it and for each of them we start a new request to the cache to know whether the package is anywhere to be found. If it isn't we do just like we did in the previous step and we ask our plugins (through the [`Fetcher`](https://github.com/yarnpkg/berry/blob/master/packages/yarnpkg-core/sources/Fetcher.ts) interface) whether they know about the package ([`supports`](https://github.com/yarnpkg/berry/blob/master/packages/yarnpkg-core/sources/Fetcher.ts#L43)) and if so to retrieve it from whatever its remote location is ([`fetch`](https://github.com/yarnpkg/berry/blob/master/packages/yarnpkg-core/sources/Fetcher.ts#L67)).
        - Interesting tidbit regarding the fetchers: they communicate with the core through an abstraction layer over `fs`. We do this so that our packages can come from many different sources - it can be from a zip archive for packages downloaded from a registry, or from an actual directory on the disk for [`portal:`](https://yarnpkg.com/advanced/architecture) dependencies.
    - “link step”:
        - In order to work properly, the packages you use must be installed on the disk in some way. For example, in the case of a native Node application, your packages would have to be installed into a set of `node_modules` directories so that they could be located by the interpreter. That's what the linker is about. Through the [`Linker`](https://github.com/yarnpkg/berry/blob/master/packages/yarnpkg-core/sources/Linker.ts) and [`Installer`](https://github.com/yarnpkg/berry/blob/master/packages/yarnpkg-core/sources/Installer.ts) interfaces the Yarn core will communicate with the registered plugins to let them know about the packages listed in the dependency tree, and describe their relationships (for example it would tell them that `tapable` is a dependency of `webpack`). The plugins can then decide what to do of this information in whatever way they see fit.
        - Doing this means that new linkers can be created for other programming languages pretty easily - you just need to write your own logic regarding what should happen from the packages provided by Yarn. Want to generate an `__autoload.php`? Do it! Want to setup a Python virtual env? No problemo!
        - Something else that's pretty cool is that the packages from within the dependency tree don't have to all be of the same type. Our plugin design allows instantiating multiple linkers simultaneously. Even better - the packages can depend on one another across linkers! You could have a JavaScript package depending on a Python package (which is technically the case of `node-gyp`, for example).
- Чтение информации о модулях
    
    ```Bash
    # ---------------------------------------------
    # Список установленных зависимостей:
    yarn list # построит дерево зависимостей проекта
    yarn global list
    
    # покажет только верхний уровень дерева
    yarn list --depth=0 | --depth 0
    # ---------------------------------------------
    
    
    # ---------------------------------------------
    # получить информацию о пакете
    yarn info [package-name]
    # или
    yarn why [package-name]
    
    # примеры
    yarn info react
    yarn info react description
    yarn why webpack
    # ---------------------------------------------
    ```
    

## Поля package.json

читать доку на сайте ==**yarn**==-а: [https://yarnpkg.com/configuration/manifest](https://yarnpkg.com/configuration/manifest)

- `“files”`(default - [”*”], то есть, все файлы в папке нашего модуля). Синтаксис аналогичен синтаксису **.gitignore**. Работает обратным образом: включает упомянутые файлы в архивированную версию пакета для загрузки в npm хранилище.
    
    Вне зависимости от настроек следующие файлы всегда включаются:
    
    - `package.json`
    - `README`
    - `LICENSE` / `LICENCE`
    - The file in the "main" field
    
    Вне зависимости от настроек следующие файлы всегда исключаются:
    
    - `.git`
    - `CVS`
    - `.svn`
    - `.hg`
    - `.lock-wscript`
    - `.wafpickle-N`
    - `.*.swp`
    - `.DS_Store`
    - `._*`
    - `npm-debug.log`
    - `.npmrc`
    - `node_modules`
    - `config.gypi`
    - `.orig`
    - `package-lock.json` (use [`npm-shrinkwrap.json`](https://docs.npmjs.com/cli/v8/configuring-npm/npm-shrinkwrap-json) if you wish it to be published)

---

- `"main”` (default - index.js)
    
    Указывает файл, который будет импортирован в проект пользователя командами ==**require**== или ==**import**==.
    
    Команда `require(”my-package-name”)` ищет в папке ==**node_modules**== папку с названием ==**my-package-name**== и возвращает экспорты в _main-_файле этого модуля.
    

---

- `“browser”` - можно использовать вместо `“main”` для модулей, написанных для использования в браузерной среде. Семантически намекает на то, что модуль может использовать браузерные примитивы типа ==**window**== объекта

---

- `“bin”` - используется в том случае, если устанавливаемый пакет должен быть доступен глобально на машине пользователя (при установке с флагом —==**global**==). 
    
    ```JSON
    {
      "bin": {
        "myapp": "./cli.js"
      }
    }
    ```
    
    В этом примере, после установки на машине пользователя, npm добавит в PATH путь до исполняемого файла `cli.js`. Этот файл теперь будет доступен в консоли по имени команды `myapp`.
    
    Сам модуль будет установлен в `**/usr/local/bin/myapp**` + путь, который указан в `package.json.bin`
    

---

- `“man”` - указать массив с относительным путём до файлов для системной команды ==**man**==
    
    ```JSON
    {
      "name": "foo",
      "version": "1.2.3",
      "description": "A packaged foo fooer for fooing foos",
      "main": "foo.js",
      "man": [
        "./man/foo.1",
        "./man/bar.1"
      ]
    }
    ```
    
    Цифра нужна, чтобы указать секцию мануала, в которую будет установлен наш файл с документацией
    

---

- `“config”` - объект с настройками модуля и переменных окружения, которые могут оставаться неизменными для разных версий модуля.
    
    Просто пример:
    
    ```JSON
    {
      "name": "foo",
      "config": {
        "port": "8080"
      }
    }
    ```
    
    При такой конфигурации скриптам доступна переменная окружения `npm_package_config_port`
    

---

- `“engines”` - объект для указания допустимых версий node и/или npm
    
    ```JSON
    {
      "engines": {
        "node": ">=0.10.3 <15",
    		"npm": "~1.0.20"
      }
    }
    ```
    
    Указание версии npm может иметь смысл в том случае, если по какой-то причине не все версии npm могут правильно установить наш модуль со всеми его зависимостями.
    

---

- `“os”` - массив для указания допустимых и блокирования недопустимых операционных систем:
    
    ```JSON
    {
      "os": [
        "darwin",
        "linux",
    		"!win32"
      ]
    }
    ```