---
title: flex布局
date: 2021-12-28
author: 7
---
#### 简介：

- flex布局（Flexible布局，弹性布局）是在小程序开发经常使用的布局方式
- 开启了flex布局的元素叫做`flex container`

- `flex container`里面的直接子元素叫做`flex items`（也就是开启了flex布局的盒子包裹的第一层子元素）
- 设置display的属性为flex或者`inline-flex`可以开启flex布局即成为`flex container`

------

#### 属性值设置为flex和inline-flex的区别：

1. 如果display对应的值是flex的话，那么`flex container`是以`block-level`的形式存在的，相当于是一个块级元素
2. 如果display的值设置为`inline-flex`的话，那么`flex container`是以`inline-level`的形式存在的，相当于是一个行内块元素

1. 这两个属性值差异的影响在设置了属性值的元素上面，它们在子元素上的效果都是一样的
2. 如果一个元素的父元素开启了flex布局；那么其子元素的display属性对自身的影响将会失效，但是对其内容的影响依旧存在的；

举个例子：父元素设置了`display: flex`，即使子元素设置了`display：block`或者`display：inline`的属性，子元素还是会表现的像个行内块元素一样，这就是父元素对其的影响使其display属性对自身的影响失效了；

但是为什么我们说其对内容的影响还在呢？假如说父子元素都设置了`display: flex`，那么子元素自身依然是行块级元素，并不会因为其开启了flex布局就变为块级元素，但是该子元素的内容依然会受到它flex布局的影响，各种flex特有的属性就会生效；

总结：我们如果想让设置flex布局的盒子变成块级元素的话，那就dispaly的属性值就设置为flex；如果想让盒子变为行内块元素的话，就设置为`inline-flex`；父元素开启了flex布局之后，子元素的display属性对元素本身的影响就会失效，但是依旧可以影响盒子内部的元素；

#### 应用在flex container上的CSS属性

1. **flex-flow**

- `felx-flow`是`flex-direction || flex-wrap`的缩写，这个属性很灵活，你可以只写一个属性，也可以两个都写，甚至交换前后顺序都是可以的
- `flex-flow：column wrap` === `flex-direction：column；flex-wrap：wrap`

- 如果只写了一个属性值的话，那么另一个属性就直接取默认值；`flex-flow：row-reverse` === `flex-direction：row-reverse；flex-wrap：nowrap`；

1. **flex-direction**

`flex items`默认都是沿着`main axis`（主轴）从`main start`开始往`main end`方向排布的

- `flex-direction`决定了主轴的方向，有四个取值
- 分别为`row`（默认值）、`row-reverse`、`column`、`column-reverse`

- 注意：`flex-direction`并不是直接改变`flex items`的排列顺序，他只是通过改变了主轴方向间接的改变了顺序

| **属性值**     | **main start（主轴的起始位置）** | **main end（主轴的结束位置）** | **主轴方向** |
| -------------- | -------------------------------- | ------------------------------ | ------------ |
| row（默认值）  | 左                               | 右                             | -------->    |
| row-reverse    | 右                               | 左                             | <--------    |
| column         | 上                               | 下                             | 从上往下     |
| column-reverse | 下                               | 上                             | 从下往上     |

1. **flex-wrap**

flex-wrap能够决定flex items是在单行还是多行显示

- nowrap（默认）：单行

本例中父盒子宽度为500px，子盒子为100px；当增加了多个子盒子并且给父盒子设置了flex-wrap：nowrap属性后，效果如下图所示：

我们会惊奇的发现，父盒子的宽度没有变化，子盒子也确实没有换行，但是他们的宽度均缩小至能适应不换行的条件为止了，这也就是flex布局又称为弹性布局的原因

所以，我们也可以得出一个结论：如果使用了flex布局的话，一个盒子的大小就算是将宽高写死了也是有可能发生改变的

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6cd369915cc14ef19525f1f087c277ae~tplv-k3u1fbpfcp-watermark.awebp)![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de6237bbe45a4c3c8df9d6e586a37b36~tplv-k3u1fbpfcp-watermark.awebp)

- wrap：多行

换行后元素是往哪边排列跟交叉轴的方向有很大的关系，排列方向是顺着交叉轴的方向来的；

用的还是刚刚的例子，只不过现在将属性flex-wrap的值设置为了wrap，效果如下图所示：

子盒子的高度在能够正常换行的情况不会发生变化，但因为当前交叉轴的方向是从上往下的，那么要换行的元素就会排列在下方

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb15a665a2f645da93028e2340479486~tplv-k3u1fbpfcp-watermark.awebp)

