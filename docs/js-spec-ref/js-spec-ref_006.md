# 2.2 数值

*   概述
    *   整数和浮点数
    *   数值精度
    *   数值范围
    *   数值的表示法
    *   特殊数值
        *   正零和负零
        *   NaN
        *   Infinity
    *   与数值相关的全局方法
        *   parseInt 方法
        *   parseFloat 方法
    *   参考链接

## 概述

### 整数和浮点数

JavaScript 内部，所有数字都是以 64 位浮点数形式储存，即使整数也是如此。所以，1 与 1.0 是相等的，而且 1 加上 1.0 得到的还是一个整数，不会像有些语言那样变成小数。

```js
1 === 1.0 // true

1 + 1.0 // 2
```

由于浮点数不是精确的值，所以涉及小数的比较和运算要特别小心。

```js
0.1 + 0.2 === 0.3
// false

0.3 / 0.1
// 2.9999999999999996

(0.3-0.2) === (0.2-0.1)
// false
```

### 数值精度

根据国际标准 IEEE 754，64 位浮点数格式的 64 个二进制位中，第 0 位到第 51 位储存有效数字部分，第 52 到第 62 位储存指数部分，第 63 位是符号位，0 表示正数，1 表示负数。

因此，JavaScript 提供的有效数字的精度为 53 个二进制位（IEEE 754 规定有效数字第一位默认为 1，再加上后面的 52 位），也就是说，绝对值小于等于 2 的 53 次方的整数都可以精确表示。

```js
Math.pow(2, 53)  // 54 个二进制位
// 9007199254740992

Math.pow(2, 53) + 1
// 9007199254740992

Math.pow(2, 53) + 2
// 9007199254740994

Math.pow(2, 53) + 3
// 9007199254740996

Math.pow(2, 53) + 4
// 9007199254740996
```

从上面示例可以看到，大于 2 的 53 次方以后，整数运算的结果开始出现错误。所以，大于等于 2 的 53 次方的数值，都无法保持精度。

```js
Math.pow(2, 53) 
// 9007199254740992

9007199254740992111
// 9007199254740992000
```

上面示例表明，大于 2 的 53 次方以后，多出来的有效数字（最后三位的 111）都会无法保存，变成 0。

### 数值范围

另一方面，64 位浮点数的指数部分的长度是 11 个二进制位，意味着指数部分的最大值是 2047（2 的 11 次方减 1）。也就是说，64 位浮点数的指数部分的值最大为 2047，分出一半表示负数，则 JavaScript 能够表示的数值范围为 21024 到 2-1023（开区间），超出这个范围的数无法表示。

如果指数部分等于或超过最大正值 1024，JavaScript 会返回 Infinity（关于 Infinity 的介绍参见下文），这称为“正向溢出”；如果等于或超过最小负值-1023（即非常接近 0），JavaScript 会直接把这个数转为 0，这称为“负向溢出”。事实上，JavaScript 对指数部分的两个极端值（11111111111 和 00000000000）做了定义，11111111111 表示 NaN 和 Infinity，00000000000 表示 0。

```js
var x = 0.5;
for(var i =0;i<25;i++) x = x*x;

x // 0
```

上面代码对 0.5 连续做 25 次平方，由于最后结果太接近 0，超出了可表示的范围，JavaScript 就直接将其转为 0。

至于具体的最大值和最小值，JavaScript 提供 Number 对象的 MAX_VALUE 和 MIN_VALUE 属性表示（参见《Number 对象》一节）。

```js
Number.MAX_VALUE // 1.7976931348623157e+308
Number.MIN_VALUE // 5e-324
```

## 数值的表示法

JavaScript 的数值有多种表示方法，可以用字面形式直接表示，也可以采用科学计数法表示，下面是两个科学计数法的例子。

```js
123e3 // 123000
123e-3 // 0.123
```

以下两种情况，JavaScript 会自动将数值转为科学计数法表示，其他情况都采用字面形式直接表示。

（1）小数点前的数字多于 21 位。

```js
1234567890123456789012
// 1.2345678901234568e+21

123456789012345678901
// 123456789012345680000
```

（2）小数点后的零多于 5 个。

```js
0.0000003 // 3e-7
0.000003 // 0.000003
```

