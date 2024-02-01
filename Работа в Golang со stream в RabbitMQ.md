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
// вернет ошибку если нет сообщений в стриме
lastOffset, err := stats.LastOffset()

// закомиченный оффсет, пока не ясно что это такое!!!
committedChunkId, err := statsAfter.CommittedChunkId()
```

## Публикация сообщений в stream
Определяем отправителя:
```go
producer, err :=  env.NewProducer("my-stream", nil)
```

Можно сделать подробное описание отправителя:
```go
type ProducerOptions struct {
	Name       string // имя отправителя
	QueueSize  int // размер внутренней очереди
	BatchSize  int // размер пакета
	BatchPublishingDelay int    // Period to send a batch of messages.
}
```

Есть два варианта отправки сообщений:
```Go
// первый вариант
// отправка одного сообщения
// этот метод медленее но используется чаще и в целом он предпочтительнее
var message message.StreamMessage
message = amqp.NewMessage([]byte("hello"))
err = producer.Send(message)

// второй вариант
// отправка пакета сообщений
// отправка производится быстрее
var messages []message.StreamMessage
for z := 0; z < 10; z++ {
  messages = append(messages, amqp.NewMessage([]byte("hello")))
}
err = producer.BatchSend(messages)
```

для каждого отправления сервер предоставляет отчет, получил ли он сообщение.
```go
chPublishConfirm := producer.NotifyPublishConfirmation()
handlePublishConfirm(chPublishConfirm)

func handlePublishConfirm(confirms stream.ChannelPublishConfirm) {
	go func() {
		for confirmed := range confirms {
			for _, msg := range confirmed {
				if msg.IsConfirmed() {
					fmt.Printf("message %s stored \n  ", msg.GetMessage().GetData())
				} else {
					fmt.Printf("message %s failed \n  ", msg.GetMessage().GetData())
				}
			}
		}
	}()
}
```
## Закрытие отправителя
```go
producer.Close()
```
 если другого отправителя не будет на этом коннекте то коннект закроется. (надо проверить)

## Consumer
```go

handleMessages := func(consumerContext stream.ConsumerContext, message *amqp.Message) {
		if atomic.AddInt32(&count, 1)%1000 == 0 {
			fmt.Printf("cousumed %d  messages \n", atomic.LoadInt32(&count))
			// AVOID to store for each single message, it will reduce the performances
			// The server keeps the consume tracking using the consumer name
			err := consumerContext.Consumer.StoreOffset()
			if err != nil {
				CheckErr(err)
			}
		}

	}

consumer, err := env.NewConsumer(
		streamName,
		handleMessages,
		stream.NewConsumerOptions().
			SetConsumerName("my_consumer").                  // set a consumer name
			SetOffset(stream.OffsetSpecification{}.First())) 
)
```
 