---
layout: post
title:  "HTML5 - WebSocket"
date:   2017-09-16 00:46:52
categories: HTML5
---

HTML5 WebSocket技术定义了客户端和服务器之间的*全双工*通信通道。避免了以往通过各种*hacks*方式的通信。

# 历史

在WebSocket技术出现之前，前端在处理一些*实时*更新的数据展现，例如：股票走势、位置信息、进度等业务场景时，往往通过：*要求用户不断刷新*、*通过定时器不停查询*或者*Comet的服务端推送*。

## 要求用户不断刷新

这是一个*不可理喻*的要求，特别对于广泛的大众用户或者非IT从业者来说。也许只有专业的IT从业者才能够理解并忍受如此糟糕的用户体验。

## 轮询

实现：前端通过JS脚本*定时发送HTTP请求*，返回数据后更新界面显示。

```js
setInterval(() => {
    
    fetch('url').then((response) => {
        // update page
    }) 
    
}, 5000) // every five seconds

```

轮询带来的问题：
- 客户端*无法预知*是否有新的数据
- 大量的*无效请求*增加*带宽消耗*
- *厚重*的HTTP请求头，*频繁*的连接创建和断开

## 长轮询

服务器对每个请求*保持一段时间*的*打开*状态，如果出现数据更新，会通过该连接发送信息，并关闭该连接。

虽然在一定程度上*减少了连接次数*，但是服务端保持连接会造成*资源消耗*，而且*无法保证消息返回顺序*，同样难于*管理和维护*


## iframe流

在页面中插入一个隐藏的iframe，利用其src属性在服务器和客户端之间建立一条长链接，服务器向iframe传输数据（通常是HTML，内有负责插入信息的javascript），来实时更新页面。  iframe流方式的优点是浏览器兼容好


![](/images/2017-09-17-15-59-39.jpg)

# WebSocket

WebSocket是Web应用程序的*传输协议*，类似于TCP，提供了*双向、按序到达*的数据流。

- [阮一峰老师的WebSocket教程](http://www.ruanyifeng.com/blog/2017/05/websocket.html)

- 在创建websocket通信时，客户端于服务器进行握手，将HTTP协议*升级为WebSocket协议*
- 建立连接时，使用*ws://或wss://*前缀作为url

![](/images/2017-09-17-16-06-00.jpg)

- 客户端使用时，基于socket对象的*事件*来进行管理和数据获取
- 没有同源限制，客户端可以与任意服务器通信。
- 使用WebSocket，避免了*循环请求的性能开销*，而且避免了*HTTP请求头造成的带宽消耗*

# 建立websocket服务器

基于*[nodejs-websocket](https://github.com/sitegui/nodejs-websocket)*

- 安装依赖

```shell
$ yarn add nodejs-websocket
```

- websocket-server.js

```js
const ws = require("nodejs-websocket")

// Scream server example: "hi" -> "HI!!!"
const server = ws.createServer(function (conn) {
    console.log("New connection")
    conn.on("text", function (str) {
        console.log("Received "+str)
        conn.sendText(str.toUpperCase()+"!!!")
    })
    conn.on("close", function (code, reason) {
        console.log("Connection closed")
    })
}).listen(8001)
```
- 启动服务
```shell
$ node websocket-server.js
```

# 客户端使用WebSocket获取数据

## 首先检查浏览器支持情况

```js
window.onload = () => {
    if (window.WebSocket) {
        createConnect()
    }
}
```

- 通过检查全局WebSocket是否存在，判断浏览器是否支持

![](/images/2017-09-17-16-38-34.jpg)

## 创建连接

```js
let socket = null

function createConnect() {
    socket = new WebSocket('ws://localhost:8001')
    bindEvents(socket)
}
```

## 绑定事件

```js
function bindEvents(socket) {
    socket.onopen = () => {
        appendMsg('connected')
    }

    socket.onclose = () => {
        appendMsg('disconnected')
    }

    socket.onerror = () => {
        appendMsg('error')
    }

    socket.onmessage = (e) => { 
        appendMsg(`Received: ${e.data}`)
    }
}
```

- onmessage - 收到服务器推送的数据时，更新界面显示

## 完整代码如下（为了便于演示，js代码嵌在html文件中）

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>WebSocket Client</title>
</head>
<body>

    <script type="text/javascript">

        let socket = null

        function createConnect() {
            socket = new WebSocket('ws://localhost:8001')
            bindEvents(socket)
        }

        function bindEvents(socket) {
            socket.onopen = () => {
                appendMsg('connected')
            }

            socket.onclose = () => {
                appendMsg('disconnected')
            }

            socket.onerror = () => {
                appendMsg('error')
            }

            socket.onmessage = (e) => {
                appendMsg(`Received: ${e.data}`)
            }
        }

        function appendMsg(msg) {
            const li = document.createElement('li')
            li.innerHTML = msg

            document.getElementById('result').appendChild(li)
        }

        function sendMessage() {
            const msg = document.getElementById('inputText').value
            appendMsg(`Send: ${msg}`)
            socket.send(msg)
        }

        window.onload = () => {

            if (window.WebSocket) {
                createConnect()
            }
        }

    </script>

    <div>
        <input type="text" id="inputText" />
        <input type="button" onclick="sendMessage()" value="Send"/>
    </div>

    <ul id="result">
    </ul>

</body>
</html>
```


![](/images/2017-09-17-16-45-08.jpg)

# 关注一下握手时的请求信息


![](/images/2017-09-17-16-47-31.jpg)

- 请求头信息包含websocket的协商

![](/images/2017-09-17-16-48-47.jpg)

- 与服务端建立连接后，返回101状态码


![](/images/2017-09-17-16-49-46.jpg)

# 其他资料

- [MDN WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
