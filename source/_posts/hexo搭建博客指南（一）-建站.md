---
title: hexo搭建博客指南（一）--建站
date: 2017-02-16 15:08:29
categories: hexo
tags:
- hexo
---
### 一 hexo简介   

> hexo是一款快速，高效，简洁的博客框架。

### 二 搭建博客   

#### 2.1 安装环境

&emsp;&emsp;hexo是基于node.js的，因此要想使用hexo，必须安装node环境，同时还需要git，用于将hexo部署到github上。所需软件：  
node.js &emsp;&emsp;下载链接：[http://nodejs.cn/download/](http://nodejs.cn/download/)  
git &emsp;&emsp;&emsp;&emsp;下载链接：[https://git-for-windows.github.io/](https://git-for-windows.github.io/)  


下载后直接安装，完成后打开windows的cmd，输入以下命令：  

``` bash
node -v    
npm -v 
```        

当出现以下信息时，则说明安装成功。    
   
``` bash
zhengrui-ds@ZHENGRUI-DS MINGW64 ~
$ node -v
v6.9.4

zhengrui-ds@ZHENGRUI-DS MINGW64 ~
$ npm -v
3.10.10
```


#### 2.2 安装hexo    

安装客户端   

```bash
npm install -g hexo-cli    
```   

安装服务端  

```bash
npm install hexo --save   
```   

新建一个用于装在hexo的文件夹，如hexo  
切换到该文件夹，执行以下命令：   

```bash
 hexo init  //初始化该文件夹  
 npm install  //安装相应的依赖包
```    

此时hexo搭建博客已基本完成。可执行以下命令来查看博客的初始效果：  

```bash
	hexo clean   //清空public文件夹下面的内容，该文件夹用于存放生成的网页文件。  
	hexo g       //该命令用于产生相应的网页文件，在public文件夹下  
	hexo s      //启动一个本地服务器，一般是http://localhost:4000，可查看生成的网页情况。
```    

**注意**：有时会存在4000端口被占用的情况，此时可以使用hexo s -p 端口号&emsp;来指定端口号。其实如果你有使用linux的经验，应该知道这些命令一般都可以在命令的后面加上-h或者--help来查看它的用法。hexo中的命令可以连写，如hexo g与hexo s可合写成hexo s -g  

#### 2.3 将博客部署到github上  
&emsp;&emsp;要想将博客部署到github上，你需要先有一个github账号，并且将你本地的git（计算机）与github关联起来。本文已认为你已经有了自己的github账号并已关联本地，如果没有请移步我的另一篇博客TODO  
其次你需要了解一下hexo的配置文件_config.yml：  
站点配置
``` bash
# Site
title: ZHENGRUI'BLOG
subtitle: 今日之果，皆因曾经之因
description:
author: zhengrui
language: zh-CN
timezone:
``` 
将博客部署到github上  
``` bash
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:zhezaoyizhuding/zhezaoyizhuding.github.io.git
  branch: master
```  
你需要执行以下命令，安装hexo的git插件  

```bash
npm install hexo-deployer-git --save
```    

后面要加--save否则无法将依赖写入hexo的package.json中，执行以下命令即可将网站部署到github上。  

```bash
hexo d
```  

至此你一讲将博客部署到github上了，在浏览器输入https://你的github用户名.github.io即可访问。