# 第十一章 Promise 与异步编程

## 第十一章 Promise 与异步编程

JS 最强大的一方面就是它能极其轻易地处理异步编程。作为因互联网而生的语言， JS 从一开始就必须能够响应点击或按键之类的用户交互行为。 Node.js 通过使用回调函数来代替事件，进一步推动了 JS 中的异步编程。随着越来越多的程序开始使用异步编程，事件与回调函数已不足以支持开发者的所有需求。 **Promise** 正是为了解决这方面的问题。

Promises 是异步编程的另一种选择，它的工作方式类似于在其他语言中延迟并在将来执行作业。一个 Promise 指定一些稍后要执行的代码（就像事件与回调函数一样），并且也明确标示了作业的代码是否执行成功。你能以成功处理或失败处理为基准，将 Promise 串联在一起，让代码更易理解、更易调试。

不过为了更好理解 Promise 是如何工作的，重要的是理解建立它所依据的一些基本概念。

*   异步编程的背景
    *   事件模型
    *   回调模式
*   Promise 基础
    *   Promise 的生命周期
    *   创建未决的 Promise
    *   创建已决的 Promise
        *   使用 Promise.resolve()
        *   使用 Promise.reject()
        *   非 Promise 的 Thenable
    *   执行器错误
*   全局的 Promise 拒绝处理
    *   Node.js 的拒绝处理
    *   浏览器的拒绝处理
*   串联 Promise
    *   捕获错误
    *   在 Promise 链中返回值
    *   在 Promise 链中返回 Promise
*   响应多个 Promise
    *   Promise.all() 方法
    *   Promise.race() 方法
*   继承 Promise
*   异步任务运行
*   总结

### 异步编程的背景

JS 引擎建立在单线程事件循环的概念上。**单线程**（ **Single-threaded** ）意味着同一时刻只能执行一段代码，与 Java 或 C++ 这种允许同时执行多段不同代码的多线程语言形成了反差。多段代码可以同时访问或修改状态，维护并保护这些状态就变成了难题，这也是基于多线程的软件中出现 bug 的常见根源之一。

JS 引擎在同一时刻只能执行一段代码，所以引擎无须留意那些“可能”运行的代码。代码会被放置在**作业队列**（ **job queue** ）中，每当一段代码准备被执行，它就会被添加到作业队列。当 JS 引擎结束当前代码的执行后，事件循环就会执行队列中的下一个作业。**事件循环**（ **event loop** ）是 JS 引擎的一个内部处理线程，能监视代码的执行并管理作业队列。要记住既然是一个队列，作业就会从队列中的第一个开始，依次运行到最后一个。

#### 事件模型

当用户点击一个按钮或按下键盘上的一个键时，一个**事件**（ **event** ）——例如 `onclick` ——就被触发了。该事件可能会对此交互进行响应，从而将一个新的作业添加到作业队列的尾部。这就是 JS 关于异步编程的最基本形式。事件处理程序代码直到事件发生后才会被执行，此时它会拥有合适的上下文。例如：

```
let button = document.getElementById("my-btn");
button.onclick = function(event) {
    console.log("Clicked");
}; 
```

在此代码中， `console.log("Clicked")` 直到 `button` 被点击后才会被执行。当 `button` 被点击，赋值给 `onclick` 的函数就被添加到作业队列的尾部，并在队列前部所有任务结束之后再执行。

事件可以很好地工作于简单的交互，但将多个分离的异步调用串联在一起却会很麻烦，因为必须追踪每个事件的事件对象（例如上例中的 `button` ）。此外，你还需确保所有的事件处理程序都能在事件第一次触发之前被绑定完毕。例如，若 `button` 在 `onclick` 被绑定之前就被点击，那就不会有任何事发生。因此虽然在响应用户交互或类似的低频功能时，事件很有用，但它在面对更复杂的需求时仍然不够灵活。

#### 回调模式

当 Node.js 被创建时，它通过普及回调函数编程模式提升了异步编程模型。回调函数模式类似于事件模型，因为异步代码也会在后面的一个时间点才执行。不同之处在于需要调用的函数（即回调函数）是作为参数传入的，如下所示：

```
readFile("example.txt", function(err, contents) {
    if (err) {
        throw err;
    }

    console.log(contents);
});
console.log("Hi!"); 
```

此例使用了 Node.js 惯例，即**错误优先**（ **error-first** ）的回调函数风格。 `readFile()` 函数用于读取磁盘中的文件（由第一个参数指定），并在读取完毕后执行回调函数（即第二个参数）。如果存在错误，回调函数的 `err` 参数会是一个错误对象；否则 `contents` 参数就会以字符串形式包含文件内容。

使用回调函数模式， `readFile()` 会立即开始执行，并在开始读取磁盘时暂停。这意味着 `console.log("Hi!")` 会在 `readFile()` 被调用后立即进行输出，要早于 `console.log(contents)` 的打印操作。当 `readFile()` 结束操作后，它会将回调函数以及相关参数作为一个新的作业添加到作业队列的尾部。在之前的作业全部结束后，该作业才会执行。

回调函数模式要比事件模型灵活得多，因为使用回调函数串联多个调用会相对容易。例如：

```
readFile("example.txt", function(err, contents) {
    if (err) {
        throw err;
    }

    writeFile("example.txt", function(err) {
        if (err) {
            throw err;
        }

        console.log("File was written!");
    });
}); 
```

在此代码中，对于 `readFile()` 的一次成功调用引出了另一个异步调用，即调用 `writeFile()` 函数。注意这两个函数都使用了检查 `err` 的同一基本模式。当 `readFile()` 执行结束后，它添加一个作业到作业队列，从而导致 `writeFile()` 在之后被调用（假设没有出现错误）。接下来， `writeFile()` 也会在执行结束后向队列添加一个作业。

这种模式运作得相当好，但你可能会迅速察觉陷入了**回调地狱**（ **callback hell** ），这会在嵌套过多回调函数时发生，就像这样：

```
method1(function(err, result) {

    if (err) {
        throw err;
    }

    method2(function(err, result) {

        if (err) {
            throw err;
        }

        method3(function(err, result) {

            if (err) {
                throw err;
            }

            method4(function(err, result) {

                if (err) {
                    throw err;
                }

                method5(result);
            });

        });

    });

}); 
```

像本例一样嵌套多个方法调用会创建错综复杂的代码，会难以理解与调试。当想要实现更复杂的功能时，回调函数也会存在问题。要是你想让两个异步操作并行运行，并且在它们都结束后提醒你，那该怎么做？要是你想同时启动两个异步操作，但只采用首个结束的结果，那又该怎么做？

在这些情况下，你需要追踪多个回调函数并做清理操作， Promise 能大幅度改善这种情况。

### Promise 基础

