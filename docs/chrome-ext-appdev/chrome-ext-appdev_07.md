# 第四章　管理你的浏览器

前面介绍了 Chrome 扩展基础和 UI 界面，接下来我们来讲一讲有关管理浏览器的相关内容。本章将涉及到书签、Cookies、历史记录、扩展管理和标签有关的内容，通过本章的内容，你将能够创建功能更加强大的扩展。

## 4.1　书签

书签这个功能在早期的浏览器就是标配了，浏览器在几十年的更新中，很多功能都已经被新的技术和方法替代，但书签这个功能一直保留至今，可见它对用户的重要程度。

在搜索引擎如此强大的今天，传统的书签已经不再拥有往日的优势，那么我们为什么现在还要保留和讨论这个功能呢？既然互联网索引从早期的人工编排（雅虎早期就是人工编排互联网黄页的）进化到了机器自动抓取并排序，那么书签这个古老的功能也没有理由止步不前。

说到了书签功能的进步，我们不妨来想一想哪些功能是现在书签所具有而曾经没有的。首先是同步，这个一定要放在第一位。当初多少人重新安装系统后望着浏览器空空如也的收藏夹（书签在原来的部分浏览器中也叫收藏夹）捶胸顿足，甚至当时把导出浏览器收藏夹都写入了重装系统的标配步骤中。现在我们再也不担心这个问题了，各大浏览器基本都支持了同步书签的功能，当然前提是你绑定了一个支持同步的账户。其次就是搜索功能，大家发现本来将页面放入书签是方便以后继续查看，但当书签数量变得庞大之后，这种方便也就无从谈起了，所以书签的搜索功能也就出现了。

在前面提到书签发展的进步并非与本节内容无关，这种进步会激发你的创造力，来想一想怎么通过下面将要讲解的浏览器书签管理接口打造更加智能的书签。

Chrome 为开发者提供了添加、分类（书签文件夹）和排序等方法来操作书签，同时也提供了读取书签的方法。

要在扩展中操作书签，需要在 Manifest 中声明 bookmarks 权限：

```js
"permissions": [
    "bookmarks"
] 
```

在具体讲解操作书签的方法前，先让我们来了解一下书签对象的数据结构。书签对象有 8 个属性，分别是`id`、`parentId`、`index`、`url`、`title`、`dateAdded`、`dateGroupModified`和`children`。这 8 个属性并不是每个书签对象都具有的，比如书签分类，即一个文件夹，它就不具有`url`属性。`index`属性是这个书签在其父节点中的位置，它的值是从 0 开始的。`children`属性值是一个包含若干书签对象的数组。`dateAdded`和`dateGroupModified`的值是自 1970 年 1 月 1 日至修改时间所经过的毫秒数。只有`id`和`title`是书签对象必有的属性，其他的属性都是可选的。`id`不需要人为干预，它是由 Chrome 管理的。根的`id`为`'0'`。

创建书签。可以通过`create`方法来创建书签，下面的代码创建了一个标题为“Google”，URL 为“http://www.google.com/”的书签：

```js
chrome.bookmarks.create({
    parentId: '1',
    index: 0,
    title: 'Google',
    url: 'http://www.google.com/'
}, function(bookmark){
    console.log(bookmark);
}); 
```

请注意上面代码的`parentId`属性，`'0'`为根节点`id`，根节点下是不允许创建书签和书签分组的，它的下面默认只有三个书签分组：书签栏、其他书签和移动设备书签，如果创建时不指定`parentId`，则所创建的书签会默认加入到其他书签中。`create`方法成功后会调用指定的回调函数，回调结果是书签对象。`create`方法支持指定的书签属性只有上述代码中所列出的 4 个：`parentId`、`index`、`title`和`url`，其他属性均不支持指定。如果不指定`index`，这个书签就将自动添加到相应父节点的尾部。

创建书签分类。创建书签分类的方法和创建书签的方法大致相同，如果创建的书签不包含`url`属性，则 Chrome 自动将其视作为书签分类。

调整书签位置。通过`move`方法可以调整书签的位置，这种调整可以是跨越父节点的，下面的代码将 id 为`'16'`的书签移动到了 id 为`'7'`的父节点第 5 个位置：

```js
chrome.bookmarks.move('16', {
    parentId:'7',
    index:4
}, function(bookmark){
    console.log(bookmark);
}); 
```

更新书签。通过 update 方法可以更改书签属性，包括标题和 URL，更新时未指定的属性值将不会更改。下面的代码将将`id`为`'16'`的书签标题改为`'Gmail'`，URL 改为`'https://mail.google.com/'`：

