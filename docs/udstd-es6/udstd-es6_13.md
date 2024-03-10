# 第十二章 代理与反射接口

## 第十二章 代理与反射接口

ES5 与 ES6 都推进了 JS 功能的公开。例如， JS 运行环境包含一些不可枚举、不可写入的对象属性，然而在 ES5 之前开发者无法定义他们自己的不可枚举属性或不可写入属性。 ES5 引入了 `Object.defineProperty()` 方法以便开发者在这方面能够像 JS 引擎那样做。

ES6 通过增加内置对象使得开发者能进一步接近 JS 引擎的能力。为了让开发者能够创建内置对象，语言通过**代理**（ **proxy** ）暴露了在对象上的内部工作，代理是一种封装，能够拦截并改变 JS 引擎的底层操作。本章会首先介绍代理所想要处理的问题，并且会讨论如何更有效地创建并使用代理。

*   数组的问题
*   代理与反射是什么？
*   创建一个简单的代理
*   使用 set 陷阱函数验证属性值
*   使用 get 陷阱函数进行对象外形验证
*   使用 has 陷阱函数隐藏属性
*   使用 deleteProperty 陷阱函数避免属性被删除
*   原型代理的陷阱函数
    *   原型代理的陷阱函数如何工作
    *   为何存在两组方法？
*   对象可扩展性的陷阱函数
    *   两个基本范例
    *   可扩展性的重复方法
*   属性描述符的陷阱函数
    *   阻止 Object.defineProperty()
    *   描述符对象的限制
    *   重复的描述符方法
        *   defineProperty() 方法
        *   getOwnPropertyDescriptor() 方法
*   ownKeys 陷阱函数
*   使用 apply 与 construct 陷阱函数的函数代理
    *   验证函数的参数
    *   调用构造器而无须使用 new
    *   重写抽象基础类的构造器
    *   可被调用的类构造器
*   可被撤销的代理
*   解决数组的问题
    *   检测数组的索引
    *   在添加新元素时增加长度属性
    *   在减少长度属性时移除元素
    *   实现 MyArray 类
*   将代理对象作为原型使用
    *   在原型上使用 get 陷阱函数
    *   在原型上使用 set 陷阱函数
    *   在原型上使用 has 陷阱函数
    *   将代理作为类的原型
*   总结

### 数组的问题

在 ES6 之前， JS 的数组对象拥有特定的行为方式，无法被开发者在自定义对象中进行模拟。当你给数组元素赋值时，数组的 `length` 属性会受到影响，同时你也可以通过修改 `length` 属性来变更数组的元素。例如：

```js
let colors = ["red", "green", "blue"];

console.log(colors.length);         // 3

colors[3] = "black";

console.log(colors.length);         // 4
console.log(colors[3]);             // "black"

colors.length = 2;

console.log(colors.length);         // 2
console.log(colors[3]);             // undefined
console.log(colors[2]);             // undefined
console.log(colors[1]);             // "green" 
```

`colors` 开始时有三个元素。把 `"black"` 赋值给 `colors[3]` 会自动将 `length` 增加到 `4` ；而此后设置 `length` 为 `2` 则会移除数组的最末两个元素，从而只保留起始处的两个元素。在 ES5 中开发者无法模拟实现这种行为，但代理的出现改变了这种情况。

> 这种不规范行为就是 ES6 将数组认定为奇异对象的原因。

### 代理与反射是什么？

通过调用 `new Proxy()` ，你可以创建一个代理用来替代另一个对象（被称为**目标**），这个代理对目标对象进行了虚拟，因此该代理与该目标对象表面上可以被当作同一个对象来对待。

代理允许你拦截在目标对象上的底层操作，而这原本是 JS 引擎的内部能力。拦截行为使用了一个能够响应特定操作的函数（被称为**陷阱**）。

被 `Reflect` 对象所代表的反射接口，是给底层操作提供默认行为的方法的集合，这些操作是能够被代理重写的。每个代理陷阱都有一个对应的反射方法，每个方法都与对应的陷阱函数同名，并且接收的参数也与之一致。下表总结了这些行为：

| 代理陷阱 | 被重写的行为 | 默认行为 |
| --- | --- | --- |
| `get` | 读取一个属性的值 | `Reflect.get()` |
| `set` | 写入一个属性 | `Reflect.set()` |
| `has` | `in` 运算符 | `Reflect.has()` |
| `deleteProperty` | `delete` 运算符 | `Reflect.deleteProperty()` |
| `getPrototypeOf` | `Object.getPrototypeOf()` | `Reflect.getPrototypeOf()` |
| `setPrototypeOf` | `Object.setPrototypeOf()` | `Reflect.setPrototypeOf()` |
| `isExtensible` | `Object.isExtensible()` | `Reflect.isExtensible()` |
| `preventExtensions` | `Object.preventExtensions()` | `Reflect.preventExtensions()` |
| `getOwnPropertyDescriptor` | `Object.getOwnPropertyDescriptor()` | `Reflect.getOwnPropertyDescriptor()` |
| `defineProperty` | `Object.defineProperty()` | `Reflect.defineProperty` |
| `ownKeys` | `Object.keys` 、 `Object.getOwnPropertyNames()` 与 `Object.getOwnPropertySymbols()` | `Reflect.ownKeys()` |
| `apply` | 调用一个函数 | `Reflect.apply()` |
| `construct` | 使用 `new` 调用一个函数 | `Reflect.construct()` |

每个陷阱函数都可以重写 JS 对象的一个特定内置行为，允许你拦截并修改它。如果你仍然需要使用原先的内置行为，则可使用对应的反射接口方法。一旦创建了代理，你就能清晰了解代理与反射接口之间的关系，因此我们最好通过一些例子来进行深入研究。

> ES6 的原始草案还有一个名为 `enumerate` 的陷阱函数，其设计意图是更改 `for-in` 与 `Object.keys()` 在对象上进行属性枚举的机制。然而，该陷阱函数在 ECMAScript 7 （也被称为 ECMAScript 2016 ）中被移除了，因为它太难于实现。 `enumerate` 陷阱函数不会再出现在任何 JS 运行环境中，也不会在本章进行介绍。

### 创建一个简单的代理

当你使用 `Proxy` 构造器来创建一个代理时，需要传递两个参数：目标对象以及一个处理器（handler），后者是定义了一个或多个陷阱函数的对象。如果未提供陷阱函数，代理会对所有操作采取默认行为。为了创建一个仅进行传递的代理，你需要使用不包含任何陷阱函数的处理器：

```js
let target = {};

let proxy = new Proxy(target, {});

proxy.name = "proxy";
console.log(proxy.name);        // "proxy"
console.log(target.name);       // "proxy"

target.name = "target";
console.log(proxy.name);        // "target"
console.log(target.name);       // "target" 
```

该例中的 `proxy` 对象将所有操作直接传递给 `target` 对象。当 `proxy.name` 属性被赋值为字符串 `"proxy"` 的时候，`target.name` 属性也同时被创建，代理对象 `proxy` 自身其实并没有存储该属性，它只是简单将值传递给 `target` 对象。同样， `proxy.name` 与 `target.name` 的属性值总是相等，因为它们都指向 `target.name` ，这就意味着：为 `target.name` 设置一个新值会在 `proxy.name` 上反映出相同的改变。当然，缺少陷阱函数的代理没什么用，那么若为其定义一个陷阱函数，又会如何？

### 使用 set 陷阱函数验证属性值

假设你想要创建一个对象，并要求其属性值只能是数值，这就意味着该对象的每个新增属性都要被验证，并且在属性值不为数值类型时应当抛出错误。为此你需要定义 `set` 陷阱函数来重写设置属性值时的默认行为，该陷阱函数能接受四个参数：

1.  `trapTarget` ：将接收属性的对象（即代理的目标对象）；
2.  `key` ：需要写入的属性的键（字符串类型或符号类型）；
3.  `value` ：将被写入属性的值；
4.  `receiver` ：操作发生的对象（通常是代理对象）。

`Reflect.set()` 是 `set` 陷阱函数对应的反射方法，同时也是 `set` 操作的默认行为。 `Reflect.set()` 方法与 `set` 陷阱函数一样，能接受这四个参数，让该方法能在陷阱函数内部被方便使用。该陷阱函数需要在属性被设置完成的情况下返回 `true` ，否则就要返回 `false` ，而 `Reflect.set()` 也会基于操作是否成功而返回相应的结果。

你需要使用 `set` 陷阱函数来拦截传入的 `value` 值，以便对属性值进行验证。这里有个例子：

```js
let target = {
    name: "target"
};

let proxy = new Proxy(target, {
    set(trapTarget, key, value, receiver) {

        // 忽略已有属性，避免影响它们
        if (!trapTarget.hasOwnProperty(key)) {
            if (isNaN(value)) {
                throw new TypeError("Property must be a number.");
            }
        }

        // 添加属性
        return Reflect.set(trapTarget, key, value, receiver);
    }
});

// 添加一个新属性
proxy.count = 1;
console.log(proxy.count);       // 1
console.log(target.count);      // 1

// 你可以为 name 赋一个非数值类型的值，因为该属性已经存在
proxy.name = "proxy";
console.log(proxy.name);        // "proxy"
console.log(target.name);       // "proxy"

// 抛出错误
proxy.anotherName = "proxy"; 
```

这段代码定义了一个代理陷阱，用于对 `target` 对象新增属性的值进行验证。当执行 `proxy.count = 1` 时， `set` 陷阱函数被调用，此时 `trapTarget` 的值等于 `target` 对象， `key` 的值是字符串 `"count"` ， `value` 的值是 `1` ，而 `receiver` 的值是 `proxy` （该参数在本例中并没有被使用）。 `target` 对象上尚不存在名为 `count` 的属性，因此代理将 `value` 参数传递给 `isNaN()` 方法进行验证；如果验证结果是 `NaN` ，表示传入的属性值不是一个数值，需要抛出错误；但由于这段代码将 `count` 参数设置为 `1` ，验证通过，代理使用一致的四个参数去调用 `Reflect.set()` 方法，从而创建了一个新的属性。

