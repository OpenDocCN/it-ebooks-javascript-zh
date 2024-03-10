# Ember.js 入门指南四十一设置记录到 Store

[Ember](http://emberjs.com)的[Store](http://emberjs.com/api/data/classes/DS.Store.html)就像一个缓存池，用户提交的数据以及从服务器获取的数据会首先保存到 Store。如果用户再次请求相同的数据会直接从 Store 中获取，而不是发送 HTTP 请求去服务器获取。

当数据发生变化，数据首先更新到 Store 中，Store 会理解更新到其他页面。所以当你改变 Store 中的数据时，会立即反应出来，比如上一篇更新记录小结，当你修改`article`的数据时会立即反应到页面上。

## 1，push()方法

你可以调用`push()`方法一次性把多条数据保存到 Store 中。比如下面的代码：

```
//  app/routes/application.js

import Ember from 'ember';

export default Ember.Route.extend({  
    model: function() {
        this.store.push({
            data: [
                {
                    id: 1,
                    type: 'album',
                    attributes: {   //  设置 model 属性值
                        title: 'Fewer Moving Parts',
                        artist: 'David Bazan'
                        songCount: 10
                    },
                    relationships: {}  //  设置两个 model 的关联关系
                },
                {
                    id: 2,
                    type: 'album',
                    attributes: {   //  设置 model 属性值
                        title: 'Calgary b/w I Can\'t Make You Love Me/Nick Of Time',
                        artist: 'Bon Iver',
                        songCount: 2
                    },  
                    relationships: {}  //  设置两个 model 的关联关系
                }
            ]
        });
    }
}); 
```

**注意**：`type`属性值必须是模型的属性名字。`attributes`哈希里的属性值与模型里的属性对应。

本篇不是很重要，就简单提一提，如有兴趣请看[Pushing Records Into The Store](https://guides.emberjs.com/v2.5.0/models/pushing-records-into-the-store/)的详细介绍。

博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与 github 代码可能有出入，不过影响不大！），如果你觉得博文对你有点用，请在 github 项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！