# 混入

# 介绍

除了传统的面向对象继承方式，还流行一种通过可重用组件创建类的方式，就是联合另一个简单类的代码。 你可能在 Scala 等语言里对 mixins 及其特性已经很熟悉了，但它在 JavaScript 中也是很流行的。

# 混入示例

下面的代码演示了如何在 TypeScript 里使用混入。 后面我们还会解释这段代码是怎么工作的。

```js
// Disposable Mixin
class Disposable {
    isDisposed: boolean;
    dispose() {
        this.isDisposed = true;
    }

}

// Activatable Mixin
class Activatable {
    isActive: boolean;
    activate() {
        this.isActive = true;
    }
    deactivate() {
        this.isActive = false;
    }
}

class SmartObject implements Disposable, Activatable {
    constructor() {
        setInterval(() => console.log(this.isActive + " : " + this.isDisposed), 500);
    }

    interact() {
        this.activate();
    }

    // Disposable
    isDisposed: boolean = false;
    dispose: () => void;
    // Activatable
    isActive: boolean = false;
    activate: () => void;
    deactivate: () => void;
}
applyMixins(SmartObject, [Disposable, Activatable]);

let smartObj = new SmartObject();
setTimeout(() => smartObj.interact(), 1000);

////////////////////////////////////////
// In your runtime library somewhere
////////////////////////////////////////

function applyMixins(derivedCtor: any, baseCtors: any[]) {
    baseCtors.forEach(baseCtor => {
        Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
            derivedCtor.prototype[name] = baseCtor.prototype[name];
        });
    });
} 
```

# 理解这个例子

代码里首先定义了两个类，它们将做为 mixins。 可以看到每个类都只定义了一个特定的行为或功能。 稍后我们使用它们来创建一个新类，同时具有这两种功能。

```js
// Disposable Mixin
class Disposable {
    isDisposed: boolean;
    dispose() {
        this.isDisposed = true;
    }

}

// Activatable Mixin
class Activatable {
    isActive: boolean;
    activate() {
        this.isActive = true;
    }
    deactivate() {
        this.isActive = false;
    }
} 
```

下面创建一个类，结合了这两个 mixins。 下面来看一下具体是怎么操作的：

```js
class SmartObject implements Disposable, Activatable { 
```

首先应该注意到的是，没使用`extends`而是使用`implements`。 把类当成了接口，仅使用 Disposable 和 Activatable 的类型而非其实现。 这意味着我们需要在类里面实现接口。 但是这是我们在用 mixin 时想避免的。

我们可以这么做来达到目的，为将要 mixin 进来的属性方法创建出占位属性。 这告诉编译器这些成员在运行时是可用的。 这样就能使用 mixin 带来的便利，虽说需要提前定义一些占位属性。

```js
// Disposable
isDisposed: boolean = false;
dispose: () => void;
// Activatable
isActive: boolean = false;
activate: () => void;
deactivate: () => void; 
```

最后，把 mixins 混入定义的类，完成全部实现部分。

```js
applyMixins(SmartObject, [Disposable, Activatable]); 
```

最后，创建这个帮助函数，帮我们做混入操作。 它会遍历 mixins 上的所有属性，并复制到目标上去，把之前的占位属性替换成真正的实现代码。

```js
function applyMixins(derivedCtor: any, baseCtors: any[]) {
    baseCtors.forEach(baseCtor => {
        Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
            derivedCtor.prototype[name] = baseCtor.prototype[name];
        })
    });
} 
```

# 声明合并

# 介绍

TypeScript 有一些独特的概念，有的是因为我们需要描述 JavaScript 顶级对象的类型发生了哪些变化。 这其中之一叫做`声明合并`。 理解了这个概念，对于你使用 TypeScript 去操作现有的 JavaScript 来说是大有帮助的。 同时，也会有助于理解更多高级抽象的概念。

首先，在了解如何进行声明合并之前，让我们先看一下什么叫做`声明合并`。

