# 第十章 增强的数组功能

## 第十章 增强的数组功能

数组是 JS 中的一种基本对象。 JS 的其他方面都随着时间的推移在进化，而数组却基本保持不变，直到 ES5 才添加了几个相关的方法让数组更易使用。 ES6 也添加了很多功能来继续强化数组，例如新的创建方法、几个有用的便捷方法，还增加了创建类型化数组（typed array）的能力。

*   创建数组
    *   Array.of() 方法
    *   Array.from() 方法
        *   映射转换
        *   在可迭代对象上使用
*   所有数组上的新方法
    *   find() 与 findIndex() 方法
    *   fill() 方法
    *   copyWithin() 方法
*   类型化数组
    *   数值数据类型
    *   数组缓冲区
    *   使用视图操作数组缓冲区
        *   获取视图信息
        *   读取与写入数据
        *   类型化数组即为视图
        *   创建特定类型视图
*   类型化数组与常规数组的相似点
    *   公共方法
    *   相同的迭代器
    *   of() 与 from() 方法
*   类型化数组与常规数组的区别
    *   行为差异
    *   遗漏的方法
    *   附加的方法
*   总结

### 创建数组

在 ES6 之前创建数组主要存在两种方式： `Array` 构造器与数组字面量写法。这两种方式都需要将数组的项分别列出，并且还要受到其他限制。将“类数组对象”（即：拥有数值类型索引与长度属性的对象）转换为数组也并不自由，经常需要书写额外的代码。为了使数组更易创建， ES6 新增了 `Array.of()` 与 `Array.from()` 方法。

#### Array.of() 方法

ES6 为数组新增创建方法的目的之一，是帮助开发者在使用 `Array` 构造器时避开 JS 语言的一个怪异点。调用 `new Array()` 构造器时，根据传入参数的类型与数量的不同，实际上会导致一些不同的结果，例如：

```js
let items = new Array(2);
console.log(items.length);          // 2
console.log(items[0]);              // undefined
console.log(items[1]);              // undefined

items = new Array("2");
console.log(items.length);          // 1
console.log(items[0]);              // "2"

items = new Array(1, 2);
console.log(items.length);          // 2
console.log(items[0]);              // 1
console.log(items[1]);              // 2

items = new Array(3, "2");
console.log(items.length);          // 2
console.log(items[0]);              // 3
console.log(items[1]);              // "2" 
```

当使用单个数值参数来调用 `Array` 构造器时，数组的长度属性会被设置为该参数；而如果使用单个的非数值型参数来调用，该参数就会成为目标数组的唯一项；如果使用多个参数（无论是否为数值类型）来调用，这些参数也会成为目标数组的项。数组的这种行为既混乱又有风险，因为有时可能不会留意所传参数的类型。

ES6 引入了 `Array.of()` 方法来解决这个问题。该方法的作用非常类似 `Array` 构造器，但在使用单个数值参数的时候并不会导致特殊结果。 `Array.of()` 方法总会创建一个包含所有传入参数的数组，而不管参数的数量与类型。下面几个例子演示了 `Array.of()` 的用法：

```js
let items = Array.of(1, 2);
console.log(items.length);          // 2
console.log(items[0]);              // 1
console.log(items[1]);              // 2

items = Array.of(2);
console.log(items.length);          // 1
console.log(items[0]);              // 2

items = Array.of("2");
console.log(items.length);          // 1
console.log(items[0]);              // "2" 
```

在使用 `Array.of()` 方法创建数组时，只需将想要包含在数组内的值作为参数传入。第一个例子创建了一个包含两个项的数组，第二个数组只包含了单个数值项，而最后一个数组则包含了单个字符串项。这个结果类似于使用数组字面量写法，通常你都可以在原生数组上使用字面量写法来代替 `Array.of()` ，但若想向函数传递参数，使用 `Array.of()` 而非 `Array` 构造器能够确保行为一致。例如：

```js
function createArray(arrayCreator, value) {
    return arrayCreator(value);
}

let items = createArray(Array.of, value); 
```

此代码中的 `createArray()` 函数接受两个参数：一个数组创建器与一个值，并会将后者插入到目标数组中。你应当向 `createArray()` 函数传递 `Array.of()` 作为第一个参数来创建新数组；相反，若传递 `Array` 构造器则会有危险，因为你无法保证第二个参数不是数值类型。

> `Array.of()` 方法并没有使用 `Symbol.species` 属性（参阅第九章）来决定返回值的类型，而是使用了当前的构造器（即 `of()` 方法内部的 `this` ）来做决定。

#### Array.from() 方法

在 JS 中将非数组对象转换为真正的数组总是很麻烦。例如，若想将类数组的 `arguments` 对象当做数组来使用，那么你首先需要对其进行转换。在 ES5 中，进行这种转换需要编写一个函数，类似下面这样：

```js
function makeArray(arrayLike) {
    var result = [];

    for (var i = 0, len = arrayLike.length; i < len; i++) {
        result.push(arrayLike[i]);
    }

    return result;
}

function doSomething() {
    var args = makeArray(arguments);

    // 使用 args
} 
```

该方式手动创建了一个 `result` 数组，并将 `arguments` 对象的所有项复制到该数组中。这种方式虽然有效，却为一个简单操作书写了过多的代码。开发者最终发现他们可以调用数组原生的 `slice()` 方法来减少代码量，就像这样：

```js
function makeArray(arrayLike) {
    return Array.prototype.slice.call(arrayLike);
}

function doSomething() {
    var args = makeArray(arguments);

    // 使用 args
} 
```

