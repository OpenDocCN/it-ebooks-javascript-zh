# 第七章 Set 与 Map

## 第七章 Set 与 Map

JS 的大部分历史时期都只存在一种集合类型，也就是数组类型（尽管有人会争论说，所有非数组的对象都是键值对的集合，它们曾被用于与数组完全不同的用途）。数组在 JS 中的使用正如其它语言的数组一样，但缺少更多类型的集合导致数组也经常被当作队列与栈来使用。数组只使用了数值型的索引，而如果非数值型的索引是必要的，开发者便会使用非数组的对象。这种技巧引出了非数组对象的定制实现，即 Set 与 Map 。

**Set** 是不包含重复值的列表。你一般不会像对待数组那样来访问 Set 中的某个项；相反更常见的是，只在 Set 中检查某个值是否存在。 **Map** 则是键与相对应的值的集合。因此， Map 中的每个项都存储了两块数据，通过指定所需读取的键即可检索对应的值。 Map 常被用作缓存，存储数据以便此后快速检索。由于 Set 与 Map 并不正式存在于 ES5 中，开发者就只能使用非数组的对象。

ES6 向 JS 添加了 Set 与 Map ，本章将论述这两种集合类型你所需了解的全部内容。

首先，我会论述在 ES6 之前开发者为了实现 Set 与 Map 而采用的变通方法，并且这些方法为何是有问题的。在论述这些重要的背景之后，我会介绍 Set 与 Map 在 ES6 中如何工作。

*   ES5 中的 Set 与 Map
*   变通方法的问题
*   ES6 的 Set
    *   创建 Set 并添加项目
    *   移除值
    *   Set 上的 forEach() 方法
    *   将 Set 转换为数组
    *   Weak Set
        *   创建 Weak Set
        *   Set 类型之间的关键差异
*   ES6 的 Map
    *   Map 的方法
    *   Map 的初始化
    *   Map 上的 forEach 方法
    *   Weak Map
        *   使用 Weak Map
        *   Weak Map 的初始化
        *   Weak Map 的方法
        *   对象的私有数据
        *   Weak Map 的用法与局限性
*   总结

### ES5 中的 Set 与 Map

在 ES5 中，开发者使用对象属性来模拟 Set 与 Map ，就像这样：

```js
let set = Object.create(null);

set.foo = true;

// 检查属性的存在性
if (set.foo) {

    // 一些操作
} 
```

本例中的 `set` 变量是一个原型为 `null` 的对象，确保在此对象上没有继承属性。使用对象的属性作为需要检查的唯一值在 ES5 中是很常用的方法。当一个属性被添加到 `set` 对象时，它的值也被设为 `true` ，因此条件判断语句（例如本例中的 `if` 语句）就可以简单判断出该值是否存在。

使用对象模拟 Set 与模拟 Map 之间唯一真正的区别是所存储的值。例如，以下例子将对象作为 Map 使用：

```js
let map = Object.create(null);

map.foo = "bar";

// 提取一个值
let value = map.foo;

console.log(value);         // "bar" 
```

此代码将字符串值 `"bar"` 存储在 `foo` 键上。与 Set 不同， Map 多数被用来提取数据，而不是仅检查键的存在性。

### 变通方法的问题

尽管在简单情况下将对象作为 Set 与 Map 来使用都是可行的，但一旦接触到对象属性的局限性，此方式就会遇到更多麻烦。例如，由于对象属性的类型必须为字符串，你就必须保证任意两个键不能被转换为相同的字符串。研究以下代码：

```js
let map = Object.create(null);

map[5] = "foo";

console.log(map["5"]);      // "foo" 
```

本例将字符串值 `"foo"` 赋值到数值类型的键 `5` 上，而数值类型的键会在内部被转换为字符串，因此 `map["5"]` 与 `map[5]` 实际上引用了同一个属性。当你想将数值与字符串都作为键来使用时，这种内部转换会引起问题。而若使用对象作为键，就会出现另一个问题，例如：

```js
let map = Object.create(null),
    key1 = {},
    key2 = {};

map[key1] = "foo";

console.log(map[key2]);     // "foo" 
```

