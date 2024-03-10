# 3.1 Object 对象

*   概述
*   Object 对象的方法
    *   Object()
    *   Object.keys()，Object.getOwnPropertyNames()
    *   Object.observe()
    *   其他方法
    *   Object 实例对象的方法
        *   Object.prototype.valueOf()
        *   Object.prototype.toString()
        *   toString()的应用：判断数据类型
    *   对象的属性模型
        *   属性的 attributes 对象，Object.getOwnPropertyDescriptor()
        *   Object.defineProperty()，Object.defineProperties()
        *   可枚举性（enumerable）
        *   Object.getOwnPropertyNames()
        *   Object.prototype.propertyIsEnumerable()
        *   可配置性（configurable）
        *   可写性（writable）
        *   存取器（accessor）
    *   控制对象状态
        *   Object.preventExtensions 方法
        *   Object.isExtensible 方法
        *   Object.seal 方法
        *   Object.isSealed 方法
        *   Object.freeze 方法
        *   Object.isFrozen 方法
        *   局限性
    *   参考链接

## 概述

JavaScript 原生提供一个 Object 对象（注意起首的 O 是大写），所有其他对象都继承自这个对象。Object 本身也是一个构造函数，可以直接通过它来生成新对象。

```js
var o = new Object();
```

Object 作为构造函数使用时，可以接受一个参数。如果该参数是一个对象，则直接返回这个对象；如果是一个原始类型的值，则返回该值对应的包装对象。

```js
var o1 = {a:1};
var o2 = new Object(o1);
o1 === o2 // true

new Object(123) instanceof Number
// true
```

> 注意，通过 new Object() 的写法生成新对象，与字面量的写法 o = {} 是等价的。

与其他构造函数一样，如果要在 Object 对象上面部署一个方法，有两种做法。

（1）部署在 Object 对象本身

比如，在 Object 对象上面定义一个 print 方法，显示其他对象的内容。

```js
Object.print = function(o){ console.log(o) };

var o = new Object();

Object.print(o)
// Object
```

（2）部署在 Object.prototype 对象

所有构造函数都有一个 prototype 属性，指向一个原型对象。凡是定义在 Object.prototype 对象上面的属性和方法，将被所有实例对象共享。（关于 prototype 属性的详细解释，参见《面向对象编程》一章。）

```js
Object.prototype.print = function(){ console.log(this)};

var o = new Object();

o.print() // Object
```

上面代码在 Object.prototype 定义了一个 print 方法，然后生成一个 Object 的实例 o。o 直接继承了 Object.prototype 的属性和方法，可以在自身调用它们，也就是说，o 对象的 print 方法实质上是调用 Object.prototype.print 方法。。

可以看到，尽管上面两种写法的 print 方法功能相同，但是用法是不一样的，因此必须区分“构造函数的方法”和“实例对象的方法”。

## Object 对象的方法

### Object()

Object 本身当作工具方法使用时，可以将任意值转为对象。其中，原始类型的值转为对应的包装对象（参见《原始类型的包装对象》一节）。

```js
Object() // 返回一个空对象
Object(undefined) // 返回一个空对象
Object(null) // 返回一个空对象

Object(1) // 等同于 new Number(1)
Object('foo') // 等同于 new String('foo')
Object(true) // 等同于 new Boolean(true)

Object([]) // 返回原数组
Object({}) // 返回原对象
Object(function(){}) // 返回原函数
```

上面代码表示 Object 函数将各种值，转为对应的对象。

如果 Object 函数的参数是一个对象，它总是返回原对象。利用这一点，可以写一个判断变量是否为对象的函数。

```js
function isObject(value) {
    return value === Object(value);
}
```

### Object.keys()，Object.getOwnPropertyNames()

Object.keys 方法和 Object.getOwnPropertyNames 方法很相似，一般用来遍历对象的属性。它们的参数都是一个对象，都返回一个数组，该数组的成员都是对象自身的（而不是继承的）所有属性名。它们的区别在于，Object.keys 方法只返回可枚举的属性（关于可枚举性的详细解释见后文），Object.getOwnPropertyNames 方法还返回不可枚举的属性名。

