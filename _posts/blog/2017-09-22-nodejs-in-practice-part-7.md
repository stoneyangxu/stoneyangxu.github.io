---
layout: post
title:  "Node.js硬实战 - 编写模块"
date:   2017-09-16 02:46:52
categories: 读书笔记
---

> 构建模块并回馈给社区

- - - - -
# 第十三章 掌握Node的所有
- - - - -

- 计划开发一个模块
- 设置package.json
- 依赖处理与与异化版本号
- 添加可执行脚本
- 模块测试
- 发布模块

- - - - -

# 头脑风暴：明确需要做什么

> 使用递归来实现斐波那契数列计算

```js
function fibonacci(n) {
    if (n === 0) return 0;
    if (n === 1) return 1;
    return fibonacci(n - 1) + fibonacci(n - 2);
}
```

因为JavaScript引擎的单线程机制，该算法在V8引擎中运行是很慢的；而且因为没有尾递归优化机制，运算过程中可能导致堆栈移除。

*编写一个模块来实现一个更快的斐波那契数列计算*

## 规划

- 调查一下现有模块，并且确保自己的模块只*专注于一件事*

```shell
$ npm search fibonacci
```


![](/images/2017-09-22-23-51-39.jpg)

在列表中可以看到搜索结果以及*历史版本*，可以观察模块的*发布日期*以及*更新频率*等信息

## 创建一个自己的模块

- 使用一句话描述自己的模块:*简单和明确*

```
Calculate fibonacci as quickly as possible with JavaScript
```

## 创建目录并初始化package.json

```shell
$ mkdir fastfib && cd fastfib
$ yarn init
```

## 使用TDD来验证我们的想法

想法是否可行？目的是否明确？功能是否可用？

```js
// test/fastfib.spec.js
const assert = require('assert')
const fastfib = require('../index')

assert.equal(fastfib(0), 0)
assert.equal(fastfib(1), 1)
assert.equal(fastfib(2), 1)
assert.equal(fastfib(3), 2)
assert.equal(fastfib(4), 3)
assert.equal(fastfib(5), 5)
assert.equal(fastfib(6), 8)
assert.equal(fastfib(7), 13)
assert.equal(fastfib(8), 21)
assert.equal(fastfib(9), 34)
assert.equal(fastfib(10), 55)
assert.equal(fastfib(11), 89)
assert.equal(fastfib(12), 144)

console.log('Test passed')
```

## 创建lib目录以及实现代码

```js
// lib/recurse.js
function recurse(n) {
    if (n === 0) return 0
    if (n === 1) return 1
    return recurse(n - 1) + recurse(n - 2)
}

module.exports = recurse
```

## 在根目录创建入口文件index.js

```js
const recurse = require('./lib/recurse')

module.exports = recurse
```

## 运行测试并校验结果


![](/images/2017-09-23-00-13-08.jpg)

## 基准测试

我们的目的是实现一个*更快的*算法，那么现在的性能怎么样？

- 使用Benchmark.js来进行基准测试

```shell
$ yarn add benchmark --dev
```
- 创建一个benchmark目录并添加index.js文件来执行基准测试
```js
const assert = require('assert')
const recurse = require('../lib/recurse')
const suite = new (require('benchmark')).Suite // 创建测试套件

suite
    .add('recurse', () => { // 添加一个测试，计算20个数字
        recurse(20)
    })
    .on('complete', function() {
        console.log('result: ')
        this.forEach((result) => { // 输出测试结果
            console.log(result.name, result.count, result.times.elapsed)
        })

        assert.equal( // 断言递归的方式是正确的
            this.filter('fastest').map('name'),
            'recurse',
            'expect recurse to be the fastest'
        )
    })
    .run()
```

执行结果如下，在*5.424秒内执行了507次*：

```
result: 
recurse 507 5.424
```

## 当前实现没有实现尾递归优化，lib目录下新增一个tail.js