这段代码的功能与前一段代码等效。它能正常工作是因为将 `slice()` 方法的 `this` 设置为类数组对象， `slice()` 只需要有数值类型的索引与长度属性就能正常工作，而类数组对象能满足这些要求。

尽管这种技巧所用的代码量更少，但调用 `Array.prototype.slice.call(arrayLike)` 并没有明确体现出“要将类数组对象转换为数组”的目的。幸运的是， ES6 新增了 `Array.from()` 方法来提供一种明确清晰的方式以解决这方面的需求。

将可迭代对象或者类数组对象作为第一个参数传入， `Array.from()` 就能返回一个数组。这里有个简单的例子：

```js
function doSomething() {
    var args = Array.from(arguments);

    // 使用 args
} 
```

此处调用 `Array.from()` 方法，使用 `arguments` 对象创建了一个新数组 `args` ，它是一个数组实例，并且包含了 `arguments` 对象的所有项，同时还保持了项的顺序。

> `Array.from()` 方法同样使用 `this` 来决定要返回什么类型的数组。

##### 映射转换

如果你想实行进一步的数组转换，你可以向 `Array.from()` 方法传递一个映射用的函数作为第二个参数。此函数会将类数组对象的每一个值转换为目标形式，并将其存储在目标数组的对应位置上。例如：

```js
function translate() {
    return Array.from(arguments, (value) => value + 1);
}

let numbers = translate(1, 2, 3);

console.log(numbers);               // 2,3,4 
```

此代码将 `(value) => value + 1` 作为映射函数传递给了 `Array.from()` 方法，对每个项进行了一次 +1 处理。如果映射函数需要在对象上工作，你可以手动传递第三个参数给 `Array.from()` 方法，从而指定映射函数内部的 `this` 值。

```js
let helper = {
    diff: 1,

    add(value) {
        return value + this.diff;
    }
};

function translate() {
    return Array.from(arguments, helper.add, helper);
}

let numbers = translate(1, 2, 3);

console.log(numbers);               // 2,3,4 
```

这个例子使用了 `helper.add()` 作为映射函数。由于该函数使用了 `this.diff` 属性，你必须向 `Array.from()` 方法传递第三个参数用于指定 `this` 。借助这个参数， `Array.from()` 就可以方便地进行数据转换，而无须调用 `bind()` 方法、或用其他方式去指定 `this` 值。

##### 在可迭代对象上使用

`Array.from()` 方法不仅可用于类数组对象，也可用于可迭代对象，这意味着该方法可以将任意包含 `Symbol.iterator` 属性的对象转换为数组。例如：

```js
let numbers = {
    *[Symbol.iterator]() {
        yield 1;
        yield 2;
        yield 3;
    }
};

let numbers2 = Array.from(numbers, (value) => value + 1);

console.log(numbers2);              // 2,3,4 
```

由于代码中的 `numbers` 对象是一个可迭代对象，你可以把它直接传递给 `Array.from()` 方法，从而将它包含的值转换为数组。映射函数对每个数都进行了 +1 处理，因此目标数组的内容就是 2 、 3 、 4 ，而不是 1 、 2 、 3 。

> 如果一个对象既是类数组对象，又是可迭代对象，那么迭代器就会使用 `Array.from()` 方法来决定需要转换的值。

### 所有数组上的新方法

ES6 延续了 ES5 的工作，为数组增加了几个新方法。 `find()` 与 `findIndex()` 方法是为了让开发者能够处理包含任意值的数组，而 `fill()` 与 `copyWithin()` 方法则是受到了**类型化数组**（ **typed arrays** ）的启发。类型化数组是在 ES6 中引入的，只允许包含数值类型的值。

#### find() 与 findIndex() 方法

在 ES5 之前，检索数组是件麻烦事，因为没有对应的原生方法可用。 ES5 增加了 `indexOf()` 与 `lastIndexOf()` 方法，从而允许开发者在数组中查找特定值。这虽然是很大的进步，但依然受到了一些限制，因为你每次只能用它们来查找某个特定值。例如，若想在一系列的数中间查找第一个偶数，你必须自己写代码来实现这个意图。而 ES6 引入了 `find()` 与 `findIndex()` 方法，从而解决了这方面的问题。

`find()` 与 `findIndex()` 方法均接受两个参数：一个回调函数、一个可选值用于指定回调函数内部的 `this` 。该回调函数可接收三个参数：数组的某个元素、该元素对应的索引位置、以及该数组自身，这与 `map()` 和 `forEach()` 方法的回调函数所用的参数一致。该回调函数应当在给定的元素满足你定义的条件时返回 `true` ，而 `find()` 与 `findIndex()` 方法均会在回调函数第一次返回 `true` 时停止查找。

二者唯一的区别是： `find()` 方法会返回匹配的值，而 `findIndex()` 方法则会返回匹配位置的索引。这里有个示例：

```js
let numbers = [25, 30, 35, 40, 45];

console.log(numbers.find(n => n > 33));         // 35
console.log(numbers.findIndex(n => n > 33));    // 2 
```

这段代码使用了 `find()` 与 `findIndex()` 方法在 `numbers` 数组中查找第一个大于 33 的元素，前者返回 35 ，而后者返回 2 （也就是 35 这个元素在数组中的索引值）。

`find()` 与 `findIndex()` 方法在查找满足特定条件的数组元素时非常有用。但若想查找特定值，则使用 `indexOf()` 与 `lastIndexOf()` 方法会是更好的选择。

#### fill() 方法

