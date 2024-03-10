# 七、方法

## 去抖动函数

### 问题

你想只执行某个函数一次，在开始或结束时把多个连续的调用合并成一个简单的操作。

### 解决方案

使用一个命名函数：

```js
debounce: (func, threshold, execAsap) ->
  timeout = null
  (args...) ->
    obj = this
    delayed = ->
      func.apply(obj, args) unless execAsap
      timeout = null
    if timeout
      clearTimeout(timeout)
    else if (execAsap)
      func.apply(obj, args)
    timeout = setTimeout delayed, threshold || 100
mouseMoveHandler: (e) ->
  @debounce((e) ->
    # 只能在鼠标光标停止 300 毫秒后操作一次。
  300)

someOtherHandler: (e) ->
  @debounce((e) ->
    # 只能在初次执行 250 毫秒后操作一次。
  250, true)
```

### 讨论

可参阅 John Hann 的博客文章，了解 [JavaScript 去抖动方法](http://unscriptable.com/2009/03/20/debouncing-javascript-methods/)。

## 当函数括号不可选

### 问题

你想要调用一个没有参数的函数，但不希望使用括号。

### 解决方案

不管怎样都使用括号。

另一个方法是使用 do 表示法，如下：

```js
notify = -> alert "Hello, user!"
do notify if condition
```

编译成 JavaScript 则可表示为：

```js
var notify;
notify = function() {
    return alert("Hello, user!");
};
if (condition) {
    notify();
}
```

### 讨论

这个方法与 Ruby 类似，在于都可以不使用括号来完成方法的调用。而不同点在于，CoffeeScript 把空的函数名作为函数的指针。这样以来，如果你不赋予一个方法任何参数，那么 CoffeeScript 将无法分辨你是想要调用函数还是把它作为引用。

这是好是坏呢？其实只是有所不同。它创造了一个意想不到的语法实例——括号并不*总是*可选的——但是它能让你流利地使用名字来传递和接收函数，这对于 Ruby 来说是难以实现的。

对于 CoffeeScript 来说，使用 do 表示法是一个巧妙的方法来克服括号使用恐惧症。尽管有部分人宁愿在函数调用中写出所有括号。

## 递归函数

### 问题

你想在一个函数中调用相同的函数。

### 解决方案

使用一个命名函数：

```js
ping = ->
    console.log "Pinged"
    setTimeout ping, 1000
```

若为未命名函数，则使用 @arguments.callee@：

```js
delay = 1000

setTimeout((->
    console.log "Pinged"
    setTimeout arguments.callee, delay
    ), delay)
```

### 讨论

虽然 **arguments.callee** 允许未命名函数的递归，在内存密集型应用中占有一定优势，但是命名函数相对来说目的更加明确，也更易于代码的维护。

## 提示参数

### 问题

你的函数将会被可变数量的参数所调用。

### 解决方案

使用 *splat* 。

```js
loadTruck = (firstDibs, secondDibs, tooSlow...) ->
    truck:
        driversSeat: firstDibs
        passengerSeat: secondDibs
        trunkBed: tooSlow

loadTruck("Amanda", "Joel")
 # => { truck: { driversSeat: "Amanda", passengerSeat: "Joel", trunkBed: [] } }

loadTruck("Amanda", "Joel", "Bob", "Mary", "Phillip")
 # => { truck: { driversSeat: "Amanda", passengerSeat: "Joel", trunkBed: ["Bob", "Mary", "Phillip"] } }
```

使用尾部参数：

```js
loadTruck = (firstDibs, secondDibs, tooSlow..., leftAtHome) ->
    truck:
        driversSeat: firstDibs
        passengerSeat: secondDibs
        trunkBed: tooSlow
    taxi:
        passengerSeat: leftAtHome

loadTruck("Amanda", "Joel", "Bob", "Mary", "Phillip", "Austin")
 # => { truck: { driversSeat: 'Amanda', passengerSeat: 'Joel', trunkBed: [ 'Bob', 'Mary', 'Phillip' ] }, taxi: { passengerSeat: 'Austin' } }

loadTruck("Amanda")
 # => { truck: { driversSeat: "Amanda", passengerSeat: undefined, trunkBed: [] }, taxi: undefined }
```

### 讨论

通过在函数其中的（不多于）一个参数之后添加一个省略号（...），CoffeeScript 能把所有不被其他命名参数采用的参数值整合进一个列表中。就算并没有提供命名参数，它也会制造一个空列表。