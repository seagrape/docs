{% extends "./realtime_guide.tmpl" %}

{% set platform_name = 'Android' %}
{% set segment_code = 'android' %}
{% set sdk_name = platform_name + " SDK" %}

{% block setup_init %}
请参考详细的 [{{sdk_name}} 安装指南](sdk_setup-android.html)。
{% endblock %}

{% block demo %}
* [LeanMessage](https://github.com/leancloud/LeanMessage-Demo)（推荐）
* [LeanChat](https://github.com/leancloud/leanchat-android)
{% endblock %}

{% block oneOnOneChat_sent %}
<div class="callout callout-info">注意：**启用实时通信一定要正确配置** `AndroidManifest.xml`，请仔细阅读 [Android SDK 初始化配置](sdk_setup-android.html#初始化)。</div>

```
  public void sendMessageToJerryFromTom() {
    // Tom 用自己的名字作为clientId，获取AVIMClient对象实例
    AVIMClient tom = AVIMClient.getInstance("Tom");
    // 与服务器连接
    tom.open(new AVIMClientCallback() {
      @Override
      public void done(AVIMClient client, AVIMException e) {
        if (e == null) {
          // 创建与Jerry之间的对话
          client.createConversation(Arrays.asList("Jerry"), "Tom & Jerry", null,
              new AVIMConversationCreatedCallback() {

                @Override
                public void done(AVIMConversation conversation, AVIMException e) {
                  if (e == null) {
                    AVIMTextMessage msg = new AVIMTextMessage();
                    msg.setText("耗子，起床！");
                    // 发送消息
                    conversation.sendMessage(msg, new AVIMConversationCallback() {

                      @Override
                      public void done(AVIMException e) {
                        if (e == null) {
                          Log.d("Tom & Jerry", "发送成功！");
                        }
                      }
                    });
                  }
                }
              });
        }
      }
    });
  }
```
{% endblock %}

{% block oneOnOneChat_received %}

```
public class MyApplication extends Application{
 public static class CustomMessageHandler extends AVIMMessageHandler{
   //接收到消息后的处理逻辑 
   @Override
   public void onMessage(AVIMMessage message,AVIMConversation conversation,AVIMClient client){
     if(message instanceof AVIMTextMessage){
       Log.d("Tom & Jerry",((AVIMTextMessage)message).getText());
     }
   }
   
   public void onMessageReceipt(AVIMMessage message,AVIMConversation conversation,AVIMClient client){
   
   }
 }	
 public void onCreate(){
   ...
   AVOSCloud.initialize(this,"{{appid}}","{{appkey}}");   
   //注册默认的消息处理逻辑
   AVIMMessageManager.registerDefaultMessageHandler(new CustomMessageHandler());
   ...
 }
...
public void jerryReceiveMsgFromTom(){
  //Jerry登录
  AVIMClient jerry = AVIMClient.getInstance("Jerry");
  jerry.open(new AVIMClientCallback(){
  
    @Override
    public void done(AVIMClient client,AVIMException e){
    	if(e==null){
    	 ...//登录成功后的逻辑
    	}
    }
  });
}
}
```
{% endblock %}

{% block androidMessageHandler %}
#### MessageHandler 的处理逻辑

在 Android SDK 中接收消息的 AVIMMessageHandler 在 AVIMMessageManager 中进行注册时有两个不同的方法： `registerDefaultMessageHandler` 和 `registerMessageHandler`。

当客户端收到一条消息的时候，会优先根据消息类型通知当前所有注册的对应类型的普通的 `messageHandler`,如果发现当前没有任何注册的普通的 `messageHandler`，才会去通知 `defaultMessageHandler`。

在 `AVIMMessageManager` 中多次注册 `defaultMessageHandler` ，只有最后一次调用的才是有效的；而通过 `registerMessageHandler` 注册的 `AVIMMessageHandler`，则是可以同存的。

通过在 UI 组件（比如 Activity）的 `onResume` 方法中间去调用 `registerMessageHandler`,而在 `onPaused` 方法中间调用 `unregisterMessageHandler` 的组合，让对应的 `messageHandler` 处理当前页面的处理逻辑；而当没有页面时，则通过 defaultMessageHandler 去发送 `Notification`。

{% endblock %}

{% block oneOnOneChat_received_steps %}
接收消息之前，需要先定义好自己的 `AVIMMessageHandler` 来响应新消息到达的通知，如上例中的 `CustomMessageHandler`。然后通过 `AVIMMessageManager.registerMessageHandler()` 函数来实现绑定。`AVIMMessageManager` 类中还有一个方法 `registerDefaultMessageHandler()` 则用来指定全局默认的消息处理 handler。

> 注意：`AVIMMessageManager.registerDefaultMessageHandler()` 一定要在 `AVIMClient.open()` 之前调用，否则可能导致服务器发回来的部分消息丢失。

然后通过 `AVIMMessageHandler.onMessage()` 函数来接收消息。
{% endblock %}

{% block groupChat_sent %}
```
  public void sendMessageToJerryFromTom() {
    // Tom 用自己的名字作为clientId，获取AVIMClient对象实例
    AVIMClient tom = AVIMClient.getInstance("Tom");
    // 与服务器连接
    tom.open(new AVIMClientCallback() {
      @Override
      public void done(AVIMClient client, AVIMException e) {
        if (e == null) {
          // 创建与 Jerry，Bob,Harry,William 之间的对话
          client.createConversation(Arrays.asList("Jerry","Bob","Harry","William"), "Tom & Jerry & friedns", null,
              new AVIMConversationCreatedCallback() {

                @Override
                public void done(AVIMConversation conversation, AVIMException e) {
                  if (e == null) {
                    AVIMTextMessage msg = new AVIMTextMessage();
                    msg.setText("你们在哪儿？");
                    // 发送消息
                    conversation.sendMessage(msg, new AVIMConversationCallback() {

                      @Override
                      public void done(AVIMException e) {
                        if (e == null) {
                          Log.d("Tom & Jerry", "发送成功！");
                        }
                      }
                    });
                  }
                }
              });
        }
      }
    });
  }
```
{% endblock %}

{% block groupChat_received %}
```
public class MyApplication extends Application{
  public void onCreate(){
   ...
   AVOSCloud.initialize(this,"{{appid}}","{{appkey}}");
   //这里指定只处理AVIMTextMessage类型的消息
   AVIMMessageManager.registerMessageHandler(AVIMTextMessage.class,new CustomMessageHanlder());
  }
}

- CustomMessageHandler.java
public class CustomMessageHandler<AVIMTextMessage> implements AVIMTypedMessageHandler{
 
  @Override
  public void onMessage(AVIMTextMessage msg,AVIMConversation conv,AVIMClient client){
    Log.d("Tom & Jerry",msg.getText();)//你们在哪儿?
    // 收到消息之后一般的做法是做 UI 展现，示例代码在此处做消息回复，仅为了演示收到消息之后的操作，仅供参考。
    AVIMTextMessage reply = new AVIMTextMessage();
    reply.setText("Tom，我在 Jerry 家，你跟 Harry 什么时候过来？还有 William 和你在一起么？");
    conv.sendMessage(reply,new AVIMConversationCallback(){
  	   public void done(AVIMException e){
  	     if(e==null){
  	     //回复成功!
  	     }
  	   }
  	 });
  }
  
public void onMessageReceipt(AVIMTextMessage msg,AVIMConversation conv,AVIMClient client){
  
}
}


- SomeActivity.java
public void loginAsBob(){
	AVIMClient bob = AVIMClient.getInstance("Bob");
	//Bob登录
	bob.open(new AVIMClientCallback(){
	  public void done(AVIMClient client,AVIMException e){
	  	if(e==null){
	  		//登录成功
	  	}
	  }
	});
}
```
{% endblock %}

{% block imageMessage_local_sent %}

```
public void sendImage(String filePath){
  AVIMClient tom = AVIMClient.getInstance("Tom");

  tom.open(new AVIMClientCallback(){
  
    @Override
    public void done(AVIMClient client,AVIMException e){
      if(e==null){
      //登录成功
      // 创建对话，默认创建者是在包含在成员列表中的
      client.createConversation(Arrays.asList("Jerry"),new AVIMConversationCreatedCallback(){
      
        @Override
        public void done(AVIMConversation conv,AVIMException e){
          if(e==null){
            AVIMImageMessage picture = new AVIMImageMessage(filePath);
            picture.setText("发自我的小米");
            Map<String,Object> attributes = new HashMap<String,Object>();
            attributes.put("location","旧金山");
            picture.setAttribute(attributes);
            conv.sendMessage(picture,new AVIMConversationCallback(){
              
              @Override
              public void done(AVIMException e){
                if(e==null){
                //发送成功！
                }
              }
            });
          }
        }
      });
      }
    }
  });
}
```
{% endblock %}

{% block imageMessage_url_sent %}
```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback() {
      @Override
      public void done(AVIMClient client, AVIMException e) {
        if (e == null) {
          // 创建名为“猫和老鼠”的对话
          client.createConversation(Arrays.asList("Jerry"), "猫和老鼠", null,
              new AVIMConversationCreatedCallback() {
                @Override
                public void done(AVIMConversation conv, AVIMException e) {
                  if (e == null) {
                    AVFile file =new AVFile("萌妹子","http://pic2.zhimg.com/6c10e6053c739ed0ce676a0aff15cf1c.gif", null);
                    AVIMImageMessage m = new AVIMImageMessage(file);
                    m.setText("萌妹子一枚");
                    // 创建一条图片消息
                    conv.sendMessage(m, new AVIMConversationCallback() {
                      @Override
                      public void done(AVIMException e) {
                        if (e == null) {
                          // 发送成功
                        }
                      }
                    });
                  }
                }
              });
        }
      }
});
```
{% endblock %}

{% block imageMessage_received_intro %}{% endblock %}

{% block imageMessage_received %}

```
//注册消息处理逻辑
AVIMMessageManager.registerMessageHandler(AVIMImageMessage.class,
        new AVIMTypedMessageHandler<AVIMImageMessage>() {

          @Override
          public void onMessage(AVIMImageMessage msg, AVIMConversation conv, AVIMClient client) {
          	//只处理 Jerry 这个客户端的消息
          	//并且来自 conversationId 为 55117292e4b065f7ee9edd29 的conversation 的消息	
            if ("Jerry".equals(client.getClientId()) && "55117292e4b065f7ee9edd29".equals(conv.getConversationId())) {
                String fromClientId = msg.getFrom();
                String messageId = msg.getMessageId();
                String url = msg.getFileUrl();
                Map<String, Object> metaData = msg.getFileMetaData();
                if (metaData.containsKey("size")) {
                  int size = (Integer) metaData.get("size");
                }
                if (metaData.containsKey("width")) {
                  int width = (Integer) metaData.get("width");
                }
                if (metaData.containsKey("height")) {
                  int height = (Integer) metaData.get("height");
                }
                if (metaData.containsKey("format")) {
                  String format = (String) metaData.get("format");
                }
            }
          }
        });
        
    AVIMClient jerry = AVIMClient.getInstance("Jerry");
    jerry.open(new AVIMClientCallback() {
      @Override
      public void done(AVIMClient client, AVIMException e) {
        if (e == null) {

        }
      }
    });
```
{% endblock %}

{% block audioMessage_local_sent %}
```
 AVIMClient tom = AVIMClient.getInstance("Tom");
    tom.open(new AVIMClientCallback() {
      @Override
      public void done(AVIMClient client, AVIMException e) {
        if (e == null) {
          // 创建名为“猫和老鼠”的对话
          client.createConversation(Arrays.asList("Jerry"), "猫和老鼠", null,
              new AVIMConversationCreatedCallback() {
                @Override
                public void done(AVIMConversation conv, AVIMException e) {
                  if (e == null) {
                    AVFile file = AVFile.withAbsoluteLocalPath("忐忑.mp3",localFilePath);
                    AVIMAudioMessage m = new AVIMAudioMessage(file);
                    m.setText("听听人类的神曲~");
                    // 创建一条音频消息
                    conv.sendMessage(m, new AVIMConversationCallback() {
                      @Override
                      public void done(AVIMException e) {
                        if (e == null) {
                          // 发送成功
                        }
                      }
                    });
                  }
                }
              });
        }
      }
    });
``` 
{% endblock %}

{% block audioMessage_url_sent %}
```
AVIMClient tom = AVIMClient.getInstance("Tom");
    tom.open(new AVIMClientCallback() {
      @Override
      public void done(AVIMClient client, AVIMException e) {
        if (e == null) {
          // 创建名为「猫和老鼠」的对话
          client.createConversation(Arrays.asList("Jerry"), "猫和老鼠", null,
              new AVIMConversationCreatedCallback() {
                @Override
                public void done(AVIMConversation conv, AVIMException e) {
                  if (e == null) {
                    AVFile file = new AVFile("music", "http://ac-lhzo7z96.clouddn.com/1427444393952", null);
                    AVIMAudioMessage m = new AVIMAudioMessage(file);
                    // 创建一条音频消息
                    conv.sendMessage(m, new AVIMConversationCallback() {
                      @Override
                      public void done(AVIMException e) {
                        if (e == null) {
                          // 发送成功
                        }
                      }
                    });
                  }
                }
              });
        }
      }
    });
```
{% endblock %}

{% block audioMessage_received_intro %}
```
AVIMMessageManager.registerMessageHandler(AVIMAudioMessage.class,
        new AVIMTypedMessageHandler<AVIMAudioMessage>() {

          @Override
          public void onMessage(AVIMAudioMessage msg, AVIMConversation conv, AVIMClient client) {
          	//只处理 Jerry 这个客户端的消息
          	//并且来自 conversationId 为 55117292e4b065f7ee9edd29 的conversation 的消息	
            if ("Jerry".equals(client.getClientId()) && "55117292e4b065f7ee9edd29".equals(conv.getConversationId())) {
                String fromClientId = msg.getFrom();
                String messageId = msg.getMessageId();
                String url = msg.getFileUrl();
                Map<String, Object> metaData = msg.getFileMetaData();
                if (metaData.containsKey("size")) {
                  int size = (Integer) metaData.get("size");
                }
                if (metaData.containsKey("format")) {
                  String format = (String) metaData.get("format");
                }
            }
          }
        });
        
    AVIMClient jerry = AVIMClient.getInstance("Jerry");
    jerry.open(new AVIMClientCallback() {
      @Override
      public void done(AVIMClient client, AVIMException e) {
        if (e == null) {

        }
      }
    });
```
{% endblock %}

{% block videoMessage_local_sent %}
```
  AVIMClient tom = AVIMClient.getInstance("Tom");
  tom.open(new AVIMClientCallback() {
    @Override
    public void done(AVIMClient client, AVIMException e) {
      if (e == null) {
        // 创建名为“猫和老鼠”的对话
        client.createConversation(Arrays.asList("Jerry"), "猫和老鼠", null,
                new AVIMConversationCreatedCallback() {
                  @Override
                  public void done(AVIMConversation conv, AVIMException e) {
                    if (e == null) {
                      AVFile file = AVFile.withAbsoluteLocalPath("bbc_奶酪.mp4", localFilePath);
                      AVIMVideoMessage m = new AVIMVideoMessage(file);
                      // 创建一条视频消息
                      conv.sendMessage(m, new AVIMConversationCallback() {
                        @Override
                        public void done(AVIMException e) {
                          if (e == null) {
                            // 发送成功
                          }
                        }
                      });
                    }
                  }
                });
      }
    }
  });
```
{% endblock %}

{% block videoMessage_url_sent %}
```
 AVIMClient tom = AVIMClient.getInstance("Tom");
    tom.open(new AVIMClientCallback() {
      @Override
      public void done(AVIMClient client, AVIMException e) {
        if (e == null) {
          // 创建名为「猫和老鼠」的对话
          client.createConversation(Arrays.asList("Jerry"), "猫和老鼠", null,
              new AVIMConversationCreatedCallback() {
                @Override
                public void done(AVIMConversation conv, AVIMException e) {
                  if (e == null) {
                    AVFile file =
                        new AVFile("video", "http://ac-lhzo7z96.clouddn.com/1427267336319", null);
                    AVIMVideoMessage m = new AVIMVideoMessage(file);
                    // 创建一条视频消息
                    conv.sendMessage(m, new AVIMConversationCallback() {
                      @Override
                      public void done(AVIMException e) {
                        if (e == null) {
                          // 发送成功
                        }
                      }
                    });
                  }
                }
              });
        }
      }
    });
```
{% endblock %}

{% block videoMessage_received_intro %}
视频消息的接收与图像消息一样，它的元数据都可以通过 `getFileMetaData()` 来获取。
{% endblock %}

{% block fileMessage_sent %}
```

    AVIMClient tom = AVIMClient.getInstance("Tom");
    tom.open(new AVIMClientCallback() {
      @Override
      public void done(AVIMClient client, AVIMException e) {
        if (e == null) {
          // 创建名为“猫和老鼠”的对话
          client.createConversation(Arrays.asList("Jerry"), "猫和老鼠", null,
              new AVIMConversationCreatedCallback() {
                @Override
                public void done(AVIMConversation conv, AVIMException e) {
                  if (e == null) {
                    AVFile file =
                        AVFile.withAbsoluteLocalPath("leancloud.doc", localFilePath);
                    AVIMFileMessage m = new AVIMFileMessage(file);
                    // 创建一条文件消息
                    conv.sendMessage(m, new AVIMConversationCallback() {
                      @Override
                      public void done(AVIMException e) {
                        if (e == null) {
                          // 发送成功
                        }
                      }
                    });
                  }
                }
              });
        }
      }
    });

```
{% endblock %}

{% block fileMessage_received_intro %}
文件消息的接收与图像消息一样，它的元数据获取都可以通过 `getFileMetaData()` 来获取。
{% endblock %}

{% block locationMessage_new %}
```
      AVIMLocationMessage m = new AVIMLocationMessage();
      m.setLocation(new AVGeoPoint(45.0,34.0));
 ```
{% endblock %}

{% block locationMessage_sent %}
```
final AVIMLocationMessage locationMessage=new AVIMLocationMessage();
// 开发者更可以通过具体的设备的 API 去获取设备的地理位置，此处仅设置了 2 个经纬度常量仅做演示
locationMessage.setLocation(new AVGeoPoint(138.12454,52.56461));
locationMessage.setText("新开的蛋糕店！耗子咱们有福了…");
conversation.sendMessage(locationMessage, new AVIMConversationCallback() {
    @Override
    public void done(AVIMException e) {
        if (null != e) {
          e.printStackTrace();
        } else {
          // 发送成功
        }
    }
});
```
{% endblock %}

{% block locationMessage_received_intro %}

地址消息的接收与图像消息一样，它的地址信息可以通过 `getLocation` 方法来获取

{% endblock %}

{% block typedMessage_received %}
### 接收富媒体消息

实时通信 SDK 内部封装了对富媒体消息的支持，所有富媒体消息都是从 AVIMTypedMessage 派生出来的。发送的时候可以直接调用 `conversation.sendMessage()` 函数。在接收端，我们也专门增加了一类回调接口 AVIMTypedMessageHandler，其定义为：

```
public class AVIMTypedMessageHandler<T extends AVIMTypedMessage> extends MessageHandler<T> {

  @Override
  public void onMessage(T message, AVIMConversation conversation, AVIMClient client);

  @Override
  public void onMessageReceipt(T message, AVIMConversation conversation, AVIMClient client);
}
```

开发者可以编写自己的消息处理 handler，然后调用 `AVIMMessageManager.registerMessageHandler()` 函数来注册目标 handler。

接收端对于富媒体消息的通知处理的示例代码如下：

```
class MsgHandler extends AVIMTypedMessageHandler<AVIMTypedMessage> {

  @Override
  public void onMessage(AVIMTypedMessage message, AVIMConversation conversation, AVIMClient client) {
    // 请按自己需求改写
    switch(message.getMessageType()) {
    case AVIMReservedMessageType.TextMessageType:
      AVIMTextMessage textMsg = (AVIMTextMessage)message;
      Logger.d("收到文本消息:" + textMsg.getText() + ", msgId:" + textMsg.getMessageId());
      break;

    case AVIMReservedMessageType.FileMessageType:
      AVIMFileMessage fileMsg = (AVIMFileMessage)message;
      Logger.id("收到文件消息。msgId=" + fileMsg.getMessageId() + ", url=" + fileMsg.getFileUrl() + ", size=" + fileMsg.getSize());
      break;

    case AVIMReservedMessageType.ImageMessageType:
      AVIMImageMessage imageMsg = (AVIMImageMessage)message;
      Logger.id("收到图片消息。msgId=" + imageMsg.getMessageId() + ", url=" + imageMsg.getFileUrl() + ", width=" + imageMsg.getWidth() + ", height=" + imageMsg.getHeight());
      break;

    case AVIMReservedMessageType.AudioMessageType:
      AVIMAudioMessage audioMsg = (AVIMAudioMessage)message;
      Logger.id("收到音频消息。msgId=" + audioMsg.getMessageId() + ", url=" + audioMsg.getFileUrl() + ", duration=" + audioMsg.getDuration());
      break;

    case AVIMReservedMessageType.VideoMessageType:
      AVIMVideoMessage videoMsg = (AVIMAudioMessage)message;
      Logger.id("收到视频消息。msgId=" + videoMsg.getMessageId() + ", url=" + videoMsg.getFileUrl() + ", duration=" + videoMsg.getDuration());
      break;

    case AVIMReservedMessageType.LocationMessageType:
      AVIMLocationMessage locMsg = (AVIMLocationMessage)message;
      Logger.id("收到位置消息。msgId=" + locMsg.getMessageId() + ", latitude=" + locMsg.getLocation().getLatitude() + ", longitude=" + locMsg.getLocation().getLongitude());
      break;
    }
  }

  @Override
  public void onMessageReceipt(AVIMTypedMessage message, AVIMConversation conversation, AVIMClient client) {
    // 请加入你自己需要的逻辑...
  }
}

MsgHandler msgHandler = new MsgHandler();
AVIMMessageManager.registerMessageHandler(AVIMTypedMessage.class, msgHandler);
```

SDK 内部在接收消息时的处理逻辑是这样的：

* 当收到新消息时，实时通信 SDK 会先解析消息的类型，然后找到开发者为这一类型所注册的处理响应 handler chain，再逐一调用这些 handler 的 onMessage 函数
* 如果没有找到专门处理这一类型消息的 handler，就会转交给 defaultHandler 处理。

这样一来，在开发者为 `AVIMTypedMessage`（及其子类） 指定了专门的 handler，也指定了全局的 defaultHandler 了的时候，如果发送端发送的是通用的 AVIMMessage 消息，那么接受端就是 `AVIMMessageManager.registerDefaultMessageHandler()` 中指定的 handler 被调用；如果发送的是 AVIMTypedMessage（及其子类）的消息，那么接受端就是 `AVIMMessageManager.registerMessageHandler()` 中指定的 handler 被调用。
{% endblock %}

{% block transientMessage_sent %}

```
//自定义的消息类型，用于发送和接收所有的用户操作消息
- AVIMOperationMessage.java
 
//指定type类型，可以根据实际换成其他正整数
@AVIMMessageType(type = 1)
public class AVIMOperationMessage extends AVIMTypedMessage {

  @AVIMMessageField(name = "op")
  String op;

  public String getOp() {
    return op;
  }

  public void setOp(String op) {
    this.op = op;
  }
}

- CustomApplication.java
public CustomApplication extends Application {
   ...
   //注册自定义的消息类型
   AVIMMessageManager.registerAVIMMessageType(AVIMOperationMessage.class);
   ...
}

- SomeActivity.java
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback() {
      @Override
      public void done(AVIMClient client, AVIMException e) {
        if (e == null) {
          // 登录成功
          AVIMConversation conv = client.getConversation("551260efe4b01608686c3e0f");
          AVIMOperationMessage msg = new AVIMOperationMessage();
          msg.setOp("keyboard inputing");
          // AVIMConversation.TRANSIENT_MESSAGE_FLAG 表示该条消息为暂态消息
          // 
          conv.sendMessage(msg, AVIMConversation.TRANSIENT_MESSAGE_FLAG,
              new AVIMConversationCallback() {
                @Override
                public void done(AVIMException e) {
                  if (e == null) {
                    // 发送成功
                  }
                }
              });
        }
      }
    });
```
{% endblock %}

{% block transientMessage_received %}

```
//自定义的消息类型，用于发送和接收所有的用户操作消息
- AVIMOperationMessage.java
 
//指定type类型，可以根据实际换成其他正整数
@AVIMMessageType(type = 1)
public class AVIMOperationMessage extends AVIMTypedMessage {

  @AVIMMessageField(name = "op")
  String op;

  public String getOp() {
    return op;
  }

  public void setOp(String op) {
    this.op = op;
  }
}

- CustomApplication.java
public CustomApplication extends Application {
   ...
   //注册自定义的消息类型
   AVIMMessageManager.registerAVIMMessageType(AVIMOperationMessage.class);
   ...
}

- SomeActivity.java
final String USER_OPERATION = "% is %";
// 设置消息接收的 Handler，接收到消息之后的将执行具体的操作
AVIMMessageManager.registerMessageHandler(AVIMOperationMessage.class,
    new AVIMTypedMessageHandler<AVIMOperationMessage>() {
        @Override
        public void onMessage(AVIMOperationMessage msg, AVIMConversation conv, AVIMClient client) {
            if ("Jerry".equals(client.getClientId())
                && "551260efe4b01608686c3e0f".equals(conv.getConversationId())) {
              String opeartion = String.format(USER_OPERATION, msg.getFrom(), msg.getOp());
              System.out.println(opeartion);
            }
        }
});
        
// 登录操作，建立和服务端的连接，开始接收消息
AVIMClient jerry = AVIMClient.getInstance("Jerry");
jerry.open(new AVIMClientCallback() {
    @Override
    public void done(AVIMClient client, AVIMException e) {
        if (e == null) {
          // 登录成功
        }
      }
});
```
{% endblock %}

{% block offlineMessage_android %}>**Android 聊天服务是和后台的推送服务共享连接的，所以只要有网络就永远在线，不需要专门做推送。**消息达到后，你可以根据用户的设置来判断是否需要弹出通知。网络断开时，我们为每个对话保存 20 条离线消息。{% endblock %}

{% block message_sent_ack %}

```
AVIMMessageHandler handler = new AVIMMessageHandler(){

    public void onMessageReceipt(AVIMMessage message, AVIMConversation conversation, AVIMClient client) {
     //此处就是对方收到消息以后的回调
	  Log.i("Tom & Jerry","msg received");
  }
}

//注册对应的handler
AVIMMessageManager.registerMessageHandler(AVIMMessage.class,handler);

//发送消息

AVIMClient jerry = AVIMClient.getInstance("Jerry");
AVIMConversation conv = jerry.getConversation("551260efe4b01608686c3e0f");
AVIMMessage msg = new AVIMMessage();
msg.setContent("Ping");
conv.sendMessage(msg,AVIMConversation.RECEIPT_MESSAGE_FLAG);

```
{% endblock %}

{% block messagePolicy_received_intro %}{% endblock %}

{% block message_unread %}
要开启未读消息，需要在 AVOSCloud 初始化语句后面加上：

```
AVIMClient.setOfflineMessagePush(true);
```

然后实现 AVIMConversationEventHandler 的代理方法 `onOfflineMessagesUnread` 来从服务端取回未读消息：

```
onOfflineMessagesUnread(AVIMClient client, AVIMConversation conversation, int unreadCount) {
  //如果有多个 conversation 有未读消息，此函数会执行多次
  if (unreadCount > 0) {
    // 可以根据 readCount 更新 UI
    
    // 也可以拉取对应的未读消息
    conversation.queryMessages(unreadCount, new AVIMMessagesQueryCallback() {
      @Override
      public void done(List<AVIMMessage> list, AVIMException e) {
        if (e == null) {
          // 获得对应的未读消息
        }
      }
    });
  }
}
```
`AVIMConversationEventHandler` 的实现和定义在[自身主动加入](#自身主动加入)里面有详细的代码和介绍。
{% endblock %}

{% block message_Relation_intro %}
消息类型之间的关系

![消息的类图](http://ac-lhzo7z96.clouddn.com/1440485935481)

{% endblock %}

{% block message_Properties_intro %}
消息类均包含以下公用属性：

属性|描述|类型
---|---|---
content|String|消息内容
clientId|String|指消息发送者的 clientId 
conversationId|String|消息所属对话 id
messageId|String|消息发送成功之后，由 LeanCloud 云端给每条消息赋予的唯一 id 
timestamp|long|消息发送的时间。消息发送成功之后，由 LeanCloud 云端赋予的全局的时间戳。
receiptTimestamp|long| 消息被对方接收到的时间。消息被接收之后，由 LeanCloud 云端赋予的全局的时间戳。
status|AVIMMessageStatus 枚举|消息状态，有五种取值：<br/><br/>`AVIMMessageStatusNone`（未知）<br/>`AVIMMessageStatusSending`（发送中）<br/>`AVIMMessageStatusSent`（发送成功）<br/>`AVIMMessageStatusReceipt`（被接收）<br/>`AVIMMessageStatusFailed`（失败）
ioType|AVIMMessageIOType 枚举|消息传输方向，有两种取值：<br/><br/>`AVIMMessageIOTypeIn`（发给当前用户）<br/>`AVIMMessageIOTypeOut`（由当前用户发出）
我们为每一种富媒体消息定义了一个消息类型，实时通信 SDK 自身使用的类型是负数（如下面列表所示），所有正数留给开发者自定义扩展类型使用，0 作为「没有类型」被保留起来。

消息 | 类型
--- | ---
文本消息|-1
图像消息|-2
音频消息|-3
视频消息|-4
位置消息|-5
文件消息|-6
 
{% endblock %}

{% set attributes = "`AVIMTypedMessage.attributes`" %}

{% block attributes_property %}attributes{% endblock %}

{% block customAttributesMessage_sent %}

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback() {
    @Override
    public void done(AVIMClient client, AVIMException e) {
        if (e == null) {
            AVIMImageMessage msg = new AVIMImageMessage(someLocalFile);
            Map<String, Object> attributes = new HashMap<String, Object>();
            attributes.put("location", "拉萨布达拉宫");
            attributes.put("Title", "这蓝天……我彻底是醉了");
            msg.setAttrs(attributes);
            client.getConversation("551260efe4b01608686c3e0f").sendMessage(msg,
                new AVIMConversationCallback() {
                    @Override
                    public void done(AVIMException e) {
                      if (e == null) {
                    // 发送成功
                    }
                }
            });
        }
      }
});
```
{% endblock %}

{% block customAttributesMessage_received %}

```
AVIMMessageManager.registerMessageHandler(AVIMImageMessage.class,
    new AVIMTypedMessageHandler<AVIMImageMessage>() {
        @Override
            public void onMessage(AVIMImageMessage msg, AVIMConversation conv, AVIMClient client) {
                //此处应该是"拉萨布达拉宫"
                System.out.println(msg.getAttrs().get("location"));
            }
    }
);

AVIMClient friend = AVIMClient.getInstance("friend");
friend.open(new AVIMClientCallback() {
    @Override
    public void done(AVIMClient client, AVIMException e) {
        if (e == null) {}
    }
});
```
{% endblock %}

{% block customMessage_create %}
继承于 AVIMTypedMessage，开发者也可以扩展自己的富媒体消息。其要求和步骤是：

* 实现新的消息类型，继承自 AVIMTypedMessage。这里需要注意两点：
  * 在 class 上增加一个 @AVIMMessageType(type=123) 的 Annotation，具体消息类型的值（这里是 `123`）由开发者自己决定（LeanCloud 内建的 [消息类型使用负数](#消息类详解)，所有正数都预留给开发者扩展使用）。
  * 在消息内部属性上要增加 @AVIMMessageField(name="") 的 Annotation，name 为可选字段在声明字段属性，同时自定义的字段要有对应的 getter/setter 方法。
* 调用 `AVIMMessageManager.registerAVIMMessageType()` 函数进行类型注册。
* 调用 `AVIMMessageManager.registerMessageHandler()` 函数进行消息处理 handler 注册。

AVIMTextMessage 的源码如下，可供参考：

```
@AVIMMessageType(type = -1)
public class AVIMTextMessage extends AVIMTypedMessage {

  @AVIMMessageField(name = "_lctext")
  String text;
  @AVIMMessageField(name = "_lcattrs")
  Map<String, Object> attrs;

  public String getText() {
    return this.text;
  }

  public void setText(String text) {
    this.text = text;
  }

  public Map<String, Object> getAttrs() {
    return this.attrs;
  }

  public void setAttrs(Map<String, Object> attr) {
    this.attrs = attr;
  }
}
```
{% endblock %}

{% block api_send_message_method_intro %}
#### 消息发送接口
在 Android SDK 中，发送消息的方法是：`AVIMConversation.sendMessage`，它最核心的一个重载声明如下：

```
/**
 *
 * @param message 发送的消息实体，可以是任何 AVIMMessage 子类
 * @param messageFlag AVIMConversation.TRANSIENT_MESSAGE_FLAG(0)：暂态消息，只有在消息发送时，对方也是在线的才能收到这条消息；
 *                    AVIMConversation.NONTRANSIENT_MESSAGE_FLAG(1)：非暂态消息，当消息发送时，对方不在线的话，消息会变成离线消息；
 *                    AVIMConversation.RECEIPT_MESSAGE_FLAG(17)：回执消息，当消息送到到对方以后，发送方会收到消息回执说明消息已经成功达到接收方
 * @param callback 消息发送之后的回调，发送异常或者发送成功都可以在回调里进行操作
 */
public void sendMessage(final AVIMMessage message, final int messageFlag, final AVIMConversationCallback callback)
```
为了满足通用需求，SDK 还提供了一个更为常用的重载声明：

```
/**
 *
 * @param message 发送的消息实体，可以是任何 AVIMMessage 子类
 * @param callback 消息发送之后的回调，发送异常或者发送成功都可以在回调里进行操作
 */
public void sendMessage(AVIMMessage message, AVIMConversationCallback callback)
```

其实本质上，调用 `sendMessage(message, callback)` 就等价于调用 `sendMessage(message,1, callback)` ，因为一般情况下消息存在的形式多以**非暂态**消息为主

{% endblock %}

{% block messagePolicy_sent_method %} `AVIMClient.OnMessageReceived` {% endblock %}

{% block messagePolicy_received_method %}{% endblock %}

{% block messagePolicy_received %}{% endblock %}

{% block conversation_init %}
```
AVIMClient jerry = AVIMClient.getInstance("Jerry");
jerry.open(new AVIMClientCallback() {
    @Override
    public void done(AVIMClient client, AVIMException e) {
        if (e == null) {
            // 创建名为“猫和老鼠”的对话
            client.createConversation(Arrays.asList("Bob", "Harry", "William"), "猫和老鼠", null,
                new AVIMConversationCreatedCallback() {
                    @Override
                        public void done(AVIMConversation conv, AVIMException e) {
                            if (e == null) {
                                // 创建成功
                            }
                    }
             });
        }
      }
});
```
{% endblock %}

{% block conversation_creation_api %}
### 创建对话
创建对话的接口在 `AVIMClient` 中共有 4 个方法重写，下面我们以参数最详尽的这个重写来说明其中每个参数的意义。

```
  /**
   * 创建或查询一个已有 conversation
   *
   * @param members 对话的成员
   * @param name 对话的名字
   * @param attributes 对话的额外属性
   * @param isTransient 是否是暂态对话
   * @param isUnique 如果已经存在符合条件的对话，是否返回已有对话
   *                 为 false 时，则一直为创建新的对话
   *                 为 true 时，则先查询，如果已有符合条件的对话，则返回已有的，否则，创建新的并返回
   *                 为 true 时，仅 members 为有效查询条件
   * @param callback
   */
  public void createConversation(final List<String> members, final String name,
      final Map<String, Object> attributes, final boolean isTransient, final boolean isUnique,
      final AVIMConversationCreatedCallback callback)
```
参数说明：

* members - 对话的初始成员列表。在对话创建成功后，这些成员会收到和邀请加入对话一样的相应通知。
* name - 对话的名字，主要是用于标记对话，让用户更好地识别对话。
* attributes - 额外属性
* isTransient - 是否为 [暂态对话](#聊天室)
* isUnique - 是否创建唯一对话，当 `isUnique` 为 true 时，如果当前已经有**相同成员**的对话存在则返回该对话，否则会创建新的对话。该值默认为 false。

<div class="callout callout-info">由于暂态对话不支持创建唯一对话，所以将 `isTransient` 和 `isUnique` 同时设为 true 时并不会产生预期效果。</div>
{% endblock %}

{% block event_memberJoin %} `onMemberJoined` {% endblock %}

{% block event_memberLeft %} `onMemberLeft` {% endblock %}

{% block event_kicked %} `onKicked` {% endblock %}

{% block event_invited %} `onInvited` {% endblock %}

{% block api_method_conversation_join %} `AVIMConversation.join`{% endblock %}

{% block api_method_conversation_invite %} `AVIMConversation.addMembersWithClientIds`{% endblock %}

{% block api_method_conversation_quit %} `AVIMConversation.quitWithCallback`{% endblock %}

{% block api_method_conversation_kick %} `AVIMConversation.removeMembersWithClientIds`{% endblock %}

{% block conversation_members_change_notice_intro %}
在 Android 中，开发者需要实现 `AVIMConversationEventHandler` 代理，并且为 `AVIMClient` 指定该代理的一个实例。

`AVIMConversationEventHandler` 的实现和定义在下一节[自身主动加入](#自身主动加入)里面有详细的代码和介绍。

{% endblock %}

{% block conversation_join %}

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	    //登录成功
		AVIMConversation conv = client.getConversation("551260efe4b01608686c3e0f");
		conv.join(new AVIMConversationCallback(){
			@Override
			public void done(AVIMException e){
			  if(e==null){
			  //加入成功
			  }
			}
		});
	  }
	}
});

```
{% endblock %}

{% block conversation_membersChanged_callBack %}
该群的其他成员（比如 Bob）登录之后，调用 `AVIMMessageManager.setConversationEventHandler` 设置一下回调的代理，会收到该操作的事件回调：

```
AVIMMessageManager.setConversationEventHandler(new CustomConversationEventHandler());
AVIMClient bob = AVIMClient.getInstance("Bob");
bob.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
	  }
	}
});

-- CustomConversationEventHandler.java
public class CustomConversationEventHandler extends AVIMConversationEventHandler {

  @Override
  public void onMemberLeft(AVIMClient client, AVIMConversation conversation, List<String> members,
      String kickedBy) {
      // 有其他成员离开时，执行此处逻辑
  }

  @Override
  public void onMemberJoined(AVIMClient client, AVIMConversation conversation,
      List<String> members, String invitedBy) {
      // 手机屏幕上会显示一小段文字：Tom 加入到 551260efe4b01608686c3e0f ；操作者为：Tom
      Toast.makeText(AVOSCloud.applicationContext,
        members + "加入到" + conversation.getConversationId() + "；操作者为： "
            + invitedBy, Toast.LENGTH_SHORT).show();
  }

  @Override
  public void onKicked(AVIMClient client, AVIMConversation conversation, String kickedBy) {
    // 当前 ClientId(Bob) 被踢出对话，执行此处逻辑
  }

  @Override
  public void onInvited(AVIMClient client, AVIMConversation conversation, String invitedBy) {
    // 当前 ClientId(Bob) 被邀请到对话，执行此处逻辑
  }
}

```
{% endblock %}

{% block conversation_invite %}

```
 AVIMClient jerry = AVIMClient.getInstance("Jerry");
    jerry.open(new AVIMClientCallback() {

      @Override
      public void done(AVIMClient client, AVIMException e) {
        if (e == null) {
          //登录成功
          final AVIMConversation conv = client.getConversation("551260efe4b01608686c3e0f");
          conv.join(new AVIMConversationCallback() {
            @Override
            public void done(AVIMException e) {
              if (e == null) {
                //加入成功
                conv.addMembers(Arrays.asList("Mary"), new AVIMConversationCallback() {
                  @Override
                  public void done(AVIMException e) {
                  }
                });
              }
            }
          });
        }
      }
    });

```
{% endblock %}

{% block conversation_invite_events %}
邀请成功以后，相关方收到通知的时序是这样的：

No.|邀请者|被邀请者|其他人
---|---|---|---
1|发出请求 addMembers| | 
2| |收到 onInvited 通知| 
3|收到 onMemberJoined 通知| | 收到 onMemberJoined 通知
{% endblock %}

{% block conversation_left %}

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
		final AVIMConversation conv = client.getConversation("551260efe4b01608686c3e0f");
		conv.join(new AVIMConversationCallback(){
			@Override
			public void done(AVIMException e){
			  if(e==null){
			  //加入成功
			  conv.quit(new AVIMConversationCallback(){
			    @Override
			    public void done(AVIMException e){
			      if(e==null){
			      //退出成功
			      }
			    } 
			  });
			  }
			}
		});
	  }
	}
});
``` 
{% endblock %}

{% block conversation_kick %}

```
AVIMClient william = AVIMClient.getInstance("William");
william.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
		final AVIMConversation conv = client.getConversation("551260efe4b01608686c3e0f");
		conv.join(new AVIMConversationCallback(){
			@Override
			public void done(AVIMException e){
			  if(e==null){
			  //加入成功
			  conv.kickMembers(Arrays.asList("Harry"),new AVIMConversationCallback(){
			  
			  	 @Override
			    public void done(AVIMException e){
			    }
			  );
			  }
			}
		});
	  }
	}
});
```
{% endblock %}

{% block conversation_kick_events %}
踢人时，相关方收到通知的时序如下：

No.|操作者（管理员）|被踢者|其他人
---|---|---|---
1|发出请求 kickMembers| | 
2| |收到 onKicked 通知| 
3|收到 onMemberLeft 通知| |收到 onMemberLeft 通知
{% endblock %}

{% block conversation_countMember_method %} `AVIMConversation.getMemberCount` {% endblock %}

{% block conversation_countMember %}

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
	  AVIMConversationQuery query = client.getQuery();
	  query.setLimit(1);
	  query.findInBackground(new AVIMConversationQueryCallback(){
       @Override
       public void done(List<AVIMConversation> convs,AVIMException e){
         if(e==null){
           if(convs!=null && !convs.isEmpty()){
             AVIMConversation conv = convs.get(0);
             conv.getMemberCount(new AVIMConversationMemberCountCallback(){
               
               @Override
               public void done(Integer count,AVIMException e){			
               if(e==null){						
               Log.d("Tom & Jerry","conversation got "+count+" members");
				}
               }
             });
           }
         }
       }
     });
	  }
	}
});
```
{% endblock %}

