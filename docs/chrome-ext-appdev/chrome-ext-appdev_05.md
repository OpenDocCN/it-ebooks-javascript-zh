# 第二章　Chrome 扩展基础

本章会讲解 Chrome 扩展的一些基础功能，这些基础的功能在后续的扩展编写中可能会被频繁用到，所以有必要提前进行详细的讲解。本章会配有多个实例，一步步带着读者完成一个个有趣的例子。

## 2.1　操作用户正在浏览的页面

通过 Chrome 扩展我们可以对用户当前浏览的页面进行操作，实际上就是对用户当前浏览页面的 DOM 进行操作。通过 Manifest 中的`content_scripts`属性可以指定将哪些脚本何时注入到哪些页面中，当用户访问这些页面后，相应脚本即可自动运行，从而对页面 DOM 进行操作。

Manifest 的`content_scripts`属性值为数组类型，数组的每个元素可以包含`matches`、`exclude_matches`、`css`、`js`、`run_at`、`all_frames`、`include_globs`和`exclude_globs`等属性。其中`matches`属性定义了哪些页面会被注入脚本，`exclude_matches`则定义了哪些页面不会被注入脚本，`css`和`js`对应要注入的样式表和 JavaScript，`run_at`定义了何时进行注入，`all_frames`定义脚本是否会注入到嵌入式框架中，`include_globs`和`exclude_globs`则是全局 URL 匹配，最终脚本是否会被注入由`matches`、`exclude_matches`、`include_globs`和`exclude_globs`的值共同决定。简单的说，如果 URL 匹配`mathces`值的同时也匹配`include_globs`的值，会被注入；如果 URL 匹配`exclude_matches`的值或者匹配`exclude_globs`的值，则不会被注入。

`content_scripts`中的脚本只是共享页面的 DOM¹，而并不共享页面内嵌 JavaScript 的命名空间。也就是说，如果当前页面中的 JavaScript 有一个全局变量`a`，`content_scripts`中注入的脚本也可以有一个全局变量`a`，两者不会相互干扰。当然你也无法通过`content_scripts`访问到页面本身内嵌 JavaScript 的变量和函数。

^(1 DOM 中的自定义属性不会被共享。)

下面我们来写一个恶作剧的小扩展，名字就叫做永远点不到的搜索按钮吧 :)

首先创建 Manifest 文件，内容如下：

```
{
    "manifest_version": 2,
    "name": "永远点不到的搜索按钮",
    "version": "1.0",
    "description": "让你永远也点击不到 Google 的搜索按钮",
    "content_scripts": [
        {
            "matches": ["*://www.google.com/"],
            "js": ["js/cannot_touch.js"]
        }
    ]
} 
```

在`content_scripts`属性中我们定义了一个匹配规则，当 URL 符合`*://www.google.com/`规则的时候，就将`js/cannot_touch.js`注入到页面中。其中`*`代表任意字符，这样当用户访问 http://www.google.com/和 https://www.google.com/时就会触发脚本。

右键单击搜索按钮，选择“审查元素”，我们发现 Google 搜索按钮的`id`为`'gbqfba'`。

![enter image description here](img/00009.jpeg)
*通过 Chrome 浏览器的开发者工具可以看到 Google 搜索按钮的 id*

接下来我们开始编写 cannot_touch.js。

```
function btn_move(el, mouseLeft, mouseTop){
    var leftRnd = (Math.random()-0.5)*20;
    var topRnd = (Math.random()-0.5)*20;
    var btnLeft = mouseLeft+(leftRnd>0?100:-100)+leftRnd;
    var btnTop = mouseTop+(topRnd>0?30:-30)+topRnd;
    btnLeft = btnLeft<100?(btnLeft+window.innerWidth-200):(btnLeft>window.innerWidth-100?btnLeft-window.innerWidth+200:btnLeft);
    btnTop =  btnTop<100?( btnTop+window.innerHeight-200):(btnTop>window.innerHeight-100?btnTop-window.innerHeight+200:btnTop);
    el.style.position = 'fixed';
    el.style.left = btnLeft+'px';
    el.style.top = btnTop+'px';
}

function over_btn(e){
    if(!e){
        e = window.event;
    }
    btn_move(this, e.clientX, e.clientY);
}

document.getElementById('gbqfba').onmouseover = over_btn; 
```

