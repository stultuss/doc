
Node.js 其他
=========================
> 记录日常工作中的一些问题

## 把 Callback 接口包装成 Promise 接口

作为 Node.js 的开发者，经常会接触到 ECMAScript 6 标准之前的代码，最典型的就是 Callback Hell，特别是使用 Callback 方式提供的异步回调，通常这类接口有一个 **特征** 或 **规范** ：

**最后一个参数传入一个回调函数，点那个出现异常时，将错误信息作为第一个参数传给回调函数，如果正常，则第一个参数为null，后面的参数为对应其他的值。**

例如：

```javascript
const fs = require('fs');
fs.readFile('foo.json', 'utf8', function(err, res) {
    if (err) {
        console.log(err);
    } else {
        console.log(res);
    }
})
```

自从 Node.js 的 LTS 版本支持 ECMAScript 6 之后，开发者会选择将这些 Callback 接口改写成 Promise 接口，来解决 Callback Hell 的问题。

### 解决方案

那么如何将 Callback 接口包装成 Promise 接口呢？首先，必须确定原来的代码是否符合上面提到的特征或规范？如果是，那么恭喜你，改造会非常方便：

- Node.js v8.0.0 以下环境，使用 bluebird 模块的 promisify 方法。
- Node.js v8.0.0 以上环境，使用内置 util 模块的 promisify 方法。

例子：

```javascript
const fs = require('fs');
const util = require('util'); // 或者引入 bluebird 模块
const readFileAsync = util.promisify(fs.readFile);

readFileAsync('foo.json', 'utf8')
    .then((res) => {
    	console.log(res);
	})
    .catch((err) => {
    	console.log(err);
	})
```

通过 `util.promisify` 将 `fs.readFile` 方法转换成了基于 promise 的异步方法。

### 其他情况

但事实上并不是所有人的代码都会符合规范的，总会有一些奇葩的接口，例如：

```javascript
const fs = require('fs');

exports.demo = (callback, file, character) => {
  fs.readFile(file, character, function (err, res) {
    callback(err, res);
  });
};
```

**像类似 `demo` 这类设计不合理的接口，如果使用 promisify 进行转换，是无法正常工作的。**

对于这种情况，只能手工编写 promisify 方法使其能够正常工作。

```javascript
const promisify = (fn, receiver) => {
  return (...args) => {
    return new Promise((resolve, reject) => {
      fn.apply(receiver, [(err, res) => {
        return err ? reject(err) : resolve(res);
      }, ...args]); // 注意：这里的 ...args 和 callback 的位置是根据实际情况调整的。
    });
  };
};
exports.demoAsync = promisify(demo);
```

## 对 V8 友好的高性能代码

### 隐藏类(Hidden Class)

#### 1. 总是以相同的次序初始化对象

```javascript
function Point(x, y) {
	this.x = x;
	this.y = y;
}

var p1 = new Point(11, 22);
var p2 = new Point(33, 44);
// 这里的 p1 和 p2 拥有共享的隐藏类

p2.z = 55;
// 注意：这时的 p1 和 p2 的隐藏类已经不同了。

// 在我们为p2添加“z”这个成员之前，p1和p2一直共享相同的内部隐藏类——因此V8可以生成一段单独版本的优化汇编码，这段代 码可以同时封装p1和p2的JavaScript代码。派生出这个新的隐藏类还将使编译器无法在Optimized模式执行。我们越避免隐藏类的派生，就会获得越高的性能。
```

#### 2. 永远不要delete对象的某个属性

```javascript
function Point(x, y) {
	this.x = x;
	this.y = y;
}

for (var i = 0; i < 10000; i++) {
	var p1 = new Point(11, 22);
	delete p1.x;
	p1.y++;		
}

// 由于调用了delete，将导致hidden class产生变化，从而使p1.y不能用inline cache直接获取。以上程序在使用了delete之后耗时0.339s，在注释掉delete后只需0.05s。
```

### 非优化(Deoptimized)

#### 1. 单态操作优于多态操作

```javascript
function Point(x, y) {
	return x + y
}

add(1, 2); // add 方法中的 + 操作是单态操作
add("a", "b"); // add 方法中的 + 操作变成了多态操作

// 由于传入的数据类型不同，使得 add 操作无法编译成 Optimized 代码。
```

**谨慎使用 try catch 与 for in** 

- 来自Google I/O 2013的一个演讲：Accele­rating Oz with V8。The oz story的游戏有频繁的GC，游戏的帧率在运行一段时间后不断下降。导致频繁 GC 的疑犯有三个：
  - 1. new 出来的对象没有释放，通常是闭包或集合类的操作导致的。
  - 2. 对象在初始化后改变属性，参考上面的隐藏类例子
  - 3. 某段特别热的代码运行在 Deoptimized 模式下
- 在通过诊断 time 后的结果发现，LazyComplie:*drawSprites 运行在 Optimized 状态，但 UpdateSprites 一直运行在 Deoptimized 模式下。

```javascript
function UpdateSprites(sprites) {
	for (var sprite in sprites) {
		// ....
	}
}

// 因为 for in 下面的代码在 V8 下暂时无法优化，把 for in 内部的代码提出成单独的 function， V8 就可以优化这个 function，这时候，掉帧和GC的问题就立刻解决了。
```

**减少闭包的使用**

- 闭包会使得程序逻辑变的复杂，无法直观的看清除内存是否被释放。尤其注意释放闭包中的大对象。
- 定时器引起的内存泄漏很普遍，并且难以发现，之前在制作发行网站的时候，曾经遇到过，在方法中调用了 `setTimeout`，调用该方法再注销掉后，会发现 `setTimeout` 依然会继续运行。这是因为内部的`setTimeout` 调用会将闭包加到 node.js 事件循环的队列里。

```javascript
var myObj = {
    callMeMaybe: function () {
        var myRef = this;
        var val = setTimeout(function () { 
            console.log('Time is running out!'); 
            myRef.callMeMaybe();
        }, 1000);
    }
};

myObj.callMeMaybe();
// 定时器会不停打印“Time is running out”。

myObj = null;
// myObj对象不会被释放掉，因为内部的myRef对象也指向了myObj
// 定时器仍然会不停打印“Time is running out”。
```

