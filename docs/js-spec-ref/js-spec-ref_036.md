# 5.5 Event 对象

事件是一种异步编程的实现方式，本质上是程序各个组成部分之间传递的特定消息。DOM 支持大量的事件，本节介绍 DOM 的事件编程。

*   EventTarget 接口
    *   addEventListener()
    *   removeEventListener()
    *   dispatchEvent()
    *   监听函数
        *   HTML 标签的 on-属性
        *   Element 节点的事件属性
        *   addEventListener 方法
        *   this 对象的指向
    *   事件的传播
        *   传播的三个阶段
        *   事件的代理
    *   Event 对象
        *   bubbles，eventPhase
        *   cancelable，defaultPrevented
        *   currentTarget，target
        *   type，detail，timeStamp，isTrusted
        *   preventDefault()
        *   stopPropagation()
        *   stopImmediatePropagation()
    *   鼠标事件
        *   事件种类
        *   MouseEvent 对象
        *   altKey，ctrlKey，metaKey，shiftKey
        *   button，buttons
        *   clientX，clientY，movementX，movementY，screenX，screenY
        *   relatedTarget
        *   wheel 事件
    *   键盘事件
        *   altKey，ctrlKey，metaKey，shiftKey
        *   key，charCode
    *   进度事件
    *   拖拉事件
        *   事件种类
        *   DataTransfer 对象概述
        *   DataTransfer 对象的属性
        *   DataTransfer 对象的方法
    *   触摸事件
        *   Touch 对象
        *   TouchList 对象
        *   TouchEvent 对象
        *   触摸事件的种类
    *   表单事件
        *   Input 事件，select 事件，change 事件
        *   reset 事件，submit 事件
    *   文档事件
        *   beforeunload 事件，unload 事件，load 事件，error 事件，pageshow 事件，pagehide 事件
        *   DOMContentLoaded 事件，readystatechange 事件
        *   scroll 事件，resize 事件
        *   hashchange 事件，popstate 事件
        *   cut 事件，copy 事件，paste 事件
        *   焦点事件
    *   自定义事件和事件模拟
        *   CustomEvent()
        *   事件的模拟
        *   自定义事件的老式写法
        *   事件模拟的老式写法
    *   参考链接

## EventTarget 接口

DOM 的事件操作（监听和触发），都定义在 EventTarget 接口。Element 节点、document 节点和 window 对象，都部署了这个接口。此外，XMLHttpRequest、AudioNode、AudioContext 等浏览器内置对象，也部署了这个接口。

该接口就是三个方法，addEventListener 和 removeEventListener 用于绑定和移除监听函数，dispatchEvent 用于触发事件。

### addEventListener()

addEventListener 方法用于在当前节点或对象上，定义一个特定事件的监听函数。

```js
target.addEventListener(type, listener[, useCapture]);
```

上面是使用格式，addEventListener 方法接受三个参数。

*   type，事件名称，大小写不敏感。

*   listener，监听函数。指定事件发生时，会调用该监听函数。

*   useCapture，监听函数是否在捕获阶段（capture）触发（参见后文《事件的传播》部分）。该参数是一个布尔值，默认为 false（表示监听函数只在冒泡阶段被触发）。老式浏览器规定该参数必写，较新版本的浏览器允许该参数可选。为了保持兼容，建议总是写上该参数。

下面是一个例子。

```js
function hello(){
  console.log('Hello world');
}

var button = document.getElementById("btn");
button.addEventListener('click', hello, false);
```

上面代码中，addEventListener 方法为 button 节点，绑定 click 事件的监听函数 hello，该函数只在冒泡阶段触发。

可以使用 addEventListener 方法，为当前对象的同一个事件，添加多个监听函数。这些函数按照添加顺序触发，即先添加先触发。如果为同一个事件多次添加同一个监听函数，该函数只会执行一次，多余的添加将自动被去除（不必使用 removeEventListener 方法手动去除）。

```js
function hello(){
  console.log('Hello world');
}

document.addEventListener('click', hello, false);
document.addEventListener('click', hello, false);
```

执行上面代码，点击文档只会输出一行“Hello world”。

如果希望向监听函数传递参数，可以用匿名函数包装一下监听函数。

```js
function print(x) {
  console.log(x);
}

var el = document.getElementById("div1");
el.addEventListener("click", function(){print('Hello')}, false);
```

上面代码通过匿名函数，向监听函数 print 传递了一个参数。

### removeEventListener()

removeEventListener 方法用来移除 addEventListener 方法添加的事件监听函数。

```js
div.addEventListener('click', listener, false);
div.removeEventListener('click', listener, false);
```

removeEventListener 方法的参数，与 addEventListener 方法完全一致。它对第一个参数“事件类型”，也是大小写不敏感。

注意，removeEventListener 方法移除的监听函数，必须与对应的 addEventListener 方法的参数完全一致，而且在同一个元素节点，否则无效。

### dispatchEvent()

dispatchEvent 方法在当前节点上触发指定事件，从而触发监听函数的执行。该方法返回一个布尔值，只要有一个监听函数调用了`Event.preventDefault()`，则返回值为 false，否则为 true。

```js
target.dispatchEvent(event)
```

dispatchEvent 方法的参数是一个 Event 对象的实例。

```js
para.addEventListener('click', hello, false);
var event = new Event('click');
para.dispatchEvent(event);
```

上面代码在当前节点触发了 click 事件。

如果 dispatchEvent 方法的参数为空，或者不是一个有效的事件对象，将报错。

下面代码根据 dispatchEvent 方法的返回值，判断事件是否被取消了。

```js
var canceled = !cb.dispatchEvent(event);
  if (canceled) {
    console.log('事件取消');
  } else {
    console.log('事件未取消');
  }
}
```

## 监听函数

监听函数（listener）是事件发生时，程序所要执行的函数。它是事件驱动编程模式的主要编程方式。

DOM 提供三种方法，可以用来为事件绑定监听函数。

### HTML 标签的 on-属性

HTML 语言允许在元素标签的属性中，直接定义某些事件的监听代码。

```js
<body onload="doSomething()">

<div onclick="console.log('触发事件')">
```

上面代码为 body 节点的 load 事件、div 节点的 click 事件，指定了监听函数。

使用这个方法指定的监听函数，只会在冒泡阶段触发。

注意，使用这种方法时，on-属性的值是“监听代码”，而不是“监听函数”。也就是说，一旦指定事件发生，这些代码是原样传入 JavaScript 引擎执行。因此如果要执行函数，必须在函数名后面加上一对圆括号。

另外，Element 节点的 setAttribue 方法，其实设置的也是这种效果。

```js
el.setAttribute('onclick', 'doSomething()');
```

### Element 节点的事件属性

Element 节点有事件属性，可以定义监听函数。

```js
window.onload = doSomething;

div.onclick = function(event){
  console.log('触发事件');
};
```

使用这个方法指定的监听函数，只会在冒泡阶段触发。

### addEventListener 方法

通过 Element 节点、document 节点、window 对象的 addEventListener 方法，也可以定义事件的监听函数。

```js
window.addEventListener('load', doSomething, false);
```

addEventListener 方法的详细介绍，参见本节 EventTarget 接口的部分。

在上面三种方法中，第一种“HTML 标签的 on-属性”，违反了 HTML 与 JavaScript 代码相分离的原则；第二种“Element 节点的事件属性”的缺点是，同一个事件只能定义一个监听函数，也就是说，如果定义两次 onclick 属性，后一次定义会覆盖前一次。因此，这两种方法都不推荐使用，除非是为了程序的兼容问题，因为所有浏览器都支持这两种方法。

addEventListener 是推荐的指定监听函数的方法。它有如下优点：

*   可以针对同一个事件，添加多个监听函数。

*   能够指定在哪个阶段（捕获阶段还是冒泡阶段）触发回监听函数。

*   除了 DOM 节点，还可以部署在 window、XMLHttpRequest 等对象上面，等于统一了整个 JavaScript 的监听函数接口。

### this 对象的指向

实际编程中，监听函数内部的 this 对象，常常需要指向触发事件的那个 Element 节点。

addEventListener 方法指定的监听函数，内部的 this 对象总是指向触发事件的那个节点。

```js
// HTML 代码为
// <p id="para">Hello</p>

var id = 'doc';
var para = document.getElementById('para');

function hello(){
  console.log(this.id);
}

para.addEventListener('click', hello, false);
```

执行上面代码，点击 p 节点会输出 para。这是因为监听函数被“拷贝”成了节点的一个属性，使用下面的写法，会看得更清楚。

```js
para.onclick = hello;
```

如果将监听函数部署在 Element 节点的 on-属性上面，this 不会指向触发事件的元素节点。

```js
<p id="para" onclick="hello()">Hello</p>
<!-- 或者使用 JavaScript 代码  -->
<script>
  pElement.setAttribute('onclick', 'hello()');
</script>
```

执行上面代码，点击 p 节点会输出 doc。这是因为这里只是调用 hello 函数，而 hello 函数实际是在全局作用域执行，相当于下面的代码。

```js
para.onclick = function(){
  hello();
}
```

一种解决方法是，不引入函数作用域，直接在 on-属性写入所要执行的代码。因为 on-属性是在当前节点上执行的。

```js
<p id="para" onclick="console.log(id)">Hello</p>
<!-- 或者 -->
<p id="para" onclick="console.log(this.id)">Hello</p>
```

上面两行，最后输出的都是 para。

总结一下，以下写法的 this 对象都指向 Element 节点。

```js
// JavaScript 代码
element.onclick = print
element.addEventListener('click', print, false)
element.onclick = function () {console.log(this.id);}

// HTML 代码
<element onclick="console.log(this.id)">
```

以下写法的 this 对象，都指向全局对象。

```js
// JavaScript 代码
element.onclick = function (){ doSomething() };
element.setAttribute('onclick', 'doSomething()');

// HTML 代码
<element onclick="doSomething()">
```

## 事件的传播

### 传播的三个阶段

当一个事件发生以后，它会在不同的 DOM 节点之间传播（propagation）。这种传播分成三个阶段：

*   第一阶段：从 window 对象传导到目标节点，称为“捕获阶段”（capture phase）。

*   第二阶段：在目标节点上触发，称为“目标阶段”（target phase）。

*   第三阶段：从目标节点传导回 window 对象，称为“冒泡阶段”（bubbling phase）。

这种三阶段的传播模型，会使得一个事件在多个节点上触发。比如，假设 div 节点之中嵌套一个 p 节点。

```js
<div>
  <p>Click Me</p>
</div>
```

如果对这两个节点的 click 事件都设定监听函数，则 click 事件会被触发四次。

