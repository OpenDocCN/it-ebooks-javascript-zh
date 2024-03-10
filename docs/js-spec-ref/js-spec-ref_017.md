# 3.2 Array 对象

*   概述
*   Array 对象的静态方法
    *   isArray 方法
    *   Array 实例的方法
        *   valueOf 方法，toString 方法
        *   push 方法，pop 方法
        *   join 方法，concat 方法
        *   shift 方法，unshift 方法
        *   reverse 方法
        *   slice 方法
        *   splice()
        *   sort()
    *   ECMAScript 5 新加入的数组方法
        *   map 方法，forEach 方法
        *   filter 方法
        *   some 方法，every 方法
        *   reduce 方法，reduceRight 方法
        *   indexOf 和 lastIndexOf
        *   链式使用
    *   参考链接

## 概述

Array 是 JavaScript 的内置对象，同时也是一个构造函数，可以用它生成新的数组。

作为构造函数时，Array 可以接受参数，但是不同的参数，会使得 Array 产生不同的行为。

*   无参数时，返回一个空数组。

*   单个参数时，如果该参数是正整数，则这个正整数表示新数组的长度；如果该参数是非正整数（比如字符串、布尔值、对象等），则该值是新数组的成员。

*   多个参数时，这些参数都是新数组的成员。

```js
var a1 = new Array();
var a2 = new Array(1);
var a3 = new Array('abc');
var a4 = new Array([1]);
var a5 = new Array(1,2);

a1 // []
a2 // [undefined × 1]
a3 // ['abc']
a4 // [Array[1]]
a5 // [1, 2]
```

从上面代码可以看到，Array 的构造函数行为很不一致。因此，不建议使用它生成新数组，直接使用数组的字面量是更好的方法。

## Array 对象的静态方法

### isArray 方法

Array.isArray 方法用来判断一个值是否为数组。它可以弥补 typeof 运算符的不足。

```js
var a = [1,2,3];

typeof a // "object"

Array.isArray(a) // true
```

上面代码表示，typeof 运算符只能显示数组的类型是 Object，而 Array.isArray 方法可以对数组返回 true。

## Array 实例的方法

以下这些 Array 实例对象的方法，都是数组实例才能使用。如果不想创建实例，只是想单纯调用这些方法，可以写成 [].method.call(调用对象，参数) 的形式，或者 Array.prototype.method.call(调用对象，参数)的形式。

### valueOf 方法，toString 方法

valueOf 方法返回数组本身。

```js
var a = [1,2,3];

a.valueOf()
// [1,2,3]
```

toString 方法返回数组的字符串形式。

```js
var a = [1,2,3];

a.toString()
// "1,2,3"

var a = [1,2,3,[4,5,6]];

a.toString()
// "1,2,3,4,5,6"
```

### push 方法，pop 方法

push 方法用于在数组的末端添加一个或多个元素，并返回添加后的数组的长度。

```js
var a = [];

a.push(1) // 1
a.push("a") // 2
a.push(true, {}) // 4
a // [1, "a", true, {}]
```

上面代码使用 push 方法，先后往数组中添加了四个成员。

如果需要合并两个数组，可以这样写。

```js
var a = [1,2,3];
var b = [4,5,6];

Array.prototype.push.apply(a, b)
// 或者
a.push.apply(a,b)

// 上面两种写法等同于
a.push(4,5,6)

a
// [1, 2, 3, 4, 5, 6]
```

push 方法还可以用于向对象添加元素，添加后的对象变成“类似数组的”对象，即新加入元素的键对应数组的索引，并且对象有一个 length 属性。

```js
var a = { a: 1 };

[].push.call(a, 2);
a
// {a:1, 0:2, length: 1}

[].push.call(a, [3]);
a
// {a:1, 0:2, 1:[3], length: 2}
```

pop 方法用于删除数组的最后一个元素，并返回该元素。

```js
var a = ['a', 'b', 'c'];

a.pop() // 'c'
a // ['a', 'b']
```

对空数组使用 pop 方法，不会报错，而是返回 undefined。

