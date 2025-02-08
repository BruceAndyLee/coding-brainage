---
tags:
  - essentials
---
Служит для объявления набора методов, которые должны быть реализованы на сущности, чтобы она считалась инстансом интерфейса.

```Go
type Reader interface {
	Read(p []byte) (n int, err error)
}

type Writer interface {
	Write(p []byte) (n int, err error)
}
```

Conicidentally, могут быть использованы для передания значений с неизвестным типом.

```Go
type SugaryArray struct {
	value []interface{}
}

func (arr SugaryArray) Map(mapper func (el interface{}, ind int) bool) SugaryArray {
	res := make([]interface{}, len(arr.value))
	for ind, el := range arr.value {
		res[ind] = mapper(el, ind)
	}
	return SugaryArray{value: res}
}
```

Интерфейсы можно объединять в композицию (тогда сущность должна удовлетворять обоим объединенным интерфейсам, чтобы считаться инстансом интерфейса-объединения)

```Go
type adder interface {
	add(float64, float64) float64
}

type subtractor interface {
	sub(float64, float64) float64
}

type multiplier interface {
	mul(float64, float64) float64
}

type divider interface {
	div(float64, float64) float64
}

type calculator interface {
	adder
	subtractor
	multiplier
	divider
}
```

### Приведение типа интерфейса

Конструкция, которая позволяет принимать разные решения в зависимости от типа переданного значения:

```Go
var number interface{} = 1

if coerced, ok := number.(int); ok {
	fmt.Println("Unknown type has been successfully cast to int:", coerced)
}
```

и реализовать переключатель типов:

```Go
func simpleTypeChecker(val interface{}) {
	switch val.(type) { // this to-type coercion is only available for the switch block heading
	case int:
		fmt.Println("integer")
	case string:
		fmt.Println("string")
	default:
		fmt.Printf("I am too simple to know type %T\n", val)
	}
}
```