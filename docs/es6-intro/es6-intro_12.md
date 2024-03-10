# Generator 函数

## 简介

### 基本概念

Generator 函数是 ES6 提供的一种异步编程解决方案，语法行为与传统函数完全不同。本章详细介绍 Generator 函数的语法和 API，它的异步编程应用请看《异步操作》一章。

Generator 函数有多种理解角度。从语法上，首先可以把它理解成一个函数的内部状态的遍历器（也就是说，Generator 函数是一个状态机）。它每调用一次，就进入下一个内部状态。Generator 函数可以控制内部状态的变化，依次遍历这些状态。

形式上，Generator 函数是一个普通函数，但是有两个特征。一是，function 命令与函数名之间有一个星号；二是，函数体内部使用 yield 语句，定义遍历器的每个成员，即不同的内部状态（yield 语句在英语里的意思就是“产出”）。

```js
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}

var hw = helloWorldGenerator();

```

上面代码定义了一个 Generator 函数 helloWorldGenerator，它内部有两个 yield 语句“hello”和“world”，即该函数有三个状态：hello，world 和 return 语句（结束执行）。

然后，Generator 函数的调用方法与普通函数一样，也是在函数名后面加上一对圆括号。不同的是，调用 Generator 函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象，也就是上一章介绍的遍历器对象（Iterator Object）。

下一步，必须调用遍历器对象的 next 方法，使得指针移向下一个状态。也就是说，每次调用 next 方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个 yield 语句（或 return 语句）为止。换言之，Generator 函数是分段执行的，yield 命令是暂停执行的标记，而 next 方法可以恢复执行。

```js
hw.next()
// { value: 'hello', done: false }

hw.next()
// { value: 'world', done: false }

hw.next()
// { value: 'ending', done: true }

hw.next()
// { value: undefined, done: true }

```

上面代码一共调用了四次 next 方法。

第一次调用，Generator 函数开始执行，直到遇到第一个 yield 语句为止。next 方法返回一个对象，它的 value 属性就是当前 yield 语句的值 hello，done 属性的值 false，表示遍历还没有结束。

第二次调用，Generator 函数从上次 yield 语句停下的地方，一直执行到下一个 yield 语句。next 方法返回的对象的 value 属性就是当前 yield 语句的值 world，done 属性的值 false，表示遍历还没有结束。

第三次调用，Generator 函数从上次 yield 语句停下的地方，一直执行到 return 语句（如果没有 return 语句，就执行到函数结束）。next 方法返回的对象的 value 属性，就是紧跟在 return 语句后面的表达式的值（如果没有 return 语句，则 value 属性的值为 undefined），done 属性的值 true，表示遍历已经结束。

第四次调用，此时 Generator 函数已经运行完毕，next 方法返回对象的 value 属性为 undefined，done 属性为 true。以后再调用 next 方法，返回的都是这个值。

总结一下，调用 Generator 函数，返回一个部署了 Iterator 接口的遍历器对象，用来操作内部指针。以后，每次调用遍历器对象的 next 方法，就会返回一个有着 value 和 done 两个属性的对象。value 属性表示当前的内部状态的值，是 yield 语句后面那个表达式的值；done 属性是一个布尔值，表示是否遍历结束。

### yield 语句

由于 Generator 函数返回的遍历器，只有调用 next 方法才会遍历下一个内部状态，所以其实提供了一种可以暂停执行的函数。yield 语句就是暂停标志。

遍历器 next 方法的运行逻辑如下。

（1）遇到 yield 语句，就暂停执行后面的操作，并将紧跟在 yield 后面的那个表达式的值，作为返回的对象的 value 属性值。

（2）下一次调用 next 方法时，再继续往下执行，直到遇到下一个 yield 语句。

（3）如果没有再遇到新的 yield 语句，就一直运行到函数结束，直到 return 语句为止，并将 return 语句后面的表达式的值，作为返回的对象的 value 属性值。

（4）如果该函数没有 return 语句，则返回的对象的 value 属性值为 undefined。

