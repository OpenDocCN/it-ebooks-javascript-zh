# 3.10 ArrayBuffer：类型化数组

*   分配内存
*   视图
    *   视图的生成
    *   视图的操作
    *   复合视图
    *   DataView 视图
    *   应用
        *   Ajax
        *   Canvas
        *   File
    *   参考链接

类型化数组是 JavaScript 操作二进制数据的一个接口。

这要从 WebGL 项目的诞生说起，所谓 WebGL，就是指浏览器与显卡之间的通信接口，为了满足 JavaScript 与显卡之间大量的、实时的数据交换，它们之间的数据通信必须是二进制的，而不能是传统的文本格式。

比如，以文本格式传递一个 32 位整数，两端的 JavaScript 脚本与显卡都要进行格式转化，将非常耗时。这时要是存在一种机制，可以像 C 语言那样，直接操作字节，然后将 4 个字节的 32 位整数，以二进制形式原封不动地送入显卡，脚本的性能就会大幅提升。

类型化数组（Typed Array）就是在这种背景下诞生的。它很像 C 语言的数组，允许开发者以数组下标的形式，直接操作内存。有了类型化数组以后，JavaScript 的二进制数据处理功能增强了很多，接口之间完全可以用二进制数据通信。

## 分配内存

类型化数组是建立在 ArrayBuffer 对象的基础上的。它的作用是，分配一段可以存放数据的连续内存区域。

```
var buf = new ArrayBuffer(32);
```

上面代码生成了一段 32 字节的内存区域。

ArrayBuffer 对象的 byteLength 属性，返回所分配的内存区域的字节长度。

```
var buffer = new ArrayBuffer(32);
buffer.byteLength
// 32
```

如果要分配的内存区域很大，有可能分配失败（因为没有那么多的连续空余内存），所以有必要检查是否分配成功。

```
if (buffer.byteLength === n) {
  // 成功
} else {
  // 失败
}
```

ArrayBuffer 对象有一个 slice 方法，允许将内存区域的一部分，拷贝生成一个新的 ArrayBuffer 对象。

```
var buffer = new ArrayBuffer(8);
var newBuffer = buffer.slice(0,3);
```

上面代码拷贝 buffer 对象的前 3 个字节，生成一个新的 ArrayBuffer 对象。slice 方法其实包含两步，第一步是先分配一段新内存，第二步是将原来那个 ArrayBuffer 对象拷贝过去。

slice 方法接受两个参数，第一个参数表示拷贝开始的字节序号，第二个参数表示拷贝截止的字节序号。如果省略第二个参数，则默认到原 ArrayBuffer 对象的结尾。

除了 slice 方法，ArrayBuffer 对象不提供任何直接读写内存的方法，只允许在其上方建立视图，然后通过视图读写。

## 视图

### 视图的生成

ArrayBuffer 作为内存区域，可以存放多种类型的数据。不同数据有不同的存储方式，这就叫做“视图”。目前，JavaScript 提供以下类型的视图：

*   Int8Array：8 位有符号整数，长度 1 个字节。
*   Uint8Array：8 位无符号整数，长度 1 个字节。
*   Int16Array：16 位有符号整数，长度 2 个字节。
*   Uint16Array：16 位无符号整数，长度 2 个字节。
*   Int32Array：32 位有符号整数，长度 4 个字节。
*   Uint32Array：32 位无符号整数，长度 4 个字节。
*   Float32Array：32 位浮点数，长度 4 个字节。
*   Float64Array：64 位浮点数，长度 8 个字节。

每一种视图都有一个 BYTES_PER_ELEMENT 常数，表示这种数据类型占据的字节数。

```
Int8Array.BYTES_PER_ELEMENT // 1
Uint8Array.BYTES_PER_ELEMENT // 1
Int16Array.BYTES_PER_ELEMENT // 2
Uint16Array.BYTES_PER_ELEMENT // 2
Int32Array.BYTES_PER_ELEMENT // 4
Uint32Array.BYTES_PER_ELEMENT // 4
Float32Array.BYTES_PER_ELEMENT // 4
Float64Array.BYTES_PER_ELEMENT // 8
```

每一种视图都是一个构造函数，有多种方法可以生成：

（1）在 ArrayBuffer 对象之上生成视图。

同一个 ArrayBuffer 对象之上，可以根据不同的数据类型，建立多个视图。

```
// 创建一个 8 字节的 ArrayBuffer
var b = new ArrayBuffer(8);

// 创建一个指向 b 的 Int32 视图，开始于字节 0，直到缓冲区的末尾
var v1 = new Int32Array(b);

// 创建一个指向 b 的 Uint8 视图，开始于字节 2，直到缓冲区的末尾
var v2 = new Uint8Array(b, 2);

// 创建一个指向 b 的 Int16 视图，开始于字节 2，长度为 2
var v3 = new Int16Array(b, 2, 2);
```

