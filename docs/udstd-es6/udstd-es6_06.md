# 第五章 解构：更方便的数据访问

## 第五章 解构：更方便的数据访问

对象与数组的字面量在 JS 中是最常用的两种表示法，并且感谢流行的 JSON 数据格式，它们已成为这门语言中的格外重要的部分。定义对象与数组非常普遍，定义之后就能有条不紊地从这些结构中提取出相关信息。为了简化提取信息的任务， ES6 新增了**解构**（ **destructuring** ），这是将一个数据结构分解为更小的部分的过程。本章介绍如何在对象与数组上利用解构。

*   解构为何有用？
*   对象解构
    *   解构赋值
    *   默认值
    *   赋值给不同的本地变量名
    *   嵌套的对象解构
*   数组解构
    *   解构赋值
    *   默认值
    *   嵌套的解构
    *   剩余项
*   混合解构
*   参数解构
    *   解构的参数是必需的
    *   参数解构的默认值
*   总结

### 解构为何有用？

在 ES5 及更早版本中，从对象或数组中获取信息、并将特定数据存入本地变量，需要书写许多并且相似的代码。例如：

```js
let options = {
        repeat: true,
        save: false
    };

// 从对象中提取数据
let repeat = options.repeat,
    save = options.save; 
```

此代码提取了 `options` 对象的 `repeat` 与 `save` 值，并将其存在同名的本地变量上。虽然这段代码看起来简单，但想象一下若有大量变量需要处理，你就必须逐个为其赋值；并且若有一个嵌套的数据结构需要遍历以寻找信息，你可能会为了一点数据而挖掘整个结构。

这就是 ES6 为何要给对象与数组添加解构。当把数据结构分解为更小的部分时，从中提取你要的数据会变得容易许多。很多语言都能用精简的语法来实现解构，让它更易使用。 ES6 的解构实际使用的语法其实你早已熟悉，那就是对象与数组的字面量语法。

### 对象解构

对象解构语法在赋值语句的左侧使用了对象字面量，例如：

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo" 
```

在此代码中， `node.type` 的值被存储到 `type` 本地变量中， `node.name` 的值则存储到 `name` 变量中。此语法相同于第四章介绍的简写的属性初始化器。 `type` 与 `name` 标识符既声明了本地变量，也读取了对象的相应属性值。

> **不要遗忘初始化器**
> 
> 当使用解构来配合 `var` 、 `let` 或 `const` 来声明变量时，必须提供初始化器（即等号右边的值）。下面的代码都会因为缺失初始化器而抛出错误：
> 
> ```js
> // 语法错误！
> var { type, name };
> 
> // 语法错误！
> let { type, name };
> 
> // 语法错误！
> const { type, name }; 
> ```
> 
> `const` 总是要求有初始化器，即使没有使用解构的变量；而 `var` 与 `let` 则仅在使用解构时才作此要求。

##### 解构赋值

以上对象解构示例都用于变量声明。不过，也可以在赋值的时候使用解构。例如，你可能想在变量声明之后改变它们的值，如下所示：

```js
let node = {
        type: "Identifier",
        name: "foo"
    },
    type = "Literal",
    name = 5;

// 使用解构来分配不同的值
({ type, name } = node);

console.log(type);      // "Identifier"
console.log(name);      // "foo" 
```

在本例中， `type` 与 `name` 属性在声明时被初始化，而两个同名变量也被声明并初始化为不同的值。接下来一行使用了解构表达式，通过读取 `node` 对象来更改这两个变量的值。注意你必须用圆括号包裹解构赋值语句，这是因为暴露的花括号会被解析为代码块语句，而块语句不允许在赋值操作符（即等号）左侧出现。圆括号标示了里面的花括号并不是块语句、而应该被解释为表达式，从而允许完成赋值操作。

解构赋值表达式的值为表达式右侧（在 `=` 之后）的值。也就是说在任何期望有个值的位置都可以使用解构赋值表达式。例如，传递值给函数：

```js
let node = {
        type: "Identifier",
        name: "foo"
    },
    type = "Literal",
    name = 5;

function outputInfo(value) {
    console.log(value === node);        // true
}

outputInfo({ type, name } = node);

