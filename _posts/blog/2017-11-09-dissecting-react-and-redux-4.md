---
layout: post
title:  "<深入浅出React和Redux> - React组件的性能优化"
date:   2017-11-09 00:14:52
categories: 读书笔记
---

# React组件的性能优化

## 单个React组件的性能优化

React通过Virtual DOM来提高组件渲染的性能, 通过**最小修改**的方式刷新真实DOM

但是, **对比Virtual DOM**依然是不小的性能开销, 尽量在计算和对比之前判断渲染结果是否有变化

> [Profiling Components with the Chrome Performance Tab](https://reactjs.org/docs/optimizing-performance.html#profiling-components-with-the-chrome-performance-tab)

在界面上切换一个任务的完成状态, 可以看到, 页面上所有的TodoItem都被刷新, 而实际上需要刷新的只有一个TodoItem:


![](/images/2017-11-09-00-43-29.jpg)


> "过早优化是万恶之源"

> 所谓的"过早优化"指的是**没有量化证据**的情况下, 开发者对性能优化的猜测, 对于性能有关键影响的部分, 优化并不嫌早.

> 如果能够优化性能的同时, 又能避免代码变得复杂, 何乐而不为

在我们的例子中, 对于实例数量是动态的组件, 一点点时间的浪费会随着数量增多而累积成为很长的时间

利用组件声明周期中的shouldComponentUpdate函数来避免多余的刷新:

```js
// TodoItem.js
  shouldComponentUpdate(nextProps, nextState) {
    return (nextProps.text !== this.props.text) || (nextProps.completed !== this.props.completed)
  }
```

![](/images/2017-11-09-00-44-27.jpg)

可以看到, 优化之后TodoItem的刷新**只进行了一次**, 整个列表的刷新时间从1.24ms降低到1.06ms

如果在每个组件中都编写shouldComponentUpdate会是一件非常**繁琐**的工作, 好在react-redux的connect方法内置了该方法的实现, 但是需要注意, 内置的方法是进行**浅层对比**, 在我们的例子中, 即使将TodoItem改造为connect模式, 也无法避免渲染:

```js
export default connect()(TodoItem)
```

![](/images/2017-11-09-00-55-20.jpg)

原因在于TodoList组件中的使用方式, onToggle和onRemove每次都会创建一个**新的匿名函数**, 这对于浅层比较来说, 每次的props都是不同的: 

```js
  <TodoItem
    key={todo.id}
    text={todo.text}
    completed={todo.completed}
    onToggle={() => onToggle(todo.id)}
    onRemove={() => onRemove(todo.id)}
  />

```

解决方法有两个:

1. onToggle和onRemove指向唯一确定的函数
2. 将onToggle和onRemove封装在TodoItem内部

第二种方案使得组件的内聚性更好, 这里只实现第二种方案:

```js
// todoList.js 增加一个id字段
  <TodoItem
    id={todo.id}
    key={todo.id}
    text={todo.text}
    completed={todo.completed}
  />

function mapStateToProps(state = [], ownProps) {
  return {
    todos: filterByCompleteState(state.todos, state.filter)
  }
}

// 移除动作分发

export default connect(mapStateToProps)(TodoList);


// TodoItem.js

import React from 'react';
import { connect } from 'react-redux';
import { toggleTodo, removeTodo } from '../actions'

const TodoItem = ({ text, completed, onToggle, onRemove }) => (
  <li className="list-group-item">
    <input type="checkbox" checked={completed ? 'checked' : ''} onClick={onToggle} />
    <span>{text}</span>
    <button className="btn btn-sm btn-light" onClick={onRemove}>x</button>
  </li>
);

// 在内部处理动作分发
const mapDispatchToProps = (dispatch, ownProps) => {
  const { id } = ownProps
  return {
    onToggle: () => dispatch(toggleTodo(id)),
    onRemove: () => dispatch(removeTodo(id))
  }
}

export default connect(null, mapDispatchToProps)(TodoItem)

```

可以看到, 只执行了一个TodoItem的刷新

![](/images/2017-11-09-01-23-43.jpg)


## 多个React组件的性能优化

## 用reselect提高数据获取性能


