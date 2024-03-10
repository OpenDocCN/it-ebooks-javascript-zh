# 第四章 扩展的对象功能

## 第四章 扩展的对象功能

ES6 注重于提高对象的效用，这是因为在 JS 中几乎所有的值都是某种类型的对象。此外，随着 JS 应用的复杂度增长，在 JS 程序中所使用的对象的平均数也在持续增长，更多的对象就让有效使用它们变得更加必要。

在对象的许多方面——从简单的语法扩展，到操作与交互—— ES6 都进行了改进。

*   对象类别
*   对象字面量语法的扩展
    *   属性初始化器的速记法
    *   方法简写
    *   需计算属性名
*   新的方法
    *   Object.is() 方法
    *   Object.assign() 方法
*   重复的对象字面量属性
*   自有属性的枚举顺序
*   更强大的原型
    *   修改对象的原型
    *   使用 super 引用的简单原型访问
*   正式的“方法”定义
*   总结

### 对象类别

JS 使用混合术语来描述能在标准中找到的对象，而不是那些由运行环境（例如浏览器或 Node.js ）所添加的，并且 ES6 规范还明确定义了对象的每种类别。理解对象术语对于从整体上清楚认识这门语言来说非常重要。对象类别包括：

*   **普通对象**：拥有 JS 对象所有默认的内部行为。
*   **奇异对象**：其内部行为在某些方面有别于默认行为。
*   **标准对象**：在 ES6 中被定义的对象，例如 `Array` 、 `Date` ，等等。标准对象可以是普通的，也可以是奇异的。
*   **内置对象**：在脚本开始运行时由 JS 运行环境提供的对象。所有的标准对象都是内置对象。

我会在整本书中使用这些术语来讲解在 ES6 中定义的各种对象。

### 对象字面量语法的扩展

对象字面量是 JS 中最流行的模式之一（ JSON 就是基于这种语法），而它还存在于互联网上的几乎所有 JS 文件中。对象字面量如此流行，是因为它是一种创建对象的简洁语法（否则要多写不少代码）。对于开发者来说，幸运的是 ES6 用几种方式扩展了对象字面量，将这种语法变得更加强大、更加简洁。

#### 属性初始化器的速记法

在 ES5 及更早版本中，对象字面量是“键/值对”的简单集合。这意味着在属性值被初始化时可能会有些重复，例如：

```js
function createPerson(name, age) {
    return {
        name: name,
        age: age
    };
} 
```

`createPerson()` 函数创建了一个对象，其属性名与函数的参数名相同。此结果看起来重复了 `name` 与 `age` ，尽管一边是对象属性的名称，而另一边则负责给属性提供值。在所返回的对象中，它的 `name` 键与 `age` 键分别被变量 `name` 与 `age` 变量所赋值。

在 ES6 中，你可以使用**属性初始化器**的速记法来消除对象名称与本地变量的重复情况。当对象的一个属性名称与本地变量名相同时，你可以简单书写名称而省略冒号与值。例如， `createPerson()` 可以像这样用 ES6 重写：

```js
function createPerson(name, age) {
    return {
        name,
        age
    };
} 
```

当对象字面量中的属性只有名称时， JS 引擎会在周边作用域查找同名变量。若找到，该变量的值将会被赋给对象字面量的同名属性。在本例中，局部变量 `name` 的值就被赋给了 `name` 属性。

这个扩展使得对象字面量的初始化更加简洁，也有助于消除命名错误。用局部变量为对象同名属性赋值在 JS 中是极其常见的模式，因此这个扩展自然非常受欢迎。

#### 方法简写

ES6 同样改进了为对象字面量方法赋值的语法。在 ES5 及更早版本中，你必须指定一个名称并用完整的函数定义来为对象添加方法，如下：

```js
var person = {
    name: "Nicholas",
    sayName: function() {
        console.log(this.name);
    }
}; 
```

通过省略冒号与 `function` 关键字， ES6 将这个语法变得更简洁，这意味着你可以这样重写上个例子：

```js
var person = {
    name: "Nicholas",
    sayName() {
        console.log(this.name);
    }
}; 
```

