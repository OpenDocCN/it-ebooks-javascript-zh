# 12.2 jQuery 工具方法

jQuery 函数库提供了一个 jQuery 对象（简写为$），这个对象本身是一个构造函数，可以用来生成 jQuery 对象的实例。有了实例以后，就可以调用许多针对实例的方法，它们定义 jQuery.prototype 对象上面（简写为$.fn）。

除了实例对象的方法以外，jQuery 对象本身还提供一些方法（即直接定义 jQuery 对象上面），不需要生成实例就能使用。由于这些方法类似“通用工具”的性质，所以我们把它们称为“工具方法”（utilities）。

*   常用工具方法
*   判断数据类型的方法
*   Ajax 操作
    *   $.ajax
    *   简便写法
    *   Ajax 事件
    *   返回值
    *   JSONP
    *   文件上传
    *   参考链接

## 常用工具方法

（1）$.trim

$.trim 方法用于移除字符串头部和尾部多余的空格。

```
$.trim('   Hello   ') // Hello
```

（2）$.contains

$.contains 方法返回一个布尔值，表示某个 DOM 元素（第二个参数）是否为另一个 DOM 元素（第一个参数）的下级元素。

```
$.contains(document.documentElement, document.body); 
// true

$.contains(document.body, document.documentElement); 
// false
```

（3）$.each，$.map

$.each 方法用于遍历数组和对象，然后返回原始对象。它接受两个参数，分别是数据集合和回调函数。

```
$.each([ 52, 97 ], function( index, value ) {
  console.log( index + ": " + value );
});
// 0: 52 
// 1: 97 

var obj = {
  p1: "hello",
  p2: "world"
};
$.each( obj, function( key, value ) {
  console.log( key + ": " + value );
});
// p1: hello
// p2: world
```

需要注意的，jQuery 对象实例也有一个 each 方法（$.fn.each），两者的作用差不多。

$.map 方法也是用来遍历数组和对象，但是会返回一个新对象。

```
var a = ["a", "b", "c", "d", "e"];
a = $.map(a, function (n, i){
  return (n.toUpperCase() + i);
});
// ["A0", "B1", "C2", "D3", "E4"]
```

（4）$.inArray

$.inArray 方法返回一个值在数组中的位置（从 0 开始）。如果该值不在数组中，则返回-1。

```
var a = [1,2,3,4];
$.inArray(4,a) // 3
```

（5）$.extend

$.extend 方法用于将多个对象合并进第一个对象。

```
var o1 = {p1:'a',p2:'b'};
var o2 = {p1:'c'};

$.extend(o1,o2);
o1.p1 // "c"
```

$.extend 的另一种用法是生成一个新对象，用来继承原有对象。这时，它的第一个参数应该是一个空对象。

```
var o1 = {p1:'a',p2:'b'};
var o2 = {p1:'c'};

var o = $.extend({},o1,o2);
o
// Object {p1: "c", p2: "b"}
```

默认情况下，extend 方法生成的对象是“浅拷贝”，也就是说，如果某个属性是对象或数组，那么只会生成指向这个对象或数组的指针，而不会复制值。如果想要“深拷贝”，可以在 extend 方法的第一个参数传入布尔值 true。

```
var o1 = {p1:['a','b']};

var o2 = $.extend({},o1);
var o3 = $.extend(true,{},o1);

o1.p1[0]='c';

o2.p1 // ["c", "b"]
o3.p1 // ["a", "b"]
```

上面代码中，o2 是浅拷贝，o3 是深拷贝。结果，改变原始数组的属性，o2 会跟着一起变，而 o3 不会。

（6）$.proxy

$.proxy 方法类似于 ECMAScript 5 的 bind 方法，可以绑定函数的上下文（也就是 this 对象）和参数，返回一个新函数。

jQuery.proxy()的主要用处是为回调函数绑定上下文对象。

```
var o = {
    type: "object",
    test: function(event) {
        console.log(this.type);
    }
};

$("#button")
  .on("click", o.test) // 无输出
  .on("click", $.proxy(o.test, o)) // object
```

