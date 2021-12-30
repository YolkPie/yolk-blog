---
title: CSS中的百分比
date: 2021-12-30 22:00:00
tags:
- css
categories: css
keywords: 百分比,css
description: 百分比,css
cover: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.xyhtml5.com%2Fwp-content%2Fuploads%2F2020%2F06%2F5ed45650851a7305.jpg&refer=http%3A%2F%2Fwww.xyhtml5.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1643462644&t=8267e67a83a70ddfdbe025c19b1c74b0
top_img: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.xyhtml5.com%2Fwp-content%2Fuploads%2F2020%2F06%2F5ed45650851a7305.jpg&refer=http%3A%2F%2Fwww.xyhtml5.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1643462644&t=8267e67a83a70ddfdbe025c19b1c74b0
---


在写 CSS 的时候，我们有时会用到百分比。百分比是一种相对值，如果浏览器要将百分比正确展示到页面上，首先需要找到它的参照，之后根据其参照计算转化为一个绝对值（比如100px），然后在页面绘制出 100px 的长度。

CSS中的参照有很多，有包含块、自身、字体大小等。在区分 CSS 中百分比的参照类型之前，我们先来看下包含块是什么。

## 包含块

### 包含块的定义
 在 CSS中，一个元素的位置和尺寸的计算，都取决于一个矩形的边界，这个矩形被称作是包含块(containing block)。
- 一个框的包含块，指的是该框所存在的那个包含块，并不是它建造的包含块
- 包含块可用于自动值的计算/宽高的计算/浮动元素的定位/绝对元素的定位

### 包含块的判定

- 根元素：称为初始包含块（initial containing block），大小为可视区域大小
- static/relative：最近的块级、单元格（table cell）或者行内块（inline-block）祖先元素的内容框
- fixed：脱离文档流，包含块为可视窗口
- absolute：包含块为最近的 'position' 属性为 'absolute'、'relative' 或者 'fixed' 的祖先元素
- 行内包含块由两部分决定：1. 第一个和最后一个inline框的内边距； 2. 排版方向。direction:ltr => 包含块的左边界是祖先元素第一行的顶、左内边距边界，右边界是祖先元素最后一个框的底、右内边距边界。但是包含块的左边框不能比右边框靠右，最多是同一个位置；direction:rtl => 包含块的右边界一定是元素的最右边，而左边界则最后一行的左边界

## 参照类型划分

### 参照包含块宽度
- width / max-width / min-width
- left / right
- padding / margin （如果照高度计算，会造成死循环；方便计算上下左右一致的边距）
- text-indent
- grid-template-columns / grid-auto-columns / column-gap
- ...

### 参照包含块高度
- height / max-height / min-height
- top / bottom
- grid-template-rows / grid-auto-rows / row-gap
- ...

### 参照自身宽高
- border-radius
- background-size
- transform:translate()
- transform-origin
- border-image-width
- zoom
- clip-path
- ...

### 参照继承字号
- font-size

### 参照自身字号
- line-height

### 参照自身行高
- vertical-align 

### 其他计算方式
- background-position: 放置背景图的区域尺寸，减去背景图的尺寸得到，可以为负值）
- border-image-slice: 图片尺寸
- filter: opacity/blur等

## 延伸实例

这里我简单总结了工作中看到了现象，但不知道是因为什么的情况。

### height: auto 不生效
我们有时会遇到设置了 height：100%，但高度仍然不正确的情况，这是因为如果其包含块没有明确的高度定义（也就是说，取决于内容高度），且这个元素不是绝对定位，则该百分比值等同于 auto（参照上图中的判断条件）。如果我们设置 html 最近包裹的元素的高度为 100% 却可以，是因为标签 html 的包含块是初始包含块，高度为视口高度，定义百分比是有效的，因此 html 标签的100%是有效的。

### height:100% 和height:inherit 的区别
height:100% 参照包含块的高度， height:inherit 参照的父级高度，如果子元素为 absolute 定位，直接父级不是其包含块，两者表现会不同。

### 父级overflow:hidden不生效
同 height:100%，只有包含块设置的 overflow:hidden 才能生效。



