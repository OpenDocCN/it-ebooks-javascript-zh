## Class 基本语法

### （1）概述

JavaScript 语言的传统方法是通过构造函数，定义并生成新对象。下面是一个例子。

```
      function Point(x,y){
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function () {
  return '(' + this.x + ', ' + this.y + ')';
}

```

上面这种写法跟传统的面向对象语言（比如 C++和 Java）差异很大，很容易让新学习这门语言的程序员感到困惑。

ES6 提供了更接近传统语言的写法，引入了 Class（类）这个概念，作为对象的模板。通过 class 关键字，可以定义类。基本上，ES6 的 class 可以看作只是一个语法糖，它的绝大部分功能，ES5 都可以做到，新的 class 写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。上面的代码用 ES6 的“类”改写，就是下面这样。

```
      //定义类
class Point {

  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '('+this.x+', '+this.y+')';
  }

}

```

上面代码定义了一个“类”，可以看到里面有一个 constructor 方法，这就是构造方法，而 this 关键字则代表实例对象。也就是说，ES5 的构造函数 Point，对应 ES6 的 Point 类的构造方法。

Point 类除了构造方法，还定义了一个 toString 方法。注意，定义“类”的方法的时候，前面不需要加上 function 这个保留字，直接把函数定义放进去了就可以了。

ES6 的类，完全可以看作构造函数的另一种写法。

```
      class Point{
  // ...
}

typeof Point // "function"

```

上面代码表明，类的数据类型就是函数。

构造函数的 prototype 属性，在 ES6 的“类”上面继续存在。事实上，除了 constructor 方法以外，类的方法都定义在类的 prototype 属性上面。

```
      class Point {
  constructor(){
    // ...
  }

  toString(){
    // ...
  }

  toValue(){
    // ...
  }
}

// 等同于

Point.prototype = {
  toString(){},
  toValue(){}
}

```

由于类的方法（除 constructor 以外）都定义在 prototype 对象上面，所以类的新方法可以添加在 prototype 对象上面。`Object.assign`方法可以很方便地一次向类添加多个方法。

```
      class Point {
  constructor(){
    // ...
  }
}

Object.assign(Point.prototype, {
  toString(){},
  toValue(){}
})

```

prototype 对象的 constructor 属性，直接指向“类”的本身，这与 ES5 的行为是一致的。

```
      Point.prototype.constructor === Point // true

```

另外，类的内部所有定义的方法，都是不可枚举的（enumerable）。

```
      class Point {
  constructor(x, y) {
    // ...
  }

  toString() {
    // ...
  }
}

Object.keys(Point.prototype)
// []
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]

```

上面代码中，toString 方法是 Point 类内部定义的方法，它是不可枚举的。这一点与 ES5 的行为不一致。

```
      var Point = function (x, y){
  // ...
}

Point.prototype.toString = function() {
  // ...
}

Object.keys(Point.prototype)
// ["toString"]
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]

```

上面代码采用 ES5 的写法，toString 方法就是可枚举的。

类的属性名，可以采用表达式。

```
      let methodName = "getArea";
class Square{
  constructor(length) {
    // ...
  }

  [methodName]() {
    // ...
  }
}

```

上面代码中，Square 类的方法名 getArea，是从表达式得到的。

### （2）constructor 方法

constructor 方法是类的默认方法，通过 new 命令生成对象实例时，自动调用该方法。一个类必须有 constructor 方法，如果没有显式定义，一个空的 constructor 方法会被默认添加。

```
      constructor() {}

```

constructor 方法默认返回实例对象（即 this），完全可以指定返回另外一个对象。

```
      class Foo {
  constructor() {
    return Object.create(null);
  }
}

new Foo() instanceof Foo
// false

```

上面代码中，constructor 函数返回一个全新的对象，结果导致实例对象不是 Foo 类的实例。

### （3）实例对象

生成实例对象的写法，与 ES5 完全一样，也是使用 new 命令。如果忘记加上 new，像函数那样调用 Class，将会报错。

