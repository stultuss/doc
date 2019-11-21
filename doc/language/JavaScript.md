JavaScript 基础问题
=========================

> 记录日常工作中的一些问题

## 类型判断

### 1. undefined，null

```javascript
console.log(undefined); // undefined
console.log(null); // null

console.log(typeof undefined); // undefined
console.log(typeof null); // object

console.log(undefined == null); // 值相等 true
console.log(undefined === null); // 类型不相等 false
```

### 2. [lodash](https://github.com/lodash/lodash) 基础类型解析

```javascript
// isNull.js
function isNull(value) {
  return value === null
}
// isUndefined.js
function isUndefined(value) {
  return value === undefined
}
// isBoolean.js
function isBoolean(value) {
  return value === true || value === false
}
// isNumber.js
function isNumber(value) {
  return typeof value === 'number';
}
// isString.js
function isString(value) {
  return typeof value === 'string'
}
// isObject.js
function isObject(value) {
  return value !== null && (typeof value === 'object' || typeof value === 'function')
}
// isLength.js
function isLength(value) {
  return typeof value == 'number' && value > -1 && value % 1 == 0 && value <= 9007199254740991
}
// isArray.js
function isArray(value) {
  return value !== null && typeof isObject(value) && isLength(value.length)
}
```

### 3. **`==` 和 `===` 的区别**

> 首先定一个基调：`==` 在 JavaScript 中是一种非常糟糕的运算符。

我们如果有其他语言的基础，那么我们会很自然而然的认为：

- `==` 是判断值相等的运算符。
- `===` 是判断值和类型相等的运算符。

这样简单的去理解，其实也不算错，但是通过上面的 `undefind == null ` 的例子，我们就应该需要引起警觉了，或许 `==` 并没有想象中的那么简单，来看一个例子。

```javascript
console.log([1] == 1); // true
console.log([1] == [1]); // false
```

运行结果，颠覆了以往的概念。通过查阅 MDN 的[文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators)，我们得到一个概念：

> 相等(==)
>
> 比较操作符会 **为两个不同类型的操作数转换类型**，然后进行严格比较。当两个操作数都是对象时，JavaScript会比较其内部引用，当且仅当他们的引用指向内存中的相同对象（区域）时才相等，即他们在栈内存中的引用地址相同。

上面的概念中存在三个判断：

- 情况1：相同类型的操作数，并且是基础类型，则进行严格比较（值，类型全部相等）
- 情况2：相同类型的操作数，如果是操作数是对象，则比较引用地址是否相同。
- 情况3：不同类型的操作数，会先转换类型。

所以上面的例子适用情况如下：

```javascript
console.log([1] == 1); // 适用情况3，经过类型转换 [1] 被转换成了 1，具体过程后面再解析。
console.log([1] == [1]); // 适用情况2，一前一后创建了两个对象，并且这两个对象的内存地址不一样
```

那么，`[1]` 是如何被类型转换成为 `1` 的？