```js
[].pop() // undefined
```

### join 方法，concat 方法

join 方法以参数作为分隔符，将所有数组成员组成一个字符串返回。如果不提供参数，默认用逗号分隔。

```js
var a = [1,2,3,4];

a.join() // "1,2,3,4"
a.join('') // '1234'
a.join("|") // "1|2|3|4"
```

通过函数的 call 方法，join 方法（即 Array.prototype.join）也可以用于字符串。

```js
Array.prototype.join.call('hello', '-')
// "h-e-l-l-o"
```

concat 方法将新数组的成员，添加到原数组的尾部，然后返回一个新数组，常用于连接多个数组。

```js
["hello"].concat(["world"])
// ["hello", "world"]

[1,2,3].concat(4,5,6)
// [1, 2, 3, 4, 5, 6]
```

上面代码表明，concat 方法的参数可以是一个或多个数组，以及原始类型的值。

如果不提供参数，concat 方法返回当前数组的一个浅拷贝。所谓“浅拷贝”，指的是如果数组成员包括复合类型的值（比如对象），则新数组拷贝的是该值的引用。

```js
var o = {a:1};
var oldArray = [o];

var newArray = oldArray.concat();

o.a = 2;
newArray[0].a // 2
```

上面代码中，原数组包含一个对象，concat 方法生成的新数组包含这个对象的引用。所以，改变原对象以后，新数组跟着改变。事实上，只要原数组的成员中包含对象，concat 方法不管有没有参数，总是返回该对象的引用。

concat 方法也可以用于将对象合并为数组，但是必须借助 call 方法。

```js
[].concat.call({ a: 1 }, [2])
// [{a:1}, 2]

// 等同于

[2].concat({a:1})
```

### shift 方法，unshift 方法

shift 方法用于删除数组的第一个元素，并返回该元素。

```js
var a = ['a', 'b', 'c'];

a.shift() // 'a'
a // ['b', 'c']
```

shift 方法可以遍历并清空一个数组。

```js
var list = [1,2,3,4,5,6,7,8,9,10];

var item;

while (item = list.shift()) {
    console.log(item);
}

list // []
```

unshift 方法用于在数组的第一个位置添加元素，并返回添加新元素后的数组长度。

```js
var a = ['a', 'b', 'c'];

a.unshift('x'); // 4
a // ['x', 'a', 'b', 'c']
```

### reverse 方法

reverse 方法用于颠倒数组中元素的顺序，使用这个方法以后，返回改变后的原数组。

```js
var a = ['a', 'b', 'c'];

a.reverse() // ["c", "b", "a"] 
a // ["c", "b", "a"]
```

### slice 方法

slice 方法返回指定位置的数组成员组成的新数组，原数组不变。它的第一个参数为起始位置（从 0 开始），第二个参数为终止位置（但该位置的元素本身不包括在内）。如果省略第二个参数，则一直返回到原数组的最后一个成员。

```js
var a = ["a","b","c"];

a.slice(1,2) // ["b"]
a.slice(1) // ["b", "c"]
a.slice(0) // ["a","b","c"]
a.slice(-2) // ["b", "c"]
a.slice(4) // []
a.slice(2, 6) // ["c"]
a.slice(2, 1) // []
```

上面代码表示，如果 slice 方法的参数是负数，则从尾部开始选择的成员个数；如果参数值大于数组成员的个数，或者第二个参数小于第一个参数，则返回空数组。

slice 方法的一个重要应用，是将类似数组的对象转为真正的数组。

```js
Array.prototype.slice.call({ 0: 'a', 1: 'b', length: 2 })
// ['a', 'b']

Array.prototype.slice.call(document.querySelectorAll("div"));

Array.prototype.slice.call(arguments);
```

上面代码的参数都不是数组，但是通过 call 方法，在它们上面调用 slice 方法，就可以把它们转为真正的数组。

### splice()

splice 方法用于删除元素，并可以在被删除的位置添加入新的数组元素。它的返回值是被删除的元素。需要特别注意的是，该方法会改变原数组。

