---
layout: post
title:  "前端工程化"
date:   2017-09-19 23:08:29
categories: Web
---

# 前端工程化

## 前端工程化的必要性

> “软件工程”是一门研究如何*系统化、规范化、数量化*的开发和维护软件的学科

随着前端项目复杂度的增加、代码量加大、更高的性能要求，出现了大量的*重复性*工作，例如：
- JavaScript、css文件的混淆和压缩
- 图片的压缩和雪碧图合并
- ES6、Sass/Less的预编译
- 三方件的依赖管理和瘦身
- 自动化的静态代码检查、测试执行
- 自动化打包部署
- 本地调试时的自动编译和浏览器自动刷新
- ……

## 发展历史

### 石器时代

早期的Web页面主要承担*静态内容*的展现，用户交互仅仅限于*提交表单*或者*点击链接*。

页面内容通常由后台生成，例如在php或jsp中，在页面中*嵌入后台代码*。

随着项目复杂度的增加，这种方式已经难以维护。

### 铜器时代

比较典型的改进是*组件化*和*异步加载*

以Gmail为代表的Web 2.0时代的到来，大量AJAX技术的应用，使得页面提供更为丰富的*用户交互*功能，承担了更多的业务逻辑。

以往的方式已经难以处理快速增加的复杂度，独立的业务逻辑被*分割*到单独的js文件中，页面通过*<script>*标签引入。

### 农业时代

使用script标签引入脚本时，当脚本之间存在依赖关系，就必须要保证脚本的*加载顺序*，同时，这种方式下所有的内容都是声明在*全局变量*之上，依然难以满足高度复杂的业务需要。

由于JavaScript自身的先天缺陷，缺少模块化的支持，开发人员通过*命名空间*、*AMD*、*CMD*等模块化方案来分割模块。

### 工业时代

伴随着移动互联网的快速发展，前端承担了越来越多的业务应用。为了降低开发难度，合理的控制项目复杂性，各种MV*框架被应用到实际项目中。

同时，ES6标准的确定、前后端分离的普及、工程化理念的加深，出现了*预编译、自动化测试、静态代码检查、工程化管理等*优秀的开发实践，使得开发人员可以充分利用自动化工具来*简化日常工作、提高工作效率*。

### 常见的工程化工具

- Grunt - 基于JavaScript的任务管理器，通过配置文件和外置插件，自动进行预编译、混淆压缩、单元测试、项目构建等自动化任务
- Gulp - 针对Grunt一次性加载文件，导致构建效率低、系统资源消耗高的缺陷，Glup通过*流*技术将待处理的文件在插件之间传递。
- Webpack - 后起之秀却有着一统天下之势，能够把包括脚本文件、样式文件、图片等所有资源都进行加载和处理

## 基于Gulp构建ES6、Sass应用

### 构建一个简单的应用

- 创建一个定时器，在页面刷新当前事件

![](/images/2017-09-20-00-24-01.jpg)

- index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Main</title>

  <link rel="stylesheet" href="sass/index.scss">
</head>
<body>

  <div id="timer"></div>

  <script type="text/javascript" src="js/timer.js"></script>
  <script type="text/javascript" src="js/index.js"></script>
</body>
</html>

```

- index.js

```js
window.onload = () => {
    load()
}

function load() {
  const dom = document.getElementById('timer')
  const timer = new Timer(dom)

  timer.render()
}

```

- timer.js
```js
class Timer {

  constructor(dom) {
    this.dom = dom
  }

  render() {
    setInterval(() => {
        this.dom.textContent = new Date()
    }, 1000)
  }
}
```

- index.scss

```scss
#timer {
  color: cornflowerblue;
  font-size: 20px;
  font-weight: bold;
}
```

### 安装gulp和gulp-cli

```shell
$ yarn add gulp gulp-cli --dev
```

### 加入静态代码检查，纠正编码风格
- 安装eslint(全局安装)，和eslint-cli（本地安装）
```shell
$ yarn global add eslint
$ yarn add eslint-cli --dev
```
- 通过命令行初始化配置文件，回答一系列问题后，生成.eslintrc文件
```shell
$ eslint --init
```

![](/images/2017-09-20-00-29-27.jpg)

```js
module.exports = {
    "env": {
        "browser": true,
        "es6": true
    },
    "extends": "eslint:recommended",
    "parserOptions": {
        "sourceType": "module"
    },
    "rules": {
        "indent": [
            "error",
            4
        ],
        "linebreak-style": [
            "error",
            "unix"
        ],
        "quotes": [
            "error",
            "single"
        ],
        "semi": [
            "error",
            "never"
        ]
    }
};
```

- 安装gulp的eslint插件
```shell
$ yarn add gulp-eslint --dev
```

- 创建*gulpfile.js*文件，加入eslint任务
```js
const eslint = require('gulp-eslint')
const gulp = require('gulp')

