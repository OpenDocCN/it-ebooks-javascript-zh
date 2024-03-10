# 2.9 错误处理机制

*   Error 对象
*   JavaScript 的原生错误类型
*   自定义错误
*   throw 语句
*   try...catch 结构
*   finally 代码块
*   参考连接

## Error 对象

一旦代码解析或运行时发生错误，JavaScript 引擎就会自动产生并抛出一个 Error 对象的实例，然后整个程序就中断在发生错误的地方。

Error 对象的实例有三个最基本的属性：

*   name：错误名称
*   message：错误提示信息
*   stack：错误的堆栈（非标准属性，但是大多数平台支持）

利用 name 和 message 这两个属性，可以对发生什么错误有一个大概的了解。

```js
if (error.name){
  console.log(error.name + ": " + error.message);
}
```

上面代码表示，显示错误的名称以及出错提示信息。

stack 属性用来查看错误发生时的堆栈。

```js
function throwit() {
  throw new Error('');
}

function catchit() {
  try {
    throwit();
  } catch(e) {
    console.log(e.stack); // print stack trace
  }
}

catchit()
// Error
//    at throwit (~/examples/throwcatch.js:9:11)
//    at catchit (~/examples/throwcatch.js:3:9)
//    at repl:1:5
```

上面代码显示，抛出错误首先是在 throwit 函数，然后是在 catchit 函数，最后是在函数的运行环境中。

## JavaScript 的原生错误类型

Error 对象是最一般的错误类型，在它的基础上，JavaScript 还定义了其他 6 种错误，也就是说，存在 Error 的 6 个派生对象。

（1）SyntaxError

SyntaxError 是解析代码时发生的语法错误。

```js
// 变量名错误
var 1a;

// 缺少括号
console.log 'hello');
```

（2）ReferenceError

ReferenceError 是引用一个不存在的变量时发生的错误。

```js
unknownVariable
// ReferenceError: unknownVariable is not defined
```

另一种触发场景是，将一个值分配给无法分配的对象，比如对函数的运行结果或者 this 赋值。

```js
console.log() = 1
// ReferenceError: Invalid left-hand side in assignment

this = 1
// ReferenceError: Invalid left-hand side in assignment
```

上面代码对函数 console.log 的运行结果和 this 赋值，结果都引发了 ReferenceError 错误。

（3）RangeError

RangeError 是当一个值超出有效范围时发生的错误。主要有几种情况，一是数组长度为负数，二是 Number 对象的方法参数超出范围，以及函数堆栈超过最大值。

```js
new Array(-1)
// RangeError: Invalid array length

(1234).toExponential(21)
// RangeError: toExponential() argument must be between 0 and 20
```

（4）TypeError

TypeError 是变量或参数不是预期类型时发生的错误。比如，对字符串、布尔值、数值等原始类型的值使用 new 命令，就会抛出这种错误，因为 new 命令的参数应该是一个构造函数。

```js
new 123
//TypeError: number is not a func

var obj = {};
obj.unknownMethod()
// TypeError: undefined is not a function
```

上面代码的第二种情况，调用对象不存在的方法，会抛出 TypeError 错误。

（5）URIError

URIError 是 URI 相关函数的参数不正确时抛出的错误，主要涉及 encodeURI()、decodeURI()、encodeURIComponent()、decodeURIComponent()、escape()和 unescape()这六个函数。

```js
decodeURI('%2')
// URIError: URI malformed
```

（6）EvalError

eval 函数没有被正确执行时，会抛出 EvalError 错误。该错误类型已经不再在 ES5 中出现了，只是为了保证与以前代码兼容，才继续保留。

以上这 6 种派生错误，连同原始的 Error 对象，都是构造函数。开发者可以使用它们，人为生成错误对象的实例。

```js
new Error("出错了！");
new RangeError("出错了，变量超出有效范围！");
new TypeError("出错了，变量类型无效！");
```

上面代码表示新建错误对象的实例，实质就是手动抛出错误。可以看到，错误对象的构造函数接受一个参数，代表错误提示信息（message）。

## 自定义错误

除了 JavaScript 内建的 7 种错误对象，还可以定义自己的错误对象。

```js
function UserError(message) {
   this.message = message || "默认信息";
   this.name = "UserError";
}

UserError.prototype = new Error();
UserError.prototype.constructor = UserError;
```

上面代码自定义一个错误对象 UserError，让它继承 Error 对象。然后，就可以生成这种自定义的错误了。

```js
new UserError("这是自定义的错误！");
```

## throw 语句

throw 语句的作用是中断程序执行，抛出一个意外或错误。它接受一个表达式作为参数。

```js
throw "Error！";
throw 42;
throw true;
throw {toString: function() { return "Error!"; } };
```

上面代码表示，throw 可以接受各种值作为参数。JavaScript 引擎一旦遇到 throw 语句，就会停止执行后面的语句，并将 throw 语句的参数值，返回给用户。

如果只是简单的错误，返回一条出错信息就可以了，但是如果遇到复杂的情况，就需要在出错以后进一步处理。这时最好的做法是使用 throw 语句手动抛出一个 Error 对象。

```js
throw new Error('出错了!');
```

