# 5.2 document 节点

*   document 节点概述
*   document 节点的属性
    *   doctype，documentElement，defaultView，body，head，activeElement
    *   documentURI，URL，domain，lastModified，location，referrer，title，characterSet
    *   readyState，designModed
    *   implementation，compatMode
    *   anchors，embeds，forms，images，links，scripts，styleSheets
    *   cookie
    *   document 对象的方法
        *   open()，close()，write()，writeln()
        *   hasFocus()
        *   querySelector()，getElementById()，querySelectorAll()，getElementsByTagName()，getElementsByClassName()，getElementsByName()，elementFromPoint()
        *   createElement()，createTextNode()，createAttribute()，createDocumentFragment()
        *   createEvent()
        *   createNodeIterator()，createTreeWalker()
        *   adoptNode()，importNode()
        *   addEventListener()，removeEventListener()，dispatchEvent()

## document 节点概述

document 节点是文档的根节点，每张网页都有自己的 document 节点。window.document 属性就指向这个节点。也就是说，只要浏览器开始载入 HTML 文档，这个节点对象就开始存在了，可以直接调用。

document 节点有不同的办法可以获取。

*   对于正常的网页，直接使用 document 或 window.document。
*   对于 iframe 载入的网页，使用 iframe 节点的 contentDocument 属性。
*   对 Ajax 操作返回的文档，使用 XMLHttpRequest 对象的 responseXML 属性。
*   对于某个节点包含的文档，使用该节点的 ownerDocument 属性。

