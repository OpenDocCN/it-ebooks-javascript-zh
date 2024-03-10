# 核心

# 核心

## 为什么不要使用 `eval`

`eval` 函数会在当前作用域中执行一段 JavaScript 代码字符串。

```
var foo = 1;
function test() {
    var foo = 2;
    eval('foo = 3');
    return foo;
}
test(); // 3
foo; // 1 
```

但是 `eval` 只在被**直接**调用并且调用函数就是 `eval` 本身时，才在当前作用域中执行。

```
var foo = 1;
function test() {
    var foo = 2;
    var bar = eval;
    bar('foo = 3');
    return foo;
}
test(); // 2
foo; // 3 
```

**[译者注](http://cnblogs.com/sanshi/)：**上面的代码等价于在全局作用域中调用 `eval`，和下面两种写法效果一样：

```
// 写法一：直接调用全局作用域下的 foo 变量
var foo = 1;
function test() {
    var foo = 2;
    window.foo = 3;
    return foo;
}
test(); // 2
foo; // 3

// 写法二：使用 call 函数修改 eval 执行的上下文为全局作用域
var foo = 1;
function test() {
    var foo = 2;
    eval.call(window, 'foo = 3');
    return foo;
}
test(); // 2
foo; // 3 
```

在**任何情况下**我们都应该避免使用 `eval` 函数。99.9% 使用 `eval` 的场景都有**不使用** `eval` 的解决方案。

### 伪装的 `eval`

定时函数 `setTimeout` 和 `setInterval` 都可以接受字符串作为它们的第一个参数。 这个字符串**总是**在全局作用域中执行，因此 `eval` 在这种情况下没有被直接调用。

### 安全问题

`eval` 也存在安全问题，因为它会执行**任意**传给它的代码， 在代码字符串未知或者是来自一个不信任的源时，绝对不要使用 `eval` 函数。

### 结论

绝对不要使用 `eval`，任何使用它的代码都会在它的工作方式，性能和安全性方面受到质疑。 如果一些情况必须使用到 `eval` 才能正常工作，首先它的设计会受到质疑，这**不应该**是首选的解决方案， 一个更好的不使用 `eval` 的解决方案应该得到充分考虑并优先采用。

## `undefined` 和 `null`

JavaScript 有两个表示‘空’的值，其中比较有用的是 `undefined`。

### `undefined` 的值

`undefined` 是一个值为 `undefined` 的类型。

这个语言也定义了一个全局变量，它的值是 `undefined`，这个变量也被称为 `undefined`。 但是这个变量**不是**一个常量，也不是一个关键字。这意味着它的*值*可以轻易被覆盖。

**ES5 提示:** 在 ECMAScript 5 的严格模式下，`undefined` **不再是** *可写*的了。 但是它的名称仍然可以被隐藏，比如定义一个函数名为 `undefined`。

下面的情况会返回 `undefined` 值：

*   访问未修改的全局变量 `undefined`。
*   由于没有定义 `return` 表达式的函数隐式返回。
*   `return` 表达式没有显式的返回任何内容。
*   访问不存在的属性。
*   函数参数没有被显式的传递值。
*   任何被设置为 `undefined` 值的变量。

### 处理 `undefined` 值的改变

由于全局变量 `undefined` 只是保存了 `undefined` 类型实际*值*的副本， 因此对它赋新值**不会**改变类型 `undefined` 的值。

然而，为了方便其它变量和 `undefined` 做比较，我们需要事先获取类型 `undefined` 的值。

为了避免可能对 `undefined` 值的改变，一个常用的技巧是使用一个传递到匿名包装器的额外参数。 在调用时，这个参数不会获取任何值。

```
var undefined = 123;
(function(something, foo, undefined) {
    // 局部作用域里的 undefined 变量重新获得了 `undefined` 值

})('Hello World', 42); 
```

另外一种达到相同目的方法是在函数内使用变量声明。

```
var undefined = 123;
(function(something, foo) {
    var undefined;
    ...

})('Hello World', 42); 
```

这里唯一的区别是，在压缩后并且函数内没有其它需要使用 `var` 声明变量的情况下，这个版本的代码会多出 4 个字节的代码。

**[译者注](http://cnblogs.com/sanshi/)：**这里有点绕口，其实很简单。如果此函数内没有其它需要声明的变量，那么 `var` 总共 4 个字符（包含一个空白字符） 就是专门为 `undefined` 变量准备的，相比上个例子多出了 4 个字节。

### `null` 的用处

JavaScript 中的 `undefined` 的使用场景类似于其它语言中的 *null*，实际上 JavaScript 中的 `null` 是另外一种数据类型。

它在 JavaScript 内部有一些使用场景（比如声明原型链的终结 `Foo.prototype = null`），但是大多数情况下都可以使用 `undefined` 来代替。

## 自动分号插入

尽管 JavaScript 有 C 的代码风格，但是它**不**强制要求在代码中使用分号，实际上可以省略它们。

JavaScript 不是一个没有分号的语言，恰恰相反上它需要分号来就解析源代码。 因此 JavaScript 解析器在遇到由于缺少分号导致的解析错误时，会**自动**在源代码中插入分号。

```
var foo = function() {
} // 解析错误，分号丢失
test() 
```

自动插入分号，解析器重新解析。

```
var foo = function() {
}; // 没有错误，解析继续
test() 
```

自动的分号插入被认为是 JavaScript 语言**最大**的设计缺陷之一，因为它*能*改变代码的行为。

### 工作原理

下面的代码没有分号，因此解析器需要自己判断需要在哪些地方插入分号。

```
(function(window, undefined) {
    function test(options) {
        log('testing!')

        (options.list || []).forEach(function(i) {

        })

        options.value.test(
            'long string to pass here',
            'and another long string to pass'
        )

        return
        {
            foo: function() {}
        }
    }
    window.test = test

})(window)

(function(window) {
    window.someLibrary = {}
})(window) 
```

下面是解析器"猜测"的结果。

```
(function(window, undefined) {
    function test(options) {

        // 没有插入分号，两行被合并为一行
        log('testing!')(options.list || []).forEach(function(i) {

        }); // <- 插入分号

        options.value.test(
            'long string to pass here',
            'and another long string to pass'
        ); // <- 插入分号

        return; // <- 插入分号, 改变了 return 表达式的行为
        { // 作为一个代码段处理
            foo: function() {} 
        }; // <- 插入分号
    }
    window.test = test; // <- 插入分号

// 两行又被合并了
})(window)(function(window) {
    window.someLibrary = {}; // <- 插入分号
})(window); //<- 插入分号 
```

**注意:** JavaScript 不能正确的处理 `return` 表达式紧跟换行符的情况， 虽然这不能算是自动分号插入的错误，但这确实是一种不希望的副作用。

解析器显著改变了上面代码的行为，在另外一些情况下也会做出**错误的处理**。

### 前置括号

在前置括号的情况下，解析器**不会**自动插入分号。

```
log('testing!')
(options.list || []).forEach(function(i) {}) 
```

上面代码被解析器转换为一行。

```
log('testing!')(options.list || []).forEach(function(i) {}) 
```

`log` 函数的执行结果**极大**可能**不是**函数；这种情况下就会出现 `TypeError` 的错误，详细错误信息可能是 `undefined is not a function`。

### 结论

建议**绝对**不要省略分号，同时也提倡将花括号和相应的表达式放在一行， 对于只有一行代码的 `if` 或者 `else` 表达式，也不应该省略花括号。 这些良好的编程习惯不仅可以提到代码的一致性，而且可以防止解析器改变代码行为的错误处理。