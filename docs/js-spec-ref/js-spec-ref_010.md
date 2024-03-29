# 2.6 函数

*   概述
    *   函数的声明
    *   圆括号运算符和 return 语句
    *   第一等公民
    *   函数名的提升
    *   不能在条件语句中声明函数
    *   函数的属性和方法
        *   name 属性
        *   length 属性
    *   toString()
    *   函数作用域
        *   定义
        *   函数内部的变量提升
        *   函数本身的作用域
    *   参数
        *   概述
        *   参数的省略
        *   默认值
        *   传递方式
        *   同名参数
        *   arguments 对象
    *   函数的其他知识点
        *   闭包
        *   立即调用的函数表达式（IIFE）
    *   eval 命令
    *   参考链接

## 概述

### 函数的声明

（1）function 命令

函数就是使用 function 命令命名的代码区块，便于反复调用。

```js
function print(){
  // ...
}
```

上面的代码命名了一个 print 函数，以后使用 print()这种形式，就可以调用相应的代码。这叫做函数的声明（Function Declaration）。

（2）函数表达式

除了用 function 命令声明函数，还可以采用变量赋值的写法。

```js
var print = function (){
  // ...
};
```

这种写法将一个匿名函数赋值给变量。这时，这个匿名函数又称函数表达式（Function Expression），因为赋值语句的等号右侧只能放表达式。

采用函数表达式声明函数时，function 命令后面不带有函数名。如果加上函数名，该函数名只在函数体内部有效，在函数体外部无效。

```js
var print = function x(){
  console.log(typeof x);
};

x
// ReferenceError: x is not defined

print()
// function
```

上面代码在函数表达式中，加入了函数名 x。这个 x 只在函数体内部可用，指代函数表达式本身，其他地方都不可用。这种写法的用处有两个，一是可以在函数体内部调用自身，二是方便除错（除错工具显示函数调用栈时，将显示函数名，而不再显示这里是一个匿名函数）。因此，需要时，可以采用下面的形式声明函数。

```js
var f = function f(){};
```

需要注意的是，函数的表达式需要在语句的结尾加上分号，表示语句结束。而函数的声明在结尾的大括号后面不用加分号。总的来说，这两种声明函数的方式，差别很细微（参阅后文《变量提升》一节），这里可以近似认为是等价的。

（3）Function 构造函数

还有第三种声明函数的方式：通过 Function 构造函数声明。

```js
var add = new Function("x","y","return (x+y)");
// 相当于定义了如下函数
// function add(x, y) {
//   return (x+y);
// }
```

在上面代码中，Function 对象接受若干个参数，除了最后一个参数是 add 函数的“函数体”，其他参数都是 add 函数的参数。如果只有一个参数，该参数就是函数体。

```js
var foo = new Function('return "hello world"');
// 相当于定义了如下函数
// function foo() {
//   return "hello world";
// }
```

Function 构造函数可以不使用 new 命令，返回结果完全一样。

总的来说，这种声明函数的方式非常不直观，几乎无人使用。

（4）函数的重复声明

如果多次采用 function 命令，重复声明同一个函数，则后面的声明会覆盖前面的声明。

```js
function f(){
  console.log(1);
}

f() // 2

function f(){
  console.log(2);
}

f() // 2
```

上面代码说明，由于存在函数名的提升，前面的声明在任何时候都是无效的，这一点要特别注意。

### 圆括号运算符和 return 语句

调用函数时，要使用圆括号运算符。圆括号之中，可以加入函数的参数。

```js
function add(x,y) {
  return x+y;
}

add(1,1) // 2
```

函数体内部的 return 语句，表示返回。JavaScript 引擎遇到 return 语句，就直接返回 return 后面的那个表达式的值，后面即使还有语句，也不会得到执行。也就是说，return 语句所带的那个表达式，就是函数的返回值。return 语句不是必需的，如果没有的话，该函数就不返回任何值，或者说返回 undefined。

函数可以调用自身，这就是递归（recursion）。下面就是使用递归，计算斐波那契数列的代码。

```js
function fib(num) {
  if (num > 2) {
    return fib(num - 2) + fib(num - 1);
  } else {
    return 1;
  }
}

fib(6)
// 8
```

### 第一等公民

JavaScript 的函数与其他数据类型处于同等地位，可以使用其他数据类型的地方就能使用函数。比如，可以把函数赋值给变量和对象的属性，也可以当作参数传入其他函数，或者作为函数的结果返回。这表示函数与其他数据类型的地方是平等，所以又称函数为第一等公民。

