# 8.9 JavaScript 程序测试

*   为什么要写测试？
*   测试的类型
    *   单元测试
    *   集成测试
    *   功能测试
    *   开发模式
        *   TDD
        *   BDD
    *   断言
    *   Mocha.js
    *   WebDriver
        *   定制测试环境
        *   操作浏览器的方法
        *   网页元素的定位
        *   网页元素的方法
        *   页面跳转的方法
        *   cookie 的方法
        *   浏览器窗口的方法
        *   弹出窗口
        *   鼠标和键盘的方法

## 为什么要写测试？

Web 应用程序越来越复杂，这意味着有更多的可能出错。测试是帮助我们提高代码质量、降低错误的最好方法和工具之一。

*   测试可以确保得到预期结果。
*   加快开发速度。
*   方便维护。
*   提供用法的文档。

## 测试的类型

### 单元测试

单元测试（unit testing）指的是以软件的单元为单位，对软件进行测试。单元（unit）可以是一个函数，也可以是一个模块或组件。它的基本特征就是，只要输入不变，必定返回同样的输出。

单元测试这个词，本身就暗示，软件应该以模块化结构存在。每个模块的运作，是独立于其他模块的。一个软件越容易写单元测试，往往暗示着它的模块化结构越好，各模块之间的耦合就越弱；越难写单元测试，或者每次单元测试，不得不模拟大量的外部条件，很可能暗示软件的模块化结构越差，模块之间存在较强的耦合。

单元测试的要求是，每个模块都必须有单元测试，而软件由模块组成。

单元测试通常采取断言（assertion）的形式，也就是测试某个功能的返回结果，是否与预期结果一致。如果与预期不一致，就表示测试失败。

单元测试是函数正常工作、不出错的最基本、最有效的方法之一。 每一个单元测试发出一个特定的输入到所要测试的函数，看看函数是否返回预期的输出，或者采取了预期的行动。单元测试证明了所测试的代码行为符合预期。

单元测试有助于代码的模块化，因此有助于长期的重用。因为有了测试，你就知道代码是可靠的，可以按照预期运行。从这个角度说，测试可以节省开发时间。单元测试的另一个好处是，有了测试，就等于就有了代码功能的文档，有助于其他开发者了解代码的意图。

单元测试应该避免依赖性问题，比如不存取数据库、不访问网络等等，而是使用工具虚拟出运行环境。这种虚拟使得测试成本最小化，不用花大力气搭建各种测试环境。

单元测试的步骤

*   准备所有的测试条件
*   调用（触发）所要测试的函数
*   验证运行结果是否正确
*   还原被修改的记录

### 集成测试

集成测试（Integration test）指的是多个部分在一起测试，比如在一个测试数据库上，测试数据库连接模块。

### 功能测试

功能测试（Functional test）指的是，自动测试整个应用程序的某个功能，比如使用 Selenium 工具自动打开浏览器运行程序。

## 开发模式

### TDD

TDD 是“测试驱动的开发”（Test-Driven Development）的简称，指的是先写好测试，然后再根据测试完成开发。使用这种开发方式，会有很高的测试覆盖率。

TDD 的开发步骤如下。

*   先写一个测试。
*   写出最小数量的代码，使其能够通过测试。
*   优化代码。
*   重复前面三步。

TDD 开发的测试覆盖率通常在 90%以上，这意味着维护代码和新增特性会非常容易。因为测试保证了你可以信任这些代码，修改它们不会破坏其他代码的运行。

TDD 接口提供以下四个方法。

*   suite()
*   test()
*   setup()
*   teardown()

下面代码是测试计数器是否加 1。

```js
suite('Counter', function() {
  test('tick increases count to 1', function() {
    var counter = new Counter();
    counter.tick();
    assert.equal(counter.count, 1);
  });
});
```

### BDD

BDD 是“行为驱动的开发”（Behavior-Driven Development）的简称，指的是写出优秀测试的最佳实践的总称。

