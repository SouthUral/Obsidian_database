[[Стандартная библиотека Golang]]

Модуль ==net/http== является модулем стандартной библиотеки Golang.
Он имеет готовый к работе веб-сервер и простой http-клиент для отправки запросов.

Пример запуска веб-сервера на Golang:
```Go
package main

import (
	"fmt"
	"net/http"
)

const portNumber = ":8080"

func HomePageHandler(w http.ResponseWriter, r *http.Request) {
	w.Write([]bite("We're live !"))
}

func main() {
	http.HandleFunc("/", HomePageHandler)
	fmt.Printf("Starting application on port %v\n", portNumber)
	http.ListenAndServe(portNumber, nil)
}
```

Другой способ написать обработчик:
```Go
func HomePageHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "We're live!")
}
```
==`Fprintf` принимает любой тип, реализующий интерфейс `io.Writer`==

Пример `http` клиента для отправки запросов на сервер:
```Go
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
)

func main() {
	req, err := http.Get("https://api.github.com/users")
	if err != nil {
		log.Fatal(err)
	}
	defer req.Body.Close()
	data, err := ioutil.ReadAll(req.Body)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(string(data))
}
```