# 表达式

## 主值表达式

语法：

`PrimaryExpression : this Identifier Literal ArrayLiteral ObjectLiteral ( Expression )`

### this 关键字

this 关键字执行为当前执行环境的 ThisBinding。

### 标识符引用

Identifier 的执行遵循 10.3.1 所规定的标识符查找。标识符执行的结果总是一个 Reference 类型的值。

### 字面量引用

Literal 按照 7.8 所描述的方式执行。

### 数组初始化

数组初始化是一个以字面量的形式书写的描述数组对象的初始化的表达式。它是一个零个或者多个表达式的序列，其中每一个表示一个数组元素，并且用方括号括起来。元素并不一定要是字面量，每次数组初始化执行时它们都会被执行一次。

数组元素可能在元素列表的开始、结束，或者中间位置被省略。当元素列表中的一个逗号没有被 AssignmentExpression 优先处理（如，一个逗号在另一个逗号之前。）的情况下，缺失的数组元素仍然会对数组长度有贡献，并且增加后续元素的索引值。省略数组元素是没有定义的。假如元素在数组末尾被省略，那么元素不会贡献数组长度。

语法：

`ArrayLiteral : [ Elisionopt ] [ ElementList ] [ ElementList , Elisionopt ]``ElementList : ElisionoptAssignmentExpression ElementList , ElisionoptAssignmentExpression``Elision : , Elision ,`

语义：

产生式 ArrayLiteral : [ Elision[opt] ] 按照下面的过程执行 :

1.  令 array 为以表达式 new Array() 完全一致的方式创建一个新对象的结果，其中 Array 是一个标准的内置构造器
2.  令 pad 为解释执行 Elision 的结果 ; 如果不存在的话，使用数值 0.
3.  以参数 "length", pad, 和 false 调用 array 的 [[Put]] 内置方法
4.  返回 array.

产生式 ArrayLiteral : [ ElementList ] 按照下面的过程执行 :

1.  返回解释执行 ElementList 的结果 .

产生式 ArrayLiteral : '[ ElementList , 'Elision[opt] ] 按照下面的过程执行 :

1.  令 array 为解释执行 ElementList 的结果 .
2.  令 pad 为解释执行 Elision 的结果 ; 如果不存在的话，使用数值 0.
3.  令 len 为以参数 "length". 调用 array 的 [[Get]] 内置方法的结果
4.  以参数 "length", ToUint32(pad+len), 和 false 调用 array 的 [[Put]] 内置方法
5.  返回 array.

产生式 ElementList : Elision[opt] AssignmentExpression 按照下面的过程执行 :

