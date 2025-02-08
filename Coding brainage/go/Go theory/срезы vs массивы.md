---
tags:
  - basics
---
Синоним динамического массива. Задается тем же синтаксисом, что и обычный массив, но должна быть пропущена длина:

```Go
var arr = [5]int{1, 2, 3} // creates an array
var slice = []int{1, 2, 3} // creates a slice

// creates a slice of length 10, all elements are initialized as zeros
var madeSlice = make([]int, 10)

// creates a slice of length 0, w/ capacity 10, there are no elements.
// which means, the slice won't have to call for realloc
// until the capacity of 10 is exceeded.
var madeSliceEmpty = make([]int, 0, 10)
```

Слайсы представлены структурой из трех значений: длина, вместительность и указатель на начало области в памяти. Таким образом, слайсы можно сравнивать и перезаписывать друг в друга, потому что у двух разных слайсов один и тот же тип.

Массивы же, скорее, описываются структурой, которая, груба говоря, создается каждый раз под каждый конкретный массив заданной длины.

```Go
var array 5ints = [5]int{1,2,3,4,5}

// array's type can be thought of as
type 5ints struct {
	el0 int
	el1 int
	el2 int
	el3 int
	el4 int
}
// initialization of every array's element is equivalent
// to how you would initialize the fields of a struct as shown above
```

Для сравнения, слайс/срез описывается вот такой структурой:

```Go
type slice struct {
	length uint64
	capacity uint64
	ptr *int
}

// struct init somewhere inside the 'make' function
// ...
// slice{length, capacity, pointerToSystemAllocatedMemory}
```