BDD 认为，不应该针对代码的实现细节写测试，而是要针对行为写测试。BDD 测试的是行为，即软件应该怎样运行。

BDD 接口提供以下四个方法。

*   describe()
*   it()
*   before()
*   after()
*   beforeEach()
*   afterEach()

下面是测试计数器是否加 1 的 BDD 写法。

```js
describe('Counter', function() {
  it('should increase count by 1 after calling tick', function() {
    var counter = new Counter();
    var expectedCount = counter.count + 1;
    counter.tick();
    assert.equal(counter.count, expectedCount);
  });
});
```

## 断言

断言是判断实际值与预期值是否相等的工具。

断言有 assert、expext、should 三种风格，或者称为三种写法。

```js
// assert 风格
assert.equal(event.detail.item, '(item).);

// expect 风格
expect(event.detail.item).to.equal('(item)');

// should 风格
event.detail.item.should.equal('(item)');
```

Chai.js 是一个很流行的断言库，同时支持上面三种风格。

（1） assert 风格

```js
var assert = require('chai').assert;
var foo = 'bar';
var beverages = { tea: [ 'chai', 'matcha', 'oolong' ] };

assert.typeOf(foo, 'string', 'foo is a string');
assert.equal(foo, 'bar', 'foo equal `bar`');
assert.lengthOf(foo, 3, 'foo`s value has a length of 3');
assert.lengthOf(beverages.tea, 3, 'beverages has 3 types of tea');
```

上面代码中，assert 方法的最后一个参数是错误提示信息，只有测试没有通过时，才会显示。

（2）expect 风格

```js
var expect = require('chai').expect;
var foo = 'bar';
var beverages = { tea: [ 'chai', 'matcha', 'oolong' ] };

expect(foo).to.be.a('string');
expect(foo).to.equal('bar');
expect(foo).to.have.length(3);
expect(beverages).to.have.property('tea').with.length(3);
```

（3）should 风格

```js
var should = require('chai').should();
var foo = 'bar';
var beverages = { tea: [ 'chai', 'matcha', 'oolong' ] };

