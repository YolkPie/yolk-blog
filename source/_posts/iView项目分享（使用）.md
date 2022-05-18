---
title: iView 项目分享（使用）
date: 2022-05-15 18:00:00
tags:
- Nginx
top_img: https://img10.360buyimg.com/imagetools/jfs/t1/182416/8/24894/1589572/627b823bE3136642f/9656e527822e4703.png
cover: https://img11.360buyimg.com/imagetools/jfs/t1/177355/25/23849/1202313/627b84b8E2d78637c/dc9cbe48ca92c62f.png
---
本文会首先介绍下使用 iView 组件前需要了解的简单的样式问题，之后会根据实际工作中的使用场景介绍下 iView 的使用。这里的列举的是在后台系统中经常用到的组件或者组件的组合（示例采用 CDN 引入的方式）。

# CSS 基础知识

## CSS 盒模型

CSS 盒模型本质上是一个盒子，盒子包裹着 DOM 元素，盒子由四个属性组成，从内到外分别是：content 内容、padding 内填充、border 边框、外边距 margin。

``` html
    <style>
        .box {
            width: 100px;
            height: 100px;
            padding: 20px;
            border: 10px solid lightblue;
            margin: 30px;
            background: lightgray;
        }
    </style>
    <div class="box"></div>
```