```js
var o = {
    p1: 123,
    p2: 456
}; 

Object.keys(o)
// ["p1", "p2"]

Object.getOwnPropertyNames(o)
// ["p1", "p2"]
```

上面的代码表示，对于一般的对象来说，这两个方法返回的结果是一样的。只有涉及不可枚举属性时，才会有不一样的结果。

```js
var a = ["Hello", "World"];

Object.keys(a) 
// ["0", "1"]

Object.getOwnPropertyNames(a)
// ["0", "1", "length"]
```

上面代码中，数组的 length 属性是不可枚举的属性，所以只出现在 Object.getOwnPropertyNames 方法的返回结果中。

由于 JavaScript 没有提供计算对象属性个数的方法，所以可以用这两个方法代替。

```js
Object.keys(o).length
Object.getOwnPropertyNames(o).length
```

一般情况下，几乎总是使用 Object.keys 方法，遍历数组的属性。

### Object.observe()

Object.observe 方法用于观察对象属性的变化。

```js
var o = {};

Object.observe(o, function(changes) {
  changes.forEach(function(change) {
    console.log(change.type, change.name, change.oldValue);
  });
});

o.foo = 1; // add, 'foo', undefined
o.foo = 2; // update, 'foo', 1
delete o.foo; // delete, 'foo', 2
```

上面代码表示，通过 Object.observe 函数，对 o 对象指定回调函数。一旦 o 对象的属性出现任何变化，就会调用回调函数，回调函数通过一个参数对象读取 o 的属性变化的信息。

该方法非常新，只有 Chrome 浏览器的最新版本才部署。

### 其他方法

除了上面提到的方法，Object 还有不少其他方法，将在后文逐一详细介绍。

（1）对象属性模型的相关方法

*   Object.getOwnPropertyDescriptor()：获取某个属性的 attributes 对象。
*   Object.defineProperty()：通过 attributes 对象，定义某个属性。
*   Object.defineProperties()：通过 attributes 对象，定义多个属性。
*   Object.getOwnPropertyNames()：返回直接定义在某个对象上面的全部属性的名称。

（2）控制对象状态的方法

*   Object.preventExtensions()：防止对象扩展。
*   Object.isExtensible()：判断对象是否可扩展。
*   Object.seal()：禁止对象配置。
*   Object.isSealed()：判断一个对象是否可配置。
*   Object.freeze()：冻结一个对象。
*   Object.isFrozen()：判断一个对象是否被冻结。

（3）原型链相关方法

*   Object.create()：生成一个新对象，并该对象的原型。
*   Object.getPrototypeOf()：获取对象的 Prototype 对象。

## Object 实例对象的方法

除了 Object 对象本身的方法，还有不少方法是部署在 Object.prototype 对象上的，所有 Object 的实例对象都继承了这些方法。

Object 实例对象的方法，主要有以下六个。

*   valueOf()：返回当前对象对应的值。
*   toString()：返回当前对象对应的字符串形式。
*   toLocalString()：返回当前对象对应的本地字符串形式。
*   hasOwnProperty()：判断某个属性是否为当前对象自身的属性，还是继承自原型对象的属性。
*   isPrototypeOf()：判断当前对象是否为另一个对象的原型。
*   propertyIsEnumerable()：判断某个属性是否可枚举。

本节介绍前两个方法，其他方法将在后文相关章节介绍。

### Object.prototype.valueOf()

valueOf 方法的作用是返回一个对象的值，默认情况下返回对象本身。

```js
var o = new Object();

o.valueOf() === o // true
```

上面代码比较 o 的 valueOf 方法返回值与 o 本身，两者是一样的。

valueOf 方法的主要用途是，JavaScript 自动类型转换时会默认调用这个方法（详见上一章《数据类型转换》一节）。

```js
var o = new Object();

1 + o // "1[object Object]"
```

上面代码将对象 o 与数字 1 相加，这时 JavaScript 就会默认调用 valueOf()方法。所以，如果自定义 valueOf 方法，就可以得到想要的结果。

```js
var o = new Object();
o.valueOf = function (){return 2;};

1 + o // 3
```

上面代码自定义了 o 对象的 valueOf 方法，于是 1 + o 就得到了 3。这种方法就相当于用 o.valueOf 覆盖 Object.prototype.valueOf。