Promise 是为异步操作的结果所准备的占位符。函数可以返回一个 Promise，而不必订阅一个事件或向函数传递一个回调参数，就像这样：

```
// readFile 承诺会在将来某个时间点完成
let promise = readFile("example.txt"); 
```

在此代码中， `readFile()` 实际上并未立即开始读取文件，这将会在稍后发生。此函数反而会返回一个 Promise 对象以表示异步读取操作，因此你可以在将来再操作它。你能对结果进行操作的确切时刻，完全取决于 Promise 的生命周期是如何进行的。

#### Promise 的生命周期

每个 Promise 都会经历一个短暂的生命周期，初始为**挂起**态（ **pending** state），这表示异步操作尚未结束。一个挂起的 Promise 也被认为是**未决的**（ **unsettled** ）。上个例子中的 Promise 在 `readFile()` 函数返回它的时候就是处在挂起态。一旦异步操作结束， Promise 就会被认为是**已决的**（ **settled** ），并进入两种可能状态之一：

1.  **已完成**（ **fulfilled** ）： Promise 的异步操作已成功结束；
2.  **已拒绝**（ **rejected** ）： Promise 的异步操作未成功结束，可能是一个错误，或由其他原因导致。

内部的 `[[PromiseState]]` 属性会被设置为 `"pending"` 、 `"fulfilled"` 或 `"rejected"` ，以反映 Promise 的状态。该属性并未在 Promise 对象上被暴露出来，因此你无法以编程方式判断 Promise 到底处于哪种状态。不过你可以使用 `then()` 方法在 Promise 的状态改变时执行一些特定操作。

> 译注：**相关词汇翻译汇总**
> 
> Promise 是相对比较新的一个概念，相关的许多词汇有一定的交叉性，并且在翻译为中文时可能有些并不太容易分辨。因此涉及 Promise 的许多资料都对相关大部分词汇不作翻译，直接使用英文原词。
> 
> 译者在本章斗胆对几乎所有词汇进行了翻译，如有不妥，欢迎指出。此处是词汇翻译的汇总，以便参考：
> 
> 1.  pending ：挂起，表示未结束的 Promise 状态。相关词汇“挂起态”。
> 2.  fulfilled ：已完成，表示已成功结束的 Promise 状态，可以理解为“成功完成”。相关词汇“完成”、“被完成”、“完成态”。
> 3.  rejected ：已拒绝，表示已结束但失败的 Promise 状态。相关词汇“拒绝”、“被拒绝”、“拒绝态”。
> 4.  resolve ：决议，表示将 Promise 推向成功态，可以理解为“决议通过”，在 Promise 概念中与“完成”是近义词。相关词汇“决议态”、“已决议”、“被决议”。
> 5.  unsettled ：未决，或者称为“未解决”，表示 Promise 尚未被完成或拒绝，与“挂起”是近义词。
> 6.  settled ：已决，或者称为“已解决”，表示 Promise 已被完成或拒绝。注意这与“已完成”或“已决议”不同，“已决”的状态也可能是“拒绝态”（已失败）。
> 7.  fulfillment handler ：完成处理函数，表示 Promise 为完成态时会被调用的函数。
> 8.  rejection handler ：拒绝处理函数，表示 Promise 为拒绝态时会被调用的函数。

`then()` 方法在所有的 Promise 上都存在，并且接受两个参数。第一个参数是 Promise 被完成时要调用的函数，与异步操作关联的任何附加数据都会被传入这个完成函数。第二个参数则是 Promise 被拒绝时要调用的函数，与完成函数相似，拒绝函数会被传入与拒绝相关联的任何附加数据。

用这种方式实现 `then()` 方法的任何对象都被称为一个 **thenable** 。所有的 Promise 都是 thenable ，反之则未必成立。

传递给 `then()` 的两个参数都是可选的，因此你可以监听完成与拒绝的任意组合形式。例如，研究这组 `then()` 调用：

```
let promise = readFile("example.txt");

promise.then(function(contents) {
    // 完成
    console.log(contents);
}, function(err) {
    // 拒绝
    console.error(err.message);
});

promise.then(function(contents) {
    // 完成
    console.log(contents);
});

promise.then(null, function(err) {
    // 拒绝
    console.error(err.message);
}); 
```

这三个 `then()` 调用都操作在同一个 Promise 上。第一个调用同时监听了完成与失败；第二个调用只监听了完成，错误不会被报告；第三个则只监听了拒绝，并不报告成功信息。

Promis 也具有一个 `catch()` 方法，其行为等同于只传递拒绝处理函数给 `then()` 。例如，以下的 `catch()` 与 `then()` 调用是功能等效的。

```
promise.catch(function(err) {
    // 拒绝
    console.error(err.message);
});

// 等同于：

promise.then(null, function(err) {
    // 拒绝
    console.error(err.message);
}); 
```

`then()` 与 `catch()` 背后的意图是让你组合使用它们来正确处理异步操作的结果。此系统要优于事件与回调函数，因为它让操作是成功还是失败变得完全清晰（事件模式倾向于在出错时不被触发，而在回调函数模式中你必须始终记得检查错误参数）。只需知道若你未给 Promise 附加拒绝处理函数，所有的错误就会静默发生。建议始终附加一个拒绝处理函数，即使该处理程序只是用于打印错误日志。

即使完成或拒绝处理函数在 Promise 已经被解决之后才添加到作业队列，它们仍然会被执行。这允许你随时添加新的完成或拒绝处理函数，并保证它们会被调用。例如：

```
let promise = readFile("example.txt");

// 原始的完成处理函数
promise.then(function(contents) {
    console.log(contents);

    // 现在添加另一个
    promise.then(function(contents) {
        console.log(contents);
    });
}); 
```

在此代码中，完成处理函数又为同一个 Promise 添加了另一个完成处理函数。这个 Promise 此刻已经完成了，因此新的处理程序就被添加到任务队列，并在就绪时（前面的作业执行完毕后）被调用。拒绝处理函数使用同样方式工作。

> 每次调用 `then()` 或 `catch()` 都会创建一个新的作业，它会在 Promise 已决议时被执行。但这些作业最终会进入一个完全为 Promise 保留的作业队列。这个独立队列的确切细节对于理解如何使用 Promise 是不重要的，你只需理解作业队列通常来说是如何工作的。

#### 创建未决的 Promise

新的 Promise 使用 `Promise` 构造器来创建。此构造器接受单个参数：一个被称为**执行器**（ **executor** ）的函数，包含初始化 Promise 的代码。该执行器会被传递两个名为 `resolve()` 与 `reject()` 的函数作为参数。 `resolve()` 函数在执行器成功结束时被调用，用于示意该 Promise 已经准备好**被决议**（ **resolved** ），而 `reject()` 函数则表明执行器的操作已失败。

