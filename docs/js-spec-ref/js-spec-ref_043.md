# 6.4 history 对象

*   概述
*   history.pushState()，history.replaceState()
*   history.state 属性
*   popstate 事件
*   参考链接

## 概述

浏览器窗口有一个 history 对象，用来保存浏览历史。比如，该窗口先后访问了三个地址，那么 history 对象就包括三项，length 属性等于 3。

```js
history.length // 3
```

history 对象提供了一系列方法，允许在浏览历史之间移动。

*   back()：移动到上一个访问页面，等同于浏览器的后退键。
*   forward()：移动到下一个访问页面，等同于浏览器的前进键。
*   go()：接受一个整数作为参数，移动到该整数指定的页面，比如 go(1)相当于 forward()，go(-1)相当于 back()。

```js
history.back();
history.forward();
history.go(-2);
```

如果移动的位置超出了访问历史的边界，以上三个方法并不报错，而是默默的失败。

以下命令相当于刷新当前页面。

```js
history.go(0);
```

## history.pushState()，history.replaceState()

HTML5 为 history 对象添加了两个新方法，history.pushState() 和 history.replaceState()，用来在浏览历史中添加和修改记录。所有主流浏览器都支持该方法（包括 IE10）。

```js
if (!!(window.history && history.pushState)){
  // 支持 History API
} else {
  // 不支持
}
```

上面代码可以用来检查，当前浏览器是否支持 History API。如果不支持的话，可以考虑使用 Polyfill 库[History.js](https://github.com/browserstate/history.js/)。

history.pushState 方法接受三个参数，依次为：

*   state：一个与指定网址相关的状态对象，popstate 事件触发时，该对象会传入回调函数。如果不需要这个对象，此处可以填 null。
*   title：新页面的标题，但是所有浏览器目前都忽略这个值，因此这里可以填 null。
*   url：新的网址，必须与当前页面处在同一个域。浏览器的地址栏将显示这个网址。

假定当前网址是`example.com/1.html`，我们使用 pushState 方法在浏览记录（history 对象）中添加一个新记录。

```js
var stateObj = { foo: "bar" };

history.pushState(stateObj, "page 2", "2.html");
```

添加上面这个新记录后，浏览器地址栏立刻显示`example.com/2.html`，但并不会跳转到 2.html，甚至也不会检查 2.html 是否存在，它只是成为浏览历史中的最新记录。假定这时你访问了 google.com，然后点击了倒退按钮，页面的 url 将显示 2.html，但是内容还是原来的 1.html。你再点击一次倒退按钮，url 将显示 1.html，内容不变。

> 注意，pushState 方法不会触发页面刷新。

如果 pushState 的 url 参数，设置了一个当前网页的#号值（即 hash），并不会触发 hashchange 事件。如果设置了一个非同域的网址，则会报错。

```js
// 报错
history.pushState(null, null, 'https://twitter.com/hello');
```

上面代码中，pushState 想要插入一个非同域的网址，导致报错。这样设计的目的是，防止恶意代码让用户以为他们是在另一个网站上。

history.replaceState 方法的参数与 pushState 方法一模一样，区别是它修改浏览历史中当前页面的值。假定当前网页是 example.com/example.html。

```js
history.pushState({page: 1}, "title 1", "?page=1");
history.pushState({page: 2}, "title 2", "?page=2");
history.replaceState({page: 3}, "title 3", "?page=3");
history.back(); // url 显示为 http://example.com/example.html?page=1
history.back(); // url 显示为 http://example.com/example.html
history.go(2);  // url 显示为 http://example.com/example.html?page=3
```

## history.state 属性

history.state 属性保存当前页面的 state 对象。

```js
history.pushState({page: 1}, "title 1", "?page=1");

history.state
// { page: 1 }
```

## popstate 事件

每当同一个文档的浏览历史（即 history 对象）出现变化时，就会触发 popstate 事件。需要注意的是，仅仅调用 pushState 方法或 replaceState 方法 ，并不会触发该事件，只有用户点击浏览器倒退按钮和前进按钮，或者使用 JavaScript 调用 back、forward、go 方法时才会触发。另外，该事件只针对同一个文档，如果浏览历史的切换，导致加载不同的文档，该事件也不会触发。

使用的时候，可以为 popstate 事件指定回调函数。这个回调函数的参数是一个 event 事件对象，它的 state 属性指向 pushState 和 replaceState 方法为当前 url 所提供的状态对象（即这两个方法的第一个参数）。

```js
window.onpopstate = function(event) {
  console.log("location: " + document.location);
  console.log("state: " + JSON.stringify(event.state));
};

// 或者

window.addEventListener('popstate', function(event) {  
  console.log("location: " + document.location);
  console.log("state: " + JSON.stringify(event.state));  
});
```

上面代码中的 event.state，就是通过 pushState 和 replaceState 方法，为当前 url 绑定的 state 对象。

这个 state 对象也可以直接通过 history 对象读取。

```js
var currentState = history.state;
```

另外，需要注意的是，当页面第一次加载的时候，在 onload 事件发生后，Chrome 和 Safari 浏览器（Webkit 核心）会触发 popstate 事件，而 Firefox 和 IE 浏览器不会。

## 参考链接

*   MOZILLA DEVELOPER NETWORK，[Manipulating the browser history](https://developer.mozilla.org/en-US/docs/DOM/Manipulating_the_browser_history)
*   MOZILLA DEVELOPER NETWORK，[window.onpopstate](https://developer.mozilla.org/en-US/docs/DOM/window.onpopstate)
*   Johnny Simpson, [Controlling History: The HTML5 History API And ‘Selective’ Loading](http://www.inserthtml.com/2013/06/history-api/)
*   Louis Lazaris, [HTML5 History API: A Syntax Primer](http://www.impressivewebs.com/html5-history-api-syntax/)