# 11.3 Datejs

*   概述
*   方法
    *   日期信息
    *   日期的变更
    *   日期的解析
    *   参考链接

## 概述

Datejs 是一个用来操作日期的库，官方网站为[datejs.com](http://www.datejs.com/)。

下载后插入网页，就可以使用。

```js
<script type="text/javascript" src="date.js"></script>
```

官方还提供多种语言的版本，可以选择使用。

```js
// 美国版
<script type="text/javascript" src="date-en-US.js"></script>

// 中国版
<script type="text/javascript" src="date-zh-CN.js"></script>
```

## 方法

Datejs 在原生的 Date 对象上面，定义了许多语义化的方法，可以方便地链式使用。

### 日期信息

```js
Date.today() // 返回当天日期，时间定在这一天开始的 00:00 

Date.today().getDayName() // 今天是星期几

Date.today().is().friday()      // 今天是否为星期五，返回 true 或者 false
Date.today().is().fri()         // 等同于上一行

Date.today().is().november()    // 今天是否为 11 月，返回 true 或者 false
Date.today().is().nov()         // 等同于上一行

Date.today().isWeekday() // 今天是否为工作日（周一到周五）
```

### 日期的变更

```js
Date.today().next().friday()    // 下一个星期五
Date.today().last().monday()    // 上一个星期一

new Date().next().march()       // 下个三月份的今天
new Date().last().week()        // 上星期的今天

Date.today().add(5).days() // 五天后

Date.friday() // 本周的星期五

Date.march() // 今年的三月

Date.january().first().monday() // 今年一月的第一个星期一

Date.dec().final().fri() // 今年 12 月的最后一个星期五

// 先将日期定在本月 15 日的下午 4 点 30 分，然后向后推 90 天
Date.today().set({ day: 15, hour: 16, minute: 30 }).add({ days: 90 })

(3).days().fromNow() // 三天后

(6).months().ago() // 6 个月前

(12).weeks().fromNow() // 12 个星期后

(30).days().after(Date.today()) // 30 天后
```

### 日期的解析

```js
Date.parse('today')

Date.parse('tomorrow')

Date.parse('July 8')

Date.parse('July 8th, 2007')

Date.parse('July 8th, 2007, 10:30 PM')

Date.parse('07.15.2007')
```

## 参考链接

*   [Getting Started With Datejs](http://www.datejs.com/2007/11/27/getting-started-with-datejs/)