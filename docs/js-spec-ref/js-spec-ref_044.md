# 6.5 Ajax

Ajax 指的是不刷新页面，发出异步请求，向服务器端要求数据，然后再进行处理的方法。

*   XMLHttpRequest 对象
    *   Open()
    *   setRequestHeader()
    *   send()
    *   readyState 属性和 readyStateChange 事件
    *   progress 事件
    *   服务器返回的信息
    *   setRequestHeader 方法
    *   overrideMimeType 方法
    *   responseType 属性
    *   文件上传
    *   JSONP
    *   CORS
    *   Fetch API
        *   基本用法
        *   fetch()
        *   Headers
        *   Request 对象
        *   Response
        *   body 属性
    *   参考链接

## XMLHttpRequest 对象

XMLHttpRequest 对象用于从 JavaScript 发出 HTTP 请求，下面是典型用法。

```js
// 新建一个 XMLHttpRequest 实例对象
var xhr = new XMLHttpRequest();

// 指定通信过程中状态改变时的回调函数
xhr.onreadystatechange = function(){

    // 通信成功时，状态值为 4
    var completed = 4;
    if(xhr.readyState === completed){
        if(xhr.status === 200){
            // 处理服务器发送过来的数据
        }else{
            // 处理错误
        }
    }
};

// open 方式用于指定 HTTP 动词、请求的网址、是否异步
xhr.open('GET', '/endpoint', true);

// 发送 HTTP 请求
xhr.send(null);
```

### Open()

open 方法用于指定发送 HTTP 请求的参数，它有三个参数如下：

*   发送方法，一般来说为“GET”、“POST”、“PUT”和“DELETE”中的一个值。
*   网址。
*   是否异步，true 表示异步，false 表示同步。

下面发送 POST 请求的例子。

```js
xhr.open('POST', encodeURI('someURL'));
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.onload = function() {};
xhr.send(encodeURI('dataString'));
```

上面方法中，open 方法向指定 URL 发出 POST 请求，send 方法送出实际的数据。

### setRequestHeader()

setRequestHeader 方法用于设置 HTTP 请求的头信息。

### send()

send 方法用于实际发出 HTTP 请求。如果不带参数，就表示 HTTP 请求只包含头信息，也就是只有一个 URL，典型例子就是 GET 请求；如果带有参数，就表示除了头信息，还带有包含具体数据的信息体，典型例子就是 POST 请求。

在 XHR 2 之中，send 方法可以发送许多类型的数据。

```js
void send();
void send(ArrayBuffer data);
void send(Blob data);
void send(Document data);
void send(DOMString data);
void send(FormData data);
```

Blob 类型可以用来发送二进制数据，这使得通过 Ajax 上传文件成为可能。

FormData 类型可以用于构造表单数据。

```js
var formData = new FormData();

formData.append('username', '张三');
formData.append('email', 'zhangsan@example.com');
formData.append('birthDate', 1940);

xhr.send(formData);
```

上面的代码构造了一个 formData 对象，然后使用 send 方法发送。它的效果与点击下面表单的 submit 按钮是一样的。

```js
<form id='registration' name='registration' action='/register'>
    <input type='text' name='username' value='张三'>
    <input type='email' name='email' value='zhangsan@example.com'>
    <input type='number' name='birthDate' value='1940'>
    <input type='submit' onclick='return sendForm(this.form);'>
</form>
```

FormData 对象还可以对现有表单添加数据，这为我们操作表单提供了极大的灵活性。

```js
function sendForm(form) {
    var formData = new FormData(form);
    formData.append('csrf', 'e69a18d7db1286040586e6da1950128c');

    var xhr = new XMLHttpRequest();
    xhr.open('POST', form.action, true);
    xhr.onload = function(e) {
        // ...
    };
    xhr.send(formData);

    return false; 
}

var form = document.querySelector('#registration');
sendForm(form);
```

FormData 对象也能用来模拟 File 控件，进行文件上传。

