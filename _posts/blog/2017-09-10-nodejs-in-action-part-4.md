---
layout: post
title:  "NodeJs in Action (四)"
date:   2017-09-10 17:19:23
categories: 读书笔记
---

- - - - -
第四部分 进阶篇
- - - - -

# 部署和开发

- 产品服务器上部署和运行Node应用的各种方式
- 如何利用拥有多核处理器的机器
- 如何在服务器上添加对虚拟主机的支持
- 添加SSH/HTTPS来确保应用安全
- 快速浏览在Windows和UNIX/Mac机器上跨平台开发Node应用时带来的问题

## 部署

运行nodejs程序，只需要执行：

```shell
$ node script_name.js
```

为了确保可靠性，在程序崩溃时能够快速恢复：

- 最简单到时通过while循环

![](/images/2017-09-10-17-23-38.jpg)

- 更进一步，我们可以使用tee命令将node进程的标准输出写入到日志文件中


![](/images/2017-09-10-17-24-30.jpg)

## 多处理器部署:使用代理
在每个想利用的内核上都运行node进程


![](/images/2017-09-10-17-26-39.jpg)

通过负载均衡器映射到不同nodejs进程到端口上

## 多服务器和会话
使用多个进程启动时，session需要共享，将session数据使用的内存存储放到存储池中，这样所有Node进程都可以访问它。

最佳候选方案是memcached和Redis，两者都是基于内存的键/值存储 (memory-based key/value stores)，可以在不同计算机之间传递数据。


![](/images/2017-09-10-17-29-04.jpg)

## 虚拟主机：在同一个服务器上运行多个网站

通过向HTTP请求中添加"Host:"头来实现虚拟主机，这是 HTTP/1.1的主要特性之一

express直接内置了运行多虚拟主机的功能。要让它工作，需要为每一个支持的主机创建一个express服务器，然后为虚拟服务器提供一个“主”服务器，用来将请求转发到合适的虚拟服务器。最后，使用vhost连接中间件组件将所有的服务器都组合起来


![](/images/2017-09-10-17-31-30.jpg)


![](/images/2017-09-10-17-31-54.jpg)

![](/images/2017-09-10-17-32-10.jpg)

如果想在浏览器中进行测试，则需要修改计算机的DNS条目。

## 代理服务器支持

虽然express内置支持虚拟主机，但一个进程运行多个Web应用程序还是会存在风险。

![](/images/2017-09-10-17-34-13.jpg)

## 使用HTTPS/SSL保障项目安全

要为应用添加SSL/HTTPS支持，首先需要生成一些测试证书，并为应用程序添加加密传输的支持。
所有的UNIX/Mac机器都带有叫做openssl的工具，可以用它来生成这两个文件。


![](/images/2017-09-10-17-35-25.jpg)

### 内置支持
通过Node提供的内置https模块，Node.js和express提供了对SSL/HTTPS数据流的支持

实际上我们是运行一个https模块服务器来监听HTTPS端口，然后当加密数据流(encrypted stream)经过握手和创建之后，https模块会将请求传输到express服务器。


![](/images/2017-09-10-17-37-40.jpg)

### 代理服务器支持

也可以使用强大的http-proxy模块来处理SSL/HTTPS传输
- “隐藏”在HTTPS代理服务器背后并让我们从乏味的加密工作中释放出来

![](/images/2017-09-10-17-39-10.jpg)

## 多平台开发

配置上的不同和路径区别。

### 位置和配置文件

Windows和类UNIX操作系统在不同的地方保存文件。一个能减轻该问题带来的影响的方法就是使用配置文件来存储这些地址

### 处理路径差异
- 绝大多数情况下，我们会发现Node接受斜杠(/)字符
- 对于那些必须去处理不同路径格式的情况，可以使用Node.js的path.sep属性

![](/images/2017-09-10-17-42-02.jpg)

- process全局对象总是能够告诉我们当前的运行平台

![](/images/2017-09-10-17-42-31.jpg)

# 命令行编程
- 在Node中运行命令行脚本的基本知识，包括一些常用的同步文件系统API
- 学习如何利用缓冲和无缓冲输入与用户交互
- 进程的创建和执行

## 运行命令行脚本

在shell中运行Node程序有两种方式：
- 指定node程序和需要运行的脚本：node something(.js)
- 直接修改文件的第一行代码，使其变为可执行文件：`#!/usr/local/bin/node`

