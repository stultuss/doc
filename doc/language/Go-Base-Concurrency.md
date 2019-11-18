Golang 基础-并发
=========================
> 学习笔记

## 概念

首先要理解两个概念，“多核” 和 “并发”，分别了解他们在 Go 语言中如何实现。

- “多核” 是指有效利用 CPU 的多核提高程序执行效率，在 Go 中通过 GOMAXPROCS 实现。
- “并发” 是由 CPU 内核通过时间片或者终端来控制的，遇到 IO 阻塞或者时间片用完时会交出线程的使用权，从而实现在一个内核上处理多个任务，本质还是同一时刻只有一条指令在执行，在 Go 中通过 Goroutine 实现。

## 原理

> Go 语言天生支持并发，它实现了两种并发模型：传统并发模型和 CSP 并发模型。

传统并发模型常见于 Java，Python，C++，通过共享内存的方式来进行。比较典型的例子：在访问共享数据时，要通过锁来访问，所以衍生出了一种数据结构：**线程安全的数据结构**。而 CSP 并发模型则不同于传统模型，他的作者给出了下面这句话：

> **Do not communicate by sharing memory; instead, share memory by communicating.**
>
> “不要以共享内存的方式来通信，相反，要通过通信来共享内存。”

Go 语言的 CSP 并发模型是通过 Goroutine 和 channel 来实现的， Goroutine 是 Go 语言的灵魂所在。在使用 Goroutine 之前，我们需要理解 Goroutine 是如何被调度和执行的。

##### 调度模型 ：

1. *G (Goroutine)* : 一个 Goroutine 对应一个 G 结构体，存储了 Goroutine 的运行对战，状态以及任务函数。每个 G 必须绑定 P 才能被调度执行。
2. *P (Process)* : 表示逻辑处理器，对于 G 来说， P 相当于 CPU 核。P 提供了相关的执行上下文（Context），P 的数量决定了系统内最大可并行的 G 的数量， P 的数量由用户设置的 GOMAXPROCS 决定，最大值为 256。
3. *M (Machine)* : 一个 M 代表一个内核线程，代表真正执行计算的 CPU 资源。

##### 调度流程

首先创建一个 G 对象，G 对象保存到 P 本地队列或者全局队列，然后 P 去唤醒一个 M， P 继续执行它的执行序。 M 寻找是否有空闲的 P，如果有则将该 G 对象移动到它本身，接下来 M 执行一个调度循环。



![v2-8bc01c320938ff28083388a0bb762357_hd](https://pic4.zhimg.com/80/v2-8bc01c320938ff28083388a0bb762357_hd.jpg)

## Goroutine

> Go 程是一个由 Go 程序的环境管理的轻量级线程，创建一个 Go 程意味着并行运行一个 Go 程。

**创建**一个 Go 程很简单

```go
package main

import (
	"fmt"
)

func main() {
	go func() {
		fmt.Println("Does it work?")
	}()
	time.Sleep(100 * time.Microsecond) // 等待 100 微秒
	fmt.Println("Exit")
}
```

当 main() 程序退出时，Go 程也会自动退出。想一想上面的例子 Go 程会打印什么？为什么需要增加 time.Sleep 等待 Go 程运行结束？删掉会发生什么事情？

## Channel

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

## Sync

Go 语言在 sync 包中提供了用于同步的一些基本原语，包括常见的互斥锁 `Mutex` 与读写互斥锁 `RWMutex` 以及 `Once`、`WaitGroup`，主要作用是提供较为基础的同步功能。官方推荐使用 `Channel` 来实现通信和同步，而 sync 包作为次优选择。

### Mutex

互斥锁，由两个字端 `state` 和 `sema` 组程，它的最低三位表示 `Locked` 、 `Woken` 以及`Starving`，剩下的位置都用来表示当前有多少个 Goroutine 等待互斥锁被释放，它的结构体如下：

```go
type Mutex struct {
    state int32
    sema  uint32
}
```

## 例子

### 1. 简单并发计算

根据核心数量来分配计算任务，并进行合并计算。

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func counter(seq int, ch chan int) {
	defer func() {
		close(ch)
		fmt.Printf("协程结束, 序号%d\n", seq)
	}()
	sum := 0
	for i := 1; i <= 1000000; i++ {
		sum++
	}
	ch <- sum
}

func main() {
	// 开始时间
	start := time.Now()
	// 获取 CPU 核心数量
	cpus := runtime.NumCPU()
	// 允许开启多少数量的 P 线程
	runtime.GOMAXPROCS(cpus) // cpus | 1
	chs := make([]chan int, cpus)
	for i := 0; i < len(chs); i++ {
		chs[i] = make(chan int, 1)
		go counter(i, chs[i])
	}
	// 获取通道信号
	sum := 0
	for _, ch := range chs {
		res := <-ch
		sum += res
	}
	// 结束时间
	end := time.Now()
	fmt.Printf("结果: %d, 耗时(s): %f\n", sum, end.Sub(start).Seconds())
}
```

### 2. 并发同步计算

通过 Go 程和 sync.Mutex 互斥锁进行并发同步。Mutex 本质还是通过共享内存实现并发同步。

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
	"time"
)

var count = 0

func counter(seq int, lock *sync.Mutex) {
	lock.Lock()
	original := count
	count++
	fmt.Printf("协程结束, 序号%d, 计算前%d, 计算后%d\n", seq, original, count)
	lock.Unlock()
}

func main() {
	// 开始时间
	start := time.Now()
	// 获取 CPU 核心数量
	cpus := runtime.NumCPU()
	// 允许开启多少数量的 P 线程
	runtime.GOMAXPROCS(cpus) // cpus | 1

	// 传递指针是为了防止 函数内的锁和 调用锁不一致
	lock := &sync.Mutex{}
	sum := 100
	for i := 0; i < cpus; i++ {
		for j := 0; j < sum; j++ {
			go counter(i * j, lock)
		}
	}

	for {
		// 让出时间片给别的 Go 程
		runtime.Gosched()
		if count >= cpus * sum {
			// 结束时间
			end := time.Now()
			fmt.Printf("结果: %d, 耗时(s): %f\n", count, end.Sub(start).Seconds())
			break
		}
	}
}

```

使用 channel 实现符合 CSP 并发模型的并发同步 (实现一个互斥锁)

> 信号量模式：通道信道回报，通知主线程在协程中的处理已经完成。

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

// 创建一个信号量
var count = 0

func counter(seq int, sem chan int) {
	sem <- 1 // 通道上锁，通道数据未输出之前，go 程被阻塞
	original := count
	count++
	fmt.Printf("协程结束, 序号%d, 计算前%d, 计算后%d\n", seq, original, count)
	<- sem // 释放通道
}

func main() {
	// 开始时间
	start := time.Now()
	// 获取 CPU 核心数量
	cpus := runtime.NumCPU()
	// 允许开启多少数量的 P 线程
	runtime.GOMAXPROCS(cpus) // cpus | 1

	// 创建一个通道
	sem := make(chan int, 1)
	sum := 100
	// 传递指针是为了防止 函数内的锁和 调用锁不一致
	for i := 0; i < cpus; i++ {
		for j := 0; j < sum; j++ {
			go counter(i * j, sem)
		}
	}

	for {
		// 让出时间片给别的 Go 程
		runtime.Gosched()
		if count >= cpus * sum {
			// 结束时间
			end := time.Now()
			time.Sleep(time.Second)
			fmt.Printf("结果: %d, 耗时(s): %f\n", count, end.Sub(start).Seconds())
			break
		}
	}
}
```