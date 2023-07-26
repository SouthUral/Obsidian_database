[[Логирование в golang]]

простое использование пакета:
```go
package main

import (
	log "github.com/sirupsen/logrus"
)

func init() {
	// логи в формате JSON, по умолчанию формат ASCII
	log.SetFormatter(&log.JSONFormatter{})

	// логи идут на стандартный вывод, их можно перенаправить в файл
	log.SetOutput(os.Stdout)
	
	// установка уровня логирования 
	log.SetLevel(log.WarnLevel)
}

func main() {
	log.Info("app is starting")
}
```

Так же можно использовать разные уровни логгирования в разных частях программы, для этого можно создать экземпляр `logger`
```go
package main

import (
  "os"
  "github.com/sirupsen/logrus"
)

// Create a new instance of the logger. You can have any number of instances.
var log = logrus.New()

func main() {
  // The API for setting attributes is a little different than the package level
  // exported logger. See Godoc.
  log.Out = os.Stdout

  // You could set this to any `io.Writer` such as a file
  // file, err := os.OpenFile("logrus.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
  // if err == nil {
  //  log.Out = file
  // } else {
  //  log.Info("Failed to log to file, using default stderr")
  // }

  log.WithFields(logrus.Fields{
    "animal": "walrus",
    "size":   10,
  }).Info("A group of walrus emerges from the ocean")
}
```

Логрус поощряет тщательную, структурированную регистрацию через поля регистрации вместо длинных, неразборчивых сообщений об ошибках. Например, вместо: `log.Fatalf("Failed to send event %s to topic %s with key %d")`вы должны регистрировать гораздо более поддающийся обнаружению:
```GO
log.WithFields(log.Fields{
  "event": event,
  "topic": topic,
  "key": key,
}).Fatal("Failed to send event")
```