# 5.1 Node 节点

*   DOM 的概念
*   节点的概念
*   Node 节点的属性
    *   nodeName，nodeType
    *   ownerDocument，nextSibling，previousSibling，parentNode，parentElement
    *   textContent，nodeValue
    *   childNodes，firstNode，lastChild
    *   baseURI
    *   Node 节点的方法
        *   appendChild()，hasChildNodes()
        *   cloneNode()，insertBefore()，removeChild()，replaceChild()
        *   contains()，compareDocumentPosition()，isEqualNode()
        *   normalize()
    *   NodeList 接口，HTMLCollection 接口
        *   NodeList 接口
        *   HTMLCollection 接口
    *   ParentNode 接口，ChildNode 接口
        *   ParentNode 接口
        *   ChildNode 接口
    *   html 元素
        *   dataset 属性
        *   tabindex 属性
        *   页面位置相关属性
        *   style 属性
        *   Element 对象的方法
        *   table 元素
    *   参考链接

## DOM 的概念

DOM 是文档对象模型（Document Object Model）的简称，它的基本思想是把结构化文档（比如 HTML 和 XML）解析成一系列的节点，再由这些节点组成一个树状结构（DOM Tree）。所有的节点和最终的树状结构，都有规范的对外接口，以达到使用编程语言操作文档的目的（比如增删内容）。所以，DOM 可以理解成文档（HTML 文档、XML 文档和 SVG 文档）的编程接口。

