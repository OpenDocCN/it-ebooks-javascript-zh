# 4.1 概述

*   简介
    *   对象和面向对象编程
    *   构造函数
    *   new 命令
    *   new 命令的原理
    *   instanceof 运算符
    *   this 关键字
        *   涵义
        *   使用场合
        *   使用注意点
    *   固定 this 的方法
        *   call 方法
        *   apply 方法
        *   bind 方法
    *   参考链接

## 简介

### 对象和面向对象编程

“面向对象编程”（Object Oriented Programming，缩写为 OOP）是目前主流的编程范式。它的核心思想是将真实世界中各种复杂的关系，抽象为一个个对象，然后由对象之间的分工与合作，完成对真实世界的模拟。

传统的计算机程序由一系列函数或一系列指令组成，而面向对象编程的程序由一系列对象组成。每一个对象都是功能中心，具有明确分工，可以完成接受信息、处理数据、发出信息等任务。因此，面向对象编程具有灵活性、代码的可重用性、模块性等特点，容易维护和开发，非常适合多人合作的大型软件项目。

那么，“对象”（object）到底是什么？

我们从两个层次来理解。

（1）“对象”是单个实物的抽象。

一本书、一辆汽车、一个人都可以是“对象”，一个数据库、一张网页、一个与远程服务器的连接也可以是“对象”。当实物被抽象成“对象”，实物之间的关系就变成了“对象”之间的关系，从而就可以模拟现实情况，针对“对象”进行编程。

（2）“对象”是一个容器，封装了“属性”（property）和“方法”（method）。

所谓“属性”，就是对象的状态；所谓“方法”，就是对象的行为（完成某种任务）。比如，我们可以把动物抽象为 animal 对象，“属性”记录具体是那一种动物，“方法”表示动物的某种行为（奔跑、捕猎、休息等等）。

虽然不同于传统的面向对象编程语言，但是 JavaScript 具有很强的面向对象编程能力。本章介绍 JavaScript 如何进行“面向对象编程”。

### 构造函数

“面向对象编程”的第一步，就是要生成对象。

前面说过，“对象”是单个实物的抽象。所以，通常需要一个模板，表示某一类实物的共同特征，然后“对象”根据这个模板生成。

典型的面向对象编程语言（比如 C++和 Java），存在“类”（class）这样一个概念。所谓“类”就是对象的模板，对象就是“类”的实例。JavaScript 语言没有“类”，而改用构造函数（constructor）作为对象的模板。

所谓“构造函数”，就是专门用来生成“对象”的函数。它提供模板，作为对象的基本结构。一个构造函数，可以生成多个对象，这些对象都有相同的结构。

构造函数是一个正常的函数，但是它的特征和用法与普通函数不一样。下面就是一个构造函数：

```
var Vehicle = function() {
  this.price = 1000;
};
```

上面代码中，Vehicle 就是构造函数，它提供模板，用来生成车辆对象。

构造函数的最大特点就是，函数体内部使用了 this 关键字，代表了所要生成的对象实例。生成对象的时候，必需用 new 命令，调用 Vehicle 函数。

### new 命令

new 命令的作用，就是执行构造函数，返回一个实例对象。

```
var Vehicle = function (){
  this.price = 1000;
};

var v = new Vehicle();
v.price // 1000
```

上面代码通过 new 命令，让构造函数 Vehicle 生成一个实例对象，保存在变量 v 中。这个新生成的实例对象，从构造函数 Vehicle 继承了 price 属性。在 new 命令执行时，构造函数内部的 this，就代表了新生成的实例对象，this.price 表示实例对象有一个 price 属性，它的值是 1000。

使用 new 命令时，根据需要，构造函数也可以接受参数。

```
var Vehicle = function (p){
  this.price = p;
};

var v = new Vehicle(500);
```

new 命令本身就可以执行构造函数，所以后面的构造函数可以带括号，也可以不带括号。下面两行代码是等价的。

