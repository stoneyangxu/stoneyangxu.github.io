---
layout: post
title:  "NodeJs in Action (三)"
date:   2017-09-10 14:59:34
categories: 读书笔记
---

- - - - -
第三部分 实战篇
- - - - -

# 使用express构建Web应用

在前面的学习过程中，我们使用大量重复的代码创建了很多应用。
在本章中，我们将会开始学习express——一个为Node量身打造的Web应用框架。

## 安装express

- 执行yarn add express 或者 npm install express --save

## 使用express创建"Hello World"

- 手动处理请求和响应时，代码如下：

![](/images/2017-09-10-15-08-16.jpg)

- 使用express创建应用

```js
const express = require('express')

const app = express()

app.get('/', (req, res) => {
    res.end('hello world')
})

app.listen(8080)
```
# express中的路由和分层

express完全构建在一个名叫connect的Node模块之上。 connect作为中间件(middleware)类库而为大家所熟知，它提供了一系列网络应用所需的常用功能。该模型的核心是函数流，也就是大家常说的函数分层机制。

## 中间件:名称的奥秘

中间件一开始是被用来描述软件组件的(一般是服务器端的东西)，这种组件会将两个东西连接起来，如事务逻辑层和数据库服务器的连接或者存储服务和应用模块的连接等。

随着时间的迁移，这个术语的定义变得更加通用和宽泛，现在可以用来描述任何连接两个事物的软件片段。

## 路由基础


![](/images/2017-09-10-15-16-12.jpg)

