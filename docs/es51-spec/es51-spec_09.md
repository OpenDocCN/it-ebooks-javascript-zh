# 词法

ECMAScript 程序的源文本首先转换成一个输入元素序列；tokens，行终结符，注释，空白构成输入元素序列。从左到右扫描源文本，反复获取作为下一个输入元素的尽可能长的字符序列。

词法文法有两个目标符。InputElementDiv 目标符用在允许除法 (/) 或除赋值 (/=) 运算符开始的语法文法上下文中。InputElementRegExp 目标符用在其他语法文法上下文。

没有允许除法或除赋值运算符开头，同时又允许 RegularExpressionLiteral 开头的语法文法上下文。这不会被分号插入（见 7.9）影响；如下面的例子：

`a = b /hi/g.exec(c).map(d);`

其中 LineTerminator 后的第一个非空白，非注释字符是斜线（/），并且这个语法上下文允许除法或除赋值运算符，所以不会在这个 LineTerminator 位置插入分号。也就是说，上面的例子解释为：

`a = b / hi / g.exec(c).map(d);`

语法：

`InputElementDiv :: WhiteSpace LineTerminator Comment Token DivPunctuator``InputElementRegExp :: WhiteSpace LineTerminator Comment Token RegularExpressionLiteral`

## Unicode 格式控制字符

Unicode 格式控制字符（即，Unicode 字符数据库中“Cf”分类里的字符，如“左至右符号 (left-to-right mark)”或“右至左符号 (left-to-right mark)”）是用来控制被更高层级协议（如标记语言）忽略的文本范围的格式的控制代码。

允许在源文本中出现控制字符是有用的，以方便编辑和显示。所有格式控制字符可写入到注释，字符串字面量，正则表达式字面量中。

在某些语言中和控制字符用于创建必要的的分隔符分割词或短语。在 ECMAScript 源文本里，和还可以用在一个标识符后的第一个字符。

控制字符主要出现的文本的开头，标记它是 Unicode，并允许检测文本的编码和字节顺序。用于这一目的字符，有时也可能出现在文本开始的后面，例如，一个合并的文件。字符被视为空白字符（见 [7.2]）。

表 1 总结了一些在注释，字符串字面量，正则表达式字面量之外被特殊对待的格式控制字符。

控制字符的使用

| 字符编码值 | 名称 | 正式名称 | 用途 |
| \u200C | 零宽非连接符 | <ZWNJ> | IdentifierPart |
| \u200D | 零宽连接符 | <ZWJ> | IdentifierPart |
| \uFEFF | 位序掩码 | <BOM> | Whitespace |

## 空白字符

空白字符用来改善源文本的可读性和分割 tokens（不可分割的词法单位），此外就无关紧要。空白字符可以出现的两个 token 之间还可以出现在输入的开始或结束位置。空白字符，还可以出现在字符串 字面量 (StringLiteral) 或正则 表达式字面量 (RegularExpressionLiteral)( 在这里它表示组成字面量的字符 ) 或 注释 (Comment) 中，但是不能出现的其他任何 token 内。

表 2 中列出了 ECMAScript 空白字符。

空白字符

| 字符编码值 | 名称 | 正式名称 |
| \u0009 | 制表符 | <TAB> |
| \u000B | 纵向制表符 | <VT> |
| \u000C | 进纸符 | <FF> |
| \u0020 | 空格 | <SP> |
| \u00A0 | 非断空格 | <NBSP> |
| \uFEFF | 位序掩码 | <BOM> |
| 其它分类“Zs” | 其它任何 Unicode"空白分隔符" | <USP> |

ECMAScript 实现必须认可 Unicode 3.0 中定义的所有空白字符。后续版本的 Unicode 标准可能定义其他空白字符。ECMAScript 实现可以认可更高版本 Unicode 标准里的空白字符。

语法：

`WhiteSpace :: <tab> <vt> <ff> <sp> <nbsp> <bom> <usp>`

## 行终结符