上面这四种 document 节点，都部署了[Document 接口](http://dom.spec.whatwg.org/#interface-document)，因此有共同的属性和方法。当然，各自也有一些自己独特的属性和方法，比如 HTML 和 XML 文档的 document 节点就不一样。

## document 节点的属性

document 节点有很多属性，用得比较多的是下面这些。

### doctype，documentElement，defaultView，body，head，activeElement

以下属性指向文档内部的某个节点。

（1）doctype

对于 HTML 文档来说，document 对象一般有两个子节点。第一个子节点是 document.doctype，它是一个对象，包含了当前文档类型（Document Type Declaration，简写 DTD）信息。对于 HTML5 文档，该节点就代表。如果网页没有声明 DTD，该属性返回 null。

```js
var doctype = document.doctype;

doctype // "<!DOCTYPE html>"
doctype.name // "html"
```

document.firstChild 通常就返回这个节点。

（2）documentElement

document.documentElement 属性，表示当前文档的根节点（root）。它通常是 document 节点的第二个子节点，紧跟在`document.doctype`节点后面。

对于 HTML 网页，该属性返回 HTML 节点，代表。

（3）defaultView

defaultView 属性，在浏览器中返回 document 对象所在的 window 对象，否则返回 null。

```js
var win = document.defaultView;
```

（4）body

body 属性返回当前文档的 body 或 frameset 节点，如果不存在这样的节点，就返回 null。这个属性是可写的，如果对其写入一个新的节点，会导致原有的所有子节点被移除。

（4）head

head 属性返回当前文档的 head 节点。如果当前文档有多个 head，则返回第一个。

```js
document.head === document.querySelector("head") 
```

（5）activeElement

activeElement 属性返回当前文档中获得焦点的那个元素。用户通常可以使用 tab 键移动焦点，使用空格键激活焦点，比如如果焦点在一个链接上，此时按一下空格键，就会跳转到该链接。

### documentURI，URL，domain，lastModified，location，referrer，title，characterSet

以下属性返回文档信息。

（1）documentURI，URL

documentURI 属性和 URL 属性都返回当前文档的网址。不同之处是 documentURI 属性是所有文档都具备的，URL 属性则是 HTML 文档独有的。

（2）domain

domain 属性返回当前文档的域名。比如，某张网页的网址是 [`www.example.com/hello.html`](http://www.example.com/hello.html)，domain 属性就等于 [www.example.com](http://www.example.com/) 。如果无法获取域名，该属性返回 null。

```js
var badDomain = "www.example.xxx";

if (document.domain === badDomain)
  window.close();
```

上面代码判断，如果当前域名等于指定域名，则关闭窗口。

二级域名的情况下，domain 属性可以设置为对应的一级域名。比如，当前域名是 sub.example.com，则 domain 属性可以设置为 example.com。除此之外的写入，都是不可以的。

（3）lastModified

lastModified 属性返回当前文档最后修改的时间戳，格式为字符串。

```js
document.lastModified
// Tuesday, July 10, 2001 10:19:42
```

注意，lastModified 属性的值是字符串，所以不能用来直接比较，两个文档谁的日期更新，需要用 Date.parse 方法转成时间戳格式，才能进行比较。

```js
if (Date.parse(doc1.lastModified) > Date.parse(doc2.lastModified)) {
  // ...
}
```

（4）location

location 属性返回一个只读对象，提供了当前文档的 URL 信息。

```js
// 假定当前网址为 http://user:passwd@www.example.com:4097/path/a.html?x=111#part1

document.location.href // "http://user:passwd@www.example.com:4097/path/a.html?x=111#part1"
document.location.protocol // "http:"
document.location.host // "www.example.com:4097"
document.location.hostname // "www.example.com"
document.location.port // "4097"
document.location.pathname // "/path/a.html"
document.location.search // "?x=111"
document.location.hash // "#part1"
document.location.user // "user"
document.location.password // "passed"

// 跳转到另一个网址
document.location.assign('http://www.google.com')
// 优先从服务器重新加载
document.location.reload(true)
// 优先从本地缓存重新加载（默认值）
document.location.reload(false)
// 跳转到另一个网址，但当前文档不保留在 history 对象中，
// 即无法用后退按钮，回到当前文档
document.location.assign('http://www.google.com')
// 将 location 对象转为字符串，等价于 document.location.href
document.location.toString()
```

虽然 location 属性返回的对象是只读的，但是可以将 URL 赋值给这个属性，网页就会自动跳转到指定网址。

```js
document.location = 'http://www.example.com';
// 等价于
document.location.href = 'http://www.example.com';
```

document.location 属性与 window.location 属性等价，历史上，IE 曾经不允许对 document.location 赋值，为了保险起见，建议优先使用 window.location。如果只是单纯地获取当前网址，建议使用 document.URL。

（5）referrer

referrer 属性返回一个字符串，表示前文档的访问来源，如果是无法获取来源或是用户直接键入网址，而不是从其他网页点击，则返回一个空字符串。

（6）title

title 属性返回当前文档的标题，该属性是可写的。

```js
document.title = '新标题';
```

（7）characterSet

characterSet 属性返回渲染当前文档的字符集，比如 UTF-8、ISO-8859-1。

### readyState，designModed

以下属性与文档行为有关。

（1）readyState

readyState 属性返回当前文档的状态，共有三种可能的值，加载 HTML 代码阶段（尚未完成解析）是“loading”，加载外部资源阶段是“interactive”，全部加载完成是“complete”。

（2）designModed

designMode 属性控制当前 document 是否可编辑。通常会打开 iframe 的 designMode 属性，将其变为一个所见即所得的编辑器。

```js
iframe_node.contentDocument.designMode = "on";
```

### implementation，compatMode

以下属性返回文档的环境信息。

（1）implementation

implementation 属性返回一个对象，用来甄别当前环境部署了哪些 DOM 相关接口。implementation 属性的 hasFeature 方法，可以判断当前环境是否部署了特定版本的特定接口。

```js
document.implementation.hasFeature( 'HTML, 2.0 )
// true

document.implementation.hasFeature('MutationEvents','2.0')
// true
```

上面代码表示，当前环境部署了 DOM HTML 2.0 版和 MutationEvents 的 2.0 版。

（2）compatMode

compatMode 属性返回浏览器处理文档的模式，可能的值为 BackCompat（向后兼容模式）和 CSS1Compat（严格模式）。

### anchors，embeds，forms，images，links，scripts，styleSheets

以下属性返回文档内部特定元素的集合（即 HTMLCollection 对象，详见下文）。这些集合都是动态的，原节点有任何变化，立刻会反映在集合中。

（1）anchors

anchors 属性返回网页中所有的 a 节点元素。注意，只有指定了 name 属性的 a 元素，才会包含在 anchors 属性之中。

（2）embeds

embeds 属性返回网页中所有嵌入对象，即 embed 标签，返回的格式为类似数组的对象（nodeList）。

（3）forms

forms 属性返回页面中所有表单。

```js
var selectForm = document.forms[index];
var selectFormElement = document.forms[index].elements[index];
```

上面代码获取指定表单的指定元素。

（4）images

images 属性返回页面所有图片元素（即 img 标签）。

```js
var ilist = document.images;

for(var i = 0; i < ilist.length; i++) {
  if(ilist[i].src == "banner.gif") {
    // ...
  }
}
```

上面代码在所有 img 标签中，寻找特定图片。

（4）links

links 属性返回当前文档所有的链接元素（即 a 标签，或者说具有 href 属性的元素）。

（5）scripts

scripts 属性返回当前文档的所有脚本（即 script 标签）。

```js
var scripts = document.scripts;
if (scripts.length !== 0 ) {
  console.log("当前网页有脚本");
}
```

（6）styleSheets

styleSheets 属性返回一个类似数组的对象，包含了当前网页的所有样式表。该属性提供了样式表操作的接口。然后，每张样式表对象的 cssRules 属性，返回该样式表的所有 CSS 规则。这又方便了操作具体的 CSS 规则。

```js
var allSheets = [].slice.call(document.styleSheets);
```

上面代码中，使用 slice 方法将 document.styleSheets 转为数组，以便于进一步处理。

### cookie

（1）概述

cookie 属性返回当前网页的 cookie。

```js
// 读取当前网页的所有 cookie
allCookies = document.cookie;
```

该属性是可写的，但是一次只能写入一个 cookie，也就是说写入并不是覆盖，而是添加。另外，cookie 的值必须对分号、逗号和空格进行转义。

```js
// 写入一个新 cookie
document.cookie = "test1=hello";
// 再写入一个 cookie
document.cookie = "test2=world";

document.cookie
// test1=hello;test2=world
```

cookie 属性的读写操作含义不同，跟服务器与浏览器的通信格式有关。浏览器向服务器发送 cookie，是一次性所有 cookie 全部发送。

```js
GET /sample_page.html HTTP/1.1
Host: www.example.org
Cookie: cookie_name1=cookie_value1; cookie_name2=cookie_value2
Accept: */*
```

服务器告诉浏览器需要储存 cookie，则是分行指定。

```js
HTTP/1.0 200 OK
Content-type: text/html
Set-Cookie: cookie_name1=cookie_value1
Set-Cookie: cookie_name2=cookie_value2; expires=Sun, 16 Jul 3567 06:23:41 GMT
```

cookie 的值可以用 encodeURIComponent 方法进行处理，对逗号、分号、空格进行转义（这些符号都不允许作为 cookie 的值）。

（2）cookie 的属性

除了 cookie 本身的内容，还有一些可选的属性也是可以写入的，它们都必须以分号开头。

```js
Set-Cookie: value[; expires=date][; domain=domain][; path=path][; secure]
```

*   ; path=path，指定路径，必须是绝对路径（比如'/'，'/mydir'），如果未指定，默认为设定该 cookie 的路径。只有 path 属性匹配向服务器发送的路径，cookie 才会发送。这里的匹配不是绝对匹配，而是从根路径开始，只要 path 属性匹配发送路径的一部分，就可以发送。比如，path 属性等于`/blog`，则发送路径是`/blog`或者`/blogroll`，cookie 都会发送。path 属性生效的前提是 domain 属性匹配。

*   ; domain=domain，指定 cookie 所在的域名，比如'example.com'，'.example.com'（这种写法将对所有子域名生效）、'subdomain.example.com'。如果未指定，默认为设定该 cookie 的域名。所指定的域名必须是当前发送 cookie 的域名的一部分，比如当前访问的域名是 example.com，就不能将其设为 google.com。只有访问的域名匹配 domain 属性，cookie 才会发送到服务器。

*   ; max-age=max-age-in-seconds，指定 cookie 有效期，比如 60*60*24*365（即一年 31536e3 秒）。

*   ; expires=date-in-GMTString-format，指定 cookie 过期时间，日期格式等同于 Date.toUTCString()的格式。如果不设置该属性，则 cookie 只在当前会话（session）有效，浏览器窗口一旦关闭，当前 session 结束，该 cookie 就会被删除。浏览器根据本地时间，决定 cookie 是否过期，由于本地时间是不精确的，所以没有办法保证 cookie 一定会在服务器指定的时间过期。

*   ; secure，指定 cookie 只能在加密协议 HTTPS 下发送到服务器。该属性只是一个开关，不需要设定值。在 HTTPS 协议下设定的 cookie，该开关自动打开。

以上属性可以同时设置一个或多个，也没有次序的要求。如果服务器想改变一个早先设置的 cookie，必须同时满足四个条件：cookie 的名字、domain、path 和 secure。也就是说，如果原始的 cookie 是用如下的 Set-Cookie 头命令设置的。

```js
Set-Cookie: key1=value1; domain=example.com; path=/blog
```

改变上面这个 cookie 的值，就必须使用同样的 Set-Cookie 命令。

```js
Set-Cookie: key1=value2; domain=example.com; path=/blog
```

只要有一个属性不同，就会生成一个全新的 cookie，而不是替换掉原来那个 cookie。

```js
Set-Cookie: key1=value2; domain=example.com; path=/
```

上面的命令设置了一个全新的同名 cookie。下一次访问`example.com/blog`的时候，浏览器将向服务器发送两个同名的 cookie。

```js
Cookie: key1=value1; key1=value2
```

上面代码的两个 cookie 是同名的，匹配越精确的 cookie 排在越前面。

```js
var str = 'someCookieName=true';
str += '; expires=Fri, 31 Dec 9999 23:59:59 GMT';
str += '; path=/';

document.cookie = str;
```

另外，上面这些 cookie 属性只能用来设置 cookie。一旦设置完成，就没有办法从某个 cookie 读取这些属性的值。

删除一个 cookie 的简便方法，就是设置 expires 属性等于 0。

（3）cookie 的限制

浏览器对 cookie 的数量和长度有限制，但是每个浏览器的规定是不一样的。

*   IE6：每个域名 20 个 cookie。
*   IE7，IE8，Firefox：每个域名 50 个 cookie
*   Safari，Chrome：没有域名数量的限制。

所有 cookie 的累加长度限制为 4KB。超过这个长度的 cookie，将被忽略，不会被设置。

由于 cookie 存在数量限制，有时为了规避限制，可以将 cookie 设置成下面的形式。

```js
name=a=b&c=d&e=f&g=h
```

上面代码实际上是设置了一个 cookie，但是这个 cookie 内部使用&符号，设置了多部分的内容。因此，可以在一个 cookie 里面，通过自行解析，可以得到多个键值对。这样就规避了 cookie 的数量限制。

（4）HTTP-Only cookie

设置 cookie 的时候，如果服务器加上了 HTTPOnly 属性，则这个 cookie 无法被 JavaScript 读取（即`document.cookie`不会返回这个 cookie 的值），只用于向服务器发送。

```js
Set-Cookie: key=value; HttpOnly
```

上面的这个 cookie 将无法用 JavaScript 获取。进行 AJAX 操作时，getAllResponseHeaders 方法或 getResponseHeader 方法也不会显示这个头命令。

## document 对象的方法

document 对象主要有以下一些方法。

### open()，close()，write()，writeln()

document.open 方法用于新建一个文档，供 write 方法写入内容。它实际上等于清除当前文档，重新写入内容。不要将此方法与 window.open()混淆，后者用来打开一个新窗口，与当前文档无关。

document.close 方法用于关闭 open 方法所新建的文档。一旦关闭，write 方法就无法写入内容了。如果再调用 write 方法，就等同于又调用 open 方法，新建一个文档，再写入内容。

document.write 方法用于向当前文档写入内容。只要当前文档还没有用 close 方法关闭，它所写入的内容就会追加在已有内容的后面。

```js
// 页面显示“helloworld”
document.open();
document.write("hello");
document.write("world");
document.close();
```

如果页面已经渲染完成（DOMContentLoaded 事件发生之后），再调用 write 方法，它会先调用 open 方法，擦除当前文档所有内容，然后再写入。

```js
document.addEventListener("DOMContentLoaded", function(event) {
  document.write('<p>Hello World!</p>');
});

// 等同于

document.addEventListener("DOMContentLoaded", function(event) {
  document.open();
  document.write('<p>Hello World!</p>');
  document.close();
});
```

如果在页面渲染过程中调用 write 方法，并不会调用 open 方法。（可以理解成，open 方法已调用，但 close 方法还未调用。）

```js
<html>
<body>
hello
<script type="text/javascript">
  document.write("world")
</script>
</body>
</html>
```

在浏览器打开上面网页，将会显示“hello world”。

需要注意的是，虽然调用 close 方法之后，无法再用 write 方法写入内容，但这时当前页面的其他 DOM 节点还是会继续加载。

```js
<html>
<head>
<title>write example</title>
<script type="text/javascript">
  document.open();
  document.write("hello");
  document.close();
</script>
</head>
<body>
world
</body>
</html>
```

在浏览器打开上面网页，将会显示“hello world”。

总之，除了某些特殊情况，应该尽量避免使用 document.write 这个方法。

document.writeln 方法与 write 方法完全一致，除了会在输出内容的尾部添加换行符。

```js
document.write(1);
document.write(2);
// 12

document.writeln(1);
document.writeln(2);
// 1
// 2
//
```

注意，writeln 方法添加的是 ASCII 码的换行符，渲染成 HTML 网页时不起作用。

### hasFocus()

document.hasFocus 方法返回一个布尔值，表示当前文档之中是否有元素被激活或获得焦点。

```js
focused = document.hasFocus();
```

注意，有焦点的文档必定被激活（active），反之不成立，激活的文档未必有焦点。比如如果用户点击按钮，从当前窗口跳出一个新窗口，该新窗口就是激活的，但是不拥有焦点。

### querySelector()，getElementById()，querySelectorAll()，getElementsByTagName()，getElementsByClassName()，getElementsByName()，elementFromPoint()

以下方法用来选中当前文档中的元素。

（1）querySelector()

querySelector 方法返回匹配指定的 CSS 选择器的元素节点。如果有多个节点满足匹配条件，则返回第一个匹配的节点。如果没有发现匹配的节点，则返回 null。

```js
var el1 = document.querySelector(".myclass");
var el2 = document.querySelector('#myParent > [ng-click]');
```

querySelector 方法无法选中 CSS 伪元素。

（2）getElementById()

getElementById 方法返回匹配指定 ID 属性的元素节点。如果没有发现匹配的节点，则返回 null。

```js
var elem = document.getElementById("para1");
```

注意，在搜索匹配节点时，ID 属性是大小写敏感的。比如，如果某个节点的 ID 属性是 main，那么`document.getElementById("Main")`将返回 null，而不是指定节点。

getElementById 方法与 querySelector 方法都能获取元素节点，不同之处是 querySelector 方法的参数使用 CSS 选择器语法，getElementById 方法的参数是 HTML 标签元素的 id 属性。

```js
document.getElementById('myElement')
document.querySelector('#myElement')
```

上面代码中，两个方法都能选中 id 为 myElement 的元素，但是 getElementById()比 querySelector()效率高得多。

（3）querySelectorAll()

querySelectorAll 方法返回匹配指定的 CSS 选择器的所有节点，返回的是 NodeList 类型的对象。NodeList 对象不是动态集合，所以元素节点的变化无法实时反映在返回结果中。

```js
elementList = document.querySelectorAll(selectors);
```

querySelectorAll 方法的参数，可以是逗号分隔的多个 CSS 选择器，返回所有匹配其中一个选择器的元素。

```js
var matches = document.querySelectorAll("div.note, div.alert");
```

上面代码返回 class 属性是 note 或 alert 的 div 元素。

querySelectorAll 方法支持复杂的 CSS 选择器。

```js
// 选中 data-foo-bar 属性等于 someval 的元素
document.querySelectorAll('[data-foo-bar="someval"]');

// 选中 myForm 表单中所有不通过验证的元素
document.querySelectorAll('#myForm :invalid');

// 选中 div 元素，那些 class 含 ignore 的除外
document.querySelectorAll('DIV:not(.ignore)');

// 同时选中 div，a，script 三类元素
document.querySelectorAll('DIV, A, SCRIPT');
```

如果 querySelectorAll 方法和 getElementsByTagName 方法的参数是字符串“*”，则会返回文档中的所有 HTML 元素节点。

与 querySelector 方法一样，querySelectorAll 方法无法选中 CSS 伪元素。

（4）getElementsByClassName()

getElementsByClassName 方法返回一个类似数组的对象（HTMLCollection 类型的对象），包括了所有 class 名字符合指定条件的元素（搜索范围包括本身），元素的变化实时反映在返回结果中。这个方法不仅可以在 document 对象上调用，也可以在任何元素节点上调用。

```js
// document 对象上调用
var elements = document.getElementsByClassName(names);
// 非 document 对象上调用
var elements = rootElement.getElementsByClassName(names);
```

getElementsByClassName 方法的参数，可以是多个空格分隔的 class 名字，返回同时具有这些节点的元素。

```js
document.getElementsByClassName('red test');
```

上面代码返回 class 同时具有 red 和 test 的元素。

（5）getElementsByTagName()

getElementsByTagName 方法返回所有指定标签的元素（搜索范围包括本身）。返回值是一个 HTMLCollection 对象，也就是说，搜索结果是一个动态集合，任何元素的变化都会实时反映在返回的集合中。这个方法不仅可以在 document 对象上调用，也可以在任何元素节点上调用。

```js
var paras = document.getElementsByTagName("p");
```

上面代码返回当前文档的所有 p 元素节点。

注意，getElementsByTagName 方法会将参数转为小写后，再进行搜索。

（6）getElementsByName()

getElementsByName 方法用于选择拥有 name 属性的 HTML 元素，比如 form、img、frame、embed 和 object，返回一个 NodeList 格式的对象，不会实时反映元素的变化。

```js
// 假定有一个表单是<form name="x"></form>
var forms = document.getElementsByName("x");
forms[0].tagName // "FORM"
```

注意，在 IE 浏览器使用这个方法，会将没有 name 属性、但有同名 id 属性的元素也返回，所以 name 和 id 属性最好设为不一样的值。

（7）elementFromPoint()

elementFromPoint 方法返回位于页面指定位置的元素。

```js
var element = document.elementFromPoint(x, y);
```

上面代码中，elementFromPoint 方法的参数 x 和 y，分别是相对于当前窗口左上角的横坐标和纵坐标，单位是 CSS 像素。elementFromPoint 方法返回位于这个位置的 DOM 元素，如果该元素不可返回（比如文本框的滚动条），则返回它的父元素（比如文本框）。如果坐标值无意义（比如负值），则返回 null。

### createElement()，createTextNode()，createAttribute()，createDocumentFragment()

以下方法用于生成元素节点。

（1）createElement()

createElement 方法用来生成 HTML 元素节点。

```js
var element = document.createElement(tagName);
// 实例
var newDiv = document.createElement("div");
```

createElement 方法的参数为元素的标签名，即元素节点的 tagName 属性。如果传入大写的标签名，会被转为小写。如果参数带有尖括号（即）或者是 null，会报错。

（2）createTextNode()

createTextNode 方法用来生成文本节点，参数为所要生成的文本节点的内容。

```js
var newDiv = document.createElement("div");
var newContent = document.createTextNode("Hello");
newDiv.appendChild(newContent);
```

上面代码新建一个 div 节点和一个文本节点，然后将文本节点插入 div 节点。

（3）createAttribute()

createAttribute 方法生成一个新的属性对象节点，并返回它。

```js
attribute = document.createAttribute(name);
```

createAttribute 方法的参数 name，是属性的名称。

```js
var node = document.getElementById("div1");
var a = document.createAttribute("my_attrib");
a.value = "newVal";
node.setAttributeNode(a);

// 等同于

var node = document.getElementById("div1");
node.setAttribute("my_attrib", "newVal");
```

（4）createDocumentFragment()

createDocumentFragment 方法生成一个 DocumentFragment 对象。

```js
var docFragment = document.createDocumentFragment();
```

DocumentFragment 对象是一个存在于内存的 DOM 片段，但是不属于当前文档，常常用来生成较复杂的 DOM 结构，然后插入当前文档。这样做的好处在于，因为 DocumentFragment 不属于当前文档，对它的任何改动，都不会引发网页的重新渲染，比直接修改当前文档的 DOM 有更好的性能表现。

```js
var docfrag = document.createDocumentFragment();

[1, 2, 3, 4].forEach(function(e) {
  var li = document.createElement("li");
  li.textContent = e;
  docfrag.appendChild(li);
});

document.body.appendChild(docfrag);
```

### createEvent()

createEvent 方法生成一个事件对象，该对象可以被 element.dispatchEvent 方法使用，触发指定事件。

```js
var event = document.createEvent(type);
```

createEvent 方法的参数是事件类型，比如 UIEvents、MouseEvents、MutationEvents、HTMLEvents。

```js
var event = document.createEvent('Event');
event.initEvent('build', true, true);
document.addEventListener('build', function (e) {
  // ...
}, false);
document.dispatchEvent(event);
```

### createNodeIterator()，createTreeWalker()

以下方法用于遍历元素节点。

（1）createNodeIterator()

createNodeIterator 方法返回一个 DOM 的子节点遍历器。

```js
var nodeIterator = document.createNodeIterator(
  document.body,
  NodeFilter.SHOW_ELEMENT
);
```

上面代码返回 body 元素的遍历器。createNodeIterator 方法的第一个参数为遍历器的根节点，第二个参数为所要遍历的节点类型，这里指定为元素节点。其他类型还有所有节点（NodeFilter.SHOW_ALL）、文本节点（NodeFilter.SHOW_TEXT）、评论节点（NodeFilter.SHOW_COMMENT）等。

所谓“遍历器”，在这里指可以用 nextNode 方法和 previousNode 方法依次遍历根节点的所有子节点。

```js
var nodeIterator = document.createNodeIterator(document.body);
var pars = [];
var currentNode;

while (currentNode = nodeIterator.nextNode()) {
  pars.push(currentNode);
}
```

上面代码使用遍历器的 nextNode 方法，将根节点的所有子节点，按照从头部到尾部的顺序，读入一个数组。nextNode 方法先返回遍历器的内部指针所在的节点，然后会将指针移向下一个节点。所有成员遍历完成后，返回 null。previousNode 方法则是先将指针移向上一个节点，然后返回该节点。

```js
var nodeIterator = document.createNodeIterator(
  document.body,
  NodeFilter.SHOW_ELEMENT
);

var currentNode = nodeIterator.nextNode();
var previousNode = nodeIterator.previousNode();

currentNode === previousNode // true
```

上面代码中，currentNode 和 previousNode 都指向同一个的节点。

有一个需要注意的地方，遍历器返回的第一个节点，总是根节点。

（2）createTreeWalker()

createTreeWalker 方法返回一个 DOM 的子树遍历器。它与 createNodeIterator 方法的区别在于，后者只遍历子节点，而它遍历整个子树。

createTreeWalker 方法的第一个参数，是所要遍历的根节点，第二个参数指定所要遍历的节点类型。

```js
var treeWalker = document.createTreeWalker(
  document.body,
  NodeFilter.SHOW_ELEMENT
);

var nodeList = [];

while(treeWalker.nextNode()) nodeList.push(treeWalker.currentNode);
```

上面代码遍历 body 节点下属的所有元素节点，将它们插入 nodeList 数组。

### adoptNode()，importNode()

以下方法用于获取外部文档的节点。

（1）adoptNode()

adoptNode 方法将某个节点，从其原来所在的文档移除，插入当前文档，并返回插入后的新节点。

```js
node = document.adoptNode(externalNode);
```

importNode 方法从外部文档拷贝指定节点，插入当前文档。

```js
var node = document.importNode(externalNode, deep);
```

（2）importNode()

importNode 方法用于创造一个外部节点的拷贝，然后插入当前文档。它的第一个参数是外部节点，第二个参数是一个布尔值，表示对外部节点是深拷贝还是浅拷贝，默认是浅拷贝（false）。虽然第二个参数是可选的，但是建议总是保留这个参数，并设为 true。

另外一个需要注意的地方是，importNode 方法只是拷贝外部节点，这时该节点的父节点是 null。下一步还必须将这个节点插入当前文档的 DOM 树。

```js
var iframe = document.getElementsByTagName("iframe")[0];
var oldNode = iframe.contentWindow.document.getElementById("myNode");
var newNode = document.importNode(oldNode, true);
document.getElementById("container").appendChild(newNode);
```

上面代码从 iframe 窗口，拷贝一个指定节点 myNode，插入当前文档。

### addEventListener()，removeEventListener()，dispatchEvent()

以下三个方法与 Document 节点的事件相关。这些方法都继承自 EventTarget 接口，详细介绍参见《Event 对象》章节的《EventTarget》部分。

```js
// 添加事件监听函数
document.addEventListener('click', listener, false);

// 移除事件监听函数
document.removeEventListener('click', listener, false);

// 触发事件
var event = new Event('click');
document.dispatchEvent(event);
```