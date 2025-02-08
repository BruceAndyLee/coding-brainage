---
tags:
  - essentials
---
### Введение с примерами

go-рутина это поток исполнения, который выполняет одну функцию.  
Потоки в go управляются самим языком и  
_**вроде как**_ считаются более легковесными, чем отдельные процессы в системе

- **go-рутина, исполняющая анонимную функцию**
    
    ```Go
    
    func main() {
    	go func() {
    	  fmt.Println("Me a gopher")
    	}
    	time.Sleep(time.Second * 1) // give the child-goroutine some time to execute
    }
    ```
    

Го-рутины могут общаться между собой с помощью каналов.

Чтение из и запись в каналы - блокирующая, если не использовать специальные подпрыгивания.

Когда канал больше не нужен его можно закрыть встроенной функцией `close`.

Чтение сообщений из канала можно организовать с помощью `for .. range`

- пример
    
    ```Go
    package main
    
    import (
    	"fmt"
    	"time"
    )
    
    func main() {
    	messageChan := make(chan string) // buffer size is 1, only 1 string per read/write op
    
    	go func() {
    		messageChan <- "hewwo"
    		time.Sleep(time.Second * 1)
    		messageChan <- "there"
    		close(messageChan)
    	}()
    
    	go func() {
    		// greeting := <-messageChan
    		for message := range messageChan {
    			fmt.Println("I got a message:", message)
    		}
    	}()
    	time.Sleep(time.Second * 2)
    }
    
    // I got a message: hewwo
    // I got a message: there
    ```
    

нельзя писать новые данные в закрытый канал

читать данные из закрытого канала можно, читающая go-рутина получит все непрочитанные на момент чтения данные

при попытке прочитать данные из пустого канала, читающая go-рутина получит нулевое значение, соответствующее типу канала

---

### Синтаксис

блокирующая запись в канал, пошлет значение переменной `my_number`

```Go
nums_channel ← my_number
```

блокирующее чтение из канала. Запишет значение из канала в локальную переменную

```Go
received_number = ← nums_channel
```

Неблокирующая запись в канал

```Go
select {
case nums_channel <- my_number:
  // do something
}
```

Неблокирующее чтение из канала (самое простое применение - ожидание сигнала завершать работу)

```Go
select {
case received_number = <- nums_channel:
  // do something
}
```

количество непрочитаных значений в канале на текущий момент

```Go
len(chan)
```

вернет размер буфера канала

```Go
cap(chan)
```

объявление read-only и write-only каналов

```Go
send_numbers := make(chan<- int)
receive_numbers := make(<-chan int)
```

разумеется, каналы можно передавать и возвращать из функций

### Пример с неблокирующим поведением

Возможно с помощью конструкции `select-case`. В отдельной го-рутине можно запустить вычисление _некоторых_ данных и их отправку в канал для принимающей го-рутины.

  

```Go
func generate_fibonacci(nums, control chan int) {
	x, y := 0, 1
	var control_message int
	for {
		select {
		case nums <- x:
			x, y = y, x+y
		case control_message = <-control:
			if control_message == 1 {
				fmt.Println("fibonacci generator was asked to stop!")
				// если не завершить функцию, то получится run-time error: fatal error: all goroutines are asleep - deadlock!
				return
			}
		}
	}
}

func read_results(nums, control chan int, read_count int) {
	for i := 0; i < read_count; i++ {
		fmt.Println("Obtained number:", <-nums)
	}
	control <- 1 // send in the 'stop' command
}

func main() {
	numbers_channel := make(chan int)
	control_channel := make(chan int) // send 1 to stop the producer go-routine

	go read_results(numbers_channel, control_channel, 10)
	generate_fibonacci(numbers_channel, control_channel)
}
```

---

### Управление выполнением горутин

Вместо того, чтобы писать команды типа sleep, чтобы дать время всем горутинам отработать, можно повесить в main-горутине блокирующее чтение из канала, в который все дочерние горутины сообщат о своей готовности. Причем, необязательно для этого посылать явное сообщение, достаточно закрыть канал, чтобы go сам понял, что все блокирующие чтения можно разблокировать:

```Go
func sideRoutine(done chan struct{}) {
	fmt.Println("Oh boye, I sure did do some hefty work here")
	close(done) // go will understand that the blocking read should be relinquished in the main-func go-routine
}

func main() {
	done := make(chan struct{}) // use empty struct to avoid allocating memory
	go sideRoutine(done)
	<-done // blocking read operation
}
```

Для пущей красивости, го-рутину можно обернуть в отдельную функцию, которая возвращает тот самый канал для управляющих сообщений:

```Go
func sideRoutine() <-chan struct{} {
	done := make(chan struct{}) // если создать done в объявлении возвращаемого типа, почему-то go запретит закрывать такой канал
	go func() {
		defer close(done) // EVEN MORE GO FLAVOUR
		fmt.Println("Oh boye, I sure did do some hefty work here")
		time.Sleep(time.Second * 2)
	}()
	return done
}

func main() {
	<-sideRoutine() // вызывающий код теперь гораздо изящщщнее (go-flavoured?)
}
```