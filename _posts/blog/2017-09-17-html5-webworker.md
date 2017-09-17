---
layout: post
title:  "HTML5 - Web Workers"
date:   2017-09-17 16:56:47
categories: HTML5
---

JavaScript是单线程的，耗时的计算过程，会导致浏览器出现*未响应*错误提示。使用Web Worders技术，可以将耗时的计算*分发给一个Worker*，主线程与worker之间通过*事件*进行交互。


![](/images/2017-09-17-17-22-50.jpg)


> Web Workers makes it possible to run a script operation in background thread separate from the main execution thread of a web application. The advantage of this is that laborious processing can be performed in a separate thread, allowing the main (usually the UI) thread to run without being blocked/slowed down.


# 客户端支持情况


![](/images/2017-09-17-17-01-14.jpg)

- 浏览器检测

```js
if (typeof(Worker) !== 'undefined') {
    // use web workers
}
```

# 使用Web Worker

## 创建一个Worker

```js
// worker.js

addEventListener('message', (msg) => {
    const [number1, number2] = msg.data
    postMessage(parseInt(number1) * parseInt(number2))
})
```

## 页面使用worker执行计算

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Web Worker</title>
</head>
<body>

<script type="text/javascript">

    function bindEvents() {
        const number1 = document.getElementById('number1')
        const number2 = document.getElementById('number2')

        const worker = new Worker('worker.js') // create worder

        worker.addEventListener('message', (msg) => { // received result and update pages
            document.getElementById('result').innerHTML = msg.data
        })

        number1.onchange = () => {
            worker.postMessage([number1.value, number2.value]) // send message to worker
        }

        number2.onchange = () => {
            worker.postMessage([number1.value, number2.value]) // send message to worker
        }
    }

    window.onload = () => {
        if (typeof(Worker) !== 'undefined') {
            bindEvents()
        }
    }

</script>

<input type="text" class="text" id="number1" value="0">
X
<input type="text" class="text" id="number2" value="0">
=
<span id="result">0</span>

</body>
</html>
```

- 使用const worker = new Worker('worker.js')创建worker执行后台计算任务
- 监听message事件，等待worker执行完成并返回结果
- 通过postMessage向worker发送消息


![](/images/2017-09-17-17-15-37.jpg)

## 在worker中引用其他文件

```js
importScripts('helper.js')
importScripts('helper.js', 'another.js')
```

## 关闭worker

```js
worker.terminate();
```

## 错误处理
监听error事件捕获worker执行错误

```js
worker.addEventListener('error', () = {})
```