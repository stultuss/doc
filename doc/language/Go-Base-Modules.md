Golang 基础-模块
=========================
> 学习笔记

## vgo

>注意：Go 1.11 及以上版本不再需要额外安装 vgo，使用 go mod 命令替代。

输入命令行安装:

```shell
$ go get golang.org/x/vgo
```

如果安装失败，出现 timeout 错误，说明你被墙了，需要设置代理。

```shell
export all_proxy=http://proxyAddress:port // 注意：必须是 http 协议，而非 socks5 协议
```

如果依旧无法获取资源，则考虑使用GOPROXY​

## modules

创建并进入 go-demo-proj 文件夹，使用 `go mod` 建立 modules

```shell
$ go mod init github.com/niklaus0823/go-demo-proj
go: creating new go.mod: module github.com/niklaus0823/go-demo-proj
```

这是会在项目下生成一个 go.mod 文件，内容为：

```
module github.com/niklaus0823/go-demo-proj

go 1.13
```

## demo

接着约定目录结构以及编写示例代码

```
/
├── main.go
└── pkg
    └── api
        └── getdemo.go
```

1. ./pkg/api/getdemo.go

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
	pkgapi "github.com/niklaus0823/go-demo-proj/pkg/api"
	"net/http"
)

func main() {
	router := gin.Default()

	router.GET("/demo", pkgapi.GetDemo)

	s := &http.Server{
		Addr:    ":8080",
		Handler: router,
	}
	s.ListenAndServe()
}

```

接着运行命令行进行构建并运行

```shell
$ go build
$ go run main.go
```

### 