- wrap-reverse：多行（对比wrap，cross start与cross end相反），这个方法可以让交叉轴起点和终点相反，这样整体的布局就会翻转过来

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8529d8d76304b8ab4fb0f19aa3d8887~tplv-k3u1fbpfcp-watermark.awebp)

注意：这里就不是单纯的将要换行的元素向上排列，所有的元素都会受到影响，因为交叉轴的起始点和终止点已经反过来了

1. **justify-content**

Tip：下列图像灰色部分均无任何元素，其他颜色的区域为盒子内容区域

justify-content决定了flex items在主轴上的对齐方式，总共有6个属性值：

- **flex-start（默认值）：在主轴方向上与main start对齐**

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab7497e3556e4870b46b3b58d4b091b5~tplv-k3u1fbpfcp-watermark.awebp)

- **flex-end：在主轴方向上与main end对齐**

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1996d5820de4222b3e7dde5c5ccb1ca~tplv-k3u1fbpfcp-watermark.awebp)

- **center：在主轴方向上居中对齐**

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e8254152e9f40e4ae8c796d1ebf3ec6~tplv-k3u1fbpfcp-watermark.awebp)

- **space-between**

特点：

1. 与main start、main end两端对齐
2. flex items之间的距离相等

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e27be892fe4b4faaba3a826b9808fdcf~tplv-k3u1fbpfcp-watermark.awebp)

- **space-evenly**

特点：

1. flex items之间的距离相等
2. flex items与main start、main end之间的距离等于flex items的距离

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9250f45b2bd4273b11d10f47dd95401~tplv-k3u1fbpfcp-watermark.awebp)

- **space-around**

特点：

1. flex items之间的距离相等
2. flex items与main start、main end之间的距离等于flex items的距离的一半

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88a5bd2f2bc84b47afcb8300e92c1bd2~tplv-k3u1fbpfcp-watermark.awebp)

1. **align-items**

`align-items`决定了**单行**`flex items`在cross axis（交叉轴）上的对齐方式

注意：主轴只要是横向的，无论`flex-direction`设置的是`row`还是`row-reverse`，其交叉轴都是从上指向下的；

主轴只要是纵向的，无论`flex-direction`设置的是`column`还是`column-reverse`，其交叉轴都是从左指向右的；

也就是说：主轴可能会有四种，但是交叉轴只有两种

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c68dda4b69be4ca88bd2321b19d6cd52~tplv-k3u1fbpfcp-watermark.awebp)

该属性具有如下几个属性值：

- stretch（默认值）：当**flex items**在交叉轴方向上的size（指width或者height，由交叉轴方向确定）为auto时，会自动拉伸至填充；但是如果`flex items`的size并不是`auto`，那么产生的效果就和设置为`flex-start`一样

注意：触发条件为：父元素设置`align-items`的属性值为`stretch`，而子元素在交叉轴方向上的size设置为auto

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1557f2fae43a4143aa89fdd5e6740f6f~tplv-k3u1fbpfcp-watermark.awebp)

- flex-start：与cross start对齐

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4be2b0407e3485a8f0d44071da4532a~tplv-k3u1fbpfcp-watermark.awebp)

- flex-end：与cross end对齐

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2a2f4a755f7401ea9d2e6494f84f984~tplv-k3u1fbpfcp-watermark.awebp)

- center：居中对齐

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0170f55d19884403b673e47b6d1fc65a~tplv-k3u1fbpfcp-watermark.awebp)

- baseline：与基准线对齐

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3f149616f86e4653af6a02d34737b2f1~tplv-k3u1fbpfcp-watermark.awebp)

至于baseline这个属性值，平时用的并不是很多，基准线可以认为是盒子里面文字的底线，基准线对齐就是让每个盒子文字的底线对齐

注意：`align-items`的默认值与`justify-content`的默认值不同，它并不是`flex-start`，而是`stretch`

1. **align-content**

- `align-content`决定了**多行**`flex-items`在主轴上的对齐方式，用法与`justify-content`类似，具有以下属性值
- `stretch`（默认值）、`flex-start`、`flex-end`、`center`、`space-bewteen`、`space-around`、`space-evenly`

- 大部分属性值看图应该就能明白，主要说一下`stretch`，当`flex items`在交叉轴方向上的size设置为`auto`之后，多行元素的高度之和会挤满父盒子，并且他们的高度是均分的，这和`align-items`的`stretch`属性有点不一样，后者是每一个元素对应的size会填充父盒子，而前者则是均分

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7dd0cb1b03d148f7a847c78ca56a11bc~tplv-k3u1fbpfcp-watermark.awebp)

