# 9.1 Promise 对象

*   JavaScript 的异步执行
    *   概述
    *   回调函数
    *   事件监听
    *   发布/订阅
    *   异步操作的流程控制
        *   串行执行
        *   并行执行
        *   并行与串行的结合
    *   Promise 对象
        *   简介
        *   Promise 接口
        *   用法辨析
        *   Promises 对象的实现
        *   实例：Ajax 操作
        *   小结
    *   参考链接

Promise 是 JavaScript 异步操作解决方案。介绍 Promise 之前，先对异步操作做一个详细介绍。

## JavaScript 的异步执行

### 概述

Javascript 语言的执行环境是"单线程"（single thread）。所谓"单线程"，就是指一次只能完成一件任务。如果有多个任务，就必须排队，前面一个任务完成，再执行后面一个任务。

这种模式的好处是实现起来比较简单，执行环境相对单纯；坏处是只要有一个任务耗时很长，后面的任务都必须排队等着，会拖延整个程序的执行。常见的浏览器无响应（假死），往往就是因为某一段 Javascript 代码长时间运行（比如死循环），导致整个页面卡在这个地方，其他任务无法执行。

JavaScript 语言本身并不慢，慢的是读写外部数据，比如等待 Ajax 请求返回结果。这个时候，如果对方服务器迟迟没有响应，或者网络不通畅，就会导致脚本的长时间停滞。

为了解决这个问题，Javascript 语言将任务的执行模式分成两种：同步（Synchronous）和异步（Asynchronous）。"同步模式"就是传统做法，后一个任务等待前一个任务结束，然后再执行，程序的执行顺序与任务的排列顺序是一致的、同步的。这往往用于一些简单的、快速的、不涉及读写的操作。

"异步模式"则完全不同，每一个任务分成两段，第一段代码包含对外部数据的请求，第二段代码被写成一个回调函数，包含了对外部数据的处理。第一段代码执行完，不是立刻执行第二段代码，而是将程序的执行权交给第二个任务。等到外部数据返回了，再由系统通知执行第二段代码。所以，程序的执行顺序与任务的排列顺序是不一致的、异步的。

以下总结了"异步模式"编程的几种方法，理解它们可以让你写出结构更合理、性能更出色、维护更方便的 JavaScript 程序。

### 回调函数

回调函数是异步编程最基本的方法。

假定有两个函数 f1 和 f2，后者等待前者的执行结果。

```
f1();

f2();
```

如果 f1 是一个很耗时的任务，可以考虑改写 f1，把 f2 写成 f1 的回调函数。

```
function f1(callback){
  setTimeout(function () {
    // f1 的任务代码
    callback();
  }, 1000);
}
```

执行代码就变成下面这样：

```
f1(f2);
```

采用这种方式，我们把同步操作变成了异步操作，f1 不会堵塞程序运行，相当于先执行程序的主要逻辑，将耗时的操作推迟执行。

