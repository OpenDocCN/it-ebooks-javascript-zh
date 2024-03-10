# 第三章　Chrome 扩展的 UI 界面

前两章我们所设计的扩展，使用的 UI 设计都非常简单。对于一个面向用户的产品，这样显然是不合适的。用户对一个程序的第一印象就是 UI 的设计，拙劣的 UI 设计完全可能将 90%的用户挡在门外——即使功能设计得非常完美。

本章将专门讲解 Chrome 扩展的 UI 界面，通过 Chrome 提供丰富的界面 API，我们可以设计出交互出色的扩展。

## 3.1　CSS 简述

CSS 是 Cascading Style Sheets 的缩写，翻译过来叫做层叠样式表，一般简称为样式表，但通常大家还是习惯叫 CSS。

最初的 HTML 很单一，甚至无法显示图片，随着使用范围越来越广泛，HTML 支持的标签开始多了起来，所支持的样式也开始增多。但是把样式完全交给 HTML 去做不是一个好想法，因为 HTML 更侧重于页面的结构，于是在 1994 年 CSS 被提出。CSS 旨在对 HTML 元素的外观加以描述，来提供更多更加复杂丰富的样式。

现在多数浏览器会默认使用一些样式，比如`div`元素默认会占据整行——两个`div`元素不会出现在同一行，而`span`元素则不是这样，这就是因为浏览器默认将`div`元素样式的`display`属性值设为了`block`。

浏览器这种为 HTML 元素附加默认样式的做法，大部分情况下是好的，但有时为了个性化设计，我们需要另外编写 CSS 来自定义 HTML 元素的外观。

![enter image description here](img/00019.jpeg)
*Chrome 会自动为 HTML 元素附加 margin 和 padding 样式*

我们来看看上图所示的页面在 Chrome 浏览器中的渲染结果。可以看到 HTML 元素并没有被指定样式，因为我们没有编写 CSS。但是 Chrome 已经自动为文本框添加了`margin`和`padding`样式，这在外观表现上，会在文本框周围有一圈间隙，这样其他 HTML 元素不会与它挨得太紧。这种设计显然是出于好意，但有时我们需要更加灵活个性化的样式，这就是为什么在前面的例子中都会出现下面的代码。

```js
* {
    margin: 0;
    padding: 0;
} 
```

CSS 的选择器在第一章第 4 节已经介绍过，在此就不再赘述。下面讲一讲 CSS 的基本语法。

CSS 是一种描述型语言，它更像是一种陈述，而不是逻辑运算。CSS 的语法形式如下所示：

```js
选择器 {
    属性名: 属性值;
} 
```

CSS 的选择器非常灵活，更加高级的使用方法大家可以参考相关的书籍。CSS 的属性名也非常丰富，涉及到尺寸、边框、边距、位置、层叠顺序、文字（包括颜色、字体、粗细等等）和背景等等。

CSS 使用 box 模型处理元素的尺寸、边框和边距，下图展示了它们之间的关系。

![enter image description here](img/00020.jpeg)
*CSS 的 box 模型*

那么`margin`和`padding`有什么区别呢？`padding`区域算元素的一部分，所以元素的背景样式同样也会适用于`padding`区域。比如元素背景颜色设定为灰色，`padding`区域的背景颜色也会是灰色的，就如上图所示的那样。

需要注意的地方是，虽然`padding`是元素的内边距，也算元素的一部分，但元素的高度和宽度却并不包含`padding`区域。

元素的`margin`、`padding`、`height`和`width`的单位一般为`px`，即像素，也可以使用百分百的形式，如`50%`。如果使用的是百分百的形式，所相对的是此元素指定了绝对尺寸的父系元素。比如下面的例子：

```js
<div id="outer" style="width: 500px">
    <div id="inner">
        <div id="content" style="width: 80%">Hello</div>
    </div>
</div> 
```

其中`id`为`content`的`div`元素的宽度为`80%`，这`80%`是相对`id`为`outer`的元素而言的。虽然`content`的直接父系元素为`inner`，但是由于`inner`并没有指定宽度，所以会继续向上寻找父系元素，直到找到定义了`width`的元素为止。如果所有的父系元素都没有指定，则这个值是相对于`body`的。

对于元素的位置，默认情况就像报纸排版一样，将一个板块设计好之后，下一个板块会接着排列。这种像瀑布一样的排列方式我们形象地称为 HTML 流（flow）。默认情况下元素的`position`属性值为`static`，元素排列在正常的流中。`position`属性还有另外的三个值，分别是`absolute`、`relative`和`fixed`。如果元素的位置属性为`absolute`，则它的位置是相对于除`static`定位以外的父系元素的，如果没有这样的父系元素，则相对于`body`；如果元素的位置属性为`relative`，则它的位置是相对于它默认在 HTML 流中位置的；如果元素的位置属性为`fixed`，则它的位置是相对于浏览器窗口的。

