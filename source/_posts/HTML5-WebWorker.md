---
title: HTML5-WebWorker
date: 2019-08-14 11:56:00
tags:
---

## Web Worker
众所周知，JavaScript 是单线程的。Web Worker 的作用，就是为 JavaScript 创造多线程环境。主线程创建 Worker 线程，将一些任务分配给后者运行。在主线程运行的同时，Worker 线程在后台运行，两者互不干扰。等到 Worker 线程完成计算任务，再把结果返回给主线程。目的是，为主线程分担一些含耗时较大的算法代码或高延迟任务。

Worker 线程一旦新建成功，就会始终运行，不会被主线程上的活动（比如用户点击按钮、提交表单）打断。这样有利于随时响应主线程的通信。但是，这也造成了 Worker 比较耗费资源，不应该过度使用，而且一旦使用完毕，就应该关闭。

### API
  1. 创建新的Worker
  ```
  var worker = new Worker('work.js'); 
  ```
    *Worker()* 构造函数的参数是一个脚本文件，该文件就是 Worker 线程所要执行的任务。由于 Worker 不能读取本地文件，所以这个脚本必须来自网络。
  2. 传递参数
  ```
  worker.postMessage('hello')
  ```
    该方法的参数，就是主线程传给 Worker 的数据。它可以是各种数据类型，包括二进制数据。这种通信是拷贝关系，即是传值而不是传址。Worker 对通信内容的修改，不会影响到主线程。

  3. 接收消息
  ```
  // 主线程
  worker.onMessage = function(event){
    console.log('Received message ' + event.data);
  }

  //Worker线程
  self.addEventListener('message', function (e) {
    self.postMessage('You said: ' + e.data);
  }, false);

  或

  this.addEventListener('message', function (e) {
    this.postMessage('You said: ' + e.data);
  }, false);

  或

  addEventListener('message', function (e) {
    postMessage('You said: ' + e.data);
  }, false);
  ```
    主线程通过worker.onmessage指定监听函数，接收子线程发回来的消息，从事件对象的data属性中获取Worker发来的数据。

    Worker 线程中self代表子线程自身，即子线程的全局对象（Worker 线程所在的全局对象，与主线程不一样，无法读取主线程所在网页的 DOM 对象，也无法使用document、window、parent这些对象。但是，Worker 线程可以使用navigator对象和location对象）。除了使用self.addEventListener()指定监听函数，也可以使用self.onmessage指定。self.postMessage()方法用来向主线程发送消息。

  4. 异常处理
  ```
  worker.onerror(function (event) {
    console.log([
      'ERROR: Line ', e.lineno, ' in ', e.filename, ': ', e.message
    ].join(''));
  });

  // 或者
  worker.addEventListener('error', function (event) {
    // ...
  });
  ```
  5. 关闭Worker
  ```
  // 主线程
  worker.terminate();

  // Worker 线程
  self.close();
  ```
  使用完毕，为了节省系统资源，必须关闭 Worker。

  6. Worker中加载其他脚本
  ```
  importScripts('script1.js', 'script2.js');
  ```

  7. 使用Web Worker还有以下注意点：
    - 分配给 Worker 线程运行的脚本文件，必须与主线程的脚本文件同源，不能跨域加载JS。
    - Worker 线程不能执行alert()方法和confirm()方法，不能访问DOM，但可以使用 XMLHttpRequest 对象发出 AJAX 请求。
    - Worker 线程内部还能再新建 Worker 线程（目前只有 Firefox 浏览器支持）。
    - 这种普通的webworker是当前页面专有的。然后还有种共享worker(SharedWorker)，这种是可以多个标签页、iframe共同使用的，接下来介绍如何使用SharedWorker实现标签页之间的通信。


## Shared Worker
    SharedWorker可以被多个脚本共同使用，但必须保证这些标签页都是同源的(相同的协议，主机和端口号)。
  1. 应用页面中
  ```
    if (typeof Worker === "undefined") {
      alert('当前浏览器不支持webworker')
    } else {
      let worker = new SharedWorker('worker.js') 
      worker.port.addEventListener('message', (e) => {
        console.log('来自worker的数据：', e.data)
      }, false)

       // 显示指定worker.port.start()方法建立与worker间的连接
      worker.port.start()
      window.worker = worker
    }

    // 获取和发送消息都是调用postMessage方法，我这里约定的是传递'get'表示获取数据。
    window.worker.port.postMessage('get')
    window.worker.port.postMessage('发送信息给worker')
  ```

  2. 其中worker.js中的内容
  sharedWorker所要用到的js文件，不必打包到项目中，直接放到服务器即可。
  worker.js和index.html在同一目录。
  ```
  let data = ''
  onconnect = function (e) {
    let port = e.ports[0]

    port.onmessage = function (e) {
      if (e.data === 'get') {
        port.postMessage(data)
      } else {
        data = e.data
      }
    }
  }
  ```

  3. 页面A发送数据给worker，然后打开页面B，调用window.worker.port.postMessage('get')，即可收到页面A发送给worker的数据。

  - 因为客户端和webworker端的通信不像websocket那样是全双工的，所以客户端发送数据和接收数据要分成两步来处理。示例中会有两个按钮，分别对应的向sharedWorker发送数据的请求以及获取数据的请求，但他们本质上都是相同的事件--发送消息。



## 实现多标签页之前通信的其他方法
  1. webSocket
  2. localStorage

