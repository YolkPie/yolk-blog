---
title: JS中的纯函数
date: 2021-06-28 19:52:00
categories: JS
author: zjk537
cover: "http://m.360buyimg.com/img/jfs/t1/182815/14/11597/305566/60d9b9d1E24d45785/4fae9cdafc224cc3.jpg"
---
# 纯函数
晦涩的定义是这样的：一个函数的返回结果只依赖于它的参数，并且在执行过程里面没有副作用;

总共2点：
> 1. 结果只依赖参数
> 2. 执行过程中没有副作用

```js
// demo1
const a = 1
const foo = (b) => a + b
foo(2) // => 3


// demo2
const a = 1
const foo = (x, b) => x + b
foo(1, 2) // => 3


// demo3
const a = 1
const foo = (obj, b) => {
  return obj.x + b
}
const counter = { x: 1 }
foo(counter, 2) // => 3
counter.x // => 1


// demo4 
const a = 1
const foo = (obj, b) => {
  obj.x = 2
  return obj.x + b
}
const counter = { x: 1 }
foo(counter, 2) // => 4
counter.x // => 2

// demo5
const foo = (b) => {
  const obj = { x: 1 }
  obj.x = 2
  return obj.x + b
}

```


<font color="white">

分析
1、demo1: 不是。 因为函数结果还依赖了外部变量a
2、demo2: 是。
3、demo3: 是。 只依赖入参，不产生副作用
4、demo4: 不是。 产生副作用。修改了obj.x的值
5、demo5: 是。 对内部变量obj的修改，外部程序不可见，无副作用
</font>


- 副作用都包含哪些？
1、调用 DOM API 修改页面；
2、发送了 Ajax 请求；
3、调用 window.reload 刷新浏览器；
4、甚至是 console.log 往控制台打印数据；