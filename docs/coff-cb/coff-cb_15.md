# 十五、测试

## 使用 Jasmine 测试

### 问题

假如你正在使用 CoffeeScript 写一个简单地计算器，并且想要验证其功能是否与预期一致。可以使用 [Jasmine](http://jasmine.github.io/) 测试框架。

### 讨论

在使用 Jasmine 测试框架时，你要在一个参数（spec）文档中写测试，文档描述的是代码需要测试的预期功能。

例如，我们希望计算器可以实现加法和减法的功能，并且可以正确进行正数和负数的运算。我们的 spec 文档如下列所示。

```js
 # calculatorSpec.coffee

describe 'Calculator', ->
    it 'can add two positive numbers', ->
        calculator = new Calculator()
        result = calculator.add 2, 3
        expect(result).toBe 5

    it 'can handle negative number addition', ->
        calculator = new Calculator()
        result = calculator.add -10, 5
        expect(result).toBe -5

    it 'can subtract two positive numbers', ->
        calculator = new Calculator()
        result = calculator.subtract 10, 6
        expect(result).toBe 4

    it 'can handle negative number subtraction', ->
        calculator = new Calculator()
        result = calculator.subtract 4, -6
        expect(result).toBe 10
```

### 配置 Jasmine

在你运行测试之前，必须要先下载并配置 Jasmine。包括：1.下载最新的 [Jasmine](https://github.com/pivotal/jasmine/tree/master/dist) 压缩文件；2.在你的项目工程中创建一个 spec 以及一个 spec/jasmine 目录；3.将下载的 Jasmine 文件解压到 spec/jasmine 目录中；4.创建一个测试流

### 创建测试流

Jasmine 可以使用 spec runner 的 HTML 文档在 web 浏览器中运行你的测试。 spec runner 是一个简单地 HTML 页面，连接着 Jasmine 以及你的代码所需要的必要的 JavaScript 和 CSS 文件。示例如下。

```js
 1 <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
 2   "http://www.w3.org/TR/html4/loose.dtd">
 3 <html>
 4 <head>
 5   <title>Jasmine Spec Runner</title>
 6   <link rel="shortcut icon" type="image/png" href="spec/jasmine/jasmine_favicon.png">
 7   <link rel="stylesheet" type="text/css" href="spec/jasmine/jasmine.css">
 8   <script src="http://code.jquery.com/jquery.min.js"></script>
 9   <script src="spec/jasmine/jasmine.js"></script>
10   <script src="spec/jasmine/jasmine-html.js"></script>
11   <script src="spec/jasmine/jasmine-jquery-1.3.1.js"></script>
12 
13   <!-- include source files here... -->
14   <script src="js/calculator.js"></script>
15 
16   <!-- include spec files here... -->
17   <script src="spec/calculatorSpec.js"></script>
18 
19 </head>
20 
21 <body>
22   <script type="text/javascript">
23     (function() {
24       var jasmineEnv = jasmine.getEnv();
25       jasmineEnv.updateInterval = 1000;
26 
27       var trivialReporter = new jasmine.TrivialReporter();
28 
29       jasmineEnv.addReporter(trivialReporter);
30 
31       jasmineEnv.specFilter = function(spec) {
32         return trivialReporter.specFilter(spec);
33       };
34 
35       var currentWindowOnload = window.onload;
36 
37       window.onload = function() {
38         if (currentWindowOnload) {
39           currentWindowOnload();
40         }
41         execJasmine();
42       };
43 
44       function execJasmine() {
45         jasmineEnv.execute();
46       }
47 
48     })();
49   </script>
50 </body>
51 </html>
```

此 spec runner 可以在 GitHub [gist](https://gist.github.com/2623232) 上下载。

使用 SpecRunner.html ，只是简单地参考你编译后的 JavaScript 文件，并且在 jasmine.js 以及其依赖项后编译的测试文件。

在上述示例中，我们在第 14 行包含了尚待开发的 calculator.js 文件，在第 17 行编译了 calculatorSpec.js 文件。

### **运行测试**

要运行我们的测试，只需要简单地在 web 浏览器中打开 SpecRunner.html 页面。在我们的示例中可以看到 4 个失败的 specs 共 8 个失败情况（如下）。

![Alt text](img/jasmine_failing_all.jpg)

**图片 15.1** Alt text

看来我们的测试是失败的，因为 jasmine 无法找到 Calculator 变量。那是因为它还没有被创建。现在让我们来创建一个新文件命名为 js/calculator.coffee 。

```js
 # calculator.coffee

window.Calculator = class Calculator
```

编译 calculator.coffee 并刷新浏览器来重新运行测试组。

![Alt text](img/jasmine_failing_better.jpg)

**图片 15.2** Alt text

现在我们还有 4 个失败而不是原来的 8 个了，只用一行代码便做出了 50%的改进。

### **测试通过**

实现我们的方法来看是否可以通过测试。

```js
 # calculator.coffee

window.Calculator = class Calculator
    add: (a, b) ->
        a + b

    subtract: (a, b) ->
        a - b
```

当我们刷新页面时可以看到全部通过。

![Alt text](img/jasmine_passing.jpg)

**图片 15.3** Alt text

### **重构测试**

既然测试全部通过了，我们应看一看我们的代码或测试是否可以被重构。

在我们的 spec 文件中，每个测试都创建了自己的 calculator 实例。这会使我们的测试相当的重复，特别是对于大型的测试套件。理想情况下，我们应该考虑将初始化代码移动到每次测试之前运行。

幸运的是 Jasmine 拥有一个 beforeEach 函数,就是为了这一目的设置的。

```js
describe 'Calculator', ->
    calculator = null

    beforeEach ->
        calculator = new Calculator()

    it 'can add two positive numbers', ->
        result = calculator.add 2, 3
        expect(result).toBe 5

    it 'can handle negative number addition', ->
        result = calculator.add -10, 5
        expect(result).toBe -5

    it 'can subtract two positive numbers', ->
        result = calculator.subtract 10, 6
        expect(result).toBe 4

    it 'can handle negative number subtraction', ->
        result = calculator.subtract 4, -6
        expect(result).toBe 10
```

当我们重新编译我们的 spec 然后刷新浏览器，可以看到测试仍然全部通过。

![Alt text](img/jasmine_passing2.jpg)

**图片 15.4** Alt text

## 使用 Nodeunit 测试

### 问题

假如你正在使用 CoffeeScript 并且想要验证功能是否与预期一致，便可以决定使用 [Nodeunit](https://github.com/caolan/nodeunit) 测试框架。

### 讨论

Nodeunit 是一种 JavaScript 对于单元测试库（ Unit Testing libraries ）中 xUnit 族的实现，Java, Python, Ruby, Smalltalk 中均可以使用。

当使用 xUnit 族测试框架时，你需要将所需测试的描述预期功能的代码写在一个文件中。

例如，我们希望我们的计算器可以进行加法和减法，并且对于正负数均可以正确计算，我们的测试如下。

```js
 # test/calculator.test.coffee

Calculator = require '../calculator'
exports.CalculatorTest =
    'test can add two positive numbers': (test) ->
        calculator = new Calculator
        result = calculator.add 2, 3
        test.equal(result, 5)
        test.done()

    'test can handle negative number addition': (test) ->
        calculator = new Calculator
        result = calculator.add -10, 5
        test.equal(result,  -5)
        test.done()

    'test can subtract two positive numbers': (test) ->
        calculator = new Calculator
        result = calculator.subtract 10, 6
        test.equal(result, 4)
        test.done()

    'test can handle negative number subtraction': (test) ->
        calculator = new Calculator
        result = calculator.subtract 4, -6
        test.equal(result, 10)
        test.done()
```

### **安装 Nodeunit**

在可以运行你的测试之前，你必须先安装 Nodeunit ：

首先创建一个 package.json 文件

```js
{
  "name": "calculator",
  "version": "0.0.1",
  "scripts": {
    "test": "./node_modules/.bin/nodeunit test"
  },
  "dependencies": {
    "coffee-script": "~1.4.0",
    "nodeunit": "~0.7.4"
  }
}
```

接下来从一个终端运行。

```js
$ npm install
```

### **运行测试**

使用代码行可以简便地运行测试文件：

```js
$ npm test
```

测试失败，因为我们并没有 calculator.coffee

```js
suki@Yuzuki:nodeunit_testing (master)$ npm test
npm WARN package.json calculator@0.0.1 No README.md file found!

> calculator@0.0.1 test /Users/suki/tmp/nodeunit_testing
> ./node_modules/.bin/nodeunit test

/Users/suki/tmp/nodeunit_testing/node_modules/nodeunit/lib/nodeunit.js:72
        if (err) throw err;
                       ^
Error: ENOENT, stat '/Users/suki/tmp/nodeunit_testing/test'
npm ERR! Test failed.  See above for more details.
npm ERR! not ok code 0
```

我们创建一个简单文件

```js
 # calculator.coffee

class Calculator

module.exports = Calculator
```

并且重新运行测试套件。

```js
suki@Yuzuki:nodeunit_testing (master)$ npm test
npm WARN package.json calculator@0.0.1 No README.md file found!

> calculator@0.0.1 test /Users/suki/tmp/nodeunit_testing
> ./node_modules/.bin/nodeunit test

calculator.test
✖ CalculatorTest - test can add two positive numbers

TypeError: Object #<Calculator> has no method 'add'
  ...

✖ CalculatorTest - test can handle negative number addition

TypeError: Object #<Calculator> has no method 'add'
  ...

✖ CalculatorTest - test can subtract two positive numbers

TypeError: Object #<Calculator> has no method 'subtract'
  ...

✖ CalculatorTest - test can handle negative number subtraction

TypeError: Object #<Calculator> has no method 'subtract'
  ...

FAILURES: 4/4 assertions failed (31ms)
npm ERR! Test failed.  See above for more details.
npm ERR! not ok code 0
```

### **通过测试**

让我们对方法进行实现来观察测试是否可以通过。

```js
 # calculator.coffee

class Calculator

  add: (a, b) ->
    a + b

  subtract: (a, b) ->
    a - b

module.exports = Calculator
```

当我们重新运行测试时可以看到全部通过：

```js
suki@Yuzuki:nodeunit_testing (master)$ npm test
npm WARN package.json calculator@0.0.1 No README.md file found!

> calculator@0.0.1 test /Users/suki/tmp/nodeunit_testing
> ./node_modules/.bin/nodeunit test

calculator.test
✔ CalculatorTest - test can add two positive numbers
✔ CalculatorTest - test can handle negative number addition
✔ CalculatorTest - test can subtract two positive numbers
✔ CalculatorTest - test can handle negative number subtraction

OK: 4 assertions (27ms)
```

### **重构测试**

既然测试全部通过，我们应看一看我们的代码或测试是否可以被重构。

在我们的测试文件中，每个测试都创建了自己的 calculator 实例。这会使我们的测试相当的重复，特别是对于大型的测试套件。理想情况下，我们应该考虑将初始化代码移动到每次测试之前运行。

通常在其他的 xUnit 库中，Nodeunit 会提供一个 setUp（以及 tearDown ）功能会在测试前调用。

```js
Calculator = require '../calculator'

exports.CalculatorTest =

    setUp: (callback) ->
        @calculator = new Calculator
        callback()

    'test can add two positive numbers': (test) ->
        result = @calculator.add 2, 3
        test.equal(result, 5)
        test.done()

    'test can handle negative number addition': (test) ->
        result = @calculator.add -10, 5
        test.equal(result,  -5)
        test.done()

    'test can subtract two positive numbers': (test) ->
        result = @calculator.subtract 10, 6
        test.equal(result, 4)
        test.done()

    'test can handle negative number subtraction': (test) ->
        result = @calculator.subtract 4, -6
        test.equal(result, 10)
        test.done()
```

我们可以重新运行测试，仍然可以全部通过。

![](img/jk_book.png)![](img/jk_weixin.png)更多信息请访问 ![](http://wiki.jikexueyuan.com/project/coffeescript/)[`wiki.jikexueyuan.com/project/coffeescript/`](http://wiki.jikexueyuan.com/project/coffeescript/)