### Object.prototype.toString()

toString 方法的作用是返回一个对象的字符串形式。

```js
var o1 = new Object();
o1.toString() // "[object Object]"

var o2 = {a:1};
o2.toString() // "[object Object]"
```

上面代码表示，对于一个对象调用 toString 方法，会返回字符串[object Object]。

字符串[object Object]本身没有太大的用处，但是通过自定义 toString 方法，可以让对象在自动类型转换时，得到想要的字符串形式。

```js
var o = new Object();

o.toString = function (){ return 'hello' }; 

o + ' ' + 'world' // "hello world"
```

上面代码表示，当对象用于字符串加法时，会自动调用 toString 方法。由于自定义了 toString 方法，所以返回字符串 hello world。

数组、字符串和函数都分别部署了自己版本的 toString 方法。

```js
[1,2,3].toString() // "1,2,3"

'123'.toString() // "123"

(function (){return 123}).toString() // "function (){return 123}"
```

### toString()的应用：判断数据类型

toString 方法的主要用途是返回对象的字符串形式，除此之外，还有一个重要的作用，就是判断一个值的类型。

```js
var o = {};
o.toString() // "[object Object]"
```

上面代码调用空对象的 toString 方法，结果返回一个字符串“object Object”，其中第二个 Object 表示该值的准确类型。这是一个十分有用的判断数据类型的方法。

实例对象的 toString 方法，实际上是调用 Object.prototype.toString 方法。使用 call 方法，可以在任意值上调用 Object.prototype.toString 方法，从而帮助我们判断这个值的类型。不同数据类型的 toString 方法返回值如下：

*   数值：返回[object Number]。
*   字符串：返回[object String]。
*   布尔值：返回[object Boolean]。
*   undefined：返回[object Undefined]。
*   null：返回[object Null]。
*   对象：返回"[object " + 构造函数的名称 + "]" 。

```js
Object.prototype.toString.call(2) // "[object Number]"
Object.prototype.toString.call('') // "[object String]"
Object.prototype.toString.call(true) // "[object Boolean]"
Object.prototype.toString.call(undefined) // "[object Undefined]"
Object.prototype.toString.call(null) // "[object Null]"
Object.prototype.toString.call(Math) // "[object Math]"
Object.prototype.toString.call({}) // "[object Object]"
Object.prototype.toString.call([]) // "[object Array]"
```

可以利用这个特性，写出一个比 typeof 运算符更准确的类型判断函数。

```js
var type = function (o){
    var s = Object.prototype.toString.call(o);
        return s.match(/\[object (.*?)\]/)[1].toLowerCase();
};

type({}); // "object"
type([]); // "array"
type(5); // "number"
type(null); // "null"
type(); // "undefined"
type(/abcd/); // "regex"
type(new Date()); // "date"
```

在上面这个 type 函数的基础上，还可以加上专门判断某种类型数据的方法。

```js
['Null',
 'Undefined',
 'Object',
 'Array',
 'String',
 'Number',
 'Boolean',
 'Function',
 'RegExp',
 'Element',
 'NaN',
 'Infinite'
].forEach(function (t) {
    type['is' + t] = function (o) {
        return type(o) === t.toLowerCase();
    };
});

type.isObject({}); // true
type.isNumber(NaN); // false
type.isElement(document.createElement('div')); // true
type.isRegExp(/abc/); // true
```

## 对象的属性模型

ECMAScript 5 对于对象的属性，提出了一个精确的描述模型。

### 属性的 attributes 对象，Object.getOwnPropertyDescriptor()

在 JavaScript 内部，每个属性都有一个对应的 attributes 对象，保存该属性的一些元信息。使用 Object.getOwnPropertyDescriptor 方法，可以读取 attributes 对象。

```js
var o = { p: 'a' };

Object.getOwnPropertyDescriptor(o, 'p') 
// Object { value: "a", 
//         writable: true, 
//         enumerable: true, 
//         configurable: true
// }
```

上面代码表示，使用 Object.getOwnPropertyDescriptor 方法，读取 o 对象的 p 属性的 attributes 对象。

attributes 对象包含如下元信息：

*   value：表示该属性的值，默认为 undefined。

*   writable：表示该属性的值（value）是否可以改变，默认为 true。