```js
var phases = {
  1: 'capture',
  2: 'target',
  3: 'bubble'
};

var div = document.querySelector('div');
var p = document.querySelector('p');

div.addEventListener('click', callback, true);
p.addEventListener('click', callback, true);
div.addEventListener('click', callback, false);
p.addEventListener('click', callback, false);

function callback(event) {
  var tag = event.currentTarget.tagName;
  var phase = phases[event.eventPhase];
  console.log("Tag: '" + tag + "'. EventPhase: '" + phase + "'");
}

// 点击以后的结果
// Tag: 'DIV'. EventPhase: 'capture'
// Tag: 'P'. EventPhase: 'target'
// Tag: 'P'. EventPhase: 'target'
// Tag: 'DIV'. EventPhase: 'bubble'
```

上面代码表示，click 事件被触发了四次：p 节点的捕获阶段和冒泡阶段各 1 次，div 节点的捕获阶段和冒泡阶段各 1 次。

1.  捕获阶段：事件从 div 向 p 传播时，触发 div 的 click 事件；
2.  目标阶段：事件从 div 到达 p 时，触发 p 的 click 事件；
3.  目标阶段：事件离开 p 时，触发 p 的 click 事件；
4.  冒泡阶段：事件从 p 传回 div 时，再次触发 div 的 click 事件。

注意，用户点击网页的时候，浏览器总是假定 click 事件的目标节点，就是点击位置的嵌套最深的那个节点（嵌套在 div 节点的 p 节点）。

事件传播的最上层对象是 window，接着依次是 document，html（document.documentElement）和 body（document.dody）。也就是说，如果 body 元素中有一个 div 元素，点击该元素。事件的传播顺序，在捕获阶段依次为 window、document、html、body、div，在冒泡阶段依次为 div、body、html、document、window。

### 事件的代理

由于事件会在冒泡阶段向上传播到父节点，因此可以把子节点的监听函数定义在父节点上，由父节点的监听函数统一处理多个子元素的事件。这种方法叫做事件的代理（delegation）。

```js
var ul = document.querySelector('ul');

ul.addEventListener('click', function(event) {
  if (event.target.tagName.toLowerCase() === 'li') {
    // some code
  }
});
```

上面代码的 click 事件的监听函数定义在 ul 节点，但是实际上，它处理的是子节点 li 的 click 事件。这样做的好处是，只要定义一个监听函数，就能处理多个子节点的事件，而且以后再添加子节点，监听函数依然有效。

如果希望事件到某个节点为止，不再传播，可以使用事件对象的 stopPropagation 方法。

```js
p.addEventListener('click', function(event) {
  event.stopPropagation();
});
```

使用上面的代码以后，click 事件在冒泡阶段到达 p 节点以后，就不再向上（父节点的方向）传播了。

但是，stopPropagation 方法不会阻止 p 节点上的其他 click 事件的监听函数。如果想要不再触发那些监听函数，可以使用 stopImmediatePropagation 方法。

```js
p.addEventListener('click', function(event) {
 event.stopImmediatePropagation();
});

p.addEventListener('click', function(event) {
 // 不会被触发
});
```

## Event 对象

事件发生以后，会生成一个事件对象，作为参数传给监听函数。浏览器原生提供一个 Event 对象，所有的事件都是这个对象的实例，或者说继承了`Event.prototype`对象。

Event 对象本身就是一个构造函数，可以用来生成新的实例。

```js
event = new Event(typeArg, eventInit);
```

Event 构造函数接受两个参数。第一个参数是字符串，表示事件的名称；第二个参数是一个对象，表示事件对象的配置。该参数可以有以下两个属性。

*   bubbles：布尔值，可选，默认为 false，表示事件对象是否冒泡。

*   cancelable：布尔值，可选，默认为 false，表示事件是否可以被取消。

```js
var ev = new Event("look", {"bubbles":true, "cancelable":false});
document.dispatchEvent(ev);
```

上面代码新建一个 look 事件实例，然后使用 dispatchEvent 方法触发该事件。

IE8 及以下版本，事件对象不作为参数传递，而是通过 window 对象的 event 属性读取，并且事件对象的 target 属性叫做 srcElement 属性。所以，以前获取事件信息，往往要写成下面这样。

```js
function myEventHandler(event) {
  var actualEvent = event || window.event;
  var actualTarget = actualEvent.target || actualEvent.srcElement;
  // ...
}
```

上面的代码只是为了说明以前的程序为什么这样写，在新代码中，这样的写法不应该再用了。

以下介绍 Event 实例的属性和方法。

### bubbles，eventPhase

以下属性与事件的阶段有关。

（1）bubbles

bubbles 属性返回一个布尔值，表示当前事件是否会冒泡。该属性为只读属性，只能在新建事件时改变。除非显式声明，Event 构造函数生成的事件，默认是不冒泡的。

```js
function goInput(e) {
  if (!e.bubbles) {
    passItOn(e);
  } else {
    doOutput(e);
  }
}
```

上面代码根据事件是否冒泡，调用不同的函数。

（2）eventPhase

eventPhase 属性返回一个整数值，表示事件目前所处的节点。

```js
var phase = event.eventPhase;
```

*   0，事件目前没有发生。
*   1，事件目前处于捕获阶段，即处于从祖先节点向目标节点的传播过程中。该过程是从 Window 对象到 Document 节点，再到 HTMLHtmlElement 节点，直到目标节点的父节点为止。
*   2，事件到达目标节点，即 target 属性指向的那个节点。
*   3，事件处于冒泡阶段，即处于从目标节点向祖先节点的反向传播过程中。该过程是从父节点一直到 Window 对象。只有 bubbles 属性为 true 时，这个阶段才可能发生。

### cancelable，defaultPrevented

以下属性与事件的默认行为有关。

（1）cancelable

cancelable 属性返回一个布尔值，表示事件是否可以取消。该属性为只读属性，只能在新建事件时改变。除非显式声明，Event 构造函数生成的事件，默认是不可以取消的。

```js
var bool = event.cancelable;
```

如果要取消某个事件，需要在这个事件上面调用 preventDefault 方法，这会阻止浏览器对某种事件部署的默认行为。

（2）defaultPrevented

defaultPrevented 属性返回一个布尔值，表示该事件是否调用过 preventDefault 方法。

```js
if (e.defaultPrevented) {
  // ...
}
```

### currentTarget，target

以下属性与事件的目标节点有关。

（1）currentTarget

currentTarget 属性返回事件当前所在的节点，即正在执行的监听函数所绑定的那个节点。作为比较，target 属性返回事件发生的节点。如果监听函数在捕获阶段和冒泡阶段触发，那么这两个属性返回的值是不一样的。

```js
function hide(e){
  console.log(this === e.currentTarget);  // true
  e.currentTarget.style.visibility = "hidden";
}

para.addEventListener('click', hide, false);
```

上面代码中，点击 para 节点，该节点会不可见。另外，在监听函数中，currentTarget 属性实际上等同于 this 对象。

（2）target

target 属性返回触发事件的那个节点，即事件最初发生的节点。如果监听函数不在该节点触发，那么它与 currentTarget 属性返回的值是不一样的。

```js
function hide(e){
  console.log(this === e.target);  // 有可能不是 true
  e.target.style.visibility = "hidden";
}

// HTML 代码为
// <p id="para">Hello <em>World</em></p>
para.addEventListener('click', hide, false);
```

上面代码中，如果在 para 节点的 em 子节点上面点击，则`e.target`指向 em 子节点，导致 em 子节点（即 World 部分）会不可见，且输出 false。

在 IE6—IE8 之中，该属性的名字不是 target，而是 srcElement，因此经常可以看到下面这样的代码。

```js
function hide(e) {
  var target = e.target || e.srcElement;
  target.style.visibility = 'hidden';
}
```

### type，detail，timeStamp，isTrusted

以下属性与事件对象的其他信息相关。

（1）type

type 属性返回一个字符串，表示事件类型，具体的值同 addEventListener 方法和 removeEventListener 方法的第一个参数一致，大小写不敏感。

```js
var string = event.type;
```

（2）detail

detail 属性返回一个数值，表示事件的某种信息。具体含义与事件类型有关，对于鼠标事件，表示鼠标按键在某个位置按下的次数，比如对于 dblclick 事件，detail 属性的值总是 2。

```js
function giveDetails(e) {
  this.textContent = e.detail;
}

el.onclick = giveDetails;
```

（3）timeStamp

timeStamp 属性返回一个毫秒时间戳，表示事件发生的时间。

```js
var number = event.timeStamp;
```

（4）isTrusted

isTrusted 属性返回一个布尔值，表示该事件是否可以信任。

```js
var bool = event.isTrusted;
```

Firefox 浏览器中，用户触发的事件会返回 true，脚本触发的事件返回 false；IE 浏览器中，除了使用 createEvent 方法生成的事件，所有其他事件都返回 true；Chrome 浏览器不支持该属性。

### preventDefault()

preventDefault 方法取消浏览器对当前事件的默认行为，比如点击链接后，浏览器跳转到指定页面，或者按一下空格键，页面向下滚动一段距离。该方法生效的前提是，事件的 cancelable 属性为 true，如果为 false，则调用该方法没有任何效果。

该方法不会阻止事件的进一步传播（stopPropagation 方法可用于这个目的）。只要在事件的传播过程中（捕获阶段、目标阶段、冒泡阶段皆可），使用了 preventDefault 方法，该事件的默认方法就不会执行。

```js
// HTML 代码为
// <input type="checkbox" id="my-checkbox" />

var cb = document.getElementById('my-checkbox');

cb.addEventListener(
  'click',
  function (e){ e.preventDefault(); },
  false
);
```

上面代码为点击单选框的事件，设置监听函数，取消默认行为。由于浏览器的默认行为是选中单选框，所以这段代码会导致无法选中单选框。

利用这个方法，可以为文本输入框设置校验条件。如果用户的输入不符合条件，就无法将字符输入文本框。

```js
function checkName(e) {
  if (e.charCode < 97 || e.charCode > 122) {
    e.preventDefault();
  }
}
```

上面函数设为文本框的 keypress 监听函数后，将只能输入小写字母，否则输入事件的默认事件（写入文本框）将被取消。

如果监听函数最后返回布尔值 false（即 return false），浏览器也不会触发默认行为，与 preventDefault 方法有等同效果。

### stopPropagation()

stopPropagation 方法阻止事件在 DOM 中继续传播，防止再触发定义在别的节点上的监听函数，但是不包括在当前节点上新定义的事件监听函数。

```js
function stopEvent(e) {
  e.stopPropagation();
}

el.addEventListener('click', stopEvent, false);
```

将上面函数指定为监听函数，会阻止事件进一步冒泡到 el 节点的父节点。

### stopImmediatePropagation()

stopImmediatePropagation 方法阻止同一个事件的其他监听函数被调用。

