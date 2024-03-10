# 7.5 文件和二进制数据的操作

*   Blob 对象
*   FileList 对象
*   File 对象
*   FileReader 对象
*   综合实例：显示用户选取的本地图片
*   URL 对象
*   参考链接

历史上，JavaScript 无法处理二进制数据。如果一定要处理的话，只能使用 charCodeAt()方法，一个个字节地从文字编码转成二进制数据，还有一种办法是将二进制数据转成 Base64 编码，再进行处理。这两种方法不仅速度慢，而且容易出错。ECMAScript 5 引入了 Blob 对象，允许直接操作二进制数据。

Blob 对象是一个代表二进制数据的基本对象，在它的基础上，又衍生出一系列相关的 API，用来操作文件。

*   File 对象：负责处理那些以文件形式存在的二进制数据，也就是操作本地文件；
*   FileList 对象：File 对象的网页表单接口；
*   FileReader 对象：负责将二进制数据读入内存内容；
*   URL 对象：用于对二进制数据生成 URL。

## Blob 对象

Blob（Binary Large Object）对象代表了一段二进制数据，提供了一系列操作接口。其他操作二进制数据的 API（比如 File 对象），都是建立在 Blob 对象基础上的，继承了它的属性和方法。

生成 Blob 对象有两种方法：一种是使用 Blob 构造函数，另一种是对现有的 Blob 对象使用 slice 方法切出一部分。

（1）Blob 构造函数，接受两个参数。第一个参数是一个包含实际数据的数组，第二个参数是数据的类型，这两个参数都不是必需的。

```js
var htmlParts = ["<a id=\"a\"><b id=\"b\">hey!<\/b><\/a>"];

var myBlob = new Blob(htmlParts, { "type" : "text\/xml" });
```

下面是一个利用 Blob 对象，生成可下载文件的例子。

```js
var blob = new Blob(["Hello World"]);

var a = document.createElement("a");
a.href = window.URL.createObjectURL(blob);
a.download = "hello-world.txt";
a.textContent = "Download Hello World!";

body.appendChild(a);
```

上面的代码生成了一个超级链接，点击后提示下载文本文件 hello-world.txt，文件内容为“Hello World”。

（2）Blob 对象的 slice 方法，将二进制数据按照字节分块，返回一个新的 Blob 对象。

```js
var newBlob = oldBlob.slice(startingByte, endindByte);
```

下面是一个使用 XMLHttpRequest 对象，将大文件分割上传的例子。

```js
function upload(blobOrFile) {
  var xhr = new XMLHttpRequest();
  xhr.open('POST', '/server', true);
  xhr.onload = function(e) { ... };
  xhr.send(blobOrFile);
}

document.querySelector('input[type="file"]').addEventListener('change', function(e) {
  var blob = this.files[0];

  const BYTES_PER_CHUNK = 1024 * 1024; // 1MB chunk sizes.
  const SIZE = blob.size;

  var start = 0;
  var end = BYTES_PER_CHUNK;

  while(start < SIZE) {
    upload(blob.slice(start, end));

    start = end;
    end = start + BYTES_PER_CHUNK;
  }
}, false);

})();
```

（3）Blob 对象有两个只读属性：

*   size：二进制数据的大小，单位为字节。
*   type：二进制数据的 MIME 类型，全部为小写，如果类型未知，则该值为空字符串。

在 Ajax 操作中，如果 xhr.responseType 设为 blob，接收的就是二进制数据。

## FileList 对象

FileList 对象针对表单的 file 控件。当用户通过 file 控件选取文件后，这个控件的 files 属性值就是 FileList 对象。它在结构上类似于数组，包含用户选取的多个文件。

```js
<input type="file" id="input" onchange="console.log(this.files.length)" multiple />
```

当用户选取文件后，就可以读取该文件。

```js
var selected_file = document.getElementById('input').files[0];
```

采用拖放方式，也可以得到 FileList 对象。

```js
var dropZone = document.getElementById('drop_zone');
dropZone.addEventListener('drop', handleFileSelect, false);

function handleFileSelect(evt) {
    evt.stopPropagation();
    evt.preventDefault();

    var files = evt.dataTransfer.files; // FileList object.

    // ...
}
```

上面代码的 handleFileSelect 是拖放事件的回调函数，它的参数 evt 是一个事件对象，该参数的 dataTransfer.files 属性就是一个 FileList 对象，里面包含了拖放的文件。

## File 对象

