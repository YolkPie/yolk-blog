---
title: css行高line-height的一些理解
date: 2020-09-29 21:00:00
tags:
- line-height
categories: css
author: 崔梦林
keywords: line-height,css
description: line-height,css
cover: https://img13.360buyimg.com/imagetools/jfs/t1/135283/19/11127/8821/5f73dde9E13365429/19b59ab164c39f68.png
top_img: https://img11.360buyimg.com/imagetools/jfs/t1/138181/30/9495/263053/5f73de8bE1c29c613/fdc1af9b6e91eb24.png
---

## 前言

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在工作中遇到过设置行高不生效的问题，仔细研究后才知道自己对行高的理解远远不够，因此特地在这里总结一下。看一个比较简单的例子：

    ``` js
        <style>
            .parent {
                line-height: 40px;
            }
            .child {
                line-height: 20px;
                background-color: red;
            }
        </style>
        <div class="parent">
            <span class="child">我的行高是20px我的行高是20px我的行高是20px我的行高是20px我的行高是20px我的行高是20px我的行高是20px我的行高是20px</span>
            <p class="child">---我的行高是20px我的行高是20px我的行高是20px我的行高是20px我的行高是20px我的行高是20px我的行高是20px我的行高是20px---</p>
        </div>
    ```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在上面的一段代码中，父级div的line-height是40px，子级span和p的line-height为20px。按照我之前的理解，line-height就是设置字体行高的，设置什么是什么就是了，这种明确指定像素的情况，不就是20px嘛。好吧，看看结果：

![简单的例子的结果](https://img10.360buyimg.com/imagetools/jfs/t1/149728/21/9503/71634/5f707f4aE06b483ea/4c4763d52b7292da.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;yeah～，不是我认为的结果，p标签和span标签显示的行高完全是不同的。还是先熟悉下相关的概念吧。

## 要熟悉的概念

### 内联盒模型
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一说到盒模型，我们都会立马想到 margin、padding、content、border、box-sizing等，但这个只是 CSS 中盒模型的块级盒模型（block box）。我们今天要说的是另外一个被我们忽略掉的内联盒模型（inline box）。内联盒指的是盒子内部的构建模型，作用上关键是内容区域（conetnt）。

![内联盒模型](https://pic2.zhimg.com/v2-52951e8e4d1bebb7a3d060a7674e6f81_1440w.jpg?source=172ae18b)

- 内联盒子（inline box）分为内联盒子和匿名内联盒子。内联盒子不会让内容成块显示，而是排成一行。
- 行框盒子（line box）每一行就是一个行框盒子，由许多内联盒子（inline box）组成。
- 幽灵空白节点 （struct），每个行框盒子之前都有一个。

### 幽灵空白节点

> Each line box starts with a zero-width inline box with the element’s font and line height properties. We call that imaginary box a “strut”.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上面是规范中对幽灵空白节点的定义：每个行框盒子（line box）的前面都有一个0宽度的内联盒子（inline box），这个内联盒子有字体和行高的属性，被称做struct。因为这个内联盒子在页面上看不到，也无法通过脚本获取，但却真实存在，有点像文字节点一样（with the element’s font and line height properties）影响着页面的渲染，因此这个struct被我们成为“幽灵空白节点”。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;需要注意的是，幽灵空白节点的出现的前提是文档声明必须是 HTML5 文档声明（<!doctype html><html>），如果是 HTML5 之前的文档声明，幽灵空白节点是不存在的。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们先来看下幽灵空白节点吧。

    ``` js
        <style>
            .parent {
                background-color: lightgrey;
            }
            .child1 {
                display: inline-block;
            }
        </style>
        <div class="parent">
            <span class="child1"></span>
        </div>
        <div class="parent">
            <p class="child2"></p>
        </div>
    ```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;结果见下图。可以看到，父级div没有设置高度，span标签里没有文字，但最终的渲染结果是父级是有高度的，这个高度就是幽灵空白节点造成的。

![幽灵空白节点](https://img13.360buyimg.com/imagetools/jfs/t1/135899/16/10998/10438/5f715271E6ba5f83f/54a53b6f62b3ead9.png)

眼尖的你可能会提出质疑：既然每个行框盒子之前都有一个空白幽灵节点，为什么要设置span标签为inline-blcok。嗯，这里再补充一段规范。
> Line boxes are created as needed to hold inline-level content within an inline formatting context. 
<strong>
Line boxes that contain no text, no preserved white space, no inline elements with non-zero margins, padding, or borders, and no other in-flow content (such as images, inline blocks or inline tables), and do not end with a preserved newline must be treated as zero-height line boxes.
</strong>
for the purposes of determining the positions of any elements inside of them, and must be treated as not existing for any other purpose.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;意思就是说：如果一个line box里没有文字、空格、非0的margin或padding或border的inline元素、或其他in-flow内容（比如图片、inline-block 或 inline-table元素），且不以保留的换行符结束的话，就会被视作高度为0的line box。换言之，如果只有一个空的span标签，是不会有空白幽灵节点出现的。

## 结论与理解

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;先抛出网上大佬们给出的结论：
- 行内元素的line-height属性是去设置该元素所在行框盒子（line box）的行高，行框盒子取其内部所有内联盒子的行高的最大值，定为当前行的行高
- 换行后生成新的行框盒子，新生成的行的行高，重新在当前行包含的内联盒子的行高中取最大值

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;按照上面的结论，我们先画出内联盒模型：

![内联盒模型](https://img10.360buyimg.com/imagetools/jfs/t1/136071/12/10921/81186/5f713c1dEfcf1719b/d2f96148dc90fa81.png)

- 先解释span标签包裹的文字为什么行高是40px。由上文我们可以知道，幽灵空白节点和文本节点一样，有行高的属性，因为自身没有设置line-height， 因此继承了父级 div 的行高 40px。由span标签组成的inline box 的高度则由自身的行高决定，为20px。 这样，第一行的行框盒子line box的行高取struct和inline box的最大值，就是40px。如果要使span标签上的line-height 生效，那么按照最大值的原则，必须要给span大于40的行高才行。由于换行产生了新的行框盒子（line box），第二行和第三行的内联盒模型和第一行是一致的，行高同样为40px。
- 至于p标签包裹的文字为什么行高是20px，就很好理解了，因为没有幽灵空白节点呗。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们再看一个稍微复杂那么一点点的情况：

    ``` js
        <style>
            .parent {
            line-height: 0;
        }
        .child1 {
            line-height: 20px;
            background-color: red;
        }
        .child2 {
            line-height: 40px;
            background-color: green;
        }
        .child3 {
            line-height: 80px;
            background-color: blue;
        }
        </style>
        <p class="parent">
            <span class="child1">我的行高是20px我的行高是20px我的行高是20px我的行高是20px我的行高是20px我的行高是20px</span>   
            <em class="child2">我的行高是40px我的行高是40px我的行高是40px我的行高是40px我的行高是40px我的行高是40px</em>   
            <b class="child3">我的行高是80px我的行高是80px我的行高是80px我的行高是80px我的行高是80px我的行高是80px</b>
        </p>
    ```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;结果见下图，如果你能清楚的解释原因，就说明我讲明白了。

![练习](https://img14.360buyimg.com/imagetools/jfs/t1/131124/36/10700/81711/5f71db32E940ae3f0/e5b1ba96639168e2.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（完结）
