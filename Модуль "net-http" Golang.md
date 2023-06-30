[[Стандартная библиотека Golang]]
[[Разработка Web приложения на Golang]]

## Основы

Модуль ==net/http== является модулем стандартной библиотеки Golang.
Он имеет готовый к работе веб-сервер и простой http-клиент для отправки запросов.

Пример запуска веб-сервера на Golang:

```Go
package main

import (
	"fmt"
	"net/http"
	"log"
)

const portNumber = ":8080"

func HomePageHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hi there, I love %s!", r.URL.Path[1:])
}

func main() {
	http.HandleFunc("/", HomePageHandler)
	fmt.Printf("Starting application on port %v\n", portNumber)
	log.Fatal(http.ListenAndServe(portNumber, nil))
}
```
Функция `main` начинается с вызова `http.HandleFunc` , который сообщает пакету `http` обрабатывать все корневые веб запросы ("/") с помощью `handler`

Затем он вызывает `http.ListenAndServe`, указывая, что он должен прослушивать порт 8080 на любом интерфейсе (`":8080"`). (Не беспокойтесь о втором параметре, `nil`, пока.) Эта функция будет блокироваться до завершения программы.

`ListenAndServe` всегда возвращает ошибку, поскольку она возвращается только тогда, когда случилась неожиданная ошибка. Чтобы записать эту ошибку в лог, мы заключаем вызов функции в `log.Fatal`.

Функция ==`handler`== имеет тип ==`http.HandlerFunc`==. Он принимает ==`http.ResponseWriter`== и ==`http.Request`== как его аргументы.

Значение ==`http.ResponseWriter`== собирает ответ HTTP-сервера; написав в него, мы отправляем данные HTTP-клиенту.

`http.Request` - это структура данных, которая представляет клиентский HTTP-запрос. `r.URL.Path` является компонентом пути URL запроса. Конечный `[1:]` означает "создать под-срез `Path` от 1-го символа до конца." Это удаляет ведущий "/" из имени пути.

Если вы запустите эту программу и обратитесь к URL:

```
http://localhost:8080/monkeys
```

программа представит страницу, содержащую:

```
Hi there, I love monkeys!
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

---

## Сервер

### Простой роутинг

Пример для простых приложений:
```go
package main

import (
	"net/http"
	"log"
)

type MyHandler struct {
}

func (MyHandler) ServerHTTP(w http.ResponseWriter, r *http.Request) {
	if r.URL.Path == "/" {
		w.Write([]byte("Home"))
		return
	}

	if r.URL.Path == "/hello" {
		w.Write([]byte("Hello, user"))
		return
	}

	w.WriteHeader(http.StatusNotFound)
	w.Write([]byte("404 Page Not Found"))

}

func main() {
	err := http.ListenAndServe(":3000", MyHandler{})
	if err != nil {
		log.Fatal(err)
	}
}

```

### Стандартный роутер

Для переключения между различными хендлерами используется мультиплексор (роутер во всех остальных языках). 
```go
func main() {
	// определяем мультиплексор
	mux := http.NewServeMux()
	// Добавляем в него обработчики(хендлеры)
	mux.HandleFunc("/", index)
	mux.HandleFunc("/", index2)

	// mux реализует интерфейс хендлера, поэтому передаем ее в функцию
	http.ListenAndServe(":3000", mux)
	
}
```
>[!info] Замечание
>если в проекте используется один роутер то можно обойтись без инициализации `mux := http.NewServeMux()`, в этом случае будет задействован мультиплексор по умолчанию `DefaultServeMux`
```go
func main() {
	http.HandleFunc("/", index)
	http.HandleFunc("/hello/", hello)

	http.ListenAndServe(":3000", nil)
}
```

### Обработка 404

В самом обработчике можно сделать проверку на правильность пути
```go

func hello (w http.ResponseWriter, r *http.Request) {
	if r.URL.Path != "/hello/" {
		handler404(w, r)
	}
	w.Write([]byte("hello"))
}
```

Либо можно сделать обработку с помощью регулярных выражений

```go
func index(w http.ResponseWriter, r *http.Request) {
	pathRegexp := regexp.MustCompile(`^/\w+$`)
	if !pathRegexp.Match([]byte(r.URL.Path)) {
		handler404(w, r)
		return	
	}
}
```