foo.should.be.a('string');
foo.should.equal('bar');
foo.should.have.length(3);
beverages.should.have.property('tea').with.length(3);
```

## Mocha.js

Mocha 是一个测试框架，也就是运行测试的工具。它使用下面的命令安装。

```js
$ npm install -g mocha
```

Mocha 自身不带断言库，所以还需要安装一个断言库，这里选用 Chai.js。

```js
$ npm install -g chai
```

Mocha 默认执行 test 目录的脚本文件，所以我们将所有测试脚本放在 test 子目录。

Mocha 允许指定测试脚本文件，可以使用通配符，同时指定多个文件。

```js
$ mocha --reporter spec spec/{my,awesome}.js
$ mocha --ui tdd test/unit/*.js etc
```

如果希望测试非存放于 test 子目录的测试用例，可以在 test 子目录中新建 Mocha 的配置文件 mocha.opts。在该文件中写入以下内容。

```js
server-tests
```

上面代码指定 Mocha 默认测试 server-tests 子目录的测试脚本。

```js
server-tests
--recursive
```

上面代码中，--recursive 参数指定同时运行子目录中的测试用例脚本。

report 参数用于指定 Mocha 的报告格式。

```js
$ mocha --reporter spec server-test/*.js
```

上面代码指定报告格式是 spec。

grep 参数用于搜索测试用例的名称（即 it 方法的第一个参数），然后只执行匹配的测试用例。

```js
$ mocha --reporter spec --grep "Fnord:" server-test/*.js
```

上面代码只测试名称中包含“Fnord：”的测试用例。

invert 参数表示只运行不符合条件的测试脚本。

```js
$ mocha --grep auth --invert
```

测试脚本中，describe 方法和 it 方法都允许调用 only 方法，表示只运行某个测试套件或测试用例。

```js
describe("using only", function() {
  it.only("this is the only test to be run", function() {

  });

  it("this is not run", function() {

  });
});
```

上面代码中，只有第一个测试用例会运行。

describe 方法和 it 方法还可以调用 skip 方法，表示跳过指定的测试套件或测试用例。

```js
describe("using only", function() {
  it.skip("this is the only test to be run", function() {

  });

  it("this is not run", function() {

  });
});
```

上面代码中，只有第二个测试用例会执行。

如果测试用例包含异步操作，可以 done 方法显式指定测试用例的运行结束时间。

```js
it('logs a', function(done) {
  var f = function(){
    console.log('logs a');
    done();
  };
  setTimeout(f, 500);
});
```

上面代码中，正常情况下，函数 f 还没有执行，Mocha 就已经结束运行了。为了保证 Mocha 等到测试用例跑完再结束运行，可以手动调用 done 方法

## WebDriver

WebDriver 是一个浏览器的自动化框架。它在各种浏览器的基础上，提供一个统一接口，将接收到的指令转为浏览器的原生指令，驱动浏览器。

WebDriver 由 Selenium 项目演变而来。Selenium 是一个测试自动化框架，它的 1.0 版叫做 Selenium RC，通过一个代理服务器，将测试脚本转为 JavaScript 脚本，注入不同的浏览器，再由浏览器执行这些脚本后返回结果。WebDriver 就是 Selenium 2.0，它对每个浏览器提供一个驱动，测试脚本通过驱动转换为浏览器原生命令，在浏览器中执行。

### 定制测试环境

DesiredCapabilities 对象用于定制测试环境。

*   定制 DesiredCapabilities 对象的各个属性
*   创建 DesiredCapabilities 实例
*   将 DesiredCapabilities 实例作为参数，新建一个 WebDriver 实例

### 操作浏览器的方法

WebDriver 提供以下方法操作浏览器。

close()：退出或关闭当前浏览器窗口。

```js
driver.close();
```

quit()：关闭所有浏览器窗口，中止当前浏览器 driver 和 session。

```js
driver.quit();
```

getTitle()：返回当前网页的标题。

```js
driver.getTitle();
```

getCurrentUrl()：返回当前网页的网址。

```js
driver.getCurrentUrl();
```

getPageSource()：返回当前网页的源码。

```js
// 断言是否含有指定文本
assert(driver.getPageSource().contains("Hello World"),
  "预期含有文本 Hello World");
```

click()：模拟鼠标点击。

```js
// 例一
driver.findElement(By.locatorType("path"))
  .click();

// 例二
driver.get("https://www.google.com");
driver.findElement(By.name("q"))
  .sendKeys("webDriver");
driver.findElement(By.id("sblsbb"))
  .click();
```

clear()：清空文本输入框。

```js
// 例一
driver.findElement(By.locatorType("path")).clear();

// 例二
driver.get("https://www.google.com");
driver.findElement(By.name("q"))
  .sendKeys("webDriver");
driver.findElement(By.name("q"))
  .clear();
driver.findElement(By.name("q"))
  .sendKeys("testing");
```

sendKeys()：在文本输入框输入文本。

```js
driver.findElement(By.locatorType("path"))
  .sendKeys("your text");
```

submit()：提交表单，或者用来模拟按下回车键。

```js
// 例一
driver.findElement(By.locatorType("path"))
  .submit();

// 例二
driver.get("https://www.google.com");
driver.findElement(By.name("q"))
  .sendKeys("webdriver");
element.submit();
```

findElement()：返回选中的第一个元素。

```js
driver.get("https://www.google.com");
driver.findElement(By.id("lst-ib"));
```

findElements()：返回选中的所有元素（0 个或多个）。

```js
// 例一
driver.findElement(By.id("searchbox"))
  .sendKeys("webdriver");
driver.findElements(By.xpath("//div[3]/ul/li"))
  .get(0)
  .click();

// 例二
driver.findElements(By.tagName("select"))
  .get(0)
  .findElements(By.tagName("option"))
  .get(3)
  .click()
  .get(4)
  .click()
  .get(5)
  .click();

// 例三：获取页面所有链接
var links = driver
  .get("https://www.google.com")
  .findElements(By.tagName("a"));
var linkSize = links.size();
var linksSrc = [];

console.log(linkSize);

for(var i=0;i<linkSize;i++) {
  linksSrc[i] = links.get(i).getAttribute("href");
}

for(int i=0;i<linkSize;i++){
  driver.navigate().to(linksSrc[i]);
  Thread.sleep(3000);
}
```

可以使用`size()`，查看到底选中了多少个元素。

### 网页元素的定位

WebDriver 提供 8 种定位器，用于定位网页元素。

*   By.id：HTML 元素的 id 属性
*   By.name：HTML 元素的 name 属性
*   By.xpath：使用 XPath 语法选中 HTML 元素
*   By.cssSelector：使用 CSS 选择器语法
*   By.className：HTML 元素的 class 属性
*   By.linkText：链接文本（只用于选中链接）
*   By.tagName：HTML 元素的标签名
*   By.partialLinkText：部分链接文本（只用于选中链接）

下面是一个使用 id 定位器，选中网页元素的例子。

```js
driver.findElement(By.id("sblsbb")).click();
```

### 网页元素的方法

以下方法属于网页元素的方法，而不是 webDriver 实例的方法。需要注意的是，有些方法是某些元素特有的，比如只有文本框才能输入文字。如果在网页元素上调用不支持的方法，WebDriver 不会报错，也不会给出给出任何提示，只会静静地忽略。

getAttribute()：返回网页元素指定属性的值。

```js
driver.get("https://www.google.com");
driver.findElement(By.xpath("//div[@id='lst-ib']"))
  .getAttribute("class");
```

getText()：返回网页元素的内部文本。

```js
driver.findElement(By.locatorType("path")).getText();
```

getTagName()：返回指定元素的标签名。

```js
driver.get("https://www.google.com");
driver.findElement(By.xpath("//div[@class='sbib_b']"))
  .getTagName();
```

isDisplayed()：返回一个布尔值，表示元素是否可见。

```js
driver.get("https://www.google.com");
assert(driver.findElement(By.name("q"))
  .isDisplayed(),
  '搜索框应该可选择');
```

isEnabled()：返回一个布尔值，表示文本框是否可编辑。

```js
driver.get("https://www.google.com");
var Element = driver.findElement(By.name("q"));
if (Element.isEnabled()) {
  driver.findElement(By.name("q"))
    .sendKeys("Selenium Essentials");
} else {
  throw new Error();
}
```

isSelected()：返回一个布尔值，表示一个元素是否可选择。

```js
driver.findElement(By.xpath("//select[@name='jump']/option[1]"))
  .isSelected()
```

getSize()：返回一个网页元素的宽度和高度。

```js
var dimensions=driver.findElement(By.locatorType("path"))
  .getSize(); 
dimensions.width;
dimensions.height;
```

getLocation()：返回网页元素左上角的 x 坐标和 y 坐标。

```js
var point = driver.findElement(By.locatorType("path")).getLocation();
point.x; // 等同于 point.getX();
point.y; // 等同于 point.getY();
```

getCssValue()：返回网页元素指定的 CSS 属性的值。

```js
driver.get("https://www.google.com");
var element = driver.findElement(By.xpath("//div[@id='hplogo']"));
console.log(element.getCssValue("font-size"));
console.log(element.getCssValue("font-weight"));
console.log(element.getCssValue("color"));
console.log(element.getCssValue("background-size"));
```

### 页面跳转的方法

以下方法用来跳转到某一个页面。

get()：要求浏览器跳到某个网址。

```js
driver.get("URL");
```

navigate().back()：浏览器回退。

```js
driver.navigate().back();
```

navigate().forward()：浏览器前进。

```js
driver.navigate().forward();
```

navigate().to()：跳转到浏览器历史中的某个页面。

```js
driver.navigate().to("URL");
```

navigate().refresh()：刷新当前页面。

```js
driver.navigate().refresh();
// 等同于
driver.navigate()
  .to(driver.getCurrentUrl());
// 等同于
driver.findElement(By.locatorType("path"))
  .sendKeys(Keys.F5);
```

### cookie 的方法

getCookies()：获取 cookie

```js
driver.get("https://www.google.com");
driver.manage().getCookies();
```

getCookieNamed() ：返回指定名称的 cookie。

```js
driver.get("https://www.google.com");
console.log(driver.manage().getCookieNamed("NID"));
```

addCookie()：将 cookie 加入当前页面。

```js
driver.get("https://www.google.com");
driver.manage().addCookie(cookie0);
```

deleteCookie()：删除指定的 cookie。

```js
driver.get("https://www.google.co.in");
driver.manage().deleteCookieNamed("NID");
```

### 浏览器窗口的方法

maximize()：最大化浏览器窗口。

```js
var driver = new FirefoxDriver();
driver.manage().window().maximize();
```

getSize()：返回浏览器窗口、图像、网页元素的宽和高。

```js
driver.manage().window().getSize();
```

getPosition()：返回浏览器窗口左上角的 x 坐标和 y 坐标。

```js
console.log("Position X: " + driver.manage().window().getPosition().x);
console.log("Position Y: " + driver.manage().window().getPosition().y);
console.log("Position X: " + driver.manage().window().getPosition().getX());
console.log("Position Y: " + driver.manage().window().getPosition().getY());
```

setSize()：定制浏览器窗口的大小。

```js
var d = new Dimension(320, 480);
driver.manage().window().setSize(d);
driver.manage().window().setSize(new Dimension(320, 480));
```

setPosition()：移动浏览器左上角到指定位置。

```js
var p = new Point(200, 200);
driver.manage().window().setPosition(p);
driver.manage().window().setPosition(new Point(300, 150));
```

getWindowHandle()：返回当前浏览器窗口。

```js
var parentwindow = driver.getWindowHandle();
driver.switchTo().window(parentwindow);
```

getWindowHandles()：返回所有浏览器窗口。

```js
var childwindows =  driver.getWindowHandles();
driver.switchTo().window(childwindow);
```

switchTo.window()：在浏览器窗口之间切换。

```js
driver.SwitchTo().Window(childwindow);
driver.close();
driver.SwitchTo().Window(parentWindow);
```

### 弹出窗口

以下方法处理浏览器的弹出窗口。

dismiss() ：关闭弹出窗口。

```js
var alert = driver.switchTo().alert();
alert.dismiss();
```

accept()：接受弹出窗口，相当于按下接受 OK 按钮。

```js
var alert = driver.switchTo().alert();
alert.accept();
```

getText()：返回弹出窗口的文本值。

```js
var alert = driver.switchTo().alert();
alert.getText();
```

sendKeys()：向弹出窗口发送文本字符串。

```js
var alert = driver.switchTo().alert();
alert.sendKeys("Text to be passed");
```

authenticateUsing()：处理 HTTP 认证。

```js
var user = new UserAndPassword("USERNAME", "PASSWORD");
alert.authenticateUsing(user);
```

### 鼠标和键盘的方法

以下方法模拟鼠标和键盘的动作。

*   click()：鼠标在当前位置点击。
*   clickAndHold()：按下鼠标不放
*   contextClick()：右击鼠标
*   doubleClick()：双击鼠标
*   dragAndDrop()：鼠标拖放到目标元素
*   dragAndDropBy()：鼠标拖放到目标坐标
*   keyDown()：按下某个键
*   keyUp()：从按下状态释放某个键
*   moveByOffset()：移动鼠标到另一个坐标位置
*   moveToElement()：移动鼠标到另一个网页元素
*   release()：释放拖拉的元素
*   sendKeys()：控制键盘输出