此处的 `map[key2]` 与 `map[key1]` 引用了同一个值。由于对象的属性只能是字符串， `key1` 与 `key2` 对象就均被转换为字符串；又因为对象默认的字符串类型表达形式是 `"[object Object]"` ， `key1` 与 `key2` 就被转换为了同一个字符串。这种行为导致的错误可能不太显眼，因为貌似合乎逻辑的假设是：键如果使用了不同对象，它们就应当是不同的键。

将对象转换为默认的字符串表现形式，使得对象很难被当作 Map 的键来使用（此问题同样存在于将对象作为 Set 来使用的尝试上）。

当键的值为假值时， Map 也遇到了自身的特殊问题。在需要布尔值的位置（例如在 `if` 语句内），任何假值都会被自动转换为 false 。这种转换单独说来并不是问题——只要对如何使用值的问题足够小心。例如，查看以下代码：

```js
let map = Object.create(null);

map.count = 1;

// 是想检查 "count" 属性的存在性，还是想检查非零值？
if (map.count) {
    // ...
} 
```

此例中 `map.count` 的用法存在歧义。此处的 `if` 语句是想检查 `map.count` 属性的存在性，还是想检查非零值？该 `if` 语句内的代码会被执行是因为 1 是真值。然而若 `map.count` 的值为 0 ，或者该属性不存在，则 `if` 语句内的代码都将不会被执行。

在大型应用中，这类问题都是难以确认、难以调试的，这也是 ES6 新增 Set 与 Map 类型的首要原因。

> JS 存在 `in` 运算符，若属性存在于对象中，就会返回 `true` 而无须读取对象的属性值。不过， `in` 运算符会搜索对象的原型，这使得它只有在处理原型为 `null` 的对象时才是安全的。但即使原型没问题，许多开发者仍然错误地使用与上例类似的代码，而不使用 `in` 运算符。

### ES6 的 Set

ES6 新增了 `Set` 类型，这是一种无重复值的有序列表。 Set 允许对它包含的数据进行快速访问，从而增加了一个追踪离散值的更有效方式。

#### 创建 Set 并添加项目

Set 使用 `new Set()` 来创建，而调用 `add()` 方法就能向 Set 中添加项目，检查 `size` 属性还能查看其中包含有多少项：

```js
let set = new Set();
set.add(5);
set.add("5");

console.log(set.size);    // 2 
```

Set 不会使用强制类型转换来判断值是否重复。这意味着 Set 可以同时包含数值 `5` 与 字符串 `"5"` ，将它们都作为相对独立的项（在 Set 内部的比较使用了第四章讨论过的 `Object.is()` 方法，来判断两个值是否相等）。你还可以向 Set 添加多个对象，它们不会被合并为同一项：

```js
let set = new Set(),
    key1 = {},
    key2 = {};

set.add(key1);
set.add(key2);

console.log(set.size);    // 2 
```

由于 `key1` 与 `key2` 并不会被转换为字符串，所以它们在这个 Set 内部被认为是两个不同的项（记住：如果它们被转换为字符串，那么都会等于 `"[object Object]"` ）。

如果 `add()` 方法用相同值进行了多次调用，那么在第一次之后的调用实际上会被忽略：

```js
let set = new Set();
set.add(5);
set.add("5");
set.add(5);     // 重复了，该调用被忽略

console.log(set.size);    // 2 
```

你可以使用数组来初始化一个 Set ，并且 `Set` 构造器会确保不重复地使用这些值。例如：

```js
let set = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
console.log(set.size);    // 5 
```

在此例中，带有重复值的数组被用来初始化这个 Set 。虽然数值 5 在数组中出现了四次，但 Set 中却只有一个 5 。若要把已存在的代码或 JSON 结构转换为 Set 来使用，这种特性会让转换更轻松。

> `Set` 构造器实际上可以接收任意可迭代对象作为参数。能使用数组是因为它们默认就是可迭代的， Set 与 Map 也是一样。 `Set` 构造器会使用迭代器来提取参数中的值。（可迭代对象与迭代器详见第八章）

你可以使用 `has()` 方法来测试某个值是否存在于 Set 中，就像这样：

```js
let set = new Set();
set.add(5);
set.add("5");

console.log(set.has(5));    // true
console.log(set.has(6));    // false 
```

此处的 Set 不包含 6 这个值，因此 `set.has(6)` 会返回 false 。

#### 移除值

也可以从 Set 中将值移除。你可以使用 `delete()` 方法来移除单个值，或调用 `clear()` 方法来将所有值从 Set 中移除。以下代码展示了二者的作用：

