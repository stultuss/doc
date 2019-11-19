Golang 基础-通道
=========================
> 学习笔记

通道（channel）是 Go 中的一种特殊类型，就像一个可以用于发送类型化数据的管道，由其负责协程间的通信。从而避开所有由共享内存导致的陷阱；通过通道进行通信的方式保证了同步性。数据可以在通道间进行传递：**在任何给定时间，一个数据被设计只有一个协程可以对其访问，所以不会发生数据竞争。**数据的所有权也可以可以被传递。所有类型都可以被通道传递，包括空接口，甚至通道的通道，非常灵活。

创建一个通道

```go
ch := make(chan int) // 无 buffer 的通道很容易被阻塞，导致 goroutine 睡眠。
```

创建一个带缓存的通道

```go
ch := make(chan int, 10) // 有 buffer 的通道在 buffer 耗尽后，同样也会被阻塞。
```

### 例子

生产者-消费者模式（无 buffer）

```go
package main

import (
	"fmt"
	"time"
)

func sender(ch chan<- int) {
	i := 1 // 建议传递的信号不是通道类型的默认值。
	for {
		fmt.Println("Send:", i)
		ch <- i
		time.Sleep(time.Millisecond * 100)
		if i == 10 {
			close(ch) // 生产者执行结束，关闭通道，但消费者依然可以收到信号，直到信号为 0
			fmt.Println("Send Exit!")
			break
		} else {
			i++
		}
	}
}

func receiver(ch <-chan int) {
	for {
		v := <-ch
		if v == 0 { // 当收到通道信号的默认值就认为信号已经结束了。
			fmt.Println("Receive Exit!")
			break
		} else {
			fmt.Println("Receive:", v)
		}
	}
}

func main() {
	ch := make(chan int) // 创建一个无缓冲的通道
	go sender(ch)        // 生产者向通道内传递信号
	go receiver(ch)      // 消费者阻塞并等待生产者传递的数据，收到信号执行打印，然后继续阻塞并等待
	time.Sleep(1 * time.Second)
}
```

生产者-消费者模式（有 buffer）

```go
package main

import (
	"fmt"
	"time"
)

func sender(ch chan<- int) {
	i := 1 // 建议通道第一个值不是通道类型的默认值，即 int = 0, string = "" 等，这样消费者在收到默认值的时候就直到消息结束了。
	for {
		fmt.Println("Send:", i)
		ch <- i // 一次性向通道里写入 10 个信号，先进先出
		if i == 10 {
			close(ch) // 生产者执行结束，关闭通道，但消费者依然可以收到信号，直到信号为 0
			fmt.Println("Send Exit!")
			break
		} else {
			i++
		}
	}
}

func receiver(ch <-chan int) {
	for {
		v := <-ch
		if v == 0 {
			fmt.Println("Receive Exit!")
			break
		} else {
			fmt.Println("Receive:", v)
		}
	}
}

func main() {
	ch := make(chan int, 10) // 创建一个带缓冲的通道
	go sender(ch)        // 生产者向通道内传递信号
	go receiver(ch)      // 消费者阻塞并等待生产者传递的数据，收到信号执行打印，然后继续阻塞并等待下一次个信号
	time.Sleep(1 * time.Second)
}
```

上面两个生产者消费者的例子的运行结果表明

- 无缓冲的通道：每一次的通道数据交换，消费者都要阻塞一次，共计10次阻塞，打印10条数据。
- 带缓冲的通道：生产者一次可写入10条数据，消费者只阻塞一次，就可以打印10条数据。

## Select

从不同的并发执行的协程中获取值可以通过关键字 `select` 来完成，结构如下：

```go
select {
case u:= <- ch1:
        ...
case v:= <- ch2:
        ...
        ...
default: // no value ready to be received
        ...
}
```

选择处理列出的多个通信情况中的一个。

- 如果都阻塞了，会等待直到其中一个可以处理
- 如果多个可以处理，随机选择一个
- 如果没有通道操作可以处理并且写了 `default` 语句，它就会执行：`default` 永远是可运行的（这就是准备好了，可以执行）。

## Ticker

计时器（Ticker）是 time 包中的一个功能，它以指定的时间间隔重复的向通道C发送时间值，它的结构体如下：

```go
type Ticker struct {
    C <-chan Time // the channel on which the ticks are delivered.
    // contains filtered or unexported fields
    ...
}
```

简单的例子：

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	tick := time.Tick(time.Second)
	boom := time.After(time.Second * 10)

	i := 0
	for {
		select {
		case <-tick:
			fmt.Println("tick.", i)
			i++
		case <-boom:
			fmt.Println("BOOM!")
			return
		}
	}
}
```

# 服务限流

> 服务限流：微服务架构设计-服务治理中的一个概念，用来保护服务节点和集群节点后面的数据节点，防止瞬时流量过大导致服务和数据崩溃，造成整体服务不可用。

使用带缓冲区的通道可以很容易的设计出一个简易的限流服务，真实场景中还需要考虑其他一系列的问题，例如限制频率，请求分流等。

```go
package main

import (
	"fmt"
	"time"
)

const MAX_REQ_COUNT = 5

// 创建一个带缓冲的通道，缓冲长度为 5
var sem = make(chan int, MAX_REQ_COUNT)

type TaskStruct struct {
	id    int
	reply chan int
}

func main() {
	reply := make(chan int)

  // 一次接收 100 个任务，每 2 秒处理 5 个任务
	n := 100
	i := 0
	for {
		if i == n {
			break
		}

		task := TaskStruct{id: i, reply: reply}
		fmt.Printf("Task Id: %d \n", i)
		go handleTask(&task)
		i++
	}

	for i := 0; i < n; i++ {
		<-reply
	}
}

func handleTask(t *TaskStruct) {
	sem <- 1
	processTask(t)
	<-sem
}

func processTask(t *TaskStruct) {
	time.Sleep(time.Second * 2)
	fmt.Printf("Task Id: %d done \n", t.id)
	t.reply <- t.id
}

```

