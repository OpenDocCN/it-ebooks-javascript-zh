# 第 5 版内容的增加与变化，介绍第 3 版不兼容问题

7.1：Unicode 格式控制字符在受到处理之前不再从 ECMAScript 源文本中剥离。在第五版中，如果这样一个字符在字符串字面量或者正则表达式字面量中出现，这个字符会被合并到字面量中，而在第三版里，这个字符不会被合并。

7.2：Unicode 字符 <BOM> 现在是作为空格使用，如果它出现在本该是一个标识符的位置的中间，则会产生一个语法错误，而在第三版里不会。

7.3：换行符以前是作为转义字符处理，而现在允许换行符被包含在字符串字面量标记中。这在第三版中会产生一个语法错误。

7.8.5：现在的正则表达式字面量在字面量解析执行的时候都会返回一个唯一的对象。这个改变可以被任意测试字面量值的对象 ID 或者一些敏感的副作用的程序检测到。

7.8.5：第五版要求提前抛出任意可能的正则表达式结构错误，这些结构错误会在将正则表达式字面量转换成正则表达式对象的时候产生。在第五版之前的实现允许延迟抛出 [TypeError]，直到真正执行到这个对象。

7.8.5：在第五版中，未转义的 "/" 字符可以作为 CharacterClass 存在于正则表达式字面量中。在第三版里，这样的字符是作为字面量的最后一个字符存在。

10.4.2：在第五版中，间接调用 eval 函数会将全局对象作为 [执行代码](http://es5.github.com/#eval-code) 的变量环境和 [词法环境](http://es5.github.com/#x10.2) 。在第三版中，[eval] 函数的间接调用者的变量和 [词法环境](http://es5.github.com/#x10.2) 是作为 [执行代码](http://es5.github.com/#eval-code) 的环境使用。

15.4.4：在第五版中，所有 [Array.prototype](http://es5.github.com/#x15.4.3.1) 下的方法都是通用的。在第三版中，toString 和 toLocaleString 方法不是通用的，如果被非 Array 实例调用时会抛出一个 TypeError 的异常。

10.6：在第五版中，argument 对象与实际的参数符合，它的数组索引属性是可枚举的。在第三版中，这些属性是不可枚举的。

10.6：在第五版中，一个 arguments 对象的 [Class](http://www.w3.org/html/ig/zh/wiki/index.php?title=Class&action=edit&redlink=1) 内置属性值是“Arguments”。在第三版中，它是“Object”。当对 argument 对象调用 toString 的时候

12.6.4：当 in 表达式执行一个 null 或者 undefined 时 ,for-in 语句不再抛出 TypeError。取而代之的是将其作为不包含可枚举属性的对象执行。

15：在第五版中，下面的新属性都是在第三种中已存在的内建对象中定义，Object.getPrototypeOf, Object.getOwnPropertyDescriptor, Object.getOwnPropertyNames, Object.create, Object.defineProperty, Object.defineProperties, Object.seal, Object.freeze, Object.preventExtensions, Object.isSealed, Object.isFrozen, Object.isExtensible, Object.keys, Function.prototype.bind, Array.prototype.indexOf, Array.prototype.lastIndexOf, Array.prototype.every, Array.prototype.some, Array.prototype.forEach, Array.prototype.map, Array.prototype.filter, Array.prototype.reduce, Array.prototype.reduceRight, String.prototype.trim, Date.now, Date.prototype.toISOString, Date.prototype.toJSON。

15：实现现在要求忽略内建方法中的额外参数，除非明确指定。在第三版中，并没有规定额外参数的处理方式，实现中明确允许抛出一个 TypeErrorBold text 错误。

15.1.1：全局对象的值属性 NaN，Infinity 和 Undefined 改为只读属性。

15.1.2.1：实现不再允许约束非直接调用 eval 的方式。另外间接调用 eval 会使用全局对象作为变量环境，而不是使用调用者的变量环境作为变量环境。

15.1.2.2：parseInt 的规范不再允许实现将 0 开头的字符串作为 8 进制值。

15.3.4.3：在第三版中，如果传入 [Function.prototype.apply](http://es5.github.com/#x15.3.4.3) 的第二个参数不是一个数组对象或者一个 arguments 对象，就会抛出一个 TypeError。在第五版中，参数也可以是任意类型的含有 length 属性的类数组对象。

15.3.4.3，15.3.4.4：在第三版中，在 [Function.prototype.apply](http://es5.github.com/#x15.3.4.3) 或者 [Function.prototype.call](http://es5.github.com/#x15.3.4.4) 中传入 undefined 或者 null 作为第一个参数会导致 [全局对象](http://es5.github.com/#x15.1) 被作为一个个参数传入，间接导致目标函数的 [this] 会指向全局变量环境。如果第一个参数是一个 [原始值](http://es5.github.com/#primitive_value) ，在 [原始值](http://es5.github.com/#primitive_value) 上调用 [ToObject](http://es5.github.com/#x9.9) 的结果会作为 this 的值。在第五版中，这些转换不会出现，目标函数的 this 会指向真实传入的参数。这个不同点一般情况下对已存在的遵循 ECMAScript 第三版的代码来说不太明显，因为相应转换会在目标函数生效之前执行。然而，基于不同的实现，如果使用 apply 或者 call 调用函数时，这个不同点就会很明显。另外，用这个方法调用一个标准的内建函数，并使用 null 或者 undefined 作为参数时，很可能会导致第五版标准下的实现与第三版标准下的实现不同。特别是第五版中代表性地规定了需要将实际调用的传入的 this 值作为对象的内建函数，在传入 null 或者 undefined 作为 this 值时，会抛出一个 TypeError 异常。

15.3.5.2：在第五版中，函数实例的 prototype 属性是不可枚举的。在第三版中，是可以枚举的。

15.5.5.2：在第五版中，一个字符串对象的 [primitiveValue](http://www.w3.org/html/ig/zh/wiki/index.php?title=PrimitiveValue&action=edit&redlink=1) 的单个字符可以作为字符串对象的数组索引属性访问。这些属性是不可泄也不可配置的，并会影响任意名字相同的继承属性。在第三版中，这些属性不会存在，ECMAScript 代码可以通过这些名字动态添加和移除可写的属性并访问以这些名字继承的属性。

15.9.4.2：[Date.parse](http://es5.github.com/#x15.9.4.2) 方法现在不要求第一个参数首先作为 ISO 格式字符串解析。使用这个格式但是基于特定行为实现（包括未来的一些行为）或许会表现的不太一样。

15.10.2.12：在第五版中，\s 现在可以匹配 <BOM> 了

15.10.4.1：在第三版中，由 RegExp 构造器创建的对象的 source 字符串的精确形式由实现定义。在第五版中，字符串必须符合确定的指定条件，因此会和第三版标准的实现的结果不一样。

15.10.6.4：在第三版中，[RegExp.prototype.toString](http://es5.github.com/#x15.10.6.4) 的规则不需要由 RegExp 对象的 source 属性决定。在第五版中，结果必须由 source 属性经由一个指定的规则，因此会和第三版实现的结果不一样。

15.11.2.1，15.11.4.3：在第五版中，如果一个错误对象的 message 属性原始值没有通过 Error 构造器指定，那么这个原始值就是一个空的字符串。在第三版中，这个原始值由实现决定。

15.11.4.4：在第三版中，Error.prototype.toString 的结果是由实现定义的。在第五版中，有完整的规范指定，因此可能会和第三版的实现不同。

15.12： 在第五版中，JSON 是在全局环境中定义的。第三版中，测试这个名词的存在会发现它是 undefined，除非这个程序或者实现定义了这个名词。