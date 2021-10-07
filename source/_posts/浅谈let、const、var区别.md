---
title: 浅谈let、const、var区别
date: 2021-07-31

author: YYY
---

## 浅谈let、const、var区别

**var**

在 ES6 之前我们都是通过 var 关键字定义 JavaScript 变量。ES6 才新增了 let 和 const 关键字


在全局作用域下使用 var 声明一个变量，默认它是挂载在顶层对象 window 对象下（Node 是 global）

```
var num = 1
console.log(window.num) // 1
```

用 var 声明的变量的作用域是它当前的执行上下文，可以是函数也可以是全局
```
var x = 1 // 声明在全局作用域下
function foo() {
    var x = 2 // 声明在 foo 函数作用域下
    console.log(x) // 2
}
foo()
console.log(x) // 1
```

如果在 foo 没有声明 x ，而是赋值，则赋值的是 foo 外层作用域下的 x
```
var x = 1 // 声明在全局作用域下
function foo() {
    x = 2 // 赋值
    console.log(x) // 2
}
foo()
console.log(x) // 2
```

如果赋值给未声明的变量，该变量会被隐式地创建为全局变量（它将成为顶层对象的属性）
```
a = 2
console.log(window.a) // 2

function foo(){
    b = 3
}
foo()
console.log(window.b) // 3
```

var 缺陷一：所有未声明直接赋值的变量都会自动挂在顶层对象下，造成全局环境变量不可控、混乱

**变量提升（hoisted）**
使用var声明的变量存在变量提升的情况

```
console.log(b) // undefined
var b = 3
```

注意，提升仅仅是变量声明，不会影响其值的初始化，可以与隐式的理解为：

```
var b
console.log(b) // undefined
b = 3
```

作用域规则
var 声明可以在包含它的函数，模块，命名空间或全局作用域内部任何位置被访问，包含它的代码块对此没有什么影响，所以多次声明同一个变量并不会报错：

```
var x = 1
var x = 2
```

这种作用域规则可能会引发一些错误

```
function sumArr(arrList) {
    var sum = 0;
    for (var i = 0; i < arrList.length; i++) {
        var arr = arrList[i];
        for (var i = 0; i < arr.length; i++) {
            sum += arr[i];
        }
    }

    return sum;
}
```

这里很容易看出一些问题，里层的 for 循环会覆盖变量 i，因为所有 i 都引用相同的函数作用域内的变量。有经验的开发者们很清楚，这些问题可能在代码审查时漏掉，引发无穷的麻烦。

var 缺陷二：允许多次声明同一变量而不报错，造成代码不容易维护

捕获变量怪异之处
```
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10
```

i 是全局变量，全局只有一个变量i ， for 循环结束时， i=10 ，所以 a[6]() 也为 10 ，并且 a 的所有元素里面的 i 都为 10
而我们期望的是 a[6]() 输出 6，所以我们有了下面的块级作用域

**let**

let 与 var 的写法一致，不同的是它使用的是块作用域
块作用域变量在包含它们的块或 for 循环之外是不能访问的

```
{
    let x = 1
}
console.log(x) // Uncaught ReferenceError: x is not defined
```

所以：

```
var a = [];
for (let i = 0; i < 10; i++) { // 每一次循环的 i 其实都是一个新的变量
  a[i] = function () {
    console.log(i);
  };
} // JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量i时，就在上一轮循环的基础上进行计算
a[6](); // 6
```

同时， let 解决了 var 的两个缺陷：
使用 let 在全局作用域下声明的变量也不是顶层对象的属性

```
let b = 2
window.b // undefined
```

那它在哪里喃？

在全局作用域中，用 let 和 const 声明的全局变量没有在全局对象中，只是一个块级作用域（Script）中
不允许同一块中重复声明

```
let x = 1
let x = 2
// Uncaught SyntaxError: Identifier 'x' has already been declared
```
如果在不同块中是可以声明的

```
{
    let x = 1
    {
        let x = 2
    }
}
```

这种在一个嵌套作用域中声明同一个变量名称的行为称做 屏蔽 ，它可以完美解决上面的 sumArr 问题：

```
function sumArr(arrList) {
    let sum = 0;
    for (let i = 0; i < arrList.length; i++) {
        var arr = arrList[i];
        for (let i = 0; i < arr.length; i++) {
            sum += arr[i];
        }
    }

    return sum;
}
```

此时将得到正确的结果，因为内层循环的 i 可以屏蔽掉外层循环的 i
通常来讲应该避免使用屏蔽，因为我们需要写出清晰的代码。同时也有些场景适合利用它，你需要好好打算一下

**暂时性死区（TDZ）**
指 let 声明的变量在被声明之前不能被访问

```
console.log(x) // Uncaught ReferenceError: x is not defined
let x = 1
```

如果你在块中声明 let ，它会报以下错误：
```
// let
{
    console.log(x) // Uncaught ReferenceError: Cannot access 'x' before initialization
    let x = 2
}
```

**const**

const 声明一个只读的常量。一旦声明，常量的值就不能改变。


```
const a = 1

a = 2 // Uncaught TypeError: Assignment to constant variable.
```
因此， const声明的变量不得改变值，这意味着，const一旦声明变量，就必须立即初始化，不能留到以后赋值。
```
const s // 声明未赋值
// Uncaught SyntaxError: Missing initializer in const declaration
```
注意，这里 const 保证的不是变量的值不得改动，而是变量指向的那个内存地址不得改动，如果是基本类型的话，变量的值就保存在那个内存地址上，也就是常亮，如果是引用类型，它内部的值是可以变更的

```
const num = 1
const user = {
    name: "sisterAn",
    age: num,
}

user = {
    name: "pingzi",
    age: num
} // Uncaught TypeError: Assignment to constant variable.

// 下面这些都是运行成功的
user.name = "Hello"
user.name = "Kitty"
user.name = "Cat"
user.age--
```
其它 const 与 let 相同，例如：

- 作用域相同，只在声明所在的块级作用域内有效
- 常量也是不提升，同样存在暂时性死区
- 这里不再赘述

**var vs let vs const**

1. var 、 let 、 const 的不同主要有以下几个方面：

- 作用域规则
- 重复声明/重复赋值
- 变量提升（hoisted）
- 暂时死区（TDZ）
- 作用域规则
- let/const 声明的变量属于块作用域，只能在其块或子块中可用。而 var 声明的变量的作用域是是全局或者整个封闭函数

2. 重复声明/重复赋值

- var 可以重复声明和重复赋值
- let 仅允许重复赋值，但不能重复声明
- const 既不可以重复赋值，但不能重复声明

3. 变量提升（hoisted）

- var 声明的变量存在变量提升，即可以在变量声明前访问变量，值为undefined
- let 和 const 不存在变量提升，即它们所声明的变量一定要在声明后使用，否则报错 ReferenceError

var：
```
console.log(a) // undefined
var a = 1
```

let：
```
console.log(b) // Uncaught ReferenceError: b is not defined
let b = 2
```

const：
```
console.log(c) // Uncaught ReferenceError: c is not defined
let c = 3
```

3. 暂时死区（TDZ）
var不存在暂时性死区， let和const存在暂时性死区，只有变量声明后，才能被访问或使用

4. 编程风格
ES6 提出了两个新的声明变量的命令：let 和 const 。其中，let 完全可以取代 var ，因为两者语义相同，而且 let 没有副作用。所以，我们在开发中建议使用 let 、 const ，不使用 var