![enter image description here](img/00021.jpeg)
*不同位置属性的元素的定位效果*

上图中浅灰色的元素是所有元素的父系元素，它的`position`属性为`relative`。深灰色和黑色边框的元素`position`都是默认的`static`，所以它们按照 HTML 流的方式依次布局。黑色的元素拥有`absolute`的位置属性，并指定`left`为`10px`，`top 为 10px`，它的定位是相对于浅灰色元素的。

对于`relative`定位，更像是对原有定位的偏移。对于`left`来说，负值向元素的左侧偏移，正值向右侧偏移，`right`与其相反；对于`top`来说，负值向元素的上侧偏移，正值向下侧偏移，`bottom`与其相反。需要注意的是，`relative`定位所定义的偏移不会影响元素原本在 HTML 流中的位置，下图给出了说明。

![enter image description here](img/00022.jpeg)
*relative 定位所定义的偏移不会影响元素原本在 HTML 流中的位置*

虽然中间深灰色的元素相对于 HTML 流中的位置产生了偏移，但它原本在 HTML 流中的位置却没有改变，所以并没有影响下面浅灰色元素的位置。

默认情况下，如果元素和元素有重叠的部分，在 HTML 文档中靠后的元素会被显示在上面。但是可以通过 CSS 的`z-index`属性改变层叠顺序，`z-index`的值大的元素会显示`在 z-index`值小的元素下面。如果一个元素没有被指定`z-index`的值，则在 Chrome 中默认为`0`（注意，并非所有浏览器都是这样，比如 IE 默认为负无穷大）。`position`属性为`static`的元素（即没有指定`position`属性的元素）`z-index`的值会被浏览器忽略。

CSS 还可以定义元素中文字的大小、字体和颜色等，高级的属性还可以定义文字之间的距离、段首缩进、文字阴影等特殊的效果，下面我们主要讲讲对文字大小、字体和颜色的控制。

CSS 使用`font-size`属性控制文字的大小，`font-size`的值可以是固定值也可以是百分比。如果是百分比，则相对的是父系的文字尺寸。如果是固定值，常见的单位有`px`、`pt`和`em`，另外还有一些其他的单位，如`in`、`cm`、`mm`、`ex`和`pc`。`px`最好理解，就是像素，和其他属性一个道理；`pt`是印刷界的单位，这个单位与物理尺寸相对应，如果使用`pt`作为单位，则在任何设备上，显示出来的大小都是一样的；`em`是个相对的单位，它是相对于元素当前文字尺寸的，比如元素当前文字尺寸为`16px`，则`font-size`为`2em`，显示出来的文字大小为`32px`。另外也可以使用特定的常量来设定文字大小，如`xx-small`、`medium`和`large`等，`smaller`和`larger`则把`font-size`设置为比父元素更小和更大的尺寸。

文字的字体使用`font-family`属性控制，这个属性可以有多个值。浏览器优先使用靠前的值，但如果用户的系统中没有安装指定的字体，则浏览器就会考虑使用后面的值，如果所有指定的字体用户的系统中都没有，则浏览器使用默认字体。对于 Windows 操作系统，中文的默认字体一般是宋体。需要注意的是，如果字体的名称中包含空格，需要用引号将字体名包含，多个值之间用逗号隔开。

文字的颜色使用`color`属性控制，`color`的值常见的有三种方式，分别是颜色名、十六进制颜色值和 rgba。除此之外还可以使用 HSL 和 HSLA 格式。颜色名有`black`、`red`等，网上可以找到一份比较全的颜色名列表。但是能用名称表示的颜色十分有限，多数情况还是需要用颜色值表示。用十六进制的颜色值表示颜色的方法，是一个`#`符号后面接着 6 位十六进制数值，这 6 位数值每两位为一组，从左至右分别代表红色、绿色和蓝色的强度，`#000000`代表黑色，`#FFFFFF`代表白色。有时我们会遇到用三位十六进制数值表示颜色的情况，这是颜色值的缩略表示方式，表示每组颜色的十六进制码两位相同，如`#ABC`和`#AABBCC`表示的颜色相同。rgba 表示方式除了包含红绿蓝三种颜色强度外还包含不透明度。其中前三个数字表示色值，第四个数字表示透明度。表示色值的数字有效值为 0-255 的整数或百分比（百分比也可以表示成小数，如 50%也可以用 0.5 表示），表示透明度的数字有效值为 0-1 的小数。比如`rgba(255, 0, 0, 0.5)`表示透明度为 0.5 的红色。

CSS 可以通过`font`属性将多种和文字相关的属性连在一起作为值，这种方式对初学者来说不直观，但对于熟练的人是个节省时间的好办法。

另外不得不再提一下`line-height`这个属性。对于文字来说，它每行所占据的高度是它的大小决定的，默认情况下两行相邻的文字不会重叠，也不会离得太远。下图展示了通过调整`line-height`属性使得两行相邻的文字重叠。

