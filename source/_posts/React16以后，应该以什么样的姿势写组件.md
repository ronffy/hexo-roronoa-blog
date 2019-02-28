---
title: React16解锁了哪些新姿势
date: 2019-01-07 11:30:00
keyword: react
tags: react
describe: React16以后，都增加了哪些新功能呢
author: ronffy
---

> React16 后的各功能点是多个版本陆陆续续迭代增加的，本篇文章的讲解是建立在 `16.6.0` 版本上
> 本篇文章主旨在介绍 React16 之后版本中新增或修改的地方，所以对于 React16 之前版本的功能，本篇文章当作您已充分了解了，不再赘述

## 更新概览
从 React v16.0 ~ React v16.6 的更新概览（只涉及部分常用api）：

- React v16.0  

1. render 支持返回数组和字符串
2. 支持自定义 DOM 属性
3. 减少文件体积

- React v16.3  

1. createContext
2. createRef、
3. 生命周期函数的更新

- React v16.4  

更新 getDerivedStateFromProps


- React v16.6  

1. memo
2. lazy
3. Suspense
4. static contextType
5. static getDerivedStateFromError()

- React v16.7（~Q1 2019）  

Hooks

接下来将针对影响较大，使用频率较高的更新点逐一讲解。

## 纯函数的PureComponent

我们知道，对 React 组件的性能优化，`shouldComponentUpdate`函数是很重要的一啪，所以 React 才会在 `React.Component`的基础上增加了`React.PureComponent`，但是对于非class类的纯函数写法，却没法增加这样的便捷处理。
对于这个问题，React16.6 增加了`React.memo`这个高阶组件

一般使用方式：
```js
const C = React.memo(props => {
  // xxx
})
```

`React.memo`的实现类似`React.PureComponent`，所以它内部是对对象进行浅比较。
`React.memo`允许你自定义比较方法，如下：
```ts
// 函数的返回值为 true 时则更新组件，反之则不更新
const equalMethod = (prevProps, nextProps): boolean => {
  // 定义你的比较逻辑
}
const C = React.memo(props => {
  // xxx
}, equalMethod)
```


## 新的生命周期函数是怎样的

React生命周期分为三个阶段：挂载、更新、卸载，React16后又多了一个异常，我们一一看下。

![](/img/post/react16/new-ing.jpg)

### 挂载

#### 生命周期的执行顺序  

1. `constructor`
2. `static getDerivedStateFromProps`
3. `render`
4. `componentDidMount`

`render`和`componentDidMount`较 React16 之前无变化。对于挂载过程，我们着重看下`constructor`、`componentWillMount`和`static getDerivedStateFromProps`。

#### constructor

1. 初始化 state  
  注意：应避免使用props给state赋值，这样的话， state 的初始化可以提到constructor外面处理  
```js
constructor(props) {
  super(props);
  this.state = {
    x: 1,
    // y: props.y, // 避免这样做，后面我们会讲应该怎样处理
  }
}
```

2. 给方法绑定this
```js
constructor(props) {
  super(props);
  this.handleClick = this.handleClick.bind(this);
}
```

但是，以上两件事放到`constructor`外面处理会更简单些，如下：

```js
class C extends React.Component {
  state = {
    x: 1
  }
  handleClick = (e) => {
    // xxx
  }
}
```

所以，React16 以后用到`constructor`的场景会变少。


#### componentWillMount

可以看到，`componentWillMount`在 react16 中被“删掉”了（这样说其实是有问题的，因为 react 并未真正删除该生命周期函数，只是告诫开发者，该函数在未来版本中会被废弃掉），那么问题就出现了，原先在这个生命周期中的做的事情，现在该放到哪里去做呢？

首先问自己一个问题，原先的时候都在这个生命周期里做什么？答案是大部分时候会在这里做AJAX请求，然后执行 setState 重新渲染。

然而在`componentWillMount`里做AJAX请求实在不是一个明智之举，因为对于同构项目中，`componentWillMount`是会被调用的。

还有人会在这里面初始化 state ，关于 state 的初始化，请参看楼上小节。