```
      // 报错
var point = Point(2, 3);

// 正确
var point = new Point(2, 3);

```

与 ES5 一样，实例的属性除非显式定义在其本身（即定义在 this 对象上），否则都是定义在原型上（即定义在 class 上）。

```
      //定义类
class Point {

  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '('+this.x+', '+this.y+')';
  }

}

var point = new Point(2, 3);

point.toString() // (2, 3)

point.hasOwnProperty('x') // true
point.hasOwnProperty('y') // true
point.hasOwnProperty('toString') // false
point.__proto__.hasOwnProperty('toString') // true

```

上面代码中，x 和 y 都是实例对象 point 自身的属性（因为定义在 this 变量上），所以 hasOwnProperty 方法返回 true，而 toString 是原型对象的属性（因为定义在 Point 类上），所以 hasOwnProperty 方法返回 false。这些都与 ES5 的行为保持一致。

与 ES5 一样，类的所有实例共享一个原型对象。

```
      var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__ === p2.__proto__
//true

```

上面代码中，p1 和 p2 都是 Point 的实例，它们的原型都是 Point，所以**proto**属性是相等的。

这也意味着，可以通过实例的**proto**属性为 Class 添加方法。

```
      var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__.printName = function () { return 'Oops' };

p1.printName() // "Oops"
p2.printName() // "Oops"

var p3 = new Point(4,2);
p3.printName() // "Oops"

```

上面代码在 p1 的原型上添加了一个 printName 方法，由于 p1 的原型就是 p2 的原型，因此 p2 也可以调用这个方法。而且，此后新建的实例 p3 也可以调用这个方法。这意味着，使用实例的**proto**属性改写原型，必须相当谨慎，不推荐使用，因为这会改变 Class 的原始定义，影响到所有实例。

### （4）name 属性

由于本质上，ES6 的 Class 只是 ES5 的构造函数的一层包装，所以函数的许多特性都被 Class 继承，包括 name 属性。

```
      class Point {}
Point.name // "Point"

```

name 属性总是返回紧跟在 class 关键字后面的类名。

### （5）Class 表达式

与函数一样，Class 也可以使用表达式的形式定义。

```
      const MyClass = class Me {
  getClassName() {
    return Me.name;
  }
};

```

上面代码使用表达式定义了一个类。需要注意的是，这个类的名字是 MyClass 而不是 Me，Me 只在 Class 的内部代码可用，指代当前类。

```
      let inst = new MyClass();
inst.getClassName() // Me
Me.name // ReferenceError: Me is not defined

```

上面代码表示，Me 只在 Class 内部有定义。

如果 Class 内部没用到的话，可以省略 Me，也就是可以写成下面的形式。

```
      const MyClass = class { /* ... */ };

```

采用 Class 表达式，可以写出立即执行的 Class。

```
      let person = new class {
  constructor(name) {
    this.name = name;
  }

  sayName() {
    console.log(this.name);
  }
}("张三");

person.sayName(); // "张三"

```

上面代码中，person 是一个立即执行的 Class 的实例。

### （6）不存在变量提升

Class 不存在变量提升（hoist），这一点与 ES5 完全不同。

```
      new Foo(); // ReferenceError
class Foo {}

```

上面代码中，Foo 类使用在前，定义在后，这样会报错，因为 ES6 不会把变量声明提升到代码头部。这种规定的原因与下文要提到的继承有关，必须保证子类在父类之后定义。

```
      {
  let Foo = class {};
  class Bar extends Foo {
  }
}

```

如果存在 Class 的提升，上面代码将报错，因为 let 命令也是不提升的。

### （7）严格模式

类和模块的内部，默认就是严格模式，所以不需要使用`use strict`指定运行模式。只要你的代码写在类或模块之中，就只有严格模式可用。

考虑到未来所有的代码，其实都是运行在模块之中，所以 ES6 实际上把整个语言升级到了严格模式。

## Class 的继承

