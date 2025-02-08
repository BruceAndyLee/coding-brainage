---
tags:
  - libs
---
Пакет удовлетворяет многим базовым интерфейсам (в т.ч. `Reader` и `Writer`).

Используется для создания, переименования, чтения, удаления файлов и для записи в них.

```Go
import "os"

func main() {
	os.Read("todo.txt")
	os.Rename("todo.txt", "latest-todo.txt")
	os.Remove("latest-todo.txt")
	os.Exit(0)
}
```

При работе с файлами надо планировать их закрытие на случай аварийной остановки программы

```Go
f, err := os.Open("daily-log.todo")

if err != nil {
	// lol 
}
defer f.Close()
```

---

Можно получить мета-информацию о прочитанном файле:

```Go
todoFile, err := os.Read("daily-log.todo")

if err != nil {
	os.Exit(1)
}

todoFileMeta, err := todoFile.Stat()

if err != nil {
	os.Exit(1)
}

todoFileMeta // implements the following interface:

type FileInfo interface {
	Name() string       // base name of the file
	Size() int64        // length in bytes for regular files; system-dependent for others
	Mode() FileMode     // file mode bits
	ModTime() time.Time // modification time
	IsDir() bool        // abbreviation for Mode().IsDir()
	Sys() interface{}   // underlying data source (can return nil)
}
```