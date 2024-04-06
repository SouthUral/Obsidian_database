
Запуск контейнера:
```shell
docker run 
```

Просмотр всех сетей в docker
```shell
docker network ls 
```

Просмотр информации по конкретной сети 
```shell
docker network inspect name-network
```

Команда очищающая все не связанные с контейнером ресурсы, в т.ч. образы, контейнеры, тома, сети.
```shell
docker system prune
```

Удаление всех остановленных контейнеров и неиспользуемых образов:
```shell
docker system prune -a
```

Вывести список образов:
```shell
docker images -a
```

Удаление образа:
```shell
docker rmi Image
```

Список томов:
```shell
docker volume ls
```

Удаление всех томов (наверное, я не знаю точно, но все тома удалились):
```shell
docker volume prune -a
```