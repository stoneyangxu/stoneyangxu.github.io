---
layout: post
title:  "Node.js硬实战 - 网络"
date:   2017-09-14 23:37:54
categories: 读书笔记
---

> 网络：Node世界真正的“Hello World”

- - - - -
# 第七章 网络：Node真正的“Hello World”
- - - - -

Node.js平台的卖点就是开发*快速稳定*的网络应用

- 网络的概念和与Node的关系
- TCP、UDP、HTTP客户端和服务端
- DNS
- 网络加密

- - - - -

## Node中的网络

### 网络技术

- Layer - 代表一个逻辑组的*网络协议切片*，我们工作在应用层，是顶层；物理层是底层
- HTTP - 超文本传输协议，一个基于TCP的应用层*客户端-服务端协议*
- TCP - 传输控制协议，允许*客户端和服务端的双向通信*，目的是创建HTTP一样的应用层协议
- UDP - 用户数据报协议，一个*轻量*协议，在*期望速度，而非可靠*的时候选择他
- Socket - *一个IP和一个端口号结合*通常被成为一个socket
- Packet - TCP数据包也被看作*一个数据块和一个首部*
- Datagram - UDP相当于一个包
- MTU - 最大传输单元，一个协议数据单元的最大尺寸，每个层的MTU不同

### TCP/IP & UDP

- 最重要且最早被定义的协议
- 使用net模块创建TCP连接
- dns模块能够查询ipv4和ipv6
- 网络模块能够子啊ipv4和ipv6网络发送和接收数据
- TCP和UDP都使用了相同的网络层——IP
- TCP是面向连接且可靠的字节流服务
- UDP是基于数据报的，而且不能保证数据传输
### Sockets
- 网络的基础单元，TCP和UDP都有sockets
- TCP和UDP可以叠加使用相同的端口号，典型的如DNS
- Node中可以使用net模块创建TCP sockets，UDP使用dgram

## Node网络模块

- DNS - dns模块来查找和处理地址
- HTTP - http模块是基于net、stream、buffer、events模块的
- Encryption - 使用TLS加密TCP连接，Node中的tls是基于OpenSSL的。

## TCP客户端和服务端

### 创建TCP服务端和客户端

- 使用net.createServer创建服务，调用server.listen绑定到一个端口
- 使用命令行工具telnet或者进程内的客户端连接（net.connect）

```js
const net = require('net')

let clients = 0

const server = net.createServer((client) => {

    clients++

    const clientId = clients

    console.log(`Client connected: ${clientId}`)

    client.on('end', () => {
        console.log(`Client disconnected: ${clientId}`)
    })

    client.write(`Welcome client: ${clientId}`)
    client.pipe(client) // 将欢迎信息返回给客户端
})

server.listen(8000, () => {
    console.log('Server started on port 8000')
})
```

- 使用telnet连接到服务端

```shell
$ telnet localhost 8000
```


![](/images/2017-09-15-00-11-21.jpg)

### 使用客户端测试TCP服务端

- 带有断言的服务器和客户端
- 客户端使用net.connect创建连接
- 客户端和服务端可以*同时*运行在单进程内
- 客户端和服务端都很容易支持*单元测试*

```js
const net = require('net')
const assert = require('assert')

let clients = 0
let expectedAssertions = 2 // 预期的断言数

const server = net.createServer((client) => {

    clients++

    const clientId = clients

    console.log(`Client connected: ${clientId}`)

    client.on('end', () => {
        console.log(`Client disconnected: ${clientId}`)
    })

    client.write(`Welcome client: ${clientId}`)
    client.pipe(client) // 将欢迎信息返回给客户端
})

server.listen(8000, () => {
    console.log('Server started on port 8000')

    runTest(1, () => {
        runTest(2, () => {
            console.log('Tests finished')
            assert.equal(0, expectedAssertions)
            server.close()
        })
    })
})


const runTest = (expectedId, done) => {
    const client = net.connect(8000)

    client.on('data', (data) => {
        const expected = `Welcome client: ${expectedId}`
        assert.equal(expected, data.toString())

        expectedAssertions--
        client.end()
    })

    client.on('end', done)
}
```

### 改进实时性低的应用

- 要提高一个实时应用的*连接等待时间*
- 使用socket.setNoDelay()开启TCP_NODELAY
- Nagle算法：当一个连接有*未确认*的数据，小片段应该*保留*，当足够的数据被*收件人确认*，这些小片段将被*分批*成能够被传输的*更大的片段*
- 理想情况下，收集小片段能够*减少拥堵*
- 在互动应用场景下，小分片应该被实时发送

