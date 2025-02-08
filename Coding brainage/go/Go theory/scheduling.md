---
tags:
  - essentials
---
s`time.After` - создает канал, в который будет отправлено текущее время, через заданный аргументом функции duration

```Go
timeoutChan := time.After(time.Second)
<-timeoutChan // sleep for 1 second
```

`time.Tick` - создает канал, в котором будут появляться значения с заданным периодом

```Go
intervalChan := time.Tick(time.Second)
count := 0
for {
	<-intervalChan
	fmt.Println("interval update", count)
	count += 1
	if count >= 5 {
		break
	}
}
```

получим вот такой вывод

```Plain
interval update 2024-06-16 21:17:14.050621 +0300 MSK m=+1.001199590
interval update 2024-06-16 21:17:15.050623 +0300 MSK m=+2.001185679
interval update 2024-06-16 21:17:16.050636 +0300 MSK m=+3.001182580
interval update 2024-06-16 21:17:17.050659 +0300 MSK m=+4.001190181
interval update 2024-06-16 21:17:18.050636 +0300 MSK m=+5.001151816
```

Так же есть два метода, которые возвращают обертки над этими каналами с ручками для изменения параметров таймаута или тикера.
`time.NewTimer` - работает аналогично time.After, но такой таймер можно отключить и запустить заново с новым duration

```go
timer := time.NewTimer(time.Second * 3)
	go func() { // blocking read of the timer channel
		<-timer.C
		// this one won't be seen
		fmt.Println("go-routine got the timer-channel update")
	}()
	timer.Reset(time.Second * 1) // will trigger the channel in two seconds instead
	<-timer.C
```

---

`time.NewTicker` - работает так же как и `time.Tick`, но может быть остановлен или перенастроен на новый интервал, как и в случае с `time.NewTimer`
```go
ticker := time.NewTicker(time.Second * 1)
tickerCounter := 0
for ; ; tickerCounter++ {
	currentTime := <-ticker.C
	fmt.Println("Ticker", currentTime.Format(handyFormat))
	if tickerCounter == 3 {
		fmt.Println("Ticker: reseting timer interval from 1 to 3 seconds")
		ticker.Reset(time.Second * 3)
	}

	if tickerCounter == 5 {
		fmt.Println("Ticker: shutting down!")
		ticker.Stop()
		break
	}
}
```
---
### Throttling
Код, который N раз выводит указанное сообщение, делая это не чаще заданного количества миллисекунд
```go
func printThrottled(message string, n int, ms time.Duration) (chan struct{}) {
	done := make(chan struct{})
	workScheduler := time.NewTicker(time.Milliseconds * ms)
	go func () {
		for i := 0; i < n; i++ {
			<-workScheduler.C
			fmt.Println(message)
		}
		workScheduler.Stop()
	}()
	return done
}

func main() {
	<-printThrottled("yoooo", 10, 200)
}
```
---
### Multi-thread throttling
```go
func worker(wg *sync.WaitGroup, throttler *time.Ticker, index int, message string) {
	<-throttler.C
	fmt.Println(fmt.Sprintf("%d: %s", index, message))
	wg.Done()
}

func workThrottledMultithreaded(message string, n int, ms time.Duration) {
	wg := new(sync.WaitGroup)
	throttler := time.NewTicker(ms)
	for i := 0; i < n; i++ {
		wg.Add(1)
		go worker(wg, throttler, i, message)
	}
	wg.Wait()
}

func main() {
	workThrottledMultithreaded("worker done", 10, 200)
}
```
---
### Дефолтное поведение, когда нет апдейтов
ни от одного из каналов, на которые завязано неблокирующее чтение. (Наносекунды подобраны просто, чтобы можно было хоть что-то репрезентативное увидеть в консоли)
```go
ticker := time.NewTicker(time.Nanosecond * 10)
stop := time.NewTimer(time.Nanosecond * 50)
for {
	select {
	case stopSignal := <-stop.C:
		fmt.Println("stopped by the authority", stopSignal)
		return
	case ticketForWork := <-ticker.C:
		fmt.Println("got update from scheduler", ticketForWork)
	default:
		fmt.Println("Nothing from scheduler")
	}
}

// в консоли будет
// Nothing from scheduler
// got update from scheduler 2024-06-30 21:34:39.599871 +0300 MSK m=+0.000148541
// stopped by the authority 2024-06-30 21:34:39.599872 +0300 MSK m=+0.000149306
```
Если сделать интервалы тикера и сам таймаут побольше - то в консоль выведется миллиард строчек "Nothing from scheduler"