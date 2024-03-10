# 深入浅出 ES6（十六）：模块 Modules

作者 Jason Orendorff ，译者 刘振涛

*编者按：ECMAScript 6 已经正式发布了，作为它最重要的方言，Javascript 也即将迎来语法上的重大变革，InfoQ 特开设“[深入浅出 ES6](http://www.infoq.com/cn/es6-in-depth/)”专栏，来看一下 ES6 将给我们带来哪些新内容。本专栏文章来自[Mozilla Web 开发者博客](https://hacks.mozilla.org/category/es6-in-depth/)，由作者授权翻译并发布。*

早在 2007 年我刚加入 Mozilla 的 JavaScript 团队的时候广为流传一个笑话：通常来说 JavaScript 程序的长度只有一行。

那时候 Google Maps 的诞龄还只有两岁，在它诞生之前，JavaScript 主要被用来验证表单，毫无疑问，每一个处`理<input onchange=>`的程序真的就只有一行代码。

今 非昔比，JavaScript 项目的规模已经成长到令人惊叹的地步，社区也开发了很多工具来协助构建这些规模庞大的应用。诸多工具中最核心的非模块系统莫 属，在这样一个系统中，你可以通过多个文件和目录来组织你的项目，最重要的是，所有代码都可以按需彼此访问并高效加载。所以 JavaScript 就这么顺 理成章地拥也了几个知名的模块系统。当然，随之而来的还有几个包管理器，可以用它们安装所有软件以及处理高层次的依赖。如此一来，你一定会觉得 ES6 的模 块语法真是姗姗来迟啊。

好的，那今天我们就一起看看 ES6 中新增的模块系统，探讨一下我们可以通过新语法打造什么样的工具并进一步推动未来标准的发展。但是首先，我们还是来了解一下 ES6 模块的基础知识。

## 模块基础知识

每一个 ES6 模块都是一个包含 JS 代码的文件，模块本质上就是一段脚本，而不是用`module`关键字定义一个模块，但是模块与脚本还是有两点区别：

*   在 ES6 模块中，无论你是否加入“`use strict;`”语句，默认情况下模块都是在严格模式下运行。
*   在模块中你可以使`用 import`和`export`关键字。

我们先来讨论`export`。默认情况下，你在模块中的所有声明相对于模块而言都是寄存在本地的。如果你希望公开在模块中声明的内容，并让其它模块加以使用，你一定要*导出*这些功能。想要导出模块的功能有很多方法，其中最简单的方式是添加`export`关键字。

```
 // kittydar.js - 找到一幅图像中所有猫的位置
    // （事实上是 Heather Arthur 写的这个库）
    // （但是她没有使用 ES6 中新的模块特性，因为那时候是 2013 年）
    export function detectCats(canvas, options) {
      var kittydar = new Kittydar(options);
      return kittydar.detectCats(canvas);
    }
    export class Kittydar {
      ... 处理图片的几种方法 ...
    }
    // 这个 helper 函数没有被 export。
    function resizeCanvas() {
      ...
    }
    ...
```

你可以`导出`所有的最外层`函数`、`类`以及`var`、`let`或`const`声明的变量。

了解这些，你就可以编写一个简单的模块。你不需要将所有代码都放在一个[IIFE](https://en.wikipedia.org/wiki/Immediately-invoked_function_expression)或回调中，你只需要在模块中解放手脚，声明你需要的所有内容。代码就是模块，不是一段脚本，所以所有的声明都被限定在模块的作用域中，对所有脚本和模块全局*不可见*。你需要做的是将组成模块公共 API 的声明全部导出。

在模块中，除 export 之外的代码无异于普通代码，你可以访问类似`Object`和`Array`这样的全局对象。如果你在 web 浏览器中运行模块，你甚至可以使用`document`对象和`XMLHttpRequest`对象。

在一个独立文件中，我们可以导入`detectCats()`函数然后用它来做点儿什么：

```
 // demo.js - Kittydar 的 demo 程序
    import {detectCats} from "kittydar.js";
    function go() {
        var canvas = document.getElementById("catpix");
        var cats = detectCats(canvas);
        drawRectangles(canvas, cats);
    }
```

如果想从一个模块中导入多个名称，你可以这样写：

```
 import {detectCats, Kittydar} from "kittydar.js";
```

当你运行的模块中包含一条`import`声明时，首先会加载被导入的模块；然后依赖图的深度优先遍历按顺序执行每一个模块的主体代码；为了避免形成回环，所有已执行的模块都会被忽略。

这些就是模块的基本知识了，相当简单吧。;-)

## Export 列表

你不需要标记每一个被导出的特性，你只需要在花括号中按照列表的格式写下你想导出的所有名称：

```
 export {detectCats, Kittydar};
    // 此处不需要 `export`关键字
    function detectCats(canvas, options) { ... }
    class Kittydar { ... }
```

`export`列表可以在模块文件最外层作用域的每一处声明，不一定非要把它放在模块文件的首行。你也可以声明多个`export`列表，甚至通过其它的`export`声明打造一个混合的`export`列表，只要保证每一个被导出的名称是唯一的即可。

## 重命名 import 和 export

恰恰有时候，导出的名称会与你需要使用的其它名称产生冲突，ES6 为你提供了重命名的方法解决这个问题，当你在导入名称时可以这样做：

```
 // suburbia.js
    // 这两个模块都会导出以`flip`命名的东西。
    // 要同时导入两者，我们至少要将其中一个的名称改掉。
    import {flip as flipOmelet} from "eggs.js";
    import {flip as flipHouse} from "real-estate.js";
    ...
```

同样，当你在导出的时候也可以重命名。你可能会想用两个不同的名称导出相同的值，这样的情况偶尔也会遇到：

```
 // unlicensed_nuclear_accelerator.js - 无 DRM（数字版权管理）的媒体流
    // （这不是一个真实存在的库，但是或许它应该被做成一个库）

    function v1() { ... }
    function v2() { ... }

    export {
      v1 as streamV1,
      v2 as streamV2,
      v2 as streamLatestVersion
    };
```

## Default exports

现在广泛使用的模块系统有 CommonJS、AMD 两种，设计出来的新标准可以与这两种模块进行交互。所以假设你有一个 Node 项目，你已经执行了`npm install lodash`，你的 ES6 模块可以从 Lodash 中导入独立的函数：

```
 import {each, map} from "lodash";

    each([3, 2, 1], x => console.log(x));
```

但是也许你已经习惯看到`_.each`的书写方式而不想直接用`each`函数呢？或者你就真的想导入整个`_`函数呢，毕竟[_ 对于 Lodash 而言至关重要](https://lodash.com/docs#_)。

针对这种情况，你可以换用一种稍微不太一样的方法：不用花括号来导入模块。

```
 import _ from "lodash";
```

这种简略的表达方法等价于`import {default as _} from "lodash";`。在 ES6 的模块中导入的 CommonJS 模块和 AMD 模块都有一个`默认的`导出，如果你用`require()`加载这些模块也会得到相同的结果——`exports`对象。

ES6 模块不只导出 CommonJS 模块，它的设计逻辑为你提供导出不同内容的多种方法，默认导出的是你得到的所有内容。举个例子，在用这种写法的时候，据我所知，著名的[colors](https://github.com/Marak/colors.js)包就没有任何针对 ES6 的支持。像大多数 npm 上的包一样，它是诸多 CommonJS 模块的集合，但是你可以正确地将它导入到你的 ES6 代码中。

```
 // `var colors = require("colors/safe");`的 ES6 等效代码
    import colors from "colors/safe";
```

如果你想让自己的 ES6 模块有一个默认的导出，实现的方法很简单，默认导出与其它类型的导出相似，没有什么技巧可言，唯一的不同之处是它被命名为“`default`”。你可以用我们刚才讨论的重命名语法来实现：

```
 let myObject = {
      field1: value1,
      field2: value2
    };
    export {myObject as default};
```

这种简略的表达方法看起来更清爽：

```
 export default {
      field1: value1,
      field2: value2
    };
```

关键字`export default`后可跟随任何值：一个函数、一个类、一个对象字面量，只要你能想到的都可以。

## 模块对象

很抱歉新特性有点儿多，但 JavaScript 不是唯一这样做的语言：出于某些原因，每一种语言中的模块系统都有这么一堆又独立又小，虽然无聊但是很方便的特性。不过还好，我们只剩一样东西没讲了。好吧，是两样。

```
 import * as cows from "cows";
```

当你`import *`时，导入的其实是一个模块命名空间对象，模块将它的所有属性都导出了。所以如果“cows”模块导出一个名为`moon()`的函数，然后用上面这种方法“cows”将其全部导入后，你就可以这样调用函数了：`cows.moo()`。

## 聚合模块

有时一个程序包中主模块的代码比较多，为了简化这样的代码，可以用一种统一的方式将其它模块中的内容聚合在一起导出，可以通过这种简单的方式将所有所需内容导入再导出：

```
 // world-foods.js - 来自世界各地的好东西

    // 导入"sri-lanka"并将它导出的内容的一部分重新导出
    export {Tea, Cinnamon} from "sri-lanka";

    // 导入"equatorial-guinea"并将它导出的内容的一部分重新导出
    export {Coffee, Cocoa} from "equatorial-guinea";

    // 导入"singapore"并将它导出的内容全部导出
    export * from "singapore";
```

这些`export-from`语句每一个都好比是在一条`import-from`语句后伴随着一个`export`。与真正的导入内容的方法不同的是，这些导入内容再重新导出的方法不会在作用域中绑定你导入的内容。如果你打算用`world-foods.js`中的`Tea`来写一些代码，可别用这种方法导入模块，你会发现当前模块作用域中根本找不到`Tea`。

如果从“singapore”导出的任何名称碰巧与其它的导出冲突了，可能会触发一个错误，所以使用`export *`语句的时候要格外小心。

呼！终于讲完了所有的语法！现在来讲一些有趣的内容。

## import 实际都做了些什么？

如果我说它*什么都没做*，你敢信？

哦，看来你没那么容易上当啊。好吧，你相信标准里面通常都不会规定`import`的行为么？如果真是这样，那这是件好事儿么？

ES6 将模块加载过程的细节完全[交由最终的实现来定义](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-hostresolveimportedmodule)，模块执行的其它部分倒是[在规范中有详细定义](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-toplevelmoduleevaluationjob)。

粗略地讲，当你通知 JS 引擎运行一个模块时，它一定会按照以下四个步骤执行下去：

1.  语法解析：阅读模块源代码，检查语法错误。
2.  加载：递归地加载所有被导入的模块。这也正是没被标准化的部分。
3.  连接：每遇到一个新加载的模块，为其创建作用域并将模块内声明的所有绑定填充到该作用域中，其中包括由其它模块导入的内容。
4.  如果你的代码中有`import {cake} from "paleo"`这样的语句，而此时“paleo”模块并没有导出任何“`cake`”，你就会触发一个错误。这实在是太糟糕了，你都*快要*运行模块中的代码了，都是 cake 惹的祸！
5.  运行时：最终，在每一个新加载的模块体内执行所有语句。此时，导入的过程就已经结束了，所以当执行到达有一行`import`声明的代码的时候……什么都没发生！

看到了嘛？我可告诉过你结果是“啥都没有”哦。事关编程语言我绝不撒谎！

但是现在我们真的要深入了解这个系统最有趣的部分了！有一个很酷的小技巧我可以教给你。系统不指定加载过程的实现方式，你也可以通过在源代码中查找`import`声明提前计算出所有依赖，你可以将 ES6 系统实现为：在编译时计算所有依赖并将所有模块打包成一个文件，通过网络一次传输所有模块！像[webpack](http://www.2ality.com/2015/04/webpack-es6.html)这样的工具就实现了这个功能。

这种做法的意义非常深远，因为通过网络加载脚本需要花费时间，每当你请求到一个模块，你可能发现它里面也包含着`import`声 明，这就需要你再花费一些时间加载更多的脚本。基于如此天真的思想实现的加载器需要消耗更多的网络往返时间。但是 webpack 就不一样啦，它所用的加载 器是经过精心设计的，吸收了软件工程领域的精华，所以你不仅可以立即开始使用 ES6 模块系统，还不会损耗运行时的性能。

最初的时候，标准委员会已经制定并实现了详细的 ES6 模块加载标准，它未成为最终的标准的原因是成员们没有就代码封包（bundle）功能的实现方式达成一 致意见。我希望有人能搞定这个问题，正如我们所见，模块加载的过程亟待被标准化；最关键的是，封包的功能实在是太好，就这样放弃对其进行标准化有些可惜 啊。

## 静态 vs 动态：论规则及破例之法

JavaScript 作为一门动态语言已经得到了一个令人惊讶的静态模块系统。

*   你只可以在模块的最外层作用域使用`import`和`export`，不可在条件语句中使用，也不能在函数作用域中使用`import`。
*   所有导出的标识符一定要在源代码中明确地导出它们的名称，你不能通过编写代码遍历一个数组然后用数据驱动的方式导出一堆名称。
*   模块对象被冻结了，所以你无法 hack 模块对象并为其添加 polyfill 风格的新特性。
*   一个模块的*所有*依赖必须在模块代码运行前完全加载、解析并且及早连接，不存在一种通过`import`来按需懒加载的语法。
*   `import`模块产生的错误没有错误恢复机制。一个 app 可能囊括了上百个模块，一旦有一个模块无法加载或连接，所有的模块都不会运行，而且你不能在`try/catch`代码块中捕捉`import`的错误信息。（上面这些描述的本意是说：系统是静态的，webpack 可在编译时为你检测那些错误。）
*   不支持在模块加载依赖前运行其它代码的钩子，这也意味着无法控制模块的依赖加载过程。

只要你的需求是静态的，系统就会运行良好，但是你有时可以设想下需要一点儿 hack，对么？

这也就是无论你用什么模块加载系统，你都将有一个编程 API 来支持 ES6 的静态`import/export`语法。举个例子，[webpack 中引入了一个“代码分割”API](http://webpack.github.io/docs/code-splitting.html)，从而可以按需懒加载一些模块的多个封包。相同的 API 可以帮你打破上面列举的绝大多数其它的规则。

ES6 模块语法非常静态，这是很好的——它通过强有力的编译时工具的形式进行弥补。但是设计静态语法的初衷是要与丰富的动态编程加载器 API 一起增强 ES6 的模块系统。

## 我什么时候可以使用 ES6 模块？

如果你现在就想在项目中加入新的模块语法，你需要使用[Babel](http://babeljs.io/)或[Traceur](https://github.com/google/traceur-compiler#what-is-traceur)这样的转译器。在系列之前的文章中，[Gastón I. Silva 展示了如何使用 Babel 和 Broccoli](http://www.infoq.com/cn/articles/es6-in-depth-babel-and-broccoli)来为 web 平台编译 ES6 代码；在那篇文章的基础上，Gastón 准备了[一个支持 ES6 模块的工作示例](https://github.com/givanse/broccoli-babel-examples/tree/master/es6-modules)。[Axel Rauschmayer 写的这篇文章](http://www.2ality.com/2015/04/webpack-es6.html)给出了一个用 Babel 和 webpack 构建项目的示例。

ES6 模块系统主要由 Dave Herman 和 Sam Tobin-Hochstadt 进行设计，在近几年的争论中，他们与所有参与者（包括我）为新模块系统的静态部分进行辩护。Jon Coppeard 负责在 Firefox 中实现这些模块的特性。[JavaScript 加载器标准](https://github.com/whatwg/loader)也在制定当中，接下来标准委员会可能会为 HTML 添加一些类似`<script type=module>`特性。

然后这就是 ES6 的全部啦。

深入浅出 ES6 非常有趣，我可不想匆匆结束。可能我们应该再多写一期文章，讨论一下 ES6 规范犄角旮旯里的特性，这些东西不太重要，甚至连标准委员会自己都不愿意写文章来说说这些特性。我可能再讨论一下 JavaScript 未来的导向，下一次请记得回来围观深入浅出 ES6 的惊天大结局！

查看原文：[深入浅出 ES6（十六）：模块 Modules](http://www.infoq.com/cn/articles/es6-in-depth-modules)

