# 实时通信开发指南 &middot; JavaScript 

<div class="callout callout-info">本文介绍 JavaScript 实时通讯 SDK version 3 的使用，version 2 的文档请参考[《JavaScript 实时通信开发指南（version 2）》](./js_realtime.html)。</div>

## 简介

实时通信服务可以让你一行后端代码都不用写，就能做出一个功能完备的实时聊天应用，或是一个实时对战类的游戏。所有聊天记录都保存在云端，离线消息会通过消息推送来及时送达，推送消息文本可以灵活进行定制。

>在继续阅读本文档之前，请先阅读[《实时通信开发指南》](./realtime_v2.html)，了解一下实时通信的基本概念和模型。

### 兼容性

JavaScript 实时通信 SDK 支持如下运行时：

- 浏览器/WebView
  - IE 10+ / Edge
  - Chrome 31+
  - Firefox latest
  - iOS 8.0+
  - Android 4.4+
- Node.js 0.12+
- React Native
  - iOS 0.22+
  - Android 0.25+

### 文档贡献
我们欢迎和鼓励大家对本文档的不足提出修改建议。请访问我们的 [Github 文档仓库](https://github.com/leancloud/docs) 来提交 Pull Request。

### API 文档

[https://leancloud.github.io/js-realtime-sdk/docs/](https://leancloud.github.io/js-realtime-sdk/docs/)

## 安装和初始化
### 安装
建议使用 npm 安装 SDK，在终端运行以下命令：
```bash
npm install leancloud-realtime --save
```

### 引用
SDK 暴露（export）了以下成员：[SDK API 文档](https://leancloud.github.io/js-realtime-sdk/docs/module-leancloud-realtime.html)。

如果是在浏览器中使用，需要加载以下 script：
```html
<script src="./node_modules/leancloud-realtime/dist/realtime.browser.js"></script>
```
在浏览器中直接加载时，SDK 暴露的所有的成员都挂载在 `AV` 命名空间下:
```javascript
var Realtime = AV.Realtime;
var TextMessage = AV.TextMessage;
```

如果是在 Node.js 或其他支持 CommonJS 模块规范的环境中使用，需要按以下方法进行 require：
```javascript
var Realtime = require('leancloud-realtime').Realtime;
var TextMessage = require('leancloud-realtime').TextMessage;
```

### 初始化
按照上面的方式拿到 `Realtime` 类后，可以按照下面用法初始化一个 `realtime` 实例，在下面的文档中如果出现了未定义的 `realtime` 指的均是这个实例。

```javascript
var realtime = new Realtime({
  appId: '{{appid}}',
  region: 'cn', // 美国节点为 "us"
});
```

在 React Native 中使用时需要在初始化时指定 noBinary 为 true：
```javascript
const realtime = new Realtime({
  appId: '{{appid}}',
  region: 'cn', // 美国节点为 "us"
  noBinary: true,
});
```

### 富媒体消息插件
如果需要使用 [富媒体消息](#富媒体消息) 中的 `ImageMessage`、`AudioMessage`、`VideoMessage`、`FileMessage` 或 `LocationMessage`，需要额外安装 leancloud-realtime-plugin-typed-messages 与 leancloud-storage：
```bash
npm install --save leancloud-realtime-plugin-typed-messages leancloud-storage
```

在浏览器中使用时按照以下顺序加载：
```html
<script src="./node_modules/leancloud-storage/dist/av.js"></script>
<script src="./node_modules/leancloud-realtime/dist/realtime.browser.js"></script>
<script src="./node_modules/leancloud-realtime-plugin-typed-messages/dist/typed-messages.js"></script>
```
然后依次进行初始化：
```javascript
// 初始化存储 SDK
AV.initialize('{{appid}}', '{{appkey}}');
// 初始化实时通讯 SDK
var Realtime = AV.Realtime;
var realtime = new Realtime({
  appId: '{{appid}}',
  plugins: [AV.TypedMessagesPlugin], // 注册富媒体消息插件
});
// 在浏览器中直接加载时，富媒体消息插件暴露的所有的成员都挂载在 AV 命名空间下
var imageMessage = new AV.ImageMessage(file);
```

如果是在 Node.js 或其他支持 CommonJS 模块规范的环境中使用，需要按以下方法进行引用与初始化：
```javascript
var AV = require('leancloud-storage');
var Realtime = require('leancloud-realtime').Realtime;
var TypedMessagesPlugin = require('leancloud-realtime-plugin-typed-messages').TypedMessagesPlugin;
var ImageMessage = require('leancloud-realtime-plugin-typed-messages').ImageMessage;

// 初始化存储 SDK
AV.initialize('{{appid}}', '{{appkey}}');
// 初始化实时通讯 SDK
var realtime = new Realtime({
  appId: '{{appid}}',
  plugins: [TypedMessagesPlugin], // 注册富媒体消息插件
});
var imageMessage = new ImageMessage(file);
```

富媒体消息插件暴露（export）的成员完整列表请参见： [富媒体消息插件 API 文档](https://leancloud.github.io/js-realtime-sdk/plugins/typed-messages/docs/module-leancloud-realtime-plugin-typed-messages.html)


## 单聊

我们先从最简单的环节入手。此场景类似于微信的私聊、微博的私信和 QQ 单聊。我们创建了一个统一的概念来描述聊天的各种场景：Conversation（对话），在[《实时通信开发指南》](./realtime_v2.html) 里也有相关的详细介绍。

### 发送消息

![Tom and Jerry](images/tom-and-jerry-avatar.png)

Tom 想给 Jerry 发一条消息，实现代码如下：

```javascript
// Tom 用自己的名字作为 clientId，获取 IMClient 对象实例
realtime.createIMClient('Tom').then(function(tom) {
  // 创建与Jerry之间的对话
  return tom.createConversation({
    members: ['Jerry'],
    name: 'Tom & Jerry',
  });
}).then(function(conversation) {
  // 发送消息
  return conversation.send(new AV.TextMessage('耗子，起床！'));
}).then(function(message) {
  console.log('Tom & Jerry', '发送成功！');
}).catch(console.error);

// https://jsplay.avosapps.com/fuq/embed?js,console
```

{% if node=='qcloud' %}
执行完以上代码，在 LeanCloud 网站的 控制台 /（选择应用）/ 存储 / 数据 / `_Conversation ` 表 中多了一行数据，其字段含义如下：
{% else %}
执行完以上代码，在 LeanCloud 网站的 [控制台 /（选择应用）/ 存储 / 数据 / `_Conversation ` 表](/data.html?appid={{appid}}#/_Conversation) 中多了一行数据，其字段含义如下：
{% endif %}

名称|类型|描述
---|---|---
name|String|对话唯一的名字
m|Array|对话中成员的列表
lm|Date|对话中最后一条消息发送的时间
c|String|对话的创建者的 ClientId
mu|Array|对话中设置了静音的成员，仅针对 iOS 以及 Windows Phone 用户有效。

>提示：每次调用 `createConversation()` 方法，都会生成一个新的 Conversation 实例，即便使用相同 members 和 name 也是如此。如果想要不重复创建相同成员的对话，请参阅 [常见问题](#常见问题) 。

### 接收消息

要让 Jerry 收到 Tom 的消息，需要这样写：

```javascript
// Jerry 登录
realtime.createIMClient('Jerry').then(function(jerry) {
  jerry.on('message', function(message, conversation) {
    console.log('Message received: ' + message.text);
  });
}).catch(console.error);

// https://jsplay.avosapps.com/nuh/embed?js,console
```



## 群聊

对于多人同时参与的固定群组，我们有成员人数限制，最大不能超过 500 人。对于另外一种多人聊天的形式，譬如聊天室，其成员不固定，用户可以随意进入发言的这种「临时性」群组，后面会单独介绍。

### 发送消息

Tom 想建立一个群，把自己好朋友都拉进这个群，然后给他们发消息，他需要做的事情是：

1. 建立一个朋友列表
2. 新建一个对话，把朋友们列为对话的参与人员
3. 发送消息

```javascript
// Tom 用自己的名字作为 clientId，获取 Client 对象实例
realtime.createIMClient('Tom').then(function(tom) {
  // 创建与 Jerry,Bob,Harry,William 之间的对话
  return tom.createConversation({
    members: ['Jerry', 'Bob', 'Harry', 'William'],
    name: 'Tom & Jerry & friends',
  })
}).then(function(conversation) {
  // 发送消息
  return conversation.send(new AV.TextMessage('你们在哪儿？'));
}).then(function(message) {
  console.log('发送成功！');
}).catch(console.error);

// https://jsplay.avosapps.com/vep/embed?js,console
```
### 接收消息

群聊的接收消息与单聊的接收消息在代码写法上是一致的。

```javascript

// Bob 登录
realtime.createIMClient('Bob').then(function(bob) {
  bob.on('message', function(message, conversation) {
    console.log('[Bob] received a message from [' + message.from + ']: ' + message.text);
    // 收到消息之后一般的做法是做 UI 展现，示例代码在此处做消息回复，仅为了演示收到消息之后的操作，仅供参考。
    conversation.send(new AV.TextMessage('Tom，我在 Jerry 家，你跟 Harry 什么时候过来？还有 William 和你在一起么？'));
  });
}).catch(console.error);

// William 登录
realtime.createIMClient('William').then(function(william) {
  william.on('message', function(message, conversation) {
    console.log('[William] received a message from [' + message.from + ']: ' + message.text);
  });
}).catch(console.error);

// https://jsplay.avosapps.com/coz/embed?js,console
```

以上由 Tom 和 Bob 发送的消息，William 在上线时都会收到。

由此可以看出，**群聊和单聊本质上都是对话**，只是参与人数不同。单聊是一对一的对话，群聊是多对多的对话。

用户在开始聊天之前，需要先登录 LeanCloud 云端。这个登录并不需要用户名和密码认证，只是与 LeanCloud 云端建立一个长连接，所以只需要传入一个唯一标识作为当前用户的 `clientId` 即可。

为直观起见，我们使用了 Tom、Jerry 等字符串作为 clientId 登录聊天系统。LeanCloud 云端只要求 clientId 在应用内唯一、不超过 64 个字符的字符串即可，具体用什么数据由应用层决定。

实时通信 SDK 在内部会为每一个 clientId 创建唯一的 `Client` 实例，也就是说多次使用相同的 clientId 创建出来的实例还是同一个。因此，如果要支持同一个客户端内多账号登录，只要使用不同的 clientId 来创建多个实例即可。我们的 SDK 也支持多账户同时登录。

## 消息

消息是一个对话的基本组成部分，我们支持的消息类型有：

- 文本消息：`TextMessage`
- 图像消息：`ImageMessage`
- 音频消息：`AudioMessage`
- 视频消息：`VideoMessage`
- 文件消息：`FileMessage`
- 位置消息：`LocationMessage`

除了 `TextMessage` 已经内置，其他的消息类型需要额外安装插件 leancloud-realtime-plugin-typed-messages，具体的安装与初始化方法参见 [安装 - 富媒体消息插件](#富媒体消息插件)。

### 富媒体消息

#### 发送消息

##### 图像消息、音频消息、视频消息、文件消息

图像可以通过浏览器或 Node.js 提供的 API 获取，也可以用有效的图像 URL。先使用存储 SDK 的 `AV.File` 类 [构造出一个文件对象](leanstorage_guide-js.html#文件)，再调用其 `save` 方法将其保存到服务端，然后把它当做参数构造一个 `ImageMessage` 的实例，最后通过 `Conversation#send` 方法即可发送这条消息。

音频消息、视频消息、文件消息的构造与发送与图像消息类似，不再赘述。

###### 发送图像消息

【场景一】用浏览器提供的 API 去获取本地的照片，然后构造出 `ImageMessage` 来发送：



```javascript
/* html: <input type="file" id="photoFileUpload"> */

var fileUploadControl = $('#photoFileUpload')[0];
var file = new AV.File('avatar.jpg', fileUploadControl.files[0]);
file.save().then(function() {
  var message = new AV.ImageMessage(file);
  message.setText('发自我的小米');
  message.setAttributes({ location: '旧金山' });
  return conversation.send(message);
}).then(function() {
  console.log('发送成功');
}).catch(console.error.bind(console));

// http://jsplay.avosapps.com/vug/embed?html,js,console,output
```


【场景二】从微博上复制的一个图像链接来创建图像消息：


```javascript
var file = new AV.File.withURL('萌妹子', 'http://pic2.zhimg.com/6c10e6053c739ed0ce676a0aff15cf1c.gif');
file.save().then(function() {
  var message = new AV.ImageMessage(file);
  message.setText('萌妹子一枚');
  return conversation.send(message);
}).then(function() {
  console.log('发送成功');
}).catch(console.error.bind(console));

// http://jsplay.avosapps.com/poj/embed?js,console
```


以上两种场景对于 SDK 的区别为：

* 场景一：SDK 获取了完整的图像数据流，先上传文件到云端，再将文件 URL 包装在消息体内发送出去。
* 场景二：SDK 并没有将图像实际上传到云端，而仅仅把 URL 包装在消息体内发送出去。

需要特别指出，与其他 SDK 不同的是，由于 JavaScript 存储 SDK 在处理文件时不会自动获取图像文件的大小、宽高等元信息，在默认的情况下情况下接收方是无法从消息体中获取图像的元信息数据，但是接收方可以自行通过客户端技术去分析图片的格式、大小、长宽之类的元数据。或者你也可以通过其他方式获得图像的元信息，然后通过 `AV.File#metaData` 方法手动设置这些信息。


##### 地理位置消息

先使用存储 SDK 的 `AV.GeoPoint` 类 [构造出一个地理位置对象](leanstorage_guide-js.html#地理位置)，然后把它当做参数构造一个 `LocationMessage` 的实例，最后通过 `Conversation#send` 方法即可发送这条消息。

```javascript
var location = new AV.GeoPoint(31.3753285,120.9664658);
var message = new AV.LocationMessage(location);
message.setText('新开的蛋糕店！耗子咱们有福了…');
conversation.send(message).then(function() {
  console.log('发送成功');
}).catch(console.error.bind(console));

// http://jsplay.avosapps.com/xol/embed?js,console
```


#### 接收富媒体消息

实时通信 SDK 提供的所有富媒体消息类都是从 TypedMessage 派生出来的。发送的时候可以直接调用 `conversation.send()` 函数。在接收端，SDK 会在 IMClient 实例上派发 `message` 事件，接收端处理富媒体消息的示例代码如下：

```javascript
// 在初始化 Realtime 时，需加载 TypedMessagesPlugin
// var realtime = new Realtime({
//   appId: appId,
//   plugins: [AV.TypedMessagesPlugin,]
// });
// 注册 message 事件的 handler
client.on('message', function messageEventHandler(message, conversation) {
  // 请按自己需求改写
  var file;
  switch (message.type) {
    case AV.TextMessage.TYPE:
      console.log('收到文本消息， text: ' + message.getText() + ', msgId: ' + message.id);
      break;
    case AV.FileMessage.TYPE:
      file = message.getFile(); // file 是 AV.File 实例
      console.log('收到文件消息，url: ' + file.url() + ', size: ' + file.metaData('size'));
      break;
    case AV.ImageMessage.TYPE:
      file = message.getFile();
      console.log('收到图片消息，url: ' + file.url() + ', width: ' + file.metaData('width'));
      break;
    case AV.AudioMessage.TYPE:
      file = message.getFile();
      console.log('收到音频消息，url: ' + file.url() + ', width: ' + file.metaData('duration'));
      break;
    case AV.VideoMessage.TYPE:
      file = message.getFile();
      console.log('收到视频消息，url: ' + file.url() + ', width: ' + file.metaData('duration'));
      break;
    case AV.LocationMessage.TYPE:
      var location = message.getLocation();
      console.log('收到位置消息，latitude: ' + location.latitude + ', longitude: ' + location.longitude);
      break;
    default:
      console.warn('收到未知类型消息');
  }
});

// http://jsplay.avosapps.com/fux/embed?js,console
```

同时，对应的 conversation 上也会派发 `message` 事件：
```javascript
conversation.on('message', function messageEventHandler(message) {
  // your logic
});
```


### 暂态消息

暂态消息不会被自动保存（以后在历史消息中无法找到它），也不支持延迟接收，离线用户更不会收到推送通知，所以适合用来做控制协议。譬如聊天过程中「某某正在输入...」这样的状态信息，就适合通过暂态消息来发送；或者当群聊的名称修改以后，也可以用暂态消息来通知该群的成员「群名称被某某修改为...」。



```javascript
// operation-message.js

var { TypedMessage, messageType, messageField } = require('leancloud-realtime');
// 自定义的消息类型，用于发送和接收所有的用户操作消息
// 这里使用了 TypeScript 的语法，也可以使用其他的继承机制的实现，详见「自定义消息类型」章节

// 指定 type 类型，可以根据实际换成其他正整数
@messageType(1)
@messageField('op')
class OperationMessage extends TypedMessage {}

// app.js

realtime.createIMClient('tom').then(function(tom) {
  return tom.createConversation({
    members: ['bob'],
  });
}).then(function(conversation) {
  var message = new OperationMessage();
  message.op = 'typing';
  // 设置该条消息为暂态消息
  message.setTransient(true);
  return conversation.send(message);
}).then(function() {
  console.log('发送成功');
}).catch(console.error.bind(console));

// http://jsplay.avosapps.com/cew/embed?js,console,output
```


而对话中的其他成员在程序中需要有以下代码做出响应：



```javascript
// operation-message.js 同发送

// app.js

// 首先需要注册自定义消息类型
realtime.register(OperationMessage);
realtime.createIMClient('bob').then(function(bob) {
  // 注册 message 事件的 handler
  client.on('message', function messageEventHandler(message, conversation) {
    switch (message.type) {
      case OperationMessage.TYPE:
        console.log(message.from + ' is ' + message.op);
        break;
      // case ...
      default:
        console.warn('收到未知类型消息');
    }
  });
});

// http://jsplay.avosapps.com/cew/embed?js,console,output
```


### 消息送达回执

是指消息被对方收到之后，云端会发送一个回执通知给发送方，表明消息已经送达。

发送时标记消息为「需要回执」：

```javascript
var message = new AV.TextMessage('very important message');
message.setNeedReceipt(true);
conversation.send(message);
```

当消息的接收方收到消息后，服务端会通知消息的发送方「消息已送达」，发送方的 SDK 会在 conversation 上派发一个 `receipt` 事件：

```javascript
conversation.on('receipt', function(payload) {
  // payload.message 为送达的消息，与先前发送的是同一实例
  // message.status 更新为 MessageStatus.DELIVERED
  // message.deliveredAt 为消息送达的时间
  console.log(payload.message);
});
```

需要注意的是：

> 只有在发送时设置了「需要回执」标记，云端才会发送回执，默认不发送回执。该回执并不代表用户已读。

### 未读消息

未读消息有两种处理方式，未读消息数量通知与离线消息通知。

#### 未读消息数量通知

未读消息数量通知是默认的未读消息处理方式：当客户端上线时，会收到其参与过的会话的未读消息数量的通知，然后由客户端负责主动拉取未读的消息并手动标记为已读。

当收到未读消息数量通知时，SDK 会在 Client 上派发 `unreadmessages` 事件。

```javascript
client.on('unreadmessages', function unreadMessagesEventHandler(payload, conversation) {
  console.log(payload);
  // {
  //   count: 4,
  //   lastMessageId: "UagNXHK0RHqIvM_VB7Injg",
  //   lastMessageTimestamp: [object Date],
  // }
})

// http://jsplay.avosapps.com/xuc/embed?js,console
```

如果有多个对话有未读消息，这个事件会被派发多次，对应的 conversation 的未读消息数（`conversation.unreadMessagesCount`）会自动更新，此时开发者可以在对话列表界面上更新这些对话的未读消息数量。

当用户点击进入某个对话时，开发者需要做两件事：

0. 拉取消息记录，参见[聊天记录](#聊天记录)
0. 调用 `Conversation#markAsRead` 标记该会话为已读：

```javascript
conversation.markAsRead().then(function(conversation) {
  console.log('对话已标记为已读');
}).catch(console.error.bind(console));
```

此时，当前用户其他在线的客户端会收到 `unreadmessages` 消息，将该会话的未读消息数更新为 0。

除了 `Conversation#markAsRead`，SDK 还提供了 `IMClient#markAllAsRead` 方法来批量标记对话为已读：
```javascript
client.markAllAsRead([conversation]).then(function() {
  console.log('对话已全部标记已读');
}).catch(console.error.bind(console));
```

#### 离线消息通知

离线消息通知方式是指，当客户端上线时，服务器会主动将所有离线时收到的消息推送过来，每个对话最多推送 20 条最近的消息。当收到离线消息时，SDK 会在 Client 上派发 `messages` 事件，与在线时收到消息无异。

要使用离线消息通知方式，需要在初始化 Realtime 时设置参数 `pushOfflineMessages` 为 `true`：

```javascript
var realtime = new AV.Realtime({
  appId: '{{appid}}',
  pushOfflineMessages: true,
});
```

### 消息类详解

消息类型之间的关系

![消息的类图](http://ac-lhzo7z96.clouddn.com/1440485935481)




消息类均包含以下属性：

属性|类型|描述
---|---|---
from|String|消息发送者的 clientId
cid|String|消息所属对话 id
id|String|消息发送成功之后，由 LeanCloud 云端给每条消息赋予的唯一 id
timestamp|Date|消息发送的时间。消息发送成功之后，由 LeanCloud 云端赋予的全局的时间戳。
deliveredAt|Date|消息送达时间
status|Symbol|消息状态，其值为枚举 [`MessageStatus`](https://leancloud.github.io/js-realtime-sdk/docs/module-leancloud-realtime.html#.MessageStatus) 的成员之一：<br/><br/>`MessageStatus.NONE`（未知）<br/>`MessageStatus.SENDING`（发送中）<br/>`MessageStatus.SENT`（发送成功）<br/>`MessageStatus.DELIVERED`（已送达）<br/>`MessageStatus.FAILED`（失败）

我们为每一种富媒体消息定义了一个消息类型，实时通信 SDK 自身使用的类型是负数（如下面列表所示），所有正数留给开发者自定义扩展类型使用，0 作为「没有类型」被保留起来。

消息 | 类型
--- | ---
文本消息|-1
图像消息|-2
音频消息|-3
视频消息|-4
位置消息|-5
文件消息|-6



### 自定义消息

在某些场景下，开发者需要在发送消息时附带上自己业务逻辑需求的自定义属性，比如消息发送的设备名称，或是图像消息的拍摄地点、视频消息的来源等等，开发者可以通过  实现这一需求。

【场景】发照片给朋友，告诉对方照片的拍摄地点：



```javascript
// predefined: someAVFile, conversation
var message = new AV.ImageMessage(someAVFile);
message.setAttributes({
  location: '拉萨布达拉宫',
  title: '这蓝天……我彻底是醉了',
});
conversation.send(message).then(function() {
  console.log('发送成功');
}).catch(console.error.bind(console));
```


接收时可以读取这一属性：



```javascript
// predefined: client
client.on('message', function(message) {
  console.log(message.getAttributes().location); // 拉萨布达拉宫
});
```


所有的 `TypedMessage` 消息都支持 `attributes` 这一属性。

#### 创建新的消息类型


通过继承 TypedMessage，开发者也可以扩展自己的富媒体消息。其要求和步骤是：

* 申明新的消息类型，继承自 TypedMessage 或其子类，然后：
  * 对 class 使用 `messageType(123)` 装饰器，具体消息类型的值（这里是 `123`）由开发者自己决定（LeanCloud 内建的 [消息类型使用负数](#消息类详解)，所有正数都预留给开发者扩展使用）。
  * 对 class 使用 `messageField(['fieldName'])` 装饰器来声明需要发送的字段。
* 调用 `Realtime#register()` 函数注册这个消息类型。

举个例子，实现一个在 [暂态消息](#暂态消息) 中提出的 OperationMessage：

```javascript
// TypedMessage, messageType, messageField 都是由 leancloud-realtime 这个包提供的
// 在浏览器中则是 var { TypedMessage, messageType, messageField } = AV;
var { TypedMessage, messageType, messageField } = require('leancloud-realtime');
var inherit = require('inherit');
// 定义 OperationMessage 类，用于发送和接收所有的用户操作消息
export const OperationMessage = inherit(TypedMessage);
// 指定 type 类型，可以根据实际换成其他正整数
messageType(1)(OperationMessage);
// 申明需要发送 op 字段
messageField('op')(OperationMessage);
// 注册消息类，否则收到消息时无法自动解析为 OperationMessage
realtime.register(OperationMessage);
```


> **什么时候需要自己创建新的消息类型？**
>
>譬如有一条图像消息，除了文本之外，还需要附带地理位置信息，为此开发者需要创建一个新的消息类型吗？从上面的例子可以看出，其实完全没有必要。这种情况只要使用消息类中预留的 `attributes` 属性就可以保存额外的地理位置信息了。
>
>只有在我们的消息类型完全无法满足需求的时候，才需要扩展自己的消息类型。譬如「今日头条」里面允许用户发送某条新闻给好友，在展示上需要新闻的标题、摘要、图片等信息（类似于微博中的 linkcard）的话，这时候就可以扩展一个新的 NewsMessage 类。

## 对话

以上章节基本演示了实时通信 SDK 的核心概念「对话」，即 `Conversation`。我们将单聊和群聊（包括聊天室）的消息发送和接收都依托于 `Conversation` 这个统一的概念进行操作，所以开发者需要强化理解的一个概念就是：
>SDK 层面不区分单聊和群聊。



对话的管理包括「成员管理」和「属性管理」两个方面。

在讲解下面的内容之前，我们先使用 Jerry 的身份登录并创建一个多人对话。后面的举例中 `jerry` 指 Jerry 登录的 client，conversation 指创建好的这个对话，CONVERSATION_ID 指这个对话的 ID。


```javascript
realtime.createIMClient('Jerry').then(function(jerry) {
  return jerry.createConversation({
    members: ['Bob', 'Harry', 'William'],
  });
}).then(function(conversation) {
  var CONVERSATION_ID = conversation.id;
  // now we have jerry, conversation and CONVERSATION_ID
})
```



### 创建对话

```javascript
jerry.createConversation({
  members: ['Bob', 'Harry', 'William'],
  name: '周末滑雪',
  location: '42.86335,140.6843287',
  transient: false,
  unique: false,
});
```
参数说明：

* members - 对话的初始成员列表。在对话创建成功后，这些成员会收到和邀请加入对话一样的相应通知。
* name - 对话的名字，主要是用于标记对话，让用户更好地识别对话。
* transient - 是否为 [暂态对话](#聊天室)
* unique - 是否创建唯一对话，当其为 true 时，如果当前已经有**相同成员**的对话存在则返回该对话，否则会创建新的对话。该值默认为 false。

option 参数中所有其他的字段（如上面例子中的 `location`）都会作为对话的自定义属性保存。

<div class="callout callout-info">由于暂态对话不支持创建唯一对话，所以将 `transient` 和 `unique` 同时设为 true 时并不会产生预期效果。</div>

### 对话的成员管理

成员管理，是在对话中对成员的一个实时生效的操作，一旦操作成功则不可逆。

#### 成员变更接口
成员变更操作接口简介如下表：

操作目的|接口名
----|---
自身主动加入 |  `Conversation#join`
添加其他成员 |  `Conversation#add`
自身主动退出 |  `Conversation#quit`
移除其他成员 |  `Conversation#remove`

#### 成员变更事件

成员变动之后，所有在线的对话成员，都会得到相应的通知。SDK 会在 client 上派发对应的事件：

```javascript
// 有用户被添加至某个对话
jerry.on('membersjoined', function membersjoinedEventHandler(payload, conversation) {
  console.log(payload.members, payload.invitedBy, conversation.id);
});
// 有成员被从某个对话中移除
jerry.on('membersleft', function membersleftEventHandler(payload, conversation) {
  console.log(payload.members, payload.kickedBy, conversation.id);
});
// 当前用户被添加至某个对话
jerry.on('invited', function invitedEventHandler(payload, conversation) {
  console.log(payload.invitedBy, conversation.id);
});
// 当前用户被从某个对话中移除
jerry.on('kicked', function kickedEventHandler(payload, conversation) {
  console.log(payload.kickedBy, conversation.id);
});
```

同时在相应的 conversation 上也会派发同样的事件：

```javascript
// 有用户被添加至某个对话
conversation.on('membersjoined', function membersjoinedEventHandler(payload) {
  console.log(payload.members, payload.invitedBy);
});
// 有成员被从某个对话中移除
conversation.on('membersleft', function membersleftEventHandler(payload) {
  console.log(payload.members, payload.kickedBy);
});
// 当前用户被添加至某个对话
conversation.on('invited', function invitedEventHandler(payload) {
  console.log(payload.invitedBy);
});
// 当前用户被从某个对话中移除
conversation.on('kicked', function kickedEventHandler(payload) {
  console.log(payload.kickedBy);
});
```

#### 添加成员

##### 自身主动加入

Tom 想主动加入 Jerry、Bob、Harry 和 William 的对话，以下代码将帮助他实现这个功能：



```javascript
realtime.createIMClient('Tom').then(function(tom) {
  return tom.getConversation(CONVERSATION_ID);
}).then(function(conversation) {
  return conversation.join();
}).then(function(conversation) {
  console.log('加入成功', conversation.members);
  // 加入成功 ['Bob', 'Harry', 'William', 'Tom']
}).catch(console.error.bind(console));
```


##### 添加其他成员

Jerry 想再把 Mary 加入到对话中，需要如下代码帮助他实现这个功能：


```javascript
conversation.add(['Mary']).then(function(conversation) {
  console.log('添加成功', conversation.members);
  // 添加成功 ['Bob', 'Harry', 'William', 'Tom', 'Mary']
}).catch(console.error.bind(console));
```

##### 添加成员相关事件

添加成员后，对话中的成员会收到事件的通知，各方收到的事件是这样的：

邀请者|被邀请者|其他人
---|---|---
`membersjoined`|`invited` 与 `membersjoined`|`membersjoined`


>注意：如果在进行邀请操作时，被邀请者不在线，那么通知消息并不会被离线缓存，所以等到 Ta 再次上线的时候将不会收到通知。

#### 移除成员

##### 自身退出对话

Tom 主动从对话中退出，他需要如下代码实现需求：



```javascript
conversation.quit().then(function(conversation) {
  console.log('退出成功', conversation.members);
  // 退出成功 ['Bob', 'Harry', 'William', 'Mary']
}).catch(console.error.bind(console));
```


##### 移除其他成员

Harry 被 William 从对话中删除。实现代码如下（关于 William 如何获得权限在后面的 [安全与签名](#安全与签名) 中会做详细阐述，此处不扩大话题范围。）：



```javascript
realtime.createIMClient('William').then(function(william) {
  return william.getConversation(CONVERSATION_ID);
}).then(function(conversation) {
  return conversation.remove(['Harry']);
}).then(function(conversation) {
  console.log('移除成功', conversation.members);
  // 移除成功 ['Bob', 'William', 'Mary']
}).catch(console.error.bind(console));
```



##### 移除成员相关事件

移除成员后，对话中的成员会收到事件的通知，各方收到的事件是这样的：

操作者|被移除者|其他人
---|---|---
`membersleft`|`kicked`|`membersleft`


>注意：如果在进行踢人操作时，被踢者不在线，那么通知消息并不会被离线缓存，所以等到 Ta 再次上线的时候将不会收到通知。

#### 查询成员数量
除了直接访问 `conversation.members.length`，也可以通过 `Conversation#count` 方法获得当前对话的成员数量：



```javascript
conversation.count().then(function(membersCount) {
  console.log(membersCount);
}).catch(console.error.bind(console));
```


### 对话的属性管理

对话实例（Conversation）与控制台中 `_Conversation` 表是一一对应的，默认提供的属性的对应关系如下：


Conversation 属性名 | _Conversation 字段|含义
--- | ------------ | -------------
`id`| `objectId` |全局唯一的 Id
`name` |  `name` |成员共享的统一的名字
`members`|`m` |成员列表
`creator` | `c` |对话创建者
`transient`|`tr`|是否为聊天室（暂态对话）
`mutedMembers`|`mu`|静音该对话的成员
`muted`|N/A|当前用户是否静音该对话
`createdAt`|`createdAt`|创建时间
`updatedAt`|`updatedAt`|最后更新时间
`lastMessageAt`|`lm`|最后一条消息发送时间，也可以理解为最后一次活跃时间
`lastMessage`|N/A|最后一条消息，可能会空
`unreadMessagesCount`|N/A|未读消息数




#### 名称

这是一个全员共享的属性，它可以在创建时指定，也可以在日后的维护中被修改。

Tom 想建立一个名字叫「喵星人」 对话并且邀请了好友 Black 加入对话：



```javascript
tom.createConversation({
  members: ['Black'],
  name: '喵星人',
}).then(function(conversation) {
  console.log('创建成功。id: ' + conversation.id + ' name: ' + conversation.name);
}).catch(console.error.bind(console));
```


Black 发现对话名字不够酷，他想修改成「聪明的喵星人」 ，他需要如下代码：



```javascript
black.getConversation(CONVERSATION_ID).then(function(conversation) {
  conversation.name = '聪明的喵星人';
  return conversation.save();
}).then(function(conversation) {
  console.log('更新成功。name: ' + conversation.name);
}).catch(console.error.bind(console));
```


####  成员

是当前对话中所有成员的 `clientId`。默认情况下，创建者是在包含在成员列表中的，直到 TA 退出对话。

>**切勿在控制台中对其进行修改**。所有关于成员的操作请参照上一章节中的 [对话的成员管理](#对话的成员管理) 来进行。

#### 静音
假如某一用户不想再收到某对话的消息提醒，但又不想直接退出对话，可以使用静音操作，即开启「免打扰模式」。

比如 Tom 工作繁忙，对某个对话设置了静音：



```javascript
black.getConversation(CONVERSATION_ID).then(function(conversation) {
  return conversation.mute();
}).then(function(conversation) {
  console.log('静音成功');
}).catch(console.error.bind(console));
```


>设置静音之后，iOS 和 Windows Phone 的用户就不会收到推送消息了。

与之对应的就是取消静音的操作，即取消免打扰模式。此操作会修改云端 `_Conversation` 里面的 `mu` 属性。**切勿在控制台中对 `mu` 进行修改**。

#### 创建者

即对话的创建者，它的值是对话创建者的 `clientId`。

它等价于 QQ 群中的「群创建者」，但区别于「群管理员」。比如 QQ 群的「创建者」是固定不变的，它的图标颜色与「管理员」的图标颜色都不一样。所以根据对话中成员的 `clientId` 是否与 `conversation.creator` 一致就可以判断出他是不是群的创建者。

#### 自定义属性

开发者可以为对话添加自定义属性，来满足业务逻辑需求。

给某个对话加上两个自定义的属性：type = "private"（类型为私有）、pinned = true（置顶显示）：


```javascript
tom.createConversation({
  members: ['Jerry'],
  name: '猫和老鼠',
  type: 'private',
  pinned: true,
}).then(function(conversation) {
  console.log('创建成功。id: ' + conversation.id);
}).catch(console.error.bind(console));
```




**自定义属性在 SDK 级别是对所有成员可见的**。要对属性进行查询，请参见 [对话的查询](#对话的查询)。

### 对话的查询

<!-- #### 基础查询 -->

#### 根据 id 查询

假如已知某一对话的 Id，可以使用它来查询该对话的详细信息：



```javascript
tom.getConversation(CONVERSATION_ID).then(function(conversation) {
  console.log(conversation.id);
}).catch(console.error.bind(console));
```


#### 对话列表

用户登录进应用后，获取最近的 10 个对话（包含暂态对话，如聊天室）：



```javascript
tom.getQuery().containsMembers(['Tom']).find().then(function(conversations) {
  // 默认按每个对话的最后更新日期（收到最后一条消息的时间）倒序排列
  conversations.map(function(conversation) {
    console.log(conversation.lastMessageAt.toString(), conversation.members);
  });
}).catch(console.error.bind(console));

// http://jsplay.avosapps.com/biy/embed?js,console
```


对话的查询默认返回 10 个结果，若要更改返回结果数量，请设置 `limit` 值。



```javascript
var query = tom.getQuery();
query.limit(20).containsMembers(['Tom']).find().then(function(conversations) {
  console.log(conversations.length);
}).catch(console.error.bind(console));

// http://jsplay.avosapps.com/mej/embed?js,console
```


#### 条件查询


```javascript
// 查询对话名称为「LeanCloud 粉丝群」的对话
query.equalTo('name', 'LeanCloud 粉丝群');

// 查询对话名称包含 「LeanCloud」 的对话
query.contains('name', 'LeanCloud');

// 查询过去24小时活跃的对话
var yesterday = new Date(Date.now() - 24 * 3600 * 1000);
query.greaterThan('lm', yesterday);
```


条件查询又分为：比较查询、正则匹配查询、包含查询，以下会做分类演示。

##### 比较查询

比较查询在一般的理解上都包含以下几种：



比较查询最常用的是等于查询：



```javascript
// topic 是自定义属性
query.equalTo('topic','movie');
```



下面检索一下类型不是私有的对话：



```javascript
// type 是自定义属性
query.notEqualTo('type','private');
```


对于可以比较大小的整型、浮点等常用类型，可以参照以下示例代码进行扩展：



```javascript
// age 是自定义属性
query.greaterThan('age',18);
```


##### 正则匹配查询


匹配查询是指在 `ConversationQuery` 的查询条件中使用正则表达式来匹配数据。


比如要查询所有 language 是中文的对话：



```javascript
// 自定义属性 language 是中文字符
query.matches('language',/[\\u4e00-\\u9fa5]/);
```


##### 包含查询

包含查询是指方法名字包含 `Contains` 单词的方法，例如查询关键字包含「教育」的对话：



```javascript
// 自定义属性 keywords 包含「教育」
query.contains('keywords','教育');
```


另外，包含查询还能检索与成员相关的对话数据。以下代码将帮助 Tom 查找出 Jerry 以及 Bob 都加入的对话：



```javascript
// 查询对话成员有 Bob 和 Jerry 的 conversations
query.withMembers(['Bob', 'Jerry']);
```


##### 组合查询

组合查询的概念就是把诸多查询条件合并成一个查询，再交给 SDK 去云端进行查询。

例如，要查询年龄小于 18 岁，并且关键字包含「教育」的对话：



```javascript
// 查询 keywords 包含「教育」且 age 小于 18 的对话
query.contains('keywords', '教育').lessThan('age', 18);
```

#### 查询结果选项

##### 排序

`ConversationQuery` 支持使用 `ascending`、`addAscending`、`descending`、`addDescending` 方法来对查询结果进行排序：

```javascript
// 对查询结果按照 name 升序，然后按照创建时间降序排序
query.addAscending('name').addDescending('createdAt');
```

##### 精简模式

普通对话最多可以容纳 500 个成员，在有些业务逻辑不需要对话的成员列表的情况下，可以使用 `ConversationQuery` 的 `compact` 方法指定查询为「精简模式」，返回的查询结果中则不会有成员列表（`members` 字段会是空数组），这有助于提升应用的性能同时减少流量消耗。

```javascript
query.compact(true);
```

##### 对话的最后一条消息

对于一个聊天应用，一个典型的需求是在对话的列表界面显示最后一条消息，默认情况下，`ConversationQuery` 的查询结果是不带最后一条消息的，使用 `withLastMessagesRefreshed` 方法可以指定让查询结果带上最后一条消息：

```javascript
query.withLastMessagesRefreshed(true);
```

需要注意的是，这个选项真正的意义是「刷新对话的最后一条消息」。这意味着由于 SDK 缓存机制的存在，将这个选项设置为 `false` 查询得到的对话也还是有可能会存在最后一条消息的。

#### 缓存查询

JavaScript SDK 会对按照对话 id 对对话进行内存字典缓存，但不会进行持久化的缓存。


## 聊天室

聊天室本质上就是一个对话，所以上面章节提到的**所有属性、方法、操作以及管理都适用于聊天室**。它仅仅在逻辑上是一种暂态、临时的对话，应用场景有弹幕、直播等等。

聊天室与普通对话或群聊不一样的地方具体体现为：

* 无人数限制，而普通对话最多允许 500 人加入。
* 不支持查询成员列表，但可以通过相关 API 查询在线人数。
* 不支持离线消息、离线推送通知、消息回执等功能。
* 没有成员加入、成员离开的通知。
* 一个用户一次登录只能加入一个聊天室，加入新的聊天室后会自动离开原来的聊天室。
* 加入后半小时内断网重连会自动加入原聊天室，超过这个时间则需要重新加入。

### 创建聊天室


建立一个聊天室需要在 `IMClient#createConversation()` 时传入 `transient=true`。


比如喵星球正在直播选美比赛，主持人 Tom 创建了一个临时对话，与喵粉们进行互动：



```javascript
tom.createConversation({
  name: 'Hello Kitty PK 加菲猫',
  transient: true,
}).then(function(conversation) {
  console.log('创建聊天室成功。id: ' + conversation.id);
}).catch(console.error.bind(console));
```




### 查询在线人数

 `Conversation.count()`  可以用来查询普通对话的成员总数，在聊天室中，它返回的就是实时在线的人数：



```javascript
conversation.count().then(function(count) {
  console.log('在线人数: ' + count);
}).catch(console.error.bind(console));
```


### 查找聊天室

开发者需要注意的是，通过 `IMClient#getQuery()` 这样得到的 `ConversationQuery` 实例默认是查询全部对话的，也就是说，如果想查询指定的聊天室，需要限定 `tr` 字段的查询条件：

比如查询主题包含「奔跑吧，兄弟」的聊天室：



```javascript
var query = tom.getQuery();
query
  .equalTo('topic', '奔跑吧，兄弟')
  .equalTo('tr', true)
  .find()
  .then(function(conversations) {
    console.log(conversations[0].id);
  })
  .catch(console.error.bind(console));
```




## 聊天记录

聊天记录一直是客户端开发的一个重点，QQ 和 微信的解决方案都是依托客户端做缓存，当收到一条消息时就按照自己的业务逻辑存储在客户端的文件或者是各种客户端数据库中。

我们的 SDK 会将普通的对话消息自动保存在云端，开发者可以通过 `Conversation#queryMessages` 方法来获取该对话的所有历史消息。

获取该对话中最近的 N 条（默认 20，最大值 1000）历史消息，通常在第一次进入对话时使用：



```javascript
conversation.queryMessages({
  limit: 10, // limit 取值范围 1~1000，默认 20
}).then(function(messages) {
  // 最新的十条消息，按时间增序排列
}).catch(console.error.bind(console));
```


对于翻页加载更多历史消息的场景，SDK 还提供了 `Conversation#createMessagesIterator` 方法来生成一个历史消息迭代器。假如每一页为 10 条信息，下面的代码将演示如何翻页：



```javascript
// 创建一个迭代器，每次获取 10 条历史消息
var messageIterator = conversation.createMessagesIterator({ limit: 10 });
// 第一次调用 next 方法，获得前 10 条消息，还有更多消息，done 为 false
messageIterator.next().then(function(result) {
  // result: {
  //   value: [message1, ..., message10],
  //   done: false,
  // }
}).catch(console.error.bind(console));
// 第二次调用 next 方法，获得第 11 ~ 20 条消息，还有更多消息，done 为 false
messageIterator.next().then(function(result) {
  // result: {
  //   value: [message11, ..., message20],
  //   done: false,
  // }
}).catch(console.error.bind(console));
// 第二次调用 next 方法，获得第 21 条消息，没有更多消息，done 为 true
messageIterator.next().then(function(result) {
  // No more messages
  // result: { value: [message21], done: true }
}).catch(console.error.bind(console));
```



### 客户端聊天记录缓存

JavaScript SDK 没有客户端聊天记录缓存机制

## 客户端事件

### 网络状态响应

>注意：在网络中断的情况下，所有的消息收发和对话操作都会失败。开发者应该监听与网络状态相关的事件并更新 UI，以免影响用户的使用体验。

当网络连接出现中断、恢复等状态变化时，SDK 会在 Realtime 实例上派发以下事件：

* `disconnect`：网络连接断开，此时聊天服务不可用。
* `schedule`：计划在一段时间后尝试重连，此时聊天服务仍不可用。
* `retry`：正在重连。
* `reconnect`：网络连接恢复，此时聊天服务可用。

```javascript
realtime.on('disconnect', function() {
  console.log('网络连接已断开');
});
realtime.on('schedule', function(attempt, delay) {
  console.log(delay + 'ms 后进行第' + (attempt + 1) + '次重连');
});
realtime.on('retry', function(attempt) {
  console.log('正在进行第' + attempt + '次重连');
});
realtime.on('reconnect', function() {
  console.log('网络连接已恢复');
});
```



在 `schedule` 与 `retry` 事件之间，开发者可以调用 `Realtime#retry()` 方法手动进行重连。

在断线重连的过程中，SDK 也会在所有的 IMClient 实例上派发同名的事件。Realtime 与 IMClient 上的同名事件是先后同步派发的，唯一的例外是 `reconnect` 事件。在网络连接恢复，Realtime 上派发了 `reconnect` 事件之后，IMClient 会尝试重新登录，成功后再派发 `reconnect` 事件。所以，Realtime 的 `reconnect` 事件意味着 Realtime 实例的 API 能够正常使用了，IMClient 的 `reconnect` 事件意味着 IMClient 实例的 API 能够正常使用了。

下面显示的是一次典型的断线重连过程中 SDK 派发的事件：

```c
// 连接断开，计划 1s 后重连
[Realtime & IMClient] disconnect
[Realtime & IMClient] schedule (attempt=0, delay=1000)
// 1s 后，尝试重连
[Realtime & IMClient] retry (attempt=0)
// 重连失败，计划 2s 后进行第二次重连
[Realtime & IMClient] schedule (attempt=1, delay=2000)
// 在 2s 内，手动调用 realtime.retry() 进行重连，重连次数重置
[Realtime & IMClient] retry (attempt=0)
// 重连失败，计划 2s 后进行第二次重连
[Realtime & IMClient] schedule (attempt=1, delay=2000)
// 2s 后，尝试第二次重连
[Realtime & IMClient] retry (attempt=1)
// 连接恢复，此时可以创建新的客户端了
[Realtime] reconnect
// 客户端重新登录上线，此时该客户端可以收发消息了
[IMClient] reconnect
```

## 退出登录

tom 要退出当前的登录状态或要切换账户，方法如下：



```javascript
tom.close().then(function() {
  console.log('Tom 退出登录');
}).catch(console.error.bind(console));
```




## 安全与签名

在继续阅读下文之前，请确保你已经对 [实时通信服务开发指南 &middot; 权限和认证](realtime_v2.html#权限和认证) 有了充分的了解。

### 实现签名工厂

为了满足开发者对权限和认证的要求，我们设计了操作签名的机制。签名启用后，所有的用户登录、对话创建/加入、邀请成员、踢出成员等登录都需要验证签名，这样开发者就对消息具有了完全的掌控。

{% if node=='qcloud' %}
我们强烈推荐启用签名，具体步骤是 `控制台 > 设置 > 应用选项`，勾选 **聊天、推送** 下的 **聊天服务，启用签名认证**。
{% else %}
我们强烈推荐启用签名，具体步骤是 [控制台 > 设置 > 应用选项](/app.html?appid={{appid}}#/permission)，勾选 **聊天、推送** 下的 **聊天服务，启用签名认证**。
{% endif %}


客户端这边究竟该如何使用呢？我们只需要实现 signature 工厂方法，然后作为参数实例化 IMClient 即可

设定了 signature 工厂方法后，对于需要鉴权的操作，实时通信 SDK 与服务器端通讯的时候都会带上应用自己生成的 Signature 信息，LeanCloud 云端会使用 app 的 masterKey 来验证信息的有效性，保证聊天渠道的安全。

对于不同的操作，我们需要实现两个不同的 signature 工厂方法：`signatureFactory` 与 `conversationSignatureFactory`。

```javascript
/**
 * IMClient 登录签名工厂
 *
 * @param {String} clientId 登录用户 ID
 * @return {Object} signatureResult
 * @return {String} signatureResult.signature
 * @return {Number} signatureResult.timestamp
 * @return {String} signatureResult.nonce
 */
var signatureFactory = function(clientId) {
  // to be implemented
};

/**
 * Conversation 相关操作签名工厂
 *
 * @param {String} conversationId
 * @param {String} clientId 当前用户 ID
 * @param {String[]} targetIds 此次操作的目标用户 IDs
 * @param {String} action  此次行为的动作，可能的值为 create（创建会话）、add（加群和邀请）和 remove（踢出群）之一
 * @return {Object} signatureResult
 * @return {String} signatureResult.signature
 * @return {Number} signatureResult.timestamp
 * @return {String} signatureResult.nonce
 */
var conversationSignatureFactory = function(clientId) {
  // to be implemented
};
```

`signatureFactory` 函数会在用户登录的时候被调用，`conversationSignatureFactory` 会在对话创建/加入、邀请成员、踢出成员等操作时被调用。

你需要做的就是按照前文所述的签名算法实现签名，返回 signatureResult，其中四个属性分别是:

* signature 签名
* timestamp 时间戳，单位秒
* nonce 随机字符串 nonce

如果签名是异步的，比如需要发送一个网络请求，那么 signature 工厂方法也可以返回一个 Promise，resolved with signatureResult。

下面的代码展示了基于 LeanCloud 云引擎进行签名时，客户端的实现片段，你可以参考它来完成自己的逻辑实现：

```javascript
var signatureFactory = function(clientId) {
  return AV.Cloud.rpc('sign', { clientId: clientId }); // AV.Cloud.rpc returns a Promise
};
var conversationSignatureFactory = function(conversationId, clientId, targetIds, action) {
  return AV.Cloud.rpc('sign-conversation', {
    conversationId: conversationId,
    clientId: clientId,
    targetIds: targetIds,
    action: action,
  });
};

realtime.createIMClient('Tom', {
  signatureFactory: signatureFactory,
  conversationSignatureFactory: conversationSignatureFactory,
}).then(function(tom) {
  console.log('Tom 登录');
}).catch(function(error) {
  // 如果 signatureFactory 抛出了异常，或者签名没有验证通过，会在这里被捕获
});
```


> 需要强调的是：开发者切勿在客户端直接使用 MasterKey 进行签名操作，因为 MaterKey 一旦泄露，会造成应用的数据处于高危状态，后果不容小视。因此，强烈建议开发者将签名的具体代码托管在安全性高稳定性好的服务器上（例如 LeanCloud 云引擎）。


### 单点登录

一款聊天应用，随着不断的发展，会衍生出多个平台的不同客户端。以 QQ 为例，目前它所提供的客户端如下：

- PC：Windows PC、Mac OS、Linux（已停止更新）
- 移动：Windows Phone、iOS、Android
- Web：<http://w.qq.com/>

经过测试，我们发现 QQ 存在以下几种行为：

1. 同一个 QQ 账号不可以同时在 2 个 PC 端登录（例如，在 Mac OS 上登录已经在另外一台 Windows PC 上登录的 QQ，该 QQ 号在 Windows PC 上会被强行下线）。
2. 同一个 QQ 账号不可以同时在 2 个移动端上登录。
3. Web QQ 也不能与 PC 端同时登录
4. 同一个 QQ 只能同时在 1 个移动版本和 1 PC 版本（或者 Web 版本）上登录，并实现一些 PC 与移动端互动的功能，例如互传文件。

通过规律不难发现，QQ 按照自己的需求实现了「单点登录」的功能：同一个平台上只允许一个 QQ 登录一台设备。

下面我们来详细说明：如何使用我们的 SDK 去实现单点登录。

#### 设置登录标记 Tag

假设开发者想实现 QQ 这样的功能，那么需要在登录到服务器的时候，也就是打开与服务器长连接的时候，标记一下这个链接是从什么类型的客户端登录到服务器的：



```javascript
realtime.createIMClient('Tom', null, 'Web').then(function(tom) {
  console.log('Tom 登录');
});
```



上述代码可以理解为 LeanCloud 版 QQ 的登录，而另一个带有同样 Tag 的客户端打开连接，则较早前登录系统的客户端会被强制下线。

#### 处理登录冲突

我们可以看到上述代码中，登录的 Tag 是 `Web`。当存在与其相同的 Tag 登录的客户端，较早前登录的设备会被服务端强行下线，而且他会收到被服务端下线的通知：



```javascriptjava
tom.on('conflict', function() {
  // 弹出提示，告知当前用户的 Client Id 在其他设备上登陆了
});
```

如上述代码中，当前用户被服务端强行下线时，SDK 会在 client 上派发 `conflict` 事件，客户端在做展现的时候也可以做出类似于 QQ 一样友好的通知。

## 插件

SDK 支持通过插件来对功能进行扩展，比如在解析消息前对原始消息进行修改，为内部的类添加方法，注册自定义消息等。

### 插件列表

请参阅 [https://github.com/leancloud/js-realtime-sdk/wiki/Plugins]()。

### 使用插件

Realtime 支持在初始化时传入指定一个 plugins 数组：

```javascript
var Realtime = require('leancloud-realtime').Realtime;
var WebRTCPlugin = require('leancloud-realtime-plugin-webrtc').WebRTCPlugin;

var realtime = new Realtime({
  appId: appId,
  plugins: [WebRTCPlugin],
});
```

插件的具体使用方式请参考具体插件的文档。

### 创建插件

#### 扩展点

一个插件是由一个或多个扩展点组成的字典（Object）。扩展点可以分为三类：

第一类扩展点是 SDK 内部类实例化之后的回调，包括 `onRealtimeCreated`、`onIMClientCreated` 与 `onConversationCreated`。这些扩展点可以通过一个方法（function）进行扩展，该方法接受一个对应的实例并对其进行一些操作。我们称这一类方法为 Decorator。插件可以利用这些扩展点为内部类添加新的方法或修改原有的方法。

下面这个例子利用了 `onConversationCreate` 扩展点，修改了 Conversation 的 quit 方法，在调用 quit 方法时统一弹出一个确认窗口。

```javascript
var ConfirmOnQuitPlugin = {
  name: 'leancloud-realtime-plugin-confirm-on-quit',
  onConversationCreate: function onConversationCreate(conversation) {
    var originalQuit = conversation.quit;
    conversation.quit = function() {
      var confirmed = window.confirm('退出会话？退出后将无法收到消息。');
      if (confirmed) {
        return originalQuit.apply(this, arguments);
      } else {
        return Promise.reject(new Error('user canceled'));
      }
    }
  }
};
```

第二类扩展点允许你在某些事件前、后注入逻辑。这些扩展点可以通过一个方法（function）进行扩展，该方法接受一个对象，返回一个同类型对象（如果该方法是异步的，则返回一个 Promise）。我们称这一类方法为 Middleware。
以消息解析为例，可以将 SDK 从接收原始消息 - 解析消息 - 派发的富媒体消息的过程看成一条管道，这些扩展点允许你在这个管道中加入一段你的节点，这个节点就是 Middleware。如果指定了多个 Middleware，这些 Middleware 会按照顺序依次执行，前一个 Middleware 的返回值会作为参数传给后一个 Middleware。

目前可扩展的点有:

- `beforeMessageParse`: 在解析消息前对原始消息进行处理，参数是 json 格式的原始消息
- `afterMessageParse`: 在解析消息后对消息进行处理，参数是对应的富媒体消息类的实例

举个例子，有一些对话中存在一些 FileMessage 类型的历史消息，由于某种原因缺少了必须的 file.id 字段，会导致解析到这些消息时 SDK 抛出异常。这时可以通过 `beforeMessageParse` 扩展点来在 SDK 解析消息前「修补」这个问题。

```javascript
var EnsureFileIdPlugin = {
  name: 'leancloud-realtime-plugin-ensure-file-id',
  beforeMessageParse: function onConversationCreate(message) {
    if (!message._lcfile.id) message._lcfile.id = '';
    return message;
  }
};
```

第三类扩展点是一个特殊的扩展点：`messageClasses`，这是一个由自定义消息类型组成的数组，数组中的自定义消息类型会被自动注册（通过 Realtime#register）。在富文本消息一节中用到的 TypedMessagesPlugin 就是使用了这个扩展点的插件。

如果有必要，我们会在未来开放更多的扩展点。

#### 插件规范

如果你的插件可能会被其他开发者用到，我们推荐你将其封装为一个 package 并发布到 npm 上，发布的插件请遵循以下规范：

- package 名称以 `leancloud-realtime-plugin-` 为前缀；
- 插件对象需要有 `name` 字段，用于在日志中显示异常的插件名称，建议与 package 名称相同。

## 从 v2 迁移
如果你的应用正在使用 JavaScript SDK version 2 并希望升级到 version 3，请参考 [《JavaScript 实时通信 SDK v3 迁移指南》](./realtime_js-v3-migration-guide.html)。


## 常见问题

**我只想实现两个用户的私聊，是不是每次都得重复创建对话？**

答：不需要重复创建。我们推荐的方式是开发者可以用**自定义属性**来实现对私聊和群聊的标识，并且在进行私聊之前，需要查询当前两个参与对话的 ClientId 是否之前已经存在一个私聊的对话了。另外，SDK 已经提供了创建唯一对话的接口，请查看 [创建对话](#创建对话)。


**某个成员退出对话之后，再加入，在他离开的这段期间内的产生的聊天记录，他还能获取么？**

答：可以。目前聊天记录从属关系是属于对话的，也就是说，只要对话 Id 不变，不论人员如何变动，只要这个对话产生的聊天记录，当前成员都可以获取。

**我自己没有服务器，如何实现签名的功能？**

答：LeanCloud 云引擎提供了托管 Python 和 Node.js 运行的方式，开发者可以所以用这两种语言按照签名的算法实现签名，完全可以支持开发者的自定义权限控制。

## 问题排查

1. 客户端连接被关闭有许多原因，请参考 [服务器端错误码说明](realtime_v2.html#服务器端错误码说明)。
