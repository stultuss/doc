Golang 基础
=========================
> 自 go 1.11 之后，golang 支持了[modules](https://github.com/golang/go/wiki/Modules)，解决了包管理的问题，这是一个重大的变化。由此开始，我才逐渐对 golang 产生兴趣，虽然之前也有零零碎碎写过几个 golang 的工具，但对 golang 都处于观望态度。

## 随机数

你可以使用 `Seed(value)` 函数来提供伪随机数的生成种子，一般情况下都会使用当前时间的纳秒级数字。随机方法 `Math/rand` 必须结合 `Seed` 才可以正常获得随机值，否则只会是同一个值。

## 变量传递

Go 方法和函数默认是按值传递，每次将一个变量作为参数传递，都会创建一个新的变量并将其传递给所调用的函数和方法，副本分配在不同的内存地址，**修改变量数据不会影响原来的变量**。

下面例子表示，update 并没有修改 p1，而是复制了一份 p1 并进行了修改，所以存在 **内存的浪费**。

```go
package main

import (
	"fmt"
)

type Person struct {
	Id   int64  `json:"id"`
	Name string `json:"name"`
}

func update(p Person, name string) Person {
	p.Name = name
  return p
}

func main() {
	p1 := Person{
		Id:   1,
		Name: "person1",
	}

	fmt.Println(p1.Name) // persion1
	p2 := update(p1, "person2")
	fmt.Println(p1.Name) // persion1
	fmt.Println(p2.Name) // persion2
}

```

修改一下例子，将值传递改为引用传递，问题就解决了。

```go
package main

import (
	"fmt"
)

type Person struct {
	Id   int64  `json:"id"`
	Name string `json:"name"`
}

func update(p *Person, name string) {
	p.Name = name
}

func main() {
	p1 := Person{
		Id:   1,
		Name: "person1",
	}

	fmt.Println(p1.Name) // persion1
	update(&p1, "person2")
	fmt.Println(p1.Name) // persion2
}

```

注意点：

1. 变量是一个**大的结构**，建议使用指针传递，避免在内存中复制整个结构。
2. 变量是 map , chan, slice，这三种都属于引用类型。但是 slice 的内存地址可以直接通过 `%p` 打印，不需要 `&` 取地址符转换。
3. 通过在 update 方法里面打印变量的内存地址，可以确认所有的传参都是值传递，都是一个副本。当参数是引用类型的时候，即可修改参数数据。**引用类型和传引用是两个概念**



## 文章

参考文章

- https://github.com/unknwon/the-way-to-go_ZH_CN/
- https://chai2010.cn/advanced-go-programming-book/ch1-basic/ch1-01-genesis.html

并发和并行

- https://www.jianshu.com/p/80f69dad849f

- https://segmentfault.com/a/1190000019661223

- https://blog.csdn.net/lengyuezuixue/article/details/79738573

- https://xueyuanjun.com/post/19910.html

Json

- https://sanyuesha.com/2018/05/07/go-json/

- https://juejin.im/post/5ca2f37ce51d4502a27f0539

面向对象

- https://www.jianshu.com/p/365675fa71b5

错误处理 

- https://studygolang.com/articles/23462?fr=sidebar

监控

- https://xenojoshua.com/2019/03/golang-cpu/

ORM：

- [jinzhu/gorm](https://github.com/jinzhu/gorm)
- [Go-MySQL-Driver > DSN (Data Source Name)](https://github.com/go-sql-driver/mysql#dsn-data-source-name)
- [Question on pooling/concurrency regarding recent documentation change, suggest clarification #1053](https://github.com/jinzhu/gorm/issues/1053#issuecomment-265344516)
- [[Question\] Do you need to close the db connection every request? #1427](https://github.com/jinzhu/gorm/issues/1427#issuecomment-332498453)
- [Best practice #461](https://github.com/go-sql-driver/mysql/issues/461#issuecomment-227008369)
- [使用Go，Gin和Gorm开发简单的CRUD API](https://www.jianshu.com/p/ed13eb3caa4e)

Logger：

- [uber-go/zap](https://github.com/uber-go/zap)
- [gin-contrib/zap](https://github.com/gin-contrib/zap)
- [wantedly/gorm-zap](https://github.com/wantedly/gorm-zap)
- [使用zap配置结构体创建记录器](https://studygolang.com/articles/19388)
- [zap.Config](https://github.com/uber-go/zap/blob/v1.10.0/config.go#L53)
- [zapcore.EncoderConfig](https://github.com/uber-go/zap/blob/v1.10.0/zapcore/encoder.go#L222)
- [zap/example_test.go](https://github.com/uber-go/zap/blob/v1.10.0/example_test.go)
- [zap-examples/src/simple1/main.go](https://github.com/sandipb/zap-examples/blob/master/src/simple1/main.go)
- [hnakamur/go-log-benchmarks](https://github.com/hnakamur/go-log-benchmarks)
- [imkira/go-loggers-bench](https://github.com/imkira/go-loggers-bench)

Cache：

- [bradfitz/gomemcache](https://github.com/bradfitz/gomemcache)

- [Memcached Wiki](https://github.com/memcached/memcached/wiki#memcached)

通道管线：

- [Golang Pipeline](https://xenojoshua.com/2019/05/golang-pipeline/)

单例模式：

- [How singleton pattern works with Golang](https://medium.com/golang-issue/how-singleton-pattern-works-with-golang-2fdd61cd5a7f)

数组切片：

- [Arrays, slices (and strings): The mechanics of ‘append’](https://blog.golang.org/slices)

错误处理：

- [Error handling and Go](https://blog.golang.org/error-handling-and-go)
- [Defer, Panic, and Recover](https://blog.golang.org/defer-panic-and-recover)

类型转换：

- [Go语言 类型断言性能测试](https://blog.csdn.net/erlib/article/details/24197069)

