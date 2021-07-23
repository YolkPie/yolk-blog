---
title: 我如何解决Android上字体不能垂直居中的问题
date: 2021-07-23 15:20:23
tags:
- 兼容

author: YYY
---

# 我如何解决Android上字体不能垂直居中的问题

某个风和日丽（coding）的下午，老大突然发给我一个截图，[单品页标签文字不居中.png]

![](./unnamed.jpeg)

拿出iPhone测试机检查 ？？？ 是居中的呀。。。。。。

看来事情并不简单，基本是Android手机兼容性的问题

换了两个Android手机测试，果然都有不同程度的偏移

而且发现同一页面上，有的会偏移有的却能正常垂直居中

通过测试，发现字体小于12px会产生很明显的偏移


### 问题：

Android手机上当字体小于12px通过lineheight或padding撑开无法做到垂直居中

### 听他们说

[字体原因及解决办法](https://blog.csdn.net/yangxiqian/article/details/80204072)

> 字体基本不可取，UI已经指定字体
### 实践出真知

- table布局

``` html
<div class="solution" style="display: table; height: 16px;">
  <span style=" display: table-cell; font-size: 10px; vertical-align: middle;">table 布局</span>
</div>
```

> 文本偏上

- flex 布局

```
<div class="solution" style="display: inline-flex; align-items: center; height: 16px; line-height: 1; font-size: 10px;">
  <span>flex 布局</span>
</div>
```

> 文本偏上

- zoom 缩放

``` html
<div class="solution" style="height: 32px; line-height: 32px; font-size: 20px; zoom: 0.5;">
  <span>zoom 缩放</span>
</div>
```

> 文本偏上

- 固定高度+内边距+行高设定为字体大小

``` html
<div class="solution" style="box-sizing: border-box; height: 16px; padding: 3px 0; line-height: 10px; font-size: 10px;">
  <span>固定高度+内边距+行高设定为字体大小</span>
</div>
```

> 文本偏上

- 固定高度+内边距+行高设为 normal

``` html
<div class="solution" style="box-sizing: border-box; height: 16px; padding: 3px; line-height: normal; font-size: 10px;">
  <span>固定高度+内边距+行高设为 normal</span>
</div>
```
> 文本偏上

- 内边距+行高设为 normal

``` html
<div class="solution" style="box-sizing: border-box; padding: 2px; line-height: normal; font-size: 10px;">
  <span>内边距+行高设为 normal</span>
</div>
```

> 文本居中，但在部分客户端上不居中

- 行高+字体大小设为 initial

``` html
<div class="solution" style="line-height: 16px; font-size: initial;">
  <span style="font-size: 10px;">行高+字体大小设为 initial</span>
</div>
```

> 文本居中，在最新的 Chrome 浏览器上不居中

- transform 缩放

``` html
<div class="solution" style="height: 32px; line-height: 32px; font-size: 20px; transform: scale(0.5, 0.5); transform-origin: left top;">
  <span>transform 缩放</span>
</div>
```

> 文本居中了，但是 transform 不能还原元素在 dom 上的占用区域大小


### 综上所述

只有transform缩放能完全解决居中问题，同时带来的问题是，transform不会引起重排，所以会有大面积占用空间

那我们只有能解决多余的占用空间问题不就能做到完全居中了么

那怎么解决多余的空间占用问题？

众所周知[margin是可以设置为负值的](https://zhuanlan.zhihu.com/p/25892372) 并且有一些应用margin设置为负值实现的布局

那么我们是否可以通过设置margin，来解决多余空间的占用问题？

``` html
<div class="solution" style="height: 32px; line-height: 32px; margin: -8px -8%; font-size: 20px; transform: scale(0.5, 0.5); transform-origin: center center;">
  <span>transform 缩放</span>
</div>
```

结果出奇的好，基本实现了相同的占用空间

现在为止，基本方向已经确定了，但是这种固定数值还是有些死板，同时随着dom宽度的改变 margin的值并不完全准确

此时我想到可以借助js来确定margin 的值

``` vue
<template>
  <span
    class="tag-view"
    :class="{ 'tag-view_android': isAndroid }"
    :style="`margin: -0.16rem -${widthMeth - 4}px -0.16rem -${widthMeth}px`"
    v-show="message"
  >
    {{ message }}
  </span>
</template>


<script>
export default {
  data() {
    return {
      isAndroid: this.isAndroid(),
      widthMeth: 0,
    };
  },

  props: {
    message: {
      type: String,
      default: "",
    },
  },
  methods: {
    isAndroid() {
      const u = navigator.userAgent
      return u.indexOf('Android') > -1 || u.indexOf('Adr') > -1
    }
  },

  watch: {
    message: {
      handler() {
        this.$nextTick(() => {
          this.widthMeth = this.$el.offsetWidth / 4;
        });
      },

      immediate: true,
    },
  },
};
</script>


<style lang="less" scoped>
.tag-view {
  background: rgba(240, 242, 245, 1);
  border-radius: 0.04rem;
  padding: 0 0.15rem;
  height: 0.32rem;
  line-height: 0.32rem;
  font-size: 0.22rem;
  color: rgba(66, 72, 84, 1);
  display: inline-block;
  margin-right: 0.1rem;
}

.tag-view_android {
  font-family: miui;
  padding: 0 0.3rem;
  height: 0.64rem;
  line-height: 0.64rem;
  font-size: 0.44rem;
  transform: scale(0.5, 0.5);
  transform-origin: center center;
  margin: -0.16rem -9%;
}
</style>
```

至此，margin的准确性解决了

但是如果想应用到项目里，一个样式就要封装一个组件未免也太麻烦了吧，而且对于已经存在的样式，改动也蛮多的

那能不能像 v-bind 一样 通过自定义指令来实现呢？

说干就干！

``` js
Vue.directive('tag', {
  inserted(el, binding, vnode) {
    if(!isAndroid()) return
    const needBook = [
      'fontSize',
      'height',
      'paddingTop',
      'paddingLeft',
      'paddingBottom',
      'paddingRight',
    ]

    needBook.map(item => {
      let value = getComputedStyle(el)[item]
      if (getValue(value)) {
        el.style[item] = getValue(value) * 2 + 'px'
        if(item === 'height') {
          el.style.lineHeight = getValue(value) * 2 + 'px'
        }
      }
    })

    el.style.transform = 'scale(0.5, 0.5)'
    el.style.transformOrigin = 'center center'
    el.style.margin = `-${el.offsetHeight / 4}px -${el.offsetWidth / 4 - 4}px -${el.offsetHeight / 4}px -${el.offsetWidth / 4}px`

    function getValue(params) {
      if (!params) return ''
      const arr = params.split('px')
      return arr && arr[0]
    }
    function isAndroid() {
      const u = navigator.userAgent
      return u.indexOf('Android') > -1 || u.indexOf('Adr') > -1
    }
  }
})
```

一顿操作，终于实现了效果，迫不及待去真机上验证一番

居然不居中了。。。。。

![](./unnamed.jpeg)

百思不解，只能去再问问度娘了～

原来，字号为奇数也会导致字体偏移，再看加上v-tag后的样式，height居然还有小数点

对代码进行再一次改造

``` js
needBook.map(item => {
  let value = getComputedStyle(el)[item]
  if (getValue(value)) {
    - el.style[item] = getValue(value) * 2 + 'px'
    - if(item === 'height') {
    -   el.style.lineHeight = getValue(value) * 2 + 'px'
    - }
    + if(item === 'height') {
    +   const num = Math.ceil(getValue(value))
    +   el.style[item] = (num - 1) * 2 + 'px'
    +   el.style.lineHeight = num * 2 + 'px'
    +   return
    + }
    + el.style[item] = getValue(value) * 2 + 'px'
  }
})
```

> 对获取到的高度进行向上取整，同时减少1容错误差

至此，实现了本次逻辑 

最后，分享大家一个自定义指令自用的文件结构

1. 新建directives文件夹
2. 在directives文件夹下，新建tag.js copy代码
3. 在directives文件夹下，新建index.js
``` js
import tag from './tag'
// 自定义指令
const directives = {
  tag,
}

export default {
  install(Vue) {
    Object.keys(directives).forEach((key) => {
      Vue.directive(key, directives[key])
    })
  },
}
```
4. 在main.js中挂载
``` js
import Vue from 'vue'
import Directives from './directives'

Vue.use(Directives)
```


THE END 感谢你看到最后！















