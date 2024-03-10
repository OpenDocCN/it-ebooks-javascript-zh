# 第六章 符号与符号属性

## 第六章 符号与符号属性

在 JS 已有的基本类型（字符串、数值、布尔类型、 `null` 与 `undefined` ）之外， ES6 引入了一种新的基本类型：符号（ Symbol ）。 符号起初被设计用于创建对象私有成员，而这也是 JS 开发者期待已久的特性。在符号诞生之前，将字符串作为属性名称导致属性可以被轻易访问，无论命名规则如何。而“私有名称”意味着开发者可以创建非字符串类型的属性名称，由此可以防止使用常规手段来探查这些名称。

“私有名称”提案最终发展成为 ES6 中的符号，而本章将会教你如何有效使用它。虽然它只保留了实现细节（即：引入了非字符串类型的属性名）而丢弃了私有性意图，但它仍然显著有别于对象的其余属性。

*   创建符号值
*   使用符号值
*   共享符号值
*   符号值的转换
*   检索符号属性
*   使用知名符号暴露内部方法
    *   Symbol.hasInstance 属性
    *   Symbol.isConcatSpreadable
    *   Symbol.match 、 Symbol.replace 、 Symbol.search 与 Symbol.split
    *   Symbol.toPrimitive
    *   Symbol.toStringTag
        *   识别问题的变通解决方法
        *   ES6 给出的答案
    *   Symbol.unscopables
*   总结

### 创建符号值

符号没有字面量形式，这在 JS 的基本类型中是独一无二的，有别于布尔类型的 `true` 或数值类型的 `42` 等等。你可以使用全局 `Symbol` 函数来创建一个符号值，正如下面这个例子：

```js
let firstName = Symbol();
let person = {};

person[firstName] = "Nicholas";
console.log(person[firstName]);     // "Nicholas" 
```

此代码创建了一个符号类型的 `firstName` 变量，并将它作为 `person` 对象的一个属性，而每次访问该属性都要使用这个符号值。为符号变量适当命名是个好主意，这样你就可以很容易地说明它的含义。

> 由于符号值是基本类型的值，因此调用 `new Symbol()` 将会抛出错误。你可以通过 `new Object(yourSymbol)` 来创建一个符号实例，但尚不清楚这能有什么作用。

`Symbol` 函数还可以接受一个额外的参数用于描述符号值，该描述并不能用来访问对应属性，但它能用于调试，例如：

```js
let firstName = Symbol("first name");
let person = {};

person[firstName] = "Nicholas";

console.log("first name" in person);        // false
console.log(person[firstName]);             // "Nicholas"
console.log(firstName);                     // "Symbol(first name)" 
```

符号的描述信息被存储在内部属性 `[[Description]]` 中，当符号的 `toString()` 方法被显式或隐式调用时，该属性都会被读取。在本例中， `console.log()` 隐式调用了 `firstName` 变量的 `toString()` 方法，于是描述信息就被输出到日志。此外没有任何办法可以从代码中直接访问 `[[Description]]` 属性。我建议始终应给符号提供描述信息，以便更好地阅读代码或进行调试。

> **识别符号值**
> 
> 由于符号是基本类型的值，因此你可以使用 `typeof` 运算符来判断一个变量是否为符号。 ES6 扩充了 `typeof` 的功能以便让它在作用于符号值的时候能够返回 `"symbol"`，例如：
> 
> ```js
> let symbol = Symbol("test symbol");
> console.log(typeof symbol);         // "symbol" 
> ```
> 
> 尽管有其他方法可以判断一个变量是否为符号， `typeof` 运算符依然是最准确、最优先的判别手段。

### 使用符号值

你可以在任意能使用“需计算属性名”的场合使用符号。此前的例子已经展示了符号的方括号用法，而你还能在对象的“需计算字面量属性名”中使用符号，此外还可以在 `Object.defineProperty()` 或 `Object.defineProperties()` 调用中使用它，例如：

```js
let firstName = Symbol("first name");

// 使用一个需计算字面量属性
let person = {
    [firstName]: "Nicholas"
};

// 让该属性变为只读
Object.defineProperty(person, firstName, { writable: false });

let lastName = Symbol("last name");

Object.defineProperties(person, {
    [lastName]: {
        value: "Zakas",
        writable: false
    }
});

console.log(person[firstName]);     // "Nicholas"
console.log(person[lastName]);      // "Zakas" 
```