```js
chrome.bookmarks.update('16', {
    title: 'Gmail',
    url: 'https://mail.google.com/'
}, function(bookmark){
    console.log(bookmark);
}); 
```

移除书签。通过`remove`和`removeTree`可以删除书签，`remove`方法可以删除书签和空的书签分组，`removeTree`可以删除包含书签的书签分组。下面的代码移除了`id`为`'16'`的书签和`id`为`'6'`的书签分组。请注意，下面的代码实际上并不能看出删除的是书签还是分组，这要结合用户的实际情况。

```js
chrome.bookmarks.remove('16', function(){
    console.log('Bookmark 16 has been removed.');
});

chrome.bookmarks.removeTree('6', function(){
    console.log('Bookmark group 6 has been removed.');
}); 
```

下面我们来了解一下如何获取用户的书签内容。通过`getTree`方法可以获得用户完整的书签树，但请注意，如果用户的书签树结构过于复杂或内容过多，`getTree`方法的效率会很低，而且也会消耗较多的资源，所以请考虑使用后面的方法按需获取部分书签树。下面的代码获取了用户的整个书签树：

```js
chrome.bookmarks.getTree(function(bookmarkArray){
    console.log(bookmarkArray);
}); 
```

需要指出，上面的代码的返回结果依然是一个数组，虽然这个数组永远都只包含一个元素，书签树的根节点。

`getChildren`方法可以返回以指定节点为父节点的下一级书签节点，但不包括再下一级的节点，也就是说返回的书签对象不包括`children`属性，无论它是否具有子节点。通过这个方法我们可以一层一层地按需获取用户的书签结构。下面的方法获取了根节点的所有子节点。

```js
chrome.bookmarks.getChildren('0', function(bookmarkArray){
    console.log(bookmarkArray);
}); 
```

`getSubTree`方法可以返回自指定节点开始包括当前节点及向下的所有节点，这个方法与`getChildren`的区别是返回值会包含父节点，且没有层级限制，即包含书签对象的`children`属性。下面的代码返回的结果与`getTree`方法返回的结果相同：

```js
chrome.bookmarks.getSubTree('0', function(bookmarkArray){
    console.log(bookmarkArray);
}); 
```

`get`方法可以返回指定节点不包含`children`属性的书签对象数组，指定的节点可以是一个或多个。比如下面的代码获取了`id`为`'16'`和`'17'`的书签对象：

```js
chrome.bookmarks.get(['16', '17'], function(bookmarkArray){
    console.log(bookmarkArray);
}); 
```

`getRecent`方法提供了获取最近添加的多个书签，下面的代码获取了最近添加的 5 个书签：

```js
chrome.bookmarks.getRecent(5, function(bookmarkArray){
    console.log(bookmarkArray);
}); 
```

`search`方法可以返回匹配指定条件的书签对象，匹配的条件只能字符串，比如下面的代码会返回所有标题或 URL 中包含`google`的书签：

```js
chrome.bookmarks.search('google', function(bookmarkArray){
    console.log(bookmarkArray);
}); 
```

最后我们来看一看书签的事件，Chrome 提供了多个事件来监控书签操作行为。

`onCreated`事件用以监控书签的创建行为：

```js
chrome.bookmarks.onCreated.addListener(function(bookmark){
    console.log(bookmark);
}); 
```

`onRemoved`事件用以监控书签的移除行为：

```js
chrome.bookmarks.onRemoved.addListener(function(id, removeInfo){
    console.log('Bookmark '+id+' has been removed:');
    console.log(removeInfo);
}); 
```

`removeInfo`包含`parentId`和`index`属性，与所删除书签对象之前的属性相对应。

`onChanged`事件用以监控书签的更新行为：

```js
chrome.bookmarks.onChanged.addListener(function(id, changeInfo){
    console.log('Bookmark '+id+' has been changed:');
    console.log(changeInfo);
}); 
```

`changeInfo`包含`title`和`url`属性，与所更改书签对象更新后的属性相对应。

`onMoved`事件用以监控书签的移动行为：

```js
chrome.bookmarks.onMoved.addListener(function(id, moveInfo){
    console.log('Bookmark '+id+' has been moved:');
    console.log(moveInfo);
}); 
```

`moveInfo`包含`parentId`、`index`、`oldParentId`和`oldIndex`属性，与所移动书签对象移动前后的属性相对应。

`onChildrenReordered`事件用以监控一个书签分组下的更改子节点顺序的行为：

```js
chrome.bookmarks.onChildrenReordered.addListener(function(id, reorderInfo){
    console.log('Bookmark '+id+' has a new children order:');
    console.log(reorderInfo);
}); 
```

`reorderInfo`是包含顺序更改后子节点 id 的数组。

