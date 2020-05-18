
Node.js 事件循环
=========================
## 结构

Node.js 的结构大致分三层：

- Node.js 标准库：由 JavaScript 编写，即我们调用的 API，可以在源码的 lib 目录下看到。
- Node bindings：这一层是 JavaScript 与底层 C/C++ 沟通的关键，起到桥的作用，实现在 **node.cc**。
- Node 底层：这一层是 Node.js 能够运行的关键，由 C/C++ 实现，主要由以下几部分组成。
  - v8引擎：JavaScript 虚拟机，它为 JavaScript 提供了在非浏览器端运行的环境。负责把 JavaScript 代码解释成本地二进制代码运行。
  - libuv：类似Windows上的窗口消息机制，它同样也实现了消息循环（Event-Loop）
  - 其他模块：OpenSSL、zlib 等等。

> 通过 Node.js 的结构，我们发现真正执行**系统调用**的其实是 libuv，所以当我们将 I/O 操作的请求传达给 libuv 之后，libuv 开启线程来执行这次 I/O 调用，并在执行完成后，传回给 JavaScript 进行后续处理。

### libuv 

Node.js 底层的 libuv 库实现了事件循环，事件循环通过将操作分配给系统内核来处理，从而使得 Node.js 拥有了异步非阻塞I/O的特性。**事件循环是 Node.js 可以实现异步，非阻塞I/O的基础**。

### 单线程

从 Node.js - libuv 的调用流程可以看出，Node.js 的单线程指的是自身 JavaScript 运行环境的单线程，Node.js 并没有给 JavaScript 执行时创建新线程的能力。实际上最终的执行还是通过 libuv 以及它的事件循环来执行的。

这也就是为什么 JavaScript 一个单线程的语言，能在 Node.js 里面实现异步操作的原因，两者并不冲突。**只有 JavaScript 执行的主线程是单线程的，而内部的 libuv 的执行仍然是多线程的** 

## 事件循环

libuv 使得 Node.js 拥有了事件循环的特性。那么什么是事件循环？

先看两篇文章：

