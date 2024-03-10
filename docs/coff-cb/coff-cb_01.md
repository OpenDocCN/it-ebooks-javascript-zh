# 一、语法

## 服务端和客户端的代码重用

### 问题

当你在 CoffeeScript 上创建了一个函数，并希望将它用在有网页浏览器的客户端和有 Node.js 的服务端时。

### 解决方案

以下列方法输出函数：

```js
 # simpleMath.coffee

 # these methods are private

add = (a, b) ->
    a + b

subtract = (a, b) ->
    a - b

square = (x) ->
    x * x

 # create a namespace to export our public methods

SimpleMath = exports? and exports or @SimpleMath = {}

 # items attached to our namespace are available in Node.js as well as client browsers

class SimpleMath.Calculator
    add: add
    subtract: subtract
    square: square
```

### 讨论

在上面的例子中，我们创建了一个新的名为 “SimpleMath” 的命名空间。如果 “export” 是有效的，我们的类就会作为一个 Node.js 模块输出。如果 “export” 是无效的，那么 “SimpleMath” 就会被加入全局命名空间，这样就可以被我们的网页使用了。

在 Node.js 中，我们可以使用 “require” 命令包含我们的模块。

```js
$ node

> var SimpleMath = require('./simpleMath');
undefined
> var Calc = new SimpleMath.Calculator();
undefined
> console.log("5 + 6 = ", Calc.add(5, 6));
5 + 6 =  11
undefined
>
```

在网页中，我们可以通过将模块作为一个脚本嵌入其中。

```js
<!DOCTYPE HTML>
<html lang="en-US">
<head>
    <meta charset="UTF-8">
    <title>SimpleMath Module Example</title>
    <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.min.js"></script>
    <script src="simpleMath.js"></script>
    <script>
        jQuery(document).ready(function    (){
            var Calculator = new SimpleMath.Calculator();
            var result = $('<li>').html("5 + 6 = " + Calculator.add(5, 6));
            $('#SampleResults').append(result); 
        });
    </script>
</head>
<body>
    <h1>A SimpleMath Example</h1>
    <ul id="SampleResults"></ul>
</body>
</html>
```

输出结果：

**A SimpleMath Example**

**· 5 + 6 = 11**

## 比较范围

### 问题

如果你想知道某个变量是否在给定的范围内。

### 解决方案

使用 CoffeeScript 的连缀比较语法。

```js
maxDwarfism = 147
minAcromegaly = 213

height = 180

normalHeight = maxDwarfism < height < minAcromegaly
 # => true
```

### 讨论

这是从 Python 中借鉴过来的一个很棒的特性。利用这个特性，不必像下面这样写出完整的比较：

```js
    normalHeight = height > maxDwarfism && height < minAcromegaly
```

CoffeeScript 支持像写数学中的比较表达式一样连缀两个比较，这样更直观。

## 嵌入 JavaScript

### 问题

你想在 CoffeeScript 中嵌入找到的或预先编写的 JavaScript 代码。

### 解决方案

把 JavaScript 包装到撇号中：

```js
`function greet(name) {
return "Hello "+name;
}`

 # Back to CoffeeScript

greet "Coffee"
 # => "Hello Coffee"
```

### 讨论

这是在 CoffeeScript 代码中集成少量 JavaScript 而不必用 CoffeeScript 语法转换它们的最简单的方法。正如 [CoffeeScript Language Reference](http://jashkenas.github.com/coffee-script/#embedded) 中展示的，可以在一定范围内混合这两种语言的代码：

```js
hello = `function (name) {
return "Hello "+name
}`
hello "Coffee"
 # => "Hello Coffee"
```

这里的变量 "hello" 还在 CoffeeScript 中，但赋给它的函数则是用 JavaScript 写的。

## For 循环

### 问题

你想通过一个 for 循环来迭代数组、对象或范围。

### 解决方案

```js
 # for(i = 1; i<= 10; i++)

x for x in [1..10]
 # => [ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 ]

 # To count by 2

 # for(i=1; i<= 10; i=i+2)

x for x in [1..10] by 2
 # => [ 1, 3, 5, 7, 9 ]

 # Perform a simple operation like squaring each item.

x * x for x in [1..10]
 # = > [1,4,9,16,25,36,49,64,81,100]
```

### 讨论

CoffeeScript 使用推导（comprehension）来代替 for 循环，这些推导最终会被编译成 JavaScript 中等价的 for 循环。