```js
let set = new Set();
set.add(5);
set.add("5");

console.log(set.has(5));    // true

set.delete(5);

console.log(set.has(5));    // false
console.log(set.size);      // 1

set.clear();

console.log(set.has("5"));  // false
console.log(set.size);      // 0 
```

在调用 `delete()` 之后，只有 `5` 被移走；而执行 `clear()` 方法后， `set` 就被清空了。

所有这些方法都提供了一个非常简单的机制来追踪有序的唯一值。不过，在给 Set 添加项之后，要如何对每个项执行一些操作呢？此时 `forEcah()` 方法就派上用场了。

#### Set 上的 forEach() 方法

若你曾处理过数组，可能就已经熟悉了 `forEach()` 方法。 ES5 给数组添加了 `forEach()` 方法，使得更易处理数组中的每一项，而无须建立 `for` 循环。该方法被开发者普遍使用，于是 Set 类型也添加了相同方法，其工作方式也一样。

`forEach()` 方法会被传递一个回调函数，该回调接受三个参数：

1.  Set 中下个位置的值；
2.  与第一个参数相同的值；
3.  目标 Set 自身。

Set 版本的 `forEach()` 方法与数组版本有个奇怪差异：前者传给回调函数的第一个与第二个参数是相同的。虽然看起来像是错误，但这种行为却有个正当理由。

具有 `forEach()` 方法的其它对象（即数组与 Map ）都会给回调函数传递三个参数，前两个参数都分别是下个位置的值与键（给数组使用的键是数值索引）。

然而 Set 却没有键。 ES6 标准的制定者本可以将 Set 版本的 `forEach()` 方法的回调函数设定为只接受两个参数，但这会让它不同于另外两个版本的方法。不过他们找到了一种方式让这些回调函数保持参数相同：将 Set 中的每一项同时认定为键与值。于是为了让 Set 的 `forEach()` 方法与数组及 Map 版本的保持一致，该回调函数的前两个参数就始终相同了。

除了参数特点的差异外，在 Set 上使用 `forEach()` 方法与在数组上基本相同。这里有些代码展示了该方法如何工作：

```js
let set = new Set([1, 2]);

set.forEach(function(value, key, ownerSet) {
    console.log(key + " " + value);
    console.log(ownerSet === set);
}); 
```

此代码在 Set 的每一项上进行迭代，并对传递给 `forEach()` 的回调函数的值进行了输出。回调函数每次执行时， `key` 与 `value` 总是相同的，同时 `ownerSet` 也始终等于 `set` 。此代码输出：

```js
1 1
true
2 2
true 
```

与使用数组相同，如果想在回调函数中使用 `this` ，你可以给 `forEach()` 传入一个 `this` 值作为第二个参数：

```js
let set = new Set([1, 2]);

let processor = {
    output(value) {
        console.log(value);
    },
    process(dataSet) {
        dataSet.forEach(function(value) {
            this.output(value);
        }, this);
    }
};

processor.process(set); 
```

本例中 `processor.process()` 方法在 Set 上调用了 `forEach()` ，并传递了当前 `this` 作为回调函数的 `this` 值。 这个传递十分必要，这样 `this.output()` 就能正确地解析到 `processor.output()` 方法。此处 `forEach()` 的回调函数仅使用了第一个参数 `value` ，其余参数则被省略了。你也可以使用箭头函数来达到相同效果，而无须传入第二个参数，就像这样：

```js
let set = new Set([1, 2]);

let processor = {
    output(value) {
        console.log(value);
    },
    process(dataSet) {
        dataSet.forEach((value) => this.output(value));
    }
};

processor.process(set); 
```

本例中的箭头函数读取了包含它的 `process()` 函数的 `this` 值，因此就能正确地将 `this.output()` 解析为调用 `processor.output()` 。

要记住，虽然 Set 能非常好地追踪值，并且 `forEach()` 可以让你按顺序处理每一项，但是却无法像数组那样用索引来直接访问某个值。如果你想这么做，最好的选择是将 Set 转换为数组。

#### 将 Set 转换为数组

将数组转换为 Set 相当容易，因为可以将数组传递给 `Set` 构造器；而使用扩展运算符也能简单地将 Set 转换回数组。第三章介绍的扩展运算符（ `...` ），能将数组中的项分割开并作为函数的分离参数。你同样能将扩展运算符用于可迭代对象（例如 Set ），将它们转换为数组。例如：

