---
title: react生命周期
date: 2019-08-05 14:22:04
tags:
---

## react生命周期(v16.0前)

 - 组件初始化(initialization)阶段
   -  继承react Component这个基类,获得render(),生命周期等方法可以使用；
   -  super(props)用来调用基类的构造方法( constructor() ), 也将父组件的props注入给子组件；
   -  constructor()用来做一些组件的初始化工作，如定义this.state的初始内容。
 - 组件的挂载(Mounting)阶段
   -  componentWillMount()
      在这边调用this.setState不会引起组件重新渲染，也可以把写在这边的内容提前到constructor()中，很少用。
   -  render()
      render是纯函数,不能在里面执行this.setState，会有改变组件状态的副作用。
   -  componentDidMount()

- 组件的更新(update)阶段
    -  父组件重新render致更新
       -   父组件重新render导致的**重传**props，无论props有没有变化这可以通过shouldComponentUpdate(nextProps,nextState)优化。
       -   props变化，在componentWillReceiveProps方法中，将props转换成自己的state。
    -  组件本身调用setState致更新

  updation分为：
    -  componentWillReceiveProps(nextProps)
       父组件重传props时就会调用,但父组件render方法的调用不能保证重传给当前组件的props是有变化的。
    -  shouldComponentUpdate(nextProps, nextState)
       返回false则当前组件更新停止，可用于优化组件性能。
    -  componentWillUpdate(nextProps, nextState)
    -  render
    -  componentDidUpdate(prevProps, prevState)
- 卸载(Unmounting)阶段
    -  componentWillUnmount()
       在组件被卸载前调用，可以在这里执行一些清理工作，比如清除组件中使用的定时器，清除componentDidMount中手动创建的DOM元素等，以避免引起内存泄漏。

![react生命周期](https://upload-images.jianshu.io/upload_images/5287253-bd799f87556b5ecc.png)