综上所述，`componentWillMount`其实本来没有什么主要作用，如果你的代码规范，去掉的话，不会对现在的项目产生什么影响。


#### static getDerivedStateFromProps

上面我们讲到，应避免使用props给state赋值，但是在 React16 前我们都是这么做的，现在如果不让这么操作了，那该在哪里处理这块逻辑呢？ React16 给出的答案就是 `static getDerivedStateFromProps` ，挂载组件时，该静态方法会在`render`前执行。

```ts
class C extends React.Component {
  state = {
    y: 0
  }
  static getDerivedStateFromProps(props, state): State {
    if(props.y !== state.y) {
      return {
        y: props.y
      };
    }
  }
}
```

`getDerivedStateFromProps`的返回值将作为setState的参数，如果返回null，则不更新state，不能返回object 或 null 以外的值，否则会警告。

`getDerivedStateFromProps`是一个静态方法，是拿不到实例`this`的，所以开发者应该将该函数设计成纯函数。

这样，有没有发现`componentWillReceiveProps`也就没有用武之地了？是的，React16 把它也“删掉”了（这样说其实是有问题的，因为 react 并未真正删除该生命周期函数，只是告诫开发者，该函数在未来版本中会被废弃掉，建议使用更好的`getSnapshotBeforeUpdate` 或 `getDerivedStateFromProps`）


### 更新

#### 生命周期函数的执行顺序

1. `static getDerivedStateFromProps`
2. `shouldComponentUpdate`
3. `render`
4. `getSnapshotBeforeUpdate`
5. `componentDidUpdate`

`static getDerivedStateFromProps`前面已经介绍过了，而其他的几个生命周期函数与 React16 之前基本无异，所以这里主要介绍下`getSnapshotBeforeUpdate`。

#### getSnapshotBeforeUpdate

在 react 更新 dom 之前调用，此时 state 已更新；
返回值作为 componentDidUpdate 的第3个参数；
一般用于获取 render 之前的 dom 数据

- 语法
```ts
class C extends React.Component {
  getSnapshotBeforeUpdate (prevProps, prevState): Snapshot {
    
  }
  componentDidUpdate(prevProps, prevState, snapshot) {
    // snapshot 是从 getSnapshotBeforeUpdate 的返回值，默认是 null
  }
}
```

`getSnapshotBeforeUpdate` 的使用场景一般是获取组建更新之前的滚动条位置。


### 卸载

`componentWillUnmount`

较之前无变化


### 异常

`componentDidCatch` 这个函数是 React16 新增的，用于捕获组件树的异常，如果`render()`函数抛出错误，则会触发该函数。可以按照 `try catch` 来理解和使用，在可能出现错误的地方，使用封装好的包含 `componentDidCatch` 生命周期的组建包裹可能出错的组件。

```js
class PotentialError extends React.Component {
  state = {
    error: false,
  }
  componentDidCatch(error, info) {
    console.error(info);
    this.setState({
      error
    });
  }
  render() {
    if (this.state.error) {
      return <h1>出错了，请打卡控制台查看详细错误！</h1>;
    }
    return this.props.children;   
  } 
}
```


如：
```js
const Demo = () => (
  <PotentialError>
    <div>{{a: 1}}</div>
  </PotentialError>
)
```
这样，`Demo` 组件即使直接使用对象作为子组件也不会报错了，因为被 `PotentialError` 接收了。


### 新生命周期的完整demo

看看穿上新生命周期这身新衣服后的样子吧

```js
import React from 'react'

export default class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    // 初始化state方式（1）
    this.state = {

    }
  }
    
  static defaultProps = {

  }

  // 初始化state方式（2）
  state = {

  }
  static getDerivedStateFromProps(props, state) {
    return state
  }
  componentDidCatch(error, info) {

  }
  render() {

  }
  componentDidMount() {

  }
  shouldComponentUpdate(nextProps, nextState) {
    
  }
  getSnapshotBeforeUpdate(prevProps, prevState) {

  }
  componentDidUpdate(prevProps, prevState, snapshot) {

  }
  componentWillUnmount() {

  }

}
```


## Suspense


## Hooks


## time slicing





【未完待续】
