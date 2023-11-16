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

## Генерация кода (неактуально)
```shell
protoc -I ecommerce \
	ecommerce/product_info.proto \
	--go_out=plugins=grpc:<module_dir_path>/ecommerce
```

- `protoc -I ecommerce` - каталог в котором хранится исходный __proto-файл__ и его зависимости. Указывается с помощью флага `--proto_path` или  `-I`. Если ничего не указать то в качестве исходного будет использоваться текущий каталог.

 - `ecommerce/product_info.proto` - путь к __proto-файлу__ который нужно скомпилировать.

 - `--go_out=plugins=grpc:<module_dir_path>` - каталог, в котором будет сохранен сгенерированный код.

>[!warning]
>Генерация кода, которая указана выше не поддерживается

## Создание сервера
### Создание спецификации
Создать файл с расширением `.proto`
```proto
// Определение сервиса начинается с указания версии Protocol Buffers
syntax = "proto3";

// Имя пакета
package ecommers;

option go_package = "github.com/productinfo/service";

// Определение интерфейса gRPC-сервиса
service ProductInfo {
	// Удаленный метод для добавление Product, возвращает ProductID
	rpc addProduct(Product) returns(ProductID); 
	// Удаленный метод получения товара по ID
	rpc getProduct(ProductID) returns(Product);
}

// Определение формата сообщения Product
message Product {
	// ПОля сообщения Product, которые имеют свой уникальный ID
	string id = 1;
	string name = 2;
	string description = 3;
}

// Определение формата сообщения ProductID
message ProductID {
	string value = 1;
}
```

### Генерация кода
Теперь нужно сгенерировать код на основе созданного протофайла.
```shell
protoc \
--go_out=service \
--go_opt=paths=source_relative \
--go-grpc_out=service \
--go-grpc_opt=paths=source_relative \
ecommerce/product_info.proto
```
В этом варианте код будет сгенерирован в папке `service`
- `protoc` - вызов программы кодогенерации из протофайла;
- `--go_out=service` - флаг, в который означает что мы хотим сгенерировать golang код в папке service;
- `--go_opt=paths=source_relative` - это  флаг опции, который означает, что путь будет относительным местоположению протофайла;
- `--go-grpc_out=service` - флаг, для генерации grpc файла???;
- `--go-grpc_opt=paths=source_relative` -  это  флаг опции, который означает, что путь будет относительным местоположению протофайла;
- `ecommerce/product_info.proto` - указание местоположения самого протофайла относительно текущего положения в терминале.

```shell
protoc \
--go_out=. \
--go_opt=paths=source_relative \
--go-grpc_out=. \
--go-grpc_opt=paths=source_relative \
ecommerce/product_info.proto
```
В этом варианте файлы с кодом будут помещены в ту же папку, где находится протофайл.

### Загрузка сторонних пакетов
В сгенерированных файлах используются сторонние пакеты:
файл `product_info_grpc.pb.go`
```go
package service

import (
context "context"

	grpc "google.golang.org/grpc"
	codes "google.golang.org/grpc/codes"
	status "google.golang.org/grpc/status"
)
```
и файл `product_info.pb.go`
```Go
package service

import (
	protoreflect "google.golang.org/protobuf/reflect/protoreflect"
	protoimpl "google.golang.org/protobuf/runtime/protoimpl"
	reflect "reflect"

	sync "sync"
)
```

Для загрузки последних версий пакетов нужно выполнить команду:
```shell
go get -u google.golang.org/grpc 
go mod tidy 
```
`go mod tidy` - используется для обновления зависимостей

```shell
protoc --go_out=. --go_opt=paths=source_relative \                               
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    ecommerce/product_info.proto
```
Эта команда создаст в папке `ecommerce/` два файла:
 - `product_info_grpc.pb.go` -  содержит:
	 - Тип интерфейса (или заглушку), который могут вызвать клиенты;
	 - Тип интерфейса, который будут реализовывать серверы.
 - `product_info.pb.go` - содержит весь код буфера протокола для заполнения, сериализации и получения типов сообщений запроса ответа.