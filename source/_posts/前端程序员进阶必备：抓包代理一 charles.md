---
title: 前端程序员进阶必备：抓包代理一 charles抓包
date: 2022-03-30 10:24:27
tags: 
author: 李珍改
keywords: 抓包 代理 charles
---
## 前端程序员进阶必备：抓包代理一 charles抓包

> 什么是抓包，为什么要抓包，不用多讲，大家都已经很清楚了

抓包代理可以说是前端同学的日常必备了，尤其是在我开始接触RN项目开发的时候，不会抓包真的感两眼抹黑，编码无力。下面就介绍一下我常用的几款抓包工具

1、charles

2、whistle

3、fillder



本次先介绍Charles



一、下载charles包

https://www.charlesproxy.com/download/

二、安装证书

​	1. 电脑安装SSL证书

​			Charles－》Help－》SSL Proxying－》Install Charles Root

![img](https://upload-images.jianshu.io/upload_images/17053865-cb4dee283eb7a355.gif)

​	2. 浏览器安装ssl证书

​			Charles－》Help－》SSL Proxying－》Install Charles Root Certificate on a Mobile

![img](https://upload-images.jianshu.io/upload_images/17053865-e3c5db5c9a789ada.gif)


三、移动app抓包

1、使手机和电脑在一个局域网内，不一定非要是一个ip段，只要是同一个路由器下就可以了；

2、Charles菜单栏“Proxy->Access Control Settings”设置允许接收的ip地址的范围

例：如果接收的ip范围是[http://192.168.1.xxx](https://link.zhihu.com/?target=http%3A//192.168.1.xxx)的话，那么就添加并设置成192.168.1.0/24 如果全部范围都接收的话，那么就直接设置成0.0.0.0/0

3、手机端的WiFi代理设置手动进行相关配置

①代理服务器地址填写电脑的IP地址（cmd-ipconfig命令查看IPV4；或Charles菜单栏“Help->Local IP Address”中查看）

②端口填写8888（Charles的默认代理端口，可在菜单栏“Proxy->Proxy Settings”中修改）

4、配置好之后，打开手机上的任意需要网络通讯的程序，就可以看到 Charles 弹出请求连接的确认菜单（如下图所示），点击 “Allow” 即可完成设置。


![企业咚咚20220412105401](https://storage.360buyimg.com/imgtools/8c8b7f8ab2-f81783e0-ba0b-11ec-ae49-a3c46816489f.jpeg)