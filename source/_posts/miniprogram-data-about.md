---
title: 小程序开发-数据预拉取和数据上报
date: 2022-05-07
cover: https://m.360buyimg.com/img/jfs/t1/182118/18/24568/116420/6275d6a7E8ae4ac42/ea2e417140ee4af7.jpg
---

## 周期性更新

> 生效条件：用户七天内使用过的小程序

周期性更新能够在用户未打开小程序的情况下，也能从服务器提前拉取数据，当用户打开小程序时可以更快地渲染页面，减少用户等待时间，增强在弱网条件下的可用性。

### 使用流程 

#### 1. 配置数据下载地址
登录小程序 MP 管理后台，进入设置 -> 开发设置 -> 数据周期性更新，点击开启，填写数据下载地址。

配置前：
![](https://m.360buyimg.com/img/jfs/t1/132809/38/842/64278/5ed2356bE59d5feff/ade47add4d131f87.png)

配置后：
![](https://m.360buyimg.com/img/jfs/t1/130807/24/860/77529/5ed2379eEf130b379/5be76e9a5f95df54.png)

> 1.每次修改都需要管理员扫码

> 2.只能配置一个

> 3.是一个整个带参数的请求地址

#### 2. 设置 TOKEN
第一次启动小程序时，调用 `wx.setBackgroundFetchToken()` 设置一个 TOKEN 字符串，可以跟用户态相关，会在后续微信客户端向开发者服务器请求时带上，便于给后者校验请求合法性。

示例：
```
App({
  onLaunch() {
    wx.setBackgroundFetchToken({
      token: 'xxx'
    })
  }
})
```

#### 3. 微信客户端定期拉取数据
微信客户端会在一定的网络条件下，每隔 12 小时（以上一次成功更新的时间为准）向配置的数据下载地址发起一个 HTTP GET 请求，其中包含的 query 参数如下，数据获取到后会将整个 HTTP body 缓存到本地。

| 参数      |  类型  |                             说明 |
| :-------- | :----: | -------------------------------: |
| appid     | String |                       小程序标识 |
| token     | String |                 前面设置的 TOKEN |
| timestamp | Number | 时间戳，微信客户端发起请求的时间 |

> query 参数会使用 urlencode 处理
> 开发者服务器接口返回的数据类型应为字符串，且大小应不超过 256KB，否则将无法缓存数据

#### 4. 读取数据
用户启动小程序时，调用 `wx.getBackgroundFetchData()` 获取已缓存到本地的数据。
示例：
```
App({
  onLaunch() {
    wx.getBackgroundFetchData({
      fetchType: 'periodic',
      success(res) {
        console.log(res.fetchedData) // 缓存数据
        console.log(res.timeStamp) // 客户端拿到缓存数据的时间戳
      }
    })
  }
})
```

#### 5. 调试方法
由于微信客户端每隔 12 个小时才会发起一次请求，调试周期性更新功能会显得不太方便。 因此为了方便调试周期性数据，工具提供了下面的调试能力给到开发者。

### 周期性数据调试

#### 工具拉取周期性数据
由于微信客户端每隔 12 个小时才会发起一次请求，调试周期性更新功能会显得不太方便。 目前新增能够在开发者工具上调试整个流程，操作路径为点击菜单 工具 -> 拉取周期性缓存数据, 点击后开发者工具会立即向配置的数据下载地址请求数据，如下图所示：

![](https://m.360buyimg.com/img/jfs/t1/130486/12/813/468771/5ed237eaEb94e6e81/5ab58fd6c67f8973.png)

#### 工具清除周期性数据
如果需要清除工具缓存的周期性数据，可以通过点击工具栏的 清除数据缓存 或者 全部清除 来进行清除。

#### 真机调试触发客户端立即拉取
若需要在真机进一步检验，开发者工具（20190919 及以上的版本）提供触发客户端立即拉取周期性数据的调试能力。 通过点击面板 周期性缓存 -> 拉取缓存 ，将会通知客户端拉取周期性数据。

![](https://m.360buyimg.com/img/jfs/t1/117248/35/8778/249237/5ed2381eE76eecd83/b21e86b26c64d0fe.png)

> 注：只有 Android/iOS 7.0.7 及其以上版本才可使用真机调试触发客户端立即拉取的调试能力

#### 常规错误情况

- 没有设置 TOKEN
当出现 token not set, maybe you should setBackgroundFetchToken first 错误信息，表示当前小程序没有主动去设置预拉取数据所需要的 TOKEN 信息。

## 数据预拉取
预拉取能够在小程序冷启动的时候通过微信后台提前向第三方服务器拉取业务数据，当代码包加载完时可以更快地渲染页面，减少用户等待时间，从而提升小程序的打开速度 。

### 使用流程

#### 1. 配置数据下载地址
登录小程序 MP 管理后台，进入设置 -> 开发设置 -> 数据预加载，点击开启，填写数据下载地址，只支持 HTTPS 。

#### 2. 设置 TOKEN
第一次启动小程序时，调用 `wx.setBackgroundFetchToken()` 设置一个 TOKEN 字符串，可以跟用户态相关，会在后续微信客户端向开发者服务器请求时带上，便于给后者校验请求合法性。(方法同上面周期性更新)

#### 3. 微信客户端提前拉取数据
当用户打开小程序时，微信服务器将向开发者服务器（上面配置的数据下载地址）发起一个 HTTP GET 请求，其中包含的 query 参数如下，数据获取到后会将整个 HTTP body 缓存到本地。

| 参数      |  类型  | 必填  |                                                                                                说明 |
| :-------- | :----: | :---: | --------------------------------------------------------------------------------------------------: |
| appid     | String |  是   |                                                                                          小程序标识 |
| token     | String |  否   |                                                                                    前面设置的 TOKEN |
| code      | String |  否   | 用户登录凭证，未设置TOKEN时由微信侧预生成，可在开发者后台调用 auth.code2Session，换取 openid 等信息 |
| timestamp | Number |  是   |                                                                    时间戳，微信客户端发起请求的时间 |
| path      | String |  否   |                                                                                    打开小程序的路径 |
| query     | String |  否   |                                                                                   打开小程序的query |
| scene     | Number |  否   |                                                                                  打开小程序的场景值 |

> query 参数会使用 urlencode 处理

> token和code只会存在一个，用于标识用户身份。

> 开发者服务器接口返回的数据类型应为字符串，且大小应不超过 256KB，否则将无法缓存数据

#### 4. 读取数据
用户启动小程序时，调用 `wx.getBackgroundFetchData()` 获取已缓存到本地的数据。
```
App({
  onLaunch() {
    wx.getBackgroundFetchData({
      fetchType: 'pre', // 区分数据途径
      success(res) {
        console.log(res.fetchedData) // 缓存数据
        console.log(res.timeStamp) // 客户端拿到缓存数据的时间戳
        console.log(res.path) // 页面路径
        console.log(res.query) // query 参数
        console.log(res.scene) // 场景值
      }
    })
  }
})
```
#### 调试方法
为了方便调试数据预拉取，工具提供了下面的调试能力给到开发者

### 数据预拉取调试

#### 工具触发数据预拉取
目前在新版开发者工具（1.02.1911202及以上版本）本地设置中新增设置 启用数据预拉取, 设置后开发者工具会在小程序运行时向用户配置的预拉取数据下载地址请求数据，如下图所示：

![](https://m.360buyimg.com/img/jfs/t1/122009/15/3438/221467/5ed23c55E272bfe9e/6ebec9a0e3006e4b.png)

#### 工具清除预拉取数据
如果需要清除工具缓存的预拉取数据，可以通过点击工具栏的 清除数据缓存 或者 全部清除 来进行清除。

#### 常规错误情况
- 没有设置 TOKEN
当出现 token not set, maybe you should setBackgroundFetchToken first 错误信息，表示当前小程序没有主动去设置预拉取数据所需要的 TOKEN 信息

## 自定义数据上报
> 自定义数据上报仅对微信6.5.4及以上版本生效，用户微信版本更新前无法收集数据，新版本覆盖全量用户前，数据可能有缺失。

> 目前自定义分析只提供最近30天数据。

开发者工具上可以编辑和调试[自定义分析](https://developers.weixin.qq.com/miniprogram/analysis/custom/)的数据上报功能，点击菜单栏中的 “工具 - 自定义分析” 即可弹窗打开自定义分析：

![](https://res.wx.qq.com/wxdoc/dist/assets/img/event_list.011e466a.png)


在页面中可以新建、查看或修改事件，在修改事件的页面中编辑完毕后，点击底部的保存并测试按钮将保存当前配置，同时工具将在调试器上提示收到最新配置，并展示配置信息，展示的内容包括事件的 ID 和名称，以及每个动作的触发条件和上报数据，详细介绍请移步官方文档[自定义分析](https://developers.weixin.qq.com/miniprogram/analysis/custom/)：

![](https://res.wx.qq.com/wxdoc/dist/assets/img/begin_test.52222154.png)
![](https://res.wx.qq.com/wxdoc/dist/assets/img/on_app_config.5338b60e.png)

接着可以在模拟器中操作和触发事件。在模拟器中刷新小程序也将获取该测试配置，除非窗口被关闭。窗口关闭后模拟器不会再收到配置。当事件被触发上报时，工具上会展示上报信息，包括事件 ID、触发页面、触发方式、触发时动作、以及上报的字段值和数据：

![](https://res.wx.qq.com/wxdoc/dist/assets/img/report_ide.9358e199.png)

同时可以在窗口中点击 “同步结果” 会同步显示上报的数据：

![](https://res.wx.qq.com/wxdoc/dist/assets/img/report_mp.af47c41c.png)

关闭窗口后，配置将全部失效，模拟器不再收到配置并不再触发上报（小程序中使用 wx.reportAnalytics API 进行的数据上报仍会在工具中输出）。 测试成功后，可到小程序后台发布事件配置，即可正式生效收集所定义的事件数据。



### 相关链接

- [周期性更新](https://developers.weixin.qq.com/miniprogram/dev/framework/ability/background-fetch.html)
- [周期性数据调试](https://developers.weixin.qq.com/miniprogram/dev/devtools/periodic-data.html)
- [数据预拉取](https://developers.weixin.qq.com/miniprogram/dev/framework/ability/pre-fetch.html)
- [数据预拉取调试](https://developers.weixin.qq.com/miniprogram/dev/devtools/prefetch-data.html)
- [自定义数据上报](https://developers.weixin.qq.com/miniprogram/dev/devtools/debug.html#%E8%87%AA%E5%AE%9A%E4%B9%89%E6%95%B0%E6%8D%AE%E4%B8%8A%E6%8A%A5)
- [自定义分析](https://developers.weixin.qq.com/miniprogram/analysis/custom/)
