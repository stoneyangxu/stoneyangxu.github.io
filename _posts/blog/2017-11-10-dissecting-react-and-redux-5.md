---
layout: post
title:  "<深入浅出React和Redux> - Redux和服务器通信 & 测试"
date:   2017-11-10 23:16:52
categories: 读书笔记
---

# Redux和服务器通信

## React组件访问服务器

如果没有Redux之类的简单应用, React也可以自己承担与服务器通信的工作.

现在有一个趋势是使用**原生的fetch函数**来访问网络资源

fetch函数返回一个Promise对象来获取服务器响应, 对于不支持的浏览器, 可以通过**polyfill**来增加支持

> https://github.com/github/fetch

使用 http://www.weather.com.cn/data/cityinfo/101280601.html 来查询深圳的天气信息

因为这个地址查询天气信息, 会出现**跨域**问题, 解决的方式是通过**代理**, 页面访问服务器API接口, 服务器把这个请求**转发**给另一个域名下的API

create-react-app内置代理功能, 在package.json中通过**proxy**字段来配置

```js
"proxy": "http://www.weather.com.cn/"
```

这种代理只适合**开发环境**, 在生产环境下需要开发自己的代理服务器

### React组件访问服务器的生命周期

网络请求是**异步的**, 而组件加载时**同步的**, 不能让组件一直等待网络返回

- 在装载的时候, 请求还未返回, 需要让组件显示**正在加载**之类的提示信息
- 当服务器获取结果后, 引发组件的**刷新**
- 通常在 componentDidMount 中进行请求, 因为此时DOM已经装载完毕, 请求可能会依赖DON的内容

```js
class Weather extends Component {
  constructor(props) {
    super(props)
    this.state = {
      weather: null
    }
  }

  componentDidMount() {
    const apiUrl = `/data/cityinfo/101280601.html`
    fetch(apiUrl).then(response => {
      if (response.status !== 200) {
        throw new Error(`Fail to get response with status ${response.status}`)
      }

      response.json().then(responseJson => {
        this.setState({ weather: responseJson.weatherinfo })
      }).catch(error => { this.setState({ weather: null }) })
    }).catch(error => { this.setState({ weather: null }) })
  }

  render() {
    if (this.state.weather) {
      const { city, weather, temp1, temp2 } = this.state.weather
      return (
        <div>
          {city} {weather} 最低气温 {temp1} 最高气温 {temp2}
        </div>
      );
    } else {
      return <div>Loading...</div>
    }
  }
}
```

界面显示为:

```
深圳 晴 最低气温 14℃ 最高气温 23℃
```

### React访问服务器的优缺点

将请求放在组件内, 会是组件变得复杂

Redux是用来帮助**管理应用状态**的, 应该尽量将状态管理以及服务请求放在Redux中

## Redux访问服务器

使用Redux访问服务器, 要解决的是**异步**问题

Redux的单向数据流是**同步**操作, redux提供了一种**thunk**的方式, 使用一个独立的**redux-thunk**发布包, 为了保持redux的中立性, 没有集成到一起, 让开发者可以自由的选择其他的解决方式

```shell
$ yarn add redux-thunk
```

> thunk是一个术语, 表示辅助调用另一个子程序的子程序

```js
const f = (x) => {
    return x() + 5
}

const g = () => {
    return 3 + 4
}

f(g)
```
> f把x当做一个子程序来执行, 好处就是g的执行只有在f实际执行的时候才执行, 起到**延迟**执行的作用

在Redux架构下, 一个action对象通过store.dispatch派发, 在action被reducer处理之前, 会经过**中间件**环节, 这就是产生异步的机会

```js
// Store.js
import thunkMiddleware from 'redux-thunk'

const middlewires = [thunkMiddleware]
```

当我们让Redux帮忙处理一个异步操作的时候, 需要派发一个action对象, 但是这个对象比较特殊, 叫做**异步action对象**

这个对象并不是一个普通的对象, 而是一个**函数**, 在这个函数上reducer无法获取type字段, 所以也不做什么实际处理

不过, 有了redux-thunk之后, 这些对象会提前**被中间件拦截**, 不会真的接触到reducer

