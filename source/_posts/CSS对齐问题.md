---
title: css对齐问题
date: 2022-06-23 17:00:00
tags:
- 对齐
categories: css
cover: https://img13.360buyimg.com/imagetools/jfs/t1/135283/19/11127/8821/5f73dde9E13365429/19b59ab164c39f68.png
top_img: https://img13.360buyimg.com/imagetools/jfs/t1/151511/20/1122/210584/5f72e54fEe1fe1377/b237b0fb200f4e77.png
---

# 左右对齐

## text-align

可以通过 text-align 设置文字的左右对齐方式。

``` html
    <p style="text-align: left;">文字左对齐</p>
    <p style="text-align: center;">文字居中对齐</p>
    <p style="text-align: right;">文字右对齐</p>
```
text-algin 设置的对齐方式可以适用于所有 display 为 inline 或 inline-blcok 的元素，比如图片、按钮等。比如在搜索的时候，可以设置两个按钮左右居中：

``` html
    <div style="text-align: left;">
        <button>搜索</button>
        <button>重置</button>
    </div>
    <div style="text-align: center;">
        <button>搜索</button>
        <button>重置</button>
    </div>
    <div style="text-align: right;">
        <button>搜索</button>
        <button>重置</button>
    </div>
```

## margin-left / margin-right

可以通过设置 div 的 margin-left 或者 margin-right 为 auto 实现 div 的对齐。默认情况下 div 是靠左对齐的。

``` html
    <div>默认左对齐</div>
    <div style="margin-left: auto; margin-right: auto;">div左右居中</div>
    <div style="margin-left: auto; margin-right: 0;">div右对齐</div>
```

## float

可以通过设置 div 的 float 来使 div 在同一行展示，并且设置对齐方式。(单个 div 也可以使用 float 设置对齐方式)

``` html
    <div classs="parent">
        <p>div左对齐</p>
        <div style="float: left;">div 1</div>
        <div style="float: left;">div 2</div>
    </div>
    <div classs="parent">
        <p>div右对齐</p>
        <div style="float: right;">div 1</div>
        <div style="float: right;">div 2</div>
    </div>
```
使用 float 需要注意两件事情：

- 设置 float 为 right 的时候，页面的展示顺序和 dom 结构是相反的。按上面的例子，如果想 div1 在 div2 的左边，则需要调整 div1 和 div2 的顺序：

    ``` html
        <p>div右对齐（div1在左侧）</p>
        <div style="float: right;">div 1</div>
        <div style="float: right;">div 2</div>
    ```

- 使用 float 时需要给父级清除浮动。如果不清除父级的浮动，会导致父级的高度为 0。下面列举了清除浮动的两种方式：  
   第一种是使用 clear: both 清除浮动，给父级增加下面的样式：
    ``` css
        .parent::after {
            display: block;
            content: "";
            clear: both;
        }
    ```
    另外一种是设置父级的 overflow 为 hidde
    ``` css
        .parent1 {
            overflow: hidden;
        }
    ```
注意： 在实际工作中，如果去掉元素的 overflow: hidden 之后页面布局发生变化，可能是浮动的问题，可以参照第一种方式，用 clear: both 的方式给父级清除浮动。

总结：
- 如果要实现左右居中，对于 inline 和 inline-block 元素，可以使用 text-align: center
- 对于单个 block 元素，可以通过设置 margin 的左右边距为 auto 来实现
- 对于多个 block 元素，使用 float 来设置左对齐或者右对齐

# 上下对齐

由于 html 渲染元素基本上都是自上而下的对齐方式，实际工作中也不怎么使用底部对齐的方式，这里只介绍上下居中的情况。

## height / line-height

如果想让文字上下居中，需要设置文字所在父级容器的 height 和 line-height 相等（适用于单行文字）。

``` html
    <div style="width: 200px; height: 60px; line-height: 60px; border: 1px solid lightblue;">单行文字</div>
```

## height / line-height / vertical-align

对于 inline-block 元素，除了设置 line-height 和 height 相等之外，还需要设置 vertical-align: middle 才能实现上下居中。

