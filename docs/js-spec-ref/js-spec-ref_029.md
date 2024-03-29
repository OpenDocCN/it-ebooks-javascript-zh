# 4.3 继承

*   构造函数的继承
*   proto 属性
*   属性的继承
    *   对象的原生属性
    *   对象的继承属性
    *   获取所有属性
    *   对象的拷贝
    *   参考链接

## 构造函数的继承

JavaScript 通过构造函数生成实例对象，所以要实现对象的继承，就是要实现构造函数的继承。

假定有一个 Shape 构造函数。

```js
function Shape() {
  this.x = 0;
  this.y = 0;
}

Shape.prototype.move = function(x, y) {
  this.x += x;
  this.y += y;
  console.info('Shape moved.');
};
```

Rectangle 构造函数继承 Shape。

```js
function Rectangle() {
  Shape.call(this); // 调用父类构造函数
}

// 子类继承父类的方法
Rectangle.prototype = Object.create(Shape.prototype);
Rectangle.prototype.constructor = Rectangle;

var rect = new Rectangle();

rect instanceof Rectangle  // true
rect instanceof Shape  // true

rect.move(1, 1) // 'Shape moved.'
```

上面代码表示，构造函数的继承分成两部分，一部分是子类调用父类构造方法，另一部分是子类的原型指向父类的原型。

上面代码中，子类的原型是整体指向父类的原型。有时，只需单个方法指向原型，或修改父类同名方法，这时可以采用下面的写法。

```js
ClassB.prototype.print = function() {
  ClassA.prototype.print.call(this);
  // some code
}
```

上面代码中，子类 B 的 print 方法先调用父类 A 的 print 方法，再部署自己的代码。这就等于继承了父类 A 的 print 方法。

## **proto**属性

除了 IE 浏览器，其他浏览器都在 Object 对象的实例上，部署了一个非标准的**proto**属性（前后各两个下划线），指向该对象的原型对象，即构造函数的 prototype 属性。

```js
var o = new Object();

o.__proto__ === Object.prototype
// true

o.__proto__ === o.constructor.prototype
// true
```

上面代码首先新建了一个对象 o，它的`__proto__`属性，指向构造函数（Object 或 o.constructor）的 prototype 属性。所以，两者比较以后，返回 true。因此，获取一个实例对象的原型，直接的方法是`o.__proto__`，间接的方法是`o.constructor.prototype`。

获取对象原型还有第三种方法，就是使用 Object.getPrototypeOf 方法。该方法的参数是实例对象，返回值是实例对象的原型对象。因此，上面代码改写如下。

```js
var o = new Object();

o.__proto__ === Object.getPrototypeOf(o)
// true
```

因此，可以使用 Object.getPrototypeOf 方法，检查浏览器是否支持**proto**属性，毕竟这个属性不是标准属性。

```js
Object.getPrototypeOf({ __proto__: null }) === null
```

上面代码将一个对象的`__proto__`属性设为 null，然后使用 Object.getPrototypeOf 方法获取这个对象的原型，判断是否等于 null。如果当前环境支持`__proto__`属性，两者的比较结果应该是 true。否则，等式左边的值等于 Object。

有了`__proto__`属性，就可以很方便得设置实例对象的原型了。假定有三个对象 machine、vehicle 和 car，其中 machine 是 vehicle 的原型，vehicle 又是 car 的原型，只要两行代码就可以设置。

```js
vehicle.__proto__ = machine;
car.__proto__ = vehicle;
```

下面是一个实例，通过`__proto__`属性与 constructor.prototype 两种方法，分别读取定义在原型对象上的属性。

```js
Array.prototype.p = 'abc';
var a = new Array();

a.__proto__.p // abc
a.constructor.prototype.p // abc
```

显然，`__proto__`看上去更简洁一些。

因为这个属性目前还不是标准，所以不应该在生产代码中使用。我们这里用它，只是因为它可以帮助理解继承。

```js
var a = { x: 1};

var b = { __proto__: a};

b.x
// 1
```

上面代码中，b 对象本身并没有 x 属性，但是 JavaScript 引擎通过`__proto__`属性，找到它的原型对象 a，然后读取 a 的 x 属性。

