# 函数的扩展

## 函数参数的默认值

在 ES6 之前，不能直接为函数的参数指定默认值，只能采用变通的方法。

```
function log(x, y) {
  y = y || 'World';
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello World

```

上面代码检查函数 log 的参数 y 有没有赋值，如果没有，则指定默认值为 World。这种写法的缺点在于，如果参数 y 赋值了，但是对应的布尔值为 false，则该赋值不起作用。就像上面代码的最后一行，参数 y 等于空字符，结果被改为默认值。

为了避免这个问题，通常需要先判断一下参数 y 是否被赋值，如果没有，再等于默认值。这有两种写法。

```
// 写法一
if (typeof y === 'undefined') {
  y = 'World';
}

// 写法二
if (arguments.length === 1) {
  y = 'World';
}

```

ES6 允许为函数的参数设置默认值，即直接写在参数定义的后面。

```
function log(x, y = 'World') {
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello

```

可以看到，ES6 的写法比 ES5 简洁许多，而且非常自然。下面是另一个例子。

```
function Point(x = 0, y = 0) {
  this.x = x;
  this.y = y;
}

var p = new Point();
// p = { x:0, y:0 }

```

除了简洁，ES6 的写法还有两个好处：首先，阅读代码的人，可以立刻意识到哪些参数是可以省略的，不用查看函数体或文档；其次，有利于将来的代码优化，即使未来的版本彻底拿到这个参数，也不会导致以前的代码无法运行。

默认值的写法非常灵活，下面是一个为对象属性设置默认值的例子。

```
fetch(url, { body = '', method = 'GET', headers = {} }){
  console.log(method);
}

```

上面代码中，传入函数 fetch 的第二个参数是一个对象，调用的时候可以为它的三个属性设置默认值。

甚至还可以设置双重默认值。

```
fetch(url, { method = 'GET' } = {}){
  console.log(method);
}

```

上面代码中，调用函数 fetch 时，如果不含第二个参数，则默认值为一个空对象；如果包含第二个参数，则它的 method 属性默认值为 GET。

定义了默认值的参数，必须是函数的尾部参数，其后不能再有其他无默认值的参数。这是因为有了默认值以后，该参数可以省略，只有位于尾部，才可能判断出到底省略了哪些参数。

```
// 以下两种写法都是错的

function f(x = 5, y) {
}

function f(x, y = 5, z) {
}

```

如果传入 undefined，将触发该参数等于默认值，null 则没有这个效果。

```
function foo(x = 5, y = 6){
  console.log(x,y);
}

foo(undefined, null)
// 5 null

```

上面代码中，x 参数对应 undefined，结果触发了默认值，y 参数等于 null，就没有触发默认值。

指定了默认值以后，函数的 length 属性，将返回没有指定默认值的参数个数。也就是说，指定了默认值后，length 属性将失真。

```
(function(a){}).length // 1
(function(a = 5){}).length // 0
(function(a, b, c = 5){}).length // 2

```

上面代码中，length 属性的返回值，等于函数的参数个数减去指定了默认值的参数个数。

利用参数默认值，可以指定某一个参数不得省略，如果省略就抛出一个错误。

```
function throwIfMissing() {
  throw new Error('Missing parameter');
}

function foo(mustBeProvided = throwIfMissing()) {
  return mustBeProvided;
}

foo()
// Error: Missing parameter

```

上面代码的 foo 函数，如果调用的时候没有参数，就会调用默认值 throwIfMissing 函数，从而抛出一个错误。

从上面代码还可以看到，参数 mustBeProvided 的默认值等于 throwIfMissing 函数的运行结果（即函数名之后有一对圆括号），这表明参数的默认值不是在定义时执行，而是在运行时执行（即如果参数已经赋值，默认值中的函数就不会运行），这与 python 语言不一样。

另一个需要注意的地方是，参数默认值所处的作用域，不是全局作用域，而是函数作用域。

```
var x = 1;

function foo(x, y = x) {
  console.log(y);
}

foo(2) // 2

```

上面代码中，参数 y 的默认值等于 x，由于处在函数作用域，所以 x 等于参数 x，而不是全局变量 x。

参数变量是默认声明的，所以不能用 let 或 const 再次声明。

```
function foo(x = 5) {
  let x = 1; // error
  const x = 2; // error
}

```

上面代码中，参数变量 x 是默认声明的，在函数体中，不能用 let 或 const 再次声明，否则会报错。

参数默认值可以与解构赋值，联合起来使用。