```js
let set = new Set([1, 2, 3, 3, 3, 4, 5]),
    array = [...set];

console.log(array);             // [1,2,3,4,5] 
```

此处的 Set 在初始化时载入了一个包含重复值的数组。 Set 清除了重复值之后，又使用了扩展运算符将自身的项放到一个新数组中。而这个 Set 仍然包含在创建时所接收的项（ `1` 、 `2` 、 `3` 、 `4` 与 `5` ），这些项只是被复制到了新数组中，而并未从 Set 中消失。

当已经存在一个数组，而你想用它创建一个无重复值的新数组时，该方法十分有用。例如：

```js
function eliminateDuplicates(items) {
    return [...new Set(items)];
}

let numbers = [1, 2, 3, 3, 3, 4, 5],
    noDuplicates = eliminateDuplicates(numbers);

console.log(noDuplicates);      // [1,2,3,4,5] 
```

在 `eliminateDuplicates()` 函数中， Set 只是一个临时的中介物，以便在创建一个无重复的数组之前将重复值过滤掉。

#### Weak Set

由于 `Set` 类型存储对象引用的方式，它也可以被称为 Strong Set 。对象存储在 `Set` 的一个实例中时，实际上相当于把对象存储在变量中。只要对于 `Set` 实例的引用仍然存在，所存储的对象就无法被垃圾回收机制回收，从而无法释放内存。例如：

```js
let set = new Set(),
    key = {};

set.add(key);
console.log(set.size);      // 1

// 取消原始引用
key = null;

console.log(set.size);      // 1

// 重新获得原始引用
key = [...set][0]; 
```

在本例中，将 `key` 设置为 `null` 清除了对 `key` 对象的一个引用，但是另一个引用还存于 `set` 内部。你仍然可以使用扩展运算符将 Set 转换为数组，然后访问数组的第一项， `key` 变量就取回了原先的对象。这种结果在大部分程序中是没问题的，但有时，当其它引用消失之后若 Set 内部的引用也能消失，那就更好。例如，当 JS 代码在网页中运行，同时你想保持与 DOM 元素的联系，在该元素可能被其他脚本移除的情况下，你应当不希望自己的代码保留对该 DOM 元素的最后一个引用（这种情况被称为**内存泄漏**）。

为了缓解这个问题， ES6 也包含了 **Weak Set** ，该类型只允许存储对象弱引用，而不能存储基本类型的值。对象的**弱引用**在它自己成为该对象的唯一引用时，不会阻止垃圾回收，

##### 创建 Weak Set

Weak Set 使用 `WeakSet` 构造器来创建，并包含 `add()` 方法、 `has()` 方法以及 `delete()` 方法。以下例子使用了这三个方法：

```js
let set = new WeakSet(),
    key = {};

// 将对象加入 set
set.add(key);

console.log(set.has(key));      // true

set.delete(key);

console.log(set.has(key));      // false 
```

使用 Weak Set 很像在使用正规的 Set 。你可以在 Weak Set 上添加、移除或检查引用，也可以给构造器传入一个可迭代对象来初始化 Weak Set 的值：

```js
let key1 = {},
    key2 = {},
    set = new WeakSet([key1, key2]);

console.log(set.has(key1));     // true
console.log(set.has(key2));     // true 
```

在本例中，一个数组被传给了 `WeakSet` 构造器。由于该数组包含了两个对象，这些对象就被添加到了 Weak Set 中。要记住若数组中包含了非对象的值，就会抛出错误，因为 `WeakSet` 构造器不接受基本类型的值。

##### Set 类型之间的关键差异

Weak Set 与正规 Set 之间最大的区别是对象的弱引用。此处有个例子说明了这种差异：

```js
let set = new WeakSet(),
    key = {};

// 将对象加入 set
set.add(key);

console.log(set.has(key));      // true

// 移除对于键的最后一个强引用，同时从 Weak Set 中移除
key = null; 
```

当此代码被执行后， Weak Set 中的 `key` 引用就不能再访问了。核实这一点是不可能的，因为需要把对于该对象的一个引用传递给 `has()` 方法（而只要存在其他引用， Weak Set 内部的弱引用就不会消失）。这会使得难以对 Weak Set 的引用特征进行测试，但 JS 引擎已经正确地将引用移除了，这一点你可以信任。

