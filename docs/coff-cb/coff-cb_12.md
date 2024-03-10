# 十二、网络

## 客户端

### 问题

你想使用网络上提供的服务。

### 解决方案

创建一个基本的 TCP 客户机。

#### 在 Node.js 中

```js
net = require 'net'

domain = 'localhost'
port = 9001

connection = net.createConnection port, domain

connection.on 'connect', () ->
    console.log "Opened connection to #{domain}:#{port}."

connection.on 'data', (data) ->
    console.log "Received: #{data}"
    connection.end()
```

### 使用示例

可访问 [Basic Server](http://coffeescript-cookbook.github.io/chapters/networking/basic-server) ：

```js
$ coffee basic-client.coffee
Opened connection to localhost:9001
Received: Hello, World!
```

### 讨论

最重要的工作发生在 *connection.on 'data'* 处理过程中，客户端接收到来自服务器的响应并最有可能安排对它的应答。

另请参阅 [Basic Server](http://coffeescript-cookbook.github.io/chapters/networking/basic-server)，[Bi-Directional Client](http://coffeescript-cookbook.github.io/chapters/networking/bi-directional-client) 和 [Bi-Directional Server](http://coffeescript-cookbook.github.io/chapters/networking/bi-directional-server) 。

### 练习

*   根据命令行参数或配置文件为选定的目标域和端口添加支持。

## HTTP 客户端

### 问题

你想创建一个 HTTP 客户端。

### 解决方案

在这个方法中,我们将使用 [node.js's](http://nodejs.org/) HTTP 库。我们将从一个简单的客户端 GET 请求示例返回计算机的外部 IP 。

#### 关于 GET

```js
http = require 'http'

http.get { host: 'www.google.com' }, (res) ->
    console.log res.statusCode
```

get 函数，从 node.js's http 模块，发出一个 GET 请求到一个 http 服务器。响应是以回调的形式，我们可以在一个函数中处理。这个例子仅仅输出响应状态代码。检查一下：

```js
$ coffee http-client.coffee 
200
```

#### 我的 IP 是什么?

如果你是在一个类似局域网的依赖于 NAT 的网络中,你可能会面临找出外部 IP 地址的问题。让我们为这个问题写一个小的 coffeescript 。

```js
http = require 'http'

http.get { host: 'checkip.dyndns.org' }, (res) ->
    data = ''
    res.on 'data', (chunk) ->
        data += chunk.toString()
    res.on 'end', () ->
        console.log data.match(/([0-9]+\.){3}[0-9]+/)[0]
```

我们可以从监听 'data' 事件的结果对象中得到数据，知道它结束了一次 'end' 的触发事件。当这种情况发生时，我们可以做一个简单的正则表达式来匹配我们提取的 IP 地址。试一试：

```js
$ coffee http-client.coffee 
123.123.123.123
```

### 讨论

请注意 http.get 是 http.request 的快捷方式。后者允许您使用不同的方法发出 HTTP 请求，如 POST 或 PUT。

在这个问题上的 API 和整体信息，检查 node.js's [http](http://nodejs.org/docs/latest/api/http.html) 和 [https](http://nodejs.org/docs/latest/api/https.html) 文档页面。此外, [HTTP spec](http://www.ietf.org/rfc/rfc2616.txt) 可能派上用场。

### 练习

*   为键值存储 HTTP 服务器创建一个客户端，使用基本的 HTTP 服务器方法。

## 基本的 HTTP 服务器

### 问题

你想在网络上创建一个 HTTP 服务器。在这个方法中，我们将逐步从最小的服务器成为一个功能键值存储。

### 解决方案

我们将使用 [node.js](http://nodejs.org/) HTTP 库并在 Coffeescript 中创建最简单的 web 服务器。

#### 开始 'hi\n'

我们可以通过导入 node.js HTTP 模块开始。这会包含 createServer ，一个简单的请求处理程序返回 HTTP 服务器。我们可以使用该服务器监听 TCP 端口。

```js
http = require 'http'
server = http.createServer (req, res) -> res.end 'hi\n'
server.listen 8000
```

要运行这个例子，只需放在一个文件中并运行它。你可以用 ctrl-c 终止它。我们可以使用 curl 命令测试它,可用在大多数 *nix 平台：

```js
$ curl -D - http://localhost:8000/
HTTP/1.1 200 OK
Connection: keep-alive
Transfer-Encoding: chunked

hi
```

#### 发生什么了?

让我们一点点来反馈服务器上发生的事情。这时，我们可以友好的对待用户并提供他们一些 HTTP 头文件。

```js
http = require 'http'

server = http.createServer (req, res) ->
    console.log req.method, req.url
    data = 'hi\n'
    res.writeHead 200,
        'Content-Type':     'text/plain'
        'Content-Length':   data.length
    res.end data

server.listen 8000
```

再次尝试访问它，但是这一次使用不同的 URL 路径，比如 `http://localhost:8000/coffee` 。你会看到这样的服务器控制台：

```js
$ coffee http-server.coffee 
GET /
GET /coffee
GET /user/1337
```

#### 得到的东西

假如我们的网络服务器能够保存一些数据会怎么样？我们将在通过 GET 方法 请求检索的元素中设法想出一个简单的键值存储。并提供一个关键路径，服务器将请求返回相应的值,如果不存在则返回 404 错误。

```js
http = require 'http'

store = # we'll use a simple object as our store
    foo:    'bar'
    coffee: 'script'

server = http.createServer (req, res) ->
    console.log req.method, req.url

    value = store[req.url[1..]]

    if not value
        res.writeHead 404
    else
        res.writeHead 200,
            'Content-Type': 'text/plain'
            'Content-Length': value.length + 1
        res.write value + '\n'

    res.end()

server.listen 8000
```

我们可以试试几种 url，看看它们如何回应：

```js
$ curl -D - http://localhost:8000/coffee
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 7
Connection: keep-alive

script

$ curl -D - http://localhost:8000/oops
HTTP/1.1 404 Not Found
Connection: keep-alive
Transfer-Encoding: chunked
```

#### 使用你的头文件

text/plain 是站不住脚的。如果我们使用 application/json 或 text/xml 会怎么样？同时,我们的存储检索过程也可以用一点重构——一些异常的抛出 & 处理怎么样? 来看看我们能想出什么：

```js
http = require 'http'

 # known mime types

[any, json, xml] = ['*/*', 'application/json', 'text/xml']

 # gets a value from the db in format [value, contentType]

get = (store, key, format) ->
    value = store[key]
    throw 'Unknown key' if not value
    switch format
        when any, json then [JSON.stringify({ key: key, value: value }), json]
        when xml then ["<key>#{ key }</key>\n<value>#{ value }</value>", xml]
        else throw 'Unknown format'

store =
    foo:    'bar'
    coffee: 'script'

server = http.createServer (req, res) ->
    console.log req.method, req.url

    try
        key = req.url[1..]
        [value, contentType] = get store, key, req.headers.accept
        code = 200
    catch error
        contentType = 'text/plain'
        value = error
        code = 404

    res.writeHead code,
        'Content-Type': contentType
        'Content-Length': value.length + 1
    res.write value + '\n'
    res.end()

server.listen 8000
```

这个服务器仍然会返回一个匹配给定键的值,如果不存在则返回 404 错误。但它根据标头 Accept 将响应在 JSON 或 XML 结构中。可亲眼看一下：

```js
$ curl http://localhost:8000/
Unknown key

$ curl http://localhost:8000/coffee
{"key":"coffee","value":"script"}

$ curl -H "Accept: text/xml" http://localhost:8000/coffee
<key>coffee</key>
<value>script</value>

$ curl -H "Accept: image/png" http://localhost:8000/coffee
Unknown format
```

### 你需要有所返回

我们的最后一步是提供客户端存储数据的能力。我们将通过监听 POST 请求来保持 RESTiness。

```js
http = require 'http'

 # known mime types

[any, json, xml] = ['*/*', 'application/json', 'text/xml']

 # gets a value from the db in format [value, contentType]

get = (store, key, format) ->
    value = store[key]
    throw 'Unknown key' if not value
    switch format
        when any, json then [JSON.stringify({ key: key, value: value }), json]
        when xml then ["<key>#{ key }</key>\n<value>#{ value }</value>", xml]
        else throw 'Unknown format'

 # puts a value in the db

put = (store, key, value) ->
    throw 'Invalid key' if not key or key is ''
    store[key] = value

store =
    foo:    'bar'
    coffee: 'script'

 # helper function that responds to the client

respond = (res, code, contentType, data) ->
    res.writeHead code,
        'Content-Type': contentType
        'Content-Length': data.length
    res.write data
    res.end()

server = http.createServer (req, res) ->
    console.log req.method, req.url
    key = req.url[1..]
    contentType = 'text/plain'
    code = 404

    switch req.method
        when 'GET'
            try
                [value, contentType] = get store, key, req.headers.accept
                code = 200
            catch error
                value = error
            respond res, code, contentType, value + '\n'

        when 'POST'
            value = ''
            req.on 'data', (chunk) -> value += chunk
            req.on 'end', () ->
                try
                    put store, key, value
                    value = ''
                    code = 200
                catch error
                    value = error + '\n'
                respond res, code, contentType, value

server.listen 8000
```

在一个 POST 请求中注意数据是如何接收的。通过在“数据”和“结束”请求对象的事件中附上一些处理程序，我们最终能够从客户端缓冲和保存数据。

```js
$ curl -D - http://localhost:8000/cookie
HTTP/1.1 404 Not Found # ...
Unknown key

$ curl -D - -d "monster" http://localhost:8000/cookie
HTTP/1.1 200 OK # ...

$ curl -D - http://localhost:8000/cookie
HTTP/1.1 200 OK # ...
{"key":"cookie","value":"monster"}
```

### 讨论

给 http.createServer 一个函数 (request，response) - >…… 它将返回一个服务器对象，我们可以用它来监听一个端口。让服务器与 request 和 response 对象交互。使用 server.listen 8000 监听端口 8000。

在这个问题上的 API 和整体信息，参考 node.js [http](http://nodejs.org/docs/latest/api/http.html) 和 [https](http://nodejs.org/docs/latest/api/https.html) 文档页面。此外，[HTTP spec](http://www.ietf.org/rfc/rfc2616.txt) 可能派上用场。

### 练习

在服务器和开发人员之间创建一个层，允许开发人员做类似的事情：

```js
server = layer.createServer
    'GET /': (req, res) ->
        ...
    'GET /page': (req, res) ->
        ...
    'PUT /image': (req, res) ->
        ...
```

## 服务器

### 问题

你想在网络上提供一个服务器。

### 解决方案

创建一个基本的 TCP 服务器。

### 在 Node.js 中

```js
net = require 'net'

domain = 'localhost'
port = 9001

server = net.createServer (socket) ->
    console.log "Received connection from #{socket.remoteAddress}"
    socket.write "Hello, World!\n"
    socket.end()

console.log "Listening to #{domain}:#{port}"
server.listen port, domain
```

### 使用示例

可访问 [Basic Client](http://coffeescript-cookbook.github.io/chapters/networking/basic-client)：

```js
$ coffee basic-server.coffee
Listening to localhost:9001
Received connection from 127.0.0.1
Received connection from 127.0.0.1
[...]
```

### 讨论

函数将为每个客户端新连接的新插口传递给 @net.createServer@ 。基本的服务器与访客只进行简单地交互，但是复杂的服务器会将插口连上一个专用的处理程序,然后返回等待下一个用户的任务。

另请参阅 [Basic Client](http://coffeescript-cookbook.github.io/chapters/networking/basic-client)，[Bi-Directional Server](http://coffeescript-cookbook.github.io/chapters/networking/bi-directional-client) 和 [Bi-Directional Client](http://coffeescript-cookbook.github.io/chapters/networking/bi-directional-client)。

### 练习

*   为选定的目标域和基于命令行参数或配置文件的端口添加支持。

## 双向客户端

### 问题

你想通过网络提供持续的服务,与客户保持持续的联系。

### 解决方案

创建一个双向 TCP 客户机。

### 在 Node.js 中

```js
net = require 'net'

domain = 'localhost'
port = 9001

ping = (socket, delay) ->
    console.log "Pinging server"
    socket.write "Ping"
    nextPing = -> ping(socket, delay)
    setTimeout nextPing, delay

connection = net.createConnection port, domain

connection.on 'connect', () ->
    console.log "Opened connection to #{domain}:#{port}"
    ping connection, 2000

connection.on 'data', (data) ->
    console.log "Received: #{data}"

connection.on 'end', (data) ->
    console.log "Connection closed"
    process.exit()
```

### 使用示例

可访问 [Bi-Directional Server](http://coffeescript-cookbook.github.io/chapters/networking/bi-directional-server)：

```js
$ coffee bi-directional-client.coffee
Opened connection to localhost:9001
Pinging server
Received: You have 0 peers on this server
Pinging server
Received: You have 0 peers on this server
Pinging server
Received: You have 1 peer on this server
[...]
Connection closed
```

### 讨论

这个特殊示例发起与服务器联系并在 @connection.on 'connect'@ 处理程序中开启对话。大量的工作在一个真正的用户中，然而 @connection.on 'data'@ 处理来自服务器的输出。@ping@ 函数递归是为了说明连续与服务器通信可能被真实的用户移除。

另请参阅 [Bi-Directional Server](http://coffeescript-cookbook.github.io/chapters/networking/bi-directional-server)，[Basic Client](http://coffeescript-cookbook.github.io/chapters/networking/basic-client) 和 [Basic Server](http://coffeescript-cookbook.github.io/chapters/networking/basic-server)。

### 练习

*   为选定的目标域和基于命令行参数或配置文件的端口添加支持。

## 双向服务器

### 问题

你想通过网络提供持续的服务，与客户保持持续的联系。

### 解决方案

创建一个双向 TCP 服务器。

### 在 Node.js 中

```js
net = require 'net'

domain = 'localhost'
port = 9001

server = net.createServer (socket) ->
    console.log "New connection from #{socket.remoteAddress}"

    socket.on 'data', (data) ->
        console.log "#{socket.remoteAddress} sent: #{data}"
        others = server.connections - 1
        socket.write "You have #{others} #{others == 1 and "peer" or "peers"} on this server"

console.log "Listening to #{domain}:#{port}"
server.listen port, domain
```

### 使用示例

可访问 [Bi-Directional Client](http://coffeescript-cookbook.github.io/chapters/networking/bi-directional-client)：

```js
$ coffee bi-directional-server.coffee
Listening to localhost:9001
New connection from 127.0.0.1
127.0.0.1 sent: Ping
127.0.0.1 sent: Ping
127.0.0.1 sent: Ping
[...]
```

### 讨论

大部分工作在 @socket.on 'data'@ 中 ，处理所有的输入端。真正的服务器可能会将数据传给另一个函数处理并生成任何响应以便源程序处理。

### 练习

*   为选定的目标域和基于命令行参数或配置文件的端口添加支持。