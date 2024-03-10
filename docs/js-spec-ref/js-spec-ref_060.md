# 7.9 Fullscreen API：全屏操作

全屏 API 可以控制浏览器的全屏显示，让一个 Element 节点（以及子节点）占满用户的整个屏幕。目前各大浏览器的最新版本都支持这个 API（包括 IE11），但是使用的时候需要加上浏览器前缀。

*   方法
    *   requestFullscreen()
    *   exitFullscreen()
    *   属性
        *   document.fullscreenElement
        *   document.fullscreenEnabled
    *   全屏事件
    *   全屏状态的 CSS
    *   参考链接

## 方法

### requestFullscreen()

Element 节点的 requestFullscreen 方法，可以使得这个节点全屏。

```js
function launchFullscreen(element) {
  if(element.requestFullscreen) {
    element.requestFullscreen();
  } else if(element.mozRequestFullScreen) {
    element.mozRequestFullScreen();
  } else if(element.msRequestFullscreen){
    element.msRequestFullscreen();
  } else if(element.webkitRequestFullscreen) {
    element.webkitRequestFullScreen();
  }
}

launchFullscreen(document.documentElement);
launchFullscreen(document.getElementById("videoElement"));
```

放大一个节点时，Firefox 和 Chrome 在行为上略有不同。Firefox 自动为该节点增加一条 CSS 规则，将该元素放大至全屏状态，`width: 100%; height: 100%`，而 Chrome 则是将该节点放在屏幕的中央，保持原来大小，其他部分变黑。为了让 Chrome 的行为与 Firefox 保持一致，可以自定义一条 CSS 规则。

```js
:-webkit-full-screen #myvideo {
  width: 100%;
  height: 100%;
}
```

### exitFullscreen()

document 对象的 exitFullscreen 方法用于取消全屏。该方法也带有浏览器前缀。

```js
function exitFullscreen() {
  if (document.exitFullscreen) {
    document.exitFullscreen();
  } else if (document.msExitFullscreen) {
    document.msExitFullscreen();
  } else if (document.mozCancelFullScreen) {
    document.mozCancelFullScreen();
  } else if (document.webkitExitFullscreen) {
    document.webkitExitFullscreen();
  }
}

exitFullscreen();
```

用户手动按下 ESC 键或 F11 键，也可以退出全屏键。此外，加载新的页面，或者切换 tab，或者从浏览器转向其他应用（按下 Alt-Tab），也会导致退出全屏状态。

## 属性

### document.fullscreenElement

fullscreenElement 属性返回正处于全屏状态的 Element 节点，如果当前没有节点处于全屏状态，则返回 null。

```js
var fullscreenElement =
  document.fullscreenElement ||
  document.mozFullScreenElement ||
  document.webkitFullscreenElement;
```

### document.fullscreenEnabled

fullscreenEnabled 属性返回一个布尔值，表示当前文档是否可以切换到全屏状态。

```js
var fullscreenEnabled =
  document.fullscreenEnabled ||
  document.mozFullScreenEnabled ||
  document.webkitFullscreenEnabled ||
  document.msFullscreenEnabled;

if (fullscreenEnabled) {
  videoElement.requestFullScreen();
} else {
  console.log('浏览器当前不能全屏');
}
```

## 全屏事件

以下事件与全屏操作有关。

*   fullscreenchange 事件：浏览器进入或离开全屏时触发。

*   fullscreenerror 事件：浏览器无法进入全屏时触发，可能是技术原因，也可能是用户拒绝。

```js
document.addEventListener("fullscreenchange", function( event ) {
  if (document.fullscreenElement) {
    console.log('进入全屏');
  } else {
    console.log('退出全屏');
  }
});
```

上面代码在发生 fullscreenchange 事件时，通过 fullscreenElement 属性判断，到底是进入全屏还是退出全屏。

## 全屏状态的 CSS

全屏状态下，大多数浏览器的 CSS 支持`:full-screen`伪类，只有 IE11 支持`:fullscreen`伪类。使用这个伪类，可以对全屏状态设置单独的 CSS 属性。

```js
:-webkit-full-screen {
  /* properties */
}

:-moz-full-screen {
  /* properties */
}

:-ms-fullscreen {
  /* properties */
}

:full-screen { /*pre-spec */
  /* properties */
}

:fullscreen { /* spec */
  /* properties */
}

/* deeper elements */
:-webkit-full-screen video {
  width: 100%;
  height: 100%;
}
```

## 参考链接

*   David Walsh, [Fullscreen API](http://davidwalsh.name/fullscreen)
*   David Storey, [Is your Fullscreen API code up to date? Find out how to make it work the same in modern browsers](http://generatedcontent.org/post/70347573294/is-your-fullscreen-api-code-up-to-date-find-out-how-to)