{% block table_conversation_attributes_intro %}
AVIMConversation 属性名 | _Conversation 字段|含义
--- | ------------ | -------------
`conversationId`| `objectId` |全局唯一的 Id
`name` |  `name` |成员共享的统一的名字
`members`|`m` |成员列表
`creator` | `c` |对话创建者
`attributes`| `attr`|自定义属性
`isTransient`|`tr`|是否为聊天室（暂态对话）
`lastMessageAt`|`lm`|该对话最后一条消息，也可以理解为最后一次活跃时间

{% endblock %}

{% block conversation_name %}

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
	  client.createConversation(Arrays.asList("Black"),"喵星人",null,
	           new AVIMConversationCreatedCallback(){
	           
	             @Override
	             public void done(AVIMConversation conv,AVIMException e){
	               if(e==null){
	                 //创建成功
	               }
	             }
	           });
	  }
	}
});

```
{% endblock %}

{% block conversation_changeName %}

```
AVIMClient black = AVIMClient.getInstance("Black");
black.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
	  AVIMConversation conv = client.getConversation("55117292e4b065f7ee9edd29");
	  conv.setName("聪明的喵星人");
	  conv.updateInfoInBackground(new AVIMConversationCallback(){
	    
	    @Override
	    public void done(AVIMException e){	    
	      if(e==null){
	      //更新成功
	      }
	    }
	  });
	  }
	}
});
```
{% endblock %}

{% block conversation_mute %}

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
	  AVIMConversation conv = client.getConversation("551260efe4b01608686c3e0f");
	  conv.mute(new AVIMConversationCallback(){
	  
	    @Override
	    public void done(AVIMException e){
	      if(e==null){
	      //设置成功
	      }
	    }
	  });
	  }
	}
});

```
{% endblock %}

