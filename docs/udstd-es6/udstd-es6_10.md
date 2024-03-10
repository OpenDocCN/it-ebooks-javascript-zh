# 第九章 JS 的类

## 第九章 JS 的类

与大多数正规的面向对象编程语言不同， JS 从创建之初就不支持类，也没有把类继承作为定义相似对象以及关联对象的主要方式，这让不少开发者感到困惑。而从 ES1 诞生之前直到 ES5 时期，很多库都创建了一些工具，让 JS 从表面看来仿佛能支持类。尽管一些 JS 开发者强烈认为这门语言不需要类，但为处理类而创建的代码库如此之多，导致 ES6 最终引入了类。

在探索 ES6 的类的过程中，理解类的潜在机制会很有帮助，因此本章将会首先讨论 ES5 的开发者如何实现对类行为的模仿。然而正如你将在后面看到的， ES6 的类并不与其他语言的类完全相同，所具备的独特性正配合了 JS 的动态本质。

*   ES5 中的仿类结构
*   类的声明
    *   基本的类声明
    *   为何要使用类的语法
*   类表达式
    *   基本的类表达式
    *   具名类表达式
*   作为一级公民的类
*   访问器属性
*   需计算的成员名
*   生成器方法
*   静态成员
*   使用派生类进行继承
    *   屏蔽类方法
    *   继承静态成员
    *   从表达式中派生类
    *   继承内置对象
    *   Symbol.species 属性
*   在类构造器中使用 new.target
*   总结

### ES5 中的仿类结构

JS 在 ES5 及更早版本中都不存在类。与类最接近的是：创建一个构造器，然后将方法指派到该构造器的原型上。这种方式通常被称为创建一个自定义类型。例如：

```
function PersonType(name) {
    this.name = name;
}

PersonType.prototype.sayName = function() {
    console.log(this.name);
};

let person = new PersonType("Nicholas");
person.sayName();   // 输出 "Nicholas"

console.log(person instanceof PersonType);  // true
console.log(person instanceof Object);      // true 
```

此代码中的 `PersonType` 是一个构造器函数，并创建了单个属性 `name` 。 `sayName()` 方法被指派到原型上，因此在 `PersonType` 对象的所有实例上都共享了此方法。接下来，使用 `new` 运算符创建了 `PersonType` 的一个新实例 `person` ，此对象会被认为是一个通过原型继承了 `PersonType` 与 `Object` 的实例。

这种基本模式在许多对类进行模拟的 JS 库中都存在，而这也是 ES6 类的出发点。

### 类的声明

类在 ES6 中最简单的形式就是类声明，它看起来很像其他语言中的类。

#### 基本的类声明

类声明以 `class` 关键字开始，其后是类的名称；剩余部分的语法看起来就像对象字面量中的方法简写，并且在方法之间不需要使用逗号。作为范例，此处有个简单的类声明：

```
class PersonClass {

    // 等价于 PersonType 构造器
    constructor(name) {
        this.name = name;
    }

    // 等价于 PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
}

let person = new PersonClass("Nicholas");
person.sayName();   // 输出 "Nicholas"

console.log(person instanceof PersonClass);     // true
console.log(person instanceof Object);          // true

console.log(typeof PersonClass);                    // "function"
console.log(typeof PersonClass.prototype.sayName);  // "function" 
```

这个 `PersonClass` 类声明的行为非常类似上个例子中的 `PersonType` 。类声明允许你在其中使用特殊的 `constructor` 方法名称直接定义一个构造器，而不需要先定义一个函数再把它当作构造器使用。由于类的方法使用了简写语法，于是就不再需要使用 `function` 关键字。 `constructor` 之外的方法名称则没有特别的含义，因此可以随你高兴自由添加方法。

> **自有属性**（ **Own properties** ）：该属性出现在实例上而不是原型上，只能在类的构造器或方法内部进行创建。在本例中， `name` 就是一个自有属性。我建议应在构造器函数内创建所有可能出现的自有属性，这样在类中声明变量就会被限制在单一位置（有助于代码检查）。

有趣的是，相对于已有的自定义类型声明方式来说，类声明仅仅是以它为基础的一个语法糖。 `PersonClass` 声明实际上创建了一个拥有 `constructor` 方法及其行为的函数，这也是 `typeof PersonClass` 会得到 `"function"` 结果的原因。此例中的 `sayName()` 方法最终也成为 `PersonClass.prototype` 上的一个方法，类似于上个例子中 `sayName()` 与 `PersonType.prototype` 之间的关系。这些相似处允许你把自定义类型与类混合使用，而不必太担忧到底该用哪个。

#### 为何要使用类的语法

尽管类与自定义类型之间有相似性，但仍然要记住一些重要的区别：

