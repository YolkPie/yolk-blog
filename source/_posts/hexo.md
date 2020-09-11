---
title: 基于Hexo的个人博客搭建
date: 2020-09-11 19:22:01
tags:
  - hexo
  - blog
categories: hexo
keywords: hexo,Butterfly
description: 搭建博客其实很简单，但要注意很多细节问题，之前本人也搭建过博客，那时也踩过一些坑，但是现在重新搭建博客，感觉自己又把之前踩过的坑又踩了一遍。所以为了避免以后搭建博客采坑，所以在此记录一下搭建博客的全流程以及一些注意事项。
cover: https://img10.360buyimg.com/imagetools/jfs/t1/126870/12/12238/22058/5f5b5ea6E1c09a3b9/dabe2aa379c28834.jpg
top_img: https://img14.360buyimg.com/imagetools/jfs/t1/139467/5/8161/8390/5f5b5ec5E63d8b3e6/110dea52017c578e.jpg
---

## 前言
搭建博客其实很简单，但要注意很多细节问题，之前本人也搭建过博客，那时也踩过一些坑，但是现在重新搭建博客，感觉自己又把之前踩过的坑又踩了一遍。所以为了避免以后搭建博客采坑，所以在此记录一下搭建博客的全流程以及一些注意事项。
本文搭建的博客基于[Hexo](https://hexo.io/zh-cn/)，主题选用[Butterfly](https://demo.jerryc.me/posts/21cfbf15/)，使用[Travis CI](https://travis-ci.com/)自动部署到Github Pages和Coding Pages上，并在腾讯云上申请个人域名与博客进行绑定。

## 安装Hexo
安装Hexo并初始化博客，这个没啥好说的，按照[Hexo官网](https://hexo.io/zh-cn/)指示安装初始化即可。
```sh
$ npm install hexo-cli -g     # 全局安装hexo-cli脚手架，如果不想全局安装，那就使用npx安装
$ hexo init blog              # 初始化博客
$ cd blog                     # 进入博客根目录
$ npm install                 # 安装依赖
$ hexo s                      # 本地预览
```

## 使用主题
本文的博客使用[Butterfly](https://github.com/jerryc127/hexo-theme-butterfly/tree/master)，个人觉得这个主题比较好看，大家可以根据自己的喜好进行选择。

### 安装主题
在博客根目录中执行：
```sh
$ git clone -b 3.0.1 https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly
```

**注意：** 此处我们使用的主题版本是3.0.1，建议大家固定使用一个主题版本，因为不同的主题版本的配置有所不同。在每次自动化部署时都需要重新安装主题，如果指定的是`master`，那每次安装的都是最新版本，会导致部署失败！

这里所说的版本，即tag，大家可以到主题的仓库中查找最新的tag并使用。

![主题tag](tag.png)

### 应用主题
修改站点配置文件_config.yml，把主题改为 butterfly
```yml
theme: butterfly
```

### 安装插件
如果你没有 pug 以及 stylus 的渲染器，请下载安装：
```sh
$ npm install hexo-renderer-pug hexo-renderer-stylus --save
```

## 配置主题
把主题文件夹中的 _config.yml 复制到 Hexo 根目录里，同时重新命名为 _config.butterfly.yml。

Hexo 会自动合并主题中的_config.yml 和 _config.butterfly.yml 里的配置，如果存在同名配置，会使用_config.butterfly.yml 的配置，其优先度较高。

### 配置标签页
- 1.在博客根目录中输入`hexo new page tags`
- 2.找到 source/tags/index.md 这个文件
- 3.修改为：
```Markdown
---
title: 标签
date: 2020-08-29 19:14:29
type: "tags"
---
```

### 配置分类页
- 1.在博客根目录中输入`hexo new page categories`
- 2.找到 source/categories/index.md 这个文件
- 3.修改为：
```Markdown
---
title: 分类
date: 2020-08-29 19:15:35
type: "categories"
---
```

### 配置友情链接
- 1.在博客根目录中输入`hexo new page link`
- 2.找到 source/link/index.md 这个文件
- 3.修改为：

```Markdown
---
title: 友情链接
date: 2020-08-29 19:10:49
type: "link"
---
```

- 4.添加友情链接：

在博客目录中的 source/_data，创建一个文件 link.yml
```yml
class:
  class_name: 友链
  link_list:
    1:
      name: YolkPie
      link: https://yolkpie.net/
      avatar: /img/yolkpie.png
      descr: 前端研发部
    2:
      name: Yolk CLI
      link: https://yolkpie.net/yolk-cli/
      avatar: /img/yolkcli.png
      descr: 前端项目脚手架
    3:
      name: Yolk Works
      link: https://yolkpie.net/yolkworks-list/#/
      avatar: /img/yolkworks.png
      descr: 海量模板及组件
```

### 配置关于自己
- 1.在博客根目录中输入`hexo new page about`
- 2.找到 source/about/index.md 这个文件
- 3.修改为：

```Markdown
---
title: 关于我
date: 2020-08-29 19:16:57
type: "about"
---
```

### 配置博客语言
修改博客配置文件 `_config.yml`
```yml
language: zh-CN
```

### 配置导航菜单
```yml
menu:
  首页: / || fas fa-home
  文章: /archives/ || fas fa-archive
  标签: /tags/ || fas fa-tags
  分类: /categories/ || fas fa-folder-open
  友链: /link/ || fas fa-link
  关于: /about/ || fas fa-heart
```

### 配置相关图片
包括网站图标、头像、首页顶部图片、文章页面顶部默认图、文章封面默认图、其他页面顶部默认图等等。

如果想要使用本地图片，在`source`目录下创建`img`目录，在图片放到该目录下，这样我们就可以使用`/img/avatar.jpg`访问到本地图片。

### 配置本地搜索
- 1.安装 `hexo-generator-search`
```sh
$ npm install hexo-generator-search --save
```
- 2.修改站点配置文件_config.yml，添加：
```yml
search:
  path: search.xml
  field: post
  content: true
```
- 3.修改主题配置文件_config.butterfly.yml
```yml
local_search:
  enable: true
```

### 配置评论
本博客的评论基于[Gitalk](https://github.com/gitalk/gitalk/blob/master/readme-cn.md)，Gitalk 是一个基于 GitHub Issue 和 Preact 开发的评论插件。

- 1.选择一个公共github存储库（已存在或创建一个新的github存储库）用于存储评论，一般选择我们的Github Pages。
- 2.创建 GitHub Application，如果没有[点击这里申请](https://github.com/settings/applications/new)，`Authorization callback URL`填写当前使用插件页面的域名。
- 3.修改主题配置文件_config.butterfly.yml
```yml
gitalk:
  client_id: 你的client id 
  client_secret: 你的client secret
  repo: 你的github仓库
  owner: 你的github用户名
  admin: 该仓库的拥有者或协作者
  language: zh-CN # en, zh-CN, zh-TW, es-ES, fr, ru
  perPage: 10 # Pagination size, with maximum 100.
  distractionFreeMode: false # Facebook-like distraction free mode.
  pagerDirection: last # Comment sorting direction, available values are last and first.
  createIssueManually: false # Gitalk will create a corresponding github issue for your every single page automatically
```

### 其他配置
- [Butterfly 安装文档(三) 主题配置-1](https://demo.jerryc.me/posts/4aa8abbe/)
- [Butterfly 安装文档(四) 主题配置-2](https://demo.jerryc.me/posts/ceeb73f/)

## 写篇博客
一般博客会引用一些图片，如果想要使用本地图片，则配置`_config.yml`：
```yml
post_asset_folder: true
```

执行`hexo new <title>`创建一篇文章，可以发现`source/_posts`目录下多出了`<title>`目录及`<title>.md`，其中`<title>.md`用于我们写博客，`<title>`目录用于存放我们博客使用的图片。

文章的配置，下面是我本篇文章的配置，大家可以参考：
```yml
title: 基于Hexo的个人博客搭建
date: 2020-08-30 10:45:29
tags:
  - hexo
  - blog
categories: hexo
keywords: hexo,Butterfly
description: 搭建博客其实很简单，但要注意很多细节问题，之前本人也搭建过博客，那时也踩过一些坑，但是现在重新搭建博客，感觉自己又把之前踩过的坑又踩了一遍。所以为了避免以后搭建博客采坑，所以在此记录一下搭建博客的全流程以及一些注意事项。
cover: /img/hexo.jpg
top_img: hexo.jpg
```

- title：文章标题
- tags：文章标签
- categories：文章分类
- cover：文章封面
- top_img：文章顶部图片

## github托管

将我们博客的源代码托管到github上，在github上创建代码仓库，将我们博客的源代码push上去即可。

## github pages
新建一个repo，仓库名字为`xxx.github.io`，其中xxx为你github的username。

## coding pages
- 1.新建一个`coding.net`团队，如果你已经拥有自己的团队，那么久不需要创建团队了。
- 2.创建一个DevOps项目。
- 3.在项目下创建代码仓库，用于托管博客静态资源，仓库名字为`xxx.coding.io`。

![project](project.png)

## 自动部署
- 1.在博客根目录下创建`.travis.yml`文件，并写入自动化脚本：
```yml
language: node_js
node_js:
  - 10
cache:
  directories:
    - node_modules
before_install:
  - npm install hexo-cli -g
install:
  - npm install
script:
  - git clone -b 3.0.1 https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly
  - cp -rf source/img/* themes/butterfly/source/img/
  - hexo clean
  - hexo g
after_success:
  - git config --global user.name "${U_NAME}"
  - git config --global user.email "${U_EMAIL}"
  - git clone "https://${GH_TOKEN}@${GH_REF}"
  - cp -rf public/* yujihu.github.io/
  - cd ./yujihu.github.io
  - git add .
  - git commit -m 'travis-ci auto build yujihu-blog'
  - git push origin "${P_BRANCH}"
  - cd ../
  - git clone "https://${U_NAME2}:${CO_TOKEN}@${CO_REF}"
  - cp -rf public/* yujihu.coding.io/
  - cd ./yujihu.coding.io
  - git add .
  - git commit -m 'travis-ci auto build yujihu-blog'
  - git push origin "${P_BRANCH}"
branches:
  only:
    - master
```

- node_js: 与我们本地开发的node.js版本保持一致。
- before_install：全局安装hexo脚手架。
- install：安装博客的依赖。
- script： 此处为生成博客静态资源的代码，需要将source/img复制到themes/butterfly/source/img下面，否则会导致图片无法访问。
- after_success：生成成功后，将静态资源分别提交至github pages和coding pages仓库。
- 环境变量：`${***}` 这块后面会提到。

- 2.Github 增加一个 Personal access tokens，位置在[Settings/Developer settings](https://github.com/settings/tokens)。

![token](token.png)


- 3.Coding 增加一个访问令牌，位置在`个人账户设置/访问令牌`。 

![token](coding.png)

- 4.进入[Travis CI](https://travis-ci.com/)，使用 Github登陆， 进入[dashboard](https://travis-ci.com/dashboard)，此时应该可以看到你刚创建的项目。

![travis1](travis1.png)

- 5.进入该项目的setting，配置环境变量

![travis2](travis2.png)

> GH_REF: Github Pages项目地址`github.com/[name]/[name].github.io.git`注意去掉 https://。
> CO_REF: Coding Pages项目地址`e.coding.net/[name]/[name].coding.io/[name].coding.io.git`注意去掉 https://。
> GH_TOKEN: Github Token，是通过上面第2步拿到的。
> CO_TOKEN: Coding Token，是通过上面第3步拿到的。
> P_BRANCH: 要上传的分支，这里我们要传到 master。
> U_EMAIL: 你的 Github 邮箱。
> U_NAME: 你的 Github 用户名。
> U_NAME2: 你的 Coding 用户名，这个很关键，一定要设置正确。

- 6.将博客源代码push到远程仓库，我们可以发现`Travis CI`会开启一次自动构建。

## 域名绑定
1.在腾讯云上购买一个域名，首年一般很便宜，大家可以先买一年试试，域名可以不备案，但需要实名认证。
2.开启Github Pages，在[name].github.io的setting中进行配置。

![github2](github2.png)

3.开启Coding Pages, 在`项目/持续部署/静态网站`下进行配置。

![coding2](coding2.png)
![coding3](coding3.png)

4.配置域名解析

进入刚刚购买域名的管理控制台，配置域名解析规则，境内CNAME到Coding Pages，境外CNAME到Github Pages.
