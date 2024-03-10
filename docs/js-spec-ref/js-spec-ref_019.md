# 3.4 Number 对象

*   概述
*   Number 对象的属性
*   Number 对象实例的方法
    *   Number.prototype.toString()
    *   Number.prototype.toFixed()
    *   Number.prototype.toExponential()
    *   Number.prototype.toPrecision()
    *   自定义方法

## 概述

Number 对象是数值对应的包装对象，可以作为构造函数使用，也可以作为工具函数使用。

作为构造函数时，它用于生成值为数值的对象。

```
var n = new Number(1);
typeof n // "object"
```

上面代码中，Number 对象作为构造函数使用，返回一个值为 1 的对象。

作为工具函数时，它可以将任何类型的值转为数值。

```
Number(true) // 1
```

上面代码将布尔值 true 转为数值 1。Number 对象的工具方法，详细介绍参见上一章的《数据类型转换》一节。

## Number 对象的属性

Number 对象拥有一些特别的属性。

（1）Number.POSITIVE_INFINITY

表示正的无限，指向关键字 Infinity。

（2）Number.NEGATIVE_INFINITY

表示负的无限，指向-Infinity。

（3）Number.NaN

表示非数值，指向 NaN。

（4）Number.MAX_VALUE

表示最大的正数，相应的，最小的负数为-Number.MAX_VALUE。

（5）Number.MIN_VALUE

表示最小的正数（即最接近 0 的正数，在 64 位浮点数体系中为 5e-324），相应的，最接近 0 的负数为-Number.MIN_VALUE。

```
Number.POSITIVE_INFINITY // Infinity
Number.NEGATIVE_INFINITY // -Infinity
Number.NaN // NaN
Number.MAX_VALUE // 1.7976931348623157e+308
Number.MIN_VALUE // 5e-324
```

## Number 对象实例的方法

### Number.prototype.toString()

Number 对象部署了单独的 toString 方法，可以接受一个参数，表示将一个数字转化成某个进制的字符串。

```
(10).toString() // "10"
(10).toString(2) // "1010"
(10).toString(8) // "12"
(10).toString(16) // "a"
```

之所以要把 10 放在括号里，是为了表明 10 是一个单独的数值，后面的点表示调用对象属性。如果不加括号，这个点会被 JavaScript 引擎解释成小数点，从而报错。

```
10.toString(2) 
// SyntaxError: Unexpected token ILLEGAL
```

但是，在 10 后面加两个点，JavaScript 会把第一个点理解成小数点（即 10.0），把第二个点理解成调用对象属性，从而得到正确结果。

```
10..toString(2) 
// "1010"
```

这实际上意味着，可以直接对一个小数使用 toString 方法。

```
10.5.toString() // "10.5"
10.5.toString(2) // "1010.1"
10.5.toString(8) // "12.4"
10.5.toString(16) // "a.8"
```

通过方括号运算符也可以调用 toString 方法。

```
10'toString' // "1010"
```

将其他进制的数，转回十进制，需要使用 parseInt 方法。

### Number.prototype.toFixed()

toFixed 方法用于将一个数转为指定位数的小数。

```
(10).toFixed(2)
// "10.00"
// 10 必须放在括号里，否则后面的点运算符会被处理小数点，而不是表示调用对象的方法。

(10.005).toFixed(2)
// "10.01"
```

toFixed 方法的参数为小数的位数，有效范围为 0 到 20，超出这个范围将抛出 RangeError 错误。。

### Number.prototype.toExponential()

toExponential 方法用于将一个数转为科学计数法形式。

```
(10).toExponential(1)
// "1.0e+1"

(1234).toExponential(1)
// "1.2e+3"
```

toExponential 方法的参数表示小数点后有效数字的位数，范围为 0 到 20，超出这个范围，会抛出一个 RangeError。

### Number.prototype.toPrecision()

toPrecision 方法用于将一个数转为指定位数的有效数字。

```
(12.34).toPrecision(1)
// "1e+1"

(12.34).toPrecision(2)
// "12"

(12.34).toPrecision(3)
// "12.3"

(12.34).toPrecision(4)
// "12.34"

(12.34).toPrecision(5)
// "12.340"
```

toPrecision 方法的参数为有效数字的位数，范围是 1 到 21，超出这个范围会抛出 RangeError 错误。

toPrecision 方法用于四舍五入时不太可靠，可能跟浮点数不是精确储存有关。

```
(12.35).toPrecision(3)
// "12.3"

(12.25).toPrecision(3)
// "12.3"

(12.15).toPrecision(3)
// "12.2"

(12.45).toPrecision(3)
// "12.4"
```

## 自定义方法

与其他对象一样，Number.prototype 对象上面可以自定义方法，被 Number 的实例继承。

```
Number.prototype.add = function (x) {
  return this + x;
};
```

上面代码为 Number 对象实例定义了一个 add 方法。

由于 Number 对象的实例就是数值，在数值上调用某个方法，数值会自动转为对象，所以就得到了下面的结果。

```
8'add'
// 10
```

上面代码中，调用方法之所以写成`8['add']`，而不是`8.add`，是因为数值后面的点，会被解释为小数点，而不是点运算符。将数值放在圆括号中，就可以使用点运算符调用方法了。

```
(8).add(2)
// 10
```

由于 add 方法返回的还是数值，所以可以链式运算。

```
Number.prototype.subtract = function (x) {
  return this - x;
};

(8).add(2).subtract(4)
// 6
```

上面代码在 Number 对象的实例上部署了 subtract 方法，它可以与 add 方法链式调用。

我们还可以部署更复杂的方法。

```
Number.prototype.iterate = function () {
  var result = [];
  for (var i = 0; i <= this; i++) {
    result.push(i);
  }
  return result;
};

(8).iterate()
// [0, 1, 2, 3, 4, 5, 6, 7, 8]
```

上面代码在 Number 对象的原型上部署了 iterate 方法，可以将一个数值自动遍历为一个数组。

需要注意的是，数值的自定义方法，只能定义在它的原型对象 Number.prototype 上面，数值本身是无法自定义属性的。

```
var n = 1;
n.x = 1;
n.x // undefined
```

上面代码中，n 是一个原始类型的数值。直接在它上面新增一个属性 x，不会报错，但毫无作用，总是返回 undefined。这是因为一旦被调用属性，n 就自动转为 Number 的实例对象，调用结束后，该对象自动销毁。所以，下一次调用 n 的属性时，实际取到的是另一个对象，属性 x 当然就读不出来。