如果同一个节点对于同一个事件指定了多个监听函数，这些函数会根据添加的顺序依次调用。只要其中有一个监听函数调用了 stopImmediatePropagation 方法，其他的监听函数就不会再执行了。

```js
function l1(e){
  e.stopImmediatePropagation();
}

function l2(e){
  console.log('hello world');
}

el.addEventListener('click', l1, false);
el.addEventListener('click', l2, false);
```

上面代码在 el 节点上，为 click 事件添加了两个监听函数 l1 和 l2。由于 l1 调用了 stopImmediatePropagation 方法，所以 l2 不会被调用。

## 鼠标事件

### 事件种类

鼠标事件指与鼠标相关的事件，主要有以下一些。

（1）click 事件

click 事件当用户在 Element 节点、document 节点、window 对象上，单击鼠标（或者按下回车键）时触发。“鼠标单击”定义为，用户在同一个位置完成一次 mousedown 动作和 mouseup 动作。它们的触发顺序是：mousedown 首先触发，mouseup 接着触发，click 最后触发。

下面是一个设置 click 事件监听函数的例子。

```js
div.addEventListener("click", function( event ) {
  // 显示在该节点，鼠标连续点击的次数
  event.target.innerHTML = "click count: " + event.detail;
}, false);
```

下面的代码是利用 click 事件进行 CSRF 攻击（Cross-site request forgery）的一个例子。

```js
<a href="http://www.harmless.com/" onclick="
  var f = document.createElement('form');
  f.style.display = 'none';
  this.parentNode.appendChild(f);
  f.method = 'POST';
  f.action = 'http://www.example.com/account/destroy';
  f.submit();
  return false;">伪装的链接</a>
```

（2）dblclick 事件

dblclick 事件当用户在 element、document、window 对象上，双击鼠标时触发。该事件会在 mousedown、mouseup、click 之后触发。

（3）mouseup 事件，mousedown 事件

mouseup 事件在释放按下的鼠标键时触发。

mousedown 事件在按下鼠标键时触发。

（4）mousemove 事件

mousemove 事件当鼠标在一个节点内部移动时触发。当鼠标持续移动时，该事件会连续触发。为了避免性能问题，建议对该事件的监听函数做一些限定，比如限定一段时间内只能运行一次代码。

（5）mouseover 事件，mouseenter 事件

mouseover 事件和 mouseenter 事件，都是鼠标进入一个节点时触发。

两者的区别是，mouseover 事件会冒泡，mouseenter 事件不会。子节点的 mouseover 事件会冒泡到父节点，进而触发父节点的 mouseover 事件。mouseenter 事件就没有这种效果，所以进入子节点时，不会触发父节点的监听函数。

下面的例子是 mouseenter 事件与 mouseover 事件的区别。

```js
// HTML 代码为
// <ul id="test">
//   <li>item 1</li>
//   <li>item 2</li>
//   <li>item 3</li>
// </ul>

var test = document.getElementById("test");

// 进入 test 节点以后，该事件只会触发一次
test.addEventListener("mouseenter", function( event ) {
  event.target.style.color = "purple";
  setTimeout(function() {
    event.target.style.color = "";
  }, 500);
}, false);

// 接入 test 节点以后，只要在子 Element 节点上移动，该事件会触发多次
test.addEventListener("mouseover", function( event ) {
  event.target.style.color = "orange";
  setTimeout(function() {
    event.target.style.color = "";
  }, 500);
}, false);
```

上面代码中，由于 mouseover 事件会冒泡，所以子节点的 mouseover 事件会触发父节点的监听函数。

（6）mouseout 事件，mouseleave 事件

mouseout 事件和 mouseleave 事件，都是鼠标离开一个节点时触发。

两者的区别是，mouseout 事件会冒泡，mouseleave 事件不会。子节点的 mouseout 事件会冒泡到父节点，进而触发父节点的 mouseout 事件。mouseleave 事件就没有这种效果，所以离开子节点时，不会触发父节点的监听函数。

（7）contextmenu

contextmenu 事件在一个节点上点击鼠标右键时触发，或者按下“上下文菜单”键时触发。

### MouseEvent 对象

鼠标事件使用 MouseEvent 对象表示，它继承 UIEvent 对象和 Event 对象。浏览器提供一个 MouseEvent 构造函数，用于新建一个 MouseEvent 实例。

```js
event = new MouseEvent(typeArg, mouseEventInit);
```

MouseEvent 构造函数的第一个参数是事件名称（可能的值包括 click、mousedown、mouseup、mouseover、mousemove、mouseout），第二个参数是一个事件初始化对象。该对象可以配置以下属性。

*   screenX，设置鼠标相对于屏幕的水平坐标（但不会移动鼠标），默认为 0，等同于 MouseEvent.screenX 属性。
*   screenY，设置鼠标相对于屏幕的垂直坐标，默认为 0，等同于 MouseEvent.screenY 属性。
*   clientX，设置鼠标相对于窗口的水平坐标，默认为 0，等同于 MouseEvent.clientX 属性。
*   clientY，设置鼠标相对于窗口的垂直坐标，默认为 0，等同于 MouseEvent.clientY 属性。
*   ctrlKey，设置是否按下 ctrl 键，默认为 false，等同于 MouseEvent.ctrlKey 属性。
*   shiftKey，设置是否按下 shift 键，默认为 false，等同于 MouseEvent.shiftKey 属性。
*   altKey，设置是否按下 alt 键，默认为 false，等同于 MouseEvent.altKey 属性。
*   metaKey，设置是否按下 meta 键，默认为 false，等同于 MouseEvent.metaKey 属性。
*   button，设置按下了哪一个鼠标按键，默认为 0。-1 表示没有按键，0 表示按下主键（通常是左键），1 表示按下辅助键（通常是中间的键），2 表示按下次要键（通常是右键）。
*   buttons，设置按下了鼠标哪些键，是一个 3 个比特位的二进制值，默认为 0。1 表示按下主键（通常是左键），2 表示按下次要键（通常是右键），4 表示按下辅助键（通常是中间的键）。
*   relatedTarget，设置一个 Element 节点，在 mouseenter 和 mouseover 事件时，表示鼠标刚刚离开的那个 Element 节点，在 mouseout 和 mouseleave 事件时，表示鼠标正在进入的那个 Element 节点。默认为 null，等同于 MouseEvent.relatedTarget 属性。

以下属性也是可配置的，都继承自 UIEvent 构造函数和 Event 构造函数。

*   bubbles，布尔值，设置事件是否冒泡，默认为 false，等同于 Event.bubbles 属性。
*   cancelable，布尔值，设置事件是否可取消，默认为 false，等同于 Event.cancelable 属性。
*   view，设置事件的视图，一般是 window 或 document.defaultView，等同于 Event.view 属性。
*   detail，设置鼠标点击的次数，等同于 Event.detail 属性。

下面是一个例子。

```js
function simulateClick() {
  var event = new MouseEvent('click', {
    'bubbles': true,
    'cancelable': true
  });
  var cb = document.getElementById('checkbox');
  cb.dispatchEvent(event);
}
```

上面代码生成一个鼠标点击事件，并触发该事件。

以下介绍 MouseEvent 实例的属性。

### altKey，ctrlKey，metaKey，shiftKey

以下属性返回一个布尔值，表示鼠标事件发生时，是否按下某个键。

*   altKey 属性：alt 键
*   ctrlKey 属性：key 键
*   metaKey 属性：Meta 键（Mac 键盘是一个四瓣的小花，Windows 键盘是 Windows 键）
*   shiftKey 属性：Shift 键

```js
// HTML 代码为
// <body onclick="showkey(event);">

function showKey(e){
  console.log("ALT key pressed: " + e.altKey);
  console.log("CTRL key pressed: " + e.ctrlKey);
  console.log("META key pressed: " + e.metaKey);
  console.log("META key pressed: " + e.shiftKey);
}
```

上面代码中，点击网页会输出是否同时按下 Alt 键。

### button，buttons

以下属性返回事件的鼠标键信息。

（1）button

button 属性返回一个数值，表示按下了鼠标哪个键。

*   -1：没有按下键。
*   0：按下主键（通常是左键）。
*   1：按下辅助键（通常是中键或者滚轮键）。
*   2：按下次键（通常是右键）。

```js
// HTML 代码为
// <button onmouseup="whichButton(event);">点击</button>

var whichButton = function (e) {
  switch (e.button) {
    case 0:
      console.log('Left button clicked.');
      break;
    case 1:
      console.log('Middle button clicked.');
      break;
    case 2:
      console.log('Right button clicked.');
      break;
    default:
      console.log('Unexpected code: ' + e.button);
  }
}
```

（2）buttons

buttons 属性返回一个 3 个比特位的值，表示同时按下了哪些键。它用来处理同时按下多个鼠标键的情况。

*   1：二进制为 001，表示按下左键。
*   2：二进制为 010，表示按下右键。
*   4：二进制为 100，表示按下中键或滚轮键。

同时按下多个键的时候，每个按下的键对应的比特位都会有值。比如，同时按下左键和右键，会返回 3（二进制为 011）。

### clientX，clientY，movementX，movementY，screenX，screenY

以下属性与事件的位置相关。

（1）clientX，clientY

clientX 属性返回鼠标位置相对于浏览器窗口左上角的水平坐标，单位为像素，与页面是否横向滚动无关。

clientY 属性返回鼠标位置相对于浏览器窗口左上角的垂直坐标，单位为像素，与页面是否纵向滚动无关。

```js
// HTML 代码为
// <body onmousedown="showCoords(event)">

function showCoords(evt){
  console.log(
    "clientX value: " + evt.clientX + "\n" +
    "clientY value: " + evt.clientY + "\n"
  );
}
```

（2）movementX，movementY

movementX 属性返回一个水平位移，单位为像素，表示当前位置与上一个 mousemove 事件之间的水平距离。在数值上，等于 currentEvent.movementX = currentEvent.screenX - previousEvent.screenX。

movementY 属性返回一个垂直位移，单位为像素，表示当前位置与上一个 mousemove 事件之间的垂直距离。在数值上，等于 currentEvent.movementY = currentEvent.screenY - previousEvent.screenY。

（3）screenX，screenY

screenX 属性返回鼠标位置相对于屏幕左上角的水平坐标，单位为像素。

screenY 属性返回鼠标位置相对于屏幕左上角的垂直坐标，单位为像素。

```js
// HTML 代码为
// <body onmousedown="showCoords(event)">

function showCoords(evt){
  console.log(
    "screenX value: " + evt.screenX + "\n"
    + "screenY value: " + evt.screenY + "\n"
  );
}
```

### relatedTarget

relatedTarget 属性返回事件的次要相关节点。对于那些没有次要相关节点的事件，该属性返回 null。

