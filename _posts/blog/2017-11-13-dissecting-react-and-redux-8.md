---
layout: post
title:  "<深入浅出React和Redux> - 动画"
date:   2017-11-13 23:53:06
categories: 读书笔记
---

# 动画

动画的作用并不是核心功能, 但是能够提高**用户体验**

## 动画的实现方式

### CSS方式

通过CSS的transition方式来渲染动画, 运行**效率**比脚本方式高, 浏览器**原生支持**, 不会造成JavaScript的**解释执行负担**

不过: 

- 动作规则的定义是基于时间和速度曲线的, 可能不利于动画的流畅
- 动画随着用户的操作, 可能会被打断
- 动画效果难以进行调试

### 脚本方式

脚本方式最大的好处是提供了更强的**灵活性**, 开发者可以**任意控制**动画效果

缺点是:

- 消耗的计算资源更多

> requestAnimationFrame 不是以固定的16ms间隔去渲染, 而是通过脚本给他传入一个回调函数, 浏览器来决定何时调用, 并根据逝去的时间来决定界面的渲染状态
> 16ms : 保证每秒60帧, 以达到流畅的动画效果



## ReactCSSTransitionGroup

需要借助react-addons-css-transition-group这个库

```shell
$ yarn add react-addons-css-transition-group
```

接着引入这个库:

```js
import TransitionGroup from 'react-addons-css-transition-group'
```

TransitionGroup的主要工作是帮助我们在**装载和卸载**的时候处理动画过程

使用它来包裹我们的todoList组件渲染结果:

```js
  return (
    <ul className="row list-group">
      <TransitionGroup transitionName="fade" transitionEnterTimeout={500}
        transitionLeaveTimeout={200} >
        {
          todos.map(todo => (
            <TodoItem
              id={todo.id}
              key={todo.id}
              text={todo.text}
              completed={todo.completed}
            />
          ))
        }
      </TransitionGroup>
    </ul>
  )
```

- transitionName指定了CSS类的前缀规则

```css
.fade-enter {
  opacity: .01;
}

.fade-enter.fade-enter-active {
  opacity: 1;
  transition: opacity 500ms ease-in;
}

.fade-leave {
  opacity: 1;
}

.fade-leave.fade-leave-active {
  opacity: .01;
  transition: opacity 200ms ease-in;
}

```

### ReactTransitionGroup规则

ReactTransitionGroup依赖于CSS, 它的作用是让React组件在生命周期的特定阶段使用不同的CSS规则

#### 类名规则

- transitionName 是所有css类名的统一**前缀**
- enter 代表**装载**
- leave 代表**卸载**
- active 代表动画**结束**时的状态

在加载动画的时候, 并不是一次装载的: 

- 首先装载-enter类
- 在下一个**时钟周期**才装载-enter-active的类
- leave的过程类似

#### 动画时间长度

动画的持续时间在**两个地方**都要指定, 第一个是在组件声明上, 第二个是在transition-duration上指定, 两个时间一般是**一致的**

- 组件上指定的时间是active类的**存在**时间
- css中的时间是**实际**的动画时间

加入两者不一致, 前者小于后者, 会在第一个时间到达时**移除**active类, 导致动画**突变**, 直接到达最终结果

#### 装载时机

TransitionGroup要发挥作用, 必须**自身**已经装载完成

也就是说, TransitionGroup必须在**目标组件之外**装载, 这样才能够使动画生效

#### 首次装载

如果已经存在了若干TodoItem组件实例, 这些组件是不会有动画效果的, 因为enter过程不包括首次装载, 只包括**装载完成之后新加入的组件**

如果想让首次装载也存在动画效果, 就要使用**appear**过程, 即-appear和-appear-action为后缀的样式

appear有一点特殊, 通常来说首次装载是不需要动画的, 所以默认控制动画的开关是**关闭**的

```js
var defaultProps = {
  transitionAppear: false,
  transitionEnter: true,
  transitionLeave: true
};
```

## React-Motion动画库

React-Motion使用的是**脚本**方式来实现动画

### 设计原则

- 大部分情况下, 友好的API比性能更重要
- 不要以时间和速度曲线来定义动画, React-Motion使用了两个参数来定义动画: 刚度和阻尼

在React-Motion中大量的使用了**函数作为子组件**的模式

```shell
$ yarn add react-motion
```

```js
<Motion defaultStyle={{ x: 100 }}
  style={
    { x: spring(0, { stiffness: 100, damping: 100 }) }
  } >
  {value => <div>{Math.ceil(value.x)}</div>}
</Motion>
```

- defaultStyle 指定了**初始值**
- style 指定了**目标值**
- 通过stiffness和damping定义了**节奏**
- 期间不断**调用子函数**, 完成动画过程

Motion内部使用requestAnimationFrame来出发, 具体的绘制是由**子组件**来完成