- [Node.js 事件循环一: 浅析](https://github.com/ccforward/cc/issues/47) 
- [阮一峰 - Node 定时器详解](http://www.ruanyifeng.com/blog/2018/02/node-event-loop.html)

上述两篇文章中，Alibaba 的同学用几个详细的例子解释了任务队列，任务，微任务以及事件循环的概念，而阮一峰老师则从定时器的角度去解释事件循环。两篇文章中都对事件循环的初始化进行阐述，但有一个明显的分歧点：

- ccforward 认为事件循环初始化时，是将类似 `setTimeout`，`http.get` 等异步操作发送给另外一个线程，即他认为的 **call-stack** 

> 当我们去调用 setTimeout http.get fs.readFile, Node.js 会把这些定时器、http、IO操作发送给另一个线程以保证V8继续执行我们的代码。

- 阮一峰老师则认为，只有一个主线程，事件循环是在主线程上完成的。

> 有些人以为，除了主线程，还存在一个单独的事件循环线程。不是这样的，只有一个主线程，事件循环是在主线程上完成的。

我详细阅读了阮一峰老师的文档，再结合 Bert Belder（libuv 的开发者）的 [Node 交互的主题演讲](https://www.youtube.com/watch?v=PNa9OMajw9w) 中的内容，确定了阮一峰老师的理解是对的。

演讲中主要通过 Google 的图像搜索展示了不同方式描述的事件循环的图片，但他指出大部分图片描绘都是错误的，其中就包含了 ccforward 对事件循环的错误认识，他讲述的主要的错误集中在以下几点：

**1：在用户代码中，事件循环在单独的线程中运行**

- 误解：用户的 JavaScript 代码运行在主线程上面，而另开一个线程运行事件循环。每次异步操作发生时，主线程将把工作交给事件循环线程，一旦完成，事件循环线程将通知主线程执行回调。

- 结论：只有一个线程执行 JavaScript 代码，事件循环也运行在这个线程上面。回调的执行（在运行的 Node.js 应用程序中被传入、后又被调用的代码都是一个回调）是由事件循环完成地。

**2：异步的所有内容都由线程池处理**

- 误解：异步操作，像操作文件系统，向外发送 HTTP 请求以及与数据库通信等都是由 libuv 提供的线程池处理的。

- 结论：libuv 默认使用四个线程创建一个线程池来完成异步工作。现在的操作系统已经为许多 I/O 任务提供了异步接口（[例子 AIO on Linux](http://man7.org/linux/man-pages/man7/aio.7.html)）。libuv 将优先使用这些异步接口，避免使用线程池。

**3：事件循环类似栈或队列（ccforward 的又一个误解）**

- 误解：事件循环采用先进先出的方式执行异步任务，类似于队列，当一个任务执行完毕后调用对应的回调函数。

- 结论：虽然涉及到类似队列的结构，事件循环并不是采用栈的方式处理任务。事件循环作为一个进程被划分为多个阶段，每个阶段处理一些特定任务，各阶段轮询调度。

### 轮询调度

> 关于事件循环，阮一峰老师的文档里已经描述的非常精确和完整了，这里我只进行简单的归纳和理解。

Node.js 进程启动后，会进行一次事件循环的初始化，但事件循环尚未开始，初始化中包含：

1. 同步任务的执行
2. 发出异步请求：将异步操作塞入到 libuv 的事件循环任务队列。
3. 规划定时器生效的时间：参考定时器的原理。
4. 执行`process.nextTick()`等等

完成初始化后，事件循环开始，每一次事件循环会依次执行六个阶段：

1. **timers 阶段** ： 检查是否有到期的 timers 函数，到期了就执行它的回调。
2. **I/O callbacks 阶段** ：处理异步事件的回调，比如网络I/O，磁盘I/O，当I/O动作 **结束** 后并执行它们的回调，除了以下操作的回调函数。
   - `setTimeout()`和`setInterval()`的回调函数
   - `setImmediate()`的回调函数
   - 用于关闭请求的回调函数，比如`socket.on('close', ...)`
3. **idle, prepare 阶段** ：该阶段是 `libuv` 内部调用，这里可以忽略。
4. **I/O poll 阶段** ：轮询阶段，用于等待还未返回的 I/O 事件，这个阶段是 **选择运行** 的，具体规则如下：
   - poll 队列（未返回的 I/O 事件队列）不为空，事件循环先遍历队列，并同步执行回调。
   - poll 队列为空
     - 如果有  `setImmediate()`，直接结束 poll 阶段，进入 check 阶段
     - 如果没有 `setImmediate()` 
       - 但有设定 timers，并且存在到期的 timers，则会绕回 timers 阶段
       - 如果没有设定 timers，则阻塞在 poll 阶段等待回调被加入 poll 队列。
5. **check 阶段** ：该阶段执行 `setImmediate()` 的回调函数。
6. **close callbacks 阶段** ：该阶段执行关闭请求的回调函数，例如 `socket.on('close', ...)` 的事件就是在这个阶段被触发。

每个阶段都有一个先进先出的回调函数队列。只有一个阶段的回调函数队列清空了，该执行的回调函数都执行了，事件循环才会进入下一个阶段。

![Event-Loop](https://github.com/stultuss/doc/blob/master/images/language/event-loop.jpg?raw=true)

其他参考资料：[JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)

### 监控

我们现在知道 Node.js 上进行的所有事件都将通过事件循环运行，但通过查阅官方文档，发现并没有现成的 API 可以从事件循环中获取运行时的指标，这表示每一种监控工具都会有自己独特的指标来分析出程序整体运行状况和性能。

开源工具：[pebble/event-loop-lag](https://github.com/pebble/event-loop-lag)，使用方法可阅读 [Node.JS Profile 2.1 EventLoop Lag](https://xenojoshua.com/2018/03/node-event-loop/) 的 Lag Monitor 一节。

APM工具：[nodejs-monitoring](https://www.dynatrace.com/technologies/nodejs-monitoring/)
