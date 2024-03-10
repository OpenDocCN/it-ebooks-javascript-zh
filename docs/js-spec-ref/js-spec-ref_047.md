# 6.8 IndexedDB：浏览器端数据库

*   概述
*   indexedDB.open 方法
*   indexedDB 实例对象的方法
    *   createObjectStore 方法
    *   objectStoreNames 属性
    *   transaction 方法
    *   createIndex 方法
    *   index 方法
    *   IDBKeyRange 对象
    *   参考链接

## 概述

随着浏览器的处理能力不断增强，越来越多的网站开始考虑，将大量数据储存在客户端，这样可以减少用户等待从服务器获取数据的时间。

现有的浏览器端数据储存方案，都不适合储存大量数据：cookie 不超过 4KB，且每次请求都会发送回服务器端；Window.name 属性缺乏安全性，且没有统一的标准；localStorage 在 2.5MB 到 10MB 之间（各家浏览器不同）。所以，需要一种新的解决方案，这就是 IndexedDB 诞生的背景。

通俗地说，IndexedDB 就是浏览器端数据库，可以被网页脚本程序创建和操作。它允许储存大量数据，提供查找接口，还能建立索引。这些都是 localStorage 所不具备的。就数据库类型而言，IndexedDB 不属于关系型数据库（不支持 SQL 查询语句），更接近 NoSQL 数据库。

IndexedDB 具有以下特点。

（1）键值对储存。 IndexedDB 内部采用对象仓库（object store）存放数据。所有类型的数据都可以直接存入，包括 JavaScript 对象。在对象仓库中，数据以“键值对”的形式保存，每一个数据都有对应的键名，键名是独一无二的，不能有重复，否则会抛出一个错误。

（2）异步。 IndexedDB 操作时不会锁死浏览器，用户依然可以进行其他操作，这与 localStorage 形成对比，后者的操作是同步的。异步设计是为了防止大量数据的读写，拖慢网页的表现。

（3）支持事务。 IndexedDB 支持事务（transaction），这意味着一系列操作步骤之中，只要有一步失败，整个事务就都取消，数据库回到事务发生之前的状态，不存在只改写一部分数据的情况。

（4）同域限制 IndexedDB 也受到同域限制，每一个数据库对应创建该数据库的域名。来自不同域名的网页，只能访问自身域名下的数据库，而不能访问其他域名下的数据库。

（5）储存空间大 IndexedDB 的储存空间比 localStorage 大得多，一般来说不少于 250MB。IE 的储存上限是 250MB，Chrome 和 Opera 是剩余空间的某个百分比，Firefox 则没有上限。

（6）支持二进制储存。 IndexedDB 不仅可以储存字符串，还可以储存二进制数据。

目前，Chrome 27+、Firefox 21+、Opera 15+和 IE 10+支持这个 API，但是 Safari 完全不支持。

下面的代码用来检查浏览器是否支持这个 API。

```js
if("indexedDB" in window) {
    // 支持
} else {
    // 不支持
}
```

## indexedDB.open 方法

浏览器原生提供 indexedDB 对象，作为开发者的操作接口。indexedDB.open 方法用于打开数据库。

```js
var openRequest = indexedDB.open("test",1);
```

open 方法的第一个参数是数据库名称，格式为字符串，不可省略；第二个参数是数据库版本，是一个大于 0 的正整数（0 将报错）。上面代码表示，打开一个名为 test、版本为 1 的数据库。如果该数据库不存在，则会新建该数据库。如果省略第二个参数，则会自动创建版本为 1 的该数据库。

打开数据库的结果是，有可能触发 4 种事件。

*   success：打开成功。
*   error：打开失败。
*   upgradeneeded：第一次打开该数据库，或者数据库版本发生变化。
*   blocked：上一次的数据库连接还未关闭。

第一次打开数据库时，会先触发 upgradeneeded 事件，然后触发 success 事件。

根据不同的需要，对上面 4 种事件设立回调函数。

```js
var openRequest = indexedDB.open("test",1);
var db;

openRequest.onupgradeneeded = function(e) {
    console.log("Upgrading...");
}

openRequest.onsuccess = function(e) {
    console.log("Success!");
    db = e.target.result;
}

openRequest.onerror = function(e) {
    console.log("Error");
    console.dir(e);
}
```

上面代码有两个地方需要注意。首先，open 方法返回的是一个对象（IDBOpenDBRequest），回调函数定义在这个对象上面。其次，回调函数接受一个事件对象 event 作为参数，它的 target.result 属性就指向打开的 IndexedDB 数据库。

## indexedDB 实例对象的方法

获得数据库实例以后，就可以用实例对象的方法操作数据库。

### createObjectStore 方法

