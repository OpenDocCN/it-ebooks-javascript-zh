# 可执行代码与执行环境

## 可执行代码类型

一共有 3 种 ECMA 脚本可执行代码：

*   全局代码 是指被作为 ECMA 脚本 程序 处理的源代码文本。一个特定 程序 的全局代码不包括作为 函数体 被解析的源代码文本。
*   Eval 代码 是指提供给 eval 内置函数的源代码文本。更精确地说，如果传递给 eval 内置函数的参数为一个字符串，该字符串将被作为 ECMA 脚本 程序 进行处理。在特定的一次对 eval 的调用过程中，eval 代码作为该 程序 的 #global-code 部分。
*   函数代码 是指作为 函数体 被解析的源代码文本。一个 函数体 的 函数代码 不包括作为其嵌套函数的 函数体 被解析的源代码文本。 函数代码 同时还特指 以构造器方式调用 Function 内置对象 时所提供的源代码文本。更精确地说，调用 Function 构造器时传递的最后一个参数将被转换为字符串并作为 函数体 使用。如果调用 Function 构造器时，传递了一个以上的参数，除最后一个参数以外的其他参数都将转换为字符串，并以逗号作为分隔符连接在一起成为一个字符串，该字符串被解析为 形参列表 供由最后一个参数定义的 函数体 使用。初始化 Function 对象时所提供的函数代码，并不包括作为其嵌套函数的 函数体 被解析的源代码文本。

### 严格模式下的代码

一个 ECMA 脚本程序的语法单元可以使用非严格或严格模式下的语法及语义进行处理。当使用严格模式进行处理时，以上三种代码将被称为严格全局代码、严格 eval 代码和严格函数代码。当符合以下条件时，代码将被解析为严格模式下的代码：

*   当 全局代码 以指令序言开始，且该指令序言包含一个使用严格模式的指令序言（参考 14.1 章 ）时，即为严格全局代码。
*   当 全局代码 以指令序言开始，且该指令序言包含一个使用严格模式的指令序言时；或者在 严格模式下的代码 中通过直接调用 eval 函数 （参考 15.1.2.1.1 章 ）时，即为严格 eval 代码。
*   当一个 函数声明 、 函数表达式 或 函数赋值 访问器处在一段 严格模式下的代码 中，或其函数代码以指令序言开始，且该指令序言包含一个使用严格模式的指令序言时，该函数代码 即为严格函数代码。
*   当调用内置的 Function 构造器时，如果最后一个参数所表达的字符串在作为 函数体 处理时以指令序言开始，且该指令序言包含一个使用严格模式的指令序言，则该 函数代码即为严格函数代码。

## 词法环境

词法环境 是一个用于定义特定变量和函数标识符在 ECMAScript 代码的词法嵌套结构上关联关系的规范类型。一个词法环境由一个环境记录项和可能为空的外部词法环境引用构成。通常词法环境会与特定的 ECMAScript 代码诸如 FunctionDeclaration,WithStatement 或者 TryStatement 的 Catch 块这样的语法结构相联系，且类似代码每次执行都会有一个新的语法环境被创建出来。

环境记录项记录了在它的关联词法环境域内创建的标识符绑定情形。

外部词法环境引用用于表示词法环境的逻辑嵌套关系模型。（内部）词法环境的外部引用是逻辑上包含内部词法环境的词法环境。外部词法环境自然也可能有多个内部词法环境。例如，如果一个 FunctionDeclaration 包含两个嵌套的 FunctionDeclaration，那么每个内嵌函数的词法环境都是外部函数本次执行所产生的词法环境。

词法环境和环境记录项是纯粹的规范机制，而不需要 ECMAScript 的实现保持一致。ECMAScript 程序不可能直接访问或者更改这些值。

### 环境记录项

在本标准中，共有 2 类环境记录项： 声明式环境记录项 和 对象式环境记录项 。声明式环境记录项用于定义那些将 标识符 与语言值直接绑定的 ECMA 脚本语法元素，例如 函数定义 ， 变量定义 以及 Catch 语句。对象式环境记录项用于定义那些将 标识符 与具体对象的属性绑定的 ECMA 脚本元素，例如 程序 以及 With 表达式 。

出于标准规范的目的，可以将环境记录项理解为面向对象中的一个简单继承结构，其中环境记录项是一个抽象类花前月下有 2 个具体实现类，分别为声明式环境记录项和对象式环境记录项。抽象类包含了表 17 所描述的抽象方法定义，针对每一个具体实现类，每个抽象方法都有不同的具体算法。

环境记录项的抽象方法

