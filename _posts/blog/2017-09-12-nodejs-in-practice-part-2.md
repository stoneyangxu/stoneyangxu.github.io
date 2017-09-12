---
layout: post
title:  "Node.js硬实战 - Buffers & EventEmitter"
date:   2017-09-12 23:58:31
categories: 读书笔记
---

- - - - -
# 第三章 Buffers 使用比特、字节以及编码
- - - - -
- 介绍Buffer数据类型
- 修改数据编码
- 二进制文件转化为JSON格式
- 创建自定义对二进制协议
- - - - -
# 介绍
历史上JavaScript对二进制支持欠佳，其自身没有提供一个良好对方式去处理原始对内存数据，出于对性能对考虑，Node关于原始数据对处理都在Buffer数据类型中。

Buffers是代表*原始堆对分配额*对数据类型，在JS中使用类数组对方式来使用。

# 修改数据编码

## 文件操作以及很多网络操作都会将数据作为Buffer返回

```js
const fs = require('fs')

fs.readFile('./package.json', (err, buf) => {
    console.log(Buffer.isBuffer(buf)) // true
})
```
## 转换为其他格式

```js
const fs = require('fs')

fs.readFile('./package.json', (err, buf) => {
    console.log(Buffer.isBuffer(buf))

    console.log(buf)

    console.log(buf.toString())
    
    console.log(buf.toString('ascii'))
})
```

![](/images/2017-09-13-00-06-59.jpg)
- toString方法默认将Buffer转换为utf8格式的字符串
- 如果知道内容只包含ascii字符，也可以指定编码类型

## 修改字符串编码
```js
    const encoded = new Buffer('yangxu: passwd').toString('base64')

// eWFuZ3h1OiBwYXNzd2Q=
```

## 使用Buffer创建png图片的[data URIs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs)


![](/images/2017-09-13-00-12-33.jpg)

```js
const fs = require('fs')

const mine = 'image/png'
const encoding = 'base64'


const data = fs.readFileSync('./images.png').toString(encoding)

const uri = `data:${mine};${encoding},${data}`
console.log(uri)

// data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAMgAAADICAIAAAAiOjnJAAAABGdBTUEAALGPC/...

```

## 将dataUri写入文件

```js
const fs = require('fs')

const uri = 'data:image/png;base64,iVBORw0KGgoAAAA...'
const data = uri.split(',')[1]

const buffer = new Buffer(data, 'base64')

fs.writeFileSync('./image_write.png', buffer)
```

## 二进制文件转换为JSON

## 头部信息

- 二进制文件利用头部信息来存储文件相关的*基础信息*
- Buffer获取索引为0的数据和JavaScript中数组相关操作类似，只是索引是指在*内存中的字节位置*
- buf[0]代表第0位的八位字节，或无符号的8比特整数，或8比特的正整数

> 头部信息中1-3位包含最后更新时间，YYMMDD格式

```js
const fs = require('fs')

fs.readFile('./world.dbf', (error, buffer) => {
    const header = {}

    const date = new Date()
    date.setUTCFullYear(buffer[1])
    date.setUTCMonth(buffer[2])
    date.setUTCDate(buffer[3])

    header.lastUpdated = date.toUTCString()

    console.log(header) // { lastUpdated: 'Fri, 20 Dec  109 16:31:44 GMT' }

})
```

> 4-7 记录数量，32-bit的数字

```js
    header.totalRecords = buffer.readUInt32LE(4)
```

> 8-9 16-bit数字 头部的字节数
> 10-11 16-bit数字 记录部分的字节数
```js
    header.bytesInHeader = buffer.readUInt16LE(8)
    header.bytesPerRecord = buffer.readUInt16LE(10)
```

> 32-n each 32byte 字段描述，可能包含多个字段的数据
> n+1 1byte 字段分隔符