```js

function tail (n) {
    return fib(n, 0, 1)
}

function fib(n, current, next) {
    if (n === 0) return current
    return fib(n - 1, next, current + next)
}

module.exports = tail
```
- - - - -
> [尾递归优化](http://www.ruanyifeng.com/blog/2015/04/tail-call.html)

函数调用会在内存形成一个"调用记录"，又称"调用帧"（call frame），保存调用位置和内部变量等信息。*如果在函数A的内部调用函数B，那么在A的调用记录上方，还会形成一个B的调用记录。等到B运行结束，将结果返回到A，B的调用记录才会消失。如果函数B内部还调用函数C，那就还有一个C的调用记录栈，以此类推*。所有的调用记录，就形成一个"调用栈"(call stack)

*尾调用由于是函数的最后一步操作，所以不需要保留外层函数的调用记录*，因为调用位置、内部变量等信息都不会再用到了，只要直接用内层函数的调用记录，取代外层函数的调用记录就可以了。

"尾调用优化"（Tail call optimization），即*只保留内层函数的调用记录。如果所有函数都是尾调用，那么完全可以做到每次执行时，调用记录只有一项，这将大大节省内存。*

- - - - -

- 添加一个对比测试
```js
    .add('recurse', () => { // 添加一个测试，计算20个数字
        recurse(20)
    })
    .add('tail', () => { // 添加一个测试，计算20个数字
        tail(20)
    })
```


![](/images/2017-09-23-00-34-05.jpg)

测试结果中，原有的*断言失败*，因为尾递归拥有更高的效率

## 在入口文件index.js中，将尾递归作为默认实现

```js
module.exports = require('./lib/tail')
```

## 执行功能测试，保证功能正确

![](/images/2017-09-23-00-37-28.jpg)

## 尝试更快的算法：使用迭代

```js
function iter(n) {
    let current = 0
    let next = 1
    let swap = 0

    for (let i = 0; i < n; i++) {
        swap = current
        current = next
        next = swap + next
    }

    return current
}

module.exports = iter
```

## 添加对比测试并验证结果

```js
suite
    .add('recurse', () => { // 添加一个测试，计算20个数字
        recurse(20)
    })
    .add('tail', () => {
        tail(20)
    })
    .add('iter', () => {
        iter(20)
    })
```


![](/images/2017-09-23-00-48-59.jpg)

*使用iter作为最终实现*，最后的目录结构如下：


![](/images/2017-09-23-00-49-54.jpg)

## 编辑package.json

```json
{
  "name": "fastfib-demo",
  "version": "0.1.0",
  "description": "Calculate a Fibonacci number as fast as possible with JavaScript",
  "main": "index.js",
  "repository": "git@github.com:stoneyangxu/fastfib-demo.git",
  "author": "stoneyangxu <stoneyangxu@icloud.com>",
  "license": "MIT",
  "private": false,
  "scripts": {
    "test": "node test/fastfib.spec.js && node benchmark/index.js"
  },
  "keywords": [
    "fibonacci",
    "fast"
  ],
  "bugs": "https://github.com/stoneyangxu/fastfib-demo/issues",
  "homepage": "https://github.com/stoneyangxu/fastfib-demo",
  "devDependencies": {
    "benchmark": "^2.1.4"
  }
}

```

- - - - -

> npm的依赖

模块的package.json中会定义依赖的其他模块，以便使用者维持模块的完整性。

- dependencies 模块正常运行时需要的依赖
- devDependencies 开发时需要的依赖，例如测试、基准测试、服务器加载工具等
- optionalDependencies 非必要的依赖，可以增强模块功能，安装依赖时如果失败，可以跳过并降级使用其他模块
- peerDependencies 运行时需要的依赖，但是已经被安装了，例如grunt等，保证只有在安装了指定依赖（版本）时，才能够安装

- - - - -

## 语义化版本号

### 版本号规则

*主版本号.次版本号.修订号*

- 主版本号 做了不兼容的API修改
- 次版本号 做了向下兼容的功能增强
- 修订号 做了向下兼容的问题修正

### 版本控制

*低于1.0.0*的版本来表示当前API还*未完全实现*，会频繁变更。

### 变更日志

格式：

```
Version 0.5.0--2017-09-23
---
added: feature x
removed: feature y [breaking change!]
updated: feature z
fixed: bug xx
```

## 用户体验

在发布模块之前确定模块正常工作。

### 添加可执行脚本

- 添加脚本

```js
// bin/index.js

#! /usr/bin/env node

const fastfib = require('../')
const seqNo = Number(process.argv[2])

if (isNaN(seqNo)) {
    return console.error('\nInvalid sequence number provided, try: \n fastfib 30\n')
}

console.log(fastfib(seqNo))
```

- package.json中添加可执行脚本

```json
  "bin": {
    "fastfib": "./bin/index.js"
  },

```

- 使用npm link命令来测试脚本

```js
$ npm link
$ fastfib 20
```


![](/images/2017-09-23-01-27-22.jpg)

fastlib被模拟为全局安装，可以当作脚本来执行

- 在其他模块（或应用）中连接模块

```shell
$ cd application
$ npm link ../fastlib
```

# 发布

- 注册，注册信息会保存在.npmrc中

```shell
$ npm adduser
```

![](/images/2017-09-23-01-40-41.jpg)

- 发布
```shell
$ npm publish
```

![](/images/2017-09-23-01-47-00.jpg)

- 更新版本
```shell
$ npm version patch
$ npm publish
```
![](/images/2017-09-23-01-48-55.jpg)

- 删除模块
```shell
$ npm unpublich --force
```

![](/images/2017-09-23-01-52-50.jpg)


# 使用私有模块

当我们只希望在内部共享模块时，将其设置为私有，在package.json中添加：

```json
"private": true
```

执行npm public时会被阻止

在内部共享时，有几种方式：

### 使用GIT来共享

- 在安装时指定git路径，会自动从master拉取模块

```shell
$ npm install git+ssh://git@github.com:mycompany/fastfib.git --save
```
- 在package.json文件中指定仓库地址

```json
"dependencies": {
    "a": "git+ssh://git@github.com:mycompany/a.git#0.1.0",
    "b": "git+ssh://git@github.com:mycompany/b.git#develop",
    "b": "git+ssh://git@github.com:mycompany/c.git#dacc525c"
}
```

> a模块指定tag
> b模块指定分支
> c模块指定提交的SHA-1

### 使用URL来共享

- 将模块打包
```shell
$ tar -czf fastfib.tar.gz fastfib
```

- 将文件上传到web服务器上
- 安装时指定url

```shell
$ npm install http://some-server/fastfib.tar.gz --save
```
### 使用私有仓库

自定搭建npm私有源，将模块发布到私有源上