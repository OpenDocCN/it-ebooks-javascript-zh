# 八、元编程

## 检测与构建丢失的函数

### 问题

你想要检测一个函数是否存在，如果不存在则构建该函数。（比如 Internet Explorer 8 的 ECMAScript 5 函数）。

### 解决方案

使用存在赋值运算符（?=）来把函数分配给类库的原型（使用 :: 简写），然后把它放于一个立即执行函数表达式中（do ->）使其含有所有变量。

```js
do -> Array::filter ?= (callback) ->
  element for element in this when callback element

array = [1..10]

array.filter (x) -> x > 5
 # => [6,7,8,9,10]
```

### 讨论

在 JavaScript （同样地，在 CoffeeScript）中，对象都有一个原型成员，它定义了什么成员函数能够适用于基于该原型的所有对象。
在 CoffeeScript 中，你可以使用 :: 捷径来访问这个原型。所以如果你想要把过滤函数添加至数组类中，就执行 **Array::filter = ...** 语句。它能把过滤函数加至所有数组中。

但是，不要去覆盖一个在第一时间还没有构造的原型。比如，如果 **Array::filter = ...** 已经以快速本地形式存在于浏览器中，或者库制造者拥有其对于 **Array::filter = ...** 的独特版本，这样以来，你要么换一个慢速的 JavaScript 版本，要么打破这种依赖于其自身 Array::shuffle 的库。
你需要做的仅仅是在函数不存在的时候添加该函数。这就是存在赋值运算符（?=）的意义。如果我们执行 **Array::filter = ...** 语句，它会首先判断 **Array::filter** 是否已经存在。如果存在的话，它就会使用现在的版本。否则，它会添加你的版本。

最后，由于存在的赋值运算符在编译时会创建一些变量，我们会通过把它们封装在[立即调用函数表达式（ IIFE ）](http://benalman.com/news/2010/11/immediately-invoked-function-expression/)中来简化代码。这将隐藏那些内部专用的变量，以防止泄露。所以假如我们写的函数已经存在，那么它将运行，基本上什么都没做然后退出，绝对不会对你的代码造成影响。但是假如我们写的函数并不存在，那么我们发送出去的仅是一个作为闭包的函数。所以只有你写的函数能够对代码产生影响。无论哪种方式，?= 的内部运行都会被隐藏。

### 举例

接下来，我们用上述的方法编译了 CoffeeScript 并附加了说明：

```js
// (function(){ ... })() 是一个 IIFE， 使用 `do ->` 来编译它。
(function() {

  // 它来自 `?=`  运算符，用来检查 Array.prototype.filter (`Array::filter`) 是否存在。
  // 如果确实存在，我们把它设置给其自身，并返回。如果不存在，则把它设置给函数，并返回函数。
  // The IIFE is only used to hide _base and _ref from the outside world.
  var _base, _ref;
  return (_ref = (_base = Array.prototype).filter) != null ? _ref : _base.filter = function(callback) {

    // `element for element in this when callback element`
    var element, _i, _len, _results;
    _results = [];
    for (_i = 0, _len = this.length; _i < _len; _i++) {
      element = this[_i];
      if (callback(element)) {
        _results.push(element);
      }
    }
    return _results;

  };
// The end of the IIFE from `do ->`
})();
```

## 扩展内置对象

### 问题

你想要扩展一个类来增加新的函数或者替换旧的。

### 解决方案

使用 :: 把你的新函数分配到对象或者类的原型中。

```js
String::capitalize = () ->
  (this.split(/\s+/).map (word) -> word[0].toUpperCase() + word[1..-1].toLowerCase()).join ' '

"foo bar     baz".capitalize()
 # => 'Foo Bar Baz'
```

### 讨论

在 JavaScript （同样地，在 CoffeeScript ）中，对象都有一个原型成员，它定义了什么成员函数能够适用于基于该原型的所有对象。在 CoffeeScript 中，你可以使用 :: 捷径来直接访问这个原型。

> 注意：虽然这种做法在很多种语言中相当普遍，比如 Ruby，但是在 JavaScript 中，扩展本地对象通常被认为是不好的做法（可参考：[可维护的 JavaScript：不要修改你不拥有的对象](http://www.nczonline.net/blog/2010/03/02/maintainable-javascript-dont-modify-objects-you-down-own/)；[扩展内置的本地对象。对还是错？](http://perfectionkills.com/extending-native-builtins/)。）