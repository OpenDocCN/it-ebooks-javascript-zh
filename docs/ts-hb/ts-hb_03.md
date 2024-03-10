# 类

# 介绍

传统的 JavaScript 程序使用函数和基于原型的继承来创建可重用的组件，但这对于熟悉使用面向对象方式的程序员来说有些棘手，因为他们用的是基于类的继承并且对象是从类构建出来的。 从 ECMAScript 2015，也就是 ECMAScript 6，JavaScript 程序将可以使用这种基于类的面向对象方法。 在 TypeScript 里，我们允许开发者现在就使用这些特性，并且编译后的 JavaScript 可以在所有主流浏览器和平台上运行，而不需要等到下个 JavaScript 版本。

# 类

下面看一个使用类的例子：

```js
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter = new Greeter("world"); 
```

如果你使用过 C#或 Java，你会对这种语法非常熟悉。 我们声明一个`Greeter`类。这个类有 3 个成员：一个叫做`greeting`的属性，一个构造函数和一个`greet`方法。

你会注意到，我们在引用任何一个类成员的时候都用了`this`。 它表示我们访问的是类的成员。

最后一行，我们使用`new`构造了 Greeter 类的一个实例。 它会调用之前定义的构造函数，创建一个`Greeter`类型的新对象，并执行构造函数初始化它。

# 继承

在 TypeScript 里，我们可以使用常用的面向对象模式。 当然，基于类的程序设计中最基本的模式是允许使用继承来扩展一个类。

看下面的例子：

```js
class Animal {
    name:string;
    constructor(theName: string) { this.name = theName; }
    move(distanceInMeters: number = 0) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}

class Snake extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 5) {
        console.log("Slithering...");
        super.move(distanceInMeters);
    }
}

class Horse extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 45) {
        console.log("Galloping...");
        super.move(distanceInMeters);
    }
}

let sam = new Snake("Sammy the Python");
let tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34); 
```

这个例子展示了 TypeScript 中继承的一些特征，与其它语言类似。 我们使用`extends`来创建子类。你可以看到`Horse`和`Snake`类是基类`Animal`的子类，并且可以访问其属性和方法。

这个例子演示了如何在子类里可以重写父类的方法。 `Snake`类和`Horse`类都创建了`move`方法，重写了从`Animal`继承来的`move`方法，使得`move`方法根据不同的类而具有不同的功能。 注意，即使`tom`被声明为`Animal`类型，因为它的值是`Horse`，`tom.move(34)`调用`Horse`里的重写方法：

包含 constructor 函数的派生类必须调用`super()`，它会执行基类的构造方法。

```js
Slithering...
Sammy the Python moved 5m.
Galloping...
Tommy the Palomino moved 34m. 
```

# 公共，私有与受保护的修饰符

## 默认为公有

在上面的例子里，我们可以自由的访问程序里定义的成员。 如果你对其它语言中的类比较了解，就会注意到我们在之前的代码里并没有使用`public`来做修饰；例如，C#要求必须明确地使用`public`指定成员是可见的。 在 TypeScript 里，每个成员默认为`public`的。

你也可以明确的将一个成员标记成`public`。 我们可以用下面的方式来重写上面的`Animal`类：

```js
class Animal {
    public name: string;
    public constructor(theName: string) { this.name = theName; }
    move(distanceInMeters: number) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
} 
```

## 理解`private`

当成员被标记成`private`时，它就不能在声明它的类的外部访问。比如：

```js
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

new Animal("Cat").name; // Error: 'name' is private; 
```

TypeScript 使用的是结构性类型系统。 当我们比较两种不同的类型时，并不在乎它们从哪儿来的，如果所有成员的类型都是兼容的，我们就认为它们的类型是兼容的。

然而，当我们比较带有`private`或`protected`成员的类型的时候，情况就不同了。 如果其中一个类型里包含一个`private`成员，那么只有当另外一个类型中也存在这样一个`private`成员， 并且它们是来自同一处声明时，我们才认为这两个类型是兼容的。 对于`protected`成员也使用这个规则。

下面来看一个例子，详细的解释了这点：

```js
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

class Rhino extends Animal {
    constructor() { super("Rhino"); }
}

class Employee {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

let animal = new Animal("Goat");
let rhino = new Rhino();
let employee = new Employee("Bob");

animal = rhino;
animal = employee; // Error: Animal and Employee are not compatible 
```

