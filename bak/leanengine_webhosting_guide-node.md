{# 指定继承模板 #}
{% extends "./leanengine_webhosting_guide.tmpl" %}

{% set productName = '云引擎' %}
{% set platformName = 'Node.js' %}
{% set fullName = productName + ' ' + platformName %}
{% set sdk_name = 'JavaScript' %}
{% set leanengine_middleware = '[LeanEngine Node.js SDK](https://github.com/leancloud/leanengine-node-sdk)' %}

{% block project_constraint %}
<script>$(window).load(function() {$('.do-expand-all').click();})</script>

你的项目根目录项目 **必须** 有一个 `package.json` 文件，才会正确地被云引擎识别为 Node.js 项目。

<div class="callout callout-info">因为一些历史遗留问题，请确保你的项目中 **没有** 名为 `cloud/main.js` 的文件。</div>

### package.json

Node.js 的 `package.json` 中可以指定 [很多选项](https://docs.npmjs.com/files/package.json)，它通常看起来是这样：

```json
{
    "name": "node-js-getting-started",
    "scripts": {
        "start": "node server.js"
    },
    "engines": {
        "node": "4.x"
    },
    "dependencies": {
        "express": "4.12.3",
        "leanengine": "1.2.2"
    }
}
```

其中云引擎会尊重的选项包括：

* `scripts.start` 启动项目时使用的命令；默认为 `node server.js`，如果你希望为 node 附加启动选项（如 `--es_staging`）或使用其他的文件作为入口点，可以修改该选项。
* `scripts.prepublish` 会在项目部署最后运行一次；可以将构建命令（如 `gulp build`）写在这里。
* `engines.node` 指定所需的 Node.js 版本；出于兼容性考虑默认版本仍为比较旧的 `0.12`，**因此建议大家自行指定一个更高的版本，建议使用 `4.x` 版本进行开发**，你也可以设置为 `*` 表示总是使用最新版本的 Node.js。
* `dependencies` 项目所依赖的包；云引擎会在部署时用 `npm install --production` 为你安装这里列出的所有依赖。
* `devDependencies` 项目开发时所依赖的包；云引擎目前 **不会** 安装这里的依赖。

建议你参考我们的 [项目模板](https://github.com/leancloud/node-js-getting-started/blob/master/package.json) 来编写自己的 `package.json`。
{% endblock %}

{% block project_start %}
{% endblock %}

{% block custom_runtime %}
{% endblock %}

{% block supported_frameworks %}
Node SDK 为 [express](http://expressjs.com/) 和 [koa](http://koajs.com/) 提供了集成支持，如果你使用这两个框架，只需通过下面的方式加载 Node SDK 提供的中间件即可。

你可以在你的项目根目录运行 `npm install leanengine@next --save` 来安装 Node SDK。

### Express

```js
var express = require('express');
var AV = require('leanengine');

AV.init({
  appId: process.env.LEANCLOUD_APP_ID || '{{appid}}',
  appKey: process.env.LEANCLOUD_APP_KEY || '{{appkey}}',
  masterKey: process.env.LEANCLOUD_APP_MASTER_KEY || '{{masterkey}}'
});

var app = express();
app.use(AV.express());
app.listen(process.env.LEANCLOUD_APP_PORT);
```

你可以使用 express 的路由定义功能来提供自定义的 HTTP API：

```js
app.get('/', function(req, res) {
  res.render('index', {title: 'Hello world'});
});

app.get('/time', function(req, res) {
  res.json({
    time: new Date()
  });
});

app.get('/todos', function(req, res) {
  new AV.Query('Todo').find().then(function(todos) {
    res.json(todos);
  }).catch(function(err) {
    res.status(500).json({
      error: err.message
    });
  });
});

```

更多最佳实践请参考我们的 [项目模板](https://github.com/leancloud/node-js-getting-started) 和 [云引擎项目示例](leanengine_examples.html)。

### Koa

```js
var koa = require('koa');
var AV = require('leanengine');

AV.init({
  appId: process.env.LEANCLOUD_APP_ID || '{{appid}}',
  appKey: process.env.LEANCLOUD_APP_KEY || '{{appkey}}',
  masterKey: process.env.LEANCLOUD_APP_MASTER_KEY || '{{masterkey}}'
});

var app = koa();
app.use(AV.koa());
app.listen(process.env.LEANCLOUD_APP_PORT);
```

你可以使用 koa 来渲染页面、提供自定义的 HTTP API：

```js
app.use(function *(next) {
  if (this.url === '/') {
    // https://github.com/tj/co-views
    yield coViews('views')('index', {title: 'Hello world'});
  } else {
    yield next;
  }
});

app.use(function *(next) {
  if (this.url === '/time') {
    this.body = {
      time: new Date()
    };
  } else {
    yield next;
  }
});

app.use(function *(next) {
  if (this.url === '/todos') {
    return new AV.Query('Todo').find().then(todos => {
      this.body = todos;
    });
  } else {
    yield next;
  }
});

```

使用 Koa 时建议按照前面 [package.json](#package_json) 一节将 Node.js 的版本设置为 `4.x` 以上。

### 其他 Web 框架

你也可以使用其他的 Web 框架进行开发，但你需要自行去实现 [健康监测](#健康监测) 中提到的逻辑。下面是一个使用 Node.js 内建的 [http](https://nodejs.org/api/http.html) 实现的最简示例，可供参考：

```js
require('http').createServer(function(req, res) {
  if (req.url == '/') {
    res.statusCode = 200;
    res.end();
  } else {
    res.statusCode = 404;
    res.end();
  }
}).listen(process.env.LEANCLOUD_APP_PORT);
```
{% endblock %}

{% block use_leanstorage %}
云引擎中的 Node SDK 是对 [JavaScript 存储 SDK](https://github.com/leancloud/javascript-sdk) 的拓展，增加了服务器端需要的云函数和 Hook 相关支持，在云引擎中你需要用 `leanengine` 这个包来操作 [LeanCloud 的存储服务](leanstorage_guide-js.html) 中的数据，你可以在你的项目根目录运行 `npm install leanengine@next --save` 来安装 Node SDK。

Node SDK 的 [API 文档](https://github.com/leancloud/leanengine-node-sdk/blob/master/API.md) 和 [更新日志](https://github.com/leancloud/leanengine-node-sdk/releases) 都在 GitHub 上。

```js
var AV = require('leanengine');

AV.init({
  appId: process.env.LEANCLOUD_APP_ID || '{{appid}}',
  appKey: process.env.LEANCLOUD_APP_KEY || '{{appkey}}',
  masterKey: process.env.LEANCLOUD_APP_MASTER_KEY || '{{masterkey}}'
});

// 你可以使用 useMasterKey 在云引擎中开启 masterKey 权限，将会跳过 ACL 和其他权限限制。
AV.Cloud.useMasterKey();

// 使用 JavaScript 的 API 查询云存储中的数据。
new AV.Query('Todo').find().then(function(todos) {
  console.log(todos);
}).catch(function(err) {
  console.log(err)
});
```

Node SDK 有过两个大版本：

* `0.x`：最初的版本，对 Node.js 4.x 及以上版本兼容不佳，建议用户参考 [升级到云引擎 Node.js SDK 1.0](leanengine-node-sdk-upgrade-1.html) 来更新
* `1.x`：**推荐使用** 的版本，彻底废弃了全局的 currentUser，依赖的 JavaScript 也升级到了 1.x 分支，支持了 Koa 和 Node.js 4.x 及以上版本。

## 本地运行和调试

在你首次启动应用之前需要先安装依赖：

```sh
npm install
```

然后便可以在项目根目录，用我们的命令行工具来启动本地调试了：

```sh
lean up
```

更多有关命令行工具和本地调试的内容请看 [云引擎命令行工具使用详解](leanengine_cli.html)。

{% endblock %}

{% block get_env %}
```javascript
var NODE_ENV = process.env.NODE_ENV || 'development';
if (NODE_ENV === 'development') {
  // 当前环境为「开发环境」，是由命令行工具启动的
} else if(NODE_ENV == 'production') {
  // 当前环境为「生产环境」，是线上正式运行的环境
} else {
  // 当前环境为「预备环境」
}
```
{% endblock %}

{% block cookie_session %}
云引擎提供了一个 `AV.Cloud.CookieSession` 中间件，用 Cookie 来维护用户（`AV.User`）的登录状态，要使用这个中间件可以在 `app.js` 中添加下列代码：

```javascript
app.use(AV.Cloud.CookieSession({ secret: 'my secret', maxAge: 3600000, fetchUser: true }));
```

Koa 需要添加一个 `framework: 'koa'` 的参数：

```javascript
app.use(AV.Cloud.CookieSession({ framework: 'koa', secret: 'my secret', maxAge: 3600000, fetchUser: true }));
```

你需要传入一个 secret 用于签名 Cookie（必须提供），这个中间件会将 `AV.User` 的登录状态信息记录到 Cookie 中，用户下次访问时自动检查用户是否已经登录，如果已经登录，可以通过 `req.currentUser` 获取当前登录用户。

`AV.Cloud.CookieSession` 支持的选项包括：

* **fetchUser**：**是否自动 fetch 当前登录的 AV.User 对象。默认为 false。**
  如果设置为 true，每个 HTTP 请求都将发起一次 LeanCloud API 调用来 fetch 用户对象。如果设置为 false，默认只可以访问 `req.currentUser` 的 `id`（`_User` 表记录的 ObjectId）和 `sessionToken` 属性，你可以在需要时再手动 fetch 整个用户。
* **name**：Cookie 的名字，默认为 `avos.sess`。
* **maxAge**：设置 Cookie 的过期时间。

在 Node SDK 1.x 之后我们不再允许通过 `AV.User.current()` 获取登录用户的信息（详见 [升级到云引擎 Node.js SDK 1.0](leanengine-node-sdk-upgrade-1.html#废弃_currentUser)），而是需要你：

* 在云引擎方法中，通过 `request.currentUser` 获取用户信息。
* 在网站托管中，通过 `request.currentUser` 获取用户信息。
* 在后续的方法调用显示传递 user 对象。

你可以这样简单地实现一个具有登录功能的站点：

```javascript
// 处理登录请求（可能来自登录界面中的表单）
app.post('/login', function(req, res) {
  AV.User.logIn(req.body.username, req.body.password).then(function(user) {
    res.saveCurrentUser(user); // 保存当前用户到 Cookie
    res.redirect('/profile'); // 跳转到个人资料页面
  }, function(error) {
    //登录失败，跳转到登录页面
    res.redirect('/login');
  });
})

// 查看个人资料
app.get('/profile', function(req, res) {
  // 判断用户是否已经登录
  if (req.currentUser) {
    // 如果已经登录，发送当前登录用户信息。
    res.send(req.currentUser);
  } else {
    // 没有登录，跳转到登录页面。
    res.redirect('/login');
  }
});

// 登出账号
app.get('/logout', function(req, res) {
  req.currentUser.logOut();
  res.clearCurrentUser(); // 从 Cookie 中删除用户
  res.redirect('/profile');
});
```
{% endblock %}

{% block http_client %}

推荐使用 [request](https://www.npmjs.com/package/request) 这个第三方模块来完成 HTTP 请求。

安装 request:

```sh
npm install request --save
```

代码示例：

```javascript
var request = require('request');

request({
  method: 'POST',
  url: 'http://www.example.com/create_post',
  json: {
    title: 'Vote for Pedro',
    body: 'If you vote for Pedro, your wildest dreams will come true'
  }
}, function(err, res, body) {
  if (err) {
    console.error('Request failed with response code ' + res.statusCode);
  } else {
    console.log(body);
  }
});
```
{% endblock %}

{% block code_get_client_ip_address %}
```js
app.get('/', function(req, res) {
  console.log(req.headers['x-real-ip']);
  res.send(req.headers['x-real-ip']);
});
```
{% endblock %}

{% block upload_file_special_middleware %}
然后配置应用使用 [multiparty](https://www.npmjs.com/package/multiparty) 中间件：

```javascript
var multiparty = require('multiparty');
```
{% endblock %}

{% block code_upload_file_sdk_function %}
```javascript
var fs = require('fs');
app.post('/upload', function(req, res){
  var form = new multiparty.Form();
  form.parse(req, function(err, fields, files) {
    var iconFile = files.iconImage[0];
    if(iconFile.size !== 0){
      fs.readFile(iconFile.path, function(err, data){
        if(err) {
          return res.send('读取文件失败');
        }
        var theFile = new AV.File(iconFile.originalFilename, data);
        theFile.save().then(function(theFile){
          res.send('上传成功！');
        }).catch(console.error);
      });
    } else {
      res.send('请选择一个文件。');
    }
  });
});
```
{% endblock %}

{% block custom_session %}
有时候你需要将一些自己需要的属性保存在 session 中，你可以增加通用的 `cookie-session` 组件，详情可以参考 [express.js &middot; cookie-session](https://github.com/expressjs/cookie-session)。该组件和 `AV.Cloud.CookieSession` 组件可以并存。

<div class="callout callout-info">express 框架的 `express.session.MemoryStore` 在云引擎中是无法正常工作的，因为云引擎是多主机、多进程运行，因此内存型 session 是无法共享的，建议用 [express.js &middot; cookie-session 中间件](https://github.com/expressjs/cookie-session)。</div>
{% endblock %}

{% block leancache %}
首先添加相关依赖到 `package.json` 中：

```json
"dependencies": {
  ...
  "redis": "2.2.x",
  ...
}
```

然后可以使用下列代码获取 Redis 连接：

```js
var client = require('redis').createClient(process.env['REDIS_URL_<实例名称>']);
// 建议增加 client 的 on error 事件处理，否则可能因为网络波动或 redis server 主从切换等原因造成短暂不可用导致应用进程退出。
client.on('error', function(err) {
  return console.error('redis err: %s', err);
});
```
{% endblock %}

{% block https_redirect %}
Express:

```javascript
app.enable('trust proxy');
app.use(AV.Cloud.HttpsRedirect());
```

Koa:

```javascript
app.proxy = true;
app.use(AV.Cloud.HttpsRedirect({framework: 'koa'}));
```
{% endblock %}

{% block extra_examples %}
### 多进程运行

因为 Node.js 本身的单线程模型，无法充分利用多个 CPU 核心，所以如果你使用了 2CPU 或以上的实例，需要自行使用 Node.js 的 [cluster](https://nodejs.org/api/cluster.html) 配置多进程运行，创建一个 `server-cluster.js`：

```javascript
var cluster = require('cluster');

// 取决于你的实例的可用 CPU 数量
var workers = 2;

if (cluster.isMaster) {
  for (var i = 0; i < workers; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log('worker %s died, restarting...', worker.process.pid);
    cluster.fork();
  });
} else {
  require('./server.js')
}
```

然后在 `package.json` 中将 `scripts.start` 改为 `node server-cluster.js` 即可：

```json
"scripts": {
  "start": "node server-cluster.js"
}
```

<div class="callout callout-info">多进程运行要求你的程序中没有在内存中维护全局状态（例如锁），建议在首次切换到多进程或多实例运行时进行充分的测试。</div>

{% endblock %}

{% block code_calling_custom_variables %}
```javascript
// 在云引擎 Node.js 环境中使用自定义的环境变量
var MY_CUSTOM_VARIABLE = process.env.MY_CUSTOM_VARIABLE;
console.log(MY_CUSTOM_VARIABLE);
```
{% endblock %}

{% block loggerExample %}
```javascript
console.log('hello');
console.error('some error!');
```
{% endblock %}

{% block loggerExtraDescription %}
你可以通过设置一个 `DEBUG=leancloud:request` 的环境变量来打印由 LeanCloud SDK 发出的网络请求。在本地调试时你可以通过这样的命令启动程序：

```sh
env DEBUG=leancloud:request lean up
```

当有对 LeanCloud 的调用时，你可以看到类似这样的日志：

```
leancloud:request request(0) +0ms GET https://{{host}}/1.1/classes/Todo?&where=%7B%7D&order=-createdAt { where: '{}', order: '-createdAt' }
leancloud:request response(0) +220ms 200 {"results":[{"content":"1","createdAt":"2016-08-09T06:18:13.028Z","updatedAt":"2016-08-09T06:18:13.028Z","objectId":"57a975a55bbb5000643fb690"}]}
```

我们不建议在线上生产环境开启这个日志，否则将会打印大量的日志。

{% endblock %}

{% block section_timezone %}
需要注意 JavaScript 中 Date 类型的不同方法，一部分会返回 UTC 时间、一部分会返回当地时间（在中国区是北京时间）：

函数|时区|结果
---|---|---
`toISOString`|UTC 时间|2015-04-09T03:35:09.678Z
`toJSON`（JSON 序列化时）|UTC 时间|2015-04-09T03:35:09.678Z
`toUTCString`|UTC 时间|Thu, 09 Apr 2015 03:35:09 GMT
`getHours`|UTC 时间|3
`toString`（`console.log` 打印时）|当地时间|Thu Apr 09 2015 03:35:09 GMT+0000 (UTC)
`toLocaleString`|当地时间|Thu Apr 09 2015 03:35:09 GMT+0000 (UTC)

提醒大家需要在向用户展示时注意区分，否则就会出现时间「偏差八小时」的现象。
{% endblock %}