1.  类声明不会被提升，这与函数定义不同。类声明的行为与 `let` 相似，因此在程序的执行到达声明处之前，类会存在于暂时性死区内。
2.  类声明中的所有代码会自动运行在严格模式下，并且也无法退出严格模式。
3.  类的所有方法都是不可枚举的，这是对于自定义类型的显著变化，后者必须用 `Object.defineProperty()` 才能将方法改变为不可枚举。
4.  类的所有方法内部都没有 `[[Construct]]` ，因此使用 `new` 来调用它们会抛出错误。
5.  调用类构造器时不使用 `new` ，会抛出错误。
6.  试图在类的方法内部重写类名，会抛出错误。

这样看来，上例中的 `PersonClass` 声明实际上就直接等价于以下未使用类语法的代码：

```
// 直接等价于 PersonClass
let PersonType2 = (function() {
 "use strict";

    const PersonType2 = function(name) {

        // 确认函数被调用时使用了 new
        if (typeof new.target === "undefined") {
            throw new Error("Constructor must be called with new.");
        }

        this.name = name;
    }

    Object.defineProperty(PersonType2.prototype, "sayName", {
        value: function() {

            // 确认函数被调用时没有使用 new
            if (typeof new.target !== "undefined") {
                throw new Error("Method cannot be called with new.");
            }

            console.log(this.name);
        },
        enumerable: false,
        writable: true,
        configurable: true
    });

    return PersonType2;
}()); 
```

首先要注意这里有两个 `PersonType2` 声明：一个在外部作用域的 `let` 声明，一个在 IIFE 内部的 `const` 声明。这就是为何类的方法不能对类名进行重写、而类外部的代码则被允许。构造器函数检查了 `new.target` ，以保证被调用时使用了 `new` ，否则就抛出错误。接下来， `sayName()` 方法被定义为不可枚举，并且此方法也检查了 `new.target` ，它则要保证在被调用时没有使用 `new` 。最后一步是将构造器函数返回出去。

此例说明了尽管不使用新语法也能实现类的任何特性，但类语法显著简化了所有功能的代码。

> **不变的类名**
> 
> 只有在类的内部，类名才被视为是使用 `const` 声明的。这意味着你可以在外部重写类名，但不能在类的方法内部这么做。例如：
> 
> ```
> class Foo {
>   constructor() {
>       Foo = "bar";    // 执行时抛出错误
>   }
> }
> 
> // 但在类声明之后没问题
> Foo = "baz"; 
> ```
> 
> 在此代码中，类构造器内部的 `Foo` 与在类外部的 `Foo` 是不同的绑定。内部的 `Foo` 就像是用 `const` 定义的，不能被重写，当构造器尝试使用任何值重写 `Foo` 时，都会抛出错误。但由于外部的 `Foo` 就像是用 `let` 声明的，你可以随时重写类名。

### 类表达式

类与函数有相似之处，即它们都有两种形式：声明与表达式。函数声明与类声明都以适当的关键词为起始（分别是 `function` 与 `class` ），随后是标识符（即函数名或类名）。函数具有一种表达式形式，无须在 `function` 后面使用标识符；类似的，类也有不需要标识符的表达式形式。**类表达式**被设计用于变量声明，或可作为参数传递给函数。

#### 基本的类表达式

此处是与上例中的 `PersonClass` 等效的类表达式，随后的代码使用了它：

```
let PersonClass = class {

    // 等价于 PersonType 构造器
    constructor(name) {
        this.name = name;
    }

    // 等价于 PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
};

let person = new PersonClass("Nicholas");
person.sayName();   // 输出 "Nicholas"

console.log(person instanceof PersonClass);     // true
console.log(person instanceof Object);          // true

console.log(typeof PersonClass);                    // "function"
console.log(typeof PersonClass.prototype.sayName);  // "function" 
```

正如此例所示，类表达式不需要在 `class` 关键字后使用标识符。除了语法差异，类表达式的功能等价于类声明。

在匿名的类表达式内 `PersonClass.name` 是个空的字符串。而若使用了类声明，则 `PersonClass.name` 就会是 `"PersonClass"` 。

> 使用类声明还是类表达式，主要是代码风格问题。相对于函数声明与函数表达式之间的区别，类声明与类表达式都不会被提升，因此对代码运行时的行为影响甚微。唯一显著的差异是匿名类表达式的 `name` 属性是一个空字符串，而类声明的 `name` 属性则与类名相同（例如，使用类声明时， `PersonClass.name` 的值为 `"PersonClass"` ）。

* * *

> **译注：**测试发现匿名类表达式的名称只在 FireFox 中才是空字符串，而其他浏览器都会使用表达式赋值目标的变量的名称。（该测试由 **oshotokill** 提供）

#### 具名类表达式

上一节的示例使用了一个匿名的类表达式，不过就像函数表达式那样，你也可以为类表达式命名。为此需要在 `class` 关键字后添加标识符，就像这样：

```
let PersonClass = class PersonClass2 {

    // 等价于 PersonType 构造器
    constructor(name) {
        this.name = name;
    }

    // 等价于 PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
};

console.log(typeof PersonClass);        // "function"
console.log(typeof PersonClass2);       // "undefined" 
```

