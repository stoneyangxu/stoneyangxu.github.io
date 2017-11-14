---
layout: post
title:  "<深入浅出React和Redux> - 从Flux到Redux"
date:   2017-11-06 01:21:34
categories: 读书笔记
---

# 从Flux到Redux

针对React自己来管理数据的问题, 将使用Flux或Redux来管理数据

## Flux

Flux是React官方推出的数据管理框架, 如果说React用来替代jQuery, 那么Flux就是用来替换Backbone, Ember等MVC框架的

其核心思想是**单向数据流**

### MVC框架的缺陷

- Model 负责管理数据, 大部分业务逻辑应该位于Model中
- View 负责渲染用户界面, 应该**避免**在View中包含业务逻辑
- Controller 负责**接受用户输入**, 并调用Model中的业务逻辑, 把结果交给View进行渲染


![](/images/2017-11-06-01-27-45.png)


但是, **MVC真的很快就会变得非常复杂**, 不同模块之间复杂的依赖关系, 会导致系统**脆弱而且不可预测**

在实际使用的时候, Model和View很容易形成复杂的调用关系:


![](/images/2017-11-06-01-31-17.png)

原因就是View和Model始终存在, 开发者为了**便利**, 往往忽视理想的数据流规则, 直接让两者进行对话, 从而导致了上图中复杂的依赖关系.

Flux就是一个**更严格的数据流控制**


![](/images/2017-11-06-01-36-26.png)

- Dispatcher 处理**动作分发**, 维持**Store之间的依赖**
- Store 负责**存储数据和处理数据相关的逻辑**
- Action **驱动Dispatcher**的对象
- View 视图部分, 负责**显示用户界面**

在使用Flux的时候, 每增加一个新的功能, 并不需要新增一个Dispatcher, 只需要**增加一种新的Action类型**, Dispatcher对外的接口并不改变

### Flux应用

首先进行安装:

```shell
$ yarn add flux
```

#### Dispatcher

几乎所有应用都只有**一个**Dispatcher

创建AppDispatcher.js声明唯一的对象, 其他代码使用的时候, 只需要**引用**他, 它的作用就是**派发action**

```js
import { Dispatcher } from 'flux';

export default new Dispatcher();
```

#### action

表示一个**动作**
- 一个**纯粹的数据对象**
- 必须包含一个**type**字段, 代表动作类型, 应该是**字符串类型**

定义action通常需要两个文件:

- 一个定义action的**类型**
- 一个定义action的**构造函数**

分成两个文件的原因是: Store中会针对不同类型进行不同操作, 会**单独引用类型文件**

在Actions.js中, 借助Dispatcher对象分发不同类型的action

```js
// ActionTypes.js
export const INCREASE = 'increase';
export const DECREASE = 'decrease';

// Actions.js
import * as ActionTypes from './ActionTypes';
import AppDispatcher from './AppDispatcher';

export const increase = (caption) => {
  AppDispatcher.dispatch({
    type: ActionTypes.INCREASE,
    caption: caption
  })
}

export const decrease = (caption) => {
  AppDispatcher.dispatch({
    type: ActionTypes.DECREASE,
    caption: caption
  })
}
```

#### Store

也是一个对象, 用来**存储应用状态**, 同时**接受Dispatcher派发的动作**, 由此来**决定是否需要更新**应用状态

使用Flux之后会带来一个弊端: **文件数量大大增加**, 所以需要使用单独的stores来存储所有的Store文件

- CounterStore.js

```js
const counterValues = {
  'First': 0,
  'Second': 10,
  'Thired': 100
}

const CounterStore = Object.assign({}, EventEmitter.prototype, {
  getCounterValues: () => {
    return counterValues
  },
  emitChange: () => {
    this.emit(CHANGE_EVENT)
  },
  addChangeListener: (callback) => {
    this.on(CHANGE_EVENT, callback)
  },
  removeChangeListener: (callback) => {
    this.removeListener(CHANGE_EVENT, callback)
  }
})
```

当Store状态发生变化的时候, 使用**消息**的方式建立Store和View之间的联系, 更新界面状态

理论上, getCounterValues应该返回一个**不可变**对象, 这里只是为了便于演示, 没有引入Immutable

Store只有**注册到Dispatcher**上才能发挥真正的作用

```js
CounterStore.dispatchToken = AppDispatcher.register((action) => {
  if (action.type === ActionTypes.INCREASE) {
    counterValues[action.caption]++;
    CounterStore.emitChange();
  } else if (action.type === ActionTypes.DECREASE) {
    counterValues[action.caption]--;
    CounterStore.emitChange();
  }
})
```

register函数会返回一个token值, 用来**在store之间的同步**


- SummaryStore.js

