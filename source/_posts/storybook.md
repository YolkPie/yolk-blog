---
title: storybook
date: 2020-09-16 20:08:07
cover: https://m.360buyimg.com/img/jfs/t1/132223/5/10083/3194/5f621705E31795792/21fcad6c66a6fd47.png
---

# storybook

### Storybook简介
  Storybook是一个辅助UI控件开发的工具。通过story创建独立的控件，让每个控件开发都有一个独立的开发调试环境。
  Storybook的运行不依赖于项目，开发人员不用担心由于开发环境、依赖问题导致不能开发控件。
  Storybook支持的框架覆盖主流的框架（React、Vue、Angular）。
  本文将介绍使用react的项目如何配置Storybook环境。

### 安装
* 1、安装命令
```
  npm i --save-dev @storybook/react
```
* 2、package.json中
```
  {
    "scripts": {
      "storybook": "start-storybook -p 9001 -c .storybook"
    }
  }
```
* 3、在工程根目录创建.storybook目录
 
* 4、在.storybook目录下创建config.js文件
```
  import { configure } from '@storybook/react';
  import 'index.scss';
  
  function loadStories() {
    require('./stories/userStory');
  }

configure(loadStories, module);
```
* 5、在项目根目录下创建stories目录
```
  // stories/index.jsx
  import React from 'react';
  import { storiesOf } from '@storybook/react';
  import BasicInfo from 'pages/components/BasicInfo';
  
  storiesOf('用户信息', module)
    .add('基础信息', () => <BasicInfo />);

```

### storybook 配置
##### storybook基础webpack配置只包含以下几项：
+  Babel
    - ES2016+ Support
    - .babelrc support
+ Webpack
  - CSS Support
  - Image and Static File Support
  - JSON Loader
+ 在.storybook目录下增加webpack.config.js文件扩展配置 （例如）
```
//处理文件需要配置相应的file-loader
  storybookBaseConfig.module.rules.push({
    test: /\.(gif|png|jpe?g|eot|woff|ttf|pdf)$/,
    loader: 'file-loader',
  });
``` 
```
//处理文件处理相应的样式
  storybookBaseConfig.module.rules.push({
    test: /\.s?css$/,
    use: ['style-loader', 
    { loader: 'css-loader', options: { importLoaders: 1 } }, 
    'postcss-loader'],
    include: path.resolve(__dirname, '../'),
  });
```

### addons
  可以理解成扩展storybook功能的插件
  * addon-actions
    - 可用于显示、事件处理程序接收的数据（多个事件触发可以写成对象形式）
  * addon-storysource 
    - 此插件主要是在插件面板中显示 story 源代码
  * addon-knobs
    - 在 story 中定义了 style、visible、children 三个动态变量传入 SlidePanel 组件，其变量类型分别对应的是 boolean、object、text。在 knob 中还可以导入许多的类型约束(number, color, array)等。
  * addon-notes
    - 可以在面板中添加 story 注释文本信息。  
    
### 实践
+ 1、目录结构
##### ![](catalog.png)
  
 + 1、结果图
 ##### ![](result.png)



#### 参考资料
- https://www.learnstorybook.com/intro-to-storybook/react/zh-cn/get-started/
- https://juejin.im/post/6844903752982331405