这些例子演示了 Weak Set 与正规 Set 的一些共有特征，但是它们还有一些关键的差异，即：

1.  对于 `WeakSet` 的实例，只要调用 `add()` 、 `has()` 或 `delete()` 方法时传入了非对象的参数，就会抛出错误；
2.  Weak Set 不可迭代，因此不能被用在 `for-of` 循环中；
3.  Weak Set 无法暴露出任何迭代器（例如 `keys()` 与 `values()` 方法），因此没有任何编程手段可用于判断 Weak Set 的内容；
4.  Weak Set 没有 `forEach()` 方法；
5.  Weak Set 没有 `size` 属性。

Weak Set 看起来功能有限，而这对于正确管理内存而言是必要的。一般来说，若只想追踪对象的引用，应当使用 Weak Set 而不是正规 Set 。

Set 给了你处理值列表的新方式，不过若需要给这些值添加额外信息，它就没用了。这就是 ES6 还添加了 Map 类型的原因。

### ES6 的 Map

ES6 的 `Map` 类型是键值对的有序列表，而键和值都可以是任意类型。键的比较使用的是 `Object.is()` ，因此你能将 `5` 与 `"5"` 同时作为键，因为它们类型不同。这与使用对象属性作为键的方式（指的是用对象来模拟 Map ）截然不同，因为对象的属性会被强制转换为字符串。

你可以调用 `set()` 方法并给它传递一个键与一个关联的值，来给 Map 添加项；此后使用键名来调用 `get()` 方法便能提取对应的值。例如：

```js
let map = new Map();
map.set("title", "Understanding ES6");
map.set("year", 2016);

console.log(map.get("title"));      // "Understanding ES6"
console.log(map.get("year"));       // 2016 
```

此例存储了两个键值对。 `"title"` 键存储了一个字符串，而 `"year"` 键则存储了一个数值，此后调用 `get()` 方法提取出了二者的值。如果任意一个键不存在于 Map 中， 则 `get()` 方法就会返回特殊值 `undefined` 。

你也可以将对象作为键，这也是从前使用对象属性来创建 Map 的变通方法所无法做到的。此处有个例子：

```js
let map = new Map(),
    key1 = {},
    key2 = {};

map.set(key1, 5);
map.set(key2, 42);

console.log(map.get(key1));         // 5
console.log(map.get(key2));         // 42 
```

此代码使用了对象 `key1` 与 `key2` 作为 Map 的键，并存储了两个不同的值。由于这些键不会被强制转换成其它形式，每个对象就都被认为是唯一的。这允许你给对象关联额外数据，而无须修改对象自身。

#### Map 的方法

Map 与 Set 共享了几个方法，这是有意的，允许你使用相似的方式来与 Map 及 Set 进行交互。以下三个方法在 Map 与 Set 上都存在：

*   `has(key)` ：判断指定的键是否存在于 Map 中；
*   `delete(key)` ：移除 Map 中的键以及对应的值；
*   `clear()` ：移除 Map 中所有的键与值。

Map 同样拥有 `size` 属性，用于指明包含了多少个键值对。以下代码用不同方式使用了这三种方法以及 `size` 属性：

```js
let map = new Map();
map.set("name", "Nicholas");
map.set("age", 25);

console.log(map.size);          // 2

console.log(map.has("name"));   // true
console.log(map.get("name"));   // "Nicholas"

console.log(map.has("age"));    // true
console.log(map.get("age"));    // 25

map.delete("name");
console.log(map.has("name"));   // false
console.log(map.get("name"));   // undefined
console.log(map.size);          // 1

map.clear();
console.log(map.has("name"));   // false
console.log(map.get("name"));   // undefined
console.log(map.has("age"));    // false
console.log(map.get("age"));    // undefined
console.log(map.size);          // 0 
```

与用于 Set 时一样， `size` 属性总是包含了 Map 中键值对的数量。此例中的 `Map` 实例起初有 `"name"` 与 `"age"` 两个键，因此传递这两个键给 `has()` 方法都会返回 `true` 。在 `"name"` 键被使用 `delete()` 方法移除后， `has()` 方法在接收 `"name"` 的时候就会返回 `false` 了，并且 `size` 属性表明 Map 的项减少了一个。之后 `clear()` 方法移除了残存的键， `has()` 方法此时对这两个键都会返回 `false` ，而 `size` 属性则变成了 0 。