此处有个范例，在 Node.js 中使用了一个 Promise ，实现了本章前面的 `readFile()` 函数：

```
// Node.js 范例

let fs = require("fs");

function readFile(filename) {
    return new Promise(function(resolve, reject) {

        // 触发异步操作
        fs.readFile(filename, { encoding: "utf8" }, function(err, contents) {

            // 检查错误
            if (err) {
                reject(err);
                return;
            }

            // 读取成功
            resolve(contents);

        });
    });
}

let promise = readFile("example.txt");

// 同时监听完成与拒绝
promise.then(function(contents) {
    // 完成
    console.log(contents);
}, function(err) {
    // 拒绝
    console.error(err.message);
}); 
```

在此例中， Node.js 原生的 `fs.readFile()` 异步调用被包装在一个 Promise 中。执行器要么传递错误对象给 `reject()` 函数，要么传递文件内容给 `resolve()` 函数。

要记住执行器会在 `readFile()` 被调用时立即运行。当 `resolve()` 或 `reject()` 在执行器内部被调用时，一个作业被添加到作业队列中，以便**决议**（ **resolve** ）这个 Promise 。这被称为**作业调度**（ **job scheduling** ），若你曾用过 `setTimeout()` 或 `setInterval()` 函数，那么应该已经熟悉这种方式。在作业调度中，你添加新作业到队列中是表示：“不要立刻执行这个作业，但要在稍后执行它”。例如， `setTimeout()` 函数能让你指定一个延迟时间，延迟之后作业才会被添加到队列：

```
// 在 500 毫秒之后添加此函数到作业队列
setTimeout(function() {
    console.log("Timeout");
}, 500);

console.log("Hi!"); 
```

此代码安排一个作业在 500 毫秒之后被添加到作业队列。此处两个 `console.log()` 调用产生了以下输出：

```
Hi!
Timeout 
```

多亏这 500 毫秒的延迟，被传递给 `setTimeout()` 的匿名函数的输出，被排在了 `console.log("Hi!")` 输出之后。

> **译注：**实际上前面范例中的输出顺序与 500 毫秒的延时没有关系，而与 `setTimeout()` 的机制有关。我们可以把延时改为 0 ，依然会得到相同的结果：
> 
> ```
> // 在 0 毫秒之后添加此函数到作业队列
> setTimeout(function() {
>    console.log("Timeout");
> }, 0);
> 
> console.log("Hi!"); 
> ```
> 
> 输出结果会保持不变。 `setTimeout()` 确实有延时效果，但原书的例子不当，没有完全说清其中的机制。

Promise 工作方式与之相似。 Promise 的执行器会立即执行，早于源代码中在其之后的任何代码。例如：

```
let promise = new Promise(function(resolve, reject) {
    console.log("Promise");
    resolve();
});

console.log("Hi!"); 
```

此代码的输出结果为：

```
Promise
Hi! 
```

调用 `resolve()` 触发了一个异步操作。传递给 `then()` 与 `catch()` 的函数会异步地被执行，并且它们也被添加到了作业队列（先进队列再执行）。此处有个例子：

```
let promise = new Promise(function(resolve, reject) {
    console.log("Promise");
    resolve();
});

promise.then(function() {
    console.log("Resolved.");
});

console.log("Hi!"); 
```

此例的输出结果为：

```
Promise
Hi!
Resolved 
```

注意：尽管对 `then()` 的调用出现在 `console.log("Hi!")` 代码行之前，它实际上稍后才会执行（与执行器中那行 `"Promise"` 不同）。这是因为完成处理函数与拒绝处理函数总是会在执行器的操作结束后被添加到作业队列的尾部。

#### 创建已决的 Promise

基于 Promise 执行器行为的动态本质， `Promise` 构造器就是创建未决的 Promise 的最好方式。但若你想让一个 Promise 代表一个已知的值，那么安排一个单纯传值给 `resolve()` 函数的作业并没有意义。相反，有两种方法可使用指定值来创建已决的 Promise 。

##### 使用 Promise.resolve()

`Promise.resolve()` 方法接受单个参数并会返回一个处于完成态的 Promise 。这意味着没有任何作业调度会发生，并且你需要向 Promise 添加一个或更多的完成处理函数来提取这个参数值。例如：

```
let promise = Promise.resolve(42);

promise.then(function(value) {
    console.log(value);         // 42
}); 
```

此代码创建了一个已完成的 Promise ，因此完成处理函数就接收到 42 作为 `value` 参数。若一个拒绝处理函数被添加到此 Promise ，该拒绝处理函数将永不会被调用，因为此 Promise 绝不可能再是拒绝态。

##### 使用 Promise.reject()

你也可以使用 `Promise.reject()` 方法来创建一个已拒绝的 Promise 。此方法像 `Promise.resolve()` 一样工作，区别是被创建的 Promise 处于拒绝态，如下：

```
let promise = Promise.reject(42);

promise.catch(function(value) {
    console.log(value);         // 42
}); 
```

任何附加到这个 Promise 的拒绝处理函数都将会被调用，而完成处理函数则不会执行。

> 若你传递一个 Promise 给 `Promise.resolve()` 或 `Promise.reject()` 方法，该 Promise 会不作修改原样返回。

* * *

> **译注：** 经过测试，在几大浏览器中都存在与上一句话不符的情况。
> 
> 1.  若传入的 Promise 为挂起态，则 `Promise.resolve()` 调用会将该 Promise 原样返回。此后，若决议原 Promise ，在 `then()` 中可以接收到原例中的参数 `42` ；而若拒绝原 Promise ，则在 `catch()` 中可以接收到参数 `42` 。 但 `Promise.reject()` 调用则会对原先的 Promise 重新进行包装，对其使用 `catch()` 可以捕捉到错误，处理函数中的 `value` 参数不会是数值 `42` ，而是原先处于挂起态的 Promise 。
> 2.  若传入的 Promise 为完成态，则 `Promise.resolve()` 调用会将该 Promise 原样返回，在 `then()` 中可以接收到原例中的参数 `42` 。 但 `Promise.reject()` 调用则会对原先的 Promise 重新进行包装，对其使用 `catch()` 可以捕捉到错误，处理函数中的 `value` 参数不会是数值 `42` ，而是原先处于完成态的 Promise 。
> 3.  若传入的 Promise 为拒绝态，则 `Promise.reject()` 调用会将该 Promise 原样返回，在 `catch()` 中可以接收到参数 `42` 。 但 `Promise.resolve()` 调用则会对原先的 Promise 重新进行包装，对其使用 `then()` 可以进行完成处理，处理函数中的 `value` 参数不是 `42` ，而是原先处于拒绝态的 Promise 。也就是说此时的情况与上一种情况相反。
> 
> 总结：对挂起态或完成态的 Promise 使用 `Promise.resolve()` 没问题，会返回原 Promise ；对拒绝态的 Promise 使用 `Promise.reject()` 也没问题。而除此之外的情况全都会在原 Promise 上包装出一个新的 Promise 。

