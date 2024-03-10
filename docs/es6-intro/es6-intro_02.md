# ECMAScript 6 简介

ECMAScript 6（以下简称 ES6）是 JavaScript 语言的下一代标准，已经在 2015 年 6 月正式发布了。Mozilla 公司将在这个标准的基础上，推出 JavaScript 2.0。

ES6 的目标，是使得 JavaScript 语言可以用来编写大型的复杂的应用程序，成为企业级开发语言。

标准的制定者有计划，以后每年发布一次标准，使用年份作为标准的版本。因为当前版本的 ES6 是在 2015 年发布的，所以又称 ECMAScript 2015。

## ECMAScript 和 JavaScript 的关系

很多初学者感到困惑：ECMAScript 和 JavaScript 到底是什么关系？

简单说，ECMAScript 是 JavaScript 语言的国际标准，JavaScript 是 ECMAScript 的实现。

要讲清楚这个问题，需要回顾历史。1996 年 11 月，JavaScript 的创造者 Netscape 公司，决定将 JavaScript 提交给国际标准化组织 ECMA，希望这种语言能够成为国际标准。次年，ECMA 发布 262 号标准文件（ECMA-262）的第一版，规定了浏览器脚本语言的标准，并将这种语言称为 ECMAScript。这个版本就是 ECMAScript 1.0 版。

之所以不叫 JavaScript，有两个原因。一是商标，Java 是 Sun 公司的商标，根据授权协议，只有 Netscape 公司可以合法地使用 JavaScript 这个名字，且 JavaScript 本身也已经被 Netscape 公司注册为商标。二是想体现这门语言的制定者是 ECMA，不是 Netscape，这样有利于保证这门语言的开放性和中立性。因此，ECMAScript 和 JavaScript 的关系是，前者是后者的规格，后者是前者的一种实现。在日常场合，这两个词是可以互换的。

## ECMAScript 的历史

1998 年 6 月，ECMAScript 2.0 版发布。

1999 年 12 月，ECMAScript 3.0 版发布，成为 JavaScript 的通行标准，得到了广泛支持。

2007 年 10 月，ECMAScript 4.0 版草案发布，对 3.0 版做了大幅升级，预计次年 8 月发布正式版本。草案发布后，由于 4.0 版的目标过于激进，各方对于是否通过这个标准，发生了严重分歧。以 Yahoo、Microsoft、Google 为首的大公司，反对 JavaScript 的大幅升级，主张小幅改动；以 JavaScript 创造者 Brendan Eich 为首的 Mozilla 公司，则坚持当前的草案。

2008 年 7 月，由于对于下一个版本应该包括哪些功能，各方分歧太大，争论过于激进，ECMA 开会决定，中止 ECMAScript 4.0 的开发，将其中涉及现有功能改善的一小部分，发布为 ECMAScript 3.1，而将其他激进的设想扩大范围，放入以后的版本，由于会议的气氛，该版本的项目代号起名为 Harmony（和谐）。会后不久，ECMAScript 3.1 就改名为 ECMAScript 5。

2009 年 12 月，ECMAScript 5.0 版正式发布。Harmony 项目则一分为二，一些较为可行的设想定名为 JavaScript.next 继续开发，后来演变成 ECMAScript 6；一些不是很成熟的设想，则被视为 JavaScript.next.next，在更远的将来再考虑推出。

2011 年 6 月，ECMAscript 5.1 版发布，并且成为 ISO 国际标准（ISO/IEC 16262:2011）。

2013 年 3 月，ECMAScript 6 草案冻结，不再添加新功能。新的功能设想将被放到 ECMAScript 7。

2013 年 12 月，ECMAScript 6 草案发布。然后是 12 个月的讨论期，听取各方反馈。

2015 年 6 月，ECMAScript 6 正式通过，成为国际标准。

ECMA 的第 39 号技术专家委员会（Technical Committee 39，简称 TC39）负责制订 ECMAScript 标准，成员包括 Microsoft、Mozilla、Google 等大公司。TC39 的总体考虑是，ES5 与 ES3 基本保持兼容，较大的语法修正和新功能加入，将由 JavaScript.next 完成。当时，JavaScript.next 指的是 ES6，第六版发布以后，就指 ES7。TC39 的判断是，ES5 会在 2013 年的年中成为 JavaScript 开发的主流标准，并在此后五年中一直保持这个位置。

## 部署进度