| 方法 | 作用 |
| HasBinding(N) | 判断环境记录项是否包含对某个标识符的绑定。如果包含该绑定则返回 true，反之返回 false。其中字符串 N 是标识符文本。 |
| CreateMutableBinding(N, D) |  在环境记录项中创建一个新的可变绑定。其中字符串 N 指定绑定名称。如果可选参数 D 的值为 true，则该绑定在后续操作中可以被删除。 |
| SetMutableBinding(N,V, S) | 在环境记录项中设置一个已经存在的绑定的值。其中字符串 N 指定绑定名称。V 用于指定绑定的值，可以是任何 ECMA 脚本语言的类型。S 是一个布尔类型的标记，当 S 为 true 并且该绑定不允许赋值时，则抛出一个 TypeError 异常。S 用于指定是否为严格模式。 |
| GetBindingValue(N,S) | 返回环境记录项中一个已经存在的绑定的值。其中字符串 N 指定绑定的名称。S 用于指定是否为严格模式。如果 S 的值为 true 并且该绑定不存在或未初始化，则抛出一个 ReferenceError 异常。 |
| DeleteBinding(N) | 从环境记录项中删除一个绑定。其中字符串 N 指定绑定的名称。如果 N 指定的绑定存在，将其删除并返回 true。如果绑定存在但无法删除则返回 false。如果绑定不存在则返回 true。 |
| ImplicitThisValue() | 当从该环境记录项的绑定中获取一个函数对象并且调用时，该方法返回该函数对象使用的 this 对象的值。 |

#### 声明式环境记录项

每个声明式环境记录项都与一个包含变量和（或）函数声明的 ECMA 脚本的程序作用域相关联。声明式环境记录项用于绑定作用域内定义的一系列标识符。

除了所有环境记录项都支持的可变绑定外，声明式环境记录项还提供不可变绑定。在不可变绑定中，一个标识符与它的值之间的关联关系建立之后，就无法改变。创建和初始化不可变绑定是两个独立的过程，因此类似的绑定可以处在已初始化阶段或者未初始化阶段。除了环境记录项定义的抽象方法外，声明式环境记录项还支持表 18 中列出的方法：

声明式环境记录项的额外方法

| 方法 | 作用 |
| CreateImmutableBinding(N) | 在环境记录项中创建一个未初始化的不可变绑定。其中字符串 N 指定绑定名称。 |
| InitializeImmutableBinding(N,V) | 在环境记录项中设置一个已经创建但未初始化的不可变绑定的值。其中字符串 N 指定绑定名称。V 用于指定绑定的值，可以是任何 ECMA 脚本语言的类型。 |

环境记录项定义的方法的具体行为将由以下算法给予描述。

##### HasBinding (N)

声明式环境记录项的 HasBinding 具体方法用于简单地判断作为参数的标识符是否是当前对象绑定的标识符之一：

1.  令 envRec 为函数调用时对应的声明式环境记录项。
2.  如果 envRec 有一个名称为 N 的绑定，返回 true。
3.  如果没有该绑定，返回 false。

##### CreateMutableBinding (N, D)

声明式环境记录项的 CreateMutableBinding 具体方法会创建一个名称为 N 的绑定，并初始化其值为 undefined。方法调用时，当前环境记录项中不能存在 N 的绑定。如果调用时提供了布尔类型的参数 D 且其值为 true，则新建的绑定被标记为可删除。

1.  令 envRec 为函数调用时对应的声明式环境记录项。
2.  执行断言：envRec 没有 N 的绑定。
3.  在 envRec 中为 N 创建一个可变绑定，并将绑定的值设置为 undefined。如果 D 为 true 则新创建的绑定可在后续操作中通过调用 DeleteBinding 删除。

##### SetMutableBinding (N,V,S)

声明式环境记录项的 SetMutableBinding 具体方法尝试将当前名称为参数 N 的绑定的值修改为参数 V 指定的值。方法调用时，必须存在 N 的绑定。如果该绑定为不可变绑定，并且 S 的值为 true，则抛出一个 TypeError 异常。

1.  令 envRec 为函数调用时对应的声明式环境记录项。
2.  执行断言：envRec 必须有 N 的绑定。
3.  如果 envRec 中 N 的绑定为可变绑定，则将其值修改为 V。
4.  否则该操作会尝试修改一个不可变绑定的值，因此如果 S 的值为 true，则抛出一个 TypeError 异常。

##### GetBindingValue (N,S)

声明式环境记录项的 GetBindingValue 具体方法简单地返回名称为参数 N 的绑定的值。方法调用时，该绑定必须存在。如果 S 的值为 true 且该绑定是一个未初始化的不可变绑定，则抛出一个 ReferenceError 异常。

1.  令 envRec 为函数调用时对应的声明式环境记录项。
2.  执行断言：envRec 必须有 N 的绑定。
3.  如果 envRec 中 N 的绑定是一个未初始化的不可变绑定，则：
    1.  如果 S 为 false，返回 undefined，否则抛出一个 ReferenceError 异常。
4.  否则返回 envRec 中与 N 绑定的值。

##### DeleteBinding (N)

声明式环境记录项的 DeleteBinding 具体方法只能删除显示指定可被删除的那些绑定。