```
function foo({x, y = 5}) {
  console.log(x, y);
}

foo({}) // undefined, 5
foo({x: 1}) // 1, 5
foo({x: 1, y: 2}) // 1, 2

```

上面代码中，foo 函数的参数是一个对象，变量 x 和 y 用于解构赋值，y 有默认值 5。

## rest 参数

ES6 引入 rest 参数（形式为“...变量名”），用于获取函数的多余参数，这样就不需要使用 arguments 对象了。rest 参数搭配的变量是一个数组，该变量将多余的参数放入数组中。

```
function add(...values) {
   let sum = 0;

   for (var val of values) {
      sum += val;
   }

   return sum;
}

add(2, 5, 3) // 10

```

上面代码的 add 函数是一个求和函数，利用 rest 参数，可以向该函数传入任意数目的参数。

下面是一个 rest 参数代替 arguments 变量的例子。

```
// arguments 变量的写法
const sortNumbers = () =>
  Array.prototype.slice.call(arguments).sort();

// rest 参数的写法
const sortNumbers = (...numbers) => numbers.sort();

```

上面代码的两种写法，比较后可以发现，rest 参数的写法更自然也更简洁。

rest 参数中的变量代表一个数组，所以数组特有的方法都可以用于这个变量。下面是一个利用 rest 参数改写数组 push 方法的例子。

```
function push(array, ...items) {
  items.forEach(function(item) {
    array.push(item);
    console.log(item);
  });
}

var a = [];
push(a, 1, 2, 3)

```

注意，rest 参数之后不能再有其他参数（即只能是最后一个参数），否则会报错。

```
// 报错
function f(a, ...b, c) {
  // ...
}

```

函数的 length 属性，不包括 rest 参数。

```
(function(a) {}).length  // 1
(function(...a) {}).length  // 0
(function(a, ...b) {}).length  // 1

```

## 扩展运算符

扩展运算符（spread）是三个点（...）。它好比 rest 参数的逆运算，将一个数组转为用逗号分隔的参数序列。该运算符主要用于函数调用。

```
function push(array, ...items) {
  array.push(...items);
}

function add(x, y) {
  return x + y;
}

var numbers = [4, 38];
add(...numbers) // 42

```

上面代码中，`array.push(...items)`和`add(...numbers)`这两行，都是函数的调用，它们的都使用了扩展运算符。该运算符将一个数组，变为参数序列。

下面是 Date 函数的参数使用扩展运算符的例子。

```
const date = new Date(...[2015, 1, 1]);

```

由于扩展运算符可以展开数组，所以不再需要 apply 方法，将数组转为函数的参数了。

```
// ES5 的写法
function f (x, y, z){}
var args = [0, 1, 2];
f.apply(null, args);

// ES6 的写法
function f (x, y, z){}
var args = [0, 1, 2];
f(...args);

```

扩展运算符与正常的函数参数可以结合使用，非常灵活。

```
function f(v, w, x, y, z) { }
var args = [0, 1];
f(-1, ...args, 2, ...[3]);

```

下面是扩展运算符取代 apply 方法的一个实际的例子，应用 Math.max 方法，简化求出一个数组最大元素的写法。

```
// ES5 的写法
Math.max.apply(null, [14, 3, 77])

// ES6 的写法
Math.max(...[14, 3, 77])

// 等同于
Math.max(14, 3, 77);

```

上面代码表示，由于 JavaScript 不提供求数组最大元素的函数，所以只能套用 Math.max 函数，将数组转为一个参数序列，然后求最大值。有了扩展运算符以后，就可以直接用 Math.max 了。

另一个例子是通过 push 函数，将一个数组添加到另一个数组的尾部。

```
// ES5 的写法
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
Array.prototype.push.apply(arr1, arr2);

// ES6 的写法
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
arr1.push(...arr2);

```

上面代码的 ES5 写法中，push 方法的参数不能是数组，所以只好通过 apply 方法变通使用 push 方法。有了扩展运算符，就可以直接将数组传入 push 方法。

扩展运算符还可以用于数组的赋值。

```
var a = [1];
var b = [2, 3, 4];
var c = [6, 7];
var d = [0, ...a, ...b, 5, ...c];

d
// [0, 1, 2, 3, 4, 5, 6, 7]

```

上面代码其实也提供了，将一个数组拷贝进另一个数组的便捷方法。

```
const arr2 = [...arr1];

```

扩展运算符也可以与解构赋值结合起来，用于生成数组。

