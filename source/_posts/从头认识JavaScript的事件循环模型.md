---
title: 从头认识JavaScript的事件循环模型
date: 2022-03-29 10:24:27
tags:
---
## 1. Js的运行机制

### 介绍

众所周知JavaScript是一门单线程的语言，所以在JavaScript的世界中默认的情况下同一个时间节点只能做一件事情，这就造成了JavaScript这门语言的一些局限性，比如在我们的页面加载一些远程数据时，如果按照单线程同步的方式运行，一旦有HTTP请求向服务器发送，就会出现等待数据返回之前页面假死的现象。因为JavaScript在同一时间只能做一件事，这就导致了页面渲染和事件的执行无法同进进行。

### 关于同步和异步

关于上述的情况，我们知道在JavaScript的世界中应该存在一种解决方案，来处理单线程造成的诟病。这就是同步【阻塞】和异步【非阻塞】执行模式的出现。

##### 同步（阻塞）

同步的意思是JavaScript会严格按照单线程（从上到下，从左到右的方式）执行代码逻辑，进行代码的解释和运行，所以在运行代码时，不会出现先运行4、5行的代码，再回头支； 行1、3行的代码的这种情况。比如：

```javascript
var a = 1
var b = 2
var c = a + b

console.log(c) // c的返回一定是先执行了上面几行代码之后才会执行
```

升级场景

```javascript
var a = 1
var b = 2
var d1 = new Date().getTime()
var d2 = new Date().getTime()

while(d2-d1 < 2000){
  d2 = new Date().getTime()
}

// 这段代码在输出结果之前网页出进入一个类似假死的状态
console.log(a+b)
```

##### 异步（非阻塞）

如上例子，我们明白了单线程同步模型中的问题，接下来引入异步模型介绍。

```javascript
var a = 1
var b = 2
setTimeout(function(){
  console.log('打印一段内容'）
}, 2000)

// 这段代码会先输出3，过2秒左右再输出function的内部打印
console.log(a+b)
```

这段代码的setTimeout定时任务规定了2秒之后执行一些内容，在运行当前程序执行到setTimeout时，并不会直接执行内部的回调函数，而是会先将内部的函数在另外一个位置保存起来，然扣继续执行下面的console.log进行输出，输出之后代码执行完成，然后等待2秒后执行之前保存起来了函数。



##### 总结

JavaScript的运行顺序就完全单线程的异步模型：同步在前，异步在后。所有的异步任务都要等待当前的同步任务执行完毕之后才能执行。请看下面案例：

```javascript
var a = 1
var b = 2
var d1 = new Date().getTime()
var d2 = new Date().getTime()
setTimeout(function(){
  console.log('我是异步任务'）
}, 1000)
while(d2-d1 < 2000){
  d2 = new Date().getTime()
}
// 这段代码在输出3之前会进入假死状态，'我是异步任务'一定会在3之后输出 
console.log(a+b)
```



#### JS的线程组成

上面几个例子我们大概了解了Js的运行顺序，那么为是这个顺序呢？这个顺序的原理是什么样的？我们应该如保更好更深的探究真相？带着这些问题，我们一起来了解一下浏览器的一个页面的实际线程组成。

在了解线程组成前要了解一点，虽然浏览器是单线程执行JavaScript代dcbg的，但是浏览器实际是以多个线程协助操作来实现单线程异步模型的，具体线程组成如下：

1. GUI渲染线程

2. JavaScript引擎线程

3. 事件触发线程

4. 定时器触发线程

5. Http请求线程

6. 其他线程


按照真实浏览器线程组成分析，我们会发现实际上运行JavaScript的线程其实并不是一个，但是为什么说JavaScript是一门单线程的语言呢？因为这些线程实际参与代码执行的线程并不是所有的线程，如比GUI渲染线程为什么单独存在，这个是防止我们在html网页渲染一半的时间突然执行了一段阻塞式的JS代码而导致网页卡在一半停住的这种效果。**在JavaScritp代码运行的过程中实际执行程序时同时只存在一个活动线程，这里实现同步异步就是靠多个线程切换的形式来进行实现的**。

所以，我们通常分析时，将上面的线分线程归纳为下面两条线程：

1. 【主线程】：这个线程用来执行页面的渲染，JS代码运行，事件触发等

2. 【工作线程】：这个线程是幕后工作的，用来处理异步任务的执行



## 2. JavaScript的运行模型

<img src="/Users/lizhengai/Documents/工作总结/学习总结/javaScript图片/图1.jpg" alt="图1" style="zoom:50%;" />

##### 从代码片段开发分析

```javascript
function task1(){
  console.log('第一个任务')
}
function task1(){
  console.log('第二个任务')
}
function task1(){
  console.log('第三个任务')
}
function task1(){
  console.log('第四个任务')
}

task1()
setTimeout(task2, 1000)
setTimeout(task3, 500)
task4()
```

我们创建了四个函数代表4个任务，函数本身都是同步代码。在执行的时候会按照1，2，3，4进行解析，解析过程中我们发现任务2和3被setTimout进行了定时托管，这样就只能先运行任务1和4了。当任务1和4运行完毕后500毫秒运行3，1000毫秒运行2

<img src="/Users/lizhengai/Documents/工作总结/学习总结/javaScript图片/图2.jpg" alt="图2" style="zoom:50%;" />

当同步任务都执行完成时，执行异步任务

<img src="/Users/lizhengai/Documents/工作总结/学习总结/javaScript图片/图3.jpg" alt="图3" style="zoom:50%;" />



##### 总结

通过图解我们就更清晰的能搞懂异步任务的执行方式了。



##### 关于执行栈

执行栈是一个栈的数据结构，当我们运行单层函数时，执行栈执行的函数进栈后，会出栈销毁然后下一个进栈出栈。如下例子

```javascript
function task1(){
  console.log('tak1执行')
  task2()
  console.log('task2执行完毕')
}
function task2(){
  console.log('task2执行')
  task3()
  console.log('task3执行完毕')
}
function task2(){
  console.log('task3执行')
}
console.log('task1执行完毕')


// 打印
/*
task1执行
task2执行
task3执行
task3执行完毕
task2执行完毕
task1执行完毕
*/
```

<img src="/Users/lizhengai/Documents/工作总结/学习总结/javaScript图片/图4.jpg" alt="图4" style="zoom:50%;" />

<img src="/Users/lizhengai/Documents/工作总结/学习总结/javaScript图片/图5.jpg" alt="图5" style="zoom:50%;" />

<img src="/Users/lizhengai/Documents/工作总结/学习总结/javaScript图片/图6.jpg" alt="图6" style="zoom:50%;" />

<img src="/Users/lizhengai/Documents/工作总结/学习总结/javaScript图片/图7.jpg" alt="图7" style="zoom:50%;" />