上面代码在一段长度为 8 个字节的内存（b）之上，生成了三个视图：v1、v2 和 v3。视图的构造函数可以接受三个参数：

*   第一个参数：视图对应的底层 ArrayBuffer 对象，该参数是必需的。
*   第二个参数：视图开始的字节序号，默认从 0 开始。
*   第三个参数：视图包含的数据个数，默认直到本段内存区域结束。

因此，v1、v2 和 v3 是重叠：v1[0]是一个 32 位整数，指向字节 0～字节 3；v2[0]是一个 8 位无符号整数，指向字节 2；v3[0]是一个 16 位整数，指向字节 2～字节 3。只要任何一个视图对内存有所修改，就会在另外两个视图上反应出来。

（2）直接生成。

视图还可以不通过 ArrayBuffer 对象，直接分配内存而生成。

```
var f64a = new Float64Array(8);
f64a[0] = 10;
f64a[1] = 20;
f64a[2] = f64a[0] + f64a[1];
```

上面代码生成一个 8 个成员的 Float64Array 数组（共 64 字节），然后依次对每个成员赋值。这时，视图构造函数的参数就是成员的个数。可以看到，视图数组的赋值操作与普通数组的操作毫无两样。

（3）将普通数组转为视图数组。

将一个数据类型符合要求的普通数组，传入构造函数，也能直接生成视图。

```
var typedArray = new Uint8Array( [ 1, 2, 3, 4 ] );
```

上面代码将一个普通的数组，赋值给一个新生成的 8 位无符号整数的视图数组。

视图数组也可以转换回普通数组。

```
var normalArray = Array.apply( [], typedArray );
```

### 视图的操作

建立了视图以后，就可以进行各种操作了。这里需要明确的是，视图其实就是普通数组，语法完全没有什么不同，只不过它直接针对内存进行操作，而且每个成员都有确定的数据类型。所以，视图就被叫做“类型化数组”。

（1）数组操作

普通数组的操作方法和属性，对类型化数组完全适用。

```
var buffer = new ArrayBuffer(16);

var int32View = new Int32Array(buffer);

for (var i=0; i<int32View.length; i++) {
  int32View[i] = i*2;
}
```

上面代码生成一个 16 字节的 ArrayBuffer 对象，然后在它的基础上，建立了一个 32 位整数的视图。由于每个 32 位整数占据 4 个字节，所以一共可以写入 4 个整数，依次为 0，2，4，6。

如果在这段数据上接着建立一个 16 位整数的视图，则可以读出完全不一样的结果。

```
var int16View = new Int16Array(buffer);

for (var i=0; i<int16View.length; i++) {
  console.log("Entry " + i + ": " + int16View[i]);
}
// Entry 0: 0
// Entry 1: 0
// Entry 2: 2
// Entry 3: 0
// Entry 4: 4
// Entry 5: 0
// Entry 6: 6
// Entry 7: 0
```

由于每个 16 位整数占据 2 个字节，所以整个 ArrayBuffer 对象现在分成 8 段。然后，由于 x86 体系的计算机都采用小端字节序（little endian），相对重要的字节排在后面的内存地址，相对不重要字节排在前面的内存地址，所以就得到了上面的结果。

比如，一个占据四个字节的 16 进制数 0x12345678，决定其大小的最重要的字节是“12”，最不重要的是“78”。小端字节序将最不重要的字节排在前面，储存顺序就是 78563412；大端字节序则完全相反，将最重要的字节排在前面，储存顺序就是 12345678。目前，所有个人电脑几乎都是小端字节序，所以类型化数组内部也采用小端字节序读写数据，或者更准确的说，按照本机操作系统设定的字节序读写数据。

这并不意味大端字节序不重要，事实上，很多网络设备和特定的操作系统采用的是大端字节序。这就带来一个严重的问题：如果一段数据是大端字节序，类型化数组将无法正确解析，因为它只能处理小端字节序！为了解决这个问题，JavaScript 引入 DataView 对象，可以设定字节序，下文会详细介绍。

下面是另一个例子。

```
// 假定某段 buffer 包含如下字节 [0x02, 0x01, 0x03, 0x07]
// 计算机采用小端字节序
var uInt16View = new Uint16Array(buffer);

// 比较运算 
if (bufView[0]===258) {
     console.log("ok");
}

// 赋值运算
uInt16View[0] = 255;    // 字节变为[0xFF, 0x00, 0x03, 0x07]
uInt16View[0] = 0xff05; // 字节变为[0x05, 0xFF, 0x03, 0x07]
uInt16View[1] = 0x0210; // 字节变为[0x05, 0xFF, 0x10, 0x02]
```

