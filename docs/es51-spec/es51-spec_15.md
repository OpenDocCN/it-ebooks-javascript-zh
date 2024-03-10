# 函数定义

语法

`FunctionDeclaration : function Identifier ( FormalParameterListopt ) { FunctionBody }` `FunctionExpression : function Identifieropt ( FormalParameterListopt ) { FunctionBody }` `FormalParameterList : Identifier FormalParameterList , Identifier` `FunctionBody : SourceElementsopt`

语义

产生式 FunctionDeclaration : function Identifier ( FormalParameterList[opt] ) { FunctionBody } 依照定义绑定初始化 (10.5) 如下初始化：

1.  依照 13.2，指定 FormalParameterList[opt] 为参数，指定 FunctionBody 为 body，创建一个新函数对象，返回结果。运行中的执行环境的 VariableEnvironment 传递为 Scope。如果 FunctionDeclaration 包含在 严格模式代码 里或 FunctionBody 是 严格模式代码 ，那么传递 true 为 Strict 标志。

产生式 FunctionExpression : function ( FormalParameterList[opt] ) { FunctionBody } 的解释执行如下：

1.  依照 13.2，指定 FormalParameterList[opt] 为参数，指定 FunctionBody 为 body，创建一个新函数对象，返回结果。运行中的执行环境的 LexicalEnvironment 传递为 Scope。如果 FunctionExpression 包含在 严格模式代码 里或 FunctionBody 是 严格模式代码 ，那么传递 true 为 Strict 标志。

产生式 FunctionExpression : function Identifier[opt] ( FormalParameterList[opt] ) { FunctionBody } 的解释执行如下：

1.  令 funcEnv 为以运行中执行环境的 Lexical Environment 为参数调用 NewDeclarativeEnvironment 的结果。
2.  令 envRec 为 funcEnv 的环境记录项。
3.  以 Identifier 的字符串值为参数调用 envRec 的具体方法 CreateImmutableBinding(N)。
4.  令 closure 为依照 13.2，指定 FormalParameterList[opt] 为参数，指定 FunctionBody 为 body，创建一个新函数对象的结果。传递 funcEnv 为 Scope。如果 FunctionExpression 包含在严格模式代码 里或 FunctionBody 是 严格模式代码 ，那么传递 true 为 Strict 标志。
5.  以 Identifier 的字符串值和 closure 为参数调用 envRec 的具体方法 InitializeImmutableBinding(N,V)。
6.  返回 closure。

可以从 FunctionExpression 的 FunctionBody 里面引用 FunctionExpression 的 Identifier，以允许函数递归调用自身。然而不像 FunctionDeclaration，FunctionExpression 的 Identifier 不能被范围封闭的 FunctionExpression 引用，也不会影响它。

产生式 FunctionBody : SourceElements[opt] 的解释执行如下：

1.  如果这个 FunctionBody 所在 FunctionDeclaration 或 FunctionExpression 包含在严格模式代码内，或其 SourceElements 的指令序言 (14.1) 包含一个 use strict 指令，或满足 10.1 的任何条件，那么其代码是严格模式代码。如果 FunctionBody 的代码是严格模式代码，SourceElements 的解释执行为以下的严格模式代码步骤。否则，SourceElements 的解释执行为以下的非严格模式步骤。
2.  如果 SourceElements 是当前的，则返回 SourceElements 的解释执行结果。
3.  否则返回 (normal, undefined, empty)。

## 严格的模式的限制

如果严格模式 FunctionDeclaration 或 FunctionExpression 的 FormalParameterList 里出现多个相同 Identifier 值，那么这是个 SyntaxError。

如果严格模式 FunctionDeclaration 或 FunctionExpression 的 FormalParameterList 里出现标识符 "eval" 或标识符 "arguments"，那么这是个 SyntaxError。

如果严格模式 FunctionDeclaration 或 FunctionExpression 的 Identifier 是标识符 "eval" 或标识符 "arguments"，那么这是个 SyntaxError。

## 创建函数对象

指定 FormalParameterList 为可选参数列表，指定 FunctionBody 为函数体，指定 Scope 为 词法环境 ，Strict 为布尔标记，按照如下步骤构建函数对象：