{% block conversation_property_name %}`AVIMConversation.creator`{% endblock %}

{% block conversation_attributes_new %}
```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
	  HashMap<String,Object> attr = new HashMap<String,Object>();
	  attr.put("type","private");
    attr.put("isSticky",true);
	  client.createConversation(Arrays.asList("Jerry"),"猫和老鼠",attr,
	           new AVIMConversationCreatedCallback(){
	             @Override
	             public void done(AVIMConversation conv,AVIMException e){
	               if(e==null){
	                 //创建成功
	               }
	             }
	           });
	  }
	}
});
```
{% endblock %}

{% block conversation_attributes_modify %}{% endblock %}

{% block conversation_getSingle %}

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
	  AVIMConversationQuery query = client.getQuery();
	  query.whereEqualTo("objectId","551260efe4b01608686c3e0f");
	  query.findInBackground(new AVIMConversationQueryCallback(){
	    @Override
	    public void done(List<AVIMConversation> convs,AVIMException e){
	      if(e==null){
			  if(convs!=null && !convs.isEmpty()){
			    //convs.get(0) 就是想要的conversation
			  }
	      }
	    }
	  });
	  }
	}
});
```
{% endblock %}

{% block conversation_getList %}

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
	  AVIMConversationQuery query = client.getQuery();
	  query.findInBackground(new AVIMConversationQueryCallback(){
	    @Override
	    public void done(List<AVIMConversation> convs,AVIMException e){
	      if(e==null){
          //convs就是获取到的conversation列表
          //注意：按每个对话的最后更新日期（收到最后一条消息的时间）倒序排列
	      }
	    }
	  });	  
	  }
	}
});
```
{% endblock %}

