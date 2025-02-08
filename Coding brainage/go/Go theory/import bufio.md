---
tags:
  - libs
---
Содержит инструменты для чтения из и записи в объекты, удовлетворяющие интерфейсам `io.Reader` и `io.Writer`.

Использование пакета для чтения из файла в буфер и из буфера куда угодно:

```Go
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
)

func main() {
	f, err := os.Open("./output/bufio-writer.txt")
	if err != nil {
		panic("could not open file" + err.Error())
	}
	defer f.Close()

	byteReader := bufio.NewReader(f)

	bytes := make([]byte, 10)

	n, err := byteReader.Read(bytes)

	if err != nil && err != io.EOF {
		panic("I broke down trying to read the files!" + err.Error())
	}

	fmt.Printf("I read %d bytes!\n", n)
	fmt.Printf("here they are as a string:", string(bytes))
}
```

Использование пакета для записи буфера в файл

```Go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	f, err := os.Create("./output/bufio-writer.txt")
	if err != nil {
		panic("could not create file" + err.Error())
	}
	defer f.Close()

	byteWriter := bufio.NewWriter(f)

	n, err := byteWriter.Write([]byte("hello file"))

	if err != nil {
		panic("I shat myself trying to write a string to file!" + err.Error())
	}

	fmt.Printf("Successfuly wrote %d bytes to file!\n", n)
	byteWriter.Flush()
}
```

Использование пакета для построчного чтения данных

```Go
file, err := os.Open("test.txt")
if err != nil {
	panic(err)
}
defer file.Close()

s := bufio.NewScanner(file)

// Я заранее записал в файл 5 цифр, каждую на новой строке
for s.Scan() { // возвращает true, пока файл не будет прочитан до конца
	fmt.Printf("%s\n", s.Text()) // s.Text() содержит данные, считанные на данной итерации
}
```

Так же можно читать буфер целиком и записывать в него сразу несколько кусков информации

```Go
writer := csv.WriteAll([][]string{
	{"header1", "header2", "header3"},
	{"value11", "value12", "value13"}
})
```