此例中的类表达式被命名为 `PersonClass2` 。 `PersonClass2` 标识符只在类定义内部存在，因此只能用在类方法内部（例如本例的 `sayName()` 内）。在类的外部， `typeof PersonClass2` 的结果为 `"undefined"` ，这是因为外部不存在 `PersonClass2` 绑定。要理解为何如此，请查看未使用类语法的等价声明：

```
// 直接等价于 PersonClass 具名的类表达式
let PersonClass = (function() {
 "use strict";

    const PersonClass2 = function(name) {

        // 确认函数被调用时使用了 new
        if (typeof new.target === "undefined") {
            throw new Error("Constructor must be called with new.");
        }

        this.name = name;
    }

    Object.defineProperty(PersonClass2.prototype, "sayName", {
        value: function() {

            // 确认函数被调用时没有使用 new
            if (typeof new.target !== "undefined") {
                throw new Error("Method cannot be called with new.");
            }

            console.log(this.name);
        },
        enumerable: false,
        writable: true,
        configurable: true
    });

    return PersonClass2;
}()); 
```

创建具名的类表达式稍微改变了在 JS 引擎内部发生的事。对于类声明来说，外部绑定（用 `let` 定义）与内部绑定（用 `const` 定义）有着相同的名称。而类表达式可在内部使用 `const` 来定义它的不同名称，因此此处的 `PersonClass2` 只能在类的内部使用。

尽管具名类表达式的行为异于具名函数表达式，但它们之间仍然有许多相似点。二者都能被当作值来使用，这开启了许多可能性，接下来我将会对此进行介绍。

### 作为一级公民的类

在编程中，能被当作值来使用的就称为**一级公民**（ **first-class citizen** ），意味着它能作为参数传给函数、能作为函数返回值、能用来给变量赋值。 JS 的函数就是一级公民（它们有时又被称为一级函数），此特性让 JS 独一无二。

ES6 延续了传统，让类同样成为一级公民。这就使得类可以被多种方式所使用。例如，它能作为参数传入函数：

```
function createObject(classDef) {
    return new classDef();
}

let obj = createObject(class {

    sayHi() {
        console.log("Hi!");
    }
});

obj.sayHi();        // "Hi!" 
```

此例中的 `createObject()` 函数被调用时接收了一个匿名类表达式作为参数，使用 `new` 创建了该类的一个实例，并将其返回出来。随后变量 `obj` 储存了所返回的实例。

类表达式的另一个有趣用途是立即调用类构造器，以创建单例（ Singleton ）。为此，你必须使用 `new` 来配合类表达式，并在表达式后面添加括号。例如：

```
let person = new class {

    constructor(name) {
        this.name = name;
    }

    sayName() {
        console.log(this.name);
    }

}("Nicholas");

person.sayName();       // "Nicholas" 
```

此处创建了一个匿名类表达式，并立即执行了它。此模式允许你使用类语法来创建单例，从而不留下任何可被探查的类引用（回忆一下 `PersonClass` 的例子，匿名类表达式只在类的内部创建了绑定，而外部无绑定）。类表达式后面的圆括号表示要调用前面的函数，并且还允许传入参数。

本章至今的例子都集中于带有方法的类，但你还能在类上创建访问器属性，所用的语法类似于对象字面量。

### 访问器属性

自有属性需要在类构造器中创建，而类还允许你在原型上定义访问器属性。为了创建一个 getter ，要使用 `get` 关键字，并要与后方标识符之间留出空格；创建 setter 用相同方式，只是要换用 `set` 关键字。例如：

```
class CustomHTMLElement {

    constructor(element) {
        this.element = element;
    }

    get html() {
        return this.element.innerHTML;
    }

    set html(value) {
        this.element.innerHTML = value;
    }
}

var descriptor = Object.getOwnPropertyDescriptor(CustomHTMLElement.prototype, "html");
console.log("get" in descriptor);   // true
console.log("set" in descriptor);   // true
console.log(descriptor.enumerable); // false 
```

此代码中的 `CustomHTMLElement` 类用于包装一个已存在的 DOM 元素。它的属性 `html` 拥有 getter 与 setter ，委托了元素自身的 `innerHTML` 方法。该访问器属性被创建在 `CustomHTMLElement.prototype` 上，并且像其他类属性那样被创建为不可枚举属性。非类的等价表示如下：

```
// 直接等价于上个范例
let CustomHTMLElement = (function() {
 "use strict";

    const CustomHTMLElement = function(element) {

        // 确认函数被调用时使用了 new
        if (typeof new.target === "undefined") {
            throw new Error("Constructor must be called with new.");
        }

        this.element = element;
    }

    Object.defineProperty(CustomHTMLElement.prototype, "html", {
        enumerable: false,
        configurable: true,
        get: function() {
            return this.element.innerHTML;
        },
        set: function(value) {
            this.element.innerHTML = value;
        }
    });

    return CustomHTMLElement;
}()); 
```