下表列出不同事件的 target 属性和 relatedTarget 属性含义。

| 事件名称 | target 属性 | relatedTarget 属性 |
| --- | --- | --- |
| focusin | 接受焦点的节点 | 丧失焦点的节点 |
| focusout | 丧失焦点的节点 | 接受焦点的节点 |
| mouseenter | 将要进入的节点 | 将要离开的节点 |
| mouseleave | 将要离开的节点 | 将要进入的节点 |
| mouseout | 将要离开的节点 | 将要进入的节点 |
| mouseover | 将要进入的节点 | 将要离开的节点 |
| dragenter | 将要进入的节点 | 将要离开的节点 |
| dragexit | 将要离开的节点 | 将要进入的节点 |

下面是一个例子。

```js
// HTML 代码为
// <div id="outer" style="height:50px;width:50px;border-width:1px solid black;">
//   <div id="inner" style="height:25px;width:25px;border:1px solid black;"></div>
// </div>

var inner = document.getElementById("inner");

inner.addEventListener("mouseover", function (){
  console.log('进入' + event.target.id + " 离开" + event.relatedTarget.id);
});
inner.addEventListener("mouseenter", function (){
  console.log('进入' + event.target.id + " 离开" + event.relatedTarget.id);
});
inner.addEventListener("mouseout", function (){
  console.log('离开' + event.target.id + " 进入" + event.relatedTarget.id);
});
inner.addEventListener("mouseleave", function (){
  console.log('离开' + event.target.id + " 进入" + event.relatedTarget.id);
});

// 鼠标从 outer 进入 inner，输出
// 进入 inner 离开 outer
// 进入 inner 离开 outer

// 鼠标从 inner 进入 outer，输出
// 离开 inner 进入 outer
// 离开 inner 进入 outer
```

### wheel 事件

wheel 事件是与鼠标滚轮相关的事件，目前只有一个 wheel 事件。用户滚动鼠标的滚轮，就触发这个事件。

该事件除了继承了 MouseEvent、UIEvent、Event 的属性，还有几个自己的属性。

*   deltaX：返回一个数值，表示滚轮的水平滚动量。
*   deltaY：返回一个数值，表示滚轮的垂直滚动量。
*   deltaZ：返回一个数值，表示滚轮的 Z 轴滚动量。
*   deltaMode：返回一个数值，表示滚动的单位，适用于上面三个属性。0 表示像素，1 表示行，2 表示页。

浏览器提供一个 WheelEvent 构造函数，可以用来生成滚轮事件的实例。它接受两个参数，第一个是事件名称，第二个是配置对象。

```js
var syntheticEvent = new WheelEvent("syntheticWheel", {"deltaX": 4, "deltaMode": 0});
```

## 键盘事件

键盘事件用来描述键盘行为，主要有 keydown、keypress、keyup 三个事件。

*   keydown：按下键盘时触发该事件。

*   keypress：只要按下的键并非 Ctrl、Alt、Shift 和 Meta，就接着触发 keypress 事件。

*   keyup：松开键盘时触发该事件。

下面是一个例子，对文本框设置 keypress 监听函数，只允许输入数字。

```js
// HTML 代码为
// <input type="text"
//   name="myInput"
//   onkeypress="return numbersOnly(this, event);"
//   onpaste="return false;"
// />

function numbersOnly(oToCheckField, oKeyEvent) {
  return oKeyEvent.charCode === 0
    || /\d/.test(String.fromCharCode(oKeyEvent.charCode));
}
```

如果用户一直按键不松开，就会连续触发键盘事件，触发的顺序如下。

1.  keydown
2.  keypress
3.  keydown
4.  keypress
5.  （重复以上过程）
6.  keyup

键盘事件使用 KeyboardEvent 对象表示，该对象继承了 UIEvent 和 MouseEvent 对象。浏览器提供 KeyboardEvent 构造函数，用来新建键盘事件的实例。

```js
event = new KeyboardEvent(typeArg, KeyboardEventInit);
```

KeyboardEvent 构造函数的第一个参数是一个字符串，表示事件类型，第二个参数是一个事件配置对象，可配置以下字段。

*   key，对应 KeyboardEvent.key 属性，默认为空字符串。
*   ctrlKey，对应 KeyboardEvent.ctrlKey 属性，默认为 false。
*   shiftKey，对应 KeyboardEvent.shiftKey 属性，默认为 false。
*   altKey，对应 KeyboardEvent.altKey 属性，默认为 false。
*   metaKey，对应 KeyboardEvent.metaKey 属性，默认为 false。

下面就是 KeyboardEvent 实例的属性介绍。

### altKey，ctrlKey，metaKey，shiftKey

以下属性返回一个布尔值，表示是否按下对应的键。

*   altKey：alt 键
*   ctrlKey：ctrl 键
*   metaKey：meta 键（mac 系统是一个四瓣的小花，windows 系统是 windows 键）
*   shiftKey：shift 键

```js
function showChar(e){
  console.log("ALT: " + e.altKey);
  console.log("CTRL: " + e.ctrlKey);
  console.log("Meta: " + e.metaKey);
  console.log("Meta: " + e.shiftKey);
}
```

### key，charCode

key 属性返回一个字符串，表示按下的键名。如果同时按下一个控制键和一个符号键，则返回符号键的键名。比如，按下 Ctrl+a，则返回 a。如果无法识别键名，则返回字符串 Unidentified。

主要功能键的键名（不同的浏览器可能有差异）：Backspace，Tab，Enter，Shift，Control，Alt，CapsLock，CapsLock，Esc，Spacebar，PageUp，PageDown，End，Home，Left，Right，Up，Down，PrintScreen，Insert，Del，Win，F1～F12，NumLock，Scroll 等。

charCode 属性返回一个数值，表示 keypress 事件按键的 Unicode 值，keydown 和 keyup 事件不提供这个属性。注意，该属性已经从标准移除，虽然浏览器还支持，但应该尽量不使用。

## 进度事件

进度事件用来描述一个事件进展的过程，比如 XMLHttpRequest 对象发出的 HTTP 请求的过程、、、、、加载外部资源的过程。下载和上传都会发生进度事件。

进度事件有以下几种。

*   abort 事件：当进度事件被中止时触发。如果发生错误，导致进程中止，不会触发该事件。

*   error 事件：由于错误导致资源无法加载时触发。

*   load 事件：进度成功结束时触发。

*   loadstart 事件：进度开始时触发。

*   loadend 事件：进度停止时触发，发生顺序排在 error 事件\abort 事件\load 事件后面。

*   progress 事件：当操作处于进度之中，由传输的数据块不断触发。

*   timeout 事件：进度超过限时触发。

```js
image.addEventListener('load', function(event) {
  image.classList.add('finished');
});

image.addEventListener('error', function(event) {
  image.style.display = 'none';
});
```

上面代码在图片元素加载完成后，为图片元素的 class 属性添加一个值“finished”。如果加载失败，就把图片元素的样式设置为不显示。

有时候，图片加载会在脚本运行之前就完成，尤其是当脚本放置在网页底部的时候，因此有可能使得 load 和 error 事件的监听函数根本不会被执行。所以，比较可靠的方式，是用 complete 属性先判断一下是否加载完成。

```js
function loaded() {
  // code after image loaded
}

if (image.complete) {
  loaded();
} else {
  image.addEventListener('load', loaded);
}
```

由于 DOM 没有提供像 complete 属性那样的，判断是否发生加载错误的属性，所以 error 事件的监听函数最好放在 img 元素的 HTML 属性中，这样才能保证发生加载错误时百分之百会执行。

```js
<img src="/wrong/url" onerror="this.style.display='none';" />
```

error 事件有一个特殊的性质，就是不会冒泡。这样的设计是正确的，防止引发父元素的 error 事件监听函数。

进度事件使用 ProgressEvent 对象表示。ProgressEvent 实例有以下属性。

*   lengthComputable：返回一个布尔值，表示当前进度是否具有可计算的长度。如果为 false，就表示当前进度无法测量。

*   total：返回一个数值，表示当前进度的总长度。如果是通过 HTTP 下载某个资源，表示内容本身的长度，不含 HTTP 头部的长度。如果 lengthComputable 属性为 false，则 total 属性就无法取得正确的值。

*   loaded：返回一个数值，表示当前进度已经完成的数量。该属性除以 total 属性，就可以得到目前进度的百分比。

下面是一个例子。

```js
var xhr = new XMLHttpRequest();

xhr.addEventListener("progress", updateProgress, false);
xhr.addEventListener("load", transferComplete, false);
xhr.addEventListener("error", transferFailed, false);
xhr.addEventListener("abort", transferCanceled, false);

xhr.open();

function updateProgress (e) {
  if (e.lengthComputable) {
    var percentComplete = e.loaded / e.total;
  } else {
    console.log('不能计算进度');
  }
}

function transferComplete(e) {
  console.log('传输结束');
}

function transferFailed(evt) {
  console.log('传输过程中发生错误');
}

function transferCanceled(evt) {
  console.log('用户取消了传输');
}
```

loadend 事件的监听函数，可以用来取代 abort 事件/load 事件/error 事件的监听函数。

```js
req.addEventListener("loadend", loadEnd, false);

function loadEnd(e) {
  console.log('传输结束，成功失败未知');
}
```

loadend 事件本身不提供关于进度结束的原因，但可以用它来做所有进度结束场景都需要做的一些操作。

另外，上面是下载过程的进度事件，还存在上传过程的进度事件。这时所有监听函数都要放在 XMLHttpRequest.upload 对象上面。

```js
var xhr = new XMLHttpRequest();

xhr.upload.addEventListener("progress", updateProgress, false);
xhr.upload.addEventListener("load", transferComplete, false);
xhr.upload.addEventListener("error", transferFailed, false);
xhr.upload.addEventListener("abort", transferCanceled, false);

xhr.open();
```

浏览器提供一个 ProgressEvent 构造函数，用来生成进度事件的实例。

```js
progressEvent = new ProgressEvent(type, {
  lengthComputable: aBooleanValue,
  loaded: aNumber,
  total: aNumber
});
```

上面代码中，ProgressEvent 构造函数的第一个参数是事件类型（字符串），第二个参数是配置对象，用来指定 lengthComputable 属性（默认值为 false）、loaded 属性（默认值为 0）、total 属性（默认值为 0）。

## 拖拉事件

拖拉指的是，用户在某个对象上按下鼠标键不放，拖动它到另一个位置，然后释放鼠标键，将该对象放在那里。

拖拉的对象有好几种，包括 Element 节点、图片、链接、选中的文字等等。在 HTML 网页中，除了 Element 节点默认不可以拖拉，其他（图片、链接、选中的文字）都是可以直接拖拉的。为了让 Element 节点可拖拉，可以将该节点的 draggable 属性设为 true。