```js
const net = require('net')

const server = net.createServer((client) => {
    client.setNoDelay(true) // 关闭Nagle算法

    client.write('377323467283822324', 'binary') // 强制使用二进制传输

    console.log('server connected')

    client.on('end', () => {
        console.log('server disconnected')
        server.unref() // 客户端断开时，保证服务器关闭
    })

    client.on('data', (data) => {
        process.stdout.write(data.toString())
        client.write(data.toString())
    })
})

server.listen(8000, () => {
    console.log('server started')
})
```
## UDP客户端和服务端

相对TCP来说，UDP是更简单的协议，他是*无状态*的，如果想要更高的传输效率并且对*完整性*要求不高，可以使用UDP。例如：流媒体协议和游戏

### 通过UDP传输文件

通过一个*流*发送到UDP服务器

```js
const dgram = require('dgram')
const fs = require('fs')

const port = 41230
const defaultSize = 16

class Client {
    constructor(remoteIp) {
        this.remoteIp = remoteIp
        this.inStream = fs.createReadStream(__filename) // 读取自身所在文件
        this.socket = dgram.createSocket('udp4')

        this.inStream.on('readable', () => {
            this.sendData() // 开始发送数据
        })
    }

    sendData() {
        const message = this.inStream.read(defaultSize) // 读取指定大小
        if (!message) { // 传输完毕时，关闭socket
            return socket.unref()
        }

        socket.send(message, 0, message.length, port, this.remoteIp, (err, data) => {
            this.sendData() // 递归发送剩余数据
        })

    }
}

class Server {
    constructor() {
        this.socket = dgram.createSocket('udp4')

        this.socket.on('message', (msg, rinfo) => {
            process.stdout.write(msg.toString())
        })

        this.socket.on('listening', () => {
            console.log(`Server ready:`, this.socket.address())
        })

        this.socket.bind(port)
    }
}

if (process.argv[2] === 'client') {
    new Client(process.argv[3])
} else {
    new Server()
}
```

- 首先运行服务端

```shell
$ node udp-client-server.js server
```

- 其次运行客户端

```shell
$ node udp-client-server.js client localhost
```

- udp-client-server.js自身被发送给服务端

### UDP客户端服务应用

UDP经常被用于*查询-响应*协议，比如DNS和DHCP

- 已经有一个UDP服务来响应和请求，希望将消息发送回客户端
- 通过message事件的*rinfo*参数，rinfo参数包含message事件以及相关细节，可以利用他来传输数据
- TCP中建立的是*双向*事件流，通过*client.write*即可向客户端写入数据

- 如下所示的简单聊天应用：

```js
const assert = require('assert')
const dgram = require('dgram')
const readline = require('readline')

const port = 41234

class Client {
    constructor(remoteIp) {
        this.remoteIp = remoteIp

        this.socket = dgram.createSocket('udp4')

        this.rl = readline.createInterface(process.stdin, process.stdout) // 处理用户输入和输出

        this.socket.send(new Buffer('<JOIN>'), 0, 6, port, this.remoteIp) // 发送加入信息

        this.rl.setPrompt('Message>') // 显示用户提示
        this.rl.prompt()

        this.rl.on('line', (line) => {
            this.sendData(line) // 用户输入后，发送信息
        }).on('close', () => {
            process.exit(0)
        })

        this.socket.on('message', (msg, rinfo) => {
            console.log(`\n<${rinfo.address}>`, msg.toString()) // 收到消息后打印来源和内容
            this.rl.prompt()
        })
    }

    sendData(message) {
        this.socket.send(new Buffer(message), 0, message.length, port, this.remoteIp, (err, bytes) => {
            console.log('Send: ', message)
            this.rl.prompt()
        })
    }
}

class Server {
    constructor() {
        this.clients = []
        this.server = dgram.createSocket('udp4')

        this.server.on('message', (msg, rinfo) => {
            const clientId = rinfo.address + ':' + rinfo.port // 获取用户地址
            msg = msg.toString()

            if (!this.clients[clientId]) {
                this.clients[clientId] = rinfo // 如果用户第一次注册，保存注册信息
            }

            for (let client in this.clients) {
                if (client !== clientId) { // 消息广播给除自己之外的其他人
                    const clientRInfo = this.clients[client]
                    this.server.send(
                        new Buffer(msg), 0, msg.length,
                        clientRInfo.port, clientRInfo.address, // 消息发送给其他客户端
                        (err, bytes) => {
                            if (err) console.error(err)
                            console.log(`Bytes sent: `, bytes)
                        })
                }
            }
        })

        this.server.on('listening', () => {
            console.log('Server ready:', this.server.address())
        })

        this.server.bind(port)
    }
}

module.exports = {
    Client,
    Server,
}

if (!module.parent) {
    switch (process.argv[2]) {
        case 'client':
            new Client(process.argv[3])
            break
        case 'server':
            new Server()
            break
        default:
            console.log('unknown option')
            break
    }
}
```

![](/images/2017-09-15-01-27-13.jpg)

## HTTP客户端和服务端

