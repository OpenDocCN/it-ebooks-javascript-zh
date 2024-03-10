# 8.5 Gulp：任务自动管理工具

Gulp 与 Grunt 一样，也是一个自动任务运行器。它充分借鉴了 Unix 操作系统的管道（pipe）思想，很多人认为，在操作上，它要比 Grunt 简单。

*   安装
*   gulpfile.js
*   gulp 模块的方法
    *   src()
    *   dest()
    *   task()
    *   watch()
    *   gulp-load-plugins 模块
    *   gulp-livereload 模块
    *   参考链接

## 安装

Gulp 需要全局安装，然后再在项目的开发目录中安装为本地模块。先进入项目目录，运行下面的命令。

```
npm install -g gulp

npm install --save-dev gulp
```

除了安装 gulp 以外，不同的任务还需要安装不同的 gulp 插件模块。举例来说，下面代码安装了 gulp-uglify 模块。

```
$ npm install --save-dev gulp-uglify
```

## gulpfile.js

项目根目录中的 gulpfile.js，是 Gulp 的配置文件。下面就是一个典型的 gulpfile.js 文件。

```
var gulp = require('gulp');
var uglify = require('gulp-uglify');

gulp.task('minify', function () {
  gulp.src('js/app.js')
    .pipe(uglify())
    .pipe(gulp.dest('build'))
});
```

上面代码中，gulpfile.js 加载 gulp 和 gulp-uglify 模块之后，使用 gulp 模块的 task 方法指定任务 minify。task 方法有两个参数，第一个是任务名，第二个是任务函数。在任务函数中，使用 gulp 模块的 src 方法，指定所要处理的文件，然后使用 pipe 方法，将上一步的输出转为当前的输入，进行链式处理。

task 方法的回调函数使用了两次 pipe 方法，也就是说做了两种处理。第一种处理是使用 gulp-uglify 模块，压缩源码；第二种处理是使用 gulp 模块的 dest 方法，将上一步的输出写入本地文件，这里是 build.js（代码中省略了后缀名 js）。

执行 minify 任务时，就在项目目录中执行下面命令就可以了。

```
$ gulp minify
```

从上面的例子中可以看到，gulp 充分使用了“管道”思想，就是一个数据流（stream）：src 方法读入文件产生数据流，dest 方法将数据流写入文件，中间是一些中间步骤，每一步都对数据流进行一些处理。

下面是另一个数据流的例子。

```
gulp.task('js', function () {
  return gulp.src('js/*.js')
    .pipe(jshint())
    .pipe(uglify())
    .pipe(concat('app.js'))
    .pipe(gulp.dest('build'));
});
```

上面代码使用 pipe 命令，分别进行 jshint、uglify、concat 三步处理。

## gulp 模块的方法

### src()

gulp 模块的 src 方法，用于产生数据流。它的参数表示所要处理的文件，这些指定的文件会转换成数据流。参数的写法一般有以下几种形式。

*   js/app.js：指定确切的文件名。
*   js/*.js：某个目录所有后缀名为 js 的文件。
*   js/**/*.js：某个目录及其所有子目录中的所有后缀名为 js 的文件。
*   !js/app.js：除了 js/app.js 以外的所有文件。
*   *.+(js|css)：匹配项目根目录下，所有后缀名为 js 或 css 的文件。

src 方法的参数还可以是一个数组，用来指定多个成员。

```
gulp.src(['js/**/*.js', '!js/**/*.min.js'])
```

### dest()

dest 方法将管道的输出写入文件，同时将这些输出继续输出，所以可以依次调用多次 dest 方法，将输出写入多个目录。如果有目录不存在，将会被新建。

```
gulp.src('./client/templates/*.jade')
  .pipe(jade())
  .pipe(gulp.dest('./build/templates'))
  .pipe(minify())
  .pipe(gulp.dest('./build/minified_templates'));
```

dest 方法还可以接受第二个参数，表示配置对象。

```
gulp.dest('build', {
  cwd: './app',
  mode: '0644'
})
```

配置对象有两个字段。cwd 字段指定写入路径的基准目录，默认是当前目录；mode 字段指定写入文件的权限，默认是 0777。

### task()

task 方法用于定义具体的任务。它的第一个参数是任务名，第二个参数是任务函数。下面是一个非常简单的任务函数。

```
gulp.task('greet', function () {
   console.log('Hello world!');
});
```

task 方法还可以指定按顺序运行的一组任务。

```
gulp.task('build', ['css', 'js', 'imgs']);
```

上面代码先指定 build 任务，它由 css、js、imgs 三个任务所组成，task 方法会并发执行这三个任务。注意，由于每个任务都是异步调用，所以没有办法保证 js 任务的开始运行的时间，正是 css 任务运行结束。

