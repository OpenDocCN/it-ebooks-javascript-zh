# 接口

# 介绍

TypeScript 的核心原则之一是对值所具有的*shape*进行类型检查。 它有时被称做“鸭式辨型法”或“结构性子类型化”。 在 TypeScript 里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。

# 接口初探

下面通过一个简单示例来观察接口是如何工作的：

```js
function printLabel(labelledObj: { label: string }) {
  console.log(labelledObj.label);
}

let myObj = { size: 10, label: "Size 10 Object" };
printLabel(myObj); 
```

类型检查器会查看`printLabel`的调用。 `printLabel`有一个参数，并要求这个对象参数有一个名为`label`类型为`string`的属性。 需要注意的是，我们传入的对象参数实际上会包含很多属性，但是编译器只会检查那些必需的属性是否存在，并且其类型是否匹配。 然而，有些时候 TypeScript 却并不会这么宽松，我们下在会稍做讲解。

下面我们重写上面的例子，这次使用接口来描述：必须包含一个`label`属性且类型为`string`：

```js
interface LabelledValue {
  label: string;
}

function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj); 
```

`LabelledValue`接口就好比一个名字，用来描述上面例子里的要求。 它代表了有一个`label`属性且类型为`string`的对象。 需要注意的是，我们在这里并不能像在其它语言里一样，说传给`printLabel`的对象实现了这个接口。我们只会去关注值的外形。 只要传入的对象满足上面提到的必要条件，那么它就是被允许的。

还有一点值得提的是，类型检查器不会去检查属性的顺序，只要相应的属性存在并且类型也是对的就可以。

# 可选属性

接口里的属性不全都是必需的。 有些是只在某些条件下存在，或者根本不存在。 可选属性在应用“option bags”模式时很常用，即给函数传入的参数对象中只有部分属性赋值了。

下面是应用了“option bags”的例子：

```js
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  let newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.color;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

let mySquare = createSquare({color: "black"}); 
```

带有可选属性的接口与普通的接口定义差不多，只是在可选属性名字定义的后面加一个`?`符号。

可选属性的好处之一是可以对可能存在的属性进行预定义，好处之二是可以捕获引用了不存在的属性时的错误。 比如，我们故意将`createSquare`里的`color`属性名拼错，就会得到一个错误提示：

```js
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  let newSquare = {color: "white", area: 100};
  if (config.color) {
    // Error: Property 'collor' does not exist on type 'SquareConfig'
    newSquare.color = config.collor;  // Type-checker can catch the mistyped name here
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

let mySquare = createSquare({color: "black"}); 
```

# 额外的属性检查

我们在第一个例子里使用了接口，TypeScript 让我们传入`{ size: number; label: string; }`到仅期望得到`{ label: string; }`的函数里。 我们已经学过了可选属性，并且知道他们在“option bags”模式里很有用。

然而，天真地将这两者结合的话就会像在 JavaScript 里那样搬起石头砸自己的脚。 比如，拿`createSquare`例子来说：

```js
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}

let mySquare = createSquare({ colour: "red", width: 100 }); 
```

注意传入`createSquare`的参数拼写为*`colour`*而不是`color`。 在 JavaScript 里，这会默默地失败。

你可以会争辩这个程序已经正确的类型化了，因为`width`属性是兼容的，不存在`color`属性，而且额外的`colour`属性是无意义的。

然而，TypeScript 会认为这段代码可能存在 bug。 对象字面量会被特殊对待而且会经过*额外属性检查*，当将它们赋值给变量或作为参数传递的时候。 如果一个对象字面量存在任何“目标类型”不包含的属性时，你会得到一个错误。

```js
// error: 'colour' not expected in type 'SquareConfig'
let mySquare = createSquare({ colour: "red", width: 100 }); 
```

绕开这些检查非常简单。 最好而简便的方法是使用类型断言：

```js
let mySquare = createSquare({ colour: "red", width: 100 } as SquareConfig); 
```

另一个方法，可能会让人有点惊压，就是将一个对象赋值给别一个变量：

```js
let squareOptions = { colour: "red", width: 100 };
let mySquare = createSquare(squareOptions); 
```

因为`squareOptions`不会经过额外属性检查，所以编译器不会报错。

