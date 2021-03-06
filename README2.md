
## 背景

    由于开发自己的博客网站需要实现评论功能，而网上第三方评论基本已经挂掉，留下来的只有gitment和一些必须要翻墙才能实现的评论服务。故而打算实现自己的评论服务应用在博客中

## 做法

    为了讲评论功能做的更加通用，取消作为一个组件引用于项目的形式，而是采用第三方服务的形式引进项目中。只要App接入了该服务，就可以为其提供评论功能。这种方式由于只是单纯的提供评论服务，故而，只需要提供简单的评论应有的新增，删除，获取等功能，其余的逻辑由使用该服务的App决定，比如权限管理，具体功能实现。

    总之，这里打算做一个服务器端的SDK，用于进行实现评论服务。并且先只做服务器评论，客户端评论以后再说。

## 鉴权

同其他第三方服务一样，本SDK的所有的功能都需要合法的授权。授权凭证的签算需要申请一对有效的Access Key和Secret Key。我们这里先写死。。。。23333

## 缓存

以后再说。。。

## 上传图片，文件

评论支持文件，图片。。。emmmmm

## 数据库设计

由于只是做一个单纯的评论服务，只涉及的到评论，对于权限管理，功能自定义，UI设计等不必要关心，这些都是接入本服务的App需要自己实现的。本服务只提供操作评论数据的接口。那么数据库可以这样：

数据库的设计的关键在于如何搞笑的对不同数据量的数据进行处理。也就是说关键在于如何处理同一个目标下不同数量级评论。

一个目标下评论的数量级：
    <=1000 简单的一个 单个评论文档没什么多说点
    <=100000 分块处理，一个块100条评论，数据结构如下
    >100000 三级分块 分部分处理，一个部分100块
```javascript
// 基本结构
{
    "id": 1223                  // 评论ID
    "userId":                   // 用户ID
    "targetId":                 // 评论对象ID
    "content":                  // 评论内容
    "createdTime":                     // 评论时间
    "updateTime":
    "extra": {
        "user": {
            "avatar":
            "nickname":
            "email":
            "ip":
        },
        "like":
        "unlike":
        "anouymous":
        "reported":             // 是否被举报  true已经被举报
        "reply": [SectionId:ChunkId:CommentId]               // 对于本评论的回复
    }
}
// chunk 100倍
{
    "targetId":
    "userId": 
    "chunkNum":
    "count":
    "comments":[Comment]
}
// chunk下由于不存在comment，所以回复功能会存在问题
// section 结构
{
    "targetId":
    "userId"
    "sectionNum":
    "count":
    "chuncks": [ChunkId]
}
```

只是简单的记录评论信息即可，其他信息可由第三方App提供。

## API 

#### 0. 通用以及注意事项

**App为通用部分**

**App组成部分如下：**

```javascript
{
    "AppKey": "",         // 应用Key
    "AppSecret": "",      // 应用secret
}
```

#### 0. 初始化

使用App对象来初始化

**`LFment.init(App)`**

#### 1. 发送评论

**`LFment.sendComment(Comment)`**

**Comment组成部分：**

```javascript
{
    "targetId": "",        // 目标ID        // 必须
    "userId": "",          // 必须     
    "content": "测试",      // 内容          // 必须
    "time": "1233223323",   // 发送时间      // 必须
    "extra"：             // 自定义信息 对照数据表结构
}
```

**注意**：匿名游客发送评论需要进行校验码校验。防止机器人。(额第三方应用自己做吧)

#### 2. 删除评论

**`LFment.deleteComment(commentId)`**

删除某个用户的所有评论

**`LFment.deleteUserComments(userId)`**

**PS**: 权限管理交由App处理

#### 3. 获取评论

获取单个评论

**`LFment.getCommentById(commentId)`**

获取目标评论

**`LFment.getCommentsByTarget(targetId, options)`**

没有options默认返回所有目标的评论

Options属性：

```javascript
{
    "count":    // 数目限制
    "group":    // 是否是按组获取。按组获取意味着返回的数据是将一个评论以及其相关评论一起返回
    "sort":     // 排序（可按照时间排序，也可按照热度排序）
}
```

获取用户评论

**`LFment.getCommentsByUser(userId, options)`**

Options属性：

```javascript
{
    "count":    // 数目限制
    "sort":     // 排序（可按照时间排序，也可按照热度排序）
}
```

#### 4. 设置extra部分的模型（范式）

用于自定义自己应用的额外属性的模式，类似于Mongoose的Schema

定义的常见行为同schema定义一样

**`LFment.defineExtra(schema)`**

默认模式：

```javascript
{
    "user": {
        "avatar": String
        "nickname": String
        "email": String
        "ip": String
    },
    "like": Number
    "unlike": Number
    "anouymous": {
        "type": Boolean,
        "default": true
    },
    "reported": Boolean            // 是否被举报  true已经被举报
    "reply": [ObjectId]               // 对于本评论的回复
}
```

`e.g:`
```javascript
const schema = {
    like: Number,
    unlike: Number
}
LFment.setExtraSchema(schema)
```

#### 5. 更改额外属性

**`LFment.setExtra(propName, value)`**

支持逐层往下查找，找到同名属性立即设置并且返回

<!-- #### 4. 点赞

点赞评论： **`LFment.like(commentId)`**

<!-- 点赞文章： **`LFment.Like(articleId)`** -->

<!-- #### 5. 踩 -->

<!-- 踩评论： **`LFment.unLike(commentId)`** -->

<!-- 踩文章： **`LFment.UnLike(articleId)`** -->

<!-- #### 6. 举报

**`LFment.report(commentId)`** -->

<!-- #### 10. 屏蔽用户

屏蔽方式有昵称，邮箱，ip。可设置屏蔽时间。支持正则。

**`LFment.shield(target)`**

target字段：

```javascript
{
    "email": "",                // 根据邮箱
    "ip": "",                   // 根据ip
    "nickname": "",             // 根据昵称屏蔽
    "startTime": "",            // 屏蔽开始时间
    "endTime": "",              // 屏蔽结束时间
    "timeLength": ""            // 屏蔽时长， -1表示无限期
}
``` -->

<!-- **PS**:只有站主有这权限,因为会校验token -->

<!-- #### 11. 邮件订阅

订阅邮件，分为三种情况，一种是文章变化邮件通知，一种是评论变化邮件通知， 一种是和自己有关的评论变化邮件通知

**`LFment.subscribe(options)`**

options字段：

```javascript
{
    "article": false,           // 文章变化通知（这里只能知道文章是否删除等消息，无法知道文章是否更改）
    "aboutMe": false,           // 关于自己评论变化通知
    "comments": false           // 评论变化通知
}
``` -->

## 其他

采取分库的形式，统一管理不同业务方的数据。并且可以很方便的对其增值服务进行管理。比如数据库扩容，备份等操作。

问题： 一个目标有几十万几百万评论该怎么存储。 解决方法： 这种算在增值服务里面（量级服务），采取外键+分块的形式来进行。100条(chunk)一个分块。10000条一个大分块（如果需要的话， section）

## TODO： 

- [] 针对不同app业务方数据库权限管理
- [] 客户端简单实现
- [] 客户端评论凭证
<!-- - [] 三方登录 -->