这个例子首先使用对象的“需计算字面量属性”方式创建了一个符号类型的属性 `firstName` ，该属性默认不可枚举，此行为与非符号类型的“需计算字面量属性名”正相反。下一行代码将该属性设置为只读。接下来，使用 `Object.defineProperties()` 方法创建了一个只读的符号类型属性 `lastName` ，而此时再次使用了“需计算字面量属性”方式，并将其作为第二个调用参数的一部分。

既然能在任意可使用“需计算属性名”的场合使用符号，你就需要一种在不同代码段中共享符号值的方式，以便更有效地使用它们。

### 共享符号值

你或许想在不同的代码段中使用相同的符号值，例如：假设在应用中需要在两个不同的对象类型中使用同一个符号属性，用来表示一个唯一标识符。跨越文件或代码来追踪符号值是很困难并且易错的，为此， ES6 提供了“全局符号注册表”供你在任意时间点进行访问。

若你想创建共享符号值，应使用 `Symbol.for()` 方法而不是 `Symbol()` 方法。 `Symbol.for()` 方法仅接受单个字符串类型的参数，作为目标符号值的标识符，同时此参数也会成为该符号的描述信息。例如：

```js
let uid = Symbol.for("uid");
let object = {};

object[uid] = "12345";

console.log(object[uid]);       // "12345"
console.log(uid);               // "Symbol(uid)" 
```

`Symbol.for()` 方法首先会搜索全局符号注册表，看是否存在一个键值为 `"uid"` 的符号值。若是，该方法会返回这个已存在的符号值；否则，会创建一个新的符号值，并使用该键值将其记录到全局符号注册表中，然后返回这个新的符号值。这就意味着此后使用同一个键值去调用 `Symbol.for()` 方法都将会返回同一个符号值，就像下面这个例子：

```js
let uid = Symbol.for("uid");
let object = {
    [uid]: "12345"
};

console.log(object[uid]);       // "12345"
console.log(uid);               // "Symbol(uid)"

let uid2 = Symbol.for("uid");

console.log(uid === uid2);      // true
console.log(object[uid2]);      // "12345"
console.log(uid2);              // "Symbol(uid)" 
```

本例中， `uid` 与 `uid2` 包含同一个符号值，因此它们可以互换使用。第一次调用 `Symbol.for()` 创建了这个符号值，而第二次调用则从全局符号注册表中将其检索了出来。

共享符号值还有另一个独特用法，你可以使用 `Symbol.keyFor()` 方法在全局符号注册表中根据符号值检索出对应的键值，例如：

```js
let uid = Symbol.for("uid");
console.log(Symbol.keyFor(uid));    // "uid"

let uid2 = Symbol.for("uid");
console.log(Symbol.keyFor(uid2));   // "uid"

let uid3 = Symbol("uid");
console.log(Symbol.keyFor(uid3));   // undefined 
```

注意：使用符号值 `uid` 与 `uid2` 都返回了键值 `"uid"` ，而符号值 `uid3` 在全局符号注册表中并不存在，因此没有关联的键值， `Symbol.keyFor()` 方法只会返回 `undefined` 。

> 全局符号注册表类似于全局作用域，是一个共享环境，这意味着你不应当假设某些值是否已存在于其中。在使用第三方组件时，为符号的键值使用命名空间能够减少命名冲突的可能性，举个例子： jQuery 代码应当为它的所有键值使用 `"jquery."` 的前缀，如 `"jquery.element"` 或类似的形式。

### 符号值的转换

类型转换是 JS 语言重要的一部分，能够非常灵活地将一种数据类型转换为另一种。然而符号类型在进行转换时非常不灵活，因为其他类型缺乏与符号值的合理等价，尤其是符号值无法被转换为字符串值或数值。因此将符号作为属性所达成的效果，是其他类型所无法替代的。

本章之前的例子使用了 `console.log()` 来展示符号值的输出，能这么做是由于自动调用了符号的 `String()` 方法来产生输出。你也可以直接调用 `String()` 方法来获取相同结果，例如：

```js
let uid = Symbol.for("uid"),
    desc = String(uid);

console.log(desc);              // "Symbol(uid)" 
```

`String()` 方法调用了 `uid.toString()` 来获取符号的字符串描述信息。但若你想直接将符号转换为字符串，则会引发错误：

```js
let uid = Symbol.for("uid"),
    desc = uid + "";            // 引发错误！ 
```

将 `uid` 与空字符串相连接，会首先要求把 `uid` 转换为一个字符串，而这会引发错误，从而阻止了转换行为。

相似地，你不能将符号转换为数值，对符号使用所有数学运算符都会引发错误，例如：