总之，与普通数组相比，类型化数组的最大优点就是可以直接操作内存，不需要数据类型转换，所以速度快得多。

（2）buffer 属性

类型化数组的 buffer 属性，返回整段内存区域对应的 ArrayBuffer 对象。该属性为只读属性。

```
var a = new Float32Array(64);
var b = new Uint8Array(a.buffer);
```

上面代码的 a 对象和 b 对象，对应同一个 ArrayBuffer 对象，即同一段内存。

（3）byteLength 属性和 byteOffset 属性

byteLength 属性返回类型化数组占据的内存长度，单位为字节。byteOffset 属性返回类型化数组从底层 ArrayBuffer 对象的哪个字节开始。这两个属性都是只读属性。

```
var b = new ArrayBuffer(8);

var v1 = new Int32Array(b);
var v2 = new Uint8Array(b, 2);
var v3 = new Int16Array(b, 2, 2);

v1.byteLength // 8
v2.byteLength // 6
v3.byteLength // 4

v1.byteOffset // 0
v2.byteOffset // 2
v3.byteOffset // 2
```

注意将 byteLength 属性和 length 属性区分，前者是字节长度，后者是成员长度。

```
var a = new Int16Array(8);

a.length // 8
a.byteLength // 16
```

（4）set 方法

类型化数组的 set 方法用于复制数组，也就是将一段内容完全复制到另一段内存。

```
var a = new Uint8Array(8);
var b = new Uint8Array(8);

b.set(a);
```

上面代码复制 a 数组的内容到 b 数组，它是整段内存的复制，比一个个拷贝成员的那种复制快得多。set 方法还可以接受第二个参数，表示从 b 对象哪一个成员开始复制 a 对象。

```
var a = new Uint16Array(8);
var b = new Uint16Array(10);

b.set(a,2)
```

上面代码的 b 数组比 a 数组多两个成员，所以从 b[2]开始复制。

（5）subarray 方法

subarray 方法是对于类型化数组的一部分，再建立一个新的视图。

```
var a = new Uint16Array(8);
var b = a.subarray(2,3);

a.byteLength // 16
b.byteLength // 2
```

subarray 方法的第一个参数是起始的成员序号，第二个参数是结束的成员序号（不含该成员），如果省略则包含剩余的全部成员。所以，上面代码的 a.subarray(2,3)，意味着 b 只包含 a[2]一个成员，字节长度为 2。

（6）ArrayBuffer 与字符串的互相转换

ArrayBuffer 转为字符串，或者字符串转为 ArrayBuffer，有一个前提，即字符串的编码方法是确定的。假定字符串采用 UTF-16 编码（JavaScript 的内部编码方式），可以自己编写转换函数。

```
// ArrayBuffer 转为字符串，参数为 ArrayBuffer 对象
function ab2str(buf) {
   return String.fromCharCode.apply(null, new Uint16Array(buf));
}

// 字符串转为 ArrayBuffer 对象，参数为字符串
function str2ab(str) {
    var buf = new ArrayBuffer(str.length*2); // 每个字符占用 2 个字节
    var bufView = new Uint16Array(buf);
    for (var i=0, strLen=str.length; i<strLen; i++) {
         bufView[i] = str.charCodeAt(i);
    }
    return buf;
}
```

### 复合视图

由于视图的构造函数可以指定起始位置和长度，所以在同一段内存之中，可以依次存放不同类型的数据，这叫做“复合视图”。

```
var buffer = new ArrayBuffer(24);

var idView = new Uint32Array(buffer, 0, 1);
var usernameView = new Uint8Array(buffer, 4, 16);
var amountDueView = new Float32Array(buffer, 20, 1);
```

上面代码将一个 24 字节长度的 ArrayBuffer 对象，分成三个部分：

*   字节 0 到字节 3：1 个 32 位无符号整数
*   字节 4 到字节 19：16 个 8 位整数
*   字节 20 到字节 23：1 个 32 位浮点数

这种数据结构可以用如下的 C 语言描述：

```
struct someStruct {
  unsigned long id;
  char username[16];
  float amountDue;
};
```

## DataView 视图

如果一段数据包括多种类型（比如服务器传来的 HTTP 数据），这时除了建立 ArrayBuffer 对象的复合视图以外，还可以通过 DataView 视图进行操作。

