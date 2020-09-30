---
title: css垂直居中的方式
date: 2020-09-29 21:00:00
tags:
- 垂直居中
categories: css
author: 崔梦林
keywords: css, 垂直居中
description: css, 垂直居中
cover: https://img13.360buyimg.com/imagetools/jfs/t1/135283/19/11127/8821/5f73dde9E13365429/19b59ab164c39f68.png
top_img: https://img13.360buyimg.com/imagetools/jfs/t1/151511/20/1122/210584/5f72e54fEe1fe1377/b237b0fb200f4e77.png
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于前端来说，用CSS实现垂直居中是常见的工作，但自从flex出现后，因为做的差不多都是移动端的页面，大家基本上都用flex了。为了方便大家写PC端样式时可以少走弯路，我把之前工作中用到过的和别人总结的垂直居中方式一并总结在这里。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;基本的DOM结构如下：

    ``` js
        <div class="parent">
            <div id="child"></div>
        </div>
    ```

## margin-top / padding-top

- 示例

    ``` js
        .parent {
            width: 300px;
            height: 300px;
        }
        .child {
            width: 100px;
            height: 100px;
            margin-top: 50px;
        }

        ---- 或者 ----

        .parent {
            width: 300px;
            height: 300px;
            padding-top: 50px;
            box-sizing: border-box;
        }
        .child {
            width: 100px;
            height: 100px;
        }

    ```

- 兼容性