正如之前的例子，此例说明了使用类语法能够少写大量的代码。仅仅为 `html` 访问器属性定义的代码量，就几乎相当于等价的类声明的全部代码量了。

### 需计算的成员名

对象字面量与类之间的相似点还不仅前面那些。类方法与类访问器属性也都能使用需计算的名称。语法相同于对象字面量中的需计算名称：无须使用标识符，而是用方括号来包裹一个表达式。例如：

```
let methodName = "sayName";

class PersonClass {

    constructor(name) {
        this.name = name;
    }

    [methodName]() {
        console.log(this.name);
    }
}

let me = new PersonClass("Nicholas");
me.sayName();           // "Nicholas" 
```

此版本的 `PersonClass` 使用了一个变量来命名类定义内的方法。字符串 `"sayName"` 被赋值给了 `methodName` 变量，而 `methodName` 变量则被用于声明方法。 `sayName()` 方法在此后能被直接访问。

访问器属性能以相同方式使用需计算的名称，就像这样：

```
let propertyName = "html";

class CustomHTMLElement {

    constructor(element) {
        this.element = element;
    }

    get [propertyName]() {
        return this.element.innerHTML;
    }

    set propertyName {
        this.element.innerHTML = value;
    }
} 
```

此处 `html` 的 getter 与 setter 被设置为需使用 `propertyName` 变量，使用 `.html` 依然能访问此属性，这里影响的只有定义方式。

你已经看到了在类与对象字面量之间有许多相似点，包括方法、访问器属性、需计算的名称。此外还有一个相似点需要介绍，即生成器。

### 生成器方法

第八章介绍了生成器，你已学会如何在对象字面量上定义一个生成器：只要在方法名称前附加一个星号（ `*` ）。这一语法对类同样有效，允许将任何方法变为一个生成器。此处有个范例：

```
class MyClass {

    *createIterator() {
        yield 1;
        yield 2;
        yield 3;
    }

}

let instance = new MyClass();
let iterator = instance.createIterator(); 
```

此代码创建了一个拥有 `createIterator()` 生成器的 `MyClass` 类。该方法返回了一个迭代器，它的值在生成器内部用硬编码提供。当你使用一个对象来表示值的集合、并要求能简单迭代这些值，那么生成器方法就非常有用。数组、 Set 与 Map 都拥有多个生成器方法，负责让开发者用多种方式来操作它们的项。

既然生成器方法很有用，那么在表示集合的自定义类中定义一个默认迭代器，那就更好。你可以使用 `Symbol.iterator` 来定义生成器方法，从而定义出类的默认迭代器，就像这样：

```
class Collection {

    constructor() {
        this.items = [];
    }

    *[Symbol.iterator]() {
        yield *this.items.values();
    }
}

var collection = new Collection();
collection.items.push(1);
collection.items.push(2);
collection.items.push(3);

for (let x of collection) {
    console.log(x);
}

// 输出：
// 1
// 2
// 3 
```

此例为生成器方法使用了一个需计算名称，并将此方法委托到 `this.items` 数组的 `values()` 迭代器上。任意管理集合的类都包含一个默认迭代器，这是因为一些集合专用的操作都要求目标集合具有迭代器。现在， `Collection` 的任意实例都可以在 `for-of` 循环内被直接使用，也能配合扩展运算符使用。

当你想让方法与访问器属性在对象实例上出现时，把它们添加到类的原型上就会对此目的有帮助。而另一方面，若想让方法与访问器属性只存在于类自身，那么你就需要使用静态成员。

### 静态成员

直接在构造器上添加额外方法来模拟静态成员，这在 ES5 及更早版本中是另一个通用的模式。例如：

```
function PersonType(name) {
    this.name = name;
}

// 静态方法
PersonType.create = function(name) {
    return new PersonType(name);
};

// 实例方法
PersonType.prototype.sayName = function() {
    console.log(this.name);
};

var person = PersonType.create("Nicholas"); 
```

在其他编程语言中，工厂方法 `PersonType.create()` 会被认定为一个静态方法，它的数据不依赖 `PersonType` 的任何实例。 ES6 的类简化了静态成员的创建，只要在方法与访问器属性的名称前添加正式的 `static` 标注。作为一个例子，此处有个与上例等价的类：

```
class PersonClass {

    // 等价于 PersonType 构造器
    constructor(name) {
        this.name = name;
    }

    // 等价于 PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }

    // 等价于 PersonType.create
    static create(name) {
        return new PersonClass(name);
    }
}

let person = PersonClass.create("Nicholas"); 
```

`PersonClass` 的定义拥有名为 `create()` 的单个静态方法，此语法与 `sayName()` 基本相同，只多了一个 `static` 关键字。你能在类中的任何方法与访问器属性上使用 `static` 关键字，唯一限制是不能将它用于 `constructor` 方法的定义。

