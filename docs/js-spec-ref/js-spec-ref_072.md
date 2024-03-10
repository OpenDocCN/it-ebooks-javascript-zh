# 8.6 Browserify：浏览器加载 Node.js 模块

*   基本用法
*   管理前端模块
*   生成前端模块
*   脚本文件的实时生成
*   browserify-middleware 模块
*   参考链接

随着 JavaScript 程序逐渐模块化，在 ECMAScript 6 推出官方的模块处理方案之前，有两种方案在实践中广泛采用：一种是 AMD 模块规范，针对模块的异步加载，主要用于浏览器端；另一种是 CommonJS 规范，针对模块的同步加载，主要用于服务器端，即 node.js 环境。

Browserify 是一个 node.js 模块，主要用于改写现有的 CommonJS 模块，使得浏览器端也可以使用这些模块。使用下面的命令，在全局环境下安装 Browserify。

```
$ npm install -g browserify
```

## 基本用法

先看一个例子。假定有一个很简单的 CommonJS 模块文件 foo.js。

```
// foo.js

module.exports = function(x) {
  console.log(x);
};
```

然后，还有一个 main.js 文件，用来加载 foo 模块。

```
// main.js

var foo = require("./foo");
foo("Hi");
```

使用 Browserify，将 main.js 转化为浏览器可以加载的脚本 compiled.js。

```
browserify main.js > compiled.js

# 或者

browserify main > compiled.js

# 或者

browserify main.js -o compiled.js
```

之所以转化后的文件叫做 compiled.js，是因为该文件不仅包括了 main.js，还包括了它所依赖的 foo.js。两者打包在一起，保证浏览器加载时的依赖关系。

```
<script src="compiled.js"></script>
```

使用上面的命令，在浏览器中运行 compiled.js，控制台会显示 Hi。

我们再看一个在服务器端的 backbone 模块转为客户端 backbone 模块的例子。先安装 backbone 和它所依赖的 jQuery 模块。

```
npm install backbone jquery
```

然后，新建一个 main.js 文件。

```
// main.js

var Backbone = require('backbone');
var $ = Backbone.$ = require('jquery/dist/jquery')(window);

var AppView = Backbone.View.extend({
  render: function(){
    $('main').append('<h1>Browserify is a great tool.</h1>');
  }
});

var appView = new AppView();
appView.render();
```

接着，使用 browserify 将 main.js 转为 app.js。

```
browserify main.js -o app.js
```

app.js 就可以直接插入 HTML 网页了。

```
<script src="app.js"></script>
```

注意，只要插入 app.js 一个文件就可以了，完全不需要再加载 backbone.js 和 jQuery 了。

## 管理前端模块

Browserify 的主要作用是将 CommonJS 模块转为浏览器可以调用的格式，但是纯粹的前端模块，也可以用它打包。

首先，新建一个项目目录，添加 package.json 文件。

```
{
  "name": "demo",
  "version": "1.0.0"
}
```

接着，新建 index.html。

```
<!doctype html>
<html>
<head>
  <title>npm and jQuery demo</title>
</head>
<body>
  <span class="title-tipso tipso_style" title="This is a loaded TIPSO!">
    Roll over to see the tip
  </span>
  <script src="./bundle.js">
</body>
</html>
```

上面代码中的 bundle.js，就是 Browserify 打包后将生成的文件。

然后，安装 jquery 和它的插件。

```
$ npm install --save jquery tipso
```

接着，新建一个文件 entry.js。

```
global.jQuery = require('jquery');
require('tipso');

jQuery(function(){
  jQuery('.title-tipso').tipso();
});
```

上面的文件中，第一行之所以要把 jQuery 写成 global 的属性，是为了转码之后，它可以变成一个全局变量。

最后，Browserify 打包。

```
$ browserify entry.js --debug > bundle.jsOA
```

上面代码中，--debug 参数表示在打包后的文件中加入 source map 以便除错。

这时，浏览器打开 index.html，脚本已经可以运行。如果不希望将 jQuery 一起打包，而是通过 CDN 加载，可以使用 browserify-shim 模块。

另外一个问题是，某些 jQuery 插件还有自带的 CSS 文件，这时可以安装 parcelify 模块。

```
$ npm install -g parcelify
```

然后，在 package.json 中写入规则，声明 CSS 文件的位置。

```
"style": [
  "./node_modules/tipso/src/tipso.css"
]
```

接着，运行 parcelify 进行 CSS 打包。

```
$ parcelify entry.js -c bundle.css
```

最后，将打包后的 CSS 文件插入 index.html。

```
<link rel="stylesheet" href="bundle.css" />
```

## 生成前端模块

有时，我们只是希望将 node.js 的模块，移植到浏览器，使得浏览器端可以调用。这时，可以采用 browserify 的-r 参数（--require 的简写）。

```
browserify -r through -r ./my-file.js:my-module > bundle.js
```

上面代码将 through 和 my-file.js（后面的冒号表示指定模块名为 my-module）都做成了模块，可以在其他 script 标签中调用。

```
<script src="bundle.js"></script>
<script>
  var through = require('through');
  var myModule = require('my-module');
  /* ... */
</script>
```

可以看到，-r 参数的另一个作用，就是为浏览器端提供 require 方法。

## 脚本文件的实时生成

Browserify 还可以实时生成脚本文件。

下面是一个服务器端脚本，启动 Web 服务器之后，外部用户每次访问这个脚本，它的内容是实时生成的。

```
var browserify = require('browserify');
var http = require('http');

http.createServer(function (req, res) {
  if (req.url === '/bundle.js') {
    res.setHeader('content-type', 'application/javascript');
    var b = browserify(__dirname + '/main.js').bundle();
    b.on('error', console.error);
    b.pipe(res);
  }
  else res.writeHead(404, 'not found')
});
```

## browserify-middleware 模块

上面是将服务器端模块直接转为客户端脚本，然后在网页中调用这个转化后的脚本文件。还有一种思路是，在运行时动态转换模块，这就需要用到[browserify-middleware 模块](https://github.com/ForbesLindesay/browserify-middleware)。

比如，网页中需要加载 app.js，它是从 main.js 转化过来的。

```
<!-- index.html -->

<script src="app.js"></script>
```

你可以在服务器端静态生成一个 app.js 文件，也可以让它动态生成。这就需要用 browserify-middleware 模块，服务器端脚本要像下面这样写。

```
var browserify = require('browserify-middleware');
var express = require('express');
var app = express();

app.get('/app.js', browserify('./client/main.js'));

app.get('/', function(req, res){
  res.render('index.html');
});
```

## 参考链接

*   Jack Franklin, [Dependency Management with Browserify](http://javascriptplayground.com/blog/2013/09/browserify/)
*   Seth Vincent, [Using Browserify with Express](http://learnjs.io/blog/2013/12/22/express-and-browserify/)
*   Patrick Mulder, [Browserify - Unix in the browser](http://thinkingonthinking.com/unix-in-the-browser/)
*   Patrick Catanzariti, [Getting Started with Browserify](http://www.sitepoint.com/getting-started-browserify/)
*   Lin Clark, [Using jQuery plugins with npm](http://blog.npmjs.org/post/112064849860/using-jquery-plugins-with-npm)