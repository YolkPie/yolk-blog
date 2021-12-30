---
title: CSS开发快速提升技巧
date: 2021-12-20 10:08:07
---

### 使用CSS重置（reset）

css重置库如normalize.css已经被使用很多年了，它们可以为你的网站样式提供一个比较清晰的标准，来确保跨浏览器之间的一致性。

大多数项目并不需要这些库包含的所有规则，可以通过一条简单的规则来应用于布局中的所有元素，删除所有的margin、padding改变浏览器默认的盒模型。

```
*{ box-sizing:border-box; margin:0; padding:0 }
```
使用box-sizing声明是可选择，如果你使用下面继承的盒模型形式可以跳过它。

### 继承盒模型

让盒模型从html 继承：

```
html { box-sizing: border-box;  } *, *:before, *:after { box-sizing: inherit; }
```
### 使用flexbox布局来避免margin的问题

当你多少次试着去设计栅格布局如：组合或者图片画廊，如果使用浮动的方式，那么就需要去清除浮动和重置外边距来使其分解成所需要行数。为了避免nth-、first-、last-child 问题 ，可以使用flexbox 的space-between 属性值。

```
.flex-container{ display:flex; justify-content:space-between;}.flex-container .item{flex-basis:23%;}
```
### 使用:not() 解决lists边框的问题

在web设计中，我们通常使用：last-child nth-child 选择器来覆盖原先声明应在父选择器上的样式。比如说一个导航菜单，通过使用borders 来给每个链接Link创建分割符，然后再在加上一条规则 解除最后一个link的border

```
.nav li { border-right: 1px solid #666;}.nav li:last-child {border-right: none;}
```
这是一种很混乱的方式，它不仅强制浏览器以一种方式渲染，然后又通过特定的选择器来撤销它。这样覆盖样式是不可避免的。然而，最重要的是，我们可以通过使用：not伪类(pseudo-class) 在你想声明的元素上仅仅只使用一种样式：

```
.nav li:not(:last-child) {border-right: 1px solid #666;}
```
上面就是，除了最后一个li以外，所有的 .nav li 都加上了border样式，是不是很简单！

当然，你也可以使用 .nav li+li或者 .nav li:first-child ~li ,但是 ：not是更有语义化（semantic）和容易理解的。

### body上加入line-height样式

导致低样式效率（inefficient stylesheets）的一件事就是不断的重复声明。最好是做下项目规划和组合规则，这样CSS会更流畅。实现这一点，就需要我们理解级联（cascade）,以及如何在通用选择器写的样式可以继承在其他地方。

行间距（line-height）可以作为给你的整个项目设置的一个属性，不仅可以减小代码量，而且可以让你的网站的样式给一个标准的外观

```
body {line-height: 1.5;}
```
请注意，这里的声明没有单位，我们只是告诉浏览器 让它渲染行高是 渲染字体大小的1.5倍

### 垂直居中任何元素

在没有准备使用CSSGrid 布局的时候，设置垂直居中布局的全局规则是一个很好的方式，可以为优雅（elegantly）的设置内容布局奠定一个基础

```
html, body {height: 100%;margin: 0;}body {-webkit-align-items: center; -ms-flex-align: center; align-items:center; display: -webkit-flex;display: flex;  }
```
### 使用SVG

SVG使用于所有分辨类，并且所有浏览器也都支持。所以可以将.png .jpg .gif 等文件 丢弃。FontAwsome5中 也提供了SVG的图标字体。设置SVG的格式就跟其他图片类型一样：

```
.logo { background: url("logo.svg"); }
```
温馨提示：如果将SVG用在可交互的元素上比如说button，SVG 会产生无法加载的问题。可以通过下面这个规则来确保SVG可以访问到（确保在HTML中已设置适当的aria属性）

```
.no-svg .icon-only:after { content: attr(aria-label); }
```
### 使用 “OWL选择器”

使用通用选择器（universal selector）* 和相邻的兄弟选择器（adjacent sibling selector）+ 可以提供一个强大的的CSS功能，给紧跟其他元素中的文档流中的所有元素设置统一的规则

```
* + * { margin-top: 1.5rem; }
```
这是一个很棒的技巧，可以帮你创建更加均匀的类型跟间距。在上面的列子中，跟在其他元素后面的元素，比如说H3后面的H4，或者一个段落之后的一个段落，他们之间至少1.5rems的间距（大约为30px）

### 一致的垂直结构

一致的垂直节奏提供了一种视觉美学，使内容更具可读性。如果owl选择器过于通用，请在元素内使用通用选择器（*）为布局的特定部分创建一致的垂直节奏：

```
.intro > * {  margin-bottom: 1.25rem; }
```
### 对更漂亮的换行文本使用 box-decoration-break

假设您希望对换行到多行的长文本行应用统一的间距、边距、突出显示或背景色，但不希望整个段落或标题看起来像一个大块。Box Decoration Break属性允许您仅对文本应用样式，同时保持填充和页边距的完整性。

