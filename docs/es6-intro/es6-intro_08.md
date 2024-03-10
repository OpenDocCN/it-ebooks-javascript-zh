# 对象的扩展

## 属性的简洁表示法

ES6 允许直接写入变量和函数，作为对象的属性和方法。这样的书写更加简洁。

```
function f( x, y ) {
  return { x, y };
}

// 等同于

function f( x, y ) {
  return { x: x, y: y };
}

```

上面是属性简写的例子，方法也可以简写。

```
var o = {
  method() {
    return "Hello!";
  }
};

// 等同于

var o = {
  method: function() {
    return "Hello!";
  }
};

```

下面是一个更实际的例子。

```
var Person = {

  name: '张三',

  //等同于 birth: birth
  birth,

  // 等同于 hello: function ()...
  hello() { console.log('我的名字是', this.name); }

};

```

这种写法用于函数的返回值，将会非常方便。

```
function getPoint() {
  var x = 1;
  var y = 10;

  return {x, y};
}

getPoint()
// {x:1, y:10}

```

## 属性名表达式

JavaScript 语言定义对象的属性，有两种方法。

```
// 方法一
obj.foo = true;

// 方法二
obj['a'+'bc'] = 123;

```

上面代码的方法一是直接用标识符作为属性名，方法二是用表达式作为属性名，这时要将表达式放在方括号之内。

但是，如果使用字面量方式定义对象（使用大括号），在 ES5 中只能使用方法一（标识符）定义属性。

```
var obj = {
  foo: true,
  abc: 123
};

```

ES6 允许字面量定义对象时，用方法二（表达式）作为对象的属性名，即把表达式放在方括号内。

```
let propKey = 'foo';

let obj = {
   [propKey]: true,
   ['a'+'bc']: 123
};

```

下面是另一个例子。

```
var lastWord = "last word";

var a = {
    "first word": "hello",
    [lastWord]: "world"
};

a["first word"] // "hello"
a[lastWord] // "world"
a["last word"] // "world"

```

表达式还可以用于定义方法名。

```
let obj = {
  ['h'+'ello']() {
    return 'hi';
  }
};

console.log(obj.hello()); // hi

```

## 方法的 name 属性

函数的 name 属性，返回函数名。ES6 为对象方法也添加了 name 属性。

```
var person = {
  sayName: function() {
    console.log(this.name);
  },
  get firstName() {
    return "Nicholas"
  }
}

person.sayName.name   // "sayName"
person.firstName.name // "get firstName"

```

上面代码中，方法的 name 属性返回函数名（即方法名）。如果使用了存值函数，则会在方法名前加上 get。如果是存值函数，方法名的前面会加上 set。

```
var doSomething = function() {
  // ...
};

console.log(doSomething.bind().name);   // "bound doSomething"

console.log((new Function()).name);     // "anonymous"

```

有两种特殊情况：bind 方法创造的函数，name 属性返回“bound”加上原函数的名字；Function 构造函数创造的函数，name 属性返回“anonymous”。

```
(new Function()).name // "anonymous"

var doSomething = function() {
  // ...
};
doSomething.bind().name // "bound doSomething"

```

## Object.is()

Object.is()用来比较两个值是否严格相等。它与严格比较运算符（===）的行为基本一致，不同之处只有两个：一是+0 不等于-0，二是 NaN 等于自身。

```
+0 === -0 //true
NaN === NaN // false

Object.is(+0, -0) // false
Object.is(NaN, NaN) // true

```

ES5 可以通过下面的代码，部署 Object.is()。

```
Object.defineProperty(Object, 'is', {
  value: function(x, y) {
    if (x === y) {
      // 针对+0 不等于 -0 的情况
      return x !== 0 || 1 / x === 1 / y;
    }
    // 针对 NaN 的情况
    return x !== x && y !== y;
  },
  configurable: true,
  enumerable: false,
  writable: true
});

```

## Object.assign()

Object.assign 方法用来将源对象（source）的所有可枚举属性，复制到目标对象（target）。它至少需要两个对象作为参数，第一个参数是目标对象，后面的参数都是源对象。只要有一个参数不是对象，就会抛出 TypeError 错误。

```
var target = { a: 1 };

var source1 = { b: 2 };
var source2 = { c: 3 };

Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}

```

注意，如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性。

```
var target = { a: 1, b: 1 };

var source1 = { b: 2, c: 2 };
var source2 = { c: 3 };

Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}

```

assign 方法有很多用处。

**（1）为对象添加属性**

```
class Point {
  constructor(x, y) {
    Object.assign(this, {x, y});
  }
}

```

上面方法通过 assign 方法，将 x 属性和 y 属性添加到 Point 类的对象实例。

**（2）为对象添加方法**