{% block conversation_query_limit %}

```
AVIMConversationQuery query = client.getQuery();
query.limit(20);
query.findInBackground(new AVIMConversationQueryCallback(){
	@Override
	public void done(List<AVIMConversation> convs,AVIMException e){
	if(e==null){
    //convs就是获取到的conversation列表
    //注意：按每个对话的最后更新日期（收到最后一条消息的时间）倒序排列
	}
	}
});	
```
{% endblock %}

{% block pattern_conservation_query_default_property %}

```
// 查询对话名称为「LeanCloud 粉丝群」的对话
conversationQuery.whereEqualTo("name", "LeanCloud 粉丝群");

// 查询对话名称包含 「LeanCloud」 的对话
conversationQuery.whereContains("name", "LeanCloud");

// 查询过去24小时活跃的对话
Calendar yesterday= Calendar.getInstance();
yesterday.add(Calendar.DATE, -1);
conversationQuery.whereGreaterThan("lm", yesterday);
```
针对默认属性的查询可以如上进行构建。
{% endblock %}

{% block pattern_conservation_query_custom_property %}

```
// 查询话题为 DOTA2 对话
conversationQuery.whereEqualTo("attr.topic", "DOTA2");
// 查询等级大于 5 的对话
conversationQuery.whereGreaterThan("attr.level", 5);
```

