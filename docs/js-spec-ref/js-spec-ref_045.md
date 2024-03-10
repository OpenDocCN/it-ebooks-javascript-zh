# 6.6 同域限制和 window.postMessage 方法

*   概述
*   iframe 与主页面的通信
*   参考链接

## 概述

所谓“同域限制”指的是，出于安全考虑，浏览器只允许脚本与同样协议、同样端口、同样域名的地址进行通信。比如，www1.example.com 页面上面的脚本，只能与该域名（相同协议、相同端口）进行通信，如果与 www2.example.com 通信，浏览器就会报错（不过可以设置两者的 document.domain 为相同的值）。这是为了防止恶意脚本将用户信息发往第三方网站。

window.postMessage 方法就是用来在某种程度上，绕过同域限制，实现不同域名的窗口（包括 iframe 窗口）之间的通信。它的格式如下。

```js
targetWindow.postMessage(message, targetURL[, transferObject]);
```

上面代码的 targetWindow 是指向目标窗口的变量，message 是要发送的信息，targetURL 是指定目标窗口的网址，不符合该网址就不发送信息，transferObject 则是跟随信息一起发送的 Transferable 对象。

下面是一个 postMessage 方法的实例。假定当前网页弹出一个新窗口。

```js
var popup = window.open(...popup details...);

popup.postMessage("Hello World!", "http://example.org");
```

上面代码的 postMessage 方法的第一个参数是实际发送的信息，第二个参数是指定发送对象的域名必须是 example.org。如果对方窗口不是这个域名，信息不会发送出去。

然后，在当前网页上监听 message 事件。

```js
window.addEventListener("message", receiveMessage, false);

function receiveMessage(event) {
    if (event.origin !== "http://example.org")
    return;

    if (event.data == 'Hello World') {
      event.source.postMessage('Hello', event.origin);
    } else {
      console.log(event.data);
    }

}
```

上面代码指定 message 事件的回调函数为 receiveMessage，一旦收到其他窗口发来的信息，receiveMessage 函数就会被调用。receiveMessage 函数接受一个 event 事件对象作为参数，该对象的 origin 属性表示信息的来源网址，如果该网址不符合要求，就立刻返回，不再进行下一步处理。event.data 属性则包含了实际发送过来的信息，event.source 属性，指向当前网页发送信息的窗口对象。

最后，在 popup 窗口中部署下面的代码。

```js
// popup 窗口

function receiveMessage(event) {
  event.source.postMessage("Nice to see you!", "*");
}

window.addEventListener("message", receiveMessage, false);
```

上面代码有几个地方需要注意。首先，receiveMessage 函数里面没有过滤信息的来源，任意网址发来的信息都会被处理。其次，postMessage 方法中指定的目标窗口的网址是一个星号，表示该信息可以向任意网址发送。通常来说，这两种做法是不推荐的，因为不够安全，可能会被恶意利用。

所有浏览器都支持这个方法，但是 IE 8 和 IE 9 只允许 postMessage 方法与 iFrame 窗口通信，不能与新窗口通信。IE 10 允许与新窗口通信，但是只能使用 IE 特有的[MessageChannel 对象](http://msdn.microsoft.com/en-us/library/windows/apps/hh441303.aspx)。

## iframe 与主页面的通信

iframe 中的网页，如果与主页面来自同一个域，通过设置 document.domain 属性，可以使用 postMessage 方法实现双向通信。

下面是一个 LocalStorage 的例子。LocalStorage 只能用同一个域名的网页读写，但是如果 iframe 是主页面的子域名，主页面就可以通过 postMessage 方法，读写 iframe 网页设置的 LocalStorage 数据。

iframe 页面的代码如下。

```js
document.domain = "domain.com";
window.onmessage = function(e) {
  if (e.origin !== "http://domain.com") {
    return;
  }
  var payload = JSON.parse(e.data);
  localStorage.setItem(payload.key, JSON.stringify(payload.data));
};
```

主页面的代码如下。

```js
window.onload = function() {
    var win = document.getElementsByTagName('iframe')[0].contentWindow;
    var obj = {
       name: "Jack"
    };
    win.postMessage(JSON.stringify({key: 'storage', data: obj}), "*");
};
```

上面的代码已经可以实现，主页面向 iframe 传入数据。如果还想读取或删除数据，可以进一步加强代码。

加强版的 iframe 代码如下。

```js
document.domain = "domain.com";
window.onmessage = function(e) {
    if (e.origin !== "http://domain.com") {
        return;
    }
    var payload = JSON.parse(e.data);
    switch(payload.method) {
        case 'set':
            localStorage.setItem(payload.key, JSON.stringify(payload.data));
            break;
        case 'get':
            var parent = window.parent;
            var data = localStorage.getItem(payload.key);
            parent.postMessage(data, "*");
            break;
        case 'remove':
            localStorage.removeItem(payload.key);
            break;
    }
};
```

加强版的主页面代码如下。

```js
window.onload = function() {
    var win = document.getElementsByTagName('iframe')[0].contentWindow;
    var obj = {
       name: "Jack"
    };
    // 存入对象
    win.postMessage(JSON.stringify({key: 'storage', method: "set", data: obj}), "*");
    // 读取以前存取的对象
    win.postMessage(JSON.stringify({key: 'storage', method: "get"}), "*");
    window.onmessage = function(e) {
        if (e.origin != "http://sub.domain.com") {
            return;
        }
        // 下面会输出"Jack"
        console.log(JSON.parse(e.data).name);
    };
};
```

## 参考链接

*   Mozilla Developer Network, [Window.postMessage](https://developer.mozilla.org/en-US/docs/Web/API/window.postMessage)
*   Jakub Jankiewicz, [Cross-Domain LocalStorage](http://jcubic.wordpress.com/2014/06/20/cross-domain-localstorage/)
*   David Baron, [setTimeout with a shorter delay](http://dbaron.org/log/20100309-faster-timeouts): 利用 window.postMessage 可以实现 0 毫秒触发回调函数