`onImportBegan`和`onImportEnded`事件分别用以监控导入书签开始和结束的行为：

```js
onImportBegan(function(){
    console.log('Bookmark import began.');
});

onImportEnded(function(){
    console.log('Bookmark import ended.');
}); 
```

请注意，如果检测到浏览器正在导入书签（`onImportBegan`事件被触发但`onImportEnded`事件还未被触发），应当忽略`onCreated`事件，但其他的操作可以被立即执行。

以上就是书签相关的全部内容，读者可以结合之前的内容创建更加智能方便的书签管理扩展。比如可以直接通过地址栏搜索书签，或者当用户使用 Google 搜索时将匹配到的书签结果添加到 Google 搜索结果的前端，类似 Google 广告推广那样。这些新奇的点子就交给读者们自行实现吧，在此就不给出实例了。

## 4.2　Cookies

Cookies 是浏览器记录在本地的用户数据，如用户的登录信息。Chrome 为扩展提供了 Cookies API 用以管理 Cookies。

要管理 Cookies，需要在 Manifest 中声明`cookies`权限，同时也要声明所需管理 Cookies 所在的域：

```js
"permissions": [
    "cookies",
    "*://*.google.com"
] 
```

如果想要管理所有的 Cookies 可以声明如下权限：

```js
"permissions": [
    "cookies",
    "<all_urls>"
] 
```

请注意，除非必要，否则请不要如此声明权限，这会提示此扩展可以访问所有网络资源，给用户带来不安。

Chrome 定义的`Cookie`对象包含如下属性：`name`（名称）、`value`（值）、`domain`（域）、`hostOnly`（是否只允许完全匹配 domain 的请求访问）、`path`（路径）、`secure`（是否只允许安全连接调用）、`httpOnly`（是否禁止客户端调用）、`session`（是否是 session cookie）、`expirationDate`（过期时间）和`storeId`（包含此 cookie 的 cookie store 的 id）。

读 Cookies。Chrome 提供了`get`和`getAll`两个方法读取 Cookies，`get`方法可以读取指定`name`、`url`和`storeId`的 Cookie，其中`storeId`可以不指定，但是`name`和`url`必须指定。如果在同一 URL 中包含多个`name`相同的 Cookies，则会返回`path`最长的那个，如果有多个 Cookies 的`path`长度相同，则返回创建最早的那个。

```js
chrome.cookies.get({
    url: 'https://github.com',
    name: 'dotcom_user'
}, function(cookie){
    console.log(cookie.value);
}); 
```

这里需要注意一点，如果`cookie`的`secure`属性值为`true`，那么通过`get`获取时`url`应该是 https 协议。

`getAll`方法与 get 方法不同，它可以获取所有符合条件的 Cookies，支持的匹配条件包括`url`、`name`、`domain`、`path`、`secure`、`session`和`storeId`中的任意一个或多个，如果一个都不指定，则返回所有此扩展有权访问到的 Cookies。比如下面的代码就可以获取到所有可以读取的 Cookies：

```js
chrome.cookies.getAll({}, function(cookies){
    console.log(cookies);
}); 
```

设置 Cookie。`set`方法可以设置 Cookie：

```js
chrome.cookies.set({
    'url':'http://github.com/test_cookie',
    'name':'TEST',
    'value':'foo',
    'secure':false,
    'httpOnly':false
}, function(cookie){
    console.log(cookie);
}); 
```

如果创建成功，则回调函数会获取到创建后的`cookie`对象，否则会得到`null`。`url`是必须指定的，其他的属性可选。另外扩展对 URL 必须有访问权限，否则会设置失败。如果不指定`expirationDate`属性，则所创建的 Cookie 将在浏览器关闭后被删除。

![enter image description here](img/00032.jpeg)
*设置 Cookies*

删除 Cookie。`remove`方法可以删除指定`url`、`name`和`storeId`的 Cookie。

```js
chrome.cookies.remove({
    url: 'http://www.google.com',
    name: '_ga'
}, function(result){
    console.log(result);
}); 
```

同样，扩展首先要具有对 URL 的访问权限，否则删除操作会失败。

除非你清楚在做什么，不要轻易删除用户的 Cookies，否则你可能会收到大量用户抱怨气愤的邮件。

`getAllCookieStores`方法用来获取全部的 cookie store，cookie store 包含一个`id`属性和一个`tabIds`属性，`id`的属性值为这个 cookie store 的`id`，`tabIds`为包含共享这个 cookie store 所有 tab 的`id`的数组。有关 tab 的内容将在后面的章节讲解。

`onChanged`事件用来监控 cookie 的设置和删除行为：

```js
chrome.cookies.onChanged.addListener(function(changeInfo){
    console.log(changeInfo);
}); 
```

