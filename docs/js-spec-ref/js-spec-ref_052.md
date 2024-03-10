# 7.1 HTML 网页元素

*   image 元素
    *   alt 属性，src 属性
    *   complete 属性
    *   height 属性，width 属性
    *   naturalWidth 属性，naturalHeight 属性
    *   audio 元素，video 元素

## image 元素

### alt 属性，src 属性

alt 属性返回 image 元素的 HTML 标签的 alt 属性值，src 属性返回 image 元素的 HTML 标签的 src 属性值。

```
// 方法一：HTML5 构造函数 Image
var img1 = new Image(); 
img1.src = 'image1.png';
img1.alt = 'alt';
document.body.appendChild(img1);

// 方法二：DOM HTMLImageElement
var img2 = document.createElement('img'); 
img2.src = 'image2.jpg';
img2.alt = 'alt text';
document.body.appendChild(img2);

document.images[0].src
// image1.png
```

### complete 属性

complete 属性返回一个布尔值，true 表示当前图像属于浏览器支持的图形类型，并且加载完成，解码过程没有出错，否则就返回 false。

### height 属性，width 属性

这两个属性返回 image 元素被浏览器渲染后的高度和宽度。

### naturalWidth 属性，naturalHeight 属性

这两个属性只读，表示 image 对象真实的宽度和高度。

```
myImage.addEventListener('onload', function() {
    console.log('My width is: ', this.naturalWidth);
    console.log('My height is: ', this.naturalHeight);
});
```

## audio 元素，video 元素

audio 元素和 video 元素加载音频和视频时，以下事件按次序发生。

*   loadstart：开始加载音频和视频。
*   durationchange：音频和视频的 duration 属性（时长）发生变化时触发，即已经知道媒体文件的长度。如果没有指定音频和视频文件，duration 属性等于 NaN。如果播放流媒体文件，没有明确的结束时间，duration 属性等于 Inf（Infinity）。
*   loadedmetadata：媒体文件的元数据加载完毕时触发，元数据包括 duration（时长）、dimensions（大小，视频独有）和文字轨。
*   loadeddata：媒体文件的第一帧加载完毕时触发，此时整个文件还没有加载完。
*   progress：浏览器正在下载媒体文件，周期性触发。下载信息保存在元素的 buffered 属性中。
*   canplay：浏览器准备好播放，即使只有几帧，readyState 属性变为 CAN_PLAY。
*   canplaythrough：浏览器认为可以不缓冲（buffering）播放时触发，即当前下载速度保持不低于播放速度，readyState 属性变为 CAN_PLAY_THROUGH。

除了上面这些事件，audio 元素和 video 元素还支持以下事件。

| 事件 | 触发条件 |
| --- | --- |
| abort | 播放中断 |
| emptied | 媒体文件加载后又被清空，比如加载后又调用 load 方法重新加载。 |
| ended | 播放结束 |
| error | 发生错误。该元素的 error 属性包含更多信息。 |
| pause | 播放暂停 |
| play | 暂停后重新开始播放 |
| playing | 开始播放，包括第一次播放、暂停后播放、结束后重新播放。 |
| ratechange | 播放速率改变 |
| seeked | 搜索操作结束 |
| seeking | 搜索操作开始 |
| stalled | 浏览器开始尝试读取媒体文件，但是没有如预期那样获取数据 |
| suspend | 加载文件停止，有可能是播放结束，也有可能是其他原因的暂停 |
| timeupdate | 网页元素的 currentTime 属性改变时触发。 |
| volumechange | 音量改变时触发（包括静音）。 |
| waiting | 由于另一个操作（比如搜索）还没有结束，导致当前操作（比如播放）不得不等待。 |