```js
let uid = Symbol.for("uid"),
    sum = uid / 1;            // 引发错误！ 
```

此例试图把符号值除以 1 ，同样引发了错误。无论对符号使用哪种数学运算符都会导致错误，但使用逻辑运算符则不会，因为符号值在逻辑运算中会被认为等价于 `true` （就像 JS 中其他的非空值那样）。

### 检索符号属性

`Object.keys()` 与 `Object.getOwnPropertyNames()` 方法可以检索对象的所有属性名称，前者返回所有的可枚举属性名称，而后者则返回所有属性名称而无视其是否可枚举。然而两者都不能返回符号类型的属性，以保持它们在 ES5 中的功能不发生变化。而 ES6 新增了 `Object.getOwnPropertySymbols()` 方法，以便让你可以检索对象的符号类型属性。

`Object.getOwnPropertySymbols()` 方法会返回一个数组，包含了对象自有属性名中的符号值，例如：

```js
let uid = Symbol.for("uid");
let object = {
    [uid]: "12345"
};

let symbols = Object.getOwnPropertySymbols(object);

console.log(symbols.length);        // 1
console.log(symbols[0]);            // "Symbol(uid)"
console.log(object[symbols[0]]);    // "12345" 
```

这段代码中， `object` 对象只拥有一个名为 `uid` 的符号类型属性， `Object.getOwnPropertySymbols()` 方法返回的数组包含了这个符号值。

所有对象初始情况下都不包含任何自有符号类型属性，但对象可以从它们的原型上继承符号类型属性。 ES6 预定义了一些此类属性，它们被称为“知名符号”。

### 使用知名符号暴露内部方法

ES5 的中心主题之一是披露并定义了一些魔术般的成分，而这些部分是当时开发者所无法自行模拟的。 ES6 延续了这些工作，对原先属于语言内部逻辑的部分进行了进一步的暴露，允许使用符号类型的原型属性来定义某些对象的基础行为。

ES6 定义了“知名符号”来代表 JS 中一些公共行为，而这些行为此前被认为只能是内部操作。每一个知名符号都对应全局 `Symbol` 对象的一个属性，例如 `Symbol.create` 。

这些知名符号是：

*   `Symbol.hasInstance` ：供 `instanceof` 运算符使用的一个方法，用于判断对象继承关系。
*   `Symbol.isConcatSpreadable` ：一个布尔类型值，在集合对象作为参数传递给 `Array.prototype.concat()` 方法时，指示是否要将该集合的元素扁平化。
*   `Symbol.iterator` ：返回迭代器（参阅第七章）的一个方法。
*   `Symbol.match` ：供 `String.prototype.match()` 函数使用的一个方法，用于比较字符串。
*   `Symbol.replace` ：供 `String.prototype.replace()` 函数使用的一个方法，用于替换子字符串。
*   `Symbol.search` ：供 `String.prototype.search()` 函数使用的一个方法，用于定位子字符串。
*   `Symbol.species` ：用于产生派生对象（参阅第八章）的构造器。
*   `Symbol.split` ：供 `String.prototype.split()` 函数使用的一个方法，用于分割字符串。
*   `Symbol.toPrimitive` ：返回对象所对应的基本类型值的一个方法。
*   `Symbol.toStringTag` ：供 `String.prototype.toString()` 函数使用的一个方法，用于创建对象的描述信息。
*   `Symbol.unscopables` ：一个对象，该对象的属性指示了哪些属性名不允许被包含在 `with` 语句中。

一些公用的知名符号将在下面诸小节进行论述，而其余知名符号则会在本书其他部分中讨论，以保证它们出现在正确的上下文中。

> 重写知名符号所定义的方法，会把一个普通对象改变成奇异对象，因为它改变了一些默认的内部行为。这并不会对你的代码造成实际影响，它只是改变了规范所描述的对象特征。

#### Symbol.hasInstance 属性

每个函数都具有一个 `Symbol.hasInstance` 方法，用于判断指定对象是否为本函数的一个实例。这个方法定义在 `Function.prototype` 上，因此所有函数都继承了面对 `instanceof` 运算符时的默认行为。 `Symbol.hasInstance` 属性自身是不可写入、不可配置、不可枚举的，从而保证它不会被错误地重写。

`Symbol.hasInstance` 方法只接受单个参数，即需要检测的值。如果该值是本函数的一个实例，则方法会返回 `true` 。为了理解该方法是如何工作的，可研究下述代码：

