# 记法约定

## 语法和词法的文法

### 上下文无关文法

一个 上下文无关文法 由一定数量的 产生式 (productions) 组成。每个产生式的 左边 (left-hand side) 是一个被称为非终结符 (nonterminal) 的抽象符号， 右边 (right-hand side) 是零或多个非终结符和 终结符 (terminal symbols) 的有序排列。任何文法，它的终结符都来自指定的字母集。

当从一个叫做 目标符 (goal symbol) 的特殊非终端符组成的句子起始，那么给出的上下文无关文法就表示 语言 (language)，即，将产生式右边序列的非终结符当作左边，进行反复替换的结果就成为可能的终结符序列集合（可能无限）。

### 词法和正则的文法

第七章给出了 ECMAScript 的 词法文法 (lexical grammar)。作为此文法的终结符字符（Unicode 代码单元）符合第六章定义的 SourceCharacter 的规则。它定义了一套产生式，从目标符 InputElementDiv 或 InputElementRegExp 起始，描述了如何将这样的字符序列翻译成一个输入元素序列。

空白和注释之外的输入元素构成 ECMAScript 语法文法的终结符，它们被称为 ECMAScript 的 tokens。这些 tokens 是，ECMAScript 语言的保留字，标识符，字面量，标点符号。此外，行结束符虽然不被视为 tokens，但会成为输入元素流的一部分，用于引导处理自动插入分号（ 7.9 ）。空白和单行注释会被简单的丢弃，不会出现在语法文法的输入元素的流中。如果一个 多行注释 (MultiLineComment)（即形式为“/ ... /”的注释，不管是否跨越多行）不包含行结束符也会简单地丢弃，但如果一个 多行注释 包含一个或多个结束符，那么，注释会被替换为一个行结束符，成为语法文法输入元素流的一部分。

15.10 给出了 ECMAScript 的 正则文法 (RegExp grammar)。此文法的终结符字符也由 SourceCharacter 定义。它定义了一套产生式，从目标符 Pattern 起始，描述了如何将这样的字符序列翻译成一个正则表达式模式。

两个冒号“::”作为分隔符分割词法和正则的文法产生式。词法和正则的文法共享某些产生式。

### 数字字符串文法

用于转换字符串为数字值的一种文法。此文法与词法文法的一部分（与数字字面量有关的）类似，并且有终结符 SourceCharacter。此文法出现在 9.3.1 。

三个冒号“:::”作为分隔符分割数字字符串文法的产生式。

### 语法文法

第 11，12，13，14 章给出了 ECMAScript 的 语法文法 。词法文法定义的 ECMAScript tokens 是此文法的终结符（ 5.1.2 ）。它定义了一组起始于 Program 目标符的产生式，描述了语法正确的 ECMAScript 程序应该怎样排列 tokens。

当一个字符流被解析为 ECMAScript 程序，它首先通过词法文法应用程序反复转换为一个输入元素流；然后再用一个语法文法应用程序解析这个输入元素流。当输入元素流没有更多 tokens 时，如果 tokens 不能解析为 Program 目标非终结符的单一实例，那么程序在语法上存在错误。

只用一个冒号“:”作为分隔符分割语法词法的产生式。

事实上第 11，12，13 和 14 章给出的语法语法，并不能完全说明一个正确的 ECMAScript 程序能接受的 token 序列。一些额外的 token 序列也被接受，即某些特殊位置（如行结束符前）加入分号可以被文法接受。此外，文法描述的某些 token 序列不被文法接受，如一个行结束符出现在“尴尬”的位置。

### JSON 文法

JSON 文法用于将描述 ECMAScript 对象的字符串转换为实际的对象。15.12.1 给出了 JSON 文法 。

JSON 文法由 JSON 词法文法和 JSON 语法文法组成。JSON 词法文法用于将字符序列转换为 tokens，类似 ECMAScript 词法文法。JSON 语法文法说明 JSON 词法文法给出怎样的 tokens 序列才能转换为语法上是正确的 JSON 对象。

两个冒号“::”作为分隔符分割 JSON 词法文法的产生式。JSON 词法文法使用某些 ECMAScript 词法文法的产生式。JSON 语法文法与 ECMAScript 语法文法类似。JSON 语法文法产生式被一个冒号“:”作为分隔符分割。

### 文法标记法

