# 2.1 语法概述

*   基本句法和变量
    *   语句
    *   变量
    *   变量提升
    *   标识符
    *   注释
    *   区块
    *   条件语句
    *   循环语句
    *   数据类型
        *   概述
        *   typeof 运算符
        *   null 和 undefined
        *   布尔值
    *   结尾的分号
        *   不使用分号结尾的语句
        *   分号的自动添加
    *   参考链接

## 基本句法和变量

### 语句

JavaScript 程序的执行单位为行（line），也就是一行一行地执行。一般情况下，每一行就是一个语句。

语句（statement）是为了完成某种任务而进行的操作，比如下面就是一行赋值语句：

```js
var a = 1 + 3;
```

这条语句先用 var 命令，声明了变量 a，然后将`1 + 3`的运算结果赋值给变量 a。

`1 + 3`叫做表达式（expression），指一个为了得到返回值的计算式。语句和表达式的区别在于，前者主要为了进行某种操作，一般情况下不需要返回值；后者则是为了得到返回值，一定会返回一个值。

凡是 JavaScript 语言中预期为值的地方，都可以使用表达式。比如，赋值语句的等号右边，预期是一个值，因此可以放置各种表达式。一条语句可以包含多个表达式。

语句以分号结尾，一个分号就表示一个语句结束。多个语句可以写在一行内。

```js
var a = 1 + 3 ; var b = "abc";
```

分号前面可以没有任何内容，JavaScript 引擎将其视为空语句。

```js
;;;
```

上面的代码就表示 3 个空语句。（关于分号的更多介绍，请看后文《结尾的分号》一节。）

表达式不需要分号结尾。一旦在表达式后面添加分号，则 JavaScript 引擎就将表达式视为语句，这样会产生一些没有任何意义的语句。

```js
1 + 3;

"abc";
```

上面两行语句有返回值，但是没有任何意义，因为只是返回一个单纯的值，没有任何其他操作。

### 变量

变量是对“值”的引用，使用变量等同于引用一个值。每一个变量都有一个变量名。

```js
var a = 1;
```

上面的代码先声明变量 a，然后在变量 a 与数值 1 之间建立引用关系，也称将数值 1“赋值”给变量 a。以后，引用变量 a 就会得到数值 1。最前面的 var，是变量声明命令。它表示通知解释引擎，要创建一个变量 a。

变量的声明和赋值，是分开的两个步骤，上面的代码将它们合在了一起，实际的步骤是下面这样。

```js
var a;

a = 1;
```

如果只是声明变量而没有赋值，则该变量的值为 undefined。

```js
var a;
a
// undefined
```

JavaScript 允许省略 var，直接对未声明的变量赋值。也就是说，var a = 1 与 a = 1，这两条语句的效果相同。但是由于这样的做法很容易不知不觉地创建全局变量（尤其是在函数内部），所以建议总是使用 var 命令声明变量。

> 严格地说，var a = 1 与 a = 1，这两条语句的效果不完全一样，主要体现在 delete 命令无法删除前者。不过，绝大多数情况下，这种差异是可以忽略的。

如果一个变量没有声明就直接使用，JavaScript 会报错，告诉你变量未定义。

```js
x
// ReferenceError: x is not defined
```

上面代码直接使用变量 x，系统就报错，告诉你变量 x 没有声明。

可以在同一条 var 命令中声明多个变量。

```js
var a,b;
```

JavaScirpt 是一种动态类型语言，也就是说，变量的类型没有限制，可以赋予各种类型的值。

```js
var a = 1;

a = "hello";
```

上面代码中，变量 a 起先被赋值为一个数值，后来又被重新赋值为一个字符串。第二次赋值的时候，因为变量 a 已经存在，所以不需要使用 var 命令。如果用了，就等于重新声明一个变量 a，会覆盖掉前面的同名变量。

### 变量提升

JavaScript 引擎的工作方式是，先解析代码，获取所有被声明的变量，然后再一行一行地运行。这造成的结果，就是所有的变量的声明语句，都会被提升到代码的头部，这就叫做变量提升（hoisting）。

