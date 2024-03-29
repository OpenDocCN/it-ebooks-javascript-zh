# 类型

## 相等与比较

JavaScript 有两种方式判断两个值是否相等。

### 等于操作符

等于操作符由两个等号组成：`==`

JavaScript 是*弱类型*语言，这就意味着，等于操作符会为了比较两个值而进行**强制类型转换**。

```js
""           ==   "0"           // false
0            ==   ""            // true
0            ==   "0"           // true
false        ==   "false"       // false
false        ==   "0"           // true
false        ==   undefined     // false
false        ==   null          // false
null         ==   undefined     // true
" \t\r\n"    ==   0             // true 
```

上面的表格展示了强制类型转换，这也是使用 `==` 被广泛认为是不好编程习惯的主要原因， 由于它的复杂转换规则，会导致难以跟踪的问题。

此外，强制类型转换也会带来性能消耗，比如一个字符串为了和一个数字进行比较，必须事先被强制转换为数字。

### 严格等于操作符

严格等于操作符由**三**个等号组成：`===`

不像普通的等于操作符，严格等于操作符**不会**进行强制类型转换。

```js
""           ===   "0"           // false
0            ===   ""            // false
0            ===   "0"           // false
false        ===   "false"       // false
false        ===   "0"           // false
false        ===   undefined     // false
false        ===   null          // false
null         ===   undefined     // false
" \t\r\n"    ===   0             // false 
```

上面的结果更加清晰并有利于代码的分析。如果两个操作数类型不同就肯定不相等也有助于性能的提升。

### 比较对象

虽然 `==` 和 `===` 操作符都是等于操作符，但是当其中有一个操作数为对象时，行为就不同了。

```js
{} === {};                   // false
new String('foo') === 'foo'; // false
new Number(10) === 10;       // false
var foo = {};
foo === foo;                 // true 
```

这里等于操作符比较的**不是**值是否相等，而是是否属于同一个**身份**；也就是说，只有对象的同一个实例才被认为是相等的。 这有点像 Python 中的 `is` 和 C 中的指针比较。

