# 数字

JavaScript 的数字只有一种类型，不需要区分不同的类型，比如整型、短整型、长整型、浮点型等等。一个数字可以是 **浮点型** (例:1.23) 也可以是 **整型** (例: 10)。

这一点都不神奇。你可以定义变量并设置为任何类型。

在这章节，我们将学习如何创建数字和使用运算符(比如加减)。

# 创建

创建一个数字很容易，创建任何类型都只需要使用关键词 `var`。

数字可以被创建来自一个常量：

```js
// 这是浮点型:
var a = 1.2;

// 这是整型:
var b = 10; 
```

或来自另一个变量：

```js
var a = 2;
var b = a; 
```

Exercise 创建一个值为 `10` 的变量 `x` ， 创建一个值等于 `a` 的变量 `y`。

```js
var a = 11;
```

# 基本运算符

你可以对数字使用一些简单的数学运算符比如：

*   **加**: `c = a + b`
*   **减**: `c = a - b`
*   **乘**: `c = a * b`
*   **除**: `c = a / b`

你可以像在数学中一样，使用括号分隔进行分隔比如：`c = (a / b) + d`

Exercise 创建一个变量 `x` ，它的值为`a` 和 `b` 之和再被 `c` 除，最后乘上 `d`.

```js
var a = 2034547;
var b = 1.567;
var c = 6758.768;
var d = 45084;

var x =
```

# 高级运算符

一些高级的运算符可以这样用，比如：

*   **求余 (除法的余数)**: `x = y % 2`
*   **累加**: 让 a = 5
    *   `c = a++`, 结果: c = 5 和 a = 6
    *   `c = ++a`, 结果: c = 6 和 a = 6
*   **递减**: 让 a = 5
    *   `c = a--`, 结果: c = 5 和 a = 4
    *   `c = --a`, 结果: c = 4 和 a = 4

Exercise 定义一个变量 `c` 作为自减变量 `x` 对 3 的模。

```js
var x = 10;

var c =
```