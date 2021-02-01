## 前端博客源码

### 指令
#### 安装
```shell
$ npm install hexo-cli -g
$ npm install
# 需要安装hexo-theme-butterfly主题，否则打开是空白页
$ git clone -b dev https://github.com/jerryc127/hexo-theme-butterfly.git themes/Butterfly
```
#### 开发
```shell
$ hexo s                 # 本地预览
$ hexo new <title>       # 创建新文章
```

### 发布
* 提交代码至远程仓库master分支，travis-ci会自动完成部署。
* 如果自动部署没有成功，可打开[travis-ci](https://travis-ci.com/github/YolkPie/)，选择相应的项目，点击`restart build`进行手动部署。

### 主题
[hexo-theme-butterfly](https://demo.jerryc.me/)

### 搭建
[基于Hexo的个人博客搭建](https://yolkpie.net/2020/09/11/hexo/)
