---
title: react-native实现沉浸式标题栏
date: 2020-12-07 20:46:07
tags:
- react-native
categories: react-native
author: 马金坤
keywords: react-native, 沉浸式, 标题栏
description: 在react-native项目中实现沉浸式标题栏
cover: https://img12.360buyimg.com/imagetools/jfs/t1/139355/38/17398/8687/5fce3691E780c4ba3/de55e2d253898034.jpg
top_img: https://img10.360buyimg.com/imagetools/jfs/t1/130200/34/17431/262954/5fc092d7E0b54491c/bb832c9742a8f536.png
---
### 前言
沉浸式标题栏，简单来说，即是透明栏，标题栏和状态栏不再是传统的黑色或白色，而是透明的，使得手机应用界面占据整个屏幕空间，页面从上向下滚动时，状态栏和标题内容慢慢由透明变成不透明，退出沉浸模式。以上交互，主要通过设置状态栏 StatusBar 和 透明度 opacity 来实现。

### 设置 opacity
初始情况下，标题栏和状态栏的透明度opacity为0，页面向下滚动一段距离(这里设定为标题栏的高度)后，其透明度一点点的由0变为1。  
基本思路是：    
const opacity = 滚动的距离 / 标题栏的高度  
完整代码：
```jsx
import React, {PureComponent} from 'react'
import { View, Text, ScrollView } from 'react-native'
import TitleBar from '../../components/TitleBar'

export default class TitlePage extends PureComponent {
	constructor(props) {
		super(props)
		this.state = {
			titleBarHeight: 50 // 顶部标题栏的高度
		}
		// 顶部标题栏
		this.titleBarView = null
	}

	// 监听列表滚动事件
	scrollViewScroll = (event) => {
		const y = event.nativeEvent.contentOffset.y
		// 设置标题栏和状态栏的透明度 titleOpacity
		// 当页面滚动的距离等于标题栏的高度时，其透明度变为1
		const scale = y * 1.0 / this.state.titleBarHeight
		this.titleBarView.setState({
			titleOpacity: scale,
		})
	}

	render() {
		const { titleBarHeight} = this.state
		return (
			<View>
				{/*沉浸式标题*/}
				<TitleBar
					onRef={(ref) => this.titleBarView = ref}
					titleBarHeight={titleBarHeight}
				/>
				<ScrollView
					onScroll={this.scrollViewScroll}
				>
					<Text>页面内容</Text>
				</ScrollView>
			</View>
		)
	}
}
```
### 设置 StatusBar
在 ios 端，状态栏默认是沉浸式的默认情况下， View 的内容会从屏幕顶部开始绘制；在 android 端，状态栏会遮住主题内容，需设置状态栏是否透明
* barStyle 设置状态栏文本的颜色
* translucent 指定状态栏是否透明。设置为true时，应用会在状态栏之下绘制（即所谓“沉浸式”）-- android
* backgroundColor 状态栏背景色 -- android

```jsx
_renderStatusBar = () => {
    const { titleOpacity } = this.state
    const backgroundColor = `rgba(0, 0, 0, ${titleOpacity})`
    if (Platform.OS === 'ios') {
        return <StatusBar barStyle={'dark-content'}/>
    } else if (Platform.OS === 'android') {
        return <StatusBar
            backgroundColor={backgroundColor}
            barStyle={'dark-content'}
            translucent={true}/>
    }
    return null
}
```

### 设置标题栏
* 整个标题栏的高度应该等于状态栏的高度 statusBarHeight 加上 标题模块的高度 titleBarHeight  
获取状态栏的高度:  
```jsx
import { Platform, NativeModules } from "react-native"
const { StatusBarManager } = NativeModules
// 获取状态栏的高度
const getStatusBarHeight = () => {
	return new Promise((resolve) => {
		const OS = Platform.OS
		if (OS === 'ios') {
			StatusBarManager.getHeight(statusBarHeight => {
				resolve(statusBarHeight)
			})
		} else if (OS === 'android') {
			resolve(StatusBarManager || {}).HEIGHT || 0
		}
		resolve(0)
	})

}
let statusBarHeight = 0
getStatusBarHeight().then((res) => {
    statusBarHeight = res
})
```
* 标题栏需设置成 absolute 定位，这样标题栏展示在手机主体内容上层，使得主体内容占据整个屏幕空间

完整代码：
```jsx
import React, {PureComponent} from 'react'
import {
	View,
	StatusBar,
	StyleSheet,
	Platform
} from 'react-native'
import { statusBarHeight } from '../common/globalData'

export default class TitleBar extends PureComponent {
	constructor(props) {
		super(props)
		this.props.onRef(this)
		this.state = {
			// 背景透明度
			titleOpacity: 0
		}
	}
	_renderStatusBar = () => {
		const { titleOpacity } = this.state
		const backgroundColor = `rgba(0, 0, 0, ${titleOpacity})`
		if (Platform.OS === 'ios') {
			return <StatusBar barStyle={'dark-content'}/>
		} else if (Platform.OS === 'android') {
			return <StatusBar
				backgroundColor={backgroundColor}
				barStyle={'dark-content'}
				translucent={true}/>
		}
		return null
	}
	render() {
		const { titleBarHeight } = this.props
		const { titleOpacity } = this.state
		return (
			<View style={[{height: titleBarHeight + statusBarHeight}, TitleStyle.titleBarWrapper]}>
				{this._renderStatusBar()}
				<View style={[TitleStyle.titleBarBg, {
					opacity: titleOpacity,
				}]}/>
				<View style={[TitleStyle.titleBarContent, {
					marginTop: statusBarHeight,
					height: titleBarHeight
				}]}>
				  {/*	标题栏主题内容*/}
				</View>
			</View>
		)
	}
}

const TitleStyle = StyleSheet.create({
	titleBarWrapper: {
		flexDirection: 'row',
		position: 'absolute',
		top: 0,
		zIndex: 100,
		width: '100%'
	},
	titleBarContent: {
		alignItems: 'center',
		justifyContent: 'center',
		width: '100%',
		height: 50,
	},
	titleBarBg: {
		position: 'absolute',
		top: 0,
		height: '100%',
		width: '100%',
		backgroundColor: '#fff',
	}
})
```
