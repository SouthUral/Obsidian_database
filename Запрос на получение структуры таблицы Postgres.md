[[Запросы в PostgreSQL]]

Для получения структуры таблицы можно сделать запрос
```sql
SELECT column_name, column_default, data_type 
FROM INFORMATION_SCHEMA.COLUMNS 
WHERE table_name = 'super_table';
```

Пример:
```sql
SELECT column_name, column_default, data_type
FROM INFORMATION_SCHEMA.COLUMNS
WHERE table_name = 'registrations_drivers';
```

Результат:
![[Pasted image 20230815150722.png]]