在 Andorid SDK 中，如果在针对自定义查询的时候，不主动加上 `attr` 的前缀，SDK 会自动添加，比如上述的代码中查询话题为 DOTA2 的对话如下书写效果一致：

```
conversationQuery.whereEqualTo("topic", "DOTA2");
```
特别注意：

> 因为 Android 会自动添加 attr 前缀进行查询构建，所以在设置自定义属性的时候，**禁止**使用以下：`name`,`lm`,`c`,`tr`,`m`,`objectId`等已被默认属性占用的 key 值。

{% endblock %}

{% block conversation_query_content %}
条件查询又分为：比较查询、正则匹配查询、包含查询、空值查询，以下会做分类演示。
{% endblock %}

{% block conversation_messageHistoryByLimit %}

```
  AVIMClient tom = AVIMClient.getInstance("Tom");
  tom.open(new AVIMClientCallback() {

    @Override
    public void done(AVIMClient client, AVIMException e) {
      if (e == null) {
        //登录成功
        AVIMConversation conv = client.getConversation("551260efe4b01608686c3e0f");
        int limit = 10;// limit 取值范围 1~1000 之内的整数
        // 不使用 limit 默认返回 20 条消息
        conv.queryMessages(limit, new AVIMMessagesQueryCallback() {
          @Override
          public void done(List<AVIMMessage> messages, AVIMException e) {
            if (e == null) {
              //成功获取最新10条消息记录
            }
          }
        });
      }
    }
  });
```
{% endblock %}