![enter image description here](img/00023.jpeg)
*调整 line-height 属性使得两个相邻行的文字重叠*

当想让文字在元素中垂直居中时，就可以通过指定`line-height`与`height`相同而达到目的¹。

^(1 也可以使用`vertical-align`属性控制垂直位置。)

CSS 还可以控制元素的背景颜色和背景图片。背景颜色通过`background-color`进行控制，值的形式与`color`属性相同。背景图片通过`background-image`进行控制，值为`url(图片位置)`。对于背景图片，往往还要结合`background-repeat`和`background-position`使用。前者是控制图片重复的方式，默认是平铺，还可以指定为`repeat-x`（横向重复）、`repeat-y`（纵向重复）和`no-repeat`（不重复）。`background-position`是控制背景图片的位置，值的形式可以是`top`、`bottom`、`left`、`right`和`center`的结合，比如`top left`为左上角，`center left`为左侧中间，如果只指定了一个值，则另一个值默认为`center`。也可以是`x% y%`的形式，同样是相对于父系元素尺寸的。也可以以像素为单位，如`10px 20px`为距左侧 10 像素，上侧 20 像素。

背景样式还有很多更加丰富的属性，如`background-size`，这些更加高级的属性留给感兴趣的读者自行研究吧 :)

最后再强调一次，本节只是对 CSS 的简述，如果想学好 CSS 还应参考相关更加专业的书籍和资料，本节的作用只是避免没有任何基础的读者阅读后面的内容有障碍。

## 3.2　Browser Actions

Browser Actions 将扩展图标置于 Chrome 浏览器工具栏中，地址栏的右侧。如果声明了 popup 页面，当用户点击图标时，在图标的下侧会打开这个页面¹。同时图标上面还可以附带 badge——一个带有显示有限字符空间的区域——用以显示一些有用的信息，如未读邮件数、当前音乐播放时间等。

^(1 如果没有足够的空间，会在图标的上侧打开。)

下面将对 Browser Actions 中的图标、popup 页面、标题和 badge 做详细介绍。

### 3.2.1　图标

Browser Actions 可以在 Manifest 中设定一个默认的图标，比如：

```js
"browser_action": {
    "default_icon": {
        "19": "images/icon19.png",
        "38": "images/icon38.png"
    }
} 
```

一般情况下，Chrome 会选择使用 19 像素的图片显示在工具栏中，但如果用户正在使用视网膜屏幕的计算机，则会选择 38 像素的图片显示。两种尺寸的图片并不是必须都指定的，如果只指定一种尺寸的图片，在另外一种环境下，Chrome 会试图拉伸图片去适应，这样可能会导致图标看上去很难看。

另外，`default_icon`也不是必须指定的，如果没有指定，Chrome 将使用一个默认图标。

通过`setIcon`方法可以动态更改扩展的图标，`setIcon`的完整方法如下：

```js
chrome.browserAction.setIcon(details, callback) 
```

其中`details`的类型为对象，可以包含三个属性，分别是`imageData`、`path`和`tabId`。

`imageData`的值可以是`imageData`，也可以是对象。如果是对象，其结构为`{size: imageData}`，比如`{'19': imageData}`，这样可以单独更换指定尺寸的图片。`imageData`是图片的像素数据，可以通过 HTML 的`canvas`标签获取到。

`path`的值可以是字符串，也可以是对象。如果是对象，结构为`{size: imagePath}`。`imagePath`为图片在扩展根目录下的相对位置。

不必同时指定`imageData`和`path`，这两个属性都是指定图标所要更换的图片的。

`tabId`的值限定了浏览哪个标签页时，图标将被更改。

`callback`为回调函数，当`chrome.browserAction.setIcon`方法执行成功后，`callback`指定的函数将被运行。此函数没有可接收的回调结果。

下面来编写一个图标不停旋转的扩展。

首先在 Manifest 中定义如下`browser_action`：

```js
"browser_action": {
    "default_icon": {
        "19": "images/icon19_0.png",
        "38": "images/icon38_0.png"
    },
    "default_title": "Turtle"
} 
```

为了让图标动起来，需要一个 background 脚本在后台不停地换图标，这个脚本如下：

```js
function chgIcon(index){
    if(index === undefined){
        index = 0;
    }
    else{
        index = index%20;
    }
    chrome.browserAction.setIcon({path: {'19': 'images/icon19_'+index+'.png'}});
    chrome.browserAction.setIcon({path: {'38': 'images/icon38_'+index+'.png'}});
    setTimeout(function(){chgIcon(index+1)},50);
}

chgIcon(); 
```

为了达到动态旋转的效果，我们需要制作多张图片连续替换，这和 gif 的工作原理是一样的。