需要注意的是，yield 语句后面的表达式，只有当调用 next 方法、内部指针指向该语句时才会执行，因此等于为 JavaScript 提供了手动的“惰性求值”（Lazy Evaluation）的语法功能。

```js
function* gen{
  yield  123 + 456;
}

```

上面代码中，yield 后面的表达式`123 + 456`，不会立即求值，只会在 next 方法将指针移到这一句时，才会求值。

yield 语句与 return 语句既有相似之处，也有区别。相似之处在于，都能返回紧跟在语句后面的那个表达式的值。区别在于每次遇到 yield，函数暂停执行，下一次再从该位置继续向后执行，而 return 语句不具备位置记忆的功能。一个函数里面，只能执行一次（或者说一个）return 语句，但是可以执行多次（或者说多个）yield 语句。正常函数只能返回一个值，因为只能执行一次 return；Generator 函数可以返回一系列的值，因为可以有任意多个 yield。从另一个角度看，也可以说 Generator 生成了一系列的值，这也就是它的名称的来历（在英语中，generator 这个词是“生成器”的意思）。

Generator 函数可以不用 yield 语句，这时就变成了一个单纯的暂缓执行函数。

```js
function* f() {
  console.log('执行了！')
}

var generator = f();

setTimeout(function () {
  generator.next()
}, 2000);

```

上面代码中，函数 f 如果是普通函数，在为变量 generator 赋值时就会执行。但是，函数 f 是一个 Generator 函数，就变成只有调用 next 方法时，函数 f 才会执行。

另外需要注意，yield 语句不能用在普通函数中，否则会报错。

```js
(function (){
  yield 1;
})()
// SyntaxError: Unexpected number

```

上面代码在一个普通函数中使用 yield 语句，结果产生一个句法错误。

下面是另外一个例子。

```js
var arr = [1, [[2, 3], 4], [5, 6]];

var flat = function* (a){
  a.forEach(function(item){
    if (typeof item !== 'number'){
      yield* flat(item);
    } else {
      yield item;
    }
  }
};

for (var f of flat(arr)){
  console.log(f);
}

```

上面代码也会产生句法错误，因为 forEach 方法的参数是一个普通函数，但是在里面使用了 yield 语句。一种修改方法是改用 for 循环。

```js
var arr = [1, [[2, 3], 4], [5, 6]];

var flat = function* (a){
  var length = a.length;
  for(var i =0;i<length;i++){
    var item = a[i];
    if (typeof item !== 'number'){
      yield* flat(item);
    } else {
      yield item;
    }
  }
};

for (var f of flat(arr)){
  console.log(f);
}
// 1, 2, 3, 4, 5, 6

```

### 与 Iterator 的关系

上一章说过，任意一个对象的 Symbol.iterator 属性，等于该对象的遍历器函数，调用该函数会返回该对象的一个遍历器。

遍历器本身也是一个对象，它的 Symbol.iterator 属性执行后，返回自身。

```js
function* gen(){
  // some code
}

var g = gen();

g[Symbol.iterator]() === g
// true

```

上面代码中，gen 是一个 Generator 函数，调用它会生成一个遍历器 g。遍历器 g 的 Symbol.iterator 属性是一个遍历器函数，执行后返回它自己。

## next 方法的参数

yield 语句本身没有返回值，或者说总是返回 undefined。next 方法可以带一个参数，该参数就会被当作上一个 yield 语句的返回值。

```js
function* f() {
  for(var i=0; true; i++) {
    var reset = yield i;
    if(reset) { i = -1; }
  }
}

var g = f();

g.next() // { value: 0, done: false }
g.next() // { value: 1, done: false }
g.next(true) // { value: 0, done: false }

```

上面代码先定义了一个可以无限运行的 Generator 函数 f，如果 next 方法没有参数，每次运行到 yield 语句，变量 reset 的值总是 undefined。当 next 方法带一个参数 true 时，当前的变量 reset 就被重置为这个参数（即 true），因此 i 会等于-1，下一轮循环就会从-1 开始递增。

这个功能有很重要的语法意义。Generator 函数从暂停状态到恢复运行，它的上下文状态（context）是不变的。通过 next 方法的参数，就有办法在 Generator 函数开始运行之后，继续向函数体内部注入值。也就是说，可以在 Generator 函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为。