这个例子中有`Animal`和`Rhino`两个类，`Rhino`是`Animal`类的子类。 还有一个`Employee`类，其类型看上去与`Animal`是相同的。 我们创建了几个这些类的实例，并相互赋值来看看会发生什么。 因为`Animal`和`Rhino`共享了来自`Animal`里的私有成员定义`private name: string`，因此它们是兼容的。 然而`Employee`却不是这样。当把`Employee`赋值给`Animal`的时候，得到一个错误，说它们的类型不兼容。 尽管`Employee`里也有一个私有成员`name`，但它明显不是`Animal`里面定义的那个。

## 理解`protected`

`protected`修饰符与`private`修饰符的行为很相似，但有一点不同，`protected`成员在派生类中仍然可以访问。例如：

```js
class Person {
    protected name: string;
    constructor(name: string) { this.name = name; }
}

class Employee extends Person {
    private department: string;

    constructor(name: string, department: string) {
        super(name)
        this.department = department;
    }

    public getElevatorPitch() {
        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
    }
}

let howard = new Employee("Howard", "Sales");
console.log(howard.getElevatorPitch());
console.log(howard.name); // error 
```

注意，我们不能在`Person`类外使用`name`，但是我们仍然可以通过`Employee`类的实例方法访问，因为`Employee`是由`Person`派生出来的。

构造函数也可被标记为`protected`. 这就是说这个类不能在包含它的类之外实例外，但是可以被继承。比如，

```js
class Person {
    protected name: string;
    protected constructor(theName: string) { this.name = theName; }
}

// Employee can extend Person
class Employee extends Person {
    private department: string;

    constructor(name: string, department: string) {
        super(name);
        this.department = department;
    }

    public getElevatorPitch() {
        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
    }
}

let howard = new Employee("Howard", "Sales");
let john = new Person("John"); // Error: The 'Person' constructor is protected 
```

## 参数属性

在上面的例子中，我们不得不定义一个受保护的成员`name`和一个构造函数参数`theName`在`Person`类里，并且立刻给`name`和`theName`赋值。 这种情况经常会遇到。*参数属性*可以方便地让我们在一个地方定义并初始化一个成员。 下面的例子是对之前`Animal`类的修改版，使用了参数属性：

```js
class Animal {
    constructor(private name: string) { }
    move(distanceInMeters: number) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
} 
```

注意看我们是如何舍弃了`theName`，仅在构造函数里使用`private name: string`参数来创建和初始化`name`成员。 我们把声明和赋值合并至一处。

参数属性通过给构造函数参数添加一个访问限定符来声明。 使用`private`限定一个参数属性会声明并初始化一个私有成员；对于`public`和`protected`来说也是一样。

# 存取器

TypeScript 支持 getters/setters 来截取对对象成员的访问。 它能帮助你有效的控制对对象成员的访问。

下面来看如何把一类改写成使用`get`和`set`。 首先是一个没用使用存取器的例子。

```js
class Employee {
    fullName: string;
}

let employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    console.log(employee.fullName);
} 
```

我们可以随意的设置`fullName`，这是非常方便的，但是这也可能会带来麻烦。

下面这个版本里，我们先检查用户密码是否正确，然后再允许其修改 employee。 我们把对`fullName`的直接访问改成了可以检查密码的`set`方法。 我们也加了一个`get`方法，让上面的例子仍然可以工作。

```js
let passcode = "secret passcode";

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }

    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            console.log("Error: Unauthorized update of employee!");
        }
    }
}

let employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
} 
```

我们可以修改一下密码，来验证一下存取器是否是工作的。当密码不对时，会提示我们没有权限去修改 employee。

注意：若要使用存取器，要求设置编译器输出目标为 ECMAScript 5 或更高。

# 静态属性

到目前为止，我们只讨论了类的实例成员，那些仅当类被实例化的时候才会被初始化的属性。 我们也可以创建类的静态成员，这些属性存在于类本身上面而不是类的实例上。 在这个例子里，我们使用`static`定义`origin`，因为它是所有网格都会用到的属性。 每个实例想要访问这个属性的时候，都要在 origin 前面加上类名。 如同在实例属性上使用`this.`前缀来访问属性一样，这里我们使用`Grid.`来访问静态属性。

```js
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        let xDist = (point.x - Grid.origin.x);
        let yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

let grid1 = new Grid(1.0);  // 1x scale
let grid2 = new Grid(5.0);  // 5x scale

console.log(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
console.log(grid2.calculateDistanceFromOrigin({x: 10, y: 10})); 
```

