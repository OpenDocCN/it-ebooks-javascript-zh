# 第九章　网络通信

Chrome 应用通过`sockets`接口支持 TCP 和 UDP 协议，使网络通信成为可能。使用`sockets`接口时，声明权限比较特殊，并不在`permissions`中声明，而是直接在 Manifest 的`sockets`中声明：

```js
"sockets": {
    "udp": {
        "send": ["host-pattern1", ...],
        "bind": ["host-pattern2", ...],
        ...
    },
    "tcp" : {
        "connect": ["host-pattern1", ...],
        ...
    },
    "tcpServer" : {
        "listen": ["host-pattern1", ...],
        ...
    }
} 
```

但在早期的 Chrome 版本中`socket`权限依然在`permissions`中声明。

`sockets`接口传输的数据类型`为 ArrayBuffer`，有关`ArrayBuffer`的内容可以参阅 7.6.1 节的内容。

最后本章还会介绍有关 WebSocket 的内容，这是 HTML5 原生支持的方法。

## 9.1　UDP 协议

UDP 协议是一个简单的面向数据报的传输层协议，它是一种不可靠数据报协议。由于缺乏可靠性且属于非连接导向协定，UDP 应用一般必须允许一定量的丢包和出错。

Chrome 提供`sockets.udp`接口使 Chrome 应用可以进行 UDP 通信。要使用`sockets.udp`接口需要在`sockets`域中声明`udp`权限：

```js
"sockets": {
    "udp": {
        "send": ["192.168.1.106:8000", ":8001"],
        "bind": ":8000"
    }
} 
```

上面的代码表示应用可以通过 UDP 与 192.168.1.106 的 8000 端口通信，也可以与任意主机的 8001 端口通信。应用可以绑定本地的 8000 端口用来接收 UDP 消息。

如果想连接任意主机的任意端口可以声明为`"send": "*"`。主机和端口可以是一个特定的字符串，也可以是一个数组表示多个规则，如`"bind": [":8000", ":8001"]`。如果想连接 192.168.1.106 的任意端口可以声明为`"send": "192.168.1.106"`。

### 9.1.1　建立与关闭连接

**创建 socket**

```js
var socketOption = {
    persistent: true,
    name: 'udpSocket',
    bufferSize: 4096
};

chrome.sockets.udp.create(socketOption, function(socketInfo){
    //We'll do the next step later
}); 
```

`socketOption`是一个可选参数，其中`persistent`代表 Chrome 应用的生命周期结束后（参见 6.4 节）这个 socket 是否还依然处于开启状态。`bufferSize`定义 socket 接收数据的缓存区大小，如果这个值过小，会导致数据丢失，默认值是 4096。

当 socket 创建完毕就返回一个对象，这个对象包含代表这个 socket 的唯一 id，之后对 socket 的操作都将根据这个 id。

**更新 socket 属性**

此处说的属性是指创建 socket 时提到的`socketOptions`。通过`update`方法可以更新 socket 的属性：

```js
chrome.sockets.udp.update(socketId, newSocketOption, function(){
    //do something when update complete
}); 
```

**阻止和解除阻止 socket 接收数据**

当一个 socket 被阻止后，将不会触发消息接收事件，解除阻止后将恢复正常。

```js
//Blocking socket receiving data
var isPaused = true;

chrome.sockets.udp.setPaused(socketId, isPaused, function(){
    //do something after pause a socket
}); 
```

如果想解除阻止，将上述代码中的`isPaused`设为`false`即可。

**绑定端口**

绑定固定端口用以接收 UDP 消息：

```js
var localAddress = '10.60.37.105';
var localPort = 6259;

chrome.sockets.udp.bind(socketId, localAddress, localPort, function(code){
    //if a negative value is returned, an error occurred
    //otherwise do something after bind a port
}); 
```

一台计算机往往有多个 IP，如本地回路 IP、局域网 IP 和公网 IP 等，那么指定`localAddress`为`"0.0.0.0"`系统会自动绑定一个合适的 IP 地址，而不必让开发者获取所有 IP 地址后再进行选择。同样可以指定`localPort`为`0`，这样系统会自动分配一个空闲的端口给应用使用。

**关闭 socket**

当一个 socket 不再被使用了我们应该关闭它。

```js
chrome.socket.udp.close(socketId, function(){
    //do something after close a socket
}); 
```

下面我们来试着封装一个`udp`类，并在之后的内容逐步扩充它：