回调函数的优点是简单、容易理解和部署，缺点是不利于代码的阅读和维护，各个部分之间高度[耦合](http://en.wikipedia.org/wiki/Coupling_(computer_programming))（Coupling），使得程序结构混乱、流程难以追踪（尤其是回调函数嵌套的情况），而且每个任务只能指定一个回调函数。

### 事件监听

另一种思路是采用事件驱动模式。任务的执行不取决于代码的顺序，而取决于某个事件是否发生。

还是以 f1 和 f2 为例。首先，为 f1 绑定一个事件（这里采用的 jQuery 的[写法](http://api.jquery.com/on/)）。

```
f1.on('done', f2);
```

上面这行代码的意思是，当 f1 发生 done 事件，就执行 f2。然后，对 f1 进行改写：

```
function f1(){
    setTimeout(function () {
        // f1 的任务代码
        f1.trigger('done');
    }, 1000);
}
```

f1.trigger('done')表示，执行完成后，立即触发 done 事件，从而开始执行 f2。

这种方法的优点是比较容易理解，可以绑定多个事件，每个事件可以指定多个回调函数，而且可以"[去耦合](http://en.wikipedia.org/wiki/Decoupling)"（Decoupling），有利于实现模块化。缺点是整个程序都要变成事件驱动型，运行流程会变得很不清晰。

### 发布/订阅

"事件"完全可以理解成"信号"，如果存在一个"信号中心"，某个任务执行完成，就向信号中心"发布"（publish）一个信号，其他任务可以向信号中心"订阅"（subscribe）这个信号，从而知道什么时候自己可以开始执行。这就叫做"[发布/订阅模式](http://en.wikipedia.org/wiki/Publish-subscribe_pattern)"（publish-subscribe pattern），又称"[观察者模式](http://en.wikipedia.org/wiki/Observer_pattern)"（observer pattern）。

这个模式有多种[实现](http://msdn.microsoft.com/en-us/magazine/hh201955.aspx)，下面采用的是 Ben Alman 的[Tiny Pub/Sub](https://gist.github.com/661855)，这是 jQuery 的一个插件。

首先，f2 向"信号中心"jQuery 订阅"done"信号。

```
jQuery.subscribe("done", f2);
```

然后，f1 进行如下改写：

```
function f1(){
    setTimeout(function () {
        // f1 的任务代码
        jQuery.publish("done");
    }, 1000);
}
```

jQuery.publish("done")的意思是，f1 执行完成后，向"信号中心"jQuery 发布"done"信号，从而引发 f2 的执行。

f2 完成执行后，也可以取消订阅（unsubscribe）。

```
jQuery.unsubscribe("done", f2);
```

这种方法的性质与"事件监听"类似，但是明显优于后者。因为我们可以通过查看"消息中心"，了解存在多少信号、每个信号有多少订阅者，从而监控程序的运行。

## 异步操作的流程控制

如果有多个异步操作，就存在一个流程控制的问题：确定操作执行的顺序，以后如何保证遵守这种顺序。

```
function async(arg, callback) {
  console.log('参数为 ' + arg +' , 1 秒后返回结果');
  setTimeout(function() { callback(arg * 2); }, 1000);
}
```

上面代码的 async 函数是一个异步任务，非常耗时，每次执行需要 1 秒才能完成，然后再调用回调函数。

如果有 6 个这样的异步任务，需要全部完成后，才能执行下一步的 final 函数。

```
function final(value) {
  console.log('完成: ', value);
}
```

请问应该如何安排操作流程？

```
async(1, function(value){
  async(value, function(value){
    async(value, function(value){
      async(value, function(value){
        async(value, function(value){
          async(value, final);
        });
      });
    });
  });
});
```

上面代码采用 6 个回调函数的嵌套，不仅写起来麻烦，容易出错，而且难以维护。

### 串行执行

我们可以编写一个流程控制函数，让它来控制异步任务，一个任务完成以后，再执行另一个。这就叫串行执行。

```
var items = [ 1, 2, 3, 4, 5, 6 ];
var results = [];
function series(item) {
  if(item) {
    async( item, function(result) {
      results.push(result);
      return series(items.shift());
    });
  } else {
    return final(results);
  }
}
series(items.shift());
```

上面代码中，函数 series 就是串行函数，它会依次执行异步任务，所有任务都完成后，才会执行 final 函数。items 数组保存每一个异步任务的参数，results 数组保存每一个异步任务的运行结果。

### 并行执行

流程控制函数也可以是并行执行，即所有异步任务同时执行，等到全部完成以后，才执行 final 函数。

```
var items = [ 1, 2, 3, 4, 5, 6 ];
var results = [];

items.forEach(function(item) {
  async(item, function(result){
    results.push(result);
    if(results.length == items.length) {
      final(results);
    }
  })
});
```

上面代码中，forEach 方法会同时发起 6 个异步任务，等到它们全部完成以后，才会执行 final 函数。

并行执行的好处是效率较高，比起串行执行一次只能执行一个任务，较为节约时间。但是问题在于如果并行的任务较多，很容易耗尽系统资源，拖慢运行速度。因此有了第三种流程控制方式。

### 并行与串行的结合

所谓并行与串行的结合，就是设置一个门槛，每次最多只能并行执行 n 个异步任务。这样就避免了过分占用系统资源。

```
var items = [ 1, 2, 3, 4, 5, 6 ];
var results = [];
var running = 0;
var limit = 2;

function launcher() {
  while(running < limit && items.length > 0) {
    var item = items.shift();
    async(item, function(result) {
      results.push(result);
      running--;
      if(items.length > 0) {
        launcher();
      } else if(running == 0) {
        final();
      }
    });
    running++;
  }
}

launcher();
```

上面代码中，最多只能同时运行两个异步任务。变量 running 记录当前正在运行的任务数，只要低于门槛值，就再启动一个新的任务，如果等于 0，就表示所有任务都执行完了，这时就执行 final 函数。

## Promise 对象

### 简介

Promises 对象是 CommonJS 工作组提出的一种规范，目的是为异步操作提供[统一接口](http://wiki.commonjs.org/wiki/Promises/A)。

那么，什么是 Promises？首先，它是一个对象，也就是说与其他 JavaScript 对象的用法，没有什么两样；其次，它起到代理作用（proxy），使得异步操作具备同步操作（synchronous code）的接口，即充当异步操作与回调函数之间的中介，使得程序具备正常的同步运行的流程，回调函数不必再一层层包裹起来。

简单说，它的思想是，每一个异步任务立刻返回一个 Promise 对象，由于是立刻返回，所以可以采用同步操作的流程。这个 Promises 对象有一个 then 方法，允许指定回调函数，在异步任务完成后调用。比如，f1 的回调函数 f2,可以写成：

```
(new Promise(f1)).then(f2);
```

这种写法对于嵌套的回调函数尤其有用。

```
// 传统写法

step1(function (value1) {
    step2(value1, function(value2) {
        step3(value2, function(value3) {
            step4(value3, function(value4) {
                // ...
            });
        });
    });
});

// Promises 的写法

(new Promise(step1))
.then(step2)
.then(step3)
.then(step4);
```

从上面代码可以看到，采用 Promises 接口以后，程序流程变得非常清楚，十分易读。

总的来说，传统的回调函数写法使得代码混成一团，变得横向发展而不是向下发展。Promises 规范就是为了解决这个问题而提出的，目标是使用正常的程序流程（同步），来处理异步操作。它先返回一个 Promise 对象，后面的操作以同步的方式，寄存在这个对象上面。等到异步操作有了结果，再执行前期寄放在它上面的其他操作。

Promises 原本只是社区提出的一个构想，一些外部函数库率先实现了这个功能。ECMAScript 6 将其写入语言标准，Chrome 和 Firefox 浏览器都已经部署了这个功能。

### Promise 接口

当异步任务返回一个 promise 对象（小写表示这是 Promise 的实例）时，该对象只有三种状态：未完成（pending）、已完成（fulfilled）、失败（rejected）。

这三种的状态的变化途径只有两个，且只能发生一次：从“未完成”到“已完成”，或者从“未完成”到“失败”。一旦当前状态变为“已完成”或“失败”，就意味着不会再发生状态变化了。

Promise 对象的运行结果，最终只有两种。

*   得到一个值，状态变为 fulfilled
*   抛出一个错误，状态变为 rejected

promise 对象的 then 方法用来添加回调函数。它可以接受两个回调函数，第一个是操作成功（fulfilled）时的回调函数，第二个是操作失败（rejected）时的回调函数（可以不提供）。一旦状态改变，就调用相应的回调函数。

```
(new Promise(step1))
.then(step2)
.then(step3)
.then(step4)
.then(console.log, console.error);
```

再来看上面的代码就很清楚，step1 是一个耗时很长的异步任务，然后使用 then 方法，依次绑定了三个 step1 操作成功后的回调函数 step2、step3、step4，最后再用 then 方法绑定两个回调函数：操作成功时的回调函数 console.log，操作失败时的回调函数 console.error。

console.log 和 console.error 这两个最后的回调函数，用法上有一点重要的区别。console.log 只显示回调函数 step4 的返回值，而 console.error 可以显示 step2、step3、step4 之中任何一个发生的错误。也就是说，假定 step2 操作失败，抛出一个错误，这时 step3 和 step4 都不会再运行了，promises 对象开始寻找接下来的第一个错误回调函数，在上面代码中是 console.error。所以，结论就是 Promises 对象的错误有传递性。

换言之，上面的代码等同于下面的形式。

```
try {
  var v1 = step1();
  var v2 = step2(v1);
  var v3 = step3(v2);
  var v4 = step4(v3);
  console.log(v4);
} catch (error) {
  console.error(error);
}
```

上面代码表示，try 部分任何一步的错误，都会被 catch 部分捕获，并导致整个 Promise 操作的停止。

### 用法辨析

Promise 的用法，简单说就是一句话：使用 then 方法添加回调函数。但是，不同的写法有一些细微的差别，请看下面四种写法，它们的差别在哪里？

```
// 写法一
doSomething().then(function () {
  return doSomethingElse();
});

// 写法二
doSomething().then(function () {
  doSomethingElse();
});

// 写法三
doSomething().then(doSomethingElse());

// 写法四
doSomething().then(doSomethingElse);
```

为了便于解释，上面四种写法都再用 then 方法接一个回调函数。

写法一的 finalHandler 回调函数的参数，是 doSomethingElse 函数的运行结果。

```
doSomething().then(function () {
  return doSomethingElse();
}).then(finalHandler);
```

写法二的 finalHandler 回调函数的参数，是 undefined。

```
doSomething().then(function () {
  doSomethingElse();
}).then(finalHandler);
```

写法三的 finalHandler 回调函数的参数，是 doSomethingElse 函数返回的回调函数的运行结果。

```
doSomething().then(doSomethingElse())
  .then(finalHandler);
```

写法四与写法一只有一个差别，那就是 doSomethingElse 会接收到`doSomething()`返回的结果。

```
doSomething().then(doSomethingElse)
  .then(finalHandler);
```

### Promises 对象的实现

为了真正理解 Promise 对象，下面我们自己动手写一个 Promise 的实现。

首先，将 Promise 定义成构造函数。

```
var Promise = function () {
  this.state = 'pending';
  this.thenables = [];
};
```

上面代码表示，Promise 的实例对象的 state 属性默认为“未完成”状态（pending），还有一个 thenables 属性指向一个数组，用来存放 then 方法生成的内部对象。

接下来，部署实例对象的 resolve 方法，该方法用来将实例对象的状态从“未完成”变为“已完成”。

```
Promise.prototype.resolve = function (value) {
  if (this.state != 'pending') return;

  this.state = 'fulfilled';
  this.value = value;
  this._handleThen();
  return this;
}
```

上面代码除了改变实例的状态，还将异步任务的返回值存入实例对象的 value 属性，然后调用内部方法 _handleThen，最后返回实例对象本身。

类似地，部署实例对象的 reject 方法。

```
Promise.prototype.reject = function (reason) {
  if (this.state != 'pending') return;

  this.state = 'rejected';
  this.reason = reason;
  this._handleThen();
  return this;
};
```

然后，部署实例对象的 then 方法。它接受两个参数，分别是异步任务成功时的回调函数（onFulfilled）和出错时的回调函数（onRejected）。为了可以部署链式操作，它必须返回一个新的 Promise 对象。

```
Promise.prototype.then = function (onFulfilled, onRejected) {
  var thenable = {};

  if (typeof onFulfilled == 'function') {
    thenable.fulfill = onFulfilled;
  };

  if (typeof onRejected == 'function') {
    thenable.reject = onRejected;
  };

  if (this.state != 'pending') {
    setImmediate(function () {
      this._handleThen();
    }.bind(this));
  }

  thenable.promise = new Promise();
  this.thenables.push(thenable);

  return thenable.promise;
}
```

上面代码首先定义了一个内部变量 thenable 对象，将 then 方法的两个参数都加入这个对象的属性。然后，检查当前状态，如果不等于“未完成”，则在当前操作结束后，立即调用 _handleThen 方法。接着，在 thenable 对象的 promise 属性上生成一个新的 Promise 对象，并在稍后返回这个对象。最后，将 thenable 对象加入实例对象的 thenables 数组。

下一步就要部署内部方法 _handleThen，它用来处理通过 then 方法绑定的回调函数。

```
Promise.prototype._handleThen = function () {
  if (this.state === 'pending') return;

  if (this.thenables.length) {
    for (var i = 0; i < this.thenables.length; i++) {
      var thenPromise = this.thenables[i].promise;
      var returnedVal;
      try {
        // 运行回调函数
      } catch (e) {
        thenPromise.reject(e);
      }
    }
    this.thenables = [];
  }
}
```

上面代码的逻辑是这样的：如果实例对象的状态是“未完成”，就返回，否则检查 thenables 属性是否有值。如果有值，表明里面储存了需要执行的回调函数，则依次运行回调函数。运行完成后，清空 thenables 属性。

之所以把回调函数的执行放在 try...catch 结构中，是因为一旦出错，就会自动执行 catch 代码块，从而可以运行下一个 Promise 实例对象的 reject 方法，这使得调用 reject 方法变得很简单。下面是 try 代码块中的代码。

```
try {
  switch (this.state) {
    case 'fulfilled':
      if (this.thenables[i].fulfill) {
        returnedVal = this.thenables[i].fulfill(this.value);
      } else {
        thenPromise.resolve(this.value);
      }
      break;
    case 'rejected':
      if (this.thenables[i].reject) {
        returnedVal = this.thenables[i].reject(this.reason);
      } else {
        thenPromise.reject(this.reason);
      }
      break;
  }
  if (returnedVal === null) {
    this.thenables[i].promise.resolve(returnedVal);
  } else if (returnedVal instanceof Promise || typeof returnedVal.then === 'function') {
    returnedVal.then(thenPromise.resolve.bind(thenPromise), thenPromise.reject.bind(thenPromise));
  } else {
    this.thenables[i].promise.resolve(returnedVal);
  }
}
```

上面代码首先根据实例对象的状态，分别调用 fulfill 或 reject 回调函数，并传入相应的参数，并将返回值存入 returnVal 变量。然后再去改变 this.thenables[i].promise 对象的状态，触发下一个 Promise 对象的 resolve 或者 reject 方法。

针对 returnedVal 的判断，目的是如果 returnedVal 是一个 Promise 对象，那么后面的回调函数都要改为绑定在它上面。

最后，由于我们写的是供调用的函数库，需要将构造函数输出。

```
module.exports = Promise;
```

### 实例：Ajax 操作

Ajax 操作是典型的异步操作，传统上往往写成下面这样。

```
function search(term, onload, onerror) {
    var xhr, results, url;

    url = 'http://example.com/search?q=' + term;

    xhr = new XMLHttpRequest();
    xhr.open('GET', url, true);

    xhr.onload = function (e) {
        if (this.status === 200) {
            results = JSON.parse(this.responseText);
            onload(results);
        }
    };

    xhr.onerror = function (e) {
        onerror(e);
    };

    xhr.send();
}

search("Hello World", f1, f2);
```

上面代码的回调函数，必须直接传入。如果使用 Promises 方法，就可以写成下面这样。

```
function search(term) {

    var url = 'http://example.com/search?q=' + term;
    var p = new Promise();
    var xhr = new XMLHttpRequest();
    var result;

    xhr.open('GET', url, true);

    xhr.onload = function (e) {
        if (this.status === 200) {
            results = JSON.parse(this.responseText);
            p.resolve(results);
        }
    };

    xhr.onerror = function (e) {
        p.reject(e);
    };

    xhr.send();

    return p;
}

search("Hello World").then(f1, f2);
```

用了 Promises 以后，回调函数就可以用 then 方法加载。

### 小结

Promises 的优点在于，让回调函数变成了规范的链式写法，程序流程可以看得很清楚。它的一整套接口，可以实现许多强大的功能，比如为多个异步操作部署一个回调函数、为多个回调函数中抛出的错误统一指定处理方法等等。

而且，它还有一个前面三种方法都没有的好处：如果一个任务已经完成，再添加回调函数，该回调函数会立即执行。所以，你不用担心是否错过了某个事件或信号。这种方法的缺点就是，编写和理解都相对比较难。

实际可以使用的 Promises 实现，参见 jQuery 的 deferred 对象一节。

## 参考链接

*   Sebastian Porto, [Asynchronous JS: Callbacks, Listeners, Control Flow Libs and Promises](http://sporto.github.com/blog/2012/12/09/callbacks-listeners-promises/)
*   Rhys Brett-Bowen, [Promises/A+ - understanding the spec through implementation](http://modernjavascript.blogspot.com/2013/08/promisesa-understanding-by-doing.html)
*   Matt Podwysocki, Amanda Silver, [Asynchronous Programming in JavaScript with “Promises”](http://blogs.msdn.com/b/ie/archive/2011/09/11/asynchronous-programming-in-javascript-with-promises.aspx)
*   Marc Harter, [Promise A+ Implementation](https://gist.github.com//wavded/5692344)
*   Bryan Klimt, [What’s so great about JavaScript Promises?](http://blog.parse.com/2013/01/29/whats-so-great-about-javascript-promises/)
*   Jake Archibald, [JavaScript Promises There and back again](http://www.html5rocks.com/en/tutorials/es6/promises/)
*   Mikito Takada, [7\. Control flow, Mixu's Node book](http://book.mixu.net/node/ch7.html)