在这个手册里，声明合并是指编译器会把两个相同名字的声明合并成一个单独的声明。 合并后的声明同时具有那两个被合并的声明的特性。 声明合并不限于只合并两个，任意数量都可以。

# 基础概念

Typescript 中的声明会创建以下三种实体之一：命名空间，类型或者值。 用于创建命名空间的声明会新建一个命名空间：它包含了可以用（.）符号访问的一些名字。 用于创建类型的声明所做的是：用给定的名字和结构创建一种类型。 最后，创建值的声明就是那些可以在生成的 JavaScript 里看到的那部分（比如：函数和变量）。

| Declaration Type | Namespace | Type | Value |
| --- | --- | --- | --- |
| Namespace | X |  | X |
| Class |  | X | X |
| Enum |  | X | X |
| Interface |  | X |  |
| Type Alias |  | X |  |
| Function |  |  | X |
| Variable |  |  | X |

理解每个声明创建了什么，有助于理解当声明合并时什么东西被合并了。

理解了每种声明会对应创建什么对于理解如果进行声明合并是有帮助的。

# 合并接口

最简单最常见的就是合并接口，声明合并的种类是：接口合并。 从根本上说，合并的机制是把各自声明里的成员放进一个同名的单一接口里。

```js
interface Box {
    height: number;
    width: number;
}

interface Box {
    scale: number;
}

let box: Box = {height: 5, width: 6, scale: 10}; 
```

接口中非函数的成员必须是唯一的。如果多个接口中具有相同名字的非函数成员就会报错。

对于函数成员，每个同名函数声明都会被当成这个函数的一个重载。

需要注意的是，接口 A 与它后面的接口 A（把这个接口叫做 A'）合并时，A'中的重载函数具有更高的优先级。

如下例所示：

```js
interface Document {
    createElement(tagName: any): Element;
}
interface Document {
    createElement(tagName: string): HTMLElement;
}
interface Document {
    createElement(tagName: "div"): HTMLDivElement;
    createElement(tagName: "span"): HTMLSpanElement;
    createElement(tagName: "canvas"): HTMLCanvasElement;
} 
```

这三个接口合并成一个声明。 注意每组接口里的声明顺序保持不变，只是靠后的接口会出现在它前面的接口声明之前。

```js
interface Document {
    createElement(tagName: "div"): HTMLDivElement;
    createElement(tagName: "span"): HTMLSpanElement;
    createElement(tagName: "canvas"): HTMLCanvasElement;
    createElement(tagName: string): HTMLElement;
    createElement(tagName: any): Element;
} 
```

# 合并命名空间

与接口相似，同名的命名空间也会合并其成员。 命名空间会创建出命名空间和值，我们需要知道这两者都是怎么合并的。

命名空间的合并，模块导出的同名接口进行合并，构成单一命名空间内含合并后的接口。

值的合并，如果当前已经存在给定名字的命名空间，那么后来的命名空间的导出成员会被加到已经存在的那个模块里。

`Animals`声明合并示例：

```js
namespace Animals {
    export class Zebra { }
}

namespace Animals {
    export interface Legged { numberOfLegs: number; }
    export class Dog { }
} 
```

等同于：

```js
namespace Animals {
    export interface Legged { numberOfLegs: number; }

    export class Zebra { }
    export class Dog { }
} 
```

除了这些合并外，你还需要了解非导出成员是如何处理的。 非导出成员仅在其原始存在于的命名空间（未合并的）之内可见。这就是说合并之后，从其它命名空间合并进来的成员无法访问非导出成员。

下例提供了更清晰的说明：

```js
namespace Animal {
    let haveMuscles = true;

    export function animalsHaveMuscles() {
        return haveMuscles;
    }
}

namespace Animal {
    export function doAnimalsHaveMuscles() {
        return haveMuscles;  // <-- error, haveMuscles is not visible here
    }
} 
```