`fill()` 方法能使用特定值填充数组中的一个或多个元素。当只使用一个参数的时候，该方法会用该参数的值填充整个数组，例如：

```js
let numbers = [1, 2, 3, 4];

numbers.fill(1);

console.log(numbers.toString());    // 1,1,1,1 
```

此代码中的 `numbers.fill(1)` 调用将 `numbers` 数组中的所有元素都填充为 `1` 。若你不想改变数组中的所有元素，而只想改变其中一部分，那么可以使用可选的起始位置参数与结束位置参数（不包括结束位置的那个元素），就像这样：

```js
let numbers = [1, 2, 3, 4];

numbers.fill(1, 2);

console.log(numbers.toString());    // 1,2,1,1

numbers.fill(0, 1, 3);

console.log(numbers.toString());    // 1,0,0,1 
```

当进行 `numbers.fill(1,2)` 调用时，第二个参数 `2` 指定从数组索引值为 2 的元素（即数组的第 3 个元素）开始填充，而此时没有指定第三个参数，因此结束位置默认为 `numbers` 数组的长度，意味着该数组的最后两个元素会被填充为 `1` 。而 `numbers.fill(0, 1, 3)` 调用则将该数组索引值为 1 与 2 的元素填充为 `0` 。在调用 `fill()` 方法时指定第二个和第三个参数，允许你一次性填充数组中多个元素，避免改写整个数组。

> 如果提供的起始位置或结束位置为负数，则它们会被加上数组的长度来算出最终的位置。例如：将起始位置指定为 `-1` ，就等于是 `array.length - 1` ，这里的 `array` 指的是 `fill()` 方法所要处理的数组。

#### copyWithin() 方法

`copyWithin()` 方法与 `fill()` 类似，可以一次性修改数组的多个元素。不过，与 `fill()` 使用单个值来填充数组不同， `copyWithin()` 方法允许你在数组内部复制自身元素。为此你需要传递两个参数给 `copyWithin()` 方法：从什么位置开始进行填充，以及被用来复制的数据的起始位置索引。

例如，将数组的前两个元素复制到数组的最后两个位置，你可以这么做：

```js
let numbers = [1, 2, 3, 4];

// 从索引 2 的位置开始粘贴
// 从数组索引 0 的位置开始复制数据
numbers.copyWithin(2, 0);

console.log(numbers.toString());    // 1,2,1,2 
```

这段代码从 `numbers` 数组索引值为 2 的元素开始进行填充，因此索引值为 2 与 3 的元素都会被覆盖；调用 `copyWithin()` 方法时将第二个参数指定为 `0` ，表示被复制的数据从索引值为 0 的元素开始，一直到没有元素可供复制为止。

默认情况下， `copyWithin()` 方法总是会一直复制到数组末尾，不过你还可以提供一个可选参数来限制到底有多少元素会被覆盖。这第三个参数指定了复制停止的位置（不包含该位置自身），这里有个范例：

```js
let numbers = [1, 2, 3, 4];

// 从索引 2 的位置开始粘贴
// 从数组索引 0 的位置开始复制数据
// 在遇到索引 1 时停止复制
numbers.copyWithin(2, 0, 1);

console.log(numbers.toString());    // 1,2,1,4 
```

在这个例子中，因为可选的结束位置参数被指定为 `1` ，于是只有索引值为 0 的元素被复制了，而该数组的最后一个元素并没有被修改。

> 类似于 `fill()` 方法，如果你向 `copyWithin()` 方法传递负数参数，数组的长度会自动被加到该参数的值上，以便算出正确的索引位置。

`fill()` 与 `copyWithin()` 方法初看起来不是那么有用，因为它们起源于类型化数组的需求，而出于功能一致性的目的才被添加到常规数组上。不过，接下来的小节你就会学到如何用类型化数组来按位操作数值，此时这两个方法就会变得非常有用了。

### 类型化数组

类型化数组是有特殊用途的数组，被设计用来处理数值类型数据（而不像名称暗示的那样，能处理所有类型）。类型化数组的起源可以追溯到 WebGL —— Open GL ES 2.0 的一个接口，设计用于配合网页上的 `<canvas>` 元素。类型化数组作为该接口实现的一部分，为 JS 提供了快速的按位运算能力。

对于 WebGL 的需求来说， JS 原生的数学运算实在太慢，因为它使用 64 位浮点数格式来存储数值，并在必要时将其转换为 32 位整数。引入类型化数组突破了格式限制并带来了更好的数学运算性能，其设计概念是：单个数值可以被视为由“位”构成的数组，并且可以对其使用与 JS 数组现有方法类似的方法。

ES6 采纳了类型化数组，将其作为语言的一个正式部分，以确保在 JS 引擎之间有更好的兼容性，并确保与 JS 数组有更好的互操作性。尽管 ES6 的类型化数组与 WebGL 的类型化数组并不完全一样，但它们已足够相似，使得前者可以被视为后者的进化版本，而不至于是完全不同的。

#### 数值数据类型

JS 数值使用 IEEE 754 标准格式存储，使用 64 位来存储一个数值的浮点数表示形式，该格式在 JS 中被同时用来表示整数与浮点数；当值改变时，可能会频繁发生整数与浮点数之间的格式转换。而类型化数组则允许存储并操作八种不同的数值类型：

1.  8 位有符号整数（int8）
2.  8 位无符号整数（uint8）
3.  16 位有符号整数（int16）
4.  16 位无符号整数（uint16）
5.  32 位有符号整数（int32）
6.  32 位无符号整数（uint32）
7.  32 位浮点数（float32）
8.  64 位浮点数（float64）

