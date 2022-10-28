---
title: chrome高级技巧
date: 2022-06-28
author: 7
---
### 1. 一键重新发起请求  
在与后端接口联调或排查线上BUG时，你是不是也经常听到他们说这句话：你再发起一次请求试试，我这边看下为啥出错了！  
重发请求，这有一种简单到发指的方式。

选中Network  
点击Fetch/XHR  
选择要重新发送的请求  
右键选择Replay XHR  
![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e056245b2a9e4e6dbfb39db6903f9275~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

不用刷新页面，不用走页面交互。  
### 2. 在控制台快速发起请求
还是联调或修BUG的场景，针对同样的请求，有时候需要修改入参重新发起，有啥快捷方式？

选中Network  
点击Fetch/XHR  
选择Copy as fetch  
控制台粘贴代码  
修改参数，回车搞定  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f91af146bbee42cc9e99badf83de83a8~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)
### 3. 复制JavaScript变量
假如你的代码经过计算会输出一个复杂的对象，且需要被复制下来发送给其他人，怎么办？

使用copy函数，将对象作为入参执行即可

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8eac65d357c04a779149719621f477c0~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)
### 4. 控制台获取Elements面板选中的元素
调试网页时通过Elements面板选中元素后，如果想通过JS知道它的一些属性，如宽、高、位置等怎么办呢？

通过Elements选择要调试的元素  
控制台直接用$0访问

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fd8a970c19842a7b73ee5d43f64efa6~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 5. 截取一张全屏的网页
偶尔咱们也会有对网页截屏的需求，一屏还好，系统自带的截屏或者微信截图等都可以办到，但是要求将超出一屏的内容也截下来咋办呢？

准备好需要截屏的内容  
cmd + shift + p 执行Command命令  
输入Capture full size screenshot 按下回车  
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44643079db90418d8d359d4278605732~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

如果要截取选中的部分元素呢？  
答案也很简单，第三步输入Capture node screenshot即可
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3835874255224edbbd98977a1727ca7e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 6. 一键展开所有DOM元素
调试元素时，在层级比较深的情况下，你是不是也经常一个个展开去调试？有一种更加快捷的方式

按住opt键 + click（需要展开的最外层元素）
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c09ba071e1e34b9387ee0071905ad21a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)


### 7. 控制台引用上一次执行的结果
来看看这个场景，我猜你也一定遇到过, 对某个字符串进行了各种工序，然后我们想知道每一步执行的结果，该咋办？。
```js
'fatfish'.split('').reverse().join('') // hsiftaf

你可能会这样做
// 第1步
'fatfish'.split('') // ['f', 'a', 't', 'f', 'i', 's', 'h']
// 第2步
['f', 'a', 't', 'f', 'i', 's', 'h'].reverse() // ['h', 's', 'i', 'f', 't', 'a', 'f']
// 第3步
['h', 's', 'i', 'f', 't', 'a', 'f'].join('') // hsiftaf
```


更简单的方式
使用$_引用上一次操作的结果，不用每次都复制一遍
```js
// 第1步
'fatfish'.split('') // ['f', 'a', 't', 'f', 'i', 's', 'h']
// 第2步
$_.reverse() // ['h', 's', 'i', 'f', 't', 'a', 'f']
// 第3步
$_.join('') // hsiftaf
```


### 8. "$"和"$$"选择器
在控制台使用document.querySelector和document.querySelectorAll选择当前页面的元素是最常见的需求了，不过着实有点太长了，咱们可以使用$和$$替代。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b99835a401524edeae0eebe599042df7~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 9. $i直接在控制台安装npm包
你遇到过这个场景吗？有时候想使用比如dayjs或者lodash的某个API，但是又不想去官网查，如果可以在控制台直接试出来就好了。
Console Importer 就是这么一个插件，用来在控制台直接安装npm包。

安装Console Importer插件  
$i('name')安装npm包

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/980db6a2b2d74115bdab37e5b061a7a1~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)


### 10. Add conditional breakpoint条件断点的妙用
假设有下面这段代码，咱们希望食物名字是🍫时才触发断点，可以怎么弄？
const foods = [
  {
    name: '🍔',
    price: 10
  },
  {
    name: '🍫',
    price: 15
  },
  {
    name: '🍵',
    price: 20
  },
]

foods.forEach((v) => {
  console.log(v.name, v.price)
})
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43a4882a64374e3eb3e492380c248ae6~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

这在大量数据下，只想对符合条件时打断点条件将会非常方便。

