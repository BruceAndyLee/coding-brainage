## [[Coding brainage/tg-scraper-bot/planner|tg-scraper]]
```dataview
TABLE WITHOUT ID
	task.text as Задачи,
	choice(meta(task.section).subpath = "new", "○", "◒") AS Статус
where contains(file.path, "tg-scraper-bot") and contains(file.path, "planner") 
flatten filter(file.tasks, (t) => meta(t.section).subpath != "completed") as task
sort file.mtime desc
limit 10
```

## [[go/planner|go]]
```dataview
TABLE WITHOUT ID
	task.text as Задачи,
	choice(meta(task.section).subpath = "new", "○", "◒") AS Статус
where contains(file.path, "go") and contains(file.path, "planner") 
flatten filter(file.tasks, (t) => meta(t.section).subpath != "completed") as task
sort file.mtime desc
limit 10

// filter(file.tasks, (t) => t.completed)
```

## [[Coding brainage/butlerov/planner|butlerov]]
```dataview
TABLE WITHOUT ID
	task.text as Задачи,
	choice(meta(task.section).subpath = "new", "○", "◒") AS Статус
where contains(file.path, "butlerov") and contains(file.path, "planner") 
flatten filter(file.tasks, (t) => meta(t.section).subpath != "completed") as task
sort file.mtime desc
limit 10
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


