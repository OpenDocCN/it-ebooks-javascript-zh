# 13.9 Events 模块

*   概述
    *   基本用法
    *   on 方法
    *   emit 方法
    *   EventEmitter 接口的部署
    *   事件类型
    *   EventEmitter 实例的方法
        *   once 方法
        *   removeListener 方法
    *   参考链接

## 概述

### 基本用法

Events 模块是 node.js 对“发布/订阅”模式（publish/subscribe）的部署。一个对象通过这个模块，向另一个对象传递消息。该模块通过 EventEmitter 属性，提供了一个构造函数。该构造函数的实例具有 on 方法，可以用来监听指定事件，并触发回调函数。任意对象都可以发布指定事件，被 EventEmitter 实例的 on 方法监听到。

下面是一个实例，先建立一个消息中心，然后通过 on 方法，为各种事件指定回调函数，从而将程序转为事件驱动型，各个模块之间通过事件联系。

```js
var EventEmitter = require("events").EventEmitter;

var ee = new EventEmitter();
ee.on("someEvent", function () {
  console.log("event has occured");
});

ee.emit("someEvent");
```

上面代码在加载 events 模块后，通过 EventEmitter 属性建立了一个 EventEmitter 对象实例，这个实例就是消息中心。然后，通过 on 方法为 someEvent 事件指定回调函数。最后，通过 emit 方法触发 someEvent 事件。

### on 方法

默认情况下，Node.js 允许同一个事件最多可以指定 10 个回调函数。

```js
ee.on("someEvent", function () { console.log("event 1"); });
ee.on("someEvent", function () { console.log("event 2"); });
ee.on("someEvent", function () { console.log("event 3"); });
```

超过 10 个回调函数，会发出一个警告。这个门槛值可以通过 setMaxListeners 方法改变。

```js
ee.setMaxListeners(20);
```

### emit 方法

EventEmitter 实例的 emit 方法，用来触发事件。它的第一个参数是事件名称，其余参数都会依次传入回调函数。

```js
var EventEmitter = require('events').EventEmitter;
var myEmitter = new EventEmitter;

var connection = function(id){
  console.log('client id: ' + id);
};

myEmitter.on('connection', connection);
myEmitter.emit('connection', 6);
```

## EventEmitter 接口的部署

Events 模块的作用，还在于其他模块可以部署 EventEmitter 接口，从而也能够订阅和发布消息。

```js
var EventEmitter = require('events').EventEmitter;

function Dog(name) {
  this.name = name;
}

Dog.prototype.__proto__ = EventEmitter.prototype;
// 另一种写法
// Dog.prototype = Object.create(EventEmitter.prototype);

var simon = new Dog('simon');

simon.on('bark', function(){
  console.log(this.name + ' barked');
});

setInterval(function(){
  simon.emit('bark');
}, 500);
```

上面代码新建了一个构造函数 Dog，然后让其继承 EventEmitter，因此 Dog 就拥有了 EventEmitter 的接口。最后，为 Dog 的实例指定 bark 事件的监听函数，再使用 EventEmitter 的 emit 方法，触发 bark 事件。

Node 内置模块 util 的 inherits 方法，提供了另一种继承 EventEmitter 的写法。

```js
var util = require('util');
var EventEmitter = require('events').EventEmitter;

var Radio = function(station) {

    var self = this;

    setTimeout(function() {
        self.emit('open', station);
    }, 0);

    setTimeout(function() {
        self.emit('close', station);
    }, 5000);

    this.on('newListener', function(listener) {
        console.log('Event Listener: ' + listener);
    });

};

util.inherits(Radio, EventEmitter);

module.exports = Radio;
```

上面代码中，Radio 是一个构造函数，它的实例继承了 EventEmitter 接口。下面是使用这个模块的例子。

```js
var Radio = require('./radio.js');

var station = {
  freq: '80.16',
  name: 'Rock N Roll Radio',
};

var radio = new Radio(station);

radio.on('open', function(station) {
  console.log('"%s" FM %s 打开', station.name, station.freq);
  console.log('♬ ♫♬');
});

radio.on('close', function(station) {
  console.log('"%s" FM %s 关闭', station.name, station.freq);
});
```

## 事件类型

Events 模块默认支持两个事件。

*   newListener 事件：添加新的回调函数时触发。
*   removeListener 事件：移除回调时触发。

```js
ee.on("newListener", function (evtName){
  console.log("New Listener: " + evtName);
});

ee.on("removeListener", function (evtName){
  console.log("Removed Listener: " + evtName);
});

function foo (){}

ee.on("save-user", foo);
ee.removeListener("save-user", foo);

// New Listener: removeListener
// New Listener: save-user
// Removed Listener: save-user
```

上面代码会触发两次 newListener 事件，以及一次 removeListener 事件。

## EventEmitter 实例的方法

### once 方法

该方法类似于 on 方法，但是回调函数只触发一次。

```js
var EventEmitter = require('events').EventEmitter;
var myEmitter = new EventEmitter;

myEmitter.once('message', function(msg){
  console.log('message: ' + msg);
});

myEmitter.emit('message', 'this is the first message');
myEmitter.emit('message', 'this is the second message');
myEmitter.emit('message', 'welcome to nodejs');
```

上面代码触发了三次 message 事件，但是回调函数只会在第一次调用时运行。

下面代码指定，一旦服务器连通，只调用一次的回调函数。

```js
server.once('connection', function (stream) {
  console.log('Ah, we have our first user!');
});
```

该方法返回一个 EventEmitter 对象，因此可以链式加载监听函数。

### removeListener 方法

该方法用于移除回调函数。它接受两个参数，第一个是事件名称，第二个是回调函数名称。这就是说，不能用于移除匿名函数。

```js
var EventEmitter = require('events').EventEmitter;

var emitter = new EventEmitter;

emitter.on('message', console.log);

setInterval(function(){
  emitter.emit('message', 'foo bar');
}, 300);

setTimeout(function(){
  emitter.removeListener('message', console.log);
}, 1000);
```

上面代码每 300 毫秒触发一次 message 事件，直到 1000 毫秒后取消监听。

另一个例子是使用 removeListener 方法模拟 once 方法。

```js
var EventEmitter = require('events').EventEmitter;

var emitter = new EventEmitter;

function onlyOnce () {
    console.log("You'll never see this again");
    emitter.removeListener("firstConnection", onlyOnce);
}

emitter.on("firstConnection", onlyOnce);
```

（3）removeAllListeners 方法

该方法用于移除某个事件的所有回调函数。

```js
var EventEmitter = require('events').EventEmitter;

var emitter = new EventEmitter;

// some code here

emitter.removeAllListeners("firstConnection");
```

如果不带参数，则表示移除所有事件的所有回调函数。

```js
emitter.removeAllListeners();
```

（4）listener 方法

该方法接受一个事件名称作为参数，返回该事件所有回调函数组成的数组。

```js
var EventEmitter = require('events').EventEmitter;

var ee = new EventEmitter;

function onlyOnce () {
    console.log(ee.listeners("firstConnection"));
    ee.removeListener("firstConnection", onlyOnce);
    console.log(ee.listeners("firstConnection"));
}

ee.on("firstConnection", onlyOnce)
ee.emit("firstConnection");
ee.emit("firstConnection");

// [ [Function: onlyOnce] ]
// []
```

上面代码显示两次回调函数组成的数组，第一次只有一个回调函数 onlyOnce，第二次是一个空数组，因为 removeListener 方法取消了回调函数。

## 参考链接

*   Hage Yaapa, [Node.js EventEmitter Tutorial](http://www.hacksparrow.com/node-js-eventemitter-tutorial.html)