# 九、jQuery

## AJAX

### 问题

你想要使用 jQuery 来调用 AJAX。

### 解决方案

```
$ ?= require 'jquery' # 由于 Node.js 的兼容性

$(document).ready ->
    # 基本示例
    $.get '/', (data) ->
        $('body').append "Successfully got the page."

    $.post '/',
        userName: 'John Doe'
        favoriteFlavor: 'Mint'
        (data) -> $('body').append "Successfully posted to the page."

    # 高级设置
    $.ajax '/',
        type: 'GET'
        dataType: 'html'
        error: (jqXHR, textStatus, errorThrown) ->
            $('body').append "AJAX Error: #{textStatus}"
        success: (data, textStatus, jqXHR) ->
            $('body').append "Successful AJAX call: #{data}"
```

jQuery 1.5 和更新版本都增加了一种新补充的 API ，用于处理不同的回调。

```
request = $.get '/'
    request.success (data) -> $('body').append "Successfully got the page again."
    request.error (jqXHR, textStatus, errorThrown) -> $('body').append "AJAX Error: ${textStatus}."
```

### 讨论

其中的 jQuery 和 $ 变量可以互换使用。另请参阅 [Callback bindings](http://coffeescript-cookbook.github.io/chapters/jquery/callback-bindings-jquery) 。

## 回调绑定

### 问题

你想要把一个回调与一个对象绑定在一起。

### 解决方案

```
$ ->
  class Basket
    constructor: () ->
      @products = []

      $('.product').click (event) =>
        @add $(event.currentTarget).attr 'id'

    add: (product) ->
      @products.push product
      console.log @products

  new Basket()
```

### 讨论

通过使用等号箭头（=>）取代正常箭头（->），函数将会自动与对象绑定，并可以访问 @- 可变量。

## 创建 jQuery 插件

### 问题

你想用 CoffeeScript 来创建 jQuery 插件。

### 解决方案

```
 # 参考 jQuery

$ = jQuery

 # 给 jQuery 添加插件对象

$.fn.extend
  # 把 pluginName 改成你的插件名字。
  pluginName: (options) ->
    # 默认设置
    settings =
      option1: true
      option2: false
      debug: false

    # 合并选项与默认设置。
    settings = $.extend settings, options

    # Simple logger.
    log = (msg) ->
      console?.log msg if settings.debug

    # _Insert magic here._
    return @each ()->
      log "Preparing magic show."
      # 你可以使用你的设置了。
      log "Option 1 value: #{settings.option1}"
```

### 讨论

这里有几个关于如何使用新插件的例子。

### JavaScript

```
$("body").pluginName({
  debug: true
});
```

### CoffeeScript

```
$("body").pluginName
  debug: true
```