# 7.6 Web Worker

*   概述
*   新建和启动子线程
*   子线程的事件监听
*   主线程的事件监听
*   错误处理
*   关闭子线程
*   主线程与子线程的数据通信
*   同页面的 Web Worker
*   参考链接

## 概述

JavaScript 语言采用的是单线程模型，也就是说，所有任务排成一个队列，一次只能做一件事。随着电脑计算能力的增强，这一点带来很大的不便，无法充分发挥 JavaScript 的潜力。尤其考虑到，File API 允许 JavaScript 读取本地文件，就更是如此了。

Web Worker 的目的，就是为 JavaScript 创造多线程环境，允许主线程将一些任务分配给子线程。在主线程运行的同时，子线程在后台运行，两者互不干扰。等到子线程完成计算任务，再把结果返回给主线程。因此，每一个子线程就好像一个“工人”（worker），默默地完成自己的工作。

普通的 Wek Worker，只能与创造它们的主进程通信。还有另一类 Shared worker，能被所有同源的进程获取（比如来自不同的浏览器窗口、iframe 窗口和其他 Shared worker）。本节不涉及这一类的 worker 进程。

Web Worker 有以下几个特点：

*   同域限制。子线程加载的脚本文件，必须与主线程的脚本文件在同一个域。

*   DOM 限制。子线程无法读取网页的 DOM 对象，即 document、window、parent 这些对象，子线程都无法得到。（但是，navigator 对象和 location 对象可以获得。）

*   脚本限制。子线程无法读取网页的全局变量和函数，也不能执行 alert 和 confirm 方法，不过可以执行 setInterval 和 setTimeout，以及使用 XMLHttpRequest 对象发出 AJAX 请求。

*   文件限制。子线程无法读取本地文件，即子线程无法打开本机的文件系统（file://），它所加载的脚本，必须来自网络。

使用之前，检查浏览器是否支持这个 API。支持的浏览器包括 IE10、Firefox (从 3.6 版本开始)、Safari (从 4.0 版本开始)、Chrome 和 Opera 11，但是手机浏览器还不支持。

```
if (window.Worker) {
  // 支持
} else {
  // 不支持
}
```

## 新建和启动子线程

主线程采用 new 命令，调用 Worker 构造函数，可以新建一个子线程。

```
var worker = new Worker('work.js');
```

Worker 构造函数的参数是一个脚本文件，这个文件就是子线程所要完成的任务，上面代码中是 work.js。由于子线程不能读取本地文件系统，所以这个脚本文件必须来自网络端。如果下载没有成功，比如出现 404 错误，这个子线程就会默默地失败。

子线程新建之后，并没有启动，必需等待主线程调用 postMessage 方法，即发出信号之后才会启动。postMessage 方法的参数，就是主线程传给子线程的信号。它可以是一个字符串，也可以是一个对象。

```
worker.postMessage("Hello World");
worker.postMessage({method: 'echo', args: ['Work']});
```

## 子线程的事件监听

在子线程内，必须有一个回调函数，监听 message 事件。

```
/* File: work.js */

self.addEventListener('message', function(e) {
  self.postMessage('You said: ' + e.data);
}, false);
```

self 代表子线程自身，self.addEventListener 表示对子线程的 message 事件指定回调函数（直接指定 onmessage 属性的值也可）。回调函数的参数是一个事件对象，它的 data 属性包含主线程发来的信号。self.postMessage 则表示，子线程向主线程发送一个信号。

根据主线程发来的不同的信号值，子线程可以调用不同的方法。

```
/* File: work.js */

self.onmessage = function(event) {
  var method = event.data.method;
  var args = event.data.args;

  var reply = doSomething(args);
  self.postMessage({method: method, reply: reply});
};
```

## 主线程的事件监听

主线程也必须指定 message 事件的回调函数，监听子线程发来的信号。

```
/* File: main.js */

worker.addEventListener('message', function(e) {
    console.log(e.data);
}, false);
```

## 错误处理

主线程可以监听子线程是否发生错误。如果发生错误，会触发主线程的 error 事件。

```
worker.onerror(function(event) {
  console.log(event);
});

// or

worker.addEventListener('error', function(event) {
  console.log(event);
});
```

## 关闭子线程

