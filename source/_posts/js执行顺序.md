---
title: js执行顺序
date: 2022-04-26 15:26:47
tags:
- git

author: 7
---
首先来看一道常见的面试题
```javascript
    async function async1() {
        console.log('async1 start');
        await async2();
        console.log('async1 end');
    }
    async function async2() {
        console.log('async2');
    }

    console.log('script start');

    setTimeout(function() {
        console.log('setTimeout');
    }, 0)

    async1();

    new Promise(function(resolve) {
        console.log('promise1');
        resolve();
    }).then(function() {
        console.log('promise2');
    });
    console.log('script end');



    答案
    /* 
    script start
    async1 start
    async2
    promise1
    script end
    async1 end
    promise2
    setTimeout
    */


```

1. js运行机制
1.1 单线程的JavaScript

js是单线程的，基于事件循环，非阻塞IO的。
特点： 处理I／O型的应用，不适合CPU运算密集型的应用。
说明： 事件循环中使用一个事件队列，在每个时间点上，系统只会处理一个事件，即使电脑有多个CPU核心，也无法同时并行的处理多个事件。因此，node.js在I／O型的应用中，给每一个输入输出定义一个回调函数，node.js会自动将其加入到事件轮询的处理队列里，当I／O操作完成后，这个回调函数会被触发，系统会继续处理其他的请求。


然而单线程不应该是自上而下按照顺序执行的吗？
下面的代码输出顺序就被打乱了
```javascript
function fn(){
    console.log('start');
    setTimeout(()=>{
        console.log('setTimeout');
    },0);
    console.log('end');
}

fn() 

// 输出 start end setTimeout
```

1.2 JavaScript中的同步异步

js的同步异步是如何实现的?
js中包含诸多创建异步的函数如:
seTimeout，setInterval，dom事件，ajax，Promise，process.nextTick等函数


> 因为单线程，所以代码是自上而下执行，所有代码被放到`执行栈`中执行；
> 遇到异步函数将回调函数添加到一个`任务队列`里面（按顺序寄存起来）；
> 当`执行栈`中的代码执行完以后，会去循环`任务队列`里的函数（按顺序获取寄存的函数）;
> 将`任务队列`里的函数放到`执行栈`中执行;
> 如此往复，称为`事件循环`;

这样分析，上一段的代码就得到了合理的解释；
再来看一下这段代码；
```javascript
function fn() {
    setTimeout(()=>{
        console.log('a');
    },0);
    new Promise((resolve)=>{
        console.log('b');
        resolve();
    }).then(()=>{
        console.log('c')
    });
}
fn() 

答案
// b c a
```

解释
》执行栈中有fn() 函数，开始执行
》fn()执行中发现有异步函数 setTimeout(()=>{ console.log('a'); },0);将它寄存到任务队列；
》继续往下执行……，所以为什么看到a 最后输入；
那么问题来了，b和c 如何区分顺序呢？ 往下看

1.2.1 Promise和async中的立即执行
我们知道Promise中的异步体现在then和catch中，所以写在Promise中的代码是被当做同步任务 立即执行 的。
而在async/await中，在出现await出现之前，其中的代码也是 立即执行 的。
那么出现了await时候发生了什么呢？

1.2.2 await做了什么

从字面意思上看await就是等待，await 等待的是一个表达式，这个表达式的返回值可以是一个promise对象也可以是其他值。
很多人以为await会一直等待之后的表达式执行完之后才会继续执行后面的代码，实际上await是一个让出线程的标志。await后面的表达式会先执行一遍，将await后面的代码加入到microtask中，然后就会跳出整个async函数来执行后面的代码。
由于因为async await 本身就是promise+generator的语法糖。所以await后面的代码是microtask。所以对于开始面试题中的
```javascript
async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
}
```
等价于
```javascript
async function async1() {
    console.log('async1 start');
    Promise.resolve(async2()).then(() => {
                console.log('async1 end');
        })
}
```
例子解释
```javascript
async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
}
async function async2() {
    console.log('async2');
}
async1();
```

所以说 async里面，即await之前的代码是立即执行的；所以输出顺序
```javascript
//async1 start
//async2
//async1 end
```
1.3 宏任务和微任务

两任务在同步异步中处于什么地位？
两个任务分别处于任务队列中的宏队列与微队列中;
宏队列与微队列组成了任务队列;
任务队列将任务放入执行栈中执行

1.3.1 宏任务：
宏队列，macrotask，也叫tasks

宏队列，macrotask，也叫tasks。
异步任务的回调会依次进入macro task queue，等待后续被调用，
这些异步任务包括：


setTimeout
setInterval
setImmediate (Node独有)
requestAnimationFrame (浏览器独有)
I/O
UI rendering (浏览器独有)


常见的setTimeout一定要记住，其他如果能全部记住就最好

1.3.2 微任务：
微队列，microtask，也叫jobs

微队列，microtask，也叫jobs。
异步任务的回调会依次进入micro task queue，等待后续被调用，
这些异步任务包括：


process.nextTick (Node独有)
Promise
Object.observe
MutationObserver


常见的Promise一定要记住，其他如果能全部记住就最好

整个周期串起来