DOM 有自己的国际标准，目前的通用版本是[DOM 3](http://www.w3.org/TR/2004/REC-DOM-Level-3-Core-20040407/core.html)，下一代版本[DOM 4](http://www.w3.org/TR/dom/)正在拟定中。本章介绍的就是 JavaScript 对 DOM 标准的实现和用法。

严格地说，DOM 不属于 JavaScript，但是操作 DOM 是 JavaScript 最常见的任务，而 JavaScript 也是最常用于 DOM 操作的语言。所以，DOM 往往放在 JavaScript 里面介绍。

## 节点的概念

DOM 的最小组成单位叫做节点（node），一个文档的树形结构（DOM 树），就是由各种不同类型的节点组成。

对于 HTML 文档，节点主要有以下六种类型：Document 节点、DocumentType 节点、Element 节点、Attribute 节点、Text 节点和 DocumentFragment 节点。

| 节点 | 名称 | 含义 |
| --- | --- | --- |
| Document | 文档节点 | 整个文档（window.document） |
| DocumentType | 文档类型节点 | 文档的类型（比如） |
| Element | 元素节点 | HTML 元素（比如、等） |
| Attribute | 属性节点 | HTML 元素的属性（比如 class="right"） |
| Text | 文本节点 | HTML 文档中出现的文本 |
| DocumentFragment | 文档碎片节点 | 文档的片段 |

浏览器原生提供一个 Node 对象，上表所有类型的节点都是 Node 对象派生出来的。也就是说，它们都继承了 Node 的属性和方法。

## Node 节点的属性

### nodeName，nodeType

nodeName 属性返回节点的名称，nodeType 属性返回节点的常数值。具体的返回值，可查阅下方的表格。

| 类型 | nodeName | nodeType |
| --- | --- | --- |
| DOCUMENT_NODE | #document | 9 |
| ELEMENT_NODE | 大写的 HTML 元素名 | 1 |
| ATTRIBUTE_NODE | 等同于 Attr.name | 2 |
| TEXT_NODE | #text | 3 |
| DOCUMENT_FRAGMENT_NODE | #document-fragment | 11 |
| DOCUMENT_TYPE_NODE | 等同于 DocumentType.name | 10 |

以 document 节点为例，它的 nodeName 属性等于#document，nodeType 属性等于 9。

```js
document.nodeName // "#document"
document.nodeType // 9
```

通常来说，使用 nodeType 属性确定一个节点的类型，比较方便。

```js
document.querySelector('a').nodeType === 1
// true

document.querySelector('a').nodeType === Node.ELEMENT_NODE
// true
```

上面两种写法是等价的。

### ownerDocument，nextSibling，previousSibling，parentNode，parentElement

以下属性返回当前节点的相关节点。

（1）ownerDocument

ownerDocument 属性返回当前节点所在的顶层文档对象，即 document 对象。

```js
var d = p.ownerDocument;
d === document // true
```

document 对象本身的 ownerDocument 属性，返回 null。

（2）nextSibling

nextsibling 属性返回紧跟在当前节点后面的第一个同级节点。如果当前节点后面没有同级节点，则返回 null。注意，该属性还包括文本节点和评论节点。因此如果当前节点后面有空格，该属性会返回一个文本节点，内容为空格。

```js
var el = document.getelementbyid('div-01').firstchild;
var i = 1;

while (el) {
  console.log(i + '. ' + el.nodename);
  el = el.nextsibling;
  i++;
}
```

上面代码遍历 div-01 节点的所有子节点。

（3）previousSibling

previoussibling 属性返回当前节点前面的、距离最近的一个同级节点。如果当前节点前面没有同级节点，则返回 null。

```js
// html 代码如下
// <a><b1 id="b1"/><b2 id="b2"/></a>

document.getelementbyid("b1").previoussibling // null
document.getelementbyid("b2").previoussibling.id // "b1"
```

对于当前节点前面有空格，则 previoussibling 属性会返回一个内容为空格的文本节点。

（4）parentNode

parentNode 属性返回当前节点的父节点。对于一个节点来说，它的父节点只可能是三种类型：element 节点、document 节点和 documentfragment 节点。

下面代码是如何从父节点移除指定节点。

```js
if (node.parentNode) {
  node.parentNode.removeChild(node);
}
```

对于 document 节点和 documentfragment 节点，它们的父节点都是 null。另外，对于那些生成后还没插入 DOM 树的节点，父节点也是 null。

（5）parentElement

parentElement 属性返回当前节点的父 Element 节点。如果当前节点没有父节点，或者父节点类型不是 Element 节点，则返回 null。

```js
if (node.parentElement) {
  node.parentElement.style.color = "red";
}
```

上面代码设置指定节点的父 Element 节点的 CSS 属性。

在 IE 浏览器中，只有 Element 节点才有该属性，其他浏览器则是所有类型的节点都有该属性。

### textContent，nodeValue

以下属性返回当前节点的内容。

（1）textContent

textContent 属性返回当前节点和它的所有后代节点的文本内容。

```js
// HTML 代码为
// <div id="divA">This is <span>some</span> text</div>

document.getElementById("divA").textContent
// This is some text
```

上面代码的 textContent 属性，自动忽略当前节点内部的 HTML 标签，返回所有文本内容。

该属性是可读写的，设置该属性的值，会用一个新的文本节点，替换所有它原来的子节点。它还有一个好处，就是自动对 HTML 标签转义。这很适合用于用户提供的内容。

```js
document.getElementById('foo').textContent = '<p>GoodBye!</p>';
```

上面代码在插入文本时，会将 p 标签解释为文本，即<p>，而不会当作标签处理。

对于 Text 节点和 Comment 节点，该属性的值与 nodeValue 属性相同。对于其他类型的节点，该属性会将每个子节点的内容连接在一起返回，但是不包括 Comment 节点。如果一个节点没有子节点，则返回空字符串。

document 节点和 doctype 节点的 textContent 属性为 null。如果要读取整个文档的内容，可以使用`document.documentElement.textContent`。

在 IE 浏览器，所有 Element 节点都有一个 innerText 属性。它与 textContent 属性基本相同，但是有几点区别。

*   innerText 受 CSS 影响，textcontent 不受。比如，如果 CSS 规则隐藏（hidden）了某段文本，innerText 就不会返回这段文本，textcontent 则照样返回。

*   innerText 返回的文本，会过滤掉空格、换行和回车键，textcontent 则不会。

*   innerText 属性不是 DOM 标准的一部分，Firefox 浏览器甚至没有部署这个属性，而 textcontent 是 DOM 标准的一部分。

（2）nodeValue

nodeValue 属性返回或设置当前节点的值，格式为字符串。但是，该属性只对 Text 节点、Comment 节点、XML 文档的 CDATA 节点有效，其他类型的节点一律返回 null。

因此，nodeValue 属性一般只用于 Text 节点。对于那些返回 null 的节点，设置 nodeValue 属性是无效的。

### childNodes，firstNode，lastChild

以下属性返回当前节点的子节点。

（1）childNodes

childNodes 属性返回一个 NodeList 集合，成员包括当前节点的所有子节点。注意，除了 HTML 元素节点，该属性返回的还包括 Text 节点和 Comment 节点。如果当前节点不包括任何子节点，则返回一个空的 NodeList 集合。由于 NodeList 对象是一个动态集合，一旦子节点发生变化，立刻会反映在返回结果之中。

```js
var ulElementChildNodes = document.querySelector('ul').childNodes;
```

（2）firstNode

firstNode 属性返回当前节点的第一个子节点，如果当前节点没有子节点，则返回 null。注意，除了 HTML 元素子节点，该属性还包括文本节点和评论节点。

（3）lastChild

lastChild 属性返回当前节点的最后一个子节点，如果当前节点没有子节点，则返回 null。

### baseURI

baseURI 属性返回一个字符串，由当前网页的协议、域名和所在的目录组成，表示当前网页的绝对路径。如果无法取到这个值，则返回 null。浏览器根据这个属性，计算网页上的相对路径的 URL。该属性为只读。

通常情况下，该属性由当前网址的 URL（即 window.location 属性）决定，但是可以使用 HTML 的标签，改变该属性的值。

```js
<base href="http://www.example.com/page.html">
<base target="_blank" href="http://www.example.com/page.html">
```

该属性不仅 document 对象有（`document.baseURI`），元素节点也有（`element.baseURI`）。通常情况下，它们的值是相同的。

## Node 节点的方法

### appendChild()，hasChildNodes()

以下方法与子节点相关。

（1）appendChild()

appendChild 方法接受一个节点对象作为参数，将其作为最后一个子节点，插入当前节点。

```js
var p = document.createElement("p");
document.body.appendChild(p);
```

如果参数节点是文档中现有的其他节点，appendChild 方法会将其从原来的位置，移动到新位置。

hasChildNodes 方法返回一个布尔值，表示当前节点是否有子节点。

```js
var foo = document.getElementById("foo");

if ( foo.hasChildNodes() ) {
  foo.removeChild( foo.childNodes[0] );
}
```

上面代码表示，如果 foo 节点有子节点，就移除第一个子节点。

（2）hasChildNodes()

hasChildNodes 方法结合 firstChild 属性和 nextSibling 属性，可以遍历当前节点的所有后代节点。

```js
function DOMComb (oParent, oCallback) {
  if (oParent.hasChildNodes()) {
    for (var oNode = oParent.firstChild; oNode; oNode = oNode.nextSibling) {
      DOMComb(oNode, oCallback);
    }
  }
  oCallback.call(oParent);
}
```

上面代码的 DOMComb 函数的第一个参数是某个指定的节点，第二个参数是回调函数。这个回调函数会依次作用于指定节点，以及指定节点的所有后代节点。

```js
function printContent () {
  if (this.nodeValue) {
    console.log(this.nodeValue);
  }
}

DOMComb(document.body, printContent);
```

### cloneNode()，insertBefore()，removeChild()，replaceChild()

下面方法与节点操作有关。

（1）cloneNode()

cloneNode 方法用于克隆一个节点。它接受一个布尔值作为参数，表示是否同时克隆子节点，默认是 false，即不克隆子节点。

```js
var cloneUL = document.querySelector('ul').cloneNode(true);
```

需要注意的是，克隆一个节点，会拷贝该节点的所有属性，但是会丧失 addEventListener 方法和 on-属性（即`node.onclick = fn`），添加在这个节点上的事件回调函数。

克隆一个节点之后，DOM 树有可能出现两个有相同 ID 属性（即`id="xxx"`）的 HTML 元素，这时应该修改其中一个 HTML 元素的 ID 属性。

（2）insertBefore()

insertBefore 方法用于将某个节点插入当前节点的指定位置。它接受两个参数，第一个参数是所要插入的节点，第二个参数是当前节点的一个子节点，新的节点将插在这个节点的前面。该方法返回被插入的新节点。

```js
var text1 = document.createTextNode('1');
var li = document.createElement('li');
li.appendChild(text1);

var ul = document.querySelector('ul');
ul.insertBefore(li,ul.firstChild);
```

上面代码在 ul 节点的最前面，插入一个新建的 li 节点。

如果 insertBefore 方法的第二个参数为 null，则新节点将插在当前节点的最后位置，即变成最后一个子节点。

将新节点插在当前节点的最前面（即变成第一个子节点），可以使用当前节点的 firstChild 属性。

```js
parentElement.insertBefore(newElement, parentElement.firstChild);
```

上面代码中，如果当前节点没有任何子节点，`parentElement.firstChild`会返回 null，则新节点会插在当前节点的最后，等于是第一个子节点。

由于不存在 insertAfter 方法，如果要插在当前节点的某个子节点后面，可以用 insertBefore 方法结合 nextSibling 属性模拟。

```js
parentDiv.insertBefore(s1, s2.nextSibling);
```

上面代码可以将 s1 节点，插在 s2 节点的后面。如果 s2 是当前节点的最后一个子节点，则`s2.nextSibling`返回 null，这时 s1 节点会插在当前节点的最后，变成当前节点的最后一个子节点，等于紧跟在 s2 的后面。

（3）removeChild()

removeChild 方法接受一个子节点作为参数，用于从当前节点移除该节点。它返回被移除的节点。

```js
var divA = document.getElementById('A');
divA.parentNode.removeChild(divA);
```

上面代码是如何移除一个指定节点。

下面是如何移除当前节点的所有子节点。

```js
var element = document.getElementById("top");
while (element.firstChild) {
  element.removeChild(element.firstChild);
}
```

被移除的节点依然存在于内存之中，但是不再是 DOM 的一部分。所以，一个节点移除以后，依然可以使用它，比如插入到另一个节点。

（4）replaceChild()

replaceChild 方法用于将一个新的节点，替换当前节点的某一个子节点。它接受两个参数，第一个参数是用来替换的新节点，第二个参数将要被替换走的子节点。它返回被替换走的那个节点。

```js
replacedNode = parentNode.replaceChild(newChild, oldChild);
```

下面是一个例子。

```js
var divA = document.getElementById('A');
var newSpan = document.createElement('span');
newSpan.textContent = 'Hello World!';
divA.parentNode.replaceChild(newSpan,divA);
```

上面代码是如何替换指定节点。

### contains()，compareDocumentPosition()，isEqualNode()

下面方法用于节点的互相比较。

（1）contains()

contains 方法接受一个节点作为参数，返回一个布尔值，表示参数节点是否为当前节点的后代节点。

```js
document.body.contains(node)
```

上面代码检查某个节点，是否包含在当前文档之中。

注意，如果将当前节点传入 contains 方法，会返回 true。虽然从意义上说，一个节点不应该包含自身。

```js
nodeA.contains(nodeA) // true
```

（2）compareDocumentPosition()

compareDocumentPosition 方法的用法，与 contains 方法完全一致，返回一个 7 个比特位的二进制值，表示参数节点与当前节点的关系。

| 二进制值 | 数值 | 含义 |
| --- | --- | --- |
| 000000 | 0 | 两个节点相同 |
| 000001 | 1 | 两个节点不在同一个文档（即有一个节点不在当前文档） |
| 000010 | 2 | 参数节点在当前节点的前面 |
| 000100 | 4 | 参数节点在当前节点的后面 |
| 001000 | 8 | 参数节点包含当前节点 |
| 010000 | 16 | 当前节点包含参数节点 |
| 100000 | 32 | 浏览器的私有用途 |

```js
// HTML 代码为
// <div id="writeroot">
//   <form>
//     <input id="test" />
//   </form>
// </div>

var x = document.getElementById('writeroot');
var y = document.getElementById('test');

x.compareDocumentPosition(y) // 20
y.compareDocumentPosition(x) // 10
```

上面代码中，节点 x 包含节点 y，而且节点 y 在节点 x 的后面，所以第一个 compareDocumentPosition 方法返回 20（010100），第二个 compareDocumentPosition 方法返回 10（0010010）。

由于 compareDocumentPosition 返回值的含义，定义在每一个比特位上，所以如果要检查某一种特定的含义，就需要使用比特位运算符。

```js
var head = document.head;
var body = document.body;
if (head.compareDocumentPosition(body) & 4) {
  console.log("文档结构正确");
} else {
  console.log("<head> 不能在 <body> 前面");
}
```

上面代码中，compareDocumentPosition 的返回值与 4（又称掩码）进行与运算（&），得到一个布尔值，表示 head 是否在 body 前面。

在这个方法的基础上，可以部署一些特定的函数，检查节点的位置。

```js
Node.prototype.before = function (arg) {
  return !!(this.compareDocumentPosition(arg) & 2)
}

nodeA.before(nodeB)
```

上面代码在 Node 对象上部署了一个 before 方法，返回一个布尔值，表示参数节点是否在当前节点的前面。

（3）isEqualNode()

isEqualNode 方法返回一个布尔值，用于检查两个节点是否相等。所谓相等的节点，指的是两个节点的类型相同、属性相同、子节点相同。

```js
var targetEl = document.getElementById("targetEl");
var firstDiv = document.getElementsByTagName("div")[0];

targetEl.isEqualNode(firstDiv)
```

### normalize()

normailize 方法用于清理当前节点内部的所有 Text 节点。它会去除空的文本节点，并且将毗邻的文本节点合并成一个。

```js
var wrapper = document.createElement("div");

wrapper.appendChild(document.createTextNode("Part 1 "));
wrapper.appendChild(document.createTextNode("Part 2 "));

wrapper.childNodes.length // 2

wrapper.normalize();

wrapper.childNodes.length // 1
```

上面代码使用 normalize 方法之前，wrapper 节点有两个 Text 子节点。使用 normalize 方法之后，两个 Text 子节点被合并成一个。

该方法是`Text.splitText`的逆方法，可以查看《Text 节点》章节，了解更多内容。

## NodeList 接口，HTMLCollection 接口

节点对象都是单个节点，但是有时会需要一种数据结构，能够容纳多个节点。DOM 提供两种接口，用于部署这种节点的集合：NodeList 接口和 HTMLCollection 接口。

### NodeList 接口

有些属性和方法返回的是一组节点，比如 Node.childNodes、document.querySelectorAll()。它们返回的都是一个部署了 NodeList 接口的对象。

NodeList 接口有时返回一个动态集合，有时返回一个静态集合。所谓动态集合就是一个活的集合，DOM 树删除或新增一个相关节点，都会立刻反映在 NodeList 接口之中。Node.childNodes 返回的，就是一个动态集合。

```js
var parent = document.getElementById('parent');
parent.childNodes.length // 2
parent.appendChild(document.createElement('div'));
parent.childNodes.length // 3
```

上面代码中，`parent.childNodes`返回的是一个部署了 NodeList 接口的对象。当 parent 节点新增一个子节点以后，该对象的成员个数就增加了 1。

document.querySelectorAll 方法返回的是一个静态，DOM 内部的变化，并不会实时反映在该方法的返回结果之中。

NodeList 接口提供 length 属性和数字索引，因此可以像数组那样，使用数字索引取出每个节点，但是它本身并不是数组，不能使用 pop 或 push 之类数组特有的方法。

```js
// 数组的继承链
myArray --> Array.prototype --> Object.prototype --> null

// NodeList 的继承链
myNodeList --> NodeList.prototype --> Object.prototype --> null
```

从上面的继承链可以看到，NodeList 接口对象并不继承 Array.prototype，因此不具有数组接口提供的方法。如果要在 NodeList 接口使用数组方法，可以将 NodeList 接口对象转为真正的数组。

```js
var div_list = document.querySelectorAll('div');
var div_array = Array.prototype.slice.call(div_list);
```

也可以通过下面的方法调用。

```js
var forEach = Array.prototype.forEach;

forEach.call(element.childNodes, function(child){
  child.parentNode.style.color = '#0F0';
});
```

上面代码让数组的 forEach 方法在 NodeList 接口对象上调用。

不过，遍历 NodeList 接口对象的首选方法，还是使用 for 循环。

```js
for (var i = 0; i < myNodeList.length; ++i) {
  var item = myNodeList[i];
}
```

不要使用 for...in 循环去遍历 NodeList 接口对象，因为 for...in 循环会将非数字索引的 length 属性和下面要讲到的 item 方法，也遍历进去，而且不保证各个成员遍历的顺序。

ES6 新增的 for...of 循环，也可以正确遍历 NodeList 接口对象。

```js
var list = document.querySelectorAll( 'input[type=checkbox]' );
for (var item of list) {
  item.checked = true;
}
```

NodeList 接口提供 item 方法，接受一个数字索引作为参数，返回该索引对应的成员。如果取不到成员，或者索引不合法，则返回 null。

```js
nodeItem = nodeList.item(index)

// 实例
var divs = document.getElementsByTagName("div");
var secondDiv = divs.item(1);
```

上面代码中，由于数字索引从零开始计数，所以取出第二个成员，要使用数字索引 1。

所有类似数组的对象，都可以使用方括号运算符取出成员，所以一般情况下，都是使用下面的写法，而不使用 item 方法。

```js
nodeItem = nodeList[index]
```

### HTMLCollection 接口

HTMLCollection 接口与 NodeList 接口类似，也是节点的集合，但是集合成员都是 Element 节点。该接口都是动态集合，节点的变化会实时反映在集合中。document.links、docuement.forms、document.images 等属性，返回的都是 HTMLCollection 接口对象。

部署了该接口的对象，具有 length 属性和数字索引，因此是一个类似于数组的对象。

item 方法根据成员的位置参数（从 0 开始），返回该成员。如果取不到成员或数字索引不合法，则返回 null。

```js
var c = document.images;
var img1 = c.item(10);

// 等价于下面的写法
var img1 = c[1];
```

namedItem 方法根据成员的 ID 属性或 name 属性，返回该成员。如果没有对应的成员，则返回 null。

```js
// HTML 代码为
// <form id="myForm"></form>
var elem = document.forms.namedItem("myForm");
// 等价于下面的写法
var elem = document.forms["myForm"];
```

由于 item 方法和 namedItem 方法，都可以用方括号运算符代替，所以建议一律使用方括号运算符。

## ParentNode 接口，ChildNode 接口

不同的节点除了继承 Node 接口以外，还会继承其他接口。ParentNode 接口用于获取当前节点的 Element 子节点，ChildNode 接口用于处理当前节点的子节点（包含但不限于 Element 子节点）。

### ParentNode 接口

ParentNode 接口用于获取 Element 子节点。Element 节点、Document 节点和 DocumentFragment 节点，部署了 ParentNode 接口。凡是这三类节点，都具有以下四个属性，用于获取 Element 子节点。

（1）children

children 属性返回一个动态的 HTMLCollection 集合，由当前节点的所有 Element 子节点组成。

下面代码遍历指定节点的所有 Element 子节点。

```js
if (el.children.length) {
  for (var i = 0; i < el.children.length; i++) {
    // ...
  }
}
```

（2）firstElementChild

firstChild 属性返回当前节点的第一个 Element 子节点，如果不存在任何 Element 子节点，则返回 null。

```js
document.firstElementChild.nodeName
// "HTML"
```

上面代码中，document 节点的第一个 Element 子节点是。

（3）lastElementChild

lastElementChild 属性返回当前节点的最后一个 Element 子节点，如果不存在任何 Element 子节点，则返回 null。

```js
document.lastElementChild.nodeName
// "HTML"
```

上面代码中，document 节点的最后一个 Element 子节点是。

（4）childElementCount

childElementCount 属性返回当前节点的所有 Element 子节点的数目。

### ChildNode 接口

ChildNode 接口用于处理子节点（包含但不限于 Element 子节点）。Element 节点、DocumentType 节点和 CharacterData 接口，部署了 ChildNode 接口。凡是这三类节点（接口），都可以使用下面四个方法。但是现实的情况是，除了第一个 remove 方法，目前没有浏览器支持后面三个方法。

（1）remove()

remove 方法用于移除当前节点。

```js
el.remove()
```

上面方法在 DOM 中移除了 el 节点。注意，调用这个方法的节点，是被移除的节点本身，而不是它的父节点。

（2）before()

before 方法用于在当前节点的前面，插入一个同级节点。如果参数是节点对象，插入 DOM 的就是该节点对象；如果参数是文本，插入 DOM 的就是参数对应的文本节点。

（3）after()

after 方法用于在当前节点的后面，插入一个同级节点。如果参数是节点对象，插入 DOM 的就是该节点对象；如果参数是文本，插入 DOM 的就是参数对应的文本节点。

（4）replaceWith()

replaceWith 方法使用参数指定的节点，替换当前节点。如果参数是节点对象，替换当前节点的就是该节点对象；如果参数是文本，替换当前节点的就是参数对应的文本节点。

## html 元素

html 元素是网页的根元素，document.documentElement 就指向这个元素。

（1）clientWidth 属性，clientHeight 属性

这两个属性返回视口（viewport）的大小，单位为像素。所谓“视口”，是指用户当前能够看见的那部分网页的大小

document.documentElement.clientWidth 和 document.documentElement.clientHeight，基本上与 window.innerWidth 和 window.innerHeight 同义。只有一个区别，前者不将滚动条计算在内（很显然，滚动条和工具栏会减小视口大小），而后者包括了滚动条的高度和宽度。

（2）offsetWidth 属性，offsetHeight 属性

这两个属性返回 html 元素的宽度和高度，即网页的总宽度和总高度。

### dataset 属性

dataset 属性用于操作 HTML 标签元素的 data-*属性。目前，Firefox、Chrome、Opera、Safari 浏览器支持该 API。

假设有如下的网页代码。

```js
<div id="myDiv" data-id="myId"></div>
```

以 data-id 属性为例，要读取这个值，可以用 dataset.id。

```js
var id = document.getElementById("myDiv").dataset.id;
```

要设置 data-id 属性，可以直接对 dataset.id 赋值。这时，如果 data-id 属性不存在，将会被创造出来。

```js
document.getElementById("myDiv").dataset.id = "hello";
```

删除一个 data-*属性，可以直接使用 delete 命令。

```js
delete document.getElementById("myDiv").dataset.id
```

IE 9 不支持 dataset 属性，可以用 getAttribute('data-foo')、removeAttribute('data-foo')、setAttribute('data-foo')、hasAttribute('data-foo') 代替。

需要注意的是，dataset 属性使用骆驼拼写法表示属性名，这意味着 data-hello-world 会用 dataset.helloWorld 表示。而如果此时存在一个 data-helloWorld 属性，该属性将无法读取，也就是说，data 属性本身只能使用连词号，不能使用骆驼拼写法。

### tabindex 属性

tabindex 属性用来指定，当前 HTML 元素节点是否被 tab 键遍历，以及遍历的优先级。

```js
var b1 = document.getElementById("button1");

b1.tabIndex = 1;
```

如果 tabindex = -1 ，tab 键跳过当前元素。

如果 tabindex = 0 ，表示 tab 键将遍历当前元素。如果一个元素没有设置 tabindex，默认值就是 0。

如果 tabindex 大于 0，表示 tab 键优先遍历。值越大，就表示优先级越大。

### 页面位置相关属性

（1）offsetParent 属性、offsetTop 属性和 offsetLeft 属性

这三个属性提供 Element 对象在页面上的位置。

*   offsetParent：当前 HTML 元素的最靠近的、并且 CSS 的 position 属性不等于 static 的父元素。
*   offsetTop：当前 HTML 元素左上角相对于 offsetParent 的垂直位移。
*   offsetLeft：当前 HTML 元素左上角相对于 offsetParent 的水平位移。

如果 Element 对象的父对象都没有将 position 属性设置为非 static 的值（比如 absolute 或 relative），则 offsetParent 属性指向 body 元素。另外，计算 offsetTop 和 offsetLeft 的时候，是从边框的左上角开始计算，即 Element 对象的 border 宽度不计入 offsetTop 和 offsetLeft。

### style 属性

style 属性用来读写页面元素的行内 CSS 属性，详见本章《CSS 操作》一节。

### Element 对象的方法

（1）选择子元素的方法

Element 对象也部署了 document 对象的 4 个选择子元素的方法，而且用法完全一样。

*   querySelector 方法
*   querySelectorAll 方法
*   getElementsByTagName 方法
*   getElementsByClassName 方法

上面四个方法只用于选择 Element 对象的子节点。因此，可以采用链式写法来选择子节点。

```js
document.getElementById('header').getElementsByClassName('a')
```

各大浏览器对这四个方法都支持良好，IE 的情况如下：IE 6 开始支持 getElementsByTagName，IE 8 开始支持 querySelector 和 querySelectorAll，IE 9 开始支持 getElementsByClassName。

（2）elementFromPoint 方法

该方法用于选择在指定坐标的最上层的 Element 对象。

```js
document.elementFromPoint(50,50)
```

上面代码了选中在(50,50)这个坐标的最上层的那个 HTML 元素。

（3）HTML 元素的属性相关方法

*   hasAttribute()：返回一个布尔值，表示 Element 对象是否有该属性。
*   getAttribute()
*   setAttribute()
*   removeAttribute()

（4）matchesSelector 方法

该方法返回一个布尔值，表示 Element 对象是否符合某个 CSS 选择器。

```js
document.querySelector('li').matchesSelector('li:first-child')
```

这个方法需要加上浏览器前缀，需要写成 mozMatchesSelector()、webkitMatchesSelector()、oMatchesSelector()、msMatchesSelector()。

（5）focus 方法

focus 方法用于将当前页面的焦点，转移到指定元素上。

```js
document.getElementById('my-span').focus();
```

### table 元素

表格有一些特殊的 DOM 操作方法。

*   insertRow()：在指定位置插入一个新行（tr）。
*   deleteRow()：在指定位置删除一行（tr）。
*   insertCell()：在指定位置插入一个单元格（td）。
*   deleteCell()：在指定位置删除一个单元格（td）。
*   createCaption()：插入标题。
*   deleteCaption()：删除标题。
*   createTHead()：插入表头。
*   deleteTHead()：删除表头。

下面是使用 JavaScript 生成表格的一个例子。

```js
var table = document.createElement('table');
var tbody = document.createElement('tbody');
table.appendChild(tbody);

for (var i = 0; i <= 9; i++) {
  var rowcount = i + 1;
  tbody.insertRow(i);
  tbody.rows[i].insertCell(0);
  tbody.rows[i].insertCell(1);
  tbody.rows[i].insertCell(2);
  tbody.rows[i].cells[0].appendChild(document.createTextNode('Row ' + rowcount + ', Cell 1'));
  tbody.rows[i].cells[1].appendChild(document.createTextNode('Row ' + rowcount + ', Cell 2'));
  tbody.rows[i].cells[2].appendChild(document.createTextNode('Row ' + rowcount + ', Cell 3'));
}

table.createCaption();
table.caption.appendChild(document.createTextNode('A DOM-Generated Table'));

document.body.appendChild(table);
```

这些代码相当易读，其中需要注意的就是 insertRow 和 insertCell 方法，接受一个表示位置的参数（从 0 开始的整数）。

table 元素有以下属性：

*   caption：标题。
*   tHead：表头。
*   tFoot：表尾。
*   rows：行元素对象，该属性只读。
*   rows.cells：每一行的单元格对象，该属性只读。
*   tBodies：表体，该属性只读。

## 参考链接

*   Louis Lazaris, [Thinking Inside The Box With Vanilla JavaScript](http://coding.smashingmagazine.com/2013/10/06/inside-the-box-with-vanilla-javascript/)
*   David Walsh, [HTML5 classList API](http://davidwalsh.name/classlist)
*   Derek Johnson, [The classList API](http://html5doctor.com/the-classlist-api/)
*   Mozilla Developer Network, [element.dataset API](http://davidwalsh.name/element-dataset)
*   David Walsh, [The element.dataset API](http://davidwalsh.name/element-dataset)