### 基本用法

Class 之间可以通过 extends 关键字，实现继承，这比 ES5 的通过修改原型链实现继承，要清晰和方便很多。

```
      class ColorPoint extends Point {}

```

上面代码定义了一个 ColorPoint 类，该类通过 extends 关键字，继承了 Point 类的所有属性和方法。但是由于没有部署任何代码，所以这两个类完全一样，等于复制了一个 Point 类。下面，我们在 ColorPoint 内部加上代码。

```
      class ColorPoint extends Point {

  constructor(x, y, color) {
    super(x, y); // 调用父类的 constructor(x, y)
    this.color = color;
  }

  toString() {
    return this.color + ' ' + super.toString(); // 调用父类的 toString()
  }

}

```

上面代码中，constructor 方法和 toString 方法之中，都出现了 super 关键字，它指代父类的实例（即父类的 this 对象）。

子类必须在 constructor 方法中调用 super 方法，否则新建实例时会报错。这是因为子类没有自己的 this 对象，而是继承父类的 this 对象，然后对其进行加工。如果不调用 super 方法，子类就得不到 this 对象。

```
      class Point { /* ... */ }

class ColorPoint extends Point {
  constructor() {
  }
}

let cp = new ColorPoint(); // ReferenceError

```

上面代码中，ColorPoint 继承了父类 Point，但是它的构造函数没有调用 super 方法，导致新建实例时报错。

ES5 的继承，实质是先创造子类的实例对象 this，然后再将父类的方法添加到 this 上面（`Parent.apply(this)`）。ES6 的继承机制完全不同，实质是先创造父类的实例对象 this（所以必须先调用 super 方法），然后再用子类的构造函数修改 this。

如果子类没有定义 constructor 方法，这个方法会被默认添加，代码如下。也就是说，不管有没有显式定义，任何一个子类都有 constructor 方法。

```
      constructor(...args) {
  super(...args);
}

```

另一个需要注意的地方是，在子类的构造函数中，只有调用 super 之后，才可以使用 this 关键字，否则会报错。这是因为子类实例的构建，是基于对父类实例加工，只有 super 方法才能返回父类实例。

```
      class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}

class ColorPoint extends Point {
  constructor(x, y, color) {
    this.color = color; // ReferenceError
    super(x, y);
    this.color = color; // 正确
  }
}

```

上面代码中，子类的 constructor 方法没有调用 super 之前，就使用 this 关键字，结果报错，而放在 super 方法之后就是正确的。

下面是生成子类实例的代码。

```
      let cp = new ColorPoint(25, 8, 'green');

cp instanceof ColorPoint // true
cp instanceof Point // true

```

上面代码中，实例对象 cp 同时是 ColorPoint 和 Point 两个类的实例，这与 ES5 的行为完全一致。

### 类的 prototype 属性和**proto**属性

在 ES5 中，每一个对象都有`__proto__`属性，指向对应的构造函数的 prototype 属性。Class 作为构造函数的语法糖，同时有 prototype 属性和`__proto__`属性，因此同时存在两条继承链。

（1）子类的`__proto__`属性，表示构造函数的继承，总是指向父类。

（2）子类 prototype 属性的`__proto__`属性，表示方法的继承，总是指向父类的 prototype 属性。

```
      class A {
}

class B extends A {
}

B.__proto__ === A // true
B.prototype.__proto__ === A.prototype // true

```

上面代码中，子类 A 的`__proto__`属性指向父类 B，子类 A 的 prototype 属性的**proto**属性指向父类 B 的 prototype 属性。

这两条继承链，可以这样理解：作为一个对象，子类（B）的原型（`__proto__ 属性`）是父类（A）；作为一个构造函数，子类（B）的原型（prototype 属性）是父类的实例。

```
      B.prototype = new A();
// 等同于
B.prototype.__proto__ = A.prototype;

```

此外，考虑三种特殊情况。第一种特殊情况，子类继承 Object 类。

