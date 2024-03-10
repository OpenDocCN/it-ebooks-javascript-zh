# 类型

本规范的算法操作各个有类型的值，可处理的类型在算法相关叙述中定义。类型又再分为 ECMAScript 语言类型 与 规范类型 。

ECMAScript 语言类型 是 ECMAScript 程序员使用 ECMAScript 语言直接操作的值对应的类型。ECMAScript 语言类型包括 未定义 （Undefined）、 空值 （Null）、 布尔值（Boolean）、 字符串 （String）、 数值 （Number）、 对象 （Object）。

规范类型 是描述 ECMAScript 语言构造与 ECMAScript 语言类型语意的算法所用的元值对应的类型。规范类型包括 引用 、 列表 、 完结 、 属性描述式 、 属性标示 、 词法环境（Lexical Environment）、 环境纪录（Environment Record）。规范类型的值是不一定对应 ECMAScript 实现里任何实体的虚拟对象。规范类型可用来描述 ECMAScript 表式运算的中途结果，但是这些值不能存成对象的变量或是 ECMAScript 语言变量的值。

在本规范中，我们将「`x` 的类型」简写为 `Type(`x`)` ，而类型指的就是上述的 ECMAScript 语言类型 与 规范类型 。

## Undefined 类型

Undefined 类型有且只有一个值，称为 `undefined` 。任何没有被赋值的变量都有 `undefined` 值。

## Null 类型

Null 类型有且只有一个值，称为 `null` 。

## Boolean 类型

Boolean 类型表示逻辑实体，有两个值，称为 `true` 和 `false`。

## String 类型

字符串类型是所有有限的零个或多个 16 位无符号整数值（“元素”）的有序序列。在运行的 ECMAScript 程序中，字符串类型常被用于表示文本数据，此时字符串中的每个元素都被视为一个代码点（参看 章节 6）。 每个元素都被认为占有此序列中的一个位置。用非负数值索引这些位置。任何时候，第一个元素（若存在）在位置 0，下一个元素（若存在）在位置 1，依此类推。字符串的长度即其中元素的个数（比如，16 位值）。空字符串长度为零，因而不包含任何元素。

若一个字符串包含实际的文本数据，每个元素都被认为是一个单独的 UTF-16 单元。无论这是不是 String 实际的存储格式，String 中的字符都被当作表示为 UTF-16 来计数。除非特别声明，作用在字符串上的所有操作都视它们为无差别的 16 位无符号整数；这些操作不保证结果字符串仍为常规化的形式，也不保证语言敏感结果。

这些决议背后的原理是尽可能地保持字符串的实现简单而高效。这意味着，在运行中的程序读到从外部进入执行环境的文本数据（即，用户输入，从文件读取文本 ，或从网络上接收文本，等等）之前，它们已被转为 Unicode 常规化形式 C。通常情况下，这个转化在进入的文本被从其原始字符编码转为 Unicode 的同时进行（且强制去除头部附加信息）。因此，建议 ECMAScript 程序源代码为常规化形式 C，（如果保证源代码文本是常规化的）保证字符串常量是常规化的，即便它们不包含任何 Unicode 转义序列。

## Number 类型

精确地，数值类型拥有 18437736874454810627（即，2⁶⁴-2⁵³ +3）个值，表示为 IEEE-754 格式 64 位双精度数值（IEEE 二进制浮点数算术中描述了它），除了 IEEE 标准中的 9007199254740990（即，2⁵³-2）个明显的“非数字 ”值；在 ECMAScript 中，它们被表示为一个单独的特殊值：`NaN`。（请注意，NaN 值由程序表达式 NaN 产生，并假设执行程序不能调整定义的全局变量 `NaN`。） 在某些实现中，外部代码也许有能力探测出众多非数字值之间的不同，但此类行为依赖于具体实现；对于 ECMAScript 代码而言，`NaN` 值相互之间无法区别。

还有另外两个特殊值，称为正无穷和负无穷。为简洁起见，在说明目的时，用符号 +∞ 和 -∞ 分别代表它们。（请注意，两个无限数值由程序表达式 `+Infinity`（简作`Infinity`） 和 `-Infinity` 产生，并假设执行程序不能调整定义的全局变量 `Infinity`。）

另外 18437736874454810624（即，2⁶⁴-2⁵³） 个值被称为有限数值。其中的一半是正数，另一半是负数，对于每个正数而言，都有一个与之对应的、相同规模的负数。

请注意，还有一个 `正零` 和一个 `负零` 。为简洁起见，类似地，在说明目的时，分别用用符号 `+0` 和 `-0` 代表这些值。（请注意，这两个数字零由程序表达式 `+0`（简作 `0`） 和`-0` 产生。）

这 18437736874454810622（即，2⁶⁴-2⁵³-2） 个有限非零值分为两种：

其中 18428729675200069632（即，2⁶⁴-2⁵⁴） 个是常规值，形如

`s * m * 2e`

这里的 s 是 +1 或 -1，m 是一个小于 2⁵³ 但不小于 2⁵² 的正整数，e 是一个闭区间 -1074 到 971 中的整数。

剩下的 9007199254740990（即，2⁵³-2）个值是非常规的，形如

`s * m * 2e`

这里的 s 是 +1 或 -1，m 是一个小于 2⁵² 的 正整数，e 为 -1074

请注意，所有规模不超过 2⁵³ 的正整数和负整数都可被数值类型表示（不过，整数 0 有两个呈现形式，+0 和 0）。

如果一个有限的数值非零且用来表达它（上文两种形式之一）的整数 m 是奇数，则该数值有 奇数标记 (odd significand)。否则，它有 偶数标记 (even significand)。