gulp.task('eslint', () => {
    gulp.src('src/**/*.js')
      .pipe(eslint({
        useEslintrc: true // 使用.eslintrc配置文件
      }))
      .pipe(eslint.format()) // 输出检查结果
      .pipe(eslint.failAfterError() // 检查失败时，终止任务
})
```

- 执行*gulp task*命令执行检查，看似简单的实现却隐藏者众多的缺陷

![](/images/2017-09-20-00-35-10.jpg)

- 修复错误后，任务正常完成


![](/images/2017-09-20-00-38-00.jpg)

### 类似的步骤，添加sass的静态检查

> 需要先安装gem：*sudo gem install scss_lint*

```shell
$ yarn add gulp-sass-lint --dev
```

```js
gulp.task('scsslint', () => {
  gulp.src('src/**/*.scss')
    .pipe(scsslint())
    .pipe(scsslint.failReporter())
})
```

![](/images/2017-09-20-00-47-53.jpg)

### 使用babel编译js文件

```shell
$ yarn add babel babel-cli --dev
$ yarn add gulp-babel babel-preset-es2015 --dev
```

```js
gulp.task('compile-js', () => {
    gulp.src('src/**/*.js')
      .pipe(babel({
        presets: ['es2015'] // 使用es6插件集编译
      }))
      .pipe(gulp.dest('dist/js')) // 编译后的文件输出到dist
})
```


![](/images/2017-09-20-00-55-05.jpg)

编译后的文件位于dist目录下：


![](/images/2017-09-20-00-55-36.jpg)

### 编译scss文件

```shell
$ yarn add gulp-sass --dev
```

```js
gulp.task('compile-scss', () => {
  gulp.src('src/**/*.scss')
    .pipe(sass().on('error', sass.logError))
    .pipe(gulp.dest('dist/css')) // 编译后的文件输出到dist
})
```

### 代码合并、压缩、重命名

```shell
$ yarn add gulp-concat gulp-uglify gulp-clean-css gulp-rename --dev
```

```js
gulp.task('concat', ['compile-js', 'compile-scss'], () => {
  gulp.src('dist/css/**/*.css')
    .pipe(concat('style.css'))
    .pipe(gulp.dest('dist/latest'))
  gulp.src('dist/js/**/*.js')
    .pipe(concat('app.js'))
    .pipe(gulp.dest('dist/latest'))
})

gulp.task('minify', ['concat'], () => {
  gulp.src('dist/latest/app.js')
    .pipe(uglify())
    .pipe(rename((path) => {
        return path.basename += '.min'
    }))
    .pipe(gulp.dest('dist/latest'))
  gulp.src('dist/latest/style.css')
    .pipe(cleanCSS())
    .pipe(rename((path) => {
      return path.basename += '.min'
    }))
    .pipe(gulp.dest('dist/latest'))
})
```

### 监听文件变化

- 当文件变更时，重新生成文件

```js
gulp.task('watch', ['concat'], () => {
    gulp.watch(['src/**/*.js', 'src/**/*.scss'], ['concat'])
})
```

### 替换html中的url引用

```shell
$ yarn add gulp-html-replace --dev
```

在html文件中，通过注释配置替换块


![](/images/2017-09-20-01-45-49.jpg)

其中build后的名称与gulp中的配置对应：

```js
gulp.task('html-replace',function() {
  return gulp.src('./src/*.html')
    .pipe(htmlreplace({
      'css': 'style.min.css',
      'js': 'app.min.js'
    }))
    .pipe(gulp.dest('./dist/latest'));
});
```

执行替换后的html文件：


![](/images/2017-09-20-01-46-57.jpg)

### 文件变化后，自动刷新浏览器

```shell
$ yarn add browser-sync --dev
```

```js

gulp.task('watch', ['concat', 'minify', 'html-replace'], () => {
    gulp.watch(['src/**/*.js', 'src/**/*.scss'], ['concat', 'minify', 'html-replace'])
})

gulp.task('browser-sync', ['watch'], () => {
    browserSync({
      server: 'dist/latest'
    })
})
```

## 使用webpack构建一个React应用

### 安装相关依赖

```shell
$ yarn add react react-dom
$ yarn add babel-core babel-loader babel-preset-latest babel-preset-react webpack webpack-dev-server --dev
```

### 初始化项目结构如下


![](/images/2017-09-20-02-10-25.jpg)

### 简单实现页面功能

- index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Webpack Demo</title>
</head>
<body>
    <div id="root"></div>

    <script src="./bundle.js"></script> <!-- 引用打包后的文件 -->
</body>
</html>
```

- index.js

```js
import React, {Component} from 'react'
import ReactDom from 'react-dom'
import Hello from './hello'

class App extends Component {
    render() {
        return (
            <div>
                <Hello/>
            </div>
        )
    }
}

ReactDom.render(<App />, document.getElementById('root'))
```
- hello.js
```js
import React, {Component} from 'react'

export default class Hello extends Component {
    render() {
        return (
            <div>
                Hello
            </div>
        )
    }
}
```
### 创建webpack配置文件*webpack.config.js*

```js
const webpack = require('webpack')

module.exports = {
    entry: __dirname + '/app/index.js', // 入口文件
    output: {
        path: __dirname + '/public', // 输入路径
        filename: 'bundle.js' // 打包文件名
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /(node_modules|bower_components)/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['es2015', 'react']
                    }
                }
            }
        ]
    },
    devServer: {
        contentBase: './public',
        inline: true
    }
}
```

### 执行webpack-dev-server执行编译和加载

![](/images/2017-09-20-02-19-09.jpg)


