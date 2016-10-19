{% extends "./leanengine_cloudfunction_guide.tmpl" %}

{% set platformName = "Python" %}
{% set productName = "LeanEngine" %}
{% set storageName = "LeanStorage" %}
{% set leanengine_middleware = "[LeanCloud Python SDK](https://github.com/leancloud/python-sdk)" %}
{% set storage_guide_url = "[Python SDK](./leanstorage_guide-python.html)" %}
{% set cloud_func_file = "`$PROJECT_DIR/cloud.py`" %}
{% set runFuncName = "`leancloud.cloudfunc.run`" %}
{% set defineFuncName = "`engine.define`" %}
{% set runFuncApiLink = "[leancloud.cloudfunc.run](http://leancloud.readthedocs.io/zh_CN/latest/#leancloud.cloudfunc.run)" %}
{% set hook_before_save = "before_save" %}
{% set hook_after_save = "after_save" %}
{% set hook_before_update = "before_update" %}
{% set hook_after_update = "after_update" %}
{% set hook_before_delete = "before_delete" %}
{% set hook_after_delete = "after_delete" %}
{% set hook_on_verified = "on_verified" %}
{% set hook_on_login = "on_login" %}
{% set hook_message_received = "_messageReceived" %}
{% set hook_receiver_offline = "_receiversOffline" %}
{% set hook_message_sent = "_messageSent" %}
{% set hook_conversation_start = "_conversationStart" %}
{% set hook_conversation_started = "_conversationStarted" %}
{% set hook_conversation_add = "_conversationAdd" %}
{% set hook_conversation_remove = "_conversationRemove" %}
{% set hook_conversation_update = "_conversationUpdate" %}

{% block initialize %}
## 初始化

定义云函数 / Hook 函数都需要一个 leancloud.Engine 实例，你需要在项目中自己初始化此实例。

```python
import leancloud

engine = leancloud.Engine(your_wsgi_func)
```