```js
obj instanceof Array; 
```

这句代码等价于：

```js
ArraySymbol.hasInstance; 
```

ES6 从本质上将 `instanceof` 运算符重定义为上述方法调用的简写语法，这样使用 `instanceof` 便会触发一次方法调用，实际上允许你改变该运算符的工作。

例如，假设你想定义一个函数，使得任意对象都不会被判断为该函数的一个实例，你可以采用硬编码的方式让该函数的 `Symbol.hasInstance` 方法始终返回 `false` ，就像这样：

```js
function MyObject() {
    // ...
}

Object.defineProperty(MyObject, Symbol.hasInstance, {
    value: function(v) {
        return false;
    }
});

let obj = new MyObject();

console.log(obj instanceof MyObject);       // false 
```

要重写一个不可写入的属性，你必须像这个例子一样使用 `Object.defineProperty()` 。此代码将 `Symbol.hasInstance` 方法重写为一个始终返回 `false` 的函数，所以此后即使传入的对象确实是 `MyObject` 类的一个实例， `instanceof` 运算符仍然会返回 `false` 。

当然，你可以基于各种条件来决定一个值是否应当被判断为某个类的实例。例如，将介于 1 到 100 之间的数值认定为一个特殊的数值类型，为此你可以书写如下代码：

```js
function SpecialNumber() {
    // empty
}

Object.defineProperty(SpecialNumber, Symbol.hasInstance, {
    value: function(v) {
        return (v instanceof Number) && (v >=1 && v <= 100);
    }
});

let two = new Number(2),
    zero = new Number(0);

console.log(two instanceof SpecialNumber);    // true
console.log(zero instanceof SpecialNumber);   // false 
```

此代码重写了 `Symbol.hasInstance` 方法，在目标对象是数值对象的实例、并且其值介于 1 到 100 之间时，返回 `true` 。于是， `SpecialNumber` 类会把变量 `two` 判断为自身的一个实例，即使二者之间并不存在直接的定义关联。需要注意的是： `instanceof` 的操作数必须是一个对象，以便触发 `Symbol.hasInstance` 调用；若操作数并非对象， `instanceof` 只会简单地返回 `false` 。

> 你可以重写所有内置函数（例如 `Date` 或 `Error` ）的 `Symbol.hasInstance` 属性，但我并不建议这么做，因为这会让你的代码变得难以预测而又混乱。最好仅在必要时对你自己的函数重写 `Symbol.hasInstance` 。

#### Symbol.isConcatSpreadable

JS 在数组上设计了 `concat()` 方法用于将两个数组连接到一起，此处示范了如何使用该方法：

```js
let colors1 = [ "red", "green" ],
    colors2 = colors1.concat([ "blue", "black" ]);

console.log(colors2.length);    // 4
console.log(colors2);           // ["red","green","blue","black"] 
```

此代码将一个新数组连接到 `colors1` 末尾，并创建了 `colors2` ，后者包含了前两个数组中所有的项。不过， `concat()` 方法也可以接受非数组的参数，此时这些参数只是简单地被添加到数组末尾，例如：

```js
let colors1 = [ "red", "green" ],
    colors2 = colors1.concat([ "blue", "black" ], "brown");

console.log(colors2.length);    // 5
console.log(colors2);           // ["red","green","blue","black","brown"] 
```

此代码向 `concat()` 方法传递了一个额外参数 `"brown"` ，使得它成为数组 `colors2` 的第 5 项。为何数组类型的参数与字符串类型的参数会被区别对待？这是因为 JS 规范要求此时数组类型的参数需要被自动分离出各个子项，而其他类型的参数无需如此处理。在 ES6 之前，没有任何手段可以改变这种行为。

`Symbol.isConcatSpreadable` 属性是一个布尔类型的属性，它表示目标对象拥有长度属性与数值类型的键、并且数值类型键所对应的属性值在参与 `concat()` 调用时需要被分离为个体。该符号与其他的知名符号不同，默认情况下并不会作为任意常规对象的属性。它只出现在特定类型的对象上，用来标示该对象在作为 `concat()` 参数时应如何工作，从而有效改变该对象的默认行为。你可以用它来定义任意类型的对象，让该对象在参与 `concat()` 调用时能够表现得像数组一样，例如：

```js
let collection = {
    0: "Hello",
    1: "world",
    length: 2,
    [Symbol.isConcatSpreadable]: true
};

let messages = [ "Hi" ].concat(collection);

console.log(messages.length);    // 3
console.log(messages);           // ["hi","Hello","world"] 
```

