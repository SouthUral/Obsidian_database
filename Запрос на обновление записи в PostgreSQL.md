[[Запросы в PostgreSQL]]

Пример синтаксиса:
```sql
UPDATE disp.driver_sessions

SET av_speed = %s, offset_mess = %s, mess_date = %s, update_date = current_timestamp

WHERE id_session = %s
```