这种速记语法也被称为**方法简写**语法（ **concise method** syntax ），与上例一样在 `person` 对象中创建了一个方法。 `sayName()` 属性被一个匿名函数所赋值，并且具备 ES5 的 `sayName()` 方法的所有特征。而有一点区别是：方法简写能使用 `super` ，而非简写的方法则不能（ `super` 会在后面的“使用 super 引用的简单原型访问”小节中讨论）。

> 使用方法简写速记法创建的方法，其 `name` 属性（名称属性）就是括号之前的名称。上面这个例子中， `person.sayName()` 的名称属性就是 `"sayName"` 。

#### 需计算属性名

在 ES5 及更早版本中，对象实例能使用“需计算的属性名”，只要用方括号表示法来代替小数点表示法即可。方括号允许你将变量或字符串字面量指定为属性名，而在字符串中允许存在作为标识符时会导致语法错误的特殊字符。这里有个范例：

```js
var person = {},
    lastName = "last name";

person["first name"] = "Nicholas";
person[lastName] = "Zakas";

console.log(person["first name"]);      // "Nicholas"
console.log(person[lastName]);          // "Zakas" 
```

`lastName` 变量已被赋值为 `"last name"` ，因此该例中两个属性名都包含了空格，这样就无法用小数点表示法来引用它们了。然而，方括号表示法允许将任意字符串用作属性名，这样 `"first name"` 与 `"last name"` 属性就能分别被赋值为 `"Nicholas"` 与 `"Zakas"` 。

此外，你可以在对象字面量中将字符串字面量直接用作属性，就像这样：

```js
var person = {
    "first name": "Nicholas"
};

console.log(person["first name"]);      // "Nicholas" 
```

这种模式要求属性名事先已知、并且能用字符串字面量表示。然而，若属性名被包含在变量中（就像前面例子中的 `"first name"` ），或者必须通过计算才能获得，那么在 ES5 的对象字面量中就无法定义这种属性。

在 ES6 中，需计算属性名是对象字面量语法的一部分，它用的也是方括号表示法，与此前在对象实例上的用法一致。例如：

```js
var lastName = "last name";

var person = {
    "first name": "Nicholas",
    [lastName]: "Zakas"
};

console.log(person["first name"]);      // "Nicholas"
console.log(person[lastName]);          // "Zakas" 
```

对象字面量内的方括号表明该属性名需要计算，其结果是一个字符串。这意味着其中可以包含表达式，像下面这样：

```js
var suffix = " name";

var person = {
    ["first" + suffix]: "Nicholas",
    ["last" + suffix]: "Zakas"
};

console.log(person["first name"]);      // "Nicholas"
console.log(person["last name"]);       // "Zakas" 
```

这些属性名被计算为 `"first name"` 与 `"last name"` ，而这两个字符串此后可以用来引用对应属性。使用方括号表示法，任何能放在对象实例方括号内的东西，都可以作为需计算属性名用在对象字面量中。

### 新的方法

ES 从 ES5 开始就有一个设计意图：避免创建新的全局函数，避免在 `Object` 对象的原型上添加新方法，而是尝试寻找哪些对象应该被添加新方法。因此，对其他对象不适用的新方法就被添加到全局的 `Object` 对象上。 ES6 在 `Object` 对象上引入了两个新方法，以便让特定任务更易完成。

#### Object.is() 方法

当在 JS 中要比较两个值时，你可能会使用相等运算符（ `==` ）或严格相等运算符（ `===` ）。为了避免在比较时发生强制类型转换，许多开发者更倾向于使用后者。但严格相等运算符也并不完全准确，例如，它认为 `+0` 与 `-0` 相等，即使这两者在 JS 引擎中有不同的表示；另外 `NaN === NaN` 会返回 `false` ，因此有必要使用 `isNaN()` 函数来正确检测 `NaN` 。

ES6 引入了 `Object.is()` 方法来弥补严格相等运算符残留的怪异点。此方法接受两个参数，并会在二者的值相等时返回 `true` ，此时要求二者类型相同并且值也相等。这有个例子：

```js
console.log(+0 == -0);              // true
console.log(+0 === -0);             // true
console.log(Object.is(+0, -0));     // false

console.log(NaN == NaN);            // false
console.log(NaN === NaN);           // false
console.log(Object.is(NaN, NaN));   // true

console.log(5 == 5);                // true
console.log(5 == "5");              // true
console.log(5 === 5);               // true
console.log(5 === "5");             // false
console.log(Object.is(5, 5));       // true
console.log(Object.is(5, "5"));     // false 
```