```
var v = new Vehicle();
var v = new Vehicle;
```

一个很自然的问题是，如果忘了使用 new 命令，直接调用构造函数会发生什么事？

这种情况下，构造函数就变成了普通函数，并不会生成实例对象。而且由于下面会说到的原因，this 这时代表全局对象，将造成一些意想不到的结果。

```
var Vehicle = function (){
  this.price = 1000;
};

var v = Vehicle();
v.price
// Uncaught TypeError: Cannot read property 'price' of undefined

price
// 1000
```

上面代码中，调用 Vehicle 构造函数时，忘了加上 new 命令。结果，price 属性变成了全局变量，而变量 v 变成了 undefined。

因此，应该非常小心，避免出现不使用 new 命令、直接调用构造函数的情况。为了保证构造函数必须与 new 命令一起使用，一个解决办法是，在构造函数内部使用严格模式，即第一行加上`use strict`。

```
function Fubar(foo, bar){
  "use strict";

  this._foo = foo;
  this._bar = bar;
}

Fubar()
// TypeError: Cannot set property '_foo' of undefined
```

上面代码的 Fubar 为构造函数，use strict 命令保证了该函数在严格模式下运行。由于在严格模式中，函数内部的 this 不能指向全局对象，默认等于 undefined，导致不加 new 调用会报错（JavaScript 不允许对 undefined 添加属性）。

另一个解决办法，是在构造函数内部判断是否使用 new 命令，如果发现没有使用，则直接返回一个实例对象。

```
function Fubar(foo, bar){
  if (!(this instanceof Fubar)) {
    return new Fubar(foo, bar);
  }

  this._foo = foo;
  this._bar = bar;
}

Fubar(1, 2)._foo // 1
(new Fubar(1, 2))._foo // 1
```

上面代码中的构造函数，不管加不加 new 命令，都会得到同样的结果。

### new 命令的原理

使用 new 命令时，它后面的函数调用就不是正常的调用，而是被 new 命令控制了。内部的流程是，先创造一个空对象，作为上下文对象，赋值给函数内部的 this 关键字。也就是说，this 指的是一个新生成的空对象，所有针对 this 的操作，都会发生在这个空对象上。

构造函数之所以叫“构造函数”，就是说这个函数的目的，就是操作上下文对象（即 this 对象），将其“构造”为需要的样子。如果构造函数的 return 语句返回的是对象，new 命令会返回 return 语句指定的对象；否则，就会不管 return 语句，返回构造后的上下文对象。

```
var Vehicle = function (){
  this.price = 1000;
  return 1000;
};

(new Vehicle()) === 1000
// false
```

上面代码中，Vehicle 是一个构造函数，它的 return 语句返回一个数值。这时，new 命令就会忽略这个 return 语句，返回“构造”后的 this 对象。

但是，如果 return 语句返回的是一个跟 this 无关的新对象，new 命令会返回这个新对象，而不是 this 对象。这一点需要特别引起注意。

```
var Vehicle = function (){
  this.price = 1000;
  return { price: 2000 };
};

(new Vehicle()).price
// 2000
```

上面代码中，构造函数 Vehicle 的 return 语句，返回的是一个新对象。new 命令会返回这个对象，而不是 this 对象。

new 命令简化的内部流程，可以用下面的代码表示。

```
function _new(/* constructor, param, ... */) {
  var args = [].slice.call(arguments);
  var constructor = args.shift();
  var context = Object.create(constructor.prototype);
  var result = constructor.apply(context, args);
  return (typeof result === 'object' && result != null) ? result : context;
}

var actor = _new(Person, "张三", 28);
```

### instanceof 运算符

instanceof 运算符用来确定一个对象是否为某个构造函数的实例。

```
var v = new Vehicle();

v instanceof Vehicle
// true
```

instanceof 运算符的左边放置对象，右边放置构造函数。在 JavaScript 之中，只要是对象，就有对应的构造函数。因此，instanceof 运算符可以用来判断值的类型。

