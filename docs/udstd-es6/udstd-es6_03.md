# 第二章 字符串与正则表达式

## 第二章 字符串与正则表达式

字符串可以说是编程中最重要的数据类型之一。它们在几乎所有高级语言中存在，并且能有效使用它们也是开发者编写程序的基础。作为扩展的正则表达式也十分重要，因为它们给了开发者操纵字符串额外的能力。考虑到这些事实， ES6 的创造者加强了字符串与正则表达式，为它们添加了新的能力，并补充了一些长期缺失的功能。本章会介绍这些变化。

*   更好的 Unicode 支持
    *   UTF-16 代码点
    *   codePointAt() 方法
    *   String.fromCodePoint() 方法
    *   normalize() 方法
    *   正则表达式 u 标志
        *   u 标志如何运作
        *   计算代码点数量
        *   判断是否支持 u 标志
*   字符串的其他改动
    *   识别子字符串的方法
    *   repeat() 方法
*   正则表达式的其他改动
    *   正则表达式 y 标志
    *   复制正则表达式
    *   flags 属性
*   模板字面量
    *   基本语法
    *   多行字符串
        *   ES6 之前的权宜之计
        *   多行字符串的简单解决方法
    *   制造替换位
    *   标签化模板
        *   定义标签
        *   使用模板字面量中的原始值
*   总结

### 更好的 Unicode 支持

在 ES6 之前， JS 的字符串以 16 位字符编码（ UCS-2 ）为基础。每个 16 位序列都是一个**码元**（ **code unit** ），用于表示一个字符。字符串所有的属性与方法（像是 `length` 属性与 `charAt()` 方法）都是基于 16 位的码元。当然， 16 位曾经足以容纳任何字符，然而由于 Unicode 引入了扩展字符集，这就不再够用了。

> 译注：此段第一句话的原文是：
> 
> > Before ES6, JavaScript strings revolved around 16-bit character encoding (UTF-16).
> 
> 然而 UTF-16 是变长的字符编码方式，有 16 位与 32 位两种情况。 JS 原先使用的则是固定 16 位（双字节）的字符编码方式，即 UCS-2 。

#### UTF-16 代码点

Unicode 的明确目标是给世界上所有的字符提供全局唯一标识符，而 16 位的字符长度限制已不能满足这种需求。这些全球唯一标识符被称为**代码点**（ **code points** ），是从 0 开始的简单数字。代码点是如你想象的字符代码那样，用一个数字来代表一个字符。字符编码要求将代码点转换为内部一致的码元，而对于 UTF-16 来说，代码点可以由多个码元组成。

在 UTF-16 中的第一个 2¹⁶ 代码点表示单个 16 位码元，这个范围被称为**多语言基本平面**（ **Basic Multilingual Plane** ， BMP ）。任何超出该范围的代码点都不能用单个 16 位码元表示，而是会落在**扩展平面**（ **supplementary planes** ）内。 UTF-16 引入了**代理对**（ **surrogate pairs** ）来解决这个问题，允许使用两个 16 位码元来表示单个代码点。这意味着字符串内的任意单个字符都可以用一个码元（共 16 位）或两个码元（共 32 位）来表示，前者对应基本平面字符，而后者对应扩展平面字符。

在 ES5 中，所有字符串操作都基于 16 位码元，这表示在处理包含代理对的 UTF-16 字符时会出现预期外的结果，就像这个例子：

```
var text = "𠮷";

console.log(text.length);           // 2
console.log(/^.$/.test(text));      // false
console.log(text.charAt(0));        // ""
console.log(text.charAt(1));        // ""
console.log(text.charCodeAt(0));    // 55362
console.log(text.charCodeAt(1));    // 57271 
```

这个 Unicode 字符 `"𠮷"` 使用了代理对，因此，上面的 JS 字符串操作会将该字符串当作两个 16 位字符来对待，这意味着：

*   `text` 的长度属性值是 2 ，而不是应有的 1 。
*   意图匹配单个字符的正则表达式匹配失败了，因为它认为这里有两个字符。
*   `charAt()` 方法无法返回一个有效的字符，因为这里每 16 位代码点都不是一个可打印字符。

`charCodeAt()` 方法同样无法正确识别该字符，它只能返回每个码元的 16 位数字，但在 ES5 中，这已经是对 `text` 变量所能获取到的最精确的值了。

另一方面，ES6 要求这类 UTF-16 字符的编码问题必须得到解决。基于这种字符编码的字符串操作的标准化，也就意味着 JS 可以支持针对代理对的专门功能设计。本章接下来的部分会讨论与此有关的几个关键案例。

#### codePointAt() 方法

ES6 为全面支持 UTF-16 而新增的方法之一是 `codePointAt()` ，它可以在给定字符串中按位置提取 Unicode 代码点。该方法接受的是码元位置而非字符位置，并返回一个整数值，就像下面的 `console.log()` 范例所展示的：

