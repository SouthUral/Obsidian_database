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

Через миддлвары обеспечивается вся дополнительная логика, как, например, авторизация, ведение журнала:

```Go
var handler http.Handler = mux
handler = logging.NewLoggingMiddleware(logger, handler)
handler = logging.NewGoogleTraceIDMiddleware(logger, handler)
handler = checkAuthHeaders(handler)
return handler
```

# Настройка сервера
```Go
// создание обработчика
srv := NewServer(
	logger,
	config,
	tenantsStore,
	slackLinkStore,
	msteamsLinkStore,
	proxy,
)

// настройка сервера
httpServer := &http.Server{
	Addr:    net.JoinHostPort(config.Host, config.Port),
	Handler: srv,
}

// в горутине производится запуск сервера
go func() {
	log.Printf("listening on %s\n", httpServer.Addr)
	if err := httpServer.ListenAndServe(); err != nil && err != http.ErrServerClosed {
		fmt.Fprintf(os.Stderr, "error listening and serving: %s\n", err)
	}
}()

var wg sync.WaitGroup
wg.Add(1)

// горутина, которая должна остановить сервер при отмене контекста
go func() {
	defer wg.Done()
	<-ctx.Done()
	if err := httpServer.Shutdown(ctx); err != nil {
		fmt.Fprintf(os.Stderr, "error shutting down http server: %s\n", err)
	}
}()

wg.Wait()
return nil
```