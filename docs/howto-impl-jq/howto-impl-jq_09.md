# 引入 delegate 机制

# Lesson-8

===

事件机制

在讲事件机制之前呢,我们有一个很重要的东西要先讲,那就是如何实现事件委托(代理).

只有必须先明白了如何实现一个事件委托,我们才能更好的去实现 on 和 off.在我看来,on 和 off 里最难实现的就是他的事件委托.

```
function delegate(agent,type,selector,fn) {
    agent.addEventListener(type,function(e) {

        var target = e.target;
        var ctarget = e.currentTarget;
        var bubble = true;

        while(bubble && target != ctarget) {
            if(filiter(agent,selector,target)) {
                bubble = fn.call(target,e);
                return bubble;
            }
            target = target.parentNode;
        }
    },false);
    function filiter(agent,selector,target) {
        var nodes = agent.querySelectorAll(selector);
        for (var i = 0; i < nodes.length; i++) {
            if (nodes[i] == target) {
                return true;
            }
        }
    }
} 
```

以上是我对整个委托的实现,当然在这只做了非常简单的实现,没有对很多别的情况进行判断,也没有多个参数是否捕捉.

我们先拆解下分析.

```
function filiter(agent,selector,target) {
    var nodes = agent.querySelectorAll(selector);
    for (var i = 0; i < nodes.length; i++) {
        if (nodes[i] == target) {
            return true;
        }
    }
} 
```

先看这个方法,这其实就是一个元素过滤,作用就是为了过滤出我们委托的元素具体是谁.target 就是我们具体的委托元素

```
agent.addEventListener(type,function(e) {

    var target = e.target;
    var ctarget = e.currentTarget;
    var bubble = true; //是否阻止冒泡

    while(bubble && target != ctarget) {
        if(filiter(agent,selector,target)) {
            bubble = fn.call(target,e);
        }
        target = target.parentNode;
        return bubble;
    }
},false); 
```

然后是我们的主要部分.其实这里就很简单,while 的条件判断两个,第一个是是否阻止冒泡,第二个判断是冒泡是否到顶.

接着我们进行 filiter 进行过滤,如果返回 true,则是我们的委托元素,直接 call 即可.

如果不做过多的兼容处理,实现一个委托还是比较容易的.

PS:如果您还是不太明白,可以来这看更具体的解释.[`www.meckodo.com/?p=309`](http://www.meckodo.com/?p=309)

您的 star 是检验代码的唯一标准!:)