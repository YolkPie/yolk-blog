---
title: vertical-align的一些理解
date: 2021-07-21 16:00:00
tags:
- vertical-align
categories: css
keywords: vertical-align,css
description: vertical-align,css
cover: https://img10.360buyimg.com/imagetools/jfs/t1/174603/21/20420/7958/60f666c7E00eec665/064780ec4f8d53aa.jpg
top_img: https://img10.360buyimg.com/imagetools/jfs/t1/174603/21/20420/7958/60f666c7E00eec665/064780ec4f8d53aa.jpg
---

# 例子

首先看个具体的例子：

``` html

    <style>
        .parent {
            width: 250px;
            background: lightgray;
        }
        .child {
            display: inline-block;
            width: 100px;
            height: 100px;
            background: cornflowerblue;
        }
    </style>

    <div class="parent">
        <div class="child"></div>
        <div class="child"></div>
    </div>

```

上面的代码中定义了一个父元素，里面包含了两个 inline-block 的子元素，效果如下：

![没有文字](https://img12.360buyimg.com/imagetools/jfs/t1/191263/40/14083/5234/60f667a6Eaa14baf3/1b3e2b32fee5334e.jpg)

有没有发现，我们**没有定义父元素的高度，子元素也没有设置 margin-bottom，为什么父元素底部还有段间距**呢？

还是上面的样式和 DOM 结构，我们在其中一个子元素上添加一段文字，效果又成了下面这个样子：

![添加了文字](https://img10.360buyimg.com/imagetools/jfs/t1/174603/21/20420/7958/60f666c7E00eec665/064780ec4f8d53aa.jpg)

为什么**添加了文字之后，两个子元素的对齐情况完全变了**呢？

其实上面的现象都是与 CSS 的 vertical-align 属性有关系，希望你看完本文之后能了解为什么会产生上面的现象并处理这些对齐问题。

# vertical-align 起作用的前提

我们有时会遇到设置了 vertical-align 的属性但是没有任何效果的情况，这是因为 vertical-align 的作用对象是有限制的。MDN 上关于 vertical-align 定义如下：

> CSS 的属性 vertical-align 用来指定行内元素（inline）或表格单元格（table-cell）元素的垂直对齐方式。

也就是说，vertical-align 作用于 inline 或 table-cell 元素（当然也作用于将 display 属性设置为 inline/inline-block/table-cell 的元素）。虽然 inline 和 table-cell 元素是很基础的 CSS 知识了，我这里还是罗列了一下：
1. inline 元素
- inline： img/span/em/b/匿名元素/...
- inline-block: input/button

2. table-cell 元素
- table-cell: td

因为 table-cell 元素的对齐比较简单，使用 table 时大多也会内嵌一层 dom，单独对模块进行排版，本文的讨论范围只限于 inline 元素。

# vertical-align 的属性值

## 继承类（inherit）

我们对 inherit 再熟悉不过了，vertical-align 是支持继承的，子元素可以从父元素继承 vertical-align 属性的值。

## 文本类（text-top/text-bottom）

文本类的属性值定义如下：
- text-top: 使元素的顶部与父元素的字体顶部对齐
- text-bottom: 使元素的底部与父元素的字体底部对齐

从定义看出，这类属性值与父元素的字体有关。我们定义父元素的字体大小为24px，3个包含了文字的子元素的字体大小分别为12px、24px和48px，对齐效果如下：

![text-top](https://img14.360buyimg.com/imagetools/jfs/t1/184967/36/14715/17623/60f6705fE00a71cd8/381a3549ccd68f67.jpg)

为了更清楚看到对齐的边界，在设置了 vertical-align: text-top 的元素的上边缘和字体的背景进行了标注，如下：

![text-top标注](https://img10.360buyimg.com/imagetools/jfs/t1/176662/21/20007/21352/60f667b9E44a06d76/57ca2cd15b69ba11.jpg)

上图可以看出，左侧 div 确实是与 24px 的文字的顶部对齐的。

再复杂一点，我们增加一个 inline-block 的 div 作为子元素，效果如下：

![text-top-div](https://img12.360buyimg.com/imagetools/jfs/t1/180270/12/15093/32986/60f675e5Ede949710/abe894573253645c.jpg)

从上面的例子我们可以知道，text-top/text-bottom 只与父级字体大小有关系，与左右元素是没有关系的。如果我们要确定两个元素的对齐关系，不管中间是否插入元素，可以考虑使用这种对齐方式。

## 线类（baseline/top/middle/bottom）

vertical-align 中我们用的最多，遇到问题也最多的应该就是线类属性了。线类属性值的定义如下：

- baseline（默认值）: 使元素的基线与父元素的基线对齐
- middle: 使元素的中部与父元素的基线加上父元素x-height（译注：x高度）的一半对齐
- top: 使元素及其后代元素的顶部与整行的顶部对齐
- bottom: 使元素及其后代元素的底部与整行的底部对齐

### top 和 bottom

- top: 使元素及其后代元素的顶部与整行的顶部对齐
- bottom: 使元素及其后代元素的底部与整行的底部对齐

我们先来看下相对简单的 top 和 bottom，这里以 top 为例，定义中包含了两个概念，一个是元素的顶部，一个是整行的顶部。

> Align the top of the aligned subtree with the top of the line box.

- 元素的顶部：元素盒模型的顶部，包含margin
- 整行的顶部：行框盒子的顶部。如果对行框盒子不了解，请看这里：[css行高line-height的一些理解](https://yolkpie.net/2020/09/29/css%E8%A1%8C%E9%AB%98line-height%E7%9A%84%E4%B8%80%E4%BA%9B%E7%90%86%E8%A7%A3/）

下面的例子中，div 为 inline-block，右侧的字体设置了 margin-top: 20px。内联盒子和幽灵节点的标注为黄色，行框盒子的标注为绿色，可以看到，盒子的顶部是对齐的。
![top的box拆分](https://img13.360buyimg.com/imagetools/jfs/t1/189699/34/14117/48245/60f68e60E1bfaa8ad/159f50fccf019ce1.jpg)

如果把中间的文字设置margin-bottom: 100px，其他子元素的对齐方式由 top 变为 bottom，效果如下：
![bottom](https://img10.360buyimg.com/imagetools/jfs/t1/196386/36/13857/53504/60f69068E5c17673b/0b3d7981c0208058.jpg)

从上面的例子我们可以看到，使用 top 和 bottom 属性值时，对齐的是行框盒子的顶部或底部。遇到问题时，把行框盒子拆分下就比较容易理解了。

### baseline

- baseline（默认值）: 使元素的基线与父元素的基线对齐

baseline 是无处不在的，因为 vertical-align 的默认值就是 baseline。从 baseline 的定义来看，我们需要弄清楚元素的基线是什么（[baseline定义](https://www.w3.org/TR/CSS2/visudet.html#leading)）：

> The baseline of an 'inline-block' is the baseline of its last line box in the normal flow, unless it has either no in-flow line boxes or if its 'overflow' property has a computed value other than 'visible', in which case the baseline is the bottom margin edge.

inline-block 的基线是正常流中（非float/absolute/fixed/html根节点）最后一个内联盒子的基线，除非这个元素里既没有内联盒子或者本身的 overflow 的计算值不是 visile，这种情况下 baseline 是元素的 margin 底边缘。也就是说，如果 inline-block 中没有元素或者 overflow 属性不是 visible，这个元素的基线是 margin 底边缘，其他的情况都是最后一个内联盒子的基线。

我们知道，文字的基线是 x 字母的下边缘。我们可以通过这个原理来模拟父元素的基线，可以通过在父元素上添加一个 after 伪元素选择器，内容为字母 x，那么父元素的基线就是字母 x 的下边缘线。

``` css
.line-box::after {
    content: 'x';
    background: red;
}
```

现在我们来看下开头的第一个问题：为什么父元素底部还有段间距？

首先，我们先来确定每个子元素的基线：按照 baseline 的定义，因为两个 inline-block 的 div 中没有元素，所以这两个 div 的基线为div的下边缘。另外，每个行框盒子的最前面都有一个幽灵节点，继承了父级的行高和字体，我们可以用一个 x 来模拟这个幽灵节点，所以幽灵节点的基线就是字母 x 的下边缘。子元素的基线确定好了之后，我们来看下父元素的基线，父元素的基线是最后一个元素，也就是第二个 div 的基线，即第二个div的下边缘。让子元素的基线和父元素的基线挨个对齐（如下图），因为幽灵节点（最左侧 x）占有高度，所以父元素底部还是有段间距的。

![baseline](https://img12.360buyimg.com/imagetools/jfs/t1/175280/21/20204/15111/60f6a48fE9c78ad9d/0c82afb21cb01e2a.jpg)

这个底部间距的问题我们也会经常遇到，最常见的就是图片（对应例子中的 inline-block 的 div 元素）的底部会有间距。要去掉这个间距，有下面几种方式：

1. 设置 line-height 为 0

既然产生间距的原因是幽灵空白节点产生了高度，我们设置 line-height 为 0 就可以了。

![设置 line-height 为 0](https://img11.360buyimg.com/imagetools/jfs/t1/37070/2/15867/3913/60f667b9E3e6b1901/d61cf9162e999f21.jpg)

2. 设置 font-size 为 0

设置 font-size 为 0 的原理和设置 line-height 为 0 是一样的，都是要消灭幽灵空白节点的高度：

![设置 font-size 为 0](https://img13.360buyimg.com/imagetools/jfs/t1/175112/38/20270/4276/60f667b8E5094edea/f07099f37888fdd5.jpg)

3. 设置图片为 block 元素，可以消灭空白节点

无话可说，但是这种情况下图片只能一行一个。

4. 设置 vertical-align 为 top/bottom/middle

使用这种方式的目的是使幽灵节点上移，这样底部的边距就不见了

![设置 vertical-align 为 bottom](https://img12.360buyimg.com/imagetools/jfs/t1/177490/28/14879/3376/60f667b9E6f4eb049/b7cac5b87b1f0bc3.jpg)

我们再来看第二个问题：为什么加了字母之后对齐方式变了？

还是先来分析下每个子元素的基线（幽灵空白节点依然用字母 x代替），第一个 div 的基线是底边缘，第二个 div 里因为有文字，这个 div 的基线是文字的基线。父元素的基线是第二个 div 的基线，即第二个 div 中 x 的底边缘。挨个对齐后，如下图：

![baseline+文字](https://img14.360buyimg.com/imagetools/jfs/t1/196320/22/13985/20645/60f6a706E8ee4b769/711c4b8a8c8e1073.jpg)

### middle

- middle: 使元素的中部与父元素的基线加上父元素x-height（译注：x高度）的一半对齐

直观一点，就是使元素的中部和我们在父元素上添加的 x 字母的中心对齐。通常我们使用 vertical-align 来实现垂直居中的效果。

我们设置子元素（可以是图片，也可以是任意长度的文字）为 inline-block, vertical-align 为middle，如下：

![middle 近似垂直居中](https://img14.360buyimg.com/imagetools/jfs/t1/177456/35/15068/14974/60f7b5bfEd782eba6/37e1ab4dd1c3b0f6.jpg)

上图中，黄色虚线为父元素的中线，红色为各个子元素的中线，可以看到，父元素和子元素的中线是重合的。但是，上面的居中只是“近似”垂直居中，为什么呢，上面的例子中，父元素的基线应该是幽灵空白节点的基线，我们知道，因为视觉上的需要，字体的基线普遍是偏下的，所以字母 x 的中心也是在中点偏下的位置。我们把父元素的字体增大，就会看到这种差异了：

![middle 近似垂直居中细节](https://img13.360buyimg.com/imagetools/jfs/t1/177479/26/15032/21212/60f7b8edE7fcfa534/5e2bceee9b52f87e.jpg)
 
那如何实现“真正”的垂直居中呢？我们首先想到的是把父元素的 font-size 或者 line-height 设置为 0，但是这种方法有局限性，比如后面要加一段文本，如果 font-size 为 0 的话文本就会被隐藏掉，line-height 为 0 的话就会失去高度。这时我们可以给父元素的最后添加一个空白的内联元素，效果如下：

``` html
    <span style="display:inline-block;height: 100%;vertical-align: middle;"></span>
```

![middle 垂直居中](https://img11.360buyimg.com/imagetools/jfs/t1/193366/24/14097/25238/60f7bf9eE4c58b131/9c725c1e32a17c70.jpg)

## 数值类百分比类（20px/2rem/20%...）

数值类百分比类的定义如下：

- length: 使元素的基线对齐到父元素的基线之上的给定长度。可以是负数
- percentage: 使元素的基线对齐到父元素的基线之上的给定百分比，该百分比是line-height属性的百分比。可以是负数。

理解了 baseline 之后，这部分已经很简单了，就是在 baseline 的基础上加上偏移。有时候我们要把文字或图片上下调几个像素的时候会使用 relative + top 来处理，这种情况下可以考虑用 vertical-align 来代替。

## 上标下标类（sub/super）

上标下标类的属性值定义如下：
- sub: 使元素的基线与父元素的下标基线对齐
- super: 使元素的基线与父元素的上标基线对齐

这种对齐方式在实际中并没有用到。不过联想到 html 中 的 sup 和 sub 标签实现的是相同的功能，我们来比较下：

![sup标签](https://img14.360buyimg.com/imagetools/jfs/t1/175594/25/20431/34632/60f6705fE2b77fea6/3df735778a7d5bea.jpg)

从图上可以看出，使用 sup 标签和设置 vertical-align 为 super，这两种对齐效果是一样的，不同的是 sup 标签里的文字要比正常的小一些（原字体的75%）。通常上标和下标都会要求字体要比正文小一些，因此我认为使用标签要比设置vertical-align更合适。

# 总结

CSS 的知识点很碎，而且因为兼容性的问题表现出来的总是莫名奇妙，但还是建议大家从 CSS 的各种概念入手，遇到不清楚的情况回到盒模型、行高、字体这些最基础的含义上，才能一步步梳理清楚。






