createObjectStore 方法用于创建存放数据的“对象仓库”（object store），类似于传统关系型数据库的表格。

```js
db.createObjectStore("firstOS");
```

上面代码创建了一个名为 firstOS 的对象仓库，如果该对象仓库已经存在，就会抛出一个错误。为了避免出错，需要用到下文的 objectStoreNames 属性，检查已有哪些对象仓库。

createObjectStore 方法还可以接受第二个对象参数，用来设置“对象仓库”的属性。

```js
db.createObjectStore("test", { keyPath: "email" }); 
db.createObjectStore("test2", { autoIncrement: true });
```

上面代码中的 keyPath 属性表示，所存入对象的 email 属性用作每条记录的键名（由于键名不能重复，所以存入之前必须保证数据的 email 属性值都是不一样的），默认值为 null；autoIncrement 属性表示，是否使用自动递增的整数作为键名（第一个数据为 1，第二个数据为 2，以此类推），默认为 false。一般来说，keyPath 和 autoIncrement 属性只要使用一个就够了，如果两个同时使用，表示键名为递增的整数，且对象不得缺少指定属性。

### objectStoreNames 属性

objectStoreNames 属性返回一个 DOMStringList 对象，里面包含了当前数据库所有“对象仓库”的名称。可以使用 DOMStringList 对象的 contains 方法，检查数据库是否包含某个“对象仓库”。

```js
if(!db.objectStoreNames.contains("firstOS")) {
     db.createObjectStore("firstOS");
}
```

上面代码先判断某个“对象仓库”是否存在，如果不存在就创建该对象仓库。

### transaction 方法

transaction 方法用于创建一个数据库事务。向数据库添加数据之前，必须先创建数据库事务。

```js
var t = db.transaction(["firstOS"],"readwrite");
```

transaction 方法接受两个参数：第一个参数是一个数组，里面是所涉及的对象仓库，通常是只有一个；第二个参数是一个表示操作类型的字符串。目前，操作类型只有两种：readonly（只读）和 readwrite（读写）。添加数据使用 readwrite，读取数据使用 readonly。

transaction 方法返回一个事务对象，该对象的 objectStore 方法用于获取指定的对象仓库。

```js
var t = db.transaction(["firstOS"],"readwrite");

var store = t.objectStore("firstOS");
```

transaction 方法有三个事件，可以用来定义回调函数。

*   abort：事务中断。
*   complete：事务完成。
*   error：事务出错。

```js
var transaction = db.transaction(["note"], "readonly");  

transaction.oncomplete = function(event) {
      // some code
};
```

事务对象有以下方法，用于操作数据。

（1）添加数据：add 方法

获取对象仓库以后，就可以用 add 方法往里面添加数据了。

```js
var store = t.objectStore("firstOS");

var o = {p: 123};

var request = store.add(o,1);
```

add 方法的第一个参数是所要添加的数据，第二个参数是这条数据对应的键名（key），上面代码将对象 o 的键名设为 1。如果在创建数据仓库时，对键名做了设置，这里也可以不指定键名。

add 方法是异步的，有自己的 success 和 error 事件，可以对这两个事件指定回调函数。

```js
var request = store.add(o,1);

request.onerror = function(e) {
     console.log("Error",e.target.error.name);
    // error handler
}

request.onsuccess = function(e) {
    console.log("数据添加成功！");
}
```

（2）读取数据：get 方法

读取数据使用 get 方法，它的参数是数据的键名。

```js
var t = db.transaction(["test"], "readonly");
var store = t.objectStore("test");

var ob = store.get(x);
```

get 方法也是异步的，会触发自己的 success 和 error 事件，可以对它们指定回调函数。

```js
var ob = store.get(x);

ob.onsuccess = function(e) {
    // ...
}
```

从创建事务到读取数据，所有操作方法也可以写成下面这样链式形式。

```js
db.transaction(["test"], "readonly")
  .objectStore("test")
  .get(X)
  .onsuccess = function(e){}
```

（3）更新记录：put 方法

put 方法的用法与 add 方法相近。

```js
var o = { p:456 };
var request = store.put(o, 1);
```

（4）删除记录：delete 方法

删除记录使用 delete 方法。

```js
var t = db.transaction(["people"], "readwrite");
var request = t.objectStore("people").delete(thisId);
```

delete 方法的参数是数据的键名。另外，delete 也是一个异步操作，可以为它指定回调函数。

（5）遍历数据：openCursor 方法

如果想要遍历数据，就要 openCursor 方法，它在当前对象仓库里面建立一个读取光标（cursor）。

```js
var t = db.transaction(["test"], "readonly");
var store = t.objectStore("test");

var cursor = store.openCursor();
```

openCursor 方法也是异步的，有自己的 success 和 error 事件，可以对它们指定回调函数。

