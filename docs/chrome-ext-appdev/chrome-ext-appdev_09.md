# 第六章　Chrome 应用基础

从本章开始将为大家讲解应用（App）的部分。很多人难以区分 Chrome 中扩展和应用的区别，后面的内容将向大家介绍何时使用扩展而何时使用应用，以及创建 Chrome 应用需要注意的地方。

## 6.1　应用与扩展的区别

Chrome 将其平台上的程序分为扩展与应用，并且使用了同样的文件结构，那么两者的区别是什么呢？在早期的 Chrome 版本中两者的区别非常模糊，而且有些扩展也可以用应用实现，反之亦然。但今天看来，Google 正在努力使两者的界限变得清晰。

总的来说，扩展与浏览器结合得更紧密些，更加强调扩展浏览器功能。而应用无法像扩展一样轻易获取用户在浏览器中浏览的内容并进行更改，实际上应用有更加严格的权限限制。所以应用更强调是一个独立的与 Chrome 浏览器关联不大的程序，此时你可以把 Chrome 看成是一个开发环境，而不是一个浏览器。

不过到目前为止，Google 还没有强制规定只能用扩展做什么，只能用应用做什么，所以对于那些扩展和应用都可以实现的功能，到底用何种方式实现，那是你自己的选择。不过我建议大家遵照上述的原则选择实现方式。

除此之外，Chrome 应用还分为 Hosted App（托管应用）和 Packaged App（打包应用），这两者也是有明显区别的。相对而言，Hosted App 更像是一个高级的书签，这种应用只提供一个图标和 Manifest 文件，在 Manifest 中声明了此应用的启动页面 URL，以及包含的其他页面 URL 和这些页面请求的高级权限。比如下面的例子创建了一个启动页面为 http://mail.google.com/mail/，包含 mail.google.com/mail/和 www.google.com/mail/且请求`unlimitedStorage`和`notifications`权限的应用。

```js
{
    "name": "Google Mail",
    "description": "Read your gmail",
    "version": "1",
    "app": {
        "urls": [
            "*://mail.google.com/mail/",
            "*://www.google.com/mail/"
        ],
        "launch": {
            "web_url": "http://mail.google.com/mail/"
        }
    },
    "icons": {
        "128": "icon_128.png"
    },
    "permissions": [
        "unlimitedStorage",
        "notifications"
    ]
} 
```

Packaged App，顾名思义，就是将所有文件打包在一起的应用，这类应用通常可以在离线时使用，因为它运行所需的全部文件都在本地。

由于 Hosted App 结构和功能都相对简单，所以后面的内容都将重点讲解 Packaged App。

## 6.2　更加严格的内容安全策略

在讲解 Chrome 扩展的安全策略时，提到过其不允许`inline-script`，默认也不允许引用外部的 JavaScript 文件，而 Chrome 应用使用了更加严格的限制。

Chrome 扩展和应用都使用了 CSP（Content Security Policy）声明可以引用哪些资源，虽然之前我们并没有涉及到 CSP 的内容，但是 Chrome 扩展和应用会在我们创建时提供一个默认的值，对于 Chrome 扩展来说是`script-src 'self'; object-src 'self'`。上面的 CSP 规则表示只能引用自身（同域下）的 JavaScript 文件和自身的 object 元素（如 Flash 等），其他资源未做限定。

Chrome 扩展允许开发者放宽一点点 CSP 的限制，即可以在声明权限的情况下引用 https 协议的外部 JavaScript 文件，如`script-src 'self' https://ajax.googleapis.com; object-src 'self'`。但是 Chrome 应用不允许更改默认的 CSP 设置。

那么 CSP 到底是什么呢？它是如何定义安全内容引用范围的？

CSP 通常是在 header 或者 HTML 的 meta 标签中定义的，它声明了一系列可以被当前网页合法引用的资源，如果不在 CSP 声明的合法范围内，浏览器会拒绝引用这些资源，CSP 的主要目的是防止跨站脚本攻击（XSS）。

CSP 还是 W3C 草案，最新的 1.1 版文档还在撰写之中，所以在未来可能还会增加更多特性。目前 CSP 定义了 9 种属性，分别是`connect-src`、`font-src`、`frame-src`、`img-src`、`media-src`、`object-src`、`style-src`、`script-src`和`default-src`。`connect-src`声明了通过 XHR 和 WebSocket 等方式的合法引用范围，`font-src`声明了在线字体的合法引用范围，`frame-src`声明了嵌入式框架的合法引用范围，`img-src`声明了图片的合法引用范围，`media-src`声明了声音和视频媒体的合法引用范围，`object-src`声明了 Flash 等对象的合法引用范围，`style-src`声明了 CSS 的合法引用范围，`script-src`声明了 JavaScript 的合法引用范围，最后`default-src`声明了未指定的其他引用方式的合法引用范围。

CSP 的可选属性值有`'self'`、`'unsafe-inline'`、`'unsafe-eval'`、`'none'`，这四个属性值都必须带引号代表特殊含义的值，分别表示允许引用同域资源、允许执行`inline-script`、允许执行字符串转换的代码（如在`eval`函数和`setTimeout`中的字符串代码）、不允许引用任何资源。

