# 2.4 对象

*   概述
    *   生成方法
    *   读写属性
    *   属性的删除
    *   对象的引用
    *   in 运算符
    *   for...in 循环
    *   类似数组的对象
    *   with 语句
    *   参考链接

## 概述

（1）定义

对象（object）是 JavaScript 的核心概念，也是最重要的数据类型。JavaScript 的所有数据都可以被视为对象。

简单说，所谓对象，就是一种无序的数据集合，由若干个“键值对”（key-value）构成。

```js
var o = {
  p: "Hello World"
};
```

上面代码中，大括号就定义了一个对象，它被赋值给变量 o。这个对象内部包含一个键值对（又称为“成员”），p 是“键名”（成员的名称），字符串“Hello World”是“键值”（成员的值）。键名与键值之间用冒号分隔。如果对象内部包含多个键值对，每个键值对之间用逗号分隔。

（2）键名

键名加不加引号都可以，上面的代码也可以写成下面这样。

```js
var o = {
  "p": "Hello World"
};
```

但是，如果键名不符合标识名的条件（比如包含数字、字母、下划线以外的字符，或者第一个字符为数字），也不是正整数，则必须加上引号。

```js
var o = {
  "1p": "Hello World",
  "h w": "Hello World",
  "p+q": "Hello World"
};
```

上面对象的三个键名，都不符合标识名的条件，所以必须加上引号。由于对象是键值对的封装，所以可以把对象看成是一个容器，里面封装了多个成员，上面的对象就包含了三个成员。

（3）属性

对象的每一个“键名”又称为“属性”（property），它的“键值”可以是任何数据类型。如果一个属性的值为函数，通常把这个属性称为“方法”，它可以像函数那样调用。

```js
var o = {
  p: function(x) {return 2*x;}
};

o.p(1)
// 2
```

上面的对象就有一个方法 p，它就是一个函数。

对象的属性之间用逗号分隔，ECMAScript 5 规定最后一个属性后面可以加逗号（trailing comma），也可以不加。

```js
var o = {
  p: 123,
  m: function () { ... },
}
```

上面的代码中 m 属性后面的那个逗号，有或没有都不算错。但是，ECMAScript 3 不允许添加逗号，所以如果要兼容老式浏览器（比如 IE 8），那就不能加这个逗号。

### 生成方法

对象的生成方法，通常有三种方法。除了像上面那样直接使用大括号生成（{}），还可以用 new 命令生成一个 Object 对象的实例，或者使用 Object.create 方法生成。

```js
var o1 = {};
var o2 = new Object();
var o3 = Object.create(null);
```

上面三行语句是等价的。一般来说，第一种采用大括号的写法比较简洁，第二种采用构造函数的写法清晰地表示了意图，第三种写法一般用在需要对象继承的场合。关于第二种写法，详见《标准库》一章的 Object 对象一节，第三种写法详见《面向对象编程》一章。

### 读写属性

（1）读取属性

读取对象的属性，有两种方法，一种是使用点运算符，还有一种是使用方括号运算符。

```js
var o = {
  p: "Hello World"
};

o.p // "Hello World"
o["p"] // "Hello World"
```

上面代码分别采用点运算符和方括号运算符，读取属性 p。

请注意，如果使用方括号运算符，键名必须放在引号里面，否则会被当作变量处理。但是，数字键可以不加引号，因为会被当作字符串处理。

```js
var o = {
  0.7: "Hello World"
};

o.["0.7"] // "Hello World"
o[0.7] // "Hello World"
```

方括号运算符内部可以使用表达式。

```js
o['hello' + ' world']
o[3+3]
```

（2）检查变量是否声明

如果读取一个不存在的键，会返回 undefined，而不是报错。可以利用这一点，来检查一个变量是否被声明。

```js
// 检查 a 变量是否被声明

if(a) {...} // 报错

if(window.a) {...} // 不报错
if(window['a']) {...} // 不报错
```

上面的后二种写法之所以不报错，是因为在浏览器环境，所有全局变量都是 window 对象的属性。window.a 的含义就是读取 window 对象的 a 属性，如果该属性不存在，就返回 undefined，并不会报错。

需要注意的是，后二种写法有漏洞，如果 a 属性是一个空字符串（或其他对应的布尔值为 false 的情况），则无法起到检查变量是否声明的作用。正确的写法是使用 in 运算符。

```js
if('a' in window) {
  ...
}
```

（3）写入属性

点运算符和方括号运算符，不仅可以用来读取值，还可以用来赋值。

```js
o.p = "abc";
o["p"] = "abc";
```

上面代码分别使用点运算符和方括号运算符，对属性 p 赋值。

JavaScript 允许属性的“后绑定”，也就是说，你可以在任意时刻新增属性，没必要在定义对象的时候，就定义好属性。

```js
var o = { p:1 };

// 等价于

var o = {};
o.p = 1;
```

（4）查看所有属性

查看一个对象本身的所有属性，可以使用 Object.keys 方法。

