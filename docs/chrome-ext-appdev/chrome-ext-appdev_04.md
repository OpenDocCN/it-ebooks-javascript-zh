# 第一章　初步接触 Chrome 扩展应用开发

Chrome 是 Google 公司基于 WebKit 开发的一款浏览器¹，但从某种角度上来说它已经超越了浏览器成为了一个平台甚至是一个操作系统。Chrome 继承了 WebKit 内核对 HTML 的高速渲染，同时 Google 自行开发的 V8 引擎使得 JavaScript 在 Chrome 中的执行效率大幅提升，这使得更加高级复杂的 JavaScript 程序在 Chrome 中运行成为可能。

^(1 Chrome 28 之后使用的 Blink 渲染引擎是 WebKit 中 WebCore 组件的一个分支。)

Chrome 浏览器除了页面渲染速度快，JavaScript 执行速度快以外，另一大特点就是支持开发者为其编写各种各样的扩展来扩充其功能，用 HTML5 编写桌面程序，这使得 Chrome 变得更加强大。编写这样的程序就是本书所要讲解的内容。

本章首先对 Chrome 扩展应用进行简单概述，之后带着读者编写一个简单的扩展，使读者对扩展的认识更加深入。在讲解扩展 Manifest 文件格式时，也会简单讲解一下 JSON 数据格式²，避免没有接触过 JSON 的读者阅读后续的内容产生困难。另外本章也用一小节简单讲解了一下 DOM，这对从未接触过网页编程的读者会非常有帮助。

^(2 JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式。)

## 1.1　认识 Chrome 扩展及应用

Chrome 扩展是用于扩充 Chrome 浏览器功能的程序，Chrome 应用是以 Chrome 为平台运行的程序，两者似乎并没有太明确的区别，甚至有些程序既可以设计成 Chrome 扩展也可以设计成 Chrome 应用。但既然 Google 将基于 Chrome 平台的程序分为了两类，说明两者还是有区别的。

Chrome 扩展主要用于对浏览器功能的增强，它更强调与浏览器相结合。比如 Chrome 扩展可以在浏览器的工具栏和地址栏中显示图标，它可以更改用户当前浏览的网页中的内容，也可以更改浏览器代理服务器的设置等等。

Chrome 应用更强调是独立的程序，你可以不打开 Chrome 浏览器而运行这些程序。同时这些程序可以调用更加底层的系统接口，比如串口、USB、本地文件读写等等。同时 Chrome 应用可以拥有样式更加自由的独立窗口，而 Chrome 扩展的界面只能限定在浏览器窗口中。

由于 Chrome 扩展和 Chrome 应用有很多相似之处，为了叙述方便本章会统一说成 Chrome 扩展，但应该清楚同样适用于 Chrome 应用。

Chrome 扩展是一系列文件的集合，这些文件包括 HTML 文件、CSS 样式文件、JavaScript 脚本文件、图片等静态文件以及 manifest.json。个别扩展还会包含二进制文件，如 DLL 动态库和 so 动态库等，但这需要调用 NPAPI，而 Google 出于安全性考虑已经决定逐渐淘汰 NPAPI，所以我不准备在本书中向大家介绍有关 NPAPI 的内容。

扩展被安装后，Chrome 就会读取扩展中的 manifest.json 文件。这个文件的文件名固定为 manifest.json，内容是按照一定格式描述的扩展相关信息，如扩展名称、版本、更新地址、请求的权限、扩展的 UI 界面入口等等。这样 Chrome 就可以知道在浏览器中如何呈现这个扩展，以及这个扩展如何同用户进行交互。

由于 Chrome 扩展是基于 Chrome 平台的，说得直白些，是基于 WebKit 浏览器的——当然有些更加高级的接口不仅仅依赖于 WebKit 浏览器——所以 Chrome 扩展在处理逻辑运算和实现程序功能时所采用的编程语言必然只能是 JavaScript。

可能你会感到惊讶，毕竟 JavaScript 最开始是为提升网站与用户交互体验而设计出的一种轻量级脚本语言，怎么会脱离网站而成为一种程序的逻辑语言呢？随着 Chrome 浏览器 V8 引擎的出现，JavaScript 的执行效率得到了大幅提升，甚至出现了将其作为后端语言的项目——Node.js。所以将 JavaScript 作为一种客户端程序语言就更是绰绰有余了——只要提供更加丰富的功能函数——而 Chrome 平台正提供了这样的环境。

