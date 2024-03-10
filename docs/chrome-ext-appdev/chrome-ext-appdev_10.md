# 第七章　文件系统

读写本地文件是一个程序最基本的功能，而对于 Web 技术来说，出于安全因素考虑，浏览器一直没有完全将这一功能开放给 JavaScript，直到 HTML5 提出了 FileSystem API。

Chrome 为应用提供了权限更加开放，功能更加强大的一系列文件系统接口，以满足 Chrome 应用作为桌面程序对磁盘读写的需求。在本章将详细为大家讲解选择目录、读取文件和写文件的方法。

要使用 FileSystem API 需要在 Manifest 中声明`fileSystem`权限：

```
permissions: {
    "fileSystem"
} 
```

但如果只声明了上述权限，并不能写入文件及获取目录。如果还需要写入文件和获取目录需要进行如下声明：

```
permissions: {
    {"fileSystem": ["write", "directory"]}
} 
```

值得注意的是，上面的权限声明中请求的权限值为对象型，即`{"fileSystem": ["write", "directory"]}`，而多数情况下是字符串型，如`"storage"`。

## 7.1　目录及文件操作对象

在 C 语言中操作文件时，实际操作的是文件指针，在 Chrome 应用中是通过目录及文件操作对象进行的。我们称目录操作对象为`DirectoryEntry`，文件操作对象为`FileEntry`，两者均继承自`Entry`对象。

`Entry`有五个属性，分别是`filesystem`、`fullPath`、`isDirectory`、`isFile 和 name`。其中`filesystem`是当前`Entry`所在的文件系统，`filesystem`还有两个属性，分别是`name`和`root`，`name`是此文件系统的名称，`root`是此文件系统的根目录 Entry。`fullPath`是当前目录的绝对地址，字符串型。`isDirectory`和`isFile`分别用于标识当前操作对象的类型，对于`DirectoryEntry`来说，两者的值分别为`true`和`false`。`name`是当前目录的名称，字符串型。

另外`DirectoryEntry`还有四种方法，分别是`createReader`、`getDirectory`、`getFile`和`removeRecursively`。`createReader`用于创建新的 DirectoryReader 对象来读取当前目录中的子目录和文件。`getDirectory`用于读取或创建当前目录下的子目录。`getFile`用于读取或创建当前目录下的文件。`removeRecursively`用于删除当前目录下的所有文件和子目录，以及当前目录本身。

![enter image description here](img/00050.jpeg)
*DirectoryEntry 的结构*

`FileEntry`与`DirectoryEntry`有很多类似的地方，如`FileEntry`具有和`DirectoryEntry`一样的五个属性，只不过对于`FileEntry`来说`isDirectory`和`isFile`的值分别为`false`和`true`。

但是`FileEntry`所具有的方法与`DirectoryEntry`不同，`FileEntry`只有两种方法，分别是`createWriter`和`file`。其中`createWriter`用于创建一个新的 FileWriter 对象以用来向当前文件写入数据。`file`方法会返回`File`对象，继承自`Blob`对象（包含文件内容、大小、MIME 类型），包含文件名和最后修改时间。

![enter image description here](img/00051.jpeg)
*FileEntry 的结构*

Chrome 应用中的`fileSystem`接口是对 HTML5 已有的文件系统接口的扩充，它允许 Chrome 应用读写硬盘中用户选择的任意位置，而 HTML5 本身提供的文件系统接口则只能在沙箱中读写文件，并不能获取用户磁盘中真正的目录。

