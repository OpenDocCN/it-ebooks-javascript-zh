# 5.1 版中技术上的重大更正和阐明

7.8.4: CV 定义追加了 DoubleStringCharacter :: LineContinuation 与 SingleStringCharacter :: LineContinuation.

10.2.1.1.3：参数 S 是不能被忽略的。它控制着试图设置一个不可改变的绑定时是否抛出异常。

10.2.1.2.2：在算法的第 5 步，真被传递后最后一个参数为 [[DefineOwnProperty]]。

10.5：当重定义全局函数使，原算法步骤 5.E 调整为现在的 5.F，并加入一个新的步骤 5.E 用来还原与第三版的兼容性。

11.5.3：在最后符号项，指定使用 IEEE754 舍入到最接近模式。

12.6.3：在步骤 3.a.ii 的两种算法中修复缺失的 ToBoolean。

12.6.4：在最后两段的额外最后一句中，阐明某些属性枚举的规定。

12.7，12.8，12.9：BNF 的修改为阐明 continue 或 break 语句没有一个 Identifier 或一个 return 语句没有一个 Expression 时，在分号之前可以有一个 LineTerminator 。

12.14：算法 1 的步骤 3 算法 3 的步骤 2.a 中，纠正这样的值域 B 是作为参数传递而不是 B 本身。

15.1.2.2：在算法的步骤 2 中阐明 S 可能是空字符串。

15.1.2.3：在算法的步骤 2 中阐明 trimmedString 可以是空字符串。

15.1.3：添加注释阐明 ECMAScript 中的 URI 语法基于 [RFC 2396](http://tools.ietf.org/html/rfc2396) 和较新的 [RFC 3986](http://tools.ietf.org/html/rfc3986)。

15.2.3.7：在算法步骤 5 和 6 中更正使用变量 P。

15.2.4.2：第五版处理 undefined 和 null 值导致现有代码失败。规范修改为保持这样的代码的兼容性。在算法中加入新的步骤 1 和 2。

15.3.4.3：步骤 5 和 7 版 5 算法已被删除，因为它们规定要求 argArray 参数与泛数组状对象的其它用法不一致。

15.4.4.12：在步骤 9.A，用 actualStart 替换不正确 relativeStart 引用。

15.4.4.15：阐明 fromIndex 的默认值是数组的长度减去 1。

15.4.4.18：在算法的第 9 步，undefined 是现在指定的返回值。

15.4.4.22：在步骤 9.c.ii 中，第一个参数的 [[Call]] 内部方法已经改变为 undefined，保持与 Array.prototype.reduce 定义的一致性。

15.4.5.1：在算法步骤 3.l.ii 和 3.l.iii 中，变量的名字是相反的，导致一个不正确的相反测试。

15.5.4.9：规范要求每有关规范等效字符串删除，算法从每一个段落都承接 ，因为它在注 2 中被列为建议的。

15.5.4.14：在 split 算法步骤 11.A 和 13.a，SplitMatch 参数的位置顺序已修正为匹配 SplitMatch 的实际参数特征。在步 13.a.iii.7.d，lengthA 取代 A.length。

15.5.5.2：在首段中，删除的单个字符属性访问“array index”语义的含义。改进算法步骤 3 和 5，这样它们不执行“array index”的要求。

15.9.1.15：为缺失字段指定了合法值范围。淘汰“time-only”格式。所有可选字段指定默认值。

15.10.2.2：算法步骤编号为第二步所产生的内部闭包被错误的编号，它们是额外的算法步骤。

15.10.2.6：在步骤 3 中的列表中抽象运算符 IsWordChar 的第一个字符是“a”而不是“A”。

15.10.2.8：在闭包算法返回抽象运算符 CharacterSetMatcher 中 ，为了避免与一个闭包的形参名称冲突，步骤 3 中定义的变量作为参数传递在第 4 步更名为 ch。

15.10.6.2：步骤 9.e 被删除，因为它执行了 I 的额外增量。

15.11.1.1：当 message 参数是 undefined 时，撤销 message 自身属性设置为空字符串的要求。

15.11.1.2：当 message 参数是 undefined 时，撤销 message 自身属性设置为空字符串的要求。

15.11.4.4：步骤 6-10 修改 / 添加正确处理缺少或空的 message 属性值。

15.11.1.2: 移除了当 message 参数为 undefined 时将 messge 自身属性设为空字符串的要求。

15.12.3：在 JA 的内部操作的第一步 10.b.iii，串联的最后一个元素是 “]”。

B.2.1：追加注释，说明编码是基于 [RFC 1738](http://tools.ietf.org/html/rfc1738) 而不是新的 [RFC 3986](http://tools.ietf.org/html/rfc3986)。

附录 C：增加了 FutureReservedWords 在标准模式下的相应内容到 7.6.12 节。