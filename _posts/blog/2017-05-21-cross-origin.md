---
layout: post
title:  "浏览器同源限制以及跨域访问"
date:   2017-05-21 19:01:35
categories: Http
---

# 浏览器同源限制以及跨域访问

# 什么是浏览器同源策略？

## 概念

同源策略限制从一个源加载的文档或脚本如何与来自另一个源的资源进行交互。这是一个用于隔离潜在恶意文件的关键的安全机制。

## 如何判断是否同源？

根据url中的：

- 协议
- 域名
- 端口

完全相同时才认为是同源。

## 限制内容包括

- Cookie、LocalStorage 和 IndexDB 无法读取。
- DOM 无法获得。
- AJAX 请求不能发送。

# 为什么要限制跨域访问？

## 起源
1995年，同源政策由 Netscape 公司引入浏览器。最初的时候只限制Cookie的互相操作，包括读和写操作。

## 目的
同源策略的主要目的是保证用户的安全性，屏蔽恶意的跨域操作导致用户信息泄漏和身份仿冒。

# 如何绕过浏览器跨域限制？

## document.domain(适用于子域跨域)

子域名的页面可以在全局设置`document.domain`，指向到主域名；通过iframe载入子域名的页面后，页面之间可以绕过同源策略，解决跨域访问。

```js
// URL http://a.com/foo
var ifr = document.createElement('iframe');
ifr.src = 'http://b.a.com/bar'; 
ifr.onload = function(){
    var ifrdoc = ifr.contentDocument || ifr.contentWindow.document;
    ifrdoc.getElementsById("foo").innerHTML);
};

ifr.style.display = 'none';
document.body.appendChild(ifr);
```

```js
// URL http://b.a.com/bar
document.domain = 'a.com'
```

## 通过页面标签src

在body中append一个标签，通过标签的src属性发送get请求

```js
var img = new Image();
img.src = 'http://some/picture';        // 发送HTTP请求

var ifr = $('<iframe>', {src: 'http://b.a.com/bar'});
$('body').append(ifr);                  // 发送HTTP请求
```
## JSONP

script标签的请求是可以跨域的，在url中加入一个回调函数的名称，客户端返回数据后自动调用回调函数，数据作为参数传入回调函数中。

```js
var callback = function(data){
    // 处理跨域请求得到的数据
};
$.getJSON( "http://b.a.com/bar?callback=callback", function( data ){
    // 处理跨域请求得到的数据
});
```
## navigation 对象(IE6/7)

在IE6/7中，iframe之间是共享navigator对象的，用它来传递信息

```js
// a.com
navigation.onData(){
    // 数据到达的处理函数
}
typeof navigation.getData === 'function' || navigation.getData()
```
```js
// b.com
navigation.getData = function(){
    $.get('/path/under/b.com')
        .success(function(data){
            typeof navigation.onData === 'function'
                || navigation.onData(data)
        });
}
```
## 跨域资源共享（CORS）

服务器设置Access-Control-Allow-OriginHTTP响应头之后，浏览器将会允许跨域请求

浏览器需要支持HTML5，可以支持POST，PUT等方法

在目标服务器上设置CORS：
```
Access-Control-Allow-Origin: *              # 允许所有域名访问，或者
Access-Control-Allow-Origin: http://a.com   # 只允许指定域名访问
```

- xhr 设置 withCredentials 后，CORS 方法跨域还可 携带Cookie
- PUT/POST 请求需要注意处理 preflight 请求
> CORS 机制跨域会首先进行 preflight（一个 OPTIONS 请求）， 该请求成功后才会发送真正的请求。 这一设计旨在确保服务器对 CORS 标准知情，以保护不支持 CORS 的旧服务器。

## window.postMessage

HTML5允许窗口之间发送消息

```js
// URL: http://a.com/foo
var win = window.open('http://b.com/bar');
win.postMessage('Hello, bar!', 'http://b.com'); 
```

```js
// URL: http://b.com/bar
window.addEventListener('message',function(event) {
    console.log(event.data);
});
```
- iframe使用场景

```js
//捕获iframe
var domain = 'http://scriptandstyle.com';
var iframe = document.getElementById('myIFrame').contentWindow;

//发送消息
setInterval(function(){
	var message = 'Hello!  The time is: ' + (new Date().getTime());
	console.log('blog.local:  sending message:  ' + message);
        //send the message and target URI
	iframe.postMessage(message,domain); 
},6000);
```

> window.postMessage() 方法可以安全地实现跨源通信。通常，当且仅当执行它们的页面位于具有相同的协议（通常为https），端口号（443为https的默认值），以及主机(模数 Document.domain 由两个页面设置为相同的值)。 window.postMessage() 方法提供了一种受控机制，以便在正确使用时以安全的方式规避此限制。

> window.postMessage() 方法被调用时，会在所有页面脚本执行完毕之后（e.g., 在该方法之后设置的事件、之前设置的timeout 事件,etc.）向目标窗口派发一个  MessageEvent 消息。 该MessageEvent消息有四个属性需要注意： message 属性表示该message 的类型； data 属性为 window.postMessage 的第一个参数；origin 属性表示调用window.postMessage() 方法时调用页面的当前状态； source 属性记录调用 window.postMessage() 方法的窗口信息。


![](/images/2017-05-21-21-24-05.jpg)


![](/images/2017-05-21-21-26-43.jpg)

```js
// Called sometime after postMessage is called
function receiveMessage(event)
{
  // Do we trust the sender of this message?
  if (event.origin !== "http://example.com:8080")
    return;

  // event.source is window.opener
  // event.data is "hello there!"

  // Assuming you've verified the origin of the received message (which
  // you must do in any case), a convenient idiom for replying to a
  // message is to call postMessage on event.source and provide
  // event.origin as the targetOrigin.
  event.source.postMessage("hi there yourself!  the secret response " +
                           "is: rheeeeet!",
                           event.origin);
}

window.addEventListener("message", receiveMessage, false);
```
