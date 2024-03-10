# 13.17 Express 框架

*   概述
    *   搭建 HTTPs 服务器
    *   运行原理
        *   底层：http 模块
        *   对 http 模块的再包装
        *   什么是中间件
        *   use 方法
    *   Express 的方法
        *   all 方法和 HTTP 动词方法
        *   set 方法
        *   response 对象
        *   requst 对象
    *   项目开发实例
        *   编写启动脚本
        *   配置路由
        *   静态网页模板
    *   动态网页模板
        *   安装模板引擎
        *   新建数据脚本
        *   新建网页模板
        *   渲染模板
        *   指定静态文件目录
    *   ExpressJS 4.0 的 Router 用法
        *   基本用法
        *   router.route 方法
        *   router 中间件
        *   对路径参数的处理
        *   app.route
    *   上传文件
    *   参考链接

## 概述

Express 是目前最流行的基于 Node.js 的 Web 开发框架，提供各种模块，可以快速地搭建一个具有完整功能的网站。

Express 的上手非常简单，首先新建一个项目目录，假定叫做 hello-world。

```js
$ mkdir hello-world
```

进入该目录，新建一个 package.json 文件，内容如下。

```js
{
  "name": "hello-world",
  "description": "hello world test app",
  "version": "0.0.1",
  "private": true,
  "dependencies": {
    "express": "4.x"
  }
}
```

上面代码定义了项目的名称、描述、版本等，并且指定需要 4.0 版本以上的 Express。

然后，就可以安装了。

```js
$ npm install
```

安装了 Express 及其依赖的模块以后，在项目根目录下，新建一个启动文件，假定叫做 index.js。

```js
var express = require('express');
var app = express();

app.use(express.static(__dirname + '/public'));

app.listen(8080);
```

上面代码运行之后，访问`http://localhost:8080`，就会在浏览器中打开当前目录的 public 子目录。如果 public 目录之中有一个图片文件 my_image.png，那么可以用`http://localhost:8080/my_image.png`访问该文件。

你也可以在 index.js 之中，生成动态网页。

```js
// index.js
var express = require('express');
var app = express();
app.get('/', function (req, res) {
  res.send('Hello world!');
});
app.listen(3000);
```

然后，在命令行下运行下面的命令，就可以在浏览器中访问项目网站了。

```js
$ node index
```

默认情况下，网站运行在本机的 3000 端口，网页显示 Hello World。

index.js 中的`app.get`用于指定不同的访问路径所对应的回调函数，这叫做“路由”（routing）。上面代码只指定了根目录的回调函数，因此只有一个路由记录。实际应用中，可能有多个路由记录。

```js
// index.js
var express = require('express');
var app = express();

app.get('/', function (req, res) {
  res.send('Hello world!');
});
app.get('/customer', function(req, res){
  res.send('customer page');
});
app.get('/admin', function(req, res){
  res.send('admin page');
});

app.listen(3000);
```

这时，最好就把路由放到一个单独的文件中，比如新建一个 routes 子目录。

```js
// routes/index.js

module.exports = function (app) {
  app.get('/', function (req, res) {
    res.send('Hello world');
  });
  app.get('/customer', function(req, res){
    res.send('customer page');
  });
  app.get('/admin', function(req, res){
    res.send('admin page');
  });
};
```

然后，原来的 index.js 就变成下面这样。

```js
// index.js
var express = require('express');
var app = express();
var routes = require('./routes')(app);
app.listen(3000);
```

### 搭建 HTTPs 服务器

使用 Express 搭建 HTTPs 加密服务器，也很简单。

```js
var fs = require('fs');
var options = {
  key: fs.readFileSync('E:/ssl/myserver.key'),
  cert: fs.readFileSync('E:/ssl/myserver.crt'),
  passphrase: '1234'
};

var https = require('https');
var express = require('express');
var app = express();

app.get('/', function(req, res){
  res.send('Hello World Expressjs');
});

var server = https.createServer(options, app);
server.listen(8084);
console.log('Server is running on port 8084');
```