```
[1, 2, 3] instanceof Array // true

({}) instanceof Object // true
```

上面代码表示数组和对象则分别是 Array 对象和 Object 对象的实例。最后那一行的空对象外面，之所以要加括号，是因为如果不加，JavaScript 引擎会把一对大括号解释为一个代码块，而不是一个对象，从而导致这一行代码被解释为“{}; instanceof Object”，引擎就会报错。

需要注意的是，由于原始类型的值不是对象，所以不能使用 instanceof 运算符判断类型。

```
"" instanceof String // false
1 instanceof Number // false
```

上面代码中，字符串不是 String 对象的实例（因为字符串不是对象），数值 1 也不是 Number 对象的实例（因为数值 1 不是对象）。

如果存在继承关系，也就是某个对象可能是多个构造函数的实例，那么 instanceof 运算符对这些构造函数都返回 true。

```
var a = [];

a instanceof Array // true
a instanceof Object // true
```

上面代码表示，a 是一个数组，所以它是 Array 的实例；同时，a 也是一个对象，所以它也是 Object 的实例。

利用 instanceof 运算符，还可以巧妙地解决，调用构造函数时，忘了加 new 命令的问题。

```
function Fubar (foo, bar) {
  if (this instanceof Fubar) {
    this._foo = foo;
    this._bar = bar;
  }
  else return new Fubar(foo, bar);
}
```

上面代码使用 instanceof 运算符，在函数体内部判断 this 关键字是否为构造函数 Fubar 的实例。如果不是，就表明忘了加 new 命令。

## this 关键字

### 涵义

构造函数内部需要用到 this 关键字。那么，this 关键字到底是什么意思呢？

简单说，this 就是指函数当前的运行环境。在 JavaScript 语言之中，所有函数都是在某个运行环境之中运行，this 就是这个环境。对于 JavaScipt 语言来说，一切皆对象，运行环境也是对象，所以可以理解成，所有函数总是在某个对象之中运行，this 就指向这个对象。这本来并不会让用户糊涂，但是 JavaScript 支持运行环境动态切换，也就是说，this 的指向是动态的，没有办法事先确定到底指向哪个对象，这才是最让初学者感到困惑的地方。

举例来说，有一个函数 f，它同时充当 a 对象和 b 对象的方法。JavaScript 允许函数 f 的运行环境动态切换，即一会属于 a 对象，一会属于 b 对象，这就要靠 this 关键字来办到。

```
function f(){ console.log(this.x); };

var a = {x:'a'};
a.m = f;

var b = {x:'b'};
b.m = f;

a.m() // a
b.m() // b
```

上面代码中，函数 f 可以打印出当前运行环境中 x 变量的值。当 f 属于 a 对象时，this 指向 a；当 f 属于 b 对象时，this 指向 b，因此打印出了不同的值。由于 this 的指向可变，所以可以手动切换运行环境，以达到某种特定的目的。

前面说过，所谓“运行环境”就是对象，this 指函数运行时所在的那个对象。如果一个函数在全局环境中运行，this 就是指顶层对象（浏览器中为 window 对象）；如果一个函数作为某个对象的方法运行，this 就是指那个对象。

可以近似地认为，this 是所有函数运行时的一个隐藏参数，决定了函数的运行环境。

### 使用场合

this 的使用可以分成以下几个场合。

（1）全局环境

在全局环境使用 this，它指的就是顶层对象 window。

```
this === window // true 

function f() {
    console.log(this === window); // true
}
```

上面代码说明，不管是不是在函数内部，只要是在全局环境下运行，this 就是指全局对象 window。

（2）构造函数

构造函数中的 this，指的是实例对象。

```
var O = function(p) {
    this.p = p;
};

O.prototype.m = function() {
    return this.p;
};
```

