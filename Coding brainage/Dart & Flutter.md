---
Status: In progress
tags:
- native
- frontend
---
DART PAD [https://dartpad.dev/](https://dartpad.dev/)?

firebase core [https://pub.dev/packages/firebase_core](https://pub.dev/packages/firebase_core)

flutter intro [https://www.youtube.com/watch?v=1xipg02Wu8s](https://www.youtube.com/watch?v=1xipg02Wu8s)

crypto-thingie [https://pub.flutter-io.cn/documentation/cryptography/latest/cryptography/Ed25519-class.html](https://pub.flutter-io.cn/documentation/cryptography/latest/cryptography/Ed25519-class.html)

### Flutter cli commands

```Bash
# creating an application with specified org
flutter create —org com.orgdomain appname

# adding a dependency (published at) pub.dev
# to your pubspec.yaml
flutter pub add dependencyname
```

### installing firebase to flutter

```Bash
flutter pub add firebase_core
flutter pub add firebase_auth
flutter pub add cloud_firestore
flutter pub add firebase_analytics
```

as a result in pubspec.yaml

```YAML
dependencies:
  flutter:
    sdk: flutter


  # The following adds the Cupertino Icons font to your application.
  # Use with the CupertinoIcons class for iOS style icons.
  cupertino_icons: ^1.0.2
  firebase_core: ^2.6.1
  firebase_auth: ^4.2.8
  cloud_firestore: ^4.4.2
  firebase_analytics: ^10.1.3
```

### qr code tinkering

[https://medium.com/flutter-community/building-flutter-qr-code-generator-scanner-and-sharing-app-703e73b228d3](https://medium.com/flutter-community/building-flutter-qr-code-generator-scanner-and-sharing-app-703e73b228d3)

# BUGS (I hate that this is a thing)

### принимаем лицензии android

Пытаемся принять лицензии от андроид-студио

```Bash
flutter doctor --android-licenses
```

выскакивает ошибка

```Bash
Exception in thread "main" java.lang.UnsupportedClassVersionError: com/android/prefs/AndroidLocationsProvider
has been compiled by a more recent version of the Java Runtime (class file version 55.0),
this version of the Java Runtime only recognizes class file versions up to 52.0
```

пытаемся поменять версию джавы с помощью sdkman. Но команда sdk не работает:

```Bash
limmy@Andrews-MacBook-Pro privatenotes % sdk
zsh: command not found: sdk
```

обновляем переменные окружения

```Bash
export SDKMAN_DIR="$HOME/.sdkman"
[[ -s "$HOME/.sdkman/bin/sdkman-init.sh" ]] && source "$HOME/.sdkman/bin/sdkman-init.sh"
```

переключаемся на нужную нам версию java:

```Bash
sdk use java 11
```

обнаруживаем, что она не установлена

```Bash
Stop! Candidate version is not installed.
Tip: Run the following to install this version
$ sdk install java 11
```

устанавлиаем нужную версию java с помощью sdkman:

```Bash
sdk install java 11
```

обнаруживаем, что так версии не пишутся!

```Bash
Stop! java 11 is not available. Possible causes:
 * 11 is an invalid version
 * java binaries are incompatible with your platform
 * java has not been released yet

Tip: see all available versions for your platform:

  $ sdk list java
```

смотрим на доступные версии:

```Bash
sdk list java
================================================================================
Available Java Versions for macOS 64bit
================================================================================
 Vendor        | Use | Version      | Dist    | Status     | Identifier
--------------------------------------------------------------------------------
 Corretto      |     | 20           | amzn    |            | 20-amzn             
               |     | 19.0.2       | amzn    |            | 19.0.2-amzn         
               |     | 19.0.1       | amzn    |            | 19.0.1-amzn         
               |     | 17.0.7       | amzn    |            | 17.0.7-amzn         
               |     | 17.0.6       | amzn    |            | 17.0.6-amzn         
               |     | 17.0.5       | amzn    |            | 17.0.5-amzn         
               |     | 11.0.19      | amzn    |            | 11.0.19-amzn        
               |     | 11.0.18      | amzn    |            | 11.0.18-amzn        
               |     | 11.0.17      | amzn    |            | 11.0.17-amzn        
               | >>> | 11.0.14.10.1 | amzn    | local only | 11.0.14.10.1-amzn   
               |     | 8.0.372      | amzn    |            | 8.0.372-amzn        
               |     | 8.0.362      | amzn    |            | 8.0.362-amzn        
               |     | 8.0.352      | amzn    |            | 8.0.352-amzn        
 Gluon         |     | 22.1.0.1.r17 | gln     |            | 22.1.0.1.r17-gln    
               |     | 22.1.0.1.r11 | gln     |            | 22.1.0.1.r11-gln    
               |     | 22.0.0.3.r17 | gln     |            | 22.0.0.3.r17-gln    
               |     | 22.0.0.3.r11 | gln     |            | 22.0.0.3.r11-gln    
 GraalVM       |     | 22.3.r19     | grl     |            | 22.3.r19-grl        
               |     | 22.3.r17     | grl     |            | 22.3.r17-grl        
               |     | 22.3.r11     | grl     |            | 22.3.r11-grl        
               |     | 22.3.1.r19   | grl     |            | 22.3.1.r19-grl      
               |     | 22.3.1.r17   | grl     |            | 22.3.1.r17-grl      
               |     | 22.3.1.r11   | grl     |            | 22.3.1.r11-grl      
               |     | 22.2.r17     | grl     |            | 22.2.r17-grl        
               |     | 22.2.r11     | grl     |            | 22.2.r11-grl        
               |     | 22.1.0.r17   | grl     |            | 22.1.0.r17-grl
```

выбираем какую-нибудь мажорную 11-ю версию:

```Bash
sdk install java 11.0.17-amzn # используем именно Identifier из полседнего столбца
```

после того, как она скачалась, снова принимаем лицензии:

```Bash
flutter doctor --android-licenses
[=======================================] 100% Computing updates...             
All SDK package licenses accepted.
```

ебоба, с победой!

### проблема с bundled java version

проверяем готовность flutter к работе:

```Bash
flutter doctor
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.7.3, on macOS 13.2.1 22D68 darwin-x64, locale en-RU)
[✓] Android toolchain - develop for Android devices (Android SDK version 33.0.1)
[✓] Xcode - develop for iOS and macOS (Xcode 14.2)
[✓] Chrome - develop for the web
[!] Android Studio (version 2022.1)
    ✗ Unable to find bundled Java version.
[✓] VS Code (version 1.71.0)
[✓] Connected device (2 available)
[✓] HTTP Host Availability
```

даем терминалу доступ full disk access и выполняем команду:

```Bash
# For those having these issues with the latest Android Studio - Electric Eel version,
# and other canaries and preview releases,
# note that the bundled jre directory in the Android Studio installation folder is now renamed to jbr

To resolve this, just create a sym link jre -> jbr and Flutter won't complain.
cd /Applications/Android\ Studio.app/Contents
ln -s jbr jre
```

теперь флаттер должен быть готов к работе:

```Bash
flutter doctor
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.7.3, on macOS 13.2.1 22D68 darwin-x64, locale en-RU)
[✓] Android toolchain - develop for Android devices (Android SDK version 33.0.1)
[✓] Xcode - develop for iOS and macOS (Xcode 14.2)
[✓] Chrome - develop for the web
[✓] Android Studio (version 2022.1)
[✓] VS Code (version 1.71.0)
[✓] Connected device (2 available)
[✓] HTTP Host Availability
```

### настройка android visual studio

[https://www.youtube.com/watch?v=AUjlX24yC2Q](https://www.youtube.com/watch?v=AUjlX24yC2Q)

- создать эмулятор. Установить операционную систему, выбрать версию андроида, посмотреть на настройки
- в settings найти пункт plugins. Установить плагины flutter и dart. Перезапустить adnroid-studio
- дальше можно запускать проект на установленном эмуляторе даже без необходимости запускать android-studio!

### firebase sdkVersion mismatch

Плагин для подключения firebase ругается на версию используемого во flutter-проекте sdk: говорит, что ему нужна минимум 19-я, а у меня 16-я.

Советует в проекте перенастроить переменную `minSdkVersion` на 19, но предупреждает, что это приведет к сужению версий android, на которых будет доступно мое конечное приложение.

**РЕШЕНИЕ:** в файле `<rootDir>/android/app/build.gradle`

```Bash
defaultConfig {
        // TODO: Specify your own unique Application ID (https://developer.android.com/studio/build/application-id.html).
        applicationId "com.learningflutter.privatenotes"
        // You can update the following values to match your application needs.
        // For more information, see: https://docs.flutter.dev/deployment/android\#reviewing-the-gradle-build-configuration.
        # minSdkVersion flutter.minSdkVersion убрал
				minSdkVersion 21
        targetSdkVersion flutter.targetSdkVersion
        versionCode flutterVersionCode.toInteger()
        versionName flutterVersionName
    }
```

### версионирование в java

[Stackoverflow: какие версии джавы каким версиям классов соответствуют](https://stackoverflow.com/questions/9170832/list-of-java-class-file-format-major-version-numbers)

[Wikipedia: java class-file, byte-layout](https://en.wikipedia.org/wiki/Java_class_file#General_layout)

[[java]]

  

# Про приложение

Что мы хотим получить в конце

1. Зайти в приложение. Захардкожен пользователь. Будет сертификат, которым надо будет подписывать транзакции
2. Посмотреть, какие есть события, которые можно посмотреть. Есть два метода: **query** - запросить инфу из блокчейна, **invoke** - запуск пункта в смартконтракте. Посмотреть мои билеты.
3. **Invoke**: Можно билет купить, вернуть. Можно сделать куаркод.  
    Генеришь на фронте приватную строчку, сохраняешь в блокчейн хеш от этой приватной штучки. В куаркоде зашить.