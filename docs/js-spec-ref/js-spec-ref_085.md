# 11.1 Underscore.js

*   概述
*   集合相关方法
    *   集合处理
    *   集合特征
    *   集合过滤
    *   对象相关方法
    *   与函数相关的方法
        *   绑定运行环境和参数
        *   函数运行控制
    *   工具方法
        *   链式操作
        *   template
    *   参考链接

## 概述

[Underscore.js](http://underscorejs.org/)是一个很精干的库，压缩后只有 4KB。它提供了几十种函数式编程的方法，弥补了标准库的不足，大大方便了 JavaScript 的编程。MVC 框架 Backbone.js 就将这个库作为自己的工具库。除了可以在浏览器环境使用，Underscore.js 还可以用于 Node.js。

Underscore.js 定义了一个下划线（_）对象，函数库的所有方法都属于这个对象。这些方法大致上可以分成：集合（collection）、数组（array）、函数（function）、对象（object）和工具（utility）五大类。

## 集合相关方法

Javascript 语言的数据集合，包括两种结构：数组和对象。以下的方法同时适用于这两种结构。

### 集合处理

数组处理指的是对数组元素进行加工。

（1）map

map 方法对集合的每个成员依次进行某种操作，将返回的值依次存入一个新的数组。

```js
_.map([1, 2, 3], function(num){ return num * 3; });
// [3, 6, 9]

_.map({one : 1, two : 2, three : 3}, function(num, key){ return num * 3; });
// [3, 6, 9]
```

（2）each

each 方法与 map 类似，依次对数组所有元素进行某种操作，不返回任何值。

```js
_.each([1, 2, 3], alert);

_.each({one : 1, two : 2, three : 3}, alert);
```

（3）reduce

reduce 方法依次对集合的每个成员进行某种操作，然后将操作结果累计在某一个初始值之上，全部操作结束之后，返回累计的值。该方法接受三个参数。第一个参数是被处理的集合，第二个参数是对每个成员进行操作的函数，第三个参数是累计用的变量。

```js
_.reduce([1, 2, 3], function(memo, num){ return memo + num; }, 0);
// 6
```

reduce 方法的第二个参数是操作函数，它本身又接受两个参数，第一个是累计用的变量，第二个是集合每个成员的值。

（4）reduceRight

reduceRight 是逆向的 reduce，表示从集合的最后一个元素向前进行处理。

```js
var list = [[0, 1], [2, 3], [4, 5]];
var flat = _.reduceRight(list, function(a, b) { return a.concat(b); }, []);
// [4, 5, 2, 3, 0, 1]
```

（5）shuffle

shuffle 方法返回一个打乱次序的集合。

```js
_.shuffle([1, 2, 3, 4, 5, 6]);
// [4, 1, 6, 3, 5, 2]
```

（6）invoke

invoke 方法对集合的每个成员执行指定的操作。

```js
_.invoke([[5, 1, 7], [3, 2, 1]], 'sort')
// [[1, 5, 7], [1, 2, 3]]
```

（7）sortBy

sortBy 方法根据处理函数的返回值，返回一个排序后的集合，以升序排列。

```js
_.sortBy([1, 2, 3, 4, 5, 6], function(num){ return Math.sin(num); });
// [5, 4, 6, 3, 1, 2]
```

（8）indexBy

indexBy 方法返回一个对象，根据指定键名，对集合生成一个索引。

```js
var person = [{name: 'John', age: 40}, {name: 'larry', age: 50}, {name: 'curly', age: 60}];
_.indexBy(person, 'age');
// { "50": {name: 'larry', age: 50},
     "60": {name: 'curly', age: 60} }
```

### 集合特征

Underscore.js 提供了一系列方法，判断数组元素的特征。这些方法都返回一个布尔值，表示是否满足条件。

（1）every

every 方法判断数组的所有元素是否都满足某个条件。如果都满足则返回 true，否则返回 false。

```js
_.every([1, 2, 3, 4, 5, 6], function(num){ return num % 2 == 0; });
// false
```

（2）some

some 方法则是只要有一个元素满足，就返回 true，否则返回 false。

```js
_.some([1, 2, 3, 4, 5, 6], function(num){ return num % 2 == 0; });
// true

_.some([null, 0, 'yes', false])
// true
```

（3）size

size 方法返回集合的成员数量。

```js
_.size({one : 1, two : 2, three : 3});
// 3
```

（4）sample

sample 方法用于从集合中随机取样。

```js
_.sample([1, 2, 3, 4, 5, 6])
// 4
```

### 集合过滤

Underscore.js 提供了一系列方法，用于过滤数组，找到符合要求的成员。

（1）filter

filter 方法依次对集合的每个成员进行某种操作，只返回操作结果为 true 的成员。

```js
_.filter([1, 2, 3, 4, 5, 6], function(num){ return num % 2 == 0; });
// [2, 4, 6]
```

（2）reject

reject 方法只返回操作结果为 false 的成员。

```js
_.reject([1, 2, 3, 4, 5, 6], function(num){ return num % 2 == 0; });
// [1, 3, 5]
```

（3）find

find 方法依次对集合的每个成员进行某种操作，返回第一个操作结果为 true 的成员。如果所有成员的操作结果都为 false，则返回 undefined。

```js
_.find([1, 2, 3, 4, 5, 6], function(num){ return num % 2 == 0; });
// 2
```

（4）contains

contains 方法表示如果某个值在数组内，则返回 true，否则返回 false。

```js
_.contains([1, 2, 3], 3);
// true
```

（5）countBy

countBy 方法依次对集合的每个成员进行某种操作，将操作结果相同的成员算作一类，最后返回一个对象，表明每种操作结果对应的成员数量。

```js
_.countBy([1, 2, 3, 4, 5], function(num) {
  return num % 2 == 0 ? 'even' : 'odd';
});
// {odd: 3, even: 2}
```

（6）where

where 方法检查集合中的每个值，返回一个数组，其中的每个成员都包含指定的键值对。

```js
_.where(listOfPlays, {author: "Shakespeare", year: 1611});
// [{title: "Cymbeline", author: "Shakespeare", year: 1611},
//  {title: "The Tempest", author: "Shakespeare", year: 1611}]
```

（7）max，min

max 方法返回集合中的最大值。如果提供一个处理函数，则该函数的返回值用作排名标准。

```js
var person = [{name: 'John', age: 40}, 
              {name: 'larry', age: 50}, 
              {name: 'curly', age: 60}];
_.max(person, function(per){ return per.age; });
// {name: 'curly', age: 60};
```

min 方法返回集合中的最小值。如果提供一个处理函数，则该函数的返回值用作排名标准。

```js
var numbers = [10, 5, 100, 2, 1000];
_.min(numbers)
// 2
```

## 对象相关方法

（1）toArray

toArray 方法将对象转为数组，只包含对象成员的值。典型应用是将对类似数组的对象转为真正的数组。

```js
_.toArray({a:0,b:1,c:2});
// [0, 1, 2]
```

（2）pluck

pluck 方法将多个对象的某一个属性的值，提取成一个数组。

```js
var person = [{name: 'John', age: 40}, 
              {name: 'larry', age: 50}, 
              {name: 'curly', age: 60}];

_.pluck(person, 'name');
// ["moe", "larry", "curly"]
```

## 与函数相关的方法

### 绑定运行环境和参数

在不同的运行环境下，JavaScript 函数内部的变量所在的上下文是不同的。这种特性会给程序带来不确定性，为了解决这个问题，Underscore.js 提供了两个方法，用来给函数绑定上下文。

（1）bind 方法

该方法绑定函数运行时的上下文，返回一个新函数。

```js
var o = {
    p: 2,
    m: function (){console.log(this.p);}
};

o.m()
// 2

_.bind(o.m,{p:1})()
// 1
```

上面代码将 o.m 方法绑定到一个新的对象上面。

除了前两个参数以外，bind 方法还可以接受更多参数，它们表示函数方法运行时所需的参数。

```js
var add = function(n1,n2,n3) {
  console.log(this.sum + n1 + n2 + n3);
};

_.bind(add, {sum:1}, 1, 1, 1)()
// 4
```

上面代码中 bind 方法有 5 个参数，最后那三个是给定 add 方法的运行参数，所以运行结果为 4。

（2）bindall 方法

该方法可以一次将多个方法，绑定在某个对象上面。

```js
var o = {
  p1 : '123',
  p2 : '456',
  m1 : function() { console.log(this.p1); },
  m2 : function() { console.log(this.p2); },
};

_.bindAll(o, 'm1', 'm2');
```

上面代码一次性将两个方法（m1 和 m2）绑定在 o 对象上面。

（3）partial 方法

除了绑定上下文，Underscore.js 还允许绑定参数。partial 方法将函数与某个参数绑定，然后作为一个新函数返回。

```js
var add = function(a, b) { return a + b; };

add5 = _.partial(add, 5);

add5(10);
// 15
```

（4）wrap 方法

该方法将一个函数作为参数，传入另一个函数，最终返回前者的一个新版本。

```js
var hello = function(name) { return "hello: " + name; };

hello = _.wrap(hello, function(func) {
  return "before, " + func("moe") + ", after";
});

hello();
// 'before, hello: moe, after'
```

上面代码先定义 hello 函数，然后将 hello 传入一个匿名定义，返回一个新版本的 hello 函数。

（5）compose 方法

该方法接受一系列函数作为参数，由后向前依次运行，上一个函数的运行结果，作为后一个函数的运行参数。也就是说，将 f(g(),h())的形式转化为 f(g(h()))。

```js
var greet    = function(name){ return "hi: " + name; };
var exclaim  = function(statement){ return statement + "!"; };
var welcome = _.compose(exclaim, greet);
welcome('moe');
// 'hi: moe!'
```

上面代码调用 welcome 时，先运行 greet 函数，再运行 exclaim 函数。并且，greet 函数的运行结果是 exclaim 函数运行时的参数。

### 函数运行控制

Underscore.js 允许对函数运行行为进行控制。

（1）memoize 方法

该方法缓存一个函数针对某个参数的运行结果。

```js
var fibonacci = _.memoize(function(n) {
  return n < 2 ? n : fibonacci(n - 1) + fibonacci(n - 2);
});
```

（2）delay 方法

该方法可以将函数推迟指定的时间再运行。

```js
var log = _.bind(console.log, console);

_.delay(log, 1000, 'logged later');
// 'logged later'
```

上面代码推迟 1000 毫秒，再运行 console.log 方法，并且指定参数为“logged later”。

（3）defer 方法

该方法可以将函数推迟到待运行的任务数为 0 时再运行，类似于 setTimeout 推迟 0 秒运行的效果。

```js
_.defer(function(){ alert('deferred'); });
```

（4）throttle 方法

该方法返回一个函数的新版本。连续调用这个新版本的函数时，必须等待一定时间才会触发下一次执行。

```js
// 返回 updatePosition 函数的新版本
var throttled = _.throttle(updatePosition, 100);

// 新版本的函数每过 100 毫秒才会触发一次
$(window).scroll(throttled);
```

（5）debounce 方法

该方法返回的新函数有调用的时间限制，每次调用必须与上一次调用间隔一定的时间，否则就无效。它的典型应用是防止用户双击某个按钮，导致两次提交表单。

```js
$("button").on("click", _.debounce(submitForm, 1000, true));
```

上面代码表示 click 事件发生后，调用函数 submitForm 的新版本。该版本的两次运行时间，必须间隔 1000 毫秒以上，否则第二次调用无效。最后那个参数 true，表示 click 事件发生后，立刻触发第一次 submitForm 函数，否则就是等 1000 毫秒再触发。

（6）once 方法

该方法返回一个只能运行一次的新函数。该方法主要用于对象的初始化。

```js
var initialize = _.once(createApplication);
initialize();
initialize();
// Application 只被创造一次
```

（7）after 方法

该方法返回的新版本函数，只有在被调用一定次数后才会运行，主要用于确认一组操作全部完成后，再做出反应。

```js
var renderNotes = _.after(notes.length, render);

_.each(notes, function(note) {
  note.asyncSave({success: renderNotes});
});
```

上面代码表示，函数 renderNotes 是函数 render 的新版本，只有调用 notes.length 次以后才会运行。所以，后面就可以放心地等到 notes 的每个成员都处理完，才会运行一次 renderNotes。

## 工具方法

### 链式操作

Underscore.js 允许将多个操作写成链式的形式。

```js
_.(users)
.filter(function(user) { return user.name === name })
.sortBy(function(user) { return user.karma })
.first()
.value()
```

### template

该方法用于编译 HTML 模板。它接受三个参数。

```js
_.template(templateString, [data], [settings])
```

三个参数的含义如下：

*   templateString：模板字符串
*   data：输入模板的数据
*   settings：设置

（1）templateString

模板字符串 templateString 就是普通的 HTML 语言，其中的变量使用的形式插入；data 对象负责提供变量的值。

```js
var txt = "<h2><%= word %></h2>";

_.template(txt, {word : "Hello World"})
// "<h2>Hello World</h2>"
```

如果变量的值包含五个特殊字符（& " ' /），就需要用转义。

```js
var txt = "<h2><%- word %></h2>";

_.template(txt, {word : "H & W"})
// <h2>H &amp; W</h2>
```

JavaScript 命令可以采用的形式插入。下面是判断语句的例子。

```js
var txt = "<% var i = 0; if (i<1){ %>"
        + "<%= word %>"
        + "<% } %>";

_.template(txt, {word : "Hello World"})
// Hello World
```

常见的用法还有循环语句。

```js
var list = "<% _.each(people, function(name) { %> <li><%= name %></li> <% }); %>";

_.template(list, {people : ['moe', 'curly', 'larry']});
// "<li>moe</li><li>curly</li><li>larry</li>"
```

如果 template 方法只有第一个参数 templateString，省略第二个参数，那么会返回一个函数，以后可以向这个函数输入数据。

```js
var t1 = _.template("Hello <%=user%>!");  

t1({ user: "<Jane>" }) 
// 'Hello <Jane>!'
```

**（2）data**

templateString 中的所有变量，在内部都是 obj 对象的属性，而 obj 对象就是指第二个参数 data 对象。下面两句语句是等同的。

```js
_.template("Hello <%=user%>!", { user: "<Jane>" })
_.template("Hello <%=obj.user%>!", { user: "<Jane>" })
```

如果要改变 obj 这个对象的名字，需要在第三个参数中设定。

```js
_.template("<%if (data.title) { %>Title: <%= title %><% } %>", null,
                { variable: "data" });
```

因为 template 在变量替换时，内部使用 with 语句，所以上面这样的做法，运行速度会比较快。

## 参考链接

*   Amy Lee, [Using Underscore.js's debounce() to filter double-clicks](http://eng.wealthfront.com/2012/12/using-underscorejss-debounce-to-filter.html)
*   Axel Rauschmayer, [A closer look at Underscore templates](http://www.2ality.com/2012/06/underscore-templates.html)
*   Jules Boussekeyt, [Write concise code with UnderscoreJS](http://jules.boussekeyt.org/2012/underscorejs.html)