正常情况下，所有数值都为十进制。如果要表示十六进制的数，必须以 0x 或 0X 开头，比如十进制的 255 等于十六进制的 0xff 或 0Xff。如果要表示八进制数，必须以 0 开头，比如十进制的 255 等于八进制的 0377。由于八进制表示法的前置 0，在处理时很容易造成混乱，有时为了区分一个数到底是八进制还是十进制，会增加很大的麻烦，所以建议不要使用这种表示法。

## 特殊数值

JavaScript 提供几个特殊的数值。

### 正零和负零

严格来说，JavaScript 提供零的三种写法：0、+0、-0。它们是等价的。

```js
-0 === +0 // true
0 === -0 // true
0 === +0 // true
```

但是，如果正零和负零分别当作分母，它们返回的值是不相等的。

```js
(1/+0) === (1/-0) // false
```

上面代码之所以出现这样结果，是因为除以正零得到+Infinity，除以负零得到-Infinity，这两者是不相等的（关于 Infinity 详见后文）。

### NaN

（1）含义

NaN 是 JavaScript 的特殊值，表示“非数字”（Not a Number），主要出现在将字符串解析成数字出错的场合。

```js
5 - 'x'
// NaN
```

上面代码运行时，会自动将字符串“x”转为数值，但是由于 x 不是数字，所以最后得到结果为 NaN，表示它是“非数字”（NaN）。

另外，一些数学函数的运算结果会出现 NaN。

```js
Math.acos(2) // NaN
Math.log(-1) // NaN
Math.sqrt(-1) // NaN
```

0 除以 0 也会得到 NaN。

```js
0 / 0 // NaN
```

需要注意的是，NaN 不是一种独立的数据类型，而是一种特殊数值，它的数据类型依然属于 Number，使用 typeof 运算符可以看得很清楚。

```js
typeof NaN // 'number'
```

（2）运算规则

NaN 不等于任何值，包括它本身。

```js
NaN === NaN // false
```

由于数组的 indexOf 方法，内部使用的是严格相等运算符，所以该方法对 NaN 不成立。

```js
[NaN].indexOf(NaN) // -1
```

NaN 在布尔运算时被当作 false。

```js
Boolean(NaN) // false
```

NaN 与任何数（包括它自己）的运算，得到的都是 NaN。

```js
NaN + 32 // NaN
NaN - 32 // NaN
NaN * 32 // NaN
NaN / 32 // NaN
```

（3）判断 NaN 的方法

isNaN 方法可以用来判断一个值是否为 NaN。

```js
isNaN(NaN) // true
isNaN(123) // false
```

但是，isNaN 只对数值有效，如果传入其他值，会被先转成数值。比如，传入字符串的时候，字符串会被先转成 NaN，所以最后返回 true，这一点要特别引起注意。也就是说，isNaN 为 true 的值，有可能不是 NaN，而是一个字符串。

```js
isNaN("Hello") // true
// 相当于
isNaN(Number("Hello")) // true
```

出于同样的原因，对于数组和对象，isNaN 也返回 true。

```js
isNaN({}) // true
isNaN(Number({})) // true

isNaN(["xzy"]) // true
isNaN(Number(["xzy"])) // true
```

因此，使用 isNaN 之前，最好判断一下数据类型。

```js
function myIsNaN(value) {
    return typeof value === 'number' && isNaN(value);
}
```

判断 NaN 更可靠的方法是，利用 NaN 是 JavaScript 之中唯一不等于自身的值这个特点，进行判断。

```js
function myIsNaN(value) {
    return value !== value;
}
```

### Infinity

（1）定义

Infinity 表示“无穷”。除了 0 除以 0 得到 NaN，其他任意数除以 0，得到 Infinity。

```js
1 / -0 // -Infinity
1 / +0 // Infinity
```

上面代码表示，非 0 值除以 0，JavaScript 不报错，而是返回 Infinity。这是需要特别注意的地方。

Infinity 有正负之分。

```js
Infinity === -Infinity // false
Math.pow(+0, -1) // Infinity
Math.pow(-0, -1) // -Infinity
```

运算结果超出 JavaScript 可接受范围，也会返回无穷。

```js
Math.pow(2, 2048) // Infinity
-Math.pow(2, 2048) // -Infinity
```

由于数值正向溢出（overflow）、负向溢出（underflow）和被 0 除，JavaScript 都不报错，所以单纯的数学运算几乎没有可能抛出错误。

（2）运算规则

Infinity 的四则运算，符合无穷的数学计算规则。

```js
Infinity + Infinity // Infinity
5 * Infinity // Infinity
5 - Infinity // -Infinity
Infinity / 5 // Infinity
5 / Infinity // 0
```