因为`haveMuscles`并没有导出，只有`animalsHaveMuscles`函数共享了原始未合并的命名空间可以访问这个变量。 `doAnimalsHaveMuscles`函数虽是合并命名空间的一部分，但是访问不了未导出的成员。

# 命名空间与类和函数和枚举类型合并

命名空间可以与其它类型的声明进行合并。 只要命名空间的定义符合将要合并类型的定义。合并结果包含两者的声明类型。 Typescript 使用这个功能去实现一些 JavaScript 里的设计模式。

首先，尝试将命名空间和类合并。 这让我们可以定义内部类。

```js
class Album {
    label: Album.AlbumLabel;
}
namespace Album {
    export class AlbumLabel { }
} 
```

合并规则与上面`合并命名空间`小节里讲的规则一致，我们必须导出`AlbumLabel`类，好让合并的类能访问。 合并结果是一个类并带有一个内部类。 你也可以使用命名空间为类增加一些静态属性。

除了内部类的模式，你在 JavaScript 里，创建一个函数稍后扩展它增加一些属性也是很常见的。 Typescript 使用声明合并来达到这个目的并保证类型安全。

```js
function buildLabel(name: string): string {
    return buildLabel.prefix + name + buildLabel.suffix;
}

namespace buildLabel {
    export let suffix = "";
    export let prefix = "Hello, ";
}

alert(buildLabel("Sam Smith")); 
```

相似的，命名空间可以用来扩展枚举型：

```js
enum Color {
    red = 1,
    green = 2,
    blue = 4
}

namespace Color {
    export function mixColor(colorName: string) {
        if (colorName == "yellow") {
            return Color.red + Color.green;
        }
        else if (colorName == "white") {
            return Color.red + Color.green + Color.blue;
        }
        else if (colorName == "magenta") {
            return Color.red + Color.blue;
        }
        else if (colorName == "cyan") {
            return Color.green + Color.blue;
        }
    }
} 
```

# 非法的合并

并不是所有的合并都被允许。 现在，类不能与类合并，变量与类型不能合并，接口与类不能合并。 想要模仿类的合并，请参考 Mixins in TypeScript。

# 类型推论

# 介绍

这节介绍 TypeScript 里的类型推论。即，类型是在哪里如何被推断的。

# 基础

TypeScript 里，在有些没有明确指出类型的地方，类型推论会帮助提供类型。如下面的例子

```js
let x = 3; 
```

变量`x`的类型被推断为数字。 这种推断发生在初始化变量和成员，设置默认参数值和决定函数返回值时。

大多数情况下，类型推论是直截了当地。 后面的小节，我们会浏览类型推论时的细微差别。

# 最佳通用类型

当需要从几个表达式中推断类型时候，会使用这些表达式的类型来推断出一个最合适的通用类型。例如，

```js
let x = [0, 1, null]; 
```

为了推断`x`的类型，我们必须考虑所有元素的类型。 这里有两种选择：`number`和`null`。 计算通用类型算法会考虑所有的候选类型，并给出一个兼容所有候选类型的类型。

由于最终的通用类型取自候选类型，有些时候候选类型共享相同的通用类型，但是却没有一个类型能做为所有候选类型的类型。例如：

```js
let zoo = [new Rhino(), new Elephant(), new Snake()]; 
```

这里，我们想让 zoo 被推断为`Animal[]`类型，但是这个数组里没有对象是`Animal`类型的，因此不能推断出这个结果。 为了更正，当候选类型不能使用的时候我们需要明确的指出类型：

```js
let zoo: Animal[] = [new Rhino(), new Elephant(), new Snake()]; 
```

如果没有找到最佳通用类型的话，类型推论的结果是空对象类型，`{}`。 因为这个类型没有任何成员，所以访问其成员的时候会报错。

# 上下文类型

