# 13.2 CommonJS 规范

*   概述
*   module 对象
    *   module.exports 属性
    *   exports 变量
    *   AMD 规范与 CommonJS 规范的兼容性
    *   require 命令
        *   基本用法
        *   加载规则
        *   模块的缓存
        *   模块的循环加载
        *   require.main
    *   参考链接

## 概述

CommonJS 是服务器模块的规范，Node.js 采用了这个规范。

根据 CommonJS 规范，一个单独的文件就是一个模块。每一个模块都是一个单独的作用域，也就是说，在一个文件定义的变量（还包括函数和类），都是私有的，对其他文件是不可见的。

```js
var x = 5;
var addX = function(value) {
  return value + x;
};
```

上面代码中，变量 x 和函数 addX，是当前文件私有的，其他文件不可见。

如果想在多个文件分享变量，必须定义为 global 对象的属性。

```js
global.warning = true;
```

上面代码的 waining 变量，可以被所有文件读取。当然，这样写法是不推荐的。

CommonJS 规定，每个文件的对外接口是 module.exports 对象。这个对象的所有属性和方法，都可以被其他文件导入。

```js
var x = 5;
var addX = function(value) {
  return value + x;
};
module.exports.x = x;
module.exports.addX = addX;
```

上面代码通过 module.exports 对象，定义对外接口，输出变量 x 和函数 addX。module.exports 对象是可以被其他文件导入的，它其实就是文件内部与外部通信的桥梁。

require 方法用于在其他文件加载这个接口，具体用法参见《Require 命令》的部分。

```js
var example = require('./example.js');

console.log(example.x); // 5
console.log(addX(1)); // 6
```

## module 对象

每个模块都有一个 module 变量，该变量指向当前模块。module 不是全局变量，而是每个模块都有的本地变量。

*   module.id 模块的识别符，通常是带有绝对路径的模块文件名。
*   module.filename 模块的文件名。
*   module.loaded 返回一个布尔值，表示模块是否已经完成加载。
*   module.parent 返回一个对象，表示调用该模块的模块。
*   module.children 返回一个数组，表示该模块要用到的其他模块。

下面是一个示例文件，最后一行输出 module 变量。

```js
// example.js
var jquery = require('jquery');
exports.$ = jquery;
console.log(module);
```

执行这个文件，命令行会输出如下信息。

```js
{ id: '.',
  exports: { '/div>: [Function] },
  parent: null,
  filename: '/path/to/example.js',
  loaded: false,
  children:
   [ { id: '/path/to/node_modules/jquery/dist/jquery.js',
       exports: [Function],
       parent: [Circular],
       filename: '/path/to/node_modules/jquery/dist/jquery.js',
       loaded: true,
       children: [],
       paths: [Object] } ],
  paths:
   [ '/home/user/deleted/node_modules',
     '/home/user/node_modules',
     '/home/node_modules',
     '/node_modules' ]
}
```

### module.exports 属性

module.exports 属性表示当前模块对外输出的接口，其他文件加载该模块，实际上就是读取 module.exports 变量。

```js
var EventEmitter = require('events').EventEmitter;
module.exports = new EventEmitter();

setTimeout(function() {
  module.exports.emit('ready');
}, 1000);
```

上面模块会在加载后 1 秒后，发出 ready 事件。其他文件监听该事件，可以写成下面这样。

```js
var a = require('./a');
a.on('ready', function() {
  console.log('module a is ready');
});
```

### exports 变量

为了方便，Node 为每个模块提供一个 exports 变量，指向 module.exports。这等同在每个模块头部，有一行这样的命令。

```js
var exports = module.exports;
```

造成的结果是，在对外输出模块接口时，可以向 exports 对象添加方法。

```js
exports.area = function (r) {
  return Math.PI * r * r;
};

exports.circumference = function (r) {
  return 2 * Math.PI * r;
};
```

注意，不能直接将 exports 变量指向一个函数。因为这样等于切断了 exports 与 module.exports 的联系。