```js
console.log(a);
var a = 1;
```

上面代码首先使用 console.log 方法，在控制台（console）显示变量 a 的值。这时变量 a 还没有声明和赋值，所以这是一种错误的做法，但是实际上不会报错。因为存在变量提升，真正运行的是下面的代码：

```js
var a;
console.log(a);
a = 1;
```

最后的结果是显示 undefined，表示变量 a 已声明，但还未赋值。

请注意，变量提升只对 var 命令声明的变量有效，如果一个变量不是用 var 命令声明的，就不会发生变量提升。

```js
console.log(b);
b = 1;
```

上面的语句将会报错，提示“ReferenceError: b is not defined”，即变量 b 未声明，这是因为 b 不是用 var 命令声明的，JavaScript 引擎不会将其提升，而只是视为对顶层对象的 b 属性的赋值。

### 标识符

标识符（identifier）是用来识别具体对象的一个名称。最常见的标识符就是变量名，以及后面要提到的函数名。JavaScript 语言的标识符对大小写敏感，所以 a 和 A 是两个不同的标识符。

标识符有一套命名规则，不符合规则的就是非法标识符。JavaScript 引擎遇到非法标识符，就会报错。

简单说，标识符命名规则如下：

*   第一个字符可以是任意 Unicode 字母，以及美元符号（$）和下划线（_）。
*   第二个字符及后面的字符，还可以用数字。

下面这些都是合法的标识符。

```js
arg0
_tmp
$elem
π
```

下面这些则是不合法的标识符。

```js
1a
23
***
a+b
-d
```

中文是合法的标识符，可以用作变量名。

```js
var 临时变量 = 1;
```

> JavaScript 有一些保留字，不能用作标识符：arguments、break、case、catch、class、const、continue、debugger、default、delete、do、else、enum、eval、export、extends、false、finally、for、function、if、implements、import、in、instanceof、interface、let、new、null、package、private、protected、public、return、static、super、switch、this、throw、true、try、typeof、var、void、while、with、yield。

另外，还有三个词虽然不是保留字，但是因为具有特别含义，也不应该用作标识符：Infinity、NaN、undefined。

### 注释

源码中被 JavaScript 引擎忽略的部分就叫做注释，它的作用是对代码进行解释。Javascript 提供两种注释：一种是单行注释，用//起头；另一种是多行注释，放在/ *和* /之间。

```js
// 这是单行注释

/*
 这是
 多行
 注释
*/
```

本教程后面的代码部分，会采用这两种形式说明代码的运行结果，以及需要注意的地方。

此外，由于历史上 JavaScript 兼容 HTML 代码的注释，所以也被视为单行注释。

```js
x = 1; <!-- x = 2;
--> x = 3;
```

上面代码中，只有`x = 1`会执行，其他的部分都被注释掉了。

需要注意的是，-->只有在行首，才会被当成单行注释，否则就是一个运算符。

```js
function countdown(n) {
  while (n --> 0) console.log(n);
}
countdown(3)
// 2
// 1
// 0
```

上面代码中，`n --> 0`实际上会当作`n-- > 0`，因为输出 2、1、0。

### 区块

JavaScript 使用大括号，将多个相关的语句组合在一起，称为“区块”（block）。

与大多数编程语言不一样，JavaScript 的区块不构成单独的作用域（scope）。也就是说，区块中的变量与区块外的变量，属于同一个作用域。

```js
{ 
    var a = 1;
}

a // 1
```

上面代码在区块内部，声明并赋值了变量 a，然后在区块外部，变量 a 依然有效，这说明区块不构成单独的作用域，与不使用区块的情况没有任何区别。所以，单独使用的区块在 JavaScript 中意义不大，很少出现。区块往往用来构成其他更复杂的语法结构，比如 for、if、while、functions 等。

### 条件语句

JavaScript 提供 if 结构和 switch 结构，完成条件判断。

（1）if 结构

if 结构先判断一个表达式的布尔值，然后根据布尔值的真伪，执行不同的语句。