上面代码定义了一个构造函数 O。由于 this 指向实例对象，所以在构造函数内部定义 this.p，就相当于定义实例对象有一个 p 属性；然后 m 方法可以返回这个 p 属性。

```
var o = new O("Hello World!");

o.p // "Hello World!"
o.m() // "Hello World!"
```

（3）对象的方法

当 a 对象的方法被赋予 b 对象，该方法就变成了普通函数，其中的 this 就从指向 a 对象变成了指向 b 对象。这就是 this 取决于运行时所在的对象的含义，所以要特别小心。如果将某个对象的方法赋值给另一个对象，会改变 this 的指向。

```
var o1 = new Object();
o1.m = 1;
o1.f = function (){ console.log(this.m);};

o1.f() // 1

var o2 = new Object();
o2.m = 2;
o2.f = o1.f

o2.f() // 2
```

从上面代码可以看到，f 是 o1 的方法，但是如果在 o2 上面调用这个方法，f 方法中的 this 就会指向 o2。这就说明 JavaScript 函数的运行环境完全是动态绑定的，可以在运行时切换。

如果不想改变 this 的指向，可以将 o2.f 改写成下面这样。

```
o2.f = function (){ o1.f() };

o2.f() // 1
```

上面代码表示，由于 f 方法这时是在 o1 下面运行，所以 this 就指向 o1。

有时，某个方法位于多层对象的内部，这时如果为了简化书写，把该方法赋值给一个变量，往往会得到意想不到的结果。

```
var a = {
        b : {
            m : function() {
                console.log(this.p);
            },
            p : 'Hello'
        }
};

var hello = a.b.m;
hello() // undefined
```

上面代码表示，m 属于多层对象内部的一个方法。为求简写，将其赋值给 hello 变量，结果调用时，this 指向了全局对象。为了避免这个问题，可以只将 m 所在的对象赋值给 hello，这样调用时，this 的指向就不会变。

```
var hello = a.b;
hello.m() // Hello
```

（4）Node.js

在 Node.js 中，this 的指向又分成两种情况。全局环境中，this 指向全局对象 global；模块环境中，this 指向 module.exports。

```
// 全局环境
this === global // true

// 模块环境
this === module.exports // true
```

### 使用注意点

（1）避免多层 this

由于 this 的指向是不确定的，所以切勿在函数中包含多层的 this。

```
var o = {
    f1: function() {
        console.log(this); 
        var f2 = function() {
            console.log(this);
        }();
    }
}

o.f1()
// Object
// Window
```

上面代码包含两层 this，结果运行后，第一层指向该对象，第二层指向全局对象。一个解决方法是在第二层改用一个指向外层 this 的变量。

```
var o = {
    f1: function() {
        console.log(this); 
        var that = this;
        var f2 = function() {
            console.log(that);
        }();
    }
}

o.f1()
// Object
// Object
```

上面代码定义了变量 that，固定指向外层的 this，然后在内层使用 that，就不会发生 this 指向的改变。

（2）避免数组处理方法中的 this

数组的 map 和 foreach 方法，允许提供一个函数作为参数。这个函数内部不应该使用 this。

```
var o = {
    v: 'hello',
    p: [ 'a1', 'a2' ],
    f: function f() {
        this.p.forEach(function (item) {
            console.log(this.v+' '+item);
        });
    }
}

o.f()
// undefined a1
// undefined a2
```

上面代码中，foreach 方法的参数函数中的 this，其实是指向 window 对象，因此取不到 o.v 的值。

解决这个问题的一种方法，是使用中间变量。

```
var o = {
    v: 'hello',
    p: [ 'a1', 'a2' ],
    f: function f() {
        var that = this;
        this.p.forEach(function (item) {
            console.log(that.v+' '+item);
        });
    }
}

o.f()
// hello a1
// hello a2
```

另一种方法是将 this 当作 foreach 方法的第二个参数，固定它的运行环境。