```
var text = "𠮷a";

console.log(text.charCodeAt(0));    // 55362
console.log(text.charCodeAt(1));    // 57271
console.log(text.charCodeAt(2));    // 97

console.log(text.codePointAt(0));   // 134071
console.log(text.codePointAt(1));   // 57271
console.log(text.codePointAt(2));   // 97 
```js

`codePointAt()` 方法的返回值一般与 `charCodeAt()` 相同，除非操作对象并不是 BMP 字符。 `text` 字符串的第一个字符不是 BMP 字符，因此它占用了两个码元，意味着该字符串的 `length` 属性是 3 而不是 2 。 `charCodeAt()` 方法只返回了位置 0 的第一个码元；而 `codePointAt()` 返回的是完整的代码点，即使它占用了多个码元。对于位置 1 （第一个字符的第二个码元）和位置 2 （ `"a"` 字符）来说，两个方法返回的值则是相同的。

判断字符包含了一个还是两个码元，对该字符调用 `codePointAt()` 方法就是最简单的方法。可以照下面的函数这么写：

```
function is32Bit(c) {
    return c.codePointAt(0) > 0xFFFF;
}

console.log(is32Bit("𠮷"));         // true
console.log(is32Bit("a"));          // false 
```js

16 位字符的上边界用十六进制表示就是 `FFFF` ，因此任何大于该数字的代码点必须用两个码元（共 32 位）来表示。

#### String.fromCodePoint() 方法

当 ECMAScript 提供了某种方法时，它一般也会给出方法来处理相反的操作。你可以使用 `codePointAt()` 来提取字符串内中某个字符的代码点，也可以借助 `String.fromCodePoint()` 用给定的代码点来产生包含单个字符的字符串。例如：

```
console.log(String.fromCodePoint(134071));  // "𠮷" 
```js

可以将 `String.fromCodePoint()` 视为 `String.fromCharCode()` 的完善版本。两者处理 BMP 字符时会返回相同结果，只有处理 BMP 范围之外的字符时才会有差异。

#### normalize() 方法

Unicode 另一个有趣之处是，不同的字符在排序或其它一些比较操作中可能会被认为是相同的。有两种方式可以定义这种关联性：第一种是**规范相等性**（ **canonical equivalence** ），意味着两个代码点序列在所有方面都被认为是可互换的。例如，两个字符的组合可以按规范等同于另一个字符。第二种关联性是**兼容性**（ **compatibility** ），两个兼容的代码点序列看起来有差别，但在特定条件下可互换使用。

由于这些关联性，文本内容在根本上相同的两个字符串就可以包含不同的代码点序列。例如，字符 `"æ"` 与双字符的字符串 `"ae"` 或许能互换使用，但它们并不严格相等，除非使用某种手段来标准化。

ES6 给字符串提供了 `normalize()` 方法，以支持 Unicode 标准形式。该方法接受单个可选的字符串参数，用于指示需要使用下列哪种 Unicode 标准形式：

*   Normalization Form Canonical Composition (`"NFC"`)，这是默认值；
*   Normalization Form Canonical Decomposition (`"NFD"`)；
*   Normalization Form Compatibility Composition (`"NFKC"`)；
*   Normalization Form Compatibility Decomposition (`"NFKD"`)。

解释这四种形式的差异超出了本书的范围。只需记住，当比较字符串时，它们必须被标准化为同一种形式。例如：

```
var normalized = values.map(function(text) {
    return text.normalize();
});

normalized.sort(function(first, second) {
    if (first < second) {
        return -1;
    } else if (first === second) {
        return 0;
    } else {
        return 1;
    }
}); 
```js

此代码将 `values` 数组中的字符串转换为一种标准形式，以便让转换后的数组可以被正确排序。你也可以在比较过程中调用 `normalize()` 来对原始数组进行排序。如下所示：

```
values.sort(function(first, second) {
    var firstNormalized = first.normalize(),
        secondNormalized = second.normalize();

    if (firstNormalized < secondNormalized) {
        return -1;
    } else if (firstNormalized === secondNormalized) {
        return 0;
    } else {
        return 1;
    }
}); 
```js

关于此代码最需要重点注意的是： `first` 与 `second` 再一次使用同一方式被标准化了。这两个例子使用了默认值（即 NFC ），不过你还能轻易指定其他任意一种，就像这样：

```
values.sort(function(first, second) {
    var firstNormalized = first.normalize("NFD"),
        secondNormalized = second.normalize("NFD");

    if (firstNormalized < secondNormalized) {
        return -1;
    } else if (firstNormalized === secondNormalized) {
        return 0;
    } else {
        return 1;
    }
}); 
```js

如果你之前从未担心过 Unicode 标准化方面的问题，那么可能暂时还不太会用到这个方法。然而若你曾经开发过国际化的应用，你就一定会发现 `normalize()` 方法非常有用。

