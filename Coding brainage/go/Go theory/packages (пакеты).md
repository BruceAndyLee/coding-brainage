---
tags:
  - pain
---
пакет - это набор сурс-файлов в одной директории. Сурсфайлы видят друг друга и компилируются вместе

исполняемый пакет должен называться `main`, функция `main` является точкой входа. Все остальные пакеты могут только импортироваться. В одном пакете может быть несколько файлов. В таком случае, функции можно вызывать друг через друга без декларации импортов.

```Go
// entry-point.go

package main

func main() {
	initEndpoints(
		// endpoint-cfgs from argc/argv
	)
}

// api-module.go
func initEndpoints() {
	// endp initialization code
	fmt.Println("Initialization complete... Should be available at port 9090")
}
```

Запуск ентри-поинта:

```Bash
go run .
go run main.go api-module.go
```

---

импортирование без сохранения `namespace` (типа `import * from “@/lol.js”`)

```Go
package main
import . "fmt"

func main() {
	Println("Obscured hello world");
}
```

импортирование с переименованием модуля (типа `import HellYeah from “@/hell_no.jks”`)

```Go
package main
import AliasedFMT "fmt"

func main() {
	AliasedFMT.Println("Obscured hello world");
}
```