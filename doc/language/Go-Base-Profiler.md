Golang 基础-测试
=========================
> 学习笔记

## 基准测试

基准测试是一种测试代码性能的方法，主要通过测试CPU和内存的效率问题，来评估被测试代码的性能，进而找到更好的解决方案。编写规则如下：

- 基准测试的代码文件必须以_test.go结尾
- 基准测试的函数必须以Benchmark开头，必须是可导出的
- 基准测试函数必须接受一个指向Benchmark类型的指针作为唯一参数
- 基准测试函数不能有返回值
- `b.ResetTimer`是重置计时器，这样可以避免for循环之前的初始化代码的干扰
- 最后的for循环很重要，被测试的代码要放到循环里
- b.N是基准测试框架提供的，表示循环的次数，因为需要反复调用测试的代码，才可以评估性能

编写测试文件 channel_test.go

```go
package main

import "testing"

func BenchmarkChannelSync(b *testing.B) {
	ch := make(chan int)
	go func() {
		for i := 0; i < b.N; i++ {
			ch <- i
		}
		close(ch)
	}()

	for range ch {
		<-ch
	}
}

func BenchmarkChannelBuffered(b *testing.B) {
	ch := make(chan int, 128)
	go func() {
		for i := 0; i < b.N; i++ {
			ch <- i
		}
		close(ch)
	}()

	for range ch {
		<-ch
	}
}
```

运行基准测试:

```bash
$ go test -bench=. -run=none
```

结果如下：

```bash
goos: darwin
goarch: amd64
pkg: github.com/demo
BenchmarkChannelSync-8           6163188               195 ns/op
BenchmarkChannelBuffered-8      19745815                61 ns/op
PASS
ok      github.com/demo  2.976s
```

通过上面的测试结果，可以发现使用缓冲区的通道，一次操作需要 60 纳秒，单位时间内操作了 19745815 次。

## 性能测试

Go 语言内置工具 tool 继承了 pprof，以便进行性能测试找出瓶颈，测试方法和基准测试类似，案例代码沿用上面的。

生成分析数据：

```bash
$ go test -bench=. -run=none -cpuprofile cpu.out -memprofile mem.out
```

读取分析数据：

```shell
$ go tool pprof -text cpu.out # 读取 CPU 的分析数据

Type: cpu
Time: Nov 19, 2019 at 2:47pm (CST)
Duration: 2.84s, Total samples = 3.83s (134.77%)
Showing nodes accounting for 3.82s, 99.74% of 3.83s total
Dropped 4 nodes (cum <= 0.02s)
      flat  flat%   sum%        cum   cum%
     1.99s 51.96% 51.96%      1.99s 51.96%  runtime.pthread_cond_wait
     1.13s 29.50% 81.46%      1.13s 29.50%  runtime.pthread_cond_signal
     0.54s 14.10% 95.56%      0.54s 14.10%  runtime.usleep
     0.11s  2.87% 98.43%      0.11s  2.87%  runtime.(*waitq).dequeue
     0.03s  0.78% 99.22%      0.03s  0.78%  runtime.pthread_mutex_lock
     0.01s  0.26% 99.48%         2s 52.22%  runtime.notesleep
     0.01s  0.26% 99.74%      0.55s 14.36%  runtime.runqgrab
         0     0% 99.74%      0.05s  1.31%  github.com/demo-golang.BenchmarkChannelBuffered
         0     0% 99.74%      0.04s  1.04%  github.com/demo-golang.BenchmarkChannelBuffered.func1
         0     0% 99.74%      0.02s  0.52%  github.com/demo-golang.BenchmarkChannelSync.func1
         0     0% 99.74%      0.06s  1.57%  runtime.chanrecv
         0     0% 99.74%      0.05s  1.31%  runtime.chanrecv2
         0     0% 99.74%      0.06s  1.57%  runtime.chansend
         0     0% 99.74%      0.06s  1.57%  runtime.chansend1
         0     0% 99.74%      2.55s 66.58%  runtime.findrunnable
         0     0% 99.74%      1.03s 26.89%  runtime.goready.func1
         0     0% 99.74%      2.68s 69.97%  runtime.mcall
         0     0% 99.74%      1.03s 26.89%  runtime.mstart
         0     0% 99.74%      1.16s 30.29%  runtime.notewakeup
         0     0% 99.74%      2.68s 69.97%  runtime.park_m
         0     0% 99.74%      1.03s 26.89%  runtime.ready
         0     0% 99.74%      0.13s  3.39%  runtime.resetspinning
         0     0% 99.74%      0.55s 14.36%  runtime.runqsteal
         0     0% 99.74%      2.68s 69.97%  runtime.schedule
         0     0% 99.74%      1.99s 51.96%  runtime.semasleep
         0     0% 99.74%      1.16s 30.29%  runtime.semawakeup
         0     0% 99.74%      1.16s 30.29%  runtime.startm
         0     0% 99.74%         2s 52.22%  runtime.stopm
         0     0% 99.74%      1.03s 26.89%  runtime.systemstack
         0     0% 99.74%      1.16s 30.29%  runtime.wakep
         0     0% 99.74%      0.06s  1.57%  testing.(*B).launch
         0     0% 99.74%      0.06s  1.57%  testing.(*B).runN
         
$ go tool pprof -text mem.out # 读取内存的分析数据

Type: alloc_space
Time: Nov 19, 2019 at 2:49pm (CST)
Showing nodes accounting for 512.19kB, 100% of 512.19kB total
      flat  flat%   sum%        cum   cum%
  512.19kB   100%   100%   512.19kB   100%  runtime.malg
         0     0%   100%   512.19kB   100%  runtime.mstart
         0     0%   100%   512.19kB   100%  runtime.newproc.func1
         0     0%   100%   512.19kB   100%  runtime.newproc1
         0     0%   100%   512.19kB   100%  runtime.systemstack
```

交互模式

```shell
$ go tool pprof demo.test cpu.out

File: demo.test
Type: cpu
Time: Nov 19, 2019 at 2:49pm (CST)
Duration: 2.84s, Total samples = 3.81s (133.94%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) 

# top、top10: 显示前几条信息
# web: 以 svg 文件显示，需要安装 [graphviz](http://www.graphviz.org/download/)
```