redux-thunk的工作是**检查action是不是函数**, 不是函数就放行, 如果是函数, 就**执行函数**, 并且把**dispatch和getState作为参数传入**, 不会继续让action派发到reducer

```js
// 同步action
const increment = () => {
    type: ActionType.INCREMENT
}

// 异步action
const incrementAsync = () => {
    return (dispatch) => {
        setTimeout(() => {
            dispatch(increment())
        }, 1000)
    }
}

```

异步action构造函数incrementAsync返回一个新的函数, 这样一个函数被dispatch派发之后, 会被redux-thunk中间件执行, 于是setTimeout发生作用, 1秒之后利用dispatch函数派发同步的action构造函数increment的执行结果, 这就是**异步action的工作原理**, 异步action最后依然需要产生同步action派发才能对Redux产生影响

redux-thunk引入了**一次函数执行**, 这个函数能够**访问到dispatch和getState**, 给异步操作带来可能

### 异步操作的模式

一个访问服务器的action, 至少要设计**三个**action类型:

- 表示异步操作**开始**的action类型
- 表示异步操作**成功**的action类型
- 表示异步操作**失败**的action类型

三种action会使React组件进入三种不同的状态:

- 异步操作正在进行中
- 异步操作已经成功完成
- 异步操作已经失败

使用redux-thunk来改造我们获取天气的应用:

- actionTypes.js定义三种action类型

```js
export const FETCH_STARTED = 'WEATHER/FETCH_STARTED'
export const FETCH_SUCCESS = 'WEATHER/FETCH_SUCCESS'
export const FETCH_FAILURE = 'WEATHER/FETCH_FAILURE'
```

- status.js定义三种状态

```js
export const LOADING = 'loading'
export const SUCCESS = 'success'
export const FAILURE = 'failure'
```

- actions.js定义同步action和异步action

```js
import { FETCH_STARTED, FETCH_SUCCESS, FETCH_FAILURE } from './actionTypes'

export const fetchWeatherStarted = () => ({
  type: FETCH_STARTED
})

export const fetchWeatherSuccess = (result) => ({
  type: FETCH_SUCCESS,
  result
})

export const fetchWeatherFailure = (error) => ({
  type: FETCH_FAILURE,
  error
})

export const fetchWeather = (cityCode) => {
  return (dispatch) => {
    const apiUrl = `/data/cityinfo/${cityCode}.html`
    
    // 派发同步状态, 组件现实loading
    dispatch(fetchWeatherStarted())

    fetch(apiUrl).then(response => {
      if (response.status !== 200) {
        throw new Error(`Fail to get response with status ${response.status}`)
      }

      response.json().then(responseJson => {
        // 派发成功状态, 组件现实天气
        dispatch(fetchWeatherSuccess(responseJson.weatherinfo))
      }).catch(error => {
        throw new Error(`Invalid json response ${error}`)
      })
    }).catch(error => {
      // 派发失败状态, 组件现实失败
      dispatch(fetchWeatherFailure(error))
    })
  }
}
```

fetchWeather是异步action, 返回**一个新的函数**, 其模式是固定的:

```js
export const asyncAction = () => {
    return (dispatch, getState) => {
        // 1. 调用异步函数
        // 2. 派发start状态的同步action
        // 3. 请求成功时, 派发success状态的同步action
        // 4. 请求失败时, 派发failure状态的同步action
    }
}
```

- reducer.js 只会接受**同步action**, 更新state

```js
import * as Status from './status'
import { FETCH_STARTED, FETCH_SUCCESS, FETCH_FAILURE } from './actionTypes'

export default (state = { status: Status.LOADING }, action) => {
  switch (action.type) {
    case FETCH_STARTED:
      return { status: Status.LOADING }
    case FETCH_SUCCESS:
      return { ...state, status: Status.SUCCESS, ...action.result }
    case FETCH_FAILURE:
      return { status: Status.FAILURE }
    default:
      return state;
  }
}
```

- Weather.js中, 只响应同步action出发的状态变更