上面的代码中，第一个回调函数没有绑定上下文，所以结果为空，没有任何输出；第二个回调函数将上下文绑定为对象 o，结果就为 object。

这个例子的另一种等价的写法是：

```
$("#button").on( "click", $.proxy(o, test))
```

上面代码的$.proxy(o, test)的意思是，将 o 的方法 test 与 o 绑定。

这个例子表明，proxy 方法的写法主要有两种。

```
jQuery.proxy(function, context)

// or

jQuery.proxy(context, name)
```

第一种写法是为函数（function）指定上下文对象（context），第二种写法是指定上下文对象（context）和它的某个方法名（name）。

再看一个例子。正常情况下，下面代码中的 this 对象指向发生 click 事件的 DOM 对象。

```
$('#myElement').click(function() {
    $(this).addClass('aNewClass');
});
```

如果我们想让回调函数延迟运行，使用 setTimeout 方法，代码就会出错，因为 setTimeout 使得回调函数在全局环境运行，this 将指向全局对象。

```
$('#myElement').click(function() {
    setTimeout(function() {
        $(this).addClass('aNewClass');
    }, 1000);
});
```

上面代码中的 this，将指向全局对象 window，导致出错。

这时，就可以用 proxy 方法，将 this 对象绑定到 myElement 对象。

```
$('#myElement').click(function() {
    setTimeout($.proxy(function() {
        $(this).addClass('aNewClass'); 
    }, this), 1000);
});
```

（7）$.data，$.removeData

$.data 方法可以用来在 DOM 节点上储存数据。

```
// 存入数据
$.data(document.body, "foo", 52 );

// 读取数据
$.data(document.body, "foo");

// 读取所有数据
$.data(document.body);
```

上面代码在网页元素 body 上储存了一个键值对，键名为“foo”，键值为 52。

$.removeData 方法用于移除$.data 方法所储存的数据。

```
$.data(div, "test1", "VALUE-1");
$.removeData(div, "test1");
```

（8）$.parseHTML，$.parseJSON，$.parseXML

$.parseHTML 方法用于将字符串解析为 DOM 对象。

$.parseJSON 方法用于将 JSON 字符串解析为 JavaScript 对象，作用与原生的 JSON.parse()类似。但是，jQuery 没有提供类似 JSON.stringify()的方法，即不提供将 JavaScript 对象转为 JSON 对象的方法。

$.parseXML 方法用于将字符串解析为 XML 对象。

```
var html = $.parseHTML("hello, <b>my name is</b> jQuery.");
var obj = $.parseJSON('{"name": "John"}');

var xml = "<rss version='2.0'><channel><title>RSS Title</title></channel></rss>";
var xmlDoc = $.parseXML(xml);
```

（9）$.makeArray

$.makeArray 方法将一个类似数组的对象，转化为真正的数组。

```
var a = $.makeArray(document.getElementsByTagName("div"));
```

（10）$.merge

$.merge 方法用于将一个数组（第二个参数）合并到另一个数组（第一个参数）之中。

```
var a1 = [0,1,2];
var a2 = [2,3,4];
$.merge(a1, a2);

a1
// [0, 1, 2, 2, 3, 4]
```

（11）$.now

$.now 方法返回当前时间距离 1970 年 1 月 1 日 00:00:00 UTC 对应的毫秒数，等同于(new Date).getTime()。

```
$.now()
// 1388212221489
```

## 判断数据类型的方法

jQuery 提供一系列工具方法，用来判断数据类型，以弥补 JavaScript 原生的 typeof 运算符的不足。以下方法对参数进行判断，返回一个布尔值。

*   jQuery.isArray()：是否为数组。
*   jQuery.isEmptyObject()：是否为空对象（不含可枚举的属性）。
*   jQuery.isFunction()：是否为函数。
*   jQuery.isNumeric()：是否为数值（整数或浮点数）。
*   jQuery.isPlainObject()：是否为使用“{}”或“new Object”生成的对象，而不是浏览器原生提供的对象。
*   jQuery.isWindow()：是否为 window 对象。
*   jQuery.isXMLDoc()：判断一个 DOM 节点是否处于 XML 文档之中。