新方法并不是 ES6 为 Unicode 字符串提供的唯一改进，它还新增了两个有用的语法要素。

#### 正则表达式 u 标志

你可以使用正则表达式来完成字符串的很多通用操作。但要记住，正则表达式假定单个字符使用一个 16 位的码元来表示。为了解决这个问题， ES6 为正则表达式定义了用于处理 Unicode 的 `u` 标志。

##### u 标志如何运作

当一个正则表达式设置了 u 标志时，它的工作模式将切换到针对字符，而不是针对码元。这意味着正则表达式将不会被字符串中的代理对所混淆，而是会如预期那样工作。例如，研究以下代码：

```
var text = "𠮷";

console.log(text.length);           // 2
console.log(/^.$/.test(text));      // false
console.log(/^.$/u.test(text));     // true 
```js

正则表达式 `/^.$/` 会匹配只包含单个字符的任意输入字符串。当不使用 `u` 标志时，该正则表达式只匹配码元，所以不能匹配由两个码元表示的这个日文字符。启用 `u` 标志后，正则表达式就会比较字符而不是码元，所以这个日文字符就会被匹配到。

##### 计算代码点数量

可惜的是， ES6 并没有添加方法用于判断一个字符串包含多少个代码点，但借助 `u` 标志，你就可以使用正则表达式来进行计算，如下所示：

```
function codePointLength(text) {
    var result = text.match(/[\s\S]/gu);
    return result ? result.length : 0;
}

console.log(codePointLength("abc"));    // 3
console.log(codePointLength("𠮷bc"));   // 3 
```

此例调用了 `match()` 方法来检查 `text` 中的空白字符与非空白字符（使用 `[\s\S]` 以确保该模式能匹配换行符），所用的正则表达式启用了全局与 Unicode 特性。在匹配至少成功一次的情况下， `result` 变量会是包含匹配结果的数组，因此该数组的长度就是字符串中代码点的数量。在 Unicode 中，字符串 `"abc"` 与 `"𠮷bc"` 同样包含三个字符，所以数组长度为 3 。

> 虽然这种方法可用，但它并不快，尤其在操作长字符串时。你也可以使用字符串的迭代器（详见第八章）来达到相同目的。一般来说，只要有可能就应尽量减少对代码点数量的计算。

##### 判断是否支持 u 标志

既然 `u` 标志是一项语法变更，在不兼容 ES6 的 JS 引擎中试图使用它就会抛出语法错误。使用一个函数来判断是否支持 `u` 标志是最安全的方式，像这样：

```js
function hasRegExpU() {
    try {
        var pattern = new RegExp(".", "u");
        return true;
    } catch (ex) {
        return false;
    }
} 
```

此函数将 `u` 作为一个参数来调用 `RegExp` 构造器，该语法即使在旧版 JS 引擎中都是有效的，而构造器在 `u` 未被支持的情况下会抛出错误。

> 若你的代码仍然需要在旧版 JS 引擎中工作，那么在使用 `u` 标志时应当始终使用 `RegExp` 构造器。这会防止语法错误，并允许你有选择地检测并使用 `u` 标志，而不会导致执行被中断。

### 字符串的其他改动

JS 字符串的特性总是落后于其它语言，例如，直到 ES5 中字符串才获得了 `trim()` 方法。而 ES6 则继续添加新功能以扩展 JS 解析字符串的能力。

#### 识别子字符串的方法

自从 JS 引入了 indexOf() 方法，开发者们就使用它来识别字符串是否存在于其它字符串中。 ES6 包含了以下三个方法来满足这类需求：

*   `includes()` 方法，在给定文本存在于字符串中的任意位置时会返回 true ，否则返回 false ；
*   `startsWith()` 方法，在给定文本出现在字符串起始处时返回 true ，否则返回 false ；
*   `endsWith()` 方法，在给定文本出现在字符串结尾处时返回 true ，否则返回 false 。

每个方法都接受两个参数：需要搜索的文本，以及可选的搜索起始位置索引。当提供了第二个参数时， `includes()` 与 `startsWith()` 方法会从该索引位置开始尝试匹配；而 `endsWith()` 方法会将字符串长度减去该参数，以此为起点开始尝试匹配。当第二个参数未提供时， `includes()` 与 `startsWith()` 方法会从字符串起始处开始查找，而 `endsWith()` 方法则从尾部开始。实际上，第二个参数减少了搜索字符串的次数。以下是使用这些方法的演示：

```js
var msg = "Hello world!";

console.log(msg.startsWith("Hello"));       // true
console.log(msg.endsWith("!"));             // true
console.log(msg.includes("o"));             // true

console.log(msg.startsWith("o"));           // false
console.log(msg.endsWith("world!"));        // true
console.log(msg.includes("x"));             // false

console.log(msg.startsWith("o", 4));        // true
console.log(msg.endsWith("o", 8));          // true
console.log(msg.includes("o", 8));          // false 
```