```js

function computeSummary(counterValues) {
  let summary = 0
  for (const key in counterValues) {
    if (counterValues.hasOwnProperty(key)) {
      summary += counterValues[key]
    }
  }

  return summary
}

const SummaryStore = Object.assign({}, EventEmitter.prototype, {
  getSummary: () => {
    return computeSummary(CounterStore.getCounterValues())
  }
})
```
SummaryStore虽然叫做Store, 但是他实际上不存储数据, 调用CounterStore获取数据的方法进行计算, 并返回计算结果

```js
SummaryStore.dispatchToken = AppDispatcher.register((action) => {
  if (action.type === ActionTypes.INCREASE || action.type === ActionTypes.DECREASE) {
    AppDispatcher.waitFor([CounterStore.dispatchToken])
    SummaryStore.emitChange()
  }
})
```

waitFor暗示了回调函数的**执行顺序**, 它会等待指定token对应的回调函数执行后才执行.

register只能声明**当有任何动作派发时,请调用我**, 而不是**让Store只监听某些action**, 当一个动作被派发的时候, Flux会把**所有**注册的回调函数都调用一遍, 由回调函数**自己判断**是否需要处理.

这种设计会让Dispatcher的逻辑**最简化**

#### View

首先, View不一定非要使用React, 可以使用任何UI库

在Flux框架下的React组件需要实现以下几个功能:

- 创建时, 读取Store状态来**初始化**
- Store上状态变化时, **同步更新**组件状态
- 如果需要View中更新Store, 只能**派发Action**


CounterPanel中, 只保留caption的赋值, 其余的属性都会通过Store来获取: 

```js
  <div>
    <ClickCounter caption="First" />
    <ClickCounter caption="Second" />
    <ClickCounter caption="Third" />

    <Summary />
  </div>
```

ClickCounter中通过Store获取初始数据:

```js
this.state = {
  count: CounterStore.getCounterValues()[props.caption]
}
```

并且绑定Store的变更事件:

```js
  onChange() {
    const newCount = CounterStore.getCounterValues()[this.props.caption];
    this.setState({count: newCount});
  }

  componentDidMount() {
    CounterStore.addChangeListener(this.onChange)
  }

  componentWillUnmount() {
    CounterStore.removeChangeListener(this.onChange)
  }
```

将+和-按钮的事件切换到helper函数的分发操作:

```js
  increase() {
    Actions.increase(this.props.caption)
  }

  decrease() {
    Actions.decrease(this.props.caption)
  }
```

完整代码参考: 