#### 应用在flex items上的CSS属性

1. **flex**

- flex是flex-grow flex-shrink？|| flex-basis的简写，说明flex属性值可以是一个、两个、或者是三个，剩下的为默认值
- 默认值为flex： 0 1 auto（不放大但会缩小）
- none： 0 0 auto（既不放大也不缩小）
- auto：1 1 auto（放大且缩小）
- 但是其简写方式是多种多样的，不过我们用到最多的还是flex：n；举个"栗子"：如果flex是一个非负整数n，则该数字代表的是flex-grow的值，对应的flex-shrink默认为1，但是要格外注意：这里flex-basis的值并不是默认值auto，而是改成了0%；即flex：n === flex：n 1 0%；所以我们常用的flex：1 --> flex：1 1 0%；下图是flex简写的所有情况：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aa0bb655a1064d1f8ce0cdd3568c00cb~tplv-k3u1fbpfcp-watermark.awebp)

1. **flex-grow**

- `flex-grow`决定了`flex-items`如何扩展
- 可以设置任何非负数字（正整数、正小数、0），**默认值为0**

- 只有当`flex container`在主轴上有剩余的size时，该属性才会生效
- 如果所有的`flex items`的`flex-grow`属性值总和sum超过1，每个`flex item`扩展的size就为`flex container剩余size * flex-grow / sum`
- **利用上一条计算公式，我们可以得出：当**`flex items`**的**`flex-grow`**属性值总和sum不超过1时，扩展的总长度为剩余 size \* sum，但是sum又小于1，所以最终flex items不可能完全填充felx container**

```
<style>
  .box {
    display: flex;
    align-items: center;
    width: 500px;
    height: 500px;
    background-color: aquamarine;
  }
  .item {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100px;
    height: 100px;
    font-size: 34px;
    color: white;
  }
  .item:nth-child(1) {
    flex-grow: 1;
    background-color: bisque;
  }
  .item:nth-child(2) {
    flex-grow: 1;
    background-color: #f8f;
  }
  .item:nth-child(3) {
    flex-grow: 1;
    background-color: #ccc;
  }
</style>
复制代码
```

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a937333d070b4b819b2f6f88cd68c91b~tplv-k3u1fbpfcp-watermark.awebp)

- 如果所有的`flex items`的`flex-grow`属性值总和sum不超过1，每个`flex item`扩展的size就为`flex container剩余size * flex-grow`

```
<style>
	.item:nth-child(1) {
    flex-grow: 0.1;
    background-color: bisque;
  }
  .item:nth-child(2) {
    flex-grow: 0.2;
    background-color: #f8f;
  }
  .item:nth-child(3) {
    flex-grow: 0.3;
    background-color: #ccc;
  }
</style>
复制代码
```

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9aaef8fe29a4793a5b16a55cc2d0bba~tplv-k3u1fbpfcp-watermark.awebp)

注意：不要认为`flex item`扩展的值都是按照`flex-grow/sum`的比例来进行分配，也并不是说看到`flex-grow`是小数，就认为其分配到的空间是剩余`size*flex-grow`，这些都是不准确的。当看到flex item使用了该属性时，首先判断的应该是sum是否大于1，再来判断通过哪种方法来计算比例

- flex items扩展后的最终size不能超过max-width/max-height

```
<style>
	.item:nth-child(1) {
    flex-grow: 1;
    max-width: 150px; // 当设置了max-width之后，就算flex-grow生效且分配到的空间加上原空间大于max-width，但是最多宽度也只能到max-width的大小
    background-color: bisque;
  }
  .item:nth-child(2) {
    background-color: #f8f;
  }
  .item:nth-child(3) {
    background-color: #ccc;
  }
</style>
复制代码
```

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f9de49b4b074939a2dad96b1b877694~tplv-k3u1fbpfcp-watermark.awebp)

1. **flex-basis**

- `flex-basis`用来设置`flex items`在**主轴**方向上的base size，以后`flew-grow`和`flex-shrink`计算时所需要用的base size就是这个
- auto（默认值）、content：取决于内容本身的size，这两个属性可以认为效果都是一样的，当然也可以设置具体的值和百分数（根据父盒子的比例计算）