> 静态成员不能用实例来访问，你始终需要直接用类自身来访问它们。

### 使用派生类进行继承

ES6 之前，实现自定义类型的继承是个繁琐的过程。严格的继承要求有多个步骤。例如，研究以下范例：

```
function Rectangle(length, width) {
    this.length = length;
    this.width = width;
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width;
};

function Square(length) {
    Rectangle.call(this, length, length);
}

Square.prototype = Object.create(Rectangle.prototype, {
    constructor: {
        value:Square,
        enumerable: true,
        writable: true,
        configurable: true
    }
});

var square = new Square(3);

console.log(square.getArea());              // 9
console.log(square instanceof Square);      // true
console.log(square instanceof Rectangle);   // true 
```

`Square` 继承了 `Rectangle` ，为此它必须使用 `Rectangle.prototype` 所创建的一个新对象来重写 `Square.prototype` ，并且还要调用 `Rectangle.call()` 方法。这些步骤常常会搞晕 JS 的新手，并会成为有经验开发者出错的根源之一。

类让继承工作变得更轻易，使用熟悉的 `extends` 关键字来指定当前类所需要继承的函数，即可。生成的类的原型会被自动调整，而你还能调用 `super()` 方法来访问基类的构造器。此处是与上个例子等价的 ES6 代码：

```
class Rectangle {
    constructor(length, width) {
        this.length = length;
        this.width = width;
    }

    getArea() {
        return this.length * this.width;
    }
}

class Square extends Rectangle {
    constructor(length) {

        // 与 Rectangle.call(this, length, length) 相同
        super(length, length);
    }
}

var square = new Square(3);

console.log(square.getArea());              // 9
console.log(square instanceof Square);      // true
console.log(square instanceof Rectangle);   // true 
```

此次 `Square` 类使用了 `extends` 关键字继承了 `Rectangle` 。 `Square` 构造器使用了 `super()` 配合指定参数调用了 `Rectangle` 的构造器。注意与 ES5 版本的代码不同， `Rectangle` 标识符仅在类定义时被使用了（在 `extends` 之后）。

继承了其他类的类被称为**派生类**（ **derived classes** ）。如果派生类指定了构造器，就需要使用 `super()` ，否则会造成错误。若你选择不使用构造器， `super()` 方法会被自动调用，并会使用创建新实例时提供的所有参数。例如，下列两个类是完全相同的：

```
class Square extends Rectangle {
    // 没有构造器
}

// 等价于：

class Square extends Rectangle {
    constructor(...args) {
        super(...args);
    }
} 
```

此例中的第二个类展示了与所有派生类默认构造器等价的写法，所有的参数都按顺序传递给了基类的构造器。在当前需求下，这种做法并不完全准确，因为 `Square` 构造器只需要单个参数，因此最好手动定义构造器。

> 使用 `super()` 时需牢记以下几点：
> 
> 1.  你只能在派生类中使用 `super()` 。若尝试在非派生的类（即：没有使用 `extends` 关键字的类）或函数中使用它，就会抛出错误。
> 2.  在构造器中，你必须在访问 `this` 之前调用 `super()` 。由于 `super()` 负责初始化 `this` ，因此试图先访问 `this` 自然就会造成错误。
> 3.  唯一能避免调用 `super()` 的办法，是从类构造器中返回一个对象。

#### 屏蔽类方法

派生类中的方法总是会屏蔽基类的同名方法。例如，你可以将 `getArea()` 方法添加到 `Square` 类，以便重定义它的功能：

```
class Square extends Rectangle {
    constructor(length) {
        super(length, length);
    }

    // 重写并屏蔽 Rectangle.prototype.getArea()
    getArea() {
        return this.length * this.length;
    }
} 
```

由于 `getArea()` 已经被定义为 `Square` 的一部分， `Rectangle.prototype.getArea()` 方法就不能在 `Square` 的任何实例上被调用。当然，你总是可以使用 `super.getArea()` 方法来调用基类中的同名方法，就像这样：

```
class Square extends Rectangle {
    constructor(length) {
        super(length, length);
    }

    // 重写、屏蔽并调用了 Rectangle.prototype.getArea()
    getArea() {
        return super.getArea();
    }
} 
```

用这种方式使用 `super` ，其效果等同于第四章讨论过的 super 引用（详见“使用 super 引用的简单原型访问”）。 `this` 值会被自动设置为正确的值，因此你就能进行简单的调用。

#### 继承静态成员

如果基类包含静态成员，那么这些静态成员在派生类中也是可用的。继承的工作方式类似于其他语言，但对于 JS 而言则是新概念。此处有个范例：

```
class Rectangle {
    constructor(length, width) {
        this.length = length;
        this.width = width;
    }

    getArea() {
        return this.length * this.width;
    }

    static create(length, width) {
        return new Rectangle(length, width);
    }
}

class Square extends Rectangle {
    constructor(length) {

        // 与 Rectangle.call(this, length, length) 相同
        super(length, length);
    }
}

var rect = Square.create(3, 4);

console.log(rect instanceof Rectangle);     // true
console.log(rect.getArea());                // 12
console.log(rect instanceof Square);        // false 
```

