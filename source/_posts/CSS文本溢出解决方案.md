---
title: CSS文本溢出解决方案
date: 2021-09-14 13:55:25
tags:
  - CSS
categories: CSS
keywords: 文本溢出 break-word word-break word-wrap
description: 关于文本的换行可能大家都不是很在意，但有时候这个问题却能导致前端的整个样式乱掉；css控制文字换行的属性有好几个，它们有什么区别呢。
cover: https://img10.360buyimg.com/imagetools/jfs/t1/205648/34/7635/44563/614afc3fEb5ed47f8/7bd73005994f755a.jpg
top_img: https://img10.360buyimg.com/imagetools/jfs/t1/205648/34/7635/44563/614afc3fEb5ed47f8/7bd73005994f755a.jpg
---

### 前言
CSS的学习，就我个人看来，是有别于JavaScript这张传统程序语言的学习的。本身属性就多，值也多，不同属性在一起表现也不一样，不同属性和不同类型的HTML标签在一起又不一样，再加上兼容性差异和未定义行为。就像是很多个不确定因素，有着无穷多的组合和可能性。掌握这些不确定性，看书是绝对不够的，一定是要多多实践，多多思考，多多积累。对于底层机理的理解，也是需要一定的天赋的。

因此，就是自己很多年一直与CSS密切打交道，学习它，也有很多不知道的，理解不透彻，或者说因为要学习和思考的东西太多，还来不及估计到一些属性或者声明。

比方说本文要介绍的word-break:break-all和word-wrap:break-word, 虽然都有使用，都照过面，实际上，却一直没有机会能够好好看看这两个到底有什么区别，各个浏览器的兼容性如何，等等。换句话说，就是深入理解。


### 背景
做移动端项目时，在详情页富文本传过来一段很长的空格；页面展示的时候由于空格没有及时换行；导致空格后文字超出屏幕；页面展示缺少文字。当然，这个情况有兼容性，就是大部分的安卓手机；同样的在PC端（小屏幕）展示同样有问题；我的14.7寸的展示就有问题；





### word-break属性

- 属性值：

normal;
使用默认的换行规则。(即单词中间不换行，会提前折行)；（会溢出）
![示例](1-nor.jpg)

break-all;
允许任意非CJK(Chinese/Japanese/Korean)文本间的单词断行。
一个长长的单词不会新起一行展示，而是直接在本行剩余空间展示，展示不全时折断展示；（万恶原则）
兼容性：所有浏览器都支持；
![示例](1-all.jpg)

keep-all;
不允许CJK(Chinese/Japanese/Korean)文本中的单词换行，只能在半角空格或连字符处换行。非CJK文本的行为实际上和normal一致。（会溢出）
![示例](3-keep.jpg)

break-word;
一个长长的单词会新起一行展示，新的一行展示不全时折断展示；（新人榨干原则）
![示例](4-word.jpg)

- 万恶原则：不会新起一行，而是在本行剩余空间继续展示，不够则打断折行。能利用每一行的剩余空间，不浪费一点点空间；我这里将这样的‘不留余地’称作万恶的压榨原则；

- 新人榨干原则：本行不够展示的时候，会另起新的一行展示，如果在新的一行上仍然不够的话，会打断折行展示；这种新起一行，不利用上一行的剩余空间原则；我称之为新人榨干原则；

- 兼容性：
![示例](1-jian.jpg)





### word-wrap 属性

normal;
正常的换行规则（长单词新起一行且不折断）（会溢出）

bredak-word;
一个长长的单词会新起一行展示，且在新的一行展示不全时折断展示；（新人榨干原则）


**word-wrap**属性其实也是很有故事的，之前由于和word-break长得太像，难免会让人记不住搞混淆，晕头转向，于是在CSS3规范里，把这个属性的名称给改了，叫做：**overflow-wrap** 这个新属性名称显然语义更准确，也更容易区别和记忆。
但是呢，也就Chrome/Safari等WebKit/Blink浏览器支持。
所以，虽然换了个好看好用的新名字，为了兼容使用，目前，还是乖乖使用word-wrap吧。兼容性见下表（黄绿色的表示不支持overflow-wrap新的标准属性的）：





### overflow-wrap属性 (word-wrap的别名)
1. 是用来说明当一个不能被分开的字符串太长而不能填充其包裹盒时，为防止其溢出，浏览器是否允许这样的单词中断换行。
2. 与word-break相比，overflow-wrap仅在无法实现-将整个单词放在自己的行而不溢出的情况下，才会产生中断。