```js
<div draggable="true">
  此区域可拖拉
</div>
```

draggable 属性可用于任何 Element 节点，但是图片（img 元素）和链接（a 元素）不加这个属性，就可以拖拉。对于它们，用到这个属性的时候，往往是将其设为 false，防止拖拉。

注意，一旦某个 Element 节点的 draggable 属性设为 true，就无法再用鼠标选中该节点内部的文字或子节点了。

### 事件种类

当 Element 节点或选中的文本被拖拉时，就会持续触发拖拉事件，包括以下一些事件。

*   drag 事件：拖拉过程中，在被拖拉的节点上持续触发。

*   dragstart 事件：拖拉开始时在被拖拉的节点上触发，该事件的 target 属性是被拖拉的节点。通常应该在这个事件的监听函数中，指定拖拉的数据。

*   dragend 事件：拖拉结束时（释放鼠标键或按下 escape 键）在被拖拉的节点上触发，该事件的 target 属性是被拖拉的节点。它与 dragStart 事件，在同一个节点上触发。不管拖拉是否跨窗口，或者中途被取消，dragend 事件总是会触发的。

*   dragenter 事件：拖拉进入当前节点时，在当前节点上触发，该事件的 target 属性是当前节点。通常应该在这个事件的监听函数中，指定是否允许在当前节点放下（drop）拖拉的数据。如果当前节点没有该事件的监听函数，或者监听函数不执行任何操作，就意味着不允许在当前节点放下数据。在视觉上显示拖拉进入当前节点，也是在这个事件的监听函数中设置。

*   dragover 事件：拖拉到当前节点上方时，在当前节点上持续触发，该事件的 target 属性是当前节点。该事件与 dragenter 事件基本类似，默认会重置当前的拖拉事件的效果（DataTransfer 对象的 dropEffect 属性）为 none，即不允许放下被拖拉的节点，所以如果允许在当前节点 drop 数据，通常会使用 preventDefault 方法，取消重置拖拉效果为 none。

*   dragleave 事件：拖拉离开当前节点范围时，在当前节点上触发，该事件的 target 属性是当前节点。在视觉上显示拖拉离开当前节点，就在这个事件的监听函数中设置。

*   drop 事件：被拖拉的节点或选中的文本，释放到目标节点时，在目标节点上触发。注意，如果当前节点不允许 drop，即使在该节点上方松开鼠标键，也不会触发该事件。如果用户按下 Escape 键，取消这个操作，也不会触发该事件。该事件的监听函数负责取出拖拉数据，并进行相关处理。

关于拖拉事件，有以下几点注意事项。

*   拖拉过程只触发以上这些拖拉事件，尽管鼠标在移动，但是鼠标事件不会触发。

*   将文件从操作系统拖拉进浏览器，不会触发 dragStart 和 dragend 事件。

*   dragenter 和 dragover 事件的监听函数，用来指定可以放下（drop）拖拉的数据。由于网页的大部分区域不适合作为 drop 的目标节点，所以这两个事件的默认设置为当前节点不允许 drop。如果想要在目标节点上 drop 拖拉的数据，首先必须阻止这两个事件的默认行为，或者取消这两个事件。

```js
<div ondragover="return false">
<div ondragover="event.preventDefault()">
```

上面代码中，如果不取消拖拉事件或者阻止默认行为，就不可能在 div 节点上 drop 被拖拉的节点。

拖拉事件用一个 DragEvent 对象表示，该对象继承 MouseEvent 对象，因此也就继承了 UIEvent 和 Event 对象。DragEvent 对象只有一个独有的属性 DataTransfer，其他都是继承的属性。DataTransfer 属性用来读写拖拉事件中传输的数据，详见下文《DataTransfer 对象》的部分。

下面的例子展示，如何动态改变被拖动节点的背景色。

```js
div.addEventListener("dragstart", function(e) {
  this.style.backgroundColor = "red";
}, false);
div.addEventListener("dragend", function(e) {
  this.style.backgroundColor = "green";
}, false);
```

上面代码中，div 节点被拖动时，背景色会变为红色，拖动结束，又变回绿色。

下面是一个例子，显示如何实现将一个节点从当前父节点，拖拉到另一个父节点中。

```js
// HTML 代码为
// <div class="dropzone">
//    <div id="draggable" draggable="true">
//       该节点可拖拉
//    </div>
// </div>
// <div class="dropzone"></div>
// <div class="dropzone"></div>
// <div class="dropzone"></div>

// 被拖拉节点
var dragged;

document.addEventListener("dragstart", function( event ) {
  // 保存被拖拉节点
  dragged = event.target;
  // 被拖拉节点的背景色变透明
  event.target.style.opacity = .5;
}, false);

document.addEventListener("dragend", function( event ) {
  // 被拖拉节点的背景色恢复正常
  event.target.style.opacity = "";
}, false);

document.addEventListener("dragover", function( event ) {
  // 防止拖拉效果被重置，允许被拖拉的节点放入目标节点
  event.preventDefault();
}, false);

document.addEventListener("dragenter", function( event ) {
  // 目标节点的背景色变紫色
  // 由于该事件会冒泡，所以要过滤节点
  if ( event.target.className == "dropzone" ) {
    event.target.style.background = "purple";
  }
}, false);

document.addEventListener("dragleave", function( event ) {
  // 目标节点的背景色恢复原样
  if ( event.target.className == "dropzone" ) {
    event.target.style.background = "";
  }
}, false);

document.addEventListener("drop", function( event ) {
  // 防止事件默认行为（比如某些 Elment 节点上可以打开链接）
  event.preventDefault();
  if ( event.target.className == "dropzone" ) {
    // 恢复目标节点背景色
    event.target.style.background = "";
    // 将被拖拉节点插入目标节点
    dragged.parentNode.removeChild( dragged );
    event.target.appendChild( dragged );
  }
}, false);
```

### DataTransfer 对象概述

所有的拖拉事件都有一个 dataTransfer 属性，用来保存需要传递的数据。这个属性的值是一个 DataTransfer 对象。

拖拉的数据保存两方面的数据：数据的种类（又称格式）和数据的值。数据的种类是一个 MIME 字符串，比如 text/plain 或者 image/jpeg，数据的值是一个字符串。一般来说，如果拖拉一段文本，则数据默认就是那段文本；如果拖拉一个链接，则数据默认就是链接的 URL。

当拖拉事件开始的时候，可以提供数据类型和数据值；在拖拉过程中，通过 dragenter 和 dragover 事件的监听函数，检查数据类型，以确定是否允许放下（drop）被拖拉的对象。比如，在只允许放下链接的区域，检查拖拉的数据类型是否为 text/uri-list。

发生 drop 事件时，监听函数取出拖拉的数据，对其进行处理。

### DataTransfer 对象的属性

DataTransfer 对象有以下属性。

（1）dropEffect

dropEffect 属性设置放下（drop）被拖拉节点时的效果，可能的值包括 copy（复制被拖拉的节点）、move（移动被拖拉的节点）、link（创建指向被拖拉的节点的链接）、none（无法放下被拖拉的节点）。设置除此以外的值，都是无效的。

```js
target.addEventListener('dragover', function(e) {
  e.preventDefault();
  e.stopPropagation();
  e.dataTransfer.dropEffect = 'copy';
});
```

dropEffect 属性一般在 dragenter 和 dragove 事件的监听函数中设置，对于 dragstart、drag、dragleave 这三个事件，该属性不起作用。进入目标节点后，拖拉行为会初始化成用户设定的效果，用户可以通过按下 Shift 键和 Control 键，改变初始设置，在 copy、move、link 三种效果中切换。

鼠标箭头会根据 dropEffect 属性改变形状，提示目前正处于哪一种效果。这意味着，通过鼠标就能判断是否可以在当前节点 drop 被拖拉的节点。

（2）effectAllowed

effectAllowed 属性设置本次拖拉中允许的效果，可能的值包括 copy（复制被拖拉的节点）、move（移动被拖拉的节点）、link（创建指向被拖拉节点的链接）、copyLink（允许 copy 或 link）、copyMove（允许 copy 或 move）、linkMove（允许 link 或 move）、all（允许所有效果）、none（无法放下被拖拉的节点）、uninitialized（默认值，等同于 all）。如果某种效果是不允许的，用户就无法在目标节点中达成这种效果。

dragstart 事件的监听函数，可以设置被拖拉节点允许的效果；dragenter 和 dragover 事件的监听函数，可以设置目标节点允许的效果。

```js
event.dataTransfer.effectAllowed = "copy";
```

dropEffect 属性和 effectAllowed 属性，往往配合使用。

```js
event.dataTransfer.effectAllowed = "copyMove";
event.dataTransfer.dropEffect = "copy";
```

上面代码中，copy 是指定的效果，但是可以通过 Shift 或 Ctrl 键（根据平台而定），将效果切换成 move。

只要 dropEffect 属性和 effectAllowed 属性之中，有一个为 none，就无法在目标节点上完成 drop 操作。

（3）files

files 属性是一个 FileList 对象，包含一组本地文件，可以用来在拖拉操作中传送。如果本次拖拉不涉及文件，则属性为空的 FileList 对象。

下面就是一个接收拖拉文件的例子。

```js
// HTML 代码为
// <div id="output" style="min-height: 200px;border: 1px solid black;">
//   文件拖拉到这里
// </div>

var div = document.getElementById('output');

div.addEventListener("dragenter", function( event ) {
  div.textContent = '';
  event.stopPropagation();
  event.preventDefault();
}, false);

div.addEventListener("dragover", function( event ) {
  event.stopPropagation();
  event.preventDefault();
}, false);

div.addEventListener("drop", function( event ) {
  event.stopPropagation();
  event.preventDefault();
  var files = event.dataTransfer.files;
  for (var i = 0; i < files.length; i++) {
    div.textContent += files[i].name + ' ' + files[i].size + '字节\n';
  }
}, false);
```

上面代码中，通过 files 属性读取拖拉文件的信息。如果想要读取文件内容，就要使用 FileReader 对象。

```js
div.addEventListener('drop', function(e) {
  e.preventDefault();
  e.stopPropagation();

  var fileList = e.dataTransfer.files;
  if (fileList.length > 0) {
    var file = fileList[0];
    var reader = new FileReader();
    reader.onloadend = function(e) {
      if (e.target.readyState == FileReader.DONE) {
        var content = reader.result;
        contentDiv.innerHTML = "File: " + file.name + "\n\n" + content;
      }
    }
    reader.readAsBinaryString(file);
  }
});
```

（4）types

types 属性是一个数组，保存每一次拖拉的数据格式，比如拖拉文件，则格式信息就为 File。

下面是一个例子，通过检查 dataTransfer 属性的类型，决定是否允许在当前节点执行 drop 操作。

