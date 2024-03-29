# 6.9 Web Notifications API

*   概述
*   Notification 对象的属性和方法
    *   Notification.permission
    *   Notification.requestPermission()
    *   Notification 实例对象
        *   Notification 构造函数
        *   实例对象的事件
        *   close 方法
    *   参考链接

## 概述

Notification API 是浏览器的通知接口，用于在用户的桌面（而不是网页上）显示通知信息，桌面电脑和手机都适用，比如通知用户收到了一封 Email。具体的实现形式由浏览器自行部署，对于手机来说，一般显示在顶部的通知栏。

如果网页代码调用这个 API，浏览器会询问用户是否接受。只有在用户同意的情况下，通知信息才会显示。

下面的代码用于检查浏览器是否支持这个 API。

```js
if (window.Notification) {
  // 支持
} else {
  // 不支持
}
```

目前，Chrome 和 Firefox 在桌面端部署了这个 API，Firefox 和 Blackberry 在手机端部署了这个 API。

```js
if(window.Notification && Notification.permission !== "denied") {
    Notification.requestPermission(function(status) {
        var n = new Notification('通知标题', { body: '这里是通知内容！' }); 
    });
}
```

上面代码检查当前浏览器是否支持 Notification 对象，并且当前用户准许使用该对象，然后调用 Notification.requestPermission 方法，向用户弹出一条通知。

## Notification 对象的属性和方法

### Notification.permission

Notification.permission 属性，用于读取用户给予的权限，它是一个只读属性，它有三种状态。

*   default：用户还没有做出任何许可，因此不会弹出通知。
*   granted：用户明确同意接收通知。
*   denied：用户明确拒绝接收通知。

### Notification.requestPermission()

Notification.requestPermission 方法用于让用户做出选择，到底是否接收通知。它的参数是一个回调函数，该函数可以接收用户授权状态作为参数。

```js
Notification.requestPermission(function (status) {
  if (status === "granted") {
    var n = new Notification("Hi!");
  } else {
    alert("Hi!");
  }
});
```

上面代码表示，如果用户拒绝接收通知，可以用 alert 方法代替。

## Notification 实例对象

### Notification 构造函数

Notification 对象作为构造函数使用时，用来生成一条通知。

```js
var notification = new Notification(title, options);
```

Notification 构造函数的 title 属性是必须的，用来指定通知的标题，格式为字符串。options 属性是可选的，格式为一个对象，用来设定各种设置。该对象的属性如下：

*   dir：文字方向，可能的值为 auto、ltr（从左到右）和 rtl（从右到左），一般是继承浏览器的设置。
*   lang：使用的语种，比如 en-US、zh-CN。
*   body：通知内容，格式为字符串，用来进一步说明通知的目的。。
*   tag：通知的 ID，格式为字符串。一组相同 tag 的通知，不会同时显示，只会在用户关闭前一个通知后，在原位置显示。
*   icon：图表的 URL，用来显示在通知上。

上面这些属性，都是可读写的。

下面是一个生成 Notification 实例对象的例子。

```js
var notification = new Notification('收到新邮件', {
  body: '您总共有 3 封未读邮件。'
});

notification.title // "收到新邮件"
notification.body // "您总共有 3 封未读邮件。"
```

### 实例对象的事件

Notification 实例会触发以下事件。

*   show：通知显示给用户时触发。
*   click：用户点击通知时触发。
*   close：用户关闭通知时触发。
*   error：通知出错时触发（大多数发生在通知无法正确显示时）。

这些事件有对应的 onshow、onclick、onclose、onerror 方法，用来指定相应的回调函数。addEventListener 方法也可以用来为这些事件指定回调函数。

```js
notification.onshow = function() {
  console.log('Notification shown');
};
```

### close 方法

Notification 实例的 close 方法用于关闭通知。

```js
var n = new Notification("Hi!");

// 手动关闭
n.close();

// 自动关闭
n.onshow = function () { 
  setTimeout(n.close.bind(n), 5000); 
}
```

上面代码说明，并不能从通知的 close 事件，判断它是否为用户手动关闭。

## 参考链接

*   Aurelio De Rosa, [An Introduction to the Web Notifications API](http://www.sitepoint.com/introduction-web-notifications-api/)
*   MDN, [Notification](https://developer.mozilla.org/en-US/docs/Web/API/Notification)