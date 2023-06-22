[[Работа с PostgreSQL через Python]]

psycopg2 - это синхронная библиотека Python для работы с PostgreSQL

## Установка

```bash
pip3 install psycopg2-binary
```

## Настройка подключения

Обычное подключение, примерная реализация.
```Python

import logging
import psycopg2
from psycopg2.extras import DictCursor


logging.basicConfig(filename=file_log, encoding='utf-8', level=logging.DEBUG, format='%(asctime)s %(message)s')


class Postgres:
	def __init__(self, dbname: str, user: str, password: str, host: str, port: int):
	self.dbname = dbname
	self.user = user
	self.password = password
	self.host = host
	self.port = port
	self.conn = self.connect()
	self.application_name = 'rec_session'

	def connect(self):
		try:
			connect = psycopg2.connect(**self.__dict__, cursor_factory=DictCursor)
			return connect
		except Exception as err:
			logging.error('Error: %s', err)

```
Но при такой реализации нужно будет закрывать connect

Говорят, что лучше не создавать курсор отдельно а открывать его по мере необходимости, пример из практики.
```Python
def request_one(self, query_txt: str):
	try:
		with self.conn.cursor() as curs:
			curs.execute(query_txt)
			data = curs.fetchone()
			return data[0]
		except psycopg2.OperationalError as err:
			if self.conn:
				self.conn.close()
			log_error(f"postgres.PostgresMain.request_one: {err}", exit=True)
		except:
			return None
```