```js
function uploadFiles(url, files) {
  var formData = new FormData();

  for (var i = 0, file; file = files[i]; ++i) {
    formData.append(file.name, file);
  }

  var xhr = new XMLHttpRequest();
  xhr.open('POST', url, true);
  xhr.onload = function(e) { ... };

  xhr.send(formData);  // multipart/form-data
}

document.querySelector('input[type="file"]').addEventListener('change', function(e) {
  uploadFiles('/server', this.files);
}, false);
```

### readyState 属性和 readyStateChange 事件

在通信过程中，每当发生状态变化的时候，readyState 属性的值就会发生改变。

这个值每一次变化，都会触发 readyStateChange 事件。我们可以指定这个事件的回调函数，对不同状态进行不同处理。尤其是当状态变为 4 的时候，表示通信成功，这时回调函数就可以处理服务器传送回来的数据。

### progress 事件

上传文件时，XMLHTTPRequest 对象的 upload 属性有一个 progress，会不断返回上传的进度。

假定网页上有一个 progress 元素。

```js
<progress min="0" max="100" value="0">0% complete</progress>
```

文件上传时，对 upload 属性指定 progress 事件回调函数，即可获得上传的进度。

```js
function upload(blobOrFile) {
  var xhr = new XMLHttpRequest();
  xhr.open('POST', '/server', true);
  xhr.onload = function(e) { ... };

  // Listen to the upload progress.
  var progressBar = document.querySelector('progress');
  xhr.upload.onprogress = function(e) {
    if (e.lengthComputable) {
      progressBar.value = (e.loaded / e.total) * 100;
      progressBar.textContent = progressBar.value; // Fallback for unsupported browsers.
    }
  };

  xhr.send(blobOrFile);
}

upload(new Blob(['hello world'], {type: 'text/plain'}));
```

下面是一个上传 ArrayBuffer 对象的例子。

```js
function sendArrayBuffer() {
  var xhr = new XMLHttpRequest();
  xhr.open('POST', '/server', true);
  xhr.onload = function(e) { ... };

  var uInt8Array = new Uint8Array([1, 2, 3]);

  xhr.send(uInt8Array.buffer);
}
```

### 服务器返回的信息

（1）status 属性

status 属性表示返回的 HTTP 状态码。一般来说，如果通信成功的话，这个状态码是 200。

（2）responseText 属性

responseText 属性表示服务器返回的文本数据。

### setRequestHeader 方法

setRequestHeader 方法用于设置 HTTP 头信息。

```js
xhr.setRequestHeader('Content-Type', 'application/json');

xhr.setRequestHeader('Content-Length', JSON.stringify(data).length);

xhr.send(JSON.stringify(data));
```

上面代码首先设置头信息 Content-Type，表示发送 JSON 格式的数据；然后设置 Content-Length，表示数据长度；最后发送 JSON 数据。

### overrideMimeType 方法

该方法用来指定服务器返回数据的 MIME 类型。

传统上，如果希望从服务器取回二进制数据，就要使用这个方法，人为将数据类型伪装成文本数据。

```js
var xhr = new XMLHttpRequest();
xhr.open('GET', '/path/to/image.png', true);

// 强制将 MIME 改为文本类型
xhr.overrideMimeType('text/plain; charset=x-user-defined');

xhr.onreadystatechange = function(e) {
  if (this.readyState == 4 && this.status == 200) {
    var binStr = this.responseText;
    for (var i = 0, len = binStr.length; i < len; ++i) {
      var c = binStr.charCodeAt(i);
      var byte = c & 0xff;  // 去除高位字节，留下低位字节
    }
  }
};

xhr.send();
```

上面代码中，因为传回来的是二进制数据，首先用 xhr.overrideMimeType 方法强制改变它的 MIME 类型，伪装成文本数据。字符集必需指定为“x-user-defined”，如果是其他字符集，浏览器内部会强制转码，将其保存成 UTF-16 的形式。字符集“x-user-defined”其实也会发生转码，浏览器会在每个字节前面再加上一个字节（0xF700-0xF7ff），因此后面要对每个字符进行一次与运算（&），将高位的 8 个位去除，只留下低位的 8 个位，由此逐一读出原文件二进制数据的每个字节。