再看一个例子。

```js
function* foo(x) {
  var y = 2 * (yield (x + 1));
  var z = yield (y / 3);
  return (x + y + z);
}

var a = foo(5);

a.next() // Object{value:6, done:false}
a.next() // Object{value:NaN, done:false}
a.next() // Object{value:NaN, done:false}

```

上面代码中，第二次运行 next 方法的时候不带参数，导致 y 的值等于`2 * undefined`（即 NaN），除以 3 以后还是 NaN，因此返回对象的 value 属性也等于 NaN。第三次运行 Next 方法的时候不带参数，所以 z 等于 undefined，返回对象的 value 属性等于`5 + NaN + undefined`，即 NaN。

如果向 next 方法提供参数，返回结果就完全不一样了。

```js
function* foo(x) {
  var y = 2 * (yield (x + 1));
  var z = yield (y / 3);
  return (x + y + z);
}

var it = foo(5);

it.next()
// { value:6, done:false }
it.next(12)
// { value:8, done:false }
it.next(13)
// { value:42, done:true }

```

上面代码第一次调用 next 方法时，返回`x+1`的值 6；第二次调用 next 方法，将上一次 yield 语句的值设为 12，因此 y 等于 24，返回`y / 3`的值 8；第三次调用 next 方法，将上一次 yield 语句的值设为 13，因此 z 等于 13，这时 x 等于 5，y 等于 24，所以 return 语句的值等于 42。

注意，由于 next 方法的参数表示上一个 yield 语句的返回值，所以第一次使用 next 方法时，不能带有参数。V8 引擎直接忽略第一次使用 next 方法时的参数，只有从第二次使用 next 方法开始，参数才是有效的。

## for...of 循环

for...of 循环可以自动遍历 Generator 函数，且此时不再需要调用 next 方法。

```js
function *foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5

```

上面代码使用 for...of 循环，依次显示 5 个 yield 语句的值。这里需要注意，一旦 next 方法的返回对象的 done 属性为 true，for...of 循环就会中止，且不包含该返回对象，所以上面代码的 return 语句返回的 6，不包括在 for...of 循环之中。

下面是一个利用 generator 函数和 for...of 循环，实现斐波那契数列的例子。

```js
function* fibonacci() {
  let [prev, curr] = [0, 1];
  for (;;) {
    [prev, curr] = [curr, prev + curr];
    yield curr;
  }
}

for (let n of fibonacci()) {
  if (n > 1000) break;
  console.log(n);
}

```

从上面代码可见，使用 for...of 语句时不需要使用 next 方法。

## throw 方法

Generator 函数还有一个特点，它可以在函数体外抛出错误，然后在函数体内捕获。

```js
var g = function* () {
  while (true) {
    try {
      yield;
    } catch (e) {
      if (e != 'a') throw e;
      console.log('内部捕获', e);
    }
  }
};

var i = g();
i.next();

try {
  i.throw('a');
  i.throw('b');
} catch (e) {
  console.log('外部捕获', e);
}
// 内部捕获 a
// 外部捕获 b

```

上面代码中，遍历器 i 连续抛出两个错误。第一个错误被 Generator 函数体内的 catch 捕获，然后 Generator 函数执行完成，于是第二个错误被函数体外的 catch 捕获。

注意，上面代码的错误，是用遍历器的 throw 方法抛出的，而不是用 throw 命令抛出的。后者只能被函数体外的 catch 语句捕获。

```js
var g = function* () {
  while (true) {
    try {
      yield;
    } catch (e) {
      if (e != 'a') throw e;
      console.log('内部捕获', e);
    }
  }
};

var i = g();
i.next();

try {
  throw new Error('a');
  throw new Error('b');
} catch (e) {
  console.log('外部捕获', e);
}
// 外部捕获 [Error: a]

```

上面代码之所以只捕获了 a，是因为函数体外的 catch 语句块，捕获了抛出的 a 错误以后，就不会再继续执行 try 语句块了。

如果遍历器函数内部没有部署 try...catch 代码块，那么 throw 方法抛出的错误，将被外部 try...catch 代码块捕获。

