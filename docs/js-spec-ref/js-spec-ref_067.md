# 8.1 console 对象

*   浏览器实现
*   console 对象的方法
    *   log()，info()，debug()
    *   warn()，error()
    *   table()
    *   count()
    *   dir()
    *   assert()
    *   time()，timeEnd()
    *   timeline()，timelineEnd()，timeStamp()
    *   profile()，profileEnd()
    *   group()，groupend()，groupCollapsed()
    *   trace()，clear()
    *   命令行 API
    *   debugger 语句
    *   移动端开发
        *   模拟手机视口（viewport）
        *   模拟 touch 事件
        *   模拟经纬度
        *   远程除错
    *   Google Closure
    *   Javascript 性能测试
        *   第一种做法
        *   第二种做法
    *   参考链接

console 对象是 JavaScript 的原生对象，它有点像 Unix 系统的标准输出 stdout 和标准错误 stderr，可以输出各种信息用来调试程序，而且还提供了很多额外的方法，供开发者调用。它的常见用途有两个。

*   显示网页代码运行时的错误信息。

*   提供了一个命令行接口，用来与网页代码互动。

## 浏览器实现

console 对象的浏览器实现，包含在浏览器自带的开发工具之中。以 Chrome 浏览器的“开发者工具”（Developer Tools）为例，首先使用下面三种方法的一种打开它。

1.  按 F12 或者 Control+Shift+i（PC 平台）/ Alt+Command+i（Mac 平台）。

2.  在菜单中选择“工具/开发者工具”。

3.  在一个页面元素上，打开右键菜单，选择其中的“Inspect Element”。

![2015-07-26/55b444d953503](img/67-1.png)

打开“开发者工具”以后，可以看到在顶端有八个面板卡可供选择，分别是：

*   Elements：用来调试网页的 HTML 源码和 CSS 代码。

*   Resources：查看网页加载的各种资源文件（比如代码文件、字体文件、css 文件等），以及在硬盘上创建的各种内容（比如本地缓存、Cookie、Local Storage 等）。

*   Network：查看网页的 HTTP 通信情况。

*   Sources：调试 JavaScript 代码。

*   Timeline：查看各种网页行为随时间变化的情况。

*   Profiles：查看网页的性能情况，比如 CPU 和内存消耗。

*   Audits：提供网页优化的建议。

*   Console：用来运行 JavaScript 命令。

这八个面板都有各自的用途，以下内容针对 Console 面板，又称为控制台。Console 面板基本上就是一个命令行窗口，你可以在提示符下，键入各种命令。

下面介绍 console 对象提供的各种方法。

## console 对象的方法

### log()，info()，debug()

console.log 方法用于在 console 窗口输出信息。它可以接受多个参数，将它们的结果连接起来输出。

```js
console.log("Hello World")
// Hello World

console.log("a","b","c")
// a b c
```

console.log 方法会自动在每次输出的结尾，添加换行符。

```js
console.log(1);
console.log(2);
console.log(3);
// 1
// 2
// 3
```

如果第一个参数是格式字符串（使用了格式占位符），console.log 方法将依次用后面的参数替换占位符，然后再进行输出。

```js
console.log(" %s + %s = %s", 1, 1, 2)
//  1 + 1 = 2
```

上面代码中，console.log 方法的第一个参数有三个占位符（%s），第二、三、四个参数会在显示时，依次替换掉这个三个占位符。console.log 方法支持的占位符有以下几个，不同格式的数据必须使用对应格式的占位符。

*   %s 字符串
*   %d 整数
*   %i 整数
*   %f 浮点数
*   %o 对象的链接
*   %c CSS 格式字符串

```js
var number = 11 * 9;
var color = 'red';

console.log('%d %s balloons', number, color);
// 99 red balloons
```

上面代码中，第二个参数是数值，对应的占位符是%d，第三个参数是字符串，对应的占位符是%s。

使用%c 占位符时，对应的参数必须是 CSS 语句，用来对输出内容进行 CSS 渲染。

```js
console.log('%cThis text is styled!',
  'color: #86CC00; background-color: blue; font-size: 20px; padding: 3px;'
)
```

上面代码运行后，输出的内容将显示为蓝底绿字。

