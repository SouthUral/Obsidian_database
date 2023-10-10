[[Работа с БД в Golang]]
[[Rabbit MQ]]

### Простое подключение
[Документация простого консюмера и паблишера](https://www.rabbitmq.com/tutorials/tutorial-one-go.html)

```go
import (
	"context"

	amqp "github.com/rabbitmq/amqp091-go"

	log "github.com/sirupsen/logrus"
)
```

#### Создание коннекта
```go
func createConnRabbit(URL string) (*amqp.Connection, error) {
	сonn, err := amqp.Dial(URL)
	if err != nil {
		log.Error(err.Error())
	}
	return сonn, err
}
```

#### Создание канала
```Go
func createChannRabbit(conn *amqp.Connection) (*amqp.Channel, error) {
	rabbitChan, err := conn.Channel()
	if err != nil {
		log.Error(err.Error())
	}
	return rabbitChan, err
}
```

#### Декларирование Exchange
```go
// Метод для создания Exchange
func declaringExchange(rabbitChan *amqp.Channel, nameExchange string) error {
	err := rabbitChan.ExchangeDeclare(
		nameExchange, // имя exchange
		"direct", // тип exchange
		true, // durable - сохранится ли exchange после перезагрузки Rabbit
		false, // auto delete - удалить если не используется
		false, // internal
		false, // noWait
		nil, // дополнительные аргументы
	)

	if err != nil {
		log.Errorf("exchange declare failed: %v\n", err)
	}
	return err
}
```

#### Декларирование очереди (простой пример)
При объявлении очереди, она будет создана только если она еще не была создана.
```Go
func declaringQueue(rabbitChan *amqp.Channel, nameQueue string) (amqp.Queue, error) {
	queue, err := rabbitChan.QueueDeclare(
		nameQueue, // name
		false, // durable
		false, // autoDelete - удалить если не используется
		false, // exclusive
		false, // noWait
		nil, // args
)
	if err != nil {
		log.Errorf("Ошибка создания очереди: %s", err.Error())
	}
	return queue, err
}
```

#### Binding очереди к exchange
```go
// Метод для bindig очереди к exchange
func bindingQueue(rabbitChan *amqp.Channel, nameQueue, nameExchange, routingKey string) error {
	err := rabbitChan.QueueBind(
		nameQueue, // queue name
		routingKey, // routing key
		nameExchange, // name exchange
		false, // noWait
		nil,
	)
	if err != nil {
			log.Errorf("Ошибка binding очереди %s, к exchange %s : %s", nameQueue, nameExchange, err.Error())
	}
	return err
}
```

#### Отправка сообщения (простой пример)
```Go
// Метод для отправки сообщений
func sendingMess(queue amqp.Queue, chanRabbit *amqp.Channel, context context.Context, exchange, mess string) error {
	err := chanRabbit.PublishWithContext(
		context,    // контекст
		exchange,   // exchange
		queue.Name, // routing key
		false,      // mandatory
		false,      // immediate
		amqp.Publishing{
			ContentType: "text/plain",
			Body: []byte(mess),
	})
	
	if err != nil {
		log.Errorf(log.ErrorKey)
	} else {
		log.Infof("the message: {%s} has been sent", mess)
	}
	return err
}
```

### Publisher (простой пример)
```Go
func Publisher(URL string, nameQueue string) {
	defer exitPublisher()
	
	conn, err := createConnRabbit(URL)
	if err != nil {
		return
	}

	chanRb, errChan := createChannRabbit(conn)
	if errChan != nil {
		return
	}

	queue, errQueue := declaringQueue(chanRb, nameQueue)
	if errQueue != nil {
		return
	}

	context, _ := context.WithTimeout(context.Background(), 5*time.Second)

	for counter := 0; counter < 10; {
		mess := fmt.Sprintf("Message № %s", counter)
		err := sendingMess(queue, chanRb, context, mess)
		if err != nil {
			return
		}
		counter++
	}
}

func exitPublisher() {
	log.Info("The publisher was closed")
}
```

==`context.WithTimeout(context.Background(), 5*time.Second)` этот контекст нужен для закрытия коннекта при большом ожидании (но это не точно)==
### Простой Consumer
```Go
func Consumer(URL string, nameQueue string, consumerName string) {
	conn, err := createConnRabbit(URL)
	if err != nil {
		return
	}

	chanRb, errChan := createChannRabbit(conn)
	if errChan != nil {
		return
	}

	// Qos определяет, сколько сообщений или сколько байт сервер попытается
	// сохранить в сети для потребителей,
	// прежде чем получит подтверждение доставки.
	errQos := chanRb.Qos(1, 0, false)
	if errQos != nil {
		log.Errorf("Qos Rabbit failed: %v\n", err)
	}

	queue, errQueue := declaringQueue(chanRb, nameQueue)
	if errQueue != nil {
		return
	}

	msgs, errConsume := chanRb.Consume(
		queue.Name,
		consumerName,
		false,
		false,
		false,
		false,
		nil,
	)

	if errConsume != nil {
		log.Error("Failed to register a consumer", errConsume)
		return
	}

	for msg := range msgs {
		log.Infof("Received a message: %s", msg.Body)
		msg.Ack(true)
	}
	defer exitAndClose(chanRb, conn, "Consumer")
}

// это вспомогательная функция, для закрытия канала и коннекта с Rabbit
func exitAndClose(chanRb *amqp.Channel, connRb *amqp.Connection, whoIsIt string) {
	errChan := chanRb.Close()
	if errChan != nil {
		log.Error("channel closing error")
	}

	errConn := connRb.Close()
	if errConn != nil {
		log.Error("connection closing error")
	}

	log.Warningf("The %s was closed", whoIsIt)
}
```