在此代码中，一个新的静态方法 `create()` 被添加到 `Rectangle` 类中。通过继承，该方法会以 `Square.create()` 的形式存在，并且其行为方式与 `Rectangle.create()` 一样。

#### 从表达式中派生类

在 ES6 中派生类的最强大能力，或许就是能够从表达式中派生类。只要一个表达式能够返回一个具有 `[[Construct]]` 属性以及原型的函数，你就可以对其使用 `extends` 。例如：

```
function Rectangle(length, width) {
    this.length = length;
    this.width = width;
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width;
};

class Square extends Rectangle {
    constructor(length) {
        super(length, length);
    }
}

var x = new Square(3);
console.log(x.getArea());               // 9
console.log(x instanceof Rectangle);    // true 
```

`Rectangle` 被定义为 ES5 风格的构造器，而 `Square` 则是一个类。由于 `Rectangle` 具有 `[[Construct]]` 以及原型， `Square` 类就能直接继承它。

`extends` 后面能接受任意类型的表达式，这带来了巨大可能性，例如动态地决定所要继承的类：

```
function Rectangle(length, width) {
    this.length = length;
    this.width = width;
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width;
};

function getBase() {
    return Rectangle;
}

class Square extends getBase() {
    constructor(length) {
        super(length, length);
    }
}

var x = new Square(3);
console.log(x.getArea());               // 9
console.log(x instanceof Rectangle);    // true 
```

`getBase()` 函数作为类声明的一部分被直接调用，它返回了 `Rectangle` ，使得此例的功能等价于前一个例子。并且由于可以动态地决定基类，那也就能创建不同的继承方式。例如，你可以有效地创建混入：

```
let SerializableMixin = {
    serialize() {
        return JSON.stringify(this);
    }
};

let AreaMixin = {
    getArea() {
        return this.length * this.width;
    }
};

function mixin(...mixins) {
    var base = function() {};
    Object.assign(base.prototype, ...mixins);
    return base;
}

class Square extends mixin(AreaMixin, SerializableMixin) {
    constructor(length) {
        super();
        this.length = length;
        this.width = length;
    }
}

var x = new Square(3);
console.log(x.getArea());               // 9
console.log(x.serialize());             // "{"length":3,"width":3}" 
```

此例使用了混入（ mixin ）而不是传统继承。 `mixin()` 函数接受代表混入对象的任意数量的参数，它创建了一个名为 `base` 的函数，并将每个混入对象的属性都赋值到新函数的原型上。此函数随后被返回，于是 `Square` 就能够对其使用 `extends` 关键字了。注意由于仍然使用了 `extends` ，你就必须在构造器内调用 `super()` 。

`Square` 的实例既有来自 `AreaMixin` 的 `getArea()` 方法，又有来自 `SerializableMixin` 的 `serialize()` 方法，这是通过原型继承实现的。 `mixin()` 函数使用了混入对象的所有自有属性，动态地填充了新函数的原型（记住：若多个混入对象拥有相同的属性，则只有最后添加的属性会被保留）。

> 任意表达式都能在 `extends` 关键字后使用，但并非所有表达式的结果都是一个有效的类。特别的，下列表达式类型会导致错误：
> 
> *   `null` ；
> *   生成器函数（详见第八章）。
> 
> 试图使用结果为上述值的表达式来创建一个新的类实例，都会抛出错误，因为不存在 `[[Construct]]` 可供调用。

#### 继承内置对象

几乎从 JS 数组出现那天开始，开发者就想通过继承机制来创建他们自己的特殊数组类型。在 ES5 及早期版本中，这是不可能做到的。试图使用传统继承并不能产生功能正确的代码，例如：

```
// 内置数组的行为
var colors = [];
colors[0] = "red";
console.log(colors.length);         // 1

colors.length = 0;
console.log(colors[0]);             // undefined

// 在 ES5 中尝试继承数组

function MyArray() {
    Array.apply(this, arguments);
}

MyArray.prototype = Object.create(Array.prototype, {
    constructor: {
        value: MyArray,
        writable: true,
        configurable: true,
        enumerable: true
    }
});

var colors = new MyArray();
colors[0] = "red";
console.log(colors.length);         // 0

colors.length = 0;
console.log(colors[0]);             // "red" 
```

`console.log()` 在此代码尾部的输出说明了：对数组使用传统形式的 JS 继承，产生了预期外的行为。 `MyArray` 实例上的 `length` 属性以及数值属性，其行为与内置数组并不一致，因为这些功能并未被涵盖在 `Array.apply()` 或数组原型中。

在 ES6 中的类，其设计目的之一就是允许从内置对象上进行继承。为了达成这个目的，类的继承模型与 ES5 或更早版本的传统继承模型有轻微差异，体现在以下两个重要方面：