本例中的 `collection` 对象的特征类似于数组：拥有长度属性以及两个数值类型的键，并且 `Symbol.isConcatSpreadable` 属性值被设为 `true` ，用于指示该对象在被添加到数组时应该使用分离的属性值。当 `collection` 对象被传递给 `concat()` 方法时， `"Hello"` 与 `"world"` 被分离为独立的项，并跟在 `"hi"` 元素之后。

> 你也可以将数组的子类的 `Symbol.isConcatSpreadable` 属性值设为 `false` ，用于在 `concat()` 调用时避免项目被分离。子类的介绍位于第八章。

#### Symbol.match 、 Symbol.replace 、 Symbol.search 与 Symbol.split

在 JS 中，字符串与正则表达式有着密切的联系，尤其是字符串具有几个可以接受正则表达式作为参数的方法：

*   `match(regex)` ：判断指定字符串是否与一个正则表达式相匹配；
*   `replace(regex, replacement)` ：对正则表达式的匹配结果进行替换；
*   `search(regex)` ：在字符串内对正则表达式的匹配结果进行定位；
*   `split(regex)` ：使用正则表达式将字符串分割为数组。

这些与正则表达式交互的方法，在 ES6 之前其实现细节是对开发者隐藏的，使得开发者无法将自定义对象模拟成正则表达式。而 ES6 定义了 4 个符号以及对应的方法，将原生行为外包到内置的 `RegExp` 对象上。

这 4 个符号表示可以将正则表达式作为对应方法的第一个参数传入， `Symbol.match` 对应 `match()` 方法， `Symbol.replace` 对应 `replace()` ， `Symbol.search` 对应 `search()` ， `Symbol.split` 则对应 `split()` 。这些符号属性被定义在 `RegExp.prototype` 上作为默认实现，以供对应的字符串方法使用。

了解这些之后，你就可以创建一个类似于正则表达式的对象，以便配合字符串的那些方法使用。在代码中使用下述的符号函数即可：

*   `Symbol.match` ：此函数接受一个字符串参数，并返回一个包含匹配结果的数组；若匹配失败，则返回 `null` 。
*   `Symbol.replace` ：此函数接受一个字符串参数与一个替换用的字符串，并返回替换后的结果字符串。
*   `Symbol.search` ：此函数接受一个字符串参数，并返回匹配结果的数值索引；若匹配失败，则返回 -1。
*   `Symbol.split` ：此函数接受一个字符串参数，并返回一个用匹配值分割而成的字符串数组。

在对象上定义这些属性，允许你创建能够进行模式匹配的对象，而无需使用正则表达式，并且允许在任何需要正则表达式的方法中使用该对象。这里有一个例子，展示了这些符号的用法：

```js
// 有效等价于 /^.{10}$/
let hasLengthOf10 = {
    [Symbol.match]: function(value) {
        return value.length === 10 ? [value.substring(0, 10)] : null;
    },
    [Symbol.replace]: function(value, replacement) {
        return value.length === 10 ?
            replacement + value.substring(10) : value;
    },
    [Symbol.search]: function(value) {
        return value.length === 10 ? 0 : -1;
    },
    [Symbol.split]: function(value) {
        return value.length === 10 ? ["", ""] : [value];
    }
};

let message1 = "Hello world",   // 11 characters
    message2 = "Hello John";    // 10 characters

let match1 = message1.match(hasLengthOf10),
    match2 = message2.match(hasLengthOf10);

console.log(match1);            // null
console.log(match2);            // ["Hello John"]

let replace1 = message1.replace(hasLengthOf10, "Howdy!"),
    replace2 = message2.replace(hasLengthOf10, "Howdy!");

console.log(replace1);          // "Hello world"
console.log(replace2);          // "Howdy!"

let search1 = message1.search(hasLengthOf10),
    search2 = message2.search(hasLengthOf10);

console.log(search1);           // -1
console.log(search2);           // 0

let split1 = message1.split(hasLengthOf10),
    split2 = message2.split(hasLengthOf10);

console.log(split1);            // ["Hello world"]
console.log(split2);            // ["", ""] 
```

`hasLengthOf10` 对象模拟了正则表达式的工作方式，在字符串长度恰好为 10 的时候起作用。 `hasLengthOf10` 对象上的四个方法都对相应的符号属性进行了实现，并依次在两个字符串上被调用。第一个字符串 `message1` 长度为 11 ，因此不会匹配成功；而字符串 `message2` 长度为 10，可以正确匹配。尽管 `hasLengthOf10` 对象不是正则表达式，但它仍然作为参数传递给这些字符串方法，并能够正常工作。