```
Object.assign(SomeClass.prototype, {
  someMethod(arg1, arg2) {
    ···
  },
  anotherMethod() {
    ···
  }
});

// 等同于下面的写法
SomeClass.prototype.someMethod = function (arg1, arg2) {
  ···
};
SomeClass.prototype.anotherMethod = function () {
  ···
};

```

上面代码使用了对象属性的简洁表示法，直接将两个函数放在大括号中，再使用 assign 方法添加到 SomeClass.prototype 之中。

**（3）克隆对象**

```
function clone(origin) {
  return Object.assign({}, origin);
}

```

上面代码将原始对象拷贝到一个空对象，就得到了原始对象的克隆。

不过，采用这种方法克隆，只能克隆原始对象自身的值，不能克隆它继承的值。如果想要保持继承链，可以采用下面的代码。

```
function clone(origin) {
  let originProto = Object.getPrototypeOf(origin);
  return Object.assign(Object.create(originProto), origin);
}

```

**（4）合并多个对象**

将多个对象合并到某个对象。

```
const merge =
  (target, ...sources) => Object.assign(target, ...sources);

```

如果希望合并后返回一个新对象，可以改写上面函数，对一个空对象合并。

```
const merge =
  (...sources) => Object.assign({}, ...sources);

```

**（5）为属性指定默认值**

```
const DEFAULTS = {
  logLevel: 0,
  outputFormat: 'html'
};

function processContent(options) {
  let options = Object.assign({}, DEFAULTS, options);
}

```

上面代码中，DEFAULTS 对象是默认值，options 对象是用户提供的参数。assign 方法将 DEFAULTS 和 options 合并成一个新对象，如果两者有同名属性，则 option 的属性值会覆盖 DEFAULTS 的属性值。

## **proto**属性，Object.setPrototypeOf()，Object.getPrototypeOf()

**（1）**proto**属性**

**proto**属性，用来读取或设置当前对象的 prototype 对象。该属性一度被正式写入 ES6 草案，但后来又被移除。目前，所有浏览器（包括 IE11）都部署了这个属性。

```
// es6 的写法

var obj = {
  __proto__: someOtherObj,
  method: function() { ... }
}

// es5 的写法

var obj = Object.create(someOtherObj);
obj.method = function() { ... }

```

有了这个属性，实际上已经不需要通过 Object.create()来生成新对象了。

**（2）Object.setPrototypeOf()**

Object.setPrototypeOf 方法的作用与**proto**相同，用来设置一个对象的 prototype 对象。它是 ES6 正式推荐的设置原型对象的方法。

```
// 格式
Object.setPrototypeOf(object, prototype)

// 用法
var o = Object.setPrototypeOf({}, null);

```

该方法等同于下面的函数。

```
function (obj, proto) {
  obj.__proto__ = proto;
  return obj;
}

```

**（3）Object.getPrototypeOf()**

该方法与 setPrototypeOf 方法配套，用于读取一个对象的 prototype 对象。

```
Object.getPrototypeOf(obj)

```

## Symbol

### 概述

在 ES5 中，对象的属性名都是字符串，这容易造成属性名的冲突。比如，你使用了一个他人提供的对象，但又想为这个对象添加新的方法，新方法的名字有可能与现有方法产生冲突。如果有一种机制，保证每个属性的名字都是独一无二的就好了，这样就从根本上防止属性名的冲突。这就是 ES6 引入 Symbol 的原因。

ES6 引入了一种新的原始数据类型 Symbol，表示独一无二的 ID。它通过 Symbol 函数生成。这就是说，对象的属性名现在可以有两种类型，一种是原来就有的字符串，另一种就是新增的 Symbol 类型。凡是属性名属于 Symbol 类型，就都是独一无二的，可以保证不会与其他属性名产生冲突。

```
let s = Symbol();

typeof s
// "symbol"

```

上面代码中，变量 s 就是一个独一无二的 ID。typeof 运算符的结果，表明变量 s 是 Symbol 数据类型，而不是字符串之类的其他类型。

注意，Symbol 函数前不能使用 new 命令，否则会报错。这是因为生成的 Symbol 是一个原始类型的值，不是对象。也就是说，由于 Symbol 值不是对象，所以不能添加属性。基本上，它是一种类似于字符串的数据类型。

Symbol 函数可以接受一个字符串作为参数，表示对 Symbol 实例的描述，主要是为了在控制台显示，或者转为字符串时，比较容易区分。

```
var s1 = Symbol('foo');
var s2 = Symbol('bar');

s1 // Symbol(foo)
s2 // Symbol(bar)

s1.toString() // "Symbol(foo)"
s2.toString() // "Symbol(bar)"

```

上面代码中，s1 和 s2 是两个 Symbol 值。如果不加参数，它们在控制台的输出都是`Symbol()`，不利于区分。有了参数以后，就等于为它们加上了描述，输出的时候就能够分清，到底是哪一个值。

