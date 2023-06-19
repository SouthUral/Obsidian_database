[[Запросы в PostgreSQL]]

Для того чтобы запрос вернул значение нужно использовать такой синтаксис:
```sql
INSERT INTO disp.driver_sessions (object_id, driver_id, av_speed, offset_mess, mess_date, update_date)

VALUES (%s, %s, %s, %s, %s, current_timestamp)

RETURNING id_session
```