```js
function udp(){
    var _udp = chrome.sockets.udp;
    this.option = {},
    this.socketId = 0,
    this.localAddress = '0.0.0.0',
    this.localPort = 0,

    this.create = function(callback){
        _udp.create(this.option, function(socketInfo){
            this.socketId = socketInfo.socketId;
            callback();
        }.bind(this));
    }.bind(this),

    this.update = function(){
        _udp.update(this.socketId, newSocketOption, callback);
    }.bind(this),

    this.pause = function(isPaused, callback){
        _udp.setPaused(this.socketId, isPaused, callback);
    }.bind(this),

    this.bind = function(callback){
        _udp.bind(this.socketId, this.localAddress, this.localPort, callback);
    }.bind(this),

    this.close = function(callback){
        _udp.close(this.socketId, callback);
    }.bind(this),

    this.init = function(callback){
        this.create(function(){
            this.bind(callback);
        }.bind(this));
    }.bind(this)
} 
```

调用时指定 socket 属性、绑定 IP 和端口就可以进行初始化了：

```js
var udpSocket = new udp();
udpSocket.option = {
    persistent: true
};
udpSocket.localPort = 8000;
udpSocket.init(function(code){
    if(code<0){
        console.log('UDP Socket bind failed, error code: '+code);
        return false;
    }
    else{
        //We'll do something after udp socket init later
    }
}); 
```

### 9.1.2　发送与接收数据

**发送数据**

Socket 发送的数据类型为`ArrayBuffer`，对`ArrayBuffer`不熟悉的读者请参阅 7.6.1 节的内容。

```js
chrome.sockets.udp.send(socketId, data, address, port, function(){
    //do something after send some data
}); 
```

其中`data`为`ArrayBuffer`类型的数据，`address`为接收方的 ip，`port`为接收方的端口。

**接收数据**

当 socket 接收到数据时，就会触发`onReceive`事件：

```js
chrome.socket.udp.onReceive.addListener(function(info){
    //We'll do something with info later
}); 
```

返回的`info`是一个对象，包括 4 个属性：`socketId`、`data`、`remoteAddress`和`remotePort`。其中`data`为`ArrayBuffer`类型数据，`remoteAddress`和`remotePort`分别为发送方 ip 地址和端口。

**处理异常**

当网络出现问题时，会触发`onReceiveError`事件，同时 socket 会被阻断：

```js
chrome.sockets.udp.onReceiveError.addListener(function(info){
    //We'll do something with info later
}); 
```

与接收数据相似，`info`也是一个对象，但只包含`socketId`和`resultCode`两个属性。

现在我们来把 8.1.1 中的`udp`类完善一下，由于篇幅限制，将只写出添加或改动的部分：

```js
function udp(){
    this.send = function(address, port, data, callback){
        _udp.send(this.socketId, data, address, port, callback);
    }.bind(this),

    this.receive = function(info){
        console.log('Received data from '+info.removeAddress+':'+info.removePort);
    },

    this.error = function(code){
        console.log('An error occurred with code '+code);
    },

    this.init = function(callback){
        this.create(function(){
            this.bind(function(code){
                if(code<0){
                    this.error(code);
                    return false;
                }
                else{
                    callback();
                }
            }.bind(this));
            _udp.onReceive.addListener(function(info){
                if(info.socketId==this.socketId){
                    this.receive(info);
                }
            }.bind(this));
            _udp.onReceiveError.addListener(function(info){
                if(info.socketId==this.socketId){
                    this.error(info.resultCode);
                }
            });
        }.bind(this));
    }.bind(this)
} 
```

### 9.1.3　多播

多播用于一个有限的局部网络中的 UDP 一对多通信。之所以说是一个有限的局部网络是因为这个范围是无法确定的，一个是因为一个数据能传多远由 TTL 决定，多播中 TTL 一般被设为 15，最多不会超过 30，也有设为 0 的（数据不会流出本地）。再一个就是有的路由是不会转发多播数据的，即使 TTL 在此节点并没有减为 0。

多播使用一段保留的 ip 地址，224.0.0.0 到 239.255.255.255。其中 224.0.0.0 到 224.0.0.255 为局部连接多播地址，224.0.1.0 到 238.255.255.255 为预留多播地址，239.0.0.0 到 239.255.255.255 为管理权限多播地址。局部连接多播地址和管理权限多播地址均为保留多播地址，可被自由使用的只有预留多播地址，即 224.0.1.0 到 238.255.255.255。

多播地址是特殊的 ip 地址，它不对应任何物理设备。但 UDP 进行多播时，只将其看做普通的 ip 地址就可以。

要使用多播，需要在`sockets`的`udp`中声明`multicastMembership`权限：

```js
"sockets": {
    "udp": {
        "multicastMembership": "*";
    }
} 
```

如果提示 Invalid host:port pattern，这是 Chrome 的 Bug，解决方法是`multicastMembership`的值设定成空字符串：

