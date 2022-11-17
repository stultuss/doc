Golang 版本更新
=========================
> 主要记录基于 go 1.14 以上版本的特性更新

## go1.20 前瞻

>  实现完全版范性

## go1.19 [简介](https://go.dev/doc/go1.19)

新特性：

- 无

其他信息：

- 修订内存模型与其他语言一致 （The Go Memory Model）
- 开发分支进入 freeze 阶段

## go1.18 [简介](https://go.dev/doc/go1.18)

**go 1.18 是一个重要版本。**

新特性：

- 初步支持泛型，且1.19版本中进行bug修复，预计1.20会全面支持范型。

  > 有三个使用泛型的实验包可能有用。这些包在 x/exp 仓库中；他们的 API 不在 Go 1 保证范围内，并且可能会随着我们获得更多泛型经验而改变。
  >
  > - [`golang.org/x/exp/constraints`](https://pkg.go.dev/golang.org/x/exp/constraints)
  >
  >   对通用代码有用的约束，例如 [`constraints.Ordered`](https://pkg.go.dev/golang.org/x/exp/constraints#Ordered).
  >
  > - [`golang.org/x/exp/slices`](https://pkg.go.dev/golang.org/x/exp/slices)
  >
  >   对任何元素类型的切片进行操作的通用函数的集合。
  >
  > - [`golang.org/x/exp/maps`](https://pkg.go.dev/golang.org/x/exp/maps)
  >
  >   对任何键或元素类型的映射进行操作的通用函数的集合。

- 模糊测试（fuzzing）：模糊测试会消耗大量内存，并且可能会影响机器运行时的性能，运行时会将扩展测试覆盖率的值写入模糊缓存目录 $GOCACHE/fuzz。

- 工作区（workspace）：增加 go work 命令：支持“工作区”模式。

## go1.17 [简介](https://go.dev/doc/go1.17)

新特性：

- 无

改进：

- 从切片刀数组指针的转换：s类型表达式[]T现在可以转换为数组指针类型 *[N]T。如果a是此类转换的结果，则范围内的相应索引引用相同的基础元素：&a[i] == &s[i] for 0 <= i < N。如果len(s)小于，则转换会发生混乱。
- unsafe.Add
- unsafe.Slice

其他信息：

- 需要 macOS 10.13 High Sierra 或更高版本

- go module 不再使用完整 module 依赖图，而是引入 pruned module graph。、
- time 包增加 Time 对象的 GoString 方法。

## go1.16 [简介](https://go.dev/doc/go1.16)

**go 1.16 是一个重要版本。**

新特性：

- 无

其他信息：

- go 1.16 是 macOS 10.12 Sierra 上运行的最后一个版本，支持 Apple Silicon M1。

- 核心库增加 embed：支持静态资源嵌入。
- 默认开启 go module：即 GO111MODULE 环境变量默认为 on，需要切换到以前，需要设置为 auto。

## go1.15 [简介](https://go.dev/doc/go1.15)

Go 1.15 包括[对链接器的实质性改进](https://go.dev/doc/go1.15#linker)，改进[了高核心数下小对象的分配](https://go.dev/doc/go1.15#runtime)，并弃用[了 X.509 CommonName](https://go.dev/doc/go1.15#commonname)。 `GOPROXY`现在支持跳过返回错误的代理。

新特性：

- 无

其他信息：

- 核心库增加 time/tzdata 包

## go1.14 [简介](https://go.dev/doc/go1.14)

**go 1.14 是一个重要版本。**

新特性：

- Go Module已经具备生产环境中使用条件了，我们鼓励所有用户迁移到Go Module进行依赖项管理。
- Go 1.14 现在允许嵌入具有**重叠方法集**的接口：来自嵌入式接口的方法可能具有与（嵌入）接口中已有的方法相同的名称和相同的签名。这解决了菱形的嵌入图通常（但不是完全）发生的问题。接口中显式声明的方法必须像以前一样保持唯一。
- defer 性能提升，**使用 defer 关闭 channel 几乎 0 开销**
- goroutine 支持异步抢占，（之前需要goroutine主动让出 CPU资源，存在非常严重的调度问题）
- time.Timer 定时器性能得到“巨幅”提升

其他信息：

- go mod 和 go test 工具完善
