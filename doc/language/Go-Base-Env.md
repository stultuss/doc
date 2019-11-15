Golang 基础-环境
=========================
> 学习笔记

## 代理 (Go Proxy)

**Go 1.13 及以上：**

打开终端并执行

```bash
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.io,direct
```

**macOS 或 Linux**

```shell
export GO111MODULE=on
export GOPROXY=https://goproxy.io
```

七牛云提供国内代理 [goproxy.cn](https://github.com/goproxy/goproxy.cn/blob/master/README.zh-CN.md)，缺点：如果库有新的修改，需要一定时间才会刷新。

