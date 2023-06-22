Библиотека datetime
[[Библиотеки Python]]

[[Datetime#Datetime.Datetime|Datetime.Datetime]]
 - [[Datetime#datetime.strptime()]]
 - [[Datetime#datetime.fromisoformat()]]
 - [[Datetime#datetime.strftime()]]


### Datetime.Datetime

##### datetime.strptime()
Считывание из строкового формата и перевод в формат datetime.datetime
метод ==datetime.datetime.strptime(str, str)==, метод на вход принимает дату в строковом формате и строку с форматом даты
```Python

from datetime import datetime


date: str = "2023-06-22 15:32:20.278"
date_2: datetime = datetime.strptime(date, "%Y-%m-%dT%H:%M:%S.%f")
```

##### datetime.fromisoformat()
Или можно считать строку без указания формата методом ==datetime.datetime.fromisoformat(str)==:
```Python

from datetime import datetime


date: str = "2023-06-22 15:32:20.278"
date_2: datetime = datetime.fromisoformat(date)
```

##### datetime.strftime()
Для перевода из формата ==datetime.datetime== можно использовать метод ==datetime.datetime.strftime("%Y-%m-%d %H:%M:%S.%f")==:
```Python

from datetime import datetime


date: str = "2023-06-22 15:32:20.278"
date_2: datetime = datetime.fromisoformat(date)

date_3: str = date_2.strftime("%Y-%m-%d %H:%M:%S.%f")
```