像空白字符一样，行终止字符用于改善源文本的可读性和分割 tokens（不可分割的词法单位）。然而，不像空白字符，行终结符对语法文法的行为有一定的影响。一般情况下，行终结符可以出现在任何两个 token 之间，但也有少数地方，语法文法禁止这样做。行终结符也影响自动插入分号过程（7.9）。行终结符不能出现在 StringLiteral 之外的任何 token 内。行终结符只能出现在作为 LineContinuation 一部分的 StringLiteral token 里。

行终结符可以出现在 MultiLineComment（7.4）内，但不能出现在 SingleLineComment 内。

正则表达式的 \s 类匹配的空白字符集中包含行终结符。

表 3 列出了 ECMAScript 的行终止字符。

行终止字符

| 字符编码值 | 名称 | 正式名称 |
| \u000A | 进行符 | <LF> |
| \u000D | 回车符 | <CR> |
| \u2028 | 行分隔符 | <LS> |
| \u2029 | 段分隔符 | <PS> |

只有表 3 中的字符才被视为行终结符。其他新行或折行字符被视为空白，但不作为行终结符。字符序列 <cr><lf>作一个行终结符。计算行数时它应该被视为一个字符。</lf></cr>

语法：

`LineTerminator ::` `LineTerminatorSequence :: [lookahead ? ]`

## 注释

注释可以是单行或多行。多行注释不能嵌套。

因为单行注释可以包含除了 LineTerminator 字符之外的任何字符，又因为有一般规则：一个 token 总是尽可能匹配更长，所以一个单行注释总是包含从 // 到行终结符之间的所有字符。然而，在该行末尾的 LineTerminator 不被看成是单行注释的一部分，它被词法文法识别成语法文法输入元素流的一部分。这一点非常重要，因为这意味着是否存在单行注释都不影响自动分号插入进程（见 7.9）。

像空白一样，注释会被语法文法简单丢弃，除了 MultiLineComment 包含行终结符字符的情况，这种情况下整个注释会当作一个 LineTerminator 提供给语法文法解析。

语法：

`Comment :: MultiLineComment SingleLineComment` `MultiLineComment :: /* MultiLineCommentCharsopt*/` `MultiLineCommentChars :: MultiLineNotAsteriskChar MultiLineCommentCharsopt * PostAsteriskCommentCharsopt``PostAsteriskCommentChars :: MultiLineNotForwardSlashOrAsteriskChar MultiLineCommentCharsopt * PostAsteriskCommentCharsopt` `MultiLineNotAsteriskChar :: SourceCharacter but not asterisk *` `MultiLineNotForwardSlashOrAsteriskChar :: SourceCharacter but not forward-slash / or asterisk *` `SingleLineComment :: // SingleLineCommentCharsopt` `SingleLineCommentChars :: SingleLineCommentChar SingleLineCommentCharsopt` `SingleLineCommentChar :: SourceCharacter but not LineTerminator`

## Tokens

语法：

`Token :: IdentifierName Punctuator NumericLiteral StringLiteral`

DivPunctuator 和 RegularExpressionLiteral 产生式定义 tokens ，但 Token 的产生式不包含它们。

## 标识符名和标识符

标识符名是 tokens，Unicode 标准第五章的“标识符”节给出的文法加入了一些小的修改来解释它。Identifier 是一个 IdentifierName 但不是一个 ReservedWord( 见 7.6.1)。Unicode 标识符文法基于 Unicode 标准指出的 normative 和 informative 字符分类。所有符合 ECMAScript 的实现必须能够正确处理 Unicode 标准 3.0 版本中指定的分类里的字符的分类。

本标准增加了个别字符：在 IdentifierName 的任何位置允许出现美元符（$）和下划线（_）。

IdentifierName 还允许出现 Unicode 转义序列，它们被 UnicodeEscapeSequence 的 CV 计算成单个字符贡献给 IdentifierName（见 7.8.4）。UnicodeEscapeSequence 前面的 \ 不给 IdentifierName 贡献字符。UnicodeEscapeSequence 不能提供单个字符给将要成为非法字符的 IdentifierName。换句话说，如果一个 \ UnicodeEscapeSequence 序列被 UnicodeEscapeSequence 的 CV 替换，结果必须仍是有效的包含与原 IdentifierName 精确相同字符序列的 IdentifierName。本规范说明的所有标识符是根据它的实际字符，不管转义序列贡献特定字符与否。

