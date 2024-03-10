# 3.7 Date 对象

*   概述
    *   Date()
    *   new Date()
    *   new Date(milliseconds)
    *   new Date(datestring)
    *   [new Date(year, month [, day, hours, minutes, seconds, ms])](#new+date%28year%2C+month+%5B%2C+day%2C+hours%2C+minutes%2C+seconds%2C+ms%5D%29)
    *   日期的运算
    *   Date 对象的方法
        *   Date.now()
        *   Date.parse()
        *   Date.UTC()
    *   Date 实例对象的方法
        *   Date.prototype.toString()
        *   Date.prototype.toUTCString()，Date.prototype.toISOString()
        *   Date.prototype.toDateString()，Date.prototype.toTimeString()
        *   Date.prototype.toLocalDateString()，Date.prototype.toLocalTimeString()
        *   Date.prototype.valueOf()
        *   Date.prototype.get 系列方法
        *   Date.prototype.set 系列方法
        *   Date.prototype.toJSON()
    *   参考链接

## 概述

Date 对象是 JavaScript 提供的日期和时间的操作接口。它有多种用法。

### Date()

作为一个函数，Date 对象可以直接调用，返回一个当前日期和时间的字符串。

```
Date()
// "Sat Mar 09 2013 08:46:54 GMT+0800 (CST)"

Date(2000, 1, 1)
// "Sat Mar 09 2013 08:46:54 GMT+0800 (CST)"
```

上面代码说明，无论有没有参数，直接调用 Date 总是返回当前时间。

### new Date()

Date 对象还是一个构造函数，对它使用 new 命令，会返回一个 Date 对象的实例。如果不加参数，生成的就是代表当前时间的对象。

```
var today = new Date();

today 
// "Sat Mar 09 2013 08:46:54 GMT+0800 (CST)"

// 等同于
today.toString() 
// "Sat Mar 09 2013 08:46:54 GMT+0800 (CST)"
```

### new Date(milliseconds)

Date 对象接受从 1970 年 1 月 1 日 00:00:00 UTC 开始计算的毫秒数作为参数。这意味着如果将 Unix 时间戳作为参数，必须将 Unix 时间戳乘以 1000。

```
new Date(1378218728000)
// Tue Sep 03 2013 22:32:08 GMT+0800 (CST)

// 1970 年 1 月 2 日的零时
var Jan02_1970 = new Date(3600*24*1000);
// Fri Jan 02 1970 08:00:00 GMT+0800 (CST)

// 1969 年 12 月 31 日的零时
var Dec31_1969 = new Date(-3600*24*1000);
// Wed Dec 31 1969 08:00:00 GMT+0800 (CST)
```

上面代码说明，Date 构造函数的参数可以是一个负数，表示 1970 年 1 月 1 日之前的时间。Date 对象能够表示的日期范围是 1970 年 1 月 1 日前后各一亿天。

### new Date(datestring)

Date 对象还接受一个日期字符串作为参数，返回所对应的时间。

```
new Date("January 6, 2013");
// Sun Jan 06 2013 00:00:00 GMT+0800 (CST)
```

所有可以被 Date.parse()方法解析的日期字符串，都可以当作 Date 对象的参数。

```
new Date("2013-02-15")
new Date("2013-FEB-15")
new Date("FEB, 15, 2013")
new Date("FEB 15, 2013")
new Date("Feberuary, 15, 2013")
new Date("Feberuary 15, 2013")
new Date("15, Feberuary, 2013")
// Fri Feb 15 2013 08:00:00 GMT+0800 (CST)
```

上面多种写法，返回的都是同一个时间。

### new Date(year, month [, day, hours, minutes, seconds, ms])

在多个参数的情况下，Date 对象将其分别视作对应的年、月、日、小时、分钟、秒和毫秒。如果采用这种用法，最少需要指定两个参数（年和月），其他参数都是可选的，默认等于 0。如果只使用年一个参数，Date 对象会将其解释为毫秒数。

```
new Date(2013) // Thu Jan 01 1970 08:00:02 GMT+0800 (CST)
new Date(2013,0) // Tue Jan 01 2013 00:00:00 GMT+0800 (CST)
new Date(2013,0,1) // Tue Jan 01 2013 00:00:00 GMT+0800 (CST)
new Date(2013,0,1,0) // Tue Jan 01 2013 00:00:00 GMT+0800 (CST)
new Date(2013,0,1,0,0,0,0) // Tue Jan 01 2013 00:00:00 GMT+0800 (CST)
```

上面代码（除了第一行）返回的是 2013 年 1 月 1 日零点的时间，可以看到月份从 0 开始计算，因此 1 月是 0，12 月是 11。但是，月份里面的天数从 1 开始计算。

这些参数如果超出了正常范围，会被自动折算。比如，如果月设为 15，就算折算为下一年的 4 月。参数还可以使用负数，表示扣去的时间。

```
new Date(2013,0) // Tue Jan 01 2013 00:00:00 GMT+0800 (CST)
new Date(2013,-1) // Sat Dec 01 2012 00:00:00 GMT+0800 (CST) 
new Date(2013,0,1) // Tue Jan 01 2013 00:00:00 GMT+0800 (CST)
new Date(2013,0,0) // Mon Dec 31 2012 00:00:00 GMT+0800 (CST)
new Date(2013,0,-1) // Sun Dec 30 2012 00:00:00 GMT+0800 (CST)
```

上面代码分别对月和日使用了负数，表示从基准日扣去相应的时间。

年的情况有所不同，如果为 0，表示 1900 年；如果为负数，则表示公元前。

```
new Date(1,0) // Tue Jan 01 1901 00:00:00 GMT+0800 (CST)
new Date(0,0) // Mon Jan 01 1900 00:00:00 GMT+0800 (CST)
new Date(-1,0) // Fri Jan 01 -1 00:00:00 GMT+0800 (CST)
```

### 日期的运算

类型转换时，Date 对象的实例如果转为数值，则等于对应的毫秒数；如果转为字符串，则等于对应的日期字符串。所以，两个日期对象进行减法运算，返回的就是它们间隔的毫秒数；进行加法运算，返回的就是连接后的两个字符串。

```
var then = new Date(2013,2,1);
var now = new Date(2013,3,1);

now - then
// 2678400000

now + then
// "Mon Apr 01 2013 00:00:00 GMT+0800 (CST)Fri Mar 01 2013 00:00:00 GMT+0800 (CST)"
```

## Date 对象的方法

### Date.now()

now 方法返回当前距离 1970 年 1 月 1 日 00:00:00 UTC 的毫秒数（Unix 时间戳乘以 1000）。

```
Date.now()
// 1364026285194
```

如果需要更精确的时间，可以使用 window.performance.now()。它提供页面加载到命令运行时的已经过去的时间，单位是浮点数形式的毫秒。

```
window.performance.now()
// 21311140.415
```

### Date.parse()

parse 方法用来解析日期字符串，返回距离 1970 年 1 月 1 日 00:00:00 的毫秒数。日期字符串的格式应该完全或者部分符合 RFC 2822 和 ISO 8061，即 YYYY-MM-DDTHH:mm:ss.sssZ 格式，其中最后的 Z 表示时区，是可选的。

*   YYYY
*   YYYY-MM
*   YYYY-MM-DD
*   THH:mm（比如“T12:00”）
*   THH:mm:ss
*   THH:mm:ss.sss

请看例子。

```
Date.parse("Aug 9, 1995")
// 返回 807897600000，以下省略返回值

Date.parse("January 26, 2011 13:51:50")
Date.parse("Mon, 25 Dec 1995 13:30:00 GMT")
Date.parse("Mon, 25 Dec 1995 13:30:00 +0430")
Date.parse("2011-10-10")
Date.parse("2011-10-10T14:48:00")
```

如果解析失败，返回 NaN。

```
Date.parse("xxx")
// NaN
```

### Date.UTC()

默认情况下，Date 对象返回的都是当前时区的时间。Date.UTC 方法可以返回 UTC 时间（世界标准时间）。该方法接受年、月、日等变量作为参数，返回当前距离 1970 年 1 月 1 日 00:00:00 UTC 的毫秒数。

```
// 使用的格式
Date.UTC(year, month[, date[, hrs[, min[, sec[, ms]]]]]) 

Date.UTC(2011,0,1,2,3,4,567)
// 1293847384567
```

该方法的参数用法与 Date 构造函数完全一致，比如月从 0 开始计算。

## Date 实例对象的方法

使用 new 命令生成的 Date 对象的实例，有很多自己的方法。

### Date.prototype.toString()

toString 方法返回一个完整的时间字符串。

```
var today = new Date(1362790014000);

today.toString()
// "Sat Mar 09 2013 08:46:54 GMT+0800 (CST)"

today
// "Sat Mar 09 2013 08:46:54 GMT+0800 (CST)"
```

因为 toString 是默认的调用方法，所以如果直接读取 Date 对象实例，就相当于调用这个方法。

### Date.prototype.toUTCString()，Date.prototype.toISOString()

toUTCString 方法返回对应的 UTC 时间，也就是比北京时间晚 8 个小时。toISOString 方法返回对应时间的 ISO8601 写法。

```
var today = new Date(1362790014000);

today.toUTCString()
// "Sat, 09 Mar 2013 00:46:54 GMT"

today.toISOString()
// "2013-03-09T00:46:54.000Z"
```

### Date.prototype.toDateString()，Date.prototype.toTimeString()

toDateString 方法返回日期的字符串形式，toTimeString 方法返回时间的字符串形式。

```
var today = new Date(1362790014000);

today.toDateString()
// "Sat Mar 09 2013"

today.toTimeString()
// "08:46:54 GMT+0800 (CST)"
```

### Date.prototype.toLocalDateString()，Date.prototype.toLocalTimeString()

toLocalDateString 方法返回一个字符串，代表日期的当地写法；toLocalTimeString 方法返回一个字符串，代表时间的当地写法。

```
var today = new Date(1362790014000);

today.toLocaleDateString()
// "2013 年 3 月 9 日"

today.toLocaleTimeString()
"上午 8:46:54"
```

### Date.prototype.valueOf()

valueOf 方法返回实例对象距离 1970 年 1 月 1 日 00:00:00 UTC 对应的毫秒数，该方法等同于 getTime 方法。

```
var today = new Date();

today.valueOf()
// 1362790014817

today.getTime()
// 1362790014817
```

该方法可以用于计算精确时间。

```
var start = new Date();

doSomething();
var end = new Date();
var elapsed = end.getTime() - start.getTime();
```

### Date.prototype.get 系列方法

Date 对象提供了一系列 get 方法，用来获取实例对象某个方面的值。

*   Date.prototype.getTime()：返回实例对象距离 1970 年 1 月 1 日 00:00:00 对应的毫秒数，等同于 valueOf 方法。
*   Date.prototype.getDate()：返回实例对象对应每个月的几号（从 1 开始）。
*   Date.prototype.getDay()：返回星期，星期日为 0，星期一为 1，以此类推。
*   Date.prototype.getFullYear()：返回四位的年份。
*   Date.prototype.getMonth()：返回月份（0 表示 1 月，11 表示 12 月）。
*   Date.prototype.getHours()：返回小时（0-23）。
*   Date.prototype.getMilliseconds()：返回毫秒（0-999）。
*   Date.prototype.getMinutes()：返回分钟（0-59）。
*   Date.prototype.getSeconds()：返回秒（0-59）。
*   Date.prototype.getTimezoneOffset()：返回当前时间与 UTC 的时区差异，以分钟表示，返回结果考虑到了夏令时因素。

```
var d = new Date("January 6, 2013");

d.getDate() // 6
d.getMonth() // 0
d.getFullYear() // 2013
d.getTimezoneOffset() // -480
```

上面这些方法默认返回的都是当前时区的时间，Date 对象还提供了这些方法对应的 UTC 版本，用来返回 UTC 时间，比如 getUTCFullYear()、getUTCMonth()、getUTCDay()、getUTCHours()等等。

### Date.prototype.set 系列方法

Date 对象提供了一系列 set 方法，用来设置实例对象的各个方面。

*   Date.prototype.setDate(date)：设置实例对象对应的每个月的几号（1-31），返回改变后毫秒时间戳。
*   Date.prototype.setFullYear(year [, month, date])：设置四位年份。
*   Date.prototype.setHours(hour [, min, sec, ms])：设置小时（0-23）。
*   Date.prototype.setMilliseconds()：设置毫秒（0-999）。
*   Date.prototype.setMinutes(min [, sec, ms])：设置分钟（0-59）。
*   Date.prototype.setMonth(month [, date])：设置月份（0-11）。
*   Date.prototype.setSeconds(sec [, ms])：设置秒（0-59）。
*   Date.prototype.setTime(milliseconds)：设置毫秒时间戳。

```
var d = new Date ("January 6, 2013");

d 
// Sun Jan 06 2013 00:00:00 GMT+0800 (CST)

d.setDate(9) 
// 1357660800000

d
// Wed Jan 09 2013 00:00:00 GMT+0800 (CST)
```

set 方法的参数都会自动折算。以 setDate 为例，如果参数超过当月的最大天数，则向下一个月顺延，如果参数是负数，表示从上个月的最后一天开始减去的天数。

```
var d = new Date("January 6, 2013");

d.setDate(32)
// 1359648000000
d 
// Fri Feb 01 2013 00:00:00 GMT+0800 (CST)

var d = new Date ("January 6, 2013");

d.setDate(-1)
// 1356796800000
d
// Sun Dec 30 2012 00:00:00 GMT+0800 (CST)
```

使用 setDate 方法，可以算出今天过后 1000 天是几月几日。

```
var d = new Date();
d.setDate( d.getDate() + 1000 );
d.getDay();
```

set 系列方法除了 setTime()，都有对应的 UTC 版本，比如 setUTCHours()。

### Date.prototype.toJSON()

toJSON 方法返回 JSON 格式的日期对象。

```
var jsonDate = (new Date()).toJSON();

jsonDate
"2013-09-03T14:26:31.880Z"

var backToDate = new Date(jsonDate);
```

## 参考链接

*   Rakhitha Nimesh，[Getting Started with the Date Object](http://jspro.com/raw-javascript/beginners-guide-to-javascript-date-and-time/)
*   Ilya Kantor, [Date/Time functions](http://javascript.info/tutorial/datetime-functions)