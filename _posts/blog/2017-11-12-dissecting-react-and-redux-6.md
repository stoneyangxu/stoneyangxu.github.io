---
layout: post
title:  "<深入浅出React和Redux> - 扩展Redux"
date:   2017-11-12 16:19:34
categories: 读书笔记
---

# 扩展Redux

Redux本身提供了强大的数据流管理功能, 而其更强大之处在于它提供了**扩展能力**

## 中间件

中间件是一些**函数**, 相互之间是**独立的**, 用于定制对特定action的处理过程


![](/images/2017-11-12-16-29-19.png)

- 中间件是**独立**的函数
- 中间件是可以**组合**使用的
- 中间件有一个**统一**的接口

### 中间件接口

Redux框架中, 中间件处理的是**action对象**, action在进入到reducer通过dispatch派发之前, 会先经过中间件

中间件会**接收**到action对象, 经过处理后**交给**下一个中间件, 或者**中断**这个action的处理

```js
function doNothingMiddleware({ dispatch, getState }) {
    return function(next) {
        return function(action) {
            return next(action)
        }
    }   
} 
```

- 每个中间件都是一个**函数**
- 返回一个接受**next参数**的函数
- 接受next参数的函数, 又返回一个接受**action参数**的函数
- next本身也是一个**函数**, 中间件调用next函数通知redux自己的工作已经**结束**
- dispatch和getState是Redux上的两个函数, 不过并不是所有中间件都能用到

借助上述这些函数, 中间件可以完成很多工作:

- 调用dispatch**派发**一个新的action对象
- 调用getState**获得**当前状态
- 调用next告诉Redux当前中间件**工作完毕**, 让下一个中间件处理
- **访问**action对象上的数据

> Redux中间件借助函数式的思想, 让每个函数功能尽可能的小, 通过函数的嵌套组合来实现复杂功能

redux-thunk的实现如下:

```js
function createThunkMiddleware(extraArgument) {
    return ({ dispatch, getState }) => next => action => {
        if (typeof action === 'function') {
            return action(dispatch, getState, extraArgument)
        }   
        return next(action)
    }
}

const trunk = createThunkMiddleware()
export default thunk
```

- ({ dispatch, getState }) => next => action => 是ES6提供的箭头语法, 用来表示嵌套函数
- dispatch函数的返回结果是**不可预测**的, 中间件不能依赖dispatch的返回值

### 使用中间件

中间件有两种使用方法:

- 使用applyMiddleware来包装, 通过createStore产生一个新撞见的Store

```js
import { createStore, applyMiddleware } from 'redux'
import thunkMiddleware from 'redux-thunk'

const configureStore = applyMiddleware(thunkMiddleware)(createStore)
const store = configureStore(reducer, initialState)
```

这种方法如果需要增加其他中间件, 使用起来会很不方便, 这种方法已经很少使用

- 把applyMiddleware的结果当做Store Enhancer

```js
import { createStore, applyMiddleware, compose } from 'redux'
import thunkMiddleware from 'redux-thunk'

const middlewires = [thunkMiddleware]
if (process.env.NODE_ENV !== 'production') {
  middlewires.push(require('redux-immutable-state-invariant').default())
}

const win = window
const storeEnhancers = compose(
  applyMiddleware(...middlewires),
  (win && win.devToolsExtension) ? win.devToolsExtension() : (f) => f,
)

export default createStore(reducer, {}, storeEnhancers)

```

通过compose连接多个中间件, 第一个参数是已有的中间件, 中间件会被顺序执行; 将storeEnhancers作为参数传递给createStore



### Promise中间件

实现一个处理Promise的中间件, 以便于更好的理解中间件的工作方式

```js
function isPromise(obj) {
  return obj && typeof obj.then === 'function'
}

export default ({dispatch}) => next => action => {
  return isPromise(action) ? action.then(dispatch) : next(action)
}
```

进一步完善功能:

```js
function isPromise(obj) {
  return obj && typeof obj.then === 'function'
}

export default ({dispatch}) => next => action => {

  const {types, promise, ...rest} = action

  if (!isPromise(promise) || !(types && types.length === 3)) {
    return next(action)
  }

  const [PENDING, DONE, FAIL] = types
  dispatch({...rest, type: PENDING})

  return promise.then(
    (result) => dispatch({...rest, result, type: DONE}),
    (error) => dispatch({...rest, error, type: FAIL})
  )
}
```

- 处理异步请求的进行中, 成功和失败三种状态
- 首先派发PENDING状态, 当任务完成或失败的时候, 再派发DONE或FAIL
- 上述promise中间件处理的action格式为

```js
{
    promise: fetch(apiUrl),
    types: ['pending', 'success', 'failure']
}
```

相应的, weather中的代码可以修改为:

```js
export const fetchWeather = (cityCode) => {
  const apiUrl = `/data/cityinfo/${cityCode}.html`
  
  return {
    promise: fetch(apiUrl).then(response => {
      if (response.status !== 200) {
        throw new Error(`Fail to get response with status ${response.status}`)
      }

      response.json().then(responseJson => responseJson.weatherinfo)
    }),
    types: [FETCH_STARTED, FETCH_SUCCESS, FETCH_FAILURE]
  }
}
```

### 中间件开发原则

- 首先, 要明确中间件的**目的**, 保证中间件**简洁和独立**, 通过**组合**来完成更复杂的功能
- 每个中间件要独立存在, 但是要**考虑到**其他中间件的存在
- 一个中间件如果产生了**新的**action对象, 应该使用dispatch派发, 而不是使用next函数, 因为next函数不会让action经历所有中间件

## Store Enhancer

中间件可以用来增强dispatch方法, 但也**仅限于此**, 如果想要对Redux Store进行**更深层次**的定制, 就需要使用Store Enhancer

### 增强器接口

Store Enhancer是一个**函数**, 接受一个createStore模样的函数作为参数, 并返回一个**新的**createStore

```js
const doNothingEnhancer = (createStore) => (reducer, preloadedState, enhancer) => {
    return createStore(reducer, preloadedState, enhancer) 
}
```

创建store, 定制store并将store返回

```js
const logEnhancer = (createStore) => (reducer, preloadedState, enhancer) => {
  const store = createStore(reducer, preloadedState, enhancer)

  const originalDispatch = store.dispatch
  store.dispatch = (action) => {
    console.log(`dispatch action:`, action)
    originalDispatch(action)
  }

  return store
}
```

可以定制的接口包括:

- dispatch
- getState
- subscribe
- replaceReducer
