# 对象

Javascript 最原始的类型是`true`, `false`, 数字, 字符串, `null` and `undefined`。**其他每个值都是 `对象`.**

Javascript 对象包含成对的`propertyName（属性名）`: `propertyValue（属性值）`。

# 创建

There are two ways to create an `object` in JavaScript: 在 Javascript 中，有两种方法去创建一个 `对象` ：

1.  字面上

    ```js
     var object = {};
       // 是的，简单是使用一对大括号！ 
    ```

    > ***注意:*** 这是 **推荐** 的方法

2.  面向对象

    ```js
     var object = new Object(); 
    ```

    > ***注意:*** 这很像 Java

# 属性

对象的属性是一对的`propertyName(属性名)`: `propertyValue(属性值)`，**属性的名字只能是字符串**。如果不是字符串，将会转换为字符串。可以在创建对象或之后初始化属性。

```js
var language = {
    name: 'JavaScript',
    isSupportedByBrowsers: true,
    createdIn: 1995,
    author:{
        firstName: 'Brendan',
        lastName: 'Eich'
    },
 // 是的，对象可以嵌套！
    getAuthorFullName: function(){
        return this.author.firstName + " " + this.author.lastName;
    }
 // 是的，函数可以有值！
}; 
```

以下代码展示如何 **获取** 属性的值。

```js
var variable = language.name;
 // 变量包含字符串"JavaScript"
    variable = language['name'];
 // 这行代码和上行功能一样。不同之处在于这行代码将书面化的字符串作为属性名，不过缺少可读性。
    variable = language.newProperty; 
 // 变量没定义，因为该属性没赋值。 
```

以下代码展示如何**添加**一个新属性或**改变**一个存在的属性。

```js
language.newProperty = 'new value';
 // 现在对象有一个新的属性。如果该属性已经存在，值将会被替换。
language['newProperty'] = 'changed value';
 // 两以上种方法都可以使用，更推荐第一种(使用`.`)。 
```

# 可变性

对象与原始值不同之处在于**对象可以改变**，而原始值不可改变。

```js
var myPrimitive = "first value";
    myPrimitive = "another value";
 // myPrimitive 现在指向另一个字符串
var myObject = { key: "first value"};
    myObject.key = "another value";
 // myObject 指向的还是原来的对象 
```

# 引用

对象是**不可复制**的。它们的传递靠引用。

```js
// 假设我有一个匹萨
var myPizza = {slices: 5};
 // 然后我和你分析它
var yourPizza = myPizza;
 // 我吃了一小片
myPizza.slices = myPizza.slices - 1;
var numberOfSlicesLeft = yourPizza.slices;
 // 我们总共有 4 片
 // 因为我们引用了同一块匹萨
var a = {}, b = {}, c = {};
 // a, b, 和 c 都引用不同的空对象
a = b = c = {};
 // a, b, 和 c 都引用同一个空对象 
```

# 原型

每个对象都与对象原型关联，继承了对象原型的属性。

从文本对象（`{}`）创建的所有对象都自动链接到的 Object.prototype，这个对象来自 JavaScript 标准。

当 JavaScript 解释器（在浏览器中一个模块），试图找到一个属性，它要检索，如下面的代码：

```js
var adult = {age: 26},
    retrievedProperty = adult.age;
 // 看上一行 
```

首先，解释器检查对象有的每个属性。例如，`adult`只有一个自己的属性 - `age`。但是，除此之外，实际上还有几个属性，这是继承自 Object.prototype。

```js
var stringRepresentation = adult.toString();
 // 变量的值为 '[object Object]' 
```

`toString` 是一个 Object.prototype 的属性, 这是继承。它有一个函数，返回值为一个对象的字符串。如果希望它返回一个更有意义的东西，那么你可以将其覆盖。简单的添加一个属性到 adult 对象。

```js
adult.toString = function(){
    return "I'm "+this.age;
} 
```

如果现在调用 `toString` 函数，解释器将发现一个新的对象中的属性然后停止。

因此，通过深入原型，解释器检索第一个对象本身的属性。

要设置自己的对象为原型而不是默认的 Object.prototype，你可以调用以下的`Object.create`：

```js
var child = Object.create(adult);
 /* 通过这种方式创建的对象可以让我们轻松替换默认的 Object.prototype 成我们想要的。在这里，child 的原型是 adult 对象。
 */
child.age = 8;
 /* 在此之前，child 根本没有自己的年龄属性，解释器会寻找 child 的原型中是否有该属性。现在，当我们设置了 child 自身年龄，解释器就不深入寻找了。
注意：adult 的年龄仍为 26。
  */
var stringRepresentation = child.toString();
 // 值为 "I'm 8"。
 /* 注意：我们没覆盖 child 的 toString 属性，因此 adult 类函数不会被调用。如果 adult 没有 toString 属性，那么 Object.prototype 的 toString 类函数将被调用，我们将得到"[object Object]" 而不是 "I'm 8" 。
 */ 
```

`child`'的原型是`adult`，其原型为`Object.prototype`。这一系列原型被称为**原型链**。

# 销毁

`销毁` 被用来从一个对象中 **删除一个属性** 。 从一个对象中删除一个属性就是将改属性从原型中移出：

```js
var adult = {age:26},
    child = Object.create(adult);
    child.age = 8;

delete child.age;
 /* 从 child 中删除 age 属性，表明这之后该属性不在被覆盖 */
var prototypeAge = child.age;
 // 26，因为该孩子没有自己的 age 属性。 
```

# 枚举

`for in` 语句可以遍历对象中所有的属性。枚举包括函数和原型属性。

```js
var fruit = {
    apple: 2,
    orange:5,
    pear:1
},
sentence = 'I have ',
quantity;
for (kind in fruit){
    quantity = fruit[kind];
    sentence += quantity+' '+kind+
                (quantity===1?'':'s')+
                ', ';
}
 // The following line removes the trailing coma.
sentence = sentence.substr(0,sentence.length-2)+'.';
 // I have 2 apples, 5 oranges, 1 pear. 
```

# 全局化

如果想开发一个模块，它可以在网页上运行或也可以运行其他模块，因此你必须注意变量名是否重复。

假设我们正在卡开发一个计数器模块：

```js
var myCounter = {
    number : 0,
    plusPlus : function(){
        this.number : this.number + 1;
    },
    isGreaterThanTen : function(){
        return this.number > 10;
    }
} 
```

> ***注意:*** this 技巧通常用在闭包中，以使来自外部的内部状态不变。

模块使用唯一一个变量名 — `myCounter`。如果其他模块使用名字比如`number` 或 `isGreaterThanTen` ，这样就会很安全，因为不会覆盖每个其他的值。