TypeScript 类型推论也可能按照相反的方向进行。 这被叫做“按上下文归类”。按上下文归类会发生在表达式的类型与所处的位置相关时。比如：

```js
window.onmousedown = function(mouseEvent) {
    console.log(mouseEvent.buton);  //<- Error
}; 
```

这个例子会得到一个类型错误，TypeScript 类型检查器使用`Window.onmousedown`函数的类型来推断右边函数表达式的类型。 因此，就能推断出`mouseEvent`参数的类型了。 如果函数表达式不是在上下文类型的位置，`mouseEvent`参数的类型需要指定为`any`，这样也不会报错了。

如果上下文类型表达式包含了明确的类型信息，上下文的类型被忽略。 重写上面的例子：

```js
window.onmousedown = function(mouseEvent: any) {
    console.log(mouseEvent.buton);  //<- Now, no error is given
}; 
```

这个函数表达式有明确的参数类型注解，上下文类型被忽略。 这样的话就不报错了，因为这里不会使用到上下文类型。

上下文归类会在很多情况下使用到。 通常包含函数的参数，赋值表达式的右边，类型断言，对象成员和数组字面量和返回值语句。 上下文类型也会做为最佳通用类型的候选类型。比如：

```js
function createZoo(): Animal[] {
    return [new Rhino(), new Elephant(), new Snake()];
} 
```

这个例子里，最佳通用类型有 4 个候选者：`Animal`，`Rhino`，`Elephant`和`Snake`。 当然，`Animal`会被做为最佳通用类型。

# 类型兼容性

# 介绍

TypeScript 里的类型兼容性是基于结构子类型的。 结构类型是一种只使用其成员来描述类型的方式。 它正好与名义（nominal）类型形成对比。（译者注：在基于名义类型的类型系统中，数据类型的兼容性或等价性是通过明确的声明和/或类型的名称来决定的。这与结构性类型系统不同，它是基于类型的组成结构，且不要求明确地声明。） 看下面的例子：

```js
interface Named {
    name: string;
}

class Person {
    name: string;
}

let p: Named;
// OK, because of structural typing
p = new Person(); 
```

在使用基于名义类型的语言，比如 C#或 Java 中，这段代码会报错，因为 Person 类没有明确说明其实现了 Named 接口。

TypeScript 的结构性子类型是根据 JavaScript 代码的典型写法来设计的。 因为 JavaScript 里广泛地使用匿名对象，例如函数表达式和对象字面量，所以使用结构类型系统来描述这些类型比使用名义类型系统更好。

## 关于可靠性的注意事项

TypeScript 的类型系统允许某些在编译阶段无法确认其安全性的操作。当一个类型系统具此属性时，被当做是“不可靠”的。TypeScript 允许这种不可靠行为的发生是经过仔细考虑的。通过这篇文章，我们会解释什么时候会发生这种情况和其有利的一面。

# 开始

TypeScript 结构化类型系统的基本规则是，如果`x`要兼容`y`，那么`y`至少具有与`x`相同的属性。比如：

```js
interface Named {
    name: string;
}

let x: Named;
// y's inferred type is { name: string; location: string; }
let y = { name: 'Alice', location: 'Seattle' };
x = y; 
```

这里要检查`y`是否能赋值给`x`，编译器检查`x`中的每个属性，看是否能在`y`中也找到对应属性。 在这个例子中，`y`必须包含名字是`name`的`string`类型成员。`y`满足条件，因此赋值正确。

检查函数参数时使用相同的规则：

```js
function greet(n: Named) {
    alert('Hello, ' + n.name);
}
greet(y); // OK 
```

注意，`y`有个额外的`location`属性，但这不会引发错误。 只有目标类型（这里是`Named`）的成员会被一一检查是否兼容。

这个比较过程是递归进行的，检查每个成员及子成员。

# 比较两个函数

比较原始类型和对象类型时是容易理解的，问题是如何判断两个函数是兼容的。 让我们以两个函数开始，它们仅有参数列表不同：