```js
function add(x,y){
  return x+y;
}

// 将函数赋值给一个变量
var operator = add;

// 将函数作为参数和返回值
function a(op){
  return op;
}
a(add)(1,1)
// 2
```

### 函数名的提升

JavaScript 引擎将函数名视同变量名，所以采用 function 命令声明函数时，整个函数会被提升到代码头部。所以，下面的代码不会报错。

```js
f();
function f(){}
```

表面上，上面代码好像在声明之前就调用了函数 f。但是实际上，由于“变量提升”，函数 f 被提升到了代码头部，也就是在调用之前已经声明了。但是，如果采用赋值语句定义函数，JavaScript 就会报错。

```js
f();
var f = function (){};
// TypeError: undefined is not a function
```

上面的代码等同于

```js
var f;
f();
f = function (){};
```

当调用 f 的时候，f 只是被声明，还没有被赋值，等于 undefined，所以会报错。因此，如果同时采用 function 命令和赋值语句声明同一个函数，最后总是采用赋值语句的定义。

```js
var f = function() {
  console.log ('1');
}

function f() {
  console.log('2');
}

f()
// 1
```

### 不能在条件语句中声明函数

根据 ECMAScript 的规范，不得在非函数的代码块中声明函数，最常见的情况就是 if 和 try 语句。

```js
if (foo) {
  function x() { return; }
}

try {
  function x() {return; }
} catch(e) {
  console.log(e);
}
```

上面代码分别在 if 代码块和 try 代码块中声明了两个函数，按照语言规范，这是不合法的。但是，实际情况是各家浏览器往往并不报错，能够运行。

但是由于存在函数名的提升，所以在条件语句中声明函数是无效的，这是非常容易出错的地方。

```js
if (false){
  function f(){}
}

f()
// 不报错
```

由于函数 f 的声明被提升到了 if 语句的前面，导致 if 语句无效，所以上面的代码不会报错。要达到在条件语句中定义函数的目的，只有使用函数表达式。

```js
if (false){
  var f = function (){};
}

f()
// undefined
```

## 函数的属性和方法

### name 属性

name 属性返回紧跟在 function 关键字之后的那个函数名。

```js
function f1() {}
f1.name // 'f1'

var f2 = function () {};
f2.name // ''

var f3 = function myName() {};
f3.name // 'myName'
```

上面代码中，函数的 name 属性总是返回紧跟在 function 关键字之后的那个函数名。对于 f2 来说，返回空字符串，匿名函数的 name 属性总是为空字符串；对于 f3 来说，返回函数表达式的名字（真正的函数名还是 f3，myName 这个名字只在函数体内部可用）。

### length 属性

length 属性返回函数定义中参数的个数。

```js
function f(a,b) {}

f.length
// 2
```

上面代码定义了空函数 f，它的 length 属性就是定义时参数的个数。不管调用时输入了多少个参数，length 属性始终等于 2。

length 属性提供了一种机制，判断定义时和调用时参数的差异，以便实现面向对象编程的”方法重载“（overload）。

## toString()

函数的 toString 方法返回函数的源码。

```js
function f() {
  a();
  b();
  c();
}

f.toString()
// function f() {
//  a();
//  b();
//  c();
// }
```

## 函数作用域

### 定义

作用域（scope）指的是变量存在的范围。Javascript 只有两种作用域：一种是全局作用域，变量在整个程序中一直存在；另一种是函数作用域，变量只在函数内部存在。

在函数外部声明的变量就是全局变量（global variable），它可以在函数内部读取。

```js
var v = 1;

function f(){
   console.log(v);
}

f()
// 1
```

上面的代码表明，函数 f 内部可以读取全局变量 v。

在函数内部定义的变量，外部无法读取，称为“局部变量”（local variable）。

```js
function f(){
   var v = 1;
}

v
// ReferenceError: v is not defined
```

函数内部定义的变量，会在该作用域内覆盖同名全局变量。

```js
var v = 1;

function f(){
   var v = 2;
   console.log(v);
}

f()
// 2

v
// 1
```

### 函数内部的变量提升

与全局作用域一样，函数作用域内部也会产生“变量提升”现象。var 命令声明的变量，不管在什么位置，变量声明都会被提升到函数体的头部。

```js
function foo(x) {
    if (x > 100) {
         var tmp = x - 100;
    }
}
```

上面的代码等同于

```js
function foo(x) {
  var tmp;
  if (x > 100) {
    tmp = x - 100;
  };
}
```

### 函数本身的作用域

函数本身也是一个值，也有自己的作用域。它的作用域绑定其声明时所在的作用域。

