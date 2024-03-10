# TypeScript 手册

> 从前打心眼儿里讨厌编译成 JavaScript 的这类语言，像 Coffee，Dart 等。 但是在 15 年春节前后却爱上了 TypeScript。 同时非常喜欢的框架 Dojo，Angularjs 也宣布使用 TypeScript 做新版本的开发。 那么 TypeScript 究竟为何物？又有什么魅力呢？

TypeScript 是 Microsoft 公司注册商标。

TypeScript 具有类型系统，且是 JavaScript 的超集。 它可以编译成普通的 JavaScript 代码。 TypeScript 支持任意浏览器，任意环境，任意系统并且是开源的。

TypeScript 目前还在积极的开发完善之中，不断地会有新的特性加入进来。 因此本手册也会紧随官方的每个 commit，不断地更新新的章节以及修改措词不妥之处。

如果你对 TypeScript 一见钟情，可以订阅~~and star~~本手册，及时了解 ECMAScript 2015 以及 2016 里新的原生特性，并借助 TypeScript 提前掌握使用它们的方式！ 如果你对 TypeScript 的爱愈发浓烈，可以与楼主一起边翻译边学习，*[PRs Welcome!!!](https://github.com/zhongsp/TypeScript/pulls)* 在相关链接的末尾可以找到本手册的[Github 地址](https://github.com/zhongsp/TypeScript)。

## 目录

*   基础类型
*   枚举
*   变量声明
*   接口
*   高级类型
*   类
*   命名空间和模块
*   命名空间
*   模块
*   函数
*   泛型
*   混入
*   声明合并
*   类型推论
*   类型兼容性
*   书写.d.ts 文件
*   Iterators 和 Generators
*   Symbols
*   Decorators
*   JSX
*   tsconfig.json
*   编译选项
*   在 MSBuild 里使用编译选项
*   与其它构建工具整合
*   NPM 包的类型
*   Wiki
    *   TypeScript 里的 this
    *   编码规范
    *   常见编译错误
    *   支持 TypeScript 的编辑器
    *   结合 ASP.NET v5 使用 TypeScript
    *   架构概述
    *   发展路线图
*   快速上手
    *   React 和 webpack

## 主要修改

*   2016-02-27 新增章节：快速上手 React 和 webpack
*   2016-01-31 新增章节：TypeScript 里的 this
*   2016-01-24 新增章节：发展路线图
*   2016-01-23 新增章节：编码规范
*   2016-01-23 新增章节：架构概述
*   2015-12-27 新增章节：结合 ASP.NET v5 使用 TypeScript
*   2015-12-26 新增章节：支持 TypeScript 的编辑器
*   2015-12-26 新增章节：常见编译错误
*   2015-12-19 新增章节：JSX
*   2015-12-12 新增章节：NPM 包的类型
*   2015-12-12 新增章节：与其它构建工具整合
*   2015-12-12 新增章节：在 MSBuild 里使用编译选项

## 相关链接

*   [TypeScript 官网](http://typescriptlang.org)
*   [TypeScript on Github](https://github.com/Microsoft/TypeScript)
*   [TypeScript 语言规范](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md)
*   [本手册中文版 Github 地址](https://github.com/zhongsp/TypeScript)

# 基础类型

# 介绍

为了让程序有价值，我们需要能够处理最简单的数据单元：数字，字符串，结构体，布尔值等。 TypeScript 支持与 JavaScript 几乎相同的数据类型，此外还提供了实用的枚举类型方便我们使用。

# 布尔值

最基本的数据类型就是简单的 true/false 值，在 JavaScript 和 TypeScript 里叫做`boolean`（其它语言中也一样）。

```js
let isDone: boolean = false; 
```

# 数字

和 JavaScript 一样，TypeScript 里的所有数字都是浮点数。 这些浮点数的类型是`number`。 除了支持十进制和十六进制字面量，Typescript 还支持 ECMAScript 2015 中引入的二进制和八进制字面量。

```js
let decLiteral: number = 6;
let hexLiteral: number = 0xf00d;
let binaryLiteral: number = 0b1010;
let octalLiteral: number = 0o744; 
```

# 字符串

JavaScript 程序的另一项基本操作是处理网页或服务器端的文本数据。 像其它语言里一样，我们使用`string`表示文本数据类型。 和 JavaScript 一样，可以使用双引号（`"`）或单引号（`'`）表示字符串。

```js
let name: string = "bob";
name = "smith"; 
```

你还可以使用*模版字符串*，它可以定义多行文本和内嵌表达式。 这种字符串是被反引号包围（`` ` ``），并且以`${ expr }`这种形式嵌入表达式

```js
let name: string = `Gene`;
let age: number = 37;
let sentence: string = `Hello, my name is ${ name }.

I'll be ${ age + 1 } years old next month.`; 
```

这与下面定义`sentence`的方式效果相同：

```js
let sentence: string = "Hello, my name is " + name + ".\n\n" +
    "I'll be " + (age + 1) + " years old next month."; 
```

# 数组

TypeScript 像 JavaScript 一样可以操作数组元素。 有两种方式可以定义数组。 第一种，可以在元素类型后面接上`[]`，表示由此类型元素组成的一个数组：

```js
let list: number[] = [1, 2, 3]; 
```

第二种方式是使用数组泛型，`Array<元素类型>`：

```js
let list: Array<number> = [1, 2, 3]; 
```

# 元组 Tuple

元组类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。 比如，你可以定义一对值分别为`string`和`number`类型的元组。

```js
// Declare a tuple type
let x: [string, number];
// Initialize it
x = ['hello', 10]; // OK
// Initialize it incorrectly
x = [10, 'hello']; // Error 
```

当访问一个已知索引的元素，会得到正确的类型：

```js
console.log(x[0].substr(1)); // OK
console.log(x[1].substr(1)); // Error, 'number' does not have 'substr' 
```

当访问一个越界的元素，会使用联合类型替代：

```js
x[3] = 'world'; // OK, 字符串可以赋值给(string | number)类型

console.log(x[5].toString()); // OK, 'string' 和 'number' 都有 toString

x[6] = true; // Error, 布尔不是(string | number)类型 
```

联合类型是高级主题，我们会在以后的章节里讨论它。

# 枚举

`enum`类型是对 JavaScript 标准数据类型的一个补充。 像 C#等其它语言一样，使用枚举类型可以为一组数值赋予友好的名字。

```js
enum Color {Red, Green, Blue};
let c: Color = Color.Green; 
```

默认情况下，从`0`开始为元素编号。 你也可以手动的指定成员的数值。 例如，我们将上面的例子改成从`1`开始编号：

```js
enum Color {Red = 1, Green, Blue};
let c: Color = Color.Green; 
```

或者，全部都采用手动赋值：

```js
enum Color {Red = 1, Green = 2, Blue = 4};
let c: Color = Color.Green; 
```

枚举类型提供的一个便利是你可以由枚举的值得到它的名字。 例如，我们知道数值为 2，但是不确定它映射到 Color 里的哪个名字，我们可以查找相应的名字：

```js
enum Color {Red = 1, Green, Blue};
let colorName: string = Color[2];

alert(colorName); 
```

# 任意值

有时候，我们会想要为那些在编程阶段还不清楚类型的变量指定一个类型。 这些值可能来自于动态的内容，比如来自用户输入或第三方代码库。 这种情况下，我们不希望类型检查器对这些值进行检查而是直接让它们通过编译阶段的检查。 那么我们可以使用`any`类型来标记这些变量：

```js
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean 
```

在对现有代码进行改写的时候，`any`类型是十分有用的，它允许你在编译时可选择地包含或移除类型检查。 你可能认为`Object`有相似的作用，就像它在其它语言中那样。 但是`Object`类型的变量只是允许你给它赋任意值 -- 但是却不能够在它上面调用任意的方法，即便它真的有这些方法：

```js
let notSure: any = 4;
notSure.ifItExists(); // okay, ifItExists might exist at runtime
notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)

let prettySure: Object = 4;
prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'. 
```

当你只知道一部分数据的类型时，`any`类型也是有用的。 比如，你有一个数组，它包含了不同的类型的数据：

```js
let list: any[] = [1, true, "free"];

list[1] = 100; 
```

# 空值

某种程度上来说，`void`类型像是与`any`类型相反，它表示没有任何类型。 当一个函数没有返回值时，你通常会见到其返回值类型是`void`：

```js
function warnUser(): void {
    alert("This is my warning message");
} 
```

声明一个`void`类型的变量没有什么大用，因为你只能为它赋予`undefined`和`null`：

```js
let unusable: void = undefined; 
```

# 类型断言

有时候你会遇到这样的情况，你会比 TypeScript 更了解某个值的详细信息。 通常这会发生在你清楚地知道一个实体具有比它现有类型更确切的类型。

通过*类型断言*这种方式可以告诉编译器，“相信我，我知道自己在干什么”。 类型断言好比其它语言里的类型转换，但是不进行特殊的数据检查和解构。 它没有运行时的影响，只是在编译阶段起作用。 TypeScript 会假设你，程序员，已经进行了必须的检查。

类型断言有两种形式。 其一是“尖括号”语法：

```js
let someValue: any = "this is a string";

let strLength: number = (<string>someValue).length; 
```

另一个为`as`语法：

```js
let someValue: any = "this is a string";

let strLength: number = (someValue as string).length; 
```

两种形式是等价的。 至于使用哪个大多数情况下是凭个人喜好；然而，当你在 TypeScript 里使用 JSX 时，只有`as`语法断言是被允许地。

# 关于`let`

你可能已经注意到了，我们使用`let`关键字来代替大家所熟悉的 JavaScript 关键字`var`。 `let`关键字是 JavaScript 的一个新概念，TypeScript 实现了它。 我们会在以后详细介绍它，很多常见的问题都可以通过使用`let`来解决，所以尽可能地使用`let`来代替`var`吧。