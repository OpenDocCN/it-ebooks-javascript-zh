# 13.4 npm 模块管理器

*   简介
*   npm info
*   npm search
*   npm list
*   npm install
*   语义版本（SemVer）
*   npm update，npm uninstall
*   npm shrinkwrap
*   npm prune
*   npm run
*   npm link
*   npm publish
*   npm version
*   参考链接

## 简介

npm 有两层含义。一层含义是 Node.js 的开放式模块登记和管理系统，网址为[`npmjs.org`](http://npmjs.org)。另一层含义是 Node.js 默认的模块管理器，是一个命令行下的软件，用来安装和管理 node 模块。

npm 不需要单独安装。在安装 node 的时候，会连带一起安装 npm。但是，node 附带的 npm 可能不是最新版本，最好用下面的命令，更新到最新版本。

```
$ npm install npm@latest -g
```

上面的命令之所以最后一个参数是 npm，是因为 npm 本身也是 Node.js 的一个模块。

node 安装完成后，可以用下面的命令，查看一下 npm 的帮助文件。

```
# npm 命令列表
$ npm help

# 各个命令的简单用法
$ npm -l
```

下面的命令分别查看 npm 的版本和配置。

```
$ npm -v
$ npm config list -l
```

## npm info

`npm info`命令可以查看每个模块的具体信息。比如，查看 underscore 模块信息的命令是：

```
$ npm info underscore
{ name: 'underscore',
  description: 'JavaScript\'s functional programming helper library.',
  'dist-tags': { latest: '1.5.2', stable: '1.5.2' },
  repository:
   { type: 'git',
     url: 'git://github.com/jashkenas/underscore.git' },
  homepage: 'http://underscorejs.org',
  main: 'underscore.js',
  version: '1.5.2',
  devDependencies: { phantomjs: '1.9.0-1' },
  licenses:
   { type: 'MIT',
     url: 'https://raw.github.com/jashkenas/underscore/master/LICENSE' },
  files:
   [ 'underscore.js',
     'underscore-min.js',
     'LICENSE' ],
  readmeFilename: 'README.md'}
```

上面命令返回一个 JavaScript 对象，包含了 underscore 模块的详细信息。这个对象的每个成员，都可以直接从 info 命令查询。

```
$ npm info underscore description
JavaScript's functional programming helper library.

$ npm info underscore homepage
http://underscorejs.org

$ npm info underscore version
1.5.2
```

## npm search

向 npm 仓库搜索某个模块，使用 search 命令（可使用正则搜索）。

```
$ npm search <搜索词>
```

如果不加搜索词，`npm search`默认返回 npm 仓库的所有模块。

## npm list

`npm list`命令列出当前目录安装的所有模块。如果使用 global 参数，就是列出全局安装的模块。

```
$ npm list
$ npm -global list
```

## npm install

Node 模块采用`npm install`命令安装。每个模块可以“全局安装”，也可以“本地安装”。两者的差异是模块的安装位置，以及调用方法。

“全局安装”指的是将一个模块直接下载到 Node 的安装目录中，各个项目都可以调用。“本地安装”指的是将一个模块下载到当前目录的 node_modules 子目录，然后只有在当前目录和它的子目录之中，才能调用这个模块。一般来说，全局安装只适用于工具模块，比如 npm 和 grunt。

默认情况下，`npm install`命令是“本地安装”某个模块。

```
$ npm install <package name>
```

npm 也支持直接输入 github 地址。

```
$ npm install git://github.com/package/path.git
$ npm install git://github.com/package/path.git#0.1.0
```

运行上面命令后，模块文件将下载到当前目录的`node_modules`子目录。

使用 global 参数，可以“全局安装”某个模块。global 参数可以被简化成 g 参数。

```
$ sudo npm install -global [package name]
$ sudo npm install -g [package name]
```

install 命令总是安装模块的最新版本，如果要安装模块的特定版本，可以在模块名后面加上@和版本号。

```
$ npm install sax@latest
$ npm install sax@0.1.1
$ npm install sax@">=0.1.0 <0.2.0"
```

install 命令可以使用不同参数，指定所安装的模块属于哪一种性质的依赖关系，即出现在 packages.json 文件的哪一项中。

*   --save：模块名将被添加到 dependencies，可以简化为参数`-S`。
*   --save-dev: 模块名将被添加到 devDependencies，可以简化为参数`-D`。

```
$ npm install sax --save
$ npm install node-tap --save-dev
# 或者
$ npm install sax -S
$ npm install node-tap -D
```

`npm install`默认会安装 dependencies 字段和 devDependencies 字段中的所有模块，如果使用 production 参数，可以只安装 dependencies 字段的模块。

```
$ npm install --production
# 或者
$ NODE_ENV=production npm install
```

一旦安装了某个模块，就可以在代码中用 require 命令调用这个模块。

```
var backbone = require('backbone')

console.log(backbone.VERSION)
```

## 语义版本（SemVer）

npm 采用”语义版本“管理软件包。所谓语义版本，就是指版本号为`a.b.c`的形式，其中 a 是大版本号，b 是小版本号，c 是补丁号。

一个软件发布的时候，默认就是 1.0.0 版。如果以后发布补丁，就增加最后一位数字，比如 1.0.1；如果增加新功能，且不影响原有的功能，就增加中间的数字（即小版本号），比如 1.1.0；如果引入的变化，破坏了向后兼容性，就增加第一位数字（即大版本号），比如 2.0.0。

npm 允许使用特殊符号，指定所要使用的版本范围，假定当前版本是 1.0.4。

*   只接受补丁包：1.0 或者 1.0.x 或者 ~1.0.4
*   只接受小版本和补丁包：1 或者 1.x 或者 ¹.0.4
*   接受所有更新：* or x

对于~和^，要注意区分。前者表示接受当前小版本（如果省略小版本号，则是当前大版本）的最新补丁包，后者表示接受当前大版本的最新小版本和最新补丁包。

```
~2.2.1 // 接受 2.2.1，不接受 2.3.0
².2.1 // 接受 2.2.1 和 2.3.0

~2.2 // 接受 2.2.0 和 2.2.1，不接受 2.3.0
².2 // 接受 2.2.0、2.2.1 和 2.3.0

~2 // 接受 2.0.0、2.1.0、2.2.0、2.2.1 和 2.3.0
² // 接受 2.0.0、2.1.0、2.2.0、2.2.1 和 2.3.0
```

还可以使用数学运算符（比如>, = or <=等），指定版本范围。

```
>2.1
1.0.0 - 1.2.0
>1.0.0-alpha
>=1.0.0-rc.0 <1.0.1
² <2.2 || > 2.3
```

注意，如果使用连字号，它的两端必须有空格。如果不带空格，会被 npm 理解成预发布的 tag，比如 1.0.0-rc.1。

## npm update，npm uninstall

npm update 命令可以升级本地安装的模块。

```
npm update [package name]
```

加上 global 参数，可以升级全局安装的模块。

```
npm update -global [package name]
```

npm uninstall 命令，删除本地安装的模块。

```
npm uninstall [package name]
```

加上 global 参数，可以删除全局安装的模块。

```
sudo npm uninstall [package name] -global
```

## npm shrinkwrap

对于一个项目来说，通常不会写死依赖的 npm 模块的版本。比如，开发时使用某个模块的版本是 1.0，那么等到用户安装时，如果该模块升级到 1.1，往往就会安装 1.1。

但是，对于开发者来说，有时最好锁定所有依赖的版本，防止模块升级带来意想不到的问题。但是，由于模块自己还依赖别的模块，这一点往往很难做到。举例来说，项目依赖 A 模块，A 模块依赖 B 模块。即使写死 A 模块的版本，但是 B 模块升级依然可能导致不兼容。

`npm shrinkwrap`命令就是用来彻底锁定所有模块的版本。

```
$ npm shrinkwrap
```

运行上面这个命令以后，会在项目目录下生成一个 npm-shrinkwrap.json 文件，里面包含当前项目用到的所有依赖（包括依赖的依赖，以此类推），以及它们的准确版本，也就是当前正在使用的版本。

只要存在`npm-shrinkwrap.json`文件，下一次用户使用`npm install`命令安装依赖的时候，就会安装所有版本完全相同的模块。

如果执行`npm shrinkwrap`的时候，加上参数 dev，还可以记录 devDependencies 字段中模块的准确版本。

```
$ npm shrinkwrap --dev
```

## npm prune

`npm prune`命令与`npm shrinkwrap`配套使用。

使用`npm shrinkwrap`的时候，有时可能存在某个已经安装的模块不在 dependencies 字段内的情况，这时`npm shrinkwrap` 就会报错。

`npm prune`命令可以移除所有不在 dependencies 字段内的模块。所有指定模块名，则移除单个模块。

```
$ npm prune
$ npm package <package name>
```

## npm run

package.json 文件有一项 scripts，用于指定脚本命令，供 npm 直接调用。

```
{
  "name": "myproject",
  "devDependencies": {
    "jshint": "latest",
    "browserify": "latest",
    "mocha": "latest"
  },
  "scripts": {
    "lint": "jshint **.js",
    "test": "mocha test/"
  }
}
```

上面代码中，scripts 指定了两项命令 lint 和 test。命令行输入`npm run lint`，就会执行`jshint **.js`，输入`npm run test`，就会执行`mocha test/`。npm 内置了两个命令简写，`npm test`等同于执行`npm run lint`，`npm start`等同于执行`npm run start`。

`npm run`会创建一个 shell，执行指定的命令，并将`node_modules/.bin`加入 PATH 变量，这意味着本地模块可以直接运行。也就是说，`npm run lint`直接运行`jshint **.js`即可，而不用`./node_modules/.bin/jshint **.js`。

如果直接运行`npm run`不给出任何参数，就会列出 scripts 属性下所有命令。

```
Available scripts in the user-service package:
  lint
     jshint **.js
  test
    mocha test/
```

下面是另一个 package.json 文件的例子。

```
"scripts": {
  "watch": "watchify client/main.js -o public/app.js -v",
  "build": "browserify client/main.js -o public/app.js",
  "start": "npm run watch & nodemon server.js",
  "test": "node test/all.js"
},
```

上面代码在 scripts 项，定义了四个别名，每个别名都有对应的脚本命令。

```
npm run watch
npm run build
npm run start
npm run test
```

其中，start 和 test 属于特殊命令，可以省略 run。

```
npm start
npm test
```

如果希望一个操作的输出，是另一个操作的输入，可以借用 Linux 系统的管道命令，将两个操作连在一起。

```
"build-js": "browserify browser/main.js | uglifyjs -mc > static/bundle.js"
```

但是，更方便的写法是引用其他`npm run`命令。

```
"build": "npm run build-js && npm run build-css"
```

上面的写法是先运行`npm run build-js`，然后再运行`npm run build-css`，两个命令中间用`&&`连接。如果希望两个命令同时平行执行，它们中间可以用`&`连接。

下面是一个流操作的例子。

```
"devDependencies": {
  "autoprefixer": "latest",
  "cssmin": "latest"
},

"scripts": {
  "build:css": "autoprefixer -b 'last 2 versions' < assets/styles/main.css | cssmin > dist/main.css"
}
```

写在 scripts 属性中的命令，也可以在`node_modules/.bin`目录中直接写成 bash 脚本。

```
#!/bin/bash

cd site/main;
browserify browser/main.js | uglifyjs -mc > static/bundle.js
```

假定上面的脚本文件名为 build.sh，并且权限为可执行，就可以在 scripts 属性中引用该文件。

```
"build-js": "bin/build.sh"
```

`npm run`为每条命令提供了 pre 和 post 两个钩子（hook）。以`npm run lint`为例，执行这条命令之前，npm 会先查看有没有定义 prelint 和 postlint 两个钩子，如果有的话，就会先执行`npm run prelint`，然后执行`npm run lint`，最后执行`npm run postlint`。所有命令都是这样，包括`npm test`（即实际存在`npm run pretest`、`npm run test`、`npm run posttest`三条命令）。如果执行过程出错，就不会执行排在后面的命令，即如果 pretest 命令执行出错，就不会接着执行 test 和 posttest 命令。不能在 pre 命令之前再加 pre，即 prepretest 命令不起作用。另外，还可以为一些内部命令指定 pre 和 post 的钩子：install、uninstall、publish、update。

```
"scripts": {
  "lint": "jshint **.js",
  "build": "browserify index.js > myproject.min.js",
  "test": "mocha test/",

  "prepublish": "npm run build # also runs npm run prebuild",
  "prebuild": "npm run test # also runs npm run pretest",
  "pretest": "npm run lint"
}
```

`npm run`命令还可以添加参数。

```
"scripts": {
  "test": "mocha test/"
}
```

上面代码指定`npm test`，实际运行`mocha test/`。可以在`npm test`命令后面加上参数，比如`npm run test -- anothertest.js`，实际运行的是`mocha test/ anothertest.js`。

## npm link

一般来说，每个项目都会在项目目录内，安装所需的模块文件。也就是说，各个模块是局部安装。但是有时候，我们希望模块是一个符号链接，连到外部文件，这时候就需要用到 npm link 命令。

为了理解 npm link，请设想这样一个场景。你开发了一个模块 myModule，目录为 src/myModule，你自己的项目 myProject 要用到这个模块，项目目录为 src/myProject。每一次，你更新 myModule，就要用`npm publish`命令发布，然后切换到项目目录，使用`npm update`更新模块。这样显然很不方便，如果我们可以从项目目录建立一个符号链接，直接连到模块目录，就省去了中间步骤，项目可以直接使用最新版的模块。

先在模块目录（src/myModule）下运行 npm link 命令。

```
src/myModule$ npm link
```

上面的命令会在 npm 的全局模块目录内（比如/usr/local/lib/node_modules/），生成一个符号链接文件，该文件的名字就是 package.json 文件中指定的文件名。

```
/usr/local/lib/node_modules/myModule -> src/myModule
```

然后，切换到你需要放置该模块的项目目录，再次运行 npm link 命令，并指定模块名。

```
src/myProject$ npm link myModule
```

上面命令等同于生成了本地模块的符号链接。

```
src/myProject/node_modules/myModule -> /usr/local/lib/node_modules/myModule
```

然后，就可以在你的项目中，加载该模块了。

```
var myModule = require('myModule');
```

这样一来，myModule 的任何变化，都可以直接在 myProject 中调用。但是，同时也出现了风险，任何在 myProject 目录中对 myModule 的修改，都会反映到模块的源码中。

npm link 命令有一个简写形式，显示连接模块的本地目录。

```
$ src/myProject$ npm link ../myModule
```

上面的命令等同于下面几条命令。

```
$ src/myProject$ cd ../myModule
$ src/myModule$ npm link
$ src/myModule$ cd ../myProject
$ src/myProject$ npm link myModule
```

如果你的项目不再需要该模块，可以在项目目录内使用 npm unlink 命令，删除符号链接。

```
src/myProject$ npm unlink myModule
```

一般来说，npm 公共模块都安装在系统目录（比如/usr/local/lib/），普通用户没有写入权限，需要用到 sudo 命令。这不是很方便，我们可以在没有 root 的情况下，用好 npm。

首先在主目录下新建配置文件.npmrc，然后在该文件中将 prefix 变量定义到主目录下面。

```
prefix = /home/yourUsername/npm
```

然后在主目录下新建 npm 子目录。

```
$ mkdir ~/npm
```

此后，全局安装的模块都会安装在这个子目录中，npm 也会到`~/npm/bin`目录去寻找命令。因此，`npm link`就不再需要 root 权限了。

最后，将这个路径在.bash_profile 文件（或.bashrc 文件）中加入 PATH 变量。

```
export PATH=~/npm/bin:$PATH
```

## npm publish

在发布你的模块之前，需要先设定个人信息。

```
$ npm set init.author.name “张三”
$ npm set init.author.email “zhangsan@email.com”
$ npm set init.author.url “http://your.url.com"
```

这些信息会存放在用户主目录的`~/.npmrc`文件，使得用户不用每个项目都输入。如果遇到某个模块的作者信息，与全局设置不同，可以在项目目录中，单独设置一次上面这些命令。

然后，向 npm 系统申请用户名。

```
$ npm adduser
```

运行上面的命令之后，屏幕上会提示输入用户名，然后是输入 Email 地址和密码。

如果已经注册过，就使用下面的命令登录。

```
$ npm login
```

最后，使用 npm publish 命令发布。

```
$ npm publish
```

## npm version

`npm version`命令用来修改项目的版本号。当你完成代码修改，要发布新版本的时候，就用这个命令更新一下软件的版本。

```
$ npm version <update_type> -m "<message>"
```

`npm version`命令的 update_type 参数有三个选项：patch、minor、major。

*   `npm version patch`增加一位补丁号（比如 1.1.1 -> 1.1.2）
*   `npm version minor`增加一位小版本号（比如 1.1.1 -> 1.2.0）
*   `npm version major`增加一位大版本号（比如 1.1.1 -> 2.0.0）。

下面是一个例子。

```
$ npm version patch -m "Version %s - xxxxxx"
```

上面命令的%s 表示新的版本号。

除了增加版本号，`npm version`命令还会为这次修改，新增一个 git commit 记录，以及一个新的 git tag。

由于更新 npm 网站的唯一方法，就是发布一个新的版本。因此，除了第一次发布，这个命令与`npm publish`几乎是配套的，先使用它，再使用`npm publish`。

## 参考链接

*   James Halliday, [task automation with npm run](http://substack.net/task_automation_with_npm_run): npm run 命令（package.json 文件的 script 属性）的用法
*   Keith Cirkel, [How to Use npm as a Build Tool](http://blog.keithcirkel.co.uk/how-to-use-npm-as-a-build-tool/)
*   justjs, [npm link: developing your own npm modules without tears](http://justjs.com/posts/npm-link-developing-your-own-npm-modules-without-tears)