```js
var a = 1;
var x = function (){
  console.log(a);
};

function f(){
  var a = 2;
  x();
}

f() // 1
```

上面代码中，函数 x 是在函数 f 的外部声明的，所以它的作用域绑定外层，内部变量 a 不会到函数 f 体内取值，所以输出 1，而不是 2。

很容易犯错的一点是，如果函数 A 调用函数 B，却没考虑到函数 B 不会引用函数 A 的内部变量。

```js
var x = function (){
  console.log(a);
};

function y(f){
  var a = 2;
  f();
}

y(x)
// ReferenceError: a is not defined
```

上面代码将函数 x 作为参数，传入函数 y。但是，函数 x 是在函数 y 体外声明的，作用域绑定外层，因此找不到函数 y 的内部变量 a，导致报错。

## 参数

### 概述

函数运行的时候，有时需要提供外部数据，不同的外部数据会得到不同的结果，这种外部数据就叫参数。

```js
function square(x){
    return x*x;
}

square(2) // 4
square(3) // 9
```

上式的 x 就是 square 函数的参数。每次运行的时候，需要提供这个值，否则得不到结果。

### 参数的省略

参数不是必需的，Javascript 语言允许省略参数。

```js
function f(a,b){
    return a;
}

f(1,2,3) // 1
f(1) // 1
f() // undefined

f.length // 2
```

上面代码的函数 f 定义了两个参数，但是运行时无论提供多少个参数（或者不提供参数），JavaScript 都不会报错。被省略的参数的值就变为 undefined。需要注意的是，函数的 length 属性与实际传入的参数个数无关，只反映定义时的参数个数。

但是，没有办法只省略靠前的参数，而保留靠后的参数。如果一定要省略靠前的参数，只有显式传入 undefined。

```js
function f(a,b){
    return a;
}

f(,1) // error
f(undefined,1) // undefined
```

### 默认值

通过下面的方法，可以为函数的参数设置默认值。

```js
function f(a){
    a = a || 1;
    return a;
}

f('') // 1
f(0) // 1
```

上面代码的||表示“或运算”，即如果 a 有值，则返回 a，否则返回事先设定的默认值（上例为 1）。

这种写法会对 a 进行一次布尔运算，只有为 true 时，才会返回 a。可是，除了 undefined 以外，0、空字符、null 等的布尔值也是 false。也就是说，在上面的函数中，不能让 a 等于 0 或空字符串，否则在明明有参数的情况下，也会返回默认值。

为了避免这个问题，可以采用下面更精确的写法。

```js
function f(a){
    (a !== undefined && a != null)?(a = a):(a = 1);
    return a;
}

f('') // ""
f(0) // 0
```

### 传递方式

JavaScript 的函数参数传递方式是传值传递（passes by value），这意味着，在函数体内修改参数值，不会影响到函数外部。

```js
// 修改原始类型的参数值
var p = 2; 

function f(p){
    p = 3;
}

f(p);
p // 2

// 修改复合类型的参数值
var o = [1,2,3];

function f(o){
    o = [2,3,4];
}

f(o);
o // [1, 2, 3]
```

上面代码分成两段，分别修改原始类型的参数值和复合类型的参数值。两种情况下，函数内部修改参数值，都不会影响到函数外部。

需要十分注意的是，虽然参数本身是传值传递，但是对于复合类型的变量来说，属性值是传址传递（pass by reference），也就是说，属性值是通过地址读取的。所以在函数体内修改复合类型变量的属性值，会影响到函数外部。

```js
// 修改对象的属性值
var o = { p:1 };

function f(obj){
    obj.p = 2;
}

f(o);
o.p // 2

// 修改数组的属性值
var a = [1,2,3];

function f(a){
    a[0]=4;
}

f(a);
a // [4,2,3]
```

上面代码在函数体内，分别修改对象和数组的属性值，结果都影响到了函数外部，这证明复合类型变量的属性值是传址传递。

某些情况下，如果需要对某个变量达到传址传递的效果，可以将它写成全局对象的属性。

```js
var a = 1;

function f(p){
    window[p]=2;
}

f('a');

a // 2
```

上面代码中，变量 a 本来是传值传递，但是写成 window 对象的属性，就达到了传址传递的效果。

### 同名参数

如果有同名的参数，则取最后出现的那个值。

```js
function f(a, a){
    console.log(a);
}

f(1,2)
// 2
```

上面的函数 f 有两个参数，且参数名都是 a。取值的时候，以后面的 a 为准。即使后面的 a 没有值或被省略，也是以其为准。

```js
function f(a, a){
    console.log(a);
}

f(1)
// undefined
```

