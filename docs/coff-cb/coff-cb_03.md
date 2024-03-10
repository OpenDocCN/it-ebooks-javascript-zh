# 三、字符串

## 大写单词首字母

### 问题

你想把字符串中每个单词的首字母转换为大写形式。

### 解决方案

使用“拆分-映射-拼接”模式：先把字符串拆分成单词，然后通过映射来大写单词第一个字母小写其他字母，最后再将转换后的单词拼接成字符串。

```js
("foo bar baz".split(' ').map (word) -> word[0].toUpperCase() + word[1..-1].toLowerCase()).join ' '
 # => 'Foo Bar Baz'
```

或者使用列表推导（comprehension），也可以实现同样的结果：

```js
(word[0].toUpperCase() + word[1..-1].toLowerCase() for word in "foo   bar   baz".split /\s+/).join ' '
 # => 'Foo Bar Baz'
```

### 讨论

“拆分-映射-拼接”是一种常用的脚本编写模式，可以追溯到 Perl 语言。如果能把这个功能直接通过“[扩展类](http://coffeescript-cookbook.github.io/chapters/objects/extending-classes)”放到 String 类里，就更方便了。

需要注意的是，“拆分-映射-拼接”模式存在两个问题。第一个问题，只有在文本形式统一的情况下才能有效拆分文本。如果来源字符串中有分隔符包含多个空白符，就需要考虑怎么过滤掉多余的空单词。一种解决方案是使用正则表达式来匹配空白符的串，而不是像前面那样只匹配一个空格：

```js
("foo    bar    baz".split(/\s+/).map (word) -> word[0].toUpperCase() + word[1..-1].toLowerCase()).join ' '
 # => 'Foo Bar Baz'
```

但这样做又会导致第二个问题：在结果字符串中，原来的空白符串经过拼接就只剩下一个空格了。

不过，一般来说，这两个问题还是可以接受的。所以，“拆分-映射-拼接”仍然是一种有效的技术。

## 查找子字符串

### 问题

你想在一条消息中查找某个关键字第一次或最后一次出现的位置。

### 解决方案

分别使用 JavaScript 的 indexOf() 和 lastIndexOf() 方法查找字符串第一次和最后一次出现的位置。语法: string.indexOf searchstring, start

```js
message = "This is a test string. This has a repeat or two. This might even have a third."
message.indexOf "This", 0
 # => 0

 # Modifying the start parameter

message.indexOf "This", 5
 # => 23

message.lastIndexOf "This"
 # => 49
```

### 讨论

还需要想办法统计出给定字符串在一条消息中出现的次数。

## 生成唯一 ID

### 问题

你想随机生成一个唯一的标识符。

### 解决方案

可以根据一个随机数值生成一个 Base 36 编码的字符串。

```js
uniqueId = (length=8) ->
  id = ""
  id += Math.random().toString(36).substr(2) while id.length < length
  id.substr 0, length

uniqueId()    # => n5yjla3b
uniqueId(2)   # => 0d
uniqueId(20)  # => ox9eo7rt3ej0pb9kqlke
uniqueId(40)  # => xu2vo4xjn4g0t3xr74zmndshrqlivn291d584alj
```

### 讨论

使用其他技术也可以，但这种方法相对来说性能更高，也更灵活。

## 字符串插值

### 问题

你想创建一个字符串，让它包含体现某个 CoffeeScript 变量的文本。

### 解决方案

使用 CoffeeScript 中类似 Ruby 的字符串插值，而不是 JavaScript 的字符串拼接。

插值：

```js
muppet = "Beeker"
favorite = "My favorite muppet is #{muppet}!"

 # => "My favorite muppet is Beeker!"
```

```js
square = (x) -> x * x
message = "The square of 7 is #{square 7}."

 # => "The square of 7 is 49."
```

### 讨论

CoffeeScript 的插值与 Ruby 类似，多数表达式都可以用在 #{ ... } 插值结构中。

CoffeeScript 支持在插值结构中放入多个有副作用的表达式，但建议大家不要这样做。因为只有表达式的最后一个值会被插入。

```js
 # 可以这样做，但不要这样做。否则，你会疯掉。

square = (x) -> x * x
muppet = "Beeker"
message = "The square of 10 is #{muppet='Animal'; square 10}. Oh, and your favorite muppet is now #{muppet}."

 # => "The square of 10 is 100\. Oh, and your favorite muppet is now Animal."
```

## 把字符串转换为小写形式

### 问题

你想把字符串转换成小写形式。

### 解决方案

使用 JavaScript 的 String 的 toLowerCase() 方法：

```js
"ONE TWO THREE".toLowerCase()
 # => 'one two three'
```

### 讨论

toLowerCase() 是一个标准的 JavaScript 方法。不要忘了带圆括号。

### 语法块

通过下面的快捷方式可以添加某种类似　Ruby 的语法块：

```js
String::downcase = -> @toLowerCase()
"ONE TWO THREE".downcase()
 # => 'one two three'
```

上面的代码演示了 CoffeeScript 的两个特性:

*   双冒号 :: 是引用 `.prototype` 的快捷方式；
*   “at” 字符 @ 是引用 this 的快捷方式。

上面的代码会编译成如下 JavaScript 代码：

```js
String.prototype.downcase = function() {
  return this.toLowerCase();
};
"ONE TWO THREE".downcase();
```

**提示** 尽管上面的用法在类似 Ruby 的语言中很常见，但在 JavaScript 中对本地对象的扩展经常被视为不好的。（请看：[Maintainable JavaScript: Don’t modify objects you don’t own](http://www.nczonline.net/blog/2010/03/02/maintainable-javascript-dont-modify-objects-you-down-own/);[Extending built-in native objects. Evil or not?](http://perfectionkills.com/extending-built-in-native-objects-evil-or-not/)）

## 匹配字符串

### 问题

你想要匹配两个或多个字符串。

### 解决方案

计算把一个字符串转换成另一个字符串所需的编辑距离或操作数。

```js
levenshtein = (str1, str2) ->

    l1 = str1.length
    l2 = str2.length
    prevDist = [0..l2]
    nextDist = [0..l2]

    for i in [1..l1] by 1
      nextDist[0] = i
      for j in [1..l2] by 1
        if (str1.charAt i-1) == (str2.charAt j-1)
          nextDist[j] = prevDist[j-1]
        else
          nextDist[j] = 1 + Math.min prevDist[j], nextDist[j-1], prevDist[j-1]
        [prevDist,nextDist]=[nextDist, prevDist]

    prevDist[l2]
```

### 讨论

可以使用赫斯伯格（ Hirschberg ）或瓦格纳菲舍尔（ Wagner–Fischer）的算法来计算来文史特（ Levenshtein ）距离。这个例子用的是瓦格纳菲舍尔算法。

这个版本的文史特算法和内存呈线性关系，和时间呈二次方关系。

在这里我们使用 str.charAt i 这种表示法而不用 str[i] 这种方式，是因为后者在某些浏览器（如 IE7）中不支持。

起初，"by 1" 在两次循环中看起来似乎是没用的。它在这里是用来避免一个 coffeescript [i..j] 语法的常见错误。如果 str1 或 str2 为空字符串，那么 [1..l1] 或 [1..l2] 将会返回 [1,0] 。添加了 "by 1" 的循环也能编译出更加简洁高效的 javascript 。

最后，循环结尾处对回收数组的优化在这里主要是为了演示 coffeescript 中交换两个变量的语法。

## 重复字符串

### 问题

你想重复一个字符串。

### 解决方案

创建一个包含 n+1 个空元素的数组，然后用要重复的字符串作为连接字符将数组元素拼接到一起：

```js
 # 创建包含 10 个 foo 的字符串

Array(11).join 'foo'

 # => "foofoofoofoofoofoofoofoofoofoo"
```

### 为字符串重复方法

你也可以在字符串的原型中为其创建方法。它十分简单：

```js
 # 为所有的字符串添加重复方法，这会重复返回 n 次字符串

String::repeat = (n) -> Array(n+1).join(this)
```

### 讨论

JavaScript 缺少字符串重复函数，CoffeeScript 也没有提供。虽然在这里也可以使用列表推导（ comprehensions ），但对于简单的字符串重复来说，还是像这样先构建一个包含 n+1 个空元素的数组，然后再把它拼接起来更方便。

## 拆分字符串

### 问题

你想拆分一个字符串。

### 解决方案

使用 JavaScript 字符串的 split() 方法：

```js
"foo bar baz".split " "
 # => [ 'foo', 'bar', 'baz' ]
```

### 讨论

String 的这个 split() 方法是标准的 JavaScript 方法。可以用来基于任何分隔符——包括正则表达式来拆分字符串。这个方法还可以接受第二个参数，用于指定返回的子字符串数目。

```js
"foo-bar-baz".split "-"
 # => [ 'foo', 'bar', 'baz' ]
```

```js
"foo bar \t baz".split /\s+/ # => [ 'foo', 'bar', 'baz' ]

"the sun goes down and I sit on the old broken-down river pier".split " ", 2 # => [ 'the', 'sun' ]

```

清理字符串前后的空白符

问题

你想清理字符串前后的空白符。

解决方案

使用 JavaScript 的正则表达式来替换空白符。

要清理字符串前后的空白符，可以使用以下代码：
```js

" padded string ".replace /^\s+|\s+$/g, "" # => 'padded string'

```

如果只想清理字符串前面的空白符，使用以下代码：
```js

" padded string ".replace /^\s+/g, "" # => 'padded string '

```

如果只想清理字符串后面的空白符，使用以下代码：
```js

" padded string ".replace /\s+$/g, "" # => ' padded string'

```

讨论

Opera、Firefox 和 Chrome 中 String 的原型都有原生的 trim 方法，其他浏览器也可以添加一个。对于这个方法而言，还是尽可能使用内置方法，否则就创建一个 polyfill：
```js

unless String::trim then String::trim = -> @replace /^\s+|\s+$/g, ""

" padded string ".trim() # => 'padded string'

```

语法块

还可以添加一些类似 Ruby 中的语法块，定义如下快捷方法：
```js

String::strip = -> if String::trim? then @trim() else @replace /^\s+|\s+$/g, "" String::lstrip = -> @replace /^\s+/g, "" String::rstrip = -> @replace /\s+$/g, ""

" padded string ".strip() # => 'padded string'

" padded string ".lstrip() # => 'padded string '

" padded string ".rstrip() # => ' padded string'

```

要想深入了解 JavaScript 执行 trim 操作时的性能，请参见 Steve Levithan 的这篇博客文章。

把字符串转换为大写形式

问题

你想把字符串转换成大写形式。

解决方案

使用 JavaScript 的 String 的 toUpperCase() 方法：
```js

"one two three".toUpperCase() # => 'ONE TWO THREE'

```

讨论

toUpperCase() 是一个标准的 JavaScript 方法。不要忘了带圆括号。

语法块

通过下面的快捷方式可以添加某种类似 Ruby 的语法块：
```js

String::upcase = -> @toUpperCase() "one two three".upcase() # => 'ONE TWO THREE'

```

上面的代码演示了 CoffeeScript 的两个特性:

* 双冒号 :: 是引用 .prototype 的快捷方式；
* “at” 字符 @ 是引用 this 的快捷方式。

上面的代码会编译成如下 JavaScript 代码：
```js

String.prototype.upcase = function() { return this.toUpperCase(); }; "one two three".upcase(); ```

**提示** 尽管上面的用法在类似 Ruby 的语言中很常见，但在 JavaScript 中对本地对象的扩展经常被视为不好的。（请看：[Maintainable JavaScript: Don’t modify objects you don’t own](http://www.nczonline.net/blog/2010/03/02/maintainable-javascript-dont-modify-objects-you-down-own/);[Extending built-in native objects. Evil or not?](http://perfectionkills.com/extending-built-in-native-objects-evil-or-not/)）