Infinity 减去或除以 Infinity，得到 NaN。

```js
Infinity - Infinity // NaN
Infinity / Infinity // NaN
```

Infinity 可以用于布尔运算。可以记住，Infinity 是 JavaScript 中最大的值（NaN 除外），-Infinity 是最小的值（NaN 除外）。

```js
5 > -Infinity // true
5 > Infinity // false
```

（3）isFinite 函数

isFinite 函数返回一个布尔值，检查某个值是否为正常值，而不是 Infinity。

```js
isFinite(Infinity) // false
isFinite(-1) // true
isFinite(true) // true
isFinite(NaN) // false
```

上面代码表示，如果对 NaN 使用 isFinite 函数，也返回 false，表示 NaN 不是一个正常值。

## 与数值相关的全局方法

### parseInt 方法

parseInt 方法可以将字符串或小数转化为整数。如果字符串头部有空格，空格会被自动去除。

```js
parseInt("123") // 123
parseInt(1.23) // 1
parseInt('   81') // 81
```

如果字符串包含不能转化为数字的字符，则不再进行转化，返回已经转好的部分。

```js
parseInt("8a") // 8
parseInt("12**") // 12
parseInt("12.34") // 12
```

如果字符串的第一个字符不能转化为数字（正负号除外），返回 NaN。

```js
parseInt("abc") // NaN
parseInt(".3") // NaN
parseInt("") // NaN
```

parseInt 方法还可以接受第二个参数（2 到 36 之间），表示被解析的值的进制。

```js
parseInt(1000, 2) // 8
parseInt(1000, 6) // 216
parseInt(1000, 8) // 512
```

这意味着，可以用 parseInt 方法进行进制的转换。

```js
parseInt("1011", 2) // 11
```

需要注意的是，如果第一个参数是数值，则会将这个数值先转为 10 进制，然后再应用第二个参数。

```js
parseInt(010, 10) // 8
parseInt(010, 8) // NaN
parseInt(020, 10) // 16
parseInt(020, 8) // 14
```

上面代码中，010 会被先转为十进制 8，然后再应用第二个参数，由于八进制中没有 8 这个数字，所以 parseInt(010, 8)会返回 NaN。同理，020 会被先转为十进制 16，然后再应用第二个参数。

如果第一个参数是字符串，则不会将其先转为十进制。

```js
parseInt("010") // 10
parseInt("010",10) // 10
parseInt("010",8) // 8
```

可以看到，parseInt 的很多复杂行为，都是由八进制的前缀 0 引发的。因此，ECMAScript 5 不再允许 parseInt 将带有前缀 0 的数字，视为八进制数。但是，为了保证兼容性，大部分浏览器并没有部署这一条规定。

另外，对于那些会自动转为科学计数法的数字，parseInt 会出现一些奇怪的错误。

```js
parseInt(1000000000000000000000.5, 10) // 1
// 等同于
parseInt('1e+21', 10) // 1

parseInt(0.0000008, 10) // 8
// 等同于
parseInt('8e-7', 10) // 8
```

### parseFloat 方法

parseFloat 方法用于将一个字符串转为浮点数。

如果字符串包含不能转化为浮点数的字符，则不再进行转化，返回已经转好的部分。

```js
parseFloat("3.14");
parseFloat("314e-2");
parseFloat("0.0314E+2");
parseFloat("3.14more non-digit characters");
```

上面四个表达式都返回 3.14。

parseFloat 方法会自动过滤字符串前导的空格。

```js
parseFloat("\t\v\r12.34\n ")
// 12.34
```

如果第一个字符不能转化为浮点数，则返回 NaN。

```js
parseFloat("FF2") // NaN
parseFloat("") // NaN
```

上面代码说明，parseFloat 将空字符串转为 NaN。

这使得 parseFloat 的转换结果不同于 Number 函数。

```js
parseFloat(true)  // NaN
Number(true) // 1

parseFloat(null) // NaN
Number(null) // 0

parseFloat('') // NaN
Number('') // 0

parseFloat('123.45#') // 123.45
Number('123.45#') // NaN
```

## 参考链接

*   Dr. Axel Rauschmayer, [How numbers are encoded in JavaScript](http://www.2ality.com/2012/04/number-encoding.html)
*   Humphry, [JavaScript 中 Number 的一些表示上/下限](http://blog.segmentfault.com/humphry/1190000000407658)