- 决定`flex items`最终base size因素的优先级为`max-width/max-height/min-width/min-height` > `flex-basis` > `width/height` > `内容本身的size`
- 可以理解为给`flex items`设置了flex-basis属性且属性值为具体的值或者百分数的话，主轴上对应的size（width/height）就不管用了

1. **flex-shrink**

- `flex-shrink`决定了`flex items`如何收缩
- 可以设置任意非负数字（正小数、正整数、0），默认值是1

- 当`flex items`在主轴方向上超过了`flex container`的size之后，`flex-shrink`属性才会生效
- 注意：与`flex-grow`不同，计算每个`flex item`缩小的大小都是通过同一个公式来的，计算比例的方式也有所不同

- 收缩比例 = `flex-shrink * flex item的base size`，base size就是`flex item`放入`flex container`之前的size
- 每个`flex item`收缩的size为`flex items超出flex container的size * 收缩比例 / 所有flex items 的收缩比例之和`

- `flex items`收缩后的最终size不能小于`min-width/min-height`
- **总结：当flex items的flex-shrink属性值的总和小于1时，通过其计算收缩size的公式可知，其总共收缩的距离是超出的size \* sum，由于sum是小于1的，那么无论如何子盒子都不会完全收缩至超过的距离，也就是说在不换行的情况下子元素一定会有超出**

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f7ace10e8f4496aa266b07146863d20~tplv-k3u1fbpfcp-watermark.awebp)

```
<style>
  .item {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-shrink: 1;
    height: 100px;
    font-size: 34px;
    color: white;
  }
  .item:nth-child(1) {
    width: 100px;
    background-color: bisque;
  }
  .item:nth-child(2) {
    width: 110px;
    background-color: #f8f;
  }
  .item:nth-child(3) {
    width: 120px;
    background-color: #ccc;
  }
  .item:nth-child(4) {
    width: 130px;
    background-color: yellowgreen;
  }
  .item:nth-child(5) {
    width: 140px;
    background-color: green;
  }
  .item:nth-child(6) {
    width: 150px;
    background-color: blue;
  }
  .item:nth-child(7) {
    width: 160px;
    background-color: red;
  }
</style>
复制代码
```

不同的盒子缩小的值和其自身的flex-shrink属性有关，而且还与自己的原始宽度有关，这是跟flex-grow最大的区别

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb48707bd1cf4d919d43bf69cec95949~tplv-k3u1fbpfcp-watermark.awebp)

1. **order**

- `order`决定了`flex items`的排布顺序
- 可以设置为任意整数（正整数、负整数、0），值越小就排在越前面

- 默认值为0，当`flex items`的`order`一致时，则按照渲染的顺序排列

```
<style>
  .item:nth-child(1) {
    background-color: bisque;
  }
  .item:nth-child(2) {
    order: 2;
    background-color: #f8f;
  }
  .item:nth-child(3) {
    order: -1;
    background-color: #ccc;
  }
 </style>
复制代码
```

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36f8124783fc417b9971b3faa3028fb2~tplv-k3u1fbpfcp-watermark.awebp)

1. **align-self**

- `flex items`可以通过`align-self`覆盖`flex container`设置的`align-items`
- 默认值为`auto`：默认遵从`flex container`的`align-items`设置

- `stretch`、`flex-start`、`flex-end`、`center`、`baseline`，效果跟`align-items`一致，简单来说，就是`align-items`有什么属性，`align-self`就有哪些属性，当然auto除外

```
.item:nth-child(2) {
    align-self: flex-start;
    background-color: #f8f;
 }
复制代码
```

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/873ad14a004f4a88800573ba0734fa19~tplv-k3u1fbpfcp-watermark.awebp)

#### 疑难点解析：

大家在看到flex-wrap那里换行的图片会不会有疑惑，为什么换行的元素不是紧挨着上一行的元素呢？而是有点像居中了的感觉

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8cf1e0b004984a3280515c9aa384ff56~tplv-k3u1fbpfcp-watermark.awebp)

想想多行元素在交叉轴上是上依靠哪一个属性进行排列的，当然是`align-content`了，那它的默认属性值是什么呢？--->stretch

对，就是因为默认值是stretch，但是`flex item`又设置了高度，所以flex item不会被拉伸，但是它们会排列在要被拉伸的位置；我们可以测试一下，将`flex-items`交叉轴上的size设置为auto之后，stretch属性值才会表现的更加明显，平分`flex-container`在主轴上的高度，每个元素所在的位置就是上一张图所在的位置

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e31a8512d656453396b8c414d268e3ec~tplv-k3u1fbpfcp-watermark.awebp)