![margin兼容性](https://img12.360buyimg.com/imagetools/jfs/t1/134743/34/11064/106929/5f728f81Ea80a9300/28f5d6b365a0926f.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本来我是不打算写这种方式的，但笨方法也是一种方法。

## flex

- 示例
    ``` js
        .parent {
            display: flex;
            height: 300px;
            align-items: center;
        }
        .child {
            width: 100px;
            height: 100px;
        }
    ```
- 兼容性

![flex兼容性](https://img12.360buyimg.com/imagetools/jfs/t1/143413/30/9644/176104/5f71e12bE5ff9c425/76b6644fb23fd202.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;flex在移动端的兼容性都很好，但是如果要在PC端使用的话，IE 10 以下就不要考虑了，IE 10 和11 也有一些兼容性问题需要注意：[flex兼容性](https://caniuse.com/?search=flex)。

## 绝对定位 + transform

- 示例

    ``` js
        .parent {
            position: relative;
            height: 300px;
        }
        .child {
            width: 100px;
            height: 100px;
            position: absolute;
            top: 50%;
            transform: translate(0, -50%);
        }

    ```

- 兼容性
![transform兼容性](https://img14.360buyimg.com/imagetools/jfs/t1/149912/26/9726/615389/5f728c2eE37d4379f/9ed28cd717b71f0f.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;transform是CSS 3的属性，自然对IE 8 及以下是不兼容的，IE 9 以上使用前缀就可以了。

## 绝对定位 + 负margin

- 示例

    ``` js
        .parent {
            position: relative;
            height: 300px;
        }
        .child {
            width: 100px;
            height: 100px;
            position: absolute;
            top: 50%;
            margin-top: -50px;
        }
    ```

- 兼容性

![负margin兼容性](https://img12.360buyimg.com/imagetools/jfs/t1/134743/34/11064/106929/5f728f81Ea80a9300/28f5d6b365a0926f.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这种实现方式的兼容性没什么好质疑的，CSS 2 的属性，兼容到IE 6（虽然我们早就不考虑IE 678 的兼容性了）。这种方式不够灵活的一点是，margin值必须指定为自身高度的一半，这就意味着这种实现方式只适用于child高度明确的情况。负margin在工作中还有许多应用场景，比如在父级宽高一定的情况下扩充子级宽高，利用负margin做叠加，和padding配合实现左右div等高等等。总之，在写样式的时候要记得margin可以为负的哦。

## table-cell

- 示例

    ``` js
        .parent {
            height: 300px;
            display: table-cell;
            vertical-align: middle;
        }
        .child {
            width: 100px;
            height: 100px;
        }
    ```

- 兼容性

![table-cell兼容性](https://img12.360buyimg.com/imagetools/jfs/t1/131536/2/11264/121529/5f72996eE7a195372/a618f17f7355f385.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这种实现方式不兼容IE 67，Firefox 2 有一些bug，但鉴于目前Firefox 2的版本占比为0，使用table-cell是不用考虑兼容性的问题的。这个例子只是展示table-cell这个属性在垂直居中中的作用，实际工作中我们常常使用table的一整套DOM。由于浏览器生成table会进行多次重排重绘，非常损耗性能，因此如果不是后台系统或者要展示比较规范的表格类型的数据的时候，尽量选择其他的实现方式。

## 绝对定位 + margin: auto

- 示例

    ``` js
        .parent {
            position: relative;
            height: 300px;
        }
        .child {
            width: 100px;
            height: 100px;
            position: absolute;
            top: 0;
            bottom: 0;
            margin: auto;
        }
    ```

- 兼容性

![margin:auto兼容性](https://img13.360buyimg.com/imagetools/jfs/t1/117341/34/19063/116030/5f72dd25E6b06c156/7587b74748408b51.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上图是margin: auto的兼容性。需要注意的是图中标“1”的浏览器，在怪异模式下是不支持margin:auto的。如果你对怪异模式很陌生，那么记得保证你的html页面开头是<!doctype html><html>这样的声明就行，这段声明可以保证你的页面以标准模式渲染。

## grid
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用grid其实已经超出我们在开头规定的DOM结构的范围了，但是因为grid和flex相比，可以更轻易的实现二维布局，所以还是现在这里认识一下
吧。使用grid实现垂直居中的DOM结构如下：

    ``` js
        <div class="parent">
            <div class="child1"></div>
            <div class="child"></div>
            <div class="child2"></div>
        </div>

    ```
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;和之前规定的DOM结构相比，我们在在子级增加了两个辅助的div，通过设置子级三个div的高度来实现中间div的垂直居中。如果不设置grid-template-rows的话，三个子级div会是相同的高度，也是垂直居中的。

    ``` js
        .parent {
            height: 300px;
            display: grid;
            grid-template-rows: 50px 200px 50px;
        }
    ```
- 兼容性

![margin:auto兼容性](https://img13.360buyimg.com/imagetools/jfs/t1/151511/20/1122/210584/5f72e54fEe1fe1377/b237b0fb200f4e77.png)

 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;嗯，grid最大的问题就是兼容性。PC就不要考虑了，M的如果你的项目不需要兼容那些飘红的浏览器，就可以先用起来了。grid详细的兼容性在这里：[grid兼容性](https://caniuse.com/?search=grid)。

## line-height + vertical-align

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们都知道使用line-height实现文本的垂直居中，同样的，我们也可以使用line-height和vertical-align实现图片的垂直居中。

    ``` js
        <div class="parent">
            <img class="child" src="demo.jpg">
        </div>
        <style>
            .parent {
                line-height: 300px;
            }
            .child {
                width: 100px;
                height: 100px;
                vertical-align: middle;
            }
        </style>
    ```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;按照line-height可以使inline元素垂直居中的思路，我们也可以利用line-height实现任意div的垂直居中，前提是设置其为 inline-block：

    ``` js
        .parent {
            height: 300px;
            line-height: 300px;
        }
        .child {
            display: inline-block;
            width: 100px;
            height: 100px;
            vertical-align: middle;
        }
    ```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为什么要把line-height放到最后，是因为vertical-align:middle并不是真正的垂直居中（见下图），而且各个浏览器、IOS和安卓的渲染还有些差异，如果你的项目对还原设计稿的要求很高，这个方案就放弃吧。

![垂直居中偏差](https://img12.360buyimg.com/imagetools/jfs/t1/111590/26/19209/21454/5f72f305Ee1d903b0/9941c632da17521b.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（完结）