有关 HTML5 文件系统更加详细的说明可以参见 W3C 文档[`www.w3.org/TR/file-system-api/`](http://www.w3.org/TR/file-system-api/)

## 7.2　获取目录及文件操作对象

无论是操作文件还是操作目录，都是对相应的操作对象进行操作，所以第一步都需要获取到目录及文件操作对象。Chrome 应用无法像 C 语言那样通过路径直接操作文件，目录及文件操作对象总是需要通过 Chrome 自带的文件选择窗口获取的。

通过`chooseEntry`方法可以获取到目录及文件操作对象。当`chooseEntry`被执行时，一个文件选择窗口会马上弹出，所以应该让一些事件来触发其运行，比如点击按钮等，否则可能会让用户感到困惑。

```
document.getElementById('openfile').onclick = function(){
    chrome.fileSystem.chooseEntry({}, function(fileEntry){
        console.log(fileEntry);
        //do something with fileEntry
    });
} 
```

上面这段代码会让用户选择一个已存在的文件，并返回此文件对应的操作对象。如果在 Manifest 中声明了写权限，`fileEntry`是可写的，否则是只读的。

在调用`chooseEntry`方法时，我们在上例中传递了一个空对象，这个对象用来定义`chooseEntry`打开的参数，默认情况下会以打开文件的方式获取操作对象。这个定义打开参数的对象完整结构如下：

```
{
    type: 打开类型，包括 openFile、openWritableFile、saveFile 和 openDirectory,
    suggestedName: 建议的文件名，会自动显示在保存窗口的文件名输入框中,
    accepts: [
        {
            description: 此选项的文字描述,
            mimeTypes: [接受的 mime 类型，如"image/jpeg"或"audio/*"],
            extensions: [接受的文件后缀，如"jpg"或"gif"]
        }
    ],
    acceptsAllTypes: 如果设定了接受的指定类型文件，是否接受所有的类型文件,
    acceptsMultiple: 是否接受多个文件，只支持 openFile 和 openWritableFile 的打开方式
} 
```

在参数对象中如果未指定`type`属性，则默认为`openFile`。由于在声明`write`权限后`openFile`方法获取的`FileEntry`可写，所以请考虑避免使用`openWritableFile`，因为在以后`openWritableFile`很可能被`openFile`替代。

但是`saveFile`却无法被`openFile`替代，因为`saveFile`可以创建新的文件，`openFile`则不可以。

将`type`指定为`openDirectory`则可以获取到目录操作对象：

```
document.getElementById('opendirectory').onclick = function(){
    chrome.fileSystem.chooseEntry({type: 'openDirectory'}, function(Entry){
        console.log(Entry);
        //do something with Entry
    });
} 
```

## 7.3　读取文件

在 7.1 节中提到过`FileEntry`的`file`方法可以获取到文件的相关信息，实际上`file`方法返回的是 HTML5 中的`File`类型对象，所以有必要先介绍一下 HTML5 中的`FILE`对象。

HTML5 可以在文件未上传之前在浏览器端获取到文件的相关信息，就是通过 File API。当用户通过文件选择控件选择文件后，JavaScript 就可以通过控件 DOM 的`files`属性获取到对应的 File 对象：

```
document.getElementById('myFile').onchange = function(){
    var file = this.files[0];
    console.log(file);
} 
```

对应的 HTML 为：

```
<input type="file" id="myFile" /> 
```

获取到的 FILE 对象包括文件最后更改日期、文件名、文件大小和文件类型等。

![enter image description here](img/00052.jpeg)
*获取到的 File 对象*

HTML5 还提供了`FileReader`对象，通过`FileReader`可以读取`File`对象对应文件的内容。

```
var reader = new FileReader();
reader.onload = function(){
    console.log(this.result);
}
reader.readAsText(File); 
```

上述代码中`readAsText`是以文本方式读取文件内容，还可以通过`readAsDataURL`方式将文件内容读取成`dataURL`，或者通过`readAsBinaryString`方式将文件内容读取成二进制字符串，以及`readAsArrayBuffer`方法读取二进制原始缓存区。

下面我们回到 Chrome 应用中。首先通过`chooseEntry`方法以`openFile`的方式获取`fileEntry`：

```
chrome.fileSystem.chooseEntry({type: 'openFile'}, function(fileEntry){
    //We'll do something with fileEntry later
}); 
```

之后通过`FileEntry`的`file`方法获取到`File`对象：

```
fileEntry.file(function(file){
    //We'll do something with file later
}); 
```

最后用`FileReader`读取`file`中的内容：

```
var reader = new FileReader();
reader.onload = function(){
    var text = this.result;
    console.log(text);
    //do something with text
}
reader.readAsText(file); 
```

将上面这三个过程连起来就可以得到如下代码：

```
chrome.fileSystem.chooseEntry({type: 'openFile'}, function(fileEntry){
    fileEntry.file(function(file){
        var reader = new FileReader();
        reader.onload = function(){
            var text = this.result;
            console.log(text);
            //do something with text
        }
        reader.readAsText(File);
    });
}); 
```

当然如果读取的文件中，并非是文本类型的数据，可以使用`readAsBinaryString`方式直接读取文件的二进制数据。

![enter image description here](img/00053.jpeg)
*读取文件内容*

## 7.4　遍历目录

通过 Entry 的`createReader`方法可以创建`DirectoryReader`对象，而`DirectoryReader`对象的`readEntries`方法又可以读取出当前目录下的一级子目录和文件，依次类推就可以遍历整个目录。

下面我们来实践写一个遍历目录的函数。

首先通过`chooseEntry`方法获取`Entry`：

```
chrome.fileSystem.chooseEntry({type: 'openDirectory'}, function(Entry) {
    //We'll do something with Entry later
}); 
```

接下来我们来获取`Entry`下的子目录和文件：

```
var dirReader = Entry.createReader();
dirReader.readEntries (function(Entries) {
    //We'll do something with Entries later
}, errorHandler); 
```

获取到`Entries`之后要对其中的每个元素进行判断是目录还是文件，如果是文件直接输出文件名，如果还是目录，则继续遍历：

```
for(var i=0; i<Entries.length; i++){
    //We'll print name of this Entry
    if(Entries[i].isDirectory){
        //We'll get sub Entries for this Entry
    }
} 
```

基本的过程已经搞清楚了，现在开始编写打印`Entry`名的函数。我们希望设计成以下输出格式：

```
The full path of the selected Entry
|-Entry1
| |-sub Entry1
| | |-File1 in sub Entry1
| |-File1 in Entry1
| |-File2 in Entry1
|-File1
|-File2 
```

所以显示`Entry`需要指定当前的目录深度以输出相应的层次格式：

```
function echoEntry(depth, Entry){
    var tree = '|';
    for(var i=0; i<depth-1; i++){
        tree += ' |';
    }
    console.log(tree+'-'+Entry.name);
} 
```

然后我们将获取子目录和文件的代码也封装成一个函数以便复用：

```
function getSubEntries(depth, Entry){
    var dirReader = Entry.createReader();
    dirReader.readEntries (function(Entries) {
        for(var i=0; i<Entries.length; i++){
            echoEntry(depth+1, Entries[i]);
            if(Entries[i].isDirectory){
                getSubEntries(depth+1, Entries[i]);
            }
        }
    }, errorHandler);
} 
```

最后在`chooseEntry`获取到`Entry`之后调用`getSubEntries`函数：

```
chrome.fileSystem.chooseEntry({type: 'openDirectory'}, function(Entry) {
    console.log(Entry.fullPath);
    getSubEntries(0, Entry);
}); 
```

别忘了定义`errorHandler`函数用于抓取错误：

```
function errorHandler(e){
    console.log(e.message);
} 
```

但是细心的读者会发现按照上面的写法会先显示一级目录，而后显示二级目录以此类推，并不是像我们所设计的那样展示实际的目录结构。这是因为`getSubEntries`函数得到的结果是以回调的形式传递的，也就是说`getSubEntries`函数未执行结束并不会阻塞循环体。这个问题只是在显示结果时会造成一点小麻烦，在实际遍历目录时我们并不在意哪些先得到哪些后得到。但为了使本小节的例子更加完善，现将代码修改如下：

```
var loopEntriesButton = document.getElementById('le');

loopEntriesButton.addEventListener('click', function(e) {
    chrome.fileSystem.chooseEntry({type: 'openDirectory'}, function(Entry) {
        document.getElementById('loopEntry').innerText = Entry.fullPath;
        getSubEntries(0, Entry, document.getElementById('loopEntry'));
    });
});

function getSubEntries(depth, Entry, parent){
    var dirReader = Entry.createReader();
    dirReader.readEntries(function(Entries) {
        for(var i=0; i<Entries.length; i++){
            var newParent = document.createElement('div');
            newParent.id = Date.now();
            newParent.innerText = echoEntry(depth+1, Entries[i]);
            parent.appendChild(newParent);
            if(Entries[i].isDirectory){
                getSubEntries(depth+1, Entries[i], newParent);
            }
        }
    }, errorHandler);
}

function echoEntry(depth, Entry){
    var tree = '|';
    for(var i=0; i<depth-1; i++){
        tree += ' |';
    }
    return (tree+'-'+Entry.name);
} 
```

对应的 HTML 为：

```
<input type="button" id="le" value="Loop Entries" />
<div id="loopEntry"></div> 
```

最终运行的结果如下图所示：

![enter image description here](img/00054.jpeg)
*遍历目录所得到的结果*

## 7.5　创建及删除目录和文件

在 7.1 节中介绍过，Entry 的`getDirectory`和`getFile`方法可以获取和创建子目录和文件，在本节将主要讲解创建目录和文件。同时也会介绍删除目录和文件的方法。

在调用`getDirectory`方法时，如果在参数对象中指定`create`属性为`true`，则会创建相应的子目录，如：

```
chrome.fileSystem.chooseEntry({type: 'openDirectory'}, function(Entry) {
    Entry.getDirectory('new_folder', {create: true}, function(subEntry) {
        //We'll do something with subEntry later
    }, errorHandler);
}); 
```

这将在用户所选择的目录下创建一个名为 new_folder 的子文件夹。同时也可以指定参数对象`exclusive`属性为`true`，这将避免创建同名子目录——如果一旦创建的目录名与一已存在的子目录相同，会返回错误，而不会自动使用其他目录名。

![enter image description here](img/00055.jpeg)
*当指定 exclusive 为 true，且创建同名目录时会抛出错误*

同样在调用`getFile`方法时，参数对象中指定`create`属性为`true`会创建文件：

```
chrome.fileSystem.chooseEntry({type: 'openDirectory'}, function(Entry) {
    Entry.getFile('log.txt', {create: true}, function(fileEntry) {
        //We'll do something with fileEntry later
    }, errorHandler);
}); 
```

创建文件与创建目录基本相同，指定`exclusive`属性为`true`时，创建同名文件也会引起错误，所得到的错误信息与目录相同。

除了在用户选择的目录下创建文件外，也可以指定`chooseEntry`方法的打开类型为`saveFile`，这样用户看到的将不是一个目录选择窗口，而是一个另存为窗口：

```
chrome.fileSystem.chooseEntry({
    type: 'saveFile',
    suggestedName: 'log.txt'
}, function(fileEntry) {
    //We'll do something with fileEntry later
}); 
```

通过指定`suggestedName`的值可以在另存为窗口中给出默认文件名，但用户可以自行更改这个文件名。

![enter image description here](img/00056.jpeg)
*带有默认文件名的另存为窗口*

Entry 和 FileEntry 的`remove`方法可以删除自身：

```
chrome.fileSystem.chooseEntry({type: 'openDirectory'}, function(Entry) {
    Entry.getDirectory('new_folder', {}, function(subEntry) {
        subEntry.remove(function(){
            console.log('Directory has been removed.');
        }, errorHandler);
    }, errorHandler);

    Entry.getFile('log.txt', {}, function(fileEntry) {
        fileEntry.remove(function(){
            console.log('File has been removed.');
        }, errorHandler);
    }, errorHandler);
}); 
```

对于目录来说，只有当目录不包含任何文件和子目录的时候`remove`方法才会调用成功，否则会报错。如果想删除包含内容的目录，需要使用`removeRecursively`方法：

```
chrome.fileSystem.chooseEntry({type: 'openDirectory'}, function(Entry) {
    Entry.getDirectory('new_folder', {}, function(subEntry) {
        subEntry.removeRecursively(function(){
            console.log('Directory has been removed.');
        }, errorHandler);
    }, errorHandler);
}); 
```

## 7.6　写入文件

通过 FileEntry 的`createWriter`方法可以获取 FileWriter 对象，通过 FileWriter 可以对文件进行写操作：

```
fileEntry.createWriter(function(fileWriter) {
    //We'll do something with fileWriter later
}, errorHandler); 
```

对于 FileEntry，可以通过 Entry 的`getFile`方法获取，也可以直接通过指定 s`aveFile`类型的`chooseEntry`获得：

```
chrome.fileSystem.chooseEntry({type: 'openDirectory'}, function(Entry) {
    Entry.getFile('log.txt', {}, function(fileEntry) {
        fileEntry.createWriter(function(fileWriter) {
            //We'll do something with fileWriter later
        }, errorHandler);
    }, errorHandler);
}); 
```

或

```
chrome.fileSystem.chooseEntry({
    type: 'saveFile',
    suggestedName: 'log.txt'
}, function(fileEntry) {
    fileEntry.createWriter(function(fileWriter) {
        //We'll do something with fileWriter later
    }, errorHandler);
}); 
```

由于之后的操作都是针对 FileWriter 的，下面将只讲解与 FileWriter 相关的内容。

### 7.6.1　Typed Array

Typed Array（类型数组）为 JavaScript 直接处理原始二进制数据提供了接口。随着 HTML5 功能的增加，JavaScript 处理的数据已不仅仅局限于数字和字符串等基本类型，也会处理图像、声音、视频等更加复杂的数据，所以 JavaScript 需要一个直接操作原始二进制数据的接口。有关 Typed Array 草案和 WebGL 的内容可以通过[`www.khronos.org/registry/typedarray/specs/latest/`](http://www.khronos.org/registry/typedarray/specs/latest/)查看。

Typed Array 接口定义了一类固定长度的，可以直接获取缓存区数据的数组类型，`ArrayBuffer`类型。可以通过`new ArrayBuffer(length)`来创建一个长度为`length`字节的二进制缓存区，如：

```
var buf = new ArrayBuffer(8); 
```

创建了一个长度为 8 字节（64 位）的`ArrayBuffer`。

`ArrayBuffer`类型的数据不可以直接读写，需要再构建`ArrayBufferView`类型数据才可以进行操作。那么`ArrayBuffer`和`ArrayBufferView`是什么样的关系呢？`ArrayBuffer`是最原始的二进制数据，它没有附加任何信息，如数据是如何构造的。而`ArrayBufferView`则指定了原始二进制数据应该被如何看待——多少位被看做一个基本处理单元。为更加直观阐述这一关系，现举例如下：

```
var buf = new ArrayBuff(8); 
```

此时对应于`buf`的数据是 8 字节（64 位），数据结构为：

```
+----+-+-+-+-+-+-+-+-+
|byte|0|1|2|3|4|5|6|7|
+----+-+-+-+-+-+-+-+-+ 
```

如果通过`Uint32Array`这一`ArrayBufferView`来格式化`buf`数据：

```
var uintBuf = new Uint32Array(buf); 
```

则`uintBuf`的数据结构为：

```
+----+-+-+-+-+-+-+-+-+
|byte|0|1|2|3|4|5|6|7|
+----+-+-+-+-+-+-+-+-+
|uint|   0   |   1   |
+----+-+-+-+-+-+-+-+-+ 
```

在处理`uintBuf`数据时，JavaScript 会自动以一个`uint`为单位读写数组。

需要说明的是，创建`ArrayBufferView`并不会改变`ArrayBuffer`数据，它只是定义了`ArrayBuffer`的操作方式，实际上多个`ArrayBufferView`可以指向同一个`ArrayBuffer`，但最终对`ArrayBufferView`的操作都是对`ArrayBuffer`数据的操作。

`ArrayBufferView`一共有 8 种，分别是`Int8Array`、`Uint8Array`、`Int16Array`、`Uint16Array`、`Int32Array`、`Uint32Array`、`Float32Array`和`Float64Array`，名称中间的数字代表格式化后数据基本单元位（bit）的长度。

`ArrayBufferView`也可以指定`ArrayBuffer`中数据的起止位置，如：

```
var partUintBuf = new Uint8Array(buf, 3, 4); 
```

这样`partUintBuf[0]`将指向`buf`的第 4 个字节，`partUintBuf[1]`将指向`buf`的第 5 个字节……而`partUintBuf`这个数组只有 4 个元素。

下面我们来将一个`ArrayBuffer`数据按照字符串的方式读取出来。首先在 JavaScript 中字符类型（`String`）是占 16 位的，所以应该使用`Uint16Array`这个`ArrayBufferView`指定读取格式：

```
var stringBuf = new Uint16Array(buf); 
```

这样`stringBuf`中的每个元素保存的就都是字符的 Unicode 码了，再使用`fromCharCode`方法转换成字符就可以了。但是`fromCharCode`方法需要传递多个参数：

```
String.fromCharCode(num0, num1, ..., numX); 
```

而不是一个数组：

```
String.fromCharCode([num0, num1, ..., numX]); 
```

可是我们获得的`stringBuf`是一个数组，所以不能直接传给`fromCharCode`。当然可以使用一个循环将每个 Unicode 码进行转换，之后再拼接起来，但有简单的方法，`apply`方法。`apply`方法可以将一个对象的方法应用到另一个对象上，同时改变原方法中的`this`替换为指定的值。虽然看着有点乱，但这不是我们关心的，重要的是它可以自动将一个数组中的元素转化为函数的参数列表，即`foo.apply(null, [a, b, c])`等同于`foo(a, b, c)`，这正是我们所需要的。所以将`stringBuf`转换为字符串的方法就是：

```
String.fromCharCode.apply(null, stringBuf); 
```

将`stringBuf`变量省略，就可以得到如下`ArrayBuffer`转换为`String`的函数：

```
function ab2str(buf){
    return String.fromCharCode.apply(null, new Uint16Array(buf));
} 
```

### 7.6.2　Blob 对象

`Blob`对象是对二进制数据的封装，它介于`ArrayBuffer`和应用层面数据之间。创建`Blob`对象非常简单，只需指定数据内容和数据类型即可：

```
var str = 'Internet Explorer is a good tool to download Chrome.';
var oneBlob = new Blob([str], {type: 'text/plain'}); 
```

值得注意的是创建`Blob`对象时的第一个参数永远都是一个数组，即使只有一个元素。第二个参数是创建`Blob`对象的可选参数，目前只包含`type`属性，指定`Blob`对象数据的类型，值为`MIME`。如果不指定`type`的值，则`type`默认为一个空字符串。

创建`Blob`对象时可以通过字符串指定数据，如上例代码；也可以通过`ArrayBuffer`、`ArrayBufferView`和`Blob`类型数据，还可以是它们的组合，如：

```
var str = 'Internet Explorer is a good tool to download Chrome.';
var ab = new ArrayBuffer(8);
var abv = new Unit16Array(ab, 2, 2);
var oneBlob = new Blob([str], {type: 'text/plain'});
var anotherBlob = new Blob([ab, abv, oneBlob]); 
```

当通过一个`Blob`被作为另一个`Blob`的数据时，它的类型会被忽略，即使数据数组中只有它一个元素时，如：

```
var oneBlob = new Blob(['Hello World.'], {type: 'text/plain'});
var anotherBlob = new Blob([oneBlob]); 
```

`anotherBlob`的类型不是`'text/plain'`，而是一个空字符串（因为创建时没有指定）。

`Blob`对象有两个属性，分别是`size`和`type`，其中`size`为 Blob 数据的字节长度，`type`为指定的数据类型，两者均只读。

`Blob`对象还有两种方法，分别是`slice`和`close`。`slice`方法与 String 中的分割非常像，只不过在`Blob`中分割的是二进制数据。如：

```
var oneBlob = new Blob(['Hello World.'], {type: 'text/plain'});
var anotherBlob = oneBlob.slice(2, 4, 'text/plain'); 
```

`slice`方法不会将原始`Blob`对象的类型传递给新的`Blob`对象，如果不指定新的 Blob 对象类型，其类型是一个空字符串。

`close`方法用于永久删除`Blob`对象释放空间，一旦`Blob`被关闭，它将永远无法被再次调用。

### 7.6.3　FileWriter 对象

在本节开始介绍过，通过`FileWriter`可以对文件进行写操作，下面来详细介绍`FileWriter`相关的内容。

`FileWriter`有两个属性，分别是`length`和`position`。其中`length`为文件的长度，`position`为指针的当前位置，即在文件中写入下一个数据的位置。两者均只读。

另外`FileWriter`还有三种方法，分别是`write`、`seek`和`truncate`。其中`write`方法用来写入数据，数据类型为`Blob`。如：

```
fileWriter.write(new Blob(['Hello World'], {type: 'text/plain'})); 
```

可以通过`onwrite`和`onwriteend`监听数据开始写入和写入完毕事件：

```
fileWriter.onwrite = function(){
    console.log('Write begin.');
}

fileWriter.onwriteend = function(){
    console.log('Write complete.');
} 
```

`seek`方法用于移动指针到文件指定位置，之后的写操作将从指针指向的位置开始。如果`seek`给出的偏移量为负数，则将指针移动到距文件末端`n`个字节的位置。如果`seek`给出的偏移量为负数且绝对值比文件长度大，则将指针指向`0`。如果偏移量比文件长度大，则指向文件末端。

```
//set position to beginning of the file
fileWriter.seek(0);

//set position to subtracting 5 from file length
fileWriter.seek(-5);

//set position to end of the file
fileWriter.seek(fileWriter.length); 
```

`truncate`方法用于更改文件长度，如果文件之前的长度比给定的值小，则以`0`填补，如果比给定的大，则舍弃超出部分的数据。

注意，如果`truncate`将文件长度缩小，而文件的指针又处于更改长度后文件范围之外（如将文件长度更改为 10，而文件指针的位置在 20），那么新写入的数据将不会出现在文件中！在调用`truncate`方法后，一定记得检查指针位置是否依然在文件内部¹。

^(1 W3C 标准中指明`truncate`的值必须大于当前指针位置，但实际发现突破这一限制时，Chrome 依然可以成功执行。)

结合前面的内容，我们就可以得到完整的写入文件的代码了：

```
chrome.fileSystem.chooseEntry({
    type: 'saveFile',
    suggestedName: 'log.txt'
}, function(fileEntry) {
    fileEntry.createWriter(function(fileWriter) {
        fileWriter.write(new Blob(['Hello World'], {type: 'text/plain'}));
    }, errorHandler);
}); 
```

## 7.7　复制及移动目录和文件

Entry 和 FileEntry 均有`copyTo`和`moveTo`方法用来复制和移动目录和文件。

```
Entry.copyTo(newEntry, 'new_Entry_name', function(copiedEntry){
    console.log('Entry moved.');
}, errorHandler);

Entry.moveTo(newEntry, 'new_Entry_name', function(movedEntry){
    console.log('Entry copied.');
}, errorHandler);

fileEntry.copyTo(newEntry, 'new_fileEntry_name', function(copiedFileEntry){
    console.log('fileEntry copied.');
}, errorHandler);

fileEntry.moveTo(newEntry, 'new_fileEntry_name', function(movedFileEntry){
    console.log('fileEntry moved.');
}, errorHandler); 
```

如果不指定新的名称，则使用目录和文件原来的名称。

对于`moveTo`方法，不可以：

*   将目录移动到自身路径或其子目录路径下；
*   在其父系目录下移动且不指定新的名称；
*   将文件移动到已被其他目录占用的路径；
*   将目录移动到已被其他文件占用的路径；
*   将目录移动到一个非空目录占用的路径。

对于`copyTo`方法，不可以：

*   将一个目录复制到自身路径或其子目录路径下；
*   在其父系目录下复制且不指定新的名称；
*   将文件复制到已被其他目录占用的路径；
*   将目录复制到已被其他文件占用的路径；
*   将目录复制到一个非空目录占用的路径。