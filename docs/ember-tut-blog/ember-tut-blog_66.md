# Ember.js 常见问题汇总

本篇为`Ember.js`开发者汇总常见问题，比如浏览器兼容性问题、组件使用等。大部分内容都摘了于[`stackoverflow`](http://stackoverflow.com/search?q=ember.js)和[`emberjs discuss`](http://discuss.emberjs.com/)。

**1\. 浏览器兼容性支持**

[emberjs browser support](http://stackoverflow.com/questions/9873744/ember-js-browser-support)

**2\. 部署服务器后刷新出现 404 问题**

如果你的项目使用命令`ember build --prod`打包后部署到服务器上，进入首页没问题，并且在首页刷新也没问题。但是进入某个子路由后再刷新页面就会出现`404`错误。但是这个错误在开发环境却没有任何问题。这是由于服务器设置的问题。以 nginx 为例。只需要修改：`nginx/conf/vhosts/default.conf`即可*配置文件路由要根据自己 nginx 路径为准*。在此配置文件中加入如下内容：

```js
location / {  
        rewrite ^ /index.html break;
    }

    location /assets/ {
        # do nothing and let nginx handle this as usual
    } 
```

参考网址：[`discuss.emberjs.com/t/how-to-serve-all-routes-on-a-production-server-exactly/6372/2`](http://discuss.emberjs.com/t/how-to-serve-all-routes-on-a-production-server-exactly/6372/2)

还有另外一种更加简单的解决办法：直接在`router.js`设置`location`属性为`hash`即可。更多解释请看官网教程[EMBER-ROUTING MODULE](http://emberjs.com/api/modules/ember-routing.html)

**3\. 如何修改使用命令`ember build --prod`打包后的静态文件的名字，每个名字中会自动生成一个 hash 值一个类似于 md5 加密后的密文。**

参考网址：[`github.com/ember-cli/ember-cli/issues/1418`](https://github.com/ember-cli/ember-cli/issues/1418)