1.  令 envRec 为函数调用时对应的声明式环境记录项。
2.  如果 envRec 不包含名称为 N 的绑定，返回 true。
3.  如果 envRec 中 N 的绑定不能删除，返回 false。
4.  移除 envRec 中 N 的绑定。
5.  返回 true。

##### ImplicitThisValue()

声明式环境记录项永远将 undefined 作为其 ImplicitThisValue 返回。

1.  返回 undefined。

##### CreateImmutableBinding (N)

声明式环境记录项的 CreateImmutableBinding 具体方法会创建一个不可变绑定，其名称为 N 且初始化其值为 undefined。调用方法时，该环境记录项中不得存在 N 的绑定。

1.  令 envRec 为函数调用时对应的声明式环境记录项。
2.  执行断言：envRec 不存在 N 的绑定。
3.  在 envRec 中为 N 创建一个不可变绑定，并记录为未初始化。

##### InitializeImmutableBinding (N,V)

声明式环境记录项的 InitializeImmutableBinding 具体方法用于将当前名称为参数 N 的绑定的值修改为参数 V 指定的值。方法调用时，必须存在 N 对应的未初始化的不可变绑定。

1.  令 envRec 为函数调用时对应的声明式环境记录项。
2.  执行断言：envRec 存在一个与 N 对应的未初始化的不可变绑定。
3.  在 envRec 中将 N 的绑定的值设置为 V。
4.  在 envRec 中将 N 的不可变绑定记录为已初始化。

#### 对象式环境记录项

每一个对象式环境记录项都有一个关联的对象，这个对象被称作 绑定对象 。对象式环境记录项直接将一系列标识符与其绑定对象的属性名称建立一一对应关系。不符合 IdentifierName 的属性名不会作为绑定的标识符使用。无论是对象自身的，还是继承的属性都会作为绑定，无论该属性的 [[Enumerable]] 特性的值是什么。由于对象的属性可以动态的增减，因此对象式环境记录项所绑定的标识符集合也会隐匿地变化，这是增减绑定对象的属性而产生的副作用。通过以上描述的副作用而建立的绑定，均被视为可变绑定，即使该绑定对应的属性的 Writable 特性的值为 false。对象式环境记录项没有不可变绑定。

对象式环境记录项可以通过配置的方式，将其绑定对象合为函数调用时的隐式 this 对象的值。这一功能用于规范 With 表达式（12.10 章 ）引入的绑定行为。该行为通过对象式环境记录项中布尔类型的 provideThis 值控制，默认情况下，provideThis 的值为 false。

环境记录项定义的方法的具体行为将由以下算法给予描述。

##### HasBinding(N)

对象式环境记录项的 HasBinding 具体方法判断其关联的绑定对象是否有名为 N 的属性：

1.  令 envRec 为函数调用时对应的声明式环境记录项。
2.  令 bindings 为 envRec 的绑定对象。
3.  以 N 为属性名，调用 bindings 的 [[HasProperty]] 内部方法，并返回调用的结果。

##### CreateMutableBinding (N, D)

对象式环境记录项的 CreateMutableBinding 具体方法会在其关联的绑定对象上创建一个名称为 N 的属性，并初始化其值为 undefined。调用方法时，绑定对象不得包含名称为 N 的属性。如果调用方法时提供了布尔类型的参数 D 且其值为 true，则设置新创建的属性的 [[Configurable]] 特性的值为 true，否则设置为 false。

1.  令 envRec 为函数调用时对应的声明式环境记录项。
2.  令 bindings 为 envRec 的绑定对象。
3.  执行断言：以 N 为属性名，调用 bindings 的 [[HasProperty]] 内部方法，调用的结果为 false。
4.  如果 D 的值为 true，则令 configValue 的值为 true，否则令 configValue 的值为 false。
5.  以 N、属性描述符 {[[Value]]:undefined, [[Writable]]: true, [[Enumerable]]: true , [[Configurable]]: configValue} 和布尔值 true 为参数，调用 bindings 的 [[DefineOwnProperty]] 内部方法。

##### SetMutableBinding (N,V,S)

对象式环境记录项的 SetMutableBinding 具体方法会尝试设置其关联的绑定对象中名为 N 的属性的值为 V。方法调用时，绑定对象中应当存在该属性，如果该属性不存在或属性不可写，则根据 S 参数的值来执行错误处理。

1.  令 envRec 为函数调用时对应的声明式环境记录项。
2.  令 bindings 为 envRec 的绑定对象
3.  以 N、V 和 S 为参数，调用 bindings 的 [[Put]] 内部方法。

##### GetBindingValue(N,S)

对象式环境记录项的 GetBindingValue 具体方法返回其关联的绑定对象中名为 N 的属性的值。方法调用时，绑定对象中应当存在该属性，如果该属性不存在，则方法的返回值由 S 参数决定：

1.  令 envRec 为函数调用时对应的声明式环境记录项。
2.  令 bindings 为 envRec 的绑定对象
3.  以 N 为属性名，调用 bindings 的 [[HasProperty]] 内部方法，并令 value 为调用的结果。
4.  如果 value 的值为 false，则：
    1.  如果 S 的值为 false，则返回 undefined，否则抛出一个 ReferenceError 异常。