## 运行原理

### 底层：http 模块

Express 框架建立在 node.js 内置的 http 模块上。 http 模块生成服务器的原始代码如下。

```js
var http = require("http");

var app = http.createServer(function(request, response) {
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.end("Hello world!");
});

app.listen(3000, "localhost");
```

上面代码的关键是 http 模块的 createServer 方法，表示生成一个 HTTP 服务器实例。该方法接受一个回调函数，该回调函数的参数，分别为代表 HTTP 请求和 HTTP 回应的 request 对象和 response 对象。

### 对 http 模块的再包装

Express 框架的核心是对 http 模块的再包装。上面的代码用 Express 改写如下。

```js
var express = require('express');
var app = express();
app.get('/', function (req, res) {
    res.send('Hello world!');
});
app.listen(3000);

var express = require("express");
var http = require("http");

var app = express();

app.use(function(request, response) {
  response.writeHead(200, { "Content-Type": "text/plain" });
  response.end("Hello world!\n");
});

http.createServer(app).listen(1337);
```

比较两段代码，可以看到它们非常接近，唯一的差别是 createServer 方法的参数，从一个回调函数变成了一个 Epress 对象的实例。而这个实例使用了 use 方法，加载了与上一段代码相同的回调函数。

Express 框架等于在 http 模块之上，加了一个中间层，而 use 方法则相当于调用中间件。

### 什么是中间件

简单说，中间件（middleware）就是处理 HTTP 请求的函数，用来完成各种特定的任务，比如检查用户是否登录、分析数据、以及其他在需要最终将数据发送给用户之前完成的任务。它最大的特点就是，一个中间件处理完，再传递给下一个中间件。

node.js 的内置模块 http 的 createServer 方法，可以生成一个服务器实例，该实例允许在运行过程中，调用一系列函数（也就是中间件）。当一个 HTTP 请求进入服务器，服务器实例会调用第一个中间件，完成后根据设置，决定是否再调用下一个中间件。中间件内部可以使用服务器实例的 response 对象（ServerResponse，即回调函数的第二个参数），以及一个 next 回调函数（即第三个参数）。每个中间件都可以对 HTTP 请求（request 对象）做出回应，并且决定是否调用 next 方法，将 request 对象再传给下一个中间件。

一个不进行任何操作、只传递 request 对象的中间件，大概是下面这样：

```js
function uselessMiddleware(req, res, next) { 
    next();
}
```

上面代码的 next 为中间件的回调函数。如果它带有参数，则代表抛出一个错误，参数为错误文本。

```js
function uselessMiddleware(req, res, next) {
  next('出错了！');
}
```

抛出错误以后，后面的中间件将不再执行，直到发现一个错误处理函数为止。

### use 方法

use 是 express 调用中间件的方法，它返回一个函数。下面是一个连续调用两个中间件的例子。

```js
var express = require("express");
var http = require("http");

var app = express();

app.use(function(request, response, next) {
  console.log("In comes a " + request.method + " to " + request.url);
  next();
});

app.use(function(request, response) {
  response.writeHead(200, { "Content-Type": "text/plain" });
  response.end("Hello world!\n");
});

http.createServer(app).listen(1337);
```

上面代码先调用第一个中间件，在控制台输出一行信息，然后通过 next 方法，调用第二个中间件，输出 HTTP 回应。由于第二个中间件没有调用 next 方法，所以不再 request 对象就不再向后传递了。

使用 use 方法，可以根据请求的网址，返回不同的网页内容。