```js
const fs = require('fs')

fs.readFile('./world.dbf', (error, buffer) => {
    const header = {}

    const date = new Date()
    date.setUTCFullYear(buffer[1])
    date.setUTCMonth(buffer[2])
    date.setUTCDate(buffer[3])

    header.lastUpdated = date.toUTCString()

    header.totalRecords = buffer.readUInt32LE(4)
    header.bytesInHeader = buffer.readUInt16LE(8)
    header.bytesPerRecord = buffer.readUInt16LE(10)


    const fields = []
    let fieldOffset = 32
    const fieldLength = 32
    const fieldTerminator = 0x0D

    const FIELD_TYPES = {
        C: 'Character',
        N: 'Numeric'
    }

    while(buffer[fieldOffset] !== fieldTerminator) {
        const fieldBuf = buffer.slice(fieldOffset, fieldOffset + fieldLength) // 只是快照，并非拷贝

        const field = {
            name: fieldBuf.toString('ascii', 0, 11).replace(/\u0000/, ''),
            type: FIELD_TYPES[fieldBuf.toString('ascii', 11, 12)],
            length: fieldBuf[16]
        }

        fields.push(field)

        fieldOffset += fieldLength
    }

    console.log(header)
    console.log(fields)
})
```

```
{ lastUpdated: 'Fri, 20 Dec  109 16:45:03 GMT',
  totalRecords: 2127,
  bytesInHeader: 225,
  bytesPerRecord: 74 }
[ { name: 'CODE\u0000\u0000\u0000\u0000\u0000\u0000',
    type: 'Character',
    length: 2 },
  { name: 'CNTRY_NAME', type: 'Character', length: 39 },
  { name: 'POP_CNTRY\u0000', type: 'Numeric', length: 10 },
  { name: 'CURR_TYPE\u0000', type: 'Character', length: 16 },
  { name: 'CURR_CODE\u0000', type: 'Character', length: 4 },
  { name: 'FIPS\u0000\u0000\u0000\u0000\u0000\u0000',
    type: 'Character',
    length: 2 } ]
```

## 创建自己的二进制协议
使用良好的二进制协议来传输数据是一种简洁高效的方法

- 首先要确定*要传输哪些数据*以及*如何去表示*，确定规范
    - 使用*掩码*来确定数据存放在哪个数据库
    - 数据保存以一个在*0-255范围内的无符号正数（单字节）的键值*标识
    - 通过*zlib*压缩
> 0 1byte 数据库
> 1 1byte 数据库键
> 2-n 0-nbyte 数据

- 数值的二进制表示：`8..toString(2)` 两个点用来区分小数

```js
const database = [[],[],[],[],[],[],[],[]]
const bitmasks = [1, 2, 4, 8, 16, 32, 64, 128]

function store(buf) {
    const db = buf[0]
    const key = buf.readUInt8(1)
    
    bitmasks.forEach((bitmask, index) => {
        if ((db & bitmask) === bitmask) {
            database[index][key] = 'some data'
        }
    })
}
```
- 掩码计算利用&运算判断属于哪个db

### 使用zlib解压缩
```js
const zlib = require('zlib')

const database = [[],[],[],[],[],[],[],[]]
const bitmasks = [1, 2, 4, 8, 16, 32, 64, 128]

function store(buf) {
    const db = buf[0]
    const key = buf.readUInt8(1)

    if (buf[2] === 0x78) { // 判读是否被压缩
        zlib.inflate(buf.slice(2), (error, inflatedBuf) => { // 解压缩
            if (error) return console.error(error)
            
            const data = inflatedBuf.toString()

            bitmasks.forEach((bitmask, index) => {
                if ((db & bitmask) === bitmask) {
                    database[index][key] = 'some data'
                }
            })
        })
    }
}
```

### 使用zlib压缩并存储数据

```js

const header = new Buffer(2)
header[0] = 8
header[1] = 0

zlib.deflate('my message', (error, deflateBuf) => {
    if (error) return console.error(error)

    const message = Buffer.concat([header, deflateBuf])

    store(message)
})
```

- - - - -
# 第四章 玩转EventEmitter
- - - - -

- 使用Node的EventEmitter模块
- 异常管理
- 第三方模块中如何使用EventEmitter
- 如何使用domains模块的events
- EventEmitter的替代品
- - - - -

## 基础用法：从EventEmitter继承

