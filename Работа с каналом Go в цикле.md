[[Каналы golang]]

Когда неизвестно сколько данных ожидать можно получать сообщения из канала и проверять при этом статус канала, закрыт он или нет.
Пример:
```go
func main() {
	newChan := make(chan int)
}

func squares(ch chan int) {
	for {
		val, ok := <- ch
		if ok == false {
			fmt.Println(val, ok, "<-- loop broke")
			break
		}
		fmt.Println(val, ok)
		
	}
}
```
Для удобства можно использовать в цикле `range` чтобы остановить цикл когда канал будет закрыт:
```go
func squares(ch chan int) {
	for val := range ch {
		fmt.Println(val)
	}
}
```
Конструкция будет блокироваться если сообщения в канале не будет