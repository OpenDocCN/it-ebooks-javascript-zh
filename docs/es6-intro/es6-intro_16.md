# Module

ES6 的 Class 只是面向对象编程的语法糖，升级了 ES5 的构造函数的原型链继承的写法，并没有解决模块化问题。Module 功能就是为了解决这个问题而提出的。

历史上，JavaScript 一直没有模块（module）体系，无法将一个大程序拆分成互相依赖的小文件，再用简单的方法拼装起来。其他语言都有这项功能，比如 Ruby 的 require、Python 的 import，甚至就连 CSS 都有@import，但是 JavaScript 任何这方面的支持都没有，这对开发大型的、复杂的项目形成了巨大障碍。

在 ES6 之前，社区制定了一些模块加载方案，最主要的有 CommonJS 和 AMD 两种。前者用于服务器，后者用于浏览器。ES6 在语言规格的层面上，实现了模块功能，而且实现得相当简单，完全可以取代现有的 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案。

ES6 模块的设计思想，是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块，都只能在运行时确定这些东西。比如，CommonJS 模块就是对象，输入时必须查找对象属性。

```js
var { stat, exists, readFile } = require('fs');

```

ES6 模块不是对象，而是通过 export 命令显式指定输出的代码，输入时也采用静态命令的形式。

```js
import { stat, exists, readFile } from 'fs';

```

所以，ES6 可以在编译时就完成模块编译，效率要比 CommonJS 模块高。

## export 命令

模块功能主要由两个命令构成：export 和 import。export 命令用于用户自定义模块，规定对外接口；import 命令用于输入其他模块提供的功能，同时创造命名空间（namespace），防止函数名冲突。

ES6 允许将独立的 JS 文件作为模块，也就是说，允许一个 JavaScript 脚本文件调用另一个脚本文件。该文件内部的所有变量，外部无法获取，必须使用 export 关键字输出变量。下面是一个 JS 文件，里面使用 export 命令输出变量。

```js
// profile.js
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1958;

```

上面代码是 profile.js 文件，保存了用户信息。ES6 将其视为一个模块，里面用 export 命令对外部输出了三个变量。

export 的写法，除了像上面这样，还有另外一种。

```js
// profile.js
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;

export {firstName, lastName, year};

```

上面代码在 export 命令后面，使用大括号指定所要输出的一组变量。它与前一种写法（直接放置在 var 语句前）是等价的，但是应该优先考虑使用这种写法。因为这样就可以在脚本尾部，一眼看清楚输出了哪些变量。

export 命令除了输出变量，还可以输出函数或类（class）。

```js
export function multiply (x, y) {
  return x * y;
};

```

上面代码对外输出一个函数 multiply。

## import 命令

使用 export 命令定义了模块的对外接口以后，其他 JS 文件就可以通过 import 命令加载这个模块（文件）。

```js
// main.js

import {firstName, lastName, year} from './profile';

function sfirsetHeader(element) {
  element.textContent = firstName + ' ' + lastName;
}

```

上面代码属于另一个文件 main.js，import 命令就用于加载 profile.js 文件，并从中输入变量。import 命令接受一个对象（用大括号表示），里面指定要从其他模块导入的变量名。大括号里面的变量名，必须与被导入模块（profile.js）对外接口的名称相同。

如果想为输入的变量重新取一个名字，import 语句中要使用 as 关键字，将输入的变量重命名。

```js
import { lastName as surname } from './profile';

```

ES6 支持多重加载，即所加载的模块中又加载其他模块。

```js
import { Vehicle } from './Vehicle';

class Car extends Vehicle {
  move () {
    console.log(this.name + ' is spinning wheels...')
  }
}

export { Car }

```

上面的模块先加载 Vehicle 模块，然后在其基础上添加了 move 方法，再作为一个新模块输出。

如果在一个模块之中，先输入后输出同一个模块，import 语句可以与 export 语句写在一起。

```js
export { es6 as default } from './someModule';

// 等同于
import { es6 } from './someModule';
export default es6;

```

上面代码中，export 和 import 语句可以结合在一起，写成一行。但是从可读性考虑，不建议采用这种写法，h 应该采用标准写法。

## 模块的整体输入

下面是一个 circle.js 文件，它输出两个方法 area 和 circumference。

```js
// circle.js

export function area(radius) {
  return Math.PI * radius * radius;
}

export function circumference(radius) {
  return 2 * Math.PI * radius;
}

```

然后，main.js 文件输入 circlek.js 模块。

```js
// main.js

import { area, circumference } from 'circle';

console.log("圆面积：" + area(4));
console.log("圆周长：" + circumference(14));

```

上面写法是逐一指定要输入的方法。另一种写法是整体输入。

```js
import * as circle from 'circle';

console.log("圆面积：" + circle.area(4));
console.log("圆周长：" + circle.circumference(14));

```

## module 命令

module 命令可以取代 import 语句，达到整体输入模块的作用。

```js
// main.js

module circle from 'circle';

console.log("圆面积：" + circle.area(4));
console.log("圆周长：" + circle.circumference(14));

```

module 命令后面跟一个变量，表示输入的模块定义在该变量上。

## export default 命令