```js
if (expression) 
  statement
```

上面是 if 结构的基本形式。需要注意的是，expression（表达式）必须放在圆括号中，表示对表达式求值。如果结果为 true，就执行紧跟在后面的 statement（语句）；如果结果为 false，则跳过 statement。

```js
if (m === 3) 
  m += 1;
```

上面代码表示，只有在 m 等于 3 时，才会将其值加上 1。

这种写法要求 statement 只能有一个语句。如果想将多个语句放在 statement 之中，必须在 if 的条件判断之后，加上大括号。

```js
if (m === 3) {
  m += 1; 
}
```

建议总是在 if 语句中使用大括号，因为这样方便插入语句。

（2）if...else 结构

if 代码块后面，还可以跟一个 else 代码块，表示括号中的表示式为 false 时，所要执行的代码。

```js
if (m === 3) {
    // then
 } else {
   // else
 }
```

上面代码判断变量 m 是否等于 3，如果等于就执行 if 代码块，否则执行 else 代码块。

对同一个变量进行多次判断时，多个 if...else 语句可以连写在一起。

```js
if (m === 0) {
    // ...
} else if (m === 1) {
   // ...
} else if (m === 2) {
   // ...
} else {
   // ...
}
```

else 代码块总是跟随离自己最近的那个 if 语句。

```js
var m = 1;
var n = 2;

if (m !== 1) 
if (n === 2) console.log('hello'); 
else console.log('world');
```

上面代码不会有任何输出，else 代码块也不会得到执行，因为它跟着的是最近的那个 if 语句，相当于下面这样。

```js
if (m !== 1) {
    if (n === 2) {
        console.log('hello');  
    } else {
        console.log('world');
    }
}
```

如果想让 else 代码块跟随最上面的那个 if 语句，就要改变大括号的位置。

```js
if (m !== 1) {
    if (n === 2) {
        console.log('hello');  
    } 
} else {
        console.log('world');
} 
// world
```

（3）switch 结构

多个 if...else 连在一起使用的时候，可以转为使用更方便的 switch 结构。

```js
switch (fruit) {
    case "banana":
        // ...
        break;
    case "apple":
        // ...
        break;
    default:
        // ...
}
```

上面代码根据变量 fruit 的值，选择执行相应的 case。如果所有 case 都不符合，则执行最后的 default 部分。需要注意的是，每个 case 代码块内部的 break 语句不能少，否则会接下去执行下一个 case 代码块，而不是跳出 switch 结构。

switch 语句部分和 case 语句部分，都可以使用表达式。

```js
switch(1 + 3) {
    case 2 + 2:
        f();
        break;
    default:
        neverhappens();
}
```

上面代码的 default 部分，是永远不会执行到的。

需要注意的是，switch 语句后面的表达式与 case 语句后面的表示式，在比较运行结果时，采用的是严格相等运算符（===），而不是相等运算符（==），这意味着比较时不会发生类型转换。

switch 结构不利于代码重用，往往可以用对象形式重写。

```js
var o = {
    banana: function (){ return },
    apple: function (){ return },
    default: function (){ return }
};

if (o[fruit]){
    o[fruit]();
} else {
    o['default']();
}
```

### 循环语句

循环语句用于重复执行某个操作，它有多种形式。

（1）while 循环

While 语句包括一个循环条件，只要该条件为真，就不断循环。

```js
while (expression)    
statement
```

while 语句的循环条件是一个表达式（express），必须放在圆括号中。语句（statement）部分默认只能写一条语句，如果需要包括多条语句，必须添加大括号。

```js
while (expression){   
    statement
}
```

下面是 while 语句的一个例子。

```js
var i = 0;

while (i<100){
    console.log('i 当前为：' + i);
    i++;
}
```

上面的代码将循环 100 次，直到 i 等于 100 为止。

（2）for 循环

for 语句是循环命令的另一种形式。

```js
for(initialize; test; increment)
statement

// 或者

for(initialize; test; increment){
    statement
}
```

它分成三步：