虽然这仅仅是一个简单的例子，但它表明可以进行比现有正则表达式功能更复杂的匹配，这在自定义模式匹配方面开启了更多可能性。

#### Symbol.toPrimitive

JS 经常在使用特定运算符的时候试图进行隐式转换，以便将对象转换为基本类型值。例如，当你使用相等（ == ）运算符来对字符串与对象进行比较的时候，该对象会在比较之前被转换为一个基本类型值。到底转换为什么基本类型值，在此前属于内部操作，而 ES6 则通过 `Symbol.toPrimitive` 方法将其暴露出来，以便让对应方法可以被修改。

`Symbol.toPrimitive` 方法被定义在所有常规类型的原型上，规定了在对象被转换为基本类型值的时候会发生什么。当需要转换时， `Symbol.toPrimitive` 会被调用，并按照规范传入一个提示性的字符串参数。该参数有 3 种可能：当参数值为 `"number"` 的时候， `Symbol.toPrimitive` 应当返回一个数值；当参数值为 `"string"` 的时候，应当返回一个字符串；而当参数为 `"default"` 的时候，对返回值类型没有特别要求。

对于大部分常规对象，“数值模式”依次会有下述行为：

1.  调用 `valueOf()` 方法，如果方法返回值是一个基本类型值，那么返回它；
2.  否则，调用 `toString()` 方法，如果方法返回值是一个基本类型值，那么返回它；
3.  否则，抛出一个错误。

类似的，对于大部分常规对象，“字符串模式”依次会有下述行为：

1.  调用 `toString()` 方法，如果方法返回值是一个基本类型值，那么返回它；
2.  否则，调用 `valueOf()` 方法，如果方法返回值是一个基本类型值，那么返回它；
3.  否则，抛出一个错误。

在多数情况下，常规对象的默认模式都等价于数值模式（只有 `Date` 类型例外，它默认使用字符串模式）。通过定义 `Symbol.toPrimitive` 方法，你可以重写这些默认的转换行为。

> “默认模式”只在使用 == 运算符、 + 运算符、或者传递单一参数给 `Date` 构造器的时候被使用，而大部分运算符都使用字符串模式或是数值模式。

使用 `Symbol.toPrimitive` 属性并将一个函数赋值给它，便可以重写默认的转换行为，例如：

```js
function Temperature(degrees) {
    this.degrees = degrees;
}

Temperature.prototype[Symbol.toPrimitive] = function(hint) {

    switch (hint) {
        case "string":
            return this.degrees + "\u00b0"; // 温度符号

        case "number":
            return this.degrees;

        case "default":
            return this.degrees + " degrees";
    }
};

let freezing = new Temperature(32);

console.log(freezing + "!");            // "32 degrees!"
console.log(freezing / 2);              // 16
console.log(String(freezing));          // "32°" 
```

这段脚本定义了一个 `Temperature` 构造器，并重写了其原型上的 `Symbol.toPrimitive` 方法。返回值会依据方法的提示性参数而有所不同，可以使用字符串模式、数值模式或是默认模式，而该提示性参数会在调用时由 JS 引擎自动填写。字符串模式中， `Temperature` 函数返回的温度会附带着 Unicode 温度符号；数值模式只会返回温度数值；而默认模式中，返回的温度会附带着字符串 `"degrees"` 。

此后的三个 log 语句分别触发了不同的提示性参数值： + 运算符使用 `"default"` 触发了默认模式； / 运算符使用 `"number"` 触发了数值模式；而 `String()` 函数则使用了 `"string"` 触发了字符串模式。允许在三种模式下返回互不相同的结果，但一般来说默认模式的返回值都会等于字符串模式或数值模式。

#### Symbol.toStringTag

JS 最有趣的课题之一是在多个不同的全局执行环境中使用，这种情况会在浏览器页面包含内联帧（ iframe ）的时候出现，此时页面与内联帧均拥有各自的全局执行环境。大多数情况下这并不是一个问题，使用一些轻量级的转换操作就能够在不同的运行环境之间传递数据。问题出现在想要识别目标对象到底是什么类型的时候，而此时该对象已经在环境之间经历了传递。

