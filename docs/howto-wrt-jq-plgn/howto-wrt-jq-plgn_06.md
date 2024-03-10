# 使用 grunt 创建项目

## 1.6\. 使用 grunt 创建项目

[grunt](http://gruntjs.com/getting-started)是基于任务的构建工具，和 make，rake，ant，cake，maven，gradle 等是一样的

### 1.6.1\. 前置条件

前置条件需要有 nodejs 和 npm，请确保已安装成功：

```
npm install -g grunt
npm install -g grunt-init
git clone https://github.com/gruntjs/grunt-init-jquery.git ~/.grunt-init/jquery 
grunt-init jquery 
```

如果是万恶的 window 系统，请修改:

```
git clone https://github.com/gruntjs/grunt-init-jquery.git %USERPROFILE%/.grunt-init/jquery 
```

另外如果是 linux 或者 mac，使用-g 安装的时候可能需要 sudo 权限，具体自己看日志

### 1.6.2\. 创建项目

```
➜  jquery_plugin git:(master) ✗ mkdir plugin_grunt     
➜  jquery_plugin git:(master) ✗ cd plugin_grunt 
➜  plugin_grunt git:(master) ✗ grunt-init jquery
Running "init:jquery" (init) task
This task will create one or more files in the current directory, based on the
environment and the answers to a few questions. Note that answering "?" to any
question will show question-specific help and answering "none" to most questions
will leave its value blank.

"jquery" template notes:
Project name should not contain "jquery" or "js" and should be a unique ID not
already in use at plugins.jquery.com. Project title should be a human-readable
title, and doesn't need to contain the word "jQuery", although it may. For
example, a plugin titled "Awesome Plugin" might have the name "awesome-plugin".

For more information, please see the following documentation:

Naming Your Plugin      http://plugins.jquery.com/docs/names/
Publishing Your Plugin  http://plugins.jquery.com/docs/publish/
Package Manifest        http://plugins.jquery.com/docs/package-manifest/

Please answer the following:
[?] Project name (plugin_grunt) i5ting-mobile
[?] Project title (I5ting Mobile) 
[?] Description (The best jQuery plugin ever.) this is a test jq plugin
[?] Version (0.1.0) 
[?] Project git repository (git://github.com/i5ting/How-to-write-jQuery-plugin.git) 
[?] Project homepage (https://github.com/i5ting/How-to-write-jQuery-plugin) 
[?] Project issues tracker (https://github.com/i5ting/How-to-write-jQuery-plugin/issues) 
[?] Licenses (MIT) 
[?] Author name (shiren1118) i5ting
[?] Author email (shiren1118@126.com) i5ting@126.com
[?] Author url (none) 
[?] Required jQuery version (*) 
[?] Do you need to make any changes to the above before continuing? (y/N) 

Writing .gitignore...OK
Writing .jshintrc...OK
Writing CONTRIBUTING.md...OK
Writing Gruntfile.js...OK
Writing README.md...OK
Writing libs/jquery-loader.js...OK
Writing libs/jquery/jquery.js...OK
Writing libs/qunit/qunit.css...OK
Writing libs/qunit/qunit.js...OK
Writing src/.jshintrc...OK
Writing src/i5ting-mobile.js...OK
Writing test/.jshintrc...OK
Writing test/i5ting-mobile.html...OK
Writing test/i5ting-mobile_test.js...OK
Writing LICENSE-MIT...OK
Writing package.json...OK
Writing i5ting-mobile.jquery.json...OK

Initialized from template "jquery".
You should now install project dependencies with npm install. After that, you
may execute project tasks with grunt. For more information about installing
and configuring Grunt, please see the Getting Started guide:

http://gruntjs.com/getting-started

Done, without errors.
➜  plugin_grunt git:(master) ✗ 
```

### 1.6.3\. 安装依赖

切换到 plugin_grunt 根目录，通过下面命令安装 grunt 依赖的包

```
➜  plugin_grunt git:(master) npm install 
```

qunit 依赖 phantomjs，需要翻墙，自备梯子或者 [`i5ting.github.io/How-to-write-jQuery-plugin/node_modules.zip`](http://i5ting.github.io/How-to-write-jQuery-plugin/node_modules.zip)

### 1.6.4\. 测试

grunt 的 task 是在 Gruntfile.js 里定义的，所以看最后的 2 句

```
// Default task.
grunt.registerTask('default', ['jshint', 'qunit', 'clean', 'concat', 'uglify']); 
```

通过上面可以知道 grunt 默认的 tast 包括'jshint', 'qunit', 'clean', 'concat', 'uglify'，也就是说执行 grunt 命令会依次执行这些 task。

任务说明

*   jshint 语法校验
*   qunit 单元测试
*   clean 清理历史
*   concat 合并多个 src 到一个文件中
*   uglify 将 concat 的文件进行混淆

当然你也可以分别运行，比如，运行单元测试：

```
grunt qunit 
```

比如，运行混淆代码

```
grunt uglify 
```

没有 error，通过即可。