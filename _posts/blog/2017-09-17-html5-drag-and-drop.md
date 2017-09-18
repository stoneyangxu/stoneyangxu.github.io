---
layout: post
title:  "HTML5 - Drag and Drop"
date:   2017-09-17 22:53:30
categories: HTML5
---

在html出现以来，一直未将拖拽功能加入其核心功能中，开发人员通过底层的鼠标事件来模拟拖拽，难以达到良好的交互效果。html5加入了新的模型、事件、API，满足开发人员开发出体验良好的拖拽功能。

# 概念

将某些元素（图片、文件等），通过从区域A开始拖动，路过区域B，并最终放置到区域C。

在这一过程中：

- 拖动源（drag source） - 即区域A
- 放置目标（drop target） - 拖动过程和最终放置目标，即区域B和区域C
- 数据传输（data transfer） - 拖动时需要传输的数据

> The DataEvent.dataTransfer property holds the drag operation's data (as a DataTransfer object).

> This property is Read only .

> The application is free to include any number of data items in a drag operation. Each data item is a string of a particular type, typically a MIME type such as text/html.

- MITE类型 - 传输数据的类型，让源和目标协商哪种类型满足放置要求
    - text/plain 非格式化文本 
    - image/png png图片
    - image/jpeg jpeg图片
    - text/x-age 非标准化类型，传输自定义的信息类型

在拖动过程中，需要通过样式（鼠标、区域）给出用户友好的*提示*：
- 目标区域是否允许放置？
- 拖动的操作类型是移动？复制？还是链接？
- 当前悬停区域是否需要通过某种样式变化，提示用户是否可以放置？

# 事件

拖拽过程中存在一系列的事件，为开发人员提供合适的切入点进行更精细的控制


![](/images/2017-09-17-23-09-09.jpg)

其中：
- drag是在拖动过程中*持续触发*的，事件在*拖动源*上面触发
- dragover同样是在*路过的拖动目标*上持续触发的
- dropend是在拖动完成后，在*拖动源*上触发的


![](/images/2017-09-18-00-47-54.jpg)


# 设置元素可拖动

```html
<div draggable="true"></div>
```

# dataTransfer对象

## 方法

### DataTransfer.clearData([format])
> Remove the data associated with a given type. The type argument is optional. If the type is empty or not specified, the data associated with all types is removed. If data for the specified type does not exist, or the data transfer contains no data, this method will have no effect.

清除所有（或指定格式）的数据

### DataTransfer.getData(format)
> Retrieves the data for a given type, or an empty string if data for that type does not exist or the data transfer contains no data.

获取指定格式的数据

### DataTransfer.setData(format, data)
> Set the data for a given type. If data for the type does not exist, it is added at the end, such that the last item in the types list will be the new format. If data for the type already exists, the existing data is replaced in the same position.

按数据类型设置数据

### DataTransfer.setDragImage()
> Set the image to be used for dragging if a custom one is desired.

设置拖拽过程中的鼠标样式

## 属性

### DataTransfer.dropEffect
> Gets the type of drag-and-drop operation currently selected or sets the operation to a new type. The value must be none, copy, link or move.

指定拖拽操作类型：copy、move、line、none
 
### DataTransfer.effectAllowed
> Provides all of the types of operations that are possible. Must be one of none, copy, copyLink, copyMove, link, linkMove, move, all or uninitialized.

指定允许的操作类型：none、copy、copyLink、copyMove、link、linkMove、move、all

### DataTransfer.files
> Contains a list of all the local files available on the data transfer. If the drag operation doesn't involve dragging files, this property is an empty list.

允许拖拽的本地文件列表

### DataTransfer.items （Read only）
> Gives a DataTransferItemList object which is a list of all of the drag data.

只读属性，包含拖拽的所有数据

### DataTransfer.types （Read only）
> An array of strings giving the formats that were set in the dragstart event.

只读属性，包含拖拽数据的所有类型



## 示例

### 构造页面 index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Drag & Drop</title>

    <script type="text/javascript" src="index.js"></script>
    <link rel="stylesheet" type="text/css" href="index.css" >

</head>
<body>

    <ul id="members">
        <li draggable="true" data-age="18">Brian</li>
        <li draggable="true" data-age="19">Frank</li>
        <li draggable="true" data-age="20">Clark</li>
        <li draggable="true" data-age="21">Mark</li>
        <li draggable="true" data-age="22">Peter</li>
        <li draggable="true" data-age="23">Combs</li>
    </ul>

    <div class="dropList">
        <fieldset id="racersField">
            <legend>Racers (by age)</legend>
            <ul id="racers" class="drag-target"></ul>
        </fieldset>
    </div>

    <div class="dropList">
        <fieldset id="volunteersFiled">
            <legend>Racers (by age)</legend>
            <ul id="volunteers" class="drag-target"></ul>
        </fieldset>
    </div>