```
      class A extends Object {
}

A.__proto__ === Object // true
A.prototype.__proto__ === Object.prototype // true

```

这种情况下，A 其实就是构造函数 Object 的复制，A 的实例就是 Object 的实例。

第二种特性情况，不存在任何继承。

```
      class A {
}

A.__proto__ === Function.prototype // true
A.prototype.__proto__ === Object.prototype // true

```

这种情况下，A 作为一个基类（即不存在任何继承），就是一个普通函数，所以直接继承`Funciton.prototype`。但是，A 调用后返回一个空对象（即 Object 实例），所以`A.prototype.__proto__`指向构造函数（Object）的 prototype 属性。

第三种特殊情况，子类继承 null。

```
      class A extends null {
}

A.__proto__ === Function.prototype // true
A.prototype.__proto__ === null // true

```

这种情况与第二种情况非常像。A 也是一个普通函数，所以直接继承`Funciton.prototype`。但是，A 调用后返回的对象不继承任何方法，所以它的`__proto__`指向`Function.prototype`，即实质上执行了下面的代码。

```
      class C extends null {
  constructor() { return Object.create(null); }
}

```

### Object.getPrototypeOf()

Object.getPrototypeOf 方法可以用来从子类上获取父类。

```
      Object.getPrototypeOf(ColorPoint) === Point
// true

```

### 实例的**proto**属性

父类实例和子类实例的**proto**属性，指向是不一样的。

```
      var p1 = new Point(2, 3);
var p2 = new ColorPoint(2, 3, 'red');

p2.__proto__ === p1.__proto // false
p2.__proto__.__proto__ === p1.__proto__ // true

```

通过子类实例的**proto**属性，可以修改父类实例的行为。

```
      p2.__proto__.__proto__.printName = function () {
  console.log('Ha');
};

p1.printName() // "Ha"

```

上面代码在 ColorPoint 的实例 p2 上向 Point 类添加方法，结果影响到了 Point 的实例 p1。

### 原生构造函数的继承

原生构造函数是指语言内置的构造函数，通常用来生成数据结构，比如`Array()`。以前，这些原生构造函数是无法继承的，即不能自己定义一个 Array 的子类。

```
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

```

上面代码定义了一个继承 Array 的 MyArray 类。但是，这个类的行为与 Array 完全不一致。

```
      var colors = new MyArray();
colors[0] = "red";
colors.length  // 0

colors.length = 0;
colors[0]  // "red"

```

之所以会发生这种情况，是因为原生构造函数无法外部获取，通过`Array.apply()`或者分配给原型对象都不行。ES5 是先新建子类的实例对象 this，再将父类的属性添加到子类上，由于父类的属性无法获取，导致无法继承原生的构造函数。

ES6 允许继承原生构造函数定义子类，因为 ES6 是先新建父类的实例对象 this，然后再用子类的构造函数修饰 this，使得父类的所有行为都可以继承。下面是一个继承 Array 的例子。

```
      class MyArray extends Array {
  constructor(...args) {
    super(...args);
  }
}

var arr = new MyArray();
arr[0] = 12;
arr.length // 1

arr.length = 0;
arr[0] // undefined

```

上面代码定义了一个 MyArray 类，继承了 Array 构造函数，因此就可以从 MyArray 生成数组的实例。这意味着，ES6 可以自定义原生数据结构（比如 Array、String 等）的子类，这是 ES5 无法做到的。

上面这个例子也说明，extends 关键字不仅可以用来继承类，还可以用来继承原生的构造函数。下面是一个自定义 Error 子类的例子。

```
      class MyError extends Error {
}

throw new MyError('Something happened!');

```

## class 的取值函数（getter）和存值函数（setter）

与 ES5 一样，在 Class 内部可以使用 get 和 set 关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。

```
      class MyClass {
  constructor() {
    // ...
  }
  get prop() {
    return 'getter';
  }
  set prop(value) {
    console.log('setter: '+value);
  }
}

let inst = new MyClass();

inst.prop = 123;
// setter: 123

inst.prop
// 'getter'

```