# 抽象类

抽象类是供其它类继承的基类。 他们一般不会直接被实例化。 不同于接口，抽象类可以包含成员的实现细节。 `abstract`关键字是用于定义抽象类和在抽象类内部定义抽象方法。

```js
abstract class Animal {
    abstract makeSound(): void;
    move(): void {
        console.log('roaming the earch...');
    }
} 
```

抽象类中的抽象方法不包含具体实现并且必须在派生类中实现。 抽象方法的语法与接口方法相似。 两者都是定义方法签名不包含方法体。 然而，抽象方法必须使用`abstract`关键字并且可以包含访问符。

```js
abstract class Department {

    constructor(public name: string) {
    }

    printName(): void {
        console.log('Department name: ' + this.name);
    }

    abstract printMeeting(): void; // 必须在派生类中实现
}

class AccountingDepartment extends Department {

    constructor() {
        super('Accounting and Auditing'); // constructors in derived classes must call super()
    }

    printMeeting(): void {
        console.log('The Accounting Department meets each Monday at 10am.');
    }

    generateReports(): void {
        console.log('Generating accounting reports...');
    }
}

let department: Department; // ok to create a reference to an abstract type
department = new Department(); // error: cannot create an instance of an abstract class
department = new AccountingDepartment(); // ok to create and assign a non-abstract subclass
department.printName();
department.printMeeting();
department.generateReports(); // error: method doesn't exist on declared abstract type 
```

# 高级技巧

## 构造函数

当你在 TypeScript 里定义类的时候，实际上同时定义了很多东西。 首先是类的*实例*的类型。

```js
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter: Greeter;
greeter = new Greeter("world");
console.log(greeter.greet()); 
```

在这里，我们写了`let greeter: Greeter`，意思是`Greeter`类实例的类型是`Greeter`。 这对于用过其它面向对象语言的程序员来讲已经是老习惯了。

我们也创建了一个叫做*构造函数*的值。 这个函数会在我们使用`new`创建类实例的时候被调用。 下面我们来看看，上面的代码被编译成 JavaScript 后是什么样子的：

```js
let Greeter = (function () {
    function Greeter(message) {
        this.greeting = message;
    }
    Greeter.prototype.greet = function () {
        return "Hello, " + this.greeting;
    };
    return Greeter;
})();

let greeter;
greeter = new Greeter("world");
console.log(greeter.greet()); 
```

上面的代码里，`let Greeter`将被赋值为构造函数。 当我们使用`new`并执行这个函数后，便会得到一个类的实例。 这个构造函数也包含了类的所有静态属性。 换个角度说，我们可以认为类具有实例部分与静态部分这两个部分。

让我们来改写一下这个例子，看看它们之前的区别：

```js
class Greeter {
    static standardGreeting = "Hello, there";
    greeting: string;
    greet() {
        if (this.greeting) {
            return "Hello, " + this.greeting;
        }
        else {
            return Greeter.standardGreeting;
        }
    }
}

let greeter1: Greeter;
greeter1 = new Greeter();
console.log(greeter1.greet());

let greeterMaker: typeof Greeter = Greeter;
greeterMaker.standardGreeting = "Hey there!";
let greeter2:Greeter = new greeterMaker();
console.log(greeter2.greet()); 
```

这个例子里，`greeter1`与之前看到的一样。 我们实例化`Greeter`类，并使用这个对象。 与我们之前看到的一样。

再之后，我们直接使用类。 我们创建了一个叫做`greeterMaker`的变量。 这个变量保存了这个类或者说保存了类构造函数。 然后我们使用`typeof Greeter`，意思是取 Greeter 类的类型，而不是实例的类型。 或者更确切的说，"告诉我`Greeter`标识符的类型"，也就是构造函数的类型。 这个类型包含了类的所有静态成员和构造函数。 之后，就和前面一样，我们在`greeterMaker`上使用`new`，创建`Greeter`的实例。

## 把类当做接口使用

如上一节里所讲的，类定义会创建两个东西：类实例的类型和一个构造函数。 因为类可以创建出类型，所以你能够在可以使用接口的地方使用类。

```js
class Point {
    x: number;
    y: number;
}

interface Point3d extends Point {
    z: number;
}

let point3d: Point3d = {x: 1, y: 2, z: 3}; 
```

# 命名空间和模块