##### 非 Promise 的 Thenable

`Promise.resolve()` 与 `Promise.reject()` 都能接受非 Promise 的 thenable 作为参数。当传入了非 Promise 的 thenable 时，这些方法会创建一个新的 Promise ，此 Promise 会在 `then()` 函数之后被调用。

当一个对象拥有一个能接受 `resolve` 与 `reject` 参数的 `then()` 方法，该对象就会被认为是一个非 Promise 的 thenable ，就像这样：

```
let thenable = {
    then: function(resolve, reject) {
        resolve(42);
    }
}; 
```

此例中的 `thenable` 对象，除了 `then()` 方法之外没有任何与 Promise 相关的特征。你可以调用 `Promise.resolve()` 来将 `thenable` 转换为一个已完成的 Promise ：

```
let thenable = {
    then: function(resolve, reject) {
        resolve(42);
    }
};

let p1 = Promise.resolve(thenable);
p1.then(function(value) {
    console.log(value);     // 42
}); 
```

在此例中， `Promise.resolve()` 调用了 `thenable.then()` ，确定了这个 `thenable` 的 Promise 状态：由于 `resolve(42)` 在 `thenable.then()` 方法内部被调用，这个 `thenable` 的 Promise 状态也就被设为已完成。一个名为 `p1` 的新 Promise 被创建为完成态，并从 `thenable` 中接收到了值（此处为 42 ），于是 `p1` 的完成处理函数就接收到一个值为 42 的参数。

使用 `Promise.resolve()` ，同样还能从一个 thenable 创建一个已拒绝的 Promise ：

```
let thenable = {
    then: function(resolve, reject) {
        reject(42);
    }
};

let p1 = Promise.resolve(thenable);
p1.catch(function(value) {
    console.log(value);     // 42
}); 
```

此例类似于上例，区别是此处的 `thenable` 被拒绝了。当 `thenable.then()` 执行时，一个处于拒绝态的新 Promise 被创建，并伴随着一个值（ 42 ）。这个值此后会被传递给 `p1` 的拒绝处理函数。

`Promise.resolve()` 与 `Promise.reject()` 用类似方式工作，让你能轻易处理非 Promise 的 thenable 。在 Promise 被引入 ES6 之前，许多库都使用了 thenable ，因此将 thenable 转换为正规 Promise 的能力就非常重要了，能对之前已存在的库提供向下兼容。当你不能确定一个对象是否是 Promise 时，将该对象传递给 `Promise.resolve()` 或 `Promise.reject()` （取决于你的预期结果）是能找出的最好方式，因为传入真正的 Promise 只会被直接传递出来，并不会被修改（**但请注意前面译注提到的特殊情况**）。

#### 执行器错误

如果在执行器内部抛出了错误，那么 Promise 的拒绝处理函数就会被调用。例如：

```
let promise = new Promise(function(resolve, reject) {
    throw new Error("Explosion!");
});

promise.catch(function(error) {
    console.log(error.message);     // "Explosion!"
}); 
```

在此代码中，执行器故意抛出了一个错误。此处在每个执行器之内并没有显式的 `try-catch` ，因此错误就被捕捉并传递给了拒绝处理函数。这个例子等价于：

```
let promise = new Promise(function(resolve, reject) {
    try {
        throw new Error("Explosion!");
    } catch (ex) {
        reject(ex);
    }
});

promise.catch(function(error) {
    console.log(error.message);     // "Explosion!"
}); 
```

执行器处理程序捕捉了抛出的任何错误，以简化这种常见处理。但在执行器内抛出的错误仅当存在拒绝处理函数时才会被报告，否则这个错误就会被隐瞒。这在开发者早期使用 Promise 的时候是一个问题，但 JS 环境通过提供钩子（ hook ）来捕捉被拒绝的 Promise ，从而解决了此问题。

### 全局的 Promise 拒绝处理

Promise 最有争议的方面之一就是：当一个 Promise 被拒绝时若缺少拒绝处理函数，就会静默失败。有人认为这是规范中最大的缺陷，因为这是 JS 语言所有组成部分中唯一不让错误清晰可见的。

由于 Promise 的本质，判断一个 Promise 的拒绝是否已被处理并不直观。例如，研究以下示例：

```
let rejected = Promise.reject(42);

// 在此刻 rejected 不会被处理

// 一段时间后……
rejected.catch(function(value) {
    // 现在 rejected 已经被处理了
    console.log(value);
}); 
```

无论 Promise 是否已被解决，你都可以在任何时候调用 `then()` 或 `catch()` 并使它们正确工作，这导致很难准确知道一个 Promise 何时会被处理。此例中的 Promise 被立刻拒绝，但它后来才被处理。

虽然下个版本的 ES 可能会处理此问题，不过浏览器与 Node.js 已经实施了变更来解决开发者的这个痛点。这些变更不是 ES6 规范的一部分，但却是使用 Promise 时的宝贵工具。

#### Node.js 的拒绝处理

在 Node.js 中， `process` 对象上存在两个关联到 Promise 的拒绝处理的事件：

*   `unhandledRejection` ：当一个 Promise 被拒绝、而在事件循环的一个轮次中没有任何拒绝处理函数被调用，该事件就会被触发；
*   `rejectionHandled` ：若一个 Promise 被拒绝、并在事件循环的一个轮次之后再有拒绝处理函数被调用，该事件就会被触发。

这两个事件旨在共同帮助识别已被拒绝但未曾被处理 promise。

`unhandledRejection` 事件处理函数接受的参数是拒绝原因（常常是一个错误对象）以及已被拒绝的 Promise 。以下代码展示了 `unhandledRejection` 的应用：

```
let rejected;

process.on("unhandledRejection", function(reason, promise) {
    console.log(reason.message);            // "Explosion!"
    console.log(rejected === promise);      // true
});

rejected = Promise.reject(new Error("Explosion!")); 
```

此例创建了一个带有错误对象的已被拒绝的 Promise ，并监听了 `unhandledRejection` 事件。事件处理函数接收了该错误对象作为第一个参数，原 Promise 则是第二个参数。

`rejectionHandled` 事件处理函数则只有一个参数，即已被拒绝的 Promise 。例如：

```
let rejected;

process.on("rejectionHandled", function(promise) {
    console.log(rejected === promise);              // true
});

rejected = Promise.reject(new Error("Explosion!"));

// 延迟添加拒绝处理函数
setTimeout(function() {
    rejected.catch(function(value) {
        console.log(value.message);     // "Explosion!"
    });
}, 1000); 
```

