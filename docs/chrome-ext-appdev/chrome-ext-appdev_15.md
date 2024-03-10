# 附录 B　i18n

使用`i18n`接口实现扩展应用程序的国际化。本节内容部分参考[`crxdoc-zh.appspot.com/apps/i18n`](https://crxdoc-zh.appspot.com/apps/i18n)。

在扩展应用程序的根目录下创建`_locales`文件夹，在`_locales`下以每个支持的语言的代码为名称创建子文件夹，然后在其中放入`message.json`指定对应语言的字符串。支持的语言和对应的语言代码参加[`developer.chrome.com/webstore/i18n#localeTable`](https://developer.chrome.com/webstore/i18n#localeTable)。

```js
root directiory
|- manifest.json
|- *.html, *.js
|- _locales
   |- en
      |- message.json
   |- zh-CN
      |- message.json 
```

在`message.json`指定字符串：

```js
{
    "extName": {
        "message": "一个国际化扩展",
        "description": "Extension Name"
    },
    ...
} 
```

在 Manifest 中调用国际化字符串：

```js
{
    "name": "__MSG_extName__",
    "default_locale": "en",
    ...
} 
```

在 JavaScript 中获取国际化字符串：

```js
title = chrome.i18n.getMessage('extName'); 
```

有一些预定的国际化字符串：

```js
@@extension_id：扩展应用 id
@@ui_locale：当前语言
@@bidi_dir：当前语言的文字方向，ltr 或 rtl
@@bidi_reversed_dir：如果@@bidi_dir 是 ltr 则该消息为 rtl，否则为 ltr
@@bidi_start_edge：如果@@bidi_dir 是 ltr 则该消息为 left，否则为 right
@@bidi_end_edge：如果@@bidi_dir 是 ltr 则该消息为 right，否则为 left 
```

上面这些预定义字符串可以直接在 JavaScript 和 CSS 中引用，如：

```js
body {
    direction: __MSG_@@bidi_dir__;
}

div#header {
    margin-bottom: 1.05em;
    overflow: hidden;
    padding-bottom: 1.5em;
    padding-__MSG_@@bidi_start_edge__: 0;
    padding-__MSG_@@bidi_end_edge__: 1.5em;
    position: relative;
} 
```

获取浏览器可接受的语言：

```js
chome.i18n.getAcceptLanguages(function(languageArray){
    //do something with languageArray
}); 
```

获得指定消息的本地化字符串。如果消息不存在，该方法返回空字符串`""`：

```js
var msg = chrome.i18n.getMessage(messageName, substitutions); 
```

获取浏览器用户界面的语言¹：

```js
var currentLanguage = chrome.i18n.getUILanguage(); 
```

^(1 从 Chrome 35 开始支持。)