总的来说，Chrome 扩展更像是一个运行于本地的网站，只是它可以利用 Chrome 平台提供的丰富的接口，获得更加全面的信息，进行更加复杂的操作。而它的界面则使用 HTML 和 CSS 进行描述，这样的好处是可以用很短的时间构建出赏心悦目的 UI。界面的渲染完全不需要开发者操心，而是交给 Chrome 去做。同时由于 JavaScript 是一门解释语言¹，无需对其配置编译器，调试代码时你只要刷新一下浏览器就可以看到修改后的结果，这使得开发周期大大缩短。

^(1 现代浏览器使用的 JavaScript 引擎会对 JavaScript 编译，V8。)

同时 Chrome 浏览器相比于 Java 虚拟机、Python 解释器（Linux 和 OS X 中默认安装了 Python，而 Windows 中默认没有安装）等其他语言环境更加普及——我甚至可以在我们学校的图书馆计算机中找到 Chrome 浏览器——所以你所开发的 Chrome 扩展就可以在更多的计算机中运行。当你眼前遇到一个问题需要利用计算机去处理，而这台计算机恰好安装了 Chrome 浏览器，那么你就可以欢快地打开记事本开始编写程序了，之后加载到 Chrome 浏览器中就可以查看运行结果，这是一件多么酷的事啊！

别急，后面的内容就会让你得到这项新技能！

## 1.2　我的第一个 Chrome 扩展

我发现很多讲解编程的书籍，在前面都会详细地讲解相关的预备知识，而大多数读者却更希望马上进行实践。没错，人们总是对基础知识很排斥，这也就是为什么在教育行业开始推崇自顶向下的教材设计方案了——先让读者看到一个最接近表面的东西，之后再慢慢深入地讲解内在的原理和基础。所以我决定在还没有讲什么的时候，先带大家写一个 Demo 程序。这样不仅可以让大家在实践中对基础知识掌握得更加牢靠，同时也调动了大家的积极性。

Chrome 扩展的启动入口可以在浏览器的工具栏和地址栏中，用户单击后激活扩展进行下一步的操作，也可以干脆没有图标，在后台静默地运行。比如微博的扩展，可以设计成将图标显示在工具栏中，用户点击后打开一个显示用户微博时间轴的界面；RSS 订阅器扩展可以设计成将图标显示在地址栏中，当用户点击后，订阅地址栏中当前显示的 URL；自动使用 Google SSL 链接的扩展可以不显示图标，只是在后台默默地监视，当用户访问了非 SSL 的 Google 链接后，自动跳转到 SSL 链接即可。

![enter image description here](img/00001.jpeg)
*Chrome 扩展图标在浏览器中的位置*

我们准备编写一款显示用户计算机当前时间的扩展，这应该比 Hello World 有趣得多。设计思路是在浏览器的工具栏中显示一个时钟的图标，当用户点击这个图标时显示一个实时显示计算机时间的界面。

首先新建一个名为 my_clock 的文件夹，在此文件夹中新建一个名为 manifest.json 的文件，内容如下：

```
{
    "manifest_version": 2,
    "name": "我的时钟",
    "version": "1.0",
    "description": "我的第一个 Chrome 扩展",
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
        "default_title": "我的时钟",
        "default_popup": "popup.html"
    }
} 
```

上面的字段有些我们可以一眼看出在定义什么，比如`name`定义了扩展的名称，`version`定义了扩展的版本，`description`定义了扩展的描述，`icons`定义了扩展相关图标文件的位置。`version`的值最多可以是由三个圆点分为四段的版本号，每段只能是数字，每段数字不能大于 65535 且不能以 0 开头（可以是 0，但不可以是 0123），版本号段左侧为高位，比如 1.0.2.0 版本比 1.0.0.1 版本更高。每次更新扩展时，新的版本号必须比之前的版本号高。

`browser_action`指定扩展的图标放在 Chrome 的工具栏中，`browser_action`中的`default_icon`属性定义了相应图标文件的位置，`default_title`定义了当用户鼠标悬停于扩展图标上所显示的文字，`default_popup`则定义了当用户单击扩展图标时所显示页面的文件位置。

接下来我们开始编写 popup.html。

```
<html>
<head>
<style>
* {
    margin: 0;
    padding: 0;
}

body {
    width: 200px;
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
<div id="clock_div"></div>
<script src="js/my_clock.js"></script>
</body>
</html> 
```

