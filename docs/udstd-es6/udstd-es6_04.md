# 第三章 函数

## 第三章 函数

函数在任何编程语言中都是非常重要的一部分，而从 JS 诞生一直到 ES6 之前，函数并未有过较大的变化。这导致诸多问题以及细微行为差异被积压，由此容易诱发错误，并且经常需要用大量代码来实现非常基本的功能。

ES6 的函数考虑了 JS 开发者多年的抱怨和要求，向前大步迈进，于是便在 ES5 函数之上实现了不少增量改进，让 JS 的编程错误更少并且更加强大。

*   带参数默认值的函数
    *   在 ES5 中模拟参数默认值
    *   ES6 中的参数默认值
    *   参数默认值如何影响 arguments 对象
    *   参数默认值表达式
    *   参数默认值的暂时性死区
*   使用不具名参数
    *   ES5 中的不具名参数
    *   剩余参数
        *   剩余参数的限制条件
        *   剩余参数如何影响 arguments 对象
*   函数构造器的增强能力
*   扩展运算符
*   ES6 的名称属性
    *   选择合适的名称
    *   名称属性的特殊情况
*   明确函数的双重用途
    *   在 ES5 中判断函数如何被调用
    *   new.target 元属性
*   块级函数
    *   决定何时使用块级函数
    *   非严格模式的块级函数
*   箭头函数
    *   箭头函数语法
    *   创建立即调用函数表达式
    *   没有 this 绑定
    *   箭头函数与数组
    *   没有 arguments 绑定
    *   识别箭头函数
*   尾调用优化
    *   有何不同？
    *   如何控制尾调用优化
*   总结

### 带参数默认值的函数

JS 函数的独特之处是可以接受任意数量的参数，而无视函数声明处的参数数量。这让你定义的函数可以使用不同的参数数量来调用，调用时未提供的参数经常会使用默认值来代替。本章将介绍默认的参数值在 ES6 之中以及之前是如何实现的，顺带介绍的内容还有： `arguments` 对象的一些重要信息、将表达式作为参数使用，以及另一种 TDZ 。

#### 在 ES5 中模拟参数默认值

在 ES5 或更早的版本中，你可能会使用下述模式来创建带有参数默认值的函数：

```
function makeRequest(url, timeout, callback) {

    timeout = timeout || 2000;
    callback = callback || function() {};

    // 函数的剩余部分

} 
```

在本例中， `timeout` 与 `callback` 实际上都是可选参数，因为他们都会在参数未被提供的情况下使用默认值。逻辑或运算符（ `||` ）在左侧的值为假的情况下总会返回右侧的操作数。由于函数的具名参数在未被明确提供时会是 `undefined` ，逻辑或运算符就经常被用来给缺失的参数提供默认值。不过此方法有个瑕疵，此处的 `timeout` 的有效值实际上有可能是 `0` ，但因为 `0` 是假值，就会导致 `timeout` 的值在这种情况下会被替换为 `2000` 。

在这种情况下，更安全的替代方法是使用 `typeof` 来检测参数的类型，正如下例：

```
function makeRequest(url, timeout, callback) {

    timeout = (typeof timeout !== "undefined") ? timeout : 2000;
    callback = (typeof callback !== "undefined") ? callback : function() {};

    // 函数的剩余部分

} 
```

虽然这种方法更安全，但依然为实现一个基本需求而书写了过多的代码。它代表了一种公共模式，而流行的 JS 库中都充斥着类似的模式。

#### ES6 中的参数默认值

ES6 能更容易地为参数提供默认值，它使用了初始化形式，以便在参数未被正式传递进来时使用。例如：

```
function makeRequest(url, timeout = 2000, callback = function() {}) {

    // 函数的剩余部分

} 
```

此函数只要求第一个参数始终要被传递。其余两个参数则都有默认值，这使得函数体更为小巧，因为不需要再添加更多代码来检查缺失的参数值。

如果使用全部三个参数来调用 `makeRequest()` ，那么默认值将不会被使用，例如：

```
// 使用默认的 timeout 与 callback
makeRequest("/foo");

// 使用默认的 callback
makeRequest("/foo", 500);

// 不使用默认值
makeRequest("/foo", 500, function(body) {
    doSomething(body);
}); 
```

ES6 会认为 `url` 参数是必须的，这就是三次调用 `makeRequest()` 都传入了 `"/foo"` 的原因。而拥有默认值的两个参数都被认为是可选的。

在函数声明中能指定任意一个参数的默认值，即使该参数排在未指定默认值的参数之前也是可以的。例如，下面这样是可行的：

```
function makeRequest(url, timeout = 2000, callback) {

    // 函数的剩余部分

} 
```

在本例中，只有在未传递第二个参数、或明确将第二个参数值指定为 `undefined` 时， `timeout` 的默认值才会被使用，例如：

```
// 使用默认的 timeout
makeRequest("/foo", undefined, function(body) {
    doSomething(body);
});

// 使用默认的 timeout
makeRequest("/foo");

// 不使用默认值
makeRequest("/foo", null, function(body) {
    doSomething(body);
}); 
```

在关于参数默认值的这个例子中， `null` 值被认为是有效的，意味着对于 `makeRequest()` 的第三次调用并不会使用 `timeout` 的默认值。

#### 参数默认值如何影响 arguments 对象

需要记住的是， `arguments` 对象会在使用参数默认值时有不同的表现。在 ES5 的非严格模式下， `arguments` 对象会反映出具名参数的变化。以下代码说明了该工作机制：

```
function mixArgs(first, second) {
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
    first = "c";
    second = "d";
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
}

mixArgs("a", "b"); 
```

输出：

```
true
true
true
true 
```

在非严格模式下， `arguments` 对象总是会被更新以反映出具名参数的变化。因此当 `first` 与 `second` 变量被赋予新值时， `arguments[0]` 与 `arguments[1]` 也就相应地更新了，使得这里所有的 `===` 比较的结果都为 `true` 。

然而在 ES5 的严格模式下，关于 `arguments` 对象的这种混乱情况被消除了，它不再反映出具名参数的变化。在严格模式下重新使用上例中的函数：

```
function mixArgs(first, second) {
 "use strict";

    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
    first = "c";
    second = "d"
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
}

mixArgs("a", "b"); 
```

