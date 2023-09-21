[[docker-compose]]

Команда для сборки и запуска сервиса не затрагивая остальные сервисы в docker-compose.yaml
```shell
docker-compose up --no-deps --build my_service
```
