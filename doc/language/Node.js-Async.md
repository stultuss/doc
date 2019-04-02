# Node.js 异步

[TOC]

---

## Promise

### 1. Promise 中 .then 的第二参数与 .catch 有什么区别?

这个问题在日常编码中不太遇到，编写了两个个例子，运行后大概就可以了解了。我们来分析一下这两个例子：

```javascript
let call = new Promise((resolve) => {
  resolve();
});

call
  .then(() => {
    console.log(`then1`);
    new HelloWorld();
  }, () => {
    console.log('error1');
  })
  .then(() => {
    console.log(`then2`);
  }, () => {
    console.log('error2');
  })
  .catch(() => {
    console.log('catch');
  });

// then1
// error2
// ReferenceError: HelloWorld is not defined
```

```javascript
let call = new Promise((resolve) => {
  resolve();
});

call
  .then(() => {
    console.log(`then1`);
  }, () => {
    console.log('error1');
  })
  .then(() => {
    console.log(`then2`);
    new HelloWorld(); // 从 then1 调整到 then2
  }, () => {
    console.log('error2');
  })
  .catch(() => {
    console.log('catch');
  });

// then1
// then2
// catch
```

这两个例子的差别就在于报错的 `new HelloWorld()` 从 then1 调整到 then2，而在 then2 中报错，并没有触发 `error2` 的报错。所以我们可以得到结论：

- `promise.then(onFulfilled, onRejected)` 中，如果 `onFulfilled` 发生异常，`onRejected` 是无法捕获的。
- `promise.then(onFulfilled).catch(onRejected)` 中，如果 `onFulfilled` 发生异常，能在 `.catch` 中被捕获
- `.then` 的 `onRejected` 和 `.catch` 的 `onRejected` 本质上都是错误捕获，但前者是就近异常捕获，而后者是全局异常捕获。

### 2. Promise 的同步/异步

我们都知道用 Promise 封装的代码是同步的，而 then 的执行是异步的，那么我们尝试分析一下，下面例子的执行顺序是怎么样的？

```javascript
setTimeout(() => {
  console.log(1);
}, 0);

new Promise((resolve) => {
  console.log(2);
  for (let i = 0; i < 10000; i++) {
    if (i === 9999) {
      resolve();
    }
  }
  console.log(3);
}).then(() => {
  console.log(4);
});
console.log(5);
```

由于 Promise 封装的内容实际是同步的，所以先打印 2，然后循环10000次，执行 resolve()，并打印 3，Promise 的同步代码运行结束后，执行打印 5，剩下到底是先执行 `setTimeout` 定时器 0 秒并打印 1 呢，还是 Promise 的 then 并打印 4？

1. 同步任务的执行
2. 发出异步请求：将异步操作塞入到 libuv 的事件循环任务队列。
3. 规划定时器生效的时间：参考定时器的原理。
4. 执行`process.nextTick()`等等

通过这个调度顺序，我们就可以得出结论：先执行异步请求，打印 4，再执行定时器生效，打印 1。

## Events（事件）

`Events` 是 Node.js 中一个非常重要的 core 模块, 在 node 中有许多重要的 core API 都是依赖其建立的. 比如 `Stream` 是基于 `Events` 实现的, 而 `fs` ， `net` ， `http` 等模块都依赖 `Stream` 。

### 1. EventEmitter 的 emit 是同步还是异步?

> The EventListener calls all listeners synchronously in the order in which they were registered. This is important to ensure the proper sequencing of events and to avoid race conditions or logic errors.
>
> 翻译：
>
> 事件监听器**按照注册的顺序同步调用所有监听器**。这对于确保事件的正确排序和避免竞态条件或逻辑错误非常重要。

根据官方说明，emit 是同步的，且多个监听器是顺序执行的，我们写一个例子来验证一下：

```javascript
const EventEmitter = require('events');

let emitter = new EventEmitter();

emitter.on('click', () => {
  console.log('Hello');
});

emitter.on('click', () => {
  console.log('World');
});

emitter.emit('click'); 
// Hello
// World
```

### 2. EventEmitter 死循环

如果开发人员编写的事件监听器中触发当前事件，那么就会发生事件死循环，Node 进程会退出。

```javascript
const EventEmitter = require('events');

let emitter = new EventEmitter();

emitter.on('click', () => {
  console.log('Hello');
  emitter.emit('click');
});

emitter.on('click', () => {
  console.log('World');
});

emitter.emit('click'); 
// Hello
// Hello
// Hello
// ...
```

如果事件监听器B定义在事件监听器A中，只有当监听器A被触发后，监听器B才能生效，并且每次触发监听器A，都会重新定义一次监听器B（内存溢出）

```javascript
const EventEmitter = require('events');

let emitter = new EventEmitter();

emitter.on('click', () => {
  console.log('Hello');

  emitter.on('click', () => {
    console.log('World');
  });
});

// Hello
emitter.emit('click');
// Hello
// World
emitter.emit('click');
// Hello
// World
// World
emitter.emit('click');
// Hello
// World
// World
// World
emitter.emit('click');
```

## 阻塞/异步

### 1. 如何判断接口是否异步? 是否只要有回调函数就是异步? 

可以看两个个关键点：

- 是否存在 `setTimeout` ，`Promise` ，`async/await` 等明显的定时器和异步函数。
- 是否存在 I/O 操作，例如文件读取。

### 2. 死循环阻塞会造成什么影响？

> 有这样一个场景, 你在线上使用 `koa` 搭建了一个网站, 这个网站项目中有一个你同事写的接口 A, 而 A 接口中在特殊情况下会变成死循环. 那么首先问题是, 如果触发了这个死循环, 会对网站造成什么影响?

Node.js 是单线程的，只有当代码执行完，才会切入事件循环，如果当死循环阻塞了线程，那么所有到这个线程上的请求都会发生超时。

## 并行/并发

官方解释：

- 并发：以可独立执行的进程集合的方式编程，同时处理(dealing)很多的相同任务
- 并行：以可同时执行的（可能相关的）计算指令方式编程，同时做(doing)很多的不同的任务

我的理解：

- 并发：多个任务A1~A10 同时执行，最终达成结果B1~B10。


- 并行：一个任务A，过程拆分成 A1~A10并同时执行，必须全部完成才能达成最终结果B。

参考资料： [并发不是并行，它更好!](http://www.iteye.com/news/28915) 

## 常见问题

### 1. 如何实现一个 sleep 函数? 

```javascript
function sleep(ms) {
  var start = Date.now(), expire = start + ms;
  while (Date.now() < expire) ;
  return;
}
```

### 2. 如何实现一个限制多个请求并发数量的函数？

这个题目主要是考察对并发和异步的理解

```javascript
function sendRequest(urls, max, callback) {
  const len = urls.length;
  let idx = 0;
  let counter = 0;

  function _request() {
    while (idx < len && max > 0) {
      max--;
      fetch(urls[idx++]).finally(() => {
        max++;
        counter++;
        if (counter === len) {
          return callback();
        } else {
          _request();
        }
      });
    }
  }
  _request();
}
```

