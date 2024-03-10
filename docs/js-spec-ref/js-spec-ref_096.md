# 12.5 如何做到 jQuery-free？

*   概述
*   选取 DOM 元素
*   DOM 操作
*   事件的监听
*   事件的触发
*   $(document).ready
*   attr 方法
*   addClass 方法
*   CSS
*   数据储存
*   Ajax
*   动画
*   替代方案
*   参考链接

## 概述

jQuery 是最流行的 JavaScript 工具库。据[统计](http://w3techs.com/technologies/details/js-jquery/all/all)，目前全世界 57.3%的网站使用它。也就是说，10 个网站里面，有 6 个使用 jQuery。如果只考察使用工具库的网站，这个比例就会上升到惊人的 91.7%。

jQuery 如此受欢迎，以至于有被滥用的趋势。许多开发者不管什么样的项目，都一股脑使用 jQuery。但是，jQuery 本质只是一个中间层，提供一套统一易用的 DOM 操作接口，消除浏览器之间的差异。多了这一层中间层，操作的性能和效率多多少少会打一些折扣。

2006 年，jQuery 诞生的时候，主要是为了解决 IE6 与标准的不兼容问题。如今的[情况](http://en.wikipedia.org/wiki/Usage_share_of_web_browsers)已经发生了很大的变化。IE 的市场份额不断下降，以 ECMAScript 为基础的 JavaScript 标准语法，正得到越来越广泛的支持，不同浏览器对标准的支持越来越好、越来越趋同。开发者直接使用 JavaScript 标准语法，就能同时在各大浏览器运行，不再需要通过 jQuery 获取兼容性。

另一方面，jQuery 臃肿的[体积](http://mathiasbynens.be/demo/jquery-size)也让人头痛不已。jQuery 2.0 的原始大小为 235KB，优化后为 81KB；如果是支持 IE6、7、8 的 jQuery 1.8.3，原始大小为 261KB，优化后为 91KB。即使有 CDN，浏览器加载这样大小的脚本，也会产生不小的开销。

所以，对于一些不需要支持老式浏览器的小型项目来说，不使用 jQuery，直接使用 DOM 原生接口，可能是更好的选择。开发者有必要了解，jQuery 的一些常用操作所对应的 DOM 写法。而且，理解 jQuery 背后的原理，会帮助你更好地使用 jQuery。要知道有一种极端的说法是，如果你不理解一样东西，就不要使用它。

下面就探讨如何用 JavaScript 标准语法，取代 jQuery 的一些主要功能，做到 jQuery-free。

## 选取 DOM 元素

jQuery 的核心是通过各种选择器，选中 DOM 元素，可以用 querySelectorAll 方法模拟这个功能。

```js
var $ = document.querySelectorAll.bind(document);
```

这里需要注意的是，querySelectorAll 方法返回的是 NodeList 对象，它很像数组（有数字索引和 length 属性），但不是数组，不能使用 pop、push 等数组特有方法。如果有需要，可以考虑将 Nodelist 对象转为数组。

```js
myList = Array.prototype.slice.call(myNodeList);
```

## DOM 操作

DOM 本身就具有很丰富的操作方法，可以取代 jQuery 提供的操作方法。

获取父元素。

```js
// jQuery 写法
$("#elementID").parent()

// DOM 写法
document.getElementById("elementID").parentNode
```

获取下一个同级元素。

```js
// jQuery 写法
$("#elementID").next()

// DOM 写法
document.getElementById("elementID").nextSibling
```

尾部追加 DOM 元素。

```js
// jQuery 写法
$(parent).append($(child));

// DOM 写法
parent.appendChild(child)
```

头部插入 DOM 元素。

```js
// jQuery 写法
$(parent).prepend($(child));

// DOM 写法
parent.insertBefore(child, parent.childNodes[0])
```

生成 DOM 元素。

```js
// jQuery 写法
$("<p>")

// DOM 写法
document.createElement("p")
```

删除 DOM 元素。

```js
// jQuery 写法
$(child).remove()

// DOM 写法
child.parentNode.removeChild(child)
```

清空子元素。

```js
// jQuery 写法
$("#elementID").empty()

// DOM 写法
var element = document.getElementById("elementID");
while(element.firstChild) element.removeChild(element.firstChild);
```

检查是否有子元素。

```js
// jQuery 写法
if (!$("#elementID").is(":empty")){}

// DOM 写法
if (document.getElementById("elementID").hasChildNodes()){}
```

克隆元素。

```js
// jQuery 写法
$("#elementID").clone()

// DOM 写法
document.getElementById("elementID").cloned(true)
```

## 事件的监听

jQuery 使用 on 方法，监听事件和绑定回调函数。

```js
$('button').on('click', function(){
    ajax( ... );
});
```

完全可以自己定义 on 方法，将它指向 addEventListener 方法。

```js
Element.prototype.on = Element.prototype.addEventListener;
```

为了使用方便，可以在 NodeList 对象上也部署这个方法。

```js
NodeList.prototype.on = function (event, fn) {

    []['forEach'].call(this, function (el) {
        el.on(event, fn);
    });

    return this;

};
```

取消事件绑定的 off 方法，也可以自己定义。

```js
Element.prototype.off = Element.prototype.removeEventListener;
```

## 事件的触发

jQuery 的 trigger 方法则需要单独部署，相对复杂一些。

```js
Element.prototype.trigger = function (type, data) {
    var event = document.createEvent('HTMLEvents');
    event.initEvent(type, true, true);
    event.data = data || {};
    event.eventName = type;
    event.target = this;
    this.dispatchEvent(event);
    return this;
};
```

在 NodeList 对象上也部署这个方法。

```js
NodeList.prototype.trigger = function (event) {

    []['forEach'].call(this, function (el) {

        el'trigger';

    });

    return this;
};
```

## $(document).ready

DOM 加载完成，会触发 DOMContentLoaded 事件，等同于 jQuery 的$(document).ready 方法。

```js
document.addEventListener("DOMContentLoaded", function() {
    // ...
});
```

不过，目前的最佳实践，是将 JavaScript 脚本文件都放在页面底部加载。这样的话，其实$(document).ready 方法（可以简写为$(function)）已经不必要了，因为等到运行的时候，DOM 对象已经生成了。

## attr 方法

jQuery 使用 attr 方法，读写网页元素的属性。

```js
$("#picture").attr("src", "http://url/to/image")
```

DOM 提供 getAttribute 和 setAttribute 方法读写元素属性。

```js
imgElement.setAttribute("src", "http://url/to/image")
```

DOM 还允许直接读取属性值，写法要简洁许多。

```js
imgElement.src = "http://url/to/image";
```

> 需要注意的是，文本框元素（input）的 this.value 返回的是输入框中的值，链接元素（a 标签）的 this.href 返回的是绝对 URL。如果需要用到这两个网页元素的属性准确值，可以用 this.getAttribute('value')和 this.getAttibute('href')。

## addClass 方法

jQuery 的 addClass 方法，用于为 DOM 元素添加一个 class。

```js
$('body').addClass('hasJS');
```

DOM 元素本身有一个可读写的 className 属性，可以用来操作 class。

```js
document.body.className = 'hasJS';

// or

document.body.className += ' hasJS';
```

HTML 5 还提供一个 classList 对象，功能更强大（IE 9 不支持）。

```js
document.body.classList.add('hasJS');

document.body.classList.remove('hasJS');

document.body.classList.toggle('hasJS');

document.body.classList.contains('hasJS');
```

## CSS

jQuery 的 css 方法，用来设置网页元素的样式。

```js
$(node).css( "color", "red" );
```

DOM 元素有一个 style 属性，可以直接操作。

```js
element.style.color = "red”;;

// or

element.style.cssText += 'color:red';
```

## 数据储存

jQuery 对象可以储存数据。

```js
$("body").data("foo", 52);
```

HTML 5 有一个 dataset 对象，也有类似的功能（IE 10 不支持），不过只能保存字符串。

```js
element.dataset.user = JSON.stringify(user);

element.dataset.score = score;
```

## Ajax

jQuery 的 ajax 方法，用于异步操作。

```js
$.ajax({
    type: "POST",
    url: "some.php",
    data: { name: "John", location: "Boston" }
}).done(function( msg ) {
    alert( "Data Saved: " + msg );
});
```

我们自定义一个 ajax 函数，简单模拟 jQuery 的 ajax 方法。

```js
function ajax(url, opts){
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function(){
        var completed = 4;
        if(xhr.readyState === completed){
            if(xhr.status === 200){
                opts.success(xhr.responseText, xhr);
            }else{
                opts.error(xhr.responseText, xhr);
            }
        }
    };
    xhr.open(opts.method, url, true);
    xhr.send(opts.data);
}
```

使用的时候，除了网址，还需要传入一个自己构造的 option 对象。

```js
ajax('/foo', { 
    method: 'GET',
    success: function(response){
        console.log(response);
    },
    error: function(response){
        console.log(response);
    }
});
```

## 动画

jQuery 的 animate 方法，用于生成动画效果。

```js
$foo.animate('slow', { x: '+=10px' })
```

jQuery 的动画效果，很大部分基于 DOM。但是目前，CSS 3 的动画远比 DOM 强大，所以可以把动画效果写进 CSS，然后通过操作 DOM 元素的 class，来展示动画。

```js
foo.classList.add('animate')
```

如果需要对动画使用回调函数，CSS 3 也定义了相应的事件。

```js
el.addEventListener("webkitTransitionEnd", transitionEnded);

el.addEventListener("transitionend", transitionEnded);
```

## 替代方案

由于 jQuery 体积过大，替代方案层出不穷。

其中，最有名的是[zepto.js](http://zeptojs.com/)。它的设计目标是以最小的体积，做到最大兼容 jQuery 的 API。它的 1.0 版的原始大小是 55KB，优化后是 29KB，gzip 压缩后为 10KB。

如果不求最大兼容，只希望模拟 jQuery 的基本功能。那么，[min.js](https://github.com/remy/min.js)优化后只有 200 字节，而[dolla](https://github.com/lelandrichardson/dolla)优化后是 1.7KB。

此外，jQuery 本身也采用模块设计，可以只选择使用自己需要的模块。具体做法参见 jQuery 的[github 网站](https://github.com/jquery/jquery)，或者使用专用的[Web 界面](http://projects.jga.me/jquery-builder/)。

## 参考链接

*   Remy Sharp，[I know jQuery. Now what?](http://remysharp.com/2013/04/19/i-know-jquery-now-what/)
*   Hemanth.HM，[Power of Vanilla JS](http://h3manth.com/new/blog/2013/power-of-vanilla-js/)
*   Burke Holland, [5 Things You Should Stop Doing With jQuery](http://flippinawesome.org/2013/05/06/5-things-you-should-stop-doing-with-jquery/)
*   Burke Holland, [Out-Growing jQuery](http://tech.pro/tutorial/1385/out-growing-jquery)
*   Nicolas Bevacqua, [Uncovering the Native DOM API](http://flippinawesome.org/2013/06/17/uncovering-the-native-dom-api/)
*   Pony Foo, [Getting Over jQuery](http://blog.ponyfoo.com/2013/07/09/getting-over-jquery)
*   Hemanth.HM, [JavaScript vs Jquery+CoffeeScript](http://h3manth.com/notes/jq-cs.html)