```js
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // OK
x = y; // Error 
```

要查看`x`是否能赋值给`y`，首先看它们的参数列表。 `x`的每个参数必须能在`y`里找到对应类型的参数。 注意的是参数的名字相同与否无所谓，只看它们的类型。 这里，`x`的每个参数在`y`中都能找到对应的参数，所以允许赋值。

第二个赋值错误，因为`y`有个必需的第二个参数，但是`x`并没有，所以不允许赋值。

你可能会疑惑为什么允许`忽略`参数，像例子`y = x`中那样。 原因是忽略额外的参数在 JavaScript 里是很常见的。 例如，`Array#forEach`给回调函数传 3 个参数：数组元素，索引和整个数组。 尽管如此，传入一个只使用第一个参数的回调函数也是很有用的：

```js
let items = [1, 2, 3];

// Don't force these extra arguments
items.forEach((item, index, array) => console.log(item));

// Should be OK!
items.forEach((item) => console.log(item)); 
```

下面来看看如何处理返回值类型，创建两个仅是返回值类型不同的函数：

```js
let x = () => ({name: 'Alice'});
let y = () => ({name: 'Alice', location: 'Seattle'});

x = y; // OK
y = x; // Error because x() lacks a location property 
```

类型系统强制源函数的返回值类型必须是目标函数返回值类型的子类型。

## 函数参数双向协变

当比较函数参数类型时，只有当源函数参数能够赋值给目标函数或者反过来时才能赋值成功。 这是不稳定的，因为调用者可能传入了一个具有更精确类型信息的函数，但是调用这个传入的函数的时候却使用了不是那么精确的类型信息。 实际上，这极少会发生错误，并且能够实现很多 JavaScript 里的常见模式。例如：

```js
enum EventType { Mouse, Keyboard }

interface Event { timestamp: number; }
interface MouseEvent extends Event { x: number; y: number }
interface KeyEvent extends Event { keyCode: number }

function listenEvent(eventType: EventType, handler: (n: Event) => void) {
    /* ... */
}

// Unsound, but useful and common
listenEvent(EventType.Mouse, (e: MouseEvent) => console.log(e.x + ',' + e.y));

// Undesirable alternatives in presence of soundness
listenEvent(EventType.Mouse, (e: Event) => console.log((<MouseEvent>e).x + ',' + (<MouseEvent>e).y));
listenEvent(EventType.Mouse, <(e: Event) => void>((e: MouseEvent) => console.log(e.x + ',' + e.y)));

// Still disallowed (clear error). Type safety enforced for wholly incompatible types
listenEvent(EventType.Mouse, (e: number) => console.log(e)); 
```

## 可选参数及剩余参数

比较函数兼容性的时候，可选参数与必须参数是可交换的。 原类型上额外的可选参数并不会造成错误，目标类型的可选参数没有对应的参数也不是错误。

当一个函数有剩余参数时，它被当做无限个可选参数。

这对于类型系统来说是不稳定的，但从运行时的角度来看，可选参数一般来说是不强制的，因为对于大多数函数来说相当于传递了一些`undefinded`。

有一个好的例子，常见的函数接收一个回调函数并用对于程序员来说是可预知的参数但对类型系统来说是不确定的参数来调用：

```js
function invokeLater(args: any[], callback: (...args: any[]) => void) {
    /* ... Invoke callback with 'args' ... */
}

// Unsound - invokeLater "might" provide any number of arguments
invokeLater([1, 2], (x, y) => console.log(x + ', ' + y));

// Confusing (x and y are actually required) and undiscoverable
invokeLater([1, 2], (x?, y?) => console.log(x + ', ' + y)); 
```

## 函数重载

对于有重载的函数，源函数的每个重载都要在目标函数上找到对应的函数签名。 这确保了目标函数可以在所有源函数可调用的地方调用。 对于特殊的函数重载签名不会用来做兼容性检查。

