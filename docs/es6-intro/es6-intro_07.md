# 数组的扩展

## Array.from()

Array.from 方法用于将两类对象转为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括 ES6 新增的数据结构 Set 和 Map）。

```js
let ps = document.querySelectorAll('p');

Array.from(ps).forEach(function (p) {
  console.log(p);
});

```

上面代码中，querySelectorAll 方法返回的是一个类似数组的对象，只有将这个对象转为真正的数组，才能使用 forEach 方法。

Array.from 方法可以将函数的 arguments 对象，转为数组。

```js
function foo() {
  var args = Array.from( arguments );
}

foo( "a", "b", "c" );

```

任何有 length 属性的对象，都可以通过 Array.from 方法转为数组。

```js
Array.from({ 0: "a", 1: "b", 2: "c", length: 3 });
// [ "a", "b" , "c" ]

```

对于还没有部署该方法的浏览器，可以用 Array.prototyp.slice 方法替代。

```js
const toArray = (() =>
  Array.from ? Array.from : obj => [].slice.call(obj)
)();

```

Array.from()还可以接受第二个参数，作用类似于数组的 map 方法，用来对每个元素进行处理。

```js
Array.from(arrayLike, x => x * x);
// 等同于
Array.from(arrayLike).map(x => x * x);

```

下面的例子将数组中布尔值为 false 的成员转为 0。

```js
Array.from([1, , 2, , 3], (n) => n || 0)
// [1, 0, 2, 0, 3]

```

Array.from()的一个应用是，将字符串转为数组，然后返回字符串的长度。这样可以避免 JavaScript 将大于\uFFFF 的 Unicode 字符，算作两个字符的 bug。

```js
function countSymbols(string) {
  return Array.from(string).length;
}

```

## Array.of()

Array.of 方法用于将一组值，转换为数组。

```js
Array.of(3, 11, 8) // [3,11,8]
Array.of(3) // [3]
Array.of(3).length // 1

```

这个方法的主要目的，是弥补数组构造函数 Array()的不足。因为参数个数的不同，会导致 Array()的行为有差异。

```js
Array() // []
Array(3) // [undefined, undefined, undefined]
Array(3,11,8) // [3, 11, 8]

```

上面代码说明，只有当参数个数不少于 2 个，Array()才会返回由参数组成的新数组。

Array.of 方法可以用下面的代码模拟实现。

```js
function ArrayOf(){
  return [].slice.call(arguments);
}

```

## 数组实例的 find()和 findIndex()

数组实例的 find 方法，用于找出第一个符合条件的数组成员。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为 true 的成员，然后返回该成员。如果没有符合条件的成员，则返回 undefined。

```js
var found = [1, 4, -5, 10].find((n) => n < 0);
console.log("found:", found);

```

上面代码找出数组中第一个小于 0 的成员。

```js
[1, 5, 10, 15].find(function(value, index, arr) {
  return value > 9;
}) // 10

```

上面代码中，find 方法的回调函数可以接受三个参数，依次为当前的值、当前的位置和原数组。

数组实例的 findIndex 方法的用法与 find 方法非常类似，返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回-1。

```js
[1, 5, 10, 15].findIndex(function(value, index, arr) {
  return value > 9;
}) // 2

```

这两个方法都可以接受第二个参数，用来绑定回调函数的 this 对象。

另外，这两个方法都可以发现 NaN，弥补了数组的 IndexOf 方法的不足。

```js
[NaN].indexOf(NaN)
// -1

[NaN].findIndex(y => Object.is(NaN, y))
// 0

```

上面代码中，indexOf 方法无法识别数组的 NaN 成员，但是 findIndex 方法可以借助 Object.is 方法做到。

## 数组实例的 fill()

fill()使用给定值，填充一个数组。

```js
['a', 'b', 'c'].fill(7)
// [7, 7, 7]

new Array(3).fill(7)
// [7, 7, 7]

```

上面代码表明，fill 方法用于空数组的初始化非常方便。数组中已有的元素，会被全部抹去。

fill()还可以接受第二个和第三个参数，用于指定填充的起始位置和结束位置。

```js
['a', 'b', 'c'].fill(7, 1, 2)
// ['a', 7, 'c']

```

## 数组实例的 entries()，keys()和 values()

ES6 提供三个新的方法——entries()，keys()和 values()——用于遍历数组。它们都返回一个遍历器，可以用 for...of 循环进行遍历，唯一的区别是 keys()是对键名的遍历、values()是对键值的遍历，entries()是对键值对的遍历。

