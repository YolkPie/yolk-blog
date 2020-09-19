---
title: npm发包
cover: https://m.360buyimg.com/img/jfs/t1/140402/22/8997/490/5f659cc4E83d34415/c7ad8f46aa4ed8af.png
---

# npm发包


### 配置环境 安装node.js 
+ 官网下载最新的node安装包，正常安装node.js
+ 安装完成后，执行 node -v, 查看是否正常安装
+ 执行：sudo npm update npm -g，将npm 升级到最新
+ 查看npm是否正常： npm -v

### 发包具体示例
+ 注册npm账号，填写 用户名、密码和邮箱
+ 找个位置，新建一个文件夹
+ cd 进入文件夹，执行 npm init -y, 有需要改变的内容自己再去文件修改和添加
+ 使用 npm login，登录自己的 npm 账号
+ 使用 npm publish，发布自己的包到 npm
+ 查看自己发布的包是否成功，可以去别的项目执行 npm install 你发布的包名，下载成功

```
注意：
1. 发布自己包之前，应先去 npm 官网搜索自己要发布的包名是否已经存在，已存在的包名会提交失败
2. 自己发布的包更新时，每次都要到package.json, 将 version 修改，例如：从1.0.0改为1.0.1。然后再执行 npm publish进行更新
整体步骤示例图片如下：
```

##### ![](npm-1.png)
 
### 查看自己发布的包
 + 打开npm官网，输入你的包名，如有结果则存在 (如图)
 
#### ![](npm-2.png)

+ 点击个人头像 -> Packages, 可查看自己已发布的所有包 (如图)

#### ![](npm-3.jpg)