当 `proxy.name` 被赋值为字符串时，操作成功完成。这是因为 `target` 对象已经拥有一个 `name` 属性，因此验证时通过调用 `trapTarget.hasOwnProperty()` 会忽略该属性，这就确保允许在该对象的已有属性上使用非数值的属性值。

当 `proxy.anotherName` 被赋值为字符串时，抛出了一个错误。这是因为该对象上并不存在 `anotherName` 属性，因此该属性的值必须被验证，而因为提供的值不是一个数值，验证过程就会抛出错误。

`set` 代理陷阱允许你在写入属性值的时候进行拦截，而 `get` 代理陷阱则允许你在读取属性值的时候进行拦截。

### 使用 get 陷阱函数进行对象外形验证

JS 语言有趣但有时却令人困惑的特性之一，就是读取对象不存在的属性时并不会抛出错误，而会把 `undefined` 当作该属性的值，例如：

```js
let target = {};

console.log(target.name);       // undefined 
```

在多数语言中，试图读取 `target.name` 属性都会抛出错误，因为该属性并不存在；但 JS 语言却会使用 `undefined` 。如果你曾经在大型代码库上进行过工作，那么你可能明白这种行为会导致严重的问题，尤其是当属性名称存在书写错误时。使用代理进行对象外形验证，可以帮你从这个错误中拯救出来。

**对象外形**（ **Object Shape** ）指的是对象已有的属性与方法的集合， JS 引擎使用对象外形来进行代码优化，经常会创建一些类来表示对象。如果你能大胆假设某个对象总是拥有与起始时相同的属性与方法（可以通过 `Object.preventExtensions()` 方法、 `Object.seal()` 方法或 `Object.freeze()` 方法来达到这种效果），那么在访问不存在的属性时抛出错误在这种场合就会非常有用。代理能够让对象外形验证变得轻而易举。

由于该属性验证只须在读取属性时被触发，因此只要使用 `get` 陷阱函数。该陷阱函数会在读取属性时被调用，即使该属性在对象中并不存在，它能接受三个参数：

1.  `trapTarget` ：将会被读取属性的对象（即代理的目标对象）；
2.  `key` ：需要读取的属性的键（字符串类型或符号类型）；
3.  `receiver` ：操作发生的对象（通常是代理对象）。

这些参数借鉴了 `set` 陷阱函数的参数，只有一个明显的不同，也就是没有使用 `value` 参数，因为 `get` 陷阱函数并不需要为属性写入数据。 `Reflect.get()` 方法同样接收这三个参数，并且默认会返回属性的值。

你可以使用 `get` 陷阱函数与 `Reflect.get()` 方法在目标属性不存在时抛出错误，就像这样：

```js
let proxy = new Proxy({}, {
        get(trapTarget, key, receiver) {
            if (!(key in receiver)) {
                throw new TypeError("Property " + key + " doesn't exist.");
            }

            return Reflect.get(trapTarget, key, receiver);
        }
    });

// 添加属性的功能正常
proxy.name = "proxy";
console.log(proxy.name);            // "proxy"

// 读取不存在属性会抛出错误
console.log(proxy.nme);             // 抛出错误 
```

在本例中， `get` 陷阱函数拦截了属性读取操作，它使用 `in` 运算符来判断 `receiver` 对象上是否已存在对应属性。 `receiver` 并没有使用 `trapTarget` ，而是用了 `in` ，这是因为 `receiver` 本身就是拥有一个 `has` 陷阱函数的代理对象（ `has` 陷阱函数会在下一节介绍），在此处使用 `trapTarget` 会跳过 `has` 陷阱函数，并可能给你一个错误的结果。如果要查找的属性不存在，那么就会抛出错误；否则会执行默认的行为。

这段代码允许添加新的属性（例如 `proxy.name` ）以供写入，并且此后可以正常读取该属性的值。最后一行代码有一个拼写错误： `proxy.nme` 应当是 `proxy.name` ，由于 `nme` 属性并不存在，程序抛出了一个错误。

### 使用 has 陷阱函数隐藏属性

`in` 运算符用于判断指定对象中是否存在某个属性，如果对象的属性名与指定的字符串或符号值相匹配，那么 `in` 运算符应当返回 `true` ，无论该属性是对象自身的属性还是其原型的属性。例如：

```js
let target = {
    value: 42;
}

console.log("value" in target);     // true
console.log("toString" in target);  // true 
```

`value` 与 `toString` 均存在于 `object` 对象中，因此 `in` 运算符都会返回 `true` 。其中 `value` 是对象自身的属性，而 `toString` 则是原型属性（从 `Object` 对象上继承而来）。代理允许你使用 `has` 陷阱函数来拦截这个操作，从而在使用 `in` 运算符时返回不同的结果。

`has` 陷阱函数会在使用 `in` 运算符的情况下被调用，并且会被传入两个参数：

1.  `trapTarget` ：需要读取属性的对象（即代理的目标对象）；
2.  `key` ：需要检查的属性的键（字符串类型或符号类型）。

`Reflect.has()` 方法接受与之相同的参数，并向 `in` 运算符返回默认响应结果。使用 `has` 陷阱函数以及 `Reflect.has()` 方法，允许你修改部分属性在接受 `in` 检测时的行为，但保留其他属性的默认行为。例如，假设你只想要隐藏 `value` 属性，你可以这么做：

```js
let target = {
    name: "target",
    value: 42
};

let proxy = new Proxy(target, {
    has(trapTarget, key) {

        if (key === "value") {
            return false;
        } else {
            return Reflect.has(trapTarget, key);
        }
    }
});

console.log("value" in proxy);      // false
console.log("name" in proxy);       // true
console.log("toString" in proxy);   // true 
```

这里的 `proxy` 对象使用了 `has` 陷阱函数，用于检查 `key` 值是否为 `"value"` 。如果是，则返回 `false` ；否则通过调用 `Reflect.has()` 方法来返回默认的结果。这样，虽然 `value` 属性确实存在于目标对象中，但 `in` 运算符却会对该属性返回 `false` ；而其他的属性（ `name` 与 `toString` ）则会正确地返回 `true` 。

### 使用 deleteProperty 陷阱函数避免属性被删除

`delete` 运算符能够从指定对象上删除一个属性，在删除成功时返回 `true` ，否则返回 `false` 。如果试图用 `delete` 运算符去删除一个不可配置的属性，在严格模式下将会抛出错误；而非严格模式下只是单纯返回 `false` 。这里有个例子：

```js
let target = {
    name: "target",
    value: 42
};

Object.defineProperty(target, "name", { configurable: false });

console.log("value" in target);     // true

let result1 = delete target.value;
console.log(result1);               // true

console.log("value" in target);     // false

// 注：下一行代码在严格模式下会抛出错误
let result2 = delete target.name;
console.log(result2);               // false

console.log("name" in target);      // true 
```

这里使用了 `delete` 运算符删除了 `value` 属性，因此在第三行代码的 `console.log()` 调用中，使用 `in` 操作符检测该属性会得到 `false` 。 `name` 属性是不可配置的，因此对其使用 `delete` 操作符只会返回 `false` 而不能删除该属性（如果代码运行在严格模式下，则会抛出错误）。你可以在代理对象中使用 `deleteProperty` 陷阱函数以改变这种行为。

`deleteProperty` 陷阱函数会在使用 `delete` 运算符去删除对象属性时下被调用，并且会被传入两个参数：

1.  `trapTarget` ：需要删除属性的对象（即代理的目标对象）；
2.  `key` ：需要删除的属性的键（字符串类型或符号类型）。

`Reflect.deleteProperty()` 方法也接受这两个参数，并提供了 `deleteProperty` 陷阱函数的默认实现。你可以结合 `Reflect.deleteProperty()` 方法以及 `deleteProperty` 陷阱函数，来修改 `delete` 运算符的行为。例如，能确保 `value` 属性不被删除：

```js
let target = {
    name: "target",
    value: 42
};

let proxy = new Proxy(target, {
    deleteProperty(trapTarget, key) {

        if (key === "value") {
            return false;
        } else {
            return Reflect.deleteProperty(trapTarget, key);
        }
    }
});

// 尝试删除 proxy.value

console.log("value" in proxy);      // true

let result1 = delete proxy.value;
console.log(result1);               // false

console.log("value" in proxy);      // true

// 尝试删除 proxy.name

console.log("name" in proxy);       // true

let result2 = delete proxy.name;
console.log(result2);               // true

console.log("name" in proxy);       // false 
```

这段代码与 `has` 陷阱函数的例子相似，在 `deleteProperty` 陷阱函数中检查 `key` 的值是否为 `"value"` 。如果是，返回 `false` ；否则通过调用 `Reflect.deleteProperty()` 方法来进行默认的操作。 `value` 属性是不能被删除的，因为该操作被 `proxy` 对象拦截；而 `name` 则能如期被删除。这么做允许你在严格模式下保护属性避免其被删除，并且不会抛出错误。

### 原型代理的陷阱函数

第四章介绍了 `Object.setPrototypeOf()` 方法， ES6 引入该方法用于对 ES5 的 `Object.getPrototypeOf()` 方法进行补充。代理允许你通过 `setPrototypeOf` 与 `getPrototypeOf` 陷阱函数来对这两个方法的操作进行拦截。 `Object` 对象上的这两个方法都会调用代理中对应名称的陷阱函数，从而允许你改变这两个方法的行为。

由于存在着两个陷阱函数与原型代理相关联，因此分别有一组方法对应着每个陷阱函数。 `setPrototypeOf` 陷阱函数接受三个参数：

1.  `trapTarget` ：需要设置原型的对象（即代理的目标对象）；
2.  `proto` ：需用被用作原型的对象。

`Object.setPrototypeOf()` 方法与 `Reflect.setPrototypeOf()` 方法会被传入相同的参数。另一方面， `getPrototypeOf` 陷阱函数只接受 `trapTarget` 参数， `Object.getPrototypeOf()` 方法与 `Reflect.getPrototypeOf()` 方法也是如此。