console.log(type);      // "Identifier"
console.log(name);      // "foo" 
```

`outputInfo()` 函数被使用一个解构赋值表达式进行了调用。该表达式计算结果为 `node` ，因为这就是表达式右侧的值。对 `type` 与 `name` 的赋值正常进行，同时 `node` 也被传入了 `outputInfo()` 函数。

> 当解构赋值表达式的右侧（ `=` 后面的表达式）的计算结果为 `null` 或 `undefined` 时，会抛出错误。因为任何读取 `null` 或 `undefined` 的企图都会导致“运行时”错误（ runtime error ）。

##### 默认值

当你使用解构赋值语句时，如果所指定的本地变量在对象中没有找到同名属性，那么该变量会被赋值为 `undefined` 。例如：

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name, value } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo"
console.log(value);     // undefined 
```

此代码定义了一个额外的本地变量 `value` ，并试图对其赋值。然而， `node` 对象中不存在同名属性，因此 `value` 不出预料地被赋值为 `undefined` 。

你可以选择性地定义一个默认值，以便在指定属性不存在时使用该值。若要这么做，需要在属性名后面添加一个等号并指定默认值，就像这样：

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name, value = true } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo"
console.log(value);     // true 
```

在此例中，变量 `value` 被指定了一个默认值 `true` ，只有在 `node` 的对应属性缺失、或对应的属性值为 `undefined` 的情况下，该默认值才会被使用。由于此处不存在 `node.value` 属性，变量 `value` 就使用了该默认值。这种工作方式很像函数参数的默认值（详见第三章）。

##### 赋值给不同的本地变量名

至此的每个解构赋值示例都使用了对象中的属性名作为本地变量的名称，例如，把 `node.type` 的值存储到 `type` 变量上。若想使用相同名称，这么做就没问题，但若你不想呢？ ES6 有一个扩展语法，允许你在给本地变量赋值时使用一个不同的名称，而且该语法看上去就像是使用对象字面量的非简写的属性初始化。这里有个示例：

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type: localType, name: localName } = node;

console.log(localType);     // "Identifier"
console.log(localName);     // "foo" 
```

此代码使用了解构赋值来声明 `localType` 与 `localName` 变量，分别获得了 `node.type` 与 `node.name` 属性的值。 `type: localType` 这种语法表示要读取名为 `type` 的属性，并把它的值存储在变量 `localType` 上。该语法实际上与传统对象字面量语法相反，传统语法将名称放在冒号左边、值放在冒号右边；而在本例中，则是名称在右边，需要进行值读取的位置则被放在了左边。

你也可以给变量别名添加默认值，依然是在本地变量名称后添加等号与默认值，例如：

```js
let node = {
        type: "Identifier"
    };

let { type: localType, name: localName = "bar" } = node;

console.log(localType);     // "Identifier"
console.log(localName);     // "bar" 
```

此处的 `localName` 变量拥有一个默认值 `"bar"` ，该变量最终被赋予了默认值，因为 `node.name` 属性并不存在。

到此为止，你已经看到如何处理属性值为基本类型值的对象的解构，而对象解构也可被用于从嵌套的对象结构（即：对象的属性可能还是一个对象）中提取属性值。

##### 嵌套的对象解构

使用类似于对象字面量的语法，可以深入到嵌套的对象结构中去提取你想要的数据。这里有个示例：

```js
let node = {
        type: "Identifier",
        name: "foo",
        loc: {
            start: {
                line: 1,
                column: 1
            },
            end: {
                line: 1,
                column: 4
            }
        }
    };

let { loc: { start }} = node;

console.log(start.line);        // 1
console.log(start.column);      // 1 
```

本例中的解构模式使用了花括号，表示应当下行到 `node` 对象的 `loc` 属性内部去寻找 `start` 属性。记住上一节介绍过的，每当有一个冒号在解构模式中出现，就意味着冒号之前的标识符代表需要检查的位置，而冒号右侧则是赋值的目标。当冒号右侧存在花括号时，表示目标被嵌套在对象的更深一层中。

你还能更进一步，在对象的嵌套解构中同样能为本地变量使用不同的名称：

```js
let node = {
        type: "Identifier",
        name: "foo",
        loc: {
            start: {
                line: 1,
                column: 1
            },
            end: {
                line: 1,
                column: 4
            }
        }
    };

// 提取 node.loc.start
let { loc: { start: localStart }} = node;

console.log(localStart.line);   // 1
console.log(localStart.column); // 1 
```

在此版本的代码中， `node.loc.start` 的值被存储在一个新的本地变量 `localStart` 上，解构模式可以被嵌套在任意深度的层级，并且在每个层级的功能都一样。

