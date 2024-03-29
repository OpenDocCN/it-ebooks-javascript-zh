# 深入浅出 ES6（七）：箭头函数 Arrow Functions

作者 Jason Orendorff ，译者 刘振涛

> 译者按：ECMAScript 6 已经正式发布了，作为它最重要的方言，Javascript 也即将迎来语法上的重大变革，InfoQ 特开设“[深入浅出 ES6](http://www.infoq.com/cn/es6-in-depth/)”专栏，来看一下 ES6 将给我们带来哪些新内容。本专栏文章来自[Mozilla Web 开发者博客](https://hacks.mozilla.org/category/es6-in-depth/)，由作者授权翻译并发布。

箭头符号在 JavaScript 诞生时就已经存在，当初第一个 JavaScript 教程曾建议在 HTML 注释内包裹行内脚本，这样可以避免不支持 JS 的浏览器误将 JS 代码显示为文本。你会写这样的代码：

```js
 <script language="javascript">
    <!--
      document.bgColor = "brown";  // red
    // -->
    </script>
```

老式浏览器会将这段代码解析为两个不支持的标签和一条注释，只有新式浏览器才能识别出其中的 JS 代码。

为了支持这种奇怪的 hack 方式，浏览器中的 JavaScript 引擎将`<!--`这四个字符解析为单行注释的起始部分，我没开玩笑，这自始至终就是语言的一部分，直到现在仍然有效，这种注释符号不仅出现`<script>`标签后的首行，在 JS 代码的每个角落你都有可能见到它，甚至在 Node 中也是如此。

碰巧，[这种注释风格](http://people.mozilla.org/~jorendorff/es6-draft.html#sec-html-like-comments)首次在 ES6 中被标准化了，但在新标准中箭头被用来做其它事情。

箭头序列`-->`同样是单行注释的一部分。古怪的是，在 HTML 中`-->`之前的字符是注释的一部分，而在 JS 中`-->`之后的部分才是注释。

你一定感到陌生的是，只有当箭头在行首时才会注释当前行。这是因为在其它上下文中，`-->`是一个 JS 运算符：“趋向于”运算符！

```js
 function countdown(n) {
      while (n --> 0)  // "n goes to zero"
        alert(n);
      blastoff();
    }
```

上面这段代码[可以正常运行](http://codepen.io/anon/pen/oXZaBY?editors=001)，循环会一直重复直到`n`趋于 0，这当然不是 ES6 中的新特性，它只不过是将两个你早已熟悉的特性通过一些误导性的手段结合在一起。你能理解么？通常来说，类似这种谜团都可以在[Stack Overflow](http://stackoverflow.com/questions/1642028/what-is-the-name-of-the-operator)上找到答案。

当然，同样地，小于等于操作符`<=`也形似箭头，你可以在 JS 代码、隐藏的图片样式中找到更多类似的箭头，但是我们就不继续寻找了，你应该注意到我们漏掉了一种特殊的箭头。

| <!-- | `单行注释` |
| --> | `“趋向于”操作符` |
| <= | `小于等于` |
| => | `这又是什么？` |

`=>`到底是什么？我们今天就来一探究竟。

首先，我们谈论一些有关函数的事情。

## 函数表达式无处不在

JavaScript 中有一个有趣的特性，无论何时，当你需要一个函数时，你都可以在想添加的地方输入这个函数。

举个例子，假设你尝试告诉浏览器用户点击一个特定按钮后的行为，你会这样写：

```js
 $("#confetti-btn").click(
```

jQuery 的`.click()`方法接受一个参数：一个函数。没问题，你可以在这里输入一个函数：

```js
 $("#confetti-btn").click(function (event) {
      playTrumpet();
      fireConfettiCannon();
    });
```

对 于现在的我们来说，写出这样的代码相当自然，而回忆起在这种编程方式流行之前，这种写法相对陌生一些，许多语言中都没有这种特性。1958 年，Lisp 首 先支持函数表达式，也支持调用 lambda 函数，而 C++，Python、C#以及 Java 在随后的多年中一直不支持这样的特性。

现在截然不同，所有的四种语言都已支持 lambda 函数，更新出现的语言普遍都支持内建的 lambda 函数。我们必须要感谢 JavaScript 和早期的 JavaScript 程序员，他们勇敢地构建了重度依赖 lambda 函数的库，让这种特性被广泛接受。

令人伤感的是，随后在所有我提及的语言中，只有 JavaScript 的 lambda 的语法最终变得冗长乏味。

```js
 // 六种语言中的简单函数示例
    function (a) { return a > 0; } // JS
    [](int a) { return a > 0; }  // C++
    (lambda (a) (> a 0))  ;; Lisp
    lambda a: a > 0  # Python
    a => a > 0  // C#
    a -> a > 0  // Java
```

## 箭袋中的新羽

ES6 中引入了一种编写函数的新语法

```js
 // ES5
    var selected = allJobs.filter(function (job) {
      return job.isSelected();
    });
    // ES6
    var selected = allJobs.filter(job => job.isSelected());
```

当你只需要一个只有一个参数的简单函数时，可以使用新标准中的箭头函数，它的语法非常简单：`标识符=>表达式`。你无需输入`function`和`return`，一些小括号、大括号以及分号也可以省略。

（我个人对于这个特性非常感激，不再需要输入`function`这几个字符对我而言至关重要，因为我总是不可避免地错误写成`functoin`，然后我就不得不回过头改正它。）

如果要写一个接受多重参数（也可能没有参数，或者是[不定参数、默认参数](http://www.infoq.com/cn/articles/es6-in-depth-rest-parameters-and-defaults)、[参数解构](http://www.infoq.com/cn/articles/es6-in-depth-destructuring)）的函数，你需要用小括号包裹参数 list。

```js
 // ES5
    var total = values.reduce(function (a, b) {
      return a + b;
    }, 0);
    // ES6
    var total = values.reduce((a, b) => a + b, 0);
```

我认为这看起来酷毙了。

正如你使用类似[Underscore.js](http://underscorejs.org/)和[Immutable.js](https://facebook.github.io/immutable-js/)这样的库提供的函数工具，箭头函数运行起来同样美不可言。事实上，[Immutable 的文档](https://facebook.github.io/immutable-js/docs/#/)中的示例全都由 ES6 写成，其中的许多特性已经用上了箭头函数。

那么不是非常函数化的情况又如何呢？除表达式外，箭头函数还可以包含一个块语句。回想一下我们之前的示例：

```js
 // ES5
    $("#confetti-btn").click(function (event) {
      playTrumpet();
      fireConfettiCannon();
    });
```

这是它们在 ES6 中看起来的样子：

```js
 // ES6
    $("#confetti-btn").click(event => {
      playTrumpet();
      fireConfettiCannon();
    });
```

这是一个微小的改进，对于使用了[Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)的代码来说箭头函数的效果可以变得更加戏剧性，`}).then(function (result) {` 这样的一行代码可以堆积起来。

注意，使用了块语句的箭头函数不会自动返回值，你需要使用`return`语句将所需值返回。

*小提示：当使用箭头函数创建普通对象时，你总是需要将对象包裹在小括号里。*

```js
 // 为与你玩耍的每一个小狗创建一个新的空对象
    var chewToys = puppies.map(puppy => {});   // 这样写会报 Bug！
    var chewToys = puppies.map(puppy => ({})); //
```

用小括号包裹空对象就可以了。

不幸的是，一个空对象`{}`和一个空的块`{}`看起来完全一样。ES6 中的规则是，紧随箭头的{被解析为块的开始，而不是对象的开始。因此，`puppy => {}`这段代码就被解析为没有任何行为并返回`undefined`的箭头函数。

更令人困惑的是，你的 JavaScript 引擎会将类似`{key: value}`的对象字面量解析为一个包含标记语句的块。幸运的是，`{`是唯一一个有歧义的字符，所以用小括号包裹对象字面量是唯一一个你需要牢记的小窍门。

## 这个函数的 this 值是什么呢？

普通`function`函数和箭头函数的行为有一个微妙的区别，**箭头函数没有它自己的`this`值**，箭头函数内的`this`值继承自外围作用域。

在我们尝试说明这个问题前，先一起回顾一下。

JavaScript 中的`this`是如何工作的？它的值从哪里获取？[这些问题的答案可都不简单](http://stackoverflow.com/questions/3127429/how-does-the-this-keyword-work)，如果你对此倍感清晰，一定因为你长时间以来一直在处理类似的问题。

这个问题经常出现的其中一个原因是，无论是否需要，`function`函数总会自动接收一个`this`值。你是否写过这样的 hack 代码：

```js
 {
      ...
      addAll: function addAll(pieces) {
        var self = this;
        _.each(pieces, function (piece) {
          self.add(piece);
        });
      },
      ...
    }
```

在这里，你希望在内层函数里写的是`this.add(piece)`，不幸的是，内层函数并未从外层函数继承`this`的值。在内层函数里，`this`会是`window`或`undefined`，临时变量`self`用来将外部的`this`值导入内部函数。（另一种方式是在内部函数上执行`.bind(this)`，两种方法都不甚美观。）

在 ES6 中，不需要再 hack`this`了，但你需要遵循以下规则：

*   通过`object.method()`语法调用的方法使用非箭头函数定义，这些函数需要从调用者的作用域中获取一个有意义的`this`值。
*   其它情况全都使用箭头函数。

```js
 // ES6
    {
      ...
      addAll: function addAll(pieces) {
        _.each(pieces, piece => this.add(piece));
      },
      ...
    }
```

在 ES6 的版本中，注意`addAll`方法从它的调用者处获取了`this`值，内部函数是一个箭头函数，所以它继承了外围作用域的`this`值。

超赞的是，在 ES6 中你可以用更简洁的方式编写对象字面量中的方法，所以上面这段代码可以简化成：

```js
 // ES6 的方法语法
    {
      ...
      addAll(pieces) {
        _.each(pieces, piece => this.add(piece));
      },
      ...
    }
```

在方法和箭头函数之间，我再也不会错写`functoin`了，这真是一个绝妙的设计思想！

箭头函数与非箭头函数间还有一个细微的区别，箭头函数不会获取它们自己的`arguments`对象。诚然，在 ES6 中，你可能更多地会使用不定参数和默认参数值这些新特性。

## 借助箭头函数洞悉计算机科学的风尘往事

我们已经讨论了许多箭头函数的实际用例，它还有一种可能的使用方法：将 ES6 箭头函数作为一个学习工具，来深入挖掘计算的本质，是否实用，终将取决于你自己。

1936 年，Alonzo Church 和 Alan Turing 各自开发了强大的计算数学模型，图灵将他的模型称为 a-machines，但是每一个人都称其为图灵机。Church 写的是函数模型，他的模型被称为[lambda 演算](https://zh.wikipedia.org/wiki/%CE%9B%E6%BC%94%E7%AE%97)（[λ-calculus](https://en.wikipedia.org/wiki/Lambda_calculus)）。这一成果也被 Lisp 借鉴，用`LAMBDA`来指示函数，这也是为何我们现在将函数表达式称为 lambda 函数。

但什么是 lambda 演算呢？“计算模型”又意味着什么呢？

用 几句话解释清楚很难，但是我会努力阐释：lambda 演算是第一代编程语言的一种形式，但毕竟存储程序计算机在十几二十年后才诞生，所以它原本不是为编程 语言设计的，而是为了表达任意你想到的计算问题设计的一种极度简化的纯数学思想的语言。Church 希望用这个模型来证明普遍意义的计算。

最终他发现，在他的系统中只需要一件东西：函数。

这种声明方式无与伦比，不借助对象、数组、数字、`if`语句、`while`循环、分号、赋值、逻辑运算符甚或是事件循环，只须使用函数就可以从 0 开始重建 JavaScript 能实现的每一种计算。

这是用 Church 的 lambda 标记写出来的数学家风格的“程序”示例：

```js
 fix = λf.(λx.f(λv.x(x)(v)))(λx.f(λv.x(x)(v)))
```

等效的 JavaScript 函数是这样的：

```js
 var fix = f => (x => f(v => x(x)(v)))
                   (x => f(v => x(x)(v)));
```

所以，在 JavaScript 中实现了一个可以运行的 lambda 演算，它根植于这门语言中。

Alonzo Church 和 lambda 演算后继研究者们的故事，以及它是如何潜移默化地入驻每一门主流编程语言的，已经远超本文的讨论范围。但是如果你对计算机科学 的奠基感兴趣，或者你只是对一门只用函数就可以做许多类似循环和递归这样的事情的语言倍感兴趣，你可以在一个下雨的午后深入[邱奇数](https://zh.wikipedia.org/wiki/%E9%82%B1%E5%A5%87%E6%95%B0)（[Church numerals](https://en.wikipedia.org/wiki/Church_encoding)）和[不动点组合子](https://zh.wikipedia.org/wiki/%E4%B8%8D%E5%8A%A8%E7%82%B9%E7%BB%84%E5%90%88%E5%AD%90)（[Fixed-point combinator](https://en.wikipedia.org/wiki/Fixed-point_combinator#Strict_fixed_point_combinator)），在你的 Firefox 控制台或[Scratchpad](https://developer.mozilla.org/en-US/docs/Tools/Scratchpad)中仔细研究一番。结合 ES6 的箭头函数以及其它强大的功能，JavaScript 称得上是一门探索 lambda 演算的最好的语言。

## 我何时可以使用箭头函数？

早在 2013 年，我就在 Firefox 中实现了 ES6 箭头函数的功能，Jan de Mooij 为其优化加快了执行速度。感谢 Tooru Fujisawa 以及[ziyunfei](https://developer.mozilla.org/zh-CN/profiles/ziyunfei)（译者注：中国开发者，为 Mozilla 作了许多贡献）后续打的补丁。

微软 Edge 预览版中也实现了箭头函数的功能，如果你想立即在你的 Web 项目中使用箭头函数，可以使用[Babel](http://babeljs.io/)、[Traceur](https://github.com/google/traceur-compiler#what-is-traceur)或[TypeScript](http://www.typescriptlang.org/)，这三个工具均已实现相关功能。

我们的下一个话题是 ES6 中的一个非常陌生的特性，我们将一睹`typeof x`返回的全新的类型值。思考一下：不使用字符串如何命名？敬请期待我们下回分解，观众老爷们记得回来！

查看原文：[深入浅出 ES6（七）：箭头函数 Arrow Functions](http://www.infoq.com/cn/articles/es6-in-depth-arrow-functions)

