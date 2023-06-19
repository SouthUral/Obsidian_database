[[Python]]
[[Библиотеки Python]]

[[type hinting python#Простая аннотация типов|Простая аннотация типов]]
[[type hinting python#Модуль typing|Модуль typing]]


>[!note] Запись
>
_Аннотации типов не обеспечивают проверку типов на уровне интерпретатора. Они предназначены для использования разработчиками, средами разработки, линтерами, программами проверки типов и т. д._

## Простая аннотация типов
```python
data: str = "data"
data_2: int = 2
data_l: list[str] = ['1', '2']
```

Аннотация типов в функции:
```python
def function(param: str) -> list[str]:
	return list(param)

def indent_right(s: str, width: int) -> str: 
	return " " * (max(0, width - len(s))) + s
```

Аннотации в классе:
```python
class Book:
	title: str 
	author: str 
	def __init__(self, title: str, author: str) -> None:
		self.title = title
		self.author = author


b: Book = Book(title='Fahrenheit 451', author='Bradbury')
```

Когда может быть сразу несколько типов:
```python
data = str | None
```

## Модуль typing

### Any
Переменная типа `Any` является неограниченной. Поэтому такие переменные совместимы со всеми другими типами (`int`, `str`, `List` и т. д.), а все остальные типы совместимы с ними.
```python
from typing import Any 

result: Any = "SUCCESS" 
# также работает, потому что переменные типа Any совместимы с другими типами 
result = 10 

state: str = "PENDING" 
# работает, потому что все остальные типы совместимы с типом Any. state = result
```

### Literal

Литералы используются для указания программам проверки типов, что значение, которое имеет переменная или функция, равно одному из указанных значений.

```python
from typing import Literal

GENDER = Literal["male", "female", "non–conforming"] 

def create_user(first_name: str, last_name: str, gender: GENDER) –> dict[str, str, GENDER]:
	return {"first_name": first_name, "last_name": last_name, "gender": gender}
	
create_user("John", "Bond", "male")
```