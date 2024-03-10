# 类型转换与测试

ECMAScript 运行时系统会在需要时从事自动类型转换。为了阐明某些结构的语义，定义一集转换运算符是很有用的。这些运算符不是语言的一部分；在这里定义它们是为了协助语言语义的规范。转换运算符是多态的 — 它们可以接受任何 ECMAScript 语言类型 的值，但是不接受 规范类型 。

## ToPrimitive

ToPrimitive 运算符接受一个值，和一个可选的 `期望类型` 作参数。ToPrimitive 运算符把其值参数转换为非对象类型。如果对象有能力被转换为不止一种原语类型，可以使用可选的 `期望类型` 来暗示那个类型。根据下表完成转换：

ToPrimitive 转换

| 输入类型 | 结果 |
| Undefined | 结果等于输入的参数（不转换）。 |
| Null | 结果等于输入的参数（不转换）。 |
| Boolean | 结果等于输入的参数（不转换）。 |
| Number | 结果等于输入的参数（不转换）。 |
| String | 结果等于输入的参数（不转换）。 |
| Object | 返回该对象的默认值。（调用该对象的内部方法[[DefaultValue]]一樣）。 |

## ToBoolean

ToBoolean 运算符根据下表将其参数转换为布尔值类型的值：

ToBoolean 转换

| 输入类型 | 结果 |
| Undefined | false |
| Null | false |
| Boolean | 结果等于输入的参数（不转换）。 |
| Number | 如果参数是 +0, -0, 或 NaN，结果为 false ；否则结果为 true。 |
| String | 如果参数参数是空字符串（其长度为零），结果为 false，否则结果为 true。 |
| Object | true |

## ToNumber

ToNumber 运算符根据下表将其参数转换为数值类型的值：

ToNumber 转换

| 输入类型 | 结果 |
| Undefined | NaN |
| Null | +0 |
| Boolean | 如果参数是 true，结果为 1。如果参数是 false，此结果为 +0。 |
| Number | 结果等于输入的参数（不转换）。 |
| String | 参见下文的文法和注释。 |
| Object | 应用下列步骤： 
1.  設 `原始值` 為 `ToPrimitive` ( `输入参数` , 暗示 `数值类型`)。
2.  返回 `ToNumber` ( `原始值` )。

 |

### 对字符串类型应用 ToNumber

对字符串应用 ToNumber 时，对输入字符串应用如下文法。如果此文法无法将字符串解释为「字符串数值常量」的扩展，那么 ToNumber 的结果为 NaN。

语法

`StringNumericLiteral ::: StrWhiteSpaceopt StrWhiteSpaceoptStrNumericLiteral StrWhiteSpaceopt` `StrWhiteSpace ::: StrWhiteSpaceChar StrWhiteSpaceopt` `StrWhiteSpaceChar ::: WhiteSpace LineTerminator``StrNumericLiteral ::: StrDecimalLiteral HexIntegerLiteral``StrDecimalLiteral ::: StrUnsignedDecimalLiteral + StrUnsignedDecimalLiteral - StrUnsignedDecimalLiteral``StrUnsignedDecimalLiteral ::: Infinity DecimalDigits . DecimalDigitsopt ExponentPartopt . DecimalDigits ExponentPartopt DecimalDigits ExponentPartopt``DecimalDigits ::: DecimalDigit DecimalDigits DecimalDigit` `DecimalDigit ::: 以下之一 0 1 2 3 4 5 6 7 8 9` `ExponentPart ::: ExponentIndicator SignedInteger` `ExponentIndicator ::: 以下之一 e E` `SignedInteger ::: DecimalDigits + DecimalDigits - DecimalDigits``HexIntegerLiteral ::: 0x HexDigit 0X HexDigit HexIntegerLiteral HexDigit` `HexDigit ::: 以下之一 0 1 2 3 4 5 6 7 8 9 a b c d e f A B C D E F`

需要注意到「字符串数值常量」和 「数值常量」 语法上的不同：

*   「字符串数值常量」之前和、或之后可以有空白和／或行结束符。
*   十进制的「字符串数值常量」可有任意位数的 0 在前头。
*   十进制的「字符串数值常量」可有指示其符号的 + 或 - 前缀。
*   空的，或只包含空白的「字符串值常量」會被转换为 +0。

