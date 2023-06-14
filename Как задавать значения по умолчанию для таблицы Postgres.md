[[Как создавать таблицы в Postgres]]

В примере задается значение по умолчанию для даты, т.е. дата , при добавлении в таблицу записи, будет добавляться автоматически
```sql
create table disp.driver_sessions (

	id_session serial PRIMARY key,
	object_id int,
	driver_id int,
	av_speed numeric(5,1),
	offset_mess int,
	mess_date timestamp,
	create_date timestamp default current_timestamp,
	update_date timestamp
);
```