```js
var g = function* () {
  while (true) {
    yield;
    console.log('内部捕获', e);
  }
};

var i = g();
i.next();

try {
  i.throw('a');
  i.throw('b');
} catch (e) {
  console.log('外部捕获', e);
}
// 外部捕获 a

```

上面代码中，遍历器函数 g 内部，没有部署 try...catch 代码块，所以抛出的错误直接被外部 catch 代码块捕获。

如果遍历器函数内部部署了 try...catch 代码块，那么遍历器的 throw 方法抛出的错误，不影响下一次遍历，否则遍历直接终止。

```js
var gen = function* gen(){
  yield console.log('hello');
  yield console.log('world');
}

var g = gen();
g.next();

try {
  g.throw();
} catch (e) {
  g.next();
}
// hello

```

上面代码只输出 hello 就结束了，因为第二次调用 next 方法时，遍历器状态已经变成终止了。但是，如果使用 throw 方法抛出错误，不会影响遍历器状态。

```js
var gen = function* gen(){
  yield console.log('hello');
  yield console.log('world');
}

var g = gen();
g.next();

try {
  throw new Error();
} catch (e) {
  g.next();
}
// hello
// world

```

上面代码中，throw 命令抛出的错误不会影响到遍历器的状态，所以两次执行 next 方法，都取到了正确的操作。

这种函数体内捕获错误的机制，大大方便了对错误的处理。如果使用回调函数的写法，想要捕获多个错误，就不得不为每个函数写一个错误处理语句。

```js
foo('a', function (a) {
  if (a.error) {
    throw new Error(a.error);
  }

  foo('b', function (b) {
    if (b.error) {
      throw new Error(b.error);
    }

    foo('c', function (c) {
      if (c.error) {
        throw new Error(c.error);
      }

      console.log(a, b, c);
    });
  });
});

```

使用 Generator 函数可以大大简化上面的代码。

```js
function* g(){
  try {
    var a = yield foo('a');
    var b = yield foo('b');
    var c = yield foo('c');
  } catch (e) {
    console.log(e);
  }

  console.log(a, b, c);
}

```

反过来，Generator 函数内抛出的错误，也可以被函数体外的 catch 捕获。

```js
function *foo() {
  var x = yield 3;
  var y = x.toUpperCase();
  yield y;
}

var it = foo();

it.next(); // { value:3, done:false }

try {
  it.next(42);
} catch (err) {
  console.log(err);
}

```

上面代码中，第二个 next 方法向函数体内传入一个参数 42，数值是没有 toUpperCase 方法的，所以会抛出一个 TypeError 错误，被函数体外的 catch 捕获。

一旦 Generator 执行过程中抛出错误，就不会再执行下去了。如果此后还调用 next 方法，将返回一个 value 属性等于 undefined、done 属性等于 true 的对象，即 JavaScript 引擎认为这个 Generator 已经运行结束了。

```js
function* g() {
  yield 1;
  console.log('throwing an exception');
  throw new Error('generator broke!');
  yield 2;
  yield 3;
}

function log(generator) {
  var v;
  console.log('starting generator');
  try {
    v = generator.next();
    console.log('第一次运行 next 方法', v);
  } catch (err) {
    console.log('捕捉错误', v);
  }
  try {
    v = generator.next();
    console.log('第二次运行 next 方法', v);
  } catch (err) {
    console.log('捕捉错误', v);
  }
  try {
    v = generator.next();
    console.log('第三次运行 next 方法', v);
  } catch (err) {
    console.log('捕捉错误', v);
  }
  console.log('caller done');
}

log(g());
// starting generator
// 第一次运行 next 方法 { value: 1, done: false }
// throwing an exception
// 捕捉错误 { value: 1, done: false }
// 第三次运行 next 方法 { value: undefined, done: true }
// caller done

```

上面代码一共三次运行 next 方法，第二次运行的时候会抛出错误，然后第三次运行的时候，Generator 函数就已经结束了，不再执行下去了。

## yield*语句