*   enumerable： 表示该属性是否可枚举，默认为 true，也就是该属性会出现在 for...in 和 Object.keys()等操作中。

*   configurable：表示“可配置性”，默认为 true。如果设为 false，表示无法删除该属性，也不得改变 attributes 对象（value 属性除外），也就是 configurable 属性控制了 attributes 对象的可写性。

*   get：表示该属性的取值函数（getter），默认为 undefined。

*   set：表示该属性的存值函数（setter），默认为 undefined。

### Object.defineProperty()，Object.defineProperties()

Object.defineProperty 方法允许通过定义 attributes 对象，来定义或修改一个属性，然后返回修改后的对象。它的格式如下：

```js
Object.defineProperty(object, propertyName, attributesObject)
```

Object.defineProperty 方法接受三个参数，第一个是属性所在的对象，第二个是属性名（它应该是一个字符串），第三个是属性的描述对象。比如，新建一个 o 对象，并定义它的 p 属性，可以这样写：

```js
var o = Object.defineProperty({}, "p", {
        value: 123,
        writable: false,
        enumerable: true,
        configurable: false
});

o.p
// 123

o.p = 246;
o.p
// 123
// 因为 writable 为 false，所以无法改变该属性的值
```

如果一次性定义或修改多个属性，可以使用 Object.defineProperties 方法。

```js
var o = Object.defineProperties({}, {
        p1: { value: 123, enumerable: true },
        p2: { value: "abc", enumerable: true },
        p3: { get: function() { return this.p1+this.p2 },
              enumerable:true,
              configurable:true
        }
});

o.p1 // 123
o.p2 // "abc"
o.p3 // "123abc"
```

上面代码中的 p3 属性，定义了取值函数 get。这时需要注意的是，一旦定义了取值函数 get（或存值函数 set），就不能将 writable 设为 true，或者同时定义 value 属性，否则会报错。

```js
var o = {};

Object.defineProperty(o, "p", {
    value: 123, 
    get: function() { return 456; } 
});
// TypeError: Invalid property. 
// A property cannot both have accessors and be writable or have a value,
```

上面代码同时定义了 get 属性和 value 属性，结果就报错。

Object.defineProperty() 和 Object.defineProperties() 的第三个参数，是一个属性对象。它的 writable、configurable、enumerable 这三个属性的默认值都为 false。

writable 属性为 false，表示对应的属性的值将不得改写。

```js
var o = {};

Object.defineProperty(o, "p", {
    value: "bar"
});

o.p // bar

o.p = "foobar";
o.p // bar

Object.defineProperty(o, "p", {
    value: "foobar",
});
// TypeError: Cannot redefine property: p
```

上面代码由于 writable 属性默认为 false，导致无法对 p 属性重新赋值，但是不会报错（严格模式下会报错）。不过，如果再一次使用 Object.defineProperty 方法对 value 属性赋值，就会报错。

configurable 属性为 false，将无法删除该属性，也无法修改 attributes 对象（value 属性除外）。

```js
var o = {};

Object.defineProperty(o, "p", {
    value: "bar",
});

delete o.p
o.p // bar
```

上面代码中，由于 configurable 属性默认为 false，导致无法删除某个属性。

enumerable 属性为 false，表示对应的属性不会出现在 for...in 循环和 Object.keys 方法中。

```js
var o = {
    p1: 10,
    p2: 13,
};

Object.defineProperty(o, "p3", {
  value: 3,
});

for (var i in o) {
  console.log(i, o[i]);
}
// p1 10 
// p2 13
```

上面代码中，p3 属性是用 Object.defineProperty 方法定义的，由于 enumerable 属性默认为 false，所以不出现在 for...in 循环中。

### 可枚举性（enumerable）

可枚举性（enumerable）用来控制所描述的属性，是否将被包括在 for...in 循环之中。具体来说，如果一个属性的 enumerable 为 false，下面三个操作不会取到该属性。

*   for..in 循环
*   Object.keys 方法
*   JSON.stringify 方法

因此，enumerable 可以用来设置“秘密”属性。

