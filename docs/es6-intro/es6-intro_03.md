# let 和 const 命令

## let 命令

### 基本用法

ES6 新增了 let 命令，用来声明变量。它的用法类似于 var，但是所声明的变量，只在 let 命令所在的代码块内有效。

```js
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1

```

上面代码在代码块之中，分别用 let 和 var 声明了两个变量。然后在代码块之外调用这两个变量，结果 let 声明的变量报错，var 声明的变量返回了正确的值。这表明，let 声明的变量只在它所在的代码块有效。

for 循环的计数器，就很合适使用 let 命令。

```js
for(let i = 0; i < arr.length; i++){}

console.log(i)
//ReferenceError: i is not defined

```

上面代码的计数器 i，只在 for 循环体内有效。

下面的代码如果使用 var，最后输出的是 10。

```js
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10

```

如果使用 let，声明的变量仅在块级作用域内有效，最后输出的是 6。

```js
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6

```

### 不存在变量提升

let 不像 var 那样，会发生“变量提升”现象。

```js
function do_something() {
  console.log(foo); // ReferenceError
  let foo = 2;
}

```

上面代码在声明 foo 之前，就使用这个变量，结果会抛出一个错误。

这也意味着 typeof 不再是一个百分之百安全的操作。

```js
if (1) {
  typeof x; // ReferenceError
  let x;
}

```

上面代码中，由于块级作用域内 typeof 运行时，x 还没有值，所以会抛出一个 ReferenceError。

只要块级作用域内存在 let 命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。

```js
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}

```

上面代码中，存在全局变量 tmp，但是块级作用域内 let 又声明了一个局部变量 tmp，导致后者绑定这个块级作用域，所以在 let 声明变量前，对 tmp 赋值会报错。

ES6 明确规定，如果区块中存在 let 和 const 命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些命令，就会报错。

总之，在代码块内，使用 let 命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。

```js
if (true) {
  // TDZ 开始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ 结束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}

```

上面代码中，在 let 命令声明变量 tmp 之前，都属于变量 tmp 的“死区”。

有些“死区”比较隐蔽，不太容易发现。

```js
function bar(x=y, y=2) {
  return [x, y];
}

bar(); // 报错

```

上面代码中，调用 bar 函数之所以报错，是因为参数 x 默认值等于另一个参数 y，而此时 y 还没有声明，属于”死区“。

需要注意的是，函数的作用域是其声明时所在的作用域。如果函数 A 的参数是函数 B，那么函数 B 的作用域不是函数 A。

```js
let foo = 'outer';

function bar(func = x => foo) {
  let foo = 'inner';
  console.log(func()); // outer
}

bar();

```

上面代码中，函数 bar 的参数 func，默认是一个匿名函数，返回值为变量 foo。这个匿名函数的作用域就不是 bar。这个匿名函数声明时，是处在外层作用域，所以内部的 foo 指向函数体外的声明，输出 outer。它实际上等同于下面的代码。

```js
let foo = 'outer';
let f = x => foo;

function bar(func = f) {
  let foo = 'inner';
  console.log(func()); // outer
}

bar();

```

### 不允许重复声明

let 不允许在相同作用域内，重复声明同一个变量。

```js
// 报错
function () {
  let a = 10;
  var a = 1;
}

// 报错
function () {
  let a = 10;
  let a = 1;
}

```

因此，不能在函数内部重新声明参数。

```js
function func(arg) {
  let arg; // 报错
}

function func(arg) {
  {
    let arg; // 不报错
  }
}

```

## 块级作用域

let 实际上为 JavaScript 新增了块级作用域。

```js
function f1() {
  let n = 5;
  if (true) {
      let n = 10;
  }
  console.log(n); // 5
}

```

上面的函数有两个代码块，都声明了变量 n，运行后输出 5。这表示外层代码块不受内层代码块的影响。如果使用 var 定义变量 n，最后输出的值就是 10。

块级作用域的出现，实际上使得获得广泛应用的立即执行匿名函数（IIFE）不再必要了。

```js
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

另外，ES6 也规定，函数本身的作用域，在其所在的块级作用域之内。

```js
function f() { console.log('I am outside!'); }
(function () {
  if(false) {
    // 重复声明一次函数 f
    function f() { console.log('I am inside!'); }
  }

  f();
}());

