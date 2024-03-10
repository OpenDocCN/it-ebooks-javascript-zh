# 初步体验

# Lesson-1

===

这个版本呢,先来加四个很简单的方法感受感受下!

首先 3 个 class 不用说了

```js
hasClass : function(cls) {
    var reg = new RegExp('(\\s|^)' + cls + '(\\s|$)');
    for (var i = 0; i < this.length; i++) {
        if (this[i].className.match(reg)) return true;
            return false;
    }
    return this;
},
addClass : function(cls) {
    var reg = new RegExp('(\\s|^)' + cls + '(\\s|$)');
    for (var i = 0; i < this.length; i++) {
        if(!this[i].className.match(reg))
            this[i].className += ' ' + cls;
    }
    return this;
},
removeClass : function(cls) {
    var reg = new RegExp('(\\s|^)' + cls + '(\\s|$)');
    for (var i = 0; i < this.length; i++) {
        if (this[i].className.match(reg))
            this[i].className = this[i].className.replace(cls,'');
    }
    return this;
} 
```

然后新增一个

```js
css : function(attr,val) {//链式测试
    console.log(this.length);
    for(var i = 0;i < this.length; i++) {
        if(arguments.length == 1) {
            return getComputedStyle(this[i],null)[attr];
        }
        this[i].style[attr] = val;
    }
    return this;
} 
```

这些其实都很简单,我们都要记住,我们封装的 DOM 对象是一个数组,所以一定都需要用循环来进行各种个样的处理.

然后 css 这我是用 arguments 的个数来进行判断是取值还是社值.

最后千万别忘了每个方法的最后都要 return this 以便链式调用.

大家可以自行拿这几个方法 log 出来看看是否是与 jQuery 的一样就知道是否成功了.