`clear()` 方法是从 Map 中移除大量数据的快速方法，但同时也有一次性将大量数据添加到 Map 的方法：

#### Map 的初始化

依然与 Set 类似，你能将数组传递给 `Map` 构造器，以便使用数据来初始化一个 Map 。该数组中的每一项也必须是数组，内部数组的首个项会作为键，第二项则为对应值。因此整个 Map 就被这些双项数组所填充。例如：

```js
let map = new Map([["name", "Nicholas"], ["age", 25]]);

console.log(map.has("name"));   // true
console.log(map.get("name"));   // "Nicholas"
console.log(map.has("age"));    // true
console.log(map.get("age"));    // 25
console.log(map.size);          // 2 
```

通过构造器中的初始化， `"name"` 与 `"age"` 这两个键就被添加到 `map` 变量中。虽然由数组构成的数组看起来有点奇怪，这对于准确表示键来说却是必要的：因为键允许是任意数据类型，将键存储在数组中，是确保它们在被添加到 Map 之前不会被强制转换为其他类型的唯一方法。

#### Map 上的 forEach 方法

Map 的 `forEach()` 方法类似于 Set 与数组的同名方法，它接受一个能接收三个参数的回调函数：

1.  Map 中下个位置的值；
2.  该值所对应的键；
3.  目标 Map 自身。

回调函数的这些参数更紧密契合了数组 `forEach()` 方法的行为，即：第一个参数是值、第二个参数则是键（数组中的键是数值索引）。此处有个示例：

```js
let map = new Map([ ["name", "Nicholas"], ["age", 25]]);

map.forEach(function(value, key, ownerMap) {
    console.log(key + " " + value);
    console.log(ownerMap === map);
}); 
```

`forEach()` 的回调函数输出了传给它的信息。其中 `value` 与 `key` 被直接输出， `ownerMap` 与 `map` 进行了比较，说明它们是相等的。这就输出了：

```js
name Nicholas
true
age 25
true 
```

传递给 `forEach()` 的回调函数接收了每个键值对，按照键值对被添加到 Map 中的顺序。这种行为与在数组上调用 `forEach()` 方法有所不同，后者的回调函数会按数值索引的顺序接收到每一个项。

> 你也可以给 `forEach()` 提供第二个参数来指定回调函数中的 `this` 值，其行为与 Set 版本的 `forEach()` 一致。

#### Weak Map

Weak Map 对 Map 而言，就像 Weak Set 对 Set 一样： Weak 版本都是存储对象弱引用的方式。在 **Weak Map** 中，所有的键都必须是对象（尝试使用非对象的键会抛出错误），而且这些对象都是弱引用，不会干扰垃圾回收。当 Weak Map 中的键在 Weak Map 之外不存在引用时，该键值对会被移除。

Weak Map 的最佳用武之地，就是在浏览器中创建一个关联到特定 DOM 元素的对象。例如，某些用在网页上的 JS 库会维护一个自定义对象，用于引用该库所使用的每一个 DOM 元素，并且其映射关系会存储在内部的对象缓存中。

该方法的困难之处在于：如何判断一个 DOM 元素已不复存在于网页中，以便该库能移除此元素的关联对象。若做不到，该库就会继续保持对 DOM 元素的一个无效引用，并造成内存泄漏。使用 Weak Map 来追踪 DOM 元素，依然允许将自定义对象关联到每个 DOM 元素，而在此对象所关联的 DOM 元素不复存在时，它就会在 Weak Map 中被自动销毁。

> 必须注意的是， Weak Map 的键才是弱引用，而值不是。在 Weak Map 的值中存储对象会阻止垃圾回收，即使该对象的其他引用已全都被移除。

##### 使用 Weak Map

ES6 的 `WeakMap` 类型是键值对的无序列表，其中键必须是非空的对象，值则允许是任意类型。 `WeakMap` 的界面与 `Map` 的非常相似，都使用 `set()` 与 `get()` 方法来分别添加与提取数据：

```js
let map = new WeakMap(),
    element = document.querySelector(".element");

map.set(element, "Original");

let value = map.get(element);
console.log(value);             // "Original"

// 移除元素
element.parentNode.removeChild(element);
element = null;

// 该 Weak Map 在此处为空 
```