字符串到数字值的转换，大体上类似于判定 数值常量 的数字值，不过有些细节上的不同，所以，这里给出了把字符串数值常量转换为数值类型的值的全部过程。这个值分两步来判定：首先，从字符串数值常量中导出数学值；第二步，以下面所描述的方式对该数学值进行舍入。

*   「字符串整数常量 ::: [empty]」的数学值是 0。
*   「字符串整数常量 ::: 串空白」的数学值是 0。
*   不管有没有空白「字符串整数常量 ::: 串空白 [opt] 串数值常量 串空白 [opt]」的数学值是「串数值常量」的数学值
*   「串数值常量 ::: 串十进制常量」的数学值是「串十进制常量」的数学值
*   「串数值常量 ::: 十六进制整数常量」的数学值是「十六进制整数常量」的数学值
*   「串十进制常量 ::: 串无符号整数常量」的数学值是「串无符号整数常量」的数学值
*   「串十进制常量 ::: + 串无符号整数常量」的数学值是「串无符号整数常量」的数学值。
*   「串十进制常量 ::: - 串无符号整数常量」的数学值是「串无符号整数常量」的数学值的负数。 （需要注意的是，如果「串无符号整数常量」的数学值是 0, 其负数也是 0。下面中描述的舍入规则会合适地处理小于数学零到浮点数 +0 或 -0 的变换。）
*   「串无符号整数常量 ::: Infinity」的数学值是 10¹⁰⁰⁰⁰（一个大到会舍入为 +∞ 的值过大的值会返回为 ）。
*   「串无符号整数常量 ::: 十进制数 .」的数学值是「十进制数」的数学值。
*   「串无符号整数常量 ::: 十进制数 . 十进制数」的数学值是第一个「十进制数」的数学值加（第二个「十进制数」的数学值乘以 10^(-`n`)），这里的 `n` 是 the number of characters in the 第二个「十进制数」字符数。
*   「串无符号整数常量 ::: 十进制数 . 指数部分」的数学值是「十进制数」的数学值乘以 10^(`e`), 这里的 `e` 是「指数部分」的数学值
*   「串无符号整数常量 ::: 十进制数 . 十进制数 指数部分」的数学值是（第一个「十进制数」的数学值加（第二个「十进制数」的数学值乘以 10^(-`n`)））乘以 10^e，这里的 `n` 是 第二个「十进制数」中的字符个数，`e` 是「指数部分」的数学值。
*   「串无符号整数常量 ::: . 十进制数」的数学值是「十进制数」的数学值乘以 10^(-`n`)，这里的 `n` 是「十进制数」中的字符个数。
*   「串无符号整数常量 ::: . 十进制数 指数部分」的数学值是「十进制数」的数学值乘以 10^(`e`-`n`)，这里的 `n` 是「十进制数」中的字符个数，`e` 是「指数部分」的数学值
*   「串无符号整数常量 ::: 十进制数」的数学值是「十进制数」的数学值
*   「串无符号整数常量 ::: 十进制数 指数部分」的数学值是「十进制数」的数学值乘以 10^(`e`)，这里的 `e` 是「指数部分」的数学值
*   「十进制数 ::: 十进制数字」是「十进制数字」的数学值
*   「十进制数 ::: 十进制数 十进制数字」的数学值是（「十进制数」的数学值乘以 10）加「十进制数字」的数学值
*   「指数部分 ::: 幂指示符 有符号整数」的数学值是「有符号整数」的数学值
*   「有符号整数 ::: 十进制数」的数学值是「十进制数」的数学值
*   「有符号整数 ::: + 十进制数」的数学值是「十进制数」的数学值
*   「有符号整数 ::: - 十进制数」是「十进制数」的数学值的负数。
*   「十进制数字 ::: 0」或「十六进制数字 ::: 0」的数学值是 0。
*   「十进制数字 ::: 1」或「十六进制数字 ::: 1」的数学值是 1。
*   「十进制数字 ::: 2」或「十六进制数字 ::: 2」的数学值是 2。
*   「十进制数字 ::: 3」或「十六进制数字 ::: 3」的数学值是 3。
*   「十进制数字 ::: 4」或「十六进制数字 ::: 4」的数学值是 4。
*   「十进制数字 ::: 5」或「十六进制数字 ::: 5」的数学值是 5。
*   「十进制数字 ::: 6」或「十六进制数字 ::: 6」的数学值是 6。
*   「十进制数字 ::: 7」或「十六进制数字 ::: 7」的数学值是 7。
*   「十进制数字 ::: 8」或「十六进制数字 ::: 8」的数学值是 8。
*   「十进制数字 ::: 9」或「十六进制数字 ::: 9」的数学值是 9。
*   「十六进制数字 ::: a」或「十六进制数字 ::: A」的数学值是 10。
*   「十六进制数字 ::: b」或「十六进制数字 ::: B」的数学值是 11。
*   「十六进制数字 ::: c」或「十六进制数字 ::: C」的数学值是 12。
*   「十六进制数字 ::: d」或「十六进制数字 ::: D」的数学值是 13。
*   「十六进制数字 ::: e」或「十六进制数字 ::: E」的数学值是 14。
*   「十六进制数字 ::: f」或「十六进制数字 ::: F」的数学值是 15。
*   「十六进制整数常量 ::: 0x 十六进制数字」的数学值是「十六进制数字」的数学值。
*   「十六进制整数常量 ::: 0X 十六进制数字」的数学值是「十六进制数字」的数学值。
*   「十六进制整数常量 ::: 十六进制整数常量 十六进制数字」的数学值是（「十六进制整数常量」的数学值乘以 16）加「十六进制数字」的数学值。

