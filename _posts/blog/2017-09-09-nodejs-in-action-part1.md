---
layout: post
title:  "NodeJs in Action (一)"
date:   2017-09-09 01:31:50
categories: 读书笔记
---

# 前言
- Node.js是专为编写网络和Web应用 而生的全新平台
- Node.js基于v8引擎带来了优越的性能
- Node.js的主要理论和创新就是它完全构 建在*事件驱动、非阻塞*的编程模型之上。
- 大量的三方模块给他的开发效率带来极大的提高
- Node.js采用单进程运行，占用的资源大大减少
- *Node.js适合与频繁的短时请求场景，对于密集计算长耗时的应用并不适合*
> 书籍源码：https://github.com/marcwan/LearningNodeJS

- - - - -
第一部分 基础
- - - - -
# 入门

## Node Shell - Node REPL（Read-Eval-Print-Loop）

安装Node.js之后，在控制台输入node命令进入Shell

输入命令并执行，控制台可以看到执行结果

最后一行的输出往往是指令的返回结果，下图中的console.log并没有返回值，显示为undefined

![](/images/2017-09-08-00-23-57.jpg)

如果想要退出REPL，只需要按下Ctrl+D

如果控制台中出现三个点（...），表示需要进一步输入，以完成表达式，便于多行输入。


![](/images/2017-09-08-00-27-32.jpg)

## 编辑并运行JS文件

创建js文件，通过 node file.js来执行：
![](/images/2017-09-08-00-28-43.jpg)

## 第一个web服务器

新建文件first-web-server.js

```js
const http = require('http') // 引入http模块

const processRequest = (req, res) => {
    const body = `Thank you for calling!`
    const bodyLength = body.length
    res.writeHead(200, { // 写入响应头
        'Content-Length': bodyLength,
        'Content-Type': 'text/plain'
    })
    res.end(body) // 写入响应内容
}

const server = http.createServer(processRequest) // 创建web服务和处理函数
server.listen(2000) // 启动并监听2000端口
```

执行node first-web-server.js启动web服务，并在页面访问


![](/images/2017-09-08-00-36-33.jpg)

或者通过curl查看请求的头信息

![](/images/2017-09-08-00-37-27.jpg)

### 过程
- 用户访问2000端口时，进入到processRequest函数
- 参数中带有请求信息req（ServerRequest实例）和响应对象res（ServerReponse实例）
- 从res中获取参数并通过res返回给最终用户
## 调试Node.js应用

执行`node debug first-web-server.js`启动调试模式

![](/images/2017-09-08-00-41-27.jpg)

### 添加断点
使用setBreakPoint(n)设置断点
![](/images/2017-09-08-00-44-33.jpg)

### 通过repl查看变量值
![](/images/2017-09-08-00-47-57.jpg)

### 使用vscode调试

调试面板中加在nodejs配置，修改文件路径
![](/images/2017-09-08-00-51-57.jpg)

在程序中添加断点后，在调试面板运行配置

![](/images/2017-09-08-00-53-06.jpg)

### 使用WebStorm调试

添加断点后，点击调试
![](/images/2017-09-08-00-56-09.jpg)

# Node.js中的特殊对象

- global - node中的全局对象，类似于web端的window对象
- console - 控制台对象
- process - 包含当前执行进程的信息和控制方法
# 异步编程

![](/images/2017-09-08-01-09-16.jpg)

## 在nodejs中，文件的读写是异步执行

执行如下代码：
```js
const fs = require('fs') // 加载文件模块

let fileHandler

const buffer = new Buffer(10000) // 创建Buffer缓存

fs.open('some.text', 'r', (handler) => { // 使用只读方式打开文件
    fileHandler = handler
})

fs.read(fileHandler, buffer, 0, 10000, null, () => { // 读取文件
    console.log(buffer.toString())
    fs.close(fileHandler, () => {})
})
```

![](/images/2017-09-08-01-16-39.jpg)

原因是：文件的打开操作`open`是异步执行的，当调用read的时候，尚未完成。
修复代码如下：

```js
const fs = require('fs') // 加载文件模块


const buffer = new Buffer(10000) // 创建Buffer缓存

fs.open('some.text', 'r', (err, handler) => { // 使用只读方式打开文件

    fs.read(handler, buffer, 0, 10000, null, (err, length) => { // 读取文件
        console.log(buffer.toString('utf8', 0, length)) 
        fs.close(handler, () => {})
    })
})
```

![](/images/2017-09-08-01-20-27.jpg)

*nodejs使用单线程队列机制，等待用户请求，将操作放入队列等待执行；当执行完成后，通过回调函数响应用户请求；同时清空队列，挂起等待新请求*