> **关于术语的一点说明:** 请务必注意一点，TypeScript 1.5 里术语名已经发生了变化。 “内部模块”现在称做“命名空间”。 “外部模块”现在则简称为“模块”，这是为了与[ECMAScript 2015](http://www.ecma-international.org/ecma-262/6.0/)里的术语保持一致，(也就是说 `module X {` 相当于现在推荐的写法 `namespace X {`)。

# 介绍

这篇文章将概括介绍在 TypeScript 里使用模块与命名空间来组织代码的方法。 我们也会谈及命名空间和模块的高级使用场景，和在使用它们的过程中常见的陷井。

查看模块章节了解关于模块的更多信息。 查看命名空间章节了解关于命名空间的更多信息。

# 使用命名空间

命名空间是位于全局命名空间下的一个普通的带有名字的 JavaScript 对象。 这令命名空间十分容易使用。 它们可以在多文件中同时使用，并通过`--outFile`结合在一起。 命名空间是帮你组织 Web 应用不错的方式，你可以把所有依赖都放在 HTML 页面的`<script>`标签里。

但就像其它的全局命名空间污染一样，它很难去了解组件之间的依赖关系，尤其是在大型的应用中。

# 使用模块

像命名空间一样，模块可以包含代码和声明。 不同的是模块可以*声明*它的依赖。

模块会把依赖添加到模块加载器上（例如 CommonJs / Require.js）。 对于小型的 JS 应用来说可能没必要，但是对于大型应用，这一点点的花费会带来长久的模块化和可维护性上的便利。 模块也提供了更好的代码重用，更强的封闭性以及更好的使用工具进行优化。

对于 Node.js 应用来说，模块是默认并推荐的组织代码的方式。

从 ECMAScript 2015 开始，模块成为了语言内置的部分，应该会被所有正常的解释引擎所支持。 因此，对于新项目来说推荐使用模块做为组织代码的方式。

# 命名空间和模块的陷井

这部分我们会描述常见的命名空间和模块的使用陷井和如何去避免它们。

## 对模块使用`/// <reference>`

一个常见的错误是使用`/// <reference>`引用模块文件，应该使用`import`。 要理解这之间的区别，我们首先应该弄清编译器是如何根据`import`路径（例如，`import x from "...";`或`import x = require("...")`里面的`...`，等等）来定位模块的类型信息的。

编译器首先尝试去查找相应路径下的`.ts`，`.tsx`再或者`.d.ts`。 如果这些文件都找不到，编译器会查找*外部模块声明*。 回想一下，它们是在`.d.ts`文件里声明的。

*   `myModules.d.ts`

```js
// In a .d.ts file or .ts file that is not a module:
declare module "SomeModule" {
    export function fn(): string;
} 
```

*   `myOtherModule.ts`

```js
/// <reference path="myModules.d.ts" />
import * as m from "SomeModule"; 
```

这里的引用标签指定了外来模块的位置。 这就是一些 Typescript 例子中引用`node.d.ts`的方法。

## 不必要的命名空间

如果你想把命名空间转换为模块，它可能会像下面这个文件一件：

*   `shapes.ts`

```js
export namespace Shapes {
    export class Triangle { /* ... */ }
    export class Square { /* ... */ }
} 
```

顶层的模块`Shapes`包裹了`Triangle`和`Square`。 对于使用它的人来说这是令人迷惑和讨厌的：

*   `shapeConsumer.ts`

```js
import * as shapes from "./shapes";
let t = new shapes.Shapes.Triangle(); // shapes.Shapes? 
```

TypeScript 里模块的一个特点是不同的模块永远也不会在相同的作用域内使用相同的名字。 因为使用模块的人会为它们命名，所以完全没有必要把导出的符号包裹在一个命名空间里。

再次重申，不应该对模块使用命名空间，使用命名空间是为了提供逻辑分组和避免命名冲突。 模块文件本身已经是一个逻辑分组，并且它的名字是由导入这个模块的代码指定，所以没有必要为导出的对象增加额外的模块层。

下面是改进的例子：

*   `shapes.ts`

```js
export class Triangle { /* ... */ }
export class Square { /* ... */ } 
```

*   `shapeConsumer.ts`

```js
import * as shapes from "./shapes";
let t = new shapes.Triangle(); 
```

## 模块的取舍

就像每个 JS 文件对应一个模块一样，TypeScript 里模块文件与生成的 JS 文件也是一一对应的。 这会产生一个效果，就是无法使用`--out`来让编译器合并多个模块文件为一个 JavaScript 文件。