一旦字符串数值常量的数学值被精确地确定，接下来就会被舍入为数值类型的一个值。如果数学值是 0，那么舍入值为 +0，除非字符串数值常量中第一个非空白字符是 ‘-’ — 在这种情况下，舍入值为 -0。否则，舍入值必须是数学值的 数字值 ，除非该常量包括一个「串无符号十进制常量」，且此常量多于 20 位 重要数字 — 在这种情况下，此数字的值是下面两种之一：一是将其 20 位之后的每个重要数字用 0 替换，产生此字符串解析出的数学值的数字值；二是将其 20 位之后的每个有效数字用 0 替换，并在第 20 位重要数字加一，产生此字符串解析出的数学值的数字值 ![](img/Question.png)。判断一个数字是否为 重要数字 ，首先它不能是「指数部分」的一部分，且

*   它不是 0；或
*   它的左边是一个非零值，右边是一个不在「指数部分」中的非零值。

## ToInteger

ToInteger 运算符将其参数转换为整数值。此运算符功能如下所示：

1.  对输入参数调用 `ToNumber` 。
2.  如果 Result(1) 是 NaN，返回 +0。
3.  如果 Result(1) 是 +0 ，-0，+∞，或 -∞，返回 Result(1)。
4.  计算 sign(Result(1)) * floor(abs(Result(1)))。
5.  返回 Result(4)。

## ToInt32：（32 位有符号整数）

ToInt32 运算符将其在 -2³¹ 到 2³¹-1 闭区间内的参数转换为 2³² 个整数值之一。此运算符功能如下所示：

1.  对输入参数调用 `ToNumber` 。
2.  如果 Result(1) 是 +0 ，-0，+∞，或 -∞，返回 +0。
3.  计算 sign(Result(1)) * floor(abs(Result(1)))。
4.  计算 Result(3) modulo 2³² ；也就是说，数值类型的有限整数值 k 为正，且小于 2³² ，规模相对于 Result(3) 的数学值差异 ，2³² 是 k 的整数倍。
5.  如果 Result(4) 是大于等于 2³¹ 的整数，返回 Result(4) - 2³² ，否则返回 Result(4)。

按照 ToInt32 以上的定义：ToInt32 抽象操作是幂等的：如果作用于它产生的结果，第二次程序将会保持值不变，对任何值 x，ToInt32(ToUint32(x))都与 ToInt32(x)相等。（将+∞和?∞映射到+0 就是为了保持这一特性。） ToInt32 将-0 映射为+0。

## ToUint32：（32 位无符号整数）

ToUint32 运算符将其在 0 到 2³²-1 闭区间内的参数转换为 2³² 个整数值之一。此运算符功能如下所示：

1.  对输入参数调用 `ToNumber` 。
2.  如果 Result(1) 是 +0 ，-0，+∞，或 -∞，返回 +0。
3.  计算 sign(Result(1)) * floor(abs(Result(1)))。
4.  计算 Result(3) modulo 2³² ；也就是说，数值类型的有限整数值 k 为正，且小于 2³² ，规模相对于 Result(3) 的数学值差异 ，2³² 是 k 的整数倍。
5.  返回 Result(4)。

上面给出的 ToUint32 的定义中：

