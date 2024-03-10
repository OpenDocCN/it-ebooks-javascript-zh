## 二进制和八进制表示法

ES6 提供了二进制和八进制数值的新的写法，分别用前缀 0b 和 0o 表示。

```
      0b111110111 === 503 // true
0o767 === 503 // true

```

八进制不再允许使用前缀 0 表示，而改为使用前缀 0o。

```
      011 === 9 // 不正确
0o11 === 9 // 正确

```

## Number.isFinite(), Number.isNaN()

ES6 在 Number 对象上，新提供了 Number.isFinite()和 Number.isNaN()两个方法，用来检查 Infinite 和 NaN 这两个特殊值。

Number.isFinite()用来检查一个数值是否非无穷（infinity）。

```
      Number.isFinite(15); // true
Number.isFinite(0.8); // true
Number.isFinite(NaN); // false
Number.isFinite(Infinity); // false
Number.isFinite(-Infinity); // false
Number.isFinite("foo"); // false
Number.isFinite("15"); // false
Number.isFinite(true); // false

```

ES5 通过下面的代码，部署 Number.isFinite 方法。

```
      (function (global) {
  var global_isFinite = global.isFinite;

  Object.defineProperty(Number, 'isFinite', {
    value: function isFinite(value) {
      return typeof value === 'number' && global_isFinite(value);
    },
    configurable: true,
    enumerable: false,
    writable: true
  });
})(this);

```

Number.isNaN()用来检查一个值是否为 NaN。

```
      Number.isNaN(NaN); // true
Number.isNaN(15); // false
Number.isNaN("15"); // false
Number.isNaN(true); // false

```

ES5 通过下面的代码，部署 Number.isNaN()。

```
      (function (global) {
  var global_isNaN = global.isNaN;

  Object.defineProperty(Number, 'isNaN', {
    value: function isNaN(value) {
      return typeof value === 'number' && global_isNaN(value);
    },
    configurable: true,
    enumerable: false,
    writable: true
  });
})(this);

```

它们与传统的全局方法 isFinite()和 isNaN()的区别在于，传统方法先调用 Number()将非数值的值转为数值，再进行判断，而这两个新方法只对数值有效，非数值一律返回 false。

```
      isFinite(25) // true
isFinite("25") // true
Number.isFinite(25) // true
Number.isFinite("25") // false

isNaN(NaN) // true
isNaN("NaN") // true
Number.isNaN(NaN) // true
Number.isNaN("NaN") // false

```

## Number.parseInt(), Number.parseFloat()

ES6 将全局方法 parseInt()和 parseFloat()，移植到 Number 对象上面，行为完全保持不变。

```
      // ES5 的写法
parseInt("12.34") // 12
parseFloat('123.45#') // 123.45

// ES6 的写法
Number.parseInt("12.34") // 12
Number.parseFloat('123.45#') // 123.45

```

这样做的目的，是逐步减少全局性方法，使得语言逐步模块化。

## Number.isInteger()和安全整数

Number.isInteger()用来判断一个值是否为整数。需要注意的是，在 JavaScript 内部，整数和浮点数是同样的储存方法，所以 3 和 3.0 被视为同一个值。

```
      Number.isInteger(25) // true
Number.isInteger(25.0) // true
Number.isInteger(25.1) // false
Number.isInteger("15") // false
Number.isInteger(true) // false

```

ES5 通过下面的代码，部署 Number.isInteger()。

```
      (function (global) {
  var floor = Math.floor,
    isFinite = global.isFinite;

  Object.defineProperty(Number, 'isInteger', {
    value: function isInteger(value) {
      return typeof value === 'number' && isFinite(value) &&
        value > -9007199254740992 && value < 9007199254740992 &&
        floor(value) === value;
    },
    configurable: true,
    enumerable: false,
    writable: true
  });
})(this);

```

JavaScript 能够准确表示的整数范围在-2ˆ53 and 2ˆ53 之间。ES6 引入了 Number.MAX_SAFE_INTEGER 和 Number.MIN_SAFE_INTEGER 这两个常量，用来表示这个范围的上下限。Number.isSafeInteger()则是用来判断一个整数是否落在这个范围之内。