log 方法的两种参数格式，可以结合在一起使用。

```js
console.log(" %s + %s ", 1, 1, "= 2")
// 1 + 1  = 2
```

如果参数是一个对象，console.log 会显示该对象的值。

```js
console.log({foo: 'bar'})
// Object {foo: "bar"}

console.log(Date)
// function Date() { [native code] }
```

上面代码输出 Date 对象的值，结果为一个构造函数。

console.info()和 console.debug()都是 console.log 方法的别名，用法完全一样。console.info 方法会在输出信息的前面，加上一个蓝色图标。

console 对象的所有方法，都可以被覆盖。因此，可以按照自己的需要，定义 console.log 方法。

```js
["log", "info", "warn", "error"].forEach(function(method) {
    console[method] = console[method].bind(
        console,
        new Date().toISOString()
    );
});

console.log("出错了！");
// 2014-05-18T09:00.000Z 出错了！
```

上面代码表示，使用自定义的 console.log 方法，可以在显示结果添加当前时间。

### warn()，error()

warn 方法和 error 方法也是输出信息，它们与 log 方法的不同之处在于，warn 方法输出信息时，在最前面加一个黄色三角，表示警告；error 方法输出信息时，在最前面加一个红色的叉，表示出错，同时会显示错误发生的堆栈。其他用法都一样。

```js
console.error("Error: %s (%i)", "Server is not responding",500)
// Error: Server is not responding (500)

console.warn('Warning! Too few nodes (%d)', document.childNodes.length)
// Warning! Too few nodes (1)
```

本质上，log 方法是写入标准输出（stdout），warn 方法和 error 方法是写入标准错误（stderr）。

### table()

对于某些复合类型的数据，console.table 方法可以将其转为表格显示。

```js
var languages = [
  { name: "JavaScript", fileExtension: ".js" },
  { name: "TypeScript", fileExtension: ".ts" },
  { name: "CoffeeScript", fileExtension: ".coffee" }
];

console.table(languages);
```

上面代码的 language，转为表格显示如下。

| (index) | name | fileExtension |
| --- | --- | --- |
| 0 | "JavaScript" | ".js" |
| 1 | "TypeScript" | ".ts" |
| 2 | "CoffeeScript" | ".coffee" |

复合型数据转为表格显示的条件是，必须拥有主键。对于上面的数组来说，主键就是数字键。对于对象来说，主键就是它的最外层键。

```js
var languages = {
    csharp: { name: "C#", paradigm: "object-oriented" },
    fsharp: { name: "F#", paradigm: "functional" }
};

console.table(languages);
```

上面代码的 language，转为表格显示如下。

| (index) | name | paradigm |
| --- | --- | --- |
| csharp | "C#" | "object-oriented" |
| fsharp | "F#" | "functional" |

### count()

count 方法用于计数，输出它被调用了多少次。

```js
function greet(user) {
  console.count();
  return "hi " + user;
}

greet('bob')
//  : 1
// "hi bob"

greet('alice')
//  : 2
// "hi alice"

greet('bob')
//  : 3
// "hi bob"
```

上面代码每次调用 greet 函数，内部的 console.count 方法就输出执行次数。

该方法可以接受一个字符串作为参数，作为标签，对执行次数进行分类。

```js
function greet(user) {
  console.count(user);
  return "hi " + user;
}

greet('bob')
// bob: 1
// "hi bob"

greet('alice')
// alice: 1
// "hi alice"

greet('bob')
// bob: 2
// "hi bob"
```

上面代码根据参数的不同，显示 bob 执行了两次，alice 执行了一次。

### dir()

dir 方法用来对一个对象进行检查（inspect），并以易于阅读和打印的格式显示。

```js
console.log({f1: 'foo', f2: 'bar'})
// Object {f1: "foo", f2: "bar"}

console.dir({f1: 'foo', f2: 'bar'})
// Object
//   f1: "foo"
//   f2: "bar"
//   __proto__: Object
```

上面代码显示 dir 方法的输出结果，比 log 方法更易读，信息也更丰富。

该方法对于输出 DOM 对象非常有用，因为会显示 DOM 对象的所有属性。

```js
console.dir(document.body)
```

