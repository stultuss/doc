Golang 基础
=========================
> 自 go 1.11 之后，golang 支持了[modules](https://github.com/golang/go/wiki/Modules)，解决了包管理的问题，这是一个重大的变化。Go modules 是 Go 语言的依赖解决方案，发布于 Go1.11，成长于 Go1.12，丰富于 Go1.13，正式于 Go1.14 推荐在生产上使用。正式 “淘汰” 之前的 GOPATH 的使用模式。

## 基础配置

```bash
$ vim ~/.bashrc

export GOPATH="/Users/proj/";
export GOENV="/Users/proj/.env"

$ source ~/.bashrc
```

## 环境变量

### Go Modules

> Go Modules 的前身是 vgo，在 Go 1.11 版本之后就不再需要额外安装 vgo，使用 go mod 命令替代

在 Go modules 中，我们能够使用如下命令进行操作：

| 命令            | 作用                             |
| --------------- | -------------------------------- |
| go mod init     | 生成 go.mod 文件                 |
| go mod download | 下载 go.mod 文件中指明的所有依赖 |
| go mod tidy     | 整理现有的依赖                   |
| go mod graph    | 查看现有的依赖结构               |
| go mod edit     | 编辑 go.mod 文件                 |
| go mod vendor   | 导出项目所有的依赖到vendor目录   |
| go mod verify   | 校验一个模块是否被篡改过         |
| go mod why      | 查看为什么需要依赖某模块         |

Go语言提供了 GO111MODULE 这个环境变量来作为 Go modules 的开关，其允许设置以下参数：

```bash
# Go 1.13 及以上
$ go env -w GO111MODULE=on

# Mac & Linux
$ vim ~/.bashrc
export GO111MODULE=on # auto | on | off
$ source ~/.bashrc
```

> 当 modules 功能启用时，依赖包的存放位置变更为`$GOPATH/pkg`，允许同一个package多个版本并存，且多个项目可以共享缓存的 module。

### Go Proxy

这个环境变量主要是用于设置 Go 模块代理（Go module proxy），其作用是用于使 Go 在后续拉取模块版本时能够脱离传统的 VCS 方式，直接通过镜像站点来快速拉取。

```bash
# Go 1.13 及以上
$ go env -w GOPROXY=https://goproxy.io,direct

# Mac & Linux
$ vim ~/.bashrc
export GOPROXY=https://goproxy.io,direct
$ source ~/.bashrc
```

七牛云提供国内代理 [goproxy.cn](https://github.com/goproxy/goproxy.cn/blob/master/README.zh-CN.md)，缺点：如果库有新的修改，需要一定时间才会刷新。

##  初始化项目

创建并进入 go-demo 文件夹，使用 `go mod` 建立 modules

```bash
$ go mod init github.com/stultuss/go-demo

go: creating new go.mod: module github.com/stultuss/go-demo
```

这是会在项目下生成一个 go.mod 文件，内容为：

```
module github.com/stultuss/go-demo

go 1.14
```

接着约定目录结构以及编写示例代码

```
/
├── main.go
└── api
    └── getdemo.go
```

1. ./api/getdemo.go

```go
package api

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func GetDemo(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H{
		"code":    0,
		"message": "success",
	})
}
```

2. ./main.go

```go
package main

import (
	"github.com/gin-gonic/gin"
	pkgapi "github.com/stultuss/go-demo/api"
	"net/http"
)

func main() {
	router := gin.Default()

	router.GET("/demo", pkgapi.GetDemo)

	s := &http.Server{
		Addr:    ":8080",
		Handler: router,
	}
	_ = s.ListenAndServe()
}

```

接着运行命令行进行构建并运行

```bash
$ go build
$ go run main.go
# or
$ ./go-demo
```

## [编码风格](https://www.bookstack.cn/read/go-code-convention/zh-CN-README.md)

### **1、包命名：package**

保持package的名字和目录保持一致，尽量采取有意义的包名，简短，有意义，尽量和标准库不要冲突。包名应该为**小写**单词，不要使用下划线或者混合大小写。

```go
package demo
// or
package main
```

### **2、 文件命名**

尽量采取有意义的文件名，简短，有意义，必须为 **小写** 单词。如非必要请勿使用 **下划线** 分隔各个单词。

```bash
# 平台区分
bolt_windows.go

# 版本区分
trap_windows_1.4.go

# 测试单元
trap_windows_1.4_test.go

# 类型区分
trap_windows_amd64.go
```

### **3、 结构体命名**

- 采用驼峰命名法，首字母根据访问控制大写或者小写
- struct 申明和初始化格式采用多行，例如下面：

```go
// 多行申明
type User struct{
    Username  string
    Email     string
}

// 多行初始化
u := User{
    Username: "astaxie",
    Email:    "astaxie@gmail.com",
}
```

### **4、 接口命名**

- 命名规则基本和上面的结构体类型
- 单个函数的结构名以 “er” 作为后缀，例如 Reader , Writer 。

```go
type Reader interface {
  Read(p []byte) (n int, err error)
}
```

### **5、变量命名**

- 和结构体类似，变量名称一般遵循驼峰法，首字母根据访问控制原则大写或者小写，但遇到特有名词时，需要遵循以下规则：
- 如果变量为私有，且特有名词为首个单词，则使用小写，如 apiClient
- 其它情况都应当使用该名词原有的写法，如 APIClient、repoID、UserID
- 错误示例：UrlArray，应该写成 urlArray 或者 URLArray
- 若变量类型为 bool 类型，则名称应以 Has, Is, Can 或 Allow 开头

```go
var isExist bool
var hasConflict bool
var canManage bool
var allowGitHook bool
```

### **6、常量命名**

常量均需使用全部大写字母组成，并使用下划线分词

```go
const APP_VER = "1.0"
```

如果是枚举类型的常量，需要先创建相应类型：

```go
type Scheme string

const (
    HTTP  Scheme = "http"
    HTTPS Scheme = "https"
)

```

### **7、 关键字**

下面的列表显示了Go中的保留字。这些保留字不能用作常量或变量或任何其他标识符名称。

| break    | default     | func   | interface |
| -------- | ----------- | ------ | --------- |
| case     | defer       | go     | map       |
| chan     | else        | goto   | package   |
| const    | fallthrough | if     | range     |
| continue | for         | import | return    |



https://www.bookstack.cn/read/go-code-convention/zh-CN-naming_rules.md

https://www.lagou.com/lgeduarticle/106838.html

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