*   初始化（initialize）：确定循环的初始值，只在循环开始时执行一次;
*   测试（test）：检查循环条件，只要为真就进行后续操作;
*   递增（increment）：完成后续操作，然后返回上一步，再一次检查循环条件。

下面是一个循环打印数组每个元素的例子。

```js
for (var i=0; i < arr.length; i++) {
    console.log(arr[i]);
}
```

所有 for 循环，都可以改写成 while 循环。

```js
var i = 0;

while (i < arr.length) {
    console.log(arr[i]);
    i++;
}
```

for 语句表达式的三个部分（initialize，test，increment），可以省略任何一个，也可以全部省略。

```js
for (;;){
    console.log('Hello World');
}
```

上面代码省略了 for 语句表达式的三个部分，结果就导致了一个无限循环。

（3）do...while 循环

do...while 循环与 while 循环类似，唯一的区别就是先运行一次循环体，然后判断循环条件。

```js
do 
statement
while(expression);

// 或者

do { 
    statement
} while(expression);
```

不管条件是否为真，do..while 循环至少运行一次，这是这种结构最大的特点。另外，while 语句后面的分号不能省略。

（4）break 语句和 continue 语句

break 语句和 continue 语句都具有跳转作用，可以让代码不按既有的顺序执行。

break 语句用于跳出代码块或循环。

```js
var i = 0;

while (i<100){
    console.log('i 当前为：' + i);
    i++;
    if (i === 10) break;
}
```

上面代码只会执行 10 次循环，一旦 i 等于 10，就会跳出循环。

continue 语句用于立即终止本次循环，返回循环结构的头部，开始下一次循环。

```js
var i = 0;

while (i<100){
    i++;
    if (i%2===0) continue;
    console.log('i 当前为：' + i);
}
```

上面代码只有在 i 为奇数时，才会输出 i 的值。如果 i 为偶数，则直接进入下一轮循环。

如果存在多重循环，不带参数的 break 语句和 continue 语句都只针对最内层循环。

（5）标签（label）

JavaScript 语言允许，语句的前面有标签（label）。标签通常与 break 语句和 continue 语句配合使用，跳出特定的循环。

```js
top:
    for (var i=0;i<3;i++){
        for (var j=0;j<3;j++){
            if (i===1 && j===1) break top;
            console.log("i="+i+",j="+j);
        }
}
// i=0,j=0
// i=0,j=1
// i=0,j=2
// i=1,j=0
```

上面代码为一个双重循环区块，加上了 top 标签（注意，top 不用加引号）。当满足一定条件时，使用 break 语句加上标签名，直接跳出双层循环。如果 break 语句后面不使用标签，则只能跳出内层循环，进入下一次的外层循环。

continue 语句也可以与标签配合使用。

```js
top:
    for (var i=0;i<3;i++){
        for (var j=0;j<3;j++){
            if (i===1 && j===1) continue top;
            console.log("i="+i+",j="+j);
        }
}
// i=0,j=0
// i=0,j=1
// i=0,j=2
// i=1,j=0
// i=2,j=0
// i=2,j=1
// i=2,j=2
```

上面代码在满足一定条件时，使用 continue 语句加上标签名，直接进入下一轮外层循环。如果 continue 语句后面不使用标签，则只能进入下一轮的内层循环。

## 数据类型

### 概述

JavaScript 语言的每一个值，都属于某一种数据类型。JavaScript 的数据类型，共有六个类别和两个特殊值。

六个类别的数据类型又可以分成两组：原始类型（primitive type）和合成类型（complex type）。

原始类型包括三种数据类型。

*   数值（number）
*   字符串（string）
*   布尔值（boolean）

“数值”就是整数和小数(比如 1 和 3.14)，“字符串”就是由多个字符组成的文本（比如"Hello World"），“布尔值”则是 true（真）和 false（假）两个特定值。

合成类型也包括三种数据类型。

*   对象（object）
*   数组（array）
*   函数（function）

对象和数组是两种不同的数据组合方式，而函数其实是处理数据的方法。JavaScript 把函数当成一种数据类型，可以像其他类型的数据一样，进行赋值和传递，这为编程带来了很大的灵活性，体现了 JavaScript 作为“函数式语言”的本质。