调用 `mixArgs()` 则输出：

```
true
true
false
false 
```

这一次更改 `first` 与 `second` 就不会再影响 `arguments` 对象，因此输出结果符合通常的期望。

然而在使用 ES6 参数默认值的函数中， `arguments` 对象的表现总是会与 ES5 的严格模式一致，无论此时函数是否明确运行在严格模式下。参数默认值的存在触发了 `arguments` 对象与具名参数的分离。这是个细微但重要的细节，因为 `arguments` 对象的使用方式发生了变化。研究如下代码：

```
// 非严格模式
function mixArgs(first, second = "b") {
    console.log(arguments.length);
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
    first = "c";
    second = "d"
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
}

mixArgs("a"); 
```

输出：

```
1
true
false
false
false 
```

本例中 `arguments.length` 的值为 `1` ，因为只给 `mixArgs()` 传递了一个参数。这也意味着 `arguments[1]` 的值是 `undefined` ，符合将单个参数传递给函数时的预期；这同时意味着 `first` 与 `arguments[0]` 是相等的。改变 `first` 和 `second` 的值不会对 `arguments` 对象造成影响，无论是否在严格模式下，所以你可以始终依据 `arguments` 对象来反映初始调用状态。

#### 参数默认值表达式

参数默认值最有意思的特性或许就是默认值并不要求一定是基本类型的值。例如，你可以执行一个函数来产生参数的默认值，就像这样：

```
function getValue() {
    return 5;
}

function add(first, second = getValue()) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 6 
```

此处若未提供第二个参数， `getValue()` 函数就会被调用以获取正确的默认值。需要注意的是，仅在调用 `add()` 函数而未提供第二个参数时， `getValue()` 函数才会被调用，而在 `getValue()` 的函数声明初次被解析时并不会进行调用。这意味着 `getValue()` 函数若被写为可变的，则它有可能会返回可变的值，例如：

```
let value = 5;

function getValue() {
    return value++;
}

function add(first, second = getValue()) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 6
console.log(add(1));        // 7 
```

本例中 `value` 的初始值是 5 ，并且会随着对 `getValue()` 的每次调用而递增。首次调用 `add(1)` 返回的值为 6 ，再次调用则返回 7 ，因为 `value` 的值已经被增加了。由于 `second` 参数的默认值总是在 `add()` 函数被调用的情况下才被计算，因此就能随时更改该参数的值。

> 将函数调用作为参数的默认值时需要小心，如果你遗漏了括号，例如在上面例子中使用 `second = getValue` ，你就传递了对于该函数的一个引用，而没有传递调用该函数的结果。

这种行为引出了另一种有趣的能力：可以将前面的参数作为后面参数的默认值，这里有个例子：

```
function add(first, second = first) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 2 
```

此代码中 `first` 为 `second` 参数提供了默认值，意味着只传入一个参数会让两个参数获得相同的值，因此 `add(1, 1)` 与 `add(1)` 同样返回了 2 。进一步说，你可以将 `first` 作为参数传递给一个函数来产生 `second` 参数的值，正如下例：

```
function getValue(value) {
    return value + 5;
}

function add(first, second = getValue(first)) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 7 
```

此例将 `second` 的值设为等于 `getValue(first)` 函数的返回值，因此 `add(1)` 会返回 7 （ 1 + 6 ），而 `add(1, 1)` 仍然返回 2 。

引用其他参数来为参数进行默认赋值时，仅允许引用前方的参数，因此前面的参数不能访问后面的参数，例如：

```
function add(first = second, second) {
    return first + second;
}

console.log(add(1, 1));         // 2
console.log(add(undefined, 1)); // 抛出错误 
```

调用 `add(undefined, 1)` 发生了错误，是因为 `second` 在 `first` 之后定义，因此不能将其作为后者的默认值。要理解为何会发生这种情况，需要着重回顾“暂时性死区”。

#### 参数默认值的暂时性死区

第一章介绍了 `let` 与 `const` 的暂时性死区（ TDZ ），而参数默认值同样有着无法访问特定参数的暂时性死区。与 `let` 声明相似，函数每个参数都会创建一个新的标识符绑定，它在初始化之前不允许被访问，否则会抛出错误。参数初始化会在函数被调用时进行，无论是给参数传递了一个值、还是使用了参数的默认值。

为了探寻参数默认值中的暂时性死区，可再次研究“参数默认值表达式”中的例子：

```
function getValue(value) {
    return value + 5;
}

function add(first, second = getValue(first)) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 7 
```

调用 `add(1, 1)` 和 `add(1)` 事实上执行了以下代码来创建 `first` 与 `second` 的参数值：

```
// JS 调用 add(1, 1) 可表示为
let first = 1;
let second = 1;

// JS 调用 add(1) 可表示为
let first = 1;
let second = getValue(first); 
```

当函数 `add()` 第一次执行时， `first` 与 `second` 的绑定被加入了特定参数的暂时性死区（类似于 `let` 声明的行为）。因此 `second` 可以使用 `first` 来初始化，因为此处 `first` 总是已经完成了初始化，但反之则不行。现在再研究以下重写过的 `add()` 函数：

```
function add(first = second, second) {
    return first + second;
}

console.log(add(1, 1));         // 2
console.log(add(undefined, 1)); // 抛出错误 
```

本例中调用 `add(1, 1)` 与 `add(undefined, 1)` 对应着以下的后台代码：

```
// JS 调用 add(1, 1) 可表示为
let first = 1;
let second = 1;

// JS 调用 add(1) 可表示为
let first = second;
let second = 1; 
```

本例中调用 `add(undefined, 1)` 抛出了错误，是因为在 `first` 被初始化时 `second` 尚未被初始化。此处的 `second` 存在于暂时性死区内，对于 `second` 的引用就抛出了错误。这反映出第一章讨论过的 `let` 绑定的行为。

> 函数参数拥有各自的作用域和暂时性死区，与函数体的作用域相分离，这意味着参数的默认值不允许访问在函数体内部声明的任意变量。

### 使用不具名参数

到目前为止，本章的例子只涵盖了在函数定义中的已被命名的参数。然而 JS 的函数并不强求参数的数量要等于已定义具名参数的数量，你所传递的参数总是允许少于或多于正式指定的参数。参数的默认值让函数在接收更少参数时的行为更清晰，而 ES6 试图让相反情况的问题也被更好地解决。

