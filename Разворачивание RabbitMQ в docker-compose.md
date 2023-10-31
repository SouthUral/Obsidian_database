[[Rabbit MQ]]
[[docker]]
[[docker-compose]]

[ссылка на статью на хабре](https://habr.com/ru/companies/slurm/articles/704208/)

### Простой запуск rabbitMQ через docker:

```shell
docker run --rm -p 15672:15672 rabbitmq:3.10.7-management
```

Но такой запуск не позволяет тонко настроить работу rabbitMQ

#### Формирование простого docker-compose.yml:
```yaml
version: "3.8"

services:
 rabbit_test_shoveltrucks:
  image: rabbitmq:3.13-rc-management
  ports:
   - 15672:15672
```

Запуск командой: `docker-compose up`
Для авторизации стандартный логин/пароль `guest/guest`

### Добавление _hostname_ и изменение стандартного _user_

![[Pasted image 20231002132656.png]]
в строчке где указан кластер `rabbit@9effa89f4b1f`,  `9effa89f4b1f` - это ID контейнера в docker.
Это не очень, т.к. RabbitMQ хранит свой стейт в папках, содержащих название сервера. А Docker при пересеоздании контейнера дает ему случайные ID. При каждом пересоздании контейнера RabbitMQ будет работать с нуля.

Дополним `docker-compose.yml`:
```yml
version: "3.8"

services:
 rabbitmq:
  image: rabbitmq:3.13-rc-management
  hostname: rabbitmq
  restart: always
  enviroment:
   - RABBITMQ_DEFAULT_USER=test_user
   - RABBITMQ_DEFAULT_PASS=rmpassword
  ports:
   - 15672:15672
```

### Изменение лимита записи
После запуска перейдем на страницу ==Overview==. Нужно обратить внимание на это поле:
![[Pasted image 20231002134020.png]]
на `48 MiB low watermark` -  это означает, что RabbitMQ перейдет в защиту и перестанет писать в стейт при свободном месте менее  48Mbit, это очень мало и забивается в 90% случаях. Если RabbitMQ попытается записать на диск, где нет места это скорее всего уничтожит стейт без возможности восстановления. Поэтому рекомендуется для прода менять это значение на 2Gb (2147483648 бит).
Это делается через переменную окружения ==**RABBITMQ_SERVER_ADDITIONAL_ERL_ARGS**== и поле ==**disk_free_limitdisk_free_limit**==.

```yml
version: "3.8"

services:
 rabbitmq:
  image: rabbitmq:3.13-rc-management
  hostname: rabbitmq
  restart: always
  enviroment:
   - RABBITMQ_DEFAULT_USER=test_user
   - RABBITMQ_DEFAULT_PASS=rmpassword
   - RABBITMQ_SERVER_ADDITIONAL_ERL_ARGS=disk_free_limitdisk_free_limit 2147483648
  ports:
   - 15672:15672
```

### Добавление _volume_

Добавим volume для сохранения стейта RabbitMQ локально на диск, чтобы стейт хранился не только в контейнере но и на диске.

```yml
version: "3.8"

services:
 rabbitmq:
  image: rabbitmq:3.13-rc-management
  hostname: rabbitmq
  restart: always
  enviroment:
   - RABBITMQ_DEFAULT_USER=test_user
   - RABBITMQ_DEFAULT_PASS=rmpassword
   - RABBITMQ_SERVER_ADDITIONAL_ERL_ARGS=disk_free_limitdisk_free_limit 2147483648
  volumes:
   - ./rabbitmq:/var/lib/rabbitmq
  ports:
   - 15672:15672
```

### Изменение уровня логирования 
По умолчанию уровень логирования в RabbitMQ - _info_ , что слишком много для нагруженных систем. Изменение уровня логирования на ==error==:

```yml
version: "3.8"

services:
 rabbitmq:
  image: rabbitmq:3.13-rc-management
  hostname: rabbitmq
  restart: always
  enviroment:
   - RABBITMQ_DEFAULT_USER=test_user
   - RABBITMQ_DEFAULT_PASS=rmpassword
   - RABBITMQ_SERVER_ADDITIONAL_ERL_ARGS=-rabbit log_levels [{connection,error},{default,error}] disk_free_limit 2147483648
  volumes:
   - ./rabbitmq:/var/lib/rabbitmq
  ports:
   - 15672:15672
```

### Открытие порта AMQP 5672

Если сервисы которые будут подписываться на очереди или публиковать будут находиться в одной сети докера, то открывать порт не нужно, если же консьюмеры и паблишеры находятся вне сети docker тогда нужно открыть этот порт.

```yml
version: "3.8"

services:
 rabbitmq:
  image: rabbitmq:3.13-rc-management
  hostname: rabbitmq
  restart: always
  environment:
   - RABBITMQ_DEFAULT_USER=test_user
   - RABBITMQ_DEFAULT_PASS=rmpassword
   - RABBITMQ_SERVER_ADDITIONAL_ERL_ARGS=-rabbit log_levels [{connection,error},{default,error}] disk_free_limit 2147483648
  volumes:
   - ./rabbitmq:/var/lib/rabbitmq
  ports:
   - 15672:15672
   - 5672:5672
```

### Вход в docker контейнер с RabbitMQ
```shell
docker-compose exec rabbitmq bash
```
Здесь `rabbitmq` это имя контейнера