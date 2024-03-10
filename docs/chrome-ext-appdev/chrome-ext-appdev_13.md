# 第十章　其他接口

除上述接口外 Chrome 应用还有其他各类丰富的接口，在本章将对其他的接口做以介绍。

## 10.1　操作 USB 设备

通过`usb`接口可以与 USB 设备进行交互，这能让 Chrome 应用作为 USB 设备的驱动程序。要使用`usb`接口需要在 Manifest 中声明`usb`权限：

```
"permissions": [
    "usb"
] 
```

本章内容参考自[`crxdoc-zh.appspot.com/apps/usb`](https://crxdoc-zh.appspot.com/apps/usb)。

### 10.1.1　发现设备

列出指定的 USB 设备：

```
var options = {
    vendorId: 0x05ac,  //Apple, Inc.
    productId: 0x12a0  //iPhone 4s
};

chrome.usb.getDevices(options, function(deviceArray){
    //do something with deviceArray;
}); 
```

请求访问操作系统占用的设备（仅限 Chrome OS）：

```
chrome.usb.requestAccess(device, interfaceId, function(sucess){
    //sucess is boolean
}); 
```

其中`interfaceId`为接口标识符。

打开设备：

```
chrome.usb.openDevice(device, function(ConnectionHandle){
    //do something with handle
}); 
```

`ConnectionHandle`为连接句柄，包括三个属性，`handle`、`vendorId`和`productId`，其中 handle 为连接句柄的标识符。

寻找指定的 USB 设备（如果权限允许的话同时打开设备以便使用）：

```
var options = {
    vendorId: 0x05ac,  //Apple, Inc.
    productId: 0x12a0  //iPhone 4s
    interfaceId: null  //only for Chrome OS
};

chrome.usb.findDevices(options, function(ConnectionHandleArray){
    //do something with ConnectionHandleArray
}); 
```

关闭连接句柄：

```
chrome.usb.closeDevice(ConnectionHandle, function(){
    //do something after close a device
}); 
```

重置 USB 设备：

```
chrome.usb.resetDevice(ConnectionHandle, function(result){
    //result is boolean
}); 
```

### 10.1.2　操作接口

列举 USB 设备上的所有接口：

```
chrome.usb.listInterfaces(ConnectionHandle, function(descriptorsArray){
    //do something with descriptorsArray
}); 
```

`descriptorsArray`是一个包含多个`descriptors`对象的数组，`descriptors`包含的属性有`interfaceNumber`、`alternateSetting`、`interfaceClass`、`interfaceSubclass`、`interfaceProtocol`、`description`和`endpoints`。`endpoints`也是一个数组，每个对象包含的属性有`address`、`type`、`direction`、`maximumPacketSize`、`synchronization`、`usage`和`pollingInterval`。

在指定 USB 设备上获取接口：

```
chrome.usb.claimInterface(ConnectionHandle, interfaceNumber, function(){
    //do something after the interface is claimed
}); 
```

释放在提供的设备上获取的接口：

```
chrome.usb.releaseInterface(ConnectionHandle, interfaceNumber, function(){
    //do something after the interface is released
}); 
```

在之前获取的设备接口上选择替代的设置：

```
chrome.usb.setInterfaceAlternateSetting(ConnectionHandle, interfaceNumber, alternateSetting, function(){
    //do something after the interface setting is set
}); 
```

`alternateSetting`为要设置的替代设置。

### 10.1.3　操作传输

在指定设备上进行控制传输：

```
chrome.usb.controlTransfer(ConnectionHandle, transferInfo, function(info){
    //do something with info
}); 
```

其中`transferInfo`为对象，包含的属性有`direction`、`recipient`、`requestType`、`request`、`value`、`index`、`length`和`data`。如果输出数据，`transferInfo.data`必须指定，值的类型为`ArrayBuffer`。

`info`为对象，包含的属性有`resultCode`和`data`，`resultCode`为 0 时表示成功，`data`为传入数据，类型为`ArrayBuffer`。

在指定设备上进行大块传输：

```
chrome.usb.bulkTransfer(ConnectionHandle, transferInfo, function(info){
    //do something with info
}); 
```

`transferInfo`为对象，包含的属性有`direction`、`endpoint`、`length`和`data`。此方法返回的`info`类型与`controlTransfer`相同。

在指定设备上进行中断传输：

```
chrome.usb.interruptTransfer(ConnectionHandle, transferInfo, function(info){
    //do something with info
}); 
```

此方法各参数类型与`bulkTransfer`相同。

在指定设备上进行同步传输：

```
chrome.usb.isochronousTransfer(ConnectionHandle, transferInfo, function(info){
    //do something with info
}); 
```

此方法各参数类型与`bulkTransfer`相同。

## 10.2　串口通信

通过`serial`接口可以使 Chrome 应用进行串口通信。使用`serial`接口需要在 Manifest 中声明`serial`权限：

```
"permissions": [
    "serial"
] 
```

本章内容参考自[`crxdoc-zh.appspot.com/apps/serial`](https://crxdoc-zh.appspot.com/apps/serial)。

### 10.2.1　建立连接

获取系统中可用串行设备的有关信息：

```
chrome.serial.getDevices(function(portsArray){
    //do something with portsArray
}); 
```

`portsArray`为包含多个`ports`的数组。每个`ports`是一个包含多个属性的对象，其属性有`path`、`vendorId`、`productId`和`displayName`。

连接到指定的串行端口：

```
chrome.serial.connect(path, options, function(connectionInfo){
    //do something with connectionInfo
}); 
```

其中`options`为端口配置选项，完整的结构如下：

```
{
    persistent: 应用关闭时连接是否保持打开状态,
    name: 与连接相关联的字符串,
    bufferSize: 用于接收数据的缓冲区大小,
    bitrate: 打开连接时请求的比特率,
    dataBits: 默认为"eight",
    parityBit: 默认为"no",
    stopBits: 默认为"one",
    ctsFlowControl: 是否启用 RTS/CTS 硬件流控制,
    receiveTimeout: 等待新数据的最长时间，以毫秒为单位,
    sendTimeout: 等待 send 操作完成的最长时间，以毫秒为单位
} 
```

更新打开的串行端口连接的选项设置：

```
chrome.serial.update(connectionId, options, function(result){
    //result is boolean
}); 
```

断开串行端口连接：

```
chrome.serial.disconnect(connectionId, function(result){
    //result is boolean
}); 
```

暂停或恢复打开的连接：

```
chrome.serial.setPaused(connectionId, paused, function(){
    //do something after pause a connection
}); 
```

### 10.2.2　发送和接收数据

向指定连接写入数据：

```
chrome.serial.send(connectionId, data, function(sendInfo){
    //do something with sendInfo
}); 
```

`sendInfo`为一个包含两个属性的对象，分别为`bytesSent`和`error`。

清除指定连接输入输出缓存中的所有内容：

```
chrome.serial.flush(connectionId, function(result){
    //result is boolean
}); 
```

当接收到数据时，会触发`onReceive`事件：

```
chrome.serial.onReceive.addListener(function(info){
    //do something with info
}); 
```

`info`是一个包含两个属性的对象，分别是`connectionId`和`data`。

当发生错误时，会触发`onReceiveError`事件：

```
chrome.serial.onReceiveError.addListener(function(info){
    //do something with info
}); 
```

`info`为一个包含两个属性的对象，分别为`connectionId`和`error`。

### 10.2.3　获取连接及状态

获取指定连接的状态：

```
chrome.serial.getInfo(connectionId, function(connectionInfo){
    //do something with connectionInfo
}); 
```

获取当前应用拥有并打开的串行端口连接列表：

```
chrome.serial.getConnections(function(connectionInfoArray){
    //do something with connectionInfoArray
}); 
```

获取指定连接上控制信号的状态：

```
chrome.serial.getControlSignals(connectionId, function(signals){
    //do something with signals
}); 
```

`signals`是一个包含 4 个属性的对象，分别是`dcd`、`cts`、`ri`和`dsr`。

设置指定连接上控制信号的状态：

```
chrome.serial.setControlSignals(connectionId, signals, function(result){
    //result is boolean
}); 
```

`signals`参数为一个包含两个属性的对象，分别是`dtr`和`rts`。

## 10.3　文字转语音

使用`tts`接口可以将文字转换为语音，`tts`接口可以使用不同语速、音调阅读文字。文字转语音对视力不佳的用户来说非常重要。

要在应用中使用`tts`接口，需要在 Manifest 的`permissions`中声明`tts`权限：

```
"permissions": [
    "tts"
] 
```

### 10.3.1　朗读文字

使用`speak`方法来朗读文字：

```
chrome.tts.speak('Hello, world.'); 
```

`speak`方法还可以指定朗读参数和回调函数：

```
chrome.tts.speak(utterance, options, callback); 
```

回调函数`callback`会在`speak`方法调用成功后立刻执行，这意味着不会等到朗读结束后才调用`callback`。`options`指定了朗读时所采用的语调、语速、音量等等，`options`完整的结构如下所示：

```
{
    enqueue: 是否将朗读任务放入队列，如果为 true，此朗读任务将在之前的任务结束后才开始,
    voiceName: 朗读所使用的声音名称,
    extensionId: 为朗读提供声音引擎扩展的 id,
    lang: 所朗读文字的语言,
    gender: 朗读声音所使用的性别，male 或 female,
    rate: 朗读语速，默认值为 1.0，允许的值为 0.1 到 10.0，但具体范围还要结合具体使用的声音，值越大速度越快,
    pitch: 朗读语调，默认值为 1.0，允许的值为 0 到 2.0,
    volume: 朗读音量，默认值为 1.0，允许的值为 0 到 1.0,
    requiredEventTypes: 声音必须支持的事件,
    desiredEventTypes: 需要监听的事件，如未指定则监听全部事件,
    onEvent: 用于监听事件的函数
} 
```

如下面的例子，用美式英文以正常语速的 2 倍阅读“Hello, world.”：

```
chrome.tts.speak('Hello, world.', {lang: 'en-US', rate: 2.0}); 
```

而下一个例子会在“Speak this first.”阅读完毕后才阅读“Speak this next, when the first sentence is done.”：

```
chrome.tts.speak('Speak this first.');
chrome.tts.speak('Speak this next, when the first sentence is done.', {enqueue: true}); 
```

使用`chrome.runtime.lastError`来抓取`tts`接口使用中可能发生的错误：

```
chrome.tts.speak(utterance, options, function() {
    if (chrome.runtime.lastError) {
        console.log('Error: ' + chrome.runtime.lastError.message);
    }
}); 
```

使用`stop`方法可以随时停止正在进行的朗读任务：

```
chrome.tts.stop(); 
```

使用`pause`方法可以随时暂停正在进行的朗读任务：

```
chrome.tts.pause(); 
```

使用`resume`方法可以随时恢复被暂停的朗读任务：

```
chrome.tts.resume(); 
```

### 10.3.2　获取声音

通过`getVoices`方法可以获取到目前计算机中提供的声音：

```
chrome.tts.getVoices(function(voices){
    //do something with voices
}); 
```

返回的结果`voices`是一个包含多个声音对象的数组。声音对象包含 6 个属性，分别是`voiceName`、`lang`、`gender`、`remote`、`extensionId`和`eventTypes`，其中`remote`属性表示此声音是否为网络资源，`eventTypes`为此声音支持的全部事件。

如下面为一个声音对象的实例：

```
{
    "eventTypes": [
        "start",
        "end",
        "interrupted",
        "cancelled",
        "error"
    ],
    "extensionId": "neajdppkdcdipfabeoofebfddakdcjhd",
    "gender": "female",
    "lang": "en-GB",
    "remote": true,
    "voiceName": "Google UK English Female"
} 
```

获取到声音对象后，通过指定`speak`方法中的相应参数来应用声音，如：

```
chrome.tts.speak(utterance, {
    voiceName: 'Google UK English Female',
    lang: 'en-GB'
}, callback); 
```

### 10.3.3　获取朗读状态及监听事件

如果当前应用正在朗读文本，执行一个新的朗读任务会立即停止之前的朗读任务。为了避免打断正在进行的朗读任务，可以通过`isSpeaking`方法获取当前的朗读状态：

```
chrome.tts.isSpeaking(function(isSpeaking){
    //isSpeaking is boolean
}); 
```

在 10.3.1 中提到`speak`参数的`onEvent`属性用来监听朗读事件：

```
chrome.tts.speak(utterance,{
    onEvent: function(event) {
        console.log('Event ' + event.type ' at position ' + event.charIndex);
        if (event.type == 'error') {
            console.log('Error: ' + event.errorMessage);
        }
    }
}, callback); 
```

其中`event`对象包含三个属性，分别是`type`、`charIndex`和`errorMessage`。`type`为事件类型，可能的值包括`start`、`end`、`word`、`sentence`、`marker`、`interrupted`、`cancelled`、`error`、`pause`和`resume`。

朗读任务一开始就会监听到`start`类型事件，当朗读到一个新的词语时会监听到`word`类型事件，朗读完一个句子时会监听到`sentence`类型事件，当朗读任务被中断会监听到`interrupted`类型事件，而如果朗读任务尚未开始即被移除会监听到`cancelled`类型事件，`error`、`pause`和`resume`类型事件分别会在朗读过程中遇到错误、被暂停和被恢复时接收到。

对于`marker`类型事件，它是在朗读任务到达 SSML 标记时触发的，有关 SSML 的详细介绍请读者自行参考[`www.w3.org/TR/speech-synthesis/`](http://www.w3.org/TR/speech-synthesis/)，此处不做详细介绍。

不过实际能接收到的类型事件需要根据具体选择的声音的支持情况，在 10.3.2 中获取到的声音对象`eventTypes`属性列出了相应声音支持的全部事件类型。

## 10.4　系统信息

在讲解 Chrome 扩展时我们提到过获取 CPU、内存和存储设备信息的方法，具体可以参见 5.4 节。Chrome 应用也可以获取到系统信息，并且与 Chrome 扩展类似。

Chrome 应用可以获取到的系统信息包括 CPU、内存、存储设备、显示器和网卡。要获取信息，需要在 Manifest 中声明相应权限：

```
"permissions": [
    "system.cpu",
    "system.memory",
    "system.storage",
    "system.display",
    "system.network"
] 
```

由于 CPU、内存和存储设备相关的内容已经在 5.4 节讲过了，所以本节只讲解显示器和网卡的内容。

通过`chrome.system.display.getInfo`方法可以获取到显示器相关信息：

```
chrome.system.display.getInfo(function(displayInfoArray){
    //do something with displayInfoArray
}); 
```

`displayInfoArray`是一个包含多个`displayInfo`对象的数组。`displayInfo`对象的完整结构如下：

```
{
    id: 显示器的唯一 id,
    name: 显示器名称，如 HP LCD monitor,
    mirroringSourceId: 镜像的显示器 id，目前只支持 Chrome OS 平台,
    isPrimary: 是否为主显示器,
    isInternal: 是否为笔记本的自带显示器,
    isEnabled: 是否被启用,
    dpiX: 显示器水平 DPI,
    dpiY: 显示器垂直 DPI,
    rotation: 显示器旋转角度，目前只支持 Chrome OS 平台,
    bounds: {
        left: 显示器逻辑范围左上角横坐标,
        top: 显示器逻辑范围左上角纵坐标,
        width: 显示器逻辑范围像素宽度,
        height: 显示器逻辑范围像素高度
    },
    overscan: {
        left: 显示范围距左边框的距离,
        top: 显示范围距上边框的距离,
        right: 显示范围距右边框的距离,
        bottom: 显示范围距下边框的距离
    },
    workArea: {
        left: 工作区范围左上角横坐标,
        top: 工作区范围左上角纵坐标,
        width: 工作区范围像素宽度,
        height: 工作区范围像素高度
    }
} 
```

其中`overscan`属性目前只在 Chrome OS 平台有效，`workArea`的范围不包括系统占用部分，如任务栏等。

通过`chrome.system.display.setDisplayProperties`方法可以更改显示器设置，支持更改的属性包括`mirroringSourceId`、`isPrimary`、`overscan`、`rotation`、`boundsOriginX`和`boundsOriginY`。`boundsOriginX`和`boundsOriginY`对应于`bounds.left`和`bounds.top`，即显示器逻辑范围的原点坐标。

```
chrome.system.display.setDisplayProperties(id, info, function(){
    //do something after set display properties
}); 
```

通过`chrome.system.display.onDisplayChanged`监听显示器设置更改事件：

```
chrome.system.display.onDisplayChanged.addListener(function(){
    //do something after display properties are changed
}); 
```

通过`chrome.system.network.getNetworkInterfaces`方法可以获取到网卡信息：

```
chrome.system.network.getNetworkInterfaces(function(networkInterfaces){
    //do something with networkInterfaces
    console.log(networkInterfaces);
}); 
```

`networkInterfaces`是一个包含多个`networkInterface`对象的数组，`networkInterface`对象包含 3 个属性，分别是`name`、`address`和`prefixLength`。`name`为网卡的名称，在*nix 系统上通常是 eth0 或 wlan0 等。`address`为网卡可用的 IPv4 或 IPv6 地址。`prefixLength`为前缀长度。