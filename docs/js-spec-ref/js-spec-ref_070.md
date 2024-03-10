# 8.4 Grunt：任务自动管理工具

*   安装
*   命令脚本文件 Gruntfile.js
*   Gruntfile.js 实例：grunt-contrib-cssmin 模块
*   常用模块设置
    *   grunt-contrib-jshint
    *   grunt-contrib-concat
    *   grunt-contrib-uglify
    *   grunt-contrib-copy
    *   grunt-contrib-watch
    *   其他模块
    *   参考链接

在 Javascript 的开发过程中，经常会遇到一些重复性的任务，比如合并文件、压缩代码、检查语法错误、将 Sass 代码转成 CSS 代码等等。通常，我们需要使用不同的工具，来完成不同的任务，既重复劳动又非常耗时。Grunt 就是为了解决这个问题而发明的工具，可以帮助我们自动管理和运行各种任务。

简单说，Grunt 是一个自动任务运行器，会按照预先设定的顺序自动运行一系列的任务。这可以简化工作流程，减轻重复性工作带来的负担。

## 安装

Grunt 基于 Node.js，安装之前要先安装 Node.js，然后运行下面的命令。

```js
sudo npm install grunt-cli -g
```

grunt-cli 表示安装的是 grunt 的命令行界面，参数 g 表示全局安装。

Grunt 使用模块结构，除了安装命令行界面以外，还要根据需要安装相应的模块。这些模块应该采用局部安装，因为不同项目可能需要同一个模块的不同版本。

首先，在项目的根目录下，创建一个文本文件 package.json，指定当前项目所需的模块。下面就是一个例子。

```js
{
  "name": "my-project-name",
  "version": "0.1.0",
  "author": "Your Name",
  "devDependencies": {
    "grunt": "0.x.x",
    "grunt-contrib-jshint": "*",
    "grunt-contrib-concat": "~0.1.1",
    "grunt-contrib-uglify": "~0.1.0",
    "grunt-contrib-watch": "~0.1.4"
  }
}
```

上面这个 package.json 文件中，除了注明项目的名称和版本以外，还在 devDependencies 属性中指定了项目依赖的 grunt 模块和版本：grunt 核心模块为最新的 0.x.x 版，jshint 插件为最新版本，concat 插件不低于 0.1.1 版，uglify 插件不低于 0.1.0 版，watch 插件不低于 0.1.4 版。

然后，在项目的根目录下运行下面的命令，这些插件就会被自动安装在 node_modules 子目录。

```js
npm install
```

上面这种方法是针对已有 package.json 的情况。如果想要自动生成 package.json 文件，可以使用 npm init 命令，按照屏幕提示回答所需模块的名称和版本即可。

```js
npm init
```

如果已有的 package.json 文件不包括 Grunt 模块，可以在直接安装 Grunt 模块的时候，加上--save-dev 参数，该模块就会自动被加入 package.json 文件。

```js
npm install <module> --save-dev
```

比如，对应上面 package.json 文件指定的模块，需要运行以下 npm 命令。

```js
npm install grunt --save-dev
npm install grunt-contrib-jshint --save-dev
npm install grunt-contrib-concat --save-dev
npm install grunt-contrib-uglify --save-dev
npm install grunt-contrib-watch --save-dev
```

## 命令脚本文件 Gruntfile.js

模块安装完以后，下一步在项目的根目录下，新建脚本文件 Gruntfile.js。它是 grunt 的配置文件，就好像 package.json 是 npm 的配置文件一样。Gruntfile.js 就是一般的 Node.js 模块的写法。

```js
module.exports = function(grunt) {

  // 配置 Grunt 各种模块的参数
  grunt.initConfig({
    jshint: { /* jshint 的参数 */ },
    concat: { /* concat 的参数 */ },
    uglify: { /* uglify 的参数 */ },
    watch:  { /* watch 的参数 */ }
  });

  // 从 node_modules 目录加载模块文件
  grunt.loadNpmTasks('grunt-contrib-jshint');
  grunt.loadNpmTasks('grunt-contrib-concat');
  grunt.loadNpmTasks('grunt-contrib-uglify');
  grunt.loadNpmTasks('grunt-contrib-watch');

  // 每行 registerTask 定义一个任务
  grunt.registerTask('default', ['jshint', 'concat', 'uglify']);
  grunt.registerTask('check', ['jshint']);

};
```