#### 原型代理的陷阱函数如何工作

这些陷阱函数受到一些限制。首先， `getPrototypeOf` 陷阱函数的返回值必须是一个对象或者 `null` ，其他任何类型的返回值都会引发“运行时”错误。对于返回值的检测确保了 `Object.getPrototypeOf()` 会返回预期的结果。类似的， `setPrototypeOf` 必须在操作没有成功的情况下返回 `false` ，这样会让 `Object.setPrototypeOf()` 抛出错误；而若 `setPrototypeOf` 的返回值不是 `false` ，则 `Object.setPrototypeOf()` 就会认为操作已成功。

下面这个例子通过返回 `null` 隐藏了代理对象的原型，并且使得该原型不可被修改：

```js
let target = {};
let proxy = new Proxy(target, {
    getPrototypeOf(trapTarget) {
        return null;
    },
    setPrototypeOf(trapTarget, proto) {
        return false;
    }
});

let targetProto = Object.getPrototypeOf(target);
let proxyProto = Object.getPrototypeOf(proxy);

console.log(targetProto === Object.prototype);      // true
console.log(proxyProto === Object.prototype);       // false
console.log(proxyProto);                            // null

// 成功
Object.setPrototypeOf(target, {});

// 抛出错误
Object.setPrototypeOf(proxy, {}); 
```

这段代码突出了 `target` 对象与 `proxy` 对象的行为差异。使用 `target` 对象作为参数调用 `Object.getPrototypeOf()` 会返回一个对象值；而使用 `proxy` 对象调用该方法则会返回 `null` ，因为 `getPrototypeOf` 陷阱函数被调用了。类似的，使用 `target` 去调用 `Object.setPrototypeOf()` 会成功；而由于 `setPrototypeOf` 陷阱函数的存在，使用 `proxy` 则会引发错误。

如果你想在这两个陷阱函数中使用默认的行为，那么只需调用 `Reflect` 对象上的相应方法。例如，下面的代码为 `getPrototypeOf` 方法与 `setPrototypeOf` 方法实现了默认的行为：

```js
let target = {};
let proxy = new Proxy(target, {
    getPrototypeOf(trapTarget) {
        return Reflect.getPrototypeOf(trapTarget);
    },
    setPrototypeOf(trapTarget, proto) {
        return Reflect.setPrototypeOf(trapTarget, proto);
    }
});

let targetProto = Object.getPrototypeOf(target);
let proxyProto = Object.getPrototypeOf(proxy);

console.log(targetProto === Object.prototype);      // true
console.log(proxyProto === Object.prototype);       // true

// 成功
Object.setPrototypeOf(target, {});

// 同样成功
Object.setPrototypeOf(proxy, {}); 
```

在这个例子中，你可以将 `target` 对象与 `proxy` 对象互换使用，因为 `getPrototypeOf` 与 `setPrototypeOf` 陷阱函数只是直接传递参数去调用默认的实现。需要特别注意的是，本例使用了 `Reflect.getPrototypeOf()` 方法与 `Reflect.setPrototypeOf()` 方法，而没有使用 `Object` 对象上的同名方法，因为这些方法存在重要差别。

#### 为何存在两组方法？

关于 `Reflect.getPrototypeOf()` 与 `Reflect.setPrototypeOf()` ，令人困惑的是它们看起来与 `Object.getPrototypeOf()` 与 `Object.setPrototypeOf()` 非常相似。然而虽然两组方法分别进行着相似的操作，它们之间仍然存在显著差异。

首先， `Object.getPrototypeOf()` 与 `Object.setPrototypeOf()` 属于高级操作，从产生之初便已提供给开发者使用；而 `Reflect.getPrototypeOf()` 与 `Reflect.setPrototypeOf()` 属于底层操作，允许开发者访问 `[[GetPrototypeOf]]` 与 `[[SetPrototypeOf]]` 这两个原先仅供语言内部使用的操作。 `Reflect.getPrototypeOf()` 方法是对内部的 `[[GetPrototypeOf]]` 操作的封装（并附加了一些输入验证），而 `Reflect.setPrototypeOf()` 方法与 `[[SetPrototypeOf]]` 操作之间也存在类似的关系。虽然 `Object` 对象上的同名方法也调用了 `[[GetPrototypeOf]]` 与 `[[SetPrototypeOf]]` ，但它们在调用这两个操作之前添加了一些步骤、并检查返回值，以决定如何行动。

`Reflect.getPrototypeOf()` 方法在接收到的参数不是一个对象时会抛出错误，而 `Object.getPrototypeOf()` 则会在操作之前先将参数值转换为一个对象。如果你分别传入一个数值给这两个方法，会得到截然不同的结果：

```js
let result1 = Object.getPrototypeOf(1);
console.log(result1 === Number.prototype);  // true

// 抛出错误
Reflect.getPrototypeOf(1); 
```

`Object.getPrototypeOf()` 方法能够为数值 `1` 找到一个原型，因为它首先会将数值 `1` 转换为一个 `Number` 对象，这样就可以使用 `Number` 对象的原型。而 `Reflect.getPrototypeOf()` 方法并不会转换这个参数，由于数值 `1` 不是一个对象，因此该方法调用会导致一个错误。

`Reflect.setPrototypeOf()` 方法与 `Object.setPrototypeOf()` 方法还有几点差异。首先， `Reflect.setPrototypeOf()` 方法返回一个布尔值用于表示操作是否已成功，成功时返回 `true` ，而失败时返回 `false` ；但若 `Object.setPrototypeOf()` 方法的操作失败，它会抛出错误。

在“原型代理的陷阱函数如何工作”那个小节的第一个例子中，当 `setPrototypeOf` 代理陷阱返回 `false` 时，它导致 `Object.setPrototypeOf()` 方法抛出了错误。此外， `Object.setPrototypeOf()` 方法会将传入的第一个参数作为自身的返回值，因此并不适合用来实现 `setPrototypeOf` 代理陷阱的默认行为。下面的代码演示了这些区别：

```js
let target1 = {};
let result1 = Object.setPrototypeOf(target1, {});
console.log(result1 === target1);                   // true

let target2 = {};
let result2 = Reflect.setPrototypeOf(target2, {});
console.log(result2 === target2);                   // false
console.log(result2);                               // true 
```

在本例中， `Object.setPrototypeOf()` 方法将 `target1` 对象作为返回值，而 `Reflect.setPrototypeOf()` 方法则返回了 `true` 。这个微妙的差异非常重要，虽然 `Object` 对象与 `Reflect` 对象貌似存在重复的方法，但在代理陷阱内却必须使用 `Reflect` 对象上的方法。

> 在使用代理时，这两组方法都会调用 `getPrototypeOf` 与 `setPrototypeOf` 陷阱函数。

### 对象可扩展性的陷阱函数

ES5 通过 `Object.preventExtensions()` 与 `Object.isExtensible()` 方法给对象增加了可扩展性。而 ES6 则通过 `preventExtensions` 与 `isExtensible` 陷阱函数允许代理拦截对于底层对象的方法调用。这两个陷阱函数都接受名为 `trapTarget` 的单个参数，此参数代表方法在哪个对象上被调用。 `isExtensible` 陷阱函数必须返回一个布尔值用于表明目标对象是否可被扩展，而 `preventExtensions` 陷阱函数也需要返回一个布尔值，用于表明操作是否已成功。

同时也存在 `Reflect.preventExtensions()` 与 `Reflect.isExtensible()` 方法，用于实现默认的行为。这两个方法都返回布尔值，因此它们可以在对应的陷阱函数内直接使用。

#### 两个基本范例

为了弄懂对象可扩展性的陷阱函数如何运作，可研究如下代码，该代码实现了 `isExtensible` 与 `preventExtensions` 陷阱函数的默认行为。

```js
let target = {};
let proxy = new Proxy(target, {
    isExtensible(trapTarget) {
        return Reflect.isExtensible(trapTarget);
    },
    preventExtensions(trapTarget) {
        return Reflect.preventExtensions(trapTarget);
    }
});

console.log(Object.isExtensible(target));       // true
console.log(Object.isExtensible(proxy));        // true

Object.preventExtensions(proxy);

console.log(Object.isExtensible(target));       // false
console.log(Object.isExtensible(proxy));        // false 
```

这个例子将 `Object.preventExtensions()` 与 `Object.isExtensible()` 方法直接从 `proxy` 对象传递到 `target` 对象。当然，你也可以自行修改这种行为。例如，如果不想让代理上的 `Object.preventExtensions()` 操作成功，你可以强制 `preventExtensions` 陷阱函数返回 `false` 。

```js
let target = {};
let proxy = new Proxy(target, {
    isExtensible(trapTarget) {
        return Reflect.isExtensible(trapTarget);
    },
    preventExtensions(trapTarget) {
        return false
    }
});

console.log(Object.isExtensible(target));       // true
console.log(Object.isExtensible(proxy));        // true

Object.preventExtensions(proxy);

console.log(Object.isExtensible(target));       // true
console.log(Object.isExtensible(proxy));        // true 
```

这段代码中，对于 `Object.preventExtensions(proxy)` 的调用被有效地忽略了。因为 `preventExtensions` 陷阱函数返回了 `false`，因此该操作并不会被传递到 `target` 对象上，于是后面的 `Object.isExtensible()` 仍然会返回 `true`。

> 译注：此代码在 FireFox 和 Edge 中能够正常执行，但在 Chrome 中却会在 `Object.preventExtensions(proxy)` 这一行抛出错误。

#### 可扩展性的重复方法

你可能已经注意到：在可扩展性方面， `Object` 对象与 `Reflect` 对象再次出现了重复的方法。不过它们之间的差异相对要小得多： `Object.isExtensible()` 方法与 `Reflect.isExtensible()` 方法几乎一样，只在接收到的参数不是一个对象时才有例外。此时 `Object.isExtensible()` 总是会返回 `false` ，而 `Reflect.isExtensible()` 则会抛出一个错误。这里有个示例：

