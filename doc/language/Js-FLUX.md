
FLUX 学习资料
=========================

> 文字参考来源于**阮一峰**的《[FLUX 架构入门教程](http://www.ruanyifeng.com/blog/2016/01/flux.html)》。

## 关于 FLUX

### 1. 什么是 FLUX 

简单的说，FLUX 是一个架构思想，专门解决软件的结构问题，它和 MVC 架构是同一类东西，但是更加 [简单和清晰](http://www.infoq.com/cn/news/2014/05/facebook-mvc-flux)

* MVC 架构运用在大规模应用的环境中，由于需要不断的增加新的特性与功能。
* 系统的复杂性就会成级数增长，进而让代码变得 **脆弱和不可预测** 。
* 对于新接触这类代码库的开发人员来说，理解与培训成本会成为一个严重的问题。
* 本文后续将会见到 FLUX 和 React 是如何解决这部分问题的。

### 2. 基本概念

* FLUX 框架分为四个部分
1. View：视图层
		2. Action：动作，视图层发出的消息
	3. Dispatcher：派发器，替换 MVC 的 Controller，用来接收 Actions，执行回调函数
	4. Store：数据层，用来存放应用的状态（数据），一旦发生变动，就提醒 Views 进行更新页面
* FLUX 框架的运行过程：
1. 用户访问 View
		2. View 发出用户触发的 Actions
	3. Dispatcher 接收到 Actions，并要求 Store 进行相应的更新
	4. Store 更新后，发出一个 “change” 事件
	5. View 收到 “change” 事件后，更新页面

上述过程中，数据总是 “单向流动” 的，任何相邻的部分不会发生数据的 “双向流动”，确保整个过程是 **线性** 的，就像一辆火车，只能向前开。

### 3 数据流动

> 本段落主要介绍 **双向数据流动** 与 **单向数据流动**  ，并用 MVC 模型与 FLUX 模型进行对比的方式来分析其中的差异与优劣

#### 3.1 简单模型对比

* MVC
```
Action -> Controller -> Model <-> View 
```
* FLUX
```
Action -> Dispatcher -> Store --> View
                A                  |
                |                  V
                <----  Action  <---- 
```

* 当 Action 触发时，决定了 Store 如何更新。
* 当 Store 变化后，View 同时被更新，还可能生成一个由 Dispatcher 处理的 Action。这确保了数据在系统组件间单向流动。
* 当系统有多个 Store 和 View 时，仍可视为只有一个 Store 和一个 View，因为数据只朝一个方向流动。
* 不同的 Store 和 View 之间不会直接影响彼此。

**总结：从上图对比来看，FLUX 与 MVC 非常相似，唯一区别支出在于 模型（Model / Store）与视图的数据流动方向不同**

#### 3.2 复杂应用中的模型对比

> 随着业务增多，系统的复杂性就会成级数增长，就会出现以下情况

* MVC
```
                     -> Model <---> View
                    /          \ /  
                   /            X 
                  /            / \  
Action -> Controller -> Model <---> View
                  \            \ /  
                   \            X  
                    \          / \  
                     -> Model <---> View
```
 以上当使用单一控制器处理多个模型的时候，模型与视图间就可能存在双向数据的流动，复杂度的增加，会让程序非常难以理解和调试

* FLUX
```
           -> Store ---> View ---> Action -> 
          /         \ /       \ /           \
         /           X         X             \
        /           / \       / \             \  
Dispatcher -> Store ---> View ---> Action -> Dispatcher (所有的Action最后都会走到Dispatcher)
        \           \ /       \ /             /   
         \           X         X             /
          \         / \       / \           / 
           -> Store ---> View ---> Action ->
```

由以上模型的模型，可以知道 FLUX 并不比 MVC简单，关键的不同之处在于 FLUX 所有的箭头都指向一个方向。这就让 FLUX 的代码更具有可预测性。FLUX 中的派发器确保系统中只一次只会有一种 Action 流，如果这个 Action 还没有处理完， 那么这时候再派发一个 Action 将会触发一个错误， **具体案例在后续文章中进行解释**

#### 3.3 总结

**双向数据流** 会导致瀑布式的更新，一个对象变化会引起另外一个对象的改变，并导致更多的更新。随着应用的增大，这种瀑布流式的更新方式会使得开发者很难预测用户交互可能导致的改变。

**单向数据流动** 可以让开发者更容易的推断程序是如何工作的，因为应用中的数据是单向流动的，不存在双向绑定，应用的状态只保存在 Store 中，这就允许应用汇总不同的组件可以保持高度的低耦合，由 Dispatcher 来管理并同步更新。

### 4. FLUX 的结构

#### 4.1 派发器（Dispatcher）

* Dispatcher 是 FLUX 的中心枢纽，管理 FLUX 应用的所有数据流，其本质是 Store 的回调注册机制。它的作用是向 Stores 分发 Actions。每一个 Store 各自注册属于自己的 callback 用来提供对应的处理操作。 当 Dispatcher 发出一个 Action 时，应用中**所有**的 Store 都会通过注册的 callback 收到这个 Action。

> 上面谈到一个概念，所有的 Store 都会收到这个 Action， 意味着同一个 Action 会同时更新多个 Store, 那么就会涉及到 Store 间的依赖`(TODO： 暂时这部分功能尚未验证过)`， Dispatcher可以去执行注册的 callback 的执行顺序来管理这部分依赖，Store 可以被声明等待其他 Store 完成更新后，再执行更新。

#### 4.2 数据仓库（Stores）

* Stores 包含了应用的状态和逻辑，类似 MVC 的 Model 层, 但是却管理着多个对象的状态（传统的ORM Model 只管理单个对象）。
* Store 在 Dispatcher 中注册，并提供相应的回调， 回调会绑定 Action 并把它当成自己的一个参数，当 Action 被触发时，回调函数会使用 Switch 语句来解析 Action 中的 type 参数（**不可变参数** ），达成匹配后，提供钩子执行内部的方法。这就允许 Action 通过 Dispatcher 来响应 Store 的 State 更新。Store 更新完成后，会向应用中广播一个 change 事件，而 Views 中可以选择响应事件来重新获取新的数据并更新

#### 4.3 视图（Views）

> 在这里我们通过 React 来获得一种可组合式的 Views。

* 在 React 中，Views 分为 Controller-View 与 View，区别在于 Controller-View 实际维持着类似 Controller 的行为，并且让后代（**children**） view 保持 function 特性，通过向后代传递所有数据，有助于减少管理props的数量。
	* Controller-View：
		1. 使用胶水代码从 Store 中获得数据
		2. 具有类似 Controller 的行为，向下传递 Store 数据
		3. 在 UI 中的位置处在相对顶层的位置
	* View：获得上层 View 传递的数据
		1. 只从上层 View 获得数据
		2. 具有视图显示作用
		3. 在 UI 中的位置必然在最深层的位置

> 关于 Controller-View，我们可能在某些特定情况下，会在更深层的地方加入 Controller-View 来保持我们的组件的简单，这有助于封装一个特定数据域下的相关部分，但是需要注意，在应用深层的 Controller-Views 可能会影响数据的单向流动。 因为它们可能会引入一些新的，潜在的存在冲突的数据流入口。在决定是否增加深层的 Controller-Views 时，我们需要多方面权衡简单组件与复杂数据更新流这两点。复杂数据流更新可能会导致一些稀奇古怪的副作用，伴随着不同的 Controller-Views 的 rendor 调用，潜在的增加了 Debug 的难度。

#### 4.4 动作（Actions）

* Action 是 Dispatcher提供的允许Stores中触发派发的方法，它包含了一个数据的 payload。Action 生成被包含进一个语义化的辅助方法中，来发送 Action 到 Dispatcher。

> 上述文字理解难度较大，但实际在应用中 Action 的定义非常简单。
> 举例：我们想更新 todo 应用中一个 todo-item 的文本内容。我们会在 TodoActions 模块中生成一个类似 updateText(todoId, newText)的函数，这个函数可以被视图事件处理调用执行，因此我们可以通过它来响应用户交互。Action 生成函数同样会增加一个 type 参数， 根据 type 的不同， Store 可以作出合适的响应。

**Actions 也可能来自其他地方，比如服务器端，这种情况可能会在数据初始化的时候出现，也可能是当服务器视图更新的时候返回了错误的出现**(`暂时没有遇到过`) 

### X：其他参考
* [What is the Flux Application Architecture](https://medium.com/brigade-engineering/what-is-the-flux-application-architecture-b57ebca85b9e)