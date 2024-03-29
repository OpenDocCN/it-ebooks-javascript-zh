# 3.9 JSON 对象

*   JSON 格式
*   JSON 对象
    *   JSON.stringify()
    *   JSON.parse()
    *   参考链接

## JSON 格式

JSON 格式（JavaScript Object Notation 的缩写）是一种用于数据交换的文本格式，2001 年由 Douglas Crockford 提出，目的是取代繁琐笨重的 XML 格式。

相比 XML 格式，JSON 格式有两个显著的优点：书写简单，一目了然；符合 JavaScript 原生语法，可以由解释引擎直接处理，不用另外添加代码。所以，JSON 迅速被接受，已经成为各大网站交换数据的标准格式，并被写入 ECMAScript 5，成为标准的一部分。

简单说，JSON 格式就是一种表示一系列的“值”的方法，这些值包含在数组或对象之中，是它们的成员。对于这一系列的“值”，有如下几点格式规定：

1.  数组或对象的每个成员的值，可以是简单值，也可以是复合值。

2.  简单值分为四种：字符串、数值（必须以十进制表示）、布尔值和 null（NaN, Infinity, -Infinity 和 undefined 都会被转为 null）。

3.  复合值分为两种：符合 JSON 格式的对象和符合 JSON 格式的数组。

4.  数组或对象最后一个成员的后面，不能加逗号。

5.  数组或对象之中的字符串必须使用双引号，不能使用单引号。

6.  对象的成员名称必须使用双引号。

以下是合格的 JSON 值。

```js
["one", "two", "three"]

{ "one": 1, "two": 2, "three": 3 }

{"names": ["张三", "李四"] }

[ { "name": "张三"}, {"name": "李四"} ]
```

以下是不合格的 JSON 值。

```js
{ name: "张三", 'age': 32 }  // 属性名必须使用双引号

[32, 64, 128, 0xFFF] // 不能使用十六进制值

{ "name": "张三", age: undefined } // 不能使用 undefined

{ "name": "张三",
  "birthday": new Date('Fri, 26 Aug 2011 07:13:10 GMT'),
  "getName": function() {
      return this.name;
  }
} // 不能使用函数和日期对象
```

> 需要注意的是，空数组和空对象都是合格的 JSON 值，null 本身也是一个合格的 JSON 值。

## JSON 对象

ES5 新增了 JSON 对象，用来处理 JSON 格式数据。它有两个方法：JSON.stringify 和 JSON.parse。

### JSON.stringify()

JSON.stringify 方法用于将一个值转为字符串。该字符串符合 JSON 格式，并且可以被 JSON.parse 方法还原。

```js
JSON.stringify("abc") // '"abc"'
JSON.stringify(1) // "1"
JSON.stringify(false) // "false"
JSON.stringify([]) // "[]"
JSON.stringify({}) // "{}"

JSON.stringify([1, "false", false])
// '[1,"false",false]'

JSON.stringify({ name: "张三" })
// '{"name":"张三"}'
```

上面代码将各种类型的值，转成 JSON 字符串。需要注意的是，对于原始类型的字符串，转换结果会带双引号，即字符串`abc`会被转成`"abc"`，这是因为将来还原的时候，双引号可以让 JavaScript 引擎知道，abc 是一个字符串，而不是一个变量名。

如果原始对象中，有一个成员的值是 undefined、函数或 XML 对象，这个成员会被省略。如果数组的成员是 undefined、函数或 XML 对象，则这些值被转成 null。

```js
JSON.stringify({
    f: function(){},
    a: [ function(){}, undefined ]
});
// "{"a":[null,null]}"
```

上面代码中，原始对象的 f 属性是一个函数，JSON.stringify 方法返回的字符串会将这个属性省略。而 a 属性是一个数组，成员分别为函数和 undefined，它们都被转成了 null。

正则对象会被转成空对象。

```js
JSON.stringify(/foo/) // "{}"
```

JSON.stringify 方法会忽略对象的不可遍历属性。

```js
var obj = {};
Object.defineProperties(obj, {
  'foo': {
    value: 1,
    enumerable: true
  },
  'bar': {
    value: 2,
    enumerable: false
  }
});

JSON.stringify(obj); // {"foo":1}
```

上面代码中，bar 是 obj 对象的不可遍历属性，JSON.stringify 方法会忽略这个属性。

JSON.stringify 方法还可以接受一个数组参数，指定需要转成字符串的属性。

```js
JSON.stringify({ a:1, b:2 }, ['a'])
// '{"a":1}'
```

上面代码中，JSON.stringify 方法的第二个参数指定，只转 a 属性。

JSON.stringify 方法还可以接受一个函数作为参数，用来更改默认的字符串化的行为。

```js
function f(key, value) {
  if (typeof value === "number") {
    value = 2 * value;
  }
  return value;
}

JSON.stringify({ a:1, b:2 }, f)
// '{"a":2,"b":4}'
```

