# jQuery 支持 2 种插件方式

## 1.2\. jQuery 支持 2 种插件方式

*   $.somePlugin
*   $.fn.somePlugin

### 1.2.1\. Global 工具类的插件

比如

```js
$.trim('hello world  ') 
```

这个方法是用于去掉空格的工具方法。它其实是给 jQuery 对象上增加了 trim 方法。

此种插件一般是工具类的方法。

### 1.2.2\. 基于 selector 的插件

比如

```js
$('.mytab').tab({
    change: function(index){
        console.log('current index = ' + index);
    }
}); 
```

这实际是一个 tab 插件最简单的用法。

这个插件的特点是它前面必须是选择器，那么就可以是一个也可以多个，因为选择器有很多种，选择器返回的是数组，比如例子中的.mytab，在一个 html 文档里可以有多个

比如

```js
<div class='mytab'>
</div>

<div class='mytab'>
</div>

<div class='mytab'>
</div>

<div class='mytab'>
</div> 
```

这样就可以形成 4 个 tab 里。

jQuery 的哲学是“write less，do more”,它做到了么？

### 1.2.3\. 我们要讲的是后者

相比较而言，工具类的插件基本没有难度，它就是普通的 function，即没有配置也没有 selector，所以这里不做介绍。 而基于 selector 的插件复用程度高，又是基于配置的，在实际中应用非常广泛，是我们本节介绍的重点。