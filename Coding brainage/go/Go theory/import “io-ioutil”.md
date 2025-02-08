---
tags:
  - libs
---
Библиотека с утилитами для работы с файлами.

Запись данных в файл:

```run-go
import (
	"os"
	 "io/ioutil"
)
func main() {
	buffer := []byte("The message the user asked to store via your tg bot")
	
	if err := ioutil.WriteFile("userid/db.txt", buffer, 0600); err != nil {
		os.Exit(1);
	}
}
```

Чтение данных из файла:

```Go
userMessages, err := ioutil.ReadFile("./userid/db.txt")
```

Чтение файлов из директории:

```Go
tinyProgramFiles, err := os.ReadDir("./basics/")
if err != nil {
	fmt.Println("Couldn't open directory basics:", err)
	os.Exit(1)
}
tinyProgramFilesData := make([][]byte, len(tinyProgramFiles))
for _, file := range tinyProgramFiles {
	fmt.Printf("file: %s\n", file.Name())
	if fileData, err := os.ReadFile(fmt.Sprintf("./basics/%s", file.Name())); err == nil {
		tinyProgramFilesData = append(tinyProgramFilesData, fileData)
	} else {
		fmt.Println("Couldn't open file: ", file.Name(), "error", err)
	}
}

for _, fileBytes := range tinyProgramFilesData {
	fmt.Println(string(fileBytes))
}
```