DataView 视图提供更多操作选项，而且支持设定字节序。本来，在设计目的上，ArrayBuffer 对象的各种类型化视图，是用来向网卡、声卡之类的本机设备传送数据，所以使用本机的字节序就可以了；而 DataView 的设计目的，是用来处理网络设备传来的数据，所以大端字节序或小端字节序是可以自行设定的。

DataView 本身也是构造函数，接受一个 ArrayBuffer 对象作为参数，生成视图。

```
DataView(ArrayBuffer buffer [, 字节起始位置 [, 长度]]);
```

下面是一个实例。

```
var buffer = new ArrayBuffer(24);

var dv = new DataView(buffer);
```

DataView 视图提供以下方法读取内存：

*   getInt8：读取 1 个字节，返回一个 8 位整数。
*   getUint8：读取 1 个字节，返回一个无符号的 8 位整数。
*   getInt16：读取 2 个字节，返回一个 16 位整数。
*   getUint16：读取 2 个字节，返回一个无符号的 16 位整数。
*   getInt32：读取 4 个字节，返回一个 32 位整数。
*   getUint32：读取 4 个字节，返回一个无符号的 32 位整数。
*   getFloat32：读取 4 个字节，返回一个 32 位浮点数。
*   getFloat64：读取 8 个字节，返回一个 64 位浮点数。

这一系列 get 方法的参数都是一个字节序号，表示从哪个字节开始读取。

```
var buffer = new ArrayBuffer(24);
var dv = new DataView(buffer);

// 从第 1 个字节读取一个 8 位无符号整数
var v1 = dv.getUint8(0);

// 从第 2 个字节读取一个 16 位无符号整数
var v2 = dv.getUint16(1); 

// 从第 4 个字节读取一个 16 位无符号整数
var v3 = dv.getUint16(3);
```

上面代码读取了 ArrayBuffer 对象的前 5 个字节，其中有一个 8 位整数和两个十六位整数。

如果一次读取两个或两个以上字节，就必须明确数据的存储方式，到底是小端字节序还是大端字节序。默认情况下，DataView 的 get 方法使用大端字节序解读数据，如果需要使用小端字节序解读，必须在 get 方法的第二个参数指定 true。

```
// 小端字节序
var v1 = dv.getUint16(1, true);

// 大端字节序
var v2 = dv.getUint16(3, false);

// 大端字节序
var v3 = dv.getUint16(3);
```

DataView 视图提供以下方法写入内存：

*   setInt8：写入 1 个字节的 8 位整数。
*   setUint8：写入 1 个字节的 8 位无符号整数。
*   setInt16：写入 2 个字节的 16 位整数。
*   setUint16：写入 2 个字节的 16 位无符号整数。
*   setInt32：写入 4 个字节的 32 位整数。
*   setUint32：写入 4 个字节的 32 位无符号整数。
*   setFloat32：写入 4 个字节的 32 位浮点数。
*   setFloat64：写入 8 个字节的 64 位浮点数。

这一系列 set 方法，接受两个参数，第一个参数是字节序号，表示从哪个字节开始写入，第二个参数为写入的数据。对于那些写入两个或两个以上字节的方法，需要指定第三个参数，false 或者 undefined 表示使用大端字节序写入，true 表示使用小端字节序写入。

```
// 在第 1 个字节，以大端字节序写入值为 25 的 32 位整数
dv.setInt32(0, 25, false); 

// 在第 5 个字节，以大端字节序写入值为 25 的 32 位整数
dv.setInt32(4, 25); 

// 在第 9 个字节，以小端字节序写入值为 2.5 的 32 位浮点数
dv.setFloat32(8, 2.5, true);
```

如果不确定正在使用的计算机的字节序，可以采用下面的判断方式。

```
var littleEndian = (function() {
  var buffer = new ArrayBuffer(2);
  new DataView(buffer).setInt16(0, 256, true);
  return new Int16Array(buffer)[0] === 256;
})();
```

如果返回 true，就是小端字节序；如果返回 false，就是大端字节序。

## 应用

### Ajax

传统上，服务器通过 Ajax 操作只能返回文本数据。XMLHttpRequest 第二版允许服务器返回二进制数据，这时分成两种情况。如果明确知道返回的二进制数据类型，可以把返回类型（responseType）设为 arraybuffer；如果不知道，就设为 blob。

```
xhr.responseType = 'arraybuffer';
```

如果知道传回来的是 32 位整数，可以像下面这样处理。

```
xhr.onreadystatechange = function () {
if (req.readyState === 4 ) {
    var arrayResponse = xhr.response;
    var dataView = new DataView(arrayResponse);
    var ints = new Uint32Array(dataView.byteLength / 4);

    xhrDiv.style.backgroundColor = "#00FF00";
    xhrDiv.innerText = "Array is " + ints.length + "uints long";
    }
}
```