注意，Symbol 函数的参数只是表示对当前 Symbol 类型的值的描述，因此相同参数的 Symbol 函数的返回值是不相等的。

```
// 没有参数的情况
var s1 = Symbol();
var s2 = Symbol();

s1 === s2 // false

// 有参数的情况
var s1 = Symbol("foo");
var s2 = Symbol("foo");

s1 === s2 // false

```

上面代码中，s1 和 s2 都是 Symbol 函数的返回值，而且参数相同，但是它们是不相等的。

Symbol 类型的值不能与其他类型的值进行运算，会报错。

```
var sym = Symbol('My symbol');

"your symbol is " + sym
// TypeError: can't convert symbol to string
`your symbol is ${sym}`
// TypeError: can't convert symbol to string

```

但是，Symbol 类型的值可以转为字符串。

```
var sym = Symbol('My symbol');

String(sym) // 'Symbol(My symbol)'
sym.toString() // 'Symbol(My symbol)'

```

### 作为属性名的 Symbol

由于每一个 Symbol 值都是不相等的，这意味着 Symbol 值可以作为标识符，用于对象的属性名，就能保证不会出现同名的属性。这对于一个对象由多个模块构成的情况非常有用，能防止某一个键被不小心改写或覆盖。

```
var mySymbol = Symbol();

// 第一种写法
var a = {};
a[mySymbol] = 'Hello!';

// 第二种写法
var a = {
  [mySymbol]: 123
};

// 第三种写法
var a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello!' });

// 以上写法都得到同样结果
a[mySymbol] // "Hello!"

```

上面代码通过方括号结构和 Object.defineProperty，将对象的属性名指定为一个 Symbol 值。

注意，Symbol 值作为对象属性名时，不能用点运算符。

```
var mySymbol = Symbol();
var a = {};

a.mySymbol = 'Hello!';
a[mySymbol] // undefined
a['mySymbol'] // "Hello!"

```

上面代码中，因为点运算符后面总是字符串，所以不会读取 mySymbol 作为标识名所指代的那个值，导致 a 的属性名实际上是一个字符串，而不是一个 Symbol 值。

同理，在对象的内部，使用 Symbol 值定义属性时，Symbol 值必须放在方括号之中。

```
let s = Symbol();

let obj = {
  [s]: function (arg) { ... }
};

objs;

```

上面代码中，如果 s 不放在方括号中，该属性的键名就是字符串 s，而不是 s 所代表的那个 Symbol 值。

采用增强的对象写法，上面代码的 obj 对象可以写得更简洁一些。

```
let obj = {
  s { ... }
};

```

Symbol 类型还可以用于定义一组常量，保证这组常量的值都是不相等的。

```
log.levels = {
    DEBUG: Symbol('debug'),
    INFO: Symbol('info'),
    WARN: Symbol('warn'),
};
log(log.levels.DEBUG, 'debug message');
log(log.levels.INFO, 'info message');

```

还有一点需要注意，Symbol 值作为属性名时，该属性还是公开属性，不是私有属性。

### 属性名的遍历

Symbol 作为属性名，该属性不会出现在 for...in、for...of 循环中，也不会被`Object.keys()`、`Object.getOwnPropertyNames()`返回。但是，它也不是私有属性，有一个 Object.getOwnPropertySymbols 方法，可以获取指定对象的所有 Symbol 属性名。

Object.getOwnPropertySymbols 方法返回一个数组，成员是当前对象的所有用作属性名的 Symbol 值。

```
var obj = {};
var a = Symbol('a');
var b = Symbol.for('b');

obj[a] = 'Hello';
obj[b] = 'World';

var objectSymbols = Object.getOwnPropertySymbols(obj);

objectSymbols
// [Symbol(a), Symbol(b)]

```

下面是另一个例子，Object.getOwnPropertySymbols 方法与 for...in 循环、Object.getOwnPropertyNames 方法进行对比的例子。

```
var obj = {};

var foo = Symbol("foo");

Object.defineProperty(obj, foo, {
  value: "foobar",
});

for (var i in obj) {
  console.log(i); // 无输出
}

Object.getOwnPropertyNames(obj)
// []

Object.getOwnPropertySymbols(obj)
// [Symbol(foo)]

```

上面代码中，使用 Object.getOwnPropertyNames 方法得不到 Symbol 属性名，需要使用 Object.getOwnPropertySymbols 方法。

另一个新的 API，Reflect.ownKeys 方法可以返回所有类型的键名，包括常规键名和 Symbol 键名。

```
let obj = {
  [Symbol('my_key')]: 1,
  enum: 2,
  nonEnum: 3
};

Reflect.ownKeys(obj)
// [Symbol(my_key), 'enum', 'nonEnum']

```

