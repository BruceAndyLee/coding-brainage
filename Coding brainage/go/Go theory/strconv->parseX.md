---
tags:
  - cheatsheets
---
`Atoi`

```Go

a := "15"
b := "10"
diff, err := strconv.Atoi(a) - strconv.Atoi(b)
if err != nil {
	panic(err)
}
```

`ParseInt` + `ParseUint`

```Go
// strconv.ParseInt(str, base, bitSize)

str := "-12345678900000000000000"

parsedInt, err := strconv.ParseInt(str, 10, 64)

if err != nil {
	panic(err) // kills the program bc of an out-of-range error
}
fmt.Println(parsedInt)
```

`ParseFloat`

```Go
// strconv.parseFloat(str, bitSize)

str := "1.0000000001234"
fmt.Println(strconv.ParseFloat(str, 32)) // 1 <nil>
fmt.Println(strconv.ParseFloat(str, 64)) // 1.0000000001234 <nil>
```

`ParseBool`

```Go
str := "true"

fmt.Println(strconv.ParseBool(str)) // true
```