[stoneyangxu/dissecting-react-and-redux](https://github.com/stoneyangxu/dissecting-react-and-redux/tree/flux)

整体的数据流程:

- 点击View中的按钮, 在increase方法中调用Actions中的函数
- Actions中的函数调用Dispatcher的方法分发事件
- Dispatcher广播事件, 并且被CounterStore(或SummaryStore)中register注册的回调获取
- 回调函数中自行判断是否需要处理该事件, 并操作自身存储的数据, 触发新的事件通知View
- 新的事件是在组件的生命周期方法中绑定的, 从而获知数据变化并更新显示

### Flux的优势

在不使用Flux的时候, 数据都**保存在组件内**, 每个组件都需要维护自己的数据, 当应用变得复杂的时候, 这种关联难以维护

在Flux架构下, 组件专注于自身的职责**渲染组件**, 数据都是保存在Store中, 组件只根据**事件**将数据映射为界面.

Flux中最大的**优势**就是严格的**单向数据流**: 想要改变View, 必须通过Store来改变数据状态, 而Store中的数据状态必须通过派发一个action来改变, 这就是**规矩**

在这个规矩之下, 想**追溯**一个应用的逻辑就变得非常容易

MVC框架最大的缺点是**无法禁绝**Model和View之间的通信, 在Flux架构下, Store相当于Model, 而**Store只有get方法而没有set方法**, 想要修改Store就只能派发action, 这种限制**杜绝了混乱的数据流**

### Flux的不足

1. Store之间的**依赖关系** - 必须使用waitFor来进行控制, **最好的依赖管理是不产生依赖**
2. 难以进行**服务端渲染** - 服务端渲染输出的不是DOM而是字符串, flux不是设计出来进行服务端渲染的, 想要实现会很困难
3. Store混杂了逻辑和状态 - 如果需要**动态**替换Store, 就只能**整体替换**, 也就无法保留状态. 在开发时的调试, 或者生产环境下的**动态加载**, 也就是**热加载**

## Redux 

如果把Flux看做一个框架**理念**的话, Redux就是Flux的一种**实现**

### Redux的基本原则

- 唯一数据源
- 保持状态只读
- 数据改变只能通过纯函数完成

#### 唯一数据源

应用的状态数据保存在**唯一**的一个Store上

分为多个Store保存造成数据冗余或数据一致性的问题, 使用waitFor进行控制则会导致依赖关系, 使得应用变得复杂.

而这个唯一的Store解决了这些问题, 在Store上的状态是一个**树形**, 每个组件只使用树形结构上的一部分数据

#### 保持状态只读

改变状态的方法不是去修改状态上的值, 而是**创建**一个新的状态对象返回给Redux, 由Redux完成新的状态的组装

#### 数据改变只能通过纯函数完成

所谓纯函数就是一个**reducer**, 根据**上一个状态**和**action**规约为新的状态

```js
reducer(state, action)
```

函数的返回结果完全由state和action决定, 并且不产生任何副作用, 也不会修改state和action

```js
function reducer(state, action) => {
    const {counterCaption} = action;
    switch(action.type) {
        case ActionTypes.INCREASE: 
            return {...state, [counterCaption]: state[counterCaption] + 1};
        case ActionTypes.DECREASE: 
            return {...state, [counterCaption]: state[counterCaption] - 1};
        default: 
            return state;
    }
}
```

> "如果你愿意限制做事方式的灵活性, 你几乎总会发现可以做的更好"

### Redux实例

先不借助react-redux组件, 以便我们更好的了解Redux的工作方式

```shell
$ yarn add redux
```

ActionTypes.js与Flux版本没有任何区别

```js
export const INCREASE = 'increase';
export const DECREASE = 'decrease';
```

Actions.js则变得不同, flux中是将Dispatcher分发动作, 而在Redux中则是返回一个action对象:

```js
import * as ActionTypes from './ActionTypes';
import AppDispatcher from './AppDispatcher';

export const increase = (caption) => {
  return {
    type: ActionTypes.INCREASE,
    caption: caption
  }
}

export const decrease = (caption) => {
  return {
    type: ActionTypes.DECREASE,
    caption: caption
  }
}

export default {increase, decrease}
```

在Redux中没有Dispatcher存在, 在flux中其作用是将action对象分发给多个Store, 而Redux中只有一个Store, 所以将其简化为**Store上的dispatch方法**

创建一个全局唯一的Store.js:

```js
import { createStore } from 'redux'
import reducer from './Reducer'

const initValues = {
  'First': 0,
  'Second': 10,
  'Third': 100
}

const store = createStore(reducer, initValues)
export default store;
```

在初始值上只记录三个counter组件的值, Summary的值通过计算得出, 原则是: **避免冗余的数据**

创建Reducer.js, 根据action的type字段执行对应的数据操作:

```js
import * as ActionTypes from './ActionTypes'

export default (state, action) => {
  const { caption } = action

  switch (action.type) {
    case ActionTypes.INCREASE:
      return { ...state, [caption]: state[caption] + 1 }
    case ActionTypes.DECREASE:
      return { ...state, [caption]: state[caption] - 1 }
    default:
      return state
  }
}

```

Redux中将存储state的工作交给框架自身, 让reducer值关心**如何更新state**, 而不关心怎么存

接着是View部分: 
- 首先的区别是数据来源不同了, 通过唯一的store获取
- 其次, 通过store对象来订阅事件
- 更新的时候, 也是通过store对象的dispatch方法来派发动作

```js
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import Actions from '../Actions'
import store from '../Store'


class ClickCounter extends Component {

  constructor(props) {
    super(props);

    this.state = this.getOwnState()

    this.onChange = this.onChange.bind(this);
    this.increase = this.increase.bind(this);
    this.decrease = this.decrease.bind(this);
  }

  getOwnState() {
    return {
      value: store.getState()[this.props.caption]
    }
  }

  increase() {
    store.dispatch(Actions.increase(this.props.caption))
  }

  decrease() {
    store.dispatch(Actions.decrease(this.props.caption))
  }

  onChange() {
    this.setState(this.getOwnState());
  }

  componentDidMount() {
    store.subscribe(this.onChange)
  }

  componentWillUnmount() {
    store.unsubscribe(this.onChange)
  }

  render() {
    const { caption } = this.props;
    return (
      <div>
        <button onClick={this.increase}>+</button>
        <button onClick={this.decrease}>-</button>
        <span>{caption} Count: {this.state.value}</span>
      </div>
    );
  }
}

ClickCounter.propTypes = {
  caption: PropTypes.string.isRequired,
  initValue: PropTypes.number,
  onUpdate: PropTypes.func
}

ClickCounter.defaultProps = {
  initValue: 0,
  onUpdate: f => f // 默认是一个什么都不做的函数
}

export default ClickCounter;
```

#### 容器组件和傻瓜组件

在Redux框架下, React组件通常要完成两项工作:

- 读取store的状态初始化组件, 监听store的变化刷新组件, 派发action更新store
- 根据props和state渲染用户界面

根据单一职责原则, 这两个任务是可以拆分的:

- 容器组件 - 负责和Store打交道
- 展示组件 - 负责渲染界面, 一个**纯函数**, 而且不需要有状态, 即**无状态组件**

```js
import React from 'react';

// 缩略为一个纯函数
function Counter({onIncrease, onDecrease, caption, value}) { // 直接在参数位置解构
  return (
    <div>
      <button onClick={onIncrease}>+</button>
      <button onClick={onDecrease}>-</button>
      <span>{caption} Count: {value}</span>
    </div>
  );
}

export default Counter;

```

```js
// container

  render() {
    const { caption } = this.props;
    return (
      <Counter caption={caption}
        onIncrease={this.increase}
        onDecrease={this.decrease}
        value={this.state.value} />
    );
  }
```

#### 组件Context

在上述例子中, 对store的引用使用相对路径:

```js
import store from '../Store'
```

当项目规模变大, 路径会变得很长, 而且如果将组件独立发布, 根本无法获知store的位置

最好**只有一个地方导入store**, 就是在React应用的最顶层

当前我们掌握的方式是将store通过props层层传递下来, 这么做的问题就是**每一层都要传递这个props**

React提供了一个叫做**context**的功能, 本质上就是一个所有组件都能访问的上下文环境

首先创建一个Provider, 作为一个通用的**context提供者**:

```js
import { Component } from 'react'
import PropTypes from 'prop-types'

class Provider extends Component {

  getChildContext() {
    return {
      store: this.props.store
    }
  }

  render() {
    return this.props.children;
  }
}

Provider.childContextTypes = {
  store: PropTypes.object
}

export default Provider;
```

- getChildContext 返回的就是代表context的对象
- children 是一个特殊属性, 表示<Provider></Provider>之间的组件
- childrenContextTypes 让React认可它是一个context提供者
- childContextTypes和getChildContext的属性必须匹配

其次, 在顶层组件使用Provider包裹其他组件:

```js
ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>
  , document.getElementById('root'));
```

最后, 在底层组件使用store的时候需要进行相应的调整:

- 定义contextTypes, 与context匹配

```js
CounterContainer.contextTypes = {
  store: PropTypes.object
}
```
- 构造函数中, 传入context, 并传入父类的构造函数中

```js
  constructor(props, context) {
    super(props, context);
    // ...
  }
```

- 所有使用store的地方, 都通过this.context.store来使用

```js
  getOwnState() {
    return {
      value: this.context.store.getState()[this.props.caption]
    }
  }
```

## React-Redux

在上述例子中, 经过**组件拆分**和**提供Context**, 可以找到固定的规律, 使用react-redux库来帮助我们自动完成

```shell
$ yarn add react-redux
```

### react-redux提供Provider

react-redux要求的Provider要求store不光是一个object, 必须是包含三个函数的object:

- subscribe
- dispatch
- getState

```js
import { Provider } from 'react-redux';

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>
  , document.getElementById('root'));
```

### 使用connect连接容器组件和展示组件

语法格式如下, 作用是将**无状态组件转换为容器组件**

```js
export default connect(mapStateToProps, mapDispatchToProps)(connect)
```

- mapStateToProps 将Store上的状态转换为无状态组件的props
- mapDispatchToProps 将无状态组件的动作转换为派送给Store的动作

```js
import { connect } from 'react-redux';

function mapStateToProps(state, ownProps) {
  return {
    caption: ownProps.caption,
    value: state[ownProps.caption]
  }
}

function mapDispatchToProps(dispatch, ownProps) {
  return {
    onIncrease: () => {
      dispatch(Actions.increase(ownProps.caption))
    },
    onDecrease: () => {
      dispatch(Actions.decrease(ownProps.caption))
    }
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(Counter);
```

使用react-redux简化容器后, 可以将容器组件和无状态组件进行合并:

```js
import React from 'react';
import { connect } from 'react-redux';
import Actions from '../Actions'

function Counter({ onIncrease, onDecrease, caption, value }) {
  return (
    <div>
      <button onClick={onIncrease}>+</button>
      <button onClick={onDecrease}>-</button>
      <span>{caption} Count: {value}</span>
    </div>
  );
}

function mapStateToProps(state, ownProps) {
  return {
    caption: ownProps.caption,
    value: state[ownProps.caption]
  }
}

function mapDispatchToProps(dispatch, ownProps) {
  return {
    onIncrease: () => {
      dispatch(Actions.increase(ownProps.caption))
    },
    onDecrease: () => {
      dispatch(Actions.decrease(ownProps.caption))
    }
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(Counter);
```

## 总结

- React的设计思想是: UI=render(data), flux架构很好的遵循了**单向数据流**的原则
- Flux因为存在多个Store, 导致**数据容易**和**Store之间的依赖**
- Redux只有唯一的一个Store, 有效的填补了Flux的缺陷
- react-redux库提供了全局的Provider, store不需要层层传递, 而且通过connect方法简化了容器组件的创建