如果 yield 命令后面跟的是一个遍历器，需要在 yield 命令后面加上星号，表明它返回的是一个遍历器。这被称为 yield*语句。

```js
let delegatedIterator = (function* () {
  yield 'Hello!';
  yield 'Bye!';
}());

let delegatingIterator = (function* () {
  yield 'Greetings!';
  yield* delegatedIterator;
  yield 'Ok, bye.';
}());

for(let value of delegatingIterator) {
  console.log(value);
}
// "Greetings!
// "Hello!"
// "Bye!"
// "Ok, bye."

```

上面代码中，delegatingIterator 是代理者，delegatedIterator 是被代理者。由于`yield* delegatedIterator`语句得到的值，是一个遍历器，所以要用星号表示。运行结果就是使用一个遍历器，遍历了多个 Genertor 函数，有递归的效果。

yield*语句等同于在 Generator 函数内部，部署一个 for...of 循环。

```js
function* concat(iter1, iter2) {
  yield* iter1;
  yield* iter2;
}

// 等同于

function* concat(iter1, iter2) {
  for (var value of iter1) {
    yield value;
  }
  for (var value of iter2) {
    yield value;
  }
}

```

上面代码说明，yield*不过是 for...of 的一种简写形式，完全可以用后者替代前者。

再来看一个对比的例子。

```js
function* inner() {
  yield 'hello!'
}

function* outer1() {
  yield 'open'
  yield inner()
  yield 'close'
}

var gen = outer1()
gen.next() // -> 'open'
gen.next() // -> a generator
gen.next() // -> 'close'

function* outer2() {
  yield 'open'
  yield* inner()
  yield 'close'
}

var gen = outer2()
gen.next() // -> 'open'
gen.next() // -> 'hello!'
gen.next() // -> 'close'

```

上面例子中，outer2 使用了`yield*`，outer1 没使用。结果就是，outer1 返回一个遍历器，outer2 返回该遍历器的内部值。

如果`yield*`后面跟着一个数组，由于数组原生支持遍历器，因此就会遍历数组成员。

```js
function* gen(){
  yield* ["a", "b", "c"];
}

gen().next() // { value:"a", done:false }

```

上面代码中，yield 命令后面如果不加星号，返回的是整个数组，加了星号就表示返回的是数组的遍历器。

如果被代理的 Generator 函数有 return 语句，那么就可以向代理它的 Generator 函数返回数据。

```js
function *foo() {
  yield 2;
  yield 3;
  return "foo";
}

function *bar() {
  yield 1;
  var v = yield *foo();
  console.log( "v: " + v );
  yield 4;
}

var it = bar();

it.next(); //
it.next(); //
it.next(); //
it.next(); // "v: foo"
it.next(); //

```

上面代码在第四次调用 next 方法的时候，屏幕上会有输出，这是因为函数 foo 的 return 语句，向函数 bar 提供了返回值。

`yield*`命令可以很方便地取出嵌套数组的所有成员。

```js
function* iterTree(tree) {
  if (Array.isArray(tree)) {
    for(let i=0; i < tree.length; i++) {
      yield* iterTree(tree[i]);
    }
  } else {
    yield tree;
  }
}

const tree = [ 'a', ['b', 'c'], ['d', 'e'] ];

for(let x of iterTree(tree)) {
  console.log(x);
}
// a
// b
// c
// d
// e

```

下面是一个稍微复杂的例子，使用 yield*语句遍历完全二叉树。

```js
// 下面是二叉树的构造函数，
// 三个参数分别是左树、当前节点和右树
function Tree(left, label, right) {
  this.left = left;
  this.label = label;
  this.right = right;
}

// 下面是中序（inorder）遍历函数。
// 由于返回的是一个遍历器，所以要用 generator 函数。
// 函数体内采用递归算法，所以左树和右树要用 yield*遍历
function* inorder(t) {
  if (t) {
    yield* inorder(t.left);
    yield t.label;
    yield* inorder(t.right);
  }
}

// 下面生成二叉树
function make(array) {
  // 判断是否为叶节点
  if (array.length == 1) return new Tree(null, array[0], null);
  return new Tree(make(array[0]), array[1], make(array[2]));
}
let tree = make([[['a'], 'b', ['c']], 'd', [['e'], 'f', ['g']]]);

// 遍历二叉树
var result = [];
for (let node of inorder(tree)) {
  result.push(node);
}

result
// ['a', 'b', 'c', 'd', 'e', 'f', 'g']

```

