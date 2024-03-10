# 13.12 Http 模块

*   基本用法
    *   处理 GET 请求
    *   处理 POST 请求
    *   发出请求
        *   get()
        *   request()
    *   搭建 HTTPs 服务器
    *   模块属性
    *   模块方法

## 基本用法

### 处理 GET 请求

Http 模块主要用于搭建 HTTP 服务。使用 Node.js 搭建 HTTP 服务器非常简单。

```js
var http = require('http');

http.createServer(function (request, response){
  response.writeHead(200, {'Content-Type': 'text/plain'});
  response.end('Hello World\n');
}).listen(8080, "127.0.0.1");

console.log('Server running on port 8080.');
```

上面代码第一行`var http = require("http")`，表示加载 http 模块。然后，调用 http 模块的 createServer 方法，创造一个服务器实例，将它赋给变量 http。

ceateServer 方法接受一个函数作为参数，该函数的 request 参数是一个对象，表示客户端的 HTTP 请求；response 参数也是一个对象，表示服务器端的 HTTP 回应。response.writeHead 方法表示，服务器端回应一个 HTTP 头信息；response.end 方法表示，服务器端回应的具体内容，以及回应完成后关闭本次对话。最后的 listen(8080)表示启动服务器实例，监听本机的 8080 端口。

将上面这几行代码保存成文件 app.js，然后用 node 调用这个文件，服务器就开始运行了。

```js
$ node app.js
```