*   在 ES5 的传统继承中， `this` 的值会先被派生类（例如 `MyArray` ）创建，随后基类构造器（例如 `Array.apply()` 方法）才被调用。这意味着 `this` 一开始就是 `MyArray` 的实例，之后才使用了 `Array` 的附加属性对其进行了装饰。
*   在 ES6 基于类的继承中， `this` 的值会先被基类（ `Array` ）创建，随后才被派生类的构造器（ `MyArray` ）所修改。结果是 `this` 初始就拥有作为基类的内置对象的所有功能，并能正确接收与之关联的所有功能。

以下范例实际展示了基于类的特殊数组：

```
class MyArray extends Array {
    // 空代码块
}

var colors = new MyArray();
colors[0] = "red";
console.log(colors.length);         // 1

colors.length = 0;
console.log(colors[0]);             // undefined 
```

`MyArray` 直接继承了 `Array` ，因此工作方式与正规数组一致。与数值索引属性的互动更新了 `length` 属性，而操纵 `length` 属性也能更新索引属性。这意味着你既能适当地继承 `Array` 来创建你自己的派生数组类，也同样能继承其他的内置对象。伴随着这些附加功能， ES6 与派生类型有效解决了从内置类型进行派生这最后的特殊情况，不过这种情况仍然值得继续探索。

#### Symbol.species 属性

继承内置对象一个有趣的方面是：任意能返回内置对象实例的方法，在派生类上却会自动返回派生类的实例。因此，若你拥有一个继承了 `Array` 的派生类 `MyArray` ，诸如 `slice()` 之类的方法都会返回 `MyArray` 的实例。例如：

```
class MyArray extends Array {
    // 空代码块
}

let items = new MyArray(1, 2, 3, 4),
    subitems = items.slice(1, 3);

console.log(items instanceof MyArray);      // true
console.log(subitems instanceof MyArray);   // true 
```

在此代码中， `slice()` 方法返回了 `MyArray` 的一个实例。 `slice()` 方法是从 `Array` 上继承的，原本应当返回 `Array` 的一个实例。而 `Symbol.species` 属性在后台造成了这种变化。

`Symbol.species` 知名符号被用于定义一个能返回函数的静态访问器属性。每当类实例的方法（构造器除外）必须创建一个实例时，前面返回的函数就被用为新实例的构造器。下列内置类型都定义了 `Symbol.species` ：

*   `Array`
*   `ArrayBuffer` （详见第十章）
*   `Map`
*   `Promise`
*   `RegExp`
*   `Set`
*   类型化数组（详见第十章）

以上每个类型都拥有默认的 `Symbol.species` 属性，其返回值为 `this` ，意味着该属性总是会返回自身的构造器函数。若你准备在一个自定义类上实现此功能，代码就像这样：

```
// 几个内置类型使用 species 的方式类似于此
class MyClass {
    static get [Symbol.species]() {
        return this;
    }

    constructor(value) {
        this.value = value;
    }

    clone() {
        return new this.constructorSymbol.species;
    }
} 
```

在此例中， `Symbol.species` 知名符号被用于定义 `MyClass` 的一个静态访问器属性。注意此处只有 getter 而没有 setter ，这是因为修改类的 species 是不允许的。任何对 `this.constructor[Symbol.species]` 的调用都会返回 `MyClass` ， `clone()` 方法使用了该定义来返回一个新的实例，而没有直接使用 `MyClass` ，这就允许派生类重写这个值。例如：

```
class MyClass {
    static get [Symbol.species]() {
        return this;
    }

    constructor(value) {
        this.value = value;
    }

    clone() {
        return new this.constructorSymbol.species;
    }
}

class MyDerivedClass1 extends MyClass {
    // 空代码块
}

class MyDerivedClass2 extends MyClass {
    static get [Symbol.species]() {
        return MyClass;
    }
}

let instance1 = new MyDerivedClass1("foo"),
    clone1 = instance1.clone(),
    instance2 = new MyDerivedClass2("bar"),
    clone2 = instance2.clone();

console.log(clone1 instanceof MyClass);             // true
console.log(clone1 instanceof MyDerivedClass1);     // true
console.log(clone2 instanceof MyClass);             // true
console.log(clone2 instanceof MyDerivedClass2);     // false 
```

此处, `MyDerivedClass1` 继承了 `MyClass` ，并且未修改 `Symbol.species` 属性。由于 `this.constructor[Symbol.species]` 会返回 `MyDerivedClass1` ，当 `clone()` 被调用时，它就返回了 `MyDerivedClass1` 的一个实例。 `MyDerivedClass2` 类也继承了 `MyClass` ，但重写了 `Symbol.species` ，让其返回 `MyClass` 。当 `clone()` 在 `MyDerivedClass2` 的一个实例上被调用时，返回值就变成 `MyClass` 的一个实例。使用 `Symbol.species` ，任意派生类在调用应当返回实例的方法时，都可以判断出需要返回什么类型的值。

