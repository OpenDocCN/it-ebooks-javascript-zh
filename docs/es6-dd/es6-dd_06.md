# 深入浅出 ES6（六）：解构 Destructuring

作者 Nick Fitzgerald ，译者 刘振涛

> *编者按：ECMAScript 6 已经正式发布了，作为它最重要的方言，Javascript 也即将迎来语法上的重大变革，InfoQ 特开设“[深入浅出 ES6](http://www.infoq.com/cn/es6-in-depth/)”专栏，来看一下 ES6 将给我们带来哪些新内容。本专栏文章来自[Mozilla Web 开发者博客](https://hacks.mozilla.org/category/es6-in-depth/https:/hacks.mozilla.org/category/es6-in-depth/)，由作者授权翻译并发布。*

*译者按：Firefox 开发者工具工程师[Nick Fitzgerald](http://fitzgeraldnick.com/)最初在他的博客上发布了一篇文章——《[ES6 中的解构赋值](http://fitzgeraldnick.com/weblog/50/)》，本文基于这篇博文做了些许补充。*

## 什么是解构赋值？

解构赋值允许你使用类似数组或对象字面量的语法将数组和对象的属性赋给各种变量。这种赋值语法极度简洁，同时还比传统的属性访问方法更为清晰。

通常来说，你很可能这样访问数组中的前三个元素：

```
 var first = someArray[0];
    var second = someArray[1];
    var third = someArray[2];
```

如果使用解构赋值的特性，将会使等效的代码变得更加简洁并且可读性更高：

```
 var [first, second, third] = someArray;
```

SpiderMonkey（Firefox 的 JavaScript 引擎）已经支持解构的大部分功能，但是仍不健全。你可以通过[bug 694100](https://bugzilla.mozilla.org/show_bug.cgi?id=694100)跟踪解构和其它 ES6 特性在 SpiderMonkey 中的支持情况。

## 数组与迭代器的解构

以上是数组解构赋值的一个简单示例，其语法的一般形式为：

```
 [ variable1, variable2, ..., variableN ] = array;
```

这将为 variable1 到 variableN 的变量赋予数组中相应元素项的值。如果你想在赋值的同时声明变量，可在赋值语句前加入`var`、`let`或`const`关键字，例如：

```
 var [ variable1, variable2, ..., variableN ] = array;
    let [ variable1, variable2, ..., variableN ] = array;
    const [ variable1, variable2, ..., variableN ] = array;
```

事实上，用`变量`来描述并不恰当，因为你可以对任意深度的嵌套数组进行解构：

```
 var [foo, [[bar], baz]] = [1, [[2], 3]];
    console.log(foo);
    // 1
    console.log(bar);
    // 2
    console.log(baz);
    // 3
```

此外，你可以在对应位留空来跳过被解构数组中的某些元素：

```
 var [,,third] = ["foo", "bar", "baz"];
    console.log(third);
    // "baz"
```

而且你还可以通过“[不定参数](http://www.infoq.com/cn/articles/es6-in-depth-rest-parameters-and-defaults)”模式捕获数组中的所有尾随元素：

```
 var [head, ...tail] = [1, 2, 3, 4];
    console.log(tail);
    // [2, 3, 4]
```

当访问空数组或越界访问数组时，对其解构与对其索引的行为一致，最终得到的结果都是：`undefined`。

```
 console.log([][0]);
    // undefined
    var [missing] = [];
    console.log(missing);
    // undefined
```

请注意，数组解构赋值的模式同样适用于任意迭代器：

```
 function* fibs() {
      var a = 0;
      var b = 1;
      while (true) {
        yield a;
        [a, b] = [b, a + b];
      }
    }
    var [first, second, third, fourth, fifth, sixth] = fibs();
    console.log(sixth);
    // 5
```

## 对象的解构

通过解构对象，你可以把它的每个属性与不同的变量绑定，首先指定被绑定的属性，然后紧跟一个要解构的变量。

```
 var robotA = { name: "Bender" };
    var robotB = { name: "Flexo" };
    var { name: nameA } = robotA;
    var { name: nameB } = robotB;
    console.log(nameA);
    // "Bender"
    console.log(nameB);
    // "Flexo"
```

当属性名与变量名一致时，可以通过一种实用的句法简写：

```
 var { foo, bar } = { foo: "lorem", bar: "ipsum" };
    console.log(foo);
    // "lorem"
    console.log(bar);
    // "ipsum"
```

与数组解构一样，你可以随意嵌套并进一步组合对象解构：

```
 var complicatedObj = {
      arrayProp: [
        "Zapp",
        { second: "Brannigan" }
      ]
    };
    var { arrayProp: [first, { second }] } = complicatedObj;
    console.log(first);
    // "Zapp"
    console.log(second);
    // "Brannigan"
```

当你解构一个未定义的属性时，得到的值为`undefined`：

```
 var { missing } = {};
    console.log(missing);
    // undefined
```

请注意，当你解构对象并赋值给变量时，如果你已经声明或不打算声明这些变量（亦即赋值语句前没有`let`、`const`或`var`关键字），你应该注意这样一个潜在的语法错误：

```
 { blowUp } = { blowUp: 10 };
    // Syntax error 语法错误
```

为什么会出错？这是因为 JavaScript 语法通知解析引擎将任何以{开始的语句解析为一个块语句（例如，`{console}`是一个合法块语句）。解决方案是将整个表达式用一对小括号包裹：

```
 ({ safe } = {});
    // No errors 没有语法错误
```

## 解构值不是对象、数组或迭代器

当你尝试解构`null`或`undefined`时，你会得到一个类型错误：

```
 var {blowUp} = null;
    // TypeError: null has no properties（null 没有属性）
```

然而，你可以解构其它原始类型，例如：`布尔值`、`数值`、`字符串`，但是你将得到`undefined`：

```
 var {wtf} = NaN;
    console.log(wtf);
    // undefined
```

你可能对此感到意外，但经过进一步审查你就会发现，原因其实非常简单。当使用对象赋值模式时，被解构的值[需要被强制转换为对象](https://people.mozilla.org/~jorendorff/es6-draft.html#sec-requireobjectcoercible)。大多数类型都可以被转换为对象，但`null`和`undefined`却无法进行转换。当使用数组赋值模式时，被解构的值一定要[包含一个迭代器](https://people.mozilla.org/~jorendorff/es6-draft.html#sec-getiterator)。

## 默认值

当你要解构的属性未定义时你可以提供一个默认值：

```
 var [missing = true] = [];
    console.log(missing);
    // true
    var { message: msg = "Something went wrong" } = {};
    console.log(msg);
    // "Something went wrong"
    var { x = 3 } = {};
    console.log(x);
    // 3
```

*(译者按：Firefox 目前只实现了这个特性的前两种情况，第三种尚未实现。详情查看[bug 932080](https://bugzilla.mozilla.org/show_bug.cgi?id=932080)。)*

## 解构的实际应用

### 函数参数定义

作 为开发者，我们需要实现设计良好的 API，通常的做法是为函数为函数设计一个对象作为参数，然后将不同的实际参数作为对象属性，以避免让 API 使用者记住 多个参数的使用顺序。我们可以使用解构特性来避免这种问题，当我们想要引用它的其中一个属性时，大可不必反复使用这种单一参数对象。

```
 function removeBreakpoint({ url, line, column }) {
      // ...
    }
```

这是一段来自 Firefox 开发工具 JavaScript 调试器（同样使用 JavaScript 实现——没错，就是这样！）的代码片段，它看起来非常简洁，我们会发现这种代码模式特别讨喜。

### 配置对象参数

延伸一下之前的示例，我们同样可以给需要解构的对象属性赋予默认值。当我们构造一个提供配置的对象，并且需要这个对象的属性携带默认值时，解构特性就派上用场了。举个例子，jQuery 的`ajax`函数使用一个配置对象作为它的第二参数，我们可以这样重写函数定义：

```
 jQuery.ajax = function (url, {
      async = true,
      beforeSend = noop,
      cache = true,
      complete = noop,
      crossDomain = false,
      global = true,
      // ... 更多配置
    }) {
      // ... do stuff
    };
```

如此一来，我们可以避免对配置对象的每个属性都重复`var foo = config.foo || theDefaultFoo;`这样的操作。

*（编者按：不幸的是，对象的默认值简写语法仍未在 Firefox 中实现，我知道，上一个编者按后的几个段落讲解的就是这个特性。点击[bug 932080](https://bugzilla.mozilla.org/show_bug.cgi?id=932080)查看最新详情。）*

### 与 ES6 迭代器协议协同使用

ECMAScript 6 中定义了一个[迭代器协议](https://hacks.mozilla.org/2015/04/es6-in-depth-iterators-and-the-for-of-loop/)，我们在《[深入浅出 ES6（二）：迭代器和 for-of 循环](http://www.infoq.com/cn/articles/es6-in-depth-iterators-and-the-for-of-loop)》中已经详细解析过。当你迭代[Maps](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)（ES6 标准库中新加入的一种对象）后，你可以得到一系列形如`[key, value]`的键值对，我们可将这些键值对解构，更轻松地访问键和值：

```
 var map = new Map();
    map.set(window, "the global");
    map.set(document, "the document");
    for (var [key, value] of map) {
      console.log(key + " is " + value);
    }
    // "[object Window] is the global"
    // "[object HTMLDocument] is the document"
```

只遍历键：

```
 for (var [key] of map) {
      // ...
    }
```

或只遍历值：

```
 for (var [,value] of map) {
      // ...
    }
```

### 多重返回值

JavaScript 语言中尚未整合多重返回值的特性，但是无须多此一举，因为你自己就可以返回一个数组并将结果解构：

```
 function returnMultipleValues() {
      return [1, 2];
    }
    var [foo, bar] = returnMultipleValues();
```

或者，你可以用一个对象作为容器并为返回值命名：

```
 function returnMultipleValues() {
      return {
        foo: 1,
        bar: 2
      };
    }
    var { foo, bar } = returnMultipleValues();
```

这两个模式都比额外保存一个临时变量要好得多。

```
 function returnMultipleValues() {
      return {
        foo: 1,
        bar: 2
      };
    }
    var temp = returnMultipleValues();
    var foo = temp.foo;
    var bar = temp.bar;
```

或者使用 CPS 变换：

```
 function returnMultipleValues(k) {
      k(1, 2);
    }
    returnMultipleValues((foo, bar) => ...);
```

### 使用解构导入部分 CommonJS 模块

你是否尚未使用 ES6 模块？还用着 CommonJS 的模块呢吧！没问题，当我们导入 CommonJS 模块 X 时，很可能在模块 X 中导出了许多你根本没打算用的函数。通过解构，你可以显式定义模块的一部分来拆分使用，同时还不会污染你的命名空间：

```
 const { SourceMapConsumer, SourceNode } = require("source-map");
```

（如果你使用 ES6 模块，你一定知道在`import`声明中有一个相似的语法。）

## 文后盘点

所以，正如你所见，解构在许多独立小场景中非常实用。在 Mozilla 我们已经积累了许多有关解构的使用经验。十年前，Lars Hansen 在 Opera 中引入了 JS 解构特性，Brendan Eich 随后就给 Firefox 也增加了相应的支持，移植时版本为 Firefox 2。所以我们可以肯定，渐渐地，你会在每天使用的语言中加入解构这个新特性，它可以让你的代码变得更加精简整洁。

在第一篇文章中，我说过 ES6 很可能改变你写 JavaScript 的方式。这正是我日思夜想的特性：轻松学习，简单改进，合力出击，优化项目，在不断的进化中改革这门语言。

感谢团队对于整个 ES6 解构特性的努力，特别感谢 Tooru Fujisawa（arai）和 Arpad Borsos（Swatinem）的出色贡献。

Chrome 中有关解构的支持正在开发中，其它浏览器也将适时增加支持。现在，如果你想在 Web 上使用解构功能，你需要使用[Babel](http://babeljs.io/)或[Traceur](https://github.com/google/traceur-compiler#what-is-traceur)将 ES6 代码转译为相应的 ES5 代码。

* * *

*再次感谢 Nick Fitzgerald 撰写 ES6 的解构特性。*

*下一次，我们将讲解一个新特性，函数一直以来都是 JavaScript 语言的基础构建单元，如果我们用稍短的方式实现函数是否令你感到激动？我自信地预测你 的答案一定是 yes！不过别轻易相信我说的话，发布下一篇文章时记得亲自来看看，我们一起深入浅出 ES6 箭头函数（Arrow Function）。*

查看原文：[深入浅出 ES6（六）：解构 Destructuring](http://www.infoq.com/cn/articles/es6-in-depth-destructuring)

