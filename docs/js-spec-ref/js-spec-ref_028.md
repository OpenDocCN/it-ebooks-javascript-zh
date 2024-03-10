# 4.2 封装

*   prototype 对象
    *   构造函数的缺点
    *   prototype 属性的作用
    *   原型链
    *   constructor 属性
    *   Object.getPrototypeOf 方法
    *   Object.create 方法
    *   isPrototypeOf 方法

## prototype 对象

### 构造函数的缺点

JavaScript 通过构造函数生成新对象，因此构造函数可以视为对象的模板。实例对象的属性和方法，可以定义在构造函数内部。

```
function Animal (name) {
  this.name = name;
  this.color = 'white';
}

var cat1 = new Animal('大毛');

cat1.name // '大毛'
cat1.color // 'white'
```

上面代码的 Animal 函数是一个构造函数，函数内部定义了 name 属性和 color 属性，所有实例对象都会生成这两个属性。

但是，这样做是对系统资源的浪费，因为同一个构造函数的对象实例之间，无法共享属性。

### prototype 属性的作用

在 JavaScript 语言中，每一个对象都有一个对应的原型对象，被称为 prototype 对象。定义在原型对象上的所有属性和方法，都能被派生对象继承。这就是 JavaScript 继承机制的基本设计。

除了这种方法，JavaScript 还提供了另一种定义实例对象的方法。我们知道，构造函数是一个函数，同时也是一个对象，也有自己的属性和方法，其中有一个 prototype 属性指向另一个对象，一般称为 prototype 对象。该对象非常特别，只要定义在它上面的属性和方法，能被所有实例对象共享。也就是说，构造函数生成实例对象时，自动为实例对象分配了一个 prototype 属性。

```
function Animal (name) {
  this.name = name;
}

Animal.prototype.color = "white";

var cat1 = new Animal('大毛');
var cat2 = new Animal('二毛');

cat1.color // 'white'
cat2.color // 'white'
```

上面代码对构造函数 Animal 的 prototype 对象，添加了一个 color 属性。结果，实例对象 cat1 和 cat2 都带有该属性。

更特别的是，只要修改 prototype 对象，变动就立刻会体现在实例对象。

```
Animal.prototype.color = "yellow";

cat1.color // 'yellow'
cat2.color // 'yellow'
```

上面代码将 prototype 对象的 color 属性的值改为 yellow，两个实例对象的 color 属性的值立刻就跟着变了。这是因为实例对象其实没有 color 属性，都是读取 prototype 对象的 color 属性。也就是说，当实例对象本身没有某个属性或方法的时候，它会到构造函数的 prototype 对象去寻找该属性或方法。这就是 prototype 对象的特殊之处。

如果实例对象自身就有某个属性或方法，它就不会再去 prototype 对象寻找这个属性或方法。

```
cat1.color = 'black';

cat2.color // 'yellow'
Animal.prototype.color // "yellow";
```

上面代码将实例对象 cat1 的 color 属性改为 black，就使得它不用再去 prototype 对象读取 color 属性，后者的值依然为 yellow。

总而言之，prototype 对象的作用，就是定义所有实例对象共享的属性和方法，所以它也被称为实例对象的原型，而实例对象可以视作从 prototype 对象衍生出来的。

```
Animal.prototype.walk = function () {
  console.log(this.name + ' is walking.');
};
```

上面代码在 Animal.protype 对象上面定义了一个 walk 方法，这个方法将可以在所有 Animal 实例对象上面调用。

### 原型链

由于 JavaScript 的所有对象都有构造函数，而所有构造函数都有 prototype 属性（其实是所有函数都有 prototype 属性），所以所有对象都有自己的 prototype 原型对象。

因此，一个对象的属性和方法，有可能是定义它自身上面，也有可能定义在它的原型对象上面（就像上面代码中的 walk 方法）。由于原型本身也是对象，又有自己的原型，所以形成了一条原型链（prototype chain）。比如，a 对象是 b 对象的原型，b 对象是 c 对象的原型，以此类推。因为追根溯源，最源头的对象都是从 Object 构造函数生成（使用 new Object()命令），所以如果一层层地上溯，所有对象的原型最终都可以上溯到 Object.prototype。那么，Object.prototype 有没有原型呢？回答可以是有，也可以是没有，因为 Object.prototype 的原型是没有任何属性和方法的 null。

```
Object.getPrototypeOf(Object.prototype)
// null
```

上面代码表示 Object.prototype 对象的原型是 null，由于 null 没有任何属性，所以原型链到此为止。

“原型链”的作用在于，当读取对象的某个属性时，JavaScript 引擎先寻找对象本身的属性，如果找不到，就到它的原型去找，如果还是找不到，就到原型的原型去找。以此类推，如果直到最顶层的 Object.prototype 还是找不到，则返回 undefined。

举例来说，如果让某个函数的 prototype 属性指向一个数组，就意味着该函数可以用作数组的构造函数，因为它生成的实例对象都可以通过 prototype 属性调用数组方法。

```
function MyArray (){}

MyArray.prototype = new Array();
MyArray.prototype.constructor = MyArray;

var mine = new MyArray();
mine.push(1, 2, 3);

mine.length // 3
mine instanceof Array // true
```

