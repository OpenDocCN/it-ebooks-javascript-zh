# 开始写第一个插件

## 1.4\. 开始写第一个插件

代码位于 plugin_first

plugin_first

让我们动手改造一下 tab.js 吧：

### 1.4.1\. 代码

```js
;(function($) { 

    $.fn.tab = function(options) {
        // 将 defaults 和 options 参数合并到{}
        var opts = $.extend({},$.fn.tab.defaults,options);

        return this.each(function() {
            var obj = $(this);

            $(obj).find('.tab_header li').on('mouseover', function (){
                $(obj).find('.tab_header li').removeClass('active');
                $(this).addClass('active');

                $(obj).find('.tab_content div').hide();
                $(obj).find('.tab_content div').eq($(this).index()).show();
            })      
        });
        // each end
    }

    //定义默认
    $.fn.tab.defaults = {

    };

})(jQuery); 
```

这段代码除了套用 jQuery plugin 模板外，就是几处 select 变成基于 obj 的查找的 selector，其他与之前无异。

是不是很简单？

### 1.4.2\. 解释一下配置项

```js
// 将 defaults 和 options 参数合并到{}
var opts = $.extend({},$.fn.tab.defaults,options); 
```

### 1.4.3\. 缓存 this

```js
// 将 defaults 和 options 参数合并到{}
var obj = $(this); 
```

### 1.4.4\. select 和$.fn

### 1.4.5\. 调用方式

```js
<script>
    $(function(){
        $('.tab').tab();
    });
</script> 
```

是不是更简单？

### 1.4.6\. jQuery plugin template

```js
;(function($) { 

    $.fn.XXXXXX = function(options) {
        // 将 defaults 和 options 参数合并到{}
        var opts = $.extend({},$.fn.XXXXXX.defaults,options);

        return this.each(function() {
            var obj = $(this);

            ...
        });
        // each end
    }

    //定义默认
    $.fn.XXXXXX.defaults = {

    };

})(jQuery); 
```

### 1.4.7\. 插件特点

1.  有默认项

```js
``` 
```js

$.fn.XXXXXX.defaults

```
```js 
```

1.  基于 selector

```js
``` 
```js

return this.each(function() { var obj = $(this);

```
 ...
});
```js 
```

解读：

*   有默认项，是约定大约配置，让用户用的时候如果没有个性化需求，可以很简单，安装插件的默认配置走，如果有个性化需求，修改配置项，同样很简单
*   基于 selector 意味着你可以复用，给 tag 或 class 应用此插件，以便写更少代码，完成更多功能

***亲，你明白插件的好处了么？如果没明白继续往下看***