这里需要明确的是，JavaScript 的所有数据，都可以视为对象。不仅合成类型的数组和函数属于对象的特例，就连原始类型的数据（数值、字符串、布尔值）也可以用对象方式调用。

除了上面这六个类别，JavaScript 还定义了两个特殊值 null 和 undefined。

本书将分别详细介绍这六个类别和两个特殊值。其中，两个特殊值和布尔类型比较简单，将在本节介绍，其他类型将各自有单独的一节。

### typeof 运算符

JavaScript 有三种方法，可以确定一个值到底是什么类型。

*   typeof 运算符
*   instanceof 运算符
*   Object.prototype.toString 方法

instanceof 运算符和 Object.prototype.toString 方法，将在后文相关章节介绍。这里着重介绍 typeof 运算符。

typeof 运算符可以返回一个值的数据类型，可能有以下结果。

（1）原始类型

数值、字符串、布尔值分别返回 number、string、boolean。

```js
typeof 123 // "number"
typeof "123" // "string"
typeof false // "boolean"
```

（2）函数

函数返回 function。

```js
// 定义一个空函数
function f(){}

typeof f
// "function"
```

（3）undefined

undefined 返回 undefined。

```js
typeof undefined
// "undefined"
```

利用这一点，typeof 可以用来检查一个没有声明的变量，而不报错。

```js
v
// ReferenceError: v is not defined

typeof v
// "undefined"
```

实际编程中，这个特点通常用在判断语句。

```js
// 错误的写法
if (v){
    // ...
}
// ReferenceError: v is not defined

// 正确的写法
if (typeof v === "undefined"){
    // ...
}
```

（4）其他

除此以外，都返回 object。

```js
typeof window // "object"
typeof {} // "object"
typeof [] // "object"
typeof null // "object"
```

从上面代码可以看到，空数组（[]）的类型也是 object，这表示在 JavaScript 内部，数组本质上只是一种特殊的对象。另外，null 的类型也是 object，这是由于历史原因造成的，为了兼容以前的代码，后来就没法修改了，并不是说 null 就属于对象，本质上 null 是一个类似于 undefined 的特殊值。

既然 typeof 对数组（array）和对象（object）的显示结果都是 object，那么怎么区分它们呢？instanceof 运算符可以做到。

```js
var o = {};
var a = [];

o instanceof Array // false
a instanceof Array // true
```

instanceof 运算符的详细解释，请见《面向对象编程》一章。

### null 和 undefined

（1）相似性

首先，null 与 undefined 都可以表示“无”，含义非常相似。将一个变量赋值为 undefined 或 null，老实说，几乎没区别。

```js
var a = undefined;

// 或者

var a = null;
```

上面代码中，a 变量分别被赋值为 undefined 和 null，这两种写法几乎等价。

在 if 语句中，都会被自动转为 false，相等运算符甚至直接报告两者相等。

```js
if (!undefined) 
    console.log('undefined is false');
// undefined is false

if (!null) 
    console.log('null is false');
// null is false

undefined == null
// true
```

上面代码说明，两者的行为是何等相似！Google 公司开发的 JavaScript 语言的替代品 Dart 语言，就明确规定只有 null，没有 undefined！

既然含义与用法都差不多，为什么要同时设置两个这样的值，这不是无端增加复杂度，令初学者困扰吗？这与历史原因有关。

（2）历史原因

1995 年 JavaScript 诞生时，最初像 Java 一样，只设置了 null 作为表示"无"的值。根据 C 语言的传统，null 被设计成可以自动转为 0。

```js
Number(null)
// 0

5 + null
// 5
```

但是，JavaScript 的设计者 Brendan Eich，觉得这样做还不够，有两个原因。

首先，null 像在 Java 里一样，被当成一个对象。但是，JavaScript 的数据类型分成原始类型和合成类型两大类，Brendan Eich 觉得表示"无"的值最好不是对象。

