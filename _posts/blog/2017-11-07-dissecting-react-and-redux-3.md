---
layout: post
title:  "<深入浅出React和Redux> - 模块化React和Redux应用"
date:   2017-11-07 23:16:07
categories: 读书笔记
---

# 模块化React和Redux应用

实际工作中, 应用汇变得很复杂, 需要划分为多个模块进行管理, 有效的组织代码结构, 精心设计状态树, 并且充分利用开发辅助工具.

## 模块化应用的要点

React适合**视图层**的工作, 而Redux才适合担当**状态管理**的工作.

开始一个新应用的时候, 需要考虑:

- 代码文件的**组织结构**
- 确定**模块边界**
- Store的**状态树设计**

## 代码文件的组织方式

### 按角色组织

在MVC框架下, 经常按照文件的角色划分目录:

```
controllers/
models/
views/
```

在MVC框架的影响下, Redux中也存在按角色组织代码的方式:

```
reducers/
actions/
components/
containers/
```

但是, 按这种方式组织, 代码十分**不利于扩展和迁移**, 每次修改都需要跨目录进行.

### 按功能组织

按功能组织就是将**完成相同功能的代码放在一起**

```
todoList/
    actions.js
    actionTypes.js
    index.js
    reducer.js
    views/
        component.js
        container.js
filter/
    actions.js
    actionTypes.js
    index.js
    reducer.js
    views/
        component.js
        container.js
```

在这种组织形式下, 每个功能对应一个模块, 每个模块对应一个目录

- actionTypes.js 定义action类型
- actions.js 定义action构造函数, 决定了这个模块可以接受的动作
- reducer.js 定义这个模块如何响应actions.js中定义的动作
- views 目录包含所有的react组件, 包括无状态组件和容器组件
- index.js 把所有角色导入, 然后统一导出, 作为模块的唯一入口

## 模块接口

模块化软件的要求: **"理想情况下, 只需要新增代码就能增加功能, 而不是修改现有代码"**

React+Redux能够帮助我们更为接近这一目标.

- 低耦合 - 模块之间的依赖关系应该简单而且清晰
- 高内聚 - 模块应该封装自己的功能

在React+Redux应用中, 一个模块就是**React组件, 加上actions, reducer**组成的小整体

如果在filter模块中想使用todoList中的功能:

```js
import * from actions from '../todoList/actions'
import container as TodoList from '../todoList/views/container'
```

这种导入方式使得filter模块依赖于todoList的内部结构, 并没有做到低耦合

我们需要一个**接口**, 将内部逻辑封装起来: index.js

```js
import * as actions from './actions'
import reducer from './reducer'
import view from './views/container'

export {actions, reducer, view}
```

filter模块中的使用代码就会变为:

```js
import {actions, reducer, view as TodoList} from '../todoList'
```

模块的内部结构被封装, 客户端只需要关注模块的接口

## 状态树的设计

Redux中所有的状态都保存在一个store上, 状态树的设计直接决定了**要写哪些reducer**, 以及**action怎么写**, 是**程序逻辑的源头**

- 一个模块控制一个状态节点 - reducer在状态树上的修改权是**互斥**的, 不可能让两个reducer修改同一个节点
- 避免冗余数据 - 冗余数据是**一致性**的大敌, 相对于性能问题, 一致性问题更加重要
- 树形结构扁平 - 深层次的数据结构很难进行管理

## Todo应用实例

Todo应用包含三部分功能:

- 待办事项列表
- 增加新待办的输入框和按钮
- 待办事项过滤器, 选择不同状态的待办

前两个部分功能更为紧密, 所以整体划分为两个模块: **todos和filter**

### todo的状态设计

- 使用数组来表示待办事项列表
- 每个待办事项包含id, text, completed来表示
- 过滤器包含三种状态: 'all', 'completed', 'uncompleted'

最终的状态树格式类似于:

```js
{
    todos: [
        {
            id: 0,
            text: 'First todo',
            completed: false
        },  
        {
            id: 1,
            text: 'Second todo',
            completed: true
        } 
    ],
    filter: 'all'
}
```

index.js中使用react-redux来导入Provider:

```js
import store from './Store'
import { Provider } from 'react-redux';

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>
  , document.getElementById('root'));
registerServiceWorker();
```

App.js中引入todos和filter

```js
import { Todos } from './todos';
import { Filter } from './filter';

class App extends Component {
  render() {
    return (
      <div>
        <Todos />
        <Filter />
      </div>
    );
  }
}
```

todos和filter只包含基本的文本显示:

```
├── App.js
├── Store.js
├── filter
│   ├── index.js
│   └── views
│       └── Filter.js
├── index.js
├── todos
    ├── index.js
    └── views
        └── Todos.js

```

### action构造函数

确定状态树结构之后, 就可以给每个模块创建action构造函数了

分为actionTypes和actions两个文件

