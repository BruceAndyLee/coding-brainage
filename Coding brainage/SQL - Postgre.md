---
Status: In progress
tags:
- databases
- sql
---
# Basics

Все примеры [отсюда](https://www.sql-practice.com/)

- ==**HAVING**== фильтрация и аггрегация (where может фильтровать только единичные записи, но не аггрегируемые группы)
    
    ```SQL
    # оставляем только те имена, которые встречаются один раз! крейзи!
    SELECT
    	first_name
    FROM patients
    GROUP BY first_name
    HAVING COUNT(*) = 1;
    ```
    
- **LEN + LIKE** Символы в строках + длина строки
    
    ```SQL
    select
      patient_id,
      first_name
    from patients
    where
      first_name LIKE "s%s"
      and len(first_name) >= 6;
    ```
    
- ==**NO FROM**== аггрегации внутри одной таблицы (типа **horizontal union**)
    
    “вы такой себе бэкендер” варик:
    
    ```SQL
    SELECT
    	(SELECT * from patients where gender='M') as males,
    	(SELECT * from patients where gender='F') as females
    FROM patients
    LIMIT 1;
    ```
    
    “вы восхитительны” варик
    
    ```SQL
    SELECT 
      (SELECT count(*) FROM patients WHERE gender='M') AS male_count, 
      (SELECT count(*) FROM patients WHERE gender='F') As female_count;
    ```
    
- **GROUP BY +** ==**HAVING**== Фильтрация с аггрегацией по двум полям
    
    ```SQL
    # выбор пациентов, которые приходили больше одного раза,
    # больше одного раза уходя с одним и тем же диагнозом 
    SELECT
      patient_id,
      diagnosis
    FROM admissions
    GROUP BY
      patient_id,
      diagnosis
    HAVING COUNT(*) > 1;
    ```
    
- **GROUP BY + COUNT** Группировка с аггрегацией по одному и тому же полю
    
    ```SQL
    # группирцем пациентов по городу
    # в каждую группу добавляем количество пациентов, проживающих в городе
    # полученные записи упорядочиваем по количеству и по названию города
    SELECT
      city,
      COUNT(*) as patients_in_the_city
    FROM patients
    GROUP BY city
    ORDER BY
      patients_in_the_city DESC,
      city ASC;
    ```
    
- ==**UNION**== + **constant column** конкатенация 1+ запросов
    
    ```SQL
    # вывести всех людей с колонкой role: "Doctor" | "Patient"
    SELECT
      patients.first_name,
      patients.last_name,
      "Patient" as role
    from patients
    UNION ALL
    select
      doctors.first_name,
      doctors.last_name,
      "Doctor" as role
    from doctors;
    ```
    
- **GROUP BY + ORDER + COUNT** - сортировка по популярности
    
    ```SQL
    SELECT
      allergies,
      COUNT(*) AS people_susceptible
    FROM patients
    WHERE allergies IS NOT NULL
    GROUP BY allergies
    ORDER BY COUNT(*) DESC;
    ```
    
- **YEAR + ORDER BY + BETWEEN-AND** - фильтрация по десятилетию
    
    Сравниваем строки
    
    ```SQL
    # выводим всех пациентов, родившихся в семидесятых
    SELECT
      first_name,
      last_name,
      birth_date
    from patients
    where year(birth_date) LIKE '197%'
    order by birth_date;
    ```
    
    Проверяем что дата попадает в период
    
    ```SQL
    SELECT
      first_name,
      last_name,
      birth_date
    FROM patients
    WHERE
      YEAR(birth_date) BETWEEN 1970 AND 1979
    ORDER BY birth_date ASC;
    ```
    
- **CONCAT + ORDER BY**
    
    ```SQL
    # собираем имена в строки вида "MILLER,zoe"
    SELECT
      concat(upper(last_name), ",", lower(first_name)) AS full_name
    FROM patients
    ORDER BY first_name DESC;
    ```
    
- **GROUP BY + SUM_aggr +** ==**HAVING**== - можно делать having по аггрегированному полю
    
    ```SQL
    # считаем суммарный рост по провинциям и оставляем провинции
    # с суммарным ростом, перевалившим за значение
    SELECT
      province_id,
      SUM(height) AS height_sum
    FROM patients
    GROUP BY province_id
    HAVING height_sum >= 7000;
    ```
    
- **MAX + MIN** вычисление интервала значений в столбце
    
    “похуже” - менее читаемый вариант с двумя подзапросами и дублированием кода
    
    ```SQL
    SELECT - (
        SELECT MIN(weight)
        FROM patients
        WHERE last_name = 'Maroni'
      ) + (
        SELECT MAX(weight)
        FROM patients
        WHERE
          last_name = 'Maroni'
      ) AS weight_delta;
    ```
    
    “нормалды” - без подзапросов, просто две агрегации
    
    ```SQL
    SELECT
    	(MAX(weight) - MIN(weight)) as weight_delta
    FROM patients
    WHERE last_name = 'Maroni';
    ```
    
- **DAY + GROUP BY + ORDER BY** извлечение дня из даты + сортировка по популярности
    
    ```SQL
    SELECT
      DAY(admission_date) AS admission_day,
      COUNT(*) AS admissions_number
    FROM admissions
    GROUP BY DAY(admission_date)
    ORDER BY admissions_number DESC;
    ```
    
- Извлечение записи по максимуму колонки (e.g. latest admission)
    
    ORDER BY + LIMIT
    
    ```SQL
    SELECT *
    FROM admissions
    WHERE patient_id = 542
    ORDER BY admission_date DESC
    LIMIT 1;
    ```
    
    GROUP BY + HAVING
    
    ```SQL
    SELECT *
    FROM admissions
    WHERE patient_id = 542
    GROUP BY patient_id
    HAVING
      admission_date = MAX(admission_date);
    ```
    
- **JOIN | LEFTJOIN** вывод данных с аггрегацией по отдельной таблице
    
    группировка и аггрегация по докторам в подзапросе + left join
    
    ```SQL
    # подсчет посещений для каждого доктора.
    # Посещения в отдельной таблице
    SELECT
      first_name,
      last_name,
      number_of_admissions
    FROM doctors
    LEFT JOIN (
      SELECT
        COUNT(*) AS number_of_admissions,
        attending_doctor_id
      FROM admissions
      GROUP BY
        attending_doctor_id
    ) ON attending_doctor_id = doctor_id;
    ```
    
    группировка и агрегация на верхнем уровне.
    
    ```SQL
    # 1. Получаем огромную таблицу на верхнем уровне
    #  - каждый доктор повторяется столько раз, сколько раз его посещали
    # 2. Группируем записи в ней по айди доктора
    #  - внутрь каждой группы добавляем аггрегированное поле
    SELECT
      first_name,
      last_name,
      COUNT(*) as number_of_admissions
    FROM doctors
      JOIN admissions ON attending_doctor_id = doctor_id
    GROUP BY doctor_id;
    # Note: здесь достаточно обычного джоина (он же INNER)
    # В результирующей таблице будут только те записи из обеих таблиц
    # которые нашли себе пару в другой таблице, то есть, если бы какой-то
    # из докторов оказался без приёмов - его бы не включили в результат
    # и наоборот, все приемы, в которых не участвовал доктор - никак 
    # не могут и не будут фигурировать в результате.
    ```
    
- **JOIN | LEFT JOIN** применение функции к аггрегированным данным (==**!!!!**==)
    
    ```SQL
    # Аналогичная предыдущей задача,
    # но приемы надо не посчитать, а найти первый и последний.
    SELECT
      doctor_id as id,
      CONCAT(first_name, " ", last_name) as full_name,
      MAX(admission_date) as last_admission_date,
      MIN(admission_date) AS first_admission_date
    FROM doctors
      JOIN admissions on attending_doctor_id = doctor_id
    group by doctor_id;
    ```
    
    (==!!!!==) **Внимание вопрос**: что выведется, если не написать в конце **group by**, и почему?
    
- **LEFT JOIN x2 + string concat shortcut**
    
    ```SQL
    SELECT
      pt.first_name || ' ' || pt.last_name as patient,
      ph.first_name || ' ' || ph.last_name AS doctor,
      adm.diagnosis
    FROM admissions adm
      LEFT JOIN patients pt ON pt.patient_id = adm.patient_id
      LEFT JOIN doctors ph ON ph.doctor_id = adm.attending_doctor_id;
    ```
    
- **GROUP BY x2**
    
    ```SQL
    SELECT
      first_name,
      last_name,
      COUNT(*) AS duplicated
    from patients
    GROUP BY
      first_name,
      last_name
    HAVING duplicated > 1;
    ```
    
- количество уникальных дубликатов
    
    ```SQL
    SELECT COUNT(*) AS duplicates
    FROM (
        SELECT
          first_name || last_name as full_name,
          "Duplicate" AS duplicate
        from patients
        group by full_name
        HAVING COUNT(*) > 1
      )
    group by duplicate;
    ```
    
- **DISTINCT + JOIN** вычищение дубликатов после джоина
    
    ```SQL
    SELECT
      DISTINCT p.patient_id,
      p.patient_id || FLOOR(LEN(last_name)) || FLOOR(YEAR(birth_date)) as temp_password,
      adm.admission_date
    FROM patients p
      INNER JOIN admissions adm ON adm.patient_id = p.patient_id;
    ```
    

# Basics + math

- **FLOOR + GROUP BY** разбиение на группы-промежутки на ч. прямой
    
    ```SQL
    SELECT
      COUNT(*) AS patients_in_group,
      FLOOR(weight / 10) * 10 as weight_group
    FROM patients
    GROUP BY (weight / 10)
    ORDER BY weight_group DESC;
    ```
    
- **POWER + CASE** (aka ternary)
    
    ```SQL
    # Вычисляем наличие ожирения по пороговому значению для каждого пациента
    SELECT
      patient_id,
      weight,
      height,
      (
        CASE
          WHEN weight / (POWER(height / 100.0, 2)) >= 30 THEN 1
          ELSE 0
        END
      ) AS isObese
    from patients;
    ```
    

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
			{ name: "Santana",},			
	  ]
	},
	{
		name: "Gibson",
	   guitarists: [
				{ name: "Jimmy Page", },
				{ name: "Joe Bonamassa", }
	   ]
	}
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