*   ToUint32 和 `ToInt32` 唯一的不同在于第 5 步。
*   ToUint32 的操作具有鉴一性：如果应用于一个已经产生的结果，第二次应用保持值不变。
*   对于 x 的所有值，ToUint32( `ToInt32` (x)) 与 ToUint32 相等。（这是为了保证后来的属性 +∞ 和 -∞ 被映射为 +0。）
*   ToUint32 把 -0 映射为 +0。

## ToUint16：（16 位无符号整数）

ToUint32 运算符将其在 0 到 2¹⁶-1 闭区间内的参数转换为 2¹⁶ 个整数值之一。此运算符功能如下所示：

1.  对输入参数调用 `ToNumber` 。
2.  如果 Result(1) 是 +0 ，-0，+∞，或 -∞，返回 +0。
3.  计算 sign(Result(1)) * floor(abs(Result(1)))。
4.  计算 Result(3) modulo 2¹⁶ ；也就是说，数值类型的有限整数值 k 为正，且小于 2¹⁶ ，规模相对于 Result(3) 的数学值差异 ，2¹⁶ 是 k 的整数倍。
5.  返回 Result(4)。

上面给出的 ToUint16 的定义中：

*   `ToUint32` 和 ToUint16 之间唯一的不同是第 4 步中，2¹⁶ 代替了 2³²。
*   `ToUint32` 把 -0 映射为 +0。

## ToString

ToString 运算符根据下表将其参数转换为字符串类型的值：

ToString 转换

| 输入类型 | 结果 |
| Undefined | "undefined" |
| Null | "null" |
| Boolean | 如果参数是 true，那么结果为 "true"。 如果参数是 false，那么结果为 "false"。 |
| Number | 结果等于输入的参数（不转换）。 |
| String | 参见下文的文法和注释。 |
| Object | 应用下列步骤： 
1.  调用 `ToPrimitive` ( `输入参数` , 暗示 `字符串类型`)。
2.  调用 ToString(Result(1))。
3.  返回 Result(2)。

 |

### 对数值类型应用 ToString

ToString 运算符将数字 `m` 转换为字符串格式的给出如下所示：

1.  如果 `m` 是 NaN，返回字符串 "NaN"。
2.  如果 `m` 是 +0 或 -0，返回字符串 "0"。
3.  如果 `m` 小于零，返回连接 "-" 和 `ToString` (-`m`) 的字符串。
4.  如果 `m` 无限大，返回字符串 "Infinity"。
5.  否则，令 `n`, `k`, 和 `s` 是整数，使得 `k` ≥ 1, 10^(`k`-1) ≤ `s` < 10^(`k`)，`s` × 10^(`n`-`k`) 的数字值是 `m`，且 `k` 足够小。要注意的是，`k` 是 `s` 在十进制表示中的数字的个数。`s` 不被 10 整除，且`s` 的至少要求的有效数字位数不一定要被这些标准唯一确定。
6.  如果 `k` ≤ `n` ≤ 21，返回由 `k` 个 `s` 在十进制表示中的数字组成的字符串（有序的，开头没有零），后面跟随字符 '0' 的 `n`-`k` 次出现。
7.  如果 0 < `n` ≤ 21，返回由 `s` 在十进制表示中的、最多 `n` 个有效数字组成的字符串，后面跟随一个小数点 '. '，再后面是余下的 `k`-`n` 个 `s` 在十进制表示中的数字。
8.  如果 -6 < `n` ≤ 0，返回由字符 '0' 组成的字符串，后面跟随一个小数点 '. '，再后面是字符 '0' 的 -`n` 次出现，再往后是 `k` 个 `s` 在十进制表示中的数字。
9.  否则，如果 `k` = 1，返回由单个数字 `s` 组成的字符串，后面跟随小写字母 'e'，根据 `n`-1 是正或负，再后面是一个加号 '+' 或减号 '-' ，再往后是整数 abs(`n`-1) 的十进制表示（没有前置的零）。
10.  返回由 `s` 在十进制表示中的、最多的有效数字组成的字符串，后面跟随一个小数点 '. '，再后面是余下的是 `k`-1 个 `s` 在十进制表示中的数字，再往后是小写字母 'e'，根据`n`-1 是正或负，再后面是一个加号 '+ ' 或减号 '-' ，再往后是整数 abs(`n`-1) 的十进制表示（没有前置的零）。

下面的评语可能对指导实现有用，但不是本标准的常规要求。

*   如果 x 是除 -0 以外的任一数字值，那么 `ToNumber` ( `ToString` (x)) 与 x 是完全相同的数字值。
*   s 至少要求的有效数字位数并非总是由步骤 5 中所列的要求唯一确定。

