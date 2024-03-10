# 二、类和对象

## 对象的链式调用

### 问题

你想调用一个对象上的多个方法，但不想每次都引用该对象。

### 解决方案

在每次链式调用后返回 this（即@）对象

```js
class CoffeeCup
    constructor:  ->
        @properties=
            strength: 'medium'
            cream: false
            sugar: false
    strength: (newStrength) ->
        @properties.strength = newStrength
        this
    cream: (newCream) ->
        @properties.cream = newCream
        this
    sugar: (newSugar) ->
        @properties.sugar = newSugar
        this

morningCup = new CoffeeCup()

morningCup.properties # => { strength: 'medium', cream: false, sugar: false }

eveningCup = new CoffeeCup().strength('dark').cream(true).sugar(true)

eveningCup.properties # => { strength: 'dark', cream: true, sugar: true }
```

### 讨论

jQuery 库使用类似的手段从每一个相似的方法中返回选择符对象，并在后续方法中通过调整选择的范围修改该对象：

```js
    $('p').filter('.topic').first()
```

对我们自己对象而言，一点点元编程就可以自动设置这个过程并明确声明返回 this 的意图。

```js
addChainedAttributeAccessor = (obj, propertyAttr, attr) ->
    obj[attr] = (newValues...) ->
        if newValues.length == 0
            obj[propertyAttr][attr]
        else
            obj[propertyAttr][attr] = newValues[0]
            obj

class TeaCup
    constructor:  ->
        @properties=
            size: 'medium'
            type: 'black'
            sugar: false
            cream: false
        addChainedAttributeAccessor(this, 'properties', attr) for attr of @properties

earlgrey = new TeaCup().size('small').type('Earl Grey').sugar('false')

earlgrey.properties # => { size: 'small', type: 'Earl Grey', sugar: false }

earlgrey.sugar true

earlgrey.sugar() # => true
```

## 类方法和实例方法

### 问题

你想创建类和实例的方法。

### 解决方案

#### 类方法

```js
class Songs
  @_titles: 0    # Although it's directly accessible, the leading _ defines it by convention as private property.

  @get_count: ->
    @_titles

  constructor: (@artist, @title) ->
    @constructor._titles++     # Refers to <Classname>._titles, in this case Songs.titles

Songs.get_count()
 # => 0

song = new Songs("Rick Astley", "Never Gonna Give You Up")
Songs.get_count()
 # => 1

song.get_count()
 # => TypeError: Object <Songs> has no method 'get_count'
```

#### 实例方法

```js
class Songs
  _titles: 0    # Although it's directly accessible, the leading _ defines it by convention as private property.

  get_count: ->
    @_titles

  constructor: (@artist, @title) ->
    @_titles++

song = new Songs("Rick Astley", "Never Gonna Give You Up")
song.get_count()
 # => 1

Songs.get_count()
 # => TypeError: Object function Songs(artist, title) ... has no method 'get_count'
```

### 讨论

Coffeescript 会在对象本身中保存类方法（也叫静态方法），而不是在对象原型中（以及单一的对象实例），在保存了记录的同时也将类级的变量保存在中心位置。

## 类变量和实例变量

### 问题

你想创建类变量和实例变量（属性）。

### 解决方案

#### 类变量

```js
class Zoo
  @MAX_ANIMALS: 50
  MAX_ZOOKEEPERS: 3

  helpfulInfo: =>
    "Zoos may contain a maximum of #{@constructor.MAX_ANIMALS} animals and #{@MAX_ZOOKEEPERS} zoo keepers."

Zoo.MAX_ANIMALS
 # => 50

Zoo.MAX_ZOOKEEPERS
 # => undefined (it is a prototype member)

Zoo::MAX_ZOOKEEPERS
 # => 3

zoo = new Zoo
zoo.MAX_ZOOKEEPERS
 # => 3

zoo.helpfulInfo()
 # => "Zoos may contain a maximum of 50 animals and 3 zoo keepers."

zoo.MAX_ZOOKEEPERS = "smelly"
zoo.MAX_ANIMALS = "seventeen"
zoo.helpfulInfo()
 # => "Zoos may contain a maximum of 50 animals and smelly zoo keepers."
```

#### 实例变量

你必须在一个类的方法中才能定义实例变量（例如属性），在 constructor 结构中初始化你的默认值。

```js
class Zoo
  constructor: ->
    @animals = [] # Here the instance variable is defined

  addAnimal: (name) ->
    @animals.push name

zoo = new Zoo()
zoo.addAnimal 'elephant'

otherZoo = new Zoo()
otherZoo.addAnimal 'lion'

zoo.animals
 # => ['elephant']

otherZoo.animals
 # => ['lion']
```

### 警告！

