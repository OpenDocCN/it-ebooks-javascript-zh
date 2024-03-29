# 6.1 浏览器的 JavaScript 引擎

*   JavaScript 代码嵌入网页的方法
    *   直接添加代码块
    *   加载外部脚本
    *   行内代码
    *   外部脚本的加载
        *   网页底部加载
        *   多个脚本的加载
        *   defer 属性
        *   async 属性
        *   脚本的动态嵌入
        *   加载使用的协议
    *   JavaScript 虚拟机
    *   单线程模型
    *   Event Loop
    *   任务队列
    *   参考链接

浏览器通过内置的 JavaScript 引擎，读取网页中的代码，对其处理后运行。

## JavaScript 代码嵌入网页的方法

在网页中嵌入 JavaScript 代码有多种方法。

### 直接添加代码块

通过 script 标签，可以直接将 JavaScript 代码嵌入网页。

```js
<script>
// some JavaScript code
</script>
```

### 加载外部脚本

script 标签也可以指定加载外部的脚本文件。

```js
<script src="example.js"></script>
```

如果脚本文件使用了非英语字符，还应该注明编码。

```js
<script charset="utf-8" src="example.js"></script>
```

加载外部脚本和直接添加代码块，这两种方法不能混用。下面代码的 console.log 语句直接被忽略。

```js
<script charset="utf-8" src="example.js">
  console.log('Hello World!');
</script>
```

### 行内代码

除了上面两种方法，HTML 语言允许在某些元素的事件属性和 a 元素的 href 属性中，直接写入 JavaScript。

```js
<div onclick="alert('Hello')"></div>

<a href="javascript:alert('Hello')"></a>
```

这种写法将 HTML 代码与 JavaScript 代码混写在一起，非常不利于代码管理，不建议使用。

## 外部脚本的加载

### 网页底部加载

正常的网页加载流程是这样的。

1.  浏览器一边下载 HTML 网页，一边开始解析
2.  解析过程中，发现 script 标签
3.  暂停解析，下载 script 标签中的外部脚本
4.  下载完成，执行脚本
5.  恢复往下解析 HTML 网页

也就是说，加载外部脚本时，浏览器会暂停页面渲染，等待脚本下载并执行完成后，再继续渲染。如果加载时间很长（比如一直无法完成下载），就会造成网页长时间失去响应，浏览器就会呈现“假死”状态，失去响应，这被称为“阻塞效应”。这样设计是因为 JavaScript 代码可能会修改页面，所以必须等它执行完才能继续渲染。为了避免这种情况，较好的做法是将 script 标签都放在页面底部，而不是头部。当然，如果某些脚本代码非常重要，一定要放在页面头部的话，最好直接将代码嵌入页面，而不是连接外部脚本文件，这样能缩短加载时间。

将脚本文件都放在网页尾部加载，还有一个好处。在 DOM 结构生成之前就调用 DOM，JavaScript 会报错，如果脚本都在网页尾部加载，就不存在这个问题，因为这时 DOM 肯定已经生成了。

```js
<head>
    <script>
        console.log(document.body.innerHTML); 
    </script>
</head>
```

上面代码执行时会报错，因为此时 body 元素还未生成。

一种解决方法是设定 DOMContentLoaded 事件的回调函数。

```js
<head>
    <script>
        document.addEventListener("DOMContentLoaded", function(event) {
            console.log(document.body.innerHTML);
         });
    </script>
</head>
```

另一种解决方法是，使用 script 标签的 onload 属性。当 script 标签指定的外部脚本文件下载和解析完成，会触发一个 load 事件，可以为这个事件指定回调函数。

```js
<script src="jquery.min.js" onload="console.log(document.body.innerHTML)">
</script>
```

但是，如果将脚本放在页面底部，就可以完全按照正常的方式写，上面两种方式都不需要。

```js
<body>
    <!-- 其他代码  -->
    <script>
        console.log(document.body.innerHTML);
    </script>
</body>
```

### 多个脚本的加载

如果有多个 script 标签，比如下面这样。

```js
<script src="1.js"></script>
<script src="2.js"></script>
```

浏览器会同时平行下载 1.js 和 2.js，但是执行时会保证先执行 1.js，然后再执行 2.js，即使后者先下载完成，也是如此。也就是说，脚本的执行顺序由它们在页面中的出现顺序决定，这是为了保证脚本之间的依赖关系不受到破坏。

当然，加载这两个脚本都会产生“阻塞效应”，必须等到它们都加载完成，浏览器才会继续页面渲染。

此外，对于来自同一个域名的资源，比如脚本文件、样式表文件、图片文件等，浏览器一般最多同时下载六个。如果是来自不同域名的资源，就没有这个限制。所以，通常把静态文件放在不同的域名之下，以加快下载速度。

