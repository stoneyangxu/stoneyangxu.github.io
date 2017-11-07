---
layout: post
title:  "<深入浅出React和Redux> - 基础"
date:   2017-11-05 17:46:41
categories: 读书笔记
---

> 互联网技术发展一日千里, 网页应用开发技术也不例外

> jQuery长期占领着网页开发领域的统治地位, 随着工程的逐渐增大, jQuery和其他MVC框架已经难以管理快速增加的复杂度
> React, Flux以及Redux克服了很多传统MVC框架的弊端, 给大家带来了一种全新的应用开发方式

# React新的前端开发方式

## 如何初始化一个React项目

React开发需要一些前期准备:

- 命令行运行环境
- 具备良好开发调试功能的浏览器, 例如Chrome
- Node.js运行环境以及其自带的npm包管理器(或yarn)
- 熟悉的IDE或编辑器, 例如WebStrom或VS Code

React技术依赖于一系列庞大的技术栈:

- 使用Babel进行代码转译
- 使用Webpack, grunt或gulp进行代码打包
- 使用ESLint进行代码检查
- 自动化测试执行
- ......

想要快速开始React的开发, 可以借助Facebook官方提供的工具: **create-react-app**

> [facebookincubator/create-react-app](https://github.com/facebookincubator/create-react-app)

```shell
$ yarn global add create-react-app
```

而后借助命令行来帮助我们快速初始化一个项目:

```shell
$ create-react-app dissecting-react-and-redux
```

项目创建完成后, 会给出提示信息, 显示可以直接使用的命令(create-react-app模式使用yarn包管理器): 

```
Success! Created dissecting-react-and-redux at /Users/yangxu/Documents/projects/demo/dissecting-react-and-redux
Inside that directory, you can run several commands:

  yarn start
    Starts the development server.

  yarn build
    Bundles the app into static files for production.

  yarn test
    Starts the test runner.

  yarn eject
    Removes this tool and copies build dependencies, configuration files
    and scripts into the app directory. If you do this, you can’t go back!

We suggest that you begin by typing:

  cd dissecting-react-and-redux
  yarn start

```

- yarn start 运行开发模式, 通过内置的webpack-dev-server自动更新模块并刷新浏览器, 辅助开发进行快速调测
- yarn build 构建生产发布包
- yarn test 启动内置的自动化测试
- yarn eject 弹出内置的配置文件, 当内置的配置无法满足需要时使用, 但是该过程不可逆

create-react-app隐藏了大量的配置文件, 使得开发者可以快速上手而不必被繁琐的配置阻塞.


![](/images/2017-11-05-18-18-32.jpg)


通过`yarn start`命令启动项目后, 会自动打开浏览器页签并显示初始页面:


![](/images/2017-11-05-18-32-09.jpg)



## 如何创建一个React组件

React的首要思想是通过**组件**来开发应用.

所谓**组件**, 就是能够完成某个功能的**独立的**, **可重用**的代码, 用**分而治之**的方法将大应用分解为若干易于管理的小组件.

```
.
├── README.md
├── package.json
├── public
│   ├── favicon.ico
│   ├── index.html
│   └── manifest.json
├── src
│   ├── App.css
│   ├── App.js
│   ├── App.test.js
│   ├── index.css
│   ├── index.js
│   ├── logo.svg
│   └── registerServiceWorker.js
└── yarn.lock
```

在生成的项目中, 组件代码位于src目录下, 以**index.js**作为入口:

```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import registerServiceWorker from './registerServiceWorker';

ReactDOM.render(<App />, document.getElementById('root'));
registerServiceWorker();
```

- 首先导入react依赖, react和react-dom
- 导入主页样式index.css
- 导入需要渲染的组件**App**
- **ReactDOM.render**意味着在页面的root元素内渲染App组件, 这是整个应用的起点
- registerServiceWorker就是为react项目注册了一个service worker，用来做资源的缓存，这样用户下次访问时，就可以更快的获取资源。而且因为资源被缓存，所以即使在离线的情况下也可以访问应用


我们将要完成一个带有交互功能的组件, **点击按钮, 计数器中的数字增加**, 新建一个ClickCounter.js

```js
import React, { Component } from 'react';

class ClickCounter extends Component {

  constructor(props) {
    super(props);

    this.state = { count: 0 };
    this.onClickButton = this.onClickButton.bind(this);
  }

  onClickButton() {
    this.setState({
      count: this.state.count + 1
    })
  }

  render() {
    return (
      <div>
        <button onClick={this.onClickButton}>Click Me</button>
        <div>
          Click Count: {this.state.count}
        </div>
      </div>
    );
  }
}

export default ClickCounter;
```


- 引入的React是为了支持JSX语法, JSX最终会被翻译为React的表达式
- Component是所有组件的基类, 代替过时的React.createClass方法
- JSX是JavaScript的语法扩展(eXtension), 使得我们可以在JavaScript中编写像HTML一样的代码
    - JSX不仅可以包含HTML标签, 也可以使用**React组件**, 判断方式就是**首字母是否大写**
    - 也可以通过onClick这样的方式给元素添加**事件处理函数**
    - 并且通过**{}**语法来嵌入JavaScript表达式
    - 相对于外挂模板的方式, JSX的理念是提高组件的**內聚性**, 将组件相关的内容放置在同一个文件内
- JSX中处理onClick的方式与DOM原生方式不同
    - 挂在的每个函数控制在**组件范围内**, 而不是HTML中的全局范围, 不会污染全局空间
    - 事件绑定使用了**事件委托**, 所有事件被挂载在**顶层DOM元素**上的同一个函数处理, 然后再分发给具体的组件处理, 避免过多事件挂载导致的性能问题
    - React负责控制组件的**生命周期**, 在unmount的时候能够自动清理所有相关的事件处理函数, 避免了内存泄漏
- React可以在组件内定义样式, 进一步提高內聚性

```js
  render() {

    const counterStyle = {
      margin: '16px'
    }

    return (
      <div style={counterStyle}>
        <button onClick={this.onClickButton}>Click Me</button>
        <div>
          Click Count: {this.state.count}
        </div>
      </div>
    );
  }
```

在App.js中导入并使用新增的组件:

```js
import React, { Component } from 'react';
import ClickCounter from './ClickCounter';

class App extends Component {
  render() {
    return (
      <div>
        <ClickCounter />
      </div>
    );
  }
}

export default App;
```


页面显示如下:


![](/images/2017-11-05-18-49-00.jpg)

## React的工作方式

借助上文创建的ClickCounter组件来简单了解一下React的工作方式.

### jQuery是如何工作的

如果使用jQuery来完成上述功能:

```js
$(function() {
  $('#clickMe').click(function() {
      var clickCounter = $('#clickCounter');
      var count = parseInt(clickCounter.text(), 10);
      clickCounter.text(count + 1);
  }); 
})
```

使用jQuery的常见模式:

- 利用css选择器**选择一些元素**
- 进行一些**业务处理**
- 将结果**写入页面**

但是随着项目越来越复杂, 这种模式会导致**代码结构复杂, 难以维护**, 而且**取值, 写入**这种操作必须要**手工完成**, 使得业务逻辑**严重依赖**页面结构, 无法解耦而且难以测试, 其本质相当于工作在一个**隐藏的全局变量**之上.

### React的理念

React中并没有"选取一些元素"之类的操作

- 使用jQuery, 需要告诉他**如何去做**
- 使用React, 则是告诉他**做成什么样子**

React的工作理念, 归结为一个公式:

> UI = render(data)

这是一个**纯函数**, 对于相同的数据, 永远渲染出相同的页面.

对于开发者来说, 重要的是**区分data和render**, 想要更新界面要做的就是更新data, 界面自动作出**响应**, 所以React实践的也是**响应式编程**的思想

在我们的例子中: 
- data就是state中的count
- render方法根据data渲染界面
- onClickButton方法修改数据

### Virtual DOM

表面上看, react在更新界面的时候都是通过重复渲染, 会比jQuery的"定点"更新要更加耗费性能.

实际上React利用Vietual DOM, 让**每次渲染都只更新数量最少的DOM元素**

遵循前端性能优化的原则:**尽量减少DOM操作**, 避免浏览器的重排和重绘

- JSX会被Babel**解析为React或HTML语句**
- 使用这些语句首先**构造Virtual DOM**
- 而后在Virtual DOM树上**自上而下**进行遍历, 并依次对比发现的**差别**
- 只将需要变化的部分更新到真实的DOM树上

### React工作方式的优点

当项目变得复杂时, jQuery很容易写出**纠缠**的代码结构


![](/images/2017-11-05-21-14-14.png)

使用React的时候, 只需要关心事件是如何影响数据的, 对DOM的操作完全交给框架去维护, 大大的提高了项目的**可维护性和可读性**


![](/images/2017-11-05-21-19-18.png)


# 设计高质量的React组件

> 不要只满足于编写可运行的代码, 要了解代码背后运行的原理, 还要让代码可读而且易于维护.

## 易于维护组件的设计要素

每个复杂的组件都是由简单的组件逐渐**演变**而来, 当组件变得复杂的时候, 就应该**拆分**这个组件, 使用一个个**单一功能**的小组件组合成复杂的功能, 这就是**分而治之**的思想.

拆分组件最重要的就是确定组件的**边界**: 每个组件都应该可以**独立存在**, 否则就不应该拆分.

在软件设计上存在通则: 组件的划分要**高内聚, 低耦合**

- 高内聚 - 将逻辑紧密相关的内容放在一个组件中: 内容, 行为, 样式.
- 低耦合 - 不同组件的依赖关系要尽量弱化, 每个组件尽量独立


## React组件的数据

> 差劲的程序员关心代码, 优秀的程序员操心数据结构和它们之间的关系

如何**组织数据**是程序中最重要的问题.

React的数据分为两种: prop和state, 它们的变化都会引起组件的重新渲染.

两者之间的区别在于: **prop是组件的对外接口**, 而**state是组件的内部状态**

### prop

从组件外部获取的数据

#### 给prop赋值 - 通过key=value的语法将字符串, 表达式将数据从父组件传递给子组件; 同样可以传递函数给组件, 让子组件在合适的时候调用函数, 将数据从子组件传递回父组件.

```js
<SampleButton id="sample" borderWidth={2} onClick={onButtonClick} style={{color: "red"}} />
```

#### 读取prop值 - 调用`super(props)`之后, 通过this.props来获取传入的数据

在父组件中调用子组件, 传入prop

```js
<ClickCounter caption="First" initValue={1}/>
<ClickCounter caption="Second" initValue={10} />
<ClickCounter caption="Third" initValue={100} />
```

在子组件中通过props来获取数据:

```js
import React, { Component } from 'react';

class ClickCounter extends Component {

  constructor(props) {
    super(props);

    this.state = { count: this.props.initValue || 0 };
    this.onClickButton = this.onClickButton.bind(this);
  }

  onClickButton() {
    this.setState({ count: this.state.count + 1 })
  }

  render() {

    const {caption} = this.props;

    return (
      <div>
        <button onClick={this.onClickButton}>Click Me</button>
        <div>
          {caption} Count: {this.state.count}
        </div>
      </div>
    );
  }
}

export default ClickCounter;

```

#### propTypes检查

prop是对外的接口, 就需要某种方式来声明**自己的接口规范**

- 组件支持**哪些prop**
- 每个prop应该是**什么格式**

React通过propTypes来定义接口规范, 在**运行时和静态检查时**检查外部是否正确的使用了接口.

- 首先需要安装独立的prop-types库

> [Typechecking With PropTypes](https://reactjs.org/docs/typechecking-with-proptypes.html)

```shell
$ yarn add prop-types
```

- 而后在组件上定义类型声明

```js
import PropTypes from 'prop-types';

ClickCounter.propTypes = {
  caption: PropTypes.string.isRequired,
  initValue: PropTypes.number
}
```

- 当父组件违反接口规则的时候, 浏览器会给出错误信息, 但是这个错误信息**并不会阻塞组件的运行, 只是一个辅助功能**

![](/images/2017-11-05-21-59-55.jpg)

- 这样的检查和提示只在开发环境有用, 在生产环境下只会导致包体积增大和性能下降, 通过构建工具可以在打包的时候自动剔除

### state

驱动组件渲染的除了prop, 还有组件的**内部状态state**


#### 初始化state

在**构造函数**中, 通过对state赋值, 完成初始化动作. state必须是一个对象

```js
  constructor(props) {
    super(props);

    this.state = { count: this.props.initValue || 0 };
  }

```

我们可以通过`||`操作符给state默认值, 也可以通过defaultProps给他默认值:

```js
ClickCounter.defaultProps = {
  initValue: 0
}
```

#### 读取和更新state

可以通过this.state来获取状态, 但是**必须**使用setState方法来更新状态, 这样才能**驱动组件重新渲染**

```js
  onClickButton() {
    this.setState({ count: this.state.count + 1 })
  }
```

### 对比prop和state

- prop用于定义外部接口, 而state用于定义内部状态
- prop在组件外部赋值, state在组件内部赋值
- 组件不应该改变prop的值, 而state存在的目的就是被组件改变
- prop是被多个子组件共享的, React没有办法阻止我们修改prop, 一旦这么做了, 程序会陷入一片混乱



## 组件的生命周期

React严格的定义了组件的声明周期:

- 装载阶段 - Mount 把组件**第一次**在DOM中渲染
- 更新过程 - Update 当组件被**重新渲染**的过程
- 卸载过程 - Unmount 组件**从DOM上移除**的过程

### 装载过程 Mount

当组件第一次被渲染的时候, 会依次调用如下方法:

- constructor
- getInitialState
- getDefaultProps
- componentWillMount
- render
- componentDidMount

#### constructor

并不是每个组件都需要构造函数, 特别是后面将会看到的**无状态组件**, 定义构造函数的目的是:

- 初始化state, 在最开始的地方初始化大家都要使用的state
- 绑定成员函数的this, ES6语法下, 成员函数在执行时的this不是和实例自动绑定的

> 也可以使用`this.foo - ::this.foo`进行绑定

#### getInitialState & getDefaultProps

getInitialState的返回值会用来**初始化state**, 只有使用React.createClass方法时才会发挥作用

getDefaultProps的返回值会用来**作为props的初始值**, 只有使用React.createClass方法时才会发挥作用

因为React.createClass已经被弃用, 这两个函数实际上已经**根本不会用到**

在ES6语法下, 通过**给this.state赋值**来完成第一项工作, 通过**defaultProps**来完成第二项工作.


#### render

render是组件中最重要的函数, React组件的父类对其他函数都有默认实现, render是我们**必须实现**的函数

render函数并不做实际的渲染动作, 而是**返回一个JSX描述的结构**, 最终由react来完成渲染.

如果确实没有内容需要渲染, 可以让render返回**null或者false**, 告诉react不需要渲染任何DOM元素

render应该是一个**纯函数**, 完全根据**state和props**来决定结果, 而且不要产生任何副作用

#### componentWillMount & componentDidMount

```
componentWillMount -> render -> componentDidMount
```

- componentWillMount 运行在render之前, 这时还**没有任何元素渲染在DOM上**
- componentDidMount 运行在render之后, 这时**已经完成了DOM的加载**

在ClickCounter组件中添加如下日志输出: 
```js

  constructor(props) {
    super(props);

    console.log(`enter constructor ${this.props.caption}`);

      // ...
  }
  componentWillMount() {
    console.log(`enter componentWillMount ${this.props.caption}`);
  }

  componentDidMount() {
    console.log(`enter componentDidMount ${this.props.caption}`);
  }

  render() {
    console.log(`enter render ${this.props.caption}`);
    // ...
  }
``` 

可以在控制台看到执行顺序, constructor/componentWillMount/render都是按顺序执行的, componentDidMount**并不是紧挨着render之后调用**, 当三个组件的render都被调用后, 三个组件的componentDidMount才**连在一起调用**

React的组件只是返回一个JSX结构, 框架来决定什么时候渲染或装载, 它会将**所有结果组合起来, 尽可能减少渲染的次数**, 而后才调用各个组件的componentDidMount方法. 

![](/images/2017-11-05-22-56-51.jpg)

> 另外一个特点, componentWillMount可以在服务端或客户端调用, 而componentDidMount只能在客户端调用. 这在后续的服务端渲染时会发挥作用.
> 真正的装载不可能在服务端完成, 服务器返回的只能是一个字符串, 在浏览器装载之后才能执行componentDidMount

componentDidMount给了我们一个很好的位置处理**只有浏览器才做的逻辑**, 例如AJAX来获取数据并填充组件的内容

另外一个使用场景, **与其他UI库配合**, 因为componentDidMount被调用的时候, DOM元素已经完全就位, 可以安全的进行UI操作.

### 更新过程 Update

当prop或state被修改的时候, 会触发组件的重新渲染, 并依次调用如下方法:

- componentWillReceiveProps
- shouldComponentUpdate
- componentWillUpdate
- render
- componentDidUpdate

#### componentWillReceiveProps(nextProps)

当组件的render的方法被调用时, componentWillReceiveProps就会被调用, **无论props是否真的发生变化**

注意, 调用setState方法并**不会**调用componentWillReceiveProps

在CounterPanel中增加一个方法, 强制执行父组件的刷新, 并在子组件中新增log输出: 

```js
// CounterPanel <- ParentComponent
<button onClick={() => this.forceUpdate()}>Click</button>

// ClickCounter <- ChildrenComponent
componentWillReceiveProps(nextProps) {
  console.log(`enter componentWillReceiveProps ${this.props.caption}`);
}
```

- onClick={() => this.forceUpdate()} - 通过匿名函数添加毁掉**不是推荐做法**, 因为每次渲染都会创建一个新的函数对象
- forceUpdate - **不是推荐方法**, 强制刷新会导致不必要的渲染动作

在父组件上点击按钮, 通过控制台可以看到执行顺序:


![](/images/2017-11-06-00-15-35.jpg)

可以看到, 无论父组件传递的props是否变化, 都会调用到该函数, 所以在方法内部, **有必要对比this.props和nextProps**

#### shouldComponentUpdate(nextProps, nextState)

除了render函数, shouldComponentUpdate可能是最重要的一个函数.

render决定了组件应该渲染什么, 而shouldComponentUpdate决定了**组件什么时候不需要渲染**

和render函数一起, 是**仅有的两个**需要返回的函数.

只要使用的恰当, 它能够**大大提高React组件的性能**, 根据当前的state和props, 结合下一个props和state来判断**返回true还是false**, 默认的实现是返回true, 即每次都渲染.

#### componentWillUpdate & componentDidUpdate

如果shouldComponentUpdate返回true, React就会一次调用componentWillUpdate, render, componentDidUpdate

与装载过程不同的是componentDidUpdate会**在客户端和服务端都执行**, componentDidUpdate用来处理DOM更新后与其他UI的配合, 正常情况下服务端不会调用它.

### 卸载过程 Unmount

卸载过程只涉及componentWillUnMount一个函数, 在DOM被清除之前, 用来处理一些**清理动作**

componentWillUnMount做的工作往往和componentDidMount相关, 为了**避免内存泄漏**, 清理掉componentDidMount中创建的DOM元素.


![](/images/2017-11-06-01-01-37.jpg)


## 组件向外传递数据

例如在我们的例子中, 需要在父组件计算三个组件之和, 这就需要我们将子组件的数据传递到父组件.

我们通过props传递一个回调函数, 让子组件通过这种方式来**向外传递数据**

props本身是一个对象, 而函数在JavaScript中是一等公民, 同样可以进行传递.

```js
  onClickButton() {

    const previousValue = this.state.count;
    const newValue = previousValue + 1;

    this.setState({ count: newValue })
    this.props.onUpdate(previousValue, newValue)
  }
```

父组件定义这个回调函数:

```js
class CounterPanel extends Component {

  constructor(props) {
    super(props)
    this.state = { sum: 111 }

    this.onUpdate = this.onUpdate.bind(this);
  }

  onUpdate(previousValue, newValue) {
    if (newValue !== previousValue) {
      this.setState({ sum: this.state.sum + (newValue - previousValue) })
    }
  }

  render() {
    return (
      <div>
        <ClickCounter caption="First" initValue={1} onUpdate={this.onUpdate}/>
        <ClickCounter caption="Second" initValue={10} onUpdate={this.onUpdate}/>
        <ClickCounter caption="Third" initValue={100} onUpdate={this.onUpdate}/>

        <div>
          <span>Sum is : {this.state.sum}</span>
        </div>
      </div>
    );
  }
}
```

另外, 需要在props上定义函数的对外规范和默认值:

```js
ClickCounter.propTypes = {
  caption: PropTypes.string.isRequired,
  initValue: PropTypes.number,
  onUpdate: PropTypes.func
}

ClickCounter.defaultProps = {
  initValue: 0,
  onUpdate: f => f // 默认是一个什么都不做的函数
}
```


## React组件state和props的局限

在上述例子中, 数据被**重复存储**, 在父组件和子组件中都保存了一份, 这就带来了**数据一致性**的问题, 在复杂的应用中, 这类问题很难定位和处理.

直观的处理方式是在**一个组件上记录所有状态**, 另一个方式是保存在**组件之外, 形成全局状态**, 第二种方案就是Flux和Redux中Store的概念.

另外一个问题是, 如果组件层级较深, 数据就需要被**层层传递**, 中间的父组件必须处理与他无关的prop, 这违反了**低耦合**的原则.