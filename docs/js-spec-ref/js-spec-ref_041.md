# 6.2 定时器

JavaScript 提供定时执行代码的功能，叫做定时器（timer），主要由 setTimeout()和 setInterval()这两个函数来完成。

*   setTimeout()
*   setInterval()
*   clearTimeout()，clearInterval()
*   运行机制
*   setTimeout(f,0)
    *   含义
    *   应用
    *   参考链接

## setTimeout()

setTimeout 函数用来指定某个函数或某段代码，在多少毫秒之后执行。它返回一个整数，表示定时器的编号，以后可以用来取消这个定时器。

```js
var timerId = setTimeout(func|code, delay)
```

上面代码中，setTimeout 函数接受两个参数，第一个参数`func|code`是将要推迟执行的函数名或者一段代码，第二个参数`delay`是推迟执行的毫秒数。

```js
console.log(1);
setTimeout('console.log(2)',1000);
console.log(3);
```

上面代码的输出结果就是 1，3，2，因为 setTimeout 指定第二行语句推迟 1000 毫秒再执行。

需要注意的是，推迟执行的代码必须以字符串的形式，放入 setTimeout，因为引擎内部使用 eval 函数，将字符串转为代码。如果推迟执行的是函数，则可以直接将函数名，放入 setTimeout。一方面 eval 函数有安全顾虑，另一方面为了便于 JavaScript 引擎优化代码，setTimeout 方法一般总是采用函数名的形式，就像下面这样。

```js
function f(){
  console.log(2);
}

setTimeout(f,1000);

// 或者

setTimeout(function (){console.log(2)},1000);
```

除了前两个参数，setTimeout 还允许添加更多的参数。它们将被传入推迟执行的函数（回调函数）。

```js
setTimeout(function(a,b){
  console.log(a+b);
},1000,1,1);
```

上面代码中，setTimeout 共有 4 个参数。最后那两个参数，将在 1000 毫秒之后回调函数执行时，作为回调函数的参数。

IE 9.0 及以下版本，只允许 setTimeout 有两个参数，不支持更多的参数。这时有三种解决方法。第一种是在一个匿名函数里面，让回调函数带参数运行，再把匿名函数输入 setTimeout。

```js
setTimeout(function() {
  myFunc("one", "two", "three");
}, 1000);
```

上面代码中，myFunc 是真正要推迟执行的函数，有三个参数。如果直接放入 setTimeout，低版本的 IE 不能带参数，所以可以放在一个匿名函数。

第二种解决方法是使用 bind 方法，把多余的参数绑定在回调函数上面，生成一个新的函数输入 setTimeout。

```js
setTimeout(function(arg1){}.bind(undefined, 10), 1000);
```

上面代码中，bind 方法第一个参数是 undefined，表示将原函数的 this 绑定全局作用域，第二个参数是要传入原函数的参数。它运行后会返回一个新函数，该函数不带参数。

第三种解决方法是自定义 setTimeout，使用 apply 方法将参数输入回调函数。

```js
<!--[if lte IE 9]><script>
(function(f){
window.setTimeout =f(window.setTimeout);
window.setInterval =f(window.setInterval);
})(function(f){return function(c,t){
var a=[].slice.call(arguments,2);return f(function(){c.apply(this,a)},t)}
});
</script><![endif]-->
```

除了参数问题，setTimeout 还有一个需要注意的地方：如果被 setTimeout 推迟执行的回调函数是某个对象的方法，那么该方法中的 this 关键字将指向全局环境，而不是定义时所在的那个对象。

```js
var x = 1;

var o = {
  x: 2,
  y: function(){
    console.log(this.x);
  }
};

setTimeout(o.y,1000);
// 1
```

上面代码输出的是 1，而不是 2，这表示 o.y 的 this 所指向的已经不是 o，而是全局环境了。

再看一个不容易发现错误的例子。

