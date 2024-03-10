# 6.11 移动设备 API

为了更好地为移动设备服务，HTML 5 推出了一系列针对移动设备的 API。

*   Viewport
*   Geolocation API
    *   getCurrentPosition 方法
    *   watchPosition 方法和 clearWatch 方法
    *   Vibration API
    *   Luminosity API
    *   Orientation API
    *   参考链接

## Viewport

Viewport 指的是网页的显示区域，也就是不借助滚动条的情况下，用户可以看到的部分网页大小，中文译为“视口”。正常情况下，viewport 和浏览器的显示窗口是一样大小的。但是，在移动设备上，两者可能不是一样大小。

比如，手机浏览器的窗口宽度可能是 640 像素，这时 viewport 宽度就是 640 像素，但是网页宽度有 950 像素，正常情况下，浏览器会提供横向滚动条，让用户查看窗口容纳不下的 310 个像素。另一种方法则是，将 viewport 设成 950 像素，也就是说，浏览器的显示宽度还是 640 像素，但是网页的显示区域达到 950 像素，整个网页缩小了，在浏览器中可以看清楚全貌。这样一来，手机浏览器就可以看到网页在桌面浏览器上的显示效果。

viewport 缩放规则，需要在 HTML 网页的 head 部分指定。

```js
<head>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no"/>
</head>
```

上面代码指定，viewport 的缩放规则是，缩放到当前设备的屏幕宽度（device-width），初始缩放比例（initial-scale）为 1 倍，禁止用户缩放（user-scalable）。

viewport 全部属性如下。

*   width: viewport 宽度
*   height: viewport 高度
*   initial-scale: 初始缩放比例
*   maximum-scale: 最大缩放比例
*   minimum-scale: 最小缩放比例
*   user-scalable: 是否允许用户缩放

其他的例子如下。

```js
<meta name = "viewport" content = "width = 320,
       initial-scale = 2.3, user-scalable = no">
```

## Geolocation API

Geolocation 接口用于获取用户的地理位置。它使用的方法基于 GPS 或者其他机制（比如 IP 地址、Wifi 热点、手机基站等）。

下面的方法，可以检查浏览器是否支持这个接口。

```js
if(navigator.geolocation) { 
   // 支持
} else {
   // 不支持
}
```

这个 API 的支持情况非常好，所有浏览器都支持（包括 IE 9+），所以上面的代码不是很必要。

### getCurrentPosition 方法

getCurrentPosition 方法，用来获取用户的地理位置。使用它需要得到用户的授权，浏览器会跳出一个对话框，询问用户是否许可当前页面获取他的地理位置。必须考虑两种情况的回调函数：一种是同意授权，另一种是拒绝授权。如果用户拒绝授权，会抛出一个错误。

```js
navigator.geolocation.getCurrentPosition(geoSuccess,geoError);
```

上面代码指定了处理当前地理位置的两个回调函数。

（1）同意授权

如果用户同意授权，就会调用 geoSuccess。

```js
function geoSuccess(event) { 
   console.log(event.coords.latitude + ', ' + event.coords.longitude);
}
```

geoSuccess 的参数是一个 event 对象。event 有两个属性：timestamp 和 coords。timestamp 属性是一个时间戳，返回获得位置信息的具体时间。coords 属性指向一个对象，包含了用户的位置信息，主要是以下几个值：

*   coords.latitude：纬度
*   coords.longitude：经度
*   coords.accuracy：精度
*   coords.altitude：海拔
*   coords.altitudeAccuracy：海拔精度（单位：米）
*   coords.heading：以 360 度表示的方向
*   coords.speed：每秒的速度（单位：米）

大多数桌面浏览器不提供上面列表的后四个值。

（2）拒绝授权

如果用户拒绝授权，就会调用 getCurrentPosition 方法指定的第二个回调函数 geoError。

```js
function geoError(event) { 
   console.log("Error code " + event.code + ". " + event.message);
}
```

geoError 的参数也是一个 event 对象。event.code 属性表示错误类型，有四个值：

*   0：未知错误，浏览器没有提示出错的原因，相当于常量 event.UNKNOWN_ERROR。
*   1：用户拒绝授权，相当于常量 event.PERMISSION_DENIED。
*   2：没有得到位置，GPS 或其他定位机制无法定位，相当于常量 event.POSITION_UNAVAILABLE。
*   3：超时，GPS 没有在指定时间内返回结果，相当于常量 event.TIMEOUT。

(3)设置定位行为

getCurrentPosition 方法还可以接受一个对象作为第三个参数，用来设置定位行为。

```js
var option = {
            enableHighAccuracy : true,
            timeout : Infinity,
            maximumAge : 0
        };

navigator.geolocation.getCurrentPosition(geoSuccess, geoError, option);
```

这个参数对象有三个成员：

*   enableHighAccuracy：如果设为 true，就要求客户端提供更精确的位置信息，这会导致更长的定位时间和更大的耗电，默认设为 false。

*   Timeout：等待客户端做出回应的最大毫秒数，默认值为 Infinity（无限）。

*   maximumAge：客户端可以使用缓存数据的最大毫秒数。如果设为 0，客户端不读取缓存；如果设为 infinity，客户端只读取缓存。

### watchPosition 方法和 clearWatch 方法

watchPosition 方法可以用来监听用户位置的持续改变，使用方法与 getCurrentPosition 方法一样。

```js
var watchID = navigator.geolocation.watchPosition(geoSuccess,geoError, option);
```

一旦用户位置发生变化，就会调用回调函数 geoSuccess。这个回调函数的事件对象，也包含 timestamp 和 coords 属性。