```js
let result1 = Object.isExtensible(2);
console.log(result1);                       // false

// 抛出错误
let result2 = Reflect.isExtensible(2); 
```

这种区别与 `Object.getPrototypeOf()` 方法和 `Reflect.getPrototypeOf()` 方法之间的区别相似，底层功能的方法与对应的高层方法相比，会进行更严格的错误检查。

`Object.preventExtensions()` 方法与 `Reflect.preventExtensions()` 方法也是非常相似的。 `Object.preventExtensions()` 方法总是将传递给它的参数值作为自身的返回值，即使该参数不是一个对象；而另一方面 `Reflect.preventExtensions()` 方法则会在参数不是对象时抛出错误。当参数确实是一个对象时， `Reflect.preventExtensions()` 会在操作成功时返回 `true` ，否则返回 `false` 。例如：

```js
let result1 = Object.preventExtensions(2);
console.log(result1);                               // 2

let target = {};
let result2 = Reflect.preventExtensions(target);
console.log(result2);                               // true

// 抛出错误
let result3 = Reflect.preventExtensions(2); 
```

此代码中的 `Object.preventExtensions()` 将传递给它的参数 `2` 作为返回值，尽管 `2` 并不是一个对象。 而 `Reflect.preventExtensions()` 方法在接收一个对象作为参数时返回了 `true` ，但在接收 `2` 时抛出了错误。

### 属性描述符的陷阱函数

ES5 最重要的特征之一就是引入了 `Object.defineProperty()` 方法用于定义属性的特性。在 JS 之前的版本中，没有方法可以定义一个访问器属性，也不能让属性变成只读或是不可枚举。而这些特性都能够利用 `Object.defineProperty()` 方法来实现，并且你还可以利用 `Object.getOwnPropertyDescriptor()` 方法来检索这些特性。

代理允许你使用 `defineProperty` 与 `getOwnPropertyDescriptor` 陷阱函数，来分别拦截对于 `Object.defineProperty()` 与 `Object.getOwnPropertyDescriptor()` 的调用。 `defineProperty` 陷阱函数接受下列三个参数：

1.  `trapTarget` ：需要被定义属性的对象（即代理的目标对象）；
2.  `key` ：属性的键（字符串类型或符号类型）；
3.  `descriptor` ：为该属性准备的描述符对象。

`defineProperty` 陷阱函数要求你在操作成功时返回 `true` ，否则返回 `false` 。 `getOwnPropertyDescriptor` 陷阱函数则只接受 `trapTarget` 与 `key` 这两个参数，并会返回对应的描述符。 `Reflect.defineProperty()` 与 `Reflect.getOwnPropertyDescriptor()` 方法作为上述陷阱函数的对应方法，接受与之相同的参数。这里有个例子，实现了每个陷阱函数的默认行为：

```js
let proxy = new Proxy({}, {
    defineProperty(trapTarget, key, descriptor) {
        return Reflect.defineProperty(trapTarget, key, descriptor);
    },
    getOwnPropertyDescriptor(trapTarget, key) {
        return Reflect.getOwnPropertyDescriptor(trapTarget, key);
    }
});

Object.defineProperty(proxy, "name", {
    value: "proxy"
});

console.log(proxy.name);            // "proxy"

let descriptor = Object.getOwnPropertyDescriptor(proxy, "name");

console.log(descriptor.value);      // "proxy" 
```

这段代码使用了 `Object.defineProperty()` 方法在代理对象上定义了名为 `"name"` 的属性，该属性的描述符可以使用 `Object.getOwnPropertyDescriptor()` 方法进行检索。

#### 阻止 Object.defineProperty()

`defineProperty` 陷阱函数要求你返回一个布尔值用于表示操作是否已成功。当它返回 `true` 时， `Object.defineProperty()` 会正常执行；而如果它返回了 `false` ，则 `Object.defineProperty()` 会抛出错误。 你可以使用该功能来限制哪些属性可以被 `Object.defineProperty()` 方法定义。例如，如果想阻止定义符号类型的属性，你可以检查传入的键是否为字符串，若不是则返回 `false` ，就像这样：

```js
let proxy = new Proxy({}, {
    defineProperty(trapTarget, key, descriptor) {

        if (typeof key === "symbol") {
            return false;
        }

        return Reflect.defineProperty(trapTarget, key, descriptor);
    }
});

Object.defineProperty(proxy, "name", {
    value: "proxy"
});

console.log(proxy.name);                    // "proxy"

let nameSymbol = Symbol("name");

// 抛出错误
Object.defineProperty(proxy, nameSymbol, {
    value: "proxy"
}); 
```

当 `key` 是一个符号时， `defineProperty` 代理陷阱会返回 `false` ，而其他情况下则会保持默认的行为。当使用字符串 `"name"` 作为键去调用 `Object.defineProperty()` 时，该方法能够成功执行；然而当使用符号变量 `nameSymbol` 去调用 `Object.defineProperty()` 的时候， `defineProperty` 陷阱函数返回了 `false`，导致程序抛出了错误。

> 你可以让陷阱函数返回 `true` ，同时不去调用 `Reflect.defineProperty()` 方法，这样 `Object.defineProperty()` 就会静默失败，如此便可在未实际去定义属性的情况下抑制运行错误。

#### 描述符对象的限制

为了确保 `Object.defineProperty()` 与 `Object.getOwnPropertyDescriptor()` 方法的行为一致，传递给 `defineProperty` 陷阱函数的描述符对象必须是正规的。出于同一原因， `getOwnPropertyDescriptor`陷阱函数返回的对象也始终需要被验证。

任意对象都能作为 `Object.defineProperty()` 方法的第三个参数；然而传递给 `defineProperty` 陷阱函数的描述符对象参数，则只有 `enumerable` 、 `configurable` 、 `value` 、 `writable` 、 `get` 与 `set` 这些属性是被许可的。例如：

```js
let proxy = new Proxy({}, {
    defineProperty(trapTarget, key, descriptor) {
        console.log(descriptor.value);              // "proxy"
        console.log(descriptor.name);               // undefined

        return Reflect.defineProperty(trapTarget, key, descriptor);
    }
});

Object.defineProperty(proxy, "name", {
    value: "proxy",
    name: "custom"
}); 
```

此代码中调用 `Object.defineProperty()` 时，在第三个参数上使用了一个非标准的 `name` 属性。当 `defineProperty` 陷阱函数被调用时， `descriptor` 对象不会拥有 `name` 属性，却拥有一个 `value` 属性。这是因为 `descriptor` 对象实际上并不是原先传递给 `Object.defineProperty()` 方法的第三个参数，而是一个新的对象，其中只包含了被许可的属性（因此 `name` 属性被丢弃了）。 `Reflect.defineProperty()` 方法同样也会忽略描述符上的非标准属性。

`getOwnPropertyDescriptor` 陷阱函数有一个微小差异，要求返回值必须是 `null` 、 `undefined` ，或者是一个对象。如果返回值是一个对象，则只允许该对象拥有 `enumerable` 、 `configurable` 、 `value` 、 `writable` 、 `get` 或 `set` 这些自有属性。如果你返回的对象包含了不被许可的自有属性，则程序会抛出错误，就像下面演示的这样：

```js
let proxy = new Proxy({}, {
    getOwnPropertyDescriptor(trapTarget, key) {
        return {
            name: "proxy"
        };
    }
});

// 抛出错误
let descriptor = Object.getOwnPropertyDescriptor(proxy, "name"); 
```

`name` 属性在属性描述符中是不被许可的，因此当 `Object.getOwnPropertyDescriptor()` 被调用时， `getOwnPropertyDescriptor` 的返回值会触发一个错误。这个限制保证了 `Object.getOwnPropertyDescriptor()` 的返回值总是拥有可信任的结构，无论是否使用了代理。

#### 重复的描述符方法

ES6 再次出现了令人困惑的相似方法， `Object.defineProperty()` 和 `Object.getOwnPropertyDescriptor()` 方法貌似分别与 `Reflect.defineProperty()` 和 `Reflect.getOwnPropertyDescriptor()` 方法相同。正如本章之前讨论过的那些配套方法一样，这些方法也存在一些微小但重要的差异。

##### defineProperty() 方法

`Object.defineProperty()` 方法与 `Reflect.defineProperty()` 方法几乎一模一样，只是返回值有区别。前者返回调用它时的第一个参数，而后者在操作成功时返回 `true` 、失败时返回 `false` 。例如：

```js
let target = {};

let result1 = Object.defineProperty(target, "name", { value: "target "});

console.log(target === result1);        // true

let result2 = Reflect.defineProperty(target, "name", { value: "reflect" });

console.log(result2);                   // true 
```

使用 `target` 对象去调用 `Object.defineProperty()` 方法，返回值也是 `target` 。而同样使用 `target` 对象去调用 `Reflect.defineProperty()` ，返回值却是 `true` ，表示操作已经成功。由于 `defineProperty` 代理陷阱需要一个布尔值作为返回值，因此最好在必要时使用 `Reflect.defineProperty()` 来实现默认的行为。

##### getOwnPropertyDescriptor() 方法

`Object.getOwnPropertyDescriptor()` 方法会在接收的第一个参数是一个基本类型值时，将该参数转换为一个对象。另一方面， `Reflect.getOwnPropertyDescriptor()` 方法则会在第一个参数是基本类型值的时候抛出错误。下面这个例子展示了二者的特性：

```js
let descriptor1 = Object.getOwnPropertyDescriptor(2, "name");
console.log(descriptor1);       // undefined

// 抛出错误
let descriptor2 = Reflect.getOwnPropertyDescriptor(2, "name"); 
```

此代码中的 `Object.getOwnPropertyDescriptor()` 方法返回了 `undefined` ，因为它将 `2` 转换为一个对象，转换后的对象并不包含 `name` 属性，而返回 `undefined` 是指定属性名在目标对象中不存在时的标准行为。然而当 `Reflect.getOwnPropertyDescriptor()` 被调用时，立刻抛出了一个错误，因为该方法不接受基本类型值作为它的第一个参数。

### ownKeys 陷阱函数

