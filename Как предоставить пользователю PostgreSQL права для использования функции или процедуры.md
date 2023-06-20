[[Создание процедур в PostgreSQL]]
[[Создание функций в PostgreSQL]]
[[Назначение владельца на функцию или таблицу PostgreSQL]]

Пример:
```PostgreSQL
GRANT EXECUTE ON функция_имя() TO имя_пользователя;
```

Ещё пример:
```PostgreSQL

GRANT EXECUTE ON FUNCTION sh_log.get_history(integer, varchar, varchar, integer) TO viewport;
```

Примеры из практики:
```PostgreSQL
GRANT EXECUTE ON FUNCTION sh_disp.get_active_session() TO service_dispatcher;

GRANT EXECUTE ON FUNCTION sh_disp.add_session(integer, integer, numeric, integer, timestamp) TO service_dispatcher;

GRANT EXECUTE ON PROCEDURE sh_disp.update_session(numeric, integer, timestamp, integer) TO service_dispatcher;

```