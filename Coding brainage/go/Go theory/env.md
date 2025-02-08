---
tags:
  - libs
---
[Статья](https://habr.com/ru/articles/446468/) про env

```Go
go get github.com/joho/godotenv
```

в функцию `init` (запускается до `main`!!!!) добавляется код для инициализации пакета для чтения переменных окружения:

```Go
import (
	"log"
	"github.com/joho/godotenv"
)

func init() {
	if err := godotenv.Load(); err != nil {
		log.Print("No .env file")
	}
}
```