```js
function contains(list, value){
  for( var i = 0; i < list.length; ++i ){
    if(list[i] === value) return true;
  }
  return false;
}

function doDragOver(event){
  var isLink = contains( event.dataTransfer.types, "text/uri-list");
  if (isLink) event.preventDefault();
}
```

上面代码中，只有当被拖拉的节点是一个链接时，才允许在当前节点放下。

### DataTransfer 对象的方法

DataTransfer 对象有以下方法。

（1）setData()

setData 方法用来设置事件所带有的指定类型的数据。它接受两个参数，第一个是数据类型，第二个是具体数据。如果指定的类型在现有数据中不存在，则该类型将写入 types 属性；如果已经存在，在该类型的现有数据将被替换。

```js
event.dataTransfer.setData("text/plain", "Text to drag");
```

上面代码为事件加入纯文本格式的数据。

如果拖拉文本框或者拖拉选中的文本，会默认将文本数据添加到 dataTransfer 属性，不用手动指定。

```js
<div draggable="true" ondragstart="
  event.dataTransfer.setData('text/plain', 'bbb')">
  aaa
</div>
```

上面代码中，拖拉数据实际上是 bbb，而不是 aaa。

下面是添加其他类型的数据。由于 text/plain 是最普遍支持的格式，为了保证兼容性，建议最后总是将数据保存一份纯文本的格式。

```js
var dt = event.dataTransfer;

// 添加链接
dt.setData("text/uri-list", "http://www.example.com");
dt.setData("text/plain", "http://www.example.com");
// 添加 HTML 代码
dt.setData("text/html", "Hello there, <strong>stranger</strong>");
dt.setData("text/plain", "Hello there, <strong>stranger</strong>");
// 添加图像的 URL
dt.setData("text/uri-list", imageurl);
dt.setData("text/plain", imageurl);
```

可以一次提供多种格式的数据。

```js
var dt = event.dataTransfer;
dt.setData("application/x-bookmark", bookmarkString);
dt.setData("text/uri-list", "http://www.example.com");
dt.setData("text/plain", "http://www.example.com");
```

上面代码中，通过在同一个事件上面，存放三种类型的数据，使得拖拉事件可以在不同的对象上面，drop 不同的值。注意，第一种格式是一个自定义格式，浏览器默认无法读取，这意味着，只有某个部署了特定代码的节点，才可能 drop（读取到）这个数据。

（2）getData()

getData 方法接受一个字符串（表示数据类型）作为参数，返回事件所带的指定类型的数据（通常是用 setData 方法添加的数据）。如果指定类型的数据不存在，则返回空字符串。通常只有 drop 事件触发后，才能取出数据。如果取出另一个域名存放的数据，将会报错。

下面是一个 drop 事件的监听函数，用来取出指定类型的数据。

```js
function onDrop(event){
  var data = event.dataTransfer.getData("text/plain");
  event.target.textContent = data;
  event.preventDefault();
}
```

上面代码取出拖拉事件的文本数据，将其替换成当前节点的文本内容。注意，这时还必须取消浏览器的默认行为，因为假如用户拖拉的是一个链接，浏览器默认会在当前窗口打开这个链接。

getData 方法返回的是一个字符串，如果其中包含多项数据，就必须手动解析。

```js
function doDrop(event){
  var lines = event.dataTransfer.getData("text/uri-list").split("\n");
  for (let line of lines) {
    let link = document.createElement("a");
    link.href = line;
    link.textContent = line;
    event.target.appendChild(link);
  }
  event.preventDefault();
}
```

上面代码中，getData 方法返回的是一组链接，就必须自行解析。

类型值指定为 URL，可以取出第一个有效链接。

```js
var link = event.dataTransfer.getData("URL");
```

下面是一次性取出多种类型的数据。

```js
function doDrop(event){
  var types = event.dataTransfer.types;
  var supportedTypes = ["text/uri-list", "text/plain"];
  types = supportedTypes.filter(function (value) types.includes(value));
  if (types.length)
    var data = event.dataTransfer.getData(types[0]);
  event.preventDefault();
}
```

（3）clearData()

clearData 方法接受一个字符串（表示数据类型）作为参数，删除事件所带的指定类型的数据。如果没有指定类型，则删除所有数据。如果指定类型不存在，则原数据不受影响。

```js
event.dataTransfer.clearData("text/uri-list");
```

上面代码清除事件所带的 URL 数据。

（4）setDragImage()

拖动过程中（dragstart 事件触发后），浏览器会显示一张图片跟随鼠标一起移动，表示被拖动的节点。这张图片是自动创造的，通常显示为被拖动节点的外观，不需要自己动手设置。setDragImage 方法可以用来自定义这张图片，它接受三个参数，第一个是 img 图片元素或者 canvas 元素，如果省略或为 null 则使用被拖动的节点的外观，第二个和第三个参数为鼠标相对于该图片左上角的横坐标和右坐标。

下面是一个例子。

```js
// HTML 代码为
// <div id="drag-with-image" class="dragdemo" draggable="true">
     drag me
// </div>

var div = document.getElementById("drag-with-image");
div.addEventListener("dragstart", function(e) {
  var img = document.createElement("img");
  img.src = "http://path/to/img";
  e.dataTransfer.setDragImage(img, 0, 0);
}, false);
```

## 触摸事件

触摸 API 由三个对象组成。

*   Touch
*   TouchList
*   TouchEvent

Touch 对象表示触摸点（一根手指或者一根触摸笔），用来描述触摸动作，包括位置、大小、形状、压力、目标元素等属性。有时，触摸动作由多个触摸点（多根手指或者多根触摸笔）组成，多个触摸点的集合由 TouchList 对象表示。TouchEvent 对象代表由触摸引发的事件，只有触摸屏才会引发这一类事件。

很多时候，触摸事件和鼠标事件同时触发，即使这个时候并没有用到鼠标。这是为了让那些只定义鼠标事件、没有定义触摸事件的代码，在触摸屏的情况下仍然能用。如果想避免这种情况，可以用 preventDefault 方法阻止发出鼠标事件。

### Touch 对象

Touch 对象代表一个触摸点。触摸点可能是一根手指，也可能是一根触摸笔。它有以下属性。

（1）identifier

identifier 属性表示 Touch 实例的独一无二的识别符。它在整个触摸过程中保持不变。

```js
var id = touchItem.identifier;
```

TouchList 对象的 identifiedTouch 方法，可以根据这个属性，从一个集合里面取出对应的 Touch 对象。

（2）screenX，screenY，clientX，clientY，pageX，pageY

screenX 属性和 screenY 属性，分别表示触摸点相对于屏幕左上角的横坐标和纵坐标，与页面是否滚动无关。

clientX 属性和 clientY 属性，分别表示触摸点相对于浏览器视口左上角的横坐标和纵坐标，与页面是否滚动无关。

pageX 属性和 pageY 属性，分别表示触摸点相对于当前页面左上角的横坐标和纵坐标，包含了页面滚动带来的位移。

（3）radiusX，radiusY，rotationAngle

radiusX 属性和 radiusY 属性，分别返回触摸点周围受到影响的椭圆范围的 X 轴和 Y 轴，单位为像素。

rotationAngle 属性表示触摸区域的椭圆的旋转角度，单位为度数，在 0 到 90 度之间。

上面这三个属性共同定义了用户与屏幕接触的区域，对于描述手指这一类非精确的触摸，很有帮助。指尖接触屏幕，触摸范围会形成一个椭圆，这三个属性就用来描述这个椭圆区域。

（4）force

force 属性返回一个 0 到 1 之间的数值，表示触摸压力。0 代表没有压力，1 代表硬件所能识别的最大压力。

（5）target

target 属性返回一个 Element 节点，代表触摸发生的那个节点。

### TouchList 对象

TouchList 对象是一个类似数组的对象，成员是与某个触摸事件相关的所有触摸点。比如，用户用三根手指触摸，产生的 TouchList 对象就有三个成员，每根手指对应一个 Touch 对象。

TouchList 实例的 length 属性，返回 TouchList 对象的成员数量。

TouchList 实例的 identifiedTouch 方法和 item 方法，分别使用 id 属性和索引值（从 0 开始）作为参数，取出指定的 Touch 对象。

### TouchEvent 对象

TouchEvent 对象继承 Event 对象和 UIEvent 对象，表示触摸引发的事件。除了被继承的属性以外，它还有一些自己的属性。

（1）键盘相关属性

以下属性都为只读属性，返回一个布尔值，表示触摸的同时，是否按下某个键。

*   altKey 是否按下 alt 键
*   ctrlKey 是否按下 ctrl 键
*   metaKey 是否按下 meta 键
*   shiftKey 是否按下 shift 键

（2）changedTouches

changedTouches 属性返回一个 TouchList 对象，包含了由当前触摸事件引发的所有 Touch 对象（即相关的触摸点）。

对于 touchstart 事件，它代表被激活的触摸点；对于 touchmove 事件，代表发生变化的触摸点；对于 touchend 事件，代表消失的触摸点（即不再被触碰的点）。

```js
var touches = touchEvent.changedTouches;
```

（3）targetTouches

targetTouches 属性返回一个 TouchList 对象，包含了触摸的目标 Element 节点内部，所有仍然处于活动状态的触摸点。

```js
var touches = touchEvent.targetTouches;
```

（4）touches

touches 属性返回一个 TouchList 对象，包含了所有仍然处于活动状态的触摸点。

```js
var touches = touchEvent.touches;
```

### 触摸事件的种类

触摸引发的事件，有以下几类。可以通过 TouchEvent.type 属性，查看到底发生的是哪一种事件。

*   touchstart：用户接触触摸屏时触发，它的 target 属性返回发生触摸的 Element 节点。

*   touchend：用户不再接触触摸屏时（或者移出屏幕边缘时）触发，它的 target 属性与 touchstart 事件的 target 属性是一致的，它的 changedTouches 属性返回一个 TouchList 对象，包含所有不再触摸的触摸点（Touch 对象）。

*   touchmove：用户移动触摸点时触发，它的 target 属性与 touchstart 事件的 target 属性一致。如果触摸的半径、角度、力度发生变化，也会触发该事件。

*   touchcancel：触摸点取消时触发，比如在触摸区域跳出一个情态窗口（modal window）、触摸点离开了文档区域（进入浏览器菜单栏区域）、用户放置更多的触摸点（自动取消早先的触摸点）。

下面是一个例子。

