# 12.3 jQuery 插件开发

所谓“插件”，就是用户自己新增的 jQuery 实例对象的方法。由于该方法要被所有实例共享，所以只能定义在 jQuery 构造函数的原型对象（prototype）之上。对于用户来说，把一些常用的操作封装成插件（plugin），使用起来会非常方便。

*   插件的编写
    *   原理
    *   侦测环境
    *   实例
    *   插件的发布
    *   参考链接

## 插件的编写

### 原理

本质上，jQuery 插件是定义在 jQuery 构造函数的 prototype 对象上面的一个方法，这样做就能使得所有 jQuery 对象的实例都能共享这个方法。因为 jQuery 构造函数的 prototype 对象被简写成 jQuery.fn 对象，所以插件采用下面的方法定义。

```js
jQuery.fn.myPlugin = function() {
  // Do your awesome plugin stuff here
};
```

更好的做法是采用下面的写法，这样就能在函数体内自由使用美元符号（$）。

```js
;(function ($){
  $.fn.myPlugin = function (){
    // Do your awesome plugin stuff here
  };
})(jQuery);
```

上面代码的最前面有一个分号，这是为了防止多个脚本文件合并时，其他脚本的结尾语句没有添加分号，造成运行时错误。

有时，还可以把顶层对象（window）作为参数输入，这样可以加快代码的执行速度和执行更有效的最小化操作。

```js
;(function ($, window) {
  $.fn.myPlugin = function() {
    // Do your awesome plugin stuff here
  };
}(jQuery, window));
```

需要注意的是，在插件内部，this 关键字指的是 jQuery 对象的实例。而在一般的 jQuery 回调函数之中，this 关键字指的是 DOM 对象。

```js
(function ($){
  $.fn.maxHeight = function (){
    var max = 0;
    // 下面这个 this，指的是 jQuery 对象实例
    this.each(function() {
        // 回调函数内部，this 指的是 DOM 对象
        max = Math.max(max, $(this).height());
    });
    return max;
  };
})(jQuery);
```

上面这个 maxHeight 插件的作用是，返回一系列 DOM 对象中高度最高的那个对象的高度。

大多数情况下，插件应该返回 jQuery 对象，这样可以保持链式操作。

```js
(function ($){
  $.fn.greenify = function (){
    this.css("color", "green");
    return this;
  };
})(jQuery);

$("a").greenify().addClass("greenified");
```

上面代码返回 this 对象，即 jQuery 对象实例，所以接下来可以采用链式操作。

对于包含多个 jQuery 对象的结果集，可以采用 each 方法，进行处理。

```js
$.fn.myNewPlugin = function() {
    return this.each(function() {
        // 处理每个对象
    });
};
```

插件可以接受一个属性对象参数。

```js
(function ($){
  $.fn.tooltip = function (options){
    var settings = $.extend( {
      'location'         : 'top',
      'background-color' : 'blue'
    }, options);
    return this.each(function (){
      // 填入插件代码
    });
  };
})(jQuery);
```

上面代码使用 extend 方法，为参数对象设置属性的默认值。

### 侦测环境

jQuery 逐渐从浏览器环境，变为也可以用于服务器环境。所以，定义插件的时候，最好首先侦测一下运行环境。

```js
if (typeof module === "object" && typeof module.exports === "object") {
  // CommonJS 版本
} else {
  // 浏览器版本
}
```

## 实例

下面是一个将 a 元素的 href 属性添加到网页的插件。

```js
(function($){
    $.fn.showLinkLocation = function() {
        return this.filter('a').append(function(){
            return ' (' + this.href + ')';
        });
    };
}(jQuery));

// 用法
$('a').showLinkLocation();
```

从上面的代码可以看到，插件的开发和使用都非常简单。

## 插件的发布

编写插件以后，可以将它发布到[jQuery 官方网站](http://plugins.jquery.com/)上。

首先，编写一个插件的信息文件 yourPluginName.jquery.json。文件名中的 yourPluginName 表示你的插件名。

```js
{
  "name": "plugin_name",
  "title": "plugin_long_title",
  "description": "...",
  "keywords": ["jquery", "plugins"],
  "version": "0.0.1",
  "author": {
    "name": "...",
    "url": "..."
  },
  "maintainers": [
    {
      "name": "...",
      "url": "..."
    }
  ],
  "licenses": [
    {
      "type": "MIT",
      "url": "http://www.opensource.org/licenses/mit-license.php"
    }
  ],
  "bugs": "...", // bugs url
  "homepage": "...", // homepage url
  "docs": "...", // docs url
  "download": "...", // download url
  "dependencies": {
    "jquery": ">=1.4"
  }
}
```

上面是一个插件信息文件的实例。

然后，将代码文件发布到 Github，在设置页面点击“Service Hooks/WebHook URLs”选项，填入网址[`plugins.jquery.com/postreceive-hook，再点击“Update`](http://plugins.jquery.com/postreceive-hook%EF%BC%8C%E5%86%8D%E7%82%B9%E5%87%BB%E2%80%9CUpdate) Settings”进行保存。

最后，为代码加上版本，push 到 github，你的插件就会加入 jQuery 官方插件库。

```js
git tag 0.1.0
git push origin --tags
```

以后，你要发布新版本，就做一个新的 tag。

## 参考链接

*   jquery-boilerplate, [How did we get here?](https://github.com/jquery-boilerplate/jquery-boilerplate/wiki/How-did-we-get-here%3F)
*   jquery-boilerplate, [How to publish a plugin in jQuery Plugin Registry](https://github.com/jquery-boilerplate/jquery-boilerplate/wiki/How-to-publish-a-plugin-in-jQuery-Plugin-Registry)