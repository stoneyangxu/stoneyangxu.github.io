---
layout: post
title:  "Node.js硬实战 - 文件系统"
date:   2017-09-14 00:59:44
categories: 读书笔记
---

- - - - -
# 第六章 文件系统：通过异步和同步的方法处理文件
- - - - -

fs模块相对其他IO模块比较特殊，存在同步和异步两种接口

- 理解fs模块及其组件
- 使用配置文件及文件描述
- 使用文件锁技术
- 递归文件操作
- 编写文件数据库
- 监听文件及文件夹

- - - - -

## fs模块概述

### fs模块中支持POSIX文件I/O的封装

> 可移植操作系统接口（英语：Portable Operating System Interface，缩写为POSIX），是IEEE为要在各种UNIX操作系统上运行软件，而定义API的一系列互相关联的标准的总称

> [File System @ Node.js Documentation](https://nodejs.org/api/fs.html)

```js
const fs = require('fs')
const assert = require('assert')

const fd = fs.openSync('./file.txt', 'w+') // 用于写或者读读方式打开文件

const writeBuf = new Buffer('some text to write')
fs.writeSync(fd, writeBuf, 0, writeBuf.length, 0) // 写入文件

const readBuffer = new Buffer(writeBuf.length)
fs.readSync(fd, readBuffer, 0, writeBuf.length, 0) // 读入文件

assert.equal(writeBuf.toString(), readBuffer.toString())

fs.closeSync(fd) // 关闭文件
```

### 流
- createReadStream创建读入流
- createWriteStream创建写入流
- 使用pipe连接流

```js
const fs = require('fs')

const readStream = fs.createReadStream('./file.txt')
const writeStream = fs.createWriteStream('./copy.txt')

readStream.pipe(writeStream)
```

### 批量文件操作

文件系统接口包含一些*批量*的方法来读写或追加，在一次性读入内存或者一次性写入文件时很有用

```js
const fs = require('fs')
fs.readFile('./file.txt', (err, buf) => {
    console.log(buf.toString())
})
```

### 文件检视

- 通过fs.watch和fs.watchFile来检视文件的变化
- fs.watch通过底层操作系统来通知文件改变，非常高效
- 但是fs.watch比较难用，而且不能用在网络磁盘，可以使用相对低效的fs.watchFile替代

### 同步的替代方案

- 同步方法应该在*初始化*时使用，而不应该在*回调*中使用

## 读取配置文件

```js
const fs = require('fs')

// 异步
fs.readFile('./package.json', (error, buf) => {
    if (error) throw error

    const config = JSON.parse(buf.toString())

    console.log(config)
})

// 同步

try {
    const config = JSON.parse(fs.readFileSync('./package.json'))
    console.log(config)
} catch (error) {
    console.error(error)
}
```
- 同步方法中，如果有错误会自动抛出，可以使用try/catch捕获异常

## 使用文件描述

- 文件描述是在操作系统中管理的在进程中打开文件所关联的一些数字或者索引
- 文件描述可以指向目录、管道、网络套接字以及常规文件
- 常规文件描述符
    - stdin 0 标准输入
    - stdout 1 标准输出
    - stderr 2 标准错误

- 一个描述符是open以及openSync方法调用返回的一个数字
- 文件描述更多的是在多进程场景下使用

## 使用文件锁

锁住文件来防止多个进程冲突修改

大多数操作系统提供*强制锁*和*咨询锁*

- 强制锁 - 在内核级别执行
- 咨询锁 - 只在涉及到的进程订阅了相同的锁机制

除了使用*flock*直接锁住文件，还能通过*锁文件*

锁文件是普通的文件或者文件夹，创建的必须是*原子性的*

### Node中的实现方式

#### 使用独占标记创建

通过*x*标记以独占模式打开文件

```js
fs.open('config.lock', 'wx', () => {})
```

将进程id写入文件，可以在异常发生时知道*最后的拥有者*

```js
fs.writeFile('config.log', process.pid, { flags: 'wx' } () => {})
```

#### 使用mkdir创建

当锁文件在*网络磁盘*上时，独占模式可能无法工作，而mkdir是原子操作，可以很好的跨平台，并且将pid写入目录中的一个文件

```js
fs.mkdir('config.lock', (err) => {
    if (err) return console.error(err)
    fs.writeFile('config.lock/' + process.pid, () => {})    
})
```

#### 锁文件模块

```js
const fs = require('fs')

let hasLock = false
const lockDir = 'config.lock'

exports.lock = (cb) => {
    if (hasLock) return cb()

    fs.mkdir(lockDir, (err) => {
        if (err) return console.error(err)

        fs.writeFile(lockDir + '/' + process.pid, (err) => {
            if (err) return console.error(err)

            hasLock = true
            cb()
        })
    })
}

exports.unlock = (cb) => {

    if (!hasLock) return cb()

    fs.unlink(lockDir + '/' + process.pid, (err) => {
        if (err) return console.error(err)

        fs.rmdir(lockDir, (err) => {
            if (err) return console.error(err)

            hasLock = false
            cb()
        })
    })
}

process.on('exit', () => {
    if (hasLock) { // 如果在退出时依然存在锁，同步删除
        fs.unlinkSync(lockDir + '/' + process.pid)
        fs.rmdirSync(lockDir)
        console.log('removed lock')
    }
})
```
## 递归文件操作

### 从一个多层目录中查找文件

- 同步版本

```js
const fs = require('fs')
const join = require('path').join

exports.findSync = (nameRe, startPath) => {
    const results = []

    const finder = (path) => {
        const files = fs.readdirSync(path)

        files.forEach((file) => {
            const fpath = join(path, file)

            const states = fs.statSync(fpath)

            if (states.isDirectory()) finder(fpath)
            if (states.isFile() && nameRe.test(file)) results.push(fpath)
        })
    }

    finder(startPath)
    return results
}
```
客户端在处理异常的时候，只需要简单的try/cache即可
```js
try {
    const results = findSync(/^package.json$/, '/Users/yangxu/Documents/projects/demo/nodejs-in-practice/')
    console.log(results)
} catch (err) {
    console.error(err)
}
```

- 异步版本

```js
const fs = require('fs')
const join = require('path').join


exports.find = (nameRe, startPath, cb) => {

    const results = []
    let asyncOps = 0 // 需要计数器来判断是否完成了遍历
    let errored = false

    const error = (err) => {
        if (!errored) cb(err) // 如果存在多个错误，确保只执行一次回调
        errored = true
    }

    const finder = (path) => {
        asyncOps++

        fs.readdir(path, (err, files) => {

            files.forEach((file) => {
                const fpath = join(path, file)

                asyncOps++

                fs.stat(fpath, (err, stats) => {
                    if (err) return error(err)

                    if (stats.isDirectory()) finder(fpath)
                    if (stats.isFile() && nameRe.test(file)) results.push(fpath)

                    asyncOps-- // 在每个异步操作完成时，计数器减1

                    if (asyncOps === 0) cb(null, results) // 完成所有异步操作后，执行回调
                })
            })

            asyncOps-- // 在每个异步操作完成时，计数器减1
            if (asyncOps === 0) cb(null, results)  // 完成所有异步操作后，执行回调
        })
    }

    finder(startPath)
}

```

客户端使用时，使用标准的回调

```js
const result = find(/^package.json$/, path, (err, results) => {
    if (err) console.error(err)
    console.log(results)
})
```

## 编写文件数据库

需要编写一个简单快速的数据存储结构，并且保持一致性
通过*追加日志*使用内存数据库

- 简单的key/value模块
- 有效的IO效率 - 只写到文件的最后
- 持久 - 文件上的一个状态永远不变
- 简单的创建备份 - 可以复制任何时间点的状态

```js
const fs = require('fs')
const events = require('events')

class Database extends events.EventEmitter {
    constructor(path) {
        super()
        this.path = path

        this._records = Object.create(null) // 创建在内存中的记录映射

        this._writeStream = fs.createWriteStream(this.path, {
            encoding: 'utf8',
            flags: 'a' // 创建一个追加流
        })

        this._load()
    }

    _load() { // 加载已有的数据
        const stream = fs.createReadStream(this.path, {encoding: 'utf8'})
        const database = this

        let data = ''

        stream.on('readable', () => {
            data += stream.read() // 读取可用数据

            const records = data.split('\n') // 按行分隔数据
            data = records.pop() // 获取最后一个可能未完成的记录，最后一行通常是''

            records.forEach((record) => {
                try {
                    const json = JSON.parse(record)
                    if (json.value === null) { // 如果有null则删除记录
                        delete database._records[json.key]
                    } else {
                        database._records[json.key] = json.value // 按照键值存储
                    }
                } catch (e) {
                    database.emit('error', 'found invalid record: ', record)
                }
            })

        })

        stream.on('end', () => {
            database.emit('load')
        })
    }

    get(key) {
        return this._records[key]
    }

    set(key, value, cb) {
        const toWrite = JSON.stringify({key, value}) + '\n'

        if (value === null) {
            delete this._records[key]
        } else {
            this._records[key] = value
        }

        this._writeStream.write(toWrite, cb)
    }

    del(key, cb) {
        this.set(key, null, cb)
    }
}

module.exports = Database
```

- 客户端代码

```js
const Database = require('./database')
const client = new Database('./test.db')

client.on('load', () => {
    client.set('bar', 'my sweet value', (err) => {
        if (err) return console.error(err)

        console.log(client.get('bar'))
    })

    client.del('baz')
})
```

## 检视文件及文件夹

- fs.watch 渗透到操作系统的通知系统，提供统一的API、更可靠的监听、更快的实现
- fs.watchFile 通过不停轮询，不成熟也不可靠，但是优势是*跨平台*，在*网络文件系统*中更可靠

选择时：
- 运行测试，优先考虑fs.watch
- 尽量减少监听范围
- 如果需要在变化中比较文件，watchFile更友好，输出结果中带有stat信息
- fs.watch需要确保不同平台能够正确运行

```js
const fs = require('fs')

fs.watch('./watchdir', console.log)
fs.watchFile('./watchdir', console.log)
```