在本规范中，当 x 表示一个精确的非零实数数学量（甚至可以是无理数，比如 π）时，短语 "the number value for x" 意为，以下面的方式选择一个数字 值。考虑数值类型的所有有限值的集合（不包括 -0 和两个被加入在数值类型中但不可呈现的值，即 21024（即 +1 * 2⁵³ * 2⁹⁷¹）和 -2¹⁰²⁴ （那是 -1 * 2⁵³ * 2⁹⁷¹）。选择此集合 中值最接近 x 的一员，若集合中的两值近似相等，那么选择有偶数标记的那个；为此，2¹⁰²⁴ 和 -2¹⁰²⁴ 这两个超额值被认为有偶数标记。最终，若选择 2¹⁰²⁴ ，用 +∞替换它；若选择 -2¹⁰²⁴ ，用 -∞替换它；若选择 +0，有且只有 x 小于零时，用 -0 替换它；其它任何被选取的值都不用改变。结果就是 x 的数字值。（此过程正是 IEEE-754"round to nearest" 模式对应的行为。）

某些 ECMAScript 运算符仅涉及闭区间 -2³¹ 到 2³¹-1 的整数，或闭区间 0 到 2³²-1。这些运算符接受任何数值类型的值，不过，数值首先被转换为 2³² 个整数值中的一个。参见 ToInt32 和 ToUint32 的 描述，分别在 章节 9.5 和 9.6 中。

## Object 类型

Object 是一个属性的集合。每个属性既可以是一个命名的数据属性，也可以是一个命名的访问器属性，或是一个内部属性：

*   命名的数据属性（named data property）由一个名字与一个 ECMAScript 语言值和一个 Boolean 属性集合组成
*   命名的访问器属性（named accessor property）由一个名字与一个或两个访问器函数，和一个 Boolean 属性集合组成。访问器函数用于存取一个与该属性相关联的 ECMAScript 语言值
*   内部属性（internal property）没有名字，且不能直接通过 ECMAScript 语言操作。内部属性的存在纯粹为了规范的目的。

有两种带名字的访问器属性（非内部属性）：get 和 put，分别对应取值和赋值。

### Property 特性

本规范中的特性（Attributes）用于定义和解释命名属性（properties）的状态。命名的数据属性由一个名字关联到一个下表中列出的特性 (attributes)

命名的数据属性的特性

| 特性名称 | 取值范围 | 描述 |
| [[Value]] | 任何 ECMAScript 语言类型 | 通过读 property 来取到该值 |
| [[Writable]] | Boolean | 如果为 false，试图通过 ECMAScript 代码 [[Put]] 去改变该属性的 [[Value]]，将会失败 |
| [[Enumerable]] | Boolean | 如果为 true，则该属性可被 for-in 枚举出来（参见 12.6.4），否则，该属性不可枚举。 |
| [[Configurable]] | Boolean | 如果为 false，试图删除该属性，改变该属性为访问器属性，或改变它的 attributes（和 [[Value]] 不同），都会失败。 |

命名的访问器属性由一个名字关联到一个下表中列出的特性 (attributes)

命名的访问器属性的特性

| 特性名称 | 取值范围 | 描述 |
| [[Get]] | Object 或 Undefined | 如果该值为一个 Object 对象，那么它必须是一个函数对象。每次有对该属性进行 get 访问时，该函数的内部方法 [[Call]]（8.6.2）会被一个空参数列表调用，以返回该属性值 |
| [[Set]] | Object 或 Undefined | 如果该值为一个 Object 对象，那么它必须是一个函数对象。每次有对该属性进行 set 访问时，该函数的内部方法 [[Call]]（8.6.2）会被一个参数列表调用，这个参数列表包含分配的值作为唯一的参数。property 的内部方法 [[Set]] 产生的影响可能会，但不是必须的，对随后的 property 内部方法 [[Get]] 的调用返回结果产生影响。 |
| [[Enumerable]] | />Boolean | 如果为 true，则该属性可被 for-in 枚举出来（参见 12.6.4），否则，该属性不可枚举。 |
| [[Configurable]] | />Boolean | 如果为 false，试图删除该属性，改变该属性为访问器属性，或改变它的 attributes（和 [[Value]] 不同），都会失败。 |

如果某个命名属性的特性值没有在此规范中明确给出，那么它的默认值将使用下表的定义。

默认特性值

| 特性名称 | 默认值 |
| [[Value]] | undefined |
| [[Get]] | undefined |
| [[Set]] | undefined |
| [[Writable]] | false |
| [[Enumerable]] | false |
| [[Configurable]] | false |

### Object 内部属性及方法

本规范使用各种内部属性来定义对象值的语义。这些内部属性不是 ECMAScript 语言的一部分。本规范中纯粹是以说明为目的定义它们。ECMAScript 实现必须表现为仿佛它被这里描述的内部属性产生和操作。内部属性的名字用闭合双方括号 括起来。如果一个算法使用一个对象的一个内部属性，并且此对象没有实现需要的内部属性，那么抛出 TypeError 异常。

表 8 总结了本规范中适用于所有 ECMAScript 对象的内部属性。表 9 总结了本规范中适用于某些 ECMAScript 对象的内部属性。这些表中的描述如果没有特别指出是特定的原生 ECMAScript 对象，那么就说明了其在原生 ECMAScript 对象中的行为。宿主对象的内部属性可以支持任何依赖于实现的行为，只要其与本文档说的宿主对象的个别限制一直。

下面表的“值的类域”一列定义了内部属性关联值的类型。类型名称参考第八章定义的类型，作为增强添加了一下名称：“any”指值可以是任何 ECMAScript 语言类型；“primitive”指 Undefined, Null, Boolean, String, , Number；“SpecOp”指内部属性是一个内部方法，一个抽象操作规范定义一个实现提供它的步骤。“SpecOp”后面跟着描述性参数名的列表。如果参数名和类型名一致那么这个名字用于描述参数的类型。如果“SpecOp”有返回值，那么这个参数列表后跟着“→”符号和返回值的类型。

所有对象共有的内部属性

