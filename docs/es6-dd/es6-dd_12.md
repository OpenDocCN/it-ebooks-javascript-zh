# 深入浅出 ES6（十二）：代理 Proxies

作者 Jason Orendorff ，译者 刘振涛

*编者按：ECMAScript 6 已经正式发布了，作为它最重要的方言，Javascript 也即将迎来语法上的重大变革，InfoQ 特开设“[深入浅出 ES6](http://www.infoq.com/cn/es6-in-depth/)”专栏，来看一下 ES6 将给我们带来哪些新内容。本专栏文章来自[Mozilla Web 开发者博客](https://hacks.mozilla.org/category/es6-in-depth/)，由作者授权翻译并发布。*

请看这样一段代码：

```js
 var obj = new Proxy({}, {
      get: function (target, key, receiver) {
        console.log(`getting ${key}!`);
        return Reflect.get(target, key, receiver);
      },
      set: function (target, key, value, receiver) {
        console.log(`setting ${key}!`);
        return Reflect.set(target, key, value, receiver);
      }
    });
```

代码乍一看有些复杂，使用了一些陌生的特性，稍后我会详细讲解每一部分。现在，一起来看一下我们创建的对象：

```js
 > obj.count = 1;
        setting count!
    > ++obj.count;
        getting count!
        setting count!
        2
```

显示结果可能与我们的理解不太一样，为什么会输出“`setting count`”和“`getting count`”？其实，我们拦截了这个对象的属性访问方法，然后将“.”运算符重载了。

## 它是如何做到的？

计算领域最好的的技巧是虚拟化，这种技术一般用来实现惊人的功能。它的工作机制如下：

1.  随便选一张照片。

    ![](img/1power-plant-500x266.jpg)

    [图片来源：Martin Nikolaj Bech](https://www.flickr.com/photos/martini_dk/369891979)

2.  在图片中围绕某物勾勒出一个轮廓。

    ![](img/power-plant-with-outline-500x266.png)

3.  现在替换掉轮廓中的内容，或者替换掉轮廓外的内容，但是始终要遵循向后兼容的规则，替换前后的图片要尽可能相似，不能让轮廓两侧的图像过于突兀。

    ![](img/wind-farm-500x266.png)

    [图片来源：Beverley Goodwin](https://www.flickr.com/photos/bevgoodwin/8671334130/)

你可能在《*[楚门的世界](http://movie.douban.com/subject/1292064/)*》和《*[黑客帝国](http://movie.douban.com/subject/1291843/)*》这类经典的计算机科学电影中见到过类似的 hack 方法，将世界划分为两个部分，主人公生活在内部世界，外部世界被精心编造的常态幻觉所替换。

为了满足向后兼容的规则，你需要巧妙地设计填补进去的图片，但是真正的技巧是正确地勾勒轮廓。

我所谓的*轮廓*是指一个 API 边界或接口，接口可以详细说明两段代码的交互方式以及交互双方对另一半的需求。所以如果一旦在系统中设计好了接口，轮廓自然就清晰了，这样就可以任意替换接口两侧的内容而不影响二者的交互过程。

如果没有现成的接口，就需要施展你的创意才华来创造新接口，有史以来最酷的软件 hack 总是会勾勒一些之前从未有过的 API 边界，然后通过大量的工程化实践将接口引入到现有的体系中去。

[虚拟内存](https://zh.wikipedia.org/wiki/%E8%99%9A%E6%8B%9F%E5%86%85%E5%AD%98)、[硬件虚拟化](https://zh.wikipedia.org/wiki/%E7%A1%AC%E4%BB%B6%E8%99%9A%E6%8B%9F%E5%8C%96)、[Docker](https://zh.wikipedia.org/wiki/Docker_%28%E8%BB%9F%E9%AB%94%29)、[Valgrind](http://valgrind.org/)、[rr](http://rr-project.org/)等不同抽象程度的项目都会基于现有的系统推动开发一些令人意想不到的新接口。在某些情况下，需要花费数年的时间、新的操作系统特性甚至是新的硬件来使新的边界良好运转。

最棒的虚拟化 hack 会带来对需要虚拟的东西的新的理解。想要编写一个 API，你需要充分理解你所面向的对象，一旦你理解透彻，就能实现出令人惊异的成果。

而 ES6 则为 JavaScript 中最基本的概念“对象（object）”引入了虚拟化支持。

## 所以，对象到底是什么？

噢，我是说真的，请花费一点时间仔细想想这个问题的答案。当你清楚自己知道对象是什么的的时候再向下滚动。

![](img/thinker-500x274.jpg)

[图片来源：Joe deSousa](https://www.flickr.com/photos/mustangjoe/5966894496/)

这个问题于我而言太难了！我从未听到过一个非常满意的定义。

这会让你感到惊讶么？定义基础概念向来很困难——抽空看看欧几里得在《[几何原本](http://aleph0.clarku.edu/~djoyce/java/elements/bookI/bookI.html)》中的前几个定义你就知道了。ECMAScript 语言规范很棒，可是却将对象定义为“type 对象的成员”，这种定义真的对我们没什么帮助。

后来，规范中又添加了一个定义：“对象是属性的集合”。这句话没错，目前来说可以这样定义，我们稍后继续讨论。

我之前说过，*想要编写一个 API，你需要充分理解你所面向的对象*。所以在某种程度上，我也算对本文做出一个承诺，我们会一起深入理解对象的细节，然后一起实现酷炫的功能。

那么我们就跟随 ECMAScript 标准委员会的脚步，为 JavaScript 对象定义一个 API，一个接口。问题是我们需要什么方法？对象又可以做什么呢？

这个问题的答案一定程度上取决于对象的类型：DOM 元素对象可以做一部分事情，音频节点对象又可以做另外一部分事情，但是所有对象都会共享一些基础功能：

*   对象都有属性。你可以 get、set 或删除它们或做更多操作。
*   对象都有原型。这也是 JS 中继承特性的实现方式。
*   有一些对象是可以被调用的函数或构造函数。

几乎所有处理对象的 JS 程序都是使用属性、原型和函数来完成的。甚至元素或声音节点对象的特殊行为也是通过调用继承自函数属性的方法来进行访问。

所以 ECMAScript 标准委员会定义了一个由 14 种*内部方法*组成的集合，亦即一个适用于所有对象的通用接口，属性、原型和函数这三种基础功能自然成为它们关注的核心。

我们可以在[ES6 标准列表 5 和 6](http://www.ecma-international.org/ecma-262/6.0/index.html#table-5)中找到全部的 14 种方法，我只会在这里讲解其中一部分。双方括号[[ ]]代表*内部*方法，在一般的 JS 代码中不可见，你可以调用、删除或覆写普通方法，但是无法操作内部方法。

*   **obj.[[Get]](key, receiver)** – 获取属性值。

    当 JS 代码执行以下方法时被调用：`obj.prop`或`obj[key]`。

    *obj*是当前被搜索的对象，*receiver*是我们首先开始搜索这个属性的对象。有时我们必须要搜索几个对象，*obj*可能是一个在*receiver*原型链上的对象。

*   **obj.[[Set]](key, value, receiver)** – 为对象的属性赋值。

    当 JS 代码执行以下方法时被调用：`obj.prop = value`或`obj[key] = value`。

    执行类似`obj.prop += 2`这样的赋值语句时，首先调用[[Get]]方法，然后调用[[Set]]方法。对于++和--操作符来说亦是如此。

*   **obj.[HasProperty]** – 检测对象中是否存在某属性。

    当 JS 代码执行以下方法时被调用：`key in obj`。

*   **obj.[Enumerate]** – 列举对象的可枚举属性。

    当 JS 代码执行以下方法时被调用：`for (key in obj)` …

    这个内部方法会返回一个可迭代对象，`for-in`循环可通过这个方法得到对象属性的名称。

*   **obj.[GetPrototypeOf]** – 返回对象的原型。

    当 JS 代码执行以下方法时被调用：[`obj.[__proto__]`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/proto)或[`Object.getPrototypeOf`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/GetPrototypeOf)`(obj)`。

*   **functionObj.[[Call]](thisValue, arguments)** – 调用一个函数。

    当 JS 代码执行以下方法时被调用：`functionObj()`或`x.method()`。

    可选的。不是每一个对象都是函数。

*   **constructorObj.[[Construct]](arguments, newTarget)** – 调用一个构造函数。

    当 JS 代码执行以下方法时被调用：举个例子，`new Date(2890, 6, 2)`。

    可选的。不是每一个对象都是构造函数。

    参数*newTarget*在子类中起一定作用，我们将在未来的文章中详细讲解。

可能你也可以猜到其它七个内部方法。

在整个 ES6 标准中，只要有可能，任何语法或对象相关的内建函数都是基于这 14 种内部方法构建的。ES6 在对象的中枢系统周围划分了一个清晰的界限，你可以借助代理特性用任意 JS 代码替换标准中枢系统的内部方法。

既然我们马上要开始讨论覆写内部方法的相关问题，请记住，我们要讨论的是诸如`obj.prop`的核心语法、诸如[`Object.keys()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)的内建函数等的行为。

## 代理 Proxy

ES6 规范定义了一个全新的全局构造函数：[代理](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)（[Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)）。它可以接受两个参数：*目标对象（target）*与*句柄对象（handler）*。请看一个简单的示例：

```js
 var target = {}, handler = {};
    var proxy = new Proxy(target, handler);
```

我们先来探讨*代理*和*目标对象*之间的关系，然后再研究句柄对象的功用。

*代理*的行为很简单：将*代理*的所有内部方法转发至*目标*。简单来说，如果调用`proxy.[[Enumerate]]()`，就会返回`target.[[Enumerate]]()`。

现在，让我们尝试执行一条能够触发调用`proxy.[[Set]]()`方法的语句。

```js
 proxy.color = "pink";
```

好的，刚刚都发生了什么？`proxy.[[Set]]()`应该调用`target.[[Set]]()`方法，然后在*目标*上创建一个新的属性。实际的结果如何？

```js
 > target.color
        "pink"
```

是的，它做到了！对于所有其它内部方法而言同样可以做到。新创建的代理会尽可能与目标的行为一致。

当然，它们也不完全相同，你会发现`proxy !== target`。有时也有目标能够通过类型检测而代理无法通过的情况发生，举个例子，如果代理的目标是一个 DOM 元素，相应的代理就不是，此时类似`document.body.appendChild(proxy)`的操作会触发类型错误（`TypeError`）。

## 代理句柄

现在我们继续来讨论一个让代理充满魔力的功能：句柄对象。

句柄对象的方法可以覆写任意代理的内部方法。

举个例子，你可以定义一个[`handler.set()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/set)方法来拦截所有给对象属性赋值的行为：

```js
 var target = {};
    var handler = {
      set: function (target, key, value, receiver) {
        throw new Error("请不要为这个对象设置属性。");
      }
    };
    var proxy = new Proxy(target, handler);
    > proxy.name = "angelina";
        Error: 请不要为这个对象设置属性。
```

句柄方法的完整列表可以在[MDN 有关代理的页面](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy#Methods_of_the_handler_object)上找到，一共有 14 种方法，与 ES6 中定义的 14 中内部方法一致。

所有句柄方法都是可选的，没被句柄拦截的内部方法会直接指向目标，与我们之前看到的别无二致。

## 小试牛刀（一）：“不可能实现的”自动填充对象

到目前为止，我们对于代理的了解程度足够尝试去做一些奇怪的事情，实现一些不借助代理根本无法实现的功能。

我们的第一个实践，创建一个`Tree()`函数来实现以下特性：

```js
 > var tree = Tree();
    > tree
        { }
    > tree.branch1.branch2.twig = "green";
    > tree
        { branch1: { branch2: { twig: "green" } } }
    > tree.branch1.branch3.twig = "yellow";
        { branch1: { branch2: { twig: "green" },
                     branch3: { twig: "yellow" }}}
```

请注意，当我们需要时，所有中间对象*branch1*、*branch2*和*branch3*都可以自动创建。这固然很方便，但是如何实现呢？

在这之前，没有可以实现这种特性的方法，但是通过代理，我们只用寥寥几行就可以轻松实现，然后只需要接入`tree.[[Get]]()`就可以。如果你喜欢挑战，在继续阅读前可以尝试自己实现。

这里是我的解决方案：

```js
 function Tree() {
      return new Proxy({}, handler);
    }
    var handler = {
      get: function (target, key, receiver) {
        if (!(key in target)) {
          target[key] = Tree();  // 自动创建一个子树
        }
        return Reflect.get(target, key, receiver);
      }
    };
```

注意最后的`Reflect.get()`调用，在代理句柄方法中有一个极其常见的需求：只执行委托给*目标*的默认行为。所以 ES6 定义了一个新的[`反射（Reflect）对象`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect)，在其上有 14 种方法，你可以用它来实现这一需求。

## 小试牛刀（二）：只读视图

我想我可能传达给你们一个错误的印象，也就是代理易于使用。接下来的这个示例可能会让你稍感困顿。

这一次我们的赋值语句更复杂：我们需要实现一个函数，`readOnlyView(object)`，它可以接受任何对象作为参数，并返回一个与此对象行为一致的代理，该代理不可被变更，就像这样：

```js
 > var newMath = readOnlyView(Math);
    > newMath.min(54, 40);
        40
    > newMath.max = Math.min;
        Error: can't modify read-only view
    > delete newMath.sin;
        Error: can't modify read-only view
```

我们如何实现这样的功能？

即使我们不会阻断内部方法的行为，但仍然要对其进行干预，所以第一步是拦截可能修改目标对象的五种内部方法。

```js
 function NOPE() {
      throw new Error("can't modify read-only view");
    }
    var handler = {
      // 覆写所有五种可变方法。
      set: NOPE,
      defineProperty: NOPE,
      deleteProperty: NOPE,
      preventExtensions: NOPE,
      setPrototypeOf: NOPE
    };
    function readOnlyView(target) {
      return new Proxy(target, handler);
    }
```

这段代码可以正常运行，它借助只读视图阻止了赋值、属性定义等过程。

这种方案中是否有漏洞？

最大的问题是类似[[Get]]的一些方法可能仍然返回可变对象，所以即使一些对象`x`是只读视图，`x.prop`可能是可变的！这是一个巨大的漏洞。

我们需要添加一个`handler.get()`方法来堵上漏洞：

```js
 var handler = {
      ...
      // 在只读视图中包裹其它结果。
      get: function (target, key, receiver) {
        // 从执行默认行为开始。
        var result = Reflect.get(target, key, receiver);
        // 确保返回一个不可变对象！
        if (Object(result) === result) {
          // result 是一个对象。
          return readOnlyView(result);
        }
        // result 是一个原始原始类型，所以已经具备不可变的性质。
        return result;
      },
      ...
    };
```

这仍然不够，`getPrototypeOf`和`getOwnPropertyDescriptor`这两个方法也需要进行同样的处理。

然而还有更多问题，当通过这种代理调用 getter 或方法时，传递给 getter 或方法的`this`的值通常是代理自身。但是正如我们之前所见，有时代理无法通过访问器和方法执行的类型检查。在这里用目标对象代替代理更好一些。聪明的小伙伴，你知道如何解决这个问题么？

由此可见，创建代理非常简单，但是创建一个具有直观行为的代理相当困难。

## 只言片语

*   代理到底好在哪里？

    代理可以帮助你观察或记录对象访问，当调试代码时助你一臂之力，测试框架也可以用代理来创建[模拟对象](https://zh.wikipedia.org/wiki/%E6%A8%A1%E6%8B%9F%E5%AF%B9%E8%B1%A1)（[mock object](https://en.wikipedia.org/wiki/Mock_object)）。

    代理可以帮助你强化普通对象的能力，例如：惰性属性填充。

    我不太想提到这一点，但是如果要想了解代理在代码中的运行方式，将代理的句柄对象包裹在*另一个代理*中是一个非常不错的办法，每当句柄方法被访问时就可以将你想要的信息输出到控制台中。

    正如上文中只读视图的示例`readOnlyView`，我们可以用代理来限制对象的访问。当然在应用代码中很少遇到这种用例，但是 Firefox 在内部使用代理来实现不同域名之间的[安全边界](https://developer.mozilla.org/en-US/docs/Mozilla/Gecko/Script_security)，是我们的安全模型的关键组成部分。

*   与 WeakMap 深度结合。在我们的`readOnlyView`示例中，每当对象被访问的时候创建一个新的代理。这种做法可以帮助我们节省在`WeakMap`中创建代理时的缓存内存，所以无论传递多少次对象给`readOnlyView`，只会创建一个代理。

    这也是一个动人的 WeakMap 用例。

*   代理可解除。ES6 规范中还定义了另外一个函数：`Proxy.revocable(target, handler)`。这个函数可以像`new Proxy(target, handler)`一样创建代理，但是创建好的代理后续可被*解除*。（`Proxy.revocable`方法返回一个对象，该对象有一个`.proxy`属性和一个`.revoke`方法。）一旦代理被解除，它即刻停止运行并抛出所有内部方法。

*   对象不变性。在某些情况下，ES6 需要代理的句柄方法来报告与*目标*对象状态一致的结果，以此来保证所有对象甚至是代理的不变性。举个例子，除非目标不可扩展（inextensible），否则代理不能被声明为不可扩展的。

不变性的规则非常复杂，在此不展开详述，但是如果你看到类似“`proxy can't report a non-existent property as non-configurable`”这样的错误信息，就可以考虑从不变性的角度解决问题，最可能的补救方法是改变代理报告本身，或者在运行时改变目标对象来反射代理的报告指向。

## 现在，你认为对象是什么？

我记得我们之前的见解是：“对象是属性的集合。”

我不喜欢这个定义，即使给定义叠加原型和可调用能力也不会让我改变看法。我认为“集合（collection）”这个词太危险了，不适合用作对象的定义。对象的句柄方法可以做任何事情，它们也可以返回随机结果。

ECMAScript 标准委员会针对这个问题开展了许多研究，搞清楚了对象能做的事情，将那些方法进行标准化，并将虚拟化技术作为每个人都能使用的一等特性添加到语言的新标准中，为前端开发领域拓展了无限可能。

完善后的对象几乎可以表示任何事物。

对象是什么？可能现在最贴切的答案需要用 12 个内部方法进行定义：对象是在 JS 程序中拥有[[Get]]、[[Set]]等操作的实体。

* * *

我不太确定我们是否比之前更了解对象，但是我们绝对做了许多惊艳的事情，是的，我们实现了旧版 JS 根本做不到的功能。

## 我现在可以使用代理么？

不！在 Web 平台上无论如何都不行。目前只有 Firefox 和微软的 Edge 支持代理，而且还没有支持这一特性 polyfill。

如果你想在 Node.js 或 io.js 环境中使用代理，首先你需要添加名为[harmony-reflect](https://github.com/tvcutsem/harmony-reflect)的 polyfill，然后在执行时启用一个非默认的选项（`--harmony_proxies`），这样就可以暂时使用 V8 中实现的老版本代理规范。

放轻松，让我们一起来做试验吧！为每一个对象创建成千上万个相似的副本镜像却不能调试？现在就解放自己！不过目前来看，请不要将欠考虑的有关代理的代码泄露到产品中，这非常危险。

代理特性在 2010 年由 Andreas Gal 首先实现，由 Blake Kaplan 进行代码审查。标准委员会后来完全重新设计了这个特性。Eddy Bruel 在 2012 年实现了新标准。

我实现了`反射（Reflect）`特性，由 Jeff Walden 进行代码审查。Firefox Nightly 已经支持除`Reflect.enumerate()`外的所有特性。

下一次，我们将讨论 ES6 中最有争议的特性，有请在 Firefox 中实现这一特性的 Mozilla 工程师 Eric Faust 为大家深入讲解 ES6 中的类特性。

查看原文：[深入浅出 ES6（十二）：代理 Proxies](http://www.infoq.com/cn/articles/es6-in-depth-proxies-and-reflect)