根据 Unicode 标准两个规范的 IdentifierName 相等，是说除非他们的代码单元序列准确相等，否则不同（换句话说，符合 ECMAScript 的实现只需要按位比较 IdentifierName 值）。其目的是为了传入编译器之前就把源文本转换为正常化形式 C。

ECMAScript 实现可以识别后续版本 Unicode 标准定义的标识符字符。如果考虑可移植性，程序员应该只采用 Unicode 3.0 中定义的标识符字符。

语法：

`Identifier :: IdentifierName but not ReservedWord` `IdentifierName :: IdentifierStart IdentifierName IdentifierPart``IdentifierStart :: UnicodeLetter $ _ \ UnicodeEscapeSequence``IdentifierPart :: IdentifierStart UnicodeCombiningMark UnicodeDigit UnicodeConnectorPunctuation` `UnicodeLetter any character in the Unicode categories “Uppercase letter (Lu)”, “Lowercase letter (Ll)”, “Titlecase letter (Lt)”, “Modifier letter (Lm)”, “Other letter (Lo)”,or “Letter number (Nl)”.` `UnicodeCombiningMark any character in the Unicode categories “Non-spacing mark (Mn)”\\\ or “Combining spacing mark (Mc)”` `UnicodeDigit any character in the Unicode category “Decimal number (Nd)”` `UnicodeConnectorPunctuation any character in the Unicode category “Connector punctuation (Pc)”` `UnicodeEscapeSequence see 7.8.4.`

### 保留字

保留字不能作为 Identifier 的 IdentifierName。

语法

`ReservedWord :: Keyword FutureReservedWord NullLiteral BooleanLiteral`

#### 关键词

下列 token 是 ECMAScript 的关键词，不能用作 ECMAScript 程序的 Identifiers。

语法

`Keyword :: one of break do instanceof typeof case else new var catch finally return void continue for switch while debugger function this with default if throw delete in try`

#### 未来保留字

下列词被用作建议扩展关键字，因此保留，以便未来可能采用这些扩展。

语法

`FutureReservedWord :: one of class enum extends super const export import`

当下列 tokens 出现在 严格模式代码 (strict mode code )（见 10.1.1）里，将被当成是 FutureReservedWords。任意这些 tokens 出现在任意上下文中的严格模式代码 (strict mode code) 中，如果 FutureReservedWord 出现的位置会产生错误，那么必须抛出对应的异常：

`implements let private public yield interface package protected static`

## 标点符号

语法

`Punctuator :: one of { } ( ) [ ] .  ; , < > <= >= ==  != ===  !== + - *  % ++ -- << >> >>> & | ^  ! ~ && ||  ?  : = += -= *=  %= <<= >>= >>>= &= |= ^=` `DivPunctuator :: one of / /=`

## 字面量

语法

`Literal :: NullLiteral BooleanLiteral NumericLiteral StringLiteral RegularExpressionLiteral`

### 空值字面量

语法：

`NullLiteral :: null`

语义：

空值字面量的值 null，是 Null 类型的唯一值。

### 布尔值字面量

语法：

`BooleanLiteral :: true false`

语义：

布尔值字面量的值 true 是个布尔类型值 ，即 true。

布尔值字面量的值 false 是个布尔类型值 ，即 false。

### 数值字面量

语法：

`NumericLiteral :: DecimalLiteral HexIntegerLiteral``DecimalLiteral :: DecimalIntegerLiteral . DecimalDigitsopt ExponentPartopt . DecimalDigits ExponentPartopt DecimalIntegerLiteral ExponentPartopt``DecimalIntegerLiteral :: 0 NonZeroDigit DecimalDigitsopt``DecimalDigits :: DecimalDigit DecimalDigits DecimalDigit` `DecimalDigit :: one of 0 1 2 3 4 5 6 7 8 9` `NonZeroDigit :: one of 1 2 3 4 5 6 7 8 9` `ExponentPart :: ExponentIndicator SignedInteger` `ExponentIndicator :: one of e E` `SignedInteger :: DecimalDigits + DecimalDigits - DecimalDigits``HexIntegerLiteral :: 0x HexDigit 0X HexDigit HexIntegerLiteral HexDigit` `HexDigit :: one of 0 1 2 3 4 5 6 7 8 9 a b c d e f A B C D E F`