5.  以 N 为参数，调用 bindings 的 [[Get]] 内部方法，并返回调用的结果。

##### DeleteBinding (N)

对象式环境记录项的 DeleteBinding 具体方法只能用于删除其关联的绑定对象上 [[Configurable]] 特性的值为 true 的属性所对应的绑定。

1.  令 envRec 为函数调用时对应的声明式环境记录项。
2.  令 bindings 为 envRec 的绑定对象
3.  以 N 和布尔值 false 为参数，调用 bindings 的 [[Delete]] 内部方法。

##### ImplicitThisValue()

对象式环境记录项的 ImplicitThisValue 通常返回 undefined，除非其 provideThis 标识的值为 true。

1.  令 envRec 为函数调用时对应的声明式环境记录项。
2.  如果 envRec 的 provideThis 标识的值为 true，返回 envRec 的绑定对象。
3.  否则返回 undefined。

### 词法环境的运算

在本标准中，以下抽象运算将被用于操作环境记录项：

#### GetIdentifierReference (lex, name, strict)

当调用 GetIdentifierReference 抽象运算时，需要指定一个 词法环境 lex，一个标识符字符串 name 以及一个布尔型标识 strict。lex 的值可以为 null。当调用该运算时，按以下步骤进行：

1.  如果 lex 的值为 null，则：
    1.  返回一个类型为 引用 的对象，其基值为 undefined，引用的名称为 name，严格模式标识的值为 strict。
2.  令 envRec 为 lex 的环境数据。
3.  以 name 为参数 N，调用 envRec 的 HasBinding(N) 具体方法，并令 exists 为调用的结果。
4.  如果 exists 为 true，则：
    1.  返回一个类型为 引用 的对象，其基值为 envRec，引用的名称为 name，严格模式标识的值为 strict。
5.  否则：
    1.  令 outer 为 lex 的 外部环境引用 。
    2.  以 outer、name 和 struct 为参数，调用 GetIdentifierReference，并返回调用的结果。

#### NewDeclarativeEnvironment (E)

当调用 NewDeclarativeEnvironment 抽象运算时，需指定一个 词法环境 E，其值可以为 null，此时按以下步骤进行：

1.  令 env 为一个新建的 词法环境 。
2.  令 envRec 为一个新建的 声明式环境数据 ，该环境数据不包含任何绑定。
3.  令 env 的环境数据为 envRec。
4.  令 env 的外部词法环境引用至 E。
5.  返回 env。

#### NewObjectEnvironment (O, E)

当调用 NewObjectEnvironmentis 抽象运算时，需指定一个对象 O 及一个 词法环境 E（其值可以为 null），此时按以下步骤进行：

1.  令 env 为一个新建的 词法环境 。
2.  令 envRec 为一个新建的 对象环境数据 ，该环境数据包含 O 作为绑定对象。
3.  令 env 的环境数据为 envRec。
4.  令 env 的外部词法环境引用至 E。
5.  返回 env。

### 全局环境

全局环境 是一个唯一的 词法环境 ，它在任何 ECMA 脚本的代码执行前创建。全局环境的 环境数据 是一个 #object-environment-record 对象环境数据，该环境数据使用 全局对象（15.1）作为 绑定对象 。全局环境的 外部环境引用 为 null。

在 ECMA 脚本的代码执行过程中，可能会向 全局对象 添加额外的属性，也可能修改其初始属性的值。

## 执行环境

当控制器转入 ECMA 脚本的可执行代码时，控制器会进入一个执行环境。当前活动的多个执行环境在逻辑上形成一个栈结构。该逻辑栈的最顶层的执行环境称为当前运行的执行环境。任何时候，当控制器从当前运行的执行环境相关的可执行代码转入与该执行环境无关的可执行代码时，会创建一个新的执行环境。新建的这个执行环境会推入栈中，成为当前运行的执行环境。

执行环境包含所有用于追踪与其相关的代码的执行进度的状态。精确地说，每个执行环境包含如表 19 列出的组件。

执行环境的状态组件

| 组件 | 作用目的 |
| 词法环境 | 指定一个词法环境对象，用于解析该执行环境内的代码创建的标识符引用。 |
| 变量环境 | 指定一个词法环境对象，其环境数据用于保存由该执行环境内的代码通过 变量表达式 和 函数表达式 创建的绑定。 |
| >This 绑定 | 指定该执行环境内的 ECMA 脚本代码中 this 关键字所关联的值。 |

其中执行环境的词法环境和变量环境组件始终为 词法环境 对象。当创建一个执行环境时，其词法环境组件和变量环境组件最初是同一个值。在该执行环境相关联的代码的执行过程中，变量环境组件永远不变，而词法环境组件有可能改变。

