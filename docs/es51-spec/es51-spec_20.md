# 兼容性

## 附加语法

CMAScript 的过去版本中还包含了说明八进制直接量和八进制转义序列的额外语法、语义。在此版本中已将这些附加语法、语义移除。这个非规范性的附录给出与八进制直接量和八进制转义序列一致的语法、语义，以兼容某些较老的 ECMAScript 程序。

### 数字直接量

7.8.3 中的语法、语义可以做如下扩展，但在严格模式代码里不允许做这样的扩展。

语法

`NumericLiteral :: DecimalLiteral HexIntegerLiteral OctalIntegerLiteral` `OctalIntegerLiteral :: 0 OctalDigit OctalIntegerLiteral OctalDigit` `OctalDigit :: 以下之一 0 1 2 3 4 5 6 7`

语义

*   NumericLiteral :: OctalIntegerLiteral 的 MV 是 OctalIntegerLiteral 的 MV。
*   OctalDigit :: 0 的 MV 是 0。
*   OctalDigit :: 1 的 MV 是 1。
*   OctalDigit :: 2 的 MV 是 2。
*   OctalDigit :: 3 的 MV 是 3。
*   OctalDigit :: 4 的 MV 是 4。
*   OctalDigit :: 5 的 MV 是 5。
*   OctalDigit :: 6 的 MV 是 6。
*   OctalDigit :: 7 的 MV 是 7。
*   OctalIntegerLiteral :: 0 OctalDigit 的 MV 是 OctalDigit 的 MV。
*   OctalIntegerLiteral :: OctalIntegerLiteral OctalDigit 的 MV 是 OctalIntegerLiteral 的 MV 乘以 8 再加上 OctalDigit 的 MV。

### 字符串直接量

7.8.4 中的语法、语义可以做如下扩展，但在严格模式代码里不允许做这样的扩展。

语法

`EscapeSequence :: CharacterEscapeSequence OctalEscapeSequence HexEscapeSequence UnicodeEscapeSequence` `OctalEscapeSequence :: OctalDigit [lookahead ? DecimalDigit] ZeroToThree OctalDigit [lookahead ?DecimalDigit] FourToSeven OctalDigit ZeroToThree OctalDigit OctalDigit` `ZeroToThree :: 以下之一 0 1 2 3` `FourToSeven :: 以下之一 4 5 6 7`

语义

*   EscapeSequence :: OctalEscapeSequence 的 CV 是 OctalEscapeSequence 的 CV。
*   OctalEscapeSequence :: OctalDigit [lookahead ? DecimalDigit] 的 CV 是个字符，它的 unicode 代码单元值是 OctalDigit 的 MV 。
*   OctalEscapeSequence :: ZeroToThree OctalDigit [lookahead ? DecimalDigit] 的 CV 是个字符，它的 unicode 代码单元值是 ZeroToThree 的 MV 乘以 8 再加上 OctalDigit 的 MV。
*   OctalEscapeSequence :: FourToSeven OctalDigit 的 CV 是个字符，它的 unicode 代码单元值是 FourToSeven 的 MV 乘以 8 再加上 OctalDigit 的 MV。
*   OctalEscapeSequence :: ZeroToThree OctalDigit OctalDigit 的 CV 是个字符，它的 unicode 代码单元值是 (ZeroToThree 的 MV 乘以 64 (8²) ) 加上 ( 第一个 OctalDigit 的 MV 乘以 8 ) 加上 OctalDigit 的 MV。
*   ZeroToThree :: 0 的 MV 是 0.
*   ZeroToThree :: 1 的 MV 是 1。
*   ZeroToThree :: 2 的 MV 是 2。
*   ZeroToThree :: 3 的 MV 是 3。
*   FourToSeven :: 4 的 MV 是 4。
*   FourToSeven :: 5 的 MV 是 5。
*   FourToSeven :: 6 的 MV 是 6。
*   FourToSeven :: 7 的 MV 是 7。

## 附加属性

一些 ECMAScript 的实现可能会包含一些标准原生对象上的附加属性。这一非标准附录提示了一些这样的属性常见的语义，但是这并不意味着他们和其语义成为标准的一部分。

### escape(string)

escape 函数是全局对象的一个属性。它通过将一些字符替换成十六进制转义序列，计算出一个新字符串值。

对于代码单元小于等于 0xFF 的被替换字符，使用 %xx 格式的两位数转义序列。对于代码单元大于 0xFF 的被替换字符，使用 %uxxxx 格式的四位数转义序列。

用一个参数 string 调用 escape 函数，采用以下步骤：

1.  调用 ToString(string)。
2.  计算 Result(1) 的字符数。
3.  令 R 为空字符串。
4.  令 k 为 0。
5.  如果 k 等于 Result(2), 返回 R。
6.  获得 Result(1) 中 k 位置的字符（表示为 16 位无符号整数）。
7.  如果 Result(6) 是 69 个非空字符 "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789@*_+-./" 之一，则转到步骤 13。
8.  如果 Result(6) 小于 256，则转到步骤 11。
9.  令 S 为包含六个字符 "%u wxyz" 的字符串，其中 wxyz 是用四个十六进制数字编码的 Result(6) 值。
10.  转到步骤 14。
11.  令 S 为包含三个字符 "% xy" 的字符串，其中 xy 是用两个十六进制数字编码的 Result(6) 值。
12.  转到步骤 14。
13.  令 S 为包含单个字符 Result(6) 的字符串。
14.  令 R 为将之前的 R 和 S 值连起来组成的新字符串。
15.  k 递增 1。
16.  转到步骤 5。