在浏览器中，我们可以通过下面方式插件 DOM 元素的盒模型（以 chrome 为例）：
1. 在浏览器中用 F12 打开开发者工具（或者在页面中右键，选择菜单中的“检查” 选项）
2. 选择要查看的元素之后，在开发者工具的右侧，就可以看到浏览器解析的盒模型的数据。
![浏览器查看盒模型](https://img10.360buyimg.com/imagetools/jfs/t1/18825/40/16328/345900/627f72dbEda3db0e7/47ba759a01ba92ab.png)

认识了盒模型之后，我们继续来了解下盒模型对 DOM 内容区 content 大小的影响。盒模型分为两种，一种是标准盒模型，一种是怪异盒模型，通过 CSS 的 box-sizing 属性设置：

``` css
    box-sizing: content-box; // 标准盒模型（默认）
    box-sizing: border-box; // 怪异盒模型
```
以 DOM 元素的宽度为例，不同盒模型下的 content 宽度的计算方式如下：
- 标准盒模型：contentWidth = width
- 怪异盒模型：contentWidth = width - paddingWidth - borderWidth

上面的示例没有设置 box-sizing，所以是默认的标准盒模型，content 的宽度和 width 一致，为 100px。如果我们设置了 ``` box-sizing: border-box``` 之后，content 的宽度应该为 100 - 20* 2 - 10* 2 = 40。
![border-box](https://img13.360buyimg.com/imagetools/jfs/t1/145814/17/28102/311170/627f77b7E28d2107b/13384de14843dee4.jpg)

**重点：**
- 标准盒模型：padding 和 border 的值不影响 DOM 元素整体的大小，使用 border-box。
- 怪异盒模型：希望 content 的宽度固定，使用 content-box 或者不设置任何的值。

## display: inline/inline-block/block 元素的区别

DOM 元素可以分成inline（行内元素或内联元素）、inline-block（行内块级元素）、block（块级元素）。

- inline 元素不会开始新的⼀⾏，并且*只占有必要的宽度*。该元素不能设置 width、height、margin-top、margin-bottom、padding-top、padding-bottom 等属性。

- block 元素总是开始新的⼀⾏，并且*占据可获得的全部宽度*（一个 div 的宽度总是会填满父元素的 content 区域的宽度）。该元素和 inline 元素不同，可以设置 width、height、margin-top、margin-bottom、padding-top、padding-bottom 等属性。

- inline-block 结合了 inline 和 block 元素的特点，可以像 block 元素一样自由设置 width、height、margin-top、margin-bottom、padding-top、padding-bottom等属性，也保留了 inline 元素不换行的特点。

DOM 元素尽管拥有默认的属性，还可以通过设置元素的display 属性 为 inline、inline-block、block 相互转换。但是在实际工作中，我们要遵循 html 标签语义化的原则，尽量不要随便切换。另外，inline 元素尽量不要嵌套 inline-block 和 block 元素。

**重点**

|  类型   | 是否换行 | 可设置高度 | 常见元素 |
|  ----  | ----  | ---- | ---- | 
| inline | 否 | 否 | span, i, b, label, a, img, addr,em ,strong 等 |
| inline-block  | 是 | 是 | img、input、select 等 |
| block  | 是 | 是 | div，ul，li，dl，p，h 等 |

# 修改 iView 样式

## 修改部分样式

如果要修改 iView 组件的部分样式，可以自定义 CSS 样式对默认的样式进行覆盖。要进行样式覆盖，首先要知道 iView 在要修改的 DOM 元素上的样式选择器是如何定义的，可以在开发者工具中按照下图中的示意复制出该 DOM 元素上的样式选择器：
![获取样式选择器](https://img13.360buyimg.com/imagetools/jfs/t1/121521/9/28138/467842/6280a001Ea58ec723/d6e104a7ab91f0e5.png)

之后，就可以写自己的样式了。

**重点：**
- 自定义的样式一定要写在引入的 iView 的样式文件之后
- iView 组件默认以前缀 .ivu- 作为命名空间

## 定制主题

iView 提供的组件目前是蓝色主题的，如果项目有指定的主题颜色，就需要对整个 iView 的主题进行重新定制，我们可以使用 iView 提供的 iView-theme 的工具来生成自定义主题的样式文件。

**重点：**
- [iVIew-theme定制主题文档](https://www.iviewui.com/docs/guide/theme)
- 如果要自定义主题，找前端去修改 Less 文件的变量并打包样式文件

# iView 组件使用

## 布局 Layout

布局就是一个页面的大框架，iView 提供了布局组件供我们使用。
- Layout：布局容器，其下可嵌套 HeaderSiderContentFooter或 Layout 本身，可以放在任何父容器中。
- Header：顶部布局，自带默认样式，其下可嵌套任何元素，只能放在 Layout 中。
- Sider：侧边栏，自带默认样式及基本功能，其下可嵌套任何元素，只能放在 Layout 中。
- Content：内容部分，自带默认样式，其下可嵌套任何元素，只能放在 Layout 中。
- Footer：底部布局，自带默认样式，其下可嵌套任何元素，只能放在 Layout 中。

下面是后台管理系统常见的头部导航、左侧菜单栏、右侧内容的布局：

``` html
    <Layout>
        <i-header>
            <span>Header</span>
        </i-header>
        <Layout>
            <Sider hide-trigger>
                <span>Sider</span>
            </Sider>
            <i-content>
                <span>Content</span>
            </i-content>
        </Layout>
        <i-footer>
            <span>Footer</span>
        </i-footer>
    </Layout>
```

上面是默认的情况，实际工作中我们希望这个布局是要充满整个页面的。如果要充满页面，我们就需要设置最外层 Layout 的高度为 100%。这里我们给最外层的 Layout 指定 class 为 layout。需要补充的样式如下：

``` css
    // 首先采用逐级继承的方式，从页面的html节点开始继承窗口的高度，一直到 layout 的直接父级节点
    html,body,#app {
        height: 100%;
    }

    // layout 高度100%
    .layout {
        height: 100%;
    }
```

在这种布局情况下，如果我们不想要 Sider 和 Content 横向填满页面，而是想固定宽度并且左右居中，那么我们可以再给 Sider 和 Content 的父级 Layout 增加 class 为 layout-main。

*如果要想一个 div 左右居中，需要设置这个 div 的 margin-left 和 margin-right 为 auto*

``` css
    .layout-main {
        width: 1260px;
        margin: 0 auto;
    }
```

最终布局的样式和 DOM 结构如下，你可以根据自己的实际需要确定是否要加 layout-main，也可自己写些样式覆盖下 iView 提供的默认样式。
``` html
    <style>
        html,body,#app {
            height: 100%;
        }
        .layout {
            height: 100%;
        }
        .layout-main {
            width: 1260px;
            margin: 0 auto;
        }
    </style>
    <div id="app">
        <Layout class="layout">
            <i-header>
                <span>Header</span>
            </i-header>
            <Layout class="layout-main">
                <Sider hide-trigger>
                    <span>Sider</span>
                </Sider>
                <i-content>
                    <span>Content</span>
                </i-content>
            </Layout>
            <i-footer>
                <span>Footer</span>
            </i-footer>
        </Layout>
    </div>
```

**重点**
- Layout 提供的组件有默认的样式，比如都有背景色，Header 高度为 64px，Sider 宽度为 200px 等，可以参照上文提到的**修改部分样式**的方法进行样式上的调整。
- Header/Sider/Content/Footer 只能嵌套在 Layout 中。
- 如果要想 Layout 充满页面，需要从 html 节点开始逐级设置高度 100%。

## 布局 Row/Col

Layout 布局主要利用现有的布局组件快速搭建大体框架，如果想要更灵活的页面布局，可以使用 iView 中的 Row/Col 栅格。iView 中的栅格和 bootstrap 的栅格是类似的，iView 将横向区域进行24等分，使用时可以根据需要对页面进行横向区域的划分。

iView 中的栅格布局按照行（row）和列（col）进行划分：
- 使用row在水平方向创建一行
- 将一组col插入在row中
- 通过设置col的span参数，指定跨越的范围，其范围是1到24
- 每个row中的col总和应该为24

比如将页面等分为4列：

``` html
    <Row>
        <i-col span="6">
            <div class="col">Col 1</div>
        </i-col>
        <i-col span="6">
            <div class="col">Col 2</div>
        </i-col>
        <i-col span="6">
            <div class="col">Col 3</div>
        </i-col>
        <i-col span="6">
            <div class="col">Col 4</div>
        </i-col>
    </Row>
```

如果想要每列之间有固定的间隔，可以增加 gutter 属性：

``` html
    <Row gutter="40">
        <i-col span="6">
            <div class="col">Col 1</div>
        </i-col>
        <i-col span="6">
            <div class="col">Col 2</div>
        </i-col>
        <i-col span="6">
            <div class="col">Col 3</div>
        </i-col>
        <i-col span="6">
            <div class="col">Col 4</div>
        </i-col>
    </Row>
```
除了通过设置 col 的 span 的数值来确定每列的宽度，在iView 中还可以设置 col 的 flex 属性。和 col 不同，flex 不是相对于栅格的份数来设置的，flex 设置的宽度实际上为：单个col 上设置的 flex 值 / 同一 row 下所有 col 设置的 flex 　之和 * row 的宽度。同样实现等宽的布局，可以设置每个 col 的 flex 值都为 1：

```html
    <Row gutter="40">
        <i-col flex="1">
            <div class="col">Col 1</div>
        </i-col>
        <i-col flex="1">
            <div class="col">Col 2</div>
        </i-col>
        <i-col flex="1">
            <div class="col">Col 3</div>
        </i-col>
        <i-col flex="1">
            <div class="col">Col 4</div>
        </i-col>
    </Row>
```

flex 除了可以设置为数字，还可以设置为指定像素的宽度（比如 200px）；flex 还可以设置为 auto，这种情况会默认填满 row 下剩余的宽度。用 Row 和 Col 实现的左侧宽度固定，右侧自适应的布局：

```html
    <Row gutter="40">
        <i-col flex="200px">
            <div class="col">Col 1</div>
        </i-col>
        <i-col flex="auto">
            <div class="col">Col 2</div>
        </i-col>
    </Row>
```

**重点：**
- Row/Col的布局方式更灵活
- col 上设置 span 大小时，同一 row 上 span 之和为 24，不够24的右侧会有空白，超过 24 会换行
- col 上设置 flex 时，是按照当前 col 上 flex 的数值在同一 row 下所有 flex 之和的比例来计算宽度的
- flex 可以设置为像素（200px），也可是设置为 auto（多个 auto 会均分剩余的宽度）

## 顶部导航部分

页面顶部导航一般由两部分组成，左侧为 logo，中间为导航 menu，最后侧为账号管理的 dropdownMenu。在 Layout 布局的 Header 里，我们可以增加 class 为 layout-logo、layout-menus、layout-dropdown 这三个 div。这三个 div 是左中右的位置关系，这时，可以使用 Layout 布局中的 Row/Col 来进行进一步的布局:

``` html
    <i-header>
        <Row>
            <i-col flex="200px">
                <div class="layout-logo"></div>
            </i-col>
            <i-col flex="auto">
                <div class="layout-menu"></div>
            </i-col>
            <i-col flex="200px">
                <div class="layout-dwropdown"></div>
            </i-col>
        </Row>
    </i-header>
```

接下来，我们可以分别在 logo、menu 和 dropdown 里放入对应的内容或组件：
- logo：iView 没有 logo 的组件，需要自己写 DOM 结构和样式
- menu：使用 Menu 组件
- dropdown：使用 Dropdown 组件

``` html
    <i-header>
        <Row>
            <i-col flex="150px">
                <div class="layout-logo"><img src="https://file.iviewui.com/dist/bf31433c102ed612fbe82afe000dda40.png" width="100" height="34" alt=""></div>
            </i-col>
            <i-col flex="auto">
                <div class="layout-menu">
                    <i-menu mode="horizontal" theme="primary" active-name="1">
                        <menu-item name="1">
                            内容管理
                        </menu-item>
                        <menu-item name="2">
                            用户管理
                        </menu-item>
                        <Submenu name="3">
                            <template slot="title">
                                统计分析
                            </template>
                            <menu-group title="使用">
                                <menu-item name="3-1">新增和启动</menu-item>
                                <menu-item name="3-2">活跃分析</menu-item>
                                <menu-item name="3-3">时段分析</menu-item>
                            </menu-group>
                            <menu-group title="留存">
                                <menu-item name="3-4">用户留存</menu-item>
                                <menu-item name="3-5">流失用户</menu-item>
                            </menu-group>
                        </Submenu>
                        <menu-item name="4">
                            综合设置
                        </menu-item>
                    </i-menu>
                </div>
            </i-col>
            <i-col flex="200px">
                <div class="layout-dwropdown">
                    <Dropdown>
                        <a href="javascript:void(0)">
                            用户名
                            <Icon type="ios-arrow-down"></Icon>
                        </a>
                        <DropdownMenu slot="list">
                            <dropdown-item>账号设置</dropdown-item>
                            <dropdown-item>退出登录</dropdown-item>
                        </DropdownMenu>
                    </Dropdown>
                </div>
            </i-col>
        </Row>
    </i-header>
```
**重点：**
- 外层使用布局 Layout 的 Header
- 左中右分布使用 Row/Col
- 使用 Menu 和 Dropdown 组件

## 左侧菜单部分

左侧菜单可以在 Sider 中使用 Menu 组件（垂直方向）：
```html
    <Sider hide-trigger>
        <i-menu theme="dark" active-name="1" width="auto">
            <menu-item name="1">
                <Icon type="ios-paper"></Icon>
                内容管理
            </menu-item>
            <submenu name="2">
                <template slot="title">
                    <Icon type="ios-people"></Icon>
                    用户管理
                </template>
                <menu-item name="2-1">新增用户</menu-item>
                <menu-item name="2-2">活跃用户</menu-item>
            </submenu>
            <Submenu name="3">
                <template slot="title">
                    <Icon type="ios-paper"></Icon>
                    统计分析
                </template>
                <menu-group title="使用">
                    <menu-item name="3-1">新增和启动</menu-item>
                    <menu-item name="3-2">活跃分析</menu-item>
                    <menu-item name="3-3">时段分析</menu-item>
                </menu-group>
                <menu-group title="留存">
                    <menu-item name="3-4">用户留存</menu-item>
                    <menu-item name="3-5">流失用户</menu-item>
                </menu-group>
            </Submenu>
            <menu-item name="4">
                <Icon type="ios-paper"></Icon>
                综合设置
            </menu-item>
        </i-menu>
    </Sider>
```

**重点：**
- 使用垂直方向的 Menu 组件
- 垂直方向的 Menu 默认宽度为 240，需要在组件上设置 width 为 auto 进行宽度自适应，否则会超出父级 Sider（宽度为200）


## 导航菜单 Menu

在顶部导航和左侧菜单的部分，我们分别使用了 Menu 的横向和垂直方向这两种展示形式。其中，Menu 组件中还使用了 MenuItem、MenuGroup、Submenu 这几个子组件。
- MenuItem：最基本的菜单项，可以直接嵌套在 Menu 中，也可以嵌套在 Submenu、MenuGroup 中
- MenuGroup: 菜单项的集合，需要嵌套在 SubMenu 中
- Submenu：子菜单项，右侧带有展开收起的箭头

使用 Submenu 时，父级菜单的内容要用 template 包起来，并且要增加 slot="title" 的属性：
``` html
    <submenu name="2">
        <template slot="title">
            <Icon type="ios-people"></Icon>
            用户管理
        </template>
        <menu-item name="2-1">新增用户</menu-item>
        <menu-item name="2-2">活跃用户</menu-item>
    </submenu>
```

Menu 的配置项（在组件上直接增加的属性）：

|属性	|说明	|类型	|默认值|
|  ----  | ----  | ---- | --- | 
|mode	|菜单类型，可选值为 horizontal（水平） 和 vertical（垂直）	|String	|vertical|
|theme	|主题，可选值为 light、dark、primary，其中 primary 只适用于 mode="horizontal"	|String	|light|
|active-name	|激活菜单的 name 值	|String/Number	|-|
|open-names	|展开的 Submenu 的 name 集合	|Array	|[]|
|accordion	|是否开启手风琴模式，开启后每次至多展开一个子菜单	|Boolean	|false|
|width	|导航菜单的宽度，只在 mode="vertical" 时有效，如果使用 Col 等布局，建议设置为 auto	|String	|240px|

Menu 事件（在组件上增加的@xxx事件）：

|事件名	|说明	|返回值|
|  ----  | ----  | ---- |
|on-select	|选择菜单（MenuItem）时触发	|name|
|on-open-change	|当 展开/收起 子菜单时触发|	当前展开的 Submenu 的 name 值数组|

如果要获取点击了 Menu，需要在标签上增加 @on-select 事件：
``` html
    <div id="app">
        <i-menu theme="dark" active-name="1" width="auto" @on-select="getMenuSelect">
            <menu-item name="1">
                <Icon type="ios-paper"></Icon>
                内容管理
            </menu-item>
            <submenu name="2">
                <template slot="title">
                    <Icon type="ios-people"></Icon>
                    用户管理
                </template>
                <menu-item name="2-1">新增用户</menu-item>
                <menu-item name="2-2">活跃用户</menu-item>
            </submenu>
            <Submenu name="3">
                <template slot="title">
                    <Icon type="ios-paper"></Icon>
                    统计分析
                </template>
                <menu-group title="使用">
                    <menu-item name="3-1">新增和启动</menu-item>
                    <menu-item name="3-2">活跃分析</menu-item>
                    <menu-item name="3-3">时段分析</menu-item>
                </menu-group>
                <menu-group title="留存">
                    <menu-item name="3-4">用户留存</menu-item>
                    <menu-item name="3-5">流失用户</menu-item>
                </menu-group>
            </Submenu>
            <menu-item name="4">
                <Icon type="ios-paper"></Icon>
                综合设置
            </menu-item>
        </i-menu>
    </div>
    <script>
        new Vue({
            el: '#app',
            methods: {
                getMenuSelect(name) {
                    console.log(name)
                }
            }
        })
    </script>
```

Menu 中其他子组件的项目用法请参考 iView 文档的说明：https://www.iviewui.com/components/menu

**重点：**
- Menu 中使用非 MenuItem、MenuGroup、Submenu 组件时，一定要注意标签闭合，比如 <Icon /> 要写成<Icon></Icon>，不然的话会造成显示问题（Icon后的文字不展示了）

## 表格 Table

iView 中 Table 组件的可配置的属性和事件有很多，这里仅列举实际中遇到的大部分情况，如果有更为定制化的需要，还是要去查官方文档：https://www.iviewui.com/components/table

### 基础表格
首先介绍下 Table 的基本用法（带边框的表格）：Table 中最基本的两个属性是 data 和 columns，data 是要展示的数据，columns 是表格的配置项（column 的配置项在文档的最底部, title 表头要展示的名字，key 对应的是 data 中数据项的 key）。

``` html
    <div id="app">
        <h1>带边框的表格</h1>
        <i-table border :columns="columns1" :data="data1"></i-table>
    </div>
    <script>
        new Vue({
            el: '#app',
            data: {
                columns1: [
                    {
                        title: 'Name',
                        key: 'name'
                    },
                    {
                        title: 'Age',
                        key: 'age'
                    },
                    {
                        title: 'Address',
                        key: 'address'
                    },
                    {
                        title: 'Date',
                        key: 'date'
                    }
                ],
                data1: [
                    {
                        name: 'John Brown',
                        age: 18,
                        address: 'New York No. 1 Lake Park',
                        date: '2016-10-03'
                    },
                    {
                        name: 'Jim Green',
                        age: 24,
                        address: 'London No. 1 Lake Park',
                        date: '2016-10-01'
                    },
                    {
                        name: 'Joe Black',
                        age: 30,
                        address: 'Sydney No. 1 Lake Park',
                        date: '2016-10-02'
                    },
                    {
                        name: 'Jon Snow',
                        age: 26,
                        address: 'Ottawa No. 2 Lake Park',
                        date: '2016-10-04'
                    }
                ]
            }
        })
    </script>
```

### 固定列

如果表格的数据项过多在页面中展示不全，表格会左右滚动，如果有些列需要固定展示的页面上，我们可以设置该列的 fixed 属性为 left 或者 right（比如固定姓名列在左侧展示）：

```
    {
        title: 'Name',
        key: 'name',
        fixed: 'left'
    }
```

### 格式化表格数据

如果接口返回的数据和实际表格中要展示的数据格式不一致（常见于日期），除了在给 data 赋值的时候将数据格式化之外，还可以在columns 中定义 render 方法。render 函数传入两个参数，第一个是 h，第二个是对象，包含 row、column 和 index，分别指当前单元格数据，当前列数据，当前是第几行。如果要把上面表格中的日期修改为xx.xx.xx的形式，columns 的配置如下：

``` js
    {
        title: 'Date',
        key: 'date',
        render: (h, params) => {
            return h('span', params.row.date.replaceAll('-', '.'));
        }
    }
```
**重点**
- 使用 render 方法时一定要 return h()，否则页面不会渲染
- [render 方法的简单介绍](https://yolkpie.github.io/2022/05/11/%E4%B8%80%E6%AD%A5%E6%AD%A5%E6%90%AD%E5%BB%BAiView%E9%A1%B9%E7%9B%AE/)：此处的 h 方法等同于文档中的 createElement 方法

### 自定义列

在实际工作中，仅仅按照接口返回字段进行渲染是不够的，比如要展示图片 + 文字的详情介绍、根据数据状态的不同展示对应的操作按钮等。
如果要自定义列，可以使用上面讲到的 render 方法：

``` js
    {
        title: 'operation',
        render: (h, params) => {
            let buttons = []
            return h('div', [
                h('Button', {
                    props: {
                        type: 'primary',
                        size: 'small',
                    },
                    on: {
                        click() {
                            alert(`年龄：${params.row.age}`)
                        }
                    }
                }, '获取年龄'),
                h('Button', {
                    props: {
                        type: 'primary',
                        size: 'small'
                    },
                    style: {
                        marginLeft: '10px'
                    },
                    on: {
                        click() {
                            alert(`地址：${params.row.address}`)
                        }
                    }
                }, '获取地址')
            ]);
        }
    }
```

如果 DOM 结构比较复杂，使用 render 方法开发起来会很繁琐，iView 还提供了 slot-scope 的写法（从 3.2.0 版本开始）。 在 columns 的某列声明 slot 后，就可以在 Table 的 slot 中使用 slot-scope。 slot-scope 的参数有 3 个：当前行数据 row，当前列数据 column，当前行序号 index。

- 首先在 columns 中定义 slot 的名称：

``` js
    {
        title: 'operation',
        slot: 'operation'
    }
```
- 在 Table 标签中增加 template 标签，在 template 上增加属性 slot=”刚在定义的 slot 名称"，通过 slot-scope 可以获取到对应数据行的数据
``` html
    <i-table border :columns="columns5" :data="data1">
        <template slot-scope="{ row, index }" slot="operation">
            <i-button type="primary" size="small" @click="alert(row.age)">获取年龄</i-button>
            <i-button type="primary" size="small" @click="alert(row.address)" style="margin-left: 10px">获取地址</i-button>
        </template>
    </i-table>
```

**重点：**
- 如果要简单格式化数据，可以使用 render 方式
- 如果 DOM 结构比较复杂，建议使用 slot

### 自定义表格样式

使用表格时，还会遇到需要指定某个列的宽度或者给某个列增加特殊样式的情况。
- 指定对齐方式：align，可选值为 left 左对齐、right 右对齐和 center 居中对齐	
- 指定宽度：width、 minWidth、maxWidth，分别指定宽度、最小宽度、最大宽度
- 指定 class：className，为 column 最外层增加对应的 class。定义好该列的 class 之后，就可以自己写样式定义这个 class 中的 DOM 的样式了。当然还有一种办法，就是在渲染数据的时候就在渲染的 DOM 上加 class。

### 带全选功能的表格

如果要在表格中增加全选功能，需要在 columns 中增加一列，指定 type 为 selection。指定之后，表格中对应的位置会展示 checkbox 的按钮，这些按钮的选中效果都已经写好了：
``` js
    {
        type: 'selection',
        width: 60,
        align: 'center'
    }
```
如果要设置 checkbox 的选中和禁用状态，可以给 data 中对应的数据增加 _checked 或者 _disabled 属性: 
``` js
    {
        name: 'Joe Black',
        age: 30,
        address: 'Sydney No. 1 Lake Park',
        date: '2016-10-02',
        _disabled: true
    },
    {
        name: 'Jon Snow',
        age: 26,
        address: 'Ottawa No. 2 Lake Park',
        date: '2016-10-04',
        _checked: true
    }
```
Table 提供了下面的事件，我们可以通过这些事件获取到选中的数据项：
- @on-select，*选中某一项*触发，返回值为 selection 和 row，分别为已选项和刚选择的项。
- @on-select-all，*点击全选*时触发，返回值为 selection，已选项。
- @on-selection-change，只要*选中项发生变化*时就会触发，返回值为 selection，已选项。

``` html
    <i-table border :columns="columns7" :data="data1" @on-select="onSelect" @on-select-all="onSelectAll" @on-selection-change="onSelectionChange">
    </i-table>
```

如果我们想要在 Table 外部增加按钮，控制 Table 的全选或者反选操作，需要先给 Table 增加 ref=xxx，之后通过 this.$refs.xxx 获取到 Table组件，再调用组件内部提供的方法：

``` html
    <i-button type="primary" @click="triggerSelectAll(true)">全选</i-button>
    <i-button type="primary" @click="triggerSelectAll(false)">反选</i-button>
    <i-table border ref="selection" :columns="columns7" :data="data1" @on-select="onSelect" @on-select-all="onSelectAll" @on-selection-change="onSelectionChange">
    </i-table>
```
``` js
    methods: {
        triggerSelectAll(isAllSelected) {
            this.$refs.selection.selectAll(isAllSelected)
        }
    }
```
**重点：**
- data 增加 _checked 或者 _disabled 属性来设置选中或禁用状态
- @on-selection-change 可以获取选中的数据
- 通过 ref 调用 Table 的 selectAll 方法进行全选货反选

## 表单 Form

后台管理系统中除了 Table 之外，还会频繁用到表单提交。iView 的 Form 表单组件可以进行数据校验和提交表单，包含了复选框、单选框、输入框、下拉选择框等元素。
iView 的表单需要 Form 和 FormItem 配合使用，在 Form 内，每个表单域由 FormItem 组成，可包含的控件有：Input、Radio、Checkbox、Switch、Select、Slider、DatePicker、TimePicker、Cascader、Transfer、InputNumber、Rate、Upload、AutoComplete、ColorPicker 等。另外，Form 表单必须要指定数据项 model。下面是一个简单的 Form 表单：

``` html
    <div id="app">
        // model 指定数据源
        <i-form :model="formData">
            // form-item 指定 label 为该数据项的标签
            <form-item label="Name">
                // form-item 中嵌套 input
                <i-input v-model="formData.name" placeholder="Enter name..." />
            </form-item>
            <form-item label="Address">
                <i-input v-model="formData.address" placeholder="Enter address..." />
            </form-item>
        </i-form>
    </div>
    <script>
        new Vue({
            el: '#app',
            data: {
                formData: {
                    name: '',
                    address: ''
                }
            }
        })
    </script>
```
上面的示例中，label 和 input 是换行展示的，如果要在同一行展示，需要在 Form 上指定 label-width 属性：

``` html
    <i-form :model="formData" :label-width="100">
        <form-item label="Name">
            <i-input v-model="formData.name" placeholder="Enter name..." />
        </form-item>
        <form-item label="Address">
            <i-input v-model="formData.address" placeholder="Enter address..." />
        </form-item>
    </i-form>
```

如果想要 FormItem 在一行展示，需要在 Form 上增加 inline 属性（FormItem 的宽度 可以添加样式修改）：
``` html
    <style>
        .myForm .ivu-form-item {
            width: 500px;
        }
    </style>

    <i-form :model="formData" :label-width="100" inline class="myForm">
        <form-item label="Name">
            <i-input v-model="formData.name" placeholder="Enter name..." />
        </form-item>
        <form-item label="Address">
            <i-input v-model="formData.address" placeholder="Enter address..." />
        </form-item>
    </i-form>
```
**重点：**
- label 和内容在一行展示：设置 label-width
- FormItem 在一行展示：增加 inline 属性
- FormItem 通过样式定义宽度

上面简单介绍了 Form 的样式问题，下面继续介绍下 Form 表单的校验问题。Form 表单校验时需要在 From 上增加 rules 属性，同时给需要验证的 FormItem 设置属性 prop 指向对应字段即可。

``` html
    // 在 Form 上指定 rules
    <i-form :model="formData" :label-width="100" inline :rules="rules">
        // 在 FormItem 上增加 prop 属性，prop 的值需要与 rules 中的字段匹配
        <form-item label="Name" prop="name">
            <i-input v-model="formData.name" placeholder="Enter name..." />
        </form-item>
        <form-item label="Address" prop="address">
            <i-input v-model="formData.address" placeholder="Enter address..." />
        </form-item>
    </i-form>

    <script>
        new Vue({
            el: '#app',
            data: {
                formData: {
                    name: '',
                    address: ''
                },
                rules: {
                    name: [
                        // 校验姓名为必填、长度大于等于3小于等于20、必须为字母
                        { required: true, message: 'The name cannot be empty', trigger: 'blur' },
                        { max: 20, message: 'The max length is 10', trigger: 'blur'},
                        { min: 3, message: 'The min length is 3', trigger: 'blur'},
                        { pattern: /^[a-z]+$/, message: 'Must be letters', trigger: 'blur'}
                    ],
                    address: [
                        { min: 3, message: 'The min length is 3', trigger: 'blur'},
                    ]
                }
            }
        })
    </script>
```

上面的示例中，校验了 name 的非空、长度、必须为英文字母等信息，如果校验逻辑复杂，或者需要调用接口异步验证，就需要定义 validator 的方法去校验。validator 方法接收三个参数 rule、 value、 callback：
- rule 是该字段上定义的规则
- value 为表单项的值
- callbak 为校验完成后执行的回调函数，成功的时候为回调函数的参数为空，失败时参数为 Error 对象

``` js
    address: [
        { min: 3, message: 'The min length is 3', trigger: 'blur'},
        {
            validator: (rule, value, callback) => {
                if (value.indexOf('street') === -1) {
                    callback(new Error('Must contain street'))
                } else {
                    callback()
                }
            },
            trigger: 'blur'
        }
    ]
```

**重点：**
- 必填：required
- 正则：pattern
- 最大/最小：max/min
- 复杂校验：validator

上面的示例都是在单个 FormItem 失去焦点（trigger:blur）时进行校验的，要一次性校验所有的字段，需要调用 Form 组件提供的方法。这里列举了 Form 表单提供的三个方法：

|方法名	|说明	|参数|
|  ----  | ----  | ---- |
|validate	|对整个表单进行校验，参数为检验完的回调，会返回一个 Boolean 表示成功与失败，支持 Promise	|callback|
|validateField	|对部分表单字段进行校验的方法，参数1为需校验的 prop，参数2为检验完回调，返回错误信息	|callback|
|resetFields	|对整个表单进行重置，将所有字段值重置为空并移除校验结果	|无|

如果要调用这些方法，需要首先给 Form 增加 ref=xxx，之后通过 this.$refs.xxx 获取到 Form 组件，再调用组件内部提供的方法。（Table 组件中也提到过）

``` html
    <i-button type="primary" size="small" @click="validateForm">校验</i-button>
    <i-button size="small" @click="resetForm">重置</i-button>
    <i-form :model="formData" :label-width="100" inline :rules="rules" ref="myForm">
        <form-item label="Name" prop="name">
            <i-input v-model="formData.name" placeholder="Enter name..." />
        </form-item>
        <form-item label="Address" prop="address">
            <i-input v-model="formData.address" placeholder="Enter address..." />
        </form-item>
    </i-form>
```

``` js
    validateForm() {
        this.$refs.myForm.validate(isSuccess => {
            console.log(`validate ${isSuccess ? 'succeed' : 'fail'}!`)
        })
    },
    resetForm() {
        this.$refs.myForm.resetFields()
    },
    validateCustom() {
        const props = ['name', 'address']
        const errors = []
        Promise.all(props.map(field => {
            this.$refs.myForm.validateField(field, error => {
                if (error) {
                    errors.push(error)
                }
            })
        })).then(data => {
            if (errors.length) {
                console.log(errors)
            } else {
                console.log('custom validate succeed!')
            }
        }).catch(err => {
            console.log('custom validate succeed!')
        })
    }
```

**重点：**
- 如果控制台报错`must call validateField with valid prop string!`，需要检查 FormItem 上的 prop 的值是否和 rules 里的字段对应
- Form 可以通过增加 disabled 属性禁止编辑（Form 下的表单类组件如果支持 diabled 属性会一起被禁用）









