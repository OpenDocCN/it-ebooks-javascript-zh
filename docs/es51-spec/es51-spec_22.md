# 第 5 版中可能会对第 3 版产生兼容性影响的更正及澄清

全体：在第 3 版规范中像“就像用表达式 new Array() 一样”这样的短语的意思受到了误解。第 5 版规范中，对标准内置对象、属性的所有内部引用和内部调用相关文本描述，都做了澄清：应使用实际的内置对象，而不是对应命名属性的当前动态值。

11.8.2，11.8.3，11.8.5：ECMAScript 总体上是以从左到右的顺序解释执行，但是第 3 版规范中 > 和 <= 运算符的描述语言导致了局部从右到左的顺序。本规范已经更正了这些运算符，现在完全是从左到右的顺序解释执行。然而，这个对顺序的修改，如果在解释执行过程期间产生副作用，就有可能被观察到。

11.1.4：第 5 版澄清了 Array Initialiser 结束位置的尾端逗号不计入数组长度。这不是对第 3 版语义的修改，但有些实现在之前可能对此有误解。

11.2.3：第 5 版调换了算法步骤 2 和 3 的顺序。第 1 版一直到第 3 版规定的顺序是错误的，原来的顺序在解释执行 Arguments 时有副作用，可能影响到 MemberExpression 的解释执行结果。

12.14：在第 3 版中，对于传给 try 语句的 catch 子句的异常参数的名称解析，用与 new Object() 一样的方式创建一个对象来作为解析这个名称的作用域。如果实际的异常对象是个函数并且在 catch 子句中调用了它，那么作用域对象将会作为 this 值传给这个调用。在函数体里可以给它的 this 值定义新属性，并且这些属性名将在函数返回之后在 catch 子句的作用域内变成可见的标识符绑定。在第 5 版中，如果把异常参数作为函数来调用，传入的 this 值是 undefined 。

13：在第 3 版中，有 Identifier 的 FunctionExpression 产生式的算法，用与 new Object() 一样的方式创建一个对象并加入到作用域链中，用来提供函数名查找的作用域。标识符解析规则（第 3 版里的 10.1.4 ）会作用在这样的对象上，如果需要，还会用对象的原型链来尝试解析标识符。这种方式使得 Object.prototype 的所有属性以标识符的形式在这个作用域里可见。实践中，大多数第 3 版的实现都没有实行这个语义。地 5 版更改了这里的语义，用一个声明式环境记录项来绑定了函数名。

14：在第 3 版中，产生式 SourceElements : SourceElements SourceElement 的算法不像相同形式的 Block，对 statement 的结果值做正确的传递。这可导致 eval 函数解释执行一个 Program 文本时产生错误的结果。实践中，大多数第 3 版的实现都做了正确的传递，而不关心第 5 版规定了什么。

15.10.6：RegExp.prototype 现在是一个 RegExp 对象，而不是 Object 的一个实例。用 Object.prototype.toString 可看到它的 [[Class]] 内部属性值现在是 "RegExp"，不是 "Object"。