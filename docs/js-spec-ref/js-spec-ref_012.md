# 2.8 数据类型转换

JavaScript 是一种动态类型语言，变量是没有类型的，可以随时赋予任意值。但是，数据本身和各种运算是有类型的，因此运算时变量需要转换类型。大多数情况下，这种数据类型转换是自动的，但是有时也需要手动强制转换。

*   强制转换
    *   Number 函数：强制转换成数值
    *   String 函数：强制转换成字符串
    *   Boolean 函数：强制转换成布尔值
    *   自动转换
        *   自动转换为布尔值
        *   自动转换为字符串
        *   自动转换为数值
        *   小结
    *   加法运算符的类型转化
        *   三种情况
        *   四个特殊表达式
    *   参考链接

## 强制转换

强制转换主要指使用 Number、String 和 Boolean 三个构造函数，手动将各种类型的值，转换成数字、字符串或者布尔值。

### Number 函数：强制转换成数值

使用 Number 函数，可以将任意类型的值转化成数字。

（1）原始类型值的转换规则

*   数值：转换后还是原来的值。

*   字符串：如果可以被解析为数值，则转换为相应的数值，否则得到 NaN。空字符串转为 0。

*   布尔值：true 转成 1，false 转成 0。

*   undefined：转成 NaN。

*   null：转成 0。

```
Number("324") // 324

Number("324abc") // NaN

Number("") // 0

Number(false) // 0

Number(undefined) // NaN

Number(null) // 0
```

Number 函数将字符串转为数值，要比 parseInt 函数严格很多。基本上，只要有一个字符无法转成数值，整个字符串就会被转为 NaN。

```
parseInt('011') // 9
parseInt('42 cats') // 42
parseInt('0xcafebabe') // 3405691582

Number('011') // 11
Number('42 cats') // NaN
Number('0xcafebabe') // 3405691582
```

上面代码比较了 Number 函数和 parseInt 函数，区别主要在于 parseInt 逐个解析字符，而 Number 函数整体转换字符串的类型。另外，Number 会忽略八进制的前导 0，而 parseInt 不会。

Number 函数会自动过滤一个字符串前导和后缀的空格。

```
Number('\t\v\r12.34\n ')
```

（2）对象的转换规则

对象的转换规则比较复杂。

1.  先调用对象自身的 valueOf 方法，如果该方法返回原始类型的值（数值、字符串和布尔值），则直接对该值使用 Number 方法，不再进行后续步骤。

2.  如果 valueOf 方法返回复合类型的值，再调用对象自身的 toString 方法，如果 toString 方法返回原始类型的值，则对该值使用 Number 方法，不再进行后续步骤。

3.  如果 toString 方法返回的是复合类型的值，则报错。

```
Number({a:1})
// NaN
```

上面代码等同于

```
if (typeof {a:1}.valueOf() === 'object'){
    Number({a:1}.toString());
} else {
    Number({a:1}.valueOf());
}
```

上面代码的 valueOf 方法返回对象本身（{a:1}），所以对 toString 方法的返回值“[object Object]”使用 Number 方法，得到 NaN。

如果 toString 方法返回的不是原始类型的值，结果就会报错。

```
var obj = {
    valueOf: function () {
            console.log("valueOf");
            return {};
    },
    toString: function () {
            console.log("toString");
            return {}; 
    }
};

Number(obj)
// TypeError: Cannot convert object to primitive value
```

上面代码的 valueOf 和 toString 方法，返回的都是对象，所以转成数值时会报错。

从上面的例子可以看出，valueOf 和 toString 方法，都是可以自定义的。

```
Number({valueOf:function (){return 2;}})
// 2

Number({toString:function(){return 3;}})
// 3

Number({valueOf:function (){return 2;},toString:function(){return 3;}})
// 2
```

上面代码对三个对象使用 Number 方法。第一个对象返回 valueOf 方法的值，第二个对象返回 toString 方法的值，第三个对象表示 valueOf 方法先于 toString 方法执行。

### String 函数：强制转换成字符串

使用 String 函数，可以将任意类型的值转化成字符串。规则如下：

（1）原始类型值的转换规则

*   数值：转为相应的字符串。

*   字符串：转换后还是原来的值。

*   布尔值：true 转为“true”，false 转为“false”。

*   undefined：转为“undefined”。

*   null：转为“null”。

```
String(123) // "123"

String("abc") // "abc"

String(true) // "true"

String(undefined) // "undefined"

String(null) // "null"
```

（2）对象的转换规则

如果要将对象转为字符串，则是采用以下步骤。

1.  先调用 toString 方法，如果 toString 方法返回的是原始类型的值，则对该值使用 String 方法，不再进行以下步骤。

2.  如果 toString 方法返回的是复合类型的值，再调用 valueOf 方法，如果 valueOf 方法返回的是原始类型的值，则对该值使用 String 方法，不再进行以下步骤。

3.  如果 valueOf 方法返回的是复合类型的值，则报错。