```js
const events = require('events')

class MusicPlayer extends events.EventEmitter { // 继承
    constructor() {
        super()
        this.playing = false
    }
}

const musicPlayer = new MusicPlayer()

musicPlayer.on('play', (track) => { // 监听播放事件
    this.playing = true
})

musicPlayer.on('play', (track) => { // 使用多个监听器
    console.log(`play ${track}`)
})

musicPlayer.on('stop', () => { // 监听停止事件
    this.playing = false
    console.log('stop')
})

musicPlayer.emit('play', 'The Roots - The Fire')

setTimeout(() => {
    musicPlayer.emit('stop')
}, 1000)
```

- 移除监听器
```js
musicPlayer.removeAllListeners()

function listener() {
}

musicPlayer.on('play', listener)
musicPlayer.removeListener('play', listener)
```

- 只触发一次的监听器
```js
musicPlayer.once('play', listener)
```

## 混合EventEmitter

当给一个已有当类增加事件能力当时候，使用组合代替继承
```js
class MusicPlayer { // 继承
    constructor() {
        this.playing = false
    }
}

const musicPlayer = new MusicPlayer()

Object.assign(musicPlayer, events.EventEmitter.prototype) // mixin
```

## 异常处理

当一个EventEmitter实例发生错误时，*通常会发出一个error事件*，在Node中，*error事件被当作特殊情况*，如果没有监听，*默认打印堆栈并退出程序*
```js
const events = require('events')

class MusicPlayer { // 继承
    constructor() {
        this.playing = false
    }
}

const musicPlayer = new MusicPlayer()

Object.assign(musicPlayer, events.EventEmitter.prototype)

musicPlayer.on('play', (track) => { // 监听播放事件
    this.playing = true
})

musicPlayer.on('error', (track) => { // 错误处理
    console.log(`play ${track}`)
})

musicPlayer.on('stop', () => { // 监听停止事件
    this.playing = false
    console.log('stop')
})

musicPlayer.emit('error', 'The Roots - The Fire') // 触发错误

setTimeout(() => {
    musicPlayer.emit('stop')
}, 1000)
```

### 通过domains管理异常
帮助处理多个EventEmitter实例的异常，例如多个非阻塞的API
能够*集中*处理多个异步操作，在处理多个*相互依赖*的I/O操作时非常有帮助

```js

const events = require('events')
const domain = require('domain')

const audioDomain = domain.create() // 创建一个domain

class AudioDevice extends events.EventEmitter {

    play() {
        this.emit('error', 'not implemented yet')
    }
}

const audioDevice = new AudioDevice()
audioDevice.on('play', () => {
    audioDevice.play()
})

class MusicPlayer extends events.EventEmitter {

    constructor() {
        super()
        this.audioDevice = audioDevice
    }

    play() {
        this.audioDevice.emit('play')
    }
}


audioDomain.on('error', (err) => { // 捕获异常
    console.log('error')
})

audioDomain.run(() => { // domain之内执行的代码都会被捕获
    const musicPlayer = new MusicPlayer()
    musicPlayer.play()
})
```

## 高级模式

### 反射

动态的响应EventEmitter的变化，或者查询他的监听器
利用EventEmitter的特殊事件*newListener*
```js
const events = require('events')

class MusicPlayer extends events.EventEmitter {
    constructor() {
        super()
        this.playing = false
    }
}

const musicPlayer = new MusicPlayer()


musicPlayer.on('newListener', (name, listener) => { // 监听`监听`事件
    console.log(name, listener)
})

musicPlayer.on('play', (track) => { // 监听播放事件
    this.playing = true
})
```


![](/images/2017-09-13-01-52-49.jpg)

### 探索EventEmitter

大型项目中，模块之间的通信
- Express的app对象
- Redis客户端的RedisClient
### 组织事件名称
使用一个对象来存储所有事件名，在项目中创建一个统一的位置

```js
MusicPlayer.events = {
    play: 'play',
    stop: 'stop'
}
```

## 第三方模块以及扩展 - EventEmitter的替代方案

- 发布/订阅模式 - 水平扩展分布式集群，借助AMQP、js-signals进行多个Node进程之间的通信