`changeInfo`包含三个属性：`removed`，是否是删除行为；`cookie`，被设置或删除的`cookie`对象；`cause`，Cookie 变化的原因，可能的值包括`evicted`、`expired`、`explicit`、`expired_overwrite`和`overwrite`。

再次提醒，Cookies 是用户的敏感数据，在进行操作时一定倍加小心，并要让用户有知情权，必要时一定要先得到用户的确认。

## 4.3　历史

历史用于记录用户访问过页面的信息。与书签一样，历史也是浏览器很早就具有的功能，对用户来说也是一个很重要的功能。Chrome 提供了`history`接口，允许扩展对用户的历史进行管理。

要使用`history`接口，需要在 Manifest 中声明`history`权限：

```js
"permissions": [
    "history"
] 
```

管理历史的方法包括`search`、`getVisits`、`addUrl`、`deleteUrl`、`deleteRange`和`deleteAll`。其中`search`和`getVisits`用于读取历史，`addUrl`用于添加历史，`deleteUrl`、`deleteRange`和`deleteAll`用于删除历史。

读取历史。Chrome 提供了`search`和`getVisits`两种方法读取历史。通过`search`方法可以读取匹配指定文字，指定时间区间，指定条目的历史结果。

```js
chrome.history.search({
    text: 'Google',
    startTime: new Date().getTime()-24*3600*1000,
    endTime: new Date().getTime(),
    maxResults: 20
}, function(historyItemArray){
    console.log(historyItemArray);
}); 
```

上述代码会返回最近 24 小时内匹配“Google”的 20 条历史结果。`startTime`和`endTime`都是距 1970 年 1 月 1 日的毫秒数。返回结果是包含多个`historyItem`对象的数组，`historyItem`对象包含 6 个属性，分别是`id`、`url`、`title`、`lastVisitTime`、`visitCount`和`typedCount`，其中`typedCount`是用户通过在地址栏键入访问此历史的次数。若不指定`text`属性，则返回全部历史结果。

`getVisits`方法可以获取指定 URL 的访问结果。必须指定完整的 URL，返回的结果会绝对匹配指定的 URL，也就是说，如果指定`'http://www.google.com/'`，返回的结果不会包含`'http://www.google.com/a/'`的内容。不要忘记`http://`，这也是不可省略的。

```js
chrome.history.getVisits(
    url: 'http://www.google.com/'
}, function(visitItemArray){
    console.log(visitItemArray);
}); 
```

返回的结果是包含多个`visitItem`对象的数组，`visitItem`对象包含 5 个属性，分别是`id`、`visitId`、`visitTime`、`referringVisitId`和`transition`。其中`id`为与指定 URL 匹配的对象的`id`，对于匹配同一 URL 的对象拥有相同的`id`，`visitId`是这个访问结果的`id`，`visitId`是唯一的。`visitTime`同样是毫秒数。`transition`是此访问记录打开的方式，具体解释如下。

Chrome 对每一个访问记录都详细地归类了打开方式，用`transition`属性记录。打开方式一共分为 11 种，这看起来确实会让人有一些头疼。比较常见的有四种，分别为`link`、`typed`、`reload`和`form_submit`。`link`是用户通过超级链接打开的方式，`typed`是用户通过在地址栏中输入网址打开的方式，`reload`是用户通过刷新（包括恢复关闭的标签）打开的方式，`form_submit`是通过提交表单打开的方式（通过脚本提交表单的情况不算此方式）。

与浏览器 UI 和设置相关的有两种，分别为`auto_bookmark`和`auto_toplevel`。`auto_bookmark`是通过浏览器 UI 中的建议打开的方式——比如通过菜单等。`auto_toplevel`为浏览器设置中默认打开的方式，比如浏览器的主页，或者是通过命令行启动时附带的参数。

嵌入式框架相关的有两个，`auto_subframe`和`manual_subframe`，其中`auto_subframe`为自动加载的嵌入式框架打开的方式，很多广告都是这样的打开方式——很多用户并不知道其实那些广告是在一个独立的页面中。`manual_subframe`则是用户手动加载的嵌入式框架打开的方式，比如用户操作商品菜单查看不同款式商品页面，就是手动加载嵌入式框架。

最后还有三种是和 omnibox 搜索建议相关的，分别为`generated`、`keyword`和`keyword_generated`。`generated`为通过 omnibox 给出搜索建议打开的方式，所打开的页面通常为搜索引擎的结果界面。`keyword`和`keyword_generated`都是通过用户在地址栏中输入的关键字生成的 URL 访问的方式，但其 URL 并不是默认搜索引擎生成的（否则就是`generated`了）。