```
const [first, ...rest] = [1, 2, 3, 4, 5];
first // 1
rest  // [2, 3, 4, 5]

const [first, ...rest] = [];
first // undefined
rest  // []:

const [first, ...rest] = ["foo"];
first  // "foo"
rest   // []

const [first, ...rest] = ["foo", "bar"];
first // "foo"
rest  // ["bar"]

const [first, ...rest] = ["foo", "bar", "baz"];
first // "foo"
rest  // ["bar","baz"]

```

如果将扩展运算符用于数组赋值，只能放在参数的最后一位，否则会报错。

```
const [...butLast, last] = [1, 2, 3, 4, 5];
// 报错

const [first, ...middle, last] = [1, 2, 3, 4, 5];
// 报错

```

JavaScript 的函数只能返回一个值，如果需要返回多个值，只能返回数组或对象。扩展运算符提供了解决这个问题的一种变通方法。

```
var dateFields = readDateFields(database);
var d = new Date(...dateFields);

```

上面代码从数据库取出一行数据，通过扩展运算符，直接将其传入构造函数 Date。

扩展运算符还可以将字符串转为真正的数组。

```
[..."hello"]
// [ "h", "e", "l", "l", "o" ]

```

任何类似数组的对象，都可以用扩展运算符转为真正的数组。

```
var nodeList = document.querySelectorAll('div');
var array = [...nodeList];

```

上面代码中，querySelectorAll 方法返回的是一个 nodeList 对象，扩展运算符可以将其转为真正的数组。

扩展运算符内部调用的是数据结构的 Iterator 接口，因此只要具有 Iterator 接口的对象，都可以使用扩展运算符，比如 Map 结构。

```
let map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
]);

let arr = [...map.keys()]; // [1, 2, 3]

```

Generator 函数运行后，返回一个遍历器对象，因此也可以使用扩展运算符。

```
var go = function*(){
  yield 1;
  yield 2;
  yield 3;
};

[...go()] // [1, 2, 3]

```

上面代码中，变量 go 是一个 Generator 函数，执行后返回的是一个遍历器，对这个遍历器执行扩展运算符，就会将内部遍历得到的值，转为一个数组。

## 箭头函数

ES6 允许使用“箭头”（=>）定义函数。

```
var f = v => v;

```

上面的箭头函数等同于：

```
var f = function(v) {
  return v;
};

```

如果箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分。

```
var f = () => 5;
// 等同于
var f = function (){ return 5 };

var sum = (num1, num2) => num1 + num2;
// 等同于
var sum = function(num1, num2) {
  return num1 + num2;
};

```

如果箭头函数的代码块部分多于一条语句，就要使用大括号将它们括起来，并且使用 return 语句返回。

```
var sum = (num1, num2) => { return num1 + num2; }

```

由于大括号被解释为代码块，所以如果箭头函数直接返回一个对象，必须在对象外面加上括号。

```
var getTempItem = id => ({ id: id, name: "Temp" });

```

箭头函数可以与变量解构结合使用。

```
const full = ({ first, last }) => first + ' ' + last;

// 等同于
function full( person ){
  return person.first + ‘ ‘ + person.name;
}

```

箭头函数使得表达更加简洁。

```
const isEven = n => n % 2 == 0;
const square = n => n * n;

```

上面代码只用了两行，就定义了两个简单的工具函数。如果不用箭头函数，可能就要占用多行，而且还不如现在这样写醒目。

箭头函数的一个用处是简化回调函数。

```
// 正常函数写法
[1,2,3].map(function (x) {
  return x * x;
});

// 箭头函数写法
[1,2,3].map(x => x * x);

```

另一个例子是

```
// 正常函数写法
var result = values.sort(function(a, b) {
  return a - b;
});

// 箭头函数写法
var result = values.sort((a, b) => a - b);

```

下面是 rest 参数与箭头函数结合的例子。

```
const numbers = (...nums) => nums;

numbers(1, 2, 3, 4, 5)
// [1,2,3,4,5]

const headAndTail = (head, ...tail) => [head, tail];

headAndTail(1, 2, 3, 4, 5)
// [1,[2,3,4,5]]

```

箭头函数有几个使用注意点。

*   函数体内的 this 对象，绑定定义时所在的对象，而不是使用时所在的对象。
*   不可以当作构造函数，也就是说，不可以使用 new 命令，否则会抛出一个错误。
*   不可以使用 arguments 对象，该对象在函数体内不存在。

上面三点中，第一点尤其值得注意。this 对象的指向是可变的，但是在箭头函数中，它是固定的。下面的代码是一个例子，将 this 对象绑定定义时所在的对象。