例如， `Array` 使用了 `Symbol.species` 来指定方法所使用的类，让其返回值为一个数组。在 `Array` 派生出的类中，你可以决定这些继承的方法应返回何种类型的对象，正如：

```
class MyArray extends Array {
    static get [Symbol.species]() {
        return Array;
    }
}

let items = new MyArray(1, 2, 3, 4),
    subitems = items.slice(1, 3);

console.log(items instanceof MyArray);      // true
console.log(subitems instanceof Array);     // true
console.log(subitems instanceof MyArray);   // false 
```

此代码重写了从 `Array` 派生的 `MyArray` 类上的 `Symbol.species` 。所有返回数组的继承方法现在都会使用 `Array` 的实例，而不是 `MyArray` 的实例。

一般而言，每当想在类方法中使用 `this.constructor` 时，你就应当设置类的 `Symbol.species` 属性。这么做允许派生类轻易地重写方法的返回类型。此外，若你从一个拥有 `Symbol.species` 定义的类创建了派生类，要保证使用此属性，而不是直接使用构造器。

### 在类构造器中使用 new.target

在第三章你已学到了 `new.target` ，以及在调用函数的方式不同时它的值是如何变动的。你也可以在类构造器中使用 `new.target` ，来判断类是被如何被调用的。在简单情况下， `new.target` 就等于本类的构造器函数，正如下例；

```
class Rectangle {
    constructor(length, width) {
        console.log(new.target === Rectangle);
        this.length = length;
        this.width = width;
    }
}

// new.target 就是 Rectangle
var obj = new Rectangle(3, 4);      // 输出 true 
```

此代码说明在 `new Rectangle(3, 4)` 被调用时， `new.target` 就等于 `Rectangle` 。类构造器被调用时不能缺少 `new` ，因此 `new.target` 属性就始终会在类构造器内被定义。不过这个值并不总是相同的。研究以下代码：

```
class Rectangle {
    constructor(length, width) {
        console.log(new.target === Rectangle);
        this.length = length;
        this.width = width;
    }
}

class Square extends Rectangle {
    constructor(length) {
        super(length, length)
    }
}

// new.target 就是 Square
var obj = new Square(3);      // 输出 false 
```

`Square` 调用了 `Rectangle` 构造器，因此当 `Rectangle` 构造器被调用时， `new.target` 等于 `Square` 。这很重要，因为构造器能根据如何被调用而有不同行为，并且这给了更改这种行为的能力。例如，你可以使用 `new.target` 来创建一个抽象基类（一种不能被实例化的类），如下：

```
// 静态的基类
class Shape {
    constructor() {
        if (new.target === Shape) {
            throw new Error("This class cannot be instantiated directly.")
        }
    }
}

class Rectangle extends Shape {
    constructor(length, width) {
        super();
        this.length = length;
        this.width = width;
    }
}

var x = new Shape();                // 抛出错误

var y = new Rectangle(3, 4);        // 没有错误
console.log(y instanceof Shape);    // true 
```

此例中的 `Shape` 类构造器会在 `new.target` 为 `Shape` 的时候抛出错误，意味着 `new Shape()` 永远都会抛出错误。然而，你依然可以将 `Shape` 用作一个基类，正如 `Rectangle` 所做的那样。 `super()` 的调用执行了 `Shape` 构造器，而且 `new.target` 的值等于 `Rectangle` ，因此该构造器能够无错误地继续执行。

> 由于调用类时不能缺少 `new` ，于是 `new.target` 属性在类构造器内部就绝不会是 `undefined` 。

### 总结

ES6 的类让 JS 中的继承变得更简单，因此对于你已从其他语言学习到的类知识，你无须将其丢弃。 ES6 的类起初是作为 ES5 传统继承模型的语法糖，但添加了许多特性来减少错误。

ES6 的类配合原型继承来工作，在类的原型上定义了非静态的方法，而静态的方法最终则被绑定在类构造器自身上。类的所有方法初始都是不可枚举的，这更契合了内置对象的行为，后者的方法默认情况下通常都不可枚举。此外，类构造器被调用时不能缺少 `new` ，确保了不能意外地将类作为函数来调用。

基于类的继承允许你从另一个类、函数或表达式上派生新的类。这种能力意味着你可以调用一个函数来判断需要继承的正确基类，也允许你使用混入或其他不同的组合模式来创建一个新类。新的继承方式让继承内置对象（例如数组）也变为可能，并且其工作符合预期。

你可以在类构造器内部使用 `new.target` ，以便根据类如何被调用来做出不同的行为。最常用的就是创建一个抽象基类，直接实例化它会抛出错误，但它仍然允许被其他类所继承。

总之，类是 JS 的一项新特性，它提供了更简洁的语法与更好的功能，通过安全一致的方式来自定义一个对象类型。