如果你将一个 int8 范围内的数表示为常规的 JS 数值，你就浪费了 56 个位，而这些浪费的位本可用来存储额外的 int8 值、或任意需求小于 56 位的数值。更有效地利用“位”是类型化数组处理的用例之一。

所有与类型化数组相关的操作和对象都围绕着这八种数据类型。为了使用它们，你首先需要创建一个数组缓冲区用于存储数据。

> 在本书中，我将使用上述列表中括号内的缩写词来表示这些类型，不过这些缩写并不会出现在实际的 JS 代码中，因为它们仅仅是对超长描述信息的速记。

#### 数组缓冲区

**数组缓冲区**（**array buffer**）是内存中包含一定数量字节的区域，而所有的类型化数组都基于数组缓冲区。创建数组缓冲区类似于在 C 语言中使用 `malloc()` 来分配内存，而不需要指定这块内存包含什么。你可以像下例这样使用 `ArrayBuffer` 构造器来创建一个数组缓冲区：

```js
let buffer = new ArrayBuffer(10);   // 分配了 10 个字节 
```

调用 `ArrayBuffer` 构造器时，只需要传入单个数值用于指定缓冲区包含的字节数，而本例就创建了一个 10 字节的缓冲区。当数组缓冲区被创建完毕后，你就可以通过检查 `byteLength` 属性来获取缓冲区的字节数：

```js
let buffer = new ArrayBuffer(10);   // 分配了 10 个字节
console.log(buffer.byteLength);     // 10 
```

你还可以使用 `slice()` 方法来创建一个新的、包含已有缓冲区部分内容的数组缓冲区。该 `slice()` 方法类似于数组上的同名方法，可以使用起始位置与结束位置参数，返回由原缓冲区元素组成的一个新的 `ArrayBuffer` 实例。例如：

```js
let buffer = new ArrayBuffer(10);   // 分配了 10 个字节

let buffer2 = buffer.slice(4, 6);
console.log(buffer2.byteLength);    // 2 
```

此代码创建了 `buffer2` 数组缓冲区，提取了原缓冲区索引值为 4 与 5 的元素。与数组的同名方法一样，结束参数所对应的元素是不会包含在结果中的。

当然，仅仅创建一个存储区域而不能写入数据，没有什么意义。为了写入数据，你需要创建一个视图（view）：

> 数组缓冲区总是保持创建时指定的字节数，你可以修改其内部的数据，但永远不能修改它的容量。

#### 使用视图操作数组缓冲区

数组缓冲区代表了一块内存区域，而**视图**（**views**）则是你操作这块区域的接口。视图工作在数组缓冲区或其子集上，可以读写某种数值数据类型的数据。 `DataView` 类型是数组缓冲区的通用视图，允许你对前述所有八种数值数据类型进行操作。

使用 `DataView` ，首先需要创建 `ArrayBuffer` 的一个实例，再在上面创建一个新的 `ArrayBuffer` 视图。这里有个例子：

```js
let buffer = new ArrayBuffer(10),
    view = new DataView(buffer); 
```

本例中的 `view` 对象可以使用 `buffer` 对象的所有 10 个字节。而你也可以在缓冲区的一个部分上创建视图，只需要指定可选参数——字节偏移量、以及所要包含的字节数。当未提供最后一个参数时，该 `DataView` 视图会默认包含从偏移位置开始、到缓冲区末尾为止的元素。例如：

```js
let buffer = new ArrayBuffer(10),
    view = new DataView(buffer, 5, 2);      // 包含位置 5 与位置 6 的字节 
```

此例中的 `view` 只能使用索引值为 5 与 6 的字节。使用这种方式，你可以在同一个数组缓冲区上创建多个不同的视图，这样有助于将单块内存区域供给整个应用使用，而不必每次在有需要时才动态分配内存。

##### 获取视图信息

你可以通过查询以下只读属性来获取视图的信息：

*   `buffer` ：该视图所绑定的数组缓冲区；
*   `byteOffset` ：传给 `DataView` 构造器的第二个参数，如果当时提供了的话（默认值为 0）;
*   `byteLength` ：传给 `DataView` 构造器的第三个参数，如果当时提供了的话（默认值为该缓冲区的 `byteLength` 属性）。

使用这些属性，你就可以查出所操作视图的准确位置，例如：

```js
let buffer = new ArrayBuffer(10),
    view1 = new DataView(buffer),           // 包含所有字节
    view2 = new DataView(buffer, 5, 2);     // 包含位置 5 与位置 6 的字节

console.log(view1.buffer === buffer);       // true
console.log(view2.buffer === buffer);       // true
console.log(view1.byteOffset);              // 0
console.log(view2.byteOffset);              // 5
console.log(view1.byteLength);              // 10
console.log(view2.byteLength);              // 2 
```

此代码创建了包含整个缓冲区的 `view1` 视图，并创建了包含缓冲区一小部分的 `view2` 视图。这两个视图拥有相同的 `buffer` 属性值，因为它们是在同一个数组缓冲区上工作的；而二者的 `byteOffset` 与 `byteLength` 属性就不相等了。这些属性反映出视图使用了缓冲区的哪些部分。

当然，仅仅读取缓冲区的内存信息不太有用，你还需要能向其写入数据并重新读出数据。

##### 读取与写入数据

对应于 JS 所有八种数值数据类型， `DataView` 视图的原型分别提供了在数组缓冲区上写入数据的一个方法、以及读取数据的一个方法。所有方法名都以“set”或“get”开始，其后跟随着对应数据类型的缩写。下面列出了能够操作 int8 或 uint8 类型的读取/写入方法：

