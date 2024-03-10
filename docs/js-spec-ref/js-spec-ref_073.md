# 8.7 RequireJS 和 AMD 规范

*   概述
    *   define 方法：定义模块
    *   require 方法：调用模块
    *   AMD 模式小结
    *   配置 require.js：config 方法
    *   插件
    *   优化器 r.js
    *   参考链接

## 概述

RequireJS 是一个工具库，主要用于客户端的模块管理。它可以让客户端的代码分成一个个模块，实现异步或动态加载，从而提高代码的性能和可维护性。它的模块管理遵守[AMD 规范](https://github.com/amdjs/amdjs-api/wiki/AMD)（Asynchronous Module Definition）。

RequireJS 的基本思想是，通过 define 方法，将代码定义为模块；通过 require 方法，实现代码的模块加载。

首先，将 require.js 嵌入网页，然后就能在网页中进行模块化编程了。

```
<script data-main="scripts/main" src="scripts/require.js"></script>
```

上面代码的 data-main 属性不可省略，用于指定主代码所在的脚本文件，在上例中为 scripts 子目录下的 main.js 文件。用户自定义的代码就放在这个 main.js 文件中。

### define 方法：定义模块

define 方法用于定义模块，RequireJS 要求每个模块放在一个单独的文件里。

按照是否依赖其他模块，可以分成两种情况讨论。第一种情况是定义独立模块，即所定义的模块不依赖其他模块；第二种情况是定义非独立模块，即所定义的模块依赖于其他模块。

（1）独立模块

如果被定义的模块是一个独立模块，不需要依赖任何其他模块，可以直接用 define 方法生成。

```
define({
    method1: function() {},
    method2: function() {},
});
```

上面代码生成了一个拥有 method1、method2 两个方法的模块。

另一种等价的写法是，把对象写成一个函数，该函数的返回值就是输出的模块。

```
define(function () {
    return {
        method1: function() {},
        method2: function() {},
    };
});
```

后一种写法的自由度更高一点，可以在函数体内写一些模块初始化代码。

值得指出的是，define 定义的模块可以返回任何值，不限于对象。

（2）非独立模块

如果被定义的模块需要依赖其他模块，则 define 方法必须采用下面的格式。

```
define(['module1', 'module2'], function(m1, m2) {
   ...
});
```

define 方法的第一个参数是一个数组，它的成员是当前模块所依赖的模块。比如，['module1', 'module2']表示我们定义的这个新模块依赖于 module1 模块和 module2 模块，只有先加载这两个模块，新模块才能正常运行。一般情况下，module1 模块和 module2 模块指的是，当前目录下的 module1.js 文件和 module2.js 文件，等同于写成['./module1', './module2']。

define 方法的第二个参数是一个函数，当前面数组的所有成员加载成功后，它将被调用。它的参数与数组的成员一一对应，比如 function(m1, m2)就表示，这个函数的第一个参数 m1 对应 module1 模块，第二个参数 m2 对应 module2 模块。这个函数必须返回一个对象，供其他模块调用。

```
define(['module1', 'module2'], function(m1, m2) {

    return {
        method: function() {
            m1.methodA();
            m2.methodB();
        }
    };

});
```

上面代码表示新模块返回一个对象，该对象的 method 方法就是外部调用的接口，menthod 方法内部调用了 m1 模块的 methodA 方法和 m2 模块的 methodB 方法。

需要注意的是，回调函数必须返回一个对象，这个对象就是你定义的模块。

如果依赖的模块很多，参数与模块一一对应的写法非常麻烦。

```
define(
    [       'dep1', 'dep2', 'dep3', 'dep4', 'dep5', 'dep6', 'dep7', 'dep8'],
    function(dep1,   dep2,   dep3,   dep4,   dep5,   dep6,   dep7,   dep8){
        ...
    }
);
```

为了避免像上面代码那样繁琐的写法，RequireJS 提供一种更简单的写法。

```
define(
    function (require) {
        var dep1 = require('dep1'),
            dep2 = require('dep2'),
            dep3 = require('dep3'),
            dep4 = require('dep4'),
            dep5 = require('dep5'),
            dep6 = require('dep6'),
            dep7 = require('dep7'),
            dep8 = require('dep8');

            ...
    }

});
```

下面是一个 define 实际运用的例子。

```
define(['math', 'graph'], 
    function ( math, graph ) {
        return {
            plot: function(x, y){
                return graph.drawPie(math.randomGrid(x,y));
            }
        }
    };
);
```

上面代码定义的模块依赖 math 和 graph 两个库，然后返回一个具有 plot 接口的对象。

另一个实际的例子是，通过判断浏览器是否为 IE，而选择加载 zepto 或 jQuery。

```
define(('__proto__' in {} ? ['zepto'] : ['jquery']), function($) {
    return $;
});
```

上面代码定义了一个中间模块，该模块先判断浏览器是否支持 proto 属性（除了 IE，其他浏览器都支持），如果返回 true，就加载 zepto 库，否则加载 jQuery 库。

### require 方法：调用模块

require 方法用于调用模块。它的参数与 define 方法类似。

```
require(['foo', 'bar'], function ( foo, bar ) {
        foo.doSomething();
});
```

上面方法表示加载 foo 和 bar 两个模块，当这两个模块都加载成功后，执行一个回调函数。该回调函数就用来完成具体的任务。

require 方法的第一个参数，是一个表示依赖关系的数组。这个数组可以写得很灵活，请看下面的例子。

```
require( [ window.JSON ? undefined : 'util/json2' ], function ( JSON ) {
  JSON = JSON || window.JSON;

  console.log( JSON.parse( '{ "JSON" : "HERE" }' ) );
});
```

上面代码加载 JSON 模块时，首先判断浏览器是否原生支持 JSON 对象。如果是的，则将 undefined 传入回调函数，否则加载 util 目录下的 json2 模块。

require 方法也可以用在 define 方法内部。

```
define(function (require) {
   var otherModule = require('otherModule');
});
```

下面的例子显示了如何动态加载模块。

```
define(function ( require ) {
    var isReady = false, foobar;

    require(['foo', 'bar'], function (foo, bar) {
        isReady = true;
        foobar = foo() + bar();
    });

    return {
        isReady: isReady,
        foobar: foobar
    };
});
```

上面代码所定义的模块，内部加载了 foo 和 bar 两个模块，在没有加载完成前，isReady 属性值为 false，加载完成后就变成了 true。因此，可以根据 isReady 属性的值，决定下一步的动作。

下面的例子是模块的输出结果是一个 promise 对象。

```
define(['lib/Deferred'], function( Deferred ){
    var defer = new Deferred(); 
    require(['lib/templates/?index.html','lib/data/?stats'],
        function( template, data ){
            defer.resolve({ template: template, data:data });
        }
    );
    return defer.promise();
});
```

上面代码的 define 方法返回一个 promise 对象，可以在该对象的 then 方法，指定下一步的动作。

如果服务器端采用 JSONP 模式，则可以直接在 require 中调用，方法是指定 JSONP 的 callback 参数为 define。

```
require( [ 
    "http://someapi.com/foo?callback=define"
], function (data) {
    console.log(data);
});
```

require 方法允许添加第三个参数，即错误处理的回调函数。

```
require(
    [ "backbone" ], 
    function ( Backbone ) {
        return Backbone.View.extend({ /* ... */ });
    }, 
    function (err) {
        // ...
    }
);
```

require 方法的第三个参数，即处理错误的回调函数，接受一个 error 对象作为参数。

require 对象还允许指定一个全局性的 Error 事件的监听函数。所有没有被上面的方法捕获的错误，都会被触发这个监听函数。

```
requirejs.onError = function (err) {
    // ...
};
```

### AMD 模式小结

define 和 require 这两个定义模块、调用模块的方法，合称为 AMD 模式。它的模块定义的方法非常清晰，不会污染全局环境，能够清楚地显示依赖关系。

AMD 模式可以用于浏览器环境，并且允许非同步加载模块，也可以根据需要动态加载模块。

## 配置 require.js：config 方法

require 方法本身也是一个对象，它带有一个 config 方法，用来配置 require.js 运行参数。config 方法接受一个对象作为参数。

```
require.config({
    paths: {
        jquery: [
            '//cdnjs.cloudflare.com/ajax/libs/jquery/2.0.0/jquery.min.js',
            'lib/jquery'
        ]
    }
});
```

config 方法的参数对象有以下主要成员：

（1）paths

paths 参数指定各个模块的位置。这个位置可以是同一个服务器上的相对位置，也可以是外部网址。可以为每个模块定义多个位置，如果第一个位置加载失败，则加载第二个位置，上面的示例就表示如果 CDN 加载失败，则加载服务器上的备用脚本。需要注意的是，指定本地文件路径时，可以省略文件最后的 js 后缀名。

```
require(["jquery"], function($) {
    // ...
});
```

上面代码加载 jquery 模块，因为 jquery 的路径已经在 paths 参数中定义了，所以就会到事先设定的位置下载。

（2）baseUrl

baseUrl 参数指定本地模块位置的基准目录，即本地模块的路径是相对于哪个目录的。该属性通常由 require.js 加载时的 data-main 属性指定。

（3）shim

有些库不是 AMD 兼容的，这时就需要指定 shim 属性的值。shim 可以理解成“垫片”，用来帮助 require.js 加载非 AMD 规范的库。

```
require.config({
    paths: {
        "backbone": "vendor/backbone",
        "underscore": "vendor/underscore"
    },
    shim: {
        "backbone": {
            deps: [ "underscore" ],
            exports: "Backbone"
        },
        "underscore": {
            exports: "_"
        }
    }
});
```

上面代码中的 backbone 和 underscore 就是非 AMD 规范的库。shim 指定它们的依赖关系（backbone 依赖于 underscore），以及输出符号（backbone 为“Backbone”，underscore 为“_”）。

## 插件

RequireJS 允许使用插件，加载各种格式的数据。完整的插件清单可以查看[官方网站](https://github.com/jrburke/requirejs/wiki/Plugins)。

下面是插入文本数据所使用的 text 插件的例子。

```
define([
    'backbone',
    'text!templates.html'
], function( Backbone, template ){
   // ...
});
```

上面代码加载的第一个模块是 backbone，第二个模块则是一个文本，用'text!'表示。该文本作为字符串，存放在回调函数的 template 变量中。

## 优化器 r.js

RequireJS 提供一个基于 node.js 的命令行工具 r.js，用来压缩多个 js 文件。它的主要作用是将多个模块文件压缩合并成一个脚本文件，以减少网页的 HTTP 请求数。

第一步是安装 r.js（假设已经安装了 node.js）。

```
npm install -g requirejs
```

然后，使用的时候，直接在命令行键入以下格式的命令。

```
node r.js -o <arguments>
```

表示命令运行时，所需要的一系列参数，比如像下面这样：

```
node r.js -o baseUrl=. name=main out=main-built.js
```

除了直接在命令行提供参数设置，也可以将参数写入一个文件，假定文件名为 build.js。

```
({
    baseUrl: ".",
    name: "main",
    out: "main-built.js"
})
```

然后，在命令行下用 r.js 运行这个参数文件，就 OK 了，不需要其他步骤了。

```
node r.js -o build.js
```

下面是一个参数文件的范例，假定位置就在根目录下，文件名为 build.js。

```
({
    appDir: './',
    baseUrl: './js',
    dir: './dist',
    modules: [
        {
            name: 'main'
        }
    ],
    fileExclusionRegExp: /^(r|build)\.js$/,
    optimizeCss: 'standard',
    removeCombined: true,
    paths: {
        jquery: 'lib/jquery',
        underscore: 'lib/underscore',
        backbone: 'lib/backbone/backbone',
        backboneLocalstorage: 'lib/backbone/backbone.localStorage',
        text: 'lib/require/text'
    },
    shim: {
        underscore: {
            exports: '_'
        },
        backbone: {
            deps: [
                'underscore',
                'jquery'
            ],
            exports: 'Backbone'
        },
        backboneLocalstorage: {
            deps: ['backbone'],
            exports: 'Store'
        }
    }
})
```

上面代码将多个模块压缩合并成一个 main.js。

参数文件的主要成员解释如下：

*   appDir：项目目录，相对于参数文件的位置。

*   baseUrl：js 文件的位置。

*   dir：输出目录。

*   modules：一个包含对象的数组，每个对象就是一个要被优化的模块。

*   fileExclusionRegExp：凡是匹配这个正则表达式的文件名，都不会被拷贝到输出目录。

*   optimizeCss: 自动压缩 CSS 文件，可取的值包括“none”, “standard”, “standard.keepLines”, “standard.keepComments”, “standard.keepComments.keepLines”。

*   removeCombined：如果为 true，合并后的原文件将不保留在输出目录中。

*   paths：各个模块的相对路径，可以省略 js 后缀名。

*   shim：配置依赖性关系。如果某一个模块不是 AMD 模式定义的，就可以用 shim 属性指定模块的依赖性关系和输出值。

*   generateSourceMaps：是否要生成 source map 文件。

更详细的解释可以参考[官方文档](https://github.com/jrburke/r.js/blob/master/build/example.build.js)。

运行优化命令后，可以前往 dist 目录查看优化后的文件。

下面是另一个 build.js 的例子。

```
({
    mainConfigFile : "js/main.js",
    baseUrl: "js",
    removeCombined: true,
    findNestedDependencies: true,
    dir: "dist",
    modules: [
        {
            name: "main",
            exclude: [
                "infrastructure"
            ]
        },
        {
            name: "infrastructure"
        }
    ]
})
```

上面代码将模块文件压缩合并成两个文件，第一个是 main.js（指定排除 infrastructure.js），第二个则是 infrastructure.js。

## 参考链接

*   NaorYe, [Optimize (Concatenate and Minify) RequireJS Projects](http://www.webdeveasy.com/optimize-requirejs-projects/)
*   Jonathan Creamer, [Deep dive into Require.js](http://tech.pro/tutorial/1300/deep-dive-into-requirejs)
*   Addy Osmani, [Writing Modular JavaScript With AMD, CommonJS & ES Harmony](http://addyosmani.com/writing-modular-js/)
*   Jim Cowart, [Five Helpful Tips When Using RequireJS](http://tech.pro/blog/1561/five-helpful-tips-when-using-requirejs)
*   Jim Cowart, [Using r.js to Optimize Your RequireJS Project](http://tech.pro/blog/1639/using-rjs-to-optimize-your-requirejs-project)