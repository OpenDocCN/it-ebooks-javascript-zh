# 3.3 包装对象和 Boolean 对象

*   包装对象
    *   定义
    *   包装对象的构造函数
    *   包装对象实例的方法
    *   原始类型的自动转换
    *   自定义方法
    *   Boolean 对象
        *   概述
        *   Boolean 实例对象的布尔值
        *   Boolean 函数的类型转换作用

## 包装对象

### 定义

在 JavaScript 中，“一切皆对象”，数组和函数本质上都是对象，就连三种原始类型的值——数值、字符串、布尔值——在一定条件下，也会自动转为对象，也就是原始类型的“包装对象”。

所谓“包装对象”，就是分别与数值、字符串、布尔值相对应的 Number、String、Boolean 三个原生对象。这三个原生对象可以把原始类型的值变成（包装成）对象。

```js
var v1 = new Number(123);
var v2 = new String("abc");
var v3 = new Boolean(true);
```

上面代码根据原始类型的值，生成了三个对象，与原始值的类型不同。这用 typeof 运算符就可以看出来。

```js
typeof v1 // "object"
typeof v2 // "object"
typeof v3 // "object"

v1 === 123 // false
v2 === "abc" // false
v3 === true // false
```

JavaScript 设计包装对象的最大目的，首先是使得 JavaScript 的“对象”涵盖所有的值。其次，使得原始类型的值可以方便地调用特定方法。

### 包装对象的构造函数

Number、String 和 Boolean 这三个原生对象，既可以当作构造函数使用（即加上 new 关键字，生成包装对象实例），也可以当作工具方法使用（即不加 new 关键字，直接调用），这相当于生成实例后再调用 valueOf 方法，常常用于将任意类型的值转为某种原始类型的值。

```js
Number(123) // 123

String("abc") // "abc"

Boolean(true) // true
```

工具方法的详细介绍参见第二章的《数据类型转换》一节。

### 包装对象实例的方法

包装对象实例可以使用 Object 对象提供的原生方法，主要是 valueOf 方法和 toString 方法。

（1）valueOf 方法

valueOf 方法返回包装对象实例对应的原始类型的值。

```js
new Number(123).valueOf()
// 123

new String("abc").valueOf()
// "abc"

new Boolean("true").valueOf()
// true
```

（2）toString 方法

toString 方法返回该实例对应的原始类型值的字符串形式。

```js
new Number(123).toString()
// "123"

new String("abc").toString()
// "abc"

new Boolean("true").toString()
// "true"
```

### 原始类型的自动转换

原始类型可以自动调用定义在包装对象上的方法和属性。比如 String 对象的实例有一个 length 属性，返回字符串的长度。

```js
var v = new String("abc");
v.length // 3
```

所有原始类型的字符串，都可以直接使用这个 length 属性。

```js
"abc".length // 3
```

上面代码对字符串 abc 调用 length 属性，实际上是将“字符串”自动转为 String 对象的实例，再在其上调用 length 属性。这就叫原始类型的自动转换。

abc 是一个字符串，属于原始类型，本身不能调用任何方法和属性。但当对 abc 调用 length 属性时，JavaScript 引擎自动将 abc 转化为一个包装对象实例，然后再对这个实例调用 length 属性，在得到返回值后，再自动销毁这个临时生成的包装对象实例。

这种原始类型值可以直接调用的方法还有很多（详见后文对各包装对象的介绍），除了前面介绍过的 valueOf 和 stringOf 方法，还包括三个包装对象各自定义在实例上的方法。。

```js
'abc'.charAt === String.prototype.charAt
// true
```

上面代码表示，字符串 abc 的 charAt 方法，实际上就是定义在 String 对象实例上的方法（关于 prototype 对象的介绍参见《面向对象编程》一章）。

如果包装对象与原始类型值进行混合运算，包装对象会转化为原始类型（实际是调用自身的 valueOf 方法）。

```js
new Number(123) + 123
// 246

new String("abc") + "abc"
// "abcabc"
```

### 自定义方法

三种包装对象还可以在原型上添加自定义方法和属性，供原始类型的值直接调用。

比如，我们可以新增一个 double 方法，使得字符串和数字翻倍。

```js
String.prototype.double = function (){
    return this.valueOf() + this.valueOf();
};

"abc".double()
// abcabc

Number.prototype.double = function (){
    return this.valueOf() + this.valueOf();
};

(123).double()
// 246
```

上面代码在 123 外面必须要加上圆括号，否则后面的点运算符（.）会被解释成小数点。

但是，这种自定义方法和属性的机制，只能定义在包装对象的原型上，如果直接对原始类型的变量添加属性，则无效。

```js
var s = "abc";

s.p = 123;
s.p // undefined
```

上面代码直接对支付串 abc 添加属性，结果无效。

## Boolean 对象

### 概述

Boolean 对象是 JavaScript 的三个包装对象之一。作为构造函数，它主要用于生成布尔值的包装对象的实例。

```js
var b = new Boolean(true);

typeof b // "object"
b.valueOf() // true
```

上面代码的变量 b 是一个 Boolean 对象的实例，它的类型是对象，值为布尔值 true。这种写法太繁琐，几乎无人使用，直接对变量赋值更简单清晰。

```js
var b = true;
```

### Boolean 实例对象的布尔值

特别要注意的是，所有对象的布尔运算结果都是 true。因此，false 对应的包装对象实例，布尔运算结果也是 true。

```js
if (new Boolean(false)) {
    console.log("true"); 
} // true

if (new Boolean(false).valueOf()) {
    console.log("true"); 
} // 无输出
```

上面代码的第一个例子之所以得到 true，是因为 false 对应的包装对象实例是一个对象，进行逻辑运算时，被自动转化成布尔值 true（所有对象对应的布尔值都是 true）。而实例的 valueOf 方法，则返回实例对应的原始类型值，本例为 false。

### Boolean 函数的类型转换作用

Boolean 对象除了可以作为构造函数，还可以单独使用，将任意值转为布尔值。这时 Boolean 就是一个单纯的工具方法。

```js
Boolean(undefined) // false
Boolean(null) // false
Boolean(0) // false
Boolean('') // false
Boolean(NaN) // false
Boolean(1) // true
Boolean('false') // true
Boolean([]) // true
Boolean({}) // true
Boolean(function(){}) // true
Boolean(/foo/) // true
```

上面代码中几种得到 true 的情况，都值得认真记住。

使用 not 运算符（!）也可以达到同样效果。

```js
!!undefined // false
!!null // false
!!0 // false
!!'' // false
!!NaN // false
!!1 // true
!!'false' // true
!![] // true
!!{} // true
!!function(){} // true
!!/foo/ // true
```

综上所述，如果要获得一个变量对应的布尔值，有多种写法。

```js
var a = "hello world";

new Boolean(a).valueOf() // true
Boolean(a) // true
!!a // true
```

最后，对于一些特殊值，Boolean 对象前面加不加 new，会得到完全相反的结果，必须小心。

```js
if (Boolean(false)) 
        console.log('true'); // 无输出

if (new Boolean(false))
        console.log('true'); // true

if (Boolean(null)) 
        console.log('true'); // 无输出

if (new Boolean(null))
        console.log('true'); // true
```