| 内部属性 | 取值范围 | 说明 |
| [[Prototype]] | Object 或 Null | 此对象的原型 |
| [[Class]] | String | 说明规范定义的对象分类的一个字符串值 |
| [[Extensible]] | Boolean | 如果是 true，可以向对象添加自身属性。 |
| [[Get]] | SpecOp(propertyName) → any | 返回命名属性的值 |
| [[GetOwnProperty]] | SpecOp (propertyName) → Undefined 或 Property Descriptor | 返回此对象的自身命名属性的属性描述，如果不存在返回 undefined |
| [[GetProperty]] | SpecOp (propertyName) → Undefined 或 Property Descriptor | 返回此对象的完全填入的自身命名属性的属性描述，如果不存在返回 undefined |
| [[Put]] | SpecOp (propertyName, any, Boolean) | 将指定命名属性设为第二个参数的值。flog 控制失败处理。 |
| [[CanPut]] | SpecOp (propertyName) → Boolean | 返回一个 Boolean 值，说明是否可以在 PropertyName 上执行 [[Put]] 操作。 |
| [[HasProperty]] | SpecOp (propertyName) → Boolean | 返回一个 Boolean 值，说明对象是否含有给定名称的属性。 |
| [[Delete]] | SpecOp (propertyName, Boolean) → Boolean | 从对象上删除指定的自身命名属性。flog 控制失败处理。 |
| [[DefaultValue]] | SpecOp (Hint) → primitive | Hint 是一个字符串。返回对象的默认值 |
| [[DefineOwnProperty]] | SpecOp (propertyName, PropertyDescriptor, Boolean) → Boolean | 创建或修改自身命名属性为拥有属性描述里描述的状态。flog 控制失败处理。 |

所有对象（包括宿主对象）必须实现表 8 中列出的所有内部属性。然而，对某些对象的 [[DefaultValue]] 内部方法，可以简单的抛出 TypeError 异常。

所有对象都有一个叫做 [[Prototype]] 的内部属性。此对象的值是 null 或一个对象，并且它用于实现继承。一个原生属性是否可以把宿主对象作为它的 [[Prototype]] 取决于实现。所有 [[Prototype]] 链必须是有限长度（即，从任何对象开始，递归访问 [[Prototype]] 内部属性必须最终到头，并且值是 null）。从 [[Prototype]] 对象继承来的命名数据属性（作为子对象的属性可见）是为了 get 请求，但无法用于 put 请求。命名访问器属性会把 get 和 put 请求都继承。

所有 ECMASCript 对象都有一个 Boolean 值的 [[Extensible]] 内部属性，它控制是否可以给对象添加命名属性。如果 [[Extensible]] 内部属性的值是 false 那么不得给对象添加命名属性。此外，如果 [[Extensible]] 是 false 那么不得更改对象的 [[Class]] 和 [[Prototype]] 内部属性的值。一旦 [[Extensible]] 内部属性的值设为 false 之后无法再更改为 true。

本规范的定义中没有 ECMAScript 语言运算符或内置函数允许一个程序更改对象的 [[Class]] 或 [[Prototype]] 内部属性或把 [[Extensible]] 的值从 false 更改成 true。实现中修改 [[Class]], [[Prototype]], [[Extensible]] 的个别扩展必须不违反前一段定义的不变量。

本规范的每种内置对象都定义了 [[Class]] 内部属性的值。宿主对象的 [[Class]] 内部属性的值可以是除了 "Arguments", "Array", "Boolean", "Date", "Error", "Function", "JSON", "Math", "Number", "Object", "RegExp", "String" 的任何字符串。[[Class]] 内部属性的值用于内部区分对象的种类。注，本规范中除了通过 Object.prototype.toString ( 见 15.2.4.2) 没有提供任何手段使程序访问此值。

除非特别指出，原生 ECMAScrpit 对象的公共内部方法的行为描述在 8.12。Array 对象的 [[DefineOwnProperty]] 内部方法有稍不同的实现（见 15.4.5.1），又有 String 对象的 [[GetOwnProperty]] 内部方法有稍不同的实现（见 15.5.5.2）。Arguments 对象（10.6）的 [[Get]]，[[GetOwnProperty]]，[[DefineOwnProperty]]，[[Delete]] 有不同的实现。Function 对象（15.3）的 [[Get]] 的有不同的实现。

除非特别指出，宿主对象可以以任何方式实现这些内部方法，一种可能是一个特别的宿主对象的 [[Get]] 和 [[Put]] 确实可以存取属性值，但 [[HasProperty]] 总是产生 false。然而，如果任何对宿主对象内部属性的操作不被实现支持，那么当试图操作时必须抛出 TypeError 异常。

宿主对象的 [[GetOwnProperty]] 内部方法必须符合宿主对象每个属性的以下不变量 ：

*   如果属性是描述过的数据属性，并随着时间的推移，它可能返回不同的值，那么即使没有暴露提供更改值机制的其他内部方法，[[Writable]] 和 [[Configurable]] 之一或全部必须是 true。
*   如果属性是描述过的数据属性，并且其 [[Writable]] 和 [[Configurable]] 都是 false。那么所有对 [[GetOwnProperty]] 的呼出，必须返回作为属性 [[Value]] 特性的 SameValue(9.12)。
*   如果 [[Writable]] 特性可以从 false 更改为 true，那么 [[Configurable]] 特性必须是 true。
*   当 ECMAScript 代码监测到宿主对象的 [[Extensible]] 内部属性值是 false。那么如果调用 [[GetOwnProperty]] 描述一个属性是不存在，那么接下来所有调用这个属性必须也描述为不存在。

如果 ECMAScript 代码监测到宿主对象的 [[Extensible]] 内部属性是 false，那么这个宿主对象的 [[DefineOwnProperty]] 内部方法不允许向宿主对象添加新属性。

如果 ECMAScript 代码监测到宿主对象的 [[Extensible]] 内部属性是 false，那么它以后必须不能再改为 true。

只在某些对象中定义的内部属性

Object