```js
var o = {a:1, b:2};

o.c = 3;
Object.defineProperty(o, 'd', {
  value: 4,
  enumerable: false
});

o.d
// 4

for( var key in o ) console.log( o[key] ); 
// 1
// 2
// 3

Object.keys(o)  // ["a", "b", "c"]

JSON.stringify(o // => "{a:1,b:2,c:3}"
```

上面代码中，d 属性的 enumerable 为 false，所以一般的遍历操作都无法获取该属性，使得它有点像“秘密”属性，但还是可以直接获取它的值。

至于 for...in 循环和 Object.keys 方法的区别，在于前者包括对象继承自原型对象的属性，而后者只包括对象本身的属性。如果需要获取对象自身的所有属性，不管 enumerable 的值，可以使用 Object.getOwnPropertyNames 方法，详见下文。

考虑到 JSON.stringify 方法会排除 enumerable 为 false 的值，有时可以利用这一点，为对象添加注释信息。

```js
var car = {
  id: 123,
  color: red,
  owner: 12
};

var owner = {
  id: 12,
  name: Javi
};

Object.defineProperty( car, 'ownerOb', {value: owner} );
car.ownerOb // {id:12, name:Javi}

JSON.stringify(car) //  '{id: 123, color: "red", owner: 12}'
```

上面代码中，owner 对象作为注释，加入 car 对象。由于 ownerOb 属性的 enumerable 为 false，所以 JSON.stringify 最后正式输出 car 对象时，会忽略 ownerOb 属性。

### Object.getOwnPropertyNames()

Object.getOwnPropertyNames 方法返回直接定义在某个对象上面的全部属性的名称，而不管该属性是否可枚举。

```js
var o = Object.defineProperties({}, {
        p1: { value: 1, enumerable: true },
        p2: { value: 2, enumerable: false }
});

Object.getOwnPropertyNames(o)
// ["p1", "p2"]
```

一般来说，系统原生的属性（即非用户自定义的属性）都是不可枚举的。

```js
// 比如，数组实例自带 length 属性是不可枚举的
Object.keys([]) // []
Object.getOwnPropertyNames([]) // [ 'length' ]

// Object.prototype 对象的自带属性也都是不可枚举的
Object.keys(Object.prototype) // []
Object.getOwnPropertyNames(Object.prototype)
// ['hasOwnProperty',
//  'valueOf',
//  'constructor',
//  'toLocaleString',
//  'isPrototypeOf',
//  'propertyIsEnumerable',
//  'toString']
```

上面代码可以看到，数组的实例对象（[]）没有可枚举属性，不可枚举属性有 length；Object.prototype 对象也没有可枚举属性，但是有不少不可枚举属性。

### Object.prototype.propertyIsEnumerable()

对象实例的 propertyIsEnumerable 方法用来判断一个属性是否可枚举。

```js
var o = {};
o.p = 123;

o.propertyIsEnumerable("p") // true
o.propertyIsEnumerable("toString") // false
```

上面代码中，用户自定义的 p 属性是可枚举的，而继承自原型对象的 toString 属性是不可枚举的。

### 可配置性（configurable）

可配置性（configurable）决定了是否可以修改属性的描述对象。也就是说，当 configure 为 false 的时候，value、writable、enumerable 和 configurable 都不能被修改了。

```js
var o = Object.defineProperty({}, 'p', {
        value: 1,
        writable: false, 
        enumerable: false, 
        configurable: false
});

Object.defineProperty(o,'p', {value: 2})
// TypeError: Cannot redefine property: p

Object.defineProperty(o,'p', {writable: true})
// TypeError: Cannot redefine property: p

Object.defineProperty(o,'p', {enumerable: true})
// TypeError: Cannot redefine property: p

Object.defineProperties(o,'p',{configurable: true})
// TypeError: Cannot redefine property: p
```

上面代码首先生成对象 o，并且定义属性 p 的 configurable 为 false。然后，逐一改动 value、writable、enumerable、configurable，结果都报错。

需要注意的是，writable 只有在从 false 改为 true 会报错，从 true 改为 false 则是允许的。

```js
var o = Object.defineProperty({}, 'p', {
        writable: true
});

Object.defineProperty(o,'p', {writable: false})
// 修改成功
```

至于 value，只要 writable 和 configurable 有一个为 true，就可以改动。