源字符中的 NumericLiteral 后面不允许紧跟着 IdentifierStart 或 DecimalDigit。

例如：

`3in`

是错误的，不存在两个输入元素 3 和 in。

语义：

一个数值字面量代表一个 Number 类型的值。此值取决于两个步骤：第一，由字面量得出的数学值 (mathematical value)（MV）；第二，这个数学值按照后面描述的规则舍入。

*   NumericLiteral::DecimalLiteral 的 MV 是 DecimalLiteral 的 MV。
*   NumericLiteral :: HexIntegerLiteral 的 MV 是 HexIntegerLiteral 的 MV。
*   DecimalLiteral :: DecimalIntegerLiteral . 的 MV 是 DecimalIntegerLiteral 的 MV 是。
*   DecimalLiteral :: DecimalIntegerLiteral . DecimalDigits 的 MV 是 DecimalIntegerLiteral 的 MV 加上 (DecimalDigits 的 MV 乘 10-n), 这里的 n 是 DecimalDigits 的字符个数。
*   DecimalLiteral :: DecimalIntegerLiteral . ExponentPart 的 MV 是 DecimalIntegerLiteral 的 MV 乘 10e, 这里的 e 是 ExponentPart 的 MV。
*   DecimalLiteral :: DecimalIntegerLiteral . DecimalDigits ExponentPart 的 MV 是 (DecimalIntegerLiteral 的 MV 加 (DecimalDigits 的 MV 乘 10-n)) 乘 10e, 这里的 n 是 DecimalDigits 的字符个数，e 是 ExponentPart 的 MV。
*   DecimalLiteral ::. DecimalDigits 的 MV 是 DecimalDigits 的 MV 乘 10-n, 这里的 n 是 DecimalDigits 的字符个数。
*   DecimalLiteral ::. DecimalDigits ExponentPart 的 MV 是 DecimalDigits 的 MV 乘 10e-n, 这里的 n 是 DecimalDigits 的字符个数，e 是 ExponentPart 的 MV。
*   DecimalLiteral :: DecimalIntegerLiteral 的 MV 是 DecimalIntegerLiteral 的 MV。
*   DecimalLiteral :: DecimalIntegerLiteral ExponentPart 的 MV 是 DecimalIntegerLiteral 的 MV 乘 10e, 这里的 e 是 ExponentPart 的 MV。
*   DecimalIntegerLiteral :: 0 的 MV 是 0。
*   DecimalIntegerLiteral :: NonZeroDigit DecimalDigits 的 MV 是 (NonZeroDigit 的 MV 乘 10n) 加 DecimalDigits 的 MV, 这里的 n 是 DecimalDigits 的字符个数。
*   DecimalDigits :: DecimalDigit 的 MV 是 DecimalDigit 的 MV。
*   DecimalDigits :: DecimalDigits DecimalDigit 的 MV 是 (DecimalDigits 的 MV 乘 10) 加 DecimalDigit 的 MV。
*   ExponentPart :: ExponentIndicator SignedInteger 的 MV 是 SignedInteger 的 MV。
*   SignedInteger :: DecimalDigits 的 MV 是 DecimalDigits 的 MV。
*   SignedInteger :: + DecimalDigits 的 MV 是 DecimalDigits 的 MV。
*   SignedInteger :: - DecimalDigits 的 MV 是 DecimalDigits 的 MV 取负。
*   DecimalDigit :: 0 或 HexDigit :: 0 的 MV 是 0。
*   DecimalDigit :: 1 或 NonZeroDigit :: 1 或 HexDigit :: 1 的 MV 是 1。
*   DecimalDigit :: 2 或 NonZeroDigit :: 2 或 HexDigit :: 2 的 MV 是 2。
*   DecimalDigit :: 3 或 NonZeroDigit :: 3 或 HexDigit :: 3 的 MV 是 3。
*   DecimalDigit :: 4 或 NonZeroDigit :: 4 或 HexDigit :: 4 的 MV 是 4。
*   DecimalDigit :: 5 或 NonZeroDigit :: 5 或 HexDigit :: 5 的 MV 是 5。
*   DecimalDigit :: 6 或 NonZeroDigit :: 6 或 HexDigit :: 6 的 MV 是 6。
*   DecimalDigit :: 7 或 NonZeroDigit :: 7 或 HexDigit :: 7 的 MV 是 7。
*   DecimalDigit :: 8 或 NonZeroDigit :: 8 或 HexDigit :: 8 的 MV 是 8。
*   DecimalDigit :: 9 或 NonZeroDigit :: 9 或 HexDigit :: 9 的 MV 是 9。
*   HexDigit :: a 或 HexDigit :: A 的 MV 是 10。
*   HexDigit :: b 或 HexDigit :: B 的 MV 是 11。
*   HexDigit :: c 或 HexDigit :: C 的 MV 是 12。
*   HexDigit :: d 或 HexDigit :: D 的 MV 是 13。
*   HexDigit :: e 或 HexDigit :: E 的 MV 是 14。
*   HexDigit :: f 或 HexDigit :: F 的 MV 是 15。
*   HexIntegerLiteral :: 0x HexDigit 的 MV 是 HexDigit 的 MV。
*   HexIntegerLiteral :: 0X HexDigit 的 MV 是 HexDigit 的 MV。
*   HexIntegerLiteral :: HexIntegerLiteral HexDigit 的 MV 是 (HexIntegerLiteral 的 MV 乘 16) 加 HexDigit 的 MV。