这里的编码方式有部分是基于 [RFC 1738](http://tools.ietf.org/html/rfc1738) 描述的编码方式, 但本标准规定的完整编码方式只有上面描述的这些，不考虑 [RFC 1738](http://tools.ietf.org/html/rfc1738) 中的内容。 此编码方式并没有反映出从 [RFC 1738](http://tools.ietf.org/html/rfc1738) 到 [RFC 3986](http://tools.ietf.org/html/rfc3986) 的变化。

### unescape(string)

unescape 函数是全局对象的一个属性。它通过将每个可能是 escape 函数导入的转义序列，分别替换成代表这些转义序列的字符， 计算出一个新字符串值。

用一个参数 string 调用 unescape 函数，采用以下步骤：

1.  调用 ToString(string)。
2.  计算 Result(1) 的字符数。
3.  令 R 为空字符串。
4.  令 k 为 0。
5.  如果 k 等于 Result(2)，返回 R。
6.  令 c 为 Result(1) 中 k 位置的字符。
7.  如果 c 不是 % ，转到步骤 18。
8.  如果 k 大于 Result(2)?6，转到步骤 14。
9.  如果 Result(1) 中 k+1 位置的字符不是 u，转到步骤 14。
10.  如果 Result(1) 中分别在 k+2，k+3，k+4，k+5 位置的四个字符不全是十六进制数字，转到步骤 14。
11.  令 c 为一个字符，它的代码单元值是 Result(1) 中 k+2，k+3，k+4，k+5 位置的四个十六进制数字代表的整数。
12.  k 递增 5。
13.  转到步骤 18。
14.  如果 k 大于 Result(2)?3，转到步骤 18。
15.  如果 Result(1) 中分别在 k+1，k+2 位置的两个字符不都是十六进制数字，转到步骤 18。
16.  令 c 为一个字符，它的代码单元值是两个零加上 Result(1) 中 k+1，k+2 位置的两个十六进制数字代表的整数。
17.  k 递增 2。
18.  令 R 为将之前的 R 和 c 值连起来组成的新字符串。
19.  k 递增 1。
20.  转到步骤 5。

### String.prototype.substr(start, length)

substr 方法有两个参数 start 和 length，将 this 对象转换为一个字符串，并返回这个字符串中从 start 位置一直到 length 位置（或如果 length 是 undefined，就一直到字符串结束位置）的字符组成的子串。如果 start 是负数，那么它就被当作是 (sourceLength+start)，这里的 sourceLength 是字符串的长度。返回结果是一个字符串值，不是 String 对象。采用以下步骤：

1.  将 this 值作为参数调用 ToString。
2.  调用 ToInteger(start)。
3.  如果 length 是 undefined，就用 +∞；否则调用 ToInteger(length)。
4.  计算 Result(1) 的字符数。
5.  如果 Result(2) 是正数或零，就用 Result(2)；否则使用 max(Result(4)+Result(2),0)。
6.  计算 min(max(Result(3),0), Result(4)–Result(5))。
7.  如果 Result(6) ≤ 0, 返回空字符串 "" 。
8.  返回一个由 Result(1) 中的 Result(5) 位置的字符开始的连续的 Result(6) 个字符组成的字符串。

substr 方法的 length 属性是 2。

substr 函数被刻意设计成通用的；它并不要求其 this 值为字符串对象。因此它可以作为方法转移到其他种类的对象中。

### Date.prototype.getYear( )

对于近乎所有用途，getFullYear 方法都是首选的，因为它避免了“2000 年问题”。

无参数方式调用 getYear 方法，采用以下步骤：

1.  令 t 为 this 时间值。
2.  如果 t 是 NaN, 返回 NaN。
3.  返回 YearFromTime(LocalTime(t)) ? 1900。

### Date.prototype.setYear(year)

对于近乎所有用途， setFullYear 方法都是首选的，因为它避免了“2000 年问题”。

用一个参数 year 调用 setYear 方法，采用以下步骤：

1.  令 t 为 LocalTime(this 时间值) 的结果；但如果 this 时间值是 NaN，那么令 t 为 +0。
2.  调用 ToNumber(year)。
3.  如果 Result(2) 是 NaN，将 this 值的 [[PrimitiveValue]] 内部属性设为 NaN，并返回 NaN。
4.  如果 Result(2) 不是 NaN 并且 0 ≤ ToInteger(Result(2)) ≤ 99 ，则 Result(4) 是 ToInteger(Result(2)) + 1900。否则，Result(4) 是 Result(2)。
5.  计算 MakeDay(Result(4), MonthFromTime(t), DateFromTime(t))。
6.  计算 UTC(MakeDate(Result(5), TimeWithinDay(t)))。
7.  将 this 值的 [[PrimitiveValue]] 内部属性设为 TimeClip(Result(6))。
8.  返回 this 值的 [[PrimitiveValue]] 内部属性值。

### Date.prototype.toGMTString( )

toUTCString 属性是首选的，toGMTString 属性是为了兼容较老的代码才提供的。建议在新的 ECMAScript 代码中使用 toUTCString 属性。

Date.prototype.toGMTString 的初始值是与 Date.prototype.toUTCString 的初始值相同的函数对象。