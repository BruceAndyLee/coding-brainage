---
tags:
  - basics
---
Сам по себе recover нужен для того, чтобы превратить [[errors + panic]] в используемом коде в ошибку в вызывающем коде.

_Грубо говоря, в случае, когда я пишу просту функцию для деления чисел, будет разумно выкинуть панику при делении на ноль и убить всю программу. Если же программа большая, состоит из кучи модулей и обрабатывает огромное количество сценариев (ну хоть и интерактивный терминальный калькулятор), то было бы странно её убивать целиком при попытке юзера разделить на ноль. В такой ситуации разумнее в функции-вычислителе введенного выражения настроить_ `_recover_` _и обрабатывать деление на ноль как ошибку, показывая её пользователю в коносли._

---

[[errors + panic]] можно обработать инлайново с помощью [[анонимные функции]]

```Go
func panickingFunc() {
	defer func() (err error) {
		if e := recover(); e != nil {
			err = e.(error)
			// this lambda doesn't return anything,
			// cuz it's basically a procedure:
			// the type of function that only changes the values
			// from it's lexical environment
		}
	}

	panic(errors.New("Things will fall apart!"))
}

// -------

func main() {

	// whatever was put inside the err variable in the deferred lambda
	// will get proxied upwards as an error value:
	if res, err := panickingFunc(); err != nil {
		fmt.Println("Function returned an error:", err);
		// Function returned an error: Things will fall apart!
	}

}
```