File 对象是 FileList 对象的成员，包含了文件的一些元信息，比如文件名、上次改动时间、文件大小和文件类型。它的属性值如下：

*   name：文件名，该属性只读。
*   size：文件大小，单位为字节，该属性只读。
*   type：文件的 MIME 类型，如果分辨不出类型，则为空字符串，该属性只读。
*   lastModifiedDate：文件的上次修改时间。

```js
var selected_file = document.getElementById('input').files[0];

var fileName = selected_file.name;
var fileSize = selected_file.size;
var fileType = selected_file.type;
```

## FileReader 对象

FileReader 对象用于读取文件，即把文件内容读入内存。它的参数是 File 对象或 Blob 对象。

对于不同类型的文件，FileReader 使用不同的方法读取。

*   readAsBinaryString(Blob|File)：返回二进制字符串，该字符串每个字节包含一个 0 到 255 之间的整数。

*   readAsText(Blob|File, opt_encoding) ：返回文本字符串。默认情况下，文本编码格式是'UTF-8'，可以通过可选的格式参数，指定其他编码格式的文本。

*   readAsDataURL(Blob|File)：返回一个基于 Base64 编码的 data-uri 对象。

*   readAsArrayBuffer(Blob|File) ：返回一个 ArrayBuffer 对象。

readAsText 方法用于读取文本文件，它的第一个参数是 File 或 Blob 对象，第二个参数是前一个参数的编码方法，如果省略就默认为 UTF-8 编码。该方法是异步方法，一般监听 onload 事件，用来确定文件是否加载结束，方法是判断 FileReader 实例的 result 属性是否有值。其他三种读取方法，用法与 readAsText 方法类似。

```js
var reader = new FileReader();

reader.onload = function(e) {
  var text = reader.result;
}

reader.readAsText(file, encoding);
```

readAsDataURL 方法返回一个 data URL，它的作用基本上是将文件数据进行 Base64 编码。你可以将返回值设为图像的 src 属性。

```js
var reader = new FileReader();

reader.onload = function(e) {
  var dataURL = reader.result;
}

reader.readAsDataURL(file);
```

readAsBinaryString 方法可以读取任意类型的文件，而不仅仅是文本文件，返回文件的原始的二进制内容。这个方法与 XMLHttpRequest.sendAsBinary 方法结合使用，就可以使用 JavaScript 上传任意文件到服务器。

```js
var reader = new FileReader();

reader.onload = function(e) {
  var rawData = reader.result;
}

reader.readAsBinaryString(file);
```

readAsArrayBuffer 方法读取文件，返回一个类型化数组（ArrayBuffer），即固定长度的二进制缓存数据。在文件操作时（比如将 JPEG 图像转为 PNG 图像），这个方法非常方便。

```js
var reader = new FileReader();

reader.onload = function(e) {
  var arrayBuffer = reader.result;
}

reader.readAsArrayBuffer(file);
```

除了以上四种不同的读取文件方法，FileReader 对象还有一个 abort 方法，用于中止文件上传。

```js
var reader = new FileReader();

reader.abort();
```

FileReader 对象采用异步方式读取文件，可以为一系列事件指定回调函数。

*   onabort 方法：读取中断或调用 reader.abort()方法时触发。
*   onerror 方法：读取出错时触发。
*   onload 方法：读取成功后触发。
*   onloadend 方法：读取完成后触发，不管是否成功。触发顺序排在 onload 或 onerror 后面。
*   onloadstart 方法：读取将要开始时触发。
*   onprogress 方法：读取过程中周期性触发。

下面的代码是如何展示文本文件的内容。

```js
var reader = new FileReader();

reader.onload = function(e){
  console.log(e.target.result);
}

reader.readAsText(blob);
```

onload 事件的回调函数接受一个事件对象，该对象的 target.result 就是文件的内容。

下面是一个使用 readAsDataURL 方法，为 img 元素添加 src 属性的例子。

```js
var reader = new FileReader();

reader.onload = function(e) {
    document.createElement('img').src = e.target.result;

};

reader.readAsDataURL(f);
```

下面是一个 onerror 事件回调函数的例子。

```js
var reader = new FileReader();
reader.onerror = errorHandler;

function errorHandler(evt) {
  switch(evt.target.error.code) {
    case evt.target.error.NOT_FOUND_ERR:
      alert('File Not Found!');
      break;
    case evt.target.error.NOT_READABLE_ERR:
      alert('File is not readable');
      break;
    case evt.target.error.ABORT_ERR:
      break;
    default:
      alert('An error occurred reading this file.');
  };
}
```