```js
function User(login) {
  this.login = login;
  this.sayHi = function() {
    console.log(this.login);
  }
}

var user = new User('John');

setTimeout(user.sayHi, 1000);
```

上面代码只会显示 undefined，因为等到 user.sayHi 执行时，它是在全局对象中执行，所以 this.login 取不到值。

为了防止出现这个问题，一种解决方法是将 user.sayHi 放在函数中执行。

```js
setTimeout(function() {
  user.sayHi();
}, 1000);
```

上面代码中，sayHi 是在 user 作用域内执行，而不是在全局作用域内执行，所以能够显示正确的值。

另一种解决方法是，使用 bind 方法，将绑定 sayHi 绑定在 user 上面。

```js
setTimeout(user.sayHi.bind(user), 1000);
```

HTML 5 标准规定，setTimeout 的最短时间间隔是 4 毫秒。为了节电，对于那些不处于当前窗口的页面，浏览器会将时间间隔扩大到 1000 毫秒。另外，如果笔记本电脑处于电池供电状态，Chrome 和 IE 9 以上的版本，会将时间间隔切换到系统定时器，大约是 15.6 毫秒。

## setInterval()

setInterval 函数的用法与 setTimeout 完全一致，区别仅仅在于 setInterval 指定某个任务每隔一段时间就执行一次，也就是无限次的定时执行。

```js
<input type="button" onclick="clearInterval(timer)" value="stop">

<script>
  var i = 1
  var timer = setInterval(function() {
    console.log(2);
  }, 1000);
</script>
```

上面代码表示每隔 1000 毫秒就输出一个 2，直到用户点击了停止按钮。

与 setTimeout 一样，除了前两个参数，setInterval 方法还可以接受更多的参数，它们会传入回调函数，下面是一个例子。

```js
function f(){
  for (var i=0;i<arguments.length;i++){
    console.log(arguments[i]);
  }
}

setInterval(f, 1000, "Hello World");
// Hello World
// Hello World
// Hello World
// ...
```

如果网页不在浏览器的当前窗口（或 tab），许多浏览器限制 setInteral 指定的反复运行的任务最多每秒执行一次。

setInterval 指定的是“开始执行”之间的间隔，并不考虑每次任务执行本身所消耗的事件。因此实际上，两次执行之间的间隔会小于指定的时间。比如，setInterval 指定每 100ms 执行一次，每次执行需要 5ms，那么第一次执行结束后 95 毫秒，第二次执行就会开始。如果某次执行耗时特别长，比如需要 105 毫秒，那么它结束后，下一次执行就会立即开始。

```js
var i = 1;
var timer = setInterval(function() {
  alert(i++);
}, 2000);
```

上面代码每隔 2000 毫秒，就跳出一个 alert 对话框。如果用户一直不点击“确定”，整个浏览器就处于“堵塞”状态，后面的执行就一直无法触发，将会累积起来。举例来说，第一次跳出 alert 对话框后，用户过了 6000 毫秒才点击“确定”，那么第二次、第三次、第四次执行将累积起来，它们之间不会再有等待间隔。

为了确保两次执行之间有固定的间隔，可以不用 setInterval，而是每次执行结束后，使用 setTimeout 指定下一次执行的具体时间。上面代码用 setTimeout，可以改写如下。

```js
var i = 1;
var timer = setTimeout(function() {
  alert(i++);
  timer = setTimeout(arguments.callee, 2000);
}, 2000);
```

上面代码可以确保两次执行的间隔是 2000 毫秒。

根据这种思路，可以自己部署一个函数，实现间隔时间确定的 setInterval 的效果。

```js
function interval(func, wait){
  var interv = function(){
    func.call(null);
    setTimeout(interv, wait);
  };

  setTimeout(interv, wait);
}

interval(function(){
  console.log(2);
},1000);
```

上面代码部署了一个 interval 函数，用循环调用 setTimeout 模拟了 setInterval。