### defer 属性

为了解决脚本文件下载阻塞网页渲染的问题，一个方法是加入 defer 属性。

```js
<script src="1.js" defer></script>
<script src="2.js" defer></script>
```

defer 属性的运行过程是这样的。

1.  浏览器开始解析 HTML 网页
2.  解析过程中，发现带有 defer 属性的 script 标签
3.  浏览器继续往下解析 HTML 网页，同时并行下载 script 标签中的外部脚本
4.  浏览器完成解析 HTML 网页，此时再执行下载的脚本

有了 defer 属性，浏览器下载脚本文件的时候，不会阻塞页面渲染。下载的脚本文件在 DOMContentLoaded 事件触发前执行（即刚刚读取完标签），而且可以保证执行顺序就是它们在页面上出现的顺序。但是，浏览器对这个属性的支持不够理想，IE（<=9）还有一个 bug，无法保证 2.js 一定在 1.js 之后执行。如果需要支持老版本的 IE，且脚本之间有依赖关系，建议不要使用 defer 属性。

对于内置而不是连接外部脚本的 script 标签，以及动态生成的 script 标签，defer 属性不起作用。

### async 属性

解决“阻塞效应”的另一个方法是加入 async 属性。

```js
<script src="1.js" async></script>
<script src="2.js" async></script>
```

async 属性的运行过程是这样的。

1.  浏览器开始解析 HTML 网页
2.  解析过程中，发现带有 async 属性的 script 标签
3.  浏览器继续往下解析 HTML 网页，同时并行下载 script 标签中的外部脚本
4.  脚本下载完成，浏览器暂停解析 HTML 网页，开始执行下载的脚本
5.  脚本执行完毕，浏览器恢复解析 HTML 网页

async 属性可以保证脚本下载的同时，浏览器继续渲染。需要注意的是，一旦采用这个属性，就无法保证脚本的执行顺序。哪个脚本先下载结束，就先执行那个脚本。使用 async 属性的脚本文件中，不应该使用 document.write 方法。IE 10 支持 async 属性，低于这个版本的 IE 都不支持。

defer 属性和 async 属性到底应该使用哪一个？一般来说，如果脚本之间没有依赖关系，就使用 async 属性，如果脚本之间有依赖关系，就使用 defer 属性。如果同时使用 async 和 defer 属性，后者不起作用，浏览器行为由 async 属性决定。

### 脚本的动态嵌入

除了用静态的 script 标签，还可以动态嵌入 script 标签。

```js
['1.js', '2.js'].forEach(function(src) {
  var script = document.createElement('script');
  script.src = src;
  document.head.appendChild(script);
});
```

这种方法的好处是，动态生成的 script 标签不会阻塞页面渲染，也就不会造成浏览器假死。但是问题在于，这种方法无法保证脚本的执行顺序，哪个脚本文件先下载完成，就先执行哪个。

如果想避免这个问题，可以设置 async 属性为 false。

```js
['1.js', '2.js'].forEach(function(src) {
  var script = document.createElement('script');
  script.src = src;
  script.async = false;
  document.head.appendChild(script);
});
```

上面的代码依然不会阻塞页面渲染，而且可以保证 2.js 在 1.js 后面执行。不过需要注意的是，在这段代码后面加载的脚本文件，会因此都等待 2.js 执行完成后再执行。

我们可以把上面的写法，封装成一个函数。

```js
(function() {
  var script,
  scripts = document.getElementsByTagName('script')[0];
  function load(url) {
    script = document.createElement('script');
    script.async = true;
    script.src = url;
    scripts.parentNode.insertBefore(script, scripts);
  }
  load('//apis.google.com/js/plusone.js');
  load('//platform.twitter.com/widgets.js');
  load('//s.thirdpartywidget.com/widget.js');
}());
```

此外，动态嵌入还有一个地方需要注意。动态嵌入必须等待 CSS 文件加载完成后，才会去下载外部脚本文件。静态加载就不存在这个问题，script 标签指定的外部脚本文件，都是与 CSS 文件同时并发下载的。

### 加载使用的协议

如果不指定协议，浏览器默认采用 HTTP 协议下载。

```js
<script src="example.js"></script>
```

上面的 example.js 默认就是采用 http 协议下载，如果要采用 HTTPs 协议下载，必需写明（假定服务器支持）。

```js
<script src="https://example.js"></script>
```

但是有时我们会希望，根据页面本身的协议来决定加载协议，这时可以采用下面的写法。

```js
<script src="//example.js"></script>
```

## JavaScript 虚拟机

