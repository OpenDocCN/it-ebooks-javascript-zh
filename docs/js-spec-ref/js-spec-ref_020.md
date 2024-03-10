# 3.5 String 对象

*   概述
*   String.fromCharCode()

## 概述

String 对象是 JavaScript 原生提供的三个包装对象之一，用来生成字符串的包装对象实例。

```
var s = new String("abc");

typeof s // "object"
s.valueOf() // "abc"
```

上面代码生成的变量 s，就是 String 对象的实例，类型为对象，值为原来的字符串。实际上，String 对象的实例是一个类似数组的对象。

```
new String("abc")
// String {0: "a", 1: "b", 2: "c"}
```

除了用作构造函数，String 还可以当作工具方法使用，将任意类型的值转为字符串。

```
String(true) // "true"
String(5) // "5"
```

上面代码将布尔值 ture 和数值 5，分别转换为字符串。

## String.fromCharCode()

String 对象直接提供的方法，主要是 fromCharCode()。该方法根据 Unicode 编码，生成一个字符串。

```
String.fromCharCode(104, 101, 108, 108, 111)
// "hello"
```

注意，该方法不支持编号大于 0xFFFF 的字符。

```
String.fromCharCode(0x20BB7)
// "ஷ"
```

上面代码返回字符的编号是 0x0BB7，而不是 0x20BB7。这种情况下，只能使用四字节的 UTF-16 编号，得到正确结果。

```
 String.fromCharCode(0xD842, 0xDFB7)
// "
```