添加历史。`addUrl`方法可以将特定的 url 以当前时间为访问时间，添加至历史中。

```js
chrome.history.addUrl({
    url: 'http://twitter.com'
}, function(){
    console.log('Twitter has been added to history.');
}); 
```

删除历史。`deleteUrl`可以删除指定 URL 的历史，`deleteRange`可以删除指定时间段的历史，`deleteAll`可以删除全部历史。

```js
chrome.history.deleteUrl({
    url: 'http://www.google.com'
}, function(){
    console.log('Google has been deleted from history.');
});

chrome.history.deleteRange({
    startTime: new Date().getTime()-24*3600*1000,
    endTime: new Date().getTime()
}, function(){
    console.log('History in past 24 hours has been deleted.');
});

chrome.history.deleteAll(function(){
    console.log('All history has been deleted.');
}); 
```

Chrome 提供两个事件，`onVisited`和`onVisitRemoved`，分别监听用户访问历史和历史被删除的事件。

```js
chrome.history.onVisited.addListener(function(historyItem){
    console.log(historyItem);
});

chrome.history.onVisitRemoved.addListener(function(removedObject){
    console.log(removedObject);
}); 
```

对于`onVisitRemoved`事件，返回的`removedObject`结果包含两个属性，`allHistory`和`urls`。其中`urls`属性包含所有被删除历史的 URL。`allHistory`为布尔型，如果所有历史均被删除，`allHistory`的值为`ture`，同时`urls`的值会为一个空数组。

历史和 cookies 一样都是用户的敏感数据，进行操作时应让用户有知情权，尤其是要将用户历史数据与第三方共享时（包括开发者自己的服务器），一定要先得到用户的同意，并且要让用户得知哪些数据会被使用。

## 4.4　管理扩展与应用

除了通过 chrome://extensions/管理 Chrome 扩展和应用外，也可以通过 Chrome 的`management`接口管理。`management`接口可以获取用户已安装的扩展和应用信息，同时还可以卸载和禁用它们。通过`management`接口可以编写出智能管理扩展和应用的程序。

要使用`management`接口，需要在 Manifest 中声明`management`权限：

```js
"permissions": [
    "management"
] 
```

读取用户已安装扩展和应用的信息。Management 提供了两个方法获取用户已安装扩展应用的信息，分别是`getAll`和`get`。

```js
chrome.management.getAll(function(exInfoArray){
    console.log(exInfoArray);
});

chrome.management.get(exId, function(exInfo){
    console.log(exInfo);
}); 
```

`exInfo`是扩展信息对象，其结构如下：

```js
{
    id: 扩展 id,
    name: 扩展名称,
    shortName: 扩展短名称,
    description: 扩展描述,
    version: 扩展版本,
    mayDisable: 是否可被用户卸载或禁用,
    enabled: 是否已启用,
    disabledReason: 扩展被禁用原因,
    type: 类型,
    appLaunchUrl: 启动 url,
    homepageUrl: 主页 url,
    updateUrl: 更新 url,
    offlineEnabled: 离线是否可用,
    optionsUrl: 选项页面 url,
    icons: [{
        size: 图片尺寸,
        url: 图片 URL
    }],
    permissions: 扩展权限,
    hostPermissions: 扩展有权限访问的 host,
    installType: 扩展被安装的方式
} 
```

其中`type`属性的可能值为`extension`、`hosted_app`、`packaged_app`、`legacy_packaged_app`或`theme`。`installType`可能的值为`admin`（管理员安装）、`development`（载入未打包的扩展）、`normal`（通过 crx 正常安装）、`sideload`（第三方程序安装）或`other`（其他）。

获取权限警告。`getPermissionWarningsById`和`getPermissionWarningsByManifest`方法可以获取权限警告，这些警告与用户安装扩展时网上应用商店弹出的警告类似。

```js
chrome.management.getPermissionWarningsById(exId, function(permissionWarningArray){
    console.log(permissionWarningArray);
});

getPermissionWarningsByManifest(exManifest, function(permissionWarningArray){
    console.log(permissionWarningArray);
}); 
```

上述代码中，`exManifest`是字符串型的，不是对象型的。

启用、禁用、卸载扩展和启动应用。`setEnabled`方法可以启用或禁用扩展应用，如果一个扩展或应用被禁用，它的后台页面不会运行。

```js
chrome.management.setEnabled(exId, enabled, function(){
    if(enabled){
        console.log('Extension '+exId+' has been enabled.');
    }
    else{
        console.log('Extension '+exId+' has been disabled.');
    }
}); 
```

卸载扩展有两种方法，`uninstall`可以卸载指定 id 的扩展，`uninstallSelf`可以卸载扩展自身且无需请求`management`权限。

