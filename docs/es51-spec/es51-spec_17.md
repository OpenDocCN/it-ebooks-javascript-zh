# 标准 ECMAScript 内置对象

ECMAScript 代码运行时会有一些可用的内置对象。一是作为执行程序词法环境的一部分的全局对象。其他的可通过全局对象的初始属性访问。

除非另外指明，如果内置对象拥有 [[Call]] 内部属性，那么它的 [[Class]] 内部属性是 "Function"，如果没有 [[Call]] 内部属性，那么它的 [[Class]] 内部属性是 "Object"。除非另外指明，内置对象的 [[Extensible]] 内部属性的初始值是 true。

许多内置对象是函数：它们可以通过参数调用。其中有些还作为构造器：这些函数可被 new 运算符调用。对于每个内置函数，本规范描述了这些函数的必须参数和 Function 对象的属性。对于每个内置构造器，本规范还描述了这些构造器的 prototype 对象的属性，还描述了用 new 表达式调用这个构造器后返回的具体实例对象的属性。

除非另外指明了某一特定函数的描述，如果在调用本章中描述的函数或构造器时传入的参数少于必须的参数个数，那么这些函数或构造器将表现为仿佛传入了足够的参数，而那些缺少的参数会设定为 undefined 值。

除非另外指明了某一特定函数的描述，如果在调用本章中描述的函数或构造器时传入了比函数指定允许的更多的参数时，额外的参数会被函数忽略。然而，一个实现可以为这样的参数列表定义依赖于实现的特别行为，只要这种行为在单纯添加额外参数时不抛出 TypeError 异常。

实现为了给内置函数集合增添一些额外功能而添加新函数是被鼓励的，而不是为现有函数增加新参数。

每个内置函数和每个内置构造器都有 Function 原型对象 ，Function.prototype（15.3.4）表达式的初始值作为其 [[Prototype]] 内部属性的值。

除非另外指明，每个内置的原型对象都有 Object 原型对象 ，Object.prototype(15.2.4) 表达式的初始值作为其 [[Prototype]] 内部属性的值，除了 Object 的原型对象自身。

除非另外指明了特定函数的描述，否则本章描述的内置函数中不存在不是构造器而要实现 [[Construct]] 内部方法的内置函数。除非另外指明了特定函数的描述，否则本章描述的内置函数都没有 prototype 属性。

本章通常描述构造器的“作为函数调用”和“用 new 表达式调用” 有不同行为。" 作为函数调用 " 的行为对应于调用构造器的 [[Call]] 内部方法，“用 new 表达式调用”的行为对应于调用构造器的 [[Construct]] 内部方法。

本章描述的每个内置 Function 对象 -- 不管是构造器还是普通函数，或二者都是 -- 拥有一个 length 属性，其值是个整数。除非另外指明，此值等于显示在函数描述的子章节标题的形式参数的个数，包括可选参数。

例如描述 String 的 prototype 对象的 slice 属性初始值的函数对象的子章节标题是“String.prototype.slice (start, end)”，这说明有两个形参 start 和 end，所以这个函数对象的 length 属性值是 2。

任何情况下，本章描述的内置函数对象的 length 属性拥有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。除非另外指明，本章描述的所有其他属性拥有特性 { [[Writable]]: true, [[Enumerable]]: false, [[Configurable]]: true }。

## 全局对象

唯一的全局对象建立在控制进入任何执行环境之前。

除非另外指明，全局对象的标准内置属性拥有特性 {[[Writable]]: true, [[Enumerable]]: false, [[Configurable]]: true}。

全局对象没有 [[Construct]] 内部属性 ; 全局对象不可能当做构造器用 new 运算符调用。

全局对象没有 [[Call]] 内部属性，全局对象不可能当做函数来调用。

全局对象的 [[Prototype]] 和 [[Class]] 内部属性值是依赖于实现的。

除了本规范定义的属性之外，全局对象还可以拥有额外的宿主定义的属性。全局对象可包含一个值是全局对象自身的属性；例如，在 HTML 文档对象模型中全局对象的 window 属性是全局对象自身。

### 全局对象的值属性

#### NaN

NaN 的值是 NaN（见 8.5）。这个属性拥有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

#### Infinity

Infinity 的值是 +∞（见 8.5）。这个属性拥有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

#### undefined

undefined 的值是 undefined（见 8.1）。这个属性拥有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

### 全局对象的函数属性

#### eval (x)

当用一个参数 x 调用 eval 函数，采用如下步骤：

1.  如果 Type(x) 不是 String, 返回 x.
2.  令 prog 为 ECMAScript 代码，它是将 x 作为一个程序解析的结果。如果解析失败，抛出一个 SyntaxError 异常 ( 见 16 章 )。
3.  令 evalCtx 为给 eval 代码 prog 建立的新执行环境 (10.4.2)。
4.  令 result 为解释执行程序 prog 的结果。
5.  退出执行环境 evalCtx, 恢复到之前的执行环境。
6.  如果 result.type 是 normal 并且其完结类型值是 V, 则返回 V 值 .
7.  如果 result.type 是 normal 并且其完结类型值是 empty, 则返回 undefined 值 .
8.  否则，result.type 必定是 throw。将 result.value 作为异常抛出 .

##### 直接调用 Eval

一个 eval 函数的直接调用是表示为符合以下两个条件的 CallExpression：

解释执行 CallExpression 中的 MemberExpression 的结果是个 引用 ，这个引用拥有一个 环境记录项 作为其基值，并且这个引用的名称是 "eval"。

以这个 引用 作为参数调用 GetValue 抽象操作的结果是 15.1.2.1 定义的标准内置函数。

#### parseInt (string , radix)

parseInt 函数根据指定的参数 radix，和 string 参数的内容解释结果来决定，产生一个整数值。string 开头的空白会被忽略。如果 radis 是 undefined 或 0，假定它是 10，除非数字是以字符对 0x 或 0X 开头的，这时假定 radix 是 16。如果 radix 是 16，数字开头的字符对 0x 或 0X 是可选的。

当调用 parseInt 函数时，采用以下步骤：

1.  令 inputString 为 ToString(string)。
2.  令 S 为一个新创建的子字符串，它由 inputString 的第一个非 StrWhiteSpaceChar 字符和它后面跟着的所有字符组成。( 换句话说 , 删掉前面的空白。) 如果 inputString 不包含任何这样的字符 , 则令 S 为空字符串。
3.  令 sign 为 1。
4.  如果 S 不是空并且 S 的第一个字符是减号 -, 则令 sign 为?1。
5.  如果 S 不是空并且 S 的第一个字符加号 + 或减号 -, 则删除 S 的第一个字符。
6.  令 R = ToInt32(radix).
7.  令 stripPrefix 为 true.
8.  如果 R ≠ 0, 则
    1.  如果 R < 2 或 R > 36, 则返回 NaN。
    2.  如果 R ≠ 16, 令 stripPrefix 为 false.
9.  否则 , R = 0
    1.  令 R = 10.
10.  如果 stripPrefix 是 true, 则
    1.  如果 S 长度大于 2 并且 S 的头两个字符是“0x”或“0X”, 则删除 S 的头两个字符并且令 R = 16.
11.  如果 S 包含任何不是 radix-R 进制的字符，则令 Z 为 S 的这样的字符之前的所有字符组成的子字符串；否则令 Z 为 S。.
12.  如果 Z 是空 , 返回 NaN.
13.  令 mathInt 为 Z 的 radix-R 进制表示的数学值，用字母 A-Z 和 a-z 来表示 10 到 35 之间的值。( 但是 , 如果 R 是 10 并且 Z 包含多余 20 位的值 , 可以替换 20 位后的每个数字为 0, 这是实现可选的功能 ; 如果 R 不是 2, 4, 8, 10, 16, 32, 则 mathInt 可以是 Z 的 radix-R 进制表示的依赖于实现的近似值。)
14.  令 number 为 mathInt 的数值 .
15.  返回 sign × number.

parseInt 可以只把 string 的开头部分解释为整数值；它会忽略所有不能解释为整数记法的一部分的字符，并且没有指示会给出任何这些忽略的字符。

#### parseFloat (string)

parseFloat 函数根据 string 参数的内容解释为十进制字面量的结果来决定，产生一个数值。

当调用 parseFloat 函数，采用以下步骤：

1.  令 inputString 为 ToString(string).
2.  令 trimmedString 为一个新创建的子字符串，它由 inputString 的非 StrWhiteSpaceChar 字符的最左边字符和它右边跟着的所有字符组成。( 换句话说 , 删掉前面的空白。) 如果 inputString 不包含任何这样的字符 , 则令 trimmedString 为空字符串。
3.  如果 trimmedString 或 trimmedString 的任何前缀都不满足 StrDecimalLiteral ( 见 9.3.1) 的语法 , 返回 NaN。
4.  令 numberString 为满足 StrDecimalLiteral 语法的 trimmedString 的最长前缀，可能是 numberString 自身。
5.  返回 numberString 的 MV 的数值。

parseFloat 可以只把 string 的开头部分解释为数值；它会忽略所有不能解释为数值字面量记法的一部分的字符，并且没有指示会给出任何这些忽略的字符。

#### isNaN (number)

如果指定参数为 NaN，则返回 true，否则返回 false。

1.  如果 ToNumber(number) 是 NaN, 返回 true.
2.  否则 , 返回 false.

一个用 ECMAScript 代码来测试值 X 是否是 NaN 的方式是用 X !== X 表达式。当且仅当 X 是 NaN 时结果才是 true。

#### isFinite (number)

如果指定参数为 NaN 或 +∞或?∞，则返回 false，否则返回 true。

1.  如果 ToNumber(number) 是 NaN 或 +∞或?∞, 返回 false.
2.  否则 , 返回 true.

### 处理 URI 的函数属性

统一资源标识符，或叫做 URI，是用来标识互联网上的资源（例如，网页或文件）和怎样访问这些资源的传输协议（例如，HTTP 或 FTP）的字符串。除了 15.1.3.1, 15.1.3.2, 15.1.3.3，15.1.3.4 说明的用来编码和解码 URI 的函数之外 ECMAScript 语言自身不提供任何使用 URL 的支持。

许多 ECMAScript 实现提供额外的函数，方法来操作网页；这些函数超出了本标准的范围。

一个 URI 是由组件分隔符分割的组件序列组成。其一般形式是：

Scheme : First / Second ; Third ? Fourth

其中斜体的名字代表组件；“:”, “/”, “;”，“?”是当作分隔符的保留字符。encodeURI 和 decodeURI 函数操作的是完整的 URI；这俩函数假定 URI 中的任何保留字符都有特殊意义，所有不会编码它们。encodeURIComponent 和 decodeURIComponent 函数操作的是组成 URI 的个别组件；这俩函数假定任何保留字符都代表普通文本，所以必须编码它们，所以它们出现在组成一个完整 URI 的组件里面时不会解释成保留字符了。

以下词法文法指定了编码后 URI 的形式。

语法

`uri ::: uriCharactersopt` `uriCharacters ::: uriCharacter uriCharactersopt` `uriCharacter ::: uriReserved uriUnescaped uriEscaped` `uriReserved ::: one of ; / ? : @ & = + $ ,` `uriUnescaped ::: uriAlpha DecimalDigit uriMark` `uriEscaped ::: % HexDigit HexDigit` `uriAlpha ::: one of a b c d e f g h i j k l m n o p q r s t u v w x y z A B C D E F G H I J K L M N O P Q R S T U V W X Y Z` `uriMark ::: one of - _ . ! ~ * ' ( )`

以上语法是基于 RFC 2396 的，并且较新的 RFC 3986 引入的更改没有反应在这里。

当 URI 里包含一个没在上面列出的字符或有时不想让给定的保留字符有特殊意义，那么必须编码这个字符。字符被转换成 UTF-8 编码，首先从 UT F-16 转换成相应的代码点值的替代对。（注：在 [0,127] 范围的代码单元在单字节中具有相同返回值。）然后返回的字节序列转换为一个字符串，每个字节用一个“%xx”形式的转移序列表示。

描述编码和转义过程的抽象操作 Encode 需要两个字符串参数 string 和 unescapedSet。

1.  令 strLen 为 string 的字符个数 .
2.  令 R 为空字符串 .
3.  令 k 为 0.
4.  重复
    1.  如果 k 等于 strLen, 返回 R.
    2.  令 C 为 string 中位置为 k 的字符 .
    3.  如果 C 在 unescapedSet 里 , 则
        1.  令 S 为一个只包含字符 C 的字符串 .
        2.  令 R 为之前 R 的值和 S 连接得到的一个新字符串值 .
    4.  否则 , C 不在 unescapedSet 里
        1.  如果 C 的代码单元值不小于 0xDC00 并且不大于 0xDFFF, 则抛出一个 URIError 异常 .
        2.  如果 C 的代码单元值小于 0xD800 或大于 0xDBFF, 则
            1.  令 V 为 C 的代码单元值 .
        3.  否则 ,
            1.  k 递增 1.
            2.  如果 k 等于 strLen, 抛出一个 URIError 异常 .
            3.  令 kChar 为 string 的 k 位置的字符的代码单元值 .
            4.  如果 kChar 小于 0xDC00 或大于 0xDFFF, 则抛出一个 URIError 异常 .
            5.  令 V 为 (((C 的代码单元值 ) – 0xD800) * 0x400 + (kChar – 0xDC00) + 0x10000).
        4.  令 Octets 为 V 执行 UTF-8 转换的结果字节排列 , 令 L 为这个字节排列的长度 .
        5.  令 j 为 0.
        6.  只要 j < L，就重复
            1.  令 jOctet 为 Octets 的 j 位置的值 .
            2.  令 S 为一个包含三个字符“%XY”的字符串，这里 XY 是编码 jOctet 值的两个大写 16 进制数字 .
            3.  令 R 为之前 R 的值和 S 连接得到的一个新字符串值 .
            4.  j 递增 1.
    5.  k 递增 1.

描述反转义和解码过程的抽象操作 Decode 需要两个字符串参数 string 和 reservedSet。

1.  令 strLen 为 string 的字符个数 .
2.  令 R 为空字符串 .
3.  令 k 为 0.
4.  重复
    1.  如果 k 等于 strLen, 返回 R.
    2.  令 C 为 string 的 k 位置的字符 .
    3.  如果 C 不是‘%’, 则
        1.  令 S 为只包含字符 C 的字符串 .
    4.  否则 , C 是‘%’
        1.  令 start 为 k.
        2.  如果 k + 2 大于或等于 strLen, 抛出一个 URIError 异常 .
        3.  如果 string 的 (k+1) 和 (k + 2) 位置的字符没有表示为 16 进制数字，则抛出一个 URIError 异常 .
        4.  令 B 为 (k + 1) 和 (k + 2) 位置的两个 16 进制数字表示的 8 位值 .
        5.  k 递增 2.
        6.  如果 B 的最高有效位是 0, 则
            1.  令 C 为代码单元值是 B 的字符 .
            2.  如果 C 不在 reservedSet 里 , 则
                1.  令 S 为只包含字符 C 的字符串 .
            3.  否则 , C 在 reservedSet 里
                1.  令 S 为 string 的从位置 start 到位置 k 的子字符串 .
        7.  否则 , B 的最高有效位是 1
            1.  令 n 为满足 (B << n) & 0x80 等于 0 的最小非负数 .
            2.  如果 n 等于 1 或 n 大于 4, 抛出一个 URIError 异常 .
            3.  令 Octets 为一个长度为 n 的 8 位整数排列 .
            4.  将 B 放到 Octets 的 0 位置 .
            5.  如果 k + (3 * (n – 1)) 大于或等于 strLen, 抛出一个 URIError 异常 .
            6.  令 j 为 1.
            7.  重复 , 直到 j < n
                1.  k 递增 1.
                2.  如果 string 的 k 位置的字符不是‘%’, 抛出一个 URIError 异常 .
                3.  如果 string 的 (k +1) 和 (k+2) 位置的字符没有表示为 16 进制数字 , 抛出一个 URIError 异常 .
                4.  令 B 为 string 的 (k +1) 和 (k+2) 位置的两个 16 进制数字表示的 8 位值 .
                5.  如果 B 的两个最高有效位不是 10，抛出一个 URIError 异常 .
                6.  k 递增 2.
                7.  将 B 放到 Octets 的 j 位置 .
                8.  j 递增 1.
            8.  令 V 为给 Octets 执行 UTF-8 转换得到的值，这是从一个字节排列到一个 32 位值的过程。 如果 Octets 不包含有效的 UTF-8 编码的 Unicode 代码点，则抛出一个 URIError 异常 .
            9.  如果 V 小于 0x10000, 则
                1.  令 C 为代码单元值是 V 的字符 .
                2.  如果 C 不在 reservedSet 里 , 则
                    1.  令 S 为只包含字符 C 的字符串 .
                3.  否则 , C 在 reservedSet 里
                    1.  令 S 为 string 的从位置 start 到位置 k 的子字符串 .
            10.  否则 , V ≥ 0x10000
                1.  令 L 为 (((V – 0x10000) & 0x3FF) + 0xDC00).
                2.  令 H 为 ((((V – 0x10000) >> 10) & 0x3FF) + 0xD800).
                3.  令 S 为代码单元值是 H 和 L 的两个字符组成的字符串 .
5.  令 R 为之前的 R 和 S 连接成的新字符串 .
6.  k 递增 1.

统一资源标识符的语法由 RFC 2396 给出，这里并没有反应更新的替换了 RFC 2396 的 RFC 3986。RFC 3629 给出了实现 UTF-8 的正式描述。

在 UTF-8 中，用 1 到 6 个字节的序列来编码字符。只有“序列”中高阶位设置为 0 的字节，其余的 7 位才用于编码字符值。在一个 n 个字节的序列中，n>1，初始字节有 n 个设置为 1 的高阶位，其后的位设置为 0。这个字节的其他位包含是用来编码字符的比特。后面跟着的其字节都包含设定为 1 的高阶位，并且都跟着设定为 0 的位，剩下的 6 位都用作编码字符。表 21 指定了 ECMAScript 字符可能的 UTF-8 编码。

UTF-8 编码

| 字符编码值 | 表示 | 第 1 十六进制位 | 第 2 六进制位 | 第 3 十六进制位 | 第 4 十六进制位 |
| 0x0000 - 0x007F | 00000000 0zzzzzzz | 0zzzzzzz |  |  |  |
| 0x0080 - 0x07FF | 00000yyy yyzzzzzz | 110yyyyy | 10zzzzzz |  |  |
| 0x0800 - 0xD7FF | xxxxyyyy yyzzzzzz | 1110xxxx | 10yyyyyy | 10zzzzzz |  |
| 0xD800 - 0xDBFF 后跟 0xDC00 - 0xDFFF | 110110vv vvwwwwxx 后跟 110111yy yyzzzzzz | 11110uuu | 10uuwwww | 10xxyyyy | 10zzzzzz |
| 0xD800 - 0xDBFF 后无 0xDC00 - 0xDFFF | 导致 URIError |  |  |  |  |
| 0xDC00 - 0xDFFF | 导致 URIError |  |  |  |  |
| 0xE000 - 0xFFFF | xxxxyyyy yyzzzzzz | 1110xxxx | 10yyyyyy | 10zzzzzz |  |

在这里

uuuuu = vvvv + 1

来访问附加的作为代理项的 0x10000，在 Unicode 标准 3.7 章节。

0xD800-0xDFFF 范围的代码单元值用来编码代理对；如上将 UTF-16 代理对转换组合成一个 UTF-32 表示，并编码 UTF-8 值的 21 位结果。解码重建代理对。

RFC 3629 禁止对无效 UTF-8 字节序列的解码。例如，无效序列 C0 80 不能解码成字符 U+0000。当 Decode 算法的实现遇到这样的无效序列必须抛出一个 URIError 异常。

#### decodeURI (encodedURI)

decodeURI 函数计算出一个新版 URI，将 URI 中可能是 encodeURI 函数引入的每个转义序列和 UTF-8 编码组替换为代表它们的字符。不是 encodeURI 导入的转义序列不会被替换。

当以一个参数 encodedURI 调用 decodeURI 函数，采用如下步骤：

1.  令 uriString 为 ToString(encodedURI).
2.  令 reservedURISet 为一个字符串，包含 uriReserved 的每个有效字符加上 "#" 的实例。
3.  返回调用 Decode(uriString, reservedURISet) 的结果。

"#" 字符不会从转义序列中解码，即使它不是 URI 保留字符。

#### decodeURIComponent (encodedURIComponent)

decodeURIComponent 函数计算出一个新版 URI，将 URI 中可能是 encodeURIComponent 函数引入的每个转义序列和 UTF-8 编码组替换为代表它们的字符。

当以一个参数 encodedURIComponent 调用 decodeURIComponent 函数，采用如下步骤：

1.  令 componentString 为 ToString(encodedURIComponent).
2.  令 reservedURIComponentSet 为一个空字符串。
3.  返回调用 Decode(componentString, reservedURIComponentSet) 的结果。

#### encodeURI (uri)

encodeURI 函数计算出一个新版 URI，将 URI 中某些字符的每个实例替换为代表这些字符 UTF-8 编码的一个，两个或三个转义序列。

当以一个参数 uri 调用 encodeURI 函数，采用如下步骤：

1.  令 uriString 为 ToString(uri).
2.  令 unescapedURISet 为一个字符串，包含 uriReserved 和 uriUnescaped 的每个有效字符加上 "#" 的实例。
3.  返回调用 Encode(uriString, unescapedURISet) 的结果。

字符 "#" 不会被编码为一个转义序列，即使它不是 URI 保留字符或非转义字符。

#### encodeURIComponent (uriComponent)

encodeURIComponent 函数计算出一个新版 URI，将 URI 中某些字符的每个实例替换为代表这些字符 UTF-8 编码的一个，两个或三个转义序列。

当以一个参数 uriComponent 调用 encodeURIComponent 函数，采用如下步骤：

1.  令 componentString 为 ToString(uriComponent).
2.  令 unescapedURIComponentSet 为一个字符串，包含 uriUnescaped 的每个有效字符的实例。
3.  返回调用 Encode(componentString, unescapedURIComponentSet) 的结果。

### 全局对象的构造器属性

#### Object ( . . . )

见 15.2.1 和 15.2.2.

#### Function ( . . . )

见 15.3.1 和 15.3.2

#### Array ( . . . )

见 15.4.1 和 15.4.2.

#### String ( . . . )

见 15.5.1 和 15.5.2.

#### Boolean ( . . . )

见 15.6.1 和 15.6.2.

#### Number ( . . . )

见 15.7.1 和 15.7.2.

#### Date ( . . . )

见 15.9.2.

#### RegExp ( . . . )

见 15.10.3 和 15.10.4.

#### Error ( . . . )

见 15.11.1 和 15.11.2.

#### EvalError ( . . . )

见 15.11.6.1.

#### RangeError ( . . . )

见 15.11.6.2.

#### ReferenceError ( . . . )

见 15.11.6.3.

#### SyntaxError ( . . . )

见 15.11.6.4.

#### TypeError ( . . . )

见 15.11.6.5.

#### URIError ( . . . )

见 15.11.6.6.

### 全局对象的其他属性

#### Math

见 15.8.

#### JSON

见 15.12.

## Object 对象

### 作为函数调用 Object 构造器

当把 Object 当做一个函数来调用，而不是一个构造器，它会执行一个类型转换。

#### Object ( [ value ] )

当以一个参数 value 或者无参数调用 Object 函数，采用如下步骤：

1.  如果 value 是 null, undefined 或未指定，则创建并返回一个新 Object 对象 , 这个对象与仿佛用相同参数调用标准内置的 Object 构造器 (15.2.2.1) 的结果一样 .
2.  返回 ToObject(value).

### Object 构造器

当 Object 是 new 表达式调用的一部分时，它是一个构造器，可创建一个对象。

#### new Object ( [ value ] )

当以一个参数 value 或者无参数调用 Object 构造器，采用如下步骤：

1.  如果提供了 value, 则
    1.  如果 Type(value) 是 Object, 则
        1.  如果 value 是个原生 ECMAScript 对象 , 不创建新对象，简单的返回 value.
        2.  如果 value 是宿主对象 , 则采取动作和返回依赖实现的结果的方式可以使依赖于宿主对象的 .
    2.  如果 Type(value) 是 String, 返回 ToObject(value).
    3.  如果 Type(value) 是 Boolean, 返回 ToObject(value).
    4.  如果 Type(value) 是 Number, 返回 ToObject(value).
2.  断言 : 未提供参数 value 或其类型是 Null 或 Undefined.
3.  令 obj 为一个新创建的原生 ECMAScript 对象 .
4.  设定 obj 的 [[Prototype]] 内部属性为标准内置的 Object 的 prototype 对象 (15.2.4).
5.  设定 obj 的 [[Class]] 内部属性为 "Object".
6.  设定 obj 的 [[Extensible]] 内部属性为 true.
7.  设定 obj 的 8.12 指定的所有内部方法
8.  返回 obj.

### Object 构造器的属性

Object 构造器的 [[Prototype]] 内部属性值是标准内置 Function 的 prototype 对象。

除了内部属性和 length 属性（其值是 1）之外，Object 构造器拥有以下属性：

#### Object.prototype

Object.prototype 的初始值是标准内置 Object 的 prototype 对象（15.2.4）。

