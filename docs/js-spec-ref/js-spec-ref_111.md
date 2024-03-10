# 13.14 Cluster 模块

*   概述
    *   基本用法
    *   cluster.worker 对象
    *   cluster.workers 对象
    *   属性与方法
        *   isMaster，isWorker
        *   fork()
        *   kill()
        *   listening 事件
    *   实例：不中断地重启 Node 服务
    *   PM2 模块
    *   参考链接

## 概述

### 基本用法

Node.js 默认单进程运行，对于多核 CPU 的计算机来说，这样做效率很低，因为只有一个核在运行，其他核都在闲置。cluster 模块就是为了解决这个问题而提出的。

cluster 模块允许设立一个主进程和若干个 worker 进程，由主进程监控和协调 worker 进程的运行。worker 之间采用进程建通信交换消息，cluster 模块内置一个负载均衡器，采用 Round-robin 算法协调各个 worker 进程之间的负载。运行时，所有新建立的链接都由主进程完成，然后主进程再把 TCP 连接分配给指定的 worker 进程。

```
var cluster = require('cluster');
var os = require('os');

if (cluster.isMaster){
  for (var i = 0, n = os.cpus().length; i < n; i += 1){
    cluster.fork();
  }
} else {
  http.createServer(function(req, res) {
    res.writeHead(200);
    res.end("hello world\n");
  }).listen(8000);
}
```

上面代码先判断当前进程是否为主进程（cluster.isMaster），如果是的，就按照 CPU 的核数，新建若干个 worker 进程；如果不是，说明当前进程是 worker 进程，则在该进程启动一个服务器程序。

### cluster.worker 对象

cluster.worker 指向当前 worker 进程对象，主进程没有这个值。

它有如下属性。

（1）worker.id

work.id 返回当前 worker 的独一无二的进程编号。这个编号也是 cluster.workers 中指向当前进程的索引值。

（2）worker.process

所有的 worker 进程都是用 child_process.fork()生成的。child_process.fork()返回的对象，就被保存在 worker.process 之中。通过这个属性，可以获取 worker 所在的进程对象。

（3）worker.send()

该方法用于在主进程中，向子进程发送信息。

```
if (cluster.isMaster) {
  var worker = cluster.fork();
  worker.send('hi there');
} else if (cluster.isWorker) {
  process.on('message', function(msg) {
    process.send(msg);
  });
}
```

上面代码的作用是，worker 进程对主进程发出的每个消息，都做回声。

在 worker 进程中调用这个方法，等同于 process.send(message)。

### cluster.workers 对象

该对象只有主进程才有，包含了所有 worker 进程。每个成员的键值就是一个 worker 进程，键名就是该 worker 进程的 worker.id 属性。

```
function eachWorker(callback) {
  for (var id in cluster.workers) {
    callback(cluster.workers[id]);
  }
}
eachWorker(function(worker) {
  worker.send('big announcement to all workers');
});
```

上面代码用来遍历所有 worker 进程。

当前 socket 的 data 事件，也可以用 id 属性识别 worker 进程。

```
socket.on('data', function(id) {
  var worker = cluster.workers[id];
});
```

## 属性与方法

### isMaster，isWorker

isMaster 属性返回一个布尔值，表示当前进程是否为主进程。这个属性由 process.env.NODE_UNIQUE_ID 决定，如果 process.env.NODE_UNIQUE_ID 为未定义，就表示该进程是主进程。

isWorker 属性返回一个布尔值，表示当前进程是否为 work 进程。它与 isMaster 属性的值正好相反。

### fork()

fork 方法用于新建一个 worker 进程，上下文都复制主进程。只有主进程才能调用这个方法。

该方法返回一个 worker 对象。

### kill()

kill 方法用于终止 worker 进程。它可以接受一个参数，表示系统信号。

如果当前是主进程，就会终止与 worker.process 的联络，然后将系统信号法发向 worker 进程。如果当前是 worker 进程，就会终止与主进程的通信，然后退出，返回 0。

在以前的版本中，该方法也叫做 worker.destroy() 。

### listening 事件

worker 进程调用 listen 方面以后，“listening”就传向该进程的服务器，然后传向主进程。

该事件的回调函数接受两个参数，一个是当前 worker 对象，另一个是地址对象，包含网址、端口、地址类型（IPv4、IPv6、Unix socket、UDP）等信息。这对于那些服务多个网址的 Node 应用程序非常有用。