## 作为对象属性的 Generator 函数

如果一个对象的属性是 Generator 函数，可以简写成下面的形式。

```js
let obj = {
  * myGeneratorMethod() {
    ···
  }
};

```

上面代码中，myGeneratorMethod 属性前面有一个星号，表示这个属性是一个 Generator 函数。

它的完整形式如下，与上面的写法是等价的。

```js
let obj = {
  myGeneratorMethod: function* () {
    // ···
  }
};

```

## Generator 函数推导

ES7 在数组推导的基础上，提出了 Generator 函数推导（Generator comprehension）。

```js
let generator = function* () {
  for (let i = 0; i < 6; i++) {
    yield i;
  }
}

let squared = ( for (n of generator()) n * n );
// 等同于
// let squared = Array.from(generator()).map(n => n * n);

console.log(...squared);
// 0 1 4 9 16 25

```

“推导”这种语法结构，不仅可以用于数组，ES7 将其推广到了 Generator 函数。for...of 循环会自动调用遍历器的 next 方法，将返回值的 value 属性作为数组的一个成员。

Generator 函数推导是对数组结构的一种模拟，它的最大优点是惰性求值，即直到真正用到时才会求值，这样可以保证效率。请看下面的例子。

```js
let bigArray = new Array(100000);
for (let i = 0; i < 100000; i++) {
  bigArray[i] = i;
}

let first = bigArray.map(n => n * n)[0];
console.log(first);

```

上面例子遍历一个大数组，但是在真正遍历之前，这个数组已经生成了，占用了系统资源。如果改用 Generator 函数推导，就能避免这一点。下面代码只在用到时，才会生成一个大数组。

```js
let bigGenerator = function* () {
  for (let i = 0; i < 100000; i++) {
    yield i;
  }
}

let squared = ( for (n of bigGenerator()) n * n );

console.log(squared.next());

```

## 含义

### Generator 与状态机

Generator 是实现状态机的最佳结构。比如，下面的 clock 函数就是一个状态机。

```js
var ticking = true;
var clock = function() {
  if (ticking)
    console.log('Tick!');
  else
    console.log('Tock!');
  ticking = !ticking;
}

```

上面代码的 clock 函数一共有两种状态（Tick 和 Tock），每运行一次，就改变一次状态。这个函数如果用 Generator 实现，就是下面这样。

```js
var clock = function*(_) {
  while (true) {
    yield _;
    console.log('Tick!');
    yield _;
    console.log('Tock!');
  }
};

```

上面的 Generator 实现与 ES5 实现对比，可以看到少了用来保存状态的外部变量 ticking，这样就更简洁，更安全（状态不会被非法篡改）、更符合函数式编程的思想，在写法上也更优雅。Generator 之所以可以不用外部变量保存状态，是因为它本身就包含了一个状态信息，即目前是否处于暂停态。

### Generator 与协程

协程（coroutine）是一种程序运行的方式，可以理解成“协作的线程”或“协作的函数”。协程既可以用单线程实现，也可以用多线程实现。前者是一种特殊的子例程，后者是一种特殊的线程。

**（1）协程与子例程的差异**

传统的“子例程”（subroutine）采用堆栈式“后进先出”的执行方式，只有当调用的子函数完全执行完毕，才会结束执行父函数。协程与其不同，多个线程（单线程情况下，即多个函数）可以并行执行，但是只有一个线程（或函数）处于正在运行的状态，其他线程（或函数）都处于暂停态（suspended），线程（或函数）之间可以交换执行权。也就是说，一个线程（或函数）执行到一半，可以暂停执行，将执行权交给另一个线程（或函数），等到稍后收回执行权的时候，再恢复执行。这种可以并行执行、交换执行权的线程（或函数），就称为协程。

