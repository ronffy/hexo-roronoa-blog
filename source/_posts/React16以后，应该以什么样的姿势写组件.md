---
title: React16以后，应该以什么样的姿势写组件
date: 2018-12-12 11:30:00
keyword: React
describe: React16后，应该以什么样的姿势写组件
author: ronffy
---

## 前言

本篇文章主旨在介绍 React16 之后版本中新增或修改的地方，所以对于 React16 之前版本的功能，本篇文章当作您已经充分了解了，不再赘述。

## 纯函数的PureComponent

我们知道，对 React 组件的性能优化，`shouldComponentUpdate`函数是很重要的一拍，所以 React 才会在 `React.Component`的基础上增加了`React.PureComponent`，但是对于非class类的纯函数写法，却没法增加这样的便捷处理。
对于这个问题，React16 增加了`React.memo`这个高阶组件（该方法在V16.6.0后才支持）。

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


## 生命周期函数做了哪些改动

首先统一下一些概念，React生命周期分为三类：挂载、更新、卸载，React16后又多了一个异常，我们一一看下。

### 挂载

#### 生命周期函数的执行顺序
`constructor` ->  `getDerivedStateFromProps` -> `render` -> `componentDidMount`

`render`和`componentDidMount`较 React16 之前无变化，不再过多介绍。对于挂载过程，我们着重看下`constructor`和`getDerivedStateFromProps`。

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

所以，React16 以后用到`constructor`的场景会少很多。


#### static getDerivedStateFromProps

上面我们讲到，应避免使用props给state赋值，但是在 React16 前我们都是这么做的，现在如果不让这么操作了，那该在哪里处理这块逻辑呢？ React16 给出的答案就是 `static getDerivedStateFromProps` 静态方法，挂载组件时，该方法会在`render`前执行。

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

这样，有没有发现`componentWillReceiveProps`也没有用武之地了？是的，React16 把它也“删掉”了。（目前 React16 并未真的删除`componentWillReceiveProps`，只是提示用户该方法已经“out”，建议使用更好的`getSnapshotBeforeUpdate` 或 `getDerivedStateFromProps`）


### 更新

#### 生命周期函数的执行顺序

`getDerivedStateFromProps` -> `shouldComponentUpdate` -> `render` -> `getSnapshotBeforeUpdate` -> `componentDidUpdate`

除了`getSnapshotBeforeUpdate`，`getDerivedStateFromProps`已经介绍过了，而其他的几个生命周期函数与 React16 之前无异，所以这里主要介绍下`getSnapshotBeforeUpdate`。

#### getSnapshotBeforeUpdate

- 语法
```ts
class C extends React.Component {
  getSnapshotBeforeUpdate(prevProps, prevState): Snapshot {
    
  }
  componentDidUpdate(prevProps, prevState, snapshot) {
    // snapshot 是从 getSnapshotBeforeUpdate 的返回值，默认是 null
  }
}
```




### 卸载

`componentWillUnmount`


### 异常

`componentDidCatch`这个生命周期是 React16 新增的，用于捕获组件树的异常，但无法捕获这个方法内的异常。

```js
componentDidCatch(error, info) {
  // xxx
}
```