# Indexing

Индексация json поля

```SQL
CREATE INDEX sample_oligo_task_id_index ON sample USING BTREE (((
    (
      (
        ((sample.meta) :: json -> 'rel' :: text) -> 'oligo_synthesis_task' :: text
      ) ->> 'id' :: text
    )
  ) :: integer));
```

# Examples from wirk

- adding and renaming json-fields
    
    ```SQL
    do
    $$
    DECLARE
      hplc_methods_record record;
      hplc_method jsonb;
      res_methods_arr jsonb = '[]';
    BEGIN
    
        FOR hplc_methods_record in select value from config where config.key = 'oligo.hplc_methods'
        LOOP
        
            FOR hplc_method in select json_array_elements(hplc_methods_record.value::json)
            LOOP
                IF (hplc_method::jsonb->'operation_type_keys' is null) THEN
                    hplc_method = hplc_method || '{"operation_type_keys":["rp_hplc"]}'::jsonb;
                END IF;
                IF (hplc_method::jsonb->'start_ACN' is not null) THEN
                    hplc_method = hplc_method || jsonb_set('{}'::jsonb, '{weight}', hplc_method::jsonb->'start_ACN', true);
                    hplc_method = hplc_method - 'start_ACN';
                END IF;
                res_methods_arr = res_methods_arr || hplc_method;
            END LOOP;
        
            update config set value = res_methods_arr where config.key = 'oligo.hplc_methods';
        
        END LOOP;
    END;
    $$;
    ```