不要试图在 constructor 外部添加变量（即使在 [elsewhere](http://arcturo.github.io/library/coffeescript/03_classes.html#content) 中提到了，由于潜在的 JavaScript 的原型概念，这不会像预期那样运行正确）。

```js
class BadZoo
  animals: []           # Translates to BadZoo.prototype.animals = []; and is thus shared between instances

  addAnimal: (name) ->
    @animals.push name  # Works due to the prototype concept of Javascript

zoo = new BadZoo()
zoo.addAnimal 'elephant'

otherZoo = new BadZoo()
otherZoo.addAnimal 'lion'

zoo.animals
 # => ['elephant','lion'] # Oops...

otherZoo.animals
 # => ['elephant','lion'] # Oops...

BadZoo::animals
 # => ['elephant','lion'] # The value is stored in the prototype
```

### 讨论

Coffeescript 会将类变量的值保存在类中而不是它定义的原型中。这在定义类中的变量时是十分有用的，因为这不会被实体属性变量重写。

## 克隆对象（深度复制）

### 问题

你想复制一个对象，包含其所有子对象。

### 解决方案

```js
clone = (obj) ->
  if not obj? or typeof obj isnt 'object'
    return obj

  if obj instanceof Date
    return new Date(obj.getTime()) 

  if obj instanceof RegExp
    flags = ''
    flags += 'g' if obj.global?
    flags += 'i' if obj.ignoreCase?
    flags += 'm' if obj.multiline?
    flags += 'y' if obj.sticky?
    return new RegExp(obj.source, flags) 

  newInstance = new obj.constructor()

  for key of obj
    newInstance[key] = clone obj[key]

  return newInstance

x =
  foo: 'bar'
  bar: 'foo'

y = clone(x)

y.foo = 'test'

console.log x.foo isnt y.foo, x.foo, y.foo
 # => true, bar, test
```

### 讨论

通过赋值来复制对象与通过克隆函数来复制对象的区别在于如何处理引用。赋值只会复制对象的引用，而克隆函数则会:

*   创建一个全新的对象
*   这个新对象会复制原对象的所有属性，
*   并且对原对象的所有子对象，也会递归调用克隆函数，复制每个子对象的所有属性。

下面是一个通过赋值来复制对象的例子：

```js
x =
  foo: 'bar'
  bar: 'foo'

y = x

y.foo = 'test'

console.log x.foo isnt y.foo, x.foo, y.foo
 # => false, test, test
```

显然，复制之后修改 y 也就修改了 x。

## 类的混合

### 问题

你有一些通用方法，你想把他们包含到很多不同的类中。

### 解决方案

使用 mixOf 库函数，它会生成一个混合父类。

```js
mixOf = (base, mixins...) ->
  class Mixed extends base
  for mixin in mixins by -1 #earlier mixins override later ones
    for name, method of mixin::
      Mixed::[name] = method
  Mixed

...

class DeepThought
  answer: ->
    42

class PhilosopherMixin
  pontificate: ->
    console.log "hmm..."
    @wise = yes

class DeeperThought extends mixOf DeepThought, PhilosopherMixin
  answer: ->
    @pontificate()
    super()

earth = new DeeperThought
earth.answer()
 # hmm...

 # => 42
```

### 讨论

这适用于轻量级的混合。因此你可以从基类和基类的祖先中继承方法，也可以从混合类的基类和祖先中继承，但是不能从混合类的祖先中继承。与此同时，在声明了一个混合类后，此后的对这个混合类进行的改变是不会反应出来的。

## 创建一个不存在的对象字面值

### 问题

你想初始化一个对象字面值，但如果这个对象已经存在，你不想重写它。

### 解决方案

使用存在判断运算符（existential operator）。

```js

    window.MY_NAMESPACE ?= {}
```

### 讨论

这行代码与下面的 JavaScript 代码等价：

```js

    window.MY_NAMESPACE = window.MY_NAMESPACE || {};
```

这是 JavaScript 中一个常用的技巧，即使用对象字面值来定义命名空间。这样先判断是否存在同名的命名空间然后再创建，可以避免重写已经存在的命名空间。

## CoffeeScrip 的 type 函数

### 问题

你想在不使用 typeof 的情况下知道一个函数的类型。（要了解为什么 typeof 不靠谱，请参见 [`javascript.crockford.com/remedial.html`](http://javascript.crockford.com/remedial.html)。）

### 解决方案

使用下面这个 type 函数

```js
type = (obj) ->
    if obj == undefined or obj == null
      return String obj
    classToType = {
      '[object Boolean]': 'boolean',
      '[object Number]': 'number',
      '[object String]': 'string',
      '[object Function]': 'function',
      '[object Array]': 'array',
      '[object Date]': 'date',
      '[object RegExp]': 'regexp',
      '[object Object]': 'object'
    }
    return classToType[Object.prototype.toString.call(obj)]
```

### 讨论

这个函数模仿了 jQuery 的 [$.type 函数](http://api.jquery.com/jQuery.type/)。

需要注意的是，在某些情况下，只要使用鸭子类型检测及存在运算符就可以不必检测对象的类型了。例如，下面这行代码不会发生异常，它会在 myArray 的确是数组（或者一个带有 push 方法的类数组对象）的情况下向其中推入一个元素，否则什么也不做。

```js
    myArray?.push? myValue
```