前三次调用没有使用第二个参数，因此它们在必要情况下会搜索整个字符串。最后三次调用只检查了字符串的一部分：调用 `msg.startsWith("o", 4)` 从 `msg` 字符串的索引位置 4 （即 `"Hello"` 中的 `"o"` ）开始尝试匹配；调用 `msg.endsWith("o", 8)` 也从位置 4 开始搜索，因为参数 `8` 会从字符串的长度值（ 12 ）中被减去；而调用 `msg.includes("o", 8)` 则从索引位置 8 开始尝试匹配，也就是 `"world"` 中的 `"r"` 。

虽然这三个方法使得判断子字符串是否存在变得更容易，但它们只返回了一个布尔值。若你需要找到它们在另一个字符串中的确切位置，则需要使用 `indexOf()` 和 `lastIndexOf()` 。

> 如果向 `startsWith()` 、 `endsWith()` 或 `includes()` 方法传入了正则表达式而不是字符串，会抛出错误。这与 `indexOf()` 以及 `lastIndexOf()` 方法的表现形成了反差，它们会将正则表达式转换为字符串并搜索它。

#### repeat() 方法

ES6 还为字符串添加了一个 `repeat()` 方法，它接受一个参数作为字符串的重复次数，返回一个将初始字符串重复指定次数的新字符串。例如：

```js
console.log("x".repeat(3));         // "xxx"
console.log("hello".repeat(2));     // "hellohello"
console.log("abc".repeat(4));       // "abcabcabcabc" 
```

此方法比相同目的的其余方法更加方便，在操纵文本时特别有用，尤其是在需要产生缩进的代码格式化工具中，像这样：

```js
// indent 使用了一定数量的空格
var indent = " ".repeat(4),
    indentLevel = 0;

// 每当你增加缩进
var newIndent = indent.repeat(++indentLevel); 
```

第一次调用 `repeat()` 创建了一个包含四个空格的字符串，而 `indentLevel` 变量会持续追踪缩进的级别。此后，你可以仅通过增加 `indentLevel` 的值来调用 repeat() 方法，便可以改变空格数量。

ES6 也为正则表达式的功能进行了一些有益改动，这不适合纳入某个特定章节，因此集中在下一节着重介绍。

### 正则表达式的其他改动

正则表达式是在 JS 中操作字符串的重要方面之一，与该语言的其他方面相似，它在以往的版本中并未有太多改变。不过，为了配合字符串的更新， ES6 也对正则表达式进行了一些改进。

#### 正则表达式 y 标志

在 Firefox 实现了对正则表达式 `y` 标志的专有扩展之后，ES6 将该实现标准化。 `y` 标志影响正则表达式搜索时的粘连（ `sticky` ）属性，它表示从正则表达式的 `lastIndex` 属性值的位置开始检索字符串中的匹配字符。如果在该位置没有匹配成功，那么正则表达式将停止检索。为了明白它是如何工作的，考虑如下的代码：

```js
var text = "hello1 hello2 hello3",
    pattern = /hello\d\s?/,
    result = pattern.exec(text),
    globalPattern = /hello\d\s?/g,
    globalResult = globalPattern.exec(text),
    stickyPattern = /hello\d\s?/y,
    stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello1 "
console.log(stickyResult[0]);   // "hello1 "

pattern.lastIndex = 1;
globalPattern.lastIndex = 1;
stickyPattern.lastIndex = 1;

result = pattern.exec(text);
globalResult = globalPattern.exec(text);
stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello2 "
console.log(stickyResult[0]);   // Error! stickyResult is null 
```

此例中有三个正则表达式： `pattern` 中的表达式没有使用任何标志， `globalPattern` 使用了 `g` 标志， `stickyPattern` 则使用了 `y` 标志。对 `console.log()` 的第一次调用，三个正则表达式分别都返回了 `"hello1 "` ，此字符串尾部有个空格。

此后，三个模式的 `lastIndex` 属性全部被更改为 1 ，表示三个模式的正则表达式都应当从第二个字符开始尝试匹配。不使用任何标志的正则表达式完全忽略了对于 `lastIndex` 的更改，仍然毫无意外地匹配了 `"hello1 "` ；而使用 `g` 标志的正则表达式继续匹配了 `"hello2 "` ，因为它从第二个字符（ `"e"` ）开始，持续向着字符串尾部方向搜索；粘连的正则表达式则在第二个字符处没有匹配成功，因此 `stickyResult` 的值是 `null` 。

一旦匹配操作成功，粘连标志就会将匹配结果之后的那个字符的索引值保存在 `lastIndex` 中；若匹配未成功，那么 `lastIndex` 的值将重置为 0 。全局标志的行为与其相同，如下所示：