从实现上看，在内存中，子例程只使用一个栈（stack），而协程是同时存在多个栈，但只有一个栈是在运行状态，也就是说，协程是以多占用内存为代价，实现多任务的并行。

**（2）协程与普通线程的差异**

不难看出，协程适合用于多任务运行的环境。在这个意义上，它与普通的线程很相似，都有自己的执行上下文、可以分享全局变量。它们的不同之处在于，同一时间可以有多个线程处于运行状态，但是运行的协程只能有一个，其他协程都处于暂停状态。此外，普通的线程是抢先式的，到底哪个线程优先得到资源，必须由运行环境决定，但是协程是合作式的，执行权由协程自己分配。

由于 ECMAScript 是单线程语言，只能保持一个调用栈。引入协程以后，每个任务可以保持自己的调用栈。这样做的最大好处，就是抛出错误的时候，可以找到原始的调用栈。不至于像异步操作的回调函数那样，一旦出错，原始的调用栈早就结束。

Generator 函数是 ECMAScript 6 对协程的实现，但属于不完全实现。Generator 函数被称为“半协程”（semi-coroutine），意思是只有 Generator 函数的调用者，才能将程序的执行权还给 Generator 函数。如果是完全执行的协程，任何函数都可以让暂停的协程继续执行。

如果将 Generator 函数当作协程，完全可以将多个需要互相协作的任务写成 Generator 函数，它们之间使用 yield 语句交换控制权。

## 应用

Generator 可以暂停函数执行，返回任意表达式的值。这种特点使得 Generator 有多种应用场景。

### （1）异步操作的同步化表达

Generator 函数的暂停执行的效果，意味着可以把异步操作写在 yield 语句里面，等到调用 next 方法时再往后执行。这实际上等同于不需要写回调函数了，因为异步操作的后续操作可以放在 yield 语句下面，反正要等到调用 next 方法时再执行。所以，Generator 函数的一个重要实际意义就是用来处理异步操作，改写回调函数。

```js
function* loadUI() {
  showLoadingScreen();
  yield loadUIDataAsynchronously();
 hideLoadingScreen();
}
var loader = loadUI();
// 加载 UI
loader.next()

// 卸载 UI
loader.next()

```

上面代码表示，第一次调用 loadUI 函数时，该函数不会执行，仅返回一个遍历器。下一次对该遍历器调用 next 方法，则会显示 Loading 界面，并且异步加载数据。等到数据加载完成，再一次使用 next 方法，则会隐藏 Loading 界面。可以看到，这种写法的好处是所有 Loading 界面的逻辑，都被封装在一个函数，按部就班非常清晰。

Ajax 是典型的异步操作，通过 Generator 函数部署 Ajax 操作，可以用同步的方式表达。

```js
function* main() {
  var result = yield request("http://some.url");
  var resp = JSON.parse(result);
    console.log(resp.value);
}

function request(url) {
  makeAjaxCall(url, function(response){
    it.next(response);
  });
}

var it = main();
it.next();

```

上面代码的 main 函数，就是通过 Ajax 操作获取数据。可以看到，除了多了一个 yield，它几乎与同步操作的写法完全一样。注意，makeAjaxCall 函数中的 next 方法，必须加上 response 参数，因为 yield 语句构成的表达式，本身是没有值的，总是等于 undefined。

下面是另一个例子，通过 Generator 函数逐行读取文本文件。

```js
function* numbers() {
  let file = new FileReader("numbers.txt");
  try {
    while(!file.eof) {
      yield parseInt(file.readLine(), 10);
    }
  } finally {
    file.close();
  }
}

```

上面代码打开文本文件，使用 yield 语句可以手动逐行读取文件。

### （2）控制流管理

如果有一个多步操作非常耗时，采用回调函数，可能会写成下面这样。

```js
step1(function (value1) {
  step2(value1, function(value2) {
    step3(value2, function(value3) {
      step4(value3, function(value4) {
        // Do something with value4
      });
    });
  });
});

```

采用 Promise 改写上面的代码。

```js
Q.fcall(step1)
  .then(step2)
  .then(step3)
  .then(step4)
  .then(function (value4) {
    // Do something with value4
  }, function (error) {
    // Handle any error from step1 through step4
  })
  .done();

```