原型对象自己的`__proto__`属性，也可以指向其他对象，从而一级一级地形成“原型链”（prototype chain）。

```js
var a = { x: 1 };

var b = { __proto__: a};

var c = { __proto__: b};

c.x
// 1
```

空对象的`__proto__`属性，默认指向 Object.prototype。

```js
var a = {};

a.__proto__ === Object.prototype
// true
```

通过构造函数生成实例对象时，实例对象的`__proto__`属性自动指向构造函数的 prototype 对象。

```js
var f = function (){};

var a = {};

f.prototype = a;

var o = new f();

o.__proto__ === a
// true
```

## 属性的继承

属性分成两种。一种是对象自身的原生属性，另一种是继承自原型的继承属性。

### 对象的原生属性

对象本身的所有属性，可以用 Object.getOwnPropertyNames 方法获得。

```js
Object.getOwnPropertyNames(Date)
// ["parse", "arguments", "UTC", "caller", "name", "prototype", "now", "length"]
```

对象本身的属性之中，有的是可以枚举的（enumerable），有的是不可以枚举的。只获取那些可以枚举的属性，使用 Object.keys 方法。

```js
Object.keys(Date)
// []
```

判断对象是否具有某个属性，使用 hasOwnProperty 方法。

```js
Date.hasOwnProperty('length')
// true

Date.hasOwnProperty('toString')
// false
```

### 对象的继承属性

用 Object.create 方法创造的对象，会继承所有原型对象的属性。

```js
var proto = { p1: 123 };
var o = Object.create(proto);

o.p1
// 123

o.hasOwnProperty("p1")
// false
```

### 获取所有属性

判断一个对象是否具有某个属性（不管是自身的还是继承的），使用 in 运算符。

```js
"length" in Date
// true

"toString" in Date
// true
```

获得对象的所有可枚举属性（不管是自身的还是继承的），可以使用 for-in 循环。

```js
var o1 = {p1:123};

var o2 = Object.create(o1,{
  p2: { value: "abc", enumerable: true }
});

for (p in o2) {console.info(p);}
// p2
// p1
```

为了在 for...in 循环中获得对象自身的属性，可以采用 hasOwnProperty 方法判断一下。

```js
for ( var name in object ) {
  if ( object.hasOwnProperty(name) ) {
    /* loop code */
  }
}
```

获得对象的所有属性（不管是自身的还是继承的，以及是否可枚举），可以使用下面的函数。

```js
function inheritedPropertyNames(obj) {
  var props = {};
  while(obj) {
    Object.getOwnPropertyNames(obj).forEach(function(p) {
      props[p] = true;
    });
    obj = Object.getPrototypeOf(obj);
  }
  return Object.getOwnPropertyNames(props);
}
```

用法如下：

```js
inheritedPropertyNames(Date)
// ["caller", "constructor", "toString", "UTC", "call", "parse", "prototype", "__defineSetter__", "__lookupSetter__", "length", "arguments", "bind", "__lookupGetter__", "isPrototypeOf", "toLocaleString", "propertyIsEnumerable", "valueOf", "apply", "__defineGetter__", "name", "now", "hasOwnProperty"]
```

## 对象的拷贝

如果要拷贝一个对象，需要做到下面两件事情。

*   确保拷贝后的对象，与原对象具有同样的 prototype 原型对象。
*   确保拷贝后的对象，与原对象具有同样的属性。

下面就是根据上面两点，编写的对象拷贝的函数。

```js
function copyObject(orig) {
  var copy = Object.create(Object.getPrototypeOf(orig));
  copyOwnPropertiesFrom(copy, orig);
  return copy;
}

function copyOwnPropertiesFrom(target, source) {
  Object
  .getOwnPropertyNames(source)
  .forEach(function(propKey) {
    var desc = Object.getOwnPropertyDescriptor(source, propKey);
    Object.defineProperty(target, propKey, desc);
  });
  return target;
}
```

## 参考链接

*   Dr. Axel Rauschmayer, [JavaScript properties: inheritance and enumerability](http://www.2ality.com/2011/07/js-properties.html)