数值字面量的确切 MV 值一旦被确定，它就会舍入成 Number 类型的值。如果 MV 是 0，那么舍入值是 +0；否则，舍入值必须是 MV 对应的准确数字值（8.5 中定义），除非此字面量是有效数字超过 20 位的 DecimalLiteral，这种情况下，数字值可以用下面两种方式产生的 MV 值确定：一，将 20 位后的每个有效数字用 0 替换后产生的 MV，二，将 20 位后的每个有效数字用 0 替换，并且递增第 20 位有效数字位置的字面量值，产生的 MV。如果一个数字是 ExponentPart 的一部分，并且：

*   它不是 0；或
*   它的左侧是非零数字，它的右侧是不在 ExponentPart 的非零数字。

符合标准的实现，在处理严格模式代码（见 10.1.1）时，按照 B.1.1 的描述，不得扩展 NumericLiteral 包含 OctalIntegerLiteral 的语法。

### 字符串字面量

一个字符串字面量是关闭的单引号或双引号里的零个或多个字符。每个字符都可以用一个转义序列代表。除了闭合银行字符，反斜杠，回车，行分隔符，段落分隔符，换行符之外的所有字符都可以直接出现的字符串字面量里。任何字符都可以通过转移序列的形式出现。

语法

`StringLiteral :: " DoubleStringCharactersopt " ' SingleStringCharactersopt '` `DoubleStringCharacters :: DoubleStringCharacter DoubleStringCharactersopt` `SingleStringCharacters :: SingleStringCharacter SingleStringCharactersopt` `DoubleStringCharacter :: SourceCharacter but not double-quote " or backslash \ or LineTerminator \ EscapeSequence LineContinuation``SingleStringCharacter :: SourceCharacter but not single-quote ' or backslash \ or LineTerminator \ EscapeSequence LineContinuation` `LineContinuation :: \ LineTerminatorSequence` `EscapeSequence :: CharacterEscapeSequence 0 [lookahead ? DecimalDigit] HexEscapeSequence UnicodeEscapeSequence``CharacterEscapeSequence :: SingleEscapeCharacter NonEscapeCharacter` `SingleEscapeCharacter :: one of ' " \ b f n r t v` `NonEscapeCharacter :: SourceCharacter but not EscapeCharacter or LineTerminator` `EscapeCharacter :: SingleEscapeCharacter DecimalDigit x u` `HexEscapeSequence :: x HexDigit HexDigit` `UnicodeEscapeSequence :: u HexDigit HexDigit HexDigit HexDigit`