```js
// todos/actionTypes.js
export const ADD_TODO = 'TODO/ADD'
export const TOGGLE_TODO = 'TODO/TOGGLE'
export const REMOVE_TODO = 'TODO/REMOVE'

// todo/actions.js
import * as ActionTypes from './actionTypes'

let nextTodoId = 0

export const addTodo = (text) => ({
  type: ActionTypes.ADD_TODO,
  id: nextTodoId++,
  text,
  completed: false 
})

export const toggleTodo = (id) => ({
  type: ActionTypes.TOGGLE_TODO,
  id 
})

export const removeTodo = (id) => ({
  type: ActionTypes.REMOVE_TODO,
  id 
})

// todo/index.js
import Todos from './views/Todos'
import * as actions from './actions'

export {Todos, actions}
```

- 操作类型应该保持**全局唯一**, 使用模块名作为前缀
- 字符串格式的类型能够提供更好的可读性, 而且便于程序调试
- () => ({}) 的写法, 省略了return语句

### 组合reducer

每个模块都会拥有一个reducer, 而Redux的createStore方法只接受一个reducer, 所以要将多个reducer**组合**起来传递给createStore方法

```js
// Store.js

import { createStore, combineReducers } from 'redux'
import { reducer as todoReducer } from './todos'
import { reducer as filterReducer } from './filter'

const reducer = combineReducers({
  todos: todoReducer,
  filter: filterReducer
})
export default createStore(reducer)
```

- combineReducers参数对象上的字段名对应了**状态树上的属性名**
- combineReducers返回一个新的reducer, 在执行的时候, 会将整体的state**拆分开**, 分别交给不同的reducer执行
- reducer分别执行后, 将结果再**合并**为一个整体的state

```js
// todos/reducer.js
import * as ActionTypes from './actionTypes'

export default (state = [], action) => {
  switch (action.type) {
    case ActionTypes.ADD_TODO:
      return [
        {
          id: action.id,
          text: action.text,
          completed: false
        },
        ...state
      ]
    case ActionTypes.TOGGLE_TODO:
      return state.map(todo => {
        if (todo.id === action.id) {
          return { ...todo, completed: !todo.completed }
        } else {
          return todo
        }
      })
    case ActionTypes.REMOVE_TODO:
      return state.filter(todo => todo.id !== action.id)
    default:
      return state
  }
}
```

### Todo视图

首先将Todos视图拆分为两个: AddTodo和TodoList

```js
export default () => {
  return (
    <div>
      <TodoList />
      <AddTodo />
    </div>
  )
}
```

- 使用纯函数进一步简化语法

#### AddTodo

负责获取用户输入, 点击按钮后创建新的待办

```js
import React, { Component } from 'react';
import { connect } from 'react-redux';
import { addTodo } from '../actions'

class AddTodo extends Component {

  constructor(props) {
    super(props)

    this.refInput = this.refInput.bind(this)
    this.onSubmit = this.onSubmit.bind(this)
  }

  refInput(node) {
    this.input = node
  }

  onSubmit(e) {
    e.preventDefault()

    const input = this.input
    if (!input.value.trim()) {
      return
    }

    this.props.onAdd(input.value)
    input.value = ''
  }

  render() {
    return (
      <div className="row">
        <form className="form-inline" onSubmit={this.onSubmit}>
          <input className="col-8 form-control" ref={this.refInput} />
          <button className="col-3 btn btn-primary" type="submit">Add</button>
        </form>
      </div>
    )
  }
}

function mapDispatchToProps(dispatch, ownProps) {
  return {
    onAdd: (text) => {
      dispatch(addTodo(text))
    }
  }
}

export default connect(null, mapDispatchToProps)(AddTodo);
```

- ref - input元素上的ref会绑定到一个函数, 在函数refInput中, 参数是**实际的DOM节点**, 绑定动作是发生在**组件装载时**
- onSubmit - 表单的默认提交行为会刷新页面, 为了避免刷新, 需要使用e.preventDefault()**屏蔽默认行为**
- onAdd - 在mapDispatchToProps中将onAdd方法绑定到redux的分发动作
- connect - 第一个参数是null, 因为组件并没有从外部接受属性

#### TodoList

首先完成基本的展示功能:

```js
import React from 'react';
import { connect } from 'react-redux';

const TodoList = ({todos}) => {
  return (
    <ul className="row list-group">
      {
        todos.map(todo => (
          <li key={todo.id} className="list-group-item">{todo.text}</li>
        ))
      }
    </ul>
  )
}

function mapStateToProps(state = [], ownProps) {
  return {
    todos: state.todos
  }
}

export default connect(mapStateToProps)(TodoList);
```

- 在JSX中不能使用for或while这样的循环语句, 它们是语句, 而不是表达式
- 在map循环生成的结构中, 每个元素都要带有**key**属性, react以此为依据进行性能优化, 避免刷新整个列表

接着, 添加待办的状态切换和移除功能, 可以将li提取到独立的组件todoItem.js

```js
import React from 'react';
import { connect } from 'react-redux';

const TodoItem = ({ text, completed, onToggle, onRemove }) => (
  <li className="list-group-item">
    <input type="checkbox" checked={completed ? 'checked' : ''} onClick={onToggle}/>
    <span>{text}</span>
    <button className="btn btn-sm btn-light" onClick={onRemove}>x</button>
  </li>
)

export default TodoItem
```