下面是一些例子。

```
$.isEmptyObject({}) // true
$.isPlainObject(document.location) // false
$.isWindow(window) // true
$.isXMLDoc(document.body) // false
```

除了上面这些方法以外，还有一个$.type 方法，可以返回一个变量的数据类型。它的实质是用 Object.prototype.toString 方法读取对象内部的[[Class]]属性（参见《标准库》的 Object 对象一节）。

```
$.type(/test/) // "regexp"
```

## Ajax 操作

### $.ajax

jQuery 对象上面还定义了 Ajax 方法（$.ajax()），用来处理 Ajax 操作。调用该方法后，浏览器就会向服务器发出一个 HTTP 请求。

$.ajax()的用法主要有两种。

```
$.ajax(url[, options])
$.ajax([options])
```

上面代码中的 url，指的是服务器网址，options 则是一个对象参数，设置 Ajax 请求的具体参数。

```
$.ajax({
  async: true,
  url: '/url/to/json',
  type: 'GET',
  data : { id : 123 },
  dataType: 'json',
  timeout: 30000,
  success: successCallback,
  error: errorCallback,
  complete: completeCallback,
  statusCode: {
        404: handler404,
        500: handler500
  }
})

function successCallback(json) {
    $('<h1/>').text(json.title).appendTo('body');
}

function errorCallback(xhr, status){
    console.log('出问题了！');
}

function completeCallback(xhr, status){
    console.log('Ajax 请求已结束。');
}
```

上面代码的对象参数有多个属性，含义如下：

*   accepts：将本机所能处理的数据类型，告诉服务器。
*   async：该项默认为 true，如果设为 false，则表示发出的是同步请求。
*   beforeSend：指定发出请求前，所要调用的函数，通常用来对发出的数据进行修改。
*   cache：该项默认为 true，如果设为 false，则浏览器不缓存返回服务器返回的数据。注意，浏览器本身就不会缓存 POST 请求返回的数据，所以即使设为 false，也只对 HEAD 和 GET 请求有效。
*   complete：指定当 HTTP 请求结束时（请求成功或请求失败的回调函数，此时已经运行完毕）的回调函数。不管请求成功或失败，该回调函数都会执行。它的参数为发出请求的原始对象以及返回的状态信息。
*   contentType：发送到服务器的数据类型。
*   context：指定一个对象，作为所有 Ajax 相关的回调函数的 this 对象。
*   crossDomain：该属性设为 true，将强制向相同域名发送一个跨域请求（比如 JSONP）。
*   data：向服务器发送的数据，如果使用 GET 方法，此项将转为查询字符串，附在网址的最后。
*   dataType：向服务器请求的数据类型，可以设为 text、html、script、json、jsonp 和 xml。
*   error：请求失败时的回调函数，函数参数为发出请求的原始对象以及返回的状态信息。
*   headers：指定 HTTP 请求的头信息。
*   ifModified：如果该属性设为 true，则只有当服务器端的内容与上次请求不一样时，才会发出本次请求。
*   jsonp：指定 JSONP 请求“callback=?”中的 callback 的名称。
*   jsonpCallback: 指定 JSONP 请求中回调函数的名称。
*   mimeType：指定 HTTP 请求的 mime type。
*   password：指定 HTTP 认证所需要的密码。
*   statusCode：值为一个对象，为服务器返回的状态码，指定特别的回调函数。
*   success：请求成功时的回调函数，函数参数为服务器传回的数据、状态信息、发出请求的原始对象。
*   timeout: 等待的最长毫秒数。如果过了这个时间，请求还没有返回，则自动将请求状态改为失败。
*   type：向服务器发送信息所使用的 HTTP 动词，默认为 GET，其他动词有 POST、PUT、DELETE。
*   url：服务器端网址。这是唯一必需的一个属性，其他属性都可以省略。
*   username：指定 HTTP 认证的用户名。
*   xhr：指定生成 XMLHttpRequest 对象时的回调函数。

