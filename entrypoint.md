## tg-planner
```dataview
TABLE WITHOUT ID
	task.text as Задачи,
	choice(meta(task.section).subpath = "new", "○", "◒") AS Статус
where contains(file.path, "tg-scraper-bot") and contains(file.path, "planner") 
flatten filter(file.tasks, (t) => meta(t.section).subpath != "completed") as task
sort file.mtime desc
limit 10
```

## go
```dataview
TABLE WITHOUT ID
	task.text as Задачи,
	choice(meta(task.section).subpath = "new", "○", "◒") AS Статус
where contains(file.path, "go") and contains(file.path, "planning") 
flatten filter(file.tasks, (t) => meta(t.section).subpath != "completed") as task
sort file.mtime desc
limit 10

// filter(file.tasks, (t) => t.completed)
```

## [[vault planner]]
```dataview
TABLE WITHOUT ID
	task.text as Задачи,
	choice(meta(task.section).subpath = "new", "○", "◒") AS Статус
where contains(file.path, "vault") and contains(file.path, "planner") 
flatten filter(file.tasks, (t) => meta(t.section).subpath != "completed") as task
sort file.mtime desc
limit 10
```

## Orbiter
```dataview
TABLE WITHOUT ID
	task.text as Задачи,
	choice(meta(task.section).subpath = "new", "○", "◒") AS Статус
where contains(file.path, "orbiter") and contains(file.path, "planner") 
flatten filter(file.tasks, (t) => meta(t.section).subpath != "completed") as task
sort file.mtime desc
limit 10
```

## Backend entrypoints
```dataview
TABLE WITHOUT ID
	file.link as Страница
where contains(tags, "backend")
```


## Frontend entrypoints
```dataview
TABLE WITHOUT ID
	file.link as Страница
where contains(tags, "frontend")
```


