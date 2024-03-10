# 第八章　媒体库

通过`mediaGalleries`接口 Chrome 应用可以操作计算机中的媒体库，如音乐文件夹、图片文件夹、iPod 设备和 iTunes 等。

Chrome 应用操作媒体库与操作文件系统类似——其实媒体库也是文件系统的一部分，但是`mediaGalleries`接口与`fileSystem`有些区别。

首先`mediaGalleries`能自动找到计算机中的媒体库而不必像`fileSystem`那样需要用户手动指定目录或文件位置，其次`mediaGalleries`只会获取到支持的媒体文件，其他文件会被自动过滤掉。

要使用`mediaGalleries`接口需要在 Manifest 中声明`mediaGalleries`权限：

```
"permissions": {
    {"mediaGalleries": ["read", "allAutoDetected"]} 
} 
```

`mediaGalleries`权限的声明与`fileSystem`类似，需要指定更加详细的权限。`"read"`表示有读取文件内容的权限，`"allAutoDetected"`表示有自动获取媒体库位置的权限。其他的权限还包括`"delete"`和`"copyTo"`，分别代表删除文件和复制文件。

需要注意的是`mediaGalleries`接口不提供`"write"`权限——直接在媒体库中创建或更改文件是禁止的，但可以在临时文件夹中创建文件后复制到媒体库中。

## 8.1　获取媒体库

如果在 Manifest 中声明了了`"allAutoDetected"`权限，则 Chrome 应用可以无需用户手动指定，自动获取到媒体库的位置。

通过`getMediaFileSystems`方法可以获取到媒体库对应的`fileSystem`：

```
chrome.mediaGalleries.getMediaFileSystems({
    interactive: 'if_needed'
}, function(fileSystemArray){
    //We'll do something with fileSystemArray later
}); 
```

得到的是一个包含多个`fileSystem`对象的数组`fileSystemArray`。`fileSystem`对象我们在第七章虽有提及，但却接触不多，更多地是对`Entry`对象的操作。不过我们可以回忆一下 7.1 节在介绍`fileSystem`时提到过其有两个属性，分别是`name`和`root`，其中`root`是此文件系统的根目录`DirectoryEntry`。所以`filesystem.root`就是我们熟悉的`Entry`对象。

虽然通过`filesystem.root`可以像操作文件系统一样操作媒体库，但是除了文件系统中提供的属性外（如`isDirectory`和`isFile`等），对于媒体库我们还希望获得其他的信息——是否是媒体设备（如音乐播放器）、是否是可以移动设备（让我们来决定是否应进行同步操作）、目前设备是否可用等等。

为了得到这些信息，通过文件系统的接口是不够的，为此 Chrome 提供了获取此类信息的方法，`getMediaFileSystemMetadata`：

```
mediaInfo = chrome.mediaGalleries.getMediaFileSystemMetadata(mediaFileSystem); 
```

它的传入参数是`fileSystem`对象，而不是`Entry`对象。

也可以通过`getAllMediaFileSystemMetadata`方法获取到全部的媒体库信息，但是`getAllMediaFileSystemMetadata`方法与`getMediaFileSystemMetadata`方法不同的是它使用回调函数的方式传回结果：

```
chrome.mediaGalleries.getAllMediaFileSystemMetadata(function(mediaInfoArray){
    //do something with mediaInfoArray
}); 
```

结果是一个数组，描述每个媒体库信息，这些对象的结构与通过`getMediaFileSystemMetadata`方法获取到的对象结构相同。

`mediaInfo`对象包含 6 个属性，分别是`name`、`galleryId`、`deviceId`、`isRemovable`、`isMediaDevice`和`isAvailable`。其中`isRemovable`表明此媒体库是否是一个可移动设备，`isMediaDevice`表明此媒体库是否是一个媒体设备，`isAvailable`表明此媒体库现在是否可用。

下面我们来一起制作一款媒体库管理应用，并在之后的小节中逐步完善它。

首先创建 Manifest 文件：

```
{
    "app": {
        "background": {
            "scripts": ["background.js"]
        }
    },
    "manifest_version": 2,
    "name": "Media Manager",
    "version": "1.0",
    "description": "A media manage tool.",
    "icons": {
        "128": "logo.png"
    },
    "permissions": [
        {"mediaGalleries": ["read", "delete", "copyTo", "allAutoDetected"]}
    ]
} 
```