在本标准中，通常情况下，只有正在运行的执行环境（执行环境栈里的最顶层对象）会被算法直接修改。因此当遇到“词法环境”，“变量环境”和“This 绑定”这三个术语时，指的是正在运行的执行环境的对应组件。

执行环境是一个纯粹的标准机制，并不代表任何 ECMA 脚本实现的工件。在 ECMA 脚本程序中是不可能访问到执行环境的。

### 标识符解析

标识符解析是指使用正在运行的执行环境中的词法环境，通过一个 标识符 获得其对应的绑定的过程。在 ECMA 脚本代码执行过程中，PrimaryExpression : Identifier 这一语法产生式将按以下算法进行解释执行：

1.  令 env 为正在运行的执行环境的 词法环境 。
2.  如果正在解释执行的语法产生式处在 严格模式下的代码 中，则仅 strict 的值为 true，否则令 strict 的值为 false。
3.  以 env，Identifier 和 strict 为参数，调用 GetIdentifierReference 函数，并返回调用的结果。

解释执行一个标识符得到的结果必定是 引用 类型的对象，且其引用名属性的值与 Identifier 字符串相等。

## 建立执行环境

解释执行 全局代码 或使用 eval 函数（15.1.2.1）输入的代码会创建并进入一个新的执行环境。每次调用 ECMA 脚本代码定义的函数（13.2.1）也会建立并进入一个新的执行环境，即便函数是自身递归调用的。每一次 return 都会退出一个执行环境。抛出异常也可退出一个或多个执行环境。

当控制流进入一个执行环境时，会设置该执行环境的 this 绑定，定义变量环境和初始词法环境，并执行定义绑定初始化过程（10.5）。以上这些步骤的严格执行方式由进入的代码的类型决定。

### 进入全局代码

当控制流进入 全局代码 的执行环境时，执行以下步骤：

1.  按 10.4.1.1 描述的方案，使用 全局代码 初始化执行环境。
2.  按 10.5 描述的方案，使用 全局代码 执行定义绑定初始化步骤。

#### 10.4.1.1 初始化全局执行环境

以下步骤描述 ECMA 脚本的全局执行环境 C 的创建过程：

1.  将变量环境设置为 全局环境 。
2.  将词法环境设置为 全局环境 。
3.  将 this 绑定设置为 全局对象 。

### 进入 eval 代码

当控制流进入 eval 代码 的执行环境时，执行以下步骤：

1.  如果没有调用环境，或者 eval 代码 并非通过直接调用（15.1.2.1.1）eval 函数进行评估的，则
    1.  按（10.4.1.1）描述的初始化全局执行环境的方案，以 eval 代码 作为 C 来初始化执行环境。
2.  否则
    1.  将 this 绑定设置为当前执行环境下的 this 绑定。
    2.  将词法环境设置为当前执行环境下的 词法环境 。
    3.  将变量环境设置为当前执行环境下的变量环境。
3.  如果 eval 代码 是 严格模式下的代码 ，则
    1.  令 strictVarEnv 为以词法环境为参数调用 NewDeclarativeEnvironment 得到的结果。
    2.  设置词法环境为 strictVarEnv。
    3.  设置变量环境为 strictVarEnv。
4.  按 10.5 描述的方案，使用 eval 代码 执行定义绑定初始化步骤。

#### 严格模式下的限制

如果调用环境的代码或 eval 代码 是 严格模式下的代码 ，则 eval 代码不能在调用环境的变量环境中 初始化变量及函数绑定 。与之相对的，变量及函数绑定将在一个新的环境变量中被初始化，该环境变量仅可被 eval 代码 访问。

### 进入函数代码

当控制流根据一个函数对象 F、调用者提供的 thisArg 以及调用者提供的 argumentList，进入 函数代码 的执行环境时，执行以下步骤：

1.  如果 函数代码 是 严格模式下的代码 ，设 this 绑定为 thisArg。
2.  否则如果 thisArg 是 null 或 undefined，则设 this 绑定为 全局对象 。
3.  否则如果 Type(thisArg) 的结果不为 Object，则设 this 绑定为 ToObject(thisArg)。
4.  否则设 this 绑定为 thisArg。
5.  以 F 的 [[Scope]] 内部属性为参数调用 NewDeclarativeEnvironment，并令 localEnv 为调用的结果。
6.  设词法环境为 localEnv。
7.  设变量环境为 localEnv。
8.  令 code 为 F 的 [[Code]] 内部属性的值。
9.  按 10.5 描述的方案，使用 函数代码 code 和 argumentList 执行定义绑定初始化步骤。

## 定义绑定初始化

每个执行环境都有一个关联的变量环境。当在一个执行环境下评估一段 ECMA 脚本时，变量和函数定义会以绑定的形式添加到这个变量环境的 环境记录 中。对于函数 函数代码，参数也同样会以绑定的形式添加到这个变量环境的 环境记录 中。