上面代码中，prop 属性有对应的存值函数和取值函数，因此赋值和读取行为都被自定义了。

存值函数和取值函数是设置在属性的 descriptor 对象上的。

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

var descriptor = Object.getOwnPropertyDescriptor(
  CustomHTMLElement.prototype, "html");
"get" in descriptor  // true
"set" in descriptor  // true

```

上面代码中，存值函数和取值函数是定义在 html 属性的描述对象上面，这与 ES5 完全一致。

下面的例子针对所有属性，设置存值函数和取值函数。

```
      class Jedi {
  constructor(options = {}) {
    // ...
  }

  set(key, val) {
    this[key] = val;
  }

  get(key) {
    return this[key];
  }
}

```

上面代码中，Jedi 实例所有属性的存取，都会通过存值函数和取值函数。

## Class 的 Generator 方法

如果某个方法之前加上星号（*），就表示该方法是一个 Generator 函数。

```
      class Foo {
  constructor(...args) {
    this.args = args;
  }
  * [Symbol.iterator]() {
    for (let arg of this.args) {
      yield arg;
    }
  }
}

for (let x of new Foo('hello', 'world')) {
  console.log(x);
}
// hello
// world

```

上面代码中，Foo 类的 Symbol.iterator 方法前有一个星号，表示该方法是一个 Generator 函数。Symbol.iterator 方法返回一个 Foo 类的默认遍历器，for...of 循环会自动调用这个遍历器。

## Class 的静态方法

类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上 static 关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。

```
      class Foo {
  static classMethod() {
    return 'hello';
  }
}

Foo.classMethod() // 'hello'

var foo = new Foo();
foo.classMethod()
// TypeError: undefined is not a function

```

上面代码中，Foo 类的 classMethod 方法前有 static 关键字，表明该方法是一个静态方法，可以直接在 Foo 类上调用（`Foo.classMethod()`），而不是在 Foo 类的实例上调用。如果在实例上调用静态方法，会抛出一个错误，表示不存在该方法。

父类的静态方法，可以被子类继承。

```
      class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
}

Bar.classMethod(); // 'hello'

```

上面代码中，父类 Foo 有一个静态方法，子类 Bar 可以调用这个方法。

静态方法也是可以从 super 对象上调用的。

```
      class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
  static classMethod() {
    return super.classMethod() + ', too';
  }
}

Bar.classMethod();

```

## new.target 属性

new 是从构造函数生成实例的命令。ES6 为 new 命令引入了一个`new.target`属性，（在构造函数中）返回 new 命令作用于的那个构造函数。如果构造函数不是通过 new 命令调用的，`new.target`会返回 undefined，因此这个属性可以用来确定构造函数是怎么调用的。

```
      function Person(name) {
  if (new.target !== undefined) {
    this.name = name;
  } else {
    throw new Error('必须使用 new 生成实例');
  }
}

// 另一种写法
function Person(name) {
  if (new.target === Person) {
    this.name = name;
  } else {
    throw new Error('必须使用 new 生成实例');
  }
}

var person = new Person('张三'); // 正确
var notAPerson = Person.call(person, '张三');  // 报错

```

上面代码确保构造函数只能通过 new 命令调用。

Class 内部调用`new.target`，返回当前 Class。

```
      class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);
    this.length = length;
    this.width = width;
  }
}

var obj = new Rectangle(3, 4); // 输出 true

```

需要注意的是，子类继承父类时，`new.target`会返回子类。

```
      class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);
    // ...
  }
}

class Square extends Rectangle {
  constructor(length) {
    super(length, length);
  }
}

var obj = new Square(3); // 输出 false

```

上面代码中，`new.target`会返回子类。

利用这个特点，可以写出不能独立使用、必须继承后才能使用的类。

```
      class Shape {
  constructor() {
    if (new.target === Shape) {
      throw new Error('本类不能实例化');
    }
  }
}

