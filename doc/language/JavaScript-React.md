REACT
=========================

* [官方文档](http://reactjs.cn/react/docs/getting-started.html)
* [阮一峰：React入门实例教程](http://www.ruanyifeng.com/blog/2015/03/react)

## 什么是 React
* React 是 Facebook 的开源 JS 库。
* React 起源因 Facebook 对市面上 Javascript MVC 框架都不满意，然后创造了 FLUX 框架，并基于 FLUX 框架，开发了 React。
* React 与传统 Web 应用的区别在于 React 引入了虚拟DOM概念

## 虚拟DOM

* 使用 React 不需要直接操作 DOM， 它会将 DOM 结构存储在内存中，然后同 render() 的返回内容进行比较，计算出需要改动的地方，最后才渲染到DOM中。
* 另外 React 还有一个特色是事件合成机制，能够保持事件冒泡的一致性，跨浏览器执行，甚至可以在 IS8 中使用 HTML5 事件
* 最后 React 也提供了接口让我们操作底层 API, 但大部分情况下，我们只需要操作虚拟 DOM

### 1. DOM DIFF 算法

> 由于组件并非真实的 DOM 节点，只有当它插入文档以后，才会变成真实的 DOM。 
> 根据设计，所有的 DOM 变动，都会在虚拟 DOM 上发生，然后再将实际发生变动的部分，反映在真实 DOM 上，
> 这种算法叫做 [DOM Diff](http://calendar.perfplanet.com/2013/diff/), 可以极大提高网页的性能表现

### 2. 如何操作虚拟DOM

> 如第一小节所说，我们有时需要从组件获取真实 DOM 的节点，这时就需要用到 ref 属性与 findDOMNode() 方法了。

* 步骤如下：
1. 我们需要在组件中的某个需要操作的节点上，增加 ref 属性, 并自定义 refName。
2. 在组件方法中，使用 this.refs.$refName 就会返回这个真实的 DOM 节点，需要注意的是，此时的属性获取的是真实 DOM，所以必须等到虚拟 DOM 插入文档以后，才能使用这个属性，否则会报错。
3. findDOMNode()与getDOMNod()的两种用法，效果是相同的。
    * this.refs.myTextInput.getDOMNode().focus();
    * React.findDOMNode(this.refs.myTextInput).focus();

## 数据模型

> React 中有两种数据“模型”：props 和 state。两者都是用来描述 component 状态的。

### 1.State
* 定义组件内部状态，并且当 state 改变时，组件会重新进行渲染

### 2.Props
* 由组件外部定义的状态，当props改变时，组件会重新渲染

### 数据流
* 参考 `README.md`, 关于数据单项流动的相关定义

## 组件API

### render() 组件渲染 
* 渲染一个 ReactElement 到 DOM

### setState() 更改State状态 
* 合并 nextState 和 currentState，这是事件处理函数中和请求回调函数中触发 UI 重新渲染的主要方法。另外，也支持可选的回调函数，该函数在 setState 执行完毕并且重新渲染完成之后调用。

> `注意:`绝对不要直接改变 this.state，因为在之后调用 setStat() 可能会替换掉你做的更改，把 this.state 当作不可变的。调用 setState 后，总是会进行一次重新渲染，除非在 shouldComponentUpdate() 中实现了条件渲染逻辑。仅在新 state 与 旧 state 存在差异的时候调用 setState(), 可以避免不必要的重新渲染。

### forceUpdate() 强制渲染
* 你可以通过调用 forceUpdate() 告诉 React 什么时候需要再次运行 render()

> `注意：`调用 forceUpdate() 将会导致 render() 方法在相应的组件上被调用，并且子级组件也会调用自己的 render()

## 组件的生命周期

* componentWillMount：服务器端和客户端都只调用一次，在初始化渲染执行之前立刻调用。
    * 如果在这个方法内调用 setState，render() 将会感知到更新后的 state，将会执行仅一次，尽管 state 改变了
* componentDidMount：在初始化渲染执行之后立刻调用一次，仅客户端有效（服务器端不会调用）。
    * 在生命周期中的这个时间点，组件拥有一个 DOM 展现，你可以通过 this.getDOMNode() 来获取相应 DOM 节点。
* componentWillReceiveProps：在组件接收到新的 props 的时候调用。在初始化渲染的时候，该方法不会调用。
* shouldComponentUpdate：在接收到新的 props 或者 state，将要渲染之前调用。该方法在初始化渲染的时候不会调用，在使用 forceUpdate 方法的时候也不会。
* componentWillUpdate：在接收到新的 props 或者 state 之前立刻调用。在初始化渲染的时候该方法不会被调用。
* componentDidUpdate ：在组件的更新已经同步到 DOM 中之后立刻被调用。该方法不会在初始化渲染的时候调用。
* componentWillUnmount：在组件从 DOM 中移除的时候立刻被调用。

## 学习步骤

### 在 HTML 中加载 React 库
1. 我们可以通过从 [github](https://facebook.github.io/react/downloads.html) 获取 React 库
2. 通过 HTML 使用 script 标签加载 React 库

        <script src="./react.js"></script>

3. 在 HTML 使用 React 的 JSX 语法, 需要在 script 标签上需要加上  type="text/babel",

        <script type="text/babel">
            // ** Our code goes here! **
        </script>      
   
### 通过 nodeJs 使用 React 库        
1. 通过 npm 安装

        npm install --save react
        npm install --save react-dom

2. 通过 require 引入 react

        import React from 'react';
        import ReactDOM from 'react-dom';

### 渲染到DOM

1. 使用ReactDOM.render将模板语言转为HTML语言，并插入到指定节点 wrapper

        ReactDOM.render(
          <div>HELLO WORLD</div>,
          document.getElementById('wrapper')
        );
   
2. 关于 JSX 语言，查阅[JSX Syntax](http://facebook.github.io/react/docs/displaying-data.html#jsx-syntax)
    * 代码块使用 “{}” 用 javascript 进行规则解析
    * HTML标签 “<>” 用 HTML 进行规则解析

### 创建组件

1. 使用react.Component创建组件

        class ButtonComponent extends react.Component {
            render() {
                return (
                    <div>HELLO WORLD</div>
                );
              }
        }

2. 在React中使用组件，组件同样也支持嵌套组件

        import ButtonComponent from './ButtonComponent';
        
        ReactDOM.render(
          <ButtonComponent />,
          document.getElementById('wrapper')
        );
   
3. 组件的生命周期查阅上文
4. 在 JSX 中的 HTML 标签，使用 ref 字段后, 可以在代码中使用 this.refs.XXX.getDOMNode() 可以操作真实 DOM

## DEMO 讲解

> 理解 demo 需要先理解 FLUX 的数据单项流动
> DEMO 中使用了 FLUX 作为状态管理器

### constants
* actionType.js
    1. 使用 fbjs 的小工具 keyMirror 来更方便的定义 action 的 CONST 列表
    2. 将 CONST 使用 export 进行输出，方便其他 js 进行引用
    3. 用 CONST 的方式定义 action type 可以在 action 与 dispatcher 进行绑定时候，避免一些低级错误

### stores

> 由于 Store 需要在变动后向 View 发送"change"事件，因此它必须实现事件接口
> 所以 ButtonStore 继承了EventEmitter.prototype，因此就能使用ButtonStore.on()和ButtonStore.emit()，来监听和触发事件了。   
> ButtonStore 更新后（this.addItemHandler()）发出事件（this.emitChange()），表明状态已经改变。
> View（即ButtonController） 监听到这个事件，就可以查询新的状态，更新页面了。

* ButtonStore.js
    * ButtonStore.itemStore 用来保存条目，
    * ButtonStore.emitChange() 用来发出一个 change 事件。
    * ButtonStore.addChangeListener() 用来监听 itemStore 更新
    * ButtonStore.removeChangeListener() 用来移除一个 change 事件
    
    
### dispatcher

> Dispatcher 的作用是将 Action 派发到 Store，可以将他看作一个路由器，Dispatcher 只用来派发 Action，不应该有其他逻辑

* AppDispatcher.js
    * 使用 flux 的 Dispatcher.register 方法用来登记 Action 的回调函数
    * Dispatcher 收到 ADD_ITEM 动作，就会执行之后的代码，使用 ButtonStore的函数对 Store 进行操作。
    
### action

> 每个 Action 都是一个对象，包含一个 actionType 属性（说明动作的类型）和一些其他属性（用来传递数据）。

* ButtonActions.js
    * ButtonActions.addItem方法使用Dispatcher.dispatch，把动作 ADD_ITEM 与 ButtonStore 进行绑定。

### Components

* ButtonController.js
    * 通过 ButtonController.componentDidMount 将onChange绑定到 store 的change 事件
    * 当 ButtonController 发现 Store 更新时，就会调用 this.onChange 更新组件状态，从而触发重新渲染。
    * 封装 addItem 到 ButtonController.createItem，并作为 props 提供给 ButtonComponent 组件
    * 将 Store 作为 props 提供给 ButtonComponent 组件
   
* ButtonComponent.js
    * 当 ButtonController 里面的 Store 更新时候，会通过 props 告知 ButtonComponent 同时进行渲染。