比如上面的文字居中的情况，可以给文字增加一级 span 标签并设置 display 为 inline-block，设置 span 的 vertical-align 为 middle（适用于单行文字和多行文字）：

``` html
    <div style="width: 200px; height: 100px; line-height: 100px; border: 1px solid lightblue;">
        <span style="display: inline-block; vertical-align: middle; line-height: 20px;">多行文字多行文字多行文字多行文字多行文字多行文字多行文字</span>
    </div>
```

我们可以将任意元素设置成 inline-block 来实现上下居中，比如实现一个左侧 icon 右侧文字的上线居中：

``` html
    <div style="width: 200px; height: 100px; line-height: 100px; border: 1px solid lightblue;">
        <i style="display: inline-block; width: 20px; height: 20px; background: red; vertical-align: middle;"></i>
        <span style="display: inline-block; vertical-align: middle;">文字</span>
    </div>
```

总结：
- 如果要实现单行文字居中，设置 line-height 与父级的 height 相等即可
- 对于 inline-block 元素，除了设置 line-height 与父级的 height 相等以外，还需要设置 vertical-align 为 middle

# 上下左右对齐

上面介绍的 CSS 属性，只能单独实现上下对齐或者居中对齐，这里介绍下可以同时设置上下左右对齐的 CSS 属性。

## position + top / left + 负 margin

首先，要设置元素的 position 为 relative / absolute / fixed（只有设置 relative 或者 absolute 之后 top 或者 left 的属性才能生效），再设置 top 和 left 都为 50%，margin 为负的子元素宽度的 50%。其中 top 和 margin-top 配合实现上下居中，left 和 margin-left 实现左右居中。

``` html
     <!-- 在父级 div 中左右居中，需要设置父元素的 position 为 relative -->
    <div style="position: relative; width: 400px; height: 400px; border: 1px solid lightblue;">
        <div style="position: relative; width: 100px; height: 100px; background-color: lightblue; left: 50%; top: 50%; margin-top: -50px; margin-left: -50px"></div>
    </div>
    <!-- 相对于窗口上下左右居中，例如 dialog -->
    <div style="position: fixed; width: 100px; height: 100px; background-color: lightblue; left: 50%; top: 50%; margin-top: -50px; margin-left: -50px"></div>
```

使用这种对齐方式时，因为要设置 margin 为指定的像素值，因此这种情况适用于元素宽高都确定的情况。

## position + top / left + transform

如果元素宽高不确定，我们可以使用 tansform 来代替负 margin，可以设置 transform: translate(-50%, -50%)。translate 的两个参数分别为横向和纵向的比例，translate(-50%, 0) 和 left 配合实现左右居中， translate(0, -50%) 和 top 配合实现上下居中。

``` html
    <div style="position: relative; width: 400px; height: 400px; border: 1px solid lightblue;">
        <div style="position: relative; display: inline-block; background-color: lightblue; left: 50%; top: 50%; transform: translate(-50%, -50%);">div 宽度不确定</div>
    </div>
```

## flex + justify-conetnt + align-items

首先要设置父元素的 display 为 flex，之后在子元素上设置 justify-content 和 align-items 实现上下左右居中。

``` html
    <div style="display: flex; align-items: center; justify-content: center; width: 400px; height: 400px; border: 1px solid lightblue;">
        <div style="display: inline-block; background-color: lightblue;">div 宽度不确定</div>
    </div>
```
flex 是一个强大的 css 属性，还可以使不同的 div 在同一行展示：

``` html
    <div style="display: flex; align-items: center; justify-content: center; width: 400px; height: 400px; border: 1px solid lightblue;">
        <div style="width: 100px; height: 100px; background-color: lightblue;">div1</div>
        <div style="width: 100px; height: 100px; background-color: lightcoral;">div2</div>
        <div style="width: 100px; height: 100px; background-color: lightyellow;">div3</div>
    </div>
```

justify-content 为 flex-start 时为左对齐， 为 flex-end 时为右对齐：