这些参数之中，url 可以独立出来，作为 ajax 方法的第一个参数。也就是说，上面代码还可以写成下面这样。

```
$.ajax('/url/to/json',{
  type: 'GET',
  dataType: 'json',
  success: successCallback,
  error: errorCallback
})
```

作为向服务器发送的数据，data 属性也可以写成一个对象。

```
$.ajax({
  url: '/remote/url',
  data: {
    param1: 'value1',
    param2: 'value2',
    ...
  }
});

// 相当于
$.ajax({
    url: '/remote/url?param1=value1&param2=value2...'
}});
```

### 简便写法

ajax 方法还有一些简便写法。

*   $.get()：发出 GET 请求。
*   $.getScript()：读取一个 JavaScript 脚本文件并执行。
*   $.getJSON()：发出 GET 请求，读取一个 JSON 文件。
*   $.post()：发出 POST 请求。
*   $.fn.load()：读取一个 html 文件，并将其放入当前元素之中。

一般来说，这些简便方法依次接受三个参数：url、数据、成功时的回调函数。

（1）$.get()，$.post()

这两个方法分别对应 HTTP 的 GET 方法和 POST 方法。

```
$.get('/data/people.html', function(html){
  $('#target').html(html);
});

$.post('/data/save', {name: 'Rebecca'}, function (resp){
  console.log(JSON.parse(resp));
});
```

get 方法和 post 方法的参数相同，第一个参数是服务器网址，该参数是必需的，其他参数都是可选的。第二个参数是发送给服务器的数据，第三个参数是操作成功后的回调函数。

上面的 post 方法对应的 ajax 写法如下。

```
$.ajax({
    type: 'POST',
    url: '/data/save',
    data: {name: 'Rebecca'},
    dataType: 'json',
    success: function (resp){
      console.log(JSON.parse(resp));
    }
});
```

（2）$.getJSON()

ajax 方法的另一个简便写法是 getJSON 方法。当服务器端返回 JSON 格式的数据，可以用这个方法代替$.ajax 方法。

```
$.getJSON('url/to/json', {'a': 1}, function(data){
    console.log(data);
});
```

上面的代码等同于下面的写法。

```
$.ajax({
  dataType: "json",
  url: '/url/to/data',
  data: {'a': 1},
  success: function(data){
    console.log(data);
  }
});
```

（3）$.getScript()

$.getScript 方法用于从服务器端加载一个脚本文件。

```
$.getScript('/static/js/myScript.js', function() {
    functionFromMyScript();
});
```

上面代码先从服务器加载 myScript.js 脚本，然后在回调函数中执行该脚本提供的函数。

getScript 的回调函数接受三个参数，分别是脚本文件的内容，HTTP 响应的状态信息和 ajax 对象实例。

```
$.getScript( "ajax/test.js", function (data, textStatus, jqxhr){
  console.log( data ); // test.js 的内容
  console.log( textStatus ); // Success
  console.log( jqxhr.status ); // 200
});
```

getScript 是 ajax 方法的简便写法，因此返回的是一个 deferred 对象，可以使用 deferred 接口。

```
jQuery.getScript("/path/to/myscript.js")
    .done(function() {
        // ...
    })
    .fail(function() {
        // ...
});
```

（4）$.fn.load()

$.fn.load 不是 jQuery 的工具方法，而是定义在 jQuery 对象实例上的方法，用于获取服务器端的 HTML 文件，将其放入当前元素。由于该方法也属于 ajax 操作，所以放在这里一起讲。

```
$('#newContent').load('/foo.html');
```

$.fn.load 方法还可以指定一个选择器，将远程文件中匹配选择器的部分，放入当前元素，并指定操作完成时的回调函数。

```
$('#newContent').load('/foo.html #myDiv h1:first',
    function(html) {
        console.log('内容更新！');
});
```

上面代码只加载 foo.html 中匹配“#myDiv h1:first”的部分，加载完成后会运行指定的回调函数。