```js
uninstall(exId, {
    showConfirmDialog: true
}, function(){
    console.log('Extension '+exId+' has been uninstalled.');
});

uninstallSelf({
    showConfirmDialog: true
}, function(){
    console.log('This extension has been uninstalled.');
}); 
```

如果不希望在卸载前显示确认窗口，可以将`showConfirmDialog`的值设为`false`。

通过`launchApp`方法启动应用：

```js
chrome.management.launchApp(exId, function(){
    console.log('App '+exId+' has been launched.');
}); 
```

`management`接口提供了四种事件，`onInstalled`、`onUninstalled`、`onEnabled`和`onDisabled`，分别用于监听安装、卸载、启用和禁用扩展应用。

```js
chrome.management.onInstalled.addListener(function(exInfo){
    console.log('Extension '+exInfo.id+' has been installed.')
});

chrome.management.onUninstalled.addListener(function(exId){
    console.log('Extension '+exId+' has been uninstalled.');
});

chrome.management.onEnabled.addListener(function(exInfo){
    console.log('Extension '+exInfo.id+' has been enabled.');
});

chrome.management.onDisabled.addListener(function(exInfo){
    console.log('Extension '+exInfo.id+' has been disabled.');
}); 
```

本节讲解了管理扩展和应用的接口内容，看起来有些枯燥，但如果使用恰当设计合理，可以编写出让用户很 feel 的扩展。

## 4.5　标签

前面的章节中，多次提到了标签，本节将详细讲解对标签信息获取和操作的内容。在开始介绍之前，先让我们来看一下标签对象的结构：

```js
{
    id: 标签 id,
    index: 标签在窗口中的位置，以 0 开始,
    windowId: 标签所在窗口的 id,
    openerTabId: 打开此标签的标签 id,
    highlighted: 是否被高亮显示,
    active: 是否是活动的,
    pinned: 是否被固定,
    url: 标签的 URL,
    title: 标签的标题,
    favIconUrl: 标签 favicon 的 URL,
    status :标签状态，loading 或 complete,
    incognito: 是否在隐身窗口中,
    width: 宽度,
    height: 高度,
    sessionId: 用于 sessions API 的唯一 id
} 
```

Chrome 通过`tabs`方法提供了管理标签的方法与监听标签行为的事件，大多数方法与事件是无需声明特殊权限的，但有关标签的`url`、`title`和`favIconUrl`的操作（包括读取），都需要声明`tabs`权限。

```js
"permissions": [
    "tabs"
] 
```

获取标签信息。Chrome 提供了三种获取标签信息的方法，分别是`get`、`getCurrent`和`query`。`get`方法可以获取到指定 id 的标签，`getCurrent`则获取运行的脚本本身所在的标签，`query`可以获取所有符合指定条件的标签。

```js
chrome.tabs.get(tabId, function(tab){
    console.log(tab);
});

chrome.tabs.getCurrent(function(tab){
    console.log(tab);
}); 
```

`query`方法可以指定的匹配条件如下：

```js
{
    active: 是否是活动的,
    pinned: 是否被固定,
    highlighted: 是否正被高亮显示,
    currentWindow: 是否在当前窗口,
    lastFocusedWindow: 是否是上一次选中的窗口,
    status: 状态，loading 或 complete,
    title: 标题,
    url: 所打开的 url,
    windowId: 所在窗口的 id,
    windowType: 窗口类型，normal、popup、panel 或 app,
    index: 窗口中的位置
} 
```

下面的代码获取了所有在窗口中活动的标签：

```js
chrome.tabs.query({
    active: true
}, function(tabArray){
    console.log(tabArray);
}); 
```

创建标签。创建标签与在浏览器中打开新的标签行为类似，但可以指定更加丰富的信息，如 URL、窗口中的位置和活动状态等。

```js
chrome.tabs.create({
    windowId: wId,
    index: 0,
    url: 'http://www.google.com',
    active: true,
    pinned: false,
    openerTabId: tId
}, function(tab){
    console.log(tab);
}); 
```

其中`wId`是创建标签所在窗口的`id`，如果不指定，则默认在当前窗口中打开。`tId`是打开此标签的标签`id`，可以不指定，但如果指定，那么所创建的标签必须与这个标签在同一窗口中。

除了用`create`方法，还可以使用`duplicate`方法“复制”指定标签：

```js
chrome.tabs.duplicate(tabId, function(tab){
    console.log(tab);
}); 
```

更新标签。通过`update`方法可以更新标签的属性：

```js
chrome.tabs.update(tabId, {
    url: 'http://www.google.com'
}, function(tab){
   console.log(tab);
}); 
```