```js
var express = require("express");
var http = require("http");

var app = express();

app.use(function(request, response, next) {
  if (request.url == "/") {
    response.writeHead(200, { "Content-Type": "text/plain" });
    response.end("Welcome to the homepage!\n");
  } else {
    next();
  }
});

app.use(function(request, response, next) {
  if (request.url == "/about") {
    response.writeHead(200, { "Content-Type": "text/plain" });
  } else {
    next();
  }
});

app.use(function(request, response) {
  response.writeHead(404, { "Content-Type": "text/plain" });
  response.end("404 error!\n");
});

http.createServer(app).listen(1337);
```

上面代码通过 request.url 属性，判断请求的网址，从而返回不同的内容。

除了在回调函数内部，判断请求的网址，Express 也允许将请求的网址写在 use 方法的第一个参数。

```js
app.use('/', someMiddleware);
```

上面代码表示，只对根目录的请求，调用某个中间件。

因此，上面的代码可以写成下面的样子。

```js
var express = require("express");
var http = require("http");

var app = express();

app.use("/", function(request, response, next) {
    response.writeHead(200, { "Content-Type": "text/plain" });
    response.end("Welcome to the homepage!\n");
});

app.use("/about", function(request, response, next) {
    response.writeHead(200, { "Content-Type": "text/plain" });
    response.end("Welcome to the about page!\n");
});

app.use(function(request, response) {
  response.writeHead(404, { "Content-Type": "text/plain" });
  response.end("404 error!\n");
});

http.createServer(app).listen(1337);
```

## Express 的方法

### all 方法和 HTTP 动词方法

针对不同的请求，Express 提供了 use 方法的一些别名。比如，上面代码也可以用别名的形式来写。

```js
var express = require("express");
var http = require("http");
var app = express();

app.all("*", function(request, response, next) {
  response.writeHead(200, { "Content-Type": "text/plain" });
  next();
});

app.get("/", function(request, response) {
  response.end("Welcome to the homepage!");
});

app.get("/about", function(request, response) {
  response.end("Welcome to the about page!");
});

app.get("*", function(request, response) {
  response.end("404!");
});

http.createServer(app).listen(1337);
```

上面代码的 all 方法表示，所有请求都必须通过该中间件，参数中的“*”表示对所有路径有效。get 方法则是只有 GET 动词的 HTTP 请求通过该中间件，它的第一个参数是请求的路径。由于 get 方法的回调函数没有调用 next 方法，所以只要有一个中间件被调用了，后面的中间件就不会再被调用了。

除了 get 方法以外，Express 还提供 post、put、delete 方法，即 HTTP 动词都是 Express 的方法。

这些方法的第一个参数，都是请求的路径。除了绝对匹配以外，Express 允许模式匹配。

```js
app.get("/hello/:who", function(req, res) {
  res.end("Hello, " + req.params.who + ".");
});
```

上面代码将匹配“/hello/alice”网址，网址中的 alice 将被捕获，作为 req.params.who 属性的值。需要注意的是，捕获后需要对网址进行检查，过滤不安全字符，上面的写法只是为了演示，生产中不应这样直接使用用户提供的值。

如果在模式参数后面加上问号，表示该参数可选。

```js
app.get('/hello/:who?',function(req,res) {
    if(req.params.id) {
        res.end("Hello, " + req.params.who + ".");
    }
    else {
        res.send("Hello, Guest.");
    }
});
```

下面是一些更复杂的模式匹配的例子。

```js
app.get('/forum/:fid/thread/:tid', middleware)

// 匹配/commits/71dbb9c
// 或/commits/71dbb9c..4c084f9 这样的 git 格式的网址
app.get(/^\/commits\/(\w+)(?:\.\.(\w+))?$/, function(req, res){
  var from = req.params[0];
  var to = req.params[1] || 'HEAD';
  res.send('commit range ' + from + '..' + to);
});
```

### set 方法

set 方法用于指定变量的值。

```js
app.set("views", __dirname + "/views");

app.set("view engine", "jade");
```

上面代码使用 set 方法，为系统变量“views”和“view engine”指定值。

### response 对象

