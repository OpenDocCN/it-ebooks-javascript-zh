# ember.js_ 属性值模糊查询

  如果是 SQL 语句很容易实现模糊查询，但是在 Ember 的应用中怎么实现 model 属性的模糊查询呢。 说难也不难，说简单也简单。只要自定义 filter 方法即可实现。那么少废话放码出来吧！

### 新建 Ember 项目

  执行使用[Ember CLI](http://www.ember-cli.com/user-guide/)命令创建。

```js
ember new search  
cd search  
ember server 
```

  项目启动之后浏览:[`localhost:4200`](http://localhost:4200)，可以看到**Welcome to Ember**，这个欢迎信息，说明项目创建成功。

#### 下面创建测试用的文件

  仍然是使用 Ember CLI 命令创建。

```js
ember g route post  
ember g controller posts  
ember g model post 
```

  使用命令创建 route 的时候会自动也创建出对应的 template 文件。

#### 1，定义 model

```js
import DS from 'ember-data';

export default DS.Model.extend({  
    title: DS.attr('string')
    // body: DS.attr('string'),
    // timestamp: DS.attr('number')
}); 
```

#### 2， 在路由`post`的`model`回调中增加测试数据。

```js
//  app/routes/posts.js

import Ember from 'ember';

export default Ember.Route.extend({

    model: function() {
        //  增加测试数据
        var todoItem = this.store.createRecord('post', 
            { id: 1, title: 'hello world! ' }
        );
        todoItem.save();

        var todoItem2 = this.store.createRecord('post', 
            { id: 3, title: '你好，大家好才是真的好' }
        );
        todoItem2.save();

        var todoItem3 = this.store.createRecord('post', 
            { id: 4, title: 'http://www.ddlisting.com ' }
        );
        todoItem3.save();

        var todoItem4 = this.store.createRecord('post', 
            { id: 5, title: '这个是包含有字母 h 的数据' }
        );
        todoItem4.save();

        var todoItem5 = this.store.createRecord('post', 
            { id: 6, title: '有了天天列表，大事化小，小事化了。' }
        );
        todoItem5.save();

        return this.store.findAll('post');
    }

}); 
```

在`route`增加测试数据后，我直接在模板中遍历显示这些数据，此时仅仅是显示，还没增加查询功能。

```js
<!-- app/templates/posts.hbs -->  
<ul>  
    {{#each model as |item|}}
        <li>{{item.title}}</li>
    {{/each}}
</ul> 
```

页面显示效果如下图：

![图 1-1](http://i12.tietuku.com/3affe712b2a0e6ca.jpg)

下面增加查询功能，首先在 controller 类中增加两个计算属性，一个用来接受查询参数，一个用户显示结果。

```js
//  app/controllers/posts.js

import Ember from 'ember';

export default Ember.Controller.extend({  
    //  定义查询属性 queryParams 是 Ember 固定属性，queryValue 用于接受页面传过来的值
    queryParams: ['queryValue'],
    queryValue: '',

    //  此计算属性用于页面显示数据
    posts: Ember.computed('queryValue', 'model', function() {
      var queryValue = this.get('queryValue');
      var post = this.get('model');
      if (queryValue) {
          return post.filter(function(td) {
                //  通过判断包含的方式实现模糊查询效果
              return td.get('title').indexOf(queryValue) != -1;
          });
      } else {
          return post;
      }
    })

}); 
```

代码功能的解释看注释的内容。

然后修改显示数据模板文件，并增加一个查询输入框。

```js
<!-- app/templates/posts.hbs -->  
{{input type="text" value=queryValue placeholder="search..."}}
<br><hr>  
<ul>  
    <!-- 遍历 posts，而不是 model -->
    {{#each posts as |item|}}
        <li>{{item.title}}</li>
    {{/each}}
</ul> 
```

此的界面显示效果如图 1-2 所示。

![图 1-2](http://i13.tietuku.com/7b12296c8fcbabc8.jpg)

当你在查询框输入要查询的内容时，显示的信息会根据的你输入的内容动态发生变化。 比如输入：h 那么界面显示的数据有：

```js
hello world!  
http://www.ddlisting.com  
这个是包含有字母 h 的数据 
```

这三条数据都是包含有`h`这个字母的，如果输入查询的内容是`http`那么结果就只剩下一条了。

最核心的代码是`td.get('title').indexOf(queryValue) != -1`这句，filter 方法只会数组中匹配条件的数据，如果`indexOf(queryValue) != -1`返回`true`则返回这条数据，否则直接从数组中过滤掉当期这条数据。

不知道你是否看明白了，如有疑问欢迎给我留言。