`ownKeys` 代理陷阱拦截了内部方法 `[[OwnPropertyKeys]]` ，并允许你返回一个数组用于重写该行为。返回的这个数组会被用于四个方法： `Object.keys()` 方法、 `Object.getOwnPropertyNames()` 方法、 `Object.getOwnPropertySymbols()` 方法与 `Object.assign()` 方法，其中 `Object.assign()` 方法会使用该数组来决定哪些属性会被复制。

`ownKeys` 陷阱函数的默认行为由 `Reflect.ownKeys()` 方法实现，会返回一个由全部自有属性的键构成的数组，无论键的类型是字符串还是符号。 `Object.getOwnProperyNames()` 方法与 `Object.keys()` 方法会将符号值从该数组中过滤出去；相反， `Object.getOwnPropertySymbols()` 会将字符串值过滤掉；而 `Object.assign()` 方法会使用数组中所有的字符串值与符号值。

`ownKeys` 陷阱函数接受单个参数，即目标对象，同时必须返回一个数组或者一个类数组对象，不合要求的返回值会导致错误。你可以使用 `ownKeys` 陷阱函数去过滤特定的属性，以避免这些属性被 `Object.keys()` 方法、 `Object.getOwnPropertyNames()` 方法、 `Object.getOwnPropertySymbols()` 方法或 `Object.assign()` 方法使用。假设你不想在结果中包含任何以下划线打头的属性（在 JS 的编码惯例中，这代表该字段是私有的），那么可以使用 `ownKeys` 陷阱函数来将它们过滤掉，就像下面这样：

```js
let proxy = new Proxy({}, {
    ownKeys(trapTarget) {
        return Reflect.ownKeys(trapTarget).filter(key => {
            return typeof key !== "string" || key[0] !== "_";
        });
    }
});

let nameSymbol = Symbol("name");

proxy.name = "proxy";
proxy._name = "private";
proxy[nameSymbol] = "symbol";

let names = Object.getOwnPropertyNames(proxy),
    keys = Object.keys(proxy);
    symbols = Object.getOwnPropertySymbols(proxy);

console.log(names.length);      // 1
console.log(names[0]);          // "proxy"

console.log(keys.length);      // 1
console.log(keys[0]);          // "proxy"

console.log(symbols.length);    // 1
console.log(symbols[0]);        // "Symbol(name)" 
```

这个例子使用了一个 `ownKeys` 陷阱函数，首先调用了 `Reflect.ownKeys()` 方法来获取目标对象的键列表；接下来， `filter()` 方法被用于将所有下划线打头的字符串类型的键过滤出去；这之后向 `proxy` 对象添加了三个属性： `name` 、 `_name` 与 `nameSymbol`。当 `proxy` 对象上的 `Object.getOwnPropertyNames()` 方法与 `Object.keys()` 方法被调用时，只获得了 `name` 属性；类似的， `Object.getOwnPropertySymbols()` 方法被调用时只获得了 `nameSymbol` 属性；而 `_name` 属性则始终没有出现在结果里，因为它被过滤了。

虽然 `ownKeys` 代理陷阱允许你修改少数操作所返回的键值，但它不能影响一些常用操作，例如 `for-of` 循环以及 `Object.keys()` 方法，这些都是使用代理所无法改变的。

> `ownKeys` 陷阱函数也能影响 `for-in` 循环，因为这种循环调用了陷阱函数来决定哪些值能够被用在循环内。

### 使用 apply 与 construct 陷阱函数的函数代理

在所有的代理陷阱中，只有 `apply` 与 `construct` 要求代理目标对象必须是一个函数。回忆一下第三章的内容，函数拥有两个内部方法： `[[Call]]` 与 `[[Construct]]` ，前者会在函数被直接调用时执行，而后者会在函数被使用 `new` 运算符调用时执行。 `apply` 与 `construct` 陷阱函数对应着这两个内部方法，并允许你对其进行重写。当不使用 `new` 去调用一个函数时， `apply` 陷阱函数会接收到下列三个参数（ `Reflect.apply()` 也会接收这些参数）：

1.  `trapTarget` ：被执行的函数（即代理的目标对象）；
2.  `thisArg` ：调用过程中函数内部的 `this` 值；
3.  `argumentsList` ：被传递给函数的参数数组。

当使用 `new` 去执行函数时， `construct` 陷阱函数会被调用并接收到下列两个参数：

1.  `trapTarget` ：被执行的函数（即代理的目标对象）；
2.  `argumentsList` ：被传递给函数的参数数组。

`Reflect.construct()` 方法同样会接收到这两个参数，还会收到可选的第三参数 `newTarget` ，如果提供了此参数，则它就指定了函数内部的 `new.target` 值。

`apply` 与 `construct` 陷阱函数结合起来就完全控制了任意的代理目标对象函数的行为。为了模拟函数的默认行为，你可以这么做：

```js
let target = function() { return 42 },
    proxy = new Proxy(target, {
        apply: function(trapTarget, thisArg, argumentList) {
            return Reflect.apply(trapTarget, thisArg, argumentList);
        },
        construct: function(trapTarget, argumentList) {
            return Reflect.construct(trapTarget, argumentList);
        }
    });

// 使用了函数的代理，其目标对象会被视为函数
console.log(typeof proxy);                  // "function"

console.log(proxy());                       // 42

var instance = new proxy();
console.log(instance instanceof proxy);     // true
console.log(instance instanceof target);    // true 
```

本例中的函数会返回一个数值 `42` 。该函数的代理使用了 `apply` 与 `construct` 陷阱函数来将对应行为分别委托给 `Reflect.apply()` 与 `Reflect.construct()` 方法。最终结果是代理函数就像目标函数一样工作，包括使用 `typeof` 会将其检测为函数，并且使用 `new` 运算符调用会产生一个实例对象 `instance` 。 `instance` 对象会被同时判定为 `proxy` 与 `target` 对象的实例，是因为 `instanceof` 运算符使用了原型链来进行推断，而原型链查找并没有受到这个代理的影响，因此 `proxy` 对象与 `target` 对象对于 JS 引擎来说就有同一个原型。

#### 验证函数的参数

`apply` 与 `construct` 陷阱函数在函数的执行方式上开启了很多的可能性。例如，假设你想要保证所有参数都是某个特定类型的，可使用 `apply` 陷阱函数来进行验证:

```js
// 将所有参数相加
function sum(...values) {
    return values.reduce((previous, current) => previous + current, 0);
}

let sumProxy = new Proxy(sum, {
        apply: function(trapTarget, thisArg, argumentList) {

            argumentList.forEach((arg) => {
                if (typeof arg !== "number") {
                    throw new TypeError("All arguments must be numbers.");
                }
            });

            return Reflect.apply(trapTarget, thisArg, argumentList);
        },
        construct: function(trapTarget, argumentList) {
            throw new TypeError("This function can't be called with new.");
        }
    });

console.log(sumProxy(1, 2, 3, 4));          // 10

// 抛出错误
console.log(sumProxy(1, "2", 3, 4));

// 同样抛出错误
let result = new sumProxy(); 
```

此例使用了 `apply` 陷阱函数来确保所有的参数都是数值。 `sum()` 函数会将所有传递进来的参数值相加，如果传入参数的值不是数值类型，该函数仍然会尝试加法操作，这样可能会导致意外的结果。此代码通过将 `sum()` 函数封装在 `sumProxy()` 代理中，在函数运行之前拦截了函数调用，以保证每个参数都是数值。出于安全的考虑，这段代码使用 `construct` 陷阱抛出错误，以确保该函数不会被使用 `new` 运算符调用。

相反的，你也可以限制函数必须使用 `new` 运算符调用，同时确保它的参数都是数值：

```js
function Numbers(...values) {
    this.values = values;
}

let NumbersProxy = new Proxy(Numbers, {

        apply: function(trapTarget, thisArg, argumentList) {
            throw new TypeError("This function must be called with new.");
        },

        construct: function(trapTarget, argumentList) {
            argumentList.forEach((arg) => {
                if (typeof arg !== "number") {
                    throw new TypeError("All arguments must be numbers.");
                }
            });

            return Reflect.construct(trapTarget, argumentList);
        }
    });

let instance = new NumbersProxy(1, 2, 3, 4);
console.log(instance.values);               // [1,2,3,4]

// 抛出错误
NumbersProxy(1, 2, 3, 4); 
```

此代码中的 `apply` 陷阱函数会抛出错误，而 `construct` 陷阱函数则使用了 `Reflect.construct()` 方法来验证输入并返回一个新的实例。当然，你也可以不必使用代理，而是用 `new.target` 来完成相同的功能。

#### 调用构造器而无须使用 new

第三章曾介绍了 `new.target` 元属性，在使用 `new` 运算符调用函数时，这个属性就是对该函数的一个引用。这意味着你可以使用 `new.target` 来判断函数被调用时是否使用了 `new` ，就像这样：

```js
function Numbers(...values) {

    if (typeof new.target === "undefined") {
        throw new TypeError("This function must be called with new.");
    }

    this.values = values;
}

let instance = new Numbers(1, 2, 3, 4);
console.log(instance.values);               // [1,2,3,4]

// 抛出错误
Numbers(1, 2, 3, 4); 
```

这个例子在不使用 `new` 来调用 `Numbers` 函数的情况下抛出了错误，与“验证函数的参数”那个小节的例子效果一致，但并没有使用代理。相对于使用代理，这种写法更简单，并且若只想阻止不使用 `new` 来调用函数的行为，这种写法也更胜一筹。然而有时你所要修改其行为的函数是你所无法控制的，此时使用代理就有意义了。

假设 `Numbers` 函数是硬编码的，无法被修改，已知该代码依赖于 `new.target` ，而你想要在调用函数时避免这个检查。在“必须使用 `new` ”这一限制已经确定的情况下，你可以使用 `apply` 陷阱函数来规避它：