![](/images/2017-09-08-01-23-47.jpg)

# 错误处理与异步函数
考虑如下代码：
```js
try {
    setTimeout(() => {
        throw new Error('throw an error')
    }, 2000)
} catch(e) {
    console.log('i have caught an error')
}
```
我们希望捕获一个异常，但是实际情况是错误被直接抛出。


![](/images/2017-09-08-01-32-30.jpg)

实际情况是，setTimeout在try-cache上下文中执行，但是异步函数（throw语句）被加入到执行队列，2秒后在一个全新到上下文中执行。

在Node中经常使用其他的核心模式来处理错误

## 回调函数和错误处理
例如：
```js
fs.open('some.text', 'r', (err, handler) => { // 使用只读方式打开文件
    //...
})

```
err对象用来包含错误信息：
- 正常情况下err的值为null
- 错误情况下，err对象是Error实例，通常包含code和message字段以及其他的详细错误信息
## 异步函数中的this

虽然定义在对象中，但是在运行过程中指向调用者。
- 使用var self = this; 保存this的引用
- 使用ES6中的箭头函数
# 保持优雅——放弃控制权
Node采用单线程执行，在执行密集计算任务的时候，会出现程序假死：

```js
const len = 1e10

console.time()
for (let i = 0; i < len; i++) {
    Math.pow(2, 53)
}

console.timeEnd()
```

![](/images/2017-09-08-01-44-49.jpg)

通常情况下，类似的任务适合在其他平台实现，由nodejs进行调用。

或者利用全局对象process中的nextTick方法，通知系统在空闲时才来执行我的任务。

```js
const len = 1e10
const size = 1e4
let index = 0

console.time()

const calculate = () => {
    for (let i = index; i < index + size; i++) {
        Math.pow(2, 53)
    }

    if (i >= length) {
        console.log('finished')
    } else {
        index += size
        process.nextTick(calculate)
    }
}

console.timeEnd()

```

# 同步函数调用
Node有一些核心API的同步版本，尤其是在操作文件的API中：

```js
const fs = require('fs') // 加载文件模块


const buffer = new Buffer(10000) // 创建Buffer缓存

const handler  = fs.openSync('some.text', 'r') // 同步打开，返回文件描述符
const length = fs.readSync(handler, buffer, 0, 10000, null) // 同步读取，返回读取长度

console.log(buffer.toString('utf8', 0, length))

fs.closeSync(handler)
```
- - - - -
第二部分 提高
- - - - -

# 编写简单应用
> 本章中，我们将会编写一个JSON服务器，用以提供相册列表以及每个相册对应的照片列表等服务，最后，还会添加为相册重命名的功能。
> 我们会 理解JSON服务器运行的基本知识，这其中包含了服务器与HTTP进行交互的基础知识，例如GET和POST参数、头信息、请求和响应。

## 创建web服务

```js
const http = require('http')

const processRequest = (req, res) => {
    console.log(`Incoming request: ${req.method} ${req.url}`)
    res.writeHead(200, {'Content-Type': 'application/json'})

    res.end(JSON.stringify({error: null}) + '\n')
}

const server = http.createServer(processRequest)
server.listen(8080)
```

- 发起请求时，控制台打印请求信息

![](/images/2017-09-08-02-03-38.jpg)

- 页面得到响应

![](/images/2017-09-08-02-04-08.jpg)

## 创建相册目录

![](/images/2017-09-08-02-06-18.jpg)

## 首先要列出albums目录下的所有相册目录

```js
const loadAlbumList = (callback) => {
    fs.readdir('albums/', (err, files) => { // 遍历目录
        if (err) {
            callback(err)
            return
        }
        callback(null, files)
    })
}

loadAlbumList((err, files) => {
    console.log(files)
})
```

![](/images/2017-09-08-02-10-20.jpg)

异步调用中的callback就是Node应用编程的`核心技术`:告诉Node去做某件事情， 并在Node完成时告知Node将结果传递给谁

完整代码如下：
```js
const http = require('http')
const fs = require('fs')

const loadAlbumList = (callback) => {
    fs.readdir('albums/', (err, files) => { // 遍历目录
        if (err) {
            callback(err)
            return
        }
        callback(null, files)
    })
}

const processRequest = (req, res) => {
    console.log(`Incoming request: ${req.method} ${req.url}`)

    loadAlbumList((err, albums) => {
        if (err) {
            res.writeHead(503, {'Content-Type': 'application/json'})
            res.end(JSON.stringify(err) + '\n')
            return
        }

        res.writeHead(200, {'Content-Type': 'application/json'})
        res.end(JSON.stringify({error: null, albums}) + '\n')
    })
}

const server = http.createServer(processRequest)
server.listen(8080)
```

