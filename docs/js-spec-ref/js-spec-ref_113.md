# 13.16 Net 模块和 DNS 模块

net 模块用于底层的网络通信。

*   服务器端 Socket 接口
*   客户端 Socket 接口
*   DNS 模块

## 服务器端 Socket 接口

下面代码打开一个服务器端 Socket 接口，用来接受客户端的数据。

```js
var serverPort = 9099;
var net = require('net');
var server = net.createServer(function(client) {
  console.log('client connected');
  console.log('client IP Address: ' + client.remoteAddress);
  console.log('is IPv6: ' + net.isIPv6(client.remoteAddress));
  console.log('total server connections: ' + server.connections);

  // Waiting for data from the client.
  client.on('data', function(data) {
    console.log('received data: ' + data.toString());

    // Write data to the client socket.
    client.write('hello from server');
  });

  // Closed socket event from the client.
  client.on('end', function() {
    console.log('client disconnected');
  });
});

server.on('error',function(err){
  console.log(err);
  server.close();
});

server.listen(serverPort, function() {
  console.log('server started on port ' + serverPort);
});
```

上面代码中，createServer 方法建立了一个服务端，一旦收到客户端发送的数据，就发出回应，同时还监听客户端是否中断通信。最后，listen 方法打开服务端。

## 客户端 Socket 接口

客户端 Socket 接口用来向服务器发送数据。

```js
var serverPort = 9099;
var server = 'localhost';
var net = require('net');

console.log('connecting to server...');
var client = net.connect({server:server,port:serverPort},function(){
  console.log('client connected');

  // send data
  console.log('send data to server');
  client.write('greeting from client socket');
});

client.on('data', function(data) {
  console.log('received data: ' + data.toString());
  client.end();
});

client.on('error',function(err){
  console.log(err);
});
client.on('end', function() {
  console.log('client disconnected');
});
```

上面代码连接服务器之后，就向服务器发送数据，然后监听服务器返回的数据。

## DNS 模块

DNS 模块用于解析域名。resolve4 方法用于 IPv4 环境，resolve6 方法用于 IPv6 环境，lookup 方法在以上两种环境都可以使用，返回 IP 地址（address）和当前环境（IPv4 或 IPv6）。

```js
var dns = require('dns');

dns.resolve4('www.pecollege.net', function (err, addresses) {
  if (err)
    console.log(err);

  console.log('addresses: ' + JSON.stringify(addresses));
});

dns.lookup('www.pecollege.net', function (err, address, family) {
  if (err)
    console.log(err);

  console.log('addresses: ' + JSON.stringify(address));
  console.log('family: ' + JSON.stringify(family));
});
```