这种方法很麻烦，在 XMLHttpRequest 版本升级以后，一般采用下面的指定 responseType 的方法。

### responseType 属性

XMLHttpRequest 对象有一个 responseType 属性，用来指定服务器返回数据（xhr.response）的类型。

XHR 2 允许用户自行设置这个属性，也就是指定返回数据的类型，可以设置如下的值：

*   'text'：返回类型为字符串，这是默认值。
*   'arraybuffer'：返回类型为 ArrayBuffer。
*   'blob'：返回类型为 Blob。
*   'document'：返回类型为 Document。
*   'json'：返回类型为 JSON object。

text 类型适合大多数情况，而且直接处理文本也比较方便，document 类型适合返回 XML 文档的情况，blob 类型适合读取二进制数据，比如图片文件。

```js
var xhr = new XMLHttpRequest();
xhr.open('GET', '/path/to/image.png', true);
xhr.responseType = 'blob';

xhr.onload = function(e) {
  if (this.status == 200) {
    var blob = new Blob([this.response], {type: 'image/png'});
    // ...
  }
};

xhr.send();
```

如果将这个属性设为 ArrayBuffer，就可以按照数组的方式处理二进制数据。

```js
var xhr = new XMLHttpRequest();
xhr.open('GET', '/path/to/image.png', true);
xhr.responseType = 'arraybuffer';

xhr.onload = function(e) {
  var uInt8Array = new Uint8Array(this.response);
  for (var i = 0, len = binStr.length; i < len; ++i) {
    // var byte = uInt8Array[i]; 
  }
};

xhr.send();
```

如果将这个属性设为“json”，支持 JSON 的浏览器（Firefox>9，chrome>30），就会自动对返回数据调用 JSON.parse() 方法。也就是说，你从 xhr.response 属性（注意，不是 xhr.responseText 属性）得到的不是文本，而是一个 JSON 对象。

XHR2 支持 Ajax 的返回类型为文档，即 xhr.responseType="document" 。这意味着，对于那些打开 CORS 的网站，我们可以直接用 Ajax 抓取网页，然后不用解析 HTML 字符串，直接对 XHR 回应进行 DOM 操作。

## 文件上传

通常，我们使用 file 控件实现文件上传。

```js
<form id="file-form" action="handler.php" method="POST">
  <input type="file" id="file-select" name="photos[]" multiple/>
  <button type="submit" id="upload-button">上传</button>
</form>
```

上面 HTML 代码中，file 控件的 multiple 属性，指定可以一次选择多个文件；如果没有这个属性，则一次只能选择一个文件。

file 对象的 files 属性，返回一个 FileList 对象，包含了用户选中的文件。

```js
var fileSelect = document.getElementById('file-select');
var files = fileSelect.files;
```

然后，新建一个 FormData 对象的实例，用来模拟发送到服务器的表单数据，把选中的文件添加到这个对象上面。

```js
var formData = new FormData();

for (var i = 0; i < files.length; i++) {
  var file = files[i];

  if (!file.type.match('image.*')) {
    continue;
  }

  formData.append('photos[]', file, file.name);
}
```

上面代码中的 FormData 对象的 append 方法，除了可以添加文件，还可以添加二进制对象（Blob）或者字符串。

```js
// Files
formData.append(name, file, filename);

// Blobs
formData.append(name, blob, filename);

// Strings
formData.append(name, value);
```

append 方法的第一个参数是表单的控件名，第二个参数是实际的值，第三个参数是可选的，通常是文件名。

最后，使用 Ajax 方法向服务器上传文件。

```js
var xhr = new XMLHttpRequest();

xhr.open('POST', 'handler.php', true);

xhr.onload = function () {
  if (xhr.status !== 200) {
    alert('An error occurred!');
  }
};

xhr.send(formData);
```

目前，各大浏览器（包括 IE 10）都支持 Ajax 上传文件。

除了使用 FormData 接口上传，也可以直接使用 File API 上传。

```js
var file = document.getElementById('test-input').files[0];
var xhr = new XMLHttpRequest();

xhr.open('POST', 'myserver/uploads');
xhr.setRequestHeader('Content-Type', file.type);
xhr.send(file);
```

