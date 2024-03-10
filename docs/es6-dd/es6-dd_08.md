# 深入浅出 ES6（八）：Symbols

作者 Jason Orendorff ，译者 刘振涛

> 编者按：ECMAScript 6 已经正式发布了，作为它最重要的方言，Javascript 也即将迎来语法上的重大变革，InfoQ 特开设“[深入浅出 ES6](http://www.infoq.com/cn/es6-in-depth/)”专栏，来看一下 ES6 将给我们带来哪些新内容。本专栏文章来自[Mozilla Web 开发者博客](https://hacks.mozilla.org/category/es6-in-depth/https:/hacks.mozilla.org/category/es6-in-depth/)，由作者授权翻译并发布。

你是否知道 ES6 中的 Symbols 是什么，它有什么作用呢？我相信你很可能不知道，那就让我们一探究竟！

Symbols 并非用来指代某种 Logo。

它们也不是可以用作代码的小图标。

![](img/120150824164546.png)

它们不是代替其它东西的文学手法。

它们更不可能被用来指代谐音词 Cymbals（铙钹）。

![](img/302555333_c3f827fd75_z-250x188.jpg)

（编程的时候最好不要演奏铙钹，它们太过吵闹，很可能导致你的程序崩溃。）

那么，Symbols 到底是什么呢？

## 它是 JavaScript 的第七种原始类型

1997 年 JavaScript 首次被标准化，那时只有六种原始类型，在 ES6 以前，JS 程序中使用的每一个值都是以下几种类型之一：

*   Undefined 未定义
*   Null 空值
*   Boolean 布尔类型
*   Number 数字类型
*   String 字符串类型
*   Object 对象类型

每种类型都是多个值的集合，前五个集合是有限的。布尔类型只有两个值，`true`和`false`，不会再创造第三种布尔值；数字类型和字符串类型的值更多，标准指明一共有 18,437,736,874,454,810,627 种不同的数字（包括`NaN`， 亦即“Not a Number”的缩写，代表非数字），可能存在的字符串类型的值拥有无以匹敌的数量，我估算了一下大约是 (2144,115,188,075,855,872 − 1) ÷ 65,535 种……当然，我很可能得出了一个错误的答案，但字符串类型值的集合一定是有限的。

然而，对象类型值的集合是无限的。每一个对象都像珍贵的雪花一样独一无二，每一次你打开一个 Web 页面，都会创建一堆对象。

ES6 新特性中的 symbol 也是值，但它不是字符串，也不是对象，而是是全新的——第七种类型的原始值。

让我们一起探讨一下 symbol 的实际应用场景。

## 从一个简单的布尔类型出发

有时候你可以非常轻松地将别人的外部数据存储到一个 JavaScript 对象中。

举 个例子，假设你正在写一个 JS 库，可以通过 CSS transitions 使 DOM 元素在屏幕上移动。你可能会注意到，当你尝试在一个 div 元素上同时应用多重 CSS transitions 时并不会生效。实际效果是丑陋而又不连续的“跳闪”。你认为可以修复这个问题，但前提是你需要一种发现给定元素是否已经移动过的方 法。

应当如何解决这个问题呢？

一种方法是，用 CSS API 来告诉浏览器元素是否正在移动，但这样简直小题大做。在元素移动的第一时间内你的库就应该记录下移动的状态，所以它自然知道元素正在移动。

你真正想要的是一种持续跟踪某个元素正在移动的方法。你可以维护一个数组，记录所有正在移动的元素，每当你的库被调用来移动某个元素时，你可以检索数组来查看元素是否已经存在，亦即它是否正在移动中。

当然，如果数组非常大的话，线性搜索将会非常缓慢。

实际上你只想为元素设置一个标记：

```js
 if (element.isMoving) {
      smoothAnimations(element);
    }
    element.isMoving = true;
```

这样也会有一些潜在的问题，事实上，你的代码很可能不是唯一一段操作 DOM 的代码。

1.  你创建的属性很可能影响到其它使用了`for-in`或`Object.keys()`的代码。
2.  一些聪明的库作者可能已经考虑并使用了这项技术，这样一来你的库就会与已有的库产生某些冲突
3.  当然，很可能你比他们更聪明，你先采用了这项技术，但是他们的库仍然无法与你的库默契配合。
4.  标准委员会可能决定为所有的元素增加一个.isMoving()方法，到那时你需要重写相关逻辑，必定会有深深的挫败感。

当然你可以选择一个乏味而愚蠢的命名（其他人根本不会想用的那些名称）来解决最后的三个问题：

```js
 if (element.__$jorendorff_animation_library$PLEASE_DO_NOT_USE_THIS_PROPERTY$isMoving__) {
      smoothAnimations(element);
    }
    element.__$jorendorff_animation_library$PLEASE_DO_NOT_USE_THIS_PROPERTY$isMoving__ = true;
```

这只会造成无畏的眼疲劳。

借助于密码学，你可以生成一个唯一的属性名称：

```js
 // 获取 1024 个 Unicode 字符的无意义命名
    var isMoving = SecureRandom.generateName();
    ...
    if (element[isMoving]) {
      smoothAnimations(element);
    }
    element[isMoving] = true;
```

`object[name]`语法允许你使用几乎任何字符串作为属性名称。所以这个方法行之有效：冲突几乎是不可能的，并且你的代码看起来也很简洁。

但是这也将带来不良的调试体验。每当你在控制台输出（`console.log()`）包含那个属性的元素时，你将会看到一堆巨大的字符串垃圾。假使你需要比这多得多的类似属性呢？你如何保持它们整齐划一？每当你重载的时候它们的命名甚至都不一样！

为什么这个问题如此困难？我们只想要一个小小的布尔值啊！

## symbol 是最终的解决方案

symbol 是程序创建并且可以用作属性键的值，并且它能避免命名冲突的风险。

```js
 var mySymbol = Symbol();
```

调用`Symbol()`创建一个新的 symbol，它的值与其它任何值皆不相等。

字符串或数字可以作为属性的键，symbol 也可以，它不等同于任何字符串，因而这个以 symbol 为键的属性可以保证不与任何其它属性产生冲突。

```js
 obj[mySymbol] = "ok!";  // 保证不会冲突
    console.log(obj[mySymbol]);  // ok!
```

想要在上述讨论的场景中使用 symbol，你可以这样做：

```js
 // 创建一个独一无二的 symbol
    var isMoving = Symbol("isMoving");
    ...
    if (element[isMoving]) {
      smoothAnimations(element);
    }
    element[isMoving] = true;
```

有关这段代码的一些解释：

*   `Symbol("isMoving")`中的`isMoving`被称作描述。你可以通过`console.log()`将它打印出来，对调试非常有帮助；你也可以用`.toString()`方法将它转换为字符串呈现；它也可以被用在错误信息中。

*   `element[isMoving]`被称作一个*以 symbol 为键（symbol-keyed）的属性*。简而言之，它的名字是`symbol`而不是一个字符串。除此之外，它与一个普通的属性没有什么区别。

*   以 symbol 为键的属性属性与数组元素类似，不能被类似`obj.name`的点号法访问，你必须使用方括号访问这些属性。

*   如果你已经得到了 symbol，那么访问一个以 symbol 为键的属性同样简单，以上的示例很好地展示了如何获取`element[isMoving]`的值以及如何为它赋值。如果我们需要，可以查看属性是否存在：`if (isMoving in element)`，也可以删除属性：`delete element[isMoving]`。

*   另一方面，只有当`isMoving`在当前作用域中时才会生效。这是 symbol 的弱封装机制：模块创建了几个 symbol，可以在任意对象上使用，**无须担心**与其它代码创建的属性产生冲突。

symbol 键的设计初衷是避免初衷，因此 JavaScript 中最常见的对象检查的特性会忽略 symbol 键。例如，`for-in`循环只会遍历对象的字符串键，symbol 键直接跳过，`Object.keys(obj)`和`Object.getOwnPropertyNames(obj)`也是一样。但是 symbols 也不完全是私有的：用新的 API `Object.getOwnPropertySymbols(obj)`就可以列出对象的 symbol 键。另一个新的 API，`Reflect.ownKeys(obj)`，会同时返回字符串键和 symbol 键。（我们将在随后的文章中讲解 Reflect(反射) API）。

慢慢地我们会发现，越来越多的库和框架将大量使用 symbol，语言本身也会将 symbol 应用于广泛的用途。

## 但是，到底什么是 symbol 呢？

```js
 > typeof Symbol()
    "symbol"
```

确切地说，symbol 与其它类型并不完全相像。

symbol 被创建后就不可变更，你不能为它设置属性（在严格模式下尝试设置属性会得到 TypeError 的错误）。他们可以用作属性名称，这些性质与字符串类似。

另一方面，每一个 symbol 都独一无二，不与其它 symbol 等同，即使二者有相同的描述也不相等；你可以轻松地创建一个新的 symbol。这些性质与对象类似。

ES6 中的 symbol 与 Lisp 和 Ruby 这些语言中[更传统的 symbol](https://en.wikipedia.org/wiki/Symbol_%28programming%29)类似，但不像它们集成得那么紧密。在 Lisp 中，所有的标识符都是 symbol；在 JS 中，标识符和大多数的属性键仍然是字符串，symbol 只是一个额外的选项。

关于 symbol 的忠告：symbol 不能被自动转换为字符串，这和语言中的其它类型不同。尝试拼接 symbol 与字符串将得到 TypeError 错误。

```js
 > var sym = Symbol("<3");
    > "your symbol is " + sym
    // TypeError: can't convert symbol to string
    > `your symbol is ${sym}`
    // TypeError: can't convert symbol to string
```

通过`String(sym)`或`sym.toString()`可以显示地将 symbol 转换为一个字符串，从而回避这个问题。

## 获取 symbol 的三种方法

有三种获取 symbol 的方法。

*   **调用 Symbol()。**正如我们上文中所讨论的，这种方式每次调用都会返回一个新的唯一 symbol。

*   **调用 Symbol.for(string)。**这种方式会访问 symbol 注册表，其中存储了已经存在的一系列 symbol。这种方式与通过`Symbol()`定义的独立 symbol 不同，symbol 注册表中的 symbol 是共享的。如果你连续三十次调用`Symbol.for("cat")`，每次都会返回相同的 symbol。注册表非常有用，在多个 web 页面或同一个 web 页面的多个模块中经常需要共享一个 symbol。

*   **使用标准定义的 symbol，例如：Symbol.iterator。**标准根据一些特殊用途定义了少许的几个 symbol。

如果你尚不确定 symbol 是否实用，最后这一章将向你展示 symbol 在实际应用中发挥的巨大作用，非常有趣！

## symbol 在 ES6 规范中的应用

在之前的文章《[深入浅出 ES6（二）：迭代器和 for-of 循环](http://www.infoq.com/cn/articles/es6-in-depth-iterators-and-the-for-of-loop)》中，我们已经领略了借助 ES6 symbol 的力量避免代码冲突的方法，循环`for (var item of myArray)`首先调用`myArray[Symbol.iterator]()`，当时我提到这种写法是为了替代`myArray.iterator()`，拥有更好的向后兼容性。

现在我们知道 symbol 到底是什么了，自然很容易理解为什么我们要创造一个 symbol 以及它为我们带来什么新特性。

ES6 中还有其它几处使用了 symbol 的地方。（这些特性在 Firefox 里尚未实现。）

*   **使 instanceof 可扩展。**在 ES6 中，表达式`object instanceof constructor`被指定为构造函数的一个方法：`constructorSymbol.hasInstance`。这意味着它是可扩展的。

*   **消除新特性和旧代码之间的冲突。**这一点非常复杂，但是我们发现，添加某些 ES6 数组方法会破坏现有的 Web 网站。其它 Web 标准有相同的问题：向浏览器中添加新方法会破坏原有的网站。然而，破坏问题主要由动态作用域引起，所以 ES6 引入一个特殊的 symbol——`Symbol.unscopables`，Web 标准可以用这个 symbol 来阻止某些方法别加入到动态作用域中。

*   **支持新的字符串匹配类型。**在 ES5 中，`str.match(myObject)`会尝试将`myObject`转换为正则表达式对象（`RegExp`）。在 ES6 中，它会首先检查`myObject`是否有一个`myObjectSymbol.match`方法。现在的库可以提供自定义的字符串解析类，所有支持`RegExp`对象的环境都可以正常运行。

这些用例的应用范围都非常小，很难看到这些特性通过它们自身影响我们每日的代码，长期来看才能体现它们的价值。实际上，symbol 是 PHP 和 Python 中的`__doubleUnderscores`在 JavaScript 语言环境中的改进版。标准将借助 symbol 的力量在未来向语言中添加新的钩子，同时无风险地将新特性添加到你已有的代码中。

## 我何时可以使用 ES6 symbol？

symbol 在 Firefox 36 和 Chrome 38 中均已被实现。Firefox 中的实现由我亲自完成，所以如果你的 symbol 像铙钹（cymbals）一样行为异常，请直接联系我！

为了支持那些尚未支持原生 ES6 symbol 的浏览器，你可以使用一个 polyfill，例如[core.js](https://github.com/zloirock/core-js#ecmascript-6-symbols)。因为 symbol 与其它类型不尽相同，所以 polyfill 目前不是很完美。[请阅读注意事项](https://github.com/zloirock/core-js#caveats-when-using-symbol-polyfill)。

下一篇文章，我们将奉上一篇 Gastón Silva 的文章，讲解如何使用 Babel 和 Broccoli 来接触更多的 ES6 新特性，借鉴这篇文章的经验你可以轻松地开始 ES6 之旅。

接 下来，我们将深入浅出 Collections，这个特性被期待已久，最终在 ES6 版本加入到 JavaScript 中。我们将回溯到编程的起源，探索两个古 老的特性，紧接着讨论两个非常相似的特性，它们的生命周期短，但是威力巨大。所以请记得回来，一起探索接下来的旅程！到时见！

查看原文：[深入浅出 ES6（八）：Symbols](http://www.infoq.com/cn/articles/es6-in-depth-symbols)

