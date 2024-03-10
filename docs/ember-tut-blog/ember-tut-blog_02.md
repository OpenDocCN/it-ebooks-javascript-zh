# Ember.js 入门指南——内置 Helper（v2.7.0）

在博文[Ember.js 入门指南之十八工具类的助手](http://blog.ddlisting.com/2016/03/23/ember-js-ru-men-zhi-nan-zhi-shi-ba-gong-ju-lei-de-zhu-shou/)中我们学习了如何编写一个 Helper。一个 Helper 是一个可以用在任何模板的简单方法。Ember 自带了一些可以使你开发模板变简单的 Helper 方法，这些方法允许你更灵活的向其他 Helper 方法或组件中传输数据。

## 使用 Helper 方法动态的获取属性

`{{get}}`方法可以帮助你动态地发送一个变量的值到另一个 Helper 方法或组件中，如果你想输出某些`计算属性`中的一个值的时候,这会非常有用。

```js
{{get address part}} 
```

如果计算属性`part`返回值为`zip`，那这条语句会显示`this.get('address.zip')`的结果；如果返回值为`city`，那便会显示`this.get('address.city')`的结果。

## 内置 Helper 方法的嵌套

上一节我们提到过 Helper 方法允许嵌套使用，内置的方法同样可以。举个栗子,`{{concat}}`可以将参数中的数字动态地发送到组件或 Helper 方法中，然后将其合并为一个独立的参数。

```js
{{get "foo" (concat "item" index)}} 
```

当`index`为`1`时，这条语句会显示`this.get('foo.item1')`的结果；当`index`为其他值时，以此类推。

感谢[@smartepsh](https://github.com/smartepsh)的 pull request，有不妥的地方欢迎指正。

博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与 github 代码可能有出入，不过影响不大！），如果你觉得博文对你有点用，请在 github 项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！