```js
var el = document.getElementsByTagName("canvas")[0];
el.addEventListener("touchstart", handleStart, false);
el.addEventListener("touchmove", handleMove, false);

function handleStart(evt) {
  // 阻止浏览器继续处理触摸事件，
  // 也阻止发出鼠标事件
  evt.preventDefault();
  var touches = evt.changedTouches;

  for (var i = 0; i < touches.length; i++) {
    console.log(touches[i].pageX, touches[i].pageY);
  }
}

function handleMove(evt) {
  evt.preventDefault();
  var touches = evt.changedTouches;

  for (var i = 0; i < touches.length; i++) {
    var id = touches[i].identifier;
    var touch = touches.identifiedTouch(id);
    console.log(touch.pageX, touch.pageY);
  }
}
```

## 表单事件

### Input 事件，select 事件，change 事件

以下事件与表单成员的值变化有关。

（1）input 事件

input 事件当、的值发生变化时触发。此外，打开 contenteditable 属性的元素，只要值发生变化，也会触发 input 事件。

input 事件的一个特点，就是会连续触发，比如用户每次按下一次按键，就会触发一次 input 事件。

（2）select 事件

select 事件当在、中选中文本时触发。

```js
// HTML 代码为
// <input id="test" type="text" value="Select me!" />

var elem = document.getElementById('test');
elem.addEventListener('select', function() {
  console.log('Selection changed!');
}, false);
```

（3）Change 事件

Change 事件当、、的值发生变化时触发。它与 input 事件的最大不同，就是不会连续触发，只有当全部修改完成时才会触发，而且 input 事件必然会引发 change 事件。具体来说，分成以下几种情况。

*   激活单选框（radio）或复选框（checkbox）时触发。
*   用户提交时触发。比如，从下列列表（select）完成选择，在日期或文件输入框完成选择。
*   当文本框或 textarea 元素的值发生改变，并且丧失焦点时触发。

下面是一个例子。

```js
// HTML 代码为
// <select size="1" onchange="changeEventHandler(event);">
//   <option>chocolate</option>
//   <option>strawberry</option>
//   <option>vanilla</option>
// </select>

function changeEventHandler(event) {
  console.log('You like ' + event.target.value + ' ice cream.');
}
```

### reset 事件，submit 事件

以下事件发生在表单对象上，而不是发生在表单的成员上。

（1）reset 事件

reset 事件当表单重置（所有表单成员变回默认值）时触发。

（2）submit 事件

submit 事件当表单数据向服务器提交时触发。注意，submit 事件的发生对象是 form 元素，而不是 button 元素（即使它的类型是 submit），因为提交的是表单，而不是按钮。

## 文档事件

### beforeunload 事件，unload 事件，load 事件，error 事件，pageshow 事件，pagehide 事件

以下事件与网页的加载与卸载相关。

（1）beforeunload 事件

beforeunload 事件当窗口将要关闭，或者 document 和网页资源将要卸载时触发。它可以用来防止用户不当心关闭网页。

该事件的默认动作就是关闭当前窗口或文档。如果在监听函数中，调用了`event.preventDefault()`，或者对事件对象的 returnValue 属性赋予一个非空的值，就会自动跳出一个确认框，让用户确认是否关闭网页。如果用户点击“取消”按钮，网页就不会关闭。监听函数所返回的字符串，会显示在确认对话框之中。

```js
window.onbeforeunload = function() {
  if (textarea.value != textarea.defaultValue) {
    return '你确认要离开吗？';
  }
};
```

上面代码表示，当用户关闭网页，会跳出一个确认对话框，上面显示“你确认要离开吗？”。

下面的两种写法，具有同样效果。

```js
window.addEventListener('beforeunload', function( event ) {
  event.returnValue = '你确认要离开吗？';
});

// 等同于
window.addEventListener('beforeunload', function( event ) {
  event.preventDefault();
});
```

上面代码中，事件对象的 returnValue 属性的值，将会成为确认框的提示文字。

只要定义了 beforeunload 事件的监听函数，网页不会被浏览器缓存。

（2）unload 事件

unload 事件在窗口关闭或者 document 对象将要卸载时触发，发生在 window、body、frameset 等对象上面。它的触发顺序排在 beforeunload、pagehide 事件后面。unload 事件只在页面没有被浏览器缓存时才会触发，换言之，如果通过按下“前进/后退”导致页面卸载，并不会触发 unload 事件。

当 unload 事件发生时，document 对象处于一个特殊状态。所有资源依然存在，但是对用户来说都不可见，UI 互动（window.open、alert、confirm 方法等）全部无效。这时即使抛出错误，也不能停止文档的卸载。

```js
window.addEventListener('unload', function(event) {
  console.log('文档将要卸载');
});
```

如果在 window 对象上定义了该事件，网页就不会被浏览器缓存。

（3）load 事件，error 事件

load 事件在页面加载成功时触发，error 事件在页面加载失败时触发。注意，页面从浏览器缓存加载，并不会触发 load 事件。

这两个事件实际上属于进度事件，不仅发生在 document 对象，还发生在各种外部资源上面。浏览网页就是一个加载各种资源的过程，图像（image）、样式表（style sheet）、脚本（script）、视频（video）、音频（audio）、Ajax 请求（XMLHttpRequest）等等。这些资源和 document 对象、window 对象、XMLHttpRequestUpload 对象，都会触发 load 事件和 error 事件。

（4）pageshow 事件，pagehide 事件

默认情况下，浏览器会在当前会话（session）缓存页面，当用户点击“前进/后退”按钮时，浏览器就会从缓存中加载页面。

pageshow 事件在页面加载时触发，包括第一次加载和从缓存加载两种情况。如果要指定页面每次加载（不管是不是从浏览器缓存）时都运行的代码，可以放在这个事件的监听函数。

第一次加载时，它的触发顺序排在 load 事件后面。从缓存加载时，load 事件不会触发，因为网页在缓存中的样子通常是 load 事件的监听函数运行后的样子，所以不必重复执行。同理，如果是从缓存中加载页面，网页内初始化的 JavaScript 脚本（比如 DOMContentLoaded 事件的监听函数）也不会执行。

```js
window.addEventListener('pageshow', function(event) {
  console.log('pageshow: ', event);
});
```

pageshow 事件有一个 persisted 属性，返回一个布尔值。页面第一次加载时，这个属性是 false；当页面从缓存加载时，这个属性是 true。

```js
window.addEventListener('pageshow', function(event){
  if (event.persisted) {
    // ...
  }
});
```

pagehide 事件与 pageshow 事件类似，当用户通过“前进/后退”按钮，离开当前页面时触发。它与 unload 事件的区别在于，如果在 window 对象上定义 unload 事件的监听函数之后，页面不会保存在缓存中，而使用 pagehide 事件，页面会保存在缓存中。

pagehide 事件的 event 对象有一个 persisted 属性，将这个属性设为 true，就表示页面要保存在缓存中；设为 false，表示网页不保存在缓存中，这时如果设置了 unload 事件的监听函数，该函数将在 pagehide 事件后立即运行。

如果页面包含 frame 或 iframe 元素，则 frame 页面的 pageshow 事件和 pagehide 事件，都会在主页面之前触发。

### DOMContentLoaded 事件，readystatechange 事件

以下事件与文档状态相关。

（1）DOMContentLoaded 事件

当 HTML 文档下载并解析完成以后，就会在 document 对象上触发 DOMContentLoaded 事件。这时，仅仅完成了 HTML 文档的解析（整张页面的 DOM 生成），所有外部资源（样式表、脚本、iframe 等等）可能还没有下载结束。也就是说，这个事件比 load 事件，发生时间早得多。

```js
document.addEventListener("DOMContentLoaded", function(event) {
  console.log("DOM 生成");
});
```

注意，网页的 JavaScript 脚本是同步执行的，所以定义 DOMContentLoaded 事件的监听函数，应该放在所有脚本的最前面。否则脚本一旦发生堵塞，将推迟触发 DOMContentLoaded 事件。

（2）readystatechange 事件

readystatechange 事件发生在 Document 对象和 XMLHttpRequest 对象，当它们的 readyState 属性发生变化时触发。

```js
document.onreadystatechange = function () {
  if (document.readyState == "interactive") {
    // ...
  }
}
```

IE8 不支持 DOMContentLoaded 事件，但是支持这个事件。因此，可以使用 readystatechange 事件，在低版本的 IE 中代替 DOMContentLoaded 事件。

### scroll 事件，resize 事件

以下事件与窗口行为有关。

（1）scroll 事件

scroll 事件在文档或文档元素滚动时触发。

由于该事件会连续地大量触发，所以它的监听函数之中不应该有非常耗费计算的操作。推荐的做法是使用 requestAnimationFrame 或 setTimeout 控制该事件的触发频率，然后可以结合 customEvent 抛出一个新事件。

```js
(function() {
  var throttle = function(type, name, obj) {
    var obj = obj || window;
    var running = false;
    var func = function() {
      if (running) { return; }
      running = true;
      requestAnimationFrame(function() {
        obj.dispatchEvent(new CustomEvent(name));
        running = false;
      });
    };
    obj.addEventListener(type, func);
  };

  // 将 scroll 事件重定义为 optimizedScroll 事件
  throttle("scroll", "optimizedScroll");
})();

window.addEventListener("optimizedScroll", function() {
  console.log("Resource conscious scroll callback!");
});
```

上面代码中，throttle 函数用于控制事件触发频率，requestAnimationFrame 方法保证每次页面重绘（每秒 60 次），只会触发一次 scroll 事件的监听函数。改用 setTimeout 方法，可以放置更大的时间间隔。

```js
(function() {
  window.addEventListener("scroll", scrollThrottler, false);

  var scrollTimeout;
  function scrollThrottler() {
    if ( !scrollTimeout ) {
      scrollTimeout = setTimeout(function() {
        scrollTimeout = null;
        actualScrollHandler();
      }, 66);
    }
  }

  function actualScrollHandler() {
    // ...
  }
}());
```

上面代码中，setTimeout 指定 scroll 事件的监听函数，每 66 毫秒触发一次（每秒 15 次）。

（2）resize 事件

resize 事件在改变浏览器窗口大小时触发，发生在 window、body、frameset 对象上面。

```js
var resizeMethod = function(){
  if (document.body.clientWidth < 768) {
    console.log('移动设备');
  }
};

window.addEventListener("resize", resizeMethod, true);
```

该事件也会连续地大量触发，所以最好像上面的 scroll 事件一样，通过 throttle 函数控制事件触发频率。

### hashchange 事件，popstate 事件

以下事件与文档的 URL 变化相关。

（1）hashchange 事件

hashchange 事件在 URL 的 hash 部分（即#号后面的部分，包括#号）发生变化时触发。如果老式浏览器不支持该属性，可以通过定期检查 location.hash 属性，模拟该事件，下面就是代码。