### assert()

assert 方法接受两个参数，第一个参数是表达式，第二个参数是字符串。只有当第一个参数为 false，才会输出第二个参数，否则不会有任何结果。

```js
// 实例
console.assert(true === false, "判断条件不成立")
// Assertion failed: 判断条件不成立
```

下面是另一个例子，判断子节点的个数是否大于等于 500。

```js
console.assert(list.childNodes.length < 500, "节点个数大于等于 500")
```

### time()，timeEnd()

这两个方法用于计时，可以算出一个操作所花费的准确时间。

```js
console.time("Array initialize");

var array= new Array(1000000);
for (var i = array.length - 1; i >= 0; i--) {
    array[i] = new Object();
};

console.timeEnd("Array initialize");

// Array initialize: 1914.481ms
```

time 方法表示计时开始，timeEnd 方法表示计时结束。它们的参数是计时器的名称。调用 timeEnd 方法之后，console 窗口会显示“计时器名称: 所耗费的时间”。

### timeline()，timelineEnd()，timeStamp()

console.timeline 和 console.timelineEnd 这两个方法用于定义一个新的时间线，可以在 Timeline 面板查看。这两个方法只有 Chrome 开发者工具支持。

```js
console.timeline('Google Search');

// Do some work

console.timelineEnd('Google Search');
```

上面代码定义了一个名称叫做“Google Search”的时间线，在这个时间线里的所有事件的耗时，可以在 Timeline 面板中查看。

console.timeStamp 方法用在上面两个方法的中间，用于为时间线添加一个时间戳。时间戳的名字就是 timeStamp 方法的参数。

### profile()，profileEnd()

console.profile 方法用来新建一个性能测试器（profile），它的参数是性能测试器的名字。

```js
console.profile('p')
// Profile 'p' started.
```

console.profileEnd 方法用来结束正在运行的性能测试器。

```js
console.profileEnd()
// Profile 'p' finished.
```

打开浏览器的开发者工具，在 profile 面板中，可以看到这个性能调试器的运行结果。

### group()，groupend()，groupCollapsed()

console.group 和 console.groupend 这两个方法用于将显示的信息分组。它只在输出大量信息时有用，分在一组的信息，可以用鼠标折叠/展开。

```js
console.group('Group One');
console.group('Group Two');

// some code

console.groupEnd(); // Group Two 结束
console.groupEnd(); // Group One 结束
```

console.groupCollapsed 方法与 console.group 方法很类似，唯一的区别是该组的内容，在第一次显示时是收起的（collapsed），而不是展开的。

```js
console.groupCollapsed('Fetching Data');

console.log('Request Sent');
console.error('Error: Server not responding (500)');

console.groupEnd();
```

上面代码只显示一行”Fetching Data“，点击后才会展开，显示其中包含的两行。

### trace()，clear()

console.trace 方法显示当前执行的代码在堆栈中的调用路径。

```js
console.trace()
// console.trace()
//   (anonymous function)
//   InjectedScript._evaluateOn
//   InjectedScript._evaluateAndWrap
//   InjectedScript.evaluate
```

console.clear 方法用于清除当前控制台的所有输出，将光标回置到第一行。

## 命令行 API

在控制台中，除了使用 console 对象，还可以使用一些控制台自带的命令行方法。

（1）$_

$_ 属性返回上一个表达式的值。

```js
2+2
// 4
$_
// 4
```

（2）$0 - $4

控制台保存了最近 5 个在 Elements 面板选中的 DOM 元素，$0 代表倒数第一个，$1 代表倒数第二个，以此类推直到$4。

（3）$(selector)

$(selector)返回一个数组，包括特定的 CSS 选择器匹配的所有 DOM 元素。该方法实际上是 document.querySelectorAll 方法的别名。

```js
var images = $('img');
for (each in images) {
    console.log(images[each].src);
}
```

上面代码打印出网页中所有 img 元素的 src 属性。

（4）$$(selector)

$$(selector)返回一个选中的 DOM 对象，等同于 document.querySelectorAll。

（5）$x(path)

$x(path)方法返回一个数组，包含匹配特定 XPath 表达式的所有 DOM 元素。

```js
$x("//p[a]")
```

