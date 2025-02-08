---
tags:
  - cheatsheets
---
Пакет с утилитами для (рекурсивного) обхода частей файловой системы.

функция walk из этого пакета реализует паттерн walker (применяет рекурсивно произвольную функцию)

```Go
import (
	"io/fs"
	"path/filepath"
)

func printFileSize(path string, info fs.FileInfo, err error) error {
	if info.IsDir() {
		return nil
	}
	fmt.Println(path+":", info.Size(), "bytes")
	return nil
}

func main() {
	err := filepath("./", printFileSize)
	
	if err != nil {
		panic("Couldn't walk the root:" + err.Error())
	}
}
```