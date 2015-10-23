# JavaScript 指南

如果你还没有设置你的项目，请查看文档 [快速入门](./start.html)。

如果你希望从项目中学习，请前往 [JavaScript SDK Demos](https://github.com/leancloud/leancloud-demos#javascript)。

## 简介

LeanCloud 提供了一个移动 app 的完整后端解决方案，我们的目标是帮助开发团队完全消除写后端代码和维护服务器的负担。

请在阅读本文档的同时，对照查看 [JavaScript API文档](./api/javascript/)，本指南并没有完全覆盖所有的 API 调用。

该 JavaScript SDK 也可在 Nodejs 等服务器端环境运行，可以使用 LeanEngine 来搭建服务器端，可以参考[相关文档](https://leancloud.cn/docs/leanengine_guide-node.html)。

## 快速入门

建议你在阅读本文档之前，阅读我们提供的[快速入门](./start.html)文档，获取 LeanCloud 使用配置和第一印象。

### 应用

LeanCloud 的每一个账户都可以创建多个应用，每个应用都有自己的 appId 和客户端密钥，客户端代码应该使用它们来初始化 SDK。同时，为了便于开发调试，LeanCloud 也为每一个应用提供了测试和生产两套环境。

### 贡献

你可以通过 GitHub 报告 bug 或者提出建议。如果觉得这个文档写的不够好，也可以帮助我们完善。

SDK 仓库地址：[https://github.com/leancloud/javascript-sdk](https://github.com/leancloud/javascript-sdk)，相关 [change log](https://github.com/leancloud/javascript-sdk/blob/master/changelog.md)

本文档仓库地址：[https://github.com/leancloud/docs](https://github.com/leancloud/docs)

### npm 安装

可以通过 npm 安装。

该 JavaScript SDK 也可在 Nodejs 等服务器端环境运行，可以使用 LeanEngine 来搭建服务器端，可以参考[相关文档](https://leancloud.cn/docs/leanengine_guide-node.html)。

```
$ npm install avoscloud-sdk
```

### bower 安装

也支持 bower 安装

```
$ bower install leancloud-javascript-sdk
```

### CDN 加速

```
<script src="https://cdn1.lncld.net/static/js/av-mini-{版本号}.js"></script>
// 或者你只是用最核心的存储、推送等功能，可以使用精简版的core.js
<script src="https://cdn1.lncld.net/static/js/av-core-mini-{版本号}.js"></script>
```

### Web 安全

如果在前端使用 JavaScript SDK，当你打算正式发布的时候，请务必配置 `Web 安全域名`。配置方式：进入对应的 app，然后选择 `设置`——`安全中心`——`Web 安全域名`。这样就可以防止其他人，通过外网其他地址盗用你的服务器资源。

具体安全相关内容可以仔细阅读文档 [数据和安全](data_security.html) 。

## 对象

### AV.Object

LeanCloud 的数据存储服务是建立在对象 --- `AV.Object` 基础上的，每个 `AV.Object` 包含若干属性值对（key-value，也称「键值对」），属性值是与 JSON 格式兼容的数据。你不需要预先指定每个 `AV.Object` 包含哪些属性，每个属性的数据类型是什么，只要直接设定属性值对即可，你还可以随时增加新的属性。

假如我们要实现一个类似于微博的社交 app，主要有三类数据：账户、帖子、评论。以微博帖子为例，我们可以建立一个类名为 `Post` 的 `AV.Object` 对象，包含下面几个属性：

```javascript
content: "每个 JavaScript 程序员必备的 8 个开发工具", pubUser: "LeanCloud官方客服", pubTimestamp: 1435541999
```

属性名必须是由字母、数字组成的字符串，属性值可以是字符串、数字、布尔值、JSON 数组，甚至可以嵌套其他 `AV.Object`。

**注意：以下为系统保留字段，不能作为属性名来使用。**

```
acl             error            pendingKeys
ACL             fetchWhenSave    running
className       id               updatedAt
code            isDataReady      uuid
createdAt       keyValues
description     objectId
```

每一个 `AV.Object` 都是一个特定子类的实例，子类名可以来区分各种不同的数据。我们建议将类和属性名分别按照 `NameYourClassesLikeThis` 和 `nameYourKeysLikeThis` 这样的惯例来命名，即区分第一个字母的大小写，这样可以提高代码的可读性和可维护性。

为了建立一个新的子类，你可以使用 `AV.Object.extend` 方法。如果你熟悉 Backbone.Model 的话，你已经明白如何使用 AV.Object 了，它本身就是设计来让两者可以相互替换的。

**注意**：`AV.Object.extend` 产生的对象需要作为全局变量保存，因为每调用
一次，就会产生一个新的类的实例，并且和之前创建的实例形成一个链表。
如果你的应用时不时出现 `Maximum call stack size exceeded` 错误，请
确认是否误用了该方法。


```javascript
// 创建AV.Object子类.
// 该语句应该只声明一次
var Post = AV.Object.extend("Post");

// 创建该类的一个实例
var post = new Post();

// 你也可以用 Backbone 的语法.
var Post = AV.Object.extend({
  className: "Post"
});
```

你可以为 AV.Object 的子类添加任意方法：

```javascript
// AV.Object 的稍复杂一点的子类
var Post = AV.Object.extend("Post", {
  //实例方法
  pubUser: function() {
    return this.get("pubUser");
  },
  content: function(){
    return this.get('content');
  }
}, {
  //类方法
  spawn: function(username) {
    var post = new Post();
    post.set("pubUser", username);
    return post;
  }
});

var post = Post.spawn("LeanCloud官方客服");
alert(post.pubUser());
```

创建实例还可以通过 `AV.Object.new` 方法，子类对象也可以：

```javascript
//AV.Object
var post1 = AV.Object.new('Post');
//子类
var post2 = Post.new({pubUser: "LeanCloud官方客服", content:"每个 JavaScript 程序员必备的 8 个开发工具"});
```

在使用 [uglify](https://github.com/mishoo/UglifyJS) 做代码压缩的时候，推荐采用上述方式创建实例，否则可能遇到压缩后语法错误。

### 保存对象

假如你想要在 LeanCloud 上保存 `Post` 实例，方法和 Backbone.Model 差不多，就用
 `save` 就可以了。

这里要注意，我们每个存储条目的 `objectId` 是服务器端自动生成的唯一 id（非简单的自增逻辑生成），所以 `objectId` 是不可修改的。如果你有自定义 id 的需求，可以自己建立一个字段，逻辑上作为你的自定义 id。

```javascript
var post = new Post();
post.set("content", "每个 JavaScript 程序员必备的8个开发工具");
post.set("pubUser", "LeanCloud官方客服");
post.set("pubTimestamp", 1435541999);
post.save(null, {
  success: function(post) {
    // 成功保存之后，执行其他逻辑.
    alert('New object created with objectId: ' + post.id);
  },
  error: function(post, error) {
    // 失败之后执行其他逻辑
    // error 是 AV.Error 的实例，包含有错误码和描述信息.
    alert('Failed to create new object, with error message: ' + error.message);
  }
});
```

在代码运行以后，你可能会对后端发生了什么有兴趣，为了确认数据被保存了，你可以
在 LeanCloud 的[数据管理](/data.html?appid={{appid}})上查看你的数据。你大致可以看到如下的内容：

```javascript
objectId: "558e20cbe4b060308e3eb36c", content: "每个 JavaScript 程序员必备的 8 个开发工具", pubUser: "LeanCloud官方客服", pubTimestamp: 1435541999,
createdAt:"2015-06-29 09:39:35", updatedAt:"2015-06-29 09:39:35"
```

这里有 3 点需要注意的地方:

* 在运行以上代码时，如果云端（LeanCloud 的服务器，以下简称云端）不存在 `Post` 数据表，那么 LeanCloud 将根据你第一次（也就是运行的以上代码）保存的 `Post` 对象来创建数据表，并且插入相应数据。
* 如果云端的这个应用中已经存在名为 `Post` 的数据表，而且也包括 `content`、`pubUser`、`pubTimestamp` 等属性，新加入属性的值的数据类型要和创建该属性时一致，否则保存数据将失败。
* 每个 `AV.Object` 对象有几个保存元数据的属性是不需要开发者指定的，包括 `objectId` 是每个成功保存的对象的唯一标识符。`createdAt` 和 `updatedAt` 是每个对象在服务器上创建和最后修改的时间。这些属性的创建和更新是由系统自动完成的，请不要在代码里使用这些属性来保存数据。在保存之前
AV.Object 中都不会有这些字段。

如果愿意，你也可以在调用 `save` 时直接设定属性值.

```javascript
var post = new Post();

post.save({
  content: "每个 JavaScript 程序员必备的 8 个开发工具",
  pubUser: "LeanCloud官方客服",
  pubTimestamp: 1435541999
}, {
  success: function(post) {
    // 实例已经成功保存.
  },
  error: function(post, error) {
    // 失败了.
  }
});
```

保存后，可以通过 `post.createdAt` 获取对象的创建时间，通过 `post.updatedAt` 获取对象的更新时间，两者都是 `Date` 对象。

### 检索对象

在 LeanCloud 中保存数据是很简单的，获取数据也非常容易。如果事先知道 `objectId` 的话,你可以用一个 `AV.Query` 提取出整个 `AV.Object`:

```javascript
var query = new AV.Query(Post);
query.get("558e20cbe4b060308e3eb36c", {
  success: function(post) {
    // 成功获得实例
  },
  error: function(object, error) {
    // 失败了.
  }
});
```

为了获得 `AV.Object` 的属性值, 应该使用 `get` 方法.

```javascript
var content = post.get("content");
var username = post.get("pubUser");
var pubTimestamp = post.get("pubTimestamp");
```

### 更新对象

更新一个对象也是非常简单的。首先需要获取到要更新的 `AV.Object` 对象，然后进行修改值后保存数据。例如：

```javascript
// 可以先查询出要修改的那条存储
var Post = AV.Object.extend("Post");
var query = new AV.Query(Post);

// 这个 id 是要修改条目的 objectId，你在生成这个实例并成功保存时可以获取到，请看前面的文档
query.get('558e20cbe4b060308e3eb36c', {
    success: function(post) {
      // 成功，回调中可以取得这个 Post 对象的一个实例，然后就可以修改它了
      post.set('content', '每个 JavaScript 程序员必备的 8 个开发工具: http://buzzorange.com/techorange/2015/03/03/9-javascript-ide-editor/');
      post.save();
    },
    error: function(object, error) {
      // 失败了.
      console.log(object);
    }
});
```

LeanCloud 自动查找哪些数据被改动了，所以 SDK 这边只有 "dirty" 的字段会被发送到 LeanCloud 云端，
你不需要担心需要剔除你不想更新的数据。

**请注意，LeanCloud 上的更新对象都是针对单个对象，获得对象的 objectId 主键才可以去更新对象。服务端判断一个对象是新增还是更新，是根据有没有 objectId 来决定的。**

上面的例子是先查询出对象，然后在 get 的 callback 里更新对象。

如果你已经知道了 objectId（例如从查询后的列表页进入一个详情页面，传入了 objectId），想要修改一个对象，也可以采用类似下面的代码来更新对象属性：

```javascript
// 知道 objectId，创建 AVObject
var post = AV.Object.createWithoutData('Post', '558e20cbe4b060308e3eb36c');
// 更改属性
post.set('content', '每个 JavaScript 程序员必备的 8 个开发工具: http://buzzorange.com/techorange/2015/03/03/9-javascript-ide-editor/');
// 保存
post.save();
```

#### fetchWhenSave

如果想在更新成功后获取最新的对象属性，请设置`fetchWhenSave`:

```javascript
//假设post是已经存在的对象
post.fetchWhenSave(true);
post.set("content", "每个 JavaScript 程序员必备的 8 个开发工具: http://buzzorange.com/techorange/2015/03/03/9-javascript-ide-editor/");
post.set("pubUser", "LeanCloud官方客服");
post.save();
```

这个方法在对象被并发修改的时候特别有用，可以得到更新后对象的最新状态。例如维护一个计数器的场景，每次 `save` 后得到最新的计数。

#### 计数器

许多应用都需要实现计数器功能 -- 比如一条微博，我们需要记录有多少人喜欢或者转发了它。但可能很多次喜欢都是同时发生的，如果在每个客户端都直接把它们读到的计数值增加之后再写回去，那么极容易引发冲突和覆盖，导致最终结果不准。这时候怎么办？LeanCloud 提供了便捷的原子操作来实现计数器：

```javascript
post.increment("upvotes");
post.save();
```

另外，通过使用 `increment(key, amount)` 方法，你可以自行定义增减的幅度（amount 缺省值为 1）。

#### 数组

为了帮你存储数组类数据，LeanCloud 提供了三种操作让你可以原子地改动一个数组的值（当然，他们都需要一个给定的 key）:

- add: 在一个数组的末尾加入一个给定的对象
- addUnique: 只会把原本不存在的对象加入数组，所以加入的位置没有保证
- remove: 在一个数组中删除所有指定的实例

比如，我们想在一条微博的属性 "tags" 中加入多个属性值:

```javascript
post.addUnique("tags", "Frontend");
post.addUnique("tags", "JavaScript");
post.save();
```

### 删除对象

调用如下代码会在 LeanCloud 中删除一个实例:

```javascript
myObject.destroy({
  success: function(myObject) {
    // 对象的实例已经被删除了.
  },
  error: function(myObject, error) {
    // 出错了.
  }
});
```

你可以使用 `unset` 方法来删除一个实例中的单个属性：

```javascript
// 这样可以删掉 pubTimestamp 属性，本来也不必要，我们完全可以用 createdAt 属性代替它。
post.unset("pubTimestamp");

// 写回 LeanCloud
post.save();
```

批量删除一批对象可以这样：

```javascript
AV.Object.destroyAll(objects);
```

其中 objects 是一个对象集合，且其中的每个对象的 className 必须一样。


### 关系数据

对象可能与别的对象有关系。比如对于微博来说，一条微博信息(`Post` 对象)可能有很
多评论(`Comment` 对象)。LeanCloud 支持各种关系，包括一对一、一对多和多对多。

#### 一对一关系和一对多关系

一对一关系和一对多关系都可以通过在一个 AV.Object 内保存另一个对象的实例来实现。
比如,每一个 Comment 都对应了一个 Post，创建一个带一条 Comment 的 Post,
你可以这样写:

```javascript
// Declare the types.
// 该语句应该只声明一次
var Post = AV.Object.extend("Post");
var Comment = AV.Object.extend("Comment");

// Create the post
var myPost = new Post();
myPost.set("content", "作为一个程序员，你认为回家以后要不要继续写代码？");

// Create the comment
var myComment = new Comment();
myComment.set("content", "我若是写代码，进入状态之后最好不要停。下不下班已经不重要了，那种感觉最重要。");

// 将 myPost 当成属性值加入 myComment
myComment.set("post", myPost);

// 这会将 myPost 和 myComment 一起保存起来
myComment.save();
```

LeanCloud 内部会自动处理，调用 `Comment` 的 `save` 方法就可以同时保存两个新对象。

如果是现有对象想要关联到新对象，你同样可以通过**只用它们的 objectId** 来连接彼此。
请注意，不能直接像上面的例子那样将现有对象设置进去，而是必须 `new` 一个新对象并只设置 `objectId` 属性：

```javascript
var post = AV.Object.createWithoutData("Post", "5590cdfde4b00f7adb5860c8");
myComment.set("post", post);
```

或者：

```javascript
var post = new Post();
post.id = '5590cdfde4b00f7adb5860c8';
myComment.set("post", post);
```

默认情况下，当获取一个对象时，关联的 AV.Object 不会被获取到，这些对象的值不能访问，除非像下面这样获取它们：

```javascript
var post = fetchedComment.get("post");
post.fetch({
  success: function(post) {
    var content = post.get("content");
  }
});
```

#### 多对多关系

多对多关系是通过 `AV.Relation` 来建模的。这样很像在一个 key 中存储一个 AV.Object 数组。但是区别之处在于，在获取附加属性的时候，`AV.Relation` 不需要同步获取关联的所有 `AV.Object` 实例。这使得 `AV.Relation` 比数组的方式可以支持更多实例，读取方式也更加灵活。例如，一个 `User` 可以喜欢很多 `Post`。这种情况下，就可以用 `relation` 方法保存一个用户喜欢的所有 `Post` 集合。为了新增一个喜欢的 `Post`，你可以这样做：

```javascript
var user = AV.User.current();
var relation = user.relation("likes");
relation.add(post);
user.save();
```

值得一提的是，Relation 会自动去重。所以，你不用去担心用户会重复 like 同一篇 post。

你可以从一个 AV.Relation 中删除一个 post：

```javascript
relation.remove(post);
user.save();
```

你可以在用 save 方法保存前多次调用 add 和 remove 方法：

```javascript
relation.remove(post1);
relation.remove(post2);
user.save();
```

你还可以传入一个 AV.Object 数组来做 add 和 remove：

```javascript
relation.add([post1, post2, post3]);
user.save();
```

默认情况下，`relation` 关联的对象不会被同步获取到，你可以通过使用 `query` 方法返回的 AV.Query
 对象来获取 AV.Object 的列表，例如：

```javascript
relation.query().find({
  success: function(list) {
    // list 里包含当前用户喜欢的所有微博.
  }
});
```

如果你仅仅要一个 `Post` 的子集，你可以在 `AV.Query` 中加入更多的条件:

```javascript
var query = relation.query();
query.skip(10);
query.limit(10);
query.find({
  success:function(list) {
    // list 里包含当前用户喜欢的部分微博.
  }
});
```

`relation` 返回的 `AV.Relation` 如果没有做过任何保存或者移除的操作，那么可能没有设置 `targetClassName`，在查询之前最好需要明确设置下：

```javascript
relation.targetClassName = 'Post';
var query = relation.query();
...使用 query 查询 Post...
```

你可以在接下来关于 `AV.Query` 的章节中看到更详细的内容。一个 `AV.Relation` 的行为很像一个 `AV.Object` 数组，所以任何在数组可做的查询操作，也都可以作用在 `AV.Relation` 上。

如果你知道 post，想反向查询 user，可以通过 `AV.Relation.reverseQuery` 方法：

```javascript
var query = AV.Relation.reverseQuery('_User', 'likes', post);
query.find({
  success:function(users) {
   // users 是表示喜欢这个 post 的用户列表。
  }
});
```

### 数据类型

到现在为止我们使用了 String、Number 和 AV.Object 类型，LeanCloud 同样支持 JavaScript 的 Date 和 null 类型。

你可以用一个 AV.Object 中嵌套 JavaScript 对象和数组来表述更加结构化的数据。

一些例子:

```javascript
// 该语句应该只声明一次
var TestObject = AV.Object.extend("DataTypeTest");

var number = 2014;
var string = "famous film name is " + number;
var date = new Date();
var array = [string, number];
var object = { number: number, string: string };

var testObject = new TestObject();
testObject.set("testNumber", number);
testObject.set("testString", string);
testObject.set("testDate", date);
testObject.set("testArray", array);
testObject.set("testObject", object);
testObject.set("testNull", null);
testObject.save();
```

AV.Object 实例的大小不应该超过 128 KB，如果需要存储较大的文件类型如图像、文件、音乐，可以使用 `AV.File` 对象来存储，具体使用方法可见 [AV.File 指南部分](#文件)。关于处理数据的更多信息，可查看开发指南的数据安全部分。

## 查询

### 基础查询

在很多情况下，`get` 方法无法检索到符合你要求的数据，`AV.Query` 提供一些
其他的方式来进行更复杂的查询。

通常的模式是创建一个 AV.Query 对象，加入一些条件，然后用 `find` 方法返
回一个满足条件的 AV.Object 数组。例如，查询指定人员的微博信息，使用 `equalTo` 方法来添加条件值：

```javascript
var query = new AV.Query(Post);
query.equalTo("pubUser", "LeanCloud官方客服");
query.find({
  success: function(results) {
    alert("Successfully retrieved " + results.length + " posts.");
    // 处理返回的结果数据
    for (var i = 0; i < results.length; i++) {
      var object = results[i];
      alert(object.id + ' - ' + object.get('content'));
    }
  },
  error: function(error) {
    alert("Error: " + error.code + " " + error.message);
  }
});
```

### 查询条件

有几种方式来设置查询条件。
你可以用 notEqual 方法和一个特定的值来过滤不符合要求的对象:

```javascript
query.notEqualTo("pubUser", "LeanCloud官方客服");
```

你可以给定更多的条件，只有满足所有条件的对象才会作为结果返回。换句话说，
就像是 AND 查询.

```javascript
query.notEqualTo("pubUser", "LeanCloud官方客服");
query.greaterThan("createdAt", new Date("2015-06-26 18:37:09"));
```

你可以用设定 `limit` 的方法来限定返回的结果数，默认的返回结果数是 100，但是任
何 1 到 1000 之间的数值都是合法的，在 0 到 1000 范围之外的都强制转成默认的 100。

```javascript
query.limit(10); // 最多返回 10 条结果
```

如果你只想要一个结果，一个更加方便的方法可能是使用 `first`，而不是 `find` 方法.

```javascript
var query = new AV.Query(Post);
query.equalTo("pubUser", "LeanCloud官方客服");
query.first({
  success: function(object) {
    // LeanCloud官方客服的第一条微博.
  },
  error: function(error) {
    alert("Error: " + error.code + " " + error.message);
  }
});
```

你可以用 `skip` 跳过前面的结果，这可能对于分页很有用。

```javascript
query.skip(10); // 跳过前 10 条结果
```

对于可以排序的类型，比如 number 和 string，你可以控制返回结果的顺序：

```javascript
// 升序
query.ascending("pubUser");

// 降序
query.descending("pubTimestamp");
```

对于可以排序的类型，你同样可以在查询中进行比较：

```javascript
// 认证级别 < 4
query.lessThan("pubUserCertificate", 4);

// 认证级别 <= 4
query.lessThanOrEqualTo("pubUserCertificate", 4);

// 认证级别 > 4
query.greaterThan("pubUserCertificate", 4);

// 认证级别 >= 4
query.greaterThanOrEqualTo("pubUserCertificate", 4);
```

如果想让返回的对象的某个属性匹配多个值，你可以使用 `containedIn`，提供一个数组就可以了。这样通常可以用单个的查询来获取多个结果，比如
你要查询 「LeanCloud官方客服」，「LeanCloud江宏」，「滚滚艾买提」 三个账号的微博时，可以使用 `containedIn`（类似 SQL 中的 in 查询）方法来实现：

```javascript
query.containedIn("pubUser",
                  ["LeanCloud官方客服", "LeanCloud江宏", "滚滚艾买提"]);
```

相反地，你可以使用 `notContainedIn` 方法来查询在集合之外的目标对象。

如果你想要查询含有某一特定属性的对象，你可以使用 `exists`。相对地，如果你想获
取没有这一特定属性的对象，你可以使用 `doesNotExist`：

```javascript
// 查找含有 pubTimestamp 属性的微博
query.exists("pubTimestamp");

// 查找不含有 pubTimestamp 属性的微博
query.doesNotExist("pubTimestamp");
```

你可以使用 `matchesKeyInQuery` 方法来进行嵌套的子查询。例如，在微博这类应用中有三类数据：一个类是微博帖子信息（Post），另一个类是用户账户信息（AVUser），还有一个类是用户之间互相关注的信息（UserFollow），要找出当前用户关注的人发布的微博，则：

```javascript
// 该语句应该只声明一次
var UserFollow = AV.Object.extend("UserFollow");
var Post ＝ AV.Object.extend("Post");

// 先找到当前登录用户关注的用户列表
var userQuery = new AV.Query(UserFollow);
userQuery.equalTo("follower", AV.User.current());

// 找到这些被关注者发布的微博
var postQuery = new AV.Query(Post);
postQuery.matchesKeyInQuery("author", "followee", userQuery);
postQuery.find({
  success: function(results) {
    // 得到当前用户关注的人发布的微博
  }
});
```

相反，要从一个查询中获取一组对象，该对象的一个键值，与另一个对象的键值并不匹配，可以使用 `doesNotMatchKeyInQuery` 。
例如，找出当前用户没有关注的人发布的微博：

```javascript
var postQuery = new AV.Query(Post);
postQuery.doesNotMatchKeyInQuery("author", "followee", userQuery);
postQuery.find({
  success: function(results) {
    // 得到非当前用户关注的人发布的微博
  }
});
```

你可以用 `select` 和一个 keys 的列表来限定返回的字段，为了获得只包含 pubUser 和 content 字段的微博（包括内置字段，如 objectId、createdAt、updatedAt）:

```javascript
var query = new AV.Query(Post);
query.select("pubUser", "content");
query.find().then(function(results) {
  // each of results will only have the selected fields available.
});
```

剩下的字段可以之后用返回的对象的 `fetch` 方法来获取：

```javascript
query.first().then(function(result) {
  // 这里只会返回指定的属性，再次获取全部属性.
  return result.fetch();
}).then(function(result) {
  // 这里会返回所有属性.
});
```

### 对数组值做查询

对于属性值是数组的情况，你可以这样查询数组的值中有 2 的实例列表：

```javascript
query.equalTo("arrayKey", 2);
```

你同样可以用下面的方式找到属性值中同时包含元素 2,3,4 的实例列表：

```javascript
query.containsAll("arrayKey", [2, 3, 4]);
```

此外，你还可以根据数组长度来查询，比如查询 `arrayKey` 的长度为 3 的对象列表：

```javascript
query.sizeEqualTo('arrayKey', 3);
```

### 对字符串类型做查询

使用 `startWith` 来限制属性值以一个特定的字符串开头，这和 MySQL 的 LIKE 操作
符很像，因为有索引所以对于大的数据集这个操作也是很高效的。

```javascript
// 找出名字以 "LeanCloud" 开头的账户的微博帖子
var query = new AV.Query(Post);
query.startsWith("pubUser", "LeanCloud");
```

### 关系查询

对于查询关系型数据来说有几种不同的方式，如果你想要获取的对象中有某个属性
包含一个特定的 AV.Object，你可以使用 `equalTo`，就像对于别的数据类型一
样。
例如，如果每条评论 `Comment` 的 `post` 字段都有一个 `Post` 微博对象，那么找出指定微博下的评论：

```javascript
// Assume AV.Object myPost was previously created.
var query = new AV.Query(Comment);
query.equalTo("post", myPost);
query.find({
  success: function(comments) {
    // comments 包含有 myPost 下的所有评论
  }
});
```

如果你想得到其字段中包含的子对象满足另一个查询的结果，你可以使用 `matchesQuery` 操作。
注意默认的结果条数限制 100 和最大值 1000 也同样适用于子查询，
所以对于大的数据集你可能需要小心构建你的查询，否则可能出现意料之外的状
况。例如，为了找到有图片的微博的评论，你可以:

```javascript
var innerQuery = new AV.Query(Post);
innerQuery.exists("image");
var query = new AV.Query(Comment);
query.matchesQuery("post", innerQuery);
query.find({
  success: function(comments) {
    // comments 包含有所有带图片微博的评论.
  }
});
```

如果你想要获取某字段中包含的子对象不满足指定查询的结果，你可以使用
`doesNotMatchQuery`。例如，为了找到针对不含图片的微博的评论，你可以这样：

```javascript
var innerQuery = new AV.Query(Post);
innerQuery.exists("image");
var query = new AV.Query(Comment);
query.doesNotMatchQuery("post", innerQuery);
query.find({
  success: function(comments) {
    // comments 包含所有不带图片微博的评论.
  }
});
```

你可以同样用 `objectId` 来做关系查询

```javascript
var post = new Post();
post.id = "5590cdfde4b00f7adb5860c8";
query.equalTo("post", post);
```

在某些情况下，你可能希望查询结果中包含多个相关联的其他数据类型。你可以使用 `include` 方
法。比如，假设你想获得最新的 10 个 comments，你可能想同时获取它们相关的 post 数据:

```javascript
var query = new AV.Query(Comment);

// 最新的在前面
query.descending("createdAt");

// 只要 10 条
query.limit(10);

// 包含了对应的微博信息
query.include("post");

query.find({
  success: function(comments) {
    // comments 包含最近的 10 条评论，每个 comment 实例的 "post" 都有全部的帖子信息。
    for (var i = 0; i < comments.length; i++) {
      // 这里不再需要网络访问.
      var post = comments[i].get("post");
    }
  }
});
```

你同样可以用点操作符来做多级查询，如果你想同时找到 comment 的 post 和相应 post
 的 author，你可以这样做:

```javascript
query.include(["post.author"]);
```

你可以多次使用 `include` 来构建一个有多个字段的查询，这项功能同样适用于
AV.Query 的 helper 函数，例如 `first` 和 `get` 等.

### 对象计数

如果你只是想查询满足一个 query 的结果集到底有多少对象，但是你不需要得到它们，你
可以使用 `count` 来取代 `find`。比如，计算一下某位用户一共发布了多少条微博：

```javascript
var query = new AV.Query(Post);
query.equalTo("pubUser", "LeanCloud官方客服");
query.count({
  success: function(count) {
    // 成功了
    alert("LeanCloud官方客服 发布了 " + count + " 条微博");
  },
  error: function(error) {
    // 失败了
  }
});
```

对于超过 1000 个对象的类来说，`count` 操作会被时间限制所约束，它们可能会一直
返回超时错误，或者只是返回一个近似正确的值。这样的话你应该更合理地规划你
程序的结构来避免这种情况。

### 组合查询

如果你想要查找满足一系列查询的对象，你可以使用 `AV.Query.or` 方法来构建
查询，这样得到的结果是所有查询的并集。例如，你想查询出企业官方账号和个人账号的微博，可以这样:

```javascript
var officialPosts = new AV.Query("Post");
officialPosts.greaterThan("pubUserCertificate", 2);

var individualPosts = new AV.Query("Post");
individualPosts.lessThan("pubUserCertificate", 2);

var mainQuery = AV.Query.or(officialPosts, individualPosts);
mainQuery.find({
  success: function(results) {
     // results 包含企业官方账号和个人账号发布的一些微博.
  },
  error: function(error) {
    // 失败了.
  }
});
```

你也可以对 AV.Query 加入更多的条件，如同 AND 查询一样，这样得到所有查询结果的交集。

请注意我们不会在组合查询的子查询中支持非过滤型的条件(比如:limit, skip, ascending/descending, include)。

### 删除查询结果

如果你想将查询出来的对象都删除，或者删除符合查询条件的所有对象，可以调用 `destroyAll` 方法：

```javascript
query.destroyAll({
   success: function(){
      // 成功删除 query 命中的所有实例.
   },
   error: function(err){
      // 失败了.
   }
   });
```

### CQL 查询语言

从 0.4.3 版本开始，我们允许使用类 SQL 语法的 CQL 查询语言来查询 LeanCloud 应用内的数据，例如：

```javascript
AV.Query.doCloudQuery('select * from Post', {
  success: function(result){
    //results 是查询返回的结果，AV.Object 列表
    var results = result.results;
    //do something with results...
  },
  error: function(error){
    //查询失败，查看 error
    console.dir(error);
  }
});
//查询认证等级大于 2 的账户的微博，并返回前100条。
AV.Query.doCloudQuery('select count(*),* from Post where pubUserCertificate>2', {
  success: function(result){
    //results 是查询返回的结果，AV.Object 列表
    var results = result.results;
    //count 表示符合查询条件的总记录数
    var count = result.count;
    //do something with results...
  },
  error: function(error){
    //查询失败，查看 error
    console.dir(error);
  }
});
```

doCloudQuery 回调中的 `result` 包含三个属性：

* results - 查询结果的`AV.Object`列表
* count - 如果使用了`select count(*)`的查询语法，返回符合查询条件的记录数目。
* className - 查询的 class name

CQL 语法请参考 [CQL 详细指南](./cql_guide.html)。

针对查询条件，我们推荐使用占位符的 CQL 语句来提升性能，占位符对应的值按照顺序组合起来作为第二个参数 `pvalues` 数组传入：

```javascript
//查询认证等级大于 3 的账户的微博，并返回前10条。
AV.Query.doCloudQuery('select count(*),* from Post where pubUserCertificate>? limit ?',[3,10],
 {
  success: function(result){
    //results 是查询返回的结果，AV.Object 列表
    var results = result.results;
    //count 表示符合查询条件的总记录数
    var count = result.count;
    //do something with results...
  },
  error: function(error){
    //查询失败，查看 error
    console.dir(error);
  }
});
```

`AV.Query.doCloudQuery` 返回的也是下面提到的 `AV.Promise` 对象。

## Promise

除了回调函数之外，每一个在 LeanCloud JavaScript SDK 中的异步方法都会返回一个
 `Promise`。使用 `Promise`，你的代码可以比原来的嵌套 callback 的方法看起来优雅得多。

### then 方法

每一个 Promise 都有一个叫 `then` 的方法，这个方法接受一对 callback。第一个
callback 在 promise 被解决(`resolved`，也就是正常运行)的时候调用，第二个会在 promise 被拒绝(`rejected`，也就是遇到错误)的时候调用。

```javascript
obj.save().then(function(obj) {
  //对象保存成功
}, function(error) {
  //对象保存失败，处理 error
});
```

其中第二个参数是可选的。

### try、catch 和 finally 方法

你还可以使用 `try,catch,finally` 三个方法，将逻辑写成：

```javascript
obj.save().try(function(obj) {
  //对象保存成功
}).catch(function(error) {
  //对象保存失败，处理 error
}).finally(function(){
  //无论成功还是失败，都调用到这里
});
```

类似语言里的 `try ... catch ... finally` 的调用方式来简化代码。

为了兼容其他 Promise 库，我们提供了下列别名：

* `AV.Promise#done` 等价于 `try` 方法
* `AV.Promise#fail` 等价于 `catch` 方法
* `AV.Promise#always` 等价于 `finally` 方法

因此上面例子也可以写成：

```javascript
obj.save().done(function(obj) {
  //对象保存成功
}).fail(function(error) {
  //对象保存失败，处理 error
}).always(function(){
  //无论成功还是失败，都调用到这里
});
```

### 将 Promise 组织在一起

Promise 比较神奇，可以代替多层嵌套方式来解决发送异步请求代码的调用顺序问题。
如果一个 Promise 的回调会返回一个 Promise，那么第二个 then 里的 callback 在第一个 then
的 callback 没有解决前是不会解决的，也就是所谓 **Promise Chain**。

```javascript
var query = new AV.Query("Student");
query.descending("gpa");
query.find().then(function(students) {
  students[0].set("valedictorian", true);
  return students[0].save();

}).then(function(valedictorian) {
  return query.find();

}).then(function(students) {
  students[1].set("salutatorian", true);
  return students[1].save();

}).then(function(salutatorian) {
  // Everything is done!

});
```

### 错误处理

如果任意一个在链中的 Promise 返回一个错误的话，所有的成功的 callback 在接下
来都会被跳过直到遇到一个处理错误的 callback。

处理 error 的 callback 可以转换 error 或者可以通过返回一个新的 Promise 的方式来处理它。你可以想象成拒绝的
 promise 有点像抛出异常,而 error callback 函数则像是一个 catch 来处理这个异常或者重新抛
出异常。

```javascript
var query = new AV.Query("Student");
query.descending("gpa");
query.find().then(function(students) {
  students[0].set("valedictorian", true);
  // 强制失败
  return AV.Promise.error("There was an error.");

}).then(function(valedictorian) {
  // 这里的代码将被忽略
  return query.find();

}).then(function(students) {
  // 这里的代码也将被忽略
  students[1].set("salutatorian", true);
  return students[1].save();
}, function(error) {
  // 这个错误处理函数将被调用，并且错误信息是 "There was an error.".
  // 让我们处理这个错误，并返回一个“正确”的新 Promise
  return AV.Promise.as("Hello!");

}).then(function(hello) {
  // 最终处理结果
}, function(error) {
  // 这里不会调用，因为前面已经处理了错误
});
```

通常来说，在正常情况的回调函数链的末尾，加一个错误处理的回调函数，是一种很
常见的做法。

利用 `try,catch` 方法可以将上述代码改写为：

```javascript
var query = new AV.Query("Student");
query.descending("gpa");
query.find().try(function(students) {
  students[0].set("valedictorian", true);
  // 强制失败
  return AV.Promise.error("There was an error.");

}).try(function(valedictorian) {
  // 这里的代码将被忽略
  return query.find();

}).try(function(students) {
  // 这里的代码也将被忽略
  students[1].set("salutatorian", true);
  return students[1].save();

}).catch(function(error) {
  // 这个错误处理函数将被调用，并且错误信息是 "There was an error.".
  // 让我们处理这个错误，并返回一个“正确”的新 Promise
  return AV.Promise.as("Hello!");
}).try(function(hello) {
  // 最终处理结果
}).catch(function(error) {
  // 这里不会调用，因为前面已经处理了错误
});
```

### 创建 Promise

在开始阶段,你可以只用系统（譬如 find 和 save 方法等）返回的 promise。但是，在更高级
的场景下，你可能需要创建自己的 promise。
在创建了 Promise 之后,你需要调用 `resolve` 或者 `reject` 来触发它的 callback.

```javascript
var successful = new AV.Promise();
successful.resolve("The good result.");

var failed = new AV.Promise();
failed.reject("An error message.");
```

如果你在创建 promise 的时候就知道它的结果，下面有两个很方便的方法可以使用：

```javascript
var successful = AV.Promise.as("The good result.");

var failed = AV.Promise.error("An error message.");
```

除此之外，你还可以为 `AV.Promise` 提供一个函数，这个函数接收 `resolve` 和 `reject` 方法，运行实际的业务逻辑。例如：

```javascript
var promise = new AV.Promise(function(resolve, reject){
  resolve(42);
});

promise.then(functon(ret){
  console.log(ret); //print 42.
});
```

尝试下两个一起用：

```javascript
var promise = new AV.Promise(function(resolve, reject) {
      setTimeout(function() {
          if (Date.now() % 2) {
               resolve('奇数时间');
          } else {
               reject('偶数时间');
          }
      }, 2000);
});

promise.then(function(value) {
    console.log(value);  // 奇数时间
}, function(value) {
    console.log(value);  // 偶数时间
});
```

### 顺序的 Promise

在你想要某一行数据做一系列的任务的时候，Promise 链是很方便的，每一个任务都等着前
一个任务结束。比如，假设你想要删除你的博客上的所有评论：

**特别说明，下文出现在代码里的 `_.xxx` 表示引用了 [underscore.js](http://underscorejs.org/) 这个类库的方法，underscore.js
是一个非常方便的 JS 类库，提供了很多工具方法。**

```javascript
var query = new AV.Query("Comment");
query.equalTo("post", post); // 假设 post 是一个已经存在的实例

query.find().then(function(results) {
  // Create a trivial resolved promise as a base case.
  var promise = AV.Promise.as();
  _.each(results, function(result) {
    // For each item, extend the promise with a function to delete it.
    promise = promise.then(function() {
      // Return a promise that will be resolved when the delete is finished.
      return result.destroy();
    });
  });
  return promise;

}).then(function() {
  // Every comment was deleted.
});
```

### 并行的 Promise

你也可以用 Promise 来并行的进行多个任务，这时需要使用 when 方法，你可以一次同
时开始几个操作。使用 `AV.Promise.when` 来创建一个新的 promise，它会在所有输入
的 `Promise` 被 resolve 之后才被 resolve。即便一些输入的 promise 失败了，其他的 Promise 也会被
成功执行。你可以在 callback 的参数部分检查每一个 promise 的结果。并行地进行操作会比顺序进行更快，但是也会消耗更多的系统资源和带宽。

简单例子：

```javascript
  function timerPromisefy(delay) {
    return new AV.Promise(function (resolve) {
      //延迟 delay 毫秒，然后调用 resolve
      setTimeout(function () {
        resolve(delay);
      }, delay);
     });
  };

   var startDate = Date.now();

   AV.Promise.when(
     timerPromisefy(1),
     timerPromisefy(32),
     timerPromisefy(64),
     timerPromisefy(128)
   ).then(function (r1, r2, r3, r4) {
        //r1,r2,r3,r4 分别为1,32,64,128
        //大概耗时在 128 毫秒
        console.log(new Date() - startDate);
   });

   //尝试下其中一个失败的例子
   var startDate = Date.now();
   AV.Promise.when(
     timerPromisefy(1),
     timerPromisefy(32),
     AV.Promise.error('test error'),
     timerPromisefy(128)
   ).then(function () {
        //不会执行
   }, function(errors){
       //大概耗时在 128 毫秒
        console.log(new Date() - startDate);
        console.dir(errors);  //print [ , , 'test error',  ]
   });
```

下面例子执行一次批量删除某个 Post 的评论：

```javascript
var query = new AV.Query("Comment");
query.equalTo("post", post);  // 假设 post 是一个已经存在的实例

query.find().then(function(results) {
  // Collect one promise for each delete into an array.
  var promises = [];
  _.each(results, function(result) {
    // Start this delete immediately and add its promise to the list.
    promises.push(result.destroy());
  });
  // Return a new promise that is resolved when all of the deletes are finished.
  return AV.Promise.when(promises);

}).then(function() {
  // Every comment was deleted.
});
```

`when` 会在错误处理器中返回所有遇到的错误信息，以数组的形式提供。


除了 `when` 之外，还有一个类似的方法是 `AV.Promise.all`，这个方法和 `when` 的区别在于**它只接受数组形式的 promise 输入，并且如果有任何一个 promise 失败，它就会直接调用错误处理器，而不是等待所有 promise 完成，其次是它的 resolve 结果返回的是数组**，一个例子：

```javscript
     AV.Promise.all([
       timerPromisefy(1),
       timerPromisefy(32),
       timerPromisefy(64),
       timerPromisefy(128)
     ]).then(function (values) {
       //values 数组为 [1, 32, 64, 128]
     })
     //测试下失败的例子
     AV.Promise.when(
       timerPromisefy(1),
       timerPromisefy(32),
       AV.Promise.error('test error'),
       timerPromisefy(128)
     ).then(function () {
        //不会执行
     }, function(error){
       console.dir(error);  //print 'test error'
     });
```

### race 方法

`AV.Promise.race` 方法接收一个 promise 数组输入，当这组 promise 中的任何一个 promise 对象如果变为 resolve 或者 reject 的话， 该函数就会返回，并使用这个 promise 对象的值进行 resolve 或者 reject。`race`，顾名思义就是在这些 promise “赛跑”，谁先执行完成，谁就先 resolve。

```javascript
var p1 = AV.Promise.as(1),
    p2 = AV.Promise.as(2),
    p3 = AV.Promise.as(3);
Promise.race([p1, p2, p3]).then(function (value) {
    console.log(value);  // 打印 1
});
```


### 创建异步方法

有了上面这些工具以后，就很容易创建你自己的异步方法来返回 promise 了。譬如，你可以创建一个有 promise 版本的 setTimeout：

```javascript
var delay = function(millis) {
  var promise = new AV.Promise();
  setTimeout(function() {
    promise.resolve();
  }, millis);
  return promise;
};

delay(100).then(function() {
  // This ran after 100ms!
});
```

### 兼容性

在非 node.js 环境（例如浏览器环境）下，`AV.Promise` 并不兼容 [Promises/A+](https://promisesaplus.com/) 规范，特别是错误处理这块。
如果你想兼容，可以手工启用：

```javascript
AV.Promise.setPromisesAPlusCompliant(true);
```

在 node.js 环境下如果启用兼容 Promises/A+， 可能在一些情况下 promise 抛出的错误无法通过 `process.on('uncaughtException')` 捕捉，你可以启用额外的 debug 日志：

```javascript
AV.Promise.setDebugError(true);
```

默认日志是关闭的。

### JavaScript Promise 迷你书

如果你想更深入地了解和学习 Promise，我们推荐[《JavaScript Promise迷你书（中文版）》](http://liubin.github.io/promises-book/)这本书。

## Collection

一个 `AV.Collection` 就是一个 `AV.Objects` 的有序集合，它和
 `Backbone.Collection` 是兼容的，有相同的特性和功能，你可以通过用一个模型类
或者一个特定的 `AV.Query` 来创建一个新的子类。

```javascript
// A Collection containing all instances of TestObject.
var TestCollection = AV.Collection.extend({
  model: TestObject
});
var collection = new TestCollection();

// A Collection of TestObjects whose temperature is "hot".
var HotCollection = AV.Collection.extend({
  model: TestObject,
  query: (new AV.Query(TestObject)).equalTo("temperature", "hot")
});
var collection = new HotCollection();

// The Collection of TestObjects that match a complex query.
var query = new AV.Query(TestObject);
query.equalTo("temperature", "hot");
query.greaterThan("degreesF", 100);
var collection = query.collection();
```

### 获取 Collection

使用 fetch 方法来获取一个 collection 里的所有元素：

```javascript
var collection = new TestCollection();
collection.fetch({
  success: function(collection) {
    collection.each(function(object) {
      console.warn(object);
    });
  },
  error: function(collection, error) {
    // The collection could not be retrieved.
  }
});
```

### Collection 排序

你可以设定一个 comparator 来对 collection 中的元素进行排序：

```javascript
var collection = new TestCollection();
collection.comparator = function(object) {
  return object.get("temperature");
};
```

### 修改一个 Collection

Collection 是可变的，你可以访问所有元素，增加或者删除元素：

```javascript
var collection = new TestCollection();

collection.add([
  {"name": "张三"},
  {"name": "李四"}
]);

// Get the "张三" AV.Object by its sorted position.
var model = collection.at(0);

// Or you can get it by LeanCloud objectId.
var modelAgain = collection.get(model.id);

// Remove "张三" from the collection.
collection.remove(model);

// Completely replace all items in the collection.
collection.reset([
  {"name": "李雷"},
  {"name": "韩梅梅"}
]);
```

## 文件

### 新建一个 AV.File

`AV.File` 让你可以在 LeanCloud 中保存应用的文件，这样可以解决用一个
 `AV.Object` 无法存太大或者太难处理的问题。最常见的用例就是存储图片，但是你可
以随意用来存储文档、视频、音乐或者任何二进制数据。

在浏览器通过 JavaScript SDK 上传文件大小不能超过 10 M，在 Node.js 环境里没有这个限制。

开始使用 AV.File 是很容易的，有很多种不同的方式来新建一个 file。第一个是 base64 编码的字符串表示：

```javascript
var base64 = "6K+077yM5L2g5Li65LuA5LmI6KaB56C06Kej5oiR77yf";
var file = new AV.File("myfile.txt", { base64: base64 });
```

另外，也可以用一个 byte 数组来新建一个文件。

```javascript
var bytes = [ 0xBE, 0xEF, 0xCA, 0xFE ];
var file = new AV.File("myfile.txt", bytes);
```

但是最经常的对于 HTML5 的应用来说，你可能需要用 html 表单和一个文件上传控制器。在
现代的浏览器中这很容易，只需要创建一个 file input tag 来允许用户选择他们磁盘
上的文件就可以了：

```javascript
<input type="file" id="photoFileUpload">
```

然后，在一个处理 click 或其他事件的函数里，获取对那个文件的一个引用:

```javascript
var fileUploadControl = $("#photoFileUpload")[0];
if (fileUploadControl.files.length > 0) {
  var file = fileUploadControl.files[0];
  var name = "avatar.jpg";

  var avFile = new AV.File(name, file);
}
```

注意在这个例子里我们会给文件一个名字叫 `avatar.jpg`，这里有 2 点值得注意：

- 你不需要担心文件名重复的问题。每一次上传都会有一个独一无二的标识符，所
  以上传多个文件都叫 `avatar.jpg` 是没有问题的。
- 你应该给你的文件一个扩展名。这样会让 LeanCloud 明白文件的类型，并且会按文件
  类型来进行处理。所以如果你在储存 PNG 格式的文件的话，请保证你的文件名是
  以 ".png" 为结尾的。

如果你是在 Node.js 里使用我们的 SDK，从 `0.3.1` 版本开始，我们也让 AV.File 的构造函数接收 [Buffer](http://nodejs.org/api/buffer.html) 作为文件存储：

```javascript
var file = new AV.File('test.txt', new Buffer('hello world'));
```

因为 Node.js 对 IO 的读写经常都是经过 Buffer，通过支持 Buffer，我们的 SDK 也能很好地工作在 Node.js 环境。

从`0.3.2`版本开始，我们还支持保存一个现有存储在其他服务上的 URL 的文件对象：

```javascript
var file = AV.File.withURL('test.jpg', 'images/permission.png');
file.save();
```

下面你应该向 LeanCloud 上传你的文件了。就像 AV.Object 一样，有很多不同的
save 方法，你可以按你想用的 callback 和 error 处理的方式来使用它们：

```javascript
avFile.save().then(function() {
  // The file has been saved to AV.
}, function(error) {
  // The file either could not be read, or could not be saved to AV.
});
```

最后，在保存完成之后，你可以将一个 AV.File 和一个 AV.Object 关联起来，就像别的数据一样：

```javascript
var post = new AV.Object("Post");
post.set("content", "#花儿与少年# 迪拜疯狂之旅");
post.set("image", file);
post.save();
```

### 获取文件的内容

怎样才能更好地获取你的应用数据取决于你的应用环境。因为跨域请求
的问题，最好你可以让浏览器代替你做这项事情。通常，这意味着在 DOM 中渲染这个
文件的 url。下面就是我们如何用 jQuery 将一个图片文件插入页面之中：

```javascript
var avatarPhoto = profile.get("avatarFile");
$("avatarImg")[0].src = avatarPhoto.url();
```

如果你想在云引擎中处理一个文件的数据， 你可以用我们的 http 网络库来获取这个文件。

```javascript
AV.Cloud.httpRequest({ url: avatarPhoto.url() }).then(function(response) {
  // The file contents are in response.buffer.
});
```

### 文件元信息

你还可以在上传文件之前设置文件的元信息，例如：

```javascript
file.metaData().mimeType = 'text/plain';
```

获取元信息列表函数 `metaData` 返回的是一个 JSON 对象。

默认会保存 `size` 和 `ownerId` 两个元信息，分别表示文件大小和文件上传者的 objectId:

```javascript
var size = file.size();
var ownerId = file.ownerId();
```

`0.3.1`版本开始，我们增强了 metaData 方法，它同时是 getter 和 setter 方法:

```javascript
//获取所有元信息组成的JSON对象
var metadata = file.metaData();
//设置format元信息
file.metaData('format','image/jpeg');
//获取format元信息
var format = file.metaData('format');
```

### 缩略图

如果保存的文件是图片，还可以通过 `thumbnailURL` 方法获得缩略图的 URL：

```javascript
//获得宽度为100像素，高度200像素的缩略图
var url = file.thumbnailURL(100, 200);
```

更多参数（格式、质量等）请看 API 文档。

### 删除文件

使用 `destroy` 方法来删除文件：

```javascript
file.destroy().then(function(){
  //删除成功
}, function(error){
  //删除失败
  console.dir(error);
});
```

## 用户

在许多应用中，都有一个用户账户的概念。用户账户让用户可以用安全的方式访问他们自己的信息，LeanCloud 提供了一个特殊的用户类叫 `AV.User` 来自动处理有关用户的账户管理的功能。

`AV.User` 是 `AV.Object` 的一个子类，而且有 AV.Object 一样的功能，比如可变的模式、自
动的持久化，还有键值对接口。所有对 AV.Object 有用的方法同样可以作
用于 AV.User。AV.User 的不同之处在于它对于用户的账户有一些特定的功能.

### 属性

AV.User 默认有一些与 AV.Object 不一样的字段:

- username : 用户的用户名(必须提供)
- password : 用户的密码(在注册的时候必须提供)
- email : 用户的email(可选)
- mobilePhoneNumber: 用户的手机号码（可选）

我们会在下面的用例中详细介绍细节

### 注册

通常你的 app 第一件要做的事情就是让用户进行注册，下面的代码展示了怎样进行
通常的注册过程：

```javascript
var user = new AV.User();
user.set("username", "hjiang");
user.set("password", "f32@ds*@&dsa");
user.set("email", "hang@leancloud.rocks");

// other fields can be set just like with AV.Object
user.set("phone", "186-1234-0000");

user.signUp(null, {
  success: function(user) {
    // 注册成功，可以使用了.
  },
  error: function(user, error) {
    // 失败了
    alert("Error: " + error.code + " " + error.message);
  }
});
```

这个调用会异步地在你的应用中创建一个新的用户。在它这样做之前，它同
样会确认用户名和 email 在应用内都是唯一的。同样，为了安全我们会将密码散列
过后存储在 LeanCloud 中，我们从不会将用户密码以明文存储，也不会用明文
向任何客户端发送密码。

注意我们使用了 `signUp` 方法而不是 `save` 方法。新的 `AV.User` 永远应该使用
`signUp` 方法来新建，而随后的用户的信息更新可以调用 `save` 来做。

如果一个 signUp 没有成功的话，你应该读取返回的错误对象。最常见的问题是
username 或者 email 已经被其他用户所使用了，你应该清楚地反馈给用户，让
他们再次用一个不同的用户名来注册。

你也可以使用 email 来作为用户名，只要求你的用户输入他们的 email 但是同时自动
填充好 username 属性就可以了，AV.User 会跟原来一样工作。我们会在下面的重设
密码环节再次说明这个细节。

### 登录

在你要求你的用户注册之后，当然应该让他们在以后用自己的账户登录进来。你可
以使用 logIn 方法来进行登录：

```javascript
AV.User.logIn("myname", "mypass", {
  success: function(user) {
    // 成功了，现在可以做其他事情了.
  },
  error: function(user, error) {
    // 失败了.
  }
});
```

### 验证 Email

在应用设置的应用选项中中启用 email 验证可以让你的应用给最终用户一些更安全的使用体验，
譬如部分功能只开放给验证过邮箱的用户使用，等等。

Email 验证会在 AV.User 上加入一个 `emailVerified` 字段，当一个
AV.User 的 email 被设定或者修改后，`emailVerified` 会被置为 false，同时 LeanCloud 云端会
向用户的 email 来发送一个确认链接。用户接收邮件之后，点击这个链接会设置 `emailVerified` 为 true。

有三种 `emailVerified` 状态可以供参考:

1. true。 用户已经通过点击 LeanCloud 发过来的链接来确认邮箱地址，当用户账户新创
   建的时候这个值永远不应该是 true.
2. false。 在 AV.User 对象最后一次刷新的时候，用户还是没有确认他们的
   email 地址，如果 emailVerified 是 false 的话，你应该考虑调用 AV.User 的
   fetch 方法。
3. 空值。 AV.User 被创建了，但是当时的 email 验证功能还没有开启，或者说 AV.User 没有 email 地址.

关于自定义邮件模板和验证链接请看这篇[博客](http://blog.leancloud.cn/blog/2014/01/09/zi-ding-yi-ying-yong-nei-yong-hu-zhong-she-mi-ma-he-you-xiang-yan-zheng-ye-mian/)。

用户邮箱验证后，会调用 `AV.Cloud.onVerified('email',function)` 的云引擎回调函数，方便你做一些后处理。

### 修改密码

用户修改密码，跟修改其他属性没有什么区别，前提是要求处于登录状态：

```javascript
var user = AV.User.current();
user.setPassword('new password');
user.save().then(……)
```

有时候，你希望验证用户的当前密码之后才允许更新密码，可以用 `updatePassword`方法：

```javascript
var user = AV.User.current();
user.updatePassword('当前密码', '新密码',{
  success: function(){
    //更新成功
  },
  error: function(user, err){
    //更新失败
    console.dir(err);
  }
});
```

### 短信验证手机号码

如果用户注册提供了 `mobilePhoneNumber` 属性，并且你希望验证用户手机号码的真实性，你可能希望发送一条短信，并且让用户输入短信中的验证码来确认手机号码的真实性：

```javascript
var user = new AV.User();
user.set("username", "hjiang");
user.set("password", "123456");
user.setMobilePhoneNumber('186xxxxxxxx');
user.signUp(null, ……)
```

为了发送短信，你需要在应用设置的应用选项里启用：`验证注册用户手机号码`。

如果用户注册没有收到短信，你可以通过 `requestMobilePhoneVerify` 方法强制重新发送：

```javascript
AV.User.requestMobilePhoneVerify('186xxxxxxxx').then(function(){
  //发送成功
}, function(err){
   //发送失败
});
```

当用户收到验证短信后，会有 6 位数字的验证码，让用户输入，并调用 `verifyMobilePhone` 来确认是否正确：

```javascript
AV.User.verifyMobilePhone('6位数字验证码').then(function(){
  //验证成功
}, function(err){
  //验证失败
});
```

验证成功后，用户的 `mobilePhoneVerified` 属性变为 true，并且调用云引擎的 `AV.Cloud.onVerifed('sms', function)` 方法。


### 手机号码和短信登录

当用户有填写 `mobilePhoneNumber` 的时候，可以使用手机和密码登录：

```javascript
AV.User.logInWithMobilePhone('186xxxxxxxx', password).then(function(user){
  //登录成功
}, function(err){
  //登录失败
});
```

如果你在应用设置的应用选项里启用 `允许用户使用手机短信登录 （需要先启用 验证注册用户手机号码）`，并且用户通过了手机号码认证，那么可以通过请求发送短信验证码来登录：

```javascript
//请求登录验证码
AV.User.requestLoginSmsCode('186xxxxxxxx').then(function(){
  //发送成功
}, function(err){
  //发送失败
});

//用户收到6位登录验证码后，输入验证码登录
AV.User.logInWithMobilePhoneSmsCode('186xxxxxxxx', '6位登录验证码数字').then(function(user){
  //登录成功
}, function(err){
  //登录失败
});
```

### 手机号码一键登录

很多情况下，我们希望用户直接输入手机号码，获取短信然后注册或者登录，如果没有注册过就注册，否则就直接登录，我们提供了一个 API `signUpOrlogInWithMobilePhone` 来实现。

```javascript
//获取短信
//在应用选项开启 "启用帐号无关短信验证服务（针对 requestSmsCode 和 verifySmsCode 接口）"
AV.Cloud.requestSmsCode('186xxxxxxxx').then(function(){
  //发送成功
}, function(err){
  //发送失败
});
```

用户拿到短信后，使用下列代码注册或者登录：

```javascript
var user = new AV.User();
user.signUpOrlogInWithMobilePhone({
  mobilePhoneNumber: '186xxxxxxxx',
  smsCode: '手机收到的 6 位验证码字符串',
  ……其他属性，比如 username 等。
},
{
  success:function(user){
    //注册或者登录成功
  },
  error: function(user, err){
    //失败
    console.dir(err);
  }
});
```

如果是注册，属性里其他属性将作为新用户的属性保存，如果是登录，这些属性将覆盖服务端的属性。如果不提供 `username`，默认为手机号码。


### 当前用户

如果用户每次打开 app 的时候都要求登录无疑是令人感到厌烦的，你可以通过缓存当前的 AV.User 对象来避免这个问题。

无论你使用任何注册或者登录方法，用户都会在 localStorage 中储存，你可以把缓存作为一个 session 对待，并且自动假设用户已经登录了。

```javascript
var currentUser = AV.User.current();
if (currentUser) {
  // do stuff with the user
} else {
  // show the signup or login page
}
```

你可以通过 `logOut` 来清除掉当前的用户:

```javascript
AV.User.logOut();

var currentUser = AV.User.current();  // this will now be null
```

### 用户对象的安全

AV.User 类默认就是受保护的，在 AV.User 中保存的数据只能被那个用户所
修改。默认地，数据仍然可以被任意客户端所读取。这样就是说，有些 AV.User 对
象被认证后是可以修改的，其他的仍然是只读的.

>> 特别的，你不能调用 `save` 或者 `delete` 方法，除非 AV.User 经过了认证，就比如调用
过了 `logIn` 或者 `signUp` 方法，这样保证只有用户自己能改动他们的数据。

下面的代码展示了上面说的安全策略：

```javascript
var user = AV.User.logIn("my_username", "my_password", {
  success: function(user) {
    user.set("username", "my_new_username");  // attempt to change username
    user.save(null, {
      success: function(user) {
        // This succeeds, since the user was authenticated on the device

        // Get the user from a non-authenticated method
        var query = new AV.Query(AV.User);
        query.get(user.objectId, {
          success: function(userAgain) {
            userAgain.set("username", "another_username");
            userAgain.save(null, {
              error: function(userAgain, error) {
                // This will error, since the AV.User is not authenticated
              }
            });
          }
        });
      }
    });
  }
});
```

从AV.User.current() 获取的 AV.User 总是已经通过验证了的。

如果你需要查看一个 AV.User 是否已经认证过了，你可以调用 authenticated 方
法。你不需要查看一个认证方法中返回的 AV.User 对象是否已经通过验证了。

### 绑定第三方平台账户

通过 `AV.User._logInWith(platform, options)` 来将微博、微信等第三方平台号绑定到 AV.User 上，例如：

```javascript
AV.User._logInWith("weibo", {
   "authData": {
      "uid": "123456789",
       "access_token": "2.00vs3XtCI5FevCff4981adb5jj1lXE",
       "expiration_in": "36000"
    },
    success: function(user){
        //返回绑定后的用户
        console.dir(user);
    },
    error: function(err){
       console.dir(err);
    }
})
```

其中 `authData` 是微博端返回的用户信息 JSON 对象，更多平台支持和格式信息请参考 [REST API 用户账户连接](./rest_api.html#用户账户连接)。

要把这些授权信息绑定到一个已经存在的 `AV.User` 上，可以通过  `_linkWith` 方法：

```javascript
var user = ...已存在的处于登录状态的 AV.User 对象 ...
user._linkWith("weibo", {
   "authData": {
      "uid": "123456789",
       "access_token": "2.00vs3XtCI5FevCff4981adb5jj1lXE",
       "expiration_in": "36000"
    },
    success: function(user){
        //返回绑定后的用户
        console.dir(user);
    },
    error: function(err){
       console.dir(err);
    }
})
```

### 其他对象的安全

和 AV.User 相同的安全模型也使用于其他对象。对于任何对象来说，你可以指定
哪些用户会被允许读取该对象，哪些用户被允许修改该对象。为了支持这种安全机制，
每一个对象都有一个允许访问列表(ACL)，是被 AV.ACL 类所实现的。

使用一个 AV.ACL 最简单的方式是指定一个对象只能被一个单一的用户读或者
写。为了创建这样的对象，首先必须有一个已经登录的 AV.User；然后，新的
 AV.ACL(user) 生成一个 AV.ACL 来限定 user 的访问。一个对象的 ACL 会在对象
保存的时候被存储起来，就像其他的属性一样。这样，为了创建一个当前 user 私有
的一个 note:

```javascript
// 该语句应该只声明一次
var Note = AV.Object.extend("Note");
var privateNote = new Note();
privateNote.set("content", "This note is private!");
privateNote.setACL(new AV.ACL(AV.User.current()));
privateNote.save();
```

这个 note 只能由当前的用户所访问，但是对用户登录的设备没有限制，只要
是相同的用户就可以了。这项功能对于你如果想让用户再任何其他的设备上保存
和访问数据十分有用，比如说一个私人的 todo list 应用。

权限也能在使用者的基础上授予，你可以通过 setReadAccess 和 setWriteAccess 方
法独立的向 AV.ACL 中添加权限。比如，假设你有一条消息想要发送给一个组里
的多个用户，他们中的每一个都有读和写的权限：

```javascript
// 该语句应该只声明一次
var Message = AV.Object.extend("Message");
var groupMessage = new Message();
var groupACL = new AV.ACL();

// userList is an array with the users we are sending this message to.
for (var i = 0; i < userList.length; i++) {
  groupACL.setReadAccess(userList[i], true);
  groupACL.setWriteAccess(userList[i], true);
}

groupMessage.setACL(groupACL);
groupMessage.save();
```

你同样可以对所有的用户授权，只要使用 setPublicReadAccess 和 setPublicWriteAccess 就可以了。
这样允许了在一个消息板上发评论的模式，比如我
们要创建一个 post 只能被它的作者修改，但是可以被所有人读取：

```javascript
var publicPost = new Post();
var postACL = new AV.ACL(AV.User.current());
postACL.setPublicReadAccess(true);
publicPost.setACL(postACL);
publicPost.save();
```

比如说删除一个对象，但是你没有写的权限这种操作是禁止的。这样会返回一个
AV.Error.OBJECT_NOT_FOUND 的错误码。为了安全起见，这样防止了客户端区分
出到底有哪些对象被创建了但是无法读取还是根本不存在。

### 重设密码

在现实中只要你引入了密码系统，总会有用户会忘掉他们的密码。在这种情形下，
我们的库提供一个让他们安全地重设密码的功能。

为了能让用户重设密码，应该要求用户提供他们的 email 地址，然后这样调用:

```javascript
// 邮件重置
AV.User.requestPasswordReset("email@example.com", {
  success: function() {
    // Password reset request was sent successfully
  },
  error: function(error) {
    // Show the error message somewhere
    alert("Error: " + error.code + " " + error.message);
  }
});

// 短信重置
AV.User.requestPasswordResetBySmsCode("18212346648", {
  success: function() {
    // Password reset request was sent successfully
  },
  error: function(error) {
    // Show the error message somewhere
    alert("Error: " + error.code + " " + error.message);
  }
});
```

这样会尝试匹配给定的 email 和用户的 email 或者 username 字段，然后会发送用户
的密码重设邮件。由于我们是这样做的，所以你可以选择用户是否拿 email 作为他
们的用户名，或者说用户把 email 作为用户的另一个信息保存。

密码重设的流程如下:

1. 用户输入 email 来请求重设他们的密码
2. LeanCloud 向用户的 email 地址发送邮件，包含了一个重设密码的链接
3. 用户点击这个重设密码的链接，会重定向到一个 LeanCloud 页面来允许他们重设密
   码
4. 用户输入新的密码，他们的密码现在会更新为输入的新密码。

注意这个流程的信息会引用你的 App 的名字，这个名字是你初始在 LeanCloud 上创建的
 App 的名字.

关于自定义邮件模板和验证链接请看这篇[博客](http://blog.leancloud.cn/blog/2014/01/09/zi-ding-yi-ying-yong-nei-yong-hu-zhong-she-mi-ma-he-you-xiang-yan-zheng-ye-mian/)。

### 查询

**请注意，新创建应用的 `_User` 表的查询权限默认是关闭的，通常我们推荐你在云引擎里封装用户查询，只查询特定条件的用户，避免开放用户表的全部查询权限。此外，你可以通过 class 权限设置打开查询权限，请参考 [数据与安全 - Class 级别的权限](data_security.html#Class_级别的权限)。**

为了查询用户，你可以简单地针对 AV.User 来创建一个 AV.Query :

```javascript
var query = new AV.Query(AV.User);
query.equalTo("gender", "female");  // find all the women
query.find({
  success: function(women) {
    // Do stuff
  }
});
```

### 关联

关联一个 AV.User 的对象可以很快就见效。比如说，假设你有一个blog
程序,为了保存一个用户的新post还有读取他们所有的post.

```javascript
var user = AV.User.current();

// Make a new post
var post = new Post();
post.set("content", "walking in Dubai.");
post.set("author", user);
post.save(null, {
  success: function(post) {
    // Find all posts by the current user
    var query = new AV.Query(Post);
    query.equalTo("author", user);
    query.find({
      success: function(usersPosts) {
        // userPosts contains all of the posts by the current user.
      }
    });
  }
});
```

### 在后台查看 User

在后台的数据查看中，你可以看到 User 类保存了用户的信息.

## 角色

随着你的App规模和用户基数的成长,你可能发现你需要比设定用户级的权限更加
宽泛的权限设置.LeanCloud提供一种基于角色的权限管理方案来满足这种需求.角色
提供了一种逻辑的方式来将用户分组并给与相同的权限.角色是一种有名字的对
象,包含了用户和其他的角色.任何授予一个角色的权限会被它包含的所有用户和
子角色所继承.

例如,在你的App中管理着一些内容,你可能有一些类似于"主持人"的角色可以修
改和删除其他用户创建的新的内容,你可能还有一些"管理员"有着与"主持人"相
同的权限,但是还可以修改App的其他全局性设置.通过给予用户这些角色,你可以
保证新的用户可以做主持人或者管理员,不需要手动地授予每个资源的权限给各
个用户.

我们提供一个特殊的类称为AV.Role在客户端代码中表示这种角色对
象,AV.Role是一个AV.Object的子类,而且有所有的特性,比如没有固定模
式,自动持久化和key value接口等.所有的在AV.Object上有用的方法在
AV.Role上依然有作用.不同之处是AV.Role有一些普通对象没有的特殊属
性和方法.

### 属性

AV.Role有一些属性与普通的AV.Object不同:

- name 角色的名称,这个值是必须的,而且只能在角色创建的时候指定一次,名字
  必须由字母,数字,空格,减号或者下划线组成.名称会被用于表示角色名而不需
  要角色的objectId
- users 一个关系,包含了会继承角色权限的User
- roles 一个关系,包含了会继承角色权限的子角色

### 角色对象的安全性

AV.Role使用和其他LeanCloud对象一样的ACL权限策略,除开它需要ACL被显式地设
置以外.通常来说,只有用户有极大的权限(比如管理员)才应该被允许创建或者更
改Role.所以你应该按这种标准来设定Role的ACL.请注意,如果你给了用户一个
AV.Role一个写权限,这个用户有可能会在这个权限中加入另一个user,或者甚
至直接把角色删除掉.

为了创建一个新的AV.Role,你应该如下写:

```javascript
// By specifying no write privileges for the ACL, we can ensure the role cannot be altered.
var roleACL = new AV.ACL();
roleACL.setPublicReadAccess(true);
var role = new AV.Role("Administrator", roleACL);
role.save();
```

你可以通过增加"user"和"roles"关系的成员来在AV.Role中加入用户或者子
角色:

```javascript
var role = new AV.Role(roleName, roleACL);
for (var i = 0; i < usersToAddToRole.length; i++) {
  role.getUsers().add(usersToAddToRole[i]);
}
for (var i = 0; i < rolesToAddToRole.length; i++) {
  role.getRoles().add(rolesToAddToRole[i]);
}
role.save();
```

请非常注意一点,注册角色的ACL的时候,它们只能被应该有权限修改它的人修改.

### 其他对象的安全性

现在你应该已经创建了在你的程序中要使用的一系列的角色,你可以用ACL来定义
他们的用户可以拥有的权限.每一个AV.Object都可以指定一个AV.ACL,这
样提供了哪些用户或者角色应该有权限来读或者写这个对象.

将一个读或者写的权限授予一个角色是很直观的.你可以使用Rarse.Role:

```javascript
var moderators = /* Query for some AV.Role */;
var wallPost = new AV.Object("WallPost");
var postACL = new AV.ACL();
postACL.setRoleWriteAccess(moderators, true);
wallPost.setACL(postACL);
wallPost.save();
```

你可以不需要查找这个Role,直接把名字提供给ACL:

```javascript
var wallPost = new AV.Object("WallPost");
var postACL = new AV.ACL();
postACL.setRoleWriteAccess("Moderators", true);
wallPost.setACL(postACL);
wallPost.save();
```

### 角色继承

就像上面所描述的一样,一个角色可能包含其他的角色,表示两个角色之间的父-
子关系,这样做的结果就是任何被授予一个角色的权限都会被隐式地授予这个角
色的所有子角色.

这样的关系很经常会在有用户管理内容的程序之中看到,比如论坛,有一个很少量
的用户称为管理员,有最高的权限,比如程序设定,创建新的论坛,设定所有人能看
到的内容等等.另一类有一部分类似于"版主"的用户,这些人有责任保持用户创建
的内容是合适的.任何一个"版主"有的权限"管理员"都应该有.为了启用这种关系,你
应该使"管理员"成为"版主"的一个子角色.

```javascript
var administrators = /* Your "Administrators" role */;
var moderators = /* Your "Moderators" role */;
moderators.getRoles().add(administrators);
moderators.save();
```

## 云引擎 函数

云引擎 函数应该用AV.Cloud.run函数来进行调用,比如,调用云引擎中的
函数"hello"应该这样:

```javascript
AV.Cloud.run('hello', {}, {
  success: function(result) {
    // result is 'Hello world!'
  },
  error: function(error) {
  }
});
```

你可以参考我们的Cloud Code指南来进一步了解这部分功能.

## Push 通知

通过 JavaScript SDK 也可以向移动设备推送消息，使用也非常简单。

（如果想在 Web 端独立使用推送模块，包括通过 Web 端推送消息到各个设备、以及通过 Web 端也可以接收其他端的推送，可以了解下我们的「[JavaScript 推送 SDK 使用指南](./js_push.html)」来获取更详细的信息。）

一个简单例子推送给所有订阅了`public`频道的设备：

```javascript
AV.Push.send({
  channels: [ "Public" ],
  data: {
    alert: "Public message"
  }
});
```
这就向订阅了`public`频道的设备发送了一条内容为`public message`的消息。

如果希望按照某个 `_Installation` 表的查询条件来推送，例如推送给某个 `installationId` 的 Android 设备，可以传入一个 `AV.Query` 对象作为 `where` 条件：

```javascript
var query = new AV.Query("_Installation");
query.equalTo("installationId", installationId);
AV.Push.send({
  where: query,
  data: {
    alert: "Public message"
  }
});
```

此外，如果你觉得 AV.Query 太繁琐，也可以写一句 [CQL](./cql_guide.html) 来搞定：

```javascript
AV.Push.send({
  cql: "select * from _Installation where installationId='设备id'",
  data: {
    alert: "Public message"
  }
});
```

`AV.Push` 的更多使用信息参考 API 文档 [AV.Push](api/javascript/symbols/AV.Push.html)。

更多推送的查询条件和格式，请查阅我们的[Push Notification指南](./push_guide.html)来获取更详细的信息。

iOS 设备可以通过 `prod` 属性指定使用测试环境还是生产环境证书：

```javascript
AV.Push.send({
  prod: "dev",
  data: {
    alert: "Public message"
  }
});
```

`dev` 表示测试证书，`prod`表示生产证书，默认生产证书。

## 地理位置

LeanCloud 允许你能够将真实世界的经度和纬度坐标放入对象之中。在 AV.Object 中
加入一个 AV.GeoPoint 可以让你查询一个对象离一个参考点的相对位置，这
允许你轻松的发现一个用户周围最近的用户，或者离一个用户最近的地点。

### AV.GeoPoint

为了将一个对象联系到一个点上,你需要先创建一个AV.GeoPoint。举例来说,
创建一个北纬 39.9 度、东经 116.4 度的 `AV.GeoPoint` 对象（LeanCloud 北京办公室所在地）:

```javascript
var point = new AV.GeoPoint({latitude: 39.9, longitude: 116.4});
```

这个点接着就在对象中被指定了:

```javascript
post.set("location", point);
```

注意：现在 LeanCloud 限制了一个对象中只能有一个 AV.GeoPoint 的属性。

### 地理位置查询

现在你可以有一系列的对象可以做空间坐标查询了。如果想找到有哪些对
象离一个地点最近，可以通过在 AV.Query 中加入一个 `near` 来做查询。例如，
为了获得离用户最近的 10 条微博，可以这样：

```javascript
// User's location
var userGeoPoint = userObject.get("location");

// Create a query for posts
var query = new AV.Query(Post);

// Interested in posts near user.
query.near("location", userGeoPoint);

// Limit what could be a lot of points.
query.limit(10);

// Final list of objects
query.find({
  success: function(posts) {
  }
});
```

在这时 posts 会返回一个按离 userGeoPoint 的距离排序的列表。注意如果
在 AV.Query 上调用了 `ascending()`/`descending()` 的话，指定的排序属性会取代距离。

为了按距离限制返回的结果，你还可以使用 `withinMiles`、`withinKilometers` 和 `withinRadians`。

同样地，也可以查询在特定地域的对象。为了找到用矩形表示的一块地域中的对
象，需要在 AV.Query 中加入 `withinGeoBox` 约束条件：

```javascript
var point1 = new AV.GeoPoint(39.97, 116.33);
var point2 = new AV.GeoPoint(39.99, 116.37);

var query = new AV.Query(Post);
query.withinGeoBox("location", point1, point2);
query.find({
  success: function(posts) {
    ...
  }
});
```

### 警告

在这里是有一些问题是值得留心的:

1. 每一个 AV.Object 只能有一个键指向一个 AV.GeoPoint 对象
2. Points 不应该等于或者超出它的界. 纬度不应该是 -90.0 或者 90.0，经度不应
   该是 -180.0 或者 180.0。试图在 GeoPoint 上使用超出范围内的经度和纬度会导
   致问题.


### View

我们引入的AV.View只是一个Backbone.View的简单复制,你可以随意拿它来建
立视图,你可以查看Backbone.View的API来查看详细信息,注意当你使用了
AV.View的时候,你需要包含一个jQuery库或者jQuery兼容的库(实现了$方法
的)

### 转换 Backbone app

如果你已经有一个存在的Backbone程序,你可以用我们的 JavaScript SDK 轻松地
转换.在转换过后,你就有一些静态文件但是包含了你的app的所有功能.

我们的 JavaScript SDK 是 Backbone 兼容的,这意味着我们的AV.Object和
AV.Collection都可以用Backbone.Model和Backbone.Collection加上一点点
变化轻松地转换而来.下面就是怎样转换你的app:

1. 按照我们的说明来在你已经有的JavaScript程序中安装SDK.
2. 将所有的Backbone.Model都替换成AV.Object.这样做的时候,url和
   urlRoot应该用恰当的className替换,这些对象映射为LeanCloud的类

```javascript
var BackboneTodo = Backbone.Model.extend({
  urlRoot: "/todos"
});

var AVOSCloudTodo = AV.Object.extend({
  className: "Todo"
});
```

3. 将所有的Backbone.Collection替换为AV.Collection然后指定
   AV.Object类作为model.你应该同时指定query所以collection知道怎样获
   取对象

```javascript
var AVOSCloudTodoCollection = AV.Collection.extend({
  model: AVOSCloudTodo
});

var todos = new AVOSCloudTodoCollection();

// Construct a query to get the current user's todo items
var query = new AV.Query(AVOSCloudTodo);
query.equalTo("user", AV.User.current());
todos.query = query;
todos.fetch();
```

4. 在任何你建立model并从服务器获取的地方,你需要建立一个AV.Query来获
   取你感兴趣的对象,就像我们必须对AV.Collection的query属性做的一样
5. 加入或者将你的app更新为使用用户认证方式的,并且在需要的对象上应用ACL

这样就结束了,你的App应该已经就绪并使用LeanCloud作为后端.

## 错误处理

大部分LeanCloud JavaScript函数会通过一个有callback的对象来报告它们是否成功
了,与Backbone的"options"对象类似.主要的2个callback是success和error.
在一个操作都没有错误发生的时候success会被调用.通常来说,它的参数在save
或者get的情况下可能是AV.Object, 或者在find的情形下是一个
AV.Object数组.

error会在任何一种在与LeanCloud的网络连接发生错误的时候调用.这些错误
信息一般会反映连接到cloud时出现的一些问题,或者处理请求的操作时遇到的一
些问题.我们可以看下另一个例子,在下面的代码中我们想要获取一个不存在的
objectId. LeanCloud会返回一个错误,所以这里就是我们怎样在你的callback
里处理错误.

```javascript
var query = new AV.Query(Note);
query.get("aBcDeFgH", {
  success: function(results) {
    // This function will *not* be called.
    alert("Everything went fine!");
  },
  error: function(model, error) {
    // This will be called.
    // error is an instance of AV.Error with details about the error.
    if (error.code === AV.Error.OBJECT_NOT_FOUND) {
      alert("Uh oh, we couldn't find the object!");
    }
  }
});
```

查询在无法连接到LeanCloud的时候同样有可能失败.下面是同样的
callback但是有一些其他的代码来处理这种情况:

```javascript
var query = new AV.Query(Note);
query.get("thisObjectIdDoesntExist", {
  success: function(results) {
    // This function will *not* be called.
    alert("Everything went fine!");
  },
  error: function(model, error) {
    // This will be called.
    // error is an instance of AV.Error with details about the error.
    if (error.code === AV.Error.OBJECT_NOT_FOUND) {
      alert("Uh oh, we couldn't find the object!");
    } else if (error.code === AV.Error.CONNECTION_FAILED) {
      alert("Uh oh, we couldn't even connect to the LeanCloud!");
    }
  }
});
```

对于像是save或者是signUp这种方法会对一个特定的AV.Object起作用的方法
来说,error函数的第一个参数是object本身,第二个是一个AV.Error对象,这
是为了与其他的Backbone类型的框架兼容而设计的.请查看JavaScript API来得
到所有的AV.Error的返回码.

## 应用内搜索

我们虽然提供了基于正则的模糊查询，但是正则查询有两个缺点：

* 当数据量逐步增大后，查询效率将越来越低
* 没有文本相关性排序

因此，我们还提供了[应用内搜索功能](./app_search_guide.html)，基于搜索引擎构建，提供更强大的搜索功能。


## WebView 中使用

JS SDK 当然也支持在各种 WebView 中使用，可以将代码部署在 LeanCloud 的「云引擎」中。

### Android 中使用

如果是 Android WebView，在 Native 代码创建 WebView 的时候你需要打开几个选项，
这些选项生成 WebView 的时候默认并不会被打开，需要配置：

1、因为我们 JS SDK 目前使用了 window.localStorage，所以你需要开启 WebView 的 localStorage；
设置方式：

```java
yourWebView.getSettings().setDomStorageEnabled(true);
```

2、如果你希望直接调试手机中的 WebView，也同样需要在生成 WebView 的时候设置远程调试，
具体使用方式请参考 [Google 官方文档](https://developer.chrome.com/devtools/docs/remote-debugging)。

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
    yourWebView.setWebContentsDebuggingEnabled(true);
}
```

注意：这种调试方式仅支持 Android 4.4 已上版本（含 4.4）

3、如果你是通过 WebView 来开发界面，Native 调用本地特性的 Hybrid 方式开发你的 App。
比较推荐的开发方式是：
通过 Chrome 的开发者工具开发界面部分，当界面部分完成，与 Native 再来做数据连调，
这种时候才需要用 Remote debugger 方式在手机上直接调试 WebView。
这样做会大大节省你开发调试的时间，不然如果界面都通过 Remote debugger 方式开发，可能效率较低。

4、为了防止通过 JavaScript 反射调用 Java 代码访问 Android 文件系统的安全漏洞，在 Android 4.2以后的系统中间，WebView 中间只能访问通过[@JavascriptInterface](http://developer.android.com/reference/android/webkit/JavascriptInterface.html)标记过的方法。如果你的目标用户覆盖 4.2 以上的机型，请注意加上这个标记，以避免出现 "Uncaught TypeError"
