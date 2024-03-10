# 语句

语法：

`Statement : Block VariableStatement EmptyStatement ExpressionStatement IfStatement IterationStatement ContinueStatement BreakStatement ReturnStatement WithStatement LabelledStatement SwitchStatement ThrowStatement TryStatement DebuggerStatement`

语义：

一个 Statement 可以是 LabelledStatement 的一部分，这个 LabelledStatement 自身也可以是 LabelledStatement 的一部分，以此类推。当描述个别语句时引入标签的这种方式统称为“当前标签组”。一个 LabelledStatement 介绍了一个标签到一个 标签组 ，此外没有其他语义。一个 IterationStatement 或 SwitchStatement 的标签组最初包含单个 空 元素。任何其他语句的标签组最初是空的。

The result of evaluating a Statement is always a Completion value.

已知几个广泛使用的 ECMAScript 实现支持 FunctionDeclaration 当作语句使用。然而，在实现之间这种 FunctionDeclarations 应用的语义也有严重且不兼容的差异。由于这些不兼容的差异，将 FunctionDeclaration 当作 Statement 使用的结果是代码在实现之间的可移植性不可靠。建议 ECMAScript 实现禁止这样运用 FunctionDeclaration，或遇到这样的运用是发出一个警告。ECMAScript 的未来版本可能定义替代的兼容方案以在 Statement 上下文中声明函数。

## 块

语法：

`Block : { StatementListopt }` `StatementList : Statement StatementList Statement`

语义：

产生式 Block : { } 按照下面的过程执行 :

1.  返回 (normal, empty, empty)。

产生式 Block : { StatementList } 按照下面的过程执行 :

1.  返回解释执行 StatementList 的结果。

产生式 StatementList :Statement 按照下面的过程执行 :

1.  令 s 为解释执行 Statement 的结果。
2.  如果有一个异常被抛出，返回 (throw, V, empty)，这里的 V 是异常。( 仿佛没有抛出异常一样继续运行。)
3.  返回 s。

产生式 StatementList :StatementList Statement 按照下面的过程执行 :

1.  令 sl 为解释执行 StatementList 的结果。
2.  如果 sl 是个非常规完结，返回 sl。
3.  令 s 为解释执行 Statement 的结果。
4.  如果有一个异常被抛出，返回 (throw, V, empty)，这里的 V 是异常。 ( 仿佛没有抛出异常一样继续运行。)
5.  如果 s.value 是 empty ，令 V = sl.value, 否则令 V = s.value。
6.  返回 (s.type, V, s.target)。

以上算法中步骤 5 和步骤 6 确保了 StatementList 的值是 StatementList 中最后一个产生值的 Statement 的值。例如以下 eval 函数的调用全都返回 1

`eval("1;;;;;") eval("1;{}") eval("1;var a;")`

## 变量语句

语法：

`VariableStatement : var VariableDeclarationList ;` `VariableDeclarationList : VariableDeclaration VariableDeclarationList , VariableDeclaration``VariableDeclarationListNoIn : VariableDeclarationNoIn VariableDeclarationListNoIn , VariableDeclarationNoIn` `VariableDeclaration : Identifier Initialiseropt` `VariableDeclarationNoIn : Identifier InitialiserNoInopt` `Initialiser : = AssignmentExpression` `InitialiserNoIn : = AssignmentExpressionNoIn`

一个变量语句声明依 10.5 中定义创建的变量。当创建变量时初始化为 undefined。当 VariableStatement 被执行时变量关联的 Initialiser 会被分配 AssignmentExpression 的值，而不是在变量创建时。

语义：

产生式 VariableStatement : var VariableDeclarationList ; 按照下面的过程执行 :

1.  解释执行 VariableDeclarationList.
2.  返回 (normal, empty, empty).

产生式 VariableDeclarationList : VariableDeclaration 按照下面的过程执行 :

1.  解释执行 VariableDeclaration.

产生式 VariableDeclarationList : VariableDeclarationList , VariableDeclaration 按照下面的过程执行 :

1.  解释执行 VariableDeclarationList.
2.  解释执行 VariableDeclaration.