上面代码返回所有包含 a 元素的 p 元素。

（6）inspect(object)

inspect(object)方法打开相关面板，并选中相应的元素：DOM 元素在 Elements 面板中显示，JavaScript 对象在 Profiles 中显示。

（7）getEventListeners(object)

getEventListeners(object)方法返回一个对象，该对象的成员为登记了回调函数的各种事件（比如 click 或 keydown），每个事件对应一个数组，数组的成员为该事件的回调函数。

（8）keys(object)，values(object)

keys(object)方法返回一个数组，包含特定对象的所有键名。

values(object)方法返回一个数组，包含特定对象的所有键值。

```js
var o = {'p1':'a', 'p2':'b'};

keys(o)
// ["p1", "p2"]
values(o)
// ["a", "b"]
```

（9）monitorEvents(object[, events]) ，unmonitorEvents(object[, events])

monitorEvents(object[, events])方法监听特定对象上发生的特定事件。当这种情况发生时，会返回一个 Event 对象，包含该事件的相关信息。unmonitorEvents 方法用于停止监听。

```js
monitorEvents(window, "resize");

monitorEvents(window, ["resize", "scroll"])
```

上面代码分别表示单个事件和多个事件的监听方法。

```js
monitorEvents($0, "mouse");
unmonitorEvents($0, "mousemove");
```

上面代码表示如何停止监听。

monitorEvents 允许监听同一大类的事件。所有事件可以分成四个大类。

*   mouse："mousedown", "mouseup", "click", "dblclick", "mousemove", "mouseover", "mouseout", "mousewheel"
*   key："keydown", "keyup", "keypress", "textInput"
*   touch："touchstart", "touchmove", "touchend", "touchcancel"
*   control："resize", "scroll", "zoom", "focus", "blur", "select", "change", "submit", "reset"

```js
monitorEvents($("#msg"), "key");
```

上面代码表示监听所有 key 大类的事件。

（10）profile([name])，profileEnd()

profile 方法用于启动一个特定名称的 CPU 性能测试，profileEnd 方法用于结束该性能测试。

```js
profile("My profile")

profileEnd("My profile")
```

（11）其他方法

命令行 API 还提供以下方法。

*   clear()方法清除控制台的历史。
*   copy(object)方法复制特定 DOM 元素到剪贴板。
*   dir(object)方法显示特定对象的所有属性，是 console.dir 方法的别名。
*   dirxml(object)方法显示特定对象的 XML 形式，是 console.dirxml 方法的别名。

## debugger 语句

debugger 语句必须与除错工具配合使用，如果没有除错工具，debugger 语句不会产生任何结果。

在 chrome 浏览器中，当代码运行到 debugger 指定的行时，就会暂停运行，自动打开 console 界面。它的作用类似于设置断点。

```js
for(var i = 0;i<5;i++){
    console.log(i);
    if (i===2) debugger;
}
```

上面代码打印出 0，1，2 以后，就会暂停，自动打开 console 窗口，等待进一步处理。

## 移动端开发

（本节暂存此处）

### 模拟手机视口（viewport）

chrome 浏览器的开发者工具，提供一个选项，可以模拟手机屏幕的显示效果。

打开“设置”的 Overrides 面板，选择相应的 User Agent 和 Device Metrics 选项。

![选择 User Agent](img/67-2.png)

User Agent 可以使得当前浏览器发出手机浏览器的 Agent 字符串，Device Metrics 则使得当前浏览器的视口变为手机的视口大小。这两个选项可以独立选择，不一定同时选中。

### 模拟 touch 事件

我们可以在 PC 端模拟 JavaScript 的 touch 事件。

首先，打开 chrome 浏览器的开发者工具，选择“设置”中的 Overrides 面板，勾选“Enable touch events”选项。

![Enable touch events 的图片](img/67-3.png)

然后，鼠标就会触发 touchstart、touchmove 和 touchend 事件。（此时，鼠标本身的事件依然有效。）

