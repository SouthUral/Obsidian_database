[[PostgreSQL]]

Пример создания простой функции:
```postgresql

CREATE OR REPLACE FUNCTION my_function(param1 INTEGER, param2 VARCHAR) RETURNS INTEGER AS $$
BEGIN
	RETURN param1 + LENGTH(param2);
END;
$$ LANGUAGE plpgsql;

```
эта функция принимает два параметра, и возвращает тип ==INTEGER==
Тело функции начинается после ключевого слова ==AS== 
Функцию можно вызвать так:
```PostgreSQL

SELECT my_function(42, 'hello');
```

Пример из практики, функция возвращает результат запроса (таблицу):
```PostgreSQL

CREATE OR REPLACE FUNCTION sh_disp.get_active_session() 
RETURNS TABLE (
	object_id integer,
	driver_id integer,
	av_speed numeric(5,1),
	offset_mess integer,
	mess_date timestamp,
	id_session integer
) AS $$
BEGIN
	RETURN QUERY 
	SELECT DISTINCT ON (object_id)
		object_id,
		driver_id,
		av_speed,
		offset_mess,
		mess_date,
		id_session
	FROM sh_disp.driver_sessions
	ORDER BY object_id, mess_date DESC;
END;
$$ LANGUAGE plpgsql;
```

Еще один пример функции с параметрами, из практики:
```PostgreSQL
CREATE OR REPLACE FUNCTION sh_disp.add_session(
	p_object_id integer,
	p_driver_id integer,
	p_av_speed numeric(5,1),
	p_offset_mess integer,
	p_mess_date timestamp
)
RETURNS TABLE (
	id_session integer
)
AS $$
BEGIN
	INSERT INTO sh_disp.driver_sessions
	(object_id, driver_id, av_speed, offset_mess, mess_date, update_date)
	VALUES (p_object_id, p_driver_id, p_av_speed, p_offset_mess, p_mess_date, current_timestamp)
	RETURNING id_session INTO result;
	RETURN QUERY SELECT result;
END;
$$ LANGUAGE plpgsql;
```

Пример вызова функции:
```PostgreSQL
SELECT * FROM sh_disp.add_session(1, 2, 3.14, 4, '2022-01-01 12:00:00');
```
Этот запрос добавит новую запись в таблицу ==sh_disp.driver_sessions== с указанными значениями ==object_id==, ==driver_id==, ==av_speed==, ==offset_mess== и ==mess_date==, а затем вернет таблицу с результатом запроса, содержащую столбец ==id_session==.