对于那些提供了比上面的规则所要求的更精确的转换的实现，我们推荐下面这个步骤 5 的可选版本，作为指导：

否则，令 `n`, `k`, 和 `s` 是整数，使得 `k` ≥ 1, 10^(`k`-1) ≤ `s` < 10^(`k`)，`s` × 10^(`n`-`k`) 的数字值是 `m`，且 `k` 足够小。如果有数倍于 `s` 的可能性，选择 `s` × 10^(`n`-`k`) 最接近于 `m` 的值作为 `s` 的值。如果 `s`有两个这样可能的值，选择是偶数的那个。要注意的是，`k` 是 `s` 在十进制表示中的数字的个数，且 `s` 不被 10 整除。

ECMAScript 的实现者们可能会发现，David M 所写的关于浮点数进行二进制到十进制转换方面的文章和代码很有用：

Gay, David M. Correctly Rounded Binary-Decimal and Decimal-Binary Conversions. Numerical Analysis Manuscript 90-10\. AT&T Bell Laboratories (Murray Hill, New Jersey). November 30, 1990\. 在这里取得 http://cm.bell-labs.com/cm/cs/doc/90/4-10.ps.gz 。有关的代码在这里 http://cm.bell-labs.com/netlib/fp/dtoa.c.gz 还有 http://cm.bell-labs.com/netlib/fp/g_fmt.c.gz 。这些都可在众多的 netlib 镜像站点上找到。

## ToObject

ToObject 运算符根据下表将其参数转换为对象类型的值：

ToObject 转换

| 输入类型 | 结果 |
| Undefined | 抛出 TypeError 异常。 |
| Null | >抛出 TypeError 异常。 |
| Boolean | 创建一个新的 Boolean 对象，其 [[PrimitiveValue]]属性被设为该布尔值的值。 |
| Number | 创建一个新的 Number 对象，其[[PrimitiveValue]]属性被设为该布尔值的值。 |
| String | 创建一个新的 String 对象，其 [[PrimitiveValue]] 属性被设为该布尔值的值。 |
| Object | 结果是输入的参数（不转换）。 |

## CheckObjectCoercible

根据表 15 定义，抽象操作 CheckObjectCoercible 在其参数无法用 ToObject 转换成对象的情况下抛出一个异常：

CheckObjectCoercible 结果

| 输入类型 | 结果 |
| Undefined | 抛出一个 TypeError 异常 |
| Null | >抛出 TypeError 异常。 |
| Boolean | 返回 |
| Number | 返回 |
| String | 返回 |
| Object | 返回 |

## IsCallable

根据表 16，抽象操作 IsCallable 确定其必须是 ECMAScript 语言值的参数是否是一个可调用对象：

IsCallable 结果

| 输入类型 | 结果 |
| Undefined | 返回 false |
| Null | >返回 false |
| Boolean | 返回 false |
| Number | 返回 false |
| String | 返回 false |
| Object | 如果参数对象包含一个 Call 内部方法，则返回 true，否则返回 false |

## SameValue 算法

内部严格比较操作 SameValue(`x`,`y`)，`x` 和 `y` 为 ECMAScript 语言中的值，需要产出 true 或 false![](img/Note.png)。比较过程如下：

1.  如果 `Type(`x`)` 与 `Type(`y`)` 的结果不一致，返回 false，否则
2.  如果 `Type(`x`)` 结果为 Undefined，返回 true
3.  如果 `Type(`x`)` 结果为 Null，返回 true
4.  如果 `Type(`x`)` 结果为 Number，则
    1.  如果 `x` 为 NaN，且 `y` 也为 NaN，返回 true
    2.  如果 `x` 为 +0，`y` 为 -0，返回 false
    3.  如果 `x` 为 -0，`y` 为 +0，返回 false
    4.  如果 `x` 与 `y` 为同一个数字，返回 true
    5.  返回 false
5.  如果 `Type(`x`)` 结果为 String，如果 `x` 与 `y` 为完全相同的字符序列（相同的长度和相同的字符对应相同的位置），返回 true，否则，返回 false
6.  如果 `Type(`x`)` 结果为 Boolean，如果 `x` 与 `y` 都为 true 或 false，则返回 true，否则，返回 false
7.  如果 `x` 和 `y` 引用到同一个 Object 对象，返回 true，否则，返回 false