如果要在悬停时应用突出显示，或在滑块中设置子文本样式以具有突出显示的外观，则此功能尤其有用：

```
.p { display: inline-block; box-decoration-break: clone; -o-box-decoration-break: clone;-webkit-box-decoration-break:clone;}
```
内联块声明允许将颜色、背景、页边距和填充应用于每行文本，而不是整个元素，克隆声明确保将这些样式均匀地应用于每行。

### 等宽表格单元格

表格可能很难处理，所以尝试使用table-layout：fixed来保持单元格相等宽度：

```
.calendar { table-layout: fixed; }
```
### 强制使用属性选择器显示空链接

这对于通过CMS插入的链接特别有用，CMS通常不具有类属性，并帮助您在不影响级联的情况下对其进行特定样式设置。例如，\<a\>元素没有文本值，但href属性有一个链接：

```
a[href^="http"]:empty::before { content: attr(href); }
```
### 样式“默认”链接

说到链接样式，您可以在几乎每个样式表中找到一个通用的A样式。这迫使您为子元素中的任何链接编写额外的覆盖和样式规则，并且在使用像WordPress这样的CMS时，可能会导致您的主链接样式比按钮文本颜色更容易出现问题。

尝试这种较少干扰的方式为“默认”链接添加样式：

```
a[href]:not([class]) {  color: #999; text-decoration: none; transition: all ease-in-out .3s;}
```
### 比率框

要创建具有固有比率的框，您需要做的就是将顶部或底部填充应用于div：

```
.container { height: 0; padding-bottom: 20%; position: relative; } .container div { border: 2px dashed #ddd;        height: 100%; left: 0; position: absolute; top: 0; width: 100%;}
```
使用20％进行填充使得框的高度等于其宽度的20％。无论视口的宽度如何，子div都将保持其纵横比（100％/ 20％= 5：1）。

### 风格破碎的图像

这个技巧不是关于代码缩减，而是关于细化设计细节的。破碎的图像发生的原因有很多，要么不雅观，要么导致混乱（只是一个空元素）。用这个小小的CSS创建更美观的效果：

```
img {display: block;font-family: Helvetica, Arial, sans-serif;font-weight: 300;height: auto;line-height: 2;        position: relative;text-align: center;width: 100%;}
img:before { content: "We're sorry, the image below is missing :(";display: block;margin-bottom: 10px;}    
img:after {content: "(url: " attr(src) ")";display: block;font-size: 12px;}
```
### 使用rem进行全局大小调整；使用em进行局部大小调整

在设置根目录的基本字体大小后，例如html字体大小：15px；，可以将包含元素的字体大小设置为rem:

```
article { font-size: 1.25rem; }    aside { font-size: .9rem; }
```
然后将文本元素的字体大小设置为em

```
h2 { font-size: 2em; }    p { font-size: 1em; }
```
现在，每个包含的元素都变得分区化，更易于样式化、更易于维护和灵活。

### 隐藏未静音的自动播放视频

当您处理无法从源代码轻松控制的内容时,这对于自定义用户样式表来说是一个很好的技巧。这个技巧将帮助您避免在加载页面时自动播放视频中的声音干扰访问者，并再次提供了精彩的：not（）伪选择器：

```
video[autoplay]:not([muted]) {  display: none; }
```
### 灵活运用root类型

响应布局中的字体大小应该能够自动调整到视区，从而保存编写媒体查询的工作，以处理字体大小。可以使用:not和视区单位，根据视区高度和宽度计算字体大小：

```
:root { font-size: calc(1vw + 1vh + .5vmin); }
```
现在，您可以使用根em单位，该单位基于:not：

```
body { font: 1rem/1.6 sans-serif; }
```
结合上面的rem/em技巧以获得更好的控制。

### 在表单元素上设置字体大小，以获得更好的移动体验

为了避免移动浏览器（iOS Safari等）在点击\<select\>下拉列表时放大HTML表单元素，请在添加font-size样式：

```
input[type="text"], input[type="number"], select,  textarea { font-size: 16px }
```
### CSS变量

最后，最强大的CSS级别来自于CSS变量，它允许您声明一组公共属性值，这些值可以通过样式表中任何位置的关键字重用。你可能有一套颜色在整个项目中使用，以保持一致性。

在CSS中反复重复这些颜色值不仅是件烦人的事情，而且还容易出错。如果某个颜色在某个时刻需要改变，你就不得不去寻找和替换，这是不可靠或不快速的，当为最终用户构建产品时，变量使得定制变得容易得多。例如：
```
<html>
  <style>
    :root { 
      --main-color: #06c;  
      --accent-color: #999;
    }
    h1 { 
      color: var(--main-color);
    }
    p { 
      color: var(--accent-color);
    }
  </style>
  <body>
    <h1>标题<h1>
    <p>正文</p>
  </body>
</html>
```