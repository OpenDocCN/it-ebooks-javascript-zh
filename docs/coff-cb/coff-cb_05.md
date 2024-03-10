# 五、日期和时间

## 计算复活节的日期

### 问题

你需要在给出的年份中找到复活节的月份和日期。

### 解决方案

下面的函数返回数组有两个要素：复活节的月份（ 1-12 ）和日期。如果没有给出任何参数，给出的结果是当前的一年。这是 在 CoffeeScript 的[匿名公历算法](https://en.wikipedia.org/wiki/Computus#Anonymous_Gregorian_algorithm)实现的。

```
gregorianEaster = (year = (new Date).getFullYear()) ->
  a = year % 19
  b = ~~(year / 100)
  c = year % 100
  d = ~~(b / 4)
  e = b % 4
  f = ~~((b + 8) / 25)
  g = ~~((b - f + 1) / 3)
  h = (19 * a + b - d - g + 15) % 30
  i = ~~(c / 4)
  k = c % 4
  l = (32 + 2 * e + 2 * i - h - k) % 7
  m = ~~((a + 11 * h + 22 * l) / 451)
  n = h + l - 7 * m + 114
  month = ~~(n / 31)
  day = (n % 31) + 1
  [month, day]
```

### 讨论

Javascript 中的月份是 0-11 。getMonth() 查找的是三月的话将返回数字 2 ，这个函数会返回 3 。如果你想要这个功能是一致的，你可以修改这个函数。

该函数使用`~~`符号代替来 Math.floor() 。

```
gregorianEaster()    # => [4, 24] (April 24th in 2011)
gregorianEaster 1972 # => [4, 2]
```

## 计算（美国和加拿大的）感恩节日期

### 问题

你需要在给出的年份中找到感恩节的月份和日期。

### 解决方案

下面的函数返回给出年份的感恩节的日期。如果没有给出任何参数，给出的结果是当前年份。

美国的感恩节是十一月的第四个星期四。 　　

```
thanksgivingDayUSA = (year = (new Date).getFullYear()) ->
  first = new Date year, 10, 1
  day_of_week = first.getDay()
  22 + (11 - day_of_week) % 7
```

加拿大的感恩节是在十月的第二个周一。

```
thanksgivingDayCA = (year = (new Date).getFullYear()) ->
    first = new Date year, 9, 1
    day_of_week = first.getDay()
    8 + (8 - day_of_week) % 7
```

### 讨论

```
thanksgivingDayUSA() #=> 24 (November 24th, 2011)

thanksgivingDayCA() # => 10 (October 10th, 2011)

thanksgivingDayUSA(2012) # => 22 (November 22nd)

thanksgivingDayCA(2012) # => 8 (October 8th)
```

这个想法很简单：

1.  找出哪一天是以下各月份的第一天（美国十一月，加拿大十月）。
2.  计算从那天起偏移到下一个工作日的量（美国星期四，加拿大星期一）。
3.  将这个偏移量加上第一个可能的假期日期（第二十二个美国感恩节，第八个加拿大感恩节）。

## 计算两个日期中间的天数

### 问题

你需要找出两个日期间隔了几年，几个月，几天，几个小时，几分钟，几秒。

### 解决方案

利用 JavaScript 的日期计算函数 getTime() 。它提供了从 1970 年 1 月 1 日开始经过了多少毫秒。

```
DAY = 1000 * 60 * 60  * 24

d1 = new Date('02/01/2011')
d2 = new Date('02/06/2011')

days_passed = Math.round((d2.getTime() - d1.getTime()) / DAY)
```

### 讨论

使用毫秒，使计算时间跨度更容易，以避免日期的溢出错误。所以我们首先计算一天有多少毫秒。然后，给出了 2 个不同的日期，只须知道在 2 个日期之间的毫秒数，然后除以一天的毫秒数，这将得到 2 个不同的日期之间的天数。

如果你想计算出 2 个日期对象的小时数，你可以用毫秒的时间间隔除以一个小时有多少毫秒来得到。同样的可以得到几分钟和几秒。

```
HOUR = 1000 * 60 * 60

d1 = new Date('02/01/2011 02:20')
d2 = new Date('02/06/2011 05:20')

hour_passed = Math.round((d2.getTime() - d1.getTime()) / HOUR)
```

## 找到一个月中的最后一天

### 问题

你需要去找出一个月的最后一天，但是一年中的各月并没有一个固定时间表。

### 解决方案

利 用 JavaScript 的日期下溢来找到给出月份的第一天：

```
now = new Date
lastDayOfTheMonth = new Date(1900+now.getYear(), now.getMonth()+1, 0)
```

### 讨论

JavaScript 的日期构造函数成功地处理溢出和下溢情况，使日期的计算变得很简单。鉴于这种简单操作，不需要担心一个给定的月份里有多少天；只需要用数学稍加推导。在十二月，以上的解决方案就是寻找当前年份的第十三个月的第 0 天日期，那么它就是下一年的一月一日，也计算出来今年十二月份 31 号的日期。

## 找到上一个月(或下一个月)

### 问题

你需要计算相关日期范围例如“上一个月”，“下一个月”。

### 解决方案

添加或减去当月的数字，JavaScript 的日期构造函数会修复数学知识。

```
 # these examples were written in GMT-6

 # Note that these examples WILL work in January!

now = new Date
 # => "Sun, 08 May 2011 05:50:52 GMT"

lastMonthStart = new Date 1900+now.getYear(), now.getMonth()-1, 1
 # => "Fri, 01 Apr 2011 06:00:00 GMT"

lastMonthEnd = new Date 1900+now.getYear(), now.getMonth(), 0
 # => "Sat, 30 Apr 2011 06:00:00 GMT"
```

### 讨论

JavaScript 的日期对象会处理下溢和溢出的月和日，并将相应调整日期对象。例如，你可以要求寻找三月的第 42 天，你将获得 4 月 11 日。

JavaScript 对象存储日期为从 1900 开始的每年的年份数，月份为一个 0 到 11 的整数，日期为从 1 到 31 的一个整数。在上述解决方案中，上个月的起始日是要求在本年度某一个月的第一天，但月是从 -1 至 10 。如果月是 -1 的日期对象将实际返回为前一年的十二月：

```
lastNewYearsEve = new Date 1900+now.getYear(), -1, 31
 # => "Fri, 31 Dec 2010 07:00:00 GMT"
```

对于溢出是同样的：

```
thirtyNinthOfFourteember = new Date 1900+now.getYear(), 13, 39
 # => "Sat, 10 Mar 2012 07:00:00 GMT"
```

## 计算月球的相位

### 问题

你想找出月球的相位。

### 解决方案

以下代码提供了一种计算给出日期的月球相位计算方案：

```
 # moonPhase.coffee

 # Moon-phase calculator

 # Roger W. Sinnott, Sky & Telescope, June 16, 2006

 # http://www.skyandtelescope.com/observing/objects/javascript/moon_phases

 #

 # Translated to CoffeeScript by Mike Hatfield @WebCoding4Fun

proper_ang = (big) ->
    tmp = 0
    if big > 0
        tmp = big / 360.0
        tmp = (tmp - (~~tmp)) * 360.0
    else
        tmp = Math.ceil(Math.abs(big / 360.0))
        tmp = big + tmp * 360.0

    tmp

jdn = (date) ->  
    month = date.getMonth()
    day = date.getDate()
    year = date.getFullYear()
    zone = date.getTimezoneOffset() / 1440

    mm = month
    dd = day
    yy = year

    yyy = yy
    mmm = mm
    if mm < 3
        yyy = yyy - 1
        mmm = mm + 12

    day = dd + zone + 0.5
    a = ~~( yyy / 100 )
    b = 2 - a + ~~( a / 4 )
    jd = ~~( 365.25 * yyy ) + ~~( 30.6001 * ( mmm+ 1 ) ) + day + 1720994.5
    jd + b if jd > 2299160.4999999

moonElong = (jd) ->
    dr    = Math.PI / 180
    rd    = 1 / dr
    meeDT = Math.pow((jd - 2382148), 2) / (41048480 * 86400)
    meeT  = (jd + meeDT - 2451545.0) / 36525
    meeT2 = Math.pow(meeT, 2)
    meeT3 = Math.pow(meeT, 3)
    meeD  = 297.85 + (445267.1115 * meeT) - (0.0016300 * meeT2) + (meeT3 / 545868)
    meeD  = (proper_ang meeD) * dr
    meeM1 = 134.96 + (477198.8676 * meeT) + (0.0089970 * meeT2) + (meeT3 / 69699)
    meeM1 = (proper_ang meeM1) * dr
    meeM  = 357.53 + (35999.0503 * meeT)
    meeM  = (proper_ang meeM) * dr

    elong = meeD * rd + 6.29 * Math.sin( meeM1 )
    elong = elong     - 2.10 * Math.sin( meeM )
    elong = elong     + 1.27 * Math.sin( 2*meeD - meeM1 )
    elong = elong     + 0.66 * Math.sin( 2*meeD )
    elong = proper_ang elong
    elong = Math.round elong

    moonNum = ( ( elong + 6.43 ) / 360 ) * 28
    moonNum = ~~( moonNum )

    if moonNum is 28 then 0 else moonNum

getMoonPhase = (age) ->
    moonPhase = "new Moon"
    moonPhase = "first quarter" if age > 3 and age < 11 
    moonPhase = "full Moon"     if age > 10 and age < 18
    moonPhase = "last quarter"  if age > 17 and age < 25

    if ((age is 1) or (age is 8) or (age is 15) or (age is 22))
        moonPhase = "1 day past " + moonPhase

    if ((age is 2) or (age is 9) or (age is 16) or (age is 23))
        moonPhase = "2 days past " + moonPhase

    if ((age is 3) or (age is 1) or (age is 17) or (age is 24))
        moonPhase = "3 days past " + moonPhase

    if ((age is 4) or (age is 11) or (age is 18) or (age is 25))
        moonPhase = "3 days before " + moonPhase

    if ((age is 5) or (age is 12) or (age is 19) or (age is 26))
        moonPhase = "2 days before " + moonPhase

    if ((age is 6) or (age is 13) or (age is 20) or (age is 27))
        moonPhase = "1 day before " + moonPhase

    moonPhase

MoonPhase = exports? and exports or @MoonPhase = {}

class MoonPhase.Calculator
    getMoonDays: (date) ->
        jd = jdn date 
        moonElong jd

    getMoonPhase: (date) ->      
        jd = jdn date 
        getMoonPhase( moonElong jd )
```

### 讨论

此代码显示一个月球相位计算器对象的方法有两种。计算器 -> getmoonphase 将返回一用个文本表示的日期的月球相位。

这可以用在浏览器和 Node.js 中。

```
$ node
> var MoonPhase = require('./moonPhase.js');
 undefined
> var calc = new MoonPhase.Calculator();
 undefined
> calc.getMoonPhase(new Date());
 'full moon'
> calc.getMoonPhase(new Date(1972, 6, 30));
 '3 days before last quarter'
```