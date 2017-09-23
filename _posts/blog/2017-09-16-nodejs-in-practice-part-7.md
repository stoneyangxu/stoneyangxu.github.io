---
layout: post
title:  "Node.js硬实战 - 部署"
date:   2017-09-16 02:46:52
categories: 读书笔记
---

> 生产环境中的Node：安全的部署应用程序

- - - - -
# 第十二章 生产环境中的Node：安全的部署应用程序
- - - - -
> 将Node程序部署到自己的服务器上
> 将Node程序部署在云平台上
> 管理生产环境下的包管理
> 日志
> 利用代理服务器和集群扩展Node应用

- - - - -

# 部署

## 使用Apache和Nginx部署Node程序

使用Apache或者Nginx的一个原因就是他们提供的代理功能。

假设已经启动Node服务器并且监听3000端口。

可以在Apache的配置文件中进行代理：

```
ProxyPass / http://localhost:3000/
LoadModule proxy_module /lib/apache2/module/mod_proxy.so
LoadModule proxy_http_module /lib/apache2/module/mod_proxy_http.so
```

- 将代理请求转发到3000端口
- 加载proxy代理模块
- 加载http代理模块

或者使用nginx进行配置：

```
http {
    server {
        listen 80;
        
        location / {
            proxy_pass http://localhost:3000;
            proxy_http_version 1.1;
        }
}
```

- - - - -
> 正向代理 vs 反向代理

- 正向代理 客户端通过代理服务器访问目标服务，从客户端的角度*明确知道目标服务是谁*，而在服务角度来看*不清除请求的来源*。例如：通过通过代理访问google网站。
- 反向代理 客户端访问代理后，请求被分发给隐藏的服务器。从客户端的角度*只知道代理服务器的存在，不知道具体的服务*，从服务端的角度来看*明确清除请求的来源*。例如：拨打10086客服热线后被转接给某个服务中心。


![](/images/2017-09-24-00-20-09.jpg)

- - - - -

## 保持Node进程一直运行

使用一个*监听*进程自动重启Node程序，保证程序崩溃后可以*自动恢复服务*

### 服务监督管理

服务监督管理是一种针对特定操作系统的*通用方案*

例如：

- runit
- Upstart

通过配置文件来*描述需要管理的Node程序信息*，包括：
- 基本描述
- 环境变量配置
- 运行级别
- 启动脚本
- ……


### 管理Node程序的Node程序

通过一个精炼的Node程序来监控其他程序的运行状态，例如：[forever](https://github.com/foreverjs/forever)

安装forever工具后，可以通过命令行的方式使用：

```shell
$ forever start -l forever.log -o out.log -e err.log app.js
```

- 启动应用程序
- 创建一个存储当前进程PID的文件 forever.log
- 创建一个日志输出文件 out.log
- 创建一个错误输出的文件 err.log

通过执行命令，可以将应用停止：

```shell
$ forever end app.js
```

## 在生产环境中使用WebSockets

Node中非常适合使用WebSockets技术——可以在同一个应用进程中*同时提供HTTP和WebSockets服务*

首先要确保服务提供程序和代理服务器*支持HTTP的Upgrade头信息*

WebSockets的出现是针对HTTP协议的缺陷延伸出的解决方案，HTTP协议是*无状态协议*，客户端和服务端的交互被建模为*请求/响应*模式，每个请求和响应保存着各自所需的状态信息。这就导致HTTP协议无法支持*长时间的全双工*链接。

在一些场景下，例如视频流、游戏、实时消息等，HTTP协议无法满足应用场景的需要。

WebSockets用于解决这类场景，支持*类似TCP的长连接*。利用HTTP的Upgrade头，将HTTP协议升级为WebSockets协议。在一些不支持Upgrade的服务器上，只能降级为轮询、长轮询、服务端推送等传统解决方案。

实现方案：利用express和socket.io分别运行http服务器和WebSockets服务器，通过nginx的反向代理功能，根据请求信息分发到对应的server上。

```
location /wsapp/ {
    proxy_pass http://wsbackend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