此处的 `rejectionHandled` 事件在拒绝处理函数最终被调用时触发。若在 `rejected` 被创建后直接将拒绝处理函数附加到它上面，那么此事件就不会被触发。因为立即附加的拒绝处理函数在 `rejected` 被创建的事件循环的同一个轮次内就会被调用，这样 `rejectionHandled` 就不会起作用。

为了正确追踪潜在的未被处理的拒绝，使用 `rejectionHandled` 与 `unhandledRejection` 事件就能保持包含这些 Promise 的一个列表，之后等待一段时间再检查此列表。例如：

```
let possiblyUnhandledRejections = new Map();

// 当一个拒绝未被处理，将其添加到 map
process.on("unhandledRejection", function(reason, promise) {
    possiblyUnhandledRejections.set(promise, reason);
});

process.on("rejectionHandled", function(promise) {
    possiblyUnhandledRejections.delete(promise);
});

setInterval(function() {

    possiblyUnhandledRejections.forEach(function(reason, promise) {
        console.log(reason.message ? reason.message : reason);

        // 做点事来处理这些拒绝
        handleRejection(promise, reason);
    });

    possiblyUnhandledRejections.clear();

}, 60000); 
```

对于未处理的拒绝，这只是个简单追踪器。它使用了一个 Map 来储存 Promise 及其拒绝原因，每个 Promise 都是键，而它的拒绝原因就是相关的值。每当 `unhandledRejection` 被触发， Promise 及其拒绝原因就会被添加到此 Map 中。而每当 `rejectionHandled` 被触发，已被处理的 Promise 就会从这个 Map 中被移除。这样一来， `possiblyUnhandledRejections` 就会随着事件的调用而扩展或收缩。 `setInterval()` 的调用会定期检查这个列表，查看可能未被处理的拒绝，并将其信息输出到控制台（在现实情况下，你可能会想做点别的事情，以便记录或处理该拒绝）。此例使用了一个 Map 而不是 Weak Map ，这是因为你需要定期检查此 Map 来查看哪些 Promise 存在，而这是使用 Weak Map 所无法做到的。

尽管此例仅针对 Node.js ，但浏览器也实现了类似的机制来将未处理的拒绝通知给开发者。

#### 浏览器的拒绝处理

浏览器同样能触发两个事件，来帮助识别未处理的拒绝。这两个事件会被 `window` 对象触发，并完全等效于 Node.js 的相关事件：

*   `unhandledrejection` ：当一个 Promise 被拒绝、而在事件循环的一个轮次中没有任何拒绝处理函数被调用，该事件就会被触发；
*   `rejectionHandled` ：若一个 Promise 被拒绝、并在事件循环的一个轮次之后再有拒绝处理函数被调用，该事件就会被触发。

Node.js 的实现会传递分离的参数给事件处理函数，而浏览器事件的处理函数则只会接收到包含下列属性的一个对象：

*   `type` ： 事件的名称（ `"unhandledrejection"` 或 `"rejectionhandled"` ）；
*   `promise` ：被拒绝的 Promise 对象；
*   `reason` ： Promise 中的拒绝值（拒绝原因）。

浏览器的实现中存在的另一个差异就是：拒绝值（ `reason` ）在两种事件中都可用。例如：

```
let rejected;

window.onunhandledrejection = function(event) {
    console.log(event.type);                    // "unhandledrejection"
    console.log(event.reason.message);          // "Explosion!"
    console.log(rejected === event.promise);    // true
};

window.onrejectionhandled = function(event) {
    console.log(event.type);                    // "rejectionhandled"
    console.log(event.reason.message);          // "Explosion!"
    console.log(rejected === event.promise);    // true
};

rejected = Promise.reject(new Error("Explosion!")); 
```

此代码使用了 DOM 0 级写法的 `onunhandledrejection` 与 `onrejectionhandled` ，对两个事件处理函数都进行了赋值（若你喜欢，也可以使用 `addEventListener("unhandledrejection")` 与 `addEventListener("rejectionhandled")` ）。每个事件处理函数都接收一个事件对象，其中包含与被拒绝的 Promise 有关的信息， `type` 、 `promise` 与 `reason` 属性都可用。

以下代码在浏览器中追踪未被处理的拒绝，与 Node.js 的代码非常相似：

```
let possiblyUnhandledRejections = new Map();

// 当一个拒绝未被处理，将其添加到 map
window.onunhandledrejection = function(event) {
    possiblyUnhandledRejections.set(event.promise, event.reason);
};

window.onrejectionhandled = function(event) {
    possiblyUnhandledRejections.delete(event.promise);
};

setInterval(function() {

    possiblyUnhandledRejections.forEach(function(reason, promise) {
        console.log(reason.message ? reason.message : reason);

        // 做点事来处理这些拒绝
        handleRejection(promise, reason);
    });

    possiblyUnhandledRejections.clear();

}, 60000); 
```

这个实现与 Node.js 的实现几乎一模一样。使用了相同方法在 Map 中存储 Promise 及其拒绝值，并在此后进行检查。唯一真正的区别就是在事件处理函数中信息是从何处被提取出来的。

处理 Promise 的拒绝可能很麻烦，但你才刚开始见识 Promise 实际上到底有多强大。现在是时候更进一步了——把几个 promises 串联在一起使用。

### 串联 Promise

到此为止， Promise 貌似不过是个对组合使用回调函数与 `setTimeout()` 函数的增量改进，然而 Promise 的内容远比表面上所看到的更多。更确切地说，存在多种方式来将 Promise 串联在一起，以完成更复杂的异步行为。

每次对 `then()` 或 `catch()` 的调用实际上创建并返回了另一个 Promise ，仅当前一个 Promise 被完成或拒绝时，后一个 Promise 才会被决议。研究以下例子：

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

p1.then(function(value) {
    console.log(value);
}).then(function() {
    console.log("Finished");
}); 
```

此代码输出：

```
42
Finished 
```

对 `p1.then()` 的调用返回了第二个 Promise ，又在这之上调用了 `then()` 。仅当第一个 Promise 已被决议后，第二个 `then()` 的完成处理函数才会被调用。假若你在此例中不使用串联，它看起来就会是这样：

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = p1.then(function(value) {
    console.log(value);
})

p2.then(function() {
    console.log("Finished");
}); 
```

在这个无串联版本的代码中， `p1.then()` 的结果被存储在 `p2` 中，并且随后 `p2.then()` 被调用，以添加最终的完成处理函数。正如你可能已经猜到的，对于 `p2.then()` 的调用也返回了一个 Promise ，本例只是未使用此 Promise 。

#### 捕获错误