上面代码的 mine 是 MyArray 的实例对象，由于 MyArray 的 prototype 属性指向一个数组，使得 mine 可以调用数组方法（这些方法其实定义在数组的 prototype 对象上面）。至于最后那行 instanceof 表达式，我们知道 instanceof 运算符用来比较一个对象是否为某个构造函数的实例，最后一行表示 mine 为 Array 的实例。

```
mine instanceof Array

// 等同于

(Array === MyArray.prototype.constructor) ||
(Array === Array.prototype.constructor) ||
(Array === Object.prototype.constructor )
```

上面代码说明了 instanceof 运算符的实质，它依次与实例对象的所有原型对象的 constructor 属性（关于该属性的介绍，请看下一节）进行比较，只要有一个符合就返回 true，否则返回 false。

### constructor 属性

prototype 对象有一个 constructor 属性，默认指向 prototype 对象所在的构造函数。

```
function P() {}

P.prototype.constructor === P
// true
```

由于 constructor 属性定义在 prototype 对象上面，意味着可以被所有实例对象继承。

```
function P() {}

var p = new P();

p.constructor
// function P() {}

p.constructor === P.prototype.constructor
// true

p.hasOwnProperty('constructor')
// false
```

上面代码表示 p 是构造函数 P 的实例对象，但是 p 自身没有 contructor 属性，该属性其实是读取原型链上面的`P.prototype.constructor`属性。

constructor 属性的作用是分辨 prototype 对象到底定义在哪个构造函数上面。

```
function F(){};

var f = new F();

f.constructor === F // true
f.constructor === RegExp // false
```

上面代码表示，使用 constructor 属性，确定变量 f 的构造函数是 F，而不是 RegExp。

## Object.getPrototypeOf 方法

Object.getPrototypeOf 方法返回一个对象的原型。

```
// 空对象的原型是 Object.prototype
Object.getPrototypeOf({}) === Object.prototype
// true

// 函数的原型是 Function.prototype
function f() {}
Object.getPrototypeOf(f) === Function.prototype
// true

// 假定 F 为构造函数，f 为 F 的实例对象
// 那么，f 的原型是 F.prototype
var f = new F();
Object.getPrototypeOf(f) === F.prototype
// true
```

## Object.create 方法

Object.create 方法用于生成新的对象，可以替代 new 命令。它接受一个原型对象作为参数，返回一个新对象，后者完全继承前者的属性。

```
var o1 = { p: 1 };
var o2 = Object.create(o1);

o2.p // 1
```

上面代码中，o1 是 o2 的原型对象，o2 继承了 o1 的属性。

Object.create 方法基本等同于下面的代码，如果老式浏览器不支持 Object.create 方法，可以用下面代码自己部署。

```
if (typeof Object.create !== "function") {
  Object.create = function (o) {
    function F() {}
    F.prototype = o;
    return new F();
  };
}
```

上面代码表示，Object.create 方法实质是新建一个构造函数 F，然后让 F 的 prototype 属性指向作为原型的对象 o，最后返回一个 F 的实例，从而实现让实例继承 o 的属性。

下面三种方式生成的新对象是等价的。

```
var o1 = Object.create({})
var o2 = Object.create(Object.prototype)
var o3 = new Object();
```

如果想要生成一个不继承任何属性（比如 toString 和 valueOf 方法）的对象，可以将 Object.create 的参数设为 null。

```
var o = Object.create(null);

o.valueOf()
// TypeError: Object [object Object] has no method 'valueOf'
```

上面代码表示，如果对象 o 的原型是 null，它就不具备一些定义在 Object.prototype 对象上面的属性，比如 valueOf 方法。

使用 Object.create 方法的时候，必须提供对象原型，否则会报错。

```
Object.create()
// TypeError: Object prototype may only be an Object or null
```

Object.create 方法生成的新对象，动态继承了原型。在原型上添加或修改任何方法，会立刻反映在新对象之上。

```
var o1 = { p: 1 };
var o2 = Object.create(o1);

o1.p = 2;
o2.p
// 2
```

上面代码表示，修改对象原型会影响到新生成的对象。

除了对象的原型，Object.create 方法还可以接受第二个参数，表示描述属性的 attributes 对象，跟用在 Object.defineProperties 方法的格式是一样的。它所描述的对象属性，会添加到新对象。

```
var o = Object.create(Object.prototype, {
  p1: { value: 123, enumerable: true },
  p2: { value: "abc", enumerable: true }
});

o.p1 // 123
o.p2 // "abc"
```

由于 Object.create 方法不使用构造函数，所以不能用 instanceof 运算符判断，对象是哪一个构造函数的实例。这时，可以使用下面的 isPrototypeOf 方法，判读原型是哪一个对象。

## isPrototypeOf 方法

isPrototypeOf 方法用来判断一个对象是否是另一个对象的原型。

```
var o1 = {};
var o2 = Object.create(o1);
var o3 = Object.create(o2);

o2.isPrototypeOf(o3) // true
o1.isPrototypeOf(o3) // true
```

上面代码表明，只要某个对象处在原型链上，isProtypeOf 都返回 true。