*   `getInt8(byteOffset, littleEndian)` ：从 `byteOffset` 处开始读取一个 int8 值；

*   `setInt8(byteOffset, value, littleEndian)` ：从 `byteOffset` 处开始写入一个 int8 值；

*   `getUint8(byteOffset, littleEndian)` ：从 `byteOffset` 处开始读取一个无符号 int8 值；

*   `setUint8(byteOffset, value, littleEndian)` ：从 `byteOffset` 处开始写入一个无符号 int8 值。

“get”方法接受两个参数：开始进行读取的字节偏移量、以及一个可选的布尔值，后者用于指定读取的值是否采用低字节优先方式（注：默认值为 `false` ）。“set”方法则接受三个参数：开始进行写入的字节偏移量、需要写入的数据值、以及一个可选的布尔值用于指定是否采用低字节优先方式存储数据。

> 译注：**低字节优先**（**Little-endian**）也被翻译作“小端字节序”，指的是在存储数据的多个内存字节中，第一个内存字节存储着数据的最低字节数据，而最后一个内存字节存储着最高字节数据。
> 
> 例如：十进制数 5882 用十六进制表示是 16FA ，如果采用低字节优先方式、并使用 4 字节（即 32 位）存储，则该数字在内存中会被存储为 FA 16 00 00 。而如果采用相反的存储方式：**高字节优先**（**Big-endian**，大端字节序），那么该数字则会被存储为 00 00 16 FA 。

尽管上面只列出了操作 8 位值的方法，但只要将方法名中的 `8` 替换为 `16` 或 `32` ，便可以用来操作 16 位或 32 位值。而除了这些整数类方法之外， `DataView` 也提供了下列读写方法以便处理浮点数：

*   `getFloat32(byteOffset, littleEndian)` ：从 `byteOffset` 处开始读取一个 32 位的浮点数；

*   `setFloat32(byteOffset, value, littleEndian)` ：从 `byteOffset` 处开始写入一个 32 位的浮点数；

*   `getFloat64(byteOffset, littleEndian)` ：从 `byteOffset` 处开始读取一个 64 位的浮点数；

*   `setFloat64(byteOffset, value, littleEndian)` ：从 `byteOffset` 处开始写入一个 64 位的浮点数。

为了弄懂“set”与“get”方法如何使用，可研究下面的例子：

```js
let buffer = new ArrayBuffer(2),
    view = new DataView(buffer);

view.setInt8(0, 5);
view.setInt8(1, -1);

console.log(view.getInt8(0));       // 5
console.log(view.getInt8(1));       // -1 
```

该代码使用一个双字节的数组缓冲区来存储两个 int8 值。第一个值被存储在位置 0 ，而第二个值则被存储在位置 1 ，表示每个值占用了一个完整的字节（8 位），此后还使用 `getInt8()` 方法来将这些值从对应位置读取出来。尽管这个例子只使用了 int8 类型的值，但你却可以使用八种数值数据类型的所有对应方法。

视图允许你使用任意格式对任意位置进行读写，而无须考虑这些数据此前是使用什么格式存储的，这非常有意思。例如，向缓冲区写入两个 int8 值，并将其作为一个 int16 值读取出来，这是完全可行的，如同下面这个例子：

```js
let buffer = new ArrayBuffer(2),
    view = new DataView(buffer);

view.setInt8(0, 5);
view.setInt8(1, -1);

console.log(view.getInt16(0));      // 1535
console.log(view.getInt8(0));       // 5
console.log(view.getInt8(1));       // -1 
```

该代码使用 `view.getInt16(0)` 读取了该视图的所有字节，并将其解析为数值 1535 。为了理解这个范例，可以参阅下面的示意图，它揭示了每个 `setInt8()` 操作对缓冲区造成的变化：

```js
new ArrayBuffer(2)      0000000000000000
view.setInt8(0, 5);     0000010100000000
view.setInt8(1, -1);    0000010111111111 
```

开始时，该数组缓冲区 16 个位均为 0 ；使用 `setInt8()` 向第一个字节写入 `5` 之后，该字节的内容就出现了一对 1 （因为 5 可以写为 8 位二进制数 00000101 ）；向第二个字节写入 -1 会使得该字节的所有位都变成 1 （即 -1 的二进制补码形式）。接下来使用 `getInt16()` 就能将前面写入的 16 位数据以单个 16 位整数的方式读取出来，其十进制值就是 1535 。

在混用不同的数据类型时，使用 `DataView` 对象是一种完美方式。不过，若仅想使用特定的一种数据类型，那么特定类型视图会是更好的选择。

##### 类型化数组即为视图

ES6 的类型化数组实际上也是针对数组缓冲区的特定类型视图，你可以使用这些数组对象来处理特定的数据类型，而不必使用通用的 `DataView` 对象。一共存在八种特定类型视图，对应着八种数值数据类型，为处理 `uint8` 值提供了额外的选择。

特定类型视图被包含在 ES6 规范的 22.2 小节中，下表列出了它们的概要：