```
      var inside = Number.MAX_SAFE_INTEGER;
var outside = inside + 1;

Number.isInteger(inside) // true
Number.isSafeInteger(inside) // true

Number.isInteger(outside) // true
Number.isSafeInteger(outside) // false

```

## Math 对象的扩展

ES6 在 Math 对象上新增了 17 个与数学相关的方法。所有这些方法都是静态方法，只能在 Math 对象上调用。

### Math.trunc()

Math.trunc 方法用于去除一个数的小数部分，返回整数部分。

```
      Math.trunc(4.1) // 4
Math.trunc(4.9) // 4
Math.trunc(-4.1) // -4
Math.trunc(-4.9) // -4

```

对于空值和无法截取整数的值，返回 NaN。

```
      Math.trunc(NaN);      // NaN
Math.trunc('foo');    // NaN
Math.trunc();         // NaN

```

对于没有部署这个方法的环境，可以用下面的代码模拟。

```
      Math.trunc = Math.trunc || function(x) {
  return x < 0 ? Math.ceil(x) : Math.floor(x);
}

```

### Math.sign()

Math.sign 方法用来判断一个数到底是正数、负数、还是零。

它会返回五种值。

*   参数为正数，返回+1；
*   参数为负数，返回-1；
*   参数为 0，返回 0；
*   参数为-0，返回-0;
*   其他值，返回 NaN。

```
      Math.sign(-5) // -1
Math.sign(5) // +1
Math.sign(0) // +0
Math.sign(-0) // -0
Math.sign(NaN) // NaN
Math.sign('foo'); // NaN
Math.sign();      // NaN

```

对于没有部署这个方法的环境，可以用下面的代码模拟。

```
      Math.sign = Math.sign || function(x) {
  x = +x; // convert to a number
  if (x === 0 || isNaN(x)) {
    return x;
  }
  return x > 0 ? 1 : -1;
}

```

### Math.cbrt()

Math.cbrt 方法用于计算一个数的立方根。

```
      Math.cbrt(-1); // -1
Math.cbrt(0);  // 0
Math.cbrt(1);  // 1
Math.cbrt(2);  // 1.2599210498948734

```

对于没有部署这个方法的环境，可以用下面的代码模拟。

```
      Math.cbrt = Math.cbrt || function(x) {
  var y = Math.pow(Math.abs(x), 1/3);
  return x < 0 ? -y : y;
};

```

### Math.clz32()

JavaScript 的整数使用 32 位二进制形式表示，Math.clz32 方法返回一个数的 32 位无符号整数形式有多少个前导 0。

```
      Math.clz32(0) // 32
Math.clz32(1) // 31
Math.clz32(1000) // 22

```

上面代码中，0 的二进制形式全为 0，所以有 32 个前导 0；1 的二进制形式是 0b1，只占 1 位，所以 32 位之中有 31 个前导 0；1000 的二进制形式是 0b1111101000，一共有 10 位，所以 32 位之中有 22 个前导 0。

对于小数，Math.clz32 方法只考虑整数部分。

```
      Math.clz32(3.2) // 30
Math.clz32(3.9) // 30

```

对于空值或其他类型的值，Math.clz32 方法会将它们先转为数值，然后再计算。

```
      Math.clz32() // 32
Math.clz32(NaN) // 32
Math.clz32(Infinity) // 32
Math.clz32(null) // 32
Math.clz32('foo') // 32
Math.clz32([]) // 32
Math.clz32({}) // 32
Math.clz32(true) // 31

```

### Math.imul()

Math.imul 方法返回两个数以 32 位带符号整数形式相乘的结果，返回的也是一个 32 位的带符号整数。

```
      Math.imul(2, 4);          // 8
Math.imul(-1, 8);         // -8
Math.imul(-2, -2);        // 4

```

如果只考虑最后 32 位（含第一个整数位），大多数情况下，`Math.imul(a, b)`与`a * b`的结果是相同的，即该方法等同于`(a * b)|0`的效果。之所以需要部署这个方法，是因为 JavaScript 有精度限制，超过 2 的 53 次方的值无法精确表示。这就是说，对于那些很大的数的乘法，低位数值往往都是不精确的，Math.imul 方法可以返回正确的低位数值。

```
      (0x7fffffff * 0x7fffffff)|0 // 0

```