```js
cursor.onsuccess = function(e) {
    var res = e.target.result;
    if(res) {
        console.log("Key", res.key);
        console.dir("Data", res.value);
        res.continue();
    }
}
```

回调函数接受一个事件对象作为参数，该对象的 target.result 属性指向当前数据对象。当前数据对象的 key 和 value 分别返回键名和键值（即实际存入的数据）。continue 方法将光标移到下一个数据对象，如果当前数据对象已经是最后一个数据了，则光标指向 null。

openCursor 方法还可以接受第二个参数，表示遍历方向，默认值为 next，其他可能的值为 prev、nextunique 和 prevunique。后两个值表示如果遇到重复值，会自动跳过。

### createIndex 方法

createIndex 方法用于创建索引。

假定对象仓库中的数据对象都是下面 person 类型的。

```js
var person = {
    name:name,
    email:email,
    created:new Date()
}
```

可以指定这个数据对象的某个属性来建立索引。

```js
var store = db.createObjectStore("people", { autoIncrement:true });

store.createIndex("name","name", {unique:false});
store.createIndex("email","email", {unique:true});
```

createIndex 方法接受三个参数，第一个是索引名称，第二个是建立索引的属性名，第三个是参数对象，用来设置索引特性。unique 表示索引所在的属性是否有唯一值，上面代码表示 name 属性不是唯一值，email 属性是唯一值。

### index 方法

有了索引以后，就可以针对索引所在的属性读取数据。index 方法用于从对象仓库返回指定的索引。

```js
var t = db.transaction(["people"],"readonly");
var store = t.objectStore("people");
var index = store.index("name");

var request = index.get(name);
```

上面代码打开对象仓库以后，先用 index 方法指定索引在 name 属性上面，然后用 get 方法读取某个 name 属性所在的数据。如果没有指定索引的那一行代码，get 方法只能按照键名读取数据，而不能按照 name 属性读取数据。需要注意的是，这时 get 方法有可能取回多个数据对象，因为 name 属性没有唯一值。

另外，get 是异步方法，读取成功以后，只能在 success 事件的回调函数中处理数据。

## IDBKeyRange 对象

索引的有用之处，还在于可以指定读取数据的范围。这需要用到浏览器原生的 IDBKeyRange 对象。

IDBKeyRange 对象的作用是生成一个表示范围的 Range 对象。生成方法有四种：

*   lowerBound 方法：指定范围的下限。
*   upperBound 方法：指定范围的上限。
*   bound 方法：指定范围的上下限。
*   only 方法：指定范围中只有一个值。

下面是一些代码实例：

```js
// All keys ≤ x 
var r1 = IDBKeyRange.upperBound(x);

// All keys < x 
var r2 = IDBKeyRange.upperBound(x, true);

// All keys ≥ y  
var r3 = IDBKeyRange.lowerBound(y);

// All keys > y 
var r4 = IDBKeyRange.lowerBound(y, true);

// All keys ≥ x && ≤ y 
var r5 = IDBKeyRange.bound(x, y);

// All keys > x &&< y    
var r6 = IDBKeyRange.bound(x, y, true, true);

// All keys > x && ≤ y    
var r7 = IDBKeyRange.bound(x, y, true, false);

// All keys ≥ x &&< y 
var r8 = IDBKeyRange.bound(x, y, false, true);

// The key = z 
var r9 = IDBKeyRange.only(z);
```

前三个方法（lowerBound、upperBound 和 bound）默认包括端点值，可以传入一个布尔值，修改这个属性。

生成 Range 对象以后，将它作为参数输入 openCursor 方法，就可以在所设定的范围内读取数据。

```js
var t = db.transaction(["people"],"readonly");
var store = t.objectStore("people");
var index = store.index("name");

var range = IDBKeyRange.bound('B', 'D');

index.openCursor(range).onsuccess = function(e) {
        var cursor = e.target.result;
        if(cursor) {
            console.log(cursor.key + ":");
            for(var field in cursor.value) {
                console.log(cursor.value[field]);
            }
            cursor.continue();
        }
}
```

## 参考链接

*   Raymond Camden, [Working With IndexedDB – Part 1](http://net.tutsplus.com/tutorials/javascript-ajax/working-with-indexeddb/)
*   Raymond Camden, [Working With IndexedDB – Part 2](http://net.tutsplus.com/tutorials/javascript-ajax/working-with-indexeddb-part-2/)
*   Tiffany Brown, [An Introduction to IndexedDB](http://dev.opera.com/articles/introduction-to-indexeddb/)
*   David Fahlander, [Breaking the Borders of IndexedDB](https://hacks.mozilla.org/2014/06/breaking-the-borders-of-indexeddb/)