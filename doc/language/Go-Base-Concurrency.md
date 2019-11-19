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

## 多核计算

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
	defer lock.Unlock()
	original := count
	count++
	fmt.Printf("协程结束, 序号%d, 计算前%d, 计算后%d\n", seq, original, count)
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

这里有一个来自 Go 指导的很赞的例子，打印了输出的素数，使用选择器（‘筛’）作为它的算法。每个 prime 都有一个选择器：

```go
// Copyright 2009 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.package main
package main

import (
	"fmt"
	"time"
)

// Send the sequence 2, 3, 4, ... to channel 'ch'.
func generate(ch chan int) {
	for i := 2; ; i++ {
		ch <- i // Send 'i' to channel 'ch'.
	}
}

// Copy the values from channel 'in' to channel 'out',
// removing those divisible by 'prime'.
func filter(no int, in, out chan int, prime int) {
	for {
		time.Sleep(time.Second)
		i := <-in // Receive value of new variable 'i' from 'in'.
		if i%prime != 0 {
			fmt.Print("子线程", no, ", 运行 i：", i , ", prime:", prime, "\n")
			out <- i // Send 'i' to channel 'out'.
		}
	}
}

// The prime sieve: Daisy-chain filter processes together.
func main() {
	ch := make(chan int) // Create a new channel.
	go generate(ch)      // Start generate() as a goroutine.

	i := 0
	for {
		time.Sleep(time.Second)
		prime := <-ch
		fmt.Print(prime, " \n")
		ch1 := make(chan int)
		go filter(i, ch, ch1, prime)
		ch = ch1
		i++
	}
}
```

> 协程 `filter(in, out chan int, prime int)` 拷贝整数到输出通道，丢弃掉可以被 prime 整除的数字。然后每个 prime 又开启了一个新的协程，生成器和选择器并发请求。

## Sync 包

Go 语言在 sync 包中提供了用于同步的一些基本原语，包括常见的互斥锁 `Mutex` 与读写互斥锁 `RWMutex` 以及 `Once`、`WaitGroup`，主要作用是提供较为基础的同步功能。官方推荐使用 `Channel` 来实现通信和同步，而 sync 包作为次优选择。

### 使用共享内存进行同步

创建一个线程安全的数据结构

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
	"time"
)

type Pool struct {
	Mu    sync.Mutex
	Count int
}

func counter(seq int, pool *Pool) {
	// 加锁
	pool.Mu.Lock()
	// 解锁
	defer pool.Mu.Unlock()

	original := pool.Count
	pool.Count = original + 1
	fmt.Printf("协程结束, 序号%d, 计算前%d, 计算后%d\n", seq, original, pool.Count)
}


func main() {
	pool := Pool{
		Mu:    sync.Mutex{},
		Count: 0,
	}

	// 开始时间
	start := time.Now()
	// 获取 CPU 核心数量
	cpus := runtime.NumCPU()
	// 允许开启多少数量的 P 线程
	runtime.GOMAXPROCS(cpus) // cpus | 1

	// 传递指针是为了防止 函数内的锁和 调用锁不一致
	sum := 100
	for i := 0; i < cpus; i++ {
		for j := 0; j < sum; j++ {
			go counter(i * j, &pool)
		}
	}

	for {
		// 让出时间片给别的 Go 程
		runtime.Gosched()
		if pool.Count >= cpus * sum {
			// 结束时间
			end := time.Now()
			fmt.Printf("结果: %d, 耗时(s): %f\n", pool.Count, end.Sub(start).Seconds())
			break
		}
	}
}

```