如果你不想用这种费力的方法，可以只制作两幅图片，分别对应于 19 像素和 38 像素两个尺寸，在 background 中通过`canvas`绘图来动态改变图片角度，然后输出`imageData`。感兴趣的读者可以搜索 HTML5 canvas 了解更多，本文在此不做详细介绍。

本小节编写的扩展源码可以通过[`github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/browser_actions_icon`](https://github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/browser_actions_icon)下载到。

### 3.2.2　Popup 页面

Popup 页面是当用户点击扩展图标时，展示在图标下面的页面。下图是“网易云音乐（Unofficial）”扩展的 popup 页面。

![enter image description here](img/00024.jpeg)
*“网易云音乐（Unofficial）”扩展的 popup 页面*

Popup 页面提供了一个简单便捷的 UI 接口。由于有时新窗口会使用户反感，而 popup 页面的设计更像是浏览器的一部分，看上去更加友好，但 popup 页面并不适用于所有情况。由于其在关闭后，就相当于用户关闭了相应的标签页，这个页面不会继续运行。当用户再次打开这个页面时，所有的 DOM 和 js 空间变量都将被重新创建。

所以，popup 页面更多地是用来作为结果的展示，而不是数据的处理。通常情况下，如果需要扩展实时处理数据，而不是只在用户打开时才运行，我们需要创建一个在后端一直运行的页面或者脚本，这可以通过 manifest.json 的`background`域来声明，具体请参考 2.3 节所讲述的内容。而 popup 页面获取后端运行的结果，可以通过扩展内部的通信接口来完成，具体请参考 2.5 节所讲述的内容。

下面来重点讲述一下 popup 页面的设计和需要注意的地方。

Popup 页面是一个扩展与用户交互的窗口，这个窗口设计的好坏直接会影响到用户的使用体验，所以更应加以重视。

首先，popup 页面会根据内容自动显示合适的大小，但还是建议给出页面中`body`元素的尺寸，主要原因有两点：第一，如果不指定页面尺寸，在用户点击图标的瞬间会打开一个很小的窗口，DOM 渲染完毕后页面尺寸才会正常，这个小的细节可能会给用户带来不好的体验；第二，我们也应该设计尺寸可控的页面，这样才能更好地优化布局，同时又不会出现小屏幕设备无法完全显示的情况。考虑到部分小尺寸的上网本，建议 popup 页面的高度最好不要超过 500 像素。当然也可以先给出一个默认的 500 像素高度，之后再通过 js 获取当前设备屏幕的尺寸后，再决定是否需要更改这个高度。但请注意，一个默认的页面尺寸是必要的。

不要尝试模仿 Chrome 的原生 UI。有的开发者为了使自己的扩展看上去更像 Chrome 的一部分，而去刻意模仿 Chrome 的原生 UI，这样做并不值得鼓励。首先我们应该让用户一眼就能看出，哪些部分是 Chrome 自带的功能，哪些部分是来自第三方提供的扩展功能，混淆用户的判断不是一个好主意。另外如果用户使用的不是 Chrome 的默认主题，这种设计看上去将很不协调。

**使用带有滚动条的 DIV 容器。**指定`body`元素的尺寸后，如果页面内容过长，就会被撑开，导致高度不可控，这可不是我们想要的结果。一个可行的解决办法是通过带有滚动条的 DIV 容器来防止页面被撑开。通常需要设定 DIV 容器的高度为`100%`，`overflow-y`属性为`auto`，这样当不需要滚动条时，DIV 容器将不会显示滚动条。

![enter image description here](img/00025.jpeg)
*设定 body 高度为 200 像素后不使用 DIV 容器和使用 DIV 容器的对比*

**设计一个更好的滚动条样式。**每款浏览器都有自己默认的滚动条样式，一般情况下不需要去更改它们——有些浏览器也不允许我们去更改，但对于扩展的 popup 页面则是另外一回事了。首先我们并不是在设计一个网站，而是在设计一款程序，滚动条作为程序的一部分，应尽量和程序的整体风格保持一致；再者，Chrome 默认的滚动条，对尺寸有限的 popup 页面来说显得过于臃肿。“网易云音乐（Unofficial）”扩展就有一个自定义的滚动条样式，这要比直接使用默认的滚动条强上百倍。关于如何通过 CSS 来自定义 webkit 内核浏览器的滚动条样式，可以通过[`css-tricks.com/custom-scrollbars-in-webkit/`](http://css-tricks.com/custom-scrollbars-in-webkit/)了解更多。

**考虑屏蔽右键菜单。**如果你是一个追求尽善尽美的开发者，也许不希望用户在你的扩展页面上点击右键时，会出现 Chrome 的默认菜单，取而代之的应该是你自己设计的菜单。你要做的就是屏蔽掉右键，同时通过 DIV 浮动层模拟出一个自己的菜单。但需要注意的是，由于这个模拟的菜单还是在 popup 窗口之内的 DOM 元素，所以它不会像系统菜单那样可以超越页面边界。在设计这个菜单时要考虑到会不会有部分被遮挡，在用户点击鼠标唤出菜单前，请先让一段代码决定这个菜单适当的显示位置。

**使用外部引用的脚本。**这并不只是 popup 页面需要注意的地方，实际上 Google 在之前较早的某个版本开始，就不再允许 HTML 和 JavaScript 写在一个文件里了。所以要通过`<script src="script-path/script-name.js"></script>`来引用外部的脚本，而不是将 JavaScript 代码直接写在`<script>`标签内。

**不要在 popup 页面的 js 空间变量中保存数据。**由于 popup 页面只在用户点击图标时才会开启，当用户关闭这个页面时就会停止，并没有一个从始至终的实例分配给 popup 页面。所以每当用户打开 popup 页面时，它都是崭新的，之前保存在变量中的数据都会消失。如果需要通过 popup 页面保存用户的数据，可以通过通信将数据交给后台页面处理，或者通过`localStorage`和`chrome.storage`将数据保存在用户的硬盘上。

### 3.2.3　标题和 badge

将鼠标移至扩展图标上，片刻后所显示的文字就是扩展的标题。

在 Manifest 中，`browser_action`的`default_title`属性可以设置扩展的默认标题，比如如下的例子：

```js
"browser_action": {
    "default_title": "Extension Title"
} 
```

在这个扩展中，默认标题就是“Extension Title”。还可以用 JavaScript 来动态更改扩展的标题，方法如下：

```js
chrome.browserAction.setTitle({title: 'This is a new title'}); 
```

标题不仅仅只是给出扩展的名称，有时它能为用户提供更多的信息。比如一款聊天客户端的标题，可以动态地显示当前登录的帐户信息，如号码和登录状态等。所以如果能合理使用好扩展的标题，会给用户带来更好的体验。

标题我们已经清楚了，那么什么是 badge 呢？我们来看一幅图：

![enter image description here](img/00026.jpeg)
*扩展的标题和 badge*

上图中，标有“Extension Title”的地方就是扩展标题，而标有“Badg”的地方就是 badge。Badge 是扩展为用户提供有限信息的另外一种方法，这种方法较标题优越的地方是它可以一直显示，其缺点是只能显示大约 4 字节长度的信息，这就是为什么上例中显示的是“Badg”而不是“Badge”。

目前来看使用 badge 比较典型的应用是音乐播放器，它们使用 badge 显示当前音乐播放的时间；另一些内容类的应用，如邮件、微博、RSS 阅读器等，则显示未读条目。无论你打算用 badge 显示何种信息，请记住它只能显示 4 字节长度的内容。对于内容类的扩展，当用户未读条目足够多时，一般采用的解决方法是显示“999+”。

Badge 目前只能够通过 JavaScript 设定显示的内容，同时 Chrome 还提供了更改 badge 背景的方法。如果不定义 badge 的背景颜色，默认将使用红色，就是上图显示的那样。

下面的代码显示了一个背景颜色为蓝色，内容为“Dog”的 badge：

```js
chrome.browserAction.setBadgeBackgroundColor({color: '#0000FF'});
chrome.browserAction.setBadgeText({text: 'Dog'}); 
```

对于背景颜色的设定，设定值可以是十六进制的字符串颜色码，如`#FF0000`代表颜色；也可以是 rgba 格式的数组，但需要注意的是其中的 alpha 变量的取值范围同样为 0-255，这与 CSS 有所区别。

下面的例子使用 rgba 的定义方式，将背景设置为 50%透明度的绿色：

```js
chrome.browserAction.setBadgeBackgroundColor({color: [0, 255, 0, 128]}); 
```

最后需要注意的一点，就是 badge 目前还不支持更改文字的颜色——始终是白色，所以应避免使用浅颜色作为背景。

## 3.3　右键菜单

当用户在网页中点击鼠标右键后，会唤出一个菜单，在上面有复制、粘贴和翻译等选项，为用户提供快捷便利的功能。Chrome 也将这里开放给了开发者，也就是说我们可以把自己所编写的扩展功能放到右键菜单中。

要将扩展加入到右键菜单中，首先要在 Manifest 的`permissions`域中声明`contextMenus`权限。

```js
"permissions": [
    "contextMenus"
] 
```

同时还要在`icons`域声明 16 像素尺寸的图标，这样在右键菜单中才会显示出扩展的图标。

```js
"icons": {
    "16": "icon16.png"
} 
```

Chrome 提供了三种方法操作右键菜单，分别是`create`、`update`和`remove`，对应于创建、更新和移除操作。

通常`create`方法由后台页面来调用，即通过后台页面创建自定义菜单。如果后台页面是 Event Page，通常在`onInstalled`事件中调用`create`方法。

右键菜单提供了 4 种类型，分别是普通菜单、复选菜单、单选菜单和分割线，其中普通菜单还可以有下级菜单。连续相邻的单选菜单会被自动认为是对同一设置的选项，同时单选菜单会自动在两端生成分割线。下面的代码生成了一系列的菜单：

```js
chrome.contextMenus.create({
    type: 'normal',
    title: 'Menu A',
    id: 'a'
});

chrome.contextMenus.create({
    type: 'radio',
    title: 'Menu B',
    id: 'b',
    checked: true
});

chrome.contextMenus.create({
    type: 'radio',
    title: 'Menu C',
    id: 'c'
});

chrome.contextMenus.create({
    type: 'checkbox',
    title: 'Menu D',
    id: 'd',
    checked: true
});

chrome.contextMenus.create({
    type: 'separator'
});

chrome.contextMenus.create({
    type: 'checkbox',
    title: 'Menu E',
    id: 'e'
});

chrome.contextMenus.create({
    type: 'normal',
    title: 'Menu F',
    id: 'f',
    parentId: 'a'
});

chrome.contextMenus.create({
    type: 'normal',
    title: 'Menu G',
    id: 'g',
    parentId: 'a'
}); 
```

上面的代码生成的菜单如下图所示。

![enter image description here](img/00027.jpeg)
*自定义右键菜单*

我们还可以定义自定义的右键菜单在何时显示，比如当用户选择文本时，或者在超级链接上单击右键时。下面的代码定义当用户在超级链接上点击右键时，在菜单中显示“My Menu”菜单：

```js
chrome.contextMenus.create({
    type: 'normal',
    title: 'My Menu',
    contexts: ['link']
}); 
```

`contexts`域的值是数组型的，也就是说我们可以定义多种情况下显示自定义菜单，完整的选项包括`all`、`page`、`frame`、`selection`、`link`、`editable`、`image`、`video`、`audio`和`launcher`，默认情况下为`page`，即在所有的页面唤出右键菜单时都显示自定义菜单。其中`launcher`只对 Chrome 应用有效，如果包含`launcher`选项，则当用户在 chrome://apps/或者其他地方的应用图标点击右键，将显示相应的自定义菜单。需要注意的是，`all`选项不包括`launcher`。

有时我们不仅想在特定的情况下显示自定义菜单，还希望限定 URL，chrome 同样提供了匹配 URL 的选项。`documentUrlPatterns`允许限定页面的 URL，比如我们可以限定只在 Google 的网站上显示自定义菜单；`targetUrlPatterns`和`documentUrlPatterns`差不多，但它所限定的不是标签的 URL，而是诸如图片、视频和音频等资源的 URL。

如果在创建菜单时，定义了`onclick`域，则菜单被点击后就会调用`onclick`指定的函数。调用的函数会接收到两个参数，分别是点击后的相关信息和当前标签信息。点击后的相关信息包括菜单`id`、上级菜单`id`、媒体类型（`image`、`video`或`audio`）、超级链接目标、媒体 URL、页面 URL、框架 URL、选择的文字、是否可编辑（只针对`text input`和`textarea`等控件）、用户点击前是否被选中和当前是否被选中（只针对`checkbox`或`radio`）。完整的信息结构可以通过[`developer.chrome.com/extensions/contextMenus#type-OnClickData`](http://developer.chrome.com/extensions/contextMenus#type-OnClickData)查看。

`update`方法可以动态更改菜单属性，指定需要更改菜单的 id 和所需要更改的属性即可。`remove`方法可以删除指定的菜单，`removeAll`方法可以删除所有的菜单。

下面我们来创建一个通过右键菜单使用 Google 翻译当前用户所选文本的扩展。我们希望只有当用户选择了文本才显示这个菜单，所以要将`contexts`的值设为`selection`。

```js
chrome.contextMenus.create({
    type: 'normal',
    title: '使用 Google 翻译……',
    id: 'cn',
    contexts: ['selection']
}); 
```

下面来编写调用的函数。Google 翻译可以通过 http://translate.google.com.hk/#auto/zh-CN/{翻译文本}调用，所以只需要获取用户所选择的文本，同时打开这个 URL 就可以了。

```js
function translate(info, tab){
    var url = 'http://translate.google.com.hk/#auto/zh-CN/'+info.selectionText ;
    window.open(url, '_blank');
} 
```

现在我们把`create`函数补充完整，把调用函数添加进去：

```js
chrome.contextMenus.create({
    type: 'normal',
    title: '使用 Google 翻译……',
    contexts: ['selection'],
    id: 'cn',
    onclick: translate
}); 
```

最后把这段代码写进 background.js 中，让扩展在浏览器启动后自动执行就可以了。

![enter image description here](img/00028.jpeg)
*Google 翻译扩展*

但我们发现这样无法在菜单中动态显示用户所选择的内容，那么如何动态显示诸如*用 Google 翻译“XXX”*这样的菜单呢？首先要获取用户所选择的文本，可以通过下面的代码来实现：

```js
window.onmouseup = function(){
    var selection = window.getSelection();
    if(selection.anchorOffset != selection.extentOffset){
        //do something
    }
} 
```

那么这段代码在 background 中执行会成功吗？显然不能，因为 background 和当前页面并不在一个空间中，所以我们需要用`content_script`来注入脚本，对`content_script`不了解的读者可以参考 2.1 节的内容。`content_script`获取到用户所选文字后，就可以通过 2.5 节所讲述的内容，传递给后台页面。

![enter image description here](img/00029.jpeg)
*改进后的 Google 翻译扩展*

由于改进的部分不是本章的重点，所以就不详细讲解了，大家可以参考前面的章节¹。完整的代码可以通过[`github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/google_translate`](https://github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/google_translate)下载得到。

^(1 创建菜单时也可以直接使用`%s`表示选定的文字。)

Chrome 还提供了`onClicked`事件，虽然在`create`方法中可以指定点击时调用的函数，但对于 Event Page 只能通过`onClicked`事件调用函数。Event Page 与一般的 background 类似，但它只按需加载，并不像 background 那样一直驻守后台。

## 3.4　桌面提醒

之前的章节提到过利用标题和 badge 向用户提供有限的信息，那么如果需要向用户提供更加丰富的信息怎么办呢？Chrome 提供了桌面提醒功能，这个功能可以为用户提供更加丰富的信息。

![enter image description here](img/00030.jpeg)
*桌面提醒，图片来自 http://developer.chrome.com*

要使用桌面提醒功能，需要在 Manifest 中声明 notifications 权限。

```js
"permissions": [
    "notifications"
] 
```

创建桌面提醒非常容易，只需指定标题、内容和图片即可。下面的代码生成了标题为“Notification Demo”，内容为“Merry Christmas”，图片为“icon48.png”的桌面提醒窗口。

```js
var notification = webkitNotifications.createNotification(
    'icon48.png',
    'Notification Demo',
    'Merry Christmas'
); 
```

桌面系统窗口创建之后是不会立刻显示出来的，为了让其显示，还要调用`show 方法`：

```js
notification.show(); 
```

需要注意的是，对于要在桌面窗口中显示的图片，必须在 Manifest 的 web_accessible_resources 域中进行声明，否则会出现图片无法打开的情况：

```js
"web_accessible_resources": [
    "icon48.png"
] 
```

如果希望 images 文件夹下的所有 png 图片都可被显示，可以通过如下声明实现：

```js
"web_accessible_resources": [
    "images/*.png"
] 
```

桌面提醒窗口提供了四种事件：`ondisplay`、`onerror`、`onclose`和`onclick`。

除了用户主动关闭桌面提醒窗口外，还可以通过`cancel`方法自动关闭。下面的代码可以实现 5 秒后自动关闭窗口的效果。

```js
setTimeout(function(){
    notification.cancel();
},5000); 
```

由于桌面提醒界面可能将不再支持引入 JS 脚本，桌面提醒窗口与其他界面的通信本节不进行讲解。

桌面提醒已经被纳入了 W3C 草案，相关信息可以访问[`dev.chromium.org/developers/design-documents/desktop-notifications/api-specification`](http://dev.chromium.org/developers/design-documents/desktop-notifications/api-specification)查看。

除此之外，也可以通过 Chrome 提供的`chrome.notifications`方法来创建功能更加丰富的提醒框。

## 3.5　Omnibox

Chrome 和其他浏览器相比一个最大的区别就是地址栏——其实不仅仅是地址栏，而是一个多功能的输入框，Google 将其称为 omnibox（中文为“多功能框”）。我们熟悉的一个功能就是用户可以直接在 omnibox 搜索关键字，Chrome 也将 omnibox 开放给开发者，这使得 omnibox 更加强大。

要使用 omnibox 需要在 Manifest 的`omnibox`域指定`keyword`：

```js
"omnibox": { "keyword" : "hamster" } 
```

同时最好指定一个 16 像素的图标，当用户键入关键字后，这个图标会显示在地址栏的前端。

```js
"icons": {
    "16": "icon16.png"
} 
```

Chrome 会自动将这个图标渲染成灰度图标，而无需开发者指定一个灰度的图标，由于右键菜单等其他地方也会用到 16 像素的图标，所以应该指定一个彩色的图标。

Omnibox 只提供了一个方法，就是`setDefaultSuggestion`，这个方法用来定义默认建议。对于这个默认建议用文字怎么讲解恐怕都不容易讲清楚，那么不妨来看一看设置了默认建议和不设置默认建议的对比：

![enter image description here](img/00031.jpeg)
*未设置默认建议和设置了默认建议的对比*

上图中左侧为未设置默认建议，显示为“运行 XXX 命令：XXX”，这样显然看起来不够友好。右侧则用更加友好的方式显示查询当前美元价格。

默认建议会在用户输入 keyword 之后一直显示在地址栏下方并且紧挨着地址栏，所以设定一个默认建议是必要的，否则简单地显示“运行 XXX 命令：XXX”会让用户摸不到头脑。

Omnibox 有四种事件：`onInputStarted`、`onInputChanged`、`onInputEntered`和`onInputCancelled`，分别用于监听用户开始输入、输入变化、执行指令和取消输入行为。其中执行指令是指用户敲击回车键或用鼠标点击建议结果。

```js
onInputStarted(function(){console.log('Input started.')});
onInputCancelled(function(){console.log('Input cancelled.')}); 
```

上面的代码执行后，用户开始输入和取消输入时，都会在控制台记录相应日志。下面我们重点来讲一讲另外两个事件。

`onInputChanged`事件所承接的只有一个 function 类型的参数，这个 function 参数又有两个承接参数，第一个参数是字符串型，值为用户当前的输入值，第二个参数还是 function 型，用于返回建议结果，建议的结果为数组型数据，数组中的元素是建议结果对象。

```js
chrome.omnibox.onInputChanged.addListener(function(text, suggest){
    suggest([{
        content: text,
        description: 'Search '+text+' in Wikipedia'
    }]);
}); 
```

`onInputEntered`事件同样只有一个 function 类型的承接参数，这个 function 有两个承接参数，第一个是用户输入的值，字符串型，第二个是对结果的建议打开方式，字符串型，但取值范围固定。

```js
chrome.omnibox.onInputEntered.addListener(function(text, disposition){
    switch(disposition){
        case 'currentTab': //do something in the current tab
                 break;
        case 'newForegroundTab': //do something in a new tab and active it
                 break;
        case 'newBackgroundTab': //do something in a new tab
                 break;
    }
}); 
```

下面来制作一款实时查询美元价格的扩展。首先通过异步请求获取 Yahoo 上美元的价格，对这部分不熟悉的读者可以参考前面 2.2 节的内容。获取到数据后我们就要开始编写提供建议的函数了。

```js
function updateAmount(amount, exchange){
    amount = Number(amount);
    if(isNaN(amount) || !amount){
        exchange([{
            'content': '$1 = ¥'+price,
            'description': '$1 = ¥'+price
        },{
            'content': '¥1 = $'+(1/price).toFixed(6),
            'description': '¥1 = $'+(1/price).toFixed(6)
        }]);
    }
    else{
        exchange([{
            'content': '$'+amount+' = ¥'+(amount*price).toFixed(2),
            'description': '$'+amount+' = ¥'+(amount*price).toFixed(2)
        },{
            'content': '¥'+amount+' = $'+(amount/price).toFixed(6),
            'description': '¥'+amount+' = $'+(amount/price).toFixed(6)
        }]);
    }
}

var url = 'http://query.yahooapis.com/v1/public/yql?'+
          'q=select%20Rate%20from%20'+
          'yahoo.finance.xchange%20'+
          'where%20pair%20in%20(%22USDCNY%22)&'+
          'env=store://datatables.org/alltableswithkeys&'+
          'format=json';
var price;

httpRequest(url, function(r){
    price = JSON.parse(r);
    price = price.query.results.rate.Rate;
    price = Number(price);
});

chrome.omnibox.onInputChanged.addListener(updateAmount); 
```

大家可以对照前面所讲解的部分来看这段代码，代码中的每个部分都与前面的讲解有所对应。接下来编写用户执行指令时所运行的函数。

```js
function gotoYahoo(text, disposition){
    window.open('http://finance.yahoo.com/q?s=USDCNY=X');
}

chrome.omnibox.onInputEntered.addListener(gotoYahoo); 
```

此例中并没有理会`disposition`的取值，Chrome 官方也指出`disposition`只是给出结果呈现的建议方式，而非必须遵循的方式，所以是否理会这个值由你自己说了算。

最后就像前面所说的那样，记得设定一个默认的建议，这样会使你的扩展看起来更加友好。

前面讲解默认建议的截图就是这个例子运行的结果，所以在此就不重复贴图了。本例的完整代码可以通过[`github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/usd_price`](https://github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/usd_price)下载，载入扩展后在浏览器地址栏中输入“usd”后按空格键或 Tab 键就可以使用。

## 3.6　Page Actions

Page Actions 与 Browser Actions 非常类似，除了 Page Actions 没有 badge 外，其他 Browser Actions 所有的方法 Page Actions 都有。另外的区别就是，Page Actions 并不像 Browser Actions 那样一直显示图标，而是可以在特定标签特定情况下显示或隐藏，所以它还具有独有的`show`和`hide`方法。

```js
chrome.pageAction.show(integer tabId);
chrome.pageAction.hide(integer tabId); 
```

`tabId`为标签 id，可以通过 tabs 接口获取，有关 tab 相关的内容将在后面进行讲解。

由于 Page Actions 和 Browser Actions 有大量相似之处，在此就不详细介绍了，还请读者参照前面 3.2 节的内容。