# 对象

# 对象

## 对象使用和属性

JavaScript 中所有变量都可以当作对象使用，除了两个例外 `null` 和 `undefined`。

```
false.toString(); // 'false'
[1, 2, 3].toString(); // '1,2,3'

function Foo(){}
Foo.bar = 1;
Foo.bar; // 1 
```

一个常见的误解是数字的字面值（literal）不能当作对象使用。这是因为 JavaScript 解析器的一个错误， 它试图将*点操作符*解析为浮点数字面值的一部分。

```
2.toString(); // 出错：SyntaxError 
```

有很多变通方法可以让数字的字面值看起来像对象。

```
2..toString(); // 第二个点号可以正常解析
2 .toString(); // 注意点号前面的空格
(2).toString(); // 2 先被计算 
```

### 对象作为数据类型

JavaScript 的对象可以作为[*哈希表*](http://en.wikipedia.org/wiki/Hashmap)使用，主要用来保存命名的键与值的对应关系。

使用对象的字面语法 - `{}` - 可以创建一个简单对象。这个新创建的对象从 `Object.prototype` 继承下面，没有任何自定义属性。

```
var foo = {}; // 一个空对象

// 一个新对象，拥有一个值为 12 的自定义属性'test'
var bar = {test: 12}; 
```

### 访问属性

有两种方式来访问对象的属性，点操作符或者中括号操作符。

```
var foo = {name: 'kitten'}
foo.name; // kitten
foo['name']; // kitten

var get = 'name';
foo[get]; // kitten

foo.1234; // SyntaxError
foo['1234']; // works 
```

两种语法是等价的，但是中括号操作符在下面两种情况下依然有效

*   动态设置属性
*   属性名不是一个有效的变量名（**[译者注](http://cnblogs.com/sanshi/)：**比如属性名中包含空格，或者属性名是 JS 的关键词）

**[译者注](http://cnblogs.com/sanshi/)：**在 [JSLint](http://www.jslint.com/) 语法检测工具中，点操作符是推荐做法。

### 删除属性

删除属性的唯一方法是使用 `delete` 操作符；设置属性为 `undefined` 或者 `null` 并不能真正的删除属性， 而**仅仅**是移除了属性和值的关联。

```
var obj = {
    bar: 1,
    foo: 2,
    baz: 3
};
obj.bar = undefined;
obj.foo = null;
delete obj.baz;

for(var i in obj) {
    if (obj.hasOwnProperty(i)) {
        console.log(i, '' + obj[i]);
    }
} 
```

上面的输出结果有 `bar undefined` 和 `foo null` - 只有 `baz` 被真正的删除了，所以从输出结果中消失。

### 属性名的语法

```
var test = {
    'case': 'I am a keyword so I must be notated as a string',
    delete: 'I am a keyword too so me' // 出错：SyntaxError
}; 
```

对象的属性名可以使用字符串或者普通字符声明。但是由于 JavaScript 解析器的另一个错误设计， 上面的第二种声明方式在 ECMAScript 5 之前会抛出 `SyntaxError` 的错误。

这个错误的原因是 `delete` 是 JavaScript 语言的一个*关键词*；因此为了在更低版本的 JavaScript 引擎下也能正常运行， 必须使用*字符串字面值*声明方式。

## 原型

JavaScript 不包含传统的类继承模型，而是使用 *prototype* 原型模型。

虽然这经常被当作是 JavaScript 的缺点被提及，其实基于原型的继承模型比传统的类继承还要强大。 实现传统的类继承模型是很简单，但是实现 JavaScript 中的原型继承则要困难的多。 (It is for example fairly trivial to build a classic model on top of it, while the other way around is a far more difficult task.)

由于 JavaScript 是唯一一个被广泛使用的基于原型继承的语言，所以理解两种继承模式的差异是需要一定时间的。

第一个不同之处在于 JavaScript 使用*原型链*的继承方式。

**注意:** 简单的使用 `Bar.prototype = Foo.prototype` 将会导致两个对象共享**相同**的原型。 因此，改变任意一个对象的原型都会影响到另一个对象的原型，在大多数情况下这不是希望的结果。

```
function Foo() {
    this.value = 42;
}
Foo.prototype = {
    method: function() {}
};

function Bar() {}

// 设置 Bar 的 prototype 属性为 Foo 的实例对象
Bar.prototype = new Foo();
Bar.prototype.foo = 'Hello World';

// 修正 Bar.prototype.constructor 为 Bar 本身
Bar.prototype.constructor = Bar;

var test = new Bar() // 创建 Bar 的一个新实例

// 原型链
test [Bar 的实例]
    Bar.prototype [Foo 的实例] 
        { foo: 'Hello World' }
        Foo.prototype
            {method: ...};
            Object.prototype
                {toString: ... /* etc. */}; 
```

上面的例子中，`test` 对象从 `Bar.prototype` 和 `Foo.prototype` 继承下来；因此， 它能访问 `Foo` 的原型方法 `method`。同时，它也能够访问**那个**定义在原型上的 `Foo` 实例属性 `value`。 需要注意的是 `new Bar()` **不会**创造出一个新的 `Foo` 实例，而是 重复使用它原型上的那个实例；因此，所有的 `Bar` 实例都会共享**相同**的 `value` 属性。

**注意:** **不要**使用 `Bar.prototype = Foo`，因为这不会执行 `Foo` 的原型，而是指向函数 `Foo`。 因此原型链将会回溯到 `Function.prototype` 而不是 `Foo.prototype`，因此 `method` 将不会在 Bar 的原型链上。

### 属性查找

当查找一个对象的属性时，JavaScript 会**向上**遍历原型链，直到找到给定名称的属性为止。

到查找到达原型链的顶部 - 也就是 `Object.prototype` - 但是仍然没有找到指定的属性，就会返回 undefined。

### 原型属性

当原型属性用来创建原型链时，可以把**任何**类型的值赋给它（prototype）。 然而将原子类型赋给 prototype 的操作将会被忽略。

```
function Foo() {}
Foo.prototype = 1; // 无效 
```

而将对象赋值给 prototype，正如上面的例子所示，将会动态的创建原型链。

### 性能

如果一个属性在原型链的上端，则对于查找时间将带来不利影响。特别的，试图获取一个不存在的属性将会遍历整个原型链。

并且，当使用 `for in` 循环遍历对象的属性时，原型链上的**所有**属性都将被访问。

### 扩展内置类型的原型

一个错误特性被经常使用，那就是扩展 `Object.prototype` 或者其他内置类型的原型对象。

这种技术被称之为 [monkey patching](http://en.wikipedia.org/wiki/Monkey_patch) 并且会破坏*封装*。虽然它被广泛的应用到一些 JavaScript 类库中比如 [Prototype](http://prototypejs.org/), 但是我仍然不认为为内置类型添加一些*非标准*的函数是个好主意。

扩展内置类型的**唯一**理由是为了和新的 JavaScript 保持一致，比如 [`Array.forEach`](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Array/forEach)。

**[译者注](http://cnblogs.com/sanshi/)：**这是编程领域常用的一种方式，称之为 [Backport](http://en.wikipedia.org/wiki/Backport)，也就是将新的补丁添加到老版本中。

### 总结

在写复杂的 JavaScript 应用之前，充分理解原型链继承的工作方式是每个 JavaScript 程序员**必修**的功课。 要提防原型链过长带来的性能问题，并知道如何通过缩短原型链来提高性能。 更进一步，绝对**不要**扩展内置类型的原型，除非是为了和新的 JavaScript 引擎兼容。

## `hasOwnProperty` 函数

为了判断一个对象是否包含*自定义*属性而*不是*原型链上的属性， 我们需要使用继承自 `Object.prototype` 的 `hasOwnProperty` 方法。

**注意:** 通过判断一个属性是否 `undefined` 是**不够**的。 因为一个属性可能确实存在，只不过它的值被设置为 `undefined`。

`hasOwnProperty` 是 JavaScript 中唯一一个处理属性但是**不**查找原型链的函数。

```
// 修改 Object.prototype
Object.prototype.bar = 1; 
var foo = {goo: undefined};

foo.bar; // 1
'bar' in foo; // true

foo.hasOwnProperty('bar'); // false
foo.hasOwnProperty('goo'); // true 
```

只有 `hasOwnProperty` 可以给出正确和期望的结果，这在遍历对象的属性时会很有用。 **没有**其它方法可以用来排除原型链上的属性，而不是定义在对象*自身*上的属性。

### `hasOwnProperty` 作为属性

JavaScript **不会**保护 `hasOwnProperty` 被非法占用，因此如果一个对象碰巧存在这个属性， 就需要使用*外部*的 `hasOwnProperty` 函数来获取正确的结果。

```
var foo = {
    hasOwnProperty: function() {
        return false;
    },
    bar: 'Here be dragons'
};

foo.hasOwnProperty('bar'); // 总是返回 false

// 使用其它对象的 hasOwnProperty，并将其上下文设置为 foo
({}).hasOwnProperty.call(foo, 'bar'); // true 
```

### 结论

当检查对象上某个属性是否存在时，`hasOwnProperty` 是**唯一**可用的方法。 同时在使用 `for in` loop 遍历对象时，推荐**总是**使用 `hasOwnProperty` 方法， 这将会避免原型对象扩展带来的干扰。

## `for in` 循环

和 `in` 操作符一样，`for in` 循环同样在查找对象属性时遍历原型链上的所有属性。

**注意:** `for in` 循环**不会**遍历那些 `enumerable` 设置为 `false` 的属性；比如数组的 `length` 属性。

```
// 修改 Object.prototype
Object.prototype.bar = 1;

var foo = {moo: 2};
for(var i in foo) {
    console.log(i); // 输出两个属性：bar 和 moo
} 
```

由于不可能改变 `for in` 自身的行为，因此有必要过滤出那些不希望出现在循环体中的属性， 这可以通过 `Object.prototype` 原型上的 `hasOwnProperty` 函数来完成。

**注意:** 由于 `for in` 总是要遍历整个原型链，因此如果一个对象的继承层次太深的话会影响性能。

### 使用 `hasOwnProperty` 过滤

```
// foo 变量是上例中的
for(var i in foo) {
    if (foo.hasOwnProperty(i)) {
        console.log(i);
    }
} 
```

这个版本的代码是唯一正确的写法。由于我们使用了 `hasOwnProperty`，所以这次**只**输出 `moo`。 如果不使用 `hasOwnProperty`，则这段代码在原生对象原型（比如 `Object.prototype`）被扩展时可能会出错。

一个广泛使用的类库 [Prototype](http://www.prototypejs.org/) 就扩展了原生的 JavaScript 对象。 因此，当这个类库被包含在页面中时，不使用 `hasOwnProperty` 过滤的 `for in` 循环难免会出问题。

### 总结

推荐**总是**使用 `hasOwnProperty`。不要对代码运行的环境做任何假设，不要假设原生对象是否已经被扩展了。