#### ES5 中的不具名参数

JS 早就提供了 `arguments` 对象用于查看传递给函数的所有参数，这样就不必分别指定每个参数。虽然查看 `arguments` 对象在大多数情况下都工作正常，但操作它有时仍然比较麻烦。例如，参考以下查看 `arguments` 对象的代码：

```
function pick(object) {
    let result = Object.create(null);

    // 从第二个参数开始处理
    for (let i = 1, len = arguments.length; i < len; i++) {
        result[arguments[i]] = object[arguments[i]];
    }

    return result;
}

let book = {
    title: "Understanding ES6",
    author: "Nicholas C. Zakas",
    year: 2015
};

let bookData = pick(book, "author", "year");

console.log(bookData.author);   // "Nicholas C. Zakas"
console.log(bookData.year);     // 2015 
```

此函数模拟了 **Underscore.js** 代码库的 `pick()` 方法，能够返回包含原有对象特定属性的子集副本。本例中只为函数定义了一个参数，期望该参数就是需要从中拷贝属性的来源对象，除此之外传递的所有参数则都是需要拷贝的属性的名称。

这个 `pick()` 函数有两点需要注意。首先，完全看不出该函数能够处理多个参数，你能为其再多定义几个参数，但依然不足以标明该函数能处理任意数量的参数。其次，由于第一个参数被命名并被直接使用，当你寻找需要复制的属性时，就必须从 `arguments` 对象索引位置 1 开始处理而不是从位置 0 。要记住使用 `arguments` 的适当索引值并不一定困难，但毕竟多了一件需要留意的事。

ES6 引入了剩余参数以便解决这个问题。

#### 剩余参数

**剩余参数**（ **rest parameter** ）由三个点（ `...` ）与一个紧跟着的具名参数指定，它会是包含传递给函数的其余参数的一个数组，名称中的“剩余”也由此而来。例如， `pick()` 函数可以像下面这样用剩余参数来重写：

```
function pick(object, ...keys) {
    let result = Object.create(null);

    for (let i = 0, len = keys.length; i < len; i++) {
        result[keys[i]] = object[keys[i]];
    }

    return result;
} 
```

在此版本的函数中， `keys` 是一个包含所有在 `object` 之后的参数的剩余参数（这与包含所有参数的 `arguments` 不同，后者会连第一个参数都包含在内）。这意味着你可以对 `keys` 从头到尾进行迭代，而不需要有所顾虑。作为一个额外的收益，通过观察该函数便能判明它具有处理任意数量参数的能力。

> 函数的 `length` 属性用于指示具名参数的数量，而剩余参数对其毫无影响。此例中 `pick()` 函数的 `length` 属性值是 1 ，因为只有 `object` 参数被用于计算该值。

##### 剩余参数的限制条件

剩余参数受到两点限制。一是函数只能有一个剩余参数，并且它必须被放在最后。例如，如下代码是无法工作的：

```
// 语法错误：不能在剩余参数后使用具名参数
function pick(object, ...keys, last) {
    let result = Object.create(null);

    for (let i = 0, len = keys.length; i < len; i++) {
        result[keys[i]] = object[keys[i]];
    }

    return result;
} 
```

此处的 `last` 跟在了剩余参数 `keys` 后面，这会导致一个语法错误。

第二个限制是剩余参数不能在对象字面量的 setter 属性中使用，这意味着如下代码同样会导致语法错误：

```
let object = {

    // 语法错误：不能在 setter 中使用剩余参数
    set name(...value) {
        // 一些操作
    }
}; 
```

存在此限制的原因是：对象字面量的 setter 被限定只能使用单个参数；而剩余参数按照定义是不限制参数数量的，因此它在此处不被许可。

##### 剩余参数如何影响 arguments 对象

设计剩余参数是为了替代 ES 中的 `arguments` 。原先在 ES4 中就移除了 `arguments` 并添加了剩余参数，以便允许向函数传入不限数量的参数。尽管 ES4 从未被实施，但这个想法被保持下来并在 ES6 中被重新引入，虽然 `arguments` 仍未在语言中被移除。

`arguments` 对象在函数被调用时反映了传入的参数，与剩余参数能协同工作，就像如下程序所演示的：

```
function checkArgs(...args) {
    console.log(args.length);
    console.log(arguments.length);
    console.log(args[0], arguments[0]);
    console.log(args[1], arguments[1]);
}

checkArgs("a", "b"); 
```

调用 `checkArgs()` 输出了：

```
2
2
a a
b b 
```

`arguments` 对象总能正确反映被传入函数的参数，而无视剩余参数的使用。

这已是对剩余参数真正需要了解的全部内容，你可以开始使用它们了。

### 函数构造器的增强能力

`Function` 构造器允许你动态创建一个新函数，但在 JS 中并不常用。传给该构造器的参数都是字符串，它们就是目标函数的参数与函数体，这里有个范例：

```
var add = new Function("first", "second", "return first + second");

console.log(add(1, 1));     // 2 
```

ES6 增强了 `Function` 构造器的能力，允许使用默认参数以及剩余参数。对于默认参数来说，你只需为参数名称添加等于符号以及默认值，正如下例：

```
var add = new Function("first", "second = first",
        "return first + second");

console.log(add(1, 1));     // 2
console.log(add(1));        // 2 
```

在此例中，当只传递了一个参数时， `first` 的值会被赋给 `second` 参数，此处的语法与不使用 `Function` 的函数声明一致。

而对剩余参数来说，只需在最后一个参数前添加 `...` 即可，就像这样：

```
var pickFirst = new Function("...args", "return args[0]");

console.log(pickFirst(1, 2));   // 1 
```

此代码创建了一个仅使用剩余参数的函数，让其返回所传入的第一个参数。

默认参数和剩余参数的添加，确保了 `Function` 构造器拥有与函数声明形式相同的所有能力。

### 扩展运算符

与剩余参数关联最密切的就是扩展运算符。剩余参数允许你把多个独立的参数合并到一个数组中；而扩展运算符则允许将一个数组分割，并将各个项作为分离的参数传给函数。考虑一下 `Math.max()` 方法，它接受任意数量的参数，并会返回其中的最大值。这里有个此方法的简单用例：

