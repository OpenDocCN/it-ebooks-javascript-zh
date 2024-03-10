# 9.5 ECMAScript 6 介绍

*   概述
*   使用 ECMAScript 6 的方法
*   数据类型
    *   let 命令
    *   const 命令
    *   Set 数据结构
    *   Map 数据结构
    *   rest（...）运算符
    *   遍历器（Iterator）
    *   generator 函数
    *   原生对象的扩展
    *   语法糖
        *   二进制和八进制表示法
        *   增强的对象写法
        *   箭头函数（arrow）
        *   函数参数的默认值
        *   模板字符串
        *   for...of 循环
        *   数组推导
        *   多变量赋值
    *   数据结构
        *   class 结构
        *   module 定义
    *   ECMAScript 7
    *   参考链接

## 概述

ECMAScript 6 是 JavaScript 的下一代标准，正处在快速开发之中，大部分已经完成了，预计将在 2014 年正式发布。Mozilla 将在这个标准的基础上，推出 JavaScript 2.0。

ECMAScript 6 的目标，是使得 JavaScript 可以用来编写复杂的应用程序、函数库和代码的自动生成器（code generator）。

最新的浏览器已经部分支持 ECMAScript 6 的语法，可以通过[《ECMAScript 6 浏览器兼容表》](http://kangax.github.io/es5-compat-table/es6/)查看浏览器支持情况。

下面对 ECMAScript 6 新增的语法特性逐一介绍。由于 ECMAScript 6 的正式标准还未出台，所以以下内容随时可能发生变化，不一定是最后的版本。

## 使用 ECMAScript 6 的方法

目前，V8 引擎已经部署了 ECMAScript 6 的部分特性。使用 node.js 0.11 版，就可以体验这些特性。

node.js 0.11 版的一种比较方便的使用方法，是使用版本管理工具[nvm](https://github.com/creationix/nvm)。下载 nvm 以后，进入项目目录，运行下面的命令，激活 nvm。

```
source nvm.sh
```

然后，指定 node 运行版本。

```
nvm use 0.11
```

最后，用--harmony 参数进入 node 运行环境，就可以在命令行下体验 ECMAScript 6 了。

```
node --harmony
```

另外，可以使用 Google 的[Traceur](https://github.com/google/traceur-compiler)（[在线转换工具](http://google.github.io/traceur-compiler/demo/repl.html)），将 ES6 代码编译为 ES5。

```
# 安装
npm install -g traceur

# 运行 ES6 文件
traceur /path/to/es6

# 将 ES6 文件转为 ES5 文件
traceur --script /path/to/es6 --out /path/to/es5
```

## 数据类型

### let 命令

（1）概述

ECMAScript 6 新增了 let 命令，用来声明变量。它的用法类似于 var，但是所声明的变量，只在 let 命令所在的代码块内有效。

```
{
    let a = 10;
    var b = 1;
}

a // ReferenceError: a is not defined. 
b //1
```

上面代码在代码块之中，分别用 let 和 var 声明了两个变量。然后在代码块之外调用这两个变量，结果 let 声明的变量报错，var 声明的变量返回了正确的值。这表明，let 声明的变量只在它所在的代码块有效。

下面的代码如果使用 var，最后输出的是 9。

```
var a = [];
for (var i = 0; i < 10; i++) {
  var c = i;
  a[i] = function () {
    console.log(c);
  };
}
a[6](); // 9
```

如果使用 let，声明的变量仅在块级作用域内有效，最后输出的是 6。

```
var a = [];
for (var i = 0; i < 10; i++) {
  let c = i;
  a[i] = function () {
    console.log(c);
  };
}
a[6](); // 6
```

注意，let 不允许在相同作用域内，重复声明同一个变量。

```
// 报错
{
    let a = 10;
    var a = 1;
}

// 报错
{
    let a = 10;
    let a = 1;
}
```

（2）块级作用域

let 实际上为 JavaScript 新增了块级作用域。

```
function f1() {
  let n = 5;
  if (true) {
      let n = 10;
  }
  console.log(n); // 5
}
```

上面的函数有两个代码块，都声明了变量 n，运行后输出 5。这表示外层代码块不受内层代码块的影响。如果使用 var 定义变量 n，最后输出的值就是 10。

块级作用域的出现，实际上使得获得广泛应用的立即执行函数（IIFE）不再必要了。

```
// IIFE 写法
(function () { 
    var tmp = ...;
    ...
}()); 

// 块级作用域写法
{
    let tmp = ...;
    ...
}
```

（3）不存在变量提升

需要注意的是，let 声明的变量不存在“变量提升”现象。

```
console.log(x);
let x = 10;
```

上面代码运行后会报错，表示 x 没有定义。如果用 var 声明 x，就不会报错，输出结果为 undefined。

### const 命令

const 也用来声明变量，但是声明的是常量。一旦声明，常量的值就不能改变。

```
const PI = 3.1415;

PI
// 3.1415

PI = 3;

PI
// 3.1415

const PI = 3.1;

PI
// 3.1415
```

上面代码表明改变常量的值是不起作用的。需要注意的是，对常量重新赋值不会报错，只会默默地失败。

> const 的作用域与 var 命令相同：如果在全局环境声明，常量就在全局环境有效；如果在函数内声明，常量就在函数体内有效。

### Set 数据结构

ES6 提供了新的数据结构 Set。它类似于数组，但是成员的值都是唯一的，没有重复的值。

Set 本身是一个构造函数，用来生成 Set 数据结构。

```
var s = new Set();

[2,3,5,4,5,2,2].map(x => s.add(x))
for (i of s) {console.log(i)}
// 2 3 4 5
```

上面代码表示，set 数据结构不会添加重复的值。

set 数据结构有以下属性和方法：

*   size：返回成员总数。
*   add(value)：添加某个值。
*   delete(value)：删除某个值。
*   has(value)：返回一个布尔值，表示该值是否为 set 的成员。
*   clear()：清除所有成员。

```
s.add("1").add("2").add("2"); 
// 注意“2”被加入了两次

s.size // 2

s.has("1")    // true
s.has("2")    // true
s.has("3")   // false

s.delete("2");
s.has("2")    // false
```

### Map 数据结构

ES6 还提供了 map 数据结构。它类似于对象，就是一个键值对的集合，但是“键”的范围不限于字符串，甚至对象也可以当作键。

```
var m = new Map();

o = {p: "Hello World"};
m.set(o, "content")
console.log(m.get(o))
// "content"
```

上面代码将一个对象当作 m 的一个键。

Map 数据结构有以下属性和方法。

*   size：返回成员总数。
*   set(key, value)：设置一个键值对。
*   get(key)：读取一个键。
*   has(key)：返回一个布尔值，表示某个键是否在 Map 数据结构中。
*   delete(key)：删除某个键。
*   clear()：清除所有成员。

```
var m = new Map(); 

m.set("edition", 6)        // 键是字符串
m.set(262, "standard")     // 键是数值
m.set(undefined, "nah")    // 键是 undefined

var hello = function() {console.log("hello");}
m.set(hello, "Hello ES6!") // 键是函数

m.has("edition")     // true
m.has("years")       // false
m.has(262)           // true
m.has(undefined)     // true
m.has(hello)         // true

m.delete(undefined)
m.has(undefined)       // false

m.get(hello)  // Hello ES6!
m.get("edition")  // 6
```

### rest（...）运算符

（1）基本用法

ES6 引入 rest 运算符（...），用于获取函数的多余参数，这样就不需要使用 arguments.length 了。rest 运算符后面是一个数组变量，该变量将多余的参数放入数组中。

```
function add(...values) {
   let sum = 0;

   for (var val of values) {
      sum += val;
   }

   return sum;
}

add(2, 5, 3) // 10
```

上面代码的 add 函数是一个求和函数，利用 rest 运算符，可以向该函数传入任意数目的参数。

下面是一个利用 rest 运算符改写数组 push 方法的例子。

```
function push(array, ...items) { 
  items.forEach(function(item) {
    array.push(item);
    console.log(item);
  });
}

var a = [];
push(a, "a1", "a2", "a3", "a4");
```

（2）将数组转为参数序列

rest 运算符不仅可以用于函数定义，还可以用于函数调用。

```
function f(s1, s2, s3, s4, s5) {
    console.log(s1 + s2 + s3 + s4 +s5);
}

var a = ["a2", "a3", "a4", "a5"];

f("a1", ...a)
// a1a2a3a4a5
```

从上面的例子可以看出，rest 运算符的另一个重要作用是，可以将数组转变成正常的参数序列。利用这一点，可以简化求出一个数组最大元素的写法。

```
// ES5
Math.max.apply(null, [14, 3, 77])

// ES6
Math.max(...[14, 3, 77])

// 等同于
Math.max(14, 3, 77);
```

上面代码表示，由于 JavaScript 不提供求数组最大元素的函数，所以只能套用 Math.max 函数，将数组转为一个参数序列，然后求最大值。有了 rest 运算符以后，就可以直接用 Math.max 了。

### 遍历器（Iterator）

遍历器（Iterator）是一种协议，任何对象都可以部署遍历器协议，从而使得 for...of 循环可以遍历这个对象。

遍历器协议规定，任意对象只要部署了 next 方法，就可以作为遍历器，但是 next 方法必须返回一个包含 value 和 done 两个属性的对象。其中，value 属性当前遍历位置的值，done 属性是一个布尔值，表示遍历是否结束。

```
function makeIterator(array){
    var nextIndex = 0;

    return {
       next: function(){
           return nextIndex < array.length ?
               {value: array[nextIndex++], done: false} :
               {done: true};
       }
    }
}

var it = makeIterator(['a', 'b']);

it.next().value // 'a'
it.next().value // 'b'
it.next().done  // true
```

下面是一个无限运行的遍历器的例子。

```
function idMaker(){
    var index = 0;

    return {
       next: function(){
           return {value: index++, done: false};
       }
    }
}

var it = idMaker();

it.next().value // '0'
it.next().value // '1'
it.next().value // '2'
// ...
```

### generator 函数

上一部分的遍历器，用来依次取出集合中的每一个成员，但是某些情况下，我们需要的是一个内部状态的遍历器。也就是说，每调用一次遍历器，对象的内部状态发生一次改变（可以理解成发生某些事件）。ECMAScript 6 引入了 generator 函数，作用就是返回一个内部状态的遍历器，主要特征是函数内部使用了 yield 语句。

当调用 generator 函数的时候，该函数并不执行，而是返回一个遍历器（可以理解成暂停执行）。以后，每次调用这个遍历器的 next 方法，就从函数体的头部或者上一次停下来的地方开始执行（可以理解成恢复执行），直到遇到下一个 yield 语句为止，并返回该 yield 语句的值。

ECMAScript 6 草案定义的 generator 函数，需要在 function 关键字后面，加一个星号。然后，函数内部使用 yield 语句，定义遍历器的每个成员。

```
function* helloWorldGenerator() {
    yield 'hello';
    yield 'world';
}
```

yield 有点类似于 return 语句，都能返回一个值。区别在于每次遇到 yield，函数返回紧跟在 yield 后面的那个表达式的值，然后暂停执行，下一次从该位置继续向后执行，而 return 语句不具备位置记忆的功能。

上面代码定义了一个 generator 函数 helloWorldGenerator，它的遍历器有两个成员“hello”和“world”。调用这个函数，就会得到遍历器。

```
var hw = helloWorldGenerator();
```

执行遍历器的 next 方法，则会依次遍历每个成员。

```
hw.next() 
// { value: 'hello', done: false }

hw.next()
// { value: 'world', done: false }

hw.next()
// { value: undefined, done: true }

hw.next()
// Error: Generator has already finished
// at GeneratorFunctionPrototype.next (native)
// at repl:1:3
//  at REPLServer.defaultEval (repl.js:129:27)
//  ...
```

上面代码一共调用了四次 next 方法。

*   第一次调用：函数开始执行，直到遇到第一句 yield 语句为止。next 方法返回一个对象，它的 value 属性就是当前 yield 语句的值 hello，done 属性的值 false，表示遍历还没有结束。

*   第二次调用：函数从上次 yield 语句停下的地方，一直执行到下一个 yield 语句。next 方法返回一个对象，它的 value 属性就是当前 yield 语句的值 world，done 属性的值 false，表示遍历还没有结束。

*   第三次调用：函数从上次 yield 语句停下的地方，一直执行到函数结束。next 方法返回一个对象，它的 value 属性就是函数最后的返回值，由于上例的函数没有 return 语句（即没有返回值），所以 value 属性的值为 undefined，done 属性的值 true，表示遍历已经结束。

*   第四次调用：由于此时函数已经运行完毕，next 方法直接抛出一个错误。

遍历器的本质，其实是使用 yield 语句暂停执行它后面的操作，当调用 next 方法时，再继续往下执行，直到遇到下一个 yield 语句，并返回该语句的值，如果直到运行结束。

如果 next 方法带一个参数，该参数就会被当作上一个 yield 语句的返回值。

```
function* f() {
  for(var i=0; true; i++) {
    var reset = yield i;
    if(reset) { i = -1; }
  }
}

var g = f();

g.next() // { value: 0, done: false }
g.next() // { value: 1, done: false }
g.next(true) // { value: 0, done: false }
```

上面代码先定义了一个可以无限运行的 generator 函数 f，如果 next 方法没有参数，正常情况下返回一个递增的 i；如果 next 方法有参数，则上一次 yield 语句的返回值将会等于该参数。如果该参数为 true，则会重置 i 的值。

generator 函数的这种暂停执行的效果，意味着可以把异步操作写在 yield 语句里面，等到调用 next 方法时再往后执行。这实际上等同于不需要写回调函数了，因为异步操作的后续操作可以放在 yield 语句下面，反正要等到调用 next 方法时再执行。所以，generator 函数的一个重要实际意义就是用来处理异步操作，改写回调函数。

```
function* loadUI() { 
    showLoadingScreen(); 
    yield loadUIDataAsynchronously(); 
    hideLoadingScreen(); 
}
```

上面代码表示，第一次调用 loadUI 函数时，该函数不会执行，仅返回一个遍历器。下一次对该遍历器调用 next 方法，则会显示登录窗口，并且异步加载数据。再一次使用 next 方法，则会隐藏登录窗口。可以看到，这种写法的好处是所有登录窗口的逻辑，都被封装在一个函数，按部就班非常清晰。

下面是一个利用 generator 函数，实现斐波那契数列的例子。

```
function* fibonacci() {
    var previous = 0, current = 1; 
    while (true) { 
        var temp = previous; 
        previous = current; 
        current = temp + current; 
        yield current; 
    } 
} 

for (var i of fibonacci()) { 
    console.log(i); 
} 
// 1, 2, 3, 5, 8, 13, ...,
```

下面是利用 for...of 语句，对斐波那契数列的另一种实现。

```
function* fibonacci() {
    let [prev, curr] = [0, 1];
    for (;;) {
        [prev, curr] = [curr, prev + curr];
        yield curr;
    }
}

for (n of fibonacci()) {
    if (n > 1000) break;
    console.log(n);
}
```

从上面代码可见，使用 for...of 语句时不需要使用 next 方法。

这里需要注意的是，yield 语句运行的时候是同步运行，而不是异步运行（否则就失去了取代回调函数的设计目的了）。实际操作中，一般让 yield 语句返回 Promises 对象。

```
var Q = require('q');

function delay(milliseconds) {
    var deferred = Q.defer();
    setTimeout(deferred.resolve, milliseconds);
    return deferred.promise;
}

function *f(){
    yield delay(100);
};
```

上面代码 yield 语句返回的就是一个 Promises 对象。

如果有一系列任务需要全部完成后，才能进行下一步操作，yield 语句后面可以跟一个数组。下面就是一个例子。

```
function *f() {
    var urls = [
        'http://example.com/',
        'http://twitter.com/',
        'http://bbc.co.uk/news/'
    ];
    var arrayOfPromises = urls.map(someOperation);

    var arrayOfResponses = yield arrayOfPromises;

    this.body = "Results";
    for (var i = 0; i < urls.length; i++) {
        this.body += '\n' + urls[i] + ' response length is '
              + arrayOfResponses[i].body.length;
    }
};
```

### 原生对象的扩展

ES6 对 JavaScript 的原生对象，进行了扩展，提供了一系列新的属性和方法。

```
Number.EPSILON
Number.isInteger(Infinity) // false
Number.isNaN("NaN") // false

Math.acosh(3) // 1.762747174039086
Math.hypot(3, 4) // 5
Math.imul(Math.pow(2, 32) - 1, Math.pow(2, 32) - 2) // 2

"abcde".contains("cd") // true
"abc".repeat(3) // "abcabcabc"

Array.from(document.querySelectorAll('*')) // Returns a real Array
Array.of(1, 2, 3) // Similar to new Array(...), but without special one-arg behavior
[0, 0, 0].fill(7, 1) // [0,7,7]
[1,2,3].findIndex(x => x == 2) // 1
["a", "b", "c"].entries() // iterator [0, "a"], [1,"b"], [2,"c"]
["a", "b", "c"].keys() // iterator 0, 1, 2
["a", "b", "c"].values() // iterator "a", "b", "c"

Object.assign(Point, { origin: new Point(0,0) })
```

## 语法糖

ECMAScript 6 提供了很多 JavaScript 语法的便捷写法。

### 二进制和八进制表示法

ES6 提供了二进制和八进制数值的新的写法，分别用前缀 0b 和 0o 表示。

```
0b111110111 === 503 // true
0o767 === 503 // true
```

### 增强的对象写法

ES6 允许直接写入变量和函数，作为对象的属性和方法。这样的书写更加简洁。

```
var Person = {
  name: '张三',
  //等同于 birth: birth
  birth,
  // 等同于 hello: function ()...
  hello() { console.log('我的名字是', this.name); }
};
```

### 箭头函数（arrow）

（1）定义

ES6 允许使用“箭头”（=>）定义函数。

```
var f = v => v;
```

上面的箭头函数等同于：

```
var f = function(v) {
    return v;
};
```

如果箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分。

```
var f = () => 5; 
// 等同于
var f = function (){ return 5 };

var sum = (num1, num2) => num1 + num2;
// 等同于
var sum = function(num1, num2) {
    return num1 + num2;
};
```

如果箭头函数的代码块部分多于一条语句，就要使用大括号将它们括起来，并且使用 return 语句返回。

```
var sum = (num1, num2) => { return num1 + num2; }
```

由于大括号被解释为代码块，所以如果箭头函数直接返回一个对象，必须在对象外面加上括号。

```
var getTempItem = id => ({ id: id, name: "Temp" });
```

（2）实例：回调函数的简化

箭头函数的一个用处是简化回调函数。

```
// 正常函数写法
[1,2,3].map(function (x) {
  return x * x;
});

// 箭头函数写法
[1,2,3].map(x => x * x);
```

另一个例子是

```
// 正常函数写法
var result = values.sort(function(a, b) {
    return a - b;
});

// 箭头函数写法
var result = values.sort((a, b) => a - b);
```

（3）注意点

箭头函数有几个使用注意点。

*   函数体内的 this 对象，绑定定义时所在的对象，而不是使用时所在的对象。
*   不可以当作构造函数，也就是说，不可以使用 new 命令，否则会抛出一个错误。
*   不可以使用 arguments 对象，该对象在函数体内不存在。

关于 this 对象，下面的代码将它绑定定义时的对象。

```
var handler = {

    id: "123456",

    init: function() {
        document.addEventListener("click",
                event => this.doSomething(event.type), false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```

上面代码的 init 和 doSomething 方法中，都使用了箭头函数，它们中的 this 都绑定 handler 对象。否则，doSomething 方法内部的 this 对象就指向全局对象，运行时会报错。

### 函数参数的默认值

ECMAScript 6 允许为函数的参数设置默认值。

```
function Point(x = 0, y = 0) {
   this.x = x;
   this.y = y;
}

var p = new Point(); 
// p = { x:0, y:0 }
```

### 模板字符串

模板字符串（template string）是增强版的字符串，即可以当作普通字符串使用，也可以在字符串中嵌入变量。它用反引号（`）标识。

```
// 普通字符串
`In JavaScript '\n' is a line-feed.`

// 多行字符串
`In JavaScript this is
 not legal.`

// 字符串中嵌入变量
var name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`

var x = 1;
var y = 2;
console.log(`${ x } + ${ y } = ${ x + y}`) 
// "1 + 2 = 3"
```

### for...of 循环

JavaScript 原有的 for...in 循环，只能获得对象的键名，不能直接获取键值。ES6 提供 for...of 循环，允许遍历获得键值。

```
var arr = ["a", "b", "c", "d"];
for (a in arr) {
  console.log(a);
}
// 0
// 1
// 2
// 3

for (a of arr) {
  console.log(a); 
}
// a
// b
// c
// d
```

上面代码表明，for...in 循环读取键名，for...of 循环读取键值。

for...of 循环还可以遍历对象。

```
var es6 = {
  edition: 6,
  committee: "TC39",
  standard: "ECMA-262"
};

for (e in es6) {
  console.log(e);
}
// edition
// committee
// standard

var engines = Set(["Gecko", "Trident", "Webkit", "Webkit"]);
for (var e of engines) {
    console.log(e);
}
// Gecko
// Trident
// Webkit

var es6 = new Map();
es6.set("edition", 6);
es6.set("committee", "TC39");
es6.set("standard", "ECMA-262");
for (var [name, value] of es6) {
  console.log(name + ": " + value);
}
// edition: 6
// committee: TC39
// standard: ECMA-262
```

上面代码一共包含三个例子，第一个是 for...in 循环的例子，后两个是 for...of 循环的例子。最后一个例子是同时遍历对象的键名和键值。

### 数组推导

（1）基本用法

ES6 提供简洁写法，允许直接通过现有数组生成新数组，这被称为数组推导（array comprehension）。

```
var a1 = [1, 2, 3, 4];
var a2 = [i * 2 for (i of a1)];

a2 // [2, 4, 6, 8]
```

上面代码表示，通过 for...of 结构，数组 a2 直接在 a1 的基础上生成。

数组推导可以替代 map 和 filter 方法。

```
[for (i of [1, 2, 3]) i * i];
// 等价于
[1, 2, 3].map(function (i) { return i * i });

[i for (i of [1,4,2,3,-8]) if (i < 3)];
// 等价于
[1,4,2,3,-8].filter(function(i) { return i < 3 });
```

上面代码说明，模拟 map 功能只要单纯的 for...of 循环就行了，模拟 filter 功能除了 for...of 循环，还必须加上 if 语句。

（2）多重推导

新引入的 for...of 结构，可以直接跟在表达式的前面或后面，甚至可以在一个数组推导中，使用多个 for...of 结构。

```
var a1 = ["x1", "y1"];
var a2 = ["x2", "y2"];
var a3 = ["x3", "y3"];

[(console.log(s + w + r)) for (s of a1) for (w of a2) for (r of a3)];
// x1x2x3
// x1x2y3
// x1y2x3
// x1y2y3
// y1x2x3
// y1x2y3
// y1y2x3
// y1y2y3
```

上面代码在一个数组推导之中，使用了三个 for...of 结构。

需要注意的是，数组推导的方括号构成了一个单独的作用域，在这个方括号中声明的变量类似于使用 let 语句声明的变量。

（3）字符串推导

由于字符串可以视为数组，因此字符串也可以直接用于数组推导。

```
[c for (c of 'abcde') if (/[aeiou]/.test(c))].join('') // 'ae'

[c+'0' for (c of 'abcde')].join('') // 'a0b0c0d0e0'
```

上面代码使用了数组推导，对字符串进行处理。

上一部分的数组推导有一个缺点，就是新数组会立即在内存中生成。这时，如果原数组是一个很大的数组，将会非常耗费内存。

### 多变量赋值

ES6 允许简洁地对多变量赋值。正常情况下，将数组元素赋值给多个变量，只能一次次分开赋值。

```
var a = 1;
var b = 2;
var c = 3;
```

ES6 允许写成下面这样。

```
var [a, b, c] = [1, 2, 3];
```

本质上，这种写法属于模式匹配，只要等号两边的模式相同，左边的变量就会被赋予对应的值。下面是一些嵌套数组的例子。

```
var [foo, [[bar], baz]] = [1, [[2], 3]]

var [,,third] = ["foo", "bar", "baz"]

var [head, ...tail] = [1, 2, 3, 4]
```

它还可以接受默认值。

```
var [missing = true] = [];
console.log(missing)
// true

var { x = 3 } = {};
console.log(x)
// 3
```

它不仅可以用于数组，还可以用于对象。

```
var { foo, bar } = { foo: "lorem", bar: "ipsum" };

foo // "lorem"
bar // "ipsum"

var o = {
  p1: [
    "Hello",
    { p2: "World" }
  ]
};

var { a: [p1, { p2 }] } = o;

console.log(p1)
// "Hello"

console.log(p2)
// "World"
```

这种写法的用途很多。

（1）交换变量的值。

```
[x, y] = [y, x];
```

（2）从函数返回多个值。

```
function example() {
    return [1, 2, 3];
}

var [a, b, c] = example();
```

（3）函数参数的定义。

```
function f({p1, p2, p3}) {
  // ...
}
```

（4）函数参数的默认值。

```
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more config
}) {
  // ... do stuff
};
```

## 数据结构

### class 结构

（1）基本用法

ES6 提供了“类”（class）。此前，一般用构造函数模拟“类”。

```
// ES5
var Language = function(config) {
  this.name = config.name;
  this.founder = config.founder;
  this.year = config.year;
};

Language.prototype.summary = function() {
  return this.name+"由"+this.founder+"在"+this.year+"创造";
};

// ES6
class Language {
  constructor(name, founder, year) {
    this.name = name;
    this.founder = founder;
    this.year = year;
  }

  summary() {
    return this.name+"由"+this.founder+"在"+this.year+"创造";
  }
}
```

在上面代码中，ES6 用 constructor 方法，代替 ES5 的构造函数。

（2）继承

ES6 的 class 结构还允许使用 extends 关键字，表示继承。

```
class MetaLanguage extends Language {
  constructor(x, y, z, version) {
    super(x, y, z);
    this.version = version;
  }
  summary() {
    //...
    super.summary();
  }
}
```

上面代码的 super 方法，表示调用父类的构造函数。

### module 定义

（1）基本用法

ES6 允许定义模块。也就是说，允许一个 JavaScript 脚本文件调用另一个脚本文件。

假设有一个 circle.js，它是一个单独模块。

```
// circle.js

export function area(radius) {
  return Math.PI * radius * radius;
}

export function circumference(radius) {
  return 2 * Math.PI * radius;
}
```

然后，main.js 引用这个模块。

```
// main.js

import { area, circumference } from 'circle';

console.log("圆面积：" + area(4));
console.log("圆周长：" + circumference(14));
```

另一种写法是整体加载 circle.js。

```
// main.js

module circle from 'circle';

console.log("圆面积：" + circle.area(4));
console.log("圆周长：" + circle.circumference(14));
```

（2）模块的继承

一个模块也可以继承另一个模块。

```
// circleplus.js

export * from 'circle';
export var e = 2.71828182846;
export default function(x) {
    return Math.exp(x);
}
```

加载上面的模块。

```
// main.js

module math from "circleplus";
import exp from "circleplus";
console.log(exp(math.pi);
```

（3）模块的默认方法

还可以为模块定义默认方法。

```
// circleplus.js

export default function(x) {
    return Math.exp(x);
}
```

## ECMAScript 7

2013 年 3 月，ECMAScript 6 的草案封闭，不再接受新功能了。新的功能将被加入 ECMAScript 7。根据 JavaScript 创造者 Brendan Eich 的[设想](http://wiki.ecmascript.org/doku.php?id=harmony:harmony)，ECMAScript 7 将使得 JavaScript 更适于开发复杂的应用程序和函数库。

ECMAScript 7 可能包括的功能有：

*   Object.observe：对象与网页元素的双向绑定，只要其中之一发生变化，就会自动反映在另一者上。

*   Multi-Threading：多线程支持。目前，Intel 和 Mozilla 有一个共同的研究项目 RiverTrail，致力于让 JavaScript 多线程运行。预计这个项目的研究成果会被纳入 ECMAScript 标准。

*   Traits：它将是“类”功能（class）的一个替代。通过它，不同的对象可以分享同样的特性。

其他可能包括的功能还有：更精确的数值计算、改善的内存回收、增强的跨站点安全、类型化的更贴近硬件的（Typed, Low-level）操作、国际化支持（Internationalization Support）、更多的数据结构等等。

## 参考链接

*   Sayanee Basu, [Use ECMAScript 6 Today](http://net.tutsplus.com/articles/news/ecmascript-6-today/)
*   Ariya Hidayat, [Toward Modern Web Apps with ECMAScript 6](http://www.sencha.com/blog/toward-modern-web-apps-with-ecmascript-6/)
*   Nick Fitzgerald, [Destructuring Assignment in ECMAScript 6](http://fitzgeraldnick.com/weblog/50/)
*   jmar777, [What's the Big Deal with Generators?](http://devsmash.com/blog/whats-the-big-deal-with-generators)
*   Nicholas C. Zakas, [Understanding ECMAScript 6 arrow functions](http://www.nczonline.net/blog/2013/09/10/understanding-ecmascript-6-arrow-functions/)
*   Dale Schouten, [10 Ecmascript-6 tricks you can perform right now](http://html5hub.com/10-ecmascript-6-tricks-you-can-perform-right-now/)
*   Mozilla Developer Network, [Iterators and generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators)
*   Steven Sanderson, [Experiments with Koa and JavaScript Generators](http://blog.stevensanderson.com/2013/12/21/experiments-with-koa-and-javascript-generators/)
*   Matt Baker, [Replacing callbacks with ES6 Generators](http://flippinawesome.org/2014/02/10/replacing-callbacks-with-es6-generators/)
*   Domenic Denicola, [ES6: The Awesome Parts](http://www.slideshare.net/domenicdenicola/es6-the-awesome-parts)
*   Casper Beyer, [ECMAScript 6 Features and Tools](http://caspervonb.github.io/2014/03/05/ecmascript6-features-and-tools.html)
*   Luke Hoban, [ES6 features](https://github.com/lukehoban/es6features)