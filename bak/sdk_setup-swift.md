{% extends "./sdk_setup.tmpl" %}
{% set platform_name = "Swift" %}
{% set command_install_cocoapods = "$ sudo gem install cocoapods" %}
{% set code_import_sdk_core = "import LeanCloud" %}

{% block libs_tool_automatic %}
[CocoaPods](http://www.cocoapods.org/) 是开发 macOS 和 iOS 应用程序的一个第三方库的依赖管理工具，通过它可以定义自己的依赖关系（称作 pods），并且随着时间的推移，它会让整个开发环境中对第三方库的版本管理变得非常方便。具体可以参考 [CocoaPods 安装和使用教程](http://code4app.com/article/cocoapods-install-usage)。

首先确保开发环境中已经安装了 Ruby（一般安装了 Xcode，Ruby 会被自动安装上），如果没有安装请执行以下命令行：

```sh
{{command_install_cocoapods}}
```

如果遇到网络问题无法从国外主站上直接下载，我们推荐一个国内的镜像：[RubyGems 镜像](http://ruby.taobao.org/)，具体操作步骤如下：

```sh
$ gem sources --remove https://rubygems.org/
$ gem sources -a https://ruby.taobao.org/
# 请确保下列命令的输出只有 ruby.taobao.org
$ gem sources -l
*** CURRENT SOURCES ***
https://ruby.taobao.org
```

然后再安装 CocoaPods：

```sh
{{command_install_cocoapods}}
```

在项目根目录下创建一个名为 `Podfile` 的文件（无扩展名），并添加以下内容：

```ruby
use_frameworks! # LeanCloud Swift SDK can only be integrated as framework.

target '你的项目名称' do
	pod 'LeanCloud'
end
```

执行命令 `pod install --verbose` 安装 SDK。

{% endblock %}

{% block text_manual_setup %}{% endblock %}
{% block import_sdk %}
{% endblock %}

{% block init_with_app_keys %}

打开 `AppDelegate.m` 文件，添加下列导入语句到头部：

```swift
import LeanCloud
```

然后粘贴下列代码到 `application:didFinishLaunchingWithOptions` 函数内：

```swift
// applicationId 即 App Id，applicationKey 是 App Key
LeanCloud.initialize(applicationID: "{{appid}}", applicationKey: "{{appkey}}")
```
{% endblock %}

{% block sdk_switching_node %}

```swift
// 如果使用美国站点，请加上这行代码，并且写在初始化前面
LeanCloud.setServiceRegion(.US)

// applicationId 即 App Id，applicationKey 是 App Key
LeanCloud.initialize(applicationID: "{{appid}}", applicationKey: "{{appkey}}")
```
{% endblock %}

{% block save_a_hello_world %}

```swift
let post = LCObject(className: "TestObject")

post.set("words", value: "Hello World!")

post.save()
```

然后，点击 `Run` 运行调试，真机和虚拟机均可。
{% endblock %}

{% block permission_access_network_config %}{% endblock %}

{% block platform_specific_faq %}
{% endblock %}