Promise 链允许你捕获前一个 Promise 的完成或拒绝处理函数中发生的错误。例如：

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

p1.then(function(value) {
    throw new Error("Boom!");
}).catch(function(error) {
    console.log(error.message);     // "Boom!"
}); 
```

在此代码中， `p1` 的完成处理函数抛出了一个错误，链式调用指向了第二个 Promise 上的 `catch()` 方法，能通过此拒绝处理函数接收前面的错误。若是一个拒绝处理函数抛出了错误，情况也是一样：

```
let p1 = new Promise(function(resolve, reject) {
    throw new Error("Explosion!");
});

p1.catch(function(error) {
    console.log(error.message);     // "Explosion!"
    throw new Error("Boom!");
}).catch(function(error) {
    console.log(error.message);     // "Boom!"
}); 
```

此处的执行器抛出了一个错误，就触发了 `p1` 这个 Promise 的拒绝处理函数，该处理函数随后抛出了另一个错误，并被第二个 Promise 的拒绝处理函数所捕获。链式 Promise 调用能察觉到链中其他 Promise 中的错误。

为了确保能正确处理任意可能发生的错误，应当始终在 Promise 链尾部添加拒绝处理函数。

#### 在 Promise 链中返回值

Promise 链的另一重要方面是能从一个 Promise 传递数据给下一个 Promise 的能力。传递给执行器中的 `resolve()` 处理函数的参数，会被传递给对应 Promise 的完成处理函数，这点你前面已看到过了。你可以指定完成处理函数的返回值，以便沿着一个链继续传递数据。例如：

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

p1.then(function(value) {
    console.log(value);         // "42"
    return value + 1;
}).then(function(value) {
    console.log(value);         // "43"
}); 
```

`p1` 的完成处理函数在被执行时返回了 `value + 1` 。由于 `value` 的值为 42 （来自执行器），此完成处理函数就返回了 43 。这个值随后被传递给第二个 Promise 的完成处理函数，并被其输出到控制台。

你能对拒绝处理函数做相同的事。当一个拒绝处理函数被调用时，它也能返回一个值。如果这么做，该值会被用于完成下一个 Promise ，就像这样：

```
let p1 = new Promise(function(resolve, reject) {
    reject(42);
});

p1.catch(function(value) {
    // 第一个完成处理函数
    console.log(value);         // "42"
    return value + 1;
}).then(function(value) {
    // 第二个完成处理函数
    console.log(value);         // "43"
}); 
```

此处的执行器使用 42 调用了 `reject()` ，该值被传递到这个 Promise 的拒绝处理函数中，从中又返回了 `value + 1` 。尽管后一个返回值是来自拒绝处理函数，它仍然被用于链中下一个 Promise 的完成处理函数。若有必要，一个 Promise 的失败可以通过传递返回值来恢复整个 Promise 链。

#### 在 Promise 链中返回 Promise

从完成或拒绝处理函数中返回一个基本类型值，能够在 Promise 之间传递数据，但若你返回的是一个对象呢？若该对象是一个 Promise ，那么需要采取一个额外步骤来决定如何处理。研究以下例子：

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

p1.then(function(value) {
    // 第一个完成处理函数
    console.log(value);     // 42
    return p2;
}).then(function(value) {
    // 第二个完成处理函数
    console.log(value);     // 43
}); 
```

在此代码中， `p1` 安排了一个决议 42 的作业， `p1` 的完成处理函数返回了一个已处于决议态的 Promise ： `p2` 。由于 `p2` 已被完成，第二个完成处理函数就被调用了。而若 `p2` 被拒绝，会调用拒绝处理函数（如果存在的话），而不调用第二个完成处理函数。

关于此模式需认识的首要重点是第二个完成处理函数并未被添加到 `p2` 上，而是被添加到第三个 Promise 。正因为此，上个例子就等价于：

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

let p3 = p1.then(function(value) {
    // 第一个完成处理函数
    console.log(value);     // 42
    return p2;
});

p3.then(function(value) {
    // 第二个完成处理函数
    console.log(value);     // 43
}); 
```

此处清楚说明了第二个完成处理函数被附加给 `p3` 而不是 `p2` 。这是一个细微但重要的区别，因为若 `p2` 被拒绝，则第二个完成处理函数就不会被调用。例如：

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    reject(43);
});

p1.then(function(value) {
    // 第一个完成处理函数
    console.log(value);     // 42
    return p2;
}).then(function(value) {
    // 第二个完成处理函数
    console.log(value);     // 永不被调用
}); 
```

在此例中，由于 `p2` 被拒绝了，第二个完成处理函数就永不被调用。不过你可以改为对其附加一个拒绝处理函数：

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    reject(43);
});

p1.then(function(value) {
    // 第一个完成处理函数
    console.log(value);     // 42
    return p2;
}).catch(function(value) {
    // 拒绝处理函数
    console.log(value);     // 43
}); 
```

此处 `p2` 被拒绝，导致拒绝处理函数被调用，来自 `p2` 的拒绝值 43 会被传递给拒绝处理函数。

从完成或拒绝处理函数中返回 thenable ，不会对 Promise 执行器何时被执行有所改变。第一个被定义的 Promise 将会首先运行它的执行器，接下来才轮到第二个 Promise 的执行器执行，以此类推。返回 thenable 只是让你能在 Promise 结果之外定义附加响应。你能通过在完成处理函数中创建一个新的 Promise ，来推迟完成处理函数的执行。例如：

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

p1.then(function(value) {
    console.log(value);     // 42

    // 创建一个新的 promise
    let p2 = new Promise(function(resolve, reject) {
        resolve(43);
    });

    return p2
}).then(function(value) {
    console.log(value);     // 43
}); 
```

在此例中，一个新的 Promise 在 `p1` 的完成处理函数中被创建。这意味着直到 `p2` 被完成之后，第二个完成处理函数才会执行。若你想等待前面的 Promise 被解决，之后才去触发另一个 Promise ，那么这种模式就非常有用。

### 响应多个 Promise

本章至今的每个例子在同一时刻都只响应一个 Promise 。然而有时你会想监视多个 Promise 的进程，以便决定下一步行动。 ES6 提供了能监视多个 Promise 的两个方法： `Promise.all()` 与 `Promise.race()` 。

#### Promise.all() 方法

`Promise.all()` 方法接收单个可迭代对象（如数组）作为参数，并返回一个 Promise 。这个可迭代对象的元素都是 Promise ，只有在它们都完成后，所返回的 Promise 才会被完成。例如：

> 译注：原文在此处有貌似重复的描述，相似的话语用 被决议（ resolved ）、 被完成（ fulfilled ）这两个术语说了两次，而这两个词在 Promise 中基本是同一个意思，因此译文删掉了其中一句。

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

let p3 = new Promise(function(resolve, reject) {
    resolve(44);
});

let p4 = Promise.all([p1, p2, p3]);

p4.then(function(value) {
    console.log(Array.isArray(value));  // true
    console.log(value[0]);              // 42
    console.log(value[1]);              // 43
    console.log(value[2]);              // 44
}); 
```

