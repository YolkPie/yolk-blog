---
title: react-native在Ios端提示用户App应用版本更新
date: 2020-12-21 21:19:23
tags:
- react-native
categories: react-native
author: 马金坤
keywords: react-native, 版本升级
description: react-native在Ios端提示用户App应用版本更新
cover: https://img10.360buyimg.com/imagetools/jfs/t1/140568/36/19544/25688/5fe0a9ceE022eddaa/15af0f1df9ab7668.jpg
top_img: https://img10.360buyimg.com/imagetools/jfs/t1/130200/34/17431/262954/5fc092d7E0b54491c/bb832c9742a8f536.png
---

### 前言
一般情况，react-native项目是集成在Ios或Android应用中，我们没有权利在rn项目中执行App应用更新操作。有时候，rn项目的部分功能依赖于App应用的最新版本，此时就需要提示用户版本更新，并跳转到应用商店。Android 端应用商店不统一，无法正确的引导用户；但在Ios端，可以快速引导用户跳转应用商店AppStore下载最新版本的App。下面的方法，以京东App为例：

### 判断京东App是否需要版本升级
1）获取当前引用的版本
```js
function getClientVersion() {
    return new Promise((resolve) => {
        JDNativeSystem.getClientVersion().then((res) => {
            resolve(res || '')
        }).catch((err) => {
            resolve('')
            console.log('JDNativeSystem.getClientVersion9 err!', err)
        })
    
    })
}
```
2）判断当前版本是否需要升级
```js
const miniVersion = '9.3.2'
// 比较版本号
function compareVersion(compareVersion, currentVersion) {
	var v1 = version2Num(compareVersion)
	var v2 = version2Num(currentVersion)
	if (v1 > v2) {
		return -1
	} else if (v1 == v2) {
		return 0
	} else {
		return 1
	}

	function version2Num(version) {
		//将版本号转为数字
		var c = version.toString().split('.')
		var num_place = ["", "0", "00", "000", "0000"]
		var r = num_place.reverse()
		for (var i = 0; i < c.length; i++) {
			var len = c[i].length
			c[i] = r[len] + c[i]
		}
		var res = c.join('')
		return res
	}
}
// 是否需要升级
function isNeedUpgrade() {
	return new Promise((resolve) => {
		getClientVersion().then((version) => {
			if (version) {
				const result = compareVersion(miniVersion, version)
				if (result >= 0) { // 不需要升级
					resolve(false)
				}
			}
			resolve(true) // 需要升级
		})
	})
}
```
若是需要版本升级，则弹框提示用户下载最新版本App
### 获取京东App在AppStore中的应用id
1）进入苹果应用商店https://www.apple.com/app-store/  
2）搜索京东，获取应用列表数据  
3）点击京东，跳转到京东详情页，根据京东详情页的链接https://apps.apple.com/us/app/%E4%BA%AC%E4%B8%9C-%E4%B8%8D%E8%B4%9F%E6%AF%8F%E4%B8%80%E4%BB%BD%E7%83%AD%E7%88%B1/id414245413 ，可以获取到京东应用的id——414245413

### 获取京东App在苹果商店的下载地址
```js
// 获取京东app在苹果商店的下载地址
function getJDAPPStoreInfo() {
	const JDAppStoreId = 414245413
	fetch(`https://itunes.apple.com/CN/lookup?id=${JDAppStoreId}`)
		.then((response) => response.json())
		.then((res) => {
			if (res && res.results && res.results.length) {
				this.JDAppVersion = res.results[0].version // 京东App最新版本
				this.JDAppStoreUrl = res.results[0].trackViewUrl // 苹果应用商店京东App的下载地址
			}
		}).catch((err) => {

		console.log('getJDAPPStoreInfo err!', err)
	})

}
```
### 跳转应用商店京东App下载页
```js
import { Linking } from 'react-native'
Linking.openURL(this.JDAppStoreUrl).catch((err) => {
    console.log(err, 'jump app store err!')
})
```