在许多情况下， `Object.is()` 的结果与 `===` 运算符是相同的，仅有的例外是：它会认为 `+0` 与 `-0` 不相等，而且 `NaN` 等于 `NaN` 。不过仍然没必要停止使用严格相等运算符，选择 `Object.is()` ，还是选择 `==` 或 `===` ，取决于代码的实际情况。

#### Object.assign() 方法

**混入**（ **Mixin** ）是在 JS 中组合对象时最流行的模式。在一次混入中，一个对象会从另一个对象中接收属性与方法。很多 JS 的库中都有类似下面的混入方法：

```js
function mixin(receiver, supplier) {
    Object.keys(supplier).forEach(function(key) {
        receiver[key] = supplier[key];
    });

    return receiver;
} 
```

`mixin()` 函数在 `supplier` 对象的自有属性上进行迭代，并将这些属性复制到 `receiver` 对象（浅复制，当属性值为对象时，仅复制其引用）。这样 `receiver` 对象就能获得新的属性而无须使用继承，正如下面代码：

```js
function EventTarget() { /*...*/ }
EventTarget.prototype = {
    constructor: EventTarget,
    emit: function() { /*...*/ },
    on: function() { /*...*/ }
};

var myObject = {};
mixin(myObject, EventTarget.prototype);

myObject.emit("somethingChanged"); 
```

此处 `myObject` 对象接收了 `EventTarget.prototype` 对象的行为，这给了它分别使用 `emit()` 与 `on()` 方法来发布事件与订阅事件的能力。

此模式已经足够流行，于是 ES6 就添加了 `Object.assign()` 方法来完成同样的行为。该方法接受一个接收者，以及任意数量的供应者，并会返回接收者。方法名称从 `mixin()` 变更为 `assign()` 更能反映出实际发生的操作。由于 `mixin()` 函数使用了赋值运算符（ `=` ），它就无法将访问器属性复制到接收者上， `Object.assign()` 体现了这种区别。

> 各式各样的库中都有相似但名称不同的方法，其基本功能相同，流行的替代方法包括 `extend()` 或 `mix()` 。而 ES6 也曾在 `Object.assign()` 之外短暂存在一个 `Object.mixin()` 方法，二者的主要差异在于 `Object.mixin()` 也会复制访问器属性，但考虑到 `super` 的使用（详见本章的“使用 super 引用的简单原型访问”小节），此方法最终被移除了。

你可以在任意曾使用 `mixin()` 函数的地方使用 `Object.assign()` ，此处有个例子：

```js
function EventTarget() { /*...*/ }
EventTarget.prototype = {
    constructor: EventTarget,
    emit: function() { /*...*/ },
    on: function() { /*...*/ }
}

var myObject = {}
Object.assign(myObject, EventTarget.prototype);

myObject.emit("somethingChanged"); 
```

`Object.assign()` 方法接受任意数量的供应者，而接收者会按照供应者在参数中的顺序来依次接收它们的属性。这意味着在接收者中，第二个供应者的属性可能会覆盖第一个供应者的，这在下面的代码片段中就发生了：

```js
var receiver = {};

Object.assign(receiver,
    {
        type: "js",
        name: "file.js"
    },
    {
        type: "css"
    }
);

console.log(receiver.type);     // "css"
console.log(receiver.name);     // "file.js" 
```

`receiver.type` 的值为 `"css"` ，这是因为第二个供应者覆盖了第一个供应者的值。

`Object.assign()` 方法并不是 ES6 的一项重大扩展，但它确实将很多 JS 库中的一个公共方法标准化了。

> **操作访问器属性**
> 
> 需要记住 `Object.assign()` 并未在接收者上创建访问器属性，即使供应者拥有访问器属性。由于 `Object.assign()` 使用赋值运算符，供应者的访问器属性就会转变成接收者的数据属性，例如：
> 
> ```js
> var receiver = {},
> supplier = {
>   get name() {
>       return "file.js"
>   }
> };
> 
> Object.assign(receiver, supplier);
> 
> var descriptor = Object.getOwnPropertyDescriptor(receiver, "name");
> 
> console.log(descriptor.value);      // "file.js"
> console.log(descriptor.get);        // undefined 
> ```
> 
> 此代码中的 `supplier` 对象拥有一个名为 `name` 的访问器属性。在使用了 `Object.assign()` 方法后， `receiver.name` 就作为一个数据属性存在了，其值为 `"file.js"` ，这是因为在调用 `Object.assign()` 时， `supplier.name` 返回的值是 `"file.js"` 。

