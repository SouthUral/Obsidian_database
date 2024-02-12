[[Модуль "net-http" Golang]]


Работающий простой сервер с настроенным логированием:
```go
package main

import (
	"net/http"
	"time"
	
	log "github.com/sirupsen/logrus"
)

func init() {
	log.SetFormatter(&log.JSONFormatter{})
}

func main() {
	initServer()
	time.Sleep(120 * time.Second)
}

type Server struct {
	Port string
}

func initServer() {
	srv := Server{
		Port: ":3000",
	}
	go srv.startServer()
}

func (srv *Server) startServer() {
	http.HandleFunc("/", midlware(srv.GetAll))
	http.ListenAndServe(srv.Port, nil)
}

func (srv *Server) GetAll(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Server is working"))
}

func midlware(handler http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		log.Info(r.Method)
		handler(w, r)
	}
}
```

