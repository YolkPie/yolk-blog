---
title: js代码质量-健壮性
date: 2022-04-26 15:26:47
tags:
- git

author: YYY
---
对于初入职场的我们，最重要的就是学习如何写出高质量的 js 代码。学习的途径也很简单，多看，多写，多总结：

多看：看大佬们写的代码，学习他们智慧的结晶
多写：纸上得来终觉浅，不停的敲代码才能发现自己的问题，提升自己
多总结：我一直以来奉行的学习方法有两种——问题驱动学习和输出倒闭输入。只有自己认真的思考总结过，才能发现很多问题的细节，真正的去掌握知识。

0 什么是代码健壮性？
在学校看论文时就经常遇到，在公司学习时经常听到其他人提到代码的健壮性，所以每每听到周会大佬们讨论时都会莫名觉厉。
健壮性，顾名思义，健康强壮。拿这个词来形容人，就是说明这个人身体健康强壮，在遇到小感冒，跌打损伤，不会让他卧床不起。拿这个词来形容代码，道理也一样，在代码遇到各种个样的异常问题，如读取一个值不是预期类型、查询不到指定的路径等，代码不会轻易的挂掉，而是有它自己的一套措施。

1 函数
一、函数默认参数妙用
场景：
假设我们又如下的初始化工作需要进行，在代码的最开始我们需要对config对象进行初始化工作
function initConfig(config) {
    config.map((item) => {
        item.content = Number(item.content)
    })
}
复制代码
如果我们不小心忘记给它传递参数，浏览器会报如下错误，提示我们 config 没有 map 方法，因为此时 config 为 undefined

解决办法：
我们可以给函数的参数加上一个默认的值
function initConfig(config = []) {
    config.map((item) => {
        item.content = Number(item.content)
    })
}
复制代码
总结：
这里我们可以默认设置其他的类型，如字符串、数字、对象等，提高代码的健壮性
2 解构赋值
**解构： **  ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构
一、数组
场景：
假设我们需要一个数组的值分别给几个变量，我们会使用数组的解构
function handleArr(arr) {
    let [a, b, c] = [1, 2, 3]
    return a + b + c
}
复制代码
当我们需要结构的数组不能确定值百分之百正确时，也就是没有多或者少时，我们任然按照上面的写法便会出现问题

解决办法：
给解构赋值指定默认值
function handleArr(arr) {
    let [a = 1, b = 2, c = 3] = [1, 2, 3]
    return a + b + c
}

const arr = [1, 2]

handleArr(arr)	// 6
复制代码
总结：
开发时常用到这种方法一次取出多个值，但是要考虑到解构的值不存在的情况，设置好默认值提高代码的健壮性
二、对象
场景：
对象的场景和数组的原理是一样，只不过对象解构的场景用到得更多。
解决办法：
给对象解构赋值指定默认值
function handleObj(obj) {
  const { name, list = [] } = obj
  if(name) {
    list.map((item) => {
      console.log(item)
    })
  }
}

const obj = {
  name: 'add1',
  list: [1, 2, 3]
}
复制代码
总结：
对象解构的内容可能是字符串、数字、数组或是对象等，对于不同的变量类型，我们需要指定不同的默认值，确保代码的健壮性
3 接口请求
前面两点讨论的是不要轻易相信前端自己的数据，那么从后端接口传来的数据就更不要轻易相信。
一、后端响应的数据
场景：
假设后后端规定接口响应的内容如下
{
  code: 0,
  msg: "",
  data: []
}
复制代码
我们前端代码可能会这样写
async function handleData() {
  const { code, msg, data }  = await fetchList()
  data.map((item) => {
    console.log(item)
  })
}
复制代码
如果后端接口出现了异常

后端接口挂掉，取出来的 data 为 undefined
后端返回的 data 出现异常，不是数组
....

那么代码中执行的 map 方法就会报错
解决办法：
为了防止接口不反回 data 字段，我们可以给解构指定默认值
async function handleData() {
  const { code, msg, data = [] }  = await fetchList()
  data.map((item) => {
    console.log(item)
  })
}
复制代码
为了防止接口返回的是数组，我们可以对类型进行检查
async function handleData() {
  const { code, msg, data = [] }  = await fetchList()
  Array.isArray(data) && data.map((item) => {
    console.log(item)
  })
}
复制代码
总结：
我们约定好的接口传过来的数据可能出现数据不存在，类型对不上等异常错误，在相应地方加上异常的处理提高代码的健壮性
二、异常捕获

除了接口数据的处理，我们还要做好异常的处理，这里参考一篇博客：JavaScript 中的异常处理

4 lodash
在进行业务开发时，经常会忽略一些代码健壮性考虑的地方。我们可以使用现有的工具库提升代码健壮性，同时也可以让开发者更关注于业务逻辑的开发。这里推荐的工具库是 lodash。

文档传送门：Lodash中文文档

一、取 object 值
我们在上面提到了使用解构的方法取对象的值，在这里我们可以使用_.get()方法，进而不用再关注可能出现的异常。
方法：
_.get(object, path, [defaultValue])
复制代码
根据 object对象的path路径获取值。 如果解析 value 是 undefined 会以 defaultValue 取代。
参数：

bject (Object): 要检索的对象。
path (Array|string): 要获取属性的路径。
[defaultValue] (*): 如果解析值是 undefined ，这值会被返回。

返回值：
返回解析的值
