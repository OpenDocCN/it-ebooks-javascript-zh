# 数组

# 数组

数组是编程的基础部分。一个数组就是一系列数据。我们可以储存许多数据在一个变量中，这提高了代码可读性，让人更好理解代码。这使相关数据传递到函数中执行更简单。

数组中数据称为 **元素**。

这是一个简单的数组：

```js
// 1, 1, 2, 3, 5 和 8 是数组中的元素
var numbers = [1, 1, 2, 3, 5, 8]; 
```

# 索引

# 索引

若你有一个数组，如何访问想要的特定元素？索引出现了。一个 **下标** 指向数组中的一个位置。正如其他大部分语言，索引逻辑上是一个接一个，但要注意第一个数组下标是 0。方括号[]被用来表示使用数组下标。

```js
// 这是字符串构成的数组
var fruits = ["apple", "banana", "pineapple", "strawberry"];

// 用 fruits 数组中的第二个元素的值赋值给变量 banana
// 记住索引(即下标)是从 0 开始，下标 1 即第二个元素
// 结果: banana = "banana"
var banana = fruits[1]; 
```

Exercise 定义变量使用数组的索引

```js
var cars = ["Mazda", "Honda", "Chevy", "Ford"]
var honda =
var ford =
var chevy =
var mazda =
```

# 长度

# 长度

数组有个属性称为长度，正如字面意思，这就是数组的长度。

```js
var array = [1 , 2, 3];

// 结果: l = 3
var l = array.length; 
```

Exercise 定义变量 a 为数组的长度

```js
var array = [1, 1, 2, 3, 5, 8];
var l = array.length;
var a =
```