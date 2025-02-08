---
tags:
  - basics
---
Ключевое слово keyword нужно для более короткой записи циклов.

Range возвращает копию элемента.

```Go
// key-word range can be used for concise for loops declaration

array := [5]int{1, 2, 3, 4, 5} 

for ind, elem := range array {
	fmt.Printf("[%d]:%d ", ind, elem) // [0]:1 [1]:2 [2]:3 [3]:4 [4]:5
}

// ... is the same as
for i := 0; i < len(array); i++ {
	fmt.Printf("[%d]:%d ", ind, elem)
}
```

Оператор `range` предоставляет для исполнения каждой итерации индекс и элемент.

Эти значения можно пропускать:

```Go
for ind := range array {
	fmt.Printf("[%d] ", ind, elem) // [0] [1] [2] [3] [4]
}

// if ind is of no use, it has to be replaced w/ underscore (_)
for _, el := range array {
	fmt.Printf("[%d] ", ind, elem) // [0] [1] [2] [3] [4]
}
```