background.js 用来监控应用启动事件，当用户启动应用后创建一个窗口：

```
chrome.app.runtime.onLaunched.addListener(function() {
    chrome.app.window.create('main.html', {
        id: 'main',
        bounds: {
            width: 800,
            height: 600
        }
    });
}); 
```

在 main.html 用于展示检测到的媒体库：

```
<html>
<head>
<style>
@font-face {
    font-family: 'iconfont';
    src: url('iconfont.woff') format('woff');
}

* {
    padding: 0;
    margin: 0;
}

body {
    background: #2D2D2D;
}

#appTitle {
    height: 60px;
    line-height: 60px;
    padding: 0 20px;
    font-size: 24px;
    color: #CCC;
    background: #222;
}

#path {
    height: 40px;
    line-height: 40px;
    padding: 0 20px;
    color: #888;
    font-size: 16px;
    background: #222;
    border-bottom: black 1px solid;
    box-sizing: border-box;
}

#path span {
    display: block;
    float: left;
}

#path span.name {
    max-width: 100px;
    white-space: nowrap;
    text-overflow: ellipsis;
    overflow: hidden;
}

#path span.pointer {
    padding: 0 10px 0 5px;
}

#container .item {
    display: block;
    height: 100px;
    width: 100px;
    float: left;
    color: white;
}

#container .item .icon {
    display: block;
    text-align: center;
    height: 80px;
    line-height: 80px;
    font-size: 60px;
    font-family: 'iconfont';
}

#container .item .text {
    display: block;
    text-align: center;
    padding: 0 5px;
    height: 20px;
    line-height: 20px;
    font-size: 14px;
    text-overflow: ellipsis;
    overflow: hidden;
}
</style>
</head>
<body>
<div id="appTitle">Media Manager</div>
<div id="path"><span class="name">媒体库</span><span class="pointer">»</span></div>
<div id="container"></div>
<script src="main.js"></script>
</body>
</html> 
```