{% block conversation_messageHistoryBeforeId %}

```

  AVIMClient tom = AVIMClient.getInstance("Tom");
  tom.open(new AVIMClientCallback(){

    @Override
    public void done(AVIMClient client,AVIMException e){
      if(e==null){
        //登录成功
        final AVIMConversation conv = client.getConversation("551260efe4b01608686c3e0f");
        conv.queryMessages(new AVIMMessagesQueryCallback(){
          @Override
          public void done(List<AVIMMessage> messages,AVIMException e){
            if(e==null){
              if(messages!=null && !messages.isEmpty()){
                Log.d("Tom & Jerry","got "+messages.size()+" messages ");

                //返回的消息一定是时间增序排列，也就是最早的消息一定是第一个
                AVIMMessage oldestMessage = messages.get(0);

                conv.queryMessages(oldestMessage.getMessageId(), oldestMessage.getTimestamp(),20,
                        new AVIMMessageQueryCallback(){
                          @Override
                          public void done(List<AVIMMessage> msgs,AVIMException e){
                            if(e== null){
                              //查询成功返回
                              Log.d("Tom & Jerry","got "+msgs.size()+" messages ");
                            }
                          }
                        });
              }
            }
          }
        });
      }
    }
  });
```
{% endblock %}

{% block conversation_messageHistory_pager %}
```
  final int pageSize = 10;
  conversation.queryMessages(pageSize, new AVIMMessagesQueryCallback() {
    @Override
    public void done(List<AVIMMessage> firstPage, AVIMException e) {
      if (firstPage != null && !firstPage.isEmpty()) {
        Log.d("Tom & Jerry", "got " + firstPage.size() + " messages ");

        // 获取第一页的消息里面最旧的一条消息
        AVIMMessage pager = firstPage.get(0);
        conversation.queryMessages(pager.getMessageId(), pager.getTimestamp(), pageSize, new AVIMMessagesQueryCallback() {
          @Override
          public void done(List<AVIMMessage> secondPage, AVIMException e) {
            // secondPage 就是第二页的数据
          }
        });
      }
    }
  });
```
{% endblock %}

{% block disable_im_cache %}