HTML 5 标准规定，setInterval 的最短间隔时间是 10 毫秒，也就是说，小于 10 毫秒的时间间隔会被调整到 10 毫秒。

## clearTimeout()，clearInterval()

setTimeout 和 setInterval 函数，都返回一个表示计数器编号的整数值，将该整数传入 clearTimeout 和 clearInterval 函数，就可以取消对应的定时器。

```js
var id1 = setTimeout(f,1000);
var id2 = setInterval(f,1000);

clearTimeout(id1);
clearInterval(id2);
```

setTimeout 和 setInterval 返回的整数值是连续的，也就是说，第二个 setTimeout 方法返回的整数值，将比第一个的整数值大 1。利用这一点，可以写一个函数，取消当前所有的 setTimeout。

```js
(function() {
  var gid = setInterval(clearAllTimeouts, 0);

  function clearAllTimeouts() {
    var id = setTimeout(function() {}, 0);
    while (id > 0) {
      if (id !== gid) {
        clearTimeout(id);
      }
      id--;
    }
  }
})();
```

运行上面代码后，实际上再设置任何 setTimeout 都无效了。

下面是一个 clearTimeout 实际应用的例子。有些网站会实时将用户在文本框的输入，通过 Ajax 方法传回服务器，jQuery 的写法如下。

```js
$('textarea').on('keydown', ajaxAction);
```

这样写有一个很大的缺点，就是如果用户连续击键，就会连续触发 keydown 事件，造成大量的 Ajax 通信。这是不必要的，而且很可能会发生性能问题。正确的做法应该是，设置一个门槛值，表示两次 Ajax 通信的最小间隔时间。如果在设定的时间内，发生新的 keydown 事件，则不触发 Ajax 通信，并且重新开始计时。如果过了指定时间，没有发生新的 keydown 事件，将进行 Ajax 通信将数据发送出去。

这种做法叫做 debounce（防抖动）方法，用来返回一个新函数。只有当两次触发之间的时间间隔大于事先设定的值，这个新函数才会运行实际的任务。假定两次 Ajax 通信的间隔不小于 2500 毫秒，上面的代码可以改写成下面这样。

```js
$('textarea').on('keydown', debounce(ajaxAction, 2500))
```

利用 setTimeout 和 clearTimeout，可以实现 debounce 方法。

```js
function debounce(fn, delay){
    var timer = null; // 声明计时器
    return function(){
        var context = this, args = arguments;
        clearTimeout(timer);
        timer = setTimeout(function(){
            fn.apply(context, args);
        }, delay);
    };
}
```

现实中，最好不要设置太多的 setTimeout 和 setInterval，它们耗费 CPU。比较理想的做法是，将要推迟执行的代码都放在一个函数里，然后只对这个函数使用 setTimeout 或 setInterval。

## 运行机制

setTimeout 和 setInterval 的运行机制是，将指定的代码移出本次执行，等到下一轮 Event Loop 时，再检查是否到了指定时间。如果到了，就执行对应的代码；如果不到，就等到再下一轮 Event Loop 时重新判断。这意味着，setTimeout 指定的代码，必须等到本次执行的所有代码都执行完，才会执行。

每一轮 Event Loop 时，都会将“任务队列”中需要执行的任务，一次执行完。setTimeout 和 setInterval 都是把任务添加到“任务队列”的尾部。因此，它们实际上要等到当前脚本的所有同步任务执行完，然后再等到本次 Event Loop 的“任务队列”的所有任务执行完，才会开始执行。由于前面的任务到底需要多少时间执行完，是不确定的，所以没有办法保证，setTimeout 和 setInterval 指定的任务，一定会按照预定时间执行。

```js
setTimeout(someTask,100);
veryLongTask();
```

上面代码的 setTimeout，指定 100 毫秒以后运行一个任务。但是，如果后面立即运行的任务（当前脚本的同步任务））非常耗时，过了 100 毫秒还无法结束，那么被推迟运行的 someTask 就只有等着，等到前面的 veryLongTask 运行结束，才轮到它执行。