调用函数 f 的时候，没有提供第二个参数，a 的取值就变成了 undefined。这时，如果要获得第一个 a 的值，可以使用 arguments 对象。

```js
function f(a, a){
    console.log(arguments[0]);
}

f(1)
// 1
```

### arguments 对象

（1）定义

由于 JavaScript 允许函数有不定数目的参数，所以我们需要一种机制，可以在函数体内部读取所有参数。这就是 arguments 对象的由来。

arguments 对象包含了函数运行时的所有参数，arguments[0]就是第一个参数，arguments[1]就是第二个参数，依次类推。这个对象只有在函数体内部，才可以使用。

```js
var f = function(one) {
  console.log(arguments[0]);
  console.log(arguments[1]);
  console.log(arguments[2]);
}

f(1, 2, 3)
// 1
// 2
// 3
```

arguments 对象除了可以读取参数，还可以为参数赋值（严格模式不允许这种用法）。

```js
var f = function(a,b) {
  arguments[0] = 3;
  arguments[1] = 2;
  return a+b;
}

f(1, 1)
// 5
```

可以通过 arguments 对象的 length 属性，判断函数调用时到底带几个参数。

```js
function f(){
    return arguments.length;
}

f(1,2,3) // 3
f(1) // 1
f() // 0
```

（2）与数组的关系

需要注意的是，虽然 arguments 很像数组，但它是一个对象。某些用于数组的方法（比如 slice 和 forEach 方法），不能在 arguments 对象上使用。

但是，有时 arguments 可以像数组一样，用在某些只用于数组的方法。比如，用在 apply 方法中，或使用 concat 方法完成数组合并。

```js
// 用于 apply 方法
myfunction.apply(obj, arguments).

// 使用与另一个数组合并
Array.prototype.concat.apply([1,2,3], arguments)
```

要让 arguments 对象使用数组方法，真正的解决方法是将 arguments 转为真正的数组。下面是两种常用的转换方法：slice 方法和逐一填入新数组。

```js
var args = Array.prototype.slice.call(arguments);

// or 

var args = [];
for(var i = 0; i < arguments.length; i++) {
      args.push(arguments[i]);
}
```

（3）callee 属性

arguments 对象带有一个 callee 属性，返回它所对应的原函数。

```js
var f = function(one) {
  console.log(arguments.callee === f);
}

f()
// true
```

## 函数的其他知识点

### 闭包

闭包（closure）就是定义在函数体内部的函数。更理论性的表达是，闭包是函数与其生成时所在的作用域对象（scope object）的一种结合。

```js
function f() {
    var c = function (){}; 
}
```

上面的代码中，c 是定义在函数 f 内部的函数，就是闭包。

闭包的特点在于，在函数外部可以读取函数的内部变量。

```js
function f() {
    var v = 1;

    var c = function (){
        return v;
    };

    return c;
}

var o = f();

o();
// 1
```

上面代码表示，原先在函数 f 外部，我们是没有办法读取内部变量 v 的。但是，借助闭包 c，可以读到这个变量。

闭包不仅可以读取函数内部变量，还可以使得内部变量记住上一次调用时的运算结果。

```js
function createIncrementor(start) {
        return function () { 
            return start++;
        }
}

var inc = createIncrementor(5);

inc() // 5
inc() // 6
inc() // 7
```

上面代码表示，函数内部的 start 变量，每一次调用时都是在上一次调用时的值的基础上进行计算的。

### 立即调用的函数表达式（IIFE）

在 Javascript 中，一对圆括号“()”是一种运算符，跟在函数名之后，表示调用该函数。比如，print()就表示调用 print 函数。

有时，我们需要在定义函数之后，立即调用该函数。这时，你不能在函数的定义之后加上圆括号，这会产生语法错误。

```js
function(){ /* code */ }();
// SyntaxError: Unexpected token (
```

产生这个错误的原因是，Javascript 引擎看到 function 关键字之后，认为后面跟的是函数定义语句，不应该以圆括号结尾。

解决方法就是让引擎知道，圆括号前面的部分不是函数定义语句，而是一个表达式，可以对此进行运算。你可以这样写：

```js
(function(){ /* code */ }()); 

// 或者

(function(){ /* code */ })();
```

这两种写法都是以圆括号开头，引擎就会认为后面跟的是一个表示式，而不是函数定义，所以就避免了错误。这就叫做“立即调用的函数表达式”（Immediately-Invoked Function Expression），简称 IIFE。

> 注意，上面的两种写法的结尾，都必须加上分号。

推而广之，任何让解释器以表达式来处理函数定义的方法，都能产生同样的效果，比如下面三种写法。