对象解构十分强大并有很多可用形式，而数组解构则提供了一些独特的能力，用于提取数组中的信息。

> **语法难点**
> 
> 使用嵌套的解构时需要小心，因为你可能无意中就创建了一个没有任何效果的语句。空白花括号在对象解构中是合法的，然而它不会做任何事。例如：
> 
> ```js
> // 没有变量被声明！
> let { loc: {} } = node; 
> ```
> 
> 在此语句中并未声明任何变量绑定。由于花括号在右侧， `loc` 被作为需检查的位置来使用，而不会创建变量绑定。这种情况仿佛是想用等号来定义一个默认值，但却被语法判断为想用冒号来定义一个位置。这种语法将来可能是非法的，然而现在它只是需要留意的一个疑难点。

### 数组解构

数组解构的语法看起来与对象解构非常相似，只是将对象字面量替换成了数组字面量。数组解构时，解构作用在数组内部的位置上，而不是作用在对象的具名属性上，例如：

```js
let colors = [ "red", "green", "blue" ];

let [ firstColor, secondColor ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green" 
```

此处数组解构从 `colors` 数组中取出了 `"red"` 与 `"green"` ，并将它们赋值给 `fristColor` 与 `secondColor` 变量。这些值被选择，是由于它们在数组中的位置，实际的变量名称是任意的（与位置无关）。任何没有在解构模式中明确指定的项都会被忽略。记住，数组本身并没有以任何方式被改变。

你也可以在解构模式中忽略一些项，并且只给感兴趣的项提供变量名。例如，若只想获取数组中的第三个元素，那么不必给前两项提供变量名。以下展示了这种方式：

```js
let colors = [ "red", "green", "blue" ];

let [ , , thirdColor ] = colors;

console.log(thirdColor);        // "blue" 
```

此代码使用了解构赋值来获取 `colors` 的第三个项。模式中 `thirdColor` 之前的逗号，是为数组前面的项提供的占位符。使用这种方法，你就可以轻易从数组任意位置取出值，而无须给其他项提供变量名。

> 与对象解构相似，在使用 `var` 、 `let` 、 `const` 进行数组解构时，你必须提供初始化器。

##### 解构赋值

你可以在赋值表达式中使用数组解构，但是与对象解构不同，不必将表达式包含在圆括号内，例如：

```js
let colors = [ "red", "green", "blue" ],
    firstColor = "black",
    secondColor = "purple";

[ firstColor, secondColor ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green" 
```

此代码中解构赋值的工作方式与上例相似，唯一区别是 `firstColor` 与 `secondColor` 变量已经被声明过了。大多数情况下，以上可能就是数组解构赋值你需要了解的全部内容，但其实还有一个很细微却又可能很有用的用法。

数组解构赋值有一个非常独特的用例，能轻易地互换两个变量的值。互换变量值在排序算法中十分常用，而在 ES5 中需要使用第三个变量作为临时变量，正如下例：

```js
// 在 ES5 中互换值
let a = 1,
    b = 2,
    tmp;

tmp = a;
a = b;
b = tmp;

console.log(a);     // 2
console.log(b);     // 1 
```

其中的 `tmp` 变量对于互换 `a` 与 `b` 的值来说是必要的。然而若使用数组解构赋值，就不再需要这个额外变量。以下演示了在 ES6 中如何互换变量值：

```js
// 在 ES6 中互换值
let a = 1,
    b = 2;

[ a, b ] = [ b, a ];

console.log(a);     // 2
console.log(b);     // 1 
```

本例中的数组解构赋值看起来如同镜像。赋值语句左侧（即等号之前）的解构模式正如其他数组解构的范例，右侧则是为了互换而临时创建的数组字面量。 `b` 与 `a` 的值分别被复制到临时数组的第一个与第二个位置，并对该数组进行解构，结果两个变量就互换了它们的值。

> 与对象解构赋值相同，若等号右侧的计算结果为 `null` 或 `undefined` ，那么数组解构赋值表达式也会抛出错误。

##### 默认值

数组解构赋值同样允许在数组任意位置指定默认值。当指定位置的项不存在、或其值为 `undefined` ，那么该默认值就会被使用。例如：

```js
let colors = [ "red" ];

let [ firstColor, secondColor = "green" ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green" 
```

此代码的 `colors` 数组只有一个项，因此没有能与 `secondColor` 匹配的项，又由于此处有个默认值， `secondColor` 的值就被设置为 `"green"` ，而不是 `undefined` 。