同时还支持 host，如`www.google.com`表示可以引用 www.google.com 的资源，或者`*.google.com`允许引用 google.com 所有子域的资源（但不允许 google.com 根域的资源）。也可以声明只允许引用 https 下的资源，属性值声明为`https:`即可。或者声明只允许引用特定协议特定 host 的资源，如`https://github.com`。`*`则代表任何资源，即不受限制。

Chrome 应用默认的 CSP 规则为：

```js
default-src 'self';
connect-src *;
style-src 'self' data: chrome-extension-resource: 'unsafe-inline';
img-src 'self' data: chrome-extension-resource:;
frame-src 'self' data: chrome-extension-resource:;
font-src 'self' data: chrome-extension-resource:;
media-src *; 
```

也就是说在 Chrome 应用中，我们可以使用 XHR 请求任何资源；但是只能引用应用自身的 CSS 文件或者是 dataURL 转换的 CSS 文件和 chrome-extension-resource 协议下的 CSS 文件，同时我们可以在 HTML 直接写 style 代码块和在 DOM 中指定 style 属性；图片、嵌入式框架和字体只能引用自身或者 dataURL 转换的文件和 chrome-extension-resource 协议下的文件；可以引用任何音频和视频媒体资源；其他未指定的引用方式（`object-src`和`script-src`）只能引用自身资源。

这么做显然会大大提高 Chrome 应用的安全性，防止被黑客利用盗取用户的数据，但也显然带来了新的问题。从 Chrome 应用的 CSP 规则中我们发现其不允许通过嵌入式框架引用外部资源，那么如果我们真的需要将一个外部页面展示在 Chrome 应用中怎么办呢？Google 提供了`webview`标签代替`iframe`标签，使用`webview`标签必须指定大小和引用 URL。

```js
<webview src="http://news.google.com/" width="640" height="480"></webview> 
```

同样 Chrome 应用也不允许引用外部的图片，但是我们可以通过 XHR 请求外部图片资源（XHR 是可以请求到任何资源的，只要在 Manifest 中声明权限），然后通过转换成 blob URL 再添加到应用中。

```js
var xhr = new XMLHttpRequest();
xhr.open('GET', 'https://supersweetdomainbutnotcspfriendly.com/image.png', true);
xhr.responseType = 'blob';
xhr.onload = function(e) {
    var img = document.createElement('img');
    img.src = window.URL.createObjectURL(this.response);
    document.body.appendChild(img);
};

xhr.send(); 
```

最后如果无法避免使用`inline-script`和`eval`等方式执行 JavaScript 代码，我们可以将“违规”的页面放入沙箱中执行，方法是在 Manifest 的 sandbox 中列出需要在沙箱中执行的页面。

```js
"sandbox": {
    "pages": ["sandboxed.html"]
} 
```

## 6.3　图标设计规范