| 构造器名称 | 元素大小（字节） | 描述 | 等价的 C 语言类型 |
| --- | --- | --- | --- |
| `Int8Array` | 1 | 8 位有符号整数，采用补码 | `signed char` |
| `Uint8Array` | 1 | 8 位无符号整数 | `unsigned char` |
| `Uint8ClampedArray` | 1 | 8 位无符号整数 (clamped conversion，无溢出转换) | `unsigned char` |
| `Int16Array` | 2 | 16 位有符号整数，采用补码 | `short` |
| `Uint16Array` | 2 | 16 位无符号整数 | `unsigned short` |
| `Int32Array` | 4 | 32 位有符号整数，采用补码 | `int` |
| `Uint32Array` | 4 | 32 位无符号整数 | `int` |
| `Float32Array` | 4 | 32 位 IEEE 浮点数 | `float` |
| `Float64Array` | 8 | 64 位 IEEE 浮点数 | `double` |

左边一列列出了类型化数组的构造器，而其他列则描述了对应的类型化数组所能包含的数据。 `Uint8ClampedArray` 的特性与 `Uint8Array` 基本相同，只有当缓冲区包含的值小于 0 或者大于 255 的时候才有区别：当值小于 0 时， `Uint8ClampedArray` 会将该值转换为 0 进行存储（例如 -1 会被存储为 0 ）；而当值大于 255 时，会被转换为 255 （例如 300 会被存储为 255 ）。

类型化数组只能在特定的一种数据类型上工作，例如： `Int8Array` 的所有操作都只能处理 `int8` 值。每种类型化数组的单个元素大小也都取决于对应类型， `Int8Array` 中每个元素都是单字节的，而 `Float64Array` 则使用了八个字节来存储单个元素。幸运的是，类型化数组的元素可以使用数值型的索引位置来访问，就像常规数组那样，从而规避了使用 `DataView` 存取方法时的某些尴尬情况。

> **元素大小**
> 
> 每一种类型化数组都由一定数量的元素构成，而“元素大小”则代表每个类型的单个元素所包含的字节数。这个数字被存储在类型化数组每个构造器与每个实例的 `BYTES_PER_ELEMENT` 属性中，方便你查询元素的大小：
> 
> ```js
> console.log(UInt8Array.BYTES_PER_ELEMENT);      // 1
> console.log(UInt16Array.BYTES_PER_ELEMENT);     // 2
> 
> let ints = new Int8Array(5);
> console.log(ints.BYTES_PER_ELEMENT);            // 1 
> ```

##### 创建特定类型视图

类型化数组的构造器可以接受多种类型的参数，因此存在几种创建类型化数组的方式。第一种方式是使用与创建 `DataView` 时相同的参数，即：一个数组缓冲区、一个可选的字节偏移量、以及一个可选的字节数量。例如：

```js
let buffer = new ArrayBuffer(10),
    view1 = new Int8Array(buffer),
    view2 = new Int8Array(buffer, 5, 2);

console.log(view1.buffer === buffer);       // true
console.log(view2.buffer === buffer);       // true
console.log(view1.byteOffset);              // 0
console.log(view2.byteOffset);              // 5
console.log(view1.byteLength);              // 10
console.log(view2.byteLength);              // 2 
```

此代码在 `buffer` 对象上创建了两个 `Int8Array` 类型的视图：`view1` 与 `view2` ，而这两个视图拥有相同的 `buffer` 、 `byteOffset` 与 `byteLength` 属性。如果你的操作只针对一种数值类型，那么很容易就能把代码从使用 `DataView` 视图切换到使用某种类型化数组。

第二种方式是传递单个数值给类型化数组的构造器，此数值表示该数组包含的元素数量（而不是分配的字节数）。构造器将会创建一个新的缓冲区，分配正确的字节数以便容纳指定数量的数组元素，而你也可以使用 `length` 属性来获取这个元素数量。例如：

```js
let ints = new Int16Array(2),
    floats = new Float32Array(5);

console.log(ints.byteLength);       // 4
console.log(ints.length);           // 2

console.log(floats.byteLength);     // 20
console.log(floats.length);         // 5 
```

示例中的 `ints` 数组创建时包含了两个元素，而每个 16 位整数需要使用两个字节，因此该数组一共被分配了 4 个字节。 `floats` 数组则包含五个元素，因此它就需要 20 个字节（每个元素占用四个字节）。这两个数组都创建了对应的数组缓冲区，而在必要时都可以使用 `buffer` 属性来访问各自的缓冲区。

> 如果调用类型化数组构造器时没有传入参数，构造器会认为传入了 `0` ，这种方式创建的类型化数组不会被分配任何存储空间，因此也就不能被用于保存数据。

第三种方式是向构造器传递单个对象参数，可以是下列四种对象之一：

*   **类型化数组**：数组所有元素都会被复制到新的类型化数组中。例如，如果你传递一个 int8 类型的数组给 `Int16Array` 构造器，这些 int8 的值会被复制到 int16 数组中。新的类型化数组与原先的类型化数组会使用不同的数组缓冲区。
*   **可迭代对象**：该对象的迭代器会被调用以便将数据插入到类型化数组中。如果其中包含了不匹配视图类型的值，那么构造器就会抛出错误。
*   **数组**：该数组的元素会被插入到新的类型化数组中。如果其中包含了不匹配视图类型的值，那么构造器就会抛出错误。
*   **类数组对象**：与传入数组的表现一致。

在上述任意可能中，新的类型化数组都会从原对象获取数据。若想用一些值来初始化一个类型化数组，这种方式就特别有用，就像这样：

```js
let ints1 = new Int16Array([25, 50]),
    ints2 = new Int32Array(ints1);

console.log(ints1.buffer === ints2.buffer);     // false

console.log(ints1.byteLength);      // 4
console.log(ints1.length);          // 2
console.log(ints1[0]);              // 25
console.log(ints1[1]);              // 50

console.log(ints2.byteLength);      // 8
console.log(ints2.length);          // 2
console.log(ints2[0]);              // 25
console.log(ints2[1]);              // 50 
```