该问题的典型例子就是从内联帧向容器页面传递数组，或者反过来。在 ES6 术语中，内联帧与包含它的容器页面分别拥有一个不同的“域”，以作为 JS 的运行环境，每个“域”都拥有各自的全局作用域以及各自的全局对象拷贝。无论哪个“域”创建的数组都是正规的数组，但当它跨域进行传递时，使用 `instanceof Array` 进行检测却会得到 `false` 的结果，因为该数组是由另外一个“域”的数组构造器创建的，有别于当前“域”的数组构造器。

##### 识别问题的变通解决方法

面对这个问题，开发者迅速找到了识别数组的一个好办法，他们发现通过调用常规的 `toString()` 方法，就会得到一个可预期的字符串结果。因此，很多 JS 库都包含了如下函数：

```js
function isArray(value) {
    return Object.prototype.toString.call(value) === "[object Array]";
}

console.log(isArray([]));   // true 
```

这看起来是一种迂回方式，但它在任何浏览器中都能非常准确地识别数组。在数组对象上调用 `toString()` 方法没什么用处，因为它会返回由数组元素拼接成的字符串；然而若在 `Object.prototype` 上调用 `toString()` 方法，却恰巧能达到目的：返回值会包含名为 `[[Class]]` 的内部定义名称。开发者可以在对象上使用这个方法，以获知 JS 引擎将该对象判断为什么类型。

开发者迅速意识到基于这种行为的不变性，可以用其来区别原生对象与开发者自建对象，其中最重要的范例就是 ES5 的 `JSON` 对象。

在 ES5 之前，许多开发者都使用了 Douglas Crockford 的 json2.js 脚本，用来创建全局的 `JSON` 对象。在浏览器开始实现 `JSON` 全局对象之后，区分全局 `JSON` 对象是 JS 运行环境自带的、还是由库文件引入的，就变得非常必要。使用与识别数组相同的技术，很多开发者创建了如下的函数：

```js
function supportsNativeJSON() {
    return typeof JSON !== "undefined" &&
        Object.prototype.toString.call(JSON) === "[object JSON]";
} 
```

`Object.prototype` 的特性允许开发者跨越内联帧边界去识别数组，而使用相同方式可以辨别 `JSON` 对象是否为原生的。非原生的 `JSON` 对象会返回 `[object Object]` ，而原生的 `JSON` 对象则会返回 `[object JSON]` 。这类方法也成为了识别原生对象的事实标准。

##### ES6 给出的答案

ES6 通过 `Symbol.toStringTag` 重定义了相关行为，该符号代表了所有对象的一个属性，定义了 `Object.prototype.toString.call()` 被调用时应当返回什么值。对于数组来说，在 `Symbol.toStringTag` 属性中存储了 `"Array"` 值，于是该函数的返回值也就是 `"Array"` 。

同样，你可以在自设对象上定义 `Symbol.toStringTag` 的值：

```js
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = "Person";

let me = new Person("Nicholas");

console.log(me.toString());                         // "[object Person]"
console.log(Object.prototype.toString.call(me));    // "[object Person]" 
```

本例在 `Person` 的原型上定义了 `Symbol.toStringTag` 属性，用于提供它的默认的字符串表现形式。由于 `Person` 的原型继承了 `Object.prototype.toString()` 方法， `Symbol.toStringTag` 的返回值在调用 `me.toString()` 的时候也会被使用。不过，你依然可以在该对象上定义你自己的 `toString()` 方法，让它有不同的返回值，而不用影响 `Object.prototype.toString.call()` 方法。这里有个例子：

```js
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = "Person";

Person.prototype.toString = function() {
    return this.name;
};

let me = new Person("Nicholas");

console.log(me.toString());                         // "Nicholas"
console.log(Object.prototype.toString.call(me));    // "[object Person]" 
```

这段代码让 `Object.prototype.toString.call()` 返回 `name` 属性的值。由于 `Person` 类的实例不再继承 `Object.prototype.toString()` 方法，调用 `me.toString()` 会显示不同的结果。

> 除非进行了特殊指定，否则所有对象都会从 `Object.prototype` 继承 `Symbol.toStringTag` 属性，其默认的属性值是字符串 `"Object"` 。

对于开发者自定义对象， `Symbol.toStringTag` 的返回值不受任何限制。例如，你可以自由使用 `"Array"` 作为 `Symbol.toStringTag` 属性的值，像这样：

```js
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = "Array";

Person.prototype.toString = function() {
    return this.name;
};

let me = new Person("Nicholas");

console.log(me.toString());                         // "Nicholas"
console.log(Object.prototype.toString.call(me));    // "[object Array]" 
```

