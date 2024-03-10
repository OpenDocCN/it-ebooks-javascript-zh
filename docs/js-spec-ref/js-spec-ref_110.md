# 13.13 assert 模块

assert 模块是 Node 的内置模块，主要用于断言。如果表达式不符合预期，就抛出一个错误。该模块提供 11 个方法，但只有少数几个是常用的。

*   assert()
*   assert.ok()
*   assert.equal()
*   assert.notEqual()
*   assert.deepEqual()
*   assert.notDeepEqual()
*   assert.strictEqual()
*   assert.notStrictEqual()
*   assert.throws()
*   assert.doesNotThrow()
*   assert.ifError()
*   assert.fail()

## assert()

assert 方法接受两个参数，当第一个参数对应的布尔值为 true 时，不会有任何提示，返回 undefined。当第一个参数对应的布尔值为 false 时，会抛出一个错误，该错误的提示信息就是第二个参数设定的字符串。

```
// 格式
assert(value, message)

// 例子
var assert = require('assert');

function add (a, b) {
  return a + b;
}

var expected = add(1,2);
assert( expected === 3, '预期 1 加 2 等于 3');
```

上面代码不会有任何输出，因为 assert 方法的第一个参数是 true。

```
assert( expected === 4, '预期 1 加 2 等于 3')
// AssertionError: 预期 1 加 2 等于 3
```

上面代码会抛出一个错误，因为 assert 方法的第一个参数是 false。

## assert.ok()

ok 是 assert 方法的另一个名字，与 assert 方法完全一样。

## assert.equal()

equal 方法接受三个参数，第一个参数是实际值，第二个是预期值，第三个是错误的提示信息。

```
// 格式
assert.equal(actual, expected, [message])

assert.equal(true, value, message);
// 等同于
assert(value, message);

// 例子
var assert = require('assert');

function add (a, b) {
  return a + b;
}

var expected = add(1,2);

// 以下三句效果相同
assert(expected == 3, '预期 1+2 等于 3');
assert.ok(expected == 3, '预期 1+2 等于 3');
assert.equal(expected, 3, '预期 1+2 等于 3');
```

equal 方法内部使用的是相等运算符（==），而不是严格运算符（===），进行比较运算。

## assert.notEqual()

notEqual 方法的用法与 equal 方法类似，但只有在实际值等于预期值时，才会抛出错误。

```
// 格式
assert.notEqual(actual, expected, [message])

// 用法
var assert = require('assert');

function add (a, b) {
  return a + b;
}

var expected = add(1,2);

// 以下三种写法效果相同
assert(expected != 4, '预期不等于 4');
assert.ok(expected != 4, '预期不等于 4');
assert.notEqual(expected, 4, '预期不等于 4');
```

notEqual 方法内部使用不相等运算符（!=），而不是严格不相等运算符（!==），进行比较运算。

## assert.deepEqual()

deepEqual 方法用来比较两个对象。只要它们的属性一一对应，且值都相等，就认为两个对象相等，否则抛出一个错误。

```
// 格式
assert.deepEqual(actual, expected, [message])

// 例子
var assert = require('assert');

var list1 = [1, 2, 3, 4, 5];
var list2 = [1, 2, 3, 4, 5];

assert.deepEqual(list1, list2, '预期两个数组应该有相同的属性');

var person1 = { "name":"john", "age":"21" };
var person2 = { "name":"john", "age":"21" };

assert.deepEqual(person1, person2, '预期两个对象应该有相同的属性');
```

## assert.notDeepEqual()

notDeepEqual 方法与 deepEqual 方法正好相反，用来断言两个对象是否不相等。

```
// 格式
assert.notDeepEqual(actual, expected, [message])

// 例子
var assert = require('assert');

var list1 = [1, 2, ,3, 4, 5];
var list2 = [1, 2, 3, 4, 5];

assert.notDeepEqual(list1, list2, '预期两个对象不相等');

var person1 = { "name":"john", "age":"21" };
var person2 = { "name":"jane", "age":"19" };

// deepEqual checks the elements in the objects are identical
assert.notDeepEqual(person1, person2, '预期两个对象不相等');
```

## assert.strictEqual()

strictEqual 方法使用严格相等运算符（===），比较两个表达式。

```
// 格式
assert.strictEqual(actual, expected, [message])

// 例子
var assert = require('assert');

assert.strictEqual(1, '1', '预期严格相等');
// AssertionError: 预期严格相等
```

## assert.notStrictEqual()

assert.notStrictEqual 方法使用严格不相等运算符（!==），比较两个表达式。

```
// 格式
assert.notStrictEqual(actual, expected, [message])

// 例子
var assert = require('assert');

assert.notStrictEqual(1, true, '预期严格不相等');
```

## assert.throws()

throws 方法预期某个代码块会抛出一个错误，且抛出的错误符合指定的条件。

```
// 格式
assert.throws(block, [error], [message])

// 例一，抛出的错误符合某个构造函数
assert.throws(
  function() {
    throw new Error("Wrong value");
  },
  Error,
  '不符合预期的错误类型'
);

// 例二、抛出错误的提示信息符合正则表达式
assert.throws(
  function() {
    throw new Error("Wrong value");
  },
  /value/,
  '不符合预期的错误类型'
);

// 例三、抛出的错误符合自定义函数的校验
assert.throws(
  function() {
    throw new Error("Wrong value");
  },
  function(err) {
    if ( (err instanceof Error) && /value/.test(err) ) {
      return true;
    }
  },
  '不符合预期的错误类型'
);
```

## assert.doesNotThrow()

doesNotThrow 方法与 throws 方法正好相反，预期某个代码块不抛出错误。

```
// 格式
assert.doesNotThrow(block, [message])

// 用法
assert.doesNotThrow(
  function() {
    console.log("Nothing to see here");
  },
  '预期不抛出错误'
);
```

## assert.ifError()

ifError 方法断言某个表达式是否 false，如果该表达式对应的布尔值等于 true，就抛出一个错误。它对于验证回调函数的第一个参数十分有用，如果该参数为 true，就表示有错误。

```
// 格式
assert.ifError(value)

// 用法
function sayHello(name, callback) {
  var error = false;
  var str   = "Hello "+name;
  callback(error, str);
}

// use the function
sayHello('World', function(err, value){
  assert.ifError(err);
  // ...
})
```

## assert.fail()

fail 方法用于抛出一个错误。

```
// 格式
assert.fail(actual, expected, message, operator)

// 例子
var assert = require('assert');

assert.fail(21, 42, 'Test Failed', '###')
// AssertionError: Test Failed
assert.fail(21, 21, 'Test Failed', '###')
// AssertionError: Test Failed
assert.fail(21, 42, undefined, '###')
// AssertionError: 21 ### 42
```

该方法共有四个参数，但是不管参数是什么值，它总是抛出一个错误。如果 message 参数对应的布尔值不为 false，抛出的错误信息就是 message，否则错误信息就是“实际值 + 分隔符 + 预期值”。