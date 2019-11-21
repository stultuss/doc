Node.js 模块
=========================
> 记录日常工作中的一些问题

## 模块机制

> 在理解 Node 模块机制前，需要先理解为什么需要模块化变成，以及模块化编程规范。

### 1. 模块化编程

我们先来回顾下上古时代的 `JavaScript` 是如何编写的，来看一个很简单的例子：

```javascript
// m.js
var f;
function f1() {
	f = 1;
	console.log(f);
}

function f2() {
	f = 2;
	console.log(f);
}

f1(); // 1
f2(); // 2
```

在这个例子里把不同的函数（以及记录状态的变量）简单地放在一起，也就是无模块化，所有的 `js` 代码通过`<script src="m.js"></script>` 的方式加载。这种写法会存在几个很明显的问题，例如：污染全局作用域，单个文件代码冗长，变量名和方法名被覆盖等等。

然后我们我们再看下一个例子：

```javascript
var m = new Object({
  f: 0,
  f1: function () {
    this.f = 1;
	console.log(this.f);
  },
  f2: function () {
    this.f = 2;
	console.log(this.f);
  }
});

console.log(m.f); // 0
m.f1(); // 1
m.f2(); // 2
m.f = 3;
console.log(m.f); // 3
```

我们把模块写成一个对象，所有的模块成员都放到这个对象里面，使用的时候就是调用对象的属性。这样，我们的确解决了全局变量污染的问题，但这样的写法会暴露所有模块成员，内部状态可以被外部改写。例如：

```javascript
m.f = 3;
console.log(m.f); // 3
```

接下来我们再看一下通过外面包一层自执行函数：

```javascript
var m = (function () {
  var f;

  function f1() {
    f = 1;
    console.log(f);
  }

  function f2() {
    f = 2;
    console.log(f);
  }

  return {
    f1: f1,
    f2: f2
  };
})();

m.f1(); // 1
console.log(m.f);  //undefined
```

通过上面的例子，我们在外部只能访问暴露的接口，但无法访问内部私有变量。这就是模块化编程的基础。

### 2. 模块化编程规范

Node.js 是 `CommonJs` 规范的实现。`CommonJS` 是一种模块化编程规范，由于 `CommonJS` 需要同步加载特性，导致 `CommonJS` 规范不适用于浏览器环境，所以诞生了 `AMD` 规范，即"异步模块定义"规范。除此之外，还有玉伯的 `CMD` 规范，即“通用模块定义”。

- `CommonJS` 属于服务端的模块化编程规范。
  - 一个单独的文件就是一个模块，每一个模块都是一个单独的作用域。
  - 输出模块变量的最好方法是使用`module.exports`对象
  - 加载模块使用`require`方法，该方法读取一个文件并执行，返回文件内部的`module.exports`对象
-  `AMD` 和 `CMD` 则属于客户端的模块化编程规范，具体请阅读 [玉伯：AMD 和 CMD 的区别](https://www.zhihu.com/question/20351507/answer/14859415) 。

下面看一个简单的 Node 的模块加载： 

```javascript
// m.js
var f;

function f1() {
    f = 1;
    console.log(f);
}

function f2() {
    f = 2;
    console.log(f);
}

module.exports = {
    f1: f1,
    f2: f2,
};

// index.js
var m = require('./m');

m.f1(); // 1
m.f2(); // 2
console.log(m.f); // undefined
```

接下来，我们通过一个例子来分析上面例子中 `index.js` 的具体执行顺序。

```javascript
function require() {
  var module = { exports: {} };
  ((module, exports) => {
    // m.js
    var f;

    function f1() {
      f = 1;
      console.log(f);
    }

    function f2() {
      f = 2;
      console.log(f);
    }
	
    module.exports = {
      f1: f1,
      f2: f2,
    };
    exports = module.exports
  })(module, module.exports);
  return module.exports;
}

var m = require();

m.f1(); // 1
m.f2(); // 2
console.log(m.f); // undefined 
```

我们可以看到，`require` 方法内加载的 `m.js` 是一个独立环境，其中 `module.exports` 的初始值是一个空的对象，而 `exports` 变量则是 `module.exports` 的引用。require() 返回的是 `module.exports`  而不是 `exports` 。

需要注意的是，这个例子只是告诉我们 require 具体执行了什么，实际上还是有所差别，例如 `m.js` 中实际是无法获取 `index.js` 的变量的。但这个例子中是可以获取外部的变量。

## 常见问题

### 1. ES6 中的 `import`与 `require` 方法有什么区别？

- 执行`import`时，不会去执行模块，而是只生成一个引用，等到真的需要用到时，再到模块里面去取值。
- 通过 `import` 的模块加载无论写在代码的什么位置，都会在代码**执行前**处理，而 `require` 则是根据代码执行**顺序执**行，如果是在一个永否的判断条件中，就不会进行加载。
- 通过 `import` 的模块加载不会缓存模块的运行结果。

### 2. a.js 和 b.js 两个文件互相 require 是否会死循环? 双方是否能导出变量? 如何从设计避免这种问题？

代码如下：

```javascript
// a.js
require('./b');
console.log(h); // Hello
console.log(w); // ReferenceError: w is not defined

// b.js
require('./a');
h = 'Hello';
var w = 'World';
```

不会发生死循环，require 实际是导出 module.exports，也就是空对象。双方可以导出全局变量，即全局变量 `h`，而不能导出局部变量 `w` , `w` 变量的作用域是 `b.js` 的模块。至于为什么能导出全局变量，而不能导出局部变量，则需要了解 `require` 的机制，也就是每个 `js` 的外面实际是包了一层自执行函数，而全局变量还是全局变量。

这个例子较为简单，还有一种复杂情况：如果 `b` 文件中的全局变量 `h` 是一个变动的值，例如时间戳，那 `a` 文件如果每过1秒中去获取一次全局变量 `h` 的值，只能获得相同的值，这是由于 require 在模块加载执行时会缓存结果。

解决方案是直接使用 ES6 的 `import` ，参考资料：[阮一峰：JavaScript循环加载](http://www.ruanyifeng.com/blog/2015/11/circular-dependency.html)

### 3. 如果 a.js 加载了 b.js, 那么在 b 中定义全局变量 `t = 111` 能否在 a 中直接打印出来？ 

能，结论看上题。

### 4. 为什么 Node.js 不给每一个`.js`文件以独立的上下文来避免作用域被污染?

从 Node.js 的 require 原理来看，已经是拥有独立的作用域，实际上还是会出现全局变量污染的问题。