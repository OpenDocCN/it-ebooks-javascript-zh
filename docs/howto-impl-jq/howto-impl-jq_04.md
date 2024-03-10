# 完善 init 方法

# Lesson-3

===

修改 f(selector) 里的判断,新增 domReady

我们知道在 jQuery 中还有一种选择器写法

```js
$(function() {

}); 
```

在 dom 加载完毕后马上就执行,这样的方法会比 onload 更快,所以 domReady 对于我们来说一定是必不可少的

我们在 init 方法中要新增以下判断

```js
if(!selector) { return this; }

if (typeof selector == 'object') {
    var selector = [selector];
    for (var i = 0; i < selector.length; i++) {
        this[i] = selector[i];
    }
    this.length = selector.length;
    return this;
} else if (typeof selector == 'function') {
    Kodo.ready(selector);
    return;
} 
```

首先 selector 可能为 object 的情况,比如传入的是原生 dom 对象,dom 数组对象. 另外要记得转为数组`var selector = [selector];

因为有可能是一个元素比如是 window,document 等否则没法循环

然后 selector 如果是 function 那我们就认为他是 domReady

PS:在这我判断的并没有非常的全面,仅仅具备了基础功能

```js
Kodo.ready = function(fn) {

    doc.addEventListener('DOMContentLoaded',function() {
        fn && fn();
    },false);
    doc.removeEventListener('DOMContentLoaded',fn,true);

}; 
```

然后这个是 ready 的源码,由于我们只兼容高端浏览器所以仅仅需要这样写即可.

既然你都看到这了,还不给我一个 star 说得过去么你!! :(