```js
for (let index of ['a', 'b'].keys()) {
  console.log(index);
}
// 0
// 1

for (let elem of ['a', 'b'].values()) {
  console.log(elem);
}
// 'a'
// 'b'

for (let [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
// 0 "a"
// 1 "b"

```

## 数组实例的 includes()

Array.protypeto.includes 方法返回一个布尔值，表示某个数组是否包含给定的值。该方法属于 ES7。

```js
[1, 2, 3].includes(2);     // true
[1, 2, 3].includes(4);     // false
[1, 2, NaN].includes(NaN); // true

```

该方法的第二个参数表示搜索的起始位置，默认为 0。

```js
[1, 2, 3].includes(3, 3);  // false
[1, 2, 3].includes(3, -1); // true

```

下面代码用来检查当前环境是否支持该方法，如果不支持，部署一个简易的替代版本。

```js
const contains = (() =>
  Array.prototype.includes
    ? (arr, value) => arr.includes(value)
    : (arr, value) => arr.some(el => el === value)
)();
contains(["foo", "bar"], "baz"); // => false

```

## 数组推导

数组推导（array comprehension）提供简洁写法，允许直接通过现有数组生成新数组。这项功能本来是要放入 ES6 的，但是 TC39 委员会想继续完善这项功能，让其支持所有数据结构（内部调用 iterator 对象），不像现在只支持数组，所以就把它推迟到了 ES7。Babel 转码器已经支持这个功能。

```js
var a1 = [1, 2, 3, 4];
var a2 = [for (i of a1) i * 2];

a2 // [2, 4, 6, 8]

```

上面代码表示，通过 for...of 结构，数组 a2 直接在 a1 的基础上生成。

注意，数组推导中，for...of 结构总是写在最前面，返回的表达式写在最后面。

for...of 后面还可以附加 if 语句，用来设定循环的限制条件。

```js
var years = [ 1954, 1974, 1990, 2006, 2010, 2014 ];

[for (year of years) if (year > 2000) year];
// [ 2006, 2010, 2014 ]

[for (year of years) if (year > 2000) if(year < 2010) year];
// [ 2006]

[for (year of years) if (year > 2000 && year < 2010) year];
// [ 2006]

```

上面代码表明，if 语句写在 for...of 与返回的表达式之间，可以使用多个 if 语句。

数组推导可以替代 map 和 filter 方法。

```js
[for (i of [1, 2, 3]) i * i];
// 等价于
[1, 2, 3].map(function (i) { return i * i });

[for (i of [1,4,2,3,-8]) if (i < 3) i];
// 等价于
[1,4,2,3,-8].filter(function(i) { return i < 3 });

```

上面代码说明，模拟 map 功能只要单纯的 for...of 循环就行了，模拟 filter 功能除了 for...of 循环，还必须加上 if 语句。

在一个数组推导中，还可以使用多个 for...of 结构，构成多重循环。

```js
var a1 = ["x1", "y1"];
var a2 = ["x2", "y2"];
var a3 = ["x3", "y3"];

[for (s of a1) for (w of a2) for (r of a3) console.log(s + w + r)];
// x1x2x3
// x1x2y3
// x1y2x3
// x1y2y3
// y1x2x3
// y1x2y3
// y1y2x3
// y1y2y3

```

上面代码在一个数组推导之中，使用了三个 for...of 结构。

需要注意的是，数组推导的方括号构成了一个单独的作用域，在这个方括号中声明的变量类似于使用 let 语句声明的变量。

由于字符串可以视为数组，因此字符串也可以直接用于数组推导。

```js
[for (c of 'abcde') if (/[aeiou]/.test(c)) c].join('') // 'ae'

[for (c of 'abcde') c+'0'].join('') // 'a0b0c0d0e0'

```

上面代码使用了数组推导，对字符串进行处理。

数组推导需要注意的地方是，新数组会立即在内存中生成。这时，如果原数组是一个很大的数组，将会非常耗费内存。

## Array.observe()，Array.unobserve()

这两个方法用于监听（取消监听）数组的变化，指定回调函数。

它们的用法与 Object.observe 和 Object.unobserve 方法完全一致，也属于 ES7 的一部分，请参阅《对象的扩展》一章。唯一的区别是，对象可监听的变化一共有六种，而数组只有四种：add、update、delete、splice（数组的 length 属性发生变化）。