（1）response.redirect 方法

response.redirect 方法允许网址的重定向。

```js
response.redirect("/hello/anime");
response.redirect("http://www.example.com");
response.redirect(301, "http://www.example.com");
```

（2）response.sendFile 方法

response.sendFile 方法用于发送文件。

```js
response.sendFile("/path/to/anime.mp4");
```

（3）response.render 方法

response.render 方法用于渲染网页模板。

```js
app.get("/", function(request, response) {
  response.render("index", { message: "Hello World" });
});
```

上面代码使用 render 方法，将 message 变量传入 index 模板，渲染成 HTML 网页。

### requst 对象

（1）request.ip

request.ip 属性用于获得 HTTP 请求的 IP 地址。

（2）request.files

request.files 用于获取上传的文件。

## 项目开发实例

### 编写启动脚本

上一节使用 express 命令自动建立项目，也可以不使用这个命令，手动新建所有文件。

先建立一个项目目录（假定这个目录叫做 demo）。进入该目录，新建一个 package.json 文件，写入项目的配置信息。

```js
{
   "name": "demo",
   "description": "My First Express App",
   "version": "0.0.1",
   "dependencies": {
      "express": "3.x"
   }
}
```

在项目目录中，新建文件 app.js。项目的代码就放在这个文件里面。

```js
var express = require('express');
var app = express();
```

上面代码首先加载 express 模块，赋给变量 express。然后，生成 express 实例，赋给变量 app。

接着，设定 express 实例的参数。

```js
// 设定 port 变量，意为访问端口
app.set('port', process.env.PORT || 3000);

// 设定 views 变量，意为视图存放的目录
app.set('views', path.join(__dirname, 'views'));

// 设定 view engine 变量，意为网页模板引擎
app.set('view engine', 'jade');

app.use(express.favicon());
app.use(express.logger('dev'));
app.use(express.bodyParser());
app.use(express.methodOverride());
app.use(app.router);

// 设定静态文件目录，比如本地文件
// 目录为 demo/public/images，访问
// 网址则显示为 http://localhost:3000/images
app.use(express.static(path.join(__dirname, 'public')));
```

上面代码中的 set 方法用于设定内部变量，use 方法用于调用 express 的中间件。

最后，调用实例方法 listen，让其监听事先设定的端口（3000）。

```js
app.listen(app.get('port'));
```

