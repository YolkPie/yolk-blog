---
title: React Hooks的常见应用及一些原理
date: 2020-02-04 16:44:07
tags: Hooks
categories: React
author: 7Hua
keywords: Hooks
description: React团队希望，组件不要变成复杂的容器，最好只是数据流的管道，开发者可以根据需要组合管道。
---
# React Hooks的常见应用及一些原理

## 类组件（class）

```js
import React, { Component } from "react";

export default class Button extends Component {
  constructor() {
    super();
    this.state = { buttonText: "Click me, please" };
    this.handleClick = this.handleClick.bind(this);
  }
  handleClick() {
    this.setState(() => {
      return { buttonText: "Thanks, been clicked!" };
    });
  }
  render() {
    const { buttonText } = this.state;
    return <button onClick={this.handleClick}>{buttonText}</button>;
  }
}
```
类组件的缺点：
- 大型组件很难拆分和重构，也很难测试
- 业务逻辑分散在组件的各个方法中，导致重复逻辑或关联逻辑
- 组件类引入复杂的编程模式，比如render、props

----

## 函数组件
目的：React团队希望，组件不要变成复杂的容器，最好只是数据流的管道，开发者可以根据需要组合管道。**完全不使用类就能写出一个全功能组件**

React很早就支持函数组件
```js
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

这种写法有重大限制，必须是纯函数，不能包含状态，也不支持生命周期方法，因此无法取代类。

## Hooks
React核心思想是将页面拆成一堆独立的、可复用的组件，并且用自上而下的数据流串联起来。但是在实际项目中很多组件冗长且难以复用。
React Hooks要解决的问题是**状态逻辑复用**。
**React Hooks 的意思是，组件尽量写成纯函数，如果需要外部功能和副作用，就用钩子把外部代码"钩"进来。**  
React默认提供了一些常用函数，同时也允许封装自己的钩子，React约定，所有钩子函数一律用use前缀命名，意思是为函数引入外部功能。 

4种常见函数：  
- useState()
- useContext()
- useReducer()
- useEffect()

## useState(状态钩子)
纯函数不能有状态，useState用于为函数组件引入状态。
```js
const [count, setCount] = useState(0);
  
    return (
      <div>
        <p>You clicked {count} times</p>
        <button onClick={() => setCount(count + 1)}>
          Click me
        </button>
      </div>
    );
}
```

useState接收状态初始值，返回一个数组，第一个是状态的当前值，第二个是函数，用来更新状态 *(约定命名为set+状态变量名)*。

count是怎么做到更新的呢？  
在上例中count只是一个数字，就像下面这行代码一样
```js
const count = 42;
// ...
<p>You clicked {count} times</p>
```

组件第一渲染时，从useState拿到初始值0，调用setCount,组件重新渲染，拿到1。

```js
//初始值
function Counter() {
  const count = 0; 
  // ...
  <p>You clicked {count} times</p>
  // ...
}

// 点击一次
function Counter() {
  const count = 1; 
  // ...
  <p>You clicked {count} times</p>
  // ...
}

// 点击两次
function Counter() {
  const count = 2; 
  // ...
  <p>You clicked {count} times</p>
  // ...
}
```

**更新状态，React会重新渲染组件，每一次都拿到独立的count状态，但是这个状态在一次渲染过程中是常量**

```js
function Counter() {
  const [count, setCount] = useState(0);

  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
      <button onClick={handleAlertClick}>
        Show alert
      </button>
    </div>
  );
}
```

注意：react根据顺序来保存和使用state
```js
  //第一次渲染
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);

  //第二次渲染
  useState(42);  //读取状态变量age的值（这时候传的参数42直接被忽略）
  useState('banana');  //读取状态变量fruit的值（这时候传的参数banana直接被忽略）
  useState([{ text: 'Learn Hooks' }]); //...

```
若将代码改为
```js
  let showFruit = true;
  const [age, setAge] = useState(42);
  
  if(showFruit) {
    const [fruit, setFruit] = useState('banana');
    showFruit = false;
  }
 
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);

  //第二次渲染
  useState(42);  //读取状态变量age的值（这时候传的参数42直接被忽略）
  // useState('banana');  
  useState([{ text: 'Learn Hooks' }]); //读取到的却是状态变量fruit的值，导致报错
```

## useContext(共享状态钩子)
有两个组件，我们希望共享他们的状态
```js
    <div className="App">
        <Navbar/>
        <Messages/>
    </div>
```
首先用到React Context，在组件外部建立一个Context（?）。

```js
    const AppContext = React.createContext({});
```
之后用AppContext将组件封装

```js
    <AppContext.Provider value={{
        username: 'superawesome'
    }}>
        <div className="App">
            <Navbar/>
            <Messages/>
        </div>
    </AppContext.Provider>
```
AppContext.Provider提供了一个Context对象，该对象可以被子组件共享

```js
    export default function Header() {
        const { username } = useContext(AppContext)

        return (
            <div>
                <p>my name is {username}</p>
            </div>
        );
    }
```

```js
    export default function Content() {
        const { username } = useContext(AppContext)

        return (
            <div>
                <p>Hello, {username}</p>
            </div>
        );
    }
