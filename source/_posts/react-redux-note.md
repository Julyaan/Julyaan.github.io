---
title: react-redux学习笔记
date: 2017-07-07 16:56:03
tags:
- react
- redux
- es6
categories: 前端
---

*note:这是一篇随意的不能更随意的学习随笔*
使用action创建函数封装逻辑是react与redux配合的最佳实践：

-  当所有的逻辑都被转移到redux之后，react就可以只负责渲染界面并发起action创建函数了
ActionCreator要返回一个action对象，那么它是如何处理复杂的逻辑的呢？ ---> 中间件

发起一个action创建函数，只需要将其返回结果传给dispatch()
<img src='http://7xwnfc.com1.z0.glb.clouddn.com/redux.jpg'>
为什么使用redux thunk?该中间件可以让action创建函数先不返回action对象，而是一个函数：
<!--more-->
-  通过这个函数： 接收store的两个方法dispatch和getState作为参数，延迟dispatch或只在满足指定条件情况下dispatch
如何激活redux thunk中间件？在createStore中加入参数：applyMiddleware(thunk)

将redux手动连接到react组件，即不使用react-redux:

- 更新方法：store.subscribe(render)
- 为什么要使用react-redux?
  - 手动的缺点：
    - 1.无法直接给组件传递state和方法
    - 2.任意的state变化都会导致整个组件树的重新渲染，没有性能优化
  - react-redux优势：
    - 不仅可以给组件树中的任一组件绑定state和方法，还进行了性能优化，避免了不必要的重新渲染


使用react-redux：

- 1.在所有组件的顶层使用Provider组件给整个程序提供store
  - 原理：通过context将store专递给子组件
  - store是一个包含三个函数的对象，三个函数分别是：subscribe，dispatch，getState
  - Provider中重新定义了componentWillReceiveProps，检查每次渲染时代表store的prop和上一次是否一样，如果不一样则给出警告
- 2.使用connect()将state和action创建函数绑定到组件中，接收两个参数
  - 第一个参数: 函数，参数为state，通过返回一个对象，将state或其某些属性合并到组件props的相应的属性中
  - 第二个参数：对象或函数。分别有四种情况：
    - 对象，是所有actionCreator的集合，将所有actionCreator传到组件的同名属性，并为每个actionCreator隐式绑定dispatch方法；
    - 函数，参数为dispatch,返回的对象会被自动合并到组件的props中，但不会自动为actionCreator绑定dispatch方法，需手动绑定
    - 函数，参数为dispatch,返回值使用redux的bindActionCreators()来减少样板代码:bindActionCreators(ActionCreators, dispatch)
    - 为空，组件自己使用dispatch发起action
  - 注：除了以上写法外，connect还有装饰器写法
- 3.并不是所有的组件都需要连接redux，理解容器组件和展示组件的差异性，connect相当于是对容器组件和展示组件的一个连接

源码分析：

- 顶层组件provider
```javascript```
  class Provider extends Component {
    ···
    getChildContext () {
      store: this.props.store
    }
    render () {
      return this.props.children
    }
    ···
  }
  Provider.childContextTypes = {
    store: PropType.object
  }
  ```
- 底层组件如何使用Context

模块化应用：

- 状态树的设计原则
  - 1.一个模块控制一个节点
  - 2.避免数据冗余
  - 3.树形结构扁平
- 单一数据源原则：对于嵌套过深的数据结构（状态树）问题，先定义多个reducer对数据进行`拆解`访问／修改再通过`combineReducers`函数将零散的数据`拼装`回来的方法。