```js
exports = function (x){ console.log(x);};
```

上面这样的写法是无效的，因为它切断了 exports 与 module.exports 之间的链接。

下面的写法也是无效的。

```js
exports.hello = function() {
  return 'hello';
};

module.exports = 'Hello world';
```

上面代码中，hello 函数是无法对外输出的，因为`module.exports`被重新赋值了。

如果一个模块的对外接口，就是一个函数或对象时，不能使用 exports 输出，只能使用 module.exports 输出。

```js
module.exports = function (x){ console.log(x);};
```

如果你觉得，exports 与 module.exports 之间的区别很难分清，一个简单的处理方法，就是放弃使用 exports，只使用 module.exports。

## AMD 规范与 CommonJS 规范的兼容性

CommonJS 规范加载模块是同步的，也就是说，只有加载完成，才能执行后面的操作。AMD 规范则是非同步加载模块，允许指定回调函数。由于 Node.js 主要用于服务器编程，模块文件一般都已经存在于本地硬盘，所以加载起来比较快，不用考虑非同步加载的方式，所以 CommonJS 规范比较适用。但是，如果是浏览器环境，要从服务器端加载模块，这时就必须采用非同步模式，因此浏览器端一般采用 AMD 规范。

AMD 规范使用 define 方法定义模块，下面就是一个例子：

```js
define(['package/lib'], function(lib){
  function foo(){
    lib.log('hello world!');
  }

  return {
    foo: foo
  };
});
```

AMD 规范允许输出的模块兼容 CommonJS 规范，这时 define 方法需要写成下面这样：

```js
define(function (require, exports, module){
  var someModule = require("someModule");
  var anotherModule = require("anotherModule");

  someModule.doTehAwesome();
  anotherModule.doMoarAwesome();

  exports.asplode = function (){
    someModule.doTehAwesome();
    anotherModule.doMoarAwesome();
  };
});
```

## require 命令

### 基本用法

Node.js 使用 CommonJS 模块规范，内置的 require 命令用于加载模块文件。

require 命令的基本功能是，读入并执行一个 JavaScript 文件，然后返回该模块的 exports 对象。如果没有发现指定模块，会报错。

```js
// example.js
var invisible = function () {
  console.log("invisible");
}

exports.message = "hi";

exports.say = function () {
  console.log(message);
}
```

运行下面的命令，可以输出 exports 对象。

```js
var example = require('./example.js');
example
// {
//   message: "hi",
//   say: [Function]
// }
```

如果模块输出的是一个函数，那就不能定义在 exports 对象上面，而要定义在`module.exports`变量上面。

```js
module.exports = function () {
  console.log("hello world")
}

require('./example2.js')()
```

上面代码中，require 命令调用自身，等于是执行`module.exports`，因此会输出 hello world。

### 加载规则

require 命令接受模块名作为参数。

（1）如果参数字符串以“/”开头，则表示加载的是一个位于绝对路径的模块文件。比如，`require('/home/marco/foo.js')`将加载/home/marco/foo.js。

（2）如果参数字符串以“./”开头，则表示加载的是一个位于相对路径（跟当前执行脚本的位置相比）的模块文件。比如，`require('./circle')`将加载当前脚本同一目录的 circle.js。

（3）如果参数字符串不以“./“或”/“开头，则表示加载的是一个默认提供的核心模块（位于 Node 的系统安装目录中），或者一个位于各级 node_modules 目录的已安装模块（全局安装或局部安装）。

举例来说，脚本`/home/user/projects/foo.js`执行了`require('bar.js')`命令，Node 会依次搜索以下文件。

*   /home/user/projects/node_modules/bar.js
*   /home/user/node_modules/bar.js
*   /home/node_modules/bar.js
*   /node_modules/bar.js

这样设计的目的是，使得不同的模块可以将所依赖的模块本地化。

（4）如果传入 require 方法的是一个目录，那么 require 会先查看该目录的 package.json 文件，然后加载 main 字段指定的脚本文件。否则取不到 main 字段，则会加载`index.js`文件或`index.node`文件。

