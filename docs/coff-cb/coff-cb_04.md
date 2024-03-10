# 四、数组

## 检查变量的类型是否为数组

### 问题

你希望检查一个变量是否为一个数组。

```
myArray = []
console.log typeof myArray // outputs 'object'
```

“typeof” 运算符为数组输出了一个错误的结果。

### 解决方案

使用下面的代码：

```
typeIsArray = Array.isArray || ( value ) -> return {}.toString.call( value ) is '[object Array]'
```

为了使用这个，像下面这样调用 typeIsArray 就可以了。

```
myArray = []
typeIsArray myArray // outputs true
```

### 讨论

上面方法取自 "the Miller Device"。另外一个方式是使用 Douglas Crockford 的片段。

```
typeIsArray = ( value ) ->
    value and
        typeof value is 'object' and
        value instanceof Array and
        typeof value.length is 'number' and
        typeof value.splice is 'function' and
        not ( value.propertyIsEnumerable 'length' )
```

## 将数组连接

### 问题

你希望将两个数组连接到一起。

### 解决方案

在 JavaScript 中，有两个标准方法可以用来连接数组。

第一种是使用 JavaScript 的数组方法 concat()：

```
array1 = [1, 2, 3]
array2 = [4, 5, 6]
array3 = array1.concat array2
 # => [1, 2, 3, 4, 5, 6]
```

需要指出的是 array1 没有被运算修改。连接后形成的新数组的返回值是一个新的对象。

如果你希望在连接两个数组后不产生新的对象，那么你可以使用下面的技术：

```
array1 = [1, 2, 3]
array2 = [4, 5, 6]
Array::push.apply array1, array2
array1
 # => [1, 2, 3, 4, 5, 6]
```

在上面的例子中，Array.prototype.push.apply(a, b) 方法修改了 array1 而没有产生一个新的数组对象。

在 CoffeeScript 中，我们可以简化上面的方式，通过给数组创建一个新方法 merge()：

```
Array::merge = (other) -> Array::push.apply @, other

array1 = [1, 2, 3]
array2 = [4, 5, 6]
array1.merge array2
array1
 # => [1, 2, 3, 4, 5, 6]
```

另一种方法，我可以直接将一个 CoffeeScript splat(array2) 放入 push() 中，避免了使用数组原型。

```
array1 = [1, 2, 3]
array2 = [4, 5, 6]
array1.push array2...
array1
 # => [1, 2, 3, 4, 5, 6]
```

一个更加符合语言习惯的方法是在一个数组语言中直接使用 splat 运算符(...)。这可以用来连接任意数量的数组。

```
array1 = [1, 2, 3]
array2 = [4, 5, 6]
array3 = [array1..., array2...]
array3
 # => [1, 2, 3, 4, 5, 6]
```

### 讨论

CoffeeScript 缺少了一种用来连接数组的特殊语法，但是 concat() 和 push() 是标准的 JavaScript 方法。

## 由数组创建一个对象词典

### 问题

你有一组对象，例如：

```
cats = [
  {
    name: "Bubbles"
    age: 1
  },
  {
    name: "Sparkle"
    favoriteFood: "tuna"
  }
]
```

但是你想让它像词典一样，可以通过关键字访问它，就像使用 cats["Bubbles"]。

### 解决方案

你需要将你的数组转换为一个对象。通过这样使用 reduce：

```
 # key = The key by which to index the dictionary

Array::toDict = (key) ->
  @reduce ((dict, obj) -> dict[ obj[key] ] = obj if obj[key]?; return dict), {}
```

使用它时像下面这样：

```
catsDict = cats.toDict('name')
  catsDict["Bubbles"]
  # => { age: 1, name: "Bubbles" }
```

### 讨论

另一种方法是使用数组推导：

```
Array::toDict = (key) ->
  dict = {}
  dict[obj[key]] = obj for obj in this when obj[key]?
  dict
```

如果你使用 Underscore.js，你可以创建一个 mixin：