由于以 Symbol 值作为名称的属性，不会被常规方法遍历得到。我们可以利用这个特性，为对象定义一些非私有的、但又希望只用于内部的方法。

```
var size = Symbol('size');

class Collection {
  constructor() {
    this[size] = 0;
  }

  add(item) {
    this[this[size]] = item;
    this[size]++;
  }

  static sizeOf(instance) {
    return instance[size];
  }
}

var x = new Collection();
Collection.sizeOf(x) // 0

x.add('foo');
Collection.sizeOf(x) // 1

Object.keys(x) // ['0']
Object.getOwnPropertyNames(x) // ['0']
Object.getOwnPropertySymbols(x) // [Symbol(size)]

```

上面代码中，对象 x 的 size 属性是一个 Symbol 值，所以`Object.keys(x)`、`Object.getOwnPropertyNames(x)`都无法获取它。这就造成了一种非私有的内部方法的效果。

### Symbol.for()，Symbol.keyFor()

有时，我们希望重新使用同一个 Symbol 值，`Symbol.for`方法可以做到这一点。它接受一个字符串作为参数，然后搜索有没有以该参数作为名称的 Symbol 值。如果有，就返回这个 Symbol 值，否则就新建并返回一个以该字符串为名称的 Symbol 值。

```
var s1 = Symbol.for('foo');
var s2 = Symbol.for('foo');

s1 === s2 // true

```

上面代码中，s1 和 s2 都是 Symbol 值，但是它们都是同样参数的`Symbol.for`方法生成的，所以实际上是同一个值。

`Symbol.for()`与`Symbol()`这两种写法，都会生成新的 Symbol。它们的区别是，前者会被登记在全局环境中供搜索，后者不会。`Symbol.for()`不会每次调用就返回一个新的 Symbol 类型的值，而是会先检查给定的 key 是否已经存在，如果不存在才会新建一个值。比如，如果你调用`Symbol.for("cat")`30 次，每次都会返回同一个 Symbol 值，但是调用`Symbol("cat")`30 次，会返回 30 个不同的 Symbol 值。

```
Symbol.for("bar") === Symbol.for("bar")
// true

Symbol("bar") === Symbol("bar")
// false

```

上面代码中，由于`Symbol()`写法没有登记机制，所以每次调用都会返回一个不同的值。

Symbol.keyFor 方法返回一个已登记的 Symbol 类型值的 key。

```
var s1 = Symbol.for("foo");
Symbol.keyFor(s1) // "foo"

var s2 = Symbol("foo");
Symbol.keyFor(s2) // undefined

```

上面代码中，变量 s2 属于未登记的 Symbol 值，所以返回 undefined。

需要注意的是，`Symbol.for`为 Symbol 值登记的名字，是全局环境的，可以在不同的 iframe 或 service worker 中取到同一个值。

```
iframe = document.createElement('iframe');
iframe.src = String(window.location);
document.body.appendChild(iframe);

iframe.contentWindow.Symbol.for('foo') === Symbol.for('foo')
// true

```

上面代码中，iframe 窗口生成的 Symbol 值，可以在主页面得到。

### 内置的 Symbol 值

除了定义自己使用的 Symbol 值以外，ES6 还提供一些内置的 Symbol 值，指向语言内部使用的方法。

**（1）Symbol.hasInstance**

对象的 Symbol.hasInstance 属性，指向一个内部方法。该对象使用 instanceof 运算符时，会调用这个方法，判断该对象是否为某个构造函数的实例。比如，`foo instanceof Foo`在语言内部，实际调用的是`FooSymbol.hasInstance`。

**（2）Symbol.isConcatSpreadable**

对象的 Symbol.isConcatSpreadable 属性，指向一个方法。该对象使用 Array.prototype.concat()时，会调用这个方法，返回一个布尔值，表示该对象是否可以扩展成数组。

**（3）Symbol.isRegExp**

对象的 Symbol.isRegExp 属性，指向一个方法。该对象被用作正则表达式时，会调用这个方法，返回一个布尔值，表示该对象是否为一个正则对象。

**（4）Symbol.match**

对象的 Symbol.match 属性，指向一个函数。当执行`str.match(myObject)`时，如果该属性存在，会调用它，返回该方法的返回值。

**（5）Symbol.iterator**

对象的 Symbol.iterator 属性，指向该对象的默认遍历器方法，即该对象进行 for...of 循环时，会调用这个方法，返回该对象的默认遍历器，详细介绍参见《Iterator 和 for...of 循环》一章。

```
class Collection {
  *[Symbol.iterator]() {
    let i = 0;
    while(this[i] !== undefined) {
      yield this[i];
      ++i;
    }
  }

}

let myCollection = new Collection();
myCollection[0] = 1;
myCollection[1] = 2;

for(let value of myCollection) {
  console.log(value);
}
// 1
// 2

```

**（6）Symbol.toPrimitive**

