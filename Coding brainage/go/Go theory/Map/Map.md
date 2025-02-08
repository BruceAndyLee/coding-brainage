---
tags:
  - data-structures
---
вроде понятно зачем этот вид структур нужен.

Создавать пустой map через ключевое слово var нет смысла, потому что в переменную запишется ссылочное значение nil и добавить в такую карту ничего не получится. Поэтому карты определяются вот так:

```go

import "fmt"

func main() {
	// Создание пустого расширяемого инстанса:
	squaresMap := make(map[int]int)
	
	// Создание непустого нерасширяемого инстанса:
	squaresMap = map[int]int{
		1: 1,
		2: 4,
		3: 9,
	}
	fmt.Println(squaresMap[1])
}
```

Удаление из карты:

```Go
delete(squaresMap, 3)
```

При обращении к карте по ключу, которого в ней нет, карта вернет нулевое значение, соответствующее тому типу, который назначен для значений карты.

**SUPER-COOL STUFF**: обращение по ключу возвращает еще и второе значение - boolean inMap, который указывает, было ли считано значение, или такого нет в мапе:
```Go
fourSquared, inMap := squaresMap[4]

if inMap {
	// code that uses the value.
}

// в go активно запихиваются инициализации в if/for выражения:

if fourSquared, ok := squaresMap[4]; ok {
	// code that uses the value.
} 
```

Перебор значений в мапе:
```Go
for key, value := range squaresMap {
	fmt.Printf("%d squared is %d\n", key, value)
}
```

Размер мапы (количество пар ключ-значение):
```Go
fmt.Printf("Your squares map size is: %d\n", len(squaresMap))
```

Возможные способы создания отображений и их срезов
![[Untitled 7.png|Untitled 7.png]]