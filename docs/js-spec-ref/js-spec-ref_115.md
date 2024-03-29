# 13.18 Koa 框架

Koa 是一个类似于 Express 的 Web 开发框架，开发人员也是同一组人，但是使用了 Generator 函数，进行了架构的重新设计。也就是说，Koa 的原理和内部结构很像 Express，但是语法和内部结构进行了升级。

官方[faq](https://github.com/koajs/koa/blob/master/docs/faq.md#why-isnt-koa-just-express-40)有这样一个问题：”为什么 koa 不是 Express 4.0？“，回答是这样的：”Koa 与 Express 有很大差异，整个设计都是不同的，所以如果将 Express 3.0 按照这种写法升级到 4.0，就意味着重写整个程序。所以，我们觉得创造一个新的库，是更合适的做法。“

*   Koa 应用
*   中间件
    *   多个中间件的合并
    *   路由
    *   context 对象
    *   错误处理机制
    *   cookie
    *   session
    *   Request 对象
    *   Response 对象
    *   CSRF 攻击
    *   数据压缩
    *   源码解读
    *   参考链接

## Koa 应用

一个 Koa 应用就是一个对象，包含了一个 middleware 数组，这个数组由一组 Generator 函数组成。这些函数负责对 HTTP 请求进行各种加工，比如生成缓存、指定代理、请求重定向等等。

```js
var koa = require('koa');
var app = koa();

app.use(function *(){
  this.body = 'Hello World';
});

app.listen(3000);
```

上面代码中，变量 app 就是一个 Koa 应用。它监听 3000 端口，返回一个内容为 Hello World 的网页。

app.use 方法用于向 middleware 数组添加 Generator 函数。

listen 方法指定监听端口，并启动当前应用。它实际上等同于下面的代码。

```js
var http = require('http');
var koa = require('koa');
var app = koa();
http.createServer(app.callback()).listen(3000);
```

## 中间件

Koa 的中间件很像 Express 的中间件，也是对 HTTP 请求进行处理的函数，但是必须是一个 Generator 函数。而且，Koa 的中间件是一个级联式（Cascading）的结构，也就是说，属于是层层调用，第一个中间件调用第二个中间件，第二个调用第三个，以此类推。上游的中间件必须等到下游的中间件返回结果，才会继续执行，这点很像递归。

中间件通过当前应用的 use 方法注册。

```js
app.use(function* (next){
  var start = new Date; // （1）
  yield next;  // （2）
  var ms = new Date - start; // （3）
  console.log('%s %s - %s', this.method, this.url, ms); // （4）
});
```

上面代码中，`app.use`方法的参数就是中间件，它是一个 Generator 函数，最大的特征就是 function 命令与参数之间，必须有一个星号。Generator 函数的参数 next，表示下一个中间件。

Generator 函数内部使用 yield 命令，将程序的执行权转交给下一个中间件，即`yield next`，要等到下一个中间件返回结果，才会继续往下执行。上面代码中，Generator 函数体内部，第一行赋值语句首先执行，开始计时，第二行 yield 语句将执行权交给下一个中间件，当前中间件就暂停执行。等到后面的中间件全部执行完成，执行权就回到原来暂停的地方，继续往下执行，这时才会执行第三行，计算这个过程一共花了多少时间，第四行将这个时间打印出来。

下面是一个两个中间件级联的例子。

```js
app.use(function *() {
  this.body = "header\n";
  yield saveResults.call(this);
  this.body += "footer\n";
});

function *saveResults() {
  this.body += "Results Saved!\n";
}
```

上面代码中，第一个中间件调用第二个中间件 saveResults，它们都向`this.body`写入内容。最后，`this.body`的输出如下。

```js
header
Results Saved!
footer
```

只要有一个中间件缺少`yield next`语句，后面的中间件都不会执行，这一点要引起注意。

```js
app.use(function *(next){
  console.log('>> one');
  yield next;
  console.log('<< one');
});

app.use(function *(next){
  console.log('>> two');
  this.body = 'two';
  console.log('<< two');
});

app.use(function *(next){
  console.log('>> three');
  yield next;
  console.log('<< three');
});
```

上面代码中，因为第二个中间件少了`yield next`语句，第三个中间件并不会执行。

如果想跳过一个中间件，可以直接在该中间件的第一行语句写上`return yield next`。

```js
app.use(function* (next) {
  if (skip) return yield next;
})
```

由于 Koa 要求中间件唯一的参数就是 next，导致如果要传入其他参数，必须另外写一个返回 Generator 函数的函数。

```js
function logger(format) {
  return function *(next){
    var str = format
      .replace(':method', this.method)
      .replace(':url', this.url);

    console.log(str);

    yield next;
  }
}

app.use(logger(':method :url'));
```

上面代码中，真正的中间件是 logger 函数的返回值，而 logger 函数是可以接受参数的。

### 多个中间件的合并

由于中间件的参数统一为 next（意为下一个中间件），因此可以使用`.call(this, next)`，将多个中间件进行合并。

```js
function *random(next) {
  if ('/random' == this.path) {
    this.body = Math.floor(Math.random()*10);
  } else {
    yield next;
  }
};

function *backwards(next) {
  if ('/backwards' == this.path) {
    this.body = 'sdrawkcab';
  } else {
    yield next;
  }
}

function *pi(next) {
  if ('/pi' == this.path) {
    this.body = String(Math.PI);
  } else {
    yield next;
  }
}

function *all(next) {
  yield random.call(this, backwards.call(this, pi.call(this, next)));
}

app.use(all);
```

上面代码中，中间件 all 内部，就是依次调用 random、backwards、pi，后一个中间件就是前一个中间件的参数。

Koa 内部使用 koa-compose 模块，进行同样的操作，下面是它的源码。

```js
function compose(middleware){
  return function *(next){
    if (!next) next = noop();

    var i = middleware.length;

    while (i--) {
      next = middleware[i].call(this, next);
    }

    yield *next;
  }
}

function *noop(){}
```

上面代码中，middleware 是中间件数组。前一个中间件的参数是后一个中间件，依次类推。如果最后一个中间件没有 next 参数，则传入一个空函数。

## 路由

可以通过`this.path`属性，判断用户请求的路径，从而起到路由作用。

```js
app.use(function* (next) {
  if (this.path === '/') {
    this.body = 'we are at home!';
  }
})

// 等同于

app.use(function* (next) {
  if (this.path !== '/') return yield next;
  this.body = 'we are at home!';
})
```

下面是多路径的例子。

```js
let koa = require('koa')

let app = koa()

// normal route
app.use(function* (next) {
  if (this.path !== '/') {
    return yield next
  }

  this.body = 'hello world'
});

// /404 route
app.use(function* (next) {
  if (this.path !== '/404') {
    return yield next;
  }

  this.body = 'page not found'
});

// /500 route
app.use(function* (next) {
  if (this.path !== '/500') {
    return yield next;
  }

  this.body = 'internal server error'
});

app.listen(8080)
```

上面代码中，每一个中间件负责一个路径，如果路径不符合，就传递给下一个中间件。

复杂的路由需要安装 koa-router 插件。

```js
var app = require('koa')();
var Router = require('koa-router');

var myRouter = new Router();

myRouter.get('/', function *(next) {
  this.response.body = 'Hello World!';
});

app.use(myRouter.routes());

app.listen(3000);
```

上面代码对根路径设置路由。

Koa-router 实例提供一系列动词方法，即一种 HTTP 动词对应一种方法。典型的动词方法有以下五种。

*   router.get()
*   router.post()
*   router.put()
*   router.del()
*   router.patch()

这些动词方法可以接受两个参数，第一个是路径模式，第二个是对应的控制器方法（中间件），定义用户请求该路径时服务器行为。

```js
router.get('/', function *(next) {
  this.body = 'Hello World!';
});
```

上面代码中，`router.get`方法的第一个参数是根路径，第二个参数是对应的函数方法。

注意，路径匹配的时候，不会把查询字符串考虑在内。比如，`/index?param=xyz`匹配路径`/index`。

有些路径模式比较复杂，Koa-router 允许为路径模式起别名。起名时，别名要添加为动词方法的第一个参数，这时动词方法变成接受三个参数。

```js
router.get('user', '/users/:id', function *(next) {
 // ...
});
```

上面代码中，路径模式`\users\:id`的名字就是`user`。路径的名称，可以用来引用对应的具体路径，比如 url 方法可以根据路径名称，结合给定的参数，生成具体的路径。

```js
router.url('user', 3);
// => "/users/3"

router.url('user', { id: 3 });
// => "/users/3"
```

上面代码中，user 就是路径模式的名称，对应具体路径`/users/:id`。url 方法的第二个参数 3，表示给定 id 的值是 3，因此最后生成的路径是`/users/3`。

Koa-router 允许为路径统一添加前缀。

```js
var router = new Router({
  prefix: '/users'
});

router.get('/', ...); // 等同于"/users"
router.get('/:id', ...); // 等同于"/users/:id"
```

路径的参数通过`this.params`属性获取，该属性返回一个对象，所有路径参数都是该对象的成员。

```js
// 访问 /programming/how-to-node
router.get('/:category/:title', function *(next) {
  console.log(this.params);
  // => { category: 'programming', title: 'how-to-node' }
});
```

param 方法可以针对命名参数，设置验证条件。

```js
router
  .get('/users/:user', function *(next) {
    this.body = this.user;
  })
  .param('user', function *(id, next) {
    var users = [ '0 号用户', '1 号用户', '2 号用户'];
    this.user = users[id];
    if (!this.user) return this.status = 404;
    yield next;
  })
```

上面代码中，如果`/users/:user`的参数 user 对应的不是有效用户（比如访问`/users/3`），param 方法注册的中间件会查到，就会返回 404 错误。

redirect 方法会将某个路径的请求，重定向到另一个路径，并返回 301 状态码。

```js
router.redirect('/login', 'sign-in');

// 等同于
router.all('/login', function *() {
  this.redirect('/sign-in');
  this.status = 301;
});
```

redirect 方法的第一个参数是请求来源，第二个参数是目的地，两者都可以用路径模式的别名代替。

## context 对象

中间件当中的 this 表示上下文对象 context，代表一次 HTTP 请求和回应，即一次访问/回应的所有信息，都可以从上下文对象获得。context 对象封装了 request 和 response 对象，并且提供了一些辅助方法。每次 HTTP 请求，就会创建一个新的 context 对象。

```js
app.use(function *(){
  this; // is the Context
  this.request; // is a koa Request
  this.response; // is a koa Response
});
```

context 对象的很多方法，其实是定义在 ctx.request 对象或 ctx.response 对象上面，比如，ctx.type 和 ctx.length 对应于 ctx.response.type 和 ctx.response.length，ctx.path 和 ctx.method 对应于 ctx.request.path 和 ctx.request.method。

context 对象的全局属性。

*   request：指向 Request 对象
*   response：指向 Response 对象
*   req：指向 Node 的 request 对象
*   req：指向 Node 的 response 对象
*   app：指向 App 对象
*   state：用于在中间件传递信息。

```js
this.state.user = yield User.find(id);
```

上面代码中，user 属性存放在`this.state`对象上面，可以被另一个中间件读取。

context 对象的全局方法。

*   throw()：抛出错误，直接决定了 HTTP 回应的状态码。
*   assert()：如果一个表达式为 false，则抛出一个错误。

```js
this.throw(403);
this.throw('name required', 400);
this.throw('something exploded');

this.throw(400, 'name required');
// 等同于
var err = new Error('name required');
err.status = 400;
throw err;
```

assert 方法的例子。

```js
// 格式
ctx.assert(value, [msg], [status], [properties])

// 例子
this.assert(this.user, 401, 'User not found. Please login!');
```

以下模块解析 POST 请求的数据。

*   co-body
*   [`github.com/koajs/body-parser`](https://github.com/koajs/body-parser)
*   [`github.com/koajs/body-parsers`](https://github.com/koajs/body-parsers)

```js
var parse = require('co-body');

// in Koa handler
var body = yield parse(this);
```

## 错误处理机制

Koa 提供内置的错误处理机制，任何中间件抛出的错误都会被捕捉到，引发向客户端返回一个 500 错误，而不会导致进程停止，因此也就不需要 forever 这样的模块重启进程。

```js
app.use(function *() {
  throw new Error();
});
```

上面代码中，中间件内部抛出一个错误，并不会导致 Koa 应用挂掉。Koa 内置的错误处理机制，会捕捉到这个错误。

当然，也可以额外部署自己的错误处理机制。

```js
app.use(function *() {
  try {
    yield saveResults();
  } catch (err) {
    this.throw(400, '数据无效');
  }
});
```

上面代码自行部署了 try...catch 代码块，一旦产生错误，就用`this.throw`方法抛出。该方法可以将指定的状态码和错误信息，返回给客户端。

对于未捕获错误，可以设置 error 事件的监听函数。

```js
app.on('error', function(err){
  log.error('server error', err);
});
```

error 事件的监听函数还可以接受上下文对象，作为第二个参数。

```js
app.on('error', function(err, ctx){
  log.error('server error', err, ctx);
});
```

如果一个错误没有被捕获，koa 会向客户端返回一个 500 错误“Internal Server Error”。

this.throw 方法用于向客户端抛出一个错误。

```js
this.throw(403);
this.throw('name required', 400);
this.throw(400, 'name required');
this.throw('something exploded');

this.throw('name required', 400)
// 等同于
var err = new Error('name required');
err.status = 400;
throw err;
```

`this.throw`方法的两个参数，一个是错误码，另一个是报错信息。如果省略状态码，默认是 500 错误。

`this.assert`方法用于在中间件之中断言，用法类似于 Node 的 assert 模块。

```js
this.assert(this.user, 401, 'User not found. Please login!');
```

上面代码中，如果 this.user 属性不存在，会抛出一个 401 错误。

由于中间件是层级式调用，所以可以把`try { yield next }`当成第一个中间件。

```js
app.use(function *(next) {
  try {
    yield next;
  } catch (err) {
    this.status = err.status || 500;
    this.body = err.message;
    this.app.emit('error', err, this);
  }
});

app.use(function *(next) {
  throw new Error('some error');
})
```

## cookie

cookie 的读取和设置。

```js
this.cookies.get('view');
this.cookies.set('view', n);
```

get 和 set 方法都可以接受第三个参数，表示配置参数。其中的 signed 参数，用于指定 cookie 是否加密。如果指定加密的话，必须用`app.keys`指定加密短语。

```js
app.keys = ['secret1', 'secret2'];
this.cookies.set('name', '张三', { signed: true });
```

this.cookie 的配置对象的属性如下。

*   signed：cookie 是否加密。
*   expires：cookie 何时过期
*   path：cookie 的路径，默认是“/”。
*   domain：cookie 的域名。
*   secure：cookie 是否只有 https 请求下才发送。
*   httpOnly：是否只有服务器可以取到 cookie，默认为 true。

## session

```js
var session = require('koa-session');
var koa = require('koa');
var app = koa();

app.keys = ['some secret hurr'];
app.use(session(app));

app.use(function *(){
  var n = this.session.views || 0;
  this.session.views = ++n;
  this.body = n + ' views';
})

app.listen(3000);
console.log('listening on port 3000');
```

## Request 对象

Request 对象表示 HTTP 请求。

（1）this.request.header

返回一个对象，包含所有 HTTP 请求的头信息。它也可以写成`this.request.headers`。

（2）this.request.method

返回 HTTP 请求的方法，该属性可读写。

（3）this.request.length

返回 HTTP 请求的 Content-Length 属性，取不到值，则返回 undefined。

（4）this.request.path

返回 HTTP 请求的路径，该属性可读写。

（5）this.request.href

返回 HTTP 请求的完整路径，包括协议、端口和 url。

```js
this.request.href
// http://example.com/foo/bar?q=1
```

（6）this.request.querystring

返回 HTTP 请求的查询字符串，不含问号。该属性可读写。

（7）this.request.search

返回 HTTP 请求的查询字符串，含问号。该属性可读写。

（8）this.request.host

返回 HTTP 请求的主机（含端口号）。

（9）this.request.hostname

返回 HTTP 的主机名（不含端口号）。

（10）this.request.type

返回 HTTP 请求的 Content-Type 属性。

```js
var ct = this.request.type;
// "image/png"
```

（11）this.request.charset

返回 HTTP 请求的字符集。

```js
this.request.charset
// "utf-8"
```

（12）this.request.query

返回一个对象，包含了 HTTP 请求的查询字符串。如果没有查询字符串，则返回一个空对象。该属性可读写。

比如，查询字符串`color=blue&size=small`，会得到以下的对象。

```js
{
  color: 'blue',
  size: 'small'
}
```

（13）this.request.fresh

返回一个布尔值，表示缓存是否代表了最新内容。通常与 If-None-Match、ETag、If-Modified-Since、Last-Modified 等缓存头，配合使用。

```js
this.response.set('ETag', '123');

// 检查客户端请求的内容是否有变化
if (this.request.fresh) {
  this.response.status = 304;
  return;
}

// 否则就表示客户端的内容陈旧了，
// 需要取出新内容
this.response.body = yield db.find('something');
```

（14）this.request.stale

返回`this.request.fresh`的相反值。

（15）this.request.protocol

返回 HTTP 请求的协议，https 或者 http。

（16）this.request.secure

返回一个布尔值，表示当前协议是否为 https。

（17）this.request.ip

返回发出 HTTP 请求的 IP 地址。

（18）this.request.subdomains

返回一个数组，表示 HTTP 请求的子域名。该属性必须与 app.subdomainOffset 属性搭配使用。app.subdomainOffset 属性默认为 2，则域名“tobi.ferrets.example.com”返回["ferrets", "tobi"]，如果 app.subdomainOffset 设为 3，则返回["tobi"]。

（19）this.request.is(types...)

返回指定的类型字符串，表示 HTTP 请求的 Content-Type 属性是否为指定类型。

```js
// Content-Type 为 text/html; charset=utf-8
this.request.is('html'); // 'html'
this.request.is('text/html'); // 'text/html'
this.request.is('text/*', 'text/html'); // 'text/html'

// Content-Type 为 s application/json
this.request.is('json', 'urlencoded'); // 'json'
this.request.is('application/json'); // 'application/json'
this.request.is('html', 'application/*'); // 'application/json'
```

如果不满足条件，返回 false；如果 HTTP 请求不含数据，则返回 undefined。

```js
this.is('html'); // false
```

它可以用于过滤 HTTP 请求，比如只允许请求下载图片。

```js
if (this.is('image/*')) {
  // process
} else {
  this.throw(415, 'images only!');
}
```

（20）this.request.accepts(types)

检查 HTTP 请求的 Accept 属性是否可接受，如果可接受，则返回指定的媒体类型，否则返回 false。

```js
// Accept: text/html
this.request.accepts('html');
// "html"

// Accept: text/*, application/json
this.request.accepts('html');
// "html"
this.request.accepts('text/html');
// "text/html"
this.request.accepts('json', 'text');
// => "json"
this.request.accepts('application/json');
// => "application/json"

// Accept: text/*, application/json
this.request.accepts('image/png');
this.request.accepts('png');
// false

// Accept: text/*;q=.5, application/json
this.request.accepts(['html', 'json']);
this.request.accepts('html', 'json');
// "json"

// No Accept header
this.request.accepts('html', 'json');
// "html"
this.request.accepts('json', 'html');
// => "json"
```

如果 accepts 方法没有参数，则返回所有支持的类型（text/html,application/xhtml+xml,image/webp,application/xml,*/*）。

如果 accepts 方法的参数有多个参数，则返回最佳匹配。如果都不匹配则返回 false，并向客户端抛出一个 406”Not Acceptable“错误。

如果 HTTP 请求没有 Accept 字段，那么 accepts 方法返回它的第一个参数。

accepts 方法可以根据不同 Accept 字段，向客户端返回不同的字段。

```js
switch (this.request.accepts('json', 'html', 'text')) {
  case 'json': break;
  case 'html': break;
  case 'text': break;
  default: this.throw(406, 'json, html, or text only');
}
```

（21）this.request.acceptsEncodings(encodings)

该方法根据 HTTP 请求的 Accept-Encoding 字段，返回最佳匹配，如果没有合适的匹配，则返回 false。

```js
// Accept-Encoding: gzip
this.request.acceptsEncodings('gzip', 'deflate', 'identity');
// "gzip"
this.request.acceptsEncodings(['gzip', 'deflate', 'identity']);
// "gzip"
```

注意，acceptEncodings 方法的参数必须包括 identity（意为不编码）。

如果 HTTP 请求没有 Accept-Encoding 字段，acceptEncodings 方法返回所有可以提供的编码方法。

```js
// Accept-Encoding: gzip, deflate
this.request.acceptsEncodings();
// ["gzip", "deflate", "identity"]
```

如果都不匹配，acceptsEncodings 方法返回 false，并向客户端抛出一个 406“Not Acceptable”错误。

（22）this.request.acceptsCharsets(charsets)

该方法根据 HTTP 请求的 Accept-Charset 字段，返回最佳匹配，如果没有合适的匹配，则返回 false。

```js
// Accept-Charset: utf-8, iso-8859-1;q=0.2, utf-7;q=0.5
this.request.acceptsCharsets('utf-8', 'utf-7');
// => "utf-8"

this.request.acceptsCharsets(['utf-7', 'utf-8']);
// => "utf-8"
```

如果 acceptsCharsets 方法没有参数，则返回所有可接受的匹配。

```js
// Accept-Charset: utf-8, iso-8859-1;q=0.2, utf-7;q=0.5
this.request.acceptsCharsets();
// ["utf-8", "utf-7", "iso-8859-1"]
```

如果都不匹配，acceptsCharsets 方法返回 false，并向客户端抛出一个 406“Not Acceptable”错误。

（23）this.request.acceptsLanguages(langs)

该方法根据 HTTP 请求的 Accept-Language 字段，返回最佳匹配，如果没有合适的匹配，则返回 false。

```js
// Accept-Language: en;q=0.8, es, pt
this.request.acceptsLanguages('es', 'en');
// "es"
this.request.acceptsLanguages(['en', 'es']);
// "es"
```

如果 acceptsCharsets 方法没有参数，则返回所有可接受的匹配。

```js
// Accept-Language: en;q=0.8, es, pt
this.request.acceptsLanguages();
// ["es", "pt", "en"]
```

如果都不匹配，acceptsLanguages 方法返回 false，并向客户端抛出一个 406“Not Acceptable”错误。

（24）this.request.socket

返回 HTTP 请求的 socket。

（25）this.request.get(field)

返回 HTTP 请求指定的字段。

## Response 对象

Response 对象表示 HTTP 回应。

（1）this.response.header

返回 HTTP 回应的头信息。

（2）this.response.socket

返回 HTTP 回应的 socket。

（3）this.response.status

返回 HTTP 回应的状态码。默认情况下，该属性没有值。该属性可读写，设置时等于一个整数。

（4）this.response.message

返回 HTTP 回应的状态信息。该属性与`this.response.message`是配对的。该属性可读写。

（5）this.response.length

返回 HTTP 回应的 Content-Length 字段。该属性可读写，如果没有设置它的值，koa 会自动从 this.request.body 推断。

（6）this.response.body

返回 HTTP 回应的信息体。该属性可读写，它的值可能有以下几种类型。

*   字符串：Content-Type 字段默认为 text/html 或 text/plain，字符集默认为 utf-8，Content-Length 字段同时设定。
*   二进制 Buffer：Content-Type 字段默认为 application/octet-stream，Content-Length 字段同时设定。
*   Stream：Content-Type 字段默认为 application/octet-stream。
*   JSON 对象：Content-Type 字段默认为 application/json。
*   null（表示没有信息体）

如果`this.response.status`没设置，Koa 会自动将其设为 200 或 204。

（7）this.response.get(field)

返回 HTTP 回应的指定字段。

```js
var etag = this.get('ETag');
```

注意，get 方法的参数是区分大小写的。

（8）this.response.set()

设置 HTTP 回应的指定字段。

```js
this.set('Cache-Control', 'no-cache');
```

set 方法也可以接受一个对象作为参数，同时为多个字段指定值。

```js
this.set({
  'Etag': '1234',
  'Last-Modified': date
});
```

（9）this.response.remove(field)

移除 HTTP 回应的指定字段。

（10）this.response.type

返回 HTTP 回应的 Content-Type 字段，不包括“charset”参数的部分。

```js
var ct = this.reponse.type;
// "image/png"
```

该属性是可写的。

```js
this.reponse.type = 'text/plain; charset=utf-8';
this.reponse.type = 'image/png';
this.reponse.type = '.png';
this.reponse.type = 'png';
```

设置 type 属性的时候，如果没有提供 charset 参数，Koa 会判断是否自动设置。如果`this.response.type`设为 html，charset 默认设为 utf-8；但如果`this.response.type`设为 text/html，就不会提供 charset 的默认值。

（10）this.response.is(types...)

该方法类似于`this.request.is()`，用于检查 HTTP 回应的类型是否为支持的类型。

它可以在中间件中起到处理不同格式内容的作用。

```js
var minify = require('html-minifier');

app.use(function *minifyHTML(next){
  yield next;

  if (!this.response.is('html')) return;

  var body = this.response.body;
  if (!body || body.pipe) return;

  if (Buffer.isBuffer(body)) body = body.toString();
  this.response.body = minify(body);
});
```

上面代码是一个中间件，如果输出的内容类型为 HTML，就会进行最小化处理。

（11）this.response.redirect(url, [alt])

该方法执行 302 跳转到指定网址。

```js
this.redirect('back');
this.redirect('back', '/index.html');
this.redirect('/login');
this.redirect('http://google.com');
```

如果 redirect 方法的第一个参数是 back，将重定向到 HTTP 请求的 Referrer 字段指定的网址，如果没有该字段，则重定向到第二个参数或“/”网址。

如果想修改 302 状态码，或者修改 body 文字，可以采用下面的写法。

```js
this.status = 301;
this.redirect('/cart');
this.body = 'Redirecting to shopping cart';
```

（12）this.response.attachment([filename])

该方法将 HTTP 回应的 Content-Disposition 字段，设为“attachment”，提示浏览器下载指定文件。

（13）this.response.headerSent

该方法返回一个布尔值，检查是否 HTTP 回应已经发出。

（14）this.response.lastModified

该属性以 Date 对象的形式，返回 HTTP 回应的 Last-Modified 字段（如果该字段存在）。该属性可写。

```js
this.response.lastModified = new Date();
```

（15）this.response.etag

该属性设置 HTTP 回应的 ETag 字段。

```js
this.response.etag = crypto.createHash('md5').update(this.body).digest('hex');
```

注意，不能用该属性读取 ETag 字段。

（16）this.response.vary(field)

该方法将参数添加到 HTTP 回应的 Vary 字段。

## CSRF 攻击

CSRF 攻击是指用户的 session 被劫持，用来冒充用户的攻击。

koa-csrf 插件用来防止 CSRF 攻击。原理是在 session 之中写入一个秘密的 token，用户每次使用 POST 方法提交数据的时候，必须含有这个 token，否则就会抛出错误。

```js
var koa = require('koa');
var session = require('koa-session');
var csrf = require('koa-csrf');
var route = require('koa-route');

var app = module.exports = koa();

app.keys = ['session key', 'csrf example'];
app.use(session(app));

app.use(csrf());

app.use(route.get('/token', token));
app.use(route.post('/post', post));

function* token () {
  this.body = this.csrf;
}

function* post() {
  this.body = {ok: true};
}

app.listen(3000);
```

POST 请求含有 token，可以是以下几种方式之一，koa-csrf 插件就能获得 token。

*   表单的 _csrf 字段
*   查询字符串的 _csrf 字段
*   HTTP 请求头信息的 x-csrf-token 字段
*   HTTP 请求头信息的 x-xsrf-token 字段

## 数据压缩

koa-compress 模块可以实现数据压缩。

```js
app.use(require('koa-compress')())
app.use(function* () {
  this.type = 'text/plain'
  this.body = fs.createReadStream('filename.txt')
})
```

## 源码解读

每一个网站就是一个 app，它由`lib/application`定义。

```js
function Application() {
  if (!(this instanceof Application)) return new Application;
  this.env = process.env.NODE_ENV || 'development';
  this.subdomainOffset = 2;
  this.middleware = [];
  this.context = Object.create(context);
  this.request = Object.create(request);
  this.response = Object.create(response);
}

var app = Application.prototype;

exports = module.exports = Application;
```

`app.use()`用于注册中间件，即将 Generator 函数放入中间件数组。

```js
app.use = function(fn){
  if (!this.experimental) {
    // es7 async functions are allowed
    assert(fn && 'GeneratorFunction' == fn.constructor.name, 'app.use() requires a generator function');
  }
  debug('use %s', fn._name || fn.name || '-');
  this.middleware.push(fn);
  return this;
};
```

`app.listen()`就是`http.createServer(app.callback()).listen(...)`的缩写。

```js
app.listen = function(){
  debug('listen');
  var server = http.createServer(this.callback());
  return server.listen.apply(server, arguments);
};

app.callback = function(){
  var mw = [respond].concat(this.middleware);
  var fn = this.experimental
    ? compose_es7(mw)
    : co.wrap(compose(mw));
  var self = this;

  if (!this.listeners('error').length) this.on('error', this.onerror);

  return function(req, res){
    res.statusCode = 404;
    var ctx = self.createContext(req, res);
    onFinished(res, ctx.onerror);
    fn.call(ctx).catch(ctx.onerror);
  }
};
```

上面代码中，`app.callback()`会返回一个函数，用来处理 HTTP 请求。它的第一行`mw = [respond].concat(this.middleware)`，表示将 respond 函数（这也是一个 Generator 函数）放入`this.middleware`，现在 mw 就变成了`[respond, S1, S2, S3]`。

`compose(mw)`将中间件数组转为一个层层调用的 Generator 函数。

```js
function compose(middleware){
  return function *(next){
    if (!next) next = noop();

    var i = middleware.length;

    while (i--) {
      next = middleware[i].call(this, next);
    }

    yield *next;
  }
}

function *noop(){}
```

上面代码中，下一个 generator 函数总是上一个 Generator 函数的参数，从而保证了层层调用。

`var fn = co.wrap(gen)`则是将 Generator 函数包装成一个自动执行的函数，并且返回一个 Promise。

```js
//co package
co.wrap = function (fn) {
  return function () {
    return co.call(this, fn.apply(this, arguments));
  };
};
```

由于`co.wrap(compose(mw))`执行后，返回的是一个 Promise，所以可以对其使用 catch 方法指定捕捉错误的回调函数`fn.call(ctx).catch(ctx.onerror)`。

将所有的上下文变量都放进 context 对象。

```js
app.createContext = function(req, res){
  var context = Object.create(this.context);
  var request = context.request = Object.create(this.request);
  var response = context.response = Object.create(this.response);
  context.app = request.app = response.app = this;
  context.req = request.req = response.req = req;
  context.res = request.res = response.res = res;
  request.ctx = response.ctx = context;
  request.response = response;
  response.request = request;
  context.onerror = context.onerror.bind(context);
  context.originalUrl = request.originalUrl = req.url;
  context.cookies = new Cookies(req, res, this.keys);
  context.accept = request.accept = accepts(req);
  context.state = {};
  return context;
};
```

真正处理 HTTP 请求的是下面这个 Generator 函数。

```js
function *respond(next) {
  yield *next;

  // allow bypassing koa
  if (false === this.respond) return;

  var res = this.res;
  if (res.headersSent || !this.writable) return;

  var body = this.body;
  var code = this.status;

  // ignore body
  if (statuses.empty[code]) {
    // strip headers
    this.body = null;
    return res.end();
  }

  if ('HEAD' == this.method) {
    if (isJSON(body)) this.length = Buffer.byteLength(JSON.stringify(body));
    return res.end();
  }

  // status body
  if (null == body) {
    this.type = 'text';
    body = this.message || String(code);
    this.length = Buffer.byteLength(body);
    return res.end(body);
  }

  // responses
  if (Buffer.isBuffer(body)) return res.end(body);
  if ('string' == typeof body) return res.end(body);
  if (body instanceof Stream) return body.pipe(res);

  // body: json
  body = JSON.stringify(body);
  this.length = Buffer.byteLength(body);
  res.end(body);
}
```

## 参考链接

*   [Koa Guide](https://github.com/koajs/koa/blob/master/docs/guide.md)
*   William XING, [Is Koa.js right for me?](http://william.xingyp.com/is-koa-js-right-for-me/)