更新标签时也可以不指定`tabId`，如果不指定，默认会更改当前窗口的活动标签。需要指出，直到 31.0.1650.63 m，更新`highlighted`属性为`true`后，标签`active`属性也会被指定为`true`，所以如果只是想将某个标签高亮以引起用户的注意，需要先记录当前的标签`id`，更新后再将这个标签的`active`属性改回`true`。这个 bug 在之后的版本也许会被修正。

移动标签。`move`方法可以将指定的一个或多个标签移动到指定位置：

```js
chrome.tabs.move(tabIds, {
    'windowId':wId,
    'index':0
}, function(tabs){
    console.log(tabs);
}); 
```

其中`tabIds`可以是一个数字型的标签`id`，也可以是一个包含多个标签`id`的数组。返回的`tabs`可能是标签对象也可能是包含多个标签对象的数组。如果指定的`index`为`-1`，会将标签移动到指定窗口的最后面。

重载标签。`reload`方法可以重载指定标签，同时还可以指定是否跳过缓存（强制刷新）：

```js
chrome.tabs.reload(tabId, {
    bypassCache: true
}, function(){
    console.log('The tab has been reloaded.');
}); 
```

浏览器通常会对一些静态资源进行缓存，JavaScript 中的`location.reload()`方法通常无法实现强制刷新，此时上面的方法就会很好地解决这个问题。

移除标签。通过`remove`方法可以关闭一个或多个标签：

```js
chrome.tabs.remove(tabIds, function(){
    console.log('The tabs has been closed.');
}); 
```

其中`tabIds`可以是一个数字型的标签`id`，也可以是一个包含多个标签`id`的数组。

获取当前标签页面的显示语言。有时可能需要针对用户浏览内容语言的不同，采用不同的处理方法。比如翻译扩展就要根据不同的语言决定是否提示用户进行翻译。

```js
chrome.tabs.detectLanguage(tabId, function(lang){
    console.log('The primary language of the tab is '+lang);
}); 
```

如果不指定`tabId`，则返回当前窗口当前标签的语言。

获取指定窗口活动标签可见部分的截图。Chrome 提供了截取指定窗口活动标签页面为图片的接口：

```js
chrome.tabs.captureVisibleTab(windowId, {
    format: 'jpeg',
    quality: 50
}, function(dataUrl){
    window.open(dataUrl, 'tabCapture');
}); 
```

其中`format`还支持`png`，如果指定为`png`，则`quality`属性会被忽略。如果指定`jpeg`格式，`quality`的取值范围为 0-100，数值越高，图片质量越好，体积也越大。扩展只有声明`activeTab`或`<all_url>`权限能获取到活动标签的截图：

```js
"permissions": [
    "activeTab"
] 
```

注入 JS 和 CSS。之前我们接触过`content_scripts`，它可以向匹配条件的页面注入 JS 和 CSS，但是却无法向用户指定的标签注入。通过`executeScript`和`insertCSS`可以做到向指定的标签注入脚本。

```js
chrome.tabs.executeScript(tabId, {
    file: 'js/ex.js',
    allFrames: true,
    runAt: 'document_start'
}, function(resultArray){
    console.log(resultArray);
}); 
```

也可以直接注入代码：

```js
chrome.tabs.executeScript(tabId, {
    code: 'document.body.style.backgroundColor="red"',
    allFrames: true,
    runAt: 'document_start'
}, function(resultArray){
    console.log(resultArray);
}); 
```

向指定的标签注入 CSS：

```js
chrome.tabs.insertCSS(tabId, {
    file: 'css/insert.css',
    allFrames: false,
    runAt: 'document_start'
}, function(){
    console.log('The css has been inserted.');
}); 
```

插入 CSS 也可以指定具体代码。

`executeScript`和`insertCSS`方法中`runAt`的值可以是`'document_start'`、`'document_end'`或`'document_idle'`。

与指定标签中的内容脚本（content script）通信。前面章节介绍过扩展页面间的通信，我们也可以与指定的标签通信，方法如下：

```js
chrome.tabs.sendMessage(tabId, message, function(response){
    console.log(response);
}); 
```

请注意，后台页面主动与`content_scripts`通信需要使用`chrome.tabs.sendMessage`方法¹。

^(1 `chrome.tabs.executeScript`方法也可以实现后台页面与内容脚本的通信，但更强调是后台页面向标签页注入脚本。)

由于标签的操作行为比较多，所以相应的监视事件也很多。监控标签行为的事件包含`onCreated`、`onUpdated`、`onMoved`、`onActivated`、`onHighlighted`、`onDetached`、`onAttached`、`onRemoved`和`onReplaced`。