7.6 给出了 HexDigit 非终结符的定义。 第六章 定义了 SourceCharacter。

语义

一个字符串字面量代表一个 String 类型的值。字面量的字符串值 (SV) 由字符串字面量各部分贡献的字符值 (CV) 描述。作为这一过程的一部分，字符字面量里的某些字符字符会被解释成包含数学值 (MV)，如 7.8.3 和下面描述的。

*   StringLiteral :: "" 的 SV 是空字符序列。
*   StringLiteral :: 的 SV 是空字符序列。
*   StringLiteral :: " DoubleStringCharacters " 的 SV 是 DoubleStringCharacters 的 SV。
*   StringLiteral :: ' SingleStringCharacters ' 的 SV 是 SingleStringCharacters 的 SV。
*   DoubleStringCharacters :: DoubleStringCharacter 的 SV 是包含一个字符的序列，此字符的 CV 是 DoubleStringCharacter 的 CV。
*   DoubleStringCharacters :: DoubleStringCharacter DoubleStringCharacters 的 SV 是 （DoubleStringCharacter 的 CV 后面跟着 DoubleStringCharacters 的 SV 里所有字符的）序列。
*   SingleStringCharacters :: SingleStringCharacter 的 SV 是包含一个字符的序列，此字符的 CV 是 SingleStringCharacter 的 CV。
*   SingleStringCharacters :: SingleStringCharacter SingleStringCharacters 的 SV 是（SingleStringCharacter 的 CV 后面跟着 SingleStringCharacters 的 SV 里所有字符的）序列。
*   LineContinuation :: \ LineTerminatorSequence 的 SV 是空字符序列。
*   DoubleStringCharacter :: SourceCharacter but not double-quote " or backslash \ or LineTerminator 的 CV 是 SourceCharacter 字符自身。
*   DoubleStringCharacter :: \ EscapeSequence 的 CV 是 EscapeSequence 的 CV。
*   DoubleStringCharacter :: LineContinuation 的 CV 是空字符序列。
*   SingleStringCharacter :: SourceCharacter but not single-quote ' or backslash \ or LineTerminator 的 CV 是 SourceCharacter 字符自身。
*   SingleStringCharacter :: \ EscapeSequence 的 CV 是 EscapeSequence 的 CV。
*   SingleStringCharacter :: LineContinuation 的 CV 是空字符序列。
*   EscapeSequence :: CharacterEscapeSequence 的 CV 是 CharacterEscapeSequence 的 CV。
*   EscapeSequence :: 0 [lookahead ? DecimalDigit] 的 CV 是 <nul>字符（Unicode 值 0000）。</nul>
*   EscapeSequence :: HexEscapeSequence 的 CV 是 HexEscapeSequence 的 CV。
*   EscapeSequence :: UnicodeEscapeSequence 的 CV 是 UnicodeEscapeSequence 的 CV。
*   CharacterEscapeSequence ::SingleEscapeCharacter 的 CV 是表格 4 里的 SingleEscapeCharacter 确定的代码单元值字符：

    字符串单字符转义序列

    | 转义序列 | 字符编码值 | 名称 | 符号 |
    | \b | \u0008 | 回格 | <BS> |
    | \t | \u0009 | 水平制表符 | <HT> |
    | \n | \u000A | 进行（新行） | <LF> |
    | \v | \u000B | 竖直制表符 | <VT> |
    | \f | \u000C | 进纸 | <FF> |
    | \r | \u000D | 回车 | <CR> |
    | \" | \u0022 | 双引号 | " |
    | \' | \u0027 | 单引号 | ' |
    | \\ | \u005C | 反斜杠 | \ |