| 内部属性 | 取值范围 | 说明 |
| [[PrimitiveValue]] | primitive | 与此对象的内部状态信息关联。对于标准内置对象只能用 Boolean, Date, Number, String 对象实现 [[PrimitiveValue]]。 |
| [[Construct]] | SpecOp(a List of any) → Object | 通过 new 运算符调，创建对象。SpecOp 的参数是通过 new 运算符传的参数。实现了这个内部方法的对象叫做 构造器 。 |
| [[Call]] | SpecOp(any, a List of any) → any or Reference | 运行与此对象关联的代码。通过函数调用表达式调用。SpecOp 的参数是一个 this 对象和函数调用表达式传来的参数组成的列表。实现了这个内部方法的对象是 可调用 的。只有作为宿主对象的可调用对象才可能返回 引用 值。 |
| [[HasInstance]] | SpecOp(any) → Boolean | 返回一个表示参数对象是否可能是由本对象构建的布尔值。在标准内置 ECMAScript 对象中只有 Function 对象实现 [[HasInstance]]。 |
| [[Scope]] | Lexical Environment | 一个定义了函数对象执行的环境的词法环境。在标准内置 ECMAScript 对象中只有 Function 对象实现 [[Scope]]。 |
| [[FormalParameters]] | List of Strings | 一个包含 Function 的 FormalParameterList 的标识符字符串的可能是空的列表。在标准内置 ECMAScript 对象中只有 Function 对象实现 [[FormalParameterList]]。 |
| [[Code]] | ECMAScript code | 函数的 ECMAScript 代码。在标准内置 ECMAScript 对象中只有 Function 对象实现 [[Code]]。 |
| [[TargetFunction]] | Object | 使用标准内置的 Function.prototype.bind 方法创建的函数对象的目标函数。只有使用 Function.prototype.bind 创建的 ECMAScript 对象才有 [[TargetFunction]] 内部属性。 |
| [[BoundThis]] | any | 使用标准内置的 Function.prototype.bind 方法创建的函数对象的预绑定的 this 值。只有使用 Function.prototype.bind 创建的 ECMAScript 对象才有 [[BoundThis]] 内部属性。 |
| [[BoundArguments]] | List of any | 使用标准内置的 Function.prototype.bind 方法创建的函数对象的预绑定的参数值。只有使用 Function.prototype.bind 创建的 ECMAScript 对象才有 [[BoundArguments]] 内部属性。 |
| [[Match]] | SpecOp(String, index) → MatchResult | 测试正则匹配并返回一个 MatchResult 值（见 15.10.2.1）。在标准内置 ECMAScript 对象中只有 RegExp 对象实现 [[Match]]。 |
| [[ParameterMap]] |  | 提供参数对象的属性和函数关联的形式参数之间的映射。只有参数对象才有 [[ParameterMap]] 内部属性。 |

## 引用规范类型

引用类型用来说明 delete，typeof，赋值运算符这些运算符的行为。例如，在赋值运算中左边的操作数期望产生一个引用。通过赋值运算符左侧运算子的语法案例分析可以但不能完全解释赋值行为，还有个难点：函数调用允许返回引用。承认这种可能性纯粹是为了宿主对象。本规范没有定义返回引用的内置 ECMAScript 函数，并且也不提供返回引用的用户定义函数。（另一个不使用语法案列分析的原因是，那样将会影响规范的很多地方，冗长并且别扭。）

一个 引用 (Reference) 是个已解决的命名绑定。一个引用由三部分组成， 基 (base) 值， 引用名称（referenced name） 和布尔值 严格引用 (strict reference) 标志。基值是 undefined, 一个 Object, 一个 Boolean, 一个 String, 一个 Number, 一个 environment record 中的任意一个。基值是 undefined 表示此引用可以不解决一个绑定。引用名称是一个字符串。

本规范中使用以下抽象操作接近引用的组件：

*   GetBase(V)。 返回引用值 V 的基值组件。
*   GetReferencedName(V)。 返回引用值 V 的引用名称组件。
*   IsStrictReference(V)。 返回引用值 V 的严格引用组件。
*   HasPrimitiveBase(V)。 如果基值是 Boolean, String, Number，那么返回 true。
*   IsPropertyReference(V)。 如果基值是个对象或 HasPrimitiveBase(V) 是 true，那么返回 true；否则返回 false。
*   IsUnresolvableReference(V)。 如果基值是 undefined 那么返回 true，否则返回 false。

本规范使用以下抽象操作来操作引用：

### GetValue(v)

1.  如果 Type(V) 不是引用 , 返回 V。
2.  令 base 为调用 GetBase(V) 的返回值。
3.  如果 IsUnresolvableReference(V), 抛出一个 ReferenceError 异常。
4.  如果 IsPropertyReference(V), 那么
    1.  如果 HasPrimitiveBase(V) 是 false, 那么令 get 为 base 的 [[Get]] 内部方法 , 否则令 get 为下面定义的特殊的 [[Get]] 内部方法。
    2.  将 base 作为 this 值，传递 GetReferencedName(V) 为参数，调用 get 内部方法，返回结果。
5.  否则 , base 必须是一个 environment record。
6.  传递 GetReferencedName(V) 和 IsStrictReference(V) 为参数调用 base 的 GetBindingValue( 见 10.2.1) 具体方法，返回结果。

GetValue 中的 V 是原始基值的 属性引用 时使用下面的 [[Get]] 内部方法。它用 base 作为他的 this 值，其中属性 P 是它的参数。采用以下步骤：

1.  令 O 为 ToObject(base)。
2.  令 desc 为用属性名 P 调用 O 的 [[GetProperty]] 内部方法的返回值。
3.  如果 desc 是 undefined，返回 undefined。
4.  如果 IsDataDescriptor(desc) 是 true，返回 desc.[[Value]]。
5.  否则 IsAccessorDescriptor(desc) 必须是 true，令 getter 为 desc.[[Get]]。
6.  如果 getter 是 undefined，返回 undefined。
7.  提供 base 作为 this 值，无参数形式调用 getter 的 [[Call]] 内部方法，返回结果。

上述方法之外无法访问在第一步创建的对象。实现可以选择不真的创建这个对象。使用这个内部方法给实际属性访问产生可见影响的情况只有在调用访问器函数时。

### PutValue(v,w)

