# Cloud Query Language 详细指南

Cloud Query Language（简称 CQL） 是 LeanCloud 为查询 API 定制的一套类似 SQL 查询语法的子集和变种，主要目的是降低大家学习 LeanCloud 查询的 API 的成本，可以使用传统的 SQL 语法来查询 LeanCloud 应用内的数据。

本文档将详细介绍 CQL 的语法和常见用法。

## 与 SQL 主要差异

* 不支持 join ，关联查询提供 include， relatedTo 等替代，参考[关系查询](#关系查询)。
* 不支持事务。
* 更新、删除要求一定要提供 objectId=xxx 的条件，只能根据 objectId 和其他条件来更新或者删除某个文档，不提供批量更新和删除。
* 不支持锁。
* 仅支持部分 SQL 函数，请参考 [内置函数](#内置函数)。
* 不支持 group by、having、sum、distinct 等分组聚合查询语法。

## 基本语法

插入一条数据：

```sql
insert into GameScore(name, score) values('leancloud', 100)
```

更新一条数据：

```sql
update GameScore set score=90 where objectId='558e20cbe4b060308e3eb36c'
```

但是， **CQL 强制要求更新和删除必须有 where 条件，并且 where 条件中包含 `objectId=xxx` ，也就是说你只能根据 objectId 加上其他条件来更新或者删除一条记录**。如果没有提供，将报错。

一个查询某个 class 下的 100 条数据：

```sql
select * from GameScore
```

等价于（以 Android 为例子）:

```java
AVQuery<AVObject> query = new AVQuery<AVObject>("GameScore");
List<AVObject> avObjects = query.find()
```

`select` 一个完整的语法形式类似这样：

```sql
select [查询字段列表，逗号隔开] from [class 名称]
   [where [条件列表]
   [limit [skip],limit
   [order by [排序字段列表] [asc |desc]]]]
```

一些小例子：

```sql
//查询结果只包含 name,score 以及内置字段(objectId,createdAt等)
select name,score from GameScore

//根据 name 查找
select * from GameScore where name='dennis'

//根据 name 和 score 同时查找
select * from GameScore where name is exists and score > 80 and score <= 100

//分页查找，从第 100 条开始向后查找 10 条数据
select * from GameScore limit 100,10

//根据 score 和 name 排序
select * from GameScore order by score,+name desc
```

删除一条数据：

```sql
delete from GameScore where objectId='558e20cbe4b060308e3eb36c'
```

## 插入

insert 语句的基本语法：

```sql
insert into table_name (列1, 列2,...) VALUES (值1, 值2,....)
```

例如：

```sql
insert into Player(name, age) values('LeanCloud', 2)
insert into GameScore(player, score) values(pointer('Player','player objectId'), 100)
```

上面例子里我们使用内置的 `pointer` 函数创建一个指向 Player 的 pointer，并存入 GameScore 表的 player 列。更多内置函数请参考[内置函数](#内置函数)一节。

insert 还支持多行插入：

```sql
insert into Player(name, age) values ('LeanCloud', 2), ('美味书签', 3)
```

values 后面接多个括号括起来的值列表即可。

insert 语句也支持占位符（推荐方式）:

```sql
insert into GameScore(name, score) values (?, ?)
```

## 更新

update 语句的基本语法：

```sql
update 表名称 set 列名称 = 新值 where 条件列表
```

例如：

```sql
update GameScore set score=90, name='leancloud' where objectId='558e20cbe4b060308e3eb36c'
```

如果涉及到 op 操作，例如只有当账户余额超过 100 元的时候，给账户扣款 50:

```sql
update Account set account=op('Decrement', {'amount': 50}) where objectId='558e20cbe4b060308e3eb36c' and amount>=100
```

如果当前账户少于 100 元，这个操作将失败，并返回错误信息。

CQL 的 update 和 delete 语句都强制要求 where 条件中出现 objectId=xxx 的条件，也就是只能更新或者删除特定 objectId 的文档。

op 操作提供了 `op(名称, JSON 参数)` 的内置函数，来方便用户使用，例如往一个 relation 列添加一个对象：

```sql
update _User set friends=op('AddRelation', {'objects': [pointer('_User','user id')]}) where objectId='558e20cbe4b060308e3eb36c'
```

update 语句同样也支持占位符（推荐方式）:

```sql
update GameScore set name=?,score=? where objectId=?
```

## 查询

select 语句中 `where` 之后的查询条件基本跟 SQL 语法相似，比如支持 `or` 和 `and` 的复合查询，支持 `=`、`!=`、`<`、`<` 等比较运算符，支持子查询、in 查询等。详细解释如下。

### 基本查询

查询指定信息的对象，用 `=` 比较符：

```sql
select * from GameScore where name='dennis'
```

查询不等于指定信息的对象：

```sql
select * from GameScore where name!='dennis'
```

也可以用 `<>` 运算符来表示不等于。

比较日期，使用 `date` 函数来转换，比如查询特定时间之前创建的对象：

```sql
select * from GameScore where createdAt < date('2011-08-20T02:06:57.931Z')
```

date 函数接收的日期格式必须是 `2011-08-20T02:06:57.931Z` 的 UTC 时间。更多内置函数请看最后的 [内置函数](#内置函数) 部分。

一些常见的查询运算符：

<table>
  <tr><th>Key</th><th>Operation</th></tr>
  <tr><td><</td><td>小于</td></tr>
  <tr><td><=</td><td>小于等于</td></tr>
  <tr><td>></td><td>大于</td></tr>
  <tr><td>>=</td><td>大于等于</td></tr>
  <tr><td> != 或者 <></td><td>不等于</td></tr>
  <tr><td>[not] like</td><td>模糊查询</td></tr>
  <tr><td>[not] regexp</td><td>正则匹配</td></tr>
  <tr><td>[not] in(子查询或者数组)</td><td>包含或者不包含</td></tr>
  <tr><td>is [not] exists</td><td>这个 Key 有值或者不存在值</td></tr>
</table>

比较运算符可以用在日期、字符串、数字甚至对象上。

### 模糊查询

模糊查询可以使用 `like`，比如查询名字以 dennis 开头的对象。

```sql
select * from GameScore where name like 'dennis%'
```

`%` 表示模糊匹配的位置占位符。

like 本质上是转成 `regexp` 的正则匹配查询，因此上面的例子还可以写成：

```sql
select * from GameScore where name regexp 'dennis.*'
```

否定形式，查询名字不以 dennis 开头的对象：

```sql
select * from GameScore where name not like 'dennis%'
或者
select * from GameScore where name not regexp 'dennis.*'
```
正则匹配的效率一般，类似这种全文搜索请求，我们都推荐采用 [应用内全文搜索](app_search_guide.html)。

### 值是否存在查询

只返回 `level` 字段值存在的对象：

```sql
select * from GameScore where level is exists
```

反之，使用 `is not exists`。请注意，值是否存在，跟 null 或者空字符串是不同的，不存在表示这个列在对象里都不存在。


### 数组查询

假设 `scores` 字段是一个数组，我们想查询分数里有 100 的成绩列表：

```sql
select * from GameScore where scores=100
```

如果想查找分数**只有**两个 100 分的成绩：

```sql
select * from GameScore where scores all (100,100)
```

`all` 表示数组完全匹配。

查询数组长度为特定值的对象：

```
select * from GameScore where size(scores) = 2
```

数组长度查询仅支持等于号，不支持大小和小于等其他比较运算符。

### 子查询

使用 `in` 来做子查询，后面跟的可以是一个列表，例如查询名字是 dennis、catty 和 green 三个玩家的成绩：

```sql
select * from GameScore where name in ('dennis','catty','green')
```

当然，如果想查询的不在列表里，那可以使用 `not in`:

```sql
select * from GameScore where name not in ('dennis','catty','green')
```

in 后面还可以是一个子查询，比如查询玩家信息，并且成绩大于 80 分的：

```sql
select * from Player where name in (select name from GameScore where score>80)
```

注意子查询必须指定查询的字段名称是 `select name`。

子查询另一种常见形式是使用 `=` 或 `!=` 跟一条查询语句：

```sql
select * from Player where name =(select name from GameScore where score>80)
select * from Player where name !=(select name from GameScore where score<=80)
```

**注意：子查询的语句也受上限 1000 条记录的限制。**


### 地理位置信息查询

可以通过 CQL 进行地理位置信息查询，比如我想查询自己附近的玩家（从近到远排序），假设 `location` 字段是 GeoPoint 类型：

```sql
select * from Player where location near [116.4, 39.9]
```

其中 `[116.4, 39.9]` 是经纬度坐标。也可以使用 `geopoint` 函数来创建：

```sql
select * from Player where location near geopoint(116.4, 39.9)
```

只有在地理位置信息查询里才可以使用 `[longitude, latitude]` 这样的语法。在其他查询里将被作为数组类型。

为了限定搜索的最大距离，还可以使用 `max [距离]` 来限定，比如限定在 1 公里内：

```sql
select * from Player where location near geopoint(116.4, 39.9) max 1 km
```

距离长度的其他单位包括 `miles`（英里）和 `radians`（弧度），如果不提供明确的单位，默认是**弧度**。

通过 `min [距离]` 还可以限定最小距离：

```sql
select * from Player where location near geopoint(116.4, 39.9) min 0.5 km
```

`min` 和 `max` 同时限定的时候，`min` 必须出现在 `max` 之前：

```sql
select * from Player where location near geopoint(116.4, 39.9) min 0.5 km max 1 km
```

如果想查询某个矩形框内的对象，可以使用 `within [西南坐标] and [东北坐标]` 的语法：

```sql
select * from Player where location within [116.33, 39.97] and [116.37, 39.99]
```


### 查询个数(count)

使用 `count` 查询来返回符合查询条件的数目，比如查询整张表的记录数：

```sql
select count(*) from GameScore
```

`count` 不支持 `distinct` 等语法。仅限 `count(*)` 和 `count(objectId)`。

查询分数大于 60 并且小于等于 80 的成绩数目：

```sql
select count(*) from GameScore where score>60 and score<=80
```

查询个数的同时可以返回对象：

```sql
select count(*),* from GameScore
```

也可以返回特定的字段：

```sql
select count(*),name from GameScore
```

### 关系查询

有几种方式来查询对象之间的关系数据。如果你想获取对象，而这个对象的一个字段对应了另一个对象， 你可以用一个 where 查询，自己构造一个 Pointer，和其他数据类型一样。

举例说，如果每一个 Comment 有一个 Post 对象在它的 post 字段上（Pointer 类型），你可以对一个 post 取得所有 comment：

```sql
select * from Comment where post=pointer('Post','51e3a359e4b015ead4d95ddc')
```

`pointer` 函数接收 className 和 objectId。

如果你想获取对象, 这个对象的一个字段指向的对象（必须是 Pointer）是符合另一个查询的， 你可以使用 in 查询。注意默认的 limit 是 100 而且最大的 limit 是 1000，这个限制同样适用于内部的查询，所以对于较大的数据集你可能需要细心地构建查询来获得期望的行为。举例说，假设你有一个 Post 类和一个 Comment 类，每个 Comment 都有一个指向它的 Post 的 Pointer，你可以找到对于有图片的 Post 的 Comment：

```sql
select * from Comment where post in (select * from Post where image is exists)
```

如果 Post 下面有一个 key 是 Relation 类型，并且叫做 likes，存储了喜欢这个 Post 的 User。你可以找到这些 user，他们都 like 过同一个指定的 post：

```sql
select * from _User where related likes to pointer('Post', '51e3a359e4b015ead4d95ddc')
```

基本的查询形式是 `releated <key> to <pointer>`。


如果某个字段是 Pointer，默认查询的时候，只会返回 `{__type: 'Pointer', objectId: 'objectId', className:'Post'}` 这些基本信息，如果希望同时将这个对象的其他信息查询下来，可以使用 include，比如查询 Comment 同时将 Post 带下来：

```sql
select include post, * from Comment
```

在 select 中采用 `include <key>` 就可以将某个 Pointer 字段关联查询出来。多个字段要多次 include：

```sql
select include post,include author from Comment
```

同样，还可以支持嵌套的 include 查询，比如 Post 里还有一个 Pointer 指向 Category：

```sql
select include post.category,* from Comment
```

### 复合查询

你可以使用 `and` 和 `or` 来做符合查询，例如查询分数在 80 到 100 之间，可以用 `and`：

```sql
select * from GameScore where score>80 and score<=100
```

再加个条件，或者分数为 0 分的：

```sql
select * from GameScore where score>80 and score<=100 or score=0
```

`and` 的优先级高于 `or`，因此上面的查询也可以用括号来明确地表示这种优先级：

```sql
select * from GameScore where (score>80 and score<=100) or score=0
```

## 限定返回值

通过 `limit` 语句来限定返回结果大小，比如限定返回 100 个：

```sql
select * from Comment limit 100
```

可以设定从第 m+1 个元素开始，例如从第 101 个元素（包含）开始往后取 10 个：

```sql
select * from Comment limit 100,10
```

这个形式跟 MySQL 是类似的。

### 占位符

查询条件和 limit 子句还支持占位符，也就是可以用问号 `?` 替代值，值的列表通过 SDK 提供的方法传入，具体请参考各 SDK 用法，例如：

```sql
select * from GameScore where name=? and score>? limit ?,?
```

占位符支持所有 LeanCloud 平台上的有效类型，复杂类型（如日期、GeoPoint）要求以 [REST API](./rest_api.html#数据类型) 文档说明的 JSON 格式来提供。

**我们推荐使用占位符的方式来使用 CQL，查询语句可以通过预编译被缓存起来，降低 CQL 的转换开销。**

### 排序

通过 `order` 语句来排序，`order` 语句只能出现在最后，不能在 `where` 和 `limit` 之前。

例如按照分数倒序排（分数高的前）：

```sql
select * from GameScore order by score desc
```

也可以写成：

```sql
select * from GameScore order by -score
```

加号表示升序，减号表示降序。

多个字段组合排序，例如分数高的前，名字相同的「更小」的在前（字母顺序）：

```sql
select * from GameScore order by -score,name
```

同样的语句可以写成：

```sql
select * from GameScore order by score,+name desc
或者
select * from GameScore order by -score,name asc
```

没有写上明确的加号或者减号的字段，将根据最后的 `desc` 或者 `asc` 来决定采用升序还是降序。

## 删除

delete 语句基本语法：

```sql
delete from 表名称 where objectId = ? [and 条件列表]
```

最简单的，根据 objectId 删除一个文档：

```sql
delete from GameScore where objectId=?
```

可见， delete 语句也支持占位符。

**删除的条件列表要求一定提供 objectId=xxx 的条件，也就是只能根据 objectId 来删除某个文档。**


## 内置函数

CQL 提供了一些内置函数来方便地创建 pointer、geopoint 等类型：

<table>
  <tr><th>Name</th><th>Operation</th></tr>
  <tr><td>date('YYYY-MM-DDTHH:MM:SS.MMMZ')</td><td>创建日期类型</td></tr>
  <tr><td>pointer(className, objectId)</td><td>创建 Pointer</td></tr>
  <tr><td>geopoint(经度, 纬度)</td><td>创建 GeoPoint</td></tr>
  <tr><td>file(objectId)</td><td>创建 file 类型</td></tr>
  <tr><td>base64(base64编码字符串)</td><td>创建 Bytes 类型</td></tr>
  <tr><td>current_timestamp()</td><td>创建当前日期</td></tr>
  <tr><td>op(op 名称, JSON 参数对象)</td><td>创建更新 op 对象</td></tr>
</table>

如果不使用这些函数，你也使用 [REST API 文档](./rest_api.html#数据类型) 定义的 JSON 对象来创建特定类型，例如 Pointer：

```sql
select * from Comment where post=
  {className:'Post', objectId:'51e3a334e4b0b3eb44adbe1a',__type:'Pointer'}
```

当然这样写就相对繁琐了。

## 性能和建议

CQL 最终还是转换成 [REST API](./rest_api.html) 里查询部分提到的各种 where 条件，因为多了一层转换，理论上会比直接使用 `where` 查询慢一点。并且 CQL 对长度有所限制，要求在 4096 字节以内。

此外，我们推荐查询语句都采用占位符的方式，使用占位符的查询语句将有机会被缓存复用，避免重复解释的开销。




