[[PostgreSQL]]
[[Rabbit MQ]]

[ссылка на статью](https://habr.com/ru/articles/578744/)
простой пример:
```yaml
version: "3.9"

services: 
 postgres:
 image: postgres:13.3 
 environment:
  POSTGRES_DB: "habrdb"
  POSTGRES_USER: "habrpguser"
  POSTGRES_PASSWORD: "pgpwd4habr"
 volumes:
  - .:/docker-entrypoint-initdb.d
 ports:
  - "5432:5432"
```

Любые SQL скрипты находящиеся в `docker-entrypoint-initdb.d` будут рассматриваться как скрипты для инициализации БД.

Для сохранения данных нужно использовать `volume`, так же нужно определить переменную окружения в контейнере `PGDATA` в которой нужно указать, где будут храниться данные.
```yaml
version: "3.9"

services: 
 postgres:
 image: postgres:13.3 
 environment:
  POSTGRES_DB: "examplebase"
  POSTGRES_USER: "user"
  POSTGRES_PASSWORD: "passworduser"
  PGDATA: "/var/lib/postgresql/data/pgdata"
 volumes:
  - .:/docker-entrypoint-initdb.d
  - pgdata:/var/lib/postgresql/data
 ports:
  - "5432:5432"

volumes:
 pgdata:
```

Так же можно задать свою конфигурацию, для этого понадобиться примонтировать еще один файл в контейнер.
```yaml
volumes:
 - ./postgresql.conf:/var/lib/postgresql/data/postgresql.conf
```