```
var o = {
  v: 'hello',
    p: [ 'a1', 'a2' ],
    f: function f() {
        this.p.forEach(function (item) {
            console.log(this.v+' '+item);
        }, this);
    }
}

o.f()
// hello a1
// hello a2
```

（3）避免回调函数中的 this

回调函数中的 this 往往会改变指向，最好避免使用。

```
var o = new Object();

o.f = function (){
    console.log(this === o);
}

o.f() // true
```

上面代码表示，如果调用 o 对象的 f 方法，其中的 this 就是指向 o 对象。

但是，如果将 f 方法指定给某个按钮的 click 事件，this 的指向就变了。

```
$("#button").on("click", o.f);
```

点击按钮以后，控制台会显示 false。原因是此时 this 不再指向 o 对象，而是指向按钮的 DOM 对象，因为 f 方法是在按钮对象的环境中被调用的。这种细微的差别，很容易在编程中忽视，导致难以察觉的错误。

为了解决这个问题，可以采用下面的一些方法对 this 进行绑定，也就是使得 this 固定指向某个对象，减少不确定性。

## 固定 this 的方法

this 的动态切换，固然为 JavaScript 创造了巨大的灵活性，但也使得编程变得困难和模糊。有时，需要把 this 固定下来，避免出现意想不到的情况。JavaScript 提供了 call、apply、bind 这三个方法，来切换/固定 this 的指向。

### call 方法

函数的 call 方法，可以指定该函数内部 this 的指向（即函数执行时所在的作用域），然后在所指定的作用域中，调用该函数。

```
var o = {};

var f = function (){
  return this;
};

f() === this // true
f.call(o) === o // true
```

上面代码中，在全局环境运行函数 f 时，this 指向全局环境；call 方法可以改变 this 的指向，指定 this 指向对象 o，然后在对象 o 的作用域中运行函数 f。

再看一个例子。

```
var n = 123;
var o = { n : 456 };

function a() {
  console.log(this.n);
}

a.call() // 123
a.call(null) // 123
a.call(undefined) // 123
a.call(window) // 123
a.call(o) // 456
```

上面代码中，a 函数中的 this 关键字，如果指向全局对象，返回结果为 123。如果使用 call 方法将 this 关键字指向 o 对象，返回结果为 456。可以看到，如果 call 方法没有参数，或者参数为 null 或 undefined，则等同于指向全局对象。

call 方法的完整使用格式如下。

```
func.call(thisValue, arg1, arg2, ...)
```

它的第一个参数就是 this 所要指向的那个对象，后面的参数则是函数调用时所需的参数。

```
function add(a,b) {
  return a+b;
}

add.call(this,1,2) // 3
```

上面代码中，call 方法指定函数 add 在当前环境（对象）中运行，并且参数为 1 和 2，因此函数 add 运行后得到 3。

call 方法的一个应用是调用对象的原生方法。

```
var obj = {};
obj.hasOwnProperty('toString') // false

obj.hasOwnProperty = function (){
  return true;
};
obj.hasOwnProperty('toString') // true

Object.prototype.hasOwnProperty.call(obj, 'toString') // false
```

上面代码中，hasOwnProperty 是 obj 对象继承的方法，如果这个方法一旦被覆盖，就不会得到正确结果。call 方法可以解决这个方法，它将 hasOwnProperty 方法的原始定义放到 obj 对象上执行，这样无论 obj 上有没有同名方法，都不会影响结果。

### apply 方法

apply 方法的作用与 call 方法类似，也是改变 this 指向，然后再调用该函数。唯一的区别就是，它接收一个数组作为函数执行时的参数，使用格式如下。

```
func.apply(thisValue, [arg1, arg2, ...])
```

apply 方法的第一个参数也是 this 所要指向的那个对象，如果设为 null 或 undefined，则等同于指定全局对象。第二个参数则是一个数组，该数组的所有成员依次作为参数，传入原函数。原函数的参数，在 call 方法中必须一个个添加，但是在 apply 方法中，必须以数组形式添加。