大部分事件都比较好理解，下面重点讲一讲不易理解的事件。`onHighlighted`是当标签被高亮显示时所触发的事件，`active`和`highlight`是有区别的，`active`是指标签在当前窗口中正被显示，`highlight`只是标签的颜色被显示成了白色——如果此标签没有被选中正常情况下是浅灰色。`onDetached`是当标签脱离窗口时所触发的事件，导致此事件触发的原因是用户在两个不同的窗口直接拖拽标签。`onAttached`是标签附着到窗口上时所触发的事件，同样是用户在两个不同的窗口直接拖拽标签导致的。`onReplaced`是当标签被其他标签替换时触发的事件²。

^(2 要解释清楚 onReplaced 就不得不提一下即搜即得和预呈现（Instant search, Prerendering）。例如默认搜索引擎为 Google，启用了即搜即得，网络条件也足够好，在打开的另一个网页地址栏中开始输入关键字并且即时出现结果时，此时按下回车键，当前标签页就会被 Google 搜索结果替换，产生 onReplaced 事件。如果扩展程序通过 tabId 追踪标签页的话就必须处理该事件。)

```js
chrome.tabs.onCreated.addListener(function(tab){
    console.log(tab);
});

chrome.tabs.onUpdated.addListener(function(tabId, changeInfo, tab){
    console.log('Tab '+tabId+' has been changed with these options:');
    console.log(changeInfo);
});

chrome.tabs.onMoved.addListener(function(tabId, moveInfo){
    console.log('Tab '+tabId+' has been moved:');
    console.log(moveInfo);
});

chrome.tabs.onActivated.addListener(function(activeInfo){
    console.log('Tab '+activeInfo.tabId+' in window '+activeInfo.windowId+' is active now.');
});

chrome.tabs.onHighlighted.addListener(function(highlightInfo){
    console.log('Tab '+activeInfo.tabId+' in window '+activeInfo.windowId+' is highlighted now.');
});

chrome.tabs.onDetached.addListener(function(tabId, detachInfo){
    console.log('Tab '+tabId+' in window '+detachInfo.oldWindowId+' at position '+detachInfo.oldPosition+' has been detached.');
});

chrome.tabs.onAttached.addListener(function(tabId, attachInfo){
    console.log('Tab '+tabId+' has been attached to window '+detachInfo.newWindowId+' at position '+detachInfo.newPosition+' .');
});

chrome.tabs.onRemoved.addListener(function(tabId, removeInfo){
    console.log('Tab '+tabId+' in window '+removeInfo.windowId+', and the window is '+(removeInfo.isWindowClosing?'closed.':'open.'));
});

chrome.tabs.onReplaced.addListener(function(addedTabId, removedTabId){
    console.log('Tab '+removedTabId+' has been replaced by tab '+addedTabId+'.');
); 
```

通过标签接口，扩展可以更灵活地处理不同标签。虽然标签涉及到的内容很多，但常用的部分很有限，读者在阅读此节时，不妨先把精力重点放在那些常用易懂的方法事件上，对于剩下的部分随用随查即可。

## 4.6　Override Pages

Chrome 不仅提供了管理书签、历史和标签的接口，还支持用自定义的页面替换 Chrome 相应默认的页面，这就是 override pages。目前支持替换的页面包含 Chrome 的书签页面、历史记录和新标签页面。

使用 override pages 很简单，只需在 Manifest 中进行声明即可（一个扩展只能替换一个页面）：

```js
"chrome_url_overrides" : {
    "bookmarks": "bookmarks.html"
}

"chrome_url_overrides" : {
    "history": "history.html"
}

"chrome_url_overrides" : {
    "newtab": "newtab.html"
} 
```

把上面页面的地址替换成你自己的就可以了。

Google 官方对 override pages 给出了几点建议（以下内容翻译来自[`crxdoc-zh.appspot.com/extensions/override`](https://crxdoc-zh.appspot.com/extensions/override)）：

*   使您的页面又快又小。
    用户期望内置的浏览器页面能够立即打开。请避免做任何可能花较长时间的事情，例如，避免同步地获取网络或数据库资源。

*   在您的页面中包含标题。
    否则用户可能会看到页面的 URL，会令人感到疑惑。这是一个指定标题的例子：新标签页

*   不要假定页面具有键盘焦点。
    当用户创建新标签页时总是地址栏先获得焦点。

*   不要试着模仿默认的“打开新的标签页”页面。
    用于创建与默认的“打开新的标签页”页面类似（具有最常访问的网站、最近关闭的标签页、提示、主题背景图像等等）的修改版本所需的 API 还不存在。在出现那些 API 之前您还是最好还是考虑一些完全不同的新想法。