```js
var o1 = Object.defineProperty({}, 'p', {
        value: 1,
        writable: true,
        configurable: false
});

Object.defineProperty(o1,'p', {value: 2})
// 修改成功

var o2 = Object.defineProperty({}, 'p', {
        value: 1,
        writable: false,
        configurable: true
});

Object.defineProperty(o2,'p', {value: 2}) 
// 修改成功
```

可配置性决定了一个变量是否可以被删除（delete）。

```js
var o = Object.defineProperties({}, {
        p1: { value: 1, configurable: true },
        p2: { value: 2, configurable: false }
});

delete o.p1 // true
delete o.p2 // false

o.p1 // undefined
o.p2 // 2
```

上面代码中的对象 o 有两个属性，p1 是可配置的，p2 是不可配置的。结果，p2 就无法删除。

需要注意的是，当使用 var 命令声明变量时，变量的 configurable 为 false。

```js
var a1 = 1;

Object.getOwnPropertyDescriptor(this,'a1')
// Object {
// value: 1, 
// writable: true, 
// enumerable: true, 
// configurable: false
// }
```

而不使用 var 命令声明变量时（或者使用属性赋值的方式声明变量），变量的可配置性为 true。

```js
a2 = 1;

Object.getOwnPropertyDescriptor(this,'a2')
// Object {
// value: 1, 
// writable: true, 
// enumerable: true, 
// configurable: true
// }

// 或者写成

this.a3 = 1;

Object.getOwnPropertyDescriptor(this,'a3')
// Object {
// value: 1, 
// writable: true, 
// enumerable: true, 
// configurable: true
// }
```

上面代码中的`this.a3 = 1`与`a3 = 1`是等价的写法。this 指的是当前的作用域，更多关于 this 的解释，参见《面向对象编程》一章。

这种差异意味着，如果一个变量是使用 var 命令生成的，就无法用 delete 命令删除。也就是说，delete 只能删除对象的属性。

```js
var a1 = 1;
a2 = 1;

delete a1 // false
delete a2 // true

a1 // 1
a2 // ReferenceError: a2 is not defined
```

### 可写性（writable）

可写性（writable）决定了属性的值（value）是否可以被改变。

```js
var o = {}; 

Object.defineProperty(o, "a", { value : 37, writable : false });

o.a // 37
o.a = 25;
o.a // 37
```

上面代码将 o 对象的 a 属性可写性设为 false，然后改变这个属性的值，就不会有任何效果。

这实际上将某个属性的值变成了常量。在 ES6 中，constant 命令可以起到这个作用，但在 ES5 中，只有通过 writable 达到同样目的。

这里需要注意的是，当对 a 属性重新赋值的时候，并不会抛出错误，只是静静地失败。但是，如果在严格模式下，这里就会抛出一个错误，即使是对 a 属性重新赋予一个同样的值。

关于可写性，还有一种特殊情况。就是如果原型对象的某个属性的可写性为 false，那么派生对象将无法自定义这个属性。

```js
var proto = Object.defineProperty({}, 'foo', {
    value: 'a',
    writable: false
});

var o = Object.create(proto);

o.foo = 'b';
o.foo // 'a'
```

上面代码中，对象 proto 的 foo 属性不可写，结果 proto 的派生对象 o，也不可以再自定义这个属性了。在严格模式下，这样做还会抛出一个错误。但是，有一个规避方法，就是通过覆盖 attributes 对象，绕过这个限制，原因是这种情况下，原型链会被完全忽视。

```js
Object.defineProperty(o, 'foo', { value: 'b' });

o.foo // 'b'
```

### 存取器（accessor）

除了直接定义以外，属性还可以用存取器（accessor）定义。其中，存值函数称为 setter，使用 set 命令；取值函数称为 getter，使用 get 命令。

```js
var o = {
  get p() {
    return "getter";
  },
  set p(value) {
    console.log("setter: "+value);
  }
}
```

上面代码中，o 对象内部的 get 和 set 命令，分别定义了 p 属性的取值函数和存值函数。定义了这两个函数之后，对 p 属性取值时，取值函数会自动调用；对 p 属性赋值时，存值函数会自动调用。

```js
o.p // getter
o.p = 123 // setter: 123
```

存取器往往用于，某个属性的值需要依赖对象内部数据的场合。