``` html
    <div style="display: flex; align-items: center; justify-content: flex-start; width: 400px; height: 400px; border: 1px solid lightblue;">
        <div style="width: 100px; height: 100px; background-color: lightblue;">div1</div>
        <div style="width: 100px; height: 100px; background-color: lightcoral;">div2</div>
        <div style="width: 100px; height: 100px; background-color: lightyellow;">div3</div>
    </div>
    <div style="display: flex; align-items: center; justify-content: flex-end; width: 400px; height: 400px; border: 1px solid lightblue;">
        <div style="width: 100px; height: 100px; background-color: lightblue;">div1</div>
        <div style="width: 100px; height: 100px; background-color: lightcoral;">div2</div>
        <div style="width: 100px; height: 100px; background-color: lightyellow;">div3</div>
    </div>
```

flex-items 为 flex-start 为上对齐，为 flex-end 时为下对齐：
``` html
    <div style="display: flex; align-items: flex-start; justify-content: center; width: 400px; height: 400px; border: 1px solid lightblue;">
        <div style="width: 100px; height: 100px; background-color: lightblue;">div1</div>
        <div style="width: 100px; height: 100px; background-color: lightcoral;">div2</div>
        <div style="width: 100px; height: 100px; background-color: lightyellow;">div3</div>
    </div>
    <div style="display: flex; align-items: flex-end; justify-content: center; width: 400px; height: 400px; border: 1px solid lightblue;">
        <div style="width: 100px; height: 100px; background-color: lightblue;">div1</div>
        <div style="width: 100px; height: 100px; background-color: lightcoral;">div2</div>
        <div style="width: 100px; height: 100px; background-color: lightyellow;">div3</div>
    </div>
```

总结：
- 如果元素的宽高确定，可以使用负 margin
- 如果宽高不确定，可以使用 transform
- flex 最强大，不仅可以实现上下最有对齐，还可以使 div 同行展示

# table 的对齐

## 原生 table 中的对齐

table 中的对齐方式比较简单，左右对齐使用 align，上下对齐使用 valign 或者 vertical-align。其中 align 和 valign 为 html 的属性，vertical-algin 为 css 属性。

``` html
    <table>
        <tr>
            <th>对齐方式</th>
            <th>列1</th>
            <th>列2</th>
            <th>列3</th>
            <th>列4</th>
        </tr>
        <tr>
            <td>左右对齐：默认左对齐</td>
            <td align="left">左对齐：align="left"</td>
            <td align="center">居中对齐：align="center"</td>
            <td align="right">右对齐：align="right"</td>
            <td align="middle" style="width: 200px; height: 200px;">
                <div style="width: 100px; height: 100px; background-color: lightblue;">align="middle"</div>
            </td>
        </tr>
        <tr>
            <td>上下对齐：默认居中对齐</td>
            <td valign="top">上对齐：valign="top"</td>
            <td valign="middle">居中对齐：valign="middle"</td>
            <td valign="bottom">下对齐：valign="bottom"</td>
            <td valign="middle" style="width: 200px; height: 200px;">
                <div style="width: 100px; height: 100px; background-color: lightblue;">valign="top"</div>
            </td>
        </tr>
        <tr>
            <td>上下对齐：默认居中对齐</td>
            <td style="vertical-align: top;">上对齐：vertical-align:top</td>
            <td style="vertical-align: middle;">居中对齐：vertical-align:middle</td>
            <td style="vertical-align: bottom;">下对齐：vertical-align:bottom</td>
            <td style="vertical-align: middle;" style="width: 200px; height: 200px;">
                <div style="width: 100px; height: 100px; background-color: lightblue;">vertical-align:top</div>
            </td>
        </tr>
    </table>
```

总结：
- table 中左右对齐用 align，上下对齐用 valign 或 vertical-align
- algin / valign / vertical 写在 td 元素上

## iview 中 table 的对齐