**注意:**为了更直观的看到`==`和`===`的区别,可以参见[JavaScript Equality Table](http://dorey.github.io/JavaScript-Equality-Table/)

### 结论

强烈推荐使用**严格等于操作符**。如果类型需要转换，应该在比较之前显式的转换， 而不是使用语言本身复杂的强制转换规则。

## `typeof` 操作符

`typeof` 操作符（和 `instanceof` 一起）或许是 JavaScript 中最大的设计缺陷， 因为几乎不可能从它们那里得到想要的结果。

尽管 `instanceof` 还有一些极少数的应用场景，`typeof` 只有一个实际的应用（**[译者注](http://cnblogs.com/sanshi/)：**这个实际应用是用来检测一个对象是否已经定义或者是否已经赋值）， 而这个应用却**不是**用来检查对象的类型。

**注意:** 由于 `typeof` 也可以像函数的语法被调用，比如 `typeof(obj)`，但这并不是一个函数调用。 那两个小括号只是用来计算一个表达式的值，这个返回值会作为 `typeof` 操作符的一个操作数。 实际上**不存在**名为 `typeof` 的函数。

### JavaScript 类型表格

```js
Value               Class      Type
-------------------------------------
"foo"               String     string
new String("foo")   String     object
1.2                 Number     number
new Number(1.2)     Number     object
true                Boolean    boolean
new Boolean(true)   Boolean    object
new Date()          Date       object
new Error()         Error      object
[1,2,3]             Array      object
new Array(1, 2, 3)  Array      object
new Function("")    Function   function
/abc/g              RegExp     object (function in Nitro/V8)
new RegExp("meow")  RegExp     object (function in Nitro/V8)
{}                  Object     object
new Object()        Object     object 
```

上面表格中，*Type* 一列表示 `typeof` 操作符的运算结果。可以看到，这个值在大多数情况下都返回 "object"。

*Class* 一列表示对象的内部属性 `[[Class]]` 的值。

**JavaScript 标准文档中定义:** `[[Class]]` 的值只可能是下面字符串中的一个： `Arguments`, `Array`, `Boolean`, `Date`, `Error`, `Function`, `JSON`, `Math`, `Number`, `Object`, `RegExp`, `String`.

为了获取对象的 `[[Class]]`，我们需要使用定义在 `Object.prototype` 上的方法 `toString`。

### 对象的类定义

JavaScript 标准文档只给出了一种获取 `[[Class]]` 值的方法，那就是使用 `Object.prototype.toString`。

```js
function is(type, obj) {
    var clas = Object.prototype.toString.call(obj).slice(8, -1);
    return obj !== undefined && obj !== null && clas === type;
}

is('String', 'test'); // true
is('String', new String('test')); // true 
```

上面例子中，`Object.prototype.toString` 方法被调用，this 被设置为了需要获取 `[[Class]]` 值的对象。

**[译者注](http://cnblogs.com/sanshi/)：**`Object.prototype.toString` 返回一种标准格式字符串，所以上例可以通过 `slice` 截取指定位置的字符串，如下所示：

```js
Object.prototype.toString.call([])    // "[object Array]"
Object.prototype.toString.call({})    // "[object Object]"
Object.prototype.toString.call(2)    // "[object Number]" 
```

**ES5 提示:** 在 ECMAScript 5 中，为了方便，对 `null` 和 `undefined` 调用 `Object.prototype.toString` 方法， 其返回值由 `Object` 变成了 `Null` 和 `Undefined`。

**[译者注](http://cnblogs.com/sanshi/)：**这种变化可以从 IE8 和 Firefox 4 中看出区别，如下所示：

```js
// IE8
Object.prototype.toString.call(null)    // "[object Object]"
Object.prototype.toString.call(undefined)    // "[object Object]"

// Firefox 4
Object.prototype.toString.call(null)    // "[object Null]"
Object.prototype.toString.call(undefined)    // "[object Undefined]" 
```

### 测试为定义变量

```js
typeof foo !== 'undefined' 
```

上面代码会检测 `foo` 是否已经定义；如果没有定义而直接使用会导致 `ReferenceError` 的异常。 这是 `typeof` 唯一有用的地方。

### 结论

为了检测一个对象的类型，强烈推荐使用 `Object.prototype.toString` 方法； 因为这是唯一一个可依赖的方式。正如上面表格所示，`typeof` 的一些返回值在标准文档中并未定义， 因此不同的引擎实现可能不同。

除非为了检测一个变量是否已经定义，我们应尽量避免使用 `typeof` 操作符。

## `instanceof` 操作符

`instanceof` 操作符用来比较两个操作数的构造函数。只有在比较自定义的对象时才有意义。 如果用来比较内置类型，将会和 `typeof` 操作符 一样用处不大。

### 比较自定义对象

```js
function Foo() {}
function Bar() {}
Bar.prototype = new Foo();

new Bar() instanceof Bar; // true
new Bar() instanceof Foo; // true

// 如果仅仅设置 Bar.prototype 为函数 Foo 本身，而不是 Foo 构造函数的一个实例
Bar.prototype = Foo;
new Bar() instanceof Foo; // false 
```

### `instanceof` 比较内置类型

```js
new String('foo') instanceof String; // true
new String('foo') instanceof Object; // true

'foo' instanceof String; // false
'foo' instanceof Object; // false 
```

有一点需要注意，`instanceof` 用来比较属于不同 JavaScript 上下文的对象（比如，浏览器中不同的文档结构）时将会出错， 因为它们的构造函数不会是同一个对象。

### 结论

`instanceof` 操作符应该**仅仅**用来比较来自同一个 JavaScript 上下文的自定义对象。 正如 `typeof` 操作符一样，任何其它的用法都应该是避免的。

## 类型转换

JavaScript 是*弱类型*语言，所以会在**任何**可能的情况下应用*强制类型转换*。

```js
// 下面的比较结果是：true
new Number(10) == 10; // Number.toString() 返回的字符串被再次转换为数字

10 == '10';           // 字符串被转换为数字
10 == '+10 ';         // 同上
10 == '010';          // 同上 
isNaN(null) == false; // null 被转换为数字 0
                      // 0 当然不是一个 NaN（译者注：否定之否定）

// 下面的比较结果是：false
10 == 010;
10 == '-10'; 
```

**ES5 提示:** 以 `0` 开头的数字字面值会被作为八进制数字解析。 而在 ECMAScript 5 严格模式下，这个特性被**移除**了。

为了避免上面复杂的强制类型转换，**强烈**推荐使用严格的等于操作符。 虽然这可以避免大部分的问题，但 JavaScript 的弱类型系统仍然会导致一些其它问题。

### 内置类型的构造函数

内置类型（比如 `Number` 和 `String`）的构造函数在被调用时，使用或者不使用 `new` 的结果完全不同。

```js
new Number(10) === 10;     // False, 对象与数字的比较
Number(10) === 10;         // True, 数字与数字的比较
new Number(10) + 0 === 10; // True, 由于隐式的类型转换 
```

使用内置类型 `Number` 作为构造函数将会创建一个新的 `Number` 对象， 而在不使用 `new` 关键字的 `Number` 函数更像是一个数字转换器。

另外，在比较中引入对象的字面值将会导致更加复杂的强制类型转换。

最好的选择是把要比较的值**显式**的转换为三种可能的类型之一。

### 转换为字符串

```js
'' + 10 === '10'; // true 
```

将一个值加上空字符串可以轻松转换为字符串类型。

### 转换为数字

```js
+'10' === 10; // true 
```

使用**一元**的加号操作符，可以把字符串转换为数字。

**[译者注](http://cnblogs.com/sanshi/)：**字符串转换为数字的常用方法：

```js
+'010' === 10
Number('010') === 10
parseInt('010', 10) === 10  // 用来转换为整数

+'010.2' === 10.2
Number('010.2') === 10.2
parseInt('010.2', 10) === 10 
```

### 转换为布尔型

通过使用 **否** 操作符两次，可以把一个值转换为布尔型。

```js
!!'foo';   // true
!!'';      // false
!!'0';     // true
!!'1';     // true
!!'-1'     // true
!!{};      // true
!!true;    // true 
```