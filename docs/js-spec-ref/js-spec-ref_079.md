# 9.3 MVC 框架与 Backbone.js

*   MVC 框架
*   零框架解决方案
*   Backbone 的加载
*   Backbone 的用法
*   Backbone.View
    *   基本用法
    *   initialize 方法
    *   el 属性，$el 属性
    *   tagName 属性，className 属性
    *   template 方法
    *   events 属性
    *   listento 方法
    *   remove 方法
    *   子视图（subview）
    *   Backbone.Router
    *   routes 属性
    *   Backbone.history
    *   Backbone.Model
        *   idAttribute 属性
        *   get 方法
        *   set 方法
        *   on 方法
        *   urlroot 属性
        *   fetch 事件
        *   save 方法
        *   destroy 方法
    *   Backbone.Collection
        *   add 方法，remove 方法
        *   get 方法，set 方法
        *   fetch 方法
    *   Backbone.events
    *   参考链接

## MVC 框架

随着 JavaScript 程序变得越来越复杂，往往需要一个团队协作开发，这时代码的模块化和组织规范就变得异常重要了。MVC 模式就是代码组织的经典模式。

> （……MVC 介绍。）
> 
> （1）Model
> 
> Model 表示数据层，也就是程序需要的数据源，通常使用 JSON 格式表示。
> 
> （2）View
> 
> View 表示表现层，也就是用户界面，对于网页来说，就是用户看到的网页 HTML 代码。
> 
> （3）Controller
> 
> Controller 表示控制层，用来对原始数据（Model）进行加工，传送到 View。

由于网页编程不同于客户端编程，在 MVC 的基础上，JavaScript 社区产生了各种变体框架 MVP（Model-View-Presenter）、MVVM（Model-View-ViewModel）等等，有人就把所有这一类框架的各种模式统称为 MV*。

框架的优点在于合理组织代码、便于团队合作和未来的维护，缺点在于有一定的学习成本，且限制你只能采取它的写法。

## 零框架解决方案

MVC 框架（尤其是大型框架）有一个严重的缺点，就是会产生用户的重度依赖。一旦框架本身出现问题或者停止更新，用户的处境就会很困难，维护和更新成本极高。

ES6 的到来，使得 JavaScript 语言有了原生的模块解决方案。于是，开发者有了另一种选择，就是不使用 MVC 框架，只使用各种单一用途的模块库，组合完成一个项目。下面是可供选择的各种用途的模块列表。

辅助功能库（Helper Libraries）

