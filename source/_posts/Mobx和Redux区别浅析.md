---
title: Mobx和Redux区别浅析
date: 2020-12-30 21:20:37
tags:
- 数据存储
categories: 数据存储
author: 杨红梅
keywords: mobx  redux 数据存储 react
description: mobx数据存储与redux的区别浅析
cover: https://img13.360buyimg.com/imagetools/jfs/t1/160373/20/777/20345/5fed31f7Ee319f018/55fc552bd22cf5ef.jpg
top_img: https://img13.360buyimg.com/imagetools/jfs/t1/155613/6/3401/33206/5fed3209Ea1b9a4b6/612acc6c35a1e280.jpg
---



在React项目中做数据管理，Redux已经占据半边天下，为什么会出现Mobx呢，它是什么，能给我们带来什么，和Redux相比有什么优缺点呢，今天，我就自己学习的一点知识，为大家做一个简单的分享，希望让你对Mobx有一个大致的了解。


## 目录

1.	Mobx是什么
2.	编程思维方式的不同
3.	Store的区别
4.	储存数据形式区别
5.	操作对象方式不同
6.	代码对比
7.	Props的注入不同
8.  Mobx优缺点总结


## Mobx是什么

Mobx是一个透明函数响应式编程（Transparently Functional Reactive Programming，TFRP）的状态管理库，它使得状态管理简单可伸缩：
Anything that can be derived from the application state, should be derived. Automatically.

任何起源于应用状态的数据应该自动获取。
其原理如图：
![](img1.png)

1.	Action：定义改变状态的动作函数，包括如何变更状态；

2.	Store：集中管理模块状态（State）和动作（action）；
3.  Derivation（衍生）：从应用状态中派生而出，且没有任何其他影响的数据，我们称为derivation（衍生），衍生在以下情况下存在：
a. 用户界面；
b. 衍生数据；
衍生主要有两种：

1. Computed Values（计算值）：计算值总是可以使用纯函数（pure function）从当前可观察状态中获取；
2. Reactions（反应）：反应指状态变更时需要自动发生的副作用，这种情况下，我们需要实现其读写操作；

```js
import {observable, autorun} from 'mobx';

var todoStore = observable({
    /* some observable state */
    todos: [],

    /* a derived value */
    get completedCount() {
        return this.todos.filter(todo => todo.completed).length;
    }
});

/* a function that observes the state */
autorun(function() {
    console.log("Completed %d of %d items",
        todoStore.completedCount,
        todoStore.todos.length
    );
});

/* ..and some actions that modify the state */
todoStore.todos[0] = {
    title: "Take a walk",
    completed: false
};
// -> synchronously prints: 'Completed 0 of 1 items'

todoStore.todos[0].completed = true;
// -> synchronously prints: 'Completed 1 of 1 items'

```


## 编程思维方式的不同

Redux更多的是遵循函数式编程（Functional Programming, FP）思想，而Mobx则更多从面相对象角度考虑问题。

Redux提倡编写函数式代码，如reducer就是一个纯函数（pure function），如下：
``` js
    (state, action) => {
  return Object.assign({}, state, {
    ...
  })
}

```

 纯函数，接受输入，然后输出结果，除此之外不会有任何影响，也包括不会影响接收的参数；对于相同的输入总是输出相同的结果。

 Mobx设计更多偏向于面向对象编程（OOP）和响应式编程（Reactive Programming），通常将状态包装成可观察对象，于是我们就可以使用可观察对象的所有能力，一旦状态对象变更，就能自动获得更新。



## store的区别

store是应用管理数据的地方，在Redux应用中，我们总是将所有共享的应用数据集中在一个大的store中，而Mobx则通常按模块将应用状态划分，在多个独立的store中管理。


## 储存数据形式区别

Redux默认以JavaScript原生对象形式存储数据，而Mobx使用可观察对象：

1. Redux需要手动追踪所有状态对象的变更；
2. Mobx中可以监听可观察对象，当其变更时将自动触发监听；


## 操作对象方式不同

Redux状态对象通常是不可变的（Immutable）：
```js
switch (action.type) {
  case REQUEST_POST:
  	return Object.assign({}, state, {
      post: action.payload.post
  	});
  default:
    retur nstate;
}

```

