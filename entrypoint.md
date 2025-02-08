## TODO go
```dataview
TABLE WITHOUT ID
	task.text as Задачи,
	choice(meta(task.section).subpath = "new", "○", "◒") AS Статус
where contains(file.path, "go") and contains(file.path, "planning") 
flatten filter(file.tasks, (t) => meta(t.section).subpath != "completed") as task
sort file.mtime desc

// filter(file.tasks, (t) => t.completed)
```

## Backend
```dataview
TABLE WITHOUT ID
	file.link as Страница
where contains(tags, "backend")
```


## Frontend
```dataview
TABLE WITHOUT ID
	file.link as Страница
where contains(tags, "frontend")
```