*   CharacterEscapeSequence :: NonEscapeCharacter 的 CV 是 NonEscapeCharacter 的 CV.
*   NonEscapeCharacter :: SourceCharacter but not EscapeCharacter or LineTerminator 的 CV 是 SourceCharacter 字符自身 .
*   HexEscapeSequence :: x HexDigit HexDigit 的 CV 是 ((16 乘第一个 HexDigit 的 MV) 加第二个 HexDigit 的 MV) 代码单元确定的字符。
*   UnicodeEscapeSequence :: u HexDigit HexDigit HexDigit HexDigit 的 CV 是 (4096 乘第一个 HexDigit 的 MV) 加 (256 乘第二个 HexDigit 的 MV) 加 (16 乘第三个 HexDigit 的 MV) 加 ( 第四个 HexDigit 的 MV) 代码单元确定的字符。

符合标准的实现，在处理严格模式代码（见 10.1.1）时，按照 B.1.2 的描述，不得扩展 EscapeSequence 包含 OctalEscapeSequence 的语法。

行终结符不能出现在字符串字面量里，除非它成为 LineContinuation 的一部分产生空字符序列。让字符串字面量的字符串值包含行终结符的正确方法是使用转义序列，如 \n 或 \u000A。

### 正则表达式字面量

正则表达式字面量是输入元素，每当字面量被评估时会转换为 RegExp 对象（见 15.10）。当一个程序中有两个正则表达式字面量评估成正则表达式对象，不能用 === 比较他们是否相等，即使两个字面量包含相同内容。RegExp 对象也可以在运行时使用 new RegExp（见 15.10.4）或以函数方式调用 RegExp 构造器来创建（见 15.10.3）。

下面的产生式描述了正则表达式字面量的语法，输入元素扫描器还用它搜索正则表达式字面量的结束位置。RegularExpressionBody 和 RegularExpressionFlags 包含的字符组成的字符串会直接传递给正则表达式构造器，在那里用更严格文法进行解析。一个实现可以扩展正则表达式构造器的文法。但它不能扩展 RegularExpressionBody 和 RegularExpressionFlags 产生式或使用这些产生式的产生式。

语法

`RegularExpressionLiteral :: / RegularExpressionBody / RegularExpressionFlags` `RegularExpressionBody :: RegularExpressionFirstChar RegularExpressionChars` `RegularExpressionChars :: [empty] RegularExpressionChars RegularExpressionChar``RegularExpressionFirstChar :: RegularExpressionNonTerminator but not *or \or / or [ RegularExpressionBackslashSequence RegularExpressionClass``RegularExpressionChar :: RegularExpressionNonTerminator but not \or / or [ RegularExpressionBackslashSequence RegularExpressionClass` `RegularExpressionBackslashSequence :: \ RegularExpressionNonTerminator` `RegularExpressionNonTerminator :: SourceCharacter but not LineTerminator` `RegularExpressionClass :: [ RegularExpressionClassChars ]` `RegularExpressionClassChars :: [empty]` `RegularExpressionClassChars RegularExpressionClassChar RegularExpressionClassChar :: RegularExpressionNonTerminator but not ]or \ RegularExpressionBackslashSequence` `RegularExpressionFlags :: [empty] RegularExpressionFlags IdentifierPart`

正则表达式字面量不能为空；并不是说正则表达式字面量不能代表空，字符 // 会启动一个单行注释。要指定一个空正则，使用：/(?:)/。

语义

正则表达式字面量会评估为一个 Object 类型值，它是标准内置构造器 RegExp 的一个实例。此值取决于两个步骤：首先，展开组成正则表达式产生式 RegularExpressionBody 和 RegularExpressionFlags 的字符，将其以未解析形式分别存成两个字符串 Pattern 和 Flags。然后，在每次评估字面量时创建新对象，仿佛使用 new RegExp(Pattern, Flags) 一样，这里的 RegExp 是标准内置构造器名。新构造的对象将成为 RegularExpressionLiteral 的值。如果调用 new RegExp 会产生 15.10.4.1 指定的错误，那么必须把错误当作是早期错误 ( 见 第十六章 )。

## 自动分号插入