选择使用哪一个、哪一类型的 环境记录 来绑定定义，是由执行环境下执行的 ECMA 脚本的类型决定的，而其它部分的逻辑是相同的。当进入一个执行环境时，会按以下步骤在变量环境上创建绑定，其中使用到调用者提供的代码设为 code，如果执行的是 函数代码 ，则设 参数列表 为 args：

1.  令 env 为当前运行的执行环境的环境变量的 环境记录 。
2.  如果 code 是 eval 代码 ，则令 configurableBindings 为 true，否则令 configurableBindings 为 false。
3.  如果代码是 严格模式下的代码 ，则令 strict 为 true，否则令 strict 为 false。
4.  如果代码为 函数代码 ，则：
    1.  令 func 为通过 [[Call]] 内部属性初始化 code 的执行的函数对象。令 names 为 func 的 [[FormalParameters]] 内部属性。
    2.  令 argCount 为 args 中元素的数量。
    3.  令 n 为数字类型，其值为 0。
    4.  按列表顺序遍历 names，对于每一个字符串 argName：
        1.  令 n 的值为 n 当前值加 1。
        2.  如果 n 大于 argCount，则令 v 为 undefined，否则令 v 为 args 中的第 n 个元素。
        3.  以 argName 为参数，调用 env 的 HasBinding 具体方法，并令 argAlreadyDeclared 为调用的结果。
        4.  如果 argAlreadyDeclared 的值为 false，以 argName 为参数调用 env 的 CreateMutableBinding 具体方法。
        5.  以 argName、v 和 strict 为参数，调用 env 的 SetMutableBinding 具体方法。
5.  按源码顺序遍历 code，对于每一个 FunctionDeclaration 表达式 f：
    1.  令 fn 为 FunctionDeclaration 表达式 f 中的 标识符 。
    2.  按 第十三章 中所述的步骤初始化 FunctionDeclaration 表达式 ，并令 fo 为初始化的结果。
    3.  以 fn 为参数，调用 env 的 HasBinding 具体方法，并令 argAlreadyDeclared 为调用的结果。
    4.  如果 argAlreadyDeclared 的值为 false，以 fn 和 configurableBindings 为参数调用 env 的 CreateMutableBinding 具体方法。
    5.  否则如果 env 是全局环境的 环境记录 对象，则：
        1.  令 go 为全局对象。
        2.  以 fn 为参数，调用 go 和 [[GetProperty]] 内部方法，并令 existingProp 为调用的结果。
        3.  如果 existingProp.[[Configurable]] 的值为 true，则：
            1.  以 fn、由 {[[Value]]: undefined, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: configurableBindings } 组成的 属性描述符 和 true 为参数，调用 go 的 [[DefineOwnProperty]] 内部方法。
        4.  否则如果 IsAccessorDescrptor(existingProp) 的结果为真，或 existingProp 的特性中没有 {[[Writable]]: true, [[Enumerable]]: true}，则：
            1.  抛出一个 TypeError 异常。
        5.  以 fn、fo 和 strict 为参数，调用 env 的 SetMutableBinding 具体方法。
6.  以 arguments 为参数，调用 env 的 HasBinding 具体方法，并令 argumentsAlreadyDeclared 为调用的结果。
7.  如果 code 是 函数代码 ，并且 argumentsAlreadyDeclared 为 false，则：
    1.  以 fn、names、args、env 和 strict 为参数，调用 CreateArgumentsObject 抽象运算函数，并令 argsObj 为调用的结果。
    2.  如果 strict 为 true，则：
        1.  以字符串 "arguments" 为参数，调用 env 的 CreateImmutableBinding 具体方法。
        2.  以字符串 "arguments" 和 argsObj 为参数，调用 env 的 InitializeImmutableBinding 具体函数。
    3.  否则：
        1.  以字符串 "arguments" 为参数，调用 env 的 CreateMutableBinding 具体方法。
        2.  以字符串 "arguments"、argsObj 和 false 为参数，调用 env 的 SetMutableBinding 具体函数。
8.  按源码顺序遍历 code，对于每一个 VariableDeclaration 和 VariableDeclarationNoIn 表达式：
    1.  令 dn 为 d 中的标识符。
    2.  以 dn 为参数，调用 env 的 HasBinding 具体方法，并令 varAlreadyDeclared 为调用的结果。
    3.  如果 varAlreadyDeclared 为 false，则：
        1.  以 dn 和 configurableBindings 为参数，调用 env 的 CreateMutableBinding 具体方法。
        2.  以 dn、undefined 和 strict 为参数，调用 env 的 SetMutableBinding 具体方法。

## Arguments 对象

当控制器进入到函数代码的执行环境时，将创建一个 arguments 对象，除非它作为标识符 "arguments" 出现在该函数的形参列表中，或者是该函数代码内部的变量声明标识符或函数声明标识符。

Arguments 对象通过调用抽象方法 CreateArgumentsObject 创建，调用时将以下参数传入：func, names, args, env, strict。将要执行的函数对象作为 func 参数，将该函数的所有形参名加入一个 List 列表，作为 names 参数，将所有传给内部方法 [[Call]] 的实际参数，作为 args 参数，将该函数代码的环境变量作为 env 参数，将该函数代码是否为严格代码作为 strict 参数。当 CreateArgumentsObject 调用时，按照以下步骤执行：