参考资料：[一张图彻底搞懂JavaScript的==运算](https://zhuanlan.zhihu.com/p/21650547)

![==](https://github.com/stultuss/doc/blob/master/images/language/==.jpg?raw=true)

看完这个资料，我们基本可以得出以下结论：

- 在 `[1]` 和 `1` 进行比较的时候，会先使用 `Number` 将 `[1]` 对象和 `1` 转成 `number` 类型，再使用 `valueOf` 方法转成基础类型进行比较，即`new Number([1]).valueOf() === new Number(1).valueOf()`
- 而 `Number` 的构造函数实际是接收一个 `string` 类型，而 `[1]` 是一个对象，这边实际是做了两个操作。
  - 同样先使用 `[1].valueOf()` 将对象转成基础类型，但由于数组的 `valueOf` 是自身，不是基础类型。
  - 然后再使用 `toString` 将 `[1]` 转成基础类型 `string` ，即  `[1].toString()`
  - 所以最后得到 `[1].toString() === "1"` 
- 那么很显然，最终执行的是 `new Number("1").valueOf() === new Number(1).valueOf()` ，所以返回 true

## 作用域（scope）

### 1. `let` ，`var`，`const` 的区别

- 使用`var`声明的变量，其作用域为该语句所在的函数内，且存在 **变量提升** 现象。
  - 可以重复声明
  - 通过 `var` 声明的变量没有块级作用域，但有函数作用域。
  - 坑很多，最终 ECMA 协会决定不填坑了，用 `let` 和 `const` 代替 `var`
- 使用`let`声明的变量，其作用域为该语句所在的代码块内，不存在 **变量提升** 。
  - 不能重复声明
- 使用`const`声明的是常量，在后面出现的代码中不能再修改该常量的值。
  - 不能重复声明

```javascript
// var，let 的作用域区别
if (true) {
  var a = 'a';
  let b = 'b';
}
console.log(a); // a
console.log(b); // ReferenceError: b is not defined
```

### 2. 静态作用域

b与b函数中的a函数，两者仅仅是调用关系，因此对于作用域链上两者没有任何关系，虽然是进入b函数后调用a，但其实就是 window.a()

```javascript
var x = 10;

function a() {
  console.log(x);
}

function b() {
  var x = 20;
  a();
}

a(); // 10
b(); // 10
```

### 3. 变量提升

说明：**变量提升** 意味着变量和函数的声明会在物理层面移动到代码的最前面，但这么说并不准确。实际上变量和函数声明在代码里的位置是不会动的，而是在编译阶段（初始化）被放入内存中。

```javascript
// 正常声明过程：先声明函数，再调用函数
function foo1(txt) {
  console.log("Hello " + txt);
}
foo1("World"); // Hello World

// 变量提升过程：先调用函数，再声明函数
foo2("Mars"); // Goodbye Mars
function foo2(txt) {
    console.log("Goodbye " + txt); 
}
```

再来看一个变量提升的陷阱，执行下面代码，我们会发现最后打印出来的是 undefined

```javascript
// 正常声明过程
var v = 'Hello World';
(function () {
    console.log(v); 
})(); // Hello World

// 变量提升过程：变量 v 的声明提升了，但赋值没有。
(function () {
  console.log(v);
  var v = 'Goodbye Mars';
})(); // undefined
```

通过前面的阅读变量提升的说明，我们可以了解到变量提升只提升变量的声明，而不会把赋值也提升上来（实际上也覆盖了作用域）。

```javascript
var v;
v = 'Hello World';
(function () {
  var v;
  console.log(v);
  v = 'I love you';
})(); // undefined
```

 把代码按执行顺序重构一遍后，我们可以很明显的看到 v 被重新声明了，所以打印 v 会得到 undefined。

### 4. 执行上下文

> 执行上下文堆栈在平常写代码中，基本不会用到，但基础理论还是需要了解一下。

JavaScript 解释器实现了一个执行上下文堆栈（ECStack），并且总是在栈顶执行上下文中的代码。当 JavaScript 解释器初始化执行代码时，它会首先默认压入一个全局执行上下文到栈中。在此基础上任何一次函数调用都将压入一个新的上下文到栈中，函数执行结束后，该执行上下文从栈中弹出。（先进后出）

每个执行上下文，都有三个重要属性：

- 变量对象(Variable object，VO)

- 作用域链(Scope chain)
- this


分享一个很绕的面试题：

```javascript
// 比较下面两段代码 A 和 B，试述两段代码在执行上下文堆栈的变化和作用域上的不同。
// A--------------------------
var scope = 'global scope';

function checkscope() {
  var scope = 'local scope';

  function f() {
    return scope;
  }

  return f();
}

checkscope();

// B---------------------------
var scope = 'global scope';

function checkscope() {
  var scope = 'local scope';

  function f() {
    return scope;
  }

  return f;
}

checkscope()();
```

### 5. 闭包

> 闭包是 JavaScript 语言的一个特点，在开发过程中，会有一些特殊的场景必须使用到闭包。以下内容部分出自[《阮一峰 - 学习Javascript闭包（Closure）》](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)

MDN 对闭包的定义：闭包是指那些能够访问自由变量的函数，换言之所有 JavaScript 函数都是闭包。但是从实践角度上，符合下面定义的函数才算是闭包：

- 定义在一个函数内部的函数。
  - **解读** ：在 JavaScript 语言中，只有函数内部的子函数才能读取局部变量
- 能够读取其他函数内部变量的函数。
  - **解读** ：如果了解作用域，那么我们可以知道 JavaScript 允许函数内部读取全局变量，但不允许函数外部读取函数内的局部变量。但是我们可以通过闭包的方法，就可以读取某个函数内的局部变量。
- 即使创建它的上下文已经销毁，它仍然存在
  - **解读**：有内存泄漏隐患。

我们来看一个例子：

```javascript
var scope = 'global scope';

function checkScope() {
  var scope = 'local scope';

  function f() {
    return scope;
  }

  return f;
}

var foo = checkscope();
console.log(foo()); // local scope
```

其中的 `f` 函数符合上面的定义，所以 `f` 函数就是闭包。

闭包还有一个作用是将让函数内部的变量保持在内存中，我们再来看一个例子：

```javascript
function f1() {

  var n = 999;

  nAdd = function () {
    n += 1;
  };

  function f2() {
    console.log(n);
  }

  return f2;

}

var result = f1();

result(); // 999

nAdd();

result(); // 1000

```

其中 `f2` 和 `nAdd` 函数是闭包，并且 `nAdd` 是全局变量，并定义了一个匿名函数。从这个例子中可以看到 `f1` 中的局部变量 n 一直保存在内存中，并没有因为 `f1()` 的调用而清除。

**解释**：`f1` 是 `f2` 的父函数，而通过`var result = f1();`将 `f2` 赋值给了一个全局变量，导致 `f2` 使用在内存中，所以 `f1` 也始终在内存中，不会在调用结束后，被 GC 回收。

**注意事项** ：

1. 闭包会将函数中的变量全部保存在内存中，内存消耗很大，所以不能滥用闭包，甚至导致内存泄漏。
2. 闭包会在父函数外部，改变父函数内部变量的值。

**用途** ：

JavaScript 中没有私有成员的概念，函数内部所有变量都是私有变量，而我们可以通过利用闭包的特点，将私有变量使用闭包进行封装，允许外部访问私有变量，需要注意的是，需要严格控制篡改私有变量，否则会有内存溢出的风险。

### 6. 值传递 & 引用传递

简单一句话概括：**对象是引用传递，基础类型是值传递** , 但基础类型可以通过包装就可以使用引用传递了。

前面讲解数据类型的时候，已经在分析  “==” 和 “===” 的区别时了解过了：

> 如果是操作数是对象，则比较引用地址是否相同

```javascript
console.log([1] == [1]); // false
console.log([1] === [1]); // false
```

反过来思考： JavaScript 并没有类似 C 指针概念，即传统意义上的引用传递（指针传递），那么是否可以理解 JavaScript 的引用传递实际上是传递的 **引用地址** 所以还是 **值传递** ，传递的值是引用地址。

前不久看到一篇讲解 Java 的值传递，引用传递的文章，和 JavaScript 比较类似， [《Is Java “pass-by-reference” or “pass-by-value”?》](https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value)

### 7. 内存释放

引用类型是在没有引用之后, 通过 v8 的 GC 自动回收, 值类型如果是处于闭包的情况下，要等闭包没有引用才会被 GC 回收，非闭包的情况下等待 v8 的新生代 (new space) 切换的时候回收。

判断几个例子，判断内存是否爆掉：

```javascript
let arr = [];
while(true)
  arr.push(1);

// 会爆掉，arr 对象未被释放，并且内存不停增长。
```

```javascript
let arr = [];
while(true)
  arr.push();

// 不会爆掉，arr 对象未被释放，但内存没有增长。
```

```javascript
let arr = [];
while(true)
  arr.push(new Buffer(1000));
  
// 会爆掉，arr 对象未被释放，并且因为 Buffer 的大小小于8k，会先检查内存池，所以内存实际没有增长
```

### 8. ES6 新特性

- 默认参数

```typescript
function helloWorld(hello = 'Hello') {
  console.log(hello + ' World!');
}

helloWorld(); // Hello World!
```

- 字符串模板

```typescript
let hello = 'Hello';
console.log(`${hello} World!`); // Hello World!
```

- 拆包表达式

```typescript
let helloWorld = ['Hello', 'World'];
let [h, w] = helloWorld;
console.log(`${h} ${w}!`); // Hello World!
```

- 对象表达式更新

```typescript
let helloWorld = {
  h: 'Hello',
  w: 'World',
  print: () => {
    console.log(`${helloWorld.h} ${helloWorld.w}!`);
  }
};
helloWorld.print(); // Hello World!
```

- 箭头函数

```typescript
let helloWorld = () => {
    console.log(`${helloWorld.h} ${helloWorld.w}!`);
};
helloWorld(); // Hello World!
```

- Promise （ES7后，使用 `async / await`）

```typescript
let helloWorld = () => {
  return new Promise((resolve) => {
    resolve({
      h: 'Hello',
      w: 'World',
    });
  });
};

helloWorld().then((r) => {
  console.log(`${r.h} ${r.w}!`); // Hello World!
});
```

- 类

```typescript
class HelloWorld {
  constructor() {
    console.log(`Hello World!`);
  }
}

new HelloWorld(); // Hello World!
```

- 模块化

```typescript
// a.js
module.exports = {
  h: 'Hello',
  w: 'World'
};

// b.js
let {h, w} = require('./node-metrics/index');
console.log(`${h} ${w}!`); // Hello World!
```

## 常见问题

### 1. 什么类型是引用传递, 什么类型是值传递? 如何将值类型的变量以引用的方式传递？

**对象是引用传递，基础类型是值传递** , 但基础类型可以通过包装就可以使用引用传递了。例如，把基础类型包装到一个对象里。

### 2. 浮点数陷阱，为什么 0.1 + 0.2 === 0.3 不是 true ? 在不知道浮点数位数时应该怎样判断两个浮点数之和与第三数是否相等？

因为 JavaScript 的浮点数运算缺陷。使用 `toPrecision` 或 `toFixed` 组合解决。
- `toPrecision` 是处理精度，精度是从左至右第一个不为0的数开始数起。
- `toFixed` 是小数点后指定位数取整，从小数点开始数起。

```typescript
function strip(num, precision = 12) {
    return +parseFloat(num.toFixed(precision));
}
console.log(strip(0.1 + 0.2) === strip(0.3));
```

参考资料：[JavaScript 浮点数陷阱及解法](https://github.com/camsong/blog/issues/9)，推荐库：https://github.com/nefe/number-precision

### 3. `const` 定义的 Array 中间元素能否被修改? 如果可以, 那 `const` 修饰对象的意义是？

可以修改，`const` 只保证变量指向的内存地址不可修改，而不是变量的值不可修改。

### 4. 不同类型以及不同环境下变量的内存都是何时释放？

简单理解的理解是，基础类型是存放在栈内存中的，引用类型存放在堆内存中。当一个方法执行，会建立自己的内存栈，当方法执行结束后内存栈销毁。而堆内存则是可以反复利用的，当堆内存不再被任何引用或变量引用时，下一次执行 GC 会将其回收。