在这段代码中，调用 `Object.prototype.toString()` 的结果是 `"[object Array]"` ，与在真实数组上调用的结果完全一样。这一点明确证实 `Object.prototype.toString()` 不再是用于识别对象类型的可靠方法。

改变原生对象的字符串标签也是可能的，只需要在对象的原型上对 `Symbol.toStringTag` 进行赋值，例如：

```js
Array.prototype[Symbol.toStringTag] = "Magic";

let values = [];

console.log(Object.prototype.toString.call(values));    // "[object Magic]" 
```

本例重写了数组的 `Symbol.toStringTag` 属性，导致 `Object.prototype.toString()` 被调用时会返回 `"[object Magic]"` 。尽管我建议不要用这种方式修改内置对象，但语言本身并没有禁止该行为。

#### Symbol.unscopables

`with` 语句是 JS 语言中最有争议的部分之一。 `with` 语句原本被设计用于减少重复代码的输入，但此后却遭受了全面的批评，因为它让代码变得更难理解，并且有负面性能影响，同时还易出错。

最终 `with` 语句在严格模式下被禁用了，而此限制同样影响了类与模块，因为它们无需指定就会自动工作在严格模式下。

尽管将来的代码无疑会停用 `with` 语句，但 ES6 仍然在非严格模式中提供了对于 `with` 语句的支持，以便向下兼容。为此需要寻找方法让使用 `with` 语句的代码能够适当地继续工作。

为了理解这个任务的复杂性，可研究如下代码：

```js
let values = [1, 2, 3],
    colors = ["red", "green", "blue"],
    color = "black";

with(colors) {
    push(color);
    push(...values);
}

console.log(colors);    // ["red", "green", "blue", "black", 1, 2, 3] 
```

在此例中， `with` 语句内的两次 `push()` 调用等价于 `colors.push()` ，因为 `with` 语句为 `push` 添加了局部绑定； `color` 则引用了在 `with` 语句之外定义的变量；而 `values` 的本意也是如此。

但 ES6 为数组添加了一个 `values` 方法（可查阅第七章：“迭代器与生成器”)，这意味着在 ES6 的环境中， `with` 语句内部的 `values` 并不会指向局部变量 `values` ，而是会指向数组的 `values` 方法，从而会破坏代码的意图。这也就是 `Symbol.unscopables` 符号出现的理由。

`Symbol.unscopables` 符号在 `Array.prototype` 上使用，以指定哪些属性不允许在 `with` 语句内被绑定。 `Symbol.unscopables` 属性是一个对象，当提供该属性时，它的键就是用于忽略 `with` 语句绑定的标识符，键值为 `true` 代表屏蔽绑定。以下是数组的 `Symbol.unscopables` 属性的默认值：

```js
// 默认内置在 ES6 中
Array.prototype[Symbol.unscopables] = Object.assign(Object.create(null), {
    copyWithin: true,
    entries: true,
    fill: true,
    find: true,
    findIndex: true,
    keys: true,
    values: true
}); 
```

`Symbol.unscopables` 对象使用 `Object.create(null)` 创建，因此没有原型，并包含了 ES6 数组所有的新方法（可参阅第七章“迭代器与生成器”、第九章“数组”）。在 `with` 语句内并不会对这些方法进行绑定，因此旧代码可以继续工作而不会出问题。

一般来说，你不需要在你自定义的对象上设置 `Symbol.unscopables` 属性，除非使用了 `with` 语句、并修改了代码库中已有的对象。

### 总结

符号是 JS 新引入的基本类型值，它用于创建不可枚举的属性，并且这些属性在不引用符号的情况下是无法访问的。

虽然符号类型的属性不是真正的私有属性，但它们难以被无意修改，因此在需要提供保护以防止开发者改动的场合中，它们非常合适。

你可以为符号提供描述信息以便更容易地辨识它们的值。全局符号注册表允许你使用相同的描述信息，以便在不同的代码段中共享符号值，这样相同的符号值就可以在不同位置用于相同目的。

`Object.keys()` 或 `Object.getOwnPropertyNames()` 不会返回符号值，因此 ES6 新增了一个 `Object.getOwnPropertySymbols()` 方法，允许检索符号类型的对象属性。而你依然可以使用 `Object.defineProperty()` 与 `Object.defineProperties()` 方法对符号类型的属性进行修改。

“知名符号”使用了全局符号常量（例如 `Symbol.hasInstance` ），为常规对象定义了一些功能，而这些功能原先仅限内部使用。这些符号按规范使用 `Symbol.` 的前缀，允许开发者通过多种方式去修改常规对象的行为。