splice 的第一个参数是删除的起始位置，第二个参数是被删除的元素个数。如果后面还有更多的参数，则表示这些就是要被插入数组的新元素。

```js
var a = ["a","b","c","d","e","f"];

a.splice(4,2)
// ["e", "f"]

a
// ["a", "b", "c", "d"]
```

上面代码从原数组位置 4 开始，删除了两个数组成员。

```js
var a = ["a","b","c","d","e","f"];

a.splice(4,2,1,2)
// ["e", "f"]

a
// ["a", "b", "c", "d", 1, 2]
```

上面代码除了删除成员，还插入了两个新成员。

如果只是单纯地插入元素，splice 方法的第二个参数可以设为 0。

```js
var a = [1,1,1];

a.splice(1,0,2)
// []

a
// [1, 2, 1, 1]
```

如果只提供第一个参数，则实际上等同于将原数组在指定位置拆分成两个数组。

```js
var a = [1,2,3,4];

a.splice(2)
// [3, 4]

a
// [1, 2]
```

### sort()

sort 方法对数组元素进行排序，默认是按照字典顺序排序。排序后，原数组将被改变。

```js
["d","c","b","a"].sort()
// ["a", "b", "c", "d"]

[4,3,2,1].sort()
// [1, 2, 3, 4]

[11,101].sort()
// [101, 11]

[10111,1101,111].sort()
// [10111, 1101, 111]
```

上面代码的最后两个例子，需要特殊注意。sort 方法不是按照大小排序，而是按照对应字符串的字典顺序排序，所以 101 排在 11 的前面。

如果想让 sort 方法按照大小排序，可以传入一个函数作为参数，表示按照自定义方法进行排序。该函数本身又接受两个参数，表示进行比较的两个元素。如果返回值大于 0，表示第一个元素排在第二个元素后面；其他情况下，都是第一个元素排在第二个元素前面。

```js
[10111,1101,111].sort(function (a,b){
  return a-b;
})
// [111, 1101, 10111]

[
  { name: "张三", age: 30 },
  { name: "李四", age: 24 },
  { name: "王五", age: 28  }
].sort(function(o1, o2) {
  return o1.age - o2.age;
})
// [
//   { name: "李四", age: 24 },
//   { name: "王五", age: 28  },
//   { name: "张三", age: 30 }
// ]
```

## ECMAScript 5 新加入的数组方法

ECMAScript 5 新增了 9 个数组实例的方法，分别是 map、forEach、filter、every、some、reduce、reduceRight、indexOf 和 lastIndexOf。其中，前 7 个与函数式（functional）操作有关。

这些方法可以在数组上使用，也可以在字符串和类似数组的对象上使用，这是它们不同于传统数组方法的一个地方。

在用法上，这些方法的参数是一个函数，这个作为参数的函数本身又接受三个参数：数组的当前元素 elem、该元素的位置 index 和整个数组 arr（详见下面的实例）。另外，上下文对象（context）可以作为第二个参数，传入 forEach(), every(), some(), filter(), map()方法，用来绑定函数运行时的上下文。

