# 新增 find,last,eq,get,first,ajax

# Lesson-5

===

这个版本新增 6 个方法,find(),first(),last(),eq(),get(),ajax

先给出代码

```js
find : function(selector) {
    if(!selector) return;
    var context = this.selector;
    return new Kodo(context + ' ' + selector);
},
first : function() {
    return new Kodo(this[0])
},
last : function() {
    var num = this.length - 1;
    return new Kodo(this[num]);
},
eq : function(num) {
    var num = num < 0 ? (this.length - 1) : num;
    console.log(num);
    return new Kodo(this[num]);
},
get : function(num) {
    var num = num < 0 ? (this.length - 1) : num;
    console.log(num);
    return this[num];
} 
```

我们要仔细分辨下,这 4 个方法在 jQuery 中返回的都是什么对象?到底是 dom 对象还是 jQuery 对象.

明白了这个后就很容易能写出这 4 个方法

```js
find : function(selector) {
    if(!selector) return;
    var context = this.selector;
    return new Kodo(context + ' ' + selector);
} 
```

首先 find, 我们知道一般都会这样写 $('div').find('span') 查找 div 下的 span,返回的是 span 数组对象,而不是原生的 dom 对象

那么我们就可以换个思路,因为我们能拿到 $('div') 这个 selector 对吧? 也就是 div

既然又要 find('span'),我们的 selector 就可以写成 ('div span'),之后直接返回新的数组对象不就好了吗??

`var context = this.selector;` 先缓存当前的 selector 上下文,之后拼接我们 find 的 selector

所以最后 return 就变为 `new Kodo(context + ' ' + selector);` 虽然效率不一定高,总是一种解决思路不是吗?

```js
first : function() {
    return new Kodo(this[0])
},
last : function() {
    var num = this.length - 1;
    return new Kodo(this[num]);
},
eq : function(num) {
    var num = num < 0 ? (this.length - 1) : num;
    console.log(num);
    return new Kodo(this[num]);
},
get : function(num) {
    var num = num < 0 ? (this.length - 1) : num;
    console.log(num);
    return this[num];
} 
```

find 方法比较难解决,之后这 4 个就很容易了,first,last,eq,分别返回的都是封装后的对象,只有 get 返回的是原生 dom 对象.

我们根据之前的思路,直接取数组对象的 index,return 下新的即可,是不是很简单? :)

之后是 ajax 部分

之前说过之所以,可以用`$.ajax`直接调用,是因为可以把方法直接挂在在构造函数上,作为静态方法

所以我们只需要写好 ajax 最后把你想要公开的接口放在 Kodo 上即可

```js
Kodo.get = function(url,sucBack,complete) {
    var options = {
        url : url,
        success : sucBack,
        complete : complete
    };
    ajax(options);
};
Kodo.post = function(url,data,sucback,complete) {
    var options = {
        url : url,
        type : "POST",
        data : data,
        sucback    : sucback,
        complete : complete
    };
    ajax(options);
};
function ajax(options) {
    var defaultOptions = {
        url: false, //ajax 请求地址
        type : "GET",
        data : false,
        success: false, //数据成功返回后的回调方法
        complete: false //ajax 完成后的回调方法
    };
    for (var i in defaultOptions) {
        if (options[i] === undefined) {
            options[i] = defaultOptions[i];
        }
    }
    var xhr = new XMLHttpRequest();
    var url = options.url;
    xhr.open(options.type, url);
    xhr.onreadystatechange = onStateChange;
    if (options.type === 'POST') {
        xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    }
    xhr.send(options.data ? options.data : null);

    function onStateChange() {
        if (xhr.readyState == 4) {
            var result,
                status = xhr.status;

            if ((status >= 200 && status < 300) || status == 304) {
                result = xhr.responseText;
                if (window.JSON) {
                    result = JSON.parse(result);
                } else {
                    result = eval('(' + result + ')');
                }
                ajaxSuccess(result, xhr)
            } else {
                console.log("ERR", xhr.status);
            }
        }
    }
    function ajaxSuccess(data, xhr) {
        var status = 'success';
        options.success && options.success(data, options, status, xhr)
        ajaxComplete(status)
    }
    function ajaxComplete(status) {
        options.complete && options.complete(status);
    }
} 
```

在这我就不细讲 ajax 的具体过程,我也实现了一个比较简单的 ajax,就公开了 get 和 post 方法.大家可以实现一个更加复杂好用的 ajax 替换我这段代码

你说你都耐心的翻到这了? 不给我一个 star 说的过去么你?