下面是一个 onprogress 事件回调函数的例子，主要用来显示读取进度。

```js
var reader = new FileReader();
reader.onprogress = updateProgress;

function updateProgress(evt) {
  if (evt.lengthComputable) {
    var percentLoaded = Math.round((evt.loaded / evt.totalEric Bidelman) * 100);

    var progress = document.querySelector('.percent');
    if (percentLoaded < 100) {
      progress.style.width = percentLoaded + '%';
      progress.textContent = percentLoaded + '%';
    }
  }
}
```

读取大文件的时候，可以利用 Blob 对象的 slice 方法，将大文件分成小段，逐一读取，这样可以加快处理速度。

## 综合实例：显示用户选取的本地图片

假设有一个表单，用于用户选取图片。

```js
<input type="file" name="picture" accept="image/png, image/jpeg"/>
```

一旦用户选中图片，将其显示在 canvas 的函数可以这样写：

```js
document.querySelector('input[name=picture]').onchange = function(e){
     readFile(e.target.files[0]);
}

function readFile(file){

  var reader = new FileReader();

  reader.onload = function(e){
    applyDataUrlToCanvas( reader.result );
  };

  reader.reaAsDataURL(file);
}
```

还可以在 canvas 上面定义拖放事件，允许用户直接拖放图片到上面。

```js
// stop FireFox from replacing the whole page with the file.
canvas.ondragover = function () { return false; };

// Add drop handler
canvas.ondrop = function (e) {
  e.stopPropagation();
  e.preventDefault(); 
  e = e || window.event;
  var files = e.dataTransfer.files;
  if(files){
    readFile(files[0]);
  }
};
```

所有的拖放事件都有一个 dataTransfer 属性，它包含拖放过程涉及的二进制数据。

还可以让 canvas 显示剪贴板中的图片。

```js
document.onpaste = function(e){
  e.preventDefault();
  if(e.clipboardData&&e.clipboardData.items){
    // pasted image
    for(var i=0, items = e.clipboardData.items;i<items.length;i++){
      if( items[i].kind==='file' && items[i].type.match(/^image/) ){
        readFile(items[i].getAsFile());
        break;
      }
    }
  }
  return false;
};
```

## URL 对象

URL 对象用于生成指向 File 对象或 Blob 对象的 URL。

```js
var objecturl =  window.URL.createObjectURL(blob);
```

上面的代码会对二进制数据生成一个 URL，类似于“blob:http%3A//test.com/666e6730-f45c-47c1-8012-ccc706f17191”。这个 URL 可以放置于任何通常可以放置 URL 的地方，比如 img 标签的 src 属性。需要注意的是，即使是同样的二进制数据，每调用一次 URL.createObjectURL 方法，就会得到一个不一样的 URL。

这个 URL 的存在时间，等同于网页的存在时间，一旦网页刷新或卸载，这个 URL 就失效。除此之外，也可以手动调用 URL.revokeObjectURL 方法，使 URL 失效。

```js
window.URL.revokeObjectURL(objectURL);
```

下面是一个利用 URL 对象，在网页插入图片的例子。

```js
var img = document.createElement("img");

img.src = window.URL.createObjectURL(files[0]);

img.height = 60;

img.onload = function(e) {
    window.URL.revokeObjectURL(this.src);
}

body.appendChild(img);

var info = document.createElement("span");

info.innerHTML = files[i].name + ": " + files[i].size + " bytes";

body.appendChild(info);
```

还有一个本机视频预览的例子。

```js
var video = document.getElementById('video');
var obj_url = window.URL.createObjectURL(blob);
video.src = obj_url;
video.play()
window.URL.revokeObjectURL(obj_url);
```

## 参考链接

*   [W3C Working Draft](http://www.w3.org/TR/FileAPI/)
*   Andrew Dodson, [Get Loaded with the File API](http://msdn.microsoft.com/en-gb/magazine/jj835793.aspx)
*   Mozilla Developer Network，[Using files from web applications](https://developer.mozilla.org/en-US/docs/Using_files_from_web_applications)
*   [HTML5 download attribute](http://javascript-reverse.tumblr.com/post/37056936789/html5-download-attribute)
*   Eric Bidelman, [Reading files in JavaScript using the File APIs](http://www.html5rocks.com/en/tutorials/file/dndfiles/)
*   Matt West, [Reading Files Using The HTML5 FileReader API](http://blog.teamtreehouse.com/reading-files-using-the-html5-filereader-api)