其中用到了图标字体以显示矢量图标，字体来自[`www.iconfont.cn/`](http://www.iconfont.cn/)。

下面来编写 main.js：

```
function getMedia(){
    chrome.mediaGalleries.getMediaFileSystems({
        interactive: 'if_needed'
    }, listMediaGalleries);
}

function listMediaGalleries(fileSystemArray){
    document.getElementById('container').innerHTML = '';
    for(var i=0; i<fileSystemArray.length; i++){
        var info = chrome.mediaGalleries.getMediaFileSystemMetadata(fileSystemArray[i]);
        var item = document.createElement('span');
        item.className = 'item';
        item.title = info.name;
        document.getElementById('container').appendChild(item);
        var icon = document.createElement('span');
        icon.className = 'icon';
        icon.innerHTML = '&#xf00c5;';
        item.appendChild(icon);
        var text = document.createElement('span');
        text.className = 'text';
        text.innerHTML = info.name;
        item.appendChild(text);
    }
}

getMedia(); 
```

载入到 Chrome 后可以看到目前运行的截图。

![enter image description here](img/00057.jpeg)
*获取媒体库*

## 8.2　添加及移除媒体库

除了通过`"allAutoDetected"`权限让 Chrome 应用自动查找媒体库外，也可以让用户手动添加或者移除媒体库。

在上一节中我们调用`getMediaFileSystems`方法时，将其参数中的`interactive`指定为了`if_needed`，如果将其指定为`yes`则会出现一个弹出让用户选择保留的媒体库或者添加其他媒体库：

![enter image description here](img/00058.jpeg)
*媒体库选择弹窗*

如果只想单纯提供添加其他位置的功能，可以使用`addUserSelectedFolder`方法，当调用`addUserSelectedFolder`方法时，会弹出一个目录选择窗口让用户选择新媒体库的位置：

![enter image description here](img/00059.jpeg)
*添加新媒体库位置*

`addUserSelectedFolder`方法使用回调函数传递用户选择结果：

```
chrome.mediaGalleries.addUserSelectedFolder(function(mediaFileSystems, selectedFileSystemName){
    //We'll do something with mediaFileSystems later
}); 
```

其中`mediaFileSystems`是一个包含多个`FileSystem`的数组，其包含的是应用有权限访问的所有媒体库`FileSystem`，而非只是用户刚刚选择的。`selectedFileSystemName`是一个字符串，如果用户在添加媒体库完成前点击了取消按钮，则`selectedFileSystemName`返回一个空值。

使用`dropPermissionForMediaFileSystem`方法可以取消对指定媒体库的访问权¹：

```
chrome.mediaGalleries.dropPermissionForMediaFileSystem(galleryId, function(){
    //do something after give up access a media gallery
}); 
```

^(1 从 Chrome 36 开始支持。)

下面为`Media Manager`加上添加移除媒体库按钮：

```
<div id="appTitle">Media Manager<span id="edit">&#x3466;</span></div> 
```

在 CSS 中添加按钮样式：

```
#edit {
    display: inline-block;
    font-size: 12px;
    font-family: 'iconfont';
    height: 20px;
    line-height: 20px;
    padding: 5px;
    cursor: pointer;
} 
```

在 JS 中添加按钮事件：

```
document.getElementById('edit').onclick = function(){
    document.getElementById('container').innerHTML = '';
    chrome.mediaGalleries.getMediaFileSystems({
        interactive: 'yes'
    }, listMediaGalleries);
} 
```

改进后的窗口如下所示。

![enter image description here](img/00060.jpeg)
*带有添加移除媒体库的窗口*

## 8.3　更新媒体库

有时我们需要更新媒体库以让应用自动发现最新的媒体库。

通过`startMediaScan`方法开始更新媒体库¹：

```
chrome.mediaGalleries.startMediaScan(); 
```

^(1 从 Chrome 35 开始支持。)

`startMediaScan`没有然后返回值，也不会调用任何回调函数，因为更新的过程所花费的时间可能非常长，所以要使用`onScanProgress`来监听更新过程：

```
chrome.mediaGalleries.onScanProgress.addListener(function(details){
    //do something with details
}); 
```

其中`details`是一个对象，包含 5 个属性，分别是`type`、`galleryCount`、`audioCount`、`imageCount`和`videoCount`。`type`的可能值有`start`、`cancel`、`finish`和`error`，分别对应于开始更新、取消更新、完成更新和遇到错误。

在更新媒体库的过程中，通过`cancelMediaScan`方法可以随时取消更新：

```
chrome.mediaGalleries.cancelMediaScan(); 
```

同样`cancelMediaScan`方法也没有提供回调函数，而应通过`onScanProgress`监测更新过程中的取消事件。

当`onScanProgress`监测到更新完成事件之后，可以通过`addScanResults`方法向用户展示一个选择添加最新检测到媒体库的窗口：

```
chrome.mediaGalleries.addScanResults(function(mediaFileSystems){
    //do something with mediaFileSystems
}); 
```

`mediaFileSystems`是一个包含多个`FileSystem`的数组，其包含的是应用有权限访问的所有媒体库`FileSystem`，而非只是用户刚刚选择的。

下面我们来将更新媒体库的功能加入到 Media Manager。首先在 HTML 中添加更新按钮、loading 元素和出错提示框：

```
<div id="error">更新失败</div>
<div id="appTitle">Media Manager<span id="edit">&#x3466;</span><span id="scan">&#xf015c;</span>
<div id="loading">
<div class="loading" index="0"></div>
<div class="loading" index="1"></div>
<div class="loading" index="2"></div>
<div class="loading" index="3"></div>
<div class="loading" index="4"></div>
</div>
</div> 
```

之后在 CSS 中添加相应的样式：

```
#appTitle {
    height: 60px;
    line-height: 60px;
    padding: 0 20px;
    font-size: 24px;
    color: #CCC;
    background: #222;
    position: relative;
}

#edit, #scan {
    display: inline-block;
    font-size: 12px;
    font-family: 'iconfont';
    height: 20px;
    line-height: 20px;
    padding: 5px;
    cursor: pointer;
}

@-webkit-keyframes loading {
    0% {
        left: 0;
        opacity: 0;
    }
    5% {
        opacity: 1;
    }
    95% {
        opacity: 1;
    }
    100% {
        left: 100%;
        opacity: 0;
    }
}

.loading {
    width: 5px;
    height: 5px;
    background: #CCC;
    position: absolute;
    opacity: 0;
    -webkit-animation: loading 5s;
    -webkit-animation-timing-function: cubic-bezier(0.1, 0.48, 0.9, 0.52);
    -webkit-animation-iteration-count: infinite;
}

.loading[index="0"] {
    -webkit-animation-delay: 0.3s;
}

.loading[index="1"] {
    -webkit-animation-delay: 0.6s;
}

.loading[index="2"] {
    -webkit-animation-delay: 0.9s;
}

.loading[index="3"] {
    -webkit-animation-delay: 1.2s;
}

.loading[index="4"] {
    -webkit-animation-delay: 1.5s;
}

#loading {
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 5px;
    display: none;
}

#error {
    background: rgba(255, 255, 255, 0.5);
    height: 20px;
    line-height: 20px;
    font-size: 14px;
    color: #222;
    text-align: center;
    display: none;
} 
```

其中为了让 loading 元素位置相对于`appTitle`，将`appTitle`的`position`属性更改为了`relative`。

最后在 JS 中加入相应事件：

```
var scanning = false;

document.getElementById('scan').onclick = function(){
    scanning?
        chrome.mediaGalleries.startMediaScan&&chrome.mediaGalleries.startMediaScan():
        chrome.mediaGalleries.cancelMediaScan&&chrome.mediaGalleries.cancelMediaScan();
}

document.getElementById('error').onclick = function(){
    this.style.display = 'none';
}

chrome.mediaGalleries.onScanProgress&&chrome.mediaGalleries.onScanProgress.addListener(function(details){
    switch(details.type){
        case 'start':
            scanning = true;
            document.getElementById('loading').style.display = 'block';
            break;
        case 'cancel':
            scanning = false;
            document.getElementById('loading').style.display = 'none';
            break;
        case 'finish':
            scanning = false;
            document.getElementById('loading').style.display = 'none';
            chrome.mediaGalleries.addScanResults(listMediaGalleries);
            break;
        case 'error':
            scanning = false;
            document.getElementById('loading').style.display = 'none';
            document.getElementById('error').style.display = 'block';
            break;
    }
}); 
```

其中`scanning`变量用来记录应用是否正在更新媒体库，如果正在更新，当用户点击更新按钮后会停止更新，否则开始更新。

更新媒体库相关方法和事件在部分版本中尚未生效，所以在调用时均先做以判断，防止出现方法未定义的情况发生。

## 8.4　获取媒体文件信息

在 8.1 节中提到过，通过`getMediaFileSystems`方法获取到的`fileSystem`中的`root`属性值就是`Entry`对象，结合第七章的内容就可以对媒体库中的文件进行操作。

通过`getMetadata`方法可以读取出媒体文件相关信息¹：

```
chrome.mediaGalleries.getMetadata(mediaFile, {metadataType: 'all'}, function(metadata){
    //do something with metadata
}); 
```

^(1 目前处于 Beta 分支。)

其中`mediaFile`为`Blob`类型数据。`metadataType`为获取信息的类型，如果不指定默认为`all`，即全部信息，还可以指定为`mimeTypeOnly`来只获取`MIME`。

`metadata`为一个包含媒体信息的对象，完整结构如下：

```
{
    mimeType: MIME 类型,
    height: 视频或图片的高度，单位为像素,
    width: 视频或图片的宽度，单位为像素,
    xResolution: 照片的水平分辨率，用电视线表示,
    yResolution: 照片的垂直分辨率，用电视线表示,
    duration: 视频或音乐的长度，以秒为单位,
    rotation: 视频或图片的旋转角度，以度为单位,
    cameraMake: 图片中的相机制造商,
    cameraModel: 相机模式,
    exposureTimeSeconds: 曝光时间,
    flashFired: 是否开启闪光灯,
    fNumber: 光圈大小,
    focalLengthMm: 焦距，单位为毫米,
    isoEquivalent: 等效 ISO,
    album: 视频或音乐专辑,
    artist: 艺术家,
    comment: 评分,
    copyright: 版权信息,
    disc: 盘片编号,
    genre: 流派,
    language: 语言,
    title: 标题,
    track: 音轨数,
    rawTags: [
        {
            type: 类型，如 mp3、h264,
            tags: 标签
        }
    ]
} 
```

最后值得说明的是，由于已经在 Manifest 中声明了媒体库的读取权限，而不必另外声明`fileSystem`权限用于读取媒体文件。

由于遍历目录和读取、复制、删除文件与文章讲解的内容无关，读者可自行参考第七章内容实现相关功能。Media Manager 实例的源码可以通过[`github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/media_manager`](https://github.com/sneezry/chrome_extensions_and_apps_programming/tree/master/media_manager)下载到。