```
_.mixin toDict: (arr, key) ->
    throw new Error('_.toDict takes an Array') unless _.isArray arr
    _.reduce arr, ((dict, obj) -> dict[ obj[key] ] = obj if obj[key]?; return dict), {}
catsDict = _.toDict(cats, 'name')
catsDict["Sparkle"]
 # => { favoriteFood: "tuna", name: "Sparkle" }
```

## 由数组创建一个字符串

### 问题

你想由数组创建一个字符串。

### 解决方案

使用 JavaScript 的数组方法 toString()：

```
["one", "two", "three"].toString()
 # => 'one,two,three'
```

### 讨论

toString() 是一个标准的 JavaScript 方法。不要忘记圆括号。

## 定义数组范围

### 问题

你想定义一个数组的范围。

### 解决方案

在 CoffeeScript 中，有两种方式定义数组元素的范围。

```
myArray = [1..10]
 # => [ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 ]
```

```
myArray = [1...10]
 # => [ 1, 2, 3, 4, 5, 6, 7, 8, 9 ]
```

要想反转元素的范围，则可以写成下面这样。

```
myLargeArray = [10..1]
 # => [ 10, 9, 8, 7, 6, 5, 4, 3, 2, 1 ]
```

```
myLargeArray = [10...1]
 # => [ 10, 9, 8, 7, 6, 5, 4, 3, 2 ]
```

### 讨论

包含范围以 “..” 运算符定义，包含最后一个值。 排除范围以 “...” 运算符定义，并且通常忽略最后一个值。

## 筛选数组

### 问题

你想要根据布尔条件来筛选数组。

### 解决方案

使用 `Array.filter (ECMAScript 5)： array = [1..10]`

```
array.filter (x) -> x > 5
 # => [6,7,8,9,10]
```

在 EC5 之前的实现中，可以通过添加一个筛选函数扩展 Array 的原型，该函数接受一个回调并对自身进行过滤，将回调函数返回 true 的元素收集起来。

```
 # 扩展 Array 的原型

Array::filter = (callback) ->
  element for element in this when callback(element)

array = [1..10]

 # 筛选偶数

filtered_array = array.filter (x) -> x % 2 == 0
 # => [2,4,6,8,10]

 # 过滤掉小于或等于 5 的元素

gt_five = (x) -> x > 5
filtered_array = array.filter gt_five
 # => [6,7,8,9,10]
```

### 讨论

这个方法与 Ruby 的 Array 的 #select 方法类似。

## 列表推导

### 问题

你有一个对象数组，想将它们映射到另一个数组，类似于 Python 的列表推导。

### 解决方案