```
cluster.on('listening', function(worker, address) {
  console.log("A worker is now connected to " + address.address + ":" + address.port);
});
```

## 实例：不中断地重启 Node 服务

重启服务需要关闭后再启动，利用 cluster 模块，可以做到先启动一个 worker 进程，再把原有的所有 work 进程关闭。这样就能实现不中断地重启 Node 服务。

下面是主进程的代码 master.js。

```
var cluster = require('cluster');

console.log('started master with ' + process.pid);

// 新建一个 worker 进程
cluster.fork();

process.on('SIGHUP', function () {
  console.log('Reloading...');
  var new_worker = cluster.fork();
  new_worker.once('listening', function () {
    // 关闭所有其他 worker 进程
    for(var id in cluster.workers) {
      if (id === new_worker.id.toString()) continue;
      cluster.workers[id].kill('SIGTERM');
    }
  });
});
```

上面代码中，主进程监听 SIGHUP 事件，如果发生该事件就关闭其他所有 worker 进程。之所以是 SIGHUP 事件，是因为 nginx 服务器监听到这个信号，会创造一个新的 worker 进程，重新加载配置文件。另外，关闭 worker 进程时，主进程发送 SIGTERM 信号，这是因为 Node 允许多个 worker 进程监听同一个端口。

下面是 worker 进程的代码 server.js。

```
var cluster = require('cluster');

if (cluster.isMaster) {
  require('./master');
  return;
}

var express = require('express');
var http = require('http');
var app = express();

app.get('/', function (req, res) {
  res.send('ha fsdgfds gfds gfd!');
});

http.createServer(app).listen(8080, function () {
  console.log('http://localhost:8080');
});
```

使用时代码如下。

```
$ node server.js
started master with 10538
http://localhost:8080
```

然后，向主进程连续发出两次 SIGHUP 信号。

```
$ kill -SIGHUP 10538
$ kill -SIGHUP 10538
```

主进程会连续两次新建一个 worker 进程，然后关闭所有其他 worker 进程，显示如下。

```
Reloading...
http://localhost:8080
Reloading...
http://localhost:8080
```

最后，向主进程发出 SIGTERM 信号，关闭主进程。

```
$ kill 10538
```

## PM2 模块

PM2 模块是 cluster 模块的一个包装层。它的作用是尽量将 cluster 模块抽象掉，让用户像使用单进程一样，部署多进程 Node 应用。

```
// app.js
var http = require('http');

http.createServer(function(req, res) {
  res.writeHead(200);
  res.end("hello world");
}).listen(8080);
```

上面代码是标准的 Node 架设 Web 服务器的方式，然后用 PM2 从命令行启动这段代码。

```
$ pm2 start app.js -i 4
```

上面代码的 i 参数告诉 PM2，这段代码应该在 cluster_mode 启动，且新建 worker 进程的数量是 4 个。如果 i 参数的值是 0，那么当前机器有几个 CPU 内核，PM2 就会启动几个 worker 进程。

如果一个 worker 进程由于某种原因挂掉了，会立刻重启该 worker 进程。

```
# 重启所有 worker 进程
$ pm2 reload all
```

每个 worker 进程都有一个 id，可以用下面的命令查看单个 worker 进程的详情。

```
$ pm2 show <worker id>
```

正确情况下，PM2 采用 fork 模式新建 worker 进程，即主进程 fork 自身，产生一个 worker 进程。`pm2 reload`命令则会用 spawn 方式启动，即一个接一个启动 worker 进程，一个新的 worker 启动成功，再杀死一个旧的 worker 进程。采用这种方式，重新部署新版本时，服务器就不会中断服务。

```
$ pm2 reload <脚本文件名>
```

关闭 worker 进程的时候，可以部署下面的代码，让 worker 进程监听 shutdown 消息。一旦收到这个消息，进行完毕收尾清理工作再关闭。

```
process.on('message', function(msg) {
  if (msg === 'shutdown') {
    close_all_connections();
    delete_logs();
    server.close();
    process.exit(0);
  }
});
```

## 参考链接

*   José F. Romaniello, [Reloading node with no downtime](http://joseoncode.com/2015/01/18/reloading-node-with-no-downtime/)
*   Joni Shkurti, [Node.js clustering made easy with PM2](https://keymetrics.io/2015/03/26/pm2-clustering-made-easy/)