此处前面的每个 Promise 都用一个数值进行了决议，对 `Promise.all()` 的调用创建了新的 Promise `p4` ，在 `p1` 、 `p2` 与 `p3` 都被完成后， `p4` 最终会也被完成。传递给 `p4` 的完成处理函数的结果是一个包含每个决议值（ 42 、 43 与 44 ）的数组，这些值的存储顺序保持了待决议的 Promise 的顺序（与完成的先后顺序无关），因此你可以将结果匹配到每个 Promise 。

若传递给 `Promise.all()` 的任意 Promise 被拒绝了，那么方法所返回的 Promise 就会立刻被拒绝，而不必等待其他的 Promise 结束：

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    reject(43);
});

let p3 = new Promise(function(resolve, reject) {
    resolve(44);
});

let p4 = Promise.all([p1, p2, p3]);

p4.catch(function(value) {
    console.log(Array.isArray(value))   // false
    console.log(value);                 // 43
}); 
```

在此例中， `p2` 被使用数值 43 进行了拒绝，则 `p4` 的拒绝处理函数就立刻被调用，而不会等待 `p1` 或 `p3` 结束执行（它们仍然会各自结束执行，只是 `p4` 不等它们）。

拒绝处理函数总会接收到单个值，而不是一个数组，该值就是被拒绝的 Promise 所返回的拒绝值。本例中的拒绝处理函数被传入了 43 ，反映了来自 `p2` 的拒绝。

#### Promise.race() 方法

`Promise.race()` 提供了监视多个 Promise 的一个稍微不同的方法。此方法也接受一个包含需监视的 Promise 的可迭代对象，并返回一个新的 Promise ，但一旦来源 Promise 中有一个被解决，所返回的 Promise 就会立刻被解决。与等待所有 Promise 完成的 `Promise.all()` 方法不同，在来源 Promise 中任意一个被完成时， `Promise.race()` 方法所返回的 Promise 就能作出响应。例如：

```
let p1 = Promise.resolve(42);

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

let p3 = new Promise(function(resolve, reject) {
    resolve(44);
});

let p4 = Promise.race([p1, p2, p3]);

p4.then(function(value) {
    console.log(value);     // 42
}); 
```

在此代码中， `p1` 被创建为一个已完成的 Promise ，而其他的 Promise 则需要调度作业。 `p4` 的完成处理函数被使用数值 42 进行了调用，并忽略了其他的 Promise 。传递给 `Promise.race()` 的 Promise 确实在进行赛跑，看哪一个首先被解决。若胜出的 Promise 是被完成，则返回的新 Promise 也会被完成；而胜出的 Promise 若是被拒绝，则新 Promise 也会被拒绝。此处有个使用拒绝的范例：

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = Promise.reject(43);

let p3 = new Promise(function(resolve, reject) {
    resolve(44);
});

let p4 = Promise.race([p1, p2, p3]);

p4.catch(function(value) {
    console.log(value);     // 43
}); 
```

此处的 `p4` 被拒绝了，因为 `p2` 在 `Promise.race()` 被调用时已经处于拒绝态。尽管 `p1` 与 `p3` 都被完成，其结果仍然被忽略，因为这发生在 `p2` 被拒绝之后。

> **译注：**此处范例有误。
> 
> 在各个浏览器中的测试结果都是没有任何输出；而若为 `p4` 添加一个类似的完成处理函数，则会输出 `42` 。这表示在赛跑中胜出的是 `p1` 而不是 `p2`。
> 
> 如果要让此范例正确，应当在 `p1` 与 `p3` 内部的 `resolve()` 上添加延时处理。

### 继承 Promise

正像其他内置类型，你可将一个 Promise 用作派生类的基类。这允许你自定义变异的 Promise ，在内置 Promise 的基础上扩展功能。例如，假设你想创建一个可以使用 `success()` 与 `failure()` 方法的 Promise ，对常规的 `then()` 与 `catch()` 方法进行扩展，可以像下面这样创建该 Promise 类型：

```
class MyPromise extends Promise {

    // 使用默认构造器

    success(resolve, reject) {
        return this.then(resolve, reject);
    }

    failure(reject) {
        return this.catch(reject);
    }

}

let promise = new MyPromise(function(resolve, reject) {
    resolve(42);
});

promise.success(function(value) {
    console.log(value);             // 42
}).failure(function(value) {
    console.log(value);
}); 
```

在此例中， `MyPromise` 从 `Promise` 上派生出来，并拥有两个附加方法。 `success()` 方法模拟了 `resolve()` ， `failure()` 方法则模拟了 `reject()` 。

每个附加方法都使用了 `this` 来调用它所模拟的方法。派生的 Promise 函数与内置的 Promise 几乎一样，除了可以随你需要调用 `success()` 与 `failure()` 。

由于静态方法被继承了， `MyPromise.resolve()` 方法、 `MyPromise.reject()` 方法、 `MyPromise.race()` 方法与 `MyPromise.all()` 方法在派生的 Promise 上都可用。后两个方法的行为等同于内置的方法，但前两个方法则有轻微的不同。

`MyPromise.resolve()` 与 `MyPromise.reject()` 都会返回 `MyPromise` 的一个实例，无视传递进来的值的类型，这是由于这两个方法使用了 `Symbol.species` 属性（详见第九章）来决定需要返回的 Promise 的类型。若传递内置 Promise 给这两个方法，将会被决议或被拒绝，并且会返回一个新的 `MyPromise` ，以便绑定完成或拒绝处理函数。例如：

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = MyPromise.resolve(p1);
p2.success(function(value) {
    console.log(value);         // 42
});

console.log(p2 instanceof MyPromise);   // true 
```

此处的 `p1` 是一个内置的 Promise ，被传递给了 `MyPromise.resolve()` 方法。作为结果的 `p2` 是 `MyPromise` 的一个实例，来自 `p1` 的决议值被传递给了 `p2` 的完成处理函数。

若 `MyPromise` 的一个实例被传递给了 `MyPromise.resolve()` 或 `MyPromise.reject()` 方法，它会在未被决议的情况下就被直接返回。在其他情况下，这两个方法的行为都会等同于 `Promise.resolve()` 与 `Promise.reject()` 。

#### 异步任务运行

在第八章中，我介绍了生成器，并向你展示了如何使用它来运行异步任务，就像这样：

```
let fs = require("fs");

