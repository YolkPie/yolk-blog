---
title: ReactNative初尝入坑
date: 2022-04-02 21:10:58
tags:
---

### 样式
rn是基于flex布局,可以理解为rn的布局中在最外层默认给我们添加了display:flex,并且rn中flex布局默认是竖直排列的,想要横向排列要设置flexDirection:"row"
rn页面中并不是所有样式属性都可以用,比如css2那些样式如float,position等就不能用,具体能用什么在官网中按照rn-xx组件-style顺序去查看,有些样式是rn独有的
rn中所有尺寸都是没有单位的,尺寸是逻辑像素点,比如设备的dpr=2(物理像素/逻辑像素),我们设置width=50(逻辑像素),渲染到设备上是100物理像素
使用styledSheet时,同一个元素的样式要用数组写,否则后面会覆盖前面的


```js
pageContainer: {
    height: '100%',
    backgroundColor: '#f4f5f7',
    overflow: 'hidden',
    paddingBottom: JDDevice.getRpx(150),
  },
  lookTipsBox: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
    paddingTop: JDDevice.getRpx(20),
  },
  lookTipsIconBox: {
    marginRight: JDDevice.getRpx(8),
    width: JDDevice.getRpx(28),
    height: JDDevice.getRpx(28),
  },
  lookTipsIcon: {
    width: JDDevice.getRpx(28),
    height: JDDevice.getRpx(28),
  },
  lookTipsTxt: {
    fontSize: JDDevice.getRpx(22),
    color: '#C71622',
  },
  tipsTxt: {
    fontSize: JDDevice.getRpx(22),
    color: '#999999',
  },
```
- 在rn中是不支持fixed定位的；同时很多样式属性需要注意，都需要分开写，不支持简写的方式

### 长列表

rn中提供了flatList组件用于长列表数据,而且元素可以增删,和ScrollView不同的时,FlatList并不会立即渲染所有的数据,而是优先渲染屏幕上可见的数据,所以在rn项目中FlatList已经帮我们做好了路由懒加载,我们无须在去做了



### 网络
- rn中不能使用jquery,因为jq内部很多是浏览器的api,很多操作是rn用不了的


### props
在rn项目中我们打印this.props为空,但是不代表this.props内部没有数据,只是打印不出来而已


### 三目运算


```js
{this.props.flag?<div></div>:""}//这种写法错误
{this.props.flag?<div></div>:null}//这宗写法才正确

```
  
### 常见问题

- 一.出现错误提示,内容为 “Unrecognized font family ‘antoutline’，这是因为用了AntDesign Mobile 的UI组件的缘故。

解决办法：
1. 进入 node_modules/@ant-design/icons-react-native/fonts 目录, 制 antfill.ttf 和 antoutline.ttf 到 Xcode Project -> Resources folder.
2. 两个文件 添加到 info.plist ->Fonts provided by application 配置中。

```js
<key>UIAppFonts</key>
<array>
    <string>antfill.ttf</string>
    <string>antoutline.ttf</string>
</array>
```

- 二.编译时遇到Duplicate resources – Android

```js
// Create dirs if they are not there (e.g. the “clean” task just ran)
doFirst {
jsBundleDir.deleteDir()
jsBundleDir.mkdirs()
resourcesDir.deleteDir()
resourcesDir.mkdirs()
jsIntermediateSourceMapsDir.deleteDir()
jsIntermediateSourceMapsDir.mkdirs()
jsSourceMapsDir.deleteDir()
jsSourceMapsDir.mkdirs()
}

//添加

doLast {
def moveFunc = { resSuffix ->
File originalDir = file(“$buildDir/generated/res/react/release/drawable-${resSuffix}”);
if (originalDir.exists()) {
File destDir = file(“$buildDir/../src/main/res/drawable-${resSuffix}”);
ant.move(file: originalDir, tofile: destDir);
}
}
moveFunc.curry(“ldpi”).call()
moveFunc.curry(“mdpi”).call()
moveFunc.curry(“hdpi”).call()
moveFunc.curry(“xhdpi”).call()
moveFunc.curry(“xxhdpi”).call()
moveFunc.curry(“xxxhdpi”).call()
}
```

- 三.navigation 问题
ios端 每一个界面都是一个单独的ViewController，但是react-native中的渲染任何的内容都是在一个ViewController中，本质上react-native就是个root-view，那么在打卡助手中的导航器交互的拓展性会存在隐患？

假设我们从ViewController中，到跳转到 react-native开发的 员工管理界面。

这是我们在react-native注册的组件名称RNHighScores;那么我们在ios中需要实例化一个moduleName 为 RNHighScores 的 RCTRootView，然后这个view实例化的内容是整个MyApp

那么在react-native端我们实例化的永远都是App这个组件，也就是react-navigation这个导航器，并且initialRouteName永远是IndexPage。其实在正常场景下的跳转是没有问题的，但是如果遇到native想要固定调起某个react-native下的route就会无法实现。

如果上面描述的问题成立的话，有2种解决方案

1.每个放弃react-navigation，使用AppRegistry.registerComponent注册多个RCTRootView，然后实现跳转需要依赖native代码去跳转不同的ViewController。这样的缺点是js代码无法自由的跳转到任意的界面，需要依赖native提供方法

2.保留react-navigation，利用react-navigation的initialRouteName属性，在native端实例化rootView的需要传入initialProperties参数，js通过参数中的initialRouteName来处理渲染哪个route。这样js依然可以在react-navigation内部使用js去完成跳转。

参数也可以通过initialRouteParams来设置。

例如：

```js
export default class Root extends Component {
    render() {
        const {
            initialRouteName,
            initialRouteParams,
        } = this.props
        const Container = App({ initialRouteName, initialRouteParams})
        return (
            <View style={{flex:1}}>
                <Container/>
            </View>
        );
    }
}
```