上面代码中的 f 函数，接受两个参数，分别是被转化对象的键和值。如果一个键的值是数值，就将它乘以 2，否则就原样返回。

注意，这个处理函数是递归处理所有的键。

```js
var o = {a: {b:1}};

function f(key, value) {
  console.log("["+ key +"]:" + value);
  return value;
}

JSON.stringify(o, f)
// []:[object Object] 
// [a]:[object Object]
// [b]:1
// '{"a":{"b":1}}'
```

上面代码中，对象 o 一共会被 f 函数处理三次。第一次键名为空，键值是整个对象 o；第二次键名为 a，键值是`{b:1}`；第三次键名为 b，键值为 1。

递归处理中，每一次处理的对象，都是前一次返回的值。

```js
var o = {a: 1};

function f(key, value){
  if (typeof value === "object"){
    return {b: 2};
  }
  return value*2;
}

JSON.stringify(o,f)
// '{"b":4}'
```

上面代码中，f 函数修改了对象 o，接着 JSON.stringify 方法就递归处理修改后的对象 o。

如果处理函数返回 undefined 或没有返回值，则该属性会被忽略。

```js
function f(key, value) {
  if (typeof(value) == "string") {
    return undefined;
  }
  return value;
}

JSON.stringify({ a:"abc", b:123 }, f)
// '{"b":123}'
```

上面代码中，a 属性经过处理后，返回 undefined，于是该属性被忽略了。

JSON.stringify 还可以接受第三个参数，用于增加返回的 JSON 字符串的可读性。如果是数字，表示每个属性前面添加的空格（最多不超过 10 个）；如果是字符串（不超过 10 个字符），则该字符串会添加在每行前面。

```js
JSON.stringify({ p1:1, p2:2 }, null, 2);
// "{
  "p1": 1,
  "p2": 2
}"

JSON.stringify({ p1:1, p2:2 }, null, "|-");
// "{
|-"p1": 1,
|-"p2": 2
}"
```

如果 JSON.stringify 方法处理的对象，包含一个 toJSON 方法，则它会使用这个方法得到一个值，然后再将这个值转成字符串，而忽略其他成员。

```js
JSON.stringify({
  toJSON: function() {
    return "Cool"
  }
})
// "Cool""

var o = {
  foo: 'foo',
  toJSON: function() {
    return 'bar';
  }
};
var json = JSON.stringify({x: o}); 
// '{"x":"bar"}'
```

Date 对象就部署了一个自己的 toJSON 方法。

```js
JSON.stringify(new Date("2011-07-29"))
// "2011-07-29T00:00:00.000Z"
```

toJSON 方法的一个应用是，可以将正则对象自动转为字符串。

```js
RegExp.prototype.toJSON = RegExp.prototype.toString;

JSON.stringify(/foo/)
// "/foo/"
```

上面代码，在正则对象的原型上面部署了 toJSON 方法，将其指向 toString 方法，因此遇到转换成 JSON 时，就正则对象就先调用 toJSON 方法转为字符串，然后再被 JSON.stingify 方法处理。

### JSON.parse()

JSON.parse 方法用于将 JSON 字符串转化成对象。

```js
JSON.parse('{}') // {}
JSON.parse('true') // true
JSON.parse('"foo"') // "foo"
JSON.parse('[1, 5, "false"]') // [1, 5, "false"]
JSON.parse('null') // null

var o = JSON.parse('{"name":"张三"}');
o.name // 张三
```

如果传入的字符串不是有效的 JSON 格式，JSON.parse 方法将报错。

```js
JSON.parse("'String'") // illegal single quotes
// SyntaxError: Unexpected token ILLEGAL
```

上面代码中，双引号字符串中是一个单引号字符串，因为单引号字符串不符合 JSON 格式，所以报错。

为了处理解析错误，可以将 JSON.parse 方法放在 try...catch 代码块中。

JSON.parse 方法可以接受一个处理函数，用法与 JSON.stringify 方法类似。

```js
function f(key, value) {
  if ( key === ""){
      return value;
  }
  if ( key === "a" ) {
    return value + 10;
  }
}

var o = JSON.parse('{"a":1,"b":2}', f);
o.a // 11
o.b // undefined
```

## 参考链接

*   MDN, [Using native JSON](https://developer.mozilla.org/en-US/docs/Using_native_JSON)
*   MDN, [JSON.parse](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/JSON/parse)
*   Dr. Axel Rauschmayer, [JavaScript’s JSON API](http://www.2ality.com/2011/08/json-api.html)
*   Jim Cowart, [What You Might Not Know About JSON.stringify()](http://freshbrewedcode.com/jimcowart/2013/01/29/what-you-might-not-know-about-json-stringify/)
*   Marco Rogers polotek, [What is JSON?](http://docs.nodejitsu.com/articles/javascript-conventions/what-is-json)