### Canvas

网页 Canvas 元素输出的二进制像素数据，就是类型化数组。

```
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');

var imageData = ctx.getImageData(0,0, 200, 100);
var typedArray = imageData.data;
```

需要注意的是，上面代码的 typedArray 虽然是一个类型化数组，但是它的视图类型是一种针对 Canvas 元素的专有类型 Uint8ClampedArray。这个视图类型的特点，就是专门针对颜色，把每个字节解读为无符号的 8 位整数，即只能取值 0～255，而且发生运算的时候自动过滤高位溢出。这为图像处理带来了巨大的方便。

举例来说，如果把像素的颜色值设为 Uint8Array 类型，那么乘以一个 gamma 值的时候，就必须这样计算：

```
u8[i] = Math.min(255, Math.max(0, u8[i] * gamma));
```

因为 Uint8Array 类型对于大于 255 的运算结果（比如 0xFF+1），会自动变为 0x00，所以图像处理必须要像上面这样算。这样做很麻烦，而且影响性能。如果将颜色值设为 Uint8ClampedArray 类型，计算就简化许多。

```
pixels[i] *= gamma;
```

Uint8ClampedArray 类型确保将小于 0 的值设为 0，将大于 255 的值设为 255。注意，IE 10 不支持该类型。

### File

如果知道一个文件的二进制数据类型，也可以将这个文件读取为类型化数组。

```
reader.readAsArrayBuffer(file);
```

下面以处理 bmp 文件为例。假定 file 变量是一个指向 bmp 文件的文件对象，首先读取文件。

```
var reader = new FileReader();
reader.addEventListener("load", processimage, false); 
reader.readAsArrayBuffer(file);
```

然后，定义处理图像的回调函数：先在二进制数据之上建立一个 DataView 视图，再建立一个 bitmap 对象，用于存放处理后的数据，最后将图像展示在 canvas 元素之中。

```
function processimage(e) { 
 var buffer = e.target.result; 
 var datav = new DataView(buffer); 
 var bitmap = {};
 // 具体的处理步骤
}
```

具体处理图像数据时，先处理 bmp 的文件头。具体每个文件头的格式和定义，请参阅有关资料。

```
bitmap.fileheader = {}; 
bitmap.fileheader.bfType = datav.getUint16(0, true); 
bitmap.fileheader.bfSize = datav.getUint32(2, true); 
bitmap.fileheader.bfReserved1 = datav.getUint16(6, true); 
bitmap.fileheader.bfReserved2 = datav.getUint16(8, true); 
bitmap.fileheader.bfOffBits = datav.getUint32(10, true);
```

接着处理图像元信息部分。

```
bitmap.infoheader = {};
bitmap.infoheader.biSize = datav.getUint32(14, true);
bitmap.infoheader.biWidth = datav.getUint32(18, true); 
bitmap.infoheader.biHeight = datav.getUint32(22, true); 
bitmap.infoheader.biPlanes = datav.getUint16(26, true); 
bitmap.infoheader.biBitCount = datav.getUint16(28, true); 
bitmap.infoheader.biCompression = datav.getUint32(30, true); 
bitmap.infoheader.biSizeImage = datav.getUint32(34, true); 
bitmap.infoheader.biXPelsPerMeter = datav.getUint32(38, true); 
bitmap.infoheader.biYPelsPerMeter = datav.getUint32(42, true); 
bitmap.infoheader.biClrUsed = datav.getUint32(46, true); 
bitmap.infoheader.biClrImportant = datav.getUint32(50, true);
```

最后处理图像本身的像素信息。

```
var start = bitmap.fileheader.bfOffBits;
bitmap.pixels = new Uint8Array(buffer, start);
```

至此，图像文件的数据全部处理完成。下一步，可以根据需要，进行图像变形，或者转换格式，或者展示在 Canvas 网页元素之中。

## 参考链接

*   Ilmari Heikkinen, [Typed Arrays: Binary Data in the Browser](http://www.html5rocks.com/en/tutorials/webgl/typed_arrays/)
*   Khronos, [Typed Array Specification](http://www.khronos.org/registry/typedarray/specs/latest/)
*   Ian Elliot, [Reading A BMP File In JavaScript](http://www.i-programmer.info/projects/36-web/6234-reading-a-bmp-file-in-javascript.html)

*   Renato Mangini, [How to convert ArrayBuffer to and from String](http://updates.html5rocks.com/2012/06/How-to-convert-ArrayBuffer-to-and-from-String)