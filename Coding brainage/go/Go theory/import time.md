---
tags:
  - libs
---
Пакет с функциями для перекладывания секунд в даты и обратно.

Простейший вариант создания объекта Time

```Go
time.Date(year int, month time.Month, day int, hour int, min int, sec int, nsec int, loc *time.Location)
```

Создатели придумали нечто _хитрое_ в форматировании дат:

- за самую дефолтную дату, которая ничего кроме простоты не значит и ни чем не обладает, взяли 01 02 3:04:05 06
    
    - 01 - первый месяц
    - 02 - второе число
    - 03 - три часа дня
    - 04 - четыре минуты
    - 05 - пять секунд
    - 06 - 2006-й год
    
    и еще какие-то приколы для часового пояса…
    
- каждое из этих значений считается управляющим символом, сигнализирующем, что “здесь нужно вставить извлеченный из даты месяц/день/часы/минуты/секунды/год” соответственно
- у каждого такого управляющего символа есть несколько вариаций, чтобы можно было выводить каждую из частей даты в нескольких форматах

- спецификация форматирования
    
    ```Go
    Year 	    06     2006
    Month 	  01     1         Jan          January
    Day 	    02     2         _2           (width two, right justified)
    Weekday 	Mon    Monday
    Hours 	  03     3         15
    Minutes 	04     4
    Seconds 	05     5
    ms μs ns 	.000   .000000   .000000000
    ms μs ns 	.999   .999999   .999999999   (trailing zeros removed)
    am/pm 	  PM     pm
    Timezone 	MST
    Offset 	  -0700  -07       -07:00       Z0700   Z07:00
    ```
    
- наиболее часто встречающиеся шаблоны
    
    ```Go
    ANSIC       = "Mon Jan _2 15:04:05 2006"
    UnixDate    = "Mon Jan _2 15:04:05 MST 2006"
    RubyDate    = "Mon Jan 02 15:04:05 -0700 2006"
    RFC822      = "02 Jan 06 15:04 MST"
    RFC822Z     = "02 Jan 06 15:04 -0700"
    RFC850      = "Monday, 02-Jan-06 15:04:05 MST"
    RFC1123     = "Mon, 02 Jan 2006 15:04:05 MST"
    RFC1123Z    = "Mon, 02 Jan 2006 15:04:05 -0700"
    RFC3339     = "2006-01-02T15:04:05Z07:00"
    RFC3339Nano = "2006-01-02T15:04:05.999999999Z07:00"
    Kitchen     = "3:04PM"
    // Handy time stamps.
    Stamp      = "Jan _2 15:04:05"
    StampMilli = "Jan _2 15:04:05.000"
    StampMicro = "Jan _2 15:04:05.000000"
    StampNano  = "Jan _2 15:04:05.000000000"
    ```
    

---

Из этих кусков можно собирать свой шаблон.

С помощью шаблона можно объяснить пакету time как парсить строку (`time.Parse`), в которой зашита дата, или как отформатировать конкретную дату (`time.Format`), записанную в секундах.

Таким образом, вывести дату в привычном формате можно, например, вот так:

```Go
// dd.mm.yyyy hh:mm:ss
betaFormat := "02.01.2006 03:04:05"
currentTime := time.Now().Format(betaFormat)
fmt.Println("current date-time:", currentTime)
```

Чтобы отобразить время с учетом конкретного часового пояса, можно этот часовой пояс загрузить из [каталога](https://www.iana.org/time-zones) встроенным методом `time.LoadLocation`

```Go
loc, _ := time.LoadLocation("Europe/Moscow")

localizedTime, _ := time.ParseInLocation(betaFormat, currentTime, loc)
fmt.Println("current local date-time:", localizedTime.Format(betaFormat))
```

---

- методы для извлечения фрагментов из даты
    
    ```Go
    current := time.Date(2020, time.May, 15, 17, 45, 12, 0, time.Local)
    
    fmt.Println(current.Date()) // 2020 May 15
    
    fmt.Println(current.Year()) // 2020
    
    fmt.Println(current.Month()) // May
    
    fmt.Println(current.Day()) // 15
    
    fmt.Println(current.Clock()) // 17 45 12
    
    fmt.Println(current.Hour()) //17
    
    fmt.Println(current.Minute()) // 45
    
    fmt.Println(current.Second()) // 12
    
    fmt.Println(current.Unix()) // 1589546712
    
    fmt.Println(current.Weekday()) // Friday
    
    fmt.Println(current.YearDay()) // 136
    ```
    
- перемещение между датами
    
    ```Go
    sameDayLastYear := time.Now().AddDate(-1, 0, 0)
    
    fmt.Println("this day one year ago:", sameDayLastYear.Format(betaFormat))
    ```
    
    ```Go
    leapDay2024 := time.Date(2024, time.February, 29, 0, 0, 0, 0, loc)
    
    fmt.Println("Exaccly one year before the leapDay:", leapDay2024.AddDate(-1, 0, 0).Format(betaFormat))
    // Wednesday 03.01.2023 12:00:00
    ```
    
    ```Go
    // time.Sub() returns the duration in nanoseconds
    fmt.Println("useless subbing:", time.Now().Sub(time.Now().Add(1000000000)))
    // useless subbing: -1.000000109s
    ```