这个属性包含特性 {[[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }

#### Object.getPrototypeOf ( O )

当以参数 O 调用 getPrototypeOf 函数，采用如下步骤：

1.  如果 Type(O) 不是 Object，则抛出一个 TypeError 异常 .
2.  返回 O 的 [[Prototype]] 内部属性的值 .

#### Object.getOwnPropertyDescriptor ( O, P )

当调用 getOwnPropertyDescriptor 函数，采用如下步骤：

1.  如果 Type(O) 不是 Object，则抛出一个 TypeError 异常 .
2.  令 name 为 ToString(P).
3.  令 desc 为以参数 name 调用 O 的 [[GetOwnProperty]] 内部方法的结果 .
4.  返回调用 FromPropertyDescriptor(desc) 的结果 (8.10.4).

#### Object.getOwnPropertyNames ( O )

当调用 getOwnPropertyNames 函数，采用如下步骤：

1.  如果 Type(O) 不是 Object，则抛出一个 TypeError 异常 .
2.  令 array 为仿佛是用表达式 new Array () 创建新对象的结果，这里的 Array 是标准内置构造器名。
3.  令 n 为 0.
4.  对 O 的每个自身属性 P
    1.  令 name 为值是 P 的名称的字符串 .
    2.  以 ToString(n) 和属性描述 {[[Value]]: name, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true} 和 false 为参数调用 array 的 [[DefineOwnProperty]] 内部方法 .
    3.  n 递增 1.
5.  返回 array.

如果 O 是一个字符串实例，第 4 步处理的自身属性集合包含 15.5.5.2 定义的隐藏属性，他们对应对象的 [[PrimitiveValue]] 字符串中相应位置的字符。

#### Object.create ( O [, Properties] )

create 函数按照指定的原型创建一个新对象。当调用 create 函数，采用如下步骤：

1.  如果 Type(O) 不是 Object 或 Null，则抛出一个 TypeError 异常 .
2.  令 obj 为为仿佛是用表达式 new Object() 创建新对象的结果，这里的 Object 是标准内置构造器名。
3.  设定 obj 的 [[Prototype]] 内部属性为 O.
4.  如果传入了 Properties 参数并且不是 undefined, 则仿佛是用 obj 和 Properties 当作参数调用标准内置函数 Object.defineProperties 一样给 obj 添加自身属性。
5.  返回 obj.

#### Object.defineProperty ( O, P, Attributes )

defineProperty 函数用于给一个对象添加一个自身属性 并 / 或 更新现有自身属性的特性。当调用 defineProperty 函数，采用如下步骤：

1.  如果 Type(O) 不是 Object，则抛出一个 TypeError 异常 .
2.  令 name 为 ToString(P).
3.  令 desc 为以 Attributes 作为参数调用 ToPropertyDescriptor 的结果 .
4.  以 name, desc, true 作为参数调用 O 的 [[DefineOwnProperty]] 内部方法 .
5.  返回 O.

#### Object.defineProperties ( O, Properties )

defineProperties 函数用于给一个对象添加一些自身属性 并 / 或 更新现有的一些自身属性的特性。当调用 defineProperties 函数，采用如下步骤：

1.  如果 Type(O) 不是 Object，则抛出一个 TypeError 异常 .
2.  令 props 为 ToObject(Properties).
3.  令 names 为一个内部列表，它包含 props 的每个可遍历自身属性的名称 .
4.  令 descriptors 为一个空的内部列表 .
5.  对 names 的每个元素 P，按照列表顺序 ,
    1.  令 descObj 为以 P 作为参数调用 props 的 [[Get]] 内部方法的结果 .
    2.  令 desc 为以 descObj 作为参数调用 ToPropertyDescriptor 的结果 .
    3.  将 desc 插入 descriptors 的尾部 .
6.  对 descriptors 的每个元素 desc，按照列表顺序 ,
    1.  以参数 P, desc, true 调用 O 的 [[DefineOwnProperty]] 内部方法 .
7.  返回 O

如果一个实现为 for-in 语句的定义了特定的枚举顺序，那么在这个算法的第 3 步中的列表元素必须也用相同的顺序排列。

#### Object.seal ( O )

当调用 seal 函数，采用如下步骤：

1.  如果 Type(O) 不是 Object，则抛出一个 TypeError 异常 .
2.  对 O 的每个命名自身属性名 P,
    1.  令 desc 为以参数 P 调用 O 的 [[GetOwnProperty]] 内部方法的结果 .
    2.  如果 desc.[[Configurable]] 是 true, 设定 desc.[[Configurable]] 为 false.
    3.  以 P, desc, true 为参数调用 O 的 [[DefineOwnProperty]] 内部方法 .
3.  设定 O 的 [[Extensible]] 内部属性为 false.
4.  返回 O.

#### Object.freeze ( O )

当调用 freeze 函数，采用如下步骤：

1.  如果 Type(O) 不是 Object，则抛出一个 TypeError 异常 .
2.  对 O 的每个命名自身属性名 P,
    1.  令 desc 为以参数 P 调用 O 的 [[GetOwnProperty]] 内部方法的结果 .
    2.  如果 IsDataDescriptor(desc) 是 true, 则
        1.  如果 desc.[[Writable]] 是 true, 设定 desc.[[Writable]] 为 false.
    3.  如果 desc.[[Configurable]] 是 true, 设定 desc.[[Configurable]] 为 false.
    4.  以 P, desc, true 作为参数调用 O 的 [[DefineOwnProperty]] 内部方法 .
3.  设定 O 的 [[Extensible]] 内部属性为 false.
4.  返回 O.

#### Object.preventExtensions ( O )

当调用 preventExtensions 函数，采用如下步骤：

1.  如果 Type(O) 不是 Object，则抛出一个 TypeError 异常 .
2.  设定 O 的 [[Extensible]] 内部属性为 false.
3.  返回 O.

#### Object.isSealed ( O )

当以参数 O 调用 isSealed 函数，采用如下步骤：

1.  如果 Type(O) 不是 Object，则抛出一个 TypeError 异常 .
2.  对 O 的每个命名自身属性名 P,
    1.  令 desc 为以参数 P 调用 O 的 [[GetOwnProperty]] 内部方法的结果 .
    2.  如果 desc.[[Configurable]] 是 true, 则返回 false.
3.  如果 O 的 [[Extensible]] 内部属性是 false, 则返回 true.
4.  否则 , 返回 false.

#### Object.isFrozen ( O )

当以参数 O 调用 isFrozen 函数，采用如下步骤：

1.  如果 Type(O) 不是 Object，则抛出一个 TypeError 异常 .
2.  对 O 的每个命名自身属性名 P,
    1.  令 desc 为以参数 P 调用 O 的 [[GetOwnProperty]] 内部方法的结果 .
    2.  如果 IsDataDescriptor(desc) 是 true，则
        1.  如果 desc.[[Writable]] 是 true, 则返回 false.
    3.  如果 desc.[[Configurable]] 是 true, 则返回 false.
3.  如果 O 的 [[Extensible]] 内部属性是 false, 则返回 true.
4.  否则 , 返回 false.

#### Object.isExtensible ( O )

当以参数 O 调用 isExtensible 函数，采用如下步骤：

1.  如果 Type(O) 不是 Object，则抛出一个 TypeError 异常 .
2.  返回 O 的 [[Extensible]] 内部属性布尔值 .

#### Object.keys ( O )

当以参数 O 调用 keys 函数，采用如下步骤：

1.  如果 Type(O) 不是 Object，则抛出一个 TypeError 异常 .
2.  令 n 为 O 的可遍历自身属性的个数
3.  令 array 为仿佛是用表达式 new Array () 创建新对象的结果，这里的 Array 是标准内置构造器名。
4.  令 index 为 0.
5.  对 O 的每个可遍历自身属性名 P,
    1.  以 ToString(index)，属性描述 {[[Value]]: P, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}，和 false 作为参数调用 array 的 [[DefineOwnProperty]] 内部方法。
    2.  index 递增 1.
6.  返回 array.

如果一个实现为 for-in 语句的定义了特定的枚举顺序，那么在这个算法的第 5 步中的必须使用相同的枚举顺序。

### Object 的 prototype 对象的属性

Object 的 prototype 对象的 [[Prototype]] 内部属性的值是 null，[[Class]] 内部属性的值是 "Object"，[[Extensible]] 内部属性的初始值是 true。

#### Object.prototype.constructor

Object.prototype.constructor 的初始值是标准内置的 Object 构造器。

#### Object.prototype.toString ( )

当调用 toString 方法，采用如下步骤：

1.  如果 this 的值是 undefined, 返回 "[object Undefined]".
2.  如果 this 的值是 null, 返回 "[object Null]".
3.  令 O 为以 this 作为参数调用 ToObject 的结果 .
4.  令 class 为 O 的 [[Class]] 内部属性的值 .
5.  返回三个字符串 "[object ", class, and "]" 连起来的字符串 .

#### Object.prototype.toLocaleString ( )

当调用 toLocaleString 方法，采用如下步骤：

1.  令 O 为以 this 作为参数调用 ToObject 的结果 .
2.  令 toString 为以 "toString" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
3.  如果 IsCallable(toString) 是 false, 抛出一个 TypeError 异常 .
4.  返回以 O 作为 this 值，无参数调用 toString 的 [[Call]] 内部方法的结果 .

这个函数给所有 Object 对象提供一个通用的 toLocaleString 接口，即使并不是所有的都使用它。目前，Array, Number, Date 提供了它们自身的语言环境敏感的 toLocaleString 方法。

这个函数的第一个参数可能会在此标准的未来版本中使用到；因此建议实现不要用这个位置参数来做其他事情。

#### Object.prototype.valueOf ( )

当调用 valueOf 方法，采用如下步骤：

1.  令 O 为以 this 作为参数调用 ToObject 的结果 .
2.  如果 O 是以一个宿主对象 (15.2.2.1) 为参数调用 Object 构造器的结果，则
    1.  返回 O 或传递给构造器的原来的宿主对象 . 返回的具体结果是由实现定义的 .
3.  返回 O.

#### Object.prototype.hasOwnProperty (V)

当以参数 V 调用 hasOwnProperty 方法，采用如下步骤：

1.  令 P 为 ToString(V).
2.  令 O 为以 this 值作为参数调用 ToObject 的结果 .
3.  令 desc 为以 P 为参数调用 O 的 [[GetOwnProperty]] 内部方法的结果 .
4.  如果 desc 是 undefined, 返回 false.
5.  返回 true.

不像 [[HasProperty]](8.12.6)，这个方法不考虑原形链中的对象。

为步骤 1 和 2 的选择这样的顺序，是为了确保在本规范之前版本中会在这里的步骤 1 里抛出的任何异常，即使 this 值是 undefined 或 null，也会继续抛出。

#### Object.prototype.isPrototypeOf (V)

当以参数 V 调用 isPrototypeOf 方法，采用如下步骤：

1.  如果 V 不是个对象 , 返回 false.
2.  令 O 为以 this 作为参数调用 ToObject 的结果 .
3.  重复
    1.  令 V 为 V 的 [[Prototype]] 内部属性的值 .
    2.  如果 V 是 null, 返回 false
    3.  如果 O 和 V 指向同一个对象 , 返回 true.

为步骤 1 和 2 的选择这样的顺序，是为了当 V 不是对象并且 this 值是 undefined 或 null 时能够保持本规范之前版本指定的行为。

#### Object.prototype.propertyIsEnumerable (V)

当以参数 V 调用 propertyIsEnumerable 方法，采用如下步骤：

1.  令 P 为 ToString(V).
2.  令 O 为以 this 作为参数调用 ToObject 的结果 .
3.  令 desc 为以 P 作为参数调用 O 的 [[GetOwnProperty]] 内部方法的结果 .
4.  如果 desc 是 undefined, 返回 false.
5.  返回 desc.[[Enumerable]] 的值 .

这个方法不考虑原型链中的对象。

为步骤 1 和 2 的选择这样的顺序，是为了确保在本规范之前版本中会在这里的步骤 1 里抛出的任何异常，即使 this 值是 undefined 或 null，也会继续抛出。

### Object 的实例的属性

Object 的实例除了拥从 Object 的 prototype 对象继承来的属性之外不包含特殊的属性。

## Function 对象

### 作为函数调用 Function 构造器

当将 Function 作为函数来调用，而不是作为构造器，它会创建并初始化一个新函数对象。所以函数调用 Function(…) 与用相同参数的 new Function(…) 表达式创建的对象相同。

#### Function (p1, p2, … , pn, body)

当以 p1, p2, … , pn, body 作为参数调用 Function 函数（这里的 n 可以是 0，也就是说没有“p”参数，这时还可以不提供 body），采用如下步骤：

1.  创建并返回一个新函数对象，它仿佛是用相同参数给标准内置构造器 Function (15.3.2.1). 用一个 new 表达式创建的。

### Function 构造器

当 Function 作为 new 表达式的一部分被调用时，它是一个构造器：它初始化新创建的对象。

#### new Function (p1, p2, … , pn, body)

最后一个参数指定为函数的 body( 可执行代码 )；之前的任何参数都指定为形式参数。

当以 p1, p2, … , pn, body 作为参数调用 Function 构造器（这里的 n 可以是 0，也就是说没有“p”参数，这时还可以不提供 body），采用如下步骤：

1.  令 argCount 为传给这个函数调用的参数总数 .
2.  令 P 为空字符串 .
3.  如果 argCount = 0, 令 body 为空字符串 .
4.  否则如果 argCount = 1, 令 body 为那个参数 .
5.  否则 , argCount > 1
    1.  令 firstArg 为第一个参数 .
    2.  令 P 为 ToString(firstArg).
    3.  令 k 为 2.
    4.  只要 k < argCount 就重复
        1.  令 nextArg 为第 k 个参数 .
        2.  令 P 为之前的 P 值，字符串 ","（一个逗号），ToString(nextArg) 串联的结果。
        3.  k 递增 1.
    5.  令 body 为第 k 个参数 .
6.  令 body 为 ToString(body).
7.  如果 P 不可解析为一个 FormalParameterListopt，则抛出一个 SyntaxError 异常 .
8.  如果 body 不可解析为 FunctionBody，则抛出一个 SyntaxError 异常 .
9.  如果 body 是严格模式代码 ( 见 10.1.1)，则令 strict 为 true, 否则令 strict 为 false.
10.  如果 strict 是 true, 适用 13.1 指定抛出的任何异常 .
11.  返回一个新创建的函数对象，它是依照 13.2 指定 -- 专递 P 作为 FormalParameterList，body 作为 FunctionBody，全局环境作为 Scope 参数，strict 作为严格模式标志 -- 创建的。

每个函数都会自动创建一个 prototype 属性，用来支持函数被当做构造器使用的可能性。

为每个形参指定一个参数是允许的，但没必要。例如以下三个表达式产生相同的结果： `new Function("a", "b", "c", "return a+b+c") new Function("a, b, c", "return a+b+c") new Function("a,b", "c", "return a+b+c")`

### Function 构造器的属性

Function 构造器自身是个函数对象，它的 [[Class]] 是 "Function"。Function 构造器的 [[Prototype]] 内部属性值是标准内置 Function 的 prototype 对象 (15.3.4)。

Function 构造器的 [[Extensible]] 内部属性值是 true.

Function 构造器有如下属性 :

#### Function.prototype

Function.prototype 的初始值是标准内置 Function 的 prototype 对象 (15.3.4)。

此属性拥有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

#### Function.length

这是个值为 1 的数据属性。此属性拥有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

### Function 的 prototype 对象的属性

Function 的 prototype 对象自身是一个函数对象 ( 它的 [[Class]] 是 "Function")，调用这个函数对象时，接受任何参数并返回 undefined。

Function 的 prototype 对象的 [[Prototype]] 内部属性值是标准内置 Object 的 prototype 对象 (15.2.4)。Function 的 prototype 对象的 [[Extensible]] 内部属性的初始值是 true。

Function 的 prototype 对象自身没有 valueOf 属性 ; 但是，它从 Object 的 prototype 对象继承了 valueOf 属性。

Function 的 prototype 对象的 length 属性是 0。

#### Function.prototype.constructor

Function.prototype.constructor 的初始值是内置 Function 构造器。

#### Function.prototype.toString ( )

此函数的返回值的表示是依赖于实现的。这个表示包含 FunctionDeclaration 的语法。特别注意，怎样在这个字符串表示中使用和放置空白，行终结符，分号是依赖于实现的。

这个 toString 不是通用的；如果它的 this 值不是一个函数对象，它会抛出一个 TypeError 异常。因此，它不能当做方法来转移到其他类型的对象中。

#### Function.prototype.apply (thisArg, argArray)

当以 thisArg 和 argArray 为参数在一个 func 对象上调用 apply 方法，采用如下步骤：

1.  如果 IsCallable(func) 是 false, 则抛出一个 TypeError 异常 .
2.  如果 argArray 是 null 或 undefined, 则
    1.  返回提供 thisArg 作为 this 值并以空参数列表调用 func 的 [[Call]] 内部方法的结果。
3.  如果 Type(argArray) 不是 Object, 则抛出一个 TypeError 异常 .
4.  令 len 为以 "length" 作为参数调用 argArray 的 [[Get]] 内部方法的结果。
5.  令 n 为 ToUint32(len).
6.  令 argList 为一个空列表 .
7.  令 index 为 0.
8.  只要 index < n 就重复
    1.  令 indexName 为 ToString(index).
    2.  令 nextArg 为以 indexName 作为参数调用 argArray 的 [[Get]] 内部方法的结果。
    3.  将 nextArg 作为最后一个元素插入到 argList 里。
    4.  设定 index 为 index + 1.
9.  提供 thisArg 作为 this 值并以 argList 作为参数列表，调用 func 的 [[Call]] 内部方法，返回结果。

apply 方法的 length 属性是 2。

在外面传入的 thisArg 值会修改并成为 this 值。thisArg 是 undefined 或 null 时它会被替换成全局对象，所有其他值会被应用 ToObject 并将结果作为 this 值，这是第三版引入的更改。

#### Function.prototype.call (thisArg [ , arg1 [ , arg2, … ] ] )

当以 thisArg 和可选的 arg1, arg2 等等作为参数在一个 func 对象上调用 call 方法，采用如下步骤：

1.  如果 IsCallable(func) 是 false, 则抛出一个 TypeError 异常。
2.  令 argList 为一个空列表。
3.  如果调用这个方法的参数多余一个，则从 arg1 开始以从左到右的顺序将每个参数插入为 argList 的最后一个元素。
4.  提供 thisArg 作为 this 值并以 argList 作为参数列表，调用 func 的 [[Call]] 内部方法，返回结果。

call 方法的 length 属性是 1。

在外面传入的 thisArg 值会修改并成为 this 值。thisArg 是 undefined 或 null 时它会被替换成全局对象，所有其他值会被应用 ToObject 并将结果作为 this 值，这是第三版引入的更改。

#### Function.prototype.bind (thisArg [, arg1 [, arg2, …]])

bind 方法需要一个或更多参数，thisArg 和（可选的）arg1, arg2, 等等，执行如下步骤返回一个新函数对象：

1.  令 Target 为 this 值 .
2.  如果 IsCallable(Target) 是 false, 抛出一个 TypeError 异常 .
3.  令 A 为一个（可能为空的）新内部列表，它包含按顺序的 thisArg 后面的所有参数（arg1, arg2 等等）。
4.  令 F 为一个新原生 ECMAScript 对象。
5.  依照 8.12 指定，设定 F 的除了 [[Get]] 之外的所有内部方法。
6.  依照 15.3.5.4 指定，设定 F 的 [[Get]] 内部属性。
7.  设定 F 的 [[TargetFunction]] 内部属性为 Target。
8.  设定 F 的 [[BoundThis]] 内部属性为 thisArg 的值。
9.  设定 F 的 [[BoundArgs]] 内部属性为 A。
10.  设定 F 的 [[Class]] 内部属性为 "Function"。
11.  设定 F 的 [[Prototype]] 内部属性为 15.3.3.1 指定的标准内置 Function 的 prototype 对象。
12.  依照 15.3.4.5.1 描述，设定 F 的 [[Call]] 内置属性。
13.  依照 15.3.4.5.2 描述，设定 F 的 [[Construct]] 内置属性。
14.  依照 15.3.4.5.3 描述，设定 F 的 [[HasInstance]] 内置属性。
15.  如果 Target 的 [[Class]] 内部属性是 "Function", 则
    1.  令 L 为 Target 的 length 属性减 A 的长度。
    2.  设定 F 的 length 自身属性为 0 和 L 中更大的值。
16.  否则设定 F 的 length 自身属性为 0.
17.  设定 F 的 length 自身属性的特性为 15.3.5.1 指定的值。
18.  设定 F 的 [[Extensible]] 内部属性为 true。
19.  令 thrower 为 [[ThrowTypeError]] 函数对象 (13.2.3)。
20.  以 "caller", 属性描述符 {[[Get]]: thrower, [[Set]]: thrower, [[Enumerable]]: false, [[Configurable]]: false}, 和 false 作为参数调用 F 的 [[DefineOwnProperty]] 内部方法。
21.  以 "arguments", 属性描述符 {[[Get]]: thrower, [[Set]]: thrower, [[Enumerable]]: false, [[Configurable]]: false}, 和 false 作为参数调用 F 的 [[DefineOwnProperty]] 内部方法。
22.  返回 F.

bind 方法的 length 属性是 1。

Function.prototype.bind 创建的函数对象不包含 prototype 属性或 [[Code]], [[FormalParameters]], [[Scope]] 内部属性。

##### [[Call]]

当调用一个用 bind 函数创建的函数对象 F 的 [[Call]] 内部方法，传入一个 this 值和一个参数列表 ExtraArgs，采用如下步骤：

1.  令 boundArgs 为 F 的 [[BoundArgs]] 内部属性值。
2.  令 boundThis 为 F 的 [[BoundThis]] 内部属性值。
3.  令 target 为 F 的 [[TargetFunction]] 内部属性值。
4.  令 args 为一个新列表，它包含与列表 boundArgs 相同顺序相同值，后面跟着与 ExtraArgs 是相同顺序相同值。
5.  提供 boundThis 作为 this 值，提供 args 为参数调用 target 的 [[Call]] 内部方法，返回结果。

##### [[Construct]]

当调用一个用 bind 函数创建的函数对象 F 的 [[Construct]] 内部方法，传入一个参数列表 ExtraArgs，采用如下步骤：

1.  令 target 为 F 的 [[TargetFunction]] 内部属性值。
2.  如果 target 不包含 [[Construct]] 内部方法 , 抛出一个 TypeError 异常。
3.  令 boundArgs 为 F 的 [[BoundArgs]] 内部属性值。
4.  令 args 为一个新列表，它包含与列表 boundArgs 相同顺序相同值，后面跟着与 ExtraArgs 是相同顺序相同值。
5.  提供 args 为参数调用 target 的 [[Construct]] 内部方法，返回结果。

##### [[HasInstance]] (V)

当调用一个用 bind 函数创建的函数对象 F 的 [[Construct]] 内部方法，并以 V 作为参数，采用如下步骤：

1.  令 target 为 F 的 [[TargetFunction]] 内部属性值。
2.  如果 target 不包含 [[HasInstance]] 内部方法 , 抛出一个 TypeError 异常。
3.  提供 V 为参数调用 target 的 [[HasInstance]] 内部方法，返回结果。

### Function 的实例的属性

除了必要的内部属性之外，每个函数实例还有一个 [[Call]] 内部属性并且在大多数情况下使用不同版本的 [[Get]] 内部属性。函数实例根据怎样创建的（见 8.6.2 ,13.2, 15, 15.3.4.5）可能还有一个 [[HasInstance]] 内部属性 , 一个 [[Scope]] 内部属性 , 一个 [[Construct]] 内部属性 , 一个 [[FormalParameters]] 内部属性 , 一个 [[Code]] 内部属性 , 一个 [[TargetFunction]] 内部属性 , 一个 [[BoundThis]] 内部属性 , 一个 [[BoundArgs]] 内部属性。

[[Class]] 内部属性的值是 "Function"。

对应于严格模式函数 (13.2) 的函数实例和用 Function.prototype.bind 方法 (15.3.4.5) 创建的函数实例有名为“caller”和 “arguments”的属性时，抛出一个 TypeError 异常。一个 ECMAScript 实现不得为在严格模式函数代码里访问这些属性关联任何依赖实现的特定行为。

#### length

length 属性值是个整数，它指出函数预期的“一般的”参数个数。然而，语言允许用其他数量的参数来调用函数。当以与函数的 length 属性指定的数量不同的参数个数调用函数时，它的行为依赖于函数自身。这个属性拥有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

#### prototype

prototype 属性的值用于初始化一个新创建对象的的 [[Prototype]] 内部属性，为了这个新创建对象要先将函数对象作为构造器调用。这个属性拥有特性 { [[Writable]]: true, [[Enumerable]]: false, [[Configurable]]: false }。

用 Function.prototype.bind 创建的函数对象没有 prototype 属性。

#### [[HasInstance]] (V)

设 F 是个函数对象。

当以 V 作为参数调用 F 的 [[HasInstance]] 内部方法，采用如下步骤：

1.  如果 V 不是个对象 , 返回 false。
2.  令 O 为用属性名 "prototype" 调用 F 的 [[Get]] 内部方法的结果。
3.  如果 Type(O) 不是 Object, 抛出一个 TypeError 异常。
4.  重复
    1.  令 V 为 V 的 [[Prototype]] 内部属性值。
    2.  如果 V 是 null, 返回 false.
    3.  如果 O 和 V 指向相同对象，返回 true。

用 Function.prototype.bind 创建的函数对象拥有的不同的 [[HasInstance]] 实现，在 15.3.4.5.3 中定义。

#### [[Get]] (P)

函数对象与其他原生 EMACScript 对象 (8.12.3) 用不同的 [[Get]] 内部方法。

设 F 是一个函数对象，当以属性名 P 调用 F 的 [[Get]] 内部方法 , 采用如下步骤：

1.  令 v 为传入 P 作为属性名参数调用 F 的默认 [[Get]] 内部方法 (8.12.3) 的结果。
2.  如果 P 是 "caller" 并且 v 是个严格模式函数对象 , 抛出一个 TypeError 异常。
3.  返回 v。

用 Function.prototype.bind 创建的函数对象使用默认的 [[Get]] 内部方法。

## Array 对象

数组对象会给予一些种类的属性名特殊待遇。对一个属性名 P（字符串形式），当且仅当 ToString(ToUint32(P)) 等于 P 并且 ToUint32(P) 不等于 2³²?1 时，它是个 数组索引 。一个属性名是数组索引的属性还叫做 元素 。所有数组对象都有一个 length 属性，其值始终是一个小于 2³² 的非负整数。length 属性值在数值上比任何名为数组索引的属性的名称还要大；每当创建或更改一个数组对象的属性，要调整其他的属性以保持上面那个条件不变的需要。具体来说，每当添加一个名为数组索引的属性时，如果需要就更改 length 属性为在数值上比这个数组索引大 1 的值；每当更改 length 属性，所有属性名是数组索引并且其值不小于新 length 的属性会被自动删除。这个限制只应用于数组对象的自身属性，并且从原型中继承的 length 或数组索引不影响这个限制。

对一个对象 O，如果以下算法返回 true，那么就叫这个对象为 稀疏 的：

1.  令 len 为以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果。
2.  对每个范围在 0≤i<ToUint32(len) 的整数 i
    1.  令 elem 为以 ToString(i) 作为参数调用 O 的 [[GetOwnProperty]] 内部方法的结果。
    2.  如果 elem 是 undefined, 返回 true.
3.  返回 false.

### 作为函数调用 Array 构造器

当将 Array 作为函数来调用，而不是作为构造器，它会创建并初始化一个新数组对象。所以函数调用 Array(…) 与用相同参数的 new Array(…) 表达式创建的对象相同。

#### Array ( [ item1 [ , item2 [ , … ] ] ] )

当调用 Array 函数，采用如下步骤：

1.  创建并返回一个新函数对象，它仿佛是用相同参数给标准内置构造器 Array 用一个 new 表达式创建的 (15.4.2)。

### Array 构造器

当 Array 作为 new 表达式的一部分被调用时，它是一个构造器：它初始化新创建的对象。

#### new Array ( [ item0 [ , item1 [ , … ] ] ] )

当且仅当以无参数或至少两个参数调用 Array 构造器时，适用这里的描述。

新构造对象的 [[Prototype]] 内部属性要设定为原始的数组原型对象，他是 Array.prototype(15.4.3.1) 的初始值。

新构造对象的 [[Class]] 内部属性要设定为 "Array"。

新构造对象的 [[Extensible]] 内部属性要设定为 true。

新构造对象的 length 属性要设定为参数的个数。

新构造对象的 0 属性要设定为 item0( 如果提供了 ); 新构造对象的 1 属性要设定为 item1( 如果提供了 ); 更多的参数可应用普遍规律，新构造对象的 k 属性要设定为第 k 个参数，这里的 k 是从 0 开始的。所有这些属性都有特性 {[[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}。

#### new Array (len)

新构造对象的 [[Prototype]] 内部属性要设定为原始的数组原型对象，他是 Array.prototype(15.4.3.1) 的初始值。新构造对象的 [[Class]] 内部属性要设定为 "Array"。新构造对象的 [[Extensible]] 内部属性要设定为 true。

如果参数 len 是个数字值并且 ToUint32(len) 等于 len，则新构造对象的 length 属性要设定为 ToUint32(len)。如果参数 len 是个数字值并且 ToUint32(len) 不等于 len，则抛出一个 RangeError 异常。

如果参数 len 不是数字值，则新构造对象的 length 属性要设定为 0，并且新构造对象的 0 属性要设定为 len，设定它的特性为 {[[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}。

### Array 构造器的属性

Array 构造器的 [[Prototype]] 内部属性值是函数原型对象 (15.3.4)。

Array 构造器除了有一些内部属性和 length 属性（其值是 1）之外，还有如下属性：

#### Array.prototype

Array.prototype 的初始值是数组原型对象 (15.4.4)。

此属性拥有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false}。

#### Array.isArray ( arg )

isArray 函数需要一个参数 arg，如果参数是个对象并且 class 内部属性是 "Array", 返回布尔值 true；否则它返回 false。采用如下步骤：

1.  如果 Type(arg) 不是 Object, 返回 false。
2.  如果 arg 的 [[Class]] 内部属性值是 "Array", 则返回 true。
3.  返回 false.

### 数组原型对象的属性

数组原型对象的 [[Prototype]] 内部属性值是标准内置 Object 原型对象 (15.2.4)。

数组原型对象自身是个数组；它的 [[Class]] 是 "Array"，它拥有一个 length 属性（初始值是 +0）和 15.4.5.1 描述的特殊的 [[DefineOwnProperty]] 内部方法。

在以下的对数组原型对象的属性函数的描述中，短语“this 对象”指的是调用这个函数时的 this 值对象。允许 this 是 [[Class]] 内部属性值不是 "Array" 的对象。

数组原型对象不包含它自身的 valueOf 属性；然而，它从标准内置 Object 原型对象继承 valueOf 属性。

#### Array.prototype.constructor

Array.prototype.constructor 的初始值是标准内置 Array 构造器。

#### Array.prototype.toString ( )

当调用 toString 方法，采用如下步骤：

1.  令 array 为用 this 值调用 ToObject 的结果。
2.  令 func 为以 "join" 作为参数调用 array 的 [[Get]] 内部方法的结果。
3.  如果 IsCallable(func) 是 false, 则令 func 为标准内置方法 Object.prototype.toString (15.2.4.2)。
4.  提供 array 作为 this 值并以空参数列表调用 func 的 [[Call]] 内部方法，返回结果。

toString 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 toString 函数是依赖于实现的。

#### Array.prototype.toLocaleString ( )

先用数组元素的 toLocaleString 方法，将他们转换成字符串。然后将这些字符串串联，用一个分隔符分割，这里的分隔符字符串是与特定语言环境相关，由实现定义的方式得到的。调用这个函数的结果除了与特定语言环境关联之外，与 toString 的结果类似。

结果是按照一下方式计算的：

1.  令 array 为以 this 值作为参数调用 ToObject 的结果。
2.  令 arrayLen 为以 "length" 作为参数调用 array 的 [[Get]] 内部方法的结果。
3.  令 len 为 ToUint32(arrayLen)。
4.  令 separator 为宿主环境的当前语言环境对应的列表分隔符字符串（这是实现定义的方式得到的）。
5.  如果 len 是零 , 返回空字符串。
6.  令 firstElement 为以 "0" 作为参数调用 array 的 [[Get]] 内部方法的结果。
7.  如果 firstElement 是 undefined 或 null, 则
    1.  令 R 为空字符串。
8.  否则
    1.  令 elementObj 为 ToObject(firstElement).
    2.  令 func 为以 "toLocaleString" 作为参数调用 elementObj 的 [[Get]] 内部方法的结果。
    3.  如果 IsCallable(func) 是 false, 抛出一个 TypeError 异常。
    4.  令 R 为提供 elementObj 作为 this 值并以空参数列表调用 func 的 [[Call]] 内部方法的结果。
9.  令 k 为 1。
10.  只要 k < len 就重复
    1.  令 S 为串联 R 和 separator 产生的字符串。
    2.  令 nextElement 为以 ToString(k) 作为参数调用 array 的 [[Get]] 内部方法的结果。
    3.  如果 nextElement 是 undefined 或 null, 则
        1.  令 R 为空字符串。
    4.  否则
        1.  令 elementObj 为 ToObject(nextElement).
        2.  令 func 为以 "toLocaleString" 作为参数调用 elementObj 的 [[Get]] 内部方法的结果。
        3.  如果 IsCallable(func) 是 false, 抛出一个 TypeError 异常。
        4.  令 R 为提供 elementObj 作为 this 值并以空参数列表调用 func 的 [[Call]] 内部方法的结果。
    5.  令 R 为串联 S 和 R 产生的字符串。
    6.  k 递增 1。
11.  返回 R

此函数的第一个参数可能会在本标准的未来版本中用到；建议实现不要以任何其他用途使用这个参数位置。

toLocaleString 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 toLocaleString 函数是依赖于实现的。

#### Array.prototype.concat ( [ item1 [ , item2 [ , … ] ] ] )

当以零或更多个参数 item1, item2, 等等，调用 concat 方法，返回一个数组，这个数组包含对象的数组元素和后面跟着的每个参数按照顺序组成的数组元素。

采用如下步骤：

1.  令 O 为以 this 值作为参数调用 ToObject 的结果。
2.  令 A 为仿佛是用表达式 new Array() 创建的新数组，这里的 Array 是标准内置构造器名。
3.  令 n 为 0。
4.  令 items 为一个内部列表，他的第一个元素是 O，其后的元素是传给这个函数调用的参数（以从左到右的顺序）。
5.  只要 items 不是空就重复
    1.  删除 items 的第一个元素，并令 E 为这个元素值。
    2.  如果 E 的 [[Class]] 内部属性是 "Array", 则
        1.  令 k 为 0。
        2.  令 len 为以 "length" 为参数调用 E 的 [[Get]] 内部方法的结果。
        3.  只要 k < len 就重复
            1.  令 P 为 ToString(k).
            2.  令 exists 为以 P 作为参数调用 E 的 [[HasProperty]] 内部方法的结果。
            3.  如果 exists 是 true, 则
                1.  令 subElement 为以 P 作为参数调用 E 的 [[Get]] 内部方法的结果。
                2.  以 ToString(n), 属性描述符 {[[Value]]: subElement, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, 和 false 作为参数调用 A 的 [[DefineOwnProperty]] 内部方法。
            4.  n 递增 1.
            5.  k 递增 1.
    3.  否则 , E 不是数组
        1.  以 ToString(n), 属性描述符 {[[Value]]: E, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, 和 false 作为参数调用 A 的 [[DefineOwnProperty]]</nowiki> 内部方法。
        2.  n 递增 1.
6.  返回 A.

concat 方法的 length 属性是 1。

concat 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 concat 函数是依赖于实现的。

#### Array.prototype.join (separator)

数组元素先被转换为字符串，再将这些字符串用 separator 分割连接在一起。如果没提供分隔符，将一个逗号用作分隔符。

join 方法需要一个参数 separator, 执行以下步骤 :

1.  令 O 为以 this 值作为参数调用 ToObject 的结果 .
2.  令 lenVal 为以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
3.  令 len 为 ToUint32(lenVal).
4.  如果 separator 是 undefined, 令 separator 为单字符字符串 ",".
5.  令 sep 为 ToString(separator).
6.  如果 len 是零 , 返回空字符串 .
7.  令 element0 为以 "0" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
8.  如果 element0 是 undefined 或 null, 令 R 为空字符串 ; 否则 , 令 R 为 ToString(element0).
9.  令 k 为 1.
10.  只要 k < len 就重复
    1.  令 S 为串联 R 和 sep 产生的字符串值 .
    2.  令 element 为以 ToString(k) 作为参数调用 O 的 [[Get]] 内部方法的结果 .
    3.  如果 element 是 undefined 或 null, 令 next 为空字符串 ; 否则 , 令 next 为 ToString(element).
    4.  令 R 为串联 S 和 next 产生的字符串值 .
    5.  k 递增 1.
11.  返回 R.

join 方法的 length 属性是 1。

join 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 join 函数是依赖于实现的。

#### Array.prototype.pop ( )

删除并返回数组的最后一个元素。

1.  令 O 为以 this 值作为参数调用 ToObject 的结果 .
2.  令 lenVal 为以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
3.  令 len 为 ToUint32(lenVal).
4.  如果 len 是零 ,
    1.  以 "length", 0, 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
    2.  返回 undefined.
5.  否则 , len > 0
    1.  令 indx 为 ToString(len–1).
    2.  令 element 为 以 indx 作为参数调用 O 的 [[Get]] 内部方法的结果 .
    3.  以 indx 和 true 作为参数调用 O 的 [[Delete]] 内部方法 .
    4.  以 "length", indx, 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
    5.  返回 element.

pop 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 pop 函数是依赖于实现的。

#### Array.prototype.push ( [ item1 [ , item2 [ , … ] ] ] )

将参数以他们出现的顺序追加到数组末尾。数组的新 length 属性值会作为调用的结果返回。

当以零或更多个参数 item1,item2, 等等，调用 push 方法，采用以下步骤：

1.  令 O 为以 this 值作为参数调用 ToObject 的结果 .
2.  令 lenVal 为以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
3.  令 n 为 ToUint32(lenVal).
4.  令 items 为一个内部列表，它的元素是调用这个函数时传入的参数（从左到右的顺序）.
5.  只要 items 不是空就重复
    1.  删除 items 的第一个元素，并令 E 为这个元素的值 .
    2.  以 ToString(n), E, 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
    3.  n 递增 1.
6.  以 "length", n, 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
7.  返回 n.

push 方法的 length 属性是 1。

push 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 push 函数是依赖于实现的。

#### Array.prototype.reverse ( )

重新排列数组元素，以翻转它们的顺序。对象会被当做调用的结果返回。

1.  令 O 为以 this 值作为参数调用 ToObject 的结果 .
2.  令 lenVal 为以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
3.  令 len 为 ToUint32(lenVal).
4.  令 middle 为 floor(len/2).
5.  令 lower 为 0.
6.  只要 lower ≠ middle 就重复
    1.  令 upper 为 len?lower ?1.
    2.  令 upperP 为 ToString(upper).
    3.  令 lowerP 为 ToString(lower).
    4.  令 lowerValue 为以 lowerP 作为参数调用 O 的 [[Get]] 内部方法的结果 .
    5.  令 upperValue 为以 upperP 作为参数调用 O 的 [[Get]] 内部方法的结果 .
    6.  令 lowerExists 为以 lowerP 作为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
    7.  令 upperExists 为以 upperP 作为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
    8.  如果 lowerExists 是 true 并且 upperExists 是 true, 则
        1.  以 lowerP, upperValue, 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
        2.  以 upperP, lowerValue, 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
    9.  否则如果 lowerExists 是 false 并且 upperExists 是 true, 则
        1.  以 lowerP, upperValue, 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
        2.  以 upperP 和 true 作为参数调用 O 的 [[Delete]] 内部方法 .
    10.  否则如果 lowerExists 是 true 并且 upperExists 是 false, 则
        1.  以 lowerP 和 true 作为参数调用 O 的 [[Delete]] 内部方法 .
        2.  以 upperP, lowerValue, 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
    11.  否则 , lowerExists 和 upperExists 都是 false
        1.  不需要做任何事情 .
    12.  lower 递增 1.
7.  返回 O .

reverse 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 reverse 函数是依赖于实现的。

#### Array.prototype.shift ( )

删除并返回数组的第一个元素。

1.  令 O 为以 this 值作为参数调用 ToObject 的结果 .
2.  令 lenVal 为以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
3.  令 len 为 ToUint32(lenVal).
4.  如果 len 是零 , 则
    1.  以 "length", 0, 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
    2.  返回 undefined.
5.  令 first 为以 "0" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
6.  令 k 为 1.
7.  只要 k < len 就重复
    1.  令 from 为 ToString(k).
    2.  令 to 为 ToString(k–1).
    3.  令 fromPresent 为以 from 作为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
    4.  如果 fromPresent 是 true, 则
        1.  令 fromVal 为以 from 作为参数调用 O 的 [[Get]] 内部方法的结果 .
        2.  以 to, fromVal, 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
    5.  否则 , fromPresent 是 false
        1.  以 to 和 ture 作为参数调用 O 的 [[Delete]] 内部方法 .
    6.  k 递增 1.
8.  以 ToString(len–1) 和 true 作为参数调用 O 的 [[Delete]] 内部方法 .
9.  以 "length", (len–1) , 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
10.  返回 first.

shift 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 shift 函数是依赖于实现的。

#### Array.prototype.slice (start, end)

slice 方法需要 start 和 end 两个参数，返回一个数组，这个数组包含从第 start 个元素到 -- 但不包括 -- 第 end 个元素 ( 或如果 end 是 undefined 就到数组末尾 )。如果 start 为负，它会被当做是 length+start，这里的 length 是数组长度。如果 end 为负，它会被当做是 length+end，这里的 length 是数组长度。采用如下步骤：

1.  令 O 为以 this 值作为参数调用 ToObject 的结果 .
2.  令 A 为仿佛用表达式 new Array() 创建的新数组，这里的 Array 是标准内置构造器名 .
3.  令 lenVal 为以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
4.  令 len 为 ToUint32(lenVal).
5.  令 relativeStart 为 ToInteger(start).
6.  如果 relativeStart 为负 , 令 k 为 max((len +relativeStart),0); 否则令 k 为 min(relativeStart,len).
7.  如果 end 是 undefined, 令 relativeEnd 为 len; 否则令 relativeEnd 为 ToInteger(end).
8.  如果 relativeEnd 为负 , 令 final 为 max((len + relativeEnd),0); 否则令 final 为 min(relativeEnd,len).
9.  令 n 为 0.
10.  只要 k < final 就重复
    1.  令 Pk 为 ToString(k).
    2.  令 kPresent 为 Pk 作为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
    3.  如果 kPresent 是 true, 则
        1.  令 kValue 为以 Pk 作为参数调用 O 的 [[Get]] 内部方法的结果 .
        2.  以 ToString(n), 属性描述符 {[[Value]]: kValue, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, 和 false 作为参数调用 A 的 [[DefineOwnProperty]] 内部方法 .
    4.  k 递增 1.
    5.  n 递增 1.
11.  返回 A.

slice 方法的 length 属性是 2。

slice 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 slice 函数是依赖于实现的。

#### Array.prototype.sort (comparefn)

给 this 数组的元素 排序。排序不一定是稳定的（相等的元素们不一定按照他们原来的顺序排列）。如果 comparefn 不是 undefined，它就必须是个函数，这个函数接受两个参数 x 和 y，如果 x < y 返回一个负值，如果 x = y 返回零，如果 x > y 返回一个正值。

令 obj 为以 this 值作为参数调用 ToObject 的结果。

以 "length" 作为参数调用 obj 的 [[Get]] 内部方法，将结果作为参数调用 Uint32，令 len 为返回的结果。

如果 comparefn 不是 undefined 并且不是对 this 数组的元素 保持一致的比较函数（见下面），那么这种情况下 sort 的行为是实现定义的。

令 proto 为 obj 的 [[Prototype]] 内部属性。如果 proto 不是 null 并且存在一个整数 j 满足下面列出的全部条件，那么这种情况下 sort 的行为是实现定义的：

*   obj 是 稀疏的 (15.4)
*   0 ≤ j < len
*   以 ToString(j) 作为参数调用 proto 的 [[HasProperty]] 内部方法的结果是 true

如果 obj 是 稀疏的 并且以下任何条件为真，那么这种情况下 sort 的行为是实现定义的：

*   obj 的 [[Extensible]] 内部属性是 false.
*   任何名为小于 len 的非负整数的数组索引属性中，有 [[Configurable]] 特性是 false 的数据属性。

任何名为小于 len 的非负整数的数组索引属性中，有访问器属性，或有 [[Writable]] 特性是 false 的数据属性，那么这种情况下 sort 的行为是实现定义的。

否则，采用如下步骤。

1.  对 obj 的 [[Get]] , [[Put]], [[Delete]] 内部方法和 SortCompare（下面描述）执行一个依赖于实现的调用序列，这里对每个 [[Get]], [[Put]], 或 [[Delete]] 调用的第一个参数是小于 len 的非负整数 ，SortCompare 调用的参数是前面调用 [[Get]] 内部方法的结果。调用 [[Put]] 和 [[Delete]] 内部方法时，throw 参数是 true。如果 obj 不是 稀疏的 ，则必须不调用 [[Delete]]。
2.  返回 obj。

返回的对象必须拥有下面两个性质。

*   必须有这样的数学排列π，它是由比 len 小的非负整数组成 , 对于每个比 len 小的非负整数 j, 如果属性 old[j] 存在 , 则 new[π(j)] 有与 old[j] 相同的值，如果属性 old[j] 不存在，则 new[π(j)] 也不存在。
*   对于都比 len 小的所有非负整数 j 和 k，如果 SortCompare(j,k) < 0 ( 见下面的 SortCompare), 则π(j) < π(k).

这里的符号 old[j] 用来指：假定在执行这个函数之前以 j 作为参数调用 obj 的 [[Get]] 内部方法的结果，符号 new[j] 用来指：假定在执行这个函数后以 j 作为参数调用 obj 的 [[Get]] 内部方法的结果。

如果对于集合 S 里的任何值 a，b，c（可以是相同值），都满足以下所有条件，那么函数 comparefn 是在集合 S 上保持一致的比较函数（以下，符号 a <[CF] b 表示 comparefn(a,b) < 0；符号 a =[CF] b 表示 comparefn(a,b) = 0（不论正负）； 符号 a >[CF] b 表示 comparefn(a,b) > 0）：

*   当用指定值 a 和 b 作为两个参数调用 comparefn(a,b)，总是返回相同值 v。此外 Type(v) 是 Number, 并且 v 不是 NaN。注意，这意味着对于给定的 a 和 b，a <[CF] b, a =[CF] b, and a >[CF] b 中正好有一个是真。
*   调用 comparefn(a,b) 不改变 this 对象。
*   a =[CF] a ( 自反性 )
*   如果 a =[CF] b, 则 b =[CF] a ( 对称性 )
*   如果 a =[CF] b 并且 b =[CF] c, 则 a =[CF] c (=[CF] 传递 )
*   如果 a <[CF] b 并且 b <[CF] c, 则 a <[CF] c (<[CF] 传递 )
*   如果 a >[CF] b 并且 b >[CF] c, 则 a >[CF] c (>[CF] 传递 )

这些条件是确保 comparefn 划分集合 S 为等价类并且是完全排序等价类的充分必要条件。

当用两个参数 j 和 k 调用抽象操作 SortCompare，采用如下步骤：

1.  令 jString 为 ToString(j).
2.  令 kString 为 ToString(k).
3.  令 hasj 为 以 jString 作为参数调用 obj 的 [[HasProperty]] 内部方法的结果。
4.  令 hask 为 以 kString 作为参数调用 obj 的 [[HasProperty]] 内部方法的结果。
5.  如果 hasj 和 hask 都是 false, 则返回 +0.
6.  如果 hasj 是 false, 则返回 1.
7.  如果 hask 是 false, 则返回 –1.
8.  令 x 为 以 jString 作为参数调用 obj 的 [[Get]] 内部方法的结果。
9.  令 y 为 以 kString 作为参数调用 obj 的 [[Get]] 内部方法的结果。
10.  如果 x 和 y 都是 undefined, 返回 +0.
11.  如果 x 是 undefined, 返回 1.
12.  如果 y 是 undefined, 返回 ?1.
13.  如果 参数 comparefn 不是 undefined, 则
    1.  如果 IsCallable(comparefn) 是 false, 抛出一个 TypeError 异常 .
    2.  传入 undefined 作为 this 值，以 x 和 y 作为参数调用 comparefn 的 [[Call]] 内部方法，返回结果。
14.  令 xString 为 ToString(x).
15.  令 yString 为 ToString(y).
16.  如果 xString < yString, 返回 ?1.
17.  如果 xString > yString, 返回 1.
18.  返回 +0.

因为不存在的属性值总是比 undefined 属性值大，并且 undefined 属性值总是比任何其他值大，所以 undefined 属性值总是排在结果的末尾，后面跟着不存在属性值。

sort 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 sort 函数是依赖于实现的。

#### Array.prototype.splice (start, deleteCount [ , item1 [ , item2 [ , … ] ] ] )

当以两个或更多参数 start, deleteCount 和 ( 可选的 ) item1, item2, 等等，调用 splice 方法，从数组索引 start 开始的 deleteCount 个数组元素会被替换为参数 item1, item2, 等等。返回一个包含参数元素（如果有）的数组。采用以下步骤：

1.  令 O 为 以 this 值作为参数调用 ToObject 的结果 .
2.  令 A 为 仿佛用表达式 new Array() 创建的新数组，这里的 Array 是标准内置构造器名。
3.  令 lenVal 为 以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
4.  令 len 为 ToUint32(lenVal).
5.  令 relativeStart 为 ToInteger(start).
6.  如果 relativeStart 为负 , 令 actualStart 为 max((len + relativeStart),0); 否则令 actualStart 为 min(relativeStart, len).
7.  令 actualDeleteCount 为 min(max(ToInteger(deleteCount),0),len – actualStart).
8.  令 k 为 0.
9.  只要 k < actualDeleteCount 就重复
    1.  令 from 为 ToString(actualStart+k).
    2.  令 fromPresent 为 以 from 作为参数调用 O 的 [[HasProperty]] 内部方法的结果。
    3.  如果 fromPresent 是 true, 则
        1.  令 fromValue 为 以 from 作为参数调用 O 的 [[Get]] 内部方法的结果。
        2.  以 ToString(k), 属性描述符 {[[Value]]: fromValue, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, 和 false 作为参数调用 A 的 [[DefineOwnProperty]] 内部方法。
    4.  k 递增 1.
10.  令 items 为一个内部列表，它的元素是实际参数列表中 item1 开始的参数（从左到右的顺序）。如果没传入这些项目，则列表是空的。
11.  令 itemCount 为 items 的元素个数 .
12.  如果 itemCount < actualDeleteCount, 则
    1.  令 k 为 actualStart.
    2.  只要 k < (len – actualDeleteCount) 就重复
        1.  令 from 为 ToString(k+actualDeleteCount).
        2.  令 to 为 ToString(k+itemCount).
        3.  令 fromPresent 为 以 from 作为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
        4.  如果 fromPresent 是 true, 则
            1.  令 fromValue 为 以 from 作为参数调用 O 的 [[Get]] 内部方法的结果 .
            2.  以 to, fromValue, 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
        5.  否则 , fromPresent 是 false
            1.  以 to 和 true 作为参数调用 O 的 [[Delete]] 内部方法 .
        6.  k 递增 1.
    3.  令 k 为 len.
    4.  只要 k > (len – actualDeleteCount +itemCount) 就重复
        1.  以 ToString(k–1) 和 true 作为参数调用 O 的 [[Delete]] 内部方法 .
        2.  k 递减 1.
13.  否则如果 itemCount > actualDeleteCount, 则
    1.  令 k 为 (len – actualDeleteCount).
    2.  只要 k > actualStart 就重复
        1.  令 from 为 ToString(k + actualDeleteCount – 1).
        2.  令 to 为 ToString(k + itemCount – 1)
        3.  令 fromPresent 为 以 from 作为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
        4.  如果 fromPresent 是 true, 则
            1.  令 fromValue 为 以 from 作为参数调用 O 的 [[Get]] 内部方法的结果 .
            2.  以 to, fromValue, 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
        5.  否则 , fromPresent 是 false
            1.  以 to 和 true 作为参数调用 O 的 [[Delete]] 内部方法 .
        6.  k 递减 1.
14.  令 k 为 actualStart.
15.  只要 items 不是空 就重复
    1.  删除 items 的第一个元素 , 并令 E 为这个元素值 .
    2.  以 ToString(k), E, 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
    3.  k 递增 1.
16.  以 "length", (len – actualDeleteCount + itemCount), 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
17.  返回 A.

splice 方法的 length 属性是 2。

splice 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 splice 函数是依赖于实现的。

#### Array.prototype.unshift ( [ item1 [ , item2 [ , … ] ] ] )

将参数们插入到数组的开始位置，它们在数组中的顺序与它们出现在参数列表中的顺序相同。

当以零或更多个参数 item1,item2, 等等，调用 unshift 方法，采用如下步骤：

1.  令 O 为 以 this 值作为参数调用 ToObject 的结果 .
2.  令 lenVal 为 以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
3.  令 len 为 ToUint32(lenVal).
4.  令 argCount 为 实际参数的个数 .
5.  令 k 为 len.
6.  只要 k > 0, 就重复
    1.  令 from 为 ToString(k–1).
    2.  令 to 为 ToString(k+argCount –1).
    3.  令 fromPresent 为 以 from 作为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
    4.  如果 fromPresent 是 true, 则
        1.  令 fromValue 为 以 from 作为参数调用 O 的 [[Get]] 内部方法的结果 .
        2.  以 to, fromValue, 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
    5.  否则 , fromPresent 是 false
        1.  以 to 和 true 作为参数调用 O 的 [[Delete]] 内部方法 .
    6.  k 递减 1.
7.  令 j 为 0.
8.  令 items 为一个内部列表，它的元素是调用这个函数时传入的实际参数（从左到右的顺序）。
9.  只要 items 不是空，就重复
    1.  删除 items 的第一个元素，并令 E 为这个元素值 .
    2.  以 ToString(j), E, 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
    3.  j 递增 1.
10.  以 "length", len+argCount, 和 true 作为参数调用 O 的 [[Put]] 内部方法 .
11.  返回 len+argCount.

unshift 方法的 length 属性是 1。

unshift 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 unshift 函数是依赖于实现的。

#### Array.prototype.indexOf ( searchElement [ , fromIndex ] )

indexOf 按照索引的升序比较 searchElement 和数组里的元素们，它使用内部的严格相等比较算法 (11.9.6)，如果找到一个或更多这样的位置，返回这些位置中第一个索引；否则返回 -1。

可选的第二个参数 fromIndex 默认是 0（即搜索整个数组）。如果它大于或等于数组长度，返回 -1，即不会搜索数组。如果它是负的，就把它当作从数组末尾到计算后的 fromIndex 的偏移量。如果计算后的索引小于 0，就搜索整个数组。

当用一个或两个参数调用 indexOf 方法 , 采用以下步骤 :

1.  令 O 为 以 this 值作为参数调用 ToObject 的结果 .
2.  令 lenValue 为 以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
3.  令 len 为 ToUint32(lenValue).
4.  如果 len 是 0, 返回 -1.
5.  如果 传入了参数 fromIndex, 则令 n 为 ToInteger(fromIndex); 否则令 n 为 0.
6.  如果 n ≥ len, 返回 -1.
7.  如果 n ≥ 0, 则
    1.  令 k 为 n.
8.  否则 , n<0
    1.  令 k 为 len - abs(n).
    2.  如果 k 小于 0, 则令 k 为 0.
9.  只要 k<len，就重复
    1.  令 kPresent 为 以 ToString(k) 为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
    2.  如果 kPresent 是 true, 则
        1.  令 elementK 为 以 ToString(k) 为参数调用 O 的 [[Get]] 内部方法的结果 .
        2.  令 same 为 对 searchElement 和 elementK 执行严格相等比较算法的结果 .
        3.  如果 same 是 true, 返回 k.
    3.  k 递增 1.
10.  返回 -1.

indexOf 方法的 length 属性是 1。

indexOf 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 indexOf 函数是依赖于实现的。

#### Array.prototype.lastIndexOf ( searchElement [ , fromIndex ] )

lastIndexOf 按照索引的降序比较 searchElement 和数组里的元素们，它使用内部的严格相等比较算法 (11.9.6)，如果找到一个或更多这样的位置，返回这些位置中最后一个索引；否则返回 -1。

可选的第二个参数 fromIndex 默认是数组的长度减一（即搜索整个数组）。如果它大于或等于数组长度，将会搜索整个数组。如果它是负的，就把它当作从数组末尾到计算后的 fromIndex 的偏移量。如果计算后的索引小于 0，返回 -1。

当用一个或两个参数调用 lastIndexOf 方法，采用如下步骤 :

1.  令 O 为 以 this 值作为参数调用 ToObject 的结果 .
2.  令 lenValue 为 以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
3.  令 len 为 ToUint32(lenValue).
4.  如果 len is 0, 返回 -1.
5.  如果 传入了参数 fromIndex, 则令 n 为 ToInteger(fromIndex); 否则令 n 为 len.
6.  如果 n ≥ 0, 则令 k 为 min(n, len – 1).
7.  否则 , n < 0
    1.  令 k 为 len - abs(n).
8.  只要 k≥ 0 就重复
    1.  令 kPresent 为 以 ToString(k) 作为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
    2.  如果 kPresent 是 true, 则
        1.  令 elementK 为 以 ToString(k) 作为参数调用 O 的 [[Get]] 内部方法的结果 .
        2.  令 same 为 对 searchElement 和 elementK 执行严格相等比较算法的结果 .
        3.  如果 same 是 true, 返回 k.
    3.  k 递减 1.
9.  返回 -1.

lastIndexOf 方法的 length 属性是 1。

lastIndexOf 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 lastIndexOf 函数是依赖于实现的。

#### Array.prototype.every ( callbackfn [ , thisArg ] )

callbackfn 应该是个函数，它接受三个参数并返回一个可转换为布尔值 true 和 false 的值。every 按照索引的升序，对数组里存在的每个元素调用一次 callbackfn，直到他找到一个使 callbackfn 返回 false 的元素。如果找到这样的元素，every 马上返回 false，否则如果对所有元素 callbackfn 都返回 true，every 将返回 true。callbackfn 只被数组里实际存在的元素调用；它不会被缺少的元素调用。

如果提供了一个 thisArg 参数，它会被当作 this 值传给每个 callbackfn 调用。如果没提供它，用 undefined 替代。

调用 callbackfn 时将传入三个参数：元素的值，元素的索引，和遍历的对象。

对 every 的调用不直接更改对象，但是对 callbackfn 的调用可能更改对象。

every 处理的元素范围是在首次调用 callbackfn 之前设定的。在 every 调用开始后追加到数组里的元素们不会被 callbackfn 访问。如果更改以存在数组元素，every 访问这些元素时的值会传给 callbackfn；在 every 调用开始后删除的和之前被访问过的元素们是不访问的。every 的行为就像数学量词“所有（for all）”。特别的，对一个空数组，它返回 true。

当以一个或两个参数调用 every 方法，采用以下步骤：

1.  令 O 为 以 this 值作为参数调用 ToObject 的结果 .
2.  令 lenValue 为 以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
3.  令 len 为 ToUint32(lenValue).
4.  如果 IsCallable(callbackfn) 是 false, 抛出一个 TypeError 异常 .
5.  如果提供了 thisArg, 令 T 为 thisArg; 否则令 T 为 undefined.
6.  令 k 为 0.
7.  只要 k < len ，就重复
    1.  令 Pk 为 ToString(k).
    2.  令 kPresent 为 以 Pk 作为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
    3.  如果 kPresent 是 true, 则
        1.  令 kValue 为 以 Pk 作为参数调用 O 的 [[Get]] 内部方法的结果 .
        2.  令 testResult 为 以 T 作为 this 值以包含 kValue, k, 和 O 的参数列表调用 callbackfn 的 [[Call]] 内部方法的结果 .
        3.  如果 ToBoolean(testResult) 是 false, 返回 false.
    4.  k 递增 1.
8.  返回 true.

every 方法的 length 属性是 1。

every 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 every 函数是依赖于实现的。

#### Array.prototype.some ( callbackfn [ , thisArg ] )

callbackfn 应该是个函数，它接受三个参数并返回一个可转换为布尔值 true 和 false 的值。some 按照索引的升序，对数组里存在的每个元素调用一次 callbackfn，直到他找到一个使 callbackfn 返回 true 的元素。如果找到这样的元素，some 马上返回 true，否则，some 返回 false。callbackfn 只被实际存在的数组元素调用；它不会被缺少的数组元素调用。

如果提供了一个 thisArg 参数，它会被当作 this 值传给每个 callbackfn 调用。如果没提供它，用 undefined 替代。

调用 callbackfn 时将传入三个参数：元素的值，元素的索引，和遍历的对象。

对 some 的调用不直接更改对象，但是对 callbackfn 的调用可能更改对象。

some 处理的元素范围是在首次调用 callbackfn 之前设定的。在 some 调用开始后追加到数组里的元素们不会被 callbackfn 访问。如果更改以存在数组元素，some 访问这些元素时的值会传给 callbackfn；在 some 调用开始后删除的和之前被访问过的元素们是不访问的。some 的行为就像数学量词“存在（exists）”。特别的，对一个空数组，它返回 false。

当以一个或两个参数调用 some 方法，采用以下步骤：

1.  令 O 为 以 this 值作为参数调用 ToObject 的结果 .
2.  令 lenValue 为 以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
3.  令 len 为 ToUint32(lenValue).
4.  如果 IsCallable(callbackfn) 是 false, 抛出一个 TypeError 异常 .
5.  如果提供了 thisArg, 令 T 为 thisArg; 否则令 T 为 undefined.
6.  令 k 为 0.
7.  只要 k < len ，就重复
    1.  令 Pk 为 ToString(k).
    2.  令 kPresent 为 以 Pk 作为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
    3.  如果 kPresent 是 true, 则
        1.  令 kValue 为 以 Pk 作为参数调用 O 的 [[Get]] 内部方法的结果 .
        2.  令 testResult 为 以 T 作为 this 值以包含 kValue, k, 和 O 的参数列表调用 callbackfn 的 [[Call]] 内部方法的结果 .
        3.  如果 ToBoolean(testResult) 是 true, 返回 true.
    4.  k 递增 1.
8.  返回 false.

some 方法的 length 属性是 1。

some 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 some 函数是依赖于实现的。

#### Array.prototype.forEach ( callbackfn [ , thisArg ] )

callbackfn 应该是个函数，它接受三个参数。forEach 按照索引的升序，对数组里存在的每个元素调用一次 callbackfn。callbackfn 只被实际存在的数组元素调用；它不会被缺少的数组元素调用。

如果提供了一个 thisArg 参数，它会被当作 this 值传给每个 callbackfn 调用。如果没提供它，用 undefined 替代。

调用 callbackfn 时将传入三个参数：元素的值，元素的索引，和遍历的对象。

对 forEach 的调用不直接更改对象，但是对 callbackfn 的调用可能更改对象。

forEach 处理的元素范围是在首次调用 callbackfn 之前设定的。在 forEach 调用开始后追加到数组里的元素们不会被 callbackfn 访问。如果更改以存在数组元素，forEach 访问这些元素时的值会传给 callbackfn；在 forEach 调用开始后删除的和之前被访问过的元素们是不访问的。

当以一个或两个参数调用 forEach 方法，采用以下步骤：

1.  令 O 为 以 this 值作为参数调用 ToObject 的结果 .
2.  令 lenValue 为 以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
3.  令 len 为 ToUint32(lenValue).
4.  如果 IsCallable(callbackfn) 是 false, 抛出一个 TypeError 异常 .
5.  如果提供了 thisArg, 令 T 为 thisArg; 否则令 T 为 undefined.
6.  令 k 为 0.
7.  只要 k < len ，就重复
    1.  令 Pk 为 ToString(k).
    2.  令 kPresent 为 以 Pk 作为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
    3.  如果 kPresent 是 true, 则
        1.  令 kValue 为 以 Pk 作为参数调用 O 的 [[Get]] 内部方法的结果 .
        2.  以 T 作为 this 值以包含 kValue, k, 和 O 的参数列表调用 callbackfn 的 [[Call]] 内部方法 .
    4.  k 递增 1.
8.  返回 undefined.

forEach 方法的 length 属性是 1。

forEach 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 forEach 函数是依赖于实现的。

#### Array.prototype.map ( callbackfn [ , thisArg ] )

callbackfn 应该是个函数，它接受三个参数。map 按照索引的升序，对数组里存在的每个元素调用一次 callbackfn，并用结果构造一个新数组。callbackfn 只被实际存在的数组元素调用；它不会被缺少的数组元素调用。

如果提供了一个 thisArg 参数，它会被当作 this 值传给每个 callbackfn 调用。如果没提供它，用 undefined 替代。

调用 callbackfn 时将传入三个参数：元素的值，元素的索引，和遍历的对象。

对 map 的调用不直接更改对象，但是对 callbackfn 的调用可能更改对象。

map 处理的元素范围是在首次调用 callbackfn 之前设定的。在 map 调用开始后追加到数组里的元素们不会被 callbackfn 访问。如果更改以存在数组元素，map 访问这些元素时的值会传给 callbackfn；在 map 调用开始后删除的和之前被访问过的元素们是不访问的。

当以一个或两个参数调用 map 方法，采用以下步骤：

1.  令 O 为 以 this 值作为参数调用 ToObject 的结果 .
2.  令 lenValue 为 以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
3.  令 len 为 ToUint32(lenValue).
4.  如果 IsCallable(callbackfn) 是 false, 抛出一个 TypeError 异常 .
5.  如果提供了 thisArg, 令 T 为 thisArg; 否则令 T 为 undefined.
6.  令 A 为 仿佛用 new Array( len) 创建的新数组，这里的 Array 是标准内置构造器名，len 是 len 的值。
7.  令 k 为 0.
8.  只要 k < len ，就重复
    1.  令 Pk 为 ToString(k).
    2.  令 kPresent 为 以 Pk 作为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
    3.  如果 kPresent 是 true, 则
        1.  令 kValue 为 以 Pk 作为参数调用 O 的 [[Get]] 内部方法的结果 .
        2.  令 mappedValue 为 以 T 作为 this 值以包含 kValue, k, 和 O 的参数列表调用 callbackfn 的 [[Call]] 内部方法的结果 .
        3.  以 Pk, 属性描述符 {[[Value]]: mappedValue, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, 和 false 作为参数调用 A 的 [[DefineOwnProperty]] 内部方法 .
    4.  k 递增 1.
9.  返回 A.

map 方法的 length 属性是 1。

map 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 map 函数是依赖于实现的。

#### Array.prototype.filter ( callbackfn [ , thisArg ] )

callbackfn 应该是个函数，它接受三个参数并返回一个可转换为布尔值 true 和 false 的值。filter 按照索引的升序，对数组里存在的每个元素调用一次 callbackfn，并用使 callbackfn 返回 true 的所有值构造一个新数组。callbackfn 只被实际存在的数组元素调用；它不会被缺少的数组元素调用。

如果提供了一个 thisArg 参数，它会被当作 this 值传给每个 callbackfn 调用。如果没提供它，用 undefined 替代。

调用 callbackfn 时将传入三个参数：元素的值，元素的索引，和遍历的对象。

对 filter 的调用不直接更改对象，但是对 callbackfn 的调用可能更改对象。

filter 处理的元素范围是在首次调用 callbackfn 之前设定的。在 filter 调用开始后追加到数组里的元素们不会被 callbackfn 访问。如果更改以存在数组元素，filter 访问这些元素时的值会传给 callbackfn；在 filter 调用开始后删除的和之前被访问过的元素们是不访问的。

当以一个或两个参数调用 filter 方法，采用以下步骤：

1.  令 O 为 以 this 值作为参数调用 ToObject 的结果 .
2.  令 lenValue 为 以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
3.  令 len 为 ToUint32(lenValue).
4.  如果 IsCallable(callbackfn) 是 false, 抛出一个 TypeError 异常 .
5.  如果提供了 thisArg, 令 T 为 thisArg; 否则令 T 为 undefined.
6.  令 A 为 仿佛用 new Array( len) 创建的新数组，这里的 Array 是标准内置构造器名，len 是 len 的值。
7.  令 k 为 0.
8.  令 to 为 0.
9.  只要 k < len ，就重复
    1.  令 Pk 为 ToString(k).
    2.  令 kPresent 为 以 Pk 作为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
    3.  如果 kPresent 是 true, 则
        1.  令 kValue 为 以 Pk 作为参数调用 O 的 [[Get]] 内部方法的结果 .
        2.  令 selected 为 以 T 作为 this 值以包含 kValue, k, 和 O 的参数列表调用 callbackfn 的 [[Call]] 内部方法的结果 .
        3.  如果 ToBoolean(selected) 是 true, 则
            1.  以 ToString(to), 属性描述符 {[[Value]]: kValue, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, 和 false 作为参数调用 A 的 [[DefineOwnProperty]] 内部方法 .
            2.  to 递增 1.
    4.  k 递增 1.
10.  返回 A.

filter 方法的 length 属性是 1。

filter 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 filter 函数是依赖于实现的。

#### Array.prototype.reduce ( callbackfn [ , initialValue ] )

callbackfn 应该是个函数，它需要四个参数。reduce 按照索引的升序，对数组里存在的每个元素 , 将 callbackfn 作为回调函数调用一次。

调用 callbackfn 时将传入四个参数：previousValue（initialValue 的值或上次调用 callbackfn 的返回值），currentValue（当前元素值），currentIndex，和遍历的对象。第一次调用回调函数时，previousValue 和 currentValue 的取值可以是下面两种情况之一。如果为 reduce 调用提供了一个 initialValue，则 previousValue 将等于 initialValue 并且 currentValue 将等于数组的首个元素值。如果没提供 initialValue，则 previousValue 将等于数组的首个元素值并且 currentValue 将等于数组的第二个元素值。如果数组里没有元素并且没有提供 initialValue，则抛出一个 TypeError 异常。

对 reduce 的调用不直接更改对象，但是对 callbackfn 的调用可能更改对象。

reduce 处理的元素范围是在首次调用 callbackfn 之前设定的。在 reduce 调用开始后追加到数组里的元素们不会被 callbackfn 访问。如果更改以存在数组元素，reduce 访问这些元素时的值会传给 callbackfn；在 reduce 调用开始后删除的和之前被访问过的元素们是不访问的。

当以一个或两个参数调用 reduce 方法，采用以下步骤：

1.  令 O 为 以 this 值作为参数调用 ToObject 的结果 .
2.  令 lenValue 为 以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
3.  令 len 为 ToUint32(lenValue ).
4.  如果 IsCallable(callbackfn) 是 false, 抛出一个 TypeError 异常 .
5.  如果 len 是 0 并且 initialValue 不是 present, 抛出一个 TypeError 异常 .
6.  令 k 为 0.
7.  如果 initialValue 参数有传入值 , 则
    1.  设定 accumulator 为 initialValue.
8.  否则 , initialValue 参数没有传入值
    1.  令 kPresent 为 false.
    2.  只要 kPresent 是 false 并且 k < len ，就重复
        1.  令 Pk 为 ToString(k).
        2.  令 kPresent 为 以 Pk 作为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
        3.  如果 kPresent 是 true, 则
            1.  令 accumulator 为 以 Pk 作为参数调用 O 的 [[Get]] 内部方法的结果 .
        4.  k 递增 1.
    3.  如果 kPresent 是 false, 抛出一个 TypeError 异常 .
9.  只要 k < len ，就重复
    1.  令 Pk 为 ToString(k).
    2.  令 kPresent 为 以 Pk 作为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
    3.  如果 kPresent 是 true, 则
        1.  令 kValue 为 以 Pk 作为参数调用 O 的 [[Get]] 内部方法的结果 .
        2.  令 accumulator 为 以 undefined 作为 this 值并以包含 accumulator, kValue, k, 和 O 的参数列表调用 callbackfn 的 [[Call]] 内部方法的结果 .
    4.  k 递增 1.
10.  返回 accumulator.

reduce 方法的 length 属性是 1。

reduce 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 reduce 函数是依赖于实现的。

#### Array.prototype.reduceRight ( callbackfn [ , initialValue ] )

callbackfn 应该是个函数，它需要四个参数。reduceRight 按照索引的升序，对数组里存在的每个元素 , 将 callbackfn 作为回调函数调用一次。

调用 callbackfn 时将传入四个参数：previousValue（initialValue 的值或上次调用 callbackfn 的返回值），currentValue（当前元素值），currentIndex，和遍历的对象。第一次调用回调函数时，previousValue 和 currentValue 的取值可以是下面两种情况之一。如果为 reduceRight 调用提供了一个 initialValue，则 previousValue 将等于 initialValue 并且 currentValue 将等于数组的最后一个元素值。如果没提供 initialValue，则 previousValue 将等于数组的最后一个元素值并且 currentValue 将等于数组的倒数第二个元素值。如果数组里没有元素并且没有提供 initialValue，则抛出一个 TypeError 异常。

对 reduceRight 的调用不直接更改对象，但是对 callbackfn 的调用可能更改对象。

reduceRight 处理的元素范围是在首次调用 callbackfn 之前设定的。在 reduceRight 调用开始后追加到数组里的元素们不会被 callbackfn 访问。如果更改以存在数组元素，reduceRight 访问这些元素时的值会传给 callbackfn；在 reduceRight 调用开始后删除的和之前被访问过的元素们是不访问的。

当以一个或两个参数调用 reduceRight 方法，采用以下步骤：

1.  令 O 为 以 this 值作为参数调用 ToObject 的结果 .
2.  令 lenValue 为 以 "length" 作为参数调用 O 的 [[Get]] 内部方法的结果 .
3.  令 len 为 ToUint32(lenValue ).
4.  如果 IsCallable(callbackfn) 是 false, 抛出一个 TypeError 异常 .
5.  如果 len 是 0 并且 initialValue 不是 present, 抛出一个 TypeError 异常 .
6.  令 k 为 0.
7.  如果 initialValue 参数有传入值 , 则
    1.  设定 accumulator 为 initialValue.
8.  否则 , initialValue 参数没有传入值
    1.  令 kPresent 为 false.
    2.  只要 kPresent 是 false 并且 k ≥ 0 ，就重复
        1.  令 Pk 为 ToString(k).
        2.  令 kPresent 为 以 Pk 作为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
        3.  如果 kPresent 是 true, 则
            1.  令 accumulator 为 以 Pk 作为参数调用 O 的 [[Get]] 内部方法的结果 .
        4.  k 递减 1.
    3.  如果 kPresent 是 false, 抛出一个 TypeError 异常 .
9.  只要 k ≥ 0 ，就重复
    1.  令 Pk 为 ToString(k).
    2.  令 kPresent 为 以 Pk 作为参数调用 O 的 [[HasProperty]] 内部方法的结果 .
    3.  如果 kPresent 是 true, 则
        1.  令 kValue 为 以 Pk 作为参数调用 O 的 [[Get]] 内部方法的结果 .
        2.  令 accumulator 为 以 undefined 作为 this 值并以包含 accumulator, kValue, k, 和 O 的参数列表调用 callbackfn 的 [[Call]] 内部方法的结果 .
    4.  k 递减 1.
10.  返回 accumulator.

reduceRight 方法的 length 属性是 1。

reduceRight 函数被有意设计成通用的；它的 this 值并非必须是数组对象。因此，它可以作为方法转移到其他类型的对象中。一个宿主对象是否可以正确应用这个 reduceRight 函数是依赖于实现的。

### Array 实例的属性

Array 实例从数组原型对象继承属性，Array 实例的 [[Class]] 内部属性是 "Array"。Array 实例还有以下属性。

#### [[DefineOwnProperty]] ( P, Desc, Throw )

数组对象使用一个，用在其他原生 ECMAscript 对象的 [[DefineOwnProperty]] 内部方法 (8.12.9) 的变化版。

设 A 为一个数组对象，Desc 为一个属性描述符，Throw 为一个布尔标示。

在以下算法中，术语“拒绝”指代“如果 Throw 是 true，则抛出 TypeError 异常，否则返回 false。

当用属性名 P，属性描述 Desc，布尔值 Throw 调用 A 的 [[DefineOwnProperty]] 内部方法，采用以下步骤：

1.  令 oldLenDesc 为 以 "length" 作为参数调用 A 的 [[GetOwnProperty]] 内部方法的结果。 结果绝不会是 undefined 或一个访问器描述符，因为在创建数组时的 length 是一个不可删除或重新配置的数据属性。
2.  令 oldLen 为 oldLenDesc.[[Value]].
3.  如果 P 是 "length", 则
    1.  如果 Desc 的 [[Value]] 字段不存在 , 则
        1.  以 "length", Desc, 和 Throw 作为参数在 A 上调用默认的 [[DefineOwnProperty]] 内部方法 (8.12.9)，返回结果。
    2.  令 newLenDesc 为 Desc 的一个拷贝 .
    3.  令 newLen 为 ToUint32(Desc.[[Value]]).
    4.  如果 newLen 不等于 ToNumber( Desc.[[Value]]), 抛出一个 RangeError 异常 .
    5.  设定 newLenDesc.[[Value]] 为 newLen.
    6.  如果 newLen ≥oldLen, 则
        1.  以 "length", newLenDesc, 和 Throw 作为参数在 A 上调用默认的 [[DefineOwnProperty]] 内部方法 (8.12.9)，返回结果。
    7.  如果 oldLenDesc.[[Writable]] 是 false，拒绝
    8.  如果 newLenDesc.[[Writable]] 不存在或值是 true, 令 newWritable 为 true.
    9.  否则 ,
        1.  因为它将使得无法删除任何元素，所以需要延后设定 [[Writable]] 特性为 false.
        2.  令 newWritable 为 false.
        3.  设定 newLenDesc.[[Writable]] 为 true.
    10.  令 succeeded 为 以 "length", newLenDesc, 和 Throw 作为参数在 A 上调用默认的 [[DefineOwnProperty]] 内部方法 (8.12.9) 的结果
    11.  如果 succeeded 是 false, 返回 false..
    12.  只要 newLen < oldLen，就重复 ,
        1.  设定 oldLen 为 oldLen – 1.
        2.  令 deleteSucceeded 为 以 ToString(oldLen) 和 false 作为参数调用 A 的 [[Delete]] 内部方法的结果 .
        3.  如果 deleteSucceeded 是 false, 则
            1.  设定 newLenDesc.[[Value]] 为 oldLen+1.
            2.  如果 newWritable 是 false, 设定 newLenDesc.[[Writable]] 为 false.
            3.  以 "length", newLenDesc, 和 false 为参数在 A 上调用默认的 [[DefineOwnProperty]] 内部方法 (8.12.9).
            4.  拒绝 .
    13.  如果 newWritable 是 false, 则
        1.  以 "length", 属性描述符 {[[Writable]]: false}, 和 false 作为参数在 A 上调用 [[DefineOwnProperty]] 内部方法 (8.12.9). 这个调用始终返回 true.
    14.  返回 true.
4.  否则如果 P 是一个数组索引 (15.4), 则
    1.  令 index 为 ToUint32(P).
    2.  如果 index ≥ oldLen 并且 oldLenDesc.[[Writable]] 是 false，拒绝 .
    3.  令 succeeded 为 以 P, Desc, 和 false 作为参数在 A 上调用默认的 [[DefineOwnProperty]] 内部方法 (8.12.9) 的结果 .
    4.  如果 succeeded 是 false，拒绝 .
    5.  如果 index ≥ oldLen
        1.  设定 oldLenDesc.[[Value]] 为 index + 1.
        2.  以 "length", oldLenDesc, 和 false 作为参数在在 A 上调用默认的 [[DefineOwnProperty]] 内部方法 (8.12.9). 这个调用始终返回 true.
    6.  返回 true.
5.  以 P, Desc, 和 Throw 作为参数在在 A 上调用默认的 [[DefineOwnProperty]] 内部方法 (8.12.9)，返回结果 .

#### length

数组对象的 length 属性是个数据属性，其值总是在数值上大于任何属性名是数组索引的可删除属性的属性名。

length 属性拥有的初始特性是 { [[Writable]]: true, [[Enumerable]]: false, [[Configurable]]: false }.

试图将一个数组对象的 length 属性设定为在数值上比 -- 数组中存在数组索引并且是不可参数属性中的最大数字属性名 -- 小或相等时，length 将设定为比那个最大数字属性名大一的数字子。见 15.4.5.1。

## String 对象

### 作为函数调用 String 构造器

当将 String 作为函数调用，而不是作为构造器，它执行一个类型转换。

#### String ( [ value ] )

返回一个由 ToString(value) 计算出的字符串值（不是 String 对象）。如果没有提供 value，返回空字符串 ""。

### String 构造器

当 String 作为一个 new 表达式的一部分被调用，它是个构造器：它初始化新创建的对象。

#### new String ( [ value ] )

新构造对象的 [[Prototype]] 内部属性设定为标准内置的字符串原型对象，它是 String.prototype 的初始值 (15.5.3.1)。

新构造对象的 [[Class]] 内部属性设定为 "String"。

新构造对象的 [[Extensible]] 内部属性设定为 true。

新构造对象的 [[PrimitiveValue]] 内部属性设定为 ToString(value)，或如果没提供 value 则设定为空字符串。

### String 构造器的属性

String 构造器的 [[Prototype]] 内部属性的值是标准内置的函数原型对象 (15.3.4).

除了内部属性和 length 属性（值为 1）之外，String 构造器还有以下属性：

#### String.prototype

String.prototype 的初始值是标准内置的字符串原型对象 (15.5.4)。

这个属性有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

#### String.fromCharCode ( [ char0 [ , char1 [ , … ] ] ] )

返回一个字符串值，包含的字符数与参数数目相同。每个参数指定返回字符串中的一个字符，也就是说第一个参数第一个字符，以此类推（从左到右）。一个参数转换为一个字符，通过先应用 ToUint16 (9.7) 操作，再将返回的 16 位整数看作字符的代码单元值。如果没提供参数，返回空字符串。

fromCharCode 函数的 length 属性是 1。

### 字符串原型对象的属性

字符串原型对象本身是一个值为空字符串的 String 对象（它的 [[Class]] 是 "String"）。

字符串原型对象的 [[Prototype]] 内部属性值是标准内置的 Object 原型对象 (15.2.4)。

#### String.prototype.constructor

String.prototype.constructor 的初始值是内置 String 构造器。

#### String.prototype.toString ( )

返回 this 字符串值。（注，对于一个 String 对象，toString 方法和 valueOf 方法返回相同值。）

toString 函数是非通用的 ; 如果它的 this 值不是一个字符串或字符串对象，则抛出一个 TypeError 异常。因此它不能作为方法转移到其他类型对象上。

#### String.prototype.valueOf ( )

返回 this 字符串值。

valueOf 函数是非通用的 ; 如果它的 this 值不是一个字符串或字符串对象，则抛出一个 TypeError 异常。因此它不能作为方法转移到其他类型对象上。

#### String.prototype.charAt (pos)

将 this 对象转换为一个字符串，返回包含了这个字符串 pos 位置的字符的字符串。如果那个位置没有字符，返回空字符串。返回结果是个字符串值，不是字符串对象。

如果 pos 是一个数字类型的整数值，则 x.charAt( pos) 与 x.substring( pos, pos+1) 的结果相等。

当用一个参数 pos 调用 charAt 方法，采用以下步骤：

1.  以 this 值作为参数调用 CheckObjectCoercible。
2.  令 S 为以 this 值作为参数调用 ToString 的结果 .
3.  令 position 为 ToInteger(pos).
4.  令 size 为 S 的字符数 .
5.  如果 position < 0 或 position ≥ size, 返回空字符串 .
6.  返回一个长度为 1 的字符串 , 它包含 S 中 position 位置的一个字符 , 在这里 S 中的第一个（最左边）字符被当作是在位置 0，下一个字符被当作是在位置 1，等等 .

charAt 函数被有意设计成通用的；它不要求它的 this 值是字符串对象。因此，他可以当做方法转移到其他类型对象。

#### String.prototype.charCodeAt (pos)

将 this 对象转换为一个字符串，返回一个代表这个字符串 pos 位置字符的代码单元值的数字（小于 2¹⁶ 的非负整数）。如果那个位置没有字符，返回 NaN。

当用一个参数 pos 调用 charCodeAt 方法，采用以下步骤：

1.  以 this 值作为参数调用 CheckObjectCoercible。
2.  令 S 为以 this 值作为参数调用 ToString 的结果 .
3.  令 position 为 ToInteger(pos).
4.  令 size 为 S 的字符数 .
5.  如果 position < 0 或 position ≥ size, 返回 NaN.
6.  返回一个数字类型值，值是字符串 S 中 position 位置字符的代码单元值。 在这里 S 中的第一个（最左边）字符被当作是在位置 0，下一个字符被当作是在位置 1，等等 .

charCodeAt 函数被有意设计成通用的；它不要求它的 this 值是字符串对象。因此，他可以当做方法转移到其他类型对象。

#### String.prototype.concat ( [ string1 [ , string2 [ , … ] ] ] )

当用一个或更多参数 string1, string2, 等等 , 调用 concat 方法 , 它返回一个包含了 --this 对象（转换为一个字符串）中的字符们和后面跟着的每个 string1, string2, 等等，（每个参数都转换为字符串）里的字符们 -- 的字符串。返回结果是一个字符串值，不是一个字符串对象。采用以下步骤：

1.  以 this 值作为参数调用 CheckObjectCoercible。
2.  令 S 为以 this 值作为参数调用 ToString 的结果 .
3.  令 args 为一个内部列表，它是传给这个函数的参数列表的拷贝 .
4.  令 R 为 S.
5.  只要 args 不是空，就重复
    1.  删除 args 的第一个元素，并令 next 为这个元素 .
    2.  令 R 为包含了 -- 之前的 R 值中的字符们和后面跟着的 ToString(next) 结果的字符们 -- 的字符串值。
6.  返回 R.

concat 方法的 length 属性是 1.

concat 函数被有意设计成通用的；它不要求它的 this 值是字符串对象。因此，他可以当做方法转移到其他类型对象。

#### String.prototype.indexOf (searchString, position)

将 this 对象转换为一个字符串，如果 searchString 在这个字符串里大于或等于 position 的位置中的一个或多个位置使它呈现为字符串的子串，那么返回这些位置中最小的索引；否则返回 -1。如果 position 是 undefined，就认为它是 0，以搜索整个字符串。

indexOf 需要两个参数 searchString 和 position，执行以下步骤：

1.  以 this 值作为参数调用 CheckObjectCoercible。
2.  令 S 为以 this 值作为参数调用 ToString 的结果 .
3.  令 searchStr 为 ToString(searchString).
4.  令 pos 为 ToInteger(position). ( 如果 position 是 undefined, 此步骤产生 0).
5.  令 len 为 S 的字符数 .
6.  令 start 为 min(max(pos, 0), len).
7.  令 searchLen 为 SearchStr 的字符数 .
8.  返回 一个不小于 start 的可能的最小值整数 k，使得 k+searchLen 不大于 len，并且对所有小于 searchLen 的非负数整数 j，S 的 k+j 位置字符和 searchStr 的 j 位置字符相同；但如果没有这样的整数 k，则返回 -1。

indexOf 的 length 属性是 1。

indexOf 函数被有意设计成通用的；它不要求它的 this 值是字符串对象。因此，他可以当做方法转移到其他类型对象。

#### String.prototype.lastIndexOf (searchString, position)

将 this 对象转换为一个字符串，如果 searchString 在这个字符串里小于或等于 position 的位置中的一个或多个位置使它呈现为字符串的子串，那么返回这些位置中最大的索引；否则返回 -1。如果 position 是 undefined，就认为它是字符串值的长度，以搜索整个字符串。

lastIndexOf 需要两个参数 searchString 和 position，执行以下步骤：

1.  以 this 值作为参数调用 CheckObjectCoercible。
2.  令 S 为以 this 值作为参数调用 ToString 的结果 .
3.  令 searchStr 为 ToString(searchString).
4.  令 numPos 为 Tonumber(position). ( 如果 position 是 undefined, 此步骤产生 NaN).
5.  如果 numPos 是 NaN, 令 pos 为 +∞; 否则 , 令 pos 为 ToInteger(numPos).
6.  令 len 为 S 的字符数 .
7.  令 start 为 min(max(pos, 0), len).
8.  令 searchLen 为 SearchStr 的字符数 .
9.  返回 一个不大于 start 的可能的最大值整数 k，使得 k+searchLen 不大于 len，并且对所有小于 searchLen 的非负数整数 j，S 的 k+j 位置字符和 searchStr 的 j 位置字符相同；但如果没有这样的整数 k，则返回 -1。

lastIndexOf 的 length 属性是 1。

lastIndexOf 函数被有意设计成通用的；它不要求它的 this 值是字符串对象。因此，他可以当做方法转移到其他类型对象。

#### String.prototype.localeCompare (that)

当以一个参数 that 来调用 localeCompare 方法，它返回一个非 NaN 数字值，这个数字值反应了对 this 值（转换为字符串）和 that 值（转换为字符串）进行语言环境敏感的字符串比较的结果。两个字符串 S 和 That 用实现定义的一种方式进行比较。比较结果是为了按照系统默认语言环境指定的排列顺序来排列字符串，根据按照排列顺序 S 是在 That 前面，相同，还是 S 在 That 后面，结果分别是负数，零，正数。

在执行比较之前执行以下步骤以预备好字符串：

1.  以 this 值作为参数调用 CheckObjectCoercible。
2.  令 S 为以 this 值作为参数调用 ToString 的结果 .
3.  令 That 为 ToString(that).

如果将 localeCompare 方法看做是将 this 和 that 作为两个参数的函数，那么它是在所有字符串集合上的保持一致的比较函数（在 15.4.4.11 定义）。

实际返回值是实现定义的，允许实现者们在返回值里编码附加信息。但是函数需要定义一个在所有字符串上的总的顺序，并且，当比较的字符串们被认为是 Unicode 标准定义的标准等价，则返回 0。

如果宿主环境没有在所有字符串上语言敏感的比较，此函数可执行按位比较。

localeCompare 方法自身不适合直接作为 Array.prototype.sort 的参数，因为后者需要的是两个参数的函数。

这个函数的目的是在宿主环境中任何依靠语言敏感的比较方式都可用于 ECMAScript 环境，并根据宿主环境当前语言环境设置的规则进行比较。强烈建议这个函数将 -- 根据 Unicode 标准的标准等价的 -- 字符串当做是相同的（也就是说，要比较的字符串仿佛是都先被转换为正常化形式 C 或正常化形式 D 了）。还建议这个函数不履行 Unicode 相容等价或分解。

本标准的未来版本可能会使用这个函数的第二个参数；建议实现不将这个参数位用作其他用途。

localeCompare 函数被有意设计成通用的；它不要求它的 this 值是字符串对象。因此，他可以当做方法转移到其他类型对象。

#### String.prototype.match (regexp)

当以 regexp 作为参数调用 match 方法，采用以下步骤：

1.  以 this 值作为参数调用 CheckObjectCoercible。
2.  令 S 为以 this 值作为参数调用 ToString 的结果 .
3.  如果 Type(regexp) 是 Object 并且 regexp 的 [[Class]] 内部属性的值是 "RegExp", 则令 rx 为 regexp;
4.  否则 , 令 rx 为 仿佛是用表达式 new RegExp( regexp) 创建的新正则对象，这里的 RegExp 是标准内置构造器名。
5.  令 global 为 以 "global" 为参数调用 rx 的 [[Get]] 内部方法的结果 .
6.  令 exec 为 标准内置函数 RegExp.prototype.exec ( 见 15.10.6.2)
7.  如果 global 不是 true, 则
    1.  以 rx 作为 this 值，用包含 S 的参数列表调用 exec 的 [[Call]] 内部方法，返回结果。
8.  否则 , global 是 true
    1.  以 "lastIndex" 和 0 作为参数调用 rx 的 [[Put]] 内部方法。
    2.  令 A 为 仿佛是用表达式 new Array() 创建的新数组，这里的 Array 是标准内置构造器名 .
    3.  令 previousLastIndex 为 0.
    4.  令 n 为 0.
    5.  令 lastMatch 为 true.
    6.  只要 lastMatch 是 true，就重复
        1.  令 result 为 以 rx 作为 this 值，用包含 S 的参数列表调用 exec 的 [[Call]] 内部方法的结果。
        2.  如果 result 是 null, 则设定 lastMatch 为 false.
        3.  否则 , result 不是 null
            1.  令 thisIndex 为 以 "lastIndex" 为参数调用 rx 的 [[Get]] 内部方法的结果。
            2.  如果 thisIndex = previousLastIndex 则
                1.  以 "lastIndex" 和 thisIndex+1 为参数调用 rx 的 [[Put]] 内部方法。
                2.  设定 previousLastIndex 为 thisIndex+1.
            3.  否则 , 设定 previousLastIndex 为 thisIndex.
            4.  令 matchStr 为 以 0 为参数调用 result 的 [[Get]] 内部方法的结果。
            5.  以 ToString(n), 属性描述符 {[[Value]]: matchStr, [[Writable]]: true, [[Enumerable]]: true, [[configurable]]: true}, 和 false 作为参数调用 A 的 [[DefineOwnProperty]] 内部方法。
            6.  n 递增 1.
    7.  如果 n = 0, 则返回 null.
    8.  返回 A.

match 函数被有意设计成通用的；它不要求它的 this 值是字符串对象。因此，他可以当做方法转移到其他类型对象。

#### String.prototype.replace (searchValue, replaceValue)

首先根据以下步骤设定 string：

1.  以 this 值作为参数调用 CheckObjectCoercible。
2.  令 string 为 以 this 值作为为参数调用 ToString 的结果。

如果 searchValue 是一个正则表达式（[[Class]] 内部属性是 "RegExp" 的对象），按照如下执行：如果 searchValue.global 是 false，则搜索 string，找出匹配正则表达式 searchValue 的第一个子字符串。如果 searchValue.global 是 true，则搜索 string，找出匹配正则表达式 searchValue 的所有子字符串。搜索的做法与 String.prototype.match 相同，包括对 searchValue.lastIndex 的更新。令 m 为 searchValue 的左捕获括号的个数（使用 15.10.2.1 指定的 NcapturingParens）。

如果 searchValue 不是正则表达式，令 searchString 为 ToString(searchValue)，并搜索 string，找出第一个出现的 searchString 的子字符串。令 m 为 0。

如果 replaceValue 是函数，则对每个匹配的子字符串，以 m + 3 个参数调用这个函数。第一个参数是匹配的子字符串。如果 searchValue 是正则表达式，接下来 m 个参数是 MatchResult（见 15.10.2.1）里的所有捕获值。第 m + 2 个参数是发生的匹配在 string 里的偏移量，第 m + 3 个参数是 string。结果是将输入的原字符串里的每个匹配子字符串替换为相应函数调用的返回值（必要的情况下转换为字符串）得到的字符串。

否则，令 newstring 表示 replaceValue 转换为字符串的结果。结果是将输入的原字符串里的每个匹配子字符串替换为 -- 将 newstring 里的字符替换为表 22 指定的替代文本得到的字符串 -- 得到的字符串。替换这些 $ 是由左到右进行的，并且一旦执行了这样的替换，新替换的文本不受进一步替换。例如 ，"$1,$2".replace(/(\$(\d))/g, "$$1-$1$2") 返回 "$1-$11,$1-$22"。newstring 里的一个 $ ，如果不符合以下任何格式，就保持原状。

替代文本符号替换

| 字符编码值 | 表示 |
| 字符 | 替代文本 |
| $$ | $ |
| $& | 匹配到的子字符串 |
| $`( 译注：'\u0060') | string 中匹配到的子字符串之前部分。 |
| $'( 译注：'\u0027') | string 中匹配到的子字符串之后部分。 |
| $n | 第 n 个捕获结果，n 是范围在 1 到 9 的单个数字，并且紧接着 $n 后面的不是十进制数字。如果 n≤m 且第 n 个捕获结果是 undefined，就用空字符串代替。如果 n>m，结果是实现定义的。 |
| $nn | 第 nn 个捕获结果，nn 是范围在 01 到 99 的十进制两位数。如果 nn≤m 且第 nn 个捕获结果是 undefined，就用空字符串代替。如果 nn>m，结果是实现定义的。 |

replace 函数被有意设计成通用的；它不要求它的 this 值是字符串对象。因此，他可以当做方法转移到其他类型对象。

#### String.prototype.search (regexp)

当用参数 regexp 调用 search 方法，采用以下步骤：

1.  以 this 值作为参数调用 CheckObjectCoercible。
2.  令 string 为 以 this 值作为参数调用 ToString 的结果。
3.  如果 Type(regexp) 是 Object 且 regexp 的 [[Class]] 内部属性的值是 "RegExp", 则令 rx 为 regexp;
4.  否则 , 令 rx 为仿佛是用表达式 new RegExp( regexp) 创建的新正则对象，这里的 RegExp 是标准内置构造器名。
5.  从 string 开始位置搜索正则表达式模式 rx 的匹配。如果找到匹配，令 result 为匹配在 string 里的偏移量；如果没有找到匹配，令 result 为 -1。执行搜索时 regexp 的 lastIndex 和 global 属性是被忽略的。regexp 的 lastIndex 属性保持不变。
6.  返回 result.

search 函数被有意设计成通用的；它不要求它的 this 值是字符串对象。因此，他可以当做方法转移到其他类型对象。

#### String.prototype.slice (start, end)

slice 方法需要两个参数 start 和 end，将 this 对象转换为一个字符串，返回这个字符串中从 start 位置的字符到（但不包含）end 位置的字符的一个子字符串（或如果 end 是 undefined，就直接到字符串尾部）。用 sourceLength 表示字符串长度，如果 start 是负数，就把它看做是 sourceLength+start；如果 end 是负数，就把它看做是 sourceLength+end。返回结果是一个字符串值，不是字符串对象。采用以下步骤：

1.  以 this 值作为参数调用 CheckObjectCoercible。
2.  令 S 为以 this 值作为参数调用 ToString 的结果 .
3.  令 len 为 S 的字符数 .
4.  令 intStart 为 ToInteger(start).
5.  如果 end 是 undefined, 令 intEnd 为 len; 否则 令 intEnd 为 ToInteger(end).
6.  如果 intStart 是 negative, 令 from 为 max(len + intStart,0); 否则 令 from 为 min(intStart,len).
7.  如果 intEnd 是 negative, 令 to 为 max(len +intEnd,0); 否则 令 to 为 min(intEnd, len).
8.  令 span 为 max(to – from,0).
9.  返回 一个包含 --S 中从 form 位置的字符开始的 span 个连续字符 -- 的字符串。

slice 方法的 length 属性是 2。

slice 函数被有意设计成通用的；它不要求它的 this 值是字符串对象。因此，他可以当做方法转移到其他类型对象。

#### String.prototype.split (separator, limit)

将 this 字符串转换为一个字符串，返回一个数组对象，里面存储了这个字符串的子字符串。子字符串是从左到右搜索 separator 的匹配来确定的；这些匹配结果不成为返回数组的任何子字符串元素，但被用来分割字符串。separator 的值可以是一个任意长度的字符串，也可以是一个正则对象（即，一个 [[Class]] 内部属性为 "RegExp" 的对象；见 15.10）。

separator 值可以是一个空字符串，一个空正则表达式，或一个可匹配空字符串的正则表达式。这种情况下，separator 不匹配输入字符串开头和末尾的空的子串，也不匹配分隔符的之前匹配结果末尾的空字串。（例如，如果 separator 是空字符串，要将字符串分割为单个字符们；结果数组的长度等于字符串长度，且每个字串都包含一个字符。）如果 separator 是正则表达式，在 this 字符串的给定位置中只考虑首次匹配结果，即使如果在这个位置上回溯可产生一个非空的子串。（例如，"ab".split(/a*?/) 的执行结果是数组 ["a","b"]，而 "ab".split(/a*/) 的执行结果是数组 ["","b"] 。）

如果 this 对象是（或转换成）空字符串，返回的结果取决于 separator 是否可匹配空字符串。如果可以，结果是不包含任何元素的数组。否则，结果是包含一个空字符串元素的数组。

如果 separator 是包含捕获括号的正则表达式，则对 separator 的每次匹配，捕获括号的结果 ( 包括 undefined) 都拼接为输出数组。例如，

`"Aboldand `coded` ".split(/<(\/)?([^<>]+)>/)`

执行结果是数组：

`["A", undefined, "B", "bold", "/", "B", "and", undefined, "CODE", "coded", "/", "CODE", ""]`

如果 separator 是 undefined，则返回结果是只包含 this 值（转换为字符串）一个字符串元素的数组。如果 limit 不是 undefined，则输出数组被切断为包含不大于 limit 个元素。

当调用 split 方法，采用以下步骤：

1.  以 this 值作为参数调用 CheckObjectCoercible。
2.  令 S 为以 this 值作为参数调用 ToString 的结果 .
3.  令 A 为 仿佛使用表达式 new Array() 创建的新对象，这里的 Array 是标准内置构造器名 .
4.  令 lengthA 为 0.
5.  如果 limit 是 undefined, 令 lim = 2³²–1; 否则 令 lim = ToUint32(limit).
6.  令 s 为 S 的字符数 .
7.  令 p = 0.
8.  如果 separator 是正则对象 ( 它的 [[Class]] 是 "RegExp"), 令 R = separator; 否则，令 R = ToString(separator).
9.  如果 lim = 0, 返回 A.
10.  如果 separator 是 undefined, 则
    1.  以 "0", 属性描述符 {[[Value]]: S, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, 和 false 作为参数调用 A 的 [[DefineOwnProperty]] 内部方法。
    2.  返回 A.
11.  如果 s = 0, 则
    1.  调用 SplitMatch(S, 0, R) 并 令 z 为 它的 MatchResult 结果 .
    2.  如果 z 不是 failure, 返回 A.
    3.  以 "0", 属性描述符 {[[Value]]: S, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, 和 false 作为参数调用 A 的 [[DefineOwnProperty]] 内部方法。
    4.  返回 A.
12.  令 q = p.
13.  只要 q ≠ s，就重复
    1.  调用 SplitMatch(S, q, R) 并 令 z 为 它的 MatchResult 结果 .
    2.  如果 z 是 failure, 则 令 q = q+1.
    3.  否则 , z 不是 failure
        1.  z 必定是一个 State. 令 e 为 z 的 endIndex 并 令 cap 为 z 的 captures 数组 .
        2.  如果 e = p, 则 令 q = q+1.
        3.  否则 , e ≠ p
            1.  令 T 为一个字符串，它的值等于包含 -- 在 S 中从 p（包括它）位置到 q（不包括）位置的字符 -- 的子字符串的值。
            2.  以 ToString(lengthA), 属性描述符 {[[Value]]: T, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, 和 false 作为参数调用 A 的 [[DefineOwnProperty]] 内部方法 .
            3.  lengthA 递增 1.
            4.  如果 lengthA = lim, 返回 A.
            5.  令 p = e.
            6.  令 i = 0.
            7.  只要 i 不等于 cap 中的元素个数，就重复 .
                1.  令 i = i+1.
                2.  以 ToString(lengthA), 属性描述符 {[[Value]]: cap[i], [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, 和 false 作为参数调用 A 的 [[DefineOwnProperty]] 内部方法。
                3.  lengthA 递增 1.
                4.  如果 lengthA = lim, 返回 A.
            8.  令 q = p.
14.  令 T 为 为一个字符串，它的值等于包含 -- 在 S 中从 p（包括它）位置到 q（不包括）位置的字符 -- 的子字符串的值。
15.  以 ToString(lengthA), 属性描述符 {[[Value]]: T, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, 和 false 作为参数调用 A 的 [[DefineOwnProperty]] 内部方法。
16.  返回 A.

SplitMatch 抽象操作需要三个参数，字符串 S，整数 q，字符串或正则对象 R，按照以下顺序执行并返回一个 MatchResult（见 15.10.2.1）：

1.  如果 R 是个正则对象 ( 它的 [[Class]] 是 "RegExp"), 则
    1.  以 S 和 q 作为参数调用 R 的 [[Match]] 内部方法，并返回 MatchResult 的结果。
2.  否则，Type(R) 必定是 String. 令 r 为 R 的字符数 .
3.  令 s 为 S 的字符数 .
4.  如果 q+r > s 则返回 MatchResult failure.
5.  如果存在一个在 0（包括）到 r（不包括）之间的整数 i，使得 S 的 q+i 位置上的字符和 R 的 i 位置上的字符不同，则返回 failure。
6.  令 cap 为 captures 的空数组 ( 见 15.10.2.1).
7.  返回 State 数据结构 (q+r, cap). ( 见 15.10.2.1)

split 方法的 length 属性是 2.

分隔符是正则对象时，split 方法忽略 separator.global 的值。

split 函数被有意设计成通用的；它不要求它的 this 值是字符串对象。因此，他可以当做方法转移到其他类型对象。

#### String.prototype.substring (start, end)

substring 方法需要两个参数 start 和 end，将 this 对象转换为一个字符串，返回包含 -- 在转换结果字符串中从 start 位置字符一直到（但不包括）end 位置的字符（或如果 end 是 undefined，就到字符串末尾）-- 的一个子串。返回结果是字符串值，不是字符串对象。

如果任一参数是 NaN 或负数，它被零取代；如果任一参数大于字符串长度，它被字符串长度取代。

如果 start 大于 end，交换它们的值。

采用以下步骤：

1.  以 this 值作为参数调用 CheckObjectCoercible。
2.  令 S 为以 this 值作为参数调用 ToString 的结果 .
3.  令 len 为 S 的字符数 .
4.  令 intStart 为 ToInteger(start).
5.  如果 end 是 undefined, 令 intEnd 为 len; 否则 令 intEnd 为 ToInteger(end).
6.  令 finalStart 为 min(max(intStart, 0), len).
7.  令 finalEnd 为 min(max(intEnd, 0), len).
8.  令 from 为 min(finalStart, finalEnd).
9.  令 to 为 max(finalStart, finalEnd).
10.  返回 一个长度是 to - from 的字符串，它包含 S 中从索引值 form 到 to-1（按照索引升序）的所有字符。

substring 方法的 length 属性是 2。

substring 函数被有意设计成通用的；它不要求它的 this 值是字符串对象。因此，他可以当做方法转移到其他类型对象。

#### String.prototype.toLowerCase ( )

采用以下步骤：

1.  以 this 值作为参数调用 CheckObjectCoercible。
2.  令 S 为以 this 值作为参数调用 ToString 的结果 .
3.  令 L 为一个字符串，L 的每个字符是 S 中相应字符的 Unicode 小写等量，或者（如果没有 Unicode 小写等量存在）是实际的 S 中相应字符值。
4.  返回 L.

为了此操作，字符串的 16 位代码单元被看作是 Unicode 基本多文种平面（Basic Multilingual Plane）中的代码点。代理代码点直接从 S 转移到 L，不做任何映射。

返回结果必须是根据 Unicode 字符数据库里的大小写映射得到的（对此数据库明确规定，不仅包括 UnicodeData.txt 文件，而且还包括 Unicode 2.1.8 和更高版本里附带的 SpecialCasings.txt 文件）。

某些字符的大小写映射可产生多个字符。这种情况下结果字符串与原字符串的长度未必相等。因为 toUpperCase 和 toLowerCase 都有上下文敏感的行为，所以这俩函数不是对称的。也就是说，s.toUpperCase().toLowerCase() 不一定等于 s.toLowerCase()。

toLowerCase 函数被有意设计成通用的；它不要求它的 this 值是字符串对象。因此，他可以当做方法转移到其他类型对象。

#### String.prototype.toLocaleLowerCase ( )

此函数产生依照 -- 宿主环境的当前语言设置 -- 更正的结果，而不是独立于语言环境的结果，除此之外它的运作方式与 toLowerCase 完全一样。只有在少数情况下有一个区别（如，土耳其语），就是那个语言和正规 Unicode 大小写映射有冲突时的规则。

此函数的第一个参数可能会用于本标准的未来版本 ; 建议实现不以任何用途使用这个参数位置。

toLocaleLowerCase 函数被有意设计成通用的；它不要求它的 this 值是字符串对象。因此，他可以当做方法转移到其他类型对象。

#### String.prototype.toUpperCase ( )

此函数的将字符映射到在 Unicode 字符数据库中与其等值的大写字符，除此之外此函数的行为采用与 String.prototype.toLowerCase 完全相同的方式。

toUpperCase 函数被有意设计成通用的；它不要求它的 this 值是字符串对象。因此，他可以当做方法转移到其他类型对象。

#### String.prototype.toLocaleUpperCase ( )

此函数产生依照 -- 宿主环境的当前语言设置 -- 更正的结果，而不是独立于语言环境的结果，除此之外它的运作方式与 toUpperCase 完全一样。只有在少数情况下有一个区别（如，土耳其语），就是那个语言和正规 Unicode 大小写映射有冲突时的规则。

此函数的第一个参数可能会用于本标准的未来版本 ; 建议实现不以任何用途使用这个参数位置。

toLocaleUpperCase 函数被有意设计成通用的；它不要求它的 this 值是字符串对象。因此，他可以当做方法转移到其他类型对象。

#### String.prototype.trim ( )

采用以下步骤：

1.  以 this 值作为参数调用 CheckObjectCoercible。
2.  令 S 为以 this 值作为参数调用 ToString 的结果 .
3.  令 T 为一个字符串值，它是 S 的一个拷贝，并删除了开头和结尾中空白的。空白的定义是 WhiteSpace 和 LineTerminato r 的并集。
4.  返回 T.

trim 函数被有意设计成通用的；它不要求它的 this 值是字符串对象。因此，他可以当做方法转移到其他类型对象。

### String 实例的属性

字符串实例从字符串原型对象继承属性，字符串实例的 [[Class]] 内部属性值是 "String"。字符串实例还有 [[PrimitiveValue]] 内部属性，length 属性，和一组属性名是数组索引的可遍历属性。

[[PrimitiveValue]] 内部属性是代表这个字符串对象的字符串值。以数组索引命名的属性对应字符串值里的单字符。一个特殊的 [[GetOwnProperty]] 内部方法用来为数组索引命名的属性指定数字，值，和特性。

#### length

在代表这个字符串对象的字符串值里的字符数。

一旦创建了一个字符串对象，这个属性是不可变的。它有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

#### [[GetOwnProperty]] ( P )

数组对象使用一个，用在其他原生 ECMAscript 对象的 [[GetOwnProperty]] 内部方法 (8.12.1) 的变化版。这个特殊内部方法用来给命名属性添加访问器，对应到字符串对象的单字符。

设 S 为一个字符串对象，P 为一个字符串。

当以属性名 P 调用 S 的 [[GetOwnProperty]] 内部方法，采用以下步骤：

1.  令 desc 为 以 P 为参数调用 S 的默认 [[GetOwnProperty]] 内部方法 (8.12.1) 的结果。
2.  如果 desc 不是 undefined，返回 desc.
3.  如果 ToString(abs(ToInteger(P))) 与 P 的值不同 , 返回 undefined.
4.  令 str 为 S 的 [[PrimitiveValue]] 内部属性字符串值。
5.  令 index 为 ToInteger(P).
6.  令 len 为 str 里的字符数 .
7.  如果 len ≤ index, 返回 undefined.
8.  令 resultStr 为一个长度为 1 的字符串 , 里面包含 str 中 index 位置的一个字符 , 在这里 str 中的第一个（最左边）字符被认为是在位置 0，下一个字符在位置 1，依此类推。
9.  返回一个属性描述符 { [[Value]]: resultStr, [[Enumerable]]: true, [[Writable]]: false, [[Configurable]]: false }

## 布尔对象

### 作为函数调用布尔构造器

当把 Boolean 作为函数来调用，而不是作为构造器，它执行一个类型转换。

#### Boolean (value)

返回由 ToBoolean(value) 计算出的布尔值（非布尔对象）。

### 布尔构造器

当 Boolean 作为 new 表达式的一部分来调用，那么它是一个构造器：它初始化新创建的对象。

#### new Boolean (value)

新构造对象的 [[Prototype]] 内部属性设定为原始布尔原型对象，它是 Boolean.prototype (15.6.3.1) 的初始值。

新构造对象的 [[Class]] 内部属性设定为 "Boolean"。

新构造对象的 [[PrimitiveValue]] 内部属性设定为 ToBoolean(value)。

新构造对象的 [[Extensible]] 内部属性设定为 true。

### 布尔构造器的属性

布尔构造器的 [[Prototype]] 内部属性的值是函数原型对象 (15.3.4)。

除了内部属性和 length 属性（值为 1）外，布尔构造器还有以下属性：

#### Boolean.prototype

Boolean.prototype 的初始值是布尔原型对象 (15.6.4)。

这个属性有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

### 布尔原型对象的属性

布尔原型对象自身是一个值为 false 的布尔对象（它的 [[Class]] 是 "Boolean"）。

布尔原型对象的 [[Prototype]] 内部属性值是标准内置的对象原型对象（15.2.4）。

#### Boolean.prototype.constructor

Boolean.prototype.constructor 的初始值是内置的 Boolean 构造器。

#### Boolean.prototype.toString ( )

采用以下步骤：

1.  令 B 为 this 值 .
2.  如果 Type(B) 是 Boolean, 则令 b 为 B.
3.  否则如果 Type(B) 是 Object 且 B 的 [[Class]] 内部属性值是 "Boolean", 则令 b 为 B 的 [[PrimitiveValue]] 内部属性值。
4.  否则抛出一个 TypeError 异常 .
5.  如果 b 是 true, 则返回 "true"; 否则返回 "false".

#### Boolean.prototype.valueOf ( )

采用以下步骤：

1.  令 B 为 this 值 .
2.  如果 Type(B) 是 Boolean, 则令 b 为 B.
3.  否则如果 Type(B) 是 Object 且 B 的 [[Class]] 内部属性值是 "Boolean", 则令 b 为 B 的 [[PrimitiveValue]] 内部属性值。
4.  否则抛出一个 TypeError 异常 .
5.  返回 b.

### 布尔实例的属性

布尔实例从布尔原型对象继承属性，且布尔实例的 [[Class]] 内部属性值是 "Boolean"。布尔实例还有一个 [[PrimitiveValue]] 内部属性。

[[PrimitiveValue]] 内部属性是代表这个布尔对象的布尔值。

## Number 对象

### 作为函数调用的 Number 构造器

当把 Number 当作一个函数来调用，而不是作为构造器，它执行一个类型转换。

#### Number ( [ value ] )

如果提供了 value，返回 ToNumber(value) 计算出的数字值（非 Number 对象），否则返回 +0。

### Number 构造器

当把 Number 作为 new 表达式的一部分来调用，它是构造器：它初始化新创建的对象。

#### new Number ( [ value ] )

新构造对象的 [[Prototype]] 内部属性设定为原始数字原型对象，它是 Number.prototype 的初始值（15.7.3.1）。

新构造对象的 [[Class]] 内部属性设定为 "Number"。

新构造对象的 [[PrimitiveValue]] 内部属性在提供了 value 时设定为 ToNumber(value)，否则设定为 +0。

新构造对象的 [[Extensible]] 内部属性设定为 true。

### Number 构造器的属性

Number 构造器的 [[Prototype]] 内部属性值是函数原型对象 (15.3.4)。

除了内部属性和 length 属性（值为 1）之外，Number 构造器还有以下属性：

#### Number.prototype

Number.prototype 的初始值是数字原型对象。

这个属性有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

#### Number.MAX_VALUE

Number.MAX_VALUE 的值是数字类型的最大正有限值，约为 1.7976931348623157 × 10³⁰⁸。

这个属性有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

#### Number.MIN_VALUE

Number.MIN_VALUE 的值是数字类型的最小正有限值，约为 5 × 10^(-324)。

这个属性有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

#### Number.NaN

Number.NaN 的值是 NaN。

这个属性有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

#### Number.NEGATIVE_INFINITY

Number.NEGATIVE_INFINITY 的值是?∞。

这个属性有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

#### Number.POSITIVE_INFINITY

Number.POSITIVE_INFINITY 的值是 +∞。

这个属性有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

### 数字原型对象的属性

数字原型对象其自身是 Number 对象（其 [[Class]] 是 "Number"），其值为 +0。

数字原型对象的 [[Prototype]] 内部属性值是标准内置 Object 原型对象 (15.2.4)。

除非另外明确声明，以下定义的数字原型对象的方法是非通用的，传递给它们的 this 值必须是数字值或 [[Class]] 内部属性值是 "Number" 的对象。

在以下对作为数字原型对象属性的函数的描述中，短语“this Number 对象”是指函数调用中的 this 值，或如果 Type(this 值 ) 是 Number，“this Number 对象”指仿佛是用表达式 new Number(this value) 创建的对象，这里 Number 是标准内置构造器名。此外，短语“this 数字值”是指代表 this Number 对象的数字值，也就是 this Number 对象的 [[PrimitiveValue]] 内部属性值；或如果 this 是数字类型，“this 数字值”指 this 值。如果 this 值不是 [[Class]] 内部属性为 "Number" 的对象，也不是数字类型的值，则抛出一个 TypeError 异常。

#### Number.prototype.constructor

Number.prototype.constructor 的初始值是内置 Number 构造器。

#### Number.prototype.toString ( [ radix ] )

可选参数 radix 应当是 2 到 36 闭区间上的整数。如果 radix 不存在或是 undefined，用数字 10 作为 radix 的值。如果 ToInteger(radix) 是数字 10，则将 this Number 对象作为一个参数传给 ToString 抽象操作；返回结果字符串值。

如果 ToInteger(radix) 不是在 2 到 36 闭区间上的整数，则抛出一个 RangeError 异常。如果 ToInteger(radix) 是 2 到 36 的整数，担不是 10，则结果是 this 数字值使用指定基数表示法的字符串。字母 a-z 用来指值为 10 到 35 的数字。基数不为 10 时的精确算法是依赖于实现的，然而算法应当是 9.8.1 指定算法的推广形式。

toString 函数不是通用的；如果 this 值不是数字或 Number 对象，抛出一个 TypeError 异常。因此它不能当作方法转移到其他类型对象上。

#### Number.prototype.toLocaleString()

根据宿主环境的当前语言环境惯例来格式化 this 数字值，生成代表这个值的字符串。此函数是依赖于实现的，允许但不鼓励它的返回值与 toString 相同。

此函数的第一个参数可能会用于本标准的未来版本 ; 建议实现不以任何用途使用这个参数位置。

#### Number.prototype.valueOf ( )

返回 this 数字值。

valueOf 函数不是通用的；如果 this 值不是数字或 Number 对象，抛出一个 TypeError 异常。因此它不能当作方法转移到其他类型对象上。

#### Number.prototype.toFixed (fractionDigits)

返回一个包含了 -- 代表 this 数字值的留有小数点后 fractionDigits 个数字的十进制固定小数点记法 -- 的字符串。如果 fractionDigits 是 undefined，就认为是 0。具体来说，执行以下步骤：

1.  令 f 为 ToInteger(fractionDigits). ( 如果 fractionDigits 是 undefined, 此步骤产生 0 值 ).
2.  如果 f < 0 或 f > 20, 抛出一个 RangeError 异常 .
3.  令 x 为 this 数字值 .
4.  如果 x 是 NaN, 返回字符串 "NaN".
5.  令 s 为空字符串 .
6.  如果 x < 0, 则
    1.  令 s 为 "-".
    2.  令 x = –x.
7.  如果 x ≥ 10²¹, 则
    1.  令 m = ToString(x).
8.  否则 , x < 10²¹
    1.  令 n 为一个整数，让 n ÷ 10^f – x 准确的数学值尽可能接近零。如果有两个这样 n 值，选择较大的 n。
    2.  如果 n = 0, 令 m 为字符串 "0". 否则 , 令 m 为由 n 的十进制表示里的数组成的字符串（为了没有前导零）。
    3.  如果 f ≠ 0, 则
        1.  令 k 为 m 里的字符数目 .
        2.  如果 k ≤ f, 则
            1.  令 z 为 f+1–k 个 ‘0’ 组成的字符串 .
            2.  令 m 为 串联字符串 z 的 m 的结果 .
            3.  令 k = f + 1.
        3.  令 a 为 m 的前 k–f 个字符，令 b 为其余 f 个字符。
        4.  令 m 为 串联三个字符串 a, ".", 和 b 的结果。
9.  返回串联字符串 s 和 m 的结果。

toFixed 方法的 length 属性是 1。

如果以多个参数调用 toFixed 方法，则行为是不确定的（见 15 章）。

实现是被允许在 fractionDigits 小于 0 或大于 20 时扩展 toFixed 的行为。在这种情况下，对这样的 fractionDigits 值 toFixed 将未必抛出 RangeError。

对于某些值，toFixed 的输出可比 toString 的更精确，因为 toString 只打印区分相邻数字值的足够的有效数字。例如 ,

(1000000000000000128).toString() 返回 "1000000000000000100",

而 (1000000000000000128).toFixed(0) 返回 "1000000000000000128".

#### Number.prototype.toExponential (fractionDigits)

返回一个代表 this 数字值的科学计数法的字符串，它的有效数字的小数点前有一个数字，有效数字的小数点后有 fractionDigits 个数字。如果 fractionDigits 是 undefined，包括指定唯一数字值需要的尽可能多的有效数字（就像 ToString，但在这里总是以科学计数法输出）。具体来说执行以下步骤：

1.  令 x 为 this 数字值 .
2.  令 f 为 ToInteger(fractionDigits).
3.  如果 x 是 NaN, 返回字符串 "NaN".
4.  令 s 为空字符串 .
5.  如果 x < 0, 则
    1.  令 s 为 "-".
    2.  令 x = –x.
6.  如果 x = +∞, 则
    1.  返回串联字符串 s 和 "Infinity" 的结果 .
7.  如果 fractionDigits 不是 undefined 且 (f < 0 或 f > 20), 抛出一个 RangeError 异常 .
8.  如果 x = 0, 则
    1.  令 f = 0.
    2.  令 m 为包含 f+1 个‘0’的字符串。
    3.  令 e = 0.
9.  否则 , x ≠ 0
    1.  如果 fractionDigits 不是 undefined, 则
        1.  令 e 和 n 为整数，使得满足 10^f ≤ n < 10^(f+1) 且 n × 10^(e–f) – x 的准确数学值尽可能接近零。如果 e 和 n 有两个这样的组合，选择使 n × 10^(e–f) 更大的组合。
    2.  否则 , fractionDigits 是 undefined
        1.  令 e, n, 和 f 为整数，使得满足 f ≥ 0, 10^f≤ n < 10^(f+1), n × 10^(e–f) 的数字值是 x, 且 f 的值尽可能小。注：n 的十进制表示有 f+1 个数字，n 不能被 10 整除，并且 n 的最少有效位数不一定唯一由这些条件确定。
    3.  令 m 为由 n 的十进制表示里的数 组成的字符串（没有前导零）。
10.  如果 f ≠ 0, 则
    1.  令 a 为 m 中的第一个字符 , 令 b 为 m 中的其余字符 .
    2.  令 m 为串联三个字符串 a, ".", 的 b 的结果 .
11.  如果 e = 0, 则
    1.  令 c = "+".
    2.  令 d = "0".
12.  否则
    1.  如果 e > 0, 则 令 c = "+".
    2.  否则 , e ≤ 0
        1.  令 c = "-".
        2.  令 e = –e.
    3.  令 d 为有 e 的十进制表示里的数 组成的字符串（没有前导零）。
13.  令 m 为串联四个字符串 m, "e", c, 和 d 的结果 .
14.  返回串联字符串 s 和 m 的结果 .

toExponential 方法的 length 属性是 1。

如果用多于一个参数调用 toExponential 方法，则行为是未定义的 ( 见 15 章 )。

一个实现可以扩展 fractionDigits 的值小于 0 或大于 20 时 toExponential 的行为。这种情况下对这样的 fractionDigits 值，toExponential 不一定抛出 RangeError 异常 .

对于需要提供比上述规则更准确转换的实现，建议用以下算法作为指引替代步骤 9.b.i ：

1.  令 e, n, 和 f 为整数，使得满足 f ≥ 0, 10^f≤ n < 10^(f+1), n × 10^(e–f) 的数字值是 x, 且 f 的值尽可能小。如果这样的 n 值可能多个，选择使 n × 10^(e–f) 的值尽可能接近 x 的 n 值。如果有两个这样的 n 值，选择偶数。

#### Number.prototype.toPrecision (precision)

返回一个字符串，它代表 this 数字值的科学计数法（有效数字的小数点前有一个数字，有效数字的小数点后有 precision-1 个数字）或十进制固定计数法（precision 个有效数字）。如果 precision 是 undefined，用 ToString (9.8.1) 调用代替。具体来说执行以下步骤：

1.  令 x 为 this 数字值 .
2.  如果 precision 是 undefined, 返回 ToString(x).
3.  令 p 为 ToInteger(precision).
4.  如果 x 是 NaN, 返回字符串 "NaN".
5.  令 s 为空字符串 .
6.  如果 x < 0, 则
    1.  令 s 为 "-".
    2.  令 x = –x.
7.  如果 x = +∞, 则
    1.  返回串联字符串 s 和 "Infinity" 的结果 .
8.  如果 p < 1 或 p > 21, 抛出一个 RangeError 异常 .
9.  如果 x = 0, 则
    1.  令 m 为 p 个 '0' 组成的字符串 .
    2.  令 e = 0.
10.  否则 x ≠ 0,
    1.  令 e 和 n 为整数，使得满足 10^(p–1) ≤ n < 10^p 且 n × 10^(e–p+1) – x 的准确数学值尽可能接近零。如果 e 和 n 有两个这样的组合，选择使 n × 10^(e–p+1) 更大的组合。
    2.  令 m 为由 n 的十进制表示里的数 组成的字符串（没有前导零）。
    3.  如果 e < –6 或 e ≥ p, 则
        1.  令 a 为 n 的第一个字符 , 令 b 为 m 的其余 p–1 个字符 .
        2.  令 m 为串联三个字符串 a, ".", 和 b 的结果 .
        3.  如果 e = 0, 则
            1.  令 c = "+" ，令 d = "0".
        4.  否则 e ≠ 0,
            1.  如果 e > 0, 则
                1.  令 c = "+".
            2.  否则 e < 0,
                1.  令 c = "-".
                2.  令 e = –e.
            3.  令 d 为由 e 的十进制表示里的数 组成的字符串（没有前导零）。
        5.  令 m 为串联五个字符串 s, m, "e", c, 和 d 的结果 .
11.  如果 e = p–1, 则返回串联字符串 s 和 m 的结果 .
12.  如果 e ≥ 0, 则
    1.  令 m 为 m 的前 e+1 个字符 , 字符‘.’, m 的其余 p– (e+1) 个字符 串联的结果 .
13.  否则 e < 0,
    1.  令 m 为 字符串 "0.", –(e+1) 个字符‘0’, 字符串 m 串联的结果 .
14.  返回字符串 s 和 m 串联的结果 .

toPrecision 方法的 length 属性是 1。

如果用多于一个参数调用 toPrecision 方法，则行为是未定义的 ( 见 15 章 )。

一个实现可以扩展 precision 的值小于 1 或大于 21 时 toPrecision 的行为。这种情况下对这样的 precision 值，toPrecision 不一定抛出 RangeError 异常 .

### 数字实例的属性

数字实例从数字原型对象继承属性，数字实例的 [[Class]] 内部属性是 "Number"。数字实例还有一个 [[PrimitiveValue]] 内部属性。

[[PrimitiveValue]] 内部属性是代表 this Number 对象的数字值。

## Math 对象

Math 对象是拥有一些命名属性的单一对象，其中一些属性值是函数。

Math 对象的 [[Prototype]] 内部属性值是标准内置 Object 原型对象 (15.2.4)。Math 对象的 [[Class]] 内部属性值是 "Math"。

Math 对象没有 [[Construct]] 内部属性 ; Math 对象不能作为构造器被 new 运算符调用。

Math 对象没有 [[Call]] 内部属性；Math 对象不能作为函数被调用。

本规范中，短语“x 的数字值”的技术含义定义在 8.5。

### Math 对象的值属性

#### E

自然对数的底数 e 的数字值，约为 2.7182818284590452354。

此属性有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false } 。

#### LN10

10 的自然对数的数字值，约为 2.302585092994046。

此属性有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false } 。

#### LN2

2 的自然对数的数字值，约为 0.6931471805599453。

此属性有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false } 。

#### LOG2E

自然对数的底数 e 的以 2 为底数的对数的数字值；约为 1.4426950408889634。

此属性有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false } 。

Math.LOG2E 的值约为 Math.LN2 值的倒数。

#### LOG10E

自然对数的底数 e 的以 10 为底数的对数的数字值；约为 0.4342944819032518。

此属性有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false } 。

Math.LOG10E 的值约为 Math.LN10 值的倒数。

#### PI

圆的周长与直径之比π的数字值，约为 3.1415926535897932。

此属性有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false } 。

#### SQRT1_2

? 的平方根的数字值，约为 0.7071067811865476。

此属性有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false } 。

Math.SQRT1_2 的值约为 Math.SQRT2 值的倒数。

#### SQRT2

2 的平方根的数字值，约为 1.4142135623730951。

此属性有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false } 。

### Math 对象的函数属性

对以下每个 Math 对象函数的每个参数（如果有多个，以左到右的顺序）应用 ToNumber 抽象操作，然后对结果数字值执行计算。

下面对函数的描述中，符号 NaN, ?0, +0, ?∞, +∞ 指 8.5 描述的数字值。

这里没有精确规定函数 acos, asin, atan, atan2, cos, exp, log, pow, sin, sqrt 的行为，除了需要特别说明对边界情况某些参数值的结果之外。对其他参数值，这些函数旨在计算计算常见数学函数的结果，但选择的近似算法中的某些范围是被允许的。The general intent is that an implementer should be able to use the same mathematical library for ECMAScript on a given hardware platform that is available to C programmers on that platform.

Although the choice of algorithms is left to the implementation, it is recommended (but not specified by this standard) that implementations use the approximation algorithms for IEEE 754 arithmetic contained in fdlibm, the freely distributable mathematical library from Sun Microsystems (http://www.netlib.org/fdlibm).

#### abs (x)

返回 x 的绝对值。

*   若 x 是 NaN, 返回结果是 NaN.
*   若 x 是 ?0, 返回结果是 +0.
*   若 x 是 ?∞, 返回结果是 +∞.

#### acos (x)

返回 x 的反余弦的依赖实现的近似值。结果以弧度形式表示，范围是 +0 到 +π。

*   若 x 是 NaN, 返回结果是 NaN.
*   若 x 大于 1, 返回结果是 NaN.
*   若 x 小于 ?1, 返回结果是 NaN.
*   若 x 正好是 1, 返回结果是 +0.

#### asin (x)

返回 x 的反正弦的依赖实现的近似值。结果以弧度形式表示，范围是?π/2 到 +π/2。

*   若 x 是 NaN, 返回结果是 NaN.
*   若 x 大于 1, 返回结果是 NaN.
*   若 x 小于 –1, 返回结果是 NaN.
*   若 x 是 +0, 返回结果是 +0.
*   若 x 是 ?0, 返回结果是 ?0.

#### atan (x)

返回 x 的反正切的依赖实现的近似值。结果以弧度形式表示，范围是?π/2 到 +π/2。

*   若 x 是 NaN, 返回结果是 NaN.
*   若 x 是 +0, 返回结果是 +0.
*   若 x 是 ?0, 返回结果是 ?0.
*   若 x 是 +∞, 返回结果是 一个依赖于实现的近似值 +π/2.
*   若 x 是 ?∞, 返回结果是 一个依赖于实现的近似值 ?π/2.

#### atan2 (y, x)

返回 -- 参数 y 和 x 的商 y/x-- 的反正切的依赖实现的近似值，y 和 x 的符号用于确定返回值的象限。注：命名为 y 的参数为第一个，命名为 x 的参数为第二个，这是有意，是反正切函数俩参数的惯例。结果以弧度形式表示，范围是?π到 +π。

*   若 x 和 y 至少一个是 NaN, 返回结果是 NaN.
*   若 y>0 且 x 是 +0, 返回结果是 一个依赖于实现的近似值 +π/2.
*   若 y>0 且 x 是 ?0, 返回结果是 一个依赖于实现的近似值 +π/2.
*   若 y 是 +0 且 x>0, 返回结果是 +0.
*   若 y 是 +0 且 x 是 +0, 返回结果是 +0.
*   若 y 是 +0 且 x 是 ?0, 返回结果是 一个依赖于实现的近似值 +π.
*   若 y 是 +0 且 x<0, 返回结果是 一个依赖于实现的近似值 +π.
*   若 y 是 ?0 且 x>0, 返回结果是 ?0.
*   若 y 是 ?0 且 x 是 +0, 返回结果是 ?0.
*   若 y 是 ?0 且 x 是 ?0, 返回结果是 一个依赖于实现的近似值 ?π.
*   若 y 是 ?0 且 x<0, 返回结果是 一个依赖于实现的近似值 ?π.
*   若 y<0 且 x 是 +0, 返回结果是 一个依赖于实现的近似值 ?π/2.
*   若 y<0 且 x 是 ?0, 返回结果是 一个依赖于实现的近似值 ?π/2.
*   若 y>0 且 y 是 有限的 且 x 是 +∞, 返回结果是 +0.
*   若 y>0 且 y 是 有限的 且 x 是 ?∞, 返回结果是 一个依赖于实现的近似值 +π.
*   若 y<0 且 y 是 有限的 且 x 是 +∞, 返回结果是 ?0.
*   若 y<0 且 y 是 有限的 且 x 是 ?∞, 返回结果是 一个依赖于实现的近似值 ?π.
*   若 y 是 +∞ 且 x 是 有限的 , 返回结果是 返回结果是 一个依赖于实现的近似值 +π/2.
*   若 y 是 ?∞ 且 x 是 有限的 , 返回结果是 返回结果是 一个依赖于实现的近似值 ?π/2.
*   若 y 是 +∞ 且 x 是 +∞, 返回结果是 一个依赖于实现的近似值 +π/4.
*   若 y 是 +∞ 且 x 是 ?∞, 返回结果是 一个依赖于实现的近似值 +3π/4.
*   若 y 是 ?∞ 且 x 是 +∞, 返回结果是 一个依赖于实现的近似值 ?π/4.
*   若 y 是 ?∞ 且 x 是 ?∞, 返回结果是 一个依赖于实现的近似值 ?3π/4.

#### ceil (x)

返回不小于 x 的且为数学整数的最小 ( 接近?∞) 数字值。如果 x 已是整数，则返回 x。

*   若 x 是 NaN, 返回结果是 NaN.
*   若 x 是 +0, 返回结果是 +0.
*   若 x 是 ?0, 返回结果是 ?0.
*   若 x 是 +∞, 返回结果是 +∞.
*   若 x 是 ?∞, 返回结果是 ?∞.
*   若 x 小于 0 但大于 -1, 返回结果是 ?0.

Math.ceil(x) 的值与 -Math.floor(-x) 的值相同。

#### cos (x)

返回 x 的余弦的依赖实现的近似值。参数被当做是弧度值。

*   若 x 是 NaN, 返回结果是 NaN.
*   若 x 是 +0, 返回结果是 1.
*   若 x 是 ?0, 返回结果是 1.
*   若 x 是 +∞, 返回结果是 NaN.
*   若 x 是 ?∞, 返回结果是 NaN.

#### exp (x)

返回 x 的指数的依赖实现的近似值（e 为 x 次方，e 为自然对数的底）。

*   若 x 是 NaN, 返回结果是 NaN.
*   若 x 是 +0, 返回结果是 1.
*   若 x 是 ?0, 返回结果是 1.
*   若 x 是 +∞, 返回结果是 +∞.
*   若 x 是 ?∞, 返回结果是 +0.

#### floor (x)

返回不大于 x 的且为数学整数的最大 ( 接近 +∞) 数字值。如果 x 已是整数，则返回 x。

*   若 x 是 NaN, 返回结果是 NaN.
*   若 x 是 +0, 返回结果是 +0.
*   若 x 是 ?0, 返回结果是 ?0.
*   若 x 是 +∞, 返回结果是 +∞.
*   若 x 是 ?∞, 返回结果是 ?∞.
*   若 x 大于 0 但小于 1, 返回结果是 +0.

Math.floor(x) 的值与 -Math.ceil(-x) 的值相同。

#### log (x)

返回 x 的自然对数的依赖于实现的近似值 .

*   若 x 是 NaN, 返回结果是 NaN.
*   若 x 小于 0, 返回结果是 NaN.
*   若 x 是 +0 或 ?0, 返回结果是 ?∞.
*   若 x 是 1, 返回结果是 +0.
*   若 x 是 +∞, 返回结果是 +∞.

#### max ( [ value1 [ , value2 [ , … ] ] ] )

给定零或多个参数，对每个参数调用 ToNumber 并返回调用结果里的最大值。

*   若 没有给定参数 , 返回结果是 ?∞.
*   若 任何值是 NaN, 返回结果是 NaN.
*   按照 11.8.5 指定方式进行值比较，确定最大值，与 11.8.5 指定方式的一个不同点是在这里 +0 被看作大于 -0.

max 方法的 length 属性是 2。

#### min ( [ value1 [ , value2 [ , … ] ] ] )

给定零或多个参数，对每个参数调用 ToNumber 并返回调用结果里的最小值。

*   若 没有给定参数 , 返回结果是 +∞.
*   若 任何值是 NaN, 返回结果是 NaN.
*   按照 11.8.5 指定方式进行值比较，确定最小值，与 11.8.5 指定方式的一个不同点是在这里 +0 被看作大于 -0.

min 方法的 length 属性是 2。

#### pow (x, y)

返回 x 的 y 次方的依赖于实现的近似值 .

*   若 y 是 NaN, 返回结果是 NaN.
*   若 y 是 +0, 返回结果是 1, 即使 x 是 NaN.
*   若 y 是 ?0, 返回结果是 1, 即使 x 是 NaN.
*   若 x 是 NaN 且 y 是 非零 , 返回结果是 NaN.
*   若 abs(x)>1 且 y 是 +∞, 返回结果是 +∞.
*   若 abs(x)>1 且 y 是 ?∞, 返回结果是 +0.
*   若 abs(x)==1 且 y 是 +∞, 返回结果是 NaN.
*   若 abs(x)==1 且 y 是 ?∞, 返回结果是 NaN.
*   若 abs(x)<1 且 y 是 +∞, 返回结果是 +0.
*   若 abs(x)<1 且 y 是 ?∞, 返回结果是 +∞.
*   若 x 是 +∞ 且 y>0, 返回结果是 +∞.
*   若 x 是 +∞ 且 y<0, 返回结果是 +0.
*   若 x 是 ?∞ 且 y>0 且 y 是 一个奇数 , 返回结果是 ?∞.
*   若 x 是 ?∞ 且 y>0 且 y 不是 一个奇数 , 返回结果是 +∞.
*   若 x 是 ?∞ 且 y<0 且 y 是 一个奇数 , 返回结果是 ?0.
*   若 x 是 ?∞ 且 y<0 且 y 不是 一个奇数 , 返回结果是 +0.
*   若 x 是 +0 且 y>0, 返回结果是 +0.
*   若 x 是 +0 且 y<0, 返回结果是 +∞.
*   若 x 是 ?0 且 y>0 且 y 是 一个奇数 , 返回结果是 ?0.
*   若 x 是 ?0 且 y>0 且 y 不是 一个奇数 , 返回结果是 +0.
*   若 x 是 ?0 且 y<0 且 y 是 一个奇数 , 返回结果是 ?∞.
*   若 x 是 ?0 且 y<0 且 y 不是 一个奇数 , 返回结果是 +∞.
*   若 x<0 且 x 是 有限的 且 y 是 有限的 and y 不是整数 , 返回结果是 NaN.

#### random ( )

返回一个大于或等于 0 但小于 1 的符号为正的数字值，选择随机或在该范围内近似均匀分布的伪随机，用一个依赖与实现的算法或策略。此函数不需要参数。

#### round (x)

返回最接近 x 且为数学整数的数字值。如果两个整数同等接近 x，则结果是接近 +∞的数字值 。如果 x 已是整数，则返回 x。

*   若 x 是 NaN, 返回结果是 NaN.
*   若 x 是 +0, 返回结果是 +0.
*   若 x 是 ?0, 返回结果是 ?0.
*   若 x 是 +∞, 返回结果是 +∞.
*   若 x 是 ?∞, 返回结果是 ?∞.
*   若 x 大于 0 但小于 0.5, 返回结果是 +0.
*   若 x 小于 0 但大于或等于 -0.5, 返回结果是 ?0.

Math.round(3.5) 返回 4, 但 Math.round(–3.5) 返回 –3.

当 x 为 ?0 或 x 小于 0 当大于大于等于 -0.5 时，Math.round(x) 返回 ?0, 但 Math.floor(x+0.5) 返回 +0，除了这种情况之外 Math.round(x) 的返回值与 Math.floor(x+0.5) 的返回值相同。

#### sin (x)

返回 x 的正弦的依赖实现的近似值。参数被当做是弧度值。

*   若 x 是 NaN, 返回结果是 NaN.
*   若 x 是 +0, 返回结果是 +0.
*   若 x 是 ?0, 返回结果是 ?0.
*   若 x 是 +∞ 或 ?∞, 返回结果是 NaN.

#### sqrt (x)

返回 x 的平方根的依赖实现的近似值。

*   若 x 是 NaN, 返回结果是 NaN.
*   若 x 小于 0, 返回结果是 NaN.
*   若 x 是 +0, 返回结果是 +0.
*   若 x 是 ?0, 返回结果是 ?0.
*   若 x 是 +∞, 返回结果是 +∞.

#### tan (x)

返回 x 的正切的依赖实现的近似值。参数被当做是弧度值。

*   若 x 是 NaN, 返回结果是 NaN.
*   若 x 是 +0, 返回结果是 +0.
*   若 x 是 ?0, 返回结果是 ?0.
*   若 x 是 +∞ 或 ?∞, 返回结果是 NaN.

## Date 对象

### Date 对象的概述和抽象操作的定义

下面的抽象操作函数用来操作时间值（15.9.1.1 定义）。注：任何情况下，如果这些函数之一的任意参数是 NaN，则结果将是 NaN。

#### 时间值和时间范围

一个 Date 对象包含一个表示特定时间瞬间的毫秒的数字值。这样的数字值叫做 时间值 。一个时间值也可以是 NaN，说明这个 Date 对象不表示特定时间瞬间。

ECMAScript 中测量的时间是从协调世界时 1970 年 1 月 1 日开始的毫秒数。在时间值中闰秒是被忽略的，假设每天正好有 86,400,000 毫秒。ECMAScript 数字值可表示的所有从–9,007,199,254,740,991 到 9,007,199,254,740,991 的整数；这个范围足以衡量协调世界时 1970 年 1 月 1 日前后约 285,616 年内任何时间瞬间的精确毫秒。

ECMAScript Date 对象支持的实际时间范围是略小一些的：相对协调世界时 1970 年 1 月 1 日午夜 0 点的精确的–100,000,000 天到 100,000,000 天。这给出了协调世界时 1970 年 1 月 1 日前后 8,640,000,000,000,000 毫秒的范围。

精确的协调世界时 1970 年 1 月 1 日午夜 0 点用 +0 表示。

#### 天数和天内时间

一个给定时间值 t 所属的天数是

`Day(t) = floor(t / msPerDay)`

其中每天的毫秒数是

`msPerDay = 86400000`

余数叫做天内时间

`TimeWithinDay(t) = t modulo msPerDay`

#### 年数

ECMAScript 使用一个推算公历系统，来将一个天数映射到一个年数，并确定在那年的月份的日期。在这个系统中，闰年是且仅是（可被 4 整除）且（（不可被 100 整除）或（可被 400 整除））的年份。因此，y 年的天的数目定义为

`DaysInYear(y) = 365 { 如果 (y 取模 4) ≠ 0 } = 366 { 如果 (y 取模 4) = 0 且 (y 取模 100) ≠ 0 } = 365 { 如果 (y 取模 100) = 0 且 (y 取模 400) ≠ 0 } = 366 { 如果 (y 取模 400) = 0 }`

所有非闰年有 365 天，其中每月的天的数目是常规的。闰年的二月里有个多出来的一天。 y 年第一天的天数是 :

`DayFromYear(y) = 365 × (y?1970) + floor((y?1969)/4) ? floor((y?1901)/100) + floor((y?1601)/400)`

y 年的起始时间值是：

`TimeFromYear(y) = msPerDay × DayFromYear(y)`

一个时间值决定的年数是：

`YearFromTime(t) = 满足条件 TimeFromYear(y) ≤ t 的最大整数 y （接近正无穷）`

若时间值在闰年内，闰年函数返回 1，否则返回 0：

`InLeapYear(t) = 0 { 如果 DaysInYear(YearFromTime(t)) = 365 } = 1 { 如果 DaysInYear(YearFromTime(t)) = 366 }`

#### 月数

月份是由闭区间 0 到 11 内的一个整数确定。一个时间值 t 到一个月数的映射 MonthFromTime(t) 的定义为：

`MonthFromTime(t) = 0 if 0 ≤ DayWithinYear(t) < 31 = 1 if 31 ≤ DayWithinYear (t) < 59+InLeapYear(t) = 2 if 59+InLeapYear(t) ≤ DayWithinYear (t) < 90+InLeapYear(t) = 3 if 90+InLeapYear(t) ≤ DayWithinYear (t) < 120+InLeapYear(t) = 4 if 120+InLeapYear(t) ≤ DayWithinYear (t) < 151+InLeapYear(t) = 5 if 151+InLeapYear(t) ≤ DayWithinYear (t) < 181+InLeapYear(t) = 6 if 181+InLeapYear(t) ≤ DayWithinYear (t) < 212+InLeapYear(t) = 7 if 212+InLeapYear(t) ≤ DayWithinYear (t) < 243+InLeapYear(t) = 8 if 243+InLeapYear(t) ≤ DayWithinYear (t) < 273+InLeapYear(t) = 9 if 273+InLeapYear(t) ≤ DayWithinYear (t) < 304+InLeapYear(t) = 10 if 304+InLeapYear(t) ≤ DayWithinYear (t) < 334+InLeapYear(t) = 11 if 334+InLeapYear(t) ≤ DayWithinYear (t) < 365+InLeapYear(t)`

其中

`DayWithinYear(t) = Day(t)?DayFromYear(YearFromTime(t))`

月数值 0 指一月；1 指二月；2 指三月；3 指四月；4 指五月；5 指六月；6 指七月；7 指八月；8 指九月；9 指十月；10 指十一月；11 指十二月。注：MonthFromTime(0) = 0，对应 1970 年 1 月 1 日，星期四。

#### 日期数

一个日期数用闭区间 1 到 31 内的一个整数标识。从一个时间值 t 到一个日期数的映射 DateFromTime(t) 的定义为：

`DateFromTime(t) = DayWithinYear(t)+1 if MonthFromTime(t)=0 = DayWithinYear(t)?30 if MonthFromTime(t)=1 = DayWithinYear(t)?58?InLeapYear(t) if MonthFromTime(t)=2 = DayWithinYear(t)?89?InLeapYear(t) if MonthFromTime(t)=3 = DayWithinYear(t)?119?InLeapYear(t) if MonthFromTime(t)=4 = DayWithinYear(t)?150?InLeapYear(t) if MonthFromTime(t)=5 = DayWithinYear(t)?180?InLeapYear(t) if MonthFromTime(t)=6 = DayWithinYear(t)?211?InLeapYear(t) if MonthFromTime(t)=7 = DayWithinYear(t)?242?InLeapYear(t) if MonthFromTime(t)=8 = DayWithinYear(t)?272?InLeapYear(t) if MonthFromTime(t)=9 = DayWithinYear(t)?303?InLeapYear(t) if MonthFromTime(t)=10 = DayWithinYear(t)?333?InLeapYear(t) if MonthFromTime(t)=11`

#### 星期数

特定时间值 t 对应的星期数的定义为：

`WeekDay(t) = (Day(t) + 4) modulo 7`

星期数的值 0 指星期日；1 指星期一；2 指星期二；3 指星期三；4 指星期四；5 指星期五；6 指星期六。注：WeekDay(0) = 4, 对应 1970 年 1 月 01 日 星期四。

#### 本地时区校准

期望一个 ECMAScript 的实现确定本地时区校准。本地时区校准是一个毫秒为单位的值 LocalTZA，它加上 UTC 代表本地标准时间。LocalTZA 不体现夏令时。LocalTZA 值不随时间改变，但只取决于地理位置。

#### 夏令时校准

期望一个 ECMAScript 的实现确定夏令时算法。确定夏令时校准的算法 DaylightSavingTA(t)，以毫秒为单位，必须只依赖下面四个项目：

(1) 自本年开始以来的时间

`t – TimeFromYear(YearFromTime(t))`

(2) t 是否在闰年内

`InLeapYear(t)`

(3) 本年第一天的星期数

`WeekDay(TimeFromYear(YearFromTime(t))`

(4) 地理位置。

The implementation of ECMAScript should not try to determine whether the exact time was subject to daylight saving time, but just whether daylight saving time would have been in effect if the current daylight saving time algorithm had been used at the time. This avoids complications such as taking into account the years that the locale observed daylight saving time year round.

If the host environment provides functionality for determining daylight saving time, the implementation of ECMAScript is free to map the year in question to an equivalent year (same leap-year-ness and same starting week day for the year) for which the host environment provides daylight saving time information. The only restriction is that all equivalent years should produce the same result.

#### 本地时间

从协调世界时到本地时间的转换，定义为

`LocalTime(t) = t + LocalTZA + DaylightSavingTA(t)`

从本地时间到协调世界时的转换，定义为

`UTC(t) = t – LocalTZA – DaylightSavingTA(t – LocalTZA)`

UTC(LocalTime(t)) 不一定总是等于 t。

#### 小时 , 分钟 , 秒 , 毫秒

以下函数用于分解时间值：

`HourFromTime(t) = floor(t / msPerHour) modulo HoursPerDay MinFromTime(t) = floor(t / msPerMinute) modulo MinutesPerHour SecFromTime(t) = floor(t / msPerSecond) modulo SecondsPerMinute msFromTime(t) = t modulo msPerSecond`

其中

`HoursPerDay = 24 MinutesPerHour = 60 SecondsPerMinute = 60 msPerSecond = 1000 msPerMinute = 60000 = msPerSecond × SecondsPerMinute msPerHour = 3600000 = msPerMinute × MinutesPerHour`

#### MakeTime (hour, min, sec, ms)

MakeTime 抽象操作用它的四个参数算出一个毫秒数，参数必须是 ECMAScript 数字值。此抽象操作运行如下：

1.  如果 hour 不是有限的或 min 不是有限的或 sec 不是有限的或 ms 不是有限的 , 返回 NaN。
2.  令 h 为 ToInteger(hour).
3.  令 m 为 ToInteger(min).
4.  令 s 为 ToInteger(sec).
5.  令 milli 为 ToInteger(ms).
6.  令 t 为 h * msPerHour + m * msPerMinute + s * msPerSecond + milli, 执行的四则运算根据 IEEE 754 规则（这就像使用 ECMAScript 运算符 * 和 + 一样）。
7.  返回 t。

#### MakeDay (year, month, date)

MakeDay 抽象操作用它的三个参数算出一个天数，参数必须是 ECMAScript 数字值。此抽象操作运行如下：

1.  如果 year 不是有限的或 month 不是有限的或 date 不是有限的 , 返回 NaN.
2.  令 y 为 ToInteger(year).
3.  令 m 为 ToInteger(month).
4.  令 dt 为 ToInteger(date).
5.  令 ym 为 y + floor(m /12).
6.  令 mn 为 m modulo 12.
7.  找一个满足 YearFromTime(t) == ym 且 MonthFromTime(t) == mn 且 DateFromTime(t) == 1 的 t 值 ; 但如果这些条件是不可能的（因为有些参数超出了范围）, 返回 NaN.
8.  返回 Day(t) + dt ? 1.

#### MakeDate (day, time)

MakeDate 抽象操作用它的两个参数算出一个毫秒数，参数必须是 ECMAScript 数字值。此抽象操作运行如下：

1.  如果 day 不是有限的或 time 不是有限的 , 返回 NaN.
2.  返回 day × msPerDay + time.

#### TimeClip (time)

TimeClip 抽象操作用它的参数算出一个毫秒数，参数必须是 ECMAScript 数字值。此抽象操作运行如下：

1.  如果 time 不是有限的 , 返回 NaN.
2.  如果 abs(time) > 8.64 x 1015, 返回 NaN.
3.  返回 ToInteger(time) 和 ToInteger(time) + (+0) 之一，这依赖于实现 ( 加正一是为了将 ?0 转换成 +0)。

第 3 步的重点是说允许实现自行选择时间值的内部表示形式，如 64 位有符号整数或 64 位浮点数。根据不同的实现，这个内部表示可能区分也可能无法区分 ?0 和 +0。

#### 日期时间字符串格式

ECMAScript 定义了一个基于简化的 ISO 8601 扩展格式的日期时间的字符串互换格式，格式为：YYYY-MM-DDTHH:mm:ss.sssZ

其中个字段为：

YYYY
是公历中年的十进制数字。

-
在字符串中直接以“-” ( 破折号 ) 出现两次。

MM
是一年中的月份，从 01 ( 一月 ) 到 12 ( 十二月 )。

DD
是月份中的日期，从 01 到 30。

T
在字符串中直接以“T”出现，用来表明时间元素的开始。

HH
是用两个十进制数字表示的，自午夜 0 点以来的小时数。

:
在字符串中直接以“:” ( 冒号 ) 出现两次。

mm
是用两个十进制数字表示的，自小时开始以来的分钟数。

ss
是用两个十进制数字表示的，自分开始以来的秒数。

.
在字符串中直接以“.” ( 点 ) 出现。

sss
是用三个十进制数字表示的，自秒开始以来的毫秒数。

Z
是时区偏移量，由（“Z” ( 指 UTC) 或“+” 或 “-”）和后面跟着的时间表达式 hh:mm 组成。

这个格式包括只表示日期的形式：

`YYYY YYYY-MM YYYY-MM-DD`

这个格式还包括“日期时间”形式，它由上面的只表示日期的形式之一和紧跟在后面的“T”和以下时间形式之一和可选的时区偏移量组成：

`THH:mm THH:mm:ss THH:mm:ss.sss`

所有数字必须是 10 进制的。如果缺少 MM 或 DD 字段，用“01”作为它们的值。如果缺少 mm 或 ss 字段，用“00”作为它们的值，对于缺少的 sss 用“000”作为它的值。对于缺少的时区偏移量用“Z”。

一个格式字符串里有非法值（越界以及语法错误），意味着这个格式字符串不是有效的本节描述格式的实例。

由于每天的开始和结束都在午夜，俩符号 00:00 和 24:00 可区分这样的可以是同一时间的两个午夜。这意味着两个符号 1995-02-04T24:00 和 1995-02-05T00:00 精准的指向同一时刻。

不存在用来规范像 CET, EST 这样的民间时区缩写的国际标准。有时相同的缩写甚至使用不同的时区。出于这个原因，ISO 8601 和这里的格式指定数字来表示时区。

##### 扩展的年

ECMAScript 需要能表示 6 位数年份（扩展的年份）的能力；协调世界时 1970 年 1 月 1 日前后分别约 285,616 年。对于表示 0 年之前或 9999 年之后的年份，ISO 8601 允许对年的表示法进行扩展，但只能在发送和接受信息的双方有事先共同约定的情况下才能扩展。在已经简化的 ECMAScript 的格式中这样扩展的年份表示法有 2 个额外的数字和始终存在的前缀符号 + 或 - 。0 年被认为是正的，因此用 + 符号作为前缀。

扩展年的示例

`-283457-03-21T15:00:59.008Z 283458 B.C. -000001-01-01T00:00:00Z 2 B.C. +000000-01-01T00:00:00Z 1 B.C. +000001-01-01T00:00:00Z 1 A.D. +001970-01-01T00:00:00Z 1970 A.D. +002009-12-15T00:00:00Z 2009 A.D. +287396-10-12T08:59:00.992Z 287396 A.D.`

### 作为函数调用 Date 构造器

当把 Date 作为函数来调用，而不作为构造器，它返回一个表示当前时间（协调世界时）的字符串。

函数调用 Date(…) 的结果和用相同参数调用表达式 new Date(…) 创建的对象是不同的。

#### Date ( [ year [, month [, date [, hours [, minutes [, seconds [, ms ] ] ] ] ] ] ] )

所有参数都是可选的；接受提供的任何参数，但被完全忽略。返回一个仿佛是用表达式 (new Date()).toString() 创建的字符串，这里的 Date 是标准内置构造器，toString 是标准内置方法 Date.prototype.toString。

### Date 构造器

当把 Date 作为 new 表达式的一部分来调用，它是个构造器：它初始化新创建的对象。

#### new Date (year, month [, date [, hours [, minutes [, seconds [, ms ] ] ] ] ] )

当用二到七个参数调用 Date 构造器，它用 year, month, 还有 ( 可选的 ) date, hours, minutes, seconds, ms 来计算时间。

新构造对象的 [[Prototype]] 内部属性设定为原始的时间原型对象，它是 Date.prototype(15.9.4.1) 的初始值。

新构造对象的 [[Class]] 内部属性设定为 "Date"。

新构造对象的 [[Extensible]] 内部属性设定为 ture。

新构造对象的 [[PrimitiveValue]] 内部属性按照以下步骤设定：

1.  令 y 为 ToNumber(year).
2.  令 m 为 ToNumber(month).
3.  如果提供了 date ，则令 dt 为 ToNumber(date); 否则令 dt 为 1.
4.  如果提供了 hours ，则令 h 为 ToNumber(hours); 否则令 h 为 0.
5.  如果提供了 minutes ，则令 min 为 ToNumber(minutes); 否则令 min 为 0.
6.  如果提供了 seconds ，则令 s 为 ToNumber(seconds); 否则令 s 为 0.
7.  如果提供了 ms ，则令 milli 为 ToNumber(ms); 否则令 milli 为 0.
8.  如果 y 不是 NaN 且 0 ≤ ToInteger(y) ≤ 99, 则令 yr 为 1900+ToInteger(y); 否则令 yr 为 y.
9.  令 finalDate 为 MakeDate(MakeDay(yr, m, dt), MakeTime(h, min, s, milli)).
10.  设定新构造对象的 [[PrimitiveValue]] 内部属性为 TimeClip(UTC(finalDate)).

#### new Date (value)

新构造对象的 [[Prototype]] 内部属性设定为原始的时间原型对象，它是 Date.prototype(15.9.4.1) 的初始值。

新构造对象的 [[Class]] 内部属性设定为 "Date"。

新构造对象的 [[Extensible]] 内部属性设定为 ture。

新构造对象的 [[PrimitiveValue]] 内部属性按照以下步骤设定：

1.  令 v 为 ToPrimitive(value).
2.  如果 Type(v) 是 String, 则
    1.  用与 parse 方法 (15.9.4.2) 完全相同的方式将 v 解析为一个日期时间；令 V 为这个日期时间的时间值。
3.  否则 , 令 V 为 ToNumber(v).
4.  设定新构造对象的 [[PrimitiveValue]] 内部属性为 TimeClip(V)，并返回这个值。

#### new Date ( )

新构造对象的 [[Prototype]] 内部属性设定为原始的时间原型对象，它是 Date.prototype(15.9.4.1) 的初始值。

新构造对象的 [[Class]] 内部属性设定为 "Date"。

新构造对象的 [[Extensible]] 内部属性设定为 ture。

新构造对象的 [[PrimitiveValue]] 内部属性设定为表示当前时间的时间值（协调世界时）。

### Date 构造器的属性

Date 构造器的 [[Prototype]] 内部属性的值是函数原型对象 (15.3.4)。

除了内部属性和 length 属性 ( 值为 7) 之外，Date 构造器还有以下属性：

#### Date.prototype

Date.prototype 的初始值是内置的 Date 原型对象 (15.9.5)。

此属性有特性 { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

#### Date.parse (string)

parse 函数对它的参数应用 ToString 操作并将结果字符串解释为一个日期和时间；返回一个数字值，是对应这个日期时间的 UTC 时间值。字符串可解释为本地时间，UTC 时间，或某个其他时区的时间，这取决于字符串里的内容。此函数首先尝试根据日期时间字符串格式（15.9.1.15）里的规则来解析字符串的格式。如果字符串不符合这个格式此函数可回退，用任意实现定义的试探方式或日期格式。无法识别的字符串或日期时间包含非法元素值，将导致 Date.parse 返回 NaN。

在所有属性都指向它们的初始值的情况下，如果 x 是一个在特定 ECMAScript 的实现里的毫秒数为零的任意 Date 对象，则在这个实现中以下所有表达式应产生相同数字值：

`x.valueOf()` `Date.parse(x.toString())` `Date.parse(x.toUTCString())` `Date.parse(x.toISOString())`

然而，表达式

#### Date.parse( x.toLocaleString())

是不需要产生与前面三个表达参数相同的数字值。通常，在给定的字符串不符合日期时间字符串格式（15.9.1.15）时，Date.parse 的产生值是依赖于实现，并且在同一实现中 toString 或 toUTCString 方法不能产生不符合日期时间字符串格式的字符串。

#### Date.UTC (year, month [, date [, hours [, minutes [, seconds [, ms ] ] ] ] ])

当用少于两个的参数调用 UTC 函数时，它的行为是依赖于实现的。当用二到七个参数调用 UTC 函数，它从 year, month 和 ( 可选的 ) date, hours, minutes, seconds, ms 计算出日期时间。采用以下步骤：

1.  令 y 为 ToNumber(year).
2.  令 m 为 ToNumber(month).
3.  如果提供了 date ，则令 dt 为 ToNumber(date); 否则令 dt 为 1.
4.  如果提供了 hours ，则令 h 为 ToNumber(hours); 否则令 h 为 0.
5.  如果提供了 minutes ，则令 min 为 ToNumber(minutes); 否则令 min 为 0.
6.  如果提供了 seconds ，则令 s 为 ToNumber(seconds); 否则令 s 为 0.
7.  如果提供了 ms ，则令 milli 为 ToNumber(ms); 否则令 milli 为 0.
8.  如果 y 不是 NaN 且 0 ≤ ToInteger(y) ≤ 99, 则令 yr 为 1900+ToInteger(y); 否则令 yr 为 y.
9.  返回 TimeClip(MakeDate(MakeDay(yr, m, dt), MakeTime(h, min, s, milli))).

UTC 函数的 length 属性是 7。

UTC 函数与 Date 构造器的不同点有：它返回一个时间值，而不是创建 Date 对象，还有它将参数解释为 UTC，而不是本地时间。

#### Date.now ( )

now 函数返回一个数字值，它表示调用 now 时的 UTC 日期时间的时间值。

### Date 原型对象的属性

Date 原型对象自身是一个 Date 对象（其 [[Class]] 是 "Date"），其 [[PrimitiveValue]] 是 NaN。

Date 原型对象的 [[Prototype]] 内部属性的值是标准内置 Object 原型对象 (15.2.4)。

在以下对 Date 原型对象的函数属性的描述中，短语“this Date 对象”指调用函数时的 this 值对象。除非另外说明，这些函数不是通用的；如果 this 值不是 [[Class]] 内部属性为 "Date" 的对象，则抛出一个 TypeError 异常。短语“this 时间值”指代表 this Date 对象的时间值的数字值，它是 this Date 对象的 [[PrimitiveValue]] 内部属性的值。

#### Date.prototype.constructor

Date.prototype.constructor 的初始值是内置 Date 构造器。

#### Date.prototype.toString ( )

此函数返回一个字符串值。字符串中内容是依赖于实现的，但目的是用一种方便，人类可读的形式表示当前时区的时间。

对毫秒数为零的任意 Date 值 d，Date.parse(d.toString()) 和 d.valueOf() 的结果相同。见 15.9.4.2

#### Date.prototype.toDateString ( )

此函数返回一个字符串值。字符串中内容是依赖于实现的，但目的是用一种方便，人类可读的形式表示当前时区时间的“日期”部分。

#### Date.prototype.toTimeString ( )

此函数返回一个字符串值。字符串中内容是依赖于实现的，但目的是用一种方便，人类可读的形式表示当前时区时间的“时间”部分。

#### Date.prototype.toLocaleString ( )

此函数返回一个字符串值。字符串中内容是依赖于实现的，但目的是用一种 -- 对应宿主环境的当前语言环境设定的 -- 方便，人类可读的形式表示当前时区的时间。

这个函数的第一个参数可能会在此标准的未来版本中使用到；因此建议实现不要以任何目的使用这个位置参数。

#### Date.prototype.toLocaleDateString ( )

此函数返回一个字符串值。字符串中内容是依赖于实现的，但目的是用一种 -- 对应宿主环境的当前语言环境设定的 -- 方便，人类可读的形式表示当前时区时间的“日期”部分。

这个函数的第一个参数可能会在此标准的未来版本中使用到；因此建议实现不要以任何目的使用这个位置参数。

#### Date.prototype.toLocaleTimeString ( )

此函数返回一个字符串值。字符串中内容是依赖于实现的，但目的是用一种 -- 对应宿主环境的当前语言环境设定的 -- 方便，人类可读的形式表示当前时区时间的“时间”部分。

这个函数的第一个参数可能会在此标准的未来版本中使用到；因此建议实现不要以任何目的使用这个位置参数。

#### Date.prototype.valueOf ( )

valueOf 函数返回一个数字值，它是 this 时间值。

#### Date.prototype.getTime ( )

1.  返回 this 时间值。

#### Date.prototype.getFullYear ( )

1.  令 t 为 this 时间值 .
2.  如果 t 是 NaN, 返回 NaN.
3.  返回 YearFromTime(LocalTime(t)).

#### Date.prototype.getUTCFullYear ( )

1.  令 t 为 this 时间值 .
2.  如果 t 是 NaN, 返回 NaN.
3.  返回 YearFromTime(t).

#### Date.prototype.getMonth ( )

1.  令 t 为 this 时间值 .
2.  如果 t 是 NaN, 返回 NaN.
3.  返回 MonthFromTime(LocalTime(t)).

#### Date.prototype.getUTCMonth ( )

1.  令 t 为 this 时间值 .
2.  如果 t 是 NaN, 返回 NaN.
3.  返回 MonthFromTime(t).

#### Date.prototype.getDate ( )

1.  令 t 为 this 时间值 .
2.  如果 t 是 NaN, 返回 NaN.
3.  返回 DateFromTime(LocalTime(t)).

#### Date.prototype.getUTCDate ( )

1.  令 t 为 this 时间值 .
2.  如果 t 是 NaN, 返回 NaN.
3.  返回 DateFromTime(t).

#### Date.prototype.getDay ( )

1.  令 t 为 this 时间值 .
2.  如果 t 是 NaN, 返回 NaN.
3.  返回 WeekDay(LocalTime(t)).

#### Date.prototype.getUTCDay ( )

1.  令 t 为 this 时间值 .
2.  如果 t 是 NaN, 返回 NaN.
3.  返回 WeekDay(t).

#### Date.prototype.getHours ( )

1.  令 t 为 this 时间值 .
2.  如果 t 是 NaN, 返回 NaN.
3.  返回 HourFromTime(LocalTime(t)).

#### Date.prototype.getUTCHours ( )

1.  令 t 为 this 时间值 .
2.  如果 t 是 NaN, 返回 NaN.
3.  返回 HourFromTime(t).

#### Date.prototype.getMinutes ( )

1.  令 t 为 this 时间值 .
2.  如果 t 是 NaN, 返回 NaN.
3.  返回 MinFromTime(LocalTime(t)).

#### Date.prototype.getUTCMinutes ( )

1.  令 t 为 this 时间值 .
2.  如果 t 是 NaN, 返回 NaN.
3.  返回 MinFromTime(t).

#### Date.prototype.getSeconds ( )

1.  令 t 为 this 时间值 .
2.  如果 t 是 NaN, 返回 NaN.
3.  返回 SecFromTime(LocalTime(t)).

#### Date.prototype.getUTCSeconds ( )

1.  令 t 为 this 时间值 .
2.  如果 t 是 NaN, 返回 NaN.
3.  返回 SecFromTime(t).

#### Date.prototype.getMilliseconds ( )

1.  令 t 为 this 时间值 .
2.  如果 t 是 NaN, 返回 NaN.
3.  返回 msFromTime(LocalTime(t)).

#### Date.prototype.getUTCMilliseconds ( )

1.  令 t 为 this 时间值 .
2.  如果 t 是 NaN, 返回 NaN.
3.  返回 msFromTime(t).

#### Date.prototype.getTimezoneOffset ( )

返回本地时间和 UTC 时间之间相差的分钟数。

1.  令 t 为 this 时间值 .
2.  如果 t 是 NaN, 返回 NaN.
3.  返回 (t ? LocalTime(t)) / msPerMinute.

#### Date.prototype.setTime (time)

1.  令 v 为 TimeClip(ToNumber(time)).
2.  设定 this Date 对象的 [[PrimitiveValue]] 内部属性为 v.
3.  返回 v.

#### Date.prototype.setMilliseconds (ms)

1.  令 t 为 LocalTime(this 时间值 ) 的结果 .
2.  令 time 为 MakeTime(HourFromTime(t), MinFromTime(t), SecFromTime(t), ToNumber(ms)).
3.  令 u 为 TimeClip(UTC(MakeDate(Day(t), time))).
4.  设定 this Date 对象的 [[PrimitiveValue]] 内部属性为 u.
5.  返回 u.

#### Date.prototype.setUTCMilliseconds (ms)

1.  令 t 为 this 时间值 .
2.  令 time 为 MakeTime(HourFromTime(t), MinFromTime(t), SecFromTime(t), ToNumber(ms)).
3.  令 v 为 TimeClip(MakeDate(Day(t), time)).
4.  设定 this Date 对象的 [[PrimitiveValue]] 内部属性为 v.
5.  返回 v.

#### Date.prototype.setSeconds (sec [, ms ] )

没指定 ms 参数时的行为是，仿佛 ms 被指定为调用 getMilliseconds() 的结果一样。

1.  令 t 为 LocalTime(this 时间值 ) 的结果 .
2.  令 s 为 ToNumber(sec).
3.  如果没指定 ms , 则令 milli 为 msFromTime(t); 否则，令 milli 为 ToNumber(ms).
4.  令 date 为 MakeDate(Day(t), MakeTime(HourFromTime(t), MinFromTime(t), s, milli)).
5.  令 u 为 TimeClip(UTC(date)).
6.  设定 this Date 对象的 [[PrimitiveValue]] 内部属性为 u.
7.  返回 u.

setSeconds 方法的 length 属性是 2。

#### Date.prototype.setUTCSeconds (sec [, ms ] )

没指定 ms 参数时的行为是，仿佛 ms 被指定为调用 getUTCMilliseconds() 的结果一样。

1.  令 t 为 this 时间值 .
2.  令 s 为 ToNumber(sec).
3.  如果没指定 ms , 则令 milli 为 msFromTime(t); 否则，令 milli 为 ToNumber(ms).
4.  令 date 为 MakeDate(Day(t), MakeTime(HourFromTime(t), MinFromTime(t), s, milli)).
5.  令 v 为 TimeClip(date).
6.  设定 this Date 对象的 [[PrimitiveValue]] 内部属性为 v.
7.  返回 v.

setUTCSeconds 方法的 length 属性是 2。

#### Date.prototype.setMinutes (min [, sec [, ms ] ] )

没指定 sec 参数时的行为是，仿佛 ms 被指定为调用 getSeconds() 的结果一样。

没指定 ms 参数时的行为是，仿佛 ms 被指定为调用 getMilliseconds() 的结果一样。

1.  令 t 为 LocalTime(this 时间值 ) 的结果 .
2.  令 m 为 ToNumber(min).
3.  如果没指定 sec , 则令 s 为 SecFromTime(t); 否则 , 令 s 为 ToNumber(sec).
4.  如果没指定 ms , 则令 milli 为 msFromTime(t); 否则 , 令 milli 为 ToNumber(ms).
5.  令 date 为 MakeDate(Day(t), MakeTime(HourFromTime(t), m, s, milli)).
6.  令 u 为 TimeClip(UTC(date)).
7.  设定 this Date 对象的 [[PrimitiveValue]] 内部属性为 u.
8.  返回 u.

setMinutes 方法的 length 属性是 3。

#### Date.prototype.setUTCMinutes (min [, sec [, ms ] ] )

没指定 sec 参数时的行为是，仿佛 ms 被指定为调用 getUTCSeconds() 的结果一样。

没指定 ms 参数时的行为是，仿佛 ms 被指定为调用 getUTCMilliseconds() 的结果一样。

1.  令 t 为 this 时间值 .
2.  令 m 为 ToNumber(min).
3.  如果没指定 sec , 则令 s 为 SecFromTime(t); 否则 , 令 s 为 ToNumber(sec).
4.  如果没指定 ms , 则令 milli 为 msFromTime(t); 否则 , 令 milli 为 ToNumber(ms).
5.  令 date 为 MakeDate(Day(t), MakeTime(HourFromTime(t), m, s, milli)).
6.  令 v 为 TimeClip(date).
7.  设定 this Date 对象的 [[PrimitiveValue]] 内部属性为 v.
8.  返回 v.

setUTCMinutes 方法的 length 属性是 3。

#### Date.prototype.setHours (hour [, min [, sec [, ms ] ] ] )

没指定 min 参数时的行为是，仿佛 min 被指定为调用 getMinutes() 的结果一样。

没指定 sec 参数时的行为是，仿佛 ms 被指定为调用 getSeconds() 的结果一样。

没指定 ms 参数时的行为是，仿佛 ms 被指定为调用 getMilliseconds() 的结果一样。

1.  令 t 为 LocalTime(this 时间值 ) 的结果 .
2.  令 h 为 ToNumber(hour).
3.  如果没指定 min , 则令 m 为 MinFromTime(t); 否则 , 令 m 为 ToNumber(min).
4.  如果没指定 sec , 则令 s 为 SecFromTime(t); 否则 , 令 s 为 ToNumber(sec).
5.  如果没指定 ms , 则令 milli 为 msFromTime(t); 否则 , 令 milli 为 ToNumber(ms).
6.  令 date 为 MakeDate(Day(t), MakeTime(h, m, s, milli)).
7.  令 u 为 TimeClip(UTC(date)).
8.  设定 this Date 对象的 [[PrimitiveValue]] 内部属性为 u.
9.  返回 u.

setHours 方法的 length 属性是 4。

#### Date.prototype.setUTCHours (hour [, min [, sec [, ms ] ] ] )

没指定 min 参数时的行为是，仿佛 min 被指定为调用 getUTCMinutes() 的结果一样。

没指定 sec 参数时的行为是，仿佛 ms 被指定为调用 getUTCSeconds() 的结果一样。

没指定 ms 参数时的行为是，仿佛 ms 被指定为调用 getUTCMilliseconds() 的结果一样。

1.  令 t 为 this 时间值 .
2.  令 h 为 ToNumber(hour).
3.  如果没指定 min , 则令 m 为 MinFromTime(t); 否则 , 令 m 为 ToNumber(min).
4.  如果没指定 sec , 则令 s 为 SecFromTime(t); 否则 , 令 s 为 ToNumber(sec).
5.  如果没指定 ms , 则令 milli 为 msFromTime(t); 否则 , 令 milli 为 ToNumber(ms).
6.  令 date 为 MakeDate(Day(t), MakeTime(h, m, s, milli)).
7.  令 v 为 TimeClip(date).
8.  设定 this Date 对象的 [[PrimitiveValue]] 内部属性为 v.
9.  返回 v.

setUTCHours 方法的 length 属性是 4。

#### Date.prototype.setDate (date)

1.  令 t 为 LocalTime(this 时间值 ) 的结果 .
2.  令 dt 为 ToNumber(date).
3.  令 newDate 为 MakeDate(MakeDay(YearFromTime(t), MonthFromTime(t), dt), TimeWithinDay(t)).
4.  令 u 为 TimeClip(UTC(newDate)).
5.  设定 this Date 对象的 [[PrimitiveValue]] 内部属性为 u.
6.  返回 u.

#### Date.prototype.setUTCDate (date)

1.  令 t 为 this 时间值 .
2.  令 dt 为 ToNumber(date).
3.  令 newDate 为 MakeDate(MakeDay(YearFromTime(t), MonthFromTime(t), dt), TimeWithinDay(t)).
4.  令 v 为 TimeClip(newDate).
5.  设定 this Date 对象的 [[PrimitiveValue]] 内部属性为 v.
6.  返回 v.

#### Date.prototype.setMonth (month [, date ] )

没指定 date 参数时的行为是，仿佛 ms 被指定为调用 getDate() 的结果一样。

1.  令 t 为 LocalTime(this 时间值 ) 的结果 .
2.  令 m 为 ToNumber(month).
3.  如果没指定 date , 则令 dt 为 DateFromTime(t); 否则 , 令 dt 为 ToNumber(date).
4.  令 newDate 为 MakeDate(MakeDay(YearFromTime(t), m, dt), TimeWithinDay(t)).
5.  令 u 为 TimeClip(UTC(newDate)).
6.  设定 this Date 对象的 [[PrimitiveValue]] 内部属性为 u.
7.  返回 u.

setMonth 方法的 length 属性是 2。

#### Date.prototype.setUTCMonth (month [, date ] )

没指定 date 参数时的行为是，仿佛 ms 被指定为调用 getUTCDate() 的结果一样。

1.  令 t 为 this 时间值 .
2.  令 m 为 ToNumber(month).
3.  如果没指定 date , 则令 dt 为 DateFromTime(t); 否则 , 令 dt 为 ToNumber(date).
4.  令 newDate 为 MakeDate(MakeDay(YearFromTime(t), m, dt), TimeWithinDay(t)).
5.  令 v 为 TimeClip(newDate).
6.  设定 this Date 对象的 [[PrimitiveValue]] 内部属性为 v.
7.  返回 v.

setUTCMonth 方法的 length 属性是 2。

#### Date.prototype.setFullYear (year [, month [, date ] ] )

没指定 month 参数时的行为是，仿佛 ms 被指定为调用 getMonth() 的结果一样。

没指定 date 参数时的行为是，仿佛 ms 被指定为调用 getDate() 的结果一样。

1.  令 t 为 LocalTime(this 时间值 ) 的结果 ; 但如果 this 时间值是 NaN, 则令 t 为 +0.
2.  令 y 为 ToNumber(year).
3.  如果没指定 month , 则令 m 为 MonthFromTime(t); 否则 , 令 m 为 ToNumber(month).
4.  如果没指定 date , 则令 dt 为 DateFromTime(t); 否则 , 令 dt 为 ToNumber(date).
5.  令 newDate 为 MakeDate(MakeDay(y, m, dt), TimeWithinDay(t)).
6.  令 u 为 TimeClip(UTC(newDate)).
7.  设定 this Date 对象的 [[PrimitiveValue]] 内部属性为 u.
8.  返回 u.

setFullYear 方法的 length 属性是 3。

#### Date.prototype.setUTCFullYear (year [, month [, date ] ] )

没指定 month 参数时的行为是，仿佛 ms 被指定为调用 getUTCMonth() 的结果一样。

没指定 date 参数时的行为是，仿佛 ms 被指定为调用 getUTCDate() 的结果一样。

1.  令 t 为 this 时间值 ; 但如果 this 时间值是 NaN, 则令 t 为 +0.
2.  令 y 为 ToNumber(year).
3.  如果没指定 month , 则令 m 为 MonthFromTime(t); 否则 , 令 m 为 ToNumber(month).
4.  如果没指定 date , 则令 dt 为 DateFromTime(t); 否则 , 令 dt 为 ToNumber(date).
5.  令 newDate 为 MakeDate(MakeDay(y, m, dt), TimeWithinDay(t)).
6.  令 v 为 TimeClip(newDate).
7.  设定 this Date 对象的 [[PrimitiveValue]] 内部属性为 v.
8.  返回 v.

setUTCFullYear 方法的 length 属性是 3。

#### Date.prototype.toUTCString ( )

此函数返回一个字符串值。字符串中内容是依赖于实现的，但目的是用一种方便，人类可读的形式表示 UTC 时间。

此函数的目的是为日期时间产生一个比 15.9.1.15 指定的格式更易读的字符串表示。没必要选择明确的或易于机器解析的格式。如果一个实现没有一个首选的人类可读格式，建议使用 15.9.1.15 定义的格式，但用空格而不是“T”分割日期和时间元素。

#### Date.prototype.toISOString ( )

此函数返回一个代表 --this Date 对象表示的时间的实例 -- 的字符串。字符串的格式是 15.9.1.15 定义的日期时间字符串格式。字符串中包含所有的字段。字符串表示的时区总是 UTC，用后缀 Z 标记。如果 this 对象的时间值不是有限的数字值，抛出一个 RangeError 异常。

#### Date.prototype.toJSON ( key )

此函数为 JSON.stringify (15.12.3) 提供 Date 对象的一个字符串表示。

当用参数 key 调用 toJSON 方法，采用以下步骤：

1.  令 O 为 以 this 值为参数调用 toObject 的结果。
2.  令 tv 为 ToPrimitive(O, hint Number).
3.  如果 tv 是一个数字值且不是有限的 , 返回 null.
4.  令 toISO 为以 "toISOString" 为参数调用 O 的 [[Get]] 内部方法的结果。
5.  如果 IsCallable(toISO) 是 false, 抛出一个 TypeError 异常 .
6.  O 作为以 this 值并用空参数列表调用 toISO 的 [[Call]] 内部方法，返回结果。

参数是被忽略的。

toJSON 函数是故意设计成通用的；它不需要其 this 值必须是一个 Date 对象。因此，它可以作为方法转移到其他类型的对象上。但转移到的对象必须有 toISOString 方法。对象可自由使用参数 key 来过滤字符串化的方式。

### Date 实例的属性

Date 实例从 Date 原型对象继承属性，Date 实例的 [[Class]] 内部属性值是 "Date"。Date 实例还有一个 [[PrimitiveValue]] 内部属性。

[[PrimitiveValue]] 内部属性是代表 this Date 对象的时间值。

## RegExp ( 正则表达式 ) 对象

一个 RegExp 对象包含一个正则表达式和关联的标志。

正则表达式的格式和功能是以 Perl 5 程序语言的正则表达式设施为蓝本的。

### 模式

RegExp 构造器对输入模式字符串应用以下文法。如果文法无法将字符串解释为 Pattern 的一个展开形式，则发生错误。

语法：

`Pattern :: Disjunction` `Disjunction :: Alternative Alternative | Disjunction``Alternative :: [empty] Alternative Term``Term :: Assertion Atom Atom Quantifier``Assertion :: ^ $ \ b \ B ( ? = Disjunction ) ( ? ! Disjunction )``Quantifier :: QuantifierPrefix QuantifierPrefix ?``QuantifierPrefix :: * +  ? { DecimalDigits } { DecimalDigits , } { DecimalDigits , DecimalDigits }``Atom :: PatternCharacter . \ AtomEscape CharacterClass ( Disjunction ) ( ? : Disjunction )` `PatternCharacter :: SourceCharacter but not any of: ^ $ \ . * + ? ( ) [ ] { } |` `AtomEscape :: DecimalEscape CharacterEscape CharacterClassEscape``CharacterEscape :: ControlEscape c ControlLetter HexEscapeSequence UnicodeEscapeSequence IdentityEscape` `ControlEscape :: one of f n r t v` `ControlLetter :: one of a b c d e f g h i j k l m n o p q r s t u v w x y z A B C D E F G H I J K L M N O P Q R S T U V W X Y Z` `IdentityEscape :: SourceCharacter but not IdentifierPart` `DecimalEscape :: DecimalIntegerLiteral [lookahead ? DecimalDigit]` `CharacterClassEscape :: one of d D s S w W` `CharacterClass :: [ [lookahead ? {^}] ClassRanges ] [ ^ ClassRanges ]``ClassRanges :: [empty] NonemptyClassRanges``NonemptyClassRanges :: ClassAtom ClassAtom NonemptyClassRangesNoDash ClassAtom - ClassAtom ClassRanges``NonemptyClassRangesNoDash :: ClassAtom ClassAtomNoDash NonemptyClassRangesNoDash ClassAtomNoDash - ClassAtom ClassRanges``ClassAtom :: - ClassAtomNoDash``ClassAtomNoDash :: SourceCharacter but not one of \ or ] or - \ ClassEscape``ClassEscape :: DecimalEscape b CharacterEscape CharacterClassEscape`

### 模式语义

使用下面描述的过程来将一个正则表达式模式转换为一个内部程序。实现使用比下面列出的算法跟高效的算法是被鼓励的，只要结果是相同的。内部程序用作 RegExp 对象的 [[Match]] 内部属性的值。

#### 表示法

后面的描述用到以下变量：

*   input，是正则表达式模式要匹配的字符串。符号 input[n] 表示 input 的第 n 个字符，这里的 n 可以是 0( 包括 ) 和 InputLength( 不包括 ) 之间的。
*   InputLength，是 input 字符串里的字符数目。
*   NcapturingParens，是在模式中左捕获括号的总数 ( 即，Atom :: ( Disjunction ) 产生式被展开的总次数 )。一个左捕获括号是匹配产生式 Atom :: ( Disjunction ) 中的 终结符 ( 的任意 ( 模式字符。
*   IgnoreCase，是 RegExp 对象的 ignoreCase 属性的设定值。
*   Multiline，是 RegExp 对象的 multiline 属性的设定值。

此外，后面的描述用到以下内部数据结构：

*   CharSet，是字符的一个数学上的集合。
*   State，是一个有序对 (endIndex, captures) ，这里 endIndex 是一个整数，captures 是有 NcapturingParens 个值的内部数组。 States 用来表示正则表达式匹配算法里的局部匹配状态。endIndex 是到目前为止模式匹配的最后一个输入字符的索引值加上一，而 captures 持有捕获括号的捕获结果。captures 的第 n 个元素是一个代表第 n 个捕获括号对捕获值的字符串，或如果第 n 个捕获括号对未能达到目的，captures 的第 n 个元素是 undefined。由于回溯，很多 States 可能在匹配过程中的任何时候被使用。
*   MatchResult，值为 State 或表示匹配失败特殊 token--failure。
*   Continuation 程序，是一个内部闭包（即，一些参数已经绑定了值的内部程序），它用一个 State 参数返回一个 MatchResult 结果。 如果一个内部闭包引用的变量是绑定在创建这个闭包的函数里 , 则闭包使用在创建闭包时的这些变量值。Continuation 尝试从其 State 参数给定的中间状态开始用模式的其余部分（由闭包的已绑定参数指定）匹配输入字符串。如果匹配成功，Continuation 返回最终的 State；如果匹配失败，Continuation 返回 failure。
*   Matcher 程序，是一个需要两个参数 -- 一个 State 和一个 Continuation -- 的内部闭包，它返回一个 MatchResult 结果。 Matcher 尝试从其 State 参数给定的中间状态开始用模式的一个中间子模式（由闭包的已绑定参数指定）匹配输入字符串。Continuation 参数是去匹配模式中剩余部分的闭包。用模式的子模式匹配之后获得一个新 State，之后 Matcher 用新 State 去调用 Continuation 来测试模式的剩余部分是否能匹配成功。如果匹配成功，matcher 返回 Continuation 返回的 State；如果匹配失败，Matcher 尝试用不同的可选位置重复调用 Continuation，直到 Continuation 匹配成功或用尽所有的可选位置。
*   AssertionTester 程序，是需要一个 State 参数并返回一个布尔结果的内部闭包。 AssertionTester 测试输入字符串的当前位置是否满足一个特定条件 ( 由闭包的已绑定参数指定 ) ，如果匹配了条件，返回 true；如果不匹配，返回 false。
*   EscapeValue，是一个字符或一个整数。EscapeValue 用来表示 DecimalEscape 转移序列的解释结果：一个字符 ch 在转义序列里时，它被解释为字符 ch；而一个整数 n 在转义序列里时，它被解释为对第 n 个捕获括号组的反响引用。

#### 模式（Pattern）

产生式 Pattern :: Disjunction 按照以下方式解释执行 :

1.  解释执行 Disjunction ，获得一个 Matcher m.
2.  返回一个需要两个参数的内部闭包，一个字符串 str 和一个整数 index, 执行方式如下 :
    1.  令 Input 为给定的字符串 str。15.10.2 中的算法都将用到此变量。
    2.  令 InputLength 为 Input 的长度。15.10.2 中的算法都将用到此变量。
    3.  令 c 为 一个 Continuation ，它始终对它的任何 State 参数都返回成功匹配的 MatchResult。
    4.  令 cap 为一个有 NcapturingParens 个 undefined 值的内部数组，索引是从 1 到 NcapturingParens。
    5.  令 x 为 State (index, cap).
    6.  调用 m(x, c)，并返回结果 .

一个模式解释执行（“编译”）为一个内部程序值。RegExp.prototype.exec 可将这个内部程序应用于一个字符串和字符串的一个偏移位，来确定从这个偏移位开始 , 模式是否能够匹配，如果能匹配，将返回捕获括号的值。15.10.2 中的算法被设计为只在编译一个模式时可抛出一个 SyntaxError 异常；反过来说，一旦模式编译成功，应用编译生成的内部程序在字符串中寻找匹配结果时不可抛出异常（除非是宿主定义的可在任何时候出现的异常，如内存不足）。

#### 析取（Disjunction）

产生式 Disjunction :: Alternative 的解释执行，是解释执行 Alternative 来获得 Matcher 并返回这个 Matcher。

产生式 Disjunction :: Alternative | Disjunction 按照以下方式解释执行：

1.  解释执行 Alternative 来获得一个 Matcher m1.
2.  解释执行 Disjunction 来获得一个 Matcher m2.
3.  返回一个需要两个参数的内部闭包 Matcher ，参数分别是一个 State x 和一个 Continuation c，此内部闭包的执行方式如下：
    1.  调用 m1(x, c) 并令 r 为其结果。
    2.  如果 r 不是 failure, 返回 r.
    3.  调用 m2(x, c) 并返回其结果。

正则表达式运算符 | 用来分隔两个选择项。模式首先尝试去匹配左侧的 Alternative( 紧跟着是正则表达式的后续匹配结果 )；如果失败，尝试匹配右侧的 Disjunction（紧跟着是正则表达式的后续匹配结果）。如果左侧的 Alternative，右侧的 Disjunction，还有后续匹配结果，全都有可选的匹配位置，则后续匹配结果的所有可选位置是在左侧的 Alternative 移动到下一个可选位置之前确定的。如果左侧 Alternative 的可选位置被用尽了，右侧 Disjunction 试图替代左侧 Alternative。一个模式中任何被 | 跳过的捕获括号参数 undefined 值还代替字符串。因此，如：

`/a|ab/.exec("abc")`

返回结果是 "a"，而不是 "ab"。此外

`/((a)|(ab))((c)|(bc))/.exec("abc")`

返回的数组是

`["abc", "a", "a", undefined, "bc", undefined, "bc"]`

而不是

`["abc", "ab", undefined, "ab", "c", "c", undefined]`

#### 选择项（Alternative）

产生式 Alternative :: [empty] 解释执行返回一个 Matcher，它需要两个参数，一个 State x 和 一个 Continuation c，并返回调用 c(x) 的结果。

产生式 Alternative :: Alternative Term 按照如下方式解释执行：

1.  解释执行 Alternative 来获得一个 Matcher m1.
2.  解释执行 Term 来获得一个 Matcher m2.
3.  返回一个内部闭包 Matcher，它需要两个参数，一个 State x 和一个 Continuation c, 执行方式如下 :
    1.  创建一个 Continuation d ，它需要一个 State 参数 y ，返回调用 m2(y, c) 的结果 .
    2.  调用 m1(x, d) 并返回结果 .

连续的 Term 试着同时去匹配连续输入字符串的连续部分。如果左侧的 Alternative，右侧的 Term，还有后续匹配结果，全都有可选的匹配位置，则后续匹配结果的所有可选位置是在右侧的 Term 移动到下一个可选位置之前确定的，并且则右侧的 Term 的所有可选位置是在左侧的 Alternative 移动到下一个可选位置之前确定的。

#### 匹配项（Term）

产生式 Term :: Assertion 解释执行，返回一个需要两个参数 State x 和 Continuation c 的内部闭包 Matcher，它的执行方式如下：

1.  解释执行 Assertion 来获得一个 AssertionTester t.
2.  调用 t(x) 并令 r 为调用结果布尔值 .
3.  如果 r 是 false, 返回 failure.
4.  调用 c(x) 并返回结果 .

产生式 Term :: Atom 的解释执行方式是，解释执行 Atom 来获得一个 Matcher 并返回这个 Matcher。

产生式 Term :: Atom Quantifier 的解释执行方式如下 :

1.  解释执行 Atom 来获得一个 Matcher m.
2.  解释执行 Quantifier 来获得三个结果值：一个整数 min, 一个整数 ( 或 ∞) max, 和一个布尔值 greedy.
3.  如果 max 是有限的 且小于 min, 则抛出一个 SyntaxError 异常 .
4.  令 parenIndex 为整个正则表达式中在此产生式 Term 展开形式左侧出现的左匹配括号的数目。这是此产生式 Term 前面展开的 Atom :: ( Disjunction ) 产生式总数与此 Term 里面的 Atom :: ( Disjunction ) 产生式总数之和。
5.  令 parenCount 为在展开的 Atom 产生式里的左捕获括号数目。这是 Atom 产生式里面 Atom :: ( Disjunction ) 产生式的总数。
6.  返回一个需要两个参数 State x 和 Continuation c 的内部闭包 Matcher，执行方式如下 :
    1.  调用 RepeatMatcher(m, min, max, greedy, x, c, parenIndex, parenCount) ，并返回结果 .

抽象操作 RepeatMatcher 需要八个参数，一个 Matcher m, 一个整数 min, 一个整数 ( 或 ∞) max, 一个布尔值 greedy, 一个 State x, 一个 Continuation c, 一个整数 parenIndex, 一个整数 parenCount, 执行方式如下 :

1.  如果 max 是零 , 则调用 c(x) ，并返回结果 .
2.  创建需要一个 State 参数 y 的内部 Continuation 闭包 d ，执行方式如下 :
    1.  如果 min 是零 且 y 的 endIndex 等于 x 的 endIndex, 则返回 failure.
    2.  如果 min 是零，则令 min2 为零 ; 否则令 min2 为 min–1.
    3.  如果 max 是 ∞, 则令 max2 为 ∞; 否则令 max2 为 max–1.
    4.  调用 RepeatMatcher(m, min2, max2, greedy, y, c, parenIndex, parenCount) ，并返回结果 .
3.  令 cap 为 x 的捕获内部数组的一个拷贝。
4.  对所有满足条件 parenIndex < k 且 k ≤ parenIndex+parenCount 的整数 k，设定 cap[k] 为 undefined。
5.  令 e 为 x 的 endIndex.
6.  令 xr 为 State 值 (e, cap).
7.  如果 min 不是零 , 则调用 m(xr, d)，并返回结果 .
8.  如果 greedy 是 false, 则
    1.  令 z 为调用 c(x) 的结果 .
    2.  如果 z 不是 failure, 返回 z.
    3.  调用 m(xr, d)，并返回结果 .
9.  令 z 为调用 m(xr, d) 的结果 .
10.  如果 z 不是 failure, 返回 z.
11.  调用 c(x) ，并返回结果 .

一个 Atom 后跟 Quantifier 是用 Quantifier 指定重复的次数。Quantifier 可以是非贪婪的，这种情况下 Atom 模式在能够匹配序列的情况下尽可能重复少的次数， 或者它可以使贪婪的，这种情况下 Atom 模式在能够匹配序列的情况下尽可能重复多的次数，Atom 模式重复的是他自己而不是它匹配的字符串，所以不同次的重复中 Atom 可以匹配不同的子串。

假如 Atom 和后续的正则表达式都有选择余地，Atom 首先尽量多匹配（或者尽量少，假如是非贪婪模式）在最后一次 Atom 的重复中移动到下一个选择前，所有的后续中的选项<都应该被尝试。 在倒数第二次（第 n–1 次）Atom 的重复中移动到下一个选择前，所有 Atom 的选项在最后一次（第 n 次）重复中应该被尝试。这样可以得出更多或者更少的重复次数可行。 这些情况在开始匹配下一个选项的第(n-1)次重复时已经被穷举，以此类推。

比较

`/a[a-z]{2,4}/.exec("abcdefghi")`

它返回"abcde"而

`/a[a-z]{2,4}?/.exec("abcdefghi")`

它返回"abc".

再考虑

`/(aa|aabaac|ba|b|c)*/.exec("aabaac")`

按照上面要求的选择数，它返回

`["aaba", "ba"]`

而非以下：

`["aabaac", "aabaac"] ["aabaac", "c"]`

上面要求的选择数可以用来编写一个计算两个数最大公约数的正则表达式(用单一字符重复数表示). 以下实例用来计算 10 和 15 的最大公约数:

`"aaaaaaaaaa,aaaaaaaaaaaaaaa".replace(/^(a+)\1*,\1+$/,"$1")`

它返回最大公约数的单一字符重复数表示"aaaaa".

RepeatMatcher 的步骤 4 每重复一次就清除 Atom 的捕获。我们可以看到它在正则表达式中的行为：

`/(z)((a+)?(b+)?(c))*/.exec("zaacbbbcac")`

它返回数组

`["zaacbbbcac", "z", "ac", "a", undefined, "c"]`

而非

`["zaacbbbcac", "z", "ac", "a", "bbb", "c"]`

因为最外面的*每次迭代都会清除所有括起来的 Atom 中所含的捕获字符串，在这个例子中就是包含编号为 2,3,4,5 的捕获字符串。

RepeatMatcher 的 d 闭包状态步骤 1，一旦重复的最小次数达到，任何 Atom 匹配空 String 的扩展不再会匹配到重复中。这可以避免正则引擎在匹配类似下面的模式时掉进无限循环：

`/(a*)*/.exec("b")`

或者稍微复杂一点：

`/(a*)b\1+/.exec("baaaac")`

它返回数组：

`["b", ""]`

#### Assertion

产生式 Assertion :: ^ 解释执行返回一个 AssertionTester ， 它需要 1 个参数 State x，并按如下算法执行：

1.  使 e 为 x 的 endIndex
2.  若 e = 0， 返回 true
3.  若 Multiline 为 false，返回 false
4.  若 Input[e - 1] 的字符为 LineTerminator，返回 true
5.  返回 false

产生式 Assertion :: $ 解释执行返回一个 AssertionTester ， 它需要 1 个参数 State x，并按如下算法执行：

1.  使 e 为 x 的 endIndex
2.  若 e = InputLength， 返回 true
3.  若 Multiline 为 false，返回 false
4.  若 Input[e - 1] 的字符为 LineTerminator，返回 true
5.  返回 false

产生式 Assertion :: \ b 解释执行返回一个 AssertionTester ， 它需要 1 个参数 State x，并按如下算法执行：

1.  使 e 为 x 的 endIndex
2.  调用 IsWordChar(e–1)，返回 Boolean 值赋给 a
3.  调用 IsWordChar(e)，返回 Boolean 值赋给 b
4.  若 a 为 true，b 为 false，返回 true
5.  若 b 为 false，b 为 true，返回 true
6.  若 Input[e - 1] 的字符为 LineTerminator，返回 true
7.  返回 false

产生式 Assertion :: \ B 解释执行返回一个 AssertionTester ， 它需要 1 个参数 State x，并按如下算法执行：

1.  使 e 为 x 的 endIndex
2.  调用 IsWordChar(e–1)，返回 Boolean 值赋给 a
3.  调用 IsWordChar(e)，返回 Boolean 值赋给 b
4.  若 a 为 true，b 为 false，返回 false
5.  若 b 为 false，b 为 true，返回 false
6.  若 Input[e - 1] 的字符为 LineTerminator，返回 true
7.  返回 true

产生式 Assertion :: (? = Disjunction) 按如下算法执行：

1.  执行 Disjunction ，得到 Matcher m
2.  返回一个需要两个参数的内部闭包 Matcher ，参数分别是一个 State x 和一个 Continuation c，此内部闭包的执行方式如下：：
    1.  使 d 为一个 Continuation，它始终对它的任何 State 参数都返回成功匹配的 MatchResult
    2.  调用 m(x, d)，令 r 为其结果
    3.  若 r 为 failure，返回 failure
    4.  使 y 为 r 的 State
    5.  使 cap 为 r 的 captures
    6.  使 xe 为 r 的 endIndex
    7.  使 z 为 State (xe, cap)
    8.  调用 c(z)，返回结果

产生式 Assertion :: (? ! Disjunction) 按如下算法执行：

1.  执行 Disjunction ，得到 Matcher m
2.  返回一个需要两个参数的内部闭包 Matcher ，参数分别是一个 State x 和一个 Continuation c，此内部闭包的执行方式如下：：
    1.  使 d 为一个 Continuation，它始终对它的任何 State 参数都返回成功匹配的 MatchResult
    2.  调用 m(x, d)，令 r 为其结果
    3.  若 r 为 failure，返回 failure
    4.  调用 c(z)，返回结果

抽象操作 IsWordChar ，拥有一个 integer 类型的参数 e，按如下方式执行：

1.  若 e == -1 或 e == InputLength，返回 false
2.  令 c 为 Input[e]
3.  若 c 为 以下 63 个字符，返回 true

a b c d e f g h i j k l m n o p q r s t u v w x y z A B C D E F G H I J K L M N O P Q R S T U V W X Y Z 0 1 2 3 4 5 6 7 8 9 _

1.  返回 false

#### Quantifier

产生式 Quantifier :: QuantifierPrefix 按如下方式执行：

1.  执行 QuantifierPrefix 得到 2 个数 min 和 max（或 ∞）
2.  返回 min，max，true

产生式 Quantifier :: QuantifierPrefix ? 按如下方式执行：

1.  执行 QuantifierPrefix 得到 2 个数 min 和 max（或 ∞）
2.  返回 min，max，true

产生式 Quantifier :: * 返回 0 和 ∞

产生式 Quantifier :: + 返回 1 和 ∞

产生式 Quantifier :: ? 返回 0 和 1

产生式 Quantifier :: { DecimalDigits } 按如下方式执行：

1.  令 i 为 DecimalDigits 的 MV
2.  返回 2 个结果 i，i

产生式 Quantifier :: { DecimalDigits, } 按如下方式执行：

1.  令 i 为 DecimalDigits 的 MV
2.  返回 2 个结果 i，∞

产生式 Quantifier :: { DecimalDigits, DecimalDigits} 按如下方式执行：

1.  令 i 为 DecimalDigits 的 MV
2.  令 j 为 DecimalDigits 的 MV
3.  返回 2 个结果 i，j

#### Atom

产生式 Atom :: PatternCharacter 执行方式如下：

1.  令 ch 为 PatternCharacter 表示的字符
2.  令 A 为单元素 CharSet，包含 ch
3.  调用 CharacterSetMatcher(A, false)，返回 Matcher

产生式 Atom :: . 执行方式如下：

1.  令 A 为 除去 LineTerminator 外的所有字符
2.  调用 CharacterSetMatcher(A, false)，返回 Matcher

产生式 Atom :: \ AtomEscape 通过执行 AtomEscape 返回 Matcher。

产生式 Atom :: CharacterClass 执行方式如下：

1.  执行 CharacterClass 得到 CharSet A 和 Boolean invert
2.  调用 CharacterSetMatcher(A, false)，返回 Matcher

产生式 Atom :: ( Disjunction ) 执行方式如下：

1.  执行 Disjunction 得到 Matcher
2.  令 parenIndex 为 在整个正则表达式中从产生式展开初始化左括号时，当前展开左捕获括号的索引。parenIndex 为在产生式的 Atom 被展开之前，Atom :: ( Disjunction )产生式被展开的次数，加上 Atom :: ( Disjunction ) 闭合 这个 Atom 的次数
3.  返回一个内部闭包 Matcher，拥有 2 个参数：一个 State x 和 Continuation c，执行方式如下：
    1.  创建内容闭包 Continuation d，参数为 State y，并按如下方式执行：
        1.  令 cap 为 y 的 capture 数组的一个拷贝
        2.  令 xe 为 x 的 endIndex
        3.  令 ye 为 y 的 endIndex
        4.  令 s 为 Input 从索引 xe（包括）至 ye（不包括）范围的新创建的字符串
        5.  令 s 为 cap[parenIndex+1]
        6.  令 z 为 State (ye, cap)
        7.  调用 c(z)，返回其结果
    2.  执行 m(x, d)，返回其结果

产生式 Atom :: ( ? : Disjunction ) 通过执行 Disjunction 得到并返回一个 Matcher。

抽象操作 CharacterSetMatcher ，拥有 2 个参数：一个 CharSet A 和 Boolean invert 标志，按如下方式执行：

1.  返回一个内部闭包 Matcher，拥有 2 个参数：一个 State x 和 Continuation c，执行方式如下：
    1.  令 e 为 x 的 endIndex
    2.  若 e == InputLength，返回 failure
    3.  令 ch 为字符 Input[e]
    4.  令 cc 为 Canonicalize(ch)的结果
    5.  若 invert 为 false，如果 A 中不存在 a 使得 Canonicalize(a) == cc，返回 failure
    6.  若 invert 为 true，如果 A 中存在 a 使得 Canonicalize(a) == cc， 返回 failure
    7.  令 cap 为 x 的内部 captures 数组
    8.  令 y 为 State (e+1, cap)
    9.  调用 c(y)，返回结果

抽象操作 Canonicalize，拥有一个字符参数 ch，按如下方式执行：

1.  若 IgnoreCase 为 false，返回 ch
2.  令 u 为 ch 转换为大写后的结果，仿佛通过调用标准内置方法 String.prototype.toUpperCase
3.  若 u 不含单个字符，返回 ch
4.  令 cu 为 u 的字符
5.  若 ch 的 code unit value>= 128 且 cu 的 code unit value<= 128，返回 ch
6.  返回 cu

( Disjunction ) 的括号 用来组合 Disjunction 模式，并保存匹配结果。该结果可以通过后向引用（一个非零数，前置\），在一个替换字符串中的引用，或者作为正则表达式内部匹配过程的部分结果。使用(?: Disjunction )来避免括号的捕获行为。

(? = Disjunction )指定一个零宽正向预查。为了保证匹配成功，其 Disjunction 必须首先能够匹配成功，但在匹配后续字符前，其当前位置会不变。如果 Disjunction 能在当前位置以多种方式匹配，那么只会取第一次匹配的结果。不像其他正则表达式运算符，(?= 内部不会回溯（这个特殊的行为是从 Perl 继承过来的）。在 Disjunction 含有捕获括号，模式的后续字符包括后向引用时匹配结果会有影响。

例如，

`/(?=(a+))/.exec("baaabac")`

会匹配第一个 b 后的空白字符串，得到：

`["", "aaa"]`

为了说明预查不会回溯，

`/(?=(a+))a*b\1/.exec("baaabac")`

得到：

`["aba", "a"]`

而不是：

`["aaaba", "a"]`

(?! Disjunction ) 指定一个零宽正向否定预查。为了保证匹配成功，其 Disjunction 必须首先能够匹配失败，但在匹配后续字符前，其当前位置会不变。Disjunction 能含有捕获括号，但是对这些捕获分组的后向引用只在 Disjunction 中有效。在当前模式的其他位置后向引用捕获分组都会返回 undefined。因为否定预查必须满足预查失败来保证模式成功匹配。例如，

`/(.*?)a(?!(a+)b\2c)\2(.*)/.exec("baaabaac")`

搜索 a，其后有 n 个 a，一个 b，n 个 a（\2 指定）和一个 c。第二个\2 位于负向预查模式的外部，因此它匹配 undefined，且总是成功的。整个表达式返回一个数组：

`["baaabaac", "ba", undefined, "abaac"]`

在发生比较前，一次不区分大小写的匹配中所有的字符都会隐式转换为大写。然而，如果某些单个字符在转换为大写时扩展为多个字符，那么该字符会保持原样。当某些非 ASCII 字符在转换为大写时变成 ASCII 字符，该字符也会保持原样。这样会阻止 Unicode 字符（例如\u0131 和\u017F）匹配正则表达式 （例如仅匹配 ASCII 字符的正则表达式/[a z]/i）。而且，如果转换允许，/[^\W]/i 会匹配除去 i 或 s 外的每一个 a，b，......，h。

#### AtomEscape

产生式 AtomEscape :: DecimalEscape 执行方式如下：

1.  执行 DecimalEscape 得到 EscapeValue E
2.  如果 E 为一个字符，
    1.  令 ch 为 E 的字符
    2.  令 A 为包含 ch 字符的单元素字符集 CharSet
    3.  调用 CharacterSetMatcher(A, false) 返回 Matcher 结果
3.  E 必须是一个数。令 n 为该数。
4.  如果 n=0 或 n>NCapturingParens，抛出 SyntaxError 异常
5.  返回一个内部闭包 Matcher，拥有 2 个参数：一个 State x 和 Continuation c，执行方式如下：
    1.  令 cap 为 x 的 captures 内部数组
    2.  令 s 为 cap[n]
    3.  如果 s 为 undefined，调用 c（x），返回结果
    4.  令 e 为 x 的 endIndex
    5.  令 len 为 s 的 length
    6.  令 f 为 e+len
    7.  如果 f>InputLength，返回 failure
    8.  如果存在位于 0（包括）到 len（不包括）的整数 i 使得 Canonicalize(s[i])等于 Canonicalize(Input [e+i])，那么返回 failure
    9.  令 y 为 State(f, cap)
    10.  调用 c(y)，返回结果

产生式 AtomEscape :: CharacterEscape 执行方式如下：

1.  执行 CharacterEscape 得到一个 ch 字符
2.  令 A 为包含 ch 字符的单元素字符集 CharSet
3.  调用 CharacterSetMatcher(A, false) 返回 Matcher 结果

产生式 AtomEscape :: CharacterClassEscape 执行方式如下：

1.  执行 CharacterClassEscape 得到一个 CharSet A
2.  调用 CharacterSetMatcher(A, false) 返回 Matcher 结果

格式\后为非零数 n 的转义序列匹配捕获分组的第 n 次匹配结果。如果正则表达式少于 n 个捕获括号，会报错。如果正则表达式大于等于 n 个捕获括号，由于没有捕获到任何东西，导致第 n 个捕获分组结果为 undefined，那么后向引用总是成功的。

#### CharacterEscape

产生式 CharacterEscape :: ControlEscape 执行返回一个根据表 23 定义的字符：

ControlEscape 字符转义值

| ControlEscape | 字符编码值 | 名称 | 符号 |
| t | \u0009 | 水平制表符 | <HT> |
| n | \u000A | 进行（新行） | <LF> |
| v | \u000B | 竖直制表符 | <VT> |
| f | \u000C | 进纸 | <FF> |
| r | \u000D | 回车 | <CR> |

产生式 CharacterEscape :: ch ControlLetter 执行过程如下：

1.  令 ch 为通过 ControlLetter 表示的字符
2.  令 i 为 ch 的 code unit value
3.  令 j 为 i/32 的余数
4.  返回 j

产生式 CharacterEscape :: HexEscapeSequence 执行 HexEscapeSequence 的 CV，返回其字符结果。

产生式 CharacterEscape :: UnicodeEscapeSequence 执行 UnicodeEscapeSequence 的 CV，返回其字符结果。

产生式 CharacterEscape :: IdentityEscape 执行返回由 IdentityEscape 表示的字符。

#### DecimalEscape

产生式 DecimalEscape :: DecimalIntegerLiteral [lookahead ? DecimalDigit] 按如下方式执行：

1.  令 i 为 DecimalIntegerLiteral 的 CV 值
2.  如果 i 为 0，返回包含一个<NUL>字符（Unicode 值为 0000）的 EscapeValue
3.  返回包含整数 i 的 EscapeValue

“the MV of DecimalIntegerLiteral”在 7.8.3 节定义。

如果\后面是一个数字，且首位为 0，那么，该转义序列被认为是一个后向引用。如果 n 比在整个正则表达式左捕获括号个数大，那么会出错。\0 表示 <NUL>字符，其后不能再有数字。

#### CharacterClassEscape

产生式 CharacterClassEscape :: d 执行返回包含 0 到 9 之间的十元素字符集。

产生式 CharacterClassEscape :: D 执行返回不包括 CharacterClassEscape :: d 的字符集。

产生式 CharacterClassEscape :: s 执行返回包含 WhiteSpace 或 LineTerminator 产生式右部分字符的字符集。

产生式 CharacterClassEscape :: S 执行返回不包括 CharacterClassEscape :: s 的字符集。

产生式 CharacterClassEscape :: w 执行返回包含如下 63 个字符的字符集：

a b c d e f g h i j k l m n o p q r s t u v w x y z A B C D E F G H I J K L M N O P Q R S T U V W X Y Z 0 1 2 3 4 5 6 7 8 9 _

产生式 CharacterClassEscape :: W 执行返回不包括 CharacterClassEscape :: w 的字符集。

#### CharacterClass

产生式 CharacterClass :: [ [lookahead ? {^}] ClassRanges ] 通过执行 ClassRanges 获得并返回这个 CharSet 和 Boolean false。

产生式 CharacterClass :: [ ^ ClassRanges ] 通过执行 ClassRanges 获得并返回这个 CharSet 和 Boolean true。

#### ClassRanges

产生式 ClassRanges :: [empty]执行返回一个空的 CharSet。

产生式 ClassRanges :: NonemptyClassRanges 通过执行 NonemptyClassRanges 获得并返回这个 CharSet。

#### NonemptyClassRanges

产生式 NonemptyClassRanges :: ClassAtom 通过执行 ClassAtom 获得一个 CharSet 并返回这个 CharSet。

产生式 NonemptyClassRanges :: ClassAtom NonemptyClassRangesNoDash 按如下方式执行：

1.  执行 ClassAtom 得到一个 CharSet A
2.  执行 NonemptyClassRangesNoDash 得到一个 CharSet B
3.  返回 A 与 B 的并集

产生式 NonemptyClassRanges :: ClassAtom - ClassAtom ClassRanges 按如下方式执行：

1.  执行第一个 ClassAtom 得到一个 CharSet A
2.  执行第二个 ClassAtom 得到一个 CharSet B
3.  执行 ClassRanges 得到一个 CharSet C
4.  调用 CharacterRange(A, B)，令 D 为其结果 CharSet
5.  返回 D 与 C 的并集

抽象操作 CharacterRange，拥有 2 个 CharSet 参数 A 和 B，执行方式如下：

1.  如果 A 或 B 为空，抛出 SyntaxError 异常
2.  令 a 为 CharSet A 的一个字符
3.  令 b 为 CharSet B 的一个字符
4.  令 i 为 a 的 code unit value
5.  令 j 为 b 的 code unit value
6.  如果 i>j，抛出 SyntaxError 异常
7.  返回位于在 i 到 j（包括边界）之间的所有字符的字符集

#### NonemptyClassRangesNoDash

产生式 NonemptyClassRangesNoDash :: ClassAtom 执行过程是执行 ClassAtom 产生一个 CharSet 并且返回这个 CharSet。

产生式 NonemptyClassRangesNoDash :: ClassAtomNoDash NonemptyClassRangesNoDash 按以下方式执行：

1.  执行 ClassAtomNoDash 产生 CharSet A
2.  执行 NonemptyClassRangesNoDash 产生 CharSet B
3.  返回 CharSet A 和 B 的并集

产生式 NonemptyClassRangesNoDash :: ClassAtomNoDash - ClassAtom ClassRanges 按以下方式执行：

1.  执行 ClassAtomNoDash 产生 CharSet A
2.  执行 ClassAtom 产生 CharSet B
3.  执行 ClassRanges 产生 CharSet C
4.  调用 CharacterRange(A, B)并设 CharSet D 为结果。
5.  返回 CharSet D 和 C 的并集

ClassRanges 可以拆分成单独的 ClassAtom 且/或两个用减号分隔的 ClassAtom。在后面的情况下 ClassAtom 包含第一个到第二个 ClassAtom 间的所有字符。 如果两个 ClassAtom 之一不是表示一个单独字符（例如其中一个是\w）或者第一个 ClassAtom 的字符编码值比第二个 ClassAtom 的字符编码值大则发生错误。

即使匹配忽略大小写，区间两端的大小写在区分哪些字符属于区间时仍然有效。这意味着，例如模式/[E-F]/仅仅匹配 E, F, e, 和 f。/[E-f]/i 则匹配所有大写和小写的 ASCII 字母以及[， \， ]， ^， _， 和 `

-字符可能被当做字面意思或者表示一个区间，它作为 ClassRange 的开头或者结尾、在区间指定开头或者结尾，或者紧跟一个区间指定的时候被当做字面意思。

#### ClassAtom

产生式 ClassAtom :: - 执行返回包含单个字符 - 的字符集。

产生式 ClassAtom :: ClassAtomNoDash 通过执行 ClassAtomNoDash 获得并返回这个 CharSet。

#### ClassAtomNoDash

产生式 ClassAtomNoDash :: SourceCharacter 不包括\，]，- 执行返回包含由 SourceCharacter 表示的字符的单元素字符集。

产生式 ClassAtomNoDash :: \ ClassEscape 通过执行 ClassEscape 得到并返回这个 CharSet。

#### ClassEscape

产生式 ClassEscape :: DecimalEscape 按如下方式执行：

1.  执行 DecimalEscape 得到 EscapeValue E
2.  如果 E 不是一个字符，抛出 SyntaxError 异常
3.  令 ch 为 E 的字符
4.  返回包含字符 ch 的单元素 CharSet

产生式 ClassEscape :: b 执行返回包含一个<BS>字符（Unicode 值 0008）的字符集。

产生式 ClassEscape :: CharacterEscape 通过执行 CharacterEscape 获得一个字符 Charset 并返回包含该字符的单元素字符集 CharSet。

产生式 ClassEscape :: CharacterClassEscape 通过执行 CharacterClassEscape 获得并返回这个 CharSet。

ClassAtom 可以使用除\b，\B，后向引用外的转义序列。在 CharacterClass 中，\b 表示退格符。然而，\B 和后向引用会报错。同样，在一个 ClassAtom 中使用后向引用会报错。

### The RegExp Constructor Called as a Function

#### RegExp(pattern, flags)

如果 pattern 是一个对象 R，其内部属性[[Class]]为 RegExp 且 flags 为 undefined，返回 R。否则，调用内置 RegExp 构造器，通过表达式 new RegExp（pattern，flags）返回由该构造器构造的对象。

### The RegExp Constructor

当 RegExp 作为 new 表达式一部分调用时，它是一个构造器，用来初始化一个新创建的对象。

#### new RegExp(pattern, flags)

如果 pattern 是一个对象 R，其内部 [[CLASS]] 属性为 RegExp，且 flags 为 undefined，那么，令 P 为 pattern 和令 F 为 flags 用来构造 R。如果 pattern 是一个对象 R，其内部[[CLASS]]属性为 RegExp，且 flags 为 undefined，那么，抛出 TypeError 异常。否则，如果 pattern 为 undefined 且 ToString（pattern），令 P 为空的字符串；如果 flags 为 undefined 且 ToString（flags）令 F 为空字符串。

如果字符 P 不满足 Pattern 语义，那么抛出 SyntaxError 异常。否则，令新构造的对象拥有内部[[Match]]属性，该属性通过执行（编译）字符 P 作为在 15.10.2 节描述的 Pattern。

如果 F 含有除“g”,“i”，“m”外的任意字符，或者 F 中包括出现多次的字符，那么，抛出 SyntaxError 异常。

如果 SyntaxError 异常未抛出，那么：

令 S 为一个字符串，其等价于 P 表示的 Pattern，S 中的字符按如下描述进行转义。这样，S 可能或者不会与 P 或者 pattern 相同；然而，由执行 S 作为一个 Pattern 的内部处理程序必须和通过构造对象的内部[[Match]]属性的内部处理程序完全相同。

如果 pattern 里存在字符/或者\，那么这些字符应该被转义，以确保由“/”，S，“/”构成的的字符串的 S 值有效，而且 F 能被解析（在适当的词法上下文中）为一个与构造的正则表达式行为完全相同的 RegularExpressionLiteral 。例如，如果 P 是“/”，那么 S 应该为“\/”或“\u002F”，而不是“/”，因为 F 后的 /// 会被解析为一个 SingleLineComment，而不是一个 RegularExpressionLiteral。 如果 P 为空字符串，那么该规范定义为令 S 为“(?:)”。

这个新构造对象的如下属性为数据属性，其特性在 15.10.7 中定义。各属性的[[Value]]值按如下方式设置：

其 source 属性置为 S。

其 global 属性置为一个 Boolean 值。当 F 含有字符 g 时，为 true，否则，为 false。

其 ignoreCase 属性置为一个 Boolean 值。当 F 含有字符 i 时，为 true，否则，为 false。

其 multiline 属性置为一个 Boolean 值。当 F 含有字符 m 时，为 true，否则，为 false。

其 lastIndex 属性置为 0。

其内部[[Prototype]]属性置为 15.10.6 中定义的内置 RegExp 原型对象。

其内部[[Class]]属性置为“RegExp”。

如果 pattern 为 StringLiteral，一般的转义字符替换发生在被 RegExp 处理前。如果 pattern 必须含有 RegExp 识别的转义字符，那么当构成 StringLiteral 的内容时，为了防止被移除\被移除，在 StringLiteral 中的任何\必须被转义

### Properties of the RegExp Constructor

RegExp 构造器的[[Prototype]]值为内置 Function 的原型（15.3.4）。

除了内部的一些属性和 length 属性（其值为 2），RegExp 构造器还有如下属性：

#### RegExp.prototype

RegExp.prototype 的初始值为 RegExp 的原型（15.10.6）。

该属性有这些特性： { [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

### Properties of the RegExp Prototype Object

RegExp 的原型的内部[[Prototype]]属性为 Object 的原型（15.2.4）。RegExp 的原型为其本身的一个普通的正则表达式对象；它的[[Class]]为“RegExp”。RegExp 的原型对象的数据式属性的初始值被设置为仿佛由内置 RegExp 构造器深生成的表达式 new RegExp()创建的对象。

RegExp 的原型本身没有 valueOf 属性；然而，该 valueOf 属性是继承至 Object 的原型。

在作为 RegExp 原型对象的属性的如下函数描述中，“this RegExp object”是指函数激活时 this 对象；如果 this 值不是一个对象，或者一个其内部[[Class]]属性值不是“RegExp”的对象，那么一个 TypeError 会抛出。

#### RegExp.prototype.constructor

RegExp.prototype.constructor 的初始值为内置 RegExp 构造器。

#### RegExp.prototype.exec(string)

Performs a regular expression match of string against the regular expression and returns an Array object containing the results of the match, or null if string did not match.

The String ToString(string) is searched for an occurrence of the regular expression pattern as follows:

1.  令 R 为这一 RegExp 对象.
2.  令 S 为 ToString(string)的值.
3.  令 length 为 S 的长度.
4.  令 lastIndex 为以参数"lastIndex"调用 R 的内部方法[[Get]]的结果
5.  令 i 为 ToInteger(lastIndex)的值.
6.  令 global 为以参数"global"调用 R 的内部方法[[Get]]的结果
7.  若 global 为 false, 则令 i = 0.
8.  令 matchSucceeded 为 false.
9.  到 matchSucceeded 为 false 前重复以下
    1.  若 i < 0 或者 i > length, 则
        1.  以参数"lastIndex", 0, and true 调用 R 的内部方法[[Put]]
        2.  Return null.
    2.  以参数 S 和 i 调用 R 的内部方法[[Match]]
    3.  若 [[Match]] 返回失败, 则
        1.  令 i = i+1.
    4.  否则
        1.  令 r 为调用[[Match]]的结果 State.
        2.  设 matchSucceeded 为 true.
10.  令 e 为 r 的 endIndex 值.
11.  若 global 为 true,
12.  以参数"lastIndex", e, 和 true 调用 R 的内部方法[[Put]]
13.  令 n 为 r 的捕获数组的长度. (这跟 15.10.2.1's NCapturingParens 是同一个值)
14.  令 A 为如同以表达式 new Array 创建的新数组，其中 Array 是这个名字的内置构造器.
15.  令 matchIndex 为匹配到的子串在整个字符串 S 中的位置。
16.  以参数"index", 属性描述{[[Value]]: matchIndex, [[Writable]: true, [[Enumerable]]: true, [[Configurable]]: true}, 和 true 调用 A 的内部方法[[DefineOwnProperty]]
17.  以参数"input", 属性描述{[[Value]]: S, [[Writable]: true, #[[Enumerable]]: true, [[Configurable]]: true}, 和 true 调用 A 的内部方法[[DefineOwnProperty]]
18.  以参数"length", 属性描述{[[Value]]: n + 1}, 和 true 调用 A 的内部方法[[DefineOwnProperty]]
19.  令 matchedSubstr 为匹配到的子串(i.e. the portion of S between offset i inclusive and offset e exclusive).
20.  以参数"0", 属性描述{[[Value]]: matchedSubstr, [[Writable]: true, [[Enumerable]]: true, [[Configurable]]: true}, 和 true 调用 A 的内部方法[[DefineOwnProperty]]
21.  对每一满足 I > 0 且 I ≤ n 的整数 i
    1.  令 captureI 为第 i 个捕获数组中的元素.
    2.  以参数 ToString(i), 属性描述{[[Value]]: captureI, [[Writable]: true, [[Enumerable]]: true, [[Configurable]]: true}, 和 true 调用 A 的内部方法[[DefineOwnProperty]]
22.  返回 A.

#### RegExp.prototype.test(string)

采用如下步骤：

1.  令 match 为在这个 RegExp 对象上使用 string 作为参数执行 RegExp.prototype.exec (15.10.6.2) 的结果。
2.  如果 match 不为 null，返回 true；否则返回 false。

#### RegExp.prototype.toString()

返回一个 String，由“/”，RegExp 对象的 source 属性值，“/”与“g”（如果 global 属性为 true），“i”（如果 ignoreCase 为 true），“m”（如果 multiline 为 true）通过连接组成。

如果返回的字符串包含一个 RegularExpressionLiteral，那么该 RegularExpressionLiteral 用同样的方式解释执行。

### Properties of RegExp Instances

RegExp 实例继承至 RegExp 原型对象，其[[CLASS]]内部属性值为“RegExp”。RegExp 实例也拥有一个[[Match]]内部属性和一个 length 属性。

内部属性[[Match]]的值是正则表达式对象的 Pattern 的依赖实现的表示形式。

RegExp 实例还有如下属性。

#### source

source 属性为构成正则表达式 Pattern 的字符串。该属性拥有这些特性{ [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

#### global

global 属性是一 Boolean 值，表示正则表达式 flags 是否有“g”。该属性拥有这些特性{ [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

#### ignoreCase

ignoreCase 属性是一 Boolean 值，表示正则表达式 flags 是否有“i”。该属性拥有这些特性{ [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

#### multiline

multiline 属性是一 Boolean 值，表示正则表达式 flags 是否有“m”。该属性拥有这些特性{ [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false }。

#### lastIndex

lastIndex 属性指定从何处开始下次匹配的一个字符串类型的位置索引。当需要时该值会转换为一个整型数。该属性拥有这些特性{ [[Writable]]: true, [[Enumerable]]: false, [[Configurable]]: false }。

不同于其他 RegExp 实例内置属性，lastIndex 是可写的。

## Error Objects

Error 对象的实例在运行时遇到错误的情况下会被当做异常抛出。Error 对象也可以作为用户自定义异常类的基对象。

### The Error Constructor Called as a Function

当 Error 被作为函数而不是构造器调用时，它创建并初始化一个新的 Error 对象。这样函数调用 Error(…)与同样参数的对象创建表达式 new Error(…)是等效的。

#### Error (message)

新构造的对象内部属性 Prototype 会被设为原本的 Error 原型对象，也就是 Error.prototype 的初始值。(15.11.3.1)

新构造的对象内部属性 Class 会被设为"Error"。

新构造的对象内部属性 Extensible 会被设为 true。

如果形参 message 不是 undefined，新构造的对象本身属性 message 则被设为 ToString(message)。

### The Error Constructor

当 Error 作为 new 表达式的一部分被调用时，它是一个构造器：它初始化新创建的对象。

#### new Error (message)

新构造的对象内部属性 Prototype 会被设为原本的 Error 原型对象，也就是 Error.prototype 的初始值。(15.11.3.1)

新构造的对象内部属性 Class 会被设为"Error"。

新构造的对象内部属性 Extensible 会被设为 true。

如果形参 message 不是 undefined，新构造的对象本身属性 message 则被设为 ToString(message)。

### Properties of the Error Constructor

Error 构造器的内部属性 Prototype 值为 Function 原型对象(15.3.4)。

除内部属性和 length 属性（其值为 1）以外，Error 构造器还有以下属性：

#### Error.prototype

Error.prototype 的初始值为 Error 原型对象(15.11.4)。

此属性有以下特性： { Writable: false, Enumerable: false, Configurable: false }。

### Properties of the Error Prototype Object

Error 原型对象本身是一个 Error 对象（其 Class 为"Error"）。

Error 原型对象的内部属性 Prototype 为标准内置的 Object 原型对象(15.2.4)。

#### Error.prototype.constructor

Error.prototype.constructor 初始值为内置的 Error 构造器。

#### Error.prototype.name

rror.prototype.name 初始值为"Error"。

#### Error.prototype.message

Error.prototype.message 初始值为空字符串。

#### Error.prototype.toString ( )

执行以下步骤

1.  令 o 为 this 值
2.  如果 Type(O)不是对象，抛出一个 TypeError 异常。
3.  令 name 为以"name"为参数调用 O 的 Get 内置方法的结果。
4.  如果 name 为 undefined, 令 name 为"Error";否则令 name 为 ToString(name)。
5.  令 msg 为以"message"为参数调用 O 的 Get 内置方法的结果。
6.  如果 msg 为 undefined,令 msg 为空字符串;否则令 msg 为 ToString(msg)。
7.  如果 name 与 msg 都是空字符串，返回"Error"。
8.  如果 name 为空字符串，返回 msg。
9.  如果 msg 为空字符串，返回 name。
10.  返回拼接 name,":",一个空格字符，以及 msg 的结果。

### Error 实例的属性

Error 实例从 Error 原型对象继承属性，且它们的内部属性 class 值为"Error"。Error 实例没有特殊属性。

### Native Error Types Used in This Standard

以下原生 Error 对象之一会在运行时错误发生时被抛出。所有这些对象共享同样的结构，如 15.11.7 所述。

#### EvalError

本规范现在已经不再使用这个异常，这个对象保留用于跟规范之前版本的兼容性。

#### RangeError

表示一个数值超出了允许的范围，见 15.4.2.2, 15.4.5.1, 15.7.4.2, 15.7.4.5, 15.7.4.6, 以及 15.7.4.7, 15.9.5.43.

#### ReferenceError

表示一个不正确的引用值被检测到。见 8.7.1, 8.7.2, 10.2.1, 10.2.1.1.4, 10.2.1.2.4, 以及 11.13.1

#### SyntaxError

表示一个解析错误发生。见 11.1.5, 11.3.1, 11.3.2, 11.4.1, 11.4.4, 11.4.5, 11.13.1, 11.13.2, 12.2.1, 12.10.1, 12.14.1, 13.1, 15.1.2.1, 15.3.2.1, 15.10.2.2, 15.10.2.5, 15.10.2.9, 15.10.2.15, 15.10.2.19, 15.10.4.1, 以及 15.12.2

#### TypeError

表示一个操作数的真实类型与期望类型不符。见 8.6.2, 8.7.2, 8.10.5, 8.12.5, 8.12.7, 8.12.8, 8.12.9, 9.9, 9.10, 10.2.1, 10.2.1.1.3, 10.6, 11.2.2, 11.2.3, 11.4.1, 11.8.6, 11.8.7, 11.3.1, 13.2, 13.2.3, 15, 15.2.3.2, 15.2.3.3, 15.2.3.4, 15.2.3.5, 15.2.3.6, 15.2.3.7, 15.2.3.8, 15.2.3.9, 15.2.3.10, 15.2.3.11, 15.2.3.12, 15.2.3.13, 15.2.3.14, 15.2.4.3, 15.3.4.2, 15.3.4.3, 15.3.4.4, 15.3.4.5, 15.3.4.5.2, 15.3.4.5.3, 15.3.5, 15.3.5.3, 15.3.5.4, 15.4.4.3, 15.4.4.11, 15.4.4.16, 15.4.4.17, 15.4.4.18, 15.4.4.19, 15.4.4.20, 15.4.4.21, 15.4.4.22, 15.4.5.1, 15.5.4.2, 15.5.4.3, 15.6.4.2, 15.6.4.3, 15.7.4, 15.7.4.2, 15.7.4.4, 15.7.4.8 [?], 15.9.5, 15.9.5.44, 15.10.4.1, 15.10.6, 15.11.4.4 以及 15.12.3

#### URIError

表示全局 URI 处理函数被以不符合其定义的方式使用。见 15.1.3。

### NativeError 对象结构

当 ECMAScript 实现探测到一个运行时错误时，它抛出一个 15.11.6 所定义的 NativeError 对象的实例。每个这些对象都有如下所述结构，不同仅仅是在 name 属性中以构造器名称替换掉 NativeError，以及原型对象由实现自定义的 message 属性。

对于每个错误对象，定义中到 NativeError 的引用应当用 15.11.6 中具体的对象名替换。

#### NativeError Constructors Called as Functions

当 NativeError 被作为函数而不是构造器调用时，它创建并初始化一个新的 NativeError 对象。这样函数调用 NativeError(…)与同样参数的对象创建表达式 new NativeError(…)是等效的。

#### NativeError (message)

新构造的对象内部属性 Prototype 会被设为这一错误构造器附带的原型对象。新构造的对象内部属性 Class 会被设为"Error"。新构造的对象内部属性 Extensible 会被设为 true。

如果形参 message 不是 undefined，新构造的对象本身属性 message 则被设为 ToString(message)。

#### The NativeError Constructors

当 NativeError 作为 new 表达式的一部分被调用时，它是一个构造器：它初始化新创建的对象。

#### New NativeError (message)

新构造的对象内部属性 Prototype 会被设为这一错误构造器附带的原型对象。新构造的对象内部属性 Class 会被设为"Error"。新构造的对象内部属性 Extensible 会被设为 true。

如果形参 message 不是 undefined，新构造的对象本身属性 message 则被设为 ToString(message)。

#### Properties of the NativeError Constructors

NativeError 构造器的内部属性 Prototype 值为 Function 原型对象(15.3.4)。

除内部属性和 length 属性（其值为 1）以外，Error 构造器还有以下属性：

#### NativeError.prototype

NativeError.prototype 的初始值为一个 Error(15.11.4)。

此属性有以下特性： { Writable: false, Enumerable: false, Configurable: false }。

#### Properties of the NativeError Prototype Objects

每个 NativeError 的 prototype 的初始值为一个 Error（其 Class 为"Error"）。

NativeError 原型对象的内部属性 Prototype 为标准内置的 Error 对象(15.2.4)。

#### NativeError.prototype.constructor

对于特定的 NativeError，Error.prototype.constructor 初始值为 NativeError 构造器本身。

#### NativeError.prototype.name

对于特定的 NativeError，Error.prototype.name 初始值为构造器的名字。

#### NativeError.prototype.message

对于特定的 NativeError，NativeError.prototype.message 初始值为空字符串。

NativeError 构造器的原型他们自身并不提供 toString 函数，但是错误的实例可以从 Error 原型对象继承到它。

#### NativeError 实例的属性

NativeError 实例从 NativeError 原型对象继承属性，且它们的内部属性 class 值为"Error"。Error 实例没有特殊属性。

## JSON 对象

JSON 对象是一个单一的对象，它包含两个函数，parse 和 stringify，是用于解析和构造 JSON 文本的。JSON 数据的交换格式在 RFC4627 里进行了描述。 <http://www.ietf.org/rfc/rfc4627.txt>。本规范里面的 JSON 交换格式会使用 RFC4627 里所描述的，以下两点除外：

*   ECMAScript JSON 文法中的顶级 JSONText 产生式是由 JSONValue 构成，而不是 RFC4627 中限制成的 JSONObject 或者 JSONArray。
*   确认 JSON.parse 和 JSON.stringify 的实现，它们必须准确的支持本规范描述的交换格式，而不允许对格式进行删除或扩展。这一点要区别于 RFC4627，它允许 JSON 解析器接受 non-JSON 的格式和扩展。

JSON 对象内部属性 [[Prototype]] 的值是标准内建的 Object 原型对象（15.2.4）。内部属性 [[Class]] 的值是“JSON”。内部属性 [[Extensible]] 的值设置为 true。

JSON 对象没有内部属性 [[Construct]]；不能把 JSON 对象当作构造器来使用 new 操作符。

JSON 对象没有内部属性 [[Call]]; 不能把 JSON 对象当作函数来调用。

### JSON 语法

JSON.stringify 会产生一个符合 JSON 语法的字符串。JSON.parse 接受的是一个符合 JSON 语法的字符串。

#### JSON 词法

类似于 ECMAScript 源文本，JSON 是由一系列符合 SourceCharacter 规则的字符构成的。JSON 词法定义的 tokens 使得 JSON 文本类似于 ECMAScript 词法定义的 tokens 得到的 ECMAScript 源文本。JSON 词法仅能识别由 JSONWhiteSpace 产生式得到的空白字符。在语法上，所有非终止符均不是由“JSON”字符开始，而是由 ECMAScript 词法产生式定义的。

语法

`JSONWhiteSpace :: <tab> <cr> <lf> <sp>` `JSONString :: " JSONStringCharactersopt "` `JSONStringCharacters :: JSONStringCharacter JSONStringCharactersopt` `JSONStringCharacter :: SourceCharacter 但非 双引号 " 或反斜杠 \ 或 U+0000 抑或是 U+001F \ JSONEscapeSequence``JSONEscapeSequence :: JSONEscapeCharacter UnicodeEscapeSequence` `JSONEscapeCharacter :: 以下之一 " / \ b f n r t` `JSONNumber :: -optDecimalIntegerLiteral JSONFractionopt ExponentPartopt` `JSONFraction :: . DecimalDigits` `JSONNullLiteral :: NullLiteral` `JSONBooleanLiteral :: BooleanLiteral`

#### JSON 语法

根据 JSON 词法定义的 tokens，JSON 语法定义了一个合法的 JSON 文本。语法的目标符号是 JSONText。

语法

`JSONText : JSONValue` `JSONValue : JSONNullLiteral JSONBooleanLiteral JSONObject JSONArray JSONString JSONNumber``JSONObject : { } { JSONMemberList }` `JSONMember : JSONString : JSONValue` `JSONMemberList : JSONMember JSONMemberList , JSONMember``JSONArray : [ ] [ JSONElementList ]``JSONElementList : JSONValue JSONElementList , JSONValue`

### parse ( text [ , reviver ] )

parse 函数解析一段 JSON 文本（JSON 格式字符串），生成一个 ECMAScript 值。JSON 格式是 ECMAScript 直接量 的受限模式。JSON 对象可以被理解为 ECMAScript 对象。JSON 数组可以被理解为 ECMAScript 数组。JSON 字符串、数字、布尔值以及 null 可以被认为是 ECMAScript 字符串、数字、布尔值以及 null。JSON 使用受限更多的空白字符集合，并且允许 Unicode 码点 U+2028 和 U+2029 直接出现在 JSONString 直接量当中而无需使用转义序列。解析流程与 11.1.4 和 11.1.5 一样，但是由 JSON 语法限定。

可选参数 reviver 是一个接受两个参数的函数（key 和 value）。它可以过滤和转换结果。它在每个 key/value 对产生时被调用，它的返回值可以用于替代原本的值。如果它原样返回接收到的，那么结构不会被改变。如果它返回 undefined，那么属性会被从结果中删除。

1.  令 JText 为 ToString(text)
2.  以 15.12.1 所述语法解析 JText。如果 JText 不能以 JSON grammar 解析成 JSONText，则抛出 SyntaxError 异常。
3.  令 unfiltered 为按 ECMAScript 程序（但是用 JSONString 替换 StringLiteral）解析和执行 JText 的结果。注因 JText 符合 JSON 语法，这个结果要么是原始值类型要么是 ArrayLiteral 或者 ObjectLiteral 所定义的对象。
4.  若 IsCallable(reviver)为 true 则
    1.  令 root 为由表达式 new Object()创建的新对象，其中 Object 是以 Object 为名的标准内置的构造器。
    2.  以空字符串和属性描述{Value: unfiltered, Writable: true, Enumerable: true, Configurable: true}以及 false 为参数调用 root 的 DefineOwnProperty 内置方法。
    3.  返回传入 root 和空字符串为参数调用抽象操作 Walk 的结果，抽象操作 Walk 如下文所定义。
5.  否则，返回 unfiltered。

抽象操作 Walk 是一个递归的抽象操作，它接受两个参数：一个持有对象和表示一个该对象的属性名的 String。Walk 使用最开始被传入 parse 函数的 reviver 的值。

1.  令 val 为以参数 name 调用 hold 的 get 内部方法的结果。
2.  若 val 为对象，则
    1.  若 val 的 Class 内部属性为"Array"
        1.  设 I 为 0
        2.  令 len 为以参数"length"调用 val 的 get 内部方法的结果
        3.  当 I<len 时重复
            1.  令 newElement 为调用抽象操作 Walk 的结果，传入 val 和 ToString(I)为参数。
            2.  若 newElement 为 undefined，则
                1.  以 ToString(I)和 false 做参数，调用 val 的内部方法 Delete
                2.  否则，以 ToString(I)，属性描述{Value: newElement, Writable: true, Enumerable: true, Configurable: true}以及 false 做参数调用调用 val 的内部方法 DefineOwnProperty
            3.  对 I 加 1
    2.  否则
        1.  令 keys 为包含 val 所有的具有 Enumerable 特性的属性名 String 值的内部类型 List。字符串的顺序应当与内置函数 Object.keys 一致。
        2.  对于 keys 中的每个字符串 P 做一下
            1.  令 newElement 为调用抽象操作 Walk 的结果，传入 val 和 P 为参数。
            2.  若 newElement 为 undefined，则
                1.  以 P 和 false 做参数，调用 val 的内部方法 Delete
                2.  否则，以 P，属性描述{Value: newElement, Writable: true, Enumerable: true, Configurable: true}以及 false 做参数调用调用 val 的内部方法 DefineOwnProperty
3.  返回传入 holder 作为 this 值以及以 name 和 val 构成的参数列表调用 reviver 的 Call 内部属性的结果。

实现不允许更改 JSON.parse 的实现以扩展 JSON 语法。如果一个实现想要支持更改或者扩展过的 JSON 交换格式它必须以定义一个不同的 parse 函数的方式做这件事。

在对象中存在同名字符串的情况下，同一 key 的值会被按照文本顺序覆盖掉。

### stringify ( value [ , replacer [ , space ] ] )

stringify 函数返回一个 JSON 格式的字符串，用以表示一个 ECMAScript 值。它可以接受三个参数。第一个参数是必选的。value 参数是一个 ECMAScript 值，它通常是对象或者数组，尽管它也可以是 String, Boolean, Number 或者是 null。可选的 replacer 参数要么是个可以修改对象和数组字符串化的方式的函数，要么是个扮演选择对象字符串化的属性的白名单这样的角色的 String 和 Number 组成的数组。可选的 space 参数是一个 String 或者 Number，可以允许结果中插入空白符以改善人类可读性。

以下为字符串化一对象的步骤：

1.  令 stack 为空 List
2.  令 indent 为空 String
3.  令 PropertyList 和 ReplacerFunction 为 undefined
4.  若 Type(replacer)为 Object,则
    1.  若 IsCallable(replacer)为 true,则
        1.  令 ReplacerFunction 为 replacer
    2.  否则若 replacer 的内部属性 Class 为"Array",则
        1.  令 PropertyList 为一空内部类型 List
        2.  对于所有名是数组下标的 replacer 的属性 v.以数组下标递增顺序枚举属性
            1.  令 item 为 undefined
            2.  若 Type(v)为 String 则 令 item 为 v.
            3.  否则若 Type(v)为 Number 则 令 item 为 ToString(v).
            4.  否则若 Type(v)为 Object 则,
                1.  若 v 的 Class 内部属性为"String" or "Number"则 令 item 为 ToString(v).
            5.  若 item is not undefined and item 为 not currently an element of PropertyList 则,
                1.  Append item to the end of PropertyList.
5.  若 Type(space)为 Object 则,
    1.  若 space 的 Class 内部属性为"Number"则,
        1.  令 space 为 ToNumber(space)
    2.  否则若 space 的 Class 内部属性为"String"则,
        1.  令 space 为 ToString(space)
6.  若 Type(space)为 Number 令 space 为 min(10, ToInteger(space)) 设 gap 为一包含 space 个空格的 String. 这将会是空 String 加入 space 小于 1.
7.  否则若 Type(space)为 String
    1.  若 space 中字符个数为 10 或者更小, 设 gap 为 space，否则设 gap 为包含前 10 个 space 中字符的字符串
8.  否则 设 gap 为空 String.
9.  令 wrapper 为一个如同以表达式 new Object()创建的新对象,其中 Object 是这个名字的标准内置构造器。
10.  以参数空 String, 属性描述{Value: value, Writable: true, Enumerable: true, Configurable: true}, 和 false 调用 wrapper 的 DefineOwnProperty 内部方法。
11.  返回以空 String 和 wrapper 调用抽象方法 Str 的结果。

抽象操作 Str(key, holder)可以访问调用它的 stringify 方法中的 ReplacerFunction。其算法如下：

1.  令 value 为以 key 为参数调用 holder 的内部方法 Get
2.  若 Type(value)为 Object,则
    1.  令 toJSON 为以"toJSON"为参数调用 value 的内部方法 Get
    2.  If IsCallable(toJSON) is true
        1.  令 value 为以调用 toJSON 的内部方法 Call 的结果，传入 value 为 this 值以及由 key 构成的参数列表
3.  若 ReplacerFunction 为 not undefined,则
    1.  令 value 为以调用 ReplacerFunction 的内部方法 Call 的结果，传入 holder 为 this 值以及由 key and value 构成的参数列表
4.  若 Type(value)为 Object 则,
    1.  若 value 的 Class 内部属性为"Number"则,
        1.  令 value 为 ToNumber(value)
    2.  否则若 value 的 Class 内部属性为"String"则,
        1.  令 value 为 ToString(value)
    3.  否则若 value 的 Class 内部属性为"Boolean"则,
        1.  令 value 为 value 的 value of the PrimitiveValue 内部属性
5.  若 value 为 null 则 返回 "null".
6.  若 value 为 true 则 返回 "true".
7.  若 value 为 false 则 返回 "false".
8.  若 Type(value)为 String,则返回以 value 调用 Quote 抽象操作的结果。
9.  若 Type(value)为 Number
    1.  若 value 为 finite 则 返回 ToString(value).
    2.  否则, 返回"null".
10.  若 Type(value)为 Object,且 IsCallable(value)为 false
    1.  若 value 的 Class 内部属性为"Array"则
        1.  返回以 value 为参数调用抽象操作 JA 的结果
    2.  否则, 返回以 value.为参数调用抽象操作 value 的结果
11.  返回 undefined.

抽象操作 Quote(value)将一个 String 值封装在双引号中，并且对其中的字符转义。

1.  令 product 为双引号字符
2.  对 value 中的每一个字符 C
    1.  若 C 为双引号字符或者反斜杠字符
        1.  令 product 为 product 和反斜杠连接的结果
        2.  令 product 为 product 与 C 的连接
    2.  否则若 C 为退格，formfeed 换行回车挥着 tab
        1.  令 product 为 product 与反斜杠字符的连接
        2.  令 abbrev 为如下表所示 C 对应的字符: backspace "b" formfeed "f" newline "n" carriage return "r" tab "t"
        3.  令 product 为 product 与 abbrev 的连接
    3.  否则若 C 为代码值小于 space 的控制字符
        1.  令 product 为 product 与反斜杠字符的连接
        2.  令 product 为 product 与"u"的连接
        3.  令 hex 为转换 C 代码值按十六进制转换到四位字符串的结果
        4.  令 product 为 product 与 hex 的连接
    4.  否则
        1.  令 product 为 product 与 C 的连接
3.  令 product 为 product 与双引号字符的连接
4.  返回 product.

抽象操作 JO(value)序列化一个对象，它可以访问调用它的方法中的 stack, indent, gap, PropertyList,ReplacerFunction 以及 space

1.  若 stack 包含 value，则抛出一个 TypeError，因为对象结构中存在循环。
2.  将 value 添加到 stack.
3.  令 stepback 为 indent
4.  令 indent 为 indent 与 gap 的连接
5.  若 PropertyList 没有被定义,则
    1.  令 K 为 PropertyList
6.  否则
    1.  令 K 为以由所有 Enumerable 特性为 true 的自身属性名构成的内部 String 列表类型.
7.  令 partial 为空 List
8.  对于 K 的每一个元素 P
    1.  令 strP 为以 P 和 value 为参数调用抽象操作 Str 的结果
    2.  若 strP 没有被定义
        1.  令 member 为以 P 为参数调用抽象操作 P 的结果
        2.  令 member 为 member 与冒号字符的连接
        3.  若 gap 不为空 String
        4.  令 member 为 member 与空格字符的连接
        5.  令 member 为 member 与 strP 的连接
        6.  将 member 添加到 partial.
9.  若 partial 为 empty,则
    1.  令 final 为"{}"
10.  否则
    1.  若 gap 为空 String
        1.  令 properties 为一个连接所有 partial 中的字符串而成的字符串，键值对之间用逗号分隔。第一个字符串之前和最后一个字符串之后没有逗号。
        2.  令 final 为连接 "{", properties, 和"}"的结果
    2.  否则 gap 不是空 String
        1.  令 separator 为连接 逗号字符，换行字符以及 indent 而成的字符串。
        2.  令 properties 为一个连接所有 partial 中的字符串而成的字符串，键值对之间用 separator 分隔。第一个字符串之前和最后一个字符串之后没有 separator。
        3.  令 final 为连接"{", 换行符, indent, properties, 换行符, stepback, 和"}"的结果。
11.  移除 stack 中的最后一个元素。
12.  令 indent 为 stepback
13.  返回 final.

抽象操作 JA(value)序列化一个数组。它可以访问调用它的 stringify 方法中的 stack, indent, gap, PropertyList,ReplacerFunction 以及 space。数组的表示中仅包扩零到 array.length-1 的区间。命名的属性将会被从字符串化操作中排除。数组字符串化成开头的左方括号，逗号分隔的元素，以及结束的右方括号。

1.  若 stack 包含 value，则抛出一个 TypeError，因为对象结构中存在循环。
2.  将 value 添加到 stack.
3.  令 stepback 为 indent
4.  令 indent 为 indent 与 gap 的连接
5.  令 partial 为空 List
6.  令 len 为以"length"为参数调用 value 的内部方法 Get
7.  令 index 为 0
8.  当 index < len 时重复以下
    1.  令 strP 为 the result of calling the abstract operation Str with arguments ToString(index) and value
    2.  若 strP 没有被定义
        1.  添加 null 到 partial.
    3.  否则
        1.  添加 strP 到 partial.
    4.  使 index 增加 1.
9.  若 partial 为空,则
    1.  令 final 为"[]"
10.  否则
    1.  若 gap 为空 String
        1.  令 properties 为为一个连接所有 partial 中的字符串而成的字符串，键值对之间用逗号分隔。第一个字符串之前和最后一个字符串之后没有逗号。
        2.  令 final 为连接 "[", properties, 和"]"的结果
    2.  否则
        1.  令 separator 为逗号字符，换行字符以及 indent 而成的字符串。
        2.  令 properties 为一个连接所有 partial 中的字符串而成的字符串，键值对之间用 separator 分隔。第一个字符串之前和最后一个字符串之后没有 separator。
        3.  令 final 为连接"[", 换行符, indent, properties, 换行符, stepback, 和"]"的结果。
11.  移除 stack 中的最后一个元素。
12.  令 indent 为 stepback
13.  返回 final.

JSON 结构允许任何深度的嵌套，但是不能够循环引用。若 value 是或者包含了一个循环结构，则 stringify 函数必须抛出一个 TypeError 异常。以下是一个不能够被字符串化的值的例子：

`a = []; a[0] = a; my_text = JSON.stringify(a); // This must throw an TypeError.`

符号式简单值按以下方式表示：

1.  null 值在 JSON 文本中表示为字符串 null
2.  undefined 值不出现
3.  true 值在 JSON 文本中表示为字符串 true
4.  false 值在 JSON 文本中表示为字符串 false

字符串值用双引号括起。字符"和\会被转义成带\前缀的。控制字符用转义序列\uHHHH 替换，或者使用简略形式 \b(backspace), \f (formfeed), \n (newline), \r (carriage return), \t (tab)

有穷的数字按照调用 ToString(number)字符串化。NaN 和不论正负的 Infinity 都表示为字符串 null

没有 JSON 表示的值（如 undefined 和函数）不会产生字符串。而是会产生 undefined 值。在数组中这些值表示为字符串 null。在对象中不能表示的值会导致属性被排除在字符串化过程之外。

对象表示为开头的左大括号跟着零个或者多个属性，以逗号分隔，以右大括号结束。属性是用用来表示 key 或者属性名的引号引起的字符串，冒号然后是字符串化的属性值。数组表示为开头的左方括号，后跟零个或者多个值，以逗号分隔，以右方括号结束。