从前面的例子可以看出，使用 import 的时候，用户需要知道所要加载的变量名或函数名，否则无法加载。但是，用户肯定希望快速上手，未必愿意阅读文档，去了解模块有哪些属性和方法。

为了给用户提供方便，让他们不用阅读文档就能加载模块，就要用到`export default`命令，为模块指定默认输出。

```js
// export-default.js
export default function () {
  console.log('foo');
}

```

上面代码是一个模块文件`export-default.js`，它的默认输出是一个函数。

其他模块加载该模块时，import 命令可以为该匿名函数指定任意名字。

```js
// import-default.js
import customName from './export-default';
customName(); // 'foo'

```

上面代码的 import 命令，可以用任意名称指向`export-default.js`输出的方法。需要注意的是，这时 import 命令后面，不使用大括号。

export default 命令用在非匿名函数前，也是可以的。

```js
// export-default.js
export default function foo() {
  console.log('foo');
}

// 或者写成

function foo() {
  console.log('foo');
}

export default foo;

```

上面代码中，foo 函数的函数名 foo，在模块外部是无效的。加载的时候，视同匿名函数加载。

下面比较一下默认输出和正常输出。

```js
import crc32 from 'crc32';
// 对应的输出
export default function crc32(){}

import { crc32 } from 'crc32';
// 对应的输出
export function crc32(){};

```

上面代码的两组写法，第一组是使用`export default`时，对应的 import 语句不需要使用大括号；第二组是不使用`export default`时，对应的 import 语句需要使用大括号。

`export default`命令用于指定模块的默认输出。显然，一个模块只能有一个默认输出，因此`export deault`命令只能使用一次。所以，import 命令后面才不用加大括号，因为只可能对应一个方法。

本质上，`export default`就是输出一个叫做 default 的变量或方法，然后系统允许你为它取任意名字。所以，下面的写法是有效的。

```js
// modules.js
export default function (x, y) {
  return x * y;
};
// app.js
import { default } from 'modules';

```

有了`export default`命令，输入模块时就非常直观了，以输入 jQuery 模块为例。

```js
import $ from 'jquery';

```

如果想在一条 import 语句中，同时输入默认方法和其他变量，可以写成下面这样。

```js
import customName, { otherMethod } from './export-default';

```

如果要输出默认的值，只需将值跟在`export default`之后即可。

```js
export default 42;

```

`export default`也可以用来输出类。

```js
// MyClass.js
export default class { ... }

// main.js
import MyClass from 'MyClass'
let o = new MyClass();

```

## 模块的继承

模块之间也可以继承。

假设有一个 circleplus 模块，继承了 circle 模块。

```js
// circleplus.js

export * from 'circle';
export var e = 2.71828182846;
export default function(x) {
    return Math.exp(x);
}

```

上面代码中的“export *”，表示输出 circle 模块的所有属性和方法，export default 命令定义模块的默认方法。

这时，也可以将 circle 的属性或方法，改名后再输出。

```js
// circleplus.js

export { area as circleArea } from 'circle';

```

上面代码表示，只输出 circle 模块的 area 方法，且将其改名为 circleArea。

加载上面模块的写法如下。

```js
// main.js

module math from "circleplus";
import exp from "circleplus";
console.log(exp(math.pi));

```

上面代码中的"import exp"表示，将 circleplus 模块的默认方法加载为 exp 方法。

## ES6 模块的转码

浏览器目前还不支持 ES6 模块，为了现在就能使用，可以将转为 ES5 的写法。

### ES6 module transpiler

[ES6 module transpiler](https://github.com/esnext/es6-module-transpiler)是 square 公司开源的一个转码器，可以将 ES6 模块转为 CommonJS 模块或 AMD 模块的写法，从而在浏览器中使用。

首先，安装这个转玛器。

```js
$ npm install -g es6-module-transpiler

```

然后，使用`compile-modules convert`命令，将 ES6 模块文件转码。

```js
$ compile-modules convert file1.js file2.js

```

o 参数可以指定转码后的文件名。

```js
$ compile-modules convert -o out.js file1.js

```

### SystemJS

另一种解决方法是使用[SystemJS](https://github.com/systemjs/systemjs)。它是一个垫片库（polyfill），可以在浏览器内加载 ES6 模块、AMD 模块和 CommonJS 模块，将其转为 ES5 格式。它在后台调用的是 Google 的 Traceur 转码器。

使用时，先在网页内载入 system.js 文件。

```js
<script src="system.js"></script>

```

然后，使用`System.import`方法加载模块文件。

```js
<script>
  System.import('./app');
</script>

```

上面代码中的`./app`，指的是当前目录下的 app.js 文件。它可以是 ES6 模块文件，`System.import`会自动将其转码。

需要注意的是，`System.import`使用异步加载，返回一个 Promise 对象，可以针对这个对象编程。下面是一个模块文件。

```js
// app/es6-file.js:

export class q {
  constructor() {
    this.es6 = 'hello';
  }
}

```

然后，在网页内加载这个模块文件。

```js
<script>

System.import('app/es6-file').then(function(m) {
  console.log(new m.q().es6); // hello
});

</script>

```

上面代码中，`System.import`方法返回的是一个 Promise 对象，所以可以用 then 方法指定回调函数。