HTTP是基于TCP的无状态协议

### HTTP服务器

- http.createServer方法是从net.Server创建的一个新的http.Server对象的捷径
- Node HTTP处理*响应码*的重点是解析

```js
const assert = require('assert')
const http = require('http')

const server = http.createServer((req, res) => {
    res.writeHead(200, { 'Content-Type': 'text/plain'})
    res.write('Hello World!\r\n')
    res.end()
})

server.listen(8000, () => {
    console.log('Listening on port 8000')
})

const req = http.request({
    port: 8000
}, (res) => {
    console.log('HTTP headers: ', res.headers)
    res.on('data', (data) => {
        console.log('Body: ', data.toString())

        assert.equal('Hello World!\r\n', data.toString())
        assert.equal(200, res.statusCode)
        server.unref()
    })
})

req.end() // send request
```

### 重定向
- http模块提供一个API来处理HTTP请求，但是不能处理重定向
- 可以使用三方模块*[Mikeal Rogers](https://github.com/request/request)*进行处理
- 下述例子中，使用js来维护*跨多个请求的状态*，使得*重定向*被正确执行
- HTTP标准定义了重定向发生时的*状态吗*， 也指出*客户端应该检测循环重定向*
    - 300 - 多重选择
    - 301 - 永久移动到新位置
    - 302 - 找到重定向跳转
    - 303 - 参见其他信息
    - 304 - 没有改动
    - 305 - 使用代理
    - 307 - 临时重定向

```js
const http = require('http')
const https = require('https')
const url = require('url')

let request

class Request {
    constructor() {
        this.maxRedirects = 10
        this.redirects = 0
    }

    get(href, callback) {
        const uri = url.parse(href)
        const options = {host: uri.host, path: uri.path}
        const httpGet = uri.prototype === 'http' ? http.get : https.get // 根据协议判断使用哪个方法

        console.log('GET: ' + href)

        const processResponse = (response) => {
            console.log(response)
            if (response.statusCode >= 300 && response.statusCode < 400) { // 如果是重定向请求
                if (this.redirects >= this.maxRedirects) {
                    this.error = new Error(`Too many redirects for ${href}`)
                } else {
                    this.redirects++ // 记录重定向次数
                    href = url.resolve(options.host, response.headers.location)

                    return this.get(href, callback) // 重定向时，递归调用
                }
            }

            // 非重定向请求时，记录请求信息
            response.url = href
            response.redirects = this.redirects

            console.log('Redirected: ', href) // 输出最终地址

            const end = () => {
                console.log('Connection ended')
                callback(this.error, response)
            }

            response.on('data', (data) => { // 获取真实数据
                console.log('Got data, length: ', data.length)
            })

            response.on('end', end.bind(this))
        }

        httpGet(options, processResponse.bind(this)).on('error', (err) => {
            callback(err)
        })
    }
}

request = new Request()
request.get('http://baidu.com/', (err, res) => {
    if (err) {
        console.error(err)
    } else {
        console.log('Fetched URL: ', res.url, ' with ', res.redirects, ' redirects')
        process.exit(0)
    }
})
```
### HTTP代理

- ISP使用透明代理，使得网络更加高效
- 企业系统管理员使用*缓存代理*来减少带宽使用
- web应用程序利用代理来*提高性能*
- 下述例子中，使用http模块创建一个简单的HTTP代理：

```js
const http = require('http')
const url = require('url')

http.createServer((req, res) => {

    console.log('start request: ', req.url)

    const options = url.parse(req.url) // 解析原始请求头
    options.headers = req.headers

    const proxyRequest = http.request(options, (proxyResponse) => { // 创建一个代理请求
        proxyResponse.on('data', (chunk) => { // 收到响应时，转发给原始请求的response
            console.log('proxyResponse length: ', chunk.length)
            res.write(chunk, 'binary')
        })

        proxyResponse.on('end', () => { // 响应结束时，结束原始响应
            console.log('proxied request ended')
            res.end()
        })

        res.writeHead(proxyResponse.statusCode, proxyResponse.headers) // 将响应头，写入原始响应头
    })

    req.on('data', (chunk) => { // 收到原始请求时，通过代理请求进行转发
        console.log('in request length: ', chunk.length)
        proxyRequest.write(chunk, 'binary')
    })

    req.on('end', () => { // 原始请求结束时，结束代理请求的发送
        console.log('original request ended')
        proxyRequest.end()
    })

}).listen(8080)
```


![](/images/2017-09-15-02-14-13.jpg)

## 创建DNS请求

- dns.lookup 提供更友好的API
- dns.resolve 使用更快的库

```js
const dns = require('dns')

console.time('lookup')

dns.lookup('www.manning.com', (err, address) => {
    if (err) console.log(err)

    console.log(address)
    console.timeEnd('lookup')
})

console.time('resolve')
dns.resolve('www.manning.com', (err, address) => {
    if (err) console.log(err)

    console.log(address)
    console.timeEnd('resolve')
})
```


![](/images/2017-09-15-23-54-53.jpg)

## 加密

- 加密模块tls， 使用OpenSSL安全传输层套接字（TLS/SSL）
- 每个客户端和服务端都拥有一个*私钥*，服务器可以使用*公钥*
### 一个加密都TCP服务器

- 使用tls模块开启一个客户端和服务端，使用openssl创立所需要的证书文件
- 公钥加密依赖于*公钥-私钥对*，以及一个附加的*证书验证公钥（CA）*
- 使用openssl命令行工具生成所需的文件
    - genrsa - 生成一个RSA证书，这是我们的私钥
    - req - 创建一个CASR
    - x509 - 使用CSR产生一个公钥签署的私钥
```shell
$ openssl genrsa -out server.pem 1024 # 使用1024比特创建服务器的私钥
$ openssl req -new -key server.pem -out server-csr.pem # 创建CSR， 需要输入主机名（Common Name），通过hostname查询
$ openssl x509 -req -in server-csr.pem -signkey server.pem -out server-cert.pem # 签发服务器私钥

$ openssl genrsa -out client.pem 1024 # 创建客户端私钥
$ openssl req -new -key client.pem -out client-csr.pem # 创建客户端CSR， 需要输入主机名（Common Name），通过hostname查询
$ openssl x509 -req -in client-csr.pem -signkey client.pem -out client-cert.pem # 签发客户端私钥，然后输出一个公钥
```

- 创建使用tls加密的服务器：

```js
const fs = require('fs')
const tls = require('tls')

const options = {
    key: fs.readFileSync('server.pem'), // 服务端私钥
    cert: fs.readFileSync('server-cert.pem'), // 公钥
    ca: [fs.readFileSync('client-cert.pem')], // 客户端验证证书
    requestCert: true, // 确保服务端和客户端都要检查
}

const server = tls.createServer(options, (clearTextStream) => {
    const authorized = clearTextStream.authorized ? 'authorized': 'unauthorized'

    console.log('Connected: ', authorized) // 展示服务器是否能够验证证书

    clearTextStream.write('Welcome!\n')
    clearTextStream.setEncoding('utf8')
    clearTextStream.pipe(clearTextStream)
})

server.listen(8080, () => {
    console.log('Server listening...')
})
```

- 一个使用tls加密的tcp客户端

```js
const fs = require('fs')
const os = require('os')
const tls = require('tls')

const options = {
    key: fs.readFileSync('client.pem'), // 服务端私钥
    cert: fs.readFileSync('client-cert.pem'), // 公钥
    ca: [fs.readFileSync('server-cert.pem')], // 服务端验证证书
    servername: os.hostname() // 把主机名作为服务器名称
}

const clearTextStream = tls.connect(8080, options, () => {
    const authorized = clearTextStream.authorized ? 'authorized': 'unauthorized'

    console.log('Connected: ', authorized)

    process.stdin.pipe(clearTextStream)
})

clearTextStream.setEncoding('utf8')
clearTextStream.on('data', (data) => {
    console.log(data)
})
```

### 加密的web服务器和客户端

使用*https模块*和*https.createServer*

- 服务端
```js
const fs = require('fs')
const https = require('https')


const options = {
    key: fs.readFileSync('server.pem'), // 服务端私钥
    cert: fs.readFileSync('server-cert.pem'), // 公钥
    ca: [fs.readFileSync('client-cert.pem')], // 客户端验证证书
    requestCert: true, // 确保服务端和客户端都要检查
}

const server = https.createServer(options, (req, res) => {
    const authorized = req.authorized ? 'authorized': 'unauthorized'

    console.log('Connected: ', authorized) // 展示服务器是否能够验证证书

    res.writeHead(200)
    res.write(`Welcome! You are ${authorized} \n`)
    res.end()
})

server.listen(8080, () => {
    console.log('Server listening...')
})
```


![](/images/2017-09-16-00-25-20.jpg)

- 客户端

```js
const fs = require('fs')
const https = require('https')
const os = require('os')

const options = {
    key: fs.readFileSync('client.pem'), // 服务端私钥
    cert: fs.readFileSync('client-cert.pem'), // 公钥
    ca: [fs.readFileSync('server-cert.pem')], // 客户端验证证书
    hostname: os.hostname(),
    port: 8080,
    path: '/',
    method: 'GET'
}

const req = https.request(options, (res) => {
    res.on('data', (d) => {
        process.stdout.write(d)
    })
})

req.end()

req.on('error', (e) => {
    console.error(e)
})

```


![](/images/2017-09-16-00-30-08.jpg)

- 使用浏览器访问时，由于未加载证书，提示未认证
- 使用自定义的客户端请求时，提示为已加载