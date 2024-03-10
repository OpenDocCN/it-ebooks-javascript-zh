# 11.2 Modernizr

*   概述
*   CSS 的新增 class
*   JavaScript 侦测
*   加载器
*   参考链接

## 概述

随着 HTML5 和 CSS3 加入越来越多的模块，检查各种浏览器是否支持这些模块，成了一大难题。Modernizr 就是用来解决这个问题的一个 JavaScript 库。

首先，从 modernizr.com 下载这个库。下载的时候，可以选择所需要的模块。然后，将它插入 HTML 页面的头部，放在 head 标签之中。

```js
<!DOCTYPE html>
<html class="no-js" lang="en">

<head>
  <meta charset="utf-8">
  <script src="js/modernizr.js"></script>
</head>

</html>
```

## CSS 的新增 class

使用 Modernizr 以后，首先会把 html 元素的 class 替换掉。以 chrome 浏览器为例，新增的 class 大概是下面的样子。

```js
<html class="js no-touch postmessage history multiplebgs boxshadow opacity cssanimations csscolumns cssgradients csstransforms csstransitions fontface localstorage sessionstorage svg inlinesvg blobbuilder blob bloburls download formdata">
```

IE 7 则是这样：

```js
<html class="js no-touch postmessage no-history no-multiplebgs no-boxshadow no-opacity no-cssanimations no-csscolumns no-cssgradients no-csstransforms no-csstransitions fontface localstorage sessionstorage no-svg no-inlinesvg wf-loading no-blobbuilder no-blob no-bloburls no-download no-formdata">
```

然后，就可以针对不同的 CSS class，指定不同的样式。

```js
.button {
   background: #000;
   opacity: 0.75;
}

.no-opacity .button {
   background: #444;
}
```

## JavaScript 侦测

除了提供新增的 CSS class，Modernizr 还提供 JavaScript 方法，用来侦测浏览器是否支持某个功能。

```js
Modernizr.cssgradients; //True in Chrome, False in IE7

Modernizr.fontface; //True in Chrome, True in IE7

Modernizr.geolocation; //True in Chrome, False in IE7

if (Modernizr.canvas){
    // 支持 canvas
} else {
   // 不支持 canvas
}

if (Modernizr.touch){
    // 支持触摸屏
} else {
   // 不支持触摸屏
}
```

## 加载器

Modernizr 允许根据 Javascript 侦测的不同结果，加载不同的脚本文件。

```js
Modernizr.load({
  test :        Modernizr.localstorage,
  yep  :        'localStorage.js',
  nope :        'alt-storageSystem.js',
  complete :    function () { enableStorgeSaveUI();}
});
```

Modernizr.load 方法用来加载脚本。它的属性如下：

*   test：用来测试浏览器是否支持某个属性。
*   yep：如果浏览器支持该属性，加载的脚本。
*   nope：如果浏览器不支持该属性，加载的脚本。
*   complete：加载完成后，运行的 JavaScript 代码。

可以指定在支持某个功能的情况，所要加载的 JavaScript 脚本和 CSS 样式。

```js
Modernizr.load({
  test : Modernizr.touch,
  yep :  ['js/touch.js', 'css/touchStyles.css']
});
```

## 参考链接

*   Chris Griffith, [Up and running with Modernizr](http://www.adobe.com/devnet/html5/articles/up-and-running-with-modernizr.html)