---
tags:
  - basics
---
Ошибки возвращаются в качестве значений. Их наличие проверяется перед продолжением работы программы.

Можно создавать новые ошибки, оборачивая полученную от другого вызова.

```Go
package main

import (
	"errors"
	"fmt"
)

func readNumber() (readNumber int, readCount int, err error) {
	var num int
	elementsReadCount, error := fmt.Scan(&num)

	if error != nil {
		return 0, 0, errors.New(fmt.Sprintf("Couldn't process your input: %s", error))
	}
	return num, elementsReadCount, nil
}

func main() {

	number, elementsReadCount, error := readNumber()

	if error != nil {
		fmt.Println(error)
		return
	}
	fmt.Printf("read number: %d, (%d elements)\n", number, elementsReadCount)
}
```

Можно остановить программу в любом месте при возникновении недопустимой ситуации типа деления на ноль.

```Go
func divide(int a, float64 b) float64 {
	if (b == 0) {
		panic("Division by zero! The program cannot proceed")
	} else {
		return a / b
	}
}
```

Вывод сообщения из паники (если паника не обработана с помощью [[recover]]) будет запланирован в самый конец работы программы:

```Go
func main() {
	if res, err := someFunc(); err != nil {
		defer func () {
			fmt.Println("I tried my best, sorry...")
		}()
		fmt.Println("Oh God, I am overwhelmed!..")
		panic("someFunc errorred!")
	}
}

// Oh God, I am overwhelmed!..
// I tried my best, sorry...    <- because it is deferred
// someFunc errorred            <- because it is the panic-func argument
```