```
let value1 = 25,
    value2 = 50;

console.log(Math.max(value1, value2));      // 50 
```

若像本例这样仅处理两个值，那么 `Math.max()` 非常容易使用：将这两个值传入，就会返回较大的那个。但若想处理数组中的值，此时该如何找到最大值？ `Math.max()` 方法并不允许你传入一个数组，因此在 ES5 或更早版本中，你必须自行搜索整个数组，或像下面这样使用 `apply()` 方法：

```
let values = [25, 50, 75, 100]

console.log(Math.max.apply(Math, values));  // 100 
```

该解决方案是可行的，但如此使用 `apply()` 会让人有一点疑惑，它实际上使用了额外的语法混淆了代码的真实意图。

ES6 的扩展运算符令这种情况变得简单。无须调用 `apply()` ，你可以像使用剩余参数那样在该数组前添加 `...` ，并直接将其传递给 `Math.max()` 。 JS 引擎将会将该数组分割为独立参数并把它们传递进去，就像这样：

```
let values = [25, 50, 75, 100]

// 等价于 console.log(Math.max(25, 50, 75, 100));
console.log(Math.max(...values));           // 100 
```

现在调用 `Math.max()` 看起来更传统一些，并避免了为一个简单数学操作使用复杂的 `this` 绑定（即在上个例子中提供给 `Math.max.apply()` 的第一个参数）。

你可以将扩展运算符与其他参数混用。假设你想让 `Math.max()` 返回的最小值为 0 （以防数组中混入了负值），你可以将参数 0 单独传入，并继续为其他参数使用扩展运算符，正如下例：

```
let values = [-25, -50, -75, -100]

console.log(Math.max(...values, 0));        // 0 
```

本例中传给 `Math.max()` 的最后一个参数是 `0` ，它跟在使用扩展运算符的其他参数之后传入。

用扩展运算符传递参数，使得更容易将数组作为函数参数来使用，你会发现在大部分场景中扩展运算符都是 `apply()` 方法的合适替代品。

除了你至今看到的默认参数与剩余参数的用法之外，在 ES6 中还可以在 JS 的 `Function` 构造器中使用这两类参数。

### ES6 的名称属性

定义函数有各种各样的方式，在 JS 中识别函数就变得很有挑战性。此外，匿名函数表达式的流行使得调试有点困难，经常导致堆栈跟踪难以被阅读与解释。正因为此， ES6 给所有函数添加了 `name` 属性。

#### 选择合适的名称

ES6 中所有函数都有适当的 `name` 属性值。为了理解其实际运作，请看下例——它展示了一个函数与一个函数表达式，并将二者的 `name` 属性都打印出来：

```
function doSomething() {
    // ...
}

var doAnotherThing = function() {
    // ...
};

console.log(doSomething.name);          // "doSomething"
console.log(doAnotherThing.name);       // "doAnotherThing" 
```

在此代码中，由于是一个函数声明， `doSomething()` 就拥有一个值为 `"doSomething"` 的 `name` 属性。而匿名函数表达式 `doAnotherThing()` 的`name` 属性值则是 `"doAnotherThing"` ，因为这是该函数所赋值的变量的名称。

> 译注：匿名函数的名称属性在 FireFox 与 Edge 中仍然不被支持（值为空字符串），而 Chrome 直到 51.0 版本才提供了该特性。

#### 名称属性的特殊情况

虽然函数声明与函数表达式的名称易于查找，但 ES6 更进一步确保了**所有**函数都拥有合适的名称。为了表明这点，请参考如下程序：

```
var doSomething = function doSomethingElse() {
    // ...
};

var person = {
    get firstName() {
        return "Nicholas"
    },
    sayName: function() {
        console.log(this.name);
    }
}

console.log(doSomething.name);      // "doSomethingElse"
console.log(person.sayName.name);   // "sayName"

var descriptor = Object.getOwnPropertyDescriptor(person, "firstName");
console.log(descriptor.get.name); // "get firstName" 
```

本例中的 `doSomething.name` 的值是 `"doSomethingElse"` ，因为该函数表达式自己拥有一个名称，并且此名称的优先级要高于赋值目标的变量名。 `person.sayName()` 的 `name` 属性值是 `"sayName"` ，正如对象字面量指定的那样。类似的， `person.firstName` 实际是个 getter 函数，因此它的名称是 `"get firstName"` ，以标明它的特征；同样， setter 函数也会带有 `"set"` 的前缀（ getter 与 setter 函数都必须用 `Object.getOwnPropertyDescriptor()` 来检索）。

函数名称还有另外两个特殊情况。使用 `bind()` 创建的函数会在名称属性值之前带有 `"bound"` 前缀；而使用 `Function` 构造器创建的函数，其名称属性则会有 `"anonymous"` 前缀，正如此例：

```
var doSomething = function() {
    // ...
};

console.log(doSomething.bind().name);   // "bound doSomething"

console.log((new Function()).name);     // "anonymous" 
```

绑定产生的函数拥有原函数的名称，并总会附带 `"bound"` 前缀，因此 `doSomething()` 函数的绑定版本就具有 `"bound doSomething"` 名称。

> 需要注意的是，函数的 `name` 属性值未必会关联到同名变量。 `name` 属性是为了在调试时获得有用的相关信息，所以不能用 `name` 属性值去获取对函数的引用。

### 明确函数的双重用途

在 ES5 以及更早版本中，函数根据是否使用 `new` 来调用而有双重用途。当使用 `new` 时，函数内部的 `this` 是一个新对象，并作为函数的返回值，如下例所示：

```
function Person(name) {
    this.name = name;
}

var person = new Person("Nicholas");
var notAPerson = Person("Nicholas");

console.log(person);        // "[Object object]"
console.log(notAPerson);    // "undefined" 
```

当创建 `notAPerson` 时，未使用 `new` 来调用 `Person()` ，输出了 `undefined` （并且在非严格模式下给全局对象添加了 `name` 属性）。 `Person` 首字母大写是指示其应当使用 `new` 来调用的唯一标识，这在 JS 编程中十分普遍。函数双重角色的混乱情况在 ES6 中发生了一些改变。