这时，运行下面的命令，就可以在浏览器访问[`127.0.0.1:3000`](http://127.0.0.1:3000)。

```js
node app.js
```

网页提示“Cannot GET /”，表示没有为网站的根路径指定可以显示的内容。所以，下一步就是配置路由。

### 配置路由

所谓“路由”，就是指为不同的访问路径，指定不同的处理方法。

（1）指定根路径

在 app.js 之中，先指定根路径的处理方法。

```js
app.get('/', function(req, res) {
   res.send('Hello World');
});
```

上面代码的 get 方法，表示处理客户端发出的 GET 请求。相应的，还有 app.post、app.put、app.del（delete 是 JavaScript 保留字，所以改叫 del）方法。

get 方法的第一个参数是访问路径，正斜杠（/）就代表根路径；第二个参数是回调函数，它的 req 参数表示客户端发来的 HTTP 请求，res 参数代表发向客户端的 HTTP 回应，这两个参数都是对象。在回调函数内部，使用 HTTP 回应的 send 方法，表示向浏览器发送一个字符串。然后，运行下面的命令。

```js
node app.js
```

此时，在浏览器中访问[`127.0.0.1:3000，网页就会显示“Hello`](http://127.0.0.1:3000) World”。

如果需要指定 HTTP 头信息，回调函数就必须换一种写法，要使用 setHeader 方法与 end 方法。

```js
app.get('/', function(req, res){
  var body = 'Hello World';
  res.setHeader('Content-Type', 'text/plain');
  res.setHeader('Content-Length', body.length);
  res.end(body);
});
```

（2）指定特定路径

上面是处理根目录的情况，下面再举一个例子。假定用户访问/api 路径，希望返回一个 JSON 字符串。这时，get 可以这样写。

```js
app.get('/api', function(request, response) {
   response.send({name:"张三",age:40});
});
```

上面代码表示，除了发送字符串，send 方法还可以直接发送对象。重新启动 node 以后，再访问路径/api，浏览器就会显示一个 JSON 对象。

```js
{
  "name": "张三",
  "age": 40
}
```

我们也可以把 app.get 的回调函数，封装成模块。先在 routes 目录下面建立一个 api.js 文件。

```js
// routes/api.js

exports.index = function (req, res){
  res.json(200, {name:"张三",age:40});
}
```

然后，在 app.js 中加载这个模块。

```js
// app.js

var api = require('./routes/api');
app.get('/api', api.index);
```

现在访问时，就会显示与上一次同样的结果。

如果只向浏览器发送简单的文本信息，上面的方法已经够用；但是如果要向浏览器发送复杂的内容，还是应该使用网页模板。

### 静态网页模板

在项目目录之中，建立一个子目录 views，用于存放网页模板。

假定这个项目有三个路径：根路径（/）、自我介绍（/about）和文章（/article）。那么，app.js 可以这样写：

```js
var express = require('express');
var app = express();

app.get('/', function(req, res) {
   res.sendfile('./views/index.html');
});

app.get('/about', function(req, res) {
   res.sendfile('./views/about.html');
});

app.get('/article', function(req, res) {
   res.sendfile('./views/article.html');
});

app.listen(3000);
```

上面代码表示，三个路径分别对应 views 目录中的三个模板：index.html、about.html 和 article.html。另外，向服务器发送信息的方法，从 send 变成了 sendfile，后者专门用于发送文件。

假定 index.html 的内容如下：

```js
<html>
<head>
   <title>首页</title>
</head>

<body>
<h1>Express Demo</h1>

<footer>
<p>
   <a href="/">首页</a> - <a href="/about">自我介绍</a> - <a href="/article">文章</a>
</p>
</footer>

</body>
</html>
```

上面代码是一个静态网页。如果想要展示动态内容，就必须使用动态网页模板。

## 动态网页模板

网站真正的魅力在于动态网页，下面我们来看看，如何制作一个动态网页的网站。

### 安装模板引擎

Express 支持多种模板引擎，这里采用 Handlebars 模板引擎的服务器端版本[hbs](https://github.com/donpark/hbs)模板引擎。

先安装 hbs。

```js
npm install hbs --save-dev
```

上面代码将 hbs 模块，安装在项目目录的子目录 node_modules 之中。save-dev 参数表示，将依赖关系写入 package.json 文件。安装以后的 package.json 文件变成下面这样：

```js
// package.json 文件

{
  "name": "demo",
  "description": "My First Express App",
  "version": "0.0.1",
  "dependencies": {
    "express": "3.x"
  },
  "devDependencies": {
    "hbs": "~2.3.1"
  }
}
```

安装模板引擎之后，就要改写 app.js。

```js
// app.js 文件

var express = require('express');
var app = express();

// 加载 hbs 模块
var hbs = require('hbs');

// 指定模板文件的后缀名为 html
app.set('view engine', 'html');

// 运行 hbs 模块
app.engine('html', hbs.__express);

app.get('/', function (req, res){
    res.render('index');
});

app.get('/about', function(req, res) {
    res.render('about');
});

app.get('/article', function(req, res) {
    res.render('article');
});
```

上面代码改用 render 方法，对网页模板进行渲染。render 方法的参数就是模板的文件名，默认放在子目录 views 之中，后缀名已经在前面指定为 html，这里可以省略。所以，res.render('index') 就是指，把子目录 views 下面的 index.html 文件，交给模板引擎 hbs 渲染。

### 新建数据脚本

渲染是指将数据代入模板的过程。实际运用中，数据都是保存在数据库之中的，这里为了简化问题，假定数据保存在一个脚本文件中。

在项目目录中，新建一个文件 blog.js，用于存放数据。blog.js 的写法符合 CommonJS 规范，使得它可以被 require 语句加载。

```js
// blog.js 文件

var entries = [
    {"id":1, "title":"第一篇", "body":"正文", "published":"6/2/2013"},
    {"id":2, "title":"第二篇", "body":"正文", "published":"6/3/2013"},
    {"id":3, "title":"第三篇", "body":"正文", "published":"6/4/2013"},
    {"id":4, "title":"第四篇", "body":"正文", "published":"6/5/2013"},
    {"id":5, "title":"第五篇", "body":"正文", "published":"6/10/2013"},
    {"id":6, "title":"第六篇", "body":"正文", "published":"6/12/2013"}
];

exports.getBlogEntries = function (){
   return entries;
}

exports.getBlogEntry = function (id){
   for(var i=0; i < entries.length; i++){
      if(entries[i].id == id) return entries[i];
   }
}
```

### 新建网页模板

接着，新建模板文件 index.html。

```js
<!-- views/index.html 文件 -->

<h1>文章列表</h1>

{{#each entries}}
   <p>
      <a href="/article/{{id}}">{{title}}</a><br/>
      Published: {{published}}
   </p>
{{/each}}
```

模板文件 about.html。

```js
<!-- views/about.html 文件 -->

<h1>自我介绍</h1>

<p>正文</p>
```

模板文件 article.html。

```js
<!-- views/article.html 文件 -->

<h1>{{blog.title}}</h1>
Published: {{blog.published}}

<p/>

{{blog.body}}
```

可以看到，上面三个模板文件都只有网页主体。因为网页布局是共享的，所以布局的部分可以单独新建一个文件 layout.html。

```js
<!-- views/layout.html 文件 -->

<html>

<head>
   <title>{{title}}</title>
</head>

<body>

    {{{body}}}

   <footer>
      <p>
         <a href="/">首页</a> - <a href="/about">自我介绍</a>
      </p>
   </footer>

</body>
</html>
```

### 渲染模板

最后，改写 app.js 文件。

```js
// app.js 文件

var express = require('express');
var app = express();

var hbs = require('hbs');

// 加载数据模块
var blogEngine = require('./blog');

app.set('view engine', 'html');
app.engine('html', hbs.__express);
app.use(express.bodyParser());

app.get('/', function(req, res) {
   res.render('index',{title:"最近文章", entries:blogEngine.getBlogEntries()});
});

app.get('/about', function(req, res) {
   res.render('about', {title:"自我介绍"});
});

app.get('/article/:id', function(req, res) {
   var entry = blogEngine.getBlogEntry(req.params.id);
   res.render('article',{title:entry.title, blog:entry});
});

app.listen(3000);
```

上面代码中的 render 方法，现在加入了第二个参数，表示模板变量绑定的数据。

现在重启 node 服务器，然后访问[`127.0.0.1:3000`](http://127.0.0.1:3000)。

```js
node app.js
```

可以看得，模板已经使用加载的数据渲染成功了。

### 指定静态文件目录

模板文件默认存放在 views 子目录。这时，如果要在网页中加载静态文件（比如样式表、图片等），就需要另外指定一个存放静态文件的目录。

```js
app.use(express.static('public'));
```

上面代码在文件 app.js 之中，指定静态文件存放的目录是 public。于是，当浏览器发出非 HTML 文件请求时，服务器端就到 public 目录寻找这个文件。比如，浏览器发出如下的样式表请求：

```js
<link href="/bootstrap/css/bootstrap.css" rel="stylesheet">
```

服务器端就到 public/bootstrap/css/目录中寻找 bootstrap.css 文件。

## ExpressJS 4.0 的 Router 用法

Express 4.0 的 Router 用法，做了大幅改变，增加了很多新的功能。Router 成了一个单独的组件，好像小型的 express 应用程序一样，有自己的 use、get、param 和 route 方法。

### 基本用法

Express 4.0 的 router 对象，需要单独新建。然后，使用该对象的 HTTP 动词方法，为不同的访问路径，指定回调函数；最后，挂载到某个路径

```js
var router = express.Router();

router.get('/', function(req, res) {
    res.send('首页');    
});

router.get('/about', function(req, res) {
    res.send('关于');    
});

app.use('/', router);
```

上面代码先定义了两个访问路径，然后将它们挂载到根目录。如果最后一行改为 app.use('/app', router)，则相当于/app 和/app/about 这两个路径，指定了回调函数。

这种挂载路径和 router 对象分离的做法，为程序带来了更大的灵活性，既可以定义多个 router 对象，也可以为将同一个 router 对象挂载到多个路径。

### router.route 方法

router 实例对象的 route 方法，可以接受访问路径作为参数。

```js
var router = express.Router();

router.route('/api')
    .post(function(req, res) {
        // ...
    })
    .get(function(req, res) {
        Bear.find(function(err, bears) {
            if (err) res.send(err);
            res.json(bears);
        });
    });

app.use('/', router);
```

### router 中间件

use 方法为 router 对象指定中间件，即在数据正式发给用户之前，对数据进行处理。下面就是一个中间件的例子。

```js
router.use(function(req, res, next) {
    console.log(req.method, req.url);
    next();  
});
```

上面代码中，回调函数的 next 参数，表示接受其他中间件的调用。函数体中的 next()，表示将数据传递给下一个中间件。

注意，中间件的放置顺序很重要，等同于执行顺序。而且，中间件必须放在 HTTP 动词方法之前，否则不会执行。

### 对路径参数的处理

router 对象的 param 方法用于路径参数的处理，可以

```js
router.param('name', function(req, res, next, name) {
    // 对 name 进行验证或其他处理……
    console.log(name);
    req.name = name;
    next();  
});

router.get('/hello/:name', function(req, res) {
    res.send('hello ' + req.name + '!');
});
```

上面代码中，get 方法为访问路径指定了 name 参数，param 方法则是对 name 参数进行处理。注意，param 方法必须放在 HTTP 动词方法之前。

### app.route

假定 app 是 Express 的实例对象，Express 4.0 为该对象提供了一个 route 属性。app.route 实际上是 express.Router()的缩写形式，除了直接挂载到根路径。因此，对同一个路径指定 get 和 post 方法的回调函数，可以写成链式形式。

```js
app.route('/login')
    .get(function(req, res) {
        res.send('this is the login form');
    })
    .post(function(req, res) {
        console.log('processing');
        res.send('processing the login form!');
    });
```

上面代码的这种写法，显然非常简洁清晰。

## 上传文件

首先，在网页插入上传文件的表单。

```js
<form action="/pictures/upload" method="POST" enctype="multipart/form-data">
  Select an image to upload:
  <input type="file" name="image">
  <input type="submit" value="Upload Image">
</form>
```

然后，服务器脚本建立指向`/upload`目录的路由。这时可以安装 multer 模块，它提供了上传文件的许多功能。

```js
var express = require('express');
var router = express.Router();
var multer = require('multer');

var uploading = multer({
  dest: __dirname + '../public/uploads/',
  // 设定限制，每次最多上传 1 个文件，文件大小不超过 1MB
  limits: {fileSize: 1000000, files:1},
})

router.post('/upload', uploading, function(req, res) {

})

module.exports = router
```

上面代码是上传文件到本地目录。下面是上传到 Amazon S3 的例子。

首先，在 S3 上面新增 CORS 配置文件。

```js
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration >
  <CORSRule>
    <AllowedOrigin>*</AllowedOrigin>
    <AllowedMethod>GET</AllowedMethod>
    <AllowedMethod>POST</AllowedMethod>
    <AllowedMethod>PUT</AllowedMethod>
    <AllowedHeader>*</AllowedHeader>
  </CORSRule>
</CORSConfiguration>
```

上面的配置允许任意电脑向你的 bucket 发送 HTTP 请求。

然后，安装 aws-sdk。

```js
$ npm install aws-sdk --save
```

下面是服务器脚本。

```js
var express = require('express');
var router = express.Router();
var aws = require('aws-sdk');

router.get('/', function(req, res) {
  res.render('index')
})

var AWS_ACCESS_KEY = 'your_AWS_access_key'
var AWS_SECRET_KEY = 'your_AWS_secret_key'
var S3_BUCKET = 'images_upload'

router.get('/sign', function(req, res) {
  aws.config.update({accessKeyId: AWS_ACCESS_KEY, secretAccessKey: AWS_SECRET_KEY});

  var s3 = new aws.S3()
  var options = {
    Bucket: S3_BUCKET,
    Key: req.query.file_name,
    Expires: 60,
    ContentType: req.query.file_type,
    ACL: 'public-read'
  }

  s3.getSignedUrl('putObject', options, function(err, data){
    if(err) return res.send('Error with S3')

    res.json({
      signed_request: data,
      url: 'https://s3.amazonaws.com/' + S3_BUCKET + '/' + req.query.file_name
    })
  })
})

module.exports = router
```

上面代码中，用户访问`/sign`路径，正确登录后，会收到一个 JSON 对象，里面是 S3 返回的数据和一个暂时用来接收上传文件的 URL，有效期只有 60 秒。

浏览器代码如下。

```js
// HTML 代码为
// <br>Please select an image
// <input type="file" id="image">
// <br>
// <img id="preview">

document.getElementById("image").onchange = function() {
  var file = document.getElementById("image").files[0]
  if (!file) return

  sign_request(file, function(response) {
    upload(file, response.signed_request, response.url, function() {
      document.getElementById("preview").src = response.url
    })
  })
}

function sign_request(file, done) {
  var xhr = new XMLHttpRequest()
  xhr.open("GET", "/sign?file_name=" + file.name + "&file_type=" + file.type)

  xhr.onreadystatechange = function() {
    if(xhr.readyState === 4 && xhr.status === 200) {
      var response = JSON.parse(xhr.responseText)
      done(response)
    }
  }
  xhr.send()
}

function upload(file, signed_request, url, done) {
  var xhr = new XMLHttpRequest()
  xhr.open("PUT", signed_request)
  xhr.setRequestHeader('x-amz-acl', 'public-read')
  xhr.onload = function() {
    if (xhr.status === 200) {
      done()
    }
  }

  xhr.send(file)
}
```

上面代码首先监听 file 控件的 change 事件，一旦有变化，就先向服务器要求一个临时的上传 URL，然后向该 URL 上传文件。

## 参考链接

*   Raymond Camden, [Introduction to Express](http://net.tutsplus.com/tutorials/javascript-ajax/introduction-to-express/)
*   Christopher Buecheler, [Getting Started With Node.js, Express, MongoDB](http://cwbuecheler.com/web/tutorials/2013/node-express-mongo/)
*   Stephen Sugden, [A short guide to Connect Middleware](http://stephensugden.com/middleware_guide/)
*   Evan Hahn, [Understanding Express.js](http://evanhahn.com/understanding-express/)
*   Chris Sevilleja, [Learn to Use the New Router in ExpressJS 4.0](http://scotch.io/tutorials/javascript/learn-to-use-the-new-router-in-expressjs-4)
*   Stefan Fidanov, [Limitless file uploading to Amazon S3 with Node & Express](http://www.terlici.com/2015/05/23/uploading-files-S3.html)