1.  令 len 为参数 args 的元素个数
2.  令 obj 为一个新建的 ECMAScript 对象
3.  按照 8.12 章节中的规范去设定 obj 对象的所有内部方法
4.  将 obj 对象的内部属性 [[Class]] 设置为 "Arguments"
5.  令 Object 为标准的内置对象的构造函数 (15.2.2)
6.  将 obj 对象的内部属性 [[Prototype]] 设置为标准的内置对象的原型对象
7.  调用 obj 的内部方法 [[DefineOwnProperty]]，将 "length" 传递进去，属性描述符为：{[[Value]]: len, [[Writable]]: true, [[Enumerable]]: false, [[Configurable]]: true}，参数为 false
8.  令 map 为表达式 new Object() 创建的对象，就是名为 Object 的标准的内置构造函数
9.  令 mappedNames 为一个空的 List 列表
10.  令 indx = len - 1
11.  当 indx >= 0 的时候，重复此过程：
    1.  令 val 为 args（维度从 0 开始的 list 列表）的第 indx 维度所在的元素
    2.  调用 obj 的内部方法 [[DefineOwnProperty]]，将 ToString(indx) 传递进去，属性描述符为：{[[Value]]: val, [[Writable]]: true, [[Enumerable]]: true, [[Configurable]]: true}，参数为 false
    3.  如果 indx 小于 names 的元素个数，则
        1.  令 name 为 names（维度从 0 开始的 list 列表）的第 indx 维度所在的元素
        2.  如果 strict 值为 false，且 name 不是一个 mappedNames 元素，则
            1.  将 name 添加到 mappedNames 列表中，作为它的一个元素
            2.  令 g 为调用抽象操作 MakeArgGetter 的结果，其参数为 name 和 env
            3.  令 p 为调用抽象操作 MakeArgSetter 的结果，其参数为 name 和 env
            4.  调用 map 对象的内部方法 [[DefineOwnProperty]]，将 ToString(indx) 传递进去，属性描述符为：{[[Set]]: p, [[Get]]: g, [[Configurable]]: true}，参数为 false
    4.  令 indx = indx - 1
12.  如果 mappedNames 不为空，则
    1.  将 obj 对象的内部属性 [[ParameterMap]] 设置为 map
    2.  将 obj 对象的内部方法 [[Get]], [[GetOwnProperty]], [[DefineOwnProperty]], [[Delete]] 按下面给出的定义进行设置。
13.  如果 strict 值为 false，则
    1.  调用 obj 对象的内部方法 [[DefineOwnProperty]]，将 "callee" 传递进去，属性描述符为；{[[Value]]: func, [[Writable]]: true, [[Enumerable]]: false, [[Configurable]]: true}，参数为 false
14.  否则，strict 值为 true，那么
    1.  令 thrower 为 [[ThrowTypeError]] 函数对象 (13.2.3)
    2.  调用 obj 对象的内部方法 [[DefineOwnProperty]]，将 "caller" 传递进去，属性描述符为 :{[[Get]]: thrower, [[Set]]: thrower, [[Enumerable]]: false, [[Configurable]]: false}，参数为 false
    3.  调用 obj 对象的内部方法 [[DefineOwnProperty]]，将 "callee" 传递进去，属性描述符为 :{[[Get]]: thrower, [[Set]]: thrower, [[Enumerable]]: false, [[Configurable]]: false}，参数为 false
15.  返回 obj

抽象操作 MakeArgGetter 以字符串 name 和环境记录 env 作为参数被调用时，会创建一个函数对象，当执行完后，会返回在 env 中绑定的 name 的值。执行步骤如下：

1.  令 body 为字符 "return "，name，";" 的连接字符串
2.  返回一个按照 13.2 章节中描述的方式创建的函数对象，它不需要形参列表，以 body 作为它的 FunctionBody，以 env 作为它的 Scope，并且 Strict 值为 true

抽象操作 MakeArgSetter 以字符串 name 和环境记录 env 作为参数被调用时，会创建一个函数对象，当执行完后，会给在 env 中绑定的 name 设置一个值。执行步骤如下：

1.  令 param 为 name 和字符串 "_arg" 的连接字符串
2.  令 body 为字符串 " <name>= <param> ;"；将 <name> 替换为 name 的值，将 <param> 替换为 param 的值 </name></name> 
3.  返回一个按照 13.2 章节中描述的方式创建的函数对象，以一个只包含字符串 param 的 list 列表作为它的形参，以 body 作为它的函数体（FunctionBody），以 env 作为它的 Scope，并且 Strict 值为 true

当 arguments 对象的内部方法 [[Get]] 在一个非严格模式下带有形参的函数中，在一个属性名为 P 的条件下被调用时，其执行步骤如下：