# 枚举

枚举类型与数字类型兼容，并且数字类型与枚举类型兼容。不同枚举类型之间是不兼容的。比如，

```js
enum Status { Ready, Waiting };
enum Color { Red, Blue, Green };

let status = Status.Ready;
status = Color.Green;  //error 
```

# 类

类与对象字面量和接口差不多，但有一点不同：类有静态部分和实例部分的类型。 比较两个类类型的对象时，只有实例的成员会被比较。 静态成员和构造函数不在比较的范围内。

```js
class Animal {
    feet: number;
    constructor(name: string, numFeet: number) { }
}

class Size {
    feet: number;
    constructor(numFeet: number) { }
}

let a: Animal;
let s: Size;

a = s;  //OK
s = a;  //OK 
```

## 类的私有成员

私有成员会影响兼容性判断。 当类的实例用来检查兼容时，如果它包含一个私有成员，那么目标类型必须包含来自同一个类的这个私有成员。 这允许子类赋值给父类，但是不能赋值给其它有同样类型的类。

# 泛型

因为 TypeScript 是结构性的类型系统，类型参数只影响使用其做为类型一部分的结果类型。比如，

```js
interface Empty<T> {
}
let x: Empty<number>;
let y: Empty<string>;

x = y;  // okay, y matches structure of x 
```

上面代码里，`x`和`y`是兼容的，因为它们的结构使用类型参数时并没有什么不同。 把这个例子改变一下，增加一个成员，就能看出是如何工作的了：

```js
interface NotEmpty<T> {
    data: T;
}
let x: NotEmpty<number>;
let y: NotEmpty<string>;

x = y;  // error, x and y are not compatible 
```

在这里，泛型类型在使用时就好比不是一个泛型类型。

对于没指定泛型类型的泛型参数时，会把所有泛型参数当成`any`比较。 然后用结果类型进行比较，就像上面第一个例子。

比如，

```js
let identity = function<T>(x: T): T {
    // ...
}

let reverse = function<U>(y: U): U {
    // ...
}

identity = reverse;  // Okay because (x: any)=>any matches (y: any)=>any 
```

# 高级主题

## 子类型与赋值

目前为止，我们使用了`兼容性`，它在语言规范里没有定义。 在 TypeScript 里，有两种类型的兼容性：子类型与赋值。 它们的不同点在于，赋值扩展了子类型兼容，允许给`any`赋值或从`any`取值和允许数字赋值给枚举类型或枚举类型赋值给数字。