```js
function Numbers(...values) {

    if (typeof new.target === "undefined") {
        throw new TypeError("This function must be called with new.");
    }

    this.values = values;
}

let NumbersProxy = new Proxy(Numbers, {
        apply: function(trapTarget, thisArg, argumentsList) {
            return Reflect.construct(trapTarget, argumentsList);
        }
    });

let instance = NumbersProxy(1, 2, 3, 4);
console.log(instance.values);               // [1,2,3,4] 
```

`NumbersProxy` 函数允许你调用 `Numbers` 而无须使用 `new` ，并且让这种调用的效果与使用了 `new` 的情况保持一致。为此， `apply` 陷阱函数使用传给自身的参数去对 `Reflect.construct()` 方法进行了调用，于是 `Numbers` 内部的 `new.target` 就被设置为 `Numbers` ，从而避免抛出错误。尽管这只是修改 `new.target` 的一个简单例子，但你还可以做得更加直接。

#### 重写抽象基础类的构造器

你可以进一步指定 `Reflect.construct()` 的第三个参数，用于给 `new.target` 赋值。当函数把 `new.target` 与已知值进行比较的时候，例如在创建一个抽象基础类的构造器的场合下（参阅第九章），这么做会很有帮助。在抽象基础类的构造器中， `new.target` 被要求不能是构造器自身，正如这个例子：

```js
class AbstractNumbers {

    constructor(...values) {
        if (new.target === AbstractNumbers) {
            throw new TypeError("This function must be inherited from.");
        }

        this.values = values;
    }
}

class Numbers extends AbstractNumbers {}

let instance = new Numbers(1, 2, 3, 4);
console.log(instance.values);           // [1,2,3,4]

// 抛出错误
new AbstractNumbers(1, 2, 3, 4); 
```

当 `new AbstractNumbers()` 被调用时， `new.target` 等于 `AbstractNumbers` ，从而抛出了错误；而调用 `new Numbers()` 能正常工作，因为此时 `new.target` 等于 `Numbers` 。你可以使用代理手动指定 `new.target` 从而绕过这个限制：

```js
class AbstractNumbers {

    constructor(...values) {
        if (new.target === AbstractNumbers) {
            throw new TypeError("This function must be inherited from.");
        }

        this.values = values;
    }
}

let AbstractNumbersProxy = new Proxy(AbstractNumbers, {
        construct: function(trapTarget, argumentList) {
            return Reflect.construct(trapTarget, argumentList, function() {});
        }
    });

let instance = new AbstractNumbersProxy(1, 2, 3, 4);
console.log(instance.values);               // [1,2,3,4] 
```

`AbstractNumbersProxy` 使用 `construct` 陷阱函数拦截了对于 `new AbstractNumbersProxy()` 方法的调用，这样陷阱函数就将一个空函数作为第三个参数传递给了 `Reflect.construct()` 方法，让这个空函数成为构造器内部的 `new.target` 。由于此时 `new.target` 的值并不等于 `AbstractNumbers` ，就不会抛出错误，构造器可以执行完成。

#### 可被调用的类构造器

第九章说明了构造器必须始终使用 `new` 来调用，原因是类构造器的内部方法 `[[Call]]` 被明确要求抛出错误。然而代理可以拦截对于 `[[Call]]` 方法的调用，意味着你可以借助代理有效创建一个可被调用的类构造器。例如，如果想让类构造器在缺少 `new` 的情况下能够工作，你可以使用 `apply` 陷阱函数来创建一个新实例。这里有个例子：

```js
class Person {
    constructor(name) {
        this.name = name;
    }
}

let PersonProxy = new Proxy(Person, {
        apply: function(trapTarget, thisArg, argumentList) {
            return new trapTarget(...argumentList);
        }
    });

let me = PersonProxy("Nicholas");
console.log(me.name);                   // "Nicholas"
console.log(me instanceof Person);      // true
console.log(me instanceof PersonProxy); // true 
```

`PersonProxy` 对象是 `Person` 类构造器的一个代理。类构造器实际上也是函数，因此在使用代理时它的行为就像函数一样。 `apply` 陷阱函数重写了默认的行为，返回 `trapTarget` （这里等于 `Person` ）的一个实例，此代码使用 `trapTarget` 以保证通用性，避免了手动指定特定的类。此处还使用了扩展运算符，将 `argumentList` 展开并传递给 `trapTarget` 方法。在没有使用 `new` 的情况下调用 `PersonProxy()` ，获得了 `Person` 的一个新实例；而若你试图不使用 `new` 去调用 `Person()` ，构造器仍然会抛出错误。创建一个可被调用的类构造器，是只有使用代理才能做到的。

### 可被撤销的代理

在被创建之后，代理通常就不能再从目标对象上被解绑。本章之前的例子都使用了不可被撤销的代理，但有的情况下你可能想撤销一个代理以便让它不能再被使用。当你想通过公共接口向外提供一个安全的对象，并且要求要随时都能切断对某些功能的访问，这种情况下可被撤销的代理就会非常有用。

你可以使用 `Proxy.revocable()` 方法来创建一个可被撤销的代理，该方法接受的参数与 `Proxy` 构造器的相同：一个目标对象、一个代理处理器，而返回值是包含下列属性的一个对象：

1.  `proxy` ：可被撤销的代理对象；
2.  `revoke` ：用于撤销代理的函数。

当 `revoke()` 函数被调用后，就不能再对该 `proxy` 对象进行更多操作，任何与该代理对象交互的意图都会触发代理的陷阱函数，从而抛出一个错误。例如：

```js
let target = {
    name: "target"
};

let { proxy, revoke } = Proxy.revocable(target, {});

console.log(proxy.name);        // "target"

revoke();

// 抛出错误
console.log(proxy.name); 
```

这个例子创建了一个可被撤销的代理，它对 `Proxy.revocable()` 方法返回的对象进行了解构赋值，把同名属性的值赋给了 `proxy` 与 `revoke` 变量。此时 `proxy` 对象可以像一个不可被撤销的代理那样被使用，于是 `proxy.name` 属性的值就是 `"target"` ，因为它直接传递了 `target.name` 的值。然而一旦 `revoke()` 函数被调用， `proxy` 就不再是一个函数，之后试图访问 `proxy.name` 会抛出错误，同时其他对于 `proxy` 对象的操作也都会触发陷阱函数。

### 解决数组的问题

在本章开始时，我解释了为何在 ES6 之前开发者无法准确模拟 JS 数组的行为。而代理与反射接口则允许你创建这样一种对象：在属性被添加或删除时，它的行为与内置数组类型的行为相同。为了刷新你的记忆，这里有个例子展示了代理所要模拟的行为：

```js
let colors = ["red", "green", "blue"];

console.log(colors.length);         // 3

colors[3] = "black";

console.log(colors.length);         // 4
console.log(colors[3]);             // "black"

colors.length = 2;

console.log(colors.length);         // 2
console.log(colors[3]);             // undefined
console.log(colors[2]);             // undefined
console.log(colors[1]);             // "green" 
```

这个例子可以体现出两个特别重要的行为特性：

1.  当 `colors[3]` 被赋值时， `length` 属性被自动增加到 4 ；
2.  当 `length` 属性被设置为 2 时，数组的最后两个元素被自动移除了。

当想要重现内置数组的工作方式时，仅需模拟这两个行为即可。接下来的几小节将会介绍如何正确地将一个对象模拟为数组。

#### 检测数组的索引

必须始终牢记：对于数组来说，为整数属性赋值是一种特殊情况，不同于对非整数的键的处理。在如何判断一个属性键是否为数组的索引方面， ES6 规范给出了指南：

> 对于名为 `P` 的一个字符串属性名称来说，当且仅当 `ToString(ToUint32(P))` 等于 `P` 、并且 `ToUint32(P)` 不等于 2³² - 1 时，它才能被用作数组的索引。

这个操作可以用下述的 JS 代码来实现：

```js
function toUint32(value) {
    return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32);
}

function isArrayIndex(key) {
    let numericKey = toUint32(key);
    return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1);
} 
```

`toUint32()` 函数使用规范中描述的算法，将给定值转换为一个无符号的 32 位整数。 `isArrayIndex()` 函数首先将键值转换为一个 uint32 数，并执行了比较操作来判断该键是否能够作为数组的索引。借助这两个工具函数，你就可以开始实现一个对象来模拟内置数组。

#### 在添加新元素时增加长度属性

你可能已经注意到：数组上述两个特殊行为都依赖于对属性的赋值，这就意味着你只需要使用 `set` 代理陷阱来达成这两个行为。首先，下面的例子实现了第一个行为，即：当一个大于 `length - 1` 的数组索引被使用时， `length` 属性需要被增加。

```js
function toUint32(value) {
    return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32);
}

function isArrayIndex(key) {
    let numericKey = toUint32(key);
    return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1);
}

function createMyArray(length=0) {
    return new Proxy({ length }, {
        set(trapTarget, key, value) {

            let currentLength = Reflect.get(trapTarget, "length");

            // 特殊情况
            if (isArrayIndex(key)) {
                let numericKey = Number(key);

                if (numericKey >= currentLength) {
                    Reflect.set(trapTarget, "length", numericKey + 1);
                }
            }

            // 无论键的类型是什么，都要执行这行代码
            return Reflect.set(trapTarget, key, value);
        }
    });
}

let colors = createMyArray(3);
console.log(colors.length);         // 3

colors[0] = "red";
colors[1] = "green";
colors[2] = "blue";

console.log(colors.length);         // 3

colors[3] = "black";

console.log(colors.length);         // 4
console.log(colors[3]);             // "black" 
```

这个例子使用了 `set` 代理陷阱对数组索引的设置操作进行拦截。若该键能够作为数组索引，由于传入的键值始终都是字符串，那么就需要将其转换为一个数值；接下来，如果该数值大于或等于当前的 `length` 属性值，那么要把 `length` 属性值增加到比该数值多 1 （如在索引位置 3 设置一个项，则 `length` 属性必须是 4 ）；最后通过 `Reflect.set()` 来调用属性的默认设置操作，以便让对应属性接收到指定的值。