该例使用了一个包含两个值的数组来创建一个 `Int16Array` 并初始化它，之后又利用该 `Int16Array` 创建了一个 `Int32Array` 。 25 与 50 这两个值从 `ints1` 数组中被复制到 `ints2` 数组中，但两个数组使用了全然不同的缓冲区。虽然二者都包含了相同的数值，但后者占用了 8 个字节，而前者只占用了 4 字节。

### 类型化数组与常规数组的相似点

类型化数组与常规数组有好几个相似点，并且正如你已经在本章看到的那样，类型化数组在很多场景中都可以像常规数组那样被使用。例如，你可以使用 `length` 属性来获取类型化数组包含的元素数量，还可以使用数值类型的索引值来直接访问类型化数组的元素。举个例子：

```js
let ints = new Int16Array([25, 50]);

console.log(ints.length);          // 2
console.log(ints[0]);              // 25
console.log(ints[1]);              // 50

ints[0] = 1;
ints[1] = 2;

console.log(ints[0]);              // 1
console.log(ints[1]);              // 2 
```

这段代码创建了一个包含两个元素的 `Int16Array` ，使用数值类型的索引可以读写对应的项，而数值在存储时会被自动转换为 int16 类型的值。

相似点还不限于此。

> 与常规数组不同的是，你不能使用 `length` 属性来改变类型化数组的大小。该属性是不可写的，在非严格模式下写入操作会被忽略，而严格模式下则会抛出错误。

#### 公共方法

类型化数组也拥有大量与常规数组等效的方法，你可以对类型化数组使用下列这些方法：

*   `copyWithin()`
*   `entries()`
*   `fill()`
*   `filter()`
*   `find()`
*   `findIndex()`
*   `forEach()`
*   `indexOf()`
*   `join()`
*   `keys()`
*   `lastIndexOf()`
*   `map()`
*   `reduce()`
*   `reduceRight()`
*   `reverse()`
*   `slice()`
*   `some()`
*   `sort()`
*   `values()`

注意：虽然这些方法的表现与数组原型上的对应方法相似，但它们并不完全相同。类型化数组的方法会进行额外的类型检查以确保安全，并且返回值会是某种类型化数组，而不是常规数组（归结于 `Symbol.species` 属性）。这里有个例子用于演示其中的区别：

```js
let ints = new Int16Array([25, 50]),
    mapped = ints.map(v => v * 2);

console.log(mapped.length);        // 2
console.log(mapped[0]);            // 50
console.log(mapped[1]);            // 100

console.log(mapped instanceof Int16Array);  // true 
```

这段代码通过 `map()` 方法使用 `ints` 中的值创建了一个新数组，映射函数将每个值翻倍，并返回了一个新的 `Int16Array` 。

#### 相同的迭代器

与常规数组相同，类型化数组也拥有三个迭代器，它们是 `entries()` 方法、 `keys()` 方法与 `values()` 方法。这就意味着你可以对类型化数组使用扩展运算符，或者对其使用 `for-of` 循环，就像对待常规数组。举个例子：

```js
let ints = new Int16Array([25, 50]),
    intsArray = [...ints];

console.log(intsArray instanceof Array);    // true
console.log(intsArray[0]);                  // 25
console.log(intsArray[1]);                  // 50 
```

此代码创建了一个名为 `intsArray` 的新数组，包含了类型化数组 `ints` 的所有数据。借助扩展运算符能轻易地将类型化数组转换为常规数组，就像处理其他可迭代对象那样。

#### of() 与 from() 方法

最后，所有的类型化数组都包含静态的 `of()` 与 `from()` 方法，作用类似于 `Array.of()` 与 `Array.from()` 方法。其中的区别是类型化数组的版本会返回类型化数组，而不返回常规数组。下面的例子使用这两个方法创建了几个类型化数组：

```js
let ints = Int16Array.of(25, 50),
    floats = Float32Array.from([1.5, 2.5]);

console.log(ints instanceof Int16Array);        // true
console.log(floats instanceof Float32Array);    // true

console.log(ints.length);       // 2
console.log(ints[0]);           // 25
console.log(ints[1]);           // 50

console.log(floats.length);     // 2
console.log(floats[0]);         // 1.5
console.log(floats[1]);         // 2.5 
```

此例中分别使用了 `of()` 与 `from()` 方法来创建一个 `Int16Array` 以及一个 `Float32Array` ，这两个方法确保创建类型化数组能像创建常规数组那样轻松。

### 类型化数组与常规数组的区别

二者最重要的区别就是类型化数组并不是常规数组，类型化数组并不是从 `Array` 对象派生的，使用 `Array.isArray()` 去检测会返回 `false` ，例如：

```js
let ints = new Int16Array([25, 50]);

console.log(ints instanceof Array);     // false
console.log(Array.isArray(ints));       // false 
```

由于 `ints` 变量是一个类型化数组，因此它并不是 `Array` 对象的实例，于是就不会被识别为数组。这一点区别很重要，因为虽然类型化数组与常规数组非常相似，但前者仍然有一些不同的行为。

#### 行为差异

常规数组可以被伸展或是收缩，然而类型化数组却会始终保持自身大小不变。你可以对常规数组一个不存在的索引位置进行赋值，但在类型化数组上这么做则会被忽略。这里有个例子：