上面的代码用到了 grunt 代码的三个方法：

*   grunt.initConfig：定义各种模块的参数，每一个成员项对应一个同名模块。

*   grunt.loadNpmTasks：加载完成任务所需的模块。

*   grunt.registerTask：定义具体的任务。第一个参数为任务名，第二个参数是一个数组，表示该任务需要依次使用的模块。default 任务名表示，如果直接输入 grunt 命令，后面不跟任何参数，这时所调用的模块（该例为 jshint，concat 和 uglify）；该例的 check 任务则表示使用 jshint 插件对代码进行语法检查。

上面的代码一共加载了四个模块：jshint（检查语法错误）、concat（合并文件）、uglify（压缩代码）和 watch（自动执行）。接下来，有两种使用方法。

（1）命令行执行某个模块，比如

```js
grunt jshint
```

上面代码表示运行 jshint 模块。

（2）命令行执行某个任务。比如

```js
grunt check
```

上面代码表示运行 check 任务。如果运行成功，就会显示“Done, without errors.”。

如果没有给出任务名，只键入 grunt，就表示执行默认的 default 任务。

## Gruntfile.js 实例：grunt-contrib-cssmin 模块

下面通过 cssmin 模块，演示如何编写 Gruntfile.js 文件。cssmin 模块的作用是最小化 CSS 文件。

首先，在项目的根目录下安装该模块。

```js
npm install grunt-contrib-cssmin --save-dev
```

然后，新建文件 Gruntfile.js。

```js
module.exports = function(grunt) {

  grunt.initConfig({
    cssmin: {
      minify: {
        expand: true,
        cwd: 'css/',
        src: ['*.css', '!*.min.css'],
        dest: 'css/',
        ext: '.min.css'
      },
      combine: {
        files: {
          'css/out.min.css': ['css/part1.min.css', 'css/part2.min.css']
        }
      }
    }
  });

  grunt.loadNpmTasks('grunt-contrib-cssmin');

  grunt.registerTask('default', ['cssmin:minify','cssmin:combine']);

};
```

下面详细解释上面代码中的三个方法，下面一个个来看。

（1）grunt.loadNpmTasks

grunt.loadNpmTasks 方法载入模块文件。

```js
grunt.loadNpmTasks('grunt-contrib-cssmin');
```

你需要使用几个模块，这里就要写几条 grunt.loadNpmTasks 语句，将各个模块一一加载。

如果加载模块很多，这部分会非常冗长。而且，还存在一个问题，就是凡是在这里加载的模块，也同时出现在 package.json 文件中。如果使用 npm 命令卸载模块以后，模块会自动从 package.json 文件中消失，但是必须手动从 Gruntfile.js 文件中清除，这样很不方便，一旦忘记，还会出现运行错误。这里有一个解决办法，就是安装 load-grunt-tasks 模块，然后在 Gruntfile.js 文件中，用下面的语句替代所有的 grunt.loadNpmTasks 语句。

```js
require('load-grunt-tasks')(grunt);
```

这条语句的作用是自动分析 package.json 文件，自动加载所找到的 grunt 模块。

（2）grunt.initConfig

grunt.initConfig 方法用于模块配置，它接受一个对象作为参数。该对象的成员与使用的同名模块一一对应。由于我们要配置的是 cssmin 模块，所以里面有一个 cssmin 成员（属性）。

cssmin（属性）指向一个对象，该对象又包含多个成员。除了一些系统设定的成员（比如 options），其他自定义的成员称为目标（target）。一个模块可以有多个目标（target），上面代码里面，cssmin 模块共有两个目标，一个是“minify”，用于压缩 css 文件；另一个是“combine”，用于将多个 css 文件合并一个文件。

每个目标的具体设置，需要参考该模板的文档。就 cssmin 来讲，minify 目标的参数具体含义如下：