可以看到，上面这种写法比 FormData 的写法，要简单很多。

## JSONP

JSONP 是一种常见做法，用于服务器与客户端之间的数据传输，主要为了规避浏览器的同域限制。因为 Ajax 只能向当前网页所在的域名发出 HTTP 请求（除非使用下文要提到的 CORS，但并不是所有服务器都支持 CORS），所以 JSONP 就采用在网页中动态插入 script 元素的做法，向服务器请求脚本文件。

```js
function addScriptTag(src){
    var script = document.createElement('script');
    script.setAttribute("type","text/javascript");
    script.src = src;
    document.body.appendChild(script);
}

window.onload = function(){
    addScriptTag("http://example.com/ip?callback=foo");
}

function foo(data) {
    console.log('Your public IP address is: ' + data.ip);
};
```

上面代码使用了 JSONP，运行以后当前网页就可以直接处理 example.com 返回的数据了。

由于 script 元素返回的脚本文件，是直接作为代码运行的，不像 Ajax 请求返回的是 JSON 字符串，需要用 JSON.parse 方法将字符串转为 JSON 对象。于是，为了方便起见，许多服务器支持 JSONP 指定回调函数的名称，直接将 JSON 数据放入回调函数的参数，如此一来就省略了将字符串解析为 JSON 对象的步骤。

