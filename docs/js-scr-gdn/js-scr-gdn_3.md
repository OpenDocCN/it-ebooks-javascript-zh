# 数组

## 数组遍历与属性

虽然在 JavaScript 中数组是对象，但是没有好的理由去使用 `for in` 循环 遍历数组。 相反，有一些好的理由**不去**使用 `for in` 遍历数组。

**注意:** JavaScript 中数组**不是** *关联数组*。 JavaScript 中只有对象 来管理键值的对应关系。但是关联数组是**保持**顺序的，而对象**不是**。

由于 `for in` 循环会枚举原型链上的所有属性，唯一过滤这些属性的方式是使用 `hasOwnProperty` 函数， 因此会比普通的 `for` 循环慢上好多倍。

### 遍历

为了达到遍历数组的最佳性能，推荐使用经典的 `for` 循环。

```js
var list = [1, 2, 3, 4, 5, ...... 100000000];
for(var i = 0, l = list.length; i < l; i++) {
    console.log(list[i]);
} 
```

上面代码有一个处理，就是通过 `l = list.length` 来缓存数组的长度。

虽然 `length` 是数组的一个属性，但是在每次循环中访问它还是有性能开销。 **可能**最新的 JavaScript 引擎在这点上做了优化，但是我们没法保证自己的代码是否运行在这些最近的引擎之上。

实际上，不使用缓存数组长度的方式比缓存版本要慢很多。

### `length` 属性

`length` 属性的 *getter* 方式会简单的返回数组的长度，而 *setter* 方式会**截断**数组。

```js
var foo = [1, 2, 3, 4, 5, 6];
foo.length = 3;
foo; // [1, 2, 3]

foo.length = 6;
foo; // [1, 2, 3] 
```

**译者注：** 在 Firebug 中查看此时 `foo` 的值是： `[1, 2, 3, undefined, undefined, undefined]` 但是这个结果并不准确，如果你在 Chrome 的控制台查看 `foo` 的结果，你会发现是这样的： `[1, 2, 3]` 因为在 JavaScript 中 `undefined` 是一个变量，注意是变量不是关键字，因此上面两个结果的意义是完全不相同的。

```js
// 译者注：为了验证，我们来执行下面代码，看序号 5 是否存在于 foo 中。
5 in foo; // 不管在 Firebug 或者 Chrome 都返回 false
foo[5] = undefined;
5 in foo; // 不管在 Firebug 或者 Chrome 都返回 true 
```

为 `length` 设置一个更小的值会截断数组，但是增大 `length` 属性值不会对数组产生影响。

### 结论

为了更好的性能，推荐使用普通的 `for` 循环并缓存数组的 `length` 属性。 使用 `for in` 遍历数组被认为是不好的代码习惯并倾向于产生错误和导致性能问题。

## `Array` 构造函数

由于 `Array` 的构造函数在如何处理参数时有点模棱两可，因此总是推荐使用数组的字面语法 - `[]` - 来创建数组。

```js
[1, 2, 3]; // 结果: [1, 2, 3]
new Array(1, 2, 3); // 结果: [1, 2, 3]

[3]; // 结果: [3]
new Array(3); // 结果: [] 
new Array('3') // 结果: ['3']

// 译者注：因此下面的代码将会使人很迷惑
new Array(3, 4, 5); // 结果: [3, 4, 5] 
new Array(3) // 结果: []，此数组长度为 3 
```

**译者注：**这里的模棱两可指的是数组的[两种构造函数语法](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Array)

由于只有一个参数传递到构造函数中（译者注：指的是 `new Array(3);` 这种调用方式），并且这个参数是数字，构造函数会返回一个 `length` 属性被设置为此参数的空数组。 需要特别注意的是，此时只有 `length` 属性被设置，真正的数组并没有生成。

**译者注：**在 Firebug 中，你会看到 `[undefined, undefined, undefined]`，这其实是不对的。在上一节有详细的分析。

```js
var arr = new Array(3);
arr[1]; // undefined
1 in arr; // false, 数组还没有生成 
```

这种优先于设置数组长度属性的做法只在少数几种情况下有用，比如需要循环字符串，可以避免 `for` 循环的麻烦。

```js
new Array(count + 1).join(stringToRepeat); 
```

**译者注：** `new Array(3).join('#')` 将会返回 `##`

### 结论

应该尽量避免使用数组构造函数创建新数组。推荐使用数组的字面语法。它们更加短小和简洁，因此增加了代码的可读性。