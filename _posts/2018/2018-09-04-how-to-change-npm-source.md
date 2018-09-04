---
layout: single
author_profile: true
title: "如何更改NPM源-阿里示例"
date: 2018-09-04 14:24:53
# toc: true
tags:
  - linux
  - npm
  - 前端
categories:
  - npm
  - 前端
---

默认的npm下载地址：[http://www.npmjs.org/](http://www.npmjs.org/)   
淘宝npm镜像的地址：[https://npm.taobao.org/](https://npm.taobao.org/)

临时使用阿里源  
    npm --registry https://registry.npm.taobao.org install node-red-contrib-composer@latest

全局配置切换到淘宝源  
    npm config set registry https://registry.npm.taobao.org

全局配置切换到官方源  
    npm config set registry http://www.npmjs.org

检测是否切换到了淘宝源  
    npm info underscore

```
.......

gitHead: 'e4743ab712b8ab42ad4ccb48b155034d02394e4d',
  dist: 
   { shasum: '4f3fb53b106e6097fcf9cb4109f2a5e9bdfa5022',
     size: 34172,
     noattachment: false,
    //　有　registry.npm.taobao.org　等字样　　说明切换成功
     tarball: 'http://registry.npm.taobao.org/underscore/download/underscore-1.8.3.tgz' },
  directories: {},
  publish_time: 1427988774520 }
```