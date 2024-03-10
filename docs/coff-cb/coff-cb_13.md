# 十三、设计模式

## 适配器模式

### 问题

想象你去国外旅行，一旦你意识到你的电源线插座与酒店房间墙上的插座不兼容时，幸运的是你记得带你的电源适配器。它将一边连接你的电源线插座另一边连接墙壁插座，允许它们之间进行通信。

同样的情况也可能会出现在代码中，当两个 ( 或更多 ) 实例 ( 类、模块等 ) 想跟对方通信，但其通信协议 ( 例如，他们所使用的语言交流 ) 不同。在这种情况下，[Adapter](http://en.wikipedia.org/wiki/Adapter_pattern) 模式更方便。它会充当翻译，从一边到另一边。

### 解决方案

```
 # a fragment of 3-rd party grid component

class AwesomeGrid
    constructor: (@datasource)->
        @sort_order = 'ASC' 
        @sorter = new NullSorter # in this place we use NullObject pattern (another useful pattern)
    setCustomSorter: (@customSorter) ->
        @sorter = customSorter
    sort: () ->
        @datasource = @sorter.sort @datasource, @sort_order
        # don't forget to change sort order

class NullSorter
    sort: (data, order) -> # do nothing; it is just a stub

class RandomSorter
    sort: (data)->
        for i in [data.length-1..1] #let's shuffle the data a bit
                j = Math.floor Math.random() * (i + 1)
                [data[i], data[j]] = [data[j], data[i]]
        return data

class RandomSorterAdapter
    constructor: (@sorter) ->
    sort: (data, order) ->
        @sorter.sort data

agrid = new AwesomeGrid ['a','b','c','d','e','f']
agrid.setCustomSorter new RandomSorterAdapter(new RandomSorter)
agrid.sort() # sort data with custom sorter through adapter
```

### 讨论

当你要组织两个具有不同接口的对象之间的交互时，适配器是有用的。它可以当你使用第三方库或者使用遗留代码时使用。在任何情况下小心使用适配器：它可以是有用的，但它也可以导致设计错误。

## 桥接模式

### 问题

你需要为代码保持一个可靠的接口，可以经常变化或者在多种实现间转换。

### 解决方案

使用桥接模式作为不同的实现和剩余代码的中间体。

假设你开发了一个浏览器的文本编辑器保存到云。然而，现在你需要通过独立客户端的端口将其在本地保存。

```
class TextSaver
    constructor: (@filename, @options) ->
    save: (data) ->

class CloudSaver extends TextSaver
    constructor: (@filename, @options) ->
        super @filename, @options
    save: (data) ->
        # Assuming jQuery
        # Note the fat arrows
        $( =>
            $.post "#{@options.url}/#{@filename}", data, =>
                alert "Saved '#{data}' to #{@filename} at #{@options.url}."
        )

class FileSaver extends TextSaver
    constructor: (@filename, @options) ->
        super @filename, @options
        @fs = require 'fs'
    save: (data) ->
        @fs.writeFile @filename, data, (err) => # Note the fat arrow
            if err? then console.log err
            else console.log "Saved '#{data}' to #{@filename} in #{@options.directory}."

filename = "temp.txt"
data = "Example data"

saver = if window?
    new CloudSaver filename, url: 'http://localhost' # => Saved "Example data" to temp.txt at http://localhost
else if root?
    new FileSaver filename, directory: './' # => Saved "Example data" to temp.txt in ./

saver.save data
```

### 讨论

桥接模式可以帮助你将特定实现的代码置于看不见的地方，这样你就可以专注于你的程序中的具体代码。在上面的示例中,应用程序的其余部分可以称为 saver.save data ，不考虑文件的最终结束。

## 生成器模式

### 问题

你需要准备一个复杂的、多部分的对象，你希望操作不止一次或有不同的配置。

### 解决方案

创建一个生成器封装对象的产生过程。

[Todo.txt](http://todotxt.com/) 格式提供了一个先进的但还是纯文本的方法来维护待办事项列表。手工输入每个项目有损耗且容易出错，然而 TodoTxtBuilder 类可以解决我们的麻烦：

```
class TodoTxtBuilder
    constructor: (defaultParameters={ }) ->
        @date = new Date(defaultParameters.date) or new Date
        @contexts = defaultParameters.contexts or [ ]
        @projects = defaultParameters.projects or [ ]
        @priority =  defaultParameters.priority or undefined
    newTodo: (description, parameters={ }) ->
        date = (parameters.date and new Date(parameters.date)) or @date
        contexts = @contexts.concat(parameters.contexts or [ ])
        projects = @projects.concat(parameters.projects or [ ])
        priorityLevel = parameters.priority or @priority
        createdAt = [date.getFullYear(), date.getMonth()+1, date.getDate()].join("-")
        contextNames = ("@#{context}" for context in contexts when context).join(" ")
        projectNames = ("+#{project}" for project in projects when project).join(" ")
        priority = if priorityLevel then "(#{priorityLevel})" else ""
        todoParts = [priority, createdAt, description, contextNames, projectNames]
        (part for part in todoParts when part.length > 0).join " "

builder = new TodoTxtBuilder(date: "10/13/2011")

builder.newTodo "Wash laundry"

 # => '2011-10-13 Wash laundry'

workBuilder = new TodoTxtBuilder(date: "10/13/2011", contexts: ["work"])

workBuilder.newTodo "Show the new design pattern to Lucy", contexts: ["desk", "xpSession"]

 # => '2011-10-13 Show the new design pattern to Lucy @work @desk @xpSession'

workBuilder.newTodo "Remind Sean about the failing unit tests", contexts: ["meeting"], projects: ["compilerRefactor"], priority: 'A'

 # => '(A) 2011-10-13 Remind Sean about the failing unit tests @work @meeting +compilerRefactor'
```

### 讨论

TodoTxtBuilder 类负责所有文本的生成，让程序员关注每个工作项的独特元素。此外，命令行工具或 GUI 可以插入这个代码且之后仍然保持支持，提供轻松、更高版本的格式。

### 前期建设

并不是每次创建一个新的实例所需的对象都要从头开始，我们将负担转移到一个单独的对象，可以在对象创建过程中进行调整。

```
builder = new TodoTxtBuilder(date: "10/13/2011")

builder.newTodo "Order new netbook"

 # => '2011-10-13 Order new netbook'

builder.projects.push "summerVacation"

builder.newTodo "Buy suntan lotion"

 # => '2011-10-13 Buy suntan lotion +summerVacation'

builder.contexts.push "phone"

builder.newTodo "Order tickets"

 # => '2011-10-13 Order tickets @phone +summerVacation'

delete builder.contexts[0]

builder.newTodo "Fill gas tank"

 # => '2011-10-13 Fill gas tank +summerVacation'
```

### 练习

*   扩大 project- 和 context-tag 生成代码来过滤掉重复的条目。

*   一些 Todo.txt 用户喜欢在任务描述中插入项目和上下文的标签。添加代码来识别这些标签和过滤器的结束标记。

## 命令模式

### 问题

你需要让另一个对象处理你自己的可执行的代码。

### 解决方案

使用 [Command pattern](http://en.wikipedia.org/wiki/Command_pattern) 传递函数的引用。

```
 # Using a private variable to simulate external scripts or modules

incrementers = (() ->
    privateVar = 0

    singleIncrementer = () ->
        privateVar += 1

    doubleIncrementer = () ->
        privateVar += 2

    commands = 
        single: singleIncrementer
        double: doubleIncrementer
        value: -> privateVar
)()

class RunsAll
    constructor: (@commands...) ->
    run: -> command() for command in @commands

runner = new RunsAll(incrementers.single, incrementers.double, incrementers.single, incrementers.double)
runner.run()
incrementers.value() # => 6
```

### 讨论

以函数作为一级的对象且从 Javascript 函数的变量范围中继承，CoffeeScript 使语言模式几乎看不出来。事实上，任何函数传递回调函数可以作为一个*命令*。

jqXHR 对象返回 jQuery AJAX 方法使用此模式。

```
jqxhr = $.ajax
    url: "/"

logMessages = ""

jqxhr.success -> logMessages += "Success!\n"
jqxhr.error -> logMessages += "Error!\n"
jqxhr.complete -> logMessages += "Completed!\n"

 # On a valid AJAX request:

 # logMessages == "Success!\nCompleted!\n"
```

## 修饰模式

### 问题

你有一组数据，需要在多个过程、可能变换的方式下处理。

### 解决方案

使用修饰模式来构造如何更改应用。

```
miniMarkdown = (line) ->
    if match = line.match /^(#+)\s*(.*)$/
        headerLevel = match[1].length
        headerText = match[2]
        "<h#{headerLevel}>#{headerText}</h#{headerLevel}>"
    else
        if line.length > 0
            "<p>#{line}</p>"
        else
            ''

stripComments = (line) ->
    line.replace /\s*\/\/.*$/, '' # Removes one-line, double-slash C-style comments

class TextProcessor
    constructor: (@processors) ->

    reducer: (existing, processor) ->
        if processor
            processor(existing or '')
        else
            existing
    processLine: (text) ->
        @processors.reduce @reducer, text
    processString: (text) ->
        (@processLine(line) for line in text.split("\n")).join("\n")

exampleText = '''
              # A level 1 header
              A regular line
              // a comment
              ## A level 2 header
              A line // with a comment
              '''

processor = new TextProcessor [stripComments, miniMarkdown]

processor.processString exampleText

 # => "<h1>A level 1 header</h1>\n<p>A regular line</p>\n\n<h2>A level 2 header</h2>\n<p>A line</p>"
```

### 结果

```
<h1>A level 1 header</h1>
<p>A regular line</p>

<h2>A level 1 header</h2>
<p>A line</p>
```

### 讨论

TextProcessor 服务有修饰的作用，可将个人、专业文本处理器绑定在一起。这使 miniMarkdown 和 stripComments 组件只专注于处理一行文本。未来的开发人员只需要编写函数返回一个字符串，并将它添加到阵列的处理器即可。

我们甚至可以修改现有的修饰对象动态：

```
smilies =
    ':)' : "smile"
    ':D' : "huge_grin"
    ':(' : "frown"
    ';)' : "wink"

smilieExpander = (line) ->
    if line
        (line = line.replace symbol, "<img src='#{text}.png' alt='#{text}' />") for symbol, text of smilies
    line

processor.processors.unshift smilieExpander

processor.processString "# A header that makes you :) // you may even laugh"

 # => "<h1>A header that makes you <img src='smile.png' alt='smile' /></h1>"

processor.processors.shift()

 # => "<h1>A header that makes you :)</h1>"
```

## 工厂方法模式

### 问题

直到开始运行你才知道需要的是什么种类的对象。

### 解决方案

使用 [工厂方法（Factory Method)](http://en.wikipedia.org/wiki/Factory_method_pattern) 模式和选择对象都是动态生成的。

你需要将一个文件加载到编辑器，但是直到用户选择文件时你才知道它的格式。一个类使用[工厂方法 ( Factory Method )](http://en.wikipedia.org/wiki/Factory_method_pattern) 模式可以根据文件的扩展名提供不同的解析器。

```
class HTMLParser
    constructor: ->
        @type = "HTML parser"
class MarkdownParser
    constructor: ->
        @type = "Markdown parser"
class JSONParser
    constructor: ->
        @type = "JSON parser"

class ParserFactory
    makeParser: (filename) ->
        matches = filename.match /\.(\w*)$/
        extension = matches[1]
        switch extension
            when "html" then new HTMLParser
            when "htm" then new HTMLParser
            when "markdown" then new MarkdownParser
            when "md" then new MarkdownParser
            when "json" then new JSONParser

factory = new ParserFactory

factory.makeParser("example.html").type # => "HTML parser"

factory.makeParser("example.md").type # => "Markdown parser"

factory.makeParser("example.json").type # => "JSON parser"
```

### 讨论

在这个示例中，你可以关注解析的内容，忽略细节文件的格式。更先进的工厂方法，例如，搜索版本控制文件中的数据本身，然后返回一个更精确的解析器(例如，返回一个 HTML5 解析器而不是 HTML v4 解析器)。

## 解释器模式

### 问题

其他人需要以控制方式运行你的一部分代码。相对地，你选择的语言不能以一种简洁的方式表达问题域。

### 解决方案

使用解释器模式来创建一个你翻译为特定代码的领域特异性语言（ domain-specific language ）。

我们来做个假设，例如用户希望在你的应用程序中执行数学运算。你可以让他们正向运行代码来演算指令（eval）但这会让他们运行任意代码。相反，你可以提供一个小型的“堆栈计算器（stack calculator）”语言，用来做单独分析，以便只运行数学运算，同时报告更有用的错误信息。

```
class StackCalculator
    parseString: (string) ->
        @stack = [ ]
        for token in string.split /\s+/
            @parseToken token

        if @stack.length > 1
            throw "Not enough operators: numbers left over"
        else
            @stack[0]

    parseToken: (token, lastNumber) ->
        if isNaN parseFloat(token) # Assume that anything other than a number is an operator
            @parseOperator token
        else
            @stack.push parseFloat(token)

    parseOperator: (operator) ->
        if @stack.length < 2
            throw "Can't operate on a stack without at least 2 items"

        right = @stack.pop()
        left = @stack.pop()

        result = switch operator
            when "+" then left + right
            when "-" then left - right
            when "*" then left * right
            when "/"
                if right is 0
                    throw "Can't divide by 0"
                else
                    left / right
            else
                throw "Unrecognized operator: #{operator}"

        @stack.push result

calc = new StackCalculator

calc.parseString "5 5 +" # => { result: 10 }

calc.parseString "4.0 5.5 +" # => { result: 9.5 }

calc.parseString "5 5 + 5 5 + *" # => { result: 100 }

try
    calc.parseString "5 0 /"
catch error
    error # => "Can't divide by 0"

try
    calc.parseString "5 -"
catch error
    error # => "Can't operate on a stack without at least 2 items"

try
    calc.parseString "5 5 5 -"
catch error
    error # => "Not enough operators: numbers left over"

try
    calc.parseString "5 5 5 foo"
catch error
    error # => "Unrecognized operator: foo"
```

### 讨论

作为一种替代编写我们自己的解释器的选择，你可以将现有的 CoffeeScript 解释器与更自然的（更容易理解的）表达自己的算法的正常方式相结合。

```
class Sandwich
    constructor: (@customer, @bread='white', @toppings=[], @toasted=false)->

white = (sw) ->
    sw.bread = 'white'
    sw

wheat = (sw) ->
    sw.bread = 'wheat'
    sw

turkey = (sw) ->
    sw.toppings.push 'turkey'
    sw

ham = (sw) ->
    sw.toppings.push 'ham'
    sw

swiss = (sw) ->
    sw.toppings.push 'swiss'
    sw

mayo = (sw) ->
    sw.toppings.push 'mayo'
    sw

toasted = (sw) ->
    sw.toasted = true
    sw

sandwich = (customer) ->
    new Sandwich customer

to = (customer) ->
    customer

send = (sw) ->
    toastedState = sw.toasted and 'a toasted' or 'an untoasted'

    toppingState = ''
    if sw.toppings.length > 0
        if sw.toppings.length > 1
            toppingState = " with #{sw.toppings[0..sw.toppings.length-2].join ', '} and #{sw.toppings[sw.toppings.length-1]}"
        else
            toppingState = " with #{sw.toppings[0]}"
    "#{sw.customer} requested #{toastedState}, #{sw.bread} bread sandwich#{toppingState}"

send sandwich to 'Charlie' # => "Charlie requested an untoasted, white bread sandwich"
send turkey sandwich to 'Judy' # => "Judy requested an untoasted, white bread sandwich with turkey"
send toasted ham turkey sandwich to 'Rachel' # => "Rachel requested a toasted, white bread sandwich with turkey and ham"
send toasted turkey ham swiss sandwich to 'Matt' # => "Matt requested a toasted, white bread sandwich with swiss, ham and turkey"
```

这个实例可以允许功能层实现返回修改后的对象，从而外函数可以依次修改它。示例通过借用动词和介词的用法，把自然语法提供给结构，当被正确使用时，会像自然语句一样结束。这样，利用 CoffeeScript 语言技能和你现有的语言技能可以帮助你关于捕捉代码的问题。

## 备忘录模式

### 问题

你想预测对一个对象做出改变后的反应。

### 解决方案

使用备忘录模式[（Memento Pattern）](http://en.wikipedia.org/wiki/Memento_pattern)来跟踪一个对象的变化。使用这个模式的类会输出一个存储在其他地方的备忘录对象。

如果你的应用程序可以让用户编辑文本文件，例如，他们可能想要撤销上一个动作。你可以在用户改变文件之前保存文件现有的状态，然后回滚到上一个位置。

```
class PreserveableText
    class Memento
        constructor: (@text) ->

    constructor: (@text) ->
    save: (newText) ->
        memento = new Memento @text
        @text = newText
        memento
    restore: (memento) ->
        @text = memento.text

pt = new PreserveableText "The original string"
pt.text # => "The original string"

memento = pt.save "A new string"
pt.text # => "A new string"

pt.save "Yet another string"
pt.text # => "Yet another string"

pt.restore memento
pt.text # => "The original string"
```

### 讨论

备忘录对象由 PreserveableText#save 返回，为了安全保护，分别地存储着重要的状态信息。你可以序列化备忘录以便来保证硬盘中的“撤销”缓冲或者是那些被编辑的图片等数据密集型对象。

## 观察者模式

### 问题

当一个事件发生时你不得不向一些对象发布公告。

### 解决方案

使用观察者模式[（Observer Pattern）](http://en.wikipedia.org/wiki/Observer_pattern)。

```
class PostOffice
    constructor: () ->
        @subscribers = []
    notifyNewItemReleased: (item) ->
        subscriber.callback(item) for subscriber in @subscribers when subscriber.item is item
    subscribe: (to, onNewItemReleased) ->
        @subscribers.push {'item':to, 'callback':onNewItemReleased}

class MagazineSubscriber
    onNewMagazine: (item) ->
        alert "I've got new "+item

class NewspaperSubscriber
    onNewNewspaper: (item) ->
        alert "I've got new "+item

postOffice = new PostOffice()
sub1 = new MagazineSubscriber()
sub2 = new NewspaperSubscriber()
postOffice.subscribe "Mens Health", sub1.onNewMagazine
postOffice.subscribe "Times", sub2.onNewNewspaper
postOffice.notifyNewItemReleased "Times"
postOffice.notifyNewItemReleased "Mens Health"
```

### 讨论

这里你有一个观察者对象（ PostOffice ）和可观察对象( MagazineSubscriber, NewspaperSubscriber ）。为了通报发布新的周期性可观察对象的事件，应该对 PostOffice 进行订阅。每一个被订阅的对象都存储在 PostOffice 的内部订阅数组中。当新的实体周期发布时每一个订阅者都会收到通知。

## 单件模式

### 问题

许多时候你想要一个，并且只要一个类的实例。比如，你可能需要一个创建服务器资源的类，并且你想要保证使用一个对象就可以控制这些资源。但是使用时要小心，因为单件模式可以很容易被滥用来模拟不必要的全局变量。

### 解决方案

公有类只包含获得一个实例的方法。实例被保存在该公共对象的闭包中，并且总是有返回值。

这很奏效因为 CoffeeScript 允许你在一个类的声明中定义可执行的状态。但是，因为大多数 CoffeeScript 编译成一个 [IIFE](http://benalman.com/news/2010/11/immediately-invoked-function-expression/) 包，如果这个方式适合你，你就不需要在类的声明中放置私有的类。之后的内容可能对开发模块化代码有所帮助，例如 [CommonJS](http://www.commonjs.org/)（Node.js）或 [Require.js](http://requirejs.org/) 中可见（见实例讨论）。

```
class Singleton
  # You can add statements inside the class definition
  # which helps establish private scope (due to closures)
  # instance is defined as null to force correct scope
  instance = null
  # Create a private class that we can initialize however
  # defined inside this scope to force the use of the
  # singleton class.
  class PrivateClass
    constructor: (@message) ->
    echo: -> @message
  # This is a static method used to either retrieve the
  # instance or create a new one.
  @get: (message) ->
    instance ?= new PrivateClass(message)

a = Singleton.get "Hello A"
a.echo() # => "Hello A"

b = Singleton.get "Hello B"
b.echo() # => "Hello A"

Singleton.instance # => undefined
a.instance # => undefined
Singleton.PrivateClass # => undefined
```

### 讨论

通过上面的实例我们可以看到，所有的实例是如何从同一个 Singleton 类的实例中输出的。你也可以看到，私有类和实例变量都无法在 Singleton class 外被访问到。 Singleton class 的本质是提供一个静态方法得到只返回一个私有类的实例。它也对外界也隐藏私有类，因此你无法创建一个自己的私有类。

隐藏或使私有类在内部运作的想法是更受偏爱的。尤其是由于缺省的 CoffeeScript 将编译的代码封装在自己的 IIFE（闭包）中，你可以定义类而无须担心会被文件外部访问到。在这个实例中，注意，用惯用的模块导出特点来强调模块中可被公共访问的部分。（请看 “[导出到全局命名空间](http://stackoverflow.com/questions/4214731/coffeescript-global-variables)” 中对此理解更深入的讨论）。

```
root = exports ? this

 # Create a private class that we can initialize however

 # defined inside the wrapper scope.

class ProtectedClass
  constructor: (@message) ->
  echo: -> @message

class Singleton
  # You can add statements inside the class definition
  # which helps establish private scope (due to closures)
  # instance is defined as null to force correct scope
  instance = null
  # This is a static method used to either retrieve the
  # instance or create a new one.
  @get: (message) ->
    instance ?= new ProtectedClass(message)

 # Export Singleton as a module

root.Singleton = Singleton
```

我们可以注意到 coffeescript 是如此简单地实现这个设计模式。为了更好地参考和讨论 JavaScript 的实现，请看[初学者必备 JavaScript 设计模式](http://addyosmani.com/resources/essentialjsdesignpatterns/book/)。

## 策略模式

### 问题

解决问题的方式有多种，但是你需要在程序运行时选择（或是转换）这些方法。

### 解决方案

在策略对象（Strategy objects）中封装你的算法。

例如，给定一个未排序的列表，我们可以在不同情况下改变排序算法。

#### 基类

```
StringSorter = (algorithm) ->
    sort: (list) -> algorithm list
```

#### 策略

```
bubbleSort = (list) ->
    anySwaps = false
    swapPass = ->
        for r in [0..list.length-2]
            if list[r] > list[r+1]
                anySwaps = true
                [list[r], list[r+1]] = [list[r+1], list[r]]

    swapPass()
    while anySwaps
        anySwaps = false
        swapPass()
    list

reverseBubbleSort = (list) ->
    anySwaps = false
    swapPass = ->
        for r in [list.length-1..1]
            if list[r] < list[r-1]
                anySwaps = true
                [list[r], list[r-1]] = [list[r-1], list[r]]

    swapPass()
    while anySwaps
        anySwaps = false
        swapPass()
    list
```

#### 使用策略

```
sorter = new StringSorter bubbleSort

unsortedList = ['e', 'b', 'd', 'c', 'x', 'a']

sorter.sort unsortedList

 # => ['a', 'b', 'c', 'd', 'e', 'x']

unsortedList.push 'w'

 # => ['a', 'b', 'c', 'd', 'e', 'x', 'w']

sorter.algorithm = reverseBubbleSort

sorter.sort unsortedList

 # => ['a', 'b', 'c', 'd', 'e', 'w', 'x']
```

### 讨论

“没有作战计划在第一次接触敌人时便能存活下来。” 用户如是，但是我们可以运用从变化的情况中获得的知识来做出适应改变。在示例末尾，例如，数组中的最新项是乱序排列的，知道了这个细节，我们便可以通过切换算法来加速排序，只要简单地重赋值就可以了。

### 练习

*   将 StringSorter 扩展为 AlwaysSortedArray 类来实现规则序列的所有功能，但是要基于插入方法自动分类新的项（例如 push 对比 shift）。

## 模板方法模式

### 问题

定义一个算法的结构，作为一系列的高层次的步骤，使每一个步骤的行为可以指定，使属于一个族的算法都具有相同的结构但是有不同的行为。

### 解决方案

使用模板方法（ Template Method ）在父类中描述算法的结构，再授权一个或多个具体子类来具体地进行实现。

例如，想象你希望模拟各种类型的文件的生成，并且每个文件要包含一个标题和正文。

```
class Document
    produceDocument: ->
        @produceHeader()
        @produceBody()

    produceHeader: ->
    produceBody: ->

class DocWithHeader extends Document
    produceHeader: ->
        console.log "Producing header for DocWithHeader"

    produceBody: ->
        console.log "Producing body for DocWithHeader"

class DocWithoutHeader extends Document
    produceBody: ->
        console.log "Producing body for DocWithoutHeader"

docs = [new DocWithHeader, new DocWithoutHeader]
doc.produceDocument() for doc in docs
```

### 讨论

在这个实例中，算法用两个步骤来描述文件的生成：其一是产生文件的标题，另一步是生成文件的正文。父类中是实现每一个步骤的空的方法，多态性使得每一个具体的子类可以通过重写一步步的方法来实现对方法不同的利用。在本实例中，DocWithHeader 实现了正文和标题的步骤， DocWithoutHeader 只是实现了正文的步骤。

不同类型文件的生成就是简单的将文档对象存储在一个数组中，简单的遍历每个文档对象并调用其 produceDocument 方法的问题。