String 方法的这种过程正好与 Number 方法相反。

```
String({a:1})
// "[object Object]"
```

上面代码相当于下面这样。

```
String({a:1}.toString())
// "[object Object]"
```

如果 toString 方法和 valueOf 方法，返回的都不是原始类型的值，则 String 方法报错。

```
var obj = {
    valueOf: function () {
            console.log("valueOf");
            return {}; 
    },
    toString: function () {
            console.log("toString");
            return {}; 
    }
};

String(obj)
// TypeError: Cannot convert object to primitive value
```

下面是一个自定义 toString 方法的例子。

```
String({toString:function(){return 3;}})
// "3"

String({valueOf:function (){return 2;}})
// "[object Object]"

String({valueOf:function (){return 2;},toString:function(){return 3;}})
// "3"
```

上面代码对三个对象使用 String 方法。第一个对象返回 toString 方法的值（数值 3），然后对其使用 String 方法，得到字符串“3”；第二个对象返回的还是 toString 方法的值（"[object Object]"），这次直接就是字符串；第三个对象表示 toString 方法先于 valueOf 方法执行。

### Boolean 函数：强制转换成布尔值

使用 Boolean 函数，可以将任意类型的变量转为布尔值。

（1）原始类型值的转换方法

以下六个值的转化结果为 false，其他的值全部为 true。

*   undefined
*   null
*   -0
*   +0
*   NaN
*   ''（空字符串）

```
Boolean(undefined) // false

Boolean(null) // false

Boolean(0) // false

Boolean(NaN) // false

Boolean('') // false
```

（2）对象的转换规则

所有对象的布尔值都是 true，甚至连 false 对应的布尔对象也是 true。

```
Boolean(new Boolean(false))
// true
```

请注意，空对象{}和空数组[]也会被转成 true。

```
Boolean([]) // true

Boolean({}) // true
```

## 自动转换

当遇到以下几种情况，JavaScript 会自动转换数据类型：

*   不同类型的数据进行互相运算；

*   对非布尔值类型的数据求布尔值;

*   对非数值类型的数据使用一元运算符（即“+”和“-”）。

### 自动转换为布尔值

当 JavaScript 遇到预期为布尔值的地方（比如 if 语句的条件部分），就会将非布尔值的参数自动转换为布尔值。它的转换规则与上面的“强制转换成布尔值”的规则相同，也就是说，在预期为布尔值的地方，系统内部会自动调用 Boolean 方法。

因此除了以下六个值，其他都是自动转为 true：

*   undefined
*   null
*   -0
*   +0
*   NaN
*   ''（空字符串）

```
if (!undefined && !null && !0 && !NaN && !''){
    console.log('true');
}
// true
```

### 自动转换为字符串

当 JavaScript 遇到预期为字符串的地方，就会将非字符串的数据自动转为字符串，转换规则与“强制转换为字符串”相同。

字符串的自动转换，主要发生在加法运算时。当一个值为字符串，另一个值为非字符串，则后者转为字符串。

```
'5' + 1 // '51'
'5' + true // "5true"
'5' + false // "5false"
'5' + {} // "5[object Object]"
'5' + [] // "5"
'5' + function (){} // "5function (){}"
'5' + undefined // "5undefined"
'5' + null // "5null"
```

### 自动转换为数值

当 JavaScript 遇到预期为数值的地方，就会将参数值自动转换为数值，转换规则与“强制转换为数值”相同。

除了加法运算符有可能把运算子转为字符串，其他运算符都会把两侧的运算子自动转成数值。

```
'5' - '2' // 3
'5' * '2' // 10
true - 1  // 0
false - 1 // -1
'1' - 1   // 0
'5'*[]    // 0
false/'5' // 0
'abc'-1   // NaN
```

上面都是二元算术运算符的例子，JavaScript 的两个一元算术运算符——正号和负号——也会把运算子自动转为数值。

```
+'abc' // NaN
-'abc' // NaN
+true // 1
-false // 0
```

### 小结

由于自动转换有很大的不确定性，而且不易除错，建议在预期为布尔值、数值、字符串的地方，全部使用 Boolean、Number 和 String 方法进行显式转换。

## 加法运算符的类型转化

加法运算符（+）需要特别讨论，因为它可以完成两种运算（加法和字符连接），所以不仅涉及到数据类型的转换，还涉及到确定运算类型。

### 三种情况

加法运算符的类型转换，可以分成三种情况讨论。

（1）运算子之中存在字符串

两个运算子之中，只要有一个是字符串，则另一个不管是什么类型，都会被自动转为字符串，然后执行字符串连接运算。前面的《自动转换为字符串》一节，已经举了很多例子。

（2）两个运算子都为数值或布尔值

这种情况下，执行加法运算，布尔值转为数值（true 为 1，false 为 0）。

```
true + 5 // 6

true + true // 2
```

（3）运算子之中存在对象