其次，JavaScript 的最初版本没有包括错误处理机制，发生数据类型不匹配时，往往是自动转换类型或者默默地失败。Brendan Eich 觉得，如果 null 自动转为 0，很不容易发现错误。

因此，Brendan Eich 又设计了一个 undefined。他是这样区分的：null 是一个表示"无"的对象，转为数值时为 0；undefined 是一个表示"无"的原始值，转为数值时为 NaN。

```js
Number(undefined)
// NaN

5 + undefined
// NaN
```

但是，这样的区分在实践中很快就被证明不可行。目前，null 和 undefined 基本是同义的，只有一些细微的差别。

（3）用法和含义

对于 null 和 undefined，可以大致上像下面这样理解。

null 表示"没有对象"，即该处不应该有值。典型用法是：

*   作为函数的参数，表示该函数的参数不是对象。

*   作为对象原型链的终点。

undefined 表示"缺少值"，就是此处应该有一个值，但是还未定义。典型用法是：

*   变量被声明了，但没有赋值时，就等于 undefined。

*   调用函数时，应该提供的参数没有提供，该参数等于 undefined。

*   对象没有赋值的属性，该属性的值为 undefined。

*   函数没有返回值时，默认返回 undefined。

```js
var i;
i // undefined

function f(x){console.log(x)}
f() // undefined

var  o = new Object();
o.p // undefined

var x = f();
x // undefined
```

（4）null 的特殊之处

null 的特殊之处在于，JavaScript 把它包含在对象类型（object）之中。

```js
typeof null // "object"
```

上面代码表示，查询 null 的类型，JavaScript 返回 object（对象）。

这并不是说 null 的数据类型就是对象，而是 JavaScript 早期部署中的一个约定俗成，其实不完全正确，后来再想改已经太晚了，会破坏现存代码，所以一直保留至今。

（5）注意点

JavaScript 的标识名区分大小写，所以 undefined 和 null 不同于 Undefined 和 Null（或者其他仅仅大小写不同的词形），后者只是普通的变量名。

### 布尔值

布尔值代表“真”和“假”两个状态。“真”用关键字 true 表示，“假”用关键字 false 表示。布尔值只有这两个值。

下列运算符会返回布尔值：

*   两元逻辑运算符： && (And)，|| (Or)
*   前置逻辑运算符： ! (Not)
*   相等运算符：===，!==，==，!=
*   比较运算符：>，>=，<，<=

如果 JavaScript 预期某个位置应该是布尔值，会将该位置上现有的值自动转为布尔值。转换规则是除了下面六个值被转为 false，其他值都视为 true。

*   undefined
*   null
*   false
*   0
*   NaN
*   ""（空字符串）

布尔值往往用于程序流程的控制，请看一个例子。

```js
if (""){ console.log(true);}
// 没有任何输出
```

上面代码的 if 命令后面的判断条件，预期应该是一个布尔值，所以 JavaScript 自动将空字符串，转为布尔值 false，导致程序不会进入代码块，所以没有任何输出。

需要特别注意的是，空数组（[]）和空对象（{}）对应的布尔值，都是 true。

```js
if ([]){ console.log(true);}
// true

if ({}){ console.log(true);}
// true
```

更多关于数据类型转换的介绍，参见《数据类型转换》一节。

## 结尾的分号

### 不使用分号结尾的语句

分号表示一条语句的结尾。但是，有一些语法结构不需要在语句的结尾添加分号，主要是以下三种情况。

（1）for 和 while 循环

```js
for(;;){} // 没有分号

while(true){} // 没有分号
```

需要注意的是 do...while 循环是有分号的。

```js
do {
    a--;
} while(a > 0); // 分号不能省略
```

（2）分支语句：if， switch， try

```js
if (true) {} // 没有分号

switch () {} // 没有分号

try {} catch {} // 没有分号
```

（3）函数的声明语句

```js
function f() {} // 没有分号
```

但是函数表达式仍然要使用分号。

```js
var f = function f() {};
```

以上三种情况，如果使用了分号，并不会出错。因为，解释引擎会把这个分号解释为空语句。

