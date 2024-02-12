[статья про сервер на go](https://grafana.com/blog/2024/02/09/how-i-write-http-services-in-go-after-13-years/)

# Описание

Статья содержит лучшие современные практики для реализации http сервера на golang

# Конструктор NewServer
Пример кода:
```go

// функция является конструктором обработчика для сервера
func NewServer(
	logger *Logger // указатель на объект логгера
	config *Config // указатель на объект содержащий конфиг
	commentStore *CommentStore // указатель на объект, который хранит комментарии
	anotherStore *AnotherStore
) http.Handler {
	// создается объект ServeMux который используется для маршрутизации запросов
	mux := http.NewServeMux()

	// функция добавляет маршруты для различных запросов
	addRouters(
		mux, 
		Logger,
		Config,
		commentStore,
		anotherStore,
	)

	var handler http.Handler = mux
	// добавляются мидллвары
	handler = someMiddleare(handler)
	handler = someMiddleare1(handler)
	handler = someMiddleare2(handler)

	// возвращается обработчик
	return handler
}
```