```js
"sockets": {
    "udp": {
        "multicastMembership": "";
    }
} 
```

**加入组**

```js
chrome.sockets.udp.joinGroup(socketId, address, function(code){
    //if a negative value is returned, an error occurred
    //otherwise do something after join a group
}); 
```

**离开组**

```js
chrome.sockets.udp.leaveGroup(socketId, address, function(code){
    //if a negative value is returned, an error occurred
    //otherwise do something after leave a group
}); 
```

**设置多播的 TTL**

```js
chrome.sockets.udp.setMulticastTimeToLive(socketId, ttl, function(code){
    //if a negative value is returned, an error occurred
    //otherwise do something after set multicast TTL
}); 
```

**设置多播回环模式**

这个模式定义了当主机本身处于多播的目标组中时，是否接收来自自身的数据。如果一台主机中多个程序加入了同一个多播组，但回环模式设置矛盾时，Windows 会不接收来自本机自身的数据，而基于 Unix 的系统则不接收来自程序自身的数据。

```js
chrome.sockets.udp.setMulticastLoopbackMode(sockedId, enabled, function(code){
    //if a negative value is returned, an error occurred
    //otherwise do something after set multicast loopback mode
}); 
```

让我们把多播的功能加入到 udp 类中：

```js
function udp(){
    this.joinGroup = function(address, callback){
        _udp.joinGroup(this.socketId, address, function(code){
            if(code<0){
                this.error(code);
                return false;
            }
            else{
                callback();
            }
        }.bind(this));
    }.bind(this),

    this.leaveGroup = function(address, callback){
        _udp.leaveGroup(this.socketId, address, function(code){
            if(code<0){
                this.error(code);
                return false;
            }
            else{
                callback();
            }
        }.bind(this));
    }.bind(this),

    this.setMilticastTTL = function(ttl, callback){
        _udp.setMulticastTimeToLive(this.socketId, ttl, function(code){
            if(code<0){
                this.error(code);
                return false;
            }
            else{
                callback();
            }
        }.bind(this));
    }.bind(this),

    this.setMilticastLoopback = function(enabled, callback){
        _udp.setMulticastLoopbackMode(this.sockedId, enabled, function(code){
            if(code<0){
                this.error(code);
                return false;
            }
            else{
                callback();
            }
        }.bind(this));
    }.bind(this)
} 
```

### 9.1.4　获取 socket 和组

**获取指定 socket**

```js
chrome.sockets.udp.getInfo(socketId, function(socketInfo){
    //do something with socketInfo
}); 
```

`socketInfo`是一个描述`socket`信息的对象，包括`socketId`、`persistent`、`name`、`bufferSize`、`paused`、`localAddress`和`localPort`。

**获取全部活动的 socket**

```js
chrome.sockets.udp.getSockets(function(socketInfoArray){
    //do something with socketInfoArray
}); 
```

`socketInfoArray`是一个包含一个或多个`socketInfo`对象的数组。

**获取指定 socket 加入的组**

```js
chrome.sockets.udp.getJoinedGroups(socketId, function(groupArray){
    //do something with groupArray
}); 
```

`groupArray`是一个包含多个组地址字符串的数组。

将以上方法加入到`udp`类中：

```js
function udp(){
    this.getInfo = function(callback){
        _udp.getInfo(this.socketId, callback);
    }.bind(this),

    this.getSockets = function(callback){
        _udp.getSockets (callback);
    }.bind(this),

    this.getGroups = function(callback){
        _udp.getJoinedGroups(this.socketId, callback);
    }.bind(this)
} 
```

### 9.1.5　局域网聊天应用

经过前面的介绍，我们对 UDP 在 Chrome 应用中的使用有了一定的了解，本节来和大家一起编写一款局域网聊天的应用。

这个应用利用 UDP 的多播功能，进行一对多通信，来实现局域网聊天。首先需要在 Manifest 的`sockets`中声明`udp`的权限：

```js
{
    "app": {
        "background": {
            "scripts": ["udp.js", "background.js"]
        }
    },
    "manifest_version": 2,
    "name": "Local Messager",
    "version": "1.0",
    "description": "A local network chating application.",
    "icons": {
        "128": "lm.png"
    },
    "sockets": {
        "udp": {
            "send": "224.0.1.100",
            "bind": ":*",
            "multicastMembership": "224.0.1.100"
        }
    }
} 
```

注意，由于 8.1.3 节中提到的 Bug，`multicastMembership`的值在实际操作中可能需要指定为空字符串`""`，即`"multicastMembership": ""`。

Event Page 中指定的 udp.js 就是我们在之前写好的`udp`类。下面我们来编写 background.js。

