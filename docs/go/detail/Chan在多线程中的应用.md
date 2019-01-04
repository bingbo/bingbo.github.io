信道(chan)是带有类型的管道，你可以通过它用信道操作符 <- 来发送或者接收值。箭头就是数据流的方向,默认情况下，发送和接收操作在另一端准备好之前都会阻塞。这使得 Go 程可以在没有显式的锁或竞态变量的情况下进行同步。

发送者可通过`close`关闭一个信道来表示没有需要发送的值了。接收者可以通过为接收表达式分配第二个参数来测试信道是否被关闭：若没有值可以接收且信道已被关闭，那么在执行完
```go
v, ok := <-ch
// 之后 ok 会被设置为 false 。
```

### 通过chan实现线程间交互

> 主线程进行输出结果，另一线程做计算

```Go
package main

import "fmt"



func main() {
	c := make(chan int, 10)
	// 新建线程去计算
	go fibonacci(cap(c), c)
	
	// 等待有计算完的结果则输出，不断从信道接收值，直到它被关闭
	for i := range c {
		fmt.Println(i)
	}

}

// 计算
func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y

		time.Sleep(1000 * time.Millisecond)
	}
	close(c)
}

```

### 多线程做同样的事情，等待都结束后，再进行其他操作

> 多个线程，如10个线程输出hello,world,主线程等待都结束后再往下进行

```go
package main

import "fmt"
import "time"
import "math/rand"

func main() {
	
	c1 := make(chan int, 10)
	// 通过多线程执行操作
	for i := 0; i < 10; i++ {
		go sayHi(c1)
	}
	// 等待全部结束再继续往下进行
	for i := 0; i < 10; {
		<-c1
		i++
	}

	fmt.Println("done ..")

}

// 输出helloworld
func sayHi(c chan int) {

	// 随机暂停一段时间，模拟操作过程
	time.Sleep(time.Duration(rand.Intn(10)*10000) * time.Millisecond)
	fmt.Println("hello,world")
	// 发送信号
	c <- 1

}
```

### 多线程执行，等待全部执行完毕，`sync.WaitGroup`的应用

```go
package main

import "fmt"
import "time"
import "math/rand"
import "sync"

func main() {
	

	var wg sync.WaitGroup
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func(num int) {
			time.Sleep(time.Duration(rand.Intn(10)*10000) * time.Millisecond)
			fmt.Println("compute num ", num)
			wg.Done()
		}(i)
	}
	wg.Wait()

	fmt.Println("done ..")

}
```