此例存储了一个键值对。 `element` 键是一个 DOM 元素，用于存储一个有关联的字符串值。将此 DOM 元素传递给 `get()` 方法，就能提取对应的值。随后将此 DOM 元素从页面文档中移除、并且将引用它的变量设置为 `null` ，则对应的数据也就会在 Weak Map 中被移除。

类似于 Weak Set ，没有任何办法可以确认 Weak Map 是否为空，因为它没有 `size` 属性。在其他引用被移除后，由于对键的引用不再有残留，也就无法调用 `get()` 方法来提取对应的值。 Weak Map 已经切断了对于该值的访问，其所占的内存在垃圾回收器运行时便会被释放。

##### Weak Map 的初始化

为了初始化 Weak Map ，需要把一个由数组构成的数组传递给 `WeakMap` 构造器。就像正规 Map 构造器那样，每个内部数组都应当有两个项，第一项是作为键的非空的对象，第二项则是对应的值（任意类型）。例如：

```js
let key1 = {},
    key2 = {},
    map = new WeakMap([[key1, "Hello"], [key2, 42]]);

console.log(map.has(key1));     // true
console.log(map.get(key1));     // "Hello"
console.log(map.has(key2));     // true
console.log(map.get(key2));     // 42 
```

对象 `key1` 与 `key2` 被用作 Weak Map 的键, `get()` 与 `has()` 方法则能访问它们。在传递给 `WeakMap` 构造器的参数中，若任意键值对使用了非对象的键，构造器就会抛出错误。

##### Weak Map 的方法

Weak Map 只有两个附加方法能用来与键值对交互。 `has()` 方法用于判断指定的键是否存在于 Map 中，而 `delete()` 方法则用于移除一个特定的键值对。 `clear()` 方法不存在，这是因为没必要对键进行枚举，并且枚举 Weak Map 也是不可能的，这与 Weak Set 相同。以下例子同时用到了 `has()` 与 `delete()` 方法：

```js
let map = new WeakMap(),
    element = document.querySelector(".element");

map.set(element, "Original");

console.log(map.has(element));   // true
console.log(map.get(element));   // "Original"

map.delete(element);
console.log(map.has(element));   // false
console.log(map.get(element));   // undefined 
```

此处一个 DOM 元素再次在 Weak Map 中被作为键来使用。对于查看一个引用是否正被用作 Weak Map 的键， `has()` 是非常有用的。但需要注意，必须要有对于该键的另一个非空引用，才能使用此方法。而使用 `delete()` 方法则会把键从 Weak Map 中强制移除，此后 `has()` 方法就会对该键返回 `false` ， `get()`方法则会返回 `undefined` 。

##### 对象的私有数据

虽然大多数开发者认为 Weak Map 的主要用途是关联数据与 DOM 元素，但仍然还存在许多可能的用法（并且毫无疑问，仍有一些用法尚未被发现）。 Weak Map 的一个实际应用就是在对象实例中存储私有数据。在 ES6 中对象的所有属性都是公开的，因此若想让数据对于对象自身可访问、而在其他条件下不可访问，那么你就需要使用一些创造力。研究以下例子：

```js
function Person(name) {
    this._name = name;
}

Person.prototype.getName = function() {
    return this._name;
}; 
```

此代码使用了下划线这种表示私有属性的公共约定，来表明一个成员应当被认为是私有的，不应从对象实例外进行修改，此处意图是：只允许用 `getName()` 来访问 `this._name` ，而不允许 `_name` 的值被修改。然而，毫无办法阻止任何人写入 `_name` 属性，所以它依然能够被有意或无意地改写。

在 ES5 中能够创建几乎真正私有的数据，只要在创建对象时使用类似下面的模式：

```js
var Person = (function() {

    var privateData = {},
        privateId = 0;

    function Person(name) {
        Object.defineProperty(this, "_id", { value: privateId++ });

        privateData[this._id] = {
            name: name
        };
    }

    Person.prototype.getName = function() {
        return privateData[this._id].name;
    };

    return Person;
}()); 
```

此例用 IIFE 包裹了 `Person` 的定义，其中含有两个私有属性： `privateData` 与 `privateId` 。 `privateData` 对象存储了每个实例的私有信息，而 `privateId` 则被用于为每个实例产生一个唯一 ID 。 当 `Person` 构造器被调用时，一个不可枚举、不可配置、不可写入的 `_id` 属性就被添加了。