function run(taskDef) {

    // 创建迭代器，让它在别处可用
    let task = taskDef();

    // 开始任务
    let result = task.next();

    // 递归使用函数来保持对 next() 的调用
    function step() {

        // 如果还有更多要做的
        if (!result.done) {
            if (typeof result.value === "function") {
                result.value(function(err, data) {
                    if (err) {
                        result = task.throw(err);
                        return;
                    }

                    result = task.next(data);
                    step();
                });
            } else {
                result = task.next(result.value);
                step();
            }

        }
    }

    // 开始处理过程
    step();

}

// 定义一个函数来配合任务运行器使用

function readFile(filename) {
    return function(callback) {
        fs.readFile(filename, callback);
    };
}

// 运行一个任务

run(function*() {
    let contents = yield readFile("config.json");
    doSomethingWith(contents);
    console.log("Done");
}); 
```

此实现存在一些痛点。首先，将每个函数包裹在另一个函数内、再返回一个新函数，这是有点令人困惑的（这句话本身就已经够乱了）。其次，返回值为函数的情况下，没有任何方法可以区分它是否应当被作为任务运行器的回调函数。

借助 Promise ，你可以确保每个异步操作都返回一个 Promise ，从而大幅度简化并一般化异步处理，通用接口也意味着你可以大大减少异步代码。此处有一个简化任务运行器的方式：

```
let fs = require("fs");

function run(taskDef) {

    // 创建迭代器
    let task = taskDef();

    // 启动任务
    let result = task.next();

    // 递归使用函数来进行迭代
    (function step() {

        // 如果还有更多要做的
        if (!result.done) {

            // 决议一个 Promise ，让任务处理变简单
            let promise = Promise.resolve(result.value);
            promise.then(function(value) {
                result = task.next(value);
                step();
            }).catch(function(error) {
                result = task.throw(error);
                step();
            });
        }
    }());
}

// 定义一个函数来配合任务运行器使用

function readFile(filename) {
    return new Promise(function(resolve, reject) {
        fs.readFile(filename, function(err, contents) {
            if (err) {
                reject(err);
            } else {
                resolve(contents);
            }
        });
    });
}

// 运行一个任务

run(function*() {
    let contents = yield readFile("config.json");
    doSomethingWith(contents);
    console.log("Done");
}); 
```

在此版本的代码中，一个通用的 `run()` 函数执行了生成器来创建一个迭代器。它调用了 `task.next()` 来启动任务，并递归调用 `step()` 直到迭代完成。

在 `step()` 函数内部，如果还有更多工作要做，那么 `result.done` 的值会是 `false` ，此时 `result.value` 应当是一个 Promise ，不过调用 `Promise.resolve()` 只为预防未正确返回 Promise 的函数（记住： `Promise.resolve()` 在被传入任意 Promise 时只会直接将其传递回来，而不是 Promise 的参数则会被包装为 Promise ）。接下来，一个完成处理函数被添加以便提取该 Promise 值，并将该值传回迭代器。此后，在 `step()` 函数调用自身之前， `result` 被赋值为下一个 yield 的结果。

> **译注：**此处的
> 
> > `Promise.resolve()` 在被传入任意 Promise 时只会直接将其传递回来
> 
> 表述不够准确，详情参见前面的译注。

一个拒绝处理函数将任意拒绝结果存储在一个错误对象中。 `task.throw()` 方法将这个错误对象传回给迭代器，而若一个错误在任务中被捕获， `result` 也会被赋值为下一个 yield 的结果，这样 `step()` 最终在 `catch()` 内部就会被调用，以便继续任务执行。

`run()` 函数能运行任意使用 `yield` 来实现异步代码的生成器，而不会将 Promise （或回调函数）暴露给开发者。事实上，由于函数调用后的返回值总是会被转换为一个 Promise ，该函数甚至允许返回 Promise 之外的类型。这意味着同步与异步方法在使用 `yield` 时都会正常工作，并且你永不需要检查返回值是否为一个 Promise 。

唯一需要担心的是，要确保诸如 `readFile()` 的异步方法能返回一个正确标记其状态的 Promise 。对于 Node.js 内置的方法来说，这意味着你必须转换这些方法，让它们返回 Promise 而不是使用回调函数。

> **未来的异步任务运行**
> 
> 在我写这本书的时候，针对 JS 中的异步任务运行，为之引入简单语法的一项工作正在进行。此工作开展在 `await` 语法上，极度借鉴了上述以 Promise 为基础的例子。其基本理念是使用一个被 `async` 标记的函数（而非生成器），并在调用另一个函数时使用 `await` 而非 `yield` ，就像这样：
> 
> ```
> (async function() {
>   let contents = await readFile("config.json");
>   doSomethingWith(contents);
>   console.log("Done");
> }); 
> ```
> 
> 在 `function` 之前的 `async` 关键字标明了此函数使用异步方式运行。 `await` 关键字则表示对于 `readFile("config.json")` 的函数调用应返回一个 Promise ，若返回类型不对，则会将其包装为 Promise 。与上述 `run()` 的实现一致， `await` 会在 Promise 被拒绝的情况下抛出错误，否则它将返回该 Promise 被决议的值。最终结果是你可以将异步代码当作同步代码来书写，而无须为管理基于迭代器的状态机而付出额外开销。
> 
> `await` 语法预计将在 ES2017 （即 ES8 ）中被最终敲定。（译注：已被纳入 ES8 ）

### 总结

Promise 被设计用于改善 JS 中的异步编程，与事件及回调函数对比，在异步操作方面为你提供了更多的控制权与组合性。 Promise 调度被添加到 JS 引擎作业队列，以便稍后执行。不过此处有另一个作业队列追踪着 Promise 的完成与拒绝处理函数，以确保适当的执行。

Promise 具有三种状态：挂起、已完成、已拒绝。一个 Promise 起始于挂起态，并在成功时转为完成态，或在失败时转为拒绝态。在这两种情况下，处理函数都能被添加以表明 Promise 何时被解决。 `then()` 方法允许你绑定完成处理函数与拒绝处理函数，而 `catch()` 方法则只允许你绑定拒绝处理函数。

你能用多种方式将多个 Promise 串联在一起，并在它们之间传递信息。每个对 `then()` 的调用都创建并返回了一个新的 Promise ，在前一个 Promise 被决议时，新 Promise 也会被决议。 Promise 链可被用于触发对一系列异步事件的响应。你还能使用 `Promise.race()` 与 `Promise.all()` 来监视多个 Promise 的进程，并进行相应的响应。

组合使用生成器与 Promise 会让异步任务运行得更容易，这是由于 Promise 提供了异步操作可返回的一个通用接口。这样你就能使用生成器与 `yield` 运算符来等待异步响应，并作出适当的应答。

多数新的 web API 都基于 Promise 创建，并且你可以期待未来会有更多的效仿之作。