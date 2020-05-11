# 编码规范

## 前言
在 airebnb 编码规范的基础上修改的编码规范，如果和本规范有冲突部分，以本规范为准。

## 插件配置
> Note: this guide assumes you are using WebStorm.

### 安装
``` bash
npm install -g tslint
npm install -g tslint-config-airbnb
```

tslint.json

``` json
{
  "extends": "tslint-config-airbnb"
}
```

![tslint](https://github.com/stultuss/doc/blob/master/images/language/tslint.png?raw=true)

## 配置 Webstorm Format Tools

1. 强制 Format 双引号为单引号
2. 强制在每行代码以“;”符号结尾

![punctuation](https://github.com/stultuss/doc/blob/master/images/language/punctuation.png?raw=true)

3. 强制缩进 2 格

![tabs&indents](https://github.com/stultuss/doc/blob/master/images/language/tabs&indents.png?raw=true)

4. “{”后和“}”前增加空格

![braces](https://github.com/stultuss/doc/blob/master/images/language/braces.png?raw=true)

## 编码规范

1. var 变量名，文件名, 首字母小写，使用驼峰命名法
> 文件名首字母根据情况变化，例如：DemoClass.ts, lib.ts

``` typescript
// bad
const UserId = 1;
const userid = 1;
const user_id = 1;

// good
const userId = 1;
```

2. class 类名首字母大写，使用驼峰命名法
``` typescript
// bad
class demoClass {
  // codes
}
class democlass {
  // codes
}
class demo_class {
  // codes
}

// good
class DemoClass {
  // codes
}
```

3. function 函数名首字母小写，使用驼峰命名法
``` typescript
// bad
function DemoFunc {
  // codes
}
function demofunc {
  // codes
}
function demo_func {
  // codes
}

// good
function demoFunc {
  // codes
}
```

4. 私有方法与私有属性强制变量前添加下划线“_”
> 该条目与 airbnb 规范冲突

``` typescript
// bad
class demo {
  private id: number;
}

// good
class Demo {
  private _id: number;
}
```

5. SASDN 使用的是 ES7 规范，能使用 async/await 的地方，不可以使用 Promise 方法，更不能使用 promise + async 嵌套。

``` typescript
// bad
function demo {
    return new Promise((resolve) => {
        funcAsynDemo().then((res) => resolve(1));
    })
}

// good
function async demo {
    return await funcAsynDemo();
}
```

6. 如果是第三方提供的方法，使用 function(callback: (err, res)): void 的情况，可以使用系统默认的 util 或者 bluebire 的 promisify 方法进行封装，再使用。

``` typescript
function getDemo(callback: (err, res) => void) {
    callback(null, "succeed!")
}

// bad
function demo {
    return new Promise((resolve) => {
        getDemo((err, res) => {
            resolve(res);
        })
    })
}

// good
function async demo {
    const getDemoAsync = bluebire.promisify(getDemo) as any;
    return await getDemoAsync();
}
```

## 其他规范

### proto 文件

1. proto & message 首字母必须大写。

``` protobuf
// bad
message demo {
    int64 id = 1;
}

// good
message Demo {
    int64 id = 1;
}
```

2. proto & message 的属性必须小写，且如果有2个或2个以上单词组成，则使用“_”连接

``` protobuf
// bad
message demo {
    int64 userId = 1;
}

// good
message Demo {
    int64 user_id = 1;
}

```

## 检查目标

### 代码风格
以 `airbnb 编码规范` 和 `SASDN 编码规范` 为准，请在 CodeReview 之前使用 WebStorm 的 reformat 功能，保证在review中不要在这块出问题。变量名、函数名、类名等命名，也需要注意。

### 设计思想
检查的并不仅仅代码内容，更重要的是在编写代码时候的设计思想。只有设计正确，做出来的东西才有正确可言，只要设计是错的，做出来无论如何都是错的。所以，第一要务首先是检查新添加的功能块的设计问题。

### 文档编写
文档也能保持在统一的风格之下。

### 代码细节
1. 是否在for中套for，多层循环嵌套，能否将部分循环取消
2. 获取类实例的方法，无论是new还是单例，能在循环外拿出来的，就不要放在循环内
3. 使用if判断的时候，能用else if做掉的，不要写多个if判断
4. 使用统一的return节点，不要在一个函数体内使用多处的return，导致阅读上很难判断最后返回了什么

### 算法结构
需要检查每个算法节点，保证系统的性能消耗是在可节省的极限状态。关键词：内存，CPU，I/O

``` typescript
export const getDemoHandler: RpcMiddleware = async (ctx: RpcContext, next: MiddlewareNext) => {
    const call: ServerUnaryCall = ctx.call as ServerUnaryCall;
    const callback: RpcImplCallback = ctx.callback;
    const request = call.request as GetOrderRequest;

    const order = await OrderLogic.getOrder(request); // 所有逻辑都写在 OrderLogic 里，并在此调用获取。
    callback(null, order);

    return Promise.resolve();
};
```