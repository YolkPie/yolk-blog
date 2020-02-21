---
title: React 内置 Hook 简介
date: 2020-02-21 10:12:06
author: 于吉浒
tags:
  - react
  - react hooks
categories: react
keywords: react hooks
description: Hook 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。
cover: https://img14.360buyimg.com/imagetools/jfs/t1/97654/25/12741/24574/5e4f3d7aE2ca43b3f/814d5d47d6e9b205.png
top_img: https://img14.360buyimg.com/imagetools/jfs/t1/100299/11/12771/23094/5e4f3d36E62e7ad8d/9295713b1e6e9988.jpg
---
React 内置 Hook 如下：
- 基础 Hook
  - useState
  - useEffect
  - useContext
- 额外 Hook
  - useReducer
  - useCallback
  - useMemo
  - useRef
  - useImperativeHandle
  - useLayoutEffect
  - useDebugValue

## useState
```javascript
const [state, setState] = useState(initialState);
```
为函数组件添加一个内部 state，在组件重新渲染时保留 state 的值，返回一个数组，第一个值为 state 的值，第二个值为更新 state 值的函数。

## useEffect
```javascript
useEffect(didUpdate, [deps]);
```
在函数组件的主体内，操作 DOM 、订阅事件、设置定时器、记录日志以及包含其他副作用的操作都是不允许的。
该 Hook 接收一个包含副作用操作的函数和一个依赖项数组，在每次渲染到屏幕之后执行副作用函数。可以为副作用函数返回一个清除副作用的函数，在组件卸载之前清除副作用。
当依赖项数组是一个空数组时，副作用数组只会在组件首次渲染时执行，effect 内部的 props 和 state 会一直保持其初始值。

## useLayoutEffect
```javascript
useLayoutEffect(update, [deps]);
```
并非所有的副作用都可以被延迟执行，在浏览器执行下一次绘制之前，用户可见的 DOM 变更就需要被同步执行，这样用户才不会感觉到视觉上的不一致。
该 Hook，会在所有的 DOM 变更之后同步调用 effect，可以使用它来读取 DOM 布局并同步触发重渲染。在浏览器执行绘制之前，useLayoutEffect 内部的更新计划将被同步刷新。

## useContext
```javascript
const value = useContext(MyContext);
```
接收一个由 React.createContext 创建的 context 对象，返回该context 对象的当前值。当前的 context 值由上层组件中距离当前组件最近的 `<MyContext.Provider>` 的 value prop 决定。
该 Hook 能够让组件读取 context 值并订阅其变化，需要在上层组件树中使用 `<MyContext.Provider>` 来为下层组件提供 context。在 context 的值发生变化时，会触发组件的重新渲染。

使用示例：
```javascript
const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

const ThemeContext = React.createContext(themes.light);

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);

  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  );
}
```

## useReducer
```javascript
const [state, dispatch] = useReducer(reducer, initialArg, init);
```
useState 的替代方案。它接收一个形如 (state, action) => newState 的 reducer，并返回当前的 state 以及与其配套的 dispatch 方法。
当 state 的结构较为复杂且包含多个子值或者新的 state 依赖旧的 state 时，useReducer 会比 useState 更适用。

使用示例：
```javascript
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

## useCallback
```javascript
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```
接收一个回调函数和一个依赖项数组作为参数，返回该回调函数的 memorized 版本，该回调函数仅在某个依赖项发生改变后才会更新。把该回调函数作为属性传递给子组件，当父组件重新渲染时，可以避免子组件不必要的渲染。
useCallback(fn, deps) 相当于 useMemo(() => fn, deps)

## useMemo
```javascript
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```
接收一个创建状态的函数和一个依赖项数组作为参数，返回一个 memorized 值，只有在某个依赖项发生变化时，才重新计算 memorized 值。

## useRef
```javascript
const refContainer = useRef(initialValue);
```
返回一个可变的 ref 对象，其 .current 属性被初始化为传入的参数（initialValue）。返回的 ref 对象在组件的整个生命周期内保持不变。
本质上，useRef 就像是可以在其 .current 属性中保存一个可变值的“盒子”。 如果将 ref 对象以 `<div ref={myRef} />` 形式传入组件，则无论该节点如何改变，React 都会将 ref 对象的 .current 属性设置为相应的 DOM 节点。
而 useRef() 和自建一个 {current: ...} 对象的唯一区别是，useRef 会在每次渲染时返回同一个 ref 对象。

使用示例：
```javascript
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` 指向已挂载到 DOM 上的文本输入元素
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

## useImperativeHandle
```javascript
useImperativeHandle(ref, createHandle, [deps])
```
在使用 ref 时，自定义暴露给父组件的实例值。

useImperativeHandle 应当与 forwardRef(父组件的 ref) 一起使用：
```javascript
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);
```
渲染 `<FancyInput ref={inputRef} />` 的父组件可以调用 inputRef.current.focus()。

## useDebugValue
```javascript
useDebugValue(value);
```
用于在 React 开发者工具中显示自定义 hook 的标签。

参考：
[React Hooks 官方文档](https://zh-hans.reactjs.org/docs/hooks-intro.html)