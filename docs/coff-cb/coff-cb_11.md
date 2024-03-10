# 十一、正则表达式

## 使用 Heregexes

### 问题

你需要写一个复杂的正则表达式。

### 解决方案

使用 CoffeeScript 的 “heregexes” ——可以忽视内部空白字符并可以包含注释的扩展正则表达式。

```
pattern = ///
  ^\(?(\d{3})\)? # 采集区域代码，忽略可选的括号
  [-\s]?(\d{3})  # 采集前缀，忽略可选破折号或空格
  -?(\d{4})      # 采集行号，忽略可选破折号
///
[area_code, prefix, line] = "(555)123-4567".match(pattern)[1..3]
 # => ['555', '123', '4567']
```

### 讨论

通过打破复杂的正则表达式和注释重点部分，它们变得更加容易去辨认和维护。例如，现在这是一个相当明显的做法去改变正则表达式以容许前缀和行号之间存在可选的空间。

### heregexes 中的空白字符

空白字符在 heregexes 中是被忽视的——所以如果要为 ASCII 空格匹配字符，你应该怎么做呢？

我们的解决方案是使用 @\s@ 字符组，它能够匹配空格，制表符和换行符。假如你只想匹配一个空格，你需要使用 \X20 来表示字面上的 ASCII 空格。

## 使用 HTML 命名实体替换 HTML 标签

### 问题

你需要使用命名实体来替代 HTML 标签：

`<br/> => &lt;br/&gt;`

### 解决方案

```
htmlEncode = (str) ->
  str.replace /[&<>"']/g, ($0) ->
    "&" + {"&":"amp", "<":"lt", ">":"gt", '"':"quot", "'":"#39"}[$0] + ";"

htmlEncode('<a href="http://bn.com">Barnes & Noble</a>')
 # => '&lt;a href=&quot;http://bn.com&quot;&gt;Barnes &amp; Noble&lt;/a&gt;'
```

### 讨论

可能有更好的途径去执行上述方法。

## 替换子字符串

### 问题

你需要用另一个值替换字符串的一部分。

### 解决方案

使用 JavaScript 的 **replace** 方法。它与给定字符串匹配，并返回已编辑的字符串。

第一个版本需要 2 个参数：*模式*和*字符串替换*

```
"JavaScript is my favorite!".replace /Java/, "Coffee"
 # => 'CoffeeScript is my favorite!'

"foo bar baz".replace /ba./, "foo"
 # => 'foo foo baz'

"foo bar baz".replace /ba./g, "foo"
 # => 'foo foo foo'
```

第二个版本需要 2 个参数：*模式*和*回调函数*

```
"CoffeeScript is my favorite!".replace /(\w+)/g, (match) ->
  match.toUpperCase()
 # => 'COFFEESCRIPT IS MY FAVORITE!'
```

每次匹配需要调用回调函数，并且匹配值作为参数传给回调函数。

### 讨论

正则表达式是一种强有力的方式来匹配和替换字符串。

## 查找子字符串

### 问题

你需要搜索一个字符串，并返回匹配的起始位置或匹配值本身。

### 解决方案

有几种使用正则表达式的方法来实现这个功能。其中一些方法被称为 RegExp 模式或对象还有一些方法被称为 String 对象。

#### RegExp 对象

第一种方式是在 RegExp 模式或对象中调用 test 方法。test 方法返回一个布尔值：

```
match = /sample/.test("Sample text")
 # => false

match = /sample/i.test("Sample text")
 # => true
```

下一种方式是在 RegExp 模式或对象中调用 exec 方法。exec 方法返回一个匹配信息的数组或空值：

```
match = /s(amp)le/i.exec "Sample text"
 # => [ 'Sample', 'amp', index: 0, input: 'Sample text' ]

match = /s(amp)le/.exec "Sample text"
 # => null
```

#### String 对象

match 方法使给定的字符串与表达式对象匹配。有 “g” 标识的返回一个包含匹配项的数组，没有 “g” 标识的仅返回第一个匹配项或如果没有找到匹配项则返回 null 。

```
"Watch out for the rock!".match(/r?or?/g)
 # => [ 'o', 'or', 'ro' ]

"Watch out for the rock!".match(/r?or?/)
 # => [ 'o', index: 6, input: 'Watch out for the rock!' ]

"Watch out for the rock!".match(/ror/)
 # => null
```

search 方法以字符串匹配正则表达式，且如果找到的话返回匹配的起始位置，未找到的话则返回 -1 。

```
"Watch out for the rock!".search /for/
 # => 10

"Watch out for the rock!".search /rof/
 # => -1
```

### 讨论

正则表达式是一种可用来测试和匹配子字符串的强大的方法。