![](/images/2017-09-08-02-13-35.jpg)

# 异步与循环

当相簿中还存在其他文本文件时：

![](/images/2017-09-08-23-40-19.jpg)

我们的程序会返回：

![](/images/2017-09-08-23-40-48.jpg)

需要使用fs.stat对文件类型进行判断：

```js

const loadAlbumList = (callback) => {
    fs.readdir('albums/', (err, files) => { // 遍历目录
        if (err) {
            callback(err)
            return
        }

        const dirs = [];

        for (let i = 0; i < files.length; i++) {
            const file = files[i];
            
            fs.stat(`albums/${file}`, (err, stat) => {
                if (stat.isDirectory()) {
                    dirs.push(file)
                }
            })
        }

        callback(null, dirs)
    })
}
```

得到的结果却是：


![](/images/2017-09-08-23-45-13.jpg)

原因在于：`在异步编程模型下，大部分循环都不能与异步回调不能兼容`
- 当程序执行循环中的异步函数时，主线程不会等待，已经将空数组返回给回调
- 当fs.stat执行完毕，并将结果push到数组时，数组的结果已经没有人关心了

*此时只能使用递归来解决*

![](/images/2017-09-08-23-50-04.jpg)

```js

const loadAlbumList = (callback) => {
    fs.readdir('albums/', (err, files) => { // 遍历目录
        if (err) {
            callback(err)
            return
        }

        const dirs = [];

        (function iterator(i) {
            if (i < files.length) {

                const file = files[i]

                fs.stat(`albums/${file}`, (err, stat) => {

                    if (err) {
                        callback(err)
                        return
                    }

                    if (stat.isDirectory()) {
                        dirs.push(file)
                    }

                    iterator(i + 1)
                })
            } else {
                callback(null, dirs)
            }
        })(0)
    })
}
```

### 如果使用ES6的promise进行重构：

```js

const fileState = (file) => {
    return new Promise((resolve, reject) => {
        fs.stat(`albums/${file}`, (err, stat) => {
            console.log(stat)
            if (err) {
                reject(err)
            } else {
                resolve({
                    file,
                    isDir: stat.isDirectory()
                })
            }
        })
    })
}

const filterDirectories = (files, callback) => {
    const promiseList = files.map(file => fileState(file))

    Promise.all(promiseList).then(fileStats => {
        callback(null, fileStats.filter(f => f.isDir).map(f => f.file))
    }).catch(err => {
        callback(err)
    })
}

const loadAlbumList = (callback) => {
    fs.readdir('albums/', (err, files) => { // 遍历目录
        if (err) {
            callback(err)
            return
        }

        filterDirectories(files, callback)
    })
}
```

# 添加更多功能（代码回归到书主的示例，避免后续出现歧义）

> https://github.com/marcwan/LearningNodeJS/blob/master/Chapter04/05_multiple_requests.js
- 可用相册列表——调用/albums.json请求。
- 相册中的照片列表——调用/albums/album_name.json请求。
- 为请求添加.json后缀，强调当前编写的JSON服务器只用于这类请求。

![](/images/2017-09-09-00-22-49.jpg)

![](/images/2017-09-09-00-23-55.jpg)

# 更多请求的详情

- writeHead和end。对于传入的每个请求都必须调用一次且只能调用一次end方法
- HTTP状态码：http://en.wikipedia.org/wiki/List_of_HTTP_status_codes

# 提高灵活性:GET参数
> 当开始往相册中添加大量照片时，应用需要在一个“页面”中高 效地展示大量照片，所以我们需要为应用提供分页功能，而客户端需 要能够告诉我们需要多少张照片以及是哪一页的照片

```
http://localhost:8080/albums/china2009.json?page=1&page_size=2
```

- 添加Node内置 的url模块后，可以使用url.parse函数来提取核心的URL路径名以及查询参数
- url.parse函数可以更进一步给函数添加第二个参数——true，这个参数告诉url.parse函数解析查询字符串，并生成包含GET参数的对象

```js
const url = require('url')

function handle_incoming_request(req, res) {

    console.log('INCOMING REQUEST: ' + req.method + ' ' + req.url)

    req.parsed_url = url.parse(req.url, true)
    const core_url = req.parsed_url.pathname

    if (core_url == '/albums.json') {
        handle_list_albums(req, res)
    } else if (core_url.substr(0, 7) == '/albums'
        && core_url.substr(req.url.length - 5) == '.json') {
        handle_get_album(req, res)
    } else {
        send_failure(res, 404, invalid_resource())
    }
}

```