必须用分号终止某些 ECMAScript 语句 ( 空语句 , 变量声明语句 , 表达式语句 , do-while 语句 , continue 语句 , break 语句 , return 语句 ,throw 语句 )。这些分号总是明确的显示在源文本里。然而，为了方便起见，某些情况下这些分号可以在源文本里省略。描述这种情况会说：这种情况下给源代码的 token 流自动插入分号。

### 自动分号插入规则

分号插入有三个基本规则：

1.  左到右解析程序，当遇到一个不符合任何文法产生式的 token（叫做 违规 token(offending token)），那么只要满足下面条件之一就在违规 token 前面自动插入分号。
    *   至少一个 LineTerminator 分割了违规 token 和前一个 token。
    *   违规 token 是 }。
2.  左到右解析程序，tokens 输入流已经结束，当解析器无法将输入 token 流解析成单个完整 ECMAScript 程序 ，那么就在输入流的结束位置自动插入分号。
3.  左到右解析程序，遇到一个某些文法产生式允许的 token，但是此产生式是受限产生式，受限产生式的里紧跟在 no LineTerminator here 后的第一个终结符或非终结符的 token 叫做受限的 token，当至少一个 LineTerminator 分割了受限的 token 和前一个 token，那么就在受限 token 前面自动插入分号。

然而，上述规则有一个附加的优先条件：如果插入分号后解析结果是空语句，或如果插入分号后它成为 for 语句头部的两个分号之一（见 12.6.3），那么不会自动插入分号。

文法里的受限产生式只限以下：

`PostfixExpression : LeftHandSideExpression [no LineTerminator here] ++ LeftHandSideExpression [no LineTerminator here] --` `ContinueStatement : continue [no LineTerminator here] Identifier;` `BreakStatement : break [no LineTerminator here] Identifier;` `ReturnStatement : return [no LineTerminator here] Expression;` `ThrowStatement : throw [no LineTerminator here] Expression;`

这些受限产生式的实际效果如下：

当遇到的 ++ 或 --token 将要被解析器当作一个后缀运算符，并且至少有一个 LineTerminator 出现 ++ 或 --token 和它之前的 token 之间，那么在 ++ 或 --token 前面自动插入一个分号。

当遇到 continue, break, return, throw token，并且在下一个 token 前面遇到 LineTerminator，那么在 continue, break, return, throw token 后面自动插入一个分号。

这对 ECMAScript 程序员的实际影响是：

后缀运算符 ++ 或 -- 和它的操作数应该出现在同一行。

return 或 throw 语句的表达式开始位置应该和 return 或 throw token 同一行。

break 或 continue 语句的标示符应该和 break 或 continue token 同一行。

### 自动分号插入的例子

源代码：

`{ 1 2 } 3`

即使在自动分号插入规则下，它也不符合 ECMAScript 文法。做为对比，源代码：

`{ 1 2 } 3`

它还是不符合 ECMAScript 文法，但是它会被自动分号插入成为一下形式：

`{ 1  ;2 ;} 3;`

这符合 ECMAScript 文法。

源代码：

`for (a; b )`

不符合 ECMAScript 文法，并且不会被自动分号插入所更改，因为 for 语句头部需要分号。自动分号插入从来不会插入成 for 语句头部的两个分号之一。

源代码：

`return a + b`

会被自动分号插入转换成以下形式：

`return; a + b;`

表达式 a + b 不会被当做是 return 语句要返回的值，因为有一个 LineTerminator 分割了它和 return token。

源代码：

`a = b ++c`

会被自动分号插入转换成以下形式：

`a = b; ++c;`

++token 不会被当做应用于变量 b 的后缀运算符，因为 b 和 ++ 之间出现了一个 LineTerminator。

源代码：

`if (a > b) else c = d`

它不符合 ECMAScript 文法 ，else token 前面不会被自动分号插入改变，即使没有文法产生式适用这一位置，因为自动插入分号后会解析成空语句。

源代码：

`a = b + c (d + e).print()`

它不会被自动分号插入改变，因为第二行开始位置的括号表达式可以解释成函数调用的参数列表：

`a = b + c(d + e).print()`

在赋值语句必须用左括号开头的情况下，程序员在前面语句的结束位置明确的提供一个分号是个好主意，而不是依赖于自动分号插入。