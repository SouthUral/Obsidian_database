---
cssclasses:
  - dashboard
banner: "![[premium_photo-1672115680863-9353a690495a.png]]"
banner_y: 0.484
---
# OverdueTasks
Просроченные задачи
```tasks
due yesterday
```

# AllTasksToday


## Открытые задачи на день
```tasks
due today
not done
has tags
short mode
```
## Задачи в работе
```tasks
due today
filter by function task.status.type === "IN_PROGRESS"
short mode
```
## Завершенные задачи
```tasks
due today
done
```
## Открытые задачи неделю
```tasks
has start date
```

# Work
[[Рабочие задачи]]


## WorkProjects
```dataview
TABLE WITHOUT ID
	file.link AS "Рабочие проекты",
	Статус,
	round((length(filter(file.tasks.completed, (t) => t = true))) / (length(file.tasks.text)) * 100, 2) AS "% Завершено",
	(length(file.tasks.text)) AS "Всего",
	(length(filter(file.tasks.completed, (t) => t = true))) AS Завершённых,
	(length(file.tasks.text)) - (length(filter(file.tasks.completed, (t) => t = true))) AS "Незавршенных"
FROM #work
```

# PetProjects



# DailyTasks



# EducationalTasks
[[Учебные задачи]]