使用完毕之后，为了节省系统资源，我们必须在主线程调用 terminate 方法，手动关闭子线程。

```
worker.terminate();
```

也可以子线程内部关闭自身。

```
self.close();
```

## 主线程与子线程的数据通信

前面说过，主线程与子线程之间的通信内容，可以是文本，也可以是对象。需要注意的是，这种通信是拷贝关系，即是传值而不是传址，子线程对通信内容的修改，不会影响到主线程。事实上，浏览器内部的运行机制是，先将通信内容串行化，然后把串行化后的字符串发给子线程，后者再将它还原。

主线程与子线程之间也可以交换二进制数据，比如 File、Blob、ArrayBuffer 等对象，也可以在线程之间发送。

但是，用拷贝方式发送二进制数据，会造成性能问题。比如，主线程向子线程发送一个 500MB 文件，默认情况下浏览器会生成一个原文件的拷贝。为了解决这个问题，JavaScript 允许主线程把二进制数据直接转移给子线程，但是一旦转移，主线程就无法再使用这些二进制数据了，这是为了防止出现多个线程同时修改数据的麻烦局面。这种转移数据的方法，叫做[Transferable Objects](http://www.w3.org/html/wg/drafts/html/master/infrastructure.html#transferable-objects)。

如果要使用该方法，postMessage 方法的最后一个参数必须是一个数组，用来指定前面发送的哪些值可以被转移给子线程。

```
worker.postMessage(arrayBuffer, [arrayBuffer]);
window.postMessage(arrayBuffer, targetOrigin, [arrayBuffer]);
```

## 同页面的 Web Worker

通常情况下，子线程载入的是一个单独的 JavaScript 文件，但是也可以载入与主线程在同一个网页的代码。假设网页代码如下：

```
<!DOCTYPE html>
    <body>
        <script id="worker" type="app/worker">

            addEventListener('message', function() {
                postMessage('Im reading Tech.pro');
            }, false);
        </script>
    </body>
</html>
```

我们可以读取页面中的 script，用 worker 来处理。

```
var blob = new Blob([document.querySelector('#worker').textContent]);
```

这里需要把代码当作二进制对象读取，所以使用 Blob 接口。然后，这个二进制对象转为 URL，再通过这个 URL 创建 worker。

```
var url = window.URL.createObjectURL(blob);

var worker = new Worker(url);
```

部署事件监听代码。

```
worker.addEventListener('message', function(e) {
   console.log(e.data);
}, false);
```

最后，启动 worker。

```
worker.postMessage('');
```

整个页面的代码如下：

```
<!DOCTYPE html>
    <body>
        <script id="worker" type="app/worker">

            addEventListener('message', function() {
                postMessage('Work done!');
            }, false);

        </script>

        <script>
            (function() {

                var blob = new Blob([document.querySelector('#worker').textContent]);

                var url = window.URL.createObjectURL(blob);

                var worker = new Worker(url);

                worker.addEventListener('message', function(e) {
                    console.log(e.data);
                }, false);

                worker.postMessage('');
            })();
        </script>
    </body>
</html>
```

可以看到，主线程和子线程的代码都在同一个网页上面。

上面所讲的 Web Worker 都是专属于某个网页的，当该网页关闭，worker 就自动结束。除此之外，还有一种共享式的 Web Worker，允许多个浏览器窗口共享同一个 worker，只有当所有网口关闭，它才会结束。这种共享式的 Worker 用 SharedWorker 对象来建立，因为适用场合不多，这里就省略了。

## 参考链接

*   Matt West, [Using Web Workers to Speed-Up Your JavaScript Applications](http://blog.teamtreehouse.com/using-web-workers-to-speed-up-your-javascript-applications)
*   Eric Bidelman, [The Basics of Web Workers](http://www.html5rocks.com/en/tutorials/workers/basics/)
*   Eric Bidelman, [Transferable Objects: Lightning Fast!](http://updates.html5rocks.com/2011/12/Transferable-Objects-Lightning-Fast)
*   Jesse Cravens, [Web Worker Patterns](http://tech.pro/tutorial/1487/web-worker-patterns)
*   Bipin Joshi, [7 Things You Need To Know About Web Workers](http://www.developer.com/lang/jscript/7-things-you-need-to-know-about-web-workers.html)