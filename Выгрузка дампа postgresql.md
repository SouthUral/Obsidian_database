[[PostgreSQL]]

Бэкап кластера:

```shell
pg_dumpall -U postgres --globals-only -E UTF8 --clean > dump
```

Бэкап структуры БД:

```shell
pg_dump -U postgres -E UTF8 --clean --create --schema-only -Fc poly_bgp > dump.dump 
```

==----create== - создает БД и подключается к ней
==--schema-only== - Выгрузка только схемы объектов без данных.

Бэкап данных БД:

```shell
pg_dump -h spbsrvasdpg3 -p 5432 -U postgres -E UTF8 --clean --create -v -Fc poly_bgp > dump2/poly_bgp_rc664_2023-04-17_2.dump
```

Загрузка дампа в БД:
```shell
pg_restore -p 5433 --create -d postgres -v -Fc poly_bgp-data
```