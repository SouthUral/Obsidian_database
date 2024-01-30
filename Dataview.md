# Рабочие проекты
___
```dataview
TABLE WITHOUT ID
	file.link AS "Проект",
	Статус,
	(length(filter(file.tasks.completed, (t) => t = true))) / (length(file.tasks.text)) * 100 AS "% Завершено",
	(length(file.tasks.text)) AS "Всего",
	(length(filter(file.tasks.completed, (t) => t = true))) AS Завершённых,
	(length(file.tasks.text)) - (length(filter(file.tasks.completed, (t) => t = true))) AS "Незавршенных"
FROM #work
```


```dataview
TASK
WHERE completion = date("2024-01-30")
```


```dataview
TASK
WHERE start = date("2024-01-30") 
```
