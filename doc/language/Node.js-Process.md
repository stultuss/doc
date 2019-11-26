Node.js 进程
=========================
> 记录日常工作中的一些问题

## 基本概念

进程（Process）是系统进行资源分配和调度的基本单位。启动一个 node.js 程序就是开启了一个服务进程。线程是操作系统能够进行运算调度的最小单位，包含于进程之内。一个进程可以拥有多个线程。但 Node.js 中的 JavaScript 运行环境是属于单线程的（不代表 Node.js 是单线程的，更详细的描述在事件循环的文档中进行阐述）。在 Node.js 中通过 child_process.fork 的方式开启多个进程，但这种方式下，进程间的通信性能并不是很好，并不推荐这么做。

## process.nextTick

### 1. 事件循环

Node 的事件循环机制是先执行同步任务，再执行异步任务。而异步任务又分为两种：**本轮循环** 和 **次轮循环** 。

Node 中规定 `process.nextTick` 和 `Promise` 的回调函数，是在本轮循环执行完成后执行，而 `setTimeout` ，`setInterval` ，`setImmediate` 的回调函数，则是在次轮循环中执行。

```javascript
setTimeout(() => console.log(1), 0);
setImmediate(() => console.log(2));
process.nextTick(() => console.log(3));
Promise.resolve().then(() => console.log(4));
// 3, 4, 1, 2
```

### 2. 递归调用 `process.nextTick` 会怎么样? 

由于 `process.nextTick` 是插入到每个环节结束之后执行。所以递归调用会阻塞整个事件循环！而 `setTimeout` d的递归则不会阻塞事件循环。

```javascript
function test1() { 
  process.nextTick(() => test());
}

function test2() { 
  setTimeout(() => test(), 0);
}
```

## child_process（子进程）

开发者可以通过 `require` 将 `child_process` 模块加载到代码中。`child_process` 模块有三个方法需要了解：

- **exec**：启动一个子进程来执行命令，调用 bash 来解释命令，所以如果有命令有外部参数，则需要注意被注入的情况。
- **spawn**：更安全的启动一个子进程来执行命令，使用 option 传入各种参数来设置子进程的 `stdin`、`stdout` 等。通过内置的管道来与子进程建立 IPC 通信。
- **fork**：`spawn` 的特殊情况，专门用来产生 worker 或者 worker 池。 返回值是 [ChildProcess ](https://zhuanlan.zhihu.com/p/goog_1177605021)[对象](https://link.zhihu.com/?target=https%3A//nodejs.org/dist/latest-v7.x/docs/api/child_process.html%23child_process_class_childprocess) 可以方便的与子进程交互。

**另外，通过`child_process` 创建的子进程，如果需要和主进程进行通信（IPC），请选择消息队列或者 TCP Socket 或 gRPC 来进行，因为当传输较大数据（1MB）时候，通过 `child_process` 的 send 的速度会慢好几倍**。参考资料：[饿了么前端-Node.js cluster 踩坑小结](https://zhuanlan.zhihu.com/p/27069865)

## Cluster（集群）

Cluster 是常见的 Node.js 利用多核的办法，Cluster 是基于 `child_process.fork` 的，所以多个 worker 之间的通信也是基于内置的那套 IPC，同样会有坑。还有 Cluster 通过加入 `cluster.isMaster` 这个标识，来区分父进程以及子进程，达到类似 POSIX 的 [fork](http://man7.org/linux/man-pages/man2/fork.2.html) 的效果。

## 进程间通信（IPC）

进程间通信（Inter-process communication）就是将一个进程里的消息传递到另外一个进程里。要实现这种传递的方法有很多种，例如：文件（File），信号（Signal），Socket，消息队列（MQ），管道（Pipe），共享内存（Shared memory）等等。

而在 Node.js 中的 IPC 实现，根据 OS 不同而不同，在 Windows 环境中使用的是管道，而在 UNIX 环境中则是使用 UNIX domain socket（详见[官方文档](https://github.com/nodejs/node/blob/bfade5aacd639fbac920647bf1ca4a6fb6df9e0d/doc/api/net.md#ipc-support)）

## 守护进程（daemon）

参考资料：[Linux 守护进程的启动方法](http://www.ruanyifeng.com/blog/2016/02/linux-daemon.html)

## 常见问题 

### 1. `process` 的当前工作目录是什么? 有什么作用?

通过 `process.cwd()` 获得的是当前进程启动时所在的目录。通常用来获取命令行启动的时候的目录，或以该目录为基础获得其相对位置的其他目录。

### 2. 父进程或子进程的死亡是否会影响对方? 什么是孤儿进程？

子进程死亡不会影响父进程，但父进程死亡，那么其管理的子进程都将变为孤儿进程。

### 3. 什么是守护进程？ 如何实现守护进程？

普通的进程在用户退出终端之后就会直接关闭。而通过 `&` 启动则会改成“后台任务”，但会由于用户退出 session 而终止进程。守护进程是不依赖终端的进程，不会因为用户退出终端而停止运行的进程。

通过 `nohup` 和 `&` 可以将普通进程转变为守护进程，但一个合格的守护进程首先必须是稳定的，否则守护进程一旦挂掉，其子进程就会变成孤儿进程。

```bash
nohup node server.js &
```

### 4. 通过 fork 的进程监听同一个端口是否会报端口被占用？

不会，因为当父进程会将服务器，Socket 的句柄通过 IPC 共享给子进程。