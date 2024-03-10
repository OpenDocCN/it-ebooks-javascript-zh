# 7.3 SVG 图像

SVG 是“可缩放矢量图”（Scalable Vector Graphics）的缩写，是一种描述向量图形的 XML 格式的标记化语言。也就是说，SVG 本质上是文本文件，格式采用 XML，可以在浏览器中显示出矢量图像。由于结构是 XML 格式，使得它可以插入 HTML 文档，成为 DOM 的一部分，然后用 JavaScript 和 CSS 进行操作。

相比传统的图像文件格式（比如 JPG 和 PNG），SVG 图像的优势就是文件体积小，并且放大多少倍都不会失真，因此非常合适用于网页。

SVG 图像可以用 Adobe 公司的 Illustrator 软件、开源软件 Inkscape 等生成。目前，所有主流浏览器都支持，对于低于 IE 9 的浏览器，可以使用第三方的[polyfills 函数库](https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-browser-Polyfills#svg)。

*   插入 SVG 文件
*   svg 格式
*   SVG 文件的 JavaScript 操作
    *   获取 SVG DOM
    *   读取 svg 源码
    *   将 svg 图像转为 canvas 图像
    *   实例
    *   参考链接

## 插入 SVG 文件

SVG 插入网页的方法有多种，可以用在 img、object、embed、iframe 等标签，以及 CSS 的 background-image 属性。

```
<img src="circle.svg">
<object id="object" data="circle.svg" type="image/svg+xml"></object>
<embed id="embed" src="icon.svg" type="image/svg+xml">
<iframe id="iframe" src="icon.svg"></iframe>
```

上面是四种在网页中插入 SVG 图像的方式。

此外，SVG 文件还可以插入其他 DOM 元素，比如 div 元素，请看下面的例子（使用了 jQuery 函数库）。

```
<div id="stage"></div>

<script>
$("#stage").load('icon.svg',function(response){
  $(this).addClass("svgLoaded");
  if(!response){
    // 加载失败的处理代码
  }
});
</script>
```

## svg 格式

SVG 文件采用 XML 格式，就是普通的文本文件。

```
<svg width="300" height="180">
  <circle cx="30"  cy="50" r="25" />
  <circle cx="90"  cy="50" r="25" class="red" />
  <circle cx="150" cy="50" r="25" class="fancy" />
</svg>
```

上面的 svg 文件，定义了三个圆，它们的 cx、cy、r 属性分别为 x 坐标、y 坐标和半径。利用 class 属性，可以为这些圆指定样式。

```
.red {
  fill: red; /* not background-color! */
}

.fancy {
  fill: none;
  stroke: black; /* similar to border-color */
  stroke-width: 3pt; /* similar to border-width */
}
```

上面代码中，fill 属性表示填充色，stroke 属性表示描边色，stroke-width 属性表示边线宽度。

除了 circle 标签表示圆，SVG 文件还可以使用表示其他形状的标签。

```
<svg>
  <line x1="0" y1="0" x2="200" y2="0" style="stroke:rgb(0,0,0);stroke-width:1"/></line>
  <rect x="0" y="0" height="100" width="200" style="stroke: #70d5dd; fill: #dd524b" />
  <ellipse cx="60" cy="60" ry="40" rx="20" stroke="black" stroke-width="5" fill="silver"/></ellipse>
    <polygon fill="green" stroke="orange" stroke-width="10" points="350, 75  379,161 469,161 397,215 423,301 350,250 277,301 303,215 231,161 321,161"/><polygon>
    <path id="path1" d="M160.143,196c0,0,62.777-28.033,90-17.143c71.428,28.572,73.952-25.987,84.286-21.428" style="fill:none;stroke:2;"></path>  
</svg>
```

上面代码中，line、rect、ellipse、polygon 和 path 标签，分别表示线条、矩形、椭圆、多边形和路径。

g 标签用于将多个形状组成一组，表示 group。

```
<svg width="300" height="180">
  <g transform="translate(5, 15)">
    <text x="0" y="0">Howdy!</text>
    <path d="M0,50 L50,0 Q100,0 100,50"
      fill="none" stroke-width="3" stroke="black" />
  </g>
</svg>
```

## SVG 文件的 JavaScript 操作

### 获取 SVG DOM

如果使用 img 标签插入 SVG 文件，则无法获取 SVG DOM。使用 object、iframe、embed 标签，可以获取 SVG DOM。

```
var svgObject = document.getElementById("object").contentDocument;
var svgIframe = document.getElementById("iframe").contentDocument;
var svgEmbed = document.getElementById("embed").getSVGDocument();
```

由于 svg 文件就是一般的 XML 文件，因此可以用 DOM 方法，选取页面元素。

```
// 改变填充色
document.getElementById("theCircle").style.fill = "red";

// 改变元素属性
document.getElementById("theCircle").setAttribute("class", "changedColors");

// 绑定事件回调函数
document.getElementById("theCircle").addEventListener("click", function() {
   console.log("clicked")
});
```

### 读取 svg 源码

由于 svg 文件就是一个 XML 代码的文本文件，因此可以通过读取 XML 代码的方式，读取 svg 源码。

假定网页中有一个 svg 元素。

```
<div id="svg-container">
    <svg   xml:space="preserve" width="500" height="440">
        <!-- svg code -->
    </svg>
</div>
```

使用 XMLSerializer 实例的 serializeToString 方法，获取 svg 元素的代码。

```
var svgString = new XMLSerializer().serializeToString(document.querySelector('svg'));
```

### 将 svg 图像转为 canvas 图像

首先，需要新建一个 img 对象，将 svg 图像指定到该 img 对象的 src 属性。

```
var img = new Image();
var svg = new Blob([svgString], {type: "image/svg+xml;charset=utf-8"});

var DOMURL = self.URL || self.webkitURL || self;
var url = DOMURL.createObjectURL(svg);

img.src = url;
```

然后，当图像加载完成后，再将它绘制到 canvas 元素。

```
img.onload = function() {
    var canvas = document.getElementById("canvas");
    var ctx = canvas.getContext("2d");
    ctx.drawImage(img, 0, 0);
};
```

## 实例

假定我们要将下面的表格画成图形。

| Date | Amount |
| --- | --- |
| 2014-01-01 | $10 |
| 2014-02-01 | $20 |
| 2014-03-01 | $40 |
| 2014-04-01 | $80 |

上面的图形，可以画成一个坐标系，Date 作为横轴，Amount 作为纵轴，四行数据画成一个数据点。

```
<svg width="350" height="160">
  <g class="layer" transform="translate(60,10)">
    <circle r="5" cx="0"   cy="105" />
    <circle r="5" cx="90"  cy="90"  />
    <circle r="5" cx="180" cy="60"  />
    <circle r="5" cx="270" cy="0"   />

    <g class="y axis">
      <line x1="0" y1="0" x2="0" y2="120" />
      <text x="-40" y="105" dy="5">$10</text>
      <text x="-40" y="0"   dy="5">$80</text>
    </g>
    <g class="x axis" transform="translate(0, 120)">
      <line x1="0" y1="0" x2="270" y2="0" />
      <text x="-30"   y="20">January 2014</text>
      <text x="240" y="20">April</text>
    </g>
  </g>
</svg>
```

## 参考链接

*   Jon McPartland, [An introduction to SVG animation](http://bigbitecreative.com/introduction-svg-animation/)
*   Alexander Goedde, [SVG - Super Vector Graphics](http://tavendo.com/blog/post/super-vector-graphics/)
*   Joseph Wegner, [Learning SVG](http://flippinawesome.org/2014/02/03/learning-svg/)
*   biovisualize, [Direct svg to canvas to png conversion](http://bl.ocks.org/biovisualize/8187844)