class Rectangle extends Shape {
  constructor(length, width) {
    super();
    // ...
  }
}

var x = new Shape();  // 报错
var y = new Rectangle(3, 4);  // 正确

```

上面代码中，Shape 类不能被实例化，只能用于继承。

注意，在函数外部，使用`new.target`会报错。

## 修饰器

### 类的修饰

修饰器（Decorator）是一个表达式，用来修改类的行为。这是 ES7 的一个[提案](https://github.com/wycats/javascript-decorators)，目前 Babel 转码器已经支持。

修饰器对类的行为的改变，是代码编译时发生的，而不是在运行时。这意味着，修饰器能在编译阶段运行代码。

```
      function testable(target) {
  target.isTestable = true;
}

@testable
class MyTestableClass () {}

console.log(MyTestableClass.isTestable) // true

```

上面代码中，`@testable`就是一个修饰器。它修改了 MyTestableClass 这个类的行为，为它加上了静态属性 isTestable。

修饰器函数可以接受三个参数，依次是目标函数、属性名和该属性的描述对象。后两个参数可省略。上面代码中，testable 函数的参数 target，就是所要修饰的对象。如果希望修饰器的行为，能够根据目标对象的不同而不同，就要在外面再封装一层函数。

```
      function testable(isTestable) {
  return function(target) {
    target.isTestable = isTestable;
  }
}

@testable(true) class MyTestableClass () {}
console.log(MyTestableClass.isTestable) // true

@testable(false) class MyClass () {}
console.log(MyClass.isTestable) // false

```

上面代码中，修饰器 testable 可以接受参数，这就等于可以修改修饰器的行为。

如果想要为类的实例添加方法，可以在修饰器函数中，为目标类的 prototype 属性添加方法。

```
      function testable(target) {
  target.prototype.isTestable = true;
}

@testable
class MyTestableClass () {}

let obj = new MyClass();

console.log(obj.isTestable) // true

```

上面代码中，修饰器函数 testable 是在目标类的 prototype 属性添加属性，因此就可以在类的实例上调用添加的属性。

下面是另外一个例子。

```
      // mixins.js
export function mixins(...list) {
  return function (target) {
    Object.assign(target.prototype, ...list)
  }
}

// main.js
import { mixins } from './mixins'

const Foo = {
  foo() { console.log('foo') }
}

@mixins(Foo)
class MyClass {}

let obj = new MyClass()

obj.foo() // 'foo'

```

上面代码通过修饰器 mixins，可以为类添加指定的方法。

修饰器可以用`Object.assign()`模拟。

```
      const Foo = {
  foo() { console.log('foo') }
}

class MyClass {}

Object.assign(MyClass.prototype, Foo);

let obj = new MyClass();
obj.foo() // 'foo'

```

### 方法的修饰

修饰器不仅可以修饰类，还可以修饰类的属性。

```
      class Person {
  @readonly
  name() { return `${this.first} ${this.last}` }
}

```

上面代码中，修饰器 readonly 用来修饰”类“的 name 方法。

此时，修饰器函数一共可以接受三个参数，第一个参数是所要修饰的目标对象，第二个参数是所要修饰的属性名，第三个参数是该属性的描述对象。

```
      readonly(Person.prototype, 'name', descriptor);

function readonly(target, name, descriptor){
  // descriptor 对象原来的值如下
  // {
  //   value: specifiedFunction,
  //   enumerable: false,
  //   configurable: true,
  //   writable: true
  // };
  descriptor.writable = false;
  return descriptor;
}

Object.defineProperty(Person.prototype, 'name', descriptor);

```

上面代码说明，修饰器（readonly）会修改属性的描述对象（descriptor），然后被修改的描述对象再用来定义属性。下面是另一个例子。

```
      class Person {
  @nonenumerable
  get kidCount() { return this.children.length; }
}

function nonenumerable(target, name, descriptor) {
  descriptor.enumerable = false;
  return descriptor;
}