```js
const Weather = ({status, cityName, weather, lowestTemp, highestTemp}) => {

  switch(status) {
    case Status.LOADING: {
      return <div>Loading...</div>
    }
    case Status.SUCCESS: {
      return (
        <div>
          {cityName} {weather} 最低气温 {lowestTemp} 最高气温 {highestTemp}
        </div>
      );
    }
    case Status.FAILURE: {
      return <div>Failed...</div>
    }
    default:
    throw new Error(`unexcepted status ${status}`)
  }
}

const mapStateToProps = (state) => {
  const weatherData = state.weather
  return {
    status: weatherData.status,
    cityName: weatherData.city,
    lowestTemp: weatherData.temp1,
    highestTemp: weatherData.temp2
  }
}

const mapDispatchToProps = (dispatch) => {
  dispatch(weatherActions.fetchWeather(101280601))
}

export default connect(mapStateToProps, mapDispatchToProps)(Weather)
```

### 异步操作的中止

请求发送出去并返回, 会有一段时间延迟, 在这个过程中, 用户可能希望**中止**请求.

还有另一种情况是, 通过界面操作会**连续多次发送请求**, 界面的显示要看**哪个请求最后返回**, 这个顺序是无法控制的

这种问题有两个解决方案:

1. 发起请求后**锁定界面**, 让用户无法发起下一次操作, 但是这样的用户体验不好
2. 通过请求库的api来中止, 但是fetch不支持, 这就需要一些技巧来进行处理

```js

let nextSeqId = 0

export const fetchWeather = (cityCode) => {
  return (dispatch) => {
    const apiUrl = `/data/cityinfo/${cityCode}.html`

    const seqId = ++nextSeqId
    const dispatchIfValid = (action) => {
      if (seqId === nextSeqId) {
        return dispatch(action)
      }
    }

    dispatchIfValid(fetchWeatherStarted())

    fetch(apiUrl).then(response => {
      if (response.status !== 200) {
        throw new Error(`Fail to get response with status ${response.status}`)
      }

      response.json().then(responseJson => {
        dispatchIfValid(fetchWeatherSuccess(responseJson.weatherinfo))
      }).catch(error => {
        throw new Error(`Invalid json response ${error}`)
      })
    }).catch(error => {
      dispatchIfValid(fetchWeatherFailure(error))
    })
  }
}
```

通过seqId来判断是否是最新的请求, 只有最后的请求才继续派发action

## Redux异步操作的其他方法

redux-thunk并不是Redux处理异步请求的唯一方式, 其余的还包括:

- redux-saga
- redux-effects
- redux-side-effects
- redux-loop
- redux-observable
- ... ...

所有这些库都需要**中间件或者Enhancer**来实现对异步操作的支持

在选择库的时候, 需要考虑以下几个方面:

- 在数据流中什么**时机**插入异步操作
- 库的**大小**如何
- 学习**曲线**如何
- 是否会和其他Redux库**冲突**

除了redux-thunk之外, 还有一种异步模式, 将Promise作为特殊处理的异步action对象, 这种方案**更加易用**

- redux-promise
- redux-promises
- redux-simple-promise
- redux-promise-middleware

# 单元测试

## 单元测试的原则

单元测试是保障质量的**第一道防线**

TDD(测试驱动开发)更是开发者的工作**神器**

单元测试应该让开发者的工作**更轻松更高效**, 而不是成为**包袱**

需要注意:

1. 不要执着于百分百的覆盖率
2. 程序架构的可测试性非常重要, 解决方式就是**拆分**

只要应用得当, React和Redux的可测试性非常高, 因为测试目标大多是**纯函数**

## 单元测试环境的搭建

### 测试框架

进行React和Redux的测试有多种选择:

- Mocha+Chai
- React自家出品的Jest

create-react-app中自带了Jest库, 使用`npm run test`来运行测试

```
No tests found related to files changed since last commit.
Press `a` to run all tests, or run Jest with `--watchAll`.

Watch Usage
 › Press a to run all tests.
 › Press p to filter by a filename regex pattern.
 › Press t to filter by a test name regex pattern.
 › Press q to quit watch mode.
 › Press Enter to trigger a test run.
```