*   expand：如果设为 true，就表示下面文件名的占位符（即*号）都要扩展成具体的文件名。

*   cwd：需要处理的文件（input）所在的目录。

*   src：表示需要处理的文件。如果采用数组形式，数组的每一项就是一个文件名，可以使用通配符。

*   dest：表示处理后的文件名或所在目录。

*   ext：表示处理后的文件后缀名。

除了上面这些参数，还有一些参数也是 grunt 所有模块通用的。

*   filter：一个返回布尔值的函数，用于过滤文件名。只有返回值为 true 的文件，才会被 grunt 处理。

*   dot：是否匹配以点号（.）开头的系统文件。

*   makeBase：如果设置为 true，就只匹配文件路径的最后一部分。比如，a?b 可以匹配/xyz/123/acb，而不匹配/xyz/acb/123。

关于通配符，含义如下：

*   *：匹配任意数量的字符，不包括/。
*   ?：匹配单个字符，不包括/。
*   **：匹配任意数量的字符，包括/。
*   {}：允许使用逗号分隔的列表，表示“or”（或）关系。
*   !：用于模式的开头，表示只返回不匹配的情况。

比如，foo/*.js 匹配 foo 目录下面的文件名以.js 结尾的文件，foo/**/*.js 匹配 foo 目录和它的所有子目录下面的文件名以.js 结尾的文件，!*.css 表示匹配所有后缀名不为“.css”的文件。

使用通配符设置 src 属性的更多例子：

```js
{src: 'foo/th*.js'}grunt-contrib-uglify

{src: 'foo/{a,b}*.js'}

{src: ['foo/a*.js', 'foo/b*.js']}
```

至于 combine 目标，就只有一个 files 参数，表示输出文件是 css 子目录下的 out.min.css，输入文件则是 css 子目录下的 part1.min.css 和 part2.min.css。

files 参数的格式可以是一个对象，也可以是一个数组。

```js
files: {
        'dest/b.js': ['src/bb.js', 'src/bbb.js'],
        'dest/b1.js': ['src/bb1.js', 'src/bbb1.js'],
},

// or

files: [
        {src: ['src/aa.js', 'src/aaa.js'], dest: 'dest/a.js'},
        {src: ['src/aa1.js', 'src/aaa1.js'], dest: 'dest/a1.js'},
],
```

如果 minify 目标和 combine 目标的属性设置有重合的部分，可以另行定义一个与 minify 和 combine 平行的 options 属性。

```js
grunt.initConfig({
    cssmin: {
      options: { /* ... */ },
      minify: { /* ... */ },
      combine: { /* ... */ }
    }
  });
```

（3）grunt.registerTask

grunt.registerTask 方法定义如何调用具体的任务。“default”任务表示如果不提供参数，直接输入 grunt 命令，则先运行“cssmin:minify”，后运行“cssmin:combine”，即先压缩再合并。如果只执行压缩，或者只执行合并，则需要在 grunt 命令后面指明“模块名:目标名”。

```js
grunt # 默认情况下，先压缩后合并

grunt cssmin:minify # 只压缩不合并

grunt css:combine # 只合并不压缩
```

如果不指明目标，只是指明模块，就表示将所有目标依次运行一遍。

```js
grunt cssmin
```

## 常用模块设置