### 重复的对象字面量属性

ES5 严格模式为重复的对象字面量属性引入了一个检查，若找到重复的属性名，就会抛出错误。例如，以下代码就有问题：

```js
"use strict";

var person = {
    name: "Nicholas",
    name: "Greg"        // 在 ES5 严格模式中是语法错误
}; 
```

在 ES5 严格模式下运行时，第二个 `name` 属性会造成语法错误。但 ES6 移除了重复属性的检查，严格模式与非严格模式都不再检查重复的属性。当存在重复属性时，排在后面的属性的值会成为该属性的实际值，如下所示：

```js
"use strict";

var person = {
    name: "Nicholas",
    name: "Greg"        // 在 ES6 严格模式中不会出错
};

console.log(person.name);       // "Greg" 
```

在本例中， `person.name` 的值为 `"Greg"` ，因为这是赋给该属性的最后一个值。

### 自有属性的枚举顺序

ES5 并没有定义对象属性的枚举顺序，而是把该问题留给了 JS 引擎厂商。而 ES6 则严格定义了对象自有属性在被枚举时返回的顺序。这对 `Object.getOwnPropertyNames()` 与 `Reflect.ownKeys` （详见第十二章）如何返回属性造成了影响，还同样影响了 `Object.assign()` 处理属性的顺序。

自有属性枚举时基本顺序如下：

1.  所有的数字类型键，按升序排列。
2.  所有的字符串类型键，按被添加到对象的顺序排列。
3.  所有的符号类型（详见第六章）键，也按添加顺序排列。

这里有个示例：

```js
var obj = {
    a: 1,
    0: 1,
    c: 1,
    2: 1,
    b: 1,
    1: 1
};

obj.d = 1;

console.log(Object.getOwnPropertyNames(obj).join(""));     // "012acbd" 
```

`Object.getOwnPropertyNames()` 方法按 `0` 、 `1` 、 `2` 、 `a` 、 `c` 、 `b` 、 `d` 的顺序返回了 `obj` 对象的属性。注意，数值类型的键会被合并并排序，即使这未遵循在对象字面量中的顺序。字符串类型的键会跟在数值类型的键之后，按照被添加到 `obj` 对象的顺序，在对象字面量中定义的键会首先出现，接下来是此后动态添加到对象的键。

> `for-in` 循环的枚举顺序仍未被明确规定，因为并非所有的 JS 引擎都采用相同的方式。而 `Object.keys()` 和 `JSON.stringify()` 也使用了与 `for-in` 一样的枚举顺序。

虽然枚举顺序的变动对 JS 的工作方式影响甚小，但是依赖于特定枚举顺序才能正确运行的程序并不罕见。因此 ES6 通过规定枚举的顺序，以确保依赖枚举操作的 JS 代码都能正常工作，而不用在意其运行环境。

### 更强大的原型

原型是在 JS 中进行继承的基础， ES6 则在继续让原型更强大。早期的 JS 版本对原型的使用有严重限制，然而随着语言的成熟，开发者也越来越熟悉原型的工作机制，因此他们明显希望能对原型有更多控制权，并能更方便地使用它。于是 ES6 就给原型引入了一些改进。

#### 修改对象的原型

一般来说，对象的原型会在通过构造器或 `Object.create()` 方法创建该对象时被指定。直到 ES5 为止， JS 编程最重要的假定之一就是对象的原型在初始化完成后会保持不变。尽管 ES5 添加了 `Object.getPrototypeOf()` 方法来从任意指定对象中获取其原型，但仍然缺少在初始化之后更改对象原型的标准方法。

ES6 通过添加 `Object.setPrototypeOf()` 方法而改变了这种假定，此方法允许你修改任意指定对象的原型。它接受两个参数：需要被修改原型的对象，以及将会成为前者原型的对象。例如：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

let dog = {
    getGreeting() {
        return "Woof";
    }
};