Jest会寻找:

- 文件名以**.test.js**为后缀的文件
- 保存在**__test__**目录下的文件

测试文件的组织有两种方式:

- src与test并列, 使用后缀的方式命名, 缺点是导入被测代码的时候, 需要很长的相对路径
- 在每个组件文件夹内使用__test__目录保存测试文件, 缺点是目录看着不是特别整洁

### 测试代码组织

单元测试代码的最小单位是**测试用例**, 每个测试用例考验的是被测试对象在某个**特定场景**下是否有正确行为

```js
it('renders without crashing', () => {
  const div = document.createElement('div');
  ReactDOM.render(<App />, div);
});
```

为了测试被测对象的**多种场景**, 会创建多个测试用例, 多个用例通过**测试套件**的方式组织

```js
describe('render', () => {
  it('renders without crashing', () => {
    const div = document.createElement('div');
    ReactDOM.render(<App />, div);
  });
});
```

使用describe的目的是为了**重用环境设置**, 利用函数作用域, 为多个it创建前置条件

无论使用Mocha还是Jest, 都是使用beforeAll, beforeEach, describe, it, afterEach, afterAll来组织测试代码的

### 辅助工具

#### Enzyme

Enzyme是由Airbnb贡献的开源项目: https://github.com/airbnb/enzyme

```shell
$ yarn add enzyme enzyme-adapter-react-16 --dev
```

Enzyme任务, 测试**目标组件**时, 并不需要把React组件的DOM树都渲染出来, 尤其是包含复杂子组件的React组件. 完整的渲染会耗费大量的精力去准备**测试环境**, 只需要渲染**顶层组件**就好了, 不需要测试子组件

Enzyme支持三种渲染方法:

- shallow - 只渲染顶层组件, **不**渲染子组件
- mount - 渲染**完整**的组件和子组件, 借助模拟的浏览器环境完成事件处理功能
- render - 渲染**完整**的React组件, 但**只产生HTML**, 并不进行事件处理

```js
import React from 'react';
import { expect } from 'chai';
import { shallow } from 'enzyme';
import sinon from 'sinon';

import MyComponent from './MyComponent';
import Foo from './Foo';

describe('<MyComponent />', () => {
  it('renders three <Foo /> components', () => {
    const wrapper = shallow(<MyComponent />);
    expect(wrapper.find(Foo)).to.have.length(3);
  });

  it('renders an `.icon-star`', () => {
    const wrapper = shallow(<MyComponent />);
    expect(wrapper.find('.icon-star')).to.have.length(1);
  });

  it('renders children when passed in', () => {
    const wrapper = shallow((
      <MyComponent>
        <div className="unique" />
      </MyComponent>
    ));
    expect(wrapper.contains(<div className="unique" />)).to.equal(true);
  });

  it('simulates click events', () => {
    const onButtonClick = sinon.spy();
    const wrapper = shallow(<Foo onButtonClick={onButtonClick} />);
    wrapper.find('button').simulate('click');
    expect(onButtonClick).to.have.property('callCount', 1);
  });
});
```

#### sinon.js

React和Redux已经尽量让单元测试面对的是**纯函数**, 但还是不能避免有些测试对象**依赖于**一些其他元素, 比如对于**异步action**, 还会涉及网络请求, 这就需要**模拟**网络访问的结果

```shell
$ yarn add sinon --dev
```

```js
it('makes a GET request for todo items', function () {
    sinon.stub(jQuery, 'ajax');
    getTodos(42, sinon.spy());

    assert(jQuery.ajax.calledWithMatch({ url: '/todo/42/items' }));
});

```

#### redux-mock-store

在测试的时候, 很多场景下并不需要完整的Redux功能, 一个模拟的Store会更方便易用

```shell
$ yarn add redux-mock-store --dev
```

## 单元测试的实例

### action构造函数测试

普通action构造函数式很简单的单元测试

```js
// todos/actions.js
import * as ActionTypes from './actionTypes'

let nextTodoId = 0

export const addTodo = (text) => ({
  type: ActionTypes.ADD_TODO,
  id: nextTodoId++,
  text,
  completed: false
})
```