执行全局Script同步代码，这些同步代码有一些是同步语句，有一些是异步语句（比如setTimeout等）；
全局Script代码执行完毕后，执行栈Stack会清空；
从微队列中取出位于队首的回调任务，放入执行栈Stack中执行，执行完后微队列长度减1；
继续循环取出位于微队列的任务，放入执行栈Stack中执行，以此类推，直到直到把微任务执行完毕。注意，如果在执行微任务的过程中，又产生了微任务，那么会加入到微队列的末尾，也会在这个周期被调用执行；
微队列中的所有微任务都执行完毕，此时微队列为空队列，执行栈Stack也为空；
取出宏队列中的任务，放入执行栈Stack中执行；
执行完毕后，执行栈Stack为空；
重复第3-7个步骤；


以上才是一个完整的事件循环
这里我在转译一下通俗解释

> 执行栈里面（即主函数里面）先执行（此时相当于执行第一个宏函数）；按顺序临时存储宏队列和微队列；
> 当执行栈执行完成后，开始依次去除微队列执行（如果发现微队列里面又有微队列，那就从微队列栈最后插入）；
> 当微队列执行完成后，再依次执行宏队列；
> 然后重复循环

补充说明： 如果微任务里面又发现宏任务了怎么？ --依旧按顺序插入宏队列中；
举个例子：
```javascript
function fn(){
    console.log(1);
    
    setTimeout(() => {
        console.log(2);
    },0);
    
    new Promise((resolve, reject) => {
        console.log(3);
        resolve()
        setTimeout(() => {
          console.log(4);
        },0);
    }).then(() => {
        console.log(5);
    });
    
    setTimeout(() => {
        console.log(6);
    },0);
    
    console.log(7);
}
fn(); 

答案：
1
3
7
5
2
4
6
```

1.4 面试题解读
```javascript
async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
}
async function async2() {
    console.log('async2');
}

console.log('script start');

setTimeout(function() {
    console.log('setTimeout');
}, 0)

async1();

new Promise(function(resolve) {
    console.log('promise1');
    resolve();
}).then(function() {
    console.log('promise2');
});
console.log('script end');



答案
/* 
script start
async1 start
async2
promise1
script end
async1 end
promise2
setTimeout
*/
```


1.首先，事件循环从宏任务(macrotask)队列开始，这个时候，宏任务队列中，只有一个script(整体代码)任务；当遇到任务源(task source)时，则会先分发任务到对应的任务队列中去。


2.然后我们看到首先定义了两个async函数，接着往下看，然后遇到了 console 语句，直接输出 script start。输出之后，script 任务继续往下执行，遇到 setTimeout，其作为一个宏任务源，则会先将其任务分发到对应的队列中


3.script 任务继续往下执行，执行了async1()函数，前面讲过async函数中在await之前的代码是立即执行的，所以会立即输出async1 start。
遇到了await时，会将await后面的表达式执行一遍，所以就紧接着输出async2，然后将await后面的代码也就是console.log('async1 end')加入到microtask中的Promise队列中，接着跳出async1函数来执行后面的代码


4.script任务继续往下执行，遇到Promise实例。由于Promise中的函数是立即执行的，而后续的 .then 则会被分发到 microtask 的 Promise 队列中去。所以会先输出 promise1，然后执行 resolve，将 promise2 分配到对应队列


5.script任务继续往下执行，最后只有一句输出了 script end，至此，全局任务就执行完毕了。
根据上述，每次执行完一个宏任务之后，会去检查是否存在 Microtasks；如果有，则执行 Microtasks 直至清空 Microtask Queue。
因而在script任务执行完毕之后，开始查找清空微任务队列。此时，微任务中， Promise 队列有的两个任务async1 end和promise2，因此按先后顺序输出 async1 end，promise2。当所有的 Microtasks 执行完毕之后，表示第一轮的循环就结束了


6.第二轮循环依旧从宏任务队列开始。此时宏任务中只有一个 setTimeout，取出直接输出即可，至此整个流程结束

总结一下解题思路：

1.
> 首先，事件循环从宏任务(macrotask)队列开始，这个时候，宏任务队列中，只有一个script(整体代码)任务；
> 当任务遇到 async/await和Promise 分配到这个宏队列的微队列里面；
> 当遇到setTimeout 的时候分配到下一个宏队列里面

2.
> 执行了async1()函数，前面讲过async函数中在await之前的代码是立即执行的，所以会立即输出async1 start。
> 遇到了await时，会将await后面的表达式执行一遍，所以就紧接着输出async2，然后将await后面的代码也就是console.log('async1 end')加入到microtask中的Promise队列中，接着跳出async1函数来执行后面的代码

3.
> 遇到Promise实例。由于Promise中的函数是立即执行的，而后续的 .then 则会被分发到 microtask 的 Promise 队列中去。所以会先输出 promise1，然后执行 resolve，将 promise2 分配到对应队列

4.
> 任务继续往下执行，最后只有一句输出了 script end，至此，全局任务就执行完毕了。即第一个宏任务执行完了
> 根据上述，每次执行完一个宏任务之后，会去检查是否存在 Microtasks；如果有，则执行 Microtasks 直至清空 Microtask Queue。
> 因而在script任务执行完毕之后，开始查找清空微任务队列。此时，微任务中， Promise 队列有的两个任务async1 end和promise2，因此按先后顺序输出 async1 end，promise2。当所有的 Microtasks 执行完毕之后，表示第一轮的循环就结束了

5.
> 第二轮循环依旧从宏任务队列开始。此时宏任务中只有一个 setTimeout，取出直接输出即可，至此整个流程结束