1.  如果 Type(V) 不是引用，抛出一个 ReferenceError 异常。
2.  令 base 为调用 GetBase(V) 的结果。
3.  如果 IsUnresolvableReference(V)，那么
    1.  如果 IsStrictReference(V) 是 true，那么
        1.  抛出 ReferenceError 异常。
    2.  用 GetReferencedName(V)，W，false 作为参数调用全局对象的 [[Put]] 内部方法。
4.  否则如果 IsPropertyReference(V)，那么
    1.  如果 HasPrimitiveBase(V) 是 false，那么令 put 为 base 的 [[Put]] 内部方法，否则令 put 为下面定义的特殊的 [[Put]] 内部方法。
    2.  用 base 作为 this 值，用 GetReferencedName(V)，W，IsStrictReference(V) 作为参数调用 put 内部方法。
5.  否则 base 必定是 environment record 作为 base 的引用。所以，
    1.  用 GetReferencedName(V), W, IsStrictReference(V) 作为参数调用 base 的 SetMutableBinding (10.2.1) 具体方法。
6.  返回。

PutValue 中的 V 是原始基值的属性引用时使用下面的 [[Put]] 内部方法。用 base 作为 this 值，用属性 P，值 W，布尔标志 Throw 作为参数调用它。采用以下步骤：

1.  令 O 为 ToObject(base)。
2.  如果用 P 作为参数调用 O 的 [[CanPut]] 内部方法的结果是 false，那么
    1.  如果 Throw 是 true，那么抛出一个 TypeError 异常。
    2.  否则返回。
3.  令 ownDesc 为用 P 作为参数调用 O 的 [[GetOwnProperty]] 内部方法的结果。
4.  如果 IsDataDescriptor(ownDesc) 是 true，那么
    1.  如果 Throw 是 true，那么抛出一个 TypeError 异常。
    2.  否则返回。
5.  令 desc 为用 P 作为参数调用 O 的 [[GetProperty]] 内部方法的结果。这可能是一个自身或继承的访问器属性描述或是一个继承的数据属性描述。
6.  如果 IsAccessorDescriptor(desc) 是 true，那么
    1.  令 setter 为 desc.Set，他不能是 undefined。
    2.  用 base 作为 this 值，用只由 W 组成的列表作为参数调用 setter 的 [[Call]] 内部方法。
7.  否则，这是要在临时对象 O 上创建自身属性的请求。
    1.  如果 Throw 是 true，抛出一个 TypeErroe 异常。
8.  返回。

上述方法之外无法访问在第一步创建的对象。实现可以选择不真的创建这个临时对象。使用这个内部方法给实际属性访问产生可见影响的情况只有在调用访问器函数时，或 Throw 未通过提前错误检查。当 Throw 是 true，试图在这个临时对象上创建新属性的任何属性分配操作会抛出一个错误。

## 列表规范类型

列表类型用于说明 new 表达式，函数调用，其他需要值的简单列表的算法 -- 里的参数列表的计算。列表类型的值是简单排序的一些值的序列。此序列可以是任意长度。

## 完结规范类型

完结类型用于说明执行将控制转移到外部的声明 (break, continue, return, throw) 的行为。完结类型的值是由三部分组成，形如（type，value，target），其中 type 是 normal, break, continue, return, throw 之一，value 是任何 ECMASCript 语言值或 empty，target 是任何 ECMAScript 标识符或 empty。

术语“突然完结（abrupt completion）”是指任何非正常完成的完成类型。

## 属性描述符及属性标识符规范类型

属性描述符类型是用来解释命名属性的具体的操作的特性集。属性描述符类型的值是记录项，由命名字段组成，每个字段的名称是一个特性名并且它的值是一个相应的特性值，这些特性指定在 8.6.1。此外，任何字段都可能存在或不存在。

根据是否存在或使用了某些字段，属性描述符的值可进一步划分为数据属性描述符和访问器属性描述符。一个数据属性描述符 里包括叫做 [[Value]] 或 [[Writable]] 的字段。一个访问器属性描述符里包括叫做 [[Get]] 或 [[Set]] 的字段。任何属性描述都可能有名为 [[Enumerable]] 和 [[Configurable]] 的字段。一个属性描述符不能同时是数据属性描述符和访问器属性描述符；但是，它可能二者都不是。一个通用属性描述符是，既不是数据属性描述符也不是访问器属性描述符的属性描述符值。一个完全填充属性描述符是访问器属性描述符或数据属性描述符，并且拥有 8.6.1 Table 5 或 Table 6 里定义的所有属性特性对应的字段。

本规范中为了便于标记，使用一种类似对象字面量的语法来定义属性描述符。例如，属性描述符 {[[Value]]: 42, [[Writable]]: false, [[Configurable]]: true}，就定义了一个数据属性描述符。字段名称的顺序并不重要。任何没有明确列出的字段被认为是不存在的。

在规范中的文本和算法里，可用点符号来指明一个属性描述符的特定字段。例如，如果 D 是一个属性描述符，那么 D.[[Value]] 是“D 的 [[Value]] 字段”的简写。

属性标识符类型用于关联属性名称与属性描述符。属性标识符类型的值是 (name, descriptor) 形式的一对值，其中 name 是一个字符串和 descriptor 是一个属性描述符值。

在本规范中使用以下的抽象操作来操作属性描述符值：

### IsAccessorDescriptor (Desc)

当用属性描述 Desc 调用抽象操作 IsAccessorDescriptor，采用以下步骤：

1.  如果 Desc 是 undefined，那么返回 false。
2.  如果 Desc.[[Get]] 和 Desc.[[Set]] 都不存在，则返回 false。
3.  返回 true。

### IsDataDescriptor (Desc)

当用属性描述 Desc 调用抽象操作 IsDataDescriptor，采用以下步骤：

1.  如果 Desc 是 undefined，那么返回 false。
2.  如果 Desc.[[Value]] 和 Desc.[[Writable]] 都不存在，则返回 false。
3.  返回 true。

### IsGenericDescriptor (Desc)