运算子之中存在对象（或者准确地说，存在非原始类型的值），则先调用该对象的 valueOf 方法。如果返回结果为原始类型的值，则运用上面两条规则；否则继续调用该对象的 toString 方法，对其返回值运用上面两条规则。

```
1 + [1,2]
// "11,2"
```

上面代码的运行顺序是，先调用[1,2].valueOf()，结果还是数组[1,2]本身，则继续调用[1,2].toString()，结果字符串“1,2”，所以最终结果为字符串“11,2”。

```
1 + {a:1}
// "1[object Object]"
```

对象{a:1}的 valueOf 方法，返回的就是这个对象的本身，因此接着对它调用 toString 方法。({a:1}).toString()默认返回字符串"[object Object]"，所以最终结果就是字符串“1[object Object]”

有趣的是，如果更换上面代码的运算次序，就会得到不同的值。

```
{a:1} + 1
// 1
```

原来此时，JavaScript 引擎不将{a:1}视为对象，而是视为一个代码块，这个代码块没有返回值，所以被忽略。因此上面的代码，实际上等同于 {a:1};+1 ，所以最终结果就是 1。为了避免这种情况，需要对{a:1}加上括号。

```
({a:1})+1
"[object Object]1"
```

将{a:1}放置在括号之中，由于 JavaScript 引擎预期括号之中是一个值，所以不把它当作代码块处理，而是当作对象处理，所以最终结果为“[object Object]1”。

```
1 + {valueOf:function(){return 2;}}
// 3
```

上面代码的 valueOf 方法返回数值 2，所以最终结果为 3。

```
1 + {valueOf:function(){return {};}}
// "1[object Object]"
```

上面代码的 valueOf 方法返回一个空对象，则继续调用 toString 方法，所以最终结果是“1[object Object]”。

```
1 + {valueOf:function(){return {};}, toString:function(){return 2;}}
// 3
```

上面代码的 toString 方法返回数值 2（不是字符串），则最终结果就是数值 3。

```
1 + {valueOf:function(){return {};}, toString:function(){return {};}}
// TypeError: Cannot convert object to primitive value
```

上面代码的 toString 方法返回一个空对象，JavaScript 就会报错，表示无法获得原始类型的值。

### 四个特殊表达式

有了上面这些例子，我们再进一步来看四个特殊表达式。

（1）空数组 + 空数组

```
[] + []
// ""
```

首先，对空数组调用 valueOf 方法，返回的是数组本身；因此再对空数组调用 toString 方法，生成空字符串；所以，最终结果就是空字符串。

（2）空数组 + 空对象

```
[] + {}
// "[object Object]"
```

这等同于空字符串与字符串“[object Object]”相加。因此，结果就是“[object Object]”。

（3）空对象 + 空数组

```
{} + []
// 0
```

JavaScript 引擎将空对象视为一个空的代码块，加以忽略。因此，整个表达式就变成“+ []”，等于对空数组求正值，因此结果就是 0。转化过程如下：

```
+ []
// Number([])
// Number([].toString())
// Number("")
// 0
```

如果 JavaScript 不把前面的空对象视为代码块，则结果为字符串“[object Object]”。

```
({}) + []
// "[object Object]"
```

（4）空对象 + 空对象

```
{} + {}
// NaN
```

JavaScript 同样将第一个空对象视为一个空代码块，整个表达式就变成“+ {}”。这时，后一个空对象的 ValueOf 方法得到本身，再调用 toSting 方法，得到字符串“[object Object]”，然后再将这个字符串转成数值，得到 NaN。所以，最后的结果就是 NaN。转化过程如下：

```
+ {}
// Number({})
// Number({}.toString())
// Number("[object Object]")
```

如果，第一个空对象不被 JavaScript 视为空代码块，就会得到“[object Object][object Object]”的结果。

```
({}) + {}
// "[object Object][object Object]"

({} + {})
// "[object Object][object Object]"  

console.log({} + {})
// "[object Object][object Object]"

var a = {} + {};
a
// "[object Object][object Object]"
```

需要指出的是，对于第三和第四种情况，Node.js 的运行结果不同于浏览器环境。

```
{} + {}
// "[object Object][object Object]"

{} + []
// "[object Object]"
```

可以看到，Node.js 没有把第一个空对象视为代码块。原因是 Node.js 的命令行环境，内部执行机制大概是下面的样子：

```
eval.call(this,"(function(){return {} + {}}).call(this)")
```

Node.js 把命令行输入都放在 eval 中执行，所以不会把起首的大括号理解为空代码块加以忽略。

## 参考链接

*   Axel Rauschmayer, [What is {} + {} in JavaScript?](http://www.2ality.com/2012/01/object-plus-object.html)
*   Axel Rauschmayer, [JavaScript quirk 1: implicit conversion of values](http://www.2ality.com/2013/04/quirk-implicit-conversion.html)
*   Benjie Gillam, [Quantum JavaScript?](http://www.benjiegillam.com/2013/06/quantum-javascript/)