```js
(function(window) {
  if ( "onhashchange" in window.document.body ) { return; }

  var location = window.location;
  var oldURL = location.href;
  var oldHash = location.hash;

  // 每隔 100 毫秒检查一下 URL 的 hash
  setInterval(function() {
    var newURL = location.href;
    var newHash = location.hash;

    if ( newHash != oldHash && typeof window.onhashchange === "function" ) {
      window.onhashchange({
        type: "hashchange",
        oldURL: oldURL,
        newURL: newURL
      });

      oldURL = newURL;
      oldHash = newHash;
    }
  }, 100);

})(window);
```

hashchange 事件对象除了继承 Event 对象，还有 oldURL 属性和 newURL 属性，分别表示变化前后的 URL。

（2）popstate 事件

popstate 事件在浏览器的 history 对象的当前记录发生显式切换时触发。注意，调用 history.pushState()或 history.replaceState()，并不会触发 popstate 事件。该事件只在用户在 history 记录之间显式切换时触发，比如鼠标点击“后退/前进”按钮，或者在脚本中调用 history.back()、history.forward()、history.go()时触发。

该事件对象有一个 state 属性，保存 history.pushState 方法和 history.replaceState 方法为当前记录添加的 state 对象。

```js
window.onpopstate = function(event) {
  console.log("state: " + event.state);
};
history.pushState({page: 1}, "title 1", "?page=1");
history.pushState({page: 2}, "title 2", "?page=2");
history.replaceState({page: 3}, "title 3", "?page=3");
history.back(); // state: {"page":1}
history.back(); // state: null
history.go(2);  // state: {"page":3}
```

上面代码中，pushState 方法向 history 添加了两条记录，然后 replaceState 方法替换掉当前记录。因此，连续两次 back 方法，会让当前条目退回到原始网址，它没有附带 state 对象，所以事件的 state 属性为 null，然后前进两条记录，又回到 replaceState 方法添加的记录。

浏览器对于页面首次加载，是否触发 popstate 事件，处理不一样，Firefox 不触发该事件。

### cut 事件，copy 事件，paste 事件

以下三个事件属于文本操作触发的事件。

*   cut 事件：在将选中的内容从文档中移除，加入剪贴板后触发。

*   copy 事件：在选中的内容加入剪贴板后触发。

*   paste 事件：在剪贴板内容被粘贴到文档后触发。

这三个事件都有一个 clipboardData 只读属性。该属性存放剪贴的数据，是一个 DataTransfer 对象，具体的 API 接口和操作方法，请参见《触摸事件》的 DataTransfer 对象章节。

### 焦点事件

焦点事件发生在 Element 节点和 document 对象上面，与获得或失去焦点相关。它主要包括以下四个事件。

*   focus 事件：Element 节点获得焦点后触发，该事件不会冒泡。

*   blur 事件：Element 节点失去焦点后触发，该事件不会冒泡。

*   focusin 事件：Element 节点将要获得焦点时触发，发生在 focus 事件之前。该事件会冒泡。Firefox 不支持该事件。

*   focusout 事件：Element 节点将要失去焦点时触发，发生在 blur 事件之前。该事件会冒泡。Firefox 不支持该事件。

这四个事件的事件对象，带有 target 属性（返回事件的目标节点）和 relatedTarget 属性（返回一个 Element 节点）。对于 focusin 事件，relatedTarget 属性表示失去焦点的节点；对于 focusout 事件，表示将要接受焦点的节点；对于 focus 和 blur 事件，该属性返回 null。

由于 focus 和 blur 事件不会冒泡，只能在捕获阶段触发，所以 addEventListener 方法的第三个参数需要设为 true。

```js
form.addEventListener("focus", function( event ) {
  event.target.style.background = "pink";
}, true);
form.addEventListener("blur", function( event ) {
  event.target.style.background = "";
}, true);
```

上面代码设置表单的文本输入框，在接受焦点时设置背景色，在失去焦点时去除背景色。

浏览器提供一个 FocusEvent 构造函数，可以用它生成焦点事件的实例。

```js
var focusEvent = new FocusEvent(typeArg, focusEventInit);
```

上面代码中，FocusEvent 构造函数的第一个参数为事件类型，第二个参数是可选的配置对象，用来配置 FocusEvent 对象。

## 自定义事件和事件模拟

除了浏览器预定义的那些事件，用户还可以自定义事件，然后手动触发。

```js
// 新建事件实例
var event = new Event('build');

// 添加监听函数
elem.addEventListener('build', function (e) { ... }, false);

// 触发事件
elem.dispatchEvent(event);
```

上面代码触发了自定义事件，该事件会层层向上冒泡。在冒泡过程中，如果有一个元素定义了该事件的监听函数，该监听函数就会触发。

由于 IE 不支持这个 API，如果在 IE 中自定义事件，需要使用后文的“老式方法”。

### CustomEvent()

Event 构造函数只能指定事件名，不能在事件上绑定数据。如果需要在触发事件的同时，传入指定的数据，需要使用 CustomEvent 构造函数生成自定义的事件对象。

```js
var event = new CustomEvent('build', { 'detail': 'hello' });
function eventHandler(e) {
  console.log(e.detail);
}
```

上面代码中，CustomEvent 构造函数的第一个参数是事件名称，第二个参数是一个对象，该对象的 detail 属性会绑定在事件对象之上。

下面是另一个例子。

```js
var myEvent = new CustomEvent("myevent", {
  detail: {
    foo: "bar"
  },
  bubbles: true,
  cancelable: false
});

el.addEventListener('myevent', function(event) {
  console.log('Hello ' + event.detail.foo);
});

el.dispatchEvent(myEvent);
```

IE 不支持这个方法，可以用下面的垫片函数模拟。

```js
(function () {
  function CustomEvent ( event, params ) {
    params = params || { bubbles: false, cancelable: false, detail: undefined };
    var evt = document.createEvent( 'CustomEvent' );
    evt.initCustomEvent( event, params.bubbles, params.cancelable, params.detail );
    return evt;
   }

  CustomEvent.prototype = window.Event.prototype;

  window.CustomEvent = CustomEvent;
})();
```

### 事件的模拟

有时，需要在脚本中模拟触发某种类型的事件，这时就必须使用这种事件的构造函数。

下面是一个通过 MouseEvent 构造函数，模拟触发 click 鼠标事件的例子。

```js
function simulateClick() {
  var event = new MouseEvent('click', {
    'bubbles': true,
    'cancelable': true
  });
  var cb = document.getElementById('checkbox');
  cb.dispatchEvent(event);
}
```

### 自定义事件的老式写法

老式浏览器不一定支持各种类型事件的构造函数。因此，有时为了兼容，会用到一些非标准的方法。这些方法未来会被逐步淘汰，但是目前浏览器还广泛支持。除非是为了兼容老式浏览器，尽量不要使用。

（1）document.createEvent()

document.createEvent 方法用来新建指定类型的事件。它所生成的 Event 实例，可以传入 dispatchEvent 方法。

```js
// 新建 Event 实例
var event = document.createEvent('Event');

// 事件的初始化
event.initEvent('build', true, true);

// 加上监听函数
document.addEventListener('build', doSomething, false);

// 触发事件
document.dispatchEvent(event);
```

createEvent 方法接受一个字符串作为参数，可能的值参见下表“数据类型”一栏。使用了某一种“事件类型”，就必须使用对应的事件初始化方法。

| 事件类型 | 事件初始化方法 |
| --- | --- |
| UIEvents | event.initUIEvent |
| MouseEvents | event.initMouseEvent |
| MutationEvents | event.initMutationEvent |
| HTMLEvents | event.initEvent |
| Event | event.initEvent |
| CustomEvent | event.initCustomEvent |
| KeyboardEvent | event.initKeyEvent |

（2）event.initEvent()

事件对象的 initEvent 方法，用来初始化事件对象，还能向事件对象添加属性。该方法的参数必须是一个使用`Document.createEvent()`生成的 Event 实例，而且必须在 dispatchEvent 方法之前调用。

```js
var event = document.createEvent('Event');
event.initEvent('my-custom-event', true, true, {foo:'bar'});
someElement.dispatchEvent(event);
```

initEvent 方法可以接受四个参数。

*   type：事件名称，格式为字符串。
*   bubbles：事件是否应该冒泡，格式为布尔值。可以使用 event.bubbles 属性读取它的值。
*   cancelable：事件是否能被取消，格式为布尔值。可以使用 event.cancelable 属性读取它的值。
*   option：为事件对象指定额外的属性。

### 事件模拟的老式写法

事件模拟的非标准做法是，对 document.createEvent 方法生成的事件对象，使用对应的事件初始化方法进行初始化。比如，click 事件对象属于 MouseEvent 对象，也属于 UIEvent 对象，因此要用 initMouseEvent 方法或 initUIEvent 方法进行初始化。

（1）event.initMouseEvent()

initMouseEvent 方法用来初始化 Document.createEvent 方法新建的鼠标事件。该方法必须在事件新建（document.createEvent 方法）之后、触发（dispatchEvent 方法）之前调用。

initMouseEvent 方法有很长的参数。

```js
event.initMouseEvent(type, canBubble, cancelable, view,
  detail, screenX, screenY, clientX, clientY,
  ctrlKey, altKey, shiftKey, metaKey,
  button, relatedTarget
);
```

上面这些参数的含义，参见 MouseEvent 构造函数的部分。

模仿并触发 click 事件的写法如下。

```js
var simulateDivClick = document.createEvent('MouseEvents');

simulateDivClick.initMouseEvent('click',true,true,
  document.defaultView,0,0,0,0,0,false,
  false,false,0,null,null
);

divElement.dispatchEvent(simulateDivClick);
```

（2）UIEvent.initUIEvent()

`UIEvent.initUIEvent()`用来初始化一个 UI 事件。该方法必须在事件新建（document.createEvent 方法）之后、触发（dispatchEvent 方法）之前调用。

```js
event.initUIEvent(type, canBubble, cancelable, view, detail)
```

该方法的参数含义，可以参见 MouseEvent 构造函数的部分。其中，detail 参数是一个数值，含义与事件类型有关，对于鼠标事件，这个值表示鼠标按键在某个位置按下的次数。

```js
var e = document.createEvent("UIEvent");
e.initUIEvent("click", true, true, window, 1);
```

## 参考链接

*   Wilson Page, [An Introduction To DOM Events](http://coding.smashingmagazine.com/2013/11/12/an-introduction-to-dom-events/)
*   Mozilla Developer Network, [Using Firefox 1.5 caching](https://developer.mozilla.org/en-US/docs/Using_Firefox_1.5_caching)
*   Craig Buckler, [How to Capture CSS3 Animation Events in JavaScript](http://www.sitepoint.com/css3-animation-javascript-event-handlers/)
*   Ray Nicholus, [You Don't Need jQuery!: Events](http://blog.garstasio.com/you-dont-need-jquery/events/)