解析后的结果如下：

![](/images/2017-09-09-00-55-03.jpg)

在读取相簿内容时处理分页：

```js
function handle_get_album(req, res) {
    // format of request is /albums/album_name.json

    const getp = req.parsed_url.query
    const core_url = req.parsed_url.pathname
    let page_num = getp.page || 0
    let page_size = getp.page_size || 1000

    if (isNaN(page_num)) page_num = 0
    if (isNaN(page_size)) page_size = 0

    var album_name = core_url.substr(7, core_url.length - 12)
    load_album(album_name, page_num, page_size, (err, album_contents) => {
        if (err && err.error == 'no_such_album') {
            send_failure(res, 404, err)
        } else if (err) {
            send_failure(res, 500, err)
        } else {
            send_success(res, {album_data: album_contents})
        }
    })
}
```
- 通过core_url解析相簿名称
- load_album增加page_num, page_size参数

![](/images/2017-09-09-01-02-58.jpg)

![](/images/2017-09-09-01-02-32.jpg)

# 修改内容:POST数据
为了能够使用curl客户端发送数 据，我们需要做点事情
1. 设置HTTP方法参数为POST(或者PUT)。 
2. 为传入的数据设置Content-Type。 
3. 开始发送数据。

![](/images/2017-09-09-01-04-41.jpg)

## 服务端处理post请求

在请求处理时，处理重命名操作：

![](/images/2017-09-09-01-15-01.jpg)

## 处理post数据
> 程序为了获取POST数据，会使用一种叫做数据流的Node特 性。当使用Node的异步非阻塞特性时，数据流是传输大量数据的最 佳方式。

```js

function handle_rename_album(req, res) {

    // 1. Get the album name from the URL
    var core_url = req.parsed_url.pathname;
    var parts = core_url.split('/');
    if (parts.length != 4) {
        send_failure(res, 404, invalid_resource());
        return;
    }

    var album_name = parts[2];

    // 2. get the POST data for the request. this will have the JSON
    // for the new name for the album.
    var json_body = '';
    req.on('readable', () => {
        var d = req.read();
        if (d) {
            if (typeof d == 'string') {
                json_body += d;
            } else if (typeof d == 'object' && d instanceof Buffer) {
                json_body += d.toString('utf8');
            }
        }
    });

    // 3. when we have all the post data, make sure we have valid
    //    data and then try to do the rename.
    req.on('end', () => {
        // did we get a valid body?
        if (json_body) {
            try {
                var album_data = JSON.parse(json_body);
                if (!album_data.album_name) {
                    send_failure(res, 404, missing_data('album_name'));
                    return;
                }
            } catch (e) {
                // got a body, but not valid json
                send_failure(res, 403, bad_json());
                return;
            }

            // we have a proposed new album name!
            do_rename(album_name, album_data.album_name, (err, results) => {
                if (err && err.code == "ENOENT") {
                    send_failure(res, 403, no_such_album());
                    return;
                } else if (err) {
                    send_failure(res, 500, file_error(err));
                    return;
                }
                send_success(res, null);
            });
        } else {
            send_failure(res, 403, bad_json());
            res.end();
        }
    });
}


function do_rename(old_name, new_name, callback) {
    // Rename the album folder.
    fs.rename("albums/" + old_name,
        "albums/" + new_name,
        callback);
}
```
> 对于传入请求的主体中的每一块(组块)数据，传入 on('readable',...)的处理程序函数都会被调用。上面的代码中，首先通过read方法读取来自数据流的数据并将这些传入的数据添加到json_body变量的后面;然后，当监听到end事件，会得到这些结果字符串，并尝试去解析它。当给定的字符串不是一个合法的JSON 时，JSON.parse会抛出一个错误，所以必须用try/catch语句块封装 这些代码。

## 执行重命名

```
curl -s -X POST -H "Content-Type: application/json" -d '{"album_name": "japan2011"}' http://localhost:8080/albums/japan2010/rename.json
```

相册japan2010被重命名为japan2011

![](/images/2017-09-09-01-20-45.jpg)

## 通过表单提交的数据

![](/images/2017-09-09-01-21-28.jpg)

服务端收到的数据格式如下：

![](/images/2017-09-09-01-23-39.jpg)

解析时，需要借助querystring模块：

```js
const qs = require('querystring');
var POST_data = qs.parse(body);
```
将其转化为对象：

![](/images/2017-09-09-01-24-43.jpg)

> 示例源码：https://github.com/marcwan/LearningNodeJS/blob/master/Chapter04/09_form_data.js

