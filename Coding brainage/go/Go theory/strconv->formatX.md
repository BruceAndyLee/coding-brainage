---
tags:
  - cheatsheets
---
Библиотека для форматирования чисел в строки.

```Go
import "strconv"

func main() {
```

  

int to ascii (aka Itoa)

```Go
var yearStr string = strconv.Itoa(2024) // "2024"
```

`FormatInt` и `FormatUint`

```Go
// hex to decimal
var strconv.FormatInt(0xB, 10) // "11"
var strconv.FormatInt(0xB, 16) // "b"

// FormatUint
var a uint64 = 10101
res := strconv.FormatUint(a, 10)
fmt.Println(res) // 10101
```

`FormatFloat`

```Go
// (theFloat, "fmt-template", prec, bitSize)
// ---------------------------------
// Возможные варианты fmt-template:
// 'f' (-ddd.dddd, no exponent),
// 'b' (-ddddp±ddd, a binary exponent),
// 'e' (-d.dddde±dd, a decimal exponent),
// 'E' (-d.ddddE±dd, a decimal exponent),
// 'g' ('e' for large exponents, 'f' otherwise),
// 'G' ('E' for large exponents, 'f' otherwise),
// 'x' (-0xd.ddddp±ddd, a hexadecimal fraction and binary exponent), or
// 'X' (-0Xd.ddddP±ddd, a hexadecimal fraction and binary exponent).
// ---------------------------------
// возможные варианты prec (колчиство цифр после запятой):
// -1 - вывести все цифры после запятой
// 
var b float64 = 2222 * 1023 * 245 * 2 * 52
fmt.Println(strconv.FormatFloat(b, 'e', -1, 64)) // 5.791874088e+10
```

`FormatBool`

```Go
var a = true
res := strconv.FormatBool(a)
fmt.Println(res)     	// "true"
```