# 附录 C　初识 AngularJS

AngularJS 是一套符合 MVC 架构的 JavaScript 框架，由 Google 提出与维护。AngularJS 旨在简化 Web App 的开发与测试，它以 HTML 作为视图，并将使用 JavaScript 编写的模型与其绑定在一起。AngularJS 的官方网站为[`angularjs.org/`](https://angularjs.org/)，可以在上面找到最新版的代码和完整的文档。

附录 C 的结构和内容参考了[`www.youtube.com/watch?v=i9MHigUZKEM`](http://www.youtube.com/watch?v=i9MHigUZKEM)。

## C.1　视图

AngularJS 使用 HTML 作为视图，HTML 就相当于 MVC 中的 V。在`<html>`标签中表明`ng-app`属性来声明此 HTML 为 AngularJS 视图：

```js
<html ng-app>
...
</html> 
```

但`ng-app`属性有时会让 HTML5 解释器报错，因为它不是一个标准的属性，如果你希望遵循更加严格的 HTML5 标准，可以将`ng-app`改写成`data-ng-app`，两者的效果是相同的：

```js
<html data-ng-app>
...
</html> 
```

本书为了遵循标准，将全部使用`data-ng-`作为前缀，如果读者在其他地方发现使用了`ng-`前缀不要感到困惑。

在视图中使用两个大括号来绑定变量：

```js
{{ username }} 
```

使用`data-ng-model`属性为元素快捷绑定模型：

```js
<input type="text" data-ng-model="username" placeholder="Username" /> 
```

则一个最简单的 AngularJS 视图就完成了：

```js
<html data-ng-app>
<head>
<script src="angular.min.js"></script>
</head>
<body>
<input type="text" data-ng-model="username" placeholder="Username" /> {{ username }}
</body>
</html> 
```

这个视图呈现一个输入框，在这个输入框中输入的任何字符都会立即显示在输入框的后面。

![enter image description here](img/00063.jpeg)
*一个简单的 AngularJS 视图*

除了直接对数据进行引用外，也可以使用循环来输出一个列表，如：

```js
<div data-ng-init="pets=['Dog', 'Cat', 'Rabbit', 'Parrot']">
<ul>
<li data-ng-repeat="name in pets">{{ name }}</li>
</ul>
</div> 
```

![enter image description here](img/00064.jpeg)
*在视图中循环输出数据*

下面我们来介绍过滤器。过滤器可以进一步限定展示的数据范围及形式，如下面的例子：

```js
<input type="text" data-ng-model="choosenName" />
<div data-ng-init="pets=['Dog', 'Cat', 'Rabbit', 'Parrot']">
<ul>
<li data-ng-repeat="name in pets | filter:choosenName">{{ name }}</li>
</ul>
</div> 
```

这样当在文本框中进行输入时，将只列出包含输入字符的名称：

![enter image description here](img/00065.jpeg)
*过滤器*

还可以通过`orderBy`来指定排列依据，如：

```js
<div data-ng-init="pets=[{name: 'Dog'}, {name: 'Cat'}, {name: 'Rabbit'}, {name: 'Parrot'}]">
<ul>
<li data-ng-repeat="pet in pets | orderBy:'name'">{{ pet.name }}</li>
</ul>
</div> 
```

这样最终的显示顺序将与原始数据的顺序无关，会根据给定的属性重新排列：

![enter image description here](img/00066.jpeg)
*排列顺序*

使用`uppercase`和`lowercase`来限定显示文本大小写格式，如：

```js
<div data-ng-init="pets=['Dog', 'Cat', 'Rabbit', 'Parrot']">
<ul>
<li data-ng-repeat="name in pets">{{ name | uppercase }}</li>
</ul>
</div> 
```

![enter image description here](img/00067.jpeg)
*使用大写格式显示文本*

## C.2　$scope

`$scope`是连接视图与控制器的枢纽。在上一节中我们通过`data-ng-init`指定数据，但如何动态指定数据呢？这就需要`$scope`的帮助。

```js
function petController($scope){
    $scope.pets = [
        {name: 'Dog'},
        {name: 'Cat'},
        {name: 'Rabbit'},
        {name: 'Parrot'}
    ];
} 
```

上面是一个简单的控制器，通过`data-ng-controller`将其与视图绑定：

```js
<div data-ng-controller="petController">
...
</div> 
```

这样在上述`<div>`标签内部就都可以通过`pets`来访问数据了，如：

```js
<div data-ng-controller="petController">
<ul>
<li data-ng-repeat="pet in pets | orderBy:'name'">{{ pet.name }}</li>
</ul>
</div> 
```

值得注意的是，`pets`的作用域仅限于上述`<div>`标签内部，在外部是无法访问到的。

`$scope`可以有任意多的属性，只要你喜欢， 同时相应的变量也可以在相应的区间使用。

## C.3　module 与路由

`module`用来初始化应用，通过`angular.module`方法来创建`module`，如：

```js
angular.module('myApp', []); 
```

对应的视图为：

```js
<html data-ng-app="myApp">
...
</html> 
```

在创建`module`时我们传入了一个空数组`[]`，这个数组用来指定依赖的其他`module`，如：

```js
angular.module('myApp', ['helperModule']); 
```

如果无需依赖其他`module`则只需给出一个空数组`[]`即可。可以看出`module`也方便将功能模块化，提高了复用和维护效率。

通过`controller`方法可以动态添加控制器，如：

```js
var myApp = angular.module('myApp', []);
myApp.controller('petController', function($scope){
    $scope.pets = [
        {name: 'Dog'},
        {name: 'Cat'},
        {name: 'Rabbit'},
        {name: 'Parrot'}
    ];
}); 
```

请对比上述代码与 C.2 中的不同。

为使代码更有可读性，我们可以将上述代码改写成如下形式：

```js
var myApp = angular.module('myApp', []);
controllers = {};
controllers.petController = function($scope){
    $scope.pets = [
        {name: 'Dog'},
        {name: 'Cat'},
        {name: 'Rabbit'},
        {name: 'Parrot'}
    ];
}
myApp.controller(controllers); 
```

通过`config`方法来配置路由，如：

```js
var myApp = angular.module('myApp', []);
myApp.config(function($routeProvider){
    $routeProvider
        .when('/', {
            controller: 'petController',
            templateUrl: 'view1.html'
        })
        .when('/price', {
            controller: 'petController',
            templateUrl: 'view2.html'
        })
        .otherwise({redirectTo: '/'});
}); 
```

在需要动态更改内容的容器中标明`data-ng-view`属性：

```js
<div data-ng-view></div> 
```

下面是完整的视图：

```js
<html data-ng-app="myApp">
<head>
<script src="angular.min.js"></script>
</head>
<div data-ng-view></div>
<a href="#/">Home</a> | <a href="#/price">Price</a>
<script src="module.js"></script>
</html> 
```

需要注意的是，AngularJS 在 1.2 版本以后移除了原生对`ngRoute`的支持，这意味着上面的代码在最新的 AngularJS 中并不工作，解决办法是首先到[`code.angularjs.org/1.2.0rc1/angular-route.js`](https://code.angularjs.org/1.2.0rc1/angular-route.js)下载 angular-route.js，并在视图中引用，然后创建`module`时指定依赖于`ngRoute`模块：

```js
var myApp = angular.module('myApp', [ngRoute]); 
```

下面来编写 module.js 脚本：

```js
var myApp = angular.module('myApp', ['ngRoute']);
controllers = {};
controllers.petController = function($scope){
    $scope.pets = [
        {name: 'Dog', price: 200},
        {name: 'Cat', price: 220},
        {name: 'Rabbit', price: 180},
        {name: 'Parrot', price: 240}
    ];
}
myApp.controller(controllers);
myApp.config(function($routeProvider){
    $routeProvider
        .when('/', {
            controller: 'petController',
            templateUrl: 'view1.html'
        })
        .when('/price', {
            controller: 'petController',
            templateUrl: 'view2.html'
        })
        .otherwise({redirectTo: '/'});
}); 
```

最后编写 view1.html 和 view2.html 两个视图，view1.html 视图：

```js
<ul>
<li data-ng-repeat="pet in pets">{{ pet.name | uppercase }}</li>
</ul> 
```

view2.html 视图：

```js
<ul>
<li data-ng-repeat="pet in pets">{{ pet.name }} - ${{ pet.price }}</li>
</ul> 
```

最后运行的结果如下，默认加载 view1.html 视图：

![enter image description here](img/00068.jpeg)
*默认加载 view1.html 视图*

当点击 Price 链接后加载 view2.html 视图：

![enter image description here](img/00069.jpeg)
*点击 Price 链接后加载 view2.html 视图*

由于动态加载视图是通过`XMLHttpRequest`实现的，所以测试时无法在本地运行¹。

^(1 使用命令行参数`--allow-file-access-from-files`启动 Chrome 浏览器可以解除这一限制。)