### 分号的自动添加

除了本来就不写分号的情况，JavaScript 引擎还有一个特点，就是在应该写分号却没写的情况下，它会自动添加（Automatic Semicolon Insertion，简称 ASI）。

```js
var a = b + c
// 等同于
var a = b + c;
```

但是，这种自动添加不是绝对的。如果下一行的开始可以与本行的结尾连在一起解释，就不会自动添加分号。

```js
var
a
=
3

// 等同于

var a = 3;

"abc"
.length

// 等同于

"abc".length
```

上面代码举了两个例子，每行的尾部都没有分号，JavaScript 并不会自动添加分号，因为每行的结尾与下一行的开头可以放在一起解释。下面这个例子也不会自动添加分号。

```js
3 * (2 * (4 + (3 - 5))) 
+ 
(10 * (27 / 6))

// 等同于

3 * (2 * (4 + (3 - 5))) + (10 * (27 / 6))
```

这些例子还是比较容易看出来的，但是下面的例子就不那么容易发现了。它们都不会自动添加分号。

```js
var a = b + c
(d+e).toString();
/* 结果报错，因为两行连在一起，
   解释为 c(d+e)，
   即对函数 c 的调用 */

a = b
/hi/g.exec(c).map(d);
/* 解释为 a = b / hi / g.exec(c).map(d)，
   即把正则表达式的斜杠当作除法运算符 */ 

var a = "b"
[ "red", "green" ].forEach(function(c) { console.log(c) })
/* 结果报错，因为两行连在一起，
 解释为"b"["red", "green"]，
 即把字符串当作一个数组，按索引取值 */ 

var a = 0;
var f = function(x) { return x }
(a++)
/* f 等于 0，因为(a++)被
 * 视为匿名函数的调用 */

return a +
b;

return (a
+ b)

obj.foo(arg1,
arg2)
```

一般来说，在没有分号结尾的情况下，如果下一行起首的是(、 [ 、+、-、/这五个字符中的一个，分号不会被自动添加。只有下一行的开始与本行的结尾，无法放在一起解释，JavaScript 引擎才会自动添加分号。

```js
if (a < 0) a = 0
console.log(a)

// 等同于下面的代码，
// 因为 0console 没有意义

if (a < 0) a = 0;
console.log(a)
```

另外，如果一行的起首是“自增”（++）或“自减”（--）运算符，则它们的前面会自动添加分号。

```js
a = b = c = 1
a
++
b
--
c

console.log(a, b, c)
// 1 2 0
```

之所以会得到“1 2 0”的结果，原因是自增和自减运算符前，自动被加上了分号。上面的代码实际上等同于下面的形式：

```js
a = b = c = 1;
a;
++b;
--c;
```

如果 continue、break、return 和 throw 这四个语句后面，直接跟换行符，则会自动添加分号。这意味着，如果 return 语句返回的是一个对象的字面量，起首的大括号一定要写在同一行，否则得不到预期结果。

```js
return
{ first: "Jane" };

// 解释成
return;
{ first: "Jane" };
```

由于解释引擎自动添加分号的行为难以预测，因此编写代码的时候不应该省略行尾的分号。

省略结尾的分号，还有一个问题。有些 JavaScript 代码压缩器不会自动添加分号，因此遇到没有分号的结尾，就会让代码保持原状，而不是压缩成一行。

## 参考链接

*   Axel Rauschmayer, [A quick overview of JavaScript](http://www.2ality.com/2011/10/javascript-overview.html)
*   Axel Rauschmayer, [Improving the JavaScript typeof operator](http://www.2ality.com/2011/11/improving-typeof.html)
*   Axel Rauschmayer, [Automatic semicolon insertion in JavaScript](http://www.2ality.com/2011/05/semicolon-insertion.html)
*   Axel Rauschmayer, [Categorizing values in JavaScript](http://www.2ality.com/2013/01/categorizing-values.html)
*   Rod Vagg, [JavaScript and Semicolons](http://dailyjs.com/2012/04/19/semicolons/)