```js
var o ={
  $n : 5,
  get next(){return this.$n++ },
  set next(n) {
    if (n >= this.$n) this.$n = n;
    else throw "新的值必须大于当前值";
  }
};

o.next // 5

o.next = 10;
o.next //10
```

上面代码中，next 属性的存值函数和取值函数，都依赖于对内部属性$n 的操作。

下面是另一个存取器的例子。

```js
var user = {}
var nameValue = 'Joe';

Object.defineProperty(user, 'name', {
  get: function() { return nameValue },
  set: function(newValue) { nameValue = newValue; },
  configurable: true
});

user.name //Joe
user.name = 'Bob';
user.name //Bob
nameValue //Bob
```

上面代码使用存取器，将 user 对象 name 绑定在 nameValue 属性上了。

存取器也可以使用 Object.create 方法定义。

```js
var o = Object.create(Object.prototype, {
  foo: {
    get: function () {
      return 'getter';
    },
    set: function (value) {
      console.log('setter: '+value);
    }
  }
});
```

如果使用上面这种写法，属性 foo 必须定义一个属性描述对象。该对象的 get 和 set 属性，分别是 foo 的取值函数和存值函数。

利用存取器，可以实现数据对象与 DOM 对象的双向绑定。

```js
Object.defineProperty(user, 'name', {
  get: function() {
    return document.getElementById("foo").value;
  },
  set: function(newValue) {
    document.getElementById("foo").value = newValue;
  },
  configurable: true
});
```

上面代码使用存取函数，将 DOM 对象 foo 与数据对象 user 的 name 属性，实现了绑定。两者之中只要有一个对象发生变化，就能在另一个对象上实时反映出来。

## 控制对象状态

JavaScript 提供了三种方法，精确控制一个对象的读写状态，防止对象被改变。最弱一层的保护是 preventExtensions，其次是 seal，最强的 freeze。

### Object.preventExtensions 方法

Object.preventExtensions 方法可以使得一个对象无法再添加新的属性。

```js
var o = new Object();

Object.preventExtensions(o);

Object.defineProperty(o, "p", { value: "hello" });
// TypeError: Cannot define property:p, object is not extensible.

o.p = 1;
o.p // undefined
```

如果是在严格模式下，则会抛出一个错误。

```js
(function () { 
  'use strict'; 
  o.p = '1'
}());
// TypeError: Can't add property bar, object is not extensible
```

不过，对于使用了 preventExtensions 方法的对象，可以用 delete 命令删除它的现有属性。

```js
var o = new Object();
o.p = 1;

Object.preventExtensions(o);

delete o.p;
o.p // undefined
```

### Object.isExtensible 方法

Object.isExtensible 方法用于检查一个对象是否使用了 preventExtensions 方法。也就是说，该方法可以用来检查是否可以为一个对象添加属性。

```js
var o = new Object();

Object.isExtensible(o)
// true

Object.preventExtensions(o);
Object.isExtensible(o)
// false
```

上面代码新生成了一个 o 对象，对该对象使用 Object.isExtensible 方法，返回 true，表示可以添加新属性。对该对象使用 Object.preventExtensions 方法以后，再使用 Object.isExtensible 方法，返回 false，表示已经不能添加新属性了。

### Object.seal 方法

Object.seal 方法使得一个对象既无法添加新属性，也无法删除旧属性。

```js
var o = { p:"hello" };

Object.seal(o);

delete o.p;
o.p // "hello"

o.x = 'world';
o.x // undefined
```

Object.seal 还把现有属性的 attributes 对象的 configurable 属性设为 false，使得 attributes 对象不再能改变。

```js
var o = { p: 'a' };

// seal 方法之前
Object.getOwnPropertyDescriptor(o, 'p')
// Object {value: "a", writable: true, enumerable: true, configurable: true}

Object.seal(o);

// seal 方法之后
Object.getOwnPropertyDescriptor(o, 'p') 
// Object {value: "a", writable: true, enumerable: true, configurable: false}

Object.defineProperty(o, 'p', { enumerable: false })
// TypeError: Cannot redefine property: p
```

从上面代码可以看到，使用 seal 方法之后，attributes 对象的 configurable 就变成了 false，然后如果想改变 enumerable 就会报错。