首先当应用运行时开始创建 UDP 连接并加入到多播组：

```js
var udpSocket = new udp();
udpSocket.localPort = 8943;
udpSocket.receive = receiveMsg;
udpSocket.init(function(){
    udpSocket.joinGroup('224.0.1.100', function(){
        //Joined group 224.0.1.100
    });
}); 
```

下面需要 background 来监听来自前端页面发来的指令：

```js
chrome.runtime.onMessage.addListener(function(message, sender, callback){
    if(message.action == 'send'){
        var buf = str2ab(message.msg);
        udpSocket.send('224.0.1.100', udpSocket.localPort, buf, function(){
            //message is sent
        });
    }
}); 
```

下面我们来编写接收消息的函数：

```js
function receiveMsg(info){
    var msg = ab2str(info.data);
    chrome.runtime.sendMessage({action:'receive', msg:msg});
} 
```

最后来编写`ArrayBuffer`和`String`类型数据互换的两个函数：

```js
function str2ab(str){
    var buf = new ArrayBuffer(str.length*2);
    bufView = new Uint16Array(buf);
    for(var i=0; i<str.length; i++){
        bufView[i] = str.charCodeAt(i);
    }
    return buf;
}

function ab2str(buf){
    return String.fromCharCode.apply(null, new Uint16Array(buf));
} 
```

UDP 通信相关的内容写好了，接下来创建前端窗口：

```js
chrome.app.runtime.onLaunched.addListener(function(){
    chrome.app.window.create('main.html', {
        'id': 'main',
        'bounds': {
            'width': 400,
            'height': 600
        }
    });
}); 
```

前端窗口 main.html 的 HTML 代码：

```js
<html>
<head>
<title>Local Messager</title>
<style>
* {
    margin: 0;
    padding: 0;
}

body {
    border: #EEE 1px solid;
}

#history {
    margin: 10px;
    border: gray 1px solid;
    height: 480px;
    overflow-y: auto;
    overflow-x: hidden;
}

#history div {
    padding: 10px;
    border-bottom: #EEE 1px solid;
}

#msg {
    margin: 10px;
    width: 480px;
    height: 80px;
    box-sizing: border-box;
    border: gray 1px solid;
    font-size: 24px;
}
</style>
</head>
<body>
<div id="history"></div>
<input type="text" id="msg" />
<script src="main.js"></script>
</body>
</html> 
```

main.js 的代码：

```js
document.getElementById('msg').onkeyup = function(e){
    if(e.keyCode==13){
        chrome.runtime.sendMessage({
            action:'send',
            msg:encodeURIComponent(this.value)
        });
        this.value = '';
    }
}

chrome.runtime.onMessage.addListener(function(message, sender, callback){
    if(message.action == 'receive'){
        var el = document.createElement('div');
        el.innerText = decodeURIComponent(message.msg);
        document.getElementById('history').appendChild(el);
    }
}); 
```

下面是 Local Messager 的运行截图。

![enter image description here](img/00061.jpeg)
*Local Messager*