```

上面代码在 ES5 中运行，会得到“I am inside!”，但是在 ES6 中运行，会得到“I am outside!”。这是因为 ES5 存在函数提升，不管会不会进入 if 代码块，函数声明都会提升到当前作用域的顶部，得到执行；而 ES6 支持块级作用域，不管会不会进入 if 代码块，其内部声明的函数皆不会影响到作用域的外部。

需要注意的是，如果在严格模式下，函数只能在顶层作用域和函数内声明，其他情况（比如 if 代码块、循环代码块）的声明都会报错。

## const 命令

const 也用来声明变量，但是声明的是常量。一旦声明，常量的值就不能改变。

```js
const PI = 3.1415;
PI // 3.1415

PI = 3;
PI // 3.1415

const PI = 3.1;
PI // 3.1415

```

上面代码表明改变常量的值是不起作用的。需要注意的是，对常量重新赋值不会报错，只会默默地失败。

const 的作用域与 let 命令相同：只在声明所在的块级作用域内有效。

```js
if (true) {
  const MAX = 5;
}

// 常量 MAX 在此处不可得

```

const 命令也不存在提升，只能在声明的位置后面使用。

```js
if (true) {
  console.log(MAX); // ReferenceError
  const MAX = 5;
}

```

上面代码在常量 MAX 声明之前就调用，结果报错。

const 声明的常量，也与 let 一样不可重复声明。

```js
var message = "Hello!";
let age = 25;

// 以下两行都会报错
const message = "Goodbye!";
const age = 30;

```

由于 const 命令只是指向变量所在的地址，所以将一个对象声明为常量必须非常小心。

```js
const foo = {};
foo.prop = 123;

foo.prop
// 123

foo = {} // 不起作用

```

上面代码中，常量 foo 储存的是一个地址，这个地址指向一个对象。不可变的只是这个地址，即不能把 foo 指向另一个地址，但对象本身是可变的，所以依然可以为其添加新属性。

下面是另一个例子。

```js
const a = [];
a.push("Hello"); // 可执行
a.length = 0;    // 可执行
a = ["Dave"];    // 报错

```

上面代码中，常量 a 是一个数组，这个数组本身是可写的，但是如果将另一个数组赋值给 a，就会报错。

如果真的想将对象冻结，应该使用 Object.freeze 方法。

```js
const foo = Object.freeze({});
foo.prop = 123; // 不起作用

```

上面代码中，常量 foo 指向一个冻结的对象，所以添加新属性不起作用。

除了将对象本身冻结，对象的属性也应该冻结。下面是一个将对象彻底冻结的函数。

```js
var constantize = (obj) => {
  Object.freeze(obj);
  Object.keys(obj).forEach( (key, value) => {
    if ( typeof obj[key] === 'object' ) {
      constantize( obj[key] );
    }
  });
};

```

## 跨模块常量

上面说过，const 声明的常量只在当前代码块有效。如果想设置跨模块的常量，可以采用下面的写法。

```js
// constants.js 模块
export const A = 1;
export const B = 3;
export const C = 4;

// test1.js 模块
import * as constants from './constants';
console.log(constants.A); // 1
console.log(constants.B); // 3

// test2.js 模块
import {A, B} from './constants';
console.log(A); // 1
console.log(B); // 3

```

## 全局对象的属性

全局对象是最顶层的对象，在浏览器环境指的是 window 对象，在 Node.js 指的是 global 对象。在 JavaScript 语言中，所有全局变量都是全局对象的属性。

ES6 规定，var 命令和 function 命令声明的全局变量，属于全局对象的属性；let 命令、const 命令、class 命令声明的全局变量，不属于全局对象的属性。

```js
var a = 1;
// 如果在 node 环境，可以写成 global.a
// 或者采用通用方法，写成 this.a
window.a // 1

let b = 1;
window.b // undefined

```

上面代码中，全局变量 a 由 var 命令声明，所以它是全局对象的属性；全局变量 b 由 let 命令声明，所以它不是全局对象的属性，返回 undefined。