- 使用时，method替换成对应的http方法，例如get，post，delete或put
- url支持普通字符串、正则表达式或通配符
    - /path/to/resource 使用字符串指定路径
    - /user(s)?/list 匹配/user/list或/users/list
    - /users/* 匹配/users/下所有路径
- express路由最强大的特性之一就是支持占位符从请求路由中提取指定值，使用冒号(:)来标记。当解析路由的时候，express会将匹配到的占位符值保存到req.params对象中

```js

app.get('/albums/:album_name.json', (req, res) => {
    console.log(req.params)
})
```
> http://localhost:8080/albums/chind2009.json
> { album_name: 'chind2009' }

- 第三个参数next

![](/images/2017-09-10-15-23-52.jpg)

# 改造albums应用

## 安装express和async

![](/images/2017-09-10-15-25-28.jpg)

## 使用express创建应用并监听端口

![](/images/2017-09-10-15-26-51.jpg)

## 将handle_incoming_request替换到express到路由控制

![](/images/2017-09-10-15-27-07.jpg)

- 使用:albums_name提取相簿名
- 使用*匹配其他未定义到路由，返回404响应
- 正则表达式不能匹配带有斜杠(/)字符的参数，所以路由/pages/:page_name匹配不到该请求路径，修改为：

![](/images/2017-09-10-15-29-33.jpg)

# REST API设计和模块

*在设计API之前多花些时间思考，有助于理解用户如何使用应用并能帮助 组织和重构代码，使其能够精确传达你对产品的深刻理解*

## API设计

单词REST是Rep-resentational State Transfer的缩写，表明可以从服务器上请求一个对象。
- 创建 - PUT
- 获取 - GET
- 更新 - POST
- 删除 - DELETE

几种常见的模式：
- 整个集合 - /albums
- 集合中指定元素 - /albums/china2009
- 稍微改变获取数据集合的方式，例如分页或过滤，在请求中添加参数 - /albums.json?located_in=Europe
- API版本化，需要在API URL中添加/v1/前缀 - /v1/albums
- 在所有需要返回JSON数据的URL中添加.json后缀，这样客户端就会知道返回的数据格式。

## 模块化
将功能同类或者可复用的函数提取到相同的文件，通过导出和导入进行使用

# 中间件功能
express是构建在connect这个中间件类库之上的。这些组件链会依次处理每个请求，并会在组件调用next函数之后结束调用。
而路由处理程序则是这个组件链的一部分。express和connect还提供了一些其他实用组件

> 从 4.x 版本开始，, Express 已经不再依赖 Connect 了。除了 express.static, Express 以前内置的中间件现在已经全部单独作为模块安装使用了。

> [使用中间件](http://www.expressjs.com.cn/guide/using-middleware.html)

中间件的功能包括：

- 执行任何代码。
- 修改请求和响应对象。
- 终结请求-响应循环。
- 调用堆栈中的下一个中间件。

## Express 应用可使用如下几种中间件
- 应用级中间件 - 绑定到 app 对象 使用 app.use() 和 app.METHOD()

```js
var app = express();

// 没有挂载路径的中间件，应用的每个请求都会执行该中间件(在绑定get之前才生效)
app.use(function (req, res, next) {
  console.log('Time:', Date.now());
  next();
});

// 挂载至 /user/:id 的中间件，任何指向 /user/:id 的请求都会执行它
app.use('/user/:id', function (req, res, next) {
  console.log('Request Type:', req.method);
  next();
});

// 路由和句柄函数(中间件系统)，处理指向 /user/:id 的 GET 请求
app.get('/user/:id', function (req, res, next) {
  res.send('USER');
});

```

- 路由级中间件 - 路由级中间件和应用级中间件一样，只是它绑定的对象为 express.Router()。
- 错误处理中间件 - 和其他中间件定义类似，只是要使用 4 个参数，而不是 3 个，其签名如下： (err, req, res, next)。

```js
app.use(function(err, req, res, next) {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});

```

- 内置中间件 - express.static 是 Express 唯一内置的中间件。它基于 serve-static，负责在 Express 应用中提托管静态资源。

```js
var options = {
  dotfiles: 'ignore',
  etag: false,
  extensions: ['htm', 'html'],
  index: false,
  maxAge: '1d',
  redirect: false,
  setHeaders: function (res, path, stat) {
    res.set('x-timestamp', Date.now());
  }
}

app.use(express.static('public', options));
```

- 第三方中间件 - 安装所需功能的 node 模块，并在应用中加载，可以在应用级加载，也可以在路由级加载。

```js
var express = require('express');
var app = express();
var cookieParser = require('cookie-parser');

// 加载用于解析 cookie 的中间件
app.use(cookieParser());
```

## 配置

- 可以在使用中间件的时候指定运行环境，区分中间件是否运行在开发环境或生产环境。


![](/images/2017-09-10-16-16-10.jpg)

- 在configure函数外设置的中间件或者路由适用于所有的配置环境。
- 在某个特定的配置环境下运行应用，可以通过设置NODE_ENV环境变量达到目的


![](/images/2017-09-10-16-17-03.jpg)

## 中间件的执行顺序


![](/images/2017-09-10-16-26-21.jpg)

如果将中间件移动到end函数调用之后，中间件不会执行

## 静态文件处理
express(通过connect)提供了static中间件组件，可以为我们完成静态文件的处理

存在如下目录结构，并添加static中间件的使用：
![](/images/2017-09-10-16-32-01.jpg)

访问`http://localhost:8080/china2009/UNADJUSTEDNONRAW_thumb_4bd.jpg`即可访问文件

*注意：static中间件没有提供任何安全机制*

可以借助__dirname设置路径：

## POST数据、cookie和session（内容针对express 4之前的版本，即基于connect，支持内置中间件的版本）

中间件可以帮助我们处理body，cookie和session的管理

![](/images/2017-09-10-16-38-16.jpg)

- bodyParser中间件会解析所有的请求体数据，并放置到req.body对象
- cookieParser也是类似的工作原理，但是会把值放到req.cookies

设置session则会稍微复杂一点，需要提供存储session的空间和加密值所需的密钥

![](/images/2017-09-10-16-39-11.jpg)

- 如果期望每次重启服务器之后都能保存住session，可以在npm上找到一些其他的存储方式，如使用外部存储的memcache或者redis
- 当添加session功能之后，session数据会填充到req.session中
- 想添加新的seesion数据，只需在该对象中添加新的值即可。


![](/images/2017-09-10-16-40-37.jpg)

## 文件上传

- express的bodyParser中间件组件已经自动添加了connect multipart组件，可以提供文件上传功能。
- 如果使用multipart/form-data格式向应用上传文件，可以在req.files对象中找到请求中包含的所有上传文件

![](/images/2017-09-10-16-41-28.jpg)

可以用curl测试该功能，可使用-F参数(表单数据)在请求中包含上传文件，使用@操作符指定上传的文件名


![](/images/2017-09-10-16-50-41.jpg)

## 对PUT和DELETE更友好的浏览器支持

在一些老版本的浏览器(IE 10之前的版本)中delete和put不能正常工作。究其原因，是这些浏览器中XmlHttpRequest对象的实现不支持AJAX请求中的 PUT和DELETE方法。

一个通用且优雅的解决方案是添加一个真正想用的方法到新的请求头X-HTTP-Method-Override

![](/images/2017-09-10-16-52-25.jpg)

要想在服务器上支持这种解决方案，需要添加methodOverride中间件

## 压缩输出

- 为了减少带宽成本，很多Web服务器都会在将数据发送到客户端之前，使用gzip或者其他类似的算法压缩输出数据
- 前提条件是客户端需要在HTTP中使用请求头Accept-Encoding表明能够接收压缩后的数据
- 如果客户端提供了该请求头且服务器也支持压缩功能，则需要在输出的响应头Content-Encoding中指定合适的算法

![](/images/2017-09-10-16-54-20.jpg)

唯一缺点就是它会影响开发测试(curl只会打印出压缩后的二进制数据，不够直观)，因此，一般只会在生产环境下使用该中间件


![](/images/2017-09-10-16-55-08.jpg)

## 错误处理

尽管我们可以对每个请求单独进行错误处理，但有时也希望有一个全局的错误处理来解决一些常见问题。

![](/images/2017-09-10-16-56-57.jpg)

该方法会放置到所有其他中间件和路由函数的后面，从而成为express在应用中调用的最后一个函数。

Node.js可以通过process对象在全局应用范围内指定错误处理函数:

![](/images/2017-09-10-16-57-35.jpg)


# 数据库I:NoSQL(MongoDB)

> MongoDB，它能快速简单地直接将JSON数据序列化到数据库中

## 安装MongoDB

Mac下使用brew进行安装：

```shell
$ brew install mongodb
```
安装完成后，通过mongod启动数据库，默认监听27017端口：

![](/images/2017-09-10-17-05-08.jpg)

## 连接到mongodb


![](/images/2017-09-10-17-06-26.jpg)

```
var db = require('mongoskin').db('localhost:27017/animals');

db.collection('mamals').find().toArray(function(err, result) {
  if (err) throw err;
  console.log(result);
});
```