语言里的不同地方分别使用了它们之中的机制。 实际上，类型兼容性是由赋值兼容性来控制的甚至在`implements`和`extends`语句里。 更多信息，请参阅[TypeScript 语言规范](http://go.microsoft.com/fwlink/?LinkId=267121).

# 书写.d.ts 文件

# 介绍

当使用外部 JavaScript 库或新的宿主 API 时，你需要一个声明文件（.d.ts）定义程序库的 shape。 这个手册包含了写.d.ts 文件的高级概念，并带有一些例子，告诉你怎么去写一个声明文件。

# 指导与说明

## 流程

最好从程序库的文档而不是代码开始写.d.ts 文件。 这样保证不会被具体实现所干扰，而且相比于 JS 代码更易读。 下面的例子会假设你正在参照文档写声明文件。

## 命名空间

当定义接口（例如：“options”对象），你会选择是否将这些类型放进命名空间里。 这主要是靠主观判断 -- 如果使用的人主要是用这些类型来声明变量和参数，并且类型命名不会引起命名冲突，则放在全局命名空间里更好。 如果类型不是被直接使用，或者没法起一个唯一的名字的话，就使用命名空间来避免与其它类型发生冲突。

## 回调函数

许多 JavaScript 库接收一个函数做为参数，之后传入已知的参数来调用它。 当用这些类型为函数签名的时候，不要把这些参数标记成可选参数。 正确的思考方式是“(调用者)会提供什么样的参数？”，不是“(函数)会使用到什么样的参数？”。 TypeScript 0.9.7+不会强制这种可选参数的使用，参数可选的双向协变可以被外部的 linter 强制执行。

## 扩展与声明合并

写声明文件的时候，要记住 TypeScript 扩展现有对象的方式。 你可以选择用匿名类型或接口类型的方式声明一个变量：

#### 匿名类型 var

```js
declare let MyPoint: { x: number; y: number; }; 
```

#### 接口类型 var

```js
interface SomePoint { x: number; y: number; }
declare let MyPoint: SomePoint; 
```

从使用者角度来讲，它们是相同的，但是 SomePoint 类型能够通过接口合并来扩展：

```js
interface SomePoint { z: number; }
MyPoint.z = 4; // OK 
```

是否想让你的声明是可扩展的取决于主观判断。 通常来讲，尽量符合 library 的意图。

## 类的分解

TypeScript 的类会创建出两个类型：实例类型，定义了类型的实例具有哪些成员；构造函数类型，定义了类构造函数具有哪些类型。 构造函数类型也被称做类的静态部分类型，因为它包含了类的静态成员。

你可以使用`typeof`关键字来拿到类静态部分类型，在写声明文件时，想要把类明确的分解成实例类型和静态类型时是有用且必要的。

下面是一个例子，从使用者的角度来看，这两个声明是等同的：

#### 标准版

```js
class A {
    static st: string;
    inst: number;
    constructor(m: any) {}
} 
```

#### 分解版

```js
interface A_Static {
    new(m: any): A_Instance;
    st: string;
}
interface A_Instance {
    inst: number;
}
declare let A: A_Static; 
```

这里的利弊如下：

*   标准方式可以使用 extends 来继承；分解的类不能。也可能会在未来版本的 TypeScript 里做出改变：是否允许任意 extends 表达式
*   都允许之后为类添加静态成员(通过合并声明的方式)
*   分解的类允许增加实例成员，标准版不允许
*   使用分解类的时候，需要为多类型成员起合理的名字

## 命名规则

一般来讲，不要给接口加 I 前缀（比如：IColor）。 因为 TypeScript 的接口类型概念比 C#或 Java 里的意义更为广泛，IFoo 命名不利于这个特点。

# 例子

下面进行例子部分。对于每个例子，首先使用*应用示例*，然后是类型声明。 如果有多个好的声明表示方法，会列出多个。

## 参数对象

#### 应用示例

```js
animalFactory.create("dog");
animalFactory.create("giraffe", { name: "ronald" });
animalFactory.create("panda", { name: "bob", height: 400 });
// Invalid: name must be provided if options is given
animalFactory.create("cat", { height: 32 }); 
```

#### 类型声明

```js
namespace animalFactory {
    interface AnimalOptions {
        name: string;
        height?: number;
        weight?: number;
    }
    function create(name: string, animalOptions?: AnimalOptions): Animal;
} 
```

## 带属性的函数

#### 应用示例

```js
zooKeeper.workSchedule = "morning";
zooKeeper(giraffeCage); 
```

#### 类型声明

```js
// Note: Function must precede namespace
function zooKeeper(cage: AnimalCage);
namespace zooKeeper {
    let workSchedule: string;
} 
```

## 可以用 new 调用也可以直接调用的方法

#### 应用示例

```js
let w = widget(32, 16);
let y = new widget("sprocket");
// w and y are both widgets
w.sprock();
y.sprock(); 
```

#### 类型声明

```js
interface Widget {
    sprock(): void;
}

interface WidgetFactory {
    new(name: string): Widget;
    (width: number, height: number): Widget;
}

declare let widget: WidgetFactory; 
```

## 全局的或未知的外部 Libraries

#### 应用示例

```js
// Either
import x = require('zoo');
x.open();
// or
zoo.open(); 
```

#### 类型声明

```js
declare namespace zoo {
  function open(): void;
}

declare module "zoo" {
    export = zoo;
} 
```

## 外部模块的单个复杂对象

#### 应用示例

```js
// Super-chainable library for eagles
import Eagle = require('./eagle');

// Call directly
Eagle('bald').fly();

// Invoke with new
var eddie = new Eagle('Mille');

// Set properties
eddie.kind = 'golden'; 
```

#### 类型声明

```js
interface Eagle {
    (kind: string): Eagle;
    new (kind: string): Eagle;

    kind: string;
    fly(): void
}

declare var Eagle: Eagle;

export = Eagle; 
```

## Module as a Function

#### Usage

```js
// Common pattern for node modules (e.g. rimraf, debug, request, etc.)
import sayHello = require('say-hello');
sayHello('Travis'); 
```

#### Typing

```js
declare module 'say-hello' {
  function sayHello(name: string): void;
  export = sayHello;
} 
```

## 回调函数

#### 应用示例

```js
addLater(3, 4, x => console.log('x = ' + x)); 
```

#### 类型声明

```js
// Note: 'void' return type is preferred here
function addLater(x: number, y: number, (sum: number) => void): void; 
```

如果你想看其它模式的实现方式，请在[这里](https://github.com/Microsoft/TypeScript-Handbook/issues)留言！ 我们会尽可能地加到这里来。

# Iterators 和 Generators

# 可迭代性

当一个对象实现了`Symbol.iterator`属性时，我们认为它是可迭代的。 一些内置的类型如`Array`，`Map`，`Set`，`String`，`Int32Array`，`Uint32Array`等都已经实现了各自的`Symbol.iterator`。 对象上的`Symbol.iterator`函数负责返回供迭代的值。

## `for..of` 语句

`for..of`会遍历可迭代的对象，调用对象上的`Symbol.iterator`方法。 下面是在数组上使用`for..of`的简单例子：

```js
let someArray = [1, "string", false];

for (let entry of someArray) {
    console.log(entry); // 1, "string", false
} 
```

### `for..of` vs. `for..in` 语句

`for..of`和`for..in`均可迭代一个列表；但是用于迭代的值却不同，`for..in`迭代的是对象的 *键* 的列表，而`for..of`则迭代对象的键对应的值。

下面的例子展示了两者之间的区别：

```js
let list = [4, 5, 6];

for (let i in list) {
   console.log(i); // "0", "1", "2",
}

for (let i of list) {
   console.log(i); // "4", "5", "6" 
```

另一个区别是`for..in`可以操作任何对象；它提供了查看对象属性的一种方法。 但是`for..of`关注于迭代对象的值。内置对象`Map`和`Set`已经实现了`Symbol.iterator`方法，让我们可以访问它们保存的值。

```js
let pets = new Set(["Cat", "Dog", "Hamster"]);
pets["species"] = "mammals";

for (let pet in pets) {
   console.log(pet); // "species"
}

for (let pet of pets) {
    console.log(pet); // "Cat", "Dog", "Hamster"
} 
```

### 代码生成

#### 目标为 ES5 和 ES3

当生成目标为 ES5 或 ES3，迭代器只允许在`Array`类型上使用。 在非数组值上使用`for..of`语句会得到一个错误，就算这些非数组值已经实现了`Symbol.iterator`属性。

编译器会生成一个简单的`for`循环做为`for..of`循环，比如：

```js
let numbers = [1, 2, 3];
for (let num of numbers) {
    console.log(num);
} 
```

生成的代码为：

```js
var numbers = [1, 2, 3];
for (var _i = 0; _i < numbers.length; _i++) {
    var num = numbers[_i];
    console.log(num);
} 
```

#### 目标为 ECMAScript 2015 或更高

当目标为兼容 ECMAScipt 2015 的引擎时，编译器会生成相应引擎的`for..of`内置迭代器实现方式。