产生式 VariableDeclaration : Identifier 按照下面的过程执行 :

1.  返回一个包含跟 Identifier. 完全相同的字符序列的字符串值

产生式 VariableDeclaration : Identifier Initialiser 按照下面的过程执行 :

1.  令 lhs 为解释执行 Identifier 的结果 as described in 11.1.2.
2.  令 rhs 为解释执行 Initialiser 的结果 .
3.  令 value 为 GetValue(rhs).
4.  Call PutValue(lhs, value).
5.  返回一个包含跟 Identifier. 完全相同的字符序列的字符串值

VariableDeclaration 的字符串值用在 for-in 语句 (12.6.4) 的解释执行。

如果 VariableDeclaration 嵌套在 with 语句里并且 VariableDeclaration 里的标识符与 with 语句的对象式环境记录项关联的绑定对象的一个属性名相同，则第 4 步将给这个属性分配值，而不是为 Identifier 的 VariableEnvironment 绑定分配值。

产生式 Initialiser : = AssignmentExpression 按照下面的过程执行 :

1.  返回解释执行 AssignmentExpression 的结果 .

产生式 VariableDeclarationListNoIn, VariableDeclarationNoIn, InitialiserNoIn 解释执行的方式与产生式 VariableDeclarationList, VariableDeclaration，Initialiser 相同，除了他们包含的 VariableDeclarationListNoIn, VariableDeclarationNoIn, InitialiserNoIn, AssignmentExpressionNoIn 会分别替代 VariableDeclarationList, VariableDeclaration, Initialiser, AssignmentExpression 来解释执行。

### 严格模式的限制

如果一个 VariableDeclaration 或 VariableDeclarationNoIn 出现在 严格模式代码 里并且其 Identifier 是 "eval" 或 "arguments"，那么这是个 SyntaxError。

## 空语句

语法 :

`EmptyStatement : ;`

语义：

产生式 EmptyStatement : ; 按照下面的过程执行 :

1.  返回 (normal, empty, empty).

## 表达式语句

语法：

`ExpressionStatement : [lookahead ? {{, function}]Expression ;`

一个 ExpressionStatement 不能用一个开大括号开始，因为这可能会使它和 Block 混淆。此外，ExpressionStatement 不能用 function 关键字开始，因为这可能会使它和 FunctionDeclaration 混淆。

语义：