```js
var i = function(){ return 10; }();

true && function(){ /* code */ }();

0, function(){ /* code */ }();
```

甚至像这样写

```js
!function(){ /* code */ }();

~function(){ /* code */ }();

-function(){ /* code */ }();

+function(){ /* code */ }();
```

new 关键字也能达到这个效果。

```js
new function(){ /* code */ }

new function(){ /* code */ }() // 只有传递参数时，才需要最后那个圆括号。
```

通常情况下，只对匿名函数使用这种“立即执行的函数表达式”。它的目的有两个：一是不必为函数命名，避免了污染全局变量；二是 IIFE 内部形成了一个单独的作用域，可以封装一些外部无法读取的私有变量。

```js
// 写法一
var tmp = newData;
processData(tmp);
storeData(tmp);

// 写法二
(function (){
  var tmp = newData;
  processData(tmp);
  storeData(tmp);
}());
```

上面代码中，写法二比写法一更好，因为完全避免了污染全局变量。

## eval 命令

eval 命令的作用是，将字符串当作语句执行。

```js
eval('var a = 1;');

a // 1
```

上面代码将字符串当作语句运行，生成了变量 a。

放在 eval 中的字符串，应该有独自存在的意义，不能用来与 eval 以外的命令配合使用。举例来说，下面的代码将会报错。

```js
eval('return;');
```

由于 eval 没有自己的作用域，都在当前作用域内执行，因此可能会修改其他外部变量的值，造成安全问题。

```js
var a = 1;
eval('a = 2');

a // 2
```

上面代码中，eval 命令修改了外部变量 a 的值。由于这个原因，所以 eval 有安全风险，无法做到作用域隔离，最好不要使用。此外，eval 的命令字符串不会得到 JavaScript 引擎的优化，运行速度较慢，也是另一个不应该使用它的理由。通常情况下，eval 最常见的场合是解析 JSON 数据字符串，正确的做法是这时应该使用浏览器提供的 JSON.parse 方法。

ECMAScript 5 将 eval 的使用分成两种情况，像上面这样的调用，就叫做“直接使用”，这种情况下 eval 的作用域就是当前作用域（即全局作用域或函数作用域）。另一种情况是，eval 不是直接调用，而是“间接调用”，此时 eval 的作用域总是全局作用域。

```js
var a = 1;

function f(){
    var a = 2;
    var e = eval;
    e('console.log(a)');
}

f() // 1
```

上面代码中，eval 是间接调用，所以即使它是在函数中，它的作用域还是全局作用域，因此输出的 a 为全局变量。

eval 的间接调用的形式五花八门，只要不是直接调用，几乎都属于间接调用。

```js
eval.call(null, '...')
window.eval('...')
(1, eval)('...')
(eval, eval)('...')
(1 ? eval : 0)('...')
(__ = eval)('...')
var e = eval; e('...')
(function(e) { e('...') })(eval)
(function(e) { return e })(eval)('...')
(function() { arguments0 })(eval)
this.eval('...')
this'eval'
[eval]0
eval.call(this, '...')
eval('eval')('...')
```

上面这些形式都是 eval 的间接调用，因此它们的作用域都是全局作用域。

与 eval 作用类似的还有 Function 构造函数。利用它生成一个函数，然后调用该函数，也能将字符串当作命令执行。

```js
var jsonp = 'foo({"id":42})';

var f = new Function( "foo", jsonp );
// 相当于定义了如下函数
// function f(foo) {
//   foo({"id":42});
// }

f(function(json){
  console.log( json.id ); // 42
})
```

上面代码中，jsonp 是一个字符串，Function 构造函数将这个字符串，变成了函数体。调用该函数的时候，jsonp 就会执行。这种写法的实质是将代码放到函数作用域执行，避免对全局作用域造成影响。

## 参考链接

*   Ben Alman, [Immediately-Invoked Function Expression (IIFE)](http://benalman.com/news/2010/11/immediately-invoked-function-expression/)
*   Mark Daggett, [Functions Explained](http://markdaggett.com/blog/2013/02/15/functions-explained/)
*   Juriy Zaytsev, [Named function expressions demystified](http://kangax.github.com/nfe/)
*   Marco Rogers polotek, [What is the arguments object?](http://docs.nodejitsu.com/articles/javascript-conventions/what-is-the-arguments-object)
*   Juriy Zaytsev, [Global eval. What are the options?](http://perfectionkills.com/global-eval-what-are-the-options/)
*   Axel Rauschmayer, [Evaluating JavaScript code via eval() and new Function()](http://www.2ality.com/2014/01/eval.html)