```js
var o = {
  key1: 1,
  key2: 2
};

Object.keys(o);
// ["key1", "key2"]
```

### 属性的删除

删除一个属性，需要使用 delete 命令。

```js
var o = { p:1 };

Object.keys(o) // ["p"]

delete o.p // true

o.p // undefined

Object.keys(o) // []
```

上面代码表示，一旦使用 delete 命令删除某个属性，再读取该属性就会返回 undefined，而且 Object.keys 方法返回的该对象的所有属性中，也将不再包括该属性。

麻烦的是，如果删除一个不存在的属性，delete 不报错，而且返回 true。

```js
var o = {};

delete o.p // true
```

上面代码表示，delete 命令只能用来保证某个属性的值为 undefined，而无法保证该属性是否真的存在。

只有一种情况，delete 命令会返回 false，那就是该属性存在，且不得删除。

```js
var o = Object.defineProperty({}, "p", {
        value: 123,
        configurable: false
});

o.p // 123
delete o.p // false
```

上面代码之中，o 对象的 p 属性是不能删除的，所以 delete 命令返回 false（关于 Object.defineProperty 方法的介绍，请看《标准库》一章的 Object 对象章节）。

另外，需要注意的是，delete 命令只能删除对象本身的属性，不能删除继承的属性（关于继承参见《面向对象编程》一节）。delete 命令也不能删除 var 命令声明的变量，只能用来删除属性。

### 对象的引用

如果不同的变量名指向同一个对象，那么它们都是这个对象的引用，也就是说指向同一个内存地址。修改其中一个变量，会影响到其他所有变量。

```js
var o1 = {};
var o2 = o1;

o1.a = 1;
o2.a // 1

o2.b = 2;
o1.b // 2
```

上面代码之中，o1 和 o2 指向同一个对象，因此为其中任何一个变量添加属性，另一个变量都可以读写该属性。

但是，这种引用只局限于对象，对于原始类型的数据则是传值引用，也就是说，都是值的拷贝。

```js
var x = 1;
var y = x;

x = 2;
y // 1
```

上面的代码中，当 x 的值发生变化后，y 的值并不变，这就表示 y 和 x 并不是指向同一个内存地址。

### in 运算符

in 运算符用于检查对象是否包含某个属性（注意，检查的是键名，不是键值），如果包含就返回 true，否则返回 false。

```js
var o = { p: 1 };
'p' in o // true
```

该运算符对数组也适用。

```js
var a = ["hello", "world"];

0 in a // true
1 in a // true
2 in a // false

'0' in a // true
'1' in a // true
'2' in a // false
```

上面代码表示，数字键 0 和 1 都在数组之中。由于数组是一种特殊对象，而对象的键名都是字符串，所以字符串的”0“和”1“，也是数组的键名。

在 JavaScript 语言中，所有全局变量都是顶层对象（浏览器的顶层对象就是 window 对象）的属性，因此可以用 in 运算符判断，一个全局变量是否存在。

```js
// 假设变量 x 未定义

// 写法一：报错
if (x){ return 1; }

// 写法二：不正确
if (window.x){ return 1; }

// 写法三：正确
if ('x' in window) { return 1; }
```

上面三种写法之中，如果 x 不存在，第一种写法会报错；如果 x 的值对应布尔值 false（比如 x 等于空字符串），第二种写法无法得到正确结果；只有第三种写法，才能正确判断变量 x 是否存在。

in 运算符的一个问题是，它不能识别对象继承的属性。

```js
var o = new Object();
o.hasOwnProperty('toString') // false

'toString' in o // true
```

上面代码中，toString 方法不是对象 o 自身的属性，而是继承的属性，hasOwnProperty 方法可以说明这一点。但是，in 运算符不能识别，对继承的属性也返回 true。

### for...in 循环

for...in 循环用来遍历一个对象的全部属性。

```js
var o = {a:1, b:2, c:3};

for (i in o){
  console.log(o[i]);
}
// 1
// 2
// 3
```

注意，for...in 循环遍历的是对象所有可 enumberable 的属性，其中不仅包括定义在对象本身的属性，还包括对象继承的属性。

```js
// name 是 Person 本身的属性
function Person(name) {
  this.name = name;
}

// describe 是 Person.prototype 的属性
Person.prototype.describe = function () {
  return 'Name: '+this.name;
};

var person = new Person('Jane');

// for...in 循环会遍历实例自身的属性（name），
// 以及继承的属性（describe）
for (var key in person) {
  console.log(key);
}
// name
// describe
```

上面代码中，name 是对象本身的属性，describe 是对象继承的属性，for-in 循环的遍历会包括这两者。

如果只想遍历对象本身的属性，可以使用 hasOwnProperty 方法，在循环内部做一个判断。

```js
for (var key in person) {
    if (person.hasOwnProperty(key)) {
        console.log(key);
    }
}
// name
```

为了避免这一点，可以新建一个继承 null 的对象。由于 null 没有任何属性，所以新对象也就不会有继承的属性了。

## 类似数组的对象