接下来在 `privateData` 对象中建立了与实例 ID 对应的一个入口，其中存储着 `name` 的值。随后在 `getName()` 函数中，就能使用 `this._id` 作为 `privateData` 的键来提取该值。由于 `privateData` 无法从 IIFE 外部进行访问，实际的数据就是安全的，尽管 `this._id` 在 `privateData` 对象上依然是公开暴露的。

此方式的最大问题在于 `privateData` 中的数据永不会消失，因为在对象实例被销毁时没有任何方法可以获知该数据， `privateData` 对象就将永远包含多余的数据。这个问题现在可以换用 Weak Map 来解决了，如下：

```js
let Person = (function() {

    let privateData = new WeakMap();

    function Person(name) {
        privateData.set(this, { name: name });
    }

    Person.prototype.getName = function() {
        return privateData.get(this).name;
    };

    return Person;
}()); 
```

此版本的 `Person` 范例使用了 Weak Map 而不是对象来保存私有数据。由于 `Person` 对象的实例本身能被作为键来使用，于是也就无须再记录单独的 ID 。当 `Person` 构造器被调用时，将 `this` 作为键在 Weak Map 上建立了一个入口，而包含私有信息的对象成为了对应的值，其中只存放了 `name` 属性。通过将 `this` 传递给 `privateData.get()` 方法，以获取值对象并访问其 `name` 属性， `getName()` 函数便能提取私有信息。这种技术让私有信息能够保持私有状态，并且当与之关联的对象实例被销毁时，私有信息也会被同时销毁。

##### Weak Map 的用法与局限性

当决定是要使用 Weak Map 还是使用正规 Map 时，首要考虑因素在于你是否只想使用对象类型的键。如果你打算这么做，那么最好的选择就是 Weak Map 。因为它能确保额外数据在不再可用后被销毁，从而能优化内存使用并规避内存泄漏。

要记住 Weak Map 只为它们的内容提供了很小的可见度，因此你不能使用 `forEach()` 方法、 `size` 属性或 `clear()` 方法来管理其中的项。如果你确实需要一些检测功能，那么正规 Map 会是更好的选择，只是一定要确保留意内存的使用。

当然，若你想使用非对象的键，那么正规 Map 就是唯一选择。

### 总结

ES6 正式将 Set 与 Map 引入了 JS 。在此之前，开发者往往使用对象来模拟它们，但由于与对象属性有关的限制，这么做经常会遇到问题。

Set 是无重复值的有序列表。根据 `Object.is()` 方法来判断其中的值不相等，以保证无重复。 Set 会自动移除重复的值，因此你可以使用它来过滤数组中的重复值并返回结果。 Set 并不是数组的子类型，所以你无法随机访问其中的值。但你可以使用 `has()` 方法来判断某个值是否存在于 Set 中，或通过 `size` 属性来查看其中有多少个值。 `Set` 类型还拥有 `forEach()` 方法，用于处理每个值。

Weak Set 是只能包含对象的特殊 Set 。其中的对象使用弱引用来存储，意味着当 Weak Set 中的项是某个对象的仅存引用时，它不会屏蔽垃圾回收。由于内存管理的复杂性， Weak Set 的内容不能被检查，因此最好将 Weak Set 仅用于追踪需要被归组在一起的对象。

Map 是有序的键值对，其中的键允许是任何类型。与 Set 相似，通过调用 `Object.is()` 方法来判断重复的键，这意味着能将数值 `5` 与字符串 `"5"` 作为两个相对独立的键。使用 `set()` 方法能将任何类型的值关联到某个键上，并且该值此后能用 `get()` 方法提取出来。 Map 也拥有一个 `size` 属性与一个 `forEach()` 方法，让项目访问更容易。

Weak Map 是只能包含对象类型的键的特殊 Map 。与 Weak Set 相似，键的对象引用是弱引用，因此当它是某个对象的仅存引用时，也不会屏蔽垃圾回收。当键被回收之后，所关联的值也同时从 Weak Map 中被移除。对于和对象相关联的附加信息来说，若要在访问它们的代码之外对其进行生命周期管理（也就是说，当在对象外部移除对象的引用时，要求其私有数据也能一并被销毁），则 Weak Map 在内存管理方面的特性让它们成为了唯一合适的选择。