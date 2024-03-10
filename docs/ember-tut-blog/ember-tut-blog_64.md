# 如何在{{input}}中使用 action

开发中经常遇到需要在一个`input`输入框触发 JS 函数，那么对于 Ember.js 的{{input}}又如何才能出发自定义的`action`呢？

实现起来非常简单！请看下面的代码演示：

**旧版本实现方式**

```js
{{input type="text" value=email action="clearTipInfo" on="focus-in"}} 
```

**新版本实现方式**

```js
{{input type="text" value=email focus-in="clearTipInfo"}} 
```

这是一段非常常见的输入框代码，稍微不同的是最后 2 个属性的设置，它们所起的作用就是：当输入框得到焦点的时候出发`action`所指定的方法`clearTipInfo`。触发的 JS 函数需要用`on`指定，JS 的函数不能随便写，所支持的 JS 函数请看[event names](http://emberjs.com/api/classes/Ember.View.html#toc_event-names)

**补充**

ember 并没有提供封装好的`radio`按钮组，如果你需要用到`radio`你可以自己使用组件封装，或者直接使用原生的 html。

如果你非得使用 Ember 风格的`radio`又不想自己定义组件那就是用现成的吧。下面推荐 2 个别人做好的组件：

1.  [ember-radio-button](https://www.npmjs.com/package/ember-radio-button)
2.  [ember-radio-buttons](https://www.npmjs.com/package/ember-radio-buttons)

**参考资料**

[`www.emberaddons.com/?query=radio`](https://www.emberaddons.com/?query=radio)