</body>
</html>
```
### 添加样式index.css

```css
#member li {
    cursor: move;
}

.drag-target.highlighted {
    background-color: yellow;
}

.validtarget {
    background-color: lightblue;
}

.drag-target {
    min-height: 100px;
    display: inline-block;
    width: 90%;
}
```


![](/images/2017-09-18-00-09-50.jpg)

可以将列表中的成员拖动到两个区域内

### 绑定事件

```js
    const racersList = document.querySelector('#racers')
    const volunteersList = document.querySelector('#volunteers')

    const members = document.querySelectorAll('#members li')

    document.querySelectorAll('.drag-target').forEach((item) => {
        item.addEventListener('dragenter', handleDragEnter, true)
        item.addEventListener('dragleave', handleDragLeave, true)
        item.addEventListener('drop', handleDrop, false)
        item.addEventListener('dragover', handleDragOver, true)
    })

    members.forEach((item) => {
        item.addEventListener('dragstart', handleDragStart, true) 
        item.addEventListener('dragend', handleDragEnd, true)
    })
```

- dragstart和dragend是在*拖动源*上面触发的
- 其他事件是在*拖动目标*上触发的

### 事件处理

- 拖动开始

```js
    /*
    拖动开始时：
    1. 拖动源设置允许的操作类型为copy，
    2. 添加数据
    3. 将可用目标高亮显示
     */
    function handleDragStart(event) { 
        console.log('dragstart')

        event.effectAllowed = 'copy'

        event.dataTransfer.setData('text/plain', event.target.textContent)
        event.dataTransfer.setData('text/html', event.target.dataset.age)
        
        racersList.classList.add('validtarget')
        volunteersList.classList.add('validtarget')
    }
```


![](/images/2017-09-18-00-13-40.jpg)

- 鼠标进入和退出时，加入特殊样式

```js
    /*
    鼠标进入拖动目标时，给目标区域添加样式
     */
    function handleDragEnter(event) {
        console.log('dropenter')
        event.stopPropagation()
        event.preventDefault()

        event.target.classList.add('highlighted')

        return false
    }

    /*
    鼠标离开拖动目标时，给目标区域移除特殊样式
     */
    function handleDragLeave(event) {
        console.log('dragleave')

        event.stopPropagation()
        event.preventDefault()

        event.target.classList.remove('highlighted')

        return false
    }
```


![](/images/2017-09-18-00-16-29.jpg)

- *鼠标悬浮时，必须屏蔽浏览器默认操作（event.preventDefault），否则不会触发drop事件；浏览器默认禁止将可拖拽元素放置到其他元素内*

```js
    /*
    鼠标悬浮时，设置操作类型
     */
    function handleDragOver(event) {
        console.log('dragover')

        event.dataTransfer.dropEffect = 'copy' // 必须设置，否则不会触发drop事件

        event.stopPropagation()
        event.preventDefault()
        return false
    }
```

- 释放鼠标时，将复制数据加入到目标区域列表

```js

    function handleDrop(event) {
        console.log('drop')

        event.stopPropagation()
        event.preventDefault()

        const dropTarget = event.target

        const li = document.createElement('li')
        li.textContent = event.dataTransfer.getData('text/plain')
        li.dataset.age = event.dataTransfer.getData('text/html')

        dropTarget.appendChild(li)

        return false
    }
```


![](/images/2017-09-18-00-19-31.jpg)

- 完成拖动操作后，需要清空目标区域的特殊样式
```js

    function handleDragEnd(event) {
        console.log('dragend')

        racersList.classList.remove('validtarget')
        racersList.classList.remove('highlighted')
        volunteersList.classList.remove('validtarget')
        volunteersList.classList.remove('highlighted')
    }
```

## 自定义拖动图片

存在一个图片：

```html
    <image id="icon" src="icon.png" ></image>
```

设置拖拽时的鼠标提示：

```js
const img = document.getElementById('#icon')
event.dataTransfer.setDragImage(img, 5, 10) // 图片，向左偏移5像素，向上偏移10像素
```

## 拖动文件

使用HTML5的File API允许将文件拖入页面，并且*异步读取文件信息*，并将其转换为页面元素

文件信息保存在dataTransfer对象的files属性中: 

```js
event.dataTransfer.files[0]
```


![](/images/2017-09-18-00-46-14.jpg)