要留意，在像上面一样的简单代码里，你可能不应该去绕开这些检查。 对于包含方法和内部状态的复杂对象字面量来讲，你可能需要使用这些技巧，但是大部额外属性检查错误是真正的 bug。 就是说你遇到了额外类型检查出的错误，比如选择包，你应该去审查一下你的类型声明。 在这里，如果支持传入`color`或`colour`属性到`createSquare`，你应该修改`SquareConfig`定义来体现出这一点。

# 函数类型

接口能够描述 JavaScript 中对象拥有的各种各样的外形。 除了描述带有属性的普通对象外，接口也可以描述函数类型。

为了使用接口表示函数类型，我们需要给接口定义一个调用签名。 它就像是一个只有参数列表和返回值类型的函数定义。参数列表里的每个参数都需要名字和类型。

```js
interface SearchFunc {
  (source: string, subString: string): boolean;
} 
```

这样定义后，我们可以像使用其它接口一样使用这个函数类型的接口。 下例展示了如何创建一个函数类型的变量，并将一个同类型的函数赋值给这个变量。

```js
let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  let result = source.search(subString);
  if (result == -1) {
    return false;
  }
  else {
    return true;
  }
} 
```

对于函数类型的类型检查来说，函数的参数名不需要与接口里定义的名字相匹配。 比如，我们使用下面的代码重写上面的例子：

```js
let mySearch: SearchFunc;
mySearch = function(src: string, sub: string): boolean {
  let result = src.search(sub);
  if (result == -1) {
    return false;
  }
  else {
    return true;
  }
} 
```

函数的参数会逐个进行检查，要求对应位置上的参数类型是兼容的。 如果你不想指定类型，Typescript 的类型系统会推断出参数类型，因为函数直接赋值给了`SearchFunc`类型变量。 函数的返回值类型是通过其返回值推断出来的（此例是`false`和`true`）。 如果让这个函数返回数字或字符串，类型检查器会警告我们函数的返回值类型与`SearchFunc`接口中的定义不匹配。

```js
let mySearch: SearchFunc;
mySearch = function(src, sub) {
    let result = src.search(sub);
    if (result == -1) {
        return false;
    }
    else {
        return true;
    }
} 
```

# 数组类型

与使用接口描述函数类型差不多，我们也可以描述数组类型。 数组类型具有一个`index`类型表示索引的类型，还有一个相应的返回值类型表示通过索引得到的元素的类型。

```js
interface StringArray {
  [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"]; 
```

支持两种索引类型：string 和 number。 数组可以同时使用这两种索引类型，但是有一个限制，数字索引返回值的类型必须是字符串索引返回值的类型的子类型。

索引签名能够很好的描述数组和`dictionary`模式，它们也要求所有属性要与返回值类型相匹配。 因为字符串索引表明`obj.property`和`obj["property"]`两种形式都可以。 下面的例子里，`name`的类型与字符串索引类型不匹配，所以类型检查器给出一个错误提示：

```js
interface NumberDictionary {
  [index: string]: number;
  length: number;    // 可以，length 是 number 类型
  name: string       // 错误，`name`的类型不是索引类型的子类型
} 
```

# 类类型

## 实现接口

与 C#或 Java 里接口的基本作用一样，TypeScript 也能够用它来明确的强制一个类去符合某种契约。

```js
interface ClockInterface {
    currentTime: Date;
}

class Clock implements ClockInterface {
    currentTime: Date;
    constructor(h: number, m: number) { }
} 
```

你也可以在接口中描述一个方法，在类里实现它，如同下面的`setTime`方法一样：

```js
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
} 
```

接口描述了类的公共部分，而不是公共和私有两部分。 它不会帮你检查类是否具有某些私有成员。

## 类静态部分与实例部分的区别

当你操作类和接口的时候，你要知道类是具有两个类型的：静态部分的类型和实例的类型。 你会注意到，当你用构造器签名去定义一个接口并试图定义一个类去实现这个接口时会得到一个错误：

```js
interface ClockConstructor {
    new (hour: number, minute: number);
}

class Clock implements ClockConstructor {
    currentTime: Date;
    constructor(h: number, m: number) { }
} 
```

这里因为当一个类实现了一个接口时，只对其实例部分进行类型检查。 constructor 存在于类的静态部分，所以不在检查的范围内。