- 属性值：
normal;
行只能在正常的单词断点处中断。（会溢出）。
![示例](f-nor.jpg)


anywhere;
允许任意非CJK(Chinese/Japanese/Korean)文本间的单词断行。（一个单词一行；长单词截断）
兼容性：所有浏览器都支持
![示例](f-any.jpg)

break-word;
表示如果行内没有多余的地方容纳该单词到结尾，则那些正常的不能被分割的单词会被强制分割换行（新人榨干原则）。
![示例](f-break.jpg)

- 兼容性：
![示例](space-jian.jpg)


### white-space属性
- 属性值：
white-space CSS 属性是用来设置如何处理元素中的空白；

normal
连续的空白符会被合并，换行符会被当作空白符来处理。换行在填充「行框盒子(line boxes)」时是必要。

nowrap
和 normal 一样，连续的空白符会被合并。但文本内的换行无效

pre
连续的空白符会被保留。在遇到换行符或者<br>元素时才会换行。

pre-wrap
连续的空白符会被保留。在遇到换行符或者<br>元素，或者需要为了填充「行框盒子(line boxes)」时才会换行。

pre-line
连续的空白符会被合并。在遇到换行符或者<br>元素，或者需要为了填充「行框盒子(line boxes)」时会换行。

**break-spaces**
与 pre-wrap的行为相同，除了：

- 任何保留的空白序列总是占用空间，包括在行尾。
- 每个保留的空格字符后都存在换行机会，包括空格字符之间。
- 这样保留的空间占用空间而不会挂起，从而影响盒子的固有尺寸（最小内容大小和最大内容大小）。

- 兼容性：
![示例](space-jian.jpg)


### line-break属性
- 属性值：
用来处理如何断开（break lines）带有标点符号的中文、日文或韩文（CJK）文本的行。

auto
使用默认的断行规则分解文本。

loose
使用尽可能松散（least restrictive）的断行规则分解文本。一般用于短行的情况，如报纸。

normal
使用最一般（common）的断行规则分解文本。

strict
使用最严格（stringent）的断行原则分解文本。

anywhere
在每个印刷字符单元（typographic character unit）的周围，都有一个自动换行（soft wrap）的机会，包括任何标点符号（punctuation character）或是保留的空白字符（preserved white spaces），或是单词之间。但忽略任何用于阻止换行的字符，即使是来自 GL、WJ 或 ZWJ 字符集的字符，或是由 word-break 属性强制的字符。不同的换行机会拥有相同的优先级。也不应用断字符（hyphenation，可能是 "-"）。

- 兼容性：
火狐浏览器不支持此属性；其余浏览器均支持



### 我遇到的棘手问题
 问题描述：
 在一段富文本中；有很多空格，导致这些空格联为一体，不能折行，直到遇见第一个字符才折行；导致文本溢出；
 复现条件：
 1. 部分安卓手机（oppo）
 2. 京东app（非极速版）
 两个条件缺一不可，因为尝试了在有问题的手机，自带和qq浏览器时，是没有问题的；同时极速版的京东app也是没有问题的；

 富文本结构如下:

 ```html
<p>
  <span>&nbsp;</span><span>&nbsp;</span><span>&nbsp;</span>...很多个空白span标签
  <span>&nbsp;</span>
  <span>
    <span>&nbsp;</span><span>我是要展示的文本</span>
  </span>
 </p>

 ```

 尝试属性：**white-space**；这个属性的合并空格在此处是不生效的；因为文本的空格是由多个span标签分割的；本身就不属于连续空格；所以合并空格不起作用；
 尝试属性：**line-break**;
 

  ```js
  let text = "<p style='text-align: right; line-height: 22px; text-indent: 24px; margin-top: 0px; margin-right: 21px; margin-bottom: 0px;'><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'>&nbsp;</span><span style='font-family:;'>&nbsp;</span><span style='font-family:宋体'><span style='font-family:;'>&nbsp;</span><span style='color: rgb(0, 0, 0); font-size: 21px;'>邵武市人民法院</span></span></p>";

  ```

  ### 可能的解决方案

   解决方案一：
   - 将富文本的父盒子元素设置为font-size：0；可以解决这个问题；但是存在风险，一旦文字未设置字体，则继承字体为0；文字就会不展示；所以弃用该方案；

  
   解决方案二：
   - 使用正则将空格的span标签去掉；
   尝试方案：
   word-break: break-all;
   white-space: pre-line;
   line-break: anywhere;
   ---不管用；


   最后：
   如果谁能帮忙解决此问题，欢迎留言；





