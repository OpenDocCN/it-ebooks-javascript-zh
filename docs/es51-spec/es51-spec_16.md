# 程序

语法：

`Program : SourceElementsopt` `SourceElements : SourceElement SourceElements SourceElement``SourceElement : Statement FunctionDeclaration`

语义：

产生式 Program : SourceElements[opt] 依照下面的步骤来解释执行 :

1.  若 SourceElements 的指令序言 ( 参考 14.1 章 ) 中 , 包含严格模式指令 , 或者满足 10.1.1 章节所描述的任何一个条件 . 则 Program 的代码 . 就是一段严格模式代码 . 并对应性的 , 以严格模式或非严格模式 , 依照下面列出的步骤来解释执行代码 .
2.  若没有 SourceElements 部分 , 则返回 (normal, empty, empty).
3.  令 progCxt 为一个新的 , 如 10.4.1 章节所描述的 , 应用于全局代码的执行环境 .
4.  令 result 为解释执行 SourceElements 的结果 .
5.  退出 progCxt 这个执行环境 .
6.  返回 result.

本规范不会规定 , 具体如何解释执行一个 Program 以及如何处理其结果 . 其具体行为由 ECMAScript 实现 , 自行定义

产生式 SourceElements : SourceElements SourceElement 依照下面的步骤来解释执行 :

1.  令 headResult 为解释执行 SourceElements 的结果 .
2.  若 headResult 是非常规性完结的 , 返回 headResult.
3.  令 tailResult 为解释执行 SourceElement 的结果 .
4.  若 tailResult.value 为 empty, 令 V = headResult.value, 其他情况 , 另 V = tailResult.value.
5.  返回 (tailResult.type, V, tailResult.target).

产生式 : SourceElement : Statement 依照下面的步骤来解释执行 :

1.  返回解释执行 Statement 的结果 .

产生式 : SourceElement : FunctionDeclaration 依照下面的步骤来解释执行 :

1.  返回 (normal, empty, empty)

## 指令序言和严格模式指令 .

一个指令序言 , 是那些从 Program 或 FunctionBody 的首个 SourceElement 开始，到那些完全由一个字符串字面量后面跟一个分号 , 所构成的最长的 . 那一组 ExpressionStatement 序列中的每一个 . 字符串字面量后面的分号 , 可以显式的插入 , 或者借助分号自动插入机制来插入 . 一个指令序言 , 也可以是一个空的序列 .

严格模式指令是一个 "use strict" 或 'use strict' 的字符串字面量 . 一个严格模式指令中 , 不应该包含 EscapeSequence 或 LineContinuation.

一个指令序言 , 可以不仅仅包含一个严格模式指令 . 然而 , 当这种情况出现的时候 ,ECMAScript 实现 , 可以发出一个相关警告 .

指令序言包含的 ExpressionStatement 产生式们，会在解释执行包含他们的 SourceElements 产生式期间 , 被正常的解析执行 . ECMAScript 实现 , 可以在一个指令序言中定义其他非严格模式指令 . 当一个指令序言中的某个 ExpressionStatement 并不是一个严格模式指令，也不是一个被 ECMAScript 实现所定义的指令 . 且存在某种通知机制的话 . 就要借助该机制 , 发出一个警告 .