对象的 Symbol.toPrimitive 属性，指向一个方法。该对象被转为原始类型的值时，会调用这个方法，返回该对象对应的原始类型值。

**（7）Symbol.toStringTag**

对象的 Symbol.toStringTag 属性，指向一个方法。在该对象上面调用`Object.prototype.toString`方法时，如果这个属性存在，它的返回值会出现在 toString 方法返回的字符串之中，表示对象的类型。也就是说，这个属性可以用来定制`[object Object]`或`[object Array]`中 object 后面的那个字符串。

```
class Collection {
  get [Symbol.toStringTag]() {
    return 'xxx';
  }
}
var x = new Collection();
Object.prototype.toString.call(x) // "[object xxx]"

```

**（8）Symbol.unscopables**

对象的 Symbol.unscopables 属性，指向一个对象。该对象指定了使用 with 关键字时，那些属性会被 with 环境排除。

```
Array.prototype[Symbol.unscopables]
// {
//   copyWithin: true,
//   entries: true,
//   fill: true,
//   find: true,
//   findIndex: true,
//   keys: true
// }

Object.keys(Array.prototype[Symbol.unscopables])
// ['copyWithin', 'entries', 'fill', 'find', 'findIndex', 'keys']

```

上面代码说明，数组有 6 个属性，会被 with 命令排除。

```
// 没有 unscopables 时
class MyClass {
  foo() { return 1; }
}

var foo = function () { return 2; };

with (MyClass.prototype) {
    foo(); // 1
}

// 有 unscopables 时
class MyClass {
  foo() { return 1; }
  get [Symbol.unscopables]() {
    return { foo: true };
  }
}

var foo = function () { return 2; };

with (MyClass.prototype) {
  foo(); // 2
}

```

## Proxy

### 概述

Proxy 用于修改某些操作的默认行为，等同于在语言层面做出修改，所以属于一种“元编程”（meta programming），即对编程语言进行编程。

Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。Proxy 这个词的原意是代理，用在这里表示由它来“代理”某些操作，可以译为“代理器”。

```
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

上面代码对一个空对象架设了一层拦截，重定义了属性的读取（get）和设置（set）行为。这里暂时不解释具体的语法，只看运行结果。对设置了拦截行为的对象 obj，去读写它的属性，就会得到下面的结果。

```
obj.count = 1
//  setting count!
++obj.count
//  getting count!
//  setting count!
//  2

```

上面代码说明，Proxy 实际上重载（overload）了点运算符，即用自己的定义覆盖了语言的原始定义。

ES6 原生提供 Proxy 构造函数，用来生成 Proxy 实例。

```
var proxy = new Proxy(target, handler)

```

Proxy 对象的所用用法，都是上面这种形式，不同的只是 handler 参数的写法。其中，`new Proxy()`表示生成一个 Proxy 实例，target 参数表示所要拦截的目标对象，handler 参数也是一个对象，用来定制拦截行为。

下面是另一个拦截读取属性行为的例子。

```
var proxy = new Proxy({}, {
  get: function(target, property) {
    return 35;
  }
});

proxy.time // 35
proxy.name // 35
proxy.title // 35

```

上面代码中，作为构造函数，Proxy 接受两个参数。第一个参数是所要代理的目标对象（上例是一个空对象），即如果没有 Proxy 的介入，操作原来要访问的就是这个对象；第二个参数是一个配置对象，对于每一个被代理的操作，需要提供一个对应的处理函数，该函数将拦截对应的操作。比如，上面代码中，配置对象有一个 get 方法，用来拦截对目标对象属性的访问请求。get 方法的两个参数分别是目标对象和所要访问的属性。可以看到，由于拦截函数总是返回 35，所以访问任何属性都得到 35。

注意，要使得 Proxy 起作用，必须针对 Proxy 实例（上例是 proxy 对象）进行操作，而不是针对目标对象（上例是空对象）进行操作。

一个技巧是将 Proxy 对象，设置到`object.proxy`属性，从而可以在 object 对象上调用。

```
var object = { proxy: new Proxy(target, handler) }

```

Proxy 实例也可以作为其他对象的原型对象。

```
var proxy = new Proxy({}, {
  get: function(target, property) {
    return 35;
  }
});

let obj = Object.create(proxy);
obj.time // 35

```

上面代码中，proxy 对象是 obj 对象的原型，obj 对象本身并没有 time 属性，所有根据原型链，会在 proxy 对象上读取该属性，导致被拦截。

同一个拦截器函数，可以设置拦截多个操作。

```
var handler = {
  get: function(target, name) {
    if (name === 'prototype') return Object.prototype;
    return 'Hello, '+ name;
  },
  apply: function(target, thisBinding, args) { return args[0]; },
  construct: function(target, args) { return args[1]; }
};

