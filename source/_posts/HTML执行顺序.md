---
title: HTML执行顺序（包含浏览器线程基础）
date: 2019-08-03 20:33:35
tags:
categories: html
---

## 浏览器线程基础

浏览器有主进程和渲染进程组成，其中主进程只有一个，而渲染进程可以有多个，一般一个页面一个渲染进程。。。即一个页面的呈现主要是由浏览器渲染进程实现的（render进程），主要作用为页面渲染，脚本执行，事件处理等。而render进程又是多线程的，它主要包含以下主要线程：

1. GUI渲染线程
 - 负责渲染浏览器界面，解析HTML、CSS，构建DOM树和RenderObject，布局和绘制等。
 - 当界面需要重绘（Repaint）或由于某种操作引发回流（reflow）时，该线程就会执行。
 - GUI渲染线程与JS引擎线程是互斥的，当JS引擎执行时GUI线程会被挂起（相当于被冻结），GUI更新会被保存在一个队列中等到JS引擎空闲时立即被执行。
2. JS引擎线程
 - 也称为JS内核，负责处理解析JavaScript脚本程序，运行代码。
 - JS引擎一直等待着任务队列中任务的到来，然后加以处理。一个render进程中无论什么时候都只有一个JS线程在运行JS程序。
 - 同样注意，GUI渲染线程和JS引擎线程是互斥的，所以如果JS执行时间过长，会造成页面渲染不连贯，导致页面渲染加载阻塞。
3. 事件触发线程
 - 归属于浏览器而不是JS引擎，用来控制事件循环（JS引擎自己都忙不过来，需要浏览器另开线程协助）。
 - 当JS引擎触发鼠标点击事件时（也可来自浏览器内核的其他线程如setTimeout、AJAX等），会将对应任务添加到事件线程中。
 - 当对应的事件符合触发条件被触发时，该线程会把事件添加到待处理队列的对尾，等待JS引擎处理。
 - 注意，由于JS的单线程关系，所以这些待处理队列中的事件都得排队等待（当JS引擎空闲时才会去执行）。
4. 定时触发器线程
 - 传说中setInterval与setTimeout所在线程。
 - 浏览器定时计数器并不是由JS引擎计数的（因为JS引擎是单线程的，如果处于阻塞线程状态就会影响计数的准确性），因此通过单独线程来计时并触发定时（计时完毕后，添加到事件队列中，等待JS引擎空闲后执行）。
5. 异步http请求线程
 - 在XMLHttpRequest连接后是通过浏览器新开一个线程请求。
 - 将检测到状态变更时，如果设置有回调函数，异步线程就产生状态变更事件，将这个回调再放入事件队列中，再由JS引擎执行。

![浏览器内核](https://image-static.segmentfault.com/101/154/1011542037-5a72c1b777df5_articlex)

## HTML整体执行步骤

1. 加载整体html文件。
2. 至上而下解析html。
3. 解析html建立dom树，遇到诸如`<script>`、`<link>`等标签时，就会去下载相应内容（下载和渲染是不冲突的），下载是下载线程在执行，浏览器多线程。下载好了解析、执行（此时JS引擎执行），如果是`<link>`标签，解析css构建CSSOM树。
4. DOM和CSSOM结合生成render树。
5. 布局render树（Layout/reflow），负责各元素尺寸位置计算。
6. 绘制render树（paint），绘制页面像素信息。
7. 浏览器会将各层的信息发送给GPU，GPU会将各层合成（composite），显示在屏幕上。

>>放个实验```
<!DOCTYPE html>  
<html lang="zh">  
  
<head>  
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />  
    <title>浅谈Html页面内容执行顺序</title>
    <link rel="stylesheet" href="red.css">
    <script type="text/javascript" src="jquery.js"></script>
    <script type="text/javascript" src="test.js"></script>
    <link rel="stylesheet" href="font.css">
    <script type="text/javascript" src="test2.js"></script>

</head>  
  
<body>  
    <p>html顺序测试</p>
    <img src="1.png" alt="">
    <input  value="101" />  
    <link rel="stylesheet" href="styl.css">
</body>  
  
</html> 
```
test.js代码为：
```
        for(var i=0;i<10000;i++){
            console.log("delay");
            if(i==9999){
                loadStyle('lime.css');
            }
        }


        function loadStyle(url){
            var link = document.createElement('link');
            link.type = 'text/css';
            link.rel = 'stylesheet';
            link.href = url;
            var head = document.getElementsByTagName('head')[0];
            head.appendChild(link);
        }
```

![例子图1](https://image-static.segmentfault.com/264/308/2643084598-5afabc7fbe4f8)

从下载的角度讲：
  - 当HTML解析器遇到`<script>`、`<link>`、`<img>`标签，开始下载时，只存在一种阻塞情况，就是`<script>`标签会阻止`<img>`资源下载，其余相互之间下载没有影响。但直到外部样式加载完毕，外部脚本才开始执行（即外部样式的下载，虽然不会影响外部脚本的下载，但会影响脚本的运行）

![例子图1](https://image-static.segmentfault.com/317/314/3173147971-5afabcbd7fdd4)

 下载完成之后，就会运行解析相应的JS文件或者CSS文件，运行JS文件需要JS引擎线程，前面提到，JS引擎和GUI时互斥的，所以在解析的角度讲，JS引擎运行，会阻塞GUI，即阻止页面渲染。从上图可以看出，在test.js下载之后，解析运行时，由于有for循环函数的运行，页面首次渲染时间被推至1100ms才渲染完成。


## HTML页面内容执行顺序

HTML文档按照从上到下执行，有时我们使用 `$(function(){});`控制js的执行，在HTML文档树结构加载完毕(并不包括一些静态资源：比如图片等)才执行，在同一个页面，可以使用多个$(function(){})。这是就产生了多个$(document).ready()的执行顺序问题，多个$(document).ready()的执行顺序并非单纯的顺序执行，其与嵌套层级也有一定的关系。

>>举个栗子```
<html>
<head>
<script src="./jquery-1.9.0.min.js"></script>
<script type="text/javascript">
  $(function(){
    alert('1');
    $(function(){
      alert('2');
      $(function(){
        alert('3');
      });
    });
 
  });
</script>
<body>
TTTTTTTTTTTT
<script type="text/javascript">
  $(document).ready(function() {
    alert('4');
    $(function(){
      alert('5');
    });
 
  });
</script>
KKKKKKKKKKKK
<script type="text/javascript">
  $(function(){
    alert('6');
    $(document).ready(function() {
      alert('7');
    });
 
  });
</script>
</body>
</html>
```
运行alert显示顺序为：1，4，6，2，5，7，3