*   [moment.js](http://momentjs.com/)：日期和时间的标准化
*   [underscore.js](http://underscorejs.org/) / [Lo-Dash](https://lodash.com/)：一系列函数式编程的功能函数

路由库（Routing）

*   [router.js](https://github.com/tildeio/router.js/)：Ember.js 使用的路由库
*   [route-recognizer](https://github.com/tildeio/route-recognizer)：功能全面的路由库
*   [page.js](https://github.com/visionmedia/page.js)：类似 Express 路由的库
*   [director](https://github.com/flatiron/director)：同时支持服务器和浏览器的路由库

Promise 库

*   [RSVP.js](https://github.com/tildeio/rsvp.js)：ES6 兼容的 Promise 库
*   [ES6-Promise](https://github.com/jakearchibald/es6-promise)：RSVP.js 的子集，但是全面兼容 ES6
*   [q](https://github.com/kriskowal/q)：最常用的 Promise 库之一，AngularJS 用了它的精简版
*   [native-promise-only](https://github.com/getify/native-promise-only)：严格符合 ES6 的 Promise 标准，同时兼容老式浏览器

客户端与服务器的通信库

*   [fetch](https://github.com/github/fetch)：实现 window.fetch 功能
*   [qwest](https://github.com/pyrsmk/qwest)：支持 XHR2 和 Promise 的 Ajax 库
*   [jQuery](https://github.com/jquery/jquery)：jQuery 2.0 支持按模块打包，因此可以创建一个纯 Ajax 功能库

动画库（Animation）

*   [cssanimevent](https://github.com/magnetikonline/cssanimevent)：兼容老式浏览器的 CSS3 动画库
*   [Velocity.js](http://julian.com/research/velocity/)：性能优秀的动画库

辅助开发库（Development Assistance）

*   [LogJS](https://github.com/bfattori/LogJS)：轻量级的 logging 功能库
*   [UserTiming.js](https://github.com/nicjansma/usertiming.js)：支持老式浏览器的高精度时间戳库

流程控制和架构（Flow Control/Architecture）

*   [ondomready](https://github.com/tubalmartin/ondomready)：类似 jQuery 的 ready()方法，符合 AMD 规范
*   [script.js](https://github.com/ded/script.js%5D)：异步的脚本加载和依赖关系管理库
*   [async](https://github.com/caolan/async)：浏览器和 node.js 的异步管理工具库
*   [Virtual DOM](https://github.com/Matt-Esch/virtual-dom)：react.js 的一个替代方案，参见[Virtual DOM and diffing algorithm](https://gist.github.com/Raynos/8414846)

数据绑定（Data-binding）

*   Object.observe()：Chrome 已经支持该方法，可以轻易实现双向数据绑定

模板库（Templating）

*   [Mustache](http://mustache.github.io/)：大概是目前使用最广的不含逻辑的模板系统

微框架（Micro-Framework）

某些情况下，可以使用微型框架，作为项目开发的起点。

*   [bottlejs](https://github.com/young-steveo/bottlejs)：提供惰性加载、中间件钩子、装饰器等功能
*   [Stapes.js](http://hay.github.io/stapes/#top)：微型 MVC 框架
*   [soma.js](http://somajs.github.io/somajs/site/)：提供一个松耦合、易测试的架构
*   [knockout](http://knockoutjs.com/)：最流行的微框架之一，主要关注 UI

## Backbone 的加载

```js
<script src="/javascripts/lib/jquery.js"></script>
<script src="/javascripts/lib/underscore.js"></script>
<script src="/javascripts/lib/backbone.js"></script>
<script src="/javascripts/jst.js"></script>

<script src="/javascripts/router.js"></script>
<script src="/javascripts/init.js"></script>
```

## Backbone 的用法

Backbone 是最早的 JavaScript MVC 框架，也是最简化的一个框架。它的设计思想是，只提供最基本的功能，给用户提供最大的自由。这意味着，好的一面是它没有一整套规则，强制你接受，坏的一面是很多功能你必须自己实现。Backbone 的体积相当小，最小化后只有 30 多 KB。

定义一个对象，表示 Web 应用。

```js
var AppName = {
  Models       :{},
  Views        :{},
  Collections  :{},
  Controllers  :{}
};
```

上面代码表示，应用由四部分组成：Model、Collection、Controller 和 View。

定义 Model，表示数据的一个基本单位。

```js
AppName.Models.Person = Backbone.Model.extend({
  urlRoot: "/persons"
});
```

定义 Collection，表示 Model 的集合。

```js
AppName.Collections.Library = Backbone.Collection.extend({
  model: AppName.Models.Book
});
```

上面代码表示，Collection 对象必须有 model 属性，指明由哪一个 model 构成。

定义一个 View。

```js
AppName.Views.Modals.AcceptDecline = Backbone.View.Extend({
  el: ".modal-accept",

  events: {
    "ajax:success .link-accept" :"acceptSuccess",
    "ajax:error   .link-accept" :"acceptError"
  },

  acceptSuccess :function(evt, response) {
    this.$el.modal("hide");
    alert('Cool! Thanks');
  },

  acceptError :function(evt, response) {
    var $modalContent = this.$el.find('.panel-modal');

    $modalContent.append("Something was wrong!");
  }
});
```

View 对象必须有 el 属性，指明当前 View 绑定的 DOM 节点，events 属性指明事件和对应的方法。

定义一个 Controller。

```js
AppName.Controllers.Person = {};
AppName.Controllers.Person.show = function(id) {
  var aMa = new AppName.Models.Person({id: id});

  aMa.updateAge(25);

  aMa.fetch().done(function(){
    var view = new AppName.Views.Show({model: aMa});
  });
};
```

最后，定义路由，启动应用程序。

```js
var Workspace = Backbone.Router.extend({
  routes: {
    "*"                  :"wholeApp",
    "users/:id"          :"usersShow",
    "users/:id/orders/"  :"ordersIndex"
  },

  wholeApp    :AppName.Controller.Application.default,
  usersShow   :AppName.Controller.Users.show,
  ordersIndex :AppName.Controller.Orders.index
});

new Workspace();
Backbone.history.start({pushState: true});
```

## Backbone.View

### 基本用法

Backbone.View 方法用于定义视图类。

```js
var AppView = Backbone.View.extend({
  render: function(){
    $('main').append('<h1>一级标题</h1>');
  }
});
```

上面代码通过 Backbone.View 的 extend 方法，定义了一个视图类 AppView。该类内部有一个 render 方法，用于将视图放置在网页上。

使用的时候，需要先新建视图类的实例，然后通过实例，调用 render 方法，从而让视图在网页上显示。

```js
var appView = new AppView();
appView.render();
```

上面代码新建视图类 AppView 的实例 appView，然后调用 appView.render，网页上就会显示指定的内容。

新建视图实例时，通常需要指定 Model。

```js
var document = new Document({
  model: doc
});
```

### initialize 方法

视图还可以定义 initialize 方法，生成实例的时候，会自动调用该方法对实例初始化。

```js
var AppView = Backbone.View.extend({
  initialize: function(){
    this.render();
  },
  render: function(){
    $('main').append('<h1>一级标题</h1>');
  }
});

var appView = new AppView();
```

上面代码定义了 initialize 方法之后，就省去了生成实例后，手动调用 appView.render()的步骤。

### el 属性，$el 属性

除了直接在 render 方法中，指定“视图”所绑定的网页元素，还可以用视图的 el 属性指定网页元素。

```js
var AppView = Backbone.View.extend({
  el: $('main'),
  render: function(){
    this.$el.append('<h1>一级标题</h1>');
  }
});
```

上面的代码与 render 方法直接绑定网页元素，效果完全一样。上面代码中，除了 el 属性，还是$el 属性，前者代表指定的 DOM 元素，后者则表示该 DOM 元素对应的 jQuery 对象。

### tagName 属性，className 属性

如果不指定 el 属性，也可以通过 tagName 属性和 className 属性指定。

```js
var Document = Backbone.View.extend({
  tagName: "li",
  className: "document",
  render: function() {
   // ...
  }
});
```

### template 方法

视图的 template 属性用来指定网页模板。

```js
var AppView = Backbone.View.extend({
      template: _.template("<h3>Hello <%= who %><h3>"),
});
```

上面代码中，underscore 函数库的 template 函数，接受一个模板字符串作为参数，返回对应的模板函数。有了这个模板函数，只要提供具体的值，就能生成网页代码。

```js
var AppView = Backbone.View.extend({
      el: $('#container'),
      template: _.template("<h3>Hello <%= who %><h3>"),
      initialize: function(){
        this.render();
      },
      render: function(){
        this.$el.html(this.template({who: 'world!'}));
      }
});
```

上面代码的 render 就调用了 template 方法，从而生成具体的网页代码。

实际应用中，一般将模板放在 script 标签中，为了防止浏览器按照 JavaScript 代码解析，type 属性设为 text/template。

```js
<script type="text/template" data-name="templateName">
    <!-- template contents goes here -->
</script>
```

可以使用下面的代码编译模板。

```js
window.templates = {};

var $sources = $('script[type="text/template"]');

$sources.each(function(index, el) {
    var $el = $(el);
    templates[$el.data('name')] = _.template($el.html());
});
```

### events 属性

events 属性用于指定视图的事件及其对应的处理函数。

```js
var Document = Backbone.View.extend({
  events: {
    "click .icon":          "open",
    "click .button.edit":   "openEditDialog",
    "click .button.delete": "destroy"
  }
});
```

上面代码中一个指定了三个 CSS 选择器的单击事件，及其对应的三个处理函数。

### listento 方法

listento 方法用于为特定事件指定回调函数。

```js
var Document = Backbone.View.extend({
  initialize: function() {
    this.listenTo(this.model, "change", this.render);
  }
});
```

上面代码为 model 的 change 事件，指定了回调函数为 render。

### remove 方法

remove 方法用于移除一个视图。

```js
updateView: function() {
  view.remove();
  view.render();
};
```

### 子视图（subview）

在父视图中可以调用子视图。下面就是一种写法。

```js
render : function (){

    this.$el.html(this.template());

    this.child = new Child();

    this.child.appendTo($.('.container-placeholder').render();
}
```

## Backbone.Router

Router 是 Backbone 提供的路由对象，用来将用户请求的网址与后端的处理函数一一对应。

首先，新定义一个 Router 类。

```js
Router = Backbone.Router.extend({
    routes: {
    }
});
```

## routes 属性

Backbone.Router 对象中，最重要的就是 routes 属性。它用来设置路径的处理方法。

routes 属性是一个对象，它的每个成员就代表一个路径处理规则，键名为路径规则，键值为处理方法。

如果键名为空字符串，就代表根路径。

```js
routes: {
        '': 'phonesIndex',
},

phonesIndex: function () {
        new PhonesIndexView({ el: 'section#main' });
}
```

星号代表任意路径，可以设置路径参数，捕获具体的路径值。

```js
var AppRouter = Backbone.Router.extend({
    routes: {
        "*actions": "defaultRoute" 
    }
});

var app_router = new AppRouter;

app_router.on('route:defaultRoute', function(actions) {
    console.log(actions);
})
```

上面代码中，根路径后面的参数，都会被捕获，传入回调函数。

路径规则的写法。

```js
var myrouter = Backbone.Router.extend({
  routes: {
    "help":                 "help",    
    "search/:query":        "search" 
  },

  help: function() {
    ...
  },

  search: function(query) {
    ...
  }
});

routes: {
  "help/:page":         "help",
  "download/*path":     "download",
  "folder/:name":       "openFolder",
  "folder/:name-:mode": "openFolder"
}
router.on("route:help", function(page) {
  ...
});
```

## Backbone.history

设置了 router 以后，就可以启动应用程序。Backbone.history 对象用来监控 url 的变化。

```js
App = new Router();

$(document).ready(function () {
    Backbone.history.start({ pushState: true });
});
```

打开 pushState 方法。如果应用程序不在根目录，就需要指定根目录。

```js
Backbone.history.start({pushState: true, root: "/public/search/"})
```

## Backbone.Model

Model 代表单个的对象实体。

```js
var User = Backbone.Model.extend({
        defaults: {
            name: '',
            email: ''
        }
});

var user = new User();
```

上面代码使用 extend 方法，生成了一个 User 类，它代表 model 的模板。然后，使用 new 命令，生成一个 Model 的实例。defaults 属性用来设置默认属性，上面代码表示 user 对象默认有 name 和 email 两个属性，它们的值都等于空字符串。

生成实例时，可以提供各个属性的具体值。

```js
var user = new User ({
    id: 1,
    name: 'name',
    email: 'name@email.com'
});
```

上面代码在生成实例时，提供了各个属性的具体值。

### idAttribute 属性

Model 实例必须有一个属性，作为区分其他实例的主键。这个属性的名称，由 idAttribute 属性设定，一般是设为 id。

```js
var Music = Backbone.Model.extend({ 
    idAttribute: 'id'
});
```

### get 方法

get 方法用于返回 Model 实例的某个属性的值。

```js
var user = new User({ name: "name", age: 24});
var age = user.get("age"); // 24
var name = user.get("name"); // "name"
```

### set 方法

set 方法用于设置 Model 实例的某个属性的值。

```js
var User = Backbone.Model.extend({
    buy: function(newCarsName){
        this.set({car: newCarsName });
    }
});

var user = new User({name: 'BMW',model:'i8',type:'car'});
user.buy('Porsche');
var car = user.get("car"); // ‘Porsche’
```

### on 方法

on 方法用于监听对象的变化。

```js
var user = new User({name: 'BMW',model:'i8'});

user.on("change:name", function(model){
    var name = model.get("name"); // "Porsche"
    console.log("Changed my car’s name to " + name);
});

user.set({name: 'Porsche'}); 
// Changed my car’s name to Porsche
```

上面代码中的 on 方法用于监听事件，“change:name”表示 name 属性发生变化。

### urlroot 属性

该属性用于指定服务器端对 model 进行操作的路径。

```js
var User = Backbone.Model.extend({
    urlRoot: '/user'
});
```

上面代码指定，服务器对应该 Model 的路径为/user。

### fetch 事件

fetch 事件用于从服务器取出 Model。

```js
var user = new User ({id: 1});
user.fetch({
    success: function (user){
        console.log(user.toJSON());
    }
})
```

上面代码中，user 实例含有 id 属性（值为 1），fetch 方法使用 HTTP 动词 GET，向网址“/user/1”发出请求，从服务器取出该实例。

### save 方法

save 方法用于通知服务器新建或更新 Model。

如果一个 Model 实例不含有 id 属性，则 save 方法将使用 POST 方法新建该实例。

```js
var User = Backbone.Model.extend({
    urlRoot: '/user'
});

var user = new User ();
var userDetails = {
    name: 'name',
    email: 'name@email.com'
};

user.save(userDetails, {
    success: function (user) {
        console.log(user.toJSON());
    }
})
```

上面代码先在类中指定 Model 对应的网址是/user，然后新建一个实例，最后调用 save 方法。它有两个参数，第一个是实例对象的具体属性，第二个参数是一个回调函数对象，设定 success 事件（保存成功）的回调函数。具体来说，save 方法会向/user 发出一个 POST 请求，并将{name: 'name', email: 'name@email.com'}作为数据提供。

如果一个 Model 实例含有 id 属性，则 save 方法将使用 PUT 方法更新该实例。

```js
var user = new User ({
    id: 1,
    name: '张三',
    email: 'name@email.com'
});

user.save({name: '李四'}, {
    success: function (model) {
        console.log(user.toJSON());
    }
}); 
```

上面代码中，对象实例含有 id 属性（值为 1），save 将使用 PUT 方法向网址“/user/1”发出请求，从而更新该实例。

### destroy 方法

destroy 方法用于在服务器上删除该实例。

```js
var user = new User ({
    id: 1,
    name: 'name',
    email: 'name@email.com'
});

user.destroy({
    success: function () {
       console.log('Destroyed');
    }
});
```

上面代码的 destroy 方法，将使用 HTTP 动词 DELETE，向网址“/user/1”发出请求，删除对应的 Model 实例。

## Backbone.Collection

Collection 是同一类 Model 的集合，比如 Model 是动物，Collection 就是动物园；Model 是单个的人，Collection 就是一家公司。

```js
var Song = Backbone.Model.extend({});

var Album = Backbone.Collection.extend({
    model: Song
});
```

上面代码中，Song 是 Model，Album 是 Collection，而且 Album 有一个 model 属性等于 Song，因此表明 Album 是 Song 的集合。

### add 方法，remove 方法

Model 的实例可以直接放入 Collection 的实例，也可以用 add 方法添加。

```js
var song1 = new Song({ id: 1 ,name: "歌名 1", artist: "张三" });
var song2 = new Music ({id: 2,name: "歌名 2", artist: "李四" });
var myAlbum = new Album([song1, song2]);

var song3 = new Music({ id: 3, name: "歌名 3",artist:"赵五" });
myAlbum.add(song3);
```

remove 方法用于从 Collection 实例中移除一个 Model 实例。

```js
myAlbum.remove(1);
```

上面代码表明，remove 方法的参数是 model 实例的 id 属性。

### get 方法，set 方法

get 方法用于从 Collection 中获取指定 id 的 Model 实例。

```js
myAlbum.get(2))
```

### fetch 方法

fetch 方法用于从服务器取出 Collection 数据。

```js
var songs = new Backbone.Collection;
songs.url = '/songs';
songs.fetch();
```

## Backbone.events

```js
var obj = {};
_.extend(obj, Backbone.Events);

obj.on("show-message", function(msg) {
    $('#display').text(msg);
});

obj.trigger("show-message", "Hello World");
```

## 参考链接

*   Andy Walpole, [2015: The End of the Monolithic JavaScript Framework](https://andywalpole.me/#!/blog/142134/2015-the-end-the-monolithic-javascript-framework)
*   Biswadeep Ghosh, [Introduction to Backbone.js](http://www.phloxblog.in/introduction-backbone-js/)