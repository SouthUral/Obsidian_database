[[Запросы в PostgreSQL]]


Сначала указывается схема, потом таблица и в скобках ее колонки с типами
```sql
create table disp.av_speed_data(

	id serial PRIMARY key,
	id_offset int,
	object_id int,
	av_speed numeric(5,1),
	mess_date timestamp);
```

```SQL
create table device.messages(
	id serial primary key,
	id_offset int,
	message text,
	mess_date timestamp default current_timestamp
)
```