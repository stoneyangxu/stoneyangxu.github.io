---
layout: post
title:  "Node.js硬实战 - 构建精简网络应用"
date:   2017-09-16 01:46:52
categories: 读书笔记
---

> 网络：构建精简的网络应用

- - - - -
# 第九章 构建精简的网络应用
- - - - -

- 使用Node开发客户端
- 浏览器的Node
- 服务端技术和WebSockets
- 应用从Express3迁移到Express4
- 测试Web应用
- 全栈框架和实时服务

- - - - -

# 前端技术

## 快速的静态网站服务器

仅仅希望构建一个服务器，提供*静态网站*访问能力或者*单页面Web应用*

大多数web开发任务都是*从服务器上获取某些文件开始*

在所谓的*无服务应用*上，通常是通过工程化构建工具，将客户端文件*打包*处理后，通过简单的静态服务器提供web访问能力

### 方案一：简单的connect脚本(connect 2.30.2)

提供当前目录的静态访问能力。

```js
const connect = require('connect')

connect.createServer(
    connect.static(__dirname)
).listen(8080)
```

### 方案二：通过命令行工具

```shell
$ yarn global add glance
$ glance 
```

### 方案三：使用Grunt的任务执行器

- 安装依赖
```shell
$ yarn global add grunt-cli
$ yarn add grunt grunt-contrib-connect --dev
```

- 创建Gruntfile.js文件
```js
module.exports = function (grunt) {
    grunt.loadNpmTasks('grunt-contrib-connect')

    grunt.initConfig({
        connect: {
            server: {
                options: {
                    port: 8080,
                    base: './',
                    keepalive: true
                }
            }
        }
    })

    grunt.registerTask('default', ['connect:server'])
}
```

- 命令行执行grunt调用默认的default任务

![](/images/2017-09-20-23-51-08.jpg)

## 在Node中使用DOM

在Node中模拟浏览器，例如要编写Web爬虫，浏览器处理提供JavaScript运行环境，更重要的是提供了*文档对象模型（DOM）*的API

使用*cheerio*在Node中复用依赖于DOM的客户端代码，或者渲染整个web页面内容

```js
const cheerio = require('cheerio')

const $ = cheerio.load('<p class="info">Welcome</p>')
console.log($('.info').text())
```

# 服务端技术

## Express的路由分离

随着应用规模的扩大，将所有路由都定义在app.js中会导致文件臃肿并且难以维护。

基于express内置的路由系统，可以按模块来组织路由。

- 新建birds.js文件
```js
var express = require('express');
var router = express.Router();

// middleware that is specific to this router
router.use(function timeLog(req, res, next) {
    console.log('Time: ', Date.now());
    next();
});
// define the home page route
router.get('/', function(req, res) {
    res.send('Birds home page');
});
// define the about route
router.get('/about', function(req, res) {
    res.send('About birds');
});

module.exports = router;
```

- 在app.js中应用路由
```js
var express = require('express');
var birds = require('./birds')

var app = express();

app.get('/', function (req, res) {
    res.send('Hello World!');
});

app.use('/birds', birds);

app.listen(3000, function () {
    console.log('Example app listening on port 3000!');
});
```

## 自动重启服务器

使用第三方工具检查文件变更并重启web应用

- 原生实现

```js
const fs = require('fs')
const exec = require('child_process').exec

function watch() {
    const child = exec('node index.js')
    const watcher = fs.watch(__dirname + '/index.js', function(event) {
        console.log('file changed, reloading')
        child.kill()
        watcher.close()
        watch()
    })
}

watch()
```

- 但是，web应用通常由多个文件组成，递归以及多文件监听会变得复杂，需要其他三方件的帮助

