[[PostgreSQL]]

Пример создание процедуры в PostgreSQL псевдокодом:
```PostgreSQL

CREATE OR REPLACE PROCEDURE имя_процедуры(
	параметр1 тип_параметра_1,
	параметр2 тип_параметра_2
)
LANGUAGE язык_процедуры
AS $$
BEGIN
	выражение для изменения БД;
END;
$$;
```

Проверить, что процедура была создана, можно с помощью команды:
```psql

\dp имя_процедуры
```

Пример создание процедуры из практики:
```PostgreSQL
CREATE OR REPLACE PROCEDURE sh_disp.update_session(
	p_av_speed numeric(5,1),
	p_offset_mess integer,
	p_mess_date timestamp,
	p_id_session integer
)
LANGUAGE plpgsql
AS $$
BEGIN
	UPDATE sh_disp.driver_sessions
	SET av_speed = p_av_speed,
		offset_mess = p_offset_mess,
		mess_date = p_mess_date,
		update_date = current_timestamp
	WHERE id_session = p_id_session;
END;
$$;
```

Вызвать процедуру можно так:
```PostgreSQL
CALL имя_процедуры(значение_параметра1, значение_параметра2,...);
```