##### 嵌套的解构

与解构嵌套的对象相似，可以用类似的方式来解构嵌套的数组。在整个解构模式中插入另一个数组模式，解构操作就会下行到嵌套的数组中，就像这样：

```js
let colors = [ "red", [ "green", "lightgreen" ], "blue" ];

// 随后

let [ firstColor, [ secondColor ] ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green" 
```

此处的 `secondColor` 变量指向了 `colors` 数组中的 `"green"` 值，该项被包含在第二个数组中，因此解构模式就要把 `secondColor` 包裹上方括号。与对象解构相似，你也能使用任意深度的数组嵌套。

##### 剩余项

第三章介绍过函数的剩余参数，而数组解构有个类似的、名为**剩余项**（ **rest items** ）的概念，它使用 `...` 语法来将剩余的项目赋值给一个指定的变量，此处有个范例：

```js
let colors = [ "red", "green", "blue" ];

let [ firstColor, ...restColors ] = colors;

console.log(firstColor);        // "red"
console.log(restColors.length); // 2
console.log(restColors[0]);     // "green"
console.log(restColors[1]);     // "blue" 
```

`colors` 数组的第一项被赋值给了 `firstColor` 变量，而剩余的则赋值给了一个新的 `restColors` 数组； `restColors` 数组则有两个项： `"green"` 与 `"blue"`。若要取出特定项并要求保留剩余的值，则剩余项是非常有用的，但它还有另一个有用的功能。

方便地克隆数组在 JS 中是个明显被遗漏的功能。在 ES5 中开发者往往使用的是一个简单的方式，也就是用 `concat()` 方法来克隆数组，例如：

```js
// 在 ES5 中克隆数组
var colors = [ "red", "green", "blue" ];
var clonedColors = colors.concat();

console.log(clonedColors);      //"[red,green,blue]" 
```

尽管 `concat()` 方法的本意是合并两个数组，但不使用任何参数来调用此方法，就会获得原数组的一个克隆品。而在 ES6 中，你可以使用剩余项的语法来达到同样效果。实现如下：

```js
// 在 ES6 中克隆数组
let colors = [ "red", "green", "blue" ];
let [ ...clonedColors ] = colors;

console.log(clonedColors);      //"[red,green,blue]" 
```

在本例中，剩余项被用于将 `colors` 数组的值复制到 `clonedColors` 数组中。虽然从感觉上来说，使用这种技术未必让开发者复制数组的意图体现得比使用 `concat()` 方法更明显，但这依然是个值得关注的技巧。

剩余项必须是数组解构模式中最后的部分，之后不能再有逗号，否则就是语法错误。

### 混合解构

对象与数组解构能被用在一起，以创建更复杂的解构表达式。在对象与数组混合而成的结构中，这么做便能准确提取其中你想要的信息片段。例如：

```js
let node = {
        type: "Identifier",
        name: "foo",
        loc: {
            start: {
                line: 1,
                column: 1
            },
            end: {
                line: 1,
                column: 4
            }
        },
        range: [0, 3]
    };

let {
    loc: { start },
    range: [ startIndex ]
} = node;

console.log(start.line);        // 1
console.log(start.column);      // 1
console.log(startIndex);        // 0 
```

此代码将 `node.loc.start` 与 `node.range[0]` 提取出来，并将它们的值分别存储到 `start` 与 `startIndex` 中。要记住解构模式中的 `loc:` 与 `range:` 只是对应于 `node` 对象中属性的位置。混合使用对象与数组解构， `node` 的任何部分都能提取出来。对于从 JOSN 配置结构中抽取数据来说，这种方法尤其有用，因为它不需要探索整个结构。

### 参数解构

解构还有一个特别有用的场景，即在传递函数参数时。当 JS 的函数接收大量可选参数时，一个常用模式是创建一个 `options` 对象，其中包含了附加的参数，就像这样：

```js
// options 上的属性表示附加参数
function setCookie(name, value, options) {

    options = options || {};

    let secure = options.secure,
        path = options.path,
        domain = options.domain,
        expires = options.expires;

    // 设置 cookie 的代码
}

// 第三个参数映射到 options
setCookie("type", "js", {
    secure: true,
    expires: 60000
}); 
```