```js
var text = "hello1 hello2 hello3",
    pattern = /hello\d\s?/,
    result = pattern.exec(text),
    globalPattern = /hello\d\s?/g,
    globalResult = globalPattern.exec(text),
    stickyPattern = /hello\d\s?/y,
    stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello1 "
console.log(stickyResult[0]);   // "hello1 "

console.log(pattern.lastIndex);         // 0
console.log(globalPattern.lastIndex);   // 7
console.log(stickyPattern.lastIndex);   // 7

result = pattern.exec(text);
globalResult = globalPattern.exec(text);
stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello2 "
console.log(stickyResult[0]);   // "hello2 "

console.log(pattern.lastIndex);         // 0
console.log(globalPattern.lastIndex);   // 14
console.log(stickyPattern.lastIndex);   // 14 
```

对于 `stickyPattern` 和 `globalPattern` 模式变量来说，第一次调用之后 `lastIndex` 的值均被更改为 7 ，而第二次则均被改为 14 。

有两个关于粘连标志的微妙细节需要牢记：

1.  只有调用正则表达式对象上的方法（例如 `exec()` 与 `test()` 方法）， `lastIndex` 属性才会生效。而将正则表达式作为参数传递给字符串上的方法（例如 `match()` ），并不会体现粘连特性。
2.  当使用 `^` 字符来匹配字符串的起始处时，粘连的正则表达式只会匹配字符串的起始处（或者在多行模式下匹配行首）。当 `lastIndex` 为 0 时，`^` 不会让粘连的正则表达式与非粘连的有任何区别；而当 `lastIndex` 在单行模式下不对应整个字符串起始处，或者当它在多行模式下不对应行首时，粘连的正则表达式永远不会匹配成功。

和正则表达式其他标志相同，你可以根据一个属性来检测 `y` 标志是否存在。此时你需要检查的是 `sticky` 属性，如下：

```js
var pattern = /hello\d/y;

console.log(pattern.sticky);    // true 
```

如果粘连标志存在，那么 `sticky` 属性的值会被设为 true ，否则会被设为 false 。 `sticky` 属性由 `y` 标志存在与否决定，是只读的，它的值不能在代码中修改。

与 `u` 标志相似， `y` 标志也是个语法变更，所以在旧版 JS 引擎中它会造成语法错误。你可以用如下方法来检测它是否被支持：

```js
function hasRegExpY() {
    try {
        var pattern = new RegExp(".", "y");
        return true;
    } catch (ex) {
        return false;
    }
} 
```

此函数类似于对 `u` 标志的检查，在无法使用 `y` 标志来创建正则表达式时会返回 false 。同样，如果需要在旧版 JS 引擎中运行的代码中使用 `y` 标志，请确保使用 `RegExp` 构造器来定义正则表达式，以避免语法错误。

#### 复制正则表达式

在 ES5 中，你可以将正则表达式传递给 `RegExp` 构造器来复制它，就像这样：

```js
var re1 = /ab/i,
    re2 = new RegExp(re1); 
```

`re2` 变量只是 `re1` 的一个副本。但如果你向 `RegExp` 构造器传递了第二个参数，即正则表达式的标志，那么该代码就无法工作，正如该范例：

```js
var re1 = /ab/i,

    // ES5 中会抛出错误, ES6 中可用
    re2 = new RegExp(re1, "g"); 
```

如果你在 ES5 环境中运行这段代码，那么你会收到一条错误信息，表示在第一个参数已经是正则表达式的情况下不能再使用第二个参数。 ES6 则修改了这个行为，允许使用第二个参数，并且让它覆盖第一个参数中的标志。例如：

```js
var re1 = /ab/i,

    // ES5 中会抛出错误, ES6 中可用
    re2 = new RegExp(re1, "g");

console.log(re1.toString());            // "/ab/i"
console.log(re2.toString());            // "/ab/g"

console.log(re1.test("ab"));            // true
console.log(re2.test("ab"));            // true

console.log(re1.test("AB"));            // true
console.log(re2.test("AB"));            // false 
```

此代码中的 `re1` 带有忽略大小写的 `i` 标志，而 `re2` 则只带有全局的 `g` 标志。 `RegExp` 构造器复制了 `re1` 的模式并用 `g` 标志替换了 `i` 标志。如果没有第二个参数， `re2` 就会拥有与 `re1` 相同的标志。

#### flags 属性

在新增了一个标志并且对如何使用标志进行更改之余， ES6 还新增了一个与之关联的属性。在 ES5 中，你可以使用 `source` 属性来获取正则表达式的文本，但若想获取标志字符串，你必须解析 `toString()` 方法的输出，就像下面展示的那样：

```js
function getFlags(re) {
    var text = re.toString();
    return text.substring(text.lastIndexOf("/") + 1, text.length);
}

// toString() 的输出为 "/ab/g"
var re = /ab/g;

console.log(getFlags(re));          // "g" 
```

