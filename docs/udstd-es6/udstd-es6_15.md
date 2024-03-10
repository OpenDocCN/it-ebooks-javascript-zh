# 附录 A：较小的改进

## 附录 A：较小的改进

除了本书已涵盖的主要变化之外， ES6 还做出了一些虽小但仍然有助于改进 JS 的变更，包括：使整型更易用、新增计算方法、对 Unicode 标识符的细微调整，以及规范化 `__proto__` 属性。我会在本附录中描述所有这些内容。

*   处理整型
    *   识别整型
    *   安全的整型
*   新的数学方法
*   Unicode 标识符
*   规范化的 `__proto__` 属性

### 处理整型

JS 使用 IEEE 754 编码系统来表示整型与浮点型，多年以来这引发了很多混乱。虽然这门语言煞费苦心地确保开发者不需要关心数值的编码细节，但问题仍然会时不时泄露出来。 ES6 让整型变得更易识别、更易处理，力图解决这方面的问题。

#### 识别整型

首先， ES6 新增了 `Number.isInteger()` 方法，用于判断一个值是否能在 JS 中表示整型。虽然 JS 使用了 IEEE 754 来同时表示浮点型与整型这两种数值，但它们的存储方式仍有差异。 `Number.isInteger()` 方法利用了这个差异，当使用一个值来调用此方法时， JS 引擎会查看该值的底层表示以判断它是不是一个整型。这意味着看起来像浮点型的数值实际上可能被存储为整型，此时 `Number.isInteger()` 便会返回 `true` 。例如：

```js
console.log(Number.isInteger(25));      // true
console.log(Number.isInteger(25.0));    // true
console.log(Number.isInteger(25.1));    // false 
```

在此代码中，向 `Number.isInteger()` 传入 `25` 和 `25.0` 都会返回 `true` ，尽管后者看起来是浮点型。在 JS 中，单纯添加一个小数点并不会让数字自动变为浮点型。由于 `25.0` 实际上就是 `25` ，它就被存储为整型；而数值 `25.1` 则会被存储为浮点型，因为它拥有小数部分。

#### 安全的整型

IEEE 754 只能精确表示 -2⁵³ 与 2⁵³ 之间的整型数，在该“安全”范围之外，多个不同的数值就有可能对应同一个二进制表示。这意味着 JS 只能在 IEEE 754 的精确范围内保证对整型数的安全表示。例如，研究以下例子：

```js
console.log(Math.pow(2, 53));      // 9007199254740992
console.log(Math.pow(2, 53) + 1);  // 9007199254740992 
```

此例并不包含拼写错误，然而两个不同的数值却被表示成了同一个 JS 整型数。当数值超出安全范围越远，此效果就越加明显。

ES6 引进了 `Number.isSafeInteger()` 以便更好识别该语言所能精确表示的整型；同时新增的还有 `Number.MAX_SAFE_INTEGER` 与 `Number.MIN_SAFE_INTEGER` 属性，分别用于表示整型数的上下边界。 `Number.isSafeInteger()` 方法能确认一个值是整型、并且它落在安全范围之内，正如此例：

```js
var inside = Number.MAX_SAFE_INTEGER,
    outside = inside + 1;

console.log(Number.isInteger(inside));          // true
console.log(Number.isSafeInteger(inside));      // true

console.log(Number.isInteger(outside));         // true
console.log(Number.isSafeInteger(outside));     // false 
```

数值 `inside` 是最大的安全整型数，因此用它去调用 `Number.isInteger()` 与 `Number.isSafeInteger()` 都会返回 `true` 。数值 `outside` 则是第一个可疑的整型值，尽管它依然是个整型数，但仍被认为是不安全的。

多数情况下，你只想用安全的整型数在 JS 中去进行整型运算或比较，因此使用 `Number.isSafeInteger()` 作为输入验证的一部分便是个好主意。

### 新的数学方法

ES6 的游戏与图形的新重点引导它将类型化数组（ typed array ）引入了 JS ，同时也让它意识到 JS 引擎应当更有效率地进行许多数学计算。但诸如 asm.js （工作在 JS 的一个子集上以提高效率）之类的优化策略，都需要更多的信息以便尽可能快地进行计算。例如，知道数值是被作为 32 位整型还是 64 位浮点型来处理，对基于硬件的操作来说是非常重要的，而这要比基于软件的操作快得多。

因此， ES6 给 `Math` 对象新增了几个方法来提高通用数学计算的速度，而提高通用计算速度也能让需要进行许多计算的应用（例如图形程序）提高总体速度。下列就是这些新方法：

*   `Math.acosh(x)` ：返回 x 的反双曲余弦值；
*   `Math.asinh(x)` ：返回 x 的反双曲正弦值；
*   `Math.atanh(x)` ：返回 x 的反双曲正切值；
*   `Math.cbrt(x)` ：返回 x 的立方根；
*   `Math.clz32(x)` ：返回 x 的 32 位整型二进制表达形式起始处 0 的个数；
*   `Math.cosh(x)` ：返回 x 的双曲余弦值；
*   `Math.expm1(x)` ：返回 e^x - 1 的值；
*   `Math.fround(x)` ：返回最接近 x 的单精度浮点数；
*   `Math.hypot(...values)` ：返回参数平方和的平方根；
*   `Math.imul(x, y)` ：返回两个参数真正的 32 位乘法运算结果；
*   `Math.log1p(x)` ：返回 1 + x 的自然对数；
*   `Math.log10(x)` ：返回 x 的常用对数（即以 10 为底）；
*   `Math.log2(x)` ：返回 x 的二进制对数（即以 2 为底）；
*   `Math.sign(x)` ： x 为负数时返回 -1 ， +0 与 -0 返回 0 ，正数则返回 1 ；
*   `Math.sinh(x)` ：返回 x 的双曲正弦值；
*   `Math.tanh(x)` ：返回 x 的双曲正切值；
*   `Math.trunc(x)` ：移除浮点型数值小数点后的数字，以返回一个整型值。