我们不能直接操作状态对象，而总是在原来状态对象基础上返回一个新的状态对象，这样就能很方便的返回应用上一状态；而Mobx中可以直接使用新值更新状态对象。



## 代码对比

在Redux应用中，我们首先需要配置，创建store，并使用redux-thunk或redux-saga中间件以支持异步action，然后使用Provider将store注入应用：

```js
// src/store.js
import { applyMiddleware, createStore } from "redux";
import createSagaMiddleware from 'redux-saga'
import React from 'react';
import { Provider } from 'react-redux';
import { BrowserRouter } from 'react-router-dom';
import { composeWithDevTools } from 'redux-devtools-extension';
import rootReducer from "./reducers";
import App from './containers/App/';

const sagaMiddleware = createSagaMiddleware()
const middleware = composeWithDevTools(applyMiddleware(sagaMiddleware));

export default createStore(rootReducer, middleware);

// src/index.js
…
ReactDOM.render(
  <BrowserRouter>
    <Provider store={store}>
      <App />
    </Provider>
  </BrowserRouter>,
  document.getElementById('app')
);

```

Mobx应用则可以直接将所有store注入应用：

```js
import React from 'react';
import { render } from 'react-dom';
import { Provider } from 'mobx-react';
import { BrowserRouter } from 'react-router-dom';
import { useStrict } from 'mobx';
import App from './containers/App/';
import * as stores from './flux/index';

// set strict mode for mobx
// must change store through action
useStrict(true);

render(
  <Provider {...stores}>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </Provider>,
  document.getElementById('app')
);

```

## Props的注入不同

- Redux

```js
// src/containers/Company.js
…
class CompanyContainer extends Component {
  componentDidMount () {
    this.props.loadData({});
  }
  render () {
    return <Company
      infos={this.props.infos}
      loading={this.props.loading}
    />
  }
}
…

// function for injecting state into props
const mapStateToProps = (state) => {
  return {
    infos: state.companyStore.infos,
    loading: state.companyStore.loading
  }
}

const mapDispatchToProps = dispatch => {
  return bindActionCreators({
      loadData: loadData
  }, dispatch);
}

// injecting both state and actions into props
export default connect(mapStateToProps, { loadData })(CompanyContainer);

```

- Mobx

```js
@inject('companyStore')
@observer
class CompanyContainer extends Component {
  componentDidMount () {
    this.props.companyStore.loadData({});
  }
  render () {
    const { infos, loading } = this.props.companyStore;
    return <Company
      infos={infos}
      loading={loading}
    />
  }
}

```

## Mobx优缺点总结

-  **优点**

1.	学习成本少：Mobx基础知识很简单，学习了半小时官方文档和示例代码就搭建了新项目实例；而Redux确较繁琐，流程较多，需要配置，创建store，编写reducer，action，如果涉及异步任务，还需要引入redux-thunk或redux-saga编写额外代码，Mobx流程相比就简单很多，并且不需要额外异步处理库；

2. 面向对象编程：Mobx支持面向对象编程，我们可以使用@observable and @observer，以面向对象编程方式使得JavaScript对象具有响应式能力；而Redux最推荐遵循函数式编程，当然Mobx也支持函数式编程；

3. 模版代码少：相对于Redux的各种模版代码，如，actionCreater，reducer，saga／thunk等，Mobx则不需要编写这类模板代码；

- **缺点**

1.	过于自由：Mobx提供的约定及模版代码很少，这导致开发代码编写很自由，如果不做一些约定，比较容易导致团队代码风格不统一，所以当团队成员较多时，确实需要添加一些约定；

2. 可拓展，可维护性：也许你会担心Mobx能不能适应后期项目发展壮大呢？确实Mobx更适合用在中小型项目中，但这并不表示其不能支撑大型项目，关键在于大型项目通常需要特别注意可拓展性，可维护性，相比而言，规范的Redux更有优势，而Mobx更自由，需要我们自己制定一些规则来确保项目后期拓展，维护难易程度；



## 参考文档

1. [Redux & Mobx](https://www.robinwieruch.de/redux-mobx)
2. [Mobx](https://mobx.js.org/index.html)







