---
tags:
  - basics
---
Срезы можно создавать из массивов, указателей на массивы и других срезов:

```Go
var namesArray = [5]string{"Kekker", "Loller", "Brenner", "Caller"}

namesSlice := namesArray[:3] // ["Kekker", "Loller", "Brenner"]
namesSliceSlice := namesSlice[1:] // ["Loller", "Brenner"]
```

Созданный из массива срез хранит внутри указатель на тот элемент массива, с которого срез начинается. До того, как появится необходимость выделить новую память внутри среза, будет использоваться исходный массив, поэтому capacity при создании среза оператором будет равна `len(array) - sliceStart`. Именно в исходный массив будут сохраняться элементы, при вызове функции `append`

Функция `append` добавляет элементы в срез.  
Прячет от программиста логику  
`realloc`. Возвращает новый срез, который ссылается на тот же массив (с добавленными элементами) или на новый, если требуется реаллокация. Если памяти недостаточно, то создается новый массив с удвоенной длинной исходного массива.

```Go
// helper function to get the array pointer from within slice
func toPtr(slice []Type) {
	return fmt.Sprintf("%p", slice)
} 

// the function under scrutiny
// func append(slice []Type, elems ...Type) []Type

namesArray := {4}string{"Reedtz", "Rasmussen", "Brenner", "HØlsten"}

slicedNames = namesArray[2:4] // ["Brenner", "HØlsten"]
initialSliceArrayPtr = toPtr(slicedNames)
slicedNames = append(slicedNames, "Kostylev", "Sharipov")
fmt.Println(namesArray) // ["Reedtz", "Rasmussen", "Brenner", "HØlsten"]
fmt.Println(slicedNames) // ["Brenner", "HØlsten", "Kostylev", "Sharipov"]
fmt.Println(toPtr(slicedNames) == initialSliceArrayPtr) // false
// this last check is false because the capacity of the slice was exceeded
// with the 'append' call and a new array was created under the hood.
```

Срезы можно сливать вместе троеточием
```Go
names := load_names_from_bd()

// sometime later in an update function of some sort:
names = append(names, newely_posted_names_slice...)
```

при достижении capacity `append` создаст новый слайс, который будет независим от старого 

Пропуск элемента с помощью `append`
```Go
// problem 1: the before of the slice should be copied first, cuz otherwise the
//
//	appended values will get inserted into the input array
//
// problem 2: when copying the original slice, copy-func's destination spliceSlice
//
//	needs to be created w/ a specified length.
//	Elements that would overflow the dts-slice's initial length are omitted
func spliceSlice(slice []int, spliceInd int) []int {
	newSlice := make([]int, spliceInd)
	copy(newSlice, slice[:spliceInd])
	return append(newSlice, slice[spliceInd+1:]...)
}
```

Правильное копирование с предварительной аллокацией
```Go
// the source data to be copied
var srcBuf = []int{1, 2, 3, 4, 5}

// the destination slice to copy the data to
var dstBuf = make(int, len(srcBuf), len(srcBuf)) // this is correct.

// nothing will be copied if the dst slice is of capacity 0
```

функцией `copy` можно копировать данные в середину среза
```Go
var srcBuf = []int{1, 2, 3, 4, 5}

var dstBuf = []int{0, 1, 0, 1}

copy(dstBuf[1:3], srcBuf[2:4]) // 0 3 4 1
```