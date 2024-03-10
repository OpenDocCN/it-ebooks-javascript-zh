# 六、数学

## 数学常数

### 问题

你需要使用常见的数学常数，比如 π 或者 e 。

### 解决方案

使用 Javascript 的 Math object 来提供通常需要的数学常数。

```
Math.PI
 # => 3.141592653589793

 # Note: Capitalization matters! This produces no output, it's undefined.

Math.Pi
 # =>

Math.E
 # => 2.718281828459045

Math.SQRT2
 # => 1.4142135623730951

Math.SQRT1_2
 # => 0.7071067811865476

 # Natural log of 2\. ln(2)

Math.LN2
 # => 0.6931471805599453

Math.LN10
 # => 2.302585092994046

Math.LOG2E
 # => 1.4426950408889634

Math.LOG10E
 # => 0.4342944819032518
```

### 讨论

另外一个例子是关于一个数学常数用于真实世界的问题，是数学章节有关[弧度和角度](http://coffeescript-cookbook.github.io/chapters/math/radians-degrees)的部分。

## 更快的 Fibonacci 算法

### 问题

你想计算出 Fibonacci 数列中的数值 N ，但需迅速地算出结果。

### 解决方案

下面的方案（仍有需改进的地方）最初在 Robin Houston 的博客上被提出来。

这里给出一些关于该算法和改进方法的链接：

*   [`bosker.wordpress.com/2011/04/29/the-worst-algorithm-in-the-world/`](http://bosker.wordpress.com/2011/04/29/the-worst-algorithm-in-the-world/)
*   [`www.math.rutgers.edu/~erowland/fibonacci`](http://www.math.rutgers.edu/~erowland/fibonacci)
*   [`jsfromhell.com/classes/bignumber`](http://jsfromhell.com/classes/bignumber)
*   [`www.math.rutgers.edu/~erowland/fibonacci`](http://www.math.rutgers.edu/~erowland/fibonacci)
*   [`bigintegers.blogspot.com/2010/11/square-division-power-square-root`](http://bigintegers.blogspot.com/2010/11/square-division-power-square-root)
*   [`bugs.python.org/issue3451`](http://bugs.python.org/issue3451)

以下的代码来源于：[`gist.github.com/1032685`](https://gist.github.com/1032685)

```
 ###

Author: Jason Giedymin <jasong _a_t_ apache -dot- org>
        http://www.jasongiedymin.com
        https://github.com/JasonGiedymin

CoffeeScript Javascript 的快速 Fibonacci 代码是基于 Robin Houston 博客里的 python 代码。
见下面的链接。

我要介绍一下 Newtonian，Burnikel / Ziegle 和 Binet 关于大数目框架算法的实现。

Todo:
- https://github.com/substack/node-bigint
- BZ and Newton mods.
- Timing

 ###

MAXIMUM_JS_FIB_N = 1476

fib_bits = (n) ->
    #代表一个作为二进制数字阵列的整数

    bits = []
    while n > 0
        [n, bit] = divmodBasic n, 2
        bits.push bit

    bits.reverse()
    return bits

fibFast = (n) ->
    #快速 Fibonacci

    if n < 0
        console.log "Choose an number >= 0"
        return

    [a, b, c] = [1, 0, 1]

    for bit in fib_bits n
        if bit
            [a, b] = [(a+c)*b, b*b + c*c]
        else
            [a, b] = [a*a + b*b, (a+c)*b]

        c = a + b
        return b

divmodNewton = (x, y) ->
    throw new Error "Method not yet implemented yet."

divmodBZ = () ->
    throw new Error "Method not yet implemented yet."

divmodBasic = (x, y) ->
    ###
   这里并没有什么特别的。如果可能的话，也许以后的版本将是 Newtonian 或者 Burnikel / Ziegler 的。
   ###

    return [(q = Math.floor x/y), (r = if x < y then x else x % y)]

start = (new Date).getTime();
calc_value = fibFast(MAXIMUM_JS_FIB_N)
diff = (new Date).getTime() - start;
console.log "[#{calc_value}] took #{diff} ms."
```

## 平方根倒数快速算法

### 问题

你想[快速](https://en.wikipedia.org/wiki/Fast_inverse_square_root)计算某数的平方根倒数。

### 解决方案

在 Quake Ⅲ Arena 的源代码中，这个奇怪的算法对一个幻数进行整数运算，来计算平方根倒数的浮点近似值。

在 CoffeeScript 中，他使用经典原始的变量，以及由 [Chris Lomont](http://www.lomont.org/Math/Papers/2003/InvSqrt.pdf) 发现的新的最优 32 位幻数。除此之外，还使用 64 位大小的幻数。

另一特征是可以通过控制[牛顿迭代法](https://en.wikipedia.org/wiki/Newton%27s_method)的迭代次数来改变其精确度。

相比于传统的，该算法在性能上更胜一筹，这归功于使用的机器及其精确度。

运行的时候使用 coffee -c script.coffee 来编译 script：

然后复制粘贴编译的 JS 代码到浏览器的 JavaScript 控制台。

注意：你需要一个支持[类型数组](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays)的浏览器

参考：

1.  ftp://ftp.idsoftware.com/idstuff/source/quake3-1.32b-source.zip
2.  [`www.lomont.org/Math/Papers/2003/InvSqrt.pdf`](http://www.lomont.org/Math/Papers/2003/InvSqrt.pdf)
3.  [`en.wikipedia.org/wiki/Newton%27s_method`](http://en.wikipedia.org/wiki/Newton%27s_method)
4.  [`developer.mozilla.org/en/JavaScripttypedarrays`](https://developer.mozilla.org/en/JavaScripttypedarrays)
5.  [`en.wikipedia.org/wiki/Fastinversesquare_root`](http://en.wikipedia.org/wiki/Fastinversesquare_root)

以下的代码来源于：[`gist.github.com/1036533`](https://gist.github.com/1036533)

```
 ###

Author: Jason Giedymin <jasong _a_t_ apache -dot- org>
        http://www.jasongiedymin.com
        https://github.com/JasonGiedymin

在 Quake Ⅲ Arena 的源代码 1 中，这个奇怪的算法对一个幻数进行整数运算，来计算平方根倒数的浮点近似值 [5](http://en.wikipedia.org/wiki/Fast_inverse_square_root)。

在 CoffeeScript 中，我使用经典原始的变量，以及由 Chris Lomont [2](http://www.lomont.org/Math/Papers/2003/InvSqrt.pdf) 发现的新的最优 32 位幻数。除此之外，还使用 64 位大小的幻数。

另一特征是可以通过控制牛顿迭代法 [3](http://en.wikipedia.org/wiki/Newton%27s_method) 的迭代次数来改变其精确度。

相比于传统的，该算法在性能上更胜一筹，归功于使用的机器及其精确度。

运行的时候使用 coffee -c script.coffee 来编译 script： 

然后复制粘贴编译的 JS 代码到浏览器的 JavaScript 控制台。

注意：你需要一个支持类型数组 [4](https://developer.mozilla.org/en/JavaScript_typed_arrays) 的浏览器

 ###

approx_const_quake_32 = 0x5f3759df # See [1]
approx_const_32 = 0x5f375a86 # See [2]
approx_const_64 = 0x5fe6eb50c7aa19f9 # See [2]

fastInvSqrt_typed = (n, precision=1) ->
    # 使用类型数组。现在只能在浏览器中操作。
    # Node.JS 的版本即将推出。

    y = new Float32Array(1)
    i = new Int32Array(y.buffer)

    y[0] = n
    i[0] = 0x5f375a86 - (i[0] >> 1)

    for iter in [1...precision]
        y[0] = y[0] * (1.5 - ((n * 0.5) * y[0] * y[0]))

    return y[0]

 ### 单次运行示例

testSingle = () ->
    example_n = 10

    console.log("Fast InvSqrt of 10, precision 1: #{fastInvSqrt_typed(example_n)}")
    console.log("Fast InvSqrt of 10, precision 5: #{fastInvSqrt_typed(example_n, 5)}")
    console.log("Fast InvSqrt of 10, precision 10: #{fastInvSqrt_typed(example_n, 10)}")
    console.log("Fast InvSqrt of 10, precision 20: #{fastInvSqrt_typed(example_n, 20)}")
    console.log("Classic of 10: #{1.0 / Math.sqrt(example_n)}")

testSingle()
```

## 生成可预测的随机数

### 问题

你需要生成在一定范围内的随机数，但你也需要对发生器进行“生成种子”操作来提供可预测的值。

### 解决方案

编写你自己的随机数生成器。当然有很多方法可以做到这一点，这里给出一个简单的示例。 *该发生器绝对不可以以加密为目的！*

```
class Rand
  # 如果没有种子创建，使用当前时间作为种子
  constructor: (@seed) ->
    # Knuth and Lewis' improvements to Park and Miller's LCPRNG
    @multiplier = 1664525
    @modulo = 4294967296 # 2**32-1;
    @offset = 1013904223
    unless @seed? && 0 <= seed < @modulo
      @seed = (new Date().valueOf() * new Date().getMilliseconds()) % @modulo

  # 设置新的种子值
  seed: (seed) ->
    @seed = seed

  # 返回一个随机整数满足 0 <= n < @modulo
  randn: ->
    # new_seed = (a * seed + c) % m
    @seed = (@multiplier*@seed + @offset) % @modulo

 # 返回一个随机浮点满足 0 <= f < 1.0
  randf: ->
    this.randn() / @modulo

  # 返回一个随机的整数满足 0 <= f < n
  rand: (n) ->
    Math.floor(this.randf() * n)

  #返回一个随机的整数满足 min <= f < max
  rand2: (min, max) ->
    min + this.rand(max-min)
```

### 讨论

JavaScript 和 CoffeeScript 都不提供可产生随机数的发生器。编写发生器对于我们来说将是一个挑战，在于权衡量的随机性与发生器的简单性。对随机性的全面讨论已超出了本书的范围。如需进一步阅读，可参考 Donald Kunth 的 *The Art of Computer Programming* 第 Ⅱ 卷第三章的 “ Random Numbers ” ，以及 *Numerical Recipes in C* 第二版本第七章的“ Random Numbers ”。

但是，对于这个随机数发生器只有简单的解释。这是一个线性同余伪随机数发生器，其运行源于一条数学公式 I[j+1] = (aI[j]+c) % m，其中 a 是乘数，c 是加法偏移量，m 是模数。每次请求随机数时就会执行很大的乘法和加法运算——这里的“很大”与密钥空间有关——得到的结果将以模数的形式被返回密钥空间。

这个发生器的周期为 232。虽然它绝对不能以加密为目的，但是对于最简单的随机性要求来说，它是相当足够的。randn() 在循环之前将遍历整个密钥空间，下一个数由上一个来确定。

如果你想修补这个发生器，强烈建议你去阅读 Knuth 的 * The Art of Computer Programming * 中的第三章。随机数生成是件很容易弄糟的事情，然而 Knuth 会解释如何区分好的和坏的随机数生成。

不要把发生器的输出结果变成模数。如果你需要一个整数的范围，应使用分割的方法。线性同余发生器的低位是不具有随机性的。特别的是，它总是从偶数种子产生奇数，反之亦然。所以如果你需要一个随机的 0 或者 1，不要使用：

```
 # NOT random! Do not do this!

r.randn() % 2
```

因为你肯定得不到随机数字。反而，你应该使用 r.rand(2)。

## 生成随机数

### 问题

你需要生成在一定范围内的随机数。

### 解决方案

使用 JavaScript 的 Math.random() 来获得浮点数，满足 0<=X<1.0 。使用乘法和 Math.floor 得到在一定范围内的数字。

```
probability = Math.random()
0.0 <= probability < 1.0
 # => true

 # 注意百分位数不会达到 100。从 0 到 100 的范围实际上是 101 的跨度。

percentile = Math.floor(Math.random() * 100)
0 <= percentile < 100
 # => true

dice = Math.floor(Math.random() * 6) + 1
1 <= dice <= 6
 # => true

max = 42
min = -13
range = Math.random() * (max - min) + min
-13 <= range < 42
 # => true
```

### 讨论

对于 JavaScript 来说，它更直接更快。

需要注意到 JavaScript 的 Math.random() 不能通过发生器生成随机数种子来得到特定值。详情可参考[产生可预测的随机数](http://coffeescript-cookbook.github.io/chapters/math/generating-predictable-random-numbers)。

产生一个从 0 到 n（不包括在内）的数，乘以 n。
产生一个从 1 到 n（包含在内）的数，乘以 n 然后加上 1。

## 转换弧度和度

### 问题

你需要实现弧度和度之间的转换。

### 解决方案

使用 JavaScript 的 Math.PI 和一个简单的公式来转换两者。

```
 # 弧度转换成度

radiansToDegrees = (radians) ->
    degrees = radians * 180 / Math.PI

radiansToDegrees(1)
 # => 57.29577951308232

 # 度转换成弧度

degreesToRadians = (degrees) ->
    radians = degrees * Math.PI / 180

degreesToRadians(1)
 # => 0.017453292519943295
```

## 一个随机整数函数

### 问题

你想要获得两个整数（包含在内）之间的一个随机整数。

### 解决方案

使用以下的函数。

```
randomInt = (lower, upper) ->
  [lower, upper] = [0, lower]     unless upper?           # 用一个参数调用
  [lower, upper] = [upper, lower] if lower > upper        # Lower 必须小于 upper
  Math.floor(Math.random() * (upper - lower + 1) + lower) # 最后一条语句是一个返回值

(randomInt(1) for i in [0...10])
 # => [0,1,1,0,0,0,1,1,1,0]

(randomInt(1, 10) for i in [0...10])
 # => [7,3,9,1,8,5,4,10,10,8]
```

## 指数对数运算

### 问题

你需要进行包含指数和对数的运算。

### 解决方案

使用 JavaScript 的 Math 对象来提供常用的数学函数。

```
 # Math.pow(x, y) 返回 x^y

Math.pow(2, 4)
 # => 16

 # Math.exp(x) 返回 E^x ，被简写为 Math.pow(Math.E, x)

Math.exp(2)
 # => 7.38905609893065

 # Math.log returns the natural (base E) log

Math.log(5)
 # => 1.6094379124341003

Math.log(Math.exp(42))
 # => 42

 # To get a log with some other base n, divide by Math.log(n)

Math.log(100) / Math.log(10)
 # => 2
```

### 讨论

若想了解关于数学对象的更多信息，请参阅 [Mozilla 开发者网络](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math)上的文档。另可参阅[数学常量](http://coffeescript-cookbook.github.io/chapters/math/constants)关于数学对象中各种常量的讨论。