watchPosition 和 getCurrentPosition 方法的不同之处在于，前者返回一个表示符，后者什么都不返回。watchPosition 方法返回的标识符，用于供 clearWatch 方法取消监听。

```js
navigator.geolocation.clearWatch(watchID);
```

## Vibration API

Vibration 接口用于在浏览器中发出命令，使得设备振动。显然，这个 API 主要针对手机，适用场合是向用户发出提示或警告，游戏中尤其会大量使用。由于振动操作很耗电，在低电量时最好取消该操作。

使用下面的代码检查该接口是否可用。目前，只有 Chrome 和 Firefox 的 Android 平台最新版本支持它。

```js
navigator.vibrate = navigator.vibrate 
                    || navigator.webkitVibrate 
                    || navigator.mozVibrate 
                    || navigator.msVibrate;

if (navigator.vibrate) {
    // 支持
}
```

vibrate 方法可以使得设备振动，它的参数就是振动持续的毫秒数。

```js
navigator.vibrate(1000);
```

上面的代码使得设备振动 1 秒钟。

vibrate 方法还可以接受一个数组作为参数，表示振动的模式。偶数位置的数组成员表示振动的毫秒数，奇数位置的数组成员表示等待的毫秒数。

```js
navigator.vibrate([500, 300, 100]);
```

上面代码表示，设备先振动 500 毫秒，然后等待 300 毫秒，再接着振动 500 毫秒。

vibrate 是一个非阻塞式的操作，即手机振动的同时，JavaScript 代码继续向下运行。要停止振动，只有将 0 毫秒或者一个空数组传入 vibrate 方法。

```js
navigator.vibrate(0);
navigator.vibrate([]);
```

如果要让振动一直持续，可以使用 setInterval 不断调用 vibrate。

```js
var vibrateInterval;

function startVibrate(duration) {
    navigator.vibrate(duration);
}

function stopVibrate() {
    if(vibrateInterval) clearInterval(vibrateInterval);
    navigator.vibrate(0);
}

function startPeristentVibrate(duration, interval) {
    vibrateInterval = setInterval(function() {
        startVibrate(duration);
    }, interval);
}
```

## Luminosity API

Luminosity API 用于屏幕亮度调节，当移动设备的亮度传感器感知外部亮度发生显著变化时，会触发 devicelight 事件。目前，只有 Firefox 部署了这个 API。

```js
window.addEventListener('devicelight', function(event) {
  console.log(event.value + 'lux');
});
```

上面代码表示，devicelight 事件的回调函数，接受一个事件对象作为参数。该对象的 value 属性就是亮度的流明值。

这个 API 的一种应用是，如果亮度变强，网页可以显示黑底白字，如果亮度变弱，网页可以显示白底黑字。

```js
window.addEventListener('devicelight', function(e) {
  var lux = e.value;

  if(lux < 50) {
    document.body.className = 'dim';
  }
  if(lux >= 50 && lux <= 1000) {
    document.body.className = 'normal';
  }
  if(lux > 1000)  {
    document.body.className = 'bright';
  } 
});
```

CSS 下一个版本的 Media Query 可以单独设置亮度，一旦浏览器支持，就可以用来取代 Luminosity API。

```js
@media (light-level: dim) {
  /* 暗光环境 */
}

@media (light-level: normal) {
  /* 正常光环境 */
}

@media (light-level: washed) {
  /* 明亮环境 */
}
```

## Orientation API

Orientation API 用于检测手机的摆放方向（竖放或横放）。

使用下面的代码检测浏览器是否支持该 API。

```js
if (window.DeviceOrientationEvent) {
  // 支持
} else {
  // 不支持
}
```

一旦设备的方向发生变化，会触发 deviceorientation 事件，可以对该事件指定回调函数。

```js
window.addEventListener("deviceorientation", callback);
```

回调函数接受一个 event 对象作为参数。

```js
function callback(event){
    console.log(event.alpha);
    console.log(event.beta);
    console.log(event.gamma);
}
```

上面代码中，event 事件对象有 alpha、beta 和 gamma 三个属性，它们分别对应手机摆放的三维倾角变化。要理解它们，就要理解手机的方向模型。当手机水平摆放时，使用三个轴标示它的空间位置：x 轴代表横轴、y 轴代表竖轴、z 轴代表垂直轴。event 对象的三个属性就对应这三根轴的旋转角度。

*   alpha：表示围绕 z 轴的旋转，从 0 到 360 度。当设备水平摆放时，顶部指向地球的北极，alpha 此时为 0。
*   beta：表示围绕 x 轴的旋转，从-180 度到 180 度。当设备水平摆放时，beta 此时为 0。
*   gramma：表示围绕 y 轴的选择，从-90 到 90 度。当设备水平摆放时，gramma 此时为 0。

## 参考链接

*   Ryan Stewart, [Using the Geolocation API](http://www.adobe.com/devnet/html5/articles/using-geolocation-api.html)
*   Rathnakanya K. Srinivasan, [HTML5 Geolocation](http://www.sitepoint.com/html5-geolocation/)
*   Craig Buckler, [How to Use the HTML5 Vibration API](http://www.sitepoint.com/use-html5-vibration-api/)
*   Tomomi Imura, [Responsive UI with Luminosity Level](http://girliemac.com/blog/2014/01/12/luminosity/)
*   Aurelio De Rosa, [An Introduction to the Geolocation API](http://code.tutsplus.com/tutorials/an-introduction-to-the-geolocation-api--cms-20071)
*   David Walsh, [Vibration API](http://davidwalsh.name/vibration-api)
*   Ahmet Mermerkaya, [Using Device Orientation in HTML5](http://www.sitepoint.com/using-device-orientation-html5/)