因此，我们应该直接操作类的静态部分。 看下面的例子，我们定义了两个接口，`ClockConstructor`为构造函数所用和`ClockInterface`为实例方法所用。 为了方便我们定义一个构造函数`createClock`，它用传入的类型创建实例。

```js
interface ClockConstructor {
    new (hour: number, minute: number): ClockInterface;
}
interface ClockInterface {
    tick();
}

function createClock(ctor: ClockConstructor, hour: number, minute: number): ClockInterface {
    return new ctor(hour, minute);
}

class DigitalClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("beep beep");
    }
}
class AnalogClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("tick tock");
    }
}

let digital = createClock(DigitalClock, 12, 17);
let analog = createClock(AnalogClock, 7, 32); 
```

因为`createClock`的第一个参数是`ClockConstructor`类型，在`createClock(AnalogClock, 12, 17)`里，会检查`AnalogClock`是否符合构造函数签名。

# 扩展接口

和类一样，接口也可以相互扩展。 这让我们能够从一个接口里复制成员到另一个接口里，可以更灵活地将接口分割到可重用的模块里。

```js
interface Shape {
    color: string;
}

interface Square extends Shape {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10; 
```

一个接口可以继承多个接口，创建出多个接口的合成接口。

```js
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0; 
```

# 混合类型

先前我们提过，接口能够描述 JavaScript 里丰富的类型。 因为 JavaScript 其动态灵活的特点，有时你会希望一个对象可以同时具有上面提到的多种类型。

一个例子就是，一个对象可以同时做为函数和对象使用，并带有额外的属性。

```js
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = <Counter>function (start: number) { };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0; 
```

在使用 JavaScript 第三方库的时候，你可能需要像上面那样去完整地定义类型。

# 接口继承类

当接口继承了一个类类型时，它会继承类的成员但不包括其实现。 就好像接口声明了所有类中存在的成员，但并没有提供具体实现一样。 接口同样会继承到类的 private 和 protected 成员。 这意味着当你创建了一个接口继承了一个拥有私有或受保护的成员的类时，这个接口类型只能被这个类或其子类所实现（implement）。

这是很有用的，当你有一个很深层次的继承，但是只想你的代码只是针对拥有特定属性的子类起作用的时候。子类除了继承自基类外与基类没有任何联系。 例：

```js
class Control {
    private state: any;
}

interface SelectableControl extends Control {
    select(): void;
}

class Button extends Control {
    select() { }
}
class TextBox extends Control {
    select() { }
}
class Image extends Control {
}
class Location {
    select() { }
} 
```

在上面的例子里，`SelectableControl`包含了`Control`的所有成员，包括私有成员`state`。 因为`state`是私有成员，所以只能够是`Control`的子类们才能实现`SelectableControl`接口。 因为只有`Control`的子类才能够拥有一个声明于`Control`的私有成员`state`，这对私有成员的兼容性是必需的。

在`Control`类内部，是允许通过`SelectableControl`的实例来访问私有成员`state`的。 实际上，`SelectableControl`就像`Control`一样，并拥有一个`select`方法。 `Button`和`TextBox`类是`SelectableControl`的子类（因为它们都继承自`Control`并有`select`方法），但`Image`和`Location`类并不是这样的。

# 高级类型

# 联合类型

偶尔你会遇到这种情况，一个第三方库可以传入或返回`number`或`string`类型的数据。 例如下面的函数：

```js
/**
 * 拿到一个字符串并在左边添加"padding"。
 * 如果 'padding' 是个字符串, 那么 'padding' 被加到左边。
 * 如果 'padding' 是个数字, 那么在左边加入此数量的空格.
 */
function padLeft(value: string, padding: any) {
    // ...
}

padLeft("Hello world", 4); // 返回 "    Hello world" 
```

`padLeft`存在一个问题，`padding`参数的类型指定成了`any`。 这就是说我们可以传入一个既不是`number`也不是`string`类型的参数，但是 TypeScript 却不报错。

```js
let indentedString = padLeft("Hello world", true); // 编译阶段通过。运行时报错 
```

在一些传统的面向对象语言里，我们可能将这两种类型抽象成有层级的类型。