上面语句新建一个 Error 对象，然后将这个对象抛出，整个程序就会中断在这个地方。

throw 语句还可以抛出用户自定义的错误。

```js
function UserError(message) {
   this.message = message || "默认信息";
   this.name = "UserError";
}

UserError.prototype.toString = function (){
  return this.name + ': "' + this.message + '"';
}

throw new UserError("出错了！");
```

## try...catch 结构

为了对错误进行处理，需要使用 try...catch 结构。

```js
try {
  throw new Error('出错了!');
} catch (e) {
  console.log(e.name + ": " + e.message);  // Error: 出错了！
  console.log(e.stack);  // 不是标准属性，但是浏览器支持
}
// Error: 出错了!
// Error: 出错了!
//   at <anonymous>:3:9
//   at Object.InjectedScript._evaluateOn (<anonymous>:895:140)
//   at Object.InjectedScript._evaluateAndWrap (<anonymous>:828:34)
//   at Object.InjectedScript.evaluate (<anonymous>:694:21)
```

上面代码中，try 代码块抛出的错误（包括用 throw 语句抛出错误），可以被 catch 代码块捕获。catch 接受一个参数，表示 try 代码块传入的错误对象。

```js
function throwIt(exception) {
  try {
    throw exception;
  } catch (e) {
    console.log('Caught: '+e);
  }
}

throwIt(3);
// Caught: 3
throwIt('hello');
// Caught: hello
throwIt(new Error('An error happened'));
// Caught: Error: An error happened
```

catch 代码块捕获错误之后，程序不会中断，会按照正常流程继续执行下去。

```js
try {
  throw "出错了";
} catch (e) {
  console.log(111);
}
console.log(222);
// 111
// 222
```

上面代码中，try 代码块抛出的错误，被 catch 代码块捕获后，程序会继续向下执行。

catch 代码块之中，还可以再抛出错误，甚至使用嵌套的 try...catch 结构。

```js
try {
   throw n; // 这里抛出一个整数
} catch (e) {
   if (e <= 50) {
      // 针对 1-50 的错误的处理
   } else {
      // 大于 50 的错误无法处理，再抛出一个错误
      throw e;
   }
}
```

为了捕捉不同类型的错误，catch 代码块之中可以加入判断语句。

```js
try {
  foo.bar();
} catch (e) {
  if (e instanceof EvalError) {
    console.log(e.name + ": " + e.message);
  } else if (e instanceof RangeError) {
    console.log(e.name + ": " + e.message);
  }
  // ... 
}
```

try...catch 结构是 JavaScript 语言受到 Java 语言影响的一个明显的例子。这种结构多多少少是对结构化编程原则一种破坏，处理不当就会变成类似 goto 语句的效果，应该谨慎使用。

## finally 代码块

try...catch 结构允许在最后添加一个 finally 代码块，表示不管是否出现错误，都必需在最后运行的语句。

```js
function cleansUp() {
    try {
        throw new Error('Sorry...');
    } finally {
        console.log('Performing clean-up');
    }
}

cleansUp()
// Performing clean-up
// Error: Sorry...
```

上面代码说明，throw 语句抛出错误以后，finanlly 继续得到执行。

```js
function idle(x) {
    try {
        console.log(x);
        return 'result';
    } finally {
        console.log("FINALLY");
    }
}

idle('hello')
// hello
// FINALLY
// "result"
```

上面代码说明，即使有 return 语句在前，finally 代码块依然会得到执行，且在其执行完毕后，才会显示 return 语句的值。

下面的例子说明，return 语句的执行是排在 finanlly 代码之前，只是等 finnally 代码执行完毕后才返回。

```js
var count = 0;
function countUp() {
    try {
        return count;
    } finally {
        count++;
    }
}

countUp()
// 0
count
// 1
```

上面代码说明，return 语句的 count 的值，是在 finally 代码块运行之前，就获取完成了。

下面是另一个例子。

```js
openFile();

try {
   writeFile(Data);
} catch(e) {
    handleError(e);
} finally {
   closeFile();
}
```

上面代码首先打开一个文件，然后在 try 代码块中写入文件，如果没有发生错误，则运行 finally 代码块关闭文件；一旦发生错误，则先使用 catch 代码块处理错误，再使用 finally 代码块关闭文件。

下面的例子充分反应了 try...catch...finally 这三者之间的执行顺序。

```js
function f() {
    try {
        console.log(0);
        throw "bug";
    } catch(e) {
        console.log(1);
        return true; // 这句会延迟到 finally 代码块结束再执行
        console.log(2); // 不会运行
    } finally {
        console.log(3);
        return false; // 这句会覆盖掉前面那句 return
        console.log(4); // 不会运行
    }

    console.log(5); // 不会运行
}

var result = f(); 
// 0
// 1
// 3

result
// false
```

某些情况下，甚至可以省略 catch 代码块，只使用 finally 代码块。

```js
openFile();

try {
   writeFile(Data);
} finally {
   closeFile();
}
```

## 参考连接

*   Jani Hartikainen, [JavaScript Errors and How to Fix Them](http://davidwalsh.name/fix-javascript-errors)