JavaScript 是一种解释型语言，也就是说，它不需要编译，可以由解释器实时运行。这样的好处是运行和修改都比较方便，刷新页面就可以重新解释；缺点是每次运行都要调用解释器，系统开销较大，运行速度慢于编译型语言。为了提高运行速度，目前的浏览器都将 JavaScript 进行一定程度的编译，生成类似字节码（bytecode）的中间代码，以提高运行速度。

早期，浏览器内部对 JavaScript 的处理过程如下：

1.  读取代码，进行词法分析（Lexical analysis），将代码分解成词元（token）。
2.  对词元进行语法分析（parsing），将代码整理成“语法树”（syntax tree）。
3.  使用“翻译器”（translator），将代码转为字节码（bytecode）。
4.  使用“字节码解释器”（bytecode interpreter），将字节码转为机器码。

逐行解释将字节码转为机器码，是很低效的。为了提高运行速度，现代浏览器改为采用“即时编译”（Just In Time compiler，缩写 JIT），即字节码只在运行时编译，用到哪一行就编译哪一行，并且把编译结果缓存（inline cache）。通常，一个程序被经常用到的，只是其中一小部分代码，有了缓存的编译结果，整个程序的运行速度就会显著提升。

不同的浏览器有不同的编译策略。有的浏览器只编译最经常用到的部分，比如循环的部分；有的浏览器索性省略了字节码的翻译步骤，直接编译成机器码，比如 chrome 浏览器的 V8 引擎。

字节码不能直接运行，而是运行在一个虚拟机（Virtual Machine）之上，一般也把虚拟机称为 JavaScript 引擎。因为 JavaScript 运行时未必有字节码，所以 JavaScript 虚拟机并不完全基于字节码，而是部分基于源码，即只要有可能，就通过 JIT（just in time）编译器直接把源码编译成机器码运行，省略字节码步骤。这一点与其他采用虚拟机（比如 Java）的语言不尽相同。这样做的目的，是为了尽可能地优化代码、提高性能。下面是目前最常见的一些 JavaScript 虚拟机：