JS 为函数提供了两个不同的内部方法： `[[Call]]` 与 `[[Construct]]` 。当函数未使用 `new` 进行调用时， `[[call]]` 方法会被执行，运行的是代码中显示的函数体。而当函数使用 `new` 进行调用时， `[[Construct]]` 方法则会被执行，负责创建一个被称为**新目标**的新的对象，并且使用该新目标作为 `this` 去执行函数体。拥有 `[[Construct]]` 方法的函数被称为**构造器**。

> 记住并不是所有函数都拥有 `[[Construct]]` 方法，因此不是所有函数都可以用 `new` 来调用。在“箭头函数”小节中介绍的箭头函数就未拥有该方法。

#### 在 ES5 中判断函数如何被调用

在 ES5 中判断函数是不是使用了 `new` 来调用（即作为构造器），最流行的方式是使用 `instanceof` ，例如：

```
function Person(name) {
    if (this instanceof Person) {
        this.name = name;   // 使用 new
    } else {
        throw new Error("You must use new with Person.")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person("Nicholas");  // 抛出错误 
```

此处对 `this` 值进行了检查，来判断其是否为构造器的一个实例：若是，正常继续执行；否则抛出错误。这能奏效是因为 `[[Construct]]` 方法创建了 `Person` 的一个新实例并将其赋值给 `this` 。可惜的是，该方法并不绝对可靠，因为在不使用 `new` 的情况下 `this` 仍然可能是 `Person` 的实例，正如下例：

```
function Person(name) {
    if (this instanceof Person) {
        this.name = name;   // 使用 new
    } else {
        throw new Error("You must use new with Person.")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person.call(person, "Michael");    // 奏效了！ 
```

调用 `Person.call()` 并将 `person` 变量作为第一个参数传入，这意味着将 `Person` 内部的 `this` 设置为了 `person` 。对于该函数来说，没有任何方法能将这种方式与使用 `new` 调用区分开来。

#### new.target 元属性

为了解决这个问题， ES6 引入了 `new.target` **元属性**。元属性指的是“非对象”（例如 `new` ）上的一个属性，并提供关联到它的目标的附加信息。当函数的 `[[Construct]]` 方法被调用时， `new.target` 会被填入 `new` 运算符的作用目标，该目标通常是新创建的对象实例的构造器，并且会成为函数体内部的 `this` 值。而若 `[[Call]]` 被执行， `new.target` 的值则会是 `undefined` 。

通过检查 `new.target` 是否被定义，这个新的元属性就让你能安全地判断函数是否被使用 `new` 进行了调用。

```
function Person(name) {
    if (typeof new.target !== "undefined") {
        this.name = name;   // 使用 new
    } else {
        throw new Error("You must use new with Person.")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person.call(person, "Michael");    // 出错！ 
```

使用 `new.target` 而非 `this instanceof Person` ， `Person` 构造器会在未使用 `new` 调用时正确地抛出错误。

也可以检查 `new.target` 是否被使用特定构造器进行了调用，例如以下代码：

```
function Person(name) {
    if (new.target === Person) {
        this.name = name;   // 使用 new
    } else {
        throw new Error("You must use new with Person.")
    }
}

function AnotherPerson(name) {
    Person.call(this, name);
}

var person = new Person("Nicholas");
var anotherPerson = new AnotherPerson("Nicholas");  // 出错！ 
```

> 译注：原文此段代码有误。
> 
> ```
> if (new.target === Person) { 
> ```
> 
> 这一行原先写为：
> 
> ```
> if (typeof new.target === Person) { 
> ```
> 
> 原先的写法是有问题的，不能正确发挥作用，它会在 `new Person("Nicholas")` 这行就抛出错误。

在此代码中，为了正确工作， `new.target` 必须是 `Person` 。当调用 `new AnotherPerson("Nicholas")` 时， `Person.call(this, name)` 也随之被调用，从而抛出了错误，因为此时在 `Person` 构造器内部的 `new.target` 值为 `undefined` （ `Person` 并未使用 `new` 调用）。

> 警告：在函数之外使用 `new.target` 会有语法错误。

ES6 通过新增 `new.target` 而消除了函数调用方面的不确定性。在该主题上，ES6 还随之解决了本语言之前另一个不确定的部分——在代码块内部声明函数。

### 块级函数

在 ES3 或更早版本中，在代码块中声明函数（即**块级函数**）严格来说应当是一个语法错误，但所有的浏览器却都支持该语法。可惜的是，每个支持该语法的浏览器都有轻微的行为差异，所以最佳实践就是不要在代码块中声明函数（更好的选择是使用函数表达式）。

为了控制这种不兼容行为， ES5 的严格模式为代码块内部的函数声明引入了一个错误，就像这样：

```
"use strict";

if (true) {

    // 在 ES5 会抛出语法错误， ES6 则不会
    function doSomething() {
        // ...
    }
} 
```

在 ES5 中，这段代码会抛出语法错误。然而 ES6 会将 `doSomething()` 函数视为块级声明，并允许它在定义所在的代码块内部被访问。例如：

```
"use strict";

if (true) {

    console.log(typeof doSomething);        // "function"

    function doSomething() {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething);            // "undefined" 
```

块级函数会被提升到定义所在的代码块的顶部，因此 `typeof doSomething` 会返回 `"function"` ，即便该检查位于此函数定义位置之前。一旦 `if` 代码块执行完毕， `doSomething()` 也就不复存在。

#### 决定何时使用块级函数

块级函数与 `let` 函数表达式相似，在执行流跳出定义所在的代码块之后，函数定义就会被移除。关键区别在于：块级函数会被提升到所在代码块的顶部；而使用 `let` 的函数表达式则不会，正如以下范例所示：

```
"use strict";

if (true) {

    console.log(typeof doSomething);        // 抛出错误

    let doSomething = function () {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething); 
```

此处代码在 `typeof doSomething` 被执行时中断了，因为 `let` 声明尚未被执行，将 `doSomething()` 放入了暂时性死区。知道这个区别之后，你就可以根据是否想要提升来选择应当使用块级函数还是 `let` 表达式。

#### 非严格模式的块级函数

ES6 在非严格模式下同样允许使用块级函数，但行为有细微不同。块级函数的作用域会被提升到所在函数或全局环境的顶部，而不是代码块的顶部。