```java
AVIMClient.setMessageQueryCacheEnable(false);
```
{% endblock %}

{% block conversation_query_equalTo %}

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
	  AVIMConversationQuery query = client.getQuery();
	  query.whereEqualTo("attr.topic","movie");
	  query.findInBackground(new AVIMConversationQueryCallback(){
	    @Override
	    public void done(List<AVIMConversation> convs,AVIMException e){
	      if(e==null){
			  if(convs!=null && !convs.isEmpty()){
			    //获取符合查询条件的Conversation列表
			  }
	      }
	    }
	  });
	  }
	}
});
```
{% endblock %}

{% block conversation_query_notEqualTo %}

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
	  AVIMConversationQuery query = client.getQuery();
	  query.whereNotEqualTo("attr.type","private");
	  query.setLimit(50);//limit 设为 50 ,默认为 10 个
	  
	  query.findInBackground(new AVIMConversationQueryCallback(){
	    @Override
	    public void done(List<AVIMConversation> convs,AVIMException e){
	      if(e==null){
			  if(convs!=null && !convs.isEmpty()){
			    //获取符合查询条件的Conversation列表
			  }
	      }
	    }
	  });
	  }
	}
});
```
{% endblock %}

{% block conversation_query_greaterThan %}

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
	  AVIMConversationQuery query = client.getQuery();
	  query.whereGreaterThan("attr.age",18);
	  
	  query.findInBackground(new AVIMConversationQueryCallback(){
	    @Override
	    public void done(List<AVIMConversation> convs,AVIMException e){
	      if(e==null){
			  if(convs!=null && !convs.isEmpty()){
			    //获取符合查询条件的Conversation列表
			  }
	      }
	    }
	  });
	  }
	}
});
```
{% endblock %}

{% block conversation_query_regexIntro %}
匹配查询是指在 `AVIMConversationQuery` 的查询条件中使用正则表达式来匹配数据。
{% endblock %}

{% block conversation_query_regex %}

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
	  AVIMConversationQuery query = client.getQuery();
	  query.whereMatches("attr.language","[\\u4e00-\\u9fa5]"); //attr.language 是中文字符 
	  
	  query.findInBackground(new AVIMConversationQueryCallback(){
	    @Override
	    public void done(List<AVIMConversation> convs,AVIMException e){
	      if(e==null){
			  if(convs!=null && !convs.isEmpty()){
			    //获取符合查询条件的Conversation列表
			  }
	      }
	    }
	  });
	  }
	}
});
```
{% endblock %}

{% block conversation_query_contains %}

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
	  AVIMConversationQuery query = client.getQuery();
	  
	  //查询attr.keywords 包含 「教育」的Conversation
	  query.whereContains("attr.keywords","教育"); 
	  
	  query.findInBackground(new AVIMConversationQueryCallback(){
	    @Override
	    public void done(List<AVIMConversation> convs,AVIMException e){
	      if(e==null){
			  if(convs!=null && !convs.isEmpty()){
			    //获取符合查询条件的Conversation列表
			  }
	      }
	    }
	  });
	  }
	}
});
```
{% endblock %}

{% block conversation_query_findJoinedMemebers %}

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
	  AVIMConversationQuery query = client.getQuery();
	  
	  //查询对话成员有 Bob 和 Jerry的Conversation
	  query.withMembers(Arrays.as("Bob","Jerry"));
	  
	  query.findInBackground(new AVIMConversationQueryCallback(){
	    @Override
	    public void done(List<AVIMConversation> convs,AVIMException e){
	      if(e==null){
			  if(convs!=null && !convs.isEmpty()){
			    //获取符合查询条件的Conversation列表
			  }
	      }
	    }
	  });
	  }
	}
});
```
{% endblock %}


{% block conversation_query_exists %}
#### 空值查询

空值查询是指查询相关列是否为空值的方法，例如要查询 lm 列为空值的对话：

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){

  @Override
  public void done(AVIMClient client,AVIMException e){
    if(e==null){
    //登录成功
    AVIMConversationQuery query = client.getQuery();
    
    //查询 lm 列为空的 Conversation 列表
    query.whereDoesNotExist("lm");
    
    query.findInBackground(new AVIMConversationQueryCallback(){
      @Override
      public void done(List<AVIMConversation> convs,AVIMException e){
        if(e==null){
          if(convs!=null && !convs.isEmpty()){
            //获取符合查询条件的Conversation列表
          }
        }
      }
    });
    }
  }
});

```

如果要查询 lm 列不为空的对话，则替换为如下：

```
query.whereExists("lm");

```
{% endblock %}


{% block conversation_query_combination %}

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
	  AVIMConversationQuery query = client.getQuery();
	  
	  //查询 attr.keywords 包含 「教育」并且 attr.age 小于 18 的对话
	  query.whereContains("attr.keywords", "教育");
	  query.whereLessThan("attr.age", 18);
	  
	  query.findInBackground(new AVIMConversationQueryCallback(){
	    @Override
	    public void done(List<AVIMConversation> convs,AVIMException e){
	      if(e==null){
			  if(convs!=null && !convs.isEmpty()){
			    //获取符合查询条件的Conversation列表
			  }
	      }
	    }
	  });
	  }
	}
});
```
{% endblock %}

{% block conversation_query_count %}
```
- 初始化 ClientId = Tom
- Tom 登录
- 构建查询条件：attr.keywords 包含「教育」、attr.age < 18
- 执行查询，获取符合条件的对话的数量
```
{% endblock %}

{% block chatroom_intro %}
和建立普通对话类似，建立一个聊天室只是在 `AVIMClient.createConversation(conversationMembers, name, attributes, isTransient, callback)` 中传入 `isTransient=true`。
{% endblock %}

{% block chatroom_new %}

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){

	@Override
	public void done(AVIMClient client,AVIMException e){
	  if(e==null){
	  //登录成功
	  //创建一个 名为 "HelloKitty PK 加菲猫" 的暂态对话
	  client.createConversation(Collections.emptyList(),"HelloKitty PK 加菲猫",null,true,
	    new AVIMConversationCreatedCallback(){
	      @Override
	      public void done(AVIMConversation conv,AVIMException e){
	        
	      }
	    });
	  }
	}
});

```
{% endblock %}

{% block chatroom_count_method %} `AVIMConversation.getMemberCount()` {% endblock %}

{% block chatroom_count %}

```
private void TomQueryWithLimit() {
  AVIMClient tom = AVIMClient.getInstance("Tom");
  tom.open(new AVIMClientCallback() {
    
    @Override
    public void done(AVIMClient client, AVIMException e) {
      if (e == null) {
        //登录成功
        AVIMConversationQuery query = tom.getQuery();
        query.setLimit(1);
        //获取第一个对话
        query.findInBackground(new AVIMConversationQueryCallback() {
          @Override
          public void done(List<AVIMConversation> convs, AVIMException e) {
            if (e == null) {
              if (convs != null && !convs.isEmpty()) {
                AVIMConversation conv = convs.get(0);
                //获取第一个对话的
                conv.getMemberCount(new AVIMConversationMemberCountCallback() {
                  
                  @Override
                  public void done(Integer count, AVIMException e) {
                    if (e == null) {
                      Log.d("Tom & Jerry", "conversation got " + count + " members");
                    }
                  }
                });
              }
            }
          }
        });
      }
    }
  });
}
```
{% endblock %}

{% block chatroom_query_method %} `AVIMConversationQuery.findInBackground` {% endblock %}

{% block chatroom_query_method2 %}以 `where` 开头的{% endblock %}

{% block create_query_instance_method %}`AVIMClient.getQuery()`{% endblock %}

{% block chatroom_query_single %}

```
  AVIMClient tom = AVIMClient.getInstance("Tom");
  tom.open(new AVIMClientCallback() {

    @Override
    public void done(AVIMClient client, AVIMException e) {
      if (e == null) {
        //登录成功
        //查询 attr.topic 为 "奔跑吧，兄弟" 的暂存聊天室
        AVIMConversationQuery query = client.getQuery();
        query.whereEqualTo("attr.topic", "奔跑吧，兄弟");
        query.whereEqualTo("tr", true);
        //获取第一个对话
        query.findInBackground(new AVIMConversationQueryCallback() {
          @Override
          public void done(List<AVIMConversation> convs, AVIMException e) {
            if (e == null) {
              if (convs != null && !convs.isEmpty()) {
                AVIMConversation conv = convs.get(0);
                //获取第一个对话的
                conv.getMemberCount(new AVIMConversationMemberCountCallback() {
                  @Override
                  public void done(Integer count, AVIMException e) {
                    if (e == null) {
                      Log.d("Tom & Jerry", "conversation got " + count + " members");
                    }
                  }
                });
              }
            }
          }
        });
      }
    }
  });