更多关于 **WSGI 函数** 的内容，请参考 [WSGI 接口](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432012393132788f71e0edad4676a3f76ac7776f3a16000) 或者 [PEP333](https://www.python.org/dev/peps/pep-0333/)。
{% endblock %}

{% block cloudFuncExample %}
```python
@engine.define
def averageStars(movie, **params):
    reviews = leancloud.Query(Review).equal_to('movie', movie).find()
    result = sum(x.get('stars') for x in reviews)
    return result
```
{% endblock %}

{% block cloudFuncParams %}
调用云函数时的参数会直接传递给云函数，因此直接声明这些参数即可。另外调用云函数时可能会根据不同情况传递不同的参数，这时如果定义云函数时没有声明这些参数，会触发 Python 异常，因此建议声明一个额外的关键字参数（关于关键字参数，请参考[此篇文档](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001431752945034eb82ac80a3e64b9bb4929b16eeed1eb9000) 中`关键字参数`一节。）来保存多余的参数。

```python
@engine.define
def my_cloud_func(foo, bar, baz, **params):
    pass
```

如果想获取触发此云函数的用户，可以直接通过 `engine.current_user` 获取。
{% endblock %}


{% block runFuncExample %}

参考[调用云函数](#SDK_调用云函数)一节。

但是这样调用会真的发起一次 HTTP 请求，去请求部署在云引擎上的云函数。如果想要直接调用本地（当前进程）中的云函数，或者发起调用就是在云引擎中，想要省去一次 HTTP 调用的开销，可以使用 `leancloud.cloudfunc.run.local` 来取代 `leanengine.cloudfunc.run`，这样会直接在当前进程中执行一个函数调用，而不会发起 HTTP 请求来调用此云函数。

{% endblock %}

{% block beforeSaveExample %}
```python
@engine.before_save('Review')  # Review 为需要 hook 的 class 的名称
def before_review_save(review):
	comment = review.get('comment')
	if not comment:
		raise leancloud.LeanEngineError(message='No comment!')
	if len(comment) > 140:
		review.comment.set('comment', comment[:137] + '...')
```
{% endblock %}

{% block afterSaveExample %}
```python
import leancloud


@engine.after_save('Comment')  # Comment 为需要 hook 的 class 的名称
def after_comment_save(comment):
	post = leancloud.Query('Post').get(comment.id)
	post.increment('commentCount')
	try:
		post.save()
	except leancloud.LeanCloudError:
		raise leancloud.LeanEngineError(message='An error occurred while trying to save the Post. ')
```
{% endblock %}

{% block afterSaveExample2 %}
```python
@engine.after_save('_User')
def after_user_save(user):
    print user
    user.set('from', 'LeanCloud')
    try:
        user.save()
    except LeanCloudError, e:
        print 'error:', e
```
{% endblock %}

{% block beforeUpdateExample %}
```python
@engine.before_update('Review')
def before_hook_object_update(obj):
    # 如果 comment 字段被修改了，检查该字段的长度
    assert obj.updated_keys == ['clientValue']
    if 'comment' not in obj.updated_keys:
        # comment 字段没有修改，跳过检查
        return
    if len(obj.get('comment')) > 140:
        # 拒绝过长的修改
        raise engine.LeanEngineError(message='comment 长度不得超过 140 个字符')
```
{% endblock %}

{% block afterUpdateExample %}
```python
import leancloud


@engine.after_update('Article')  # Article 为需要 hook 的 class 的名称
def after_article_update(article):
	print 'article with id {} updated!'.format(article.id)
```
{% endblock %}

{% block beforeDeleteExample %}
```python
import leancloud


@engine.before_delete('Album')  # Album 为需要 hook 的 class 的名称
def before_album_delete(album):
    query = leancloud.Query('Photo')
    query.equal_to('album', album)
    try:
        matched_count = query.count()
    except leancloud.LeanCloudError:
        raise engine.LeanEngineError(message='cloud code error')
    if count > 0:
	     raise engine.LeanEngineError(message='Can\'t delete album if it still has photos.')
```
{% endblock %}

{% block afterDeleteExample %}
```python
import leancloud


@engine.after_delete('Album')  # Album 为需要 hook 的 class 的名称
def after_album_delete(album):
    query = leancloud.Query('Photo')
    query.equal_to('album', album)
    try:
        query.destroy_all()
    except leancloud.LeanCloudError:
        raise leancloud.LeanEngineError(message='cloud code error')
```
{% endblock %}

{% block onVerifiedExample %}
```python
@engine.on_verified('sms')
def on_sms_verified(user):
	print user
```
{% endblock %}

{% block onLoginExample %}
```python
@engine.on_login
def on_login(user):
    print 'on login:', user
    if user.get('username') == 'noLogin':
      # 如果抛出 LeanEngineError，则用户无法登录（收到 401 响应）
      raise LeanEngineError('Forbidden')
    # 没有抛出异常，函数正常执行完毕的话，用户可以登录
```
{% endblock %}

{% block errorCodeExample %}
错误响应码允许自定义。云引擎抛出的  `LeanCloudError`（数据存储 API 会抛出此异常）会直接将错误码和原因返回给客户端。若想自定义错误码，可以自行构造 `LeanEngineError`，将 `code` 与 `error` 传入。否则 `code` 为 1， `message` 为错误对象的字符串形式。

```python
@engine.define
def error_code(**params):
    leancloud.User.login('not_this_user', 'xxxxxxx')
```
{% endblock %}

{% block errorCodeExample2 %}
```python
from leancloud import LeanEngineError

@engine.define
def custom_error_code(**params):
    raise LeanEngineError(123, '自定义错误信息')
```
{% endblock %}

{% block errorCodeExampleForHooks %}
```python
@engine.before_save('Review')  # Review 为需要 hook 的 class 的名称
def before_review_save(review):
    comment = review.get('comment')
    if not comment:
      raise leancloud.LeanEngineError(
        code=123,
        message='自定义错误信息'
      )
```
{% endblock %}

{% block hookDeadLoop %}
#### 防止死循环调用

在实际使用中有这样一种场景：在 `Post` 类的 `{{hook_after_update}}` Hook 函数中，对传入的 `Post` 对象做了修改并且保存，而这个保存动作又会再次触发 `{{hook_after_update}}`，由此形成死循环。针对这种情况，我们为所有 Hook 函数传入的 `leancloud.Object` 对象做了处理，以阻止死循环调用的产生。

不过请注意，以下情况还需要开发者自行处理：

- 对传入的 `leancloud.Object` 对象进行 `fetch` 操作。
- 重新构造传入的 `leancloud.Object` 对象，如使用 `leancloud.Object.create_without_data()` 方法。

对于使用上述方式产生的对象，请根据需要自行调用以下 API：

- `leancloud.Object.disable_before_hook()` 或 
- `leancloud.Object.disable_after_hook()` 

这样，对象的保存或删除动作就不会再次触发相关的 Hook 函数。

```python
@engine.after_update('Post')
def after_post_update(post):
    # 直接修改并保存对象不会再次触发 after update hook 函数
    post.set('foo', 'bar')
    post.save()

    # 如果有 fetch 操作，则需要在新获得的对象上调用相关的 disable 方法
    # 来确保不会再次触发 Hook 函数
    post.fetch()
    post.disable_after_hook()
    post.set('foo', 'bar')

    # 如果是其他方式构建对象，则需要在新构建的对象上调用相关的 disable 方法
    # 来确保不会再次触发 Hook 函数
    post = leancloud.Object.extend('Post').create_without_data(post.id)
    post.disable_after_hook()
    post.save()
```

{% endblock %}

{% block code_hook_message_received %}

```python
@engine.define
def _messageReceived(**params):
    # params = {
    #     'fromPeer': 'Tom',
    #     'receipt': false,
    #     'groupId': null,
    #     'system': null,
    #     'content': '{"_lctext":"耗子，起床！","_lctype":-1}',
    #     'convId': '5789a33a1b8694ad267d8040',
    #     'toPeers': ['Jerry'],
    #     '__sign': '1472200796787,a0e99be208c6bce92d516c10ff3f598de8f650b9',
    #     'bin': false,
    #     'transient': false,
    #     'sourceIP': '121.239.62.103',
    #     'timestamp': 1472200796764,
    # }
    print('_messageReceived start')
    content = json.loads(params['content'])
    text = content._lctext
    print('text:', text)
    processed_content = text.replace('XX中介', '**')
    print('_messageReceived end')
    # 必须含有以下语句给服务端一个正确的返回，否则会引起异常
    return {
        'content': processed_content,
    }
```
{% endblock %}

{% block code_hook_receiver_offline %}

```python
@engine.define
def _receiversOffline(**params):
    print('_receiversOffline start')
    # params['content'] 为消息内容
    content = params['content']
    short_content = content[:6]
    print('short_content:', short_content)
    payloads = {
        # 自增未读消息的数目，不想自增就设为数字
        'badge': 'Increment',
        'sound': 'default',
        # 使用开发证书
        '_profile': 'dev',
        'alert': short_content,
    }
    print('_receiversOffline end')
    return {
        'pushMessage': json.dumps(payloads),
    }
```
{% endblock %}


{% block code_hook_message_sent %}

```python
@engine.define
def _messageSent(**params):
    print('_messageSent start')
    print('params:', params)
    print('_messageSent end')
    return {}

# 在云引擎中打印的日志如下：
# _messageSent start
# params: {'__sign': '1472703266575,30e1c9b325410f96c804f737035a0f6a2d86d711',
#  'bin': False,
#  'content': '12345678',
#  'convId': '5789a33a1b8694ad267d8040',
#  'fromPeer': 'Tom',
#  'msgId': 'fptKnuYYQMGdiSt_Zs7zDA',
#  'offlinePeers': ['Jerry'],
#  'onlinePeers': [],
#  'receipt': False,
#  'sourceIP': '114.219.127.186',
#  'timestamp': 1472703266522,
#  'transient': False}
# _messageSent end
```
{% endblock %}

{% block code_hook_conversation_start %}

```python
@engine.define
def _conversationStart(**params):
    print('_conversationStart start')
    print('params:', params)
    print('_conversationStart end')
    return {}

# 在云引擎中打印的日志如下：
# _conversationStart start
# params: {'__sign': '1472703266397,b57285517a95028f8b7c34c68f419847a049ef26',
#  'attr': {'name': 'Tom & Jerry'},
#  'initBy': 'Tom',
#  'members': ['Tom', 'Jerry']}
# _conversationStart end
```
{% endblock %}

{% block code_hook_conversation_started %}

```python
@engine.define
def _conversationStarted(**params):
    print('_conversationStarted start')
    print('params:', params)
    print('_conversationStarted end')
    return {}

# 在云引擎中打印的日志如下：
# _conversationStarted start
# params: {'__sign': '1472723167361,f5ceedde159408002fc4edb96b72aafa14bc60bb',
#  'convId': '5789a33a1b8694ad267d8040'}
# _conversationStarted end
```
{% endblock %}

{% block code_hook_conversation_add %}

```python
@engine.define
def _conversationAdd(**params):
    print('_conversationAdd start')
    print('params:', params)
    print('_conversationAdd end')
    return {}

在云引擎中打印的日志如下：
# _conversationAdd start
# params: {'__sign': '1472786231813,a262494c252e82cb7a342a3c62c6d15fffbed5a0',
#  'convId': '5789a33a1b8694ad267d8040',
#  'initBy': 'Tom',
#  'members': ['Mary']}
# _conversationAdd end
```
{% endblock %}

{% block code_hook_conversation_remove %}

```python
@engine.define
def _conversationRemove(**params):
    print('_conversationRemove start')
    print('params:', params)
    print('removed client Id:', params['members'][0]);
    print('_conversationRemove end')
    return {}

# 在云引擎中打印的日志如下：
# _conversationRemove start
# params: {'__sign': '1472787372605,abdf92b1c2fc4c9820bc02304f192dab6473cd38',
#  'convId': '57c8f3ac92509726c3dadaba',
#  'initBy': 'Tom',
#  'members': ['Jerry']}
# removed client Id: Jerry
# _conversationRemove end
```
{% endblock %}

{% block code_hook_conversation_update %}

```python
@engine.define
def _conversationUpdate(**params):
    print('_conversationUpdate start')
    print('params:', params)
    print('name:', params['attr']['name'])
    print('_conversationUpdate end')
    return {}

# 在云引擎中打印的日志如下：
# _conversationUpdate start
# params: {'__sign': '1472787372605,abdf92b1c2fc4c9820bc02304f192dab6473cd38',
#  'convId': '57c8f3ac92509726c3dadaba',
#  'initBy': 'Tom',
#  'members': ['Jerry']}
# name: 聪明的喵星人
# _conversationUpdate end
```
{% endblock %}

{% block masterKeyInit %}
```python
# 第一个参数为 App Id
leancloud.init('{{appid}}', app_key='{{appKey}}', master_key='{{masterkey}}')
leancloud.use_master_key(True)
```
{% endblock %}