请看下面的例子。

```
function f(x,y){
  console.log(x+y);
}

f.call(null,1,1) // 2
f.apply(null,[1,1]) // 2
```

上面的 f 函数本来接受两个参数，使用 apply 方法以后，就变成可以接受一个数组作为参数。

利用这一点，可以做一些有趣的应用。

（1）找出数组最大元素

JavaScript 不提供找出数组最大元素的函数。结合使用 apply 方法和 Math.max 方法，就可以返回数组的最大元素。

```
var a = [10, 2, 4, 15, 9];

Math.max.apply(null, a)
// 15
```

（2）将数组的空元素变为 undefined

通过 apply 方法，利用 Array 构造函数将数组的空元素变成 undefined。

```
Array.apply(null, ["a",,"b"])
// [ 'a', undefined, 'b' ]
```

空元素与 undefined 的差别在于，数组的 foreach 方法会跳过空元素，但是不会跳过 undefined。因此，遍历内部元素的时候，会得到不同的结果。

```
var a = ["a",,"b"];

function print(i) {
  console.log(i);
}

a.forEach(print)
// a
// b

Array.apply(null,a).forEach(print)
// a
// undefined
// b
```

（3）转换类似数组的对象

另外，利用数组对象的 slice 方法，可以将一个类似数组的对象（比如 arguments 对象）转为真正的数组。

```
Array.prototype.slice.apply({0:1,length:1})
// [1]

Array.prototype.slice.apply({0:1})
// []

Array.prototype.slice.apply({0:1,length:2})
// [1, undefined]

Array.prototype.slice.apply({length:1})
// [undefined]
```

上面代码的 apply 方法的参数都是对象，但是返回结果都是数组，这就起到了将对象转成数组的目的。从上面代码可以看到，这个方法起作用的前提是，被处理的对象必须有 length 属性，以及相对应的数字键。

（4）绑定回调函数的对象

上一节按钮点击事件的例子，可以改写成

```
var o = new Object();

o.f = function (){
    console.log(this === o);
}

var f = function (){
  o.f.apply(o);
  // 或者 o.f.call(o);
};

$("#button").on("click", f);
```

点击按钮以后，控制台将会显示 true。由于 apply 方法（或者 call 方法）不仅绑定函数执行时所在的对象，还会立即执行函数，因此不得不把绑定语句写在一个函数体内。更简洁的写法是采用下面介绍的 bind 方法。

### bind 方法

bind 方法用于将函数体内的 this 绑定到某个对象，然后返回一个新函数。它的使用格式如下。

```
func.bind(thisValue, arg1, arg2,...)
```

下面是一个例子。

```
var o1 = new Object();
o1.p = 123;
o1.m = function (){
    console.log(this.p);
};

o1.m() // 123 

var o2 = new Object();
o2.p = 456;
o2.m = o1.m;

o2.m() // 456

o2.m = o1.m.bind(o1);
o2.m() // 123
```

上面代码使用 bind 方法将 o1.m 方法绑定到 o1 以后，在 o2 对象上调用 o1.m 的时候，o1.m 函数体内部的 this.p 就不再到 o2 对象去寻找 p 属性的值了。

bind 比 call 方法和 apply 方法更进一步的是，除了绑定 this 以外，还可以绑定原函数的参数。

```
var add = function (x,y) {
  return x*this.m + y*this.n;
}

var obj = {
  m: 2,
  n: 2
};

var newAdd = add.bind(obj, 5);

newAdd(5)
// 20
```

上面代码中，bind 方法除了绑定 this 对象，还绑定了 add 函数的第一个参数，结果 newAdd 函数只要一个参数就能运行了。

如果 bind 方法的第一个参数是 null 或 undefined，等于将 this 绑定到全局对象，函数运行时 this 指向全局对象（在浏览器中为 window）。

```
function add(x,y) { return x+y; }

var plus5 = add.bind(null, 5);

plus5(10) // 15
```