1.  令 map 为 arguments 对象的内部属性 [[ParameterMap]]
2.  令 isMapped 为 map 对象的内部方法 [[GetOwnPropery]] 传入参数 P 的执行结果
3.  如果 isMapped 值为 undefined，则
    1.  令 v 为 arguments 对象的内部默认的 [[Get]] 方法（8.12.3），传入参数 P 的执行结果
    2.  如果 P 为 "caller"，且 v 为严格模式下的 Function 对象，则抛出一个 TypeError 的异常
    3.  返回 v
4.  否则，map 包含一个 P 的形参映射表
    1.  返回 map 对象的内部方法 [[Get]] 传入参数 P 的执行结果

当 arguments 对象的内部方法 [[GetOwnProperty]] 在一个非严格模式下带有形参的函数中，在一个属性名为 P 的条件下被调用时，其执行步骤如下：

1.  令 desc 为 arguments 对象的内部方法 [[GetOwnPropery]]（8.12.1）传入参数 P 的执行结果
2.  如果 desc 为 undefined，返回 desc
3.  令 map 为 arguments 对象内部属性 [[ParameterMap]] 的值
4.  令 isMapped 为 map 对象的内部方法 [[GetOwnPropery]] 传入参数 P 的执行结果
5.  如果 isMapped 的值不是 undefined，则
    1.  将 desc 的值设置为 map 对象的内部方法 [[Get]] 传入参数 P 的执行结果
6.  返回 desc

当 arguments 对象的内部方法 [[DefineOwnProperty]] 在一个非严格模式下带有形参的函数中，在一个属性名为 P，属性描述符为 Desc，布尔标志为 Throw 的条件下被调用时，其执行步骤如下：

1.  令 map 为 arguments 对象的内部属性 [[ParameterMap]] 的值
2.  令 isMapped 为 map 对象的内部方法 [[GetOwnPropery]] 传入参数 P 的执行结果
3.  令 allowed 为 arguments 对象的内部方法 [[DefineOwnPropery]]（8.12.9）传入参数 P，Desc，false 的执行结果
4.  如果 allowed 为 false，则
    1.  如果 Throw 为 true，则抛出一个 TypeError 的异常，否则，返回 false
5.  如果 isMapped 的值不为 undefined，则
    1.  如果 IsAccessorDescriptor(Desc) 为 true，则
        1.  调用 map 对象的内部方法 [[Delete]]，传入 P 和 false 作为参数
    2.  否则
        1.  如果 Desc.[[Value]] 存在，则
            1.  调用 map 对象的内部方法 [[Put]]，传入 P，Desc.[[Value]] 和 Throw 作为参数
        2.  如果 Desc.[[Writable]] 存在，且其值为 false，则
            1.  调用 map 对象的内部方法 [[Delete]]，传入 P 和 false 作为参数
6.  返回 true

当 arguments 对象的内部方法 [[Delete]] 在一个非严格模式下带有形参的函数中，在一个属性名为 P，布尔标志为 Throw 的条件下被调用时，其执行步骤如下：

1.  令 map 为 arguments 对象的内部属性 [[ParameterMap]] 的值
2.  令 isMapped 为 map 对象的内部方法 [[GetOwnPropery]] 传入参数 P 的执行结果
3.  令 result 为 arguments 对象的内部方法 [[Delete]]（8.12.7）传入参数 P 和 Throw 的执行结果
4.  如果 result 为 true，且 isMapped 不为 undefined，则
    1.  调用 map 对象的内部方法 [[Delete]]，传入 P 和 false 作为参数
5.  返回 result

非严格模式下的函数，arguments 对象以数组索引（参见 15.4 的定义）作为数据属性的命名，其数字名字的值少于对应的函数对象初始时的形参数量，它们与绑定在该函数执行环境中对应的参数共享值。这意味着，改变该属性将改变这些对应的、绑定的参数的值，反之亦然。如果其中一个属性被删除然后再对其重定义，或者其中一个属性在某个访问器属性内部被更改，则这种对应关系将被打破。严格模式下的函数，arguments 对象的属性值就是传入该函数的实际参数的简单拷贝，它们与形参之间的值不存在动态的联动关系。

ParameterMap 对象和它的属性值被作为说明 arguments 对象对应绑定参数的装置。ParameterMap 对象和它的属性值对象不能直接被 ECMAScript 代码访问。作为 ECMAScript 的实现，不需要实际创建或使用这些对象去实现指定的语义。

严格模式下函数的 Arguments 对象定义的非可配置的访问器属性，"caller" 和 "callee"，在它们被访问时，将抛出一个 TypeError 的异常。在非严格模式下，"callee" 属性具有非常明确的意义，"caller" 属性有一个历史问题，它是否被提供，视为一个由实作环境决定的，在具体的 ECMAScript 实作进行扩展。在严格模式下对这些属性的定义的出现是为了确保它们俩谁也不能在规范的 ECMAScript 实作中以任何方式被定义。