请看下面的例子，假定访问 [`example.com/ip`](http://example.com/ip) ，返回如下 JSON 数据：

```js
{"ip":"8.8.8.8"}
```

现在服务器允许客户端请求时使用 callback 参数指定回调函数。访问 [`example.com/ip?callback=foo`](http://example.com/ip?callback=foo) ，返回的数据变成：

```js
foo({"ip":"8.8.8.8"})
```

这时，如果客户端定义了 foo 函数，该函数就会被立即调用，而作为参数的 JSON 数据被视为 JavaScript 对象，而不是字符串，因此避免了使用 JSON.parse 的步骤。

```js
function foo(data) {
    console.log('Your public IP address is: ' + data.ip);
};
```

jQuery 的 getJSON 方法就是 JSONP 的一个应用。

```js
$.getJSON( "http://example.com/api", function (data){ .... })
```

$.getJSON 方法的第一个参数是服务器网址，第二个参数是回调函数，该回调函数的参数就是服务器返回的 JSON 数据。

## CORS

CORS 的全称是“跨域资源共享”（Cross-origin resource sharing），它提出一种方法，允许 JavaScript 代码向另一个域名发出 XMLHttpRequests 请求，从而克服了传统上 Ajax 只能在同一个域名下使用的限制（same origin security policy）。

所有主流浏览器都支持该方法，不过 IE8 和 IE9 的该方法不是部署在 XMLHttpRequest 对象，而是部署在 XDomainRequest 对象。检查浏览器是否支持的代码如下：

```js
var request = new XMLHttpRequest();

if("withCredentials" in request) {
  // 发出跨域请求
}
```

CORS 的原理其实很简单，就是增加一条 HTTP 头信息的查询，询问服务器端，当前请求的域名是否在许可名单之中，以及可以使用哪些 HTTP 动词。如果得到肯定的答复，就发出 XMLHttpRequest 请求。这种机制叫做“预检”（preflight）。

“预检”的专用 HTTP 头信息是 Origin。假定用户正在浏览来自 www.example.com 的网页，该网页需要向 Google 请求数据，这时浏览器会向该域名询问是否同意跨域请求，发出的 HTTP 头信息如下：

```js
OPTIONS /resources/post-here/ HTTP/1.1
Host: www.google.com
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.5; en-US; rv:1.9.1b3pre) Gecko/20081130 Minefield/3.1b3pre
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
Origin: http://www.example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER
```

上面的 HTTP 请求，它的动词是 OPTIONS，表示这是一个“预检”请求。除了提供浏览器信息，里面关键的一行是 Origin 头信息。

```js
Origin: http://www.example.com
```

这行 HTTP 头信息表示，请求来自 www.example.com。服务端如果同意，就返回一个 Access-Control-Allow-Origin 头信息。

预检请求中，浏览器还告诉服务器，实际发出请求，将使用 HTTP 动词 POST，以及一个自定义的头信息 X-PINGOTHER。

```js
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER
```

服务器收到预检请求之后，做出了回应。

```js
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://www.example.com
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER
Access-Control-Max-Age: 1728000
Vary: Accept-Encoding, Origin
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
```

上面的 HTTP 回应里面，关键的是 Access-Control-Allow-Origin 头信息。这表示服务器同意 www.example.com 的跨域请求。

```js
Access-Control-Allow-Origin: http://www.example.com
```

如果不同意，服务器端会返回一个错误。

如果服务器端对所有网站都开放，可以返回一个星号（*）通配符。

```js
Access-Control-Allow-Origin: *
```

服务器还告诉浏览器，允许的 HTTP 动词是 POST、GET、OPTIONS，也允许自定义的头信息 X-PINGOTHER，

```js
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER
Access-Control-Max-Age: 1728000
```

如果服务器通过了预检请求，则以后每次浏览器正常的 HTTP 请求，都会有一个 origin 头信息；服务器的回应，也都会有一个 Access-Control-Allow-Origin 头信息。Access-Control-Max-Age 头信息表示，允许缓存该条回应 1728000 秒（即 20 天），在此期间，不用发出另一条预检请求。

由于整个过程都是浏览器自动后台完成，不用用户参与，所以对于开发者来说，使用 Ajax 跨域请求与同域请求没有区别，代码完全一样。但是，这需要服务器的支持，所以在使用 CORS 之前，要查看一下所请求的网站是否支持。

CORS 机制默认不发送 cookie 和 HTTP 认证信息，除非在 Ajax 请求中打开 withCredentials 属性。

```js
var request = new XMLHttpRequest();
request.withCredentials = true;
```

同时，服务器返回 HTTP 头信息时，也必须打开 Access-Control-Allow-Credentials 选项。否则，浏览器会忽略服务器返回的回应。

```js
Access-Control-Allow-Credentials: true
```

需要注意的是，此时 Access-Control-Allow-Origin 不能指定为星号，必须指定明确的、与请求网页一致的域名。同时，cookie 依然遵循同源政策，只有用服务器域名（前例是 www.google.com）设置的 cookie 才会上传，其他域名下的 cookie 并不会上传，且网页代码中的 document.cookie 也无法读取 www.google.com 域名下的 cookie。

CORS 机制与 JSONP 模式的使用目的相同，而且更强大。JSONP 只支持 GET 请求，CORS 可以支持所有类型的 HTTP 请求。在发生错误的情况下，CORS 可以得到更详细的错误信息，部署更有针对性的错误处理代码。JSONP 的优势在于可以用于老式浏览器，以及可以向不支持 CORS 的网站请求数据。

## Fetch API

### 基本用法

Ajax 操作所用的 XMLHttpRequest 对象，已经有十多年的历史，它的 API 设计并不是很好，输入、输出、状态都在同一个接口管理，容易写出非常混乱的代码。Fetch API 是一种新规范，用来取代 XMLHttpRequest 对象。它主要有两个特点，一是简化接口，将 API 分散在几个不同的对象上，二是返回 Promise 对象，避免了嵌套的回调函数。

检查浏览器是否部署了这个 API 的代码如下。

```js
if (fetch in window){
  // 支持
} else {
  // 不支持
}
```

下面是一个 Fetch API 的简单例子。

```js
var URL = 'http://some/path';

fetch(URL).then(function(response) {
  return response.json();
}).then(function(json) {
  someOperator(json);
});
```

上面代码向服务器请求 JSON 文件，获取后再做进一步处理。

下面比较 XMLHttpRequest 写法与 Fetch 写法的不同。

```js
function reqListener() {
  var data = JSON.parse(this.responseText);
  console.log(data);
}

function reqError(err) {
  console.log('Fetch Error :-S', err);
}

var oReq = new XMLHttpRequest();
oReq.onload = reqListener;
oReq.onerror = reqError;
oReq.open('get', './api/some.json', true);
oReq.send();
```

同样的操作用 Fetch 实现如下。

```js
fetch('./api/some.json')
  .then(function(response) {
    if (response.status !== 200) {
      console.log('请求失败，状态码：' + response.status);
      return;
    }
    response.json().then(function(data) {
      console.log(data);
    });
  }).catch(function(err) {
    console.log('出错：', err);
  });
```

上面代码中，因为 HTTP 请求返回的 response 对象是一个 Stream 对象，所以需要使用`response.json`方法转为 JSON 格式，不过这个方法返回的是一个 Promise 对象。

### fetch()

fetch 方法的第一个参数可以是 URL 字符串，也可以是后文要讲到的 Request 对象实例。Fetch 方法返回一个 Promise 对象，并将一个 response 对象传给回调函数。

response 对象还有一个 ok 属性，如果返回的状态码在 200 到 299 之间（即请求成功），这个属性为 true，否则为 false。因此，上面的代码可以写成下面这样。

```js
fetch("./api/some.json").then(function(response) {
  if (response.ok) {
    response.json().then(function(data) {
      console.log(data);
    });
  } else {
    console.log("请求失败，状态码为", response.status);
  }
}, function(err) {
  console.log("出错：", err);
});
```

response 对象除了 json 方法，还包含了 HTTP 回应的元数据。

```js
fetch('users.json').then(function(response) {
  console.log(response.headers.get('Content-Type'));
  console.log(response.headers.get('Date'));
  console.log(response.status);
  console.log(response.statusText);
  console.log(response.type);
  console.log(response.url);
});
```

上面代码中，response 对象有很多属性，其中的`response.type`属性比较特别，表示 HTTP 回应的类型，它有以下三个值。

*   basic：正常的同域请求
*   cors：CORS 机制下的跨域请求
*   opaque：非 CORS 机制下的跨域请求，这时无法读取返回的数据，也无法判断是否请求成功

如果需要在 CORS 机制下发出跨域请求，需要指明状态。

```js
fetch('http://some-site.com/cors-enabled/some.json', {mode: 'cors'})
  .then(function(response) {
    return response.text();
  })
  .then(function(text) {
    console.log('Request successful', text);
  })
  .catch(function(error) {
    log('Request failed', error)
  });
```

除了指定模式，fetch 方法的第二个参数还可以用来配置其他值，比如指定 cookie 连同 HTTP 请求一起发出。

```js
fetch(url, {
  credentials: 'include'
})
```

发出 POST 请求的写法如下。

```js
fetch("http://www.example.org/submit.php", {
  method: "POST",
  headers: {
    "Content-Type": "application/x-www-form-urlencoded"
  },
  body: "firstName=Nikhil&favColor=blue&password=easytoguess"
}).then(function(res) {
  if (res.ok) {
    console.log("Perfect! Your settings are saved.");
  } else if (res.status == 401) {
    console.log("Oops! You are not authorized.");
  }
}, function(e) {
  console.log("Error submitting form!");
});
```

目前，还有一些 XMLHttpRequest 对象可以做到，但是 Fetch API 还没做到的地方，比如中途中断 HTTP 请求，以及获取 HTTP 请求的进度。这些不足与 Fetch 返回的是 Promise 对象有关。

### Headers

Fetch API 引入三个新的对象（也是构造函数）：Headers, Request 和 Response。其中，Headers 对象用来构造/读取 HTTP 数据包的头信息。

```js
var content = "Hello World";
var reqHeaders = new Headers();
reqHeaders.append("Content-Type", "text/plain");
reqHeaders.append("Content-Length", content.length.toString());
reqHeaders.append("X-Custom-Header", "ProcessThisImmediately");
```

Headers 对象的实例，除了使用 append 方法添加属性，也可以直接通过构造函数一次性生成。

```js
reqHeaders = new Headers({
  "Content-Type": "text/plain",
  "Content-Length": content.length.toString(),
  "X-Custom-Header": "ProcessThisImmediately",
});
```

Headers 对象实例还提供了一些工具方法。

```js
reqHeaders.has("Content-Type") // true
reqHeaders.has("Set-Cookie") // false
reqHeaders.set("Content-Type", "text/html")
reqHeaders.append("X-Custom-Header", "AnotherValue")

reqHeaders.get("Content-Length") // 11
reqHeaders.getAll("X-Custom-Header") // ["ProcessThisImmediately", "AnotherValue"]

reqHeaders.delete("X-Custom-Header")
reqHeaders.getAll("X-Custom-Header") // []
```

生成 Header 实例以后，可以将它作为第二个参数，传入 Request 方法。

```js
var headers = new Headers();
headers.append('Accept', 'application/json');
var request = new Request(URL, {headers: headers});

fetch(request).then(function(response) {
  console.log(response.headers);
});
```

同样地，Headers 实例可以用来构造 Response 方法。

```js
var headers = new Headers({
  'Content-Type': 'application/json',
  'Cache-Control': 'max-age=3600'
});

var response = new Response(
  JSON.stringify({photos: {photo: []}}),
  {'status': 200, headers: headers}
);

response.json().then(function(json) {
  insertPhotos(json);
});
```

上面代码中，构造了一个 HTTP 回应。目前，浏览器构造 HTTP 回应没有太大用处，但是随着 Service Worker 的部署，不久浏览器就可以向 Service Worker 发出 HTTP 回应。

### Request 对象

Request 对象用来构造 HTTP 请求。

```js
var req = new Request("/index.html");
req.method // "GET"
req.url // "http://example.com/index.html"
```

Request 对象的第二个参数，表示配置对象。

```js
var uploadReq = new Request("/uploadImage", {
  method: "POST",
  headers: {
    "Content-Type": "image/png",
  },
  body: "image data"
});
```

上面代码指定 Request 对象使用 POST 方法发出，并指定 HTTP 头信息和信息体。

下面是另一个例子。

```js
var req = new Request(URL, {method: 'GET', cache: 'reload'});
fetch(req).then(function(response) {
  return response.json();
}).then(function(json) {
  someOperator(json);
});
```

上面代码中，指定请求方法为 GET，并且要求浏览器不得缓存 response。

Request 对象实例有两个属性是只读的，不能手动设置。一个是 referrer 属性，表示请求的来源，由浏览器设置，有可能是空字符串。另一个是 context 属性，表示请求发出的上下文，如果是 image，表示是从 img 标签发出，如果是 worker，表示是从 worker 脚本发出，如果是 fetch，表示是从 fetch 函数发出的。

Request 对象实例的 mode 属性，用来设置是否跨域，合法的值有以下三种：same-origin、no-cors（默认值）、cors。当设置为 same-origin 时，只能向同域的 URL 发出请求，否则会报错。

```js
var arbitraryUrl = document.getElementById("url-input").value;
fetch(arbitraryUrl, { mode: "same-origin" }).then(function(res) {
  console.log("Response succeeded?", res.ok);
}, function(e) {
  console.log("Please enter a same-origin URL!");
});
```

上面代码中，如果用户输入的 URL 不是同域的，将会报错，否则就会发出请求。

如果 mode 属性为 no-cors，就与默认的浏览器行为没有不同，类似 script 标签加载外部脚本文件、img 标签加载外部图片。如果 mode 属性为 cors，就可以向部署了 CORS 机制的服务器，发出跨域请求。

```js
var u = new URLSearchParams();
u.append('method', 'flickr.interestingness.getList');
u.append('api_key', '<insert api key here>');
u.append('format', 'json');
u.append('nojsoncallback', '1');

var apiCall = fetch('https://api.flickr.com/services/rest?' + u);

apiCall.then(function(response) {
  return response.json().then(function(json) {
    // photo is a list of photos.
    return json.photos.photo;
  });
}).then(function(photos) {
  photos.forEach(function(photo) {
    console.log(photo.title);
  });
});
```

上面代码是向 Flickr API 发出图片请求的例子。

Request 对象的一个很有用的功能，是在其他 Request 实例的基础上，生成新的 Request 实例。

```js
var postReq = new Request(req, {method: 'POST'});
```

### Response

fetch 方法返回 Response 对象实例，它有以下属性。

*   status：整数值，表示状态码（比如 200）
*   statusText：字符串，表示状态信息，默认是“OK”
*   ok：布尔值，表示状态码是否在 200-299 的范围内
*   headers：Headers 对象，表示 HTTP 回应的头信息
*   url：字符串，表示 HTTP 请求的网址
*   type：字符串，合法的值有五个 basic、cors、default、error、opaque。basic 表示正常的同域请求；cors 表示 CORS 机制的跨域请求；error 表示网络出错，无法取得信息，status 属性为 0，headers 属性为空，并且导致 fetch 函数返回 Promise 对象被拒绝；opaque 表示非 CORS 机制的跨域请求，受到严格限制。

Response 对象还有两个静态方法。

*   Response.error() 返回一个 type 属性为 error 的 Response 对象实例
*   Response.redirect(url, status) 返回的 Response 对象实例会重定向到另一个 URL

### body 属性

Request 对象和 Response 对象都有 body 属性，表示请求的内容。body 属性可能是以下的数据类型。

*   ArrayBuffer
*   ArrayBufferView (Uint8Array 等)
*   Blob/File
*   string
*   URLSearchParams
*   FormData

```js
var form = new FormData(document.getElementById('login-form'));
fetch("/login", {
  method: "POST",
  body: form
})
```

上面代码中，Request 对象的 body 属性为表单数据。

Request 对象和 Response 对象都提供以下方法，用来读取 body。

*   arrayBuffer()
*   blob()
*   json()
*   text()
*   formData()

注意，上面这些方法都只能使用一次，第二次使用就会报错，也就是说，body 属性只能读取一次。Request 对象和 Response 对象都有 bodyUsed 属性，返回一个布尔值，表示 body 是否被读取过。

```js
var res = new Response("one time use");
console.log(res.bodyUsed); // false
res.text().then(function(v) {
  console.log(res.bodyUsed); // true
});
console.log(res.bodyUsed); // true

res.text().catch(function(e) {
  console.log("Tried to read already consumed Response");
});
```

上面代码中，第二次通过 text 方法读取 Response 对象实例的 body 时，就会报错。

这是因为 body 属性是一个 stream 对象，数据只能单向传送一次。这样的设计是为了允许 JavaScript 处理视频、音频这样的大型文件。

如果希望多次使用 body 属性，可以使用 Response 对象和 Request 对象的 clone 方法。它必须在 body 还没有读取前调用，返回一个前的 body，也就是说，需要使用几次 body，就要调用几次 clone 方法。

```js
addEventListener('fetch', function(evt) {
  var sheep = new Response("Dolly");
  console.log(sheep.bodyUsed); // false
  var clone = sheep.clone();
  console.log(clone.bodyUsed); // false

  clone.text();
  console.log(sheep.bodyUsed); // false
  console.log(clone.bodyUsed); // true

  evt.respondWith(cache.add(sheep.clone()).then(function(e) {
    return sheep;
  });
});
```

## 参考链接

*   MDN, [Using XMLHttpRequest](https://developer.mozilla.org/en-US/docs/DOM/XMLHttpRequest/Using_XMLHttpRequest)
*   Mathias Bynens, [Loading JSON-formatted data with Ajax and xhr.responseType='json'](http://mathiasbynens.be/notes/xhr-responsetype-json)
*   Eric Bidelman, [New Tricks in XMLHttpRequest2](http://www.html5rocks.com/en/tutorials/file/xhr2/)
*   Matt West, [Uploading Files with AJAX](http://blog.teamtreehouse.com/uploading-files-ajax)
*   Monsur Hossain, [Using CORS](http://www.html5rocks.com/en/tutorials/cors/)
*   MDN, [HTTP access control (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)
*   Matt Gaunt, [Introduction to fetch()](http://updates.html5rocks.com/2015/03/introduction-to-fetch)
*   Nikhil Marathe, [This API is so Fetching!](https://hacks.mozilla.org/2015/03/this-api-is-so-fetching/)
*   Ludovico Fischer, [Introduction to the Fetch API](http://www.sitepoint.com/introduction-to-the-fetch-api/)