如果你曾经编写过网页，会发现上面这个页面省略了很多内容，比如`<title>`标签。因为对于 Chrome 扩展来说，很多对网页有意义的内容是无意义的，所以我们可以只挑需要的写，当然你全写出来也不会有什么问题。

上面的这个页面首先定义了全局元素的`margin`和`padding`为 0，这样我们可以更加自由地控制元素的外观。在编写网页时，`body`的尺寸往往不会专门给定，但对于 Chrome 扩展有时这是必要的，比如此例中我们需要告诉 Chrome 当用户单击扩展图标后展示一个多大的界面。

之后我们在`body`标签中定义了一个`id`为`clock_div`的`div`容器，用这个容器来显示当前的时间，这样我们就把 HTML 的布局写好了。接下来我们就需要引入 JavaScript 处理数据并动态显示了。值得注意的是 Chrome 不允许将 JavaScript 代码段直接内嵌入 HTML 文档，所以我们需要通过外部引入的方式引用 JS 文件。当然`inline-script`也是被禁止的，所以所有元素的事件都需要使用 JavaScript 代码进行绑定，如果你没有使用一个拥有强大选择器的库（如 jQuery），最好给需要绑定事件的元素分配一个`id`以便进行操作。

下面来编写 my_clock.js 文件。

```
function my_clock(el){
    var today=new Date();
    var h=today.getHours();
    var m=today.getMinutes();
    var s=today.getSeconds();
    m=m>=10?m:('0'+m);
    s=s>=10?s:('0'+s);
    el.innerHTML = h+":"+m+":"+s;
    setTimeout(function(){my_clock(el)}, 1000);
}

var clock_div = document.getElementById('clock_div');
my_clock(clock_div); 
```

在 my_clock.js 文件中我们定义了一个`my_clock`函数用于显示时间，这个函数包含了一个`el`参数，这个参数为显示时间的容器，由于在 HTML 文档中我们设计在`id`为`clock_div`的`div`容器中显示时间，所以调用`my_clock`函数时我们传入了这个容器，在 js 文件中用变量`clock_div`表示。之后`my_clock`函数 1000 毫秒之后又会再次调用自身，这样`clock_div`中显示的时间就会被更新。

至此这个扩展就编写完毕了，当然别忘了将图标文件也放入相应的文件夹中。

![enter image description here](img/00002.jpeg)
*扩展的文件结构*

下面我们就需要将这个扩展载入 Chrome 中运行了。依次点击“![enter image description here](img/00003.jpeg)”-“工具”-“扩展程序”打开扩展程序页面（也可以直接在地址栏中输入 chrome://extensions 进入），勾选右上角的“开发者模式”，点击“加载正在开发的扩展程序”，选择扩展所在的文件夹，就可以在浏览器工具栏中看到我们的扩展了。

![enter image description here](img/00004.jpeg)
*将扩展载入到 Chrome 中*

当鼠标点击扩展图标后，一个显示时钟的界面就出现了。

![enter image description here](img/00005.jpeg)
*时钟扩展的运行界面*