```
var handler = {
  id: "123456",

  init: function() {
    document.addEventListener("click",
      event => this.doSomething(event.type), false);
  },

  doSomething: function(type) {
    console.log("Handling " + type  + " for " + this.id);
  }
};

```

上面代码的 init 方法中，使用了箭头函数，这导致 this 绑定 handler 对象，否则回调函数运行时，this.doSomething 这一行会报错，因为此时 this 指向全局对象。

由于 this 在箭头函数中被绑定，所以不能用 call()、apply()、bind()这些方法去改变 this 的指向。

长期以来，JavaScript 语言的 this 对象一直是一个令人头痛的问题，在对象方法中使用 this，必须非常小心。箭头函数绑定 this，很大程度上解决了这个困扰。

箭头函数内部，还可以再使用箭头函数。下面是一个 ES5 语法的多重嵌套函数。

```
function insert(value) {
  return {into: function (array) {
    return {after: function (afterValue) {
      array.splice(array.indexOf(afterValue) + 1, 0, value);
      return array;
    }};
  }};
}

insert(2).into([1, 3]).after(1); //[1, 2, 3]

```

上面这个函数，可以使用箭头函数改写。

```
let insert = (value) => ({into: (array) => ({after: (afterValue) => {
  array.splice(array.indexOf(afterValue) + 1, 0, value);
  return array;
}})});

insert(2).into([1, 3]).after(1); //[1, 2, 3]

```

下面是一个部署管道机制（pipeline）的例子，即前一个函数的输出是后一个函数的输入。

```
const pipeline = (...funcs) =>
  val => funcs.reduce((a, b) => b(a), val);

const plus1 = a => a + 1;
const mult2 = a => a * 2;
const addThenMult = pipeline(plus1, mult2);

addThenMult(5)
// 12

```

如果觉得上面的写法可读性比较差，也可以采用下面的写法。

```
const plus1 = a => a + 1;
const mult2 = a => a * 2;

mult2(plus1(5))
// 12

```

箭头函数还有一个功能，就是可以很方便地改写λ演算。

```
// λ演算的写法
fix = λf.(λx.f(λv.x(x)(v)))(λx.f(λv.x(x)(v)))

// ES6 的写法
var fix = f => (x => f(v => x(x)(v)))
               (x => f(v => x(x)(v)));

```

上面两种写法，几乎是一一对应的。由于λ演算对于计算机科学非常重要，这使得我们可以用 ES6 作为替代工具，探索计算机科学。

## 函数绑定

箭头函数可以绑定 this 对象，大大减少了显式绑定 this 对象的写法（call、apply、bind）。但是，箭头函数并不适用于所有场合，所以 ES7 提出了“函数绑定”（function bind）运算符，用来取代 call、apply、bind 调用。虽然该语法还是 ES7 的一个提案，但是 Babel 转码器已经支持。

函数绑定运算符是并排的两个双引号（::），双引号左边是一个对象，右边是一个函数。该运算符会自动将左边的对象，作为上下文环境（即 this 对象），绑定到右边的函数上面。

```
let log = ::console.log;
// 等同于
var log = console.log.bind(console);

foo::bar;
// 等同于
bar.call(foo);

foo::bar(...arguments);
i// 等同于
bar.apply(foo, arguments);

```

## 尾调用优化

### 什么是尾调用？

尾调用（Tail Call）是函数式编程的一个重要概念，本身非常简单，一句话就能说清楚，就是指某个函数的最后一步是调用另一个函数。

```
function f(x){
  return g(x);
}

```

上面代码中，函数 f 的最后一步是调用函数 g，这就叫尾调用。

以下三种情况，都不属于尾调用。

```
// 情况一
function f(x){
  let y = g(x);
  return y;
}

// 情况二
function f(x){
  return g(x) + 1;
}

// 情况三
function f(x){
  g(x);
}

```

上面代码中，情况一是调用函数 g 之后，还有别的操作，所以不属于尾调用，即使语义完全一样。情况二也属于调用后还有操作，即使写在一行内。情况三等同于下面的代码。

```
function f(x){
  g(x);
  return undefined;
}

```

尾调用不一定出现在函数尾部，只要是最后一步操作即可。

```
function f(x) {
  if (x > 0) {
    return m(x)
  }
  return n(x);
}

```

上面代码中，函数 m 和 n 都属于尾调用，因为它们都是函数 f 的最后一步操作。

### 尾调用优化

尾调用之所以与其他调用不同，就在于它的特殊的调用位置。