var fproxy = new Proxy(function(x,y) {
  return x+y;
},  handler);

fproxy(1,2); // 1
new fproxy(1,2); // 2
fproxy.prototype; // Object.prototype
fproxy.foo; // 'Hello, foo'

```

下面是 Proxy 支持的拦截操作一览。

对于可以设置、但没有设置拦截的操作，则直接落在目标对象上，按照原先的方式产生结果。

**（1）get(target, propKey, receiver)**

拦截对象属性的读取，比如`proxy.foo`和`proxy['foo']`，返回类型不限。最后一个参数 receiver 可选，当 target 对象设置了 propKey 属性的 get 函数时，receiver 对象会绑定 get 函数的 this 对象。

**（2）set(target, propKey, value, receiver)**

拦截对象属性的设置，比如`proxy.foo = v`或`proxy['foo'] = v`，返回一个布尔值。

**（3）has(target, propKey)**

拦截`propKey in proxy`的操作，返回一个布尔值。

**（4）deleteProperty(target, propKey)**

拦截`delete proxy[propKey]`的操作，返回一个布尔值。

**（5）enumerate(target)**

拦截`for (var x in proxy)`，返回一个遍历器。

**6）hasOwn(target, propKey)**

拦截`proxy.hasOwnProperty('foo')`，返回一个布尔值。

**（7）ownKeys(target)**

拦截`Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`，返回一个数组。该方法返回对象所有自身的属性，而`Object.keys()`仅返回对象可遍历的属性。

**（8）getOwnPropertyDescriptor(target, propKey)**

拦截`Object.getOwnPropertyDescriptor(proxy, propKey)`，返回属性的描述对象。

**（9）defineProperty(target, propKey, propDesc)**

拦截`Object.defineProperty(proxy, propKey, propDesc）`、`Object.defineProperties(proxy, propDescs)`，返回一个布尔值。

**（10）preventExtensions(target)**

拦截`Object.preventExtensions(proxy)`，返回一个布尔值。

**（11）getPrototypeOf(target)**

拦截`Object.getPrototypeOf(proxy)`，返回一个对象。

**（12）isExtensible(target)**

拦截`Object.isExtensible(proxy)`，返回一个布尔值。

**（13）setPrototypeOf(target, proto)**

拦截`Object.setPrototypeOf(proxy, proto)`，返回一个布尔值。

如果目标对象是函数，那么还有两种额外操作可以拦截。

**（14）apply(target, object, args)**

拦截 Proxy 实例作为函数调用的操作，比如`proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`。

**（15）construct(target, args, proxy)**

拦截 Proxy 实例作为构造函数调用的操作，比如 new proxy(...args)。

下面是其中几个重要拦截方法的详细介绍。

### get()

get 方法用于拦截某个属性的读取操作。上文已经有一个例子，下面是另一个拦截读取操作的例子。

```
var person = {
  name: "张三"
};

var proxy = new Proxy(person, {
  get: function(target, property) {
    if (property in target) {
      return target[property];
    } else {
      throw new ReferenceError("Property \"" + property + "\" does not exist.");
    }
  }
});

proxy.name // "张三"
proxy.age // 抛出一个错误

```

上面代码表示，如果访问目标对象不存在的属性，会抛出一个错误。如果没有这个拦截函数，访问不存在的属性，只会返回 undefined。

利用 proxy，可以将读取属性的操作（get），转变为执行某个函数。

```
var pipe = (function () {
  var pipe;
  return function (value) {
    pipe = [];
    return new Proxy({}, {
      get: function (pipeObject, fnName) {
        if (fnName == "get") {
          return pipe.reduce(function (val, fn) {
            return fn(val);
          }, value);
        }
        pipe.push(window[fnName]);
        return pipeObject;
      }
    });
  }
}());

var double = function (n) { return n*2 };
var pow = function (n) { return n*n };
var reverseInt = function (n) { return n.toString().split('').reverse().join('')|0 };

pipe(3) . double . pow . reverseInt . get
// 63

```

上面代码设置 Proxy 以后，达到了将函数名链式使用的效果。

### set()

set 方法用来拦截某个属性的赋值操作。假定 Person 对象有一个 age 属性，该属性应该是一个不大于 200 的整数，那么可以使用 Proxy 对象保证 age 的属性值符合要求。

```
let validator = {
  set: function(obj, prop, value) {
    if (prop === 'age') {
      if (!Number.isInteger(value)) {
        throw new TypeError('The age is not an integer');
      }
      if (value > 200) {
        throw new RangeError('The age seems invalid');
      }
    }

    // 对于 age 以外的属性，直接保存
    obj[prop] = value;
  }
};

let person = new Proxy({}, validator);

person.age = 100;

person.age // 100
person.age = 'young' // 报错
person.age = 300 // 报错

```

上面代码中，由于设置了存值函数 set，任何不符合要求的 age 属性赋值，都会抛出一个错误。利用 set 方法，还可以数据绑定，即每当对象发生变化时，会自动更新 DOM。

### apply()

apply 方法拦截函数的调用、call 和 apply 操作。

```
var target = function () { return 'I am the target'; };
var handler = {
  apply: function (receiver, ...args) {
    return 'I am the proxy';
  }
};

var p = new Proxy(target, handler);

p() === 'I am the proxy';
// true

```

上面代码中，变量 p 是 Proxy 的实例，当它作为函数调用时（p()），就会被 apply 方法拦截，返回一个字符串。

### ownKeys()

ownKeys 方法用来拦截 Object.keys()操作。

```
let target = {};

let handler = {
  ownKeys(target) {
    return ['hello', 'world'];
  }
};

let proxy = new Proxy(target, handler);

Object.keys(proxy)
// [ 'hello', 'world' ]

```

上面代码拦截了对于 target 对象的 Object.keys()操作，返回预先设定的数组。

### Proxy.revocable()

Proxy.revocable 方法返回一个可取消的 Proxy 实例。

```
let target = {};
let handler = {};

let {proxy, revoke} = Proxy.revocable(target, handler);

proxy.foo = 123;
proxy.foo // 123

revoke();
proxy.foo // TypeError: Revoked

```

Proxy.revocable 方法返回一个对象，该对象的 proxy 属性是 Proxy 实例，revoke 属性是一个函数，可以取消 Proxy 实例。上面代码中，当执行 revoke 函数之后，再访问 Proxy 实例，就会抛出一个错误。

## Reflect

### 概述

Reflect 对象与 Proxy 对象一样，也是 ES6 为了操作对象而提供的新 API。Reflect 对象的设计目的有这样几个。

（1） 将 Object 对象的一些明显属于语言层面的方法，放到 Reflect 对象上。现阶段，某些方法同时在 Object 和 Reflect 对象上部署，未来的新方法将只部署在 Reflect 对象上。

（2） 修改某些 Object 方法的返回结果，让其变得更合理。比如，`Object.defineProperty(obj, name, desc)`在无法定义属性时，会抛出一个错误，而`Reflect.defineProperty(obj, name, desc)`则会返回 false。

（3） 让 Object 操作都变成函数行为。某些 Object 操作是命令式，比如`name in obj`和`delete obj[name]`，而`Reflect.has(obj, name)`和`Reflect.deleteProperty(obj, name)`让它们变成了函数行为。

（4）Reflect 对象的方法与 Proxy 对象的方法一一对应，只要是 Proxy 对象的方法，就能在 Reflect 对象上找到对应的方法。这就让 Proxy 对象可以方便地调用对应的 Reflect 方法，完成默认行为，作为修改行为的基础。

```
Proxy(target, {
  set: function(target, name, value, receiver) {
    var success = Reflect.set(target,name, value, receiver);
    if (success) {
      log('property '+name+' on '+target+' set to '+value);
    }
    return success;
  }
});

```

上面代码中，Proxy 方法拦截 target 对象的属性赋值行为。它采用 Reflect.set 方法将值赋值给对象的属性，然后再部署额外的功能。

下面是 get 方法的例子。

```
var loggedObj = new Proxy(obj, {
  get: function(target, name) {
    console.log("get", target, name);
    return Reflect.get(target, name);
  }
});

```

### 方法

Reflect 对象的方法清单如下。

*   Reflect.getOwnPropertyDescriptor(target,name)
*   Reflect.defineProperty(target,name,desc)
*   Reflect.getOwnPropertyNames(target)
*   Reflect.getPrototypeOf(target)
*   Reflect.deleteProperty(target,name)
*   Reflect.enumerate(target)
*   Reflect.freeze(target)
*   Reflect.seal(target)
*   Reflect.preventExtensions(target)
*   Reflect.isFrozen(target)
*   Reflect.isSealed(target)
*   Reflect.isExtensible(target)
*   Reflect.has(target,name)
*   Reflect.hasOwn(target,name)
*   Reflect.keys(target)
*   Reflect.get(target,name,receiver)
*   Reflect.set(target,name,value,receiver)
*   Reflect.apply(target,thisArg,args)
*   Reflect.construct(target,args)

上面这些方法的作用，大部分与 Object 对象的同名方法的作用都是相同的。下面是对其中几个方法的解释。

（1）Reflect.get(target,name,receiver)

查找并返回 target 对象的 name 属性，如果没有该属性，则返回 undefined。

如果 name 属性部署了读取函数，则读取函数的 this 绑定 receiver。

```
var obj = {
  get foo() { return this.bar(); },
  bar: function() { ... }
}

// 下面语句会让 this.bar()
// 变成调用 wrapper.bar()
Reflect.get(obj, "foo", wrapper);

```

（2）Reflect.set(target, name, value, receiver)

设置 target 对象的 name 属性等于 value。如果 name 属性设置了赋值函数，则赋值函数的 this 绑定 receiver。

（3）Reflect.has(obj, name)

等同于`name in obj`。

（4）Reflect.deleteProperty(obj, name)

等同于`delete obj[name]`。

（5）Reflect.construct(target, args)

等同于`new target(...args)`，这提供了一种不使用 new，来调用构造函数的方法。

（6）Reflect.getPrototypeOf(obj)

读取对象的**proto**属性，等同于`Object.getPrototypeOf(obj)`。

（7）Reflect.setPrototypeOf(obj, newProto)

设置对象的**proto**属性。注意，Object 对象没有对应这个方法的方法。

（8）Reflect.apply(fun,thisArg,args)

等同于`Function.prototype.apply.call(fun,thisArg,args)`。一般来说，如果要绑定一个函数的 this 对象，可以这样写`fn.apply(obj, args)`，但是如果函数定义了自己的 apply 方法，就只能写成`Function.prototype.apply.call(fn, obj, args)`，采用 Reflect 对象可以简化这种操作。

另外，需要注意的是，Reflect.set()、Reflect.defineProperty()、Reflect.freeze()、Reflect.seal()和 Reflect.preventExtensions()返回一个布尔值，表示操作是否成功。它们对应的 Object 方法，失败时都会抛出错误。

```
// 失败时抛出错误
Object.defineProperty(obj, name, desc);
// 失败时返回 false
Reflect.defineProperty(obj, name, desc);

```

上面代码中，Reflect.defineProperty 方法的作用与 Object.defineProperty 是一样的，都是为对象定义一个属性。但是，Reflect.defineProperty 方法失败时，不会抛出错误，只会返回 false。

## Object.observe()，Object.unobserve()

Object.observe 方法用来监听对象（以及数组）的变化。一旦监听对象发生变化，就会触发回调函数。

```
var user = {};
Object.observe(user, function(changes){
  changes.forEach(function(change) {
    user.fullName = user.firstName+" "+user.lastName;
  });
});

user.firstName = 'Michael';
user.lastName = 'Jackson';
user.fullName // 'Michael Jackson'

```

上面代码中，Object.observer 方法监听 user 对象。一旦该对象发生变化，就自动生成 fullName 属性。

一般情况下，Object.observe 方法接受两个参数，第一个参数是监听的对象，第二个函数是一个回调函数。一旦监听对象发生变化（比如新增或删除一个属性），就会触发这个回调函数。很明显，利用这个方法可以做很多事情，比如自动更新 DOM。

```
var div = $("#foo");

Object.observe(user, function(changes){
  changes.forEach(function(change) {
    var fullName = user.firstName+" "+user.lastName;
    div.text(fullName);
  });
});

```

上面代码中，只要 user 对象发生变化，就会自动更新 DOM。如果配合 jQuery 的 change 方法，就可以实现数据对象与 DOM 对象的双向自动绑定。

回调函数的 changes 参数是一个数组，代表对象发生的变化。下面是一个更完整的例子。

```
var o = {};

function observer(changes){
  changes.forEach(function(change) {
    console.log('发生变动的属性：' + change.name);
    console.log('变动前的值：' + change.oldValue);
    console.log('变动后的值：' + change.object[change.name]);
    console.log('变动类型：' + change.type);
  });
}

Object.observe(o, observer);

```

参照上面代码，Object.observe 方法指定的回调函数，接受一个数组（changes）作为参数。该数组的成员与对象的变化一一对应，也就是说，对象发生多少个变化，该数组就有多少个成员。每个成员是一个对象（change），它的 name 属性表示发生变化源对象的属性名，oldValue 属性表示发生变化前的值，object 属性指向变动后的源对象，type 属性表示变化的种类。基本上，change 对象是下面的样子。

```
var change = {
  object: {...},
  type: 'update',
  name: 'p2',
  oldValue: 'Property 2'
}

```

Object.observe 方法目前共支持监听六种变化。

*   add：添加属性
*   update：属性值的变化
*   delete：删除属性
*   setPrototype：设置原型
*   reconfigure：属性的 attributes 对象发生变化
*   preventExtensions：对象被禁止扩展（当一个对象变得不可扩展时，也就不必再监听了）

Object.observe 方法还可以接受第三个参数，用来指定监听的事件种类。

```
Object.observe(o, observer, ['delete']);

```

上面的代码表示，只在发生 delete 事件时，才会调用回调函数。

Object.unobserve 方法用来取消监听。

```
Object.unobserve(o, observer);

```

注意，Object.observe 和 Object.unobserve 这两个方法不属于 ES6，而是属于 ES7 的一部分。不过，Chrome 浏览器从 33 版起就已经支持。