这一点对于 setInterval 影响尤其大。

```js
setInterval(function(){
  console.log(2);
},1000);

(function (){ 
  sleeping(3000);
})();
```

上面的第一行语句要求每隔 1000 毫秒，就输出一个 2。但是，第二行语句需要 3000 毫秒才能完成，请问会发生什么结果？

结果就是等到第二行语句运行完成以后，立刻连续输出三个 2，然后开始每隔 1000 毫秒，输出一个 2。也就是说，setIntervel 具有累积效应，如果某个操作特别耗时，超过了 setInterval 的时间间隔，排在后面的操作会被累积起来，然后在很短的时间内连续触发，这可能或造成性能问题（比如集中发出 Ajax 请求）。

为了进一步理解 JavaScript 的单线程模型，请看下面这段伪代码。

```js
function init(){
  { 耗时 5ms 的某个操作 } 
  触发 mouseClickEvent 事件
  { 耗时 5ms 的某个操作 }
  setInterval(timerTask,10);
  { 耗时 5ms 的某个操作 }
}

function handleMouseClick(){
   耗时 8ms 的某个操作 
}

function timerTask(){
   耗时 2ms 的某个操作 
}
```

请问调用 init 函数后，这段代码的运行顺序是怎样的？

*   0-15ms：运行 init 函数。

*   15-23ms：运行 handleMouseClick 函数。请注意，这个函数是在 5ms 时触发的，应该在那个时候就立即运行，但是由于单线程的关系，必须等到 init 函数完成之后再运行。

*   23-25ms：运行 timerTask 函数。这个函数是在 10ms 时触发的，规定每 10ms 运行一次，即在 20ms、30ms、40ms 等时候运行。由于 20ms 时，JavaScript 线程还有任务在运行，因此必须延迟到前面任务完成时再运行。

*   30-32ms：运行 timerTask 函数。

*   40-42ms：运行 timerTask 函数。

## setTimeout(f,0)

### 含义

setTimeout 的作用是将代码推迟到指定时间执行，如果指定时间为 0，即 setTimeout(f,0)，那么会立刻执行吗？

答案是不会。因为上一段说过，必须要等到当前脚本的同步任务和“任务队列”中已有的事件，全部处理完以后，才会执行 setTimeout 指定的任务。也就是说，setTimeout 的真正作用是，在“任务队列”的现有事件的后面再添加一个事件，规定在指定时间执行某段代码。setTimeout 添加的事件，会在下一次 Event Loop 执行。

setTimeout(f,0)将第二个参数设为 0，作用是让 f 在现有的任务（脚本的同步任务和“任务队列”中已有的事件）一结束就立刻执行。也就是说，setTimeout(f,0)的作用是，尽可能早地执行指定的任务。

```js
setTimeout(function (){
  console.log("你好！");
}, 0);
```

上面代码的含义是，尽可能早地显示“你好！”。

setTimeout(f,0)指定的任务，最早也要到下一次 Event Loop 才会执行。请看下面的例子。

```js
setTimeout(function() {
  console.log("Timeout");
}, 0);

function a(x) { 
    console.log("a() 开始运行");
    b(x);
    console.log("a() 结束运行");
}

function b(y) { 
    console.log("b() 开始运行");
    console.log("传入的值为" + y);
    console.log("b() 结束运行");
}

console.log("当前任务开始");
a(42);
console.log("当前任务结束");

// 当前任务开始
// a() 开始运行
// b() 开始运行
// 传入的值为 42
// b() 结束运行
// a() 结束运行
// 当前任务结束
// Timeout
```

上面代码说明，setTimeout(f,0)必须要等到当前脚本的所有同步任务结束后才会执行。