词法文法和字符串文法的终结符，以及一些语法文法的终结符，无论是在文法的产生式还是贯穿本规范的所有文本直接给出的终结符，都用 等宽 (fixed width) 字体显示。他们表示程序书写正确。所有以这种方式指定的终结符字符，可以理解为 Unicode 字符的完整的 ASCII 范围，不是任何其他类似的 Unicode 范围字符。

非终结符以 斜体 (italic) 显示。一个非终结符的定义由非终结符名称和其后定义的一个或多个冒号给出。（冒号的数量表示产生式所属的文法。）非终结符的右侧有一个或多个替代子紧跟在下一行。 例如，语法定义：

WhileStatement

while ( Expression ) Statement

表示这个非终结符 WhileStatement 代表 while token，其后跟左括号 token，其后跟 Expression，其后跟右括号 token，其后跟 Statement。这里出现的 Expression 和 Statement 本身是非终结符。另一个例子，语法定义：

`ArgumentList : AssignmentExpression ArgumentList , AssignmentExpression`

表示这个 ArgumentList 可以代表一个 AssignmentExpression，或 ArgumentList，其后跟一个逗号，其后跟一个 AssignmentExpression。这个 ArgumentList 的定义是递归的，也就是说，它定义它自身。其结果是，一个 ArgumentList 可能包含用逗号隔开的任意正数个参数，每个参数表达式是一个 AssignmentExpression。这样，非终结符共用了递归的定义。

终结符或非终结符可能会出现后缀下标“ [opt] ”，表示它是可选符号。实际上包含可选符号的替代子包含两个右边，一个是省略可选元素的，另一个是包含可选元素的。这意味着：

`VariableDeclaration : Identifier Initialiseropt`

是以下的一种缩写：

`VariableDeclaration : Identifier Identifier Initialiser`

并且：

`IterationStatement : for ( ExpressionNoInopt ; Expressionopt ; Expressionopt ) Statement`

是以下的一种缩写：

`IterationStatement : for ( ; Expressionopt ; Expressionopt ) Statement for ( ExpressionNoIn ; Expressionopt ; Expressionopt ) Statement`

是以下的一种缩写 :

`IterationStatement : for ( ; ; Expressionopt ) Statement for ( ; Expression ; Expressionopt) Statement for ( ExpressionNoIn ; ; Expressionopt) Statement for ( ExpressionNoIn ; Expression ; Expressionopt) Statement`

是以下的一种缩写：

`IterationStatement : for ( ; ; ) Statement for ( ; ; Expression ) Statement for ( ; Expression ; ) Statement for ( ; Expression ; Expression ) Statement for ( ExpressionNoIn ; ; ) Statement for ( ExpressionNoIn ; ; Expression ) Statement for ( ExpressionNoIn ; Expression ; ) Statement for ( ExpressionNoIn ; Expression ; Expression ) Statement`

因此，非终结 IterationStatement 实际上有 8 个右侧变体。

如果文法定义的冒号后面出现文字“one of”，那么其后一行或多行出现的每个终结符都是一个选择定义。例如，ECMAScript 包含的词法文法生产器：

`NonZeroDigit :: one of 1 2 3 4 5 6 7 8 9`

这仅仅下面写法的一种缩写：

`NonZeroDigit :: 1 2 3 4 5 6 7 8 9`

如果产生式的右侧是出现“[empty]”，它表明，产生式的右侧不包含终结符或非终结符。

如果产生式的右侧出现“[lookahead ? set]”，它表明，给定 set 的成员不得成为产生式紧随其后的 token。这个 set 可以写成一个大括号括起来的终结符列表。为方便起见，set 也可以写成一个非终结符，在这种情况下，它代表了这个非终结符 set 可扩展所有终结符。例如，给出定义

`DecimalDigit :: one of 0 1 2 3 4 5 6 7 8 9` `DecimalDigits :: DecimalDigit DecimalDigits DecimalDigit`

在定义

`LookaheadExample :: n [lookahead ? {1 , 3 , 5 , 7 , 9}]DecimalDigits DecimalDigit [lookahead ? DecimalDigit ]`

能匹配字母 n 后跟随由偶数起始的一个或多个十进制数字，或一个十进制数字后面跟随一个非十进制数字。

