---
title: 'rem单位border-radius:50%在安卓手机中画圆变形'
date: 2019-09-29 11:07:32
tags:
---


使用border-radius:50%,或者border-radius的值与宽高相等，都可实现一个完美的圆形，但是在不同的安卓手机中，会有不同程度的变形（有的扁圆，有的大，有的小）；当使用px做为宽高的单位，border-radius:50%画出来的圆是不会变形的；但使用rem时，rem在换算为px时，会是一个带小数点的值，安卓对小于1px的做了处理（不同浏览器对小于1px的处理方式不同，有的采用四舍五入，有的大于某个值展示1px否则就舍去），从而导致圆角不圆；在ios下就没有这个问题。

## 推荐的方法
```
  i{
       display: inline-block;
       width: .16rem;
       height: .16rem;
       background-color: #D0021B;
       border-radius: 50%;
       transform: scale(.5);
       transform-origin: 0% center;
   }
```
先把width，height的值放大一倍，然后用transform scale(.5)缩小一倍，接着用transform-origin调整下圆的位置就大功告成了！