# ECMAScript 的严格模式

严格模式下的限制说明

*   在严格模式下的代码中，"implements", "interface", "let", "package", "private", "protected", "public", "static", 和 "yield" 都被作为未来可能会使用到的保留字（7.6.12）。
*   符合规范的实现中，当处理严格模式下的代码时，不应该像 B.1.1 中描述地那样将 OctalIntegerLiteral 扩展到 NumericLiteral（7.8.3）的语法中。
*   符合规范的实现中，当处理严格模式下的代码时，不应该像 B.1.2 中描述地那样将 OctalEscapeSequence 扩展到 EscapeSequence 的语法中。
*   无法注册一个未定义的标识符或者其他无法解析的引用到全局对象下。当在严格模式下进行一个简单注册时，它左部不能解析为一个无法解析的引用。如果是无法解析的，那么就会抛出一个 ReferenceError 异常。左部也不能是一个数据属性的引用，
*   eval 或者 arguments 不能出现在一个注册操作（11.13）或者一个 Postfix 表达式的左部，也不能作为 Prefix Increment（11.4.4）或者 prefix decrement 操作（11.4.5）上面的一元表达式操作。
*   严格模式下的 arguments 对象定义了不可配置的存取属性，包括“caller”和“callee”，如果访问这两个对象则会抛出一个类型错误。
*   严格模式下的 Arguments 对象不会动态共享它们的数组索引值，这些索引值包含了函数绑定时对应格式的参数。
*   严格模式下的函数中，如果一个参数对象绑定了作用域内的 arguments 标识符来获取参数对象，那么这个参数对象是不可变的，并在之后也不能进行注册操作。
*   在严格模式下，如果代码包含了一个含有一个以上任意数据属性的定义，那么这就是一个语法错误。
*   在严格模式下，如果 eval 或者 argument 出现在属性参数列表中，那么这就是一个语法错误