在 JavaScript 中，有些对象被称为“类似数组的对象”（array-like object）。意思是，它们看上去很像数组，可以使用 length 属性，但是它们并不是数组，所以无法使用一些数组的方法。

下面就是一个类似数组的对象。

```js
var a = {
    0:'a',
    1:'b',
    2:'c',
    length:3
};

a[0] // 'a'
a[2] // 'c'
a.length // 3
```

上面代码的变量 a 是一个对象，但是看上去跟数组很像。所以只要有数字键和 length 属性，就是一个类似数组的对象。当然，变量 a 无法使用数组特有的一些方法，比如 pop 和 push 方法。而且，length 属性不是动态值，不会随着成员的变化而变化。

```js
a[3] = 'd';

a.length // 3
```

上面代码为对象 a 添加了一个数字键，但是 length 属性没变。这就说明了 a 不是数组。

典型的类似数组的对象是函数的 arguments 对象，以及大多数 DOM 元素集，还有字符串。

```js
// arguments 对象
function args() { return arguments }
var arrayLike = args('a', 'b');

arrayLike[0] // 'a'
arrayLike.length // 2
arrayLike instanceof Array // false

// DOM 元素集
var elts = document.getElementsByTagName('h3');
elts.length // 3
elts instanceof Array // false

// 字符串
'abc'[1] // 'b'
'abc'.length // 3
'abc' instanceof Array // false
```

通过函数的 call 方法，可以用 slice 方法将类似数组的对象，变成真正的数组。

```js
var arr = Array.prototype.slice.call(arguments);
```

遍历类似数组的对象，可以采用 for 循环，也可以采用数组的 forEach 方法。

```js
// for 循环
function logArgs() {
    for (var i=0; i<arguments.length; i++) {
        console.log(i+'. '+arguments[i]);
    }
}

// forEach 方法
function logArgs() {
    Array.prototype.forEach.call(arguments, function (elem, i) {
        console.log(i+'. '+elem);
    });
}
```

## with 语句

with 语句的格式如下：

```js
with (object)
  statement
```

它的作用是操作同一个对象的多个属性时，提供一些书写的方便。

```js
// 例一
with (o) {
  p1 = 1;
  p2 = 2;
}

// 等同于

o.p1 = 1;
o.p2 = 2;

// 例二
with (document.links[0]){
  console.log(href);
  console.log(title);
  console.log(style);
}

// 等同于

console.log(document.links[0].href);
console.log(document.links[0].title);
console.log(document.links[0].style);
```

注意，with 区块内部的变量，必须是当前对象已经存在的属性，否则会创造一个当前作用域的全局变量。这是因为 with 区块没有改变作用域，它的内部依然是当前作用域。

```js
var o = {};

with (o){
  x = "abc";
}

o.x
// undefined

x
// "abc"
```

上面代码中，对象 o 没有属性 x，所以 with 区块内部对 x 的操作，等于创造了一个全局变量 x。正确的写法应该是，先定义对象 o 的属性 x，然后在 with 区块内操作它。

```js
var o = {};

o.x = 1;

with (o){
  x = 2;
}

o.x
// 2
```

这是 with 语句的一个很大的弊病，就是绑定对象不明确。

```js
with (o) {
  console.log(x);
}
```

单纯从上面的代码块，根本无法判断 x 到底是全局变量，还是 o 对象的一个属性。这非常不利于代码的除错和模块化，编译器也无法对这段代码进行优化，只能留到运行时判断，这就拖慢了运行速度。因此，建议不要使用 with 语句，可以考虑用一个临时变量代替 with。

```js
with(o1.o2.o3) {
  console.log(p1 + p2);
}

// 可以写成

var temp = o1.o2.o3;
console.log(temp.p1 + temp.p2);
```

with 语句少数有用场合之一，就是替换模板变量。

```js
var str = 'Hello <%= name %>!';
```

上面代码是一个模板字符串，为了替换其中的变量 name，可以先将其分解成三部分`'Hello ', name, '!'`，然后进行模板变量替换。

```js
var o = {
  name: 'Alice'
};

var p = [];
var tmpl = '';

with(o){
  p.push('Hello ', name, '!');
};

p.join('') // "Hello Alice!"
```

上面代码中，with 区块内部，模板变量 name 可以被对象 o 的属性替换，而 p 依然是全局变量。事实上，这就是很多模板引擎的实现原理。

## 参考链接

*   Dr. Axel Rauschmayer，[Object properties in JavaScript](http://www.2ality.com/2012/10/javascript-properties.html)
*   Lakshan Perera, [Revisiting JavaScript Objects](http://www.laktek.com/2012/12/29/revisiting-javascript-objects/)
*   Angus Croll, [The Secret Life of JavaScript Primitives](http://javascriptweblog.wordpress.com/2010/09/27/the-secret-life-of-javascript-primitives/)i
*   Dr. Axel Rauschmayer, [JavaScript’s with statement and why it’s deprecated](http://www.2ality.com/2011/06/with-statement.html)