// 原型为 person
let friend = Object.create(person);
console.log(friend.getGreeting());                      // "Hello"
console.log(Object.getPrototypeOf(friend) === person);  // true

// 将原型设置为 dog
Object.setPrototypeOf(friend, dog);
console.log(friend.getGreeting());                      // "Woof"
console.log(Object.getPrototypeOf(friend) === dog);     // true 
```

此代码定义了两个基础对象： `person` 与 `dog` ，二者都拥有一个名为 `getGreeting()` 的方法，用于返回一个字符串。 `friend` 对象起初继承了 `person` 对象，意味着 `friend.getGreeting()` 方法会输出 `"Hello"` ；当它的原型被更改为 `dog` 对象， `friend.getGreeting()` 方法就会改而输出 `"Woof"` ，因为原先与 `person` 的关联已经被破坏了。

对象原型的实际值被存储在一个内部属性 `[[Prototype]]` 上， `Object.getPrototypeOf()` 方法会返回此属性存储的值，而 `Object.setPrototypeOf()` 方法则能够修改该值。不过，使用 `[[Prototype]]` 属性的方式还不止这些。

#### 使用 super 引用的简单原型访问

正如前面提到的，原型对 JS 来说非常重要，而 ES6 也进行了很多工作来让它更易用。关于原型的另一项进步就是引入了 `super` 引用，这让在对象原型上的功能调用变得更容易。例如，若要覆盖对象实例的一个方法、但依然要调用原型上的同名方法，你可能会这么做：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

let dog = {
    getGreeting() {
        return "Woof";
    }
};

let friend = {
    getGreeting() {
        return Object.getPrototypeOf(this).getGreeting.call(this) + ", hi!";
    }
};

// 将原型设置为 person
Object.setPrototypeOf(friend, person);
console.log(friend.getGreeting());                      // "Hello, hi!"
console.log(Object.getPrototypeOf(friend) === person);  // true

// 将原型设置为 dog
Object.setPrototypeOf(friend, dog);
console.log(friend.getGreeting());                      // "Woof, hi!"
console.log(Object.getPrototypeOf(friend) === dog);     // true 
```

本例中 `friend` 上的 `getGreeting()` 调用了对象上的同名方法。 `Object.getPrototypeOf()` 方法确保了能调用正确的原型，并在其返回结果上附加了一个字符串；而附加的 `call(this)` 代码则能确保正确设置原型方法内部的 `this` 值。

调用原型上的方法时要记住使用 `Object.getPrototypeOf()` 与 `.call(this)` ，这有点复杂难懂，因此 ES6 才引入了 `super` 。简单来说， `super` 是指向当前对象的原型的一个指针，实际上就是 `Object.getPrototypeOf(this)` 的值。知道这些，你就可以像下面这样简化 `getGreeting()` 方法：

```js
let friend = {
    getGreeting() {
        // 这相当于上个例子中的：
        // Object.getPrototypeOf(this).getGreeting.call(this)
        return super.getGreeting() + ", hi!";
    }
}; 
```

此处调用 `super.getGreeting()` 等同于在上例的环境中使用 `Object.getPrototypeOf(this).getGreeting.call(this)` 。类似的，你能使用 `super` 引用来调用对象原型上的任何方法，只要这个引用是位于简写的方法之内。试图在方法简写之外的情况使用 `super` 会导致语法错误，正如下例：

```js
let friend = {
    getGreeting: function() {
        // 语法错误
        return super.getGreeting() + ", hi!";
    }
}; 
```

此例使用了一个函数作为具名方法，于是调用 `super.getGreeting()` 就导致了语法错误，因为在这种上下文中 `super` 是不可用的。

当使用多级继承时， `super` 引用就是非常强大的，因为这种情况下 `Object.getPrototypeOf()` 不再适用于所有场景，例如：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

// 原型为 person
let friend = {
    getGreeting() {
        return Object.getPrototypeOf(this).getGreeting.call(this) + ", hi!";
    }
};
Object.setPrototypeOf(friend, person);

// 原型为 friend
let relative = Object.create(friend);

