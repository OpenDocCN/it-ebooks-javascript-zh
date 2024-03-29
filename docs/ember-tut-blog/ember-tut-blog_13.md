# Ember.js 入门指南之十一 handlebars 显示对象的键

在实际的开发过程中你很有可能需要显示出对象数组的键或者值，如果你需要同时显示出对象的键和值你可以使用`{{#each-in}}`标签。
注意：`each-in`标签是`Ember 2.0`才有的功能，之前的版本是无法使用这个标签的，如果是 2.0 一下的版本会报错：`Uncaught Error: Assertion Failed: A helper named 'each-in' coulad not be found`
准备工作：使用`Ember CLI`生成一个`component`，与此同时会生成一个对应的模板文件。
`ember generate component store-categories`
执行上述命令得到下面的 3 个文件：

```js
app/components/store-categories.js  
app/templates/components/store-categories.hbs  
tests/integration/components/store-categories-test.js 
```

然后在`app/router.js`增加一个路由设置，在`map`方法里添加`this.route('store-categories');`；此时可以直接访问[`localhost:4200/store-categories`](http://localhost:4200/store-categories);

[`guides.emberjs.com/v2.0.0/templates/displaying-the-keys-in-an-object/`](http://guides.emberjs.com/v2.0.0/templates/displaying-the-keys-in-an-object/)

#### 在组件中增加测试数据

```js
// app/components/store-categories.js
import Ember from 'ember';

export default Ember.Component.extend({  
    // https://guides.emberjs.com/v2.4.0/components/the-component-lifecycle/
    willRender: function() {
        //  设置一个对象到属性“categories”上，并且设置到 categories 属性上的对象结构是：key 为字符串，value 为数组
        this.set('categories', {
          'Bourbons': ['Bulleit', 'Four Roses', 'Woodford Reserve'],
          'Ryes': ['WhistlePig', 'High West']
        });
    }
}) 
```

`willRender`方法在组件渲染的时候执行，更多有关组件的介绍会在后面章节——组件中介绍，想了解更多有关组件的介绍会在后面的文章中一一介绍，目前你暂且把组件当做是一个提取出来的公共 HTML 代码。

有了测试数据之后我们怎么去使用`each-in`标签遍历出数组的键呢？

```js
<!-- // app/templates/components/store-categories.hbs -->  
<ul>  
  {{#each-in categories as |category products|}}
    <li>{{category}}
      <ol>
        {{#each products as |product|}}
          <li>{{product}}</li>
        {{/each}}
      </ol>
    </li>
  {{/each-in}}
</ul> 
```

上述模板代码中第一个位置参数`category`就是迭代器的键，第二个位置参数`product`就是键所对应的值。

为了显示效果，在`application.hbs`中调用这个组件，组件的调用非常简单，直接使用`{{组件名}}`方式调用。

```js
<!-- //app/templates/application.hbs -->  
{{store-categories}} 
```

渲染后结果如下图：

![result](img/677ed8d75c91ae01269741360cfb6bee.jpg)

### 重渲染

**`{{each-in}}`表达式不会根据属性值变化而自动更新。**上述示例中，如果你给属性`categories`增加一个元素值，模板上显示的数据不会自动更新。为了演示这个特性在组件中增加一个触发属性变化的按钮，首先需要在组件类`app/components/store-categories.js`中增加一个`action`方法（有关 action 会在后面的章节介绍，暂时把他看做是一个普通的 js 函数），然后在`app/templates/components/store-categories.hbs`中增加一个触发的按钮。

```js
import Ember from 'ember';

export default Ember.Component.extend({  
    // willRender 方法在组件渲染的时候执行，更多有关组件的介绍会在后面章节——组件，中介绍
    willRender: function() {
        //  设置一个对象到属性“categories”上，并且设置到 categories 属性上的对象结构是：key 为字符串，value 为数组
        this.set('categories', {
          'Bourbons': ['Bulleit', 'Four Roses', 'Woodford Reserve'],
          'Ryes': ['WhistlePig', 'High West']
        });
    },
    actions: {
        addCategory: function(category) {
            console.log('清空数据');
            let categories = this.get('categories');
            // console.log(categories);
            categories['Bourbons'] = [];
            //  手动执行重渲染方法更新 dom 元素，但是并没有达到预期效果
            // 还不知道是什么原因
            this.rerender();
        }
    }
}); 
```

```js
<!-- // templates/components/store-categories.hbs -->

<ul>  
  {{#each-in categories as |category products|}}
    <li>{{category}}
      <ol>
        {{#each products as |product|}}
          <li>{{product}}</li>
        {{/each}}
      </ol>
    </li>
  {{/each-in}}
</ul>

<button onclick={{action 'addCategory'}}>点击清空数据</button> 
```

但是很遗憾，即使是手动调用了`rerender`方法也没办法触发重渲染，界面显示的数据并没有发生变化。后续找到原因后再补上！！

### 空数组处理

空数组处理与表达式`{{each}}`一样，同样是判断属性不是`null`、`undefined`、`[]`就显示出数据，否则执行`else`部分。

```js
{{#each-in people as |name person|}}
  Hello, {{name}}! You are {{person.age}} years old.
{{else}}
  Sorry, nobody is here.
{{/each-in}} 
```

可以参考上一篇的`{{each}}`标签测试，这里不再赘述。

博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与 github 代码可能又出入，不过影响不大！），如果你觉得博文对你有点用，请在 github 项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！