如果产生式的右侧出现“[no LineTerminator here]”，那么它表示此产生式是个受限的产生式：如果 LineTerminator 在输入流的指定位置出现，那么此产生式将不会被适用。例如，产生式：

`ThrowStatement : throw [no LineTerminator here] Expression ;`

表示如果程序中 return token 和 Expression 之间的出现 LineTerminator，那么不得使用此产生式。

LineTerminator 除了禁止出现在受限的产生式，可以在输入元素流的任何两个 tokens 之间出现任意次数，而不会影响程序的语法验证。

当一个词法文法产生式或数字字符串文法中出现多字符 token，它表示此字符序列将注册一个 token。

使用词组“but not“可以指定某些不允许在产生式右侧的扩展，它说明排除这个扩展。例如，产生式：

`Identifier :: IdentifierName but not ReservedWord`

此非终结符 Identifier 可以由可替换成 IdentifierName 的字符序列替换，相同的字符序列不能替换 ReservedWord。

最后，对于实际上不可能列出全部可变元的少量非终结符，我们用普通字体写出描述性的短语来描述它们：

`SourceCharacter :: any Unicode code unit`

## 算法约定

此规范通常使用带编号的列表来指定算法的步骤。这些算法是用来精确地指定 ECMAScript 语言结构所需的语义。该算法无意暗示任何具体实现使用的技术。在实践中，也许可用更有效的算法实现一个给定功能。

为了方便其使用本规范的多个部分，叫做 抽象操作 (abstract operations) 的一些算法编写成带名称的可传参函数化形式，所以在其他算法里可以通过名称引用它们。

当一个算法产生返回值 ，“return x” 指令说明该算法的返回值是 x，并且算法应该终止。“第 n 步的结果”的简写是 Result(n) 。

为了表达清晰，算法的步骤可细分为有序的子步骤。子步骤被缩进，可以将自身进一步划分为缩进子步骤。大纲编号约定用于识别分步骤，第一层次的子步骤适用小写字母标记，第二层次的子步骤使用小写罗马数字标记。如果需要超过三个层次，则重复这些规则，第四层次使用数字标记。例如 :

1.  Top-level step
    1.  Substep.
    2.  Substep
        1.  Subsubstep.
        2.  Subsubstep.
            1.  Subsubsubstep
                1.  Subsubsubsubstep

步骤或子步骤可写“if”谓词作为它的子步骤的条件。在这种情况下，当谓词为真时子步骤才适用。如果一个步骤或子步骤由单词“else”开始，那么它是一个谓词，否定前面的同一层级的“if”谓词。

步骤可以表示其子步骤的迭代应用可能指定其子步的迭代应用程序。

步骤可能断言其算法中的某一不变条件。这样的断言可以让算法中隐含的不变条件变成显式的。这种断言不会添加额外的语义要求，实现没有一定去检查的必要性。它们只是用来让算法更清晰。

数学运算，如加法，减法，取反，乘法，除法，还有稍后在本节中定义的数学函数应该总是被理解为对数学实数计算精确的数学结果，其中不包括无穷大，不包括负零区别于正零。本标准中的浮点运算算法模型，包括明确的步骤，在必要情况下处理无穷大和有符号零和执行四舍五入。如果一个数学运算或函数应用一个浮点数，它应该被应用为代表此浮点数的确切的数学值，一个浮点数必须是有限的 ，如果是 +0 或 -0 ，则相应的数学值就是 0。

数学函数 abs(x) 产生 x 的绝对值，如果 x 是负数（小于零），它是这是 - x，否则是 X 本身。

如果 x 是正数，数学函数 sign (x) 产生 1，如果 x 是负数产生 - 1。此标准中 x 为零的情况下不使用 sign 函数。

符号 “x modulo y” (y 必须有限且非零 ) 计算一个满足以下条件的 k 值 ，与 y 同号 ( 或是零 ) ，abs(k) < abs(y) ，对一些整数 q 满足 x?k = q × y。

数学函数 floor(x) 产生不大于 x 的最大整数（最大可为正无穷）。

floor(x) = x?(x modulo 1).

如果算法定义“抛出一个异常”，算法的执行将被终止，且没有返回结果。已调用的算法也被终止，直到算法步骤使用术语“如果一个异常被抛出 ...”明确指出异常处理。一旦遇到这种算法步骤，异常将不再被视已发生过。