console.log(person.getGreeting());                  // "Hello"
console.log(friend.getGreeting());                  // "Hello, hi!"
console.log(relative.getGreeting());                // error! 
```

调用 `Object.getPrototypeOf()` 时，在调用 `relative.getGreeting()` 处发生了错误。这是因为此时 `this` 的值是 `relative` ，而 `relative` 的原型是 `friend` 对象，这样 `friend.getGreeting().call()` 调用就会导致进程开始反复进行递归调用，直到发生堆栈错误。

此问题在 ES5 中很难解决，但若使用 ES6 的 `super` ，就很简单了：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

// 原型为 person
let friend = {
    getGreeting() {
        return super.getGreeting() + ", hi!";
    }
};
Object.setPrototypeOf(friend, person);

// 原型为 friend
let relative = Object.create(friend);

console.log(person.getGreeting());                  // "Hello"
console.log(friend.getGreeting());                  // "Hello, hi!"
console.log(relative.getGreeting());                // "Hello, hi!" 
```

由于 `super` 引用并非是动态的，它总是能指向正确的对象。在本例中， `super.getGreeting()` 总是指向 `person.getGreeting()` ，而不管有多少对象继承了此方法。

### 正式的“方法”定义

在 ES6 之前，“方法”的概念从未被正式定义，它此前仅指对象的函数属性（而非数据属性）。 ES6 则正式做出了定义：方法是一个拥有 `[[HomeObject]]` 内部属性的函数，此内部属性指向该方法所属的对象。研究以下例子：

```js
let person = {

    // 方法
    getGreeting() {
        return "Hello";
    }
};

// 并非方法
function shareGreeting() {
    return "Hi!";
} 
```

此例定义了拥有单个 `getGreeting()` 方法的 `person` 对象。由于 `getGreeting()` 被直接赋给了一个对象，它的 `[[HomeObject]]` 属性值就是 `person` 。 而另一方面， `shareGreeting()` 函数没有被指定 `[[HomeObject]]` 属性，因为它在被创建时并没有赋给一个对象。大多数情况下，这种差异并不重要，然而使用 `super` 引用时就完全不同了。

任何对 `super` 的引用都会使用 `[[HomeObject]]` 属性来判断要做什么。第一步是在 `[[HomeObject]]` 上调用 `Object.getPrototypeOf()` 来获取对原型的引用；接下来，在该原型上查找同名函数；最后，创建 `this` 绑定并调用该方法。这里有个例子：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

// 原型为 person
let friend = {
    getGreeting() {
        return super.getGreeting() + ", hi!";
    }
};
Object.setPrototypeOf(friend, person);

console.log(friend.getGreeting());  // "Hello, hi!" 
```

调用 `friend.getGreeting()` 返回了一个字符串，也就是 `person.getGreeting()` 的返回值与 `", hi!"` 的合并结果。此时 `friend.getGreeting()` 的 `[[HomeObject]]` 值是 `friend` ，并且 `friend` 的原型是 `person` ，因此 `super.getGreeting()` 就等价于 `person.getGreeting.call(this)` 。

### 总结

对象是 JS 编程的中心， ES6 对它进行了一些有益改进，让它更易用并且更加强大。

ES6 为对象字面量做了几个改进。速记法属性定义能够更轻易地将作用域内的变量赋值给对象的同名属性；需计算属性名允许你将非字面量的值指定为属性的名称，就像此前在其他场合的用法那样；方法简写让你在对象字面量中定义方法时能省略冒号和 `function` 关键字，从而减少输入的字符数； ES6 还舍弃了对象字面量中重复属性名的检查，意味着你可以在一个对象字面量中书写两个同名属性，而不会抛出错误。

`Object.assign()` 方法使得一次性更改单个对象的多个属性变得更加容易，这在你使用混入模式时非常有用。 `Object.is()` 方法对任何值都会执行严格相等比较，当在处理特殊的 JS 值时，它有效成为了 `===` 的一个更安全的替代品。

对象自有属性的枚举顺序在 ES6 中被明确定义了。在枚举属性时，数字类型的键总是会首先出现，并按升序排列，此后是字符串类型的键，最后是符号类型的键，后两者都分别按添加顺序排列。

感谢 ES6 的 `Object.setPrototypeOf()` 方法，现在能够在对象已被创建之后更改它的原型了。

最后，你能用 `super` 关键字来调用对象原型上的方法，所调用的方法会被设置好其内部的 `this` 绑定，以自动使用该 `this` 值来进行工作。