对应的测试用例:

```js
import {addTodo} from './actions'
import {ADD_TODO} from './actionTypes'

describe('todos/addTodo', () => {
  it('should create an action to add todo', () => {
    const text = 'first todo'
    const action = addTodo(text)

    expect(action.text).toBe(text)
    expect(action.completed).toBe(false)
    expect(action.type).toBe(ADD_TODO)
  });
});
```

- 预设参数
- 调用纯函数
- 用expect验证纯函数的返回结果

### 异步action构造函数测试

异步action因为存在副作用, 所以单元测试会更复杂

一个异步action对象就是一个**函数**, 被派发到redux-thunk中间件的时候被执行

在测试过程中, 我们并不希望使用真实的store, 使用redux-mock-store更加合适, dispatch最好不要做实际的派发动作, 只要能够把**被派发的对象**记录下来, 用来验证就可以了

```js
import thunk from 'redux-thunk'
import configureStore from 'redux-mock-store'
import {stub} from 'sinon'

import * as actions from './actions'
import * as actionTypes from './actionTypes'

const middlewares = [thunk]
const createMockStore = configureStore(middlewares)

describe('fetchWeather', () => {
  let stubbedFetch;
  const store = createMockStore()

  beforeEach(() => {
    stubbedFetch = stub(global, 'fetch')
  })

  afterEach(() => {
    stubbedFetch.restore()
  })

  it('should dispatch fetchWeatherSuccess action on fetch weather', () => {
    const mockResponse = Promise.resolve({
      status: 200,
      json: () => Promise.resolve({
        weatherinfo: {}
      })
    })

    stubbedFetch.returns(mockResponse);

    return store.dispatch(actions.fetchWeather(1)).then(() => {
      const dispatchedActions = store.getActions();
      expect(dispatchedActions.length).toBe(2);
      expect(dispatchedActions[0].type).toBe(actionTypes.FETCH_STARTED);
      expect(dispatchedActions[1].type).toBe(actionTypes.FETCH_SUCCESS);
    });
  });
});

```

- 使用createMockStore代替真实的createStore来创建模拟Store
- 在beforeEach和afterEach中模拟fetch方法并每次清理
- 使用stubbedFetch.returns方法模拟fetch的返回结果
- 使用store.dispatch派发action并验证结果
- getActions方法是redux-mock-store的方法, 获取已经派发的action列表
- Jest有两种方式校验异步任务: dong参数或者直接返回Promise

### reducer测试

reducer是纯函数, 测试非常简单

```js
import * as actions from './actions'
import reducer from './reducer'
import * as Status from './status'

describe('weather/reducer', () => {
  it('should return loading status', () => {
    const action = actions.fetchWeatherStarted()
    const newState = reducer({}, action)
    expect(newState.status).toBe(Status.LOADING)
  });
});
```

### 无状态React组件测试

无状态的React组件可以使用Enzyme的shallow方法来渲染, 让测试专注于自身

```js
import React from 'react';
import Enzyme from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';
import { shallow, mount, render } from 'enzyme';

import Filter from './filter'
import Link from './link'
import * as FilterTypes from '../filterTypes'

Enzyme.configure({ adapter: new Adapter() });

describe('filter', () => {
  it('should render three link', () => {
    const wrapper = shallow(<Filter />)

    expect(wrapper.contains(<Link filter={FilterTypes.ALL}>{FilterTypes.ALL}</Link>)).toBeTruthy()
    expect(wrapper.contains(<Link filter={FilterTypes.COMPLETED}>{FilterTypes.COMPLETED}</Link>)).toBeTruthy()
    expect(wrapper.contains(<Link filter={FilterTypes.UNCOMPLETED}>{FilterTypes.UNCOMPLETED}</Link>)).toBeTruthy()
  });
});
```

### 被连接的React组件测试

使用redux的时候, 通过connect返回的组件叫做被连接的React组件, 这样的组件依赖于Redux Store实例, 而且能够提供**真实**的内容, 不再使用redux-mock-store

```js

```

