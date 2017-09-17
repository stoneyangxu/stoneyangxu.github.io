---
layout: post
title:  "HTML5 - 基础 & Web Forms API"
date:   2017-09-17 18:17:41
categories: HTML5
---

HTML5为我们带来大量的简化和功能增强

# 新功能

## 新的DOCTYPE和字符集

化繁为简，简化以往复杂冗长的配置。

```html
<!DOCTYPE html>
<html lang="en">
```
## 语义化标记

页面文件的可读性更好，更为重要的是帮助搜索引擎分析页面
- header 页面头部区域的内容
- footer 页面地步区域的内容
- section 页面中的一块区域
- article 独立的文章内容
- aside 相关内容或引文
- nav 导航类辅助内容

![](/images/2017-09-17-21-43-41.jpg)

## 使用Selectors API简化选取操作

支持使用*css选择器*查询页面元素

- querySelector - 匹配规则的第一个元素
- querySelectorAll - 匹配规则的所有元素

```js
document.querySelector('input.error')
document.querySelectorAll('input.error')
document.querySelector('input.error', 'lowValue')
```


# Form

HTML5表单包含了大量的新API和元素类型，新增*输入控件*和*API*，其主要目的是为了*简化开发*，为终端用户，特别是移动端用户提供更好的*体验*

## 输入型控件

不同的输入类型，能够让浏览器打开*合适的输入控件*，并利用浏览器内置的*校验规则*快速提供反馈。

> [HTML5 form updates](https://developer.mozilla.org/en-US/docs/Learn/HTML/Forms/HTML5_updates)

> [Example HTML5 Form with Feature Testing](https://www.wufoo.com/html5/example/)
## <input type="xxx">
最大的不同是在*移动端*输入时，弹出的默认输入控件
- tel - 电话号码
- email - 电子邮箱地址
- url - 网页url
- search - 站点顶部的搜索框
- range - 范围内的数值选择器
- number - 数字

## 新增属性

- placeholder - 输入提示信息
- autofocus - 页面加载时自动获得焦点
- autocomplete - 记录输入历史并提供输入提示

![](/images/2017-09-17-18-36-20.jpg)

- spellcheck - 输入内容的拼写检查
- list和datelist - 构造可选列表

```html
<label>Choose a browser from this list:
<input list="browsers" name="myBrowser" /></label>
<datalist id="browsers">
  <option value="Chrome">
  <option value="Firefox">
  <option value="Internet Explorer">
  <option value="Opera">
  <option value="Safari">
  <option value="Microsoft Edge">
</datalist>
```


![](/images/2017-09-17-18-37-59.jpg)

- min和max - range输入的最大和最小值
- step - range控件每次变更的最小跨度
- required - 输入框是否必须输入，结合表单验证使用

## 表单验证

### 验证事件

当浏览器内置当校验规则发现输入内容非法时，会触发*invalid*事件

```js
function haneler(event) {
    // error handle
}

inputElement.addEventListener('invalid', handler)
```

### 验证状态

从事件对象中可以获取*validity对象*，包含各种校验规则对应的校验状态。

- valueMissing - 设置了required属性后，判断内容是否填写
- typeMismatch - 确保输入内容与类型匹配
- patternMismatch - 如果表单设置了pattern，校验输入内容是否满足规则
- maxLength - 如果设置了maxLength，判断输入内容是否过长
- rangeUnderflow - range的输入值小于min属性的值
- rangeOverflow - range的输入值大于max属性的值
- stepMismatch - range的输入值不复合min、max、step的限制
- customError - 调用setCustomValidity(message)将表单控件设置为error状态

### 关闭验证

如果想关闭表单的默认验证行为，有两种方式：

- submit按钮增加*formnovalidate*属性
- form节点上增加*noValidate*属性

### 表单样式
表单验证功能会给form表单添加css伪类

- valid - 校验通过
- invalid - 任何校验不通过
- in-range - range元素的值在min和max之间
- out-of-range - range元素的值在min和max之外
- required - 被标记为必填的字段
- optional - 没有被标记为必填的字段