0 毫秒实际上达不到的。根据[HTML 5 标准](http://www.whatwg.org/specs/web-apps/current-work/multipage/timers.html#timers)，setTimeOut 推迟执行的时间，最少是 4 毫秒。如果小于这个值，会被自动增加到 4。这是为了防止多个`setTimeout(f,0)`语句连续执行，造成性能问题。

另一方面，浏览器内部使用 32 位带符号的整数，来储存推迟执行的时间。这意味着 setTimeout 最多只能推迟执行 2147483647 毫秒（24.8 天），超过这个时间会发生溢出，导致回调函数将在当前任务队列结束后立即执行，即等同于 setTimeout(f,0)的效果。

### 应用

setTimeout(f,0)有几个非常重要的用途。它的一大应用是，可以调整事件的发生顺序。比如，网页开发中，某个事件先发生在子元素，然后冒泡到父元素，即子元素的事件回调函数，会早于父元素的事件回调函数触发。如果，我们先让父元素的事件回调函数先发生，就要用到 setTimeout(f, 0)。

```js
var input = document.getElementsByTagName('input[type=button]')[0];

input.onclick = function A() {
  setTimeout(function B() {
    input.value +=' input'; 
  }, 0)
};

document.body.onclick = function C() {
  input.value += ' body'
};
```

上面代码在点击按钮后，先触发回调函数 A，然后触发函数 C。在函数 A 中，setTimeout 将函数 B 推迟到下一轮 Loop 执行，这样就起到了，先触发父元素的回调函数 C 的目的了。

用户自定义的回调函数，通常在浏览器的默认动作之前触发。比如，用户在输入框输入文本，keypress 事件会在浏览器接收文本之前触发。因此，下面的回调函数是达不到目的的。

```js
document.getElementById('input-box').onkeypress = function(event) {
  this.value = this.value.toUpperCase();
}
```

上面代码想在用户输入文本后，立即将字符转为大写。但是实际上，它只能将上一个字符转为大写，因为浏览器此时还没接收到文本，所以`this.value`取不到最新输入的那个字符。只有用 setTimeout 改写，上面的代码才能发挥作用。

```js
document.getElementById('my-ok').onkeypress = function() {
  var self = this;
  setTimeout(function() {
    self.value = self.value.toUpperCase();
  }, 0);
}
```

上面代码将代码放入 setTimeout 之中，就能使得它在浏览器接收到文本之后触发。

由于 setTimeout(f,0)实际上意味着，将任务放到浏览器最早可得的空闲时段执行，所以那些计算量大、耗时长的任务，常常会被放到几个小部分，分别放到 setTimeout(f,0)里面执行。

```js
var div = document.getElementsByTagName('div')[0];

// 写法一
for(var i=0xA00000;i<0xFFFFFF;i++) {
  div.style.backgroundColor = '#'+i.toString(16);
}

// 写法二
var timer;
var i=0x100000;

function func() {
  timer = setTimeout(func, 0);
  div.style.backgroundColor = '#'+i.toString(16);
  if (i++ == 0xFFFFFF) clearInterval(timer);
}

timer = setTimeout(func, 0);
```

上面代码有两种写法，都是改变一个网页元素的背景色。写法一会造成浏览器“堵塞”，而写法二就能就不会，这就是`setTimeout(f,0)`的好处。

另一个使用这种技巧的例子是，代码高亮的处理。如果代码块很大，就会分成一个个小块，写成诸如`setTimeout(highlightNext, 50)`的样子，进行分块处理。

## 参考链接

*   Ilya Kantor, [Understanding timers: setTimeout and setInterval](http://javascript.info/tutorial/settimeout-setinterval)
*   Ilya Kantor, [Events and timing in-depth](http://javascript.info/tutorial/events-and-timing-depth)
*   MDN, [WindowTimers.setTimeout()](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers.setTimeout)
*   Artem Tyurin, [Being evil with setTimeout](http://agentcooper.ghost.io/being-evil-with-settimeout/)