使用值为 3 的 `length` 参数调用 `createMyArray()` 函数，初始化了一个定制数组，接下来立刻将三个项添加到该数组内。数组的 `length` 属性一直保持为 3 ，直到 `"black"` 值被赋值到索引 3 的位置，此时 `length` 属性就变成了 4 。

这样就成功模拟了第一个行为，该继续处理第二个行为了。

#### 在减少长度属性时移除元素

仅当数组索引值大于或等于 `length` 属性值时，所需模拟的第一个数组行为才会被使用。而相反的，在将 `length` 属性值设置得比之前更小的时候，才需要使用第二个行为并移除数组的元素。此时不仅需要修改 `length` 属性的值，还需要移除所有不应再保留的元素。例如，若数组的 `length` 属性从 4 被设置为 2 ，则位置 2 与位置 3 的项就需要被移除。你可以像处理第一个行为那样，在 `set` 代理陷阱中完成这个操作。下面再次使用了前一段代码，并增加了 `createMyArray` 方法：

```js
function toUint32(value) {
    return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32);
}

function isArrayIndex(key) {
    let numericKey = toUint32(key);
    return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1);
}

function createMyArray(length=0) {
    return new Proxy({ length }, {
        set(trapTarget, key, value) {

            let currentLength = Reflect.get(trapTarget, "length");

            // 特殊情况
            if (isArrayIndex(key)) {
                let numericKey = Number(key);

                if (numericKey >= currentLength) {
                    Reflect.set(trapTarget, "length", numericKey + 1);
                }
            } else if (key === "length") {

                if (value < currentLength) {
                    for (let index = currentLength - 1; index >= value; index--) {
                        Reflect.deleteProperty(trapTarget, index);
                    }
                }

            }

            // 无论键的类型是什么，都要执行这行代码
            return Reflect.set(trapTarget, key, value);
        }
    });
}

let colors = createMyArray(3);
console.log(colors.length);         // 3

colors[0] = "red";
colors[1] = "green";
colors[2] = "blue";
colors[3] = "black";

console.log(colors.length);         // 4

colors.length = 2;

console.log(colors.length);         // 2
console.log(colors[3]);             // undefined
console.log(colors[2]);             // undefined
console.log(colors[1]);             // "green"
console.log(colors[0]);             // "red" 
```

此代码中的 `set` 陷阱函数会检查键的值是否为 `"length"` ，以便正确地调整对象的剩余项。如果是，则使用 `Reflect.get()` 方法来获取当前的长度值，并与新值作比较。如果新值小于当前值，将会使用一个 `for` 循环来删除对象上所有不应再被保留的属性，该循环从当前数组长度（ `currentLength` ）的位置向前删除每个属性，直到触及新的数组长度（ `value` ）为止。

该例子先向 `colors` 对象中添加了四个颜色，再将其 `length` 属性设置为 2 ，结果移除了位置 2 与位置 3 的项，这样在试图访问这两个项的时候就会得到 `undefined` 。而位置 0 与位置 1 的项仍然可被访问。

两个行为都实现之后，你就可以轻易创建一个对象来模拟内置数组的行为。然而使用函数来做这些事并不可取，最好将其封装为一个类，因此下一步就是使用类来实现这些功能。

#### 实现 MyArray 类

创建一个使用代理的类的最简单方式，就是照常定义一个类但从构造器中返回一个代理。这种方式下，该类被实例化时返回的对象就是代理，而不是该类的实例（实例即构造器内部的 `this` 值）。代理会像实例一样被返回，而实例此时就变成了该代理的目标对象。实例将会是完全私有的，无法被直接访问，不过你可以使用代理去间接访问它。

这里有一个从类构造器返回代理的简单范例：

```js
class Thing {
    constructor() {
        return new Proxy(this, {});
    }
}

let myThing = new Thing();
console.log(myThing instanceof Thing);      // true 
```

在这个例子中， `Thing` 类从它的构造器中返回了一个代理，该代理的目标对象是构造器被调用时其内部的 `this` 。这意味着虽然 `myThing` 对象是调用 `Thing` 构造器创建的，但它实际上是一个代理对象。由于此代理将行为直接传递给它的目标对象， 因而 `myThing` 仍然可以被认定为 `Thing` 类的一个实例，并且让代理在使用 `Thing` 类时完全透明。

知道了这些，使用代理来创建一个定制的数组类就相当简单了。它的实现代码与“在减少长度属性时移除元素”那个小节的代码非常接近，使用了相同的代理代码，但这次是在类的构造器中使用它。这里有个完整的范例：

```js
function toUint32(value) {
    return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32);
}

function isArrayIndex(key) {
    let numericKey = toUint32(key);
    return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1);
}

class MyArray {
    constructor(length=0) {
        this.length = length;

        return new Proxy(this, {
            set(trapTarget, key, value) {

                let currentLength = Reflect.get(trapTarget, "length");

                // 特殊情况
                if (isArrayIndex(key)) {
                    let numericKey = Number(key);

                    if (numericKey >= currentLength) {
                        Reflect.set(trapTarget, "length", numericKey + 1);
                    }
                } else if (key === "length") {

                    if (value < currentLength) {
                        for (let index = currentLength - 1; index >= value; index--) {
                            Reflect.deleteProperty(trapTarget, index);
                        }
                    }

                }

                // 无论键的类型是什么，都要执行这行代码
                return Reflect.set(trapTarget, key, value);
            }
        });

    }
}

let colors = new MyArray(3);
console.log(colors instanceof MyArray);     // true

console.log(colors.length);         // 3

colors[0] = "red";
colors[1] = "green";
colors[2] = "blue";
colors[3] = "black";

console.log(colors.length);         // 4

colors.length = 2;

console.log(colors.length);         // 2
console.log(colors[3]);             // undefined
console.log(colors[2]);             // undefined
console.log(colors[1]);             // "green"
console.log(colors[0]);             // "red" 
```

这段代码创建了一个 `MyArray` 类，并从构造器中返回了一个代理。在构造器中，添加了 `length` 属性（使用传入的值进行初始化，或者在值未提供的情况下使用默认的 0 ），然后创建并返回了一个代理。这让 `colors` 变量看起来就像是 `MyArray` 类的一个实例，并且实现了数组的两个关键行为。

虽然从类构造器中返回一个代理是很容易的，但这意味着每个实例都会创建一个新的代理。不过你可以将代理对象作为原型使用，这样就可以在所有实例上共享一个代理。

### 将代理对象作为原型使用

代理对象可以被作为原型使用，但这么做会比本章前面的例子更复杂一些。在把代理对象作为原型时，仅当操作的默认行为会按惯例追踪原型时，代理陷阱才会被调用，这就限制了代理对象作为原型时的能力。考虑这个例子：

```js
let target = {};
let newTarget = Object.create(new Proxy(target, {

    // 永远不会被调用
    defineProperty(trapTarget, name, descriptor) {

        // 如果被调用就会引发错误
        return false;
    }
}));

Object.defineProperty(newTarget, "name", {
    value: "newTarget"
});

console.log(newTarget.name);                    // "newTarget"
console.log(newTarget.hasOwnProperty("name"));  // true 
```

一个代理被作为原型创建了 `newTarget` 对象。将 `target` 作为代理的目标对象，有效地让 `target` 成为了 `newTarget` 的原型，因为该代理是透明的。此时，只有当 `newTarget` 将操作传递给 `target` 的时候，代理陷阱才会被调用。

`Object.defineProperty()` 方法在 `newTarget` 上被调用，创建了一个自有属性 `name` 。定义对象属性的操作并不会按惯例追踪对象原型，因此代理上的 `defineProperty` 陷阱函数永远不会被调用，于是 `name` 属性就被添加到了 `newTarget` 对象上，成为它的一个自有属性。

尽管在把代理对象作为原型时会受到严重限制，但仍然存在几个很有用的陷阱函数。

#### 在原型上使用 get 陷阱函数

当内部方法 `[[Get]]` 被调用以读取属性时，该操作首先会查找对象的自有属性；如果指定名称的属性没有找到，则会继续在对象的原型上进行属性查找；这个流程会一直持续到没有原型可供查找为止。

得益于这个流程，若你设置了一个 `get` 代理陷阱，则只有在对象不存在指定名称的自有属性时，该陷阱函数才会在对象的原型上被调用。当所访问的属性无法保证存在时，你可以使用 `get` 陷阱函数来阻止预期外的行为。下例创建了一个对象，当你尝试去访问一个不存在的属性时，它会抛出错误：

```js
let target = {};
let thing = Object.create(new Proxy(target, {
    get(trapTarget, key, receiver) {
        throw new ReferenceError(`${key} doesn't exist`);
    }
}));

thing.name = "thing";

console.log(thing.name);        // "thing"

// 抛出错误
let unknown = thing.unknown; 
```

这段代码创建了一个将代理作为原型的 `thing` 对象。当 `thing` 对象中不存在指定键的时候， `get` 陷阱函数就会抛出错误。在读取 `thing.name` 时，因为该属性存在于 `thing` 对象中， `get` 陷阱函数没有被调用；而当读取不存在的 `thing.unknown`属性时， `get` 陷阱函数才被调用了。

当最后一行代码执行时， `unknown` 并不是 `thing` 的自有属性，因此查找操作延续到了它的原型上，于是 `get` 陷阱函数抛出了一个错误。这种自定义行为对 JS 来说是非常有用的，因为它能够让 JS 像其他语言那样、在访问不存在的属性时抛出错误，而不是静默地返回 `undefined` 。

`trapTarget` 与 `receiver` 是不同的对象，这对理解本例是非常重要的。当代理被用作原型时， `trapTarget` 是原型对象自身，而 `receiver` 则是实例对象。这意味着在本例中， `trapTarget` 等于 `target` ，而 `receiver` 则等于 `thing` 。这就使得你既能访问代理的原始目标对象，也能访问操作将要涉及的对象。

#### 在原型上使用 set 陷阱函数

内部方法 `[[Set]]` 同样会查找对象的自有属性，并在必要时继续对该对象的原型进行查找。当你对一个对象属性进行赋值时，如果指定名称的自有属性存在，值就会被赋在该属性上；而若该自有属性不存在，则会继续检查对象的原型。微妙之处在于：尽管赋值操作在原型上继续进行，但默认情况下它会在对象实例（而非原型）上创建一个新的属性用于赋值，无论同名属性是否存在于原型上。

为了更好地了解 `set` 陷阱函数何时会在原型上被调用、而何时不会，可研究下面这个展示了默认行为的示例：

```js
let target = {};
let thing = Object.create(new Proxy(target, {
    set(trapTarget, key, value, receiver) {
        return Reflect.set(trapTarget, key, value, receiver);
    }
}));