1.  令 array 为以表达式 new Array() 完全一致的方式创建一个新对象的结果，其中 Array 是一个标准的内置构造器
2.  令 firstIndex 为解释执行 Elision 的结果 ; 如果不存在的话，使用数值 0.
3.  令 initResult 为解释执行 AssignmentExpression 的结果 .
4.  令 initValue 为 GetValue(initResult).
5.  以参数 ToString(firstIndex), 属性描述对象 { [[Value]]: initValue, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, 和 false 调用 array 的 [[DefineOwnProperty]] 内置方法
6.  返回 array.

产生式 ElementList : ElementList , Elision[opt] AssignmentExpression 按照下面的过程执行 :

1.  令 array 为解释执行 ElementList 的结果 .
2.  令 pad 为解释执行 Elision 的结果 ; 如果不存在的话，使用数值 0.
3.  令 initResult 为解释执行 AssignmentExpression 的结果 .
4.  令 initValue 为 GetValue(initResult).
5.  令 len 为以参数 "length". 调用 array 的 [[Get]] 内置方法的结果
6.  以参数 ToString(ToUint32((pad+len)) 和 属性描述对象 { [[Value]]: initValue, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}, 和 false 调用 array 的 [[DefineOwnProperty]] 内置方法
7.  返回 array.

产生式 Elision : , 按照下面的过程执行 :

1.  返回数值 1.

产生式 Elision : Elision , 按照下面的过程执行 :

1.  令 preceding 为解释执行 Elision 的结果 .
2.  返回 preceding+1.

[[DefineOwnProperty]] 用于确保即使默认的数组原型对象被更改的情况下自身属性也会被定义，可以杜绝用 [[put]] 创建一个新的自身属性。

### 对象初始化

对象初始化是一个以直接量的方式描述对象的初始化过程的表达式。它是用花括号括起来的由零或者多对属性名 / 关联值组成的列表，值不需要是直接量，每次对象初始化被执行到时他们会执行一次。

语法：

`ObjectLiteral : { } { PropertyNameAndValueList } { PropertyNameAndValueList , }``PropertyNameAndValueList : PropertyAssignment PropertyNameAndValueList , PropertyAssignment``PropertyAssignment : PropertyName : AssignmentExpression get PropertyName() { FunctionBody } set PropertyName( PropertySetParameterList ) { FunctionBody }``PropertyName : IdentifierName StringLiteral NumericLiteral` `PropertySetParameterList : Identifier`

语义：

产生式 ObjectLiteral : { } 按照下面的过程执行 :

1.  返回 a new object created as if by the expression new Object() where Object is the st 和 ard built-in constructor with that name.

产生式 s ObjectLiteral : { PropertyNameAndValueList } 以及 ObjectLiteral : { PropertyNameAndValueList ,} 按照下面的过程执行 :

1.  返回解释执行 PropertyNameAndValueList 的结果 .

产生式 PropertyNameAndValueList : PropertyAssignment 按照下面的过程执行 :

1.  令 obj 为以表达式 new Object() 完全一致的方式创建一个新对象的结果，其中 Object 是一个标准的内置构造器
2.  令 propId 为解释执行 PropertyAssignment 的结果 .
3.  以参数 propId.name, propId.descriptor, 和 false 调用 obj 的 [[DefineOwnProperty]] 内置方法
4.  返回 obj.

产生式 PropertyNameAndValueList : PropertyNameAndValueList , PropertyAssignment 按照下面的过程执行 :

1.  令 obj 为解释执行 PropertyNameAndValueList 的结果 .
2.  令 propId 为解释执行 PropertyAssignment 的结果 .
3.  令 previous 为以参数 propId.name. 调用 obj 的 [[GetOwnProperty]] 内置方法的结果
4.  如果 previous 不是 undefined，且当以下任意一个条件为 true 时，则抛出一个 SyntaxError 异常：
    *   产生式包含在严格模式下并且 IsDataDescriptor(previous) 为 true 并且 IsDataDescriptor(propId.descriptor) 为 true
    *   IsDataDescriptor(previous) 为 true 并且 IsAccessorDescriptor(propId.descriptor) 为 true.
    *   IsAccessorDescriptor(previous) 为 true 并且 IsDataDescriptor(propId.descriptor) 为 true.
    *   IsAccessorDescriptor(previous) 为 true 并且 IsAccessorDescriptor(propId.descriptor) 为 true 并且 previous 和 propId.descriptor 都有 [[Get]] 字段 或者 previous 和 propId.descriptor 都有 [[Set]] 字段。

如果以上步骤抛出一个语法错误，那么实现应该把这个错误视为早期语法错误。

产生式 PropertyAssignment : PropertyName : AssignmentExpression 按照下面的过程执行 :

1.  令 propName 为解释执行 PropertyName 的结果 .
2.  令 exprValue 为解释执行 AssignmentExpression 的结果 .
3.  令 propValue 为 GetValue(exprValue).
4.  令 desc 为属性描述对象 {[[Value]]: propValue, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}
5.  返回 Property Identifier (propName, desc).

产生式 PropertyAssignment : get PropertyName ( ) { FunctionBody } 按照下面的过程执行 :

1.  令 propName 为解释执行 PropertyName 的结果 .
2.  令 closure 为按照 13.2 规定，以空的参数列表和 body 代表的 FunctionBody 创建的一个新的函数对象 . 传入当前执行中的执行环境的 LexicalEnvironment 作为 Scope. 假如 PropertyAssignment 包含在严格模式代码中或者 FunctionBody 是严格模式代码，传入 true 为严格模式标志。
3.  令 desc 为属性描述对象 {[[Get]]: closure, [[Enumerable]]: true, [[Configurable]]: true}
4.  返回 Property Identifier (propName, desc).

产生式 PropertyAssignment : set PropertyName ( PropertySetParameterList ) { FunctionBody } is evaluated as follows:

1.  令 propName 为解释执行 PropertyName 的结果 .
2.  令 closure 为按照 13.2 规定，以 PropertySetParameterList 作为参数列表和 body 代表的 FunctionBody 创建的一个新的函数对象 . 传入当前执行中的执行环境的 LexicalEnvironment 作为 Scope. 假如 PropertyAssignment 包含在严格模式代码中或者 FunctionBody 是严格模式代码，传入 true 为严格模式标志。
3.  令 desc 为属性描述对象 {[[Set]]: closure, [[Enumerable]]: true, [[Configurable]]: true}
4.  返回属性标识符 (propName, desc).

假如 FunctionBody 是严格模式或者被包含在严格模式代码内，PropertyAssignment 中的 PropertySetParameterList，"eval" 或者 "arguments" 作为标识符将会是一个语法错误。

产生式 PropertyName : IdentifierName 按照下面的过程执行 :

1.  返回一个包含跟 IdentifierName. 完全相同的字符序列的字符串值

产生式 PropertyName : StringLiteral 按照下面的过程执行 :

1.  返回 the SV of the StringLiteral.

产生式 PropertyName : NumericLiteral 按照下面的过程执行 :

1.  令 nbr 为求 NumericLiteral 值的结果
2.  返回 ToString(nbr).

### 分组表达式

产生式 PrimaryExpression : ( Expression ) 按照下面的过程执行 :

1.  返回执行 Expression 的结果，它可能是 Reference 类型。

这一算法并不会作用 GetValue 于执行 Expression 的结果。这样做的原则是确保 delete 和 typeof 这样的运算符可以作用于括号括起来的表达式。

## 左值表达式

语法：

`MemberExpression : PrimaryExpression FunctionExpression MemberExpression [ Expression ] MemberExpression . IdentifierName new MemberExpression Arguments``NewExpression : MemberExpression new NewExpression``CallExpression : MemberExpression Arguments CallExpression Arguments CallExpression [ Expression ] CallExpression . IdentifierName``Arguments : ( ) ( ArgumentList )``ArgumentList : AssignmentExpression ArgumentList , AssignmentExpression``LeftHandSideExpression : NewExpression CallExpression`

### 属性访问

属性是通过 name 来访问的，可以使用点表示法访问

`MemberExpression . IdentifierName CallExpression . IdentifierName`

或者括号表示法访问

`MemberExpression [ Expression ] CallExpression [ Expression ]`

点表示法是根据以下的语法转换解释

`MemberExpression . IdentifierName`

这会等同于下面这个行为

`MemberExpression [ <identifier-name-string> ]`

类似地，

`CallExpression . IdentifierName`

是等同于下面的行为

`CallExpression [ <identifier-name-string> ]`

<identifier-name-string>是一个字符串字面量，它与 Unicode 编码后的 IdentifierName 包含相同的字符序列。</identifier-name-string>

产生式 MemberExpression : MemberExpression [ Expression ] is evaluated as follows:

1.  令 baseReference 为解释执行 MemberExpression 的结果 .
2.  令 baseValue 为 GetValue(baseReference).
3.  令 propertyNameReference 为解释执行 Expression 的结果 .
4.  令 propertyNameValue 为 GetValue(propertyNameReference).
5.  调用 CheckObjectCoercible(baseValue).
6.  令 propertyNameString 为 ToString(propertyNameValue).
7.  如果正在执行中的语法产生式包含在严格模式代码当中，令 strict 为 true, 否则令 strict 为 false.
8.  返回一个值类型的引用，其基值为 baseValue 且其引用名为 propertyNameString, 严格模式标记为 strict.

产生式 CallExpression : CallExpression [ Expression ] 以完全相同的方式执行，除了第 1 步执行的是其中的 CallExpression。

### new 运算符

产生式 NewExpression : new NewExpression 按照下面的过程执行 :

1.  令 ref 为解释执行 NewExpression 的结果 .
2.  令 constructor 为 GetValue(ref).
3.  如果 Type(constructor) is not Object ，抛出一个 TypeError 异常 .
4.  如果 constructor 没有实现 [[Construct]] 内置方法 ，抛出一个 TypeError 异常 .
5.  返回调用 constructor 的 [[Construct]] 内置方法的结果 , 传入按无参数传入参数列表 ( 就是一个空的参数列表 ).

产生式 MemberExpression : new MemberExpression Arguments 按照下面的过程执行 :

1.  令 ref 为解释执行 MemberExpression 的结果 .
2.  令 constructor 为 GetValue(ref).
3.  令 argList 为解释执行 Arguments 的结果 , 产生一个由参数值构成的内部列表类型 (11.2.4).
4.  如果 Type(constructor) is not Object ，抛出一个 TypeError 异常 .
5.  如果 constructor 没有实现 [[Construct]] 内置方法，抛出一个 TypeError 异常 .
6.  返回以 argList 为参数调用 constructor 的 [[Construct]] 内置方法的结果。

### 函数调用

产生式 CallExpression : MemberExpression Arguments 按照下面的过程执行 :

1.  令 ref 为解释执行 MemberExpression 的结果 .
2.  令 func 为 GetValue(ref).
3.  令 argList 为解释执行 Arguments 的结果 , 产生参数值们的内部列表 (see 11.2.4).
4.  如果 Type(func) is not Object ，抛出一个 TypeError 异常 .
5.  如果 IsCallable(func) is false ，抛出一个 TypeError 异常 .
6.  如果 Type(ref) 为 Reference，那么 如果 IsPropertyReference(ref) 为 true，那么 令 thisValue 为 GetBase(ref). 否则 , ref 的基值是一个环境记录项 令 thisValue 为调用 GetBase(ref) 的 ImplicitThisValue 具体方法的结果
7.  否则 , 假如 Type(ref) 不是 Reference. 令 thisValue 为 undefined.
8.  返回调用 func 的 [[Call]] 内置方法的结果 , 传入 thisValue 作为 this 值和列表 argList 作为参数列表

产生式 CallExpression : CallExpression Arguments 以完全相同的方式执行，除了第 1 步执行的是其中的 CallExpression。

假如 func 是一个原生的 ECMAScript 对象，返回的结果永远不会是 Reference 类型，调用一个宿主对象是否返回一个 Reference 类型的值由实现决定。 若一 Reference 值返回，则它必须是一个非严格的属性引用。

### 参数列表

The evaluation of an argument list produces a List of values (see 8.8).

产生式 Arguments : ( ) 按照下面的过程执行 :

1.  返回一个空列表 .

产生式 Arguments : ( ArgumentList ) 按照下面的过程执行 :

1.  返回解释执行 ArgumentList 的结果 .

产生式 ArgumentList ':' AssignmentExpression 按照下面的过程执行 :

1.  令 ref 为解释执行 AssignmentExpression 的结果 .
2.  令 arg 为 GetValue(ref).
3.  返回 a List whose sole item is arg.

产生式 ArgumentList :'''' ArgumentList , AssignmentExpression 按照下面的过程执行 :

1.  令 precedingArgs 为解释执行 ArgumentList 的结果 .
2.  令 ref 为解释执行 AssignmentExpression 的结果 .
3.  令 arg 为 GetValue(ref).
4.  返回一个列表，长度比 precedingArgs 大 1 且 它的 items 为 precedingArgs 的 items, 按顺序在后面跟 arg，arg 是这个新的列表的最后一个 item.

### 函数表达式

产生式 MemberExpression : FunctionExpression 按照下面的过程执行 :

1.  返回解释执行 FunctionExpression 的结果 .

## 后缀表达式

语法：

`PostfixExpression : LeftHandSideExpression LeftHandSideExpression [ 此处无换行 LineTerminator] ++ LeftHandSideExpression [ 此处无换行 LineTerminator] --`

### 后缀自增运算符

产生式 PostfixExpression : LeftHandSideExpression [ 此处无换行 LineTerminator] ++ 按照下面的过程执行 :

1.  令 lhs 为解释执行 LeftH 和 SideExpression 的结果 .
2.  假如以下所有条件都为 true，抛出一个 SyntaxError 异常 :
    *   Type(lhs) 为 Reference
    *   IsStrictReference(lhs) 为 true
    *   Type(GetBase(lhs)) 为环境记录项
    *   GetReferencedName(lhs) 为 "eval" 或 "arguments"

### 后缀自减运算符

产生式 PostfixExpression : LeftHandSideExpression [ 此处无换行 LineTerminator] -- 按照下面的过程执行 :

1.  令 lhs 为解释执行 LeftH 和 SideExpression 的结果 .
2.  假如以下所有条件都为 true，抛出一个 SyntaxError 异常 :
    *   Type(lhs) 为 Reference
    *   IsStrictReference(lhs) 为 true
    *   Type(GetBase(lhs)) 为环境记录项
    *   GetReferencedName(lhs) 为 "eval" 或 "arguments"

## 一元运算符

语法：

`UnaryExpression : PostfixExpression delete UnaryExpression void UnaryExpression typeof UnaryExpression ++ UnaryExpression -- UnaryExpression + UnaryExpression - UnaryExpression ~ UnaryExpression  ! UnaryExpression`

### delete 运算符

产生式 UnaryExpression : delete UnaryExpression 按照下面的过程执行 :

1.  令 ref 为解释执行 UnaryExpression 的结果。
2.  如果 Type(ref) 不是 Reference，返回 true。
3.  若 IsUnresolvableReference(ref) 则 , 如果 IsStrictReference(ref) 为 true ，抛出一个 SyntaxError 异常。 否则，返回 true。
4.  如果 IsPropertyReference(ref) 为 true 则： 返回以 GetReferencedName(ref) 和 IsStrictReference(ref) 做为参数调用 ToObject(GetBase(ref)) 的 [[Delete]] 内置方法的结果。
5.  否则 , ref 是到环境记录项绑定的 Reference，所以： 如果 IsStrictReference(ref) 为 true ，抛出一个 SyntaxError 异常 . 令 bindings 为 GetBase(ref). 返回以 GetReferencedName(ref) 为参数调用绑定的 DeleteBinding 具体方法的结果。

当 delete 运算符出现在 strict 模式代码中的时候，若 UnaryExpression 是到变量，函数形参或者函数名的直接引用则抛出一个 SyntaxError 异常。此外若 delete 运算符出现在严格模式代码中且要删除的属性具有特性{ [[Configurable]]: false }，抛出一个 TypeError 异常。

### void 运算符

产生式 UnaryExpression : void UnaryExpression 按照下面的过程执行 :

1.  令 expr 为解释执行 UnaryExpression 的结果 .
2.  调用 GetValue(expr).
3.  返回 undefined.

GetValue 一定要调用，即使它的值无用，但是可能会有可见的附加效果。

### typeof 运算符

产生式 UnaryExpression : typeof UnaryExpression 按照下面的过程执行 :

1.  令 val 为解释执行 UnaryExpression 的结果 .
2.  如果 Type(val) 为 Reference，则： 如果 IsUnresolvableReference(val) 为 true，返回 "undefined"。 令 val 为 GetValue(val).
3.  返回根据表 20 由 Type(val) 决定的字符串。

typeof 运算符结果

| val 类型 | 结果 |
| Undefined | "undefined" |
| Null | "null" |
| Boolean | "boolean" |
| Number | "number" |
| String | "string" |
| Object（原生，且没有实现 [[call]]） | "object" |
| Object（原生或者宿主且实现了 [[call]]） | "function" |
| Object（宿主且没实现 [[call]]） | 由实现定义，但不能是 "undefined", "boolean", "number", or "string" |

### 前自增运算符

产生式 UnaryExpression : ++ UnaryExpression 按照下面的过程执行 :

1.  令 expr 为解释执行 UnaryExpression 的结果 .
2.  抛出一个 SyntaxError 异常当以下条件全部为真 :
    *   Type(expr) 为 Reference
    *   IsStrictReference(expr) 为 true
    *   Type(GetBase(expr)) 为环境记录项
    *   GetReferencedName(expr) 是 "eval" 或 "arguments"

### 前自减运算符

产生式 UnaryExpression : -- UnaryExpression 按照下面的过程执行 :

1.  令 expr 为解释执行 UnaryExpression 的结果 .
2.  抛出一个 SyntaxError 异常，当以下条件全部为真 :
    *   Type(expr) 为 Reference
    *   IsStrictReference(expr) 为 true
    *   Type(GetBase(expr)) 为环境记录项
    *   GetReferencedName(expr) 是 "eval" 或 "arguments"

### 一元 + 运算符

一元+运算符将其操作数转换为 Number 类型。

产生式 UnaryExpression : + UnaryExpression 按照下面的过程执行 :

1.  令 expr 为解释执行 UnaryExpression 的结果 .
2.  返回 ToNumber(GetValue(expr)).

### 一元 - 运算符

一元+运算符将其操作数转换为 Number 类型并反转其正负。注意负的+0 产生-0，负的-0 产生+0。

产生式 UnaryExpression : - UnaryExpression 按照下面的过程执行 :

1.  令 expr 为解释执行 UnaryExpression 的结果 .
2.  令 oldValue 为 ToNumber(GetValue(expr)).
3.  如果 oldValue is NaN ，return NaN.
4.  返回 oldValue 取负（即，算出一个数字相同但是符号相反的值）的结果。

### 按位非运算符

产生式 UnaryExpression : ~ UnaryExpression 按照下面的过程执行 :

1.  令 expr 为解释执行 UnaryExpression 的结果 .
2.  令 oldValue 为 ToInt32(GetValue(expr)).
3.  返回 oldValue 按位取反的结果。结果为 32 位有符号整数。

### 逻辑非运算符

产生式 UnaryExpression : ! UnaryExpression 按照下面的过程执行 :

1.  令 expr 为解释执行 UnaryExpression 的结果 .
2.  令 oldValue 为 ToBoolean(GetValue(expr)).
3.  如果 oldValue 为 true ，返回 false.
4.  返回 true.

## 乘法运算符

语法：

`MultiplicativeExpression : UnaryExpression MultiplicativeExpression * UnaryExpression MultiplicativeExpression / UnaryExpression MultiplicativeExpression % UnaryExpression`

语义：

产生式 MultiplicativeExpression : 'MultiplicativeExpression'@ 'UnaryExpression, 其中 @ 表示上面定义中的运算符之一，按照下面的过程执行 :

1.  令 left 为解释执行 MultiplicativeExpression 的结果 .
2.  令 leftValue 为 GetValue(left).
3.  令 right 为解释执行 UnaryExpression 的结果 .
4.  令 rightValue 为 GetValue(right).
5.  令 leftNum 为 ToNumber(leftValue).
6.  令 rightNum 为 ToNumber(rightValue).
7.  返回将特定运算符 (*, /, or %) 作用于 leftNum 和 rightNum 的结果。参见 11.5.1, 11.5.2, 11.5.3 后的注解。

### 使用 * 运算符

*运算符表示乘法，产生操作数的乘积。乘法运算满足交换律。因为精度问题，乘法不总是满足结合律。

浮点数的乘法遵循 IEEE 754 二进制双精度幅度浮点算法规则：

*   若两个操作数之一为 NaN，结果为 NaN。
*   假如两个操作数的正负号相同，结果就是正的，如果不同就是负的。
*   无穷大被零乘结果是 NaN。
*   无穷大被无穷大乘结果就是无穷大。符号按照前面说过的规则决定。
*   无穷大被有穷的非零值乘结果是带正负号的无穷大。符号仍然按照前面说过的规则决定。
*   其它情况下，既没有无穷大也没有 NaN 参与运算，结果计算出来后会按照 IEEE 754 round-to-nearest 模式取到最接近的能表示的数。如果值过大不能表示，则结果为相应的正负无穷大。如果值过小不能表示，则结果为相应的正负零。ECMAScript 要求支持 IEEE 754 规定的渐进下溢。

### 使用 / 运算符

/运算符表示除法，产生操作数的商。左操作数是被除数，右操作数是除数。ECMAScript 不支持整数除法。所有除法运算的操作数和结果都是双精度浮点数。浮点数的除法遵循 IEEE 754 二进制双精度幅度浮点算法规则：

*   若两个操作数之一为 NaN，结果为 NaN。
*   假如两个操作数的正负号相同，结果就是正的，如果不同就是负的。
*   无穷大被零乘结果是 NaN。
*   无穷大被无穷大除结果是 NaN。
*   无穷大被零除结果是无穷大。符号按照前面说过的规则决定。
*   无穷大被非零有穷的值除结果是有正负号的无穷大。符号按照前面说过的规则决定。
*   有穷的非零值被无穷大除结果是零。符号按照前面说过的规则决定。
*   零被零除结果是 NaN；零被其它有穷数除结果是零，符号按照前面说过的规则决定。
*   有穷的非零值被零除结果是有正负号的无穷大。符号按照前面说过的规则决定。
*   其它情况下，既没有无穷大也没有 NaN 参与运算，结果计算出来后会按照 IEEE 754 round-to-nearest 模式取到最接近的能表示的数。如果值过大不能表示，则结果为相应的正负无穷大。如果值过小不能表示，则结果为相应的正负零。ECMAScript 要求支持 IEEE 754 规定的渐进下溢。

### 使用 % 运算符

%运算符产生其运算符在除法中的余数。左操作数是被除数，右操作数是除数。

在 C 和 C++中，余数运算符只接受整数为操作数；在 ECMAScript，它还接受浮点操作数。

浮点数使用%运算符的余数运算与 IEEE 754 所定义的"remainder"运算不完全相同。IEEE 754 “remainder”运算做邻近取整除法的余数计算，而不是舍尾除法，这样它的行为跟通常意义上的整数余数运算符行为不一致。而 ECMAScript 语言定义浮点操作%为与 Java 取余运算符一致；可以参照 C 库中的函数 fmod。

ECMAScript 浮点数的取余法遵循 IEEE 754 二进制双精度幅度浮点算法规则：

*   若两个操作数之一为 NaN，结果为 NaN。
*   结果的符号等于被除数。
*   若被除数是无穷大或者除数是零，或者两者皆是，结果就是 NaN。
*   若被除数有穷而除数为无穷大，结果为被除数。
*   若被除数为零且除数非零且有穷，结果与被除数相同。
*   其它情况下，既没有 0，无穷大也没有 NaN 参与运算，从被除数 n 和除数 d 得到浮点数余数 r 以数学关系式 r = n ? (d × q) 定义，其中 q 是个整数，在 n/d 为负时为负，在 n/d 为正时为正，它应该在不超过 n 和 d 的商的前提下尽可能大。结果计算出来后会按照 IEEE 754 round-to-nearest 模式取到最接近的能表示的数。

## 加法运算符

语法：

`AdditiveExpression : MultiplicativeExpression AdditiveExpression + MultiplicativeExpression AdditiveExpression - MultiplicativeExpression`

### 加号运算符 ( + )

The addition operator either performs string concatenation or numeric addition.

产生式 AdditiveExpression : AdditiveExpression + MultiplicativeExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 AdditiveExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  令 rref 为解释执行 MultiplicativeExpression 的结果 .
4.  令 rval 为 GetValue(rref).
5.  令 lprim 为 ToPrimitive(lval).
6.  令 rprim 为 ToPrimitive(rval).
7.  如果 Type(lprim) 为 String 或者 Type(rprim) 为 String，则： 返回由 ToString(lprim) 和 ToString(rprim) 连接而成的字符串
8.  返回将加法运算作用于 ToNumber(lprim) 和 ToNumber(rprim) 的结果。参见 11.6.3 后的注解。

在步骤 5 和 6 中的 ToPrimitive 调用没有提供 hint，除了 Date 对象之外所有 ECMAScript 对象将缺少 hint 的情况当做 Number 处理；Date 对象将缺少 hint 的情况当做 hint 为字符串。宿主对象可能将缺少 hint 的情况当做别的处理。

步骤 7 与关系运算符比较算法中的步骤 3 不同，它使用逻辑或运算符而不是逻辑与运算符

### 减号运算符 ( - )

产生式 AdditiveExpression : AdditiveExpression - MultiplicativeExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 AdditiveExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  令 rref 为解释执行 MultiplicativeExpression 的结果 .
4.  令 rval 为 GetValue(rref).
5.  令 lnum 为 ToNumber(lval).
6.  令 rnum 为 ToNumber(rval).
7.  返回返回将减法运算作用于 ToNumber(lprim) 和 ToNumber(rprim) 的结果。参见 11.6.3 后的注解。

### 加法作用于数字

+运算符作用于两个数字类型的操作数时表示加法，产生两个操作数之和。-运算符表示剑法，产生两个数字之差。

加法是满足交换律的运算，但是不总满足结合律。

加法遵循 IEEE 754 二进制双精度幅度浮点算法规则：

*   两个正负号相反的无穷之和为 NaN。
*   两个正负号相同的无穷大之和是具有相同正负的无穷大。
*   无穷大和有穷值之和等于操作数中的无穷大。
*   两个负零之和为-0。
*   两个正零，或者两个正负号相反的零之和为+0。
*   零与非零有穷值之和等于非零的那个操作数。
*   两个大小相等，符号相反的非零有穷值之和为+0。
*   其它情况下，既没有无穷大也没有 NaN 或者零参与运算，并且操作数要么大小不等，要么符号相同，结果计算出来后会按照 IEEE 754 round-to-nearest 模式取到最接近的能表示的数。如果值过大不能表示，则结果为相应的正负无穷大。如果值过小不能表示，则结果为相应的正负零。ECMAScript 要求支持 IEEE 754 规定的渐进下溢。

-运算符作用于两个数字类型时表示减法，产生两个操作数之差。左边操作数是被减数右边是减数。给定操作数 a 和 b，总是有 a–b 产生与 a + ( -b )产生相同结果。

## 位运算移位运算符

语法：

`ShiftExpression : AdditiveExpression ShiftExpression << AdditiveExpression ShiftExpression >> AdditiveExpression ShiftExpression >>> AdditiveExpression`

### 左移运算符

表示对左操作数做右操作数指定次数的按位左移操作。

产生式 ShiftExpression : ShiftExpression << AdditiveExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 ShiftExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  令 rref 为解释执行 AdditiveExpression 的结果 .
4.  令 rval 为 GetValue(rref).
5.  令 lnum 为 ToInt32(lval).
6.  令 rnum 为 ToUint32(rval).
7.  令 shiftCount 为用掩码算出 rnum 的最后五个比特位 , 即计算 rnum & 0x1F 的结果。
8.  返回 lnum 左移 shiftCount 比特位的结果。结果是一个有符号 32 位整数。

### 带符号右移运算符

filling bitwise right shift operation on the left operand by the amount specified by the right operand.

产生式 ShiftExpression : ShiftExpression >> AdditiveExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 ShiftExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  令 rref 为解释执行 AdditiveExpression 的结果 .
4.  令 rval 为 GetValue(rref).
5.  令 lnum 为 ToInt32(lval).
6.  令 rnum 为 ToUint32(rval).
7.  令 shiftCount 为用掩码算出 rnum 的最后五个比特位 , 即计算 rnum & 0x1F 的结果。
8.  返回 lnum 带符号扩展的右 移 shiftCount 比特位的结果 . The most significant bit is propagated. 结果是一个有符号 32 位整数。

### 无符号右移运算符

Performs a zero-filling bitwise right shift operation on the left operand by the amount specified by the right operand.

产生式 ShiftExpression : ShiftExpression >>> AdditiveExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 ShiftExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  令 rref 为解释执行 AdditiveExpression 的结果 .
4.  令 rval 为 GetValue(rref).
5.  令 lnum 为 ToUint32(lval).
6.  令 rnum 为 ToUint32(rval).
7.  令 shiftCount 为用掩码算出 rnum 的最后五个比特位 , 即计算 rnum & 0x1F 的结果。
8.  返回 lnum 做 0 填充右移 shiftCount 比特位的结果 . 缺少的比特位填 0。 结果是一个无符号 32 位整数 .

## 比较运算符

语法：

`RelationalExpression : ShiftExpression RelationalExpression < ShiftExpression RelationalExpression > ShiftExpression RelationalExpression <= ShiftExpression RelationalExpression >= ShiftExpression RelationalExpression instanceof ShiftExpression RelationalExpression in ShiftExpression``RelationalExpressionNoIn : ShiftExpression RelationalExpressionNoIn < ShiftExpression RelationalExpressionNoIn > ShiftExpression RelationalExpressionNoIn <= ShiftExpression RelationalExpressionNoIn >= ShiftExpression RelationalExpressionNoIn instanceof ShiftExpression`

“NoIn”变体用以避免混淆关系表达式中的 in 运算符和 for 语句中的 in 运算符。

语义：

执行关系比较运算符的结果总是 Boolean 类型。表示是否由运算符指定的关系对两操作数成立。

RelationalExpressionNoIn 跟 RelationalExpression 完全按相同的方式执行，出了 RelationalExpressionNoIn 要代替 RelationalExpression 被执行。

### The Less-than Operator ( < )

产生式 RelationalExpression : RelationalExpression < ShiftExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 RelationalExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  令 rref 为解释执行 ShiftExpression 的结果 .
4.  令 rval 为 GetValue(rref).
5.  令 r 为抽象关系比较算法 lval < rval( 参见 11.8.5) 的结果
6.  如果 r 为 undefined，返回 false. 否则 , 返回 r.

### The Greater-than Operator ( > )

产生式 RelationalExpression : RelationalExpression > ShiftExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 RelationalExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  令 rref 为解释执行 ShiftExpression 的结果 .
4.  令 rval 为 GetValue(rref).
5.  令 r 为为抽象关系比较算法 lval < rval( 参见 11.8.5) 的结果，参数 LeftFirst 设为 false
6.  如果 r 为 undefined，返回 false. 否则 , 返回 r.

### The Less-than-or-equal Operator ( <= )

产生式 RelationalExpression : RelationalExpression <= ShiftExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 RelationalExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  令 rref 为解释执行 ShiftExpression 的结果 .
4.  令 rval 为 GetValue(rref).
5.  令 r 为为抽象关系比较算法 rval < lval( 参见 11.8.5) 的结果，参数 LeftFirst 设为 false
6.  如果 r 为 true 或者 undefined ，返回 false. 否则 , 返回 true.

### The Greater-than-or-equal Operator ( >= )

产生式 RelationalExpression : RelationalExpression >= ShiftExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 RelationalExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  令 rref 为解释执行 ShiftExpression 的结果 .
4.  令 rval 为 GetValue(rref).
5.  令 r 为抽象关系比较算法 lval < rval( 参见 11.8.5) 的结果
6.  如果 r 为 true 或者 undefined ，返回 false. 否则 , 返回 true.

### 抽象关系比较算法

以 x 和 y 为值进行小于比较（x<y 的比较），会产生的结果可为="" true，false 或 undefined（这说明 x、y 中最少有一个操作数是 NaN）。除了 x 和 y，这个算法另外需要一个名为 LeftFirst 的布尔值标记作为参数。这个标记用于解析顺序的控制，因为操作数 x 和 y 在执行的时候会有潜在可见的副作用。LeftFirst 标志是必须的，因为 ECMAScript 规定了表达式是从左到右顺序执行的。LeftFirst 的默认值是 true，这表明在相关的表达式中，参数 x 出现在参数 y 之前。如果 LeftFirst 值是 false，情况会相反，操作数的执行必须是先 y 后 x。这样的一个小于比较的执行步骤如下：

1.  如果 LeftFirst 标志是 true，那么
    1.  让 px 为调用 ToPrimitive(x, hint Number) 的结果。
    2.  让 py 为调用 ToPrimitive(y, hint Number) 的结果。
2.  否则解释执行的顺序需要反转，从而保证从左到右的执行顺序
    1.  让 py 为调用 ToPrimitive(y, hint Number) 的结果。
    2.  让 px 为调用 ToPrimitive(x, hint Number) 的结果。
3.  如果 Type(px) 和 Type(py) 得到的结果不都是 String 类型，那么
    1.  让 nx 为调用 ToNumber(px) 的结果。因为 px 和 py 都已经是基本数据类型（primitive values 也作原始值），其执行顺序并不重要。
    2.  让 ny 为调用 ToNumber(py) 的结果。
    3.  如果 nx 是 NaN，返回 undefined
    4.  如果 ny 是 NaN，返回 undefined
    5.  如果 nx 和 ny 的数字值相同，返回 false
    6.  如果 nx 是 +0 且 ny 是 -0，返回 flase
    7.  如果 nx 是 -0 且 ny 是 +0，返回 false
    8.  如果 nx 是 +∞，返回 fasle
    9.  如果 ny 是 +∞，返回 true
    10.  如果 ny 是 -∞，返回 flase
    11.  如果 nx 是 -∞，返回 true
    12.  如果 nx 数学上的值小于 ny 数学上的值（注意这些数学值都不能是无限的且不能都为 0），返回 ture。否则返回 false。
4.  否则，px 和 py 都是 Strings 类型
    1.  如果 py 是 px 的一个前缀，返回 false。（当字符串 q 的值可以是字符串 p 和一个其他的字符串 r 拼接而成时，字符串 p 就是 q 的前缀。注意：任何字符串都是自己的前缀，因为 r 可能是空字符串。）
    2.  如果 px 是 py 的前缀，返回 true。
    3.  让 k 成为最小的非负整数，能使得在 px 字符串中位置 k 的字符与字符串 py 字符串中位置 k 的字符不相同。（这里必须有一个 k，使得互相都不是对方的前缀）
    4.  让 m 成为字符串 px 中位置 k 的字符的编码单元值。
    5.  让 n 成为字符串 py 中位置 k 的字符的编码单元值。
    6.  如果 n<m，返回 true。否则，返回 false。

使用或代替的时候要注意，这里的步骤 3 和加号操作符 + 算法 (11.6.1) 的步骤 7 的区别。

String 类型的比较使用了其编码单元值的作为一个简单的词法表序列去比较。这里不打算使用更复杂的、语义化的字符或字符串序列，和 Unicode 规范的整理序列进行比较。因此，字符串的值和其对应的 Unicode 标准的值是不相同的。实际上，这个算法假定了所有字符串已经是正常化的格式。同时要注意，对于字符串拼接追加的字符的时候，UTF-16 编码单元值的词法表序列是不同于代码点值的序列的。

### The instanceof operator

产生式 RelationalExpression: RelationalExpression instanceof ShiftExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 RelationalExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  令 rref 为解释执行 ShiftExpression 的结果 .
4.  令 rval 为 GetValue(rref).
5.  如果 Type(rval) 不是 Object，抛出一个 TypeError 异常 .
6.  如果 rval 没有 [[HasInstance]] 内置方法，抛出一个 TypeError 异常 .
7.  返回以参数 lval. 调用 rval 的 [[HasInstance]] 内置方法的结果

### The in operator

产生式 RelationalExpression : RelationalExpression in ShiftExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 RelationalExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  令 rref 为解释执行 ShiftExpression 的结果 .
4.  令 rval 为 GetValue(rref).
5.  如果 Type(rval) 不是 Object ，抛出一个 TypeError 异常 .
6.  返回以参数 ToString(lval). 调用 rval 的 [[HasProperty]] 内置方法的结果

## 等值运算符

语法：

`EqualityExpression : RelationalExpression EqualityExpression == RelationalExpression EqualityExpression != RelationalExpression EqualityExpression === RelationalExpression EqualityExpression !== RelationalExpression``EqualityExpressionNoIn : RelationalExpressionNoIn EqualityExpressionNoIn == RelationalExpressionNoIn EqualityExpressionNoIn != RelationalExpressionNoIn EqualityExpressionNoIn === RelationalExpressionNoIn EqualityExpressionNoIn !== RelationalExpressionNoIn`

语义：

执行相等比较运算符的结果总是 Boolean 类型。表示是否由运算符指定的关系对两操作数成立。

EqualityExpressionNoIn 跟 EqualityExpression 完全按相同的方式执行，出了 RelationalExpressionNoIn 要代替 RelationalExpression 被执行。

### The Equals Operator ( == )

产生式 EqualityExpression : EqualityExpression == RelationalExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 EqualityExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  令 rref 为解释执行 RelationalExpression 的结果 .
4.  令 rval 为 GetValue(rref).
5.  返回做用相等比较算法于 rval == lval( 参见 11.9.3) 的结果

### The Does-not-equals Operator ( != )

产生式 EqualityExpression : EqualityExpression != RelationalExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 EqualityExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  令 rref 为解释执行 RelationalExpression 的结果 .
4.  令 rval 为 GetValue(rref).
5.  令 r 为做用相等比较算法于 rval == lval( 参见 11.9.3) 的结果
6.  如果 r 为 true，返回 false. 否则 , 返回 true.

### 抽象相等比较算法

比较运算`x`==`y`, 其中`x`和 `y`是值，产生 true 或者 false。这样的比较按如下方式进行：

1.  若 Type(x)与 Type(y)相同， 则
    1.  若 Type(x)为 Undefined， 返回 true。
    2.  若 Type(x)为 Null， 返回 true。
    3.  若 Type(x)为 Number， 则
        1.  若 x 为 NaN， 返回 false。
        2.  若 y 为 NaN， 返回 false。
        3.  若 x 与 y 为相等数值， 返回 true。
        4.  若 x 为 +0 且 y 为?0， 返回 true。
        5.  若 x 为 ?0 且 y 为+0， 返回 true。
        6.  返回 false。
    4.  若 Type(x)为 String, 则当 x 和 y 为完全相同的字符序列（长度相等且相同字符在相同位置）时返回 true。 否则， 返回 false。
    5.  若 Type(x)为 Boolean, 当 x 和 y 为同为 true 或者同为 false 时返回 true。 否则， 返回 false。
    6.  当 x 和 y 为引用同一对象时返回 true。否则，返回 false。
2.  若 x 为 null 且 y 为 undefined， 返回 true。
3.  若 x 为 undefined 且 y 为 null， 返回 true。
4.  若 Type(x) 为 Number 且 Type(y)为 String， 返回 comparison x == ToNumber(y)的结果。
5.  若 Type(x) 为 String 且 Type(y)为 Number，
6.  返回比较 ToNumber(x) == y 的结果。
7.  若 Type(x)为 Boolean， 返回比较 ToNumber(x) == y 的结果。
8.  若 Type(y)为 Boolean， 返回比较 x == ToNumber(y)的结果。
9.  若 Type(x)为 String 或 Number，且 Type(y)为 Object，返回比较 x == ToPrimitive(y)的结果。
10.  若 Type(x)为 Object 且 Type(y)为 String 或 Number， 返回比较 ToPrimitive(x) == y 的结果。
11.  返回 false。

按以上相等之定义：

*   字符串比较可以按这种方式强制执行: `"" + a == "" + b` 。
*   数值比较可以按这种方式强制执行: `+a == +b` 。
*   布尔值比较可以按这种方式强制执行: `!a == !b` 。

等值比较操作保证以下不变：

*   `A != B` 等价于 `!(A==B)` 。
*   `A == B` 等价于 `B == A` ，除了 A 与 B 的执行顺序。

相等运算符不总是传递的。例如，两个不同的 String 对象，都表示相同的字符串值； `==` 运算符认为每个 String 对象都与字符串值相等，但是两个字符串对象互不相等。例如：

*   `new String("a") == "a"` 和 `"a" == new String("a")` 皆为 true。
*   `new String("a")` `==` `new String("a")` 为 false。

字符串比较使用的方式是简单地检测字符编码单元序列是否相同。不会做更复杂的、基于语义的字符或者字符串相等的定义以及 Unicode 规范中定义的 collating order。所以 Unicode 标准中认为相等的 String 值可能被检测为不等。实际上这一算法认为两个字符串已经是经过规范化的形式。

### 严格等于运算符 ( === )

产生式 EqualityExpression : EqualityExpression === RelationalExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 EqualityExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  令 rref 为解释执行 RelationalExpression 的结果 .
4.  令 rval 为 GetValue(rref).
5.  返回做用严格相等比较算法于 rval === lval( 参见 11.9.6) 的结果

### The Strict Does-not-equal Operator ( !== )

产生式 EqualityExpression : EqualityExpression !== RelationalExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 EqualityExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  令 rref 为解释执行 RelationalExpression 的结果 .
4.  令 rval 为 GetValue(rref).
5.  令 r 为做用严格相等比较算法于 rval === lval( 参见 11.9.6) 的结果
6.  如果 r 为 true，返回 false. 否则 , 返回 true.

### 严格等于比较算法

比较 `x`===`y`，`x` 和 `y` 为值，需要产出 true 或 false。比较过程如下：

1.  如果 `Type(`x`)` 与 `Type(`y`)` 的结果不一致，返回 false，否则
2.  如果 `Type(`x`)` 结果为 Undefined，返回 true
3.  如果 `Type(`x`)` 结果为 Null，返回 true
4.  如果 `Type(`x`)` 结果为 Number，则
    1.  如果 `x` 为 NaN，返回 false
    2.  如果 `y` 为 NaN，返回 false
    3.  如果 `x` 与 `y` 为同一个数字，返回 true
    4.  如果 `x` 为 +0，`y` 为 -0，返回 true
    5.  如果 `x` 为 -0，`y` 为 +0，返回 true
    6.  返回 false
5.  如果 `Type(`x`)` 结果为 String，如果 `x` 与 `y` 为完全相同的字符序列（相同的长度和相同的字符对应相同的位置），返回 true，否则，返回 false
6.  如果 `Type(`x`)` 结果为 Boolean，如果 `x` 与 `y` 都为 true 或 false，则返回 true，否则，返回 false
7.  如果 `x` 和 `y` 引用到同一个 Object 对象，返回 true，否则，返回 false

此算法与 `SameValue` 算法在对待有符号的零和 NaN 上表现不同。

## 二进制位运算符

语法：

`BitwiseANDExpression : EqualityExpression BitwiseANDExpression & EqualityExpression``BitwiseANDExpressionNoIn : EqualityExpressionNoIn BitwiseANDExpressionNoIn & EqualityExpressionNoIn``BitwiseXORExpression : BitwiseANDExpression BitwiseXORExpression ^ BitwiseANDExpression``BitwiseXORExpressionNoIn : BitwiseANDExpressionNoIn BitwiseXORExpressionNoIn ^ BitwiseANDExpressionNoIn``BitwiseORExpression : BitwiseXORExpression BitwiseORExpression | BitwiseXORExpression``BitwiseORExpressionNoIn : BitwiseXORExpressionNoIn BitwiseORExpressionNoIn | BitwiseXORExpressionNoIn`

语义：

产生式 A : A' @ 'B, 其中 @ 是上述产生式中的位运算符之一，按照下面的过程执行 :

1.  令 lref 为解释执行 A 的结果 .
2.  令 lval 为 GetValue(lref).
3.  令 rref 为解释执行 B 的结果 .
4.  令 rval 为 GetValue(rref).
5.  令 lnum 为 ToInt32(lval).
6.  令 rnum 为 ToInt32(rval).
7.  返回作用位运算符 @ 到 lnum 和 rnum. 结果是 32 位有符号整数。

## 二元逻辑运算符

语法

`LogicalANDExpression : BitwiseORExpression LogicalANDExpression && BitwiseORExpression``LogicalANDExpressionNoIn : BitwiseORExpressionNoIn LogicalANDExpressionNoIn && BitwiseORExpressionNoIn``LogicalORExpression : LogicalANDExpression LogicalORExpression || LogicalANDExpression``LogicalORExpressionNoIn : LogicalANDExpressionNoIn LogicalORExpressionNoIn || LogicalANDExpressionNoIn`

语义

产生式 LogicalANDExpression : LogicalANDExpression && BitwiseORExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 LogicalANDExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  如果 ToBoolean(lval) 为 false ，返回 lval.
4.  令 rref 为解释执行 BitwiseORExpression 的结果 .
5.  返回 GetValue(rref).

产生式 LogicalORExpression : LogicalORExpression || LogicalANDExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 LogicalORExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  如果 ToBoolean(lval) 为 true ，返回 lval.
4.  令 rref 为解释执行 LogicalANDExpression 的结果 .
5.  返回 GetValue(rref).

LogicalANDExpressionNoIn 和 LogicalORExpressionNoIn 执行完全按照 LogicalANDExpression 和 LogicalORExpression 相同的方式，BitwiseORExpressionNoIn 和 LogicalORExpressionNoIn 替代了 BitwiseORExpression 和 LogicalORExpression 除外。

由&& 或者||运算符产生的值不是必须为 Boolean 类型，产生的值始终为两个运算表达式的结果之一。

## 条件运算符

语法：

`ConditionalExpression : LogicalORExpression LogicalORExpression ? AssignmentExpression : AssignmentExpression``ConditionalExpressionNoIn : LogicalORExpressionNoIn LogicalORExpressionNoIn ? AssignmentExpressionNoIn : AssignmentExpressionNoIn`

语义

产生式 ConditionalExpression : LogicalORExpression ? AssignmentExpression : AssignmentExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 LogicalORExpression 的结果 .
2.  如果 ToBoolean(GetValue(lref)) 为 true ，那么： 令 trueRef 为解释执行第一个 AssignmentExpression 的结果 . 返回 GetValue(trueRef).
3.  Else 令 falseRef 为解释执行第二个 AssignmentExpression 的结果 . 返回 GetValue(falseRef).

ConditionalExpressionNoIn 执行完全按照 ConditionalExpression 相同的方式，除了 AssignmentExpression 和 AssignmentExpressionNoIn 替代了第一个 AssignmentExpression 和第二个 AssignmentExpression。

ECMAScript 中的 ConditionalExpression 跟 C 和 Java 有一点点不同，它允许第二个子表达式是个 Expression 但是限制第三个表达式必须是 ConditionalExpression。ECMAScript 中这个差别的依据是可以允许允许赋值表达式出现在条件的任意一侧同时避免逗号表达式作为中间的表达式时无用且易混淆的使用方式。

## 赋值运算符

语法

`AssignmentExpression : ConditionalExpression LeftHandSideExpression = AssignmentExpression LeftHandSideExpression AssignmentOperator AssignmentExpression``AssignmentExpressionNoIn : ConditionalExpressionNoIn LeftHandSideExpression = AssignmentExpressionNoIn LeftHandSideExpression AssignmentOperator AssignmentExpressionNoIn` `AssignmentOperator : one of *= /= %= += -= <<= >>= >>>= &= ^= |=`

语义

AssignmentExpressionNoIn 执行完全按照 AssignmentExpression 相同的方式，除了 ConditionalExpressionNoIn 替代了 ConditionalExpression。

### 简单赋值

产生式 AssignmentExpression : LeftHandSideExpression = AssignmentExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 LeftH 和 SideExpression 的结果 .
2.  令 rref 为解释执行 AssignmentExpression 的结果 .
3.  令 rval 为 GetValue(rref).
4.  抛出一个 SyntaxError 异常，当以下条件都成立 :
    *   Type(lref) 为 Reference
    *   IsStrictReference(lref) 为 true
    *   Type(GetBase(lref)) 为环境记录项
    *   GetReferencedName(lref) 为 "eval" 或 "arguments"
5.  调用 PutValue(lref, rval).
6.  返回 rval.

当赋值发生在严格模式中时其 LeftHandSide 执行结果不能是无法解引用的引用类型。假如这样它会在赋值时抛出 ReferenceError 异常。 LeftHandSide 也不能是到具有{[[Writable]]:false}特性的数据属性的引用，到具有{[[Set]]:undefined}特性的访问器属性的引用，或者 [[Extensible]]内部属性值为 false 的对象上不存在的属性。这些情况都会抛出 TypeError 异常。

### 组合赋值

产生式 AssignmentExpression : LeftHandSideExpression'@ = AssignmentExpression, where @ represents one of the operators indicated above, 按照下面的过程执行 :

1.  令 lref 为解释执行 LeftH 和 SideExpression 的结果 .
2.  令 lval 为 GetValue(lref).
3.  令 rref 为解释执行 AssignmentExpression 的结果 .
4.  令 rval 为 GetValue(rref).
5.  令 r 为作用运算符 @ 于 lval 和 rval 的结果。
6.  抛出一个 SyntaxError 异常，当以下条件全部成立：
    *   Type(lref) 为 Reference
    *   IsStrictReference(lref) 为 true
    *   Type(GetBase(lref)) 为环境记录项
    *   GetReferencedName(lref) 为 "eval" 或 "arguments"
7.  调用 PutValue(lref, r).
8.  返回 r.

见 注 11.13.1。

## 逗号运算符

语法：

`Expression : AssignmentExpression Expression , AssignmentExpression``ExpressionNoIn : AssignmentExpressionNoIn ExpressionNoIn , AssignmentExpressionNoIn`

语义：

产生式 Expression : Expression , AssignmentExpression 按照下面的过程执行 :

1.  令 lref 为解释执行 Expression 的结果 .
2.  Call GetValue(lref).
3.  令 rref 为解释执行 AssignmentExpression 的结果 .
4.  返回 GetValue(rref).

ExpressionNoIn 执行完全按照 Expression 相同的方式，除了 AssignmentExpressionNoIn 替代了 AssignmentExpression。

GetValue 必须调用，即使它的值没有用，因为它可能有附加效果。