```
// ES6 behavior
if (true) {

    console.log(typeof doSomething);        // "function"

    function doSomething() {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething);            // "function" 
```

本例中的 `doSomething()` 会被提升到全局作用域，因此在 `if` 代码块外部它仍然存在。 ES6 标准化了这种行为来移除浏览器之前存在的不兼容性，于是在所有 ES6 运行环境中其行为都会遵循相同的方式。

允许使用块级函数增强了在 JS 中声明函数的能力，但 ES6 还引入了一种全新的声明函数的方式。

### 箭头函数

ES6 最有意思的一个新部分就是**箭头函数**（ **arrow function** ）。箭头函数正如名称所示那样使用一个“箭头”（ `=>` ）来定义，但它的行为在很多重要方面与传统的 JS 函数不同：

*   **没有 `this` 、 `super` 、`arguments` ，也没有 `new.target` 绑定**： `this` 、 `super` 、 `arguments` 、以及函数内部的 `new.target` 的值由所在的、最靠近的非箭头函数来决定（ `super` 详见第四章）。
*   **不能被使用 `new` 调用**： 箭头函数没有 `[[Construct]]` 方法，因此不能被用为构造函数，使用 `new` 调用箭头函数会抛出错误。
*   **没有原型**： 既然不能对箭头函数使用 `new` ，那么它也不需要原型，也就是没有 `prototype` 属性。
*   **不能更改 `this`**： `this` 的值在函数内部不能被修改，在函数的整个生命周期内其值会保持不变。
*   **没有 `arguments` 对象**： 既然箭头函数没有 `arguments` 绑定，你必须依赖于具名参数或剩余参数来访问函数的参数。
*   **不允许重复的具名参数**： 箭头函数不允许拥有重复的具名参数，无论是否在严格模式下；而相对来说，传统函数只有在严格模式下才禁止这种重复。

产生这些差异是有理由的。首先并且最重要的是，在 JS 编程中 `this` 绑定是发生错误的常见根源之一，在嵌套的函数中有时会因为调用方式的不同，而导致丢失对外层 `this` 值的追踪，就可能会导致预期外的程序行为。其次，箭头函数使用单一的 `this` 值来执行代码，使得 JS 引擎可以更容易对代码的操作进行优化；而常规函数可能会作为构造函数使用（导致 `this` 易变而不利优化）。

其余差异也聚集在减少箭头函数内部的错误与不确定性，这样 JS 引擎也能更好地优化箭头函数的运行。

> 注意：箭头函数也拥有 `name` 属性，并且遵循与其他函数相同的规则。

#### 箭头函数语法

箭头函数的语法可以有多种变体，取决于你要完成的目标。所有变体都以函数参数为开头，紧跟着的是箭头，再接下来则是函数体。参数与函数体都根据实际使用有不同的形式。例如，以下箭头函数接收单个参数并返回它：

```
var reflect = value => value;

// 有效等价于：

var reflect = function(value) {
    return value;
}; 
```

当箭头函数只有单个参数时，该参数可以直接书写而不需要额外的语法；接下来是箭头以及箭头右边的表达式，该表达式会被计算并返回结果。即使此处没有明确的 `return` 语句，该箭头函数仍然会将所传入的参数返回出来。

如果需要传入多于一个的参数，就需要将它们放在括号内，就像这样：

```
var sum = (num1, num2) => num1 + num2;

// 有效等价于：

var sum = function(num1, num2) {
    return num1 + num2;
}; 
```

`sum()` 函数简单地将两个参数相加并返回结果。此箭头函数与上面的 `reflect()` 之间唯一区别在于：此处的参数被封闭在括号内，相互之间使用逗号分隔（就像传统函数那样）。

如果函数没有任何参数，那么在声明时就必须使用一对空括号，就像这样：

```
var getName = () => "Nicholas";

// 有效等价于：

var getName = function() {
    return "Nicholas";
}; 
```

当你想使用更传统的函数体、也就是可能包含多个语句的时候，需要将函数体用一对花括号进行包裹，并明确定义一个返回值，正如下面这个版本的 `sum()` ：

```
var sum = (num1, num2) => {
    return num1 + num2;
};

// 有效等价于：

var sum = function(num1, num2) {
    return num1 + num2;
}; 
```

你基本可以将花括号内部的代码当做传统函数那样对待，除了 `arguments` 对象不可用之外。

若你想创建一个空函数，就必须使用空的花括号，就像这样：

```
var doNothing = () => {};

// 有效等价于：

var doNothing = function() {}; 
```

花括号被用于表示函数的主体，它在你至今看到的例子中都工作正常。但若箭头函数想要从函数体内向外返回一个对象字面量，就必须将该字面量包裹在圆括号内，例如：

```
var getTempItem = id => ({ id: id, name: "Temp" });

// 有效等价于：

var getTempItem = function(id) {

    return {
        id: id,
        name: "Temp"
    };
}; 
```

将对象字面量包裹在括号内，标示了括号内是一个字面量而不是函数体。

#### 创建立即调用函数表达式

JS 中使用函数的一种流行方式是创建立即调用函数表达式（ immediately-invoked function expression ， IIFE ）。 IIFE 允许你定义一个匿名函数并在未保存引用的情况下立刻调用它。当你想创建一个作用域并隔离在程序其他部分外，这种模式就很有用了。例如：

```
let person = function(name) {

    return {
        getName: function() {
            return name;
        }
    };

}("Nicholas");

console.log(person.getName());      // "Nicholas" 
```

此代码中 IIFE 被用于创建一个包含 `getName()` 方法的对象。该方法使用 `name` 参数作为返回值，有效地让 `name` 成为所返回对象的一个私有成员。

你可以使用箭头函数来完成同样的事情，只要将其包裹在括号内即可：

```
let person = ((name) => {

    return {
        getName: function() {
            return name;
        }
    };

})("Nicholas");

console.log(person.getName());      // "Nicholas" 
```

需要注意的是括号仅包裹了箭头函数的定义，并未包裹 `("Nicholas")` 。这有别于使用传统函数时的方式——括号既可以连函数定义与参数调用一起包裹，也可以只用于包裹函数定义。

