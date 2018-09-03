---
title: 试用Taro问题汇总
date: 2018-09-02 11:30:00
keyword: Taro
describe: 一套遵循 React 语法规范的多端统一开发框架
tags: [taro, react, javascript]
---

> [Taro](https://github.com/NervJS/taro) 是由京东 - 凹凸实验室打造的一套遵循 React 语法规范的多端统一开发框架。

我试用了有15天左右，总的来说，这是一款优秀的框架，尤其补充了目前市面上无法用 React 开发小程序的需求空缺，所以其优点就不多说了，大家可去其官方查看[详细文档](https://taro.aotu.io/)

下面说下我的试用感受，希望帮助后面使用Taro的同学少踩一些坑；因为能力有限，可能了解和认识会有一些不到位的地方，还望各路大佬不吝留言赐教

### 存在的问题

以下，是我在使用Taro过程中遇到的影响开发流程或体验的地方：

- 1.不支持source map，调试可通过debugger

- 2.不支持alias，所以项目里会有大片大片的 `../../../`，不利于后期维护

- 3.全局请求的需求，官方未有最佳方案。理应`app.tsx`是最合适的地方，但是该组件的`Provider`组件内写的任何组件都会被Taro替换掉。我目前是通过在`app.tsx`里通过`store.dispatch(action)`发送全局异步请求

- 4.redux/connect方法的`mapStateToProps`缺少第二个参数`ownProps`

- 5.组件嵌套时，taro生命周期与react生命周期执行顺序有差异，如图是Taro的生命周期执行顺序，可以看到`componentDidMount`跟React是相反的。![图片描述](https://user-gold-cdn.xitu.io/2018/9/3/1659d69bf94b31d4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 6.不可以使用 ... 拓展操作符给组件传递属性，`<Comp {...props} />`写法错误

- 7.属性不能传入 JSX 元素，`<Content footer={<View />} />`写法错误

### 展望

虽然存在以上种种问题，还是要感谢京东前端团队能够开源一款React语法的多端开发框架，让我们React粉儿能够用React开发小程序；以上有些问题我已提了PR，如Q4，并且维护人员很快将PR进行了merge，凹凸团队对这个项目的重视程度和责任心可见一斑，所以我相信，凹凸团队一定可以把Taro不断完善的，加油！


  [1]: /img/bVbgkHJ