如果希望各个任务严格按次序运行，可以把前一个任务写成后一个任务的依赖模块。

```
gulp.task('css', ['greet'], function () {
   // Deal with CSS here
});
```

上面代码表明，css 任务依赖 greet 任务，所以 css 一定会在 greet 运行完成后再运行。

task 方法的回调函数，还可以接受一个函数作为参数，这对执行异步任务非常有用。

```
// 执行 shell 命令
var exec = require('child_process').exec;
gulp.task('jekyll', function(cb) {
  // build Jekyll
  exec('jekyll build', function(err) {
    if (err) return cb(err); // return error
    cb(); // finished task
  });
});
```

如果一个任务的名字为 default，就表明它是“默认任务”，在命令行直接输入 gulp 命令，就会运行该任务。

```
gulp.task('default', function () {
  // Your default task
});

// 或者

gulp.task('default', ['styles', 'jshint', 'watch']);
```

执行的时候，直接使用 gulp，就会运行 styles、jshint、watch 三个任务。

### watch()

watch 方法用于指定需要监视的文件。一旦这些文件发生变动，就运行指定任务。

```
gulp.task('watch', function () {
   gulp.watch('templates/*.tmpl.html', ['build']);
});
```

上面代码指定，一旦 templates 目录中的模板文件发生变化，就运行 build 任务。

watch 方法也可以用回调函数，代替指定的任务。

```
gulp.watch('templates/*.tmpl.html', function (event) {
   console.log('Event type: ' + event.type);
   console.log('Event path: ' + event.path);
});
```

另一种写法是 watch 方法所监控的文件发生变化时（修改、增加、删除文件），会触发 change 事件。可以对 change 事件指定回调函数。

```
var watcher = gulp.watch('templates/*.tmpl.html', ['build']);

watcher.on('change', function (event) {
   console.log('Event type: ' + event.type);
   console.log('Event path: ' + event.path);
});
```

除了 change 事件，watch 方法还可能触发以下事件。

*   end：回调函数运行完毕时触发。
*   error：发生错误时触发。
*   ready：当开始监听文件时触发。
*   nomatch：没有匹配的监听文件时触发。

watcher 对象还包含其他一些方法。

*   watcher.end()：停止 watcher 对象，不会再调用任务或回调函数。
*   watcher.files()：返回 watcher 对象监视的文件。
*   watcher.add(glob)：增加所要监视的文件，它还可以附件第二个参数，表示回调函数。
*   watcher.remove(filepath)：从 watcher 对象中移走一个监视的文件。

## gulp-load-plugins 模块

一般情况下，gulpfile.js 中的模块需要一个个加载。

```
var gulp = require('gulp'),
    jshint = require('gulp-jshint'),
    uglify = require('gulp-uglify'),
    concat = require('gulp-concat');

gulp.task('js', function () {
   return gulp.src('js/*.js')
      .pipe(jshint())
      .pipe(jshint.reporter('default'))
      .pipe(uglify())
      .pipe(concat('app.js'))
      .pipe(gulp.dest('build'));
});
```

上面代码中，除了 gulp 模块以外，还加载另外三个模块。

这种一一加载的写法，比较麻烦。使用 gulp-load-plugins 模块，可以加载 package.json 文件中所有的 gulp 模块。上面的代码用 gulp-load-plugins 模块改写，就是下面这样。

```
var gulp = require('gulp'),
    gulpLoadPlugins = require('gulp-load-plugins'),
    plugins = gulpLoadPlugins();

gulp.task('js', function () {
   return gulp.src('js/*.js')
      .pipe(plugins.jshint())
      .pipe(plugins.jshint.reporter('default'))
      .pipe(plugins.uglify())
      .pipe(plugins.concat('app.js'))
      .pipe(gulp.dest('build'));
});
```

上面代码假设 package.json 文件包含以下内容。

```
{
   "devDependencies": {
      "gulp-concat": "~2.2.0",
      "gulp-uglify": "~0.2.1",
      "gulp-jshint": "~1.5.1",
      "gulp": "~3.5.6"
   }
}
```

## gulp-livereload 模块

gulp-livereload 模块用于自动刷新浏览器，反映出源码的最新变化。它除了模块以外，还需要在浏览器中安装插件，用来配合源码变化。

```
var gulp = require('gulp'),
    less = require('gulp-less'),
    livereload = require('gulp-livereload'),
    watch = require('gulp-watch');

gulp.task('less', function() {
   gulp.src('less/*.less')
      .pipe(watch())
      .pipe(less())
      .pipe(gulp.dest('css'))
      .pipe(livereload());
});
```

上面代码监视 less 文件，一旦编译完成，就自动刷新浏览器。

## 参考链接

*   Callum Macrae, [Building With Gulp](http://www.smashingmagazine.com/2014/06/11/building-with-gulp/)