> 译注：使用传统函数时， `(function(){/*函数体*/})();` 与 `(function(){/*函数体*/}());` 这两种方式都是可行的。
> 
> 但若使用箭头函数，则只有下面的写法是有效的： `(() => {/*函数体*/})();`

#### 没有 this 绑定

JS 最常见的错误领域之一就是在函数内的 `this` 绑定。由于一个函数内部的 `this` 值可以被改变，这取决于调用该函数时的上下文，因此完全可能错误地影响了一个对象，尽管你本意是要修改另一个对象。研究如下例子：

```
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click", function(event) {
            this.doSomething(event.type);     // 错误
        }, false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
}; 
```

此代码的 `PageHandler` 对象被设计用于处理页面上的交互。 `init()` 方法被调用以建立该交互，并注册了一个事件处理函数来调用 `this.doSomething()` 。然而此代码并未按预期工作。

调用 `this.doSomething()` 被中断是因为 `this` 是对事件目标对象（在此案例中就是 `document` ）的一个引用，而不是被绑定到 `PageHandler` 上。若试图运行此代码，你将会在事件处理函数被触发时得到一个错误，因为 `this.doSomething()` 并不存在于 `document` 对象上。

你可以明确使用 `bind()` 方法将函数的 `this` 值绑定到 `PageHandler` 上，以修正这段代码，就像这样：

```
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click", (function(event) {
            this.doSomething(event.type);     // 没有错误
        }).bind(this), false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
}; 
```

现在此代码能像预期那样运行，但看起来有点奇怪。通过调用 `bind(this)` ，你实际上创建了一个新函数，它的 `this` 被绑定到当前 `this` （也就是 `PageHandler` ）上。为了避免额外创建一个函数，修正此代码的更好方式是使用箭头函数。

箭头函数没有 `this` 绑定，意味着箭头函数内部的 `this` 值只能通过查找作用域链来确定。如果箭头函数被包含在一个非箭头函数内，那么 `this` 值就会与该函数的相等；否则， `this` 值就会是 `undefined` 。你可以使用箭头函数来书写如下代码：

```
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click",
                event => this.doSomething(event.type), false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
}; 
```

本例中的事件处理函数是一个调用 `this.doSomething()` 的箭头函数，它的 `this` 值与 `init()` 方法的相同，因此这个版本的代码的工作方式类似于使用了 `bind(this)` 的上个例子。尽管 `doSomething()` 方法并不返回任何值，它仍然是函数体内唯一被执行的语句，因此无须使用花扩花来包裹它。

箭头函数被设计为“抛弃型”的函数，因此不能被用于定义新的类型； `prototype` 属性的缺失让这个特性显而易见。对箭头函数使用 `new` 运算符会导致错误，正如下例：

```
var MyType = () => {},
    object = new MyType();  // 错误：你不能对箭头函数使用 'new' 
```

此代码调用 `new MyType()` 的操作失败了，由于 `MyType()` 是一个箭头函数，它就不存在 `[[Construct]]` 方法。了解箭头函数不能被用于 `new` 的特性后， JS 引擎就能进一步对其进行优化。

同样，由于箭头函数的 `this` 值由包含它的函数决定，因此不能使用 `call()` 、 `apply()` 或 `bind()` 方法来改变其 `this` 值。

#### 箭头函数与数组

箭头函数的简洁语法也让它成为进行数组操作的理想选择。例如，若你想使用自定义比较器来对数组进行排序，通常会这么写：

```
var result = values.sort(function(a, b) {
    return a - b;
}); 
```

这里为一个非常简单的工序使用了过多代码，可以比较一下使用了箭头函数的更简洁版本：

```
var result = values.sort((a, b) => a - b); 
```

能使用回调函数的数组方法（例如 `sort()` 、 `map()` 与 `reduce()` 方法），都能从箭头函数的简洁语法中获得收益，它将看似复杂的需求转换为简单的代码。

#### 没有 arguments 绑定

尽管箭头函数没有自己的 `arguments` 对象，但仍然能访问包含它的函数的 `arguments` 对象。无论此后箭头函数在何处执行，该对象都是可用的。例如：

```
function createArrowFunctionReturningFirstArg() {
    return () => arguments[0];
}

var arrowFunction = createArrowFunctionReturningFirstArg(5);

console.log(arrowFunction());       // 5 
```

在 `createArrowFunctionReturningFirstArg()` 内部， `arguments[0]` 元素被已创建的箭头函数 `arrowFunction` 所引用，该引用包含了传递给 `createArrowFunctionReturningFirstArg()` 函数的首个参数。当箭头函数在此后被执行时，它返回了 5 ，这也正是传递给 `createArrowFunctionReturningFirstArg()` 的首个参数。尽管箭头函数 `arrowFunction` 已不在创建它的函数的作用域内，但由于 `arguments` 标识符的作用域链解析， `arguments` 对象依然可被访问。

#### 识别箭头函数

尽管语法不同，但箭头函数依然属于函数，并能被照常识别。研究如下代码：

```
var comparator = (a, b) => a - b;

console.log(typeof comparator);                 // "function"
console.log(comparator instanceof Function);    // true 
```

`console.log()` 的输出揭示了 `typeof` 与 `instanceof` 在作用于箭头函数时的行为，与作用在其他函数上完全一致。

也像对其他函数那样，你仍然可以对箭头函数使用 `call()` 、 `apply()` 与 `bind()` 方法，虽然函数的 `this` 绑定并不会受影响。这里有几个例子：

```
var sum = (num1, num2) => num1 + num2;

console.log(sum.call(null, 1, 2));      // 3
console.log(sum.apply(null, [1, 2]));   // 3

var boundSum = sum.bind(null, 1, 2);

console.log(boundSum());                // 3 
```

`sum()` 函数被使用 `call()` 与 `apply()` 方法调用并传递了参数，就像对其他函数所做的那样。 `bind()` 方法被用于创建 `boundSum()` ，后者的两个参数已被绑定为 `1` 与 `2` ，因此不再需要直接传入这两个参数。

箭头函数能在任意位置替代你当前使用的匿名函数，例如回调函数。下一节涵盖的内容是 ES6 的另一项主要进展，不过该内容完全是内部实现，并没有使用新语法。

### 尾调用优化