```
因此，react规定我们必须把hooks写在函数的最外层，不能写在ifelse等条件语句当中，来确保hooks的执行顺序一致。

## useReducer(action钩子)
React本身不提供状态管理功能，通常需要使用外部库，最常用的是Redux

Redux核心概念：组件发出 action 与状态管理器通信。状态管理器收到 action 以后，使用 Reducer 函数算出新的状态。  

Reducer 函数的形式是：
```js
    (state, action) => newState
```

useReducer用来引入Reducer
```js
    const [state, dispatch] = useReducer(reducer, initialState);
```
下面是一个计数器

Reducer
```js
    const myReducer = (state, action) => {
        switch(action.type)  {
            case('countUp'):
            return  {
                ...state,
                count: state.count + 1
            }
            default:
            return  state;
        }
    }
```

组件
```js
    export default () => {
        const [state, dispatch] = useReducer(myReducer, { count: 0 });
        return (
            <div>
            <button onClick={() => dispatch({ type: "countUp" })}>+1</button>
            <p>Count: {state.count}</p>
            </div>
        );
    };
```

## useEffect(副作用钩子)
可将useEffect视为componentDidMount，componentDidUpdate 和 componentWillUnmount 的组合。

```js
    useEffect(() => {
      setLoading(true);
      fetch(`https://cnodejs.org/api/v1/topics?page=${pageId}`)
        .then(response => response.json())
        .then(data => {
          setTitle(data.data[0].title);
          setLoading(false);
        });
    }, [pageId]);
```

useEffect接收两个参数。第一个是函数，放所需执行的代码，放在componentDidMount里面的代码，可以直接放在useEffect中，第二个参数是一个数组，里面是Effect的依赖项，数组发生变化，useEffect就会执行。第二个参数可以省略，每次渲染就会执行useEffect。  


*effect是如何读取到最新的count值，并且执行的？*  


我们已经知道count是某个特定渲染中的常量。事件处理函数“看到”的是属于它那次特定渲染中的count状态值。对于effects也同样如此，**并不是count的值在“不变”的effect中发生了改变，而是effect 函数本身在每一次渲染中都不相同。**

```js
  // 第一次渲染
   useEffect(() => {
    fetch(`https://cnodejs.org/api/v1/topics?page=${1}`)
  });

  //第二次渲染
   useEffect(() => {
    fetch(`https://cnodejs.org/api/v1/topics?page=${2}`)
  });

  //...
```
React会记住你提供的effect函数，在每次DOM更改后调用它，并且，effect函数“看到”，都是它那次的特定的值

*关于依赖*

**effect中用到的所有组件内的值都要包含在依赖中**，如果设置了错误的依赖项，会怎么样呢？

比如,将一个类组件的定时器改写成useEffect
```js
  class Counter extends React.Component {
  state = {
    count: 0,
  };
  componentDidMount() {
    this.interval = setInterval(this.tick, 1000);
  }
  componentWillUnmount() {
    clearInterval(this.interval);
  }
  tick = () => {
    this.setState({
      count: this.state.count + 1
    });
  }
  render() {
    return <h1>{this.state.count}</h1>;
  }
}
```

我们可能会想，我只想运行一次effect，开启一次定时器，清除一次
```js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);

  return <h1>{count}</h1>;
}
```
它只递增了一次，为什么呢？是因为定时器函数被清除了吗？

React只会在浏览器绘制后运行effect。这使得你的应用更流畅，因为大部分effects不会阻塞屏幕更新，Effect的清除同样被延迟了。**上一次的effect会在重新渲染后被清除掉**

因为依赖是我们告诉effect需要重新执行的依据，第一次渲染中
```js
setCount(count + 1);
```
等价于
```js
setCount(0 + 1);
```
而我们的依赖为[],effect不会重新执行，所以之后每一次其实都在调用
```js
setCount(0 + 1);
```

### 两种解决办法
- 在依赖中包含所有effect中用到的组件内的值
  ```js
  useEffect(() => {
  const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, [count]);
  ```
  缺点：
    1. 依赖项过多
    2. 定时器会在每一次count改变后被清除

- 修改effect内部确保包含的值只在需要时发生变更（减少依赖）
  
  ```js
    useEffect(() => {
    const id = setInterval(() => {
        setCount(count + 1);
    }, 1000);
      return () => clearInterval(id);
    }, [count]);

    => 

    useEffect(() => {
    const id = setInterval(() => {
        setCount(c => c + 1);
    }, 1000);
      return () => clearInterval(id);
    }, [count]);
  ```
  React知道当前状态值，我们只需要告诉react去递增，不需要告诉他具体的值

## 自定义

我们还可以将hooks代码封装起来，变成自定义的hooks，方便共享

```js
const useTitle = (pageId) => {
    const [loading, setLoading] = useState(true);
    const [title, setTitle] = useState('');
  
    useEffect(() => {
      setLoading(true);
      fetch(`https://cnodejs.org/api/v1/topics?page=${pageId}`)
        .then(response => response.json())
        .then(data => {
          setTitle(data.data[0].title);
          setLoading(false);
        });
    }, [pageId]);
    return [loading, title]
}
```

结束。。。