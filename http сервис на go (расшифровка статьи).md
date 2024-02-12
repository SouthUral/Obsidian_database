[статья про сервер на go](https://grafana.com/blog/2024/02/09/how-i-write-http-services-in-go-after-13-years/)

# Описание

Статья содержит лучшие современные практики для реализации http сервера на golang

# Конструктор NewServer
Пример кода:
```go

// 
func NewServer(
	logger *Logger
	config *Config
	commentStore *CommentStore
	anotherStore *AnotherStore
) http.Handler {
	mux := http.NewServeMux()
	addRouters(
		mux, 
		Logger,
		Config,
		commentStore,
		anotherStore,
	)

	var handler http.Handler = mux
	handler = someMiddleare(handler)
	handler = someMiddleare1(handler)
	handler = someMiddleare2(handler)
	
	return handler
}
```
