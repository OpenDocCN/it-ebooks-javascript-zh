# 11.4 D3.js

D3.js 是一个用于网页作图、生成互动图形的 JavaScript 函数库。它提供一个 d3 对象，所有方法都通过这个对象调用。

*   操作网页元素
*   生成 svg 元素
*   生成图形
    *   选中对象集
    *   绑定数据
    *   操作 SVG 图形
    *   加载 XML 文件
    *   参考链接

## 操作网页元素

D3 提供了一系列操作网页元素的方法，很类似 jQuery，也是先选中某个元素（select 方法），然后对其进行某种操作。

```js
var body = d3.select("body");
var div = body.append("div");
div.html("Hello, world!");
```

select 方法用于选中一个元素，而 selectAll 方法用于选中一组元素。

```js
var section = d3.selectAll("section");
var div = section.append("div");
div.html("Hello, world!");
```

大部分 D3 的方法都返回 D3 对象的实例，这意味着可以采用链式写法。

```js
d3.select("body")
    .style("color", "black")
    .style("background-color", "white");
```

需要注意的是 append 方法返回一个新对象。

```js
d3.selectAll("section")
  .attr("class", "special")
  .append("div")
  .html("Hello, world!");
```

## 生成 svg 元素

`D3 作图需要 svg 元素，可以用 JavaScript 代码动态生成。

```js
var v = d3.select("#graph")
            .append("svg");

v.attr("width", 900).attr("height", 400);
```

## 生成图形

### 选中对象集

selectAll 方法不仅可以选中现有的网页元素，还可以选中不存在的网页元素。

```js
d3.select(".chart")
  .selectAll("div");
```

上面代码表示，selectAll 方法选中了.chart 元素下面所有现有和将来可能出现的 div 元素。

### 绑定数据

data 方法用于对选中的结果集绑定数据。

```js
var data = [4, 8, 15, 16, 23, 42, 12];

d3.select(".chart")
  .selectAll("div")
  .data(data)
  .enter().append("div")
  .style("width", function(d) { return d * 10 + "px"; })
  .text(function(d) { return d; });
```

上面代码中，enter 方法和 append 方法表示由于此时 div 元素还不存在，必须根据数据的个数将它们创造出来。style 方法和 text 方法的参数是函数，表示函数的运行结果就是设置网页元素的值。

上面代码的运行结果是生成一个条状图，但是没有对条状图的长度进行控制，下面采用 scale.linear 方法对数据长度进行设置。

```js
var data = [4, 8, 15, 16, 23, 42, 12];

var x = d3.scale.linear()
    .domain([0, d3.max(data)])
    .range([0, 420]);

d3.select(".chart")
  .selectAll("div")
  .data(data)
  .enter().append("div")
  .style("width", function(d) { return x(d) + "px"; })
  .text(function(d) { return d; });
```

## 操作 SVG 图形

使用 SVG 图形生成条形图，首先是选中矢量图格式，然后每个数据值生成一个 g 元素（group），再在每个 g 元素内部生成一个 rect 元素和 text 元素。

```js
var width = 840,
    barHeight = 20;

var x = d3.scale.linear()
    .domain([0, d3.max(dataArray)])
    .range([0, width]);

var chart = d3.select(".bar-chart-svg")
    .attr("width", width)
    .attr("height", barHeight * dataArray.length);

var bar = chart.selectAll("g")
    .data(dataArray)
  .enter().append("g")
    .attr("transform", function(d, i) { return "translate(0," + i * barHeight + ")"; });

bar.append("rect")
    .attr("width", x)
    .attr("height", barHeight - 1);

bar.append("text")
    .attr("x", function(d) { return x(d) - 3; })
    .attr("y", barHeight / 2)
    .attr("dy", ".35em")
    .text(function(d) { return d; });
```

## 加载 XML 文件

```js
d3.xml('example', 'image/svg+xml', function (error, data) {
    if (error) {
        console.log('加载 SVG 文件出错！', error);
    }
    else {
        // 处理 SVG 文件
    }
});
```

## 参考链接

*   Mike Bostock, [Let’s Make a Bar Chart](http://bost.ocks.org/mike/bar/)
*   Dana Silver, [The d3.js Bar Chart Tutorials with Github Data](http://danasilver.org/2013/12/31/d3-github-language-stats/)