[[Создание функций в PostgreSQL]]
[[Как создавать таблицы в Postgres]]

Пример синтаксиса:
```postgreSQL

CREATE TABLE sh_disp.driver_sessions (
	id_session serial PRIMARY key,
	object_id int,
	driver_id int,
	av_speed numeric(5,1),
	offset_mess int,
	mess_date timestamp,
	create_date timestamp default current_timestamp,
	update_date timestamp
);

ALTER TABLE sh_disp.driver_sessions OWNER TO executor;
```

Изменить владельца функции с помощью команды:
```PostgreSQL

ALTER FUNCTION функция_имя() OWNER TO новый_владелец;
```