这时命令行窗口将显示一行提示“Server running at port 8080.”。打开浏览器，访问[`localhost:8080，网页显示“Hello`](http://localhost:8080) world!”。

上面的例子是当场生成网页，也可以事前写好网页，存在文件中，然后利用 fs 模块读取网页文件，将其返回。

```js
var http = require('http');
var fs = require('fs');

http.createServer(function (request, response){
  fs.readFile('data.txt', function readData(err, data) {
    response.writeHead(200, {'Content-Type': 'text/plain'});
    response.end(data);
  });
}).listen(8080, "127.0.0.1");

console.log('Server running on port 8080.');
```

下面的修改则是根据不同网址的请求，显示不同的内容，已经相当于做出一个网站的雏形了。

```js
var http = require("http");

http.createServer(function(req, res) {

  // 主页
  if (req.url == "/") {
    res.writeHead(200, { "Content-Type": "text/html" });
    res.end("Welcome to the homepage!");
  }

  // About 页面
  else if (req.url == "/about") {
    res.writeHead(200, { "Content-Type": "text/html" });
    res.end("Welcome to the about page!");
  }

  // 404 错误
  else {
    res.writeHead(404, { "Content-Type": "text/plain" });
    res.end("404 error! File not found.");
  }

}).listen(8080, "localhost");
```

回调函数的 req（request）对象，拥有以下属性。

*   url：发出请求的网址。
*   method：HTTP 请求的方法。
*   headers：HTTP 请求的所有 HTTP 头信息。

### 处理 POST 请求

当客户端采用 POST 方法发送数据时，服务器端可以对 data 和 end 两个事件，设立监听函数。

```js
var http = require('http');

http.createServer(function (req, res) {
  var content = "";

  req.on('data', function (chunk) {
    content += chunk;
  });

  req.on('end', function () {
    res.writeHead(200, {"Content-Type": "text/plain"});
    res.write("You've sent: " + content);
    res.end();
  });

}).listen(8080);
```

data 事件会在数据接收过程中，每收到一段数据就触发一次，接收到的数据被传入回调函数。end 事件则是在所有数据接收完成后触发。

对上面代码稍加修改，就可以做出文件上传的功能。

```js
"use strict";

var http = require('http');
var fs = require('fs');
var destinationFile, fileSize, uploadedBytes;

http.createServer(function (request, response) {
  response.writeHead(200);
  destinationFile = fs.createWriteStream("destination.md");
  request.pipe(destinationFile);
  fileSize = request.headers['content-length'];
  uploadedBytes = 0;

  request.on('data', function (d) {
    uploadedBytes += d.length;
    var p = (uploadedBytes / fileSize) * 100;
    response.write("Uploading " + parseInt(p, 0) + " %\n");
  });

  request.on('end', function () {
    response.end("File Upload Complete");
  });
}).listen(3030, function () {
  console.log("server started");
});
```

## 发出请求

### get()

get 方法用于发出 get 请求。

```js
function getTestPersonaLoginCredentials(callback) {
  return http.get({
    host: 'personatestuser.org',
    path: '/email'
  }, function(response) {
    var body = '';

    response.on('data', function(d) {
      body += d;
    });

    response.on('end', function() {
      var parsed = JSON.parse(body);
      callback({
        email: parsed.email,
        password: parsed.pass
      });
    });
  });
},
```

### request()

request 方法用于发出 HTTP 请求，它的使用格式如下。

```js
http.request(options[, callback])
```

request 方法的 options 参数，可以是一个对象，也可以是一个字符串。如果是字符串，就表示这是一个 URL，Node 内部就会自动调用`url.parse()`，处理这个参数。

options 对象可以设置如下属性。

*   host：HTTP 请求所发往的域名或者 IP 地址，默认是 localhost。
*   hostname：该属性会被`url.parse()`解析，优先级高于 host。
*   port：远程服务器的端口，默认是 80。
*   localAddress：本地网络接口。
*   socketPath：Unix 网络套接字，格式为 host:port 或者 socketPath。
*   method：指定 HTTP 请求的方法，格式为字符串，默认为 GET。
*   path：指定 HTTP 请求的路径，默认为根路径（/）。可以在这个属性里面，指定查询字符串，比如`/index.html?page=12`。如果这个属性里面包含非法字符（比如空格），就会抛出一个错误。
*   headers：一个对象，包含了 HTTP 请求的头信息。
*   auth：一个代表 HTTP 基本认证的字符串`user:password`。
*   agent：控制缓存行为，如果 HTTP 请求使用了 agent，则 HTTP 请求默认为`Connection: keep-alive`，它的可能值如下：
    *   undefined（默认）：对当前 host 和 port，使用全局 Agent。
    *   Agent：一个对象，会传入 agent 属性。
    *   false：不缓存连接，默认 HTTP 请求为`Connection: close`。
*   keepAlive：一个布尔值，表示是否保留 socket 供未来其他请求使用，默认等于 false。
*   keepAliveMsecs：一个整数，当使用 KeepAlive 的时候，设置多久发送一个 TCP KeepAlive 包，使得连接不要被关闭。默认等于 1000，只有 keepAlive 设为 true 的时候，该设置才有意义。

request 方法的 callback 参数是可选的，在 response 事件发生时触发，而且只触发一次。

`http.request()`返回一个`http.ClientRequest`类的实例。它是一个可写数据流，如果你想通过 POST 方法发送一个文件，可以将文件写入这个 ClientRequest 对象。

下面是发送 POST 请求的一个例子。

```js
var postData = querystring.stringify({
  'msg' : 'Hello World!'
});

var options = {
  hostname: 'www.google.com',
  port: 80,
  path: '/upload',
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Content-Length': postData.length
  }
};

var req = http.request(options, function(res) {
  console.log('STATUS: ' + res.statusCode);
  console.log('HEADERS: ' + JSON.stringify(res.headers));
  res.setEncoding('utf8');
  res.on('data', function (chunk) {
    console.log('BODY: ' + chunk);
  });
});

req.on('error', function(e) {
  console.log('problem with request: ' + e.message);
});

// write data to request body
req.write(postData);
req.end();
```

注意，上面代码中，`req.end()`必须被调用，即使没有在请求体内写入任何数据，也必须调用。因为这表示已经完成 HTTP 请求。

发送过程的任何错误（DNS 错误、TCP 错误、HTTP 解析错误），都会在 request 对象上触发 error 事件。

## 搭建 HTTPs 服务器

搭建 HTTPs 服务器需要有 SSL 证书。对于向公众提供服务的网站，SSL 证书需要向证书颁发机构购买；对于自用的网站，可以自制。

自制 SSL 证书需要 OpenSSL，具体命令如下。

```js
openssl genrsa -out key.pem
openssl req -new -key key.pem -out csr.pem
openssl x509 -req -days 9999 -in csr.pem -signkey key.pem -out cert.pem
rm csr.pem
```

上面的命令生成两个文件：ert.pem（证书文件）和 key.pem（私钥文件）。有了这两个文件，就可以运行 HTTPs 服务器了。

Node.js 提供一个 https 模块，专门用于处理加密访问。

```js
var https = require('https');
var fs = require('fs');

var options = {
  key: fs.readFileSync('key.pem'),
  cert: fs.readFileSync('cert.pem')
};

var a = https.createServer(options, function (req, res) {
  res.writeHead(200);
  res.end("hello world\n");
}).listen(8000);
```

上面代码显示，HTTPs 服务器与 HTTP 服务器的最大区别，就是 createServer 方法多了一个 options 参数。运行以后，就可以测试是否能够正常访问。

```js
curl -k https://localhost:8000
```

## 模块属性

（1）HTTP 请求的属性

*   headers：HTTP 请求的头信息。
*   url：请求的路径。

## 模块方法

（1）http 模块的方法

*   createServer(callback)：创造服务器实例。

（2）服务器实例的方法

*   listen(port)：启动服务器监听指定端口。

（3）HTTP 回应的方法

*   setHeader(key, value)：指定 HTTP 头信息。
*   write(str)：指定 HTTP 回应的内容。
*   end()：发送 HTTP 回应。