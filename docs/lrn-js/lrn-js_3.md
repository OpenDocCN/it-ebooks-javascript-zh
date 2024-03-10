# 字符串

# 字符串

JavaScript 的字符串与其他高级语言字符串的实现类似。这表示文本基于消息和数据。

在这章节将涉及一些基础。关于如何创建新的字符串和常见的一些字符串处理。 以下是是一个例子：

```js
"Hello World" 
```

# 创建

# 创建

你可以在 Javascript 中，通过单引号或双引号定义字符串：

```js
// 可以用单引号
var str = 'Our lovely string';

// 也可以用双引号
var otherStr = "Another nice string"; 
```

在 Javascript 中，字符串可以使用 UTF-8 编码：

```js
"中文 español English हिन्दी العربية português বাংলা русский 日本語 ਪੰਜਾਬੀ 한국어"; 
```

**注意:** 字符串不能进行减，乘或除的运算。

Exercise 创建一个名为 `str` 的变量，值为 `"abc"` 。 `# 连接

# 连接

连接同时涉及两个或以上字符串，创建了一个组合这些原始数据的更长的字符串。在 Javascript 中使用 **+** 运算符。

```js
var bigStr = 'Hi ' + 'JS strings are nice ' + 'and ' + 'easy to add'; 
```

Exercise 连接不同的名字，这样变量 `fullName` 就包含了 John 的全名。

```js
var firstName = "John";
var lastName = "Smith";

var fullName =
```

# 长度

# 长度

在 Javascript 中通过使用 `.length` 很容易知道字符串中有多少字母。

```js
// 使用 .length
var size = 'Our lovely string'.length; 
```

**注意:** 字符串不能进行减，乘或除的运算。

Exercise 在变量 `size` 中储存 `str` 的长度。

```js
var str = "Hello World";

var size =
```