由于 Manifest 将此脚本的位置指定到了`js/cannot_touch.js`，所以要记得将这个脚本保存到扩展文件夹中的 js 文件夹下，否则会出现错误。

![enter image description here](img/00010.jpeg)
*“永远点不到的搜索按钮”扩展运行的结果*

可以看出，`content_scripts`很像 Userscript，它就是将指定的脚本文件插入到符合规则的特定页面中，从而使插入的脚本可以对页面的 DOM 进行操作。

这个扩展的源码可以在[`github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/cannot_touch`](https://github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/cannot_touch)下载到。

## 2.2　跨域请求

跨域指的是 JavaScript 通过`XMLHttpRequest`请求数据时，调用 JavaScript 的页面所在的域和被请求页面的域不一致。对于网站来说，浏览器出于安全考虑是不允许跨域。另外，对于域相同，但端口或协议不同时，浏览器也是禁止的。下表给出了进一步的说明：

| URL | 说明 | 是否允许请求 |
| http://a.example.com/ http://a.example.com/a.txt | 同域下 | 允许 |
| http://a.example.com/ http://a.example.com/b/a.txt | 同域下不同目录 | 允许 |
| http://a.example.com/ http://a.example.com:8080/a.txt | 同域下不同端口 | 不允许 |
| http://a.example.com/ https://a.example.com/a.txt | 同域下不同协议 | 不允许 |
| http://a.example.com/ http://b.example.com/a.txt | 不同域下 | 不允许 |
| http://a.example.com/ http://a.foo.com/a.txt | 不同域下 | 不允许 |

但这个规则如果同样限制 Chrome 扩展应用，就会使其能力大打折扣，所以 Google 允许 Chrome 扩展应用不必受限于跨域限制。但出于安全考虑，需要在 Manifest 的`permissions`属性中声明需要跨域的权限。

比如，如果我们想设计一款获取维基百科数据并显示在其他网页中的扩展，就要在 Manifest 中进行如下声明：

```
{
    ...
    "permissions": [
        "*://*.wikipedia.org/*"
    ]
} 
```

这样 Chrome 就会允许你的扩展在任意页面请求维基百科上的内容了。

我们可以利用如下的代码发起异步请求：

```
function httpRequest(url, callback){
    var xhr = new XMLHttpRequest();
    xhr.open("GET", url, true);
    xhr.onreadystatechange = function() {
        if (xhr.readyState == 4) {
            callback(xhr.responseText);
        }
    }
    xhr.send();
} 
```

这样每次发起请求时，只要调用`httpRequest`函数，并传入要请求的 URL 和接收返回结果的函数就可以了。为什么要使用`callback`函数接收请求结果，而不直接用`return`将结果作为函数值返回呢？因为`XMLHttpRequest`不会阻塞下面代码的运行。

为了更加明确地说清上述问题，让我们来举两个例子。

```
function count(n){
    var sum = 0;
    for(var i=1; i<=n; i++){
        sum += i;
    }
    return sum;
}

var c = count(5)+1;
console.log(c); 
```

上面这个例子会在控制台显示 16，因为`count(5)=1+2+3+4+5=15`，`c=15+1=16`。我们再看下面的例子：

```
function httpRequest(url){
    var xhr = new XMLHttpRequest();
    xhr.open("GET", url, true);
    xhr.onreadystatechange = function() {
        if (xhr.readyState == 4) {
            return xhr.responseText;
        }
    }
    xhr.send();
}

var html = httpRequest('test.txt');
console.log(html); 
```

![enter image description here](img/00011.jpeg)
*上例运行结果*

通过上图可以发现，虽然请求的资源内容为`Hello World!`，但却并没有正确地显示出来。

对于第一个例子，`count`函数是一个阻塞函数，在它没有运行完之前它会阻塞下面的代码运行，所以直到`count`计算结束后才将结果返回后再加`1`赋给变量`c`，最后将变量`c`的值打印出来。而第二个例子中的`httpRequest`函数不是一个阻塞函数，在它没运行完之前后面的代码就已经开始运行，这样`html`变量在`httpRequest`函数没返回值之前就被赋值，所以最终的结果必然就是`undefined`了。

既然这样，如何将非阻塞函数的最终结果传递下去呢？方法就是使用回调函数。在 Chrome 扩展应用的 API 中，大部分函数都是非阻塞函数，所以使用回调函数传递结果的方法以后会经常用到。

让我们来用回调函数的形式重写第二个例子：

```
function httpRequest(url, callback){
    var xhr = new XMLHttpRequest();
    xhr.open("GET", url, true);
    xhr.onreadystatechange = function() {
        if (xhr.readyState == 4) {
            callback(xhr.responseText);
        }
    }
    xhr.send();
}

var html;
httpRequest('test.txt', function(result){
    html = result;
    console.log(html);
}); 
```

![enter image description here](img/00012.jpeg)
*改进后第二个例子的结果*

可以看到`httpRequest`函数运行的结果已经被正确地打印出来了。

下面来实战编写一款显示用户 IP 的扩展。

```
{
    "manifest_version": 2,
    "name": "查看我的 IP",
    "version": "1.0",
    "description": "查看我的电脑当前的公网 IP",
    "icons": {
        "16": "images/icon16.png",
        "48": "images/icon48.png",
        "128": "images/icon128.png"
    },
    "browser_action": {
        "default_icon": {
            "19": "images/icon19.png",
            "38": "images/icon38.png"
        },
        "default_title": "查看我的 IP",
        "default_popup": "popup.html"
    },
    "permissions": [
        "http://sneezryworks.sinaapp.com/ip.php"
    ]
} 
```

上面的 Manifest 定义了这个扩展允许对 http://sneezryworks.sinaapp.com/ip.php 发起跨域请求，其他的属性在 1.2 节中都有介绍，在此就不再赘述了。

popup.html 的结构也完全可以按照时钟的扩展照抄下来，只是个别元素的`id`和脚本的路径根据当前扩展的名称稍加更改，同样不再赘述。

```
<html>
<head>
<style>
* {
    margin: 0;
    padding: 0;
}

body {
    width: 400px;
    height: 100px;
}

div {
    line-height: 100px;
    font-size: 42px;
    text-align: center;
}
</style>
</head>
<body>
<div id="ip_div">正在查询……</div>
<script src="js/my_ip.js"></script>
</body>
</html> 
```

下面编写 my_ip.js。

```
function httpRequest(url, callback){
    var xhr = new XMLHttpRequest();
    xhr.open("GET", url, true);
    xhr.onreadystatechange = function() {
        if (xhr.readyState == 4) {
            callback(xhr.responseText);
        }
    }
    xhr.send();
}

httpRequest('http://sneezryworks.sinaapp.com/ip.php', function(ip){
    document.getElementById('ip_div').innerText = ip;
}); 
```

![enter image description here](img/00013.jpeg)
*“查看我的 IP”扩展运行结果*

作为一个开发者，安全问题永远都不应被轻视。在你从外域获取到数据后，不要轻易作为当前页面元素的`innerHTML`直接插入，更不要用`eval`函数去执行它，否则很可能将用户置于危险的境地。如果要将请求到的数据写入页面，可以使用`innerText`，就像我们这个查看 IP 的扩展那样。如果是 JSON 格式是数据就使用`JSON.parse`函数去解析。为了避免请求数据返回的格式错误，结合`try-catch`一起使用也是不错的选择。

本节中扩展的源码可以通过[`github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/what_is_my_ip`](https://github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/what_is_my_ip)下载到。

## 2.3　常驻后台

有时我们希望扩展不仅在用户主动发起时（如开启特定页面或点击扩展图标等）才运行，而是希望扩展自动运行并常驻后台来实现一些特定的功能，比如实时提示未读邮件数量、后台播放音乐等等。

Chrome 允许扩展应用在后台常驻一个页面以实现这样的功能。在一些典型的扩展中，UI 页面，如 popup 页面或者 options 页面，在需要更新一些状态时，会向后台页面请求数据，而当后台页面检测到状态发生改变时，也会通知 UI 界面刷新。

后台页面与 UI 页面可以相互通信，这将在后续的章节中做进一步的讲解，本节将主要讲解后台页面是如何工作的。

在 Manifest 中指定`background`域可以使扩展常驻后台。`background`可以包含三种属性，分别是`scripts`、`page`和`persistent`。如果指定了`scripts`属性，则 Chrome 会在扩展启动时自动创建一个包含所有指定脚本的页面；如果指定了`page`属性，则 Chrome 会将指定的 HTML 文件作为后台页面运行。通常我们只需要使用`scripts`属性即可，除非在后台页面中需要构建特殊的 HTML——但一般情况下后台页面的 HTML 我们是看不到的。`persistent`属性定义了常驻后台的方式——当其值为`true`时，表示扩展将一直在后台运行，无论其是否正在工作；当其值为`false`时，表示扩展在后台按需运行，这就是 Chrome 后来提出的 Event Page。Event Page 可以有效减小扩展对内存的消耗，如非必要，请将`persistent`设置为`false`。`persistent`的默认值为`true`。

由于编写一个只有后台页面的扩展，很难看到扩展运行的结果，所以我决定在本节中破例使用一个尚未讲到但是很简单的扩展功能，动态改变扩展图标，这在后面的例子中会进行说明。

下面我们来编写一款实时监视网站在线状态的扩展。思路很简单，每隔 5 秒就发起一次连接请求，如果请求成功就代表网站在线，将扩展图标显示为绿色，如果请求失败就代表网站不在线，将扩展图标显示为红色。

下面是这个扩展的 Manifest 文件，此例中以检测 www.google.cn 为例，你可以根据自己的意愿更改为其他的网站。

```
{
    "manifest_version": 2,
    "name": "Google 在线状态",
    "version": "1.0",
    "description": "监视 Google 是否在线",
    "icons": {
        "16": "images/icon16.png",
        "48": "images/icon48.png",
        "128": "images/icon128.png"
    },
    "browser_action": {
        "default_icon": {
            "19": "images/icon19.png",
            "38": "images/icon38.png"
        }
    },
    "background": {
        "scripts": [
            "js/status.js"
        ]
    },
    "permissions": [
        "http://www.google.cn/"
    ]
} 
```

由于这个扩展没有 UI，所以我们不必编写 HTML 文件，下面直接编写 status.js。

```
function httpRequest(url, callback){
    var xhr = new XMLHttpRequest();
    xhr.open("GET", url, true);
    xhr.onreadystatechange = function() {
        if (xhr.readyState == 4) {
            callback(true);
        }
    }
    xhr.onerror = function(){
        callback(false);
    }
    xhr.send();
}

setInterval(function(){
    httpRequest('http://www.google.cn/', function(status){
        chrome.browserAction.setIcon({path: 'images/'+(status?'online.png':'offline.png')});
    });
},5000); 
```

status.js 调用了我们之前没有介绍过的方法，`chrome.browserAction.setIcon`。Chrome 为扩展应用提供了很多类似的方法可以使得扩展应用做更多的事情，并且与浏览器结合得更加紧密。这个方法的作用就是更换扩展在浏览器工具栏中的图标。

本节示例扩展中的`httpRequest`函数，与上节所讲述跨域请求中所使用的函数非常类似，但请注意本节在`httpRequest`函数中加入了`onerror`事件，正是因为加入了这个事件才能捕捉到请求过程中是否发生了错误，从而得知所监视的网站是否在线。将本例载入 Chrome 后，在联网的情况下可以看到扩展图标为绿色，断开网络连接后扩展图标变为了红色。

![enter image description here](img/00014.jpeg)
*本节示例扩展的运行结果*

小提示：如果想在用户打开浏览器之前就让扩展运行，可以在 Manifest 的`permissions`属性中加入`"background"`，但除非必要，否则尽量不要这么做，因为大部分用户不喜欢这样。

本例中所编写的扩展源码可以通过[`github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/website_status`](https://github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/website_status)下载得到。

## 2.4　带选项页面的扩展

有一些扩展允许用户进行个性化设置，这样就需要向用户提供一个选项页面。Chrome 通过 Manifest 文件的`options_page`属性为开发者提供了这样的接口，可以为扩展指定一个选项页面。当用户在扩展图标上点击右键，选择菜单中的“选项”后，就会打开这个页面¹。

^(1 对于没有图标的扩展，可以在 chrome://extensions 页面中单击“选项”。)

![enter image description here](img/00015.jpeg)
*指定 options_page 属性后，扩展图标上的右键菜单会包含“选项”链接*

对于网站来说，用户的设置通常保存在 Cookies 中，或者保存在网站服务器的数据库中。对于 JavaScript 来说，一些数据可以保存在变量中，但如果用户重新启动浏览器，这些数据就会消失。那么如何在扩展中保存用户的设置呢？我们可以使用 HTML5 新增的`localStorage`接口。除了`localStorage`接口以外，还可以使用其他的储存方法。后面将专门拿出一节来讲解数据存储，本节中我们先使用最简单的`localStorage`方法储存数据。

`localStorage`是 HTML5 新增的方法，它允许 JavaScript 在用户计算机硬盘上永久储存数据（除非用户主动删除）。但`localStorage`也有一些限制，首先是`localStorage`和 Cookies 类似，都有域的限制，运行在不同域的 JavaScript 无法调用其他域`localStorage`的数据；其次是单个域在`localStorage`中存储数据的大小通常有限制（虽然 W3C 没有给出限制），对于 Chrome 这个限制是 5MB²；最后`localStorage`只能储存字符串型的数据，无法保存数组和对象，但可以通过`join`、`toString`和`JSON.stringify`等方法先转换成字符串再储存。

^(2 通过声明`unlimitedStorage`权限，Chrome 扩展和应用可以突破这一限制。)

下面我们将编写一个天气预报的扩展，这个扩展将提供一个选项页面供用户填写所关注的城市。

有很多网站提供天气预报的 API，比如 OpenWeatherMap 的 API。可以通过 http://openweathermap.org/API 了解更多相关内容。

```
{
    "manifest_version": 2,
    "name": "天气预报",
    "version": "1.0",
    "description": "查看未来两周的天气情况",
    "icons": {
        "16": "images/icon16.png",
        "48": "images/icon48.png",
        "128": "images/icon128.png"
    },
    "browser_action": {
        "default_icon": {
            "19": "images/icon19.png",
            "38": "images/icon38.png"
        },
        "default_title": "天气预报",
        "default_popup": "popup.html"
    },
    "options_page": "options.html",
    "permissions": [
        "http://api.openweathermap.org/data/2.5/forecast?q=*"
    ]
} 
```

上面是这个扩展的 Manifest 文件，options.html 为设定选项的页面。下面开始编写 options.html 文件。

```
<html>
    <head>
        <title>设定城市</title>
    </head>
    <body>
        <input type="text" id="city" />
        <input type="button" id="save" value="保存" />
        <script src="js/options.js"></script>
    </body>
</html> 
```

这个页面提供了一个 id 为 city 的文本框和一个`id`为`save`的按钮。由于 Chrome 不允许将 JavaScript 内嵌在 HTML 文件中，所以我们单独编写一个 options.js 脚本文件，并在 HTML 文件中引用它。下面来编写 options.js 文件。

```
var city = localStorage.city || 'beijing';
document.getElementById('city').value = city;
document.getElementById('save').onclick = function(){
    localStorage.city = document.getElementById('city').value;
    alert('保存成功。');
} 
```

从 options.js 的代码中可以看到，`localStorage`的读取和写入方法很简单，和 JavaScript 中的变量读写方法类似。`localStorage`除了使用`localStorage.namespace`的方法引用和写入数据外，还可以使用`localStorage['namespace']`的形式。请注意第二种方法`namespace`要用引号包围，单引号和双引号都可以。如果想彻底删除一个数据，可以使用`localStorage.removeItem('namespace')`方法。

为了显示天气预报的结果，我们为扩展指定了一个 popup 页面，popup.html。下面来编写这个 UI 页面。

```
<html>
<head>
<style>
* {
    margin: 0;
    padding: 0;
}

body {
    width: 520px;
    height: 270px;
}

table {
    font-family: "Lucida Sans Unicode", "Lucida Grande", Sans-Serif;
    font-size: 12px;
    width: 480px;
    text-align: left;
    border-collapse: collapse;
    border: 1px solid #69c;
    margin: 20px;
    cursor: default;

}

table th {
    font-weight: normal;
    font-size: 14px;
    color: #039;
    border-bottom: 1px dashed #69c;
    padding: 12px 17px;
    white-space: nowrap;
}

table td {
    color: #669;
    padding: 7px 17px;
    white-space: nowrap;
}

table tbody tr:hover td {
    color: #339;
    background: #d0dafd;
}

</style>
</head>
<body>
<div id="weather"></div>
<script src="js/weather.js"></script>
</body>
</html> 
```

其中`id`为`weather`的`div`元素将用于显示天气预报的结果。下面来编写 weather.js 文件。

```
function httpRequest(url, callback){
    var xhr = new XMLHttpRequest();
    xhr.open("GET", url, true);
    xhr.onreadystatechange = function() {
        if (xhr.readyState == 4) {
            callback(xhr.responseText);
        }
    }
    xhr.send();
}

function showWeather(result){
    result = JSON.parse(result);
    var list = result.list;
    var table = '<table><tr><th>日期</th><th>天气</th><th>最低温度</th><th>最高温度</th></tr>';
    for(var i in list){
        var d = new Date(list[i].dt*1000);
        table += '<tr>';
        table += '<td>'+d.getFullYear()+'-'+(d.getMonth()+1)+'-'+d.getDate()+'</td>';
        table += '<td>'+list[i].weather[0].description+'</td>';
        table += '<td>'+Math.round(list[i].temp.min-273.15)+' °C</td>';
        table += '<td>'+Math.round(list[i].temp.max-273.15)+' °C</td>';
        table += '</tr>';
    }
    table += '</table>';
    document.getElementById('weather').innerHTML = table;
}

var city = localStorage.city;
city = city?city:'beijing';
var url = 'http://api.openweathermap.org/data/2.5/forecast/daily?q='+city+',china&lang=zh_cn';
httpRequest(url, showWeather); 
```

小提示：无论是 options.js 还是 weather.js 中都有如下语句：

```
var city = localStorage.city;
city = city?city:'beijing'; 
```

也就是说，当选项没有值时，应设定一个默认值，以避免程序出错。此处如果用户未设置城市，扩展将显示北京的天气预报。

![enter image description here](img/00016.jpeg)
*weather 扩展的选项页面，点击保存按钮后会提示保存成功*

![enter image description here](img/00017.jpeg)
*weather 扩展的运行界面*

本节示例扩展的源代码可以通过[`github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/weather`](https://github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/weather)下载得到。

## 2.5　扩展页面间的通信

有时需要让扩展中的多个页面之间，或者不同扩展的多个页面之间相互传输数据，以获得彼此的状态。比如音乐播放器扩展，当用户鼠标点击 popup 页面中的音乐列表时，popup 页面应该将用户这个指令告知后台页面，之后后台页面开始播放相应的音乐。

Chrome 提供了 4 个有关扩展页面间相互通信的接口，分别是`runtime.sendMessage`、`runtime.onMessage`、`runtime.connect`和`runtime.onConnect`。做为一部入门级教程，此节将只讲解`runtime.sendMessage`和`runtime.onMessage`接口，`runtime.connect`和`runtime.onConnect`做为更高级的接口请读者依据自己的兴趣自行学习，你可以在[`developer.chrome.com/extensions/extension`](http://developer.chrome.com/extensions/extension)得到有关这两个接口的完整官方文档。

请注意，Chrome 提供的大部分 API 是不支持在`content_scripts`中运行的，但`runtime.sendMessage`和`runtime.onMessage`可以在`content_scripts`中运行，所以扩展的其他页面也可以同`content_scripts`相互通信。

`runtime.sendMessage`完整的方法为：

```
chrome.runtime.sendMessage(extensionId, message, options, callback) 
```

其中`extensionId`为所发送消息的目标扩展，如果不指定这个值，则默认为发起此消息的扩展本身；`message`为要发送的内容，类型随意，内容随意，比如可以是`'Hello'`，也可以是`{action: 'play'}`、`2013`和`['Jim', 'Tom', 'Kate']`等等；`options`为对象类型，包含一个值为布尔型的`includeTlsChannelId`属性，此属性的值决定扩展发起此消息时是否要将 TLS 通道 ID 发送给监听此消息的外部扩展¹，有关 TLS 的相关内容可以参考[`www.google.com/intl/zh-CN/chrome/browser/privacy/whitepaper.html#tls`](http://www.google.com/intl/zh-CN/chrome/browser/privacy/whitepaper.html#tls)，这是有关加强用户连接安全性的技术，如果这个参数你捉摸不透，不必理睬它，`options`是一个可选参数；callback 是回调函数，用于接收返回结果，同样是一个可选参数。

^(1 此属性仅在扩展和网页间通信时才会用到。)

`runtime.onMessage`完整的方法为：

```
chrome.runtime.onMessage.addListener(callback) 
```

此处的`callback`为必选参数，为回调函数。`callback`接收到的参数有三个，分别是`message`、`sender`和`sendResponse`，即消息内容、消息发送者相关信息和相应函数。其中`sender`对象包含 4 个属性，分别是`tab`、`id`、`url`和`tlsChannelId`，`tab`是发起消息的标签，有关标签的内容可以参看 4.5 节的内容。

为了进一步说明，下面举一个例子。

在 popup.html 中执行如下代码：

```
chrome.runtime.sendMessage('Hello', function(response){
    document.write(response);
}); 
```

在 background 中执行如下代码：

```
chrome.runtime.onMessage.addListener(function(message, sender, sendResponse){
    if(message == 'Hello'){
        sendResponse('Hello from background.');
    }
}); 
```

查看 popup.html 页面会发现有输出“Hello from background.”。

![enter image description here](img/00018.jpeg)
*扩展内部通信 Demo 的运行画面*

上面这个小例子的源代码可以从[`github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/runtime.sendMessage_runtime.onMessage_demo`](https://github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/runtime.sendMessage_runtime.onMessage_demo)下载到。

## 2.6　储存数据

一个程序免不了要储存数据，对于 Chrome 扩展也是这样。通常 Chrome 扩展使用以下三种方法中的一种来储存数据：第一种是使用 HTML5 的`localStorage`，这种方法在上一节的内容中已经涉及；第二种是使用 Chrome 提供的存储 API；第三种是使用 Web SQL Database。

对于一般的扩展，“设置”这种简单的数据可以优先选择第一种，因为这种方法使用简单，可以看成是特殊的 JavaScript 变量；对于结构稍微复杂一些的数据可以优先选择第二种，这种方法可以保存任意类型的数据，但需要异步调用 Chrome 的 API，结果需要使用回调函数接收，不如第一种操作简单；第三种目前使用的不算太多，因为需要使用 SQL 语句对数据库进行读写操作，较前两者更加复杂，但是对于数据量庞大的应用来说是个不错的选择。开发者应根据实际的情况选择上述三种方法中的一种或几种来存储扩展中的数据。

由于上节已经讲解了`localStorage`的使用方法，下面将详细讲解后两种储存数据的方法。

**Chrome 存储 API**

Chrome 为扩展应用提供了存储 API，以便将扩展中需要保存的数据写入本地磁盘。Chrome 提供的存储 API 可以说是对`localStorage`的改进，它与`localStorage`相比有以下区别：

*   如果储存区域指定为`sync`，数据可以自动同步；

*   `content_scripts`可以直接读取数据，而不必通过 background 页面；

*   在隐身模式下仍然可以读出之前存储的数据；

*   读写速度更快；

*   用户数据可以以对象的类型保存。

对于第二点要进一步说明一下。首先`localStorage`是基于域名的，这在前面的小节中已经提到过了。而`content_scripts`是注入到用户当前浏览页面中的，如果`content_scripts`直接读取`localStorage`，所读取到的数据是用户当前浏览页面所在域中的。所以通常的解决办法是`content_scripts`通过`runtime.sendMessage`和 background 通信，由 background 读写扩展所在域（通常是 chrome-extension://extension-id/）的`localStorage`，然后再传递给`content_scripts`。

使用 Chrome 存储 API 必须要在 Manifest 的`permissions`中声明`"storage"`，之后才有权限调用。Chrome 存储 API 提供了 2 种储存区域，分别是`sync`和`local`。两种储存区域的区别在于，`sync`储存的区域会根据用户当前在 Chrome 上登陆的 Google 账户自动同步数据，当无可用网络连接可用时，`sync`区域对数据的读写和 local 区域对数据的读写行为一致。

对于每种储存区域，Chrome 又提供了 5 个方法，分别是`get`、`getBytesInUse`、`set`、`remove`和`clear`。

`get`方法即为读取数据，完整的方法为：

```
chrome.storage.StorageArea.get(keys, callback) 
```

`keys`可以是字符串、包含多个字符串的数组或对象。如果`keys`是字符串，则和`localStorage`的用法类似；如果是数组，则相当于一次读取了多个数据；如果`keys`是对象，则会先读取以这个对象属性名为键值的数据，如果这个数据不存在则返回`keys`对象的属性值（比如`keys`为`{'name':'Billy'}`，如果`name`这个值存在，就返回`name`原有的值，如果不存在就返回`Billy`）。如果`keys`为一个空数组（`[]`）或空对象（`{}`），则返回一个空列表，如果`keys`为`null`，则返回所有存储的数据。

`getBytesInUse`方法为获取一个数据或多个数据所占用的总空间，返回结果的单位是字节，完整方法为：

```
chrome.storage.StorageArea.getBytesInUse(keys, callback) 
```

此处的`keys`只能为`null`、字符串或包含多个字符串的数组。

`set`方法为写入数据，完整方法为：

```
chrome.storage.StorageArea.set(items, callback) 
```

`items`为对象类型，形式为键/值对。`items`的属性值如果是字符型、数字型和数组型，则储存的格式不会改变，但如果是对象型和函数型的，会被储存为“`{}`”，如果是日期型和正则型的，会被储存为它们的字符串形式。

`remove`方法为删除数据，完整方法为：

```
chrome.storage.StorageArea.remove(keys, callback) 
```

其中`keys`可以是字符串，也可以是包含多个字符串的数组。

`clear`方法为删除所有数据，完整方法为：

```
chrome.storage.StorageArea.clear(callback) 
```

请注意，上述五种完整方法中，`StorageArea`必须指定为`local`或`sync`中的一个。

Chrome 同时还为存储 API 提供了一个`onChanged`事件，当存储区的数据发生改变时，这个事件会被激发。

`onChanged`的完整方法为：

```
chrome.storage.onChanged.addListener(callback) 
```

`callback`会接收到两个参数，第一个为`changes`，第二个是`StorageArea`。`changes`是词典对象，键为更改的属性名称，值包含两个属性，分别为`oldValue`和`newValue`；`StorageArea`为`local`或`sync`。

**Web SQL Database**

Web SQL Database 的三个核心方法为`openDatabase`、`transaction`和`executeSql`。`openDatabase`方法的作用是与数据库建立连接，`transaction`方法的作用是执行查询，`executeSql`方法的作用是执行 SQL 语句。

下面举一个简单的例子：

```
db = openDatabase("db_name", "0.1", "This is a test db.", 1024*1024);
if(!db){
    alert('数据库连接失败。');
}
else {
    db.transaction( function(tx) {
        tx.executeSql(
            "SELECT COUNT(*) FROM db_name",
            [],
            function(tx, result){
                console.log(result);
            },
            function(tx, error){
                alert('查询失败：'+error.message);
            }
        );
    }
} 
```

更多关于 Web SQL Database 的资料可以参考[`www.w3.org/TR/webdatabase/`](http://www.w3.org/TR/webdatabase/)。由于原生的 Web SQL Database 并不算好用，也有一些开源的二次封装的库来简化 Web SQL Database 的使用，如[`github.com/KenCorbettJr/html5sql`](https://github.com/KenCorbettJr/html5sql)。

以上几种数据的存储方式都不会对数据加密，如果储存的是敏感的数据，应该先进行加密处理。比如不要将用户密码的明码直接储存，而应先进行 MD5 加密。