```js
let ints = new Int16Array([25, 50]);

console.log(ints.length);          // 2
console.log(ints[0]);              // 25
console.log(ints[1]);              // 50

ints[2] = 5;

console.log(ints.length);          // 2
console.log(ints[2]);              // undefined 
```

在本例中，尽管对索引值 2 的位置进行了赋值为 `5` 的操作，但 `ints` 数组却完全没有被伸展，数组的长度属性保持不变，所赋的值也被丢弃了。

类型化数组也会对数据类型进行检查以保证只使用有效的值，当无效的值被传入时，将会被替换为 0 ，例如：

```js
let ints = new Int16Array(["hi"]);

console.log(ints.length);       // 1
console.log(ints[0]);           // 0 
```

这段代码试图用字符串值 `"hi"` 创建一个 `Int16Array` ，而字符串对于类型化数组来说当然是无效的值，因此该字符串被替换为 `0` 并插入数组。此数组的长度仅仅是 1 ，而 `ints[0]` 只包含了 `0` 这个值。

所有在类型化数组上修改项目值的方法都会受到相同的限制，例如当 `map()` 方法使用的映射函数返回一个无效值的时候，类型化数组会使用 `0` 来代替返回值：

```js
let ints = new Int16Array([25, 50]),
    mapped = ints.map(v => "hi");

console.log(mapped.length);        // 2
console.log(mapped[0]);            // 0
console.log(mapped[1]);            // 0

console.log(mapped instanceof Int16Array);  // true
console.log(mapped instanceof Array);       // false 
```

由于字符串值 `"hi"` 并不是一个 16 位整数，它在结果数组中就被替换成为 `0` 。多亏这种纠错行为，类型化数组的内容永远不会是无效值，因此相关方法就无须再担心传入无效值会导致错误。

#### 遗漏的方法

尽管类型化数组拥有常规数组的很多同名方法，但仍然缺少了几个数组方法，包括下列这些：

*   `concat()`
*   `pop()`
*   `push()`
*   `shift()`
*   `splice()`
*   `unshift()`

除了 `concat()` 方法之外，该列表中的其余方法都会改变数组的大小，而由于类型化数组的大小不可变，因此这些方法都不能作用于类型化数组。 `concat()` 方法不可用的原因则是：连接两个类型化数组的结果是不确定的（特别是当它们处理的数据类型不同时），这种不确定情况原本就不应当使用类型化数组。

#### 附加的方法

最后，类型化数组还有两个常规数组所不具备的方法： `set()` 方法与 `subarray()` 方法。这两个方法作用相反： `set()` 方法从另一个数组中复制元素到当前的类型化数组，而 `subarray()` 方法则是将当前类型化数组的一部分提取为新的类型化数组。

`set()` 方法接受一个数组参数（无论是类型化的还是常规的）、以及一个可选的偏移量参数，后者指示了从什么位置开始插入数据（默认值为 0 ）。数组参数中的数据会被复制到目标类型化数组中，并会确保数据值有效。这里有个例子：

```js
let ints = new Int16Array(4);

ints.set([25, 50]);
ints.set([75, 100], 2);

console.log(ints.toString());   // 25,50,75,100 
```

这段代码创建了一个包含四个元素的 `Int16Array` ；第一次调用 `set()` 复制了两个值到数组起始的两个位置；而第二次调用 `set()` 则使用了一个值为 `2` 的偏移量参数，指明应当从数组的第三个位置（索引 2 ）开始放置所复制的数据。

`subarray()` 方法接受一个可选的开始位置索引参数、以及一个可选的结束位置索引参数（像 `slice()` 方法一样，结束位置的元素不会被包含在结果中），并会返回一个新的类型化数组。你可以同时省略这两个参数，从而创建原类型化数组的一个复制品。例如：

```js
let ints = new Int16Array([25, 50, 75, 100]),
    subints1 = ints.subarray(),
    subints2 = ints.subarray(2),
    subints3 = ints.subarray(1, 3);

console.log(subints1.toString());   // 25,50,75,100
console.log(subints2.toString());   // 75,100
console.log(subints3.toString());   // 50,75 
```

本例中利用 `ints` 数组创建了三个类型化数组。 `subints1` 数组是 `ints` 的一个复制品，包含了原数组的所有信息；而 `subints2` 则从原数组的索引 2 位置开始复制，因此包含了原数组的最末两个元素（即 75 与 100 ）；最后的 `subints3` 数组值包含了原数组的中间两个元素，因为调用 `subarray()` 时同时使用了起始位置与结束位置参数。

### 总结

ES6 延续了 ES5 的工作以便让数组更加有用。新增了两种创建数组的方式： `Array.of()` 方法、以及 `Array.from()` 方法，其中后者可以将可迭代对象或类数组对象转换为正规数组。这两个方法都在数组派生对象上被继承，并使用 `Symbol.species` 属性来决定返回的数组类型，而其他的继承方法在返回数组时也会使用该属性。

此外还有几个新增的数组方法。 `fill()` 方法与 `copyWithin()` 方法允许你替换数组内的元素。 `find()` 方法与 `findIndex()` 方法在数组中查找满足特定条件的元素时会非常有用，其中前者会返回满足条件的第一个元素，而后者会返回该元素的索引位置。

类型化数组并不是严格的数组，它们并没有继承 `Array` 对象，但它们的外观和行为都与数组有许多相似点。类型化数组包含的数据类型是八种数值数据类型之一，基于数组缓冲区对象建立，用于表示按位存储的一个数值或一系列数值。类型化数组能够明显提升按位运算的性能，因为它不像 JS 的常规数值类型那样需要频繁进行格式转换。