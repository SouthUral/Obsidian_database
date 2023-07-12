[[Каналы golang]]

```go
package main

import (
	"fmt"
	"time"
)

func goTimer(time_sleep int, ch chan string) {
	counter := 0
	for counter < 30 {
		time.Sleep(time.Duration(time_sleep) * time.Millisecond)
		counter++
		ch <- fmt.Sprintf("горутина : %d, сработала: %d раз", time_sleep, counter)
	}
	fmt.Println(fmt.Sprintf("горутина : %d, закончила свою работу", time_sleep))
}

func main() {
	chan1 := make(chan string, 100)
	chan2 := make(chan string, 100)
	chan3 := make(chan string, 100)
	
	go goTimer(1, chan1)
	go goTimer(2, chan2)
	go goTimer(10, chan3)
	
	for i := 0; i < 100; i++ {
		select {
		case res1 := <-chan1:
		fmt.Println(res1)
		case res2 := <-chan2:
		fmt.Println(res2)
		case res3 := <-chan3:
		fmt.Println(res3)
		}
	}
	close(chan1)
	close(chan2)
	close(chan3)
}
```