我们知道，函数调用会在内存形成一个“调用记录”，又称“调用帧”（call frame），保存调用位置和内部变量等信息。如果在函数 A 的内部调用函数 B，那么在 A 的调用帧上方，还会形成一个 B 的调用帧。等到 B 运行结束，将结果返回到 A，B 的调用帧才会消失。如果函数 B 内部还调用函数 C，那就还有一个 C 的调用帧，以此类推。所有的调用帧，就形成一个“调用栈”（call stack）。

尾调用由于是函数的最后一步操作，所以不需要保留外层函数的调用帧，因为调用位置、内部变量等信息都不会再用到了，只要直接用内层函数的调用帧，取代外层函数的调用帧就可以了。

```
function f() {
  let m = 1;
  let n = 2;
  return g(m + n);
}
f();

// 等同于
function f() {
  return g(3);
}
f();

// 等同于
g(3);

```

上面代码中，如果函数 g 不是尾调用，函数 f 就需要保存内部变量 m 和 n 的值、g 的调用位置等信息。但由于调用 g 之后，函数 f 就结束了，所以执行到最后一步，完全可以删除 f(x) 的调用帧，只保留 g(3) 的调用帧。

这就叫做“尾调用优化”（Tail call optimization），即只保留内层函数的调用帧。如果所有函数都是尾调用，那么完全可以做到每次执行时，调用帧只有一项，这将大大节省内存。这就是“尾调用优化”的意义。

### 尾递归

函数调用自身，称为递归。如果尾调用自身，就称为尾递归。

递归非常耗费内存，因为需要同时保存成千上百个调用帧，很容易发生“栈溢出”错误（stack overflow）。但对于尾递归来说，由于只存在一个调用帧，所以永远不会发生“栈溢出”错误。

```
function factorial(n) {
  if (n === 1) return 1;
  return n * factorial(n - 1);
}

factorial(5) // 120

```

上面代码是一个阶乘函数，计算 n 的阶乘，最多需要保存 n 个调用记录，复杂度 O(n) 。

如果改写成尾递归，只保留一个调用记录，复杂度 O(1) 。

```
function factorial(n, total) {
  if (n === 1) return total;
  return factorial(n - 1, n * total);
}

factorial(5, 1) // 120

```

由此可见，“尾调用优化”对递归操作意义重大，所以一些函数式编程语言将其写入了语言规格。ES6 也是如此，第一次明确规定，所有 ECMAScript 的实现，都必须部署“尾调用优化”。这就是说，在 ES6 中，只要使用尾递归，就不会发生栈溢出，相对节省内存。

### 递归函数的改写

尾递归的实现，往往需要改写递归函数，确保最后一步只调用自身。做到这一点的方法，就是把所有用到的内部变量改写成函数的参数。比如上面的例子，阶乘函数 factorial 需要用到一个中间变量 total ，那就把这个中间变量改写成函数的参数。这样做的缺点就是不太直观，第一眼很难看出来，为什么计算 5 的阶乘，需要传入两个参数 5 和 1？

两个方法可以解决这个问题。方法一是在尾递归函数之外，再提供一个正常形式的函数。

```
function tailFactorial(n, total) {
  if (n === 1) return total;
  return tailFactorial(n - 1, n * total);
}

function factorial(n) {
  return tailFactorial(n, 1);
}

factorial(5) // 120

```

上面代码通过一个正常形式的阶乘函数 factorial ，调用尾递归函数 tailFactorial ，看起来就正常多了。

函数式编程有一个概念，叫做柯里化（currying），意思是将多参数的函数转换成单参数的形式。这里也可以使用柯里化。

```
function currying(fn, n) {
  return function (m) {
    return fn.call(this, m, n);
  };
}

function tailFactorial(n, total) {
  if (n === 1) return total;
  return tailFactorial(n - 1, n * total);
}

const factorial = currying(tailFactorial, 1);

factorial(5) // 120

```

上面代码通过柯里化，将尾递归函数 tailFactorial 变为只接受 1 个参数的 factorial 。

第二种方法就简单多了，就是采用 ES6 的函数默认值。

```
function factorial(n, total = 1) {
  if (n === 1) return total;
  return factorial(n - 1, n * total);
}

factorial(5) // 120

```

上面代码中，参数 total 有默认值 1，所以调用时不用提供这个值。

总结一下，递归本质上是一种循环操作。纯粹的函数式编程语言没有循环操作命令，所有的循环都用递归实现，这就是为什么尾递归对这些语言极其重要。对于其他支持“尾调用优化”的语言（比如 Lua，ES6），只需要知道循环可以用递归代替，而一旦使用递归，就最好使用尾递归。