```
{% endblock %}

{% block message_priority %}
### 聊天室消息等级

为了保证消息的时效性，当聊天室消息过多导致客户端连接堵塞时，服务器端会根据消息等级来丢弃部分不重要的消息。可通过调用 `AVIMMessage` 的如下方法来设置消息等级：

```
 public void setPriority(AVIMMessagePriorityType priority)
```

`AVIMMessagePriorityType` 包含以下枚举值：
* Default - 默认值
* High - 高等级（针对时效性要求比较高的消息，比如直播聊天室中的礼物、弹幕等类型消息）
* Normal - 正常等级
* Low - 低等级（针对时效性要求比较低的消息，比如直播聊天室中的普通消息）

注意：此功能仅针对聊天室消息，普通会话消息不需要设置此等级，因为普通会话消息不会被丢弃。

{% endblock %}

{% block networkStatus %}
与网络相关的通知（网络断开、恢复等）会由 `AVIMClientEventHandler` 做出响应，接口函数有：

* `onConnectionPaused()` 指网络连接断开事件发生，此时聊天服务不可用。
* `onConnectionResume()` 指网络连接恢复正常，此时聊天服务变得可用。
* `onClientOffline()` 指[单点登录](#单点登录)被踢下线的事件。

在网络中断的情况下，所有的消息收发和对话操作都会出现问题。

通过 `AVIMClient.setClientEventHandler()` 可以设定全局的客户端事件响应（ClientEventHandler）。
{% endblock %}

{% block logout %}

```
AVIMClient tom = AVIMClient.getInstance("Tom");
tom.open(new AVIMClientCallback(){
  
  @Override
  public void done(AVIMClient client,AVIMException e){
  	if(e==null){
  	  //登录成功
  	  client.close(new AVIMClientCallback(){
  	  	@Override
  	  	public void done(AVIMClient client,AVIMException e){
  	  		if(e==null){
  	  		//登出成功
  	  		}
  	  	}
  	  });
  	}
  }
});
```
{% endblock %}

{% block conversation_security %}
客户端这边究竟该如何使用呢？我们只需要实现 SignatureFactory 接口，然后在用户登录之前，把这个接口的实例赋值给 AVIMClient 即可（`AVIMClient.setSignatureFactory(factory)`）。

设定了 signatureFactory 之后，对于需要鉴权的操作，实时通信 SDK 与服务器端通讯的时候都会带上应用自己生成的 Signature 信息，LeanCloud 云端会使用 app 的 masterKey 来验证信息的有效性，保证聊天渠道的安全。

对于 SignatureFactory 接口，我们只需要实现这两个函数即可：

```
  /**
   * 实现一个基础签名方法 其中的签名算法会在SessionManager和AVIMClient(V2)中被使用
   */
  public Signature createSignature(String peerId, List<String> watchIds) throws SignatureException;

  /**
   * 实现AVIMConversation相关的签名计算
   * 
   * @param conversationId
   * @param clientId
   * @param targetIds - 此次操作的member的clientIds
   * @param action - 此次行为的动作，行为分别对应常量 invite（加群和邀请）和 kick（踢出群）
   * @return
   * @throws SignatureException 如果签名计算中间发生任何问题请抛出本异常
   */  /**
   * 实现AVIMConversation相关的签名计算
   * @param action - 此次行为的动作，行为分别对应常量 invite（加群和邀请）和 kick（踢出群）
   */
  public Signature createConversationSignature(String conversationId, String clientId,
      List<String> targetIds, String action) throws SignatureException;
```

`createSignature` 函数会在用户登录、对话创建的时候被调用，`createConversationSignature` 会在对话加入成员、邀请成员、踢出成员等操作时被调用。

你需要做的就是按照前文所述的签名算法实现签名，其中 `Signature` 声明如下：

```
public class Signature {
  public List<String> getSignedPeerIds();
  public void setSignedPeerIds(List<String> signedPeerIds);

  public String getSignature();
  public void setSignature(String signature);

  public long getTimestamp();
  public void setTimestamp(long timestamp);

  public String getNonce();
  public void setNonce(String nonce);
}
```

其中四个属性分别是:

* signature 签名
* timestamp 时间戳，单位秒
* nonce 随机字符串 nonce
* signedPeerIds 放行的 clientId 列表，v2 中已经**废弃不用**

下面的代码展示了基于 LeanCloud 云引擎进行签名时，客户端的实现片段，你可以参考它来完成自己的逻辑实现：

```
public class KeepAliveSignatureFactory implements SignatureFactory {
 @Override
 public Signature createSignature(String peerId, List<String> watchIds) {
   Map<String,Object> params = new HashMap<String,Object>();
   params.put("self_id",peerId);
   params.put("watch_ids",watchIds);

   try{
     Object result =  AVCloud.callFunction("sign",params);
     if(result instanceof Map){
       Map<String,Object> serverSignature = (Map<String,Object>) result;
       Signature signature = new Signature();
       signature.setSignature((String)serverSignature.get("signature"));
       signature.setTimestamp((Long)serverSignature.get("timestamp"));
       signature.setNonce((String)serverSignature.get("nonce"));
       return signature;
     }
   }catch(AVException e){
     throw (SignatureFactory.SignatureException) e;
   }
   return null;
 }

  @Override
  public Signature createConversationSignature(String convId, String peerId, List<String> targetPeerIds,String action){
   Map<String,Object> params = new HashMap<String,Object>();
   params.put("client_id",peerId);
   params.put("conv_id",convId);
   params.put("members",targetPeerIds);
   params.put("action",action);

   try{
     Object result = AVCloud.callFunction("sign2",params);
     if(result instanceof Map){
        Map<String,Object> serverSignature = (Map<String,Object>) result;
        Signature signature = new Signature();
        signature.setSignature((String)serverSignature.get("signature"));
        signature.setTimestamp((Long)serverSignature.get("timestamp"));
        signature.setNonce((String)serverSignature.get("nonce"));
        return signature;
     }
   }catch(AVException e){
     throw (SignatureFactory.SignatureException) e;
   }
   return null;
  }
}
```
{% endblock %}

{% block connect_with_tag %}

```java
    // 第二个参数：登录标记 Tag
    AVIMClient currentClient = AVIMClient.getInstance(clientId,"Mobile");
    currentClient.open(new AVIMClientCallback() {
      @Override
      public void done(AVIMClient avimClient, AVIMException e) {
        if(e == null){
          // 与云端建立连接成功
        }
      }
    });
```

{% endblock %}

{% block disconnected_by_server_with_same_tag %}

```java
public class MyApplication extends Application{
  public void onCreate(){
   ...
   AVOSCloud.initialize(this,"{{appid}}","{{appkey}}");
   // 自定义实现的 AVIMClientEventHandler 需要注册到 SDK 后，SDK 才会通过回调 onClientOffline 来通知开发者
   AVIMClient.setClientEventHandler(new AVImClientManager());
   ...
  }
}

public class AVImClientManager extends AVIMClientEventHandler {
  ...
  @Override
  public void onClientOffline(AVIMClient avimClient, int i) {
    if(i == 4111){
      // 适当地弹出友好提示，告知当前用户的 Client Id 在其他设备上登陆了
    }
  }
  ...
}
```

{% endblock %}

{% block client_auto_open %}
### 自动登录

如果开发者希望控制 App 重新启动后是否由 SDK 自动登录实时通讯，这可通过如下接口实现：

```
AVIMClient.setAutoOpen(false);
```

如果为 true，SDK 会在 App 重新启动后进行实时通讯的自动重连，如果为 false，则 App 重新启动后不会做自动重连操作，默认值为 true。

注意：此设置并不影响在 App 生命周期内因网络获取等问题造成的重连。
{% endblock %}

{% block code_set_query_policy %}

```java
  // 设置 AVIMConversationQuery的查询策略
  public void setQueryPolicy(AVQuery.CachePolicy policy);
```
{% endblock %}

{% block code_query_from_local_cache %}
有时你希望先走网络查询，发生网络错误的时候，再从本地查询，可以这样：

```java
    AVIMConversationQuery query = client.getQuery();
    query.setQueryPolicy(AVQuery.CachePolicy.NETWORK_ELSE_CACHE);
    query.findInBackground(new AVIMConversationQueryCallback() {
      @Override
      public void done(List<AVIMConversation> conversations, AVIMException e) {
        
      }
    });
```
{% endblock %}

{% block link_avquery_chache %}[存储指南 - AVQuery 缓存查询](leanstorage_guide-android.html#缓存查询) 一节。
{% endblock %}