此处将正则表达式转换为一个字符串，并返回了最后一个 `/` 之后的字符，这些字符即为标志。

ES6 新增了 `flags` 属性用于配合 `source` 属性，让标志的获取变得更容易。这两个属性均为只有 getter 的原型访问器属性，因此都是只读的。 `flags` 属性使得检查正则表达式更容易，有助于调试与继承。

后期加入 ES6 的 `flags` 属性，会返回正则表达式中所有标志组成的字符串形式。例如：

```js
var re = /ab/g;

console.log(re.source);     // "ab"
console.log(re.flags);      // "g" 
```

本例查找了 `re` 的所有标志并将其打印到控制台，所用的代码量要比 `toString()` 方式少得多。同时使用 `source` 和 `flags` 允许你直接提取正则表达式的组成部分，而不必将正则表达式转换为字符串。

本章介绍过的字符串与正则表达式的改进已绝对强大，然而 ES6 在字符串方面还有更大的进步，它引入了一种新的字面量形式让字符串更加灵活。

### 模板字面量

JS 的字符串相对其他语言来说功能总是有限的。例如，本章介绍过的字符串方法在 ES6 之前都是缺失的，而字符串拼接的功能则尽可能简单。为了让开发者能够解决复杂的问题， ES6 的**模板字面量**（ **template literal** ）提供了创建领域专用语言（ domain-specific language ， DSL ）的语法，与 ES5 及更早版本的解决方案相比，处理内容可以更安全（领域专用语言是被设计用于特定有限目的的编程语言，与通用目的语言如 JavaScript 相反）。 ECMAScript wiki 在 [template literal strawman](http://wiki.ecmascript.org/doku.php?id=harmony:quasis) 上提供了如下描述：

> 本方案通过语法糖扩展了 ECMAScript 的语法，允许语言库提供 DSL 以便制作、查询并操纵来自于其它语言的内容，并且对注入攻击（ 如 XSS 、 SQL 注入，等等 ）能够免疫或具有抗性。

不过实际上，模板字面量是 ES6 针对 JS 直到 ES5 依然完全缺失的如下功能的回应：

*   **多行字符串**：针对多行字符串的形式概念；
*   **基本的字符串格式化**：将字符串部分替换为已存在的变量值的能力；
*   **HTML 转义**：能转换字符串以便将其安全插入到 HTML 中的能力。

模板字面量以一种新的方式解决了这些问题，而并未给 JS 已有的字符串添加额外功能。

#### 基本语法

模板字面量的最简单语法，是使用反引号（ ```js ）来包裹普通字符串，而不是用双引号或单引号。参考以下例子：

```
let message = `Hello world!`;

console.log(message);               // "Hello world!"
console.log(typeof message);        // "string"
console.log(message.length);        // 12 
```js

此代码说明了 `message` 变量包含的是一个普通的 JS 字符串。模板字面量语法被用于创建一个字符串值，并被赋值给了 `message` 变量。

若你想在字符串中包含反引号，只需使用反斜杠（ `\` ）转义即可，就像下面这个版本的 `message` 变量：

```
let message = `\`Hello\` world!`;

console.log(message);               // "`Hello` world!"
console.log(typeof message);        // "string"
console.log(message.length);        // 14 
```js

在模板字面量中无需对双引号或单引号进行转义。

#### 多行字符串

JS 开发者从该语言最初版本起就一直想要一种能创建多行字符串的方法。但在使用双引号或单引号时，整个字符串只能放在单独一行。

##### ES6 之前的权宜之计

感谢存在已久的一个语法 bug ， JS 的确有一种权宜之计：在换行之前的反斜线（ `\` ）可以用于创建多行字符串。这里有个范例：

```
var message = "Multiline \
string";

console.log(message);       // "Multiline string" 
```js

`message` 字符串打印输出时不会有换行，因为反斜线被视为续延符号而不是新行的符号。为了在输出中显示换行，你需要手动包含它：

```
var message = "Multiline \n\
string";

console.log(message);       // "Multiline
                            //  string" 
```js

在所有主流的 JS 引擎中，此代码都会输出两行，但是该行为被认定为一个 bug ，并且许多开发者都建议应避免这么做。

其他 ES6 之前创建多行字符串的尝试，一般都基于数组或字符串的拼接，就像这样：

```
var message = [
    "Multiline ",
    "string"
].join("\n");

let message = "Multiline \n" +
    "string"; 
```js

关于 JS 缺失的多行字符串功能，开发者的所有解决方法都不够完美。

##### 多行字符串的简单解决方法

ES6 的模板字面量使多行字符串更易创建，因为它不需要特殊的语法。只需在想要的位置包含换行即可，而且它会显示在结果中。例如：

```
let message = `Multiline
string`;

console.log(message);           // "Multiline
                                //  string"
console.log(message.length);    // 16 
```js

反引号之内的所有空白符都是字符串的一部分，因此需要留意缩进。例如：

```
let message = `Multiline
               string`;

console.log(message);           // "Multiline
                                //                 string"
console.log(message.length);    // 31 
```js

此代码中，模板字面量第二行前面的所有空白符都被视为字符串自身的一部分。如果让多行文本保持合适的缩进对你来说很重要，请考虑将多行模板字面量的第一行空置并在此后进行缩进，如下所示：

```
let html = `
<div>
    <h1>Title</h1>
</div>`.trim(); 
```js

此代码从第一行开始创建模板字面量，但在第二行之前并没有包含任何文本。 HTML 标签的缩进增强了可读性，之后再调用 trim() 方法移除了起始的空行。

> 如果你喜欢的话，也可以在模板字面量中使用 `\n` 来指示换行的插入位置：
> 
> ```
> let message = `Multiline\nstring`;
> 
> console.log(message);           // "Multiline
> //  string"
> console.log(message.length);    // 16 
> ```js

#### 制造替换位

此时模板字面量看上去仅仅是普通 JS 字符串的升级版，但二者之间真正的区别在于前者的“替换位”。替换位允许你将任何有效的 JS 表达式嵌入到模板字面量中，并将其结果输出为字符串的一部分。

替换位由起始的 `${` 与结束的 `}` 来界定，之间允许放入任意的 JS 表达式。最简单的替换位允许你将本地变量直接嵌入到结果字符串中，例如：

```
let name = "Nicholas",
    message = `Hello, ${name}.`;

console.log(message);       // "Hello, Nicholas." 
```js

替换位 `${name}` 会访问本地变量 `name` ，并将其值插入到 `message` 字符串中。 `message` 变量会立即保留该替换位的结果。

> 模板字面量能访问到作用域中任意的可访问变量。试图使用未定义的变量会抛出错误，无论是严格模式还是非严格模式。

既然替换位是 JS 表达式，那么可替换的就不仅仅是简单的变量名。你可以轻易嵌入计算、函数调用，等等。例如：

```
let count = 10,
    price = 0.25,
    message = `${count} items cost $${(count * price).toFixed(2)}.`;

console.log(message);       // "10 items cost $2.50." 
```js

此代码在模板字面量的一部分执行了一次计算， `count` 与 `price` 变量相乘，再使用 `.toFixed()` 方法将结果格式化为两位小数。而在第二个替换位之前的美元符号被照常输出，因为没有左花括号紧随其后。

模板字面量本身也是 JS 表达式，意味着你可以将模板字面量嵌入到另一个模板字面量内部，如同下例：

```
let name = "Nicholas",
    message = `Hello, ${
        `my name is ${ name }`
    }.`;

console.log(message);        // "Hello, my name is Nicholas." 
```js

此例在第一个模板字面量中套入了第二个。在首个 `${` 之后使用了另一个模板字面量，第二个 `${` 标示了嵌入到内层模板字面量的表达式的开始，该表达式为被插入结果的 `name` 变量。

#### 标签化模板

现在你已了解模板字面量在无须连接的情况下，是如何创建多行字符串以及将值插入字符串。不过模板字面量真正的力量来源于标签化模板。一个**模板标签**（ **template tag** ）能对模板字面量进行转换并返回最终的字符串值，标签在模板的起始处被指定，即在第一个 ``` 之前，如下所示：

```js
let message = tag`Hello world`; 
```

在本例中， `tag` 就是会被应用到 ``Hello world`` 模板字面量上的模板标签。

##### 定义标签

一个**标签**（ **tag** ）仅是一个函数，它被调用时接收需要处理的模板字面量数据。标签所接收的数据被划分为独立片段，并且必须将它们组合起来以创建结果。第一个参数是个数组，包含被 JS 解释过的字面量字符串，随后的参数是每个替换位的解释值。

标签函数的参数一般定义为剩余参数形式，以便更容易处理数据，如下：

```js
function tag(literals, ...substitutions) {
    // 返回一个字符串
} 
```

为了更好地理解传递给标签的是什么参数，可研究下例：

```js
let count = 10,
    price = 0.25,
    message = passthru`${count} items cost $${(count * price).toFixed(2)}.`; 
```

如果你拥有一个名为 `passthru()` 的函数，该函数将会接收到三个参数。首先是一个 `literals` 数组，包含如下元素：

*   在首个替换位之前的空字符串（ `""` ）；
*   首个替换位与第二个替换位之间的字符串（ `" items cost $"` ）；
*   第二个替换位之后的字符串（ `"."` ）。

接下来的参数会是 `10` ，也就是 `count` 变量的解释值，它也会成为 `substitutions` 数组的第一个元素。最后一个参数则会是 `"2.50"` ，即 `(count * price).toFixed(2)` 的解释值，并且会是 `substitutions` 数组的第二个元素。

需要注意 `literals` 的第一个元素是空字符串，以确保 `literals[0]` 总是字符串的起始部分，正如 `literals[literals.length - 1]` 总是字符串的结尾部分。同时替换位的元素数量也总是比字面量元素少 1 ，意味着表达式 `substitutions.length === literals.length - 1` 的值总是 true 。

使用这种模式，可以交替使用 `literals` 与 `substitutions` 数组来创建一个结果字符串：以 `literals` 中的首个元素开始，后面紧跟着 `substitutions` 中的首个元素，如此反复，直到结果字符串被创建完毕。你可以像下例这样交替使用两个数组中的值来模拟模板字面量的默认行为：

```js
function passthru(literals, ...substitutions) {
    let result = "";

    // 仅使用 substitution 的元素数量来进行循环
    for (let i = 0; i < substitutions.length; i++) {
        result += literals[i];
        result += substitutions[i];
    }

    // 添加最后一个字面量
    result += literals[literals.length - 1];

    return result;
}

let count = 10,
    price = 0.25,
    message = passthru`${count} items cost $${(count * price).toFixed(2)}.`;

console.log(message);       // "10 items cost $2.50." 
```

本例定义了 `passthru` 标签，能够执行与模板字面量的默认行为相同的转换操作。唯一的诀窍是在循环中使用 `substituions.length` 而不是 `literals.length` 来避免 `substituions` 数组的越界。它能工作是由于 ES6 对 `literals` 和 `substituions` 的良好定义。

> `substituions` 中包含的值不必是字符串。若表达式的计算结果为数字（就像上例），那么该数值也会被传入。决定这些值如何在结果中输出是标签的工作之一。

##### 使用模板字面量中的原始值

模板标签也能访问字符串的原始信息，主要指的是可以访问字符在转义之前的形式。获取原始字符串值的最简单方式是使用内置的 `String.raw()` 标签。例如：

```js
let message1 = `Multiline\nstring`,
    message2 = String.raw`Multiline\nstring`;

console.log(message1);          // "Multiline
                                //  string"
console.log(message2);          // "Multiline\\nstring" 
```

此代码中， `message1` 中的 `\n` 被解释为一个换行，而 `message2` 中的 `\n` 返回了它的原始形式 `"\\n"` （反斜线与 `n` 字符）。像这样提取原始字符串信息可以在必要时进行更复杂的处理。

字符串的原始信息同样会被传递给模板标签。标签函数的第一个参数为包含额外属性 `raw` 的数组，而 `raw` 属性则是含有与每个字面量值等价的原始值的数组。例如， `literals[0]` 的值总是等价于包含字符串原始信息的 `literals.raw[0]` 的值。知道这些之后，你可以用如下代码来模拟 `String.raw()`：

```js
function raw(literals, ...substitutions) {
    let result = "";

    // 仅使用 substitution 的元素数量来进行循环
    for (let i = 0; i < substitutions.length; i++) {
        result += literals.raw[i];      // 改为使用原始值
        result += substitutions[i];
    }

    // 添加最后一个字面量
    result += literals.raw[literals.length - 1];

    return result;
}

let message = raw`Multiline\nstring`;

console.log(message);           // "Multiline\\nstring"
console.log(message.length);    // 17 
```

这里使用 `literals.raw` 而非 `literals` 来输出结果字符串。这意味着任何转义字符（包括 Unicode 代码点的转义）都会以原始的形式返回。当你想在输出的字符串中包含转义字符时，原始字符串会很有帮助（例如，若想要生成包含代码的文档，那么应当输出如表面看到那样的实际代码）。

### 总结

完整的 Unicode 支持允许 JS 以合理的方式处理 UTF-16 字符。通过 `codePointAt()` 与 `String.fromCodePoint()` 在代码点和字符之间转换的能力，是字符串操作的一大进步。正则表达式新增的 `u` 标志使得直接操作代码点而不是 16 位字符变为可能，而 `normalize()` 方法则允许进行更恰当的字符串比较。

ES6 也添加了操作字符串的新方法，允许你更容易识别子字符串，而不用管它在父字符串中的位置。正则表达式同样引入了许多功能。

模板字面量是 ES6 的一项重要补充，允许你创建领域专用语言（ DSL ）让字符串的创建更容易。能将变量直接嵌入到模板字面量中，意味着开发者在组合长字符串与变量时，有了一种比字符串拼接更为安全的工具。

内置的多行字符串支持，是普通 JS 字符串绝对无法做到的，这使得模板字面量成为凌驾于前者之上的有用升级。尽管在模板字面量中允许直接使用换行，你依然可以使用 `\n` 或其它字符转义序列。

模板标签是创建 DSL 最重要的部分。标签是接收模板字面量片段作为参数的函数，你可以使用它们来返回合适的字符串。这些数据包括了字面量、等价的原始值以及替换位的值，标签使用这些信息片段来决定输出。