在TodoList中绑定切换状态和删除功能:

```js
function mapDispatchToProps(dispatch, ownProps) {
  return {
    onToggle: (id) => {
      dispatch(toggleTodo(id))
    },
    onRemove: (id) => {
      dispatch(removeTodo(id))
    }
  }
}
```

利用redux提供的bindActionCreators方法进一步简化代码:

```js
import { bindActionCreators } from 'redux'

const mapDispatchToProps = (dispatch) => bindActionCreators({
  onToggle: toggleTodo,
  onRemove: removeTodo
}, dispatch)
```

再进一步, 直接使用对象映射:

```js
const mapDispatchToProps = {
  onToggle: toggleTodo,
  onRemove: removeTodo
}
```

#### filter视图

- filter组件是一个简单的无状态组件

```js
function Filter() {
  return (
    <p>
      <Link filter={ALL}>{ALL}</Link>
      <Link filter={COMPLETED}>{COMPLETED}</Link>
      <Link filter={UNCOMPLETED}>{UNCOMPLETED}</Link>
    </p>
  )
}
```

- Link组件略微复杂, 作为容器组件获取状态和派发事件

```js
const Link = ({ active, children, onClick }) => {
  if (active) {
    return <b>{children}</b>
  } else {
    return <a href="#" className="badge badge-primary" onClick={(e) => {
      e.preventDefault()
      onClick()
    }}>{children}</a>
  }
};

function mapStateToProps(state, ownProps) {
  return {
    active: state.filter === ownProps.filter
  }
}

function mapDispatchToProps(dispatch, ownProps) {
  return {
    onClick: () => {
      dispatch(setFilter(ownProps.filter))
    }
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(Link);
```

- 当修改状态树中的filter值之后, todoList根据过滤条件过滤列表

```js
function filterByCompleteState(todos, filter) {
  switch (filter) {
    case FilterTypes.ALL:
      return todos;
    case FilterTypes.COMPLETED:
      return todos.filter(todo => todo.completed)
    case FilterTypes.UNCOMPLETED:
      return todos.filter(todo => !todo.completed)
    default:
      return todos;
  }
}

function mapStateToProps(state = [], ownProps) {
  return {
    todos: filterByCompleteState(state.todos, state.filter)
  }
}
```

### 不使用ref

我们在提交表单的时候, 通过ref获取input的真实dom节点, ref的用法**非常脆弱**

React的产生就是**避免操作DOM**, 直接访问DOM很容易产生失控的情况

替代方案是使用组件状态来记录DOM元素的值:

```js
  onInputChange(e) {
    this.setState({
      value: e.target.value
    })
  }

  onSubmit(e) {
    e.preventDefault()

    const value = this.state.value
    if (!value.trim()) {
      return
    }

    this.props.onAdd(value)
    this.setState({
      value: ''
    })
  }

  render() {
    return (
      <div className="row">
        <form className="form-inline" onSubmit={this.onSubmit}>
          <input className="col-8 form-control" value={this.state.value} onChange={this.onInputChange} />
          <button className="col-3 btn btn-primary" type="submit">Add</button>
        </form>
      </div>
    )
  }
```


## 开发辅助工具

### Chrome 扩展包

- React Devtools - 可以检视React组件树
- Redux Devtools - 可以检视Redux数据流, 并且将组件跳到任何一个**历史状态**
- React Perf - 发现React组件的性能问题(**react v16已经不支持**)

### redux-immutable-state-invariant辅助包

每个reducer都必须是**纯函数**, 不能修改state或action, 负责会让程序**陷入混乱**

虽然这是一个规则, 但是总会被无心破坏, 使用redux-immutable-state-invariant提供的Redux中间件

在每次**派发动作**之后做一个检查, 如果发现违反了规则, 就会给出警告

### 工具应用

对于工具的启用, 需要在Store中修改一些代码, 通过**中间件**来启用扩展功能.

```shell
$ yarn add redux-immutable-state-invariant --dev
```

```js
//Store.js
import { createStore, combineReducers, applyMiddleware, compose } from 'redux'

const win = window // 引用window是为了避免包过大, UglifyJsPlugin无法处理window这种全局变量

const middlewires = []
if (process.env.NODE_ENV !== 'production') {
  middlewires.push(require('redux-immutable-state-invariant').default())
}

const storeEnhancers = compose(
  applyMiddleware(...middlewires),
  (win && win.devToolsExtension) ? win.devToolsExtension() : (f) => f,
)

export default createStore(reducer, {}, storeEnhancers)
```

- storeEnhancers - 为Store提供增强功能
- compose - 将多个enhancer组合在一起
- redux-immutable-state-invariant 只有在开发环境才加入到增强功能
- 使用require时因为ES6的import不能出现在if语句中, 而且必须处于顶部


![](/images/2017-11-09-00-03-09.jpg)
