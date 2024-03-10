# 附录 D　Chrome 扩展及应用完整 API 列表

以下内容来自[`crxdoc-zh.appspot.com`](https://crxdoc-zh.appspot.com)。请注意，这些列表中的内容可能会频繁地变化，尤其是 Beta 和 Dev 部分接口甚至可能会在未来的版本中消失，使用时应更为谨慎。最新的列表可以通过官方文档查看，扩展列表参见[`developer.chrome.com/extensions/api_index`](https://developer.chrome.com/extensions/api_index)，应用列表参见[`developer.chrome.com/apps/api_index`](https://developer.chrome.com/apps/api_index)。

## D.1　Chrome 扩展全部 API

**稳定 API**

| 名称 | 描述 | 最低版本 |
| alarms | 使用 `chrome.alarms` API 安排代码周期性地或者在将来的指定时间运行。 | 22 |
| bookmarks | 使用 `chrome.bookmarks` API 创建、组织以及通过其他方式操纵书签。您也可以参见替代页面，通过它您可以创建一个自定义的书签管理器页面。 | 5 |
| browserAction | 使用浏览器按钮可以在 Google Chrome 浏览器主窗口中地址栏右侧的工具栏中添加图标。除了弹出内容。 | 5 |
| browsingData | 使用 `chrome.browsingData` API 从用户的本地配置文件删除浏览数据。 | 19 |
| commands | 使用命令 API 添加快捷键，触发您的扩展程序中的操作，例如打开浏览器按钮或向扩展程序发送命令。 | 25 |
| contentSettings | 使用 `chrome.contentSettings` API 更改设置，控制网站能否使用 Cookie、JavaScript 和插件之类的特性。大体上说，内容设置允许您针对不同的站点（而不是全局地）自定义 Chrome 浏览器的行为。 | 16 |
| contextMenus | 使用 `chrome.contextMenus` API 向 Google Chrome 浏览器的右键菜单添加项目。您可以选择您在右键菜单中添加的项目应用于哪些类型的对象，例如图片、超链接和页面。 | 6 |
| cookies | 使用 `chrome.cookies` API 查询和修改 Cookie，并在 Cookie 更改时得到通知。 | 6 |
| debugger | `chrome.debugger` API 是 Chrome 远程调试协议（英文）的另一种消息传输方式。使用 `chrome.debugger` 可以附加到一个或多个标签页，以便查看网络交互、调试 JavaScript、改变 DOM 和 CSS 等等。使用调试对象的标签页标识符来指定 sendCommand 的目标标签页，并在 onEvent 的回调函数中通过标签页标识符分发事件。 | 18 |
| declarativeContent | 使用 `chrome.declarativeContent` API 根据网页内容采取行动，而不需要读取页面内容的权限。 | 33 |
| desktopCapture | 桌面捕获 API 可以用于捕获屏幕、单个窗口或标签页的内容。 | 34 |
| devtools.inspectedWindow | 使用 `chrome.devtools.inspectedWindow` API 与审查的窗口交互：获得审查页面的标签页标识符，在审查窗口的上下文中执行代码，重新加载页面，或者获取页面中所有资源的列表。 | 18 |
| devtools.network | 使用 `chrome.devtools.network` API 获取开发者工具的网络面板中显示的与网络请求相关的信息。 | 18 |
| devtools.panels | 使用 `chrome.devtools.panels` API 将您的扩展程序整合到开发者工具窗口用户界面中：创建您自己的面板、访问现有的面板以及添加侧边栏。 | 18 |
| downloads | 使用 `chrome.downloads` API 以编程方式开始下载，监视、操纵、搜索下载的文件。 | 31 |
| events | `chrome.events` 命名空间包含 API 分发事件使用的通用类型，以便在某些有意义的事情发生时通知您。 | 21 |
| extension | `chrome.extension` API 包含任何扩展程序页面都能使用的实用方法。它包括在扩展程序和内容脚本之间或者两个扩展程序之间交换消息的支持，这一部分内容在消息传递中详细描述。 | 5 |
| fileBrowserHandler | 使用 `chrome.fileBrowserHandler` API 扩展 Chrome OS 的文件浏览器。例如，您可以使用这一 API 让用户向您的网站上传文件。 | 12 |
| fontSettings | 使用 `chrome.fontSettings` API 管理 Chrome 浏览器的字体设置。 | 22 |
| history | 使用 `chrome.history` API 与浏览器的历史记录交互，您可以添加、删除、通过 URL 查询浏览器的历史记录。如果您想要使用您自己的版本替换默认的历史记录页面，请参见替代页面。 | 5 |
| i18n | 使用 `chrome.i18n` 架构为您的整个应用或扩展程序实现国际化支持。 | 5 |
| identity | 使用 `chrome.identity` API 获取 OAuth2 访问令牌。 | 29 |
| idle | 使用 `chrome.idle` API 检测计算机空闲状态的更改。 | 6 |
| input.ime | 使用 `chrome.input.ime` API 为 Chrome OS 实现自定义的输入法，它允许您的扩展程序处理键盘输入、设置候选内容及管理候选窗口。 | 21 |
| management | `chrome.management` API 可以用来管理已经安装并且正在运行的扩展程序或应用，它对于替代内建的“打开新的标签页”页面的扩展程序特别有用。 | 8 |
| notifications | 使用 `chrome.notifications` API 通过模板创建丰富通知，并在系统托盘中向用户显示这些通知。 | 28 |
| omnibox | 多功能框 API 允许您在 Google Chrome 浏览器的地址栏（又叫多功能框）中注册一个关键字。 | 9 |
| pageAction | 使用 `chrome.pageAction` API 在地址栏中添加图标。页面按钮代表用于当前页面的操作，但是不适用于所有页面。 | 5 |
| pageCapture | 使用 `chrome.pageCapture` API 将一个标签页保存为 MHTML。 | 18 |
| permissions | 使用 `chrome.permissions` API 在运行时而不是安装时请求声明的可选权限，这样用户可以理解为什么需要这些权限，并且仅在必要时授予这些权限。 | 16 |
| power | 使用 `chrome.power` API 修改系统的电源管理特性。 | 27 |
| privacy | 使用 `chrome.privacy` API 控制 Chrome 浏览器中可能会影响用户隐私的特性。这一模块依赖于类型 API 中的 ChromeSettings 原型，用于获取和设置 Chrome 浏览器的配置。 | 18 |
| proxy | 使用 `chrome.proxy` API 管理 Chrome 浏览器的代理服务器设置。该模块依赖于类型 API 中的 ChromeSetting 原型，用于获取和设置代理服务器配置。 | 13 |
| pushMessaging | 使用 `chrome.pushMessaging` 使应用或扩展程序能够接收通过 Google 云消息服务发送的消息数据。 | 24 |
| runtime | 使用 `chrome.runtime` API 获取后台页面、返回清单文件的详情、监听并响应应用或扩展程序生命周期内的事件，您还可以使用该 API 将相对路径的 URL 转换为完全限定的 URL。 | 22 |
| storage | 使用 `chrome.storage` API 存储、获取用户数据，追踪用户数据的更改。 | 20 |
| system.cpu | 使用 `systemInfo.cpu` API 查询 CPU 元数据。 | 32 |
| system.memory | `chrome.system.memory` API。 | 32 |
| system.storage | 使用 `chrome.system.storage` API 查询存储设备信息，并在连接或移除可移动存储设备时得到通知。 | 30 |
| tabCapture | 使用 `chrome.tabCapture` API 与标签页的媒体流交互。 | 31 |
| tabs | 使用 `chrome.tabs` API 与浏览器的标签页系统交互。您可以使用该 API 创建、修改和重新排列浏览器中的标签页。 | 5 |
| topSites | 使用 `chrome.topSites` API 访问“打开新的标签页”页面中的显示的“常去网站”。 | 19 |
| tts | 使用 `chrome.tts` API 播放合成的文字语音转换（TTS），同时请您参见相关的 ttsEngine API，允许扩展程序实现语音引擎。 | 14 |
| ttsEngine | 使用 `chrome.ttsEngine` API 用扩展程序实现文字语音转换（TTS）引擎。如果您的扩展程序注册了这一 API，当任何扩展程序或 Chrome 应用使用 tts 模块朗读时，它会收到事件，包含要朗读的内容以及其他参数。您的扩展程序可以使用任何可用的网络技术合成并输出语音，并向调用方发送事件报告状态。 | 14 |
| types | `chrome.types` API 包含用于 Chrome 浏览器的类型声明。 | 13 |
| webNavigation | 使用 `chrome.webNavigation` API 实时地接收有关导航请求状态的通知。 | 16 |
| webRequest | 使用 `chrome.webRequest` API 监控与分析流量，还可以实时地拦截、阻止或者修改请求。 | 17 |
| webstore | 使用 `chrome.webstore` API 从您的网站上“内嵌”安装应用与扩展程序。 | 15 |
| windows | 使用 `chrome.windows` API 与浏览器窗口交互。您可以使用该模块创建、修改和重新排列浏览器中的窗口。 | 5 |

**Beta API**

| 名称 | 描述 |
| accessibilityFeatures | 使用 `chrome.accessibilityFeatures` API 管理 Chrome 浏览器的辅助功能。该 API 使用类型 API 的 ChromeSetting 原型获取和设置辅助功能的各种特性。如果要获取特性的状态，扩展程序必须请求 `accessibilityFeatures.read` 权限。如果要修改特性状态，扩展程序需要 `accessibilityFeatures.modify` 权限。注意，`accessibilityFeatures.modify` 权限并不包含 `accessibilityFeatures.read` 权限。 |
| declarativeWebRequest | 使用 `chrome.declarativeWebRequest` API 实时地拦截、阻止或者修改请求，它比 API 要快得多，因为您注册的规则在浏览器而不是 JavaScript 引擎中求值，这样就减少了来回延迟并且可以获得极高的效率。 |
| gcm | 使用 `chrome.gcm` 通过 Google Cloud Messaging 在应用和扩展程序中发送和接收消息。 |

**Dev API**

| 名称 | 描述 |
| infobars | 使用 `chrome.infobars` API 在标签页内容的正上方添加一个水平面板，如以下屏幕截图所示。 |
| location | 使用 `chrome.location` API 获取计算机的地理位置。该 API 是 HTML 地理定位 API 的另一种版本，与事件页面兼容。 |
| processes | 使用 `chrome.processes` API 与浏览器进程交互。 |
| sessions | 使用 `chrome.sessions` API 查询和恢复浏览器会话中的标签页和窗口。 |
| signedInDevices | 使用 `chrome.signedInDevices` API 获取以当前配置文件所对应的账户登录的设备列表。 |

## D.2　Chrome 应用全部 API

**稳定 API**

| 名称 | 描述 | 最低版本 |
| alarms | 使用 `chrome.alarms` API 安排代码周期性地或者在将来的指定时间运行。 | 22 |
| app.runtime | 使用 `chrome.app.runtime` API 管理应用的生命周期。应用运行时环境管理应用的安装，控制事件页面，并且可以在任何时候关闭应用。 | 23 |
| app.window | 使用 `chrome.app.window` API 创建窗口。窗口可以有框架，包含标题栏和大小控件，它们不和任何 Chrome 浏览器窗口关联。 | 23 |
| contextMenus | 使用 `chrome.contextMenus` API 向 Google Chrome 浏览器的右键菜单添加项目。您可以选择您在右键菜单中添加的项目应用于哪些类型的对象，例如图片、超链接和页面。 | 6 |
| events | `chrome.events` 命名空间包含 API 分发事件使用的通用类型，以便在某些有意义的事情发生时通知您。 | 21 |
| fileSystem | 使用 `chrome.fileSystem` API 创建、读取、浏览与写入用户本地文件系统中经过沙盒屏蔽的一个区域。使用该 API，Chrome 应用可以读取和写入用户选定的位置，例如文本编辑应用可以使用该 API 读取和写入本地文档。所有失败信息都通过 runtime.lastError 通知。 | 23 |
| i18n | 使用 `chrome.i18n` 架构为您的整个应用或扩展程序实现国际化支持。 | 5 |
| identity | 使用 `chrome.identity` API 获取 OAuth2 访问令牌。 | 29 |
| idle | 使用 `chrome.idle` API 检测计算机空闲状态的更改。 | 6 |
| mediaGalleries | 使用 `chrome.mediaGalleries` API 从用户的本地磁盘（包含用户内容）中访问媒体文件（音频、图片、视频）。 | 23 |
| notifications | 使用 `chrome.notifications` API 通过模板创建丰富通知，并在系统托盘中向用户显示这些通知。 | 28 |
| permissions | 使用 `chrome.permissions` API 在运行时而不是安装时请求声明的可选权限，这样用户可以理解为什么需要这些权限，并且仅在必要时授予这些权限。 | 16 |
| power | 使用 `chrome.power` API 修改系统的电源管理特性。 | 27 |
| pushMessaging | 使用 `chrome.pushMessaging` 使应用或扩展程序能够接收通过 Google 云消息服务发送的消息数据。 | 24 |
| runtime | 使用 `chrome.runtime` API 获取后台页面、返回清单文件的详情、监听并响应应用或扩展程序生命周期内的事件，您还可以使用该 API 将相对路径的 URL 转换为完全限定的 URL。 | 22 |
| serial | 使用 `chrome.serial` API 读取和写入连接到串行端口的设备。 | 23 |
| socket | 使用 `chrome.socket` API 使用 TCP 和 UDP 连接通过网络发送和接收数据。**注意：**从 Chrome 33 开始该 API 弃用，您应该改用 sockets.udp、sockets.tcp 和 sockets.tcpServer API。 | 24 |
| sockets.tcp | 使用 `chrome.sockets.tcp` API 使用 TCP 连接通过网络发送和接收数据。该 API 是对原先 `chrome.socket` API 中 TCP 功能的增强。 | 33 |
| sockets.tcpServer | 使用 `chrome.sockets.tcpServer` API 创建使用 TCP 连接的服务器应用。该 API 是对原先 `chrome.socket` API 中 TCP 功能的增强。 | 33 |
| sockets.udp | 使用 `chrome.sockets.udp` API 使用 UDP 连接通过网络发送和接收数据。该 API 是对原先套接字 API 中 UDP 功能的增强。 | 33 |
| storage | 使用 `chrome.storage` API 存储、获取用户数据，追踪用户数据的更改。 | 20 |
| syncFileSystem | 使用 `chrome.syncFileSystem` API 在 Google 云端硬盘上保存和同步数据。该 API *并不是*用来访问存储在 Google 云端硬盘上的任何用户文档的，它提供了应用专用的可同步存储，用于离线和缓存用途，这样同样的数据就可以在不同的客户端间使用。有关使用该 API 的更多信息，请阅读管理数据。 | 27 |
| system.cpu | 使用 `systemInfo.cpu` API 查询 CPU 元数据。 | 32 |
| system.display | 使用 `system.display` API 查询显示器的元数据。 | 30 |
| system.memory | `chrome.system.memory` API。 | 32 |
| system.network | 使用 `chrome.system.network` API 获取网络接口信息。 | 33 |
| system.storage | 使用 `chrome.system.storage` API 查询存储设备信息，并在连接或移除可移动存储设备时得到通知。 | 30 |
| tts | 使用 `chrome.tts` API 播放合成的文字语音转换（TTS），同时请您参见相关的 ttsEngine API，允许扩展程序实现语音引擎。 | 14 |
| types | `chrome.types` API 包含用于 Chrome 浏览器的类型声明。 | 13 |
| usb | 使用 `chrome.usb` API 与已连接的 USB 设备交互。该 API 提供了在应用的环境中进行 USB 操作的能力，通过该 API 应用可以作为硬件设备的驱动程序使用。 | 26 |
| webstore | 使用 `chrome.webstore` API 从您的网站上“内嵌”安装应用与扩展程序。 | 15 |

**Beta API**

| 名称 | 描述 |
| accessibilityFeatures | 使用 `chrome.accessibilityFeatures` API 管理 Chrome 浏览器的辅助功能。该 API 使用类型 API 的 ChromeSetting 原型获取和设置辅助功能的各种特性。如果要获取特性的状态，扩展程序必须请求 `accessibilityFeatures.read` 权限。如果要修改特性状态，扩展程序需要 `accessibilityFeatures.modify` 权限。注意，`accessibilityFeatures.modify` 权限并不包含 `accessibilityFeatures.read` 权限。 |
| gcm | 使用 `chrome.gcm` 通过 Google Cloud Messaging 在应用和扩展程序中发送和接收消息。 |

**Dev API**

| 名称 | 描述 |
| audio | `chrome.audio` API 允许用户获取连接到系统的音频设备信息，并控制它们。目前该 API 仅在 Chrome OS 上实现。 |
| bluetooth | 使用 `chrome.bluetooth` API 连接到蓝牙设备。所有函数都通过 chrome.runtime.lastError 报告错误。 |
| location | 使用 `chrome.location` API 获取计算机的地理位置。该 API 是 HTML 地理定位 API 的另一种版本，与事件页面兼容。 |
| wallpaper | 使用 `chrome.wallpaper` API 更改 ChromeOS 壁纸。 |