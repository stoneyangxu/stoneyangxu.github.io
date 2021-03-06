---
layout: post
title:  "<深入浅出React和Redux> - 同构"
date:   2017-11-13 22:42:12
categories: 读书笔记
---

# 同构

同构: 同一份代码可以在**不同环境**下运行, 功能呢组件能够在客户端**渲染**, 也能够在服务端**生成HTML**

## 服务器渲染 vs 浏览器渲染

万维网是从**服务端渲染**发展起来的, 所有的页面都是**静态HTML**文件, 但是没法提供动态内容

随着**CGI**技术的出现, 使得服务器端终于可以产生动态内容了, 但是依然是在**服务器端生成HTML**, 只是结合了数据和模板

服务器渲染统治了很长时间, 直到**AJAX**技术的出现, 使得网页可以做到**局部刷新**

直到之后的**浏览器端渲染**: 

- 一个应用框架: 包含路由和应用结构功能
- 一个模板库: 以数据为输入, 输出的就是HTML字符串
- 服务器端的API支持: 提供数据的API服务器

在React应用中, 可以将网页内容, 动态行为, 以及样式全部封装在同一个组件中, 把浏览器端渲染发挥到了极致


虽然浏览器端渲染很强大, 但是还有一个问题需要解决: **首页性能**

- TTFP(Time to First Paint) 从HTTP请求发出, 到**第一个有意义的内容渲染出来**的时间差
- TTI(Time to Interactive) 从HTTP请求发出, 到**用户可以交互**的时间差

TTI一定是大于TTFP的, 这两个指标的时间越短越好

在打开一个页面时, 不考虑**缓存**, 最坏需要三个请求:

- 向服务器**获取HTML**
- 解析HTML后获取**JavaScript**文件
- 访问API服务器**获取数据**

相对来说, **服务端渲染**的优势在于:

- 第一个HTTP请求就能够返回**有意义**的内容
- 有利于**搜索引擎优化**

但是使用服务端渲染有多种因素需要考虑:

- 服务器获取数据快还是浏览器获取数据快?
- 服务器产生的HTML过大是否会影响性能?
- 运算是否是服务器能够承受的起的?

## React同构

React本身并不是被**设计用来服务端渲染的**, 但是React因为其构建理念, 他非常适合用来做同构.

为了支持同样代码两端运行, 服务器端同样需要**能够运行JavaScript**, 所以需要Node.js作为服务器

为了实现同构, 需要实现以下功能:

- 在服务器端根据React组件生成HTML
- 数据脱水和注水
- 服务器端管理Redux
- 支持服务器和浏览器获取共同数据源

### React服务器端渲染HTML

React在客户端渲染使用render()函数, 最终产生的是DOM元素, 而在服务器端, 最终产出的是HTML字符串

客户端实际渲染之前, 会计算HTML字符串的校验和, 如果与上一次相同, 就不会重新渲染了

### 脱水和注水

服务器渲染时, 除了要返回HTML字符串, 除此之外还需要**脱水数据**, 用来注入到React组件中, 当浏览器渲染时, 根据脱水数来渲染组件, 这个过程叫做**注水**.

脱水数据的传递方式, 一般是在页面中**内嵌**一段JavaScript, 内容就是把传递给React组件的数据赋值给某个**全局变量**

### 服务器端Redux Store

服务器端渲染最大的**区别**是, 要为每个用户创建一个**新的Store**

### 支持服务器和浏览器共同获取数据

// todo

### 服务器端路由

// todo