对于不支持这些方法的老式浏览器（主要是 IE 8 及以下版本），可以使用函数库[es5-shim](https://github.com/kriskowal/es5-shim)，或者[Underscore](http://underscorejs.org/#filter)和[Lo-Dash](http://lodash.com/docs#filter)。

### map 方法，forEach 方法

map 方法对数组的所有成员依次调用一个函数，根据函数结果返回一个新数组。

```js
var numbers = [1, 2, 3];

numbers.map(function(n){ return n+1 });
// [2, 3, 4]

numbers
// [1, 2, 3]
```

上面代码中，原数组的成员都加上 1，组成一个新数组返回，原数组没有变化。

只要数组的成员可以被索引到，map 方法就不会跳过它。

```js
var f = function(n){ return n+1 };

[1, undefined, 2].map(f) // [2, NaN, 3]
[1, null, 2].map(f) // [2, 1, 3]
[1, , 2].map(f) // [2, undefined, 3]
```

上面代码中，数组的成员分别是 undefined、null 和空位，map 方法都不会跳过它们。

map 方法的回调函数依次接受三个参数，分别是当前的数组成员、当前成员的位置和数组本身。

```js
[1, 2, 3].map(function(elem, index, arr){
  return elem * elem;
});
// [1, 4, 9]
```

有时，我们需要对字符串的每个字符进行遍历。这时可以通过函数的 call 方法，将 map 方法用于字符串。

```js
var upper = function (x) { return x.toUpperCase() };

[].map.call('abc', upper)
// [ 'A', 'B', 'C' ]

// 或者
'abc'.split('').map(upper)
// [ 'A', 'B', 'C' ]
```

其他类似数组的对象（比如 querySelectorAll 方法返回 DOM 节点集合），也可以用上面的方法遍历。

map 方法还可以接受第二个参数，表示回调函数执行时 this 所指向的对象。

数组实例的 forEach 方法与 map 方法很相似，也是遍历数组的所有成员，执行某种操作，但是 forEach 方法没有返回值。如果需要有返回值，一般使用 map 方法，如果只是单纯操作数据，一般使用 forEach 方法。

```js
function log(element, index, array) {
  console.log('[' + index + '] = ' + element);
}

[2, 5, , 9].forEach(log);
// [0] = 2
// [1] = 5
// [3] = 9
```

从上面代码可以看到，forEach 方法和 map 方法的参数格式是一样的，第一个参数都是一个函数。该函数接受三个参数，分别是当前元素、当前元素的位置（从 0 开始）、整个数组。

forEach 方法会跳过数组的空位。

```js
var log = function(n){ console.log(n + 1) };

[1, undefined, 2].forEach(log)
// 2
// NaN
// 3

[1, null, 2].forEach(log)
// 2
// 1
// 3

[1, , 2].map(f)
// 2
// 3
```

上面代码中，forEach 方法不会跳过 undefined 和 null，但会跳过空位。

forEach 方法也可以接受第二个参数，用来绑定回调函数的 this 关键字。

```js
var out = [];

[1, 2, 3].map(function(elem, index, arr){
  this.push(elem * elem);
}, out);

out // [1, 4, 9]
```

上面代码表示，如果提供一个数组作为第二个参数，则函数内部的 this 关键字就指向这个数组。

### filter 方法

filter 方法依次对所有数组成员调用一个测试函数，返回结果为 true 的成员组成一个新数组返回。

```js
[1, 2, 3, 4, 5].filter(function (elem){
  return (elem > 3);
})
// [4,5]
```

上面代码将大于 3 的原数组成员，作为一个新数组返回。

filter 方法的测试函数可以接受三个参数，第一个参数是当前数组成员的值，这是必需的，后两个参数是可选的，分别是当前数组成员的位置和整个数组。

```js
[1, 2, 3, 4, 5].filter(function(elem, index, arr){
  return index % 2 === 0;
});
// [1, 3, 5]
```

上面代码返回原数组偶数位置的成员组成的新数组。

filter 方法还可以接受第二个参数，指定测试函数所在的上下文对象（即 this 对象）。

```js
var Obj = function () {
  this.MAX = 3;
};

var myFilter = function(item) {
  if (item > this.MAX) {
    return true;
  }
};

var arr = [2,8,3,4,1,3,2,9];
arr.filter(myFilter, new Obj())
// [8, 4, 9]
```

上面代码中，测试函数 myFilter 内部有 this 对象，它可以被 filter 方法的第二个参数绑定。

### some 方法，every 方法

这两个方法类似“断言”（assert），用来判断数组成员是否符合某种条件。

some 方法对所有元素调用一个测试函数，只要有一个元素通过该测试，就返回 true，否则返回 false。

```js
[1, 2, 3, 4, 5].some(function(elem, index, arr){
    return elem >= 3;
});
// 返回 true
```

上面代码表示，如果存在大于等于 3 的数组成员，就返回 true。

every 方法对所有元素调用一个测试函数，只有所有元素通过该测试，才返回 true，否则返回 false。

```js
[1, 2, 3, 4, 5].every(function(elem, index, arr){
    return elem >= 3;
});
// 返回 false
```

上面代码表示，只有所有数组成员大于等于 3，才返回 true。

从上面的代码可以看到，some 和 every 的使用方法与 map 和 forEach 是一致的，参数完全一模一样。也就是说，它们也可以使用第二个参数，用来绑定函数中的 this 关键字。

### reduce 方法，reduceRight 方法

reduce 方法和 reduceRight 方法的作用，是依次处理数组的每个元素，最终累计为一个值。这两个方法的差别在于，reduce 对数组元素的处理顺序是从左到右（从第一个成员到最后一个成员），reduceRight 则是从右到左（从最后一个成员到第一个成员），其他地方完全一样。

reduce 方法的第一个参数是一个处理函数。该函数接受四个参数，分别是：

1.  用来累计的变量（即当前状态），默认值为 0
2.  数组的当前元素
3.  当前元素在数组中的序号（从 0 开始）
4.  原数组

这四个参数之中，只有前两个是必须的，后两个则是可选的。

```js
[1, 2, 3, 4, 5].reduce(function(x, y){
    return x+y;
});
// 15
```

上面代码的参数 x 表示累计变量，默认为 0，y 则是数组的当前元素。reduce 方法依次将每个数组元素加入 x，最终返回它们的总和 15。

利用 reduce 方法，可以写一个数组求和的 sum 方法。

```js
Array.prototype.sum = function (){
    return this.reduce(function (partial, value){
        return partial + value;
    })
};

[3,4,5,6,10].sum()
// 28
```

如果要对累计变量指定初值，可以把它放在 reduce 方法的第二个参数。

```js
[1, 2, 3, 4, 5].reduce(function(x, y){
  return x+y;
}, 10);
// 25
```

上面代码指定参数 x 的初值为 10，所以数组元素从 10 开始累加，最终结果为 25。

由于 reduce 方法依次处理每个元素，所以实际上还可以用它来搜索某个元素。比如，下面代码是找出长度最长的数组元素。

```js
function findLongest(entries) {
  return entries.reduce(function (longest, entry) {
    return entry.length > longest.length ? entry : longest;
  }, '');
}
```

### indexOf 和 lastIndexOf

ECMAScript 5 新增的 9 个方法之中，有 2 个与函数式编程无关，分别是 indexOf 和 lastIndexOf。

indexOf 方法返回给定元素在数组中第一次出现的位置，如果没有出现则返回-1。

```js
var a = ['a','b','c'];

a.indexOf('b')
// 1

a.indexOf('y')
// -1
```

indexOf 方法还可以接受第二个参数，表示搜索的开始位置。

```js
['a','b','c'].indexOf('a', 1)
// -1
```

上面代码从位置 1 开始搜索字符 a，结果为-1，表示没有搜索到。

lastIndexOf 方法返回给定元素在数组中最后一次出现的位置，如果没有出现则返回-1。

```js
var a = [2, 5, 9, 2];

a.lastIndexOf(2)
// 3

a.lastIndexOf(7)
// -1
```

注意，如果数组中包含 NaN，这两个方法不适用。

```js
[NaN].indexOf(NaN) // -1
[NaN].lastIndexOf(NaN) // -1
```

这是因为这两个方法内部，使用严格相等运算符（===）进行比较，而 NaN 是唯一一个不等于自身的值。

### 链式使用

上面这些数组方法之中，有不少返回的还是数组，所以可以链式使用。

```js
var users = [{name:"tom", email:"tom@example.com"},
             {name:"peter", email:"peter@example.com"}];

users
.map(function (user){ return user.email; })
.filter(function (email) { return /^t/.test(email); })
.forEach(alert);
// 弹出 tom@example.com
```

## 参考链接

*   Nicolas Bevacqua, [Fun with JavaScript Native Array Functions](http://flippinawesome.org/2013/11/25/fun-with-javascript-native-array-functions/)