各大浏览器的最新版本，对 ES6 的支持可以查看[kangax.github.io/es5-compat-table/es6/](http://kangax.github.io/es5-compat-table/es6/)。随着时间的推移，支持度已经越来越高了，ES6 的大部分特性都实现了。

Node.js 和 io.js（一个部署新功能更快的 Node 分支）是 JavaScript 语言的服务器运行环境。它们对 ES6 的支持度，比浏览器更高。通过它们，可以体验更多 ES6 的特性。

建议使用版本管理工具[nvm](https://github.com/creationix/nvm)，来安装 Node.js 和 io.js。不过，nvm 不支持 Windows 系统，下面的操作可以改用[nvmw](https://github.com/hakobera/nvmw)或[nvm-windows](https://github.com/coreybutler/nvm-windows)代替。

安装 nvm 需要打开命令行窗口，运行下面的命令。

```
$ curl -o- https://raw.githubusercontent.com/creationix/nvm/<version number>/install.sh | bash

```

上面命令的 version number 处，需要用版本号替换。本书写作时的版本号是 v0.25.4。

该命令运行后，nvm 会默认安装在用户主目录的`.nvm`子目录。然后，激活 nvm。

```
$ source ~/.nvm/nvm.sh

```

激活以后，安装 Node 或 io.js 的最新版。

```
$ nvm install node
# 或
$ nvm install iojs

```

安装完成后，就可以在各种版本的 node 之间自由切换。

```
# 切换到 node
$ nvm use node

# 切换到 iojs
$ nvm use iojs

```

需要注意的是，Node.js 对 ES6 的支持，需要打开 harmony 参数，iojs 不需要。

```
$ node --harmony
# iojs 不需要打开 harmony 参数
$ node

```

上面命令执行后，就会进入 REPL 环境，该环境支持所有已经实现的 ES6 特性。

使用下面的命令，可以查看 Node.js 所有已经实现的 ES6 特性。

```
$ node --v8-options | grep harmony

  --harmony_typeof
  --harmony_scoping
  --harmony_modules
  --harmony_symbols
  --harmony_proxies
  --harmony_collections
  --harmony_observation
  --harmony_generators
  --harmony_iteration
  --harmony_numeric_literals
  --harmony_strings
  --harmony_arrays
  --harmony_maths
  --harmony

```

上面命令的输出结果，会因为版本的不同而有所不同。

我写了一个[ES-Checker](https://github.com/ruanyf/es-checker)模块，用来检查各种运行环境对 ES6 的支持情况。访问[ruanyf.github.io/es-checker](http://ruanyf.github.io/es-checker)，可以看到您的浏览器支持 ES6 的程度。运行下面的命令，可以查看本机支持 ES6 的程度。

```
$ npm install -g es-checker
$ es-checker

```

## Babel 转码器

[Babel](https://babeljs.io/)是一个广泛使用的 ES6 转码器，可以 ES6 代码转为 ES5 代码，从而在浏览器或其他环境执行。这意味着，你可以用 ES6 的方式编写程序，又不用担心现有环境是否支持。下面是一个例子。

```
// 转码前
input.map(item => item + 1);

// 转码后
input.map(function (item) {
  return item + 1;
});

```

上面的原始代码用了箭头函数，这个特性还没有得到广泛支持，Babel 将其转为普通函数，就能在现有的 JavaScript 环境执行了。

它的安装命令如下。

```
$ npm install --global babel

```

Babel 自带一个`babel-node`命令，提供支持 ES6 的 REPL 环境。它支持 Node 的 REPL 环境的所有功能，而且可以直接运行 ES6 代码。

```
$ babel-node
>
> console.log([1,2,3].map(x => x * x))
    [ 1, 4, 9 ]
>

```

`babel-node`命令也可以直接运行 ES6 脚本。假定将上面的代码放入脚本文件`es6.js`。

```
$ babel-node es6.js
[1, 4, 9]

```

babel 命令可以将 ES6 代码转为 ES5 代码。

```
$ babel es6.js
"use strict";

console.log([1, 2, 3].map(function (x) {
  return x * x;
}));

```

`-o`参数将转换后的代码，从标准输出导入文件。

```
$ babel es6.js -o es5.js
# 或者
$ babel es6.js --out-file es5.js

```

`-d`参数用于转换整个目录。

```
$ babel -d build-dir source-dir

```

注意，`-d`参数后面跟的是输出目录。

如果希望生成 source map 文件，则要加上`-s`参数。

```
$ babel -d build-dir source-dir -s

```

Babel 也可以用于浏览器。

```
<script src="node_modules/babel-core/browser.js"></script>
<script type="text/babel">
// Your ES6 code
</script>

```

上面代码中，`browser.js`是 Babel 提供的转换器脚本，可以在浏览器运行。用户的 ES6 脚本放在 script 标签之中，但是要注明`type="text/babel"`。

Babel 配合 Browserify 一起使用，可以生成浏览器能够直接加载的脚本。

```
$ browserify script.js -t babelify --outfile bundle.js

```

## Traceur 转码器

Google 公司的[Traceur](https://github.com/google/traceur-compiler)转码器，也可以将 ES6 代码转为 ES5 代码。

### 直接插入网页

Traceur 允许将 ES6 代码直接插入网页。首先，必须在网页头部加载 Traceur 库文件。

```
<!-- 加载 Traceur 编译器 -->
<script src="http://google.github.io/traceur-compiler/bin/traceur.js"   type="text/javascript"></script>
<!-- 将 Traceur 编译器用于网页 -->
<script src="http://google.github.io/traceur-compiler/src/bootstrap.js"   type="text/javascript"></script>
<!-- 打开实验选项，否则有些特性可能编译不成功 -->
<script>
traceur.options.experimental = true;
</script>

```

接下来，就可以把 ES6 代码放入上面这些代码的下方。

```
<script type="module">
  class Calc {
    constructor(){
      console.log('Calc constructor');
    }
    add(a, b){
      return a + b;
    }
  }

  var c = new Calc();
  console.log(c.add(4,5));
</script>

```

正常情况下，上面代码会在控制台打印出 9。

注意，`script`标签的`type`属性的值是`module`，而不是`text/javascript`。这是 Traceur 编译器识别 ES6 代码的标识，编译器会自动将所有`type=module`的代码编译为 ES5，然后再交给浏览器执行。

如果 ES6 代码是一个外部文件，也可以用`script`标签插入网页。

```
<script type="module" src="calc.js" >
</script>

```

### 在线转换

Traceur 提供一个[在线编译器](http://google.github.io/traceur-compiler/demo/repl.html)，可以在线将 ES6 代码转为 ES5 代码。转换后的代码，可以直接作为 ES5 代码插入网页运行。

上面的例子转为 ES5 代码运行，就是下面这个样子。

```
<script src="http://google.github.io/traceur-compiler/bin/traceur.js"
        type="text/javascript"></script>
<script src="http://google.github.io/traceur-compiler/src/bootstrap.js"
        type="text/javascript"></script>
<script>
        traceur.options.experimental = true;
</script>
<script>
$traceurRuntime.ModuleStore.getAnonymousModule(function() {
  "use strict";

  var Calc = function Calc() {
    console.log('Calc constructor');
  };

  ($traceurRuntime.createClass)(Calc, {add: function(a, b) {
    return a + b;
  }}, {});

  var c = new Calc();
  console.log(c.add(4, 5));
  return {};
});
</script>

```

### 命令行转换

作为命令行工具使用时，Traceur 是一个 Node.js 的模块，首先需要用 npm 安装。

```
$ npm install -g traceur

```

安装成功后，就可以在命令行下使用 traceur 了。

traceur 直接运行 es6 脚本文件，会在标准输出显示运行结果，以前面的 calc.js 为例。

```
$ traceur calc.js
Calc constructor
9

```

如果要将 ES6 脚本转为 ES5 保存，要采用下面的写法

```
$ traceur --script calc.es6.js --out calc.es5.js

```

上面代码的`--script`选项表示指定输入文件，`--out`选项表示指定输出文件。

为了防止有些特性编译不成功，最好加上`--experimental`选项。

```
$ traceur --script calc.es6.js --out calc.es5.js --experimental

```

命令行下转换的文件，就可以放到浏览器中运行。

### Node.js 环境的用法

Traceur 的 Node.js 用法如下（假定已安装 traceur 模块）。

```
var traceur = require('traceur');
var fs = require('fs');

// 将 ES6 脚本转为字符串
var contents = fs.readFileSync('es6-file.js').toString();

var result = traceur.compile(contents, {
  filename: 'es6-file.js',
  sourceMap: true,
  // 其他设置
  modules: 'commonjs'
});

if (result.error)
  throw result.error;

// result 对象的 js 属性就是转换后的 ES5 代码
fs.writeFileSync('out.js', result.js);
// sourceMap 属性对应 map 文件
fs.writeFileSync('out.js.map', result.sourceMap);

```

## ECMAScript 7

2013 年 3 月，ES6 的草案封闭，不再接受新功能了。新的功能将被加入 ES7。

ES7 可能包括的功能有：

（1）**Object.observe**：用来监听对象（以及数组）的变化。一旦监听对象发生变化，就会触发回调函数。

（2）**Async 函数**：在 Promise 和 Generator 函数基础上，提出的异步操作解决方案。

（3）**Multi-Threading**：多线程支持。目前，Intel 和 Mozilla 有一个共同的研究项目 RiverTrail，致力于让 JavaScript 多线程运行。预计这个项目的研究成果会被纳入 ECMAScript 标准。

（4）**Traits**：它将是“类”功能（class）的一个替代。通过它，不同的对象可以分享同样的特性。

其他可能包括的功能还有：更精确的数值计算、改善的内存回收、增强的跨站点安全、类型化的更贴近硬件的低级别操作、国际化支持（Internationalization Support）、更多的数据结构等等。

本书对于那些明确的、或者很有希望列入 ES7 的功能，尤其是那些 Babel 已经支持的功能，都将予以介绍。