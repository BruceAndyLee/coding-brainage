---
tags:
  - cheatsheets
---
Максимальные значения, помещающиеся в типы данных:

```Go
fmt.Println(math.MaxInt8)   // 127
fmt.Println(math.MaxUint8)  // 255
fmt.Println(math.MaxInt16)  // 32767
fmt.Println(math.MaxUint16) // 65535
```

Декларации констант

```go
const firstName = "lol" // <---- NO COLON HERE <----
const lastName = "kekker" // <---- NO COLON HERE <----
```

Константы можно вычислять, если все значения известны на момент компиляции
```go
const firstName = "lol" // <---- NO COLON HERE <----
const lastName = "kekker" // <---- NO COLON HERE <----
const name = firstName + " " + lastName 
```