上面代码除了将 add 函数的运行环境绑定为全局对象，还将 add 函数的第一个参数绑定为 5，然后返回一个新函数。以后，每次运行这个新函数，就只需要提供另一个参数就够了。

bind 方法有一些使用注意点。

（1）每一次返回一个新函数

bind 方法每运行一次，就返回一个新函数，这会产生一些问题。比如，监听事件的时候，不能写成下面这样。

```
element.addEventListener('click', o.m.bind(o));
```

上面代码表示，click 事件绑定 bind 方法生成的一个匿名函数。这样会导致无法取消绑定，所以，下面的代码是无效的。

```
element.removeEventListener('click', o.m.bind(o));
```

正确的方法是写成下面这样：

```
var listener = o.m.bind(o);
element.addEventListener('click', listener);
//  ...
element.removeEventListener('click', listener);
```

（2）bind 方法的自定义代码

对于那些不支持 bind 方法的老式浏览器，可以自行定义 bind 方法。

```
if(!('bind' in Function.prototype)){
    Function.prototype.bind = function(){
        var fn = this;
        var context = arguments[0];
        var args = Array.prototype.slice.call(arguments, 1);
        return function(){
            return fn.apply(context, args);
        }
    }
}
```

（3）jQuery 的 proxy 方法

除了用 bind 方法绑定函数运行时所在的对象，还可以使用 jQuery 的$.proxy 方法，它与 bind 方法的作用基本相同。

```
$("#button").on("click", $.proxy(o.f, o));
```

上面代码表示，$.proxy 方法将 o.f 方法绑定到 o 对象。

（4）结合 call 方法使用

利用 bind 方法，可以改写一些 JavaScript 原生方法的使用形式，以数组的 slice 方法为例。

```
[1,2,3].slice(0,1) 
// [1]

// 等同于

Array.prototype.slice.call([1,2,3], 0, 1)
// [1]
```

上面的代码中，数组的 slice 方法从[1, 2, 3]里面，按照指定位置和长度切分出另一个数组。这样做的本质是在[1, 2, 3]上面调用 Array.prototype.slice 方法，因此可以用 call 方法表达这个过程，得到同样的结果。

call 方法实质上是调用 Function.prototype.call 方法，因此上面的表达式可以用 bind 方法改写。

```
var slice = Function.prototype.call.bind(Array.prototype.slice);

slice([1, 2, 3], 0, 1) // [1]
```

可以看到，利用 bind 方法，将[1, 2, 3].slice(0, 1)变成了 slice([1, 2, 3], 0, 1)的形式。这种形式的改变还可以用于其他数组方法。

```
var push = Function.prototype.call.bind(Array.prototype.push);
var pop = Function.prototype.call.bind(Array.prototype.pop);

var a = [1 ,2 ,3];
push(a, 4)
a // [1, 2, 3, 4]

pop(a)
a // [1, 2, 3]
```

如果再进一步，将 Function.prototype.call 方法绑定到 Function.prototype.bind 对象，就意味着 bind 的调用形式也可以被改写。

```
function f(){
    console.log(this.v);
}

var o = { v: 123 };

var bind = Function.prototype.call.bind(Function.prototype.bind);

bind(f,o)() // 123
```

上面代码表示，将 Function.prototype.call 方法绑定 Function.prototype.bind 以后，bind 方法的使用形式从 f.bind(o)，变成了 bind(f, o)。

## 参考链接

*   Jonathan Creamer, [Avoiding the "this" problem in JavaScript](http://tech.pro/tutorial/1192/avoiding-the-this-problem-in-javascript)
*   Erik Kronberg, [Bind, Call and Apply in JavaScript](https://variadic.me/posts/2013-10-22-bind-call-and-apply-in-javascript.html)
*   Axel Rauschmayer, [JavaScript’s this: how it works, where it can trip you up](http://www.2ality.com/2014/05/this.html)