> 操作系统会在/usr/local/bin目录下查找可执行的node，运行Node并将文件路径作为参数传递给Node
> 使用固定路径会导致程序无法移植，通常使用环境变量：`#!/usr/bin/env node` env命令会查询PATH环境变量，找到第一个node程序实例然后运行。

## 脚本和参数
- 如何向脚本传递参数
- 如何在脚本中访问参数

### 传递参数

如果一直是通过调用解释器来运行Node脚本(在所有平台下都有效)，就无需额外做任何事情——参数会直接传递给正在执行的脚本

### 访问参数

所有传递进来的参数都会被保存到全局对象process的argv数组中


![](/images/2017-09-10-17-56-10.jpg)

## 同步处理文件
fs中几乎每一个以func命名的API都有一个相应的叫做funcSync的API。

## 用户交互:标准输入和输出
stdin、stdout和stderr，分别代表了标准输入、标准输出和错误输出


![](/images/2017-09-10-17-59-13.jpg)

### 基本缓冲输入和输出
- 默认情况下，每次只能从stdin数据流中读取和缓冲一行数据
- 想要从stdin中读取数据，只有当用户按下回车(Enter)键，程序才会从输入流中读取整行数据
- 可以通过向stdin添加readable事件监听器实现
- 默认情况下，stdin输入流处于暂停状态。因此，需要调用resume函数才能开始从输入流中接收数据

![](/images/2017-09-10-18-01-47.jpg)

### 无缓冲输入
如果想让用户按下键盘以后程序立即响应，可以通过使用setRawMode函数来开启stdin输入流中的原始模式

![](/images/2017-09-10-18-56-20.jpg)

### Readline模块

![](/images/2017-09-10-18-56-59.jpg)

### 逐行提示
如果调用readline的prompt方法，该程序就会等待一行的输入(直到回车),如果需要继续监听事件，则需要再次调用prompt方法。


![](/images/2017-09-10-18-57-58.jpg)

一旦用户按下Ctrl+C，SIGINT事件就会被调用，那么就可以选择关掉或者恢复状态，继续监听

![](/images/2017-09-10-18-58-53.jpg)

readline模块另一个重要功能就是可以提问:

![](/images/2017-09-10-18-59-34.jpg)

# 进程处理

Node中还可以使用命令行(甚至在Web应用中)启动其他程序。使用child_process模块，有两种不同复杂程度的选项，我们将 从简单的exec开始。

## 简单进程执行
child_process模块中的exec函数会接收一个命令并在系统shell中执行


![](/images/2017-09-10-19-01-11.jpg)

```js
const exec = require('child_process').exec

if (process.argv.length !== 3) {
    console.log('Need a file name!')
    process.exit(-1)
}

const fileName = process.argv[2]
const cmd = process.platform === 'win32' ? 'type' : 'cat'

child = exec(`${cmd} ${fileName}`, (error, stdout, stderr) => {
    console.log(stdout)
    console.log(stderr)

    if (error) {
        console.log('error:', error)
        process.exit(-1)
    }
})
```

## 使用Spawn创建进程
创建进程的方式是使用child_process模块中的spawn函数。该函数拥有其创建出来的子进程的stdin和stdout的完整控制权，这样可以使用一些奇妙的功能，如将输出从一个子进程通过*管道*传入另一个子进程中

```js
const spawn = require('child_process').spawn

if (process.argv.length < 3) {
    console.log('Need script to run!')
    process.exit(-1)
}

const args = process.argv.slice(2)
console.log(args)

const node = spawn('node', [...args])

node.stdout.on('readable', () => {
    console.log(process.stdout.read())
})


node.stderr.on('readable', () => {
    console.log(process.stderr.read())
})


node.on('exit', (code) => {
    console.log('exit: ' + code)
})
```

# 测试

## 测试框架选择

将测试代码添加到代码库中是个不错的主意，因为不仅仅要确保现在的代码库没有问题，而且要保证将来代码库修改以后 不会带来其他问题

在Node应用中添加测试是需要一些挑战的，因为经常需要将*同步、异步和RESTful服务API功能*混合到一起

- jasmine
- mocha/chai
- Jest

## 简单功能测试
## RESTful API测试
## 测试受保护的资源