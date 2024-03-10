# 12.1 jQuery 概述

jQuery 是目前使用最广泛的 JavaScript 函数库。据[统计](http://w3techs.com/technologies/details/js-jquery/all/all)，全世界 57.5%的网站使用 jQuery，在使用 JavaScript 函数库的网站中，93.0%使用 jQuery。它已经成了开发者必须学会的技能。

jQuery 的最大优势有两个。首先，它基本是一个 DOM 操作工具，可以使操作 DOM 对象变得异常容易。其次，它统一了不同浏览器的 API 接口，使得代码在所有现代浏览器均能运行，开发者不用担心浏览器之间的差异。

*   jQuery 的加载
*   jQuery 基础
    *   jQuery 对象
    *   jQuery 构造函数
    *   jQuery 构造函数返回的结果集
    *   链式操作
    *   $(document).ready()
    *   $.noConflict 方法
    *   jQuery 实例对象的方法
        *   结果集的过滤方法
        *   DOM 相关方法
        *   添加、复制和移动网页元素的方法
        *   动画效果方法
        *   其他方法
    *   事件处理
        *   事件绑定的简便方法
        *   on 方法，trigger 方法，off 方法
        *   event 对象
        *   一次性事件
    *   参考链接

## jQuery 的加载

一般采用下面的写法，在网页中加载 jQuery。

```js
<script type="text/javascript" src="//code.jquery.com/jquery-1.11.0.min.js"></script>
<script>
window.jQuery || document.write('<script src="js/jquery-1.11.0.min.js" type="text/javascript"><\/script>')
</script>
```

上面代码有两点需要注意。一是采用[CDN](http://jquery.com/download/#using-jquery-with-a-cdn)加载。如果 CDN 加载失败，则退回到本地加载。二是采用协议无关的加载网址（使用双斜线表示），同时支持 http 协议和 https 协议。

目前常用的 jQuery CDN 有以下这些。

*   [Google CDN](https://developers.google.com/speed/libraries/devguide#jquery)
*   [Microsoft CDN](http://www.asp.net/ajax/cdn#jQuery_Releases_on_the_CDN_0)
*   [jQuery CDN](http://jquery.com/download/#jquery-39-s-cdn-provided-by-maxcdn)
*   [CDNJS CDN](http://cdnjs.com/libraries/jquery/)
*   [jsDelivr CDN](http://www.jsdelivr.com/#!jquery)

上面这段代码最好放到网页尾部。如果需要支持 IE 6/7/8，就使用 jQuery 1.x 版，否则使用最新版。

## jQuery 基础

### jQuery 对象

jQuery 最重要的概念，就是 jQuery 对象。它是一个全局对象，可以简写为美元符号$。也就是说，jQuery 和$两者是等价的。

在网页中加载 jQuery 函数库以后，就可以使用 jQuery 对象了。jQuery 的全部方法，都定义在这个对象上面。

```js
var listItems = jQuery('li');
// or
var listItems = $('li');
```

上面两行代码是等价的，表示选中网页中所有的 li 元素。

### jQuery 构造函数

jQuery 对象本质上是一个构造函数，主要作用是返回 jQuery 对象的实例。比如，上面代码表面上是选中 li 元素，实际上是返回对应于 li 元素的 jQuery 实例。因为只有这样，才能在 DOM 对象上使用 jQuery 提供的各种方法。

```js
$('body').nodeType
// undefined

$('body') instanceof jQuery
// true
```

上面代码表示，由于 jQuery 返回的不是 DOM 对象，所以没有 DOM 属性 nodeType。它返回的是 jQuery 对象的实例。

（1）CSS 选择器作为参数

jQuery 构造函数的参数，主要是 CSS 选择器，就像上面的那个例子。下面是另外一些 CSS 选择器的例子。

```js
$("*")
$("#lastname")
$(".intro")
$("h1,div,p")
$("p:last")
$("tr:even")
$("p:first-child")
$("p:nth-of-type(2)")
$("div + p")
$("div:has(p)")
$(":empty")
$("[title^='Tom']")
```

本书不讲解 CSS 选择器，请读者参考有关书籍和 jQuery 文档。

（2）DOM 对象作为参数

jQuery 构造函数的参数，还可以是 DOM 对象。它也会被转为 jQuery 对象的实例。

```js
$(document.body) instanceof jQuery
// true
```

上面代码中，jQuery 的参数不是 CSS 选择器，而是一个 DOM 对象，返回的依然是 jQuery 对象的实例。

如果有多个 DOM 元素要转为 jQuery 对象的实例，可以把 DOM 元素放在一个数组里，输入 jQuery 构造函数。

```js
$([document.body, document.head])
```

（3）HTML 代码作为参数

如果直接在 jQuery 构造函数中输入 HTML 代码，返回的也是 jQuery 实例。

```js
$('<li class="greet">test</li>')
```

上面代码从 HTML 代码生成了一个 jQuery 实例，它与从 CSS 选择器生成的 jQuery 实例完全一样。唯一的区别就是，它对应的 DOM 结构不属于当前文档。

上面代码也可以写成下面这样。

```js
$( '<li>', {
  html: 'test',
  'class': 'greet'
});
```

上面代码中，由于 class 是 javaScript 的保留字，所以只能放在引号中。

通常来说，上面第二种写法是更好的写法。

```js
$('<input class="form-control" type="hidden" name="foo" value="bar" />')

// 相当于
$('<input/>', {
    class: 'form-control',
    type: 'hidden',
    name: 'foo',
    value: 'bar'
})

// 或者
$('<input/>')
.addClass('form-control')
.attr('type', 'hidden')
.attr('name', 'foo')
.val('bar')
```

（4）第二个参数

默认情况下，jQuery 将文档的根元素（html）作为寻找匹配对象的起点。如果要指定某个网页元素（比如某个 div 元素）作为寻找的起点，可以将它放在 jQuery 函数的第二个参数。

```js
$('li', someElement);
```

上面代码表示，只寻找属于 someElement 对象下属的 li 元素。someElement 可以是 jQuery 对象的实例，也可以是 DOM 对象。

### jQuery 构造函数返回的结果集

jQuery 的核心思想是“先选中某些网页元素，然后对其进行某种处理”（find something, do something），也就是说，先选择后处理，这是 jQuery 的基本操作模式。所以，绝大多数 jQuery 操作都是从选择器开始的，返回一个选中的结果集。

（1）length 属性

jQuery 对象返回的结果集是一个类似数组的对象，包含了所有被选中的网页元素。查看该对象的 length 属性，可以知道到底选中了多少个结果。

```js
if ( $('li').length === 0 ) {
    console.log('不含 li 元素');
}
```

上面代码表示，如果网页没有 li 元素，则返回对象的 length 属性等于 0。这就是测试有没有选中的标准方法。

所以，如果想知道 jQuery 有没有选中相应的元素，不能写成下面这样。

```js
if ($('div.foo')) { ... }
```

因为不管有没有选中，jQuery 构造函数总是返回一个实例对象，而对象的布尔值永远是 true。使用 length 属性才是判断有没有选中的正确方法。

```js
if ($('div.foo').length) { ... }
```

（2）下标运算符

jQuery 选择器返回的是一个类似数组的对象。但是，使用下标运算符取出的单个对象，并不是 jQuery 对象的实例，而是一个 DOM 对象。

```js
$('li')[0] instanceof jQuery // false
$('li')[0] instanceof Element // true
```

上面代码表示，下标运算符取出的是 Element 节点的实例。所以，通常使用下标运算符将 jQuery 实例转回 DOM 对象。

（3）is 方法

is 方法返回一个布尔值，表示选中的结果是否符合某个条件。这个用来验证的条件，可以是 CSS 选择器，也可以是一个函数，或者 DOM 元素和 jQuery 实例。

```js
$('li').is('li') // true

$('li').is($('.item')) 

$('li').is(document.querySelector('li'))

$('li').is(function() {
      return $("strong", this).length === 0;
});
```

（4）get 方法

jQuery 实例的 get 方法是下标运算符的另一种写法。

```js
$('li').get(0) instanceof Element // true
```

（5）eq 方法

如果想要在结果集取出一个 jQuery 对象的实例，不需要取出 DOM 对象，则使用 eq 方法，它的参数是实例在结果集中的位置（从 0 开始）。

```js
$('li').eq(0) instanceof jQuery // true
```

由于 eq 方法返回的是 jQuery 的实例，所以可以在返回结果上使用 jQuery 实例对象的方法。

（6）each 方法，map 方法

这两个方法用于遍历结果集，对每一个成员进行某种操作。

each 方法接受一个函数作为参数，依次处理集合中的每一个元素。

```js
$('li').each(function( index, element) {
  $(element).prepend( '<em>' + index + ': </em>' );
});

// <li>Hello</li>
// <li>World</li>
// 变为
// <li><em>0: </em>Hello</li>
// <li><em>1: </em>World</li>
```

从上面代码可以看出，作为 each 方法参数的函数，本身有两个参数，第一个是当前元素在集合中的位置，第二个是当前元素对应的 DOM 对象。

map 方法的用法与 each 方法完全一样，区别在于 each 方法没有返回值，只是对每一个元素执行某种操作，而 map 方法返回一个新的 jQuery 对象。

```js
$("input").map(function (index, element){
    return $(this).val();
})
.get()
.join(", ")
```

上面代码表示，将所有 input 元素依次取出值，然后通过 get 方法得到一个包含这些值的数组，最后通过数组的 join 方法返回一个逗号分割的字符串。

（8）内置循环

jQuery 默认对当前结果集进行循环处理，所以如果直接使用 jQuery 内置的某种方法，each 和 map 方法是不必要的。

```js
$(".class").addClass("highlight");
```

上面代码会执行一个内部循环，对每一个选中的元素进行 addClass 操作。由于这个原因，对上面操作加上 each 方法是不必要的。

```js
$(".class").each(function(index,element){
     $(element).addClass("highlight");
});

// 或者

$(".class").each(function(){
    $(this).addClass("highlight");
});
```

上面代码的 each 方法，都是没必要使用的。

由于内置循环的存在，从性能考虑，应该尽量减少不必要的操作步骤。

```js
$(".class").css("color", "green").css("font-size", "16px");

// 应该写成

$(".class").css({ 
  "color": "green",
  "font-size": "16px"
});
```

### 链式操作

jQuery 最方便的一点就是，它的大部分方法返回的都是 jQuery 对象，因此可以链式操作。也就是说，后一个方法可以紧跟着写在前一个方法后面。

```js
$('li').click(function (){
    $(this).addClass('clicked');
})
.find('span')
.attr( 'title', 'Hover over me' );
```

### $(document).ready()

$(document).ready 方法接受一个函数作为参数，将该参数作为 document 对象的 DOMContentLoaded 事件的回调函数。也就是说，当页面解析完成（即下载完标签）以后，在所有外部资源（图片、脚本等）完成加载之前，该函数就会立刻运行。

```js
$( document ).ready(function() {
  console.log( 'ready!' );
});
```

上面代码表示，一旦页面完成解析，就会运行 ready 方法指定的函数，在控制台显示“ready!”。

该方法通常作为网页初始化手段使用，jQuery 提供了一种简写法，就是直接把回调函数放在 jQuery 对象中。

```js
$(function() {
  console.log( 'ready!' );
});
```

上面代码与前一段代码是等价的。

### $.noConflict 方法

jQuery 使用美元符号（$）指代 jQuery 对象。某些情况下，其他函数库也会用到美元符号，为了避免冲突，$.noConflict 方法允许将美元符号与 jQuery 脱钩。

```js
<script src="other_lib.js"></script>
<script src="jquery.js"></script>
<script>$.noConflict();</script>
```

上面代码就是$.noConflict 方法的一般用法。在加载 jQuery 之后，立即调用该方法，会使得美元符号还给前面一个函数库。这意味着，其后再调用 jQuery，只能写成 jQuery.methond 的形式，而不能用$.method 了。

为了避免冲突，可以考虑从一开始就只使用 jQuery 代替美元符号。

## jQuery 实例对象的方法

除了上一节提到的 is、get、eq 方法，jQuery 实例还有许多其他方法。

### 结果集的过滤方法

选择器选出一组符合条件的网页元素以后，jQuery 提供了许多方法，可以过滤结果集，返回更准确的目标。

（1）first 方法，last 方法

first 方法返回结果集的第一个成员，last 方法返回结果集的最后一个成员。

```js
$("li").first()

$("li").last()
```

（2）next 方法，prev 方法

next 方法返回紧邻的下一个同级元素，prev 方法返回紧邻的上一个同级元素。

```js
$("li").first().next()
$("li").last().prev()

$("li").first().next('.item')
$("li").last().prev('.item')
```

如果 next 方法和 prev 方法带有参数，表示选择符合该参数的同级元素。

（3）parent 方法，parents 方法，children 方法

parent 方法返回当前元素的父元素，parents 方法返回当前元素的所有上级元素（直到 html 元素）。

```js
$("p").parent()
$("p").parent(".selected")

$("p").parents()
$("p").parents("div")
```

children 方法返回选中元素的所有子元素。

```js
$("div").children()
$("div").children(".selected")

// 下面的写法结果相同，但是效率较低

$('div > *')
$('div > .selected')
```

上面这三个方法都接受一个选择器作为参数。

（4）siblings 方法，nextAll 方法，prevAll 方法

siblings 方法返回当前元素的所有同级元素。

```js
$('li').first().siblings()
$('li').first().siblings('.item')
```

nextAll 方法返回当前元素其后的所有同级元素，prevAll 方法返回当前元素前面的所有同级元素。

```js
$('li').first().nextAll()
$('li').last().prevAll()
```

（5）closest 方法，find 方法

closest 方法返回当前元素，以及当前元素的所有上级元素之中，第一个符合条件的元素。find 方法返回当前元素的所有符合条件的下级元素。

```js
$('li').closest('div')
$('div').find('li')
```

上面代码中的 find 方法，选中所有 div 元素下面的 li 元素，等同于$('li', 'div')。由于这样写缩小了搜索范围，所以要优于$('div li')的写法。

（6）find 方法，add 方法，addBack 方法，end 方法

add 方法用于为结果集添加元素。

```js
$('li').add('p')
```

addBack 方法将当前元素加回原始的结果集。

```js
$('li').parent().addBack()
```

end 方法用于返回原始的结果集。

```js
$('li').first().end()
```

（7）filter 方法，not 方法，has 方法

filter 方法用于过滤结果集，它可以接受多种类型的参数，只返回与参数一致的结果。

```js
// 返回符合 CSS 选择器的结果
$('li').filter('.item')

// 返回函数返回值为 true 的结果
$("li").filter(function(index) {
    return index % 2 === 1;
})

// 返回符合特定 DOM 对象的结果
$("li").filter(document.getElementById("unique"))

// 返回符合特定 jQuery 实例的结果
$("li").filter($("#unique"))
```

not 方法的用法与 filter 方法完全一致，但是返回相反的结果，即过滤掉匹配项。

```js
$('li').not('.item')
```

has 方法与 filter 方法作用相同，但是只过滤出子元素符合条件的元素。

```js
$("li").has("ul")
```

上面代码返回具有 ul 子元素的 li 元素。

### DOM 相关方法

许多方法可以对 DOM 元素进行处理。

（1）html 方法和 text 方法

html 方法返回该元素包含的 HTML 代码，text 方法返回该元素包含的文本。

假定网页只含有一个 p 元素。

```js
<p><em>Hello World!</em></p>
```

html 方法和 text 方法的返回结果分别如下。

```js
$('p').html()
// <em>Hello World!</em> 

$('p').text()
// Hello World!
```

jQuery 的许多方法都是取值器（getter）与赋值器（setter）的合一，即取值和赋值都是同一个方法，不使用参数的时候为取值器，使用参数的时候为赋值器。

上面代码的 html 方法和 text 方法都没有参数，就会当作取值器使用，取回结果集的第一个元素所包含的内容。如果对这两个方法提供参数，就是当作赋值器使用，修改结果集所有成员的内容，并返回原来的结果集，以便进行链式操作。

```js
$('p').html('<strong>你好</strong>')
// 网页代码变为<p><strong>你好</strong></p> 

$('p').text('你好')
// 网页代码变为<p>你好</p>
```

下面要讲到的 jQuery 其他许多方法，都采用这种同一个方法既是取值器又是赋值器的模式。

html 方法和 text 方法还可以接受一个函数作为参数，函数的返回值就是网页元素所要包含的新的代码和文本。这个函数接受两个参数，第一个是网页元素在集合中的位置，第二个参数是网页元素原来的代码或文本。

```js
$('li').html(function (i, v){
    return (i + ': ' + v);       
})

// <li>Hello</li>
// <li>World</li>
// 变为
// <li>0: Hello</li>
// <li>1: World</li>
```

（2）addClass 方法，removeClass 方法，toggleClass 方法

addClass 方法用于添加一个类，removeClass 方法用于移除一个类，toggleClass 方法用于折叠一个类（如果无就添加，如果有就移除）。

```js
$('li').addClass('special')
$('li').removeClass('special')
$('li').toggleClass('special')
```

（3）css 方法

css 方法用于改变 CSS 设置。

该方法可以作为取值器使用。

```js
$('h1').css('fontSize');
```

css 方法的参数是 css 属性名。这里需要注意，CSS 属性名的 CSS 写法和 DOM 写法，两者都可以接受，比如 font-size 和 fontSize 都行。

css 方法也可以作为赋值器使用。

```js
$('li').css('padding-left', '20px')
// 或者
$('li').css({
  'padding-left': '20px'
});
```

上面两种形式都可以用于赋值，jQuery 赋值器基本上都是如此。

（4）val 方法

val 方法返回结果集第一个元素的值，或者设置当前结果集所有元素的值。

```js
$('input[type="text"]').val()

$('input[type="text"]').val('new value')
```

（5）prop 方法，attr 方法

首先，这里要区分两种属性。

一种是网页元素的属性，比如 a 元素的 href 属性、img 元素的 src 属性。这要使用 attr 方法读写。

```js
// 读取属性值
$('textarea').attr(name)

//写入属性值
$('textarea').attr(name, val)
```

另一种是 DOM 元素的属性，比如 tagName、nodeName、nodeType 等等。这要使用 prop 方法读写。

```js
// 读取属性值
$('textarea').prop(name)

// 写入属性值
$('textarea').prop(name, val)
```

所以，attr 方法和 prop 方法针对的是不同的属性。在英语中，attr 是 attribute 的缩写，prop 是 property 的缩写，中文很难表达出这种差异。有时，attr 方法和 prop 方法对同一个属性会读到不一样的值。比如，网页上有一个单选框。

```js
<input type="checkbox" checked="checked" />
```

对于 checked 属性，attr 方法读到的是 checked，prop 方法读到的是 true。

```js
$(input[type=checkbox]).attr("checked") // "checked"

$(input[type=checkbox]).prop("checked") // true
```

可以看到，attr 方法读取的是网页上该属性的值，而 prop 方法读取的是 DOM 元素的该属性的值，根据规范，element.checked 应该返回一个布尔值。所以，判断单选框是否选中，要使用 prop 方法。事实上，不管这个单选框是否选中，attr("checked")的返回值都是 checked。

```js
if ($(elem).prop("checked")) { /*... */ };

// 下面两种方法亦可

if ( elem.checked ) { /*...*/ };
if ( $(elem).is(":checked") ) { /*...*/ };
```

（6）removeProp 方法，removeAttr 方法

removeProp 方法移除某个 DOM 属性，removeAttr 方法移除某个 HTML 属性。

```js
$("a").prop("oldValue",1234).removeProp('oldValue')
$('a').removeAttr("title")
```

（7）data 方法

data 方法用于在一个 DOM 对象上储存数据。

```js
// 储存数据
$("body").data("foo", 52);

// 读取数据
$("body").data("foo");
```

该方法可以在 DOM 节点上储存各种类型的数据。

### 添加、复制和移动网页元素的方法

jQuery 方法提供一系列方法，可以改变元素在文档中的位置。

（1）append 方法，appendTo 方法

append 方法将参数中的元素插入当前元素的尾部。

```js
$("div").append("<p>World</p>")

// <div>Hello </div>
// 变为
// <div>Hello <p>World</p></div>
```

appendTo 方法将当前元素插入参数中的元素尾部。

```js
$("<p>World</p>").appendTo("div")
```

上面代码返回与前一个例子一样的结果。

（2）prepend 方法，prependTo 方法

prepend 方法将参数中的元素，变为当前元素的第一个子元素。

```js
$("p").prepend("Hello ")

// <p>World</p>
// 变为
// <p>Hello World</p>
```

如果 prepend 方法的参数不是新生成的元素，而是当前页面已存在的元素，则会产生移动元素的效果。

```js
$("p").prepend("strong")

// <strong>Hello </strong><p>World</p>
// 变为
// <p><strong>Hello </strong>World</p>
```

上面代码运行后，strong 元素的位置将发生移动，而不是克隆一个新的 strong 元素。不过，如果当前结果集包含多个元素，则除了第一个以后，后面的 p 元素都将插入一个克隆的 strong 子元素。

prependTo 方法将当前元素变为参数中的元素的第一个子元素。

```js
$("<p></p>").prependTo("div")

// <div></div>
// 变为
// <div><p></p></div>
```

（3）after 方法，insertAfter 方法

after 方法将参数中的元素插在当前元素后面。

```js
$("div").after("<p></p>")

// <div></div>
// 变为
// <div></div><p></p>
```

insertAfter 方法将当前元素插在参数中的元素后面。

```js
$("<p></p>").insertAfter("div")
```

上面代码返回与前一个例子一样的结果。

（4）before 方法，insertBefore 方法

before 方法将参数中的元素插在当前元素的前面。

```js
$("div").before("<p></p>")

// <div></div>
// 变为
// <p></p><div></div>
```

insertBefore 方法将当前元素插在参数中的元素的前面。

```js
$("<p></p>").insertBefore("div")
```

上面代码返回与前一个例子一样的结果。

（5）wrap 方法，wrapAll 方法，unwrap 方法，wrapInner 方法

wrap 方法将参数中的元素变成当前元素的父元素。

```js
$("p").wrap("<div></div>")

// <p></p>
// 变为
// <div><p></p></div>
```

wrap 方法的参数还可以是一个函数。

```js
$("p").wrap(function() {
  return "<div></div>";
})
```

上面代码返回与前一个例子一样的结果。

wrapAll 方法为结果集的所有元素，添加一个共同的父元素。

```js
$("p").wrapAll("<div></div>")

// <p></p><p></p>
// 变为
// <div><p></p><p></p></div>
```

unwrap 方法移除当前元素的父元素。

```js
$("p").unwrap()

// <div><p></p></div>
// 变为
// <p></p>
```

wrapInner 方法为当前元素的所有子元素，添加一个父元素。

```js
$("p").wrapInner('<strong></strong>')

// <p>Hello</p>
// 变为
// <p><strong>Hello</strong></p>
```

（6）clone 方法

clone 方法克隆当前元素。

```js
var clones = $('li').clone();
```

对于那些有 id 属性的节点，clone 方法会连 id 属性一起克隆。所以，要把克隆的节点插入文档的时候，务必要修改或移除 id 属性。

（7）remove 方法，detach 方法，replaceWith 方法

remove 方法移除并返回一个元素，取消该元素上所有事件的绑定。detach 方法也是移除并返回一个元素，但是不取消该元素上所有事件的绑定。

```js
$('p').remove()
$('p').detach()
```

replaceWith 方法用参数中的元素，替换并返回当前元素，取消当前元素的所有事件的绑定。

```js
$('p').replaceWith('<div></div>')
```

### 动画效果方法

jQuery 提供一些方法，可以很容易地显示网页动画效果。但是，总体上来说，它们不如 CSS 动画强大和节省资源，所以应该优先考虑使用 CSS 动画。

如果将 jQuery.fx.off 设为 true，就可以将所有动画效果关闭，使得网页元素的各种变化一步到位，没有中间过渡的动画效果。

（1）动画效果的简便方法

jQuery 提供以下一些动画效果方法。

*   show：显示当前元素。
*   hide：隐藏当前元素。
*   toggle：显示或隐藏当前元素。
*   fadeIn：将当前元素的不透明度（opacity）逐步提升到 100%。
*   fadeOut：将当前元素的不透明度逐步降为 0%。
*   slideDown：以从上向下滑入的方式显示当前元素。
*   slideUp：以从下向上滑出的方式隐藏当前元素。
*   slideToggle：以垂直滑入或滑出的方式，折叠显示当前元素。

上面这些方法可以不带参数调用，也可以接受毫秒或预定义的关键字作为参数。

```js
$('.hidden').show();
$('.hidden').show(300);
$('.hidden').show('slow');
```

上面三行代码分别表示，以默认速度、300 毫秒、较慢的速度隐藏一个元素。

jQuery 预定义的关键字是在 jQuery.fx.speeds 对象上面，可以自行改动这些值，或者创造新的值。

```js
jQuery.fx.speeds.fast = 50;
jQuery.fx.speeds.slow = 3000;
jQuery.fx.speeds.normal = 1000;
```

上面三行代码重新定义 fast 和 slow 关键字对应的毫秒数，并且自定义了 normal 关键字，表示动画持续时间为 1500 毫秒。

你可以定义自己的关键字。

```js
jQuery.fx.speeds.blazing = 30;

// 调用
$('.hidden').show('blazing');
```

这些方法还可以接受一个函数，作为第二个参数，表示动画结束后的回调函数。

```js
$('p').fadeOut(300, function() {
  $(this).remove();
});
```

上面代码表示，p 元素以 300 毫秒的速度淡出，然后调用回调函数，将其从 DOM 中移除。

（2）animate 方法

上面这些动画效果方法，实际上都是 animate 方法的简便写法。在幕后，jQuery 都是统一使用 animate 方法生成各种动画效果。

animate 方法接受三个参数。

```js
$('div').animate({
    left: '+=50', // 增加 50
    opacity: 0.25,
    fontSize: '12px'
  },
  300, // 持续事件
  function() { // 回调函数
     console.log('done!');
  }
);
```

上面代码表示，animate 方法的第一个参数是一个对象，表示动画结束时相关 CSS 属性的值，第二个参数是动画持续的毫秒数，第三个参数是动画结束时的回调函数。需要注意的是，第一个参数对象的成员名称，必须与 CSS 属性名称一致，如果 CSS 属性名称带有连字号，则需要用“骆驼拼写法”改写。

（3）stop 方法，delay 方法

stop 方法表示立即停止执行当前的动画。

```js
$("#stop").click(function() {
  $(".block").stop();
});
```

上面代码表示，点击按钮后，block 元素的动画效果停止。

delay 方法接受一个时间参数，表示暂停多少毫秒后继续执行。

```js
$("#foo").slideUp(300).delay(800).fadeIn(400)
```

上面代码表示，slideUp 动画之后，暂停 800 毫秒，然后继续执行 fadeIn 动画。

### 其他方法

jQuery 还提供一些供特定元素使用的方法。

serialize 方法用于将表单元素的值，转为 url 使用的查询字符串。

```js
$( "form" ).on( "submit", function( event ) {
  event.preventDefault();
  console.log( $( this ).serialize() );
});
// single=Single&multiple=Multiple&check=check2&radio=radio1
```

serializeArray 方法用于将表单元素的值转为数组。

```js
$("form").submit(function (event){
  console.log($(this).serializeArray());
  event.preventDefault();
});
// [
//     {name : 'field1', value : 123},
//     {name : 'field2', value : 'hello world'}
// ]
```

## 事件处理

### 事件绑定的简便方法

jQuery 提供一系列方法，允许直接为常见事件绑定回调函数。比如，click 方法可以为一个元素绑定 click 事件的回调函数。

```js
$('li').click(function (e){
  console.log($(this).text());
});
```

上面代码为 li 元素绑定 click 事件的回调函数，点击后在控制台显示 li 元素包含的文本。

这样绑定事件的简便方法有如下一些：

*   click
*   keydown
*   keypress
*   keyup
*   mouseover
*   mouseout
*   mouseenter
*   mouseleave
*   scroll
*   focus
*   blur
*   resize
*   hover

如果不带参数调用这些方法，就是触发相应的事件，从而引发回调函数的运行。

```js
$('li').click()
```

上面代码将触发 click 事件的回调函数。

需要注意的是，通过这种方法触发回调函数，将不会引发浏览器对该事件的默认行为。比如，对 a 元素调用 click 方法，将只触发事先绑定的回调函数，而不会导致浏览器将页面导向 href 属性指定的网址。

下面是一个捕捉用户按下 escape 键的函数。

```js
$(document).keyup(function(e) {
  if (e.keyCode == 27) {
    $('body').toggleClass('show-nav');
    // $('body').removeClass('show-nav');
  }
});
```

上面代码中，用户按下 escape 键，jQuery 就会为 body 元素添加/去除名为 show-nav 的 class。

hover 方法需要特别说明。它接受两个回调函数作为参数，分别代表 mouseenter 和 mouseleave 事件的回调函数。

```js
$(selector).hover(handlerIn, handlerOut)

// 等同于

$(selector).mouseenter(handlerIn).mouseleave(handlerOut)
```

### on 方法，trigger 方法，off 方法

除了简便方法，jQuery 还提供事件处理的通用方法。

（1）on 方法

on 方法是 jQuery 事件绑定的统一接口。事件绑定的那些简便方法，其实都是 on 方法的简写形式。

on 方法接受两个参数，第一个是事件名称，第二个是回调函数。

```js
$('li').on('click', function (e){
  console.log($(this).text());
});
```

上面代码为 li 元素绑定 click 事件的回调函数。

> 注意，在回调函数内部，this 关键字指的是发生该事件的 DOM 对象。为了使用 jQuery 提供的方法，必须将 DOM 对象转为 jQuery 对象，因此写成$(this)。

on 方法允许一次为多个事件指定同样的回调函数。

```js
$('input[type="text"]').on('focus blur', function (){
  console.log('focus or blur');
});
```

上面代码为文本框的 focus 和 blur 事件绑定同一个回调函数。

on 方法还可以为当前元素的某一个子元素，添加回调函数。

```js
$('ul').on('click', 'li', function (e){
  console.log(this);
});
```

上面代码为 ul 的子元素 li 绑定 click 事件的回调函数。采用这种写法时，on 方法接受三个参数，子元素选择器作为第二个参数，夹在事件名称和回调函数之间。

这种写法有两个好处。首先，click 事件还是在 ul 元素上触发回调函数，但是会检查 event 对象的 target 属性是否为 li 子元素，如果为 true，再调用回调函数。这样就比为 li 元素一一绑定回调函数，节省了内存空间。其次，这种绑定的回调函数，对于在绑定后生成的 li 元素依然有效。

on 方法还允许向回调函数传入数据。

```js
$("ul" ).on("click", {name: "张三"}, function (event){
    console.log(event.data.name);
});
```

上面代码在发生 click 事件之后，会通过 event 对象的 data 属性，在控制台打印出所传入的数据（即“张三”）。

（2）trigger 方法

trigger 方法用于触发回调函数，它的参数就是事件的名称。

```js
$('li').trigger('click')
```

上面代码触发 li 元素的 click 事件回调函数。与那些简便方法一样，trigger 方法只触发回调函数，而不会引发浏览器的默认行为。

（3）off 方法

off 方法用于移除事件的回调函数。

```js
$('li').off('click')
```

上面代码移除 li 元素所有的 click 事件回调函数。

（4）事件的名称空间

同一个事件有时绑定了多个回调函数，这时如果想移除其中的一个回调函数，可以采用“名称空间”的方式，即为每一个回调函数指定一个二级事件名，然后再用 off 方法移除这个二级事件的回调函数。

```js
$('li').on('click.logging', function (){
  console.log('click.logging callback removed');
});

$('li').off('click.logging');
```

上面代码为 li 元素定义了二级事件 click.logging 的回调函数，click.logging 属于 click 名称空间，当发生 click 事件时会触发该回调函数。将 click.logging 作为 off 方法的参数，就会移除这个回调函数，但是对其他 click 事件的回调函数没有影响。

trigger 方法也适用带名称空间的事件。

```js
$('li').trigger('click.logging')
```

### event 对象

当回调函数被触发后，它们的参数通常是一个事件对象 event。

```js
$(document).on('click', function (e){
    // ...
});
```

上面代码的回调函数的参数 e，就代表事件对象 event。

event 对象有以下属性：

*   type：事件类型，比如 click。
*   which：触发该事件的鼠标按钮或键盘的键。
*   target：事件发生的初始对象。
*   data：传入事件对象的数据。
*   pageX：事件发生时，鼠标位置的水平坐标（相对于页面左上角）。
*   pageY：事件发生时，鼠标位置的垂直坐标（相对于页面左上角）。

event 对象有以下方法：

*   preventDefault：取消浏览器默认行为。
*   stopPropagation：阻止事件向上层元素传播。

### 一次性事件

one 方法指定一次性的回调函数，即这个函数只能运行一次。这对提交表单很有用。

```js
$("#button").one( "click", function() { return false; } );
```

one 方法本质上是回调函数运行一次，即解除对事件的监听。

```js
document.getElementById("#button").addEventListener("click", handler);

function handler(e) {
    e.target.removeEventListener(e.type, arguments.callee);
    return false;
}
```

上面的代码在点击一次以后，取消了对 click 事件的监听。如果有特殊需要，可以设定点击 2 次或 3 次之后取消监听，这都是可以的。

## 参考链接

*   Elijah Manor, [Do You Know When You Are Looping in jQuery?](http://www.elijahmanor.com/2013/01/yo-dawg-i-herd-you-like-loops-so-jquery.html)
*   Craig Buckler, [How to Create One-Time Events in JavaScript](http://www.sitepoint.com/create-one-time-events-javascript/)
*   jQuery Fundamentals, [jQuery Basics](http://jqfundamentals.com/chapter/jquery-basics)
*   jQuery Fundamentals, [Animating Your Pages with jQuery](http://jqfundamentals.com/chapter/effects)