这个扩展的源代码可以通过[`github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/my_clock`](https://github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/my_clock)下载。

## 1.3　Manifest 文件格式

Chrome 扩展都包含一个 Manifest 文件——manifest.json，这个文件可以告诉 Chrome 关于这个扩展的相关信息，它是整个扩展的入口，也是 Chrome 扩展必不可少的部分。

Manifest 文件使用 JSON 格式保存数据，为了避免有的读者对 JSON 不了解而无法继续阅读，下面我将简单介绍一下 JSON。JSON 是 JavaScript Object Notation 的缩写，这是一种基于 JavaScript 语言的轻量级数据交换格式。由于 JSON 储存的数据冗余度比 XML 更低，而且便于读取，所以也被很多其他语言所支持，现在 JSON 已经成为一种跨平台跨语言的通用数据交换格式。

JSON 包含两种结构：一种是`key:value`对的形式，名称和值之间用冒号（`:`）连接，多个`key:value`对之间用逗号（`,`）连接，最后在整个对象两侧加上“`{`”和“`}`”；另一种是值的有序集合，值与值之间用逗号（`,`）连接，最后在整个数组两侧加上“`[`”和“`]`”。

![enter image description here](img/00006.jpeg)
*对象形式的结构，图片来源于 www.json.org*

![enter image description here](img/00007.jpeg)
*数组形式的结构，图片来源于 www.json.org*

其中无论是对象形式还是数组形式，它们的值均可以是字符串、数字、对象、数组、布尔和`null`中的一种，也就是说 JSON 有嵌套的性质，值也可以是 JSON 格式的数据。

下面给出了一个 JSON 的例子：

```
{
    "name" : "Harry Potter",
    "author" : {
        "name" : "J.K.Rowling",
        "birth" : 1964
    },
    "books" : [
        "Harry Potter and the Philosopher's Stone",
        "Harry Potter and the Chamber of Secrets",
        "Harry Potter and the Prisoner of Azkaban",
        "Harry Potter and the Goblet of Fire",
        "Harry Potter and the Order of the Phoenix",
        "Harry Potter and the Half-Blood Prince",
        "Harry Potter and the Deathly Hallows"
    ]
} 
```

上述例子中的 JSON 整体是一个对象的形式，这个对象包含三个属性，分别是`name`、`author`和`books`。其中`name`的值是字符串，为`"Harry Potter"`；`author`的值是一个对象，这个对象有两个属性，分别是`name`和`birth`，`name`的值是字符串，为`"J.K.Rowling"`，`birth`的值是数字，为`1964`，可以说`author`的值也是一个 JSON 格式的数据；`books`的值是数组，这个数组包含七个元素，每个元素都是一个字符串。

接下来我们看看 Chrome 扩展中 Manifest 的内容。Google 的官方文档中对于扩展和应用给出了两个不同的 Manifest 介绍界面，这是因为有些属性只能由扩展使用，而有些属性只能由应用使用。如果这两者同时出现在同一个 Manifest 文件中，就会使 Chrome 困惑，不知是按照扩展对待这个程序还是按照应用来对待这个程序。但无论是扩展还是应用，它们的 Manifest 又有很多共有的属性，所以我决定还是放到一起讲。

Chrome 扩展的 Manifest 必须包含`name`、`version`和`manifest_version`属性，目前来说`manifest_version`属性值只能为数字 2，对于应用来说，还必须包含`app`属性。

其他常用的可选属性还有`browser_action`、`page_action`、`background`、`permissions`、`options_page`、`content_scripts`，所以我们可以保留一份 manifest.json 模板，当编写新的扩展时直接填入相应的属性值。如果我们需要的属性不在这个模板中，可以再去查阅官方文档，但我想这样的一份模板可以应对大部分的扩展了。

```
{
    "app": {
        "background": {
            "scripts": ["background.js"]
        }
    },
    "manifest_version": 2,
    "name": "My Extension",
    "version": "versionString",
    "default_locale": "en",
    "description": "A plain text description",
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
        "default_title": "Extension Title",
        "default_popup": "popup.html"
    },
    "page_action": {
        "default_icon": {
            "19": "images/icon19.png",
            "38": "images/icon38.png"
        },
        "default_title": "Extension Title",
        "default_popup": "popup.html"
    },
    "background": {
        "scripts": ["background.js"]
    },
    "content_scripts": [
        {
            "matches": ["http://www.google.com/*"],
            "css": ["mystyles.css"],
            "js": ["jquery.js", "myscript.js"]
        }
    ],
    "options_page": "options.html",
    "permissions": [
        "*://www.google.com/*"
    ],
    "web_accessible_resources": [
        "images/*.png"
    ]
} 
```

在官方文档中可以找到完整的 Manifest 属性列表，扩展在[`developer.chrome.com/extensions/manifest，应用在。由于 Google 更新得非常频繁，上述页面内容可能会经常变动，但那些比较基本的属性变动的几率不会很大。`](https://developer.chrome.com/extensions/manifest，应用在<https://developer.chrome.com/apps/manifest)

## 1.4　DOM 简述

DOM 是 Document Object Model 的缩写，翻译过来叫文档对象模型，但我觉得这个听起来很生疏，不如还是直接叫 DOM，所以本节的标题就定为了 DOM 简述。由于 Chrome 扩展应用使用 HTML 渲染界面，所以不可避免地要接触 DOM。考虑到并非所有读者都编写过 HTML，我决定单独拿出一小节来讲解 DOM，帮助这些读者快速入门。当然，用短短的一节是无法讲透的——毕竟 DOM 可以写另外一本书了——这里只是要给大家引出一个方向，浅浅地打下一点基础，深入的学习还需要读者去阅读更加详细的资料。

![enter image description here](img/00008.jpeg)
*HTML DOM 树，图片来源于 www.w3school.com.cn*

DOM 分为 3 个不同的部分，分别是核心 DOM、XML DOM 和 HTML DOM，我们主要关心的是 HTML DOM，所以我也只讲解 HTML DOM。

上图给出了 HTML DOM 的树状结构图，可以看到 HTML 文档都有一个`<html>`根元素。`<html>`根元素又有两个子元素，分别是`<head>`和`<body>`，所以已经最简单而完整的 HTML 文档如下所示：

```
<html>
    <head></head>
    <body></body>
</html> 
```

这个文档没有任何内容，但拥有 HTML 完整的结构。在 DOM 中，每个元素通常是以`<tag_name>`的形式开始，并以`</tag_name>`的形式结束。在 HTML 中，有一些特定的`tag_name`，如`div`、`p`、`a`、`form`等等。

这些元素可以包含一些属性，还可以包含子节点，子节点可以是元素也可以是文本。如：

```
<img src="images/dog.png" />
<div>Hello World!</div> 
```

上面的例子中`img`元素不是以成对的标签形式出现的，而是在标签内部末尾使用“`/`”闭合标签，这样的元素在 HTML 文档中没有子节点，所以称为自闭标签。类似的元素还有`input`。

除了自闭标签，其他的标签必须成对出现，并且嵌套规则必须明确，这有点像我们小学时学习数学所使用的括号“`()`”和中括号“`[]`”。比如下面的嵌套方式是正确的：

```
<div><p>Hello World!</p></div> 
```

但下面的例子是错误的：

```
<div><p></div></p>
<div><p></div> 
```

第一个是嵌套错误，第二个是`p`标签没有成对出现，标签没有闭合。

有时元素还会拥有属性，比如下面的例子：

```
<input type="text" id="stu_name" value="Billy" /> 
```

上面这个`input`有三个属性，分别是`type`、`id`和`value`，`type="text"`表明这个输入框的类型是文本输入框，`id="stu_name"`表明给这个元素分配了一个名为`stu_name`的`id`，这样可以更加方便地被 JavaScript 和 CSS 选择器定位到，`value="Billy"`表明将这个输入框的默认值设定为`Billy`。

不同的元素往往拥有不同的属性名，比如对于`img`元素，通常会包含`src`属性以指定所显示图片的地址，而`input`元素往往会包含`type`属性来描述输入框的类型。

在 JavaScript 中有多种获取 DOM 元素的方法，常见的有`getElementById`、`getElementsByName`、`getElementsByTagName`、`getElementsByClassName`，分别是通过`id`、`name`、标签名和类名获取元素。

请注意，上面提到的四种方法中，第一个方法名中是`Element`，而后面的都是`Elements`。这是因为 HTML 中元素的`id`必须是唯一的，但是不同的元素可以拥有同样的`name`、标签名和类名，所以通过第一种方式获取的是一个元素，而后几种方法获取的是一个包含多个元素的数组。值得强调的是，即使 HTML 中只有一个元素的 name 为`"my_element"`，那么通过`getElementsByName('my_element')`获取到的也是数组型的数据——虽然这个数组只包含一个元素。

JavaScript 可以通过`getAttribute`方法读取元素的属性，通过`setAttribute`方法添加或更改元素的属性，通过`removeAttribute`方法删除元素的属性。对于非自定义的属性，JavaScript 可以直接像读取对象属性那样读取或更改它们，比如：

```
var imgurl = document.getElementById('my_image').src;
document.getElementById('my_another_image').src = imgurl;
// var imgurl = document.getElementById('my_image').getAttribute('src');
// document.getElementById('my_another_image').setAttribute('src', imgurl); 
```

CSS 的选择器基本分为三种，分别是`tagName`、`.className`和`#id`。如下面的例子：

```
p {
    width: 200px;
}

.postlist {
    width: 150px;
}

#footer {
   width: 100px;
} 
```

分别定义了`p`标签元素宽度为 200 像素，类名为`postlist`的元素宽度为 150 像素，`id`为`footer`的元素宽度为 100 像素。这个样式表分别作用于以下元素：

```
<p></p>
<div class="postlist"></div>
<div id="footer"></div> 
```

CSS 选择器还可以通过元素属性进行定位，比如下面的例子可以作用于所有文本输入框：

```
input[type="text"] {
    font-size: 16px;
} 
```

更多关于 DOM 的知识可以参阅[`www.w3school.com.cn/htmldom/`](http://www.w3school.com.cn/htmldom/)。