上面这个乘法算式，返回结果为 0。但是由于这两个数的个位数都是 1，所以这个结果肯定是不正确的。这个错误就是因为它们的乘积超过了 2 的 53 次方，JavaScript 无法保存额外的精度，就把低位的值都变成了 0。Math.imul 方法可以返回正确的值 1。

```
      Math.imul(0x7fffffff, 0x7fffffff) // 1

```

### Math.fround()

Math.fround 方法返回一个数的单精度浮点数形式。

```
      Math.fround(0);     // 0
Math.fround(1);     // 1
Math.fround(1.337); // 1.3370000123977661
Math.fround(1.5);   // 1.5
Math.fround(NaN);   // NaN

```

对于整数来说，Math.fround 方法返回结果不会有任何不同，区别主要是那些无法用 64 个二进制位精确表示的小数。这时，Math.fround 方法会返回最接近这个小数的单精度浮点数。

对于没有部署这个方法的环境，可以用下面的代码模拟。

```
      Math.fround = Math.fround || function(x) {
  return new Float32Array([x])[0];
};

```

### Math.hypot()

Math.hypot 方法返回所有参数的平方和的平方根。

```
      Math.hypot(3, 4);        // 5
Math.hypot(3, 4, 5);     // 7.0710678118654755
Math.hypot();            // 0
Math.hypot(NaN);         // NaN
Math.hypot(3, 4, 'foo'); // NaN
Math.hypot(3, 4, '5');   // 7.0710678118654755
Math.hypot(-3);          // 3

```

上面代码中，3 的平方加上 4 的平方，等于 5 的平方。

如果参数不是数值，Math.hypot 方法会将其转为数值。只要有一个参数无法转为数值，就会返回 NaN。

### 对数方法

ES6 新增了 4 个对数相关方法。

（1） Math.expm1()

`Math.expm1(x)`返回 ex - 1。

```
      Math.expm1(-1); // -0.6321205588285577
Math.expm1(0);  // 0
Math.expm1(1);  // 1.718281828459045

```

对于没有部署这个方法的环境，可以用下面的代码模拟。

```
      Math.expm1 = Math.expm1 || function(x) {
  return Math.exp(x) - 1;
};

```

（2）Math.log1p()

`Math.log1p(x)`方法返回 1 + x 的自然对数。如果 x 小于-1，返回 NaN。

```
      Math.log1p(1);  // 0.6931471805599453
Math.log1p(0);  // 0
Math.log1p(-1); // -Infinity
Math.log1p(-2); // NaN

```

对于没有部署这个方法的环境，可以用下面的代码模拟。

```
      Math.log1p = Math.log1p || function(x) {
  return Math.log(1 + x);
};

```

（3）Math.log10()

`Math.log10(x)`返回以 10 为底的 x 的对数。如果 x 小于 0，则返回 NaN。

```
      Math.log10(2);      // 0.3010299956639812
Math.log10(1);      // 0
Math.log10(0);      // -Infinity
Math.log10(-2);     // NaN
Math.log10(100000); // 5

```

对于没有部署这个方法的环境，可以用下面的代码模拟。

```
      Math.log10 = Math.log10 || function(x) {
  return Math.log(x) / Math.LN10;
};

```

（4）Math.log2()

`Math.log2(x)`返回以 2 为底的 x 的对数。如果 x 小于 0，则返回 NaN。

```
      Math.log2(3);    // 1.584962500721156
Math.log2(2);    // 1
Math.log2(1);    // 0
Math.log2(0);    // -Infinity
Math.log2(-2);   // NaN
Math.log2(1024); // 10

```

对于没有部署这个方法的环境，可以用下面的代码模拟。

```
      Math.log2 = Math.log2 || function(x) {
  return Math.log(x) / Math.LN2;
};

```

### 三角函数方法

ES6 新增了 6 个三角函数方法。

*   Math.sinh(x) 返回 x 的双曲正弦（hyperbolic sine）
*   Math.cosh(x) 返回 x 的双曲余弦（hyperbolic cosine）
*   Math.tanh(x) 返回 x 的双曲正切（hyperbolic tangent）
*   Math.asinh(x) 返回 x 的反双曲正弦（inverse hyperbolic sine）
*   Math.acosh(x) 返回 x 的反双曲余弦（inverse hyperbolic cosine）
*   Math.atanh(x) 返回 x 的反双曲正切（inverse hyperbolic tangent）