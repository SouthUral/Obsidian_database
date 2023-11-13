[[gRPC]]
[[Разработка Web приложения на Golang]]

# Простой запуск gRPC клиента и сервера

## Установить плагины компилятора протокола для Go
```shell
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2
```

## Обновить путь GOPATH
Обновить PATH, чтобы `protoc` компилятор мог найти плагины
```shell
$ export PATH="$PATH:$(go env GOPATH)/bin"
```

## Составление `proto` файла (спецификации)
