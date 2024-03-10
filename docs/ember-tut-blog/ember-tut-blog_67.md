# 升级 Ember 到 2.2.0 版本

## 版本升级

  目前(*2015-11-24*)使用[Ember CLI](http://www.ember-cli.com/user-guide/)命令安装的`ember`项目默认使用的`ember`版本是 1.13.x。如果你想升级到 2.0 或更高的版本只能手动升级。 下面讲为大家介绍怎么升级到 2.2.0 版本的`ember`。

### 1\. 安装项目

  项目安装仍然使用 Ember CLI，安装成功之后修改配置文件。

```js
ember new ember22test  
cd ember22test 
```

### 2\. 修改 bower.json

  通过手动修改 ember 的版本号。修改之后的 bower.json 如下：

```js
{
  "name": "ember20test",
  "dependencies": {
    "ember": "².2.0",
    "ember-cli-shims": "ember-cli/ember-cli-shims#0.0.3",
    "ember-cli-test-loader": "ember-cli-test-loader#0.1.3",
    "ember-data": "².0.0",
    "ember-load-initializers": "ember-cli/ember-load-initializers#0.1.5",
    "ember-qunit": "0.4.9",
    "ember-qunit-notifications": "0.0.7",
    "ember-resolver": "~0.1.18",
    "jquery": "².1.4",
    "loader.js": "ember-cli/loader.js#3.2.1",
    "qunit": "~1.18.0"
  }
} 
```

  其中需要修改的版本号有三个：`ember`、`ember-data`、`jquery`。

### 3\. 删除原有的文件

  在使用命令更新之前最好先删除原有的文件。 * 删除`bower_components/ember` * 删除`bower_components/ember-data` * 删除`bower_components/jquery`

### 4\. 执行命令更新

  需要重新执行`npm`、`bower`命令下载最新的文件和依赖文件。

```js
npm install  
bower install 
```

### 5\. 验证是否升级成功

  执行命令`ember server`待到项目启动完成，在浏览器执行`http://localhost:4200`。查看浏览器控制台打印的信息。

```js
ember.debug.js:5938DEBUG: -------------------------------  
ember.debug.js:5938DEBUG: Ember      : 2.2.0  
ember.debug.js:5938DEBUG: Ember Data : 2.2.0  
ember.debug.js:5938DEBUG: jQuery     : 2.1.4  
ember.debug.js:5938DEBUG: ------------------------------- 
```

  看到这些打印信息，并且没有报错说明升级成功了。不过你可能会看到这个错误`GET http://localhost:49156/livereload.js?snipver=1 net::ERR_CONNECTION_REFUSED`。 不要紧，不影响使用。如果你觉得看的不爽你可以禁止这个错误，解决办法请参考[stackoverflow](http://stackoverflow.com/questions/25026796/disable-turn-off-livereload-server-in-emberjs-ember-cli)。 或者你可以在启动项目的时候加上一个启动参数：`ember server --live-reload=false`。

  到此升级工作完成了，好好享受`ember2.2.0`吧~~~