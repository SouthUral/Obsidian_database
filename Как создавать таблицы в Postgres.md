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