使用列表推导，但不要忘记还有 [mapping-arrays](http://coffeescript-cookbook.github.io/chapters/arrays/mapping-arrays) 。

```
electric_mayhem = [ { name: "Doctor Teeth", instrument: "piano" },
                    { name: "Janice", instrument: "lead guitar" },
                    { name: "Sgt. Floyd Pepper", instrument: "bass" },
                    { name: "Zoot", instrument: "sax" },
                    { name: "Lips", instrument: "trumpet" },
                    { name: "Animal", instrument: "drums" } ]

names = (muppet.name for muppet in electric_mayhem)
 # => [ 'Doctor Teeth', 'Janice', 'Sgt. Floyd Pepper', 'Zoot', 'Lips', 'Animal' ]
```

### 讨论

因为 CoffeeScript 直接支持列表推导，在你使用一个 Python 的语句时，他们会很好地起到作用。对于简单的映射，列表推导具有更好的可读性。但是对于复杂的转换或链式映射，映射数组可能更合适。

## 映射数组

### 问题

你有一个对象数组，想把这些对象映射到另一个数组中，就像 Ruby 的映射一样。

### 解决方案

使用 map() 和匿名函数，但不要忘了还有列表推导。

```
electric_mayhem = [ { name: "Doctor Teeth", instrument: "piano" },
                    { name: "Janice", instrument: "lead guitar" },
                    { name: "Sgt. Floyd Pepper", instrument: "bass" },
                    { name: "Zoot", instrument: "sax" },
                    { name: "Lips", instrument: "trumpet" },
                    { name: "Animal", instrument: "drums" } ]

names = electric_mayhem.map (muppet) -> muppet.name
 # => [ 'Doctor Teeth', 'Janice', 'Sgt. Floyd Pepper', 'Zoot', 'Lips', 'Animal' ]
```

### 讨论

因为 CoffeeScript 支持匿名函数，所以在 CoffeeScript 中映射数组就像在 Ruby 中一样简单。 映射在 CoffeeScript 中是处理复杂转换和连缀映射的好方法。如果你的转换如同上例中那么简单，那可能将它当成[列表推导](http://coffeescript-cookbook.github.io/chapters/arrays/list-comprehensions) 看起来会清楚一些。

## 数组最大值

### 问题

你需要找出数组中包含的最大的值。

### 解决方案

你可以使用 JavaScript 实现，在列表推导基础上使用 Math.max()：

```
Math.max [12, 32, 11, 67, 1, 3]... 
 # => 67
```

另一种方法，在 ECMAScript 5 中，可以使用 Array 的 reduce 方法，它与旧的 JavaScript 实现兼容。

```
 # ECMAScript 5 

[12,32,11,67,1,3].reduce (a,b) -> Math.max a, b 
 # => 67
```

### 讨论

Math.max 在这里比较两个数值，返回其中较大的一个。省略号 (...) 将每个数组价值转化为给函数的参数。你还可以使用它与其他带可变数量的参数进行讨论，如执行 console.log 。

## 归纳数组

### 问题

你有一个对象数组，想要把它们归纳为一个值，类似于 Ruby 中的 reduce() 和 reduceRight() 。

### 解决方案

可以使用一个匿名函数包含 Array 的 reduce() 和 reduceRight() 方法，保持代码清晰易懂。这里归纳可能会像对数值和字符串应用 + 运算符那么简单。

```
[1,2,3,4].reduce (x,y) -> x + y
 # => 10

["words", "of", "bunch", "A"].reduceRight (x, y) -> x + " " + y
 # => 'A bunch of words'
```

或者，也可能更复杂一些，例如把列表中的元素聚集到一个组合对象中。

```
people =
    { name: 'alec', age: 10 }
    { name: 'bert', age: 16 }
    { name: 'chad', age: 17 }

people.reduce (x, y) ->
    x[y.name]= y.age
    x
, {}
 # => { alec: 10, bert: 16, chad: 17 }
```

#### 讨论

Javascript 1.8 中引入了 reduce 和 reduceRight ，而 Coffeescript 为匿名函数提供了简单自然的表达语法。二者配合使用，可以把集合的项合并为组合的结果。

## 删除数组中的相同元素

### 问题

你想从数组中删除相同元素。

### 解决方案

```
Array::unique = ->
  output = {}
  output[@[key]] = @[key] for key in [0...@length]
  value for key, value of output

[1,1,2,2,2,3,4,5,6,6,6,"a","a","b","d","b","c"].unique()
 # => [ 1, 2, 3, 4, 5, 6, 'a', 'b', 'd', 'c' ]
```

### 讨论

在 JavaScript 中有很多的独特方法来实现这一功能。这一次是基于“最快速的方法来查找数组的唯一元素”，出自[这里](http://www.shamasis.net/2009/09/fast-algorithm-to-find-unique-items-in-javascript-array/) 。

> 注意: 延长本机对象通常被认为是在 JavaScript 不好的做法，即便它在 Ruby 语言中相当普遍， (参考:[Maintainable JavaScript: Don’t modify objects you don’t own](http://www.nczonline.net/blog/2010/03/02/maintainable-javascript-dont-modify-objects-you-down-own/)

## 反转数组

### 问题

你想要反转数组元素。

### 解决方案

使用 JavaScript Array 的 reverse() 方法：

```
["one", "two", "three"].reverse()
 # => ["three", "two", "one"]
```

#### 讨论

reverse() 是标准的 JavaScript 方法，别忘了带圆括号。

## 打乱数组中的元素

### 问题

你想打乱数组中的元素。

### 解决方案

[Fisher-Yates shuffle](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle) 是一种高效、公正的方式来让数组中的元素随机化。这是一个相当简单的方法：在列表的结尾处开始，用一个随机元素交换最后一个元素列表中的最后一个元素。继续下一个并重复操作，直到你到达列表的起始端，最终列表中所有的元素都已打乱。这 [Fisher-Yates shuffle Visualization](http://bost.ocks.org/mike/shuffle/) 可以帮助你理解算法。

```
shuffle = (source) ->
  # Arrays with < 2 elements do not shuffle well. Instead make it a noop.
  return source unless source.length >= 2
  # From the end of the list to the beginning, pick element `index`.
  for index in [source.length-1..1]
    # Choose random element `randomIndex` to the front of `index` to swap with.
    randomIndex = Math.floor Math.random() * (index + 1)
    # Swap `randomIndex` with `index`, using destructured assignment
    [source[index], source[randomIndex]] = [source[randomIndex], source[index]]
  source

shuffle([1..9])
 # => [ 3, 1, 5, 6, 4, 8, 2, 9, 7 ]
```

### 讨论

#### 一种错误的方式

有一个很常见但是错误的打乱数组的方式：通过随机数。

```
shuffle = (a) -> a.sort -> 0.5 - Math.random()
```

如果你做了一个随机的排序，你应该得到一个序列随机的顺序，对吧？甚至[微软也用这种随机排序算法](http://www.robweir.com/blog/2010/02/microsoft-random-browser-ballot.html) 。原来，[这种随机排序算法产生有偏差的结果](http://blog.codinghorror.com/the-danger-of-naivete/) ，因为它存在一种洗牌的错觉。随机排序不会导致一个工整的洗牌，它会导致序列排序质量的参差不齐。

#### 速度和空间的优化

以上的解决方案处理速度是不一样的。该列表，当转换成 JavaScript 时，比它要复杂得多，变性分配比处理裸变量的速度要慢得多。以下代码并不完善，并且需要更多的源代码空间 … 但会编译量更小，运行更快：

```
shuffle = (a) ->
  i = a.length
  while --i > 0
    j = ~~(Math.random() * (i + 1)) # ~~ is a common optimization for Math.floor
    t = a[j]
    a[j] = a[i]
    a[i] = t
  a
```

#### 扩展 Javascript 来包含乱序数组

下面的代码将乱序功能添加到数组原型中，这意味着你可以在任何希望的数组中运行它，并以更直接的方式来运行它。

```
Array::shuffle ?= ->
  if @length > 1 then for i in [@length-1..1]
    j = Math.floor Math.random() * (i + 1)
    [@[i], @[j]] = [@[j], @[i]]
  this

[1..9].shuffle()
 # => [ 3, 1, 5, 6, 4, 8, 2, 9, 7 ]
```

> 注意: 虽然它像在 Ruby 语言中相当普遍，但是在 JavaScript 中扩展本地对象通常被认为是不太好的做法 ( 参考: [Maintainable JavaScript: Don’t modify objects you don’t own](http://www.nczonline.net/blog/2010/03/02/maintainable-javascript-dont-modify-objects-you-down-own/)
> 正如提到的，以上的代码的添加是十分安全的。它仅仅需要添 Array :: shuffle 如果它不存在，就要添加赋值运算符 (? =) 。这样，我们就不会重写到别人的代码，或是本地浏览器的方式。
> 
> 同时，如果你认为你会使用很多的实用功能，可以考虑使用一个工具库，像 [Lo-dash](https://lodash.com/) 。他们有很多功能，像跨浏览器的简洁高效的地图。 [Underscore](http://underscorejs.org/) 也是一个不错的选择。

## 检测每个元素

### 问题

你希望能够在特定的情况下检测出在数组中的每个元素。

### 解决方案

使用 Array.every (ECMAScript 5)：

```
evens = (x for x in [0..10] by 2)

evens.every (x)-> x % 2 == 0
 # => true
```

Array.every 被加入到 Mozilla 的 Javascript 1.6 ，ECMAScript 5 标准。如果你的浏览器支持，但仍无法实施 EC5 ，那么请检查 [_.all from underscore.js](http://documentcloud.github.io/underscore/) 。

对于一个真实例子，假设你有一个多选择列表，如下：

```
<select multiple id="my-select-list">
  <option>1</option>
  <option>2</option>
  <option>Red Car</option>
  <option>Blue Car</option>
</select>
```

现在你要验证用户只选择了数字。让我们利用 array.every ：

```
validateNumeric = (item)->
  parseFloat(item) == parseInt(item) && !isNaN(item)

values = $("#my-select-list").val()

values.every validateNumeric
```

### 讨论

这与 Ruby 中的 Array #all? 的方法很相似。

## 使用数组来交换变量

### 问题

你想通过数组来交换变量。

### 解决方案

使用 CoffeeScript 的解构赋值语法：

```
a = 1
b = 3

[a, b] = [b, a]

a
 # => 3

b
 # => 1
```

### 讨论

解构赋值可以不依赖临时变量实现变量值的交换。

这种语法特别适合在遍历数组的时候只想迭代最短数组的情况：

```
ray1 = [ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 ]
ray2 = [ 5, 9, 14, 20 ]

intersection = (a, b) ->
  [a, b] = [b, a] if a.length > b.length
  value for value in a when value in b

intersection ray1, ray2
 # => [ 5, 9 ]

intersection ray2, ray1
 # => [ 5, 9 ]
```

## 对象数组

### 问题

你想要得到一个与你的某些属性匹配的数组对象。

你有一系列的对象，如：

```
cats = [
  {
    name: "Bubbles"
    favoriteFood: "mice"
    age: 1
  },
  {
    name: "Sparkle"
    favoriteFood: "tuna"
  },
  {
    name: "flyingCat"
    favoriteFood: "mice"
    age: 1
  }
]
```

你想用某些特征来滤出想要的对象。例如：猫的位置 ({ 年龄: 1 }) 或者猫的位置 ({ 年龄: 1 , 最爱的食物: "老鼠" })

### 解决方案

你可以像这样来扩展数组：

```
Array::where = (query) ->
    return [] if typeof query isnt "object"
    hit = Object.keys(query).length
    @filter (item) ->
        match = 0
        for key, val of query
            match += 1 if item[key] is val
        if match is hit then true else false

cats.where age:1
 # => [ { name: 'Bubbles', favoriteFood: 'mice', age: 1 },{ name: 'flyingCat', favoriteFood: 'mice', age: 1 } ]

cats.where age:1, name: "Bubbles"
 # => [ { name: 'Bubbles', favoriteFood: 'mice', age: 1 } ]

cats.where age:1, favoriteFood:"tuna"
 # => []
```

### 讨论

这是一个确定的匹配。我们能够让匹配函数更加灵活：

```
Array::where = (query, matcher = (a,b) -> a is b) ->
    return [] if typeof query isnt "object"
    hit = Object.keys(query).length
    @filter (item) ->
        match = 0
        for key, val of query
            match += 1 if matcher(item[key], val)
        if match is hit then true else false

cats.where name:"bubbles"
 # => []

 # it's case sensitive

cats.where name:"bubbles", (a, b) -> "#{ a }".toLowerCase() is "#{ b }".toLowerCase()
 # => [ { name: 'Bubbles', favoriteFood: 'mice', age: 1 } ]

 # now it's case insensitive
```

处理收集的一种方式可以被叫做 “find” ，但是像 underscore 或者 lodash 这些库把它叫做 “where” 。

## 类似 Python 的 zip 函数

### 问题

你想把多个数组连在一起，生成一个数组的数组。换句话说，你需要实现与 Python 中的 zip 函数类似的功能。 Python 的 zip 函数返回的是元组的数组，其中每个元组中包含着作为参数的数组中的第 i 个元素。

### 解决方案

使用下面的 CoffeeScript 代码：

```
 # Usage: zip(arr1, arr2, arr3, ...)

zip = () ->
  lengthArray = (arr.length for arr in arguments)
  length = Math.max.apply(Math, lengthArray)
  argumentLength = arguments.length
  results = []
  for i in [0...length]
    semiResult = []
    for arr in arguments
      semiResult.push arr[i]
    results.push semiResult
  return results

zip([0, 1, 2, 3], [0, -1, -2, -3])
 # => [[0, 0], [1, -1], [2, -2], [3, -3]]
```