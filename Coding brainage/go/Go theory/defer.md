---
tags:
  - basics
---
С помощью оператора `defer` можно отложить запуск операции (вызов функции? присвоение? объявление переменной?) до того момента, когда текущий блок кода будет завершен.

```Go
func main() {
	defer someTeardown()
	defer somePreTeardown()
	for ind, day := range forever {
		doWork()
	}
} // сначала вызовется somePreTeardown, потом someTeardown
```

В примере зафиксирован тот факт, что дефернутые функции складываются в стек и достаются из него для вызова в порядке LIFO (last in first out)

**Отложенная функция выполнится, даже если где-то внутри блока будет вызвана встроенная функция panic**:

```Go
func saveUserInput(x, y float64) {
	fmt.Printf("\nUser input saved: (%f, %f)\n", );
	// write the input into a log-file or something,
	// Idk, you're the programmer, you figure it out!
}

func divide() {
	if y == 0 {
		panic("\nCannot divide by zero")
	}
	return x / y
}

func main() 
	fmt.Prinf("Provide the floats you'd like divided: ")
	var x, y float64
	fmt.Scan(x, y)
	defer saveUserInput(x, y)
	divide()
}

// ---
// Provide the floats you'd like divided: [1, 0]
// User input saved: (1.0, 0.0)
// panic: Cannot divide by zero
```

---

Если в отложенную функцию передать локальную переменную, то она (функция) получит значение переменной, которое в ней было на момент регистрации отложенного вызова (что, в принципе, логично, но на всякий)

```Go
number := 1
defer someTeardown(number)
number = 2
// someTeardown is called w/ 1
```