解释每个新方法以及它的细节已经超出了本书的范围。不过若你的应用需要合理地进行通用计算，在你自行实现它之前，请先确保检查是否已有对应的 `Math` 新方法。

### Unicode 标识符

ES6 提供了比之前版本更好的 Unicode 支持，同时也修改了能被用于标识符的字符范围。在 ES5 中已经能在标识符里使用 Unicode 转义序列，例如：

```js
// 在 ES5 与 ES6 中都有效
var \u0061 = "abc";

console.log(\u0061);     // "abc"

// 等价于：
console.log(a);          // "abc" 
```

在此例的 `var` 语句之后，你用 `\u0061` 或 `a` 都能访问这个变量。在 ES6 中，你还能在标识符里使用 Unicode 代码点转义序列，就像这样：

```js
// 在 ES5 与 ES6 中都有效
var \u{61} = "abc";

console.log(\u{61});      // "abc"

// 等价于：
console.log(a);          // "abc" 
```

本例只是将 `\u0061` 替换为它的代码点等价形式，然而，这么做的效果实际上与上个例子完全相同。

另外， ES6 还将 Unicode 标准附录 31 中的字符正式指定为有效的标识符（该附录详见 [Unicode Standard Annex #31: Unicode Identifier and Pattern Syntax](http://unicode.org/reports/tr31/)），其规则如下：

1.  第一个字符必须是 `$` 、 `_` ，或任何属于 `ID_start` 核心衍生属性的 Unicode 符号；
2.  之后的字符必须为 `$` 、 `_` 、 `\u200c` （零宽不连字）、 `\u200d` （零宽连字），或任何属于 `ID_Continue` 核心衍生属性的 Unicode 符号。

`ID_Start` 与 `ID_Continue` 核心衍生属性由 Unicode 标识符与模式语法（即上述附录）定义，提供了一种方法以识别能被用于标识符（如变量与域名）的合适符号。该规范并未针对 JS 。

### 规范化的 `__proto__` 属性

在 ES5 规范完成之前，几个 JS 引擎就已经实现了一个称为 `__proto__` 的自定义属性，能用它来获取并设置 `[[Prototype]]` 属性。实际上， `__proto__` 就是 `Object.getPrototypeOf()` 与 `Object.setPrototypeOf()` 方法的早期先驱。期望所有的 JS 引擎都移除这个属性是不现实的（因为有些流行的 JS 代码库已经利用了该属性），因此 ES6 也将该属性的行为标准化了，但在 ECMA-262 附录 B 中该规范也附带了以下警告：

> 这些特性并不被认为是 ES 语言的核心部分，程序员在书写新的 ES 代码时，不应使用它、或假定这些特性存在。 ES 的实现方案并不鼓励实现这些特性，除非该实现已是 web 浏览器的一部分、或者被用于在浏览器中运行遗留代码。

ES 规范更推荐使用 `Object.getPrototypeOf()` 与 `Object.setPrototypeOf()` 方法，因为 `__proto__` 具有如下特征：

1.  `__proto__` 在对象字面量中只能指定一次，指定多个 `__proto__` 将会抛出错误。这也是对象字面量属性中唯一受此限制的属性。
2.  对象字面量中需计算形式的 `["__proto__"]` 表现得就像是常规属性，并不会设置或返回当前对象的原型。对于字面量属性来说，需计算形式与非计算形式一般是等价的，只有 `__proto__` 例外。

> 译注：这代表以下两种写法并不等价——
> 
> ```js
> let a = {
>    ["__proto__"]: Number
> }; 
> ```
> 
> 以及
> 
> ```js
> let a = {
>    __proto__: Number
> }; 
> ```
> 
> 后者可以将 `a` 的原型设置为 `Number` ，而前者对 `a` 的原型没有造成任何影响。

你应当规避 `__proto__` 属性，不过规范文档定义它的方式却很有意思。在 ES6 引擎中， `Object.prototype.__proto__` 被定义为一个访问器属性，其 `get` 方法会调用 `Object.getPrototypeOf()` ，而 `set` 方法则会调用 `Object.setPrototypeOf()` 。这样在使用 `__proto__` 与使用 `Object.getPrototypeOf()` / `Object.setPrototypeOf()` 之间就几乎没有真正区别，唯一例外是 `__proto__` 能在对象字面量中直接使用，用于设置对象的原型。以下是使用它的范例：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

let dog = {
    getGreeting() {
        return "Woof";
    }
};

// 原型设为 person
let friend = {
    __proto__: person
};
console.log(friend.getGreeting());                      // "Hello"
console.log(Object.getPrototypeOf(friend) === person);  // true
console.log(friend.__proto__ === person);               // true

// 将 prototype 改为 dog
friend.__proto__ = dog;
console.log(friend.getGreeting());                      // "Woof"
console.log(friend.__proto__ === dog);                  // true
console.log(Object.getPrototypeOf(friend) === dog);     // true 
```

此例未调用 `Object.create()` 来创建 `friend` 对象，而是创建了一个标准的对象字面量，并将一个值（ `person` ）赋给了 `__proto__` 属性。而另一方面，当使用 `Object.create()` 方式时，你需要为对象的任意附加属性指定完整的属性描述符。