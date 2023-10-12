[[Запросы в PostgreSQL]]

Для просмотра связей между таблицами в SQL можно использовать системную таблицу `INFORMATION_SCHEMA`. `INFORMATION_SCHEMA` содержит метаданные о базе данных, включая информацию о таблицах, столбцах и связях.

Чтобы просмотреть связи между таблицами, можно использовать запрос следующего вида:
```sql
SELECT  
    tc.constraint_name, 
    tc.table_name, 
    kcu.column_name, 
    ccu.table_name AS foreign_table_name,
    ccu.column_name AS foreign_column_name 
FROM 
    information_schema.table_constraints AS tc 
    JOIN information_schema.key_column_usage AS kcu
      ON tc.constraint_name = kcu.constraint_name
      AND tc.table_schema = kcu.table_schema
    JOIN information_schema.constraint_column_usage AS ccu
      ON ccu.constraint_name = tc.constraint_name
      AND ccu.table_schema = tc.table_schema
WHERE tc.constraint_type = 'FOREIGN KEY' AND tc.table_name='имя_таблицы';
```
Здесь имя_таблицы - это имя таблицы, связи которой вы хотите просмотреть.

Этот запрос вернет список всех внешних ключей в таблице, включая имя ключа, имя таблицы, столбец таблицы, связанный с внешним ключом, имя таблицы, связанной с внешним ключом, и соответствующий столбец в этой таблице. Эти данные могут помочь вам понять, как связаны различные таблицы в вашей базе данных.