至于多点触摸，必须要有支持这个功能的设备才能模拟，具体可以参考[Multi-touch web development](http://www.html5rocks.com/en/mobile/touch/)。

### 模拟经纬度

chrome 浏览器的开发者工具，还可以模拟当前的经纬度数据。打开“设置”的 Overrides 面板，选中 Override Geolocation 选项，并填入相应经度和纬度数据。

![模拟经纬度](img/67-4.png)

### 远程除错

(1) Chrome for Android

Android 设备上的 Chrome 浏览器支持 USB 除错。PC 端需要安装 Android SDK 和 Chrome 浏览器，然后用 usb 线将手机和 PC 连起来，可参考[官方文档](https://developers.google.com/chrome-developer-tools/docs/remote-debugging)。

(2) Opera

Opera 浏览器的除错环境 Dragonfly 支持远程除错（[教程](http://www.codegeek.net/blog/2012/mobile-debugging-with-opera-dragonfly/)）。

(3) Firefox for Android

参考[官方文档](https://hacks.mozilla.org/2012/08/remote-debugging-on-firefox-for-android/)。

(4) Safari on iOS6

你可以使用 Mac 桌面电脑的 Safari 6 浏览器，进行远程除错（[教程](http://www.mobilexweb.com/blog/iphone-5-ios-6-html5-developers)）。

## Google Closure

（本节暂存此处）

Google Closure 是 Google 提供的一个 JavaScript 源码处理工具，主要用于压缩和合并多个 JavaScript 脚本文件。

Google Closure 使用 Java 语言开发，使用之前必须先安装 Java。然后，前往[官方网站](https://developers.google.com/closure/)进行下载，这里我们主要使用其中的编译器（compiler）。

首先，查看使用帮助。

```js
java -jar /path/to/closure/compiler.jar --help
```

直接在脚本命令后面跟上要合并的脚本，就能完成合并。

```js
java -jar /path/to/closure/compiler.jar *.js
```

使用--js 参数，可以确保按照参数的先后次序合并文件。

```js
java -jar /path/to/closure/compiler.jar --js script1.js --js script2.js --js script3.js
```

但是，这样的运行结果是将合并后的文件全部输出到屏幕（标准输出），因此需要使用--js_output_file 参数，指定合并后的文件名。

```js
java -jar /path/to/closure/compiler.jar --js script1.js --js script2.js --js script3.js --js_output_file scripts-compiled.js
```

## Javascript 性能测试

(本节暂存此处)

### 第一种做法

最常见的测试性能的做法，就是同一操作重复 n 次，然后计算每次操作的平均时间。

```js
var totalTime,
    start = new Date,
    iterations = 6;

while (iterations--) {
  // Code snippet goes here
}

// totalTime → the number of milliseconds it took to execute
// the code snippet 6 times
totalTime = new Date - start;
```

上面代码的问题在于，由于计算机的性能不断提高，如果只重复 6 次，很可能得到 0 毫秒的结果，即不到 1 毫秒，Javascript 引擎无法测量。

### 第二种做法

另一种思路是，测试单位时间内完成了多少次操作。

```js
var hz,
    period,
    startTime = new Date,
    runs = 0;

do {
  // Code snippet goes here
  runs++;
  totalTime = new Date - startTime;
} while (totalTime < 1000);

// convert ms to seconds
totalTime /= 1000;

// period → how long per operation
period = totalTime / runs;

// hz → the number of operations per second
hz = 1 / period;

// can be shortened to
// hz = (runs * 1000) / totalTime;
```

这种做法的注意之处在于，测试结构受外界环境影响很大，为了得到正确结构，必须重复多次。

## 参考链接

*   Chrome Developer Tools, [Using the Console](https://developers.google.com/chrome-developer-tools/docs/console)
*   Matt West, [Mastering The Developer Tools Console](http://blog.teamtreehouse.com/mastering-developer-tools-console)
*   Firebug Wiki, [Console API](https://getfirebug.com/wiki/index.php/Console_API)
*   Axel Rauschmayer, [The JavaScript console API](http://www.2ality.com/2013/10/console-api.html)
*   Marius Schulz, [Advanced JavaScript Debugging with console.table()](http://blog.mariusschulz.com/2013/11/13/advanced-javascript-debugging-with-consoletable)
*   Google Developer, [Command Line API Reference](https://developers.google.com/chrome-developer-tools/docs/commandline-api)