在 iview 中，可以在 column 中设置 align 属性来指定每列的左右对齐方式。和上面 table 中的居中方式不同，iview 中的左右居中最终是通过设置 text-align 属性实现的，因此，如果想要设置生效，每列的内容应该为 inline 或 inline-block 元素，如果不想设置 display 为 inline 或 inline-block，则需要设置 margin 为 auto 来进行对齐。 

``` html
    <div id="app">
        <i-table border :columns="columns" :data="data">
            <template slot-scope="{ row, index }" slot="custom">
                <div style="width: 150px; height: 100px; background-color: lightblue;">align:center（block）</div>
            </template>
            <template slot-scope="{ row, index }" slot="custom1">
                <div style="width: 150px; height: 100px; background-color: lightblue; display: inline-block;">align:center（inline-block）</div>
            </template>
            <template slot-scope="{ row, index }" slot="custom2">
                <div style="width: 150px; height: 100px; background-color: lightblue; margin: 0 auto;">align:center（block + margin）</div>
            </template>
        </i-table>
    </div>
    <script>
        new Vue({
            el: '#app',
            data: {
                selected: [],
                columns: [
                    {
                        title: '列1',
                        key: 'column1',
                    },
                    {
                        title: '列2',
                        key: 'column2',
                        align: 'left'
                    },
                    {
                        title: '列3',
                        key: 'column3',
                        align: 'center'
                    },
                    {
                        title: '列4',
                        key: 'column4',
                        align: 'right'
                    },
                    {
                        title: '列5',
                        slot: 'custom',
                        width: 400,
                        align: 'center'
                    },
                    {
                        title: '列6',
                        slot: 'custom1',
                        width: 400,
                        align: 'center'
                    },
                    {
                        title: '列7',
                        slot: 'custom2',
                        width: 400,
                        align: 'center'
                    },
                ],
                data: [
                    {
                        column1: '左右对齐(默认左对齐)',
                        column2: '左对齐',
                        column3: '居中对齐',
                        column4: '右对齐',
                        column5: '',
                        column6: '',
                        column7: '',
                    }
                ]
            }
        })
    </script>
```

因为 iview 中设置对齐方式是针对整个 column 的， 如果表头和内容的对齐方式不一致，就需要分开处理一下，比如表头居中、内容不居中的时候，可以设置 column 的 align 为 center，各列的内容再通过设置 margin 实现各自的对齐方式。

``` html
    <div id="app">
        <i-table border :columns="columns" :data="data">
            <template slot-scope="{ row, index }" slot="custom1">
                <div style="width: 150px; height: 100px; background-color: lightblue;">内容左对齐</div>
            </template>
            <template slot-scope="{ row, index }" slot="custom2">
                <div style="width: 150px; height: 100px; background-color: lightblue; margin-left: auto; margin-right: auto;">
                    内容居中对齐</div>
            </template>
            <template slot-scope="{ row, index }" slot="custom3">
                <div style="width: 150px; height: 100px; background-color: lightblue;margin-left: auto; margin-right: 0;">
                    内容右对齐</div>
            </template>
        </i-table>
    </div>
    <script>
        new Vue({
            el: '#app',
            data: {
                selected: [],
                columns: [
                    {
                        title: '列1',
                        with: 400,
                        align: 'center',
                        slot: 'custom1'
                    },
                    {
                        title: '列2',
                        width: 400,
                        align: 'center',
                        slot: 'custom2'
                    },
                    {
                        title: '列3',
                        width: 400,
                        align: 'center',
                        slot: 'custom3'
                    }
                ],
                data: [
                    {
                        column1: '',
                        column2: '',
                        column3: ''
                    }
                ]
            }
        })
    </script>
```

iview 目前没有提供设置上下对齐的属性，默认上下居中，如果要设置上下对齐，可以在 column 中增加 className，通过自定义样式实现上下对齐。

总结：
- iview 中需要在 column 中增加 align 属性设置左右对齐
- iview 中默认上下居中，其他对齐方式需要参照上文讲的上下对齐的方式自定义

# 附件
[示例代码](https://storage.jd.com/auction.gateway/ATTACHMENT_21cf7b731d3941d98f1f5e9d219fdb6e.zip)