虽然 Google 没有对应用图标的设计做出强制规定，但给出了一份建议文档，完整文档可以参见[`developer.chrome.com/webstore/images`](https://developer.chrome.com/webstore/images)，本节将根据原始文档内容，对图标设计规范相关的部分进行转述。

在应用展示页面（chrome://apps/），Chrome 默认会以 128 像素的尺寸展示应用图标，但根据窗口实际尺寸会自动进行缩放，最小展示 64 像素的图标。

Chrome 应用的图标只支持 png 格式，而且 Google 建议将图标的可视部分定在 96 像素之内，在可视部分周围留出边距。即如下图所示，将正方形的图标限定在方形框中。

![enter image description here](img/00034.jpeg)
*正方形图标模板，图片来自 developers.google.com*

如果是圆形的图标，同样限定在上述模板的方形框中会显得过小，可以控制在下图的圆形图标模板的圆形圈中。

![enter image description here](img/00035.jpeg)
*圆形图标模板，图片来自 developers.google.com*

对于那些不规则的图标，可以结合正方形和圆形的模板进行设计。正方形和圆形的图标看上去往往给人感觉比实际的尺寸要大一些，所以在设计图标时要注意这一点，尽量在视觉上让不同的图标保持一致的尺寸。下面是不同形状的图标在一起的对比。

![enter image description here](img/00036.jpeg)

![enter image description here](img/00037.jpeg)
*不同形状图标尺寸的对比，图片来自 developers.google.com*

下面是一些具体的例子。

![enter image description here](img/00038.jpeg)
*不同形状图标设计实例，图片来自 developers.google.com*

由于 Chrome 允许用户更换主题，所以应考虑图标在不同明暗背景下的显示效果。如果图标本身是浅色系，则应在图标周围添加深色边界，反之亦然。

最后 Google 还建议，如果设计的图标有一定浮雕效果，凸起高度最好限制在 4 像素。图标最好是正对用户的，而不是侧面 45 度的透视效果，如下图所示。

![enter image description here](img/00039.jpeg)![enter image description here](img/00040.jpeg)
*诸如此类透视效果的图标不建议使用，图片来自 developers.google.com*

虽然 Google 不会因为开发者未遵循上述规范而驳回或撤销应用，但是图标是一个应用给用户的第一印象，与周围应用图标风格明显有别的应用会流失一部分用户。

## 6.4　应用的生命周期

本章第一节的内容简单讲解了应用和扩展的区别，本节将为大家讲解应用和扩展的另一大区别，生命周期。

对于扩展来说，如果定义了后台脚本，同时指定`persistent`属性为`true`，那么这个扩展只要浏览器运行就会一直运行，除非用户手动去关闭它。如果声明了`background`权限，则这个扩展会一开机就运行，并且一直运行下去。但是对于应用来说情况会有所不同。

Chrome 应用目前不允许使用永久运行的后台脚本（仅指 Packaged App），无论你多么想让它一直运行下去，比如用于监听来自 WebSocket 的消息等，Chrome 在认为应该关闭它时就会关闭它。这种做法确实有利于减少不必一直运行的应用消耗有限的内存资源，但就像上面举的例子那样，也同样会在某些方面带来局限。

既然关闭应用的行为并非开发者可以直接控制，那么我们就有必要了解应用何时开始运行，又会在何时被关闭——也就是应用的生命周期。下图给出了应用生命周期的简单图示。

![enter image description here](img/00041.jpeg)
*Chrome 应用的生命周期，图片来自 developer.chrome.com*

Event Page 就是 Chrome 应用的后台脚本，它用于监听各种事件。当用户运行应用，Event Page 加载完成后，`onLaunched`事件就会被触发。如果这个应用运行后要向用户提供一个窗口，就是在`onLaunched`事件触发后后台脚本创建的。当这个窗口被关闭后，并且 Event Page 也没有需要处理的任务，Chrome 就会彻底关闭这个应用——连同 Event Page 一起关闭。在关闭应用前会触发`onSuspend`事件，这个事件可以提醒应用的后台脚本应用即将被关闭，以给应用一个准备退出的机会。

每一个应用都会有一个 Event Page，可以通过 Event Page 监听`onLaunched`事件，然后创建一个窗口。在 Manifest 的`app`属性中，通过`background`域定义 Event Page。

```js
"app": {
    "background": {
        "scripts": ["background.js"]
    }
} 
```

然后在 background.js 中指定应用启动时创建窗口¹。

^(1 从 Chrome 36 开始，`create`方法的选项对象不再支持`bounds`、`minWidth`、`maxWidth`、`minHeight`和`maxHeight`属性，请使用`innerBounds`和`outerBounds`属性代替。`innerBounds`和`outerBounds`的属性值包括`width`、`height`、`left`、`top`、`minWidth`、`maxWidth`、`minHeight`和`maxHeight`。)

```js
chrome.app.runtime.onLaunched.addListener(function() {
    chrome.app.window.create('main.html', {
        id: 'MyWindowID',
        bounds: {
            width: 800,
            height: 600,
            left: 100,
            top: 100
        },
        minWidth: 800,
        minHeight: 600
    });
}); 
```

有关在应用中创建窗口的具体内容将在下一节给出。

在应用首次被安装或者更新到新版本时，会触发`onInstalled`事件，Event Page 可以在此事件触发时做一些初始化任务，如向本地写入默认设置、向用户展示欢迎窗口等等。

Chrome 应用可以使用`chrome.storage`存储数据，如：

```js
chrome.runtime.onInstalled.addListener(function() {
    chrome.storage.local.set(object items, function callback);
}); 
```

数据保存在用户本地时，可能会面临数据永远丢失的风险——当用户卸载应用或者重新安装操作系统后，应用保存在本地的数据都会永久丢失。为防止这种风险，可以选择使用一种在线的存储方式，最简单的方法就是使用 Chrome storage API 中的`sync`域储存数据：

```js
chrome.runtime.onInstalled.addListener(function() {
    chrome.storage.sync.set(items, function(){...});
}); 
```

这样这些数据会随 Chrome 在线同步保存在 Google 的服务器中。这对诸如应用设置等数据非常重要，因为用户也许会有一天重新安装卸载掉的应用。

最后当 Chrome 认为一个应用处于空闲状态时就会清理掉这个应用的进程，在清理之前会触发`onSuspend`事件，以让 Event Page 在退出前有机会做一些清理工作，从而不会导致意外退出。

有一种办法可以让应用一直保持运行，就是永远都不关闭前端窗口——当用户关闭窗口时隐藏它而不是真的关闭，或者让后台直接创建一个隐藏的窗口，这样前端窗口就可以一直运行且不显示出来。当用户再次启动应用时，再显示出来就可以了。

虽然也可以使用`setInterval`定期做一些简单的任务，如每 10 秒就让 Event Page 做点什么，但是 Chrome 清理应用进程的时间间隔是可以通过指定参数更改的，并且将来很有可能会发生变化，所以这种方法并不值得推荐。

对于个别应用确实需要长时间在后台运行，Google 也清楚这一点，但是为什么 Chrome 应用却不提供`background`权限呢？按照 Chromium 开发者的说法，由于要专门为应用提供 system tray 特性，应用可以通过这一特性常驻后台，在没有提供这一特性前，为了防止应用开发者依赖`background`权限，所以禁用了它。

那么现在的情况就比较尴尬了，system tray 尚未实现，`background`权限又已被禁用，所以如果想让应用随系统启动并常驻后台，目前一种可行的方法——虽然笨拙但却有效，安装一个拥有`background`权限的扩展，这样可以让 Chrome 一直运行；在 Event Page 中添加一个`setInterval`任务以防止被 Chrome 认为应用处于空闲状态；创建一个 Chrome 应用的快捷方式（在 chrome://apps/中应用图标的右键菜单中，选择“创建快捷方式...”），并将这个快捷方式放到系统的启动文件夹下。

## 6.5　应用窗口

Chrome 应用中创建的窗口与 Chrome 浏览器中的窗口没有任何关系，这一点与 Chrome 扩展不同。 本节将详细讲解应用窗口的创建风格以及窗口相关的其他方法和事件。请记住，我们并不是在创建一个网页，而是在创建一个桌面程序，不要把应用的窗口风格搞得和网页一样。

### 6.5.1　创建窗口

通过`chrome.app.window.create`方法可以创建应用窗口，应用窗口与扩展中新建的窗口并不相同，应用窗口的默认样式与操作系统并没有太大关系，所以不同平台下 Chrome 应用的窗口能够保持较高的一致性。

如果不给出任何有关窗口外观样式的参数，应用窗口会是下面的样子：

![enter image description here](img/00042.jpeg)
*应用窗口默认样式*

创建这个窗口的代码为：

```js
chrome.app.window.create('blank.html', {
    id: 'default'
}); 
```

其中 blank.html 为新建窗口嵌入的页面。

在上面的代码中只指定了窗口的`id`，在以后创建新窗口时，也建议总是指定一个窗口`id`，`id`相同的窗口只会创建一个。

这个窗口没有指定大小，Chrome 给出的默认大小一般是 512×384 像素（不算标题栏），标题栏一般的默认高度是 22 像素，具体与系统设置有关。

窗口的控制按钮 Chrome 根据系统不同给出了相应的样式和位置，以照顾不同平台用户的使用习惯。比如 Windows 下控制按钮是方形并放置在右上角的，而对于 OS X 则是圆形的并放置在左上角。

Chrome 默认的应用窗口非常简洁，但在实际使用时需要注意在某些系统下默认窗口是没有阴影效果的，在白色背景的衬托下，用户将很难找到应用窗口的边缘。比如在早期版本的 Windows 下，如果没有定义一个窗口边框，应用窗口又恰好在一张白色网页前打开，用户就会看到下面的情景（窗口两侧是未被遮挡的 Google 搜索框）：

![enter image description here](img/00043.jpeg)
*未定义窗口边框时在个别系统下用户会很难找到窗口边缘*

最简单的方法是用 CSS 为`body`添加一个边框：

```js
body {
    border: black 1px solid;
} 
```

或者为`body`指定一个背景颜色，而不使用窗口默认的白色：

```js
body {
    background: #EEE;
} 
```

在创建窗口时也可以指定窗口的大小，如：

```js
chrome.app.window.create('main.html', {
    id: 'main',
    bounds: {
        width: 800,
        height: 600
    }
}); 
```

如果窗口定义了`id`，且用户对窗口进行了尺寸调整，下次再创建此窗口时 Chrome 会使用用户上次调整后的尺寸取代代码中的尺寸，这也是指定窗口 id 的好处之一。

通过`bounds`指定的尺寸是不包含窗口外框的，如标题栏等，只是窗口内嵌入页面的显示尺寸。如果不希望用户调整窗口尺寸可以指定窗口的`resizable`属性值为`false`：

```js
chrome.app.window.create('main.html', {
    id: 'main',
    bounds: {
        width: 800,
        height: 600
    },
    resizable: false
}); 
```

也可以指定窗口可调节尺寸的范围，比如：

```js
chrome.app.window.create('main.html', {
    id: 'main',
    bounds: {
        width: 800,
        height: 600,
        minWidth: 400,
        minHeight: 300,
        maxWidth: 1600,
        maxHeight: 1200
    }
}); 
```

除了指定窗口大小，还可以指定窗口位置，如果不指定，则默认显示在屏幕中心。

```js
chrome.app.window.create('main.html', {
    id: 'main',
    bounds: {
        top: 0,
        left: 0
    }
}); 
```

上面的代码创建了一个在屏幕左上角的窗口。如果指定了`id`，Chrome 同样会记住用户上次将窗口放置的位置，并在下次创建窗口时使用记录的值。

其他的特性还包括新建窗口状态（最大化、最小化、正常或者全屏）和窗口是否总是在最前面，在声明`app.window.alwaysOnTop`权限的情况下，下面的代码创建了一个总是在最前面的全屏窗口：

```js
chrome.app.window.create('main.html', {
    id: 'main',
    state: 'fullscreen',
    alwaysOnTop: true
}); 
```

其中`state`的值还可以是`normal`、`maximized`和`minimized`。

最后窗口的`hidden`属性是非常重要的，它可以让窗口在后台静默运行，类似于后台脚本，但在需要时可以使用`show`方法重新显示出来，具体有关隐藏窗口的内容将在后面的内容中详细讲解。下面的代码创建了一个隐藏的窗口：

```js
chrome.app.window.create('main.html', {
    id: 'main',
    hidden: true
}); 
```

窗口创建完成后我们也可以使用回调函数获取刚刚创建窗口的属性：

```js
chrome.app.window.create('main.html', {'id': 'main'}, function(appWindow){
    console.log(appWindow);
}); 
```

### 6.5.2　样式更加自由的窗口

在上一节中讲解了创建应用窗口的方法，同时也介绍了部分窗口属性。虽然默认的窗口样式已经非常简洁，我们可以在窗口内部自由地进行设计，但是自带的标题栏却无法更改样式，本节将进一步讲解如何创建样式更加自由的窗口。

将窗口的`frame`属性值定为`'none'`，新建的窗口将不显示标题栏，如：

```js
chrome.app.window.create('blank.html', {
    id: 'blank',
    frame: 'none'
}); 
```

上面的代码会生成下面所示的窗口：

![enter image description here](img/00044.jpeg)
*没有标题栏的窗口*

由于这个窗口没有标题栏，也没有控制按钮，所以无法拖拽，也无法通过在窗口上点击鼠标来关闭它，或者改变它的显示状态，比如最大化最小化等。

我们可以在 HTML 中指定可以拖拽的元素，这样当鼠标在这些元素上的时候就可以拖拽整个窗口了，下面对 blank.html 进行改进一下：

```js
<html>
<head>
<title>A more free style window</title>
<style>
body {
    margin: 0;
    padding: 0;
    border: #EEE 1px solid;
}

#title_bar {
    -webkit-app-region: drag;
    height: 22px;
    line-height: 22px;
    font-size: 16px;
    background: #EEE;
    padding: 0 10px;
    box-sizing: border-box;
}
</style>
</head>
<body>
<div id="title_bar">A more free style window</div>
</body>
</html> 
```

![enter image description here](img/00045.jpeg)
*可以拖拽的窗口*

现在这个窗口可以拖拽了，重点就在于上面代码中的`-webkit-app-region: drag`。

在介绍自定义窗口控制按钮之前需要先了解获取当前窗口的方法，因为所有控制函数都是当前窗口对象的子元素。

```js
var current_window = chrome.app.window.current(); 
```

上面的代码可以获取到当前代码所在窗口的窗口对象。

窗口对象的`close`方法可以关闭当前窗口，如：

```js
current_window.close(); 
```

同样还可以最大化窗口、最小化窗口、还原窗口或全屏窗口：

```js
current_window.maximize();
current_window.minimize();
current_window.restore();
current_window.fullscreen(); 
```

也可以获取当前窗口是否处于某种状态：

```js
var is_maximize = current_window.isMaximized();
var is_minimize = current_window.isMinimized();
var is_fullscreen = current_window.isFullscreen(); 
```

以上函数均返回布尔型结果。

下面给 blank.html 页面添加上控制按钮。首先在`title_bar`的右侧添加三个圆形的按钮，分别对应最小化、最大化（还原）和关闭。

```js
<div id="title_bar">A more free style window
    <a id="close" href="#"></a>
    <a id="maximize" href="#"></a>
    <a id="minimize" href="#"></a>
</div> 
```

然后在样式表中添加这三个按钮的显示样式：

```js
#title_bar a {
    display: inline-block;
    float: right;
    height: 12px;
    width: 12px;
    margin: 5px;
    border: black 1px solid;
    box-sizing: border-box;
    border-radius: 6px;
} 
```

在添加按钮元素时，之所以将关闭按钮放在最前面，是因为样式表中定义了`float: right`，这将使最先出现的元素放置在最右侧。

下面是添加按钮后的窗口：

![enter image description here](img/00046.jpeg)
*添加按钮后的窗口*

现在看起来虽然感觉好多了，但是当鼠标悬浮在按钮上时并没有反馈交互，所以我们还应该更加细化一下设计。继续在样式表中添加交互特性：

```js
#title_bar a:hover {
    background: black;
} 
```

当鼠标放在按钮上，按钮就会变成黑色的实心圆：

![enter image description here](img/00047.jpeg)
*鼠标悬浮在按钮上的反馈交互*

下面来为这三个按钮绑定事件。最小化和关闭按钮都很容易：

```js
var current_window = chrome.app.window.current();

document.getElementById('minimize').onclick = function(){
    current_window.minimize();
}

document.getElementById('close').onclick = function(){
    current_window.close();
} 
```

对于最大化的按钮，因为同时也是还原窗口的按钮，所以当用户点击时要进行判断：

```js
document.getElementById('maximize').onclick = function(){
    current_window.isMaximized() ?
        current_window.restore() :
        current_window.maximize();
} 
```

最后将写好的 JavaScript 紧贴在`</body>`标签之前引用。

但当我们进行测试时却发现点击按钮并没有反应，这是怎么回事呢？因为我们将控制按钮放在了`title_bar`之内，而`title_bar`之前定义了`-webkit-app-region: drag`样式用于拖拽，这会使 Chrome 阻止鼠标点击事件，解决的方法是专门为控制按钮定义`-webkit-app-region: no-drag`样式。这一点没有在创建按钮时直接提出是为了强调其重要性，尤其是对于整个窗口都可以拖动的应用，应该为所有的控制按钮都专门指定`-webkit-app-region: no-drag`样式。

下面是改进后完整的 HTML 代码：

```js
<html>
<head>
<title>A more free style window</title>
<style>
body {
    margin: 0;
    padding: 0;
    border: #EEE 1px solid;
}

#title_bar {
    -webkit-app-region: drag;
    height: 22px;
    line-height: 22px;
    font-size: 16px;
    background: #EEE;
    padding: 0 10px;
    box-sizing: border-box;
}

#title_bar a {
    -webkit-app-region: no-drag;
    display: inline-block;
    float: right;
    height: 12px;
    width: 12px;
    margin: 5px;
    border: black 1px solid;
    box-sizing: border-box;
    border-radius: 6px;
}

#title_bar a:hover {
    background: black;
}
</style>
</head>
<body>
<div id="title_bar">A more free style window
    <a id="close" href="#"></a>
    <a id="maximize" href="#"></a>
    <a id="minimize" href="#"></a>
</div>
<script src="control.js"></script>
</body>
</html> 
```

下面是完整的 JavaScript 代码：

```js
var current_window = chrome.app.window.current();

document.getElementById('minimize').onclick = function(){
    current_window.minimize();
}

document.getElementById('close').onclick = function(){
    current_window.close();
}

document.getElementById('maximize').onclick = function(){
    current_window.isMaximized() ?
        current_window.restore() :
        current_window.maximize();
} 
```

### 6.5.3　获取窗口

之前我们接触了`chrome.app.window.current`方法获取代码所在的窗口，除此之外还可以通过`chrome.app.window.getAll`方法获取全部窗口，以及`chrome.app.window.get`方法获取指定窗口。

`chrome.app.window.current`和`chrome.app.window.get`方法均返回窗口对象，`chrome.app.window.getAll`方法则返回包含若干窗口对象的数组。

调用`chrome.app.window.get`方法时需要指定窗口`id`：

```js
var main_window = chrome.app.window.get('main'); 
```

以下是窗口对象的完整结构，其中除`id`为字符串、`contentWindow`为 JavaScript window object，其他均为函数。

```js
{
    focus: 将焦点放在窗口上,
    fullscreen: 将窗口全屏,
    isFullscreen: 判断窗口是否处于全屏状态,
    minimize: 将窗口最小化,
    isMinimized: 判断窗口是否处于最小化状态,
    maximize: 将窗口最大化,
    isMaximized: 判断窗口是否处于最大化状态,
    restore: 还原窗口,
    moveTo: 将窗口移动到指定位置，调用方法为 moveTo(left, top),
    resizeTo: 将窗口尺寸设定为指定大小，调用方法为 resizeTo(width, height),
    drawAttention: 将窗口高亮显示,
    clearAttention: 清除窗口高亮显示,
    close: 关闭窗口,
    show: 显示隐藏窗口,
    hide: 隐藏窗口,
    getBounds: 获取窗口内容区域尺寸和位置,
    setBounds: 设置窗口内容区域尺寸和位置,
    isAlwaysOnTop: 判断窗口是否一直显示在最前端,
    setAlwaysOnTop: 将窗口设为总是最前端显示,
    contentWindow: JavaScript window object,
    id: 窗口 id，此 id 为创建时所指定
} 
```

获取窗口时，如果指定的窗口不存在则返回`null`。使用`getAll`方法时，如果不存在任何窗口则返回一个空数组。

在 6.4 节中提到了用隐藏窗口的方法防止应用被 Chrome 关闭，下面对之前的代码进行更改。首先将关闭按钮绑定的事件改为隐藏：

```js
var current_window = chrome.app.window.current();

document.getElementById('close').onclick = current_window.hide(); 
```

其次将 Event Page 中启动事件改写成先判断窗口是否存在，如果存在则调用`show`方法显示，否则创建：

```js
chrome.app.runtime.onLaunched.addListener(function() {
    var main_window = chrome.app.window.get('main');
    if(main_window){
        main_window.show();
    }
    else{
        chrome.app.window.create('main.html', {
            id: 'main',
            bounds: {
                width: 800,
                height: 600,
                left: 100,
                top: 100
            },
            frame: 'none'
        });
    } 
}); 
```

当窗口关闭后，可以看到扩展程序管理器里显示 main.html 依然在运行。

![enter image description here](img/00048.jpeg)
*使用 hide 方法阻止应用被关闭*

### 6.5.4　窗口事件

应用窗口有 6 种事件，其中有 4 种用于监听窗口状态，分别是`onFullscreened`、`onMaximized`、`onMinimized`和`onRestored`：

```js
chrome.app.window.onFullscreened.addListener(function(){
    //do something when the window is set to fullscreen.
});

chrome.app.window.onMaximized.addListener(function(){
    //do something when the window is set to maximized.
});

chrome.app.window.onMinimized.addListener(function(){
    //do something when the window is set to minimized.
});

chrome.app.window.onRestored.addListener(function(){
    //do something when the window is set to restored.
}); 
```

另外两种事件一个用于监听窗口尺寸变化，另一个用于监听窗口被关闭：

```js
chrome.app.window.onBoundsChanged.addListener(function(){
    //do something when the window is resized.
});

chrome.app.window.onClosed.addListener(function(){
    //do something when the window is closed.
}); 
```

## 6.6　编写第一个 Chrome 应用

在编写 Chrome 应用时请时刻记住，这已经不是单纯地开发浏览器扩展了，现在要编写的是一款真正的桌面程序，而 Chrome 只是类似 CLR 和 Java 的环境而已。

下面我们来一起编写一个计算机性能监视器。

首先来创建 Manifest 文件：

```js
{
    "app": {
        "background": {
            "scripts": ["background.js"]
        }
    },
    "manifest_version": 2,
    "name": "Performance Monitor",
    "version": "1.0",
    "description": "A performance monitor to show cpu and memory status.",
    "icons": {
        "16": "images/icon16.png",
        "48": "images/icon48.png",
        "128": "images/icon128.png"
    },
    "permissions": [
        "system.cpu",
        "system.memory"
    ]
} 
```

下面编写 background.js 脚本，根据 6.5 的内容可以直接写出如下代码：

```js
chrome.app.runtime.onLaunched.addListener(function() {
    chrome.app.window.create('main.html', {
        'id': 'main',
        'bounds': {
            'width': 542,
            'height': 360
        },
        'resizable': false,
        'frame': 'none'
    });
}); 
```

同理，main.html 中的自定义菜单我们也用上一节提到的代码，但取消最大化按钮。为绘制曲线和饼图，本例使用了一个 JS 图表库，Chart.js，有关 Chart.js 的详细内容可以通过[`www.bootcss.com/p/chart.js/`](http://www.bootcss.com/p/chart.js/)查看。

下面是 main.html 的代码：

```js
<html>
<head>
<title>Performance Monitor</title>
<style>
body {
    margin: 0;
    padding: 0;
    border: #EEE 1px solid;
}

#title_bar {
    -webkit-app-region: drag;
    height: 22px;
    line-height: 22px;
    font-size: 16px;
    background: #EEE;
    padding: 0 10px;
    box-sizing: border-box;
}

#title_bar a {
    -webkit-app-region: no-drag;
    display: inline-block;
    float: right;
    height: 12px;
    width: 12px;
    margin: 5px;
    border: gray 1px solid;
    box-sizing: border-box;
    border-radius: 6px;
}

#title_bar a:hover {
    background: gray;
}

#box_body {
    padding: 20px;
}

.chart {
    margin-bottom: 20px;
    font-size: 0;
}

.usage_item {
    color: gray;
    padding: 2px 0;
    font-size: 14px;
    border-bottom: #EEE 1px solid;
    margin-bottom: 4px;
}
</style>
</head>
<body>
<div id="title_bar">Performance Monitor
    <a id="close" href="#"></a>
    <a id="minimize" href="#"></a>
</div>
<div id="box_body">
<div class="usage_item">CPU Usage</div>
<div class="chart">
<canvas id="cpu_total" width="100" height="100"></canvas>
<canvas id="cpu_history" width="400" height="100"></canvas>
</div>
<div class="usage_item">Memory Usage</div>
<div class="chart">
<canvas id="mem_total" width="100" height="100"></canvas>
<canvas id="mem_history" width="400" height="100"></canvas>
</div>
</div>
<script src="control.js"></script>
<script src="Chart.js"></script>
<script src="main.js"></script>
</body>
</html> 
```

其中的`canvas`用来展示数据。

control.js 的代码：

```js
var current_window = chrome.app.window.current();

document.getElementById('minimize').onclick = function(){
    current_window.minimize();
}

document.getElementById('close').onclick = function(){
    current_window.close();
} 
```

下面来编写 main.js，这个脚本用来定时获取数据并进行展示。

```js
function getCpuUsage(callback){
    chrome.system.cpu.getInfo(function(info){
        var total = 0;
        var user = 0;
        var kernel = 0;
        for(var i=0; i<info.processors.length; i++){
            total += info.processors[i].usage.total - cpu_history.last_total[i];
            cpu_history.last_total[i] = info.processors[i].usage.total;
            user += info.processors[i].usage.user - cpu_history.last_user[i];
            cpu_history.last_user[i] = info.processors[i].usage.user;
            kernel += info.processors[i].usage.kernel - cpu_history.last_kernel[i];
            cpu_history.last_kernel[i] = info.processors[i].usage.kernel;
        }
        user = Math.round(user/total*100);
        kernel = Math.round(kernel/total*100);
        callback({user:user,kernel:kernel,total:user+kernel});
    });
}

function getMemUsage(callback){
    chrome.system.memory.getInfo(function(info){
        callback(info);
    });
}

function updateCpuHistory(){
    getCpuUsage(function(usage){
        cpu_history.user.shift();
        cpu_history.user.push(usage.user);
        cpu_history.kernel.shift();
        cpu_history.kernel.push(usage.kernel);
        cpu_history.total.shift();
        cpu_history.total.push(usage.total);
        showCpu();
    });
}

function updateMemHistory(){
    getMemUsage(function(usage){
        mem_history.used.shift();
        mem_history.used.push(Math.round((usage.capacity-usage.availableCapacity)/usage.capacity*100));
        showMem();
    });
}

function updateData(){
    updateCpuHistory();
    updateMemHistory();
}

function showCpu(){
    var history = {
        labels : (function(){for(var i=0,labels=[];i<ponits_num;labels.push(''),i++);return labels;})(),
        datasets : [
            {
                fillColor : "rgba(220,220,220,0.5)",
                data : cpu_history.total
            },
            {
                fillColor : "rgba(90,140,255,0.5)",
                data : cpu_history.kernel
            },
            {
                fillColor : "rgba(255,90,90,0.5)",
                data : cpu_history.user
            }
        ]
    };

    var now = [
        {
            value: cpu_history.total[ponits_num-1],
            color:"rgba(220,220,220,0.7)"
        },
        {
            value : 100-cpu_history.total[ponits_num-1],
            color : "rgba(220,220,220,0.3)"
        }            
    ];
    var his_ctx = document.getElementById('cpu_history').getContext("2d");
    var now_ctx = document.getElementById("cpu_total").getContext("2d");
    new Chart(his_ctx).Line(history, {scaleFontSize:4,pointDot:false,animation:false});
    new Chart(now_ctx).Pie(now, {segmentShowStroke:false,animation:false});
}

function showMem(){
    var history = {
        labels : (function(){for(var i=0,labels=[];i<ponits_num;labels.push(''),i++);return labels;})(),
        datasets : [
            {
                fillColor : "rgba(220,220,220,0.5)",
                data : mem_history.used
            }
        ]
    };

    var now = [
        {
            value: mem_history.used[ponits_num-1],
            color:"rgba(220,220,220,0.7)"
        },
        {
            value : 100-mem_history.used[ponits_num-1],
            color : "rgba(220,220,220,0.3)"
        }            
    ];
    var his_ctx = document.getElementById('mem_history').getContext("2d");
    var now_ctx = document.getElementById("mem_total").getContext("2d");
    new Chart(his_ctx).Line(history, {scaleFontSize:4,pointDot:false,animation:false});
    new Chart(now_ctx).Pie(now, {segmentShowStroke:false,animation:false});
}

function init(){
    cpu_history = {
        user: [],
        kernel: [],
        total: [],
        last_user: [],
        last_kernel: [],
        last_total: []
    };
    mem_history = {
        used: []
    };
    init_cpu_history();
}

function init_cpu_history(){
    for(var i=0; i<ponits_num; i++){
        cpu_history.user.push(0);
        cpu_history.kernel.push(0);
        cpu_history.total.push(0);
    }
    chrome.system.cpu.getInfo(function(info){
        for(var i=0; i<info.processors.length; i++){
            cpu_history.last_total.push(info.processors[i].usage.total);
            cpu_history.last_user.push(info.processors[i].usage.user);
            cpu_history.last_kernel.push(info.processors[i].usage.kernel);
        }
        init_mem_history();
    });
}

function init_mem_history(){
    for(var i=0; i<ponits_num; i++){
        mem_history.used.push(0);
    }
    updateData();
    setInterval(updateData, 1000);
}

var cpu_history, mem_history, ponits_num=20;

init(); 
```

其中`getCpuUsage`和`getMemUsage`函数分别用于获取 CPU 和内存的使用量。值得注意的是，Chrome 返回的 CPU 使用量是累计使用时间，并不是获取瞬间的 CPU 占用量，所以需要对前后两个时间点获得的结果做差。`showCpu`和`showMem`方法将获取的数据显示成图表，两个函数都是参照 Chart.js 文档编写的，感兴趣的读者可以自行查阅。

另外 Chart.js 本身有 new Function 声明函数的部分，由于之前介绍的 CSP 规则，这在 Chrome 应用中是不被允许的，所以本例中的 Chart.js 是编者改写后的，具体差异读者可以下载后自行对照。在以后编写 Chrome 应用引用现成的库时，可能会经常遇到由于 CSP 的限制而无法直接运行的情况，这需要读者拥有自行更改库代码的能力。

本例应用运行的截图如下所示：

![enter image description here](img/00049.jpeg)
*Performance Monitor 运行截图*

本节中讲解的实例的源代码可以通过[`github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/performance%20monitor`](https://github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/performance%20monitor)下载到。