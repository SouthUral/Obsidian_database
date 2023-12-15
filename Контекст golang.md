[[многопоточный golang, конкурентность]]

Контекст в Golang нужен для отмены работы горутины.
Тип _Context_ - это интерфейс, имеющий четыре метода:
- `Deadline()`
- `Done()`
- `Err()`
- `Value()`

>[!info] Не нужно хранить контекст в структурах.
>Рекомендуется передавать контекст первым аргументом функции.

# Виды контекстов

Любой контекст должен наследоваться от родительского контекста.
![[Pasted image 20231215113935.png]]

## Базовые контексты
### `context.Background`
Это базовый контекст на основании которого получают уже нужный конекст.

### `context.TODO`
Этот контекст используется, когда нет понимания какой контекст сейчас нужно применить.

## Дочерние контексты
### `context.WithCancel`
Контекст возвращает функцию, с помощью которой можно отменить контекст.
```Go
package main

import (
	"context"
	"time"
	"fmt"
)

func f1(t int) {
	c1 := context.Background()
	c1, cancel := context.WithCancel(c1)

	go func() {
		time.Sleep(4 * time.Second)
		cancel()
	}()

	select {
	case <- c1.Done():
		fmt.Println("f1():", c1.Err())
		return
	case r := <- time.After(time.Duration(t) * time.Second):
		fmt.Println("f1():", r)
	}
	return
}
```
Функция `context.WithCancel(ctx)` на основе переданного ей контекста создает новый контекст. Так же эта функция создает канал `Done`, который можно закрыть либо при вызове функции `cancel()`, либо когда закрывается канал `Done` родительского контекста.

### `context.WithTimeout`
```Go

func f2(t int) {
	c2 := context.Background()
	c2, cancel := context.WithTimeout(c2, time.Duration(t) * time.Second)
	defer cancel()

	select {
	case <-c2.Done():
		fmt.Println("f2():", c2.Err())
		return
	case r := <- time.After(time.Duration(t) * time.Second):
		fmt.Println("f2():", r)
	}
	return
}
```
Здесь  `context.WithTimeout()` принимает в качестве аргументов родительский контекст и длительность в формате `time.Duration`. По окончанию времени ожидания автоматически вызывается функция `cancel()`
### `context.WithDeadline`
```Go

func f3(t int) {
	c3 := context.Background()
	deadline := time.Now().Add(time.Duration(2*t) * time.Second)
	c3, cancel := context.WithDeadline(c3, deadline)

	defer cancel()

	select {
	case <- c3.Done():
		fmt.Println("f3():", c3.Err())
		return
	case r := <- time.After(time.Duration(t) * time.Second):
		fmt.Println("f3():", r)
	}
	return
}
```
Функция `context.WithDeadline()` принимает в качестве аргумента родительский контекст и время в будущем, соответствующее крайнему сроку окончания операции. По истечении этого времени автоматически будет вызвана функция `cancel()`.

### `context.WithValue`
```Go
package main

import (
	"context"
	"fmt"
)

type aKey string

func searchKey(ctx context.Context, key aKey) {
	v := ctx.Value(key)
	if v != nil {
		fmt.Println("found value: ", v)
		return
	}
	fmt.Println("key not found: ", key)
}

func main() {
	myKey := aKey("mySecret")
	ctx := context.WithValue(context.Background(), myKey, mySecretValue)

	searchKey(ctx, myKey)
}
```
Функция `searchKey()` извлекает значение по ключу из переданного ей контекста. Функция `context.WithValue()` представляет собой способ связывания значения с контекстом. 
>[!warning] Контекст не нужно хранить в структурах

## Советы для работы с контекстом
1) Всегда передавать контекст первым аргументом.
2) Передавать контекст только в функции и методы. Контекст это одноразовые и неизменяемые объекты, если сохранить контекст с тайм-аутом 15 секунд в структуре, то через 15 секунд работы он уже сработает и больше использовать его не получится.
3) Использовать `context.WithValue` только в крайних случаях.
4) `context.Background` должен использоваться только как родительский контекст.
5) `context.TODO` можно использовать, если пока нет понимания, какой контекст нужно использовать в коде.