```js
interface Padder {
    getPaddingString(): string
}

class SpaceRepeatingPadder implements Padder {
    constructor(private numSpaces: number) { }
    getPaddingString() {
        return Array(this.numSpaces).join(" ");
    }
}

class StringPadder implements Padder {
    constructor(private value: string) { }
    getPaddingString() {
        return this.value;
    }
}

function padLeft(value: string, padder: Padder) {
    return padder.getPaddingString() + value;
}

padLeft("Hello world", new SpaceRepeatingPadder(4)); 
```

这样就清楚多了，但是做的也有点过。 原始版本的`padLeft`有一点好处是我们可以传入一个原始值。 使用起来清晰明了。 如果我们想定义一个已经存在了的函数那么新的方案也不合适了。

不用`any`，我们可以使用*联合类型*做为`padding`的参数：

```js
/**
 * Takes a string and adds "padding" to the left.
 * If 'padding' is a string, then 'padding' is appended to the left side.
 * If 'padding' is a number, then that number of spaces is added to the left side.
 */
function padLeft(value: string, padding: string | number) {
    // ...
}

let indentedString = padLeft("Hello world", true); // errors during compilation 
```

联合类型表示一个值可以是几种类型之一。 我们用竖线（`|`）分隔每个类型，所以`number | string | boolean`表示一个值可以是`number`，`string`，或`boolean`。

如果一个值是联合类型，我们只能访问此联合类型的所有类型里共有的成员。

```js
interface Bird {
    fly();
    layEggs();
}

interface Fish {
    swim();
    layEggs();
}

function getSmallPet(): Fish | Bird {
    // ...
}

let pet = getSmallPet();
pet.layEggs(); // okay
pet.swim();    // errors 
```

这里的联合类型可能有点复杂，但是你很容易就习惯了。 如果一个值类型是`A | B`，我们只能*确定*它具有成员同时存在于`A`*和*`B`里。 这个例子里，`Bird`具有一个`fly`成员。 我们不能确定一个`Bird | Fish`类型的变量是否有`fly`方法。 如果变量在运行时是`Fish`类型，那么调用`pet.fly()`就出错了。

# 类型保护与区分类型

联合类型非常适合这样的情形，可接收的值有不同的类型。 当我们想明确地知道是否拿到`Fish`时会怎么做？ JavaScript 里常用来区分 2 个可能值的方法是检查它们是否存在。 像之前提到的，我们只能访问联合类型的所有类型中共有的成员。

```js
let pet = getSmallPet();

// 每一个成员访问都会报错
if (pet.swim) {
    pet.swim();
}
else if (pet.fly) {
    pet.fly();
} 
```

为了让这码代码工作，我们要使用类型断言：

```js
let pet = getSmallPet();

if ((<Fish>pet).swim) {
    (<Fish>pet).swim();
}
else {
    (<Bird>pet).fly();
} 
```

## 用户自定义的类型保护

可以注意到我们使用了多次类型断言。 如果我们只要检查过一次类型，就能够在后面的每个分支里清楚`pet`的类型的话就好了。

TypeScript 里的*类型保护*机制让它成为了现实。 类型保护就是一些表达式，它们会在运行时检查以确保在某个作用域里的类型。 要定义一个类型保护，我们只要简单地定义一个函数，它的返回值是一个*类型断言*：

```js
function isFish(pet: Fish | Bird): pet is Fish {
    return (<Fish>pet).swim !== undefined;
} 
```

在这个例子里，`pet is Fish`就是类型断言。 一个断言是`parameterName is Type`这种形式，`parameterName`必须是来自于当前函数签名里的一个参数名。

每当使用一些变量调用`isFish`时，TypeScript 会将变量缩减为那个具体的类型，只要这个类型与变量的原始类型是兼容的。

```js
// 'swim' 和 'fly' 调用都没有问题了

if (isFish(pet) {
    pet.swim();
}
else {
    pet.fly();
} 
```

注意 TypeScript 不仅知道在`if`分支里`pet`是`Fish`类型； 它还清楚在`else`分支里，一定*不是*`Fish`类型，一定是`Bird`类型。

## `typeof`类型保护

我们还没有真正的展示过如何使用联合类型来实现`padLeft`。 我们可以像下面这样利用类型断言来写：