```
$('#main-menu a').click(function(event) {
   event.preventDefault();

   $('#main').load(this.href + ' #main *');
});
```

上面的代码将指定网页中匹配“#main *”，加载入当前的 main 元素。星号表示匹配 main 元素包含的所有子元素，如果不加这个星号，就会加载整个 main 元素（包括其本身），导致一个 main 元素中还有另一个 main 元素。

load 方法可以附加一个字符串或对象作为参数，一起向服务器提交。如果是字符串，则采用 GET 方法提交；如果是对象，则采用 POST 方法提交。

```
$( "#feeds" ).load( "feeds.php", { limit: 25 }, function() {
  console.log( "已经载入" );
});
```

上面代码将`{ limit: 25 }`通过 POST 方法向服务器提交。

load 方法的回调函数，可以用来向用户提示操作已经完成。

```
$('#main-menu a').click(function(event) {
   event.preventDefault();

   $('#main').load(this.href + ' #main *', function(responseText, status) {
      if (status === 'success') {
         $('#notification-bar').text('加载成功！');
      } else {
         $('#notification-bar').text('出错了！');
      }
   });
});
```

### Ajax 事件

jQuery 提供以下一些方法，用于指定特定的 AJAX 事件的回调函数。

*   .ajaxComplete()：ajax 请求完成。
*   .ajaxError()：ajax 请求出错。
*   .ajaxSend()：ajax 请求发出之前。
*   .ajaxStart()：第一个 ajax 请求开始发出，即没有还未完成 ajax 请求。
*   .ajaxStop()：所有 ajax 请求完成之后。
*   .ajaxSuccess()：ajax 请求成功之后。

下面是示例。

```
$('#loading_indicator')
.ajaxStart(function (){$(this).show();})
.ajaxStop(function (){$(this).hide();});
```

### 返回值

ajax 方法返回的是一个 deferred 对象，可以用 then 方法为该对象指定回调函数（详细解释参见《deferred 对象》一节）。

```
$.ajax({
  url: '/data/people.json',
  dataType: 'json'
}).then(function (resp){
  console.log(resp.people);
})
```

### JSONP

由于浏览器存在“同域限制”，ajax 方法只能向当前网页所在的域名发出 HTTP 请求。但是，通过在当前网页中插入 script 元素（），可以向不同的域名发出 GET 请求，这种变通方法叫做 JSONP（JSON with Padding）。

ajax 方法可以发出 JSONP 请求，方法是在对象参数中指定 dataType 为 JSONP。

```
$.ajax({
  url: '/data/search.jsonp',
  data: {q: 'a'},
  dataType: 'jsonp',
  success: function(resp) {
    $('#target').html('Results: ' + resp.results.length);
  }
});)
```

JSONP 的通常做法是，在所要请求的 URL 后面加在回调函数的名称。ajax 方法规定，如果所请求的网址以类似“callback=?”的形式结尾，则自动采用 JSONP 形式。所以，上面的代码还可以写成下面这样。

```
$.getJSON('/data/search.jsonp?q=a&callback=?',
  function(resp) {
    $('#target').html('Results: ' + resp.results.length);
  }
);
```

### 文件上传

假定网页有一个文件控件。

```
<input type="file" id="test-input">
```

下面就是如何使用 Ajax 上传文件。

```
var file = $('#test-input')[0].files[0];
var formData = new FormData();

formData.append('file', file);

$.ajax('myserver/uploads', {
  method: 'POST',
  contentType: false,
  processData: false,
  data: formData
});
```

上面代码是将文件作为表单数据发送。除此之外，也可以直接发送文件。

```
var file = $('#test-input')[0].files[0];

$.ajax('myserver/uploads', {
  method: 'POST',
  contentType: file.type,
  processData: false,
  data: file
});
```

## 参考链接

*   David Walsh, [Loading Scripts with jQuery](http://davidwalsh.name/loading-scripts-jquery)
*   Nguyen Huu Phuoc, [Best jQuery practices](http://programer.tips/2014/09/best-jquery-practices.html)