console.log(thing.hasOwnProperty("name"));      // false

// 触发了 `set` 代理陷阱
thing.name = "thing";

console.log(thing.name);                        // "thing"
console.log(thing.hasOwnProperty("name"));      // true

// 没有触发 `set` 代理陷阱
thing.name = "boo";

console.log(thing.name);                        // "boo" 
```

在本例中， `target` 对象起初未拥有任何自有属性。 `thing` 对象把一个代理作为自身的原型，并定义了一个 `set` 陷阱函数来捕获任意创建新属性的操作。当 `thing.name` 被赋值为 `"thing"` 时，因为 `thing` 对象并不存在一个名为 `name` 的自有属性， `set` 代理陷阱就被调用。在 `set` 陷阱函数中， `trapTarget` 参数等于 `target` ，而 `receiver` 参数则等于 `thing` 。你可以将 `receiver` 作为第四个参数传递给 `Reflect.set()` 方法来实现默认的行为，最终一个新的属性就在 `thing` 对象上被创建了。

一旦 `thing` 对象的 `name` 属性被创建完毕，将 `thing.name` 另设为其他值就不会再触发原型上 `set` 代理陷阱，因为此时 `name` 变成了自有属性， `[[Set]]` 操作便不会再继续查找原型了。

#### 在原型上使用 has 陷阱函数

可以回忆一下， `has` 陷阱函数会拦截对象上 `in` 运算符的使用。 `in` 运算符首先查找对象上指定名称的自有属性；如果不存在同名自有属性，则会继续查找对象的原型；如果原型上也不存在同名自有属性，那么就会沿着原型链一直查找下去，直到找到该属性、或者没有更多原型可供查找时为止。

`has` 陷阱函数只在原型链查找触及原型对象的时候才会被调用。当使用代理作为原型时，这只会在指定名称的自有属性不存在时发生。例如：

```js
let target = {};
let thing = Object.create(new Proxy(target, {
    has(trapTarget, key) {
        return Reflect.has(trapTarget, key);
    }
}));

// 触发了 `has` 代理陷阱
console.log("name" in thing);                   // false

thing.name = "thing";

// 没有触发 `has` 代理陷阱
console.log("name" in thing);                   // true 
```

此代码在 `thing` 的原型上创建了一个 `has` 代理陷阱。 `has` 陷阱函数并没有像 `get` 或 `set` 陷阱函数那样传递一个 `receiver` 参数，因为当 `in` 运算符被使用时，对原型的查找是自动的。相反的， `has` 陷阱函数只能对 `trapTarget` 参数进行操作，该参数等于 `target` 。本例中第一次使用 `in` 运算符的时候，由于 `thing` 并不存在自有属性 `name` ，于是 `has` 陷阱函数就被调用了。而当 `thing.name` 被赋值之后，再次使用 `in` 运算符， `has` 陷阱函数则不会被调用，因为操作在找到 `thing` 的自有属性 `name` 后便已停止。

这里的原型范例都围绕着使用 `Object.create()` 方法创建的对象。然而若你想创建一个以代理为原型的对象，流程会有些不同。

#### 将代理作为类的原型

类不能直接被修改为将代理用作自身的原型，因为它们的 `prototype` 属性是不可写入的。然而你可以使用一点变通手段，利用继承来创建一个把代理作为自身原型的类。首先你需要使用构造器函数创建一个 ES5 风格的类定义。你可以将原型改写为一个代理，这里有个例子：

```js
function NoSuchProperty() {
    // empty
}

NoSuchProperty.prototype = new Proxy({}, {
    get(trapTarget, key, receiver) {
        throw new ReferenceError(`${key} doesn't exist`);
    }
});

let thing = new NoSuchProperty();

// 由于 `get` 代理陷阱而抛出了错误
let result = thing.name; 
```

`NoSuchProperty` 函数代表了将会被用于继承的基础类。此函数的 `prototype` 属性不存在任何限制，因此你可以将其改写为一个代理，其中 `get` 陷阱函数被用于在属性缺失时抛出错误。 `thing` 对象被创建为 `NoSuchProperty` 类的一个实例，当访问不存在的 `name` 属性时，错误就被抛出。

下一步是创建一个继承 `NoSuchProperty` 的类。你可以简单使用第九章介绍过的 `extends` 语法，来将代理引入该类的原型链，就像这样：

```js
function NoSuchProperty() {
    // empty
}

NoSuchProperty.prototype = new Proxy({}, {
    get(trapTarget, key, receiver) {
        throw new ReferenceError(`${key} doesn't exist`);
    }
});

class Square extends NoSuchProperty {
    constructor(length, width) {
        super();
        this.length = length;
        this.width = width;
    }
}

let shape = new Square(2, 6);

let area1 = shape.length * shape.width;
console.log(area1);                         // 12

// 由于 "wdth" 不存在而抛出了错误
let area2 = shape.length * shape.wdth; 
```

`Square` 类继承了 `NoSuchProperty` 类，因此该代理就被加入了 `Square` 类的原型链。随后 `shape` 对象被创建为 `Square` 类的一个实例，让它拥有两个属性： `length` 与 `width` 。由于 `get` 陷阱函数永远不会被调用，因此能够成功读取这两个属性的值。只有访问 `shape` 上不存在的属性时（例如这里的 `shape.wdth` 拼写错误），才触发了 `get` 陷阱函数并导致错误被抛出。

这证明了该代理存在于 `shape` 的原型链中，但这可能并不明显，因为该代理不是 `shape` 的直接原型。事实上，该代理需要用两步才能从 `shape` 的原型链上被找到。你可以修改前面的例子来更清晰地领会这一点：

```js
function NoSuchProperty() {
    // empty
}

// 对于将要用作原型的代理，存储对其的一个引用
let proxy = new Proxy({}, {
    get(trapTarget, key, receiver) {
        throw new ReferenceError(`${key} doesn't exist`);
    }
});

NoSuchProperty.prototype = proxy;

class Square extends NoSuchProperty {
    constructor(length, width) {
        super();
        this.length = length;
        this.width = width;
    }
}

let shape = new Square(2, 6);

let shapeProto = Object.getPrototypeOf(shape);

console.log(shapeProto === proxy);                  // false

let secondLevelProto = Object.getPrototypeOf(shapeProto);

console.log(secondLevelProto === proxy);            // true 
```

这个版本的代码将代理存储在一个名为 `proxy` 的变量中，以便之后可以简单识别。 `shape` 的原型是 `Shape.prototype` ，它并不是一个代理。然而 `Shape.prototype` 的原型却是一个从 `NoSuchProperty` 继承下来的代理。

继承行为在原型链上增加了一步，明白这一点很重要，因为在 `proxy` 变量上调用 `get` 陷阱函数的操作也需要多进行一步。如果欲使用的属性存在于 `Shape.prototype` 上，那么这就会防止 `get` 代理陷阱被调用，正如此例：

```js
function NoSuchProperty() {
    // empty
}

NoSuchProperty.prototype = new Proxy({}, {
    get(trapTarget, key, receiver) {
        throw new ReferenceError(`${key} doesn't exist`);
    }
});

class Square extends NoSuchProperty {
    constructor(length, width) {
        super();
        this.length = length;
        this.width = width;
    }

    getArea() {
        return this.length * this.width;
    }
}

let shape = new Square(2, 6);

let area1 = shape.length * shape.width;
console.log(area1);                         // 12

let area2 = shape.getArea();
console.log(area2);                         // 12

// 由于 "wdth" 不存在而抛出了错误
let area3 = shape.length * shape.wdth; 
```

此处的 `Square` 类拥有一个 `getArea()` 方法，该方法被自动添加到 `Square.prototype` 上，因此当 `shape.getArea()` 被调用时，对于 `getArea()` 方法的查找从 `shape` 实例上开始，并延续到它的原型上。由于在原型上找到了 `getArea()` 方法，查找就停止了，代理也没有被调用。在本例的条件下，这正是你想要的行为，而 `getArea()` 被调用时抛出错误则是不正确的。

尽管使用了一点额外的代码来创建一个类，才让代理存在于该类的原型链上，但当你确实需要这样的功能时，这种付出仍然是值得的。

### 总结

在 ES6 之前，特定对象（例如数组）会显示出一些非常规的、无法被开发者复制的行为，而代理的出现改变了这种情况。代理允许你为一些 JS 底层操作自行定义非常规行为，因此你就可以通过代理陷阱来复制 JS 内置对象的所有行为。在各种不同操作发生时（例如对于 `in` 运算符的使用），这些代理陷阱会在后台被调用。

反射接口也是在 ES6 中引入的，允许开发者为每个代理陷阱实现默认的行为。每个代理陷阱在 `Reflect` 对象（ ES6 的另一个新特性）上都有一个同名的对应方法。将代理陷阱与反射接口方法结合使用，就可以在特定条件下让一些操作有不同的表现，有别于默认的内置行为。

可被撤销的代理是一种特殊的代理，可以使用 `revoke()` 函数去有效禁用。 `revoke()` 函数终结了代理的所有功能，因此在它被调用之后，所有与代理属性交互的意图都会导致抛出错误。第三方开发者可能需要在一定时间内获取特定对象的使用权，在这种场合，可被撤销的代理对应用的安全性来说就非常重要。

尽管直接使用代理是最有力的使用方式，但你也可以把代理用作另一个对象的原型。但只有很少的代理陷阱能在作为原型的代理上被有效使用，包括 `get` 、 `set` 与 `has` 这几个，这让这方面的用例变得十分有限。