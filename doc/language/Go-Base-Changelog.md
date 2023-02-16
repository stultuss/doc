Golang 版本更新
=========================
> 主要记录基于 go 1.14 以上版本的特性更新

## go1.20 [简介](https://go.dev/doc/go1.20)

**这是一个 gap 版本，引入内存优化 area，编译优化 pgo，代码编写 slice 转换 array 减少心智负担，以及多error wrap。**

语言变化：

- 预先声明的[`comparable`](https://go.dev/ref/spec#Type_constraints)约束现在也被普通的[可比较类型](https://go.dev/ref/spec#Comparison_operators)[满足](https://go.dev/ref/spec#Satisfying_a_type_constraint)，例如接口，这将简化通用代码。
- 功能`SliceData`、`String`和`StringData`已添加到包中[`unsafe`](https://go.dev/ref/spec#Package_unsafe)。它们完成了独立于实现的切片和字符串操作的函数集。
- Go 的类型转换规则已扩展为允许 [从 slice 直接转换为 array](https://go.dev/ref/spec#Conversions_from_slice_to_array_or_array_pointer)。

```golang
slice := []int{1, 2, 3, 4, 5}

// 老方法
array1 := *(*[5]int)(slice)

// 新方法：1.20之前报错 cannot convert slice (variable of type []int) to type [5]int
array2 := [5]int(slice)
```

- bytes string 互转

```golang
// 老方法：在1.20之前，bytes/string互转需要了解底层实现，借助 unsafe 代码来实现如下
func OldBytesToString(b []byte) string {
    return *((*string)(unsafe.Pointer(&b)))
}

func OldStringToBytes(s string) []byte {
    stringHeader := (*reflect.StringHeader)(unsafe.Pointer(&s))

    var b []byte
    pbytes := (*reflect.SliceHeader)(unsafe.Pointer(&b)) // 先引用，防止原有string gc
    pbytes.Data = stringHeader.Data
    pbytes.Len = stringHeader.Len
    pbytes.Cap = stringHeader.Len

    return b
}

// 新方法：
func String(ptr *byte, len IntegerType) string：// 根据数据指针和字符长度构造一个新的 string。
func StringData(str string) *byte：// 返回指向该 string 的字节数组的数据指针。
func SliceData(slice []ArbitraryType) *ArbitraryType：//返回该 slice 的数据指针。
```

- [语言规范现在定义了比较](https://go.dev/ref/spec#Comparison_operators)数组元素和结构字段的确切顺序。这阐明了在比较过程中出现恐慌时会发生什么。

工具改进：

- 该[`cover`工具](https://go.dev/testing/coverage)现在可以收集整个程序的覆盖率概况，而不仅仅是单元测试。
- 该[`go`工具](https://go.dev/cmd/go)不再依赖于`$GOROOT/pkg`目录中预编译的标准库包存档，并且它们不再随发行版一起提供，从而减少了下载量。相反，标准库中的包是根据需要构建的，并像其他包一样缓存在构建缓存中。
- 的实施`go test -json`已得到改进，以使其在出现杂散写入时更加健壮`stdout`。
- 和其他与构建相关的命令现在接受一个启用`go build`配置文件引导优化的标志以及一个用于整个程序覆盖率分析的标志。`go install -pgo -cover`
- 该命令现在默认在没有 C 工具链的系统上`go`禁用。`cgo`因此，当 Go 安装在没有 C 编译器的系统上时，它现在将对标准库中的包使用纯 Go 构建，这些包可以选择使用 cgo，而不是使用预分发的包存档（已被删除，如上所述） .
- 该[`vet`工具](https://go.dev/cmd/vet)报告了在并行运行的测试中可能发生的更多循环变量引用错误。

标准库添加：

- 新[`crypto/ecdh`](https://go.dev/pkg/crypto/ecdh)包明确支持 NIST 曲线和 Curve25519 上的椭圆曲线 Diffie-Hellman 密钥交换。
- 新函数[`errors.Join`](https://go.dev/pkg/errors#Join)返回一个包含错误列表的错误，如果错误类型实现了该`Unwrap() []error`方法，则可以再次获取错误列表。
- 新[`http.ResponseController`](https://go.dev/pkg/net/http#ResponseController)类型提供对接口未处理的扩展的按请求功能的访问 [`http.ResponseWriter`](https://go.dev/pkg/net/http#ResponseWriter)。
- 转发代理[`httputil.ReverseProxy`](https://go.dev/pkg/net/http/httputil#ReverseProxy) 包括一个新的`Rewrite`钩子函数，取代了以前的`Director`钩子。
- 新[`context.WithCancelCause`](https://go.dev/pkg/context#WithCancelCause)函数提供了一种方法来取消具有给定错误的上下文。可以通过调用新函数来检索该错误 [`context.Cause`](https://go.dev/pkg/context#Cause)。
- 新[`os/exec.Cmd`](https://go.dev/pkg/os/exec#Cmd)字段[`Cancel`](https://go.dev/pkg/os/exec#Cmd.Cancel) 并指定其关联被取消或进程退出时[`WaitDelay`](https://go.dev/pkg/os/exec#Cmd.WaitDelay)的行为 。`Cmd``Context`

## 提高性能

- 编译器和垃圾收集器的改进减少了内存开销，并将整体 CPU 性能提高了 2%。
- 专门针对编译时间的工作导致构建改进高达 10%。这使构建速度与 Go 1.17 保持一致。

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