上面代码已经把回调函数，改成了直线执行的形式，但是加入了大量 Promise 的语法。Generator 函数可以进一步改善代码运行流程。

```js
function* longRunningTask() {
  try {
    var value1 = yield step1();
    var value2 = yield step2(value1);
    var value3 = yield step3(value2);
    var value4 = yield step4(value3);
    // Do something with value4
  } catch (e) {
    // Handle any error from step1 through step4
  }
}

```

然后，使用一个函数，按次序自动执行所有步骤。

```js
scheduler(longRunningTask());

function scheduler(task) {
  setTimeout(function() {
    var taskObj = task.next(task.value);
    // 如果 Generator 函数未结束，就继续调用
    if (!taskObj.done) {
      task.value = taskObj.value
      scheduler(task);
    }
  }, 0);
}

```

注意，yield 语句是同步运行，不是异步运行（否则就失去了取代回调函数的设计目的了）。实际操作中，一般让 yield 语句返回 Promise 对象。

```js
var Q = require('q');

function delay(milliseconds) {
  var deferred = Q.defer();
  setTimeout(deferred.resolve, milliseconds);
  return deferred.promise;
}

function* f(){
  yield delay(100);
};

```

上面代码使用 Promise 的函数库 Q，yield 语句返回的就是一个 Promise 对象。

多个任务按顺序一个接一个执行时，yield 语句可以按顺序排列。多个任务需要并列执行时（比如只有 A 任务和 B 任务都执行完，才能执行 C 任务），可以采用数组的写法。

```js
function* parallelDownloads() {
  let [text1,text2] = yield [
    taskA(),
    taskB()
  ];
  console.log(text1, text2);
}

```

上面代码中，yield 语句的参数是一个数组，成员就是两个任务 taskA 和 taskB，只有等这两个任务都完成了，才会接着执行下面的语句。

### （3）部署 iterator 接口

利用 Generator 函数，可以在任意对象上部署 iterator 接口。

```js
function* iterEntries(obj) {
  let keys = Object.keys(obj);
  for (let i=0; i < keys.length; i++) {
    let key = keys[i];
    yield [key, obj[key]];
  }
}

let myObj = { foo: 3, bar: 7 };

for (let [key, value] of iterEntries(myObj)) {
  console.log(key, value);
}

// foo 3
// bar 7

```

上述代码中，myObj 是一个普通对象，通过 iterEntries 函数，就有了 iterator 接口。也就是说，可以在任意对象上部署 next 方法。

下面是一个对数组部署 Iterator 接口的例子，尽管数组原生具有这个接口。

```js
function* makeSimpleGenerator(array){
  var nextIndex = 0;

  while(nextIndex < array.length){
    yield array[nextIndex++];
  }
}

var gen = makeSimpleGenerator(['yo', 'ya']);

gen.next().value // 'yo'
gen.next().value // 'ya'
gen.next().done  // true

```

### （4）作为数据结构

Generator 可以看作是数据结构，更确切地说，可以看作是一个数组结构，因为 Generator 函数可以返回一系列的值，这意味着它可以对任意表达式，提供类似数组的接口。

```js
function *doStuff() {
  yield fs.readFile.bind(null, 'hello.txt');
  yield fs.readFile.bind(null, 'world.txt');
  yield fs.readFile.bind(null, 'and-such.txt');
}

```

上面代码就是依次返回三个函数，但是由于使用了 Generator 函数，导致可以像处理数组那样，处理这三个返回的函数。

```js
for (task of doStuff()) {
  // task 是一个函数，可以像回调函数那样使用它
}

```

实际上，如果用 ES5 表达，完全可以用数组模拟 Generator 的这种用法。

```js
function doStuff() {
  return [
    fs.readFile.bind(null, 'hello.txt'),
    fs.readFile.bind(null, 'world.txt'),
    fs.readFile.bind(null, 'and-such.txt')
  ];
}

```

上面的函数，可以用一模一样的 for...of 循环处理！两相一比较，就不难看出 Generator 使得数据或者操作，具备了类似数组的接口。