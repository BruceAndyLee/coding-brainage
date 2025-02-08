---
tags:
  - cheatsheets
---
Это такой интерфейс, которому должны удовлетворять объекты, если мы хотим, чтобы их можно было привести к строке (грубо говоря, этому интерфейсу удовлетворяют все сущности в js, у которых имеется метод `toString`).

В реализации этого интерфейса стоит воздерживаться от `Sprintf(”%v, %T”)`

```Go
type Battery struct {
	capacity int
	level    int
}

// [ _ _ _ _ _ X X X X] <- where X is a charged slot
func (b Battery) String() string {
	return fmt.Sprintf("[%10s]", strings.Repeat("X", b.level))
}
```