本节讲解的应用源码可以通过[`github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/local_messager`](https://github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/local_messager)下载得到。

## 9.2　TCP 协议

TCP 是一种面向连接的、可靠的、基于字节流的传输层通信协议。

Chrome 提供`sockets.tcp`接口使 Chrome 应用可以进行 TCP 通信。要使用`sockets.tcp`接口需要在`sockets`域中声明`tcp`权限：

```js
"sockets": {
    "tcp": {
        "connect": ["192.168.1.100:80", ":8080"]
    }
} 
```

上面的代码表示应用可以通过 TCP 与 192.168.1.100 的 80 端口和任意主机的 8080 端口通信。

### 9.2.1　建立与关闭连接

**创建 socket**

```js
var socketOption = {
    persistent: true,
    name: 'tcpSocket',
    bufferSize: 4096
};

chrome.sockets.tcp.create(socketOption, function(socketInfo){
    //We'll do the next step later
}); 
```

`socketOption`是一个可选参数，其中`persistent`代表 Chrome 应用的生命周期结束后（参见 6.4 节）这个 socket 是否还依然处于开启状态。`bufferSize`定义 socket 接收数据的缓存区大小，默认值是 4096。

当 socket 创建完毕就返回一个对象，这个对象包含代表这个 socket 的唯一 id，之后对 socket 的操作都将根据这个 id。

**更新 socket 属性**

此处说的属性是指创建 socket 时提到的 socketOptions。通过 update 方法可以更新 socket 的属性：

```js
chrome.sockets.tcp.update(socketId, newSocketOption, function(){
    //do something when update complete
}); 
```

**阻止和解除阻止 socket 接收数据**

当一个 socket 被阻止后，将不会触发消息接收事件，解除阻止后将恢复正常。

```js
//Blocking socket receiving data
var isPaused = true;

chrome.sockets.tcp.setPaused(socketId, isPaused, function(){
    //do something after pause a socket
}); 
```

如果想解除阻止，将上述代码中的`isPaused`设为`false`即可。

**长连接**

```js
chrome.sockets.tcp.setKeepAlive(socketId, enable, delay, function(code){
    //if a negative value is returned, an error occurred
    //otherwise do something after set keep-alive
}); 
```

当`enable`设为`true`时启用保持连接功能，`delay`定义了最后接收的数据包与第一次探测之间的时间，以秒为单位。

**禁用和启用纳格算法**

纳格算法以减少封包传送量来增进 TCP/IP 网络的效能。可以通过 setNoDelay 方法禁用或启用。

```js
chrome.sockets.tcp.setNoDelay(socketId, noDelay, function(code){
    //if a negative value is returned, an error occurred
    //otherwise do something after set no delay
}); 
```

当`noDelay`设为`true`时设置 TCP 连接的 TCP_NODELAY 标志，禁用纳格算法。

**断开连接**

```js
chrome.sockets.tcp.disconnect(socketId, function(){
    //do something after disconnect a connection
}); 
```

**关闭 socket**

当一个 socket 不再被使用了我们应该关闭它。

```js
chrome.socket.tcp.close(socketId, function(){
    //do something after close a socket
}); 
```

下面我们来试着封装一个`tcp`类，并在之后的内容逐步扩充它：

```js
function tcp(){
    var _tcp = chrome.sockets.tcp;
    this.option = {},
    this.socketId = 0,

    this.create = function(callback){
        _tcp.create(this.option, function(socketInfo){
            this.socketId = socketInfo.socketId;
            callback();
        }.bind(this));
    }.bind(this),

    this.update = function(){
        _tcp.update(this.socketId, newSocketOption, callback);
    }.bind(this),

    this.pause = function(isPaused, callback){
        _tcp.setPaused(this.socketId, isPaused, callback);
    }.bind(this),

    this.keepAlive = function(enable, delay, callback){
        _tcp.setKeepAlive(this.socketId, enable, delay, function(code){
            if(code<0){
                this.error(code);
            }
            else{
                callback();
            }
        }.bind(this));
    }.bind(this),

    this.noDelay = function(noDelay, callback){
        _tcp.setNoDelay(this.socketId, noDelay, function(code){
            if(code<0){
                this.error(code);
            }
            else{
                callback();
            }
        }.bind(this));
    }.bind(this),

    this.disconnect = function(callback){
        _tcp.disconnect(this.socketId, callback);
    }.bind(this),

    this.close = function(callback){
        _tcp.close(this.socketId, callback);
    }.bind(this),

    this.error = function(code){
        console.log('An error occurred with code '+code);
    },

    this.init = function(callback){
        this.create(callback);
    }.bind(this)
} 
```

调用时指定 socket 属性就可以进行初始化了：

```js
var tcpSocket = new tcp();
tcpSocket.option = {
    persistent: true
};
tcpSocket.init(function(){
    //We'll do something after tcp socket init later
}); 
```

### 9.2.2　发送与接收数据

**连接**

```js
chrome.sockets.tcp.connect(socketId, peerAddress, peerPort, function(code){
    //if a negative value is returned, an error occurred
    //otherwise do something after bind a port
}); 
```

**发送数据**

Socket 发送的数据类型为`ArrayBuffer`，对`ArrayBuffer`不熟悉的读者请参阅 7.6.1 节的内容。

```js
chrome.sockets.tcp.send(socketId, data, function(info){
    //if info.resultCode is a negative value, an error occurred
    //otherwise do something after send some data
}); 
```

其中`data`为`ArrayBuffer`类型的数据。

**接收数据**

当 socket 接收到数据时，就会触发`onReceive`事件：

```js
chrome.socket.tcp.onReceive.addListener(function(info){
    //We'll do something with info later
}); 
```

返回的`info`是一个对象，包括两个属性：`socketId`和`data`。其中`data`为`ArrayBuffer`类型数据。

**处理异常**

当网络出现问题时，会触发`onReceiveError`事件，同时 socket 会被阻断：

```js
chrome.sockets.tcp.onReceiveError.addListener(function(info){
    //We'll do something with info later
}); 
```

与接收数据相似，`info`也是一个对象，但包含`socketId`和`resultCode`两个属性。

现在我们来把 8.2.1 中的`tcp`类完善一下，由于篇幅限制，将只写出添加或改动的部分：

```js
function tcp(){
    this.connect = function(address, port, callback){
        _tcp.connect(this.socketId, address, port, function(){
            _tcp.onReceive.addListener(function(info){
                if(info.socketId==this.socketId){
                    this.receive(info);
                }
            }.bind(this));
            _tcp.onReceiveError.addListener(function(info){
                if(info.socketId==this.socketId){
                    this.error(info.resultCode);
                }
            }.bind(this));
        }.bind(this));
    }.bind(this),

    this.send = function(data, callback){
        _tcp.send(this.socketId, data, callback);
    }.bind(this),

    this.receive = function(info){
        console.log('Received data.');
    }
} 
```

### 9.2.3　获取 socket

**获取指定 socket**

```js
chrome.sockets.tcp.getInfo(socketId, function(socketInfo){
    //do something with socketInfo
}); 
```

`socketInfo`是一个描述 socket 信息的对象，包括`socketId`、`name`、`bufferSize`、`paused`、`connected`、`localAddress`、`localPort`、`peerAddress`和`peerPort`。

**获取全部活动的 socket**

```js
chrome.sockets.tcp.getSockets(function(socketInfoArray){
    //do something with socketInfoArray
}); 
```

`socketInfoArray`是一个包含一个或多个`socketInfo`对象的数组。

将以上方法加入到`tcp`类中：

```js
function tcp(){
    this.getInfo = function(callback){
        _tcp.getInfo(this.socketId, callback);
    }.bind(this),

    this.getSockets = function(callback){
        _tcp.getSockets (callback);
    }.bind(this)
} 
```

## 9.3　TCP Server

TCP Server 可以绑定指定端口并被动地接收信息。

Chrome 提供`sockets.tcpServer`接口使 Chrome 应用可以作为 TCP 服务器。要使用`sockets.tcpServer`接口需要在`sockets`域中声明`tcpServer`权限：

```js
"sockets": {
    "tcpServer": {
        "listen": ":80"
    }
} 
```

上面的代码表示应用可以通过监听本地 80 端口接收 TCP 消息。

### 9.3.1　建立与关闭连接

**创建 socket**

```js
var socketOption = {
    persistent: true,
    name: 'tcpSocket'
};

chrome.sockets.tcpServer.create(socketOption, function(socketInfo){
    //We'll do the next step later
}); 
```

`socketOption`是一个可选参数，其中`persistent`代表 Chrome 应用的生命周期结束后（参见 6.4 节）这个 socket 是否还依然处于开启状态。

当 socket 创建完毕就返回一个对象，这个对象包含代表这个 socket 的唯一 id，之后对 socket 的操作都将根据这个 id。

**更新 socket 属性**

此处说的属性是指创建 socket 时提到的`socketOptions`。通过`update`方法可以更新 socket 的属性：

```js
chrome.sockets.tcpServer.update(socketId, newSocketOption, function(){
    //do something when update complete
}); 
```

**阻止和解除阻止 socket 接收数据**

当一个 socket 被阻止后，将不会触发消息接收事件，解除阻止后将恢复正常。

```js
//Blocking socket receiving data
var isPaused = true;

chrome.sockets.tcpServer.setPaused(socketId, isPaused, function(){
    //do something after pause a socket
}); 
```

如果想解除阻止，将上述代码中的`isPaused`设为`false`即可。

**断开连接**

```js
chrome.sockets.tcpServer.disconnect(socketId, function(){
    //do something after disconnect a connection
}); 
```

**关闭 socket**

当一个 socket 不再被使用了我们应该关闭它。

```js
chrome.socket.tcpServer.close(socketId, function(){
    //do something after close a socket
}); 
```

下面我们来试着封装一个`tcpServer`类，并在之后的内容逐步扩充它：

```js
function tcpServer(){
    var _tcpServer = chrome.sockets.tcpServer;
    this.option = {},
    this.socketId = 0,

    this.create = function(callback){
        _tcpServer.create(this.option, function(socketInfo){
            this.socketId = socketInfo.socketId;
            callback();
        }.bind(this));
    }.bind(this),

    this.update = function(){
        _tcpServer.update(this.socketId, newSocketOption, callback);
    }.bind(this),

    this.pause = function(isPaused, callback){
        _tcpServer.setPaused(this.socketId, isPaused, callback);
    }.bind(this),

    this.disconnect = function(callback){
        _tcpServer.disconnect(this.socketId, callback);
    }.bind(this),

    this.close = function(callback){
        _tcpServer.close(this.socketId, callback);
    }.bind(this),

    this.init = function(callback){
        this.create(callback);
    }.bind(this)
} 
```

调用时指定 socket 属性就可以进行初始化了：

```js
var tcpSocket = new tcpServer();
tcpSocket.option = {
    persistent: true
};
tcpSocket.init(function(){
    //We'll do something after tcpSocket socket init later
}); 
```

### 9.3.2　监听数据

**监听端口**

```js
chrome.sockets.tcpServer.listen(socketId, address, port, backlog, function(code){
    //if a negative value is returned, an error occurred
    //otherwise do something after listen complete
}); 
```

`address`和`port`分别是监听本地的地址和端口，此处的端口不要使用`0`，因为这样并不知道具体监听的端口。

**接受连接**

```js
chrome.sockets.tcpServer.onAccept.addListener(function(info){
    //do something when a connection has been made to the server socket
}); 
```

其中`info`包含两个属性，`socketId`和`clientSocketId`。`clientSocketId`是服务器主动与客户端建立的新 socket 连接，通过操作此 socket 可以向客户端发送数据。`clientSocketId`只能使用`chrome.sockets.tcp`接口操作，而不能使用`chrome.sockets.tcpServer`接口。另外`clientSocketId`指向的 socket 处于阻止状态，它并不能接收客户端发回的消息，除非手动执行`setPaused`方法解除阻止。

**处理异常**

当网络出现问题时，会触发`onAcceptError`事件，同时 socket 会被阻断：

```js
chrome.sockets.tcpServer.onAcceptError.addListener(function(info){
    //do something with info
}); 
```

与监测连接相似，`info`也是一个对象，但包含`socketId`和`resultCode`两个属性。

现在我们来把 8.3.1 中的`tcpServer`类完善一下，由于篇幅限制，将只写出添加或改动的部分：

```js
function tcpServer(){
    this.listen = function(address, port, callback){
        _tcpServer.listen(this.socketId, address, port, function(code){
            if(code<0){
                this.error(code);
                return false;
            }
            else{
                _tcpServer.onAccept.addListener(function(info){
                    if(info.socketId==this.socketId){
                        this.accept(info);
                    }
                }.bind(this));
                _tcpServer.onAcceptError.addListener(function(info){
                    if(info.socketId==this.socketId){
                        this.error(info.resultCode);
                    }
                }.bind(this));
                callback();
            }
        }.bind(this));
    }.bind(this),

    this.error = function(code){
        console.log('An error occurred with code '+code);
    },

    this.accept = function(info){
        console.log('New connection.');
    }
} 
```

注意，上面的代码中`_tcpServer.listen`后的参数没有`backlog`，这不是笔误。因为`backlog`是可选参数，如果不指定会由系统自动管理。系统会设定一个保证大多数程序正常运行的值，所以不需要手动设定。

### 9.3.3　获取 socket

**获取指定 socket**

```js
chrome.sockets.tcpServer.getInfo(socketId, function(socketInfo){
    //do something with socketInfo
}); 
```

`socketInfo`是一个描述 socket 信息的对象，包括`socketId`、`name`、`paused`、`persistent`、`localAddress`和`localPort`。

**获取全部活动的 socket**

```js
chrome.sockets.tcpServer.getSockets(function(socketInfoArray){
    //do something with socketInfoArray
}); 
```

`socketInfoArray`是一个包含一个或多个`socketInfo`对象的数组。

将以上方法加入到`tcpServer`类中：

```js
function tcpServer(){
    this.getInfo = function(callback){
        _tcpServer.getInfo(this.socketId, callback);
    }.bind(this),

    this.getSockets = function(callback){
        _tcpServer.getSockets (callback);
    }.bind(this)
} 
```

### 9.3.4　HTTP Server

通过 8.3.2 和 8.3.3 两小节的内容我们了解的 TCP 在 Chrome 应用中的使用，本小节将实战编写一个 HTTP 服务器。

HTTP 服务器需要监听 TCP 连接同时使用 TCP 与客户端进行通信，所以需要`tcp`和`tcpServer`权限：

```js
{
    "app": {
        "background": {
            "scripts": ["tcp.js", "tcpServer.js", "background.js"]
        }
    },
    "manifest_version": 2,
    "name": "HTTP Server",
    "version": "1.0",
    "description": "An HTTP server.",
    "icons": {
        "128": "http_server.png"
    },
    "sockets": {
        "tcp": {
            "connect": "*"
        },
        "tcpServer": {
            "listen": ":80"
        }
    }
} 
```

其中 tcp.js 和 tcpServer.js 就是前两节中我们写的两个类。下面来写 background.js。

首先需要创建`tcpServerSocket`：

```js
var tcpServerSocket = new tcpServer();
tcpServerSocket.option = {
    persistent: true
};
tcpServerSocket.accept = handleAccept.bind(tcpServerSocket);
tcpServerSocket.init(function(){
    tcpServerSocket.listen('127.0.0.1', 80, function(){
        console.log('Listening 127.0.0.1:80...');
    });
}); 
```

创建完成后来编写监听连接的函数`handleAccept`：

```js
function handleAccept(info){
    if(info.socketId==this.socketId){
        var _tcp = chrome.sockets.tcp;
        var tcpSocket = new tcp();
        tcpSocket.socketId = info.clientSocketId;
        tcpSocket.keepAlive(true, 5, function(){
            _tcp.onReceive.addListener(function(info){
                if(info.socketId==tcpSocket.socketId){
                    tcpSocket.receive(info);
                }
            });
            _tcp.onReceiveError.addListener(function(info){
                if(info.socketId==tcpSocket.socketId){
                    tcpSocket.error(info.resultCode);
                }
            });
            tcpSocket.receive = handleRequest.bind(tcpSocket);
            tcpSocket.pause(false, function(){
                console.log('Receiving data...');
            });
        });
    }
} 
```

在`handleAccept`函数中，我们获取到`clientSocketId`后创建了一个新的`tcp`对象`tcpSocket`，并将`clientSocketId`的值赋给了`tcpSocket`的`socketId`，这样对`tcpSocket`的操作就是对`clientSocketId`指向的 TCP socket 了。

之后设定了`keep-alive`属性，并解除了此 TCP socket 的阻止状态开始接收数据。接收到的数据通过`handleRequest`函数来处理。下面来编写`handleRequest`函数：

```js
function handleRequest(info){
    var header = ab2str(info.data);
    header = header.split("\r\n").join('<br />');
    var body = "<h1>It Works!</h1>"+
               "<hr />"+
               "Request Header:<br />"+header;
    var respondse = "HTTP/1.1 200 OK\r\n"+
                    "Connection: Keep-Alive\r\n"+
                    "Content-Length: "+body.length+"\r\n"+
                    "Content-Type: text/html\r\n"+
                    "Connection: close\r\n\r\n"+body;
    respondse = str2ab(respondse);
    this.send(respondse, function(){
        console.log('Sent.');
        this.close(function(){
            console.log('Closed.');
        })
    }.bind(this));
} 
```

`handleRequest`函数将得到的用户请求的 HTTP 头展示给用户，同时在最前端显示“It Works!”。

最后来编写`ArrayBuffer`和字符串直接转换的函数，本例中使用到的转换函数与 8.1.5 节中所使用的略有不同：

```js
function str2ab(str){
    var buf = new ArrayBuffer(str.length);
    bufView = new Uint8Array(buf);
    for(var i=0; i<str.length; i++){
        bufView[i] = str.charCodeAt(i);
    }
    return buf;
}

function ab2str(buf){
    return String.fromCharCode.apply(null, new Uint8Array(buf));
} 
```

在 8.1.5 节中我们使用的是`Uint16Array`这个`ArrayBufferView`，因为在 JavaScript 中一个字符占 16 位，所以应用两个字节来储存，但在 HTTP 协议中一个字符占 8 位，所以要用 1 个字节来储存，这点请读者注意。

![enter image description here](img/00062.jpeg)
*HTTP Server 运行截图*

本节讲到的应用源码可以通过[`github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/http_server`](https://github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/http_server)下载。

## 9.4　WebSocket

WebSocket 与本章前三节介绍的内容不同，它是 HTML5 原生支持的功能。使用 WebSocket 需要外部服务器的支持。

在使用 WebSocket 通信前需要先连接到外部的 WebSocket 服务器：

```js
var connection = new WebSocket('ws://127.0.0.1'); 
```

当连接打开后会触发`onopen`事件：

```js
connection.onopen = function(){
    //do something when the connection is open
} 
```

向服务器发送数据使用`send`方法：

```js
connection.send(data); 
```

其中`data`可以是`ArrayBuffer`、`Blob`或字符串。

当接收到来自服务器的数据时会触发`onmessage`事件：

```js
connection.onmessage = function(result){
    //do something with result
} 
```

其中`result`是一个对象，可以通过`result.data`访问数据。如果传送的数据是二进制数据，还可以通过`result.data.byteLength`和`result.data.binaryType`获取二进制数据的长度和类型（`ArrayBuffer`或`Blob`）。

当 WebSocket 连接发生异常时会触发`onerror`事件：

```js
connection.onerror = function(error){
    console.log(error);
}; 
```

更多有关 WebSocket 的内容可以参见[`www.html5rocks.com/zh/tutorials/websockets/basics/`](http://www.html5rocks.com/zh/tutorials/websockets/basics/)。