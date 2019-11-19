Golang 基础-协程
=========================
> Go 程（Goroutine）是一个由 Go 程序的环境管理的轻量级线程，创建一个 Go 程意味着并行运行一个 Go 程。

**创建**一个 Go 程很简单：

```go
package main

import (
	"fmt"
)

func main() {
	go func() {
		fmt.Println("Does it work?")
	}()
	// time.Sleep(100 * time.Microsecond) // 等待 100 微秒
	fmt.Println("Exit")
}
```

**思考：**想一想上面的例子 Go 程会打印什么？为什么需要增加 time.Sleep 等待 Go 程运行结束？删掉会发生什么事情？

## 备注

Go 程是 Go 语言的核心之一。在学习 Go 语言的过程中，只要掌握好 Go 程与 Channel 通道的搭配使用，理解运行的原理和执行的顺序，学习 Go 语言就不会很难。