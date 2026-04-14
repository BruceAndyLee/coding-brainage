# JSON tinkering

### данные для примеров:

```JSON
// пусть в таблице MUSIC есть jsonb-поле GUITAR_STAFF
guitar_stuff: {
	guitars: [
		{
			name: "PRS",
		    guitarists: [
				name: "Santana",
			]
		},
		{
			name: "Gibson",
		    guitarists: [
				{ name: "Jimmy Page", },
				{ name: "Joe Bonamassa", }
		    ]
		},
	]
}
```

### Значение по ключу

```SQL
-- достаем значение из jsonb-поля
SELECT
guitar_stuff -> 'guitars'
from music
```

```JSON
// результат: одна запись - массив guitars:
[
	{
		name: "PRS",
		guitarists: [
			name: "Santana",
		]
	},
	{
		name: "Gibson",
		guitarists: [
			{ name: "Jimmy Page", },
			{ name: "Joe Bonamassa", }
		]
	},
]
```

### вроде left-join (jsonb_array_elements)

```SQL
-- guitar_stuff.guitars - массив
-- jsonb_array_elements раскроет каждый элемент массива в строчку/запись в ответе
SELECT
jsonb_array_elements(guitar_stuff -> 'guitars')
from music
```

```JSON
// результат: две записи - каждая jsonb:
{ name: "PRS", ... }
{ name: "Gibson", ... }
```

---

```SQL
-- можно после вызова этой функции достать еще одно поле:
SELECT
jsonb_array_elements(guitar_stuff -> 'guitars') -> 'name'
from music
```

```SQL
// результат: две записи - каждая текстовое значение:
"PRS"
"Gibson"
```

---

```SQL
-- такое раскрытие можно выполнять произвольное количество раз:
SELECT
(jsonb_array_elements(guitar_stuff -> 'guitars') -> 'guitarists')
from music
```

```JSON
// результат: три отдельных записи:
{ name: "Santana" }
{ name: "Jimmy Page", }
{ name: "Joe Bonamassa", }
```

### раскрытие по нескольким полям сразу (jsonb_array_elements)

```SQL
-- плоская запись всех маршрутов в дереве
-- if you get my manner of saying
SELECT
json_array_elements(guitar_stuff -> 'guitars') -> 'name' as guitar,
jsonb_array_elements(guitar_stuff -> 'guitars') -> 'guitarists') -> 'name'  as guitarist
from music
```

```JSON
// три строчки:
----guitar-------guitarist---
|   "PRS"   |    "Santana"  |
| "Gibson"  |  "Jimmy Page" |
| "Gibson"  |"Joe Bonamassa"|
```

Рекурсивно значение уровнем выше будет продублировано для каждого значения на уровень ниже. Таким образом, количество записей в результате запроса будет равно количеству уникальных значений на уровне “листьев” дерева json-а

### реальный пример с закидонами (jsonb_array_elements)

```SQL
SELECT
  equipment.id AS equipment_id,
  (
    jsonb_array_elements((equipment.meta -> 'operations' :: text)) ->> 'key' :: text
  ) AS operation_type_key,
  (
    jsonb_array_elements(
      (
        jsonb_array_elements((equipment.meta -> 'operations' :: text)) -> 'housings' :: text
      )
    ) ->> 'role' :: text
		-- для строгости 'role' кастуется к тексту,
		-- именно текстовым значением можно обратиться по ключу внутрь json
  ) AS role,
  (
    (
      jsonb_array_elements(
        (
          jsonb_array_elements((equipment.meta -> 'operations' :: text)) -> 'housings' :: text
        )
      ) ->> 'type_id' :: text
    )
  ) :: integer AS housing_type_id,
  (
    (
      jsonb_array_elements(
        (
          jsonb_array_elements((equipment.meta -> 'operations' :: text)) -> 'housings' :: text
        )
      ) ->> 'max_count' :: text
    )	
  ) :: integer AS max_count
	-- результат раскрытия поля оборачивается в скобки для кастования к 
	-- целому числу - ситуативная мера 
FROM
  equipment;
```
