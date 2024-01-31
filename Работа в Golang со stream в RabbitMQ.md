[[RabbitMQ streams]]
[[Работа с RabbitMQ через golang]]

[страница библиотеки](https://github.com/rabbitmq/rabbitmq-stream-go-client)

Установка:
```shell
go get -u github.com/rabbitmq/rabbitmq-stream-go-client
```

Локальный запуск RabbitMQ в Docker со включенным плагином stream
```shell
docker run -it --rm --name rabbitmq -p 5552:5552 -p 15672:15672\
    -e RABBITMQ_SERVER_ADDITIONAL_ERL_ARGS='-rabbitmq_stream advertised_host localhost -rabbit loopback_users "none"' \
    rabbitmq:3.9-management
```

После запуска RabbitMQ, в контейнере нужно выполнить команду:
```shell
docker exec rabbitmq rabbitmq-plugins enable rabbitmq_stream_management
```

## Подключение
Стандартное подключение:
```go
env, err := stream.NewEnvironment(
		stream.NewEnvironmentOptions().
			SetHost("localhost").
			SetPort(5552).
			SetUser("guest").
			SetPassword("guest"))
```

Можно указать максимальное количество consumers, по умолчанию стоит 1. Лучше использовать значение по умолчанию.
```go
stream.NewEnvironmentOptions().
SetMaxProducersPerClient(2))
```

## Создание stream
```go
err = env.DeclareStream(streamName,
		stream.NewStreamOptions().
		SetMaxLengthBytes(stream.ByteCapacity{}.GB(2)))
```

## Получение информации о stream
Для получение статистики по __stream__ нужно использовать __environment.StreamStats__:
```go
stats, err := environment.StreamStats(testStreamName)

// метод для получения первого оффсета из stream
// вернет ошибку если первого оффсета нет (стрим пустой)
firstOffset, err := stats.FirstOffset()

// метод для получения последнего оффсета из стрима
// вернет
lastOffset, err := stats.LastOffset()
```