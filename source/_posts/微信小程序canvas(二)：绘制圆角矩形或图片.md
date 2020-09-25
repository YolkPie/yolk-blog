---
title: 微信小程序canvas(一)：绘制圆角矩形或图片
date: 2020-09-16 20:40:11
tags:
- 微信小程序
- canvas
categories: 小程序
author: 马金坤
keywords: 微信小程序,canvas
description: 微信小程序,canvas
cover: https://img11.360buyimg.com/imagetools/jfs/t1/137079/20/10189/13846/5f62c071Ecf27d99d/a60905e9495effd0.jpg
top_img: https://img12.360buyimg.com/imagetools/jfs/t1/149507/16/8669/6948/5f62c2dcEd9b1ed60/4fb0edd003e468d7.jpg
---

参考了大牛文章的方案，链接如下：[https://juejin.im/post/5b7e48566fb9a01a1059543f](https://juejin.im/post/5b7e48566fb9a01a1059543f)
他的方案只能同时设置四个角的圆角值，在这里优化了大牛的方案，可根据需要给四个圆角设置不同的值。
### 需要定义生成圆角矩形的函数
* 参数定义：可以通过设置borderRadius或borderTopLeftRadius、borderTopRightRadius、 borderBottomRightRadius和borderBottomLeftRadius来生成四个方向的圆角矩形或图片
```
/**
 * 画圆角矩形、圆角边框和圆角图片所用到的方法
 * @param params
 * @param ctx
 */
const toDrawRadiusRect = (params, ctx) => {
  const {
    left, top, width, height, borderRadius,
    borderTopLeftRadius, borderTopRightRadius, borderBottomRightRadius, borderBottomLeftRadius
  } = params
  ctx.beginPath()
  if (borderRadius) {
    // 全部有弧度
    const br = borderRadius / 2
    ctx.moveTo(left + br, top) // 移动到左上角的点
    ctx.lineTo(left + width - br, top) // 画上边的线
    ctx.arcTo(left + width, top, left + width, top + br, br) // 画右上角的弧
    ctx.lineTo(left + width, top + height - br) // 画右边的线
    ctx.arcTo(left + width, top + height, left + width - br, top + height, br) // 画右下角的弧
    ctx.lineTo(left + br, top + height) // 画下边的线
    ctx.arcTo(left, top + height, left, top + height - br, br) // 画左下角的弧
    ctx.lineTo(left, top + br) // 画左边的线
    ctx.arcTo(left, top, left + br, top, br) // 画左上角的弧
  } else {
    const topLeftBr = borderTopLeftRadius ? borderTopLeftRadius / 2 : 0
    const topRightBr = borderTopRightRadius ? borderTopRightRadius / 2 : 0
    const bottomRightBr = borderBottomRightRadius ? borderBottomRightRadius / 2 : 0
    const bottomLeftBr = borderBottomLeftRadius ? borderBottomLeftRadius / 2 : 0
    ctx.moveTo(left + topLeftBr, top)
    ctx.lineTo(left + width - topRightBr, top)
    if (topRightBr) { // 画右上角的弧度
      ctx.arcTo(left + width, top, left + width, top + topRightBr, topRightBr)
    }
    ctx.lineTo(left + width, top + height - bottomRightBr) // 画右边的线
    if (bottomRightBr) { // 画右下角的弧度
      ctx.arcTo(left + width, top + height,
        left + width - bottomRightBr, top + height, bottomRightBr)
    }
    ctx.lineTo(left + bottomLeftBr, top + height)
    if (bottomLeftBr) {
      ctx.arcTo(left, top + height, left, top + height - bottomLeftBr, bottomLeftBr)
    }
    ctx.lineTo(left, top + topLeftBr)
    if (topLeftBr) {
      ctx.arcTo(left, top, left + topLeftBr, top, topLeftBr)
    }
  }
}
```

### 绘制顶部或底部有凹凸弧度的线框、矩形或图片
* 画线框
```
toDrawRadiusRect(***)
this.ctx.stroke()
```
* 画矩形
```
toDrawRadiusRect(***)
this.ctx.fill()
```
* 画图片
```
this.ctx.save()
toDrawRadiusRect(***) 
ctx.strokeStyle = 'rgba(255,255,255,0)'
ctx.stroke()
this.ctx.clip()
this.ctx.drawImage(***)
this.ctx.restore()
```
_注：原理是，clip() 方法会在原始画布上剪切任意形状和尺寸。一旦剪切了某个区域，则所有之后的绘图都会被限制在被剪切的区域内（不能访问画布上的其他区域）。可以在使用 clip() 方法前通过使用 save() 方法对当前画布区域进行保存，并在以后的任意时间通过 restore() 方法对其进行恢复。