```js
function isNumber(x: any): x is number {
    return typeof x === "number";
}

function isString(x: any): x is string {
    return typeof x === "string";
}

function padLeft(value: string, padding: string | number) {
    if (isNumber(padding)) {
        return Array(padding).join(" ") + value;
    }
    if (isString(padding)) {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${value}'.`);
} 
```

然而，必须要定义一个函数来判断类型是否是原始类型，这太痛苦了。 幸运的是，现在我们不必将`typeof x === "number"`抽象成一个函数，因为 TypeScript 可以将它识别为一个类型保护。 也就是说我们可以直接在代码里检查类型了。

```js
function padLeft(value: string, padding: string | number) {
    if (typeof padding === "number") {
        return Array(padding).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${value}'.`);
} 
```

这些*`typeof`类型保护*只有 2 个形式能被识别：`typeof v === "typename"`和`typeof v !== "typename"`，`"typename"`必须是`"number"`，`"string"`，`"boolean"`或`"symbol"`。 但是 TypeScript 并不会阻止你使用除这些以外的字符串，或者将它们位置对换，且语言不会把它们识别为类型保护。

## `instanceof`类型保护

如果你已经阅读了`typeof`类型保护并且对 JavaScript 里的`instanceof`操作符熟悉的话，你可能已经猜到了这节要讲的内容。

*`instanceof`类型保护*是通过其构造函数来细化其类型。 比如，我们借鉴一下之前字符串填充的例子：

```js
interface Padder {
    getPaddingString(): string
}

class SpaceRepeatingPadder implements Padder {
    constructor(private numSpaces: number) { }
    getPaddingString() {
        return Array(this.numSpaces).join(" ");
    }
}

class StringPadder implements Padder {
    constructor(private value: string) { }
    getPaddingString() {
        return this.value;
    }
}

function getRandomPadder() {
    return Math.random() < 0.5 ?
        new SpaceRepeatingPadder(4) :
        new StringPadder("  ");
}

// 类型为 SpaceRepeatingPadder | StringPadder
let padder: Padding = getRandomPadder();

if (padder instanceof SpaceRepeatingPadder) {
    padder; // 类型细化为'SpaceRepeatingPadder'
}
if (padder instanceof StringPadder) {
    padder; // 类型细化为'StringPadder'
} 
```

`instanceof`的右侧要求为一个构造函数，TypeScript 将细化为：

1.  这个函数的`prototype`属性，如果它的类型不为`any`的话
2.  类型中构造签名所返回的类型的联合，顺序保持一至。

# 类型别名

类型别名会给一个类型起个新名字。 类型别名有时和接口很像，但是可以作用于原始值，联合类型，元组以及其它任何你需要手写的类型。

```js
type XCoord = number;
type YCoord = number;

type XYCoord = { x: XCoord; y: YCoord };
type XYZCoord = { x: XCoord; y: YCoord; z: number };

type Coordinate = XCoord | XYCoord | XYZCoord;
type CoordList = Coordinate[];

let coord: CoordList = [{ x: 10, y: 10}, { x: 0, y: 42, z: 10 }, 5]; 
```

起别名不会新建一个类型 - 它创建了一个新*名字*来引用那个类型。 所以`10`是绝对有效的`XCoord`和`YCoord`，因为它们都引用`number`。 给原始类型起别名通常没什么用，尽管可以做为文档的一种形式使用。

同接口一样，类型别名也可以是泛型 - 我们可以添加类型参数并且在别名声明的右侧传入：

```js
type Container<T> = { value: T }; 
```

我们也可以使用类型别名来在属性里引用自己：

```js
type Tree<T> = {
    value: T;
    left: Tree<T>;
    right: Tree<T>;
} 
```

然而，类型别名不可能出现在声明右侧以外的地方：

```js
type Yikes = Array<Yikes>; // 错误 
```

## 接口 vs. 类型别名

像我们提到的，类型别名可以像接口一样；然而，仍有一些细微差别。

一个重要区别是类型别名不能被`extends`和`implements`也不能去`extends`和`implements`其它类型。 因为[软件中的对象应该对于扩展是开放的，但是对于修改是封闭的](https://en.wikipedia.org/wiki/Open/closed_principle)，你应该尽量去使用接口代替类型别名。

另一方面，如果你无法通过接口来描述一个类型并且需要使用联合类型或元组类型，这时通常会使用类型别名。