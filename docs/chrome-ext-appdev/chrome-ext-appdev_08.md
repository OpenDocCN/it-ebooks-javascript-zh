# 第五章　部分高级 API

在前面的章节，我们已经接触到了 Chrome 扩展中常用的大多数 API，本章将挑选部分较为常用的高级 API 进行讲解，以便有更高要求的读者阅读。

## 5.1　下载

Chrome 提供了 downloads API，扩展可以通过此 API 管理浏览器的下载功能，包括暂停、搜索和取消等。

相对于管理下载，更令人关注的是创建下载的功能。Chrome 应用市场中之前包括很多下载页面所有图片等类似功能的扩展，大多数是将图片包含在一个网页中让用户另存为，或者是列出所有 URL 让用户自行下载。这样做明显不友好，Chrome 处于早期版本时，开发者对开放下载功能的呼声也越来越高。所以本节将重点讲解如何让扩展通过`downloads`接口创建下载，有关进一步管理下载行为的内容请感兴趣的读者自行阅读。完整有关`downloads`接口的官方文档可以通过[`developer.chrome.com/extensions/downloads`](http://developer.chrome.com/extensions/downloads)阅读。

扩展使用`downloads`接口需要在 Manifest 文件中声明`downloads`权限：

```js
"permissions": [
    "downloads"
] 
```

创建下载可以通过`downloads`中的`download`方法实现。`download`方法包含两个参数，第一个是有关下载的属性对象，包括 URL、保存位置、文件名等信息，第二个是创建成功后的回调函数。

```js
chrome.downloads.download(options, callback); 
```

其中`options`的完整结构如下：

```js
{
    url: 下载文件的 url,
    filename: 保存的文件名,
    conflictAction: 重名文件的处理方式,
    saveAs: 是否弹出另存为窗口,
    method: 请求方式（POST 或 GET），
    headers: 自定义 header 数组,
    body: POST 的数据
} 
```

其中`conflictAction`的取值只能是`uniquify`（在文件名后添加带括号的序号保证文件名唯一）、`overwrite`（覆盖）和`prompt`（给出提示让用户自行决定重命名或者覆盖）。

`filename`可以是单纯的文件名，如`'foo.txt'`；也可以带有相对路径，如`'mypath/foo.txt'`。但不可以是绝对路径，或是一个目录，也不可以在路径中包含上级路径`'..'`。如这三种情况都是非法的：`'/mypath/foo.txt'`、`'mypath/'`、`'../mypath/foo.txt'`。

如果给定了`filename`，同时`saveAs`属性为`true`，则弹出的另存为对话框中，文件名一栏的默认值会被设为`filename`指定的值。

下面让我们来一起编写一个下载当前页面所有图片的扩展。

这个扩展我准备设计成用户在页面点击右键时，菜单中包含一个下载所有图片的选项。这个过程首先要在右键菜单中创建一个选项，我们需要一个`background`脚本。因为要获取当前页面的图片元素，所以我们要向当前标签页注入脚本。这几点分析清楚之后，我们就可以开始了。

首先创建 manifest.json。

```js
{
    "manifest_version": 2,
    "name": "Save all images",
    "version": "1.0",
    "description": "Save all images in current tab",
    "background": {
        "scripts": ["background.js"],
        "persistent": false
    },
    "permissions": [
        "activeTab",
        "contextMenus",
        "downloads"
    ]
} 
```

下面编写 background.js 文件，这个文件用来创建右键菜单，并在用户点击菜单后向当前标签页注入脚本，最后还要完成下载的行为。

```js
chrome.runtime.onInstalled.addListener(function(){
    chrome.contextMenus.create({
        'id':'saveall',
        'type':'normal',
        'title':'保存所有图片',
    });
});

chrome.contextMenus.onClicked.addListener(function(info, tab){
    if(info.menuItemId == 'saveall'){
        chrome.tabs.executeScript(tab.id, {file: 'main.js'}, function(results){
            if (results && results[0] && results[0].length){
                results[0].forEach(function(url) {
                    chrome.downloads.download({
                        url: url,
                        conflictAction: 'uniquify',
                        saveAs: false
                    });
                });
            }
        });
    }
}); 
```

最后来编写注入脚本，main.js。

```js
[].map.call(document.getElementsByTagName('img'), function(img){
    return img.src;
}); 
```

至此，通过右键菜单下载所有图片的扩展就编写完成了。

本例中没有指定扩展的图标，但在成熟的产品中，自定义右键菜单时，应当指定一个 16 像素的图标。

本节所涉及到的代码可以通过[`github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/save_all_images`](https://github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/save_all_images)下载得到。

## 5.2　网络请求

Chrome 提供了较为完整的方法供扩展程序分析、阻断及更改网络请求，同时也提供了一系列较为全面的监听事件以监听整个网络请求生命周期的各个阶段。网络请求的整个生命周期所触发事件的时间顺序如下图所示。

![enter image description here](img/00033.jpeg)
*网络请求的生命周期，图片来自 developer.chrome.com*

要对网络请求进行操作，需要在 Manifest 中声明`webRequest`权限以及相关被操作的 URL。如需要阻止网络请求，需要声明`webRequestBlocking`权限。

```js
"permissions": [
    "webRequest",
    "webRequestBlocking",
    "*://*.google.com/"
] 
```

上面的权限声明表示此扩展可以对浏览器向 Google 发起的网络请求进行更改。`webRequest`接口无法在 Event Page 中使用。

目前对于网络请求，比较实用的功能包括阻断连接、更改`header`和重定向。

下面的代码阻断了所有向 bad.example.com 的连接：

```js
chrome.webRequest.onBeforeRequest.addListener(
    function(details){
        return {cancel: true};
    },
    {
        urls: [
            "*://bad.example.com/*"
        ]
    },
    [
        "blocking"
    ]
); 
```

而下面的代码则将所有连接中的`User-Agent`信息都删除了：

```js
chrome.webRequest.onBeforeSendHeaders.addListener(
    function(details){
        for(var i=0, headerLen=details.requestHeaders.length; i<headerLen; ++i){
            if(details.requestHeaders[i].name == 'User-Agent'){
                details.requestHeaders.splice(i, 1);
                break;
            }
        }
        return {requestHeaders: details.requestHeaders};
    },
    {
        urls: [
            "<all_urls>"
        ]
    },
    [
        "blocking",
        "requestHeaders"
    ]
); 
```

需要注意的是，`header`中的如下属性是不支持更改的：`Authorization`、`Cache-Control`、`Connection`、`Content-Length`、`Host`、`If-Modified-Since`、`If-None-Match`、`If-Range`、`Partial-Data`、`Pragma`、`Proxy-Authorization`、`Proxy-Connection`和`Transfer-Encoding`。

下面的代码将所有访问 www.google.com.hk 的请求重定向到了 www.google.com：

```js
chrome.webRequest.onBeforeRequest.addListener(
    function(details){
        return {redirectUrl: details.url.replace( "www.google.com.hk", "www.google.com")};
    },
    {
        urls: [
            "*://www.google.com.hk/*"
        ]
    },
    [
        "blocking"
    ]
); 
```

想想我们是不是可以做一个本地的 JS Library CDN 呢？

所有事件中，回调函数所接收到的信息对象均包括如下属性：`requestId`、`url`、`method`、`frameId`、`parentFrameId`、`tabId`、`type`和`timeStamp`。其中`type`可能的值包括`"main_frame"`、`"sub_frame"`、`"stylesheet"`、`"script"`、`"image"`、`"object"`、`"xmlhttprequest"`和`"other"`。

除了`onBeforeRequest`和`onErrorOccurred`事件外，其他所有事件返回的信息对象均包含`HttpHeaders`属性；`onHeadersReceived`、`onAuthRequired`、`onResponseStarted`、`onBeforeRedirect`和`onCompleted`事件均包括`statusLine`属性以显示请求状态，如`'HTTP/0.9 200 OK'`。其他的属性还包括`scheme`、`realm`、`challenger`、`isProxy`、`ip`、`fromCache`、`statusCode`、`redirectUrl`和`error`等，由于使用范围较小，在此不详细介绍，读者可自行到[`developer.chrome.com/extensions/webRequest`](http://developer.chrome.com/extensions/webRequest)阅读完整内容。

## 5.3　代理

代理可以让用户通过代理服务器浏览网络资源以达到匿名访问等目的。代理的类型有多种，常用的包括 http 代理和 socks 代理等。有时我们不希望所有的网络资源都通过代理浏览，这种情况下通常会使用 pac 脚本来告诉浏览器使用代理访问的规则。

Chrome 浏览器提供了代理设置管理接口，这样可以让扩展来做到更加智能的代理设置。要让扩展使用代理接口，需要声明`proxy`权限：

```js
"permissions": [
    "proxy"
] 
```

通过`chrome.proxy.settings.set`方法可以设置代理服务器，该方法需要两个参数，一个是代理设置对象，另一个是回调函数。

代理设置对象包括`mode`属性、`rules`属性和`pacScript`属性。其中`mode`属性为代理模式，可选的值有`'direct'`（直接连接，即不通过代理）、`'auto_detect'`（通过 WPAD 协议自动获取 pac 脚本）、`'pac_script'`（使用指定的 pac 脚本）、`'fixed_servers'`（固定的代理服务器）和`'system'`（使用系统的设置）。

`rules`属性和`pacScript`属性都是可选的，`rules`指定了不同的协议通过不同的代理，比如：

```js
var config = {
    mode: "fixed_servers",
    rules: {
        proxyForHttp: {
            scheme: "socks5",
            host: "1.2.3.4",
            port: 1080
        },
        proxyForHttps: {
            scheme: "socks5",
            host: "1.2.3.5",
            port: 1080
        },
        proxyForFtp: {
            scheme: "http",
            host: "1.2.3.6",
            port: 80
        }
        bypassList: ["foobar.com"]
    }
};
chrome.proxy.settings.set(
    {value: config},
    function() {
}); 
```

上面的代码定义了所有 http 协议的流量都使用 1.2.3.4:1080 这个 socks5 代理服务器代理浏览，所有 https 协议的流量都使用 1.2.3.5:1080 这个 socks5 代理服务器浏览，所有 ftp 协议的流量都使用 1.2.3.6:80 这个 http 代理服务器浏览，而 foobar.com 的流量不使用任何代理服务器，直接进行访问。`rules`还提供了`singleProxy`属性（任何协议都使用此代理）和`fallbackProxy`属性（未匹配到的协议使用此代理）。

`pacScript`指定了使用的 pac 脚本，可以通过`url`属性指定脚本位置，也可以直接通过`data`属性指定脚本内容。`pacScript`还提供了`mandatory`属性以让浏览器决定当 pac 无效时是否阻止自动切换成直接访问，此属性默认为`false`，即当 pac 无效时浏览器直接访问。

通过`chrome.proxy.settings.get`方法可以获取到浏览器当前的代理设置：

```js
chrome.proxy.settings.get(
    {},
    function(config) {
        console.log(config.value);
    }
); 
```

本节将不为大家提供 demo，而是直接带大家分析目前比较流行的 Chrome 代理管理扩展，SwitchySharp 有关代理设置的核心代码。

SwitchySharp 的完整代码可以通过[`code.google.com/p/switchysharp`](https://code.google.com/p/switchysharp)获取到，其中代理设置核心的代码为 assets/scripts/plugin.js，可以通过[`code.google.com/p/switchysharp/source/browse/assets/scripts/plugin.js`](https://code.google.com/p/switchysharp/source/browse/assets/scripts/plugin.js)在线查看此文件。

```js
var ProxyPlugin = {};
ProxyPlugin.memoryPath = memoryPath;
ProxyPlugin.proxyMode = Settings.getValue('proxyMode', 'direct');
ProxyPlugin.proxyServer = Settings.getValue('proxyServer', '');
ProxyPlugin.proxyExceptions = Settings.getValue('proxyExceptions', '');
ProxyPlugin.proxyConfigUrl = Settings.getValue('proxyConfigUrl', '');
ProxyPlugin.autoPacScriptPath = Settings.getValue('autoPacScriptPath', '');
ProxyPlugin.mute = false; 
```

SwitchySharp 首先声明了一个`ProxyPlugin`对象，此对象用来储存代理设置和代理设置方法。其中`proxyMode`属性为代理模式，和上文中讲到的代理模式相对应，但`fixed_server`模式在`proxyMode`中对应的值为`manual`；`proxyServer`属性为代理服务器地址；`proxyExceptions`属性为不使用代理设置的例外，与上文提到的`bypassList`相对应；`proxyConfigUrl`属性为 pac 脚本的 URL；`autoPacScriptPath`为 SwitchySharp 中自动切换模式下使用的 pac 脚本路径。

`mute`属性用来记录代理是否正在设置当中，如果不是，则此属性值为`false`，如果代理设置正在被更改，则此值为`ture`，用来避免设置冲突。最后`_proxy`属性用来获取 Chrome 中代理设置的方法，为了做到最大限度兼容，SwitchySharp 对代理接口依然处于实验性阶段版本的 Chrome 进行了优化：

```js
if (chrome.experimental !== undefined && chrome.experimental.proxy !== undefined)
    ProxyPlugin._proxy = chrome.experimental.proxy;
else if (chrome.proxy !== undefined)
    ProxyPlugin._proxy = chrome.proxy;
else
    alert('Need proxy api support, please update your Chrome'); 
```

`ProxyPlugin`的`updateProxy`方法用来更新代理设置选项，这个方法在开始就先判断`mute`的值是否为真，也就是判断此时代理设置是否正在被更改，如果是则退出避免设置冲突。

```js
ProxyPlugin._parseProxy = function (str) {
    if (str) {
        var proxy = {scheme:'http', host:'', port:80};
        var t1 = null;
        var t = str.indexOf(']') + 1;
        if (t > 0) {
            t1 = new Array();
            t1.push(proxy.host = str.substr(0, t));
            if (t < str.length - 1)
                t1.push(str.substr(t + 1));
        }
        else {
            t1 = str.split(':');
            proxy.host = t1[0];
        }
        var t2 = proxy.host.split('=');
        if (t2.length > 1) {
            proxy.scheme = t2[0] == 'socks' ? 'socks4' : t2[0];
            proxy.host = t2[1];
        }
        if (t1.length > 1)
            proxy.port = parseInt(t1[1]);
        return proxy;
    }
    else
        return {}
}; 
```

`_parseProxy`方法用来解析声明多种代理的规则字符串，此方法将字符串转化为用于`fixed_servers`模式下的`rules`对象。

```js
ProxyPlugin.setProxy = function (proxyMode, proxyString, proxyExceptions, proxyConfigUrl) {
    ...
    switch (proxyMode) {
        case 'system':
            config = {mode:"system"};
            break;
        ...
    }
    ProxyPlugin.mute = true;
    ProxyPlugin._proxy.settings.set({'value':config}, function () {
        ProxyPlugin.mute = false;
        if (ProxyPlugin.setProxyCallback != undefined) {
            ProxyPlugin.setProxyCallback();
            ProxyPlugin.setProxyCallback = undefined;
        }
    });
    profile = null;
    config = null;
    return 0;
}; 
```

最后`setProxy`方法将`ProxyPlugin`中与设置相关的属性重新整合成一个适用于`chrome.proxy.settings.set`方法的`config`对象，并调用`ProxyPlugin._proxy.settings.set`方法使之生效。

## 5.4　系统信息

Chrome 提供了获取系统 CPU、内存和存储设备的信息，要获取这些信息，需要在 Manifest 中分别声明如下权限：

```js
"permissions": [
    "system.cpu",
    "system.memory",
    "system.storage"
] 
```

三个接口都提供了`getInfo`方法以获取信息：

```js
chrome.system.cpu.getInfo(function(info){
    console.log(info);
});

chrome.system.memory.getInfo(function(info){
    console.log(info);
});

chrome.system.storage.getInfo(function(info){
    console.log(info);
}); 
```

CPU 的信息包括`numOfProcessors`、`archName`、`modelName`、`features`和`processors`，其中`processors`为一个记录所有逻辑处理器信息的数组。

内存信息包括`capacity`和`availableCapacity`，即总容量和可用容量。

存储空间信息为一个包含多个存储设备信息的数组，每个存储设备的信息包括`id`、`name`、`type`和`capacity`，其中`type`的可能值包括`fixed`（本地磁盘）、`removable`（可移动磁盘）和`unknown`（未知设备）。

`system.storage`还提供了获取指定设备剩余空间和移除移动磁盘的方法¹：

```js
chrome.system.storage.getAvailableCapacity(deviceId, function(info){
    console.log(info.availableCapacity);
});

chrome.system.storage.ejectDevice(deviceId, function(result){
    console.log(result);
}); 
```

^(1 目前`getAvailableCapacity`在稳定版 Chrome 中不可用。)

`chome.system.storage.onAttached`和`chome.system.storage.onDetached`事件分别用于监听可移动设备的插入和移除。

```js
chrome.system.storage.onAttached.addListener(function(info){
    console.log(info);
});

chrome.system.storage.onDetached.addListener(function(deviceId){
    console.log(deviceId);
}); 
```

以上三个接口目前来说还比较新，这意味着 Google 可能会添加新的方法或者更改现有的方法，也可能移除这些方法，建议开发者在使用这些接口时谨慎选择。