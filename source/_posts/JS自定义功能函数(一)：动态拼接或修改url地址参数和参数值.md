---
title: JS自定义功能函数(一)：动态拼接或修改url地址参数和参数值
date: 2020-09-17 20:56:29
tags:
- JavaScript
categories: JavaScript
author: 马金坤
keywords: JavaScript,函数
description: JavaScript,函数
cover: https://img13.360buyimg.com/imagetools/jfs/t1/148946/20/8503/10976/5f635ec2Ec4b824b9/ff23be1931eea077.jpg
top_img: https://img10.360buyimg.com/imagetools/jfs/t1/134571/8/10307/520305/5f636109Ea8493724/4b095f109c25c311.png
---

### 参数定义
* url: url地址参数；
* params：需要拼接或修改的参数对象列表
```
/**
 * 给url后面动态拼接参数或修改参数
 * @param url
 * @param params
 */
function changeURLArg(url, params) {
  let resUrl = url || ''
  if (url && params) {
    Object.keys(params).forEach((key, index) => {
      if (params[key]) {
        const regExp = new RegExp(`(${key}=)([^&]*)`, 'ig')
        if (regExp.test(resUrl)) {
          resUrl = resUrl.replace(regExp, `${key}=${params[key]}`)
        } else {
          let splitStr = '&'
          if (index === 0) {
            if (url.indexOf('?') === -1) {
              splitStr = '?'
            }
          }
          resUrl += `${splitStr}${key}=${params[key]}`
        }
      }
    })
  }
  return resUrl
}
```

### 示例如下
```
const url = 'https://www.jianshu.com?a=2'
const params = {
  a: 1,
  b: 2
}
const newUrl = changeURLArg(url, params) // https://www.jianshu.com?a=1&b=2
```
