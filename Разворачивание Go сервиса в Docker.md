[[Go]]
[[docker]]

[видео](https://www.youtube.com/watch?v=ZEd0giJegVI)

Пример __dockerfile__ для разворачивания сервиса на __Golang__:
```Dockerfile
#  указываем базовый образ, на основании которого будем делать сборку
FROM golang:1.21-alpine AS build

#  через пакетный менеджер устанавливаем нужные утилиты (опционально)
RUN apk --no-cache add bash git make gcc gettext musl-dev

#  указываем рабочую директорию для проекта
WORKDIR /usr/local/src/

#  скачиваем зависимости, которые указаны в go.mod и в go.sum
COPY ["go.mod", "go.sum", "./"]
RUN go mod download

#  копируем все файлы в рабочую директорию
COPY . .

#  лучший вариант это копировать только папку с приложением
COPY app ./

#  собираем бинарник, на первом месте, это куда будет положен файл
#  относительный путь, на втором месте путь до main.go файла
RUN go build -o ./bin/send_mq /usr/local/src/app/main.go

#  запуск бинарного файла
ENTRYPOINT ["/usr/local/src/bin/send_mq"]
```

Пример __docker-compose__ файла:
```yaml
version: "3.9"

services:
 go_service:
  # указываем путь до папки, в которой лежит dockerfile
  build: ./service
  container_name: service_example
  # флаг рестарта
  restart: always
  # указываем порт открытый внутри docker сети (если нужно)
  expose:
   - 5422
```

Можно доделать __dockerfile__ для быстрой сборки:
```Dockerfile
#  в этом блоке просто соберем бинарник

#  указываем базовый образ, на основании которого будем делать сборку
FROM golang:1.21-alpine AS build

#  через пакетный менеджер устанавливаем нужные утилиты (опционально)
RUN apk --no-cache add bash git make gcc gettext musl-dev

#  указываем рабочую директорию для проекта
WORKDIR /usr/local/src/

#  скачиваем зависимости, которые указаны в go.mod и в go.sum
COPY ["go.mod", "go.sum", "./"]
RUN go mod download

#  лучший вариант это копировать только папку с приложением
COPY app ./

#  собираем бинарник, на первом месте, это куда будет положен файл
#  относительный путь, на втором месте путь до main.go файла
RUN go build -o ./bin/app /usr/local/src/app/main.go

#  в этой части будет происходить загрузка конфига или env. и запуск
FROM alpine AS runner

#  копируем в чистый образ собранный бинарник
COPY --from=builder /usr/local/src/bin/app /
COPY configs/config.yaml /config.yaml

CMD ["/app"]
```
