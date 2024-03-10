# 深入浅出 ES6（十五）：子类 Subclassing

作者 Eric Faust ，译者 刘振涛

*译者按：ECMAScript 6 已经正式发布了，作为它最重要的方言，Javascript 也即将迎来语法上的重大变革，InfoQ 特开设“[深入浅出 ES6](http://www.infoq.com/cn/es6-in-depth/)”专栏，来看一下 ES6 将给我们带来哪些新内容。本专栏文章来自[Mozilla Web 开发者博客](https://hacks.mozilla.org/category/es6-in-depth/)，由作者授权翻译并发布。*

在之前的文章《[深入浅出 ES6（十三）：类 Class](http://www.infoq.com/cn/articles/es6-in-depth-classes)》中，我们一起深入探讨了 ES6 的新特性——类，在这篇文章中我写到“可以使用类来创建一些简易的对象构造函数”，于是我们共同实现了这样一段代码：

```js
 class Circle {
        constructor(radius) {
            this.radius = radius;
            Circle.circlesMade++;
        };
        static draw(circle, canvas) {
            // Canvas 绘制代码
        };
        static get circlesMade() {
            return !this._count ? 0 : this._count;
        };
        static set circlesMade(val) {
            this._count = val;
        };
        area() {
            return Math.pow(this.radius, 2) * Math.PI;
        };
        get radius() {
            return this._radius;
        };
        set radius(radius) {
            if (!Number.isInteger(radius))
                throw new Error("圆的半径必须是整数。");
            this._radius = radius;
        };
    }
```

你们或许还记得，我在文章末尾预留了一个尚未揭晓的悬念，其实在类的整个体系中还有一个威力无穷的特性等待我们深入挖掘。ES6 的类系统借鉴了传统的 C++和 Java 中的类系统，它们都支持类*继承*，子类可以继承父类，并在其基础上扩展更多自己的功能。那就让我们继续今天的文章，进一步发掘这个新特性无穷的潜力吧。

在我们开始讨论子类的概念之前，还是再一起巩固一下属性继承和*动态原型链*的相关知识。

## JavaScript 继承

我们在创建对象的时候可以为其添加各种属性，但在这个过程中，新创建的对象同时也继承了原型对象的属性。作为 JavaScript 开发者，你应该很熟悉`Object.create`这个 API，我们可以用它来创建一些新对象：

```js
 var proto = {
        value: 4,
        method() { return 14; }
    }
    var obj = Object.create(proto);
    obj.value; // 4
    obj.method(); // 14
```

进一步说，如果我们在创建`obj`时给它添加`proto`已有的属性，则新创建对象会覆盖原型对象中的同名属性。

```js
 obj.value = 5;
    obj.value; // 5
    proto.value; // 4
```

## 子类化的基本概念

现在我们已经了解，通过类创建的对象，它们的原型链之间是如何连接的。回想一下，当我们创建一个类的时候，类的定义中有一个`constructor`方法，这个方法控制着所有静态方法，实际上我们正是依据这个方法创建了一个新的函数；然后我们又创建一个对象并将它赋给这个函数的`prototype`属性。为了使新创建的类继承所有的静态属性，我们需要让这个新的函数对象继承超类的函数对象；同样，为了使新创建的类继承所有实例方法，我们需要让新函数的`prototype`对象继承超类的`prototype`对象。

看官您若要是感觉这长篇大论看得头疼，不妨来看一段简单的示例，看我们如何通过旧语法的力量将这一切连结起来，然后再进一步美化这段代码的结构。

继续我们之前的示例，假设我们有一个类`Shape`，并且想要基于这个类生成一个子类：

```js
 class Shape {
        get color() {
            return this._color;
        }
        set color(c) {
            this._color = parseColorAsRGB(c);
            this.markChanged();  // 稍后重绘 Canvas
        }
    }
```

在尝试编写这些代码时，我们仍会面临以前遇到过的`static`属性的问题：我们不能在定义函数的同时改变它的原型，到目前为止我们所知的语法不能实现这种功能。只不过你可以通过`Object.setPrototypeOf`方法绕过这一限制，但随即带来的问题是，这个方法性能很差，而 JS 引擎又很难对其进行优化，所以我们期待一种更完美的方法，能够在创建函数的同时处理好原型。

```js
 class Circle {
        // 与上文中代码相同
    }
    // 连结实例属性
    Object.setPrototypeOf(Circle.prototype, Shape.prototype);
    // 连结静态属性
    Object.setPrototypeOf(Circle, Shape);
```

这 段代码丑爆了！我们为 JavaScript 添加类语法的初衷是：能够用一段完整的代码封装目标对象的所有逻辑，而不是在类声明结束后再进行一些额外的“连 结”操作。Java、Ruby 和其它面向对象的语言中有一种特定的语法可以声明“一个类是另一个的子类”，JavaScript 中当然也要有这样的方法！ 我们可以用关键词`extends`声明子类关系，看好了，代码可以这样去写：

```js
 class Circle extends Shape {
        // 与上文中代码相同
    }
```

`extends`后面可以接驳任意合法且拥有`prototype`属性的构造函数。它可以是：

*   另一个类
*   源自现有继承框架（译者注：作者指的是原型继承，即使在 JavaScript 中类继承的本质也是原型继承）的近类函数
*   一个普通的函数
*   一个包含一个函数或类的变量
*   一个对象上的属性访问
*   一个函数调用

如果不希望创建出来的实例继承自`Object.prototype`，你甚至可以在`extends`后使用`null`来进行声明。

## Super 属性

现在我们学会怎样创建子类了，子类可以继承父类的属性，有时我们会在子类中重新定义同名方法，这样会覆盖掉我们继承的方法。但在某些情况下，如果你重新定义了一个方法，但有时你又想绕开这个方法去使用父类中的同名方法，应该怎样做呢？

假设我们想基于`Circle`类生成一个子类，这个子类可以通过一些因子来控制圆的缩放，为了实现这一目标，我们写下的这个类看起来有些不太自然：

```js
 class ScalableCircle extends Circle {
        get radius() {
            return this.scalingFactor * super.radius;
        }
        set radius() {
            throw new Error("可伸缩圆的半径 radius 是常量。" +
                            "请设置伸缩因子 scalingFactor 的值。");
        }
        // 处理 scalingFactor 的代码
    }
```

请注意`radius`的 getter 使用的是`super.radius`。这里的`super`是一个全新的关键字，它可以帮我们绕开我们在子类中定义的属性，直接从子类的原型开始查找属性，从而绕过我们覆盖到父类上的同名方法。

通过方法定义语法定义的函数，其原始对象方法的定义在初始化后就已完成，从而我们可以访问它的 super 属性（也可以访问`super[expr]`），由于该访问依赖的是原始对象，所以即使我们将方法存到本地变量，再访问时也不会改变`super`的行为。

```js
 var obj = {
        toString() {
            return "MyObject: " + super.toString();
        }
    }
    obj.toString(); // MyObject: [object Object]
    var a = obj.toString;
    a(); // MyObject: [object Object]
```

## 子类化内建方法

你想做的另一件事可能是扩展 JavaScript 的内建方法。现在你拥有极为强大的 JS 内建数据结构，它是子类化设计的基础之一，可被用来创建新的类型。假设你想编写一个带版本号的数组，你可以改变数组内容然后提交，或者将数组回滚到之前的状态。我们可以通过子类化`Array`来快速实现这一功能。

```js
 class VersionedArray extends Array {
        constructor() {
            super();
            this.history = [[]];
        }
        commit() {
            // 将变更保存到 history。
            this.history.push(this.slice());
        }
        revert() {
            this.splice(0, this.length, this.history[this.history.length - 1]);
        }
    }
```

`VersionedArray`的实例保留了一些重要的属性，包括`map`、`filter`还有`sort`，它们是组成数组的核心方法；当然，`Array.isArray()`也会把`VersionedArray`视为数组；当你向`VersionedArray`中添加元素时，它的`length`属性也会自动增长；说远一些，之前能够返回一个新数组的函数（例如`Array.prototype.slice()`）现在会返回一个`VersionedArray`！

## 派生类构造函数

你可能已经注意到在上个示例中`constructor`方法中我们调用了`super()`方法。调用过程都做了哪些事情呢？

在传统的类模型中，构造函数的作用是为类的实例初始化所有内部状态。每个子类中的构造函数都负责初始化各自的状态。我们希望将这些调用传递下去，以使所有子类都与父类共享相同的初始化代码。

我们可以通过`super`关键词调用父类中的构造函数，此时它本身也好像是函数一般。请注意，这个语法只在使用`extends`扩展的子类的`constructor`方法中有效，通过关键词`super`可以重写我们的`Shape`类。

```js
 class Shape {
        constructor(color) {
            this._color = color;
        }
    }
    class Circle extends Shape {
        constructor(color, radius) {
            super(color);
            this.radius = radius;
        }
        // 与上文中代码相同
    }
```

在 JavaScript 中，我们倾向于在构造函数中操作`this`对象，装载属性并且初始化内部状态。通常情况下，当我们通过`new`调用构造函数时`this`对象即被创建，就好像用`Object.create()`通过构造函数的`prototype`属性构造对象时所做的那样。然而，有些内建方法的内部对象布局不太一样。例如数组在内存中的布局方式就与普通对象不同。我们想要子类化内建方法，所以我们让最基础的构造函数分配`this`对象。如果它是一个内建方法，我们会得到我们想要的对象布局；如果它是一个普通构造函数，我们会得到期待中的默认的`this`对象。

你可能对`this`在子类构造函数中的绑定方式感到不解。当我们执行基类的构造函数前，`this`对象没有被分配，从而我们**无法得到一个确定的`this`值**。因此，在子类的构造函数中，调用`super`构造函数之前访问`this`会触发一个引用错误（`ReferenceError`）。

正如我们在上一篇文章中看到的，派生类构造函数与`constructor`方法的省略规则相同，看起来好像你这样写就像这样：

```js
 constructor(...args) {
        super(...args);
    }
```

构造函数有时不与`this`对象交互，它们通过其它方式创建对象并初始化后直接返回，在这种情况下，就不需要再使用`super`了。此时即使不调用`super`构造函数，任何构造函数也都能直接返回一个对象。

## new.target

还有一个奇怪的副作用，如果让最基础的类分配`this`对象，它有时并不知道应该分配哪一种对象。假设你正在写一个对象框架库，你想要一个基类`Collection`，有一些子类是数组，有一些子类是 map。此时，当你准备执行`Collection`的构造函数时，你将无法区分应该生成哪个类型的对象。

既然我们已经能够子类化内建方法，当我们执行内建构造函数时，在内部我们其实需要知道原始类的`prototype`属性指向何方。如果没有这个属性，我们无法创建有适当的实例方法的对象。为了解决这个问题，我们加入了一种语法来向 JavaScript 代码暴露这些信息。我们新添加的元属性`new.target`与构造函数有关，因为直接通过`new`来调用，所以它是一个被调函数，在这个函数中调用`super`会传递`new.target`的值。

我知道这很难理解，所以我将通过一段代码详细讲解：

```js
 class foo {
        constructor() {
            return new.target;
        }
    }
    class bar extends foo {
        // 为了清晰起见我们显式地调用 super。
        // 事实上不必去获取这些结果。
        constructor() {
            super();
        }
    }
    // 直接调用 foo，所以 new.target 是 foo
    new foo(); // foo
    // 1) 直接调用 bar，所以 new.target 就是 bar
    // 2) bar 通过 super()调用 foo，所以 new.target 仍然是 bar
    new bar(); // bar
```

现在，我们已经解决了上述的`Collection`问题，`Collection`构造函数可以通过检查`new.target`来确定子类的类型以及使用哪个内建方法这些不确定的因素。

`new.target`在任何函数中都是合法的，如果函数不是通过`new`调用，`new.target`将被赋值为`undefined`。

## 鱼和熊掌可以得兼

希望你能坚持读完整篇文章，充实大脑中的知识，我将感激不尽。现在让我们花一点时间来探讨一下，上面这些方法能很好地解决问题么？许多人已经针对“在语言中加入继承的特性是好是坏”这个问题各抒己见，你可能认为对于创建对象来说，继承永远比不上构造（[Composition](https://en.wikipedia.org/wiki/Composition_over_inheritance)） 的方法，换句话说，在与那么多古老的原型模型进行比较后你会发现，代码变得更加简洁需要付出的代价是设计灵活性的缺失。不可否认，在面向对象的世界里，如 果你想创建一些对象，并且它们以扩展的方式共享代码，最好的选择是混入类（Mixins），原因很简单：它能为你提供一种将多个对象的不同方法混入到同一 个对象中的方法，这个方法很简单，因此你无须理解两个不相关的代码片段是如何在相同的继承结构中相互适应的。

很 多人在这个话题上坚决捍卫自己的信念，但是我认为有些事情不值得一昧地坚持。首先，向语言中添加类特性并不意味着要强制使用这些特性，这一态度非常重要； 第二，同样重要的是，向语言中添加类特性也不意味着这就是解决继承问题的最好方式！事实上，有一些问题更适合通过原型继承的方式来建模。在考虑过所有情况 之后，类它只是你可以使用的一个额外的工具而已，它既不是唯一的工具也不一定是最好的工具。

如 果你想继续使用混入类，你可能希望你能有这样一种类，它继承自几个不同的主体，所以你可以继承每一个混入（Mixin）然后获取它们的精华。不幸的是，如 果改变现有的继承模型可能会使整个系统非常混乱，所以 JavaScript 没有实现类的多继承。那也就是说，在一个基于类的框架内部有一种混合解决方案可 以支持混入类特性。请看下面的这段代码，这是一个基于我们熟知的`extend`混入习语打造的函数：

```js
 function mix(...mixins) {
        class Mix {}
        // 以编程方式给 Mix 类添加
        // mixins 的所有方法和访问器
        for (let mixin of mixins) {
            copyProperties(Mix, mixin);
            copyProperties(Mix.prototype, mixin.prototype);
        }
        return Mix;
    }
    function copyProperties(target, source) {
        for (let key of Reflect.ownKeys(source)) {
            if (key !== "constructor" && key !== "prototype" && key !== "name") {
                let desc = Object.getOwnPropertyDescriptor(source, key);
                Object.defineProperty(target, key, desc);
            }
        }
    }
```

现在，我们无须在各种各样的混入之间创建显示的继承关系，我们只须使用`mix`函数就可以创建一个组合而成的超类。设想一下，如果你想编写一个协作编辑工具，在这个工具中的所有编辑动作都被记录下来，然后将内容序列化。你可以使用`mix`函数写一个`DistributedEdit`类：

```js
 class DistributedEdit extends mix(Loggable, Serializable) {
        // 事件方法
    }
```

这真是两全其美啊。如果你想将构造本身有超类的混入类，也可以用这个模型来解决：我们将超类传递给`mix`函数后它就会返回扩展后的类。

## 目前的可用性

好的，我们已经讨论了许多有关子类化内建方法还有所有的这些新玩意儿，但是你现在可以用这些新技术了么？

事实上，你只能使用其中一部分。在主流浏览器厂商中，Chrome 已经移植了我们今天所讨论的大多数特性，在严格模式下，你应该可以实现我们讨论的几乎所有的功能，当然，这不包括子类化`Array`。其它内建类型也会正常运转，但是数组会有一些不一样；我正在实现 Firefox 中的相关功能，希望能在不就之后达到与 Chrome 一样的目标（除了数组之外的每一个功能）。你可以访问[Bug 1141863](https://bugzilla.mozilla.org/show_bug.cgi?id=1141863)获取更多信息，但可能要在数周后才会登陆 Nightly 版本的 Firefox（译者注：截至翻译本文，该功能[已经实现](https://bugzilla.mozilla.org/show_activity.cgi?id=1141863)，将发布于 Firefox 44）。

此外，Edge 已经支持了`super`功能，但是没支持子类化内建方法；Safari 不支持所有的这些功能。

转译器相对来说处于劣势，它们可以创建类，支持`super`特性，但是子类化内建方法必须得到引擎的支持，它们需要通过引擎才能从内建方法中得到基类的实例（想想`Array.prototype.splice`）。

唷！真是一篇长文章啊。下一次[Jason Orendorff](http://www.infoq.com/cn/author/Jason-Orendorff)将回来与大家共同深入浅出 ES6 的模块系统。

查看原文：[深入浅出 ES6（十五）：子类 Subclassing](http://www.infoq.com/cn/articles/es6-in-depth-subclassing)