举例来说，下面是一行普通的 require 命令语句。

```js
var utils = require( "utils" );
```

Node 寻找 utils 脚本的顺序是，首先寻找核心模块，然后是全局安装模块，接着是项目安装的模块。

```js
[
  '/usr/local/lib/node',
  '~/.node_modules',
  './node_modules/utils.js',
  './node_modules/utils/package.json',
  './node_modules/utils/index.js'
]
```

（5）如果指定的模块文件没有发现，Node 会尝试为文件名添加.js、.json、.node 后，再去搜索。.js 文件会以文本格式的 JavaScript 脚本文件解析，.json 文件会以 JSON 格式的文本文件解析，.node 文件会议编译后二进制文件解析。

（6）如果想得到 require 命令加载的确切文件名，使用 require.resolve()方法。

### 模块的缓存

第一次加载某个模块时，Node 会缓存该模块。以后再加载该模块，就直接从缓存取出该模块的 exports 属性。

```js
require('./example.js');
require('./example.js').message = "hello";
require('./example.js').message
// "hello"
```

上面代码中，连续三次使用 require 命令，加载同一个模块。第二次加载的时候，为输出的对象添加了一个 message 属性。但是第三次加载的时候，这个 message 属性依然存在，这就证明 require 命令并没有重新加载模块文件，而是输出了缓存。

如果想要多次执行某个模块，可以输出一个函数，然后多次调用这个函数。

缓存是根据绝对路径识别模块的，如果同样的模块名，但是保存在不同的路径，require 命令还是会重新加载该模块。

### 模块的循环加载

如果发生模块的循环加载，即 A 加载 B，B 又加载 A，则 B 将加载 A 的不完整版本。

```js
// a.js
exports.x = 'a1';
console.log('a.js ', require('./b.js').x);
exports.x = 'a2';

// b.js
exports.x = 'b1';
console.log('b.js ', require('./a.js').x);
exports.x = 'b2';

// main.js
console.log('main.js ', require('./a.js').x);
console.log('main.js ', require('./b.js').x);
```

上面代码是三个 JavaScript 文件。其中，a.js 加载了 b.js，而 b.js 又加载 a.js。这时，Node 返回 a.js 的不完整版本，所以执行结果如下。

```js
$ node main.js
b.js  a1
a.js  b2
main.js  a2
main.js  b2
```

修改 main.js，再次加载 a.js 和 b.js。

```js
// main.js
console.log('main.js ', require('./a.js').x);
console.log('main.js ', require('./b.js').x);
console.log('main.js ', require('./a.js').x);
console.log('main.js ', require('./b.js').x);
```

执行上面代码，结果如下。

```js
$ node main.js
b.js  a1
a.js  b2
main.js  a2
main.js  b2
main.js  a2
main.js  b2
```

上面代码中，第二次加载 a.js 和 b.js 时，会直接从缓存读取 exports 属性，所以 a.js 和 b.js 内部的 console.log 语句都不会执行了。

### require.main

正常的脚本调用时，require.main 属性指向模块本身。

```js
require.main === module
// true
```

如果是在 REPL 环境使用 require 命令，则上面的表达式返回 false。

通过 require.main 属性，可以获取模块的信息。比如，module 对象有一个 filename 属性（正常情况下等于 __filename），可以通过 require.main.filename 属性，得知当前模块的入口文件。

## 参考链接

*   Addy Osmani, [Writing Modular JavaScript With AMD, CommonJS & ES Harmony](http://addyosmani.com/writing-modular-js/)
*   Pony Foo, [A Gentle Browserify Walkthrough](http://blog.ponyfoo.com/2014/08/25/a-gentle-browserify-walkthrough)
*   Nico Reed, [What is require?]（[`docs.nodejitsu.com/articles/getting-started/what-is-require`](https://docs.nodejitsu.com/articles/getting-started/what-is-require)）