1.  创建一个新的 ECMAScript 原生对象，令 F 为此对象。
2.  依照 8.12 描述设定 F 的除 [[Get]] 以外的所有内部方法。
3.  设定 F 的 [[Class]] 内部属性为 "Function"。
4.  设定 F 的 [[Prototype]] 内部属性为 15.3.3.1 指定的标准内置 Function 对象的 prototype 属性。
5.  依照 15.3.5.4 描述，设定 F 的 [[Get]] 内部属性。
6.  依照 13.2.1 描述，设定 F 的 [[Call]] 内部属性。
7.  依照 13.2.2 描述，设定 F 的 [[Construct]] 内部属性。
8.  依照 15.3.5.3 描述，设定 F 的 [[HasInstance]] 内部属性。
9.  设定 F 的 [[Scope]] 内部属性为 Scope 的值。
10.  令 names 为一个列表容器，其中元素是以从左到右的文本顺序对应 FormalParameterList 的标识符的字符串。
11.  设定 F 的 [[FormalParameters]] 内部属性为 names。
12.  设定 F 的 [[Code]] 内部属性为 FunctionBody。
13.  设定 F 的 [[Extensible]] 内部属性为 true。
14.  令 len 为 FormalParameterList 指定的形式参数的个数。如果没有指定参数，则令 len 为 0。
15.  以参数 "length"，属性描述符 {[[Value]]: len, [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false}，false 调用 F 的 [[DefineOwnProperty]] 内部方法。
16.  令 proto 为仿佛使用 new Object() 表达式创建新对象的结果，其中 Object 是标准内置构造器名。
17.  以参数 "constructor", 属性描述符 {[[Value]]: F, { [[Writable]]: true, [[Enumerable]]: false, [[Configurable]]: true}, false 调用 proto 的 [[DefineOwnProperty]] 内部方法。
18.  以参数 "prototype", 属性描述符 {[[Value]]: proto, { [[Writable]]: true, [[Enumerable]]: false, [[Configurable]]: false}, false 调用 F 的 [[DefineOwnProperty]] 内部方法。
19.  如果 Strict 是 true，则
    1.  令 thrower 为 [[ThrowTypeError]] 函数对象 (13.2.3)。
    2.  以参数 "caller", 属性描述符 {[[Get]]: thrower, [[Set]]: thrower, [[Enumerable]]: false, [[Configurable]]: false}, false 调用 F 的 [[DefineOwnProperty]] 内部方法。
    3.  以参数 "caller", 属性描述符 {[[Get]]: thrower, [[Set]]: thrower, [[Enumerable]]: false, [[Configurable]]: false}, false 调用 F 的 [[DefineOwnProperty]] 内部方法。
20.  返回 F。

每个函数都会自动创建一个 prototype 属性，以满足函数会被当作构造器的可能性。

### [[call]]

当用一个 this 值，一个参数列表调用函数对象 F 的 [[Call]] 内部方法，采用以下步骤：

1.  用 F 的 [[FormalParameters]] 内部属性值，参数列表 args，10.4.3 描述的 this 值来建立 函数代码 的一个新执行环境，令 funcCtx 为其结果。
2.  令 result 为 FunctionBody（也就是 F 的 [[Code]] 内部属性）解释执行的结果。如果 F 没有 [[Code]] 内部属性或其值是空的 FunctionBody，则 result 是 (normal, undefined, empty)。
3.  退出 funcCtx 执行环境，恢复到之前的执行环境。
4.  如果 result.type 是 throw 则抛出 result.value。
5.  如果 result.type 是 return 则返回 result.value。
6.  否则 result.type 必定是 normal。返回 undefined。

### [[Construct]]

当以一个可能的空的参数列表调用函数对象 F 的 [[Construct]] 内部方法，采用以下步骤：

1.  令 obj 为新创建的 ECMAScript 原生对象。
2.  依照 8.12 设定 obj 的所有内部方法。
3.  设定 obj 的 [[Class]] 内部方法为 "Object"。
4.  设定 obj 的 [[Extensible]] 内部方法为 true。
5.  令 proto 为以参数 "prototype" 调用 F 的 [[Get]] 内部属性的值。
6.  如果 Type(proto) 是 Object，设定 obj 的 [[Prototype]] 内部属性为 proto。
7.  如果 Type(proto) 不是 Object，设定 obj 的 [[Prototype]] 内部属性为 15.2.4 描述的标准内置的 Object 的 prototype 对象。
8.  以 obj 为 this 值，调用 [[Construct]] 的参数列表为 args，调用 F 的 [[Call]] 内部属性，令 result 为调用结果。
9.  如果 Type(result) 是 Object，则返回 result。
10.  返回 obj

### [[ThrowTypeError]] 函数对象

[[ThrowTypeError]] 对象是个唯一的函数对象，如下只定义一次：

1.  创建一个新 ECMAScript 原生对象，令 F 为此对象。
2.  依照 8.12 设定 F 的所有内部属性。
3.  设定 F 的 [[Class]] 内部属性为 "Function"。
4.  设定 F 的 [[Prototype]] 内部属性为 15.3.3.1 指定的标准内置 Function 的 prototype 对象。
5.  依照 13.2.1 描述设定 F 的 [[Call]] 内部属性。
6.  设定 F 的 [[Scope]] 内部属性为 全局环境 。
7.  设定 F 的 [[FormalParameters]] 内部属性为一个空 列表 。
8.  设定 F 的 [[Code]] 内部属性为一个 FunctionBody，它无条件抛出一个 TypeError 异常，不做其他事情。
9.  以参数 "length", 属性描述符 {[[Value]]: 0, [[Writable]]: false, [[Enumerable]]: false, [[Configurable]]: false}, false 调用 F 的 [[DefineOwnProperty]] 内部方法。
10.  设定 F 的 [[Extensible]] 内部属性为 false。
11.  令 [[ThrowTypeError]] 为 F。