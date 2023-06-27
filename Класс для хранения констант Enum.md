[[Полезные фишки с ООП Python]]

Класс ==Enum== можно использовать для хранения неизменяемых констант.
Пример из практики:
```Python

from enum import Enum


class ConstantQuery(Enum):
	QUERY_FOR_DB1: str
	QUERY_FOR_DB2: str


class GetDataLoaded(ConstantQuery):
	QUERY_FOR_DB1 = "select * from sh_ilo.get_tracks_trip_routes_loaded(%(created_id)s)"
	QUERY_FOR_DB2 = "select messages from device.get_tracks_trip_routes_loaded(%(object_id)s, %(load_created_id)s, %(unl_created_id)s) as messages"


class GetRoutesEmpty(ConstantQuery):
	QUERY_FOR_DB1 = "select * from sh_ilo.get_tracks_trip_routes_empty(%(created_id)s)"
	QUERY_FOR_DB2 = "select messages from device.get_tracks_trip_routes_empty(%(object_id)s, %(unl_created_id)s, %(load_created_id)s) as messages"


def some_func(arg: ConstantQuery):
	print(arg.QUERY_FOR_DB1.name)

some_func(GetDataLoaded)
```

Без вызова метода ==name== возвращается просто название атрибута класса.

>[!info] Для чего класс Enum
>Класс `Enum` в Python предназначен для определения перечислений (enumerations) - набора именованных констант, которые могут быть использованы для представления определенных значений.
>Использование класса `Enum` может быть полезно в различных ситуациях, например, когда необходимо определить набор значений, которые могут быть использованы в коде, или когда необходимо определить перечисление для определенной функциональности. Класс `Enum` также может быть использован для создания пользовательских перечислений, которые могут быть использованы в вашем коде.

Простой пример:
```Python

from enum import Enum


class Color(Enum):
	RED = 1
	GREEN = 2 
	BLUE = 3 

print(Color.RED.value) # выводит 1 
print(Color.GREEN.value) # выводит 2 
print(Color.BLUE.value) # выводит 3
```