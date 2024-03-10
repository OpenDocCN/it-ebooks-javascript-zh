# 条件语句

# 条件逻辑

条件语句可以用来测试。条件判断在编程中非常重要，比如：

首先，无论程序运行使用什么数据，所有的条件都能被用来确定程序是否正常。如果盲目的相信数据，你将陷入程序出错的麻烦。如果测试有效并且所需信息格式正确，程序就不会出错，还会变得更稳定。防御编程就是保持警惕。

另一种条件判断的作用就是分支。你可能已经接触过分支图，比如填写表格时。基本上这指的是依赖 if 条件语句执行不同的代码分支。

在这个章节，我们将会学习 Javascript 中条件逻辑的基础。

# If

# If

最简单的条件判断是 if 语句，语法是 `if(condition){ do this … }` 。条件判断为真，才执行分支中的代码。举个字符串的例子：

```js
var country = 'France';
var weather;
var food;
var currency;

if(country === 'England') {
    weather = 'horrible';
    food = 'filling';
    currency = 'pound sterling';
}

if(country === 'France') {
    weather = 'nice';
    food = 'stunning, but hardly ever vegetarian';
    currency = 'funny, small and colourful';
}

if(country === 'Germany') {
    weather = 'average';
    food = 'wurst thing ever';
    currency = 'funny, small and colourful';
}

var message = 'this is ' + country + ', the weather is ' +
            weather + ', the food is ' + food + ' and the ' +
            'currency is ' + currency; 
```

**注意:** 条件判断可以嵌套。

Exercise 填写 `name` 的值，验证条件判断。

```js
var name =

if (name === "John") {

}
```

# Else

# Else

当第一个条件语句不成立时， `else` 语句将被执行。如果你想要在特殊条件下才返回一个值，这非常有效：

```js
var umbrellaMandatory;

if(country === 'England'){
    umbrellaMandatory = true;
} else {
    umbrellaMandatory = false;
} 
```

`else` 语句可以和另一个 `if` 语句结合。改造一下上面的例子：

```js
if(country === 'England') {
    ...
} else if(country === 'France') {
    ...
} else if(country === 'Germany') {
    ...
} 
```

Exercise 填写 `name` 的值，验证 `else` 语句。

```js
var name =

if (name === "John") {

} else if (name === "Aaron") {
    // Valid this condition
}
```

# 比较

# 比较

把焦点放在条件判断部分：

```js
if (country === "France") {
    ...
} 
```

变量 `country` 后面跟着的三个等号(`===`)是条件判断部分。三个等号测试是否变量 `country` 和 `France` 值与类型(`String`)相同。你也可以用两个等号来测试，比如`if (x == 5)`，在`var x = 5;` 或 `var x = &quot;5&quot;;` 情况下都返回真。这很不一样取决于你的程序是做什么。比较推荐你经常去尝试比较三个等号(`===` 和 `!==`)和两个等号(`==` 和 `!=`)的区别。

其他条件判断的测试：

*   `x &gt; a`: is x bigger than a?
*   `x &lt; a`: is x less than a?
*   `x &lt;= a`: is x less than or equal to a?
*   `x &gt;=a`: is x greater than or equal to a?
*   `x != a`: is x not a?
*   `x`: does x exist?

Exercise 添加一种条件判断，如果 `x` 比 5 大，使变量 `a` 赋值为 10。

```js
var x = 6;
var a = 0;
```

## 逻辑比较

为了避免 if-else 麻烦，可以利用一种简单的逻辑比较。

```js
var topper = (marks > 85) ? "YES" : "NO"; 
```

在上述例子中，`?` 是逻辑运算符。上述源码表示如果 marks 的值大于 85 即 `marks &gt; 85` ，则 `topper = YES` ；否则 `topper = NO` 。基本上，如果比较条件为真，赋第一个参数的值，否则赋的二哥参数的值。

# 条件连接

# 条件连接

此外，你可以用 "or” 或 “and” 语句连接不同的条件判断，可以分别的测试是否存在一个为真或同为真。

在 Javascript 中，“or” 可以被写成 `||` ， “and” 可以被写成 `&amp;&amp;`。

比如你想要测试 x 的值是否在 10 到 20 之间，你可以用上述的方法：

```js
if(x > 10 && x < 20) {
    ...
} 
```

如果你想要确认 country 是 “England” 或 “Germany”：

```js
if(country === 'England' || country === 'Germany') {
    ...
} 
```

**注意**: 就像对数字的运算符，条件可以用括号来分组，比如： `if ( (name === &quot;John&quot; || name === &quot;Jennifer&quot;) &amp;&amp; country === &quot;France&quot;)`。

Exercise 填写两个条件让仅当 name 为 `"John"` ，country 为 `"England"`，`primaryCategory` 才等于 `"E/J"` ，仅当 name 为 `"John"` 或 country 为 `"England"`， `secondaryCategory` 才等于 `"E/J"` 。

```js
var name = "John";
var country = "England";
var primaryCategory, secondaryCategory;

if ( /* Fill here */ ) {
    primaryCategory = "E/J";
}
if ( /* Fill here */ ) {
    secondaryCategory = "E|J";
}
```