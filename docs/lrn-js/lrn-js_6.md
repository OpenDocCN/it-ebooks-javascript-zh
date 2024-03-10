# 循环

循环是一种变量在其中不断改变的重复状。如果您希望一遍又一遍地运行相同的代码，并且每次的值都不同，那么使用循环是很方便的。

与其用：

```js
doThing(cars[0]);
doThing(cars[1]);
doThing(cars[2]);
doThing(cars[3]);
doThing(cars[4]); 
```

不如用：

```js
for (var i=0; i < cars.length; i++) { 
    doThing(cars[i]);
} 
```

# For 循环

for 语句是最简单的循环形式。它的语法类似 if 语句，只是多了些选项：

```js
for(condition; end condition; change){
    // do it, do it now
} 
```

来看看如何使用 for 循环执行 10 次相同的代码：

```js
for(var i = 0; i < 10; i = i + 1){
    // 执行这段代码 10 次
} 
```

> ***注意***: `i = i + 1` 也可以写成 `i++`.

Exercise 使用 for 循环，创建一个 `message` 变量，使其值为整数 0 到 99 (0, 1, 2, ...) 之和。

```js
var message = "";
```

# While 循环

只要特殊条件为真，While 循环就一直执行代码块。

```js
while(condition){
    // 只要条件为真就继续执行
} 
```

举个例子，以下代码会一直执行，只要变量 i 小于 5：

```js
var i = 0, x = "";
while (i < 5) {
    x = x + "The number is " + i;
    i++;
} 
```

Do/While 循环是 while 循环的变体。这种循环将先检查条件是否为真，再执行代码块。它将重复循环只要条件为真：

```js
do {
    // 代码块被执行
} while (condition); 
```

**注意**: 要注意避免如果条件总为真导致的无限循环！

Exercise 使用 while 循环，创建一个 `message` 变量，保存连接的整数，仅当长度小于 100。

```js
var message = "";
```

# Do...While 循环

do...while 语句创建的循环会先执行语句直到条件判断为假。 do... while 的语法如下：

```js
do{
    // statement
}
while(expression) ; 
```

来看看如何使用 `do...while` 循环答应小于 10 的数字：

```js
var i = 0;
do {
    document.write(i + " ");
    i++; //  i 增加 1 
} while (i < 10); 
```

> ***注意***: `i = i + 1` 也可以写成 `i++`.

Exercise 使用 do...while 循环，打印小于 5 的数字。

```js
var i = 0;
```