### 使用nodemon
[nodemon](https://github.com/remy/nodemon)拥有良好的特性和众多配置项来控制web应用的重启


- 创建nodemon.json

```json
{
  "ignore": [
    ".git",
    "node_modules"
  ],
  "execMap": {
    "js": "node --harmony"
  },
  "watch": [
    "index.js",
    "birds.js"
  ],
  "env": {
    "NODE_ENV": "development"
  },
  "ext": "js json"
}
```

- 执行*nodemon index.js*启动应用


![](/images/2017-09-21-00-43-21.jpg)

## 配置web应用

在开发环境、测试环境、生产环境使用不同的配置项

使用JSON配置文件、环境变量或者一个模块来管理不同的配置

### 基于环境变量

```js
if (app.get('env') === 'development') {
    // ...
} else {
    // ...
}
```

根据NODE_ENV变量区分配置项

```shell
$ NODE_ENV=production node index.js
```

### 使用配置文件管理

```js
const config = {
    development: require('./development.json'),
    production: require('./production.json'),
    test: require('./test.json')
}

module.exports = config[process.env.NODE_ENV || 'development']
```

### 使用第三方模块

```shell
$ yarn add nconf
```

```js
var nconf = require('nconf')

// 命令行 > 环境变量 > 配置文件
nconf.argv().env().file({file: 'config.json'})

app.listen(nconf.get('port'), function () {
    console.log('Example app listening on port 3000!');
});
```

## 构建restful服务

RESTful架构，就是目前最流行的一种互联网软件架构。它结构清晰、符合标准、易于理解、扩展方便，所以正得到越来越多网站的采用。

REST，即Representational State Transfer(表现层状态转化)

"表现层"其实指的是"资源"（Resources）的"表现层"。

所谓*"资源"*，就是网络上的一个实体，或者说是网络上的一个具体信息。它可以是一段文本、一张图片、一首歌曲、一种服务

可以用一个URI（统一资源定位符）指向它，我们把"资源"具体呈现出来的形式，叫做它的"表现层"（Representation）。

资源具体表现形式，应该在HTTP请求的头信息中用*Accept和Content-Type*字段指定，这两个字段才是对"表现层"的描述

互联网通信协议HTTP协议，是一个无状态协议。这意味着，所有的状态都保存在服务器端。因此，如果客户端想要操作服务器，必须通过某种手段，让服务器端发生"状态转化"（State Transfer）。而这种转化是建立在表现层之上的，所以就是"表现层状态转化"。

分别对应四种基本操作：*GET用来获取资源，POST用来新建资源（也可以用于更新资源），PUT用来更新资源，DELETE用来删除资源*。

- [理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful)

### 使用express构建restful服务

#### 明确目标

- 处理CRUD（下面讲用bears）
- 加一个标准的URL（http://example.com/api/bears）和(http://example.com/api/bears/:bear_id)
- 用相应的http的动词达到rest效果（GET, POST, PUT, DELETE）
- 返回JSON数据
- 在控制台打印所有的请求

#### 使用express初始化项目

- 初始化依赖，加入express

```shell
$ yarn init
$ yarn add express body-parser
```

- 创建app.js文件，利用官方例子创建最简单的应用

```js
var express = require('express');
var app = express();

app.get('/', function (req, res) {
    res.send('Hello World!');
});

app.listen(3000, function () {
    console.log('Example app listening on port 3000!');
});
```

- 加入nodemon监听文件变化，重启服务

```shell
$ yarn add nodemon --dev
```

- 创建nodemon.json文件

```json
{
  "restartable": "rs",
  "verbose": true,
  "env": {
    "NODE_ENV": "development",
    "PORT": "3000"
  },
  "execMap": {
    "": "node --debug",
    "js": "node --debug"
  },
  "watch": [
    "app/",
    "bin/",
    "routes/",
    "views/",
    "app.js"
  ],
  "ignore": [
    ".git",
    ".idea",
    "node_modules"
  ],
  "ext": "js jade"
}
```

- 执行nodemon启动应用（注意将package.json中的main字段修改为app.js）

#### 添加bodyparser中间件来解析请求中的json数据

```js
bodyParser = require('body-parser');

// 给app配置bodyParser中间件
// 通过如下配置再路由种处理request时，可以直接获得post请求的body部分
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());
```

#### 创建路由模块

```js
// API路由配置
var router = express.Router();              // 获得express router对象
router.get('/', function(req, res) {
    res.json({ message: 'hooray! welcome to our api!' });
})

// 注册路由
// 所有的路由会加上“／api”前缀
app.use('/api', router)

```

#### 使用[POSTMAN](https://www.getpostman.com/)来测试服务

![](/images/2017-09-22-00-06-25.jpg)

#### 创建数据库，用web应用连接数据库

```shell
$ yarn add mongoose
```

```js

// app.js
var mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/bears');
```

#### 创建数据模型

```js
// model/bears.js
var mongoose   = require('mongoose');
var Schema     = mongoose.Schema;

var BearSchema = new Schema({
    name: String
});

module.exports = mongoose.model('Bear', BearSchema);
```

#### 定义路由

![](/images/2017-09-22-00-07-00.jpg)

#### 新增router/bears.js文件，声明bears路由和数据保存操作

```js
const express = require('express')
const Bear = require('../model/bears')

const router = express.Router()

// middleware that is specific to this router
router.use(function timeLog(req, res, next) {
    console.log('Time: ', Date.now());
    next();
});

router.route('/bears')
    .post((req, res) => {
        const bear = new Bear();      // 创建一个Bear model的实例
        bear.name = req.body.name;  // 从request取出name参数的值然后设置bear的name字段

        // 保存bear，加入错误处理，即把错误作为响应返回
        bear.save(function(err) {
            if (err)
                res.send(err);

            res.json({ message: 'Bear created!' });
        });

    })

module.exports = router
```

![](/images/2017-09-22-00-18-46.jpg)

#### 新增get方法

```js
    .get(function(req, res) {
        Bear.find(function(err, bears) {
            if (err)
                res.send(err);

            res.json(bears);
        });
    });
```


![](/images/2017-09-22-00-19-13.jpg)

#### 为单条记录创建GET、DELETE和PUT请求

```js

router.route('/bears/:bear_id')
    .get(function(req, res) {
        Bear.findById(req.params.bear_id, function(err, bear) {
            if (err) res.send(err);
            res.json(bear);
        });
    })
    .put((req, res) => {
        Bear.findById(req.params.bear_id, function(err, bear) {
            if (err) res.send(err);
            
            bear.name = req.body.name
            
            bear.save((err) => {
                if (err) res(err)
                res.json({ message: 'Bear updated!' });
            })
        });

    })
    .delete(function(req, res) {
        Bear.remove({
            _id: req.params.bear_id
        }, function(err, bear) {
            if (err)
                res.send(err);

            res.json({ message: 'Successfully deleted' });
        });
    });


```

![](/images/2017-09-22-00-21-33.jpg)

![](/images/2017-09-22-00-24-12.jpg)

![](/images/2017-09-22-00-25-32.jpg)

## 全栈框架

- MEAN: MongoDB, Express, Angular, Node
- Linnovate 使用Mongoose处理数据层，Passport处理身份认证，使用Bootstrap作为UI
- Derby 使用Racer代替Mongoose处理数据层
- Kraken 基于express、子目录、控制器、Grunt和测试
- Meteor 使用MongoDB，基于发布/订阅模式，拥抱反射范式，在桌面开发领域相当流行