```

修饰器有注释的作用。

```
      @testable
class Person {
  @readonly
  @nonenumerable
  name() { return `${this.first} ${this.last}` }
}

```

从上面代码中，我们一眼就能看出，MyTestableClass 类是可测试的，而 name 方法是只读和不可枚举的。

除了注释，修饰器还能用来类型检查。所以，对于 Class 来说，这项功能相当有用。从长期来看，它将是 JavaScript 代码静态分析的重要工具。

### core-decorators.js

[core-decorators.js](https://github.com/jayphelps/core-decorators.js)是一个第三方模块，提供了几个常见的修饰器，通过它可以更好地理解修饰器。

**（1）@autobind**

autobind 修饰器使得方法中的 this 对象，绑定原始对象。

```
      import { autobind } from 'core-decorators';

class Person {
  @autobind
  getPerson() {
    return this;
  }
}

let person = new Person();
let getPerson = person.getPerson;

getPerson() === person;
// true

```

**（2）@readonly**

readonly 修饰器是的属性或方法不可写。

```
      import { readonly } from 'core-decorators';

class Meal {
  @readonly
  entree = 'steak';
}

var dinner = new Meal();
dinner.entree = 'salmon';
// Cannot assign to read only property 'entree' of [object Object]

```

**（3）@override**

override 修饰器检查子类的方法，是否正确覆盖了父类的同名方法，如果不正确会报错。

```
      import { override } from 'core-decorators';

class Parent {
  speak(first, second) {}
}

class Child extends Parent {
  @override
  speak() {}
  // SyntaxError: Child#speak() does not properly override Parent#speak(first, second)
}

// or

class Child extends Parent {
  @override
  speaks() {}
  // SyntaxError: No descriptor matching Child#speaks() was found on the prototype chain.
  //
  //   Did you mean "speak"?
}

```

**（4）@deprecate (别名@deprecated)**

deprecate 或 deprecated 修饰器在控制台显示一条警告，表示该方法将废除。

```
      import { deprecate } from 'core-decorators';

class Person {
  @deprecate
  facepalm() {}

  @deprecate('We stopped facepalming')
  facepalmHard() {}

  @deprecate('We stopped facepalming', { url: 'http://knowyourmeme.com/memes/facepalm' })
  facepalmHarder() {}
}

let person = new Person();

person.facepalm();
// DEPRECATION Person#facepalm: This function will be removed in future versions.

person.facepalmHard();
// DEPRECATION Person#facepalmHard: We stopped facepalming

person.facepalmHarder();
// DEPRECATION Person#facepalmHarder: We stopped facepalming
//
//     See http://knowyourmeme.com/memes/facepalm for more details.
//

```

**（5）@suppressWarnings**

suppressWarnings 修饰器抑制 decorated 修饰器导致的`console.warn()`调用。但是，异步代码出发的调用除外。

```
      import { suppressWarnings } from 'core-decorators';

class Person {
  @deprecated
  facepalm() {}

  @suppressWarnings
  facepalmWithoutWarning() {
    this.facepalm();
  }
}

let person = new Person();

person.facepalmWithoutWarning();
// no warning is logged

```

### Mixin

在修饰器的基础上，可以实现 Mixin 模式。所谓 Mixin 模式，就是对象继承的一种替代方案，中文译为“混入”（mix in），意为在一个对象之中混入另外一个对象的方法。

请看下面的例子。

```
      const Foo = {
  foo() { console.log('foo') }
};

class MyClass {}

Object.assign(MyClass.prototype, Foo);

let obj = new MyClass();
obj.foo() // 'foo'

```

上面代码之中，对象 Foo 有一个 foo 方法，通过`Object.assign`方法，可以将 foo 方法“混入”MyClass 类，导致 MyClass 的实例 obj 对象都具有 foo 方法。这就是“混入”模式的一个简单实现。

下面，我们部署一个通用脚本`mixins.js`，将 mixin 写成一个修饰器。

```
      export function mixins(...list) {
  return function (target) {
    Object.assign(target.prototype, ...list);
  };
}

