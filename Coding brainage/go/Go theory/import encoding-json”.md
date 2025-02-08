---
tags:
  - libs
---
### Общие соображения и пример

Пакет для (де)сериализации данных в формате json.

Основные методы:

- `marshal`
- `unmarshal`

Оба метода работают с массивами байтов и с указателем на сущность, типизированную с помощью `type struct`.

Сериализуются только публичные поля, то есть, те, имена которых начинаются с большой буквы.

---

Базовый пример: читаем список студентов в группе и выводим среднее количество оценок, которое они получили.

- Простейшая реализация без сокращений и красивостей
    
    ```Go
    // declare the structs
    type Student struct {
    	LastName   string
    	FirstName  string
    	MiddleName string
    	Birthday   string
    	Address    string
    	Phone      string
    	Rating     []int
    }
    
    type Group struct {
    	ID       int
    	Number   string
    	Year     int
    	Students []Student
    }
    
    type RatingCountAverage struct {
    	Average float64
    }
    ```
    
    ```Go
    // read the json strings from the stdin
    studentsData, err := io.ReadAll(os.Stdin)
    if err != nil {
    	panic("Couldn't read the JSON from stdin: " + err.Error())
    }
    ```
    
    ```Go
    // de-serialize the data
    group := new(Group)
    err = json.Unmarshal(studentsData, group)
    
    if err != nil {
    	panic("Couldn't parse JSON into a group object: " + err.Error())
    }
    
    // ... and perform the calculations
    markCount := 0
    
    for _, student := range group.Students {
    	markCount += len(student.Rating)
    }
    ```
    
    ```Go
    // create the response-struct instance
    average := &RatingCountAverage{float64(markCount) / float64(len(group.Students))}
    
    // serialize it and show to the user
    marshalledAverage, err := json.MarshalIndent(average, "", "    ")
    
    if err != nil {
    	panic("Couldn't serialize the average struct: " + err.Error())
    }
    fmt.Println(string(marshalledAverage))
    ```
    
- Куча сокращений: `Decoder/Encoder` + `anonymous struct`
    
    ```Go
    // 1.
    // Remove all the fields that do not pertain to the cause
    type struct Group {
      // and create an inline struct for the students
    	Students []struct {
    		Rating []int
    	}
    }
    ```
    
    ```Go
    // 2.
    // use json.Decoder and Encoder
    func main() {
      group := Group{}
    	json.NewDecoder(os.Stdin).Decode(&group)
    	
    	// the same thing w/ counting the average
    	var average float64
    	for _, student := range group.Students {
    		average += float64(len(student.Rating))
    	}
    	average /= float64(len(group.Students))
    	
    	// use json.Encoder w/ yet another anonymous struct
    	json.NewEncoder(os.Stdout).Encode(struct{ Average float64 }{average})
    }
    ```
    

---

### Управление сериализацией

чтобы не использовать поля в том же виде, в котором они хранятся внутри структуры, можно использовать специальные аннотации:

```Go
type Person struct {
	Name string `json:"name"`
	Age int `json:",omitempty"` // is not included in the serialized str when empty value
	Alive bool `json:"-"` // always omit
}
```