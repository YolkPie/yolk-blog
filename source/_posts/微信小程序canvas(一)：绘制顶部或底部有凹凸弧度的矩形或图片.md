---
title: 微信小程序canvas(二)：绘制顶部或底部有凹凸弧度的矩形或图片
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

要实现顶部或底部有凹凸弧度的矩形或图片，实现的效果如下：
![图片 1.png](https://img13.360buyimg.com/imagetools/jfs/t1/90092/12/12709/105215/5e4e2a2aE8c55ded3/5caeb3c5a8aa1cf0.png)

### 第一步：需要定义生成顶部或底部有弧度的矩形的函数
* 参数定义：topRadianHeight： 顶部弧度的高度，大于0为顶部凸，小于0为顶部凹；bottomRadianHeight：底部弧度高度，大于0为底部凸，小于0为底部凹
```
/**
 * 画顶部或底部有弧度的矩形或图片需要用到的方法
 * @param params
 * @param topRadianHeight 顶部弧度，大于0为顶部凸，小于0为顶部凹
 * @param bottomRadianHeight 底部弧度，大于0为底部凸，小于0为底部凹
 * @param ctx
 */
const toDrawArcRect = (params, ctx) => { // 画上下左右方向有弧度的矩形
  const {
    left, top, width, height,
    topRadianHeight, bottomRadianHeight
  } = params
 
  const halfWidth = width / 2
  const radianHeight = topRadianHeight || bottomRadianHeight || 0
  const r = (halfWidth * halfWidth + radianHeight * radianHeight) / (2 * Math.abs(radianHeight))
  const radiusX = left + halfWidth // 圆心X的坐标
  const radianValue = Math.acos(halfWidth / r) // 弧度
  ctx.beginPath()
  if (bottomRadianHeight) { // 底部凸
    if (bottomRadianHeight > 0) {
      ctx.arc(radiusX, top + height - r, r, radianValue, -radianValue + Math.PI)
      ctx.lineTo(left, top)
      ctx.lineTo(left + width, top)
      ctx.lineTo(left + width, top - radianHeight + height)
    } else { // 底部凹
      ctx.arc(radiusX, top + height + r + radianHeight, r, radianValue - Math.PI, -radianValue)
      ctx.lineTo(left + width, top)
      ctx.lineTo(left, top)
      ctx.lineTo(left, top + height)
    }
    return
  }
  if (topRadianHeight) {
    if (topRadianHeight > 0) { // 顶部凸
      ctx.arc(radiusX, top + r, r, radianValue - Math.PI, -radianValue)
      ctx.lineTo(left + width, top + height)
      ctx.lineTo(left, top + height)
      ctx.lineTo(left, top + radianHeight)
    } else { // 顶部凹
      ctx.arc(radiusX, top - r - topRadianHeight, r, radianValue, -radianValue + Math.PI)
      ctx.lineTo(left, top + height)
      ctx.lineTo(left + width, top + height)
      ctx.lineTo(left + width, top)
    }
  }
}
```

### 第二步：绘制顶部或底部有凹凸弧度的线框、矩形或图片
* 画线框
```
toDrawArcRect(***)
this.ctx.stroke()
```
* 画矩形
```
toDrawArcRect(***)
this.ctx.fill()
```
* 画图片
```
this.ctx.save()
toDrawArcRect(***)
ctx.strokeStyle = 'rgba(255,255,255,0)'
ctx.stroke()
this.ctx.clip()
this.ctx.drawImage(***)
this.ctx.restore()
```
_注：原理是，clip() 方法会在原始画布上剪切任意形状和尺寸。一旦剪切了某个区域，则所有之后的绘图都会被限制在被剪切的区域内（不能访问画布上的其他区域）。可以在使用 clip() 方法前通过使用 save() 方法对当前画布区域进行保存，并在以后的任意时间通过 restore() 方法对其进行恢复。