产生式 ExpressionStatement : [lookahead ? {{',' function}]Expression; 按照下面的过程执行 :

1.  令 exprRef 为解释执行 Expression 的结果 .
2.  返回 (normal, GetValue(exprRef), empty).

## if 语句

语法：

`IfStatement : if ( Expression ) Statement else Statement if ( Expression ) Statement`

每个 else 选择与它相关联的 if 是不确定的，应与此 else 最近的并且原本没有与其对应的 else 的可能的 if 对应。

语义：

产生式 IfStatement : if ( Expression ) Statement else Statement 按照下面的过程执行 :

1.  令 exprRef 为解释执行 Expression 的结果 .
2.  如果 ToBoolean(GetValue(exprRef)) is true ，then
    1.  返回解释执行 the 的结果 first Statement.
3.  Else,
    1.  返回解释执行 the 的结果 second Statement.

产生式 IfStatement : if ( Expression ) Statement 按照下面的过程执行 :

1.  令 exprRef 为解释执行 Expression 的结果 .
2.  如果 ToBoolean(GetValue(exprRef)) is false ，return (normal, empty, empty).
3.  返回解释执行 Statement 的结果 .

## 迭代语句

语法：

`IterationStatement : do Statement while ( Expression ); while ( Expression ) Statement for ( ExpressionNoInopt; Expressionopt ; Expressionopt ) Statement for ( var VariableDeclarationListNoIn; Expressionopt ; Expressionopt ) Statement for ( LeftHandSideExpression in Expression ) Statement for ( var VariableDeclarationNoIn in Expression ) Statement`

### do-while 语句

产生式 do Statement while ( Expression ); 按照下面的过程执行 :

1.  令 V = empty。
2.  令 iterating 为 true。
3.  只要 iterating 为 true，就重复
    1.  令 stmt 为解释执行 Statement 的结果。
    2.  如果 stmt.value 不是 empty，令 V = stmt.value。
    3.  如果 stmt.type 不是 continue || stmt.target 不在当前标签组，则
        1.  如果 stmt.type 是 break 并且 stmt.target 在当前标签组内，返回 (normal, V, empty)。
        2.  如果 stmt 是个 非常规完结 ，返回 stmt。
    4.  令 exprRef 为解释执行 Expression 的结果。
    5.  如果 ToBoolean(GetValue(exprRef)) 是 false，设定 iterating 为 false。
4.  返回 (normal, V, empty);

### while 语句

产生式 IterationStatement : while ( Expression ) Statement 按照下面的过程执行 :

1.  令 V = empty.
2.  重复
    1.  令 exprRef 为解释执行 Expression 的结果 .
    2.  如果 ToBoolean(GetValue(exprRef)) 是 false，返回 (normal, V, empty).
    3.  令 stmt 为解释执行 Statement 的结果 .
    4.  如果 stmt.value 不是 empty，令 V = stmt.value.
    5.  如果 stmt.type 不是 continue || stmt.target 不在当前标签组内，则
        1.  如果 stmt.type 是 break 并且 stmt.target 在当前标签组内，则
            1.  返回 (normal, V, empty).
        2.  如果 stmt 是一个非常规完结，返回 stmt.

### for 语句

产生式 IterationStatement : for ( ExpressionNoIn[opt] ; Expression[opt] ; Expression[opt]) Statement 按照下面的过程执行 :

1.  如果 ExpressionNoIn 是 present，则 .
    1.  令 exprRef 为解释执行 ExpressionNoIn 的结果 .
    2.  调用 GetValue(exprRef). ( 不会用到此值。)
2.  令 V = empty.
3.  重复
    1.  如果第一个 Expression 是 present，则
        1.  令 testExprRef 为解释执行第一个 Expression 的结果 .
        2.  如果 ToBoolean(GetValue(testExprRef)) 是 false，返回 (normal, V, empty).
    2.  令 stmt 为解释执行 Statement 的结果 .
    3.  如果 stmt.value 不是 empty，令 V = stmt.value
    4.  如果 stmt.type 是 break 并且 stmt.target 在当前标签组内，返回 (normal, V, empty).
    5.  如果 stmt.type 不是 continue || stmt.target 不在当前标签组内，则
        1.  如果 stmt 是个非常规完结，返回 stmt.
    6.  如果第二个 Expression 是 present，则
        1.  令 incExprRef 为解释执行第二个 Expression 的结果 .
        2.  调用 GetValue(incExprRef). ( 不会用到此值 .)

产生式 IterationStatement : for ( var VariableDeclarationListNoIn ; Expression[opt] ; Expression[opt] ) Statement 按照下面的过程执行 :

1.  解释执行 VariableDeclarationListNoIn.
2.  令 V = empty.
3.  重复
    1.  如果第一个 Expression 是 present，则
        1.  令 testExprRef 为解释执行第一个 Expression 的结果 .
        2.  如果 ToBoolean(GetValue(testExprRef)) 是 false，则返回 (normal, V, empty).
    2.  令 stmt 为解释执行 Statement 的结果 .
    3.  如果 stmt.value 不是 empty，令 V = stmt.value.
    4.  如果 stmt.type 是 break 并且 stmt.target 在当前标签组内，返回 (normal, V, empty).
    5.  如果 stmt.type 不是 continue || stmt.target 不在当前标签组内，则
        1.  如果 stmt 是个非常规完结，返回 stmt.
    6.  如果第二个 Expression 是 present，则 .
        1.  令 incExprRef 为解释执行第二个 Expression 的结果 .
        2.  调用 GetValue(incExprRef). ( 不会用到此值 .)

### for-in 语句

产生式 IterationStatement : for ( LeftHandSideExpression in Expression ) Statement 按照下面的过程执行 :

1.  令 exprRef 为解释执行 Expression 的结果 .
2.  令 experValue 为 GetValue(exprRef).
3.  如果 experValue 是 null 或 undefined，返回 (normal, empty, empty).
4.  令 obj 为 ToObject(experValue).
5.  令 V = empty.
6.  重复
    1.  令 P 为 obj 的下一个 [[Enumerable]] 特性为 true 的属性的名。如果不存在这样的属性，返回 (normal, V, empty).
    2.  令 lhsRef 为解释执行 LeftHandSideExpression 的结果 ( 它可能解释执行多次 ).
    3.  调用 PutValue(lhsRef, P).
    4.  令 stmt 为解释执行 Statement 的结果 .
    5.  如果 stmt.value 不是 empty，令 V = stmt.value.
    6.  如果 stmt.type 是 break 并且 stmt.target 在当前标签组内，返回 (normal, V, empty).
    7.  如果 stmt.type 不是 continue || stmt.target 不在当前标签组内，则
        1.  如果 stmt 是非常规完结，返回 stmt.

产生式 IterationStatement : for ( var VariableDeclarationNoIn in Expression ) Statement 按照下面的过程执行 :

1.  令 varName 为解释执行 VariableDeclarationNoIn 的结果 .
2.  令 exprRef 为解释执行 Expression 的结果 .
3.  令 experValue 为 GetValue(exprRef).
4.  如果 experValue 是 null 或 undefined，返回 (normal, empty, empty).
5.  令 obj 为 ToObject(experValue).
6.  令 V = empty.
7.  重复
    1.  令 P 为 obj 的下一个 [[Enumerable]] 特性为 true 的属性的名。如果不存在这样的属性，返回 (normal, V, empty).
    2.  令 varRef 为解释执行 varName 的结果，仿佛它是个标示符引用 (11.1.2); 它可能解释执行多次 .
    3.  调用 PutValue(varRef, P).
    4.  令 stmt 为解释执行 Statement 的结果 .
    5.  如果 stmt.value 不是 empty，令 V = stmt.value.
    6.  如果 stmt.type 是 break 并且 stmt.target 在当前标签组内，返回 (normal, V, empty).
    7.  如果 stmt.type 不是 continue || stmt.target 不在当前标签组内，则
        1.  如果 stmt 是非常规完结，返回 stmt.

枚举的属性 ( 第一个算法中的步骤 6.a，第二个算法中的步骤 7.a) 的机制和顺序并没有指定。在枚举过程中枚举的对象属性可能被删除。如果在枚举过程中，删除了还没有被访问到的属性，那么它将不会被访问到。如果在枚举过程中添加新属性到列举的对象，新增加的属性也无法保证被当前执行中的枚举访问到。在任何枚举中对同一个属性名称的访问不得超过一次。

枚举一个对象的属性包括枚举其原型的属性，还有原型的原型，等等，递归；但是如果原型的一个属性是“阴影里的”那么不会枚举这个属性，因为原型链中之前某个对象有同名的属性了。当一个原型对象的一个属性是在原型链中之前对象的阴影里的，那么在决定是否枚举时不需要考虑 [[Enumerable]] 特性的值。

见 注 11.13.1。

## continue 语句

语法：

`ContinueStatement : continue ; continue [ 此处无换行 ] Identifier ;`

语义：

如果以下任意一个为真，那么程序被认为是语法错误的：

*   程序包含一个不带可选的 Identifier 的 continue 语句，没有直接或间接 ( 不跨越函数边界 ) 的嵌套在 IterationStatement 里。
*   程序包含一个有可选的 Identifier 的 continue 语句，这个 Identifier 没有出现在 IterationStatement 中闭合标签组里 ( 不跨越函数边界 )。

一个没有 Identifier 的 ContinueStatement 按照下面的过程执行 :

1.  返回 (continue, empty, empty).

一个有可选的 Identifier 的 ContinueStatement 按照下面的过程执行 :

1.  返回 (continue, empty, Identifier).

## break 语句

语法：

`BreakStatement : break ; break [ 此处无换行 ] Identifier ;`

语义：

如果以下任意一个为真，那么程序被认为是语法错误的：

*   程序包含一个不带可选的 Identifier 的 break 语句，没有直接或间接 ( 不跨越函数边界 ) 的嵌套在 IterationStatement 或 SwitchStatement 里。
*   程序包含一个有可选的 Identifier 的 break 语句，这个 Identifier 没有出现在 Statement 中闭合标签组里 ( 不跨越函数边界 )。

一个没有 Identifier 的 BreakStatement 按照下面的过程执行 :

1.  返回 (continue, empty, empty).

一个有可选的 Identifier 的 BreakStatement 按照下面的过程执行 :

1.  返回 (continue, empty, Identifier).

## return 语句

语法：

`ReturnStatement : return ; return [ 此处无换行 ] Expression ;`

语义：

在一个 ECMAScript 程序中包含的 return 语句没有在 FunctionBody 里面，那么就是语法错误的。一个 return 语句导致函数停止执行，并返回一个值给调用者。如果省略 Expression，返回值是 undefined。否则，返回值是 Expression 的值。

产生式 ReturnStatement :' return' [no LineTerminator here] Expression[opt] ; 按照下面的过程执行 :

1.  如果 Expression 不是 present，返回 (return, undefined, empty).
2.  令 exprRef 为解释执行 Expression 的结果 .
3.  返回 (return, GetValue(exprRef), empty).

## with 语句

语法：

`WithStatement : with ( Expression ) Statement`

with 语句为计算对象给当前执行环境的 词法环境 添加一个对象 环境记录项 。然后，用这个增强的 词法环境 执行一个语句。最后，恢复到原来的 词法环境 。

语义 :

产生式 WithStatement : with ( Expression ) Statement 按照下面的过程执行 :

1.  令 val 为解释执行 Expression 的结果 .
2.  令 obj 为 ToObject(GetValue(val)).
3.  令 oldEnv 为运行中的执行环境的 LexicalEnvironment.
4.  令 newEnv 为以 obj 和 oldEnv 为参数调用 NewObjectEnvironment 的结果。
5.  设定 newEnv 的 provideThis 标志为 true。
6.  设定运行中的执行环境的 LexicalEnvironment 为 newEnv.
7.  令 C 为解释执行 Statement 的结果，但如果解释执行是由异常抛出，则令 C 为 (throw, V, empty)，这里的 V 是异常。( 现在继续执行，仿佛没有抛出异常。)
8.  设定运行中的执行环境的 LexicalEnvironment 为 oldEnv.
9.  返回 C.

无论控制是从嵌入的 Statement 怎样离开的，不论是正常离开还是以 非常规完结 或异常，LexicalEnvironment 总是恢复到它之前的状态。

### 严格模式的限制

严格模式代码中不能包含 WithStatement。出现 WithStatement 的上下文被当作一个 SyntaxError。

## switch 语句

语法：

`SwitchStatement : switch ( Expression ) CaseBlock` `CaseBlock : { CaseClausesopt } { CaseClausesoptDefaultClause CaseClausesopt }``CaseClauses : CaseClause CaseClauses CaseClause` `CaseClause : case Expression : StatementListopt` `DefaultClause : default : StatementListopt`

语义：

产生式 SwitchStatement : switch ( Expression ) CaseBlock 按照下面的过程执行 :

1.  令 exprRef 为解释执行 Expression 的结果 .
2.  令 R 为以 GetValue(exprRef) 作为参数解释执行 CaseBlock 的结果。
3.  如果 R.type 是 break 并且 R.target 在当前标签组内，返回 (normal, R.value, empty).
4.  返回 R.

产生式 CaseBlock : { CaseClauses[opt] } 以一个给定输入参数 input, 按照下面的过程执行 :

1.  令 V = empty.
2.  令 A 为以源代码中顺序排列的 CaseClause 列表。
3.  令 searching 为 true.
4.  只要 searching 为 true，就重复
    1.  令 C 为 A 里的下一个 CaseClause。 如果没有 CaseClause 了，返回 (normal, V, empty).
    2.  令 clauseSelector 为解释执行 C 的结果 .
    3.  如果 input 和 clauseSelector 是 === 操作符定义的相等，则
        1.  设定 searching 为 false.
        2.  如果 C 有一个 StatementList, 则
            1.  令 R 为解释执行 C 的 StatementList 的结果。
            2.  如果 R 是个非常规完结 , 则返回 R。
            3.  令 V =R.value
5.  重复
    1.  令 C 为 A 里的下一个 CaseClause。 如果没有 CaseClause 了，返回 (normal, V, empty).
    2.  如果 C 有一个 StatementList, 则
        1.  令 R 为解释执行 C 的 StatementList 的结果。
        2.  如果 R.value 不是 empty, 则令 V =R.value.
        3.  如果 R 是个非常规完结 , 则返回 (R.type,V,R.target).

产生式 CaseBlock : { CaseClauses[opt]DefaultClause CaseClauses[opt] } 以一个给定输入参数 input, 按照下面的过程执行 :

1.  令 V = empty.
2.  令 A 为第一个 CaseClauses 中以源代码中顺序排列的 CaseClause 列表。
3.  令 B 为第二个 CaseClauses 中以源代码中顺序排列的 CaseClause 列表。
4.  令 found 为 false.
5.  重复，使 C 为 A 中的依次每个 CaseClause。
    1.  如果 found 是 false，则
        1.  令 clauseSelector 为解释执行 C 的结果 .
        2.  如果 input 和 clauseSelector 是 === 操作符定义的相等，则设定 found 为 true.
    2.  如果 found 是 true，则
        1.  如果 C 有一个 StatementList，则
            1.  令 R 为解释执行 C 的 StatementList 的结果。
            2.  如果 R.value 不是 empty, 则令 V =R.value.
            3.  R 是个非常规完结 , 则返回 (R.type,V,R.target).
6.  令 foundInB 为 false.
7.  如果 found 是 false，则
    1.  只要 foundInB 为 false 并且所有 B 中的元素都没有被处理，就重复
        1.  令 C 为 B 里的下一个 CaseClause.
        2.  令 clauseSelector 为解释执行 C 的结果 .
        3.  如果 input 和 clauseSelector 是 === 操作符定义的相等，则
            1.  设定 foundInB 为 true.
            2.  如果 C 有一个 StatementList, 则
                1.  令 R 为解释执行 C 的 StatementList 的结果。
                2.  如果 R.value 不是 empty, 则令 V =R.value.
                3.  R 是个非常规完结 , 则返回 (R.type,V,R.target).
8.  如果 foundInB 是 false 并且 DefaultClause 有个 StatementList, 则
    1.  令 R 为解释执行 DefaultClause 的 StatementList 的结果 .
    2.  如果 R.value 不是 empty, 则令 V = R.value.
    3.  如果 R 是个非常规完结 , 则返回 (R.type, V, R.target).
9.  重复 ( 注 : 如果已执行步骤 7.a.i, 此循环不从 B 的开头开始。)
    1.  令 C 为 B 的下一个 CaseClause。如果没有 CaseClause 了 , 返回 (normal, V, empty).
    2.  如果 C 有个 StatementList, 则
        1.  令 R 为解释执行 C 的 StatementList 的结果。
        2.  如果 R.value 不是 empty, 则令 V = R.value.
        3.  如果 R 是个非常规完结 , 则返回 (R.type, V, R.target).

产生式 CaseClause : case Expression : StatementList[opt] 按照下面的过程执行 :

1.  令 exprRef 为解释执行 Expression 的结果 .
2.  返回 GetValue(exprRef).

解释执行 CaseClause 不会运行相关的 StatementList。它只简单的解释执行 Expression 并返回值，这里的 CaseBlock 算法用于确定 StatementList 开始执行。

## 标签语句

语法：

`LabelledStatement : Identifier : Statement`

语义：

一个 Statement 可以由一个标签作为前缀。标签语句仅与标签化的 break 和 continue 语句一起使用。ECMAScript 没有 goto 语句。

如果一个 ECMAScript 程序包含有相同 Identifier 作为标签的 LabelledStatement 闭合的 LabelledStatement，那么认为它是是语法错误的 。这不适用于直接或间接嵌套在标签语句里面的 FunctionDeclaration 的 body 里出现标签的情况。

产生式 Identifier : Statement 的解释执行方式是，先添加 Identifier 到 Statement 的标签组，再解释执行 Statement。如果 LabelledStatement 自身有一个非空标签组，这些标签还是会添加到解释执行前的 Statement 的标签组里。如果 Statement 的解释执行结果是 (break, V, L)，这里的 L 等于 Identifier，则产生式的结果是 (normal, V, empty)。

在解释执行 LabelledStatement 之前，认为包含的 Statement 拥有一个空标签组，除非它是 IterationStatement 或 SwitchStatement，这种情况下认为它拥有一个包含单个元素 empty 的标签组。

## throw 语句

语法：

`ThrowStatement : throw [no LineTerminator here] Expression ;`

语义：

产生式 ThrowStatement : throw [no LineTerminator here] Expression ; 按照下面的过程执行 :

1.  令 exprRef 为解释执行 Expression 的结果 .
2.  返回 (throw, GetValue(exprRef), empty).

## try 语句

语法：

`TryStatement : try Block Catch try Block Finally try Block Catch Finally` `Catch : catch ( Identifier ) Block` `Finally : finally Block`

try 语句包裹一个可以出现特殊状况，如果运行时错误或 throw 语句的代码块。catch 子句提供了异常处理代码。如果 catch 子句捕获到一个异常，这个异常会绑定到它的 Identifier 上。

语义：

产生式 TryStatement : try Block Catch 按照下面的过程执行 :

1.  令 B 为解释执行 Block 的结果 .
2.  如果 B.type 不是 throw，返回 B.
3.  返回一参数 B 解释执行 Catch 的结果 .

产生式 TryStatement : try Block Finally 按照下面的过程执行 :

1.  令 B 为解释执行 Block 的结果 .
2.  令 F 为解释执行 Finally 的结果 .
3.  如果 F.type 是 normal，返回 B.
4.  返回 F.

产生式 TryStatement : try Block Catch Finally 按照下面的过程执行 :

1.  令 B 为解释执行 Block 的结果 .
2.  如果 B.type 是 throw，则
    1.  令 C 为以参数 B 解释执行 Catch 的结果 .
3.  否则 , B.type 不是 throw,
    1.  令 C 为 B.
4.  令 F 为解释执行 Finally 的结果 .
5.  如果 F.type 是 normal，返回 C.
6.  返回 F.

产生式 Catch : catch ( Identifier ) Block 按照下面的过程执行 :

1.  令 C 为传给这个产生式的参数 .
2.  令 oldEnv 为运行中执行环境的 LexicalEnvironment.
3.  令 catchEnv 为以 oldEnv 为参数调用 NewDeclarativeEnvironment 的结果
4.  以 Identifier 字符串值为参数调用 catchEnv 的 CreateMutableBinding 具体方法。
5.  以 Identifier, C, false 为参数调用 catchEnv 的 SetMutableBinding 具体方法。注：这种情况下最后一个参数无关紧要。
6.  设定运行中执行环境的 LexicalEnvironment 为 catchEnv.
7.  令 B 为解释执行 Block 的结果 .
8.  设定运行中执行环境的 LexicalEnvironment 为 oldEnv.
9.  返回 B.

不管控制是怎样退出 Block 的，LexicalEnvironment 总是会恢复到其之前的状态。

产生式 Finally : finally Block 按照下面的过程执行 :

1.  返回解释执行 Block 的结果 .

### 严格模式的限制

如果一个有 Catch 的 TryStatement 出现在 严格模式代码 里，并且 Catch 产生式的 Identifier 是 "eval" 或 "arguments"，那么这是个 SyntaxError

## debugger 语句

语法：

`DebuggerStatement : debugger ;`

语义：

解释执行 DebuggerStatement 产生式可允许让一个实现在调试器下运行时设置断点。如果调试器不存在或是非激活状态，这个语句没有可观测效果。

产生式 DebuggerStatement : debugger ; 按照下面的过程执行 :

1.  如果一个实现定义了可用的调试工具并且是开启的，则
    1.  执行实现定义的调试动作。
    2.  令 result 为实现定义的 Completion 值 .
2.  否则
    1.  令 result 为 (normal, empty, empty).
3.  返回 result.