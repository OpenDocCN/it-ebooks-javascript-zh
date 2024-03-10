# 13.15 os 模块

os 模块用于与硬件设备通信。

## Socket 通信

下面例子列出当前系列的所有 IP 地址。

```
var os = require('os');
var interfaces = os.networkInterfaces();

for (item in interfaces) {
  console.log('Network interface name: ' + item);
  for (att in interfaces[item]) {
    var address = interfaces[item][att];

    console.log('Family: ' + address.family);
    console.log('IP Address: ' + address.address);
    console.log('Is Internal: ' + address.internal);
    console.log('');
  }
  console.log('==================================');
}
```