*   Chakra)(Microsoft]([`en.wikipedia.org/wiki/Chakra_(JScript_engine%5C))(Microsoft`](http://en.wikipedia.org/wiki/Chakra_(JScript_engine%5C))(Microsoft)) Internet Explorer)
*   [Nitro/JavaScript Core](http://en.wikipedia.org/wiki/WebKit#JavaScriptCore) (Safari)
*   [Carakan](http://dev.opera.com/articles/view/labs-carakan/) (Opera)
*   [SpiderMonkey](https://developer.mozilla.org/en-US/docs/SpiderMonkey) (Firefox)
*   V8%5D(http://en.wikipedia.org/wiki/V8_(JavaScript_engine%5C))) (Chrome, Chromium)

## 单线程模型

JavaScript 采用单线程模型，也就是说，所有的任务都在一个线程里运行。这意味着，一次只能运行一个任务，其他任务都必须在后面排队等待。

JavaScript 之所以采用单线程，而不是多线程，跟历史有关系。JavaScript 从诞生起就是单线程，原因是不想让浏览器变得太复杂，因为多线程需要共享资源、且有可能修改彼此的运行结果，对于一种网页脚本语言来说，这就太复杂了。比如，假定 JavaScript 同时有两个线程，一个线程在某个 DOM 节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？所以，为了避免复杂性，从一诞生，JavaScript 就是单线程，这已经成了这门语言的核心特征，将来也不会改变。

为了利用多核 CPU 的计算能力，HTML5 提出 Web Worker 标准，允许 JavaScript 脚本创建多个线程，但是子线程完全受主线程控制，且不得操作 DOM。所以，这个新标准并没有改变 JavaScript 单线程的本质。

单线程模型带来了一些问题，主要是新的任务被加在队列的尾部，只有前面的所有任务运行结束，才会轮到它执行。如果有一个任务特别耗时，后面的任务都会停在那里等待，造成浏览器失去响应，又称“假死”。为了避免“假死”，当某个操作在一定时间后仍无法结束，浏览器就会跳出提示框，询问用户是否要强行停止脚本运行。

如果排队是因为计算量大，CPU 忙不过来，倒也算了，但是很多时候 CPU 是闲着的，因为 IO 设备（输入输出设备）很慢（比如 Ajax 操作从网络读取数据），不得不等着结果出来，再往下执行。JavaScript 语言的设计者意识到，这时 CPU 完全可以不管 IO 设备，挂起处于等待中的任务，先运行排在后面的任务。等到 IO 设备返回了结果，再回过头，把挂起的任务继续执行下去。这种机制就是 JavaScript 内部采用的 Event Loop。

## Event Loop

所谓 Event Loop，指的是一种内部循环，用来排列和处理事件，以及执行函数。[Wikipedia](http://en.wikipedia.org/wiki/Event_loop)的定义是：“Event Loop 是一个程序结构，用于等待和发送消息和事件。（a programming construct that waits for and dispatches events or messages in a program.）”

所有任务可以分成两种，一种是同步任务（synchronous），另一种是异步任务（asynchronous）。同步任务指的是，在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务；异步任务指的是，不进入主线程、而进入“任务队列”（task queue）的任务，只有“任务队列”通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

以 Ajax 操作为例，它可以当作同步任务处理，也可以当作异步任务处理，由开发者决定。如果是同步任务，主线程就等着 Ajax 操作返回结果，再往下执行；如果是异步任务，该任务直接进入“任务队列”，主线程跳过 Ajax 操作，直接往下执行，等到 Ajax 操作有了结果，主线程再执行对应的回调函数。

想要理解 Event Loop，就要从程序的运行模式讲起。运行以后的程序叫做"进程"（process），一般情况下，一个进程一次只能执行一个任务。如果有很多任务需要执行，不外乎三种解决方法。

1.  排队。因为一个进程一次只能执行一个任务，只好等前面的任务执行完了，再执行后面的任务。

2.  新建进程。使用 fork 命令，为每个任务新建一个进程。

3.  新建线程。因为进程太耗费资源，所以如今的程序往往允许一个进程包含多个线程，由线程去完成任务。

如果某个任务很耗时，比如涉及很多 I/O（输入/输出）操作，那么线程的运行大概是下面的样子。

![synchronous mode](img/4a60773797eddf935acbdc830652fd1e.jpg)

上图的绿色部分是程序的运行时间，红色部分是等待时间。可以看到，由于 I/O 操作很慢，所以这个线程的大部分运行时间都在空等 I/O 操作的返回结果。这种运行方式称为"同步模式"（synchronous I/O）。

如果采用多线程，同时运行多个任务，那很可能就是下面这样。

![synchronous mode](img/d8542108be7a44ce32d1a0bd2b1b1762.jpg)

上图表明，多线程不仅占用多倍的系统资源，也闲置多倍的资源，这显然不合理。

![asynchronous mode](img/f1d685c251bb190f4503e6df399cf282.jpg)

上图主线程的绿色部分，还是表示运行时间，而橙色部分表示空闲时间。每当遇到 I/O 的时候，主线程就让 Event Loop 线程去通知相应的 I/O 程序，然后接着往后运行，所以不存在红色的等待时间。等到 I/O 程序完成操作，Event Loop 线程再把结果返回主线程。主线程就调用事先设定的回调函数，完成整个任务。

可以看到，由于多出了橙色的空闲时间，所以主线程得以运行更多的任务，这就提高了效率。这种运行方式称为"[异步模式](http://en.wikipedia.org/wiki/Asynchronous_I/O)"（asynchronous I/O）。

这正是 JavaScript 语言的运行方式。单线程模型虽然对 JavaScript 构成了很大的限制，但也因此使它具备了其他语言不具备的优势。如果部署得好，JavaScript 程序是不会出现堵塞的，这就是为什么 node.js 平台可以用很少的资源，应付大流量访问的原因。

## 任务队列

如果有大量的异步任务（实际情况就是这样），它们会在“任务队列”中注册大量的事件。这些事件排成队列，等候进入主线程。本质上，“任务队列”就是一个事件“先进先出”的数据结构。比如，点击鼠标就产生一些列事件，mousedown 事件排在 mouseup 事件前面，mouseup 事件又排在 click 事件的前面。

## 参考链接

*   John Dalziel, [The race for speed part 2: How JavaScript compilers work](http://creativejs.com/2013/06/the-race-for-speed-part-2-how-javascript-compilers-work/)
*   Jake Archibald，[Deep dive into the murky waters of script loading](http://www.html5rocks.com/en/tutorials/speed/script-loading/)
*   Mozilla Developer Network, [window.setTimeout](https://developer.mozilla.org/en-US/docs/Web/API/window.setTimeout)
*   Remy Sharp, [Throttling function calls](http://remysharp.com/2010/07/21/throttling-function-calls/)
*   Ayman Farhat, [An alternative to Javascript's evil setInterval](http://www.thecodeship.com/web-development/alternative-to-javascript-evil-setinterval/)
*   Ilya Grigorik, [Script-injected "async scripts" considered harmful](https://www.igvita.com/2014/05/20/script-injected-async-scripts-considered-harmful/)
*   Axel Rauschmayer, [ECMAScript 6 promises (1/2): foundations](http://www.2ality.com/2014/09/es6-promises-foundations.html)
*   Daniel Imms, [async vs defer attributes](http://www.growingwiththeweb.com/2014/02/async-vs-defer-attributes.html)
*   Craig Buckler, [Load Non-blocking JavaScript with HTML5 Async and Defer](http://www.sitepoint.com/non-blocking-async-defer/)