当用属性描述 Desc 调用抽象操作 IsDataDescriptor，采用以下步骤：

1.  如果 Desc 是 undefined，那么返回 false。
2.  如果 IsAccessorDescriptor(Desc) 和 IsDataDescriptor(Desc) 都是 false, 则返回 true。
3.  返回 false。

### FromPropertyDescriptor (Desc)

当用属性描述 Desc 调用抽象操作 FromPropertyDescriptor，采用以下步骤：

假定以下算法的 Desc 是 [[GetOwnProperty]]( 见 8.12.1) 返回的完全填充的 属性描述。

1.  如果 Desc 是 undefined，那么返回 false。
2.  令 obj 为仿佛使用 new Object() 表达式创建的新对象，这里的 Object 是标准内置构造器名。
3.  如果 IsDataDescriptor(Desc) 是 true，则
    1.  用参数 "value", 属性描述符 {[[Value]]: Desc.[[Value]], [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, false 调用 obj 的 [[DefineOwnProperty]] 内部方法。
    2.  用参数 "writable", 属性描述符 {[[Value]]: Desc.[[Writable]], [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, false 调用 obj 的 [[DefineOwnProperty]] 内部方法。
4.  否则，IsAccessorDescriptor(Desc) 必定是 true，所以
    1.  用参数 "get", 属性描述符 {[[Value]]: Desc.[[Get]], [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, false 调用 obj 的 [[DefineOwnProperty]] 内部方法。
    2.  用参数 "set", 属性描述符 {[[Value]]: Desc.[[Set]], [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, false 调用 obj 的 [[DefineOwnProperty]] 内部方法。
5.  用参数 "enumerable", 属性描述符 {[[Value]]: Desc.[[Enumerable]], [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, false 调用 obj 的 [[DefineOwnProperty]] 内部方法。
6.  用参数 "configurable", 属性描述符 {[[Value]]: Desc.[[Configurable]], [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, false 调用 obj 的 [[DefineOwnProperty]] 内部方法。
7.  返回 obj。

### ToPropertyDescriptor (Obj)

当用对象 Desc 调用抽象操作 FromPropertyDescriptor，采用以下步骤：

1.  如果 Type(Obj) 不是 Object，抛出一个 TypeError 异常。
2.  令 desc 为创建初始不包含字段的新属性描述的结果。
3.  如果用参数 "enumerable" 调用 Obj 的 [[HasProperty]] 内部方法的结果是 true，则
    1.  令 enum 为用参数 "enumerable" 调用 Obj 的 [[Get]] 内部方法的结果。
    2.  设定 desc 的 [[Enumerable]] 字段为 ToBoolean(enum)。
4.  如果参数 "configurable" 调用 Obj 的 [[HasProperty]] 内部方法的结果是 true，则
    1.  令 conf 为用参数 "enumerable" 调用 Obj 的 [[Get]] 内部方法的结果。
    2.  设定 desc 的 [[Configurable]] 字段为 ToBoolean(conf)。
5.  如果参数 "value" 调用 Obj 的 [[HasProperty]] 内部方法的结果是 true，则
    1.  令 value 为用参数 "value" 调用 Obj 的 [[Get]] 内部方法的结果。
    2.  设定 desc 的 [[Value]] 字段为 value。
6.  如果参数 "writable" 调用 Obj 的 [[HasProperty]] 内部方法的结果是 true，则
    1.  令 writable 为用参数 "writable" 调用 Obj 的 [[Get]] 内部方法的结果。
    2.  设定 desc 的 [[Writable]] 字段为 ToBoolean(writable)。
7.  如果参数 "get" 调用 Obj 的 [[HasProperty]] 内部方法的结果是 true，则
    1.  令 getter 为用参数 "get" 调用 Obj 的 [[Get]] 内部方法的结果。
    2.  如果 IsCallable(getter) 是 false 并且 getter 不是 undefined，则抛出一个 TypeError 异常。
    3.  设定 desc 的 [[Get]] 字段为 getter。
8.  如果参数 "set" 调用 Obj 的 [[HasProperty]] 内部方法的结果是 true，则
    1.  令 setter 为用参数 "set" 调用 Obj 的 [[Get]] 内部方法的结果。
    2.  如果 IsCallable(setter) 是 false 并且 setter 不是 undefined，则抛出一个 TypeError 异常。
    3.  设定 desc 的 [[Set]] 字段为 Setter。
9.  如果存在 desc.[[Get]] 或 desc.[[Set]]，则
    1.  如果存在 desc.[[Value]] 或 desc.[[Writable]]，则抛出一个 TypeError 异常。
10.  返回 desc

## 词法环境和环境记录项规范类型

词法环境和环境记录项类型用于说明在嵌套的函数或块中的名称解析行为。这些类型和他们的操作定义在第十章。

## 对象内部方法的算法

在以下算法说明中假定 O 是一个原生 ECMAScript 对象，P 是一个字符串，Desc 是一个属性说明记录，Throw 是一个布尔标志。

### [[GetOwnProperty]](P)

当用属性名 P 调用 O 的 [[GetOwnProperty]] 内部方法，采用以下步骤：

1.  如果 O 不包含名为 P 的自身属性，返回 undefined。
2.  令 D 为无字段的新建属性描述。
3.  令 X 为 O 的名为 P 的自身属性。
4.  如果 X 是数据属性，则
    1.  设定 D.[[Value]] 为 X 的 [[Value]] 特性值。
    2.  设定 D.[[Writable]] 为 X 的 [[Writable]] 特性值。
5.  否则 X 是访问器属性，所以
    1.  设定 D.[[Get]] 为 X 的 [[Get]] 特性值。
    2.  设定 D.[[Set]] 为 X 的 [[Set]] 特性值。
6.  设定 D.[[Enumerable]] 为 X 的 [[Enumerable]] 特性值。
7.  设定 D.[[Configurable]] 为 X 的 [[Configurable]] 特性值。
8.  返回 D。

然而，如果 O 是一个字符串对象，关于其 [[GetOwnProperty]] 的更多阐述定义在 15.5.5.2。

### [[GetProperty]] (P)

当用属性名 P 调用 O 的 [[GetProperty]] 内部方法，采用以下步骤：

1.  令 prop 为用属性名 P 调用 O 的 [[GetOwnProperty]] 内部方法的结果。
2.  如果 prop 不是 undefined，返回 prop。
3.  令 proto 为 O 的 [[Prototype]] 内部属性值。
4.  如果 proto 是 null，返回 undefined。
5.  用参数 P 调用 proto 的 [[GetProperty]] 内部方法，返回结果。

### [[Get]] (P)

当用属性名 P 调用 O 的 [[Get]] 内部方法，采用以下步骤：

1.  令 desc 为用属性名 P 调用 O 的 [[GetProperty]] 内部方法的结果。
2.  如果 desc 是 undefined，返回 undefined。
3.  如果 IsDataDescriptor(desc) 是 true，返回 desc.[[Value]]。
4.  否则，IsAccessorDescriptor(desc) 必定是真，所以，令 getter 为 desc.[[Get]]。
5.  如果 getter 是 undefined，返回 undefined。
6.  用 O 作为 this，无参数调用 getter 的 [[Call]] 内部方法，返回结果。

### [[CanPut]] (P)

当用属性名 P 调用 O 的 [[CanPut]] 内部方法，采用以下步骤：

1.  令 desc 为用参数 P 调用 O 的 [[GetOwnProperty]] 内部方法的结果。
2.  如果 desc 不是 undefined，则
    1.  如果 IsAccessorDescriptor(desc) 是 true，则
        1.  如果 desc.[[Set]] 是 undefined，则返回 false。
        2.  否则返回 true。
    2.  否则，desc 必定是 DataDescriptor，所以返回 desc.[[Writable]] 的值。
3.  令 proto 为 O 的 [[Prototype]] 内部属性。
4.  如果 proto 是 null，则返回 O 的 [[Extensible]] 内部属性的值。
5.  令 inherited 为用属性名 P 调用 proto 的 [[GetProperty]] 内部方法的结果。
6.  如果 inherited 是 undefined，返回 O 的 [[Extensible]] 内部属性的值。
7.  如果 IsAccessorDescriptor(inherited) 是 true，则
    1.  如果 inherited.[[Set]] 是 undefined，则返回 false。
    2.  否则返回 true。
8.  否则，inherited 必定是 DataDescriptor
    1.  如果 O 的 [[Extensible]] 内部属性是 false，返回 false。
    2.  否则返回 inherited.[[Writable]] 的值。

宿主对象可以定义受额外约束的 [[Put]] 操作。如果可能，宿主对象不应该在 [[CanPut]] 返回 false 的情况下允许 [[Put]] 操作。

### [[Put]] (P, V, Throw)

当用属性名 P，值 V，布尔值 Throw 调用 O 的 [[Put]] 内部方法，采用以下步骤：

1.  如果用参数 P 调用 O 的 [[CanPut]] 内部方法的结果是 false，则
    1.  如果 Throw 是 true，则抛出一个 TypeError 异常。
    2.  否则返回。
2.  令 ownDesc 为用参数 P 调用 O 的 [[GetOwnProperty]] 内部方法的结果。
3.  如果 IsDataDescriptor(ownDesc) 是 true，则
    1.  令 valueDesc 为属性描述 {[[Value]]: V}。
    2.  用参数 P，valueDesc，Throw 调用 O 的 [[DefineOwnProperty]] 内部方法。
    3.  返回。
4.  令 desc 为用参数 P 调用 O 的 [[GetProperty]] 内部方法的结果。这可能是自身或继承的访问器属性描述或者是继承的数据属性描述。
5.  如果 IsAccessorDescriptor(desc) 是 true，则
    1.  令 setter 为不是 undefined 的 desc.[[Set]]。
    2.  用 O 作为 this，V 作为唯一参数调用 setter 的 [[Call]] 内部方法。
6.  否则，按照以下步骤在对象 O 上创建名为 P 的命名数据属性。
    1.  令 newDesc 为属性描述 {[[Value]]: V, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}。
    2.  用参数 P, newDesc, Throw 调用 O 的 [[DefineOwnProperty]] 内部方法。
7.  返回。

### [[HasProperty]] (P)

当用属性名 P 调用 O 的 [[HasProperty]] 内部方法，采用以下步骤：

1.  令 desc 为用属性名 P 调用 O 的 [[GetProperty]] 内部方法的结果。
2.  如果 desc 是 undefined，则返回 false。
3.  否则返回 true。

### [[Delete]] (P, Throw)

当用属性名 P 和布尔值 Throw 调用 O 的 [[Delete]] 内部方法，采用以下步骤：

1.  令 desc 为用属性名 P 调用 O 的 [[GetOwnProperty]] 内部方法的结果。
2.  如果 desc 是 undefined，则返回 true。
3.  如果 desc.[[Configurable]] 是 true，则
    1.  在 O 上删除名为 P 的自身属性。
    2.  返回 true。
4.  否则如果 Throw，则抛出一个 TypeError 异常。
5.  返回 false。

### [[DefaultValue]] (hint)

当用字符串 hint 调用 O 的 [[DefaultValue]] 内部方法，采用以下步骤：

1.  令 toString 为用参数 "toString" 调用对象 O 的 [[Get]] 内部方法的结果。
2.  如果 IsCallable(toString) 是 true，则
    1.  令 str 为用 O 作为 this 值，空参数列表调用 toString 的 [[Call]] 内部方法的结果。
    2.  如果 str 是原始值，返回 str。
3.  令 valueOf 为用参数 "valueOf" 调用对象 O 的 [[Get]] 内部方法的结果。
4.  如果 IsCallable(valueOf) 是 true，则
    1.  令 val 为用 O 作为 this 值，空参数列表调用 valueOf 的 [[Call]] 内部方法的结果。
    2.  如果 val 是原始值，返回 val。
5.  抛出一个 TypeError 异常。

当用数字 hint 调用 O 的 [[DefaultValue]] 内部方法，采用以下步骤：

1.  令 valueOf 为用参数 "valueOf" 调用对象 O 的 [[Get]] 内部方法的结果。
2.  如果 IsCallable(valueOf) 是 true，则
    1.  令 val 为用 O 作为 this 值，空参数列表调用 valueOf 的 [[Call]] 内部方法的结果。
    2.  如果 val 是原始值，返回 val。
3.  令 toString 为用参数 "toString" 调用对象 O 的 [[Get]] 内部方法的结果。
4.  如果 IsCallable(toString) 是 true，则
    1.  令 str 为用 O 作为 this 值，空参数列表调用 toString 的 [[Call]] 内部方法的结果。
    2.  如果 str 是原始值，返回 str。
5.  抛出一个 TypeError 异常。

当不用 hint 调用 O 的 [[DefaultValue]] 内部方法，O 是 Date 对象的情况下仿佛 hint 是字符串一样解释它的行为，除此之外仿佛 hint 是数字一样解释它的行为。

上面说明的 [[DefaultValue]] 在原生对象中只能返回原始值。如果一个宿主对象实现了它自身的 [[DefaultValue]] 内部方法，那么必须确保其 [[DefaultValue]] 内部方法只能返回原始值。

### [[DefineOwnProperty]] (P, Desc, Throw)

在以下算法中，术语“拒绝”指代“如果 Throw 是 true，则抛出 TypeError 异常，否则返回 false。算法包含测试具体值的属性描述 Desc 的各种字段的步骤。这种方式测试的字段事实上不需要真的在 Desc 里。如果一个字段不存在则将其值看作是 false。

当用属性名 P，属性描述 Desc，布尔值 Throw 调用 O 的 [[DefineOwnProperty]] 内部方法，采用以下步骤：

1.  令 current 为用属性名 P 调用 O 的 [[GetOwnProperty]] 内部属性的结果。
2.  令 extensible 为 O 的 [[Extensible]] 内部属性值。
3.  如果 current 是 undefined 并且 extensible 是 false，则拒绝。
4.  如果 current 是 undefined 并且 extensible 是 true，则
    1.  如果 IsGenericDescriptor(Desc) 或 IsDataDescriptor(Desc) 是 true，则
        1.  在 O 上创建名为 P 的自身数据属性，Desc 描述了它的 [[Value]], [[Writable]], [[Enumerable]]，[[Configurable]] 特性值。如果 Desc 的某特性字段值不存在，那么设定新建属性的此特性为默认值。
    2.  否则，Desc 必定是访问器属性描述，所以
        1.  在 O 上创建名为 P 的自身访问器属性，Desc 描述了它的 [[Get]], [[Set]], [[Enumerable]], [[Configurable]] 特性值。如果 Desc 的某特性字段值不存在，那么设定新建属性的此特性为默认值。
5.  如果 Desc 不存在任何字段，返回 true。
6.  如果 Desc 的任何字段都出现在 current 中，并且用 SameValue 算法比较 Desc 中每个字段值和 current 里对应字段值，结果相同，则返回 true。
7.  如果 current 的 [[Configurable]] 字段是 false，则
    1.  如果 Desc 的 [[Configurable]] 字段是 true，则拒绝。
    2.  如果 Desc 有 [[Enumerable]] 字段，并且 current 和 Desc 的 [[Enumerable]] 字段相互布尔否定，则拒绝。
8.  IsGenericDescriptor(Desc) 是 true，则不需要进一步验证。
9.  否则，如果 IsDataDescriptor(current) 和 IsDataDescriptor(Desc) 的结果不同，则
    1.  如果 current 的 [[Configurable]] 字段是 false，则拒绝。
    2.  如果 IsDataDescriptor(current) 是 true，则
        1.  将对象 O 的名为 P 的数据属性转换为访问器属性。保留转换属性的 [[Configurable]] 和 [[Enumerable]] 特性的现有值，并且设定属性的其余特性为其默认值。
    3.  否则
        1.  将对象 O 的名为 P 的访问器属性转换为数据属性。保留转换属性的 [[Configurable]] 和 [[Enumerable]] 特性的现有值，并且设定属性的其余特性为其默认值。
10.  否则，如果 IsDataDescriptor(current) 和 IsDataDescriptor(Desc) 都是 true，则
    1.  如果 current 的 [[Configurable]] 字段是 false，则
        1.  如果 current 的 [[Writable]] 字段是 false 并且 Desc 的 [[Writable]] 字段是 true，则拒绝。
        2.  如果 current 的 [[Writable]] 字段是 false，则
            1.  如果 Desc 有 [[Value]] 字段，并且 SameValue(Desc.[[Value]], current.[[Value]]) 是 false，则拒绝。
    2.  否则，current 的 [[Configurable]] 字段是 true，所以可接受任何更改。
11.  否则 IsAccessorDescriptor(current) 和 IsAccessorDescriptor(Desc) 都是 true，所以
    1.  如果 current 的 [[Configurable]] 字段是 false，则
        1.  如果 Desc 有 [[Set]] 字段，并且 SameValue(Desc.[[Set]], current.[[Set]]) 是 false，则拒绝。
        2.  如果 Desc 有 [[Get]] 字段，并且 SameValue(Desc.[[Set]], current.[[Get]]) 是 false，则拒绝。
12.  Desc 拥有所有特性字段，设定对象 O 的名为 P 的属性的对性特性为这些字段值。
13.  返回 true。

然而，如果 O 是一个 Array 对象，其 [[DefineOwnProperty]] 内部方法的更多阐述定义在 15.4.5.1

如果 current 的 [[Configurable]] 字段是 true，那么步骤 10.b 允许 Desc 的任何字段与 current 对应的字段不同。这甚至可以改变 [[Writable]] 特性为 false 的属性的 [[Value]]。允许这种情况是因为值是 true 的 [[Configurable]] 特性会允许按照：首先设定 [[Writable]] 为 true，然后设定新 [[Value]]，[[Writable]] 设为 false 的顺序调用。