grunt 的[模块](http://gruntjs.com/plugins)已经超过了 2000 个，且还在快速增加。下面是一些常用的模块（按字母排序）。

*   grunt-contrib-clean：删除文件。
*   grunt-contrib-compass：使用 compass 编译 sass 文件。
*   grunt-contrib-concat：合并文件。
*   grunt-contrib-copy：复制文件。
*   grunt-contrib-cssmin：压缩以及合并 CSS 文件。
*   grunt-contrib-imagemin：图像压缩模块。
*   grunt-contrib-jshint：检查 JavaScript 语法。
*   grunt-contrib-uglify：压缩以及合并 JavaScript 文件。
*   grunt-contrib-watch：监视文件变动，做出相应动作。

模块的前缀如果是 grunt-contrib，就表示该模块由 grunt 开发团队维护；如果前缀是 grunt（比如 grunt-pakmanager），就表示由第三方开发者维护。

以下选几个模块，看看它们配置参数的写法，也就是说如何在 grunt.initConfig 方法中配置各个模块。

### grunt-contrib-jshint

jshint 用来检查语法错误，比如分号的使用是否正确、有没有忘记写括号等等。它在 grunt.initConfig 方法里面的配置代码如下。

```js
jshint: {
    options: {
        eqeqeq: true,
        trailing: true
    },
    files: ['Gruntfile.js', 'lib/**/*.js']
},
```

上面代码先指定 jshint 的[检查项目](http://www.jshint.com/docs/options/)，eqeqeq 表示要用严格相等运算符取代相等运算符，trailing 表示行尾不得有多余的空格。然后，指定 files 属性，表示检查目标是 Gruntfile.js 文件，以及 lib 目录的所有子目录下面的 JavaScript 文件。

### grunt-contrib-concat

concat 用来合并同类文件，它不仅可以合并 JavaScript 文件，还可以合并 CSS 文件。

```js
concat: {
  js: {
    src: ['lib/module1.js', 'lib/module2.js', 'lib/plugin.js'],
    dest: 'dist/script.js'
  }
  css: {
    src: ['style/normalize.css', 'style/base.css', 'style/theme.css'],
    dest: 'dist/screen.css'
  }
},
```

js 目标用于合并 JavaScript 文件，css 目标用语合并 CSS 文件。两者的 src 属性指定需要合并的文件（input），dest 属性指定输出的目标文件（output）。

### grunt-contrib-uglify

uglify 模块用来压缩代码，减小文件体积。

```js
uglify: {
  options: {
    banner: bannerContent,
    sourceMapRoot: '../',
    sourceMap: 'distrib/'+name+'.min.js.map',
    sourceMapUrl: name+'.min.js.map'
  },
  target : {
    expand: true,
    cwd: 'js/origin',
    src : '*.js',
    dest : 'js/'
  }
},
```

上面代码中的 options 属性指定压缩后文件的文件头，以及 sourceMap 设置；target 目标指定输入和输出文件。

### grunt-contrib-copy

[copy 模块](https://github.com/gruntjs/grunt-contrib-copy)用于复制文件与目录。

```js
copy: {
  main: {
    src: 'src/*',
    dest: 'dest/',
  },
},
```

上面代码将 src 子目录（只包含它下面的第一层文件和子目录），拷贝到 dest 子目录下面（即 dest/src 目录）。如果要更准确控制拷贝行为，比如只拷贝文件、不拷贝目录、不保持目录结构，可以写成下面这样：

```js
copy: {
  main: {
    expand: true,
    cwd: 'src/',
    src: '**',
    dest: 'dest/',
    flatten: true,
    filter: 'isFile',
  },
},
```

### grunt-contrib-watch

[watch 模块](https://github.com/gruntjs/grunt-contrib-watch)用来在后台运行，监听指定事件，然后自动运行指定的任务。

```js
watch: {
   scripts: {
    files: '**/*.js',
    tasks: 'jshint',
    options: {
      livereload: true,
    },
   },
   css: {
    files: '**/*.sass',
    tasks: ['sass'],
    options: {
      livereload: true,
    },
   },
},
```

设置好上面的代码，打开另一个进程，运行 grunt watch。此后，任何的 js 代码变动，文件保存后就会自动运行 jshint 任务；任何 sass 文件变动，文件保存后就会自动运行 sass 任务。

需要注意的是，这两个任务的 options 参数之中，都设置了 livereload，表示任务运行结束后，自动在浏览器中重载（reload）。这需要在浏览器中安装[livereload 插件](http://livereload.com/)。安装后，livereload 的默认端口为 localhost:35729，但是也可以用 livereload: 1337 的形式重设端口（localhost:1337）。

### 其他模块

下面是另外一些有用的模块。

（1）grunt-contrib-clean

该模块用于删除文件或目录。

```js
clean: {
  build: {
    src: ["path/to/dir/one", "path/to/dir/two"]
  }
}
```

（2）grunt-autoprefixer

该模块用于为 CSS 语句加上浏览器前缀。

```js
autoprefixer: {
  build: {
    expand: true,
    cwd: 'build',
    src: [ '**/*.css' ],
    dest: 'build'
  }
},
```

（3）grunt-contrib-connect

该模块用于在本机运行一个 Web Server。

```js
connect: {
  server: {
    options: {
      port: 4000,
      base: 'build',
      hostname: '*'
    }
  }
}
```

connect 模块会随着 grunt 运行结束而结束，为了使它一直处于运行状态，可以把它放在 watch 模块之前运行。因为 watch 模块需要手动中止，所以 connect 模块也就会一直运行。

（4）grunt-htmlhint

该模块用于检查 HTML 语法。

```js
htmlhint: {
    build: {
        options: {
            'tag-pair': true,
            'tagname-lowercase': true,
            'attr-lowercase': true,
            'attr-value-double-quotes': true,
            'spec-char-escape': true,
            'id-unique': true,
            'head-script-disabled': true,
        },
        src: ['index.html']
    }
}
```

上面代码用于检查 index.html 文件：HTML 标记是否配对、标记名和属性名是否小写、属性值是否包括在双引号之中、特殊字符是否转义、HTML 元素的 id 属性是否为唯一值、head 部分是否没有 script 标记。

（5）grunt-contrib-sass 模块

该模块用于将 SASS 文件转为 CSS 文件。

```js
sass: {
    build: {
        options: {
            style: 'compressed'
        },
        files: {
            'build/css/master.css': 'assets/sass/master.scss'
        }
    }
}
```

上面代码指定输出文件为 build/css/master.css，输入文件为 assets/sass/master.scss。

（6）grunt-markdown

该模块用于将 markdown 文档转为 HTML 文档。

```js
markdown: {
    all: {
      files: [
        {
          expand: true,
          src: '*.md',
          dest: 'docs/html/',
          ext: '.html'
        }
      ],
      options: {
        template: 'templates/index.html',
      }
    }
},
```

上面代码指定将 md 后缀名的文件，转为 docs/html/目录下的 html 文件。template 属性指定转换时采用的模板，模板样式如下。

```js
<!DOCTYPE html>
<html>
<head>
    <title>Document</title>
</head>
<body>

    <div id="main" class="container">
        <%=content%>
    </div>

</body>
</html>
```

## 参考链接

*   Frederic Hemberger, [A build tool for front-end projects](http://frederic-hemberger.de/artikel/grunt-buildtool-for-frontend-projects/)
*   Mária Jurčovičová, [Building a JavaScript Library with Grunt.js](http://flippinawesome.org/2013/07/01/building-a-javascript-library-with-grunt-js/)
*   Ben Briggs，[Speed Up Your Web Development Workflow with Grunt](http://sixrevisions.com/javascript/grunt-tutorial-01/)
*   [Optimizing Images With Grunt](http://blog.grayghostvisuals.com/grunt/image-optimization/)
*   Swapnil Mishra, [Simplifying Chores with Grunt](http://howtonode.org/c4e0f8565942d5e6df45fb78b12d19435543c236/simplifying-chores-with-grunt)
*   AJ ONeal, [Moving to GruntJS](http://blog.coolaj86.com/articles/moving-to-grunt.html)
*   Grunt Documentation, [Configuring tasks](http://gruntjs.com/configuring-tasks)
*   Landon Schropp, [Writing an Awesome Build Script with Grunt](http://www.sitepoint.com/writing-awesome-build-script-grunt/)
*   Mike Cunsolo, [Get Up And Running With Grunt](http://coding.smashingmagazine.com/2013/10/29/get-up-running-grunt/)
*   Matt Bailey, [A Beginner’s Guide to Using Grunt With Magento](http://www.gpmd.co.uk/blog/a-beginners-guide-to-using-grunt-with-magento/)
*   Paul Bakaus, [Supercharging your Gruntfile](http://www.html5rocks.com/en/tutorials/tooling/supercharging-your-gruntfile/)