在 ES6 中对函数最有趣的改动或许就是一项引擎优化，它改变了尾部调用的系统。**尾调用**（ **tail call** ）指的是调用函数的语句是另一个函数的最后语句，就像这样：

```
function doSomething() {
    return doSomethingElse();   // 尾调用
} 
```

在 ES5 引擎中实现的尾调用，其处理就像其他函数调用一样：一个新的栈帧（ stack frame ）被创建并推到调用栈之上，用于表示该次函数调用。这意味着之前每个栈帧都被保留在内存中，当调用栈太大时会出问题。

#### 有何不同？

ES6 在严格模式下力图为特定尾调用减少调用栈的大小（非严格模式的尾调用则保持不变）。当满足以下条件时，尾调用优化会清除当前栈帧并再次利用它，而不是为尾调用创建新的栈帧：

1.  尾调用不能引用当前栈帧中的变量（意味着该函数不能是闭包）；
2.  进行尾调用的函数在尾调用返回结果后不能做额外操作；
3.  尾调用的结果作为当前函数的返回值。

作为一个例子，下面代码满足了全部三个条件，因此能被轻易地优化：

```
"use strict";

function doSomething() {
    // 被优化
    return doSomethingElse();
} 
```

该函数对 `doSomethingElse()` 进行了一次尾调用，并立即返回了其结果，同时并未访问局部作用域的任何变量。一个小改动——不返回结果，就会产生一个无法被优化的函数：

```
"use strict";

function doSomething() {
    // 未被优化：缺少 return
    doSomethingElse();
} 
```

类似的，如果你的函数在尾调用返回结果之后进行了额外操作，那么该函数也无法被优化：

```
"use strict";

function doSomething() {
    // 未被优化：在返回之后还要执行加法
    return 1 + doSomethingElse();
} 
```

此例在 `doSomethingElse()` 的结果上对其进行了加 1 操作，而没有直接返回该结果，这已足以关闭优化。

无意中关闭优化的另一个常见方式，是将函数调用的结果储存在一个变量上，之后才返回了结果，就像这样：

```
"use strict";

function doSomething() {
    // 未被优化：调用并不在尾部
    var result = doSomethingElse();
    return result;
} 
```

本例之所以不能被优化，是因为 `doSomethingElse()` 的值并没有立即被返回。

使用闭包或许就是需要避免的最困难情况，因为闭包能够访问上层作用域的变量，会导致尾调用优化被关闭。例如：

```
"use strict";

function doSomething() {
    var num = 1,
        func = () => num;

    // 未被优化：此函数是闭包
    return func();
} 
```

此例中闭包 `func()` 需要访问局部变量 `num` ，虽然调用 `func()` 后立即返回了其结果，但是对于 `num` 的引用导致优化不会发生。

#### 如何控制尾调用优化

在实践中，尾调用优化在后台进行，所以不必对此考虑太多，除非要尽力去优化一个函数。尾调用优化的主要用例是在递归函数中，而且在其中的优化具有最大效果。考虑以下计算阶乘的函数：

```
function factorial(n) {

    if (n <= 1) {
        return 1;
    } else {

        // 未被优化：在返回之后还要执行乘法
        return n * factorial(n - 1);
    }
} 
```

此版本的函数并不会被优化，因为在递归调用 `factorial()` 之后还要执行乘法运算。如果 `n` 是一个大数字，那么调用栈的大小会增长，并且可能导致堆栈溢出。

为了优化此函数，你需要确保在最后的函数调用之后不会发生乘法运算。为此你可以使用一个默认参数来将乘法操作移出 `return` 语句。有结果的函数携带着临时结果进入下一次迭代，这样创建的函数的功能与前例相同，但它能被 ES6 的引擎所优化。此处是新的代码：

```
function factorial(n, p = 1) {

    if (n <= 1) {
        return 1 * p;
    } else {
        let result = n * p;

        // 被优化
        return factorial(n - 1, result);
    }
} 
```

在重写的 `factorial()` 函数中，添加了第二个参数 `p` ，其默认值为 1 。 `p` 参数保存着前一次乘法的结果，因此下一次的结果就能在进行函数调用之前被算出。当 `n` 大于 1 时，会先进行乘法运算并将其结果作为第二个参数传入 `factorial()` 。这就允许 ES6 引擎去优化这个递归调用。

尾调用优化是你在书写任意递归函数时都需要考虑的因素，因为它能提供显著的性能提升，尤其是被应用到计算复杂度很高的函数时。

### 总结

函数在 ES6 中并未经历巨大变化，然而一系列增量改进使得函数更易使用。

在特定参数未被传入时，函数的默认参数允许你更容易指定需要使用的值。而在 ES6 之前，这要求在函数内使用一些额外代码，以便检查参数是已否提供并为其分配一个不同的值。

剩余参数允许你将余下的所有参数放入指定数组。使用真正的数组并让你指定哪些参数需要被包含，使得剩余参数成为比 `arguments` 更为灵活的解决方案。

扩展运算符是剩余参数的好伙伴，允许在调用函数时将数组解构为分离的参数。在 ES6 之前，要把数组的元素作为独立参数传给函数只有两种办法：手动指定每一个参数，或者使用 `apply()` 方法。有了扩展运算符，你就能轻易将数组传递给函数而无须担心该函数的 `this` 绑定。

新增的 `name` 属性能帮你在调试与执行方面更容易地识别函数。此外， ES6 正式定义了块级函数的行为，因此在严格模式下它们不再是语法错误。

在 ES6 中，函数的行为被 `[[Call]]` 与 `[[Construct]]` 方法所定义，前者对应普通的函数执行，后者则对应着使用了 `new` 的调用。 `new.target` 元属性也能用于判断函数被调用时是否使用了 `new` 。

ES6 函数的最大变化就是增加了箭头函数。箭头函数被设计用于替代匿名函数表达式，它拥有更简洁的语法、词法级的 `this` 绑定，并且没有 `arguments` 对象。此外，箭头函数不能修改它们的 `this` 绑定，因此不能被用作构造器。

尾调用优化允许某些函数的调用被优化，以保持更小的调用栈、使用更少的内存，并防止堆栈溢出。当能进行安全优化时，它会由引擎自动应用。不过你可以考虑重写递归函数，以便能够利用这种优化。