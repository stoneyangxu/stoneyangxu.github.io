---
layout: post
title:  "HTML5 - Web Storage API"
date:   2017-09-17 17:25:16
categories: HTML5
---

使用Web Storage API，有效的在请求之间*持久化*数据，避免cookie技术的各种*限制*、*安全问题*以及*带宽消耗*。

# 历史
在此之前，如果希望在客户端*持久化*一些数据，常常借助cookie技术，但是：
- cookie存在大小限制，每个cookie不超过4k
- 只要使用了cookie，每个http请求和响应都会带着cookie信息，无谓的消耗网络带宽
- 因为cookie在请求中不停传递，带来了很大的安全风险

# localStorage 和 sessionStorage

sessionStorage 与 localStorage 的接口类似，但保存数据的生命周期与 localStorage 不同。sessionStorage只是可以将一部分数据在当前会话中保存下来，刷新页面数据依旧存在。但当页面关闭后，sessionStorage 中的数据就会被清空。


![](/images/2017-09-17-17-33-48.jpg)

# 使用本地存储

## 检测浏览器兼容情况

```js
if (window.localStorage) {}
if (window.sessionStorage) {}
```


![](/images/2017-09-17-17-43-07.jpg)

## API
localStorage 和 sessionStorage 的API非常简单而且易于理解。
![](/images/2017-09-17-17-43-56.jpg)

## 存储事件

监听storage事件，当存储发生变更时触发

*事件是在跨页面修改时触发，不会在本页面内触发*

```js
window.addEventListener('storage', function(e) {
    console.log(e)
}, true)
```
![](/images/2017-09-17-17-58-49.jpg)

![](/images/2017-09-17-17-59-46.jpg)


## 调试工具的支持
使用Chrome的开发者工具，可以很方便的查看和编辑存储信息。

![](/images/2017-09-17-17-47-56.jpg)

# 浏览器数据库存储

## Web SQL Database

基于JavaScript异步接口访问SQLit数据库，该方案未获取浏览器厂商的一致支持，已被放弃。

## IndexedDB
> IndexedDB is a low-level API for client-side storage of significant amounts of structured data, including files/blobs. 
> This API uses indexes to enable high-performance searches of this data. 
> While Web Storage is useful for storing smaller amounts of data, it is less useful for storing larger amounts of structured data.

- 相对与storage，用于处理*大数据量*场景
- 对数据建立*索引*提供高性能对搜索
- [IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- 基本功能已经获得浏览器对广泛支持

![](/images/2017-09-17-18-06-46.jpg)

- [使用教程](https://developer.mozilla.org/zh-CN/docs/Web/API/IndexedDB_API/Using_IndexedDB)