很多 JS 的库都包含了类似于此例的 `setCookie()` 函数。在此函数内， `name` 与 `value` 参数是必需的，而 `secure` 、 `path` 、 `domain` 与 `expires` 则不是。并且因为此处对于其余数据并没有顺序要求，将它们作为 `options` 对象的具名属性会更有效率，而无须列出一堆额外的具名参数。这种方法很有用，但无法仅通过查看函数定义就判断出函数所期望的输入，你必须阅读函数体的代码。

参数解构提供了更清楚地标明函数期望输入的替代方案。它使用对象或数组解构的模式替代了具名参数。要看到其实际效果，请查看下例中重写版本的 `setCookie()` 函数：

```js
function setCookie(name, value, { secure, path, domain, expires }) {

    // 设置 cookie 的代码
}

setCookie("type", "js", {
    secure: true,
    expires: 60000
}); 
```

此函数的行为类似上例，但此时第三个参数使用了解构来抽取必要的数据。现在对于 `setCookie()` 函数的使用者来说，解构参数之外的参数明显是必需的；而可选项目存在于额外的参数组中，这同样是非常明确的；同时，若使用了第三个参数，其中应当包含什么值当然也是极其明确的。解构参数在没有传递值的情况下类似于常规参数，它们会被设为 `undefined` 。

> 参数解构拥有此前你在本章已经学过的其他解构方式的所有能力。你可以在其中使用默认参数、混合解构，或使用与属性不同的变量名。

#### 解构的参数是必需的

参数解构有一个怪异点：默认情况下调用函数时未给参数解构传值会抛出错误。例如，用以下方式调用上例中的 `setCookie()` 函数就会出错：

One quirk of using destructured parameters is that, by default, an error is thrown when they are not provided in a function call. For instance, this call to the `setCookie()` function in the last example throws an error:

```js
// 出错！
setCookie("type", "js"); 
```

调用时第三个参数缺失了，因此它不出预料地等于 `undefined` 。这导致了一个错误，因为参数解构实际上只是解构声明的简写。当 `setCookie()` 函数被调用时， JS 引擎实际上是这么做的：

```js
function setCookie(name, value, options) {

    let { secure, path, domain, expires } = options;

    // 设置 cookie 的代码
} 
```

既然在赋值右侧的值为 `null` 或 `undefined` 时，解构会抛出错误，那么未向 `setCookie()` 函数传递第三个参数就同样会出错。

若你让解构的参数作为必选参数，那么上述行为并不会令人困扰。但若你要求它是可选的，可以给解构的参数提供默认值来处理这种行为，就像这样：

```js
function setCookie(name, value, { secure, path, domain, expires } = {}) {

    // ...
} 
```

此例为第三个参数提供了一个空对象作为其默认值。给解构的参数提供默认值，也就意味着若未向 `setCookie()` 函数传递第三个参数，则 `secure` 、 `path` 、 `domain` 与 `expires` 的值全都会是 `undefined` ，此时不会有错误被抛出。

#### 参数解构的默认值

你可以为参数解构提供可解构的默认值，就像在解构赋值时所做的那样，只需在其中每个参数后面添加等号并指定默认值即可。例如：

```js
function setCookie(name, value,
    {
        secure = false,
        path = "/",
        domain = "example.com",
        expires = new Date(Date.now() + 360000000)
    } = {}
) {

    // ...
} 
```

此代码中参数解构给每个属性都提供了默认值，所以你可以避免检查指定属性是否已被传入（以便在未传入时使用正确的值）。而整个解构的参数同样有一个默认值，即一个空对象，令该参数成为可选参数。这么做使得函数声明看起来比平时要复杂一些，但却是为了确保每个参数都有可用的值而付出的微小代价。

### 总结

解构使得在 JS 中操作对象与数组变得更容易。使用熟悉的对象字面量与数组字面量语法，可以将数据结构分离并只获取你感兴趣的信息。对象解构模式允许你从对象中进行提取，而数组模式则能用于数组。

对象与数组解构都能在属性或项未定义时为其提供默认值；在赋值表达式右侧的值为 `null` 或 `undefined` 时，两种模式都会抛出错误。你也可以在深层嵌套的数据结构中使用对象与数组解构，下行到该结构的任意深度。

使用 `var` 、 `let` 或 `const` 的解构声明来创建变量，就必须提供初始化器。解构赋值能替代其他赋值，并且允许你把值解构到对象属性或已存在的变量上。

参数解构使用解构语法作为函数的参数，让“选项”（ options ）对象更加透明。你实际感兴趣的数据可以与具名参数一并列出。解构的参数可以是对象模式、数组模式或混合模式，并且你能使用它们的所有特性。