```

然后，就可以使用上面这个修饰器，为类“混入”各种方法。

```
      import { mixins } from './mixins'

const Foo = {
  foo() { console.log('foo') }
};

@mixins(Foo)
class MyClass {}

let obj = new MyClass();

obj.foo() // "foo"

```

通过 mixins 这个修饰器，实现了在 MyClass 类上面“混入”Foo 对象的 foo 方法。

### Trait

Trait 也是一种修饰器，功能与 Mixin 类型，但是提供更多功能，比如防止同名方法的冲突、排除混入某些方法、为混入的方法起别名等等。

下面采用[traits-decorator](https://github.com/CocktailJS/traits-decorator)这个第三方模块作为例子。这个模块提供的 traits 修饰器，不仅可以接受对象，还可以接受 ES6 类作为参数。

```
      import {traits } from 'traits-decorator'

class TFoo {
  foo() { console.log('foo') }
}

const TBar = {
  bar() { console.log('bar') }
}

@traits(TFoo, TBar)
class MyClass { }

let obj = new MyClass()
obj.foo() // foo
obj.bar() // bar

```

上面代码中，通过 traits 修饰器，在 MyClass 类上面“混入”了 TFoo 类的 foo 方法和 TBar 对象的 bar 方法。

Trait 不允许“混入”同名方法。

```
      import {traits } from 'traits-decorator'

class TFoo {
  foo() { console.log('foo') }
}

const TBar = {
  bar() { console.log('bar') },
  foo() { console.log('foo') }
}

@traits(TFoo, TBar)
class MyClass { }
// 报错
// throw new Error('Method named: ' + methodName + ' is defined twice.');
//        ^
// Error: Method named: foo is defined twice.

```

上面代码中，TFoo 和 TBar 都有 foo 方法，结果 traits 修饰器报错。

一种解决方法是排除 TBar 的 foo 方法。

```
      import { traits, excludes } from 'traits-decorator'

class TFoo {
  foo() { console.log('foo') }
}

const TBar = {
  bar() { console.log('bar') },
  foo() { console.log('foo') }
}

@traits(TFoo, TBar::excludes('foo'))
class MyClass { }

let obj = new MyClass()
obj.foo() // foo
obj.bar() // bar

```

上面代码使用绑定运算符（::）在 TBar 上排除 foo 方法，混入时就不会报错了。

另一种方法是为 TBar 的 foo 方法起一个别名。

```
      import { traits, alias } from 'traits-decorator'

class TFoo {
  foo() { console.log('foo') }
}

const TBar = {
  bar() { console.log('bar') },
  foo() { console.log('foo') }
}

@traits(TFoo, TBar::alias({foo: 'aliasFoo'}))
class MyClass { }

let obj = new MyClass()
obj.foo() // foo
obj.aliasFoo() // foo
obj.bar() // bar

```

上面代码为 TBar 的 foo 方法起了别名 aliasFoo，于是 MyClass 也可以混入 TBar 的 foo 方法了。

alias 和 excludes 方法，可以结合起来使用。

```
      @traits(TExample::excludes('foo','bar')::alias({baz:'exampleBaz'}))
class MyClass {}

```

上面代码排除了 TExample 的 foo 方法和 bar 方法，为 baz 方法起了别名 exampleBaz。

as 方法则为上面的代码提供了另一种写法。

```
      @traits(TExample::as({excludes:['foo', 'bar'], alias: {baz: 'exampleBaz'}}))
class MyClass {}

```

### Babel 转码器的支持

目前，Babel 转码器已经支持 Decorator，命令行的用法如下。

```
      $ babel --optional es7.decorators

```

脚本中打开的命令如下。

```
      babel.transfrom("code", {optional: ["es7.decorators"]})

```

Babel 的官方网站提供一个[在线转码器](https://babeljs.io/repl/)，只要勾选 Experimental，就能支持 Decorator 的在线转码。