可写性（writable）有点特别。如果 writable 为 false，使用 Object.seal 方法以后，将无法将其变成 true；但是，如果 writable 为 true，依然可以将其变成 false。

```js
var o1 = Object.defineProperty({}, 'p', {writable: false});
Object.seal(o1);
Object.defineProperty(o1,'p',{writable:true}) 
// Uncaught TypeError: Cannot redefine property: p 

var o2 = Object.defineProperty({}, 'p', {writable: true});
Object.seal(o2);
Object.defineProperty(o2,'p',{writable:false}) 

Object.getOwnPropertyDescriptor(o2, 'p')
/* { value: '',
  writable: false,
  enumerable: true,
  configurable: false } */
```

上面代码中，同样是使用了 Object.seal 方法，如果 writable 原为 false，改变这个设置将报错；如果原为 true，则不会有问题。

至于属性对象的 value 是否可改变，是由 writable 决定的。

```js
var o = { p: 'a' };
Object.seal(o);
o.p = 'b';
o.p // 'b'
```

上面代码中，Object.seal 方法对 p 属性的 value 无效，是因为此时 p 属性的 writable 为 true。

### Object.isSealed 方法

Object.isSealed 方法用于检查一个对象是否使用了 Object.seal 方法。

```js
var o = { p: 'a' };

Object.seal(o);
Object.isSealed(o) // true
```

另外，这时 isExtensible 方法也返回 false。

```js
var o = { p: 'a' };

Object.seal(o);
Object.isExtensible(o) // false
```

### Object.freeze 方法

Object.freeze 方法可以使得一个对象无法添加新属性、无法删除旧属性、也无法改变属性的值，使得这个对象实际上变成了常量。

```js
var o = {p:"hello"};

Object.freeze(o);

o.p = "world";
o.p // hello

o.t = "hello";
o.t // undefined
```

上面代码中，对现有属性重新赋值（o.p = "world"）或者添加一个新属性，并不会报错，只是默默地失败。但是，如果是在严格模式下，就会报错。

```js
var o = {p:"hello"};

Object.freeze(o);

// 对现有属性重新赋值
(function () { 'use strict'; o.p = "world";}())
// TypeError: Cannot assign to read only property 'p' of #<Object>

// 添加不存在的属性
(function () { 'use strict'; o.t = 123;}())
// TypeError: Can't add property t, object is not extensible
```

### Object.isFrozen 方法

Object.isFrozen 方法用于检查一个对象是否使用了 Object.freeze()方法。

```js
var o = {p:"hello"};

Object.freeze(o);
Object.isFrozen(o) // true
```

### 局限性

需要注意的是，使用上面这些方法锁定对象的可写性，但是依然可以通过改变该对象的原型对象，来为它增加属性。

```js
var o = new Object();

Object.preventExtensions(o);

var proto = Object.getPrototypeOf(o);

proto.t = "hello";

o.t
// hello
```

一种解决方案是，把原型也冻结住。

```js
var o = Object.seal(
            Object.create(Object.freeze({x:1}),
                {y: {value: 2, writable: true}})
);

Object.getPrototypeOf(o).t = "hello";
o.hello // undefined
```

## 参考链接

*   Axel Rauschmayer, [Protecting objects in JavaScript](http://www.2ality.com/2013/08/protecting-objects.html)
*   kangax, [Understanding delete](http://perfectionkills.com/understanding-delete/)
*   Jon Bretman, [Type Checking in JavaScript](http://techblog.badoo.com/blog/2013/11/01/type-checking-in-javascript/)
*   Cody Lindley, [Thinking About ECMAScript 5 Parts](http://tech.pro/tutorial/1671/thinking-about-ecmascript-5-parts)
*   Bjorn Tipling, [Advanced objects in JavaScript](http://bjorn.tipling.com/advanced-objects-in-javascript)
*   Javier Márquez, [Javascript properties are enumerable, writable and configurable](http://arqex.com/967/javascript-properties-enumerable-writable-configurable)
*   Sella Rafaeli, [Native JavaScript Data-Binding](http://www.sellarafaeli.com/blog/native_javascript_data_binding): 使用存取函数实现 model 与 view 的双向绑定