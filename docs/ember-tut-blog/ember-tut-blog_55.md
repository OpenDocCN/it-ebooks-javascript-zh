# 如何构建一个复杂的 Ember.js 项目

来源：[yoember.com](http://yoember.com/)
作者：[Zoltan](http://Zoltan.nz)

**声明**：*本文的转载与翻译是经过作者认可的，再次感谢原作，如有侵权请给我留言，我会删除博文！！*希望本系列教程能帮助更多学习 Ember.js 的初学者。

本系列教材将为读者介绍怎么样使用 Ember.js 构建一个复杂的项目。本教程分为 6 个小部分，通过这 6 篇文章一步步为你讲解怎么使用 Ember.js 构建一个稍微复杂的 Ember.js 项目。

注意：本教程主要是根据[yoember.com](http://yoember.com/)所写，其中加入了自己理解的内容，跟原文相比会有所不同！

有关 Ember.js 的前世今生我就不多做介绍了，请自行查看官方[参考文档](http://emberjs.com)

提醒：如果可以最好是在看一遍官方参考文档之后再看本系列教程，有助于把你所学的零碎的有关 Ember 的知识串联起来，否则，可能你会看得比较痛苦，建议是先把这 6 篇文章认真看一遍下来再自己动手，按照文章提供的源码自己再实践一遍。

**目录**

1.  [环境搭建以及使用 Ember.js 创建第一个静态页面](http://blog.ddlisting.com/2016/03/31/huan-jing-da-jian-yi-ji-shi-yong-ember-jschuang-jian-di-ge-jing-tai-ye-mian/)
2.  [引入计算属性、action、动态内容](http://blog.ddlisting.com/2016/03/31/yin-ru-ji-suan-shu-xing-action-dong-tai-nei-rong/)
3.  [模型，保存数据到数据库](http://blog.ddlisting.com/2016/04/09/mo-xing-bao-cun-shu-ju-dao-shu-ju-ku/)
4.  [发布项目，加入 CRUD 功能](http://blog.ddlisting.com/2016/04/16/fa-bu-xiang-mu-jia-ru-crudgong-neng/)
5.  [从服务器获取数据，引入组件](http://blog.ddlisting.com/2016/04/21/yin-ru-zu-jian/)
6.  [模型高级特性，引入模型关联关系](http://blog.ddlisting.com/2016/04/23/mo-xing-gao-ji-te-xing-yin-ru-mo-xing-guan-lian-guan-xi/)

**项目软件环境**

1.  [NodeJS 5.9.1](https://nodejs.org/en/)
2.  [Ember CLI 2.4.3](http://www.ember-cli.com/user-guide/)
3.  [chrome 插件 Ember Inspector](https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi?hl=en)，如果无法访问那你应该要去 fanqiang 了！！
4.  [Watchman（可选，如果是 Mac 系统推荐安装）](https://facebook.github.io/watchman/)

上述软件请自行安装提供的网址安装，如果安装不成功，或者安装出现错误，请谷歌、百度。如果还是解决不了给我留言获取是去[Ember 社区](http://discuss.emberjs.com/)提问。

**说明：本教程是基于 Ember2.4 而作，请注意与你自己的 Ember.js 版本区别，如果出现不兼容问题请自行升级项目。**

升级教程：[`blog.ddlisting.com/2015/11/24/sheng-ji-emberdao-2-2-0ban-ben/`](http://blog.ddlisting.com/2015/11/24/sheng-ji-emberdao-2-2-0ban-ben/) 本教程是介绍升级到 2.2 版本的，不过同样的道理，只需要修改对应的版本为 2.4 即可。